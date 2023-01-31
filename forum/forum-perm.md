---
layout: page
title: Forum Permissions
nav_order: 20
parent: Forum
---

# Forum Permissions

In a nutshell:
- a user requires a specific permission to open a thread
- a user with observation permissions (on item and participant) can view, open and answer threads
- other users may be allowed to view and answer threads on some conditions

## Specific permission

In order to open a thread for an item, a user needs a specific item permission: "can request help to" which allows the user to request help to a given group for an item and its descendants depending on propagation rules.

### Granting the permission to another user

To grant `can_request_help_to(g)` permission to a user for an item, the giver needs to have "can_grant_view ≥ content" for this item.
The group `g` needs to be visible to both the giver and the receiver, or be the all-users group of the plaforom.

There is no constraint on the prior permissions of the receiver.

### Propagation

This permission propagates as the other item permissions with a specific `request_help_propagation` boolean property on `items_items`, which behaves as the other propagation rules. The propagation of the permissions through items is not pre-computed (it cannot be as the `group` parameter could not be).

### Default and changing propagation rules

In order to change the `request_help_propagation` propagation rule between a chapter and its child to `true`, the permission requirement on the child item is to have "can_grant_view ≥ content". The permission requirement on the parent is [the same as the other permissions]({{ site.baseurl }}/design/access-rights/items/#creating-defaults-and-changing-propagation-rules).

### In practice

In practice, in order to verify that the user `U` "can request help to" a group `G` on an item `I` we need to verify whether one of the ancestors (including himself) of `U` has the `can_request_help_to(g)` on `I`, recursively on `I`'s ancestors while `request_help_propagation=1`, for `G` being a descendant of `g`.

## Writing in a thread

In order to write in a thread, the **thread needs to be opened** and the user needs to either:
- be the participant of the thread
- have `can_watch>=answer` permission on the item AND `can_watch_members` on the participant
- be part of the group the participant has requested help to AND either have `can_watch>=answer` on the item OR have `validated` the item.

## Listing / reading a thread

In order to have a thread listed and be able to view it (in read-only if not authorized to write), the user needs to be allowed to view the item AND match one of these conditions:
- be the participant of the thread
- have `can_watch>=answer` permission on the item
- be part of the group the participant has requested help to, AND the thread is either open or closed for less than 2 weeks, AND ~~either the user has `can_watch>=answer` on the item OR (duplicate from previous rule)~~ the user has `validated` the item

## Changing the status of a thread

- The **participant of a thread** can always switch the thread from open to any another other status. He can only switch it from not-open to an open status if he is allowed to request help on this item (see "specific permission" above)
- A user who has `can_watch>=answer` on the item AND `can_watch_members` on the participant: can always switch a thread to any open status (i.e. he can always open it but not close it)
- A user who can write on the thread can switch from an open status to another open status.

