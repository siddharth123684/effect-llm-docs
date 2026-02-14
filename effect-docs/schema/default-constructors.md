# Default Constructors

Source: extracted from `llms-full.txt` (`Default Constructors`).

## Overview

Default constructors let you create values that conform to a schema via `schema.make(...)`.
They are available for common schema forms such as `Struct`, `Record`, filters/refinements, and branded schemas.

Important behavior:

- constructors operate on the **decoded type** (`A`), not the encoded type (`I`)
- constructors are **unsafe** by default and throw `ParseError` when input is invalid

## Constructor Scope: Decoded Type

For `Schema<A, I, R>`, `make` expects `A`-shaped input. If a schema transforms values during decoding (for example `string -> number`), `make` still expects the post-decode type.

```ts
import { Schema } from "effect"

const schema = Schema.NumberFromString.pipe(Schema.between(1, 10))

schema.make(5)  // ok
schema.make(20) // throws ParseError (out of range)
```

## Using `make` Across Schema Kinds

### Structs

```ts
import { Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.NonEmptyString
})

Person.make({ name: "Alice" }) // ok
```

### Records

```ts
import { Schema } from "effect"

const Tags = Schema.Record({
  key: Schema.String,
  value: Schema.NonEmptyString
})

Tags.make({ a: "x", b: "y" }) // ok
```

### Filters and Brands

Refinements (for example `between`) and branded schemas also expose `make`. They validate before returning and throw on invalid values.

## Disabling Validation

`make` supports an option to bypass validation:

```ts
schema.make(value, { disableValidation: true })
```

Use this only in controlled cases (for example test setup) because it skips runtime guarantees.

## Safe Alternative: `Schema.validateEither`

When you want constructor-like behavior without throwing, use `Schema.validateEither(schema)`.
It returns `Either<ParseError, A>` rather than raising an exception.

```ts
import { Schema } from "effect"

const schema = Schema.NumberFromString.pipe(Schema.between(1, 10))
const safeMake = Schema.validateEither(schema)

safeMake(5)  // Right(5)
safeMake(20) // Left(ParseError)
```

## Field Defaults with `Schema.withConstructorDefault`

Apply defaults to property signatures so fields become optional for constructors.

```ts
import { Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.NonEmptyString,
  age: Schema.Number.pipe(
    Schema.propertySignature,
    Schema.withConstructorDefault(() => 0)
  )
})

Person.make({ name: "John" }) // { name: "John", age: 0 }
```

## Nested Defaults Are Shallow

Defaults do not automatically propagate through nested structs at the top level.
If nested defaults are needed, use the nested field constructor explicitly:

```ts
const { web: Web } = Config.fields
Config.make({ web: Web.make({ application_port: 3000 }) })
```

## Default Evaluation and Reuse

- default factories are lazily evaluated (`() => ...` runs per construction)
- default-bearing property signatures are portable when reused in other schemas
- defaults also work with `Schema.Class(...)` class-based schemas
