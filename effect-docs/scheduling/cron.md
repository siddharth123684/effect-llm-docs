---
title: Cron
description: Calendar-based schedules using UNIX cron expressions in Effect.
---

# Cron

Source: extracted from `llms-full.txt` (`Cron`).

## Overview

The `Cron` module defines calendar-based schedules using UNIX-style cron expressions.
It supports:

- building cron schedules from explicit field constraints
- parsing and validating cron expression strings
- testing whether a date matches a schedule
- computing next and future matching dates
- converting a `Cron` into a `Schedule` for `Effect.repeat` / `Effect.retry` style workflows

`Cron` also supports optional time zone handling.

## Creating a Cron with `Cron.make`

Use `Cron.make` when you want explicit control over each field:

- `seconds`
- `minutes`
- `hours`
- `days`
- `months`
- `weekdays`
- optional `tz`

If a field list is empty, that part is unconstrained (any valid value is allowed).

```ts
import { Cron, DateTime } from "effect"

const cron = Cron.make({
  seconds: [0],
  minutes: [0],
  hours: [4],
  days: [8, 9, 10, 11, 12, 13, 14],
  months: [],
  weekdays: [],
  tz: DateTime.zoneUnsafeMakeNamed("Europe/Rome")
})
```

## Parsing Cron Expressions

Use parsing helpers when you already have a cron expression string.

### `Cron.parse(expression, tz?)`

Safely parses and returns an `Either`:

- `Right<Cron>` on success
- `Left<ParseError>` on invalid input

```ts
import { Cron, Either } from "effect"

const parsed = Cron.parse("0 0 4 8-14 * *")

if (Either.isRight(parsed)) {
  console.log(parsed.right)
} else {
  console.error(parsed.left.message)
}
```

### `Cron.unsafeParse(expression, tz?)`

Parses like `Cron.parse`, but throws if invalid.

```ts
import { Cron } from "effect"

const cron = Cron.unsafeParse("0 0 4 8-14 * *")
```

## Working with Dates

### `Cron.match(cron, dateInput)`

Returns `true` if the provided `Date` (or `DateTime.Input`) satisfies the cron constraints.

### `Cron.next(cron, after?)`

Computes the next matching `Date` after the optional `after` input.
If `after` is omitted, current time is used.

### `Cron.sequence(cron, start)`

Returns an infinite iterator of future matching dates.

```ts
import { Cron } from "effect"

const cron = Cron.unsafeParse("0 0 4 8-14 * *", "UTC")

console.log(Cron.match(cron, new Date("2025-01-08T04:00:00.000Z")))
console.log(Cron.next(cron, new Date("2025-01-08T00:00:00.000Z")))

const it = Cron.sequence(cron, new Date("2025-01-08T00:00:00.000Z"))
console.log(it.next().value)
```

## Converting Cron to a Schedule

`Schedule.cron(...)` bridges `Cron` with Effect scheduling.
It accepts a cron expression or a `Cron` instance and creates a `Schedule` that triggers at each cron interval boundary.

When it runs, schedule output is a tuple `[start, end]` (milliseconds) for the current cron interval window.

```ts
import { Cron, Schedule } from "effect"

const cron = Cron.unsafeParse("0 0 4 8-14 * *", "UTC")
const schedule = Schedule.cron(cron)
```
