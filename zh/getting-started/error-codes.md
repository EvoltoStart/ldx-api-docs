---
title: "错误码"
description: "API 调用失败时的响应格式、HTTP 状态码、平台错误码和处理建议。"
---

本页说明模型调用和兼容入口常见的错误响应。排查问题时建议同时看 `HTTP status` 和 `error.code`：前者表示请求在协议层的失败类型，后者表示平台在代码中标记的具体失败原因。

## 错误响应格式

模型中转接口通常返回 OpenAI 兼容格式：

```json
{
  "error": {
    "message": "错误说明",
    "type": "ldx_api_error",
    "param": "",
    "code": "invalid_request"
  }
}
```

字段说明：

| 字段 | 说明 |
| --- | --- |
| `message` | 面向用户的错误说明，部分内部错误会被统一脱敏为“服务器繁忙”等提示。 |
| `type` | 错误来源或兼容协议类型，例如 `ldx_api_error`、`openai_error`、`claude_error`、`upstream_error`。 |
| `param` | 关联参数。部分接口不会返回该字段。 |
| `code` | 平台或上游返回的错误码。本文只列出项目代码中明确写入的本平台错误码。 |

<Note>
部分非模型管理接口使用顶层 `code`、`message`、`data`、`dateTime` 响应结构；模型中转、OpenAI 兼容、Claude 兼容和 Gemini 兼容入口优先参考本页的 `error` 结构。
</Note>

## HTTP 状态码

| HTTP 状态码 | 常见原因 | 处理建议 |
| --- | --- | --- |
| `400` | 请求体格式错误、参数不合法、模型不支持当前输入类型、指定渠道参数错误。 | 检查 JSON、必填字段、模型名、输入模态和接口路径。 |
| `401` | 未提供 API Key，或 API Key 无效、过期、额度已耗尽、格式不正确。 | 按 [认证](/zh/getting-started/authentication) 检查 `Authorization: Bearer sk-...` 或兼容请求头。 |
| `403` | 账号或 API 访问被禁用、IP 白名单不匹配、分组无权限、余额不足。 | 检查账号状态、API Key 权限、IP 限制、可用分组和账户余额。 |
| `404` | 查询的任务、会话、消息或资源不存在。 | 确认资源 ID 属于当前账号，并且任务已创建成功。 |
| `408` | 渠道响应超时。 | 可以稍后重试；如果持续出现，建议切换模型或联系支持排查渠道状态。 |
| `413` | 请求体过大或读取请求体失败。 | 缩小输入内容、文件或上下文长度后重试。 |
| `429` | 请求频率超过当前限流配置。 | 降低并发或请求频率，加入指数退避后重试。 |
| `500` | 平台内部处理失败、请求转换失败、上游响应解析失败、计费预扣失败。 | 不要立即高频重试；保留 `message` 和请求 ID 后联系支持。 |
| `502` | 代理或上游资源拉取失败。 | 稍后重试；如果是视频内容代理，确认任务已完成且结果地址有效。 |
| `503` | 当前模型或渠道不可用，或系统负载保护触发。 | 稍后重试，或切换到其他可用模型。 |

## 平台错误码

这些值来自项目代码中的 `types.ErrorCode` 定义，以及性能保护中写入的固定错误码。

### 请求与参数

| `error.code` | 含义 | 常见处理方式 |
| --- | --- | --- |
| `invalid_request` | 请求不合法，可能是参数缺失、格式错误或资源状态不满足要求。 | 检查接口路径、请求体、模型名和任务/会话 ID。 |
| `read_request_body_failed` | 读取请求体失败，或请求体超过限制。 | 减小请求体，确认客户端没有提前断开连接。 |
| `bad_request_body` | 请求体无法按目标渠道要求组装。 | 检查 JSON 结构、字段类型和多模态输入格式。 |
| `convert_request_failed` | 平台将请求转换为上游格式失败。 | 检查当前模型是否支持该接口和输入内容。 |
| `reasoning_disable_not_supported` | 当前模型不支持关闭 reasoning。 | 移除关闭 reasoning 的参数，或切换支持该能力的模型。 |

### 鉴权、权限与额度

| `error.code` | 含义 | 常见处理方式 |
| --- | --- | --- |
| `access_denied` | 请求被权限规则拒绝，例如 IP 白名单不匹配。 | 检查 API Key 的 IP 限制、分组权限和账号状态。 |
| `insufficient_user_quota` | 用户或 API Key 可用额度不足。 | 充值、提高配额或切换到成本更低的模型。 |
| `pre_consume_token_quota_failed` | 计费预扣额度失败。 | 检查余额和 API Key 状态；如余额充足仍失败，联系支持。 |

### 模型、渠道与路由

| `error.code` | 含义 | 常见处理方式 |
| --- | --- | --- |
| `model_not_found` | 当前模型没有可用渠道，或模型在当前分组不可用。 | 检查模型名、分组权限和模型可用状态。 |
| `get_channel_failed` | 获取可用渠道失败。 | 稍后重试；如果指定了渠道，检查渠道 ID 和渠道状态。 |
| `gen_relay_info_failed` | 生成中转请求信息失败。 | 检查请求路径、模型名和兼容协议。 |
| `invalid_api_type` | 当前请求无法匹配到有效 API 类型。 | 确认使用了文档中支持的接口路径。 |
| `model_price_error` | 获取或计算模型价格失败。 | 检查模型是否在价格表中配置；持续失败时联系支持。 |
| `channel:no_available_key` | 渠道没有可用上游密钥。 | 稍后重试或切换模型。 |
| `channel:param_override_invalid` | 渠道参数覆盖配置无效。 | 通常需要平台侧修复渠道配置。 |
| `channel:header_override_invalid` | 渠道请求头覆盖配置无效。 | 通常需要平台侧修复渠道配置。 |
| `channel:model_mapped_error` | 模型映射到上游模型失败。 | 检查模型名和渠道映射配置。 |
| `channel:aws_client_error` | AWS 渠道客户端创建或配置失败。 | 通常需要平台侧检查 AWS 渠道配置。 |
| `channel:invalid_key` | 上游渠道密钥无效。 | 通常需要平台侧更换或修复上游密钥。 |
| `channel:response_time_exceeded` | 渠道响应时间超过限制。 | 可以稍后重试，或切换到其他模型/渠道。 |

### 上游请求与响应

| `error.code` | 含义 | 常见处理方式 |
| --- | --- | --- |
| `do_request_failed` | 请求上游服务失败。 | 稍后重试；如果持续失败，联系支持并提供请求 ID。 |
| `read_response_body_failed` | 读取上游响应体失败。 | 稍后重试；避免客户端提前断开流式连接。 |
| `bad_response_status_code` | 上游返回了非预期 HTTP 状态码。 | 查看 `message` 中的上游状态；按状态决定是否重试。 |
| `bad_response` | 上游响应内容不符合预期。 | 稍后重试；持续失败时切换模型或联系支持。 |
| `bad_response_body` | 上游响应体解析失败。 | 稍后重试；持续失败时联系支持。 |
| `empty_response` | 上游返回空响应。 | 稍后重试。 |
| `aws_invoke_error` | AWS 模型调用失败。 | 按返回状态判断是否重试；持续失败时联系支持。 |
| `json_marshal_failed` | 平台组装 JSON 响应失败。 | 保留请求 ID 后联系支持。 |

### 内容安全

| `error.code` | 含义 | 常见处理方式 |
| --- | --- | --- |
| `sensitive_words_detected` | 输入命中敏感词或安全策略。 | 修改输入内容后重试。 |
| `prompt_blocked` | 提示词被上游或平台安全策略拦截。 | 删除违规内容后重试。 |
| `violation_fee.grok.csam` | Grok 相关违规费用场景标记。 | 立即停止相关请求并检查输入内容。 |

### 数据与系统

| `error.code` | 含义 | 常见处理方式 |
| --- | --- | --- |
| `count_token_failed` | 计算 token 失败。 | 缩短或简化输入内容后重试。 |
| `query_data_error` | 查询平台数据失败。 | 稍后重试；持续失败时联系支持。 |
| `update_data_error` | 更新平台数据失败。 | 避免重复提交；确认状态后再重试。 |
| `system_cpu_overloaded` | CPU 负载超过平台保护阈值。 | 稍后重试。 |
| `system_memory_overloaded` | 内存使用超过平台保护阈值。 | 稍后重试。 |
| `system_disk_overloaded` | 磁盘使用超过平台保护阈值。 | 稍后重试。 |

## 兼容端点的 `error.type`

少数兼容端点不会返回 `error.code`，而是只通过 `error.type` 标记错误类别。当前代码中视频代理等端点会使用：

| `error.type` | 场景 | 处理建议 |
| --- | --- | --- |
| `invalid_request_error` | 任务 ID 缺失、任务不存在、任务未完成或请求参数错误。 | 检查任务 ID、任务状态和请求参数。 |
| `server_error` | 查询任务、获取渠道、代理上游视频内容等服务端处理失败。 | 稍后重试；持续失败时联系支持。 |

## 重试建议

1. `400`、`401`、`403`、`404`、`413` 通常需要先修正请求、权限或额度，不建议直接重试。
2. `408`、`429`、`500`、`502`、`503` 可以使用指数退避重试，但应设置最大重试次数。
3. `sensitive_words_detected`、`prompt_blocked`、`violation_fee.grok.csam` 需要调整输入内容，不应自动重试相同请求。
4. `insufficient_user_quota` 需要充值或降低模型成本后再请求。
5. 如果 `message` 中包含请求 ID，请在反馈问题时一并提供，便于定位日志。

下一步可以查看 [认证](/zh/getting-started/authentication) 修正鉴权问题，或查看 [首个请求](/zh/getting-started/first-request) 对照最小可用请求格式。
