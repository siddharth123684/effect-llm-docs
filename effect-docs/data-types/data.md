# Data

Source: extracted from `llms-full.txt` (`Data`).

## Overview

The `Data` module provides constructors for immutable, value-based data
structures with built-in support for `Equal` and `Hash`.

Use it to model domain values where structural equality is important without
hand-writing equality or hashing logic.

## Value Equality

In plain JavaScript, object equality is reference-based (`===`). With `Data`,
equality is structural for values built with `Data` constructors.

```ts
import { Data, Equal } from "effect"

const a = Data.struct({ name: "Alice", age: 30 })
const b = Data.struct({ name: "Alice", age: 30 })

console.log(Equal.equals(a, b)) // true
console.log(Equal.equals(a, { name: "Alice", age: 30 })) // false
```

### Shallow Comparison

`Equal.equals` is shallow for nested plain objects. To get structural equality
for nested values, also construct nested fields with `Data`.

```ts
import { Data, Equal } from "effect"

const left = Data.struct({
  name: "Alice",
  nested: Data.struct({ value: 42 })
})

const right = Data.struct({
  name: "Alice",
  nested: Data.struct({ value: 42 })
})

console.log(Equal.equals(left, right)) // true
```

### struct, tuple, and array

- `Data.struct({...})`: structural object values
- `Data.tuple(...)`: structural tuples
- `Data.array([...])`: structural array-like values

These constructors give consistent value semantics across common shapes.

## Constructors for Domain Types

## case

`Data.case<A>()` creates a constructor for a plain object type `A` with
built-in equality and hashing.

```ts
import { Data, Equal } from "effect"

interface Person {
  readonly name: string
}

const Person = Data.case<Person>()

console.log(Equal.equals(Person({ name: "Alice" }), Person({ name: "Alice" })))
// true
```

### tagged

`Data.tagged("TagName")<A>()` (or inferred usage) automatically adds `_tag`
when creating tagged data, useful for discriminated unions.

```ts
import { Data } from "effect"

interface NotFound {
  readonly _tag: "NotFound"
  readonly file: string
}

const NotFound = Data.tagged<NotFound>("NotFound")
const err = NotFound({ file: "foo.txt" })
// { file: "foo.txt", _tag: "NotFound" }
```

### Class and TaggedClass

- `Data.Class<Fields>`: class-based value type with structural equality
- `Data.TaggedClass("Tag")<Fields>`: class-based value type with `_tag`

Use these when you want class ergonomics (methods/getters) with `Data`
semantics.

## Tagged Unions

`Data.TaggedEnum` and `Data.taggedEnum` help model unions of tagged structs.

- define a tagged union type with `Data.TaggedEnum<{ ... }>`
- derive constructors via `Data.taggedEnum<Union>()`
- use `$is` and `$match` for narrowing and pattern matching

```ts
import { Data } from "effect"

type RemoteData = Data.TaggedEnum<{
  Loading: {}
  Success: { readonly data: string }
  Failure: { readonly reason: string }
}>

const { Loading, Success, Failure, $match } = Data.taggedEnum<RemoteData>()

const render = $match({
  Loading: () => "loading",
  Success: ({ data }) => `success: ${data}`,
  Failure: ({ reason }) => `failure: ${reason}`
})

console.log(render(Success({ data: "ok" })))
```

For generic tagged unions, use `Data.TaggedEnum.WithGenerics`.

## Error Constructors

The module includes error-focused constructors:

- `Data.Error<Fields>`: extends native `Error` with typed fields
- `Data.TaggedError("Tag")<Fields>`: tagged error for `catchTag` / `catchTags`

```ts
import { Data, Effect } from "effect"

class NotFound extends Data.TaggedError("NotFound")<{
  message: string
  file: string
}> {}

const program = Effect.gen(function* () {
  yield* new NotFound({ message: "Missing file", file: "foo.txt" })
})
```

### Native `cause` Support

Errors created with `Data.Error` / `Data.TaggedError` can include a `cause`
field, integrating with JavaScript `Error` cause chaining.
