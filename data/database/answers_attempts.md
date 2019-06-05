---
layout: page
title: Answers & Attempts
nav_order: 10
grand_parent: Data Model
parent: Database
---

# Answers & Attempts

The management of submissions is mainly managed through 3 tables (`groups_attempts`, `users_answers`, `users_items`) and is currently in a messy unconsistent state in a mix a several solutions.

There is a "old" [requirement document](https://docs.google.com/document/d/19B-Bsab8ZaR72nhXIc8Tg6gvUX25C1HNu-BmctEQjdw/edit?usp=sharing) related to this topic.

## Concepts

### Teams
Users may solve a task (item) either alone or as a team, depending on the contest. This is defined by the `items.bHasAttempts` of the contest, when true the contest has to be run in team (WTF?).

When alone, the changes are done through his selfGroup. When in a team, the operations are done through his team, which is the only (current limitation) group which is direct parent of user selfGroup and which has access to the contest (item).

### Attempts

An attempts is a instance of the same task with different parameters that make the task different (e.g., a random seed changing the data set).

Only items with `items.bHasAttempts` can have multiple attempts. The other one can have at most one attempt.

### Submissions

A user can manually submit an answer that he has, typically to get a score back.

## Current state

* `groups_attempts` represents



