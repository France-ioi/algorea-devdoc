---
layout: page
title: Schema & Data Assumptions
nav_order: 8
grand_parent: Data Model
parent: Database
---

# Schema & Data Assumptions

Many constraints or assumptions are not expressed in the DB schema. The following is a non-exhaustive list of some of them before they can be managed in a better way:

* A `type` of groups (`groups` table) is "**Team**". These teams are a special type of groups with as direct children only users. Teams are meant to stay small. Some contest can be only solved in teams. It has impact on the [Answers & Attempts](../answers_attempts) as well.
