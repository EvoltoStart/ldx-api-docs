---
title: "Integrate OpenCode"
description: "Configure OpenCode to use LDX API through OpenAI-compatible endpoints."
---

# Integrate OpenCode

OpenCode can connect through the OpenAI-compatible API surface.

## 1) Required Values

- Base URL: `https://api.liandanxia.io/v1`
- API Key: `sk-your_api_key`
- Model: from `GET /v1/models`

## 2) Environment Variables

Linux / macOS:

```bash
export OPENAI_BASE_URL=https://api.liandanxia.io/v1
export OPENAI_API_KEY=<your LDX API Key>
export OPENAI_MODEL=<model from /v1/models>
```

Windows PowerShell:

```powershell
$env:OPENAI_BASE_URL="https://api.liandanxia.io/v1"
$env:OPENAI_API_KEY="<your LDX API Key>"
$env:OPENAI_MODEL="<model from /v1/models>"
```

## 3) Config File Mode (if supported)

Set provider values as:

- `base_url`: `https://api.liandanxia.io/v1`
- `api_key`: `sk-...`
- `model`: from `/v1/models`

## 4) Minimal Check

```bash
curl https://api.liandanxia.io/v1/chat/completions \
  -H "Authorization: Bearer sk-your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "messages": [{"role":"user","content":"hello"}]
  }'
```

## 5) Tool Calling Example

```bash
curl https://api.liandanxia.io/v1/chat/completions \
  -H "Authorization: Bearer sk-your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "messages": [{"role":"user","content":"Check Shanghai weather"}],
    "tools": [
      {
        "type":"function",
        "function":{
          "name":"get_weather",
          "description":"Query weather",
          "parameters":{
            "type":"object",
            "properties":{"city":{"type":"string"}},
            "required":["city"]
          }
        }
      }
    ]
  }'
```

## 6) Common Issues

- `401`: wrong key or missing Bearer header
- `404`: wrong base URL composition by tool
- `429`: rate limit; add backoff and lower concurrency
