---
layout: page
title: Tests
nav_order: 100
parent: Backend
---

# Tests

The tests of [AlgoreaBackend](https://github.com/France-ioi/AlgoreaBackend) are composed of unit tests and integrations tests.

A coverage of 100% is expected.

## Unit tests

The unit tests are implemented like regular go tests. Next to a `filename.go` file is a `filename_test.go` which contains the tests.

## Integration tests

We use the Behaviour-Driven Development framework Cucumber with Gherkin to write our integration tests.
This allows writing tests in *almost* natural language and interact with a real database.

Those tests are situated in *.feature files.  We distinguish two files:
* `filename.feature` contains the tests relative to the features of the specification
* `filename.robustness.feature` are the robustness tests: what happens with invalid data, invalid access, wrong permissions.

All `.*feature` files usually contains a definition of what's present in the database. You'll find examples by opening any of those files.

### Rules to write Integration Tests

Read the
[Architecture Decisions for Gherkin Tests]({{ site.baseurl }}/algorea-devdoc/architecture/backend/decisions-gherkin/) document.

### Tags

To refer to a specific test, you can add a tag on the line before. This will enable you to test only those tests which are tagged:

```
@wip
Scenario: User is able to save an answer
  Given I am the user with id "101"
  ...
```

It's also possible to mark all tests of a file with a tag at the beginning:

```
@wip
Feature: Create a 'saved' answer
  Background:
    ...
```

You'll find how to run only the tests marked with a specific tag in [AlgoreaBackend's README](https://github.com/France-ioi/AlgoreaBackend/blob/master/README.md).


### Debug

You might want to use your IDE debug functionalities from a Gherkin scenario in order to use breakpoints,
step by step execution, and deep inspection.

GoLand doesn't yet (2023) allows to run it from a feature file. Here's a workaround:

Add a specific tag in your *.feature file for the target scenario.
Then update the `/bdd_test.go` and update the following line:

    testhelpers.RunGodogTests(t, "tagName")

Put your tag name as the second argument.

Then, add a breakpoint where you want and run the function TestBDD with your debugger.


### Computed tables

The `groups_ancestors`, `items_ancestors` and `permissions_generated` tables are computed.
You shouldn't attempt to fill them by hand; unless that's exactly what you're testing.


#### New test system

The new test system with features defined in `steps_app_language.go` automatically compute the tables,
as long as one entity is added in the database through one of those Gherkin features.

The computation is made when the `TestContext.needPopulateDatabase` is true.


#### Old test system

For tests who aren't using `steps_app_language.go`,
the automatic process of generating the computed tables is not enabled.
The reason is
that some of those tests define the content of those computed tables without defining the data in the origin table
(e.g. `permissions_generated` is defined but not `permissions_granted`,
and the latter is supposed to be the source of the primer).
Some even define data in both computed and origin tables that are inconsistent.

In order to compute table `groups_ancestors` for those tests, populate the table `groups_groups`,
and then use the following Gherkin feature:

```
  ...
  And the database has the following table 'groups_groups':
    | parent_group_id | child_group_id |
    | 1               | 2              |
    ....................................
    | 1               | 10             |
  And the groups ancestors are computed
  ...
```

For the tables `items_ancestors` and `permission_generated`,
only defining the content of those tables manually is possible for now.

When all the old tests will be updated to always define the data in origin tables,
we'll be able to remove the `TestContext.needPopulateDatabase` switch,
and use the "New test system" for all tests.


### Freeze time

To avoid undeterminedness due to time, you can use the following Gherkin feature:  `And time is frozen`.

You can also define `time.Now` at a specific time with `And the time now is "2020-01-20T00:00:00Z"`.

Note that to define the database time requires the use of another Gherkin feature:
`And the DB time now is "2020-01-01 00:00:00"`.


### Other custom definitions

You'll find the other definitions in [`testhelpers/feature_context`](https://github.com/France-ioi/AlgoreaBackend/blob/master/testhelpers/feature_context.go)

## Resources

* [Cucumber & Gherkin doc](https://cucumber.io/docs/guides/overview/)
