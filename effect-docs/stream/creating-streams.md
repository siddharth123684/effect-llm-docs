---
title: Creating Streams
description: Learn Stream constructors including make, fromEffect, async, unfold, paginate, fromQueue, fromPubSub, and repeatEffectOption for pure values, effects, callbacks, iterables, and concurrency primitives.
---

# Creating Streams

Source: extracted from `llms-full.txt` (`Creating Streams`).

## Overview

Effect `Stream` provides many constructors so you can model data sources as:

- pure values
- effectful computations
- callbacks and iterables
- repeated/time-based signals
- paginated/unfolded state machines
- messaging primitives (`Queue` / `PubSub`)

## Common Constructors

- `Stream.make(...values)`: emit a fixed list of values.
- `Stream.empty`: emit nothing.
- `Stream.void`: emit one `void` value.
- `Stream.range(min, max)`: emit integers in `[min, max]`.
- `Stream.iterate(seed, next)`: infinite progression from a seed.
- `Stream.scoped(effect)`: one-element stream backed by scoped acquire/use/release.

```ts
import { Effect, Stream } from "effect"

const finite = Stream.make(1, 2, 3)
const sequential = Stream.range(1, 5)
const infinite = Stream.iterate(1, (n) => n + 1).pipe(Stream.take(5))

Effect.runPromise(Stream.runCollect(finite)).then(console.log)
Effect.runPromise(Stream.runCollect(sequential)).then(console.log)
Effect.runPromise(Stream.runCollect(infinite)).then(console.log)
```

## From Success, Failure, Chunks, and Effects

- `Stream.succeed(a)`: single-element stream.
- `Stream.fail(e)`: immediately fails.
- `Stream.fromChunk(chunk)`: stream from one `Chunk`.
- `Stream.fromChunks(...chunks)`: stream from multiple chunks.
- `Stream.fromEffect(effect)`: evaluate an `Effect` and emit its result.

These APIs are the direct bridge from core Effect values into stream pipelines.

## From Async Callback and Iterables

### `Stream.async`

Use `Stream.async` to adapt callback-based producers that can emit multiple times.

The callback consumes `Effect<Chunk<A>, Option<E>, R>`:

- success with `Chunk<A>`: emit elements
- failure with `Option.some(error)`: end with that error
- failure with `Option.none()`: end stream successfully

### Iterable constructors

- `Stream.fromIterable(iterable)`: pure iterable source
- `Stream.fromIterableEffect(effectReturningIterable)`: iterable produced by an effect
- `Stream.fromAsyncIterable(asyncIterable, onError)`: async iterable with explicit error mapping

## From Repetition and Time

- `Stream.repeatValue(a)`: repeat one value forever.
- `Stream.repeat(stream, schedule)`: repeat a stream according to a `Schedule`.
- `Stream.repeatEffect(effect)`: repeat an effect result forever.
- `Stream.repeatEffectOption(effect)`: repeat until effect fails with `Option.none()`.
- `Stream.tick(duration)`: periodic `void` events.

`repeatEffectOption` is useful for draining pull-based sources (for example, iterators) where `None` is used as an end-of-stream signal.

## Unfolding and Pagination

### Unfold family

- `Stream.unfold(initial, step)`
- `Stream.unfoldEffect(initial, stepEffect)`
- `Stream.unfoldChunk(...)`
- `Stream.unfoldChunkEffect(...)`

`unfold` builds a stream from state transitions. Each step either:

- returns `Option.some([output, nextState])` to continue, or
- returns `Option.none()` to stop.

### Paginate family

- `Stream.paginate(initial, step)`
- `Stream.paginateChunk(...)`
- `Stream.paginateChunkEffect(...)`

`paginate` emits a value and separately returns the optional next state. This is often more ergonomic for paginated APIs and avoids losing the final page output.

```ts
import { Effect, Option, Stream } from "effect"

const pages = Stream.paginate(0, (n) => [
  n,
  n < 3 ? Option.some(n + 1) : Option.none()
])

Effect.runPromise(Stream.runCollect(pages)).then(console.log)
// { _id: 'Chunk', values: [0, 1, 2, 3] }
```

## From Queue, PubSub, and Schedule

- `Stream.fromQueue(queue)`: stream values from a `Queue`.
- `Stream.fromPubSub(pubsub)`: stream values from a `PubSub`.
- `Stream.fromSchedule(schedule)`: emit each output produced by a schedule until it stops.

These constructors are useful when your upstream already exists as an Effect concurrency primitive or scheduling policy.

## Constructor Selection Guide

- Use `make` / `range` / `iterate` for pure synthetic data.
- Use `fromEffect` / `fromIterableEffect` for effectful acquisition.
- Use `async` for callback-style APIs.
- Use `repeat*` / `tick` for polling, retries, and periodic signals.
- Use `unfold*` / `paginate*` for stateful and paginated traversal.
- Use `fromQueue` / `fromPubSub` when integrating with concurrent producers.
