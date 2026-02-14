---
title: Tracing
description: How to instrument Effect programs with spans and traces using OpenTelemetry integration.
---

# Tracing

Source: extracted from `llms-full.txt` (`Tracing`).

## Overview

Tracing helps you understand the full lifecycle of a request across services. Logs and metrics are useful, but tracing shows how work is connected end-to-end and where time is spent.

## Core Concepts

### Span

A span represents one operation (a unit of work) in a request flow.

Common span data:

- `name` of the operation
- start time and duration
- attributes (key/value metadata)
- events (including converted logs)
- status (for example OK vs error)

### Trace

A trace is a tree of related spans for one request. The first span is the root span, and child spans represent nested or downstream work.

Tracing backends usually render traces as waterfall views, making bottlenecks easy to spot.

## Creating Spans

Use `Effect.withSpan("name")` to instrument an effect:

```ts
import { Effect } from "effect"

const program = Effect.void.pipe(
  Effect.delay("100 millis"),
  Effect.withSpan("myspan")
)
```

Adding a span does not change the effect type (`Effect<A, E, R>` stays the same type shape).

## Printing Spans with OpenTelemetry

Typical dependencies:

- `@effect/opentelemetry`
- `@opentelemetry/sdk-trace-base`
- `@opentelemetry/sdk-trace-node` (Node.js) or `@opentelemetry/sdk-trace-web` (browser)
- `@opentelemetry/sdk-metrics` (if you also export metrics)
- `@opentelemetry/api` (peer dependency of `@effect/opentelemetry`)

Minimal setup to print spans to console:

```ts
import { Effect } from "effect"
import { NodeSdk } from "@effect/opentelemetry"
import {
  BatchSpanProcessor,
  ConsoleSpanExporter
} from "@opentelemetry/sdk-trace-base"

const program = Effect.void.pipe(
  Effect.delay("100 millis"),
  Effect.withSpan("myspan")
)

const NodeSdkLive = NodeSdk.layer(() => ({
  resource: { serviceName: "example" },
  spanProcessor: new BatchSpanProcessor(new ConsoleSpanExporter())
}))

Effect.runPromise(program.pipe(Effect.provide(NodeSdkLive)))
```

## Reading Span Output

Important fields:

| Field | Meaning |
| --- | --- |
| `traceId` | Identifier for the whole trace. |
| `parentId` | Parent span id (`undefined` means root span). |
| `id` | Current span id. |
| `name` | Operation name. |
| `timestamp` / `duration` | Start time and runtime. |
| `attributes` | Extra context metadata. |
| `events` | Timeline events attached to the span. |
| `status` | Span result (`code: 1` OK, `code: 2` ERROR). |

## Error Spans

If an instrumented effect fails, the span status is recorded as error (`code: 2`), and exception details are emitted in span events.

## Annotations and Log Events

Add custom metadata to the current span:

- `Effect.annotateCurrentSpan("key", "value")`

Logs inside a span become span events. For example, `Effect.log("Hello")` appears under the span `events` list with log-related attributes.

## Nesting Spans

You can nest spans by instrumenting child and parent effects separately:

- child effect: `Effect.withSpan("child")`
- parent effect: `Effect.withSpan("parent")`

In output, `child.parentId` matches `parent.id`.

## Visualizing Traces Locally (Tutorial Flow)

High-level flow from the docs:

1. Run an OpenTelemetry backend stack (for example `docker.io/grafana/otel-lgtm`).
2. Export traces from Effect using OpenTelemetry (for example OTLP HTTP exporter).
3. Open Grafana at `http://localhost:3000/explore` and inspect generated trace IDs.

## Integration Example: Sentry

To send spans to Sentry, use Sentry's span processor:

```ts
import { NodeSdk } from "@effect/opentelemetry"
import { SentrySpanProcessor } from "@sentry/opentelemetry"

const NodeSdkLive = NodeSdk.layer(() => ({
  resource: { serviceName: "example" },
  spanProcessor: new SentrySpanProcessor()
}))
```

## Key APIs

| API | Purpose |
| --- | --- |
| `Effect.withSpan` | Wrap an effect in a span. |
| `Effect.annotateCurrentSpan` | Attach attributes to the active span. |
| `Effect.log` | Produces log events that appear as span events when tracing is enabled. |
| `NodeSdk.layer` | Provide tracing runtime configuration via OpenTelemetry. |
| `BatchSpanProcessor` | Buffer/process spans before exporting. |
| `ConsoleSpanExporter` | Print spans to stdout (debugging). |
