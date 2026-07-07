---
title: "接入 Kilo Code"
description: "通过 Kilo Code 的 OpenAI 兼容或自定义 Provider 接入 LDX API。"
---

Kilo Code 支持通过交互式 Provider 配置接入外部模型。LDX 提供 OpenAI 兼容 Chat Completions 接口，适合使用 `OpenAI Compatible` 或 `Custom OpenAI` 类型接入。

## 适用场景

- 你希望在 Kilo Code 中使用 LDX 模型。
- 你希望通过模型选择器切换 Provider。
- 你需要 OpenAI 兼容接口，不想改造请求格式。

## 前置条件

- 已按 Kilo Code 官方文档安装 CLI 或编辑器扩展。
- 已准备 LDX API Key：`sk-...`。
- 当前网络可访问 `https://api.liandanxia.com`。
- 示例模型 `qwen3.5-flash` 来自当前价格文档；实际以 `/v1/models` 返回为准。

## 启动 Kilo Code

在项目目录中启动 Kilo Code：

```bash
kilo
```

## 连接 LDX Provider

在 Kilo Code 中打开 Provider 连接流程：

```text
/connect
```

选择 `OpenAI Compatible`、`Custom OpenAI` 或同类可自定义 Base URL 的 Provider 类型，然后填写：

| 配置项 | 推荐值 |
| --- | --- |
| Provider Name | `LDX` |
| Provider Type | `OpenAI Compatible` |
| Base URL | `https://api.liandanxia.com/v1` |
| API Key | `sk-你的_api_key` |
| Default Model | `qwen3.5-flash` |

<Warning>
不要选择只能连接固定官方地址的 DeepSeek Provider。若该 Provider 不能修改 Base URL，请改用 OpenAI Compatible / Custom Provider，否则请求不会经过 LDX。
</Warning>

## 选择模型

打开模型选择器：

```text
/models
```

选择刚配置的 LDX 模型，例如：

```text
LDX / qwen3.5-flash
```

如需其他模型，先查询：

```bash
curl https://api.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"
```

## 验证接口

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

## 常见问题

| 问题 | 可能原因 | 处理方式 |
| --- | --- | --- |
| 找不到 LDX 模型 | Provider 未保存或模型列表未刷新 | 重新执行 `/connect`，再打开 `/models`。 |
| `401` | API Key 填错或多了空格 | 重新粘贴 `sk-...`，再用 curl 验证。 |
| `404` | Base URL 或模型名错误 | Base URL 填 `/v1`，模型名以 `/v1/models` 为准。 |
| 请求走到 DeepSeek 官方 | 选择了固定 DeepSeek Provider | 改用 OpenAI Compatible / Custom Provider。 |

下一步可查看 [错误码](/zh/getting-started/error-codes)。

## 参考资料

- [Kilo CLI 文档](https://kilo.ai/docs/cli)
- [Kilo Code 快速开始](https://kilo.ai/docs/getting-started)
- [首个请求示例](/zh/getting-started/first-request)
- [模型与价格](/zh/getting-started/pricing)
- [错误码](/zh/getting-started/error-codes)
