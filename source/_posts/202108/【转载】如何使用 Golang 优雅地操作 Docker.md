title: '【转载】如何使用 Golang 优雅地操作 Docker'
date: 2021-08-09 10:33:00
toc: true
category:
 - 转载
 - Go
tags: 
 - 转载
 - Go
 - golang
 - Docker
---

本文转载自：[如何使用 Golang 优雅地操作 Docker - 知乎](https://zhuanlan.zhihu.com/p/360242317)

---

Docker 是目前最流行的 Linux 容器解决方案，它解决了软件开发中环境配置的问题，因此在日常开发中得到了广泛的应用。 通常情况下，我们都会按照官网上的步骤下载 Docker 并安装，然后在终端执行 `docker run hello-world` 得到下面的输出就代表安装好了


<!-- more -->


```bash
klew@primary:~$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

## 执行 docker run 时发生了什么

Docker 是典型的 C-S 架构，执行 `docker run` 的终端相当于是客户端，而 Docker 的容器实际由 Docker Daemon 控制。这里的 Docker Daemon 运行在在 `DOCKER_HOST` 这个环境变量指定的机器上。 当我们执行 `docker run hello-world` 时， `DOCKER_HOST` 的默认值是 `unix:///var/run/docker.sock` ，此时客户端就会通过这个 socket 找到本地的 Docker Daemon，并向他发送 `run hello-world` 这个指令，然后由 Docker Daemon 去拉取镜像和启动容器。参考下图：

![](https://b3logfile.com/file/2021/08/solo-fetchupload-3135900552007930354-bade85f6.jpeg)

从图中可知，Docker 客户端和 Docker Daemon 并不一定得在同一台机器上，通过在运行时指定 `DOCKER_HOST` 环境变量，我们还能将指令发送给其他机器上的 Docker Daemon，例如要让 192.168.64.1 上的 Docker Daemon 运行 `run hello-world` ，我们只需要执行：

```bash
$ DOCKER_HOST=tcp://192.168.64.1:2375 docker run hello-world
```

需要注意的是，这条命令成功执行的条件是 192.168.64.1 开放了 2375 这个端口。 在终端执行 Docker 命令基本已经够用了，但想要集成到应用程序中还是很不方便，例如我想要用 Golang 做一个 Docker 容器监控的项目，那么如何在 Go 程序中获取容器信息呢？ Docker SDK 恰好提供了这样的功能

## Docker SDK

Docker SDK 提供了与 Docker daemon 交互的接口，官方支持 Python 和 Go，其他热门语言也有对应的第三方库

### 环境准备

1. 配置 Docker 环境

使用 Docker Daemon 要求版本在 18.09 以上，本地的 Docker 客户端也要求在 19.03 以上，不然之后尝试连接时会报错：

```bash
[] error during connect: Get "http://docker/v1.24/images/json": command [] has exited with exit status 255, please make sure the URL is valid, and Docker 18.09 or
 later is installed on the remote host: stderr=ssh: connect to host: Connection refused
```

1. 安装 Go SDK

按照 [官网-Install the SDKs](https://link.zhihu.com/?target=https%3A//docs.docker.com/engine/api/sdk/%23install-the-sdks) 的步骤，下载依赖即可：

```bash
$ go get github.com/docker/docker/client
```

### 连接本地 Docker Daemon

### 初始化客户端对象

这里我们直接连接本地的 Docker Daemon，不需要过多配置，直接用环境变量的参数初始化客户端即可。

```go
// NewEnvClient 直接使用环境变量中的 DOCKER_HOST, DOCKER_TLS_VERIFY, DOCKER_CERT_PATH, DOCKER_API_VERSION 配置
cl, err := client.NewEnvClient()
```

### 执行命令

Docker SDK 对拉取镜像、运行容器、查看状态等命令都进行了封装，具体可以参考[文档](https://link.zhihu.com/?target=https%3A//pkg.go.dev/github.com/docker/docker/client%3Futm_source%3Dgodoc)，例如想要查看镜像列表则只需要执行如下命令：

```go
cl.ImageList(context.Background(), types.ImageListOptions{})
```

### 完整代码

```go
// main.go
package main

import (
    "context"
    "fmt"

    "github.com/docker/docker/api/types"
    "github.com/docker/docker/client"
)

func main() {
    cl, err := client.NewEnvClient()
    if err != nil {
        fmt.Println("Unable to create docker client")
        panic(err)
    }

    fmt.Println(cl.ImageList(context.Background(), types.ImageListOptions{}))

}
```

### 运行结果

```bash
$ go run main.go
[{-1 1614986725 sha256:d1165f2212346b2bab48cb01c1e39ee8ad1be46b87873d9ca7a4e434980a7726 map[]  [hello-world@sha256:308866a43596e83578c7dfa15e27a73011bdd402185a84c5cd7f32a88b501a24] [hello-world:latest] -1 13336 13336}] <ni
l>
```

可以看到这里显示了刚刚的 hello-world 镜像

### 连接远程 Docker Daemon

连接远程的 Docker Daemon 和本地的类似，只不过需要在初始化客户端对象时指定连接远程的方式。 例如如果要使用 TCP 连接 192.168.64.1:2375 的 Docker Daemon，只需要将初始化客户端对象换成如下语句即可：

```go
cl, err := client.NewClient("tcp://192.168.64.1:2375", "", nil, nil)
```

### 连接需要身份验证的服务器

上面连接远程 Docker Daemon 的方法的前提条件是目标机器开放了 2375 端口。然而在大多数情况下，出于安全考虑，服务器对端口开放有严格的限制，开发者通常需要使用 ssh 登录到服务器后才能操作服务器上的 Docker。对于这种情况，我们可以利用 ```github.com/docker/cli/cli/connhelper` 这个库帮我们完成 ssh 登录验证的工作。

### 创建连接客户端

这一步与使用 Go 发送 HTTP 请求时需要创建的客户端很像，但是需要把 Transport 的 DialContext 换成 connhelper 提供的 Dialer

```go
helper, _ := connhelper.GetConnectionHelper("ssh://klew@192.168.64.2:22")
httpClient := &http.Client{
    Transport: &http.Transport{
        DialContext: helper.Dialer,
    },
}
```

### 初始化 Docker 客户端

这里我们不再使用环境变量默认的配置，而是通过配置参数的方式初始化 Docker 客户端，同时指定连接的 HTTP 客户端和上下文等信息。这样当我们尝试连接远程服务器的 Docker Daemon 时，connhelper 就会自动帮我们完成 ssh key 的验证操作

```go
cl, err := client.NewClientWithOpts(
    client.WithHTTPClient(httpClient),
    client.WithHost(helper.Host),
    client.WithDialContext(helper.Dialer),
)
```

### 执行命令

依然使用查看镜像列表的方式来验证代码

```go
fmt.Println(cl.ImageList(context.Background(), types.ImageListOptions{}))
```

### 完整代码

```go
package main

import (
    "context"
    "fmt"
    "net/http"

    "github.com/docker/cli/cli/connhelper"
    "github.com/docker/docker/api/types"
    "github.com/docker/docker/client"
)

func main() {
    helper, err := connhelper.GetConnectionHelper("ssh://klew@192.168.64.2:22")
    if err != nil {
        return
    }
    httpClient := &http.Client{
        Transport: &http.Transport{
            DialContext: helper.Dialer,
        },
    }
    cl, err := client.NewClientWithOpts(
        client.WithHTTPClient(httpClient),
        client.WithHost(helper.Host),
        client.WithDialContext(helper.Dialer),
    )
    if err != nil {
        fmt.Println("Unable to create docker client")
        panic(err)
    }

    fmt.Println(cl.ImageList(context.Background(), types.ImageListOptions{}))
}
```

### 运行结果

```bash
$ go run main.go
[{-1 1614986725 sha256:d1165f2212346b2bab48cb01c1e39ee8ad1be46b87873d9ca7a4e434980a7726 map[]  [hello-world@sha256:308866a43596e83578c7dfa15e27a73011bdd402185a84c5cd7f32a88b501a24] [hello-world:latest] -1 13336 13336}] <ni
l>
```

## 总结

Docker SDK 封装了 Docker 客户端会用到的指令，[Go client for the Docker Engine API](https://link.zhihu.com/?target=https%3A//pkg.go.dev/github.com/docker/docker/client%3Futm_source%3Dgodoc) 详细介绍了各个指令的使用方法，充分满足我们使用程序与 Docker Daemon 的需求

## 参考

* [Develop with Docker Engine SDKs](https://link.zhihu.com/?target=https%3A//docs.docker.com/engine/api/sdk/)
* [Docker overview](https://link.zhihu.com/?target=https%3A//docs.docker.com/get-started/overview/)
