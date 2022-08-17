title: Golang经纬度DMS解决实际问题
date: 2022-05-30 20:56:00
toc: false
index_img: https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rOU6dDf5z5eHjtKaxyHSNLafiaq6Z9sVjzQ/0
category:
- Go
tags:
- 自动
- 创建
- 计算
- 字符串
- 打印
---

## 题目

请编写一个程序，使用 `gps` 结构表示全球定位系统（Global Positioning System，GPS）。这个结构应该由两个 `location` 结构和一个 `world` 结构组成，其中前者用于表示当前位置和目标位置，而后者则用于表示位置所在的世界。

请为 `location` 类型实现 `description` 方法，该方法可以返回一个包含名称、纬度和经度的字符串。至于 `world` 类型实现则参考文章：

{post cid="32"}

提到的 `func (w world) distance(p1, p2 location) float64` 方法。

请为 `gps` 类型绑定两个方法。第一个是 `distance` 方法，它用于计算当前位置和目标位置之间的距离。第二个是 `message` 方法，它会以字符串形式描述距离目的地还有多少 km。

最后，请创建 `rover` 结构并将 `gps` 嵌入其中，然后编写 `main` 函数来测试所有功能。请在测试函数中为火星初始化一个全球定位系统，并将它的当前位置设置为布莱德柏利着陆点`(-4.5895, 137.4417)`，而目标位置则设置为爱律西昂平原`(4.5, 135.9)`，然后创建 `curiosity` 探测器并使用 `message` 方法打印出相应的信息（该方法将被转发 `gps` 类型的同名方法）。

## gps.go

```go
package main

import (
	"fmt"
	"math"
)

type world struct {
	radius float64
}

type location struct {
	name      string
	lat, long float64
}

func (l location) description() string {
	return fmt.Sprintf("%v (%.1f°, %.1f°)", l.name, l.lat, l.long)
}

type gps struct {
	world       world
	current     location
	destination location
}

func (g gps) distance() float64 {
	return g.world.distance(g.current, g.destination)
}

func (g gps) message() string {
	return fmt.Sprintf("%.1f km to %v", g.distance(), g.destination.description())
}

func (w world) distance(p1, p2 location) float64 {
	s1, c1 := math.Sincos(rad(p1.lat))
	s2, c2 := math.Sincos(rad(p2.lat))
	clong := math.Cos(rad(p1.long - p2.long))
	return w.radius * math.Acos(s1*s2+c1*c2*clong)
}

func rad(deg float64) float64 {
	return deg * math.Pi / 180
}

type rover struct {
	gps
}

func main() {
	mars := world{3389.5}
	bradbury := location{"Bradbury Landing", -4.5895, 137.4417}
	elysium := location{"Elysium Planitia", 4.5, 135.9}

	gps := gps{mars, bradbury, elysium}
	curiosity := rover{gps}
	fmt.Println(curiosity.message())
}
```

## 总结

- 组合是一种将大型结构分解为小型结构并把它们合并在一起的技术。
- 嵌入使外部结构能够访问内部结构的字段。
- 当类型被嵌入结构之后，它的方法将自动实现转发。
- 如果多个嵌入类型具有同名的方法，并且这些方法被程序调用了，那么 Go 语言将提示命名冲突。
