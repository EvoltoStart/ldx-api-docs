---
title: "接入 OpenClaw"
description: "使用 OpenClaw 自定义模型 Provider 接入 LDX API。"
---

本指南用于把 OpenClaw 的模型调用接入 LDX API。LDX 的 OpenAI 兼容 Base URL 是 `https://api.liandanxia.com/silver/v1`。

## 适用场景

- 你需要在 OpenClaw 中显式管理多个模型 Provider。
- 你希望通过 `/models` 或 `openclaw models list` 查看模型。
- 你希望将默认模型固定为 LDX 网关中的模型。

## 前置条件

- 已安装 OpenClaw。
- 已准备 LDX API Key：`sk-...`。
- 当前网络可访问 `https://api.liandanxia.com/silver`。
- 示例模型 `qwen3.5-flash`、`qwen3.5-plus` 来自当前价格文档；实际可用模型以 `GET /v1/models` 返回为准。

## 先验证 LDX 接口

```bash
curl https://api.liandanxia.com/silver/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"
```

```bash
curl https://api.liandanxia.com/silver/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "messages": [{"role": "user", "content": "hello"}],
    "stream": false
  }'
```

## 配置 Provider

OpenClaw 官方建议在修改 provider 或模型 allowlist 这类 map 时使用 `--strict-json --merge`，避免覆盖已有配置。

```bash
openclaw config set models.providers.ldx '{
  "api": "openai-completions",
  "baseUrl": "https://api.liandanxia.com/silver/v1",
  "apiKey": "sk-你的_api_key",
  "models": [
    {"id": "qwen3.5-flash"},
    {"id": "qwen3.5-plus"}
  ]
}' --strict-json --merge
```

如果你的 OpenClaw 版本不支持整块 JSON 写入，也可以直接编辑 OpenClaw 配置文件中的 `models.providers.ldx`，保持同样字段：

```json
{
  "models": {
    "providers": {
      "ldx": {
        "api": "openai-completions",
        "baseUrl": "https://api.liandanxia.com/silver/v1",
        "apiKey": "sk-你的_api_key",
        "models": [
          {"id": "qwen3.5-flash"},
          {"id": "qwen3.5-plus"}
        ]
      }
    }
  }
}
```

## 设置默认模型

```bash
openclaw models set ldx/qwen3.5-flash
```

如启用了模型 allowlist，需要把模型加入 `agents.defaults.models`：

```bash
openclaw config set agents.defaults.models '{
  "ldx/qwen3.5-flash": {},
  "ldx/qwen3.5-plus": {}
}' --strict-json --merge
```

## 验证配置

```bash
openclaw models list --provider ldx
openclaw models status
openclaw run "你好，请介绍一下当前项目"
```

验证通过标准：

- `openclaw models list --provider ldx` 能看到 LDX 模型。
- `openclaw models status` 显示默认模型为 `ldx/qwen3.5-flash`。
- `openclaw run` 能返回正常回答。

## 常见问题

| 问题 | 可能原因 | 处理方式 |
| --- | --- | --- |
| `Model is not allowed` | 启用了 allowlist 但没有加入 LDX 模型 | 使用 `agents.defaults.models` 的 `--merge` 命令加入模型。 |
| `401` | API Key 错误 | 重新设置 `models.providers.ldx.apiKey`。 |
| `404` | Base URL 错误 | 使用 `https://api.liandanxia.com/silver/v1`，不要填根地址。 |
| 模型列表为空 | provider 未保存或模型 ID 不正确 | 先确认 `/v1/models` 返回值，再更新 `models.providers.ldx.models`。 |

下一步可查看 [错误码](/zh/getting-started/error-codes) 了解 API 失败响应。

## 参考资料

- [OpenClaw 配置与自定义 Provider](https://docs.openclaw.ai/gateway/config-tools)
- [OpenClaw Models CLI](https://documentation.openclaw.ai/concepts/models)
- [首个请求示例](/zh/getting-started/first-request)
- [模型与价格](/zh/getting-started/pricing)
- [错误码](/zh/getting-started/error-codes)
