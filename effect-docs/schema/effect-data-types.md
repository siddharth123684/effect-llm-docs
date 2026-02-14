---
title: Effect Data Types
description: Schema adapters for Data, Config, Option, Either, Exit, collections, Duration, and Redacted.
---

# Effect Data Types

Source: extracted from `llms-full.txt` (`Effect Data Types`).

## Overview

Effect Schema includes adapters for Effect-native data types so values can move
between runtime structures and serializable representations while preserving
type safety.

This subsection covers:

- `Schema.Data` for structural equality/hash semantics
- `Schema.Config` for schema-driven config decoding
- `Option`, `Either`, and `Exit` schema families
- `ReadonlySet` / `ReadonlyMap` and hash/sorted collection schemas
- `Duration` schemas and duration clamps
- `Redacted` schemas for sensitive values

## Interop With Data

`Schema.Struct(...)` values do not automatically implement `Equal` and `Hash`.
Wrap with `Schema.Data(...)` to enable value-based equality on decoded values.

```ts
import { Equal, Schema } from "effect"

const Person = Schema.Data(
  Schema.Struct({
    name: Schema.String,
    age: Schema.Number
  })
)

const decode = Schema.decodeUnknownSync(Person)
Equal.equals(decode({ name: "Alice", age: 30 }), decode({ name: "Alice", age: 30 }))
// true
```

## Config

`Schema.Config(name, schema)` builds a `Config<A>` from a schema that decodes
from `string` input (`I extends string`).

- looks up the config value by `name`
- decodes using the provided schema
- formats parse failures with tree-style schema errors

```ts
import { Effect, Schema } from "effect"

const Port = Schema.Config("PORT", Schema.NumberFromString)

Effect.runSync(
  Effect.gen(function* () {
    const port = yield* Port
    console.log(port)
  })
)
```

## Option Schemas

- `Schema.Option(inner)`: JSON-tagged encoding (`{ _tag: "None" | "Some", ... }`)
- `Schema.OptionFromSelf(inner)`: works with `Option` on both encoded/decoded sides
- `Schema.OptionFromUndefinedOr(inner)`: `undefined <-> Option.none()`
- `Schema.OptionFromNullOr(inner)`: `null <-> Option.none()`
- `Schema.OptionFromNullishOr(inner, onNoneEncoding)`: null/undefined to none,
  with configurable none encoding
- `Schema.OptionFromNonEmptyTrimmedString`: empty/whitespace -> `Option.none()`

## Either Schemas

- `Schema.Either({ left, right })`: tagged JSON encoding (`Left` / `Right`)
- `Schema.EitherFromSelf({ left, right })`: `Either` on both sides
- `Schema.EitherFromUnion({ left, right })`: raw union input to `Either`

Use these when external payloads and in-memory `Either` values use different
representations.

## Exit Schemas

- `Schema.Exit({ failure, success, defect })`: serializable form of `Exit`
- `Schema.ExitFromSelf({ failure, success, defect })`: transforms inner values
  while preserving `Exit` representation
- `Schema.Defect` can be used for defect serialization/deserialization of
  JavaScript `Error` values

## Set and Map Schemas

- `Schema.ReadonlySet(inner)` and `Schema.ReadonlySetFromSelf(inner)`
- `Schema.ReadonlyMap({ key, value })`
- `Schema.ReadonlyMapFromSelf({ key, value })`
- `Schema.ReadonlyMapFromRecord({ key, value })` for object-record encoding

Key idea: non-self variants typically encode as arrays/records for transport,
self variants keep the same runtime container type on both sides.

## Hash and Sorted Collections

- `Schema.HashSet(inner)` / `Schema.HashSetFromSelf(inner)`
- `Schema.HashMap({ key, value })` / `Schema.HashMapFromSelf({ key, value })`
- `Schema.SortedSet(inner, order)` / `Schema.SortedSetFromSelf(inner, ...)`

`SortedSet` schemas require order definitions so values remain ordered during
decode/encode.

## Duration Schemas

- `Schema.Duration`: hrtime tuple (`[seconds, nanos]`) to `Duration`
- `Schema.DurationFromSelf`: validates existing `Duration` values
- `Schema.DurationFromMillis`: `number` milliseconds to `Duration`
- `Schema.DurationFromNanos`: `bigint` nanoseconds to `Duration`
- `Schema.clampDuration(min, max)`: bounds duration values after decoding

## Redacted Schemas

- `Schema.Redacted(inner)` converts an encoded value (commonly `string`) into
  `Redacted<...>` so output stays obscured (`<redacted>`)
- `Schema.RedactedFromSelf(inner)` validates existing `Redacted` values

When composing `Redacted` with refinements/transformations, parse errors can
still leak sensitive raw input unless error messages are customized. Prefer
redacted-safe messages in annotations for secret-bearing schemas.
