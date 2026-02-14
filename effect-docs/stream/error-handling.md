---
title: Error Handling
description: Stream error recovery, fallback, retry, timeout, and typed error handling.
---

# Error Handling

Source: extracted from `llms-full.txt` (`Error Handling in Streams`).

## Overview

`Stream` error handling lets you recover from typed failures, handle full causes (including defects), run cleanup logic, retry transient failures, and apply timeout behavior.

## Recovering from Failure

Use fallback streams when the current stream fails:

- `Stream.orElse`: switch to an alternative stream.
- `Stream.orElseEither`: switch and keep source information with `Either`.

```ts
import { Effect, Stream } from "effect"

const primary = Stream.make(1, 2, 3).pipe(Stream.concat(Stream.fail("boom")))
const fallback = Stream.make(10, 20, 30)

const recovered = Stream.orElse(primary, () => fallback)

Effect.runPromise(Stream.runCollect(recovered))
```

## Error-Aware Recovery

When fallback depends on the error value, use:

- `Stream.catchAll`: recover from all typed failures.
- `Stream.catchSome`: recover only selected failures via `Option`.

```ts
import { Effect, Option, Stream } from "effect"

const source = Stream.fail("retryable" as const)

const selective = Stream.catchSome(source, (error) =>
  error === "retryable" ? Option.some(Stream.succeed("ok")) : Option.none()
)

Effect.runPromise(Stream.runCollect(selective))
```

## Recovering from Causes and Defects

For full failure information (`Cause`), including defects:

- `Stream.catchAllCause`: recover from any cause.
- `Stream.catchSomeCause`: recover from matching causes only.

These are useful when failures are not limited to typed `E` errors (for example, defects from `Stream.die*`).

## Running Cleanup on Failure

`Stream.onError` lets you run an effect when the stream fails (for logging, cleanup, or notifications). It does not replace recovery operators; it adds failure-side effects.

## Retrying Failing Streams

Use `Stream.retry(schedule)` for transient failures. The stream is re-run following the provided `Schedule` policy (for example, exponential backoff).

## Refining Errors

`Stream.refineOrDie` keeps only selected typed errors and escalates unmatched errors to defects.

Use this when you want a narrower error channel and treat all other failures as unrecoverable.

## Timeout Strategies

Streams support multiple timeout behaviors:

- `Stream.timeout(duration)`: end the stream if no value arrives in time.
- `Stream.timeoutFail(error, duration)`: fail with a custom error.
- `Stream.timeoutFailCause(cause, duration)`: fail with a custom cause.
- `Stream.timeoutTo(duration, fallbackStream)`: switch to a fallback stream.

## Quick API Map

| Goal | API |
| --- | --- |
| Fallback stream on failure | `Stream.orElse` |
| Fallback with source tagging | `Stream.orElseEither` |
| Recover using error value | `Stream.catchAll` |
| Recover only some typed errors | `Stream.catchSome` |
| Recover from full cause/defects | `Stream.catchAllCause`, `Stream.catchSomeCause` |
| Run cleanup/logging on failure | `Stream.onError` |
| Retry transient failures | `Stream.retry` |
| Keep subset of errors, die on others | `Stream.refineOrDie` |
| Apply timeout behavior | `Stream.timeout*`, `Stream.timeoutTo` |
