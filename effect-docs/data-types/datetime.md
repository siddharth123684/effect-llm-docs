# DateTime

Source: extracted from `llms-full.txt` (`DateTime`).

## Overview

`DateTime` is Effect's immutable date-time model. It addresses common JavaScript
`Date` issues by separating:

- the exact instant (`epochMillis`)
- optional time zone context (`TimeZone`)

This design improves correctness for scheduling, logging, and time zone-aware
applications.

## Core Model

`DateTime` has two variants:

- `Utc`: instant only (`epochMillis`)
- `Zoned`: instant plus a `TimeZone`

`TimeZone` also has two variants:

- `TimeZone.Offset`: fixed offset from UTC
- `TimeZone.Named`: IANA zone id (for example, `Europe/Rome`), including DST rules

```ts
type DateTime = Utc | Zoned

interface Utc {
  readonly _tag: "Utc"
  readonly epochMillis: number
}

interface Zoned {
  readonly _tag: "Zoned"
  readonly epochMillis: number
  readonly zone: TimeZone
}
```

## Parts and Input Types

`DateTime.Parts` exposes date fields (`year`, `month`, `day`, `hours`,
`minutes`, `seconds`, `millis`) with `PartsWithWeekday` adding `weekDay`.

`DateTime.Input` accepts:

- an existing `DateTime`
- `Date`
- epoch milliseconds (`number`)
- `Partial<DateTime.Parts>`
- parseable date-time string

## Constructors

### Utc Constructors

- `DateTime.unsafeFromDate(date)`: convert `Date` to `Utc`, throws on invalid
- `DateTime.unsafeMake(input)`: build `Utc` from `DateTime.Input`, throws on invalid
- `DateTime.make(input)`: safe version returning `Option<Utc>`

### Zoned Constructors

- `DateTime.unsafeMakeZoned(input, options?)`: create `Zoned`, throws on invalid input/zone
- `DateTime.makeZoned(input, options?)`: safe `Option<Zoned>` variant
- `DateTime.makeZonedFromString(text)`: parse
  `YYYY-MM-DDTHH:mm:ss.sss+HH:MM[Zone]` into `Option<Zoned>`

`unsafeMakeZoned` supports:

- `timeZone` as object, IANA string, or numeric offset
- `adjustForTimeZone` to interpret input in the target zone before conversion

```ts
import { DateTime, Option } from "effect"

const utc = DateTime.unsafeMake("2025-01-01")
const zoned = DateTime.unsafeMakeZoned("2025-01-01T04:00:00", {
  timeZone: "Europe/Rome"
})

const maybe = DateTime.make("2025-01-01")
if (Option.isSome(maybe)) {
  console.log(DateTime.formatIso(maybe.value))
}
```

## Getting Current Time

- `DateTime.now`: `Effect<DateTime.Utc>` using the `Clock` service
- `DateTime.unsafeNow()`: immediate `Utc` via `Date.now()`

Use `now` when you want testable, service-based time access.

## Guards

- `isDateTime`, `isUtc`, `isZoned`
- `isTimeZone`, `isTimeZoneOffset`, `isTimeZoneNamed`

## Time Zone Management

Core APIs:

- `setZone`, `setZoneOffset`, `setZoneNamed`, `unsafeSetZoneNamed`
- `zoneUnsafeMakeNamed`, `zoneMakeNamed`, `zoneMakeNamedEffect`
- `zoneMakeOffset`, `zoneMakeLocal`, `zoneFromString`, `zoneToString`

Use named zones when daylight saving transitions matter.

## Comparison APIs

- `distance`, `distanceDurationEither`, `distanceDuration`
- `min`, `max`
- ordering predicates (`greaterThan`, `lessThanOrEqualTo`, etc.)
- `between`, `isFuture`, `isPast` (and `unsafe*` variants)

## Conversions and Parts

Conversion helpers:

- `toDateUtc`, `toDate`, `toEpochMillis`
- `zonedOffset`, `zonedOffsetIso`
- `removeTime`

Parts helpers:

- `toParts`, `toPartsUtc`
- `getPart`, `getPartUtc`
- `setParts`, `setPartsUtc`

## Math and Formatting

Math:

- `addDuration`, `subtractDuration`
- `add`, `subtract`
- `startOf`, `endOf`, `nearest`

Formatting:

- `format`, `formatIntl`
- `formatLocal`, `formatUtc`
- `formatIso`, `formatIsoDate`, `formatIsoDateUtc`
- `formatIsoOffset`, `formatIsoZoned`

## Current Time Zone Layers

Effect integration for zone-aware programs:

- `CurrentTimeZone`
- `nowInCurrentZone`, `setZoneCurrent`
- `withCurrentZone`, `withCurrentZoneLocal`, `withCurrentZoneOffset`,
  `withCurrentZoneNamed`
- `layerCurrentZone`, `layerCurrentZoneLocal`, `layerCurrentZoneOffset`,
  `layerCurrentZoneNamed`

```ts
import { DateTime, Effect } from "effect"

const program = Effect.gen(function* () {
  const now = yield* DateTime.nowInCurrentZone
  console.log(DateTime.formatIsoZoned(now))
}).pipe(DateTime.withCurrentZoneNamed("Europe/London"))
```

## Practical Guidance

- Prefer safe constructors (`make`, `makeZoned`, `zoneFromString`) for external input.
- Use `Utc` for canonical storage and ordering.
- Add `Zoned` context only when display/locale semantics are required.
- Prefer named zones over fixed offsets when DST correctness is important.
