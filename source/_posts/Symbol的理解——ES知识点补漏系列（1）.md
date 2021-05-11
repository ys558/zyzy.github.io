---
title: Symbol的理解——ES知识点补漏系列（1）
date: 2021-05-11 09:27:58
tags:
---

对于 `Symbol` 的理解，一直比较陌生，只知道他是 `ES6` 的语法规范，具体概念，可参考阮一峰老师的 [`Symbol` 文章](https://es6.ruanyifeng.com/#docs/symbol)。这里我就自己的理解写，不讲概念，写一个容易理解的demo，介绍 `Symbol` 的作用。

<!-- more -->

## 传统obj key值的痛点

一个亿万富豪，有几个儿子和女儿，还有一私生子，如果此时，我们直接这样定义这个亿万富豪：

```js
let billionaire = {
  son: ['son1', 'son2', 'son3',],
  daughter: ['daughter1', 'daughter2'],
  son: 'bastard'
}
```

在控制台我们会发现，私生子bastard 会直接覆盖掉前面的son，这显然是不合适的
![直接覆盖掉前面的属性](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.30/articles/Symbol的理解——ES知识点补漏系列（1）/01.png)

于是 `Symbol` 作为ES里原始的数据类型的作用就出现了

## 利用 `Symbol` 添加私生子, 解决痛点

正确的做法应该如下，先定义了儿子和女儿们：

```js
let billionaire = {
  son: ['son1', 'son2', 'son3',],
  daughter: ['daughter1', 'daughter2']
}
```

把私生子 `son: 'bastard'` 单独抽出来，用 `Symbol` 定义，以下有3种定义方法：

```js
const son = Symbol('son')

// 1.
billionaire[son] = 'bastard'

// 2.
let billionaire = {
  son: ['son1', 'son2', 'son3',],
  daughter: ['daughter1', 'daughter2'],
  [son] : 'bastard'
}

// 3. 
Object.defineProperty(billionaire, son, { value: 'bastard' })
```


![用Symbol生成私生子](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.30/articles/Symbol的理解——ES知识点补漏系列（1）/02.png)

至此，`Symbol` 的作用很明确了，

> **由于 JS 的 obj key值为字符串，为了避免 JS 的 obj 中属性名字的冲突而产生**

## 私生子身份神秘，普通new，. 等方法对其无效，只有亿万富翁自己知道

由于 `Symbol` 是原始类型，没有构造函数，不能使用 `new` 关键字

![Symbol不能用 new 关键字](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.30/articles/Symbol的理解——ES知识点补漏系列（1）/04.png)

也不能用 . 运算将其点出来：
![Symbol用键名选择器才能将其选出来](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.30/articles/Symbol的理解——ES知识点补漏系列（1）/05.png)

## 私生子的身份确认如何得到保障？

作为亿万富豪的私生子，只有亿万富豪自己一个人知道其身份，所以，我们用普通方法是找不到这个私生子的：

![普通方法找不到私生子](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.30/articles/Symbol的理解——ES知识点补漏系列（1）/03.png)

而私生子有独特的方法能得到确认，`Symbol` 为我们提供了几种方法   

### `Object.getOwnPropertySymbols()`

```js
const querySon = Object.getOwnPropertySymbols(billionaire)

console.log(querySon)
// [Symbol(son)]
```

## 多个私生子

亿万富豪也可能有不止一个私生子，比如和 '知己小秘' 生了一个私生子，和 '灵魂伴侣' 又生了一个私生子，此时我们可以用 `Symbol.for()` 定义:

```js
const bastard1 = Symbol.for('知己小秘')
billionaire[bastard1] = '知己小秘儿子1'
const bastard2 = Symbol.for('灵魂伴侣')
billionaire[bastard2] = '灵魂伴侣儿子1'

console.log(billionaire)
```

```bash
{
  son: [ 'son1', 'son2', 'son3' ],
  daughter: [ 'daughter1', 'daughter2' ],
  [Symbol(知己小秘)]: '知己小秘儿子1',
  [Symbol(灵魂伴侣)]: '灵魂伴侣儿子1'
}
```

## `Symbol` key值同名时，其 value 也能被覆盖掉

如果 '灵魂伴侣' 又生多了一个私生子 '灵魂伴侣儿子2' ，会出现什么情况？

```js
const bastard3 = Symbol.for('灵魂伴侣')
billionaire[bastard3] = '灵魂伴侣儿子2'

console.log(billionaire)
```

直接打印出来，会发现 value 值 '灵魂伴侣儿子2' 覆盖掉了 '灵魂伴侣儿子1'

```bash
{
  son: [ 'son1', 'son2', 'son3' ],
  daughter: [ 'daughter1', 'daughter2' ],
  [Symbol(知己小秘)]: '知己小秘儿子1',
  [Symbol(灵魂伴侣)]: '灵魂伴侣儿子2'
}
```

`bastard2` 和 `bastard3` 都是Symbol值，它们都是由同样参数的 `Symbol.for` 方法生成的key，所以实际上是仍是同一个值:   

```js
console.log(bastard2 === bastard3)
// true
```

## 查询所有儿女，包括私生子的Key值：`Reflect.ownKeys()`

会以一个数组返回所有 Key 值，包括 `Symbol Key` 
```js
console.log(Reflect.ownKeys(billionaire))
// [ 'son', 'daughter', Symbol(知己小秘), Symbol(灵魂伴侣) ]
```

## `Symbol.for()` 和 `Symbol()` 生成的Key值有何不同

`Symbol.for()` 会被登记在全局环境中供搜索，而 `Symbol()` 就不会，我们改动私生子为 `Symbol()` 生成，如：

```js
const bastard1 = Symbol.for('知己小秘')
billionaire[bastard1] = '知己小秘儿子1'
const bastard2 = Symbol('灵魂伴侣')
billionaire[bastard2] = '灵魂伴侣儿子1'

console.log(bastard1, Symbol.keyFor(bastard1))
console.log(bastard2, Symbol.keyFor(bastard2))

// Symbol(知己小秘) 知己小秘
// Symbol(灵魂伴侣) undefined
```

换句话说，`Symbol.for()` 每次生成前，会检查全局环境中是否存在该Key；`Symbol()` 则不会，每次调用会每次都生成

```js
console.log(Symbol.for('a') === Symbol.for('a'))
// true

console.log(Symbol('a') === Symbol('a'))
// false
```
