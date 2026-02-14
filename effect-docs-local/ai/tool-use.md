# Tool Use in Effect AI

Effect provides a type-safe `Toolkit` abstraction for LLMs to invoke external functions (Tool Calling).

## 1. Defining a Tool

Use `Tool.make` with `Schema` to describe inputs and outputs.

```ts
import { Tool } from "@effect/ai"
import { Schema } from "effect"

const GetDadJoke = Tool.make("GetDadJoke", {
  description: "Get a hilarious dad joke from the ICanHazDadJoke API",
  success: Schema.String,
  failure: Schema.Never,
  parameters: {
    searchTerm: Schema.String.annotations({
      description: "The search term to use to find dad jokes"
    })
  }
})
```

## 2. Creating a Toolkit

Group tools into a `Toolkit`.

```ts
import { Toolkit } from "@effect/ai"

const DadJokeTools = Toolkit.make(GetDadJoke)
```

## 3. Implementing Tool Logic

Use `.toLayer` to provide the implementation for each tool. This allows you to access other Effect services (like `HttpClient`).

```ts
import { Effect, Layer } from "effect"

// Assume ICanHazDadJoke is a service defined elsewhere
const DadJokeToolHandlers = DadJokeTools.toLayer(
  Effect.gen(function*() {
    const icanhazdadjoke = yield* ICanHazDadJoke
    return {
      GetDadJoke: ({ searchTerm }) => icanhazdadjoke.search(searchTerm)
    }
  })
).pipe(Layer.provide(ICanHazDadJoke.Default))
```

## 4. Providing Tools to the Model

Pass the `toolkit` to `LanguageModel.generateText`.

```ts
import { LanguageModel } from "@effect/ai"

const program = LanguageModel.generateText({
  prompt: "Generate a dad joke about pirates",
  toolkit: DadJokeTools
})
```

## 5. Running the Program

Provide both the `OpenAi` (client) layer and the `ToolHandlers` layer to the program.

```ts
import { Effect } from "effect"
import { OpenAiLanguageModel } from "@effect/ai-openai"

// ... (Client configuration)

program.pipe(
  // Provide the model implementation
  Effect.provide(OpenAiLanguageModel.model("gpt-4o")),
  // Provide the tool handlers
  Effect.provide(DadJokeToolHandlers),
  // Provide the OpenAI client
  Effect.provide(OpenAi),
  Effect.runPromise
)
```

## Benefits
- **Type Safety**: Inputs and outputs are validated by Schema.
- **Effect Native**: Implementations can use full Effect capabilities (concurrency, error handling, services).
- **Separation of Concerns**: Tool definition (interface) is separate from implementation (logic).
