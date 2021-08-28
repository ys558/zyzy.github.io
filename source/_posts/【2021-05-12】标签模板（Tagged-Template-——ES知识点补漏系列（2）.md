---
title: 标签模板（Tagged Template)——ES知识点补漏系列（2）
date: 2021-05-12 09:18:46
tags:
  - ES
cover: https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.31/articles/标签模板（Tagged Template)——ES知识点补漏系列（2）/cover.jpg
---

字符串模板相信大家用的挺多了，这里不赘述，今天解锁字符串模板的另一个功能，标签模板（Tagged Template）

<!-- more -->

## 可作为传参用

标签模板可用于传参，就行普通函数一样接收参数，请看示例代码：

```js
const custom = (strings, ...placeholder) => {
    console.log(strings)
    console.log(placeholder)
}

const firstName = '呵呵'
const hobby = '点赞'

// 调用时改为标签模板传参，不用普通函数的()传参
custom`my name is ${firstName}, my hobby is ${hobby}`
```
 
直接在控制台执行后，会发现结果如下：

```bash
[ 'my name is ', ', my hobby is ', '' ]
[ '呵呵', '点赞' ]
```

我们发现标签模板实际上是一种特殊函数，接收两个参数    

> @第一个参数 ： 除了${}以外的部分字符串部分
> @第二个参数 ： ${}里面的部分，可以用解构...接收所有${}的参数，否则则按形参一个个按顺序传入

### 实际作用

下面，我们就这种特殊函数的实际作用举2个例子

#### 给特定文本添加样式

标签模板被发明出来的最主要目的就是拼接HTML字符串的，下面我举2个例子：

1. 用于一段文字中添加HTML样式：
```js
const a = (arr, ...placeholder) => arr.reduce((prev, cur, i) => prev + `<span class="addColor">${placeholder[i-1]}</span>`+ cur)

console.log(a`my name is ${firstName}, my hobby is ${hobby}`)

// my name is <span class="addColor">呵呵</span>, my hobby is <span class="addColor">点赞</span>
```

2. 用于校验HTML里的字符串，防止用户进行XSS攻击

```js
function filterMsg(data) {
  let ret = data[0]
  // 直接利用 function 的 arguments 参数进行拼接：
  for (let i = 1; i < arguments.length; i++) {
    const arg = String(arguments[i])
    ret += arg.replace(/&/g, "&amp;")
            .replace(/</g, "&lt;")
            .replace(/>/g, "&gt;")
    ret += data[i] 
  }
  return ret
}

// 用户输入：
const userInput = '<script>alert("123")</script>'

// 调用处：
const safeMsg = filterMsg`<div>${userInput} has been send</div>`

console.log(safeMsg)
// 输出：
// <div>&lt;script&gt;alert("123")&lt;/script&gt; has been send</div>
```
