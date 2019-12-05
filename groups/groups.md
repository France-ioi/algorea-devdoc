---
layout: page
title: Groups & Related Entities
nav_order: 70
has_children: true
permalink: /groups/
---

# Groups

Groups are ways to organize users, mainly to be able to give them permissions.

## Hierarchy

Groups have, as members, other groups. Relations between groups form an acyclic graph. Users are groups themselves (which cannot contain any other groups).

Direct children of groups are called **members**. Members of members at any level of the hierarchy are called **descendants**. Permissions given to a group applies (implicitely) to all its descendants.

## Managers

Groups may have managers. Managers are **not** members (but can be) and have the right to permission specific actions on the group.
