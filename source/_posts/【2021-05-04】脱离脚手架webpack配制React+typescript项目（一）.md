---
title: 脱离脚手架webpack配制React+typescript项目（一）
date: 2021-05-04 11:11:58
tags:
    - React
    - Webpack
    - Babel
cover: https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.26/articles/脱离脚手架webpack配制React+typescript项目/cover2.png
---

![](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.26/articles/脱离脚手架webpack配制React+typescript项目/cover2.png)

如题。以前配过，但没直接支持ts的，这次复习一遍记录下来，webpack 这东西，在项目开始时整一次后经常就很少碰了，更要动手记录下来。

<!-- more -->

## 初始化项目   
```shell
yarn init -y
```
## 基础安装

配置 `webpack`

```shell
yarn add webpack webpack-cli@3* -D
```

`babel` 用于处理ES6+的代码转换
```shell
yarn add -D @babel/core @babel/preset-env babel-loader
```

配置 `less` 、 `sass` 或 `postcss` 等样式预处理语言，这里三个都给他配上：   
`autoprefixer` 可以自动在样式中添加浏览器厂商前缀，避免手动处理样式兼容问题   
```shell
# less
yarn add -D css-loader less less-loader style-loader
# sass

# postcss

```

配置处理 HTML 插件 
```shell
yarn add -D html-webpack-plugin
```

配置热更新插件
```shell
yarn add -D webpack-dev-server 
```

新建 `src/app.js`， `webpack.config.js`， `src/index.html`   
```shell
mkdir src && touch src/app.js && touch webpack.config.js 
```

## 支持react的配置

基础：
```shell
yarn add react-dom react
```

支持各种语法：   
@babel/preset-react —— 支持 `jsx` 语法   
@babel/plugin-proposal-class-properties —— 支持 `class` 语法   
@babel/plugin-proposal-decorators —— 支持装饰器语法   


```shell
yarn add -D @babel/preset-react @babel/plugin-proposal-class-properties @babel/plugin-proposal-decorators
```


## babel 配置

原来的 `.babelrc` 仅适用于简单单个包的静态配置，

`babel7+` 版本采用 `babel.config.js`，可以静态编译 `node_modules`  

> 我们建议使用babel.config.js格式。babel本身正在使用它。   

对于大型项目，babel官方还是建议采用 `babel.config.js` 文件进行配置，因为 `.babelrc` 是从每一个文件向上查找配置的，`babel.config.js`则不会     

```shell
touch babel.config.js
```

编写如下：   
```js
module.exports = function (api) {
  api.cache(true)

  const presets = [
    [ '@babel/preset-env', { modules: false } ],
    '@babel/preset-react',
  ]

  const plugins = [
    ['@babel/plugin-proposal-decorators', { legacy: true }],
    ['@babel/plugin-proposal-class-properties', { loose: true }]
  ]
  return { presets, plugins }
}
```

[官方文档](https://babel.docschina.org/docs/en/config-files/#project-wide-configuration)
## webpack.config.js
基础配制配置如下：

```js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  entry: './src/app.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
    library: 'ReactByWebpack',
    libraryTarget: 'umd'
  },
  module: {
    rules: [
      { test: /\.(jsx?)$/, exclude: /node_modules/, use: ['babel-loader'] },
      { test: /\.less$/, exclude: /node_modules/, use: ['style-loader', 'css-loader', 'less-loader']}
    ]
  },
 plugins: [
    new HtmlWebpackPlugin({
      template: "./src/index.html",
    })
  ],
  devServer: {
    contentBase: './dist',
    port: 8888,
    compress: true
  }
}
```

## 错误解决 ：`webpack`4 和 `webpack-cli`4 发生冲突

```shell
yarn run v1.22.10
$ webpack-dev-server
internal/modules/cjs/loader.js:883
  throw err;
  ^

Error: Cannot find module 'webpack-cli/bin/config-yargs'
Require stack:
- E:\study\code\tech-blog-code\2021\11-webpack-react-ts-config\node_modules\webpack-dev-server\bin\webpack-dev-server.js
    at Function.Module._resolveFilename (internal/modules/cjs/loader.js:880:15)
    at Function.Module._load (internal/modules/cjs/loader.js:725:27)
    at Module.require (internal/modules/cjs/loader.js:952:19)
    at require (internal/modules/cjs/helpers.js:88:18)
    at Object.<anonymous> (E:\study\code\tech-blog-code\2021\11-webpack-react-ts-config\node_modules\webpack-dev-server\bin\webpack-dev-server.js:65:1)
    at Module._compile (internal/modules/cjs/loader.js:1063:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1092:10)
    at Module.load (internal/modules/cjs/loader.js:928:32)
    at Function.Module._load (internal/modules/cjs/loader.js:769:14)
    at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:72:12) {
  code: 'MODULE_NOT_FOUND',
  requireStack: [
    'E:\\study\\code\\tech-blog-code\\2021\\11-webpack-react-ts-config\\node_modules\\webpack-dev-server\\bin\\webpack-dev-server.js'     
  ]
}
error Command failed with exit code 1.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
```

解决办法：   
从 [webpack repo issue #1948](https://github.com/webpack/webpack-cli/issues/1948) 里可以找到答案: 

> If you upgrade webpack to 5. *, and webpack cli to 4. *, an error > will be reported:
> 
> Error: Cannot find module 'webpack-cli/bin/config-yargs'
> 
> Temporary solution:
> Back off webpack cli to version 3. * for example:
> 
> "webpack-cli": "^ 3.3.12"

须要把 `webpack-cli` 降回3及以下版本

```shell
yarn remove webpack-cli
yarn add -D webpack-cli@3
```

## 配置入口文件

```shell
mkdir src && touch src/app.js && touch src/index.html
```

### `app.js`
```js
import React, { Component } from 'react'
import ReactDOM from 'react-dom'

const A = () => 123
export default class App extends Component {
  render() {
    return <div>
        <A/>
      </div>
  }
}

ReactDOM.render(<App />, document.getElementById('root'))
```

### `index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Document</title>
</head>
<body>
  <div id='root'></div>
</body>
</html>
```

## 运行

添加 `script` 如下
```json
"scripts": {
  "dev": "webpack-dev-server",
  "build": "webpack --mode=production"
},
```
至此，普通的 `React` 项目已配置成功，可以尽情写 `jsx` 了，下一节会继续加上TS配置
