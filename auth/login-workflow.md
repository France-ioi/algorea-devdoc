---
layout: page
title: Login Workflows
nav_order: 10
parent: Authentication & Authorization
has_children: true
---

Authentication and authorization are managed in common through most France-IOI applications, using the OAuth2 protocol. The authorization is centrally managed in the [Login Module](https://github.com/France-ioi/login-module). The PHP clients share a lib which implement the OAuth2 client to *Login Module*: [login-module-client](https://github.com/France-ioi/login-module-client).

The previous AlgoreaPlaform workflow and the different options are described in the [working docs](../workingdoc).

# OAuth Login Workflow

The following workflow uses OAuth2 "authorization code" workflow which is the only one which now recommended for SPAs.

{% include login_workflow.html %}

# Logout Workflow

{% include logout_workflow.html %}

# Token Refresh

{% include token_refresh_workflow.html %}
