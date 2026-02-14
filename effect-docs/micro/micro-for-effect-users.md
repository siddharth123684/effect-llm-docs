---
title: Micro for Effect Users
description: Compare Effect vs Micro APIs including Micro.service, Micro.provideContext, MicroExit, MicroCause, and MicroScope when migrating Effect-style code to the lightweight Micro runtime.
---

# Micro for Effect Users

Source: extracted from `llms-full.txt` (`Micro for Effect Users`).

## Overview

`Micro` is an experimental, lightweight subset of Effect for bundle-size-sensitive code.

- Baseline footprint is about 5kb gzipped (can increase based on features used).
- Useful for libraries that want Effect-style composition while exposing Promise-oriented APIs.
- Supports mixed architectures (for example, Micro on clients and full Effect on servers).

Compared with full Effect, Micro intentionally omits heavier features like `Layer`, `Ref`, `Queue`, and `Deferred`.

Using major runtime-heavy Effect modules (beyond lightweight data modules like `Option`, `Either`, and `Array`) can pull in runtime code and reduce Micro's bundle-size benefit.

## Importing Micro

```ts
import { Micro } from "effect"
```

```ts
import * as Micro from "effect/Micro"
```

Both imports work. Namespace import can be safer for tree shaking when deep scope analysis is limited. The source specifically calls out Rollup and Webpack 5+ as supporting deep scope analysis.

## Main Types

### Micro

`Micro<Success, Error, Requirements>` mirrors the same three type parameters as `Effect`.

### MicroExit

`MicroExit<A, E>` is the Micro outcome type:

```ts
type MicroExit<A, E> = MicroExit.Success<A, E> | MicroExit.Failure<A, E>
```

### MicroCause

`MicroCause<E>` is a reduced cause model:

```ts
type MicroCause<E> = Die | Fail<E> | Interrupt
```

| Variant | Meaning |
| --- | --- |
| `Die` | Unhandled defect. |
| `Fail<E>` | Expected/application error. |
| `Interrupt` | Interrupted computation. |

### MicroSchedule

```ts
type MicroSchedule = (attempt: number, elapsed: number) => Option<number>
```

The function returns the delay for the next repetition attempt. Returning `None` stops repetition.

## Effect vs Micro API Notes

Status meanings used below:

- `different`: feature exists but differs in behavior and/or types
- `micro-only`: available in Micro with no direct Effect entry in this source table

## Creating Effects

| Effect | Micro | Status | Notes |
| --- | --- | --- | --- |
| `Effect.try` | `Micro.try` | different | Requires a `try` block. |
| `Effect.tryPromise` | `Micro.tryPromise` | different | Requires a `try` block. |
| `Effect.sleep` | `Micro.sleep` | different | Milliseconds only. |
| `Effect.failCause` | `Micro.failWith` | different | Uses `MicroCause` instead of `Cause`. |
| `Effect.failCauseSync` | `Micro.failWithSync` | different | Uses `MicroCause` instead of `Cause`. |
| - | `Micro.make` | micro-only | - |
| - | `Micro.fromOption` | micro-only | - |
| - | `Micro.fromEither` | micro-only | - |

## Running Effects

| Effect | Micro | Status | Notes |
| --- | --- | --- | --- |
| `Effect.runSyncExit` | `Micro.runSyncExit` | different | Returns `MicroExit` instead of `Exit`. |
| `Effect.runPromiseExit` | `Micro.runPromiseExit` | different | Returns `MicroExit` instead of `Exit`. |
| `Effect.runFork` | `Micro.runFork` | different | Returns `MicroFiber` instead of `RuntimeFiber`. |

`Micro.runFork` returns a `MicroFiber` that can be observed:

```ts
import { Micro } from "effect"

const fiber = Micro.succeed(42).pipe(Micro.delay(1000), Micro.runFork)
fiber.addObserver(console.log)
```

## Building Pipelines

| Effect | Micro | Status | Notes |
| --- | --- | --- | --- |
| `Effect.andThen` | `Micro.andThen` | different | No `Promise` / `() => Promise` argument handling. |
| `Effect.tap` | `Micro.tap` | different | No `() => Promise` argument handling. |
| `Effect.all` | `Micro.all` | different | No `batching` or `mode` options. |
| `Effect.forEach` | `Micro.forEach` | different | No `batching` option. |
| `Effect.filter` | `Micro.filter` | different | No `batching` option. |
| `Effect.filterMap` | `Micro.filterMap` | different | Filter is effectful. |

## Errors, Timeouts, and Sandboxing

| Effect | Micro | Status | Notes |
| --- | --- | --- | --- |
| `Effect.exit` | `Micro.exit` | different | Returns `MicroExit` instead of `Exit`. |
| - | `Micro.catchCauseIf` | micro-only | - |
| - | `Micro.timeoutOrElse` | micro-only | - |
| `Effect.sandbox` | `Micro.sandbox` | different | Uses `MicroCause<E>` instead of `Cause<E>`. |
| - | `Micro.filterOrFailWith` | micro-only | - |
| `Effect.tapErrorCause` | `Micro.tapErrorCause` | different | Works with `MicroCause<E>`. |
| - | `Micro.tapCauseIf` | micro-only | - |
| `Effect.tapDefect` | `Micro.tapDefect` | different | Uses `unknown` rather than `Cause<never>`. |

## Requirements Management

When using `Micro.gen`, access services through `Micro.service(...)`:

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
```

Related API differences:

| Effect | Micro | Status | Notes |
| --- | --- | --- | --- |
| `Effect.provide` | `Micro.provideContext` | different | Handles `Context` only. |
| - | `Micro.provideScope` | micro-only | - |
| - | `Micro.service` | micro-only | - |

## Scope, Resources, and Finalization

| Effect | Micro | Status | Notes |
| --- | --- | --- | --- |
| `Scope` | `MicroScope` | different | Type differs from `Scope`. |
| `Scope.make` | `Micro.scopeMake` | different | Returns `MicroScope`. |
| `Effect.addFinalizer` | `Micro.addFinalizer` | different | Uses `MicroExit`; no `R`. |
| `Effect.acquireRelease` | `Micro.acquireRelease` | different | Uses `MicroExit`. |
| `Effect.acquireUseRelease` | `Micro.acquireUseRelease` | different | Uses `MicroExit`. |
| `Effect.onExit` | `Micro.onExit` | different | Uses `MicroExit`. |
| `Effect.onError` | `Micro.onError` | different | Uses `MicroCause`. |
| - | `Micro.onExitIf` | micro-only | - |

## Retrying, Repetition, and Concurrency

| Effect | Micro | Status | Notes |
| --- | --- | --- | --- |
| `Effect.retry` | `Micro.retry` | different | Different options. |
| `Effect.repeat` | `Micro.repeat` | different | Different options. |
| - | `Micro.repeatExit` | micro-only | - |
| `Effect.fork` | `Micro.fork` | different | Returns `MicroFiber`. |
| `Effect.forkDaemon` | `Micro.forkDaemon` | different | Returns `MicroFiber`. |
| `Effect.forkIn` | `Micro.forkIn` | different | Returns `MicroFiber`. |
| `Effect.forkScoped` | `Micro.forkScoped` | different | Returns `MicroFiber`. |
