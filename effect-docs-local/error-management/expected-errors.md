# Expected Errors

Source: extracted from `llms-full.txt` (`Expected Errors`).

## Overview

Expected errors are failures you model as part of normal program behavior. In Effect, they are tracked in the `Error` type parameter:

```text
Effect<Success, Error, Requirements>
```

This makes error behavior explicit at compile time.

## Modeling Expected Errors

Use `Effect.fail` to produce expected failures, and prefer tagged errors for precise matching.

```ts
import { Data, Effect } from "effect"

class HttpError extends Data.TaggedError("HttpError")<{}> {}

const program: Effect.Effect<string, HttpError> = Effect.gen(function* () {
  return yield* Effect.fail(new HttpError())
})
```

`Data.TaggedError` adds a `_tag` discriminator, which makes selective recovery straightforward.

## Error Tracking and Short-Circuiting

- Error types are combined as unions when multiple failure paths exist.
- Sequential composition (`Effect.gen`, `flatMap`, `andThen`, `map`) short-circuits on the first failure.
- As errors are handled, the `Error` type narrows (for example from `HttpError | ValidationError` to `ValidationError`, or to `never` when fully recovered).

## Catching All Errors

| API | Resulting error channel | Use when |
| --- | --- | --- |
| `Effect.either` | `never` | Represent success/failure as `Either` and branch manually. |
| `Effect.option` | `never` | Convert expected failure to `Option.none`; keep going without failing. |
| `Effect.catchAll` | usually `never` if fully recovered | Recover from all expected (recoverable) errors. |
| `Effect.catchAllCause` | depends on recovery | Recover using full `Cause` (including defects). |

`catchAllCause` is the broadest handler because it receives the full failure cause, not just typed expected errors.

## Catching Specific Errors

| API | Behavior |
| --- | --- |
| `Effect.catchSome` | Conditionally recover by returning `Option.some(effect)` or `Option.none()`. |
| `Effect.catchIf` | Recover using a predicate (or type guard). |
| `Effect.catchTag` | Recover one tagged error by `_tag`. |
| `Effect.catchTags` | Recover multiple tagged errors in one call. |

Example with tag-based recovery:

```ts
import { Data, Effect } from "effect"

class HttpError extends Data.TaggedError("HttpError")<{}> {}
class ValidationError extends Data.TaggedError("ValidationError")<{}> {}

declare const program: Effect.Effect<string, HttpError | ValidationError>

const recovered = program.pipe(
  Effect.catchTag("HttpError", () => Effect.succeed("Recovered HTTP error"))
)
// recovered: Effect.Effect<string, ValidationError>
```

## TypeScript Note (`catchIf`)

- In TypeScript >= 5.5, `catchIf` commonly narrows remaining error types.
- In older versions, use a user-defined type guard predicate if you need reliable narrowing.

## Effect.fn and Error Tracing

`Effect.fn` helps define effectful functions with improved tracing:

- better stack traces around failures
- optional span creation for observability when a span name is provided

This is useful when debugging and operating effectful code that may fail.
