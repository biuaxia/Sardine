title: 您没有在符合的IP范围内登录
date: 2021-12-09 07:53:00
toc: true
category:
- Seeyon
tags:
- Seeyon
- 致远
- IP
- 范围
- 登录
---

重新在测试服务器部署了一套致远，尝试用 `system` 账户登录提示 “您没有在符合的IP范围内登录”。

![sp20211209111209005.png](https://b3logfile.com/file/2021/12/sp20211209_111209_005-6fb7655c.png)

首先服务器本地通过浏览器访问，然后直接登录 `system` 账户。

操作 `系统设置 -> 系统参数设置 -> 安全选项 -> 启用IP访问控制`，修改为 “否” 即可。

![Snipaste20211209111004.png](https://b3logfile.com/file/2021/12/Snipaste_2021-12-09_11-10-04-9d2ca01f.png)
