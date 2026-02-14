---
title: Deferred
description: One-time synchronization primitive for fiber coordination, similar to Promise with explicit types.
---

# Deferred

Source: extracted from `llms-full.txt` (`Deferred`).

## Overview

`Deferred<Success, Error>` is a one-time synchronization primitive in Effect.
It starts empty, can be completed exactly once, and then becomes immutable.

Fibers can wait on it with `Deferred.await`. Waiting is semantic (fiber-level),
so threads are not blocked and other fibers continue running.

It is similar to a `Promise`, but with explicit success and error type
parameters.

## Type Shape

```text
Deferred<Success, Error>
```

- `Success`: the value type when completed successfully
- `Error`: the error type when completed with failure

## Creating and Awaiting

Use `Deferred.make` to allocate a deferred, then `Deferred.await` to wait for
its result:

```ts
import { Deferred, Effect } from "effect"

const program = Effect.gen(function* () {
  const deferred = yield* Deferred.make<string, string>()
  const fiber = yield* Effect.fork(Deferred.await(deferred))

  yield* Deferred.succeed(deferred, "ready")
  return yield* Effect.fromFiber(fiber)
})
```

## Completing a Deferred

`Deferred` can be completed with success, failure, interruption, or an effect:

- `Deferred.succeed`
- `Deferred.fail`
- `Deferred.die`
- `Deferred.failCause`
- `Deferred.done`
- `Deferred.complete`
- `Deferred.completeWith`
- `Deferred.interrupt`

Completion operations return `Effect<boolean>`:

- `true`: this call completed the deferred
- `false`: it was already completed earlier

## Checking Completion Without Waiting

- `Deferred.poll` returns an `Option<Effect<A, E>>`
  - `None` if not completed
  - `Some(...)` if completed
- `Deferred.isDone` returns `Effect<boolean>`

These are useful when you need to inspect state without suspending.

## Typical Concurrency Uses

- Coordinate two or more fibers
- Suspend one workflow until another signals readiness
- Handoff work/results between fibers
- Gate progress on a single event occurring
