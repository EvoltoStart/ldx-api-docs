---
title: "API Key, Invite Code, and Payment Routes"
description: "Explains test routes and authentication conventions for API Key, invite-code, and payment-related APIs."
---

# API Key, Invite Code, and Payment Routes (Apifox Test Version)

> Applicable scenario: You use your **own MySQL + Redis** and deploy the frontend and backend separately.

## Test Environment Conventions (Frontend and Backend Separated)

- Backend API example: `http://localhost:8005`
- Frontend example: `http://localhost:3000`
- Recommended Apifox environment variables:
  - `{{baseUrl}} = http://localhost:8005`
  - `{{userJwt}} = JWT after user login`
  - `{{apiKey}} = sk-xxxx created by the user`
  - `{{userId}} = current logged-in user ID`
- If you unify frontend and backend under `http://localhost` through Nginx, change `{{baseUrl}}` to the corresponding gateway address.

### Important Authentication Notes

- For all endpoints that use `UserAuth()`, such as `/api/token/*`, `/api/user/aff`, and `/api/user/pay`:
  1. `Authorization: Bearer <access_token>` is required, or a valid session cookie can be used.
  2. The `New-Api-User: <current user ID>` request header is also required and must match the logged-in user.
- In the error you posted, the SQL condition was `access_token = 'Beare ...'`, which means `Authorization` was written as `Beare` and missed the final `r`.
- The `session=...` cookie you posted can be used in browser-console scenarios, but in Apifox it is usually better to pass `Authorization` and `New-Api-User` explicitly.

---

## Module 1: API Key Routes

### 1. Create an API Key

- **Full route path**: `POST {{baseUrl}}/api/token/`
- **Purpose**: Creates a new API Key with the `sk-` prefix for the current logged-in user.
- **Example request in Apifox**:

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

- **Testing notes**:
  - Use a user JWT, not an `sk-` key.
  - The request fails if the user's token count reaches the system limit.

### 2. Get API Key List

- **Full route path**: `GET {{baseUrl}}/api/token/`
- **Purpose**: Retrieves the current user's API Key list with pagination.
- **Example request in Apifox**:

```http
GET /api/token/?p=0&page_size=10
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```

- **Testing notes**:
  - Use fixed pagination parameters to make it easier to verify create/delete results.

### 3. Search API Keys

- **Full route path**: `GET {{baseUrl}}/api/token/search`
- **Purpose**: Searches by name or token keyword.
- **Example request in Apifox**:

```http
GET /api/token/search?keyword=%apifox%&token=%sk-ab%
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```

- **Testing notes**:
  - `%` wildcards have format restrictions. Invalid formats return an error.

### 4. Update an API Key

- **Full route path**: `PUT {{baseUrl}}/api/token/`
- **Purpose**: Updates API Key settings such as name, status, quota, and group.
- **Example request in Apifox**:

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

- **Testing notes**:
  - `id` must belong to the current user.
  - Call the list endpoint first to get a real ID.

### 5. Delete an API Key

- **Full route path**: `DELETE {{baseUrl}}/api/token/{id}`
- **Purpose**: Deletes the specified API Key.
- **Example request in Apifox**:

```http
DELETE /api/token/1
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```

- **Testing notes**:
  - Query the list again after deletion to confirm the result.

### 6. Get API Key Details

- **Full route path**: `GET {{baseUrl}}/api/token/{id}`
- **Purpose**: Queries the complete information of a single API Key, usually for filling an edit form.
- **Example request in Apifox**:

```http
GET /api/token/1
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```

- **Testing notes**:
  - `id` must belong to the current user, otherwise the request returns a permission or query failure.

### 7. Batch Delete API Keys

- **Full route path**: `POST {{baseUrl}}/api/token/batch`
- **Purpose**: Batch deletes API Keys using an `ids` array.
- **Example request in Apifox**:

```http
POST /api/token/batch
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "ids": [1, 2, 3]
}
```

- **Testing notes**:
  - The request must be JSON, and the field name must be `ids`, not `id`.
  - An empty `ids` array returns `Invalid parameters`.
  - Users can only delete their own tokens.

### 8. Query API Key Usage

- **Full route path**: `GET {{baseUrl}}/api/usage/token/`
- **Purpose**: Queries quota and usage statistics for a specific API Key.
- **Example request in Apifox**:

```http
GET /api/usage/token/
Authorization: Bearer {{apiKey}}
```

- **Testing notes**:
  - This endpoint uses `Bearer sk-...`, not a user JWT.

### 9. Model Relay: Verify Whether the Key Works

- **Full route paths**:
  - `POST {{baseUrl}}/v1/chat/completions`
  - `POST {{baseUrl}}/v1/messages`
  - `GET {{baseUrl}}/v1/models`
- **Purpose**: Uses the API Key to call a model for real consumption.
- **Example request in Apifox**:

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

- **Testing notes**:
  - The key must not be expired, disabled, or out of quota.
  - Some compatibility scenarios also support `x-api-key` and `x-goog-api-key`.

### 10. Generate User Access Token (Bearer Mode)

- **Full route path**: `GET {{baseUrl}}/api/user/token`
- **Purpose**: Generates or refreshes the access token used as `Authorization: Bearer` for the current logged-in user.
- **Example request in Apifox**:

```http
GET /api/user/token
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```

- **Testing notes**:
  - This endpoint itself also uses `UserAuth()`. Use either a valid session/cookie or an existing access token.
  - The response `data` value is the token that can be used as `Authorization: Bearer <token>`.

---

## Module 2: Invite-Code Routes

### 1. Register with Invite Code

- **Full route path**: `POST {{baseUrl}}/api/user/register`
- **Purpose**: Registers a user and can establish an invite relationship through `aff_code`.
- **Example request in Apifox**:

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

- **Testing notes**:
  - If human verification such as Turnstile is enabled, the corresponding parameter is required.
  - An invalid `aff_code` does not establish an invite reward relationship.

### 2. Get Current User Invite Code

- **Full route path**: `GET {{baseUrl}}/api/user/aff`
- **Purpose**: Gets the current user's invite code. If it is empty, the backend generates one automatically.
- **Example request in Apifox**:

```http
GET /api/user/aff
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```

- **Testing notes**:
  - Login is required.

### 3. Transfer Invite Quota

- **Full route path**: `POST {{baseUrl}}/api/user/aff_transfer`
- **Purpose**: Converts invite quota (`aff_quota`) into available account quota (`quota`).
- **Example request in Apifox**:

```http
POST /api/user/aff_transfer
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "quota": 500000
}
```

- **Testing notes**:
  - There is a minimum transfer amount.
  - The request fails if invite quota is insufficient.

### 4. Get Current Logged-in User Information, Including Invite Statistics

- **Full route path**: `GET {{baseUrl}}/api/user/self`
- **Purpose**: Gets complete current-user information, including invite-related fields: `aff_code`, `aff_count`, `aff_quota`, and `inviter_id`.
- **Example request in Apifox**:

```http
GET /api/user/self
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```

- **Testing notes**:
  - Use this endpoint to verify whether invite-code registration and invite rewards have taken effect.

---

## Module 3: Payment Routes

### 1. Get Top-up Configuration

- **Full route path**: `GET {{baseUrl}}/api/user/topup/info`
- **Purpose**: Returns payment switches, minimum top-up amount, payment methods, discounts, and related settings.
- **Example request in Apifox**:

```http
GET /api/user/topup/info
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```

- **Testing notes**:
  - This endpoint only returns configuration and does not create an order.

### 2. Calculate Regular Payment Amount

- **Full route path**: `POST {{baseUrl}}/api/user/amount`
- **Purpose**: Calculates the payable amount for regular payment channels.
- **Example request in Apifox**:

```http
POST /api/user/amount
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "amount": 10
}
```

- **Testing notes**:
  - This only calculates the amount and does not create an order.

### 3. Create Epay Order

- **Full route path**: `POST {{baseUrl}}/api/user/pay`
- **Purpose**: Creates an Epay order and returns payment parameters.
- **Example request in Apifox**:

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

- **Testing notes**:
  - The payment method must be configured in the backend.
  - A `pending` order is created. Final success depends on the callback.

### 4. Calculate Stripe Payment Amount

- **Full route path**: `POST {{baseUrl}}/api/user/stripe/amount`
- **Purpose**: Calculates the Stripe payment amount.
- **Example request in Apifox**:

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

- **Testing notes**:
  - This only calculates the amount and does not create Stripe Checkout.

### 5. Create Stripe Order

- **Full route path**: `POST {{baseUrl}}/api/user/stripe/pay`
- **Purpose**: Creates a Stripe Checkout link and stores the order.
- **Example request in Apifox**:

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

- **Testing notes**:
  - Redirect URLs must pass the trusted-domain validation.
  - Final accounting depends on the Stripe webhook.

### 6. Create Creem Order

- **Full route path**: `POST {{baseUrl}}/api/user/creem/pay`
- **Purpose**: Starts a Creem payment by product ID.
- **Example request in Apifox**:

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

- **Testing notes**:
  - Configure the Creem API Key and product in the backend first.

### 7. Epay Callback (POST)

- **Full route path**: `POST {{baseUrl}}/api/user/epay/notify`
- **Purpose**: Receives asynchronous server-side notifications from the payment gateway. After signature verification, it updates the order and adds quota.
- **Example request in Apifox**:

```http
POST /api/user/epay/notify
Content-Type: application/x-www-form-urlencoded

trade_status=TRADE_SUCCESS&service_trade_no=USR1NOxxxx&sign=xxxx&...
```

- **Testing notes**:
  - Correctly signed gateway parameters are required; otherwise signature verification fails.
  - This endpoint advances order status and grants quota, so it is suitable for callback integration testing.

### 8. Epay Callback (GET)

- **Full route path**: `GET {{baseUrl}}/api/user/epay/notify`
- **Purpose**: Supports gateways that send callbacks through GET. The processing logic is the same as POST notify.
- **Example request in Apifox**:

```http
GET /api/user/epay/notify?trade_status=TRADE_SUCCESS&service_trade_no=USR1NOxxxx&sign=xxxx&...
```

- **Testing notes**:
  - Parameters and signature rules are the same as POST.
  - POST callbacks are recommended in production.

### 9. Stripe Webhook

- **Full route path**: `POST {{baseUrl}}/api/stripe/webhook`
- **Purpose**: Handles Stripe payment event callbacks and processes orders after signature verification.
- **Example request in Apifox**:

```http
POST /api/stripe/webhook
Stripe-Signature: t=...,v1=...
Content-Type: application/json

{ "id": "evt_xxx", "type": "checkout.session.completed", ... }
```

- **Testing notes**:
  - This strongly depends on the signature header. Use Stripe CLI to forward events to local development when testing.

### 10. Creem Webhook

- **Full route path**: `POST {{baseUrl}}/api/creem/webhook`
- **Purpose**: Handles Creem payment events and completes accounting.
- **Example request in Apifox**:

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

- **Testing notes**:
  - A valid signature header is required.

### 11. Redeem Code Top-up (No Third-party Payment)

- **Full route path**: `POST {{baseUrl}}/api/user/topup`
- **Purpose**: Uses a redemption/top-up code to top up the account through the redeem logic.
- **Example request in Apifox**:

```http
POST /api/user/topup
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "key": "REDEEM-CODE-XXXX"
}
```

- **Testing notes**:
  - This is different from `/api/user/pay`; it is redemption-code top-up.
  - Concurrent submissions for the same user are controlled by backend locks.

### 12. User Top-up Records

- **Full route path**: `GET {{baseUrl}}/api/user/topup/self`
- **Purpose**: Queries the current user's accounting records, including online top-up orders and redemption-code records.
- **Example request in Apifox**:

```http
GET /api/user/topup/self?p=0&page_size=10
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```

- **Testing notes**:
  - Use `trade_no` to search by order number or redemption code.
  - `payment_method=redemption` returns only redemption-code records.
  - Without `payment_method`, online top-up orders and redemption-code records are returned together.

### 13. Admin: View All Top-up Records

- **Full route path**: `GET {{baseUrl}}/api/user/topup`
- **Purpose**: Allows administrators to view top-up orders across the platform.
- **Example request in Apifox**:

```http
GET /api/user/topup?p=0&page_size=20
Authorization: Bearer <admin JWT>
New-Api-User: <admin user ID>
```

- **Testing notes**:
  - Administrator permission is required.

### 14. Admin: Manually Complete an Order

- **Full route path**: `POST {{baseUrl}}/api/user/topup/complete`
- **Purpose**: Allows administrators to manually complete an order by order number, usually to fix missed callbacks.
- **Example request in Apifox**:

```http
POST /api/user/topup/complete
Authorization: Bearer <admin JWT>
New-Api-User: <admin user ID>
Content-Type: application/json

{
  "trade_no": "USR1NOxxxx"
}
```

- **Testing notes**:
  - Administrator permission is required.
  - Only orders in valid states can be completed. Duplicate completion fails or is ignored.

### 15. Subscription Payment: Get Available Plans

- **Full route path**: `GET {{baseUrl}}/api/subscription/plans`
- **Purpose**: Gets visible subscription plans before placing an order.
- **Example request in Apifox**:

```http
GET /api/subscription/plans
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```

- **Testing notes**:
  - This endpoint uses `UserAuth()`, so login state and `New-Api-User` are required.
  - An empty list usually means no subscription plan is enabled in the backend.

### 16. Subscription Payment: Create Epay Order

- **Full route path**: `POST {{baseUrl}}/api/subscription/epay/pay`
- **Purpose**: Creates an Epay order for a subscription plan and returns payment parameters.
- **Example request in Apifox**:

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

- **Testing notes**:
  - `plan_id` must be an enabled plan.
  - `payment_method` must be one of the available system payment methods.

### 17. Subscription Payment: Create Stripe Order

- **Full route path**: `POST {{baseUrl}}/api/subscription/stripe/pay`
- **Purpose**: Creates a Stripe subscription Checkout link.
- **Example request in Apifox**:

```http
POST /api/subscription/stripe/pay
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "plan_id": 1
}
```

- **Testing notes**:
  - The plan must have a valid `stripe_price_id`.
  - Final subscription activation depends on Stripe callback event handling.

### 18. Subscription Payment: Create Creem Order

- **Full route path**: `POST {{baseUrl}}/api/subscription/creem/pay`
- **Purpose**: Creates a Creem subscription payment link.
- **Example request in Apifox**:

```http
POST /api/subscription/creem/pay
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "plan_id": 1
}
```

- **Testing notes**:
  - Creem must be enabled in the backend, and the plan must be associated with an available product.

### 19. Subscription Payment: Epay Async Callback (POST)

- **Full route path**: `POST {{baseUrl}}/api/subscription/epay/notify`
- **Purpose**: Main async callback endpoint for subscription orders. After signature verification, it completes and activates the subscription order.
- **Example request in Apifox**:

```http
POST /api/subscription/epay/notify
Content-Type: application/x-www-form-urlencoded

trade_status=TRADE_SUCCESS&service_trade_no=SUBxxxx&sign=xxxx&...
```

- **Testing notes**:
  - Correctly signed data is required. Forged parameters do not pass signature verification.
  - Subscription activation should be judged primarily by the notify processing result.

### 20. Subscription Payment: Epay Async Callback (GET)

- **Full route path**: `GET {{baseUrl}}/api/subscription/epay/notify`
- **Purpose**: Compatibility endpoint for GET-mode subscription callbacks. Logic is the same as POST notify.
- **Example request in Apifox**:

```http
GET /api/subscription/epay/notify?trade_status=TRADE_SUCCESS&service_trade_no=SUBxxxx&sign=xxxx&...
```

- **Testing notes**:
  - Complete signature parameters are also required.

### 21. Subscription Payment: Epay Browser Return (GET)

- **Full route path**: `GET {{baseUrl}}/api/subscription/epay/return`
- **Purpose**: Browser GET return endpoint after payment completion. It displays the payment result and attempts to complete the order as a fallback.
- **Example request in Apifox**:

```http
GET /api/subscription/epay/return?trade_status=TRADE_SUCCESS&service_trade_no=SUBxxxx&sign=xxxx&...
```

- **Testing notes**:
  - This is mainly for frontend redirect feedback. Core accounting should still rely on notify.

### 22. Subscription Payment: Epay Browser Return (POST)

- **Full route path**: `POST {{baseUrl}}/api/subscription/epay/return`
- **Purpose**: Compatibility endpoint for payment gateways that return to the browser through POST. Logic is the same as GET return.
- **Example request in Apifox**:

```http
POST /api/subscription/epay/return
Content-Type: application/x-www-form-urlencoded

trade_status=TRADE_SUCCESS&service_trade_no=SUBxxxx&sign=xxxx&...
```

- **Testing notes**:
  - The return endpoint mainly improves user experience and fallback behavior. Do not use it as a replacement for async notifications.

### 23. Subscription Payment: View My Subscription Status

- **Full route path**: `GET {{baseUrl}}/api/subscription/self`
- **Purpose**: Queries the current user's subscription information, including current active subscriptions and historical subscriptions.
- **Example request in Apifox**:

```http
GET /api/subscription/self
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```

- **Testing notes**:
  - This is commonly used after payment success to confirm whether the subscription is active.

### 24. Subscription Payment: Update Subscription Preference

- **Full route path**: `PUT {{baseUrl}}/api/subscription/self/preference`
- **Purpose**: Updates user subscription preference settings, such as the default subscription source strategy.
- **Example request in Apifox**:

```http
PUT /api/subscription/self/preference
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "billing_preference": "subscription_first"
}
```

- **Testing notes**:
  - Parameter enums should follow the API response or backend validation rules. Invalid values return parameter errors.
