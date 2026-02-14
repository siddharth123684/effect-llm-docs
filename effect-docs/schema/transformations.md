# Transformations

Source: extracted from `llms-full.txt` (`Transformations`).

## Overview

Transformations connect schemas so values can be converted between encoded and decoded forms.
They are used when validation alone is not enough and data must be reshaped (for example string -> number, string -> branded id, array -> set).

Core APIs:

- `Schema.transform` for transformations that are expected to succeed
- `Schema.transformOrFail` for transformations that can fail
- `Schema.compose` to chain existing schemas
- `Schema.filterEffect` for effectful validation-style transformations

## `Schema.transform`

Use `Schema.transform(from, to, options)` to connect a source schema and target schema with:

- `decode`: converts source output into target input
- `encode`: converts target output back into source input

When decoding, the flow is `SourceEncoded -> TargetType`.
When encoding, the flow is `TargetType -> SourceEncoded`.

By default, `strict: true` enforces tighter type compatibility between conversion steps. Use `strict: false` only when needed for intentional, safe widening.

```ts
import { Schema } from "effect"

const BooleanFromString = Schema.transform(
  Schema.Literal("on", "off"),
  Schema.Boolean,
  {
    strict: true,
    decode: (s) => s === "on",
    encode: (b) => (b ? "on" : "off")
  }
)
```

## `Schema.transformOrFail`

Use `Schema.transformOrFail` when decode/encode can fail.

- return `ParseResult.succeed(value)` on success
- return `ParseResult.fail(issue)` on failure

Decode/encode callbacks can receive:

- the value being transformed
- parse options from the caller
- the transformation `ast` (useful for precise errors)

```ts
import { ParseResult, Schema } from "effect"

const NumberFromString = Schema.transformOrFail(
  Schema.String,
  Schema.Number,
  {
    strict: true,
    decode: (input, _options, ast) => {
      const n = Number(input)
      return Number.isNaN(n)
        ? ParseResult.fail(new ParseResult.Type(ast, input, "Invalid number"))
        : ParseResult.succeed(n)
    },
    encode: (n) => ParseResult.succeed(String(n))
  }
)
```

## Async Transformations and Requirements

`Schema.transformOrFail` also supports asynchronous decode/encode by returning an `Effect`.
If decode/encode needs services, those dependencies are tracked in the schema context (`Schema<Type, Encoded, Requirements>`), and must be provided when running decode/encode effects.

## One-Way Transformations

If reversing a transformation should never happen, return `ParseResult.Forbidden` from `encode`.
This is useful for flows like password hashing, where decode hashes plain text and encode is intentionally blocked.

## Composition and Effectful Filters

- `Schema.compose(schema1, schema2)` chains transformations into one schema.
- `strict: false` can relax composition type matching when necessary.
- `Schema.filterEffect` is for effectful validation checks (for example external service checks) while staying in schema pipelines.

## Built-in Transformation Schemas

### String

- `Schema.split(delimiter)`
- `Schema.Trim`
- `Schema.Lowercase`, `Schema.Uppercase`
- `Schema.Capitalize`, `Schema.Uncapitalize`
- `Schema.parseJson([schema])`
- `Schema.StringFromBase64`, `Schema.StringFromBase64Url`
- `Schema.StringFromHex`
- `Schema.StringFromUriComponent`

### Number

- `Schema.NumberFromString`
- `Schema.clamp(min, max)`
- `Schema.parseNumber`

### Boolean

- `Schema.Not`

### Symbol

- `Schema.Symbol` (string <-> symbol via `Symbol.for`)

### BigInt

- `Schema.BigInt`
- `Schema.BigIntFromNumber`
- `Schema.clampBigInt(min, max)`

### Date

- `Schema.Date` (string <-> valid `Date`)

### BigDecimal

- `Schema.BigDecimal`
- `Schema.BigDecimalFromNumber`
- `Schema.clampBigDecimal(min, max)`
