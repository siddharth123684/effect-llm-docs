# Unexpected Errors

Source: extracted from `llms-full.txt` (`Unexpected Errors`).

## Overview

Unexpected errors are **defects**: failures outside normal business flow.
They are not tracked in the typed error channel (`E`) and are treated like unchecked exceptions by the runtime.

Use defect APIs when recovery is not meaningful in domain logic, and prefer handling/reporting at system boundaries.

## Creating Unrecoverable Errors

### `Effect.die`

Terminates the fiber with a provided defect value.

```ts
import { Effect } from "effect"

const divide = (a: number, b: number) =>
  b === 0 ? Effect.die(new Error("Cannot divide by zero")) : Effect.succeed(a / b)
```

### `Effect.dieMessage`

Terminates with a `RuntimeException` using a message.

```ts
import { Effect } from "effect"

const fatal = Effect.dieMessage("Unexpected invariant violation")
```

## Converting Expected Failures into Defects

### `Effect.orDie`

Converts `Effect<A, E, R>` into `Effect<A, never, R>` by escalating any failure to a defect.

### `Effect.orDieWith`

Like `orDie`, but maps the typed error before terminating:

```ts
import { Effect } from "effect"

const program = Effect.orDieWith(task, (e) => new Error(`defect: ${String(e)}`))
```

## Catching/Inspecting Defects at Boundaries

Defects usually indicate serious issues. Catch them sparingly, mainly for diagnostics, logging, or boundary translation.

### `Effect.exit`

Materializes success/failure into `Exit`, so you can inspect causes without failing the surrounding effect:

```ts
import { Effect } from "effect"

const inspected = Effect.exit(task)
// Effect<Exit<A, E>, never, R>
```

### `Effect.catchAllDefect`

Recovers from any defect.
Does **not** catch typed failures (`Effect.fail`) or interruptions.

### `Effect.catchSomeDefect`

Recovers from selected defects via a partial handler (return `Option.some(effect)` to recover, `Option.none()` to let it propagate).

## Practical Guidance

- Prefer typed errors (`E`) for expected, domain-level failures.
- Use defects for irrecoverable situations only.
- Recover from defects only at integration boundaries (process edge, plugin boundary, top-level runner).
- If you need selective recovery, choose `catchSomeDefect`; for full boundary recovery, use `catchAllDefect` or inspect with `exit`.
