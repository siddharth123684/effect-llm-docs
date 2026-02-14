---
title: The Effect Type
description: Learn how the Effect type models workflows lazily with explicit Success, Error, and Requirements type parameters for type-safe, composable programs.
---

# The Effect Type

Source: extracted from `llms-full.txt` (`The Effect Type`).

## Overview

The `Effect` type is a **lazy** description of a workflow. Creating an effect does not run it immediately - it describes a program that can:

- succeed with a value
- fail with an expected error
- require contextual dependencies

General shape:

```text
         Success  Error  Requirements
            v      v         v
Effect<Success, Error, Requirements>
```

Conceptually, it can be thought of like:

```ts
type Effect<Success, Error, Requirements> = (
  context: Context<Requirements>
) => Error | Success
```

Effects are not actually functions. They can model synchronous, asynchronous, concurrent, and resourceful computations.

## Key Properties

- **Immutability**: effect values are immutable; combinators produce new effects.
- **Modeling interactions**: effects describe interactions, they do not perform them on creation.
- **Execution**: the Effect runtime interprets effects and performs real-world actions.

## Type Parameters

| Parameter        | Meaning |
| ---------------- | ------- |
| **Success**      | The value type produced on success. `void` means no meaningful value. `never` can indicate non-termination (unless it fails). |
| **Error**        | The expected error type. `never` means the effect cannot fail. |
| **Requirements** | Context/dependencies needed to run the effect. `never` means no dependencies are required. |

## Common Abbreviations

In Effect docs and APIs, these type parameters are often abbreviated:

- `A` = Success
- `E` = Error
- `R` = Requirements

## Extracting Inferred Types

You can extract success, error, and context types from an existing effect with utility types.

```ts
import { Context, Effect } from "effect"

class SomeContext extends Context.Tag("SomeContext")<SomeContext, {}>() {}

declare const program: Effect.Effect<number, Error, SomeContext>

type A = Effect.Effect.Success<typeof program>
type E = Effect.Effect.Error<typeof program>
type R = Effect.Effect.Context<typeof program>
```

## Why This Matters

`Effect<Success, Error, Requirements>` makes program behavior explicit at the type level, so you can reason about outputs, failures, and dependencies before runtime.

## Next Step

Continue with creating and composing effects, then running them at your application's runtime boundary.
