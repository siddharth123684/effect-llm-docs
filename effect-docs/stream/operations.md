# Operations

Source: extracted from `llms-full.txt` (`Operations`).

## Overview

This section covers common `Stream` operators for observing, transforming,
combining, partitioning, and rate-controlling stream values.

## Observe Without Changing Values

Use `Stream.tap` to run an effect for each emitted element while leaving the
stream values unchanged.

```ts
import { Console, Effect, Stream } from "effect"

const stream = Stream.make(1, 2, 3).pipe(
  Stream.tap((n) => Console.log(`before: ${n}`)),
  Stream.map((n) => n * 2),
  Stream.tap((n) => Console.log(`after: ${n}`))
)

Effect.runPromise(Stream.runCollect(stream))
```

## Take Elements

`take` operators limit output based on count or predicates:

| API | Behavior |
| --- | --- |
| `Stream.take(n)` | First `n` elements |
| `Stream.takeWhile(f)` | Elements while predicate holds |
| `Stream.takeUntil(f)` | Elements through the first matching element |
| `Stream.takeRight(n)` | Last `n` elements |

## Pull-Style Consumption

`Stream.toPull` converts a stream to an effectful pull function that returns
chunks until stream completion. End-of-stream is represented with `Option.None`
failure.

This model is useful when you want loop-style manual chunk consumption.

## Mapping and Transformation

| API | Purpose |
| --- | --- |
| `Stream.map` | Pure element transformation |
| `Stream.as` | Replace every element with a constant |
| `Stream.mapEffect` | Effectful mapping |
| `Stream.mapAccum` | Stateful mapping (carry state + emit output) |
| `Stream.mapConcat` | Map each element to many and flatten |

### Concurrency in `mapEffect`

`Stream.mapEffect` supports `{ concurrency: number }`, running multiple
effectful mappings in parallel while preserving downstream order.

## Filter and Scan

| API | Purpose |
| --- | --- |
| `Stream.filter` | Keep matching elements |
| `Stream.scan` | Emit intermediate accumulations |
| `Stream.runFold` | Produce only final accumulated result |

## Drain and Change Detection

| API | Purpose |
| --- | --- |
| `Stream.drain` | Run effects but discard all output values |
| `Stream.changes` | Remove consecutive duplicates |

## Zipping Streams

Basic zipping:

| API | Purpose |
| --- | --- |
| `Stream.zip` | Pair elements from two streams |
| `Stream.zipWith` | Pair with custom combine function |

Handling different lengths:

| API | Purpose |
| --- | --- |
| `Stream.zipAll` | Continue with default values |
| `Stream.zipAllWith` | Continue with custom handlers |

Handling different speeds:

| API | Purpose |
| --- | --- |
| `Stream.zipLatest` | Emit when either side updates, using latest from other side |
| `Stream.zipLatestWith` | Same with custom combine logic |

Neighborhood/index helpers:

| API | Purpose |
| --- | --- |
| `Stream.zipWithPrevious` | Pair each element with previous |
| `Stream.zipWithNext` | Pair each element with next |
| `Stream.zipWithPreviousAndNext` | Pair with previous and next |
| `Stream.zipWithIndex` | Pair each element with its index |

## Cartesian Product

`Stream.cross` forms all pair combinations from two streams.

Important behavior from source docs: the right-hand stream is re-iterated for
each left-hand element, so expensive right-hand effects can repeat.

## Partitioning and Grouping

Partitioning:

| API | Purpose |
| --- | --- |
| `Stream.partition` | Split with pure predicate |
| `Stream.partitionEither` | Split with effectful `Either` predicate |

Both return an effect that produces two scoped streams.

Grouping:

| API | Purpose |
| --- | --- |
| `Stream.groupByKey` | Group by pure key function |
| `Stream.groupBy` | Group by effectful keying |
| `GroupBy.evaluate` | Process each group and merge results |
| `Stream.grouped` | Chunk by fixed size |
| `Stream.groupedWithin` | Chunk by size or time window (whichever first) |

## Concatenation and Flat Mapping

| API | Purpose |
| --- | --- |
| `Stream.concat` | Append one stream after another |
| `Stream.concatAll` | Concatenate many streams |
| `Stream.flatMap` | Map each element to a stream, then flatten |

`Stream.flatMap` supports options such as:

- `concurrency`: run multiple inner streams concurrently
- `switch: true`: cancel older inner streams when newer ones arrive

## Merging, Interleaving, and Interspersing

Merging (interleave by availability):

| API | Purpose |
| --- | --- |
| `Stream.merge` | Interleave two streams concurrently |
| `Stream.mergeWith` | Merge while mapping each side to unified output |

`Stream.merge` accepts `haltStrategy`:
`"left" | "right" | "both" | "either"` (default `"both"`).

Interleaving (patterned alternation):

| API | Purpose |
| --- | --- |
| `Stream.interleave` | Alternate one-by-one from both streams |
| `Stream.interleaveWith` | Alternate using a controlling boolean stream |

Interspersing (insert separators/affixes):

| API | Purpose |
| --- | --- |
| `Stream.intersperse` | Insert delimiter between elements |
| `Stream.intersperseAffixes` | Add start/middle/end affixes |

## Broadcasting and Buffering

Broadcasting:

- `Stream.broadcast(n, maximumLag)` fans out one source stream to `n`
  downstream streams.
- `maximumLag` bounds how far the producer can get ahead of slow consumers.

Buffering:

`Stream.buffer` decouples producer and consumer speeds with configurable queue
behavior:

| Configuration | Behavior |
| --- | --- |
| `{ capacity: number }` | Bounded queue |
| `{ capacity: "unbounded" }` | Unbounded queue |
| `{ capacity: number, strategy: "sliding" }` | Keep latest values when full |
| `{ capacity: number, strategy: "dropping" }` | Keep earliest values when full |

## Debounce, Throttle, and Schedule

Rate/time controls:

| API | Purpose |
| --- | --- |
| `Stream.debounce(duration)` | Emit only after silence window |
| `Stream.throttle(...)` | Token-bucket rate limiting |
| `Stream.schedule(schedule)` | Delay/space emissions by schedule |

`Stream.throttle` details from source:

- Throttling applies to chunks, not individual elements.
- `strategy: "shape"` delays emissions to fit budget.
- `strategy: "enforce"` drops chunks exceeding limits.
- `burst` allows temporary extra throughput above base rate.
