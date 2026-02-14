# Class APIs

Source: extracted from `llms-full.txt` (`Class APIs`).

## Overview

`Schema.Class` lets you define a schema and a class type together.

Key benefits:

- single definition for schema + opaque class type
- class methods/getters for shared behavior
- built-in hashing/equality support via `Data.Class`
- direct constructor validation for class instances

## Defining a Schema Class

Use `Schema.Class<YourClass>("Identifier")(fields)`:

```ts
import { Schema } from "effect"

class Person extends Schema.Class<Person>("Person")({
  id: Schema.Number,
  name: Schema.NonEmptyString
}) {}
```

Create instances with either:

- `new Person({ ... })`
- `Person.make({ ... })`

The identifier (`"Person"` above) is used as a stable class identity annotation and helps avoid instance identity issues across environments such as live reload.

## Class Schemas Are Transformations

A class schema is a transformation from a struct-like encoded input to a class instance:

- decode: plain object -> class instance
- encode: class instance -> plain object

```ts
import { Schema } from "effect"

class Person extends Schema.Class<Person>("Person")({
  id: Schema.Number,
  name: Schema.NonEmptyString
}) {}

const decoded = Schema.decodeUnknownSync(Person)({ id: 1, name: "John" })
const encoded = Schema.encodeUnknownSync(Person)(decoded)
```

## Constructor Validation

`new ClassName(input)` validates input against schema fields.

- valid input creates an instance
- invalid input throws a parse error
- validation can be bypassed with:
  - `new Person(input, true)`
  - `new Person(input, { disableValidation: true })`

Use bypass options only when you intentionally accept unchecked construction.

## Equality and Hashing

Instances support value-based equality through integration with `Data.Class` / `Equal`.

- primitive and simple fields compare as expected
- nested non-data structures (for example plain arrays) use shallow behavior
- for deep value comparison in nested fields, use data-aware schemas such as `Schema.Data(...)` (for example with `Data.array(...)`)

## Custom Methods and Getters

You can add domain logic directly on schema classes:

- getters for computed properties
- methods for behavior tied to validated fields

This keeps validation + behavior close together.

## Using Classes as Schemas

A schema class can be used anywhere a schema is expected:

```ts
import { Schema } from "effect"

class Person extends Schema.Class<Person>("Person")({
  id: Schema.Number,
  name: Schema.NonEmptyString
}) {}

const Persons = Schema.Array(Person)
```

Schema classes also expose static `fields` for inspecting field definitions.

## Annotations

`Schema.Class` supports annotating three layers:

1. the "to" schema (class side)
2. the transformation schema
3. the "from" schema (input struct side)

Pass a tuple as the second argument to `Schema.Class(...)` and use `undefined` to skip layers.

The class identifier is also used as the default `identifier` annotation for the class side schema.

## Recursive Class Schemas

Use `Schema.suspend` for recursion.

```ts
import { Schema } from "effect"

class Category extends Schema.Class<Category>("Category")({
  name: Schema.String,
  subcategories: Schema.Array(
    Schema.suspend((): Schema.Schema<Category> => Category)
  )
}) {}
```

Notes:

- explicit return annotations in `Schema.suspend` are often required for correct inference
- mutually recursive class schemas are supported by suspending references between classes
- when `Type` and `Encoded` differ in recursion (for example `NumberFromString` fields), define an explicit encoded interface and annotate recursion as `Schema.Schema<Type, Encoded>`

## Tagged Variants

Schema class APIs also include tagged variants:

- `Schema.TaggedClass<...>()("Tag", fields)`
- `Schema.TaggedError<...>()("Tag", fields)`

These integrate with `_tag`-based modeling and error patterns.

## Extending Existing Classes

Use `Base.extend<Derived>("Derived")(newFields)` to add fields and behavior.

- only additional fields are allowed
- overriding existing field names is rejected (duplicate property signature)

This protects existing class logic (for example getters that depend on original field types).

## Effectful Class Transformations

For enrichment/validation pipelines, use class transformation APIs:

- `transformOrFail`: apply the new transformation at the end
- `transformOrFailFrom`: apply the new transformation from the input side

Use these when decoded entities need effectful enrichment (for example deriving fields from async sources).
