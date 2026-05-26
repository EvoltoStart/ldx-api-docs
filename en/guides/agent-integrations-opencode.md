---
title: "Integrate OpenCode"
description: "Practical OpenCode integration guide for LDX API: install, provider config, verification, and troubleshooting."
---

This guide is for users who prefer project-level model configuration via `.opencode.json` and OpenAI-compatible routing through LDX.

## When to choose OpenCode

- You want model config tracked per project
- You need different defaults across repositories
- You want a stable OpenAI-compatible integration path
- You need unified access to 75+ AI providers

## Prerequisites

- OpenCode installed (refer to [OpenCode Official Installation Guide](https://opencode.ai/docs))
- A valid API key: `sk-...`
- Network access to `https://api.liandanxia.io`
- `curl` installed

## Configure Provider

OpenCode uses JSON configuration files to manage AI providers. Configuration file locations:

- **Global config**: `~/.opencode.json` (applies to all projects)
- **Project config**: `.opencode.json` in project root (current project only)

### Create project configuration file

Create `.opencode.json` in project root:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "providers": {
    "ldx": {
      "apiKey": "sk-your_api_key",
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

### Configuration details

#### 1. Provider configuration

OpenCode has built-in support for multiple AI providers:

- `anthropic` - Anthropic API (Claude models)
- `openai` - OpenAI API (GPT models)
- `gemini` - Google Gemini API
- `groq` - Groq API
- `azure` - Azure OpenAI Service
- `bedrock` - AWS Bedrock
- `vertexai` - Google Cloud Vertex AI
- `openrouter` - OpenRouter (multi-provider proxy)

**LDX as custom provider**:

Since LDX provides OpenAI-compatible interfaces, you can use the `openai` provider and override the base URL:

```json
{
  "providers": {
    "openai": {
      "apiKey": "sk-your_api_key",
      "baseUrl": "https://api.liandanxia.io/v1",
      "disabled": false
    }
  }
}
```

Or create a custom provider (recommended):

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
        "baseURL": "https://api.liandanxia.io/v1",
        "apiKey": "sk-your_api_key"
      }
    }
  }
}
```

#### 2. Agent configuration

OpenCode supports three agent types:

- **coder** - Main coding agent for complex tasks
- **task** - Task planning agent
- **title** - Session title generation agent

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

#### 3. Model reference format

Model references use `provider/model` format:

```
ldx/gpt-4o-mini
ldx/gpt-4o
ldx/claude-sonnet-4-20250514
```

## Verify Configuration

### 1. Verify gateway request

```bash
curl https://api.liandanxia.io/v1/chat/completions \
  -H "Authorization: Bearer sk-your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model":"gpt-4o-mini",
    "messages":[{"role":"user","content":"hello"}]
  }'
```

### 2. Launch OpenCode

```bash
opencode
```

### 3. Test conversation

In the OpenCode interface, enter:

```
Hello, please introduce yourself
```

### Success criteria

- ✅ `chat/completions` returns valid JSON
- ✅ OpenCode recognizes and loads `ldx/...` models
- ✅ First interactive request completes successfully

## Advanced Configuration

### Complete configuration example

```json
{
  "$schema": "https://opencode.ai/config.json",
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
        "baseURL": "https://api.liandanxia.io/v1",
        "apiKey": "sk-your_api_key"
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

### Environment variable support

OpenCode also supports API Key configuration via environment variables:

```bash
# Set LDX API Key (if using openai provider)
export OPENAI_API_KEY="sk-your_api_key"

# Or use custom environment variable
export LDX_API_KEY="sk-your_api_key"
```

Then reference in config file:

```json
{
  "providers": {
    "ldx": {
      "options": {
        "baseURL": "https://api.liandanxia.io/v1",
        "apiKey": "${LDX_API_KEY}"
      }
    }
  }
}
```

### MCP server integration

OpenCode supports Model Context Protocol (MCP) servers:

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

### LSP language servers

Configure code intelligence support:

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

## Troubleshooting

### 1. Authentication failure (401)

**Cause**: Invalid `apiKey` or format error

**Solution**:
```bash
# Check apiKey in config file
cat .opencode.json | grep apiKey

# Ensure correct key format
"apiKey": "sk-your_api_key"
```

### 2. Endpoint not found (404)

**Cause**: Incorrect `baseURL` or missing `/v1`

**Solution**:
```json
{
  "providers": {
    "ldx": {
      "options": {
        "baseURL": "https://api.liandanxia.io/v1"
      }
    }
  }
}
```

**Note**: OpenCode's `baseURL` **must** include the `/v1` suffix.

### 3. Model unavailable (400)

**Cause**: Model name or mapping error

**Solution**:
```bash
# Query available models first
curl https://api.liandanxia.io/v1/models \
  -H "Authorization: Bearer sk-your_api_key"

# Use correct model name in config
{
  "agents": {
    "coder": {
      "model": "ldx/gpt-4o-mini"
    }
  }
}
```

### 4. Config file syntax error

**Cause**: JSON format error

**Solution**:
```bash
# Validate JSON format with jq
cat .opencode.json | jq .

# Or use online JSON validator
```

### 5. Rate limit error (429)

**Cause**: Requests too fast, triggering rate limit

**Solution**:
- Reduce request frequency
- Implement retry mechanism
- Contact admin to increase quota

## Recommended troubleshooting order

1. **Verify JSON syntax** → Use `jq` or JSON validator
2. **Verify baseURL** → Ensure `/v1` suffix is included
3. **Verify model mapping** → Confirm model name is in available list
4. **Verify API Key** → Check key permissions and validity

## Security and Best Practices

### Configuration file management

**Project config structure** (can be committed):
```json
{
  "$schema": "https://opencode.ai/config.json",
  "providers": {
    "ldx": {
      "options": {
        "baseURL": "https://api.liandanxia.io/v1",
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

**Local key file** (not committed):
Create `.env` file:
```bash
LDX_API_KEY=sk-your_api_key
```

Add to `.gitignore`:
```
.env
.opencode.local.json
```

### Multi-environment configuration

**Development environment** (`.opencode.json`):
```json
{
  "agents": {
    "coder": {
      "model": "ldx/gpt-4o-mini"
    }
  }
}
```

**Production environment** (`.opencode.prod.json`):
```json
{
  "agents": {
    "coder": {
      "model": "ldx/gpt-4o"
    }
  }
}
```

Specify config file when using:
```bash
opencode --config .opencode.prod.json
```

## Reference Resources

- [OpenCode Official Docs](https://opencode.ai/docs)
- [OpenCode Configuration Reference](https://opencode-ai-opencode.mintlify.app/core-concepts/configuration)
- [OpenCode Providers](https://opencode.ai/docs/providers/)
- [OpenCode GitHub](https://github.com/opencode-ai/opencode)
