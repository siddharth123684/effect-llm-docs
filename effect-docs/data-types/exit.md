---
title: Exit
description: Learn Schema.Exit and Schema.ExitFromSelf for JSON-serializable conversion of Exit values with failure, success, and defect schemas across boundaries in Effect Schema.
---

# Exit

Source: extracted from `llms-full.txt` (`Exit`).

## Overview

`Schema.Exit` converts `Exit` values to and from a JSON-serializable representation.
It is useful when an `Exit` must cross process or network boundaries while preserving success and failure structure.

## Schema.Exit

`Schema.Exit` requires schemas for:

- `failure`: typed failure values
- `success`: success values
- `defect`: unrecoverable defect values

```ts
import { Schema } from "effect"

const schema = Schema.Exit({
  failure: Schema.String,
  success: Schema.NumberFromString,
  defect: Schema.String
})
```

### Decoding Behavior

- `{ _tag: "Failure", cause: CauseEncoded<FI, DI> }` -> `Exit.failCause(Cause<FA>)`
- `{ _tag: "Success", value: SI }` -> `Exit.succeed(SA)`

Inner values are transformed by the provided `failure`, `success`, and `defect` schemas.

### Encoding Behavior

- `Exit.failCause(Cause<FA>)` -> `{ _tag: "Failure", cause: CauseEncoded<FI, DI> }`
- `Exit.succeed(SA)` -> `{ _tag: "Success", value: SI }`

## Handling Defects in Serialization

Use `Schema.Defect` when defects may include JavaScript `Error` instances.

- On decode, it reconstructs `Error`-like values from plain objects (for example with `name`, `message`, `stack`).
- On encode, it converts `Error` instances into plain serializable objects with essential fields.

```ts
import { Exit, Schema } from "effect"

const schema = Schema.Exit({
  failure: Schema.String,
  success: Schema.NumberFromString,
  defect: Schema.Defect
})

const encode = Schema.encodeSync(schema)
const decode = Schema.decodeSync(schema)

encode(Exit.die(new Error("Message")))
decode({ _tag: "Failure", cause: { _tag: "Die", defect: { name: "Error", message: "Message" } } })
```

## Schema.ExitFromSelf

`Schema.ExitFromSelf` is for values that are already `Exit`-shaped.
It decodes and encodes by transforming only the inner failure/success/defect payloads.

```ts
import { Exit, Schema } from "effect"

const schema = Schema.ExitFromSelf({
  failure: Schema.String,
  success: Schema.NumberFromString,
  defect: Schema.String
})

const decode = Schema.decodeUnknownSync(schema)
const encode = Schema.encodeSync(schema)

decode(Exit.succeed("1")) // Exit.succeed(1)
encode(Exit.succeed(1)) // Exit.succeed("1")
```
