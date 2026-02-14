# Arbitrary

Source: extracted from `llms-full.txt` (`Schema to Arbitrary`).

## Overview

`Arbitrary.make(schema)` builds a `fast-check` arbitrary from a `Schema<A, I, R>`.
This is useful for property-based tests because generated values follow schema constraints.

```ts
import { Arbitrary, FastCheck, Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.NonEmptyString,
  age: Schema.Int.pipe(Schema.between(1, 80))
})

const arb = Arbitrary.make(Person)
console.log(FastCheck.sample(arb, 2))
```

The full `fast-check` API is available through the `FastCheck` export from `effect`.

## How Constraints Affect Generation

Arbitrary derivation maps schema filters to matching `fast-check` primitives when possible.
For example:

```ts
Schema.Int.pipe(Schema.between(1, 80))
```

is generated with behavior equivalent to:

```ts
FastCheck.integer({ min: 1, max: 80 })
```

Conflicting or overly strict filters can make generation stall because valid values become hard (or impossible) to produce.

## Prefer `Schema.pattern` for String Regex Rules

For regex-based string constraints, prefer `Schema.pattern(...)` over generic custom filters.
It enables efficient generation via `FastCheck.stringMatching(...)`.

```ts
import { Schema } from "effect"

const Good = Schema.String.pipe(Schema.pattern(/^[a-z]+$/))
```

When multiple patterns are present, generation uses a union pattern so each alternative can be sampled.

## Transformations and Filter Ordering

During arbitrary generation, filters applied before the last transformation are ignored.
Place constraints that must be respected on the final transformed schema.

```ts
import { Arbitrary, FastCheck, Schema } from "effect"

const schema1 = Schema.compose(Schema.NonEmptyString, Schema.Trim).pipe(
  Schema.maxLength(500)
)
// May include empty strings because the non-empty filter is before the transformation
console.log(FastCheck.sample(Arbitrary.make(schema1), 2))

const schema2 = Schema.Trim.pipe(
  Schema.nonEmptyString(),
  Schema.maxLength(500)
)
// Respects final constraints
console.log(FastCheck.sample(Arbitrary.make(schema2), 2))
```

Practical order:

1. Define filters for the input shape.
2. Apply transformations.
3. Apply filters for the final output shape.

## Custom Arbitrary with Annotations

Use the `arbitrary` annotation when you want custom sample generation (or when derivation for a custom type cannot be inferred).

```ts
import { Arbitrary, FastCheck, Schema } from "effect"

const Name = Schema.NonEmptyString.annotations({
  arbitrary: () => (fc) =>
    fc.constantFrom("Alice Johnson", "Dante Howell", "Marta Reyes")
})

const Person = Schema.Struct({
  name: Name,
  age: Schema.Int.pipe(Schema.between(1, 80))
})

console.log(FastCheck.sample(Arbitrary.make(Person), 2))
```

The callback receives `fc` (the full `fast-check` export), so you can return any compatible arbitrary.

## Integrating Faker (Optional)

You can combine `fast-check` with `@faker-js/faker` for realistic generated values.

```ts
import { Arbitrary, FastCheck, Schema } from "effect"
import { faker } from "@faker-js/faker"

const Name = Schema.NonEmptyString.annotations({
  arbitrary: () => (fc) =>
    fc.constant(null).map(() => faker.person.fullName())
})

const Person = Schema.Struct({
  name: Name,
  age: Schema.Int.pipe(Schema.between(1, 80))
})

console.log(FastCheck.sample(Arbitrary.make(Person), 2))
```
