---
layout: page
title: Backend dev Tests
nav_order: 1000
parent: Ops (installation, running, ...)
---

# Tests

The tests of [AlgoreaBackend](https://github.com/France-ioi/AlgoreaBackend) are composed of unit tests and integrations tests.

A coverage of 100% is expected.

## Unit tests

The unit tests are implemented like regular go tests. Next to a `filename.go` file is a `filename_test.go` which contains the tests.

## Integration tests

We use the Behaviour-Driven Development framework Cucumber with Gherkin to write our integration tests. This allows to write tests in *almost* natural langage and interact with a real database.

Those tests are situated in *.feature files.  We distinguish two files:
* `filename.feature` contains the tests relative to the features of the specification
* `filename.robustness.feature` are the robustness tests: what happens with invalid data, invalid access, wrong permissions.

All `.*feature` files usually contains a definition of what's present in the database. You'll find examples by opening any of those files.

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

### Compute table `groups_ancestors`:

You might want to have elements in the table `groups_ancestors`. For this, you only have to populate the table `groups_groups`, and then add the following definition:

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

### Freeze time

To avoid un-determinedness due to time, you can use the following in the definition of a test:

```
  ...
  And time is frozen
  ...
```

### Other custom definitions

You'll find the other definitions in [testhelpers/feature_context](https://github.com/France-ioi/AlgoreaBackend/blob/master/testhelpers/feature_context.go)

## Ressources

* [Cucumber & Gherkin doc](https://cucumber.io/docs/guides/overview/)
