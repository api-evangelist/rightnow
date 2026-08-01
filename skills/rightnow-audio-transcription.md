---
name: audio-transcription
description: Transcribe an audio file to text on RunInfra using the OpenAI-compatible transcription endpoint.
api: RunInfra Inference API
base_url: https://api.runinfra.ai/v1
operations:
  - listModels
  - createTranscription
---

# Audio transcription on RunInfra

Speech-to-text against a deployed open Whisper (or compatible) model.

## Steps

1. **Confirm a speech-to-text model** — `listModels` (`GET /models`) and pick a
   verified deployed transcription `model` id.
2. **Upload and transcribe** — `createTranscription` (`POST /audio/transcriptions`).
   Send the audio as a multipart upload with the `model` field. The endpoint is
   OpenAI-shaped, so `client.audio.transcriptions.create(...)` works with the
   RunInfra base URL.
3. **Read** the returned `Transcription` text.

## Rules

- This is a charge-bearing, multipart call — prefer an explicit `Idempotency-Key`
  and retry policy over broad automatic retries.
- On **400** `unsupported_backend_modality` the deployment cannot serve audio —
  choose a compatible backend in the dashboard.
- On **404** the model is not deployed; on **402** top up credits.
- Errors are OpenAI-style `{"error":{...}}`; capture `X-Request-Id` on 5xx.
