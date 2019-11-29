---
layout: page
title: Access rights on groups
nav_order: 30
parent: Access Rights
has_toc: true
permalink: /design/access-rights/groups/
---

# Access rights on groups
{: .no_toc}

1. TOC
{:toc}

## Preambule on data schema

Groups are ways to organize users, mainly to be able to give them permissions. Groups are made of other groups, users are groups themselves (which cannot contain any other groups).

For permissions, our main interests are the following tables:
* `groups` contains the information about group and their required approvals.
* `groups_groups` is used to represent the group hierarchy between two regular groups, and between a group and a member (user or team), that we will also call "membership". All members in a group get the same access rights.
* `group_approvals` contains approvals given by members to a group they belong to.
* `group_managers` represents the relationships between the managing users and groups. Managers are typically not (but could be) members of the group they manage. Managers may have multiple permissions on a same group.

<div style="max-width:90%;">{% include groups_relations.html %}</div>

## Management permissions

Within the same group, managers may have different permissions.

* Implicit to all managers: can view the list of members, including members' personal info if they approved (cfr <a href="#approvals">"approvals" section</a>).
* **can_manage**: *none*, *memberships*, or *memberships_and_group*
  * "**memberships**":
     * can invite members, accept/refuse requests, remove members
     * can set how people can join this group
     * can batch-create users in the group (require additional authorization on the user himself)
     * can add / remove other groups as subgroup of this one
  * "**memberships_and_group**": *memberships* +
     * can change the name, description and type of the group
     * can define what item users are redirected to when they join the group
     * can change the required approvals (cfr <a href="#approvals">"approvals"</a>)
     * can delete the group
* **can_edit_personal_info** (bool): can change member's personal info, for those who have agreed (cfr <a href="#approvals">"approvals"</a>)
* **can_grant_group_access** (bool): can give all members the access rights to some items (requires the giver to <a href="{{ site.url }}{{ site.baseurl }}/design/access-rights/items/">be allowed to give this permission on the item</a>)
* **can_watch_members** (bool): can watch members' submissions on items. Requires the watcher to <a href="{{ site.url }}{{ site.baseurl }}/design/access-rights/items/">be allowed to watch this item</a>. For members who have agreed (cfr <a href="#approvals">"approvals"</a>).

<a name="approvals"></a>

## Member's approvals

Some privileges that the group wants to get on the members may require an approval from each member. These **must be approved for joining** the group or require post-approvals if set when there are already members in the group (see below). These privileges are the following:

* Let the managers view or edit their personal information
* Lock the user as a member until a date (cannot leave on his own or delete his account). To leave the group, he can do a leave request which has to be approved by a group manager.
* Let the managers watch all his submissions on the items they are allowed to watch

Expressed by the following attributes of the group:
* **require_personal_info_access_approval**: *no*, *view*, *edit*. (note that the "*edit*" value can only be set by the system for specific cases, not by any manager)
* **require_lock_membership_approval_until** (time)
* **require_watch_approval** (bool)

Approval of individual members is stored into the membership (`groups_groups` table) as the time of the approval:
* **personal_info_access_approved_at**
* **lock_membership_approved_at**
* **watch_approved_at**

## Changing required approval for a group with members

If a manager wants to change the approval required by the group while there are already members, he can choose to either enforce the rule now, so removing all members who have not agreed yet (so normally all), or set all members' membership expiration to a date in the near future. These members should receive a notification and accept all required approval of the group to cancel this expiration.

In any case, a manager can only use his permissions which requires user approval (view user personal info, watch users' submissions) when the member has approved it.

## Propagation of access rights

Access rights propagates implicitely through group hierarchy. In particular:
* Access granted to a group applies to all of its members and all members of its descendant groups
* Permission given to a manager on a group also applies to all descendant groups from this group.

## FAQ / Remarks

### Watching items and group members

For a user to watch the submissions of another user on an item, he needs both the "*can_watch*" permission of the item and "*can_watch_members*" on the user. This works even if the *can_watch* permission on the item is not related with the group.

### Permissions left when a manager leaves

The permissions granted by a manager to a group are kept by the group even when the manager leaves the management of the group. As a consequence, a manager can modify permissions he is not allowed to give himself, but only by decreasing them. Then,he would not able to restore to the permission to its initial level.

### Limits to management permission propagation

As a corollary of the other rules, if the manager has a permission which requires approval while a subgroup does not have this requirement, this permission does not apply for the manager on this subgroup.
