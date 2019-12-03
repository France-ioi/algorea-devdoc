---
layout: page
title: Items & Related Entities
nav_order: 60
has_children: true
permalink: /items/
---

# Items

Items are implemented through the `items` db table. A item represents some content (tasks, chapter, ...) and are structured (acyclic graph) via the `items_items` table, the ancestors hierarchy is cached in `items_ancestors`.
