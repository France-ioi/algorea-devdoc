---
layout: page
title: "Auth: Working docs"
nav_order: 35
parent: Login Workflows
grand_parent: Authentication & Authorization
---

# OAuth2 and best practices

In web browser, only OAuth2 "implicit grant" and "authorization code" workflow can be used. Here are some interesting refs to read about them:
- [OAuth2 standard](https://tools.ietf.org/html/rfc6749#section-4.2)
- [An Introduction to OAuth 2](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2)
- [How OAuth 2.0 works and how to choose the right flow](https://itnext.io/an-oauth-2-0-introduction-for-beginners-6e386b19f7a9)
- [Which OAuth 2.0 Grant should I use?](https://auth0.com/docs/api-auth/which-oauth-flow-to-use)

Discussions about recent (2019) recommendations at IETF:
- [OAuth2 Implicit Grant and SPA](https://auth0.com/blog/oauth2-implicit-grant-and-spa/)
- [What is going on with OAuth 2.0?](https://medium.com/securing/what-is-going-on-with-oauth-2-0-and-why-you-should-not-use-it-for-authentication-5f47597b2611)

These recent recommendations are to use, for SPAs, the "authorization code" workflow, with PKCE and the *state* parameter.

# AlgoreaPlatform workflow (on 06/2019)

{% include algoreaplatform_auth.html %}

It uses a tweaked "authorization code" workflow through an authorization popup.

Some remarks:
- The redirect from the auth module to http://concours.algorea.org leaks the authorization code on a weak (HTTP) channel. This is a violation of OAuth2.
- The authorization on AlgoreaPlaform only relies on its PHP session after this first authorization. It means the access and refresh token provided by the *login module* are just ignored, including their validity time. It may lead to unconsistency between the two sessions (difficulty to revoke globally a user, problem at logout or to get user data, ...). OAuth2 is not supposed to be used this way.
- Further session refresh (and more user info fetching) can be done through the backend which has a *refresh token* that it can use to contact the *login module* again.

# Possible workflows

## Using "implicit" workflow

{% include implicit_auth_workflow.html %}

Some remarks:
- The implicit workflow cannot use *refresh token* as it is considered unsecure to store it in the SPA. The refresh may be done through "silent authentication", i.e., requesting the auth popup (or an iframe) to use the session with the auth server to ask for a new *access token*.
- As written above, the recent (2019) official recommendation is not to use "implicit" anymore as it may leak the token too easily (through url, history, weak channel, ...).

## Using "authorization code " workflow

{% include authorization_code_workflow.html %}

Some remarks:
- As OAuth2 defines that apps such as SPAs are not allowed to store refresh token, there is no "OAuth" way to get a new *access token* when it has expired. One way to proceed is to use "silent authentication" as defined above for the implicit workflow.

# Current proposal

The following workflow using OAuth2 "authorization code" looks the more approriate to Algorea. Actually, the main difference with what has been done so far is that the *access token* is used by the frontend directly.

{% include login_workflow.html %}

## Refreshing tokens

*Login module* currently uses the default values of 1 hour for *access token* lifetime and 2 weeks for the *refresh token*.

As AlgoreaBackend cannot validate the *access token* provided by the frontend once it has expired, there are two strategies for keeping the user logged in (without required credentials re-entry) more than the token lifetime:
- Anticipating the access token refresh. When the frontend know the token is only valid for 10min, it may ask the backend to get a new access token (the backend has a refresh token to do so).
- Use "silent authentication" (which is a bit cheating with OAuth2), by re-opening the auth popup (may be in a hidden iframe), and use the session cookie used at first login to do a new authentication without prompt to the user. It requires this session to be long-enough (longer than the access token lifetime).
