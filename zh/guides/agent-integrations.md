---
title: "Agent 工具接入"
description: "将 Claude Code、OpenCode 等工具接入 LDX API 的入口说明。"
---

# Agent 工具接入

本页是接入入口，按工具查看请使用：

- [接入 Claude Code](/docs/zh/guides/agent-integrations-claude-code)
- [接入 OpenCode](/docs/zh/guides/agent-integrations-opencode)
- [接入 OpenClaw](/docs/zh/guides/agent-integrations-openclaw)
- [接入 Kilo Code](/docs/zh/guides/agent-integrations-kilo-code)
- [接入 GitHub Copilot](/docs/zh/guides/agent-integrations-github-copilot)
- [接入 GitHub Copilot CLI](/docs/zh/guides/agent-integrations-github-copilot-cli)
- [接入 WorkBuddy / CodeBuddy](/docs/zh/guides/agent-integrations-workbuddy-codebuddy)

## 通用接入信息

- Base URL：`https://token.liandanxia.com`
- 鉴权方式：`Authorization: Bearer sk-你的_api_key`
- 模型列表：`GET /v1/models`
- 推荐推理接口：
  - `POST /v1/chat/completions`
  - `POST /v1/responses`

## 快速连通性检查

```bash
curl https://token.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"
```

## 常见错误排查

- `401 Unauthorized`：Key 无效或认证头格式错误
- `403 Forbidden`：Key 有效但无权限
- `429 Too Many Requests`：触发限流，建议退避重试
- `400 Bad Request`：请求体不符合接口要求
