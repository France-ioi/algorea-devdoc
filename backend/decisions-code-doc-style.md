---
layout: page
title: "Architecture Decisions: Doc & Code Style"
nav_order: 1000
parent: Backend
---

# Architecture Decisions: Code Style

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
