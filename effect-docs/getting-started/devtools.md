---
title: Devtools
description: Effect Language Service (LSP) and VS Code extension for diagnostics, debugging, tracing, and metrics.
---

# Devtools

Source: extracted from `llms-full.txt` (`Devtools`).

## Overview

Effect Devtools for getting started are centered on:

- the Effect Language Service (LSP) for code intelligence and diagnostics
- the VS Code / Cursor extension for debugging, tracing, and metrics views

## Effect Language Service (LSP)

The LSP adds Effect-aware diagnostics, quick info, completions, and refactors in editors that support the TypeScript language service.

### Install

Install the language service as a dev dependency:

```sh
npm install --save-dev @effect/language-service
# pnpm add -D @effect/language-service
# yarn add --dev @effect/language-service
# bun add --dev @effect/language-service
```

Add the plugin to `tsconfig.json`:

```json
{
  "compilerOptions": {
    "plugins": [
      {
        "name": "@effect/language-service"
      }
    ]
  }
}
```

Ensure your editor uses the workspace TypeScript version (not the bundled editor version), or the plugin will not activate correctly.

### What You Get

- **Diagnostics**: catches floating effects, layer issues, invalid error handling, and version conflicts
- **Quick Info**: richer effect type information, including generator and layer details
- **Completions**: scaffolds for common Effect patterns
- **Refactors**: async-to-Effect conversion, accessor generation, and pipe/style transforms

### Build-Time Diagnostics

You can patch local TypeScript so Effect diagnostics also appear during type checking:

```sh
effect-language-service patch
```

To make this automatic for contributors:

```json
{
  "scripts": {
    "prepare": "effect-language-service patch"
  }
}
```

## VS Code / Cursor Extension

The extension is separate from the LSP and focuses on runtime debugging tools for Effect programs.

- **Context**: inspect paused fiber context
- **Span Stack**: inspect telemetry span history
- **Fibers**: inspect and interrupt running fibers
- **Breakpoints**: pause on defects

## Built-in Tracer and Metrics View

Install the experimental package:

```sh
npm install @effect/experimental
# pnpm install @effect/experimental
# yarn add @effect/experimental
# bun add @effect/experimental
```

Use `DevTools.layer()` in your app:

```ts
import { DevTools } from "@effect/experimental"
import { NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const program = Effect.log("Hello!").pipe(Effect.forever)
const DevToolsLive = DevTools.layer()

program.pipe(Effect.provide(DevToolsLive), NodeRuntime.runMain)
```

If you also use `@effect/opentelemetry`, provide the DevTools layer before tracing layers so tracer patching works correctly.
