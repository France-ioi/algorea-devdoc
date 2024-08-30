---
layout: page
title: Discussions
nav_order: 90
parent: Interactions with Other Platforms
---

# Discussions

## 2024-08 - Score token: what we provide to the score update service...

Context: The workflow for updating scores when working on a task was the following on early 2024:
- the frontend request a task-token to the backend, it sends it to the task presenter
- when it gets an answer, the frontend sends the backend the answer + task-token to get an answer-token back, a token containing the answer, info about the submission, all signed by the backend
- the frontend sends the task grader the answer-token for grading
- the task grader sends back a score-token (signed by the task grader)
- the frontend submit the score-token + task-token to the backend for "saving the grade"
Note that the communication between the frontend and the backend is authenticated with an access token.

Mid-2024, there were new requirements:
- it should possible to submit using a CLI, it receives the task-token (copy-pasted from the task to the console), then the CLI should be able to continue the process to submit code
- the task grader should be able to grade asynchronously, i.e., to send later the score directly to the backend, ideally without keeping too much state

To meet these requirements, changes were made to the protocol:
- the services to get the answer-token and save the grade using a score-token do not require/user access-token authentication anymore
- the save-grade service does not require a task token anymore

### Issue reported
The save-grade service only receives, as security, the proof that the message has been sent by the grader (by an entity which has its private key). There is no proof that the call corresponds to a task- or answer-token emitted by the backend. If we do not fully trust the grader or the grader has been compromised (his key stolen):

- can a compromised grader submit a score to any task? No as the backend, to check the signature, check the platform key corresponding to the task. So, the grader cannot change score to task for which it is not the grading platform.
- can a compromised grader submit a score to any participant? No, it needs a valid answer id to report a score, so it must corresponds to an actual request (for which an answer token was created)
- can a compromised grader change any previous score sent earlier by the same grader? Yes but only if it knows the answer info (answer id, participant id, item id)

Let's assume, we add the task-token or the answer-token back as a requirement to the save-grade service.
- Does it improve security if the grader has been hacked ? Not really. At least, not as long as the grader keep the task/answer-token for a while to allow resending the score if the answer has been regraded (flaw in the dataset, ...)
- Does it improve security if the private key has been stolen? Yes, as long as the task/answer-tokens has not been stolen as well.

### Decision

Yet to be done.
