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

- 已安装 OpenClaw（请参考 [OpenClaw 官方安装文档](https://docs.openclaw.ai/)）
- 已准备可用 API Key：`sk-...`
- 本机可访问：`https://api.liandanxia.com`
- 已安装 `curl`

## 注册 LDX Provider

OpenClaw 使用 CLI 命令或配置文件管理 Provider。

### 方式一：使用 CLI 命令（推荐）

```bash
# 设置 Provider 类型为 OpenAI 兼容
openclaw config set models.providers.ldx.api "openai-completions"

# 设置 Base URL
openclaw config set models.providers.ldx.baseUrl "https://api.liandanxia.com/v1"

# 设置 API Key
openclaw config set models.providers.ldx.apiKey "sk-你的_api_key"

# 配置可用模型列表
openclaw config set models.providers.ldx.models '[{"id":"gpt-4o-mini"},{"id":"gpt-4o"},{"id":"claude-sonnet-4-20250514"}]'

# 设置默认主模型
openclaw config set agents.defaults.model.primary "ldx/gpt-4o-mini"
```

### 方式二：直接编辑配置文件

配置文件位置：`~/.openclaw/openclaw.json`

```json
{
  "models": {
    "providers": {
      "ldx": {
        "api": "openai-completions",
        "baseUrl": "https://api.liandanxia.com/v1",
        "apiKey": "sk-你的_api_key",
        "models": [
          {"id": "gpt-4o-mini"},
          {"id": "gpt-4o"},
          {"id": "claude-sonnet-4-20250514"}
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "ldx/gpt-4o-mini",
        "fallback": "ldx/gpt-4o"
      }
    }
  }
}
```

### 配置说明

| 配置项 | 说明 | 必需 |
|--------|------|------|
| `api` | API 类型，使用 `openai-completions` | 是 |
| `baseUrl` | LDX 网关地址，**必须**包含 `/v1` | 是 |
| `apiKey` | 你的 API Key | 是 |
| `models` | 可用模型列表 | 是 |
| `primary` | 默认主模型，格式 `provider/model` | 推荐 |
| `fallback` | 备用模型，主模型失败时使用 | 可选 |

### 验证配置

```bash
# 查看所有配置
openclaw config list

# 查看 Provider 配置
openclaw config get models.providers.ldx
```

## 验证模型可用性

### 1. 列出可用模型

```bash
openclaw models list
```

应该看到类似输出：

```
Available models:
  ldx/gpt-4o-mini
  ldx/gpt-4o
  ldx/claude-sonnet-4-20250514
```

### 2. 检查模型状态

```bash
openclaw models status
```

### 3. 测试对话

```bash
openclaw run "你好，请介绍一下你自己"
```

### 验证通过标准

- ✅ `models list` 能看到目标模型
- ✅ `models status` 无关键错误
- ✅ `run` 命令返回有效响应

## 高级配置

### 多 Provider 配置

OpenClaw 支持同时配置多个 Provider：

```json
{
  "models": {
    "providers": {
      "ldx": {
        "api": "openai-completions",
        "baseUrl": "https://api.liandanxia.com/v1",
        "apiKey": "sk-ldx-key",
        "models": [
          {"id": "gpt-4o-mini"},
          {"id": "gpt-4o"}
        ]
      },
      "anthropic": {
        "api": "anthropic",
        "apiKey": "sk-ant-key",
        "models": [
          {"id": "claude-sonnet-4-20250514"}
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "ldx/gpt-4o-mini",
        "fallback": "anthropic/claude-sonnet-4-20250514"
      }
    }
  }
}
```

### 模型别名

为常用模型创建别名：

```bash
# 创建别名
openclaw config set models.aliases.fast "ldx/gpt-4o-mini"
openclaw config set models.aliases.smart "ldx/gpt-4o"
openclaw config set models.aliases.claude "ldx/claude-sonnet-4-20250514"

# 使用别名
openclaw run --model fast "快速任务"
openclaw run --model smart "复杂任务"
```

### 认证配置文件

使用独立的认证配置文件（不提交到版本控制）：

**openclaw.json**（可提交）：
```json
{
  "models": {
    "providers": {
      "ldx": {
        "api": "openai-completions",
        "baseUrl": "https://api.liandanxia.com/v1",
        "models": [
          {"id": "gpt-4o-mini"}
        ]
      }
    }
  }
}
```

**openclaw.local.json**（不提交）：
```json
{
  "models": {
    "providers": {
      "ldx": {
        "apiKey": "sk-你的_api_key"
      }
    }
  }
}
```

### 环境变量支持

OpenClaw 也支持环境变量：

```bash
# 设置 API Key
export OPENCLAW_LDX_API_KEY="sk-你的_api_key"

# 或使用标准环境变量
export ANTHROPIC_API_KEY="sk-你的_api_key"
export OPENAI_API_KEY="sk-你的_api_key"
```

### 使用配置文件 Profile

为不同环境创建独立配置：

```bash
# 开发环境
export OPENCLAW_CONFIG_PATH=~/.openclaw/dev/openclaw.json
openclaw run "测试"

# 生产环境
export OPENCLAW_CONFIG_PATH=~/.openclaw/prod/openclaw.json
openclaw run "生产任务"

# 或使用 --profile 标志
openclaw --profile dev run "测试"
openclaw --profile prod run "生产任务"
```

## 常见问题与排错

### 1. 认证失败 (401)

**原因**：`apiKey` 无效

**解决**：
```bash
# 检查配置
openclaw config get models.providers.ldx.apiKey

# 重新设置
openclaw config set models.providers.ldx.apiKey "sk-你的_api_key"
```

### 2. 端点未找到 (404)

**原因**：`baseUrl` 错误或缺少 `/v1`

**解决**：
```bash
# 正确配置（必须包含 /v1）
openclaw config set models.providers.ldx.baseUrl "https://api.liandanxia.com/v1"

# 错误配置
# openclaw config set models.providers.ldx.baseUrl "https://api.liandanxia.com"  ❌
```

### 3. 模型不可用 (400)

**原因**：模型名或配置字段错误

**解决**：
```bash
# 先查询可用模型
curl https://api.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"

# 更新模型列表
openclaw config set models.providers.ldx.models '[{"id":"gpt-4o-mini"}]'
```

### 4. API 类型错误

**原因**：`api` 字段配置错误

**解决**：
```bash
# 对于 OpenAI 兼容接口，必须使用
openclaw config set models.providers.ldx.api "openai-completions"

# 不要使用
# openclaw config set models.providers.ldx.api "openai"  ❌
```

### 5. 模型引用格式错误

**原因**：模型引用格式不正确

**解决**：
```bash
# 正确格式：provider/model
openclaw config set agents.defaults.model.primary "ldx/gpt-4o-mini"

# 错误格式
# openclaw config set agents.defaults.model.primary "gpt-4o-mini"  ❌
```

### 6. 限流错误 (429)

**原因**：请求频率超限

**解决**：
- 配置重试策略
- 降低并发请求
- 使用 fallback 模型

## 排查顺序建议

1. **验证 baseUrl** → 确保包含 `/v1` 后缀
2. **验证 api 类型** → 使用 `openai-completions`
3. **验证模型映射** → 使用 `openclaw models list` 检查
4. **验证 API Key** → 使用 `openclaw config get` 检查

```bash
# 完整排查流程
openclaw config get models.providers.ldx
openclaw models list
openclaw models status
openclaw run "测试"
```

## 安全与最佳实践

### Provider 命名

✅ **推荐**：使用短且稳定的名称
```bash
openclaw config set models.providers.ldx.api "openai-completions"
```

❌ **不推荐**：使用过长或易变的名称
```bash
# openclaw config set models.providers.liandanxia-production.api "..."  ❌
```

### 模型验证流程

在设置默认模型前，先验证可用性：

```bash
# 1. 配置 Provider
openclaw config set models.providers.ldx.baseUrl "https://api.liandanxia.com/v1"
openclaw config set models.providers.ldx.apiKey "sk-你的_api_key"
openclaw config set models.providers.ldx.models '[{"id":"gpt-4o-mini"}]'

# 2. 验证模型
openclaw models list
openclaw models status

# 3. 设置默认模型
openclaw config set agents.defaults.model.primary "ldx/gpt-4o-mini"
```

### 团队协作配置

**共享配置**（提交到版本控制）：
```json
{
  "models": {
    "providers": {
      "ldx": {
        "api": "openai-completions",
        "baseUrl": "https://api.liandanxia.com/v1",
        "models": [
          {"id": "gpt-4o-mini"},
          {"id": "gpt-4o"}
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "ldx/gpt-4o-mini"
      }
    }
  }
}
```

**个人密钥**（不提交）：
```bash
# 每个团队成员在本地设置
openclaw config set models.providers.ldx.apiKey "sk-个人_api_key"
```

### 配置备份

```bash
# 备份配置
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup

# 恢复配置
cp ~/.openclaw/openclaw.json.backup ~/.openclaw/openclaw.json
```

## 参考资源

- [OpenClaw 官方文档](https://docs.openclaw.ai/)
- [OpenClaw 模型 Providers](https://docs.openclaw.ai/concepts/model-providers)
- [OpenClaw 配置指南](https://docs.openclaw.ai/configuration)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
