---
title: "Integrate WorkBuddy / CodeBuddy"
description: "Use OpenAI-compatible provider settings to connect WorkBuddy/CodeBuddy to LDX API."
---

# Integrate WorkBuddy / CodeBuddy

WorkBuddy and CodeBuddy generally support custom OpenAI-compatible providers.

## Core Settings

- Provider: OpenAI-compatible
- Base URL: `https://api.liandanxia.io/v1`
- API Key: `sk-your_api_key`
- Model: from `GET /v1/models`

## Steps

1. Open Models/Providers settings.
2. Add a custom provider.
3. Fill Base URL, API key, and model.
4. Save and set as default.

## Minimal Check

```bash
curl https://api.liandanxia.io/v1/chat/completions \
  -H "Authorization: Bearer sk-your_api_key" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-4o-mini","messages":[{"role":"user","content":"hello"}]}'
```
