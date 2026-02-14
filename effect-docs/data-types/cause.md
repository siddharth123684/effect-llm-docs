# Cause

Source: extracted from `llms-full.txt` (`Cause`).

## Overview

`Cause<E>` captures full failure information for Effect computations, not only expected errors of type `E`.
It preserves details such as defects, interruption metadata, and composition of failures across sequential or parallel execution.

## Creating Causes

Use `Effect.failCause` when you want to fail an effect with a specific `Cause` value.

```ts
import { Cause, Effect } from "effect"

const expectedFailure = Effect.failCause(Cause.fail("Oh no!"))
const defect = Effect.failCause(Cause.die("Boom!"))
```

`Cause.fail` contributes to the typed error channel, while `Cause.die` represents an untyped defect.

## Cause Variations

- `Empty`: no error information
- `Fail<E>`: expected failure of type `E`
- `Die`: unexpected defect
- `Interrupt`: fiber interruption (contains `FiberId`)
- `Sequential`: two causes that happened one after another
- `Parallel`: two causes that happened concurrently

`Sequential` and `Parallel` preserve both branches of failure, which is important for debugging composed programs.

## Retrieving the Cause of an Effect

Use `Effect.cause` to access the exact failure cause:

```ts
import { Effect } from "effect"

const program = Effect.cause(Effect.fail("Oh no!"))
```

## Guards

The Cause module provides guards for narrowing cause shapes:

- `Cause.isEmpty`
- `Cause.isFailType`
- `Cause.isDie`
- `Cause.isInterruptType`
- `Cause.isSequentialType`
- `Cause.isParallelType`

## Pattern Matching

`Cause.match` lets you handle all cause variants explicitly by providing handlers for each case (`onEmpty`, `onFail`, `onDie`, `onInterrupt`, `onSequential`, `onParallel`).

## Pretty Printing

Use `Cause.pretty` to render a readable diagnostic string from a cause, useful for logs and debugging output.

## Retrieving Failures and Defects

Use:

- `Cause.failures(cause)` to collect expected failures
- `Cause.defects(cause)` to collect defects

These helpers are useful when you need targeted reporting from a complex combined cause.
