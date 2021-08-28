---
title: 事件总线机制EventEmitter在react中的使用
date: 2021-08-14 22:19:32
tags: 
  - react
  - eventEmitter
---

最近很忙，搁了很久没更新了，趁着今天空闲来更新一波。最近解锁了组件传值的新模式，EventEmitter，fb也封装了一个[`fbemitter`](https://www.npmjs.com/package/fbemitter)的组件，用于fb网站间的传值。用过vue的小伙伴对event emitter肯定不陌生。这里结合react的使用来简单说明一下用法。
<!-- more -->

## 引入

```js
import { EventEmitter } from 'fbemitter';
var emitter = new EventEmitter();
```

## 定义事件

写两个没有层级关系的组件：

```js
import React, { Component } from 'react'
import { EventEmitter } from 'fbemitter'

export default class EventEmitterDemo extends Component {
  render() {
    // 0. 可用于无层级关系组件传值：
    return <>
        <Child1 />
        <Child2 />
      </>
  }
}

// 1. 初始化
const emitter = new EventEmitter()

class Child1 extends Component {
  state = { show:　'' }
  componentDidMount() {
    // 2. 定义
    this.eventEmitter = emitter.addListener(
      'event', (x,y)=> this.setState({ show: <>{x},{y}</> })
    )
  }

  render(){
    return <div>
      {this.state.show}
    </div>
  }
}
```

## 发出并触发事件

```js
class Child2 extends Component {
  // 3. 发出事件名称及参数： 
  render(){
    return <div>
      <button onClick={()=> emitter.emit('event', 12, 22)}>eventEmitter发出，见控制台</button>
    </div>
  }
}
```

## 效果
![两个无嵌套关系的组件间传值](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.42/articles/事件总线机制EventEmitter在react中的使用/动画.gif)

## once(eventType, callback)

只调用一次的eventEmitter，如：
```js
  componentDidMount() {
    this.eventEmitter = emitter.once(
      'event', (x,y)=> console.log(x,y)
    )
  }

  emitter.emit('event', 13); // 10 
  //再次触发
  emitter.emit('event', 12); // 不会输出
```

## remove(eventType) 及 removeAllListeners(eventType)

删除事件，一般我们在组件卸载时应该执行`remove`进行删除，而`removeAllListeners`则可以一次性删除所有监听事件，如：

```js
componentWillUnmount(){
  // 3. 当组件卸载时要将其删除
  this.eventEmitter.remove();
  // 或：
  this.removeAllListeners()
}
```
