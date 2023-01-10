---
layout: page
title: Data Schema
nav_order: 10
parent: Groups & Related Entities
---

# Data Schema

Groups are implemented through the `groups` db table. A group can be either a user, a group of users, or a set of groups. The user-group relationship is implemented through the `users.self_group_id` property. The relationship between groups, an acyclic graph, is implemented through `groups_groups` table, the hierarchy is cached through `groups_ancestors`.

Both `groups_groups` and `groups_ancestors` tables have a `expires_at` field. If `expires_at` contains a date in the past, then the relation has expired. This is used for contests which are only available until a specific date.

If you only care about non-expired group relations, you may prefer to use the `groups_groups_active` and `groups_ancestors_active`. Those are SQL Views that represents the content of the respective `groups_groups` and `groups_ancestors` tables, but only with non-expired entries.

<span class="label label-yellow">TO FIX</span> Approvals are now in the membership table (`groups_groups`).

For permissions, our main interests are the following tables:
* `groups` contains the information about group and their required approvals.
* `groups_groups` is used to represent the group hierarchy between two regular groups, and between a group and a member (user or team), that we will also call "membership". Use `groups_groups_active` if you only need non-expired group relations.
* `group_approvals` contains approvals given by members to a group they belong to.
* `group_managers` represents the relationships between the managing users and groups. Managers are typically not (but could be) members of the group they manage. Managers may have multiple permissions on a same group.

<div style="max-width:90%;">{% include groups_relations.html %}</div>
