---
layout: page
title: Approvals
nav_order: 30
parent: Groups & Related Entities
permalink: /groups/approvals/
---

# Member's approvals

Some privileges that the group managers want to have on the members may require an approval from them. These **must be approved for joining** the group or require post-approvals if set when there are already members in the group (see below). These privileges are the following:

* Let the managers view or edit their personal information
* Lock the user as a member until a date (the member cannot leaves on his own or delete his own account). To leave the group, he can do a leave request which has to be approved by a group manager.
* Let the managers watch all his submissions on the items they are allowed to watch

Expressed by the following attributes on the group:
* **require_personal_info_access_approval**: *none*, *view*, *edit*. (note that the "*edit*" value can only be set by the system for specific cases, not by any manager)
* **require_lock_membership_approval_until** (time)
* **require_watch_approval** (bool)

Approval of individual members is stored into the membership (`groups_groups` table) as the time of the approval:
* **personal_info_access_approved_at**
* **lock_membership_approved_at**
* **watch_approved_at**

## Scope

The "view/edit personal info" and "watch" approvals are directly related to the group but have more global scope. This means that a manager who is allowed to "view/edit personal info" or "watch" a member through one of his group, can do it through all the platform.

## Changing required approval for a group with members

If a manager wants to change the approval required by the group while there are already members, he can choose to either enforce the rule now, so removing all members who have not agreed yet (so normally all), or set all members' membership expiration to a date in the near future. These members should receive a notification and accept all required approval of the group to cancel this expiration.

In any case, a manager can only use his permissions which requires user approval (view user personal info, watch users' submissions) when the member has approved it.
