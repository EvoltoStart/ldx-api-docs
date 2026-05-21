---
title: "Integrate Claude Code"
description: "Practical Claude Code integration guide for LDX API: install, configure, verify, and troubleshoot."
---

This guide is for users who work in terminal-first development workflows and want to route Claude Code traffic through the LDX gateway.

## When to choose Claude Code

- You mostly work from CLI and local repositories
- You prefer Anthropic-style message APIs
- You want minimal migration effort to LDX gateway routing

## Prerequisites

- A valid API key: `sk-...`
- Network access to `https://api.liandanxia.io`
- Node.js installed (18+ recommended)
- `curl` installed

## Step 1: Install Claude Code

Recommended:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Alternative:

```bash
npm install -g @anthropic-ai/claude-code
```

Verify:

```bash
claude --version
```

## Step 2: Configure LDX routing

### Linux / macOS

```bash
export ANTHROPIC_BASE_URL=https://api.liandanxia.io
export ANTHROPIC_AUTH_TOKEN=<your LDX API Key>
export ANTHROPIC_MODEL=<model from /v1/models>
```

### Windows PowerShell

```powershell
$env:ANTHROPIC_BASE_URL="https://api.liandanxia.io"
$env:ANTHROPIC_AUTH_TOKEN="<your LDX API Key>"
$env:ANTHROPIC_MODEL="<model from /v1/models>"
```

Tip: verify with temporary session variables first, then persist in shell startup files.

## Step 3: Verify end-to-end

Check model discovery:

```bash
curl https://api.liandanxia.io/v1/models \
  -H "Authorization: Bearer sk-your_api_key"
```

Check Anthropic-compatible messages:

```bash
curl https://api.liandanxia.io/v1/messages \
  -H "x-api-key: sk-your_api_key" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model":"claude-sonnet-4-20250514",
    "max_tokens":256,
    "messages":[{"role":"user","content":"hello"}]
  }'
```

Run Claude Code:

```bash
claude
```

### Success criteria

- `/v1/models` returns model list
- `/v1/messages` returns a valid response
- First prompt in `claude` completes successfully

## Troubleshooting

- `401`: invalid or expired key
- `404`: incorrect `ANTHROPIC_BASE_URL`
- `400`: invalid model or payload fields
- `429`: rate limit exceeded

Recommended check order: URL -> key -> model name -> request payload.

## Security and best practices

- Never commit real keys to repository files
- Inject secrets via environment or secret managers
- Use separate default models for speed vs quality workflows

## Sources

- Claude Code Getting Started: https://docs.anthropic.com/en/docs/claude-code/getting-started
- Claude Code Env Vars: https://code.claude.com/docs/en/env-vars
- Anthropic Messages API: https://docs.anthropic.com/en/api/messages
