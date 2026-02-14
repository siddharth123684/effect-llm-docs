# Configuration

Source: extracted from `llms-full.txt` (`Configuration`).

## Overview

In Effect, schedule configuration means choosing how and when an effect should recur, then refining that behavior with combinators.
Schedules are immutable values with type:

```text
Schedule<Out, In, Requirements>
```

- `Out`: value produced by the schedule as it runs
- `In`: input consumed at each step (for example, retry errors or repeat outputs)
- `Requirements`: extra services/resources needed by the schedule

## Core Schedule Choices

Choose a base policy first:

- `Schedule.once`: recur one additional time
- `Schedule.recurs(n)`: recur a fixed number of times
- `Schedule.forever`: recur indefinitely
- `Schedule.spaced(duration)`: wait a fixed gap after each run
- `Schedule.fixed(duration)`: align recurrences to fixed intervals
- `Schedule.exponential(duration)`: exponential backoff
- `Schedule.fibonacci(duration)`: Fibonacci-style increasing delays

For calendar-driven timing, use `Schedule.cron(...)` with a parsed `Cron` expression.

## Configuring Repetition

`Effect.repeat(effect, schedule)` repeats a successful effect according to a schedule.
By default, the initial execution is included, then scheduled recurrences are applied.

Useful variants:

- `Effect.schedule(effect, schedule)`: follows the schedule without the initial immediate execution
- `Effect.repeatN(effect, n)`: repeat a fixed number of times
- `Effect.repeatOrElse(effect, schedule, handler)`: handle failure after repeats
- `Effect.repeat(effect, { while | until })`: condition-based repetition

## Configuring Behavior with Combinators

You can refine base schedules with combinators:

- `Schedule.union(a, b)`: continue if either continues; uses the shorter delay
- `Schedule.intersect(a, b)`: continue only if both continue; uses the longer delay
- `Schedule.andThen(a, b)`: run `a` fully, then switch to `b`
- `Schedule.jittered(schedule)`: add randomness to delay windows
- `Schedule.modifyDelay(schedule, f)`: dynamically alter delay from schedule output
- `Schedule.whileInput` / `Schedule.whileOutput`: stop based on predicates
- `Schedule.tapInput` / `Schedule.tapOutput`: observe input/output with effects

## Minimal Example

```ts
import { Effect, Schedule } from "effect"

const policy = Schedule.intersect(
  Schedule.jittered(Schedule.exponential("100 millis")),
  Schedule.recurs(5)
)

const configured = Effect.retry(task, policy)
```

This configuration applies exponential backoff, introduces jitter to avoid synchronized retries, and caps retries to 5 recurrences.
