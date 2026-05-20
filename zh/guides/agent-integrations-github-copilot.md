---
title: "接入 GitHub Copilot"
description: "在 VS Code 的 Copilot Chat 中接入 LDX API。"
---

# 接入 GitHub Copilot

建议通过兼容扩展方式接入。

## 1) 安装扩展

- VS Code（建议 1.116+）
- `DeepSeek V4 for Copilot Chat`（`Vizards.deepseek-v4-for-copilot`）

## 2) 配置 Key

命令面板执行：`DeepSeek: Set API Key`

## 3) 设置 Base URL

```json
{
  "deepseek-copilot.baseUrl": "https://token.liandanxia.com/v1"
}
```

## 4) 模型映射（可选）

```json
{
  "deepseek-copilot.modelIdOverrides": {
    "deepseek-v4-flash": "gpt-4o-mini",
    "deepseek-v4-pro": "gpt-4o"
  }
}
```
