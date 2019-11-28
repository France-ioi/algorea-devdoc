---
layout: page
title: Contest management
nav_order: 80
parent: Items & co
---

# Contest Management

A contest is an item (typically a `chapter`) which has a duration. This duration is the amount of time a contestant has to get a maximum score once he has started the contest. This item has several children items, which are the tasks which has dependencies between them as any set of items.

Both a user on his own or a team of users (a group in both cases) can participate to a contest. Below, we will use the word **participant** for either a user or a team (users acting for and as a team). For teams, except stated otherwise, the conditions applies on the team entity itself.

## Before a contest

Before starting a contest, the participant should be able to see the title and description of the contest (the parent item) but nothing about the tasks themselves. Practically, it means he must have "grayed" access to the parent item and no access at all to the children.

## Starting a contest

For starting a contest, the participant goes on the contest item (as he has greyed access) and press a button (if the conditions defined below are met) a button to start the clock for himself.

The UI may not display the button if the conditions are not met and ask if the user wants to participate alone or as one of his team.

### Conditions for entering

* The item must be a contest, so have a duration
* The user "pressing the button" can view the contest item (at least grayed access).
* The participant must not have started the contest yet (so `group_participations.contest_started_at` is not set)
* If `items.contest_max_team_size` is set and the participant is a team, validate that the number of users in the team is lower or equals to this number.
* If `items.contest_entering_condition` (somehow former `sTeamMode`) is:
  * "None": no additional conditions (one can enter the contest at any moment)
  * "One": the current time needs to be included in the `group_contest_permissions.can_enter_from`-`can_enter_until` time range for the contest item and one of the group ancestors of either the user (if participating alone) or <span style="text-decoration: underline">at least one member of the team</span>.
  * "All": same but <span style="text-decoration: underline">all members of the team</span>.
  * "Half": same but <span style="text-decoration: underline">half of the members (ceil-rounded) of the team</span>.

## Changes on entering

* We set in `group_participations`: `started_at` to now, `contest_started_at` to now, `last_activity_at` to now
* We give access to the tasks (the ones we want the participant to have access at the beginning) by adding him to a dedicated group `items.contestant_group_id` which has a partial access to this contest, with a membership expiring at `now + duration`.

## Finishing

When the contest time of the participant is over, his group membership is expired so he loses his access to the content of the contest.

## Time extensions

In case time extensions (or reduction) should be granted to the participant (or to any of his ancestors), the end of the contest would be postponed. In practice, it means that it requires group membership expiration to be recomputed for each participant.

## Changing team after entering the contest

<span class="label label-yellow">TODO</span>
* Changing the members (adding or removing some) of a team when the team has entered a contest is allowed as long as the "conditions for entering" (defined above) related to team members are still valid.
* ++ handling modification after the first contest
* ++ handle `items.bTeamsEditable`

## Still to be discussed/decided

1. Table name for storing participation information: `group_participations`?
1. Should `can_enter_from` and `can_enter_until` be stored in a separate table (`group_contest_permissions`) or in the global permission table (`group_permissions`).
1. In what group do we put participants when starting the contest? `items.contestant_group_id`?
1. As items <-> contest is a composition relation (1 <-> 0-1) and there are multiple attributes related to contest only in `itmes`, shouldn't these specific attributes be moved in a specific `contests` table (keyed by `group_id`)?

