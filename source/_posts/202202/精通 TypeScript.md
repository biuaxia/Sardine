title: 精通 TypeScript
date: 2022-02-17 16:02:14
toc: true
category:
- typescript
tags:
- 精通
- typescript
---

## 一等公民与高阶函数

一等公民：变量类型、值(literal)、对象字段、函数参数、函数返回都可以是函数。

高阶函数：函数返回和函数参数都是高阶函数，包裹一层算一阶。

见代码：

```typescript
let a = [1, 2, 3, 4, 11, 22, 33, 44, 111, 222, 333, 444]
a.sort()
console.log(a)

const comp = createCompare({
    smallerFirst: true
})

function createCompare(p: { smallerFirst: boolean }) {
    if (p.smallerFirst) {
        return (a: number, b: number) => a - b
    } else {
        return (a: number, b: number) => b - a
    }
}

function logCompare(comp: (a: number, b: number) => number) {
    return (a: number, b: number) => {
        console.log(`compareing ${a} ${b}`)
        return comp(a, b)
    }
}

a.sort(logCompare(comp))
console.log(a)
```

运行结果：

```text
[LOG]: [1, 11, 111, 2, 22, 222, 3, 33, 333, 4, 44, 444] 
[LOG]: "compareing 11 1" 
[LOG]: "compareing 111 11" 
[LOG]: "compareing 2 111" 
[LOG]: "compareing 2 11" 
[LOG]: "compareing 2 1" 
[LOG]: "compareing 22 11" 
[LOG]: "compareing 22 111" 
[LOG]: "compareing 222 11" 
[LOG]: "compareing 222 111" 
[LOG]: "compareing 3 22" 
[LOG]: "compareing 3 2" 
[LOG]: "compareing 3 11" 
[LOG]: "compareing 33 11" 
[LOG]: "compareing 33 111" 
[LOG]: "compareing 33 22" 
[LOG]: "compareing 333 22" 
[LOG]: "compareing 333 111" 
[LOG]: "compareing 333 222" 
[LOG]: "compareing 4 22" 
[LOG]: "compareing 4 3" 
[LOG]: "compareing 4 11" 
[LOG]: "compareing 44 22" 
[LOG]: "compareing 44 222" 
[LOG]: "compareing 44 111" 
[LOG]: "compareing 44 33" 
[LOG]: "compareing 444 22" 
[LOG]: "compareing 444 111" 
[LOG]: "compareing 444 333" 
[LOG]: [1, 2, 3, 4, 11, 22, 33, 44, 111, 222, 333, 444]
```

其中变量、函数和方法的类型分别为：

```text
let a: number[]

const comp: (a: number, b: number) => number

function createCompare(p: { smallerFirst: boolean }): (a: number, b: number) => number

function logCompare(comp: (a: number, b: number) => number): (a: number, b: number) => number

(method) Array<number>.sort(compareFn?: ((a: number, b: number) => number) | undefined): number[]
```

## 闭包

见代码：

```typescript
const a = [1, 2, 3, 4, 11, 22, 33, 44, 111, 222, 333, 444, 5, 55, 555, 6, 66, 666]

// config
const GOOD_FACTOR = 2
// config end

function isGoodNum(goodFactor: number, v: number) {
    return v % goodFactor == 0
}

function filterArray(a: number[], f: (v: number) => boolean) {
    return a.filter(f)
}

function partiallyApply(f: (a: number, b: number) => boolean, a: number) {
    return (b: number) => f(a, b)
}

console.log(filterArray(a, (v) => isGoodNum(GOOD_FACTOR, v)))
console.log(filterArray(a, partiallyApply(isGoodNum, 3)))
```

运行结果：

```text
[LOG]: [2, 4, 22, 44, 222, 444, 6, 66, 666] 
[LOG]: [3, 33, 111, 222, 333, 444, 555, 6, 66, 666] 
```
