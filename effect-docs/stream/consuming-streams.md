---
title: Consuming Streams
description: Learn how to run streams and produce results using runCollect, runForEach, runFold, and Stream.run with Sink for element gathering and consumption.
---

# Consuming Streams

Source: extracted from `llms-full.txt` (`Consuming Streams`).

## Overview

Consuming a stream means running it to produce a final effectful result.
Common choices depend on whether you need all elements, per-element effects, a
single accumulated value, or sink-driven consumption.

## Collect All Elements with `Stream.runCollect`

Use `Stream.runCollect` to gather all emitted elements into one `Chunk`.

```ts
import { Effect, Stream } from "effect"

const stream = Stream.make(1, 2, 3, 4, 5)

const program = Stream.runCollect(stream)

Effect.runPromise(program).then(console.log)
// { _id: "Chunk", values: [1, 2, 3, 4, 5] }
```

## Run an Effect for Each Element with `Stream.runForEach`

Use `Stream.runForEach` when each element should trigger an effect (for example,
logging, persistence, or publishing).

```ts
import { Console, Effect, Stream } from "effect"

const program = Stream.make(1, 2, 3).pipe(
  Stream.runForEach((n) => Console.log(n))
)

Effect.runPromise(program)
```

## Fold Stream Values into One Result

Use fold-style consumers when you want one accumulated value:

- `Stream.runFold(initial, f)`: consume the whole stream
- `Stream.runFoldWhile(initial, cont, f)`: stop early when `cont` becomes false

```ts
import { Effect, Stream } from "effect"

const sumAll = Stream.make(1, 2, 3, 4, 5).pipe(
  Stream.runFold(0, (acc, n) => acc + n)
)

const sumWhileLe3 = Stream.make(1, 2, 3, 4, 5).pipe(
  Stream.runFoldWhile(0, (n) => n <= 3, (acc, n) => acc + n)
)

Effect.runPromise(sumAll).then(console.log) // 15
Effect.runPromise(sumWhileLe3).then(console.log) // 6
```

## Consume with a `Sink` via `Stream.run`

Use `Stream.run(sink)` to delegate consumption logic to a `Sink`.

```ts
import { Effect, Sink, Stream } from "effect"

const program = Stream.make(1, 2, 3).pipe(Stream.run(Sink.sum))

Effect.runPromise(program).then(console.log) // 6
```
