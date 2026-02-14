# Fibers

Source: extracted from `llms-full.txt` (`Fibers`).

## Overview

Fibers are lightweight virtual threads managed by the Effect runtime.
Every running `Effect` executes on a fiber, including the main program fiber.

Fibers provide:

- low-overhead concurrency
- cooperative scheduling
- resource-safe interruption
- explicit handles for coordination (`Fiber`)

## Effect vs Fiber

An `Effect` is a lazy, immutable description of work.
A `Fiber` is the running execution of that work.

`Fiber` shape:

```text
Fiber<Success, Error>
```

Unlike `Effect<A, E, R>`, fibers do not carry a requirements type because required services are already provided at execution time.

## Forking Effects

Use `Effect.fork` to start an effect in a new fiber and get a handle to it:

```ts
import { Effect, Fiber } from "effect"

const program = Effect.gen(function* () {
  const fiber = yield* Effect.fork(Effect.succeed(42))
  const value = yield* Fiber.join(fiber)
  console.log(value) // 42
})

Effect.runFork(program)
```

## Waiting for Fiber Results

## `Fiber.join`

- waits for completion
- returns the success value
- fails if the fiber fails

## `Fiber.await`

- waits for completion
- returns `Exit<A, E>`
- preserves full termination detail (success, failure, interruption)

Use `join` for value-oriented composition, and `await` when you need explicit completion diagnostics.

## Interruption Model

Effect uses asynchronous interruption with interruptibility control around critical regions.
When a fiber is interrupted, finalizers run so resources are released safely.

Important APIs:

- `Fiber.interrupt(fiber)`: requests interruption and waits for full termination (back-pressured)
- `Fiber.interruptFork(fiber)`: interruption in background (do not wait)
- `Effect.interrupt`: high-level self-interruption API

## Composing Fibers

Fiber handles can be combined directly:

- `Fiber.zip` / `Fiber.zipWith` to combine results from multiple fibers
- `Fiber.orElse` to provide a fallback fiber if the first fails

These operations let you coordinate already-forked computations without re-modeling them as a single effect first.

## Lifetime of Child Fibers

Effect supports multiple lifetime strategies:

| API | Lifetime behavior |
| --- | --- |
| `Effect.fork` | Child is supervised by parent (structured concurrency). |
| `Effect.forkDaemon` | Child runs in global scope (daemon), independent of parent lifetime. |
| `Effect.forkScoped` | Child is tied to local scope and can outlive parent fiber. |
| `Effect.forkIn(scope)` | Child is tied to a specific target scope. |

Structured concurrency is the default: ordinary `fork` children are cleaned up with their parent.

## When Fibers Run

Forked fibers start after the current fiber yields or completes its current step.
This means immediate updates in the current fiber may happen before a newly forked fiber begins observing state.

In practice:

- use `Effect.yieldNow()` or a short `Effect.sleep(...)` to give forked work a chance to start
- do not rely on exact start timing; fiber scheduling is non-deterministic

## Key APIs

- `Effect.fork`
- `Effect.forkDaemon`
- `Effect.forkScoped`
- `Effect.forkIn`
- `Fiber.join`
- `Fiber.await`
- `Fiber.interrupt`
- `Fiber.interruptFork`
- `Fiber.zip`
- `Fiber.orElse`
