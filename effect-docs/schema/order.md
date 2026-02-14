---
title: Order
description: Schema propertyOrder for controlling decoded and encoded object key order (none vs original).
---

# Order

Source: extracted from `llms-full.txt` (`Order`).

## Overview

In Effect Schema, order is about how object properties are arranged in decoded / encoded output.
This behavior is controlled with the `propertyOrder` parse option.

## `propertyOrder` Modes

- `"none"` (default): the system chooses key order for parsing performance.
- `"original"`: keys are emitted in the same order they appear in the input.

When using `"none"`, key order is not guaranteed to be stable and should not be relied on.

## Synchronous Decoding

```ts
import { Schema } from "effect"

const schema = Schema.Struct({
  a: Schema.Number,
  b: Schema.Literal("b"),
  c: Schema.Number
})

Schema.decodeUnknownSync(schema)({ b: "b", c: 2, a: 1 })
// Output order is implementation-defined

Schema.decodeUnknownSync(schema)(
  { b: "b", c: 2, a: 1 },
  { propertyOrder: "original" }
)
// Output preserves: { b: "b", c: 2, a: 1 }
```

## Asynchronous Decoding

`propertyOrder` also applies to async decoding (`Schema.decode(...)`), including concurrent schemas.
Use the same parse option object:

```ts
import { Effect, Schema } from "effect"

const schema = Schema.Struct({
  a: Schema.Number,
  b: Schema.Number,
  c: Schema.Number
}).annotations({ concurrency: 3 })

Schema.decode(schema)({ a: 1, b: 2, c: 3 }, { propertyOrder: "original" })
  .pipe(Effect.runPromise)
```

## Practical Guidance

- Keep the default (`"none"`) when property order does not matter.
- Use `"original"` when downstream consumers depend on input key order.
