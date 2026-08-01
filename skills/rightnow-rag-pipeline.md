---
name: rag-pipeline
description: Retrieval-augmented generation on RunInfra — embed a query, rerank candidates, then generate a grounded answer.
api: RunInfra Inference API
base_url: https://api.runinfra.ai/v1
operations:
  - createEmbedding
  - createRerank
  - createChatCompletion
---

# RAG pipeline on RunInfra

Grounded question-answering over your own corpus using three RunInfra operations.
All calls use `Authorization: Bearer <rp_live_...>` and the base URL
`https://api.runinfra.ai/v1`.

## Steps

1. **Embed the query** — `createEmbedding` (`POST /embeddings`). Send the user
   question with your deployed embedding `model` id. Use the returned vector to
   retrieve candidate passages from your vector store.
2. **Rerank candidates** — `createRerank` (`POST /rerank`). Pass the query plus the
   retrieved documents and a reranker `model`; keep the top-scoring passages.
3. **Generate the answer** — `createChatCompletion` (`POST /chat/completions`).
   Put the reranked passages in the system/context messages, ask the question, and
   read the grounded answer. Set `stream: true` for token-by-token SSE output.

## Rules

- Set `Idempotency-Key` on the embedding and chat calls if you auto-retry; a reused
  key with a different body returns **409**, a non-replayable response returns **422**.
- Honor `Retry-After` on **429**/**503**; the limiter is a per-key leaky bucket.
- On **402** top up credits; on **404** the `model` is not deployed — check `listModels`.
- Errors are OpenAI-style `{"error":{"message","type","code","param"}}`; capture
  `X-Request-Id` on 5xx for support.
