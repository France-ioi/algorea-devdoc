---
layout: page
title: Development Processes
nav_order: 10
parent: Backend
---

{% include clickable-checkboxes.html %}

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

### If the change is linked to an issue present in the [Planning](https://github.com/orgs/France-ioi/projects/2):

- [ ] Set the issue's status to "in progress" in the planning.

![Change the status to "in progress"]({{ site.url }}{{ site.baseurl }}/assets/dev-process-status-update.png)

### If the PR fully resolves the issue:

- [ ] Make sure the issue is linked in the "Development" tab on the right page of the PR. This will allow the issue to be closed automatically when the commit is merge, and update its status if it is present in the planning. You can also add "fixes #issue_number" in the description to link it automatically.

![Link the issue in the PR]({{ site.url }}{{ site.baseurl }}/assets/dev-process-link-issue.png)

### If the PR only refers to the issue, without fully resolving it:

- [ ] Add "Related to #issue_number" at the top of the description of the PR.


### If a change is related to a service, make sure:

- [ ] the swagger documentation is properly updated and formatted.
- [ ] it renders correctly. Verify locally with the command: "**make serve-swagger**".
- [ ] the fields of the request and response have the correct annotations (required, Nullable, ...).

See [Architecture Decisions: Services & Code Style]({{ site.baseurl }}/backend/decisions-services-code-style/).

**Swagger tip:** You need to have 2 empty lines between paragraphs, otherwise the generated documentation will have everything on the same line.

### If a change is related to the database, make sure:

- [ ] the migration files follow the Database Migrations guidelines.
- [ ] the migration files are tested for both the "Up" and "Down" parts.
- [ ] make sure you didn't delete any index in case you modified a table.
- [ ] did you change or create new queries? Provide an explanation about why the query need or don't need an index. Add an index if it's needed.

See [Database Migrations]({{ site.baseurl }}/backend/migrations/).

### For all PRs:

- [ ] If you added a lock, you must document why you added it with examples of concrete issues it solves.
- [ ] The changes must follow all the architecture decisions documents.
- [ ] Commits should be atomic: only contain one thing.
- [ ] All commit messages must be clear, stating which service or part they affect, why we want to change it, and explain the reason of the choices made.
- [ ] Add a description of what the PR is about.
- [ ] If you made choices in the implementation, describe the reason you made them and the alternatives you considered with a trade-off analysis: **list the PRO and CON of each alternative**.

See [Architecture Decisions documents]({{ site.baseurl }}/backend/).


### When all the elements are checked, and all tests pass, you can ask for a review:

- [ ] Ask for a review from the person in charge of the service or the part of the code you modified, in the "Reviewers" tab on the right page of the PR.

![Ask for a review]({{ site.url }}{{ site.baseurl }}/assets/dev-process-ask-review.png)


## The PR review process

You must answer to all the comments made by the reviewer.

**Do not click on the "Resolve conversation" button.** Otherwise, the reviewer might miss your answer.

If you disagree with a comment, you must explain why and propose an alternative.

If you don't understand a comment, ask for clarification.

If you agree with a comment, make the change and answer to the comment **starting with "fixed."** to let the reviewer know that you made the change.

Once all the comments are addressed, ask for a review again, using the button on the right of the page, next to the reviewer's name.



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
