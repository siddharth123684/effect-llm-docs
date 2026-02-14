# Two Types of Errors

Source: extracted from `llms-full.txt` (`Two Types of Errors`).

## Overview

Effect distinguishes between two failure categories and preserves detailed failure information at runtime.

In practice, this gives you clearer diagnostics and a better model of which errors belong to normal control flow versus which indicate defects.

## Expected Errors

Expected errors (also called failures, typed errors, or recoverable errors):

- are anticipated during normal execution
- are part of the program's domain/control flow
- are tracked at the type level in the `Error` channel of `Effect<A, E, R>`

Example:

```ts
const program: Effect<string, HttpError, never>
```

From the type, you can see the effect may fail with `HttpError`.

## Unexpected Errors

Unexpected errors (also called defects, untyped errors, or unrecoverable errors):

- are not anticipated during normal execution
- are outside intended domain behavior
- are not tracked in the `Error` type parameter

Effect still records these failures in the runtime and exposes tools to inspect/report them at system boundaries.

## Quick Comparison

| Category | Anticipated? | Typed in `E`? | Typical role |
| --- | --- | --- | --- |
| Expected errors | Yes | Yes | Domain and control-flow failures |
| Unexpected errors | No | No | Defects/bugs outside normal flow |

## Practical Takeaway

Model business-level, recoverable failures in the typed error channel. Treat defects as unexpected runtime failures and handle them at application boundaries for diagnostics and reporting.
