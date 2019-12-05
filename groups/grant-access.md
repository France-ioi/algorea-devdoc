---
layout: page
title: Granting Access to Members
nav_order: 40
parent: Groups & Related Entities
---

# Granting Access to Group Members

Only a manager who has the *can_grant_group_access* permission on a group of one its ancestor can grant the group members an access right to an item. Managers are also only allowed to grant specific access rights depending on their own access rights, as described on the ["item access rights" page]({{ site.baseurl }}/design/access-rights/items/).

An access right can be given to the whole group or to a direct member (user or larger group) of the group. An access right is always given **as** a group (the *permissions_granted.giver_group_id* attribute), this **giver group** must be the group of the member. So if group G1 is an ancestor of group G2 and M is a manager of G1, M can grant access to a member of G2 but only with G2 as giver group.

## FAQ / Remarks

### Remaining permissions when a manager leaves

The permissions granted by a manager to a group are kept by the group even when the manager leaves the management of the group. As a consequence, a manager can modify permissions he is not allowed to give himself, but only by decreasing them. Note that in this case, he would not able to restore the permission to its initial level.

