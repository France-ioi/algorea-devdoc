---
layout: page
title: Token Storage Stategies
nav_order: 20
parent: Authentication & Authorization
has_children: true
---

# Token Storage Stategies

Storing access token in browser local or session storage is considered harmful in terms of security (as any JS code on the domain can access them, leaving some doors open for XSS) <sup>[1](#ref1), [2](#ref2), [3](#ref3)</sup>.

Two options remain:
* Using httponly (not accessible via JS) cookies. This requires either using HTTP**S** or the using the same domain (SPA and backend), otherwise it will be blocked by browsers.
* relying on reauthentication each time we need and store the token in the app directly while in use (less safe)

The *cookie* option should be preferred, except if it cannot be used, for instance for local development using an online backend.
The app is capable of choosing by itself between the two by analysing the url of the backend and preferring *cookies*.

## References

1. <a name="ref1"></a>[OWASP HTML5 Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html#Local_Storage)
1. <a name="ref2"></a>[Please Stop Using Local Storage](https://dev.to/rdegges/please-stop-using-local-storage-1i04)
1. <a name="ref3"></a>[Secure Access Token Storage with Single-Page Applications](https://medium.com/@benjamin.botto/secure-access-token-storage-with-single-page-applications-part-1-9536b0021321)
