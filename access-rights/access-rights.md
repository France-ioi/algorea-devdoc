---
layout: page
title: Access Rights
nav_order: 20
has_children: true
---

(from [Principles of access rights and their propagation GDoc](https://docs.google.com/document/d/1cxSeZw_DVsYY3Z2473vza2uVvx8LfiBPS1X6bnUgAlU/edit#heading=h.6i569rv4s1vo  ))

# Access rights and their propagation

The rights management system is at the heart of the platform, and is based on the following principles.

## Basics on access rights

The two main entities of concern for access rights are "[groups]({{ site.baseurl }}/groups/)" and "[items]({{ site.baseurl }}/items/)". **Groups** are being granted access rights on **items**.

### "Given" access

Groups may be given access to items (`permissions_granted` table) by a group manager or by the platform as a reward for solving an exercise.

When they first arrive on the platform, users are automatically added to groups that have some access rights on some items. This is how they gain access to the public content when they arrive on the platform. Users may also be added automatically to groups based on other criteria, such as being a verified teacher. Finally, users may join groups voluntarily (using a code, by invitation or on request). The groups a user belongs to will determine what access rights he has.

### Access propagation

Access right of a group to an item may be propagated to the descendants of the groups, and (under some conditions) to the descendants of the items.

<span class="label label-green">Key-concept</span>
Access right propagation from a parent **group** to its descendants is **implicit**. So if a user is part of a group, he has the same access rights as the group. On the other hand, access right propagation from a parent **item** to its descendants is **explicit**, i.e., new entries in the `groups_items` table are created each time new access rights are given to propagate these rights to the children. The propagation through items follows specific rules described in [this page]({{ site.baseurl }}/design/access-rights/items/).
