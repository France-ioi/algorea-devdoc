---
layout: page
title: Principles of access rights and their propagation
nav_order: 50
parent: Design
has_children: true
---

(from [Principles of access rights and their propagation GDoc](https://docs.google.com/document/d/1cxSeZw_DVsYY3Z2473vza2uVvx8LfiBPS1X6bnUgAlU/edit#heading=h.6i569rv4s1vo  ))

# Access rights and their propagation

The rights management system is at the heart of the platform, and is based on the following principles.

## Basics on access rights

The two main entities of concern for access rights are "groups" and "items".

Groups
: Groups are implemented through the `groups` db table. A group can be either a user, a group of users, or a set of groups. The user-group relationship is implemented through the `users.self_group_id` property. The relationship between groups, an acyclic graph, is implemented through `groups_groups` table, the hierarchy is cached through `groups_ancestors`.

Items
: Items are implemented through the `items` db table. A item represents some content (tasks, chapter, ...) and are structured (acyclic graph) via the `items_items` table, the ancestors hierarchy is cached in `items_ancestors`.

### "Given" access

Groups may be given access to items (`permissions` table) by a other user or by the platform as a reward for solving an exercise.

When they first arrive on the platform, users are automatically added to groups that have some access rights on some items. This is how they gain access to the public content when they arrive on the platform. Users may also be added automatically to groups based on other criteria, such as being a verified teacher. Finally, users may join groups voluntarily (using a code, by invitation or on request). The groups a user belongs to will determine what access rights he has.

### Access propagation

Access right of a group to an item may be propagated to the descendants of the groups, and (under some conditions) to the descendants of the items.

<span class="label label-green">Key-concept</span>
Access right propagation from a parent **group** to its descendants is **implicit**. So if a user is part of a group, he has the same access rights as the group. On the other hand, access right propagation from a parent **item** to its descendants is **explicit**, i.e., new entries in the `groups_items` table are created each time new access rights are given to propagate these rights to the children.



