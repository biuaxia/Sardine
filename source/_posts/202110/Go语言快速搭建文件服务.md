title: Go语言快速搭建文件服务
date: 2021-10-15 16:25:00
toc: true
category:
- Golang
- GoFrame
tags:
- Golang
- GoFrame
- 语言
- 快速
- 搭建
- 文件
- 服务
---

效果如图：

![Snipaste20211015162838.png](https://b3logfile.com/file/2021/10/Snipaste_2021-10-15_16-28-38-9196f174.png)

代码如下：

```go
package main

import "github.com/gogf/gf/frame/g"

// 静态文件服务器基本使用
func main() {
	s := g.Server()
	s.SetIndexFolder(true)
	s.SetServerRoot("C:\\Users\\biuaxia\\Desktop\\goProjects\\Tatisaurus")
	s.SetPort(8199)
	s.Run()
}
```

## 参考资料

* [静态文件服务 - GoFrame (ZH)-Latest - GoFrame官网 - 类似PHP-Laravel, Java-SpringBoot的Go企业级开发框架](https://goframe.org/pages/viewpage.action?pageId=1114172#id-%E9%9D%99%E6%80%81%E6%96%87%E4%BB%B6%E6%9C%8D%E5%8A%A1-%E7%A4%BA%E4%BE%8B2%EF%BC%8C%E9%9D%99%E6%80%81%E7%9B%AE%E5%BD%95%E6%98%A0%E5%B0%84)