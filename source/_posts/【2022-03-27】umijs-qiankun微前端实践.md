---
title: umijs-qiankun微前端实践
date: 2022-03-27 14:18:15
tags:
  - umijs
  - qiankun
---

umijs推出的微前端框架qiankun，是国内比较流行的微前端方案，其官方文档有配置的方法，但是说的比较模糊，这里亲自实践一下

<!-- more -->

## 创建项目

微前端分位主应用和若干个微应用，分别独立运行。所以分别创建以下目录解构：

```bash
<project root>
    |__main
    |__app1
    |__app2
```

根据[umijs文档](https://umijs.org/zh-CN/docs/getting-started)，分别在每个项目里生成对应的umijs项目，并都安装
[umijs乾坤插件 @umijs/plugin-qiankun](https://umijs.org/zh-CN/plugins/plugin-qiankun)

```bash
npm i yarn tyarn -g
tyarn create @umijs/umi-app
tyarn add @umijs/plugin-qiankun
tyarn
```

## 修改.umirc配置

这个是文档没说清楚的地方，所有umijs的配置，均要在.umirc里修改，在main项目添加以下配置：

```javascript
   routes: [
    {
      path: '/',
      component: '@/pages/index',
      routes: [
        {
          path: '/app1',
          microApp: 'app1',
        },
        {
          path: '/app2',
          microApp: 'app2',
        },
      ],
    },
  ],
  qiankun: {
    master: {
      // 注册子应用信息
      apps: [
        {
          name: 'app1', // 唯一 id
          entry: '//localhost:7701', // html entry
        },
        {
          name: 'app2', // 唯一 id
          entry: '//localhost:7702', // html entry
        },
      ],
    },
  },
```

在 `app1` 和 `app2` 的 `.umirc` 分别写：

```js
  qiankun: {
    slave: {},
  },
```

在 `app1` 和 `app2` 的 `package.json` 分别指定app名字，否则qiankun插件认不出来：

```json
// app1 package.json 
"name": "app1"

// app2 package.json 
"name": "app2"
```

这里注意，微服务要跑在不同端口，根据[`在.env文件中定义`](https://umijs.org/zh-CN/docs/env-variables#%E5%9C%A8-env-%E6%96%87%E4%BB%B6%E4%B8%AD%E5%AE%9A%E4%B9%89)，我们可在 `main` `app1` `app2` 下分别创建 `.env` 文件并写下端口号，如：

```bash
# main
PORT=7700

# app1
PORT=7701

# app2
PORT=7702
```

这样，当每个项目各自跑的时候，就能按要求跑在指定的端口号

## 子项目配置 `app.ts` 文件

这个是关键一步，每个子项目都要配置对应的子项目生命周期函数，否则报错：

```ts
// app1/src/app.ts
export const qiankun = {
  // 应用加载之前
  async bootstrap(props: any) {
    console.log('app1 bootstrap', props);
  },
  // 应用 render 之前触发
  async mount(props: any) {
    console.log('app1 mount', props);
  },
  // 应用卸载之后触发
  async unmount(props: any) {
    console.log('app1 unmount', props);
  },
};

// app2/src/app.ts
export const qiankun = {
  // 应用加载之前
  async bootstrap(props: any) {
    console.log('app2 bootstrap', props);
  },
  // 应用 render 之前触发
  async mount(props: any) {
    console.log('app2 mount', props);
  },
  // 应用卸载之后触发
  async unmount(props: any) {
    console.log('app2 unmount', props);
  },
};
```

## 更改界面

走到这一步, `app1` 和 `app2` 均会配置到主应用 `main` 的`props.children`里了，我们将主应用界面更改如下：

```tsx
// main/src/pages/index.tsx

import styles from './index.less';

export default function IndexPage(props) {
  console.log('props----->', props.children.props.children);
  return (
    <div>
      <h1 className={styles.title}>Main App</h1>
      <div>{props.children}</div>
    </div>
  );
}
```

最终效果如下，通过`/app1` 可达 app1 应用，`/app2`可达 app2 应用：

![umijs-qiankun微前端效果](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@1.50/articles/umijs-qiankun微前端实践.png)

