---
layout: page
title: Frontend dev Bootstrapping
nav_order: 250
---

# Frontend dev Bootstrapping

Code is hosted on https://github.com/France-ioi/AlgoreaFrontend. This is also where tasks are created (as issues) and assigned.

## To install locally

- **VSCode** is recommended as IDE, with the following modules:
  - "Angular Language Service"
  - "EditorConfig for VS Code"
  - "ESLint"
- Node 14 (recommanded) or 12
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
To test the platform as an authenticated user (required for most dev), change the token used by your browser in "developper tools > Storage > Session Storage > access_token" to a value given by the administrator of the platform.

## Design

For the design of this platform, we mainly use [prime-ng](https://www.primefaces.org/primeng/showcase/) library that we customize.

[Static mockups of the website](https://france-ioi.github.io/algorea-designs/) are also available as a base for work for new features. Some of these designs have implemented in the design branch (see below).

Note that some of these designs might be outdated, same for the design-branch ones.

### Design branch

The "design" git branch is a specific branch which contains an old mock-up of the website. This is an important resource as it implements (in design only) many features which have not been fully implemented yet.

This website is deployed on http://dev.algorea.org/branch/design/ and the code on https://github.com/France-ioi/AlgoreaFrontend/tree/design. It might be useful to copy-paste some components or part of component from this branch. Most of time, it requries major rework, but that may be better than nothing (mainly for design aspect).


## Important ressources

* [This website](../):
  - [API](../api/): the most important
  - [the db schema](../dbschema): may be useful
  - the rest: may be useful (some part might be outdated)
* [Design branch deployed](http://dev.algorea.org/branch/design/)
* [Commented designs](https://france-ioi.github.io/algorea-designs/)

And docs of framework/libraries we use:
* [Angular](https://angular.io/guide/architecture)
* RxJS: [learnRxJS](https://www.learnrxjs.io/) and [RxJS-dev](https://rxjs-dev.firebaseapp.com/guide/overview)
* [Primeng](https://www.primefaces.org/primeng/showcase/)
* [Typescript](https://www.typescriptlang.org/docs/handbook/intro.html)
