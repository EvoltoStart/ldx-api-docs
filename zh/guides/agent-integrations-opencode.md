---
title: "接入 OpenCode"
description: "将 OpenCode 配置为使用 LDX API 的 OpenAI 兼容接口。"
---

# 接入 OpenCode

## 1) 准备参数

- Base URL：`https://token.liandanxia.com/v1`
- API Key：`sk-你的_api_key`
- Model：`GET /v1/models` 返回的可用模型

## 2) 环境变量

Linux / macOS：

```bash
export OPENAI_BASE_URL=https://token.liandanxia.com/v1
export OPENAI_API_KEY=<你的 LDX API Key>
export OPENAI_MODEL=<从 /v1/models 获取的可用模型>
```

Windows PowerShell：

```powershell
$env:OPENAI_BASE_URL="https://token.liandanxia.com/v1"
$env:OPENAI_API_KEY="<你的 LDX API Key>"
$env:OPENAI_MODEL="<从 /v1/models 获取的可用模型>"
```

## 3) 最小验证

```bash
curl https://token.liandanxia.com/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "messages": [{"role":"user","content":"hello"}]
  }'
```
