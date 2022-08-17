title: 【转载】寶塔 Linux 面闆 7.7.0 企業版【紀念版】
date: 2022-05-29 18:56:00
toc: false
index_img: http://api.btstu.cn/sjbz/?lx=m_dongman&cid=5
category:
- 技巧
tags:
- 自动
- 配置
- 用户
- 授权
- 服务
- 支持
- 永久
- 环境
---

{tip type="warning"}原文：[宝塔 Linux 面板 7.7.0 企业版【纪念版】 - 开心专区 - 元宇宙 - Powered by Discuz!](https://yu.al/bbs/thread-3-1-1.html)，本站仅作保存记录及调整排版，不做盈利性质的商业行为。{/tip}

{tabs}
{tab name="注意"}
开心版用户点击面板旁边的 “修复” 即可升到最新版！
{/tab}
{tab name="老规矩"}
每次更新 企业版授权 以更新日期为准保持 一年授权 （例如：2021.11.30日更新 授权到期时间  2022.11.30 不要在有人问一年后到期了怎么办 每次更新 会以更新日期为准 保持一年授权）
{/tab}
{tab name="提醒"}
如果你听信他人所说：本脚本有后门、挖矿、面板一直被占用等，可能他使用了其他人的脚本，或者使用过其他人的脚本在使用本脚本，本萌塔不背锅，你非要说是萌塔面板导致，那你大可不必使用，本代码全程公开透明未加密，都是基于官方还原！
{/tab}
{/tabs}

## Q&A

### 问：为什么企业版没永久？

答：因为官方就没有永久呀，只有专业版有！


## 本次主要更新内容

1. 优化面板，修复一直的BUG
2. 优化软件列表，同步官方的列表，同步插件更新
3. 新增系统工具插件：IP配置工具
4. 新增系统工具插件：webshell查杀工具
5. 新增宝塔插件：百度云存储
6. 新增宝塔插件：华为云存储
7. 移除专业版插件：旧版本-宝塔负载均衡
8. 更新专业版插件：Nginx防火墙 8.9.5 - Nginx防火墙 8.9.6
9. 更新专业版插件：网站监控报表 6.0 - 网站监控报表 6.4
10. 新增企业版插件：堡塔限制访问型证书-Linux版
11. 更新企业版插件：安全基线扫描 1.3 - 安全基线扫描 2.0
12. 更新第三方插件：百度云加速1.0 - 百度云加速 1.10
13. 更新第三方插件：我的工具箱1.6.0 - 我的工具箱 1.7.2
14. 更新第三方插件：百度网盘3.6 - 百度网盘 3.8
15. 更新第三方插件：FAST OS DOCKER
16. 更新第三方插件：网速测试
17. 更新第三方插件：sitemap生成器
18. 更新第三方插件：防红短网址生成工具
19. 更新第三方插件：七牛云存储对象多点挂载工具兼容阿里云、京东云
20. 更新第三方插件：域名whois查询
21. 更新第三方插件：小杰工具箱
22. 更新第三方插件：腾讯云对象存储COS自动挂载工具
23. 更新第三方插件：Typecho管理
24. 增加第三方插件：土拨鼠网站日记分析
25. 记性不太好，还有其他的第三方插件都更新了，记不得就不写了！
26. 修复了面板的在线更新，全新安装7.6.0无法面板更新7.7.0的 问题
27. 如果不喜欢软件商店上面的 TG群 / 网页群 / ads 可以点击上面加入群 留言希望剔除的建议！

{tip type="info"}注意：如果你是影视站用户，我不建议你使用本面板，众所周知，大部分的影视程序都是有广告js，后门等，有些用户会认为是本面板导致，甚至有人抹黑本面板，所以建议你使用官方版，或者使用本面板之前先百度/Google下影视站后门问题！！！{/tip}

{tip type="info"}注意：请安装了开心版7.6.0的用户，面板提示更新无法更新的，点下旁边的修复也能升到最新版！{/tip}

{tip type="info"}注意！注意！注意！注意！注意！ 未经授权禁止镜像或者下载插件！{/tip}

新版本 如果你安装的是  专业版 直接执行企业版脚本  还是专业版 （以前可以直接换脚本的 现在不行 我加了缓存）
正确操作：

如果你是专业版 - 先执行bt命令16修复面板 - 登录到软件商店 - 更新软件列表（确认变成免费版了） - 执行企业版脚本 - 登录到软件商店 - 更新软件列表（已经变成企业版了）
企业版变更专业版同理

---

国内节点

Centos安装命令：

```
yum install -y wget && wget -O install.sh http://download.yu.al/ltd/install/install_6.0.sh && sh install.sh
```

试验性Centos/Ubuntu/Debian安装命令 独立运行环境（py3.7） 可能存在少量兼容性问题 不断优化中

```
curl -sSO http://download.yu.al/ltd/install/install_panel.sh && bash install_panel.sh
```

Ubuntu Deepin安装命令：

```
wget -O install.sh http://download.yu.al/ltd/install/install-ubuntu_6.0.sh && sudo bash install.sh
```

Debian安装命令：

```
wget -O install.sh http://download.yu.al/ltd/install/install-ubuntu_6.0.sh && bash install.sh
```

Fedora安装命令:

```
wget -O install.sh http://download.yu.al/ltd/install/install_6.0.sh && bash install.sh
```

Linux面板7.7.0升级专业版命令（7.8.0 可以执行这个降级到 7.7.0 开心版）：
```
curl http://download.yu.al/ltd/install/update6.sh|bash
```

香港节点：

由于 `download.seele.wang` 租用的 32H-64G-20m-200gssd 服务器已经到期，所以取消该线路节点，该域名也被丢弃！

支持主机商赞助香港节点！
