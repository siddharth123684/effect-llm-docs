# Guidelines

Source: extracted from `llms-full.txt` (`Guidelines`).

## Overview

These guidelines focus on two practical style rules for Effect applications:

- use platform `runMain` entrypoints for graceful app shutdown
- avoid tacit (point-free) combinator usage when it can hide type and runtime issues

## Use `runMain` at the Application Entry Point

For long-running apps (servers, workers, CLIs that should handle interrupts),
prefer platform runtimes such as `NodeRuntime.runMain` over direct
`Effect.runPromise`.

`runMain` observes the main fiber, listens for interrupt signals (for example,
`SIGINT` from `CTRL+C`), interrupts fibers, and allows finalizers to run.

```ts
import { Console, Effect, Schedule, pipe } from "effect"
import { NodeRuntime } from "@effect/platform-node"

const program = pipe(
  Effect.addFinalizer(() => Console.log("Application is about to exit!")),
  Effect.andThen(Console.log("Application started!")),
  Effect.andThen(
    Effect.repeat(Console.log("still alive..."), {
      schedule: Schedule.spaced("1 second")
    })
  ),
  Effect.scoped
)

NodeRuntime.runMain(program)
```

Keep teardown logic in the main scoped effect so interruption reliably releases
resources.

## `runMain` by Platform

| Platform | Runtime call             | Import package              |
| -------- | ------------------------ | --------------------------- |
| Node.js  | `NodeRuntime.runMain`    | `@effect/platform-node`     |
| Bun      | `BunRuntime.runMain`     | `@effect/platform-bun`      |
| Browser  | `BrowserRuntime.runMain` | `@effect/platform-browser`  |

## Avoid Tacit (Point-Free) Usage

Prefer explicit lambdas in combinators:

```ts
Effect.map((x) => fn(x))
```

instead of tacit style:

```ts
Effect.map(fn)
```

Tacit usage can look concise, but in Effect code it can introduce subtle
problems:

- overloaded or optional-parameter functions can erase generics unexpectedly
- type inference can degrade in ways that are hard to spot
- stack traces can become less readable

Using explicit function parameters is a simple way to keep behavior and types
predictable.
