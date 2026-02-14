# Retrying

Source: extracted from `llms-full.txt` (`Retrying`).

## Overview

Retrying handles transient failures by running a failing effect again according to a policy.

In Effect, retry behavior is typically controlled with a `Schedule` (for delays, spacing, and retry limits) or with simple retry options.

## `Effect.retry`

`Effect.retry(effect, policy)` retries a failing effect using the provided policy.

- If a retry eventually succeeds, its success value is returned.
- If retries are exhausted, the last failure is propagated.

### Retry with a fixed delay

```ts
import { Effect, Schedule } from "effect"

let attempts = 0

const task = Effect.suspend(() =>
  attempts < 3 ? Effect.fail(`failure-${++attempts}`) : Effect.succeed("yay!")
)

const program = Effect.retry(task, Schedule.fixed("100 millis"))
```

### Retry immediately a fixed number of times

Use `{ times: n }` when you want quick retries without defining a full schedule:

```ts
import { Effect } from "effect"

const alwaysFail = Effect.fail("temporary")
const retried = Effect.retry(alwaysFail, { times: 5 })
```

### Retry based on error conditions

You can control stop/continue behavior with predicates:

- `until`: retry until the predicate becomes `true` for the current error.
- `while`: retry while the predicate stays `true`.

```ts
import { Effect } from "effect"

let n = 0
const flaky = Effect.failSync(() => `Error ${++n}`)

const program = Effect.retry(flaky, {
  until: (error) => error === "Error 3"
})
```

## `Effect.retryOrElse`

`Effect.retryOrElse(effect, policy, orElse)` retries using the policy, then runs a fallback effect if retries are exhausted.

Use this when repeated failure should gracefully transition to a default value or recovery action.

```ts
import { Effect, Schedule } from "effect"

const task = Effect.fail("temporary")

const program = Effect.retryOrElse(
  task,
  Schedule.recurs(2),
  () => Effect.succeed("default value")
)
```

## Quick Selection

| Goal | Operator |
| --- | --- |
| Retry with full policy control | `Effect.retry` + `Schedule` |
| Retry immediately up to `n` times | `Effect.retry` with `{ times: n }` |
| Stop/retry based on error values | `Effect.retry` with `until` / `while` |
| Fallback after retries are exhausted | `Effect.retryOrElse` |
