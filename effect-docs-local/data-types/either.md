# Either

Source: extracted from `llms-full.txt` (`Either`).

## Overview

`Either` represents one of two mutually exclusive cases:

- `Left`: commonly used for failure or alternative paths
- `Right`: commonly used for success values

Use it as a lightweight discriminated union when you need to model branching values explicitly.

## Either vs Exit

`Either` is useful for simple branching, but `Exit` is the preferred Effect result type when you need richer failure details.

`Exit` can represent:

- expected errors
- defects
- interruptions
- composed failure causes

## Creating Values

Use constructors:

- `Either.right(value)`
- `Either.left(value)`

```ts
import { Either } from "effect"

const ok = Either.right(42)
const err = Either.left("not a number")
```

## Guards and Pattern Matching

Use guards to inspect cases:

- `Either.isLeft`
- `Either.isRight`

Use `Either.match` to handle both branches in one place.

```ts
import { Either } from "effect"

const value = Either.right(42)

const message = Either.match(value, {
  onLeft: (left) => `Left: ${left}`,
  onRight: (right) => `Right: ${right}`
})
```

## Mapping

- `Either.map`: transform only `Right`
- `Either.mapLeft`: transform only `Left`
- `Either.mapBoth`: transform both sides with separate handlers

```ts
import { Either } from "effect"

const a = Either.map(Either.right(1), (n) => n + 1) // Right(2)
const b = Either.mapLeft(Either.left("bad"), (s) => s + "!") // Left("bad!")
```

## Combining Eithers

### zipWith

`Either.zipWith` combines two `Either` values with a function. If either input is `Left`, the result is `Left`.

### all

`Either.all` combines multiple `Either` values and preserves input shape:

- tuple input -> tuple output
- struct input -> struct output
- iterable input -> array output

If any input is `Left`, the first encountered `Left` is returned.

## Generator Syntax

`Either.gen` provides generator-based composition similar to `async/await` style.

When a yielded value is `Left`, evaluation short-circuits and returns that `Left`.

```ts
import { Either } from "effect"

const result = Either.gen(function* () {
  const name = yield* Either.right("John")
  const age = yield* Either.right(25)
  return { name, age }
})
```

## Interop with Effect

`Either` can be used with many `Effect` combinators:

- `Left<L>` behaves like a failure (`Effect<never, L>`)
- `Right<R>` behaves like a success (`Effect<R>`)

This makes it straightforward to combine `Either` and `Effect` values in the same pipeline.
