---
title: "接入 Hermes"
description: "通过 Hermes Agent 的模型配置接入 LDX API。"
---

Hermes Agent 支持通过 `hermes model` 或 `hermes setup` 选择模型 Provider。LDX 可按 OpenAI 兼容或 Anthropic 兼容方式接入。

## 适用场景

- 你希望使用 Hermes 的长期记忆、技能沉淀和工具能力。
- 你希望模型请求统一走 LDX API。
- 你需要在 Hermes 中切换不同 Provider 或模型。

## 前置条件

- 已准备 LDX API Key：`sk-...`。
- 当前网络可访问 `https://api.liandanxia.com`。
- 示例模型 `qwen3.5-flash` 来自当前价格文档；实际以 `/v1/models` 返回为准。

## 安装 Hermes

Linux / macOS / WSL2：

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

Windows PowerShell：

```powershell
iex (irm https://hermes-agent.nousresearch.com/install.ps1)
```

安装后重新加载 shell，并检查命令：

```bash
hermes --version
```

## 配置模型

执行：

```bash
hermes model
```

或首次完整配置：

```bash
hermes setup
```

按 Hermes 界面填写：

| 配置项 | 推荐值 |
| --- | --- |
| Provider | 选择 `OpenAI Compatible` / `Custom`，或任何允许修改 Base URL 的 Provider。 |
| OpenAI 兼容 Base URL | `https://api.liandanxia.com/v1` |
| Anthropic 兼容 Base URL | `https://api.liandanxia.com` |
| API Key | `sk-你的_api_key` |
| Model | `qwen3.5-flash`，或 `/v1/models` 返回的真实模型名。 |

<Warning>
OpenAI 兼容配置使用 `/v1` Base URL；Anthropic 兼容配置使用根地址，让工具拼接 `/v1/messages`。不要把其他平台官方 API 地址直接填进 Hermes。
</Warning>

## 验证 LDX 接口

```bash
curl https://api.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"
```

```bash
curl https://api.liandanxia.com/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "messages": [{"role": "user", "content": "hello"}],
    "stream": false
  }'
```

## 启动 Hermes

```bash
hermes
```

发送测试提示：

```text
请阅读当前项目结构，并给出三条改进建议。
```

## 常见问题

| 问题 | 可能原因 | 处理方式 |
| --- | --- | --- |
| Provider 不支持自定义 Base URL | 选择了固定官方 Provider | 改选 OpenAI Compatible / Custom Provider。 |
| `401` | API Key 无效或未保存 | 重新执行 `hermes model` 或 `hermes setup`。 |
| `404` | 模型名或 Base URL 错误 | 先查 `/v1/models`，OpenAI 兼容 Base URL 填 `/v1`。 |
| Anthropic 模式请求失败 | Base URL 写成了 `/v1` | Anthropic 兼容模式填根地址 `https://api.liandanxia.com`。 |

下一步可查看 [首个请求示例](/zh/getting-started/first-request)。

## 参考资料

- [Hermes Agent 文档](https://hermes-agent.nousresearch.com/docs/)
- [Hermes Agent GitHub](https://github.com/NousResearch/hermes-agent)
- [Hermes Agent 快速开始](https://github.com/NousResearch/hermes-agent/blob/main/website/docs/getting-started/quickstart.md)
- [首个请求示例](/zh/getting-started/first-request)
- [错误码](/zh/getting-started/error-codes)
