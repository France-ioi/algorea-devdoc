---
layout: page
title: Forum Permissions
nav_order: 20
parent: Forum
---

# Forum Permissions

In a nutshell:
- a user requires a specific permission to open a thread on an item which is described belo
- a user with observation permissions (on item and participant) can view and answer threads
- other users may be allowed to view and answer threads on some conditions

## Specific permission

In order to be allowed to open a thread on an item, a user needs a specific item permission: "can request help to" which allows the user to request help to a given group on an item and its descendants depending on propagation rules. (FIXME problem: not easy (not currently possible) to propagate perm with an non linear arg).

### Propagation

This permission propagates as the other item permissions with a specific `help_request_propagation` boolean property on `items_items`, which behaves as the other propagation rules.

### Changing and propagating permissions

Constraint on receiver: none

Constraint on giver: requires "can_grant_view ≥ content"

## Writing in a thread

In order to write in a thread, the **thread needs to be opened** and the user needs to either:
- be the participant of the thread
- have `can_watch>=answer` permission on the item AND `can_watch_members` on the participant
- be part of the group the participant has requested help to AND (have `can_watch>=answer` on the item OR have `validated` the item)


## Listing / reading a thread

In order to have a thread listed and be able to view it (in read-only if not authorized to write), the user needs to be allowed to view the item AND match one of these conditions:
- be the participant of the thread AND
    - if the thread is open: nothing more (TODO: CHECK)
    - if the thread is closed: yes (TODO: CHECK)
- have `can_watch>=answer` permission on the item
- be part of the group the participant has requested help to AND the thread is open, or closed for less than 2 weeks AND (the user has `can_watch>=answer` on the item OR the user has `validated` the item)

Note that it is possible to aggregate some part of the rule to make it shorter to implement.

## Changing the status of a thread

(TODO: check)

- the participant of a thread: can always switch a thread (TODO: check) from open to another other status. He can only switch it from not-open to an open status if he is allowed to request help on this item
- a user who has `can_watch>=answer` on the item AND `can_watch_members` on the participant: can always switch a thread from open to another status and can switch from not-open to an open status
- a user who can write on the thread can switch from an open status to another open status. He cannot switch from a not-open status to an open status. (TODO: can he close it?)

