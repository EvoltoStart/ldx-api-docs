---
title: "Integrate OpenCode"
description: "Practical OpenCode integration guide for LDX API: install, provider config, verification, and troubleshooting."
---

This guide is for users who prefer project-level model configuration via `opencode.json` and OpenAI-compatible routing through LDX.

## When to choose OpenCode

- You want model config tracked per project
- You need different defaults across repositories
- You want a stable OpenAI-compatible integration path

## Prerequisites

- A valid API key: `sk-...`
- Network access to `https://token.liandanxia.com`
- Node.js installed (18+ recommended)
- `curl` installed

## Step 1: Install OpenCode

Official installer:

```bash
curl -fsSL https://opencode.ai/install | bash
```

Alternative:

```bash
npm i -g opencode-ai@latest
```

Verify:

```bash
opencode --version
```

## Step 2: Configure provider in project

Create `opencode.json` in project root:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ldx": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LDX",
      "models": {
        "gpt-4o-mini": {}
      },
      "options": {
        "baseURL": "https://token.liandanxia.com/v1",
        "apiKey": "sk-your_api_key"
      }
    }
  },
  "model": "ldx/gpt-4o-mini"
}
```

Key checks:

- `baseURL` must include `/v1`
- `model` must match provider model mapping
- Provider alias (`ldx`) must be consistent across config

## Step 3: Verify end-to-end

API verification:

```bash
curl https://token.liandanxia.com/v1/chat/completions \
  -H "Authorization: Bearer sk-your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model":"gpt-4o-mini",
    "messages":[{"role":"user","content":"hello"}]
  }'
```

Run OpenCode:

```bash
opencode
```

### Success criteria

- `chat/completions` returns valid JSON
- OpenCode resolves `ldx/...` model successfully
- First interactive request completes

## Troubleshooting

- `401`: invalid API key
- `404`: wrong `baseURL` (often missing `/v1`)
- `400`: invalid model or request schema
- `429`: rate limit exceeded

Recommended check order: JSON syntax -> baseURL -> model mapping -> key/permissions.

## Security and best practices

- Keep config shape in repo, keep real key out of repo
- Inject production secrets from environment/secret manager
- Use separate provider aliases for different environments if needed

## Sources

- OpenCode docs: https://opencode.ai/docs
- OpenCode Providers: https://opencode.ai/docs/providers/
- OpenCode Config: https://opencode.ai/docs/config/
