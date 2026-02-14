---
title: Why Effect?
description: Rationale for Effect's explicit modeling of errors, dependencies, and success values in types.
---

# Why Effect?

Source: extracted from `llms-full.txt` (`Why Effect?`).

## Overview

Effect is a TypeScript ecosystem for building reliable applications by making program behavior explicit in types.
Instead of only modeling successful return values, Effect also models:

- expected errors
- required dependencies/context
- success values

This improves maintainability as codebases grow.

## The Problem with Hidden Exceptions

In standard TypeScript, thrown errors are not visible in a function signature:

```ts
const divide = (a: number, b: number): number => {
  if (b === 0) {
    throw new Error("Cannot divide by zero")
  }
  return a / b
}
```

From the type alone, callers cannot tell this function may fail.

## The Effect Pattern

Effect encodes outcomes directly in the type:

```ts
import { Effect } from "effect"

const divide = (
  a: number,
  b: number
): Effect.Effect<number, Error, never> =>
  b === 0
    ? Effect.fail(new Error("Cannot divide by zero"))
    : Effect.succeed(a / b)
```

`Effect.Effect<number, Error, never>` means:

- success type: `number`
- error type: `Error`
- requirements type: `never` (no dependencies required)

This gives the compiler more information to help prevent mistakes.

## Why Tracking Context Matters

The requirements type (`R`) models dependencies without threading them through every function argument.
This supports cleaner architecture and easier testing, because live services can be swapped for test implementations.

## Ecosystem Benefits

Effect provides integrated solutions for common application concerns, including:

- error handling and debugging
- async and concurrency workflows
- retries and scheduling
- streaming and caching
- resource management and tracing

Using one cohesive toolkit reduces ad hoc patterns and dependency sprawl.

## Practical Adoption

Effect is designed as a practical toolkit for everyday TypeScript work.
You can adopt it incrementally: start with the pieces that solve immediate problems, then expand as needed.
