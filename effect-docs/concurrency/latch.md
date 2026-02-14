# Latch

Source: extracted from `llms-full.txt` (`Latch`).

## Overview

A `Latch` is a synchronization gate for fibers. It has two states:

- closed: fibers wait
- open: fibers continue immediately

This is useful when multiple fibers must pause until a setup step is done
(for example, service initialization). Once setup completes, opening the latch
lets waiting fibers proceed.

## Latch Interface

`Latch` provides these operations:

| Operation | Description |
| --- | --- |
| `whenOpen` | Runs an effect only when the latch is open; waits otherwise. |
| `open` | Opens the latch and allows waiting fibers to proceed. |
| `close` | Closes the latch so future arrivals wait. |
| `await` | Suspends until latch is open; returns immediately if already open. |
| `release` | Releases waiting fibers without permanently opening the latch. |

## Creating a Latch

Use `Effect.makeLatch` with an optional initial boolean state.
If omitted, it defaults to `false` (closed).

```ts
import { Console, Effect } from "effect"

const program = Effect.gen(function* () {
  const latch = yield* Effect.makeLatch() // closed by default

  const fiber = yield* Console.log("open sesame").pipe(
    latch.whenOpen,
    Effect.fork
  )

  yield* Effect.sleep("1 second")
  yield* latch.open
  yield* fiber.await
})

Effect.runFork(program)
```

## Latch vs Semaphore

Use a latch when progress depends on a gating event ("wait until ready").

Use a semaphore with one permit when you need mutual exclusion (only one fiber
in a critical section at a time).
