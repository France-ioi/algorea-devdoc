---
layout: page
title: Development Processes
nav_order: 10
parent: Backend
---

# Development Processes


## Tackling a new task

1. Understand the context of the services or parts that are impacted: how it is used by the frontend, what are the parameters, what do the response contain. See [Algorea API documentation](https://france-ioi.github.io/algorea-devdoc/api/). Take the time to understand, or you might lose a lot more in later steps. **As a rule of thumb, everything you try to ignore to go faster will go against you later.**

2. Clarify the task if there is any discrepancy between what you understand from the first step, and what has to be done.

3. Once everything is clear, write the first test, and make it fail, using a TDD approach. The TDD approach might be difficult to follow in some specific cases, but it should be the default approach.

4. Make the test pass.

5. Refactor.

6. At this point, you can consider doing a commit. **The commit message must be clear and explain the choices made.**

7. If there is more to do, go back to step 3.

8. When everything is done and in a working state, follow with the checklist for a new PR.


## Checklist for a new PR

- [ ] If the change is linked to an issue present in the [Planning](https://github.com/orgs/France-ioi/projects/2),
set its status to `in progress`
- [ ] If it addresses an issue, make sure the issue is linked in the `Development` tab on the right.
This will allow the issue to be closed automatically when the commit is merge,
and update its status if it is present in the planning.
- [ ] Add a description of what the PR is about
- [ ] If choices were made, describe the reason you made them and the alternatives you considered with a trade-off analysis.


- **If a change is related to a service:**
  * [ ] Make sure the swagger documentation is properly updated
  * [ ] the description renders correctly
  * [ ] all the field and their annotations are correct for the request and response

Launch local:
`swagger generate spec --scan-models -o ./swagger.yaml && swagger validate ./swagger.yaml && swagger serve ./swagger.yaml`.

**Small tip:** You need to have two empty lines between paragraphs, otherwise the generated documentation will have everything on the same line.

- [ ] The changes must follow all guidelines present in: [Architecture Decisions documents]({{ site.baseurl }}/backend/)
- [ ] Commits should be atomic: only contain one thing
- [ ] The commit message must be clear and explain the choices made


## Algorea Planning

The [Planning](https://github.com/orgs/France-ioi/projects/2) contains the planned developments and the people assigned to each item.

The status of items must be updated when:
- Item is `in progress`, when this is being worked on
- Item is `blocked`, when something else must be done first or a question must be answered first
- Item is `under review`, when the PR has been made and passes the validation, ask for a review and update the item
- When the PR is merged and the issue is linked to the PR, the status is automatically changed to `merged`.


## Git tips

### Add only some changes of a file to the current commit instead of all the changes of the file

`git add -p path/to/file`


### Add current changes into the last commit

`git commit --amend`


### Update the current branch with the content of master

In your branch:

`git rebase master`


### Undo the last commit and put its changes in the current work

`git reset HEAD~1`


### Change the commit history, update commit messages, remove commits, change their order, add something in an old commit, ...

`git rebase -i HEAD~N`

With `N` the number of commits you want to treat.
It's better to check the doc before using an option.
