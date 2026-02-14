---
title: Getting Started with Effect AI
description: Installing @effect/ai, configuring providers, and using LanguageModel.generateText with provider-agnostic APIs.
---

# Getting Started with Effect AI

The `@effect/ai` ecosystem provides a provider-agnostic interface for interacting with Large Language Models (LLMs). It separates the *description* of an AI interaction from its *execution*.

## Core Packages

- **`@effect/ai`**: Core abstractions and generic services (`LanguageModel`, `EmbeddingsModel`, `Tool`). Always required.

### Supported Providers
- **`@effect/ai-openai`**: Implementations backed by OpenAI.
- **`@effect/ai-anthropic`**: Implementations backed by Anthropic.
- **`@effect/ai-amazon-bedrock`**: Implementations backed by Amazon Bedrock.
- **`@effect/ai-google`**: Implementations backed by Google Gemini.

## Installation

Install the core package and at least one provider integration (e.g., OpenAI):

```sh
npm install @effect/ai @effect/ai-openai effect
```

## Basic Interaction

Use `LanguageModel.generateText` to describe an LLM interaction. This creates an Effect that *requires* a `LanguageModel` service.

```ts
import { LanguageModel } from "@effect/ai"
import { Effect } from "effect"

// Effect<GenerateTextResponse<{}>, AiError, LanguageModel>
const generateDadJoke = Effect.gen(function*() {
  const response = yield* LanguageModel.generateText({
    prompt: "Generate a dad joke"
  })
  console.log(response.text)
  return response
})
```

## Selecting a Provider (`Model`)

A `Model` represents a concrete provider implementation (e.g., GPT-4 via OpenAI) that satisfies the `LanguageModel` requirement.

```ts
import { OpenAiLanguageModel } from "@effect/ai-openai"
import { LanguageModel } from "@effect/ai"
import { Effect } from "effect"

const generateDadJoke = Effect.gen(function*() {
  const response = yield* LanguageModel.generateText({
    prompt: "Generate a dad joke"
  })
  console.log(response.text)
  return response
})

// Create a Model for GPT-4o
const Gpt4o = OpenAiLanguageModel.model("gpt-4o")

// Provide the Model to satisfy the LanguageModel requirement
// The resulting Effect now requires an OpenAiClient
const main = generateDadJoke.pipe(
  Effect.provide(Gpt4o)
)
```

## Configuring the Client

Provider integrations require a client configuration (API keys, etc.) and an HTTP client.

```ts
import { OpenAiClient, OpenAiLanguageModel } from "@effect/ai-openai"
import { LanguageModel } from "@effect/ai"
import { NodeHttpClient } from "@effect/platform-node"
import { Config, Effect, Layer } from "effect"

const generateDadJoke = Effect.gen(function*() {
  const response = yield* LanguageModel.generateText({
    prompt: "Generate a dad joke"
  })
  console.log(response.text)
  return response
})

const Gpt4o = OpenAiLanguageModel.model("gpt-4o")

// Create a Layer for the OpenAiClient using configuration
const OpenAi = OpenAiClient.layerConfig({
  apiKey: Config.redacted("OPENAI_API_KEY")
})

// Provide the HTTP client implementation (NodeHttpClient)
const OpenAiWithHttp = Layer.provide(OpenAi, NodeHttpClient.layerUndici)

const main = generateDadJoke.pipe(
  Effect.provide(Gpt4o)
)

// Run the program with all dependencies provided
main.pipe(
  Effect.provide(OpenAiWithHttp),
  Effect.runPromise
)
```

## Mixing Providers

You can mix and match models within the same program. Effect's type system tracks which clients are required.

```ts
import { AnthropicLanguageModel } from "@effect/ai-anthropic"
import { OpenAiLanguageModel } from "@effect/ai-openai"
import { LanguageModel } from "@effect/ai"
import { Effect } from "effect"

const generateDadJoke = Effect.gen(function*() {
  return yield* LanguageModel.generateText({ prompt: "Generate a dad joke" })
})

const Gpt4o = OpenAiLanguageModel.model("gpt-4o")
const Claude37 = AnthropicLanguageModel.model("claude-3-7-sonnet-latest")

const main = Effect.gen(function*() {
  // Uses GPT-4o (provided at the end)
  const res1 = yield* generateDadJoke
  
  // Explicitly uses Claude 3.7
  const res2 = yield* Effect.provide(generateDadJoke, Claude37)
}).pipe(Effect.provide(Gpt4o))
```
