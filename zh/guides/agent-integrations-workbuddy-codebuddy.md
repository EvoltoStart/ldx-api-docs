---
title: "接入 WorkBuddy / CodeBuddy"
description: "在 WorkBuddy/CodeBuddy 中按 OpenAI 兼容方式接入 LDX API。"
---

# 接入 WorkBuddy / CodeBuddy

## 配置参数

- Provider：OpenAI-compatible
- Base URL：`https://token.liandanxia.com/v1`
- API Key：`sk-你的_api_key`
- Model：`GET /v1/models` 可见模型

## 最小验证

```bash
curl https://token.liandanxia.com/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-4o-mini","messages":[{"role":"user","content":"hello"}]}'
```
