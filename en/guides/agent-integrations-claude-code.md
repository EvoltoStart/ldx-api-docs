---
title: "Integrate Claude Code"
description: "Practical Claude Code integration guide for LDX API: install, configure, verify, and troubleshoot."
---

This guide is for users who work in terminal-first development workflows and want to route Claude Code traffic through the LDX gateway.

## When to choose Claude Code

- You mostly work from CLI and local repositories
- You prefer Anthropic-style message APIs
- You want minimal migration effort to LDX gateway routing
- You need MCP (Model Context Protocol) integration and AI Agent capabilities

## Prerequisites

- Claude Code installed (refer to [Claude Code Official Installation Guide](https://code.claude.com/docs/en/overview))
- A valid API key: `sk-...`
- Network access to `https://api.liandanxia.io`
- `curl` installed for connectivity verification

## Configure LDX Gateway

Claude Code supports configuration via environment variables or settings files.

### Method 1: Environment Variables (recommended for testing)

#### Linux / macOS / WSL

```bash
export ANTHROPIC_BASE_URL=https://api.liandanxia.io
export ANTHROPIC_AUTH_TOKEN=sk-your_api_key
export ANTHROPIC_MODEL=gpt-4o-mini
```

#### Windows PowerShell

```powershell
$env:ANTHROPIC_BASE_URL="https://api.liandanxia.io"
$env:ANTHROPIC_AUTH_TOKEN="sk-your_api_key"
$env:ANTHROPIC_MODEL="gpt-4o-mini"
```

### Method 2: Settings File (recommended for persistence)

Create or edit `~/.claude/settings.json`:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.liandanxia.io",
    "ANTHROPIC_AUTH_TOKEN": "sk-your_api_key",
    "ANTHROPIC_MODEL": "gpt-4o-mini"
  }
}
```

**Settings file precedence**:
- `~/.claude/settings.json` — Global config, applies to all projects
- `.claude/settings.json` — Project config, can be committed to version control
- `.claude/settings.local.json` — Project-local config, not committed

### Important configuration notes

| Environment Variable | Description | Required |
|---------------------|-------------|----------|
| `ANTHROPIC_BASE_URL` | LDX gateway address, **do not** add `/v1` suffix | Yes |
| `ANTHROPIC_AUTH_TOKEN` | Your API Key, sent as `Authorization: Bearer` | Yes |
| `ANTHROPIC_MODEL` | Default model name from `/v1/models` | Recommended |
| `ANTHROPIC_API_KEY` | Alternative to `AUTH_TOKEN`, sent as `x-api-key` | Optional |

## Verify Gateway Connectivity

### 1. Verify model list

```bash
curl https://api.liandanxia.io/v1/models \
  -H "Authorization: Bearer sk-your_api_key"
```

### 2. Verify Anthropic-compatible endpoint

```bash
curl https://api.liandanxia.io/v1/messages \
  -H "x-api-key: sk-your_api_key" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model":"gpt-4o-mini",
    "max_tokens":256,
    "messages":[{"role":"user","content":"hello"}]
  }'
```

### 3. Launch Claude Code

```bash
claude
```

Test with a prompt in the interactive interface:

```
Hello, please introduce yourself
```

### Success criteria

- ✅ `/v1/models` returns model list
- ✅ `/v1/messages` returns valid JSON response
- ✅ `claude` interactive interface works normally

## Advanced Configuration

### Enable gateway model discovery

If your LDX gateway supports the `/v1/models` endpoint, enable automatic model discovery:

```bash
export CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1
```

Or in `settings.json`:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.liandanxia.io",
    "ANTHROPIC_AUTH_TOKEN": "sk-your_api_key",
    "CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY": "1"
  }
}
```

### Custom request headers

Add custom headers (e.g., for internal gateway authentication):

```bash
export ANTHROPIC_CUSTOM_HEADERS="X-Custom-Auth: token123
X-Team-ID: engineering"
```

### Configure timeout and retries

```json
{
  "env": {
    "API_TIMEOUT_MS": "1200000",
    "CLAUDE_CODE_MAX_RETRIES": "5"
  }
}
```

## Troubleshooting

### 1. Authentication failure (401 Unauthorized)

**Causes**:
- Invalid or expired API Key
- Extra spaces in the key
- Wrong authentication header

**Solution**:
```bash
# Check if key is correct
echo $ANTHROPIC_AUTH_TOKEN

# Reset (remove spaces)
export ANTHROPIC_AUTH_TOKEN="sk-your_api_key"
```

### 2. Endpoint not found (404 Not Found)

**Causes**:
- Incorrect `ANTHROPIC_BASE_URL`
- Incorrectly added `/v1` suffix

**Solution**:
```bash
# Correct configuration (no /v1)
export ANTHROPIC_BASE_URL=https://api.liandanxia.io

# Wrong configuration
# export ANTHROPIC_BASE_URL=https://api.liandanxia.io/v1  ❌
```

### 3. Model unavailable (400 Bad Request)

**Causes**:
- Model name not in available list
- Request parameters don't match interface requirements

**Solution**:
```bash
# Query available models first
curl https://api.liandanxia.io/v1/models \
  -H "Authorization: Bearer sk-your_api_key"

# Use model name from returned list
export ANTHROPIC_MODEL=gpt-4o-mini
```

### 4. Rate limit error (429 Too Many Requests)

**Causes**:
- Request frequency exceeds limit
- Too many concurrent requests

**Solution**:
- Reduce request frequency
- Implement exponential backoff retry strategy
- Contact admin to increase quota

### 5. Windows installation issues

**Problem**: Claude Code doesn't support native Windows

**Solution**:
1. Install WSL2:
   ```powershell
   wsl --install
   ```
2. Or install Git Bash and configure:
   ```bash
   export CLAUDE_CODE_GIT_BASH_PATH="/c/Program Files/Git/bin/bash.exe"
   ```

### 6. Gateway-specific issues

If your gateway doesn't support certain Anthropic features, disable them:

```bash
# Disable experimental beta features
export CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1

# Disable tool search (if gateway doesn't support tool_reference)
export ENABLE_TOOL_SEARCH=false
```

## Recommended troubleshooting order

1. **Verify network connectivity** → Test Base URL with `curl`
2. **Verify API Key** → Check key format and permissions
3. **Verify model name** → Confirm model is in available list
4. **Check request format** → Compare with official API docs
5. **View debug logs** → Enable `--debug` mode

```bash
# Enable debug mode
claude --debug

# Or set environment variable
export DEBUG=1
```

## Security and Best Practices

### Key management

❌ **Don't do this**:
```bash
# Don't hardcode keys in scripts
export ANTHROPIC_AUTH_TOKEN=sk-real-key-here
```

✅ **Recommended approach**:
```bash
# Use secret management tools
export ANTHROPIC_AUTH_TOKEN=$(vault kv get -field=api_key secret/ldx/claude-code)

# Or use apiKeyHelper
```

Configure dynamic key retrieval in `settings.json`:

```json
{
  "apiKeyHelper": "~/bin/get-ldx-key.sh",
  "env": {
    "CLAUDE_CODE_API_KEY_HELPER_TTL_MS": "3600000"
  }
}
```

### Model selection strategy

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

### Project-level configuration

Create `.claude/settings.json` in project root:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.liandanxia.io",
    "ANTHROPIC_MODEL": "gpt-4o-mini"
  },
  "contextPaths": [
    ".github/copilot-instructions.md",
    "CLAUDE.md"
  ]
}
```

**Note**: Don't commit configuration files containing real API Keys to version control.

## Reference Resources

- [Claude Code Official Docs](https://code.claude.com/docs/en/overview)
- [Environment Variables Reference](https://claude-code.mintlify.app/en/env-vars)
- [LLM Gateway Configuration](https://code.claude.com/docs/en/llm-gateway)
- [Anthropic Messages API](https://docs.anthropic.com/en/api/messages)
