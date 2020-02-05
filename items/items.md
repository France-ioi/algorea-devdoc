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

|                         | "Chapter"             | "Task"                   | "Course"             | "Skill"    |
|:------------------------|:----------------------|:-------------------------|:---------------------|:-----------|
| Main usage              | Container for tasks and course | Exercice to be solved to get a score | Content to be read | Skill to be obtained by solving tasks |
| Have children items     | Tasks and courses | No                       | No                   | Chapters (with warning), tasks, courses and skills |
| Attempts have answers   | No                    | Yes                      | No                   | No |
| Attempts have a score   | Through propagation from children (content computation) | Through submitted answers | Never      | Through propagation (skill computation) |
| Relevant dates for attempts | started_at, validated_at, latest_activity_at | started_at, validated_at, latest_activity_at, score_obtained_at, latest_answer_at, latest_hint_at | started_at, latest_activity_at | started_at , validated_at, latest_activity_at (not sure) |
| Can have several attempts | Currently no (later Yes) | Yes | No | No |
| Can be unlocked by other items | Yes | Yes | Yes | Yes |
| Can have prerequisites | ? | ? | ? | Yes |
