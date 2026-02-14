# Branded Types

Source: extracted from `llms-full.txt` (`Branded Types`).

## Overview

Branded types let you create distinct compile-time types from the same runtime
representation. In Effect, branding is provided by the `Brand` module and is
useful when plain TypeScript aliases are too permissive.

## Why This Matters

TypeScript is structurally typed, so aliases like `type UserId = number` and
`type ProductId = number` are interchangeable. Branding prevents accidental
mix-ups by attaching a unique type-level tag.

```ts
import { Brand } from "effect"

type UserId = number & Brand.Brand<"UserId">
type ProductId = number & Brand.Brand<"ProductId">
```

With branding in place, a `UserId` cannot be passed where a `ProductId` is
expected.

## Constructing Branded Values

### `Brand.nominal`

Use `Brand.nominal` when you only need type distinction (no runtime checks):

```ts
import { Brand } from "effect"

type ProductId = number & Brand.Brand<"ProductId">
const ProductId = Brand.nominal<ProductId>()
```

### `Brand.refined`

Use `Brand.refined` when construction must validate input at runtime:

```ts
import { Brand } from "effect"

type Int = number & Brand.Brand<"Int">

const Int = Brand.refined<Int>(
  (n) => Number.isInteger(n),
  (n) => Brand.error(`Expected ${n} to be an integer`)
)
```

Invalid values return `BrandErrors` details via `Brand.error`.

## Combining Brands

You can compose branded constructors with `Brand.all` and derive the resulting
type from the combined constructor:

```ts
import { Brand } from "effect"

const PositiveInt = Brand.all(Int, Positive)
type PositiveInt = Brand.Brand.FromConstructor<typeof PositiveInt>
```

## Practical Guidance

- Use branding for domain identifiers and constrained primitives.
- Prefer `nominal` for pure type-level separation.
- Prefer `refined` when invalid input must be rejected at construction time.
