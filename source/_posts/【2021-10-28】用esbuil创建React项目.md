---
title: 【2021-10-28】用esbuil创建React项目
date: 2021-10-28 14:53:06
tags:
  - esbuild
  - React
  - webpack
---

webpack 用于大型项目中，特别是开发时候的热更新，速度太慢，原因是`webpack` 采用了整个项目所有文件一起打包的方案。

自从 `vite` 推出以来，打包这块做到了性能上的超越。`vite` 的原理是在SPA项目中，基于入口文件打包的，由于只打包一个文件，所以速度就上来了。

而无论是 `vite` 和 `webpack` 均是基于 `esbuild` 开发的。所以研究一下 `esbuild` 的配置是比较有价值的。

本项目是基于 `create-react-app` 创建项目，再用 `es-build` 作为开发热更新打包。

<!-- more -->

## 不改动CRA生成的基础结构下的改动

用 `create-react-app` 生成项目后，然后对原来的项目作了如下改动：

- 核心配置在 `devBuild.js` 文件，如下：

```js
import browserSync from "browser-sync";
import chalk from "chalk";
import commandLineArgs from "command-line-args";
import del from "del";
import esbuild from "esbuild";
import getPort from "get-port";
import svgrPlugin from "esbuild-plugin-svgr";
// 创建服务器。
const bs = browserSync.create();
// 解构环境变量
const { dev } = commandLineArgs({ name: "dev", type: Boolean });
// 删除文件夹 public 中的打包文件夹
del.sync("./public/dist");

// 开始 esbuild 打包
(async () => {
  const buildResult = await esbuild
    .build({
      format: "esm", // 设置生成的 JavaScript 文件的输出格式。
      target: "es2017", // 编译转化版本
      entryPoints: ["./src/index.jsx"], // 打包入口
      outdir: "./public/dist", // 输出目录
      chunkNames: "chunks/[name].[hash]", // 打包出来的文件名
      incremental: dev, // 因为我们监听文件的改变重新打包，而且我们要开发环境使用esbuild 所以 dev 为 true
      loader: {
        // 此选项更改给定输入文件的解释方式。
        ".svg": "text",
        ".png": "dataurl",
      },
      bundle: true, // 捆绑文件意味着将任何导入的依赖项内联到文件本身中。
      splitting: true, // 代码拆分目前仅适用于esm输出格式。
      plugins: [svgrPlugin()],
      inject: ["./public/react-shim.js"], // 将 React 作为全局变量导入esbuild
    })
    .catch((err) => {
      console.error(chalk.red(err));
      process.exit(1);
    });
  console.log(chalk.green("The build has finished! 📦\n"));
  // 获取可以使用的端口号
  const port = await getPort({
    port: getPort.makeRange(4000, 4999),
  });

  console.log(
    chalk.cyan(
      `Launching the Shoelace dev server at http://localhost:${port}! 🥾\n`
    )
  );
  // 服务器初始化
  bs.init({
    startPath: "/", // 初始路径
    port, // 端口号
    logLevel: "silent", // 日志级别
    logFileChanges: true, // 日志文件更改
    notify: true, // 浏览器中的小弹出通知
    single: true, // 提供单独的 index.html
    server: {
      baseDir: "public", // 基础文件夹
      index: "index.html", // 设置服务器的入口文件
    },
    files: "src/", // 监听 src 下的文件
  });

  // 监听 src 文件夹下的更改
  bs.watch(["src/"]).on("change", async (filename) => {
    console.log(`Source file changed - ${filename}`);
    // 重新打包
    buildResult.rebuild();
  });
})();


```

- 依赖安装：
  
  核心: `esbuild`,`esbuild-plugin-svgr`  
  用于创建服务渲染打包文件： `browser-sync`  
  解析命令行参数: `command-line-args`  
  打包文件删除：`del`
  获取当前可用端口：`get-port`
  美化：`chalk`

- `package.json` 的 `script` 增加了 `dev` 命令，为了跑 `devBuild.js` 文件

- `package.json` 增加了 `{"type": "module"}` 让 node 可以编译 esm 语法

- 将 `public/index.html` 文件增加如下：

  ```html
  ...
  <link rel="stylesheet" type="text/css" href="./dist/index.css" />
  ...
  <script type="module">
    import './dist/index.js'
  </script>
  ```

- 增加 `public/react-shim.js` 文件，并在 `devBuild.js`写入相应配置，在src中就不用到处引入React了：

  ```js
  import * as React from 'react'
  export { React }
  ```

## [源码在这里]()