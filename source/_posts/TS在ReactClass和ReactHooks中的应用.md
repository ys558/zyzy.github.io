---
title: TS在ReactClass和ReactHooks中的应用
date: 2021-05-04 01:15:25
tags:
---


对于现行的ES规范，已经具有很好的规范性与可读性，type script虽当下大行其道，其实我觉得意义不算很大，ts最重要的功能就是类型检查，包括 interface, type 等也是一种类型检查，但一不小心就会写成 any script。   
鉴于将来的项目或者现行的项目可能会用到，这里结合 React Hooks 写个 tsx 版的 [demo](https://github.com/ys558/tech-blog-code/tree/master/2021/06-0328-ts-react)

<!-- more -->

## 安装

### 新项目，只需带`--template typescript`参数即可：

```shell
create-react-app <app name> --template typescript
#or 
yarn create-react-app <app name> --template typescript
```

### 老项目，在原有JSX项目`create-react-app`基础上增加typescript内容：

```shell
npm install --save typescript @types/node @types/react @types/react-dom @types/jest
 
# or
 
yarn add typescript @types/node @types/react @types/react-dom @types/jest
```

### 安装一个新的库，需配置 `*.d.ts` 文件：
外部`npm install`一个库时，需要单独创建一个`.d.ts`结尾的文件, 否则报错, 如：  
`touch lib.d.ts`:
```tsx
declare module 'lodash'
```
## 初学者的编写利器 —— `vs code` 中的提示  

可以利用好 `vs code` 中自带的提示，例如下面的提示，下面的例子会截图此类提示框：   
![vs code 提示](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.22/articles/TS在ReactClass和ReactHooks中的应用/0.png)

## 最外层 **index.tsx**

```tsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

// 全局变量，如有：
declare global {
  interface Window {
    API: any,
    _store: any
  }
}

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  // HTMLElement 接口：
  document.getElementById('root') as HTMLElement
);
```

## 组件

### Function 组件
```tsx
const App: React.FC = () => null
```

vs code 里的提示：

![vs code 提示](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.22/articles/TS在ReactClass和ReactHooks中的应用/3.png)

### Class 组件
```tsx
// 没啥好说的，写到烂了
export class ClassTS extends React.Component { }
```

## props

### Function 组件里的props   

```tsx
// 1，直接写：
export const TextField: React.FC<{ text: string }> = () => {
  return <div><input type="text"/></div>
}

// 2. 将 { text: string } 独立为接口类：
interface Props {
  text: string
}
export const TextField: React.FC<Props> = () => {
  return <div><input type="text"/></div>
}

```

其中，接口也可随意扩展及定义子接口，如：  

```tsx
interface Person {
  firstName: string
  lastName: string
}

interface Props {
  text: string
  ok?: boolean
  i?: number
  fn?: (bob: string) => string
  // 
  person: Person
}
```

### Class 组件里的 props

```tsx
// 1，直接写, 两个参数间用逗号或分号隔开均可：
export class ClassTS extends React.Component <{ text: string, age?: number }> { }
 
// 2. 将props的参数独立为接口类：
interface IProps {
  text: string
  age?: number
}

// 3. 顺带一提state，只有interface的写法：
interface IState {
  email: string
  name: string
}

export class ClassTS extends React.Component <IProps> {
  state: IState = {
    name: '',
    email: ''
  }
}
```

## 组件里的函数

React特有的类型，React.FormEvent<HTMLInputElement>，虚拟DOM把所有原生函数都撸了个遍:

![vs code 提示](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.22/articles/TS在ReactClass和ReactHooks中的应用/4.gif)

```tsx
  handleChange = ( e : React.FormEvent<HTMLInputElement> ) => {
    const { name, value } : any = e.target
    this.setState({ [name]: value })
  }
```


## 函数组件的 hooks

### ***useState***
```tsx
// 或用 | ，不用 ||
const [count, setCount] = useState < string | null | { text: string } >({ text: 'hello' })

// 或 独立为接口：
interface TextNode {
  text: string
}
const [count, setCount] = useState < number | TextNode >(5)
```

### ***useRef***

```tsx
// 因为ref的目标是一个input框，所以用 <HTMLInputElement>
// 如果打上ref的是<div></div>, 则用 <HTMLDivElement>
const inputRef = useRef<HTMLInputElement>(null)

```
另外，handleChange
```tsx
// 如果是Props传过来，并且定义了interface接口，则要注明：
interface Props {
  handleChange: ( event: React.ChangeEvent<HTMLInputElement> ) => void
  // 或者如下定义为any，但不建议，因失去了检查的作用：
  // handleChange: ( event: any ) => void
}
```

### ***useReducer***
```tsx
interface Todos {
  text: string
  complete: boolean
}

// type State = Array<Todos>
// 或下面这种，更形象化：
type State = Todos[]
type Actions = { type: 'add', text: string } | { type:'remove', idx: number }

const TodoReducer = ( state: State, action: Actions ) => {
  switch (action.type) {
    case 'add':
      return [...state, { text: action.text, complete: false }]
    case 'remove':
      return state.filter((_, i) => action.idx !== i)
    default:
      return state;
  } 
}

const [todos, dispatch] = useReducer(TodoReducer, [])
```

## hooks 的 render props

将 hooks 函数 `setXXX` 作为参数传参，这种props是react独有的，`React.Dispatch<React.SetStateAction<number>>` 

```tsx
interface Props {
  children:(
    count: number,
    // 将`setCount`作为props进行传参：
    setCount: React.Dispatch<React.SetStateAction<number>>
    // 返回值为React独有的：JSX.Element
  ) => JSX.Element | null
}

export const Counter: React.FC<Props> = ({ children }) => {
  const [count, setCount] = useState(0)
  return <div>
    {children(count, setCount)}
  </div>
}
```

另一种，原生的函数在虚拟DOM里的函数：

写的时候 vs code 里的提示：   
![vs code 提示](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.22/articles/TS在ReactClass和ReactHooks中的应用/1.png)

![vs code 提示](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.22/articles/TS在ReactClass和ReactHooks中的应用/2.png)