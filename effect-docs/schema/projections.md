---
title: Projections
description: Learn how to use Schema.typeSchema and Schema.encodedSchema to derive Type or Encoded views from existing schemas and reuse structure without transformation layers.
---

# Projections

Source: extracted from `llms-full.txt` (`Schema Projections`).

## Overview

Projections let you derive a new schema from an existing one by focusing on one
representation:

- `Type` side (`A`)
- `Encoded` side (`I`)

These helpers are useful when you want to reuse structure but drop specific
transformation layers.

## `Schema.typeSchema`

`Schema.typeSchema` extracts the `Type` view of a schema.

- Keeps type-side structure and constraints.
- Drops encode/decode transformation behavior from the original schema chain.

```ts
import { Schema } from "effect"

const Original = Schema.Struct({
  quantity: Schema.NumberFromString.pipe(Schema.greaterThanOrEqualTo(2))
})

const TypeSchema = Schema.typeSchema(Original)
```

In this example, `quantity` is projected as a `number` with the `>= 2`
constraint.

## `Schema.encodedSchema`

`Schema.encodedSchema` extracts the encoded/input-facing view of a schema.

- Keeps encoded-side structure.
- Omits refinements and transformations from the original schema.

```ts
import { Schema } from "effect"

const Original = Schema.Struct({
  quantity: Schema.String.pipe(Schema.minLength(3))
})

const Encoded = Schema.encodedSchema(Original)
```

In this example, `quantity` is projected as plain `string` (without `minLength`).

## `Schema.encodedBoundSchema`

`Schema.encodedBoundSchema` is similar to `encodedSchema`, but it preserves
refinements up to the first transformation boundary.

- Keeps initial validations before the first transform/compose boundary.
- Omits transformations after that boundary.

```ts
import { Schema } from "effect"

const Original = Schema.Struct({
  foo: Schema.String.pipe(
    Schema.minLength(3),
    Schema.compose(Schema.Trim)
  )
})

const EncodedBound = Schema.encodedBoundSchema(Original)
```

In this projection, `foo` keeps `minLength(3)` but does not include the trim
transformation.

## Quick Selection Guide

- Use `typeSchema` when you need the decoded/type-side schema.
- Use `encodedSchema` when you need encoded shape only, without refinements.
- Use `encodedBoundSchema` when you need encoded shape plus pre-transform
  refinements.
