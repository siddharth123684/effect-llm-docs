# Default Services

Source: extracted from `llms-full.txt` (`Default Services`).

## Overview

Effect includes five built-in services that are provided automatically at runtime:

- `Clock`
- `ConfigProvider`
- `Console`
- `Random`
- `Tracer`

Because these services are default-provided, using them does not force extra runtime requirements in your effect type.

```ts
import { Clock, Console, Effect } from "effect"

const program = Effect.gen(function* () {
  const now = yield* Clock.currentTimeMillis
  yield* Console.log(`Application started at ${new Date(now)}`)
})
// Effect<void, never, never>
```

Even though the program uses `Clock` and `Console`, the requirements channel remains `never`.

## Overriding Default Services

When you need custom behavior (for tests, deterministic behavior, or custom runtimes), override defaults with:

- `Effect.with<Service>`: override for the duration of an effect.
- `Effect.with<Service>Scoped`: override within a scope, then restore automatically.

Common helpers:

| Service | Temporary Override | Scoped Override |
| --- | --- | --- |
| `Clock` | `Effect.withClock` | `Effect.withClockScoped` |
| `ConfigProvider` | `Effect.withConfigProvider` | `Effect.withConfigProviderScoped` |
| `Console` | `Effect.withConsole` | `Effect.withConsoleScoped` |
| `Random` | `Effect.withRandom` | `Effect.withRandomScoped` |
| `Tracer` | `Effect.withTracer` | `Effect.withTracerScoped` |

## Example: Overriding `Random`

```ts
import { Effect, Random } from "effect"

const program = Effect.gen(function* () {
  return yield* Random.next
})

const deterministic = program.pipe(
  Effect.withRandom(Random.make("myseed"))
)
```

`deterministic` now uses a seeded random generator, producing repeatable results.
