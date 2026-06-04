---
title: "First Request"
description: "Use the real endpoints to list models, send your first chat request, stream output, and call Responses."
---

Before you start, prepare an API key. It usually starts with `sk-...`. If you are not sure which header to use, read [Authentication](/en/getting-started/authentication) first.

## 1. List Models

Start with the model list endpoint to confirm that your API key works and to see which models your account can access.

```bash
curl https://api.liandanxia.io/v1/models \
  -H "Authorization: Bearer sk-your_api_key"
```

This call confirms:

- the API key exists and is not expired
- API access is enabled for the account
- the key still has quota
- the current group has available models

To retrieve one model:

```bash
curl https://api.liandanxia.io/v1/models/qwen3.5-flash \
  -H "Authorization: Bearer sk-your_api_key"
```

## 2. Send a Chat Request

`POST /v1/chat/completions` is the recommended first inference request.

```bash
curl https://api.liandanxia.io/v1/chat/completions \
  -H "Authorization: Bearer sk-your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "Introduce yourself in one sentence"}
    ]
  }'
```

The response uses the OpenAI Chat Completions shape. The actual model set depends on `GET /v1/models` and [Models and Pricing](/en/getting-started/pricing).

## 3. Enable Streaming

Set `stream` to `true` to enable streaming output.

```bash
curl https://api.liandanxia.io/v1/chat/completions \
  -H "Authorization: Bearer sk-your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "stream": true,
    "messages": [
      {"role": "user", "content": "Give me a 3-point summary"}
    ]
  }'
```

## 4. Responses API

If your client targets the OpenAI Responses style, call:

```bash
curl https://api.liandanxia.io/v1/responses \
  -H "Authorization: Bearer sk-your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "input": "hello"
  }'
```

The project also exposes the context-compaction endpoint:

```bash
curl https://api.liandanxia.io/v1/responses/compact \
  -H "Authorization: Bearer sk-your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "input": "Compact this conversation context"
  }'
```

Whether a model is suitable for Responses or compaction depends on the available backend channel configuration. If a request returns a model or channel availability error, use a model returned by `GET /v1/models`.

## 5. Compatibility Smoke Tests

If you already have a Claude request shape, test:

```bash
curl https://api.liandanxia.io/v1/messages \
  -H "x-api-key: sk-your_api_key" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "max_tokens": 128,
    "messages": [
      {"role": "user", "content": "hello"}
    ]
  }'
```

If you already have a Gemini request shape, test model listing:

```bash
curl "https://api.liandanxia.io/v1beta/models?key=sk-your_api_key"
```

You can also use the header form:

```bash
curl https://api.liandanxia.io/v1beta/models \
  -H "x-goog-api-key: sk-your_api_key"
```

## 6. Common Errors

| Status | Common cause | What to check |
| --- | --- | --- |
| `401` | Missing, invalid, expired, or exhausted API key | Recheck the auth header and key status |
| `403` | Account disabled, API access disabled, group denied, or IP outside allowlist | Check account status, key permissions, group access, and IP restrictions |
| `400` | Request body does not match the endpoint format | Check fields such as `model`, `messages`, and `input` |
| `429` | Rate limit exceeded | Reduce concurrency or retry later |

If you already have Claude or Gemini SDKs, continue with [Compatibility Formats](/en/getting-started/compatibility).
