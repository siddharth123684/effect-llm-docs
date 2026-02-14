---
title: Equal
description: Value-aware equality checks via Equal.equals for objects beyond reference equality.
---

# Equal

Source: extracted from `llms-full.txt` (`Equal`).

## Overview

`Equal` provides value-aware equality checks in Effect through `Equal.equals`.
It is designed to avoid relying only on JavaScript reference equality when
comparing objects.

Key points:

- `Equal.equals` works with values that implement the `Equal` interface.
- If a value does not implement `Equal`, comparison falls back to `===`.
- `Equal` is commonly paired with `Hash` for efficient comparisons in
  hash-based collections.

## Why `Equal.equals` Instead of `===`

JavaScript objects are usually compared by reference:

```ts
import { Equal } from "effect"

const a = { name: "Alice", age: 30 }
const b = { name: "Alice", age: 30 }

console.log(Equal.equals(a, b))
// false (fallback behaves like === for plain objects)
```

Although `a` and `b` have the same fields, they are different instances.

## Implementing `Equal` (and `Hash`) in Custom Types

For custom value semantics, implement `Equal.Equal` and `Hash`:

```ts
import { Equal, Hash } from "effect"

class Person implements Equal.Equal {
  constructor(
    readonly id: number,
    readonly name: string,
    readonly age: number
  ) {}

  [Equal.symbol](that: Equal.Equal): boolean {
    return (
      that instanceof Person &&
      Equal.equals(this.id, that.id) &&
      Equal.equals(this.name, that.name) &&
      Equal.equals(this.age, that.age)
    )
  }

  [Hash.symbol](): number {
    return Hash.hash(this.id)
  }
}
```

Using `Hash` lets Effect quickly rule out inequality before running deeper
equality checks.

## Using the `Data` Module for Value Equality

When you want value equality without manual interface implementations, use
`Data` constructors, which provide `Equal` and `Hash` behavior automatically.

```ts
import { Data, Equal } from "effect"

const alice = Data.struct({ id: 1, name: "Alice", age: 30 })

console.log(Equal.equals(alice, Data.struct({ id: 1, name: "Alice", age: 30 })))
// true

console.log(Equal.equals(alice, { id: 1, name: "Alice", age: 30 }))
// false
```

## Collections and Equality Semantics

`HashSet` and `HashMap` use `Equal` semantics:

- with `Data` values (or custom `Equal` implementations), comparisons are
  value-based
- without `Equal` support, object comparisons effectively behave like
  reference equality

This is why hash-based Effect collections are preferred when key/value equality
must be based on content rather than object identity.
