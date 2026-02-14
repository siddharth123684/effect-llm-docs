---
title: Timing Out
description: Timeout operators to bound effect execution time and handle stalled operations.
---

# Timing Out

Source: extracted from `llms-full.txt` (`Timing Out`).

## Overview

Timeout operators put an upper bound on how long an effect is allowed to run.
Use them to keep workflows responsive when external calls or long-running tasks stall.

## Core Operators

| API | Timeout result | Notes |
| --- | --- | --- |
| `Effect.timeout(duration)` | fails with `TimeoutException` | Best default when timeout is an expected failure path. |
| `Effect.timeoutOption(duration)` | returns `Option.none()` | Models timeout as data instead of failure. |
| `Effect.timeoutFail({ duration, onTimeout })` | fails with custom error `E2` | Useful for domain-specific timeout errors. |
| `Effect.timeoutFailCause({ duration, onTimeout })` | fails with custom `Cause` | Useful when timeout should become a defect or structured cause. |
| `Effect.timeoutTo({ duration, onSuccess, onTimeout })` | returns mapped output | Lets you map success/timeout into a unified value. |

## `Effect.timeout`

`Effect.timeout` keeps the original success type and introduces possible timeout failure:

```ts
import { Effect } from "effect"

const task = Effect.sleep("2 seconds").pipe(Effect.as("Result"))
const program = task.pipe(Effect.timeout("1 second"))

// Exit.Failure(Cause.fail(TimeoutException))
Effect.runPromiseExit(program).then(console.log)
```

If the effect completes before the deadline, the original value is returned.

## `Effect.timeoutOption`

Use `timeoutOption` when you want timeout to be handled like a normal branch:

```ts
import { Effect } from "effect"

const fast = Effect.sleep("500 millis").pipe(Effect.as("ok"), Effect.timeoutOption("1 second"))
const slow = Effect.sleep("2 seconds").pipe(Effect.as("ok"), Effect.timeoutOption("1 second"))
```

- `fast` evaluates to `Option.some("ok")`
- `slow` evaluates to `Option.none()`

## Interruptible vs Uninterruptible Effects

Timeout behavior depends on interruptibility:

- **Interruptible effect**: interrupted when deadline is reached, then timeout failure is returned.
- **Uninterruptible effect**: continues until completion; timeout is assessed after completion.

This distinction matters for cleanup, consistency guarantees, and responsiveness.

## `Effect.disconnect` with Timeouts

`Effect.disconnect` improves responsiveness for uninterruptible work by letting that work continue in the background while the caller observes timeout immediately:

```ts
import { Effect } from "effect"

const program = Effect.sleep("5 seconds").pipe(
  Effect.uninterruptible,
  Effect.disconnect,
  Effect.timeout("1 second")
)
```

Use this when work must continue to completion, but your main flow should not block on it after timeout.

## Choosing the Right Operator

- Use `timeout` for standard timeout failures (`TimeoutException`).
- Use `timeoutOption` when timeout is a non-error branch.
- Use `timeoutFail` / `timeoutFailCause` when timeout should carry custom error semantics.
- Use `timeoutTo` when success and timeout should be mapped into one output type.
