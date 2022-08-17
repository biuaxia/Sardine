title: Golang实现经纬度DMS格式录入和十进制格式打印功能
date: 2022-05-30 10:53:00
toc: false
index_img: https://puep.qpic.cn/coral/Q3auHgzwzM4fgQ41VTF2rPjKPe2MMQJnppZjNiaj9zdRZoYNibBRo1Xw/0
category:
- Go
tags:
- 格式
- 打印
---

## 题目

为下方表格中的每个位置声明相应的 `location` 结构，并使用十进制格式打印出这些位置。

## 表格

| 探测器或着陆器 | 着陆点           | 纬度            | 经度              |
| :------------ | :-------------- | :-------------- | :--------------- |
| 勇气号         | 哥伦比亚纪念站   | 南纬 14°34'6.2" | 东经 175°28'21.5" |
| 机遇号         | 挑战者纪念站     | 南纬 1°56'46.3" | 东经 354°28'24.2" |
| 好奇号         | 布莱德柏利着陆地 | 南纬 4°35'22.2" | 东经 137°26'30.1" |
| 洞察号         | 埃律西昂平原     | 北纬 4°30'0.0"  | 东经 135°54'0"    |

## 代码

```go
package main

import (
	"fmt"
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

func main() {
	// 南纬14°34'6.2"，东经175°28'21.5"
	spirit := newLocation(coordinate{d: 14, m: 34, s: 6.2, h: 'S'}, coordinate{d: 175, m: 28, s: 21.5, h: 'E'})
	// 南纬1°56'46.3"，东经354°28'24.2"
	opportunity := newLocation(coordinate{d: 1, m: 56, s: 46.3, h: 'S'}, coordinate{d: 354, m: 28, s: 24.2, h: 'E'})
	// 南纬4°35'22.2"，东经137°26'30.1"
	curiosity := newLocation(coordinate{d: 4, m: 35, s: 22.2, h: 'S'}, coordinate{d: 137, m: 26, s: 30.1, h: 'E'})
	// 北纬4°30'0.0"，东经135°54'0"
	insight := newLocation(coordinate{d: 4, m: 30, s: 0.0, h: 'N'}, coordinate{d: 135, m: 54, s: 0, h: 'E'})

	fmt.Printf("spirit: %+v\n", spirit)
	fmt.Printf("opportunity: %+v\n", opportunity)
	fmt.Printf("curiosity: %+v\n", curiosity)
	fmt.Printf("insight: %+v\n", insight)
}
```

## 结果

```
spirit: {lat:-14.568388888888888 long:175.4726388888889}
opportunity: {lat:-1.9461944444444446 long:354.47338888888885}
curiosity: {lat:-4.5895 long:137.44169444444444}
insight: {lat:4.5 long:135.9}
```
