---
title: "Integrate Claude Code"
description: "Migrate Claude Code setup to LDX API using the real compatibility endpoints."
---

# Integrate Claude Code

Claude Code can connect to this project through Anthropic-compatible settings.

## Migrate Existing Claude Code Setup

If Claude Code is already installed, set these environment variables.

Linux / macOS:

```bash
export ANTHROPIC_BASE_URL=https://api.liandanxia.io
export ANTHROPIC_AUTH_TOKEN=<your LDX API Key>
export ANTHROPIC_MODEL=<model from /v1/models>
export ANTHROPIC_DEFAULT_OPUS_MODEL=<model from /v1/models>
export ANTHROPIC_DEFAULT_SONNET_MODEL=<model from /v1/models>
export ANTHROPIC_DEFAULT_HAIKU_MODEL=<model from /v1/models>
```

Windows PowerShell:

```powershell
$env:ANTHROPIC_BASE_URL="https://api.liandanxia.io"
$env:ANTHROPIC_AUTH_TOKEN="<your LDX API Key>"
$env:ANTHROPIC_MODEL="<model from /v1/models>"
$env:ANTHROPIC_DEFAULT_OPUS_MODEL="<model from /v1/models>"
$env:ANTHROPIC_DEFAULT_SONNET_MODEL="<model from /v1/models>"
$env:ANTHROPIC_DEFAULT_HAIKU_MODEL="<model from /v1/models>"
```

> Note: This project supports `POST /v1/messages` with Anthropic-compatible headers.

## Fresh Setup

1. Prepare API key (`sk-...`)
2. Verify model visibility:

```bash
curl https://api.liandanxia.io/v1/models \
  -H "Authorization: Bearer sk-your_api_key"
```

3. Apply environment variables and start Claude Code.

## Minimal Compatibility Check

```bash
curl https://api.liandanxia.io/v1/messages \
  -H "x-api-key: sk-your_api_key" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-sonnet-4-20250514",
    "max_tokens": 512,
    "messages": [{"role":"user","content":"hello"}]
  }'
```

## Common Issues

- `401`: invalid `ANTHROPIC_AUTH_TOKEN`
- `400`: invalid model or payload fields
- `429`: request rate too high
