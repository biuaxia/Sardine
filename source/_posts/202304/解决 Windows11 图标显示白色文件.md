title: 解决 Windows11 图标显示白色文件
date: 2023-04-10 09:26:00
toc: false
category: [Go]
tags: [Windows11, 图标]
---

1. Win + E 打开文件资源管理器
2. 地址栏上输入 `%localappdata%`
     ![Snipaste_2023-04-10_09-28-51.png](https://b3logfile.com/file/2023/04/d5f2f50d102c4184969922f846fe8048.png)
3. 找到 `Iconcache.db` 将其删除
4. 在任务管理器中找到 `Windows 资源管理器` 并重启
     ![Snipaste_2023-04-10_09-30-38.png](https://b3logfile.com/file/2023/04/47b5958333684e259ac5fd6a2d66ebfc.png)

修复后的桌面图标 ![Snipaste_2023-04-10_09-32-08.png](https://b3logfile.com/file/2023/04/a446bcbb3ea34d81a3e6da61ce635d28.png)
