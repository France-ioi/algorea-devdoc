---
layout: page
title: Asynchronous Exports
nav_order: 90
has_children: true
permalink: /exports/
---

# Asynchronous Exports

Some platform features produce large files (ZIPs, reports) that cannot be generated and returned in a single HTTP request on the backend Lambda (ALB response size and timeout limits). Those exports are built asynchronously: the user requests the job, is notified when it is ready, and downloads the file through a short-lived URL.

Currently documented:

- [Group results export]({{ site.baseurl }}{% link exports/group-results-export.md %}) — ZIP of a group's progress and answers on a set of items
