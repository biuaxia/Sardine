title: 自动续期蓝凌License
date: 2022-03-23 15:34:00
toc: true
category: 
- Landray
tags:
- 自动
- 续期
- 蓝凌
- License
- Landray
---

## 文件下载

见：[二开成果详情页](http://mall.landray.com.cn/core01/km/reuse/km_reuse_redevelop/kmReuseRedevelop.do?method=view&fdId=17fb0b664519c279b7c58d24d69a5c14)

![image.png](https://b3logfile.com/file/2022/03/image-6d6f53ca.png)


<!-- more -->


## 定时任务

### 替换蓝凌License

- 执行周期：每天9时2分
- 脚本内容：

  ```shell
  /www/apps/ekp/landray-tools
  ```

### 停止蓝凌

- 执行周期：每天9时1分
- 脚本内容：

  ```shell
  source ~/.bashrc
  cd /www/apps/ekp/linux64
  ./stop-tomcat.sh
  ```

### 启动蓝凌

- 执行周期：每天9时2分
- 脚本内容：

  ```shell
  source ~/.bashrc
  cd /www/apps/ekp/linux64
  ./start-normal64.sh
  ```

## 效果展示

![Snipaste20220323154258.png](https://b3logfile.com/file/2022/03/Snipaste_2022-03-23_15-42-58-2f1d7039.png)

![Snipaste20220323154314.png](https://b3logfile.com/file/2022/03/Snipaste_2022-03-23_15-43-14-815a24ae.png)

![Snipaste20220323154346.png](https://b3logfile.com/file/2022/03/Snipaste_2022-03-23_15-43-46-4d36dcf1.png)
