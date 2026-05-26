---
title: "接入 OpenCode"
description: "面向实际使用的 OpenCode 接入指南：安装、Provider 配置、验证与排错。"
---

这份指南适合希望在项目内使用 `.opencode.json` 管理模型配置，并通过 LDX 统一网关调用 OpenAI 兼容接口的用户。

## 什么时候选 OpenCode

- 你希望以"项目配置文件"方式管理模型
- 你需要不同仓库有不同默认模型
- 你需要 OpenAI 兼容接口的稳定接入路径
- 你希望支持 75+ AI 提供商的统一接入

## 前置条件

- 已安装 OpenCode（请参考 [OpenCode 官方安装文档](https://opencode.ai/docs)）
- 已准备可用 API Key：`sk-...`
- 本机可访问：`https://api.liandanxia.com`
- 已安装 `curl`

## 配置 Provider

OpenCode 使用 JSON 配置文件管理 AI 提供商。配置文件位置：

- **全局配置**：`~/.opencode.json`（所有项目生效）
- **项目配置**：项目根目录 `.opencode.json`（仅当前项目）

### 创建项目配置文件

在项目根目录创建 `.opencode.json`：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "providers": {
    "ldx": {
      "apiKey": "sk-你的_api_key",
      "disabled": false
    }
  },
  "agents": {
    "coder": {
      "model": "ldx/gpt-4o-mini",
      "maxTokens": 50000
    },
    "task": {
      "model": "ldx/gpt-4o-mini",
      "maxTokens": 5000
    }
  }
}
```

### 配置说明

#### 1. Provider 配置

OpenCode 内置了对多个 AI 提供商的支持，包括：

- `anthropic` - Anthropic API (Claude 模型)
- `openai` - OpenAI API (GPT 模型)
- `gemini` - Google Gemini API
- `groq` - Groq API
- `azure` - Azure OpenAI Service
- `bedrock` - AWS Bedrock
- `vertexai` - Google Cloud Vertex AI
- `openrouter` - OpenRouter (多提供商代理)

**LDX 作为自定义 Provider**：

由于 LDX 提供 OpenAI 兼容接口，你可以使用 `openai` provider 并覆盖 base URL：

```json
{
  "providers": {
    "openai": {
      "apiKey": "sk-你的_api_key",
      "baseUrl": "https://api.liandanxia.com/v1",
      "disabled": false
    }
  }
}
```

或者创建自定义 provider（推荐）：

```json
{
  "providers": {
    "ldx": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LDX",
      "models": {
        "gpt-4o-mini": {},
        "gpt-4o": {},
        "claude-sonnet-4-20250514": {}
      },
      "options": {
        "baseURL": "https://api.liandanxia.com/v1",
        "apiKey": "sk-你的_api_key"
      }
    }
  }
}
```

#### 2. Agent 配置

OpenCode 支持三种 Agent 类型：

- **coder** - 主要编码 Agent，处理复杂任务
- **task** - 任务规划 Agent
- **title** - 会话标题生成 Agent

```json
{
  "agents": {
    "coder": {
      "model": "ldx/gpt-4o",
      "maxTokens": 50000,
      "reasoningEffort": "medium"
    },
    "task": {
      "model": "ldx/gpt-4o-mini",
      "maxTokens": 5000
    },
    "title": {
      "model": "ldx/gpt-4o-mini",
      "maxTokens": 80
    }
  }
}
```

#### 3. 模型引用格式

模型引用使用 `provider/model` 格式：

```
ldx/gpt-4o-mini
ldx/gpt-4o
ldx/claude-sonnet-4-20250514
```

## 验证配置

### 1. 验证网关请求

```bash
curl https://api.liandanxia.com/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model":"gpt-4o-mini",
    "messages":[{"role":"user","content":"hello"}]
  }'
```

### 2. 启动 OpenCode

```bash
opencode
```

### 3. 测试对话

在 OpenCode 界面中输入：

```
你好，请介绍一下你自己
```

### 验证通过标准

- ✅ `chat/completions` 返回有效 JSON
- ✅ OpenCode 能识别并加载 `ldx/...` 模型
- ✅ 首条交互请求返回正常

## 高级配置

### 完整配置示例

```json
{
  "$schema": "https://opencode.json",
  "data": {
    "directory": ".opencode"
  },
  "wd": "/path/to/working/directory",
  "agents": {
    "coder": {
      "model": "ldx/gpt-4o",
      "maxTokens": 50000,
      "reasoningEffort": "medium"
    },
    "task": {
      "model": "ldx/gpt-4o-mini",
      "maxTokens": 5000
    },
    "title": {
      "model": "ldx/gpt-4o-mini",
      "maxTokens": 80
    }
  },
  "providers": {
    "ldx": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LDX",
      "models": {
        "gpt-4o-mini": {},
        "gpt-4o": {},
        "claude-sonnet-4-20250514": {}
      },
      "options": {
        "baseURL": "https://api.liandanxia.com/v1",
        "apiKey": "sk-你的_api_key"
      }
    }
  },
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/files"],
      "type": "stdio"
    }
  },
  "lsp": {
    "typescript": {
      "command": "typescript-language-server",
      "args": ["--stdio"],
      "disabled": false
    }
  },
  "contextPaths": [
    ".github/copilot-instructions.md",
    ".cursorrules",
    "opencode.md"
  ],
  "tui": {
    "theme": "opencode"
  },
  "debug": false,
  "autoCompact": true
}
```

### 环境变量支持

OpenCode 也支持通过环境变量配置 API Key：

```bash
# 设置 LDX API Key（如果使用 openai provider）
export OPENAI_API_KEY="sk-你的_api_key"

# 或者使用自定义环境变量
export LDX_API_KEY="sk-你的_api_key"
```

然后在配置文件中引用：

```json
{
  "providers": {
    "ldx": {
      "options": {
        "baseURL": "https://api.liandanxia.com/v1",
        "apiKey": "${LDX_API_KEY}"
      }
    }
  }
}
```

### MCP 服务器集成

OpenCode 支持 Model Context Protocol (MCP) 服务器：

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/files"],
      "type": "stdio"
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": ["GITHUB_PERSONAL_ACCESS_TOKEN=ghp_..."],
      "type": "stdio"
    }
  }
}
```

### LSP 语言服务器

配置代码智能支持：

```json
{
  "lsp": {
    "typescript": {
      "command": "typescript-language-server",
      "args": ["--stdio"]
    },
    "python": {
      "command": "pylsp",
      "args": []
    },
    "go": {
      "command": "gopls",
      "args": ["serve"]
    }
  }
}
```

## 常见问题与排错

### 1. 认证失败 (401)

**原因**：`apiKey` 无效或格式错误

**解决**：
```bash
# 检查配置文件中的 apiKey
cat .opencode.json | grep apiKey

# 确保 Key 格式正确
"apiKey": "sk-你的_api_key"
```

### 2. 端点未找到 (404)

**原因**：`baseURL` 配置错误或缺少 `/v1`

**解决**：
```json
{
  "providers": {
    "ldx": {
      "options": {
        "baseURL": "https://api.liandanxia.com/v1"
      }
    }
  }
}
```

**注意**：OpenCode 的 `baseURL` **必须**包含 `/v1` 后缀。

### 3. 模型不可用 (400)

**原因**：模型名或映射错误

**解决**：
```bash
# 先查询可用模型
curl https://api.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"

# 在配置中使用正确的模型名
{
  "agents": {
    "coder": {
      "model": "ldx/gpt-4o-mini"
    }
  }
}
```

### 4. 配置文件语法错误

**原因**：JSON 格式错误

**解决**：
```bash
# 使用 jq 验证 JSON 格式
cat .opencode.json | jq .

# 或使用在线 JSON 验证工具
```

### 5. 限流错误 (429)

**原因**：请求过快，触发限流

**解决**：
- 降低请求频率
- 实现重试机制
- 联系管理员提升限额

## 排查顺序建议

1. **验证 JSON 语法** → 使用 `jq` 或 JSON 验证工具
2. **验证 baseURL** → 确保包含 `/v1` 后缀
3. **验证模型映射** → 确认模型名在可用列表中
4. **验证 API Key** → 检查 Key 权限和有效性

## 安全与最佳实践

### 配置文件管理

**项目配置结构**（可提交）：
```json
{
  "$schema": "https://opencode.ai/config.json",
  "providers": {
    "ldx": {
      "options": {
        "baseURL": "https://api.liandanxia.com/v1",
        "apiKey": "${LDX_API_KEY}"
      }
    }
  },
  "agents": {
    "coder": {
      "model": "ldx/gpt-4o-mini"
    }
  }
}
```

**本地密钥文件**（不提交）：
创建 `.env` 文件：
```bash
LDX_API_KEY=sk-你的_api_key
```

添加到 `.gitignore`：
```
.env
.opencode.local.json
```

### 多环境配置

**开发环境** (`.opencode.json`)：
```json
{
  "agents": {
    "coder": {
      "model": "ldx/gpt-4o-mini"
    }
  }
}
```

**生产环境** (`.opencode.prod.json`)：
```json
{
  "agents": {
    "coder": {
      "model": "ldx/gpt-4o"
    }
  }
}
```

使用时指定配置文件：
```bash
opencode --config .opencode.prod.json
```

## 参考资源

- [OpenCode 官方文档](https://opencode.ai/docs)
- [OpenCode 配置参考](https://opencode-ai-opencode.mintlify.app/core-concepts/configuration)
- [OpenCode Providers](https://opencode.ai/docs/providers/)
- [OpenCode GitHub](https://github.com/opencode-ai/opencode)
