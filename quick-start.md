---
description: 本页将简单介绍如何使用httpbot进行测试
---

# 快速开始



```go

var (
	help bool

	// robot number
	num int

	// lifttime 生命周期
	lifetime int

	// increase 增量
	increase bool
)

func initFlag() {
	flag.BoolVar(&help, "h", false, "this help")

	flag.BoolVar(&increase, "increase", false, "incremental robot in every second")
	flag.IntVar(&lifetime, "lifetime", 60, "life time by second")
	flag.IntVar(&num, "num", 0, "robot number")
}

func main() {

	initFlag()

	flag.Parse()
	if help {
		flag.Usage()
		return
	}

	fmt.Println("bot num", num)
	fmt.Println("increase", increase)
	fmt.Println("lifetime", lifetime)

	rand.Seed(time.Now().UnixNano())

	mode := ""
	if increase {
		mode = factory.FactoryModeIncrease
	} else {
		mode = factory.FactoryModeStatic
	}

  // 覆盖率报告用，传入的match url 代表需要测试的api请求地址列表。
	var matchUrls []string
	for _, v := range prefab.Urls {
		matchUrls = append(matchUrls, v)
	}

	bf, _ := factory.Create(
		factory.WithCreateNum(num),		// 没批次创建的bot数量，传入0会设置为当前append strategy数量
		factory.WithLifeTime(time.Duration(lifetime)*time.Second), // 工厂运行时间
		factory.WithMatchUrl(matchUrls), 
		factory.WithRunMode(mode), // 累增模式 ｜ 定量模式
	)
	defer bf.Close()

  // 添加 strategy ， 这里将返回一个创建bot的func。 用于装配不同策略的bot
	bf.Append("default", func(url string, client *http.Client) *httpbot.Bot {
		// 构建 metadata 结构
    md, err := prefab.NewBotData()
		if err != nil {
			panic(err)
		}

    // 创建一个bot
		bot := httpbot.New("default", client, md)

    // 为bot 装配运行逻辑
		bot.Timeline.AddStep(sstep.NewDefaultStep(md))

		return bot
	})

	bf.Run()

	ch := make(chan os.Signal, 1)
	signal.Notify(ch, syscall.SIGTERM, syscall.SIGINT)
	<-ch
}

```



```shell
# 编译
$ go build -o bot main.go

# 定量模式运行， 运行1000个bot
$ ./bot -num 1000

# 累增模式运行
# 每秒运行100个bot，累计执行60秒
$ ./bot -num 100 -lifetime 60 -increase true
```

