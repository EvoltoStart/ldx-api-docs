---
title: "Integrate GitHub Copilot CLI"
description: "Use BYOK and Anthropic-compatible settings to route Copilot CLI to LDX API."
---

# Integrate GitHub Copilot CLI

Copilot CLI supports BYOK. The recommended path is Anthropic-compatible settings.

## 1) Enable BYOK

```bash
gh copilot models
```

Follow prompts to enable BYOK.

## 2) Environment Variables

Linux / macOS:

```bash
export ANTHROPIC_BASE_URL=https://api.liandanxia.io
export ANTHROPIC_AUTH_TOKEN=<your LDX API Key>
export ANTHROPIC_MODEL=<model from /v1/models>
```

Windows PowerShell:

```powershell
$env:ANTHROPIC_BASE_URL="https://api.liandanxia.io"
$env:ANTHROPIC_AUTH_TOKEN="<your LDX API Key>"
$env:ANTHROPIC_MODEL="<model from /v1/models>"
```

## 3) Verify

```bash
gh copilot suggest "Write a shell plan to parse JSON and summarize fields"
```
