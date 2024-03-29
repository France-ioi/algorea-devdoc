---
layout: page
title: Permissions in General
nav_order: 10
parent: Items & Related Entities
---

# Permissions and their propagation

The permission management system is at the heart of the platform, and is based on the following principles.

## Basics on permissions

The two main entities of concern for permissions are "[groups]({{ site.baseurl }}/groups/)" and "[items]({{ site.baseurl }}/items/)". **Groups** are being granted permissions on **items**.

### "Given" permissions

Groups may be given permissions on items (`permissions_granted` table) by a group manager or by the platform as a reward for solving an exercise.

When they first arrive on the platform, users are automatically added to groups that have some permissions on some items. This is how they gain access to the public content when they arrive on the platform. Users may also be added automatically to groups based on other criteria, such as being a verified teacher. Finally, users may join groups voluntarily (using a code, by invitation or on request). The groups a user belongs to will determine what permissions he has.

### Propagation

Permissions of a group to an item are propagated to the descendants of the groups
(but not from a group of type `Team` to its members),
and (under some conditions) to the descendants of the items.

<span class="label label-green">Key-concept</span>
Permission propagation from a parent **group** to its descendants is **implicit**. So if a user is part of a group, he has the same permissions as the group (except from team to users where no permission is propagated). On the other hand, permission propagation from a parent **item** to its descendants is **explicit**, i.e., new entries in the `groups_items` table are created each time new permissions are given to propagate these rights to the children. The propagation through items follows specific rules described in [this page]({{ site.baseurl }}/design/access-rights/items/).

## Specific permissions

### Granting additional time for timed activities

In order for a user to be able to grant (or change) additional time given to a group, for a timed item, the user needs:
* "can_grant_view ≥ enter" on the timed item
* "can_watch ≥ result" on the timed item
* to be a manager of the group with "can_grant_group_access" and "can_watch_members"

### Permissions to view other's permissions

In order for a `User` to view another permission given by `GroupSource` to `Group` on `Item`, the user requires:
* either the permission to **watch `Item`** ("can_watch ≥ result") and to **watch `Group`** ("can_watch_members" on an ancestor of `Group`).
* or the permission to **grant permission on `Item`** ("can_grant_view ≥ enter") and to **grant permission to `Group`** ("can_grant_group_access" of an ancestor of `Group`).
