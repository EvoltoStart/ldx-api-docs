---
title: "认证"
description: "对外 API 的认证方式、兼容请求头和常见认证失败原因。"
---

所有对外模型调用最终都会走 API Key 校验。推荐统一使用标准 Bearer 认证；只有在接入 Claude、Gemini 等现成 SDK 时，再使用对应兼容请求头。

## 标准认证

标准认证适用于核心 `/v1/*` 接口，例如：

- `GET /v1/models`
- `GET /v1/models/{model}`
- `POST /v1/chat/completions`
- `POST /v1/completions`
- `POST /v1/responses`
- `POST /v1/responses/compact`
- `POST /v1/embeddings`
- `POST /v1/images/generations`
- `POST /v1/images/edits`
- `POST /v1/audio/transcriptions`
- `POST /v1/audio/translations`
- `POST /v1/audio/speech`
- `POST /v1/audio/generations`
- `POST /v1/rerank`

请求头：

```http
Authorization: Bearer sk-你的_api_key
```

`Bearer` 大小写不敏感，但建议按上面的格式书写。API Key 通常以 `sk-` 开头。

## Claude 兼容认证

Claude Messages 兼容接口使用：

- `POST /v1/messages`

请求头：

```http
x-api-key: sk-你的_api_key
anthropic-version: 2023-06-01
```

`GET /v1/models` 和 `GET /v1/models/{model}` 在同时带上 `x-api-key` 与 `anthropic-version` 时，会按 Anthropic / Claude 兼容格式返回。

## Gemini 兼容认证

Gemini 兼容接口使用：

- `GET /v1beta/models`
- `GET /v1beta/openai/models`
- `POST /v1beta/models/{model}:generateContent`
- `POST /v1/engines/{model}/embeddings`

推荐请求头：

```http
x-goog-api-key: sk-你的_api_key
```

部分 Gemini 风格路径也支持查询参数形式：

```http
GET /v1beta/models?key=sk-你的_api_key
```

注意：`GET /v1/models` 建议使用标准 `Authorization: Bearer ...`。不要把 `GET /v1/models?key=...` 作为通用模型列表认证方式。

## WebSocket 认证

`GET /v1/realtime` 是 WebSocket 入口，仍然使用同一套 API Key。优先使用：

```http
Authorization: Bearer sk-你的_api_key
```

如果客户端只能通过 WebSocket 子协议传 key，也支持 `Sec-WebSocket-Protocol` 中的 `openai-insecure-api-key.{key}` 形式。

## 认证会检查什么

服务端会校验：

- API Key 是否存在。
- API Key 是否过期、禁用或额度耗尽。
- 用户账号是否可用，是否被禁用 API 访问。
- API Key 是否有分组权限。
- 如果 API Key 配置了 IP 白名单，请求来源 IP 是否在允许范围内。

## 常见认证问题

| 状态码 | 常见原因 | 处理方式 |
| --- | --- | --- |
| `401` | API Key 缺失、无效、格式错误、过期或额度耗尽 | 检查认证头，确认使用的是 API Key 而不是网页登录 token |
| `403` | 用户被禁用、API 访问被禁用、分组无权限、IP 不在白名单 | 检查账号状态、Key 权限、分组和 IP 限制 |
| `429` | 请求频率超限 | 降低并发或稍后重试 |

下一步看 [首个请求示例](/zh/getting-started/first-request)。
