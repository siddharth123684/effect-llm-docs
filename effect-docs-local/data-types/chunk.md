# Chunk

Source: extracted from `llms-full.txt` (`Chunk`).

## Overview

A `Chunk<A>` is an ordered, immutable collection of values of type `A`.
It is similar to an array, but optimized for operations like repeated
concatenation.

Use `Chunk` when you need those concatenation-friendly semantics. For cases
without repeated concatenation, plain arrays are often simpler and faster.

## Why Use Chunk?

- **Immutability**: data cannot be mutated after creation.
- **Performance for composition**: operations like combining chunks are
  optimized compared to repeatedly concatenating arrays.

## Creating a Chunk

| API | What it does | Notes |
| --- | --- | --- |
| `Chunk.empty<A>()` | Creates an empty chunk | Type-safe empty value |
| `Chunk.make(...values)` | Creates a non-empty chunk from values | Returns `NonEmptyChunk` |
| `Chunk.fromIterable(iterable)` | Builds a chunk from iterable input | Clones elements into a new chunk |
| `Chunk.unsafeFromArray(array)` | Wraps an array without cloning | Faster, but can violate immutability if array is mutated later |

```ts
import { Chunk, List } from "effect"

const empty = Chunk.empty<number>()
const nonEmpty = Chunk.make(1, 2, 3)
const fromArray = Chunk.fromIterable([1, 2, 3])
const fromList = Chunk.fromIterable(List.make(1, 2, 3))
const unsafe = Chunk.unsafeFromArray([1, 2, 3])
```

## Common Operations

| API | Purpose |
| --- | --- |
| `Chunk.appendAll(left, right)` | Concatenate two chunks |
| `Chunk.drop(chunk, n)` | Drop `n` values from the start |
| `Equal.equals(a, b)` | Structural equality check |
| `Chunk.toReadonlyArray(chunk)` | Convert to `ReadonlyArray` |

```ts
import { Chunk, Equal } from "effect"

const combined = Chunk.appendAll(Chunk.make(1, 2), Chunk.make(3, 4))
const dropped = Chunk.drop(combined, 1)
const same = Equal.equals(dropped, Chunk.make(2, 3, 4))
const array = Chunk.toReadonlyArray(dropped)
```
