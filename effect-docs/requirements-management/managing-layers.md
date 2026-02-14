---
title: Managing Layers
description: Learn to build and compose dependency graphs with Layer.merge, Layer.provide, and Layer.provideMerge for Effect services and type-safe dependency injection in applications.
---

# Managing Layers

Source: extracted from `llms-full.txt` (`Managing Layers`).

## Overview

`Layer` models the dependency graph of your application and acts as a constructor for services.
It lets you handle dependencies during service construction rather than exposing them in service method signatures.

Core concepts:

| Concept | Description |
| --- | --- |
| `service` | Reusable functionality used across the app. |
| `tag` | Typed identifier used to access a service from context. |
| `context` | Map-like container of services indexed by tags. |
| `layer` | Blueprint that builds services from dependencies. |

## Why Layers Matter

Without layers, service interfaces can leak implementation dependencies.
For example, a `Database` API that requires `Config | Logger` in every method makes tests and usage more complex.

Guideline:

- Keep service operations requirement-free when possible: `Effect<Success, Error, never>`.
- Put dependency wiring in layers, not in service method types.

## Layer Type

```text
Layer<RequirementsOut, Error, RequirementsIn>
```

- `RequirementsOut`: services/resources produced by the layer
- `Error`: possible construction error
- `RequirementsIn`: dependencies needed to build the output

Common constructors:

- `Layer.succeed(tag, value)` for static/pure construction
- `Layer.effect(tag, effect)` for effectful construction that may need dependencies

## Building a Dependency Graph

Example graph:

- `Config` has no dependencies
- `Logger` depends on `Config`
- `Database` depends on `Config` and `Logger`

Resulting layer types:

- `ConfigLive`: `Layer<Config, never, never>`
- `LoggerLive`: `Layer<Logger, never, Config>`
- `DatabaseLive`: `Layer<Database, never, Config | Logger>`

```ts
import { Effect, Layer } from "effect"

const ConfigLive = Layer.succeed(Config, {
  getConfig: Effect.succeed({ logLevel: "INFO", connection: "..." })
})

const LoggerLive = Layer.effect(Logger, Effect.gen(function* () {
  const config = yield* Config
  return {
    log: (message: string) =>
      Effect.gen(function* () {
        const { logLevel } = yield* config.getConfig
        console.log(`[${logLevel}] ${message}`)
      })
  }
}))
```

## Combining Layers

### `Layer.merge`

Combine layers in parallel.

- Inputs are combined: `In1 | In2`
- Outputs are combined: `Out1 | Out2`

```ts
const AppConfigLive = Layer.merge(ConfigLive, LoggerLive)
// Layer<Config | Logger, never, Config>
```

### `Layer.provide`

Compose layers by feeding one layer's output into another layer's input.

```ts
const MainLive = DatabaseLive.pipe(
  Layer.provide(AppConfigLive),
  Layer.provide(ConfigLive)
)
// Layer<Database, never, never>
```

### `Layer.provideMerge`

Provide dependencies and keep both outputs when needed.

```ts
const MainWithConfig = DatabaseLive.pipe(
  Layer.provide(AppConfigLive),
  Layer.provideMerge(ConfigLive)
)
// Layer<Config | Database, never, never>
```

## Providing Layers to Effects

Use `Effect.provide` to satisfy an effect's requirements with a fully-resolved layer:

```ts
const runnable = Effect.provide(program, MainLive)
// Effect<Success, Error, never>
```

When requirements become `never`, the program has everything it needs to run.

## Converting and Observing Layers

- `Layer.launch(layer)`: convert a long-lived layer into an effect and keep it running until interrupted.
- `Layer.tap(f)`: run a side effect when acquisition succeeds.
- `Layer.tapError(f)`: run a side effect when acquisition fails.

These operations do not change the layer type signature.

## Layer Error Handling

- `Layer.catchAll((error) => fallbackLayer)`: recover with access to the construction error.
- `Layer.orElse(() => fallbackLayer)`: fallback without inspecting the error.

Use these when startup dependencies (config, external systems, etc.) may fail.

## `Effect.Service` and Layers

`Effect.Service` can define a service and its layer in one place, including dependencies.

Generated layers:

| Layer | Meaning |
| --- | --- |
| `MyService.Default` | Service plus declared dependencies already wired. |
| `MyService.DefaultWithoutDependencies` | Service only; dependencies must be provided externally. |

This is useful for testing:

- provide `Default` for production-like wiring
- provide `DefaultWithoutDependencies` plus mock/test dependencies
- or mock the service directly with `Effect.provideService`

## Choosing Between `Effect.Service` and `Context.Tag`

- Use `Effect.Service` for application services with a sensible default implementation.
- Use `Context.Tag` for library code or contexts where no default should be assumed.

`Effect.Service` is mostly ergonomic sugar over `Context.Tag` plus generated layer helpers.
