---
title: Redacted
description: Redacted wraps sensitive values to hide them from logs and error output.
---

# Redacted

Source: extracted from `llms-full.txt` (`Redacted`).

## Overview

`Redacted<A>` is a data type for handling sensitive values (for example API keys
or tokens) without exposing them in logs or error output.

When printed, redacted values are intentionally hidden (for example
`"<redacted>"`).

## Creating Redacted Values

Use `Redacted.make` to wrap a value:

```ts
import { Redacted } from "effect"

const apiKey = Redacted.make("1234567890")
```

This helps avoid accidental disclosure in normal logging paths.

## Accessing the Underlying Value

Use `Redacted.value` only when you explicitly need the original value:

```ts
import { Redacted } from "effect"

const apiKey = Redacted.make("1234567890")
const raw = Redacted.value(apiKey)
```

`Redacted.value` reveals the secret, so avoid passing the result to logs.

## Wiping Sensitive Data

Use `Redacted.unsafeWipe` to erase the stored value when no longer needed:

```ts
import { Redacted } from "effect"

const apiKey = Redacted.make("1234567890")
Redacted.unsafeWipe(apiKey)
```

After wiping, reading the value throws.

## Comparing Redacted Values

Use `Redacted.getEquivalence` with an equivalence for the underlying type:

```ts
import { Redacted, Equivalence } from "effect"

const eq = Redacted.getEquivalence(Equivalence.string)
```

This allows equality checks without changing how values are redacted in output.
