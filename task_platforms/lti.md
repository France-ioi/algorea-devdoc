---
layout: page
title: LTI Auth Workflow
nav_order: 80
parent: Authentication & Authorization
---

# LTI

## Authentication Workflow (LTI Launch Request)

When our platform acts as a LTI provider, the LTI consumer will typically call our infrastructure with "LTI Launch Request" providing LTI parameters. These can be handled by the login-module which can verify them and create a local user if needed.

Our current two use cases are:
* A platform using our items in an embedded iframe. The tasks and chapters are loaded via a POST request which is redirected to the content to be displayed. The score can also be sent back to the platform (not detailed here).
* A LTI-enabled website redirects their users to our platforms so that the user can utilize our platform without further authentication.

In both cases, the LTI Launch request is a POST with LTI authentication parameters, which can be redirected to the actual content:

{% include lti_launch.html %}

## Sending scores (LTI Outcome)

{% include lti_send_scores.html %}
