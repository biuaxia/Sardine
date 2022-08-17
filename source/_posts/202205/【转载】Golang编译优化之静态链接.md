title: 【转载】Golang编译优化之静态链接
date: 2022-05-26 14:34:00
toc: false
index_img: https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rAOrGvBASaprDnwukcq5GutBULkN9WJT9A/0
category:
- Go
tags:
- 资料
- Java
- 项目
- 文件
- Golang
- 解决
- 方案
- 格式
- 不同
- 出现
---

> 原文：[Golang编译优化之静态链接 - FranzKafka Blog](https://coderfan.net/optimization-golang-compilation-with-statically-linked.html?utm_source=rss&utm_medium=rss&utm_campaign=optimization-golang-compilation-with-statically-linked)，本站仅作保存记录及调整排版，不做盈利性质的商业行为。

最近开始学习golang，对于go语言的一些特性相当满意。想起来与Python兴起时类似的那句话——人生苦短，我用Python。但Python终究只是解释型语言，而作为一个真正的程序员，不写编译型语言是不太合理的。在此我单方面宣布（开个玩笑啦~），不写编译型语言的程序员不是一个合格的程序员。那么，回到golang本身，也可以得出类似的一句话：*后端苦短，我用golang* ~

大概半个月前，我简单学习了一下golang的基础知识，就开始写了自己的第一个golang程序，无论是语言特性，公有、私有的界定，与Java类似的interface，自带的格式化工具，超丰富的标准库与大量第三方库，还是高效的编译、测试与运行，都让我对自己主力C++的选择产生了怀疑。这玩意儿这么好，写什么C++呀！

后来开始参与一个小的开源项目，自己添加了一些功能。编译之后跑了跑，没什么问题，于是就放心大胆的发布了release，结果很多童鞋反馈在一些老版本的系统上报错,编译报错的内容都是GLIBC_2.28 not found。Google了半天，发现类似的问题还真不少，于是就想着解决该问题，Google出来很多的答案无非就两条：要么升级系统，要么升级glibc依赖。那这两者我都不想选呢（因为很麻烦），有没有其他的办法可以解决整个问题呢。那这篇文章就是对这个问题的一个记录啦~

## Background：为什么会报错

在直奔主题找解决方案之前，首先要了解一下为什么会报这个错误。那么就从报错本身开始，Glibc是个什么东东呢，在维基百科上可以看到，其是我们Linux内核运行所依赖的一个基础组件库，提供很多标准的API接口。这些标准的接口如open、read、write、malloc、printf、dlopen等等，可想而知其有多重要了。

在不同的Linux发行版本中，Glibc的版本是不同的。这些不同的版本之间所提供的API功能自然也是有差异的。可是这跟go程序报错有什么关系呢。这里我们就不得不引出另一个内容了，那就是——CGO

CGO其实是为了在Go语言中调用C函数而设计的，我们都知道Linux或者说Unix内核都是由C语言实现的。本身C/C++经过多年的发展已经具备了很完备的生态，拥有大量的标准API。那么Go语言在设计时就想到了直接利用这些API或者说经过包装之后的C程序，要实现这样的机制，就必须通过CGO。以下为CGO使用的一个示例：

```
package cgoexample
/*
#include <stdio.h>
#include <stdlib.h>

void myprint(char* s) {
	printf("%s\n", s);
}
*/
import "C"

import "unsafe"

func Example() {
	cs := C.CString("Hello from stdio\n")
	C.myprint(cs)
	C.free(unsafe.Pointer(cs))
}
```

既然要通过CGO调用C提供的标准API接口，那么就可能会遇到由于Glibc库版本不同导致标准的API接口实现上的差异。对于Go程序而言这是不希望看到的。所以在很多用到了CGO的标准库中会对Glibc的版本存在要求。如net标准库，os/user标准库等等。

在我们的报错信息中，我们可以看到该程序应该依赖于Glibc 2.28版本。那么我们可以通过ldd –version命令确认一下本机的Glibc版本。

![](https://b3logfile.com/file/2022/05/b3fdd0a73f2c4365b818d4412925e8d8.png)

在出现问题的系统中，我们的Glibc版本为2.23。但是我们编译出来的go程序所依赖的Glibc版本要求为2.28及以上。由于Glibc版本过低，导致我们的程序无法正常运行并报错提醒。

## Solution:如何解决报错

在这里有几个可以选择的解决方案。

**方案一** ：升级最新的系统，一般而言各大Linux发行版本都会在发布最新的版本时集成最新的Glibc版本，所以升级系统可以获得最新的Glibc版本。下表是一些流行的Linux发行版本内集成的Glibc版本：

| glibc    | rhel | centos    | sle      | debian | ubuntu      | kylin            | manylinux  |
| -------- | ---- | --------- | -------- | ------ | ----------- | ---------------- | ---------- |
| 2021/2/1 | 2.33 |           |          |        |             | 21.04 hirsute    |            |
|          | 2.32 |           |          |        |             | 20.10 groovy     |            |
| 2020/2/1 | 2.31 |           |          |        | 11 bullseye | 20.04 LTS focal  |            |
|          | 2.3  |           |          |        |             | 19.10 eoan       |            |
|          | 2.29 |           |          |        |             | 19.04 disco      |            |
| 2018/8/1 | 2.28 | 8.3 Ootpa | 8.2.2004 |        | 10 buster   | 18.10 cosmic     | V10 Tercel |
| 2018/2/1 | 2.27 |           |          |        |             | 18.04 LTS bionic |            |
|          | 2.26 |           |          |        |             | 17.10 artful     |            |
|          | 2.26 |           |          | 15-SP2 |             |                  |            |
| 2016/8/4 | 2.24 |           |          |        | 9 stretch   | 17.04 zesty      |            |
|          |      |           |          |        |             | 16.10 yakkety    |            |

从表里我们可以看到，如果要使用Glibc2.28及以上的版本，你的系统应该为这几类：CentOS 8.1，Fedora 30，Gentoo – 2020-01-22、Arch Linux 2020.01.22、ubuntu 18.10、Debian 8

**方案二** ：升级Gilbc版本，实际上这是我不推荐的方式。因为Glibc是Linux系统内很重要的一个组件，无论是升级或者降级都有可能导致系统无法正常工作。当然，在某些情况下你可以通过单独升级Glibc的方式进行尝试。

**方案三** ：将go程序以静态链接的形式进行编译。在非交叉编译时，默认CGO是开启的，此时编译出来的程序将会以动态链接的形式生成可执行文件。我们可以通过file命令或者ldd命令检测确认该信息。如下所示：

![](https://b3logfile.com/file/2022/05/656be0e4febe41368a7a1bed1fc335f6.png)

动态链接本是为了将共享库以动态加载的形式进行链接，从而减少软件自身的体积并实现最大程度上的软件服用，但是一般会对宿主系统动态链接的共享库版本有要求。而静态链接则是在编译时以编译系统内包含的共享库作为资源进行链接，在运行时不再依赖系统的共享库。

那么，针对这个问题，我们可以采用全静态链接的方式进行编译。

使用如下命令开启CGO同时静态链接编译：

```
CGO_ENABLED=1  go build -trimpath  -ldflags='-extldflags=-static' -tags musl,osusergo,netgo,sqlite_omit_load_extension -o output -v main.go
```

编译成功后我们使用命令进行查看，发现已经是静态链接的了。

![](https://b3logfile.com/file/2022/05/9068f348397949a7b3ffddea5b7059f2.png)

之后我们将该执行文件放到老系统中进行运行，也能正常运行啦~

参考资料：

1.[https://www.arp242.net/static-go.html](https://www.arp242.net/static-go.html)

2.[https://www.kawabangga.com/posts/4495#comment-34735](https://www.kawabangga.com/posts/4495#comment-34735)
