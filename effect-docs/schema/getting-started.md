# Getting Started

Source: extracted from `llms-full.txt` (`Getting Started`).

## Overview

`effect/Schema` lets you describe data shapes once and then:

- decode external input into trusted domain values
- encode domain values back to transport formats
- validate, assert, and refine runtime inputs
- control parsing behavior with configurable options

You can import Schema either as a namespace or from `effect`:

```ts
import * as Schema from "effect/Schema"
// or
import { Schema } from "effect"
```

## Defining a First Schema

Use `Schema.Struct` to describe object shapes:

```ts
import { Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.String,
  age: Schema.NumberFromString
})
```

This defines a schema where:

- decoded `Type` is `{ readonly name: string; readonly age: number }`
- encoded `Encoded` is `{ readonly name: string; readonly age: string }`

## Extracting Inferred Types

From a schema `Schema<Type, Encoded, Context>`, you can extract:

- `Schema.Schema.Type<typeof S>` (or `typeof S.Type`)
- `Schema.Schema.Encoded<typeof S>` (or `typeof S.Encoded`)
- `Schema.Schema.Context<typeof S>` (or `typeof S.Context`)

These helpers keep domain types and wire-format types aligned with the schema definition.

## Decoding Unknown Input

Common decoding entry points:

- `Schema.decodeUnknownSync(schema)` -> throws on failure
- `Schema.decodeUnknownEither(schema)` -> `Either<ParseError, A>`
- `Schema.decodeUnknown(schema)` -> `Effect<A, ParseError, R>` (useful for async transforms)

```ts
import { Either, Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.String,
  age: Schema.Number
})

const decodeEither = Schema.decodeUnknownEither(Person)
const result = decodeEither({ name: "Alice", age: 30 })

if (Either.isRight(result)) {
  console.log(result.right) // { name: "Alice", age: 30 }
}
```

Use the Effect-based decoder when schema transformations perform async work.

## Encoding Values

Encoding APIs mirror decoding:

- `Schema.encodeSync(schema)`
- `Schema.encodeEither(schema)`
- `Schema.encode(schema)` (Effect-based)

```ts
import { Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.NonEmptyString,
  age: Schema.NumberFromString
})

console.log(Schema.encodeSync(Person)({ name: "Alice", age: 30 }))
// { name: "Alice", age: "30" }
```

## Parse Options

Important parse options for decode/encode operations:

- `onExcessProperty`:
  - `"ignore"` (default) drops unknown keys
  - `"error"` fails on unknown keys
  - `"preserve"` keeps unknown keys
- `errors`:
  - `"first"` (default behavior)
  - `"all"` to collect all parse issues
- `propertyOrder`:
  - `"none"` (implementation-defined order)
  - `"original"` (preserve input key order)
- `exact`:
  - controls missing-property strictness in parsing

## Type Guards and Assertions

- `Schema.is(schema)` returns a type guard (`u is A`)
- `Schema.asserts(schema)` returns an assertion function and throws detailed `ParseError` on failure

These are useful when you need runtime checks that also improve TypeScript narrowing.

## Naming Convention Quick Rule

Schema names usually indicate their encode/decode behavior:

- JSON-compatible directly named schemas (for example `Schema.Number`)
- conversion-oriented schemas for transformed formats (for example `Schema.NumberFromString`, `Schema.DateFromSelf`)

This naming pattern makes transport conversions explicit in schema definitions.
