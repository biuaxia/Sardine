title: 【转载】golang复用http.request.body
date: 2021-07-06 11:14:22
toc: true
category:
 - Golang
tags: 
 - 转载
 - Go
 - golang
 - 复用
 - http
 - request
 - body
---

本文转载自：[golang复用http.request.body - Go语言中文网 - Golang中文社区](https://studygolang.com/articles/15641)

---

## 问题及场景

业务当中有需要分发http.request.body的场景。比如微信回调消息只能指定一个地址，所以期望可以复制一份消息发给其他服务。由服务B和接收微信回调的服务A一起处理微信回调信息。


<!-- more -->


## 解决思路

最开始考虑的是直接转发http.request。使用[ReverseProxy](https://studygolang.com/static/pkgdoc/pkg/net_http_httputil.htm#ReverseProxy)直接将http.request由服务A转发给服务B。但是微信涉及到验证等问题，完全调整好非常麻烦。所以转换思路，打算将http.request.body的内容直接post给服务B。

可是http.request是readcloser。我们将http.request readAll的时候讲无法再次读取http.request里面的信息。

## 如何才能将http.request.body复制使用呢？

其中c表示的是http的上下文

```
    // 把request的内容读取出来
    var bodyBytes []byte
    if c.Request.Body != nil {
        bodyBytes, _ = ioutil.ReadAll(c.Request.Body)
    }
    // 把刚刚读出来的再写进去
    c.Request.Body = ioutil.NopCloser(bytes.NewBuffer(bodyBytes))
```

1.我们先将body从http.request里面读取出来，保存到一个变量里面。

2.然后再将变量里面的数据使用ioutil.NopCloser方法写回到http.request里面。

[https://golang.org/pkg/io/ioutil/#NopCloser](https://golang.org/pkg/io/ioutil/#NopCloser)

NopCloser returns a ReadCloser with a no-op Close method wrapping the provided Reader r.

NopCloser用一个无操作的Close方法包装Reader r返回一个ReadCloser接口。

这样我们就可以再次使用c.request来进行处理了。
