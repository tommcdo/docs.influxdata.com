---
title: string() function
description: The `string()` function converts a single value to a string.
menu:
  flux_0_50:
    name: string
    parent: built-in-type-conversions
weight: 2
aliases:
  - /flux/v0.50/functions/built-in/transformations/type-conversions/string/
---

The `string()` function converts a single value to a string.

_**Function type:** Type conversion_  
_**Output data type:** String_

```js
string(v: 123456789)
```

## Parameters

### v
The value to convert.

## Examples
```js
from(bucket: "sensor-data")
  |> range(start: -1m)
  |> filter(fn:(r) => r._measurement == "system" )
  |> map(fn:(r) => ({ r with model_number string(v: r.model_number) }))
```
