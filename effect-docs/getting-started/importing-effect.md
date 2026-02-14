---
title: Importing Effect
description: Installing the effect package and basic named imports for Effect APIs.
---

# Importing Effect

Source: extracted from `llms-full.txt` (`Importing Effect`).

## Overview

If you are new to Effect, you do not need to learn every module at once.
In most cases, installing the `effect` package is enough to begin.

## Install `effect`

```sh
npm install effect
# pnpm add effect
# yarn add effect
# bun add effect
# deno add npm:effect
```

## Basic Import

Use a standard named import:

```ts
import { Effect } from "effect"
```

This gives you access to the core `Effect` API for creating and composing effectful programs.

## Namespace Import Alternative

You can also import the module namespace directly:

```ts
import * as Effect from "effect/Effect"
```

Both styles work. Namespace imports can be a safer choice for tree shaking when bundlers do not perform deep scope analysis.

## Tree Shaking Note

- Named imports may cause larger bundles on toolchains without deep scope analysis.
- Bundlers such as Rollup and Webpack 5+ generally handle named imports correctly.

## Why Effect Uses Functions (Not Methods)

Effect APIs are function-first because:

- **Tree shakeability**: unused imported functions can be removed from bundles.
- **Extendibility**: plain functions are easier to extend than prototype-based methods.

## Next Step

After imports, continue with the `Effect` type fundamentals before moving into more advanced modules.
