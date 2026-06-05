---
title: "接入 GitHub Copilot"
description: "在 VS Code 的 GitHub Copilot 工作流中接入 LDX API 的可行方式、限制和验证步骤。"
---

这份指南适合已经在 VS Code 中使用 GitHub Copilot Chat，并希望把部分模型请求切到 LDX API 的用户。

<Note>
GitHub Copilot Chat 官方客户端通常不提供直接填写任意 OpenAI 兼容 Base URL 的入口。要在 Copilot Chat 模型选择器中使用 LDX，需要使用支持 BYOK 或自定义 Provider 的 Copilot 扩展/插件；如果你需要稳定的命令行接入，优先看 [GitHub Copilot CLI](/zh/guides/agent-integrations-github-copilot-cli)。
</Note>

## 什么时候选 GitHub Copilot

- 你主要在 VS Code 中工作
- 你已经有 GitHub Copilot 订阅
- 你希望保留 Copilot Chat、Agent 模式、工具调用和 MCP 工作流
- 你使用的 Copilot 扩展支持自定义 API Key、Base URL 和模型名

## 前置条件

- 已安装 VS Code
- 已安装并登录 GitHub Copilot
- 已准备可用 API Key：`sk-...`
- 本机可访问：`https://api.liandanxia.com`
- 已通过 `GET /v1/models` 确认可用模型

## 先验证 LDX 网关

在配置 Copilot 前，先确认 API Key 和模型可用。

```bash
curl https://api.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"
```

再验证 Chat Completions：

```bash
curl https://api.liandanxia.com/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "messages": [
      {
        "role": "user",
        "content": "hello"
      }
    ],
    "stream": false
  }'
```

## 配置方式

如果你的 Copilot 扩展支持自定义模型或 OpenAI 兼容 Provider，按下面的值填写。

| 配置项 | 推荐值 | 说明 |
| --- | --- | --- |
| Provider 类型 | `openai-compatible` 或 `openai` | 具体名称取决于扩展。 |
| Base URL | `https://api.liandanxia.com/v1` | 如果扩展要求完整接口地址，使用 `https://api.liandanxia.com/v1/chat/completions`。 |
| API Key | `sk-你的_api_key` | 不要把 Base URL 填到 API Key 字段。 |
| Model | `qwen3.5-flash` | 示例模型；实际以 `/v1/models` 返回为准。 |
| Streaming | 开启 | Copilot Chat 通常适合流式输出。 |

常见流程：

1. 打开 VS Code 命令面板：`Ctrl+Shift+P` / `Cmd+Shift+P`。
2. 打开扩展的 Provider、Model 或 BYOK 配置入口。
3. 新增 OpenAI 兼容 Provider，名称可以填写 `LDX`。
4. 填入 Base URL、API Key 和模型名。
5. 回到 Copilot Chat 的模型选择器，选择刚添加的 LDX 模型。

## 推荐模型策略

| 场景 | 建议 |
| --- | --- |
| 日常问答、轻量代码解释 | 选择响应快、成本低的模型。 |
| 多文件修改、复杂分析 | 选择上下文更长、推理能力更强的模型。 |
| 企业团队统一接入 | 在扩展或工作区配置中固定 Provider 名称和默认模型。 |

实际模型和价格请以 [模型与价格](/zh/getting-started/pricing) 为准。

## 验证通过标准

- Copilot Chat 模型选择器中能看到 LDX 模型。
- 发送 `hello` 后能正常返回回答。
- 请求失败时能在错误信息中看到 LDX 或上游返回的状态码。

## 常见问题

| 问题 | 可能原因 | 处理方式 |
| --- | --- | --- |
| Copilot 原生设置里找不到 Base URL | 官方 Copilot Chat 不一定支持任意自定义 Provider | 使用支持 BYOK/自定义 Provider 的扩展，或改用 Copilot CLI。 |
| `401` | API Key 无效、缺失或填错位置 | 确认填写的是 `sk-...`，并放在 API Key 字段。 |
| `404` 或模型不可用 | 模型名不在当前账号可用列表中 | 先调用 `/v1/models`，再复制真实模型名。 |
| 响应很慢 | 模型较大、上下文过长或网络波动 | 降低上下文、换用更快模型，或启用流式输出。 |

下一步可以查看 [GitHub Copilot CLI](/zh/guides/agent-integrations-github-copilot-cli) 获取更稳定的 BYOK 命令行接入方式。

## 参考资料

- [GitHub Copilot 文档](https://docs.github.com/en/copilot)
- [GitHub Copilot CLI BYOK](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-byok-models)
- [GitHub Copilot CLI 产品页](https://github.com/features/copilot/cli)
- [首个请求示例](/zh/getting-started/first-request)
- [模型与价格](/zh/getting-started/pricing)
