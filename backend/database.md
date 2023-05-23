---
layout: page
title: ORM and Database
nav_order: 400
parent: Backend
---

# ORM and Database

## Current Limitation

We are limited by the current version of GORM, which as been forked here in version 1: [forked repository](https://github.com/France-ioi/gorm).

Upgrading to version 2 would allow us to:
- Create complex nested [OR queries](https://gorm.io/docs/advanced_query.html#Group-Conditions)
- Use AddSelect() to make to database code clearer (more infos in comments inside the gormv2 branch)

### Current status of upgrading

The branch here: [gormv2 branch](https://github.com/France-ioi/AlgoreaBackend/tree/gormv2)
Details in this issue: [GORMv2 Update Issue](https://github.com/France-ioi/AlgoreaBackend/issues/769)

- ~~Requires fixes from repository author~~
- ~~Requires update of GO~~
- Requires many fixes to make the tests pass, and a workaround for [this issue](https://github.com/go-gorm/gorm/issues/4533)
