---
layout: page
title: Groups & Related Entities
nav_order: 70
has_children: true
permalink: /groups/
---

# Groups

Groups are ways to organize users, for instance, to be able to give them permissions .

## Hierarchy

Groups have, as members, other groups. Relations between groups form an acyclic graph. Users are groups themselves (which cannot contain any other groups).

Direct children of groups are called **members**. Members of members at any level of the hierarchy are called **descendants**. Permissions given to a group applies (implicitely) to all its descendants.

## Managers

Groups may have managers. Managers are **not** members (but can be) and have the right to permission specific actions on the group.

## Types (work in progress)

Groups have a type. Each type has a few particularities.

### User

Groups of type 'User' have an associated entry in the "users" table. Users cannot have children groups.
Can participate to a contest.

### Team

Teams are aggregate of users, they can only have users as children.
Can participate to a contest.

### Contest Participants (`ContestParticipants`)

These are groups are used to automatically collect all the participants who entered of a contest (with an expiring membership). Children groups can only be teams and users.

### Session

A session is a set of participants who participate to a same activity, typically at a given time and place.

A session is always associated to an activity (chapter, task, or course) (`activity_id`), for instance the contest the user will participate to after joining the session. When there are members in the group, it should not be possible to change this item without warning.

The session can be joined as groups by request, by code or invitation. <span class="label label-yellow">Draft</span>  For a session to be searcheable, it needs to have groups.free_access and to have the associated item visible (can_view>list). To show a session on an item (`is_official_session` flag), the user neeeds a specific permission `can_make_session_official` on item.

A session can have a parent group, this group can be typically created (by the UI) at session creation. The session can have the `require_members_to_join_parent` flag set. In this case, the UI should force the user who wants to join the session to join both the session and the parent group.

A session has also some specific attributes (address, organizer, ...), see DB schema for more info.

### School

<span class="label label-yellow">Draft</span> To be added (?), not currently in db [28/01/2020])

Has an address.

### Class

Must be descendant of a school. <span class="label label-yellow">Draft</span>

### Club

### Friends

### Base

### Other


## Group visibility

A group `G` is visible to a user `U` if:

1) `G` is an ancestor of `U` or `G` is an ancestor of at least one team `U` is a member of or

2) `G` is an ancestor of a non-user group `G'` that `U` manages with `can_manage`>="memberships" or `can_watch_members`=true or `can_grant_group_access`=true (explicitly or implicitly) or

3) `G` is a user implicitly managed by `U` or `G` is a member of a team managed by `U` (explicitly or implicitly) with `can_manage`>="membership" or `can_watch_members`=true or `can_grant_group_access`=true or

4) `G` is public (`is_public` is true).

### Explanations

1) A user can see all the groups he is a member of as well as their ancestors (he is implicitly a member of them).

2) When a user is a manager of a group (with `can_manage`>="memberships" or `can_watch_members`=true or `can_grant_group_access`=true), we consider they may know about all groups that are ancestors of the group he manages directly (explicitly) or via descendants (implicitly). Why we do this:
  * When a user manages a group with `can_manage`>="memberships", he can himself into that group or any of its descendant (leading to the rule 1). This excludes ancestors of users the user manages as it's impossible to become a member of a user.
  * When a user manages a group with `can_watch_members`=true, we allow him to see ancestors of the group for a REASON TO BE DOCUMENTED. This excludes ancestors of users the user manages as it's impossible for a user to have members.
  * When a user manages a group with `can_grant_group_access`=true, we want the user to be able to see all the permissions given to the group via any of its ancestors, so all the ancestors of the managed group should be visible to the user. This excludes ancestors of users for a REASON TO BE DOCUMENTED.

3) A manager (implicit or explicit) of a group having `can_manage`>="membership" or `can_watch_members`=true or `can_grant_group_access`=true should be able to see the users who are members of that group. Why we do this:
  * When a user manages a group with `can_manage`>="membership", we allow them to see the members of that group for a REASON TO BE DOCUMENTED.
  * When a user manages a group with `can_watch_members`=true, he should be able to see the members of that group because that's the purpose of that permission.
  * When a user manages a group with `can_grant_group_access`=true, we want the user to be able to see all the members of that group for a REASON TO BE DOCUMENTED.

4) Public groups are by definition visible to everyone.
