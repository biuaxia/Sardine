title: protobuf和gRPC进阶
date: 2021-09-14 15:05:00
toc: true
category:
- Golang
- gRPC
tags:
- Golang
- gRPC
- Go
- protobuf
- proto
- 类型
- 默认值
- 命令
- package
- 同步
- 坑
- Map
- 枚举
- 泛型
- message
- 嵌套
- 引用
- 对象
- 属性
---

## protobuf 的基本类型和默认值

参考下表：

标量消息字段可以具有以下类型之一–该表显示了文件中指定的类型，以及自动生成的类中相应的类型：`.proto`。
| .proto Type | Notes                                                                      | C++ Type | Java/Kotlin Type | Python Type | Go Type | Ruby Type                      | C# Type    | PHP Type           | Dart Type |
| ----------- | -------------------------------------------------------------------------- | -------- | -------------------- | --------------- | ------- | ------------------------------ | ---------- | ------------------ | --------- |
| double      |                                                                            | double   | double               | float           | float64 | Float                          | double     | float              | double    |
| float       |                                                                            | float    | float                | float           | float32 | Float                          | float      | float              | double    |
| int32       | 使用可变长度编码。编码负数效率低下–如果您的字段可能有负值，请改用sint32。 | int32    | int                  | int             | int32   | Fixnum or Bignum (as required) | int        | integer            | int       |
| int64       | 使用可变长度编码。编码负数效率低下–如果您的字段可能有负值，请改用sint64。 | int64    | long                 | int/long    | int64   | Bignum                         | long       | integer/string | Int64     |
| uint32      | 使用可变长度编码。                                                         | uint32   | int              | int/long    | uint32  | Fixnum or Bignum (as required) | uint       | integer            | int       |
| uint64      | 使用可变长度编码。                                                         | uint64   | long             | int/long    | uint64  | Bignum                         | ulong      | integer/string | Int64     |
| sint32      | 使用可变长度编码。有符号整数值。它们比常规的int32s更有效地编码负数。       | int32    | int                  | int             | int32   | Fixnum or Bignum (as required) | int        | integer            | int       |
| sint64      | 使用可变长度编码。有符号整数值。它们比常规的int64s更有效地编码负数。       | int64    | long                 | int/long    | int64   | Bignum                         | long       | integer/string | Int64     |
| fixed32     | 总是四个字节。如果值通常大于2^28^，则比uint32更有效。                      | uint32   | int              | int/long    | uint32  | Fixnum or Bignum (as required) | uint       | integer            | int       |
| fixed64     | 总是八个字节。如果值通常大于2^56^，则比uint64更有效。                      | uint64   | long             | int/long    | uint64  | Bignum                         | ulong      | integer/string | Int64     |
| sfixed32    | 总是四个字节。                                                             | int32    | int                  | int             | int32   | Fixnum or Bignum (as required) | int        | integer            | int       |
| sfixed64    | 总是八个字节。                                                             | int64    | long                 | int/long    | int64   | Bignum                         | long       | integer/string | Int64     |
| bool        |                                                                            | bool     | boolean              | bool            | bool    | TrueClass/FalseClass           | bool       | boolean            | bool      |
| string      | 字符串必须始终包含UTF-8编码或7位ASCII文本，且长度不能超过2^32^。           | string   | String               | str/unicode | string  | String (UTF-8)                 | string     | string             | String    |
| bytes       | 可能包含任意顺序的字节数据，但不超过2^32^。                                | string   | ByteString           | str             | []byte  | String (ASCII-8BIT)            | ByteString | string             | List      |

你可以在文章[Protocol Buffer 编码](https://developers.google.com/protocol-buffers/docs/encoding?hl=zh-cn)中，找到更多“序列化消息时各种类型如何编码”的信息。

## option go_package 的作用

我们在定义一个.proto文件时，需要申明这个文件属于哪个包,主要是为了规范整合以及避免重复，这个概念在其他语言中也存在，比如php中的 `namespace`的概念，go中的 `package`概念。

所以，我们根据实际的分类情况，给每1个proto文件都定义1个包，一般这个包和proto所在的文件夹名子，保持一致。

比如例子中，文件在 `proto文件夹`中，那我们用的package 就为： `proto`;

`option` 单独看这个名字，就知道是选项和配置的意思，常见的选项是配置 `go_package`

```protobuf
option go_package = ".;proto";
```

现在protoc命令生成go包的时候，如果这一行没加上，会提示错误：

```protobuf
➜  proto git:(master) ✗ protoc --go_out=:. hello.proto
2020/05/21 15:59:40 WARNING: Missing 'go_package' option in "hello.proto", please specify:
        option go_package = ".;proto";
A future release of protoc-gen-go will require this be specified.
See https://developers.google.com/protocol-buffers/docs/reference/go-generated#package for more information.
```

所以，这个 `go_package`和上面那个 `package proto;`有啥区别呢？有点蒙啊。

尝试这样改一下：

```protobuf
syntax = "proto3";

package protoB;

option go_package = ".;protoA";
```

看下生成的go语言包的package到底是啥？打开，生成后的go文件：

```go
# vi hello.pb.go

package protoA

...
```

发现是 `protoA`，说明，go的package是受 `option go_package`影响的。所以，在我们没有申请这一句的时候，系统就会用proto文件的package名字，为提示，让你也加上相同的go_package名字了。

再来看一下，这个 `=".;proto"` 是啥意思。改一下：

```protobuf
option go_package = "./protoA";
```

执行后，发现，生成了一个 `protoA`文件夹。里面的 `hello.pb.go` 文件的 `package` 也是 `protoA`。

所以，`.;`表示的是就在本目录下的意思么？？？行吧。

再来看一下，我们改成1个绝对的路径目录：

```protobuf
option go_package = "/";
```

所以，总结一下：

```protobuf
package protoB; // 这个用来设定proto文件自身的package

option go_package = ".;protoA";  // 这个用来生成的go文件package。一般情况下，可以把这2个设置成一样
```

## proto 文件同步时的坑

简而言之就是客户端与服务端所使用的 `proto` 文件中对象属性的序号不同，导致业务逻辑混乱，且无法排查。

例如：

```go
message StreamReqData{
    string name = 1;
    string url = 2;
}
```

`protobuf` 在序列化时的逻辑大概为：

`name = "biuaxia", -> 17biuaxia`，其中的 1 为属性 name 的序号，7 为内容长度。

同理，`url = "biuaxia.cn", -> 210biuaxia.cn`。

如果服务端与客户端的 proto 文件中属性的序号不同则会造成难以排查的问题。

对此最好的解决办法就是不要修改 proto 文件，而是接收针对 proto 文件的统一分发。

## proto 文件嵌套引用

例如 hello.proto 引用 base.proto。

hello.proto 文件如下：

```protobuf
syntax = "proto3";

option go_package = "../proto";

service Greeter {
    rpc Ping(Empty) returns (Pong);
}
```

base.proto 文件如下：

```protobuf
syntax = "proto3";

message Empty {
}

message Pong {
    string id = 1;
}
```

直接生成或IDE中可以看到 `hello.proto` 的第6行(即 Empty 和 Pong)会报错，可以通过 `import` 关键字来实现。

修改 hello.proto 在其第三行追加内容：

```protobuf
syntax = "proto3";

import "base.proto";

option go_package = "../proto";

service Greeter {
    rpc Ping(Empty) returns (Pong);
}
```

> 注意：import 文件时，后面的内容为当前文件的相对目录。

也可以用于引入带包结构的 proto 文件且使用，例如：

```protobuf
syntax = "proto3";

import "base.proto";
import "google/protobuf/empty.proto";

option go_package = "../proto";

service Greeter {
    rpc Ping(google.protobuf.Empty) returns (Pong);
}
```

> 注意：外部文件需要写完整的包名，本地文件不需要。

想要查询 proto 自带有哪些数据可以通过 Idea 查看下图的位置，将红色的位置追加上 `.proto` 文件即可。

![image.png](https://b3logfile.com/file/2021/09/image-3e80a50f.png)

## message 嵌套

可以直接嵌套，如：

```protobuf
syntax = "proto3";

option go_package = "../proto";

message Hello {
    message Result {
        string code = 1;
        string msg = 2;
    }
  
    string ret = 1;
    Result msg = 2;
    Result data = 3;
}
```

> 注意：import 嵌套会导致被 import 的 proto 文件包含的 message 不被生成。解决办法就是再手动生成被 proto 的文件。

## 使用枚举类型

直接定义，如：

```protobuf
syntax = "proto3";

option go_package = "../proto";

enum Gender{
    MALE = 1;
    FEMALE = 2;
}

message Request {
    Gender g = 1;
}
```

即可。

## 使用 Map

使用示例：

```protobuf
syntax = "proto3";

option go_package = "../proto";

message Request {
    map<string, string> mp = 1;
}
```

> 注意：一定要和泛型一样指定 key 和 value 的类型

## 使用时间戳 timestamp

首先在 proto 文件中引入 timestamp

```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";

option go_package = "../proto";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest {
  string name = 1;
  google.protobuf.Timestamp addTime = 2;
}

message HelloReply {
  string message = 1;
}
```

然后客户端直接调用 `"google.golang.org/protobuf/types/known/timestamppb"` 此目录下的 `New(time.Time)` 即可生成对应的类型，如果不知道 `New(time.Time)` 是怎么查询到的，建议使用 Goland，它会自动有提示。

```go
package main

import (
	"biuaxia.cn/demo/grpc_test/proto"
	"context"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/protobuf/types/known/timestamppb"
	"time"
)

func main() {
	conn, err := grpc.Dial("localhost:1234", grpc.WithInsecure())
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	c := proto.NewGreeterClient(conn)
	reply, err := c.SayHello(context.Background(), &proto.HelloRequest{
		Name:    "biuaxia",
		AddTime: timestamppb.New(time.Now()),
	})
	if err != nil {
		panic(err)
	}

	fmt.Println(reply.Message)
}
```

服务端无需修改：

```go
package main

import (
	"biuaxia.cn/demo/grpc_test/proto"
	"context"
	"google.golang.org/grpc"
	"net"
)

type Server struct {
}

func (s *Server) SayHello(ctx context.Context, request *proto.HelloRequest) (*proto.HelloReply, error) {
	time := request.AddTime.AsTime().Format("2006-01-02 15:04:05.000")
	return &proto.HelloReply{
		Message: "hello " + request.Name + "_" + time,
	}, nil
}

func main() {
	s := grpc.NewServer()
	proto.RegisterGreeterServer(s, &Server{})

	listen, err := net.Listen("tcp", "localhost:1234")
	if err != nil {
		panic(err)
	}

	_ = s.Serve(listen)
}
```

## metadata 机制

可以理解为http的请求头，用于携带 token 等请求信息，与业务信息隔开理解即可。

> 注意：引入的包为 `"google.golang.org/grpc/metadata"`。

使用示例-客户端：

```go
package main

import (
	"context"
	"fmt"
	"time"

	"google.golang.org/grpc"
	"google.golang.org/grpc/metadata"
	"google.golang.org/protobuf/types/known/timestamppb"

	"biuaxia.cn/demo/grpc_test/proto"
)

func main() {
	conn, err := grpc.Dial("localhost:1234", grpc.WithInsecure())
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	c := proto.NewGreeterClient(conn)

	// md := metadata.Pairs("timestamp", time.Now().Format("2006-01-02 15:04:05.000"))
	md := metadata.New(map[string]string{
		"name": "biuaxia",
	})
	ctx := metadata.NewOutgoingContext(context.Background(), md)

	reply, err := c.SayHello(ctx, &proto.HelloRequest{
		Name:    "biuaxia",
		AddTime: timestamppb.New(time.Now()),
	})
	if err != nil {
		panic(err)
	}

	fmt.Println(reply.Message)
}
```

使用示例-服务端：

```go
package main

import (
	"context"
	"fmt"
	"net"

	"google.golang.org/grpc"
	"google.golang.org/grpc/metadata"

	"biuaxia.cn/demo/grpc_test/proto"
)

type Server struct {
}

func (s *Server) SayHello(ctx context.Context, request *proto.HelloRequest) (*proto.HelloReply, error) {
	md, ok := metadata.FromIncomingContext(ctx)
	if !ok {
		fmt.Println("get metadata failed")
	}
	fmt.Println("get metadata:", md)
	time := request.AddTime.AsTime().Format("2006-01-02 15:04:05.000")
	return &proto.HelloReply{
		Message: "hello " + request.Name + "_" + time,
	}, nil
}

func main() {
	s := grpc.NewServer()
	proto.RegisterGreeterServer(s, &Server{})

	listen, err := net.Listen("tcp", "localhost:1234")
	if err != nil {
		panic(err)
	}

	_ = s.Serve(listen)
}
```

运行截图：

![服务端.png](https://b3logfile.com/file/2021/09/Snipaste_2021-09-26_12-04-59-d56dd1a7.png)

![客户端.png](https://b3logfile.com/file/2021/09/Snipaste_2021-09-26_12-05-30-72dc1d14.png)

## 拦截器

使用示例-服务端：

```go
package main

import (
	"context"
	"fmt"
	"net"

	"google.golang.org/grpc"

	"awesomeProject/grpc/interceptor/main/proto"
)

type Server struct{}

func (s *Server) SayHello(ctx context.Context, request *proto.HelloRequest) (*proto.HelloReply,
	error) {
	return &proto.HelloReply{
		Message: "hello " + request.Name,
	}, nil
}

func interceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (resp interface{}, err error) {
	// 继续处理请求
	fmt.Println("接收到新请求")
	res, err := handler(ctx, req)
	fmt.Println("请求处理完成")
	return res, err
}

func main() {
	var opts []grpc.ServerOption
	opts = append(opts, grpc.UnaryInterceptor(interceptor))

	g := grpc.NewServer(opts...)
	proto.RegisterGreeterServer(g, &Server{})
	lis, err := net.Listen("tcp", "0.0.0.0:50051")
	if err != nil {
		panic("failed to listen:" + err.Error())
	}
	err = g.Serve(lis)
	if err != nil {
		panic("failed to start grpc:" + err.Error())
	}
}
```

使用示例-客户端：

```go
package main

import (
	"context"
	"fmt"
	"time"

	"google.golang.org/grpc"

	"awesomeProject/grpc/interceptor/main/proto"
)

func interceptor(ctx context.Context, method string, req, reply interface{}, cc *grpc.ClientConn, invoker grpc.UnaryInvoker, opts ...grpc.CallOption) error {
	start := time.Now()
	err := invoker(ctx, method, req, reply, cc, opts...)
	fmt.Printf("method=%s req=%v rep=%v duration=%s error=%v\n", method, req, reply, time.Since(start), err)
	return err
}

func main(){
	//stream
	var opts []grpc.DialOption

	opts = append(opts, grpc.WithInsecure())
	// 指定客户端interceptor
	opts = append(opts, grpc.WithUnaryInterceptor(interceptor))

	conn, err := grpc.Dial("localhost:50051", opts...)
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	c := proto.NewGreeterClient(conn)
	r, err := c.SayHello(context.Background(), &proto.HelloRequest{Name:"bobby"})
	if err != nil {
		panic(err)
	}
	fmt.Println(r.Message)
}
```

### 验证请求信息

使用示例-服务端：

```go
package main

import (
	proto2 "awesomeProject/grpc/interceptor/samples/auth_verify/proto"
	"context"
	"fmt"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/metadata"
	"google.golang.org/grpc/status"
	"net"

	"google.golang.org/grpc"
)

type Server struct{}

func (s *Server) SayHello(ctx context.Context, request *proto2.HelloRequest) (*proto2.HelloReply,
	error) {
	return &proto2.HelloReply{
		Message: "hello " + request.Name,
	}, nil
}

func interceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (resp interface{}, err error) {
	md, ok := metadata.FromIncomingContext(ctx)
	if !ok {
		fmt.Println("get metadata failed")
		return resp, status.Error(codes.Unauthenticated, "无授权认证信息")
	}

	var (
		appkey    string
		appsecret string
	)

	if val, ok := md["appkey"]; ok {
		appkey = val[0]
	}
	if val, ok := md["appsecret"]; ok {
		appsecret = val[0]
	}

	if appkey != "vditor" || appsecret != "b1d0d1ad98acdd5d7d846d" {
		return resp, status.Error(codes.Unauthenticated, "授权认证信息有误")
	}

	fmt.Println("get metadata:", md)

	// 继续处理请求
	fmt.Println("接收到新请求")
	res, err := handler(ctx, req)
	fmt.Println("请求处理完成")
	return res, err
}

func main() {
	var opts []grpc.ServerOption
	opts = append(opts, grpc.UnaryInterceptor(interceptor))

	g := grpc.NewServer(opts...)
	proto2.RegisterGreeterServer(g, &Server{})
	lis, err := net.Listen("tcp", "0.0.0.0:50051")
	if err != nil {
		panic("failed to listen:" + err.Error())
	}
	err = g.Serve(lis)
	if err != nil {
		panic("failed to start grpc:" + err.Error())
	}
}
```

使用示例-客户端：

```go
package main

import (
	proto2 "awesomeProject/grpc/interceptor/samples/auth_verify/proto"
	"context"
	"fmt"
	"time"

	"google.golang.org/grpc"
	"google.golang.org/grpc/metadata"
)

func interceptor(ctx context.Context, method string, req, reply interface{}, cc *grpc.ClientConn, invoker grpc.UnaryInvoker, opts ...grpc.CallOption) error {
	start := time.Now()

	md := metadata.New(map[string]string{
		"appkey":    "vditor",
		"appsecret": "b1d0d1ad98acdd5d7d846d",
	})
	ctx = metadata.NewOutgoingContext(context.Background(), md)

	err := invoker(ctx, method, req, reply, cc, opts...)
	fmt.Printf("method=%s req=%v rep=%v duration=%s error=%v\n", method, req, reply, time.Since(start), err)
	return err
}

type customCredential struct {
}

func (c customCredential) GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error) {
	return map[string]string{
		"appkey":    "vditor",
		"appsecret": "b1d0d1ad98acdd5d7d846d",
	}, nil
}

func (c customCredential) RequireTransportSecurity() bool {
	return false
}

func main() {
	var opts []grpc.DialOption

	opts = append(opts, grpc.WithInsecure())
	// 指定客户端interceptor
	//opts = append(opts, grpc.WithUnaryInterceptor(interceptor))
	opts = append(opts, grpc.WithPerRPCCredentials(customCredential{}))

	conn, err := grpc.Dial("localhost:50051", opts...)
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	c := proto2.NewGreeterClient(conn)
	r, err := c.SayHello(context.Background(), &proto2.HelloRequest{Name: "bobby"})
	if err != nil {
		panic(err)
	}
	fmt.Println(r.Message)
}
```

> `grpc/interceptor/samples/auth_verify/client/client.go:46` 被注释的内容是原生的调用方式，47 行是 grpc 自己封装的方式；需要对象实现 `google.golang.org/grpc/credentials/credentials.go` 下 `PerRPCCredentials` 接口的 `GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error)` 和 `RequireTransportSecurity() bool` 方法。

[^1]: Kotlin 使用来自 Java 的相应类型，即使是未签名的类型，以确保混合 Java/Kotlin 代码库的兼容性。

[^2]: 在 Java 中，未签名的 32 位和 64 位整数使用其签名的对应器进行表示，顶部位只需存储在符号位中即可。

[^3]: 在所有情况下，将值设置为字段将执行类型检查，以确保其有效。

[^4]: 64 位或未签名的 32 位整数在解码时始终表示，但如果在设置字段时给出了国际数字，则可以是国际。在所有情况下，值必须与设置时所表示的类型相符。请参阅 [^3]。

[^5]: Python 字符串在解码上表示为单码，但如果给出 ASCII 字符串（这可能会更改），则可以进行 str。

[^6]: `$` 整数用于 64 位机器，字符串用于 32 位机器。
