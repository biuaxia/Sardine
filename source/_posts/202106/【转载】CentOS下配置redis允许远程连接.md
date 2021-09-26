title: 【转载】CentOS下配置redis允许远程连接
date: 2021-06-18 23:04:22
comment: false
toc: true
category:
 - Linux
 - Centos
tags: 
 - 转载
 - Centos
 - Redis
 - 远程
 - 链接
 - 允许
 - 配置
---

本文转载自：[CentOS下配置redis允许远程连接 - 禁忌夜色153 - 博客园](https://www.cnblogs.com/jinjiyese153/p/8600703.html)

---

## 目的

因为想要学习redis，因此在虚拟机中安装了redis，为了实现在物理机可以访问redis，弄了好久；因此记录下来，以免忘记。


<!-- more -->


## 环境

- 虚拟机：CentOS Linux release 7.4.1708 (Core)
- redis：4.0.8
- 防火墙：iptables

## 配置

### 配置redis.conf

1. 将 *bind 127.0.0.1* 使用#注释掉，改为# bind 127.0.0.1（bind配置的是允许连接的ip，默认只允许本机连接；若远程连接需注释掉，或改为0.0.0.0
2. 将 *protected-mode yes* 改为 protected-mode no（3.2之后加入的新特性，目的是禁止公网访问redis cache，增强redis的安全性）
3. 将 *requirepass foobared* 注释去掉，foobared为密码，也可修改为别的值（可选，建议设置）

### 设置iptables规则，允许外部访问6379端口

```bash
iptables -I INPUT 1 -p tcp -m state --state NEW -m tcp --dport 6379 -j ACCEPT
```
临时生效，重启后失效。若想永久生效，请参考另一篇文章：[http://www.cnblogs.com/jinjiyese153/p/8600855.html](http://www.cnblogs.com/jinjiyese153/p/8600855.html)

### 启动redis，并指定配置文件

```bash
./redis-server ../redis.conf
```

## 检查

本机安装RedisDesktopManager进行redis远程连接。

![](https://b3logfile.com/file/2021/06/solo-fetchupload-4209952803740035649-53bc2bab.png)

连接成功。

