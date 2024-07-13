---
layout: page
title: "Architecture Decisions: App & Code"
nav_order: 1000
parent: Backend
---

# Architecture Decisions: App & Code


## The environment is forced to `test` in the tests, can never be `test` when running the application, and the `test` environment only read its configuration from `config.yaml.test`, and never from `config.yaml`

Date: 05/06/2024

Why?
- We want to make sure tests are never run on a live database because they erase all the data.

More info and discussion: https://github.com/France-ioi/AlgoreaBackend/pull/1085


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

## Panic/recover error handling
Date: 05/03/2019

Besides the usual way of error handling via returning an error value from a function and checking it in the caller, Go has a special way of handling errors. It is called panic and recover. The panic function is used to raise a runtime error. When a function encounters a panic, it stops executing and unwinds the stack. The deferred functions are executed and then the program terminates. The recover function is used to catch a panic and resume normal execution. It is used in a deferred function. The recover function returns the value that was passed to the panic function. If the function is not called in a deferred function, it returns nil.

The technique of handling errors via panic/recover very convenient. It allows us to get rid of error handling in many places and concentrate it in one place. It is widely used in our code.

However, it is not a good idea to return errors via panic from a public method or a public function in a package. Only the package itself can recover from a panic. All panics that are not recovered by the package are propagated to the caller and considered as a bug/unexpected behavior. As it is written in "Effective Go": "_Useful though this pattern is, it should be used only within a package. ... That is a good rule to follow._" (See https://go.dev/doc/effective_go#recover)

For the reasons mentioned above, it was decided not to return errors via panic from public methods/functions in our code. Instead, we should return an error value from a public function/method and check it in the caller.

As an exception it was decided to allow special public methods/functions prefixed with 'Must' to return errors via panic if their parameter is an error. The 'Must' prefix is a common convention in Go to indicate that the function panics if an error occurs. It is used in the standard library, for example, in the 'template' package. With this prefix the caller always knows that the function panics if an error occurs. You can find some functions named 'MustNotBeError' in our packages.
