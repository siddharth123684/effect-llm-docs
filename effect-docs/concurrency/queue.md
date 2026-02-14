---
title: Queue
description: Fiber-friendly in-memory queue for async coordination with back-pressure and typed handoff.
---

# Queue

Source: extracted from `llms-full.txt` (`Queue`).

## Overview

`Queue<A>` is an in-memory, fiber-friendly queue for asynchronous coordination.
It provides back-pressure and typed handoff between producers and consumers.

Core operations:

- `Queue.offer(queue, value)`: enqueue one value
- `Queue.take(queue)`: dequeue oldest value

## Queue Kinds

Use the constructor that matches overflow behavior:

- `Queue.bounded(capacity)`: back-pressured when full (`offer` suspends)
- `Queue.dropping(capacity)`: drops new values when full
- `Queue.sliding(capacity)`: drops old values to make room for new ones
- `Queue.unbounded()`: no fixed capacity

```ts
import { Queue } from "effect"

const bounded = Queue.bounded<number>(100)
const dropping = Queue.dropping<number>(100)
const sliding = Queue.sliding<number>(100)
const unbounded = Queue.unbounded<number>()
```

## Producing Values

### `Queue.offer`

Adds one value. On back-pressured queues, this may suspend when full.

### `Queue.offerAll`

Adds many values at once.

```ts
import { Effect, Fiber, Queue } from "effect"

const program = Effect.gen(function* () {
  const queue = yield* Queue.bounded<number>(1)
  yield* Queue.offer(queue, 1)

  // Suspends until space is available.
  const producer = yield* Effect.fork(Queue.offer(queue, 2))
  yield* Queue.take(queue)
  yield* Fiber.join(producer)
})
```

## Consuming Values

### `Queue.take`

Removes and returns the oldest value. Suspends if the queue is empty.

### `Queue.poll`

Non-blocking take:

- `Some(value)` when an item is available
- `None` when empty

### Batch-oriented reads

- `Queue.takeUpTo(queue, n)`: returns up to `n` items, never waits for missing ones
- `Queue.takeN(queue, n)`: waits until exactly `n` items are available
- `Queue.takeAll(queue)`: drains current contents immediately

## Shutdown Behavior

### `Queue.shutdown`

Shuts the queue down, interrupts fibers suspended on `offer*` / `take*`, clears
stored items, and causes future queue operations to terminate immediately.

### `Queue.awaitShutdown`

Waits for shutdown and resumes immediately if already shut down.

## Capability-Restricted Interfaces

`Queue` can be narrowed to capability-specific views:

- `Queue.Enqueue<A>`: offer-only API for producer-only code paths
- `Queue.Dequeue<A>`: take-only API for consumer-only code paths

This is useful for enforcing boundaries between writers and readers while still
sharing one underlying queue.
