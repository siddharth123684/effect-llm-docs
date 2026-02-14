# PlatformLogger

Source: extracted from `llms-full.txt` (`PlatformLogger`).

## Overview

`PlatformLogger.toFile` builds a logger that writes log output to a file instead
of only sending messages to the console.

This is useful when you want persistent logs for debugging, auditing, or
post-run inspection.

## `toFile`

`toFile` creates a file-backed logger from a string-based logger (for example
`Logger.logfmtLogger`).

Key behavior:

- Writes to a target file path using `FileSystem` APIs
- Supports optional `batchWindow` to buffer writes for a short duration
- Returns an `Effect` that can fail with `PlatformError` if file operations fail

Without `batchWindow`, entries are written as they arrive.

## Example: Write Logs to a File

```ts
import { PlatformLogger } from "@effect/platform"
import { NodeFileSystem } from "@effect/platform-node"
import { Effect, Layer, Logger } from "effect"

const fileLogger = Logger.logfmtLogger.pipe(
  PlatformLogger.toFile("/tmp/log.txt")
)

const LoggerLive = Logger.replaceScoped(
  Logger.defaultLogger,
  fileLogger
).pipe(Layer.provide(NodeFileSystem.layer))

const program = Effect.log("Hello")

Effect.runFork(program.pipe(Effect.provide(LoggerLive)))
```

## Example: Write to Console and File

```ts
import { PlatformLogger } from "@effect/platform"
import { NodeFileSystem } from "@effect/platform-node"
import { Effect, Layer, Logger } from "effect"

const fileLogger = Logger.logfmtLogger.pipe(
  PlatformLogger.toFile("/tmp/log.txt")
)

const bothLoggers = Effect.map(fileLogger, (fileLogger) =>
  Logger.zip(Logger.prettyLoggerDefault, fileLogger)
)

const LoggerLive = Logger.replaceScoped(
  Logger.defaultLogger,
  bothLoggers
).pipe(Layer.provide(NodeFileSystem.layer))

const program = Effect.log("Hello")

Effect.runFork(program.pipe(Effect.provide(LoggerLive)))
```
