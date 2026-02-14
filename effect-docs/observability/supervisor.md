# Supervisor

Source: extracted from `llms-full.txt` (`Supervisor`).

## Overview

A `Supervisor<A>` manages fiber supervision in Effect. It tracks child-fiber lifecycle events (creation and termination) and exposes a value of type `A` representing supervised state.

Supervisors are useful when you need runtime visibility into fiber behavior in concurrent programs.

## Core APIs

- `Supervisor.track`: creates a supervisor that tracks child runtime fibers.
- `Effect.supervised(supervisor)`: supervises fibers forked within the wrapped effect.
- `supervisor.value`: retrieves current supervised state (for `track`, tracked runtime fibers).

## Monitoring Fiber Count

```ts
import { Effect, Fiber, FiberStatus, Schedule, Supervisor } from "effect"

const monitorFibers = (
  supervisor: Supervisor.Supervisor<Array<Fiber.RuntimeFiber<any, any>>>
): Effect.Effect<void> =>
  Effect.gen(function* () {
    const fibers = yield* supervisor.value
    console.log(`number of fibers: ${fibers.length}`)
  })

const program = Effect.gen(function* () {
  const supervisor = yield* Supervisor.track

  const fibFiber = yield* fib(20).pipe(
    Effect.supervised(supervisor),
    Effect.fork
  )

  const policy = Schedule.spaced("500 millis").pipe(
    Schedule.whileInputEffect(() =>
      Fiber.status(fibFiber).pipe(
        Effect.andThen((status) => status !== FiberStatus.done)
      )
    )
  )

  const monitorFiber = yield* monitorFibers(supervisor).pipe(
    Effect.repeat(policy),
    Effect.fork
  )

  yield* Fiber.join(monitorFiber)
  const result = yield* Fiber.join(fibFiber)
  console.log(`fibonacci result: ${result}`)
})
```

`fib(...)` is a recursive effect that forks child fibers; the supervisor captures those forks while the supervised computation runs.

## When to Use Supervisor

- monitor active child fibers during long-running computations
- debug excessive or unexpected fiber creation
- inspect supervised fiber state as part of observability tooling
