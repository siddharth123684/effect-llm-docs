---
title: Parallel and Sequential Errors
description: How Effect preserves parallel and sequential failure causes in execution.
---

# Parallel and Sequential Errors

Source: extracted from `llms-full.txt` (`Parallel and Sequential Errors`).

## Overview

Effect usually fails with the first error encountered.
When multiple failures happen in concurrent execution or during finalization, the runtime keeps a richer `Cause` structure instead of losing information.

## Parallel Errors

Concurrent effects can fail at nearly the same time, producing a `Cause` with `_tag: "Parallel"`.

```ts
import { Effect } from "effect"

const fail = Effect.fail("Oh uh!")
const die = Effect.dieMessage("Boom!")

const program = Effect.all([fail, die], { concurrency: "unbounded" }).pipe(
  Effect.asVoid
)
```

### `Effect.parallelErrors`

`Effect.parallelErrors` collects all **failure** values from a parallel cause into the typed error channel.

```ts
import { Effect } from "effect"

const fail1 = Effect.fail("Oh uh!")
const fail2 = Effect.fail("Oh no!")
const die = Effect.dieMessage("Boom!")

const program = Effect.all([fail1, fail2, die], { concurrency: "unbounded" }).pipe(
  Effect.asVoid,
  Effect.parallelErrors
)
```

`parallelErrors` is only for failures (`Effect.fail`), not defects (`die`) or interruptions.

## Sequential Errors

Sequential error causes commonly appear with resource-safety operators such as `Effect.ensuring`.
The finalizer always runs (uninterruptibly), so both the original failure and finalizer failure can be preserved in a `Cause` with `_tag: "Sequential"`.

```ts
import { Effect } from "effect"

const fail = Effect.fail("Oh uh!")
const die = Effect.dieMessage("Boom!")

const program = fail.pipe(Effect.ensuring(die))
```

## Practical Takeaways

- Default behavior is fail-fast on the first error.
- Parallel execution can produce combined causes (`Parallel`).
- Finalizers can add a second error after an initial failure (`Sequential`).
- Use `Effect.parallelErrors` when you need all parallel failure values in the typed error channel.
