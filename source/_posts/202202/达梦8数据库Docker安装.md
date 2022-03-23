title: 达梦8数据库Docker安装
date: 2022-02-21 12:23:13
toc: true
index_img: https://b3logfile.com/file/2022/02/97cb01fffd7c44d39e951ba987618bae.jpeg
category:
- 达梦
tags:
- 达梦
- Docker
- 数据库
- 安装
---

## 安装前准备

| 软硬件 | 版本            |
| -------- | ----------------- |
| 终端   | X86-64 架构     |
| Docker | 19.0 及以上版本 |

## 下载 Docker 安装包

在根目录下创建 /dm8 文件夹，用来放置下载的 Docker 安装包。命令如下：

```
mkdir /dm8
```

切换到 /dm8 目录，下载 DM Docker 安装包。命令如下：

```
wget -O dm8_docker.tar -c https://download.dameng.com/eco/dm8/dm8_docker.tar
```

> **注意**容器提供的实例默认为大小写不敏感，如果需要修改容器中的大小写敏感参数，可以删除当前实例后重新初始化，初始化过程中设置大小写敏感。
> 或者直接下载大小写敏感版本：[https://download.dameng.com/eco/dm8/dm8_docker_case.tar](https://download.dameng.com/eco/dm8/dm8_docker_case.tar)

## 导入镜像

下载完成后，导入安装包，使用如下命令：

```
docker import dm8_docker.tar dm8:v01
```

导入完成后，可以使用 `docker images` 来查看导入的镜像，命令如下：

```
docker images
```

查看结果如下：

![docker images](https://b3logfile.com/file/2022/02/97cb01fffd7c44d39e951ba987618bae.jpeg)

## 启动容器

镜像导入后，使用 `docker run` 来启动容器，默认的端口 5236 默认的账号密码 ，启动命令如下：

```
docker run -itd -p 5236:5236 --name dm8_01 dm8:v01 /bin/bash /startDm.sh
```

容器启动完成后，使用 `docker ps` 来查看镜像的启动情况，命令如下：

```
docker ps
```

结果显示如下：

![docker ps](https://b3logfile.com/file/2022/02/eed9fc0e87bb40a48706c50b2de917db.jpeg)

启动完成后，可以查看日志来查看启动情况，命令如下：

```
docker logs -f  dm8_01
```

显示内容如下，则表示启动成功。

![docker logs -f ](https://b3logfile.com/file/2022/02/8038f56d88c245c08a84b3af9c2cd466.jpeg)

## 启动停止数据库

停止命令如下：

```
docker stop  dm8_01
```

启动命令如下：

```
docker start  dm8_01
```

重启命令如下：

```
docker restart  dm8_01
```

> **注意**如果使用docker容器里面的 disql ,进入容器后，先执行 source /etc/profile 防止中文乱码。
