---
title: "Agent 接入总览"
description: "Claude Code、GitHub Copilot、OpenCode、OpenClaw 等 Agent 工具的 LDX API 接入入口页。"
---

本页是 Agent 工具接入入口。不同工具对自定义 Provider、OpenAI 兼容接口和 Anthropic 兼容接口的支持程度不同，请按工具查看详细配置与验证步骤。

## 通用配置

- 根地址：`https://api.liandanxia.com`
- OpenAI 兼容 Base URL：`https://api.liandanxia.com/v1`
- OpenAI 兼容认证头：`Authorization: Bearer sk-你的_api_key`
- Anthropic 兼容认证头：`x-api-key: sk-你的_api_key`，并携带 `anthropic-version: 2023-06-01`
- 模型发现：`GET /v1/models`
- Chat Completions：`POST /v1/chat/completions`
- Anthropic Messages 兼容：`POST /v1/messages`

## 快速连通性检查

```bash
curl https://api.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"
```

## 工具接入指南

- [接入 Claude Code](/zh/guides/agent-integrations-claude-code)
- [接入 GitHub Copilot](/zh/guides/agent-integrations-github-copilot)
- [接入 GitHub Copilot CLI](/zh/guides/agent-integrations-github-copilot-cli)
- [接入 Kilo Code](/zh/guides/agent-integrations-kilo-code)
- [接入 WorkBuddy/CodeBuddy](/zh/guides/agent-integrations-workbuddy-codebuddy)
- [接入 OpenCode](/zh/guides/agent-integrations-opencode)
- [接入 OpenClaw](/zh/guides/agent-integrations-openclaw)
- [接入 Hermes](/zh/guides/agent-integrations-hermes)

## 参考资料

- [首个请求示例](/zh/getting-started/first-request)
- [模型与价格](/zh/getting-started/pricing)
- [错误码](/zh/getting-started/error-codes)
- [Claude Code LLM gateway](https://code.claude.com/docs/en/llm-gateway)
- [GitHub Copilot CLI BYOK](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-byok-models)
