---
title: JWT自学笔记
date: 2021-06-21 16:37:49
tags:
    - JWT
    - Token
cover: https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.40/articles/jwt自学笔记/cover.jpg
---

JWT是一种跨域鉴权技术，现广泛运用于各大网站，本文结合[demo]()代码简单阐述，本篇的概念部分1到4点搬运了阮大神的这篇[文章](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)，第5点结合我自己写的一个Demo来实现具体的JWT业务

<!-- more -->

## 1. JWT是什么？
[JSON Web Token (jwt)](https://jwt.io/)是一个开放标准(RFC 7519)，它定义了一种紧凑的、自包含的方式，用于作为JSON对象在各方之间安全地传输信息。该信息可以被验证和信任，因为它是数字签名的。

简单来说就是用于跨域鉴权。一般来说我们鉴权的过程如下：

> 1、用户向服务器发送用户名和密码。   
> 2、服务器验证通过后，在当前对话（session）里面保存相关数据，比如用户角色、登录时间等等。   
> 3、服务器向用户返回一个 session_id，写入用户的 Cookie。   
> 4、用户随后的每一次请求，都会通过 Cookie，将 session_id 传回服务器。   
> 5、服务器收到 session_id，找到前期保存的数据，由此得知用户的身份。   

上述过程如果单机当然没有问题，如果是集群，或跨域的服务导向架构，就要求 session 数据共享，每台服务器都能够读取 session。   
举例来说，A 网站和 B 网站是同一家公司的关联服务。现在要求，用户只要在其中一个网站登录，再访问另一个网站就会自动登录，请问怎么实现？   
一种解决方案是 session 数据持久化，写入数据库或别的持久层。各种服务收到请求后，都向持久层请求数据。这种方案的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败。

另一种方案是服务器索性不保存 session 数据了，所有数据都保存在客户端，每次请求都发回服务器。 jwt 就是这种方案的一个代表。

## 2. JWT的结构

我们可以在[官网](https://jwt.io/)看到他的结构

[JWT结构](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.40/articles/JWT自学笔记/00.png)

可以看到分为以下3部分：

> Header（头部）   
> Payload（负载）   
> Signature（签名）   

### Header 头部

```json
{
  "alg": "PS384",
  "typ": "JWT"
}
```

以上分别代表
- `"typ"` 表示这个令牌（token）的类型（type）
- `"alg"` 表示该token的算法类型，默认为HMAC SHA256（写成 HS256） 

### Payload 负载

以下为官方规定的可选字段：
> iss (issuer)：签发人   
> exp (expiration time)：过期时间   
> sub (subject)：主题   
> aud (audience)：受众   
> nbf (Not Before)：生效时间   
> iat (Issued At)：签发时间   
> jti (JWT ID)：编号   

注意，这部分是不加密的，是简单的Base64转码过程，所以密文信息不能放在这部分，除非你的文字本身加密过

### Signature 签名
这部分是对前两部分的签名，防止数据篡改。

需指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名。

```js
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（.）分隔，就可以返回给用户。

Header 和 Payload 串型化的算法是 Base64URL。这个算法跟 Base64 算法基本类似，但有一些小的不同。

JWT 作为一个令牌（token），有些场合可能会放到 URL（比如 api.example.com/?token=xxx）。Base64 有三个字符`+`、`/`和`=`，在 URL 里面有特殊含义，所以要被替换掉：`=`被省略、`+`替换成`-`，`/`替换成`_` 。这就是 Base64URL 算法。


## 3. JWT使用方法：

客户端收到服务器返回的 JWT，可以储存在 Cookie 里面，也可以储存在 localStorage。

此后，客户端每次与服务器通信，都要带上这个 JWT。你可以把它放在 Cookie 里面自动发送，但是这样不能跨域，所以更好的做法是放在 HTTP 请求的头信息Authorization字段里面。

```
Authorization: Bearer <token>
```

另一种做法是，跨域的时候，JWT 就放在 POST 请求的数据体里面。

## 4. 使用时几个注意点

（1）JWT <font color=#FF0000>默认是不加密</font>，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次。

（2）JWT 不加密的情况下，不能将秘密数据写入 JWT。

（3）JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。

（4）JWT 的最大<font color=#FF0000>缺点</font>是，由于服务器不保存 session 状态，因此<font color=#FF0000>无法在使用过程中废止</font>某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。

（5）JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为减少盗用，<font color=#FF0000>JWT 的有效期应该设置得比较短</font>。对于一些比较重要的权限，使用时应该再次对用户进行认证。

（6）为了减少盗用，<font color=#FF0000>JWT </font>不应该使用 HTTP 协议明码传输，<font color=#FF0000>要使用 HTTPS 协议传输。</font>

## 5. [Demo 代码](https://github.com/ys558/jwt-learn) 及实际使用场景

完整代码可点击标题可见git仓库，下面按照我编写代码的顺序阐述几个注意点，

### 代码里一些配置的解析

1. `.env` 文件中的两个token生成可以按照以下办法，利用node REPL的`crypto`模块生成：

```bash
yuyi@home-pc MINGW64 /e/study/code/2021/jwt-learn (master)
$ node
Welcome to Node.js v14.14.0.      
Type ".help" for more information.
> require('crypto').randomBytes(64).toString('hex')
'6dedc49577ebb7c685246241533dc068ee43cba49d01b34d243b6b8924786a69d6637c4335e508343ba4c27a1fec1d2f11ea5eea173bd4af04c0f263a4ee98b5'
> require('crypto').randomBytes(64).toString('hex')
'4f3ea9c60d55b2a8a58f559ae76e133353a973708d6575daa10ac4bd28f6314d261a4458703be1333f65eddb29a1b58e21ebe493fede5c198952863f1f97e023'
>
```

2. 发送请求检测时不需要安装postman，只需在 vs code 上安装神器 `REST Client` ，不仅可以测普通的接口，最新的 `GraphQL` 语法也支持，其测试文件以 rest 结尾，如 `request.rest`

3. [dotenv](https://www.npmjs.com/package/dotenv) 是个不需要依赖就能读取项目中 `.env` 文件的配制的库，只需在 文件开始引用即可，如： 

```js
// dotenv库引用：
require('dotenv').config()

...

app.post('/login', (req,res) => {
  // 登录token鉴权
  const username = req.body.username
  const user = { name: username }

  const accessToken = jwt.sign(user, process.env.ACCESS_TOKEN_SECRET)
  res.json({ accessToken })
})
```

### 解析及测试主代码

1. `npm run dev-start` 把服务跑起后，点击 `request.rest` 文件的这里下面的位置，这些最基础的功能在 `01_base` 分支的[这些代码](https://github.com/ys558/jwt-learn/tree/base)就能实现，测试接口返回的正确与否

![rest cliennt测试接口](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.40/articles/jwt自学笔记/00.png)

2. 鉴权部分抽离为单独的中间件函数，`02_authenticate_token_modulize` 分支的[这些代码](https://github.com/ys558/jwt-learn/tree/02_authenticate_token_modulize)，


```js
function authenticateToken (req, res, next) {
  const authHeader = req.headers['authorization']
  const token = authHeader && authHeader.split(' ')[1]
  if (token == null) return res.sendStatus(401)
  jwt.verify(token, process.env.ACCESS_TOKEN_SECRET, (err, user) => {
    if (err) return res.sendStatus(403)
    req.user = user
    // 交给下个中间件：
    next()
  })
}
```

将其放入`app.get('/posts')`的第二个参数即可调用：

```js
// authenticateToken 为鉴权中间件：
app.get('/posts', authenticateToken, (req, res) => {   
  res.json(posts.filter(post => post.username === req.user.name))
})
```

![加入 authenticateToken middleware 后的自测](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.40/articles/jwt自学笔记/01.gif)

3. 将鉴权部分，即token的分发，拆分为单独一个模块。集中管理各个路由，包括`/login`, `/logout`的token，在后端属于微服务的架构

单独把颁发token这部分内容封装一个单独文件，新建文件 `authServer.js` 及更新命令 `auth`
```json
  "scripts": {
    "auth-start": "nodemon authServer.js"
  },
```

4. [Refresh Token](https://github.com/ys558/jwt-learn/tree/03_refresh_token)

上面是正常流程的注册登录所用的token，但实际业务中，`Access Token` 设置的时间不可能很长，比如说10分钟，而10分钟对于用户体验来说肯定不好，一过期的话就要重新登录，所以大多数业务采用了另一种折中的方案，即多生成一个 `Refresh Token`，这种的过期时间比较长，例如7天免登录
 
- 如果携带 `Access Token` 访问需要认证的接口时鉴权失败（例如返回 401 错误），则客户端使用 `Refresh Token` 向刷新接口申请新的 `Access Token`   
- 如果 `Refresh Token` 没有过期，服务端向客户端下发新的 `Access Token`   
- 客户端使用新的 `Access Token` 访问需要重新认证的接口

![Refresh Token工作流程](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.40/articles/jwt自学笔记/05.jpg)


下面解析 `Refresh Token` 的制作过程：
`authServer.js`文件中把生成Token封装成一个函数，并在 `app.post('/login', (req,res) => {` 里应用 

```js
app.post('/login', (req,res) => {
  const username = req.body.username
  const user = { name: username }

  const accessToken = generateAccessToken(user)
  const refreshToken = jwt.sign(user, process.env.REFRESH_TOKEN_SECRET)
  refreshTokens.push(refreshToken)
  res.json({ accessToken: accessToken, refreshToken: refreshToken })
})


function generateAccessToken(user){
  return jwt.sign(user, process.env.ACCESS_TOKEN_SECRET, { expiresIn: '15s' })
}
```

再次测试，能看到 Refresh Token 的返回结果，但我们发现直接用`/posts` 接口请求，会出现 Forbidden 报错：
![测试能否返回 Refresh Token](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.40/articles/jwt自学笔记/02.gif)

此时，我们再写多个函数控制 token 更新：
```js
// 这里的 refreshTokens
let refreshTokens = []

app.post('/token', (req, res) => {
  const refreshToken = req.body.token
  if (refreshToken == null) return res.sendStatus(401)
  if (!refreshTokens.includes(refreshToken)) return res.sendStatus(403)
  jwt.verify(refreshToken, process.env.REFRESH_TOKEN_SECRET, (err, user) => {
    if (err) return res.sendStatus(403)
    const accessToken = generateAccessToken({ name: user.name })
    res.json({ accessToken })
  })
})
```

`request.rest` 文件添加一个 `/token` 进行测试，body的 `"token"` 暂且留空
```rest
###
POST http://localhost:3001/token
Content-Type: application/json

{
  "token": ""
}
```
进行以下操作，可以看到效果：

![Refresh Token效果](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.40/articles/jwt自学笔记/03.gif)

但上述的 `GET` `/posts` 操作，因 `function generateAccessToken(user){` 函数里过期时间只设置了 15秒，所以15秒后，需要 `POST` `/token` 重新获得一个 `refresh token` 才能继续维持登录状态

5. 登出模块

完整代码在[这里](https://github.com/ys558/jwt-learn/tree/04_logout)

```js
app.delete('/logout', (req, res)=>{
  refreshTokens = refreshTokens.filter( token => token !== req.body.token)
  res.sendStatus(204)
})
```

用 `delete` 方法，删除 `let refreshTokens = []` 里原本存在的 `refresh token`，再次请求 `/posts` 时，会发现已经变为 `Forbidden` 状态

![测试能否返回 Refresh Token](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.40/articles/jwt自学笔记/06.gif)

