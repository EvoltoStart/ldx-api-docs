---
title: "Integrate OpenClaw"
description: "Practical OpenClaw integration guide for LDX API: install, provider registration, verification, and troubleshooting."
---

This guide is for users who manage multiple providers via OpenClaw and want a clean default-model workflow through LDX.

## When to choose OpenClaw

- You need explicit multi-provider model management
- You want CLI visibility into model status
- You want stable default-model policy in tool config

## Prerequisites

- A valid API key: `sk-...`
- Network access to `https://token.liandanxia.com`
- Node.js installed (18+ recommended)
- `curl` installed

## Step 1: Install OpenClaw

Official installer:

```bash
curl -fsSL https://openclaw.ai/install | bash
```

Alternative:

```bash
npm install -g openclaw@latest
```

Verify:

```bash
openclaw --version
```

## Step 2: Register LDX provider

```bash
openclaw config set models.providers.ldx.api "openai-completions"
openclaw config set models.providers.ldx.baseUrl "https://token.liandanxia.com/v1"
openclaw config set models.providers.ldx.apiKey "sk-your_api_key"
openclaw config set models.providers.ldx.models '[{"id":"gpt-4o-mini"}]'
openclaw config set agents.defaults.model.primary "ldx/gpt-4o-mini"
```

Optional check:

```bash
openclaw config list
```

## Step 3: Verify end-to-end

```bash
openclaw models list
openclaw models status
openclaw run "hello"
```

### Success criteria

- target model appears in `models list`
- `models status` reports no critical issue
- `run "hello"` returns valid output

## Troubleshooting

- `401`: invalid `apiKey`
- `404`: incorrect `baseUrl` (or missing `/v1`)
- `400`: invalid model mapping or payload
- `429`: rate limit exceeded

Recommended check order: baseUrl -> api type -> model mapping -> key.

## Security and best practices

- Use a short stable provider alias (for example `ldx`)
- Confirm model list before setting default model
- Keep secrets out of committed config

## Sources

- OpenClaw install: https://docs.openclaw.ai/getting-started/installing-openclaw
- OpenClaw model providers: https://docs.openclaw.ai/concepts/model-providers
- OpenClaw OpenAI provider: https://docs.openclaw.ai/providers/openai
- OpenClaw configuration: https://docs.openclaw.ai/configuration
