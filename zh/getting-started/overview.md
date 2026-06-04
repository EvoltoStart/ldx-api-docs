---
title: "快速开始"
description: "从准备 API Key 到完成第一条模型调用的推荐接入顺序。"
---

如果你是第一次接入模型调用，按本页顺序完成认证、模型查询和第一条请求即可。

## 接入路径

建议先看这几部分：

1. `核心 API`
   - 统一调用入口
   - 主要包含 `/v1/models`、`/v1/chat/completions`、`/v1/responses`、`/v1/embeddings`、`/v1/images/generations`、`/v1/audio/*`、`/v1/videos*`

2. `兼容 API`
   - 用于 Claude、Gemini、Kling、即梦等兼容或厂商特定格式
   - 主要包含 `/v1/messages`、`/v1beta/models/{model}:generateContent`、`/kling/*`、`/jimeng`

## Base URL

- 中文站（国内）：`https://api.liandanxia.com`
- 
## 推荐接入顺序

1. 准备 API Key（`sk-...`）
2. 调用 `GET /v1/models` 确认可用模型
3. 调用 `POST /v1/chat/completions` 发出第一条请求
4. 如果你使用的是 Claude / Gemini 现成 SDK，再看 [兼容格式](/zh/getting-started/compatibility)

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

下一步建议先看 [认证](/zh/getting-started/authentication)，然后按 [首个请求](/zh/getting-started/first-request) 完成调用。
