---
layout: page
title: "Architecture Decisions: Gherkin"
nav_order: 1000
parent: Backend
---

# Architecture Decisions: Gherkin


## When using a Gherkin table to define entities parameters, put the primary key(s) in the first column(s).

Date 11/04/2023

Why?
It makes readability easier.


## Don't merge the definitions of item's permissions with the notion of item validation present in the results table

Date: 11/04/2023

Why?
A rule mixing item validated (results table) with item's permission (e.g. can_view) is specific to threads and will only be used for thread.


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
- It avoids the redundancy of defining low level database information related to entities.
- It makes scenarios easier to read.
- It makes scenarios easier to write.
- Writing the tests for one module shouldn't require deep low level knowledge of other modules.
- It makes technical refactoring that doesn't change domain rules easier by only having to update the Gherkin higher level rules, and not the low level database implementations.
- It makes the robustness tests more robust. If a new requirement is added, existing robustness tests might return an AccessDenied error for the wrong reason (because the new requirements are not added in the test database). If that happens, they won't fail anymore if what they initially tested is broken. Writing them in a higher level ensures that if some requirements change, those will be reflected in those higher level features, because the same are used in the acceptance tests.
- It makes the tests less brittle. If there is a change in the module groups, we shouldn't need to update the tests for the threads.
- The closer the code is to the language we express our requirements, the more confident we can be that we don't introduce unnecessary complexity.
