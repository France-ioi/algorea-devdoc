---
layout: page
title: "Architecture Decisions: Services & Code Style"
nav_order: 1000
parent: Backend
---

# Architecture Decisions: Services & Code Style


## When a propagation is scheduled, it is scheduled at the end of the current transaction

Date: 18/04/2024

Why?
- ???

## Propagations are executed in steps, and up to one database transaction can be executed per step

Date: 18/04/2024

Why?
- Depending on what is affected, a propagation can be very long (> 15min). The issue we have is that AWS Lambda has a maximum execution time of 15min. We want to make sure that if we reach the maximum execution time, at least some of the work has been commited. Otherwise, we would have a process that can never finish.
- We also want to avoid having a transaction that is too long, that holds many locks, and can cause deadlocks.

## Fields that are not visible to the user due to access rights must not appear in the response at all

Date: 12/09/2023

Why?
- We want to be able to distinguish between `null` values and values that are not visible to the user.

Examples:
- The name is visible and has a value: `{"id": 1, "name": "John"}`
- The name is visible but is set to `null`: `{"id": 1, "name": null}`
- The name is not visible: `{"id": 1}`


## Swagger documentation for Responses and Requests must always specify `Nullable` and `required: true` when applicable

Date: 31/08/2023

- When a field is always present (response) or must always be given (request), it must have a `// required:true` annotation. Otherwise, no annotation.
- When a field can have the value `null`, for both responses and requests, it must have a `// Nullable` annotation.

**The single source of truth for Nullable is the database.**

Example:
```
type item struct {
	// required:true
	ID int64 `json:"id,string"`
	// required:true
	// Nullable
	// enum: Chapter,Task,Skill
	Type *string `json:"type"`
	// required:true
	// Nullable
	Title *string `json:"title"`
	// required:true
	LanguageTag string `json:"language_tag"`
}
```

Why?
- Because it has an influence on how the frontend or other third-party treat the responses.


## End Of Line must be LF

Date: 22/03/2023

Why?
- go-fmt automatically changes the line endings of *.go files with LF. Make it LF everywhere for consistency.
