---
title: SynchronizedRef
description: Mutable reference supporting atomic effectful updates for concurrent shared state.
---

# SynchronizedRef

Source: extracted from `llms-full.txt` (`SynchronizedRef`).

## Overview

`SynchronizedRef<A>` is a mutable reference to a value of type `A` that supports atomic, effectful updates.

Like `Ref`, it is used for shared state. The key difference is that updates can run effects safely while preserving sequential consistency across concurrent fibers.

## How It Differs from Ref

- `Ref` updates are pure transformations of the current value.
- `SynchronizedRef` adds `updateEffect`, where the new value is produced by an effect.
- Effectful updates are synchronized so concurrent fibers do not interleave state transitions unsafely.

## Core Operation: `updateEffect`

`updateEffect` lets you:

1. read the current value
2. run an effect (for example, I/O or computation)
3. publish the next value atomically

This is useful when state transitions depend on effectful work rather than only pure functions.

## Example: Concurrent Effectful Updates

```ts
import { Effect, SynchronizedRef } from "effect"

const getUserAge = (userId: number) =>
  Effect.succeed(userId * 10).pipe(Effect.delay(10 - userId))

const program = Effect.gen(function* () {
  const ref = yield* SynchronizedRef.make<number[]>([])

  const task = (id: number) =>
    SynchronizedRef.updateEffect(ref, (ages) =>
      Effect.gen(function* () {
        const age = yield* getUserAge(id)
        return ages.concat(age)
      })
    )

  yield* Effect.all([task(1), task(2), task(3), task(4)], { concurrency: 2 })
  return yield* SynchronizedRef.get(ref)
})
```

Even with concurrent execution, updates are applied in a synchronized sequence, keeping shared state consistent.

## When to Use

Use `SynchronizedRef` when:

- updates require effectful steps before producing the next value
- many fibers update the same state concurrently
- you need deterministic, sequentially applied state transitions
