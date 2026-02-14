---
title: Layer Memoization
description: How Effect reuses layer instances across a dependency graph based on reference equality.
---

# Layer Memoization

Source: extracted from `llms-full.txt` (`Layer Memoization`).

## Overview

Layer memoization lets Effect construct a layer once and reuse it across a dependency graph when the same layer instance is referenced multiple times.

```ts
Layer.merge(Layer.provide(L2, L1), Layer.provide(L3, L1))
```

In this shape, `L1` is allocated once and shared.

## Reference Equality Rule

Layer memoization is based on reference equality. If you create layers by repeatedly calling a factory function, each call creates a distinct layer value and memoization will not apply across those values.

Use this pattern:

```ts
const shared = makeLayer()
const combined = Layer.merge(
  Layer.provide(L2, shared),
  Layer.provide(L3, shared)
)
```

## Memoization When Providing Globally

When layers are provided globally, Effect shares each layer instance across dependents. If `BLive` and `CLive` both depend on `ALive`, and both use the same `ALive` value in the graph, `ALive` is initialized once.

## Acquiring a Fresh Version

If you need separate instances instead of sharing, wrap the layer with `Layer.fresh`:

```ts
Layer.merge(
  Layer.provide(BLive, Layer.fresh(ALive)),
  Layer.provide(CLive, Layer.fresh(ALive))
)
```

This forces independent initialization for each branch.

## No Automatic Memoization for Local Provide

Local provides do not memoize by default. Re-providing the same layer locally can initialize it multiple times:

```ts
yield* Effect.provide(A, ALive)
yield* Effect.provide(A, ALive)
```

## Manual Memoization

Use `Layer.memoize` to memoize explicitly in local/scoped workflows. `Layer.memoize(layer)` returns a scoped effect that yields a memoized layer you can reuse within that scope.

```ts
const program = Effect.scoped(
  Layer.memoize(ALive).pipe(
    Effect.andThen((memoized) =>
      Effect.gen(function* () {
        yield* Effect.provide(A, memoized)
        yield* Effect.provide(A, memoized)
      })
    )
  )
)
```

Within that scope, `ALive` is initialized once.
