---
title: "接入 Claude Code"
description: "面向实际使用的 Claude Code 接入指南：从安装到验证到排错。"
---

这份指南适合希望在本地开发流程中使用 Claude Code，并通过 LDX 网关统一调用模型的用户。

## 什么时候选 Claude Code

- 你主要在终端和代码仓库中工作
- 你希望以 Anthropic 接口风格调用模型
- 你希望最少改动就能切换到 LDX 统一网关

## 前置条件

- 已准备可用 API Key：`sk-...`
- 本机可访问：`https://token.liandanxia.com`
- 已安装 Node.js（建议 18+）
- 已安装 `curl`（用于连通性验证）

## 第一步：安装 Claude Code

推荐安装：

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

可选安装：

```bash
npm install -g @anthropic-ai/claude-code
```

确认安装成功：

```bash
claude --version
```

## 第二步：配置 LDX 接入参数

Claude Code 会读取 Anthropic 兼容环境变量。最小配置如下。

### Linux / macOS

```bash
export ANTHROPIC_BASE_URL=https://token.liandanxia.com
export ANTHROPIC_AUTH_TOKEN=<你的 LDX API Key>
export ANTHROPIC_MODEL=<从 /v1/models 获取的模型名>
```

### Windows PowerShell

```powershell
$env:ANTHROPIC_BASE_URL="https://token.liandanxia.com"
$env:ANTHROPIC_AUTH_TOKEN="<你的 LDX API Key>"
$env:ANTHROPIC_MODEL="<从 /v1/models 获取的模型名>"
```

建议先在当前会话临时设置并完成验证，通过后再写入 shell 启动文件做持久化。

## 第三步：验证是否可用

先验证网关与 Key：

```bash
curl https://token.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"
```

再验证 Anthropic 消息接口：

```bash
curl https://token.liandanxia.com/v1/messages \
  -H "x-api-key: sk-你的_api_key" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model":"claude-sonnet-4-20250514",
    "max_tokens":256,
    "messages":[{"role":"user","content":"hello"}]
  }'
```

最后启动工具：

```bash
claude
```

### 验证通过标准

- `/v1/models` 返回模型列表
- `/v1/messages` 返回有效响应
- `claude` 中发起首条提问可正常收到回复

## 常见问题与排错建议

- `401 Unauthorized`：Key 无效、粘贴有空格或已过期
- `404 Not Found`：`ANTHROPIC_BASE_URL` 配置错误
- `400 Bad Request`：模型名不在可用列表，或请求字段不合法
- `429 Too Many Requests`：并发过高，需降低速率并做重试退避

建议按这个顺序排查：先 URL，再 Key，再模型名，最后看请求体字段。

## 安全与最佳实践

- 不要把 Key 写进仓库文件
- 团队环境建议使用密钥管理系统注入环境变量
- 为“快速问答”和“高质量生成”准备不同默认模型

## 参考来源

- Claude Code Getting Started: https://docs.anthropic.com/en/docs/claude-code/getting-started
- Claude Code Env Vars: https://code.claude.com/docs/en/env-vars
- Anthropic Messages API: https://docs.anthropic.com/en/api/messages
