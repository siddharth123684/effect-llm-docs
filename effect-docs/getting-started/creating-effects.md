---
title: Creating Effects
description: Learn Effect constructors including Effect.succeed, Effect.fail, Effect.sync, Effect.promise, and Effect.defer to model sync, async, and deferred computations when building Effect programs.
---

# Creating Effects

Source: extracted from `llms-full.txt` (`Creating Effects`).

## Overview

Effect constructors let you model computations explicitly:

- success values
- expected failures
- synchronous side effects
- asynchronous side effects
- deferred/lazy effect creation

Using constructors instead of throwing exceptions keeps failure behavior visible in types.

## Why Not Throw Errors?

Thrown exceptions are not represented in standard function signatures, so callers cannot see failure modes from types alone.

Effect models this explicitly with constructors like:

- `Effect.succeed(value)` for success
- `Effect.fail(error)` for expected failure

## Core Constructors

### `Effect.succeed`

Creates an effect that always succeeds.

```ts
import { Effect } from "effect"

const success = Effect.succeed(42)
// Effect<number, never, never>
```

### `Effect.fail`

Creates an effect that fails with an expected error.

```ts
import { Data, Effect } from "effect"

class HttpError extends Data.TaggedError("HttpError")<{}> {}

const failure = Effect.fail(new HttpError())
// Effect<never, HttpError, never>
```

Tagged errors (objects/classes with `_tag`) work well with error-matching operators such as `Effect.catchTag`.

## Modeling Synchronous Computations

### `Effect.sync`

Use for synchronous side effects that are expected not to throw.
If the thunk throws, the failure is treated as a defect (unexpected error channel).

```ts
import { Effect } from "effect"

const log = (message: string) =>
  Effect.sync(() => {
    console.log(message)
  })
```

### `Effect.try`

Use for synchronous logic that can throw.

```ts
import { Effect } from "effect"

const parseJson = (input: string) =>
  Effect.try({
    try: () => JSON.parse(input),
    catch: (unknown) => new Error(`Invalid JSON: ${String(unknown)}`)
  })
```

Without a custom `catch`, errors are captured as `UnknownException`.

## Modeling Asynchronous Computations

### `Effect.promise`

Use when the underlying promise is expected to resolve successfully.
Rejected promises are treated as defects.

### `Effect.tryPromise`

Use when the promise may reject.
Rejected values are mapped to the typed error channel (`UnknownException` by default, or custom via overload).

```ts
import { Effect } from "effect"

const getTodo = (id: number) =>
  Effect.tryPromise({
    try: () => fetch(`https://jsonplaceholder.typicode.com/todos/${id}`),
    catch: (unknown) => new Error(`Request failed: ${String(unknown)}`)
  })
```

### `Effect.async`

Use to wrap callback-based APIs.

- Call `resume(...)` exactly once (extra calls are ignored).
- You may return a cleanup effect to run on interruption.
- The callback also receives an `AbortSignal` for interruption-aware APIs.

## Deferred Construction with `Effect.suspend`

`Effect.suspend(() => effect)` delays effect creation until execution time.

Common uses:

- lazy re-evaluation of effect creation
- preventing eager recursive expansion
- helping TypeScript unify branch return types

## Constructor Cheatsheet

| API                     | Input                              | Output                        |
| ----------------------- | ---------------------------------- | ----------------------------- |
| `Effect.succeed`        | `A`                                | `Effect<A>`                   |
| `Effect.fail`           | `E`                                | `Effect<never, E>`            |
| `Effect.sync`           | `() => A`                          | `Effect<A>`                   |
| `Effect.try`            | `() => A`                          | `Effect<A, UnknownException>` |
| `Effect.try` (overload) | `try: () => A`, `catch: u => E`    | `Effect<A, E>`                |
| `Effect.promise`        | `() => Promise<A>`                 | `Effect<A>`                   |
| `Effect.tryPromise`     | `() => Promise<A>`                 | `Effect<A, UnknownException>` |
| `Effect.tryPromise` (overload) | `try: () => Promise<A>`, `catch: u => E` | `Effect<A, E>` |
| `Effect.async`          | `(resume, signal?) => void`        | `Effect<A, E>`                |
| `Effect.suspend`        | `() => Effect<A, E, R>`            | `Effect<A, E, R>`             |

## Next Step

After creating effects, continue with runtime boundary execution in the `Running Effects` subsection.
