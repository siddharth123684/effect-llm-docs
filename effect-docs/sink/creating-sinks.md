# Creating Sinks

Source: extracted from `llms-full.txt` (`Creating Sinks`).

## Overview

`Sink` constructors define how a stream is consumed and what final value is produced.
The `Creating Sinks` section covers ready-made constructors for:

- selecting elements
- counting/summing
- effectful consumption
- collecting into data structures
- folding with stop/weight limits
- immediate success/failure sinks

## Common Constructors

### Element selection

- `Sink.head()`: first element as `Option<A>`.
- `Sink.last()`: last element as `Option<A>`.
- `Sink.take(n)`: first `n` elements as `Chunk<A>`.

### Aggregation and execution

- `Sink.count`: count all consumed elements.
- `Sink.sum`: sum numeric elements.
- `Sink.drain`: ignore all elements and return `void`.
- `Sink.timed`: run sink and return elapsed `Duration`.
- `Sink.forEach(f)`: run an effectful function for each element.

```ts
import { Effect, Sink, Stream } from "effect"

const stream = Stream.make(1, 2, 3, 4)

Effect.runPromise(Stream.run(stream, Sink.head())).then(console.log)
Effect.runPromise(Stream.run(stream, Sink.take(3))).then(console.log)
Effect.runPromise(Stream.run(stream, Sink.count)).then(console.log)
```

## Creating Sinks from Success and Failure

- `Sink.succeed(value)` creates a sink that immediately succeeds with `value` without consuming upstream elements.
- `Sink.fail(error)` creates a sink that immediately fails with `error` without consuming upstream elements.

```ts
import { Effect, Sink, Stream } from "effect"

const stream = Stream.make(1, 2, 3, 4)

Effect.runPromise(Stream.run(stream, Sink.succeed(0))).then(console.log)
Effect.runPromiseExit(Stream.run(stream, Sink.fail("fail!"))).then(console.log)
```

## Collecting Constructors

Use these when you need materialized results rather than a scalar reduction:

- `Sink.collectAll()`: collect everything into `Chunk<A>`.
- `Sink.collectAllN(n)`: collect at most `n` elements.
- `Sink.collectAllWhile(predicate)`: collect while predicate is true.
- `Sink.collectAllToSet()`: collect unique values into `HashSet<A>`.
- `Sink.collectAllToSetN(n)`: collect unique values up to set size `n`.
- `Sink.collectAllToMap(key, merge)`: group/merge into `HashMap<K, A>`.
- `Sink.collectAllToMapN(n, key, merge)`: same, but bounded by key count.

```ts
import { Effect, Sink, Stream } from "effect"

const stream = Stream.make(1, 3, 2, 3, 1, 5, 1)

Effect.runPromise(
  Stream.run(
    stream,
    Sink.collectAllToMap(
      (n) => n % 3,
      (a, b) => a + b
    )
  )
).then(console.log)
```

## Folding Constructors

Folding sinks reduce stream input incrementally:

- `Sink.foldLeft(initial, f)`: left-associative fold across all input.
- `Sink.fold(initial, cont, f)`: fold with a continuation predicate for early stop.
- `Sink.foldUntil(initial, max, f)`: fold until `max` elements are processed.
- `Sink.foldWeighted({ initial, maxCost, cost, body })`: fold while total computed cost stays within `maxCost` (useful for chunking/transduction).

```ts
import { Effect, Sink, Stream } from "effect"

const stream = Stream.iterate(0, (n) => n + 1)

Effect.runPromise(
  Stream.run(
    stream,
    Sink.fold(
      0,
      (sum) => sum <= 10,
      (a, b) => a + b
    )
  )
).then(console.log)
```

## Selection Guide

- Use `head` / `last` / `take` for positional extraction.
- Use `count` / `sum` / `fold*` for numerical or stateful reduction.
- Use `collectAll*` when downstream needs full buffered results.
- Use `forEach` for effectful per-element handling.
- Use `succeed` / `fail` to create sinks with explicit immediate outcomes.
