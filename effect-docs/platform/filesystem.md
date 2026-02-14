# FileSystem

Source: extracted from `llms-full.txt` (`FileSystem`).

## Overview

`@effect/platform/FileSystem` provides a platform-abstracted file system service for reading, writing, and managing files and directories.

Programs access it through the `FileSystem` service tag:

```ts
import { FileSystem } from "@effect/platform"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const fs = yield* FileSystem.FileSystem
  // use fs operations here
})
```

## Core Operations

The interface includes operations for:

- Access and metadata: `access`, `exists`, `stat`, `realPath`, `utimes`
- Reading and listing: `readFile`, `readFileString`, `readDirectory`, `readLink`
- Writing and streaming: `writeFile`, `writeFileString`, `truncate`, `open`, `sink`, `stream`
- File and directory management: `copy`, `copyFile`, `rename`, `remove`, `makeDirectory`
- Temporary resources: `makeTempDirectory`, `makeTempDirectoryScoped`, `makeTempFile`, `makeTempFileScoped`
- Links and ownership: `link`, `symlink`, `chmod`, `chown`
- Change notifications: `watch`

## Example: Read a File as UTF-8 Text

```ts
import { FileSystem } from "@effect/platform"
import { NodeContext, NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const fs = yield* FileSystem.FileSystem
  const content = yield* fs.readFileString("./index.ts", "utf8")
  console.log(content)
})

NodeRuntime.runMain(program.pipe(Effect.provide(NodeContext.layer)))
```

## Mocking in Tests with `layerNoop`

`FileSystem.layerNoop` provides a no-op implementation for tests. Most methods fail or die by default, and you can override specific methods to control behavior.

```ts
import { FileSystem } from "@effect/platform"
import { Effect } from "effect"

const customMock = FileSystem.layerNoop({
  readFileString: () => Effect.succeed("mocked content"),
  exists: (path) => Effect.succeed(path === "/some/path")
})

const program = Effect.gen(function* () {
  const fs = yield* FileSystem.FileSystem
  const exists = yield* fs.exists("/some/path")
  const content = yield* fs.readFileString("/some/path")
  console.log(exists, content)
})

Effect.runPromise(program.pipe(Effect.provide(customMock)))
```
