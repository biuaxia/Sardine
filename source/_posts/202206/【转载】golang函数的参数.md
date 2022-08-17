title: 【转载】golang函数的参数
date: 2022-06-17 16:58:00
toc: false
index_img: http://api.btstu.cn/sjbz/?lx=m_dongman&cid=4
category:
- Go
tags:
- Golang
---

> 原文：[golang函数的参数 - Golang技术栈，Golang文章、教程、视频分享！](https://golang-tech-stack.com/tutorial/golang/go-function-args)，本站仅作保存记录及调整排版，不做盈利性质的商业行为。

> 提示：本文有提到指针传递的类型！

go语言函数可以有0或多个参数，参数需要指定 **数据类型** 。

声明函数时的参数列表叫做形参，调用时传递的参数叫做实参。

go语言是通过**传值的方式传参**的，意味着传递给函数的是拷贝后的副本，所以函数内部访问、修改的也是这个副本。

go语言可以使用 **变长参数** ，有时候并不能确定参数的个数，可以使用变长参数，可以在函数定义语句的参数部分使用 `ARGS...TYPE`的方式。这时会将 `...`代表的参数全部保存到一个名为ARGS的slice中，注意这些参数的数据类型都是TYPE。

## go语言函数的参数实例

**go语言传参**

```go
// 形参列表
func f1(a int, b int) int {
	if a > b {
		return a
	} else {
		return b
	}
}

func main() {
	// 实参列表
	r := f1(1, 2)
	fmt.Printf("r: %v\n", r)
}
```

**演示参数传递，按值传递**

```go
func f1(a int) {
	a = 200
	fmt.Printf("a1: %v\n", a)
}

func main() {
	a := 100
	f1(a)
	fmt.Printf("a: %v\n", a)
}
```

运行结果

```
a1: 200
a: 100
```

从运行结果可以看到，调用函数f1后，a的值并没有被改变，说明参数传递是拷贝了一个副本，也就是拷贝了一份新的内容进行运算。

> `map`、`slice`、`interface`、`channel`这些数据类型本身就是**指针**类型的，所以就算是拷贝传值也是拷贝的指针，拷贝后的参数仍然指向底层数据结构，所以修改它们**可能**会影响外部数据结构的值。

```go
package main

import "fmt"

func f1(a []int) {
	a[0] = 100
}

func main() {
	a := []int{1, 2}
	f1(a)
	fmt.Printf("a: %v\n", a)
}
```

运行结果

```
a: [1 2]
a: [100 2]
```

> 从运行结果发现，调用函数后，slice内容被改变了。

**变长参数**

```go
package main

import "fmt"

func f1(args ...int) {
	for _, v := range args {
		fmt.Printf("v: %v\n", v)
	}
}
func f2(name string, age int, args ...int) {
	fmt.Printf("name: %v\n", name)
	fmt.Printf("age: %v\n", age)
	for _, v := range args {
		fmt.Printf("v: %v\n", v)
	}
}
func main() {
	f1(1, 2, 3)
	fmt.Println("------------")
	f1(1, 2, 3, 4, 5, 6)
	fmt.Println("------------")
	f2("tom", 20, 1, 2, 3)
}
```

运行结果

```
v: 1
v: 2
v: 3
------------
v: 1
v: 2
v: 3
v: 4
v: 5
v: 6
------------
name: tom
age: 20
v: 1
v: 2
v: 3
```
