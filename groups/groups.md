---
layout: page
title: Groups & Related Entities
nav_order: 70
has_children: true
permalink: /groups/
---

# Groups

Groups are implemented through the `groups` db table. A group can be either a user, a group of users, or a set of groups. The user-group relationship is implemented through the `users.self_group_id` property. The relationship between groups, an acyclic graph, is implemented through `groups_groups` table, the hierarchy is cached through `groups_ancestors`.
