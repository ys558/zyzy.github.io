---
title: 用webpack, esbuild两种方法创建自己的js类库npm包(更新中,未完)
date: 2022-03-20 09:46:15
tags:
  - esbuild
  - webpack
---

很久没有更新, 这次趁着周末有空, 赶紧把工作中碰到的问题进行一次总结. 并记录下来.
最近碰到需要创建npm包的需求, 这里把他形成文档记录下来. 


<!-- more -->

前端的变化实在是快, 1年多前仍是babel的天下, 现在杀出个 `esbuild` , babel用js编写, 而 `esbuild`用更快的go语音编写, 他俩在webpack中均有对应的插件
关于新的打包工具 [`esbuild`](https://esbuild.github.io/), 他支持最原生的es module语法. 
在我的[这篇文章](https://zyzy.info/2021/10/28/%E3%80%902021-10-28%E3%80%91%E7%94%A8esbuil%E5%88%9B%E5%BB%BAReact%E9%A1%B9%E7%9B%AE/)里已有将其配置于CRA热更新的操作, 大家可以做参考.
写这篇文章的目的在于结合esbuild官网, 走一遍自己的配置的过程, 顺便把webpack的配置进一下对比.


## 最基础的webpack及esbuild配置

- 从最基础的配置开始, `yarn init` 生成 `package.json` 文件, 并安装 `yarn add esbuild webpack webpack-cli` 两个工具. 将包名改为mylib, 加上 `script` , 配置 `"type": "module"`为可以用esm语法

```json
{
  "name": "mylib",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "author": "ys558",
  "scripts": {
    "wp-build": "webpack",
    "es-build": "esbuild --bundle src/index.js --outfile=dist/myLib.esbuild.bundle.js --minify"
  },
  "devDependencies": {
    "esbuild": "^0.14.27",
    "webpack": "^5.70.0",
    "webpack-cli": "^4.9.2"
  },
  "type": "module"
}
```

- 新建 `scr/index.js` 和 `src/moduleA.js`, 随便写下一些东西, 用上esm语法: 

```javascript
// src/moduleA.js
const moduleA = 'module A'
export default moduleA
```

```javascript
// src/index.js
import moduleA from "./moduleA.js";

const valueA = "function A",
  valueB = "function B";

export function functionA() {
  return valueA;
}

class ClassA {
  constructor(param){
    this.param = param;
  }
}
const A = new ClassA('a');

console.log(functionA())
console.log(moduleA)
console.log('param of classA', A.param)
```

- 分别跑 `yarn run wp-build` 和 `yarn run es-build`, 可看到打包出来dist的两个文件, 

```bash
dist
    |__main.js
    |__myLib.esbuild.bundle.js
```

  无论webpack还是esbuild, 均默认生成`dist/main.js` . 所以两个命令分别运行, 会覆盖掉原来的`dist/main.js`文件
  `myLib.esbuild.bundle.js` 文件由我们指定的es script生成的, 而webpack的配置是要写在 `webpack.config.js` 里去配置. 
  
- 配置 `webpack.config.js` 文件如下:

```js
// path 和 url 均属于node的内置模块, 均是 commonJS 模块, 所以 这里里不支持直接解构导入: 
// import { fileURLToPath } from 'url'; <-- 错误
// 只能写成以下形式: 先整体导入再解构:
import url from 'url';
import path from 'path';
const { fileURLToPath } = url
const { dirname } = path

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

const webpackConfig = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'myLib.webpack.bundle.js',
  },
};

export default webpackConfig
```

再跑 `yarn run wp-build` 可发现打包出了两个文件:

```bash
dist
    |__myLib.webpack.bundle.js
    |__myLib.esbuild.bundle.js
```
