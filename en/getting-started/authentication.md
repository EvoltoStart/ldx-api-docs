---
title: "Authentication"
description: "API Reference entry guide: auth patterns for public endpoints."
---

# Authentication

This page only covers authentication used by the public API Reference.

## 1. Standard auth (Core API)

Applies to:

- `/v1/models`
- `/v1/chat/completions`
- `/v1/responses`
- `/v1/embeddings`
- `/v1/images/*`
- `/v1/audio/*`
- `/v1/videos*`

Header:

```http
Authorization: Bearer sk-your_api_key
```

## 2. Compatibility auth headers (Compatibility API)

Use headers based on request format:

- Claude-compatible:
  - `x-api-key: sk-your_api_key`
  - `anthropic-version: 2023-06-01`
- Gemini-compatible:
  - `x-goog-api-key: sk-your_api_key`
- Gemini query-string form (model list):
  - `GET /v1/models?key=sk-your_api_key`

## 3. Common auth errors

- `401`: invalid API key or malformed auth header
- `403`: key exists but lacks permission for the target capability
- `429`: request rate is limited

Next: [First Request](/docs/en/getting-started/first-request).

