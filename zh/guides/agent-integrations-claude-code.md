---
title: "接入 Claude Code"
description: "参考 Claude Code 迁移方式，将请求切到 LDX API 的真实兼容接口。"
---

# 接入 Claude Code

Claude Code 可通过 Anthropic 兼容参数接入当前项目。

## 从现有安装迁移

Linux / macOS：

```bash
export ANTHROPIC_BASE_URL=https://token.liandanxia.com
export ANTHROPIC_AUTH_TOKEN=<你的 LDX API Key>
export ANTHROPIC_MODEL=<从 /v1/models 获取的可用模型>
export ANTHROPIC_DEFAULT_OPUS_MODEL=<从 /v1/models 获取的可用模型>
export ANTHROPIC_DEFAULT_SONNET_MODEL=<从 /v1/models 获取的可用模型>
export ANTHROPIC_DEFAULT_HAIKU_MODEL=<从 /v1/models 获取的可用模型>
```

Windows PowerShell：

```powershell
$env:ANTHROPIC_BASE_URL="https://token.liandanxia.com"
$env:ANTHROPIC_AUTH_TOKEN="<你的 LDX API Key>"
$env:ANTHROPIC_MODEL="<从 /v1/models 获取的可用模型>"
$env:ANTHROPIC_DEFAULT_OPUS_MODEL="<从 /v1/models 获取的可用模型>"
$env:ANTHROPIC_DEFAULT_SONNET_MODEL="<从 /v1/models 获取的可用模型>"
$env:ANTHROPIC_DEFAULT_HAIKU_MODEL="<从 /v1/models 获取的可用模型>"
```

## 从零接入

```bash
curl https://token.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"
```

## 兼容端点验证

```bash
curl https://token.liandanxia.com/v1/messages \
  -H "x-api-key: sk-你的_api_key" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-sonnet-4-20250514",
    "max_tokens": 512,
    "messages": [{"role":"user","content":"hello"}]
  }'
```
