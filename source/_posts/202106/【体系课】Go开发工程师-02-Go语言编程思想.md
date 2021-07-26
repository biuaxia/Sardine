title: 【体系课】Go开发工程师-02-Go语言编程思想
date: '2021-06-18 16:43:21'
updated: '2021-07-13 17:13:50'
category:
 - Go
tags: [慕课, imooc, 体系课, Go]
permalink: /articles/2021/06/18/1626002681827.html
---
## 面向接口

### 接口的概念

例如想要编写”获取指定网址源码“的方法：

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	resp, err := http.Get("http://qq.com")
	if err != nil {
		panic(err)
	}

	defer resp.Body.Close()

	bytes, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		panic(err)
	}

	fmt.Printf("%s", bytes)
}
```

可以通过定义函数，从而简化 `main` 中的代码：

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func retrieve(url string) string {
	resp, err := http.Get(url)
	if err != nil {
		panic(err)
	}

	defer resp.Body.Close()

	bytes, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		panic(err)
	}

	return string(bytes)
}

func main() {
	fmt.Println(retrieve("http://qq.com"))
}
```

这里的 main 函数和 retrieve 函数的耦合性很高，为了解决这个问题引出了 接口的概念。

例如：

1. 结构包

   ```go
   package infra

   import (
   	"io/ioutil"
   	"net/http"
   )

   type Retrieverr struct{}

   func Get(url string) string {
   	resp, err := http.Get(url)
   	if err != nil {
   		panic(err)
   	}

   	defer resp.Body.Close()

   	bytes, err := ioutil.ReadAll(resp.Body)
   	if err != nil {
   		panic(err)
   	}

   	return string(bytes)
   }
   ```
2. main代码

   ```go
   package infra

   import (
   	"fmt"

   	"github.com/biuaxia/infra"
   )

   func GetRetriever() infra.Retriever {
   	return infra.Retriever{}
   }

   func main() {
   	// retriever := infra.Retriever{}
   	var retriever infra.Retriever = getRetriever()
   	fmt.Println(retriever.Get("http://qq.com"))
   }
   ```

但这时候如果有其他的 retriever 想要被 main 函数使用，但又不能修改 getRetriever 的 return。

就可以引出接口的概念：

```go
package infra

import (
	"fmt"

	"github.com/biuaxia/testing"
)

type retriever interface {
	Get(string) string
}

func GetRetriever() retriever {
	return testing.Retriever{}
}

func main() {
	var r = getRetriever()
	fmt.Println(retriever.Get("http://qq.com"))
}
```

抽取共同的 Get 方法（特征）用 interface 关键字定义。

```go
type retriever interface {
	Get(string) string
}
```

testing 和 infra 的 Retriever 都是可以正常运行的。

接口是一个抽象的概念，这里是指实现了 Get 方法的都算是接口的实现。

### duck typing 的概念

- ”像鸭子走路，像鸭子叫（长得想鸭子），那么就是鸭子“
- **描述事物的外部行为而非内部结构**
- 严格说 go 属于结构化类型系统，类似 duck typing

---

go 语言的 duck typing

- 同时需要 Readable，Appendable 怎么办？（apache polygene）：特指 Java
- 同时具有 Python，C++ 的 duck typing 的灵活性
- 又具有 Java 的类型检查

### 接口的定义和实现

参见代码：

```go
package main

import (
	"fmt"
	"time"
)

type Retriever interface {
	Get(url string) string
}

func download(r Retriever) string {
	return r.Get("http://qq.com")
}

type MockRetriever struct {
	Contents string
}

func (r MockRetriever) Get(url string) string {
	return r.Contents
}

type RealRetriever struct {
	UserAgent string
	Timeout   time.Duration
}

func (r RealRetriever) Get(url string) string {
	return fmt.Sprintf("%s -> %s", r.Timeout, r.UserAgent)
}

func main() {
	var r Retriever
	r = MockRetriever{"This is mock retriever contens."}
	fmt.Printf("%T, %+[1]v\n", r)

	r = RealRetriever{
		"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36 Edg/91.0.864.67",
		time.Second,
	}
	fmt.Printf("%T, %+[1]v\n", r)

}
```

运行结果：

![image.png](https://b3logfile.com/file/2021/07/image-e87398d9.png)

### 接口的值类型

下面的代码分别展示了值类型和指针类型的传递方式：

```go
package main

import (
	"fmt"
	"time"
)

type Retriever interface {
	Get(url string) string
}

func download(r Retriever) string {
	return r.Get("http://qq.com")
}

type MockRetriever struct {
	Contents string
}

func (r MockRetriever) Get(url string) string {
	return r.Contents
}

type RealRetriever struct {
	UserAgent string
	Timeout   time.Duration
}

func (r *RealRetriever) Get(url string) string {
	return fmt.Sprintf("%s -> %s", r.Timeout, r.UserAgent)
}

func main() {
	var r Retriever
	r = MockRetriever{"This is mock retriever contens."}
	fmt.Printf("%T, %+[1]v\n", r)

	r = &RealRetriever{
		"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36 Edg/91.0.864.67",
		time.Second,
	}
	fmt.Printf("%T, %+[1]v\n", r)

}
```

MockRetriever是值类型传递（拷贝），RealRetriever是指针传递（引用）。

---

#### 如何判断接口具体的类型

```go
package main

import (
	"fmt"
	"time"
)

type Retriever interface {
	Get(url string) string
}

func download(r Retriever) string {
	return r.Get("http://qq.com")
}

type MockRetriever struct {
	Contents string
}

func (r MockRetriever) Get(url string) string {
	return r.Contents
}

type RealRetriever struct {
	UserAgent string
	Timeout   time.Duration
}

func (r *RealRetriever) Get(url string) string {
	return fmt.Sprintf("%s -> %s", r.Timeout, r.UserAgent)
}

func main() {
	var r Retriever
	r = MockRetriever{"This is mock retriever contens."}
	fmt.Printf("%T, %+[1]v\n", r)

	r = &RealRetriever{
		"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36 Edg/91.0.864.67",
		time.Second,
	}
	fmt.Printf("%T, %+[1]v\n", r)

	fmt.Println("-------------------------------")

	switch v := r.(type) {
	case MockRetriever:
		fmt.Println("MockRetriever：", v.Contents)
	case *RealRetriever:
		fmt.Println("RealRetriever：", v.Timeout)
	}
}
```

> 注意：如果是指针类型请不要直接使用 `case RealRetriever` 这种形式，而要使用 `case *RealRetriever`。

#### 接口类型变量的强制转换

```go
package main

import (
	"fmt"
	"time"
)

type Retriever interface {
	Get(url string) string
}

func download(r Retriever) string {
	return r.Get("http://qq.com")
}

type MockRetriever struct {
	Contents string
}

func (r MockRetriever) Get(url string) string {
	return r.Contents
}

type RealRetriever struct {
	UserAgent string
	Timeout   time.Duration
}

func (r *RealRetriever) Get(url string) string {
	return fmt.Sprintf("%s -> %s", r.Timeout, r.UserAgent)
}

func inspect(r Retriever) {
	switch v := r.(type) {
	case MockRetriever:
		fmt.Println("MockRetriever：", v.Contents)
	case *RealRetriever:
		fmt.Println("RealRetriever：", v.Timeout)
	}
}

func main() {
	var r Retriever
	r = MockRetriever{"This is mock retriever contens."}
	fmt.Printf("%T, %+[1]v\n", r)

	r = &RealRetriever{
		"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36 Edg/91.0.864.67",
		time.Second,
	}
	fmt.Printf("%T, %+[1]v\n", r)

	fmt.Println("-------------------------------")
	inspect(r)
	fmt.Println("-------------------------------")

	// realRetriever := r.(*RealRetriever)
	// fmt.Println(realRetriever.Timeout)
	mockRetriever := r.(MockRetriever)
	fmt.Println(mockRetriever.Contents)
}
```

运行结果为：

![image.png](https://b3logfile.com/file/2021/07/image-9cc32538.png)

可以看到最后的 r 实际类型为 `RealRetriever` ，尝试强制将它转换为 `MockRetriever` 会出现错误。

可以使用 `val, ok := r.(class)` 的方式，`ok` 成立的情况表示转换成，反之转换失败。

```go
package main

import (
	"fmt"
	"time"
)

type Retriever interface {
	Get(url string) string
}

func download(r Retriever) string {
	return r.Get("http://qq.com")
}

type MockRetriever struct {
	Contents string
}

func (r MockRetriever) Get(url string) string {
	return r.Contents
}

type RealRetriever struct {
	UserAgent string
	Timeout   time.Duration
}

func (r *RealRetriever) Get(url string) string {
	return fmt.Sprintf("%s -> %s", r.Timeout, r.UserAgent)
}

func inspect(r Retriever) {
	switch v := r.(type) {
	case MockRetriever:
		fmt.Println("MockRetriever：", v.Contents)
	case *RealRetriever:
		fmt.Println("RealRetriever：", v.Timeout)
	}
}

func main() {
	var r Retriever
	r = MockRetriever{"This is mock retriever contens."}
	fmt.Printf("%T, %+[1]v\n", r)

	r = &RealRetriever{
		"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36 Edg/91.0.864.67",
		time.Second,
	}
	fmt.Printf("%T, %+[1]v\n", r)

	fmt.Println("-------------------------------")
	inspect(r)
	fmt.Println("-------------------------------")

	// realRetriever := r.(*RealRetriever)
	// fmt.Println(realRetriever.Timeout)
	// mockRetriever := r.(MockRetriever)
	// fmt.Println(mockRetriever.Contents)

	if mockRetriever, ok := r.(MockRetriever); ok {
		fmt.Println(mockRetriever.Contents)
	} else {
		fmt.Println("转换失败")
	}
}
```

运行结果为：

![image.png](https://b3logfile.com/file/2021/07/image-3bbb4507.png)

---

> 一个 interface 包含了最基础的两个东西：
>
> - 实现者的类型，使用 `%T`
> - 实现者的值（指针），使用 `%v`
>
> ---
>
> 判断类型的方式：
>
> - `Switch-Case` 关键字
> - `Type assertion`，即 `r.(MockRetriever)` 这种方式，它还可以在接收的时候使用 `ok` 参数来判断转换是否成功。
>
> ---
>
> 接口变量里面有什么：
>
> - 接口变量自带指针
> - 接口变量同样采用值传递，几乎不需要使用接口的指针
> - 指针接收者实现只能以指针方式使用；值接收者都可以使用


### 接口的组合

### 常用系统接口

## 函数式编程

### 函数式编程例一

### 函数式编程例二

## 错误处理和资源管理

### defer调用

### 错误处理概念

### 服务器统一出错处理1

### 服务器统一出错处理2

## 测试与性能调优

### 测试

### 代码覆盖率和性能测试

### 使用pprof进行性能调优

### 测试http服务器（上）

### 测试http服务器（下）

### 生成文档和示例代码

### 测试总结

## Goroutine

### go语言的调度器

## Channel

### 使用channel等待任务结束

### 使用channel进行树的遍历

### select

### 传统同步机制

### 并发模式（上）

### 并发模式（下）

### 并发任务的控制

## 迷宫的广度优先算法

### 迷宫_算法

### 迷宫代码实现

## http及其他标准库

### http标准库

### json数据格式的处理

### 第三方API数据格式的解析技巧

### gin框架介绍

### 为gin增加middleware
