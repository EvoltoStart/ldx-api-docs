---
title: "接入 GitHub Copilot CLI"
description: "通过 Copilot CLI BYOK 环境变量接入 LDX OpenAI 兼容接口。"
---

GitHub Copilot CLI 支持 BYOK（Bring Your Own Key），可以连接 OpenAI Chat Completions 兼容接口。LDX 的 OpenAI 兼容 Base URL 是 `https://api.liandanxia.com/silver/v1`。

## 适用场景

- 你希望在终端中使用 Copilot CLI。
- 你希望把 Copilot CLI 的模型调用切到 LDX。
- 你使用的模型支持工具调用和流式输出。

<Warning>
Copilot CLI BYOK 要求模型支持 tool calling/function calling 和 streaming。若所选模型或渠道不支持这些能力，Copilot CLI 会报错。请先用 `/v1/models` 确认可用模型，再结合模型能力选择。
</Warning>

## 前置条件

- 已按 GitHub 官方文档安装 GitHub Copilot CLI。
- 已准备 LDX API Key：`sk-...`。
- 当前网络可访问 `https://api.liandanxia.com/silver`。
- 示例模型 `qwen3.5-flash` 来自当前价格文档；实际以 `/v1/models` 返回为准。

检查 CLI 是否可用：

```bash
copilot --version
```

## 配置环境变量

Linux / macOS / WSL：

```bash
export COPILOT_PROVIDER_TYPE=openai
export COPILOT_PROVIDER_BASE_URL=https://api.liandanxia.com/silver/v1
export COPILOT_PROVIDER_API_KEY=sk-你的_api_key
export COPILOT_MODEL=qwen3.5-flash
```

Windows PowerShell：

```powershell
$env:COPILOT_PROVIDER_TYPE="openai"
$env:COPILOT_PROVIDER_BASE_URL="https://api.liandanxia.com/silver/v1"
$env:COPILOT_PROVIDER_API_KEY="sk-你的_api_key"
$env:COPILOT_MODEL="qwen3.5-flash"
```

配置说明：

| 环境变量 | 说明 |
| --- | --- |
| `COPILOT_PROVIDER_TYPE` | 使用 `openai`，表示 OpenAI Chat Completions 兼容接口。 |
| `COPILOT_PROVIDER_BASE_URL` | 填 `https://api.liandanxia.com/silver/v1`。 |
| `COPILOT_PROVIDER_API_KEY` | LDX API Key。 |
| `COPILOT_MODEL` | 默认模型名，必须是账号可用模型。 |

如模型不在 Copilot CLI 内置目录中，建议显式配置 token 上限：

```bash
export COPILOT_PROVIDER_MAX_PROMPT_TOKENS=128000
export COPILOT_PROVIDER_MAX_OUTPUT_TOKENS=8192
```

## 先验证 API

```bash
curl https://api.liandanxia.com/silver/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "messages": [{"role": "user", "content": "hello"}],
    "stream": true
  }'
```

## 启动 Copilot CLI

```bash
copilot
```

进入交互界面后发送：

```text
用一句话介绍当前项目
```

## 常见问题

| 问题 | 可能原因 | 处理方式 |
| --- | --- | --- |
| `401` | API Key 错误或当前 shell 未读取环境变量 | 在同一个终端重新设置 `COPILOT_PROVIDER_API_KEY`。 |
| `404` 或模型不可用 | `COPILOT_MODEL` 不在账号可用模型中 | 调用 `/v1/models` 后复制真实模型名。 |
| `400` | Provider 类型或 Base URL 不匹配 | 使用 `openai` 和 `https://api.liandanxia.com/silver/v1`。 |
| 工具调用不可用 | 模型或渠道不支持 tool calling/streaming | 换用支持工具调用和流式输出的模型。 |
| CLI 仍调用 GitHub 托管模型 | 环境变量未被当前进程继承 | 在启动 `copilot` 的同一个终端设置环境变量。 |

如果你更偏向 VS Code 图形界面，可查看 [GitHub Copilot](/zh/guides/agent-integrations-github-copilot)。

## 参考资料

- [GitHub Copilot CLI BYOK](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-byok-models)
- [GitHub Copilot CLI 产品页](https://github.com/features/copilot/cli)
- [首个请求示例](/zh/getting-started/first-request)
- [模型与价格](/zh/getting-started/pricing)
- [错误码](/zh/getting-started/error-codes)
