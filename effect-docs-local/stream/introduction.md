# Introduction

Source: extracted from `llms-full.txt` (`Introduction to Streams`).

## Overview

A `Stream<A, E, R>` is a program description that, when executed, can emit zero
or more values of type `A`, fail with errors of type `E`, and require services
of type `R`.

## When to Use Streams

Streams are useful for value sequences over time, including finite and infinite
flows. They are often a practical alternative to observables, Node streams, or
`AsyncIterable`-style processing.

## Stream vs Effect

`Effect<A, E, R>` and `Stream<A, E, R>` share the same typed model for values,
errors, and requirements, but they differ in cardinality:

- `Effect` produces exactly one success value (or fails).
- `Stream` produces zero, one, many, or infinitely many values.

## Common Stream Shapes

- **Empty stream**: emits no values.
- **Single-element stream**: emits exactly one value.
- **Finite stream**: emits a bounded sequence, then completes.
- **Infinite stream**: emits an unbounded sequence.

## Minimal Examples

```ts
import { Stream } from "effect"

const emptyStream = Stream.empty
const singleValue = Stream.succeed(3)
const finiteValues = Stream.range(1, 10)
const infiniteValues = Stream.iterate(1, (n) => n + 1)
```

These forms capture the core idea: a stream is an effectful, typed description
of value production over time.
