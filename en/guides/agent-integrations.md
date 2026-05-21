---
title: "Agent Integrations"
description: "Entry page for Claude Code, OpenCode, and OpenClaw integration with LDX API."
---

Use this page as the integration entry point.

## Common Configuration

- Base URL: `https://token.liandanxia.com`
- Auth header: `Authorization: Bearer sk-your_api_key`
- Model discovery: `GET /v1/models`

## Quick Connectivity Check

```bash
curl https://token.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-your_api_key"
```

## Tool Guides

- [Integrate Claude Code](/en/guides/agent-integrations-claude-code)
- [Integrate OpenCode](/en/guides/agent-integrations-opencode)
- [Integrate OpenClaw](/en/guides/agent-integrations-openclaw)
