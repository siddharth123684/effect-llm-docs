# Metrics

Source: extracted from `llms-full.txt` (`Metrics in Effect`).

## Overview

Metrics in Effect help you observe behavior in concurrent systems (for example, request rates, latency distributions, and error frequencies) without changing your effect return types.

You attach metrics to effects, run the program as usual, and inspect metric state when needed.

## Core Metric Types

Effect supports five metric families:

- **Counter**: tracks cumulative change over time (can increase or decrease).
- **Gauge**: tracks the latest value at a point in time.
- **Histogram**: tracks value distribution in configured buckets.
- **Summary**: tracks quantiles over a sliding window.
- **Frequency**: counts occurrences of distinct string values.

## Common Workflow

1. Create a metric (`Metric.counter`, `Metric.gauge`, etc.).
2. Apply the metric to an effect.
3. Run the effect.
4. Read metric state via `Metric.value(metric)`.

```ts
import { Effect, Metric } from "effect"

const requestCount = Metric.counter("request_count")

const program = Effect.gen(function* () {
  yield* requestCount(Effect.succeed(1))
  yield* requestCount(Effect.succeed(2))
  const state = yield* Metric.value(requestCount)
  return state
})
```

Applying a metric does not change the wrapped effect's success or error types; it adds observation only.

## Counter

Use `Metric.counter(name, options?)` for running totals such as request counts or completed jobs.

- Supports `number` (default) or `bigint` via `bigint: true`.
- Set `incremental: true` for increment-only counters (negative updates are ignored).
- Use `Metric.withConstantInput(n)` when each invocation should add a fixed amount.

## Gauge

Use `Metric.gauge(name, options?)` for values that move up and down (for example, queue depth or memory usage).

- Gauge state stores the latest observed value.
- Supports `number` (default) or `bigint`.

## Histogram and Timers

Use `Metric.histogram` when bucketed distributions are useful (for example, latency ranges).

```ts
import { Effect, Metric, MetricBoundaries, Random } from "effect"

const latency = Metric.histogram(
  "request_latency",
  MetricBoundaries.linear({ start: 0, width: 10, count: 11 }),
  "Latency distribution"
)

const program = latency(Random.nextIntBetween(1, 120)).pipe(Effect.repeatN(99))
```

For duration tracking, Effect provides timer helpers:

- `Metric.timerWithBoundaries(...)` to build a histogram timer.
- `Metric.trackDuration(effect, timer)` to record runtime duration automatically.

## Summary

Use `Metric.summary(...)` for quantiles over a sliding window when fixed histogram buckets are not a good fit.

Key parameters:

- `maxAge`: how long a sample is kept.
- `maxSize`: max number of samples retained.
- `error`: tolerated quantile error margin.
- `quantiles`: quantiles to compute (for example, `0.5`, `0.95`, `0.99`).

Summaries are usually instance-local and not ideal when you need straightforward cross-instance aggregation.

## Frequency

Use `Metric.frequency(name, options?)` to count string occurrences (for example, error code frequency).

Each distinct string gets its own counter-like entry in metric state.

## Tagging Metrics

Tags add context (key/value metadata) for filtering and analysis.

- `Metric.tagged(key, value)`: tag a single metric.
- `Effect.tagMetrics(key, value)`: tag all metrics in an effect context.
- `Effect.tagMetricsScoped(key, value)`: scoped/global-tag control.

```ts
import { Effect, Metric } from "effect"

const counter = Metric.counter("request_count").pipe(
  Metric.tagged("environment", "production")
)

const program = counter(Effect.succeed(1)).pipe(
  Effect.tagMetrics("service", "api")
)
```

## Practical Guidance

- Use **Counter** for totals, **Gauge** for current levels.
- Use **Histogram** for aggregatable latency distribution.
- Use **Summary** when quantiles over recent samples matter more than bucket aggregation.
- Use **Frequency** for categorical string events.
- Add tags early to keep dashboards/querying clean.
