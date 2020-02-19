---
layout: page
title: Navigation through items and attempt/result creation
nav_order: 300
parent: Work in Progress
---

# Navigation through items and attempt/result creation

From such an interface:

<img src="https://france-ioi.github.io/algorea-designs/img/03.Activities_00.Header_d.Attempt_a.Open.png" >

We assume the frontend gets as URL something like: `http://www.example.info/content/1/2/3/4#9/5` (in whatsoever format), where `1/2/3/4/5` is the breadcrumb, ending by the content id `5` (to be displayed in the right pane), and `9` is the `attempt_id` to be displayed, here on `4` (but it may be on any item). The breadcrumb is always given. As we may get shared/external URL, the attempt may not be given. If several attempts are given, only the last one is relevant.

### The attempt parameter

If given, the attempt is the context in which the item has to be displayed, so constraining the attempt of all items to its left (`1`,`2`,`3`,`4` here). The attempts for the items on its right (`5` here) are decided by other rules defined below (in "The content").

If the frontend knows (has saved) what attempt to use for one of the item on the right, it should rewrite the URL using these preferences. In our example, it would need to know what attempt to use for the current user, item `5` under attempt `9` (so `9` itself or a forked attempt from `9`).

To give other examples: If the path is `/content/1/2#9/3/4/5`, the item `1` and `2` have to be on attempt `9` or its ancestors, while items `3`, `4`, and `5` are not constrained. If the path `/content/1/2/3/4/5#9`, the full path is constrainted.

If no attempt is given and the frontend does not anything about attempt for this case, attempt `0` is used.

### The default result for an item

TODO: on the left of the constraint: via a the attempt, on the right, the one with the most recent "latest_activity"?

### Loading the components

Once the URL has been parsed and the attempt deduced, the header, left menu, and right content have to be loaded. This should be doable in parallel (without need from the result from one to load the other) so that it is faster and each frontend component can be independant.

We need services for: (described in details below)
- the breadcrumb
- the left menu (load the tree and load a subtree when "+" is pressed)
- the content (right pane)

## Loading the breadcrumb

From a list of `item_id` and an attempt (given or deduced, see above) on one of the item, the breadcrumb service:

- verifies that the item list forms a valid path (each is the parent of the next)
- verifies that the first item is a root (`is_root`) or the item of a group the current user is member of
- for all items, returns
  - the item title (in a appropriate language)
  - whether this participant has several results (within the parent attempt) for this item
  - if any, the default result (see above) info:
    - the attempt_id
    - its order among the other results (within the parent attempt) using an order on `started_at`

### Frontend

As shown [here](https://france-ioi.github.io/algorea-designs/03.Activities_00.Header_d.Attempt), the frontend does display the order only if there is more than one attempt for this item within its parent and the result has an order.

If a forbidden (403) error is received, the UI should retry to refresh this page by dropping the attempt (may be copy/pasted url with invalid attempt).

For link attached to each element of the breadcrumb, the attempt given is right-most known attempt for the given element.

## Loading the left menu

From the id of parent item and the result id so (item_id + attempt_id + implicit participant_id), returns:
- for the parent:
  - item id, item title, item type,
  - access rights
- for each child of the parent:
  - item id, item title, item type, explicit_entry,
  - whether it has children,
  - access rights,
  - best score: among all attempts of the participant on this item,
  - score, validated: among all results of this participants, on this item, within its parent attempt

Opening a chapter (by pressing the "+" button in front of it), request the same info, rooted on the expended chapter.

### Links to items

To build the links to the parent and child items, the frontend can easily
TODO


## The content (right pane)

The frontend needs to:
- load the item info (contain )
- load all results of attempt on this item (within its parent)
- if no result, can_view>=content and not an explicit-entry item, start a result (if *can_view>=content*)
- request a task token

When exercice "4. Exercice lorem ipsum", the menu is loaded rooted at its parent, with no children listed by default (if the page was a chapter). The page knows the id of the parent through the breadcrumb given to the frontend via the URL.

So the request looks like: `GET /attempts/{attempt_id}/items/{item_id}/as-nav-tree`. The `group_id` is implictely the current user, if he is in "team mode", the optional query string `?as_team_id={group_id}` is added. If not known, the `attempt_id` given can be "0".

This results returns for instance for `GET /attempts/{attempt_id}/items/{item_id}/parent-as-nav-tree`
```json
{
  "id": "123",
  "string": {
    "title": "Entry reading",
    "language_tag": "fr"
  },
  "type": "Chapter",
  "allows_multiple_attempts": false,
  "explicit_entry": false,
  "access_rights": {
    "can_view": "content",
    "other_perm": "..."
  },
  "best_score": 80, /* among all participant's attempts, even different from attempt_id and children */
  "score": 50, /* score from result given as input */
  "validated": false, /* validated from result given as input */
  "children": [
    {
      "id": "456",
      "string": {
        "title": "4. Exercice lorem ipsum",
        "language_tag": "fr"
      },
      "type": "Task",
      "allows_multiple_attempts": true,
      "explicit_entry": false,
      "access_rights": {
        "can_view": "content",
        "other_perm": "..."
      },
      "order": 0,
      "best_score": 100, /* among all participant's attempts, even different from attempt_id and children */
      "results": [ /* all results (possibly none) children of the parent attempt_id given of as input */
        {
          "attempt_id": "111",
          "score": 100,
          "validated": true,
          "started_at":
        }
      ]
    },
    {
      "same attributes": "..."
    }
  ]
}
```

