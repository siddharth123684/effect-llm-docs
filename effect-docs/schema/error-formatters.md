---
title: Error Formatters
description: TreeFormatter and ArrayFormatter for rendering Effect Schema parse errors.
---

# Error Formatters

Source: extracted from `llms-full.txt` (`Error Formatters`).

## Overview

Effect Schema parse errors can be rendered with two built-in formatters:

- `ParseResult.TreeFormatter` (default, hierarchical output)
- `ParseResult.ArrayFormatter` (array of structured issue objects)

During decoding, many APIs return only the first issue by default. Use parse options
like `{ errors: "all" }` to collect and format all issues.

## TreeFormatter (default)

`TreeFormatter` prints nested failures as a readable tree (schema title -> path ->
issue).

```ts
import { Either, ParseResult, Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.String,
  age: Schema.Number
})

const decode = Schema.decodeUnknownEither(Person, { errors: "all" })
const result = decode({})

if (Either.isLeft(result)) {
  console.error(ParseResult.TreeFormatter.formatErrorSync(result.left))
}
```

Typical output structure:

- root title (schema identity / representation)
- one branch per failing path
- leaf message (for example `is missing`, type mismatch details)

## Customizing Tree Titles

You can make tree output clearer with schema annotations:

- `identifier`
- `title`
- `description`

These can replace the default TypeScript-like schema representation shown at the root
and at nested nodes.

`parseIssueTitle` provides dynamic titles per issue:

```ts
type ParseIssueTitleAnnotation = (
  issue: ParseResult.ParseIssue
) => string | undefined
```

Resolution notes:

- If `message` annotation produces a message, it takes precedence.
- If `parseIssueTitle` returns `undefined`, formatter title fallback is:
  `identifier` -> `title` -> `description` -> default schema representation.

## ArrayFormatter

`ArrayFormatter` emits issues as objects, which is useful for programmatic handling
or custom UI rendering.

```ts
import { Either, ParseResult, Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.String,
  age: Schema.Number
})

const result = Schema.decodeUnknownEither(Person, { errors: "all" })({})

if (Either.isLeft(result)) {
  console.error(ParseResult.ArrayFormatter.formatErrorSync(result.left))
}
```

Each issue object includes:

- `_tag`: issue kind (for example `Missing`)
- `path`: location in input data
- `message`: human-readable issue text

## React Hook Form Integration

For React forms, `@hookform/resolvers` includes an adapter for `effect/Schema`.

Reference:

- `https://www.npmjs.com/package/@hookform/resolvers#effect-ts`
