---
title: "快速开始"
description: "用真实存在的接口快速接入 LDX 模型调用 API。"
---

# 快速开始

如果你是来接入模型调用，先看这几部分：

1. `核心 API`
   - 统一调用入口。
   - 主要包括 `/v1/models`、`/v1/chat/completions`、`/v1/responses`、`/v1/embeddings`、`/v1/images/generations`、`/v1/audio/*`、`/v1/videos*`。

2. `兼容 API`
   - 用于 Claude、Gemini、Kling、即梦等兼容或厂商特定格式。
   - 主要包括 `/v1/messages`、`/v1beta/models/{model}:generateContent`、`/kling/*`、`/jimeng`。

## Base URL

- 模型调用与兼容接口：`https://api.liandanxia.io`

## 推荐接入顺序

1. 准备好 API Key（`sk-...`）
2. 用 `GET /v1/models` 确认可用模型
3. 用 `POST /v1/chat/completions` 发出第一条请求
4. 如果你接的是 Claude / Gemini 等现有 SDK，再看 `兼容 API`

## 最短调用链路

```bash
curl https://api.liandanxia.io/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "messages": [
      {"role": "user", "content": "hello"}
    ]
  }'
```

下一步先看[认证](/docs/zh/getting-started/authentication)和[首个请求](/docs/zh/getting-started/first-request)。
