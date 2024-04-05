---
layout: page
title: JWS Tokens
nav_order: 500
parent: Backend
---

# JWS Tokens

The backend uses JWS tokens (encrypted data structures using public/private keys) to allow transmission of data between servers by a client, while protecting this data from alteration.

## Create a new type of token

First, define your token data structure in `app/payloads/yourtype_token.go`:

```
// ThreadToken represents data inside a thread token.
type ThreadToken struct {
	Date          string `json:"date" validate:"dmy-date"` // dd-mm-yyyy
	...
	ItemID        string `json:"item_id"`
	...

	PublicKey  *rsa.PublicKey
	PrivateKey *rsa.PrivateKey
}
```

`Date` is required by the token system. When the token is encrypted, it will automatically add the current date `time.Now` in this field.
Additionally, when a token is decrypted by the backend, `Date` must be between yesterday and tomorrow, otherwise the token will be rejected. This must be taken care of when testing tokens.

Note: use `string` to store `int` values, otherwise you'll run into type issues.

Next, define the methods to marshall/unmarshal your token in `app/token/`. Use the existing examples.

To get a token from the defined data structure:

```
  	threadToken, err := (&token.Thread{
		ItemID:        strconv.FormatInt(itemID, 10),
		...
	}).Sign(srv.TokenConfig.PrivateKey)
```

## Testing tokens

### Unit tests

For unit tests, a specific public and private key defined in `tokentest/sample_keys.go` are used.

To generate a token using those keys, you can use the function `generateSignedTestToken` defined in `token/token_test.go`.
It will allow you to create a token with the information you want inside.

Whenever you decrypt a token in your test (unmarshal), be sure to first pass the `time.Now`, otherwise it will get rejected because the date doesn`t match:

```
  monkey.Patch(time.Now, func() time.Time { return timeWhenTheTokenWasGenerated })
```

### Integration tests (Gherkin)

Since different keys are used in the local and CircleCI test environments, the integrations can't test the encrypted tokens directly.

Instead, you need to use the following Gherkin feature:

```
And the response body decoded as "ThreadTokenResponse" should be, in JSON:
    """
    {
      "participant_id": 1,
      ...
      "token": {
        "can_watch": false,
        ...
      }
    }
    """
```

This allows you to test directly the decrypted content of the token.

Note that the type `ThreadTokenResponse` has to be defined. The is currently done in `testhelpers/known_types.go`:

```
  type threadGetResponse struct {
    ItemID        int64  `json:"item_id"`
    ...

    ThreadToken token.Thread `json:"token"`

    PublicKey *rsa.PublicKey
  }

  type threadGetResponseWrapper struct {
    ItemID        int64  `json:"item_id"`
    ...

    ThreadToken *string `json:"token"`
  }

  func (resp *threadGetResponse) UnmarshalJSON(raw []byte) error {
    wrapper := threadGetResponseWrapper{}
    if err := json.Unmarshal(raw, &wrapper); err != nil {
      return err
    }

    resp.ItemID = wrapper.ItemID
    ...

    if wrapper.ThreadToken != nil {
      resp.ThreadToken.PublicKey = resp.PublicKey
      return (&resp.ThreadToken).UnmarshalString(*wrapper.ThreadToken)
    }

    return nil
  }
  ...
  var knownTypes = map[string]reflect.Type{
	...
	  "ThreadTokenResponse":       reflect.TypeOf(&threadGetResponse{}).Elem(),
	}
```

Note the type difference between `threadGetResponse` and `threadGetResponseWrapper`.
The first one contains the decrypted data, while the second one contains it as a string token.

All the other fields between those two structures are common. It would be a good idea to define them once instead of duplicating them, but this is currently not possible due to a limitation in `app/payloads/common.go:ConvertIntoMap`.
It should also be possible to remove the `PublicKey` from `threadGetResponse` to make the design clearer.
