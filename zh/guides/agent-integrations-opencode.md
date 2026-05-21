---
title: "接入 OpenCode"
description: "面向实际使用的 OpenCode 接入指南：安装、Provider 配置、验证与排错。"
---

这份指南适合希望在项目内使用 `opencode.json` 管理模型配置，并通过 LDX 统一网关调用 OpenAI 兼容接口的用户。

## 什么时候选 OpenCode

- 你希望以“项目配置文件”方式管理模型
- 你希望不同仓库有不同默认模型
- 你需要 OpenAI 兼容接口的稳定接入路径

## 前置条件

- 已准备可用 API Key：`sk-...`
- 本机可访问：`https://token.liandanxia.com`
- 已安装 Node.js（建议 18+）
- 已安装 `curl`

## 第一步：安装 OpenCode

官方安装脚本：

```bash
curl -fsSL https://opencode.ai/install | bash
```

可选安装：

```bash
npm i -g opencode-ai@latest
```

确认安装成功：

```bash
opencode --version
```

## 第二步：在项目内配置 Provider

在项目根目录创建 `opencode.json`：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ldx": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LDX",
      "models": {
        "gpt-4o-mini": {}
      },
      "options": {
        "baseURL": "https://token.liandanxia.com/v1",
        "apiKey": "sk-你的_api_key"
      }
    }
  },
  "model": "ldx/gpt-4o-mini"
}
```

配置要点：

- `baseURL` 必须是 `.../v1`
- `model` 要与 `provider.ldx.models` 中的模型一致
- Provider 名称 `ldx` 可以自定义，但后续引用要一致

## 第三步：验证是否可用

先验证网关请求：

```bash
curl https://token.liandanxia.com/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model":"gpt-4o-mini",
    "messages":[{"role":"user","content":"hello"}]
  }'
```

再启动 OpenCode：

```bash
opencode
```

### 验证通过标准

- `chat/completions` 返回有效 JSON
- OpenCode 能识别并加载 `ldx/...` 模型
- 首条交互请求返回正常

## 常见问题与排错建议

- `401 Unauthorized`：`apiKey` 错误
- `404 Not Found`：`baseURL` 配置错误（常见漏 `/v1`）
- `400 Bad Request`：模型名或字段错误
- `429 Too Many Requests`：请求过快，触发限流

建议排查顺序：JSON 语法 -> `baseURL` -> 模型名 -> Key 权限。

## 安全与最佳实践

- `opencode.json` 推荐提交“结构”，不要提交真实 Key
- 真实 Key 建议由环境注入或本地私有配置管理
- 多环境（dev/staging/prod）建议分别命名 provider

## 参考来源

- OpenCode docs: https://opencode.ai/docs
- OpenCode Providers: https://opencode.ai/docs/providers/
- OpenCode Config: https://opencode.ai/docs/config/
