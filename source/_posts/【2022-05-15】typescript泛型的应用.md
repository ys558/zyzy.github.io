---
title: 【2022-05-15】typescript泛型的应用
date: 2022-05-15 14:09:38
tags:
  - type script
---

本文介绍泛型一些基础用法。

<!-- more -->

## 泛型

### 数组里泛型的运用

如下代码，如果传入的数值为string，则须把number改为any, 但这样一来就失去了类型检查的意义。此时泛型（generics）排上用场

```TypeScript
const last = (arr: Array<number>) => {
  return arr[arr.length - 1]
}

const l = last([1,2,3,4])
const l1 = last(['1', 2]) // 错误
```

`last` 函数须改为

```TypeScript
const last = <T>(arr: Array<T>): T => {
  return arr[arr.length - 1]
}

// or
const last = <T>(arr: T[]): T => {
  return arr[arr.length - 1]
}

// 调用的地方类型检验也可写上，当然不写也行：
const l = last([1,2,3,4])
const l1 = last<string | number>(['1', 2]) // 错误
```

2. 当然，也可定义于多个参数

```TypeScript
const makeArr = <X, Y>(x: X, y: Y): [X, Y] => {
  return [x, y]
}

const v = makeArr(5, 6)
const v1 = makeArr('a','b')
// 调用时也写上类型，也可省略
const v2 = makeArr<string, number>('a', 5)
```

以上代码还有骚操作写法：
```TypeScript
// 类似es6的默认值写法：Y泛型指定了泛型默认值，
const makeArr = <X, Y = number>(x: X, y: Y): [X, Y] => {
  return [x, y]
}

// 调用时的第二个泛型可省略：
const v3 = makeArr<string | null>(null, 5)
// 当然，和es6的默认值语法一样，虽指定了默认值为number，但要传入string也一样可以，这时则不能再指定类型，否则报错：
const v4 = makeArr(null, '5')
```

### 对象里泛型的应用

如下，指定了{}的两个属性，如果有多个属性一起传入，则会报错：

```TypeScript
const fullName = (obj: { firstName: string, lastName: string}) => {
  return {
    ...obj,
    fullName: obj.firstName + ' ' + obj.lastName
  }
}

const v4 = fullName({ firstName: 'bob', lastName: 'junior', age: 15})
```

此时可以用泛型的 `extends` 关键字，扩展了原有obj，改写如下：

```TypeScript
const fullName = <T extends { firstName: string, lastName: string }>
  (obj: T): T => {
  return {
    ...obj,
    fullName: obj.firstName + ' ' + obj.lastName
  }
}
```

### 接口中泛型的应用

如下例子，可在 interface 里直接传入一个泛型 T，定义时利用 type 类型定义传入类型

```TypeScript
interface Tab<T> {
  id: string
  position: number
  date: T
}

type NumberTab = Tab<number>
type StringTab = Tab<string>
```
