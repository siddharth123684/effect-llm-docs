# BigDecimal

Source: extracted from `llms-full.txt` (`BigDecimal`).

## Overview

`BigDecimal` is Effect's decimal type for precision-sensitive arithmetic.
It avoids floating-point rounding errors common with JavaScript `number`
values, which is important for domains like finance and statistics.

## Representation Model

A `BigDecimal` is represented by:

1. `value`: a `bigint` containing the significant digits
2. `scale`: a signed 64-bit integer indicating decimal placement

The numeric value is:

`value * 10^(-scale)`

Examples:

- `value = 12345n`, `scale = 2` -> `123.45`
- `value = 12345n`, `scale = -2` -> `1234500`

## Creating Values

### Constructors and Parsers

- `BigDecimal.make(value, scale)`: build from raw parts
- `BigDecimal.fromBigInt(n)`: create integer decimal (`scale = 0`)
- `BigDecimal.fromString(text)`: safe parse, returns `Option<BigDecimal>`
- `BigDecimal.unsafeFromString(text)`: throws on invalid input
- `BigDecimal.unsafeFromNumber(n)`: throws for non-finite numbers

Use `fromString` when input can be invalid. Prefer string parsing over direct
floating-point conversion when precision matters.

```ts
import { BigDecimal, Option } from "effect"

const a = BigDecimal.make(1n, 2) // 0.01
const b = BigDecimal.fromBigInt(10n) // 10
const parsed = BigDecimal.fromString("0.02")

if (Option.isSome(parsed)) {
  console.log(BigDecimal.format(parsed.value)) // "0.02"
}
```

## Formatting and Display

- `String(decimal)`: debug-style representation (`BigDecimal(...)`)
- `BigDecimal.format(decimal)`: standard decimal string
- `BigDecimal.toExponential(decimal)`: exponential notation

## Arithmetic Operations

Core arithmetic includes:

- `sum`, `subtract`, `multiply`
- `divide` (safe, returns `Option<BigDecimal>`)
- `unsafeDivide` (throws on divide-by-zero)
- `remainder` (safe), `unsafeRemainder` (throws on divide-by-zero)
- `negate`, `abs`, `sign`

```ts
import { BigDecimal } from "effect"

const x = BigDecimal.unsafeFromString("1.05")
const y = BigDecimal.unsafeFromString("2.10")

console.log(String(BigDecimal.sum(x, y))) // BigDecimal(3.15)
console.log(String(BigDecimal.multiply(x, y))) // BigDecimal(2.205)
console.log(BigDecimal.divide(y, x)) // Option.Some(BigDecimal(2))
```

## Comparison and Predicates

Use these to compare and classify values:

- Ordering: `lessThan`, `lessThanOrEqualTo`, `greaterThan`, `greaterThanOrEqualTo`
- Selection: `min`, `max`
- Predicates: `isZero`, `isPositive`, `isNegative`, `isInteger`
- Range check: `between({ minimum, maximum })(value)`

## Normalization and Equality

Different internal `(value, scale)` pairs can represent the same numeric value.
For example, `105n/2` and `1050n/3` both represent `1.05`.

- `BigDecimal.normalize(decimal)`: canonicalize representation (trim trailing zeros)
- `BigDecimal.equals(a, b)`: numeric equality independent of representation

```ts
import { BigDecimal } from "effect"

const a = BigDecimal.make(105n, 2)
const b = BigDecimal.make(1050n, 3)

console.log(BigDecimal.equals(a, b)) // true
console.log(BigDecimal.normalize(b)) // { _id: "BigDecimal", value: "105", scale: 2 }
```

## Practical Guidance

- Prefer string-based construction for user or external numeric input.
- Use safe APIs (`fromString`, `divide`, `remainder`) when failures are expected.
- Use unsafe variants only when input invariants are guaranteed.
