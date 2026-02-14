# Using Generators

Source: extracted from `llms-full.txt` (`Using Generators`).

## Overview

`Effect.gen` lets you write effectful workflows in a style similar to `async`/`await`, while staying inside the Effect model.

- Wrap logic in `Effect.gen(function* () { ... })`
- Use `yield*` to run effects and bind their success values
- `return` the final success value

Generators are optional in Effect. If you prefer function composition, see pipeline-style APIs.

## Basic Pattern

```ts
import { Effect } from "effect"

const readA = Effect.succeed(10)
const readB = Effect.succeed(5)

const program = Effect.gen(function* () {
  const a = yield* readA
  const b = yield* readB
  return a + b
})
```

## Error Semantics (Short-Circuiting)

Inside `Effect.gen`, failures short-circuit the generator:

- Execution stops at the first failed `yield*`
- Later statements are not executed
- The whole program fails with that error

If you need to keep going after an expected failure, convert failures into values (for example with `Effect.either`) and branch explicitly.

## Control Flow Works Naturally

You can use normal JavaScript control flow inside the generator body:

- `if` / `else`
- `for` / `while`
- `break` / `continue`

This is useful for branching and loops while still sequencing effectful operations with `yield*`.

## TypeScript Narrowing Tip

TypeScript may not always narrow types after `yield* Effect.fail(...)` unless you explicitly return.

```ts
import { Effect } from "effect"

declare const findUser: Effect.Effect<{ name: string } | undefined>

const program = Effect.gen(function* () {
  const user = yield* findUser

  if (user === undefined) {
    return yield* Effect.fail("User not found")
  }

  return user.name
})
```

Using `return yield* Effect.fail(...)` helps TypeScript understand execution does not continue past that branch.

## TypeScript Configuration Requirement

The generator API requires either:

- `tsconfig.json` with `target` set to `es2015` (or higher), or
- `downlevelIteration: true`

## Passing `this`

When needed, you can pass `this` as the first argument to `Effect.gen`:

```ts
import { Effect } from "effect"

class Service {
  readonly value = 1

  readonly program = Effect.gen(this, function* () {
    return this.value + 1
  })
}
```

## Deprecated Adapter Form

Older code may use an adapter parameter (often `$` or `_`) inside `Effect.gen(function* ($) { ... })`.
With modern TypeScript versions, this is no longer necessary for inference and is considered legacy usage.
