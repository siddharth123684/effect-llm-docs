---
title: Built-In Schedules
description: Learn Effect's built-in schedules: Schedule.once, Schedule.recurs, Schedule.spaced, Schedule.exponential, and Schedule.fibonacci for retry and repeat workflows when building Effect programs and services.
---

# Built-In Schedules

Source: extracted from `llms-full.txt` (`Built-In Schedules`).

## Overview

Effect includes built-in schedules for common recurrence strategies:

- fixed or unbounded repeat counts (`once`, `recurs`, `forever`)
- interval-based repetition (`spaced`, `fixed`)
- increasing delay strategies (`exponential`, `fibonacci`)

These schedules are commonly used with `Effect.repeat(...)` and `Effect.retry(...)`.

## Helper Pattern for Inspecting Delays

The source section demonstrates schedules with a helper that runs a schedule and prints each computed delay.

```ts
import { Array, Chunk, Duration, Effect, Schedule } from "effect"

const logDelays = (
  schedule: Schedule.Schedule<unknown>,
  actionDuration: Duration.DurationInput = 0
) => {
  const maxRecurs = 10
  const delays = Chunk.toArray(
    Effect.runSync(
      Schedule.run(
        Schedule.delays(Schedule.addDelay(schedule, () => actionDuration)),
        Date.now(),
        Array.range(0, maxRecurs)
      )
    )
  )

  delays.forEach((d, i) => {
    console.log(
      i === maxRecurs
        ? "..."
        : i === delays.length - 1
        ? "(end)"
        : `#${i + 1}: ${Duration.toMillis(d)}ms`
    )
  })
}
```

## Infinite and Fixed Repeats

### `Schedule.forever`

Repeats indefinitely and outputs the recurrence index each time.

```ts
const schedule = Schedule.forever
```

### `Schedule.once`

Allows exactly one scheduled recurrence.

```ts
const schedule = Schedule.once
```

### `Schedule.recurs(n)`

Repeats a fixed number of times (`n` scheduled recurrences).

```ts
const schedule = Schedule.recurs(5)
```

## Recurring at Specific Intervals

`spaced` and `fixed` both model recurring intervals, but they measure time differently:

- `Schedule.spaced(duration)`: delay is measured from the end of the previous run
- `Schedule.fixed(duration)`: recurrences are aligned to fixed interval boundaries

When a run takes longer than the `fixed` interval, the next run happens immediately, but missed runs do not accumulate.

```text
|-----interval-----|-----interval-----|-----interval-----|
|---------action--------|action-------|action------------|
```

### `Schedule.spaced(duration)`

Useful when you want a consistent gap after each execution.
Example behavior from the source: with `spaced("200 millis")` and a simulated `100 millis` action, each cycle is about `300ms`.

```ts
const schedule = Schedule.spaced("200 millis")
```

### `Schedule.fixed(duration)`

Useful when you want wall-clock-like periodicity.
Example behavior from the source: with `fixed("200 millis")` and a simulated `100 millis` action, the first cycle is about `300ms`, then subsequent delays are about `200ms`.

```ts
const schedule = Schedule.fixed("200 millis")
```

## Increasing Delays Between Executions

### `Schedule.exponential(base)`

Backoff grows exponentially from the base delay.
Source example with `base = "10 millis"`:
`10, 20, 40, 80, 160, ...`

```ts
const schedule = Schedule.exponential("10 millis")
```

### `Schedule.fibonacci(base)`

Backoff grows like the Fibonacci sequence (scaled by the base delay).
Source example with `base = "10 millis"`:
`10, 10, 20, 30, 50, 80, ...`

```ts
const schedule = Schedule.fibonacci("10 millis")
```
