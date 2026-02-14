# Standard Schema

Source: extracted from `llms-full.txt` (`Standard Schema`).

## Overview

`Schema.standardSchemaV1` converts an Effect schema into a [Standard Schema v1](https://standardschema.dev/) object.

```ts
import { Schema } from "effect"

const schema = Schema.Struct({
  name: Schema.String
})

const standardSchema = Schema.standardSchemaV1(schema)
```

## Restrictions

Only schemas without dependencies can be converted. In Effect terms, the schema context must be `R = never`.

## Validation Behavior

Use `standardSchema["~standard"].validate(input)` to decode and validate input.

- For synchronous schemas, validation returns a value immediately.
- If the schema contains asynchronous logic (for example async checks or async message resolution), validation returns a `Promise`.

Invalid input is reported with `issues` entries that include a `message` and, when available, a `path`.

## Defects

If validation hits an unexpected defect, it is still reported as an issue instead of throwing:

- the issue contains the defect message
- the issue has no `path`

This keeps validation result handling consistent even for internal failures.
