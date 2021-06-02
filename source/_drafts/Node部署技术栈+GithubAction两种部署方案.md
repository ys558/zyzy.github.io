---
title: Node部署技术栈+GithubAction两种部署方案
date: 2021-05-30 09:49:14
tags:
---

昨天刚撸完 Nginx 的基础配置，今天迫不及待用 Node + Nginx 和其他 Node 技术栈来部署来实验一番，另外，现在流行纯前端的部署模式，这骚操作当然要第一时间学起来，所以本文也研究另一种部署方法，Github Action。

<!-- more -->

## Node + Nginx部署

### 先建一个Node应用

建一个及其简单的Node应用

```js
const express = require('express')
const app = express()

app.get('/', (req,res) => res.send('<h1>Node App</h1>'))
app.listen(5000, () => console.log('App listening on port 5000'))
```

## Github Action

