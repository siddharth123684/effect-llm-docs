---
title: Caching Effects
description: Helpers for memoizing effectful computations: cachedFunction, once, cached, and cachedWithTTL.
---

# Caching Effects

Source: extracted from `llms-full.txt` (`Caching Effects`).

## Overview

Effect provides helpers to memoize effectful computations so repeated calls can reuse results instead of recomputing.

This subsection covers:

- `Effect.cachedFunction` for memoizing effectful functions by input
- `Effect.once` for running an effect only once
- `Effect.cached` for lazily caching a computed result
- `Effect.cachedWithTTL` for time-based cache expiry
- `Effect.cachedInvalidateWithTTL` for TTL caching with manual invalidation

## `Effect.cachedFunction`

`Effect.cachedFunction` memoizes a function that returns an `Effect`. For the same input, the computation runs once and subsequent calls reuse the cached result.

```ts
import { Effect, Random } from "effect"

const randomNumber = (n: number) => Random.nextIntBetween(1, n)

const program = Effect.gen(function* () {
  const memoized = yield* Effect.cachedFunction(randomNumber)
  const first = yield* memoized(10)
  const second = yield* memoized(10)
  console.log(first, second) // same cached value
})
```

## `Effect.once`

`Effect.once` converts an effect into one that executes at most once, even if invoked multiple times.

```ts
import { Console, Effect } from "effect"

const program = Effect.gen(function* () {
  const task = yield* Effect.once(Console.log("runs once"))
  yield* task
  yield* task
  yield* task
})
```

## `Effect.cached`

`Effect.cached` returns a new effect that computes lazily on first evaluation, then reuses the cached result on later evaluations.

- First evaluation executes the original effect.
- Later evaluations return the cached value without re-running the effect.

## `Effect.cachedWithTTL`

`Effect.cachedWithTTL(effect, timeToLive)` caches the result for a fixed duration.

- Calls inside the TTL window return the cached value.
- After TTL expires, the next call recomputes and refreshes the cache.

```ts
import { Console, Effect } from "effect"

let i = 1
const expensiveTask = Effect.sync(() => `result ${i++}`)

const program = Effect.gen(function* () {
  const cached = yield* Effect.cachedWithTTL(expensiveTask, "150 millis")
  yield* cached.pipe(Effect.andThen(Console.log))
  yield* cached.pipe(Effect.andThen(Console.log))
  yield* Effect.sleep("200 millis")
  yield* cached.pipe(Effect.andThen(Console.log))
})
```

## `Effect.cachedInvalidateWithTTL`

`Effect.cachedInvalidateWithTTL(effect, timeToLive)` works like `cachedWithTTL`, but also returns an invalidation effect.

- You get `[cached, invalidate]`.
- `invalidate` clears the cached value immediately.
- The next evaluation recomputes even if TTL has not expired.

## Choosing the Right Helper

- Use `cachedFunction` when cache keys come from function arguments.
- Use `once` when side effects must run only one time.
- Use `cached` for lazy, indefinite memoization of one effect result.
- Use `cachedWithTTL` when values should refresh automatically over time.
- Use `cachedInvalidateWithTTL` when you need both TTL and explicit invalidation.
