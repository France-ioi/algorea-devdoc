---
layout: page
title: Access rights on items
nav_order: 10
parent: Principles of access rights and their propagation
grand_parent: Design
has_toc: true
---

# Access rights on items
{: .no_toc}

1. TOC
{:toc}

## Preambule on data schema

Access rights to items given to groups are expressed in the `permissions_granted` table.
The results of the explicit propagation of access rights through items is stored in the `permissions_generated` table.
Relations (parent-child) among items have several properties which affect propagation.

<div style="max-width:90%;">{% include items_relations.html %}</div>

## Overview of permissions of a group on an item

### can_view

The level of visibily the group has on the item:

* **no**: not listed and trying to get it returns not-found or forbidden
* **info**: can view basic database info on the item: title, description, type
* **content**: can view "info" and content stored by task platforms; for tasks, can try to solve
* **content_with_descendants**: can view "content" and more of the descendant items (see propagation)
* **solution**: can view "content", solutions and discussions


### can_grant_view

The level of visibility that the group can give on this item to other groups on which it has the right to (cfr group permissions).

* **no**: cannot grant view access
* **content**: can give up to *can_view* "content"
* **content_with_descendants**: can give up to *can_view* "content_with_descendants"
* **solution**: can give up to *can_view* "solution"
* **transfer**: can give up to *can_view* "solution" and grant any *can_grant_view* access to another group


### can_watch

The level of observation a group has for an item, on the activity of the users he can watch (cfr group permissions).

* **no**
* **result**: can view meta data about the submissions, including the scores
* **answer**: can watch "result" and can look at the detail of the answers
* **transfer**: can watch "answer" and grant any *can_watch* access to another group

### can_edit

The level of edition permissions a group has on an item.

* **no**
* **children**: can attach new child items to the item, and edit propagation rules between the item and some of the children (cfr propagation permissions) [still to be defined: what about relation deletion?]
* **all**: can edit "children" and can make changes to the parameters, title and content and solution of the item; cannot delete the item.
* **transfer**: can edit "all" and grant any *can_edit* access to another group

### is_owner

Whether (true/false) the group is the owner of this item. Implies the maximum level in all of the above permissions. Can delete the item.

<a name="aggregation"></a>

## Aggregation of permissions from multiple sources

We call *aggregation* the merge of the multiple granted permissions (from possibily different groups) to one permission tuple (indexed by group, item).

For each permission attribute (can_view, can_grant_view, can_watch, can_edit, and is_owner), we take its maximum level among all values. In addition, "is_owner=true" makes every other attribute get its maximum possible value.

<a name="propagation"></a>

## Propagation across parent-child items

Permissions given on an item to a group may be propagated (explicitely) to the item's children, under some conditions and depending on the permissions. This propagation computation is relaunched each time a permission is changed or an item added/removed. It is computed on the changed node (the group-item relation node) based on its parents, and then propagated to its children. When there are several parents, the higher permission among parents is kept.

The copy of the permission levels from the parent to its children is either equivalent or lower (never increase), which mainly depends on the following attributes on the item-item relationship:

* **content_view_propagation**: *none*, *as_info*, *as_content* -- defines how a *can_view="content"* permission propagates: not at all, as "info" or as "content"
* **descendants_and_solution_view_propagation**: *none*, *descendants*, *descendants_and_solution* -- defines which of the "descendants" and "solution" aspects from *can_view* propagate, so propagated to:
   * *none*: the value given in *content_view_propagation* at best
   * *descendants*: as *content_with_descendants* at best
   * *descendants_and_solution*: the same value
* **grant_view_propagation**: false, true -- Whether *can_grant_view* propagates (as the same level with, as upper limit, "solution")
* **watch_propagation**: false, true -- Whether *can_watch* propagates (as the same level with, as upper limit, "answer")
* **edit_propagation**: false, true -- Whether *can_edit* propagates (as the same level with, as upper limit, "all")

In addition, the following levels **never** propagate:
* *can_view*="info" (so propagates as "none")
* *can_grant_view*, *can_watch*, *can_edit*="transfer" (so propagate as the preceding level)
* *is_owner*=true (so propagates as "false")

## Changing and propagating permissions

The following tables defines which permissions are required to be able to change the permission of another groups, and summarize how these permissions propagates.

| "can_view" perm granted | Constraint on "giver" | Constraint on "receiver" | Propagation rule     |
|:------------------------|:----------------------|:-------------------------|:---------------------|
| info                    | can_grant_view ≥ content        | none |  Never propagates              |
| content                 | can_grant_view ≥ content        | none | Apply content_view_propagation |
| content_with_descendants| can_grant_view ≥ content_with_d | none | Only if descendants_and_solution_view_propagation ≥ "descendants".; otherwise apply content_view_propagat. |
| solution                | can_grant_view ≥ solution       | none | Only if descendants_and_solution_view_propagation ≥ "descendants_and_solution; otherwise apply content_view_propagat. |


| "can_grant_view" perm granted | Constraint on "giver" | Constraint on "receiver"        | Propagation condition     |
|:------------------------------|:----------------------|:--------------------------------|:---------------------|
| content                 | can_grant_view = transfer   | can_view ≥ content              | Only if grant_view_propagation |
| content_with_descendants| can_grant_view = transfer   | can_view ≥ content_with_d       | Only if grant_view_propagation |
| solution                | can_grant_view = transfer   | can_view ≥ solution             | Only if grant_view_propagation |
| transfer                | is_owner                    | can_view ≥ solution             | Never propagates |


| "can_watch" perm granted | Constraint on "giver" | Constraint on "receiver"        | Propagation condition     |
|:-------------------------|:----------------------|:--------------------------------|:---------------------|
| result                   | can_watch = transfer   | can_view ≥ content             | Only if watch_propagation |
| answer                   | can_watch = transfer   | can_view ≥ content             | Only if watch_propagation |
| transfer                 | is_owner               | can_view ≥ content            | Never propagates |


| "can_edit" perm granted | Constraint on "giver" | Constraint on "receiver"        | Propagation condition     |
|:-------------------------|:----------------------|:--------------------------------|:---------------------|
| children                 | can_edit = transfer   | can_view ≥ content             | Only if edit_propagation |
| all                      | can_edit = transfer   | can_view ≥ content             | Only if edit_propagation |
| transfer                 | is_owner              | can_view ≥ content             | Never propagates |

| perm granted  | Constraint on "giver" | Constraint on "receiver"        | Propagation condition     |
|:--------------|:----------------------|:--------------------------------|:------------------|
| is_owner=true | is_owner              | none                            | Never propagates          |

## Creating (defaults) and changing propagation rules

Any user can create relationship between two items at the condition he has at least *can_edit ≥ children* on the parent item and *can_view > none* access to the child. This way, he can build a custom structured chapter with the content he want. However, adding content to item he owns does not give him more access to this content and does not necessarily give him any rights to distribute this content. At creation, by default, the propagation will be set to the maximum values the groups can set (see below), but *content_view_propagation* which is set by default to maximum "as_info".

For changing propagation rules (on the item-item relationship), the giver group also needs *can_edit ≥ children* access on the parent item and the following permissions on the child item. To decrease the propagation level (whatever the level), you do not need any specific permissions on the child item.
To increase it, you need:

| Propagation rule         | Increased to value | Permission needed on the child item  |
|:-------------------------|:-------------------|:--------------------------|
| content_view_propagation | *                | can_grant_view ≥ content  |
| descendants_and_solution_view_propagation   | content_with_descendants | can_grant_view ≥ content_with_descendants  |
| descendants_and_solution_view_propagation   | solution           | can_grant_view ≥ solution  |
| grant_view_propagation   | true               | can_grant_view ≥ transfer  |
| watch_propagation        | true               | can_watch ≥ transfer       |
| edit_propagation         | true               | can_edit ≥ transfer        |


## Database details

### Granted permissions table (permissions_granted)

The `permissions_granted` table express the raw permissions given to a group on an item.
One permission entry matches exactly one group, one item, and one owner. The owner is the group who has given this permission, all permissions given by users are given as a specific group which has the required permissions to do so and which will be able to revoke or modify this permission. There may be several permissions for a same *(group, item)*.

The attributes of this table are the following:
* group_id, item_id, owner_group_id [PK]
* latest_update_on
* can_view, can_grant_view, can_watch, can_edit, is_owner

### Generated permissions table (permissions_generated)

The `permissions_generated` table represents the actual permissions that the group has, considering the <a href="#aggregation">aggregation</a> and the <a href="#propagation">propagation</a>. This table could be completely rebuild from `permissions_granted`.

The attribute of this table are the following:
* group_id, item_id [PK]
* can_view_generated, can_grant_view_generated, can_watch_generated, can_edit_generated, is_owner_generated
