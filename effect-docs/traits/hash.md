---
title: Hash
description: Hash interface for efficient equality checks in hash-based collections with Equal.
---

# Hash

Source: extracted from `llms-full.txt` (`Hash`).

## Overview

The `Hash` interface supports efficient equality checks and is designed to work
with `Equal`. Instead of always performing a full value comparison, Effect can
compare hash values first to short-circuit obvious non-equal cases.

## Role in Equality Checking

When two values implement `Equal`, their hash values are compared first:

- If hashes are different, the values are definitely not equal.
- If hashes are the same, the values might be equal, so `Equal` is used for a
  full comparison.

This pattern improves performance in hash-based data structures such as hash
maps and hash sets.

## Implementing `Hash` with `Equal`

For custom classes, implement both `Equal` and `Hash` so values can participate
in value-based comparisons efficiently.

```ts
import { Equal, Hash } from "effect"

class Person implements Equal.Equal {
  constructor(
    readonly id: number,
    readonly name: string,
    readonly age: number
  ) {}

  [Equal.symbol](that: Equal.Equal): boolean {
    if (that instanceof Person) {
      return (
        Equal.equals(this.id, that.id) &&
        Equal.equals(this.name, that.name) &&
        Equal.equals(this.age, that.age)
      )
    }
    return false
  }

  [Hash.symbol](): number {
    return Hash.hash(this.id)
  }
}
```

In this example, `Equal` compares `id`, `name`, and `age`, while `Hash`
provides a numeric hash used to accelerate comparisons.
