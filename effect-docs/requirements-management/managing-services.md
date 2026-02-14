---
title: Managing Services
description: Learn to create, use, and provide Effect services via Context.Tag, Effect.provideService, and Layer for type-safe dependency injection in Effect programs.
---

# Managing Services

Source: extracted from `llms-full.txt` (`Managing Services`).

## Overview

A **service** is a reusable capability shared across parts of an application.
In Effect, services are accessed through `Context` and tracked in the `Requirements` type parameter of `Effect`.

Instead of manually passing dependencies through function arguments, you declare service requirements in types and provide implementations at the program boundary.

## Managing Services with Effect

Effect models dependencies as:

```text
Effect<Success, Error, Requirements>
```

- `Requirements` represents required services.
- Services are identified by **tags**.
- Tags and implementations are stored in `Context`.

This keeps business logic decoupled from wiring code and lets the compiler enforce missing dependencies.

## Creating a Service

Create a tag with:
1. a unique identifier string
2. a service interface type

```ts
import { Context, Effect } from "effect"

class Random extends Context.Tag("MyRandomService")<
  Random,
  { readonly next: Effect.Effect<number> }
>() {}
```

The tag (`Random`) is the key used to retrieve and provide the service.

## Using a Service

Use the tag inside an effect program:

```ts
import { Context, Effect } from "effect"

class Random extends Context.Tag("MyRandomService")<
  Random,
  { readonly next: Effect.Effect<number> }
>() {}

const program = Effect.gen(function* () {
  const random = yield* Random
  const n = yield* random.next
  console.log(n)
})
// program: Effect.Effect<void, never, Random>
```

If you try to run this without providing `Random`, you get a type error because `Random` is still required.

## Providing a Service Implementation

Provide an implementation with `Effect.provideService`:

```ts
import { Context, Effect } from "effect"

class Random extends Context.Tag("MyRandomService")<
  Random,
  { readonly next: Effect.Effect<number> }
>() {}

const program = Effect.gen(function* () {
  const random = yield* Random
  const n = yield* random.next
  console.log(n)
})

const runnable = Effect.provideService(program, Random, {
  next: Effect.sync(() => Math.random())
})
// runnable: Effect.Effect<void, never, never>
```

After providing the service, `Requirements` becomes `never`.

## Extracting Service Type

Use `Context.Tag.Service` to get the service shape from a tag:

```ts
type RandomShape = Context.Tag.Service<Random>
```

## Using Multiple Services

If a program uses multiple tags, its requirements become a union (for example `Random | Logger`).

You can provide services by:
- chaining `Effect.provideService(...)` calls, or
- building a `Context` with `Context.add(...)` and using `Effect.provide(...)`.

## Optional Services

Use `Effect.serviceOption(Tag)` when a service may or may not be available.
It returns `Option<Service>`, so you can define fallback behavior when the service is missing.

This pattern allows access to context values without adding that service to required `R`.

## Services with Dependencies

If a service implementation depends on other services, avoid leaking those implementation details into the service interface.

Use **Layer** to construct services with dependencies and keep interfaces clean.

## Key APIs

| API | Purpose |
| --- | --- |
| `Context.Tag` | Define a service tag and interface |
| `yield* Tag` | Access a service in `Effect.gen` |
| `Effect.provideService` | Provide one service implementation |
| `Context.add` + `Effect.provide` | Provide multiple services via a full context |
| `Effect.serviceOption` | Access a service optionally |
| `Context.Tag.Service` | Extract a service type from a tag |
