title: 【转载】为brew、git、pip设置代理&为brew正确换源终极版
date: '2021-06-18 21:57:00'
toc: true
permalink: /articles/2021/06/18/1624024619981.html
category: 
 - 转载
tags: 
 - 转载
 - brew
 - http
 - https
 - git
 - pip
 - 代理
 - 设置
 - 换源
 - 终极版
---

本文转载自：[为brew/git/pip设置代理&为brew正确换源终极版 - SegmentFault 思否](https://segmentfault.com/a/1190000019758638)

---

# 如何正确设置HTTP/HTTPS代理

设置了brew通过socks5的代理后，会发现pip其实是不支持socks5的，只能通过http/https。


<!-- more -->


## 首先获取端口

* 点击状态栏的小火箭
* HTTP Proxy Preference
* 获取HTTP的端口号（我是1087）

## 将以下内容添加进.bash_profile（bash用户）/.zshrc（zsh用户）并保存

```
#设置HTTP/HTTPS Proxy
export http_proxy="http://127.0.0.1:1087"; 
export https_proxy="http://127.0.0.1:1087";
```

保存，进入shell，以bash为例

```
#更新配置
source .bash_profile

#验证
brew update
pip install --upgrade pip
```

# 如何正确为brew换源

设置全局代理或者换源都有各自的优缺点，如果你想换源，那么尝试下面的步骤，要注意的是，**换源和设置代理只能选其一** ，不然结果是一样的。

按照一般方法更换中科大源后，执行brew update还是巨慢，后来发现是cask仍然接在github上，所以要把cask一起换了。要注意的是，**Caskroom 的 Git 地址在 2018年5月25 日从 [https://github.com/caskroom/h...](https://github.com/caskroom/homebrew-cask) 迁移到了[https://github.com/Homebrew/h...](https://github.com/Homebrew/homebrew-cask)。**

## 更换中科大源：

```
# 替换brew.git:
cd "$(brew --repo)"
git remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git

# 替换homebrew-core.git:
cd "$(brew --repo)"/Library/Taps/homebrew/homebrew-core
git remote set-url origin https://mirrors.aliyun.com/homebrew/homebrew-core.git

# 替换homebrew-cask.git:
cd "$(brew --repo)"/Library/Taps/homebrew/homebrew-cask
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git

# 替换homebrew-bottles:
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile

#更新配置
source ~/.bash_profile

#验证
brew update
```

## 重置官方源：

```
#重置brew.git:
cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git

#重置homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core.git

#重置homebrew-cask.git：
cd "$(brew --repo)"/Library/Taps/homebrew/homebrew-cask
git remote set-url origin https://github.com/Homebrew/homebrew-cask
#Caskroom 的 Git 地址在 2018年5月25 日从 https://github.com/caskroom/homebrew-cask 迁移到了 https://github.com/Homebrew/homebrew-cask 

#最后注释掉/.bash_profile里的homebrew-bottles并保存，以bash为例
cd ~
open .bash_profile

#更新.bash_profile
source .bash_profile
```
