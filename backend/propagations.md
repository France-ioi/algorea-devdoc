---
layout: page
title: Propagations
nav_order: 550
parent: Backend
has_toc: true
---

# Propagations

In this page are listed all types of propagations that happen on the backend.

{: .no_toc}

1. TOC
{:toc}

## Types of propagations

### Propagation of groups_ancestors

Implemented in `DataStore.createNewAncestors()` with parameters ("groups", "group").

This process creates new entries in `groups_ancestors`.

The table `groups_ancestors` makes it easier to write queries on groups.

It contains entries with primary keys `ancestor_group_id`/`child_group_id`.
Note: **A group is always considered to be its own ancestor**, so there is always an entry where both keys have the same id.
For those entries, `is_self`=1.

Inserts new rows in `groups_ancestors` for all entries in `groups_propagate` with `ancestors_computation_state`="todo",
and all their descendants.

`groups_propagate`/`items_propagate` entries are added in SQL Triggers:
- after_insert_items/groups
- after_update_groups_groups
- before_insert_items_items/groups_groups
- before_delete_items_items/groups_groups


### Propagation of items_ancestors

Implemented in `DataStore.createNewAncestors()` with parameters ("items", "item").

Works similarly than [groups_ancestors](#propagation-of-groups_ancestors), with one big difference:

Unlike for groups where a group is always its own ancestor, **an item is never its own ancestor**.


### Propagation of permissions

Implemented in `PermissionGrantedStore.computeAllAccess()`.

Starts from group-item pairs marked with `propagate_to` = 'self' in `permissions_propagate`.
Those are marked in SQL Triggers:
- `after_insert_permissions_granted`
- `after_update_permissions_granted`
- `after_delete_permissions_granted`
- `after_insert_items_items`

**Important:** Propagation of results may have to follow propagation of permissions,
because the SQL triggers called for `permissions_generated` set rows to treat in `results_propagate`,
when an item becomes visible.


### Propagation of results

It is implemented in `ResultsStore.propagate()`.

Treat all `results_propagate` rows marked as "to_be_propagated" or "to_be_recomputed".
Those are marked either in SQL queries, or in SQL Triggers:
- after_insert_groups_groups/items_items
- after_insert_permissions_generated
- after_update_groups_groups/items_items
- after_update_permissions_generated
- before_delete_items_items

This process is called at the end of the current or next transaction when it is scheduled.

**Important:** If groups are unlocked during this propagation, we must run propagation of permissions again,
and then propagation of results again.


## Implementation of propagations

Propagations can take a long time to compute, especially when there are many entries to treat.
We had cases where the maximum execution time of a call was reached on AWS Lambda.

To avoid this, we implemented a system where the propagation is done in small chunks.

So, propagations are done in a loop (called a propagation step), and each step happens in a transaction.
In this way, if the maximum execution time is reached, we don't lose all the work done so far, but only the current step.

There are two ways to run a propagation: synchronously and asynchronously.

The current plan is to run the `items_ancestors`, `permissions`, and `results` propagations asynchronously,
because they can be long and we don't want to block the execution of the service call.

The `groups_ancestors` propagation can be run synchronously, because it is fast, and we have cases where we need
its result immediately:
- The `CreateRelation` function, used in the `groupAddChild` and the `accessTokenCreate` and `refreshToken` (for the badges) services.
- The `DeleteRelation` function, used only in the `groupRemoveChild` service.

If at some point we need to run the `groups_ancestors` propagation asynchronously, we need to adapt those two services
so that they don't need the result of the propagation immediately.


### Synchronous propagation

When called synchronously, during the execution of the service call,
the propagation is done at the end of the current transaction, after its commit.

Since propagations can be long, we want to avoid this method and do all propagations asynchronously (in progress).

To run a propagation synchronously, you can call the following method:

```
  DataStore.ScheduleResultsPropagation()
  DataStore.ScheduleGroupsAncestorsPropagation()
  DataStore.ScheduleItemsAncestorsPropagation()
  DataStore.SchedulePermissionsPropagation()
```


### Asynchronous propagation

#### Propagation endpoint

To run a propagation asynchronously, you first have to make sure that a propagation endpoint is specified in the config:

  ```yaml
    propagation:
      endpoint: "http..."
  ```
If it's not, the propagation will be done synchronously.

The endpoint must be a GET endpoint that receives a list of propagations to run in the types query parameter.
(Example: `http://localhost:8080/propagate?types=results,groups_ancestors,items_ancestors,permissions`).

If the endpoint is not reachable or returns a code != 200, the propagation will be done synchronously.

#### Call the propagation asynchronously

To run a propagation asynchronously, call the following method:

```
  service.SchedulePropagation(store, srv.GetPropagationEndpoint(), propagationsToRun)
```

Where `propagationsToRun` is a list of strings, each string being the name of a propagation to run, such as:
- "results"
- "groups_ancestors"
- "items_ancestors"
- "permissions"

Note of implementation: for now, the types are not used, each asynchronous propagation call will run the
`items_ancestors`, `permissions`, and `results` propagations. But it's a good idea to set the right types
so we keep track of which type of propagation has to be run and where.
