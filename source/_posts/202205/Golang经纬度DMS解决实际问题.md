title: Golang经纬度DMS解决实际问题
date: 2022-05-30 10:53:00
toc: false
index_img: https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rGpUMPpyPrsMm6TJcdtuTd00Qp3a7jkPEg/0
category:
- Go
tags:
- 计算
- 格式
---

## 题目

编写一个程序，他可以计算出表 `火星上的着陆点` 中每一对着陆点之间的距离，并回答以下问题：

- 哪两个着陆点之间的距离最近？
- 哪两个着陆点之间的距离最远？

之后，请基于表 `各行星的测定半径` 定义出新的世界，并执行以下计算。

- 计算出英国伦敦（北纬 51°31'，西经 0°08'）至法国巴黎（北纬 48°51'，东经 2°21'）之间的距离。
- 计算出你所在的城市与首都之间的距离。
- 计算出火星上夏普山（南纬 5°4'48"，东经 137°51'）和奥林帕斯山（北纬 18°39'，东经 226°12'）之间的距离。

## 火星上的着陆点

| 探测器或着陆器 | 着陆点           | 纬度            | 经度              |
| :------------ | :-------------- | :-------------- | :--------------- |
| 勇气号         | 哥伦比亚纪念站   | 南纬 14°34'6.2" | 东经 175°28'21.5" |
| 机遇号         | 挑战者纪念站     | 南纬 1°56'46.3" | 东经 354°28'24.2" |
| 好奇号         | 布莱德柏利着陆地 | 南纬 4°35'22.2" | 东经 137°26'30.1" |
| 洞察号         | 埃律西昂平原     | 北纬 4°30'0.0"  | 东经 135°54'0"    |

## 各行星的测定半径

| 行星 | 半径/km | 行星  | 半径/km |
| :--- | :----- | :---- | :----- |
| 水星 | 2439.7 | 木星   | 69911  |
| 金星 | 6051.8 | 土星   | 58232  |
| 地球 | 6371.0 | 天王星 | 25362  |
| 火星 | 3389.5 | 海王星 | 24622  |

## 代码

```go
package main

import (
	"fmt"
	"math"
	"strings"
)

type location struct {
	lat, long float64
}

func newLocation(lat, long coordinate) location {
	return location{
		lat:  lat.decimal(),
		long: long.decimal(),
	}
}

type coordinate struct {
	d, m, s float64
	h       rune
}

func (c coordinate) decimal() float64 {
	sign := 1.0
	switch strings.ToLower(string(c.h)) {
	case "s", "w":
		sign = -1
	}
	return sign * (c.d + c.m/60 + c.s/3600)
}

type world struct {
	radius float64
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

var (
	mars  = world{radius: 3389.5}
	earth = world{radius: 6371}
)

func main() {
	// 南纬14°34'6.2"，东经175°28'21.5"
	spirit := newLocation(coordinate{d: 14, m: 34, s: 6.2, h: 'S'}, coordinate{d: 175, m: 28, s: 21.5, h: 'E'})
	// 南纬1°56'46.3"，东经354°28'24.2"
	opportunity := newLocation(coordinate{d: 1, m: 56, s: 46.3, h: 'S'}, coordinate{d: 354, m: 28, s: 24.2, h: 'E'})
	// 南纬4°35'22.2"，东经137°26'30.1"
	curiosity := newLocation(coordinate{d: 4, m: 35, s: 22.2, h: 'S'}, coordinate{d: 137, m: 26, s: 30.1, h: 'E'})
	// 北纬4°30'0.0"，东经135°54'0"
	insight := newLocation(coordinate{d: 4, m: 30, s: 0.0, h: 'N'}, coordinate{d: 135, m: 54, s: 0, h: 'E'})

	fmt.Printf("Spirit to Opportunity %.2f km\n", mars.distance(spirit, opportunity))
	fmt.Printf("Spirit to Curiosity %.2f km\n", mars.distance(spirit, curiosity))
	fmt.Printf("Spirit to Insight %.2f km\n", mars.distance(spirit, insight))

	fmt.Printf("Opportunity to Curiosity %.2f km\n", mars.distance(opportunity, curiosity))
	fmt.Printf("Opportunity to Insight %.2f km\n", mars.distance(opportunity, insight))

	fmt.Printf("Curiosity to Insight %.2f km\n", mars.distance(curiosity, insight))

	london := newLocation(coordinate{d: 51, m: 30, s: 0, h: 'N'}, coordinate{d: 0, m: 8, s: 0, h: 'W'})
	paris := newLocation(coordinate{d: 48, m: 51, s: 0, h: 'N'}, coordinate{d: 2, m: 21, s: 0, h: 'E'})
	fmt.Printf("Lindon to Paris %.2f km\n", earth.distance(london, paris))

	// Ref: https://map.jiqrxx.com/jingweidu/
	chengdu := newLocation(coordinate{d: 30, m: 39, s: 3.24, h: 'N'}, coordinate{d: 104, m: 04, s: 35.16, h: 'E'})
	home := newLocation(coordinate{d: 30, m: 32, s: 51.02, h: 'N'}, coordinate{d: 104, m: 06, s: 15.70, h: 'E'})
	fmt.Printf("Chengdu to Home %.2f km\n", earth.distance(chengdu, home))

	mountSharp := newLocation(coordinate{5, 4, 48, 'S'}, coordinate{137, 51, 0, 'E'})
	olympusMons := newLocation(coordinate{18, 39, 0, 'N'}, coordinate{226, 12, 0, 'E'})
	fmt.Printf("Mount Sharp to Olympus Mons %.2f km\n", mars.distance(mountSharp, olympusMons))
}
```

## 结果

```
Spirit to Opportunity 9669.71 km
Spirit to Curiosity 2291.55 km
Spirit to Insight 2580.13 km
Opportunity to Curiosity 8425.63 km
Opportunity to Insight 8365.45 km
Curiosity to Insight 545.38 km
Lindon to Paris 343.62 km
Chengdu to Home 11.80 km
Mount Sharp to Olympus Mons 5328.08 km
```

## 总结

- Go 语言通过组合结构和方法，在没有引入任何新特性的情况下在很大程度上实现了传统语言的面向对象特性。
- Go 语言没有为构造函数提供特殊的语言特性，构造函数和其他函数一样只是普通的函数。
- Go 语言选择了名称格式为 `newType` 或者 `NewType` 的函数用于构造指定类型的值，至于首字母的大小写则有函数是否需要导出以供其它包使用来决定。
- DMS格式（degrees/minutes/seconds，度/分/秒）分和秒表示的是位置而不是时间，其中每 60 秒（"）为一分，每 60 分（'）为一度。例如，布莱德柏利着陆点的位置用 DMS 格式表示就是 `南纬 4°35'22.2"，东经 137°26'30.1"`
