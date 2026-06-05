---
title: "接入 OpenCode"
description: "使用 OpenCode 自定义 provider 接入 LDX OpenAI 兼容接口。"
---

本指南用于把 OpenCode 的模型请求转发到 LDX API。LDX 侧使用 OpenAI 兼容入口：`https://api.liandanxia.com/v1`。

## 适用场景

- 希望在项目内用配置文件固定默认模型。
- 希望通过 OpenAI 兼容接口接入 LDX。
- 希望不同项目使用不同模型或 API Key。

## 前置条件

- 已安装 OpenCode。
- 已准备 LDX API Key：`sk-...`。
- 当前网络可访问 `https://api.liandanxia.com`。
- 示例模型 `qwen3.5-flash` 来自当前价格文档；实际可用模型以 `GET /v1/models` 返回为准。

## 先验证 LDX 接口

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

## 配置 OpenCode

OpenCode 官方配置使用 `provider` 字段，不是 `providers`。在项目根目录创建或更新 `opencode.json`：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "ldx/qwen3.5-flash",
  "provider": {
    "ldx": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LDX",
      "options": {
        "baseURL": "https://api.liandanxia.com/v1",
        "apiKey": "sk-你的_api_key"
      },
      "models": {
        "qwen3.5-flash": {},
        "qwen3.5-plus": {}
      }
    }
  }
}
```

如果希望避免把密钥写进项目配置，可以使用环境变量引用：

```bash
export LDX_API_KEY="sk-你的_api_key"
```

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "ldx/qwen3.5-flash",
  "provider": {
    "ldx": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LDX",
      "options": {
        "baseURL": "https://api.liandanxia.com/v1",
        "apiKey": "{env:LDX_API_KEY}"
      },
      "models": {
        "qwen3.5-flash": {},
        "qwen3.5-plus": {}
      }
    }
  }
}
```

## 启动与验证

```bash
opencode
```

进入 OpenCode 后发送：

```text
你好，请介绍一下当前项目结构
```

验证通过标准：

- `curl /v1/models` 能返回模型列表。
- `curl /v1/chat/completions` 能返回正常 JSON。
- OpenCode 能识别 `ldx/qwen3.5-flash` 并完成首条回复。

## 常见问题

| 问题 | 可能原因 | 处理方式 |
| --- | --- | --- |
| 找不到 provider | 配置写成了 `providers` | 改为官方字段 `provider`。 |
| `401` | API Key 错误或环境变量未生效 | 在同一个终端重新设置 `LDX_API_KEY`，或直接验证 curl。 |
| `404` | Base URL 少了 `/v1` | OpenCode 的 `baseURL` 填 `https://api.liandanxia.com/v1`。 |
| 模型不可用 | 模型名不在账号可用列表 | 先调用 `/v1/models`，再复制真实模型名。 |

下一步可查看 [首个请求示例](/zh/getting-started/first-request) 了解底层 API 调用格式。

## 参考资料

- [OpenCode Providers](https://opencode.ai/docs/providers/)
- [OpenCode Config](https://opencode.ai/docs/config/)
- [OpenCode GitHub](https://github.com/sst/opencode)
- [首个请求示例](/zh/getting-started/first-request)
- [错误码](/zh/getting-started/error-codes)
