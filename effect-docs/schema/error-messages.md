# Error Messages

Source: extracted from `llms-full.txt` (`Error Messages`).

## Overview

`Schema` parsing emits structured `ParseError` values. By default, messages are
derived from schema shape and failure location (for example expected type, missing
fields, failed refinements, or transformation failures).

Use `errors: "all"` in decode options to keep multiple failures in one report.

## Default Error Messages

Default messages include:

- type mismatch (`Expected string, actual null`)
- missing properties/tuple elements (`is missing`)
- nested path context (`["field"]`, `[0]`, etc.)

```ts
import { Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.String,
  age: Schema.Number
})

Schema.decodeUnknownSync(Person)({}, { errors: "all" })
// ParseError with both missing properties: "name" and "age"
```

## Improving Readability with Annotations

For complex schemas, add identifiers to shorten and clarify output:

- `identifier`
- `title`
- `description`

`identifier` is especially useful for replacing long inferred type displays with
domain names like `Person`, `Name`, or `Age`.

## Refinement and Transformation Failure Kinds

Refinement errors distinguish where failure happened:

- **From side refinement failure**: input does not match the base schema
- **Predicate refinement failure**: base type matches, predicate fails

Transformation errors can occur on:

- **Encoded side transformation failure**: input does not satisfy source side
- **Transformation process failure**: custom conversion logic failed
- **Type side transformation failure**: converted output fails target schema

## Custom Messages with `message`

Attach custom messages via annotations. A message function can return:

- `string`
- `Effect<string>`
- `{ message: string | Effect<string>, override: boolean }`

General selection rules:

1. If no custom messages exist, default innermost failure message is used.
2. With custom messages, the first failed schema message is used (inner -> outer).
3. `override: true` forces that custom message to replace nested/default messages.

```ts
import { Schema } from "effect"

const Value = Schema.Union(Schema.String, Schema.Number).annotations({
  message: () => ({ message: "Please provide a string or a number", override: true })
})

Schema.decodeUnknownSync(Value)(null)
// ParseError: Please provide a string or a number
```

## Effectful Messages

Messages can be effectful (`Effect<string>`) so they can depend on services
(for example i18n dictionaries) and still participate in schema error rendering.

## Missing Value Messages with `missingMessage`

Use `missingMessage` for required properties or tuple elements that are absent.

```ts
import { Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.propertySignature(Schema.String).annotations({
    missingMessage: () => "Name is required"
  })
})

Schema.decodeUnknownSync(Person)({})
// ParseError at ["name"]: Name is required
```
