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

A group is visible to a user if one of the following conditions is fulfilled:
- The group is an ancestor of a group that the user has joined (or the group itself)
- The group is an ancestor of a group of which the user is a manager (or the group itself)
- The group is public (`is_public` is true)
