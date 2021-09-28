title: Windows 下 protobuf基础环境搭建
date: 2021-09-28 16:08:00
category:
- Golang
- gRPC
tags:
- Golang
- gRPC
- Go
- protobuf
- proto
- 命令
- package
- Windows
- 可执行
- 环境
- 变量
---

## Windows 安装 protoc 可执行文件到环境变量

前往 [Releases · protocolbuffers/protobuf](https://github.com/protocolbuffers/protobuf/releases) 下载 `protoc-x.x.x-x.zip` 将 bin 下面的内容解压到 $PATH 配置的地址中：

![image.png](https://b3logfile.com/file/2021/09/image-65198a5d.png)

比如图片中放到了 `C:\Users\biuaxia\go\bin` 下面：

![image.png](https://b3logfile.com/file/2021/09/image-a8f9589c.png)

## 为 Go 语言安装 protobuf 的 package

执行命令：

```shell
go install google.golang.org/protobuf/cmd/protoc-gen-go
```

> 注意，默认安装目录为 `$GOPATH/bin`

## 验证

打开命令行输入 `protoc` 并回车，没有报错即可。

![image.png](https://b3logfile.com/file/2021/09/image-76eaa32a.png)

例如想要根据当前目录下的 `helloworld.proto` 文件来生成 `go` 文件，命令为：

```shell
protoc --go_out=. --go-grpc_out=. --go-grpc_opt=require_unimplemented_servers=false .\helloworld.proto
```

## 常见问题

### 'protoc-gen-go-grpc' 不是内部或外部命令，也不是可运行的程序

执行命令：

```shell
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

再次尝试即可。

## 参考资料

- [protocol buffers - protoc-gen-go-grpc: program not found or is not executable - Stack Overflow](https://stackoverflow.com/questions/60578892/protoc-gen-go-grpc-program-not-found-or-is-not-executable)
- [Go Generated Code  |  Protocol Buffers  |  Google Developers](https://developers.google.com/protocol-buffers/docs/reference/go-generated)
- [protocolbuffers/protobuf-go: Go support for Google's protocol buffers](https://github.com/protocolbuffers/protobuf-go)
- [protocolbuffers/protobuf: Protocol Buffers - Google's data interchange format](https://github.com/protocolbuffers/protobuf)
- [Language Guide  |  Protocol Buffers  |  Google Developers](https://developers.google.com/protocol-buffers/docs/overview)
