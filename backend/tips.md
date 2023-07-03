---
layout: page
title: Tips
nav_order: 600
parent: Backend
---

# Tips

## When a 64bits integer, like an ID, seems to be cut as a 32bits integer

Whenever you have an `int64` in the response in JSON, you **MUST** specify `string` for the serialization:

```
type canRequestHelpTo struct {
  ID   int64  `json:"id,string"`
  ...
}
```

This will ensure that the number is put in quotes in the response: `id:"XXX"`.
Otherwise, the resulting JSON would be: `id:XXX`. This causes errors as the result would be cut into a 32bits integer
when it is unserialized, both in Javascript and with Go.
