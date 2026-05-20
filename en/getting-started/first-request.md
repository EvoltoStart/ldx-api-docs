---
title: "First Request"
description: "Use the real endpoints to list models and send your first chat request."
---

# First Request

## 1. List models

```bash
curl https://api.liandanxia.io/v1/models \
  -H "Authorization: Bearer sk-your_api_key"
```

This is the safest first call because it confirms:

- the API key is valid
- the account still has quota
- model discovery is working

## 2. Send a chat request

```bash
curl https://api.liandanxia.io/v1/chat/completions \
  -H "Authorization: Bearer sk-your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "Introduce yourself in one sentence"}
    ]
  }'
```

In the current project, the real `POST /v1/chat/completions` endpoint supports:

- non-streaming responses
- streaming responses

## 3. Enable streaming

```bash
curl https://api.liandanxia.io/v1/chat/completions \
  -H "Authorization: Bearer sk-your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "stream": true,
    "messages": [
      {"role": "user", "content": "Give me a 3-point summary"}
    ]
  }'
```

## 4. Responses API

If your client already targets the newer OpenAI Responses style:

```bash
curl https://api.liandanxia.io/v1/responses \
  -H "Authorization: Bearer sk-your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "input": "hello"
  }'
```

## 5. What to check first on errors

- `401`
  - Invalid API key or malformed auth header
- `429`
  - Rate-limited request
- `400`
  - Request shape does not match the endpoint contract

If you already have Claude or Gemini SDKs, continue with [Compatibility Formats](/docs/en/getting-started/compatibility).
