# Building Pipelines

Source: extracted from `llms-full.txt` (`Building Pipelines`).

## Overview

Pipelines let you compose transformations and effectful steps in a clear left-to-right flow. This makes programs easier to read, test, and evolve.

## Why Pipelines Help

- **Readability**: operations are applied in sequence, so data flow is explicit.
- **Organization**: complex workflows are split into smaller focused steps.
- **Reusability**: small functions can be reused across multiple pipelines.
- **Type safety**: each step has typed input/output, reducing runtime surprises.

## Functions over Methods

Effect is function-first:

- **Tree shakeability**: unused functions can be removed by bundlers.
- **Extensibility**: plain functions are easy to compose and extend.

## `pipe`

Use `pipe` to pass a value through a sequence of single-argument functions.

```ts
import { pipe } from "effect"

const result = pipe(
  5,
  (n) => n + 1,
  (n) => n * 2,
  (n) => n - 10
)
// result = 2
```

## Core Pipeline Operators

| Operator | Use it to | Notes |
| --- | --- | --- |
| `Effect.map` | transform a success value | keeps error and requirements unchanged |
| `Effect.as` | replace success value with a constant | useful when original result is irrelevant |
| `Effect.flatMap` | sequence dependent effectful steps | function must return an `Effect` |
| `Effect.andThen` | sequence with value/function/promise/effect | broad convenience operator |
| `Effect.tap` | run side effects while keeping original value | fails pipeline if tap effect fails |
| `Effect.all` | combine many effects into one | default behavior is sequential with short-circuit on first failure |

## `andThen` with `Option` and `Either`

`Effect.andThen` treats:

- `Option<A>` as `Effect<A, NoSuchElementException, never>`
- `Either<A, E>` as `Effect<A, E, never>`

So the resulting error type is merged into the pipeline's error union.

## Build a Pipeline

```ts
import { Effect, pipe } from "effect"

const addServiceCharge = (amount: number) => amount + 1

const applyDiscount = (total: number, discountRate: number) =>
  discountRate === 0
    ? Effect.fail(new Error("Discount rate cannot be zero"))
    : Effect.succeed(total - (total * discountRate) / 100)

const fetchTransactionAmount = Effect.promise(() => Promise.resolve(100))
const fetchDiscountRate = Effect.promise(() => Promise.resolve(5))

const program = pipe(
  Effect.all([fetchTransactionAmount, fetchDiscountRate]),
  Effect.andThen(([amount, discountRate]) => applyDiscount(amount, discountRate)),
  Effect.andThen(addServiceCharge),
  Effect.andThen((finalAmount) => `Final amount to charge: ${finalAmount}`)
)
```

## `pipe` Method

You can also write pipelines with the method form:

```ts
const result = effect.pipe(op1, op2, op3)
```

This is equivalent to `pipe(effect, op1, op2, op3)`.

## Cheatsheet

| API | Input | Output |
| --- | --- | --- |
| `map` | `Effect<A, E, R>`, `A => B` | `Effect<B, E, R>` |
| `flatMap` | `Effect<A, E, R>`, `A => Effect<B, E2, R2>` | `Effect<B, E | E2, R | R2>` |
| `andThen` | `Effect<A, E, R>`, next action | `Effect<B, E2, R2>` (merged where needed) |
| `tap` | `Effect<A, E, R>`, `A => Effect<B, E2, R2>` | `Effect<A, E | E2, R | R2>` |
| `all` | collection of effects | effect of collected results |
