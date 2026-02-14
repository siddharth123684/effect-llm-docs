---
title: Cache
description: Learn Cache to deduplicate overlapping lookups, cache results with TTL, and expose cache metrics for observability and performance in Effect programs.
---

# Cache

Source: extracted from `llms-full.txt` (`Cache`).

## Overview

`Cache` helps avoid duplicate work when multiple parts of an application request the same value.
It integrates with Effect so cache lookups work for synchronous and asynchronous computations, and remain safe under concurrency.

Key characteristics:

- Deduplicates overlapping work for the same key.
- Uses one lookup model for both sync and async effects.
- Supports interruption/failure-aware behavior in concurrent lookups.
- Exposes cache metrics (for example, hits and misses) for observability.

## Creating a Cache

A cache is built from:

- `capacity`: maximum number of entries to retain.
- `timeToLive`: how long loaded values remain valid.
- `lookup`: effectful function used to compute missing values.

```ts
import { Cache, Duration, Effect } from "effect"

const lookup = (key: string) => Effect.succeed(key.length)

const program = Effect.gen(function* () {
  const cache = yield* Cache.make({
    capacity: 100,
    timeToLive: Duration.infinity,
    lookup
  })

  const value = yield* cache.get("key1")
  console.log(value)
})
```

`cache.get(key)` is the main operation:

- Returns the cached value when present and still valid.
- Otherwise runs `lookup`, stores the result, and returns it.

## Concurrent Access Semantics

When multiple fibers request the same missing key at once:

- The lookup is evaluated once.
- Other requesters wait for that same computation result.
- Waiting does not block an underlying thread.

Failure/interruption behavior:

- If lookup fails, all waiters receive that failure.
- Failed results are cached to avoid repeated failed recomputation.
- If lookup is interrupted, the key is removed so later requests recompute.

## Capacity and TTL

### Capacity

Eviction is least-recently-accessed first when the cache reaches capacity.
Under high concurrency, the size can briefly exceed the configured limit between operations.

### Time To Live (TTL)

TTL is measured from when a value is loaded into the cache.
Values older than TTL are treated as expired and are not returned.

## Core Methods

- `get(key)`: read-or-compute value for a key.
- `refresh(key)`: recompute in the background while keeping previous value available.
- `size`: current approximate number of entries.
- `contains(key)`: check whether a key currently has an entry.
- `invalidate(key)`: evict a specific key.
- `invalidateAll`: clear all cached entries.

`cache.cacheStats` can be used to inspect runtime metrics such as hits and misses.
