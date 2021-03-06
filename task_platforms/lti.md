---
layout: page
title: LTI Workflows
nav_order: 80
parent: Interactions with Other Platforms
---

# LTI

Learning Tools Interoperability (LTI) is a method for a learning systems to invoke and to communicate with other systems. This is done using OAuth2, OpenID Connect, and JWT. In LTI, a system can act as a consumer (consuming external resources) or provider.

Currently, our backend acts a LTI **provider**

## Authentication (LTI Launch Request)

When our platform acts as a LTI provider, the LTI consumer will typically call our infrastructure with "LTI Launch Request" providing LTI parameters. These can be handled by the login-module which can verify them and create a local user if needed.

Our current two use cases are:
* A platform using our items in an embedded iframe. The tasks and chapters are loaded via a POST request which is redirected to the content to be displayed. The score can also be sent back to the platform (not detailed here).
* A LTI-enabled website redirects their users to our platforms so that the user can utilize our platform without further authentication.

In both cases, the LTI Launch request is a POST with LTI authentication parameters, which can be redirected to the actual content. In the following workflow, the first case includes a "content_id", the second one does not.

{% include lti_launch.html %}

## Sending scores (LTI Outcome)

If a content_id has been provided with the launch request, the score of the item can be sent back the LTI consumer, with the following workflow:

{% include lti_send_scores.html %}
