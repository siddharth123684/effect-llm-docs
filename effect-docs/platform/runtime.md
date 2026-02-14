---
title: Runtime
description: Learn NodeRuntime.runMain with exit codes, signal handling, disableErrorReporting, disablePrettyLogger, and teardown for platform-aware Effect program execution and graceful shutdown in Node.js.
---

# Runtime

Source: extracted from `llms-full.txt` (`Runtime`).

## Overview

`runMain` executes your main Effect with platform runtime behavior included.
It handles failure reporting, signal interruption, teardown, and process exit
codes so your entrypoint logic stays focused on application behavior.

## What `runMain` Handles

- **Exit codes**: success exits with `0`; failures/interruption map to non-zero
  exit codes (default failure path uses `1`)
- **Error reporting**: failures are logged by default
- **Pretty logging**: errors use pretty formatting unless disabled
- **Signal handling**: interrupts on signals such as `SIGINT` (Ctrl+C)
- **Teardown**: finalization still runs when interrupted or failed

## Configuration Options

All `runMain` options are optional:

- `disableErrorReporting: true` to skip automatic error logs
- `disablePrettyLogger: true` to avoid adding pretty logging
- `teardown(exit, onExit)` to override default termination behavior

## Basic Usage

```ts
import { NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const program = Effect.succeed("Hello, World!")

NodeRuntime.runMain(program)
```

## Failure and Logging Controls

```ts
import { NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const failure = Effect.fail("Uh oh!")

NodeRuntime.runMain(failure) // default error reporting + pretty logger
NodeRuntime.runMain(failure, { disablePrettyLogger: true })
NodeRuntime.runMain(failure, { disableErrorReporting: true })
```

## Custom Teardown

```ts
import { NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const failure = Effect.fail("Uh oh!")

NodeRuntime.runMain(failure, {
  teardown(exit, onExit) {
    if (exit._tag === "Failure") {
      onExit(1)
    } else {
      onExit(0)
    }
  }
})
```
