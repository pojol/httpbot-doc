---
description: 本页将简单介绍如何使用httpbot进行测试
---

# 快速开始

[Code](https://github.com/pojol/httpbot/tree/master/sample)

```go
func main() {
	rand.Seed(time.Now().UnixNano())

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

