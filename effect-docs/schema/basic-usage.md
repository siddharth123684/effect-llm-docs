---
title: Basic Usage
description: Learn Schema constructors, decodeUnknownSync, encodeSync, validateSync, and primitive types including Literal, TemplateLiteral, Struct, and Record for defining and validating data shapes.
---

# Basic Usage

Source: extracted from `llms-full.txt` (`Basic Usage`).

## Overview

`Schema` constructors define typed codecs (`Type`, `Encoded`, `Context`) used to
validate, decode, and encode data.

Common runtime entry points:

- `Schema.decodeUnknownSync(schema)(input)` to decode unknown input
- `Schema.encodeSync(schema)(value)` to encode typed values
- `Schema.validateSync(schema)(input)` to validate without changing shape

## Primitives and Basic Constructors

| Constructor | Type |
| --- | --- |
| `Schema.String` | `string` |
| `Schema.Number` | `number` |
| `Schema.Boolean` | `boolean` |
| `Schema.BigIntFromSelf` | `bigint` |
| `Schema.SymbolFromSelf` | `symbol` |
| `Schema.Object` | `object` |
| `Schema.Undefined` | `undefined` |
| `Schema.Void` | `void` |
| `Schema.Any` | `any` |
| `Schema.Unknown` | `unknown` |
| `Schema.Never` | `never` |

`Schema.asSchema` expands compact schema types into `Schema<Type, Encoded, Context>`.

`Schema.UniqueSymbolFromSelf(symbol)` creates a schema for one specific symbol.

## Literals and Template Literals

- `Schema.Literal(...)` builds literal schemas (single value or union of values).
- `schema.literals` exposes literal values as a readonly tuple.
- `Schema.pickLiteral(...)` narrows a literal set into a subtype.
- `Schema.TemplateLiteral(...)` validates string shapes built from spans.
- `Schema.TemplateLiteralParser(...)` validates and parses into tuple output.

Supported template-literal spans include:

- `Schema.String`
- `Schema.Number`
- literals (`string | number | boolean | null | bigint`)
- unions/brands of the above

```ts
import { Schema } from "effect"

const Code = Schema.Literal("a", "b", "c")
const Narrowed = Code.pipe(Schema.pickLiteral("a", "b"))

const Parsed = Schema.TemplateLiteralParser(
  Schema.NumberFromString,
  "a",
  Schema.NonEmptyString
)

Schema.decodeSync(Parsed)("100afoo") // [100, "a", "foo"]
```

## Enums and Unions

- `Schema.Enums(MyEnum)` creates enum schemas; use `schema.enums` to inspect members.
- `Schema.Union(...)` evaluates members in declaration order (first successful decode wins).
- For nullable forms:
  - `Schema.NullOr(schema)`
  - `Schema.UndefinedOr(schema)`
  - `Schema.NullishOr(schema)`
- `schema.members` exposes union members as a tuple.

For discriminated unions, use literal discriminator fields (for example `_tag` or
`kind`). `Schema.attachPropertySignature(name, literal)` is a concise way to add
discriminants to union members.

## Tuples and Arrays

Tuple APIs:

- `Schema.Tuple(...)` for required elements
- `Schema.optionalElement(...)` for optional tuple positions
- rest elements by passing tuple elements plus rest schema
- `schema.elements` / `schema.rest` expose tuple parts

Array APIs:

- `Schema.Array(valueSchema)`
- `Schema.NonEmptyArray(valueSchema)`
- `Schema.mutable(schema)` for shallow mutable output types
- `schema.value` exposes array element schema

## Records

`Schema.Record({ key, value })` supports:

- string keys
- symbol keys
- union/literal/template-literal keys
- refined key schemas (for example `minLength`)

Important behavior: key refinements are filters by default. Invalid keys are
removed from decoded output unless parse options require errors
(`onExcessProperty: "error"`).

Key transformations are not supported directly in `Schema.Record`; do key
normalization with an outer `Schema.transform(...)`.

`Schema.mutable(...)` creates shallow mutable record types. Use `schema.key` and
`schema.value` to inspect record key/value schemas.

## Structs and Tagged Structs

`Schema.Struct(fields, ...indexSignatures)` defines object schemas with fixed
properties and optional index signatures.

Rules and accessors:

- at most one string index signature and one symbol index signature
- conflicting fixed-property and index-signature value types can cause TypeScript
  conflicts; split and compose with transformations if needed
- `schema.fields` and `schema.records` expose fixed/indexed parts
- `Schema.mutable(...)` makes shallow mutable structs

Tagging helpers:

- `Schema.tag("User")` creates a literal property signature
- `Schema.TaggedStruct("User", fields)` creates a struct with `_tag`
- `make(...)` can auto-apply tags; decoding unknown input still expects required tags
  unless defaults are configured explicitly

## Class and Object Utilities

- `Schema.instanceOf(MyClass)` validates class instances (public constructor required).
- For private constructors, use `Schema.declare(...)` with a type guard.
- Add field-level checks to instances with `Schema.filter(...)`.

Struct/object reshaping utilities:

- `MyStruct.pick(...)` / `Schema.pick(...)`
- `MyStruct.omit(...)` / `Schema.omit(...)`
- `Schema.partial(schema)` (adds optional + `undefined`)
- `Schema.partialWith(schema, { exact: true })` (optional without widening to `undefined`)
- `Schema.required(schema)` (forces optional properties to required)
- `Schema.keyof(schema)` (schema of object keys)
