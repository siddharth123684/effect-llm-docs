---
title: Yieldable Errors
description: Custom Data.Error and Data.TaggedError for yielding failures inside Effect.gen.
---

# Yieldable Errors

Source: extracted from `llms-full.txt` (`Yieldable Errors`).

## Overview

Yieldable Errors are custom error values you can yield directly inside `Effect.gen`.

This lets you fail with domain errors without explicitly calling `Effect.fail` for every branch.

```ts
import { Data, Effect } from "effect"

class MyError extends Data.Error<{ message: string }> {}

const program = Effect.gen(function* () {
  yield* new MyError({ message: "Oh no!" })
})
```

Yielding `new MyError(...)` is equivalent to failing the effect with that error.

## `Data.Error`

Use `Data.Error` to define a base yieldable error class with a typed payload.

- good for simple custom errors
- preserves error structure in the typed error channel

## `Data.TaggedError`

Use `Data.TaggedError` when you want discriminated errors with a `_tag` for selective recovery.

```ts
import { Data, Effect } from "effect"

class FooError extends Data.TaggedError("Foo")<{ message: string }> {}
class BarError extends Data.TaggedError("Bar")<{ code: number }> {}

const recovered = Effect.gen(function* () {
  yield* new FooError({ message: "bad input" })
}).pipe(
  Effect.catchTags({
    Foo: (e) => Effect.succeed(`Foo: ${e.message}`),
    Bar: (e) => Effect.succeed(`Bar: ${e.code}`)
  })
)
```

`_tag`-based matching works well with `Effect.catchTag` and `Effect.catchTags`.

## Practical Guidance

- Use `Data.Error` for a single custom error type.
- Use `Data.TaggedError` for multiple error variants that need precise handling.
- Yieldable errors are most useful in generator-based workflows (`Effect.gen`).
