---
layout: page
title: Development Processes
nav_order: 10
parent: Backend
---

# Development Processes

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
