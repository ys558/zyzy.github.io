---
title: Vue2+3自学笔记（一）
date: 2021-05-19 16:20:33
tags:
    - Vue
cover: https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.32/articles/Vue2+3自学笔记（一）/cover.png
---

一直以来，做项目都是用React，最近公司的项目接触到 Vue，这里先捡起多年未碰的 Vue ，写一些demo， 重温一下 Vue2 最简单的用法，并结合[掘金的这篇文章](https://juejin.cn/post/6940454764421316644)，看看 Vue3 的新变化

<!-- more -->

从 Vue2 开始搬砖，自己写了个TODO List，完整代码在[这里](https://github.com/ys558/vue-learn-demos/tree/master/vue-base-demo-TODOList-2021)，配合[Vue官方文档](https://cn.vuejs.org/)内容，进行本文的阐述

## 脚手架开始项目
安装
```bash
yarn global add @vue/cli
```

脚手架创建项目并运行：
```bash
vue create <AppName> && cd <AppName>
vue run serve
```

## `import` 后仍要声明组件

由于vue是经过实例化的，属性都绑定在this上，以this作为中转，所以 `import` 后仍要在 `component` 中声明一次，否则报错：

```html
<script>
import Header from './components/Header'

export default {
  components: {
    Header,
  },
}
</script>
```

## `props` 属性可以声明数组或对象

声明对象时类似ts的类型检验

```js
  props: ['text', 'color', 'title', 'showAddTask']

// or
  props: {
    text: String, 
    color: String,
    title: String,
    showAddTask: Boolean
  },
```

## 插值表达式 

插值表达式其值可来自自身文件的 `data` 属性，也可来自 `props` 属性

```js
<button :style="{ background: color }" class="btn">{{ text }}</button>


  props: {
    text: String
  },
  // or
  data: {
    text: 'hehe'
  }
```

还有两个特殊的插值表达式

- `v-text` 用于显示字符串：

```js
<p v-text="data"></p>

data: { text: '我是要显示的信息' }
```

- `v-html` 用于显示带HTML的文本：
```js
<div v-html="data"></div>

data: `<a>1111111111</a>`
```

  - 实际上，`v-html` 编译后长这样：
  ```html
  <div>
    <a>1111111111</a>
  </div>
  ```

## 数值绑定 `v-bind:` 或简写 `:` 
用于绑定样式：

```html
<button v-bind:style="{ background: color }" class="btn">x</button>
<!-- 或简写为 -->
<button :style="{ background: color }" class="btn">x</button>
```

## 事件绑定 `v-on:` 或简写 `@`

```js
<button @click="onClick()" >{{ text }}</button>

// 需在 `methods` 里定义绑定事件：
methods: {
  onClick() {
    // ...
  }
}
```

还有动态参数的缩写(v2.6.0+)，类似属性选择器，`eventName` 方法名可动态传入，摘自官网：
```html
<a @[eventName]="doSomething"> ... </a>
```


## 循环 `v-for`

  - 基础打法——用于数组，这里用在了组件 `<Task :task="task" />` 上， 必须绑定`key` 值， 和 `React` 一样，由于虚拟DOM Diff算法的原因，key不能用数组下标，必须是不重复的字符串：
  ```html
    <div v-for="task in tasks" :key="task.id">
      <Task :task="task" />
    </div>
  ```

- 还可用于对象obj：
  ```js
  <ul id="v-for-object" class="demo">
    <li v-for="value in object">
      {{ value }}
    </li>
  </ul>

  data: {
    object: {
      title: 'How to do lists in Vue',
      author: 'Jane Doe',
      publishedAt: '2016-04-10'
    }
  }
  ```

  显示为：
  > + How to do lists in Vue
  > + Jane Doe
  > + 2016-04-10

  obj也可以用键值对解构显示，他还有第三个参数 `index`
  ```js
  <div v-for="(value, name, index) in object">
    {{ index }}. {{ name }}: {{ value }}
  </div>
  ```

  显示为：
  > + 0.title:How to do lists in Vue
  > + 1.name:Jane Doe
  > + 2.value:2016-04-10

- 你也可以用 of 替代 in 作为分隔符，因为它更接近 JavaScript 迭代器的语法：
  ```html
  <div v-for="item of items"></div>
  ```

## 事件派发 `$emit` 与监听 `:`

项目中我们最常用的数据流 子 ——> 父 的方式就是 `$emit` ——> `@` ，如：

- 发射处，将id也一起发出， `$emit('函数名', 发射的参数)`，其中，发射的参数可以如下，一个简单的id
  ```html
    <div v-for="task of tasks" :key="task.id">
      <Task :task="task" @delete-task="$emit('delete-task', task.id)" />
    </div>
    <script>
  ```
  script 里也要注册该事件：
  ```js
    emits: ['delete-task',],
  ```


- `@` 接收该 emit 出来的事件，并命名：
  ```html
    <Task :task="task" @delete-task="$emit('delete-task', task.id)" />
  ```

  在 script 里绑定 `deleteTask(id){}` 事件，要注意，这里的 `id` 即为 `$emit` 传过来的第二个参数
  ```js
  methods: {
    deleteTask(id) {
      if (confirm('r u sure?')) this.tasks = this.tasks.filter( task => task.id !== id )
    },

    ...
  }
  ```

## 双向数据绑定 `v-model`

- 这个是Vue框架的精华所在，抄袭Ng的，在输入处用 `v-model` 绑定：

  ```html
    <!-- form表单绑定 onSubmit 事件 -->
    <form class="add-form" @submit="onSubmit">
      <!-- 搜集输入 -->
      <input type="text" name="text" placeholder="Add Task"
      v-model="text" >

      <!-- 提交按钮 -->
      <input type="submit" value="Save Task" class="btn btn-block">
    </form>
  ```
 
- script 里绑定 `onSubmit(e)` 事件

  ```js
    onSubmit(e) {
      e.preventDefault()
      if (!this.text) { 
        alert('Pls add a task') 
        return
      }
      
      const newTask = {
        id: Math.floor(Math.random() * 100000),
        text: this.text,
        day: this.day,
        reminder: this.reminder
      }

      this.$emit('add-task', newTask)
      this.text = ''
      this.day = ''
      this.reminder = false
    }
  ```