---
title: "接入 WorkBuddy/CodeBuddy"
description: "通过本地 models.json 配置 WorkBuddy/CodeBuddy 的 LDX OpenAI 兼容模型。"
---

WorkBuddy/CodeBuddy 支持通过本地 `models.json` 添加自定义模型。LDX 的 Chat Completions 接口可以作为 OpenAI 兼容模型接入。

## 适用场景

- 你希望在桌面端或编辑器中使用 Agent 编程助手。
- 你需要用 `models.json` 固定团队可选模型。
- 你希望给 CodeBuddy 配置 lite、reasoning 等模型关系。

## 前置条件

- 已安装并登录 WorkBuddy/CodeBuddy。
- 已准备 LDX API Key：`sk-...`。
- 当前网络可访问 `https://api.liandanxia.com`。
- 示例模型 `qwen3.5-flash` 来自当前价格文档；实际以 `/v1/models` 返回为准。

## 配置文件位置

用户级配置：

```text
~/.codebuddy/models.json
```

Windows 用户级配置通常为：

```text
C:\Users\<你的用户名>\.codebuddy\models.json
```

项目级配置：

```text
<项目根目录>\.codebuddy\models.json
```

项目级 `availableModels` 会覆盖用户级配置，不会自动合并。团队项目配置中不要提交真实 API Key。

## 写入 models.json

CodeBuddy 官方示例要求 `apiKey` 写实际密钥值，不是环境变量名。

```json
{
  "models": [
    {
      "id": "ldx-qwen3.5-flash",
      "name": "LDX qwen3.5-flash",
      "vendor": "LDX",
      "url": "https://api.liandanxia.com/v1/chat/completions",
      "apiKey": "sk-你的_api_key",
      "maxInputTokens": 128000,
      "maxOutputTokens": 8192,
      "supportsToolCall": true,
      "supportsImages": false,
      "relatedModels": {
        "lite": "ldx-qwen3.5-flash",
        "reasoning": "ldx-qwen3.5-flash"
      }
    }
  ],
  "availableModels": [
    "ldx-qwen3.5-flash"
  ]
}
```

字段说明：

| 字段 | 说明 |
| --- | --- |
| `id` | CodeBuddy 内部模型 ID，建议加 `ldx-` 前缀。 |
| `name` | 模型选择器中的显示名称。 |
| `vendor` | 显示供应商，填写 `LDX`。 |
| `url` | 完整 Chat Completions 地址，必须到 `/v1/chat/completions`。 |
| `apiKey` | 实际 API Key 值。 |
| `supportsToolCall` | 所选模型和渠道支持工具调用时设为 `true`。 |
| `supportsImages` | 文本模型通常设为 `false`；多模态模型按实际能力配置。 |

## 重启并选择模型

1. 完全退出 WorkBuddy/CodeBuddy。
2. 重新打开应用，让它读取新的 `models.json`。
3. 在模型选择器中选择 `LDX qwen3.5-flash`。

## 验证 API

```bash
curl https://api.liandanxia.com/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{"model":"qwen3.5-flash","messages":[{"role":"user","content":"hi"}],"stream":false}'
```

## 常见问题

| 问题 | 可能原因 | 处理方式 |
| --- | --- | --- |
| 模型选择器不显示 | 文件位置错误或应用未完全重启 | 确认 `.codebuddy/models.json` 路径后完全退出并重开。 |
| `401` | `apiKey` 填错或密钥失效 | 用同一个 Key 先跑 curl 验证。 |
| `404` | `url` 不是完整 Chat Completions 地址 | 填 `https://api.liandanxia.com/v1/chat/completions`。 |
| 项目配置覆盖用户配置 | 项目级 `availableModels` 不合并用户级 | 在项目级配置里列出需要的全部模型 ID。 |

实际模型、上下文长度和价格请以 [模型与价格](/zh/getting-started/pricing) 为准。

## 参考资料

- [CodeBuddy models.json 配置指南](https://www.codebuddy.cn/docs/ide/Features/models)
- [CodeBuddy CLI Models](https://www.codebuddy.ai/docs/cli/models)
- [首个请求示例](/zh/getting-started/first-request)
- [模型与价格](/zh/getting-started/pricing)
- [错误码](/zh/getting-started/error-codes)
