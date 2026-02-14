# Runtime

Source: extracted from `llms-full.txt` (`Runtime`).

## Overview

`Runtime<R>` is the system that executes effects requiring `R`.
An `Effect` value is only a description (a blueprint); the runtime interprets that description and runs it.

A runtime includes:

- `Context<R>` for required services
- `FiberRefs` for fiber-local state and defaults
- `RuntimeFlags` for execution behavior (for example interruption and cooperative yielding)

## What the Runtime Does

At execution time, the runtime creates a root fiber and evaluates effect instructions step by step.
It is responsible for:

- executing the effect program to completion
- handling expected and unexpected errors
- managing concurrency and forked fibers
- cooperative yielding
- running finalizers and cleaning up resources
- bridging async callbacks into effect execution

Conceptually:

```text
Context<R> + Effect<A, E, R> -> Runtime -> Exit<A, E>
```

## Default Runtime

`Effect.run*` helpers use `Runtime.defaultRuntime` under the hood.
So `Effect.runPromise(program)` is equivalent to `Runtime.runPromise(Runtime.defaultRuntime)(program)`.

The default runtime contains:

- empty context
- default `FiberRefs` (including default services)
- default runtime flags

For many applications, this is enough.

## Locally Scoped Runtime Configuration

Runtime configuration is inherited by child workflows by default.
When you need a temporary override, provide configuration only around the target region with `Effect.provide`.

```ts
import { Effect, Logger } from "effect"

const addSimpleLogger = Logger.replace(
  Logger.defaultLogger,
  Logger.make(({ message }) => console.log(message))
)

const program = Effect.log("hello").pipe(Effect.provide(addSimpleLogger))
```

After that region completes, the previous runtime configuration continues.

## ManagedRuntime

Use `ManagedRuntime.make(layer)` when you want a top-level, reusable runtime built from layers.
This is useful for application boundaries and framework integrations where runtime setup should be shared.

```ts
import { Effect, Logger, ManagedRuntime } from "effect"

const appLayer = Logger.replace(
  Logger.defaultLogger,
  Logger.make(({ message }) => console.log(message))
)

const runtime = ManagedRuntime.make(appLayer)
runtime.runSync(Effect.log("Application started!"))

// Clean up managed resources when done
Effect.runFork(runtime.disposeEffect)
```

Always dispose a managed runtime when it is no longer needed.

## `Effect.Tag` and Integration Pattern

`Effect.Tag` helps model services used by effects and makes service access ergonomic.
With a service layer, you can build a `ManagedRuntime` and run effects in hosts like React or other external frameworks while keeping service lifecycle explicit.
