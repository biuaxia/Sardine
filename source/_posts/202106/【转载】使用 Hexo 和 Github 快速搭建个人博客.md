title: 【转载】使用 Hexo 和 Github 快速搭建个人博客
date: 2021-06-18 22:16:32
toc: true
category: 
 - 转载
 - Hexo
tags: 
 - 转载
 - hexo
 - github
 - 博客
 - 快速
 - 搭建
 - 个人
---

本文转载自：[使用 Hexo 和 Github 快速搭建个人博客 | ９９３５](https://hooray.github.io/posts/15e2d21f/)

---

![](https://b3logfile.com/file/2021/06/solo-fetchupload-6371403681885890992-75fc5ab1.png)

网上类似的教程一搜一大把，写得其实很详细。但对于初次接触的我来说，这是一次全新的体验。所以在经历了多次「安装——配置——部署——删除——再安装——」之后，大概弄懂了整个流程。正好也是博客的开篇文章，记录一下整个搭建流程，加深印象。


<!-- more -->


## 安装

### 安装前提

在安装前，先确保电脑上已经安装了 [Node.js](http://nodejs.org/) 和 [Git](https://git-scm.com/) 环境。

### 安装 [Hexo](https://hexo.io/zh-cn/)

```bash
$ yarn global add hexo-cli
```

> 官方是使用 npm 进行安装，个人推荐使用 [yarn](https://yarnpkg.com/zh-Hans/) 进行安装，关于 yarn 和 npm 优劣分析，可以参考这篇文章《[Yarn vs npm: 你需要知道的一切](http://web.jobbole.com/88459/)》

### 先新建一个 `blog` 文件夹，然后进入该文件夹运行：初始化

```bash
$ hexo init
```

初始化完成后， `blog` 文件夹的目录如下：

![](https://b3logfile.com/file/2021/06/solo-fetchupload-591330291074510983-a039fc0a.png)

### 本地访问

继续运行：

```bash
$ hexo g && hexo s
```

运行成功后会发现提示信息：

> Hexo is running at http://localhost:4000/.

这个时候访问 [http://localhost:4000/](http://localhost:4000/) 就会发现网站已经建好了！

### 常用命令

以下 4 个是我在搭建过程中发现使用频率最高的 4 个命令，甚至我感觉只要会这 4 个命令就可以了。

#### generate

生成静态文件

```bash
$ hexo generate
```

#### server

启动服务器

```bash
$ hexo server
```

#### 部署网站deploy

```bash
$ hexo deploy
```

#### 清除缓存文件（`db.json`）和已生成的静态文件（`public`）clean

```bash
$ hexo clean
```

更多命令和参数请看 [这里](https://hexo.io/zh-cn/docs/commands.html) ！

### 部署到 Github

首先先到 Github 上新建一个仓库，仓库名的格式为：

```bash
<username>.github.io
```

例如我的就是 `hooray.github.io` ，创建好后，复制仓库的 `HTTPS` 地址，打开 `blog` 根目录下的 `_config.yml` 文件，拖动到底部找到 `deploy` 配置，按照以下格式修改并保存：

```bash
deploy:
  type: git
  repo: <仓库地址>
```

这个时候安装 [hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git) 自动部署发布工具

```bash
$ yarn add hexo-deployer-git
```

安装完成后，就可以去发布了

```bash
$ hexo clean && hexo g && hexo d
```

第一次发布会提示输入 Github 账号和密码：

![](https://b3logfile.com/file/2021/06/solo-fetchupload-8132833756091809324-819acc4e.png)

稍微等待一会，会提示发布成功，然后在浏览器里测试访问 [https://hooray.github.io](https://hooray.github.io/) ，大功告成！

![](https://b3logfile.com/file/2021/06/solo-fetchupload-1835648699475865504-ac070980.png)

这时候打开仓库会发现 Hexo 其实是把 `public` 目录下生成好的静态页面和相关资源上传到了 `master` 分支下，但是本地博客的开发环境是没有上传到仓库里的，如果换一台电脑想继续写博客更新，这就没办法了。

要解决这个问题其实也很简单，可以单独再创建一个仓库，专门用于上传开发环境。但我的做法是创建一个分支，比如 `hexo` 分支，这个分支专门用来上传开发环境。

需要注意，通过 git clone 下来的 NexT 主题，需要手动删除隐藏的 .git 文件夹，不然 NexT 主题整个文件夹都不会被提交。

## 主题

### 安装 [NexT](http://theme-next.iissnan.com/)

```bash
$ git clone https://github.com/theme-next/hexo-theme-next themes/next
```

我安装的是 `6.0.x`的 NexT ，但在 NexT 官网上提供的还是`5.1.x`的安装命令和文档，因为`6.0.x`和`5.1.x`的配置有所不同，建议第一次练习搭建还是使用`5.1.x`，熟悉配置文档后，再用`6.0.x` 。

修改 `_config.yml` 文件里 `theme` 配置：

```bash
theme: next
```

配置

虽然我安装的是 `6.0.x` ，但大部分还是可以根据 NexT [主题配置](http://theme-next.iissnan.com/theme-settings.html) 文档来修改，以下会介绍一些文档中没有提及的设置。

> 以下分别用：
>
> * `站点配置` _config.yml
> * `主题配置` themes/next/_config.yml
>
>   区分两个配置文件

#### 设置网站语言

打开 `站点配置` 找到 `language` 修改

```yaml
language: zh-CN
```

#### 开启导航

打开 `主题配置` 找到 `menu` ，将需要开启的导航前面的 `#` 删掉

```yaml
menu:
  home: / || home
  about: /about/ || user
  tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  commonweal: /404.html || heartbeat
```

这个时候会发现网站上已经能看到导航了，但点击却提示找不到页面。

这时需要去新建对应的页面，比如新建 `标签` 页：

```bash
$ hexo new page "tags"
```

新建好后会在 `source/tags/index.md` 看到刚新建的文件，在 `Front-matter` 区域增加一句：

```markdown
type: "tags"
```

其它导航页面操作一样。

#### 增加评论模块

在 NexT 官网上可以查到第三方评论系统的配置方法，我最终选择的是 [Gitment](https://github.com/imsun/gitment) 做为博客的评论系统，因为 Gitment 是一款基于 GitHub Issues 的评论系统，这样对于评论的管理完全可以在一个仓库里实现。

步骤如下：

首先点击 [这里](https://github.com/settings/applications/new) 注册一个新的 OAuth Application 。`Homepage URL` 和 `Authorization callback URL` 均填写博客地址就行

![](https://b3logfile.com/file/2021/06/solo-fetchupload-7276422193673369679-8ba70555.png)

接着注册好后会得到 `Client ID` 和 `Client Secret` ，打开 `主题配置` 找到 `gitment` 对照着修改

```yaml
gitment:
  enable: true
  mint: true
  count: true
  lazy: false
  cleanly: true
  language:
  github_user: hooray
  github_repo: hooray.github.io
  client_id: xxxxxxxxxx
  client_secret: xxxxxxxxxx
  redirect_protocol:
```

最后就是发布页面，发布好后，访问页面并使用你的 GitHub 账号登录（请确保账号是上面 repo 的拥有者），点击初始化按钮。之后其他用户就可以在该页面发表评论了。

需要注意的是，Gitment 不支持链接里面有中文，不然初始化评论的时候会提示 `Error: Validation Failed` 。解决办法就是创建文章的时候，使用纯英文当文件名。

#### 文章链接唯一化

有时候可能需要修改一篇已经发布的文章的标题，或者是修改它的发布时间，这样就导致文章链接地址会变化，也就间接导致 Gitment 评论丢失。

解决这个问题需要安装 `hexo-abbrlink` 工具

```bash
$ yarn add hexo-abbrlink
```安装好后打开 `站点配置` 找到 `permalink` 修改

```yaml
permalink: posts/:abbrlink/
```并在增加如下代码

```yaml
# abbrlink config
abbrlink:
  alg: crc32  # 算法：crc16(default) and crc32
  rep: hex    # 进制：dec(default) and hex
```

配置好后重新发布就能看到效果。

个人强烈推荐安装这个工具，这样创建文章的时候可以继续使用中文名，如果博客里文章多了，还是中文一目了然，也方便管理，同时也解决了上面的问题，链接永久唯一化后对 SEO 会更友好。

#### 支持 emoji 表情

> 2019 年 2 月 3 日更新

因为 Hexo 默认的 markdown 渲染引擎不支持 Github 的 emoji 表情，所以要支持必须换一个引擎，然后再增加一个 emoji 插件，依次执行以下命令即可：

```bash
npm un hexo-renderer-marked --save
npm i hexo-renderer-markdown-it --save
npm install markdown-it-emoji --save
```完成插件安装后还需要修改 Hexo 站点配置文件 `_config.yml`

```yaml
## markdown 渲染引擎配置，默认是hexo-renderer-marked，这个插件渲染速度更快，且有新特性
markdown:
  render:
    html: true
    xhtmlOut: false
    breaks: true
    linkify: true
    typographer: true
    quotes: '“”‘’'
  plugins:
    - markdown-it-footnote
    - markdown-it-sup
    - markdown-it-sub
    - markdown-it-abbr
    - markdown-it-emoji
  anchors:
    level: 2
    collisionSuffix: 'v'
    permalink: true
    permalinkClass: header-anchor
    permalinkSymbol: ''
```

最后我们在 [网站1](https://www.emojicopy.com/) 或者 [网站2](https://github.com/caiyongji/emoji-list/blob/master/README-CN.md) 复制想要的 emoji 表情就可以了。😄

## 参考

* [Hexo 文档](https://hexo.io/zh-cn/docs/index.html)
* [NexT 使用文档](http://theme-next.iissnan.com/)
* [搭建个人博客-hexo+github详细完整步骤](https://www.jianshu.com/p/189fd945f38f)
* [Gitment：使用 GitHub Issues 搭建评论系统](https://imsun.net/posts/gitment-introduction/)
* [hexo的next主题个性化配置教程](https://segmentfault.com/a/1190000009544924)
* [为NexT主题添加文章阅读量统计功能](https://notes.wanghao.work/2015-10-21-%E4%B8%BANexT%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E9%98%85%E8%AF%BB%E9%87%8F%E7%BB%9F%E8%AE%A1%E5%8A%9F%E8%83%BD.html)
* [hexo 摸爬滚打之进阶教程](http://muyunyun.cn/posts/f55182c5)
* [打造个性超赞博客 Hexo + NexT + GitHub Pages 的超深度优化](https://io-oi.me/tech/hexo-next-optimization)
