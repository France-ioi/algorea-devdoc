---
layout: page
title: Items & Related Entities
nav_order: 60
has_children: true
permalink: /items/
---

# Items

Items are implemented through the `items` db table. A item represents some content (tasks, chapter, ...) and are structured (acyclic graph) via the `items_items` table, the ancestors hierarchy is cached in `items_ancestors`.

The **type** of an item (his `type` attribute) is important to know the actions which are possible on it and which attributes are relevant. The following table compares the main differences:

|                         | "Chapter"             | "Task"                   | "Course"             |
|:------------------------|:----------------------|:-------------------------|:---------------------|
| Main usage              | Container for other items | Exercice to be solved to get a score | Content to be read |
| Can have children items | Yes                   | No                       | No                   |
| Can have answers        | No                    | Yes                      | No                   |
| Can have a score        | Through propagation from children | Through submitted answers | Never                   |

