---
title: Terminal
description: Learn Terminal.Terminal, terminal.display, terminal.readLine, NodeTerminal.layer, and NodeRuntime.runMain for terminal I/O and interactive CLI applications when building Effect applications and services.
---

# Terminal

Source: extracted from `llms-full.txt` (`Terminal`).

## Overview

`@effect/platform/Terminal` provides an abstraction for terminal input/output.
It lets Effect programs read from standard input and write to standard output
through a service instead of direct process APIs.

## Accessing the Service

Use the `Terminal` tag from context:

```ts
import { Terminal } from "@effect/platform"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const terminal = yield* Terminal.Terminal
  return terminal
})
```

## Writing to Standard Output

Use `terminal.display(...)` to print text:

```ts
import { Terminal } from "@effect/platform"
import { NodeRuntime, NodeTerminal } from "@effect/platform-node"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const terminal = yield* Terminal.Terminal
  yield* terminal.display("a message\n")
})

NodeRuntime.runMain(program.pipe(Effect.provide(NodeTerminal.layer)))
```

## Reading from Standard Input

Use `terminal.readLine` to read one line of input:

```ts
import { Terminal } from "@effect/platform"
import { NodeRuntime, NodeTerminal } from "@effect/platform-node"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const terminal = yield* Terminal.Terminal
  const input = yield* terminal.readLine
  console.log(`input: ${input}`)
})

NodeRuntime.runMain(program.pipe(Effect.provide(NodeTerminal.layer)))
```

## Typical Error/Requirement Types

Interactive terminal effects commonly use:

- Requirements: `Terminal.Terminal`
- Errors: `Terminal.QuitException | PlatformError`

This is the shape used by terminal-driven loops (for example, prompt/validate
retry flows such as number-guessing games).
