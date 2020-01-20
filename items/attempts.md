---
layout: page
title: Attempts & Score Propagation
nav_order: 100
parent: Items & Related Entities
---

# Attempts & Score Propagation

**Attempts** are attempts of a participant (team or user) to solve one specific instance of a task (or chapter) optionally with specific parameters. An attempt can have zero, one, or multiple answers. Only items which supports multiple attempts (`items.allow_multiples_attempts`) can a several attempt for a same participant.

The result that a participant gets on an item are stored in the attempt and is propagated together with other data to its parent (item's parent) attempts.

The table structure is available [here](https://franceioi-algorea.s3.eu-west-3.amazonaws.com/dbdoc/tables/attempts.html).

## Attempt creation

New attempts are created only when:
* A participant first visit an item: if the UI discover the participant has no item for the currently viewed item, it requests the creation of a new one.
* A participant manually creates another attempt: when visiting an item, if the item allows it, the user has a button to create a new attempt for this item.
* For the items which requires to manually "enter" (only contests for now). The "entering" creates an corresponding attempt.
* The results are propagated to a parent attempt which has not been explored by the user yet. In this case, a new attempt is created (without `creator_id`) to store the results.

### `started_at`

The `attempt.started_at` attribute is set when the participant starts the attempt, so it is kept null when the attempt is created for result propagation. It means that the `started_at` will have to be set manually on the attempt when the item is actually first visited.


## Result propagation to parent attempts (item's parents)

Each time one of the propagated information is changed in a attempt, the results are repropagated to its parents. In the case of multiple attempts for the same items, only one of the attempts counts (typically the max value for each attribute, except for `validated_at`).

The following attributes are propagated, so their values in chapter attempts is a "summary" of the descendant attempts. (for attribute description, see [the table structure](https://franceioi-algorea.s3.eu-west-3.amazonaws.com/dbdoc/tables/attempts.html))
* `latest_activity`: the latest activity across all descendants
* `tasks_tried`, `tasks_with_help`: the sum of all descendants
* `score_computed`: the weighted sum of children scores (note that the score can be overriden manually for any item, or a bonus/malus can be given)
* `validated_at`: time at which the item was validated, chapter validation depends on several the `items.validation_type`.

Note that the `started_at` attribute is **not** propagated, it is a property of the attempt itself.

### Propagation optimization: not propagating the invible items

In order not to create many attempts for parents of items which have been included by many users in their private chapters, we do not create new attempts for parents that the participants cannot currently see. It requires creating them when new permissions are given (quite complex).

### Propagation to itemw requiring manual "enter"

In order not display results to a contest which has not been started yet by a participant, we do not auto-create attempt by result propagaton for the items which requires manual "enter" (only contest for now).
