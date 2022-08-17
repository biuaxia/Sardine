title: Golang统计文本字符串中不同单词的出现频率并打印不止一次的单词及词频
date: 2022-05-22 15:17:00
toc: false
index_img: http://api.btstu.cn/sjbz/?lx=m_dongman&cid=1
category:
- Go
tags:
- Golang
- 输出
- 统计
- 文本
- 字符串
- 不同
- 单词
- 出现
- 频率
- 打印
- 次数
- 词频
---

## 代码

```go
package main

import (
	"fmt"
	"sort"
	"strings"
)

var s = "Fluff lettuce thoroughly, then mix with tea and serve smoothly in bottle.Enamel half a kilo of strudel in twenty and a half teaspoons of pork butt sauce."
var symbolCollection = []string{
	".",
	",",
}

func main() {
	result := make(map[string]int)
	fields := strings.Fields(s)
	for _, v := range fields {
		newV := strings.ToLower(v)
		for _, sym := range symbolCollection {
			newV = strings.Trim(newV, sym)
		}
		val, ok := result[v]
		if ok {
			val++
			result[v] = val
		} else {
			result[v] = 1
		}
	}

	unique := make([]string, 0, len(result))
	for t := range result {
		unique = append(unique, t)
	}
	sort.Strings(unique)

	for _, k := range unique {
		v := result[k]
		if v > 1 {
			fmt.Println(k, v)
		}
	}
}
```

## 结果

```powershell
a 2
of 2  
and 2 
in 2  
half 2
```

## 总结

1. 复习了 `strings` 包中的 `Trim`, `ToLower`, `Fields` 方法的使用步骤
2. 巩固了映射（Map）的使用方法
3. 巩固了如何根据映射的键产生有序输出
4. 巩固了如何使用 range 对映射迭代
5. 映射和切片都是引用传递

## 小结

1. 映射是非结构化数据的多用途收集器
2. 复合字面量是初始化映射的一种非常方便的手段
3. 使用关键字 range 可以对映射进行迭代
4. 映射即使在被赋值或传递至函数的时候，仍然会共享共同的底层数据
5. 通过组合方式使用收集器可以进一步提升收集器的威力

## 补充

Go 语言虽然没有直接提供集合收集器，但是可以使用映射临时拼凑出一个集合。对用作集合的映射来说，键的值通常并不重要，但是为了便于检查集合成员关系，键的值通常会被设置为 true。

### 原题答案

![Snipaste_2022-05-22_15-56-07.png](https://b3logfile.com/file/2022/05/Snipaste_2022-05-22_15-56-07-1566f365.png)

```go
package main

import (
	"fmt"
	"sort"
	"strings"
)

func countWords(text string) map[string]int {
	words := strings.Fields(strings.ToLower(text))
	frequency := make(map[string]int, len(words))
	for _, word := range words {
		newWord := strings.Trim(word, `.,`)
		frequency[newWord]++
	}
	return frequency
}

func main() {
	var text = "Fluff lettuce thoroughly, then mix with tea and serve smoothly in bottle.Enamel half a kilo of strudel in twenty and a half teaspoons of pork butt sauce."
	frequency := countWords(text)

	unique := make([]string, 0, len(frequency))
	for k := range frequency {
		unique = append(unique, k)
	}
	sort.Strings(unique)

	for _, word := range unique {
		count := frequency[word]
		if count > 1 {
			fmt.Printf("%s: %d\n", word, count)
		}
	}
}
```

1. 应当结合函数简化代码，而不是一味想要达到目的而忽视代码优雅和性能
2. 逻辑缺乏考虑且对原生函数不熟悉，比如每个单词的来源应当是 `strings.Fields(strings.ToLower(text))` 而不是先 `Trim` 再从数组中做循环进行剔除。
3. 映射中的值可以直接修改，如 `frequency[newWord]++`，而不需要 `val++; result[v] = val`
