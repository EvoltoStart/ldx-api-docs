# API KEY / 邀请码 / 支付 路由说明（Apifox 测试版）

> 适用场景：你使用**自己的 MySQL + Redis**，并且是**前后端分离部署**。

## 测试环境约定（前后端分离）
- 后端 API 地址（示例）：`http://localhost:8005`
- 前端地址（示例）：`http://localhost:3000`
- Apifox 中建议创建环境变量：
  - `{{baseUrl}} = http://localhost:8005`
  - `{{userJwt}} = 用户登录后的 JWT`
  - `{{apiKey}} = 用户创建的 sk-xxxx`
  - `{{userId}} = 当前登录用户 ID`
- 若你通过 Nginx 把前后端统一到了 `http://localhost`，把 `{{baseUrl}}` 改成对应网关地址即可。

### 关键认证说明（非常重要）
- 对所有走 `UserAuth()` 的接口（例如 `/api/token/*`、`/api/user/aff`、`/api/user/pay`）：
  1. 需要 `Authorization: Bearer <access_token>`（或有效会话 Cookie）；
  2. 还需要请求头 `New-Api-User: <当前用户ID>`，且必须与登录用户一致。
- 你贴的报错里 SQL 条件是 `access_token = 'Beare ...'`，说明 Authorization 写成了 `Beare`（少了一个 `r`）。
- 你贴的 `session=...` Cookie 可以用于浏览器控制台场景，但在 Apifox 通常更建议显式传 `Authorization` + `New-Api-User`。

---

## 模块一：API KEY 相关路由

### 1. 创建 API KEY
- **路由完整请求路径**：`POST {{baseUrl}}/api/token/`
- **路由作用**：为当前登录用户创建一个新的 API KEY（sk-前缀）。
- **路由示例（Apifox）**：
```http
POST /api/token/
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "name": "apifox-key-1",
  "expired_time": -1,
  "remain_quota": 500000,
  "unlimited_quota": false,
  "model_limits_enabled": false,
  "model_limits": "",
  "allow_ips": "",
  "group": "default",
  "cross_group_retry": false
}
```
- **路由测试需要注意什么**：
  - 必须使用用户 JWT，不是 sk-key。
  - 用户 token 数达到系统上限会失败。

### 2. 获取 API KEY 列表
- **路由完整请求路径**：`GET {{baseUrl}}/api/token/`
- **路由作用**：分页获取当前用户的 API KEY 列表。
- **路由示例（Apifox）**：
```http
GET /api/token/?p=0&page_size=10
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```
- **路由测试需要注意什么**：
  - 建议固定分页参数，便于核对新增/删除结果。

### 3. 搜索 API KEY
- **路由完整请求路径**：`GET {{baseUrl}}/api/token/search`
- **路由作用**：按名称或 token 关键字搜索。
- **路由示例（Apifox）**：
```http
GET /api/token/search?keyword=%apifox%&token=%sk-ab%
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```
- **路由测试需要注意什么**：
  - `%` 通配符使用有规则限制，非法格式会返回错误。

### 4. 更新 API KEY
- **路由完整请求路径**：`PUT {{baseUrl}}/api/token/`
- **路由作用**：更新 API KEY 配置（名称、状态、额度、分组等）。
- **路由示例（Apifox）**：
```http
PUT /api/token/
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "id": 1,
  "name": "apifox-key-1-updated",
  "status": 1,
  "expired_time": -1,
  "remain_quota": 600000,
  "unlimited_quota": false,
  "group": "default"
}
```
- **路由测试需要注意什么**：
  - `id` 必须属于当前用户。
  - 建议先调列表接口拿真实 id。

### 5. 删除 API KEY
- **路由完整请求路径**：`DELETE {{baseUrl}}/api/token/{id}`
- **路由作用**：删除指定 API KEY。
- **路由示例（Apifox）**：
```http
DELETE /api/token/1
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```
- **路由测试需要注意什么**：
  - 删除后再查列表确认。

### 6. 获取单个 API KEY 详情
- **路由完整请求路径**：`GET {{baseUrl}}/api/token/{id}`
- **路由作用**：查询单个 API KEY 的完整信息（用于编辑前回填）。
- **路由示例（Apifox）**：
```http
GET /api/token/1
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```
- **路由测试需要注意什么**：
  - `id` 必须属于当前用户，否则会返回无权限或查询失败。

### 7. 批量删除 API KEY
- **路由完整请求路径**：`POST {{baseUrl}}/api/token/batch`
- **路由作用**：按 `ids` 数组批量删除 API KEY。
- **路由示例（Apifox）**：
```http
POST /api/token/batch
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "ids": [1, 2, 3]
}
```
- **路由测试需要注意什么**：
  - 必须是 JSON，且字段名必须是 `ids`（不是 `id`）。
  - `ids` 为空数组会报 `Invalid parameters`。
  - 只能删除当前用户自己的 token。

### 8. 查询 API KEY 用量
- **路由完整请求路径**：`GET {{baseUrl}}/api/usage/token/`
- **路由作用**：通过 API KEY 查询该 key 的额度和使用统计。
- **路由示例（Apifox）**：
```http
GET /api/usage/token/
Authorization: Bearer {{apiKey}}
```
- **路由测试需要注意什么**：
  - 这里使用的是 `Bearer sk-...`，不是用户 JWT。

### 9. 模型中转（验证 KEY 是否可用）
- **路由完整请求路径**：
  - `POST {{baseUrl}}/v1/chat/completions`
  - `POST {{baseUrl}}/v1/messages`
  - `GET {{baseUrl}}/v1/models`
- **路由作用**：真实消费 API KEY 调用模型。
- **路由示例（Apifox）**：
```http
POST /v1/chat/completions
Authorization: Bearer {{apiKey}}
Content-Type: application/json

{
  "model": "gpt-4o-mini",
  "messages": [
    {"role": "user", "content": "hello"}
  ]
}
```
- **路由测试需要注意什么**：
  - KEY 必须未过期、未禁用、额度充足。
  - 某些兼容场景也支持 `x-api-key` / `x-goog-api-key`。

### 10. 生成用户 access token（Bearer 模式）
- **路由完整请求路径**：`GET {{baseUrl}}/api/user/token`
- **路由作用**：为当前登录用户生成（或刷新）`Authorization: Bearer` 使用的 access token。
- **路由示例（Apifox）**：
```http
GET /api/user/token
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```
- **路由测试需要注意什么**：
  - 该接口本身也走 `UserAuth()`；要么带有效 session/cookie，要么带已存在 access token。
  - 响应 `data` 就是可用于 `Authorization: Bearer <token>` 的值。

---

## 模块二：邀请码相关路由

### 1. 注册（携带邀请码）
- **路由完整请求路径**：`POST {{baseUrl}}/api/user/register`
- **路由作用**：用户注册并可通过 `aff_code` 建立邀请关系。
- **路由示例（Apifox）**：
```http
POST /api/user/register
Content-Type: application/json

{
  "username": "invite_test_01",
  "password": "Password123!",
  "password2": "Password123!",
  "email": "invite_test_01@example.com",
  "aff_code": "ABCD"
}
```
- **路由测试需要注意什么**：
  - 若启用人机验证（Turnstile），需带对应参数。
  - `aff_code` 无效时不会建立邀请奖励关系。

### 2. 获取当前用户邀请码
- **路由完整请求路径**：`GET {{baseUrl}}/api/user/aff`
- **路由作用**：获取当前用户邀请码（为空则后端自动生成）。
- **路由示例（Apifox）**：
```http
GET /api/user/aff
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```
- **路由测试需要注意什么**：
  - 必须登录。

### 3. 邀请额度划转
- **路由完整请求路径**：`POST {{baseUrl}}/api/user/aff_transfer`
- **路由作用**：把邀请额度（aff_quota）转成账户可用额度（quota）。
- **路由示例（Apifox）**：
```http
POST /api/user/aff_transfer
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "quota": 500000
}
```
- **路由测试需要注意什么**：
  - 有最小划转额度限制。
  - 邀请额度不足会报错。

### 4. 获取当前登录用户信息（含邀请统计）
- **路由完整请求路径**：`GET {{baseUrl}}/api/user/self`
- **路由作用**：获取当前用户完整信息，包含邀请相关字段：`aff_code/aff_count/aff_quota/inviter_id`。
- **路由示例（Apifox）**：
```http
GET /api/user/self
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```
- **路由测试需要注意什么**：
  - 用于验证邀请码注册/邀请奖励后数据是否生效。

---

## 模块三：支付相关路由

### 1. 获取充值配置
- **路由完整请求路径**：`GET {{baseUrl}}/api/user/topup/info`
- **路由作用**：返回支付开关、最低充值、支付方式、折扣等配置。
- **路由示例（Apifox）**：
```http
GET /api/user/topup/info
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```
- **路由测试需要注意什么**：
  - 仅返回配置，不会创建订单。

### 2. 普通金额试算
- **路由完整请求路径**：`POST {{baseUrl}}/api/user/amount`
- **路由作用**：计算普通支付渠道的应付金额。
- **路由示例（Apifox）**：
```http
POST /api/user/amount
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "amount": 10
}
```
- **路由测试需要注意什么**：
  - 仅试算，不下单。

### 3. 易支付下单
- **路由完整请求路径**：`POST {{baseUrl}}/api/user/pay`
- **路由作用**：创建 Epay 订单并返回支付参数。
- **路由示例（Apifox）**：
```http
POST /api/user/pay
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "amount": 10,
  "payment_method": "alipay"
}
```
- **路由测试需要注意什么**：
  - 支付方式必须在后台配置列表中。
  - 会创建 pending 订单，最终成功要看回调。

### 4. Stripe 金额试算
- **路由完整请求路径**：`POST {{baseUrl}}/api/user/stripe/amount`
- **路由作用**：计算 Stripe 支付金额。
- **路由示例（Apifox）**：
```http
POST /api/user/stripe/amount
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "amount": 10,
  "payment_method": "stripe"
}
```
- **路由测试需要注意什么**：
  - 仅试算，不创建 Stripe Checkout。

### 5. Stripe 下单
- **路由完整请求路径**：`POST {{baseUrl}}/api/user/stripe/pay`
- **路由作用**：创建 Stripe Checkout 链接并落库订单。
- **路由示例（Apifox）**：
```http
POST /api/user/stripe/pay
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "amount": 10,
  "payment_method": "stripe",
  "success_url": "http://localhost:3000/console/log",
  "cancel_url": "http://localhost:3000/topup"
}
```
- **路由测试需要注意什么**：
  - 回跳 URL 需满足系统可信域名校验。
  - 最终入账依赖 Stripe webhook。

### 6. Creem 下单
- **路由完整请求路径**：`POST {{baseUrl}}/api/user/creem/pay`
- **路由作用**：按产品 ID 发起 Creem 支付。
- **路由示例（Apifox）**：
```http
POST /api/user/creem/pay
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "payment_method": "creem",
  "product_id": "prod_xxxxx"
}
```
- **路由测试需要注意什么**：
  - 需要先在后台配置 Creem API Key 和产品。

### 7. 易支付回调（POST）
- **路由完整请求路径**：`POST {{baseUrl}}/api/user/epay/notify`
- **路由作用**：接收支付网关服务端异步通知（推荐主回调方式），验签后更新订单并充值。
- **路由示例（Apifox）**：
```http
POST /api/user/epay/notify
Content-Type: application/x-www-form-urlencoded

trade_status=TRADE_SUCCESS&service_trade_no=USR1NOxxxx&sign=xxxx&...
```
- **路由测试需要注意什么**：
  - 必须是网关正确签名参数，否则验签失败。
  - 该接口会执行订单状态推进与额度发放，适合做回调联调。

### 8. 易支付回调（GET）
- **路由完整请求路径**：`GET {{baseUrl}}/api/user/epay/notify`
- **路由作用**：兼容某些网关用 GET 方式回调的场景，处理逻辑与 POST notify 一致。
- **路由示例（Apifox）**：
```http
GET /api/user/epay/notify?trade_status=TRADE_SUCCESS&service_trade_no=USR1NOxxxx&sign=xxxx&...
```
- **路由测试需要注意什么**：
  - 参数和签名规则与 POST 一样。
  - 实际生产建议优先使用 POST 回调。

### 9. Stripe Webhook
- **路由完整请求路径**：`POST {{baseUrl}}/api/stripe/webhook`
- **路由作用**：Stripe 支付事件回调，验签后处理订单。
- **路由示例（Apifox）**：
```http
POST /api/stripe/webhook
Stripe-Signature: t=...,v1=...
Content-Type: application/json

{ "id": "evt_xxx", "type": "checkout.session.completed", ... }
```
- **路由测试需要注意什么**：
  - 强依赖签名头，建议用 Stripe CLI 转发到本地测试。

### 10. Creem Webhook
- **路由完整请求路径**：`POST {{baseUrl}}/api/creem/webhook`
- **路由作用**：Creem 支付事件回调并处理入账。
- **路由示例（Apifox）**：
```http
POST /api/creem/webhook
creem-signature: <signature>
Content-Type: application/json

{
  "id": "evt_xxx",
  "eventType": "checkout.completed",
  "object": {
    "order": {
      "id": "order_xxx",
      "status": "paid"
    }
  }
}
```
- **路由测试需要注意什么**：
  - 需使用正确签名头。

### 11. 兑换码充值（非三方支付）
- **路由完整请求路径**：`POST {{baseUrl}}/api/user/topup`
- **路由作用**：使用兑换码/充值码充值（走 Redeem 逻辑）。
- **路由示例（Apifox）**：
```http
POST /api/user/topup
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "key": "REDEEM-CODE-XXXX"
}
```
- **路由测试需要注意什么**：
  - 与 `/api/user/pay` 不同，这是兑换码充值。
  - 并发提交同用户会被后端锁控制。

### 12. 用户入账记录
- **路由完整请求路径**：`GET {{baseUrl}}/api/user/topup/self`
- **路由作用**：查询当前用户入账记录，包含在线充值订单和兑换码兑换记录。
- **路由示例（Apifox）**：
```http
GET /api/user/topup/self?p=0&page_size=10
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```
- **路由测试需要注意什么**：
  - 可加 `trade_no` 做订单号或兑换码检索。
  - `payment_method=redemption` 只返回兑换码兑换记录。
  - 不传 `payment_method` 时会合并返回在线充值订单和兑换码兑换记录。

### 13. 管理员查看全站充值记录
- **路由完整请求路径**：`GET {{baseUrl}}/api/user/topup`
- **路由作用**：管理员查看全平台充值订单。
- **路由示例（Apifox）**：
```http
GET /api/user/topup?p=0&page_size=20
Authorization: Bearer <管理员JWT>
New-Api-User: <管理员用户ID>
```
- **路由测试需要注意什么**：
  - 必须管理员权限。

### 14. 管理员补单（手动完成订单）
- **路由完整请求路径**：`POST {{baseUrl}}/api/user/topup/complete`
- **路由作用**：管理员根据订单号手动补单，修复漏回调导致未入账的问题。
- **路由示例（Apifox）**：
```http
POST /api/user/topup/complete
Authorization: Bearer <管理员JWT>
New-Api-User: <管理员用户ID>
Content-Type: application/json

{
  "trade_no": "USR1NOxxxx"
}
```
- **路由测试需要注意什么**：
  - 仅管理员可用。
  - 仅对状态符合条件的订单可补单，重复补单会失败或被忽略。

### 15. 订阅支付-获取可购买套餐
- **路由完整请求路径**：`GET {{baseUrl}}/api/subscription/plans`
- **路由作用**：获取当前可见的订阅套餐列表（用于下单前展示）。
- **路由示例（Apifox）**：
```http
GET /api/subscription/plans
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```
- **路由测试需要注意什么**：
  - 该接口走 `UserAuth()`，要带登录态和 `New-Api-User`。
  - 返回空列表通常是后台没有启用订阅计划。

### 16. 订阅支付-Epay 下单
- **路由完整请求路径**：`POST {{baseUrl}}/api/subscription/epay/pay`
- **路由作用**：按订阅计划创建 Epay 订单，返回支付参数。
- **路由示例（Apifox）**：
```http
POST /api/subscription/epay/pay
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "plan_id": 1,
  "payment_method": "alipay"
}
```
- **路由测试需要注意什么**：
  - `plan_id` 必须是已启用计划。
  - `payment_method` 必须在系统可用支付方式中。

### 17. 订阅支付-Stripe 下单
- **路由完整请求路径**：`POST {{baseUrl}}/api/subscription/stripe/pay`
- **路由作用**：创建 Stripe 订阅 Checkout 链接。
- **路由示例（Apifox）**：
```http
POST /api/subscription/stripe/pay
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "plan_id": 1
}
```
- **路由测试需要注意什么**：
  - 计划需配置有效 `stripe_price_id`。
  - 最终订阅生效依赖 Stripe 回调事件处理。

### 18. 订阅支付-Creem 下单
- **路由完整请求路径**：`POST {{baseUrl}}/api/subscription/creem/pay`
- **路由作用**：创建 Creem 订阅支付链接。
- **路由示例（Apifox）**：
```http
POST /api/subscription/creem/pay
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "plan_id": 1
}
```
- **路由测试需要注意什么**：
  - 需要后台启用 Creem，并在计划里关联可用产品。

### 19. 订阅支付-Epay 异步回调（POST）
- **路由完整请求路径**：`POST {{baseUrl}}/api/subscription/epay/notify`
- **路由作用**：订阅订单的主异步回调入口，验签通过后完成订阅订单并生效。
- **路由示例（Apifox）**：
```http
POST /api/subscription/epay/notify
Content-Type: application/x-www-form-urlencoded

trade_status=TRADE_SUCCESS&service_trade_no=SUBxxxx&sign=xxxx&...
```
- **路由测试需要注意什么**：
  - 必须是正确签名数据，伪造参数不会通过验签。
  - 订阅是否真正生效优先看 notify 处理结果。

### 20. 订阅支付-Epay 异步回调（GET）
- **路由完整请求路径**：`GET {{baseUrl}}/api/subscription/epay/notify`
- **路由作用**：兼容 GET 模式订阅回调，逻辑与 POST notify 一致。
- **路由示例（Apifox）**：
```http
GET /api/subscription/epay/notify?trade_status=TRADE_SUCCESS&service_trade_no=SUBxxxx&sign=xxxx&...
```
- **路由测试需要注意什么**：
  - 同样需要完整签名参数。

### 21. 订阅支付-Epay 浏览器回跳（GET）
- **路由完整请求路径**：`GET {{baseUrl}}/api/subscription/epay/return`
- **路由作用**：支付完成后浏览器 GET 回跳入口，展示支付结果并尝试补全订单。
- **路由示例（Apifox）**：
```http
GET /api/subscription/epay/return?trade_status=TRADE_SUCCESS&service_trade_no=SUBxxxx&sign=xxxx&...
```
- **路由测试需要注意什么**：
  - 主要用于前端页面跳转提示，核心入账仍以 notify 为准。

### 22. 订阅支付-Epay 浏览器回跳（POST）
- **路由完整请求路径**：`POST {{baseUrl}}/api/subscription/epay/return`
- **路由作用**：支付网关 POST 方式浏览器回跳兼容入口，逻辑与 GET return 一致。
- **路由示例（Apifox）**：
```http
POST /api/subscription/epay/return
Content-Type: application/x-www-form-urlencoded

trade_status=TRADE_SUCCESS&service_trade_no=SUBxxxx&sign=xxxx&...
```
- **路由测试需要注意什么**：
  - 回跳接口主要负责用户体验展示和兜底，不建议替代异步通知。

### 23. 订阅支付-查看我的订阅状态
- **路由完整请求路径**：`GET {{baseUrl}}/api/subscription/self`
- **路由作用**：查询当前用户订阅信息（当前有效订阅 + 历史订阅）。
- **路由示例（Apifox）**：
```http
GET /api/subscription/self
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```
- **路由测试需要注意什么**：
  - 常用于支付成功后确认订阅是否生效。

### 24. 订阅支付-更新订阅偏好
- **路由完整请求路径**：`PUT {{baseUrl}}/api/subscription/self/preference`
- **路由作用**：更新用户订阅偏好设置（例如默认订阅来源策略）。
- **路由示例（Apifox）**：
```http
PUT /api/subscription/self/preference
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "billing_preference": "subscription_first"
}
```
- **路由测试需要注意什么**：
  - 参数枚举以接口返回/后端校验为准，非法值会报参数错误。
