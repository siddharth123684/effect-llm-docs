# Annotations

Source: extracted from `llms-full.txt` (`Schema Annotations`).

## Overview

Schema annotations are metadata attached to schema AST nodes (`annotations: Record<string | symbol, unknown>`).
They let you customize validation errors, documentation output, JSON Schema generation, and compiler behavior.

You can add metadata with:

- the instance method: `.annotations({...})`
- the API form: `Schema.annotations({...})`

## Basic Usage

```ts
import { Schema } from "effect"

const Password = Schema.String
  .annotations({ message: () => "not a string" })
  .pipe(
    Schema.nonEmptyString({ message: () => "required" }),
    Schema.maxLength(10, { message: (issue) => `${issue.actual} is too long` })
  )
  .annotations({
    identifier: "Password",
    title: "password",
    description: "A password used to authenticate a user",
    examples: ["1Ki77y", "jelly22fi$h"],
    documentation: "Internal password constraints docs"
  })
```

## Common Built-in Annotations

Common keys include:

- `identifier`: stable schema identity (useful for references and tooling)
- `title`, `description`, `documentation`: human-readable docs metadata
- `examples`, `default`: usage examples and defaults
- `message`: custom parse error messages
- `jsonSchema`: JSON Schema-specific metadata
- `arbitrary`, `pretty`, `equivalence`: compiler-specific configuration
- `parseIssueTitle`, `parseOptions`: parse reporting/behavior customization
- `concurrency`, `batching`: execution behavior hints for nested/effectful parsing
- `decodingFallback`: recover from decoding failure with custom fallback logic

## Concurrency Annotation

For complex schemas with nested checks (for example `Struct`, `Array`, `Union`), `concurrency` controls parallelism:

```ts
type ConcurrencyAnnotation = number | "unbounded" | "inherit" | undefined
```

- `number`: maximum concurrent tasks
- `"unbounded"`: no limit
- `"inherit"`: inherit parent setting
- `undefined`: sequential execution (default)

## Decoding Fallback Annotation

`decodingFallback` provides recovery logic when decode fails:

```ts
import { Schema } from "effect"
import { Either } from "effect"

const WithFallback = Schema.String.annotations({
  decodingFallback: () => Either.right("<fallback>")
})
```

This lets decoding return a fallback value instead of failing for selected cases.

## Custom Annotations

You can define your own annotation keys with symbols:

```ts
import { Schema, SchemaAST } from "effect"
import { Option } from "effect"

const DeprecatedId = Symbol.for("app/schema/deprecated")

const MyString = Schema.String.annotations({ [DeprecatedId]: true })

const isDeprecated = <A, I, R>(schema: Schema.Schema<A, I, R>): boolean =>
  SchemaAST.getAnnotation<boolean>(DeprecatedId)(schema.ast).pipe(
    Option.getOrElse(() => false)
  )
```

For type safety, add a module augmentation for `effect/Schema` and declare the custom key in `Annotations.GenericSchema`.
