---
title: Control Flow Operators
description: Effect helpers for branching, conditional execution, combining effects, and typed looping.
---

# Control Flow Operators

Source: extracted from `llms-full.txt` (`Control Flow Operators`).

## Overview

Effect provides control-flow helpers for:

- branching between effects
- conditional execution
- combining multiple effects
- looping with typed state and failures

These operators keep control flow explicit and type-safe.

## `if` Expression

Standard JavaScript `if` works normally when returning `Effect` values.

```ts
import { Effect } from "effect"

const validateWeightOrFail = (weight: number): Effect.Effect<number, string> => {
  if (weight >= 0) {
    return Effect.succeed(weight)
  }
  return Effect.fail(`negative input: ${weight}`)
}
```

## Conditional Operators

### `Effect.if`

Runs `onTrue` or `onFalse` based on an effectful boolean predicate.

```ts
import { Console, Effect, Random } from "effect"

const flipTheCoin = Effect.if(Random.nextBoolean, {
  onTrue: () => Console.log("Head"),
  onFalse: () => Console.log("Tail")
})
```

### `Effect.when` and `Effect.whenEffect`

- `Effect.when`: run an effect only when a pure boolean condition is `true`.
- `Effect.whenEffect`: run an effect only when an effectful condition evaluates to `true`.
- Both return `Option<A>` (`Some` when executed, `None` when skipped).

```ts
import { Effect, Random } from "effect"

const randomIntOption = Random.nextInt.pipe(
  Effect.whenEffect(Random.nextBoolean)
)
```

### `Effect.unless` and `Effect.unlessEffect`

Inverted versions of `when*`: execute only when the condition is `false`
(equivalent to `if (!condition) ...` semantics).

## Zipping

### `Effect.zip`

Combines two effects into a tuple.

- Default: sequential execution.
- With `{ concurrent: true }`: run both effects concurrently.

### `Effect.zipWith`

Like `zip`, but combines both results with a function.

```ts
import { Effect } from "effect"

const task1 = Effect.succeed(1)
const task2 = Effect.succeed("hello")

const combined = Effect.zipWith(task1, task2, (n, s) => n + s.length)
// Effect<number, never, never>
```

## Looping

### `Effect.loop`

State-based looping with:

- `while`: continue condition
- `step`: next state
- `body`: effectful work

Returns collected results by default; set `{ discard: true }` to return `void`.

### `Effect.iterate`

Effectful state iteration that repeatedly applies `body` while `while` holds.
Returns the final state.

### `Effect.forEach`

Runs an effect for each item in an iterable.

- Default behavior is sequential and short-circuits on first failure.
- `concurrency` controls parallelism.
- `{ discard: true }` ignores collected results and returns `void`.

## Collecting with `Effect.all`

`Effect.all` combines many effects into one effect and supports tuples, iterables,
structs, and records.

- Default behavior: short-circuit on the first failure.
- `{ mode: "either" }`: run all and collect `Either` results.
- `{ mode: "validate" }`: run all and collect validation-style `Option` errors.

## Cheatsheet

| API | Purpose | Notes |
| --- | --- | --- |
| `Effect.if` | choose between two effects | predicate is effectful |
| `Effect.when` | run effect when condition is true | returns `Option<A>` |
| `Effect.whenEffect` | run effect when effectful condition is true | returns `Option<A>` |
| `Effect.unless` | run effect when condition is false | inverse of `when` |
| `Effect.unlessEffect` | run effect when effectful condition is false | inverse of `whenEffect` |
| `Effect.zip` | combine two effects into a tuple | optional concurrent mode |
| `Effect.zipWith` | combine two effects with a function | optional concurrent mode |
| `Effect.loop` | while-style state loop with `step` | can collect or discard |
| `Effect.iterate` | effectful state update loop | returns final state |
| `Effect.forEach` | apply effect to each iterable element | supports concurrency and discard |
| `Effect.all` | collect many effects in one structure | short-circuits by default |
