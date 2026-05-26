---
title: "Integrate OpenClaw"
description: "Practical OpenClaw integration guide for LDX API: install, provider registration, verification, and troubleshooting."
---

This guide is for users who manage multiple providers via OpenClaw and want a clean default-model workflow through LDX.

## When to choose OpenClaw

- You need explicit multi-provider model management
- You want CLI visibility into model status
- You want stable default-model policy in tool config
- You need 24/7 autonomous AI Agent capabilities

## Prerequisites

- OpenClaw installed (refer to [OpenClaw Official Installation Guide](https://docs.openclaw.ai/))
- A valid API key: `sk-...`
- Network access to `https://api.liandanxia.io`
- `curl` installed

## Register LDX Provider

OpenClaw manages providers via CLI commands or configuration files.

### Method 1: Using CLI commands (recommended)

```bash
# Set provider type to OpenAI-compatible
openclaw config set models.providers.ldx.api "openai-completions"

# Set Base URL
openclaw config set models.providers.ldx.baseUrl "https://api.liandanxia.io/v1"

# Set API Key
openclaw config set models.providers.ldx.apiKey "sk-your_api_key"

# Configure available model list
openclaw config set models.providers.ldx.models '[{"id":"gpt-4o-mini"},{"id":"gpt-4o"},{"id":"claude-sonnet-4-20250514"}]'

# Set default primary model
openclaw config set agents.defaults.model.primary "ldx/gpt-4o-mini"
```

### Method 2: Direct config file editing

Config file location: `~/.openclaw/openclaw.json`

```json
{
  "models": {
    "providers": {
      "ldx": {
        "api": "openai-completions",
        "baseUrl": "https://api.liandanxia.io/v1",
        "apiKey": "sk-your_api_key",
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

### Configuration details

| Config Item | Description | Required |
|------------|-------------|----------|
| `api` | API type, use `openai-completions` | Yes |
| `baseUrl` | LDX gateway address, **must** include `/v1` | Yes |
| `apiKey` | Your API Key | Yes |
| `models` | Available model list | Yes |
| `primary` | Default primary model, format `provider/model` | Recommended |
| `fallback` | Backup model, used when primary fails | Optional |

### Verify configuration

```bash
# View all configuration
openclaw config list

# View provider configuration
openclaw config get models.providers.ldx
```

## Verify Model Availability

### 1. List available models

```bash
openclaw models list
```

Should see output similar to:

```
Available models:
  ldx/gpt-4o-mini
  ldx/gpt-4o
  ldx/claude-sonnet-4-20250514
```

### 2. Check model status

```bash
openclaw models status
```

### 3. Test conversation

```bash
openclaw run "Hello, please introduce yourself"
```

### Success criteria

- ✅ `models list` shows target models
- ✅ `models status` shows no critical errors
- ✅ `run` command returns valid response

## Advanced Configuration

### Multi-provider configuration

OpenClaw supports configuring multiple providers simultaneously:

```json
{
  "models": {
    "providers": {
      "ldx": {
        "api": "openai-completions",
        "baseUrl": "https://api.liandanxia.io/v1",
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

### Model aliases

Create aliases for frequently used models:

```bash
# Create aliases
openclaw config set models.aliases.fast "ldx/gpt-4o-mini"
openclaw config set models.aliases.smart "ldx/gpt-4o"
openclaw config set models.aliases.claude "ldx/claude-sonnet-4-20250514"

# Use aliases
openclaw run --model fast "Quick task"
openclaw run --model smart "Complex task"
```

### Authentication config file

Use separate authentication config file (not committed to version control):

**openclaw.json** (can be committed):
```json
{
  "models": {
    "providers": {
      "ldx": {
        "api": "openai-completions",
        "baseUrl": "https://api.liandanxia.io/v1",
        "models": [
          {"id": "gpt-4o-mini"}
        ]
      }
    }
  }
}
```

**openclaw.local.json** (not committed):
```json
{
  "models": {
    "providers": {
      "ldx": {
        "apiKey": "sk-your_api_key"
      }
    }
  }
}
```

### Environment variable support

OpenClaw also supports environment variables:

```bash
# Set API Key
export OPENCLAW_LDX_API_KEY="sk-your_api_key"

# Or use standard environment variables
export ANTHROPIC_API_KEY="sk-your_api_key"
export OPENAI_API_KEY="sk-your_api_key"
```

### Using config profiles

Create separate configs for different environments:

```bash
# Development environment
export OPENCLAW_CONFIG_PATH=~/.openclaw/dev/openclaw.json
openclaw run "Test"

# Production environment
export OPENCLAW_CONFIG_PATH=~/.openclaw/prod/openclaw.json
openclaw run "Production task"

# Or use --profile flag
openclaw --profile dev run "Test"
openclaw --profile prod run "Production task"
```

## Troubleshooting

### 1. Authentication failure (401)

**Cause**: Invalid `apiKey`

**Solution**:
```bash
# Check configuration
openclaw config get models.providers.ldx.apiKey

# Reset
openclaw config set models.providers.ldx.apiKey "sk-your_api_key"
```

### 2. Endpoint not found (404)

**Cause**: Incorrect `baseUrl` or missing `/v1`

**Solution**:
```bash
# Correct configuration (must include /v1)
openclaw config set models.providers.ldx.baseUrl "https://api.liandanxia.io/v1"

# Wrong configuration
# openclaw config set models.providers.ldx.baseUrl "https://api.liandanxia.io"  ❌
```

### 3. Model unavailable (400)

**Cause**: Model name or config field error

**Solution**:
```bash
# Query available models first
curl https://api.liandanxia.io/v1/models \
  -H "Authorization: Bearer sk-your_api_key"

# Update model list
openclaw config set models.providers.ldx.models '[{"id":"gpt-4o-mini"}]'
```

### 4. API type error

**Cause**: Incorrect `api` field configuration

**Solution**:
```bash
# For OpenAI-compatible interfaces, must use
openclaw config set models.providers.ldx.api "openai-completions"

# Don't use
# openclaw config set models.providers.ldx.api "openai"  ❌
```

### 5. Model reference format error

**Cause**: Incorrect model reference format

**Solution**:
```bash
# Correct format: provider/model
openclaw config set agents.defaults.model.primary "ldx/gpt-4o-mini"

# Wrong format
# openclaw config set agents.defaults.model.primary "gpt-4o-mini"  ❌
```

### 6. Rate limit error (429)

**Cause**: Request frequency exceeds limit

**Solution**:
- Configure retry strategy
- Reduce concurrent requests
- Use fallback model

## Recommended troubleshooting order

1. **Verify baseUrl** → Ensure `/v1` suffix is included
2. **Verify api type** → Use `openai-completions`
3. **Verify model mapping** → Check with `openclaw models list`
4. **Verify API Key** → Check with `openclaw config get`

```bash
# Complete troubleshooting workflow
openclaw config get models.providers.ldx
openclaw models list
openclaw models status
openclaw run "Test"
```

## Security and Best Practices

### Provider naming

✅ **Recommended**: Use short, stable names
```bash
openclaw config set models.providers.ldx.api "openai-completions"
```

❌ **Not recommended**: Use overly long or variable names
```bash
# openclaw config set models.providers.liandanxia-production.api "..."  ❌
```

### Model verification workflow

Verify availability before setting default model:

```bash
# 1. Configure provider
openclaw config set models.providers.ldx.baseUrl "https://api.liandanxia.io/v1"
openclaw config set models.providers.ldx.apiKey "sk-your_api_key"
openclaw config set models.providers.ldx.models '[{"id":"gpt-4o-mini"}]'

# 2. Verify model
openclaw models list
openclaw models status

# 3. Set default model
openclaw config set agents.defaults.model.primary "ldx/gpt-4o-mini"
```

### Team collaboration config

**Shared config** (committed to version control):
```json
{
  "models": {
    "providers": {
      "ldx": {
        "api": "openai-completions",
        "baseUrl": "https://api.liandanxia.io/v1",
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

**Personal keys** (not committed):
```bash
# Each team member sets locally
openclaw config set models.providers.ldx.apiKey "sk-personal_api_key"
```

### Configuration backup

```bash
# Backup configuration
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup

# Restore configuration
cp ~/.openclaw/openclaw.json.backup ~/.openclaw/openclaw.json
```

## Reference Resources

- [OpenClaw Official Docs](https://docs.openclaw.ai/)
- [OpenClaw Model Providers](https://docs.openclaw.ai/concepts/model-providers)
- [OpenClaw Configuration Guide](https://docs.openclaw.ai/configuration)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
