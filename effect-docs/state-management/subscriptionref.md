---
title: SubscriptionRef
description: SynchronizedRef with a stream of changes for reactive shared-state workflows.
---

# SubscriptionRef

Source: extracted from `llms-full.txt` (`SubscriptionRef`).

## Overview

`SubscriptionRef<A>` is a specialized `SynchronizedRef<A>` that keeps mutable shared state and exposes a stream of updates.

It supports normal ref-style operations (`get`, `set`, `modify`, and related APIs) plus a `changes` stream for observers.

```ts
interface SubscriptionRef<A> extends SynchronizedRef<A> {
  readonly changes: Stream<A>
}
```

## Key Behavior

`changes` emits:

- the current value at subscription time
- every subsequent update

Each time you run the stream, it starts by publishing the latest value and then continues with future changes.

## Creating a SubscriptionRef

Use `SubscriptionRef.make(initialValue)`:

```ts
import { SubscriptionRef } from "effect"

const ref = SubscriptionRef.make(0)
```

## Observing Changes

A common pattern is to run `ref.changes` in one fiber while other fibers update the value:

```ts
import { Console, Effect, Stream, SubscriptionRef } from "effect"

const program = Effect.gen(function* () {
  const ref = yield* SubscriptionRef.make(0)

  yield* ref.changes.pipe(
    Stream.tap((n) => Console.log(`SubscriptionRef changed to ${n}`)),
    Stream.runDrain,
    Effect.fork
  )

  yield* SubscriptionRef.set(ref, 1)
  yield* SubscriptionRef.set(ref, 2)
})
```

## Server-Client Pattern

`SubscriptionRef` is useful when one process updates shared state and multiple consumers react to updates:

- a producer continuously updates the ref
- clients read from `ref.changes` as a `Stream`
- clients can independently transform, filter, and collect values with stream operators

This makes `SubscriptionRef` a good fit for reactive shared-state workflows.
