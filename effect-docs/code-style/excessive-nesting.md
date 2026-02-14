---
title: Excessive Nesting
description: Learn to reduce deeply nested pipe calls using Effect.gen, Effect.Do, Effect.bind, and Effect.let for clearer sequential control flow and readability.
---

# Excessive Nesting

Source: extracted from `llms-full.txt` (`Excessive Nesting`).

## Overview

When Effect programs are composed with repeated nested `pipe` calls, code can
become deeply indented and difficult to read.

A common example is a helper that measures elapsed execution time:

- read start time
- run an effect
- read end time
- log elapsed milliseconds

All of this is straightforward logically, but nested combinators can obscure the
flow.

## Problem: Deeply Nested `pipe`

This style works, but quickly becomes verbose:

```ts
import { Console, Effect } from "effect"

const now = Effect.sync(() => Date.now())

const elapsed = <R, E, A>(self: Effect.Effect<A, E, R>) =>
  now.pipe(
    Effect.andThen((startMillis) =>
      self.pipe(
        Effect.andThen((result) =>
          now.pipe(
            Effect.andThen((endMillis) =>
              Console.log(`Elapsed: ${endMillis - startMillis}`).pipe(
                Effect.map(() => result)
              )
            )
          )
        )
      )
    )
  )
```

## Solution 1: Do Simulation (`Effect.Do`)

`Effect.Do` lets you bind intermediate values in a flatter structure with
`Effect.bind` and derive values with `Effect.let`.

```ts
import { Console, Effect } from "effect"

const now = Effect.sync(() => Date.now())

const elapsed = <R, E, A>(self: Effect.Effect<A, E, R>) =>
  Effect.Do.pipe(
    Effect.bind("startMillis", () => now),
    Effect.bind("result", () => self),
    Effect.bind("endMillis", () => now),
    Effect.let("elapsed", ({ startMillis, endMillis }) => endMillis - startMillis),
    Effect.tap(({ elapsed }) => Console.log(`Elapsed: ${elapsed}`)),
    Effect.map(({ result }) => result)
  )
```

Use `Effect.Do` when you want explicit named bindings while staying in
combinator style.

## Solution 2: Generator Style (`Effect.gen`)

`Effect.gen` is usually the most concise option for sequential workflows.

```ts
import { Effect } from "effect"

const now = Effect.sync(() => Date.now())

const elapsed = <R, E, A>(self: Effect.Effect<A, E, R>) =>
  Effect.gen(function* () {
    const startMillis = yield* now
    const result = yield* self
    const endMillis = yield* now
    console.log(`Elapsed: ${endMillis - startMillis}`)
    return result
  })
```

The generator keeps control flow linear and avoids excessive nesting while
preserving Effect semantics.

## Practical Guidance

- prefer `Effect.gen` for linear effectful sequences
- use `Effect.Do` when named intermediate bindings improve clarity
- avoid deeply nested `pipe` chains when the logic is inherently sequential
