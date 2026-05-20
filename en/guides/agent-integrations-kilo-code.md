---
title: "Integrate Kilo Code"
description: "Connect Kilo custom models to LDX API using OpenAI-compatible endpoints."
---

# Integrate Kilo Code

Kilo supports custom model providers and can connect in OpenAI-compatible mode.

## Required Values

- Base URL: `https://api.liandanxia.io/v1`
- API Key: `sk-your_api_key`
- Model: from `GET /v1/models`

## Setup

In Kilo `Custom Models` or `/connect` flow:

- Provider Type: OpenAI-compatible
- API Base URL: `https://api.liandanxia.io/v1`
- API Key: `sk-...`
- Model: available model (for example `gpt-4o-mini`)

## Verify

```bash
curl https://api.liandanxia.io/v1/chat/completions \
  -H "Authorization: Bearer sk-your_api_key" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-4o-mini","messages":[{"role":"user","content":"hello"}]}'
```
