# Pattern Matching

Source: extracted from `llms-full.txt` (`Pattern Matching`).

## Overview

Pattern matching in Effect is provided by the `Match` module. It helps branch on values in a concise, type-safe way and supports exhaustiveness checking so missing cases are caught at compile time.

Compared to imperative `if/else` or `switch` chains, `Match` is often clearer for unions and nested conditions.

## Core workflow

Pattern matching usually follows three steps:

1. Create a matcher with `Match.type<T>()` or `Match.value(value)`.
2. Add patterns with combinators like `Match.when`, `Match.not`, and `Match.tag`.
3. Finalize with `Match.exhaustive`, `Match.orElse`, `Match.option`, or `Match.either`.

## Creating a matcher

### `Match.type<T>()`

Use `Match.type<T>()` to define a matcher over a type (commonly a union).

```ts
import { Match } from "effect"

const describe = Match.type<string | number>().pipe(
  Match.when(Match.number, (n) => `number: ${n}`),
  Match.when(Match.string, (s) => `string: ${s}`),
  Match.exhaustive
)
```

### `Match.value(value)`

Use `Match.value(value)` when you already have a concrete value and want to match against it directly.

```ts
import { Match } from "effect"

const input = { name: "John", age: 30 }

const result = Match.value(input).pipe(
  Match.when({ name: "John" }, (user) => `${user.name} is ${user.age} years old`),
  Match.orElse(() => "Oh, not John")
)
```

### `Match.withReturnType<T>()`

Use `Match.withReturnType<T>()` to force all branches to return the same type.  
Important: place it first in the pipeline so TypeScript can enforce consistency.

## Pattern combinators

### `Match.when`

Defines match cases using literal values, object patterns, or predicates.

### `Match.not`

Matches everything except the specified value or pattern.

### `Match.tag`

Matches discriminated unions by `_tag`. You can match one or multiple tags in a single clause.

```ts
import { Match } from "effect"

type Event =
  | { readonly _tag: "fetch" }
  | { readonly _tag: "success"; readonly data: string }
  | { readonly _tag: "error"; readonly error: Error }
  | { readonly _tag: "cancel" }

const render = Match.type<Event>().pipe(
  Match.tag("fetch", "success", () => "Ok!"),
  Match.tag("error", (event) => `Error: ${event.error.message}`),
  Match.tag("cancel", () => "Cancelled"),
  Match.exhaustive
)
```

`Match.tag` assumes the discriminant field is named `_tag` (Effect convention).

### Built-in predicates

Common predicates include:

- `Match.string`, `Match.nonEmptyString`
- `Match.number`, `Match.bigint`
- `Match.boolean`
- `Match.symbol`
- `Match.date`
- `Match.record`
- `Match.null`, `Match.undefined`, `Match.defined`
- `Match.any`
- `Match.is(...values)`
- `Match.instanceOf(Class)`

## Finalizers

### `Match.exhaustive`

Requires all cases to be handled. TypeScript reports an error if any union case is missing.

### `Match.orElse`

Provides a fallback branch when no case matches.

### `Match.option`

Returns an `Option`: `Some(value)` on match, `None` on no match.

### `Match.either`

Returns an `Either`: `Right(value)` on match, `Left(originalInput)` on no match.

## Practical guidance

- Prefer `Match.exhaustive` for closed unions to prevent missing branches.
- Use `Match.orElse` when a default outcome is valid.
- Use `Match.option` / `Match.either` when you want unmatched results represented explicitly.
