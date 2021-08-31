title: 【转载】搓一个 Telegram Bot
date: 2021-06-18 22:44:51
toc: true
category: 
 - Golang
tags:  
 - 转载
 - Telegram
 - bot
 - golang
---

本文转载自：[搓一个 Telegram Bot - bleem](https://mritd.com/2021/06/03/make-a-telegram-bot/#%E4%B8%80%E3%80%81Telegram-Bot-%E4%BB%8B%E7%BB%8D)

---

> 在一个月光如雪的晚上和 PMheart 在 Telegram 闲聊，突然发现群里一个人的昵称是当前时间，然后观察一会儿发现还在不停变化… 最可气的是他还弄个 “东半球🌏最准报时” 的头衔，我一开始以为是 Telegram 又出的什么 “高级功能”(毕竟微信炸💩都是 Telegram 好久之前玩剩下的)，几经 Google 我发现其实就是自己写个 Bot，然后定时 rename；**那我要不整个 “东半球最浪漫诗人” 岂不是太面了🤔。**


<!-- more -->


## 一、Telegram Bot 介绍

在 Telegram 官方的文档描述中，其 Bot Api 实质上分为两种，这两种 Api 用途也各不相同:

### 1.1、标准 Bot

由用户自行联系 `BotFather` (人如其名)交互式创建，该 Bot 是官方所认为的标准 Bot，其主要目的就是作为一个真正的 Bot；我们可以通过一个 Token 调用 Telegram Api 来控制它，玩法很多，包括不限于发送告警、作为群管机器人、交互式的帮你做各种自动化等等；同时这个 Bot 具有严格的隐私权限控制，比如拉到群里可以控制 Bot 对群消息是否可见等等(Telegram 这点做的非常 OJBK)；借助于这类 Bot，也有些脑洞大开的大哥在浪的边缘疯狂试探，比如下面的扫雷机器人:

![I3IcQn](https://b3logfile.com/file/2021/06/solo-fetchupload-3528900614686016519-72547db5.png)

还有算命的:

![pIZYSS](https://b3logfile.com/file/2021/06/solo-fetchupload-4621623704987758258-26c9e213.png)

Github 真正干活的:

![9wPxeI](https://b3logfile.com/file/2021/06/solo-fetchupload-8949129770885056173-7b2111ac.png)

我的证书告警:

![HAvQny](https://b3logfile.com/file/2021/06/solo-fetchupload-1935740738804096368-032295fd.png)

当然肯定有高铁动车组的(🔞我是正经人):

![1zmI2m](https://b3logfile.com/file/2021/06/solo-fetchupload-2632568028480916222-2f9d4db5.png)

### 1.2、客户端 Bot

准确的官方介绍是 `TDLib – build your own Telegram`，从这个介绍可以看出，这一个 “Bot” Api 本质上并不是让你写 Bot，而是作为开发一个第三方 Telegram 客户端用的；所以这个 Api 的权限很大，可以完整的模拟一个用户；目前我发现被滥用最多的就是用这个 Api 作为恶意拉人、发广告等，简直是币圈割韭菜御用。

**TDLib 本质上是一个 C++ 的 lib，官方提供了引导页面来帮助你用主流语言跨语言调用来使用它:**

![yTAC3k](https://b3logfile.com/file/2021/06/solo-fetchupload-5649015247844219212-aa4f04d6.png)

## 二、标准 Bot 使用

标准 Bot 使用相对简单，按照官方文档跟 `BotFather` 聊天创建一个即可:

![gPhI8O](https://b3logfile.com/file/2021/06/solo-fetchupload-619754570661955923-84bb6f6f.png)

当创建完成后在 Bot 设置界面你可以获取一个 `Token`，使用这个 Token 连接 Bot Api 地址就可以开始控制你的 Bot；Golang 开发可以考虑使用 [https://github.com/tucnak/telebot](https://github.com/tucnak/telebot) 这个库，用法相当简单:

```golang
package main

import (
	"log"
	"time"

	tb "gopkg.in/tucnak/telebot.v2"
)

func main() {
	b, err := tb.NewBot(tb.Settings{
		// You can also set custom API URL.
		// If field is empty it equals to "https://api.telegram.org".
		URL: "http://195.129.111.17:8012",

		Token:  "TOKEN_HERE",
		Poller: &tb.LongPoller{Timeout: 10 * time.Second},
	})

	if err != nil {
		log.Fatal(err)
		return
	}

	b.Handle("/hello", func(m *tb.Message) {
		b.Send(m.Sender, "Hello World!")
	})

	b.Start()
}
```

基于这个库，我为了方便使用写了一个命令行小工具，方便我发送告警信息等: [tgsend](https://github.com/mritd/tgsend)

![BCSMIV](https://b3logfile.com/file/2021/06/solo-fetchupload-5972546043744276113-420c14c2.png)

[![warOjJ](https://b3logfile.com/file/2021/06/solo-fetchupload-5883360954983905372-2300a763.png)

## 三、客户端 Api 使用

标准 Bot Api 很丰富，日常干活啥的也完全能满足，但是！**人如果不会装X那和咸鱼有什么区别？** 我的 “东半球最浪漫诗人” 得提上日程。

### 3.1、TDLib 构建

关于 Api 易用性，开发生态环境，这一点说实话，Telegram 能把所有国内 IM 按在地上摩擦，就像这样:

![iI1jz8](https://b3logfile.com/file/2021/06/solo-fetchupload-2452759843295746613-91ccbf0f.jpeg)

Telegram 官方提供了完整的 “点一点” 构建 TDLib 引导页面: [https://tdlib.github.io/td/build.html](https://tdlib.github.io/td/build.html)

勾选好自己的语言、操作系统、系统版本、甚至是编译的内存大小等设置后，无脑复制下面的命令执行就行:

![DnZdvD](https://b3logfile.com/file/2021/06/solo-fetchupload-6561102641364004344-b7875350.png)

### 3.2、Telegram API HASH

TDLib 构建完成后，需要自行申请一个 API_HASH，API_HASH 类似一个让 Telegram 识别你的客户端的 “合法标识”；API_HASH 申请需要登陆 [https://my.telegram.org/](https://my.telegram.org/)，然后选择 **API development tools** :

![qp2BjK](https://b3logfile.com/file/2021/06/solo-fetchupload-2773837645750729700-f4a6f72a.png)

然后填写相关信息，最后 Telegram 就会为你生成好 API_HASH:

![QGoh8Q](https://b3logfile.com/file/2021/06/solo-fetchupload-5581779735619185355-ecad5483.png)

### 3.3、TDLib 使用

TDLib 构建好了，API HASH 也有了，那么根据自己选择的语言找一个靠谱的 SDK 使用即可；比如 Golang 开发，我选择了 [https://github.com/Arman92/go-tdlib](https://github.com/Arman92/go-tdlib)，这个库使用相当简单:

```golang
package main

import (
	"fmt"

	"github.com/Arman92/go-tdlib"
)

func main() {
	tdlib.SetLogVerbosityLevel(1)
	tdlib.SetFilePath("./errors.txt")

	// Create new instance of client
	client := tdlib.NewClient(tdlib.Config{
		APIID:               "187786",
		APIHash:             "e782045df67ba48e441ccb105da8fc85",
		SystemLanguageCode:  "en",
		DeviceModel:         "Server",
		SystemVersion:       "1.0.0",
		ApplicationVersion:  "1.0.0",
		UseMessageDatabase:  true,
		UseFileDatabase:     true,
		UseChatInfoDatabase: true,
		UseTestDataCenter:   false,
		DatabaseDirectory:   "./tdlib-db",
		FileDirectory:       "./tdlib-files",
		IgnoreFileNames:     false,
	})

	for {
		currentState, _ := client.Authorize()
		if currentState.GetAuthorizationStateEnum() == tdlib.AuthorizationStateWaitPhoneNumberType {
			fmt.Print("Enter phone: ")
			var number string
			fmt.Scanln(&number)
			_, err := client.SendPhoneNumber(number)
			if err != nil {
				fmt.Printf("Error sending phone number: %v", err)
			}
		} else if currentState.GetAuthorizationStateEnum() == tdlib.AuthorizationStateWaitCodeType {
			fmt.Print("Enter code: ")
			var code string
			fmt.Scanln(&code)
			_, err := client.SendAuthCode(code)
			if err != nil {
				fmt.Printf("Error sending auth code : %v", err)
			}
		} else if currentState.GetAuthorizationStateEnum() == tdlib.AuthorizationStateWaitPasswordType {
			fmt.Print("Enter Password: ")
			var password string
			fmt.Scanln(&password)
			_, err := client.SendAuthPassword(password)
			if err != nil {
				fmt.Printf("Error sending auth password: %v", err)
			}
		} else if currentState.GetAuthorizationStateEnum() == tdlib.AuthorizationStateReadyType {
			fmt.Println("Authorization Ready! Let's rock")
			break
		}
	}

	// Main loop
	for update := range client.RawUpdates {
		// Show all updates
		fmt.Println(update.Data)
		fmt.Print("\n\n")
	}
}
```

由于 Telegram 是允许多客户端登陆的(**跟我一起喊:微信垃圾、张小龙垃圾** )，所以使用 TDLib 我们可以完全控制我们的账户行为；那么 “东半球最浪漫诗人” 实现就相对简单:

* 自己爬一点古诗名句: [https://github.com/mritd/poetbot/blob/master/poet.txt](https://github.com/mritd/poetbot/blob/master/poet.txt)
* 写点代码定时 rename: [https://github.com/mritd/poetbot/blob/master/main.go#L165](https://github.com/mritd/poetbot/blob/master/main.go#L165)

核心代码就这几行:

```golang
// 一个定时任务工具
cn := cron.New()

// 默认 30s 执行一次
_, err = cn.AddFunc(c.String("cron"), func() {
    rand.Seed(time.Now().Unix())
    // 随机取一句诗
    name := data[rand.Intn(len(data)-1)]
    logger.Infof("update name to [%s]...", name)
    // 调用 TDLib 改名
    _, err := client.SetName(name, "")
    if err != nil {
        logger.Error(err)
    }
})
```

效果嘛，就这样:

![A15b2P](https://b3logfile.com/file/2021/06/solo-fetchupload-1179901619433350304-039ba16d.png)

![3MXSCI](https://b3logfile.com/file/2021/06/solo-fetchupload-7921647283838209148-147b136f.png)

