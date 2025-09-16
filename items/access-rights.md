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

## Permissions on permissions

Notes:
* For the conditions below, do not forget that a group is a descendant of itself.
* grant permission on an item, can be achieved by `can_grant_view > 'none'` or `can_watch = 'answer_with_grant'` or `can_edit = 'all_with_grant'` on that item

### View generated permissions

In order for a `User` to view a permission given to `Group` on `Item`, the user requires:
* either the permission to **watch `Item`** ("can_watch ≥ result") and to **watch `Group`** ("can_watch_members" on an ancestor of `Group`).
* or the permission to **grant permission on `Item`** and to **grant permission to `Group`** ("can_grant_group_access" of an ancestor of `Group`).
* or to be a descendant of `Group` .
* or to be (implicit or explicit) a manager of `Group` with `level >= membership`

### Granted permissions

Let's consider a `User` and some granted permission from `GroupSource` to `Group` on `Item`.

Note that, by definition, a `Group` is always a descendant of `GroupSource`. As a consequence, if a user is a (implicit or explicit) manager or `GroupSource`, it is (at least implicitly) a manager of `Group`.

**To list the granted permissions which applies to a group and an item, the user has to be allowed to view the generated permissions for that pair (see the previous section). But that applies to the permissions themselves (`can_...`), not to the ids (group, source group and item) as otherwise that would allow to deduce what group a user is member of.**

#### Item info

Item ids and info are visible as long `User` can view the item (`can_view>='info'`).

#### Group and source group info

`User` can view the group and source info if:
* either
  * `User` is a descendant of `Group`
* or (as a manager with membership level can add himself into a group)
  * `Group` is not a user, and
  * `User` is (explicitly or implicitly) a manager of a descendant of `Group` with `level >= membership`
* or
  * `Group` is not a user, and
  * `User` is (explicitly or implicitly) a manager with "can_watch_members" or "can_grant_group_access" of `Group`
* or
  * `Group` is a user, and
  * `User` is implicitly a manager with "can_watch_members" or "can_grant_group_access" of `Group`, and
  * `User` is (explicitly or implicitly) a manager of a non-user descendant group of `GroupSource`

#### Edit (or create) a granted permissions

Only `origin: group_membership` permissions can be edited/created.

In order to give (or edit) a permission, `User` needs the permission to **grant permission on `Item`** and to **grant permission to `GroupSource`** ("can_grant_group_access" of an ancestor of `GroupSource`). In addition, it needs specific permission depending on the given permission value (e.g., giving `can_view:content` require `can_grant_view>=content`), cfr the item permission page.

### Discussions

The base for these decisions is the following:

1) a user should be able to see the permissions which applies to himself, and to see where (from what group) it comes from.

2) a user should not be able to deduce with 100% certainty the membership of another in a group that he does not manage

3) a manager with "watch" permissions on a (group, item) pair should be able to see permissions which related to it

4) a manager with "can grant" permissions on a (group, item) pair should be able to see and edit permissions which related to it

5) The manager with `membership` level on a group can add himself to the group. As a consequence, he can choose the have the permissions applying to him, so there is not point in not allowing him to see these permissions.

#### Interesting scenarios

1) A student is a member of a school class and of a coder dojo group (both groups are unrelated). The class teacher should not be able to see that there is a permission which has been given by the coder dojo group (which would allow to deduce that the student is a member of the dojo)

2) A class teacher (manager from the class) should be able to see the permissions given to the students of his class by the school (school is ancestor of the class) even if he is not manager from the school
