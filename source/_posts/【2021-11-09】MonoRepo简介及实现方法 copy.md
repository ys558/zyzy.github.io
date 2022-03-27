---
title: Mono Repo 简介及实现方法
date: 2021-11-09 11:23:15
tags:
  - mono repo
  - yarn
  - lerna
  - conventional commit
cover: 'https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.43/articles/monorepo简介及实现方法/cover.jpeg'
---

![](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.43/articles/monorepo简介及实现方法/cover.jpeg)

本文介绍私有仓库包的管理模式 mono repo

<!-- more -->

## mono repo的好处
mono repo的好处来源于[这篇文章](https://danluu.com/monorepo/)，我用自己直白点的话总结了以下几点：
1. 多个内部模块可以集中管理
2. 只需在根目录安装 `node_modules` 即可，无须所有模块都安装

例如，一个有多端的项目，而且每个端都饮用了共有的模块，我们可以把所有项目或包放到一个packages中集中管理

例如我们平时熟悉的react，babel等库的源码，都是采用mono repo进行仓库管理。

我们需要用到两个库，`yarn workspace` 和 `lerna`， 以下的项目示例里，前者用于依赖管理，后者用于处理发布问题，在版本发布这块，使用`lerna`更方便，可用于替代一部分的git的功能。

当然 `lerna` 也可以直接替代 `yarn workspace` 的功能。

## [`yarn workspace`](https://classic.yarnpkg.com/en/docs/cli/workspace) 管理依赖

```bash
mkdir common server
cd common && yarn init -y && touch index.js
cd ../server/ && yarn init -y && touch index.js
```

### `package.json` 配置 `workspaces`  
`package.json`，将 common, server 两个文件夹视为两个package包
```json
{
  "private": true,
  "workspaces": [ "packages/*" ]
}
```

建一个简单的目录结构如下：
```bash
root
  |__package.json
  |__packages
    |__common
      |__package.json
    |__server
      |__package.json
```

假设在common包里安装react和react-dom，则直接执行：

```bash
yarn workspace common add react react-dom --dev
```

`yarn workspace` 的具体用法，可以看[官方文档](https://classic.yarnpkg.com/en/docs/cli/workspace)
### 自己写的包要相互引用的方法
我们把上面的目录结构 common 包和 server 各自添加index.js，如下：
```bash
root
  |__package.json
  |__packages
    |__common
      |__index.js
      |__package.json
    |__server
      |__index.js
      |__package.json
```

`packages\common\index.js` 里写点简单的东西：
```js
module.exports = () => console.log('hello fr common')
```

`packages\server\index.js`
```js
// 这里的 "common" 对应 common\package.json 里的 "name"，即直接引用common作为依赖包
const commonFn  = require("@mono-repo-by-yarn-lerna/common");
```

`common\package.json` 里，社区里的习惯性做法是加上前缀`@项目名称/`，如：
```json
{
  "name": "@mono-repo-by-yarn-lerna/common",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "dependencies": {
    "@mono-repo-by-yarn-lerna/common": "1.0.0"
  }
}
```

`server\package.json` 运行 `yarn install`，会看到以下，证明在server文件夹成功安装common文件夹作为依赖
```bash
$ yarn
yarn install v1.22.10
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
success Saved lockfile.
Done in 0.16s.
```

返回根目录运行 `ls node_modules/` 可以看到多了 '@mono-repo-by-yarn-lerna' / 为项目的东西
```bash
$ ls node_modules/
'@mono-repo-by-yarn-lerna'/
```

![ yarn workspace 管理依赖](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.43/articles/monorepo简介及实现方法/01.png)

运行 `node server/index.js` 可以看到运行的结果：
```bash
$ node packages/server/index.js 
hello fr common
```

如果此时我们再在 server 文件夹里添加其他模块，例如 `babel`，运行 `yarn add babel`，

![ yarn workspace 管理依赖 ](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.43/articles/monorepo简介及实现方法/02.png)

会发现 server 文件夹里的 `node_modules` 只放了 babel 的一些执行文件，其余 babel 的核心文件全部被移动到根目录里的 `node_modules`，以此达到集中管理依赖的目的

## Lerna

> A tool for managing JavaScript projects with multiple packages. 

[Lerna](https://github.com/lerna/lerna) 是希腊神话里一多头怪物，估计作者想他多头处理多个包 

在我们上面项目的基础上，根目录添加lerna，`yarn add -D -W lerna`，`-W`表示跟在本项目的 workspace 里直接安装，即根目录

`npx lerna init`

`packages\common\package.json` 和 `packages\server\package.json` 均加上test命令
```bash
  "scripts": {
    "test": "echo testing server with version: $npm_package_version"
  }
```

```json
{
  "packages": [
    "packages/*"
  ],
  "version": "0.0.0",
  + "npmClient": "yarn",
  + "useWorkspaces": true
}
```

![ lerna运行后，会收集所有子包script里对应同名的命令一起执行 ](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.43/articles/monorepo简介及实现方法/03.png)

如果要执行单个 `package.json` 里的命令，则需加上 `--scope=@包名`，如

```json
  "scripts": {
    "test": "lerna run test --scope=@mono-repo-by-yarn-lerna/common"
  }
```

![ 加上 --scope= 指定运行包名 ](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.43/articles/monorepo简介及实现方法/04.png)

如指定多个模块，只需放在`{}`里，如：
```json
  "scripts": {
    "test-since": "lerna run test --scope={@mono-repo-by-yarn-lerna/common,@mono-repo-by-yarn-lerna/server}"
  }
```
### lerna version 发布版本
接上面，用 `new-version` 命令进行发版，执行的是 `lerna version`。
后面的参数 [`convential commits`](https://www.conventionalcommits.org/en/v1.0.0/) 是一个用于优化 git commit 内容的库，可以添加commits标题正文注脚什么的，挺有趣，vs code 里也有同名的插件

![ 执行 new-version 命令后会生成版本信息 ](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.43/articles/monorepo简介及实现方法/06.png)

![ 同时，各个模块也会生成一个 `CHANGELOG.md` 文件 ](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.43/articles/monorepo简介及实现方法/07.png)

![ 远程的git仓库会生成相应的版本号 ](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.43/articles/monorepo简介及实现方法/08.png)

test 脚本加上 `--since` 参数可以看到提交版本的历史：
```json
  "scripts": {
    "test": "lerna run test --since"
  }
```