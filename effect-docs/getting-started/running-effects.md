---
title: Running Effects
description: Learn to execute effects with Effect.runSync, Effect.runPromise, Effect.runSyncExit, and Effect.runFork at program boundaries for sync, async, and forked execution modes.
---

# Running Effects

Source: extracted from `llms-full.txt` (`Running Effects`).

## Overview

Effects are lazy descriptions of work. Nothing happens until you execute an effect with a `run*` function from `Effect`.

A common guideline is to keep most logic as pure effect values and call `run*` at the edge of your program (for example, `main`, a request handler boundary, or a worker entrypoint).

## Run API Cheatsheet

| API | Returns | Typical use |
| --- | --- | --- |
| `Effect.runSync(effect)` | `A` | Immediate synchronous result (throws on failure or async work) |
| `Effect.runSyncExit(effect)` | `Exit<A, E>` | Synchronous result as `Exit` (success or failure details) |
| `Effect.runPromise(effect)` | `Promise<A>` | Promise interop; rejects on failure |
| `Effect.runPromiseExit(effect)` | `Promise<Exit<A, E>>` | Promise interop while keeping `Exit` semantics |
| `Effect.runFork(effect)` | `RuntimeFiber<A, E>` | Start in background and control via a fiber |

## `runSync`

Use when the effect is truly synchronous and you want the value immediately.

- If the effect fails, `runSync` throws.
- If the effect performs async work, `runSync` throws an async fiber exception.

```ts
import { Effect } from "effect"

const program = Effect.sync(() => 1)
const value = Effect.runSync(program) // 1
```

## `runSyncExit`

Use when you want a synchronous result without throwing, represented as `Exit`.

- Success becomes `Exit.Success`.
- Failure/defects become `Exit.Failure` with a `Cause`.
- Async work cannot be resolved synchronously, so the result is a failing `Exit`.

```ts
import { Effect } from "effect"

const ok = Effect.runSyncExit(Effect.succeed(1))
const failed = Effect.runSyncExit(Effect.fail("my error"))
```

## `runPromise`

Use when integrating with Promise-based APIs.

- Resolves with success value.
- Rejects if the effect fails.

```ts
import { Effect } from "effect"

Effect.runPromise(Effect.succeed(1)).then(console.log)
```

## `runPromiseExit`

Use when you need Promise execution but still want explicit `Exit` success/failure handling.

```ts
import { Effect } from "effect"

Effect.runPromiseExit(Effect.fail("my error")).then((exit) => {
  console.log(exit)
})
```

## `runFork`

`runFork` starts an effect in the background and returns a fiber.

Use it when you need lifecycle control (observe, join, interrupt, supervise).

```ts
import { Effect, Fiber } from "effect"

const fiber = Effect.runFork(Effect.never)
Effect.runFork(Fiber.interrupt(fiber))
```

## Best Practices

- Prefer asynchronous execution (`runPromise` or `runFork`) in most applications.
- Use `runSync` only for effects that are guaranteed synchronous.
- Keep execution points centralized near program boundaries.

