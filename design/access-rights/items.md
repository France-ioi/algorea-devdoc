---
layout: page
title: Access rights on items [WIP]
nav_order: 10
parent: Principles of access rights and their propagation
grand_parent: Design
---

<span class="label label-red">Work in progress - Not final</span>

# Access rights on items

## Preambule on data schema

Access rights to items given to groups are expressed in the `permissions_given` table.
The results of the explicit propagation of access rights through items is stored in the `permissions_computed` table.
Relations (parent-child) among items have a property `unlocked_access_propagation` which affects propagation and may have the following possible values: *none*, *as unlocked*, *as visible*.

{% include items_relations.html %}

## Overview of permissions of a group on an item

The following permissions may be given to a group for an item.

<div style="display: none;">
### Visibility levels (resulting permissions)

There are four levels of "view" of an item for a group:
* **none**
* **list**: can view the title, subtitle and description
* **content**: *list* + can load the content of the item and try to solve it
* **solutions**: *content* + can view the solutions and discussions about the item
</div>

### Permissions affecting visibility

There are four permissions (not especially matching the four resulting permissions!) which can be given to a group on an item. Technically, they are given as a time value from which the permission is applied ("since ..."), this detail is ommitted here.
* **visible**: view the title, subtitle and description, do not propagate to children
* **unlocked**: can load the content of the item and try to solve it, propagate to children in some conditions (see below in <a href="#technically">technically</a>)
* **full access**: same as unlocked except it propagates to children
* **solution access**: can view solutions and discussions, propagate to children (may change soon)

<div style="display: none;">

### Authorized Actions

* Distribution
* Reuse
* Observation
* Write
* Sessions (?)
* Ownership

</div>

### Meta permissions

Permissions of giving permissions

* **owner**: can view/do everything about this item and give all other permissions (upcoming); can delete the item

<div style="display: none;">

### Specific to some types

#### Time-limited items

Time-limited items (typically contests) have a specific "start" button which allow the user to start a participation on an item-specified duration.

* **can enter from** and **can enter until**: time frame during which the user can enter the item, i.e. is put into the *participation group* with an expiring membership.
</div>

## Parent-child Propagation

Permissions given on an item to a group may be propagated (explicitely) to the item's children, under some conditions depending on the permissions.

This propagation is relaunched each time a permission is changed or an item added/removed. It is computed on the changed node (the group-item relation node) based on its parents, and then propagated to its children.

Below, the "permissions given" correspond to the permissions which has been given directly, typically by another user. The "computed permissions" correspond to what the group has actually as permission, as a result of the permissions given and the propagation.


<div style="display: none;">
TODO: Handle deps between perm and declare if they are checked when set, when get, contrained in the db, ...

</div>



## Technically

### Permissions given

The `permissions_given` table express the raw permissions given to a group on an item.
One permission entry matches exactly one group, one item, and one owner (the group who has given this permission). There may be several permissions for a same *(group, item)*.

The attributes of this table are the following:

* **group_id**
* **item_id**
* **owner_group_id**
* **latest_update_on**

* **visible_from**
* **unlocked_from**
* **full_access_from**
* **solution_access_from**
<div style="display: none;">
* reuse_perm: t/f
* distribution_perm: no, yes, transfer
* distribution_propagated: t/f
* write_perm: no, yes, transfer
* write_propagated: t/f
* observation_perm: no, yes, transfer
* observation_propagated: t/f
* session_perm: t/f
</div>

* **is_owner**: t/f

### Computed permissions

The `permissions_computed` table represents the actual permissions that the group has, considering the given permissions and their propagations. This table could be completely rebuild from `permissions_given`.

The attribute of this table are the following:

* **group_id**
* **item_id**
* **actual_visible_from**: computed as the min of all `visible_from` of the *(group, item)* and all `actual_unlocked_from` of item's parents with `unlocked_access_propagation = 'as visible'`
* **actual_unlocked_from**: computed as the min of all `unlocked_from` of the *(group, item)* and all `actual_unlocked_from` of item's parents with `unlocked_access_propagation = 'as unlocked'`
* **actual_full_access_from**: computed as the min of all `full_access_from` of the *(group, item)* and all `actual_full_access_from` of item's parents
* **actual_solution_access_from**: computed as the min of all `solution_access_from` of the *(group, item)* and all `actual_solution_access_from` of item's parents

* **view_perm**: (virtual attribute, enum)
  * if `actual_solution_access_from` is in the past or `actual_is_owner` is true : **'solutions'**; otherwise:
  * if `actual_unlocked_from` or `actual_full_access_from` is in the past: **'content'**; otherwise:
  * if `actual_visible_from` is in the past: **'list'**; otherwise:
  * 'none'

<div style="display: none;">
* actual_reuse_perm: t/f
* actual_distribution_perm: no, yes, transfer
* actual_distribution_propagated: t/f
* actual_write_perm: no, yes, transfer
* actual_write_propagated: t/f
* actual_observation_perm: no, yes, transfer
* actual_observation_propagated: t/f
</div>
* **actual_is_owner**: bool - max of `is_owner` of the *(group, item)*
