---
title: "接入 CC Switch"
description: "通过 CC Switch 图形界面为 Claude Code、Codex、OpenCode 等 AI 编程工具配置 LDX API。"
---

CC Switch 是一款跨平台桌面应用，可在图形界面中为 Claude Code、Codex、Gemini CLI、OpenCode、OpenClaw、Hermes 等工具添加、启用和切换 Provider。本文按 CC Switch 官方快速上手流程说明如何把 LDX 配成 Provider，并补充各工具需要填写的 LDX Endpoint。

<Warning>
不同工具使用的协议不同，LDX Endpoint 也不同：Claude Code 的 Anthropic Messages 配置使用根地址 `https://api.liandanxia.com`；OpenAI 兼容配置使用 `https://api.liandanxia.com/v1`。如果用 CC Switch 的通用 Provider，请只勾选 Endpoint 类型一致的工具；同时配置 Claude Code 和 OpenAI 兼容工具时，建议按工具分别添加 Provider。
</Warning>

## 适用场景

- 你希望在 CC Switch 中统一管理 LDX API Key 和模型配置。
- 你同时使用 Claude Code、Codex、OpenCode、OpenClaw、Hermes 等多个 AI 编程工具。
- 你希望通过图形界面切换 Provider，而不是手动编辑 JSON/TOML 配置文件。
- 你希望通过 CC Switch 的模型拉取功能获取 LDX 模型列表。

## 前置条件

- 已安装 CC Switch。
- 已安装目标 CLI 工具，并至少启动过一次，让工具生成默认配置目录。
- 已准备 LDX API Key：`sk-...`。
- 当前网络可访问 `https://api.liandanxia.com`。
- 示例模型 `qwen3.5-flash` 仅用于演示；实际可用模型以 `/v1/models` 返回为准。

## 安装 CC Switch

| 平台 | 安装方式 |
| --- | --- |
| Windows | 从 [GitHub Releases](https://github.com/farion1231/cc-switch/releases) 下载 `.msi` 安装包或便携版 `.zip`。 |
| macOS | 使用 `brew install --cask cc-switch`，或从 Releases 下载 `.dmg` 手动安装。 |
| Linux | 下载 `.deb`、`.rpm` 或 `.AppImage`；Arch 用户可用 `paru -S cc-switch-bin`。 |

安装后打开 CC Switch。界面顶部选择工具，点击右上角 **+** 添加新 Provider。

## 先验证 LDX API

在配置 CC Switch 前，建议先确认 API Key 和模型可用。

```bash
curl https://api.liandanxia.com/v1/models \
  -H "Authorization: Bearer sk-你的_api_key"
```

```bash
curl https://api.liandanxia.com/v1/chat/completions \
  -H "Authorization: Bearer sk-你的_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.5-flash",
    "messages": [{"role": "user", "content": "hello"}],
    "stream": false
  }'
```

## 快速配置流程

1. 在 CC Switch 界面顶部选择要配置的工具，例如 **Claude Code**、**Codex**、**OpenCode**。
2. 点击右上角 **+** 按钮。
3. 选择合适的预设：Claude Code 通常选择 **Custom**；OpenAI 兼容工具优先选择 **OpenAI Compatible** 或 **Custom**。
4. 填写 Provider 名称、API Key、Endpoint URL 和模型。
5. 如界面支持，点击模型输入框旁边的 **获取模型** 按钮，从 LDX `/v1/models` 拉取模型列表。
6. 点击 **添加**。
7. 在 Provider 卡片上点击 **启用**。
8. 按工具要求重启终端或重新打开 CLI，然后发送一条测试消息验证。

添加 Provider 时，重点填写 **API Key** 和 **请求地址**；不同工具的请求地址请按下方表格选择。

![CC Switch 添加新供应商界面](/images/ccswitch_zh.png)

## LDX 参数填写表

| 工具 | CC Switch 工具 | 建议预设 | Endpoint URL | 说明 |
| --- | --- | --- | --- | --- |
| Claude Code | Claude Code | Custom | `https://api.liandanxia.com` | 使用 Anthropic Messages，填根地址，不加 `/v1`。 |
| Codex | Codex | Custom | `https://api.liandanxia.com/v1` | Codex 客户端仍请求 `/responses`；在 CC Switch 高级选项中开启本地路由，并将上游格式选择为 `chat/completions`。 |
| OpenCode | OpenCode | OpenAI Compatible 或 Custom | `https://api.liandanxia.com/v1` | 使用 OpenAI 兼容 Chat Completions。 |
| OpenClaw | OpenClaw | OpenAI Compatible 或 Custom | `https://api.liandanxia.com/v1` | 使用 OpenAI 兼容 Chat Completions。 |
| Hermes | Hermes | OpenAI Compatible 或 Custom | `https://api.liandanxia.com/v1` | OpenAI 兼容模式使用 `/v1` Base URL。 |

通用填写项：

| 配置项 | 值 |
| --- | --- |
| Name / 名称 | `LDX` |
| API Key | `sk-你的_api_key` |
| Model | `qwen3.5-flash`，或从 `/v1/models` 返回中选择可用模型 |

<Note>
不要把 Endpoint URL 填成完整接口路径，例如 `https://api.liandanxia.com/v1/chat/completions`。CC Switch 和目标工具会根据协议自动拼接具体路径。
</Note>

## Claude Code 配置要点

Claude Code 会在 `ANTHROPIC_BASE_URL` 后拼接 `/v1/messages`，因此 Endpoint URL 必须填写根地址：

```text
https://api.liandanxia.com
```

推荐配置：

| 配置项 | 值 | 说明 |
| --- | --- | --- |
| API Format | Anthropic Messages | 保持默认 Anthropic Messages。 |
| API Key | `sk-你的_api_key` | CC Switch 通常写入 `ANTHROPIC_API_KEY`；LDX 支持 `x-api-key` 认证。 |
| Endpoint URL | `https://api.liandanxia.com` | 填根地址，不加 `/v1`。 |
| Model | `qwen3.5-flash` | 以 `/v1/models` 返回为准。 |

如果你手动编辑 `~/.claude/settings.json`，常见配置形态如下：

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "sk-你的_api_key",
    "ANTHROPIC_BASE_URL": "https://api.liandanxia.com"
  }
}
```

也可以改用 `ANTHROPIC_AUTH_TOKEN`，让 Claude Code 以 `Authorization: Bearer` 方式发送认证信息。

## Codex 配置要点

Codex 客户端会请求 OpenAI Responses API。通过 CC Switch 接入 LDX 时，建议开启 CC Switch 本地路由，并在 Provider 高级选项中将上游格式选择为 `chat/completions`，由 CC Switch 把 Codex 的 `/responses` 请求转换为 LDX 的 OpenAI 兼容 Chat Completions 请求。

Endpoint URL 填写：

```text
https://api.liandanxia.com/v1
```

推荐配置：

| 配置项 | 值 | 说明 |
| --- | --- | --- |
| API Key | `sk-你的_api_key` | 作为 Codex 的 OpenAI API Key 使用。 |
| Endpoint URL | `https://api.liandanxia.com/v1` | CC Switch 本地路由会接收 Codex 的 `/responses` 请求。 |
| Model | `qwen3.5-flash` | 以 `/v1/models` 返回为准。 |
| Needs Local Routing | 开启 | 让 Codex 请求先进入 CC Switch 本地路由服务。 |
| Upstream Format / 上游格式 | `Chat Completions` | 必须选择 Chat Completions，避免直接走 `/v1/responses`。 |

<Warning>
不要在 CC Switch Codex Provider 中直接使用 `/v1/responses` 上游格式。若直接走 Responses，可能出现 `unexpected status 503 Service Unavailable: CC Switch local proxy failed while handling Codex endpoint /responses ... upstream_status: HTTP 503; cause: DistributorGetChannelFailed`。这种情况请开启本地路由，并把上游格式改为 `Chat Completions`。
</Warning>

### Codex 本地路由

Codex 接入 LDX 时推荐使用本地路由，适合这些场景：

- 让 CC Switch 把 Codex 的 `/responses` 请求转换为 LDX 支持更稳定的 `Chat Completions`。
- 你希望所有 Codex 请求经过 CC Switch 代理，并在 CC Switch 中查看路由日志。
- 你正在排查 Provider 切换和请求转发问题，需要观察代理链路。

配置步骤：

1. 进入 CC Switch **设置 → 高级 → 路由服务**。
2. 启动路由服务，默认监听 `127.0.0.1:15721`。
3. 在**应用路由**区域开启 **Codex** 路由开关。
   ![CC Switch 配置应用路由](/images/ccswitch_router_zh1.png)
4. 在 Codex 的 LDX Provider 高级选项中，将 Upstream Format / 上游格式 选择为 Chat Completions。
   ![CC Switch 选择上游格式](/images/ccswitch_router_zh2.png)
5. Endpoint URL 仍填写 `https://api.liandanxia.com/v1`。

转换为 Chat Completions 时，链路如下：

```text
Codex CLI
  -> http://127.0.0.1:15721/v1
  -> CC Switch 本地路由
  -> https://api.liandanxia.com/v1/chat/completions
```

## OpenCode / OpenClaw / Hermes 配置要点

这些工具使用 OpenAI 兼容 Chat Completions 时，Endpoint URL 填：

```text
https://api.liandanxia.com/v1
```

常用配置：

| 配置项 | 值 |
| --- | --- |
| Name / 名称 | `LDX` |
| API Key | `sk-你的_api_key` |
| Endpoint URL | `https://api.liandanxia.com/v1` |
| Model | `qwen3.5-flash` |

配置完成后，关闭并重新打开终端，再启动对应工具。

## 获取模型列表

CC Switch 支持从 Provider 的模型接口拉取模型列表。填写 API Key 和 Endpoint URL 后，点击模型输入框旁边的 **获取模型** 按钮即可。

常见失败原因：

| 错误 | 可能原因 | 处理方式 |
| --- | --- | --- |
| 401/403 | API Key 无效或含多余空格 | 重新粘贴 API Key，确认以 `sk-` 开头。 |
| 404/405 | Endpoint URL 填错 | Claude Code 填根地址；OpenAI 兼容工具填 `/v1` 地址。 |
| 超时 | 网络不可达或延迟过高 | 先用 `curl /v1/models` 验证网络，再重试。 |

## 生效与验证

Provider 切换后的生效方式因工具而异：

| 工具 | 生效方式 |
| --- | --- |
| Claude Code | 通常可热重载；关闭当前会话后重新运行 `claude` 更稳妥。 |
| Codex | 关闭并重新打开终端后运行 `codex`。 |
| OpenCode / OpenClaw | 关闭并重新打开终端后运行对应 CLI。 |
| Hermes | 重启终端后运行 `hermes`。 |

验证标准：

- CC Switch 中 LDX Provider 处于启用状态。
- 工具中的模型选择器或 `/model` 能看到 LDX 模型。
- 发送测试消息后能正常返回。
- 如果使用 Codex 本地路由，CC Switch 路由面板能看到代理请求日志。

## 常见问题

| 问题 | 可能原因 | 处理方式 |
| --- | --- | --- |
| 切换 Provider 后不生效 | 目标 CLI 还在使用旧进程或旧环境变量 | 关闭并重新打开终端，再启动工具。 |
| `401` 或 `403` | API Key 错误、过期或包含多余空格 | 重新编辑 Provider，确认 API Key 以 `sk-` 开头且无空格。 |
| `404` | Endpoint URL 类型填错 | Claude Code 填 `https://api.liandanxia.com`；OpenAI 兼容工具填 `https://api.liandanxia.com/v1`。 |
| Codex 提示需要路由服务 | Provider 开启了 Needs Local Routing，但路由服务未启动 | 启动 CC Switch 路由服务，并确认应用路由中已开启 Codex。 |
| Codex 本地代理连接失败 | 路由服务未运行、端口被占用或应用路由未开启 | 检查 **设置 → 高级 → 路由服务**，确认 `127.0.0.1:15721` 可用。 |
| Codex 请求 `/v1/responses` 返回 `unexpected status 503 Service Unavailable`，并包含 `CC Switch local proxy failed while handling Codex endpoint /responses`、`DistributorGetChannelFailed` | Codex Provider 直接使用了 Responses 上游格式，或没有把本地路由的上游格式切到 Chat Completions | 开启 **Needs Local Routing**，并在 Provider 高级选项中将 **Upstream Format / 上游格式** 改为 `Chat Completions`。 |
| 模型列表为空 | Endpoint URL 或 API Key 不正确 | 先用 curl 验证 `/v1/models` 是否能返回模型列表。 |
| Claude Code 非 Claude 模型行为异常 | Claude Code 附加了归属元数据 | 在 Provider 高级选项中开启 **隐藏署名**，或设置 `CLAUDE_CODE_ATTRIBUTION_HEADER=0`。 |

实际模型、上下文长度和价格请以 [模型与价格](/zh/getting-started/pricing) 为准。

## 参考资料

- [CC Switch 快速上手](https://ccswitch.io/zh/docs?section=getting-started&item=quickstart)
- [首个请求示例](/zh/getting-started/first-request)
- [模型与价格](/zh/getting-started/pricing)
- [错误码](/zh/getting-started/error-codes)
