---
title: Dual APIs
description: Data-last and data-first calling styles in Effect for pipeline and direct usage.
---

# Dual APIs

Source: extracted from `llms-full.txt` (`Dual APIs`).

## Overview

In Effect, many functions are exposed in two equivalent calling styles:

- **data-last**: pass non-data arguments first, then provide `self` later
- **data-first**: pass `self` first, then the remaining arguments

When both styles are available for the same function, the API is called a
**dual API**.

## Example: `Effect.map`

`Effect.map` is a dual API with two TypeScript overloads:

```ts
declare const map: {
  <A, B>(f: (a: A) => B): <E, R>(self: Effect<A, E, R>) => Effect<B, E, R>
  <A, E, R, B>(self: Effect<A, E, R>, f: (a: A) => B): Effect<B, E, R>
}
```

Both overloads perform the same transformation. The difference is only how
arguments are supplied.

## Data-last style

Data-last is convenient in pipelines:

```ts
import { Effect, pipe } from "effect"

const mapped = pipe(effect, Effect.map(f))
```

This style reads well when chaining several operations:

```ts
pipe(effect, Effect.map(f1), Effect.map(f2))
```

## Data-first style

Data-first is convenient for a single direct call:

```ts
import { Effect } from "effect"

const mapped = Effect.map(effect, f)
```

## Choosing Between Styles

- Use **data-last** when composing steps with `pipe`.
- Use **data-first** for simple one-off transformations.
- Prefer whichever style improves readability for your team.
