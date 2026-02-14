---
title: Equivalence
description: Schema.equivalence generates value comparison functions from schema rules with custom annotations.
---

# Equivalence

Source: extracted from `llms-full.txt` (`Schema to Equivalence`).

## Overview

`Schema.equivalence` generates an equivalence function from a schema.
The generated function compares values using schema rules instead of ad-hoc checks.

```ts
import { Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.String,
  age: Schema.Number
})

const personEquivalence = Schema.equivalence(Person)

const john = { name: "John", age: 23 }
const alice = { name: "Alice", age: 30 }

console.log(personEquivalence(john, { name: "John", age: 23 })) // true
console.log(personEquivalence(john, alice)) // false
```

## Any, Unknown, Object, and Empty Struct

For broad schemas like `Schema.Any`, `Schema.Unknown`, `Schema.Object`, and
`Schema.Struct({})`, equivalence falls back to `Equal.equals` semantics
(reference equality by default).

```ts
import { Schema } from "effect"

const schema = Schema.Struct({})

const input1 = {}
const input2 = {}

console.log(Schema.equivalence(schema)(input1, input2)) // false
```

## Customizing Equivalence with Annotations

You can override generated behavior by adding an `equivalence` annotation to a schema.
The annotation returns a comparison function.

```ts
import { Schema } from "effect"

const schema = Schema.String.annotations({
  equivalence: () => (s1, s2) => s1.charAt(0) === s2.charAt(0)
})

const customEquivalence = Schema.equivalence(schema)

console.log(customEquivalence("aaa", "abb")) // true
console.log(customEquivalence("aaa", "bba")) // false
```
