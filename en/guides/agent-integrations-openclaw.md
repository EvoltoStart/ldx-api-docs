---
title: "Integrate OpenClaw"
description: "Configure OpenClaw custom model provider with LDX API."
---

Use this guide to connect OpenClaw model calls to LDX API. The LDX OpenAI-compatible Base URL is `https://api.liandanxia.io/v1`.

## When To Use This

- You need explicit multi-provider model management in OpenClaw.
- You want to inspect models with `/models` or `openclaw models list`.
- You want the default model to point to an LDX gateway model.

## Prerequisites

- OpenClaw is installed.
- You have an LDX API key: `sk-...`.
- Your machine can reach `https://api.liandanxia.io`.
- The example models `qwen3.5-flash` and `qwen3.5-plus` appear in the current pricing docs. Use `GET /v1/models` as the source of truth for your account.

## Verify LDX First

```bash
curl https://api.liandanxia.io/v1/models \
  -H "Authorization: Bearer sk-your_api_key"
```

```bash
curl https://api.liandanxia.io/v1/chat/completions \
  -H "Authorization: Bearer sk-your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "messages": [{"role": "user", "content": "hello"}],
    "stream": false
  }'
```

## Configure Provider

OpenClaw recommends `--strict-json --merge` when changing provider and model allowlist maps, so existing config is not overwritten accidentally.

```bash
openclaw config set models.providers.ldx '{
  "api": "openai-completions",
  "baseUrl": "https://api.liandanxia.io/v1",
  "apiKey": "sk-your_api_key",
  "models": [
    {"id": "qwen3.5-flash"},
    {"id": "qwen3.5-plus"}
  ]
}' --strict-json --merge
```

If your OpenClaw version does not support writing a JSON block from CLI, edit the OpenClaw config file and keep the same fields:

```json
{
  "models": {
    "providers": {
      "ldx": {
        "api": "openai-completions",
        "baseUrl": "https://api.liandanxia.io/v1",
        "apiKey": "sk-your_api_key",
        "models": [
          {"id": "qwen3.5-flash"},
          {"id": "qwen3.5-plus"}
        ]
      }
    }
  }
}
```

## Set Default Model

```bash
openclaw models set ldx/qwen3.5-flash
```

If model allowlists are enabled, add the models to `agents.defaults.models`:

```bash
openclaw config set agents.defaults.models '{
  "ldx/qwen3.5-flash": {},
  "ldx/qwen3.5-plus": {}
}' --strict-json --merge
```

## Verify

```bash
openclaw models list --provider ldx
openclaw models status
openclaw run "Please introduce the current project."
```

Success criteria:

- `openclaw models list --provider ldx` shows LDX models.
- `openclaw models status` shows `ldx/qwen3.5-flash` as the default model.
- `openclaw run` returns a normal response.

## Troubleshooting

| Problem | Likely Cause | Fix |
| --- | --- | --- |
| `Model is not allowed` | Allowlist is enabled but LDX model is not listed | Add the model with the `agents.defaults.models` merge command. |
| `401` | API key is wrong | Reset `models.providers.ldx.apiKey`. |
| `404` | Base URL is wrong | Use `https://api.liandanxia.io/v1`, not the root URL. |
| Empty model list | Provider was not saved or model IDs are wrong | Confirm `/v1/models`, then update `models.providers.ldx.models`. |

Next, see [Error Codes](/en/getting-started/error-codes) for API failure meanings.

## References

- [OpenClaw configuration and custom providers](https://docs.openclaw.ai/gateway/config-tools)
- [OpenClaw Models CLI](https://documentation.openclaw.ai/concepts/models)
- [First Request Example](/en/getting-started/first-request)
- [Models and Pricing](/en/getting-started/pricing)
- [Error Codes](/en/getting-started/error-codes)
