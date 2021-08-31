title: 【转载】服务端请求伪造（SSRF）是什么？
date: 2021-08-14 09:01:57
toc: true
category:
 - 分享
tags:
 - 服务端
 - 伪造
 - SSRF
 - 请求
---

本文转载自：[服务端请求伪造（SSRF）是什么？](https://juejin.cn/post/6985086955247632398)

---

> * 原文地址：[What is Server-Side Request Forgery (SSRF)?](https://link.juejin.cn?target=https%3A%2F%2Fwww.acunetix.com%2Fblog%2Farticles%2Fserver-side-request-forgery-vulnerability%2F "https://www.acunetix.com/blog/articles/server-side-request-forgery-vulnerability/")
> * 原文作者：[Ian Muscat](https://link.juejin.cn?target=https%3A%2F%2Fwww.acunetix.com%2Fblog%2Fauthor%2Fianmuscat%2F "https://www.acunetix.com/blog/author/ianmuscat/")
> * 译文出自：[掘金翻译计划](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fxitu%2Fgold-miner "https://github.com/xitu/gold-miner")
> * 本文永久链接：[github.com/xitu/gold-m…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fxitu%2Fgold-miner%2Fblob%2Fmaster%2Farticle%2F2021%2Fserver-side-request-forgery-vulnerability.md "https://github.com/xitu/gold-miner/blob/master/article/2021/server-side-request-forgery-vulnerability.md")
> * 译者：[MoonBall](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMoonBall "https://github.com/MoonBall")
> * 校对者：[PassionPenguin](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FPassionPenguin "https://github.com/PassionPenguin")、[kamly](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fkamly "https://github.com/kamly")


<!-- more -->


服务端请求伪造（SSRF）漏洞让攻击者从存在漏洞的应用程序的后端服务中发送恶意请求。攻击者通常使用 SSRF 攻击位于防火墙之后的内部系统，这些内部系统是不能从外部网络访问的。攻击者也可以利用 SSRF 访问通过服务器的环回接口（(127.0.0.1）可访问的服务。

当攻击者可以完全控制或部分控制应用程序发送的请求时，SSRF 漏洞就出现了。常见的例子是攻击者能控制第三方服务的 URL，并且应用程序将发起该 URL 对应的请求。

以下是一个 PHP 例子，它易遭受 SSRF 攻击。

```php
<?php

/**
* 检查是否设置了 GET 变量 'url'
* 例如 - http://localhost/?url=http://testphp.vulnweb.com/images/logo.gif
*/
if (isset($_GET['url'])){
$url = $_GET['url'];

/**
* 发起易遭受 SSRF 攻击的请求因为
* 在发起请求之前
* 没有对 $url 做验证
*/
$image = fopen($url, 'rb');

/**
* 发送正确的响应头
*/
header("Content-Type: image/png");

/**
* 响应图片的内容
*/
fpassthru($image);}
```

在上面的例子中，攻击者可以完全控制 **url** 参数。攻击者可以向因特网上的任意网站和被攻击服务器（**localhost** ）上的资源发起 GET 请求。

在下面的例子中，攻击者向 Apache HTTP 服务发起一个请求。该 Apache HTTP 服务开启了 **mode_status** 模式（该模式默认是开启的）。

```http
GET /?url=http://localhost/server-status HTTP/1.1
Host: example.com
```

攻击者也可以使用 SSRF 向 web 服务器有权访问的其他内部资源发出请求，尽管它们不是公开可访问的。例如，攻击者可以访问类似 AWS/[Amazon EC2](https://link.juejin.cn?target=http%3A%2F%2Fdocs.aws.amazon.com%2FAWSEC2%2Flatest%2FUserGuide%2Fec2-instance-metadata.html "http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html") 和 [OpenStack](https://link.juejin.cn?target=https%3A%2F%2Fdocs.openstack.org%2Fadmin-guide%2Fcompute-networking-nova.html "https://docs.openstack.org/admin-guide/compute-networking-nova.html") 的云服务实例的元数据。攻击者甚至可以利用 SSRF 发挥创意，[对内部 IP 进行端口扫描](https://link.juejin.cn?target=https%3A%2F%2Fwww.acunetix.com%2Fblog%2Farticles%2Fssrf-vulnerability-used-to-scan-the-web-servers-network%2F "https://www.acunetix.com/blog/articles/ssrf-vulnerability-used-to-scan-the-web-servers-network/")。

```http
GET /?url=http://169.254.169.254/latest/meta-data/ HTTP/1.1
Host: example.com
```

除了 **http://** 和 **https://** URL 协议之外，攻击者可以利用鲜为人知的或古老的 URL 协议访问本地系统或内部网络上的文件。

下面的例子使用了 **file:///** 协议。

```http
GET /?url=file:///etc/passwd HTTP/1.1
Host: example.com
```

部分应用可能允许攻击者使用更多奇怪的 URL 协议。比如，如果应用使用 cURL 发起请求，那么攻击者可以使用 **dict://** URL 协议向任意的主机和端口发起请求，并发送自定义数据。

```http
GET /?url=dict://localhost:11211/stat HTTP/1.1
Host: example.com
```

上面的请求使得应用程序连接 **localhost** 的 11211 端口，并发送 `stat` 字符串。端口 11211 是 [Memcached](https://link.juejin.cn?target=https%3A%2F%2Fmemcached.org%2F "https://memcached.org/") 的默认端口，该端口通常不会对外暴露。

## 检查服务端请求伪造

为了自动检测服务端请求伪造，你需要使用一个中间服务，因为检查这种漏洞需要一种外部可访问的并存在时延的方式。Acunetix 通过将 [AcuMonitor](https://link.juejin.cn?target=https%3A%2F%2Fwww.acunetix.com%2Fvulnerability-scanner%2Facumonitor-technology%2F "https://www.acunetix.com/vulnerability-scanner/acumonitor-technology/") 作为中间服务解决该问题。

在扫描期间，Acunetix 向不同的 AcuMonitor URL 发起请求。如果 AcuMonitor 在这些不同的 URL 中收到一个请求，它将通知 Acunetix。然后 Acunetix 经发出 SSRF 警报。

下图是使用 AcuMonitor 的 Acunetix 扫描结果，它检查了服务端请求伪造。该警报包含 HTTP 请求的信息。它包括发起请求的服务器 IP 地址和请求中携带的 `User-Agent` 字符串（如果存在）。这些信息可以帮助开发者定位问题源头并修复问题。

![](https://b3logfile.com/file/2021/08/solo-fetchupload-7832292190415916248-6bdd008b.webp)

## 消除服务端请求伪造

将简单的黑名单和正则表达式应用于用户输入是消除 SSRF 攻击的一种糟糕方法。通常，黑名单是一种几乎无效的安全控制手段。因为攻击者总是可以找到方法去绕过它们。在这个例子中，攻击者可以使用 HTTP 重定向、通配符 DNS 服务（比如 **xip.io** ）或者甚至是 [可替代的 IP 编码](https://link.juejin.cn?target=http%3A%2F%2Fwww.pc-help.org%2Fobscure.htm "http://www.pc-help.org/obscure.htm")实现绕过黑名单。

### 白名单和 DNS 解析

最稳健的避免服务端请求伪造的方式是将你的应用需要访问的 DNS 名称和 IP 地址添加到白名单中。如果白名单机制不能满足你的需求，并且你必须依赖黑名单机制，那么对用户输入进行合理的校验就显得非常重要。比如，不允许向私有的 IP 地址发起请求（私用 IP 地址参考 [RFC 1918](https://link.juejin.cn?target=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc1918 "https://tools.ietf.org/html/rfc1918")）。

然而，在使用黑名单的场景中，正确的消除 SSRF 的方式将因应用而异。换句话说，没有使用黑名单机制修复 SSRF 漏洞的银弹，因为它高度依赖应用功能和业务需求。

### 处理响应

避免响应数据泄露给攻击者，你必须确保收到的响应是期望的内容。在任何情况下，都不应将服务器发送的请求的原始响应体传递给客户端。

### 禁用不使用的 URL 协议

如果你的应用仅使用 HTTP 或 HTTPS 发起请求，那么就只允许这些 URL 协议。如果你禁用掉不使用的 URL 协议，攻击者将不能使用具有潜在危险的协议发起请求，比如：**file:///** , **dict://** , **ftp://** 和 **gopher://** 。

### 内网服务进行身份认证

默认情况下，类似于 Memcached、Redis、Elasticsearch、MongoDB 的服务不要求身份认证。攻击者可以利用服务端请求伪造漏洞，无需身份认证便能访问这些服务。因此，为了确保应用程序的安全，最好尽可能启用身份认证，即使是本地网络上的服务。

> 如果发现译文存在错误或其他需要改进的地方，欢迎到 [掘金翻译计划](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fxitu%2Fgold-miner "https://github.com/xitu/gold-miner") 对译文进行修改并 PR，也可获得相应奖励积分。文章开头的 **本文永久链接** 即为本文在 GitHub 上的 MarkDown 链接。

---

> [掘金翻译计划](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fxitu%2Fgold-miner "https://github.com/xitu/gold-miner") 是一个翻译优质互联网技术文章的社区，文章来源为 [掘金](https://juejin.cn "https://juejin.cn") 上的英文分享文章。内容覆盖 [Android](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fxitu%2Fgold-miner%23android "https://github.com/xitu/gold-miner#android")、[iOS](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fxitu%2Fgold-miner%23ios "https://github.com/xitu/gold-miner#ios")、[前端](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fxitu%2Fgold-miner%23%25E5%2589%258D%25E7%25AB%25AF "https://github.com/xitu/gold-miner#%E5%89%8D%E7%AB%AF")、[后端](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fxitu%2Fgold-miner%23%25E5%2590%258E%25E7%25AB%25AF "https://github.com/xitu/gold-miner#%E5%90%8E%E7%AB%AF")、[区块链](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fxitu%2Fgold-miner%23%25E5%258C%25BA%25E5%259D%2597%25E9%2593%25BE "https://github.com/xitu/gold-miner#%E5%8C%BA%E5%9D%97%E9%93%BE")、[产品](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fxitu%2Fgold-miner%23%25E4%25BA%25A7%25E5%2593%2581 "https://github.com/xitu/gold-miner#%E4%BA%A7%E5%93%81")、[设计](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fxitu%2Fgold-miner%23%25E8%25AE%25BE%25E8%25AE%25A1 "https://github.com/xitu/gold-miner#%E8%AE%BE%E8%AE%A1")、[人工智能](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fxitu%2Fgold-miner%23%25E4%25BA%25BA%25E5%25B7%25A5%25E6%2599%25BA%25E8%2583%25BD "https://github.com/xitu/gold-miner#%E4%BA%BA%E5%B7%A5%E6%99%BA%E8%83%BD")等领域，想要查看更多优质译文请持续关注 [掘金翻译计划](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fxitu%2Fgold-miner "https://github.com/xitu/gold-miner")、[官方微博](https://link.juejin.cn?target=http%3A%2F%2Fweibo.com%2Fjuejinfanyi "http://weibo.com/juejinfanyi")、[知乎专栏](https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fjuejinfanyi "https://zhuanlan.zhihu.com/juejinfanyi")。
