---
title: "兼容格式"
description: "基于真实接口说明 Claude、Gemini 与其他兼容调用方式。"
---

# 兼容格式

当前项目除了统一的 `/v1/*` 调用面，还保留了若干真实存在的兼容接口。

## Claude 兼容

对应接口：

- `POST /v1/messages`

兼容头：

```http
x-api-key: sk-你的_api_key
anthropic-version: 2023-06-01
```

项目里的模型列表接口也明确支持在带上 `x-api-key` 和 `anthropic-version` 时返回 Anthropic 风格结果。

## Gemini 兼容

对应接口：

- `GET /v1beta/models`
- `POST /v1beta/models/{model}:generateContent`
- `POST /v1/engines/{model}/embeddings`

兼容头：

```http
x-goog-api-key: sk-你的_api_key
```

另外，`GET /v1/models` 还支持用查询参数：

```http
GET /v1/models?key=sk-你的_api_key
```

## 图像 / 视频兼容接口

当前真实存在的兼容接口还包括：

- `POST /v1/edits`
- `POST /v1/images/edits`
- `POST /kling/v1/videos/text2video`
- `POST /kling/v1/videos/image2video`
- `POST /jimeng`

这些接口适合你已经有既定请求格式，或者你就是在对接对应厂商格式时使用。

## 该优先用哪个

- 新接入项目：优先用 `核心 API`
- 已有 Claude / Gemini SDK：优先用 `兼容 API`
- 账户、用量、充值、发票等控制台流程：走内部账户路由（不在对外 API Reference）
