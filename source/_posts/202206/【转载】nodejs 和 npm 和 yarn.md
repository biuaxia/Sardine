title: 【转载】nodejs 和 npm 和 yarn
date: 2022-06-06 11:09:00
toc: false
index_img: https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rILyI8ckeUFuZTtuo8Y2TUQ8JpxuBcS3aQ/0
category:
- 前端
tags:
- yarn
- nodejs
- npm
- global
- add
- remove
- dir
- config
- list
- registry
---

> 原文：[nodejs 和 npm 和 yarn - 掘金](https://juejin.cn/post/6844904085766963208#heading-12)，本站仅作保存记录及调整排版，不做盈利性质的商业行为。

## 1. nodejs 是个啥

Node.js 是能够在服务器端运行 JavaScript 的开放源代码、跨平台 JavaScript 运行环境。Node.js 大部分基本模块都用JavaScript语言编写。在 Node.js 出现之前，JavaScript 通常作为客户端程序设计语言使用，以 JavaScript 写出的程序常在用户的浏览器上运行，因此简单来说 Node.js 就是可以在服务器端运行的 JavaScript。

所以简单来说 node.js 是一个服务器端 JavaScript 的 IDE (集成开发环境)，简称 Node 环境

## 2. npm 是个啥

npm（全称 Node Package Manager，即“ node包管理器 ”）是 Node.js 默认的、以 JavaScript 编写的软件包管理系统。

具体关于 npm 的产生和发展，推荐方方老师的回答，请移步 [npm 是干什么的？（非教程）](https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F24357770 "https://zhuanlan.zhihu.com/p/24357770")

每种语言都有自己的包管理工具，npm 就是 JavaScript 语言的包管理工具，服务器端 JavaScript 通常使用 NPM 作为依赖管理工具，npm 虽然翻译过来是对 node.js 包的管理，毕竟二者达成合作，npm 依附 node.js 火的。但终究是 JavaScript 语言的包管理工具，并且前端开发都是以 JavaScript 语言为主，我们在做前端开发时也会用到很多别人写的 JavaScript 代码包，如 jQuery.js 、vue.js 等等，因此 npm 就成了学习前端开发不得不了解和掌握的一个优秀 js 包管理工具。

## 3. node 的安装

可能有人觉得说，我不使用 node.js 做后端开发，只做前端开发，因此只安装 npm 不行吗？

可以，但是 node.js 和 npm 都合作了，谁也离不开谁了，npm 自带在了 node.js 里，你安装个 node.js 就自动带有了 npm ，而且 node.js 目前这么火，想入前端这一行，怎么能保证不在某个时候用到 node.js 呢？因此建议还是安装个 node.js 来使用 npm 吧！

### 官网下载安装单个版本

[Node.js官网](https://link.juejin.cn?target=https%3A%2F%2Fnodejs.org%2Fzh-cn%2F "https://nodejs.org/zh-cn/")

尽量选择双数版本，如 Node8、Node10、Node12，双数版本是稳定版，选择下载 `.msi` 安装文件，x64 是64位系统，x86 是32位系统，安装步骤如下：

![](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rOVr6SPJZXlqUJOxBmR0HUJQAm8AdHOVsg/0)

![](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rCRIvwUNaYviac94icj4hyyyexXVgh30HbFw/0)

![](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rPKz5QWwNMQlpY6Ap5ibQznhmaE5gg0Bicjg/0)

![](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rLGhAF23OajONMpMukOoehagrGHu9ruQjw/0)

下载完成后点击直接安装，最好修改下安装目录，因为 node.js 的安装路径不允许有空格，在选项选择那里要全部勾选，尤其是添加到 path 那个选项，不过其默认是全选的，因此直接下一步就行，安装完成后运行命令 `node --version` 查看当前版本，若成功即可，未成功则看环境变量是否配置。

### 使用 NVM 版本管理工具

除了去官网下载安装 node.js 外，还可以使用 NVM 工具来控制 node.js 的版本切换，比如我最近就得将 node.js 升级，使用了网上推荐的 `n 模块` 管理工具，根本没什么鸟用，据说不支持 `windows` ，使用了 NVM 管理工具，妥妥的好用。

NVM 是一个 node.js 的版本管理工具，提供 node.js 下载安装和版本切换功能。

#### 1. 安装 NVM

windows版下载链接：[nvm-windows](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fcoreybutler%2Fnvm-windows%2Freleases "https://github.com/coreybutler/nvm-windows/releases")

进去找到 `nvm-setup.zip` ，下载后会得到 `.exe` 的安装程序，在安装的过程中会遇到两个目录选择，第一个是 nvm 工具的安装路径，第二个是当前使用的 node.js 的版本的路径，可自行设置，如下：

**第一个目录选择**

![](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rN7OLYiaUwGvEGlxOryoliceLTwyssCHniaZA/0)

**第二个目录选择**

![](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rILyI8ckeUFuZTtuo8Y2TUQ8JpxuBcS3aQ/0)

在安装好 NVM 后运行 `nvm --version` 成功查看到版本号及命令信息即安装成功

![](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rFSBvFd7jpML01icxichAQEibI0F385Pd4wTg/0)

#### 2. 切换 NVM 下载源

在安装完 nvm 后不要着急下载 node.js ，因为此时 node.js 和 npm  的默认下载源是国外，那叫一个慢啊，因此，要先修改 nvm 的默认下载源，运行如下命令：

```
nvm node_mirror https://npm.taobao.org/mirrors/node/
nvm npm_mirror https://npm.taobao.org/mirrors/npm/
```

#### 3. 开始使用 nvm

开始安装 node.js 吧，使用命令 `nvm install <version> [arch]` ，其中，version 是版本号，arch 是可选择的参数，是指操作系统位数，默认是 64 位，若你是 32 位的操作系统，则 arch 为 32。这些命令在 `nvm --version` 显示时都有说明，可看看。

```
nvm install latest             //安装最新版的node
nvm install 版本号             //64位操作系统
nvm list                       //查看当前已经安装的所有node版本
nvm use 版本号                 //版本切换
```

不知道有哪些版本号可安装？看这里 [nodejs-realease](https://link.juejin.cn?target=https%3A%2F%2Fnodejs.org%2Fzh-cn%2Fdownload%2Freleases%2F "https://nodejs.org/zh-cn/download/releases/")

## 4. npm 的使用

### 修改默认下载源

安装好了 node.js ，在命令行输入如下命令，看是不是自带了 npm

![](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rBLTYOBSJonHkicupiaqRWhtFE40WKP7az7g/0)

先不要着急使用 npm 安装或下载各种工具包，因为此刻 npm 的默认下载源是在国外，下载速度极慢，仍要将其的下载源设置为淘宝。这里要用到 npm 的下载源管理工具 `nrm`

1. npm 安装 nrm，运行如下命令
2. `npm i -g nrm`  //第一次只能从国外下载 nrm ，有点慢，等等
3. 下载完成后运行 `nrm --version` 成功看到版本号即安装成功
4. 运行 `nrm ls` 可查看到当前可用的下载源，如下，带星号 * 的就是当前在使用的，默认是 npm
5. `nrm use taobao` 选择使用淘宝下载源，即可！

![](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rDojPEjibrqYZbGHFjYAAHGSULyg9j7uxaA/0)

### npm 安装命令

**全局安装：**

`npm i -g 包名`

或

`npm install -g 包名`

PS: `i`是 `install`的缩写，`g`是 `global`的缩写

**全局卸载：**

`npm uninstall -g 包名`

用 npm 全局安装的包在哪里？

`C:\Users\Administrator\AppData\Roaming\npm\`

不一定一样，可以使用 `which 使用npm全局安装的包名` 来查看位置，如：

![](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rLwUwsTyxp9AicYJOJvZPGJ3FG2LHUKDW1w/0)

也可以如下使用命令查看，在 `node_modules` 里面

`npm config get prefix`

## 5. yarn 的安装

### yarn 安装

yarn 是一个新的 JavaScript 语言的包管理工具，可用来代替 npm ，相比于 npm 要更快速、更稳定好用，安装方法如下：

[Yarn官网](https://link.juejin.cn?target=https%3A%2F%2Fclassic.yarnpkg.com%2Fzh-Hans%2Fdocs%2Finstall%23windows-stable "https://classic.yarnpkg.com/zh-Hans/docs/install#windows-stable")

进去这个不用选择版本，其根据你的电脑版本已经推荐好了版本，直接点击下载安装程序 `.msi` 文件即可，直接安装，最好改下路径，路径中不要出现空格

![](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rDk8MUKqZENP8CK8bkhpa1nBUkIibVGsPSQ/0)

安装好 yarn 后 `yarn --version` 查看是否安装成功

yarn 也是默认从国外下载的，但是因为已经安装好了 npm 并配置好了 npm 的默认下载源，yarn 会自动检测到 npm 的存在，并且自动设置成和 npm 一样下载源。

`yarn config list` 查看 yarn 相关的配置和下载源 `registry`

`yarn config get registry` 查看当前下载源

若不是和 `npm` 一样，可以安装 yarn 对应的源管理工具设置，和 npm 的 `nrm` 的用法一样，yarn 对应的是 `yrm` 。

`yarn global add yrm`

然后 `yrm use taobao`

### yarn 命令

#### 1. 全局安装

`yarn global add 包名`

#### 2. 全局卸载

`yarn global remove 包名`

#### 3. 查看全局安装目录

`yarn global dir`
