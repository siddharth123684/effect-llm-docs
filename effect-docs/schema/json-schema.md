---
title: JSON Schema
description: JSONSchema.make for generating JSON Schema Draft 07 documents from Effect schemas.
---

# JSON Schema

Source: extracted from `llms-full.txt` (`JSON Schema`).

## Overview

Use `JSONSchema.make` to generate JSON Schema documents from Effect schemas.

- Primary API: `JSONSchema.make(schema, options?)`
- Default output target: JSON Schema Draft 07
- Generation focuses on the input side of decoding
- Refinements are included, but generation stops at the first transformation

```ts
import { JSONSchema, Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.String,
  age: Schema.Number
})

const jsonSchema = JSONSchema.make(Person)
```

## Target Versions

Set the target format with the `target` option:

- `"jsonSchema7"` (default)
- `"jsonSchema2019-09"`
- `"jsonSchema2020-12"`
- `"openApi3.1"`

```ts
import { JSONSchema, Schema } from "effect"

const schema = Schema.Tuple(Schema.String, Schema.Number)

const jsonSchema = JSONSchema.make(schema, {
  target: "jsonSchema2020-12"
})
```

For tuples, generated shape changes by target. For example, Draft 07 uses `items` and `additionalItems`, while 2020-12 uses `prefixItems` and `items`.

## Common Schema Mappings

Effect schema forms map to JSON Schema patterns:

- literals -> `enum`
- generic unions -> `anyOf`
- unions of literals -> `enum`
- structs -> object schema with `required`, `properties`, and `additionalProperties: false`
- records -> object schema with `patternProperties`
- template literals -> string schema with `pattern`

## Identifier Annotations and `$defs`

Annotate schemas with `identifier` to emit reusable definitions under `$defs` and reference them with `$ref`.

This is especially useful for large schemas and is required for recursive or mutually recursive schemas.

```ts
import { JSONSchema, Schema } from "effect"

const Name = Schema.String.annotations({ identifier: "Name" })
const Age = Schema.Number.annotations({ identifier: "Age" })

const Person = Schema.Struct({
  name: Name,
  age: Age
})

const jsonSchema = JSONSchema.make(Person)
```

## Standard Metadata Annotations

The following annotations are emitted into generated JSON Schema when present:

- `title`
- `description`
- `default`
- `examples`

For struct fields, prefer annotating property signatures so metadata is attached to each property rather than only to a shared type.

## Custom `jsonSchema` Annotations

`jsonSchema` annotations let you customize generation behavior:

- For unsupported types (for example `bigint`), add `jsonSchema` to avoid missing-annotation errors.
- For refinements (`Schema.filter`), provided `jsonSchema` fragments are merged.
- For non-refinement schema nodes, a `jsonSchema` annotation replaces the default generated output for that node.

```ts
import { JSONSchema, Schema } from "effect"

const Positive = Schema.Number.pipe(
  Schema.filter((n) => n > 0, { jsonSchema: { minimum: 0 } }),
  Schema.filter((n) => n <= 10, { jsonSchema: { maximum: 10 } })
)

const jsonSchema = JSONSchema.make(Positive)
```

## `Schema.parseJson` Behavior

`Schema.parseJson(inner)` has specialized JSON Schema generation behavior.

Instead of emitting only the outer "from string" transformation shape, generation reflects the provided inner schema structure. Nested `parseJson` fields are represented as JSON strings with `contentMediaType: "application/json"` where appropriate.
