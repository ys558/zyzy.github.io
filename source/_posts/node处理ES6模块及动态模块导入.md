---
title: node处理ES6模块及动态模块导入
date: 2021-08-14 23:08:06
tags:
  - node
  - ES6
  - ESM
  - CJS
---

本文阐述了两个方面的内容。1. node处理ES6模块，大部分参考了阮一峰老师的[文章](https://www.ruanyifeng.com/blog/2020/08/how-nodejs-use-es6-module.html) 2. 动态模块导入

<!-- more -->

## node（CJS）处理ES6模块（ESM）


我们知道node只能通过 `require()` 加载模块， `module.exports`输出，是同步操作，这套语法规则称作`commonjs`，即`CJS`。

而ES6则是另一套语法规则，ECMA Script Module，简称`ESM`。通过`import`加载，`export`输出，是异步操作。

如果要在node里用上 `ESM` 语法规则，则得做一些必要的改变：

1. 通过改变文件名。将带有 `ESM` 语法规则的文件 $\color{CadetBlue} {后缀名改为 .mjs} $，node则可正常执行。

2. 不改变文件后缀名，$\color{CadetBlue}在 package.json 里添加 "type":"module" 。$ 相对的，如果这时想用回 `require()` 语法，则要把文件后缀名改为`cjs`。$\color{red} 但两种语法规则强烈不建议混用。$


### CommonJS 模块加载 ES6 模块

`require()` 命令不能加载 ES6 模块，会报错，node里只能使用`import()`这个方法加载，如

```js
(async () => {
  await import('./index.mjs')
})()
```

ES6 模块内部可以使用顶层 `await`命令，导致无法被同步加载。

### ES6模块加载 CommonJS 模块

`import` 可以加载CJS模块，不能单独解构加载。这是因为 ES6 模块需要支持静态代码分析，而 CommonJS 模块的输出接口是`module.exports`，是一个对象，无法被静态分析，所以只能整体加载。

```js
// 正确
import packageMain from 'commonjs-package';

// 报错
import { method } from 'commonjs-package';
```

可以改成单一输出项，写成下面这种：

```js
import packageMain from 'commonjs-package';
const { method } = packageMain;

// 或 直接export 时带 { }， node则会认为是一个整体，如：
import { method } from './other.js'

function method (user) { }
export { method }
```

而 `class` 则不受 `default` 关键字限制：
```js
import DefaultClass from './other.js'

export default class DefaultClass { }
```


### 两种格式支持模块

1. 如果原始模块是CJS，可以加一个包装层

```js
import cjsModule from './index.js'
export const foo = cjsModle.foo
```
上面代码先整体输入 CommonJS 模块，然后再根据需要输出具名接口。

你可以把这个文件的后缀名改为`.mjs`，或者将它放在一个子目录，再在这个子目录里面放一个单独的`package.json`文件，指明`{ type: "module" }

2. 另一种做法是在`package.json`文件的`exports`字段，指明两种格式模块各自的加载入口。

```js
"exports"：{ 
  "require": "./index.js"，
  "import": "./esm/wrapper.js" 
}
```

上面代码指定require()和import，加载该模块会自动切换到不一样的入口文件。

## 模块动态导入

### `import` 普通同步模块导入：
这里开始本篇第二个内容，模块动态导入。这里以ESM写法为例，用node运行
在 `package.json` 文件里加上：`"type": "module"`

下面举个简单的翻译的例子来说明，

./tranlate/en-translation.js
```js
const translations = { HI: 'hello' }
export const enTranslations = translations
```

./tranlate/sp-translation.js
```js
const translations = { HI: 'hola' }
export const spTranslations = translations
```

./locale.js
```js
import {enTranslations} from './translate/en-translation.js'
import {spTranslations} from './translate/sp-translation.js'

const user = { locale: 'sp'}
let translations
switch (user.locale) {
  case 'sp': translations = spTranslations 
    break
  default: translations = enTranslations
    break
}

console.log(translations.HI) // hola
```

运行 `node locale.js` 可以发现打印出 `hola` 的内容。

以上是我们平时import的做法，这种是同步的写法，import完后所有的文件会被加载一遍，如果文件一大的话，则会影响效率。


### `import()` 异步模块导入，按需加载，CJS写法：

对以上翻译模块的代码进行改动， 由于node里除了 `class`，函数和{} 均没有 `defalut` 关键字，所以只能具名进行解构：
./locale.js
```js
const user = { locale: 'sp'}

import(`./translate/${user.locale}-translation.js`)
  .then(({ enTranslations={}, spTranslations={} }) => {
    console.log(spTranslations.HI)
  })
```

此时可以看到控制台打印 `hola`：

```shell
yuyi@home-pc MINGW64 /e/study/code/dynamic-import-module (master)
$ npm run l

> dynamic-import-module@1.0.0 l E:\study\code\dynamic-import-module
> node locale.js

hola
```
我们刚刚改造之前的 `switch` 函数的 default 分支改写，则用 `catch`替代：如果传入为找不到的语言，则会默认输出英文的翻译

```js
// 默认只有en和sp，找不到的语言 cn：
const user = { locale: 'cn' }

  ...

  // 利用catch替代了switch函数的default分支： 
  .catch(()=> import('./translate/en-translation.js').then( module => {
    console.log(module.enTranslations.HI) // hello
  }))
```

### `import()` 的ESM写法：

将以上的代码改为在浏览器中的ESM写法，在浏览器中运行，改写如下：

locale.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
</head>
<body>
  <!-- 浏览器也得在script标签上仍要规定 type="module" -->
  <script type="module" src="./loacleEMS.js"></script>
</body>
</html>
```

./translate/ESM-en-translation.js
```js
const translations = { HI: 'hello' }
export default translations
```

./translate/ESM-sp-translation.js
```js
const translations = { HI: 'hola' }
export default translations
```

./loacleEMS.js
```js
import enTranslations from './translate/ESM-en-translation.js'
import spTranslations from './translate/ESM-sp-translation.js'

const user = { locale: 'sp'}
let translations
switch (user.locale) {
  case 'sp':
    translations = spTranslations
    break;

  default:
    translations = enTranslations
    break;
}

console.log(translations.HI) // hola
```

改写为 `import()` 方法的 ./loacleEMS.js。值得注意的是，由于ESM `export` 的关键字属性对所有函数object等都有效，能匿名到处，所以写起来更加简洁

```js
const user = { locale: 'sp'}

import(`./translate/ESM-${user.locale}-translation.js`)
  // 可直接解构匿名 default 的值：
  .then(({ default: translations }) => 
    console.log(translations.HI) // hola
  )
```

这种情况下我们怎么改写 `switch-default` 分支呢？答案是利用 `.catch()`，并且我们把 `const user = { locale: 'sp'}` 改写为 `const user = { locale: 'cn'}`，一个文件里没有的语言

```js
const user = { locale: 'cn'}

import(`./translate/ESM-${user.locale}-translation.js`)
// 将.then() 放置于.catch() 里的回调 import() 之后执行，
// 这样，控制台报错找不到 'cn'，但程序仍会往下走：
.catch(()=> import('./translate/ESM-en-translation.js')
  .then( ({ default: translations}) => {
    console.log(translations.HI) // hello
    })
)
```

