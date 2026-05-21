---
title: "Agent 工具接入"
description: "Claude Code、OpenCode、OpenClaw 的 LDX API 接入入口页。"
---

本页是接入入口，请按工具查看详细配置与验证步骤。

## 通用配置

- Base URL：`https://token.liandanxia.com`
- 认证头：`Authorization: Bearer sk-你的_api_key`
- 模型发现：`GET /v1/models`

## 快速连通性检查

```bash
curl https://token.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"
```

## 工具接入指南

- [接入 Claude Code](/zh/guides/agent-integrations-claude-code)
- [接入 OpenCode](/zh/guides/agent-integrations-opencode)
- [接入 OpenClaw](/zh/guides/agent-integrations-openclaw)
