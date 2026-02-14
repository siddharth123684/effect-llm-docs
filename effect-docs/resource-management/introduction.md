# Introduction

Source: extracted from `llms-full.txt` (`Introduction`).

## Overview

Resource management is essential in long-running applications. If resources such as sockets, database connections, or file descriptors are not released correctly, systems can leak resources and degrade over time.

Effect provides structured APIs so resource cleanup still happens when effects succeed, fail, or are interrupted.

## Finalization APIs

Effect includes three common finalization operators:

| API | When it runs | Typical use |
| --- | --- | --- |
| `Effect.ensuring(finalizer)` | Always (success, failure, interruption) | unconditional cleanup |
| `Effect.onExit(handler)` | Always, with `Exit` info | cleanup that depends on outcome |
| `Effect.onError(handler)` | Only on failure/interruption | logging or failure-only cleanup |

These finalizers are designed for reliability in concurrent programs and are suitable for release logic that must run deterministically.

## `Effect.ensuring`

Use `Effect.ensuring` when you always want a finalizer to run:

```ts
import { Console, Effect } from "effect"

const program = Console.log("work").pipe(
  Effect.ensuring(Console.log("cleanup"))
)
```

## `Effect.onExit`

Use `Effect.onExit` when cleanup needs the final result (`Exit`) of the effect.

## `Effect.onError`

Use `Effect.onError` when cleanup should only run if the effect fails (including interruption-caused failure paths).

## acquireUseRelease

For bracket-style resource safety, use:

```ts
Effect.acquireUseRelease(acquire, use, release)
```

This guarantees three phases:

1. Acquire the resource.
2. Use the resource.
3. Release the resource, even if `use` fails or is interrupted.

This pattern is the standard way to safely manage lifetimes for external resources such as files, connections, and handles.

## Key Takeaway

Effect encourages a simple invariant: every resource acquisition has a matching release path. The finalization and acquire-use-release APIs make that invariant explicit and enforceable in production code.
