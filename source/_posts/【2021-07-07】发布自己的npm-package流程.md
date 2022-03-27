---
title: 发布自己的npm-package流程
date: 2021-07-07 09:11:03
tags:
    - npm package
cover: https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.41/articles/发布自己的npm-package流程/cover.png
---

![](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.41/articles/发布自己的npm-package流程/cover.png)

最近工作需要，需发布自己的npm包，这里写一个简单的demo，做一个简单的记录。

<!-- more -->

## 1. 注册一个自己的[npm账号](https://www.npmjs.com/signup)
这个没什么好解释的

## 2. 建一个[github仓库](git@github.com:ys558/my-npm-module-test.git)
这个没什么好解释的

## 3. 项目结构

```
|__/package/
|__/test-folder/
```

`/package` 文件夹用于存放我们写的包   
`/test-folder` 文件夹用于引用并测试 `/package` 里的包

编写包 `my-package-publish-test`

在 `/package` 里初始化项目 `npm init` 并写一些关于该 package 的名字，如`my-package-publish-test` 和描述等东西    
像写普通文件一样，新建`/package/index.js` 并写下一些简单功能，如

```js
const isTest = string => string === 'test'
module.exports = isTest
```

## 4. 通过软链接在本地使用包`/package`

如不上传包到npm，本地使用可以直接使用软链接，可以用于本地调试等工作，方法如下。如要直接上传可跳过该步骤

- 在当前目录 `./package/` 下运行创建软链接
```bash
npm link
```

- 返回 `./test-folder/` 下运行软链接，软链接后的名必须对应 `package/package.json` 的 `"name"`

```bash
cd ../test-folder/
npm link my-package-publish-test
```

## 5. 登录并发布
返回 `./package/` 并执行登录，运行
```bash
cd ../package/
npm login
```

输入一些 `npmjs.org`的用户名密码等，登录npm账号
```bash
$ npm login
Username: ys558
Password:
Email: (this IS public)
Email: (this IS public) yuyi.gz@163.com
Logged in as ys558 on https://registry.npmjs.org/.
```

！！如出现以下报错需要登录自己的邮箱是否通过 npmjs.org 的验证邮件

```bash
npm ERR! code E403
npm ERR! 403 403 Forbidden - PUT http://registry.npmjs.org/my-package-publish-test - Forbidden
npm ERR! 403 In most cases, you or one of your dependencies are requesting
npm ERR! 403 a package version that is forbidden by your security policy.

npm ERR! A complete log of this run can be found in:
npm ERR!     C:\Users\yuyi\AppData\Roaming\npm-cache\_logs\2021-07-04T12_02_21_879Z-debug.log
```

最后运行以下命令，即可发布成功属于自己的npm package
```bash
npm publish
```

发布成功：
```bash
$ npm publish
npm notice 
npm notice package: my-package-publish-test@1.0.0
npm notice === Tarball Contents ===
npm notice 85B  index.js
npm notice 564B package.json
npm notice 21B  README.md
npm notice === Tarball Details ===
npm notice name:          my-package-publish-test
npm notice version:       1.0.0
npm notice package size:  509 B
npm notice unpacked size: 670 B
npm notice shasum:        98b90c3ab222a573f0379802cc3d59d43ffcb402
npm notice integrity:     sha512-Fz/SDMr5vgoAe[...]UD5J4OP0rFtFA==
npm notice total files:   3
npm notice
+ my-package-publish-test@1.0.0
```

## 6. 发布具有命名空间的package

所谓的命名空间，就是以 `@` 开头的包，例如我们熟悉的 `@babel/core`，一般这种需要创建组织organization，步骤如下：

### 6.1 在自己的npm页面申请成立一个组织，organization

我把几个关键步骤进行了截图，如下

![npmjs.org的右上角可找到添加organization的地方](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.41/articles/发布自己的npm-package流程/01.png)

如果有条件可以选择收费的，其实也不贵，一个月也就7美刀，40多软妹币；也可以直接将你的账号转换为组织账号，我这里为了演示，选择免费公开账号第二个选项：

![跟着步骤一步步添加即可](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.41/articles/发布自己的npm-package流程/04.png)

![成功创建好名为zyzy-org的账号](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.41/articles/发布自己的npm-package流程/02.png)


### 6.2 重新用组织的名进行登录

执行 `npm login scope=<组织名称>` ，用户名也和组织名称一样：

```bash
$ npm login scope=zyzy-org
Username: zyzy-org
Password:
Email: (this IS public) yuyi.gz@163.com
Logged in as zyzy-org on https://registry.npmjs.org/.
```

### 6.3 在发布的包前加上 `@组织名`

将该项目的 `package\package.json` 里 `"name"` 的属性改为如下，前面加个 `@zyzy-org/`

```json
{
  "name": "@my-test-package/my-package-publish-test",
}
```
### 6.4 运行 `npm publish --access=public`

即可见到发布成功：

```bash
$ npm publish --access=public
npm notice 
npm notice package: @zyzy-org/my-package-publish-test@1.0.0
npm notice === Tarball Contents ===
npm notice 69B  index.js
npm notice 574B package.json
npm notice 21B  README.md
npm notice === Tarball Details ===
npm notice name:          @zyzy-org/my-package-publish-test
npm notice version:       1.0.0
npm notice package size:  503 B
npm notice unpacked size: 664 B
npm notice shasum:        ddc493df353964be87f43d14e5f7fd877b7b4c04
npm notice integrity:     sha512-XFE19u9gFMcI1[...]1/LA3ocSzHZ8Q==
npm notice total files:   3
npm notice
+ @zyzy-org/my-package-publish-test@1.0.0
```

### 6.5 新项目初始化时，须运行 `npm init scope=zyzy-org`

## 7 删除已发布的包

```bash
npm unpublish @zyzy-org/my-package-publish-test --force
```