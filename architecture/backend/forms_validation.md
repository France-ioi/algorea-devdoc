---
layout: page
title: Forms and Validation
nav_order: 300
parent: Backend
---

# Parsing, validation and persistence

Here's the process to retrieve user data from a POST request, validate it, and save it into the database:

```
type datatypeRequest struct {
	field int `json:"json_label" validate:"validator1_name=validator1_value,validator2_name=validator2_value"`
}

input := datatypeRequest{}
formData := formdata.NewFormData(&input)

// Add custom validators and translations here...

err = formData.ParseJSONRequestData(r)
if err != nil {
  return service.ErrInvalidRequest(err)
}

// Get a map that can be used to make a SQL update
updateData := formData.ConstructMapForDB()

// Update...
record.UpdateColumn(updateData)
```

## Validation

**Important:** The validator package is currently stuck at version 9 in a [forked repository](https://github.com/France-ioi/validator/). We need to find out why and how we can instead depend on the lastest version.


### Fields validation

To specify validators for fields, we use *struct tags* on those fields.

Example to validate that an integer is >= 0
```
type datatype struct {
	field int `json:"field" validate:"gte=0"`
}
```

The validation is then checked with `formData.ParseJSONRequestData(r)`, which returns an error in case any validator fails.

See the link in "Resources" to find the list of validators implemented by the validator package.


### Custom validation

In case you have a validation that doesn't fit one of the standard ones, you can create a custom validator:

```
type datatype struct {
	Status string `json:"status" validate:"status_must_be_closed"`
}

formData.RegisterValidation("status_must_be_closed", statusMustBeClosed)

func statusMustBeClosed(statusField validator.FieldLevel) bool {
  status := statusField.Field().String()
	return status == "closed"
}
```

The validators use the *Reflection API* which is a little tricky. Check the custom validators already implemented or the [standard validators implementation](https://github.com/France-ioi/validator/blob/v9/baked_in.go) for examples.

**Warning / Current limitation**: You cannot generate a validation error on a field that is not provided in the input, otherwise it will be ignored. Which means, if a field is required but not set, you have to generate the error on a field that is provided.


### Arguments for validator

If you need to access some arguments in your custom validators, you can use the following pattern:

```
type datatype struct {
	Status string `json:"status" validate:"helper_group_id_set_if_non_open_to_open_status"`
}

formData.RegisterValidation("helper_group_id_set_if_non_open_to_open_status",	constructHelperGroupIDSetIfNonOpenToOpenStatus(oldThread.Status))

func constructHelperGroupIDSetIfNonOpenToOpenStatus(oldStatus string) validator.Func {
	...
	return func(fl validator.FieldLevel) bool {
	  // oldStatus is available in the validator function
	  ...
		return true
	}
}
```


### Optional arguments / default value

You might need to know whether an optional argument to a service has been provided or if the zero value have been provided. (e.g. specifying "message_count": 0 might be different from not providing "message_count" at all). In this case, use a pointer:

```
type datatype struct {
	MessageCount *int `json:"message_count" validate:"omitempty,gte=0,exclude_increment_if_message_count_set"`
}

formData.RegisterValidation("exclude_increment_if_message_count_set", excludeIncrementIfMessageCountSetValidator)
```

Note that when it is a pointer, and you need to access its value from a custom validator, you need to access it from the **top** of the structure:

```
func excludeIncrementIfMessageCountSetValidator(messageCountField validator.FieldLevel) bool {
  messageCountPtr := messageCountField.Top().Elem().FieldByName("MessageCount").Interface().(*int)

  if messageCountPtr == nil {
    ...
  }
}
```


### Validator translations

You can specify a custom error message for a validator with:

```
formData.RegisterTranslation("tag", "the value should be positive!")
```

`tag` is the tag used in field's *struct tags*. It can be a custom validator like `exclude_increment_if_set`, or a standard one like `gte=0`.


## Resources

* [validator v9 Documentation](https://pkg.go.dev/gopkg.in/go-playground/validator.v9)
* [List of validators](https://pkg.go.dev/gopkg.in/go-playground/validator.v9#pkg-overview)
