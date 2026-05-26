---
title: "接入 Claude Code"
description: "面向实际使用的 Claude Code 接入指南：从安装到验证到排错。"
---

这份指南适合希望在本地开发流程中使用 Claude Code，并通过 LDX 网关统一调用模型的用户。

## 什么时候选 Claude Code

- 你主要在终端和代码仓库中工作
- 你希望以 Anthropic 接口风格调用模型
- 你希望最少改动就能切换到 LDX 统一网关
- 你需要 MCP（Model Context Protocol）集成和 AI Agent 能力

## 前置条件

- 已安装 Claude Code（请参考 [Claude Code 官方安装文档](https://code.claude.com/docs/en/overview)）
- 已准备可用 API Key：`sk-...`
- 本机可访问：`https://api.liandanxia.com`
- 已安装 `curl`（用于连通性验证）

## 配置 LDX 网关接入

Claude Code 支持通过环境变量或配置文件两种方式配置。

### 方式一：环境变量（推荐用于测试）

#### Linux / macOS / WSL

```bash
export ANTHROPIC_BASE_URL=https://api.liandanxia.com
export ANTHROPIC_AUTH_TOKEN=sk-你的_api_key
export ANTHROPIC_MODEL=gpt-4o-mini
```

#### Windows PowerShell

```powershell
$env:ANTHROPIC_BASE_URL="https://api.liandanxia.com"
$env:ANTHROPIC_AUTH_TOKEN="sk-你的_api_key"
$env:ANTHROPIC_MODEL="gpt-4o-mini"
```

### 方式二：配置文件（推荐用于持久化）

创建或编辑 `~/.claude/settings.json`：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.liandanxia.com",
    "ANTHROPIC_AUTH_TOKEN": "sk-你的_api_key",
    "ANTHROPIC_MODEL": "gpt-4o-mini"
  }
}
```

**配置文件优先级**：
- `~/.claude/settings.json` — 全局配置，所有项目生效
- `.claude/settings.json` — 项目配置，可提交到版本控制
- `.claude/settings.local.json` — 项目本地配置，不提交到版本控制

### 重要配置说明

| 环境变量 | 说明 | 必需 |
|---------|------|------|
| `ANTHROPIC_BASE_URL` | LDX 网关地址，**不要**添加 `/v1` 后缀 | 是 |
| `ANTHROPIC_AUTH_TOKEN` | 你的 API Key，会作为 `Authorization: Bearer` 发送 | 是 |
| `ANTHROPIC_MODEL` | 默认模型名称，从 `/v1/models` 获取 | 推荐 |
| `ANTHROPIC_API_KEY` | 替代 `AUTH_TOKEN`，会作为 `x-api-key` 发送 | 可选 |

## 验证网关连通性

### 1. 验证模型列表

```bash
curl https://api.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"
```

### 2. 验证 Anthropic 兼容接口

```bash
curl https://api.liandanxia.com/v1/messages \
  -H "x-api-key: sk-你的_api_key" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model":"gpt-4o-mini",
    "max_tokens":256,
    "messages":[{"role":"user","content":"hello"}]
  }'
```

### 3. 启动 Claude Code

```bash
claude
```

在交互界面中输入测试提示：

```
你好，请介绍一下你自己
```

### 验证通过标准

- ✅ `/v1/models` 返回模型列表
- ✅ `/v1/messages` 返回有效 JSON 响应
- ✅ `claude` 交互界面能正常对话

## 高级配置

### 启用网关模型发现

如果你的 LDX 网关支持 `/v1/models` 端点，可以启用自动模型发现：

```bash
export CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1
```

或在 `settings.json` 中：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.liandanxia.com",
    "ANTHROPIC_AUTH_TOKEN": "sk-你的_api_key",
    "CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY": "1"
  }
}
```

### 自定义请求头

如果需要添加自定义请求头（如内部网关认证）：

```bash
export ANTHROPIC_CUSTOM_HEADERS="X-Custom-Auth: token123
X-Team-ID: engineering"
```

### 配置超时和重试

```json
{
  "env": {
    "API_TIMEOUT_MS": "1200000",
    "CLAUDE_CODE_MAX_RETRIES": "5"
  }
}
```

## 常见问题与排错

### 1. 认证失败 (401 Unauthorized)

**原因**：
- API Key 无效或已过期
- Key 中包含多余空格
- 使用了错误的认证头

**解决**：
```bash
# 检查 Key 是否正确
echo $ANTHROPIC_AUTH_TOKEN

# 重新设置（注意去除空格）
export ANTHROPIC_AUTH_TOKEN="sk-你的_api_key"
```

### 2. 端点未找到 (404 Not Found)

**原因**：
- `ANTHROPIC_BASE_URL` 配置错误
- 错误地添加了 `/v1` 后缀

**解决**：
```bash
# 正确配置（不要加 /v1）
export ANTHROPIC_BASE_URL=https://api.liandanxia.com

# 错误配置
# export ANTHROPIC_BASE_URL=https://api.liandanxia.com/v1  ❌
```

### 3. 模型不可用 (400 Bad Request)

**原因**：
- 模型名称不在可用列表中
- 请求参数不符合接口要求

**解决**：
```bash
# 先查询可用模型
curl https://api.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"

# 使用返回列表中的模型名
export ANTHROPIC_MODEL=gpt-4o-mini
```

### 4. 限流错误 (429 Too Many Requests)

**原因**：
- 请求频率超过限制
- 并发请求过多

**解决**：
- 降低请求频率
- 实现指数退避重试策略
- 联系管理员提升限额

### 5. Windows 安装问题

**问题**：Claude Code 不支持原生 Windows

**解决**：
1. 安装 WSL2：
   ```powershell
   wsl --install
   ```
2. 或安装 Git Bash 并配置：
   ```bash
   export CLAUDE_CODE_GIT_BASH_PATH="/c/Program Files/Git/bin/bash.exe"
   ```

### 6. 网关特定问题

如果你的网关不支持某些 Anthropic 特性，可以禁用：

```bash
# 禁用实验性 beta 功能
export CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1

# 禁用工具搜索（如果网关不支持 tool_reference）
export ENABLE_TOOL_SEARCH=false
```

## 排查顺序建议

1. **验证网络连通性** → `curl` 测试 Base URL
2. **验证 API Key** → 检查 Key 格式和权限
3. **验证模型名称** → 确认模型在可用列表中
4. **检查请求格式** → 对比官方接口文档
5. **查看调试日志** → 启用 `--debug` 模式

```bash
# 启用调试模式
claude --debug

# 或设置环境变量
export DEBUG=1
```

## 安全与最佳实践

### 密钥管理

❌ **不要这样做**：
```bash
# 不要把 Key 写入脚本
export ANTHROPIC_AUTH_TOKEN=sk-real-key-here
```

✅ **推荐做法**：
```bash
# 使用密钥管理工具
export ANTHROPIC_AUTH_TOKEN=$(vault kv get -field=api_key secret/ldx/claude-code)

# 或使用 apiKeyHelper
```

在 `settings.json` 中配置动态密钥获取：

```json
{
  "apiKeyHelper": "~/bin/get-ldx-key.sh",
  "env": {
    "CLAUDE_CODE_API_KEY_HELPER_TTL_MS": "3600000"
  }
}
```

### 模型选择策略

```json
{
  "env": {
    "ANTHROPIC_MODEL": "gpt-4o-mini",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "gpt-4o-mini",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "gpt-4o",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "claude-opus-4-20250514"
  }
}
```

### 项目级配置

在项目根目录创建 `.claude/settings.json`：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.liandanxia.com",
    "ANTHROPIC_MODEL": "gpt-4o-mini"
  },
  "contextPaths": [
    ".github/copilot-instructions.md",
    "CLAUDE.md"
  ]
}
```

**注意**：不要将包含真实 API Key 的配置文件提交到版本控制。

## 参考资源

- [Claude Code 官方文档](https://code.claude.com/docs/en/overview)
- [环境变量完整列表](https://claude-code.mintlify.app/en/env-vars)
- [LLM 网关配置指南](https://code.claude.com/docs/en/llm-gateway)
- [Anthropic Messages API](https://docs.anthropic.com/en/api/messages)
