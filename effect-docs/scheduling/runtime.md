---
title: Runtime
description: How schedules are evaluated at runtime for repeat and retry behavior in Effect.
---

# Runtime

Source: extracted from `llms-full.txt` (`Runtime`).

## Overview

In scheduling, runtime behavior is about how a `Schedule` is evaluated over time and inputs to decide:

- whether recurrence should continue
- when the next recurrence should run
- what output the schedule produces at each step

Schedules are stateful and time-aware, and are applied while running effects with repeat/retry operators.

## Running Schedules with Effects

The scheduling behavior is used at runtime through effect operators:

- `Effect.repeat(effect, schedule)`: repeats successful executions according to the schedule (includes the initial execution)
- `Effect.schedule(effect, schedule)`: follows only the schedule-driven recurrences (skips initial immediate run)
- `Effect.retry(effect, schedule)`: re-executes on failure using schedule intervals

From the scheduling overview, retries and repeats run again at interval boundaries defined by the schedule.

## Evaluating a Schedule Directly

`llms-full.txt` scheduling examples repeatedly use `Schedule.run(...)` with `Schedule.delays(...)` to inspect runtime delays.

```ts
import { Array, Effect, Schedule } from "effect"

const delays = Effect.runSync(
  Schedule.run(
    Schedule.delays(Schedule.addDelay(Schedule.fixed("200 millis"), () => 0)),
    Date.now(),
    Array.range(0, 10)
  )
)
```

This pattern executes the schedule logic directly and returns computed delay values for each recurrence.

## Runtime Timing Semantics

Key runtime timing behavior shown in scheduling docs:

- `Schedule.spaced(duration)`: spaces repetitions from the end of the previous run
- `Schedule.fixed(duration)`: aligns recurrences to fixed intervals and does not pile up missed runs
- `Schedule.exponential(...)` / `Schedule.fibonacci(...)`: increase delays over time
- `Schedule.jittered(...)`: randomizes delay windows to reduce synchronized retries

## Example: Periodic Work Until Another Task Completes

```ts
import { Console, Effect, Schedule } from "effect"

const longRunningEffect = Console.log("done").pipe(Effect.delay("5 seconds"))
const action = Console.log("action...")
const schedule = Schedule.fixed("1.5 seconds")

const program = Effect.race(
  Effect.repeat(action, schedule),
  longRunningEffect
)

Effect.runPromise(program)
```

This runs a periodic action while another effect is in progress, stopping when the competing task completes.
