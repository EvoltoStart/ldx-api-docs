---
title: "Subscription, Redemption Code, and Order Routes"
description: "Explains test routes and authentication conventions for subscription, redemption-code, and order-management APIs."
---

# Subscription, Redemption Code, and Order Management Routes (Apifox Test Version)

> Applicable scenario: You use your **own MySQL + Redis** and deploy the frontend and backend separately.

## Unified Test Conventions (Frontend and Backend Separated)

- Backend API address: `http://localhost:8005`
- Frontend address, used only for redirects: `http://localhost:3000`
- Recommended Apifox environment variables:
  - `{{baseUrl}} = http://localhost:8005`
  - `{{userJwt}} = logged-in user JWT/access token`
  - `{{userId}} = current logged-in user ID`
  - `{{adminJwt}} = administrator JWT/access token`
  - `{{adminId}} = administrator user ID`
  - `{{planId}} = subscription plan ID`
  - `{{tradeNo}} = order number`

### Authentication Notes

- When testing endpoints that use `UserAuth()`, such as `/api/subscription/*`, `/api/user/topup/*`, and `/api/user/pay`, it is recommended to always include:
  - `Authorization: Bearer {{userJwt}}`
  - `New-Api-User: {{userId}}`
- When testing administrator endpoints, such as `/api/redemption/*`, `/api/subscription/admin/*`, and `/api/user/topup`, it is recommended to include:
  - `Authorization: Bearer {{adminJwt}}`
  - `New-Api-User: {{adminId}}`

---

## Module 1: Subscription Routes (User -> Admin -> Callback Order)

### 1. Get Subscription Plans (User)

- **Full route path**: `GET http://localhost:8005/api/subscription/plans`
- **Purpose**: Gets the currently enabled subscription plans.
- **Example request in Apifox**:

```http
GET /api/subscription/plans
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```

- **Testing notes**: Only plans with `enabled=true` are returned.

### 2. Get Current User Subscription Status

- **Full route path**: `GET http://localhost:8005/api/subscription/self`
- **Purpose**: Views the current user's subscription information, including active and historical subscriptions.
- **Example request in Apifox**:

```http
GET /api/subscription/self
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```

- **Testing notes**: The response usually contains both `subscriptions` for active subscriptions and `all_subscriptions` for all subscriptions including expired ones.

### 3. Update Subscription Billing Preference

- **Full route path**: `PUT http://localhost:8005/api/subscription/self/preference`
- **Purpose**: Updates the user's billing preference.
- **Example request in Apifox**:

```http
PUT /api/subscription/self/preference
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "billing_preference": "subscription"
}
```

- **Testing notes**: Prefer sending one valid string value. Invalid values may be normalized or rejected by the backend.

### 4. Create Subscription Order with Epay (Payment Module)

- **Full route path**: `POST http://localhost:8005/api/subscription/epay/pay`
- **Purpose**: Creates an Epay order for a subscription.
- **Example request in Apifox**:

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

- **Testing notes**:
  - `plan_id` must be an enabled plan.
  - `payment_method` must be in the system payment-method whitelist.
  - A `pending` order is created first. Final success depends on the callback.

### 5. Create Subscription Order with Stripe (Payment Module)

- **Full route path**: `POST http://localhost:8005/api/subscription/stripe/pay`
- **Purpose**: Creates a Stripe Checkout payment link and a subscription order.
- **Example request in Apifox**:

```http
POST /api/subscription/stripe/pay
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "plan_id": {{planId}}
}
```

- **Testing notes**:
  - The plan must be configured with `StripePriceId`.
  - The backend must have valid Stripe keys and webhook settings.

### 6. Create Subscription Order with Creem (Payment Module)

- **Full route path**: `POST http://localhost:8005/api/subscription/creem/pay`
- **Purpose**: Creates a Creem payment link and a subscription order.
- **Example request in Apifox**:

```http
POST /api/subscription/creem/pay
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "plan_id": {{planId}}
}
```

- **Testing notes**:
  - The plan must be configured with `CreemProductId`.
  - In non-test mode, `CreemWebhookSecret` must be configured.

### 7. Admin: List Subscription Plans

- **Full route path**: `GET http://localhost:8005/api/subscription/admin/plans`
- **Purpose**: Allows administrators to view all plans, including disabled plans.
- **Example request in Apifox**:

```http
GET /api/subscription/admin/plans
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
```

- **Testing notes**: Administrator permission is required.

### 8. Admin: Create Subscription Plan

- **Full route path**: `POST http://localhost:8005/api/subscription/admin/plans`
- **Purpose**: Creates a new subscription plan.
- **Example request in Apifox**:

```http
POST /api/subscription/admin/plans
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
Content-Type: application/json

{
  "plan": {
    "title": "Pro Monthly",
    "subtitle": "Suitable for high-frequency usage",
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

- **Testing notes**:
  - `title` cannot be empty.
  - `price_amount` cannot be less than 0 or excessively large.

### 9. Admin: Update or Enable/Disable Subscription Plan

- **Full route paths**:
  - `PUT http://localhost:8005/api/subscription/admin/plans/{id}`
  - `PATCH http://localhost:8005/api/subscription/admin/plans/{id}`
- **Purpose**: Updates plan details or only updates the enabled status.
- **Example request in Apifox**:

```http
PATCH /api/subscription/admin/plans/1
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
Content-Type: application/json

{
  "enabled": false
}
```

- **Testing notes**: `PATCH` only processes the status field, while `PUT` performs full-field validation.

### 10. Subscription Epay Callback (Server Side)

- **Full route paths**:
  - `POST http://localhost:8005/api/subscription/epay/notify`
  - `GET  http://localhost:8005/api/subscription/epay/notify`
  - `GET  http://localhost:8005/api/subscription/epay/return`
  - `POST http://localhost:8005/api/subscription/epay/return`
- **Purpose**: Handles Epay asynchronous notifications and synchronous returns, completes order accounting, and redirects as needed.
- **Example request in Apifox**:

```http
POST /api/subscription/epay/notify
Content-Type: application/x-www-form-urlencoded

trade_no=platform_order_no&out_trade_no=system_order_no&money=9.90&name=SUB:Pro&trade_status=TRADE_SUCCESS&sign=signature&sign_type=MD5
```

- **Testing notes**:
  - These endpoints are essentially payment-platform callbacks. Manual testing should use a sandbox or mocked signature environment.
  - Incorrect signatures fail immediately.

---

## Module 2: Redemption-Code Routes (User Redemption -> Admin Code Management)

### 1. User Redeems a Top-up Code

- **Full route path**: `POST http://localhost:8005/api/user/topup`
- **Purpose**: Allows a user to top up an account with a redemption code, without third-party payment.
- **Example request in Apifox**:

```http
POST /api/user/topup
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "key": "xxxx-xxxx-xxxx"
}
```

- **Testing notes**:
  - The same user is locked during the same time window. Duplicate concurrent requests may return `top up processing`.
  - Invalid, used, or expired redemption codes fail.

### 2. Admin Gets Redemption Code List

- **Full route path**: `GET http://localhost:8005/api/redemption/`
- **Purpose**: Queries all redemption codes with pagination.
- **Example request in Apifox**:

```http
GET /api/redemption/?p=0&page_size=20
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
```

- **Testing notes**: Administrator permission is required.

### 3. Admin Searches Redemption Codes

- **Full route path**: `GET http://localhost:8005/api/redemption/search`
- **Purpose**: Searches redemption codes by keyword.
- **Example request in Apifox**:

```http
GET /api/redemption/search?keyword=Pro&p=0&page_size=20
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
```

- **Testing notes**: Use fixed pagination parameters to make total-count changes easier to verify.

### 4. Admin Creates Redemption Codes

- **Full route path**: `POST http://localhost:8005/api/redemption/`
- **Purpose**: Batch creates redemption codes. The backend returns the generated key list.
- **Example request in Apifox**:

```http
POST /api/redemption/
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
Content-Type: application/json

{
  "name": "Campaign Batch A",
  "quota": 500000,
  "count": 3,
  "expired_time": 1893456000
}
```

- **Testing notes**:
  - `count` must be greater than 0 and no more than 100.
  - `expired_time` cannot be earlier than the current time. `0` usually means no expiration.

### 5. Admin Updates Redemption Code

- **Full route path**: `PUT http://localhost:8005/api/redemption/`
- **Purpose**: Updates redemption-code basic information or only updates status.
- **Example request in Apifox**:

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

- **Testing notes**:
  - Add `status_only` when only changing status.
  - Full updates require a valid `expired_time`.

### 6. Admin Deletes Redemption Codes

- **Full route paths**:
  - `DELETE http://localhost:8005/api/redemption/{id}`
  - `DELETE http://localhost:8005/api/redemption/invalid`
- **Purpose**: Deletes a specified redemption code or batch deletes invalid redemption codes.
- **Example request in Apifox**:

```http
DELETE /api/redemption/invalid
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
```

- **Testing notes**: `/invalid` may affect many records. Back up first or operate in a test database.

---

## Module 3: Order Management Routes (User Query -> Order Creation -> Admin)

### 1. Get Payment Configuration and Optional Amounts

- **Full route path**: `GET http://localhost:8005/api/user/topup/info`
- **Purpose**: Gets payment switches, minimum top-up quota, payment methods, and related settings.
- **Example request in Apifox**:

```http
GET /api/user/topup/info
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```

- **Testing notes**: This endpoint returns configuration only and does not create an order.

### 2. User Queries Own Orders

- **Full route path**: `GET http://localhost:8005/api/user/topup/self`
- **Purpose**: Allows the current user to view top-up orders with pagination.
- **Example request in Apifox**:

```http
GET /api/user/topup/self?p=0&page_size=20&keyword=USR
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
```

- **Testing notes**: `keyword` can be used to search by order number.

### 3. Calculate Payable Amount Before Ordering

- **Full route path**: `POST http://localhost:8005/api/user/amount`
- **Purpose**: Calculates payable amount based on top-up quota and user-group discount.
- **Example request in Apifox**:

```http
POST /api/user/amount
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "amount": 50
}
```

- **Testing notes**: Requests below the system minimum top-up amount fail directly.

### 4. Create Epay Order

- **Full route path**: `POST http://localhost:8005/api/user/pay`
- **Purpose**: Creates a regular top-up Epay order.
- **Example request in Apifox**:

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

- **Testing notes**:
  - A `pending` order is created.
  - Whether the amount is credited depends on the `/api/user/epay/notify` callback.

### 5. Create Stripe or Creem Order

- **Full route paths**:
  - `POST http://localhost:8005/api/user/stripe/pay`
  - `POST http://localhost:8005/api/user/creem/pay`
- **Purpose**: Creates a third-party payment order and returns the payment link.
- **Example request in Apifox**:

```http
POST /api/user/stripe/pay
Authorization: Bearer {{userJwt}}
New-Api-User: {{userId}}
Content-Type: application/json

{
  "amount": 50
}
```

- **Testing notes**: Stripe/Creem keys and callbacks must be configured in the backend first.

### 6. Payment Callback for Regular Top-up

- **Full route paths**:
  - `POST http://localhost:8005/api/user/epay/notify`
  - `GET  http://localhost:8005/api/user/epay/notify`
  - `POST http://localhost:8005/api/stripe/webhook`
  - `POST http://localhost:8005/api/creem/webhook`
- **Purpose**: Receives asynchronous payment-gateway notifications and updates order status.
- **Example request in Apifox**:

```http
POST /api/stripe/webhook
Stripe-Signature: t=...,v1=...
Content-Type: application/json

{ "id": "evt_xxx", "type": "checkout.session.completed" }
```

- **Testing notes**:
  - Use payment-platform test mode when possible.
  - Webhook processing fails if the signature is invalid.

### 7. Admin Views All Platform Orders

- **Full route path**: `GET http://localhost:8005/api/user/topup`
- **Purpose**: Allows administrators to view all top-up orders with pagination.
- **Example request in Apifox**:

```http
GET /api/user/topup?p=0&page_size=50&keyword=USR
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
```

- **Testing notes**: Administrator permission is required.

### 8. Admin Manually Completes an Order

- **Full route path**: `POST http://localhost:8005/api/user/topup/complete`
- **Purpose**: Manually completes an order by order number, usually when a callback was lost.
- **Example request in Apifox**:

```http
POST /api/user/topup/complete
Authorization: Bearer {{adminJwt}}
New-Api-User: {{adminId}}
Content-Type: application/json

{
  "trade_no": "USR1NOabc123"
}
```

- **Testing notes**:
  - The same order is protected by a mutual-exclusion lock to avoid concurrent manual completion.
  - Completed orders or orders in invalid states return errors.

---

## Additional Recommendations (MySQL + Redis + Frontend/Backend Separated)

- Verify backend configuration first, including payment keys, callback URLs, subscription plans, and redemption-code expiration strategy.
- In Apifox, prefer the real backend domain or port. Do not mix it with the frontend address.
- Callback endpoints should be forwarded to `localhost:8005` through an intranet tunnel or gateway, and all signature fields must be complete.
