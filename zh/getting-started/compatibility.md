---
title: "兼容格式"
description: "基于真实接口说明 Claude、Gemini 与其他兼容调用方式。"
---

当前项目除了统一 `/v1/*` 调用面，还保留了真实可用的兼容接口。

## Claude 兼容

对应接口：
- `POST /v1/messages`

兼容请求头：

```http
x-api-key: sk-你的_api_key
anthropic-version: 2023-06-01
```

模型列表接口在带上 `x-api-key` 与 `anthropic-version` 时，也会返回 Anthropic 风格结果。

## Gemini 兼容

对应接口：
- `GET /v1beta/models`
- `POST /v1beta/models/{model}:generateContent`
- `POST /v1/engines/{model}/embeddings`

兼容请求头：

```http
x-goog-api-key: sk-你的_api_key
```

部分 Gemini 风格路径也支持查询参数形式：

```http
GET /v1beta/models?key=sk-你的_api_key
```

`GET /v1/models` 建议使用标准 `Authorization: Bearer ...`；不要把 `GET /v1/models?key=...` 作为通用模型列表认证方式。

## 图像 / 视频兼容接口

当前真实可用的兼容接口还包括：
- `POST /v1/edits`
- `POST /v1/images/edits`
- `POST /kling/v1/videos/text2video`
- `POST /kling/v1/videos/image2video`
- `POST /jimeng`

这类接口适合你已经有现成的厂商请求格式时使用。

## 如何选择

- 新接入项目：优先使用核心 API
- 已有 Claude / Gemini SDK：优先使用兼容 API
