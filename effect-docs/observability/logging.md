---
title: Logging
description: Learn Effect.log, log levels, annotations, spans, and pluggable loggers for composable, contextual logging when building observable Effect applications and services.
---

# Logging

Source: extracted from `llms-full.txt` (`Logging`).

## Overview

Effect logging is integrated into effect execution rather than being a separate side channel. This makes logging composable, contextual, and configurable through Effect APIs and layers.

Compared to traditional logging, Effect emphasizes:

- dynamic minimum log levels
- per-effect log-level control
- pluggable logger outputs
- environment/config-driven logging policies
- built-in annotations and spans

## Basic Logging

Use `Effect.log(...)` to log at `INFO` level (default visible level).

```ts
import { Cause, Effect } from "effect"

const program = Effect.gen(function* () {
  yield* Effect.log("Application started")
  yield* Effect.log("message1", "message2")
  yield* Effect.log("with cause", Cause.die("Oh no!"))
})

Effect.runFork(program)
```

Default log output includes structured metadata such as:

- `timestamp`
- `level`
- `fiber`
- `message`
- optional `cause`
- optional span timing entries

## Log Levels

Effect provides dedicated APIs for common levels:

- `Effect.logDebug`
- `Effect.logInfo`
- `Effect.logWarning`
- `Effect.logError`
- `Effect.logFatal`

`DEBUG` logs are disabled by default. Enable them by setting the minimum log level:

```ts
import { Effect, Logger, LogLevel } from "effect"

const task = Effect.logDebug("debug details").pipe(
  Logger.withMinimumLogLevel(LogLevel.Debug)
)
```

This can be applied to a specific effect for fine-grained control.

## Custom Annotations

Attach metadata to all logs in an effect scope with:

- `Effect.annotateLogs("key", "value")`
- `Effect.annotateLogs({ key1: "value1", key2: "value2" })`

Annotations propagate into nested effects.  
Use `Effect.annotateLogsScoped(...)` when metadata should only apply within a limited scoped region.

## Log Spans

Use `Effect.withLogSpan("label")` to add timing information to emitted logs for that effect region.

This is useful for lightweight performance visibility without introducing separate tracing tools.

## Disabling Logging

Set the minimum log level to `LogLevel.None` to suppress logging output.

Common approaches:

1. Per effect: `Logger.withMinimumLogLevel(LogLevel.None)`
2. Via layer: `Logger.minimumLogLevel(LogLevel.None)`
3. In a custom runtime: `ManagedRuntime.make(Logger.minimumLogLevel(LogLevel.None))`

## Loading Log Level from Configuration

You can make log verbosity environment-driven by reading a configured level and converting it to a layer:

- read with `Config.logLevel("LOG_LEVEL")`
- map to `Logger.minimumLogLevel(level)`
- convert with `Layer.unwrapEffect`
- provide that layer to the program

This allows different log behavior in development, test, and production without changing program logic.

## Custom Loggers

Define a logger with `Logger.make(...)`, then replace the default logger:

```ts
import { Effect, Logger } from "effect"

const custom = Logger.make(({ logLevel, message }) => {
  globalThis.console.log(`[${logLevel.label}] ${message}`)
})

const layer = Logger.replace(Logger.defaultLogger, custom)
const program = Effect.log("hello").pipe(Effect.provide(layer))
```

Use custom loggers to route output to external sinks, apply custom formatting, or integrate existing logging infrastructure.

## Built-in Loggers

Effect includes several built-in logger formats:

| Logger | Notes |
| --- | --- |
| `stringLogger` | Default key-value text logger (human-readable). |
| `logfmtLogger` | Compact logfmt-style key-value output. |
| `prettyLogger` | Colored/indented console-oriented output. |
| `structuredLogger` | Object-structured output with fields like `message`, `logLevel`, `timestamp`, `annotations`, `spans`, and `fiberId`. |
| `jsonLogger` | JSON string output (based on structured logger data). |

Layers are available for built-in alternatives (`Logger.logFmt`, `Logger.pretty`, `Logger.structured`, `Logger.json`) and can be provided to effects.

## Combining Loggers

Use `Logger.zip(left, right)` to forward each log event to both loggers.

This is useful when you need dual destinations (for example, console plus external collector).

## Key APIs

| API | Purpose |
| --- | --- |
| `Effect.log` | Log at `INFO` level. |
| `Effect.logDebug` / `logInfo` / `logWarning` / `logError` / `logFatal` | Log at specific severity levels. |
| `Logger.withMinimumLogLevel` | Set minimum level for an effect region. |
| `Logger.minimumLogLevel` | Build a layer that configures minimum log level. |
| `Effect.annotateLogs` | Add annotation metadata to logs in scope. |
| `Effect.annotateLogsScoped` | Add annotations in a limited scoped region. |
| `Effect.withLogSpan` | Attach span timing data to logs. |
| `Logger.make` | Build a custom logger. |
| `Logger.replace` | Replace default logger with another logger. |
| `Logger.zip` | Combine two loggers so both receive messages. |
