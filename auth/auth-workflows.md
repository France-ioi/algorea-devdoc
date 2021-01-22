---
layout: page
title: App Loading
nav_order: 65
parent: Authentication & Authorization
---

# Other auth workflows

## 401 received

When the application receives from a backend service a 401 http code, this means the token is not valid anymore.

If the user was authenticated, the user is redirected to the login prompt. If the user was temporary, a new temporary user is created.

## Token auto-refresh (before its expiration)

When a token reaches the half of its lifetime, the application requests a new token using the previous one.
