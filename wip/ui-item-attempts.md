---
layout: page
title: Navigation through items and attempt/result creation
nav_order: 300
parent: Work in Progress
---

# UI Workflow for loading an item on an attempt

From such an interface:

<img src="https://france-ioi.github.io/algorea-designs/img/03.Activities_00.Header_d.Attempt_a.Open.png" >

We assume the frontend gets its path in one the following form: `/1/2/3/4/5?attempt_id=9` or `/1/2/3/4/5?parent_attempt_id=9` or `/1/2/3/4/5` (in whatsoever format), where `1/2/3/4/5` is the breadcrumb, ending by what we will call the **current item** `5` to be displayed in the right pane, and either the attempt of the current item (`5`) or of its parent (`4`), or no attempt.

The breadcrumb is always given and a list of items from a root item to the current item.

When navigating to item ancestors, children, or siblings, the frontend must always provide the attempt to stay in a similar "attempt context" as the previous page. When a link is shared with other users or when we link to an independant item, the attempt may be missing. In this case, the frontend has first to retrieve the attempt context before loading the content.

Providing the parent attempt (`parent_attempt_id` parameter) is reserved for cases when the user does not have attempts for the current item.

### Finding the attempt when it is not provided

If no attempt (so no `attempt_id` nor `parent_attempt_id`) is given, the frontend has to get it before displaying the page. Either the frontend remember the last attempt used by the current user when last visiting the current item or its parent, or it requests this information to the backend. In both case, the url is then rewritten with the attempt in it.

TODO start or create result?

### Loading the components

Once the URL has been parsed, the header, left menu, and right content have to be loaded. This should be doable in parallel (without need from the response from one to load the other) so that it is faster and each frontend component can be independant.

We need services for: (described in details below)
- the breadcrumb
- the left menu (load the tree and load a subtree when "+" is pressed)
- the content (right pane)

## Loading the breadcrumb

From a list of `item_id` and a current-item or parent attempt, the breadcrumb service:

- verifies that the item list forms a valid path (each is the parent of the next)
- verifies that the first item is a root (`is_root`) or the item of a group the current user is member of
- for all items, returns
  - the item title (in a appropriate language)
  - attempt id for this item (result) from which the current-item or parent attempt is a descendant (null for the current item if parent attempt was given)
  - if the item allows multiple attempts, the order of this attempt result among the other results (within the parent attempt) using an order on `started_at`

The attempt of each item is either the same attempt as its child or the parent attempt of this attempt.

### Frontend

As shown [here](https://france-ioi.github.io/algorea-designs/03.Activities_00.Header_d.Attempt), the frontend does display the attempt order in the breadcrumb, but only for items which supports multiple attempts.

If a forbidden (403) error is received, the frontend should retry to refresh this page by dropping the attempt (may be copy/pasted url with invalid attempt).

## Loading the left menu

From the id of the parent item and the attempt id (of itself or one of its children), returns:
- for the parent:
  - item id, item title, item type,
  - access rights
  - attempt id
- for each child item of the parent:
  - item id, item title, item type, explicit_entry,
  - whether the item has children,
  - access rights,
  - best score: among all attempts of the participant on this item,
  - a list of the results within its parent attempt, with:
    - attempt id
    - score
    - validated
    - start time
    - latest activity

The path of the link to each child uses as attempt:
- if the child has no results, the parent attempt
- if the child has one result, use its attempt
- if the child has several results, use the most recent attempt known by the frontend for this participant and item (if any)
- otherwise the attempt with the most recent activity

### Expending a chapter

When expending a chapter in the left menu ("+" button or arrow), the same query as the one loading the menu is called, with expended chapter id and, as attempt:
- if the item is the current page and an attempt is given, use this attempt
- if this item has no results yet, create one (TODO, start or create?) and use its attempt
- it it has attempts, use the latest attempt known by the frontend for this participant and item
- if the frontend has no saved preferences, use the attempt with the most recent activity

## The content (right pane)

The frontend needs to:
1. load the item info, from the item id and its current page attempt id or the attempt of its parent
  - info from the `items` table
  - a list of the results within its parent attempt, with:
    - attempt id
    - attempt creation time
    - attempt creator
    - score
    - validated
    - start time
    - latest activity
2. select the attempt (only if the current user has can_view>=content)
  - if there no result and it is not an explicit-entry item: create one (i.e., start it) giving the parent attempt
  - if there is one result but not started (`started_at` is null) and it is not an explicit-entry item: start it, giving the attempt
  - if there is one result already started, use this one
  - if there are multiple results, the frontend chooses the most recent known for this participant and item
  - if there are multiple results and the frontend does not have preferences, use the attempt with the most recent activity
  - otherwise, do not select attempt
3. if an attempt has been selected and the item is a task, request a task token

TODO: 2 options, we consider an attempts not started as "non existing", or we just start it implicitely afterwards
