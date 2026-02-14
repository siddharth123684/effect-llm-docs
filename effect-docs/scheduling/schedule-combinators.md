---
title: Schedule Combinators
description: Learn Schedule.union, Schedule.intersect, Schedule.jitter, and Schedule.filter to combine and transform retry and repeat policies when building resilient Effect applications and services.
---

# Schedule Combinators

Source: extracted from `llms-full.txt` (`Schedule Combinators`).

## Overview

Schedules are stateful recurrence policies. Combinators let you combine and
transform schedules to build richer retry/repeat behavior.

The source examples inspect delays in this format:

```text
#<repetition>: <delay in ms>
```

## Composition

Three core composition modes:

- `Schedule.union(left, right)`: continue while either schedule continues,
  using the shorter delay.
- `Schedule.intersect(left, right)`: continue only while both continue, using
  the longer delay.
- `Schedule.andThen(first, second)`: run `first` to completion, then switch to
  `second`.

### Union

```ts
import { Schedule } from "effect"

const schedule = Schedule.union(
  Schedule.exponential("100 millis"),
  Schedule.spaced("1 second")
)
```

With exponential + spaced, early delays follow exponential growth; once the
exponential delay exceeds `1 second`, the spaced interval dominates.

### Intersection

```ts
import { Schedule } from "effect"

const schedule = Schedule.intersect(
  Schedule.exponential("10 millis"),
  Schedule.recurs(5)
)
```

This enforces both constraints: exponential backoff that also stops after
`5` recurrences.

### Sequencing

```ts
import { Schedule } from "effect"

const schedule = Schedule.andThen(
  Schedule.recurs(5),
  Schedule.spaced("1 second")
)
```

This gives immediate retries first, then periodic retries.

## Adding Randomness to Retry Delays

`Schedule.jittered` adds randomness to computed delays, which helps avoid retry
storms where many clients retry at the same instant.

```ts
import { Schedule } from "effect"

const schedule = Schedule.jittered(Schedule.exponential("10 millis"))
```

The source notes that jitter is a practical strategy to reduce synchronized
contention.

## Controlling Repetitions with Filters

Use filter combinators to stop based on schedule input/output:

- `Schedule.whileInput(schedule, predicate)`
- `Schedule.whileOutput(schedule, predicate)`

```ts
import { Schedule } from "effect"

const schedule = Schedule.whileOutput(Schedule.recurs(5), (n) => n <= 2)
```

Even if `recurs(5)` allows more runs, filtering can terminate earlier.

## Adjusting Delays Based on Output

`Schedule.modifyDelay` changes the next delay dynamically from schedule output.

```ts
import { Schedule } from "effect"

const schedule = Schedule.modifyDelay(
  Schedule.spaced("1 second"),
  (out, duration) => (out > 2 ? "100 millis" : duration)
)
```

This keeps the initial spacing, then shortens later delays.

## Tapping

`Schedule.tapInput` and `Schedule.tapOutput` run effectful callbacks for
observability/debugging without changing scheduling decisions.

```ts
import { Console, Schedule } from "effect"

const schedule = Schedule.tapOutput(Schedule.recurs(2), (out) =>
  Console.log(`Schedule Output: ${out}`)
)
```
