# API KEY / 邀请码 / 支付 代码定位

## 模块一：API KEY

文件名：`middleware/auth.go`
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
作用：统一提取并兼容多种来源的 API Key（`Authorization`/`x-api-key`/`x-goog-api-key`/query key/WebSocket 协议头），最终走 `ValidateUserToken` 做用户令牌校验和权限控制。

文件名：`model/token.go`
```go
type Token struct {
	UserId     int    `json:"user_id" gorm:"index"`
	Key        string `json:"key" gorm:"type:char(48);uniqueIndex"`
	RemainQuota int   `json:"remain_quota" gorm:"default:0"`
	Status     int    `json:"status" gorm:"default:1"`
	...
}

func ValidateUserToken(key string) (token *Token, err error) {
	if key == "" {
		return nil, errors.New("未提供令牌")
	}
	token, err = GetTokenByKey(key, false)
	...
	if !token.UnlimitedQuota && token.RemainQuota <= 0 {
		...
		return token, errors.New("该令牌额度已用尽")
	}
	return token, nil
}
```
作用：定义用户 API Token 数据模型（密钥、状态、额度、过期时间等），并负责 API Key 合法性与可用性校验（存在性、状态、过期、余额）。

文件名：`controller/token.go`
```go
func AddToken(c *gin.Context) {
	token := model.Token{}
	err := c.ShouldBindJSON(&token)
	...
	key, err := common.GenerateKey()
	...
	cleanToken := model.Token{...
		Key: key,
		...
	}
	err = cleanToken.Insert()
	...
}
```
作用：用户侧 API Key 生命周期管理入口（创建/查询/搜索/删除等）；创建时生成新 key 并持久化。

文件名：`router/api-router.go`
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
作用：暴露用户 API Key 管理接口路由。

文件名：`model/channel.go`
```go
type Channel struct {
	Key string `json:"key" gorm:"not null"`
	ChannelInfo ChannelInfo `json:"channel_info" gorm:"type:json"`
}

func (channel *Channel) GetNextEnabledKey() (string, int, *types.NewAPIError) {
	if !channel.ChannelInfo.IsMultiKey {
		return channel.Key, 0, nil
	}
	...
}
```
作用：定义上游渠道 API Key（单 key / 多 key）以及多 key 轮询、随机、可用性选择逻辑。

文件名：`relay/channel/api_request.go`
```go
func applyHeaderOverridePlaceholders(template string, c *gin.Context, apiKey string) (string, bool, error) {
	...
	if strings.Contains(template, "{api_key}") {
		template = strings.ReplaceAll(template, "{api_key}", apiKey)
	}
	...
}
```
作用：在转发到上游时，把渠道 API Key 注入 Header 覆盖模板（例如 `{api_key}` 占位符），实现各渠道鉴权头适配。

---

## 模块二：邀请码

文件名：`model/user.go`
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
作用：邀请码核心数据模型与奖励结算逻辑（邀请码反查邀请人、邀请人数统计、邀请奖励额度累加）。

文件名：`model/user.go`
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
作用：用户注册时生成邀请码；若通过邀请码注册，则给被邀请人和邀请人分别发放奖励。

文件名：`controller/user.go`
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
作用：提供邀请码查询/生成接口，以及“邀请额度转账户额度”接口。

文件名：`router/api-router.go`
```go
selfRoute.GET("/aff", controller.GetAffCode)
selfRoute.POST("/aff_transfer", controller.TransferAffQuota)
```
作用：对外暴露邀请体系 API 路由。

文件名：`web/src/components/auth/RegisterForm.jsx`
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
作用：前端注册页读取 URL 中 `aff` 参数并随注册请求提交，实现邀请关系绑定。

文件名：`web/src/components/topup/index.jsx`
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
作用：前端“充值/邀请”页面展示邀请链接，并支持邀请额度划转。

---

## 模块三：支付相关

文件名：`router/api-router.go`
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
作用：定义全部充值支付入口与回调入口（易支付/Stripe/Creem），包含下单、金额计算、Webhook 回调。

文件名：`model/topup.go`
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
		return errors.New("充值订单状态错误")
	}
	topUp.Status = common.TopUpStatusSuccess
	...
	err = tx.Model(&User{}).Where("id = ?", topUp.UserId).Updates(... "quota": gorm.Expr("quota + ?", quota)).Error
	...
}
```
作用：支付订单模型与充值入账核心事务（防重复处理、更新订单状态、增加用户额度）。

文件名：`controller/topup.go`
```go
func RequestEpay(c *gin.Context) {
	...
	uri, params, err := client.Purchase(&epay.PurchaseArgs{...})
	...
	topUp := &model.TopUp{...
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
作用：易支付下单与异步回调处理（签名校验、订单加锁幂等、成功后加额度）。

文件名：`controller/topup_stripe.go`
```go
func (*StripeAdaptor) RequestPay(c *gin.Context, req *StripePayRequest) {
	...
	payLink, err := genStripeLink(referenceId, user.StripeCustomer, user.Email, req.Amount, req.SuccessURL, req.CancelURL)
	...
	topUp := &model.TopUp{...
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
作用：Stripe 支付链接创建与 Webhook 验签/事件处理。

文件名：`controller/topup_creem.go`
```go
func RequestCreemPay(c *gin.Context) {
	...
	err = c.ShouldBindJSON(&req)
	...
	creemAdaptor.RequestPay(c, &req)
}

// CreemWebhookEvent ...
```
作用：Creem 下单入口与 webhook 事件结构解析，配合后续回调处理完成订单入账。

文件名：`web/src/components/topup/index.jsx`
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
    // 提交表单到三方支付
    form.submit();
  }
};
```
作用：前端统一支付提交流程（按支付渠道调用不同接口并跳转支付页）。

文件名：`web/src/components/topup/modals/PaymentConfirmModal.jsx`
```jsx
<Modal
  title={t('充值确认')}
  onOk={onlineTopUp}
>
  ...
  <Text>{t('支付方式')}：</Text>
</Modal>
```
作用：支付确认弹窗，承接最终支付动作触发。
