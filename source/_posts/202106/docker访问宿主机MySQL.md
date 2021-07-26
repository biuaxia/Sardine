title: docker访问宿主机MySQL
date: '2021-06-18 22:34:04'
updated: '2021-06-18 22:34:04'
category: 
 - Docker
tags: [docker]
permalink: /articles/2021/06/18/1624026844425.html
---
有时MySQL安装在宿主机中，而服务在docker容器中，此时可以通过通过以下两张方式访问

## 方法一

启动docker时使用`–net=host`

`--net host`这个参数，让容器运行在宿主机相同的网络

此时可以通过`127.0.0.1`直接连接MySQL

## 方法二

使用docker内网IP连接

宿主机输入`ifconfig`

```bash
ifconfig
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:aaff:fef6:afbf  prefixlen 64  scopeid 0x20<link>
        ether 02:42:aa:f6:af:bf  txqueuelen 0  (Ethernet)
        RX packets 14003206  bytes 1806192729 (1.6 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 16676825  bytes 96141522987 (89.5 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

`docker0` 网卡下的ip地址`172.17.0.1`就是宿主机在`docker内网的IP`，此时可以通过`172.17.0.1`连接


