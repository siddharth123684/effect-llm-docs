---
title: Batching
description: Learn Request, RequestResolver, and Effect.request to batch compatible requests and reduce repeated work across concurrent Effect programs efficiently and safely.
---

# Batching

Source: extracted from `llms-full.txt` (`Batching`).

## Overview

Batching in Effect groups compatible requests and resolves them together, reducing repeated work and API calls.
The core pieces are `Request`, `RequestResolver`, and `Effect.request`.

## Step-by-Step Model

1. Declare request data types.
2. Implement resolvers (batched where possible).
3. Define queries with `Effect.request` and run programs with batching enabled.

Requests should be modeled so equivalent requests can be identified and grouped.

## Declaring Requests

`Request<Value, Error>` models an effectful data request that may fail.
Use tagged constructors (for example `Request.tagged`) so request values are easy to construct and route.

## Declaring Resolvers

Use a normal resolver for non-batchable endpoints:

- `RequestResolver.fromEffect(...)`

Use a batched resolver for batch-capable endpoints:

- `RequestResolver.makeBatched((requests) => ...)`

Inside a batched resolver, complete each request explicitly:

- success path: `Request.completeEffect(request, Effect.succeed(value))`
- failure path: `Request.completeEffect(request, Effect.fail(error))`

```ts
const GetUserByIdResolver = RequestResolver.makeBatched((requests) =>
  Effect.tryPromise({
    try: () =>
      fetch("https://api.example.demo/getUserByIdBatch", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          users: requests.map(({ id }) => ({ id }))
        })
      }).then((res) => res.json()) as Promise<Array<User>>,
    catch: () => new GetUserError()
  }).pipe(
    Effect.andThen((users) =>
      Effect.forEach(requests, (request, index) =>
        Request.completeEffect(request, Effect.succeed(users[index]!))
      )
    ),
    Effect.catchAll((error) =>
      Effect.forEach(requests, (request) =>
        Request.completeEffect(request, Effect.fail(error))
      )
    )
  )
)
```

## Defining Queries

Bind requests to resolvers with `Effect.request`:

```ts
const getUserById = (id: number) =>
  Effect.request(GetUserById({ id }), GetUserByIdResolver)
```

When traversing many items, enable request batching in execution options:

```ts
yield* Effect.forEach(todos, (todo) => notifyOwner(todo), { batching: true })
```

In the documented example, batching reduces request volume from `1 + 2n` to `3` total API queries.

## Disabling Batching

Disable batching locally with:

```ts
program.pipe(Effect.withRequestBatching(false))
```

## Resolvers with Context

Resolvers can depend on services, but batching compatibility improves when resolver context is constrained.
Use `RequestResolver.contextFromServices(...)` to declare exactly which services resolver logic needs.

This commonly yields a resolver constructor effect like:

```ts
Effect<RequestResolver<GetTodos, never>, never, HttpService>
```

You can also assemble resolvers within `Layer` definitions, which is often a natural place to wire services and request logic together.
