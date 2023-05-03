---
layout: page
title: "Architecture Decisions: Code Style"
nav_order: 1000
parent: Backend
---

# Architecture Decisions: Code Style

## End Of Line must be LF

Date: 22/03/2023

Why?
- go-fmt automatically changes the line endings of *.go files with LF. Make it LF everywhere for consistency.
