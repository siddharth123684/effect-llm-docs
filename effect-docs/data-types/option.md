---
title: Option
description: Learn Option.some, Option.none, Option.map, Option.flatMap, Option.match, and Option.gen for modeling optional values and partial results when building Effect programs and services.
---

# Option

Source: extracted from `llms-full.txt` (`Option`).

## Overview

`Option<A>` models optional values with two cases:

- `Option.some(value)`: value is present
- `Option.none()`: value is absent

Common uses include partial function results, optional fields, and optional arguments.

## Creating Options

Use constructors directly or derive an `Option` from a predicate:

```ts
import { Option } from "effect"

const a = Option.some(1)
const b = Option.none<number>()
const parsePositive = Option.liftPredicate((n: number) => n > 0)
```

## Modeling Optional Properties

Use `Option` for values that may be missing while keeping object keys structurally present:

```ts
import { Option } from "effect"

interface User {
  readonly id: number
  readonly email: Option.Option<string>
}
```

`email` always exists as a key; its value is `Some` or `None`.

## Guards and Pattern Matching

- `Option.isSome` / `Option.isNone` narrow the variant.
- `Option.match` handles both branches in one expression.

```ts
import { Option } from "effect"

const message = Option.match(Option.some(1), {
  onNone: () => "empty",
  onSome: (n) => `value: ${n}`
})
```

## Transforming and Chaining

- `Option.map`: transform `Some`, keep `None` unchanged
- `Option.flatMap`: chain computations returning `Option`
- `Option.filter`: keep `Some` only when predicate passes

```ts
import { Option } from "effect"

const result = Option.some(2).pipe(
  Option.map((n) => n * 2),
  Option.filter((n) => n > 2)
)
```

## Getting Values

- `Option.getOrThrow`: extract `Some`, throw on `None`
- `Option.getOrElse`: provide fallback for `None`
- `Option.getOrNull` / `Option.getOrUndefined`: convert to nullable values

```ts
import { Option } from "effect"

const n1 = Option.getOrElse(Option.some(5), () => 0) // 5
const n2 = Option.getOrElse(Option.none<number>(), () => 0) // 0
```

## Fallback

- `Option.orElse`: try an alternative `Option` computation if current is `None`
- `Option.firstSomeOf`: get the first `Some` from an iterable

```ts
import { Option } from "effect"

const first = Option.firstSomeOf([Option.none(), Option.some(2), Option.some(3)])
```

## Interop with Nullable Types

Use `Option.fromNullable` to convert `null | undefined` into `Option`:

```ts
import { Option } from "effect"

const a = Option.fromNullable(null) // None
const b = Option.fromNullable(1) // Some(1)
```

Convert back with `Option.getOrNull` or `Option.getOrUndefined`.

## Interop with Effect

`Option` works with many `Effect` combinators:

- `None` behaves like `Effect<never, NoSuchElementException>`
- `Some<A>` behaves like `Effect<A>`

This lets you combine `Option` and `Effect` values in the same pipelines.

## Combining Options

- `Option.zipWith`: combine two `Option` values using a function
- `Option.all`: combine tuple/struct/iterable of `Option` values while preserving shape

If any input is `None`, combined results are `None`.

## Generator Syntax

`Option.gen` provides generator-based composition. It short-circuits on the first `None`.

```ts
import { Option } from "effect"

const person = Option.gen(function* () {
  const name = yield* Option.some("John")
  const age = yield* Option.some(25)
  return { name, age }
})
```

Keep `Option.gen` computations pure; avoid side effects inside the generator.

## Equivalence and Ordering

- `Option.getEquivalence`: compare `Option` values using an inner `Equivalence`
- `Option.getOrder`: sort `Option` values using an inner `Order` (`None` sorts lower by default)
