---
title: Electron配置React或Vue实践（未完，Vue部分待更新）
date: 2021-06-08 09:28:59
tags:
    - Electron
    - React
cover: https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.37/articles/Electron配置React或Vue实践/cover2.webp
---
本篇介绍在 `Electron` 中配置 `React` 或者 `Vue` 框架开发，毕竟现在多是用框架开发当道，`Electron` 也不可能配置框架在里面，所以只好自己动手用 `Webpack` 来搞
<!-- more -->

## Electron 基础配置   

```bash
mkdir <project name> && cd <project name>
npm init -y && npm install electron@11.1.1
```

国内直接安装 `electron` 会比较慢，可以配置淘宝源安装，直接输入：

```bash
npm set registry http://registry.npm.taobao.org
```

但下载完成后记得改回来`npm`官方的源，要不然会影响自己 `npm` 模块的发布：

```bash
npm config set registry https://registry.npmjs.org
```

可用 `npm get registry` 查看本地所属的源

### 项目基础架构

`package.json` 修改改如下：
```json
{
  "name": "electron-template",
  "version": "0.0.1",
  "description": "Electron + React 配置项目",
  "author": "余翼",
  "main": "./app/main/electron.js",
  "scripts": {
    "start:main": "electron ./app/main/electron.js"
  },
  "dependencies": {
    "electron": "^13.1.1"
  }
}
```

创建目录结构如下：
```bash
electron-template
  |__app
      |__main # 主进程文件夹，即跑起 Electron 的进程，只有一个， 控制浏览器窗口的操作等
         |__electron.js  
         |__index.html
      |__renderer # 渲染进程文件夹，即渲染页面的进程，前端的界面的js文件,可以有多个文件
```

让我们修改以下两个文件：

用于启动Electron的窗口：`app\main\electron.js`   

```js
const path = require('path');
const { app, BrowserWindow } = require('electron');

const createWindow = () => {
  const mainWindow = new BrowserWindow({
    width: 1000,
    height: 600,
    // 集成node
    webPreferences: {
      nodeIntegration: true, 
    },
  })
  mainWindow.loadFile('index.html')
}

app.whenReady().then(() => {
  createWindow()
  app.on('activate',  () => if (BrowserWindow.getAllWindows().length === 0) createWindow()
)})
```

`app\main\index.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Electron template</title>
</head>
<body>
  <h1>hello world</h1>
</body>
</html>
```

运行 `npm run start:main` 可以看到界面：

![Electron 界面](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.37/articles/Electron配置React或Vue实践/00.png)

## Electron + `React`

### 配置 React

```bash
npm install react
npm install react-router react-router-dom react-dom
```

### 配置 `babel`
`babel` 用来编译es6及jsx代码：
```bash
# 核心 babel 插件：
npm install -S @babel/polyfill
npm install -D @babel/core @babel/cli
# 处理 jsx
npm install -D @babel/preset-env @babel/preset-react @babel/preset-typescript 
# 缩小 @babel/polyfill 引入时的的库，进行按需引入
npm install @babel/plugin-transform-runtime --save --dev
# 将ES modules转换为 CommonJS
npm install @babel/plugin-transform-modules-commonjs --save --dev
```

编写 `babel.config.js` ，[babel官网](https://babeljs.io/docs/en/configuration)推荐的这种写法替换以前的 `.babelrc`，`api.cache(true);`据官网的说法，可缓存传进来的api，效率更高
```js
module.exports = function (api) {
  api.cache(true);

  const presets = [
    '@babel/preset-env', // 允许使用最新的JS语法，而无须考虑环境的影响
    '@babel/preset-react',
    '@babel/preset-typescript',
  ]
  const plugins = [
    '@babel/plugin-transform-runtime',
    [
      '@babel/plugin-transform-modules-commonjs',
      {
        allowTopLevelThis: true,
        loose: true,
        lazy: true
      }
    ]
  ]

  return {
    presets,
    plugins
  }
}
```

### 配置 webpack

！注意，截至到我写该文为止，安装最新的webpack5和webpack-cli4，并不兼容，运行时会报很多莫名其妙的错误，所以必须指定webpack4和webpack-cli3的版本，其他版本我也试过了，只有这两个版本相容较为稳定

```bash
# 基础
npm i -D webpack@4 webpack-cli@3
# 热更新
npm i -D webpack-dev-server
```

我们会配置3个webpack文件，分别是

- `webpack.base.js` -- 基础配置
- `webpack.render.js` -- 主进程配置
- `webpack.main.js` -- 渲染进程配置

所以安装这款插件，用于将下面两个个文件引入 `webpack.base.js` 中，减少webpack配置的代码:

```bash
npm i -D webpack-merge
```

安装各种loader，plugin

```bash
npm i -D html-webpack-plugin@4 # 用于读取入口HTML文件
npm i -D clean-webpack-plugin # 主进程只编译每次打包好的文件，要这个插件可以每次自动清除上次留下来的文件
npm i babel-loader 
```

配置 `cross-env` 插件，用于执行不同环境的脚本：
```bash
npm i cross-env
```

### webpack配置文件

#### `webpack.base.js`

```js
const path = require('path')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  resolve: {
    extensions: ['.js', '.jsx', '.ts', '.tsx'],
    alias: {
      // 路径别名，可将 import A from '../../../../../A'这种导入路径，变成 import A from '@src/A'
      '@src': path.join(__dirname, '../', 'app/renderer')
    }
  },
  module: {
    rules: [
      { test: /\.(jsx?|tsx?)$/, exclude: /node_modules/, use: { loader: 'babel-loader',}},
    ]
  },
  plugins: [ new CleanWebpackPlugin(), ],
}
```

#### `webpack.main.dev.js`

```js
const path = require('path')
const webpack = require('webpack')
const baseConfig = require('./webpack.base.js')
const webpackMerge = require('webpack-merge')

const mainConfig = {
  entry: path.resolve(__dirname, '../app/main/electron.js'),
  // 构建出不同运行环境的代码
  target: 'electron-main',
  output: {
    filename: 'electron.js',
    path: path.resolve(__dirname, '../dist'),
  },
  devtool: 'inline-source-map',
  mode: 'development',
  plugins: [
    // 👇 根据启动命令的 node_env，指定构建变量
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': '"development"'
    })
  ],
}

module.exports = webpackMerge.merge(baseConfig, mainConfig)
```

#### `webpack.render.dev.js`

```js
const path = require('path')
const webpackMerge = require('webpack-merge')
const baseConfig = require('./webpack.base.js')
const HtmlWebPackPlugin = require('html-webpack-plugin')

const devConfig = {
  mode: 'development',
  entry: {
    // app.jsx 入口文件
    index: path.resolve(__dirname, '../app/renderer/App.jsx'),
  },
  output: {
    filename: '[name].[hash].js',
    path: path.resolve(__dirname, '../dist'),
  },
  target: 'electron-renderer',
  devtool: 'inline-source-map',
  devServer: {
    contentBase: path.join(__dirname, '../dist'),
    compress: true,
    host: '127.0.0.1',
    port: 7001,
    hot: true,
  },
  plugins: [
    new HtmlWebPackPlugin({
      // 以此为模板，自动生成HTML
      template: path.resolve(__dirname, '../app/renderer/index.html'),
      filename: path.resolve(__dirname, '../dist/index.html'),
      chunks: ['index'],
    })
  ]
}

module.exports = webpackMerge.merge(baseConfig, devConfig)
```

### 重组项目结构

由于react是在渲染进程中执行，所以，我们将入口文件 `index.html` 移动到 `.app/render/` 文件夹， 并创建 `app.jsx`文件作为react的入口文件
```bash
mv ./app/main/index.html ./app/renderer/
touch ./app/renderer/App.jsx
```

`index.html` 更改如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Electron Platform</title>
  <style>
    * { margin: 0; }
  </style>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

`App.jsx`
```js
import React from 'react'
import ReactDOM from 'react-dom'
import { HashRouter as Router, Route, Switch } from 'react-router-dom'

function App() {
  return (
    <Router>
      <Switch>
        <Route path='/'>
          <div>可视化开发平台</div>
        </Route>
      </Switch>
    </Router>
  )
}

ReactDOM.render(<App/>, document.getElementById('root'))
```

### 修改electron主线程配置，配合react做单页面应用

既然基础文件结构改了，那么 `.app/main/electron.js` 也得跟着增加以下配置：

```js
...
const isDev = () => {
  // 对应 webpack.main.dev.js里的 webpack.DefinePlugin的定义
  return process.env.NODE_ENV === 'development';
}
...
const createWindow = () => {
  ...

  // 利用
  if (isDev()) {
    mainWindow.loadURL('http://127.0.0.1:7001')
  } else {
    mainWindow.loadURL(`file://${path.join(__dirname, './dist/index.html')}`)
  }
}
```

`package.json` 的 `script` 修改如下

```json
"scripts": {
  "start:main": "cross-env NODE_ENV=development webpack --config ./webpack.main.dev.js && electron ./dist/electron.js",
  "start:render": "webpack-dev-server --config ./webpack.renderer.dev.js"
},
```

## 报错处理

> 如果出现报错：Uncaught ReferenceError: require is not defined，请检查你是否在主进程中添加这行代码，如果添加了，请确保你搭建项目的 Electron 与本应用的版本一致(当前项目的 Electron@^11.1.1)

> 请自检查一下你的版本是否正确，进入 node_modules，找到 electron，看看 package.json 中的 version 是否是 11.1.1。