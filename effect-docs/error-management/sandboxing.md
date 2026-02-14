---
title: Sandboxing
description: Learn Effect.sandbox to expose full Cause structure including Fail, Die, and Interrupt for comprehensive failure inspection and error handling in Effect.
---

# Sandboxing

Source: extracted from `llms-full.txt` (`Sandboxing`).

## Overview

`Effect.sandbox` exposes the full failure cause of an effect, not just typed expected errors.
This lets you inspect and handle failures, defects, interruptions, or combined causes in one place.

## `Effect.sandbox`

Sandboxing transforms:

```text
Effect<A, E, R> -> Effect<A, Cause<E>, R>
```

After sandboxing, the error channel contains `Cause<E>`, so handlers can match on cause variants such as:

- `Fail` (expected failure)
- `Die` (defect)
- `Interrupt` (fiber interruption)

## `Effect.unsandbox`

Use `Effect.unsandbox` to convert a sandboxed effect back to the original error channel shape after cause-aware handling.

## Minimal Example

```ts
import { Console, Effect } from "effect"

const task = Effect.fail(new Error("Oh uh!")).pipe(Effect.as("primary result"))

const sandboxed = Effect.sandbox(task)

const handled = Effect.catchTags(sandboxed, {
  Fail: (cause) =>
    Console.log(`expected failure: ${cause.error.message}`).pipe(
      Effect.as("fallback result")
    )
})

const main = Effect.unsandbox(handled)
```

## When to Use

- you need visibility into full causes, not only typed errors
- you want one recovery point for failures, defects, and interruptions
- you need cause-aware diagnostics before translating back with `unsandbox`
