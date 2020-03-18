---
layout: page
title: Managers
nav_order: 20
parent: Groups & Related Entities
---

# Group Managers

A manager is a group which have some given managing permissions on a group. A group may have any number of managers, they may be end users or groups with many descendant users.

## Management permissions

Within the same group, managers may have different permissions.

* Implicit to all managers:
  * can view the group name and basic information
  * can view the list of members, including members' personal info if they approved (cfr [approvals]({{ site.baseurl }}/groups/approvals/)).
* **can_manage**: *none*, *memberships*, or *memberships_and_group*
  * "**memberships**":
     * can invite members, accept/refuse requests, remove members
     * can set how people can join this group
     * can batch-create users in the group (require additional authorization on the user himself)
     * can add / remove an other group as subgroup of this one (needs *can_manage:memberships_and_group* on the subgroup for adding it as member)
  * "**memberships_and_group**": *memberships* +
     * can change the name, description and type of the group
     * can define what item users are redirected to when they join the group
     * can change the required approvals (cfr [approvals]({{ site.baseurl }}/groups/approvals/))
     * can delete the group
     * can add/remove managers to the group and give them any permissions
     * can add/remove the group as a subgroup of another group (needs *can_manage:memberships* on the parent fo adding it)
* **can_grant_group_access** (bool): can give members permissions on items (requires the giver to [be allowed to give this permission on the item]({{ site.baseurl }}/design/access-rights/items/))
* **can_watch_members** (bool): can watch members' submissions on items. Requires the watcher to [be allowed to watch this item]({{ site.baseurl }}/design/access-rights/items/). For members who have agreed (cfr [approvals]({{ site.baseurl }}/groups/approvals/)).
* **can_edit_personal_info** (bool): can change member's personal info, for those who have agreed (not visible to managers, only for specific uses) (cfr [approvals]({{ site.baseurl }}/groups/approvals/))

## Propagation of management permissions

A manager of group *G* is the manager of all descendant groups *Gi* of *G* with the same level of permissions. However, the manager can apply its *can_watch_members* and *can_edit_personal_info* permissions only if the members have agreed to. So if this group *G1* does not require members to approve that the managers can watch their activites, a manager of *G* with *can_watch_members=true* will not be able to watch the members of this groups (who are not in another group on which the manager would have this approval).
