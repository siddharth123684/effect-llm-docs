---
title: Matching
description: Effect.match and matchCause for handling success and failure branches in one place.
---

# Matching

Source: extracted from `llms-full.txt` (`Matching`).

## Overview

The matching APIs in `Effect` let you handle success and failure in one place.

- Use pure matching when handlers return plain values.
- Use effectful matching when handlers must run effects (for example logging).
- Use `Cause`-based matching when you need full failure details (`Fail`, `Die`, `Interrupt`, and combinations).

## `Effect.match`

`Effect.match` transforms both branches of an effect into a single success value.

- `onSuccess` handles successful values.
- `onFailure` handles typed (expected) errors.

```ts
import { Effect } from "effect"

const success = Effect.succeed(42)
const failure = Effect.fail(new Error("Uh oh!"))

const asMessage = (self: Effect.Effect<number, Error>) =>
  Effect.match(self, {
    onFailure: (error) => `failure: ${error.message}`,
    onSuccess: (value) => `success: ${value}`
  })
```

## `Effect.ignore`

`Effect.ignore` runs an effect and discards both its success value and failure.

This is useful when you only care that side effects run and do not need to branch on the outcome.

```ts
import { Effect } from "effect"

const task = Effect.fail("Uh oh!").pipe(Effect.as(5))
const program = Effect.ignore(task)
// Effect<void, never, never>
```

## `Effect.matchEffect`

`Effect.matchEffect` is like `Effect.match`, but each handler returns an effect.

Use it when branch handling itself is effectful (for example `Effect.log`, metrics, notifications, or follow-up I/O).

```ts
import { Effect } from "effect"

const program = Effect.matchEffect(Effect.succeed(42), {
  onFailure: (error) => Effect.log(`failure: ${error.message}`),
  onSuccess: (value) => Effect.log(`success: ${value}`)
})
```

## `Effect.matchCause`

`Effect.matchCause` provides the full `Cause` on failure instead of only typed errors.

This is the right API when you need to distinguish:

- expected failures (`Fail`)
- defects (`Die`)
- interruptions (`Interrupt`)

```ts
import { Effect } from "effect"

const task: Effect.Effect<number, Error> = Effect.die("Boom!")

const program = Effect.matchCause(task, {
  onFailure: (cause) => {
    switch (cause._tag) {
      case "Fail":
        return `Fail: ${cause.error.message}`
      case "Die":
        return `Die: ${cause.defect}`
      case "Interrupt":
        return `${cause.fiberId} interrupted`
    }
    return "other cause"
  },
  onSuccess: (value) => `success: ${value}`
})
```

## `Effect.matchCauseEffect`

`Effect.matchCauseEffect` combines full-cause inspection with effectful handlers.

Use it when you need both detailed cause-based routing and side effects in each branch.

## Quick Selection

| Goal | API |
| --- | --- |
| Fold success and typed failure into one value | `Effect.match` |
| Ignore both value and error channels | `Effect.ignore` |
| Branch with effectful handlers | `Effect.matchEffect` |
| Branch on full failure cause | `Effect.matchCause` |
| Branch on full cause with effectful handlers | `Effect.matchCauseEffect` |
