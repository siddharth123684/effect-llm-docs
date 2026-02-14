---
title: Duration
description: Learn Duration.millis, Duration.seconds, Duration.decode, Duration.sum, and Duration.format for non-negative time spans in timeouts, delays, retries, and scheduling when using Effect.
---

# Duration

Source: extracted from `llms-full.txt` (`Duration`).

## Overview

`Duration` represents non-negative spans of time in Effect. It is used for
timeouts, delays, retries, scheduling, and other time-based configuration.

## Creating Durations

Use unit constructors for explicit values:

- `Duration.nanos`, `Duration.micros`, `Duration.millis`
- `Duration.seconds`, `Duration.minutes`, `Duration.hours`
- `Duration.days`, `Duration.weeks`
- `Duration.infinity` for an unbounded duration

```ts
import { Duration } from "effect"

const fast = Duration.millis(100)
const retryDelay = Duration.seconds(2)
const ttl = Duration.minutes(5)
const unbounded = Duration.infinity
```

### Decoding Inputs

`Duration.decode` converts common input shapes into a `Duration`:

- `number` -> milliseconds
- `bigint` -> nanoseconds
- `Infinity` -> infinite duration
- string like `"2 seconds"` or `"100 millis"`

```ts
import { Duration } from "effect"

Duration.decode(100) // millis
Duration.decode(10n) // nanos
Duration.decode("5 minutes")
Duration.decode(Infinity)
```

## Reading Values

- `Duration.toMillis(duration)`: returns milliseconds as `number`
- `Duration.toNanos(duration)`: returns `Option<bigint>` (infinite may not
  produce nanos)
- `Duration.unsafeToNanos(duration)`: returns `bigint` or throws on infinity

```ts
import { Duration } from "effect"

const d = Duration.seconds(30)
const ms = Duration.toMillis(d) // 30000
```

## Comparison APIs

Use these to compare two durations:

- `Duration.lessThan`
- `Duration.lessThanOrEqualTo`
- `Duration.greaterThan`
- `Duration.greaterThanOrEqualTo`

## Arithmetic APIs

Common arithmetic operations:

- `Duration.sum(a, b)` to add durations
- `Duration.times(duration, n)` to scale by a multiplier

```ts
import { Duration } from "effect"

const base = Duration.seconds(30)
const total = Duration.sum(base, Duration.minutes(1))
const doubled = Duration.times(base, 2)
```

## Formatting

Use `Duration.format(duration)` for a compact human-readable string.

```ts
import { Duration } from "effect"

Duration.format(Duration.millis(1000)) // "1s"
Duration.format(Duration.millis(1001)) // "1s 1ms"
```
