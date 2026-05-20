---
title: "认证"
description: "API Reference 入口说明：对外调用认证方式与兼容请求头。"
---

# 认证

本页只说明对外 API Reference 的认证方式。

## 1. 标准认证（核心 API）

适用范围：
- `/v1/models`
- `/v1/chat/completions`
- `/v1/responses`
- `/v1/embeddings`
- `/v1/images/*`
- `/v1/audio/*`
- `/v1/videos*`

请求头：

```http
Authorization: Bearer sk-你的_api_key
```

## 2. 兼容格式认证头（兼容 API）

按调用格式使用以下请求头：

- Claude 兼容：
  - `x-api-key: sk-你的_api_key`
  - `anthropic-version: 2023-06-01`
- Gemini 兼容：
  - `x-goog-api-key: sk-你的_api_key`
- Gemini 查询参数形式（模型列表）：
  - `GET /v1/models?key=sk-你的_api_key`

## 3. 常见认证问题

- `401`：API Key 无效，或认证头格式错误
- `403`：Key 有效，但无权限访问目标能力
- `429`：请求频率超限

下一步看 [首个请求](/docs/zh/getting-started/first-request)。
