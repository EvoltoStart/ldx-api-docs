---
title: "接入 OpenClaw"
description: "面向实际使用的 OpenClaw 接入指南：安装、Provider 注册、验证与排错。"
---

这份指南适合希望用 OpenClaw 的 `models.providers` 机制接入 LDX，并统一管理默认模型的用户。

## 什么时候选 OpenClaw

- 你需要显式管理多个模型 Provider
- 你希望通过 CLI 直接查看模型状态
- 你希望把默认模型策略固定在工具配置中

## 前置条件

- 已准备可用 API Key：`sk-...`
- 本机可访问：`https://token.liandanxia.com`
- 已安装 Node.js（建议 18+）
- 已安装 `curl`

## 第一步：安装 OpenClaw

官方安装脚本：

```bash
curl -fsSL https://openclaw.ai/install | bash
```

可选安装：

```bash
npm install -g openclaw@latest
```

确认安装成功：

```bash
openclaw --version
```

## 第二步：注册 LDX Provider

执行以下命令：

```bash
openclaw config set models.providers.ldx.api "openai-completions"
openclaw config set models.providers.ldx.baseUrl "https://token.liandanxia.com/v1"
openclaw config set models.providers.ldx.apiKey "sk-你的_api_key"
openclaw config set models.providers.ldx.models '[{"id":"gpt-4o-mini"}]'
openclaw config set agents.defaults.model.primary "ldx/gpt-4o-mini"
```

可选检查：

```bash
openclaw config list
```

确认 `models.providers.ldx` 与默认模型已生效。

## 第三步：验证是否可用

```bash
openclaw models list
openclaw models status
openclaw run "hello"
```

### 验证通过标准

- `models list` 能看到目标模型
- `models status` 无关键错误
- `run "hello"` 返回有效响应

## 常见问题与排错建议

- `401 Unauthorized`：`apiKey` 无效
- `404 Not Found`：`baseUrl` 错误或缺少 `/v1`
- `400 Bad Request`：模型名或配置字段错误
- `429 Too Many Requests`：限流触发

建议排查顺序：`baseUrl` -> `api` 类型 -> 模型映射 -> Key。

## 安全与最佳实践

- Provider 名称建议短且稳定（如 `ldx`）
- 默认模型先用 `models list` 验证后再设置
- 团队协作时将“配置结构”纳入文档，将密钥排除在仓库外

## 参考来源

- OpenClaw install: https://docs.openclaw.ai/getting-started/installing-openclaw
- OpenClaw model providers: https://docs.openclaw.ai/concepts/model-providers
- OpenClaw OpenAI provider: https://docs.openclaw.ai/providers/openai
- OpenClaw configuration: https://docs.openclaw.ai/configuration
