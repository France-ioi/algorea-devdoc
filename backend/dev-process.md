---
layout: page
title: Development Processes
nav_order: 10
parent: Backend
---

# Development Processes

## Checklist for a new PR

### If the change is linked to an issue present in the [Planning](https://github.com/orgs/France-ioi/projects/2):

- [ ] Set the issue's status to "in progress" in the planning.

![Change the status to "in progress"]({{ site.url }}{{ site.baseurl }}/assets/dev-process-status-update.png)

### If the PR fully resolves the issue:

- [ ] Make sure the issue is linked in the "Development" tab on the right page of the PR. This will allow the issue to be closed automatically when the commit is merge, and update its status if it is present in the planning. You can also start the description of the PR with "fixes #issue_number" to link it automatically, but you should check that it worked.

![Link the issue in the PR]({{ site.url }}{{ site.baseurl }}/assets/dev-process-link-issue.png)

### If the PR only refers to the issue, without fully resolving it:

- [ ] Add "Related to #issue_number" at the top of the description of the PR.

### For all PRs:

- [ ] Add a description of what the PR is about.
- [ ] If you made choices in the implementation, describe the reason you made them and the alternatives you considered with a trade-off analysis: **list the PRO and CON of each alternative**.


### If a change is related to a service, make sure:

- [ ] the swagger documentation is properly updated and formatted.
- [ ] it renders correctly. Verify locally with "swagger generate spec --scan-models -o ./swagger.yaml && swagger validate ./swagger.yaml && swagger serve ./swagger.yaml".
- [ ] the fields of the request and response have the correct annotations (required, Nullable, ...).

See [Architecture Decisions: Services & Code Style]({{ site.baseurl }}/backend/decisions-services-code-style/).

**Swagger tip:** You need to have 2 empty lines between paragraphs, otherwise the generated documentation will have everything on the same line.

### If a change is related to the database, make sure:

- [ ] the migration files follow the Database Migrations guidelines.
- [ ] the migration files are tested for both the "Up" and "Down" parts.
- [ ] make sure you didn't delete any index in case you modified a table.

See [Database Migrations]({{ site.baseurl }}/backend/migrations/).

### For all PRs:

- [ ] The changes must follow all the architecture decisions documents.
- [ ] Commits should be atomic: only contain one thing.
- [ ] All commit messages must be clear, stating which service or part they affect, why we want to change it, and explain the reason of the choices made.

See [Architecture Decisions documents]({{ site.baseurl }}/backend/).

### When all the elements are checked, and all tests pass, you can ask for a review:

- [ ] Ask for a review from the person in charge of the service or the part of the code you modified, in the "Reviewers" tab on the right page of the PR.

![Ask for a review]({{ site.url }}{{ site.baseurl }}/assets/dev-process-ask-review.png)


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
