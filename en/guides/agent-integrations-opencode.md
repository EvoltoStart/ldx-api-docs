---
title: "Integrate OpenCode"
description: "Configure OpenCode with an LDX OpenAI-compatible provider."
---

Use this guide to route OpenCode model calls through LDX API. The LDX OpenAI-compatible Base URL is `https://api.liandanxia.io/silver/v1`.

## When To Use This

- You want project-level model configuration.
- You want to use LDX through an OpenAI-compatible endpoint.
- You want different repositories to use different models or API keys.

## Prerequisites

- OpenCode is installed.
- You have an LDX API key: `sk-...`.
- Your machine can reach `https://api.liandanxia.io/silver`.
- The example model `qwen3.5-flash` appears in the current pricing docs. Use `GET /v1/models` as the source of truth for your account.

## Verify LDX First

```bash
curl https://api.liandanxia.io/silver/v1/models \
  -H "Authorization: Bearer sk-your_api_key"
```

```bash
curl https://api.liandanxia.io/silver/v1/chat/completions \
  -H "Authorization: Bearer sk-your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "messages": [{"role": "user", "content": "hello"}],
    "stream": false
  }'
```

## Configure OpenCode

OpenCode uses the `provider` config key, not `providers`. Create or update `opencode.json` in your project root:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "ldx/qwen3.5-flash",
  "provider": {
    "ldx": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LDX",
      "options": {
        "baseURL": "https://api.liandanxia.io/silver/v1",
        "apiKey": "sk-your_api_key"
      },
      "models": {
        "qwen3.5-flash": {},
        "qwen3.5-plus": {}
      }
    }
  }
}
```

To avoid storing the key in project config, reference an environment variable:

```bash
export LDX_API_KEY="sk-your_api_key"
```

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "ldx/qwen3.5-flash",
  "provider": {
    "ldx": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LDX",
      "options": {
        "baseURL": "https://api.liandanxia.io/silver/v1",
        "apiKey": "{env:LDX_API_KEY}"
      },
      "models": {
        "qwen3.5-flash": {},
        "qwen3.5-plus": {}
      }
    }
  }
}
```

## Start And Verify

```bash
opencode
```

In OpenCode, send:

```text
Please summarize the current project structure.
```

Success criteria:

- `curl /v1/models` returns a model list.
- `curl /v1/chat/completions` returns valid JSON.
- OpenCode can use `ldx/qwen3.5-flash` for the first reply.

## Troubleshooting

| Problem | Likely Cause | Fix |
| --- | --- | --- |
| Provider is not found | Config uses `providers` | Use the official `provider` key. |
| `401` | API key is wrong or env var is not visible | Set `LDX_API_KEY` again in the same terminal, or verify with curl. |
| `404` | Base URL is missing `/v1` | Use `https://api.liandanxia.io/silver/v1`. |
| Model unavailable | Model name is not enabled for your account | Call `/v1/models`, then copy an available model name. |

Next, see [First Request Example](/en/getting-started/first-request) for the underlying API format.

## References

- [OpenCode Providers](https://opencode.ai/docs/providers/)
- [OpenCode Config](https://opencode.ai/docs/config/)
- [OpenCode GitHub](https://github.com/sst/opencode)
- [First Request Example](/en/getting-started/first-request)
- [Error Codes](/en/getting-started/error-codes)
