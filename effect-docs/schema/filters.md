---
title: Filters
description: Schema.filter for validation rules that refine accepted values without changing inferred type.
---

# Filters

Source: extracted from `llms-full.txt` (`Filters`).

## Overview

`Schema.filter` adds validation rules on top of an existing schema.
It refines which values are accepted without changing the schema's inferred `Type`.

If you need to change the inferred `Type`, use a transformation or a branded type approach.

## Declaring Filters

Use `Schema.filter` in a schema pipeline and return either success or an issue from the predicate.

```ts
import { Schema } from "effect"

const LongString = Schema.String.pipe(
  Schema.filter((s) => s.length >= 10 || "a string at least 10 characters long")
)

Schema.decodeUnknownSync(LongString)("short")
/*
throws:
ParseError: { string | filter }
*/
```

## Predicate Shape and Return Semantics

A filter predicate receives:

- the current value
- parse options
- the refinement AST node (`self`)

The predicate can return:

- `true` or `undefined`: validation passes
- `false`: validation fails with a default predicate failure
- `string`: validation fails with a custom message
- `ParseResult.ParseIssue`: validation fails with a structured issue
- `FilterIssue`: validation fails with a path-aware issue
- `ReadonlyArray<FilterOutput>`: reports multiple issues at once

## Adding Annotations

You can attach metadata (for diagnostics and JSON Schema output) when defining a filter.

```ts
import { JSONSchema, Schema } from "effect"

const LongString = Schema.String.pipe(
  Schema.filter(
    (s) => (s.length >= 10 ? undefined : "a string at least 10 characters long"),
    {
      identifier: "LongString",
      description: "A string with minimum length 10",
      jsonSchema: { minLength: 10 }
    }
  )
)

JSONSchema.make(LongString)
```

## Path-specific and Multiple Errors

Filters can assign issues to specific paths, which is useful for form-like structures.

```ts
import { Schema } from "effect"

const Password = Schema.Trim.pipe(Schema.minLength(2))

const MyForm = Schema.Struct({
  password: Password,
  confirm_password: Password
}).pipe(
  Schema.filter((input) =>
    input.password === input.confirm_password
      ? undefined
      : {
          path: ["confirm_password"],
          message: "Passwords do not match"
        }
  )
)
```

To report several problems in one pass, return an array of issues from the predicate.

## Exposed Base Schema

For a filtered schema, `.from` exposes the schema before refinement:

```ts
import { Schema } from "effect"

const LongString = Schema.String.pipe(Schema.filter((s) => s.length >= 10))
const Base = LongString.from
```

## Built-in Filters

Common built-in filter families include:

- String: `minLength`, `maxLength`, `length`, `pattern`, `startsWith`, `endsWith`, `includes`, `trimmed`, `lowercased`, `uppercased`, `capitalized`
- Number: `greaterThan`, `greaterThanOrEqualTo`, `lessThan`, `lessThanOrEqualTo`, `between`, `int`, `finite`, `positive`, `nonNegative`, `multipleOf`
- ReadonlyArray: `minItems`, `maxItems`, `itemsCount`
- Date: `validDate`, `greaterThanDate`, `greaterThanOrEqualToDate`, `lessThanDate`, `betweenDate`
- BigInt: `greaterThanBigInt`, `greaterThanOrEqualToBigInt`, `lessThanBigInt`, `betweenBigInt`, positive/negative variants
- BigDecimal: `greaterThanBigDecimal`, `lessThanBigDecimal`, `betweenBigDecimal`, positive/negative variants
- Duration: `greaterThanDuration`, `greaterThanOrEqualToDuration`, `lessThanDuration`, `betweenDuration`

## Effectful Filters

`Schema.filter` handles synchronous checks.
For async or effectful checks, use `Schema.filterEffect` (documented under Transformations).
