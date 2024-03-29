---
layout: page
title: Permissions on Items
nav_order: 15
parent: Items & Related Entities
has_toc: true
permalink: /design/access-rights/items/
---

# Item Permissions
{: .no_toc}

1. TOC
{:toc}

## Preamble

Permissions a `Group` has on an `Item` are the result of [**Aggregation**](#aggregation-of-permissions-from-multiple-sources) and [**Propagation**](#aggregation-of-permissions-from-multiple-sources).

Permissions to items given to groups are expressed in the `permissions_granted` table.
Permissions given to a group are implicitly given to its children.
This process is called **Aggregation**.

The result of **Aggregation** is computed on the fly in SQL queries.

The permissions given for an item may transfer to its children if it is explicitly defined in the parent-child relation.
Multiple properties exist to define which permissions transfer or not.
This process is called **Propagation**.

The result of **Propagation** is stored into the `permissions_generated` table.

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
* **solution_with_grant**: can give up to *can_view* "solution" and grant any *can_grant_view* permission to another group


### can_watch

The level of observation a group has for an item, on the activity of the users he can watch (cfr group permissions).

* **none**
* **result**: can view meta data about the submissions, including the scores
* **answer**: can watch "result" and can look at the detail of the answers
* **answer_with_grant**: can watch "answer" and grant any *can_watch* permission to another group

### can_edit

The level of edition permissions a group has on an item.

* **none**
* **children**: can attach new child items to the item, and edit propagation rules between the item and some of the children (cfr propagation permissions) [still to be defined: what about relation deletion?]
* **all**: can edit "children" and can make changes to the parameters, title and content and solution of the item; cannot delete the item.
* **all_with_grant**: can edit "all" and grant any *can_edit* permission to another group

### can_make_session_official

Whether (true/false) the group is allowed to associate official sessions to this item.

### is_owner

Whether (true/false) the group is the owner of this item. Implies the maximum level in all of the above permissions. Can delete the item.

<a name="aggregation"></a>

## Aggregation of permissions from multiple sources

We call **aggregation** the merge of all permissions a *group* has on an *item*, granted either:
- Directly to the group
- To an ancestor of the group

This notion is generalized to a *user*, with permissions granted either:
- Directly to the user
- To any group he is a member of
- To any ancestor of a group he is a member of

From those granted permissions, here's how the permissions are **aggregated**:
- `can_view`: we take the maximum among all values
- `can_grant_view`: we take the maximum among all values
- `can_watch`: we take the maximum among all values
- `can_edit`: we take the maximum among all values
- `is_owner`: if *true* for any granted permission, then *true*. Note that when *true* it makes all above permissions get its maximum possible value.
- `can_enter_from`:
  * `NOW()` if it is currently open for any granted permission (if current time is between `can_enter_from` and `can_enter_until`)
  * otherwise, the sooner date `can_enter_from` of any granted permission located in the future
  * otherwise, `infinite / never` (year 9999)
- `can_make_session_official`: if *true* for any granted permission, then *true*
- `can_request_help_to`: is a *group id* and is not aggregated

Note: those granted permissions that are aggregated may have been given from multiple `source groups`,
and come from different `origin`.
See details in the [section on database table permissions_granted](#granted-permissions-table-permissions_granted).

### UI for granting permissions

This image shows the result of **Aggregation** for the `group_membership` origin,
for the permission `can_grant_view`, given to a `Group` on an `Item`.
It is viewed from the point of view of a specific `source_group` (the group from which the permission is granted).
- **Nothing** is the permission granted directly to `Group` by the specific `source_group`.
- **Solution** is the **aggregated** permission `Group` has, coming from the permissions given to any of its ancestors, and by other `source_group`.
- Note that starting at the third level of permission, **Content**, the text is gray: it means the current-user doesn't have the right to give permissions from this point.

![UI for granting permissions]({{ site.url }}{{ site.baseurl }}/assets/screen_permissions_view.png)

<a name="propagation"></a>

## Propagation across parent-child items

Permissions given on an item to a group may be propagated explicitly to the item's children, under some conditions and depending on the permissions. This propagation computation is relaunched each time a permission is changed or an item added/removed. It is computed on the changed node (the group-item relation node) based on its parents, and then propagated to its children. When there are several parents, the higher permission among parents is kept.

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
* *can_view*="info" (propagates as "none")
* *can_grant_view*="solution_with_grant", *can_watch*="answer_with_grant", *can_edit*="all_with_grant" (propagate as the preceding level)
* *is_owner*=true (propagates as "false")

As *can_make_session_official* and *can_enter_\** do not aggregate, they do not propagate as well.

## Changing and propagating permissions

The following tables defines which permissions are required to be able to change the permission of another groups, and summarize how these permissions propagates.

Note that a group manager which can change group permissions (`can_grant_group_access`) can always decrease a permission to any lower level.

| "can_view" perm granted | Constraint on "giver" | Constraint on "receiver" | Propagation rule     |
|:------------------------|:----------------------|:-------------------------|:---------------------|
| info                    | can_grant_view ≥ enter          | none |  Never propagates              |
| content                 | can_grant_view ≥ content        | none | Apply content_view_propagation |
| content_with_descendants| can_grant_view ≥ content_with_d | none | Only if upper_view_levels_propagation ≥ "as_content_with_descendants"; otherwise apply content_view_propagation |
| solution                | can_grant_view ≥ solution       | none | As "solution" if upper_view_levels_propagation ≥ "as_is", as "content_with_descendants" if upper_view_levels_propagation = "as_content_with_descendants", otherwise apply content_view_propagation |


| "can_grant_view" perm granted | Constraint on "giver"                | Constraint on "receiver"        | Propagation condition     |
|:------------------------------|:-------------------------------------|:--------------------------------|:---------------------|
| enter                         | can_grant_view = solution_with_grant | can_view ≥ info                 | Only if grant_view_propagation |
| content                       | can_grant_view = solution_with_grant | can_view ≥ content              | Only if grant_view_propagation |
| content_with_descendants      | can_grant_view = solution_with_grant | can_view ≥ content_with_d       | Only if grant_view_propagation |
| solution                      | can_grant_view = solution_with_grant | can_view ≥ solution             | Only if grant_view_propagation |
| solution_with_grant           | is_owner                             | can_view ≥ solution             | Never propagates |


| "can_watch" perm granted | Constraint on "giver"         | Constraint on "receiver" | Propagation condition     |
|:-------------------------|:------------------------------|:-------------------------|:---------------------|
| result                   | can_watch = answer_with_grant | can_view ≥ content       | Only if watch_propagation |
| answer                   | can_watch = answer_with_grant | can_view ≥ content       | Only if watch_propagation |
| answer_with_grant        | is_owner                      | can_view ≥ content       | Never propagates |


| "can_edit" perm granted  | Constraint on "giver"     | Constraint on "receiver" | Propagation condition     |
|:-------------------------|:--------------------------|:-------------------------|:---------------------|
| children                 | can_edit = all_with_grant | can_view ≥ content       | Only if edit_propagation |
| all                      | can_edit = all_with_grant | can_view ≥ content       | Only if edit_propagation |
| all_with_grant           | is_owner                  | can_view ≥ content       | Never propagates |

| perm granted                    | Constraint on "giver" | Constraint on "receiver"        | Propagation condition |
|:--------------------------------|:----------------------|:--------------------------------|:----------------------|
| *can_make_session_official=true | is_owner              | can_view ≥ info                 | N/A                   |

| perm granted  | Constraint on "giver" | Constraint on "receiver"        | Propagation condition     |
|:--------------|:----------------------|:--------------------------------|:------------------|
| is_owner=true | is_owner              | none                            | Never propagates          |

## Creating (defaults) and changing propagation rules

Any user can create relationship between two items at the condition he has at least *can_edit ≥ children* on the parent item and *can_view > none* permission to the child. This way, he can build a custom structured chapter with the content he want. However, adding content to item he owns does not give him more permission to this content and does not necessarily give him any rights to distribute this content. At creation, by default, the propagation will be set to the maximum values the groups can set (see below), but *content_view_propagation* which is set by default "as_info" or "none" if the user cannot grant view.

For changing propagation rules (on the item-item relationship), the giver group needs *can_edit ≥ children* permission on the parent item and the following permissions on the child item to increase the level of propagation:

| Propagation level increase                                   | Permission needed on the child item       |
|:-------------------------------------------------------------|:------------------------------------------|
| content_view_propagation to as_info                          | can_grant_view ≥ enter                    |
| content_view_propagation to as_content                       | can_grant_view ≥ content                  |
| upper_view_levels_propagation to as_content_with_descendants | can_grant_view ≥ content_with_descendants |
| upper_view_levels_propagation to as_is                       | can_grant_view ≥ solution                 |
| grant_view_propagation to true                               | can_grant_view ≥ solution_with_grant      |
| watch_propagation to true                                    | can_watch ≥ answer_with_grant             |
| edit_propagation to true                                     | can_edit ≥ all_with_grant                 |
| request_help_propagation to true                             | can_grant_view ≥ content                  |

To decrease the propagation level (whatever the level), you do not need any specific permissions on the child item.

## Database details

### Granted permissions table (permissions_granted)

The `permissions_granted` table express the raw permissions given to a group on an item.
One permission entry matches exactly one `group`, one `item`, one `source_group` and one `origin`.

The `source_group` is the group from which the destination group has received the permission.
Only the managers of this `source_group` are able to revoke or modify this permission.

The `origin` is the process which has allocated this permission (group permission, unlocking, ...).
There may be several permissions for a same *(`group`, `item`)*, e.g.,
multiple permissions to the same task given to an end-user by several groups he is a member of.

The attributes of this table are the following:
* `group_id`, `item_id`, `source_group_id`, `origin` [PK]
* `latest_update_on`
* `can_view`, `can_enter`, `can_grant_view`, `can_watch`, `can_edit`, `can_make_session_official`, `is_owner`

### Generated permissions table (permissions_generated)

The `permissions_generated` table represents the actual permissions that the group has, considering the <a href="#aggregation">aggregation</a> and the <a href="#propagation">propagation</a>. This table could be completely rebuild from `permissions_granted`.

The attribute of this table are the following:
* `group_id`, `item_id` [PK]
* `can_view_generated`, `can_grant_view_generated`, `can_watch_generated`, `can_edit_generated`, `is_owner_generated`

## FAQ / Remarks

### "is_owner" aggregation and propagation
As a consequence of these rules, when `is_owner` is set to *true* only the `is_owner` attribute is modified in the
`permissions_granted` table. It is the aggregation (so into `permissions_generated` table) which make `is_owner` set all `can_*` to their maximum value.
For propagation, while `is_owner` is not propagated, the levels it involves in all other permissions are propagated as they had been granted directly (so "`can_edit`:all_with_grant" propagates to "`can_edit`:all").

### couldn't we remove "is_owner_generated" from "permissions_generated" and retrieve it from "permissions_granted" since its value is not propagated?
`permissions_granted` may have more than one row for a `(group_id, item_id)` couple,
it's primary key also includes `source_group_id` and `origin`.
This means that if we wanted to retrieve the `is_owner` for a specific `(group_id, item_id)`,
we would have to aggregate all corresponding rows and verify if at least one of them have `is_owner=1`.

For simplicity and performance, we keep `is_owner_generated` in `permissions_generated` (20/06/2023 - see [related issue](https://github.com/France-ioi/AlgoreaBackend/issues/966) ).
