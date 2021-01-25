---
layout: page
title: App Loading
nav_order: 60
parent: Authentication & Authorization
---

# App loading auth workflows

Read [Token Storage Strategies](../token-storage/) first.

## Login callback

If *code* and *state* are provided as parameters, we are back from the login-module after a successful login.
In this case, the OAuth authentication continues (see login workflows) and the token is stored depending on the *token storage strategy*.

## Cookie-only token

If token are only stored in cookies, the SPA never has the hand on it and so does not know on launch whether the user is logged or not. So the first request should be a request of token expiry info and user type (temporary or not). This request may fail with a 401 if there is no cookie set. In this case, the app knows it has to request the creation of a temporary user.

## Reauth session storage

The application, when starting, redirects to the login-module, which redirects immediately to the platform with either the code & state (which will allow the application to get a token), or indicating that the user was not authenticated (LM service yet to be done). In the later case, the app starts a temporary session.

## Enforcing the token for easier debugging

(yet to be defined)
