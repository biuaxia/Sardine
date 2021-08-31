title: 【转载】Docker镜像原理
date: 2021-08-16 09:24:01
toc: true
category:
 - 转载
 - Docker
tags:
 - 转载
 - Docker
 - 镜像
 - 原理
---

本文转载自：[Docker镜像原理](https://juejin.cn/post/6996310038419603463?utm_source=gold_browser_extension)

---

# Docker镜像原理

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发额软件，它包含某个软件所需要的所有内容，包括代码、运行时库、环境变量和配置文件。


<!-- more -->


## UnionFs（联合文件系统）

UnionFS（联合文件系统）：Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。 **特性** ：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

下载docker镜像时一层一层的其实就是联合文件系统的体现

## Docker镜像加载原理

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。 bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

rootfs (root file system) ，在bootfs之上。包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。 

![](https://b3logfile.com/file/2021/08/solo-fetchupload-8347610906337307143-06c887bf.webp)

平时我们安装虚拟机的CentOs都是好几个G，为什么docker才几百M

对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供 rootfs 就行了。由此可见对于不同的linux发行版, bootfs基本是一致的, rootfs会有差别, 因此不同的发行版可以公用bootfs。

## 分层的理解

下载docker镜像时一层一层的其实就是分层最直观的体现（已经下载过得不会重复下载）

```shell
$ docker pull redis
Using default tag: latest
latest: Pulling from library/redis
33847f680f63: Already exists 
26a746039521: Pull complete
18d87da94363: Pull complete
5e118a708802: Pull complete
ecf0dbe7c357: Pull complete
46f280ba52da: Pull complete
Digest: sha256:cd0c68c5479f2db4b9e2c5fbfdb7a8acb77625322dd5b474578515422d3ddb59
Status: Downloaded newer image for redis:latest
docker.io/library/redis:latest
```

我们还可以通过之前文章中提到的inspect命令

```shell
docker image inspect redis:latest
```

可以看到镜像的具体分层信息，有6层对应上面镜像下载时候的6层

```json
"RootFS": {
    "Type": "layers",
    "Layers": [
        "sha256:814bff7343242acfd20a2c841e041dd57c50f0cf844d4abd2329f78b992197f4",
        "sha256:dd1ebb1f5319785e34838c7332a71e5255bda9ccf61d2a0bf3bff3d2c3f4cdb4",
        "sha256:11f99184504048b93dc2bdabf1999d6bc7d9d9ded54d15a5f09e36d8c571c32d",
        "sha256:e461360755916af80821289b1cbc503692cf63e4e93f09b35784d9f7a819f7f2",
        "sha256:45f6df6342536d948b07e9df6ad231bf17a73e5861a84fc3c9ee8a59f73d0f9f",
        "sha256:262de04acb7e0165281132c876c0636c358963aa3e0b99e7fbeb8aba08c06935"
    ]
},
```

**分层的好处** ：

最大的一个好处就是 - 共享资源

比如：有多个镜像都从相同的 base 镜像构建而来，那么 Docker Host 只需在磁盘上保存一份 base 镜像；同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。

这时可能就有人会问了：如果多个容器共享一份基础镜像，当某个容器修改了基础镜像的内容，比如 /etc 下的文件，这时其他容器的 /etc 是否也会被修改？ 答案：不会！因为修改会被限制在单个容器内。

这就是我们接下来要学习的容器 Copy-on-Write 特性

## 容器的可写层

当容器启动时，一个新的可写层被加载到镜像的顶部。 这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。所有操作都是针对容器层的，只有容器层是可写的，容器层下面的所有镜像层都是只读的。

![](https://b3logfile.com/file/2021/08/solo-fetchupload-4718177164731103431-37a8e103.webp)

## commit镜像

```shell
docker commit  提交容器成为一个新的副本
```

我们下面通过一个例子说明下

我这边之前下载过tomcat的镜像

```shell
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
tomcat       latest    710ec5c56683   7 days ago     668MB
redis        latest    aa4d65e670d6   3 weeks ago    105MB
mysql        latest    c60d96bd2b77   3 weeks ago    514MB
centos       centos7   8652b9f0cb4c   9 months ago   204MB
```

将tomcat启动起来

```shell
-- 镜像运行起来，并且将端口进行映射到宿主机端口
docker run  -it -p 8080:8080 tomcat
```

在另外一个创建通过命令查看下当前运行的容器,tomcat正在运行

```
$ docker ps
CONTAINER ID   IMAGE            COMMAND             CREATED          STATUS          PORTS                                       NAMES
8b294cd7074e   tomcat           "catalina.sh run"   29 seconds ago   Up 28 seconds   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   zealous_cohen
f42ae22e4b72   centos:centos7   "/bin/bash"         3 weeks ago      Up 46 hours                                                 centos-test
```

然后我们进入tomcat这个容器看下哈

```shell
docker exec -it 8b294cd7074e /bin/bash
```

```shell
$ docker exec -it 8b294cd7074e /bin/bash
root@8b294cd7074e:/usr/local/tomcat# ls
BUILDING.txt  CONTRIBUTING.md  LICENSE	NOTICE	README.md  RELEASE-NOTES  RUNNING.txt  bin  conf  lib  logs  native-jni-lib  temp  webapps  webapps.dist  work
```

然后再进入webapp（项目文件位置）中看下,如下可以看到webapp中是空的

```shell
root@8b294cd7074e:/usr/local/tomcat# cd webapps
root@8b294cd7074e:/usr/local/tomcat/webapps# ls
root@8b294cd7074e:/usr/local/tomcat/webapps#
```

然后我们访问下tomcat，404说明tomcat是正常启动了，只是没有找到对应的页面，这也容易理解应为wabapp中是空的

![](https://b3logfile.com/file/2021/08/solo-fetchupload-210473264531739456-cc22d33a.webp)

那么我们尝试自己做一个webapp中有内容的镜像

我们将webapps.dist中的文件copy到wabapp中

```shell
root@8b294cd7074e:/usr/local/tomcat# cp -r webapps.dist/* webapps
root@8b294cd7074e:/usr/local/tomcat# cd webapps
root@8b294cd7074e:/usr/local/tomcat/webapps# ls
ROOT  docs  examples  host-manager  manager
```

修改之后发现tomcat可以正常访问了

![](https://b3logfile.com/file/2021/08/solo-fetchupload-6500030143985229502-424205b7.webp)

```shell
$ docker ps
CONTAINER ID   IMAGE            COMMAND             CREATED          STATUS          PORTS                                       NAMES
8b294cd7074e   tomcat           "catalina.sh run"   27 minutes ago   Up 27 minutes   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   zealous_cohen
```

现在这个tomcat我们做了简单的修改，我觉得我的这个tomcat比官方的好一点点，我把我这个镜像提交下

```shell
$ docker commit -a="cb" -m "add init file" 8b294cd7074e newtomcat:1.0
sha256:44cf4d44be664d9704a3fc38ddef1f03fa7f113ad83f4049cced322a14dc216b
```

通过docker images可以看到有新镜像就创建好了，仔细观察可以发现我们提交的额newtomcat比官方的tomcat要大一点

```shell
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
newtomcat    1.0       44cf4d44be66   46 seconds ago   673MB
tomcat       latest    710ec5c56683   7 days ago       668MB
redis        latest    aa4d65e670d6   3 weeks ago      105MB
mysql        latest    c60d96bd2b77   3 weeks ago      514MB
centos       centos7   8652b9f0cb4c   9 months ago     204MB
```

后面就可以直接使用自己的镜像了，也可以发布出去给别人使用，这个后面在说

通过这个例子可以回顾下上面的分层思想，newtomcat是我们在之前容器层做了修改后生成的镜像，这个镜像和虚拟机的镜像快照差不多

关于docker的镜像就说到这里，到现在docker算是简单的入了个门，后面会继续学习docker相关的内容，大家一起加油！

> 日拱一卒无有尽，功不唐捐终入海
