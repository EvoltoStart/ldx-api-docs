---
title: "首个请求"
description: "用真实接口完成模型列表查询、第一条聊天请求、流式输出和 Responses 请求。"
---

开始前先准备一个 API Key，格式通常是 `sk-...`。如果还不确定认证头怎么写，先看 [认证](/zh/getting-started/authentication)。

## 1. 查询模型

先调用模型列表接口，确认 API Key 可用，并查看当前账号能访问哪些模型。

```bash
curl https://token.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"
```

这一步可以确认：

- API Key 存在且未过期。
- 账号没有被禁用 API 访问。
- Key 仍有可用额度。
- 当前分组下有可用模型。

如果你想查询单个模型：

```bash
curl https://token.liandanxia.com/v1/models/qwen3.5-flash \
  -H "Authorization: Bearer sk-你的_api_key"
```

## 2. 发起聊天请求

`POST /v1/chat/completions` 是推荐的第一条推理请求。

```bash
curl https://token.liandanxia.com/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "用一句话介绍你自己"}
    ]
  }'
```

请求成功后会返回 OpenAI Chat Completions 风格的响应。实际可用模型请以 `GET /v1/models` 返回结果和 [模型与价格](/zh/getting-started/pricing) 为准。

## 3. 打开流式输出

把 `stream` 设为 `true` 即可启用流式响应。

```bash
curl https://token.liandanxia.com/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "stream": true,
    "messages": [
      {"role": "user", "content": "给我写一个三点总结"}
    ]
  }'
```

## 4. Responses API

如果你的客户端使用 OpenAI Responses 风格，可以调用：

```bash
curl https://token.liandanxia.com/v1/responses \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "input": "hello"
  }'
```

项目也暴露了压缩上下文入口：

```bash
curl https://token.liandanxia.com/v1/responses/compact \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "input": "请压缩这段对话上下文"
  }'
```

不同模型是否适合 Responses 或压缩入口，取决于后端可用渠道配置；如果返回模型或渠道不可用，请换用 `GET /v1/models` 中可见的模型。

## 5. 兼容格式首测

如果你已经有 Claude 请求格式，可以先测：

```bash
curl https://token.liandanxia.com/v1/messages \
  -H "x-api-key: sk-你的_api_key" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "max_tokens": 128,
    "messages": [
      {"role": "user", "content": "hello"}
    ]
  }'
```

如果你已经有 Gemini 请求格式，可以先测模型列表：

```bash
curl "https://token.liandanxia.com/v1beta/models?key=sk-你的_api_key"
```

也可以使用请求头：

```bash
curl https://token.liandanxia.com/v1beta/models \
  -H "x-goog-api-key: sk-你的_api_key"
```

## 6. 常见报错

| 状态码 | 常见原因 | 处理方式 |
| --- | --- | --- |
| `401` | API Key 缺失、无效、过期或额度耗尽 | 重新检查认证头和 Key 状态 |
| `403` | 账号禁用、API 访问禁用、分组无权限、IP 不在白名单 | 检查账号、Key 权限、分组和 IP 限制 |
| `400` | 请求体字段不符合目标接口格式 | 检查 `model`、`messages`、`input` 等字段 |
| `429` | 请求频率超限 | 降低并发或稍后重试 |

如果你已有 Claude / Gemini 的现成 SDK，下一步看 [兼容格式](/zh/getting-started/compatibility)。
