---
title: Getting Started
description: Learn Micro.tryPromise, Micro.gen, Micro.service, Context.Tag, and Micro.provideService to build lightweight Effect-style programs with Promise-oriented APIs for small bundle sizes and performance.
---

# Getting Started

Source: extracted from `llms-full.txt` (`Getting Started with Micro`).

## Overview

`Micro` is a lightweight alternative to the full `Effect` module, designed for smaller bundles and Promise-oriented library integrations.

- Baseline footprint starts around 5kb gzipped, then grows with used features.
- It is standalone and intentionally omits heavier modules like `Layer`, `Ref`, `Queue`, and `Deferred`.
- It works well when clients use `Micro` while other parts of a system use full `Effect`.

If you import major `Effect` runtime-heavy modules, you reduce the bundle-size advantage of `Micro`.

## Installing and Importing

Install `effect`:

```sh
npm install effect
```

Import `Micro` in either style:

```ts
import { Micro } from "effect"
// or
import * as Micro from "effect/Micro"
```

Named imports can be less tree-shake friendly in some bundlers without deep scope analysis.

## Core Types

`Micro` uses the same three-channel model as `Effect`:

```ts
Micro<Success, Error, Requirements>
```

- `Success`: value produced on success.
- `Error`: expected (typed) failure channel.
- `Requirements`: contextual dependencies required from `Context`.

`MicroExit<A, E>` represents completion:

- `Success<A>`
- `Failure` carrying a `MicroCause<E>`

`MicroCause<E>` variants:

- `Fail<E>` for expected errors
- `Die` for defects (unexpected failures)
- `Interrupt` for interruptions

## Wrapping Promise APIs

Use `Micro.promise` for direct Promise lifting, or `Micro.tryPromise` when you need typed error mapping.

```ts
import { Micro } from "effect"

class WeatherError {
  readonly _tag = "WeatherError"
  constructor(readonly message: string) {}
}

const fetchWeather = (city: string): Promise<string> =>
  city === "London"
    ? Promise.resolve("Sunny")
    : Promise.reject(new Error("Weather data not found"))

const getWeather = (city: string) =>
  Micro.tryPromise({
    try: () => fetchWeather(city),
    catch: (error) => new WeatherError(String(error))
  })
```

Run with:

- `Micro.runPromise(effect)` for Promise-style success/failure
- `Micro.runPromiseExit(effect)` when you need full `MicroExit` details

## Error Handling Essentials

### Expected Errors

Expected errors are typed in the `Error` channel and are part of normal control flow.

Key APIs:

- `Micro.either` to turn failures into `Either` values
- `Micro.catchAll` to recover from any expected error
- `Micro.catchTag` to recover only specific tagged errors

### Unexpected Errors (Defects)

Defects are not tracked in the `Error` type channel.

Key APIs:

- `Micro.die` to terminate with a defect
- `Micro.orDie` to convert expected errors into defects
- `Micro.catchAllDefect` to recover from defects only

### Fallback and Matching

- `Micro.orElseSucceed` replaces failure with a success value
- `Micro.match` and `Micro.matchEffect` handle success and failure branches
- `Micro.matchCause` and `Micro.matchCauseEffect` expose full failure causes

### Inspecting Errors

- `Micro.tapError` inspects expected failures
- `Micro.tapErrorCause` inspects full causes
- `Micro.tapDefect` inspects defects only

### Yieldable Errors

`Micro.Error` and `Micro.TaggedError` allow directly yielding custom errors inside `Micro.gen` without explicitly calling `Micro.fail`.

## Retrying and Timeout

- `Micro.retry(effect, { schedule })` retries by policy.
- `Micro.timeout(duration)` fails with `TimeoutException` when time budget is exceeded.
- If an effect is `Micro.uninterruptible`, timeout is reported after work finishes (it cannot be force-stopped mid-flight).

## Requirements Management

Define services with `Context.Tag`, access them via `Micro.service`, and satisfy requirements with `Micro.provideService`.

```ts
import { Context, Micro } from "effect"

class Random extends Context.Tag("MyRandomService")<
  Random,
  { readonly next: Micro.Micro<number> }
>() {}

const program = Micro.gen(function* () {
  const random = yield* Micro.service(Random)
  return yield* random.next
})

const runnable = Micro.provideService(program, Random, {
  next: Micro.sync(() => Math.random())
})
```

## Resource Management

`MicroScope` models resource lifetime and finalization.

- Finalizers run in reverse order of registration.
- `Micro.addFinalizer` attaches cleanup behavior to scope closure.
- `Micro.acquireRelease(acquire, release)` guarantees release after successful acquire.
- `Micro.acquireUseRelease(acquire, use, release)` scopes resource usage in one operator.
- `Micro.scoped` discharges `MicroScope` requirements once resource usage is enclosed.

## Scheduling

`MicroSchedule` computes delay by attempt and elapsed time:

```ts
type MicroSchedule = (attempt: number, elapsed: number) => Option<number>
```

Common tools:

- `Micro.repeat`
- `Micro.scheduleSpaced`
- `Micro.scheduleExponential`
- `Micro.scheduleUnion`
- `Micro.scheduleIntersect`

## Concurrency, Fibers, and Racing

- `Micro.fork` starts a fiber.
- `Micro.fiberJoin` waits for the fiber result.
- `Micro.fiberAwait` returns `MicroExit` for precise completion info.
- `Micro.fiberInterrupt` and `Micro.interrupt` stop running fibers safely.
- Interruptions propagate across concurrent workflows when applicable.
- `Micro.race` returns the first successful result; wrapping contenders with `Micro.either` captures first completion even on failure.
