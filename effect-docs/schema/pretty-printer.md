---
title: Pretty Printer
description: Pretty.make for building pretty-printer functions from schemas for readable string output.
---

# Pretty Printer

Source: extracted from `llms-full.txt` (`Schema to Pretty Printer`).

## Overview

`Pretty.make` builds a pretty-printer function from a schema. The returned function
formats valid values into a readable string representation.

```ts
import { Pretty, Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.String,
  age: Schema.Number
})

const prettyPerson = Pretty.make(Person)

console.log(prettyPerson({ name: "Alice", age: 30 }))
// '{ "name": "Alice", "age": 30 }'
```

## Customizing Pretty Generation

You can override formatting by adding a `pretty` annotation to the schema.

- The annotation returns a formatter function.
- It can use schema type parameters (`typeParameters`) when needed.

```ts
import { Pretty, Schema } from "effect"

const schema = Schema.Number.annotations({
  pretty: (/* typeParameters */) => (value) => `my format: ${value}`
})

const customPretty = Pretty.make(schema)

console.log(customPretty(1))
// "my format: 1"
```
