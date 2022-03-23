title: Go开发环境搭建
date: 2021-08-27 11:33:00
toc: true
category:
 - Golang
tags:
 - Go
 - Golang
 - 开发
 - 环境
 - 搭建
---

本文基于 `Centos7.8` 介绍了常见的开发环境工具的搭建流程及命令。

## git安装

1. 安装: `yum install -y git`
    
    配置:
    
    ```shell
    git config --global user.name "biuaxia"
    git config --global user.email "biz@biuaxia.cn"
    git config --global --list
    ```
    
2. 查看有无sshKey: `ls -al ~/.ssh`
3. 生成sshKey: `ssh-keygen -t ed25519 -C "biz@biuaxia.cn"`
4. 拷贝sshKey到剪贴板(粘贴到Github账户配置): `cat ~/.ssh/id_ed25519.pub`
5. 验证git配置: `git clone git@git.zhlh6.cn:biuaxia/Sardine.git test/1`
6. 删除git clone的文件夹: `rm -rf ~/test/`

---

## dockers 和 docker-compose 安装

### docker 安装

1. 安装: `curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun`
2. 开启 `docker: systemctl start docker`
3. 设置开机启动: `docker: systemctl enable docker`
4. 检查是否启动: `ps -ef|grep docker`
5. 检查能否执行: `docker ps -a`
6. 配置容器镜像加速(访问 <https://cr.console.aliyun.com/cn-hangzhou/instances>, 左侧 `镜像工具`-`镜像加速器`)
7. 查看能否加速: `sudo docker run --name he hello-world`
8. 删除容器: `docker rm he`
9. 删除镜像: `docker rmi hello-world`

### docker-compose 安装

1. 安装: `curl -L https://get.daocloud.io/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose`
2. 授权: `chmod +x /usr/local/bin/docker-compose`

## MySQL安装

1. 安装: `docker pull mysql:5.7`
2. 检查: `docker images`
3. 运行: `docker run -p 3306:3306 --name mysql57 -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7`
    
    > 注意: 所有可选参数前者是指容器，后者是指宿主机。例如: `-v $PWD/logs:/logs` 是指将容器当前目录下的logs目录挂载到宿主机的/logs目录。
    
4. 检查: `docker ps -a`
5. 日志: `docker logs mysql57`
6. 进入: `docker exec -it mysql57 /bin/bash`
7. 登录: `mysql -uroot -p123456`
8. 创建新用户及远程登录授权
    
    ```sql
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'127.0.0.1' IDENTIFIED BY 'root' WITH GRANT OPTION;
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY 'root' WITH GRANT OPTION;
    FLUSH PRIVILEGES;
    ```
    
9. 查询: `select user,host from user;`
10. 退出MySQL: `exit`
11. 退出容器: `exit`
12. 放行防火墙或安全组规则
    
    | 来源 | 协议端口 |
    | ---- | -------- |
    | 0.0.0.0/0 | TCP:3306 |
    | ::/0 | TCP:3306 |
    
13. 通过数据库连接工具链接，如 Navicat for MySQL。
