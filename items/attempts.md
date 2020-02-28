---
layout: page
title: Attempts, Results, Propagation
nav_order: 100
parent: Items & Related Entities
---

# Attempts, Results, and Propagation

**Results** are, for a given participant (user or team), the score, starting time, validation status, ... which has been obtained on a given item, for an attempt. For tasks, a result is the aggregation of several answers. For chapters and skills, a result is the aggregaton from the results of child items.

By default, all results are related to a default **attempt** (with id "0") but a participant may decide to redo a task or a full chapter from scratch, so as a new **attempt**. For some tasks, creating doing another attempt generates a variation of the same task. Only items which supports multiple attempts (`items.allow_multiples_attempts`) can have several attempts for a same participant.

## Design

Relations between **attempts**, **results** and related entities are the following: (the full db schema is [here](https://franceioi-algorea.s3.eu-west-3.amazonaws.com/dbdoc/tables/attempts.html))
<div style="max-width:80%;">{% include attempts_relations.html %}</div>

In the following, we show a scenario of attempts for a given user. The attempt "0", the default one is used (in a **result** row) for every item which has been visited or which has a score. The user has also visited C2 but as it is a contest which requires explicit entry, a new attempt "1" has been created for it. The user has also created on T2 (and respectively T3) a second attempt "2" (resp. "3") which are subattempt of "0".

<div style="max-width:70%;">{% include attempts_example.html %}</div>

## Attempt creation

The default attempt (with id 0) is created automatically when a new user or team is created. The other attempts are created explicitely by the user or one member of the team. For explicit-entry chapters (contests for instance), the attempt is created when entering (after having checked that the conditions are met).

A explicitely-created attempt (all but default) have a "parent attempt" (`parent_attempt_id` attribute), i.e., the attempt from which this one was created. We also retain the item on which this attempt was created (`root_item_id` attribute). This is used in result propagation.

## Result creation

The results are created:
1. when the user starts working on an item. In practice, when the user has *can_view>=content* and:
  - on task, opens its page (before a task token is explicitely created)
  - on chapter, opens its page or expends it in the left menu
2. manually on explicit-entry items (contests typically), when the user presses "enter"
3. at result propagation, when results are propagated from a task to its ancestors (see result propagation below)

1 and 2 set the `started_at` attribute, while the creation at propagation leaves it null.

## Result propagation to parents

Each time one of the propagated information is changed in a result, it is propagated to its parents (i.e., in the result of the ancestor items). An aggregation occurs (typically the max value for each attribute) when there are multiple child attempts for the same item. In practice, it only happens from the root item of an attempt (`attempts.root_item_id`) to its parent (on the schema, on T2 and T3 to C4 and C5)

The following attributes are propagated, so their values in the result of a chapter is a "summary" of the descendant results. (for attribute description, see [the table structure](https://franceioi-algorea.s3.eu-west-3.amazonaws.com/dbdoc/tables/attempts.html))
* `latest_activity`: the latest activity across all descendants
* `tasks_tried`, `tasks_with_help`: the sum of all descendants
* `score_computed`: the weighted sum of children scores (note that the score can be overriden manually for any item, or a bonus/malus can be given)
* `validated_at`: time at which the item was validated, chapter validation depends on the `items.validation_type`.

Note that the `started_at` attribute is **not** propagated, it is a property of the "result" itself.

### Propagation optimization: not propagating the non-visible items

In order not to create many "results" for items which have been included by many users in their private chapters, we do not create new results for items which do not have ancestors that the participants can actually view.

### Propagation to explicit-entry items

In order not display results to a contest which has not been started yet by a participant, we do not create result at propagaton for the items which require explicit-entry (on the schema, on the attempt "0", when propagating from C4/C5 to C2).
