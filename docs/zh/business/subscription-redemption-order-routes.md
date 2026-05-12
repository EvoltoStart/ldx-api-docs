---
title: "订阅、兑换码与订单路由"
description: "说明订阅、兑换码和订单管理相关接口的测试路由与认证约定。"
---

# 订阅 / 兑换码 / 订单管理 路由说明（Apifox 测试版）

> 适用场景：你使用**自己的 MySQL + Redis**，并且是**前后端分离**部署。

## 测试前统一约定（前后端分离）
- 后端 API 地址：`http://localhost:8005`
- 前端地址（仅用于跳转）：`http://localhost:3000`
- Apifox 建议环境变量：
  - `{{baseUrl}} = http://localhost:8005`
  - `{{userJwt}} = 登录后的用户 JWT/access token`
  - `{{userId}} = 当前登录用户 ID`
  - `{{adminJwt}} = 管理员 JWT/access token`
  - `{{adminId}} = 管理员用户 ID`
  - `{{planId}} = 订阅计划 ID`
  - `{{tradeNo}} = 订单号`

### 认证注意（非常重要）
- 走 `UserAuth()` 的接口（如 `/api/subscription/*`、`/api/user/topup/*`、`/api/user/pay`）测试时，建议固定带：
  - `Authorization: Bearer {{userJwt}}`
  - `New-Api-User: {{userId}}`
- 管理员接口（如 `/api/redemption/*`、`/api/subscription/admin/*`、`/api/user/topup`）测试时，建议带：
  - `Authorization: Bearer {{adminJwt}}`
  - `New-Api-User: {{adminId}}`

---

## 模块一：订阅路由（按用户 -> 管理员 -> 回调顺序）

### 1) 获取订阅计划（用户）
- **路由完整请求路径**：`GET http://localhost:8005/api/subscription/plans`
- **路由作用**：获取当前启用的订阅套餐列表。
- **路由示例（Apifox）**：
```http
GET /api/subscription/plans
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```
- **路由测试需要注意什么**：只会返回 `enabled=true` 的套餐。

### 2) 获取当前用户订阅状态
- **路由完整请求路径**：`GET http://localhost:8005/api/subscription/self`
- **路由作用**：查看当前用户订阅信息（生效中 + 历史）。
- **路由示例（Apifox）**：
```http
GET /api/subscription/self
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```
- **路由测试需要注意什么**：响应里通常同时有 `subscriptions`（有效）和 `all_subscriptions`（含过期）。

### 3) 更新订阅计费偏好
- **路由完整请求路径**：`PUT http://localhost:8005/api/subscription/self/preference`
- **路由作用**：更新用户的计费偏好设置。
- **路由示例（Apifox）**：
```http
PUT /api/subscription/self/preference
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "billing_preference": "subscription"
}
```
- **路由测试需要注意什么**：建议只传一个合法字符串值，非法值会被后端标准化或拒绝。

### 4) 订阅下单（Epay） (属于支付模块)
- **路由完整请求路径**：`POST http://localhost:8005/api/subscription/epay/pay`
- **路由作用**：创建订阅 Epay 订单。
- **路由示例（Apifox）**：
```http
POST /api/subscription/epay/pay
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "plan_id": {{planId}},
  "payment_method": "alipay"
}
```
- **路由测试需要注意什么**：
  - `plan_id` 必须是已启用套餐。
  - `payment_method` 必须在系统支付方式白名单中。
  - 会先创建 `pending` 订单，最终是否成功要看回调。

### 5) 订阅下单（Stripe） (属于支付模块)
- **路由完整请求路径**：`POST http://localhost:8005/api/subscription/stripe/pay`
- **路由作用**：创建 Stripe Checkout 支付链接并创建订阅订单。
- **路由示例（Apifox）**：
```http
POST /api/subscription/stripe/pay
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "plan_id": {{planId}}
}
```
- **路由测试需要注意什么**：
  - 套餐必须配置 `StripePriceId`。
  - 后端需配置有效 Stripe key/webhook。

### 6) 订阅下单（Creem） (属于支付模块)
- **路由完整请求路径**：`POST http://localhost:8005/api/subscription/creem/pay`
- **路由作用**：创建 Creem 支付链接并创建订阅订单。
- **路由示例（Apifox）**：
```http
POST /api/subscription/creem/pay
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "plan_id": {{planId}}
}
```
- **路由测试需要注意什么**：
  - 套餐必须配置 `CreemProductId`。
  - 非测试模式下必须配置 `CreemWebhookSecret`。

### 7) 管理员：订阅计划列表
- **路由完整请求路径**：`GET http://localhost:8005/api/subscription/admin/plans`
- **路由作用**：管理员查看全部套餐（含未启用）。
- **路由示例（Apifox）**：
```http
GET /api/subscription/admin/plans
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
```
- **路由测试需要注意什么**：需管理员权限。

### 8) 管理员：创建订阅计划
- **路由完整请求路径**：`POST http://localhost:8005/api/subscription/admin/plans`
- **路由作用**：新增订阅套餐。
- **路由示例（Apifox）**：
```http
POST /api/subscription/admin/plans
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
Content-Type: application/json

{
  "plan": {
    "title": "Pro 月包",
    "subtitle": "适合高频调用",
    "price_amount": 9.9,
    "currency": "USD",
    "duration_unit": "month",
    "duration_value": 1,
    "enabled": true,
    "sort_order": 100,
    "max_purchase_per_user": 3,
    "total_amount": 500000
  }
}
```
- **路由测试需要注意什么**：
  - `title` 不能为空。
  - `price_amount` 不能小于 0 且不能过大。

### 9) 管理员：更新/启停订阅计划
- **路由完整请求路径**：
  - `PUT http://localhost:8005/api/subscription/admin/plans/{id}`
  - `PATCH http://localhost:8005/api/subscription/admin/plans/{id}`
- **路由作用**：更新计划详情或只更新启用状态。
- **路由示例（Apifox）**：
```http
PATCH /api/subscription/admin/plans/1
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
Content-Type: application/json

{
  "enabled": false
}
```
- **路由测试需要注意什么**：`PATCH` 只处理状态字段，`PUT` 走完整字段校验。

### 10) 订阅 Epay 回调（服务端）
- **路由完整请求路径**：
  - `POST http://localhost:8005/api/subscription/epay/notify`
  - `GET  http://localhost:8005/api/subscription/epay/notify`
  - `GET  http://localhost:8005/api/subscription/epay/return`
  - `POST http://localhost:8005/api/subscription/epay/return`
- **路由作用**：处理 Epay 异步通知/同步返回，完成订单入账和跳转。
- **路由示例（Apifox）**：
```http
POST /api/subscription/epay/notify
Content-Type: application/x-www-form-urlencoded

trade_no=平台订单号&out_trade_no=系统订单号&money=9.90&name=SUB:Pro&trade_status=TRADE_SUCCESS&sign=签名&sign_type=MD5
```
- **路由测试需要注意什么**：
  - 这类接口本质是支付平台回调，人工测建议在沙箱或 mock 签名环境。
  - 签名错误会直接失败。

---

## 模块二：兑换码路由（按用户兑换 -> 管理员码库管理顺序）

### 1) 用户兑换充值码
- **路由完整请求路径**：`POST http://localhost:8005/api/user/topup`
- **路由作用**：用户使用兑换码给账户充值（非第三方支付）。
- **路由示例（Apifox）**：
```http
POST /api/user/topup
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "key": "xxxx-xxxx-xxxx"
}
```
- **路由测试需要注意什么**：
  - 同一用户同一时刻会加锁，重复并发请求可能返回 `top up processing`。
  - 无效/已使用/过期兑换码会失败。

### 2) 管理员获取兑换码列表
- **路由完整请求路径**：`GET http://localhost:8005/api/redemption/`
- **路由作用**：分页查询全部兑换码。
- **路由示例（Apifox）**：
```http
GET /api/redemption/?p=0&page_size=20
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
```
- **路由测试需要注意什么**：需管理员权限。

### 3) 管理员搜索兑换码
- **路由完整请求路径**：`GET http://localhost:8005/api/redemption/search`
- **路由作用**：按关键字搜索兑换码。
- **路由示例（Apifox）**：
```http
GET /api/redemption/search?keyword=Pro&p=0&page_size=20
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
```
- **路由测试需要注意什么**：建议固定分页参数，便于核对总数变化。

### 4) 管理员创建兑换码
- **路由完整请求路径**：`POST http://localhost:8005/api/redemption/`
- **路由作用**：批量创建兑换码（后端会返回 key 列表）。
- **路由示例（Apifox）**：
```http
POST /api/redemption/
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
Content-Type: application/json

{
  "name": "活动批次A",
  "quota": 500000,
  "count": 3,
  "expired_time": 1893456000
}
```
- **路由测试需要注意什么**：
  - `count` 必须大于 0 且不超过 100。
  - `expired_time` 不能早于当前时间（0 通常代表不过期）。

### 5) 管理员更新兑换码
- **路由完整请求路径**：`PUT http://localhost:8005/api/redemption/`
- **路由作用**：更新兑换码基础信息，或仅更新状态。
- **路由示例（Apifox）**：
```http
PUT /api/redemption/?status_only=1
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
Content-Type: application/json

{
  "id": 1,
  "status": 2
}
```
- **路由测试需要注意什么**：
  - 仅改状态时带 `status_only`。
  - 全量更新时需要传合法 `expired_time`。

### 6) 管理员删除兑换码
- **路由完整请求路径**：
  - `DELETE http://localhost:8005/api/redemption/{id}`
  - `DELETE http://localhost:8005/api/redemption/invalid`
- **路由作用**：删除指定兑换码或批量删除无效兑换码。
- **路由示例（Apifox）**：
```http
DELETE /api/redemption/invalid
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
```
- **路由测试需要注意什么**：`/invalid` 通常会影响较多数据，建议先备份或在测试库操作。

---

## 模块三：订单管理路由（按用户查询 -> 下单 -> 管理员顺序）

### 1) 获取支付配置与可选金额
- **路由完整请求路径**：`GET http://localhost:8005/api/user/topup/info`
- **路由作用**：获取支付开关、最低充值额度、支付方式等。
- **路由示例（Apifox）**：
```http
GET /api/user/topup/info
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```
- **路由测试需要注意什么**：该接口返回配置，不会创建订单。

### 2) 用户查询自己的订单
- **路由完整请求路径**：`GET http://localhost:8005/api/user/topup/self`
- **路由作用**：分页查看当前用户充值订单。
- **路由示例（Apifox）**：
```http
GET /api/user/topup/self?p=0&page_size=20&keyword=USR
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```
- **路由测试需要注意什么**：`keyword` 可按订单号检索。

### 3) 下单前计算实付金额
- **路由完整请求路径**：`POST http://localhost:8005/api/user/amount`
- **路由作用**：根据充值额度与用户分组折扣计算应支付金额。
- **路由示例（Apifox）**：
```http
POST /api/user/amount
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "amount": 50
}
```
- **路由测试需要注意什么**：低于系统最小充值额度会直接报错。

### 4) 创建 Epay 订单
- **路由完整请求路径**：`POST http://localhost:8005/api/user/pay`
- **路由作用**：创建普通充值 Epay 订单。
- **路由示例（Apifox）**：
```http
POST /api/user/pay
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "amount": 50,
  "payment_method": "alipay"
}
```
- **路由测试需要注意什么**：
  - 会创建 `pending` 订单。
  - 是否成功到账依赖 `/api/user/epay/notify` 回调。

### 5) 创建 Stripe / Creem 订单
- **路由完整请求路径**：
  - `POST http://localhost:8005/api/user/stripe/pay`
  - `POST http://localhost:8005/api/user/creem/pay`
- **路由作用**：创建三方支付订单并返回支付链接。
- **路由示例（Apifox）**：
```http
POST /api/user/stripe/pay
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "amount": 50
}
```
- **路由测试需要注意什么**：后端需提前配置 Stripe/Creem 密钥与回调。

### 6) 支付回调（普通充值）
- **路由完整请求路径**：
  - `POST http://localhost:8005/api/user/epay/notify`
  - `GET  http://localhost:8005/api/user/epay/notify`
  - `POST http://localhost:8005/api/stripe/webhook`
  - `POST http://localhost:8005/api/creem/webhook`
- **路由作用**：接收支付网关异步通知并更新订单状态。
- **路由示例（Apifox）**：
```http
POST /api/stripe/webhook
Stripe-Signature: t=...,v1=...
Content-Type: application/json

{ "id": "evt_xxx", "type": "checkout.session.completed" }
```
- **路由测试需要注意什么**：
  - 建议使用支付平台测试模式。
  - Webhook 签名不正确会处理失败。

### 7) 管理员查看全平台订单
- **路由完整请求路径**：`GET http://localhost:8005/api/user/topup`
- **路由作用**：管理员分页查看所有充值订单。
- **路由示例（Apifox）**：
```http
GET /api/user/topup?p=0&page_size=50&keyword=USR
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
```
- **路由测试需要注意什么**：需管理员权限。

### 8) 管理员手动补单
- **路由完整请求路径**：`POST http://localhost:8005/api/user/topup/complete`
- **路由作用**：按订单号手工完成订单（回调丢失时使用）。
- **路由示例（Apifox）**：
```http
POST /api/user/topup/complete
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
Content-Type: application/json

{
  "trade_no": "USR1NOabc123"
}
```
- **路由测试需要注意什么**：
  - 同一订单会做互斥锁，避免并发补单。 
  - 已完成/不合法状态订单会返回错误。

---

## 补充建议（MySQL + Redis + 前后端分离）
- 先验证后端配置：支付密钥、回调地址、套餐配置、兑换码有效期策略。
- Apifox 优先走后端真实域名/端口，不要混用前端地址。
- 回调类接口建议通过内网穿透或网关转发到 `localhost:8005`，并确保签名字段完整。
