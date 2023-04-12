title: Map-Reduce-Filter 模式处理集合元素
date: 2022-07-04 10:37:00
toc: false
index_img: http://api.btstu.cn/sjbz/?lx=m_dongman&cid=9
category:
- Go
tags:
- 用户
- 遍历
- 计算
- 统计
- 字符串
- 函数
---

日常开发过程中，要处理数组、切片、字典等集合类型，常规做法都是循环迭代进行处理。比如将一个字典类型用户切片中的所有年龄属性值提取出来，然后求和，常规实现是通过循环遍历所有切片，然后从用户字典键值对中提取出年龄字段值，再依次进行累加。

针对简单的单个场景，这么实现没什么问题，但这是典型的面向过程思维，而且代码几乎没有什么复用性可言：每次处理类似的问题都要编写同样的代码模板，比如计算其他字段值，或者修改类型转化逻辑，都要重新编写实现代码。

在函数式编程中，我们可以通过 `Map-Reduce` 技术让这个功能实现变得更优雅，代码复用性更好。

Map-Reduce 并不是一个整体，而是要分两步实现：Map 和 Reduce，Map-Reduce 模型：先将字典类型切片转化为一个字符串类型切片（Map，字面意思就是一一映射），再将转化后的切片元素转化为整型后累加起来（Reduce，字面意思就是将多个集合元素通过迭代处理减少为一个）。

有的时候，为了让 Map-Reduce 代码更加健壮（排除无效的字段值），或者只对指定范围的数据进行统计计算，还可以在 Map-Reduce 基础上引入 Filter（过滤器），对集合元素进行过滤。

```
package main

import (
  "fmt"
  "strconv"
  "time"
)

func main() {
  var users = []map[string]string{
    {
      "name": "xmp",
      "age":  "11",
    },
    {
      "name": "wlm",
      "age":  "24",
    },
    {
      "name": "wjf",
      "age":  "30",
    },
    {
      "name": "lm",
      "age":  "50",
    },
  }
  fmt.Println("总年龄：", ageSum(users))

  fmt.Println("总年龄：", handlerMapReduce(users))

  fmt.Println("总年龄（过滤40以上和20岁以下）：", handlerValidUsers(users))
}

func ageSum(users []map[string]string) int {
  startTime := time.Now()
  var result int
  for _, user := range users {
    age, _ := strconv.Atoi(user["age"])
    result += age
  }
  fmt.Println("\tageSum 总耗时：", time.Since(startTime))
  return result
}

func handlerValidUsers(users []map[string]string) int {
  validUsers := filterMap(users, func(item map[string]string) bool {
    ageStr, ok := item["age"]
    if !ok {
      return false
    }
    age, err := strconv.Atoi(ageStr)
    if err != nil {
      return false
    }
    if age > 40 || age < 20 {
      return false
    }
    return true
  })
  return handlerMapReduce(validUsers)
}

func filterMap(items []map[string]string, f func(map[string]string) bool) []map[string]string {
  startTime := time.Now()
  var result []map[string]string
  for _, item := range items {
    if f(item) {
      result = append(result, item)
    }
  }
  fmt.Println("\tfilterMap 总耗时：", time.Since(startTime))
  return result
}

func handlerMapReduce(users []map[string]string) int {
  startTime := time.Now()
  items := mapToString(users, func(user map[string]string) string {
    return user["age"]
  })
  result := fieldSum(items, func(str string) int {
    val, _ := strconv.Atoi(str)
    return val
  })
  fmt.Println("\thandlerMapReduce 总耗时：", time.Since(startTime))
  return result
}

func mapToString(items []map[string]string,
  f func(map[string]string) string) []string {
  newSlice := make([]string, len(items))
  for _, item := range items {
    newSlice = append(newSlice, f(item))
  }
  return newSlice
}

func fieldSum(items []string, f func(string) int) int {
  var result int
  for _, item := range items {
    result += f(item)
  }
  return result
}
```

不过分开调用 Map、Reduce、Filter 函数不太优雅，我们可以通过装饰器模式将它们层层嵌套起来，或者通过管道模式（Pipeline）让这个调用逻辑可读性更好，更优雅。
