title: GOPROXY变量中的direct的作用是什么
date: 2021-11-17 08:53:00
toc: true
category:
- Golang
tags:
- golang
- Golang
- 转载
- Go
- go
- goproxy
- GOPROXY
- direct
- 作用
- 变量
---

本文转载自：[GOPROXY变量中的direct的作用是什么 · Issue #21 · goproxy/goproxy](https://github.com/goproxy/goproxy/issues/21)

---

目前在使用这个项目搭建go代理的时候发现，GORPOXY变量设置为 https://goproxy.cn,direct 的话，golang.org 下的包不会走代理而是直连最终超时，而去掉direct的话就一切正常。以下是构建代理程序的代码，是我哪里搞错了吗？

![Snipaste](https://b3logfile.com/file/2021/11/d8bc0dc251dc4f97a7a70eab660a0570.png)`GOPROXY` 环境变量逗号列表的作用是，如果请求前一个时响应了 404 或 410 那么自动 fallback 到下一个，其中 `direct` 的作用是直接回源（源指的是比如 GitHub 这种代码托管服务）拉取代码。

如果你有公司内部的模块需要服务到，比如通过了 `GOPRIVATE` 等环境变量来指定，那么你是必须在 `GOPROXY` 中包含 `direct` 的，否则 `GOPRIVATE` 之类的不能正常工作。

此外你的环境变量配置的确存在问题，主要是 `https_proxy` 这个环境变量配置错了。如果你配置了 `https_proxy`，那么 Go HTTP client 是会优先使用它而不是 `http_proxy`，此时使用 `https_proxy` 时会默认它的值是一个有正常签名 TLS 证书的可以通过 HTTPS 协议访问到的 URL，可你的是一个 `http://` 开头的错误 URL。换句话说你这么配置的话 Go HTTP client 所有请求都会失败，因为连 TLS 握手阶段都过不去，我之前在 [goproxy/goproxy.cn#71 (comment)](https://github.com/goproxy/goproxy.cn/issues/71#issuecomment-599986830) 这里也有提到过。
