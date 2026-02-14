---
title: Error Channel Operations
description: Learn Effect.mapError, Effect.filterOrFail, Effect.tapError, and Effect.catchAll to transform, filter, and inspect failures in the error channel when building Effect programs.
---

# Error Channel Operations

Source: extracted from `llms-full.txt` (`Error Channel Operations`).

## Overview

Error channel operations let you:

- transform failure and success values without changing control flow
- choose what happens when post-conditions fail
- inspect failures for logging and diagnostics
- move failures into the success channel when you want explicit branching

## Transforming Channels

### `Effect.mapError`

Use `mapError` to convert the error type while keeping success values unchanged.

```ts
import { Effect } from "effect"

const task: Effect.Effect<number, string> = Effect.fail("Oh no!").pipe(Effect.as(1))
const mapped: Effect.Effect<number, Error> = task.pipe(
  Effect.mapError((message) => new Error(message))
)
```

### `Effect.mapBoth`

Use `mapBoth` when you need to transform both channels in one step.

```ts
import { Effect } from "effect"

const task: Effect.Effect<number, string> = Effect.fail("Oh no!").pipe(Effect.as(1))
const modified: Effect.Effect<boolean, Error> = task.pipe(
  Effect.mapBoth({
    onFailure: (message) => new Error(message),
    onSuccess: (n) => n > 0
  })
)
```

## Filtering the Success Channel

These operators validate success values and choose a failure strategy when the predicate is false.

| API | On predicate failure |
| --- | --- |
| `Effect.filterOrFail` | Fail with a typed error value |
| `Effect.filterOrDie` | Terminate with a defect |
| `Effect.filterOrDieMessage` | Terminate with a defect from a message |
| `Effect.filterOrElse` | Run an alternative effect |

Type guards work well here to narrow downstream types.

```ts
import { Effect } from "effect"

interface User {
  readonly name: string
}

declare const auth: () => Promise<User | null>

const program = Effect.promise(() => auth()).pipe(
  Effect.filterOrFail(
    (user): user is User => user !== null,
    () => new Error("Unauthorized")
  ),
  Effect.andThen((user) => user.name)
)
```

## Inspecting Failures Without Changing Outcome

Inspection operators are like `tap`, but focused on failure information.

| API | Purpose |
| --- | --- |
| `Effect.tapError` | Observe typed errors |
| `Effect.tapErrorTag` | Observe only one tagged error type |
| `Effect.tapErrorCause` | Observe full `Cause` (failures, defects, interrupts) |
| `Effect.tapDefect` | Observe defects only |
| `Effect.tapBoth` | Observe both success and failure branches |

```ts
import { Console, Data, Effect } from "effect"

class NetworkError extends Data.TaggedError("NetworkError")<{
  readonly statusCode: number
}> {}

const task: Effect.Effect<number, NetworkError> =
  Effect.fail(new NetworkError({ statusCode: 504 }))

const logged = task.pipe(
  Effect.tapErrorTag("NetworkError", (error) =>
    Console.log(`network status: ${error.statusCode}`)
  )
)
```

## Exposing Failures as Values

### `Effect.either`

Converts `Effect<A, E, R>` to `Effect<Either<A, E>, never, R>`. You branch on `Left`/`Right` instead of failing.

### `Effect.cause`

Converts an effect into one that succeeds with its full `Cause`, useful when you need detailed failure structure.

### `Effect.merge`

Merges error and success channels into success: `Effect<A | E, never, R>`.

### `Effect.flip`

Swaps channels: `Effect<A, E, R>` becomes `Effect<E, A, R>`.

```ts
import { Effect } from "effect"

const task: Effect.Effect<number, string> = Effect.fail("Oh uh!").pipe(Effect.as(2))

const asEither = Effect.either(task) // Effect<Either<number, string>, never, never>
const merged = Effect.merge(task) // Effect<number | string, never, never>
const flipped = Effect.flip(task) // Effect<string, number, never>
```

## Quick Selection

| Goal | API |
| --- | --- |
| Map only error values | `Effect.mapError` |
| Map success and error together | `Effect.mapBoth` |
| Reject bad success values with typed failures | `Effect.filterOrFail` |
| Observe failures for logging/metrics | `Effect.tapError*` / `Effect.tapBoth` |
| Represent failure as data | `Effect.either`, `Effect.cause`, `Effect.merge` |
| Swap success and failure channels | `Effect.flip` |
