---
title: "Integrate OpenClaw"
description: "Configure OpenClaw to use LDX API with compatible endpoints."
---

# Integrate OpenClaw

For OpenClaw, prefer OpenAI-compatible provider mode.

## OpenAI-Compatible Mode (Recommended)

- Base URL: `https://api.liandanxia.io/v1`
- API Key: `sk-your_api_key`
- Model: from `GET /v1/models`

## Anthropic-Compatible Mode (Optional)

- Base URL: `https://api.liandanxia.io`
- Auth Token: `sk-your_api_key`
- Endpoint: `POST /v1/messages`

## Quick Checks

OpenAI-compatible:

```bash
curl https://api.liandanxia.io/v1/chat/completions \
  -H "Authorization: Bearer sk-your_api_key" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-4o-mini","messages":[{"role":"user","content":"hello"}]}'
```

Anthropic-compatible:

```bash
curl https://api.liandanxia.io/v1/messages \
  -H "x-api-key: sk-your_api_key" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"model":"claude-sonnet-4-20250514","max_tokens":256,"messages":[{"role":"user","content":"hello"}]}'
```
