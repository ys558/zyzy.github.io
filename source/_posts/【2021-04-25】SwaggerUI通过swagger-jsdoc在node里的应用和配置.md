---
title: SwaggerUI通过swagger-jsdoc在node里的应用和配置
date: 2021-04-25 08:48:17
tags: 
    - Swagger UI
    - 后端
    - Nodejs
    - swagger-jsdoc
cover: https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.23/articles/SwaggerUI通过swagger-jsdoc在node里的应用和配置/cover.jpg
---

[SwaggerUI](https://swagger.io/)，在Java里应该是常用工具，能根据api文件自动生成网页版的api文档界面，感觉挺有意思的，其实他支持各种语言，本篇分享一下在node里的简单配置方法，顺手搭建了个简单的服务。   
（p.s. 本篇后续有续篇，利用本篇的服务，介绍一下前端的状态管理组件React Query的玩法，配合本篇搭建的服务使用。）

<!-- more -->

## 基本架构

初始化项目，安装依赖: 

```shell
npm init -y
yarn add express lowdb morgan nanoid cors swagger-jsdoc swagger-ui-express
touch index.js
```

`index.js` 如下：
```js
const express  = require("express");
const cors  = require("cors");
const morgan  = require("morgan");
const low = require("lowdb");

const PORT = process.env.PORT || 4800

const FileSync = require("lowdb/adapters/FileSync")

// 用 db.json 文件充当数据库:  
const adapter = new FileSync("db.json")
const db = low(adapter)

db.defaults({ books: [] }).write()

const app = express()


app.db = db

app.use(cors())
app.use(express.json())
app.use(morgan("dev"))

app.listen(PORT, () => console.log(`the server run on port ${PORT}`))
```

试着运行一下，会发现生成了一个 `db.json` 文件，总体目录如下：

    server
      |__node_modules/
      |__db.json
      |__index.js
      |__package.json
      |__yarn.lock

## 建立路由

```shell
mkdir route && cd route && touch books.js
```

`books.js` 如下：
```js
const express  = require("express");
const router  = express.Router()
const { nanoid }  = require("nanoid");

const idLength = 8

router.get('/', (req, res) => {
  const books = req.app.db.get('books')
  res.send(books)
})

router.get('/:id', (req, res) => {
  const book = req.app.db.get('books').find({ id: req.params.id }).value()
  if (!book) res.send(404)
  res.send(book)
})

router.post('/', (req, res) => {
  try {
    const book = {
      id: nanoid(idLength),
      ...req.body
    }
    req.app.db.get('books').push(book).write()
    res.send(book)
  }catch (error) {
    return res.status(500).send(error)
  }
})

router.put('/:id', (req, res) => {
  try {
    req.app.db.get('books').find({ id: req.params.id }).assign(req.body).write()
    res.send(req.app.db.get('books')).find({id: req.params.id})
  }catch (error) {
    return res.status(500).send(error)
  }
})

router.delete('/:id', (req, res) => {
  req.app.db.get('books').remove({ id: req.params.id }).write()
  res.sendStatus(200)
})

module.exports = router
```

## 应用SwaggerUI:

在 `index.js` 里引用 `SwaggerUI`，添加如下配置：
```js
const swaggerUI  = require("swagger-ui-express");
const swaggerJsDoc  = require("swagger-jsdoc");
const booksRouter  = require("./route/books");

const specs = swaggerJsDoc({
  definition: {
    // 必须参数，不填则没有权限，填的话也只支持 3.0.n
    openapi: "3.0.0",
    // 一些描述，可选：
    info: {
      title: "Library API",
      version: "1.0.0",
      description: "A simple Express Library API"
    },
    servers: [{ url: "http://localhost:4800"}],
  },
  apis: ["./route/*.js"]
})

app.use("/api-docs", swaggerUI.serve, swaggerUI.setup(specs))

app.use('/books', booksRouter)
```

可在界面看到：   
 ![swaggerUI界面](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/SwaggerUI通过swagger-jsdoc在node里的应用和配置/0.png)

## 自动生成api文档页面：

参照其[文档](https://swagger.io/docs/specification/components/)，比较有意思的是[swagger-jsdoc](https://github.com/Surnet/swagger-jsdoc)，他将多行注释里的api配置转换为页面的文档配置，我们将 `route/books.js` 改写为以下格式：

```js
/**
 * @swagger
 *   components:
 *     schemas:
 *       Book:
 *         type: object
 *         required:
 *           - title
 *           - author
 *         properties:
 *           id:
 *             type: string
 *             description: The auto-generated id of the book
 *           title:
 *             type: string
 *             description: The book title
 *           author:
 *             type: string
 *             description: The book author
 *         example:
 *           id: d5fE_asz
 *           title: The New Turing Omnibus
 *           author: Alexander K. Dewdney
 */


/**
 * @swagger
 *   tags:
 *     name: Books
 *     description: The books managing API
 */

/**
 * @swagger
 *   /books:
 *     get:
 *       summary: Returns the list of all the books
 *       tags: [Books]
 *       responses:
 *         200:
 *           description: The list of the books
 *           content:
 *             application/json:
 *               schema:
 *                 type: array
 *                 items:
 *                   $ref: '#/components/schemas/Book'
 */

router.get('/', (req, res) => {
  const books = req.app.db.get('books')
  res.send(books)
})

/**
 * @swagger
 * /books/{id}:
 *   get:
 *     summary: Get the book by id
 *     tags: [Books]
 *     parameters: 
 *       - in: path
 *         name: id
 *         schema:
 *           type: string
 *         required: true
 *         description: The book id
 *     responses:
 *       200:
 *         description: The book description by id
 *         contents:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Book'
 *       404:
 *         description: The book was not found
 */
router.get('/:id', (req, res) => {
  const book = req.app.db.get('books').find({ id: req.params.id }).value()
  if (!book) res.send(404)
  res.send(book)
})

/**
  * @swagger
  *  /books:
  *    post:
  *      summary: Create a new book
  *      tags: [Books]
  *      requestBody: 
  *        required: true
  *        content:
  *          application/json:
  *            schema:
  *              $ref: '#/components/schemas/Book'
  *      responses:
  *        200:
  *          description: The book was successfully created
  *          content:
  *            application/json:
  *              schema:
  *                $ref: '#/components/schemas/Book'
  *        500:
  *          description: Server Error
  */
router.post('/', (req, res) => {
  try {
    const book = {
      id: nanoid(idLength),
      ...req.body
    }
    req.app.db.get('books').push(book).write()
    res.send(book)
  }catch (error) {
    return res.status(500).send(error)
  }
})

/**
  * @swagger
  *  /books/{id}:
  *    put:
  *      summary: Update the book by the id
  *      tags: [Books]
  *      parameters:
  *        - in: path
  *          name: id
  *          schema:
  *            type: string
  *          required: true
  *          description: The book id
  *      requestBody:
  *        required: true
  *        content:
  *          application/json:
  *            schema:
  *              $ref: '#/components/schemas/Book'
  *      responses:
  *        200:
  *          description: The Book was updated
  *          content:
  *            application/json:
  *              schema:
  *                $ref: '#/components/schemas/Book'
  */
router.put('/:id', (req, res) => {
  try {
    req.app.db.get('books').find({ id: req.params.id }).assign(req.body).write()
    res.send(req.app.db.get('books')).find({id: req.params.id})
  }catch (error) {
    return res.status(500).send(error)
  }
})

/**
  * @swagger
  *   /books/{id}:
  *   delete:
  *     summary: Update the book by the id
  *     tags: [Books]
  *     parameters:
  *       - in: path
  *         name: id
  *         schema:
  *           type: string
  *         required: true
  *         description: The book id
  *     responses:
  *       200:
  *         description: The Book was deleted
  *       404:
  *         description: The Book was not found
  */

router.delete('/:id', (req, res) => {
  req.app.db.get('books').remove({ id: req.params.id }).write()
  res.sendStatus(200)
})

module.exports = router
```

重跑服务，可以发现以下界面：   
 ![swaggerUI界面](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/SwaggerUI通过swagger-jsdoc在node里的应用和配置/1.png)   
`swagger-jsdoc` 是通过 `openapi` 的3.0版本进行的改动，传统的 `openapi` 是通过独立的 `json` 文件对 swagger进行配置，不在本篇讨论范围，感兴趣的可以看看这篇国外的[demo](https://levelup.gitconnected.com/how-to-add-swagger-ui-to-existing-node-js-and-express-js-project-2c8bad9364ce)
