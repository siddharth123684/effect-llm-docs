---
title: Advanced Usage
description: Custom data types, brands, optional/required properties, extending schemas, and recursion.
---

# Advanced Usage

Source: extracted from `llms-full.txt` (`Advanced Usage`).

## Overview

This section covers advanced schema patterns:

- declaring custom data types with `Schema.declare`
- using brands for nominal typing
- controlling optional/required property behavior
- extending and renaming schema shapes
- modeling recursive and mutually recursive data

## Declaring New Data Types

### Primitive and custom declarations

Use `Schema.declare` with a type guard for values that do not have a built-in schema.

```ts
import { Schema } from "effect"

const FileFromSelf = Schema.declare(
  (input: unknown): input is File => input instanceof File,
  {
    identifier: "FileFromSelf",
    description: "The JavaScript File type"
  }
)
```

### Type constructors

`Schema.declare` also supports generic constructors (for example `ReadonlySet<A>`) by providing:

- dependency schemas (for example `[item]`)
- custom `decode` and `encode` logic
- optional annotations such as `description`

Important limitation: declaration `decode` / `encode` handlers are synchronous and cannot use `R` requirements or async effects.

### Compiler annotations

When using compilers such as `Arbitrary` or `Pretty`, declared schemas often require explicit annotations (for example `arbitrary`) or compiler generation will fail with missing-annotation errors.

## Branded Types

Effect supports nominal typing on top of TypeScript structural typing.

- Create a brand directly with `Schema.brand(...)`
- Reuse an existing `Brand` constructor with `Schema.fromBrand(...)`
- Use unique symbols as brands when cross-module uniqueness matters
- Use `MyBrandSchema.make(...)` to construct branded values with the default constructor

```ts
import { Schema } from "effect"

const UserId = Schema.String.pipe(Schema.brand("UserId"))
const userId = UserId.make("123")
```

## Property Signatures

`PropertySignature` models transformations between encoded input fields and typed output fields.

Common tools:

- `Schema.propertySignature(...)` for explicit field transformations
- `Schema.fromKey("SOURCE_NAME")` to map encoded key names
- `field.from` to access the underlying, pre-optional schema

## Optional Fields

### Core APIs

- `Schema.optional(schema)`
- `Schema.optionalWith(schema, options)`

### `optionalWith` behavior cheatsheet

| Options | Missing | `undefined` | `null` | Output style |
| --- | --- | --- | --- | --- |
| none | accepted | accepted | error | optional field |
| `{ nullable: true }` | accepted | accepted | treated as missing | optional field |
| `{ exact: true }` | accepted | error | error | optional field |
| `{ exact: true, nullable: true }` | accepted | error | treated as missing | optional field |
| `{ as: "Option" }` | `Option.none()` | `Option.none()` | error | required `Option<A>` |
| `{ as: "Option", nullable: true }` | `Option.none()` | `Option.none()` | `Option.none()` | required `Option<A>` |
| `{ as: "Option", exact: true }` | `Option.none()` | error | error | required `Option<A>` |
| `{ as: "Option", exact: true, nullable: true }` | `Option.none()` | error | `Option.none()` | required `Option<A>` |

### Defaults with `optionalWith`

`default: () => A` fills missing values during decoding and also affects constructor behavior (`make`).

- default only: applies to missing and `undefined`
- with `exact: true`: applies only when the field is missing
- with `nullable: true`: also applies when input is `null`
- with both `exact` and `nullable`: applies for missing or `null`, but not `undefined`

### Optional `never` and tsconfig

To model `quantity?: never`, behavior depends on `exactOptionalPropertyTypes`:

- `false`: `Schema.optional(Schema.Never)` (allows `undefined`)
- `true`: `Schema.optionalWith(Schema.Never, { exact: true })` (strict absence)

## Optional-field Transformation Primitives

### `Schema.optionalToOptional`

Transforms optional input to optional output with `Option`-based decode/encode hooks.
Useful for dropping values (for example treating empty string as absent).

### `Schema.optionalToRequired`

Transforms optional input into required output by providing a fallback in decode logic (for example map missing to `null`).

### `Schema.requiredToOptional`

Transforms required input into optional output by returning `Option.none()` in decode logic for values to omit.

## Extending Schemas

### Spreading struct fields

Use `...Struct.fields` to extend structs while preserving struct ergonomics.

```ts
import { Schema } from "effect"

const Base = Schema.Struct({ a: Schema.String })
const Extended = Schema.Struct({ ...Base.fields, b: Schema.Number })
```

### `Schema.extend`

Use `Schema.extend(left, right)` for combinations that cannot be expressed by simple field spreading (for example struct + union of structs).

Key points:

- extension support depends on schema compatibility
- overlapping keys with incompatible types fail
- useful for combining refinements and composed struct-like shapes

## Renaming Properties

- During definition: rename encoded keys with `Schema.fromKey(...)`
- After definition: rename across an existing schema with `Schema.rename(schema, mapping)`

`Schema.rename` works on complex schemas (including unions), but on structs it may lose original field-level struct typing helpers.

## Recursive Schemas

Use `Schema.suspend(() => SchemaRef)` for self-recursive or mutually recursive schemas.

```ts
import { Schema } from "effect"

interface Category {
  readonly name: string
  readonly subcategories: ReadonlyArray<Category>
}

const Category: Schema.Schema<Category> = Schema.Struct({
  name: Schema.String,
  subcategories: Schema.Array(
    Schema.suspend((): Schema.Schema<Category> => Category)
  )
})
```

When recursive fields contain transformations (different `Type` vs `Encoded`, such as `NumberFromString`), define both interfaces and annotate recursion as `Schema.Schema<Type, Encoded>`.
