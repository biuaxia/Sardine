title: 极速上手 TypeScript
date: 2022-02-17 14:09:14
toc: true
category:
- typescript
tags:
- 极速
- 上手
- typescript
---

## 语法基础

本文将会跳过 `基本数据类型`、`逻辑控制` 和 `枚举类型` 这三种简单的基础语法。

### 数组

见代码：

```typescript
let a = [1, 2]
console.log(`判断数组长度 ${a.length !== 0}`)
a.push(3, 4)
a.pop()
console.log(`从数组尾部添加和删除元素 ${a}`)
a.unshift(3, 4)
a.shift()
console.log(`从数组头部添加和删除元素 ${a}`)

let b = a.slice(1, 3)
console.log(`从原数组区间获得一个新数组 ${b}`)

console.log(`原数组 ${a} 删除下标 1 位置的 2 个元素后得到的数组 ${a.splice(1, 2)}， 原数组 ${a}`)
a = [4, 1, 2, 3]
console.log(`在数组 ${a} 中被删掉的元素 ${a.splice(1, 2, 11, 22)} 追加新的元素后 ${a}`)

a.unshift(22, 33)

console.log(`元素 22 在数组 ${a} 中的下标为 ${a.indexOf(22)}`)
console.log(`元素 22 在数组 ${a} 中 1下标位置起的下标为 ${a.indexOf(22, 1)}`)

console.log(`数组 ${a} 字典排序后 ${a.sort()}`)

let [a1, a2] = a
console.log(`数组 ${a} 元组 ${a1} ${a2}，即按照下标位置展开到指定变量`)

let s = "a,b,c,d,e,f,g"
console.log(`将字符串 ${s} 按照分隔符 , 分割为数组 ${s.split(',')}，长度为 ${s.split(',').length}`)

console.log(`将数组 ${a} 以分隔符 % 拼接到一起 ${a.join('%')}`)

console.log('数组如果不重新赋值最好使用 const 声明')
```

运行结果：

```text
[LOG]: "判断数组长度 true" 
[LOG]: "从数组尾部添加和删除元素 1,2,3" 
[LOG]: "从数组头部添加和删除元素 4,1,2,3" 
[LOG]: "从原数组区间获得一个新数组 1,2" 
[LOG]: "原数组 4,1,2,3 删除下标 1 位置的 2 个元素后得到的数组 1,2， 原数组 4,3" 
[LOG]: "在数组 4,1,2,3 中被删掉的元素 1,2 追加新的元素后 4,11,22,3" 
[LOG]: "元素 22 在数组 22,33,4,11,22,3 中的下标为 0" 
[LOG]: "元素 22 在数组 22,33,4,11,22,3 中 1下标位置起的下标为 4" 
[LOG]: "数组 22,33,4,11,22,3 字典排序后 11,22,22,3,33,4" 
[LOG]: "数组 11,22,22,3,33,4 元组 11 22，即按照下标位置展开到指定变量" 
[LOG]: "将字符串 a,b,c,d,e,f,g 按照分隔符 , 分割为数组 a,b,c,d,e,f,g，长度为 7" 
[LOG]: "将数组 11,22,22,3,33,4 以分隔符 % 拼接到一起 11%22%22%3%33%4" 
[LOG]: "数组如果不重新赋值最好使用 const 声明" 
```

### 对象

见代码：

```typescript
let a = {
    name: {
        first: "" as string,
        last: '',
    },
    age: 0,
    bonus: undefined as (number | undefined),
}

console.log('对象声明', a)
console.log(`对象转为 json 字符串 ${JSON.stringify(a)}`)
console.log('json 字符串转为对象 ', JSON.parse(JSON.stringify(a)))
```

运行结果：

```text
[LOG]: "对象声明",  {
  "name": {
    "first": "",
    "last": ""
  },
  "age": 0,
  "bonus": undefined
} 
[LOG]: "对象转为 json 字符串 {"name":{"first":"","last":""},"age":0}" 
[LOG]: "json 字符串转为对象 ",  {
  "name": {
    "first": "",
    "last": ""
  },
  "age": 0
}
```

### 函数

见代码:

```typescript
// 函数重载
function add(a: number, b: number): number
function add(a: string, b: number): number
function add(a: any, b: number): number {
    a = parseInt(a)
    return a + b
}
console.log(add("1", 1))
console.log(add(1, 1))

// 可选参数
function div(a: number, b: number, opt?: string) {
    console.log(`${a} ${opt || "/"} ${b} = ${a / b}`)

}
div(1, 2, "/")
div(1, 2)

// 参数默认值
function mul(a: number, b: number, opt: string = '*') {
    console.log(`${a} ${opt || 0} ${b} = ${a * b}`)
}
mul(1, 2, '*')
mul(1, 2)

// 可变参数列表
function sub(a: number, ...b: number[]) {
    let tips = `${a} - (`
    // 数组最后一个元素
    let [lastVal] = b.slice(-1)
    b.forEach(item => {
        if (item !== lastVal) {
            tips += `${item} + `
        } else {
            tips += `${item}`
        }
        a -= item
    })
    tips += `) = ${a}`
    console.log(tips)
}
sub(100, 1, 2, 3, 4, 5, 6, 7, 8, 9)
let a = [1, 2, 3, 4, 5]
sub(20, ...a)

// 对象参数
function post(param: {
    url: string,
    method: 'GET' | 'POST' | 'OPTION'
}) {
    console.log(`${param.method} ${param.url}`)
}
post({
    url: "https://qq.com",
    method: 'GET'
})

// 对象方法
const obj = {
    name: {
        first: '三',
        last: '张',
    },
    age: 10,
    salary: 8000,
    bouns: 2000,
    updateBouns(num?: number) {
        this.bouns += num || 0
    }
}
console.log(obj)
obj.updateBouns()
console.log(obj)
obj.updateBouns(1200)
console.log(obj)
````

运行结果:

```text
[LOG]: 2 
[LOG]: 2 
[LOG]: "1 / 2 = 0.5" 
[LOG]: "1 / 2 = 0.5" 
[LOG]: "1 * 2 = 2" 
[LOG]: "1 * 2 = 2" 
[LOG]: "100 - (1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9) = 55" 
[LOG]: "20 - (1 + 2 + 3 + 4 + 5) = 5" 
[LOG]: "GET https://qq.com" 
[LOG]: {
  "name": {
    "first": "三",
    "last": "张"
  },
  "age": 10,
  "salary": 8000,
  "bouns": 2000
} 
[LOG]: {
  "name": {
    "first": "三",
    "last": "张"
  },
  "age": 10,
  "salary": 8000,
  "bouns": 2000
} 
[LOG]: {
  "name": {
    "first": "三",
    "last": "张"
  },
  "age": 10,
  "salary": 8000,
  "bouns": 3200
}
```
