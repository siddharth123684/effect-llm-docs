# Fallback

Source: extracted from `llms-full.txt` (`Fallback`).

## Overview

Fallback operators help you recover from failures by choosing an alternative path. In Effect, the core options are:

- run another effect if the first fails
- replace the original error with a different error
- replace failure with a default success value
- try several effects and return the first success

## `Effect.orElse`

Use `Effect.orElse` to run a fallback effect only when the original effect fails.

- If the original succeeds, its value is kept.
- If the original fails, the fallback effect is executed.

```ts
import { Effect } from "effect"

const primary = Effect.fail("primary failed")
const fallback = Effect.succeed("fallback value")

const program = Effect.orElse(primary, () => fallback)
console.log(Effect.runSync(program)) // "fallback value"
```

## `Effect.orElseFail`

Use `Effect.orElseFail` to replace any failure with a new failure value.

This is useful when you want to normalize low-level errors into a stable domain error.

```ts
import { Effect } from "effect"

const validate = (age: number): Effect.Effect<number, string> =>
  age < 0 ? Effect.fail("NegativeAgeError") : Effect.succeed(age)

const program = Effect.orElseFail(validate(-1), () => "invalid age")
console.log(Effect.runSyncExit(program))
```

## `Effect.orElseSucceed`

Use `Effect.orElseSucceed` to replace failure with a default success value.

This is useful when you want a guaranteed successful result for downstream logic.

```ts
import { Effect } from "effect"

const loadPort: Effect.Effect<number, string> = Effect.fail("Missing PORT")
const program = Effect.orElseSucceed(loadPort, () => 3000)

console.log(Effect.runSync(program)) // 3000
```

## `Effect.firstSuccessOf`

`Effect.firstSuccessOf` evaluates effects sequentially and returns the first success.

- Stops as soon as one effect succeeds.
- If all effects fail, the last failure is returned.
- Passing an empty collection throws an `IllegalArgumentException`.

```ts
import { Effect } from "effect"

const a = Effect.fail("node-a unavailable")
const b = Effect.fail("node-b unavailable")
const c = Effect.succeed("node-c config")

const program = Effect.firstSuccessOf([a, b, c])
console.log(Effect.runSync(program)) // "node-c config"
```

## Quick Selection

| Goal | Operator |
| --- | --- |
| Try another effect on failure | `Effect.orElse` |
| Fail with a different error | `Effect.orElseFail` |
| Recover with a default success value | `Effect.orElseSucceed` |
| Try many alternatives and keep first success | `Effect.firstSuccessOf` |
