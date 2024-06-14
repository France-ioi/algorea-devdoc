---
layout: page
title: Tips
nav_order: 550
parent: Sessions
---

# Sessions

The service that creates and refresh the session is the same: `accessTokenCreate`.

A user can have multiple sessions, typically one per device.

A session can have multiple access tokens. The access tokens have a lifetime of 2 hours,
after which they have to be refreshed. There can be more than one valid access token for a session at the same time,
but we always want to use the most recent one.

If there's no session, we create a temporary user, and a session is created for it.

If a `code` is sent (coming from the login-module by the frontend after authentication), the session is created for the user that logged in,
as described in the section [Creation of a session for a normal user (not a temporary user)](#creation-of-a-session-for-a-normal-user-not-a-temporary-user).

If there's already a session for the user (and no `code` is sent), the process is described in the section [Refreshing the token](#refreshing-the-token).


## Creation of a session for a normal user (not a temporary user)

A session is created when a user logs in.

This is the process of creating a session:

1. We call the login-module to get back:
  - the access token
  - the refresh token
  - the expiration time of the access token
2. If the user doesn't exist in the database, we create it, otherwise we update the `latest_login_at`, `latest_activity_at`, and `latest_profile_sync_at` fields.
3. The session is created in the database with the refresh token. The access token is added to the session.
4. If there are already 10 sessions for the user, or more, we delete the oldest ones, to have a maximum of 10 sessions in database.
5. We return the access token with its expiration time.


### There's a limit of 10 concurrent sessions per user

We limit the number of concurrent sessions per user to 10.

This is to avoid bloating the database with too many sessions.


## Refreshing the token

This is the process of refreshing the token:

1. (Authentication middleware) We check that the token is valid and not expired (same as any authenticated request).
2. We retrieve the most recent token of the session. If the token used to call the refresh is not the most recent one,
   we return the most recent token with its expiration time.
3. (From now on, we're working with the most recent token). If the token has been issued less than 5 minutes ago,
   we return the token with its expiration time.
4. We get a lock for the session.
5. We call the login-module to refresh the token, with the refresh token of the session, and get back:
  - the new access token
  - the new refresh token
  - the expiration time of the access token
6. We store the refresh token in the session, and store the new access token.
7. We delete expired old tokens for the session.


### Only the most recent token can be used to refresh the token

If another token is used to refresh the token, the current most recent token is returned with its expiration time.


### A token can only be refreshed once

Because a token can only be refreshed by the most recent token, it implies that a token can only be refreshed once.


### A token can only be refreshed if it's not expired

If an expired token is used to refresh the token, access defined error is returned.


### A token can only be refreshing 5 minutes after it was issued

It's not possible to refresh a token that was issued less than 5 minutes ago.

This was done because:
- There can be many tabs open in the browser. Each tab will ask for a refresh of the token at roughly the same time.
  We don't want to have many refreshes going at the same time: it's completely useless.


### There a maximum number of access tokens for a session that can be present in database at one moment

This follows from the facts that:
- a token can only be refreshed once
- a token can only be refreshed after 5 minutes
- expired tokens are deleted on refresh

Since tokens are valid for 2 hours, we can have a maximum of 2 hours / 5 minutes = 24 tokens in the database at the same time.


## Logout

When a user logs out, the session and all the related tokens are deleted.


## Edge cases with the frontend

The tricky edge cases come from the frontend when:
- tabs are frozen when inactive and in background for some time
- the device goes into sleep


### What happens if the frontend tries to refresh a token that was already refreshed (that is not the most recent one)?

It will receive the most recent token with its expiration time.

With the expiration time, it will know when the token has to be refreshed again. Which can be right now.


### What happens when many tabs make a refresh request at the same time?

There is a lock for the session for the refresh process.

1. The first request gets the lock, and the token gets refreshed.
2. The other requests (already made before the refresh was done), are using tokens that are not the most recent anymore.
   The most recent token (just refreshed) will be sent with its expiration time.
3. There might be other refresh requests that are made after the first one is done. Those will use the most recent token,
   but since this token was issued less than 5 minutes ago, the token will simply be returned with its expiration time.

