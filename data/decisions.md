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
