---
title: Repetition
description: How to repeat successful effects multiple times using Effect.repeat and schedules.
---

# Repetition

Source: extracted from `llms-full.txt` (`Repetition`).

## Overview

Repetition runs an effect multiple times according to a schedule. It focuses on repeating successful executions, while retrying focuses on recovering from failures.

With schedules, each recurrence is controlled by interval boundaries, so the policy defines when the next run happens.

## `Effect.repeat`

`Effect.repeat(effect, schedule)` repeats a successful effect using the provided schedule.

- The initial execution is included.
- Scheduled recurrences are additional executions after the first success.
- Repetition stops on the first failure.

```ts
import { Console, Effect, Schedule } from "effect"

const action = Console.log("success")
const policy = Schedule.recurs(2)

const program = Effect.repeat(action, policy)
```

`Effect.repeat(action, Schedule.once)` means one initial run plus one additional recurrence.

## Skipping the Initial Execution

Use `Effect.schedule(effect, schedule)` to apply the schedule without the immediate first run.

```ts
import { Console, Effect, Schedule } from "effect"

const program = Effect.schedule(Console.log("tick"), Schedule.recurs(2))
```

## `Effect.repeatN`

`Effect.repeatN(effect, n)` repeats the effect a fixed number of times, in addition to the initial execution.

## `Effect.repeatOrElse`

`Effect.repeatOrElse(effect, schedule, handler)` repeats with a schedule, and if failure occurs, runs a fallback handler with failure context instead of failing immediately.

## Conditional Repetition

You can repeat based on successful outputs:

- `Effect.repeat(effect, { while: predicate })`
- `Effect.repeat(effect, { until: predicate })`

```ts
import { Effect } from "effect"

let count = 0
const action = Effect.sync(() => ++count)

const untilThree = Effect.repeat(action, { until: (n) => n === 3 })
```

For error-based conditional reruns, use `Effect.retry`.
