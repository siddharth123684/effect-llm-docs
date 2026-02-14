---
title: TestClock
description: Learn TestClock.adjust and TestClock.setTime for deterministic time control in tests, enabling fast testing of Effect.sleep, timeouts, and scheduled effects without real delays.
---

# TestClock

Source: extracted from `llms-full.txt` (`TestClock`).

## Overview

`TestClock` lets tests control time deterministically. Instead of waiting for real
time to pass, you move the clock manually so time-based effects run immediately
in a predictable way.

This keeps tests fast and stable when using APIs like `Effect.sleep`,
timeouts, delayed/repeated effects, and other scheduled work.

## How It Works

The `TestClock` does not advance on its own. You control it with:

- `TestClock.adjust(duration)` to move time forward
- `TestClock.setTime(...)` to set an absolute time

When time is advanced, any effects scheduled at or before that instant are
eligible to run.

## Core Testing Pattern

1. Fork the effect under test.
2. Advance `TestClock` by the needed duration.
3. Join/observe results and assert outcomes.

Forking is important because it keeps control in the test fiber while the
scheduled effect is waiting.

```ts
import { Effect, Fiber, Option, TestClock, TestContext } from "effect"

const test = Effect.gen(function* () {
  const fiber = yield* Effect.sleep("5 minutes").pipe(
    Effect.timeoutTo({
      duration: "1 minute",
      onSuccess: Option.some,
      onTimeout: () => Option.none<void>()
    }),
    Effect.fork
  )

  yield* TestClock.adjust("1 minute")
  const result = yield* Fiber.join(fiber)
  return Option.isNone(result)
}).pipe(Effect.provide(TestContext.TestContext))
```

## What It Helps Test

- Timeout behavior
- Fixed-interval / recurring execution
- `Clock`-based time reads (for example, `Clock.currentTimeMillis`)
- Delayed asynchronous coordination (for example with `Deferred`)

## Practical Notes

- Recurring workflows still schedule the next run after each occurrence; move the
  clock step-by-step to assert each recurrence.
- In real applications you normally use the live clock. `TestClock` and
  `TestContext` are primarily for tests.
