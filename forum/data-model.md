---
layout: page
title: Data model
nav_order: 10
parent: forum
---

# Data model

Data related to the forum are split between:
- DynamoDB (unstructructured data), mainly for message content and notification system
- Backend relational database, mainly for everything related to permission checks

The main "entities" in forum are "threads", they correspond to a discussion thread related to one task for one participant (a user or a team). So thread are unique for a `item_id, participant_id`.

## In the relational DB

As a forum thread has `participant, item` as key, which does not correspond to any existing table, it has to be implemented on its own table. It also requires specific permissions which are described on the permission page.

One of the major property of a thread (except PK) is the status. It can be:
- waiting_for_participant: the thread is opened, waiting for the participant to answer
- waiting_for_trainer: the thread is opened, waiting for the trainer to answer
- closed: the thread is currently closed (may be reopened later), meaning the participant's questions have been answered
In addition to these three status, it is worth mentioning a PK may not have entries in the forum table, if it has never been created. Such a case is equivalent to "closed" in many aspects.
The *waiting_for_participant* and *waiting_for_trainer* status will be simplified to *open* in the explanation of this doc.


## In DynamoDB

The whole forum-related data is implemented in a single DynamoDB table. Its primary key (partition key) is `PK`, a the sort key is `time`.

### Thread events

Thread events have as PK: `THREADEV#${participantId}#${itemId}`, `time` corresponsponds to the time of the event.

(TODO: check if up to date) Thread events have a `type` and a set of custom attributes. Current event types are:

| type             | other attributes                                                       |
|:-----------------|:-----------------------------------------------------------------------|
| `thread_opened`  | `user_id` |
| `attempt_started`| `attempt_id` |
| `submission`     | `attempt_id`, `answer_id`, `score` (opt), `validated` (opt) |
| `thread_closed`  | `user_id` |
| `message `       | `user_id`, `content`

### Thread subscriptions

Thread events have as PK: `THREADSUB#${participantId}#${itemId}`.

Subscriptions for a thread correspond to all users currently watching the thread and allow sending any new event on "their websocket". Each time a user leave a item page or has his websocket connection resetted, new entries are created/deleted.

All subscriptions have a `ttl` property which allow DynamoDB to remove too old entries. Anyway, as API Gateway do not allows Websocket connections longer than 2 hours, we know that older entries are invalid. In addition, each time a message cannot be sent on a websocket connection, the subscription is deleted.


