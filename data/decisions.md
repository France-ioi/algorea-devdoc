---
layout: page
title: Change Decisions
nav_order: 20
parent: Data Model
---

# Design decisions

Here is a list of the decisions on database design which have been done recently, but usually not fully applied on the db schema currently for backward-compatibility reasons.

* 18/06/20 - Mathias, Damien. `groups_attempts.nbSubmissionsAttempts` should be renamed to something like `nbSubmissionsBeforeCompletion` and only updated while the task is not validated.
* 20/05/19 - Michel, Damien, Dmitry. Every submission has an attempt. If `items.bHasAttempts` is false, there can be only one attempt. As a consequence, many columns are not deprecated in `users_items` as they would be duplicated in `groups_attempts`.  See [this page](../database/answers_attempts)
* 22/05/19 - Michel, Damien, Dmitry. Clean format of token messages. See token page for more info.
* 05/19 - Michel, Damien, Dmitry. `sState`/`sAnswer` of both `users_items` and `users_answers` are currently updated at the same time (duplicated) on purpose during the transition to a better system. See [this page](../database/answers_attempts) for more details.
* 14/08/19 - Michel, Damien about additional time:
  * On 08/2019, extra time given to a user or group for a contest is managed in the db either through `users_items ` or `groups_attemps` (not used actually). Storing it in `users_items`, keyed by `user_id,item_id` is not suitable from team participation, storing it in `groups_attempts`, keyed by `group_id, item_id, attempt_id`, make it attempt-specific (so can be resetted), while, in practice, in a contest, the time does not have to be resetted for each attempt (case of Alkindi contest). This extra time should then be stored in a table keyed by `group_id, item_id`. This is the case `groups_items` table, the same used for permission.
  * One extra benefit of that is not to apply extra time on end-users, but also on groups, allowing the give 2min extra to the whole group, 10min to a groups with disabilities, and 5min to a specific user which has a technical problem, causing him to get possibly 17min extra.
  * The actual additional time is computed as the sum of all the additional time his group and all its ancestors got on the contest item.
  * For the transition, all past additional times can be dropped.
  * For the services to change additional time, we give to the frontend the computed additional time and the additional to the group, and let it return the new value for the group. It might be interesting to prevent parallel modification by sending the expected value so that the backend may fail if it does not match.
* Discussion on 6/09/2019 - Mathias, Damien
  * Replacing the current way the contest qualification work (currently: for teams, requiring to have a given number of users in a group; for users, requiring gray access on the contest itself and so an addition to a parent chapter to give access to this contest at a specific time; when ok, give a new partial permission which will be removed at the end of the contest):
    - New attribute in `groups_items`: `bGrayedAccessDate` allowing to give explicitely gray access (currently it is only obtained from propagation).  It does not propagate through the item hierarchy (as it is now).
    - New attributes in `groups_items`: `canEnterFrom` and `canEnterUntil`, defining a time slot during which you can join (typically through the "start contest" button) the `items.idAccessGroup`, which is a group with partial access on the item.
    - For teams, the condition "having a given number of members in a group" is transformed to "having all grey access to the contesnt and having a given number of members with enter-access"
  * Replacing the current way to end a contest (currently: check at each access if the time is not over for the user and remove him the access if no)
    - Every group membership (`groups_groups`) gets a expiration time. This expiratin time is used in the abovementioned `items.idAccessGroup` so that the user/team access expires once his time is over.
  * Replace unlockItem by "joinGroup": solving an item can make the user join a group which has access (instead of creating partial access)
* 16/09/2019 - Mathias & Damien
  * The combination of `items_items.bAlwaysVisible` and `items_items.bAccessRestricted` can represents 4 states while actually they are used to represent 3 possible ways of propagating partial access from an item to its child. We will merge these attribute to one `items_items.partial_propagation` (name to be defined) allowing 3 values: `no`, `yes`, `as grey` (to be defined).
