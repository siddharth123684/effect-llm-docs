# KeyValueStore

Source: extracted from `llms-full.txt` (`KeyValueStore`).

## Overview

`@effect/platform/KeyValueStore` provides an effectful interface for key-value storage.
It supports asynchronous access patterns and can be provided through layers, so programs stay implementation-agnostic.

## Accessing the Service

Use the `KeyValueStore` service from Effect context:

```ts
import { KeyValueStore } from "@effect/platform"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const kv = yield* KeyValueStore.KeyValueStore
  return kv
})
```

## Core Operations

Key operations exposed by the interface:

- `get(key)`: read a `string` value as `Option<string>`
- `getUint8Array(key)`: read binary values as `Option<Uint8Array>`
- `set(key, value)`: write a value
- `remove(key)`: delete one key
- `clear`: remove all entries
- `size`: count entries
- `has(key)`: check key existence
- `isEmpty`: check whether the store has entries
- `modify` / `modifyUint8Array`: update an existing value conditionally
- `forSchema(schema)`: derive a typed schema-backed store

## Built-In Implementations

Effect Platform ships two standard layers:

- `layerMemory`: in-memory store, useful for tests and lightweight usage
- `layerFileSystem`: file-system-backed store for persistence

Example using the in-memory layer:

```ts
import { Effect } from "effect"
import { KeyValueStore, layerMemory } from "@effect/platform/KeyValueStore"

const program = Effect.gen(function* () {
  const kv = yield* KeyValueStore
  yield* kv.set("key", "value")
  return yield* kv.get("key")
})

Effect.runPromise(program.pipe(Effect.provide(layerMemory)))
```

## Typed Storage with Schemas

By default, values are `string` / `Uint8Array`. For typed objects, use `forSchema` to create a `SchemaStore`.
Values are validated against the schema and encoded/decoded through JSON serialization.

```ts
import { Effect, Schema } from "effect"
import { KeyValueStore, layerMemory } from "@effect/platform/KeyValueStore"

const Person = Schema.Struct({ name: Schema.String, age: Schema.Number })

const program = Effect.gen(function* () {
  const users = (yield* KeyValueStore).forSchema(Person)
  yield* users.set("user:1", { name: "Alice", age: 30 })
  return yield* users.get("user:1")
})

Effect.runPromise(program.pipe(Effect.provide(layerMemory)))
```
