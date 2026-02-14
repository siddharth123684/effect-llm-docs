# Leftovers

Source: extracted from `llms-full.txt` (`Leftovers`).

## Overview

Sinks do not always consume all upstream elements. Any unconsumed input is
returned as leftovers. Effect provides helpers to either keep those leftovers
or discard them explicitly.

## Collecting Leftovers

Use `Sink.collectLeftover` when you want both:

- the sink result
- the leftover elements that were not consumed

It returns a tuple-like pair: `[result, leftoverChunk]`.

```ts
import { Effect, Sink, Stream } from "effect"

const stream = Stream.make(1, 2, 3, 4, 5)

const sink = Sink.take<number>(3).pipe(Sink.collectLeftover)

Effect.runPromise(Stream.run(stream, sink)).then(console.log)
/*
Output:
[
  { _id: "Chunk", values: [1, 2, 3] },
  { _id: "Chunk", values: [4, 5] }
]
*/
```

You can also combine `Sink.collectLeftover` with sinks like `Sink.head()` to
get a single selected value plus all remaining elements.

## Ignoring Leftovers

Use `Sink.ignoreLeftover` when leftovers are irrelevant to your workflow.
This discards unconsumed elements so downstream logic only sees the sink result.

```ts
import { Effect, Sink, Stream } from "effect"

const stream = Stream.make(1, 2, 3, 4, 5)

const sink = Sink.take<number>(3).pipe(
  Sink.ignoreLeftover,
  Sink.collectLeftover
)

Effect.runPromise(Stream.run(stream, sink)).then(console.log)
/*
Output:
[
  { _id: "Chunk", values: [1, 2, 3] },
  { _id: "Chunk", values: [] }
]
*/
```

## When to Use Which

- Use `collectLeftover` when another stage still needs the unconsumed elements.
- Use `ignoreLeftover` when partial consumption is expected and leftovers should
  be dropped.
