---
title: "接入 OpenClaw"
description: "将 OpenClaw 配置为使用 LDX API 的兼容端点。"
---

# 接入 OpenClaw

推荐使用 OpenAI 兼容模式。

## OpenAI 兼容配置

- Base URL：`https://token.liandanxia.com/v1`
- API Key：`sk-你的_api_key`
- Model：`GET /v1/models` 可见模型

## Anthropic 兼容配置（可选）

- Base URL：`https://token.liandanxia.com`
- Auth Token：`sk-你的_api_key`
- Endpoint：`POST /v1/messages`
