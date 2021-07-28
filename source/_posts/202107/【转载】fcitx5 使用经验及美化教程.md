title: '【转载】fcitx5 使用经验及美化教程（自己制作皮肤）'
date: '2021-07-28 09:51:49'
updated: '2021-07-28 21:14:49'
category:
 - 转载
 - Linux
tags: [转载, Linux, fcitx5, 皮肤, 经验, 美化, 教程]
permalink: /articles/2021/07/28/1627478175440.html
---

本文转载自：[fcitx5 使用经验及美化教程（自己制作皮肤） - Linux/Unix - 论坛](https://hu60.cn/q.php/bbs.topic.100994.html)

---

## fcitx5介绍及经验分享

1. 在7月23日内测发布后，fcitx5已经可以正常在deepin下使用了，但是大家应该发现一个问题，fcitx5并不能在deepin下开机启动，即使在图标右键勾选开机自启动也无法正常启动，根据我多年混迹Arch Linux的经验我们需要在终端执行这样一条命令：

```
cp /usr/share/applications/org.fcitx.Fcitx5.desktop ~/.config/autostart/
```

重启电脑后就会正常启动。

fcitx5皮肤制作（无法实现动态基于搜狗皮肤）
对，你没看错，这次是 fcitx5 是用搜狗皮肤！

废话不多说下面开始教程！

### 下载此仓库

```
git clone https://github.com/fkxxyz/ssfconv.git cd ssfconv
```

### 下载皮肤

先从搜狗输入法的皮肤官网下载自己喜欢的皮肤，得到ssf格式的文件，例如 【雨欣】蒲公英的思念.ssf

### 转换皮肤

```
./ssfconv -t fcitx5 【雨欣】蒲公英的思念.ssf 【雨欣】蒲公英的思念
```

### 复制到用户皮肤目录

```
mkdir -p ~/.local/share/fcitx5/themes/ cp -r 【雨欣】蒲公英的思念 ~/.local/share/fcitx5/themes/
```

### 使用该皮肤

打开 fcitx5 的配置，附加组件标签，经典用户界面，点配置，在主题的下拉列表里，选择这款皮肤。

或者你也可以直接修改配置文件 `~/.config/fcitx5/conf/classicui.conf`，将 Theme 的值改成这个皮肤的名称即可。

用下面这条命令可以看到该皮肤的名称：

```
grep Name ~/.local/share/fcitx5/themes/【雨欣】蒲公英的思念/theme.conf
```

### 详细介绍

使用方法被封装得非常简单，像个转换器，可以在下面四种格式之间任意转换：

ssf格式（加密）
ssf格式（未加密，本质是zip）
文件夹（解密或解压ssf格式得到）
fcitx格式（在文件夹的基础上多了fcitx_skin.conf，可用于fcitx）
fcitx5格式（在文件夹的基础上多了theme.conf，可用于fcitx5）
命令行参数

```
ssfconv [-t type]
```

源文件和目标文件是必选参数，转换的目标类型 -t 是可选参数，type值是下面四个值之一：

* `fcitx` 可直接用于fcitx的文件夹
* `fcitx5` 可直接用于fcitx5的文件夹
* `dir` 解包后的文件夹
* `encrypted` 加密的ssf皮肤
* `zip` 未加密的ssf皮肤（zip）

默认是转换为 fcitx 格式。

注意，源文件的格式可以是以上任意五个格式之一，不需要指定，程序已经可以智能识别格式。

## 已知缺陷

fcitx5 能够完美地像搜狗输入法一样调整，但是主题中所设置的字体是无效的，需要手动设置字体，经过我反复的实验，将字体设置为 “Sans 10” 似乎是大多数皮肤的最佳体验。

## 此文章参考文章：

fcitx5 使用搜狗皮肤 - 四叶草：

```
https://www.fkxxyz.com/d/ssfconv2/
```

Fcitx5 (简体中文)：

```
https://wiki.archlinux.org/title/Fcitx5_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%BC%80%E6%9C%BA%E5%90%AF%E5%8A%A8
```
