---
title: "接入 Kilo Code"
description: "使用 Kilo 的自定义模型能力接入 LDX API。"
---

# 接入 Kilo Code

Kilo 支持自定义模型提供商，可按 OpenAI 兼容模式接入。

## 配置参数

- Provider Type：OpenAI-compatible
- API Base URL：`https://token.liandanxia.com/v1`
- API Key：`sk-你的_api_key`
- Model：`GET /v1/models` 返回的可用模型

## 验证请求

```bash
curl https://token.liandanxia.com/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-4o-mini","messages":[{"role":"user","content":"hello"}]}'
```
