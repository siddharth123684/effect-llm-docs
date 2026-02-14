---
title: Ref
description: Learn Ref.make, Ref.update, Ref.get, and Ref.modify for shared mutable state across fibers with atomic updates when building concurrent Effect programs.
---

# Ref

Source: extracted from `llms-full.txt` (`Ref`).

## Overview

`Ref<A>` is Effect's mutable reference for shared state. It lets multiple parts of a program (including multiple fibers) read and update the same value in a controlled way.

Unlike plain mutable variables, `Ref` operations are performed through effects, which makes state changes explicit in Effect workflows.

## Basic Usage

Create a `Ref`, update it, then read the value:

```ts
import { Effect, Ref } from "effect"

const program = Effect.gen(function* () {
  const counter = yield* Ref.make(0)
  yield* Ref.update(counter, (n) => n + 1)
  yield* Ref.update(counter, (n) => n + 1)
  yield* Ref.update(counter, (n) => n - 1)
  return yield* Ref.get(counter)
})
```

## Effectful Operations

Reads and writes on `Ref` are effectful operations. That includes creating a `Ref` (`Ref.make`) and interacting with it (`Ref.get`, `Ref.update`, etc.).

## Concurrent Updates

`Ref` is useful in concurrent programs where multiple fibers update shared state. The docs demonstrate running several updates concurrently and then reading the final value from the same `Ref`.

## Using Ref as a Service

You can model shared application state as a service by storing a `Ref` in a `Context.Tag` and providing it to effects.

Because `Ref.make(...)` is effectful, use `Effect.provideServiceEffect` when wiring it into the environment.

## Sharing State Between Fibers

A common pattern is:

- one fiber reads external input and appends updates to state
- another fiber applies periodic/background updates
- both fibers mutate the same `Ref`
- the parent fiber joins both and reads final state

This allows coordinated shared-state workflows without passing mutable variables directly.
