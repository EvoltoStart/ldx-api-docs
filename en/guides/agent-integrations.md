---
title: "Agent Integrations"
description: "Entry point for integrating Claude Code, OpenCode, and other agent tools with LDX API."
---

# Agent Integrations

Use the tool-specific pages:

- [Integrate Claude Code](/docs/en/guides/agent-integrations-claude-code)
- [Integrate OpenCode](/docs/en/guides/agent-integrations-opencode)

## Common Setup

- Base URL: `https://api.liandanxia.io`
- Auth: `Authorization: Bearer sk-your_api_key`
- Model discovery: `GET /v1/models`
- Recommended endpoints:
  - `POST /v1/chat/completions`
  - `POST /v1/responses`

## Connectivity Check

Validate your key and network first:

```bash
curl https://api.liandanxia.io/v1/models \
  -H "Authorization: Bearer sk-your_api_key"
```

## Common Errors

- `401 Unauthorized`
  - Invalid key or wrong auth header format.
- `403 Forbidden`
  - Key is recognized but not permitted by policy/quota.
- `429 Too Many Requests`
  - Rate limit hit; reduce concurrency and add retry backoff.
- `400 Bad Request`
  - Request payload does not match endpoint schema.

## Recommended Order

1. Verify key and model visibility with `GET /v1/models`.  
2. Run a minimal `POST /v1/chat/completions`.  
3. Then apply Claude Code / OpenCode specific configuration.  
