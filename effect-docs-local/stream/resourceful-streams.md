# Resourceful Streams

Source: extracted from `llms-full.txt` (`Resourceful Streams`).

## Overview

Effect Stream includes constructors that lift scoped resources into a stream.
These constructors acquire resources before use and ensure they are closed when
the stream scope ends.

Two core APIs are highlighted:

- `Stream.acquireRelease` (similar to `Effect.acquireRelease`)
- `Stream.finalizer` (similar to `Effect.addFinalizer`)

Together with `Stream.ensuring`, these tools let you model cleanup and
post-finalization actions directly in stream pipelines.

## Acquire and Release with `Stream.acquireRelease`

`Stream.acquireRelease` is useful when a stream depends on a resource such as a
file handle, socket, or external client.

```ts
import { Stream, Console, Effect } from "effect"

const open = (filename: string) =>
  Effect.gen(function* () {
    yield* Console.log(`Opening ${filename}`)
    return {
      getLines: Effect.succeed(["Line 1", "Line 2", "Line 3"]),
      close: Console.log(`Closing ${filename}`)
    }
  })

const stream = Stream.acquireRelease(
  open("file.txt"),
  (file) => file.close
).pipe(Stream.flatMap((file) => file.getLines))

Effect.runPromise(Stream.runCollect(stream)).then(console.log)
```

The acquire effect opens the resource, and the release action runs when the
stream is done, ensuring proper closure.

## Finalization with `Stream.finalizer`

`Stream.finalizer` adds a cleanup effect that runs before the stream ends. This
is useful for tasks like deleting temporary files or writing final logs.

```ts
import { Stream, Console, Effect } from "effect"

const application = Stream.fromEffect(Console.log("Application Logic."))
const deleteDir = (dir: string) => Console.log(`Deleting dir: ${dir}`)

const program = application.pipe(
  Stream.concat(
    Stream.finalizer(
      deleteDir("tmp").pipe(
        Effect.andThen(Console.log("Temporary directory was deleted."))
      )
    )
  )
)
```

## Post-Finalization Work with `Stream.ensuring`

Use `Stream.ensuring` when you need additional actions after stream
finalization.

```ts
import { Stream, Console } from "effect"

const program = Stream.fromEffect(Console.log("Application Logic.")).pipe(
  Stream.concat(Stream.finalizer(Console.log("Finalizing the stream"))),
  Stream.ensuring(
    Console.log("Doing some other works after stream's finalization")
  )
)
```

Execution order in this example is:

1. Main application logic
2. Stream finalizer
3. Ensuring action (post-finalization)
