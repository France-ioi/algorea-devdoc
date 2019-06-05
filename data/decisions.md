---
layout: page
title: Change Decisions
nav_order: 20
parent: Data Model
---

# Design decisions

Here is a list of the decisions on database design which have been done recently, but usually not fully applied on the db schema currently for backward-compatibility reasons.

* May 2019 - Michel, Damien, Dmitry. Every submission has an attempt. If `items.bHasAttempts` is false, there can be only one attempt. See [this page](../database/answers_attempts)
* May 2019 - Michel, Damien, Dmitry. `sState`/`sAnswer` of both `users_items` and `users_answers` are currently updated at the same time (duplicated) on purpose during the transition to a better system. See [this page](../database/answers_attempts) for more details.
