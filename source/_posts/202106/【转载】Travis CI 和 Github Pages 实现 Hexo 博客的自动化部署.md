title: 【转载】Travis CI 和 Github Pages 实现 Hexo 博客的自动化部署
date: 2021-06-18 16:14:54
commens: false
toc: true
category: 
 - Hexo
tags: 
 - 转载
 - hexo
 - Travis-CI
 - Github
 - Pages
 - 自动
 - 自动化
 - 部署
 - 博客
---

本文转载自：[Travis CI 和 Github Pages 实现 Hexo 博客的自动化部署 | Haukeng's Blog](https://blog.ghkk.net/post/automatically-deploy-hexo-with-travis-ci/)

---

> 这篇文章完成于站点刚刚搭建的时候，现在已经不用这个方法来部署了，有机会我会重新写一篇搭建的文章（咕咕咕）

前几日重装系统，便想着使用 Travis CI 来构建部署我的博客，然后把源文件放到 Github repo 中。这样子做实现了用 git 来管理博客的文件，方便进行版本管理，且部署脱离本地环境的限制，如果不需要在本地预览的话，连 hexo 都可以不安装。

需要的工具： `git`, `npm`。


<!-- more -->


## 安装 Hexo

安装 hexo-cli 需要使用到 npm

> 推荐使用 NVM 来安装和管理 nodejs。可以到这里了解 [NVM](https://github.com/nvm-sh/nvm)。

```
npm install -g hexo-cli
```

## 配置 Hexo

### 创建 Hexo 站点

```
hexo init username.github.io
```

> `username` 是你的 Github 用户名，当然也可以把 `username.github.io` 替换成你想要的任何名字

### 做一些必要的修改

* 用编辑器打开站点根目录的 `_config.yml`
* 修改下面这些项目


| 项目       | 解释       | 怎么改                                       |
| ------------ | ------------ | ---------------------------------------------- |
| `title`    | 网站的标题 | 啥都行，例如我的站点标题叫`Haukeng's Blog`   |
| `author`   | 你的名字   | 这个填上你的名字                             |
| `language` | 网站的语言 | 都能看我的博客了，那就改成`zh-CN`            |
| `timezone` | 网站的时区 | 国内的地区都填上`Asia/Shanghai`              |
| `url`      | 网站的链接 | 如果你没有域名的话，就填`username.github.io` |

> 其他的修改可以参考 Hexo 的文档
>
> [Configuration | Hexo Doc](https://hexo.io/docs/configuration)

### 修改站点的主题

* 下载主题

```
cd themes
git clone https://github.com/SukkaW/hexo-theme-suka.git suka
cd suka
npm install --production
cp -i _config.example.yml _config.yml
```

> 本文以 [suka](https://github.com/SukkaW/hexo-theme-suka) 为例

* 启用主题

```
# 回到站点的根目录
cat themes/suka/site_config.yml >> _config.yml
```

打开站点根目录的 `_config.yml`，将 `theme: landscape` 改为 `theme: suka`

改好之后，你站点的 `_config.yml` 大概是这样

```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: suka

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: ''

# Suka-theme
suka_theme:
  search:
    enable: true
    path: search.json
    field: post # Page | Post | All. Default post
  prism:
    enable: false
    line_number: true
    theme: default
```

## 将站点文件夹上传到 Github

### 初始化 git

由于我是在 CI 阶段从网络直接 clone 整个主题，所以我就将 `themes/` 加入了 `.gitignore`，所以这里你还要将主题的配置文件复制到站点根目录内，之后 CI 阶段再将其放回原位置。(如果你用的主题有其他的配置文件时应该一并复制到站点根目录)

复制一份主题的配置文件

```
cp themes/suka/_config.yml  _config.theme.yml
```

初始化 git

```
git init
```

确保 `.gitignore` 内有下面的内容 (如果没有这个文件的话可以自己创建)

```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
themes/
```

### 在 Github 创建 repository

新建一个 repository 命名为 **username.github.io** (`username` 是你的 Github 账户名称)

```
# 添加 Github 仓库到本地
git remote add origin https://github.com/username/username.github.io.git

# 新建一个名为 source 的分支
git checkout -b source

# 将所有文件添加到 git
git add .

# 添加 commit
git commit -m "initial"

# 将本地的文件推送到 Github 上的 source 分支
git push -u origin source
```

如果操作上没有问题你上传之后 repository 里面的文件应该差不多是这样的

```
.
├── _config.theme.yml
├── _config.yml
├── .gitignore
├── package.json
├── package-lock.json
├── scaffolds
└── source
```

## 配置 Travis CI

* 将 [Travis CI](https://github.com/marketplace/travis-ci) 添加到你的 Github 账户
* 前往 Github 的 [Applications settings](https://github.com/settings/installations)， 配置 **Travis CI** 的权限使其能够访问你的 repository
* 前往 Github 的 [Personal Access Tokens](https://github.com/settings/tokens) 为 **Travis CI** 生成一个 Token ( **只需要 `repo` 这个权限(scopes)** )，然后**把 Token 的值记录下来**
* 前往 [Travis CI](https://travis-ci.com/)，在你的 repository 页面下点击 **More Options** 找到 **Settings** ， 然后找到 **Environment Variables** ，建立一个名(NAME)为 **`GH_TOKEN`** , 值( VALUE )为你**上一步记录的 Token** ，最后保存(确保 **DISPLAY VALUE IN BUILD LOG** 保持 **关闭** 避免你的 Token 泄漏
* 在你的 Github 的项目 source 分支内新建一个名为 `.travis.yml` 的文件，参考以下内容进行填入。

```
os: linux
language: node_js
node_js:
  - 10  # 使用 nodejs LTS v10
branches:
  only:
    - source # 只监控 source 的 branch
before_script: ## 根据你所用的主题和自定义的不同，这里会有所不同
  - npm install -g hexo-cli # 在 CI 环境内安装 Hexo
  - mkdir themes # 由于我们没有将 themes/ 上传，所以我们需要新建一个
  - cd themes
  - git clone https://github.com/SukkaW/hexo-theme-suka.git suka #从 Github 上拉取 Suka 主题
  - cd suka
  - npm install --production # 安装 Suka 主题的依赖
  - cd ../.. # 返回站点根目录
  - cp _config.theme.yml themes/suka/_config.yml # 将主题的配置文件放回原处
  - npm install # 在根目录安装站点需要的依赖
script:
  - hexo generate # generate static files
deploy: # 根据个人情况，这里会有所不同
  provider: pages
  skip_cleanup: true # 构建完成后不清除
  token: $GH_TOKEN # 你刚刚设置的 Token
  keep_history: true # 保存历史
  fqdn: blog.ghkk.net # 自定义域名，使用 username.github.io 可删除
  on:
    branch: source # hexo 站点源文件所在的 branch
  local_dir: public
  target_branch: master # 存放生成站点文件的 branch，使用 username.github.io 必须是 master
```

* 当你保存之后， Travis CI 便会开始部署， 它完成之后，你就可以在你的 repo 里 `master` 分支查看到生成的站点文件
* 这时你应该就可以访问 [https://username.github.io](https://username.github.io/) 查看你的站点了
* 之后你写作之后只需要 push 到 Github 就可以了

## 更多操作

如果你想要让搜索引擎收录你的站点，可以看我写的另外一篇文章

[为 Hexo 博客进行 SEO | Haukeng’s Blog](https://blog.ghkk.net/post/seo-for-hexo-site/)

## 参考资料

* [Hexo Doc - Deployment - Github Pages](https://hexo.io/docs/github-pages)
* [Travis CI Documents - GitHub Pages Deployment](https://docs.travis-ci.com/user/deployment/pages/)
