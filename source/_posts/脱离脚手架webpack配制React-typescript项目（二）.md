---
title: 脱离脚手架webpack配制React+typescript项目（二）
date: 2021-05-08 14:12:05
tags:    
    - React
    - Webpack
    - Babel
    - Type Script
cover: https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.26/articles/脱离脚手架webpack配制React+typescript项目/cover2.png
---


这节会着重介绍TS的配置，这个是对应的 [repo](https://github.com/ys558/react-without-create-react-app/tree/%40babel/preset-typescript)

<!-- more -->
## 安装
`jsx` 语法里已支持了 `TS`, 要先安装以下两个库: 

```shell
yarn add @types/react @types/react-dom
```

## React TS的三种方案

### 1. awesome-typescript-loader 方案  

- 这种不建议, 因为该插件作者已经很久不更新了, 且有些检查不全面

### 2. ts-loader + fork-ts-checker-webpack-plugin 方案  

- `webpack` 编译会通过本地的 `typescript` , 所以本地必须安装 `typescript`, 本地 `typescript` 会调用 `tsconfig.json` 文件
- 每次文件改动时, `ts-loader` 都会进行转移和类型检查, 当文件多时, 速度会变慢
- [`fork-ts-checker-webpack-plugin`](https://www.npmjs.com/package/fork-ts-checker-webpack-plugin/v/5.0.0-alpha.17), 利用该插件开辟一个单独的线程去执行类型检查, 会提高 `webpack` 的编译速度. 该插件有最低安装要求: Node.js 6.11.5，webpack 4，TypeScript 2.1 和可选的 ESLint 6（其本身要求最低 Node.js 8.10.0）

先安装上述lib
```shell
yarn add -D typescript ts-loader fork-ts-checker-webpack-plugin
```



#### `tsconfig.json` 文件配置

最主要是以下两项配置：
```json
{
    "compilerOptions": {        
      "jsx": "preserve",
      "moduleResolution": "node",
      // "module": "ESNEXT",
      "target": "es6",
      // "esModuleInterop": true  
  },
}
```
  
- `jsx` 参数有3个，分别是`'react'`，`'preserve'`，`'react-native'`   

  - `'react'` 模式下，ts 会将 tsx 编译成 jsx 后再将 jsx 编译成 js
         
  - `'preserve'` 和 `'react-native'` 模式下：TS 会将 tsx 编译成 jsx 后，不再将 jsx 编译成 js，保留 jsx 。保留 jsx 时，就需要在`webpack.config.js` - `module` -  `ts-loader` 的前面加上 `babel-loader` 去处理 jsx语法

- `"moduleResolution"` 须配置为 `"node"`， 否则会报错如下   
![引导ts去node文件夹里找 React](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.28/articles/脱离脚手架webpack配制React+typescript项目/02.png)


#### `webpack.config.js` 文件配置   
```js
const ForkTsCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin')

module.export = {
  ...
  module: {
    rules: [
      { test: /\.(jsx?)$/, 
        exclude: /node_modules/, 
        loader: 'babel-loader',
        options: {
            cacheDirectory: true,
            cacheCompression: false
        }
      },
      { test: /\.(tsx?)$/, 
        exclude: /node_modules/, 
        use: [
        'babel-loader', // 上面的 tsconfig.json 的 compilerOptions.jsx 值为 preserve，这里加上  abel-loader 转译 jsx
        { loader: 'ts-loader', options: {
          // disable type checker - we will use it in fork plugin
          transpileOnly: true,
        }}
      ]},
      ...
    ]
  },  
  plugins: [
      new HtmlWebpackPlugin({
        template: "./src/index.html",
        inject: true,
      }),
      // fork 一个进程进行检查：
      new ForkTsCheckerWebpackPlugin()
    ],
  }
```

### 3. `@babel/preset-typescript` 方案

- 比较轻的方案, 直接通过 `babel-loader` 的插件进行转译, 本地无须安装 `typescript` , 因此不会去做类型检查
- 在 `tsconfig.json` 里配置后在控制台提示语法错误, 但不做强制性检查, 如果写法不严格, 项目也不会编译不通过
- 这种方案相比起上一种方案少了类型检查, 如果还想做类型检查, 须进行额外本地安装`typescript` , 且进行额外即配置, 运行 `tsc` 的命令

```shell
yarn add -D @babel/preset-typescript
```

`babel.config.js` 里增加该插件:
```js
const presets = [
    ...
    '@babel/preset-typescript'
    ...
  ]
```

`webpack.config.js` 里相应的 `module` - `rules` 要做相应改动:
```js
rules: [
  { test: /\.(jsx?|tsx?)$/, exclude: /node_modules/, use: ['babel-loader'] },
]
```

以上配置, 就能顺利的写出 `ts` 语法了, 如:
`app.tsx`

```ts
import React, { Component, FC } from 'react'
import ReactDOM from 'react-dom'

interface aProps {
  data: string
}
const A: React.FC<aProps> = ({data}) => {
  return <div>
    {data}
  </div>
}

export default class App extends Component {
  render() {
    return <div>
        <A data='data from App' />
      </div>
  }
}

ReactDOM.render(<App />, document.getElementById('root'))
```

#### 额外多加监测tsc编译

1. 本地应额外安装 `typescript` 语法功能：
```shell
yarn add -D typescript
```

2. 直接 `tsc --watch` 监听运行

也可以 `script` 里增加一个命令专门用于监听：

```json
{
  "ts-chk": "tsc --watch"
}
```

3. `tsconfig.json`

```json
{ 
  "compilerOptions": {
  // 不生成编译文件，只做类型检查：         
  "noEmit": true,
  // js模块导入方式不做ts的检查：
  "esModuleInterop": true,
  // 使其辨认jsx语法，需重启tsc监听：
  "jsx": "preserve",
  }
}
```

其中，`"esModuleInterop": true` 是由于TS的模块导入方式和JS不同，如果去掉会报以下错误：   
![TS模块导入方式与JS的区别](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.27/articles/脱离脚手架webpack配制React+typescript项目/01.png)

按照其提示的错误 TS的导入规则，要改成这样，这显然不符合平时的习惯，
```js
import * as React from 'react'
import * as ReactDOM from 'react-dom'
```
关于模块导入方式可以参考知乎的[这篇文章](https://zhuanlan.zhihu.com/p/148081795)    

