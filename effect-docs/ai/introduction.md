---
title: Introduction
description: Effect AI provides provider-agnostic LLM abstractions with testability, concurrency, and observability.
---

# Introduction

Source: extracted from `llms-full.txt` (`Introduction to Effect AI`).

## Overview

Effect AI packages are currently experimental (alpha). They provide a provider-agnostic way to model LLM workflows: you describe what your program should do with an LLM first, then choose and provide the concrete provider implementation at runtime.

This keeps application logic focused on behavior rather than provider-specific APIs and wiring.

## Why Effect for AI?

LLM workflows usually need more than a single API call (streaming, retries, fallback, cancellation, and timeout handling). Effect provides composable primitives to model these concerns safely.

Key benefits:

- Provider-agnostic architecture (swap providers without rewriting business logic)
- Testability (replace real providers with mocks/stubs)
- Structured concurrency (safe parallel calls, interruption, and races)
- Observability (logging, tracing, and metrics)

## Core Concepts

Effect AI models LLM capabilities as services that can be injected like any other Effect dependency. Common capabilities include:

- Text generation
- Embedding generation
- Tool calling / structured outputs
- Streaming responses

This lets you keep AI logic declarative and defer execution details to layers.

## Package Layout

- `@effect/ai`: core abstractions and helper utilities for provider-agnostic AI programs
- `@effect/ai-openai`: OpenAI-backed implementations (for example `LanguageModel`, `EmbeddingsModel`)
- `@effect/ai-anthropic`: Anthropic-backed implementations
- `@effect/ai-amazon-bedrock`: Amazon Bedrock-backed implementations
- `@effect/ai-google`: Google Gemini-backed implementations
