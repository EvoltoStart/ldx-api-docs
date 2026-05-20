---
title: "接入 GitHub Copilot CLI"
description: "通过 BYOK / Anthropic 兼容参数，将 Copilot CLI 接入 LDX API。"
---

# 接入 GitHub Copilot CLI

## 1) 启用 BYOK

```bash
gh copilot models
```

## 2) 设置环境变量

Linux / macOS：

```bash
export ANTHROPIC_BASE_URL=https://token.liandanxia.com
export ANTHROPIC_AUTH_TOKEN=<你的 LDX API Key>
export ANTHROPIC_MODEL=<从 /v1/models 获取的可用模型>
```

Windows PowerShell：

```powershell
$env:ANTHROPIC_BASE_URL="https://token.liandanxia.com"
$env:ANTHROPIC_AUTH_TOKEN="<你的 LDX API Key>"
$env:ANTHROPIC_MODEL="<从 /v1/models 获取的可用模型>"
```

## 3) 验证

```bash
gh copilot suggest "写一个读取 JSON 并统计字段的 shell 方案"
```
