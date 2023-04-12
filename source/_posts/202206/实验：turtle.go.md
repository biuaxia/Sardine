title: 实验：turtle.go
date: 2022-06-19 20:21:00
toc: false
index_img: http://api.btstu.cn/sjbz/?lx=m_dongman&cid=5
category:
- Go
tags:
- 打印
- 反射
- 随机
- 坐标
- 方法
- 指针
- 函数
- 接收者
- 移动
- 位置
---

请编写一个可以让海龟上下左右移动的程序。

程序中的海龟需要存储一个位置(x, y)，正数坐标表示向下或向右，并通过使用方法对相应的变量实施自增和自减来实现移动。

请使用 main 函数测试这些方法并打印出海龟的最终位置。

> 提示：为了修改海龟的 x 值和 y 值，你需要将方法的接收者设置为指针。

---


标准答案：

![](https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rHX46oqRluFicHhyP06c286dBrs1SyyOSmQ/0)

结合反射和随机包的代码如下：

```
package main

import (
	"fmt"
	"math/rand"
	"reflect"
	"time"
)

type seaTurtle struct {
	x, y int
}

func (s *seaTurtle) Up() {
	s.y++
}

func (s *seaTurtle) Down() {
	s.y--
}

func (s *seaTurtle) Left() {
	s.x--
}

func (s *seaTurtle) Right() {
	s.x++
}

var positions = []string{
	"Up",
	"Down",
	"Left",
	"Right",
}

func (s *seaTurtle) randMove() {
	rand.Seed(time.Now().UnixMilli())
	time.Sleep(time.Millisecond * 5)
	randNum := rand.Intn(4)
	positionStr := positions[randNum]
	valueOf := reflect.ValueOf(s)
	method := valueOf.MethodByName(positionStr) // 方法名大写
	method.Call(nil)
	fmt.Printf("seaTurtle %s, now position: (%d, %d)\n", positionStr, s.x, s.y)
}

func main() {
	turtle := &seaTurtle{-5, 10}
	for i := 0; i < 5; i++ {
		turtle.randMove()
	}
}
```

运行结果：

```
seaTurtle Up, now position: (-5, 11)
seaTurtle Left, now position: (-6, 11)
seaTurtle Left, now position: (-7, 11)
seaTurtle Up, now position: (-7, 12)
seaTurtle Down, now position: (-7, 11)
```
