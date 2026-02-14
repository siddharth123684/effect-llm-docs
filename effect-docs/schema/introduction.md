---
title: Introduction
description: effect/Schema for describing data shapes and transforming values between runtime formats.
---

# Introduction

Source: extracted from `llms-full.txt` (`Introduction`).

## Overview

`effect/Schema` provides a `Schema<Type, Encoded, Requirements>` model for
describing data shapes and safely transforming values between runtime formats.

With a schema, you can:

- Decode (`Encoded -> Type`)
- Encode (`Type -> Encoded`)
- Assert runtime values against the schema type
- Generate Standard Schema, JSON Schema, Arbitrary, Equivalence, and Pretty printers

## Requirements

- TypeScript `5.4+`
- `strict: true` in `tsconfig.json`
- Optional but recommended: `exactOptionalPropertyTypes: true`

```json
{
  "compilerOptions": {
    "strict": true,
    "exactOptionalPropertyTypes": true
  }
}
```

`exactOptionalPropertyTypes` improves alignment between static types and runtime
decoding behavior for optional fields. Without it, optional properties can widen
to include `undefined` at the type level even when decoding rejects that value.

## The Schema Type

```text
Schema<Type, Encoded, Requirements>
```

- `Type`: decoded output type
- `Encoded`: encoded/input representation (defaults to `Type`)
- `Requirements`: context required by schema operations (defaults to `never`)

Examples:

- `Schema<string>` is equivalent to `Schema<string, string, never>`
- `Schema<number, string>` decodes from `string` to `number` and encodes back

## Understanding Schema Values

- Schemas are immutable values.
- They describe data; they do not perform effects by themselves.
- Different compilers interpret a schema into operations such as decode, encode,
  pretty printing, arbitrary generation, and more.

## Decoding and Encoding

- **Decoding** validates/parses external input into your domain type.
- **Encoding** converts domain values into external representations.
- **Decoding from unknown** first checks structure, then transforms.
- **Encoding from unknown** first checks structure, then transforms.

This is especially useful when data crosses boundaries (forms, APIs, storage)
and runtime input does not already match your domain model.

## Rule of Schemas

A practical rule: encoding and then decoding should preserve the original value.
Design schemas so round trips remain consistent and predictable.
