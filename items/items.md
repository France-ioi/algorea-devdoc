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
| Have children items     | Yes                   | No                       | No                   |
| Attempts have answers   | No                    | Yes                      | No                   |
| Attempts have a score   | Through propagation from children | Through submitted answers | Never                   |
| Relevant dates for attempts | entered_at (for contests), started_at, validated_at, latest_activity_at | started_at, validated_at, latest_activity_at, score_obtained_at, latest_answer_at, latest_hint_at | started_at, latest_activity_at |
| Can be a contest | Yes | No | No |
| Can have several attempts | currently no (later yes) | yes | no |
