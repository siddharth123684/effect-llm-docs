---
title: Error Accumulation
description: Collecting multiple failures with validate, validateAll, validateFirst, and partition.
---

# Error Accumulation

Source: extracted from `llms-full.txt` (`Error Accumulation`).

## Overview

Many sequential combinators in Effect are fail-fast:

- `Effect.zip`
- `Effect.all`
- `Effect.forEach`

Fail-fast means evaluation stops at the first failure.

When you need to collect multiple failures (instead of stopping early), use the accumulation APIs below.

## `Effect.validate`

`Effect.validate` is like `zip`, but it keeps evaluating and accumulates failures.

- Successes are combined when all effects succeed.
- Failures are accumulated in the cause when multiple effects fail.

```ts
import { Effect } from "effect"

const task1 = Effect.succeed(1)
const task2 = Effect.fail("Oh uh!")
const task3 = Effect.succeed(3)
const task4 = Effect.fail("Oh no!")

const program = task1.pipe(
  Effect.validate(task2),
  Effect.validate(task3),
  Effect.validate(task4)
)
```

## `Effect.validateAll`

`Effect.validateAll` is the accumulating variant of `forEach`.

- It evaluates all items.
- It collects all failures into the error channel.
- If any failures occur, successes are not returned (lossy behavior).

```ts
import { Effect } from "effect"

const program = Effect.validateAll([1, 2, 3, 4], (n) =>
  n < 3 ? Effect.succeed(n) : Effect.fail(`${n} is not less than 3`)
)
```

## `Effect.validateFirst`

`Effect.validateFirst` returns:

- the first successful result, or
- all accumulated errors if nothing succeeds.

```ts
import { Effect } from "effect"

const program = Effect.validateFirst([1, 2, 3, 4], (n) =>
  n < 3 ? Effect.fail(`${n} rejected`) : Effect.succeed(n)
)
```

## `Effect.partition`

`Effect.partition` processes all inputs and always succeeds with both sides:

- left: all failures
- right: all successes

It does not fail in the error channel (`E = never`).

```ts
import { Effect } from "effect"

const program = Effect.partition([0, 1, 2, 3, 4], (n) =>
  n % 2 === 0 ? Effect.succeed(n) : Effect.fail(`${n} is not even`)
)
// Effect<[string[], number[]], never, never>
```

## Quick Selection

| Goal | API |
| --- | --- |
| Accumulate failures while combining effects | `Effect.validate` |
| Traverse a collection and gather all errors | `Effect.validateAll` |
| Return first success, otherwise all errors | `Effect.validateFirst` |
| Keep both failures and successes explicitly | `Effect.partition` |
