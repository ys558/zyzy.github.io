---
title: Web性能优化——由面试题引发的思考
date: 2021-05-30 17:38:14
tags:
---

几乎所有网上的Web性能优化出处均来自这篇文章[Best Practices for Speeding Up Your Web Site](https://developer.yahoo.com/performance/rules.html)，被大家称作"雅虎35条军规"，当然，不同人有不同的解读，而且里面有的东西也已经过时了，本文结合"35条军规"进行阐述

<!-- more -->
## 从输入网址到页面响应，


## 网页性能检测工具

### Chrome自带的performance检测工具
### window.performance

打开任意网页，控制台输入以下代码，
可以看到 `window.performance` 所获取到的东西，我们主要看他的`timing`属性，用开始时间减去结束时间得出各种资源加载的时间

```js
const t = window.performance.timing;
console.log({
    "DNS": t.domainLookupEnd - t.domainLookupStart,
    "TCP": t.connectEnd - t.connectStart,
    "获得首字节耗费时间，TTFB": t.responseStart - t.navigationStart,
    "domReady时间": t.domContentLoadedEventStart - t.navigationStart,
    "DOM资源下载": t.responseEnd - t.responseStart
})
```
第一字节响应时间（TTFB）= 从发送请求到WEB服务器的时间 + WEB服务器处理请求并生成响应花费的时间 + WEB服务器生成响应到浏览器花费的时间

要考虑的问题：
步骤1：从发送请求到WEB服务器的时，即向站点地址提交首次请求
- DNS 响应时间（终端用户侧解析 DNS 请求有多块）
- 网站服务器到终端用户的距离，越短越好
- 网络稳定性

步骤2：WEB服务器处理请求并生成响应花费的时间，即由 web 服务器解析本次请求
- 物理硬件响应时间 （web 服务器解析请求有多快）
- 既有的服务器操作负载
- 数据中心任何网络相关的延迟

步骤3：WEB服务器生成响应到浏览器花费的时间， 即向客户端发送首个响应的时间
- 终端用户的网速
- 连接稳定性

### lighthouse 生成网页性能报告


## Content（内容）优化