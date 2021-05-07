---
title: 脱离脚手架webpack配制React+typescript项目（二）
tags:
---
## 配置 ts

```shell
yarn add -D @types/react @types/react-dom
```

### 创建 `tsconfig.json`
并写下：
```json
{
  "compilerOptions": {
    "jsx": "react"
  }
}
```

参数的意义：
preserve: 生成的代码会保留jsx的格式，给后续的操作做好准备（如传递给babel继续处理）   
react-native: 生成的代码是jsx,扩展名是js   
react: 会转换成纯的js语法    

修改 `webpack.config.js` 的入口文件：   

```js

```

```shell
touch tsconfig.json 
yarn add -D typescript @types/react @types/react-dom
```

```shell
yarn add -D @babel/preset-typescript
```



tsconfig.json


## webpack配置优化

### `happypack` 多进程处理打包，加快打包速度

可点击 [happypack repo](https://github.com/amireh/happypack) 查阅

`webpack.config.js` 里替换：   
```js
// @file: webpack.config.js
const HappyPack = require('happypack');

exports.module = {
  rules: [
    {
      test: /.js$/,
      // 1) replace your original list of loaders with "happypack/loader":
      // loaders: [ 'babel-loader?presets[]=es2015' ],
      use: 'happypack/loader',
      include: [ /* ... */ ],
      exclude: [ /* ... */ ]
    }
  ]
};

exports.plugins = [
  // 2) create the plugin:
  new HappyPack({
    // 3) re-add the loaders you replaced above in #1:
    loaders: [ 'babel-loader' ]
    id: 'babel',

  })
];
```

其中，实例的 `HappyPack` 参数 `loaders` 为数组，其 `'babel-loader'` 可以改为: 
```js
  new HappyPack({
    loaders: [ 'babel-loader?presets[]=es2015' ]
  })
```

表示 `babel-preset-es2015` 插件，一个老的 `babel` 插件，还可以有其他易读的写法：

```js
{
  loaders: [
    // a string with embedded query for options
    'babel-loader?presets[]=es2015',

    {
      loader: 'babel-loader'
    },

    // "query" string
    {
      loader: 'babel-loader',
      query:  '?presets[]=es2015'
    },

    // "query" object
    {
      loader: 'babel-loader',
      query: {
        presets: [ 'es2015' ]
      }
    },

    // Webpack 2+ "options" object instead of "query"
    {
      loader: 'babel-loader',
      options: {
        presets: [ 'es2015' ]
      }
    }
  ]
}
```