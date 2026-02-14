---
title: Installation
description: Learn to set up Effect for Node.js, Deno, Bun, and Vite projects with TypeScript 5.4+ and strict compiler options for type-safe development.
---

# Installation

Source: extracted from `llms-full.txt` (`Installation`).

## Overview

Effect requires:

- TypeScript 5.4 or newer
- a supported runtime (Node.js, Deno, or Bun)

Use strict typing in your TypeScript config:

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

## Install in an Existing Project

```sh
npm install effect
# pnpm add effect
# yarn add effect
# bun add effect
# deno add npm:effect
```

## Manual Setup: Node.js

Create and initialize a TypeScript project:

```sh
mkdir hello-effect
cd hello-effect
npm init -y
npm install --save-dev typescript
npx tsc --init
npm install effect
```

Create `src/index.ts`:

```ts
import { Console, Effect } from "effect"

const program = Console.log("Hello, World!")

Effect.runSync(program)
```

Run it:

```sh
npx tsx src/index.ts
```

If setup is correct, it prints `"Hello, World!"`.

## Manual Setup: Deno

```sh
mkdir hello-effect
cd hello-effect
deno init
deno add npm:effect
```

Update `main.ts` with the same hello-world program and run:

```sh
deno run main.ts
```

## Manual Setup: Bun

```sh
mkdir hello-effect
cd hello-effect
bun init
bun add effect
```

Update `index.ts` with the same hello-world program and run:

```sh
bun index.ts
```

## Vite + React (Optional Starter)

Scaffold a React + TypeScript app, install dependencies, then add Effect:

```sh
npm create vite@latest hello-effect -- --template react-ts
cd hello-effect
npm install
npm install effect
```

Example use in `src/App.tsx`:

```tsx
import { useMemo, useState } from "react"
import { Effect } from "effect"

function App() {
  const [count, setCount] = useState(0)
  const task = useMemo(
    () => Effect.sync(() => setCount((current) => current + 1)),
    []
  )

  return <button onClick={() => Effect.runSync(task)}>count is {count}</button>
}
```

Start the dev server with your package manager (`npm run dev`, `pnpm run dev`, etc.) and verify the app runs.

## Next Step

After installation works, continue with `The Effect Type`.
