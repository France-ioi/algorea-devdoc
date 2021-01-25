---
layout: page
title: App Loading
nav_order: 60
parent: Authentication & Authorization
---

# App loading workflows (related to auth)

Read [Token Storage Strategies](../token-storage/) first.


## Cookie-only token

If token are only stored in cookies, the SPA never has the hand on it. However, the SPA knows and stores (in local storage)
the expiration of the token and use this value as a base to know if the current token is authenticated
(i.e., if the browser has probably as cookie to provide to the API).

### Case 1: Login callback

If *code* and *state* are provided as parameters, it means we are back from the login-module after a successful login.
In this case, the current expiration, if any, is removed and the OAuth authentication continues... (see login workflows)

### Case 2: Valid expiration stored

If there is an expiration stored in local storage and this expiration is still valid, store it and let the app fetch the user and continue loading. In case the cookie is no longer valid, the backend will return a 401.

### Case 3: No valid expiration stored

In this case, just start a temporary session by requesting a new temporary session to the backend.


## Reauth session storage

If cookies are not used, no token can have been stored by the browser.

### Case 1: Login callback

If *code* and *state* are provided as parameters, it means we are back from the login-module after a successful login.
In this case, the OAuth authentication continues (see login workflows) and on success, the token is stored in the app state (but not stored)

### Case 2: Otherwise

The application, when starting, redirects to the login-module, which redirects immediately to the platform with either the code & state (which will allow the application to get a token), or indicating that the user was not authenticated (LM service yet to be done). In the later case, the app starts a temporary session.


## Note on local dev

For local development with an online backend, the cookie option cannot be used as we do not have the same domain or https. The reauth option may be difficult to put into place as well if localhost if not an accepted callback domain.
