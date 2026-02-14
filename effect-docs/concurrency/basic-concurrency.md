---
title: Basic Concurrency
description: Effect's concurrency model with fibers, concurrency options, racing APIs, and interruption.
---

# Basic Concurrency

Source: extracted from `llms-full.txt` (`Basic Concurrency`).

## Overview

Effect executes concurrent work using fibers. Many combinators (for example
`Effect.all`, `Effect.forEach`, and racing operators) accept concurrency
configuration so you can control throughput, fairness, and interruption
behavior.

## Concurrency Option

Concurrency-aware APIs use an option shaped like:

```ts
type Options = {
  readonly concurrency?: number | "unbounded" | "inherit"
}
```

Behavior by mode:

- Omitted (`undefined`): sequential execution (one effect starts after the
  previous one completes).
- `number`: run up to that many effects at once.
- `"unbounded"`: no explicit limit.
- `"inherit"`: use concurrency from surrounding context (set with
  `Effect.withConcurrency`), defaulting to unbounded when no context exists.

```ts
import { Effect } from "effect"

const limited = Effect.all([task1, task2, task3, task4], {
  concurrency: 2
})

const inherited = Effect.all([task1, task2, task3], {
  concurrency: "inherit"
}).pipe(Effect.withConcurrency(2))
```

## Interruption Basics

`Effect.interrupt` interrupts the current running fiber. You can attach cleanup
logic with `Effect.onInterrupt(...)`.

```ts
import { Console, Effect } from "effect"

const worker = Effect.sleep("2 seconds").pipe(
  Effect.onInterrupt(() => Console.log("cleanup completed"))
)
```

In concurrent compositions, interruption often propagates:

- if one branch interrupts in a parallel workflow, sibling fibers can be
  interrupted too
- interruption/failure details are reflected in `Cause` when inspected through
  `Exit` (for example with `Effect.runPromiseExit`)

## Racing APIs

### `Effect.race`

Runs two effects concurrently and returns the first successful result.
The losing effect is interrupted.

- if both fail, the result fails with a combined `Cause`
- if you need to preserve first completion whether success or failure, race
  `Effect.either(...)` values instead

### `Effect.raceAll`

Races many effects and returns the first success, interrupting the rest.
If none succeed, it fails with the last encountered error.

### `Effect.raceFirst`

Returns whichever effect completes first, success or failure.
Unlike `race`, this does not wait for a success specifically.

`raceFirst` waits for loser interruption/finalization by default. For quicker
return, wrap contestants with `Effect.disconnect(...)` so interruption cleanup
continues in the background.

### `Effect.raceWith`

Runs two effects concurrently and lets you handle completion through two
finisher callbacks (`onSelfDone` / `onOtherDone`) that receive `Exit` values.
Use this when you need custom logic based on whichever side finishes first.

## Practical Guidance

- use bounded numeric concurrency for predictable resource usage
- use `"inherit"` to let higher-level orchestration define policy
- attach `onInterrupt` handlers around long-running effects that need cleanup
- choose race operators by winner semantics:
  - first success: `race` / `raceAll`
  - first completion (success or failure): `raceFirst`
  - custom completion handling: `raceWith`
