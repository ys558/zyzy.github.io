---
title: RenderProps和高阶组件的反向继承
date: 2021-06-19 09:53:39
tags:
---
这两种方法是HOC的延申，具体业务中会有相应的使用场景，下面简要的说明一下
<!-- more -->

## Render Props
这是一种简单的技术，[官网](https://zh-hans.reactjs.org/docs/render-props.html)也有解释，简单理解就是将一个组件里的属性`attribute`作为jsx语法的实现。如以下代码

```js
// 父组件
export class RenderPropsDemo extends Component {
  render(){
    // 1. render 作为组件 <ToggleRenderProps/> 的属性
    // 3. 反向在render的回调函数里解构出子组件的属性出来用
    return <ToggleRenderProps render={({on, toggle, str}) => <div>
        { on && <h1>Hey zidea</h1> }
        { on && <h2>{str}</h2> }
        <button onClick={toggle}>隐藏/显示</button>
      </div>
    }/>
  }
}

// 子组件：
class ToggleRenderProps extends Component {
  state = { on:false }
  toggle = () => this.setState({ on:!this.state.on })

  render() {
    // 2. 在子组件设置render里面的属性：
    const { render } = this.props;
    return render({
        str: 'hehe',
        on: this.state.on,
        toggle: this.toggle,
      })
  }
}
```

从上面的例子可以看出，子组件 `ToggleRenderProps` 仅负责单一的传值和控制值和状态的功能，类似回调函数，把该控制的值回传给父组件 `RenderPropsDemo` 去使用。     

- 当然属性 `render` 这词不是固定的

> [任何被用于告知组件需要渲染什么内容的函数 prop 在技术上都可以被称为 “render prop”.](https://medium.com/componentdidblog/use-a-render-prop-50de598f11ce)

所以，你可以给属性取任意名字，例如 data

```js
// 父组件
export class RenderPropsDemo extends Component {
  render(){
    return <ToggleRenderProps data={({on, toggle, str}) => <>
      ...
      </>}/>}
}

// 子组件：
class ToggleRenderProps extends Component {
  ...
  render() {
    return this.props.data({
      ...
    })
  }
}
```

- 父组件还可以写为元素内部的模式，如

```js
export class RenderPropsDemo extends Component {
  render(){
    return <ToggleRenderProps>
     {data={({on, toggle, str}) => <>
      ...
      </>}
    </ToggleRenderProps>
    }
}
```

而子组件调用时，就是另一种概念了，可利用 `this.props.children` 的react自带api进行回传

```js
class ToggleRenderProps extends Component {
  state = { on:false }
  toggle = () => this.setState({ on:!this.state.on })

  render() {
    return this.props.children({
        str: 'hehe',
        on: this.state.on,
        toggle: this.toggle,
      })
  }
}
```

以上两种情况，一种在属性上定义的，类似具名函数，而包裹在元素内部，利用 `children` 进行调用的，类似匿名函数

`Rener Props`的应用场景就很广泛了，很多库例如路由库[`React Router`](https://reactrouter.com/web/api/Route/render-func)，表单校验[`formik`](https://github.com/formium/formik)都用了该方法。



## 正向代理HOC
在说反向代理之前，我们先说说正向代理，正向代理其实就是高阶组件 HOC，我们看看两者的区别：   
普通HOC，即正向代理，简言之就是把一个组件外面包裹了一层新的组件，使其成为一个新组件
关于React的HOC和反向代理也可以参考这篇[文章](https://medium.com/@franleplant/react-higher-order-components-in-depth-cf9032ee6c3e)
```js
// 父组件
class AAA extends React.Component {
  constructor(props) {
    super(props)
    this.state = { num: 111 }
  }

  render() {
    return <div num={this.state.num}>父组件AAA：{ this.state.num }</div>
  }
}

// 子组件
// 普通HOC，AAA => props 这部分，代表了 class AAA及其里面的props属性
// 相当于把原来的 AAA 提高了一层状态变成了 HOC
const HOC = function (AAA) {
  return class extends React.Component {
    render () {
      return <div style={{border: 'solid 1px black'}}>
        <AAA {...this.props}/>
      </div>
    }
  }
}
```

### 子组件也可改写为函数式写法：
```js
// 子组件
const HOC = AAA => props => {
  return <div style={{border: 'solid 1px black'}}>
    <AAA {...props}/>
  </div>
}

export const Foo = HOC(AAA)
```

### 子组件改为装饰器写法，可以省略一步命名的操作，但实际项目中还需要在babel中单独另行配制装饰器语法
```js
@InheritanceInversionHOC
export const HOC = function (AAA) {
  return class extends React.Component {
    render () {
      return <div style={{border: 'solid 1px black'}}>
        <AAA {...this.props}/>
      </div>
    }
  }
}
```
## 反向代理 Inheritance Inversion HOC

### 相对于正向代理，则是继承组件自己本身进行重写
将上面的子组件进行重写

```js
// 父组件，即原来的组件
class AAA extends React.Component {
  constructor(props) {
    super(props)
    this.state = { num: 111 }
  }

  componentDidMount() { console.log("child component Did Mount") }
  clickComponent = () =>  console.log('clickComponent func')

  render() {
    return <div num={this.state.num}>父组件AAA：{ this.state.num }</div>
  }
}


// 子组件，即反向继承后的组件
const IIHOC = AAA => class extends AAA {
  constructor(props) {
    super(props)
    this.state = { num: 222 }
  }

  componentDidMount() {
    console.log('IIHOC componentDidMount')
    this.clickComponent() // 直接调用 this.clickComponent() 即可，无需通过props进行传参
  }

  render(){
    return <div>
        <button onClick={this.clickComponent}>IIHoc点击见控制台，AAA组件的函数clickComponent在控制台执行了</button>
        {/* 下面的<AAA />会显示父组件的num值， 111 */}
        <div><AAA /></div>

        {/* 下面的 {super.render()} 为渲染劫持， 会显示本组件里的值，222 */}
        <h1>{super.render()}</h1> 
      </div>
  }
}

export const InheritanceInversionHOC = IIHOC(AAA)
```

### 反向代理的解析及实际作用
- `const IIHOC = AAA => class extends AAA {` 中，`extends AAA`是继承自己 `AAA =>`本身
- 当然，`AAA =>` 的名字`AAA`是可以任意替换的，如`Comp`
- 生命周期函数 `componentDidMount() { }`，因为是继承于自身，所以无论父子组件的生命周期函数均会执行
- 子组件里使用父组件的`this.clickComponent()`，**直接使用即可，无需通过 `props` 进行传参**
![生命周期执行及父组件函数调用](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.39/articles/RenderProps和高阶组件的反向继承/00.png)
- 反向代理这种做法乍一看好像没有具体用途，但实际操作中，我们可用来做一个原来的**复杂页面的多人同时开发**，在一些在原有代码基础上开发，不能改变原来旧的代码，则可用反向继承实现二次渲染进行开发，能使新的功能独立成一个模块，方便维护