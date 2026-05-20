---
title: "Integrate GitHub Copilot"
description: "Connect VS Code Copilot Chat to LDX API through a compatible extension workflow."
---

# Integrate GitHub Copilot

Copilot Chat does not expose a direct arbitrary OpenAI endpoint field. A practical path is using a compatible extension workflow.

## 1) Install

- VS Code (recommended 1.116+)
- Extension: `DeepSeek V4 for Copilot Chat` (`Vizards.deepseek-v4-for-copilot`)

## 2) Set API Key

Run command:

- `DeepSeek: Set API Key`

Use your LDX API key (`sk-...`).

## 3) Switch to Your API Base

In `settings.json`:

```json
{
  "deepseek-copilot.baseUrl": "https://api.liandanxia.io/v1"
}
```

## 4) Optional Model Mapping

```json
{
  "deepseek-copilot.modelIdOverrides": {
    "deepseek-v4-flash": "gpt-4o-mini",
    "deepseek-v4-pro": "gpt-4o"
  }
}
```

Use model names available from `GET /v1/models`.
