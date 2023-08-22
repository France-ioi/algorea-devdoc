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


## Propagation of groups

This process creates new entries in `groups_ancestors`.

Implemented in `GroupGroupStore.After()`.

In many places, it is also called as `GroupGroupStore.createNewAncestors()`.
We should check if it is really necessary to have two different ways to call this process.
It would be better if we can unify them.


### Creation of `groups_ancestors`

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


## Propagation of items

Implemented in `ItemsItemsStore.After()`.

This is a three-step process:


### 1. Create new items_ancestors

Works similarly than [groups_ancestors](#creation-of-groups_ancestors), with one big difference:

Unlike for groups where a group is always its own ancestor, **an item is never its own ancestor**.


### 2. Call the propagation of permissions

See section [Propagation of permissions](#propagation-of-permissions).


### 3. Call the propagation of results

See section [Propagation of results](#propagation-of-results).


## Propagation of permissions

Starts from group-item pairs marked with propagate_to = 'self' in `permissions_propagate`.
Those are marked in SQL Triggers:
- after_insert_permissions_granted
- after_update_permissions_granted
- after_delete_permissions_granted
- after_insert_items_items

**Important:** Propagation of results may have to follow propagation of permissions,
because the SQL triggers called for `permissions_generated` set rows to treat in `results_propagate`.
The case documented in the code is: when an item becomes visible.
But it should be verified whether this is the only case.

Implemented in `PermissionGrantedStore.computeAllAccess()`.

Also implemented in `PermissionGrantedStore.After()`. It would be nice to have only one way to call it.
We should check if it is really necessary to have two different ways to call this process.
It would be better if we can unify them.


## Propagation of results

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

To schedule it, `DataStore.ScheduleResultsPropagation()` has to be called.
It is implicitly called when `ResultsStore.MarkAsToBePropagated(participantID, attemptID, itemID)` is called.

It is implemented in `ResultsStore.propagate()`.

