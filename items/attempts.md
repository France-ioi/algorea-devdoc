---
layout: page
title: Attempts, Results, Propagation
nav_order: 100
parent: Items & Related Entities
---

# Attempts, Results, and Propagation

**Results** are, for a given participant (user or team), the score, starting time, validation status, ... which has been obtained on a given item, for an attempt. Results may be the aggregation of several answers (for tasks) or from the results of child items (for chapters and skills).

By default, all results are related to a default **attempt** (with id "0") but a participants may decide to redo a task or a full chapter from scratch, so as a new **attempts**. For some tasks, re-trying them in another attempt create another variation of the same task. Only items which supports multiple attempts (`items.allow_multiples_attempts`) can a several attempt for a same participant.

## Design

Relations between **attempts**, **results** and related entities are the following: (the full db schema is [here](https://franceioi-algorea.s3.eu-west-3.amazonaws.com/dbdoc/tables/attempts.html))
<div style="max-width:80%;">{% include attempts_relations.html %}</div>

In the following, we show a scenario of attempts for a given user. The attempt "0", the default one is used (in a **result** row) for every item which has been visited or which has a score. The user has also visited C2 but as it is a contest which requires explicit entry, a new attempt "1" has been created for it. The user has also created on T2 (and respectively T3) a second attempt "2" (resp. "3") which are subattempt of "0".

<div style="max-width:70%;">{% include attempts_example.html %}</div>

## Attempt creation

The default attempt (with id 0) is created automatically when a new user or team is created. The other attempts are created explicitely by the user or one member of the team. For explicit-entry chapters (contests for instance), the attempt is created when entering (after having checked that the conditions are met).

## Result creation

The results can be created:
- explictely, when the user starts working on an item, in practice:
  - on task, ...
  - on chapter, ...
  - on explicit-entry items (contests typically), when they "enter"
- implicitely, when the results are propagated from a task to its ancestors (see result propagation below)

Explicit creations set the `started_at` attribute, while the implicit creation leaves it null.

## Result propagation to parents (item's parents)

Each time one of the propagated information is changed in a attempt, the results are repropagated to its parents. In the case of multiple attempts for the same items, an aggregation occurs (typically the max value for each attribute).

The following attributes are propagated, so their values in chapter's result is a "summary" of the descendant attempts. (for attribute description, see [the table structure](https://franceioi-algorea.s3.eu-west-3.amazonaws.com/dbdoc/tables/attempts.html))
* `latest_activity`: the latest activity across all descendants
* `tasks_tried`, `tasks_with_help`: the sum of all descendants
* `score_computed`: the weighted sum of children scores (note that the score can be overriden manually for any item, or a bonus/malus can be given)
* `validated_at`: time at which the item was validated, chapter validation depends on the `items.validation_type`.

Note that the `started_at` attribute is **not** propagated, it is a property of the "result" itself.

### Propagation optimization: not propagating the invible items

In order not to create many results for parents of items which have been included by many users in their private chapters, we do not create new results for items which do not have ancestors that the participants can actually view.

### Propagation to explicit-entry items

In order not display results to a contest which has not been started yet by a participant, we do not auto-create result by propagaton for the items which requires explicit-entry.
