---
layout: page
title: Login by Group Code
nav_order: 80
parent: Authentication & Authorization
---

# Login by Group Code

A temporary user may login as an actual user by entering a group code. In practice, the temp user does not become a group user but a new user is created under the hood and the user auto-logged in.

The workflow: (every step assumes that the previous one has succeeded)
* the temp user enter a code to join a group
* the SPA knows that the user is temporary
* the SPA requests the backend whether the code is correct
* the SPA requests the login-module to create a new user and receive a login code and auto-login token (one-time token)
* the SPA auto-logins on login-module with no prompt using the auto-login token, and receives a session token from the backend
* the SPA requests the backend to join the group by code (with the code entered at step 1)
* the SAP shows the "login code" to the user that he can use the next time to login (so no login/pwd for him)
