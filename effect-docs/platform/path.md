---
title: Path
description: Platform-abstracted file path operations including join, resolve, normalize, and parse.
---

# Path

Source: extracted from `llms-full.txt` (`Path`).

## Overview

`@effect/platform/Path` provides a platform-abstracted service for working with
file paths.

Use the `Path.Path` service tag in your effect program to access path
operations.

## Accessing the Path Service

```ts
import { Path } from "@effect/platform"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const path = yield* Path.Path

  // use path operations here
})
```

## Core Operations

The `Path` interface includes:

- `basename`: get the last part of a path, optionally removing a suffix
- `dirname`: get the directory portion of a path
- `extname`: get a path extension
- `format`: convert a parsed path object to a string path
- `fromFileUrl`: convert a file URL to a path
- `isAbsolute`: check whether a path is absolute
- `join`: combine path segments
- `normalize`: resolve `.` and `..` segments
- `parse`: split a path into structured segments
- `relative`: compute relative path from one path to another
- `resolve`: resolve a sequence of paths to an absolute path
- `sep`: platform path separator (for example `/` on POSIX)
- `toFileUrl`: convert a path to a file URL
- `toNamespacedPath`: convert to a Windows namespaced path

## Example: Join Path Segments

```ts
import { Path } from "@effect/platform"
import { Effect } from "effect"
import { NodeContext, NodeRuntime } from "@effect/platform-node"

const program = Effect.gen(function* () {
  const path = yield* Path.Path

  const filePath = path.join("tmp", "file.txt")
  console.log(filePath)
})

NodeRuntime.runMain(program.pipe(Effect.provide(NodeContext.layer)))
```

On POSIX systems, this prints `tmp/file.txt`.
