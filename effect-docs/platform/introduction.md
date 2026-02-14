# Introduction

Source: extracted from `llms-full.txt` (`Introduction to Effect Platform`).

## Overview

`@effect/platform` provides platform-independent services for applications that
run on Node.js, Deno, Bun, and browsers.

It lets you program against abstract services (for example `FileSystem`,
`Terminal`, and `Path`) and provide platform-specific implementations at the
edge of your application using layers.

## Platform Packages

Use these packages to provide implementations for a target runtime:

- `@effect/platform-node` for Node.js and Deno
- `@effect/platform-bun` for Bun
- `@effect/platform-browser` for browsers

## Stable Modules

The following modules are documented as stable:

- `Command`: interact with command-line programs
- `FileSystem`: read and write files
- `KeyValueStore`: key-value storage operations
- `Path`: path manipulation utilities
- `PlatformLogger`: file-backed logging with `FileSystem`
- `Runtime`: run programs with platform runtime integration
- `Terminal`: terminal input/output interaction

## Unstable Modules

These modules are marked unstable/experimental and may change:

- `Http API`
- `Http Client`
- `Http Server`
- `Socket`
- `Worker`

For current details, refer to the `@effect/platform` README.

## Installation

```sh
npm install @effect/platform
```

```sh
pnpm add @effect/platform
```

```sh
yarn add @effect/platform
```

```sh
bun add @effect/platform
```

```sh
deno add npm:@effect/platform
```

## Cross-Platform Example (`Path`)

```ts
import { Path } from "@effect/platform"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const path = yield* Path.Path
  const filePath = path.join("tmp", "file.txt")
  console.log(filePath)
})
```

This program is platform-agnostic until you provide a platform layer.

## Running with Node.js or Deno

Install:

```sh
npm install @effect/platform-node
```

Provide runtime context:

```ts
import { Path } from "@effect/platform"
import { Effect } from "effect"
import { NodeContext, NodeRuntime } from "@effect/platform-node"

const program = Effect.gen(function* () {
  const path = yield* Path.Path
  console.log(path.join("tmp", "file.txt"))
})

NodeRuntime.runMain(program.pipe(Effect.provide(NodeContext.layer)))
```

## Running with Bun

Install:

```sh
bun add @effect/platform-bun
```

Provide runtime context:

```ts
import { Path } from "@effect/platform"
import { Effect } from "effect"
import { BunContext, BunRuntime } from "@effect/platform-bun"

const program = Effect.gen(function* () {
  const path = yield* Path.Path
  console.log(path.join("tmp", "file.txt"))
})

BunRuntime.runMain(program.pipe(Effect.provide(BunContext.layer)))
```
