---
layout: page
title: Item Permissions
nav_order: 10
parent: Permissions
has_toc: true
permalink: /design/access-rights/items/
---

# Item Permissions
{: .no_toc}

1. TOC
{:toc}

## Preambule on data schema

Permissions to items given to groups are expressed in the `permissions_granted` table.
The results of the explicit propagation of permissions through items is stored in the `permissions_generated` table.
Relations (parent-child) among items have several properties which affect propagation.

<div style="max-width:90%;">{% include items_relations.html %}</div>

## Overview of permissions of a group on an item

### can_view

The level of visibily the group has on the item:

* **none**: not listed and trying to get it returns not-found or forbidden
* **info**: can view basic database info on the item: title, description, type
* **content**: can view "info" and content stored by task platforms; for tasks, can try to solve
* **content_with_descendants**: can view "content" and more of the descendant items (see propagation)
* **solution**: can view "content", solutions and discussions

### can_enter_*

**can_enter_from** and **can_enter_until** are time from and until which the group can "enter" the item, i.e. can create an attempt under some conditions.

### can_grant_view

The level of visibility that the group can give on this item to other groups on which it has the right to (cfr group permissions).

* **none**: cannot grant view permission
* **enter**: can give *can_enter_\** permission and *can_view* "info"
* **content**: can give up to *can_view* "content"
* **content_with_descendants**: can give up to *can_view* "content_with_descendants"
* **solution**: can give up to *can_view* "solution"
* **transfer**: can give up to *can_view* "solution" and grant any *can_grant_view* permission to another group


### can_watch

The level of observation a group has for an item, on the activity of the users he can watch (cfr group permissions).

* **none**
* **result**: can view meta data about the submissions, including the scores
* **answer**: can watch "result" and can look at the detail of the answers
* **transfer**: can watch "answer" and grant any *can_watch* permission to another group

### can_edit

The level of edition permissions a group has on an item.

* **none**
* **children**: can attach new child items to the item, and edit propagation rules between the item and some of the children (cfr propagation permissions) [still to be defined: what about relation deletion?]
* **all**: can edit "children" and can make changes to the parameters, title and content and solution of the item; cannot delete the item.
* **transfer**: can edit "all" and grant any *can_edit* permission to another group

### can_make_session_official

Whether (true/false) the group is allowed to associate official sessions to this item.

### is_owner

Whether (true/false) the group is the owner of this item. Implies the maximum level in all of the above permissions. Can delete the item.

<a name="aggregation"></a>

## Aggregation of permissions from multiple sources

We call *aggregation* the merge of the multiple granted permissions (from possibily different groups) to one permission tuple (indexed by group, item).

For each permission attribute but *can_make_session_official* and *can_enter_\** (so *can_view*, *can_grant_view*, *can_watch*, *can_edit*, and *is_owner*), we take its maximum level among all values. In addition, "is_owner=true" makes every other attribute get its maximum possible value.

<a name="propagation"></a>

## Propagation across parent-child items

Permissions given on an item to a group may be propagated (explicitely) to the item's children, under some conditions and depending on the permissions. This propagation computation is relaunched each time a permission is changed or an item added/removed. It is computed on the changed node (the group-item relation node) based on its parents, and then propagated to its children. When there are several parents, the higher permission among parents is kept.

The propagation of a permission from a parent to its children is either to a same level or a lower one (never increase), and depends on the following attributes on the item-item relationship:

* **content_view_propagation**: *none*, *as_info*, *as_content* -- defines how *can_view="content"* permissions propagate: not at all, as "info" or as "content"
* **upper_view_levels_propagation**: *use_content_view_propagation*, *as_content_with_descendants*, *as_is* -- defines how *can_view="content_with_descendants"<span>|</span>"solution"* permissions propagate, so propagated to:
   * *use_content_view_propagation*: propagate as the value given in *content_view_propagation*
   * *as_content_with_descendants*: as *content_with_descendants*
   * *as_is*: propagate as the same value
* **grant_view_propagation**: false, true -- Whether *can_grant_view* propagates (as the same level with, as upper limit, "solution")
* **watch_propagation**: false, true -- Whether *can_watch* propagates (as the same level with, as upper limit, "answer")
* **edit_propagation**: false, true -- Whether *can_edit* propagates (as the same level with, as upper limit, "all")

In addition, the following levels **never** propagate:
* *can_view*="info" (so propagates as "none")
* *can_grant_view*, *can_watch*, *can_edit*="transfer" (so propagate as the preceding level)
* *is_owner*=true (so propagates as "false")

As *can_make_session_official* and *can_enter_\** do not aggregate, they do not propagate as well.

## Changing and propagating permissions

The following tables defines which permissions are required to be able to change the permission of another groups, and summarize how these permissions propagates.

| "can_view" perm granted | Constraint on "giver" | Constraint on "receiver" | Propagation rule     |
|:------------------------|:----------------------|:-------------------------|:---------------------|
| info                    | can_grant_view ≥ content        | none |  Never propagates              |
| content                 | can_grant_view ≥ content        | none | Apply content_view_propagation |
| content_with_descendants| can_grant_view ≥ content_with_d | none | Only if upper_view_levels_propagation ≥ "as_content_with_descendants"; otherwise apply content_view_propagation |
| solution                | can_grant_view ≥ solution       | none | As "solution" if upper_view_levels_propagation ≥ "as_is", as "content_with_descendants" if upper_view_levels_propagation = "as_content_with_descendants", otherwise apply content_view_propagation |


| "can_grant_view" perm granted | Constraint on "giver" | Constraint on "receiver"        | Propagation condition     |
|:------------------------------|:----------------------|:--------------------------------|:---------------------|
| enter                   | can_grant_view = transfer   | can_view ≥ info                 | Only if grant_view_propagation |
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

| perm granted                    | Constraint on "giver" | Constraint on "receiver"        | Propagation condition |
|:--------------------------------|:----------------------|:--------------------------------|:----------------------|
| *can_make_session_official=true | is_owner              | can_view ≥ info                 | N/A                   |

| perm granted  | Constraint on "giver" | Constraint on "receiver"        | Propagation condition     |
|:--------------|:----------------------|:--------------------------------|:------------------|
| is_owner=true | is_owner              | none                            | Never propagates          |

## Creating (defaults) and changing propagation rules

Any user can create relationship between two items at the condition he has at least *can_edit ≥ children* on the parent item and *can_view > none* permission to the child. This way, he can build a custom structured chapter with the content he want. However, adding content to item he owns does not give him more permission to this content and does not necessarily give him any rights to distribute this content. At creation, by default, the propagation will be set to the maximum values the groups can set (see below), but *content_view_propagation* which is set by default to maximum "as_info".

For changing propagation rules (on the item-item relationship), the giver group needs *can_edit ≥ children* permission on the parent item and the following permissions on the child item to increase the level of propagation:

| Propagation level increase                                   | Permission needed on the child item        |
|:-------------------------------------------------------------|:-------------------------------------------|
| content_view_propagation to anything                         | can_grant_view ≥ content                   |
| upper_view_levels_propagation to as_content_with_descendants | can_grant_view ≥ content_with_descendants  |
| upper_view_levels_propagation to as_is                       | can_grant_view ≥ solution                  |
| grant_view_propagation to true                               | can_grant_view ≥ transfer                  |
| watch_propagation to true                                    | can_watch ≥ transfer                       |
| edit_propagation to true                                     | can_edit ≥ transfer                        |

To decrease the propagation level (whatever the level), you do not need any specific permissions on the child item.

## Database details

### Granted permissions table (permissions_granted)

The `permissions_granted` table express the raw permissions given to a group on an item.
One permission entry matches exactly one group, one item, one *source group* and one *origin*. The *source group* is the group from which the destination group has received the permission. Only the managers of this *source group* are able to revoke or modify this permission. The *origin* is the process which has allocated this permission (group permission, unlocking, ...). There may be several permissions for a same *(group, item)*, e.g., multiple permissions to the same task given to an end-user by several groups he is member of.

The attributes of this table are the following:
* group_id, item_id, source_group_id, origin [PK]
* latest_update_on
* can_view, can_enter, can_grant_view, can_watch, can_edit, can_make_session_official, is_owner

### Generated permissions table (permissions_generated)

The `permissions_generated` table represents the actual permissions that the group has, considering the <a href="#aggregation">aggregation</a> and the <a href="#propagation">propagation</a>. This table could be completely rebuild from `permissions_granted`.

The attribute of this table are the following:
* group_id, item_id [PK]
* can_view_generated, can_grant_view_generated, can_watch_generated, can_edit_generated, is_owner_generated

## FAQ / Remarks

### "is_owner" aggregation and propagation
As a consequence of these rules, when *is_owner* is set to *true* only the *is_owner* attribute is modified in the
`permissions_granted` table. It is aggregation (so into `permissions_generated` table) which make *is_owner* set all "can_*" to their maximum value. For propagation, while "is_owner" is not propagated, the levels it involves in all other permissions are propagated as they had been granted directly (so "can_edit:transfer" propagates to "can_edit:all").
