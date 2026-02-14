---
title: Operations
description: Sink operations for adapting input, transforming output, and filtering with mapInput and dimap.
---

# Operations

Source: extracted from `llms-full.txt` (`Operations`).

## Overview

`Sink` operations let you adapt or constrain how a sink consumes upstream data
without rewriting the sink from scratch.

This subsection focuses on:

- adapting input types with `Sink.mapInput`
- transforming both input and output with `Sink.dimap`
- filtering incoming elements with `Sink.filterInput`

## Adapting Sink Input

Use `Sink.mapInput` when the stream input type does not match what the sink
expects. It transforms each incoming element before the sink sees it.

`Sink.map` transforms a sink's output; `Sink.mapInput` transforms its input.

```ts
import { Effect, Sink, Stream } from "effect"

const stream = Stream.make("1", "2", "3", "4", "5")

const numericSum = Sink.sum

const stringSum = numericSum.pipe(
  Sink.mapInput((s: string) => Number.parseFloat(s))
)

Effect.runPromise(Stream.run(stream, stringSum)).then(console.log)
// Output: 15
```

## Transforming Both Input and Output

Use `Sink.dimap` when you need both:

- an input transformation before sink processing
- an output transformation after sink completion

```ts
import { Effect, Sink, Stream } from "effect"

const stream = Stream.make("1", "2", "3", "4", "5")

const sumSink = Sink.dimap(Sink.sum, {
  onInput: (s: string) => Number.parseFloat(s),
  onDone: (n) => String(n)
})

Effect.runPromise(Stream.run(stream, sumSink)).then(console.log)
// Output: "15"
```

## Filtering Input

Use `Sink.filterInput` to ignore elements that do not satisfy a predicate.
Only matching elements are consumed by the underlying sink logic.

```ts
import { Effect, Sink, Stream } from "effect"

const stream = Stream.fromIterable([
  1, -2, 0, 1, 3, -3, 4, 2, 0, 1, -3, 1, 1, 6
]).pipe(
  Stream.transduce(
    Sink.collectAllN<number>(3).pipe(
      Sink.filterInput((n) => n > 0)
    )
  )
)

Effect.runPromise(Stream.runCollect(stream)).then(console.log)
```

## Quick Selection Guide

- Use `mapInput` when only input conversion is needed.
- Use `dimap` when both input conversion and output reshaping are needed.
- Use `filterInput` when only a subset of upstream values should be processed.
