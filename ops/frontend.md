---
layout: page
title: Frontend dev Bootstrapping
nav_order: 150
parent: Ops (installation, running, ...)
---

# Frontend dev Bootstrapping

Code is hosted on https://github.com/France-ioi/AlgoreaFrontend. This is also where tasks are created (as issues) and assigned.

## To install locally

- **VSCode** is recommended as IDE, with the following modules:
  - "Angular Language Service"
  - "EditorConfig for VS Code"
  - "ESLint"
- Node (use the same version as it is used in CI: `.circleci/config.yml` > node image version)
- Try to make the [project](https://github.com/France-ioi/AlgoreaFrontend) run locally as describe in the readme file

## Development workflow

Check the [dedicated website](https://france-ioi.github.io/contributing/github) for the specific workflow we try to apply.

In addition to these general considerations, each time a commit is pushed to the repository, continuous integration is run and the code deployed for manual test. These run on [CircleCI](https://circleci.com/gh/France-ioi/AlgoreaFrontend) for each push (see [config](https://github.com/France-ioi/AlgoreaFrontend/blob/master/.circleci/config.yml)):
- install deps
- run linting (using esling, see [config](https://github.com/France-ioi/AlgoreaFrontend/blob/master/.eslintrc.js))
- build project (with advanced Typescript compiler [options](https://github.com/France-ioi/AlgoreaFrontend/blob/master/tsconfig.json) enabled)
- unit and e2e tests are run (they fail randomly for no (known) reasons, usually re-running them fix the problem)
- deploy the code to http://dev.algorea.org/branch/<branch_name>/
- if master branch, deploy the code http://dev.algorea.org/

## Backend, authentication server and database

For starting, we recommend you to use the deployed test backend (linked with a db filled with anonymized data) which is preconfigured in the frontend configuration (in `src/environments/environment.ts`).

Same for the authentication server.

## Multi language support

On build, the website is automatically compiled to several languages (one build per language) depending on the languages defined in `src/locale`.
To make development easier, only the English version is build on dev mode.

See the [readme](https://github.com/France-ioi/AlgoreaFrontend#internationalization) for more info about internationalization.

## Authentication

Not much is possible on the platform for an unauthorized user.
To test the platform as an authenticated user (required for most dev), you can force your browser to use a specific token (cannot be written here, but ask Damien L.) but setting "forced_token" to a given value in your "Storage > Session Storage" (through DevTools).

## Design

For the design of this platform, we mainly use [prime-ng](https://www.primefaces.org/primeng/showcase/) library that we customize.

## Various (Software) Design Choices

* Observables are not given as input to components.

## Various Style Choices

* Code block (`if`s, loops, ...) are either used with brackets on multiple lines, or a without bracket on a single line (if there is a single instruction in the block of course).

## Avoiding memory leaks

A few rules to follow in our code to prevent memory leaks: (these are not always required but following these simple rules do not cost much and prevent more leaks from happening, even when code afterwards.)
- Every component or service defining its own subject must `complete` it on destroy
- In every component or service, `subscribe()` should store its subscription, and make sure to `unsubscribe` on destroy.
- For every sevices used as component provider (root injected service never really destroy) and in every component, the `shareReplay()` and `share()` should be applied on observables which are sure to complete in the same time frame as the component/service (observables of other non-http service are usually not guaranteed to complete before the end of the app). If it is not clear it is the case, the `shareReplay`/`share` should be either preceded by a `takeUntil(destroy$)` where `destroy$` is a subject which emits on destroy, or use the `refCount:true` parameter if it is clear that all subscribers will unsubscribe (not the case for exported properties).

## Important ressources

* [This website](../):
  - [API](../api/): the most important... typically you should pick the latest version
  - [the db schema](../dbschema): may be useful
  - the rest: may be useful (some part might be outdated)

And docs of framework/libraries we use:
* [Angular](https://angular.io/guide/architecture)
* RxJS: [learnRxJS](https://www.learnrxjs.io/) and [RxJS-dev](https://rxjs-dev.firebaseapp.com/guide/overview)
* [Primeng](https://www.primefaces.org/primeng/showcase/)
* [Typescript](https://www.typescriptlang.org/docs/handbook/intro.html)
