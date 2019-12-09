---
layout: page
title: Unlocking
nav_order: 80
parent: Items & Related Entities
---

# Item Unlocking

Solving a item may "unlock" access to another item.

The unlocking rules are stored in the `item_unlocking_rules` and are made of an "unlocking item", an "unlocked item", and a score. One such rule means that if a score at least equal to the given score is obtained on the "unlocking item", access is granted to the "unlocked item". If there are several rules for the same "unlocked item", the item is unlocked as long as at least one rule is valid. There is no rule combination.

## Access granted

Unlocking an item consists in giving the user or team the "can_view:content" permission on it. The `origin` of this granted permission is "unlocking" and the `source_group_id` is the user or team.

## Modifying these rules

In order to modify such an unlocking rule, the user needs `can_grant_view >= content` and `can_edit: all` on the **unlocked** item.

When creating a rule (or decreasing the required score), the new rule applies immediately to already existing scores. However, if the rule is removed (or required score increased), the former unlocks stay. There should be a service for the user to "reset" these, this service clears all the unlocks of this item and recompute them.

## FAQ / Remarks

### Team unlocking

If a team unlocks the access to an item, the permission granted is done to the team itself, not its member (<> what was done on AlgoreaPlaform). As a consequence, a member leaving a team may lose access to this content, and a member joining the team after the unlock may gain access. It insures that all team members have the same access to content during a contest.
