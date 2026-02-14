---
title: Concurrency
description: Learn Sink.zip with concurrent option and Sink.race to run multiple sinks against the same stream for parallel combined results or first-complete outcomes.
---

# Concurrency

Source: extracted from `llms-full.txt` (`Sink Concurrency`).

## Overview

Sink concurrency lets you run multiple sinks against the same upstream stream
at the same time. This is useful when you want to derive multiple results in
one pass or return as soon as one sink completes.

## Combine Results with `Sink.zip`

Use `Sink.zip(left, right, { concurrent: true })` to run both sinks
concurrently and return both results as a tuple.

```ts
import { Effect, Sink, Stream } from "effect"

const stream = Stream.make("1", "2", "3")

const sink1 = Sink.forEach((s: string) => Effect.log(`sink 1: ${s}`)).pipe(
  Sink.as(1)
)
const sink2 = Sink.forEach((s: string) => Effect.log(`sink 2: ${s}`)).pipe(
  Sink.as(2)
)

const combined = Sink.zip(sink1, sink2, { concurrent: true })

Effect.runPromise(Stream.run(stream, combined)).then(console.log)
// [1, 2]
```

## Race Sinks with `Sink.race`

Use `Sink.race(left, right)` when only the first completed result matters.
Both sinks run, and the winner's value is returned.

```ts
import { Effect, Sink, Stream } from "effect"

const stream = Stream.make("1", "2", "3")

const sink1 = Sink.forEach((s: string) => Effect.log(`sink 1: ${s}`)).pipe(
  Sink.as(1)
)
const sink2 = Sink.forEach((s: string) => Effect.log(`sink 2: ${s}`)).pipe(
  Sink.as(2)
)

const first = Sink.race(sink1, sink2)

Effect.runPromise(Stream.run(stream, first)).then(console.log)
// 1 (or 2, depending on which sink finishes first)
```

## Selection Guide

- Use `Sink.zip(..., { concurrent: true })` when you need both results.
- Use `Sink.race(...)` when you want the earliest completion.
- Prefer these operators when a single stream pass should produce multiple
  concurrent outcomes.
