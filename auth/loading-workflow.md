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

If cookies are not used, no token can have been stored by the browser. However, the expiration may still have been stored, indicating that
last time the website was used, there was a session open.

### Case 1: Login callback

If *code* and *state* are provided as parameters, it means we are back from the login-module after a successful login.
In this case, the current expiration, if any, is removed and the OAuth authentication continues... (see login workflows)
On success, the token is stored in the app state (but not stored)

### Case 2: Expiration stored (still valid of not)

FIXME: does not work in case of temporary user.

If an expiration is stored, we consider the user was logged in the last time he visited the website. So the application
redirects to the login-module, which redirects immediately to the platform with either the code & state which will allow
 the application to get a token, or without credentials indicating that the user was not authenticated (LM service yet to be done).

### Case 3: Nothing stored

Just start a temporary session.

