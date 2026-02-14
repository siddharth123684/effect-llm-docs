# PubSub

Source: extracted from `llms-full.txt` (`PubSub`).

## Overview

`PubSub<A>` is an asynchronous broadcast hub.

Publishers send values, and every active subscriber can receive each published
value. This differs from `Queue`, where each value is consumed by only one
consumer.

Use `PubSub` when you need fan-out broadcasting, not load distribution.

## Basic Operations

`PubSub` has two core operations:

- `PubSub.publish(pubsub, value)`: publish one value, returning `Effect<boolean>`
- `PubSub.subscribe(pubsub)`: scoped subscription that returns a `Dequeue<A>`

Each subscriber reads from its own dequeue (for example with `Queue.take`), and
is automatically unsubscribed when its scope ends.

```ts
import { Effect, PubSub, Queue } from "effect"

const program = Effect.scoped(
  Effect.gen(function* () {
    const pubsub = yield* PubSub.bounded<string>(2)

    const sub1 = yield* PubSub.subscribe(pubsub)
    const sub2 = yield* PubSub.subscribe(pubsub)

    yield* PubSub.publish(pubsub, "Hello from PubSub")

    const m1 = yield* Queue.take(sub1)
    const m2 = yield* Queue.take(sub2)

    return [m1, m2]
  })
)
```

## Subscription Timing

Subscribers only receive messages published while they are actively subscribed.
To guarantee delivery to a subscriber, subscribe before publishing.

## Creating a PubSub

Effect provides four creation strategies:

- `PubSub.bounded(capacity)`
  - Applies back pressure when full
  - Preserves delivery to active subscribers
- `PubSub.dropping(capacity)`
  - Drops new values when full (`publish` can return `false`)
  - Favors publisher throughput over guaranteed delivery
- `PubSub.sliding(capacity)`
  - Drops oldest buffered values when full
  - Avoids publisher blocking, but slow subscribers may miss older messages
- `PubSub.unbounded()`
  - Never blocks publishers due to capacity
  - Can grow without bound if producers outpace consumers

```ts
import { PubSub } from "effect"

const bounded = PubSub.bounded<string>(2)
const dropping = PubSub.dropping<string>(2)
const sliding = PubSub.sliding<string>(2)
const unbounded = PubSub.unbounded<string>()
```

## Useful Operators

- `PubSub.publishAll(pubsub, values)`: publish many values at once
- `PubSub.capacity(pubsub)`: static numeric capacity
- `PubSub.size(pubsub)`: current buffered size as an effect
- `PubSub.shutdown(pubsub)`: shutdown pubsub and associated queues
- `PubSub.isShutdown(pubsub)`: check shutdown status
- `PubSub.awaitShutdown(pubsub)`: wait for shutdown completion

```ts
import { Effect, PubSub } from "effect"

const program = Effect.gen(function* () {
  const pubsub = yield* PubSub.bounded<number>(2)
  yield* PubSub.publishAll(pubsub, [1, 2])
  const size = yield* PubSub.size(pubsub)
  return { capacity: PubSub.capacity(pubsub), size }
})
```

## PubSub as Enqueue

`PubSub<A>` can be treated like an enqueue-only interface:

```ts
import type { Queue } from "effect"

interface PubSub<A> extends Queue.Enqueue<A> {}
```

This is useful when an API needs something that can accept writes only.
