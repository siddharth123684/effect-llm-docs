# Introduction

Source: extracted from `llms-full.txt` (`Introduction`).

## Overview

A `Sink` consumes elements produced by a `Stream` and eventually produces a
result. While consuming input, it can fail and it may also leave unconsumed
elements as leftovers.

The core sink type is:

```ts
Sink<A, In, L, E, R>
```

- `A`: final result type produced by the sink.
- `In`: input element type consumed from upstream.
- `L`: leftover element type returned when not all input is consumed.
- `E`: error type the sink can fail with.
- `R`: required dependencies (environment) needed to run the sink.

## Running a Stream with a Sink

Use `Stream.run(stream, sink)` to execute a stream with a sink:

```ts
import { Effect, Sink, Stream } from "effect"

const stream = Stream.make(1, 2, 3)
const sink = Sink.take<number>(2)

const result = Stream.run(stream, sink)

Effect.runPromise(result).then(console.log)
/*
Output:
{ _id: 'Chunk', values: [ 1, 2 ] }
*/
```

## Type Breakdown Example

For `Sink.take<number>(2)`, the sink type is:

```ts
Sink<Chunk<number>, number, number, never, never>
```

- `Chunk<number>`: the collected result.
- `number` (input): elements consumed from the stream.
- `number` (leftover): elements that may remain unconsumed.
- `never` (error): this sink does not fail.
- `never` (requirements): this sink requires no extra services.
