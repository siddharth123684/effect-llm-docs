---
title: Examples
description: Learn practical patterns combining Effect.timeout, Effect.retry, and Schedule for API resilience, periodic work, and robust retry workflows when building Effect programs.
---

# Examples

Source: extracted from `llms-full.txt` (`Examples`).

## Overview

These scheduling examples show practical resilience patterns for real-world Effect programs:

- timeout + retry around API calls
- retry only for selected failures
- dynamic retry delays from error metadata
- periodic work that stops when another task completes

## Handling Timeouts and Retries for API Calls

For third-party calls, combine bounded retries with a hard timeout:

```ts
import { Console, Effect } from "effect"

const program = (url: string) =>
  getJson(url).pipe(
    Effect.retry({ times: 2 }),
    Effect.timeout("4 seconds"),
    Effect.catchAll(Console.error)
  )
```

This retries transient failures up to two times, but still fails fast if the total attempt exceeds the timeout window.

## Retrying API Calls Based on Specific Errors

Retries do not have to apply to every error. A common strategy is to retry only when status is `401` and fail immediately otherwise.

```ts
import { Effect } from "effect"

const program = (url: string) =>
  getJson(url).pipe(
    Effect.retry({ while: (err) => err.status === 401 })
  )
```

This keeps retry behavior intentional and prevents unnecessary retries for permanent failures like `404`.

## Retrying with Dynamic Delays from `Retry-After`

When an API returns rate-limit information (for example, `429` with a retry delay), derive the schedule delay from the error itself:

```ts
import { Duration, Effect, Schedule } from "effect"

const policy = Schedule.identity<TooManyRequestsError>().pipe(
  Schedule.addDelay((error) => Duration.millis(error.retryAfter)),
  Schedule.intersect(Schedule.recurs(5))
)

const program = request.pipe(Effect.retry(policy))
```

This adapts wait time to server guidance and limits retry count.

## Running Periodic Tasks Until Another Task Completes

Use `Effect.race` to run fixed-interval work until a separate long-running effect finishes:

```ts
import { Console, Effect, Schedule } from "effect"

const action = Console.log("action...")
const schedule = Schedule.fixed("1.5 seconds")

const program = Effect.race(
  Effect.repeat(action, schedule),
  longRunningEffect
)
```

This pattern is useful for polling, heartbeats, and progress logging while waiting for completion.
