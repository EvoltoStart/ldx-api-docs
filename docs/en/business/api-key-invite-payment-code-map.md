---
title: "API Key, Invite Code, and Payment Code Locations"
description: "Maps API Key, invite-code, and payment-related logic to source-code locations."
---

# API Key, Invite Code, and Payment Code Locations

## Module 1: API Key

File: `middleware/auth.go`

```go
func TokenAuth() func(c *gin.Context) {
	return func(c *gin.Context) {
		if strings.Contains(c.Request.URL.Path, "/v1/messages") || strings.Contains(c.Request.URL.Path, "/v1/models") {
			anthropicKey := c.Request.Header.Get("x-api-key")
			if anthropicKey != "" {
				c.Request.Header.Set("Authorization", "Bearer "+anthropicKey)
			}
		}
		...
		token, err := model.ValidateUserToken(key)
		...
	}
}
```

Purpose: Extracts API Keys from multiple sources in a unified way and supports `Authorization`, `x-api-key`, `x-goog-api-key`, query `key`, and WebSocket protocol headers. The extracted key is ultimately passed to `ValidateUserToken` for user-token validation and permission control.

File: `model/token.go`

```go
type Token struct {
	UserId       int    `json:"user_id" gorm:"index"`
	Key          string `json:"key" gorm:"type:char(48);uniqueIndex"`
	RemainQuota int    `json:"remain_quota" gorm:"default:0"`
	Status       int    `json:"status" gorm:"default:1"`
	...
}

func ValidateUserToken(key string) (token *Token, err error) {
	if key == "" {
		return nil, errors.New("token is required")
	}
	token, err = GetTokenByKey(key, false)
	...
	if !token.UnlimitedQuota && token.RemainQuota <= 0 {
		...
		return token, errors.New("token quota has been exhausted")
	}
	return token, nil
}
```

Purpose: Defines the user API Token data model, including key, status, quota, and expiration fields. It also validates API Key legality and availability, including existence, status, expiration, and remaining balance.

File: `controller/token.go`

```go
func AddToken(c *gin.Context) {
	token := model.Token{}
	err := c.ShouldBindJSON(&token)
	...
	key, err := common.GenerateKey()
	...
	cleanToken := model.Token{
		...
		Key: key,
		...
	}
	err = cleanToken.Insert()
	...
}
```

Purpose: Entry point for user-side API Key lifecycle management, including create, query, search, and delete operations. A new key is generated and persisted during creation.

File: `router/api-router.go`

```go
tokenRoute := apiRouter.Group("/token")
tokenRoute.Use(middleware.UserAuth())
{
	tokenRoute.GET("/", controller.GetAllTokens)
	tokenRoute.GET("/search", middleware.SearchRateLimit(), controller.SearchTokens)
	tokenRoute.GET("/:id", controller.GetToken)
	tokenRoute.POST("/", controller.AddToken)
	tokenRoute.PUT("/", controller.UpdateToken)
	tokenRoute.DELETE("/:id", controller.DeleteToken)
}
```

Purpose: Exposes user API Key management routes.

File: `model/channel.go`

```go
type Channel struct {
	Key         string      `json:"key" gorm:"not null"`
	ChannelInfo ChannelInfo `json:"channel_info" gorm:"type:json"`
}

func (channel *Channel) GetNextEnabledKey() (string, int, *types.NewAPIError) {
	if !channel.ChannelInfo.IsMultiKey {
		return channel.Key, 0, nil
	}
	...
}
```

Purpose: Defines upstream channel API Keys, including single-key and multi-key modes, and handles multi-key rotation, random selection, and availability selection.

File: `relay/channel/api_request.go`

```go
func applyHeaderOverridePlaceholders(template string, c *gin.Context, apiKey string) (string, bool, error) {
	...
	if strings.Contains(template, "{api_key}") {
		template = strings.ReplaceAll(template, "{api_key}", apiKey)
	}
	...
}
```

Purpose: Injects the channel API Key into header override templates, such as the `{api_key}` placeholder, when forwarding requests to upstream providers. This adapts authentication headers for different channels.

---

## Module 2: Invite Code

File: `model/user.go`

```go
type User struct {
	AffCode   string `json:"aff_code" gorm:"type:varchar(32);column:aff_code;uniqueIndex"`
	AffCount  int    `json:"aff_count" gorm:"type:int;default:0;column:aff_count"`
	AffQuota  int    `json:"aff_quota" gorm:"type:int;default:0;column:aff_quota"`
	InviterId int    `json:"inviter_id" gorm:"type:int;column:inviter_id;index"`
}

func GetUserIdByAffCode(affCode string) (int, error) {
	...
	err := DB.Select("id").First(&user, "aff_code = ?", affCode).Error
	return user.Id, err
}

func inviteUser(inviterId int) (err error) {
	...
	user.AffCount++
	user.AffQuota += common.QuotaForInviter
	...
}
```

Purpose: Core invite-code data model and reward-settlement logic. It resolves the inviter from an invite code, tracks invite counts, and accumulates invite reward quota.

File: `model/user.go`

```go
func (user *User) Insert(inviterId int) error {
	...
	user.AffCode = common.GetRandomString(4)
	...
	if inviterId != 0 {
		if common.QuotaForInvitee > 0 {
			_ = IncreaseUserQuota(user.Id, common.QuotaForInvitee, true)
		}
		if common.QuotaForInviter > 0 {
			_ = inviteUser(inviterId)
		}
	}
	return nil
}
```

Purpose: Generates an invite code during user registration. If the user registers through an invite code, both the invitee and inviter receive their configured rewards.

File: `controller/user.go`

```go
func GetAffCode(c *gin.Context) {
	...
	if user.AffCode == "" {
		user.AffCode = common.GetRandomString(4)
		_ = user.Update(false)
	}
	c.JSON(http.StatusOK, gin.H{"data": user.AffCode})
}

func TransferAffQuota(c *gin.Context) {
	...
	err = user.TransferAffQuotaToQuota(tran.Quota)
	...
}
```

Purpose: Provides invite-code query/generation and invite-quota-to-account-quota transfer APIs.

File: `router/api-router.go`

```go
selfRoute.GET("/aff", controller.GetAffCode)
selfRoute.POST("/aff_transfer", controller.TransferAffQuota)
```

Purpose: Exposes invite-system API routes.

File: `web/src/components/auth/RegisterForm.jsx`

```jsx
let affCode = new URLSearchParams(window.location.search).get('aff');
if (affCode) {
  localStorage.setItem('aff', affCode);
}
...
if (!affCode) {
  affCode = localStorage.getItem('aff');
}
inputs.aff_code = affCode;
const res = await API.post(`/api/user/register?turnstile=${turnstileToken}`, inputs);
```

Purpose: The frontend registration page reads the `aff` parameter from the URL and submits it with the registration request to bind the invite relationship.

File: `web/src/components/topup/index.jsx`

```jsx
const getAffLink = async () => {
  const res = await API.get('/api/user/aff');
  if (success) {
    let link = `${window.location.origin}/register?aff=${data}`;
    setAffLink(link);
  }
};

const transfer = async () => {
  const res = await API.post(`/api/user/aff_transfer`, { quota: transferAmount });
  ...
};
```

Purpose: The frontend top-up/invite page displays the invite link and supports transferring invite quota.

---

## Module 3: Payment

File: `router/api-router.go`

```go
apiRouter.POST("/stripe/webhook", controller.StripeWebhook)
apiRouter.POST("/creem/webhook", controller.CreemWebhook)
...
userRoute.POST("/epay/notify", controller.EpayNotify)
userRoute.GET("/epay/notify", controller.EpayNotify)
...
selfRoute.POST("/pay", controller.RequestEpay)
selfRoute.POST("/stripe/pay", controller.RequestStripePay)
selfRoute.POST("/creem/pay", controller.RequestCreemPay)
selfRoute.GET("/topup/info", controller.GetTopUpInfo)
selfRoute.POST("/topup", controller.TopUp)
```

Purpose: Defines all top-up payment entry points and callback entry points for Epay, Stripe, and Creem. These routes include order creation, amount calculation, and webhook callbacks.

File: `model/topup.go`

```go
type TopUp struct {
	UserId        int
	Amount        int64
	Money         float64
	TradeNo       string
	PaymentMethod string
	Status        string
}

func Recharge(referenceId string, customerId string) (err error) {
	...
	if topUp.Status != common.TopUpStatusPending {
		return errors.New("invalid top-up order status")
	}
	topUp.Status = common.TopUpStatusSuccess
	...
	err = tx.Model(&User{}).Where("id = ?", topUp.UserId).Updates(... "quota": gorm.Expr("quota + ?", quota)).Error
	...
}
```

Purpose: Payment order model and core top-up accounting transaction. It prevents duplicate processing, updates order status, and increases user quota.

File: `controller/topup.go`

```go
func RequestEpay(c *gin.Context) {
	...
	uri, params, err := client.Purchase(&epay.PurchaseArgs{...})
	...
	topUp := &model.TopUp{
		...
		Status: "pending",
	}
	err = topUp.Insert()
	...
}

func EpayNotify(c *gin.Context) {
	...
	verifyInfo, err := client.Verify(params)
	...
	if verifyInfo.TradeStatus == epay.StatusTradeSuccess {
		LockOrder(verifyInfo.ServiceTradeNo)
		defer UnlockOrder(verifyInfo.ServiceTradeNo)
		topUp := model.GetTopUpByTradeNo(verifyInfo.ServiceTradeNo)
		...
		err = model.IncreaseUserQuota(topUp.UserId, quotaToAdd, true)
	}
}
```

Purpose: Handles Epay order creation and asynchronous callback processing, including signature verification, idempotent order locking, and quota addition after successful payment.

File: `controller/topup_stripe.go`

```go
func (*StripeAdaptor) RequestPay(c *gin.Context, req *StripePayRequest) {
	...
	payLink, err := genStripeLink(referenceId, user.StripeCustomer, user.Email, req.Amount, req.SuccessURL, req.CancelURL)
	...
	topUp := &model.TopUp{
		...
		PaymentMethod: PaymentMethodStripe,
		Status:        common.TopUpStatusPending,
	}
	...
}

func StripeWebhook(c *gin.Context) {
	payload, _ := io.ReadAll(c.Request.Body)
	event, err := webhook.ConstructEventWithOptions(payload, signature, endpointSecret, ...)
	...
}
```

Purpose: Creates Stripe payment links and verifies/handles Stripe webhook events.

File: `controller/topup_creem.go`

```go
func RequestCreemPay(c *gin.Context) {
	...
	err = c.ShouldBindJSON(&req)
	...
	creemAdaptor.RequestPay(c, &req)
}

// CreemWebhookEvent ...
```

Purpose: Provides the Creem order entry point and parses webhook event structures, completing order accounting together with the callback handling flow.

File: `web/src/components/topup/index.jsx`

```jsx
const onlineTopUp = async () => {
  ...
  if (payWay === 'stripe') {
    res = await API.post('/api/user/stripe/pay', {...});
  } else {
    res = await API.post('/api/user/pay', {...});
  }
  ...
  if (payWay === 'stripe') {
    window.open(data.pay_link, '_blank');
  } else {
    // Submit form to the third-party payment provider.
    form.submit();
  }
};
```

Purpose: Unified frontend payment submission flow. It calls different APIs based on the selected payment channel and redirects users to the payment page.

File: `web/src/components/topup/modals/PaymentConfirmModal.jsx`

```jsx
<Modal
  title={t('Top-up Confirmation')}
  onOk={onlineTopUp}
>
  ...
  <Text>{t('Payment Method')}:</Text>
</Modal>
```

Purpose: Payment confirmation modal that triggers the final payment action.
