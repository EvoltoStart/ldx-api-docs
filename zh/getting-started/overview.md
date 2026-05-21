---
title: "API 概览"
description: "API Reference 入口说明：核心能力、兼容能力与接入顺序。"
---

如果你是来接入模型调用，建议先看这几部分：

1. `核心 API`
   - 统一调用入口
   - 主要包含 `/v1/models`、`/v1/chat/completions`、`/v1/responses`、`/v1/embeddings`、`/v1/images/generations`、`/v1/audio/*`、`/v1/videos*`

2. `兼容 API`
   - 用于 Claude、Gemini、Kling、即梦等兼容或厂商特定格式
   - 主要包含 `/v1/messages`、`/v1beta/models/{model}:generateContent`、`/kling/*`、`/jimeng`

## Base URL

- 中文站（国内）：`https://token.liandanxia.com`

## OpenAPI 入口

- 核心 API：`/zh/openapi/core-api.mintlify.json`
- 兼容 API：`/zh/openapi/compatibility-api.mintlify.json`

## 推荐接入顺序

1. 准备 API Key（`sk-...`）
2. 调用 `GET /v1/models` 确认可用模型
3. 调用 `POST /v1/chat/completions` 发出第一条请求
4. 如果你使用的是 Claude / Gemini 现成 SDK，再看兼容 API

## 最小可用请求

```bash
curl https://token.liandanxia.com/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "messages": [
      {"role": "user", "content": "hello"}
    ]
  }'
```

下一步建议先看 [认证](/zh/getting-started/authentication) 和 [首个请求](/zh/getting-started/first-request)。
