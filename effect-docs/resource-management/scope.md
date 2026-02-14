---
title: Scope
description: How Scope manages resource lifetimes and finalizers in Effect, including acquireRelease and manual control.
---

# Scope

Source: extracted from `llms-full.txt` (`Scope`).

## Overview

`Scope` is Effect's resource-lifetime boundary.
A scope owns resources and finalizers. When the scope closes, finalizers run and resources are released.

Finalizers execute in reverse registration order (LIFO). This ensures dependent resources are cleaned up safely (for example, close a file before closing the network connection it depends on).

## Adding Finalizers

Use `Effect.addFinalizer` to register cleanup logic in the current scope.
The finalizer receives an `Exit` value, so cleanup can react to whether execution ended in success, failure, or interruption.

```ts
import { Console, Effect } from "effect"

const program = Effect.gen(function* () {
  yield* Effect.addFinalizer((exit) =>
    Console.log(`Finalizer ran with: ${exit._tag}`)
  )
  return "done"
})

const runnable = Effect.scoped(program)
```

Key point: finalizers run on all termination paths, not only success.

## Manual Scope Control

By default, scoped operations in one workflow share a merged scope.
For finer control, create scopes manually with `Scope.make()`, attach effects with `Scope.extend(scope)`, then close each scope explicitly with `Scope.close(scope, exit)`.

This lets you control exactly when each resource group is released.

### Closing Scopes and Pending Tasks

Closing a scope does not automatically interrupt already-running work tied to that scope.
If a task is still running, it can continue and run its finalizer when it completes.

## Defining Resources with `Effect.acquireRelease`

`Effect.acquireRelease(acquire, release)` is the core resource constructor:

- acquisition is uninterruptible
- successful acquisition guarantees `release` runs when the scope closes
- the resulting effect requires `Scope`

```ts
import { Effect } from "effect"

const resource = Effect.acquireRelease(acquire, release)
// resource: Effect.Effect<Resource, Error, Scope>
```

Use `Effect.scoped(...)` to provide and close the scope automatically.

## Rollback Pattern with Scoped Resources

A common pattern is to compose multiple `acquireRelease` steps where each release handler rolls back only on failure (for example, delete created infrastructure if later steps fail).

This enables transaction-like behavior for external systems:

- each step acquires one resource
- release handlers inspect `Exit`
- on failure, earlier successful steps are rolled back in reverse order

## Core APIs

| API | Purpose |
| --- | --- |
| `Scope.make` | Create a new scope manually |
| `Scope.addFinalizer` / `Effect.addFinalizer` | Register cleanup logic |
| `Scope.extend` | Attach a scoped effect to a larger/manual scope |
| `Scope.close` | Close scope and trigger finalization |
| `Effect.acquireRelease` | Define acquire/release resource lifecycle |
| `Effect.scoped` | Provide a scope and close it automatically |
