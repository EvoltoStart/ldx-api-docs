---
title: "首个请求"
description: "用真实接口完成模型列表查询和第一条 chat/completions 请求。"
---

# 首个请求

## 1. 查询模型

```bash
curl https://token.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"
```

这一步用于确认：
- API Key 有效
- 账户有可用额度
- 当前可用模型能正常返回

## 2. 发起聊天请求

```bash
curl https://token.liandanxia.com/v1/chat/completions \
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

当前项目的真实接口 `POST /v1/chat/completions` 支持：
- 非流式响应
- 流式响应

## 3. 打开流式输出

```bash
curl https://token.liandanxia.com/v1/chat/completions \
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

如果你使用 OpenAI 新版 Responses 风格，可直接调用：

```bash
curl https://token.liandanxia.com/v1/responses \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "input": "hello"
  }'
```

## 5. 常见报错

- `401`
  - API Key 无效，或认证头写错
- `429`
  - 请求触发限流
- `400`
  - 请求体结构不符合接口要求

如果你已有 Claude / Gemini 的现成 SDK，下一步看 [兼容格式](/docs/zh/getting-started/compatibility)。
