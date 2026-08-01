---
name: streaming-chat
description: Stream a RunInfra chat completion token-by-token over SSE, with model discovery and safe retries.
api: RunInfra Inference API
base_url: https://api.runinfra.ai/v1
operations:
  - listModels
  - createChatCompletion
---

# Streaming chat on RunInfra

OpenAI-compatible streaming chat against a verified RunInfra deployment.

## Steps

1. **Discover a model** — `listModels` (`GET /models`) to confirm a verified deployed
   `model` id in your workspace. Optionally `retrieveModel` (`GET /models/{model}`)
   for one model.
2. **Open a streaming completion** — `createChatCompletion` (`POST /chat/completions`)
   with `stream: true`. Read the response as server-sent events; each
   `ChatCompletionChunk` carries an incremental `delta`. The SSE wire format matches
   OpenAI, so the `openai` clients or `@runinfra/sdk` work unchanged.
3. **Assemble** the streamed deltas into the final message; read `usage` from the
   terminating event for token accounting.

## Rules

- Use a **workspace-scoped** key (recommended) and select the model via the `model`
  field; pipeline-scoped keys are legacy.
- On **429**/**503** respect `Retry-After`; watch `X-RateLimit-Remaining`.
- On **504** switch to streaming (already on) or lower `max_tokens`.
- Set `Idempotency-Key` for replay-safe retries; different body + same key → **409**.
