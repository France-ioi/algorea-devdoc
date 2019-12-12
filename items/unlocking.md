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

## Possible future evolutions

### Granted permission bypassing

The fact that you just need some permissions on the destination item to create an unlock rule means you could set that everybody unlocking the a public common item unlock your content, which means that you end up giving an `can_view:content` to everybody while you would not be allowed to do so directly. Even if you need a path to this item to see it, it might be used to abuse users.

Random thoughts about this problem:
- If you can see a content (public or shared with one of your group), anyone with `can_edit:children` on it can already add any item to it without your consent.
- Just saying that every unlocking rule is in the contest of a group does not really work: You could share a full chapter with a group. If there is, within this chapter, one item unlocking another one, you do not know in advance the group that the full chapter will be shared with and you cannot given the manager of the group the right to edit the unlocking rule for this specific item of the chapter. You want to shit the chapter as a whole.
- We may enforce restriction on the source item as well. As anybody with `can_edit:children` can put items directly under the source item anyway, we can consider that having this perm is not more powerful as unlock, so requiring this perm on the source.
- We may restrict the definition on unlocking rules only between siblings, which seems to be the main usage of it.
- We may only unlock destination item from which we already see one of the parents, but it seems to be quite complicated to implement while getting `can_view:content` on an item could unlock children of the items.
