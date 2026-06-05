---
title: "首个请求示例"
description: "从模型列表到第一条 Chat Completions 请求，按真实接口完成一次最小可用调用。"
---

本页给出一条最短可跑通的调用链路：先验证 API Key，再发出第一条模型请求。示例使用标准 OpenAI 兼容入口，适合新接入项目、后端服务和命令行调试。

开始前需要准备：

| 项目 | 示例                           | 说明 |
| --- |------------------------------| --- |
| API Base URL | `https://api.liandanxia.com` | 中国区文档示例使用该域名。 |
| API Key | `sk-你的_api_key`              | 在请求头中作为 Bearer Token 传入。 |
| 模型名 | `qwen3.5-flash`              | 示例模型来自当前价格文档；实际可用模型以模型列表接口返回为准。 |

如果还不确定 API Key 应该放在哪个请求头里，先看 [认证](/zh/getting-started/authentication)。

## 1. 设置环境变量

建议先把域名和 API Key 放进环境变量，后续示例可以直接复用。

```bash
export LDX_BASE_URL="https://api.liandanxia.com"
export LDX_API_KEY="sk-你的_api_key"
```

Windows PowerShell 可以使用：

```powershell
$env:LDX_BASE_URL = "https://api.liandanxia.com"
$env:LDX_API_KEY = "sk-你的_api_key"
```

## 2. 查询模型列表

第一步先调用 `GET /v1/models`。这一步不产生模型推理内容，适合用来确认 API Key、账号状态和可用模型。

```bash
curl "$LDX_BASE_URL/v1/models" \
  -H "Authorization: Bearer $LDX_API_KEY"
```

如果请求成功，响应中会返回当前账号可访问的模型列表。后续请求中的 `model` 字段应该从这里选择。

也可以查询单个模型：

```bash
curl "$LDX_BASE_URL/v1/models/qwen3.5-flash" \
  -H "Authorization: Bearer $LDX_API_KEY"
```

这一步失败时，通常先排查：

| 状态码 | 常见原因 | 处理方式 |
| --- | --- | --- |
| `401` | API Key 缺失、无效、过期或额度耗尽 | 检查 `Authorization: Bearer ...` 是否正确，确认使用的是 API Key。 |
| `403` | 账号、分组、IP 白名单或 API 访问权限不满足要求 | 检查账号状态、Key 权限、分组和 IP 限制。 |
| `503` | 当前模型或渠道暂不可用 | 稍后重试，或换用模型列表中其他可用模型。 |

## 3. 发送第一条聊天请求

推荐把 `POST /v1/chat/completions` 作为第一条推理请求。它使用 OpenAI Chat Completions 风格，字段简单，适合快速验证接入是否成功。

```bash
curl "$LDX_BASE_URL/v1/chat/completions" \
  -H "Authorization: Bearer $LDX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "messages": [
      {
        "role": "system",
        "content": "你是一个简洁、准确的中文助手。"
      },
      {
        "role": "user",
        "content": "用一句话介绍 LDX API。"
      }
    ]
  }'
```

请求体字段说明：

| 字段 | 是否必填 | 说明 |
| --- | --- | --- |
| `model` | 是 | 要调用的模型名。建议先从 `GET /v1/models` 返回结果中选择。 |
| `messages` | 是 | 对话消息数组，通常包含 `system`、`user`、`assistant` 等角色。 |
| `role` | 是 | 消息角色。第一条用户输入使用 `user`。 |
| `content` | 是 | 消息内容。文本模型使用字符串即可。 |

成功响应通常会包含 `choices`，模型回复位于 `choices[0].message.content`。

```json
{
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": "LDX API 是一个兼容多模型调用的 API 聚合平台。"
      }
    }
  ]
}
```

<Note>
示例响应只展示关键字段。真实响应还可能包含 `id`、`object`、`created`、`model`、`usage` 等字段，具体以接口实际返回为准。
</Note>

## 4. 启用流式输出

如果希望边生成边接收结果，把 `stream` 设置为 `true`。

```bash
curl "$LDX_BASE_URL/v1/chat/completions" \
  -H "Authorization: Bearer $LDX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "stream": true,
    "messages": [
      {
        "role": "user",
        "content": "给我写一个三点总结。"
      }
    ]
  }'
```

流式响应适合聊天界面、长文本生成和需要尽快展示首字的场景。客户端需要按 Server-Sent Events 或兼容流式格式逐段读取内容。

## 5. Responses 示例

如果客户端已经使用 OpenAI Responses 风格，可以调用 `POST /v1/responses`。

```bash
curl "$LDX_BASE_URL/v1/responses" \
  -H "Authorization: Bearer $LDX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "input": "用一句话介绍 LDX API。"
  }'
```

项目也提供了压缩上下文入口：

```bash
curl "$LDX_BASE_URL/v1/responses/compact" \
  -H "Authorization: Bearer $LDX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "input": "请压缩这段对话上下文。"
  }'
```

不同模型是否支持 Responses 或压缩入口，取决于后端可用渠道配置。若返回模型或渠道不可用，请换用 `GET /v1/models` 中可见的模型。

## 6. 兼容格式示例

如果已有 Claude 或 Gemini 的现成 SDK，可以用兼容入口做首测。

Claude Messages 示例：

```bash
curl "$LDX_BASE_URL/v1/messages" \
  -H "x-api-key: $LDX_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "max_tokens": 128,
    "messages": [
      {
        "role": "user",
        "content": "hello"
      }
    ]
  }'
```

Gemini 模型列表示例：

```bash
curl "$LDX_BASE_URL/v1beta/models" \
  -H "x-goog-api-key: $LDX_API_KEY"
```

也可以使用查询参数形式：

```bash
curl "$LDX_BASE_URL/v1beta/models?key=$LDX_API_KEY"
```

兼容入口的认证头和返回格式可能与标准 `/v1/*` 接口不同。详细说明见 [兼容格式](/zh/getting-started/compatibility)。

## 7. 常见报错
| 状态码 | 常见原因 | 处理方式 |
| --- | --- | --- |
| `401` | API Key 缺失、无效、过期或额度耗尽 | 重新检查认证头和 Key 状态 |
| `403` | 账号禁用、API 访问禁用、分组无权限、IP 不在白名单 | 检查账号、Key 权限、分组和 IP 限制 |
| `400` | 请求体字段不符合目标接口格式 | 检查 `model`、`messages`、`input` 等字段 |
| `429` | 请求频率超限 | 降低并发或稍后重试 |

