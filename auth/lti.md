---
layout: page
title: LTI Auth Workflow
nav_order: 80
parent: Authentication & Authorization
---

# LTI Authentication Workflow

When our platform acts as a LTI provider, the LTI consumer will typically embed one of our items in an iframe, by sending a LTI launch request with a set of LTI parameters to authenticate the user. The workflow is the following:

{% include lti.html %}

Note also, that if the user goes on the frontend outside the LTI context, the frontend should detect it and log the user out.
