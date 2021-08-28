---
title: 通过ReactHooks直接操作api神库：ReactQuery
tags:
  - React Query
cover: https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.23/articles/通过ReactHooks直接操作api神库：ReactQuery/cover.png
date: 2021-04-26 10:17:07
---


[React Query](https://react-query.tanstack.com/) 是一个以react hooks为基础进行异步获取，缓存和更改数据的库，[react query仓库](https://github.com/tannerlinsley/react-query)文档原文描述：

> Hooks for fetching, caching and updating asynchronous data in React

换言之，是一个异步处理数据的库。这么重要而且好用的库当然要收入囊中，把他学起来，本篇着重介绍整个项目的构建过程。   
p.s 该篇的后端服务用[这篇](https://zyzy.info/2021/04/25/SwaggerUI%E9%80%9A%E8%BF%87swagger-jsdoc%E5%9C%A8node%E9%87%8C%E7%9A%84%E5%BA%94%E7%94%A8%E5%92%8C%E9%85%8D%E7%BD%AE/) 配置的服务端

<!-- more -->



## 前端基础服务搭建

初始化项目:
```shell
npx create-react-app client && cd client
```

安装依赖并运行:    
其中 `react-hook-from @rebass/forms @rebass/preset styled-components react-loader-spinner` 均为样式组件
```shell
yarn add react-query react-router-dom react-hook-from @rebass/forms @rebass/preset styled-components react-loader-spinner
yarn start
```

修改`src/client.js` 文件如下: [QueryClientProvider文档](https://react-query.tanstack.com/reference/QueryClientProvider#_top)
```js
import React from 'react'
import ReactDOM from 'react-dom'
import "react-loader-spinner/dist/loader/css/react-spinner-loader.css"
import App from './App'
import { BrowserRouter } from "react-router-dom"
import { ThemeProvider } from "styled-components"
import preset from "@rebass/preset"

// 核心部分: react-query
import { QueryClientProvider, QueryClient } from "react-query"
// 实例化queryClient
const queryClient = new QueryClient()

ReactDOM.render(
  {/* React严格模式，官方文档： https://react.html.cn/docs/strict-mode.html */}
  <React.StrictMode>
    {/* QueryClientProvider 可作为全局对象注入，类似 react-redux */}
    <QueryClientProvider client={queryClient}>
      {/* 样式组件： */}
      <ThemeProvider theme={preset}>
        <BrowserRouter>
          <App />
        </BrowserRouter>
      </ThemeProvider>
    </QueryClientProvider>
  </React.StrictMode>,
  document.getElementById('root')
)
```

## 基础路由模块及基础架构：

`src/App.js` 利用 `'react-router-dom'` 作为路由的管理器，也是react常见的结构模式：   

修改`src/App.js`文件如下: 
```js
import { Switch, Route } from 'react-router-dom'
import { BooksList } from './BookList'
import { CreateBook } from './CreateBook'
import { UpdateBook } from './UpdateBook'

function App() {
  return <>
      <Switch>
        <Route path='/update-book/:id'><UpdateBook/></Route>
        <Route path='/create-book'><CreateBook /></Route>
        <Route path='/'><BooksList /></Route>
      </Switch>
    </>
}

export default App;
```
### 获取所有图书

创建 `src/BookList/index.js` 和 `src/BookList/BookList.jsx` ：   

`src/BookList/index.js`:
```js
export * from './BooksList'
```

`src/BookList/BookList.jsx`:
```jsx
export const BooksList = () => {
  return null
}
``` 
### 创建图书   

创建 `src/BookList/index.js` 和 `src/BookList/BookList.jsx` :    

`src/CreateBook/index.js`:
```js
export * from './CreateBook'
```

`src/CreateBook/CreateBook.jsx`:
```jsx
export const CreateBook = () => {
  return null
}
```  
### 更新图书   
创建 `src/BookList/index.js` 和 `src/BookList/BookList.jsx` :   
 
`src/UpdateBook/index.js`:
```js
export * from './UpdateBook'
```

`src/UpdateBook/UpdateBook.jsx`:
```jsx
export const UpdateBook = () => {
  return null
}
```

### 基本结构：   

```shell
  src
    |_App.js
    |_BookList
    |  |_index.js
    |  |_BookList.jsx
    |_CreateBook
    |  |_CreateBook.jsx
    |  |_index.js
    |_UpdateBook
       |_UpdateBook.jsx
       |_index.js
```

### 添加导航栏样式   

`src/shared/NavBar.jsx`:     
```js
import { Flex, Box, Link as StyledLink, Image } from 'rebass/styled-components'
import { Link } from 'react-router-dom'
import { Container } from './Container'
import logo from './logo.svg'

export const NavBar = () => {
  return <Flex bg="black" color="white" justifyContent="center">
      <Container>
        <Flex px={2} width='100%' alignItems='center'>
          <Image size={20} src={logo} />
          <Link component={StyledLink} variant='nav' to='/'>
            React Query CRUD
          </Link>
          <Box mx="auto"/>
          <Link component={StyledLink} variant='nav' to='/create-book'>
            + Add new book
          </Link>
        </Flex>
      </Container>
    </Flex>
}

```

`src/shared/Container.jsx` :   
```js
import { Box } from 'rebass/styled-components'

export const Container = ({children}) => {
  return <Box sx={{ width: "100%", maxWidth: 1024, mx: "auto" }}>
      {children}
    </Box>
}
```

返回 `src/App.js` 把导航栏加上:  
```js
import { NavBar } from './shared/NavBar'

<NavBar/>
```

## 创建 api.js 接口文件获取并数据：

### 配制文件 `.env`
路径必须在根目录     
```
REACT_APP_SERVER = http://localhost:4800
```

### `src/api.js`   
```js
export const getAllBooks = async () => {
  const response = await fetch(`${process.env.REACT_APP_SERVER}/books`)
  if (!response.ok) throw new Error('something wrong')
  return response.json()
}
```

### 【 [**`useQuery`**](https://react-query.tanstack.com/reference/useQuery) 】 应用 —— `src/BookList.jsx` 中查询所有图书
```js
import { useQuery } from 'react-query'
import { Flex } from 'rebass'
import { getAllBooks } from '../api'
import { Container } from '../shared/Container'
import Loader from 'react-loader-spinner'

export const BooksList = () => {
  // useQuery已经准备好了各种状态，直接调用即可：
  const { data, error, isLoading, isError } = useQuery('books', getAllBooks)

  if (isLoading) return <Container>
    <Flex>
      <Loader type='ThreeDots' color='#ccc' height={30} />
    </Flex>
  </Container>

  if (isError) return <span> Error: {error.message} </span>

  return <Container>
    <Flex flexDirection='column' alignItems='center'>
    { data.map(({author, title, id}) => <div key={id}>
          {author} - {title}
        </div>
      )}
    </Flex>
  </Container>
}
```

此时界面可看到效果：   
![1](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/通过ReactHooks直接操作api神库：ReactQuery/01.png)

## 每条记录单独抽离做模块 `src\BookList\BookItem.jsx`   
```js
import { Flex, Text, Button, Link as StyledLink } from 'rebass/styled-components'
import { Link } from 'react-router-dom'

export const BookItem = ({ author, title, id }) => {

  return <Flex p={3} width="100%" alignItems='center'>
    <Link component={StyledLink} to={`/update-book/${id}`} mr="auto">
      { title }
    </Link>
    <Text>{author}</Text>
    <Button ml="3">
      remove
    </Button>
  </Flex>
}
```
## 从 `src\BookList\BookList.jsx` 导入并取代具体记录的位置   
```js
import { BookItem } from './BookItem'

data.map(({author, title, id}) => 
  <BookItem author={author} title={title} id={id} key={id}/>
)
```

看看页面的效果：   

![2](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/通过ReactHooks直接操作api神库：ReactQuery/02.png)

## 【 [**useMutation**](https://react-query.tanstack.com/reference/useMutation) 】和【 [**queryClient**](https://react-query.tanstack.com/reference/useQueryClient) 】 的应用 —— 删除图书   

### 在 `src\api.js` 增加删除接口：
```js
export const removeBook = async id => {
  const response = await fetch(`${process.env.REACT_APP_API_SERVER}/books/${id}`, {
    method: 'DELETE'
  })
  if (!response.ok) throw new Error(response.json().message)
  return true
}
```

### 为删除图书修改界面
并在 `src\BookList\BookList.jsx` 里增加相应的 [**useMutation**](https://react-query.tanstack.com/reference/useMutation) 和 [**queryClient**](https://react-query.tanstack.com/reference/useQueryClient) 的写法：   

```js
import { removeBook } from '../api'

const queryClient = useQueryClient()
// 将 removeBook 直接传入 useMutation
const { mutateAsync, isLoading } = useMutation(removeBook)

const remove = async () => {
  await mutateAsync(id)
  queryClient.invalidateQueries('books')
}

<Button ml="3" onClick={remove}>
  { isLoading ? <Loader type='ThreeDots' color='#fff' height={10} />: 'Remove' }
</Button>
```

**queryClient.invalidateQueries()** 用于清除缓存并刷新页面：

来自[中文官方文档](https://cangsdarm.github.io/react-query-web-i18n/guides&concepts/query-invalidation)的解释：

> 可以智能地将查询标记为过时的，并使之可用重新获取数据，

简言之，`queryClient.invalidateQueries('books')` 可清除旧 'books' 的显示缓存，并直接刷新最新的 'books' 接口数据，如果不加，页面就不会刷新，需要手动刷新


## 变更一本图书信息

### `src\api.js` 增加获取一本书的接口：   

```js
export const getBook = async ({ queryKey }) => {
  // useQuery 传过来的参数：
  const [_key, { id }] = queryKey
  const response = await fetch(`${process.env.REACT_APP_API_SERVER}/books/${id}`)
  if (!response.ok) throw new Error(response.json().message)

  return response.json()
}
```

### `src\api.js` 增加 `updateBook` 接口：   

```js
export const updateBook = async ({ id, ...data }) => {
  const response = await  fetch(`${process.env.REACT_APP_API_SERVER}/books/${id}`, {
    method: 'PUT',
    headers: {
      'Content-Type' : 'application/json'
    },
    body: JSON.stringify(data)
  })
  
  if (!response.ok) throw new Error(response.json().message)
  return response.json()
} 
```

### 新建 `src\shared\BookForm.jsx` 用做更新图书的输入界面：

```js
import { Box, Button } from 'rebass/styled-components'
import { Label, Input } from '@rebass/forms'
import { useForm } from 'react-hook-form'
import Loader from 'react-loader-spinner'

export const BookForm = ({ defaultValues, onFormSubmit, isLoading }) => {
  const { register, handleSubmit } = useForm({ defaultValues })
  const onSubmit = handleSubmit( data => {
    onFormSubmit(data)
  })

  return <form onSubmit={onSubmit}>
    <Box sx={{ marginBottom : 3 }}>
      <Label htmlFor="title">Title</Label>
      <Input ref={register} id='title' name='title' type='text' />
    </Box>
    <Box sx={{ marginBottom : 3 }}>
      <Label htmlFor='author'>Author</Label>
      <Input ref={register} id='author' name='author' type='text' />
    </Box>
    <Button>
      { isLoading ? <Loader type='ThreeDots' color='#fff' height={10} /> : 'Submit' }
    </Button>
  </form>
}
```

`src\UpdateBook\UpdateBook.jsx` 更改如下：   

```js
import Loader from "react-loader-spinner"
import { useMutation, useQuery } from "react-query"
import { useHistory, useParams } from "react-router-dom"
import { Box, Flex, Heading } from "rebass/styled-components"
import { getBook, updateBook } from "../api"
import { BookForm, Container } from "../shared"

export const UpdateBook = () => {
  const { id } = useParams()
  const history = useHistory()
  // 获取单本书的接口：
  const { data, error, isLoading, isError } = useQuery(['book', {id}], getBook)

  // 更改单本书 useMutation
  const { mutateAsync, isLoading: isMutating } = useMutation(updateBook)
  // 将数据传给后端：
  const onFormSubmit = async data => {
    await mutateAsync({ ...data, id })
    history.push('/')
  }

  if ( isLoading ) return <Container>
      <Flex>
        <Loader type='ThreeDots' color='#ccc' height={30} />
      </Flex>
    </Container>

  if ( isError ) return <Container>
    <Flex py='5' justifyContent='center'>
      Error: {error.message}
    </Flex>
  </Container>

  return <Container>
    <Box sx={{ py: 3 }}>
      <Heading sx={{ marginBottom: 3 }}>Update Book</Heading>
      <BookForm defaultValues={data} onFormSubmit={onFormSubmit} isLoading={isMutating} />
    </Box>
  </Container>
}
```

新建 `src\shared\index.js` 将所有共用组件导出：

```js
export * from './Container'
export * from './BookForm'
export * from './NavBar'
```

看看页面的效果：
![添加书籍](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/通过ReactHooks直接操作api神库：ReactQuery/03.gif)

## 新建一本图书

### `src\api.js` 里添加

```js
export const createBook = async (data) => {
  const response = await fetch(`${process.env.REACT_APP_API_SERVER}/books/`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  })

  if (!response.ok) throw new Error(response.json().message)

  return response.json()
}
```

### 更改创建图书界面 `src\CreateBook.jsx` 如下：

```js
import { useMutation } from "react-query"
import { useHistory } from "react-router-dom"
import { Box, Heading } from "rebass"
import { createBook } from "../api"
import { BookForm, Container, } from '../shared'

export const CreateBook = () => {
  const history = useHistory()

  const { mutateAsync, isLoading } = useMutation(createBook)
  const onFormSubmit = async data => {
    await mutateAsync(data)
    history.push('/')
  }

  return <Container>
    <Box sx={{ py: 3 }}>
      <Heading sx={{ marginBottom: 3 }}>Create New Book</Heading>
      <BookForm onFormSubmit={onFormSubmit} isLoading={isLoading} />
    </Box>
  </Container>
}
```

看看页面的效果：
![添加书籍](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/通过ReactHooks直接操作api神库：ReactQuery/04.gif)