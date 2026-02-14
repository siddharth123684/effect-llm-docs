# Execution Planning for LLMs

The `ExecutionPlan` module provides structured error handling, retries, and provider fallbacks for LLM interactions.

## Basic Execution Plan (Retries)

An execution plan can define retry logic for specific errors.

```ts
import { ExecutionPlan, Schedule, Effect, Data } from "effect"
import { OpenAiLanguageModel } from "@effect/ai-openai"

class NetworkError extends Data.TaggedError("NetworkError") {}
class ProviderOutage extends Data.TaggedError("ProviderOutage") {}

const DadJokePlan = ExecutionPlan.make({
  provide: OpenAiLanguageModel.model("gpt-4o"),
  attempts: 3,
  schedule: Schedule.exponential("100 millis", 1.5),
  while: (error) => error._tag === "NetworkError"
})

const main = Effect.gen(function*() {
  // Your logic here
}).pipe(Effect.withExecutionPlan(DadJokePlan))
```

## Fallback Models

Execution plans can chain multiple steps. If the first step fails (e.g., due to a `ProviderOutage`), the plan proceeds to the next step (e.g., trying a different provider).

```ts
import { ExecutionPlan, Schedule } from "effect"
import { OpenAiLanguageModel } from "@effect/ai-openai"
import { AnthropicLanguageModel } from "@effect/ai-anthropic"

const RobustPlan = ExecutionPlan.make({
  // Step 1: Try OpenAI GPT-4o
  provide: OpenAiLanguageModel.model("gpt-4o"),
  attempts: 3,
  schedule: Schedule.exponential("100 millis", 1.5),
  while: (error) => error._tag === "NetworkError" // Retry only on network errors
}, {
  // Step 2: Fallback to Anthropic Claude 3.5 Sonnet
  provide: AnthropicLanguageModel.model("claude-3-5-sonnet-20240620"),
  attempts: 2,
  while: (error) => error._tag === "ProviderOutage" // Retry on provider outages
})
```

## Complete Example

```ts
import { AnthropicClient, AnthropicLanguageModel } from "@effect/ai-anthropic"
import { OpenAiClient, OpenAiLanguageModel } from "@effect/ai-openai"
import { NodeHttpClient } from "@effect/platform-node"
import { Config, Effect, ExecutionPlan, Layer, Schedule } from "effect"

// Define the plan
const Plan = ExecutionPlan.make({
  provide: OpenAiLanguageModel.model("gpt-4o"),
  attempts: 3,
  schedule: Schedule.exponential("100 millis", 1.5)
}, {
  provide: AnthropicLanguageModel.model("claude-3-5-sonnet-20240620"),
  attempts: 2
})

// Define the program
const program = Effect.gen(function*() {
  // LLM interaction logic
}).pipe(Effect.withExecutionPlan(Plan))

// Configure clients
const Anthropic = AnthropicClient.layerConfig({
  apiKey: Config.redacted("ANTHROPIC_API_KEY")
}).pipe(Layer.provide(NodeHttpClient.layerUndici))

const OpenAi = OpenAiClient.layerConfig({
  apiKey: Config.redacted("OPENAI_API_KEY")
}).pipe(Layer.provide(NodeHttpClient.layerUndici))

// Run
program.pipe(
  Effect.provide([Anthropic, OpenAi]),
  Effect.runPromise
)
```
