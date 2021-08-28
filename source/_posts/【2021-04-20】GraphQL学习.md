---
title: GraphQL学习
date: 2021-04-20 10:15:27
tags:
    - GraphQL
cover: https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.23/articles/GraphQL学习/cover.png
---

去年自学了GraphQL，这里重新整理一下笔记。国内使用GraphQL的项目毕竟还不多，这里写个demo，做个小小的探索。   

GraphQL是facebook发明的一套API查询语言，支持多种语言，这里用js演示，[官网](https://graphql.org/)有详细的说明  
比起传统的后端API整合

<!-- more -->

> GraphQL为你的API数据提供了一种完整且易于理解的描述。（GraphQL provides a complete and understandable description of the data in your API）

以上是官网对GraphQL的解释，利用GraphQL可以按需取字段给前端利用，不必取到不需要的字段，见下图解释：  
![传统RestAPI效率低](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/GraphQL学习/0.0.png)

GraphQL则可以精确查找想要的字段，效率显著提高：
![GraphQL效率高](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/GraphQL学习/0.1.png)

下面，结合自己写的demo：https://github.com/ys558/tech-blog-code/tree/master/2020/04-graphql-learn 完成以下完成本篇博客：

## 一、HelloWorld
1. 将项目拉下来后，cd到对应目录下，用`npm install`或`yarn`安装依赖，运行`npm run hello` 或 `yarn hello`，先从 [01helloworld.js](https://github.com/ys558/tech-blog-code/blob/master/2020/04-graphql-learn/01helloworld.js) 文件跑起，浏览器运行`http://localhost:5000/`
左边输入框输入
```js
query {
  message
}
```
**注意，这种是GraphQL特有的查询语法**

点击运行，( 或快捷键 ctrl+enter ) 后，右边会显示：  
![hello world](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/GraphQL学习/1.2.png)

2. [01helloworld.js](https://github.com/ys558/tech-blog-code/blob/master/2020/04-graphql-learn/01helloworld.js)解析：  
- `graphqlHTTP` 为 `express-graphql` 库解构的实例
```js
const { graphqlHTTP } = require('express-graphql')
```
- GraphQL自带图形化界面且友好，，只需在实例 `graphqlHTTP` 中设置`graphiql: true`即可：
```js
app.use('/', graphqlHTTP({ schema, graphiql: true }))
```
- 在query实例中，`fields` 字段用于设置查询条件，是回调函数，
- `fields` 函数返回的东西用 `resolve` 函数接收其返回结果，接收结果也为回调函数
- 在接收结果中，可以规定其返回类型 `type`，其值为 [Scalar Type](https://graphql.org/graphql-js/type/#scalars) (标量类型)
```js
const schema = new GraphQLSchema({
  query: new GraphQLObjectType({
    name: 'helloWorld',
    fields: () => ({
      message: {
        type: GraphQLString,
        resolve: () => 'Hello World'
      }
    })
  })
})
```

## 二、基础操作
1. 运行`npm run ds2` 或 `yarn ds2`，浏览器跑 `http://localhost:5000/books`，
```js
{
  books {
    id
    name
  }
}
```
输入以上条件，**通过 books 字段查询其对应的 id, name** 后运行，可以查询出以下结果：
![hello world](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/GraphQL学习/2.png)

2. [02base.js](https://github.com/ys558/tech-blog-code/blob/master/2020/04-graphql-learn/02base.js) 解析：
- books查询出来的为一数组，其[标量](https://graphql.org/graphql-js/type/#scalars) 定义为 `GraphQLList`
```js
const RootQueryType = new GraphQLObjectType({
  name: 'Query',
  description: 'Root Query',
  fields: () => ({
    /* 第一个 books 是命名，也可叫 aaa，如果改 aaa 则
    在 query 条件改为：
    aaa {
     books {
       id
       name
      }
    } 
    */
    books: {
      // BookType as params pass to RootQueryType
      type: new GraphQLList(BookType),
      description: 'List of all books',
      // 这里的 books 对应实际的 books 数组，
      // 要查询数组 books里的东西：
      resolve: () => books
    }
  })
})
```
- `BookType` 独立为标量`GraphQLObjectType`的实例化，由于其为最后一层数据，无需向下深挖，所以`resolve` 方法不需要
- `GraphQLNonNull` 表示该字段为必须字段
```js
const BookType = new GraphQLObjectType({
  name: 'Book',
  description: 'this is represents a book written by an author',
  fields: () => ({
    id: { type: GraphQLNonNull(GraphQLInt) },
    name: { type: GraphQLNonNull(GraphQLString) },
  })
})
```

## 三、通过书籍查询对应作者（一对一关系查询）及作者查询对应的书籍信息（一对多查询）

1. 跑 [`03listQuery.js`](https://github.com/ys558/tech-blog-code/blob/master/2020/04-graphql-learn/03listQuery.js) 文件，输入
    ```js
    {
      authors {
        books {
          id
          name
        }
      }
    }
    ```
    可以得到结果：
    ![hello world](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/GraphQL学习/3.3.png)

2. 解析[`03listQuery.js`](https://github.com/ys558/tech-blog-code/blob/master/2020/04-graphql-learn/03listQuery.js) ：  
  - 可以按照上面的方法继续丰富查询，建立一个模拟数据 `author` ，内容如下：

    ```js
      const authors = [
        { id: 1, name: 'J. K. Rowling'},
        { id: 2, name: 'J. R. R. Tolkien'},
        { id: 3, name: 'Brent Weeks'},
      ]
    ```
  - 同样的，建立两个实例查询对应信息：`resolve` 部分对应相应的数据
    ```js
    {
      books: {
        type: new GraphQLList(BookType),
        description: 'List of all books',
        resolve: () => books
      },
      authors: {
        type: new GraphQLList(AuthorType),
        description: 'List of all Authors',
        resolve: () => authors
      }
    }
    ```

  - 根据上面的字段名字`books` 和 `authors` 创建对应的查询实例：
    - books实例里，`id`, `name`, `authorId` 均是 `books` 对象里的，可直接查出， `author`字段在 `author` 数组里
    - `books` 数据有关联的是`authorId`，用js的数组方法`.find()`去找
    - books <--> authors 是一对一的关系，`AuthorType`本身就是`GraphQLObjectType` 标量，可直接使用
    ```js
      const BookType = new GraphQLObjectType({
        name: 'Book',
        description: 'this is represents a book written by an author',
        fields: () => ({
          id: { type: GraphQLNonNull(GraphQLInt) },
          name: { type: GraphQLNonNull(GraphQLString) },
          authorId: { type: GraphQLNonNull(GraphQLInt)},
          author: {
            type: AuthorType,
            // query book by authorId
            resolve: book => {
              return authors.find( author => author.id === book.authorId )
            }
          }
        })
      })
    ```

    - authors实例也是同样道理
    - authors <--> books 是一对多的关系，`BookType`实例要装入`GraphQLList`标量中
    ```js
      const AuthorType = new GraphQLObjectType({
        name: 'Author',
        description: 'this is represents a author of a book',
        fields: () => ({
          id: { type: GraphQLNonNull(GraphQLInt) },
          name: { type: GraphQLNonNull(GraphQLString) },
          // query author by book's id
          books: {
            type: new GraphQLList(BookType),
            resolve: author => {
              return books.filter(book => book.authorId === author.id)
            }
          }
        })
      })
    ```

## 四、通过书籍id查询书本信息  
1. 运行[`04singleQuery.js`](https://github.com/ys558/tech-blog-code/blob/master/2020/04-graphql-learn/04singleQuery.js)，输入以下查询条件：

  ```js
    {
      book(id: 4) {
        name
        author {
          name
        }
      }
    }
  ```
  
  执行结果如下：  
  ![单个查询](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/GraphQL学习/4.png)   

  输入以下条件
  ```js
    {
      book(id: 4) {
        name
        author {
          name
        }
      }
    }
  ```
  
  执行结果如下：  
  ![单个查询](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/GraphQL学习/4.png)   

2. [`04singleQuery.js`](https://github.com/ys558/tech-blog-code/blob/master/2020/04-graphql-learn/04singleQuery.js)解析：

  - 从以上的查询条件 `book(id: 4)` 可看出，通过传参id查询具体书籍信息，这里的id就体现在`args`参数上：
  - `args` 在 `resolve` 函数里处于第二个参数的位置
  ```js
  book: {
    type: BookType,
    description: 'A Single Book',
    // must query this book by params of book id:
    args: { id: { type: GraphQLInt } },
    resolve: ( parent, args ) => books.find( book => book.id === args.id )
  },
  ```
  - 同样的， `author`也可以按照传入`authorId`查询得到：
  ```js
  author: {
    type: AuthorType,
    description: 'A Single Author',
    // query an author by params of author id:
    args: { id: { type: GraphQLInt } },
    resolve: ( parent, args ) => authors.find( author => author.id === args.id )
  },
  ```
  用 `author(id)` 查询结果：
  ![单个查询](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/GraphQL学习/4.2.png)

## 五、修改数据（MutationType）
1. 运行`yarn ds5`，浏览器输入：
    ```js
    mutation {
      addBook(name: "new book", authorId: 1) {
        id
        name
      }
    }
    ```
    可以看到增加单本书成功：
    ![单个查询](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/GraphQL学习/5.1.png)
    
    输入：
    ```js
    mutation {
      addAuthor(name: "new author") {
        name
      }
    }
    ```
    可以看到增加一个作者成功：
    ![单个查询](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/GraphQL学习/5.2.png)

    点击右上角的 < Docs，可以发现该文件的Mutation函数，其中`!`意为必要字段：
    ![单个查询](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/GraphQL学习/5.png)


2. [`05mutationType.js`](https://github.com/ys558/tech-blog-code/blob/master/2020/04-graphql-learn/05mutationType.js)解析： 
  - 想要实现 `Mutation Type` 须在`GraphQLSchema`实例里添加`mutation`类型：
    ```js
    const Schema = new GraphQLSchema({
      query: RootQueryType,
      mutation: RootMutationType
    })
    ```
  - 和普通的 `Query Type` 类似，只不过 `Mutation Type` 在 `resolve` 里直接对数据进行改动而已：
    ```js
      fields: () => ({
        addBook: {
          type: BookType,
          description: 'Add a book',
          args: {
            name: { type: GraphQLNonNull(GraphQLString) },
            authorId: { type: GraphQLNonNull(GraphQLInt) }
          },
          resolve: ( parent, args ) => {
            const book = { id: books.length + 1, name: args.name, authorId: args.authorId }
            books.push(book)
            return book
          }
        },
        addAuthor: {
          type: AuthorType,
          description: 'Add a author',
          args: {
            name: { type: GraphQLNonNull(GraphQLString) }
          },
          resolve: ( parent, args ) => {
            const author = { id: authors.length + 1, name: args.name }
            authors.push(author)
            return author
          }
        },
      })
    ```