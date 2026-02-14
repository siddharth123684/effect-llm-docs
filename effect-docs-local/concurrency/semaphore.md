# Semaphore

Source: extracted from `llms-full.txt` (`Semaphore`).

## Overview

A semaphore is a synchronization primitive for controlling concurrent access to a
shared resource.

In Effect, a semaphore manages a pool of **permits**:

- each running task acquires some permits
- tasks wait when not enough permits are available
- permits are returned after the guarded effect finishes

This makes a semaphore a generalized mutex (a mutex is effectively a semaphore
with one permit).

## Creating a Semaphore

Use `Effect.makeSemaphore(n)` to create a semaphore with `n` permits.

```ts
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const semaphore = yield* Effect.makeSemaphore(3)
  return semaphore
})
```

## Running Effects with `withPermits`

`semaphore.withPermits(n)(effect)` runs `effect` only after `n` permits are
available.

With one total permit, concurrent tasks are forced to run one at a time:

```ts
import { Effect } from "effect"

const task = Effect.gen(function* () {
  yield* Effect.log("start")
  yield* Effect.sleep("2 seconds")
  yield* Effect.log("end")
})

const program = Effect.gen(function* () {
  const semaphore = yield* Effect.makeSemaphore(1)
  const guarded = semaphore.withPermits(1)(task)

  yield* Effect.all([guarded, guarded, guarded], {
    concurrency: "unbounded"
  })
})
```

## Weighted Concurrency

Permits can represent resource weight, not just task count.

For example, a task that consumes more capacity can request more permits:

```ts
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const semaphore = yield* Effect.makeSemaphore(5)

  const tasks = [1, 2, 3].map((n) =>
    semaphore.withPermits(n)(Effect.log(`process: ${n}`))
  )

  yield* Effect.all(tasks, { concurrency: "unbounded" })
})
```

## Permit Release Guarantee

`withPermits` releases acquired permits automatically when the wrapped effect
ends, including failure or interruption paths.

## Key APIs

- `Effect.makeSemaphore`
- `semaphore.withPermits`
