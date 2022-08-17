title: 自动续期蓝凌License
date: 2022-03-23 15:34:00
toc: true
index_img: https://b3logfile.com/file/2022/03/Snipaste_2022-03-23_15-43-46-4d36dcf1.png
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

{file href="https://www.biuaxia.cn/usr/uploads/2022/07/1240591275.zip"}自动续期License.zip{/file}

见：[二开成果详情页](http://mall.landray.com.cn/core01/km/reuse/km_reuse_redevelop/kmReuseRedevelop.do?method=view&fdId=17fb0b664519c279b7c58d24d69a5c14)

![image.png](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rGDLR3LmGHAB1BusccJwqQUQHT7oWJIJPA/0)

## 定时任务

### 替换蓝凌License

- 执行周期：每天9时0分
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

![Snipaste20220323154258.png](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rGDLR3LmGHABjibiaYzqaeAmhkaq8bP54OCQ/0)

![Snipaste20220323154314.png](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rODtAhUc2ZL7HAbibmvO5FzUGydMz9kmx5w/0)

![Snipaste20220323154346.png](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rODtAhUc2ZL7FdycMRQIEN1JyTM8q5EByQ/0)

![Snipaste20220323154630.png](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rIySXKianadmmhFjNicCOK4am33egq6h2b3g/0)
