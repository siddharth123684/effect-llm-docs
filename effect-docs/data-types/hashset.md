---
title: HashSet
description: Learn HashSet.make, HashSet.add, HashSet.union, HashSet.has, and HashSet.mutate for immutable unique collections with O(1) membership checks when building Effect programs and pipelines.
---

# HashSet

Source: extracted from `llms-full.txt` (`HashSet`).

## Overview

`HashSet<A>` is an immutable, unordered collection of unique values.
It is optimized for fast membership checks, insertion, and removal (average
`O(1)`), while preserving functional immutability.

Effect also provides `MutableHashSet<A>`, a mutable variant with similar core
performance, but in-place updates.

## Why Use HashSet

Use `HashSet` when you need:

- uniqueness (no duplicates)
- frequent membership checks
- set algebra (`union`, `intersection`, `difference`)
- safe sharing without mutation side effects

Prefer other collections when:

- order matters (`Array`, `List`)
- you need key/value lookups (`HashMap`, `MutableHashMap`)
- you need index-based access (`Array`)

## HashSet vs MutableHashSet

| Variant | Update style | Best for |
| --- | --- | --- |
| `HashSet` | Returns a new set | Functional pipelines, shared state, predictable behavior |
| `MutableHashSet` | Mutates in place | Local, performance-sensitive mutation-heavy workflows |

For batched updates with immutable semantics, use `HashSet.mutate`:

```ts
import { HashSet } from "effect"

const original = HashSet.make(1, 2, 3)

const modified = HashSet.mutate(original, (draft) => {
  HashSet.add(draft, 4)
  HashSet.add(draft, 5)
  HashSet.remove(draft, 1)
})

console.log(HashSet.toValues(original)) // [1, 2, 3]
console.log(HashSet.toValues(modified)) // [2, 3, 4, 5]
```

## Common HashSet APIs

| Category | API | Purpose |
| --- | --- | --- |
| constructors | `HashSet.empty`, `HashSet.make`, `HashSet.fromIterable` | Create sets |
| elements | `HashSet.has`, `HashSet.some`, `HashSet.every` | Query values |
| updates | `HashSet.add`, `HashSet.remove`, `HashSet.toggle` | Add/remove/toggle |
| set ops | `HashSet.union`, `HashSet.intersection`, `HashSet.difference` | Set algebra |
| transforms | `HashSet.map`, `HashSet.flatMap`, `HashSet.filter`, `HashSet.partition` | Transform/filter |
| conversion | `HashSet.values`, `HashSet.toValues`, `HashSet.size` | Iterate/inspect |

## Equality and Uniqueness

`HashSet` uniqueness is based on Effect's `Equal`/`Hash` semantics:

- primitives are compared by value
- objects/custom types should implement `Equal` (and compatible `Hash`)
- values made with `Data`/`Schema.Data` already support value-based equality

```ts
import { Data, HashSet, pipe } from "effect"

const person1 = Data.struct({ id: 1, name: "Alice", age: 30 })
const person2 = Data.struct({ id: 1, name: "Alice", age: 30 })

const set = pipe(
  HashSet.empty(),
  HashSet.add(person1),
  HashSet.add(person2)
)

console.log(HashSet.size(set)) // 1
```

## JavaScript Interop

Both `HashSet` and `MutableHashSet` are iterable:

- use `for...of` and spread (`...`)
- use `Array.from(...)` for generic iterable conversion
- use `HashSet.toValues(...)` for an array of immutable set values

Avoid repeated set<->array conversion in hot paths, as conversion copies data.
