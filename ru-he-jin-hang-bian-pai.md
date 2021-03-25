# 如何进行编排



1. 定义 Metadata
2. 创建Card
3. 创建Step
4. 创建Strategy
5. 装配到工厂，并启动



#### 定义 Metadata

> metadata 是bot 的数据集，用户可以任意扩展这个结构用于适配自己的应用程序的逻辑需求。
>
> 它被bot持有，并被所有的Card引用。

```go
// BotDat bot的metadata
type BotDat struct {
	Token string
	AccID string
}

// NewBotData 创建bot metadata
func NewBotData() (*BotDat, error) {
	b := &BotDat{}
	return b, nil
}
```



#### 创建Card

> Card 代理了每个 http 请求，它主要被用于复用在各个策略逻辑以及不同的编排上

它主要定义了

* API的`调用方式`
* API的`参数定义` & `请求｜回复` 的打包和解析
* 参数注入（用于在不同的策略和编排下，注入api中变量的值
* 断言注入（用于在不同的策略和编排下，断言回复后参数的校验

```go
// LoginGuestCard 游客登录
type LoginGuestCard struct {
	Base  *card.Card
	URL   string
	delay time.Duration
	md    *prefab.BotDat
}

// NewGuestLoginCard 生成账号创建预制
func NewGuestLoginCard(md *prefab.BotDat) *LoginGuestCard {
	return &LoginGuestCard{
		Base:  card.NewCardWithConfig(),
		URL:   prefab.Urls[prefab.LoginGuest],
		delay: time.Millisecond,
		md:    md,
	}
}

func (card *LoginGuestCard) GetName() string { return prefab.LoginGuest }
func (card *LoginGuestCard) GetURL() string { return card.URL }
func (card *LoginGuestCard) GetClient() *http.Client { return nil }
func (card *LoginGuestCard) GetHeader() map[string]string { return card.Base.Header }
func (card *LoginGuestCard) GetMethod() string { return card.Base.Method }
func (card *LoginGuestCard) SetDelay(delay time.Duration) { card.delay = delay }
func (card *LoginGuestCard) GetDelay() time.Duration { return card.delay }
func (card *LoginGuestCard) Enter() []byte {

	b := []byte{}

	card.Base.AddInjectAssert("token assert", func() error {
		return assert.NotEqual(card.md.Token, "")
	})

	return b
}

// Leave 反序列化返回消息
func (card *LoginGuestCard) Leave(res *http.Response) error {

	var err error
	var body []byte
	cres := LoginGuestRes{}
  
	body, _ = ioutil.ReadAll(res.Body)
	err = json.Unmarshal(body, &cres)
	if err != nil {
		err = fmt.Errorf("%v json.Unmarshal err %v", card.GetURL(), err.Error())
		goto EXT
	}

	card.md.Token = cres.Token
	err = card.Base.Assert()

EXT:
	return err
}

```



#### 创建Step

> step 用于编排一段执行逻辑
>
> 例:  我们创建一个account step 他将被复用到不同的strategy中（因为登陆是每个bot必须执行的步骤
>
> 同样的我们可以编排任意多的step，用于复用在各种不同的strategy中。

```go
step := prefab.NewStep()

# 登录
step.AddCard(NewGuestLoginCard(md))

# 获取帐号数据
aic := NewAccountGetInfoCard(md)
# 注入一个参数变量，这个注入函数会在 card.enter 头部执行
aic.AddInjectParm("Token", func()interface{} { return md.Acc.Token })
# 注入一个断言，这个断言会在 card.leave 尾部执行
aic.AddInjectAssert("account id assert", 
       func()error{ return assert.NotEqual(md.Acc.ID, "", errcode) },
)
step.AddCard(aic)

# ...

return step
```



#### 创建Strategy

> strategy bot的创建函数，用于定制不同的策略&行为模式

```go
func StrategyMail() *Bot {
  md, _ := prefab.NewBotData()
  
  bot := bot.New(bot.BotConfig{ ... })
  
  // 创建一个登录步骤（这样才能将用户数据拿到本地bot的metadata中
  bot.Timeline.AddStep(prefab.NewAccountLoginStep(md))
  // 创建一个发送邮件的步骤 （用于测试验证mail的逻辑
  bot.Timeline.AddStep(prefab.NewMailSendStep(md))
  
  return bot
}
```



#### 装配到工厂，并启动

> factory 机器人的批量创建工厂
>
> 主要用于定义bot的创建模式，运行时间，以及各种运行参数。

```go
f , _ := factory.Create( opts ... )

f.Append("strategy_mail", StrategyMail)
// ...

defer f.Close()
f.Run()
```



