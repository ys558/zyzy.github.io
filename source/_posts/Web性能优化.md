---
title: Web性能优化（持续更新）
date: 2021-05-30 17:38:14
tags:
---

Web性能优化出处最早估计源自这篇文章[Best Practices for Speeding Up Your Web Site](https://developer.yahoo.com/performance/rules.html)，被大家称作"雅虎35条军规"，里面有的东西虽说也已经过时了，但我们可以由此窥探前端发展的历程，本文结合"35条"进行阐述，一些过时的建议会略去

<!-- more -->
## 一道面试题引发的前端性能优化思考

相信大家都碰过这道经典面试题：在浏览器输入url到看到页面的展示，这中间发生了什么？

- 

## 网页性能检测工具

首先应明确检测网页所需的工具，这里列举了3种方法：

### window.performance

打开任意网页，控制台输入以下代码，
可以看到 `window.performance` 所获取到的东西，我们主要看他的`timing`属性，用开始时间减去结束时间得出各种资源加载的时间。以下的数值都算是比较粗略的数值

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

### Chrome自带的performance检测工具
### 自动化检测利器—— `lighthouse` —— 自动生成网页性能报告，并有优化建议

```shell
npm i -g lighthouse
```

使用起来超级简单
```shell
lighthouse <需要测试的网页链接>
```

# [雅虎35条](https://developer.yahoo.com/performance/rules.html)

## Image *

之所以把图片放在最前，因为优化图片的效率是比较大的

### Optimize Images *
图片主要就是压缩和优化

pngcrush 或其他工具压缩png。在线压缩工具 https:/ tinypng.com/   
jpegtran或其它工具压缩jpeg。   
大图用jpg   
SVG矢量图：类似XML的图片（用来绘制地图，股票K线图等）   
Webp google开发的一种全能的图片显示技术，但浏览器的兼容性不太好   

- 渐进式显示
- 懒加载

### Optimize CSS Sprites

> - Arranging the images in the sprite horizontally as opposed to vertically usually results in a smaller file size.   
> - Combining similar colors in a sprite helps you keep the color count low, ideally under 256 colors so to fit in a PNG8.   
> - "Be mobile-friendly" and don't leave big gaps between the images in a sprite. This doesn't affect the file size as much but requires less memory for the user agent to decompress the image into a pixel map. 100x100 image is 10 thousand pixels, where 1000x1000 is 1 million pixels   

- 雪碧图最好竖放，避免横放，达到最小尺寸
- 相似图片合并，颜色相近的合并，颜色数会更少
- 移动端的雪碧图减少空隙


选择器性能
内联样式 (style="") > ID 选择器 (id) > 类选择器 (class) = 属性选择器 ( a[href], input[type="text"] 等 ) = 伪类选择器 (nth-child(n), :hover, :active 等) > 元素（类型）选择器  = 伪元素选择器

### Do Not Scale Images in HTML

> If you need `<img width="100" height="100" src="mycat.jpg" alt="My Cat" />`
> then your image (mycat.jpg) should be 100x100px rather than a scaled down 500x500px image.

不要在HTML中缩放图片，如果你需要 `<img width="100" height="100" src="mycat.jpg" alt="My Cat" />` 的图片，直接做一张 `100*100` 的图即可，而不是拿一张 `500*500` 的图片进行缩放


### Make favicon.ico Small and Cacheable

使得 `favicon.ico` 图片可以缓存，如果不进行缓存，不关心他，浏览器还是会请求他

## Content 

### Make Fewer HTTP Requests *

减少 HTTP 请求

### Reduce DNS Lookups

减少DNS查询

### Avoid Redirects *

避免重定向

### Make Ajax Cacheable

使得 Ajax进行缓存

### Postload Components

后加载组件

### Preload Components

预加载组件

### Reduce the Number of DOM Elements
### Split Components Across Domains
### Minimize Number of iframes
### Avoid 404s

## Server

### Use a Content Delivery Network (CDN) *

例如淘宝的页面，直接在头部加DNS script标签即可，
另外，加载CDN时是静态加载，不会携带Cookie，算是优化。

```html
<link rel="dns-prefetch" href="//g.alicdn.com" />
```

### Add Expires or Cache-Control Header *
> There are two aspects to this rule:
> 
> - For static components: implement "Never expire" policy by setting far future Expires > header
> - For dynamic components: use an appropriate Cache-Control header to help the browser > with conditional requests

雅虎建议将 `Expires` 字段用于所有组件，不单止于图片：
> Expires headers are most often used with images, but they should be used on all components including scripts, stylesheets, and Flash components.

静态文件，请求头可以设置永不过期或者把时间设置的长一些
```shell
expires: never

# or
Expires: Thu, 15 Apr 2099 20:00:00 GMT
```

对于动态组件，利用请求头的`Cache-Control` 控制，例如
```
cache-control: max-age=2592000
```

`Cache-Control`的值有以下几种情况：

no-cache 直接要服务的新内容，不拿缓存的   
no-store 不缓存请求或响应的任何内容   
max-age 响应的最大Age值   
min-fresh 期望在指定时间内的响应扔有效   
only-if-chache 从缓存获取资源   
max-stale 接收已过期响应   
min-fresh 期望在指定时间内的响应仍有效   
no-transform 代理不可更改媒体类型   
cache-extension 新指令标记（token）   

其中，最常用的是 `no-cache`，`no-store`，`max-age` 3个值

### Gzip Components *

网页中重复的内容，会用Gzip压缩，显著减少文件大小，在 [Nginx](https://zyzy.info/2021/05/28/Nginx%E8%87%AA%E5%AD%A6%E7%AC%94%E8%AE%B0/#Gzip%E5%8E%8B%E7%BC%A9) 里有Nigix配置Gzip的介绍

请求头中添加：
```
Accept-Encoding: gzip, deflate
```
相应头中添加：
```
Content-Encoding: gzip
```

### Configure ETags

> Entity tags (ETags) are a mechanism that web servers and  browsers use to determine whether the component in the browser's cache matches the one on the origin server. 

`ETags` 是一种机制，用来确定浏览器的缓存内容和服务器的是否匹配，如匹配，则用浏览器的内容，如不匹配，则请求服务器新的内容

请求时的头部字段：
```shell
Last-Modified: Tue, 12 Dec 2006 03:03:59 GMT
ETag: "10c24bc-4ab-457e1c1f"
```

响应的头部字段：
```shell
If-Modified-Since: Tue, 12 Dec 2006 03:03:59 GMT
If-None-Match: "10c24bc-4ab-457e1c1f"
HTTP/1.1 304 Not Modified
```

### Flush Buffer Early
### Use GET for Ajax Requests
### Avoid Empty Image src

## Cookie
### Reduce Cookie Size *

> Eliminate unnecessary cookies    
> Keep cookie sizes as low as possible to minimize the impact on the user response > time   
> Be mindful of setting cookies at the appropriate domain level so other > sub-domains are not affected   
> Set an Expires date appropriately. An earlier Expires date or none removes the cookie sooner, improving the user response time   

Cookie是请求头的一个字段，如果存储的信息过多过大，必然会影响性能，减少Cookie体积大小，只存储用户id等简单信息    
设置合适的 `expire` 字段让cookie过期


### Use Cookie-Free Domains for Components


## CSS

### Put Stylesheets at Top *
> While researching performance at Yahoo!, we discovered that moving stylesheets to the document HEAD makes pages appear to be loading faster. This is because putting stylesheets in the HEAD allows the page to render progressively.

将样式表放在头部，可以让页面逐步呈现

### Avoid CSS Expressions
### Choose `<link>` Over @import

> In IE @import behaves the same as using <link> at the bottom of the page, so it's best not to use it.

在IE浏览器中 `@import` 和 `<link>` 是一样的，位于底部执行，这和我们推荐的CSS放在HEAD中执行背道而驰，所以少用 `@import`

### Avoid Filters

## JavaScript

### Put Scripts at Bottom *
> The problem caused by scripts is that they block parallel downloads.

> While a script is downloading, however, the browser won't start any other downloads, even on different hostnames.

JS的加载本身就是一种阻塞，所以尽量让HTML+CSS先把页面渲染出来，再执行 底部的`<script></script>` 标签

### Make JavaScript and CSS External

> if the JavaScript and CSS are in external files cached by the browser, the size of the HTML document is reduced without increasing the number of HTTP requests.

(除了主页) 使用CSS或JS的外部链接，浏览器会缓存 JavaScript 和 CSS 文件，而不会增加 HTTP 请求数。

```html
<link rel="stylesheet" href="/static/css/xxx.min.css">
<script nonce="b3/zKeoFCfru0lTFQr8Dyg==" src="https://s.yimg.com/ss/rapid-3.41.3.js"></script>
```

### Minify JavaScript and CSS *
> Minification is the practice of removing unnecessary characters from code to reduce its size thereby improving load times. 

最小化 JS 和 CSS，现在我们多用 webpack等打包工具来做到这一步

### Remove Duplicate Scripts

删除重复的JS脚本

### Minimize DOM Access *

> Accessing DOM elements with JavaScript is slow so in order to have a more responsive page, you should:
> 
> - Cache references to accessed elements   
> - Update nodes "offline" and then add them to the tree   
> - Avoid fixing layout with JavaScript  

最小化DOM访问，尽可能少的进行DOM操作，这点可以从现在的`React` `Vue`等MVVM框架体现出来

### Develop Smart Event Handlers

## Mobile

### Keep Components Under 25 KB
### Pack Components Into a Multipart Document

# 新增内容

## API

### 利用