---
layout: page
title: "Architecture Decisions: Gherkin"
nav_order: 1000
parent: Backend
---

# Architecture Decisions: Gherkin


## Use higher-level features to use domain/business notions that'd already been tested in another test

Date: 19/04/2023

Why?

Many of the domain/business notions we use are complex and can be fulfilled in many different ways.
Example:
the permission to `List or Read in a thread` in [Forum Permissions](/algorea-devdoc/forum/forum-perm/).

The rule previously mentioned serves as a basis for many other tests: sorting, filters, pagination, etc.

When defining data to test for pagination,
it is tempting to just make sure that this data fulfills one way for the thread to be Listable or Readable.

This would have advert effects:
- It would make the test rigid and brittle. If we change the rules to `List or Read in a thread`, we might have to rewrite the test.
- We should already have tested the `List or Read in a thread` in another test. We don't need to test it again.

Instead, **we define a new Gherkin Feature** `The thread is listable` for example,
either as a feature or as a column in a datatable,
and we use this rule in all the test where we need to use this notion.

Advantages:
- Makes the test more specialized, only testing for one functionality
- Allows us in the future to change the rule without having to rewrite all the tests that depend on it.
- Makes the tests easier to read and write because we don't have to wonder if and how the rule was implemented.


## Use descriptive names for references

Date: 12/04/2023

Why?

It can be tempting to use fancy names in the tests, like `@AlbertEinstein is a manager of the group @Laboratory`.
But this requires a mental translation every time it is read.

It was decided to use descriptive names such as: `@LaboratoryManager is a manager of the group @Laboratory`.
It is easier to read.
It is also easier to verify that the intent is fulfilled.
Here, when we read about the user `@LaboratoryManager`, we can directly check that it was indeed defined as a manager.

For an item, we can use names such as `@B_UniversityMember_CanWatchAnswer1`,
along with a group `@B_University` and a member of this group: `@B_UniversityMember`.

Everytime a reference name is read, what it represents must be obvious.


## Use references in Gherkin instead of IDs

Date: 12/04/2023

Why?

- IDs are a technical consideration
- It makes tests difficult to read and update when we have many objects.
- IDs can hardly have a descriptive value, they don't describe the object it references


## All occurrences of reference to objects in Gherkin must start with @

Date: 12/04/2023

Why?

First,
**we need to be able to distinguish references inside other strings in some features** for the times
when we need to replace those references by their respective ID.
For example, in requests: `When I send a GET request to "/threads?watched_group_id=@Laboratory"`.
But also in responses:
```
    And the response body should be, in JSON:
    """
      [
        {
          "id": "@LaboratoryMember_WithApprovedAccessPersonalInfo",
          ...
```

It was decided that the same format, starting with a @, is also used in other features:

```
  And @LaboratoryMember_WithApprovedAccessPersonalInfo is a member of the group @Laboratory who has approved access to his personal info
```

And datatables:

```
  And there are the following groups:
    | group         | parent        | members                                               |
    | @Consortium   |               | @ConsortiumMember                                     |
```

This makes the use of references consistent and clear everywhere.

We considered defining references explicitly,
but it leads to a lot of redundancies and was abandoned:
~~`And there is a group Laboratory referenced by @Laboratory`~~.


## When using a Gherkin table to define entity parameters, put the primary key(s) in the first column(s).

Date: 11/04/2023

Why?

It makes readability easier.


## Don't merge the definitions of item's permissions with the notion of item validation present in the `results` table

Date: 11/04/2023

Why?

A rule mixing item validated (`results` table) with item's permission
(e.g. `can_view`) is specific to threads and will only be used for thread.


## When using JSONPath to test responses, have at least one test with a full response

Date: 11/04/2023

Why?

- One full response test is needed to ensure the correctness of the response.


## When testing JSON responses, use JSONPath to only verify what's covered by the scope of the test

Date: 11/04/2023

Why?

- It makes the tests less brittle. If we add/rename/remove a field, or change the order of a field in the response, we don't have to change all the tests.
- It makes the responsibility of each test more focused on a single functionality, rather than having each test, test the same things many times.
- It makes the test easier to read when the result contains a list of items: only one line per item.


## Make the definitions as close as possible to the domain language

Date: 08/03/2023

Why?

- It makes the robustness tests more robust by avoiding than they pass because of another reason such as modifying access permissions.
- It avoids the redundancy of defining low-level database information related to entities.
- It makes scenarios easier to read.
- It makes scenarios easier to write.
- Writing the tests for one module shouldn't require deep low-level knowledge of other modules.
- It makes technical refactoring that doesn't change domain rules easier by only having to update the Gherkin higher level rules, and not the low-level database implementations.
- It makes the robustness tests more robust. If a new requirement is added, existing robustness tests might return an AccessDenied error for the wrong reason (because the new requirements aren't added in the test database). If that happens, they won't fail anymore if what they initially tested is broken. Writing them in a higher level ensures that if some requirements change, those will be reflected in those higher level features, because the same are used in the acceptance tests.
- It makes the tests less brittle. If there is a change in the module groups, we shouldn't need to update the tests for the threads.
- The closer the code is to the language we express our requirements, the more confident we can be that we don't introduce unnecessary complexity.
