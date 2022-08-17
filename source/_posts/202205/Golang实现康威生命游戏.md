title: Golang实现康威生命游戏
date: 2022-05-23 17:15:00
toc: false
index_img: http://api.btstu.cn/sjbz/?lx=m_dongman&cid=2
category:
- Go
tags:
- 读取
- 创建
- Golang
- 遍历
- 计算
- 解决
- 模拟
- 输出
- 统计
- 打印
- 游戏
---

# 实验：切片人生

本次实验将要构建一个名为“康威生命游戏”（Conway's Game of Life）的模拟器，并使用它模拟人类的繁衍过程。因为模拟需要在一个布满细胞的二维网格上进行，所以这次实验将聚焦于切片。

网格中的每个细胞在水平、垂直和对角线方向上总共有 8 个相邻细胞。在每一世代，单个细胞的生死存亡将取决于相邻细胞的存活数量。

## 开天辟地

在初次实现生命游戏时，我们需要将世界限制在固定的大小之内。具体来说，我们需要决定网格的尺寸并定义相应的常量：

```go
const (
  width  = 80
  height = 15
)
```

接着还需要定义 `Universe` 类型用于持有二维细胞网格，并通过布尔类型的值 `true` 和 `false` 分别表示细胞的存活和死亡：

```go
type Universe [][]bool
```

通过使用切片而不是数组来表示世界，可以让函数和方法更容易地共享和修改世界。

在此之后，我们还要编写 `NewUniverse` 函数，它使用 `make` 分配并返回一个 `height` 行 `width` 列的 `Universe`：

```go
func NewUniverse() Universe
```

因为新分配切片的各个元素将被设置为默认的零值 `false`，所以世界在刚开始的时候将不存在任何存活细胞。

## 观察世界

请为 `Universe` 编写一个方法，它能够用 `fmt` 包中的函数将世界目前的状态打印至屏幕，其中存活的细胞用星号表示，而死亡的细胞则用空格表示。此外，它还需要在每次打印完一行细胞之后，将光标移动至新的输出行：

```go
func (u Universe) Show()
```

请编写一个 `main` 函数，它会调用 `NewUniverse` 函数创造出新世界，然后调用 `Show` 函数把这个世界打印出来。在继续进行实验之前，请先确保你的程序能够正常运行，即使整个世界目前还没有存活细胞。

## 激活细胞

请编写一个 `Seed` 方法，它可以随机激活世界中大约 25% 的细胞（将对应切片元素的值设置为 true）：

```go
func (u Universe) Seed()
```

在实现这个方法的时候，别忘了导入 `math/rand` 包以使用 `Intn` 函数。在此之后，请修改 `main` 函数并使用 `Seed` 方法对世界进行激活，然后使用 `Show` 函数将激活后的世界打印出来。

## 适者生存

以下是康威生命游戏的具体规则：

- 当一个存活细胞邻近的存活细胞少于 2 个时，该细胞死亡。
- 当一个存活细胞邻近有 2 个或 3 个存活细胞时，该细胞将延续至下一世代。
- 当一个存活细胞邻近有多余 3 个存活细胞时，该细胞死亡。
- 当一个死亡细胞邻近正好有 3 个存活细胞时，该细胞存活。

为了实现这些规则，我们需要将它们分解成以下 3 个步骤，并将每个步骤实现为相应的方法：

- 判断细胞是否存活的方法
- 统计邻近存活细胞数量的能力
- 判断细胞在下一世代存活或死亡的逻辑

### 存活还是死亡

判断细胞是否存活可以通过检查 `Universe` 切片中对应元素的布尔值来实现，只要该值为 `true`，那么细胞就是存活的。

请为 `Universe` 类型编写一个带有以下签名的 `Alive` 方法：

```go
func (u Universe) Alive(x, y int) bool
```

实现 `Alive` 方法最困难的就是处理越界情况。例如，我们如何判断位于 `(-1, -1)` 的细胞存活还是死亡呢？或者，我们如何在一个 `80x15` 的网格上，判断位于 `(80, 15)` 的细胞存活还是死亡呢？

为了解决这个问题，我们需要为世界实现回绕。这样一来，与 `(0, 0)` 相邻的上方将不再是 `(0. -1)`，而是 `(0, 14)`，这一点可以通过将 `height` 与 `y` 相加得出。如果 `y` 超过了网格的 `height`，就需要用到之前计算闰年时介绍过的取模运算符（`%`），然后通过对 `y` 取模 `height` 来得出相应的余数。这一方法也适用于 `x` 和 `width`。

### 统计相邻细胞

请编写一个方法，统计给定细胞邻近的存活细胞数量，然后返回 `0~8`：

```go
func (u Universe) Neighbors(x, y int) int
```

为了使世界实现回绕，请使用 `Alive` 方法而不是直接访问世界数据。

另外需要注意的是，在统计相邻细胞的时候别把给定的细胞也统计进去了。

### 游戏逻辑

在实现了统计邻近存活细胞数量的方法之后，我们就可以正式在 `Next` 方法里面实现本节开头列出的游戏规则了：

```go
func (u Universe) Next(x, y int) bool
```

这个方法不会直接修改世界，而会返回一个布尔值，并以此来表示给定细胞在下一世代存活或死亡。

## 平行世界

为了完成模拟操作，程序需要遍历世界中的每个细胞，并使用 `Next` 判断它们在下一世代中的状态。

这里有一个需要注意的问题，那就是统计邻近细胞必须基于世界先前的状态。如果程序在执行统计的同时直接修改世界，那么这样的修改势必会对邻近细胞的统计结果产生影响。

解决这个问题的一个简单办法就是创建两个同等大小的世界，然后在读取世界 `A` 时候对世界 `B` 进行设置。请编写函数 `Step` 以执行该操作：

```go
func Step(a, b Universe)
```

当世界 `B` 被更新到了下一世代之后，程序就可以交换这两个世界，然后继续下一次更新：

```go
a, b = b, a
```

在展示新时代的细胞之前，程序需要使用特殊的 `ANSI` 转义序列 `"\x0c"` 来清空屏幕。在此之后，程序就可以打印出整个世界，并使用 `time` 包中的 `Sleep` 函数来减缓世代更迭的速度。

> 注意：在 `Go Playground` 以外的地方，你需要使用其他机制才能清空屏幕，例如，在 `macOS` 上就需要打印 `"\033[H"` 而不是 `"\x0c"`。

现在，你应该已经有了编写并且在 `Go Playground` 上运行完整的康威生命游戏所需的全部组件。

## 实现

### 截止到开天辟地的代码

```go
package main

import "fmt"

const (
	width  = 25
	height = 15
)

type Universe [][]bool

func NewUniverse() Universe {
	u := make(Universe, height)
	for i := range u {
		u[i] = make([]bool, width)
	}
	return u
}

func main() {
	u := NewUniverse()
	for _, i := range u {
		fmt.Println(i)
	}
}
```

运行结果：

```text
[false false false false false false false false false false false false false false false false false false false false false false false false false]
[false false false false false false false false false false false false false false false false false false false false false false false false false]
[false false false false false false false false false false false false false false false false false false false false false false false false false]
[false false false false false false false false false false false false false false false false false false false false false false false false false]
[false false false false false false false false false false false false false false false false false false false false false false false false false]
[false false false false false false false false false false false false false false false false false false false false false false false false false]
[false false false false false false false false false false false false false false false false false false false false false false false false false]
[false false false false false false false false false false false false false false false false false false false false false false false false false]
[false false false false false false false false false false false false false false false false false false false false false false false false false]
[false false false false false false false false false false false false false false false false false false false false false false false false false]
[false false false false false false false false false false false false false false false false false false false false false false false false false]
[false false false false false false false false false false false false false false false false false false false false false false false false false]
[false false false false false false false false false false false false false false false false false false false false false false false false false]
[false false false false false false false false false false false false false false false false false false false false false false false false false]
[false false false false false false false false false false false false false false false false false false false false false false false false false]

Program exited.
```

### 完整代码

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

const (
	width  = 80
	height = 15
)

type Universe [][]bool

func NewUniverse() Universe {
	u := make(Universe, height)
	for i := range u {
		u[i] = make([]bool, width)
	}
	return u
}

func (u Universe) Set(x, y int, b bool) {
	u[y][x] = b
}

func (u Universe) Seed() {
	for i := 0; i < (width * height / 4); i++ {
		u.Set(rand.Intn(width), rand.Intn(height), true)
	}
}

func (u Universe) Alive(x, y int) bool {
	x = (x + width) % width
	y = (y + height) % height
	return u[y][x]
}

func (u Universe) Neighbors(x, y int) int {
	n := 0
	for v := -1; v <= 1; v++ {
		for h := -1; h <= 1; h++ {
			if !(v == 0 && h == 0) && u.Alive(x+h, y+v) {
				n++
			}
		}
	}
	return n
}

func (u Universe) Next(x, y int) bool {
	n := u.Neighbors(x, y)
	return n == 3 || n == 2 && u.Alive(x, y)
}

func (u Universe) String() string {
	var b byte
	buf := make([]byte, 0, (width+1)*height)

	for y := 0; y < height; y++ {
		for x := 0; x < width; x++ {
			b = ' '
			if u[y][x] {
				b = '*'
			}
			buf = append(buf, b)
		}
		buf = append(buf, '\n')
	}

	return string(buf)
}

func (u Universe) Show() {
	fmt.Print("\x0c", u.String())
}

func Step(a, b Universe) {
	for y := 0; y < height; y++ {
		for x := 0; x < width; x++ {
			b.Set(x, y, a.Next(x, y))
		}
	}
}

func main() {
	a, b := NewUniverse(), NewUniverse()
	a.Seed()
	for i := 0; i < 300; i++ {
		Step(a, b)
		a.Show()
		time.Sleep(time.Second / 30)
		a, b = b, a
	}
}
```

请前往 `Go Playground` 查看：[Go Playground - The Go Programming Language](https://go.dev/play/p/mzELTdURuA6)
