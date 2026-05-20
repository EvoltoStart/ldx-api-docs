---
title: "首个请求"
description: "用真实接口完成模型列表查询和第一条 chat/completions 请求。"
---

# 首个请求

## 1. 查询模型

```bash
curl https://api.liandanxia.io/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"
```

这是最稳的第一步，用来确认：

- API Key 有效
- 账户还有额度
- 当前可用模型列表正常返回

## 2. 发起聊天请求

```bash
curl https://api.liandanxia.io/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "用一句话介绍你自己"}
    ]
  }'
```

当前项目的真实接口描述里，`POST /v1/chat/completions` 支持：

- 非流式响应
- 流式响应

## 3. 打开流式

```bash
curl https://api.liandanxia.io/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "stream": true,
    "messages": [
      {"role": "user", "content": "给我写一个三点总结"}
    ]
  }'
```

## 4. Responses API

如果你接的是 OpenAI 新的 Responses 风格，可以直接用：

```bash
curl https://api.liandanxia.io/v1/responses \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "input": "hello"
  }'
```

## 5. 常见报错先看什么

- `401`
  - API Key 无效，或认证头写错
- `429`
  - 当前请求被限频
- `400`
  - 参数结构不符合接口要求

如果你已经有现成的 Claude / Gemini SDK，下一步看[兼容格式](/docs/zh/getting-started/compatibility)。
