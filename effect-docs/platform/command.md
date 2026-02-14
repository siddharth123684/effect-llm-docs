---
title: Command
description: How to run external processes as Effect values with composable, testable execution.
---

# Command

Source: extracted from `llms-full.txt` (`Command`).

## Overview

`@effect/platform/Command` models external process execution as Effect values.
You define a command (program + args + execution options), then run it through a
provided `CommandExecutor`.

This keeps shell/process interactions composable and testable inside Effect
workflows.

## Creating Commands

Use `Command.make` to build a command descriptor:

```ts
import { Command } from "@effect/platform"

const command = Command.make("ls", "-al")
```

The result is a data structure (not immediate execution). It can then be
transformed with helpers such as `Command.env`, `Command.runInShell`, and
`Command.feed`.

## Running Commands

To execute commands, provide a platform context that includes a
`CommandExecutor` (for example `NodeContext.layer`).

```ts
import { Command } from "@effect/platform"
import { NodeContext, NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const output = yield* Command.string(Command.make("ls", "-al"))
  console.log(output)
})

NodeRuntime.runMain(program.pipe(Effect.provide(NodeContext.layer)))
```

## Output Methods

Common execution helpers:

- `Command.string(command)`: collect `stdout` as a string
- `Command.lines(command)`: collect `stdout` as an array of lines
- `Command.stream(command)`: stream `stdout` as `Uint8Array` chunks
- `Command.streamLines(command)`: stream line-by-line text output

Use `Command.exitCode(command)` when only process success/failure is needed.

## Environment Variables

Use `Command.env` to provide command-specific environment values. Pair with
`Command.runInShell(true)` when shell expansion is required (for example `$VAR`).

```ts
import { Command } from "@effect/platform"

const command = Command.make("echo", "-n", "$MY_CUSTOM_VAR").pipe(
  Command.env({ MY_CUSTOM_VAR: "hello from env" }),
  Command.runInShell(true)
)
```

## Feeding stdin

Use `Command.feed` to send text input to a process:

```ts
import { Command } from "@effect/platform"

const command = Command.make("cat").pipe(Command.feed("Hello"))
```

## Working with a Running Process

Use `Command.start` to get a process handle and access:

- `process.exitCode`: effect that waits for completion and returns exit code
- `process.stdout`: output stream
- `process.stderr`: error stream

This is useful for advanced workflows where you want explicit control over
stream consumption and lifecycle.

## Inheriting stdout

When you want command output to go directly to the parent process terminal, use
`Command.stdout("inherit")`.

```ts
import { Command } from "@effect/platform"

const program = Command.make("cat", "./some-file.txt").pipe(
  Command.stdout("inherit"),
  Command.exitCode
)
```
