---
title: 无限滚动和懒加载通过IntersectionObserver和React_Hooks实现
date: 2021-04-23 09:05:33
tags: 
    - React
    - React Hooks
    - IntersectionObserver
---

用React Hooks做项目有一段时间了，炒鸡喜欢，这里结合 `IntersectionObserver` 原生js api及各种`hooks`，做个简单的无限滚动+懒加载页面，关于 `IntersectionObserver`， 阮大神的[这篇](http://www.ruanyifeng.com/blog/2016/11/intersectionobserver_api.html)博客文章里有非常详尽的描述  
（p.s. data用到数据接口需科学上网）

<!-- more -->

## 基础架构

- 用 `create-react-app` 构建项目后装 `axios`，   
`App.js`修改如下，页面呈现如下简单效果，即在输入框输入某一书籍关键字时，会出现对应所有的书籍列表：

    ```jsx
    const App = () => {
      return <div>
        <input type="text"/>
        <div>Title</div>
        <div>Title</div>
        <div>Title</div>
        <div>Title</div>
        <div>Loading...</div>
        <div>Error</div>
      </div>
    }
    ```

- 自定义 `custom hooks` 作为异步请求数据：根据hooks习惯，名字用use开头：
    ```js
    import { useState, useRef, useCallback } from 'react'

    /**
     * @params
     * input 输入框输入需要查询的字段
     * pageNum 页码，如果页面一页展示不玩，会持续添加pageNum
    */
    export const useBookSearch = (query, pageNum) => {
      useEffect(() => {
        axios({
          method: 'GET',
          url: 'http://openlibrary.org/search.json',
          params: { q: query, page: pageNum },
        }).then(res => {
          // 远程获取数据，暂时控制台打印：
          console.log(res.data)
        })
      }, [query, pageNum])
      return null
    }
    ``` 

- 在 `App.js` 里引入自定义hooks
  ```js
  const App = () => {
    useBookSearch(query, pageNum)
    return <div>
      <input type="text"/>
      <div>Title</div>
      <div>Title</div>
      <div>Title</div>
      <div>Title</div>
      <div>Loading...</div>
      <div>Error</div>
    </div>
  }
  ```

## 实现动态效果

- 继续改造 `App.js` 
  - `query` 和 `pageNum`两个参数需要在本页进行变动，于是加入 `useState` 钩子

  ```js
  import { useState, useRef, useCallback } from 'react'

  const App = () => {
    const [query, setInput] = useState('')
    const [pageNum, setPageNum] = useState(1)

    const handleSearch = e => {
      setInput(e.target.value)
      // 重新输入查询字段后，页数重新设置为1：
      pageNum(1)
    }
    
    useBookSearch(query, pageNum)
    return <div>
      <input type="text" onChange={handleSearch}/>
      <div>Title</div>
      <div>Title</div>
      <div>Title</div>
      <div>Title</div>
      <div>Loading...</div>
      <div>Error</div>
    </div>
  }
  ```

  - `query` 通过输入框改变，传递给 `useBookSearch(query, pageNum)`，控制台能看到请求回来的数据，如下演示，但这种请求是实时的，即每输入一个字符就会请求一次，会浪费性能，必须优化   
  ![请求回来的数据](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/无限滚动和懒加载通过IntersectionObserver和React_Hooks实现/0.gif)

- `useBookSearch` 优化请求性能： `axios` 有个参数 `CancelToken`，可取消重复请求，做如下改动：
  ```js
    useEffect(() => {
      setLoading(true)
      setError(false)
      // 定义取消变量
      let cancel
      axios({
        method: 'GET',
        url: 'http://openlibrary.org/search.json',
        params: { q: query, page: pageNum },
        // 执行取消变量：
        cancelToken: new axios.CancelToken(c => cancel = c)
      }).then(res => 
        console.log(res.data)
      ).catch(e => {
        if (axios.isCancel(e)) return
        setError(true)
      })

      // 最后，useEffect里必须删除new出来的实例
      return () => cancel()
    }, [query, pageNum])
  ```
  改造完成后，继续重新尝试：   
  ![请求回来的数据](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/无限滚动和懒加载通过IntersectionObserver和React_Hooks实现/1.gif)

## 页面UI实现：
  - 针对界面实现的丰富，我们对 `useBookSearch` 函数继续做改造，分别增加以下状态：
    - `loading` : 读取
    - `error` : 获取数据错误
    - `hasMore` : 判断是否还有数据
    - `books` : 获取到的所有书籍的数据
    ```js
    const useBookSearch = (query, pageNum) => {
      const [loading, setLoading] = useState(true)
      const [error, setError] = useState([])
      const [books, setBooks] = useState([])
      const [hasMore, setHasMore] = useState(false)

      useEffect(() => {
        setLoading(true)
        setError(false)
        let cancel
        axios({
          method: 'GET',
          url: 'http://openlibrary.org/search.json',
          params: { q: query, page: pageNum },
          cancelToken: new axios.CancelToken(c => cancel = c)
        })
        .then(res => {
          // 将后面请求回来books加入现有数组中：用Set去重：
          setBooks(prevBooks => {
            return [...new Set([...prevBooks, ...res.data.docs.map(b => b.title)])]
          })
          // 每次只能显示满屏的数据，如果请求的数据没显示完
          setHasMore(res.data.docs.length > 0)
          // 则把loading显示为false
          setLoading(false)
        })
        .catch(e => {
          if (axios.isCancel(e)) return
          // 设置错误：
          setError(true)
        })

        return () => cancel()
      }, [query, pageNum])

      // 每次改变
      useEffect(() => {
        setBooks([])
      }, [query]) 

      // 将上面的状态一概导出，给App函数用：
      return { loading, error, books, hasMore }
    }
    ```

    - 改造主页面的 `App` 组件:
    ```jsx
    import { useState } from 'react'

    const App = () => {
      const [query, setInput] = useState('')
      const [pageNum, setPageNum] = useState(1)
      // 将状态从useBookNameSearch解构出来：
      const { books, hasMore, loading, error } = useBookSearch(query, setPageNum)

      const handleSearch = e => {
        setInput(e.target.value)
        pageNum(1)
      }

      return (
        <>
          <input type="text" onChange={handleSearch}/>
          { books.map((book, index) => <div key={book}>{book}</div>)}
          <div>{ loading && 'Loading...' }</div>
          <div>{ error && 'Error' }</div>
        </>
      );
    }

    export default App;
    ```

    - 最后，继续做翻页的部分，我们要实现的效果是，滚动到页面最下面，则自动触发loading和翻页，在 `App` 里用到`useRef` 钩子，检测到滚动到最后一行
    - 用到 `useCallback` 钩子，具有缓存作用，依赖改变才重新渲染。
    - 用到 `IntersectionObserver` api, 第一个参数为回调函数，其 `entries` 为检测到的触发的点
    ```js
      const last = useRef()
      const lastRecordRef = useCallback( node => {
        if (loading) return
        if (observer.current) observer.current.disconnect()
        observer.current = new IntersectionObserver(entries => {
          // entries[0].isIntersecting检测可见页面内是否需要调用新的数据：
          if( entries[0].isIntersecting && hasMore ) {
            setPageNum(prevPageNumber => prevPageNumber + 1)
          }
        })
        // 检测是否最后一个节点node
        if (node) observer.current.observe(node)
      }, [loading, hasMore]
    ```
    - 控制台打印了`entries[0].isIntersecting`，当出现`Loading`状态时，该值为`true`, 单当完成加载后为`false`，如下：
    ![请求回来的数据](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/无限滚动和懒加载通过IntersectionObserver和React_Hooks实现/2.gif)

## 贴出完整代码：
  - `App.js`
    ```js
    import useBookSearch from './useBookSearch'
    import { useState, useRef, useCallback } from 'react'

    function App() {
      const [query, setQuery] = useState('')
      const [pageNumber, setPageNumber] = useState(1)
      const { books, hasMore, loading, error } = useBookSearch(query, pageNumber)


      const observer = useRef()
      const lastBookElementRef = useCallback(node => {
        if (loading) return
        if (observer.current) observer.current.disconnect()
        observer.current = new IntersectionObserver(entries => {
          // detected is it the last one:
          if( entries[0].isIntersecting && hasMore ) {
            setPageNumber(prevPageNumber => prevPageNumber + 1)
          }
        })
        if (node) observer.current.observe(node)
      }, [loading, hasMore])

      const handleSearch = e => {
        setQuery(e.target.value)
        setPageNumber(1)
      }

      return (
        <>
          <input type="text" onChange={handleSearch}/>
          { books.map( (book, index) => {
            if (books.length === index + 1) {
              return <div ref={lastBookElementRef} key={book}>{book}</div>
            }else{
              return <div key={book}>{book}</div>
            }
          })}
          <div>{ loading && 'Loading...' }</div>
          <div>{ error && 'Error' }</div>
        </>
      );
    }

    export default App;
    ```

- `useBookSearch` 独立成一个文件

  ```js
  import { useEffect, useState } from 'react'
  import axios from 'axios'

  const useBookSearch = (query, pageNumber) => {
    const [loading, setLoading] = useState(true)
    const [error, setError] = useState([])
    const [books, setBooks] = useState([])
    const [hasMore, setHasMore] = useState(false)

    useEffect(() => {
      setLoading(true)
      setError(false)
      let cancel
      axios({
        method: 'GET',
        url: 'http://openlibrary.org/search.json',
        params: { q: query, page: pageNumber },
        cancelToken: new axios.CancelToken(c => cancel = c)
      })
      .then(res => {
        setBooks(prevBooks => {
          return [...new Set([...prevBooks, ...res.data.docs.map(b => b.title)])]
        })
        setHasMore(res.data.docs.length > 0)
        setLoading(false)
      })
      .catch(e => {
        if (axios.isCancel(e)) return
        setError(true)
      })

      return () => cancel()
    }, [query, pageNumber])

    useEffect(() => {
      setBooks([])
    }, [query]) 

    return { loading, error, books, hasMore }
  }

  export default useBookSearch

  ```