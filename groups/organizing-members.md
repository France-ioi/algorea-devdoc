---
layout: page
title: Organizing members (draft)
nav_order: 80
parent: Groups & Related Entities
---

<span class="label label-yellow">WIP / Draft at a early stage</span> Results of discussions between Mathias and DLe on 4/12/19.

# Organizing Members in Groups

## Scenarios / Needs

1.
A school teacher has a class group with a set of students as members.
He wants to create two subgroups in this class to classify the students (so move them in this subgroups).
The subgroups are visible to the students and they should be able to have group discussions, etc.
Approval should remain.
Students should not have to re-accept an invitation.

2.
A school teacher has a class with a set of students as members.
He wants to be able to create a group of "weak students" which is hidden to students and give this group access to specific items.

3.
A school teacher has 3 classes whose he is manager of.
He wants to be able to create a super group with the students of his 3 classes so that he can give some common permissions to all his class.

4.
In the scenario 3, he wants to create a hidden group of weak students as in scenario 2.

## Options / Proposals

### Invisible Organizing groups

A user can create groups which is not visible to its members (the managers can see it) and move members from the parent group to this group. However, these groups:
- Can only have one single parent group
- Cannot invite users, only move them from the parent group
- Cannot require approvals, only using those from the parent group
- Cannot give permissions. Permission can still be given by the parent group on this one.

## "Can add to other group" approval

A new can_add_to_other_group approval which allows the teacher to organize their students as they want in their school groups.
