---
layout: page
title: Group's root item
nav_order: 80
parent: Groups & Related Entities
---

<span class="label label-yellow">Draft / WIP</span> Results of discussions between Michel, Mathias and DLe on 26-27/11/19.

# Group's Root Items (Sharing Content with Members)

A **group root item** is an optional chapter item that can be used for gathering content which is shared with members of a group. It is managed by group managers which can use it to structure content shared with members and also to make sure that everybody has a path to this content.

From the group member's point of view, this item is displayed as a root item in his navigation menu. It allows him to access group content directly.

## Creation/Deletion of the group root item

Any manager with *can_manage:memberships_and_group* can request the creation/deletion of this item (maximum one per group) through a specific service. The group (so its members) gets view access (*can_view:content*) to it. Each of the current managers (with *can_manage:memberships_and_group*) gets *can_edit:children* and *can_view:content* on it.

## Refresh of the manager permissions

Each time a manager permission is changed to or from *can_manage:memberships_and_group* (so if added/removed), the permissions given to the managers needs to be refreshed.

## Database implementation

This item is implemented as `root_item_id` in the `groups` table. Permissions granted to members and managers are stored in the `granted_permissions` table.

<span class="label label-red">Yet to be solved</span>

** are permission refreshes automatic or suggested to the user?

** remaining problem: if a user is both user and mananger, how do we know if a "can_edit:children" permission has been given by the managership and has to be removed is the user is not manager anymore, or if the permission has been given to the user directly.
