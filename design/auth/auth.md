---
layout: page
title: Authentication & Authorization workflow
nav_order: 30
parent: Design
has_children: true
---

Authentication and authorization are managed in common through most France-IOI applications, using the OAuth2 protocol. The authorization is centrally managed in the [Login Module](https://github.com/France-ioi/login-module). The PHP clients share a lib which implement the OAuth2 client to *Login Module*: [login-module-client](https://github.com/France-ioi/login-module-client).

The previous AlgoreaPlaform workflow and the different options are described in the [working docs](../workingdoc).

# OAuth Login Workflow

The following workflow using OAuth2 "authorization code" looks the more approriate to Algorea. Actually, the main difference with what has been done so far is that the *access token* is used by the frontend directly.

<iframe frameborder="0" style="width:100%;height:440px;" src="https://www.draw.io/?lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=loginsuccessful-authcode-full.drawio#Uhttps%3A%2F%2Fdrive.google.com%2Fa%2Fsmad.be%2Fuc%3Fid%3D1LnKvL_pudCBs9miqOlYJdAjtwe5CIIsw%26export%3Ddownload"></iframe>

# Logout Workflow

<iframe frameborder="0" style="width:100%;height:474px;" src="https://www.draw.io/?lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=logout.drawio#Uhttps%3A%2F%2Fdrive.google.com%2Fa%2Fsmad.be%2Fuc%3Fid%3D13yhLJA3MTP0peFQT-T2mtVm8l9VaG081%26export%3Ddownload"></iframe>

# Token Refresh

<iframe frameborder="0" style="width:100%;height:434px;" src="https://www.draw.io/?lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=tokenrefresh.drawio#Uhttps%3A%2F%2Fdrive.google.com%2Fa%2Fsmad.be%2Fuc%3Fid%3D1P-YMuir4xO6j64akwz3yYKpQYuCrIkZH%26export%3Ddownload"></iframe>
