---
title: "接入 Claude Code"
description: "通过 LDX Anthropic Messages 兼容接口配置 Claude Code。"
---

本指南用于让 Claude Code 通过 LDX 网关调用模型。Claude Code 会基于 `ANTHROPIC_BASE_URL` 拼接 `/v1/messages`，因此这里填写根地址：`https://api.liandanxia.com`。

## 适用场景

- 你主要在终端中使用 Claude Code。
- 你希望继续使用 Anthropic Messages 格式。
- 你希望 API Key、限额和用量统一走 LDX。

## 前置条件

- 已安装 Claude Code。
- 已准备 LDX API Key：`sk-...`。
- 当前网络可访问 `https://api.liandanxia.com`。
- 示例模型 `qwen3.5-flash` 来自当前价格文档；实际可用模型以 `/v1/models` 返回为准。

## 配置环境变量

Linux / macOS / WSL：

```bash
export ANTHROPIC_BASE_URL=https://api.liandanxia.com
export ANTHROPIC_AUTH_TOKEN=sk-你的_api_key
export ANTHROPIC_MODEL=qwen3.5-flash
```

Windows PowerShell：

```powershell
$env:ANTHROPIC_BASE_URL="https://api.liandanxia.com"
$env:ANTHROPIC_AUTH_TOKEN="sk-你的_api_key"
$env:ANTHROPIC_MODEL="qwen3.5-flash"
```

配置说明：

| 配置项 | 正确值 | 说明 |
| --- | --- | --- |
| `ANTHROPIC_BASE_URL` | `https://api.liandanxia.com` | 填根地址，不加 `/v1`。 |
| `ANTHROPIC_AUTH_TOKEN` | `sk-...` | Claude Code 会作为 Authorization Bearer 发送。 |
| `ANTHROPIC_API_KEY` | `sk-...` | 可选替代项；未设置 `AUTH_TOKEN` 时作为 `x-api-key` 使用。 |
| `ANTHROPIC_MODEL` | `qwen3.5-flash` | 使用 `/v1/models` 返回的真实模型名。 |

## 持久化配置

用户级配置写入 `~/.claude/settings.json`：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.liandanxia.com",
    "ANTHROPIC_AUTH_TOKEN": "sk-你的_api_key",
    "ANTHROPIC_MODEL": "qwen3.5-flash"
  }
}
```

项目级共享配置不要包含真实 API Key，可以只提交模型和 Base URL：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.liandanxia.com",
    "ANTHROPIC_MODEL": "qwen3.5-flash"
  }
}
```

## 验证 LDX 接口

```bash
curl https://api.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"
```

```bash
curl https://api.liandanxia.com/v1/messages \
  -H "x-api-key: sk-你的_api_key" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "max_tokens": 256,
    "messages": [{"role": "user", "content": "hello"}]
  }'
```

## 启动 Claude Code

```bash
claude
```

进入交互界面后发送：

```text
你好，请介绍一下当前项目结构
```

## 模型选择说明

Claude Code 的网关模型发现只会把部分符合 Anthropic 命名规则的模型加入模型选择器。对于 `qwen3.5-flash` 这类非 Claude 命名模型，建议显式设置 `ANTHROPIC_MODEL`。

如需尝试网关模型发现：

```bash
export CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1
```

## 常见问题

| 问题 | 可能原因 | 处理方式 |
| --- | --- | --- |
| `404` | `ANTHROPIC_BASE_URL` 错填为 `/v1` | 改为 `https://api.liandanxia.com`。 |
| `401` | API Key 错误或头部不匹配 | 优先使用 `ANTHROPIC_AUTH_TOKEN=sk-...`，再用 curl 验证。 |
| 模型不可用 | `ANTHROPIC_MODEL` 不在账号可用列表 | 调用 `/v1/models` 后复制真实模型名。 |
| 模型选择器看不到 qwen 模型 | Claude Code 网关发现有命名过滤 | 显式设置 `ANTHROPIC_MODEL`。 |

下一步可查看 [首个请求示例](/zh/getting-started/first-request)。

## 参考资料

- [Claude Code LLM gateway](https://code.claude.com/docs/en/llm-gateway)
- [Claude Code 总览](https://code.claude.com/docs/en/overview)
- [Anthropic Messages API](https://docs.anthropic.com/en/api/messages)
- [首个请求示例](/zh/getting-started/first-request)
- [错误码](/zh/getting-started/error-codes)
