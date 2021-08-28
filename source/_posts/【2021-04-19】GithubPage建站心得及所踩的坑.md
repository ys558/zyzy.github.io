---
title: Github Page建站心得及所踩的坑
date: 2021-04-19 10:46:18
tags:
    - Hexo
    - Github Page
cover: https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.23/articles/GithubPage建站心得及所踩的坑/cover.jpg

---

最近想用 `github.io` 建立我的个人博客站点，具体步骤我参考了[知乎上的一篇文章](https://zhuanlan.zhihu.com/p/26625249)，里面写的够清楚了，下面就建站时碰到的坑写的不完全总结，估计以后陆续增加

<!-- more -->

## `ys558.github.io`改为自己的域名

去阿里云申请一个自己的域名后绑定原理的[ys558.github.io](https://ys558.github.io/zyzy.github.io/)域名，步骤：

1. 登陆[阿里云的万网](https://wanwang.aliyun.com/domain/)，挑一个喜欢便宜的域名注册，我的域名 [zyzy.info](https://zyzy.info) 一年只花21元，注册完成后选择个人站点，顺便也就开通了阿里云账号，如其他域名服务商，如申请`.me`结尾的个人网站，应该去[GoDaddy](https://au.godaddy.com/)等

2. 依次进入右上角的 **控制台** => **运维管理** 选项卡下找到 **域名** 标签 => 点击 **解析**，如下：  

![阿里云域名解析](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/GithubPage建站心得及所踩的坑/old/02.png)

=> 点击**添加记录**， 在 **记录值** 处填下`ys558.github.io`，如下，点击确定。这样就把你github.io上的页面绑定了购买的域名下

![阿里云域名解析](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/GithubPage建站心得及所踩的坑/old/03.png)

3. 用`ping` 也可查询到新绑定的域名网址IP：  
![阿里云域名解析](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.21/articles/GithubPage建站心得及所踩的坑/old/04.png)

4. 按以下命令在自己的github仓库中添加CNAME文件，其中填入购买的域名。或者可以在仓库的settings中设置Custom Domain设置好自己的域名，github会自动添加CNAME文件。
    ```shell
    touch CNAME
    echo 'zyzy.info'> CNAME
    ```


## 图片显示错误：

**！！！该方法也是在网上搜索得到，虽可行，但解析中文路径时经常会出错，后来还是转用CDN，把图片全部移到另一个repo，推荐用CDN的方法，具体操作办法看我[另一篇文章](https://zyzy.info/2021/04/22/%E5%8F%91%E7%8E%B0%E4%B8%80%E5%85%8D%E8%B4%B9CDN%EF%BC%8C%E6%88%91%E7%94%A8%E6%9D%A5%E5%AD%98%E8%AF%A5%E7%AB%99%E7%9A%84%E5%9B%BE/)**

1. 需要安装一个图片路径转换的插件，这个插件名字是 `hexo-asset-image`  

    ```shell
    npm i hexo-asset-image --save
    # or
    yarn add hexo-asset-image --save
    ```


2. `_config.yml`文件，修改下述内容，用 `hexo n <博客文章名字>` 生成新博客时，会生成一个对应的文件夹，将图片放入该文件夹引用即可：

    ```json
    post_asset_folder: true
    ```

3. 修改博客目录下`node_modules\hexo-asset-image\index.js` (hexo文件有bug)   
![阿里云域名解析](GithubPage建站心得及所踩的坑/05.png)


## 证书错误，不是私密连接

Github建站好之后，经常会碰到打开页面浏览器显示不是私密链接的情况，搜来搜去，说是GitHub的SSL证书不稳定，不知道是不是墙的原因。

![不是私密连接](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.20/articles/GithubPage建站心得及所踩的坑/00.png)

有两个方法可以解决：一是申请 cloudfare 的免费证书，二是自己买证书，两种方法都介绍以下

### 方法一： 某宝买证书
去某宝买了证书，（p.s. 不差钱也可以直接买阿里云的证书，两年2000多），证书是有有效期的，我买的那家证书一年80元，两年140元，5年300元。选个自己能接受，不太贵的价格即可。一般销量高的店家都有教你怎么配置，我这里把其配置记录了下来：

#### 1. 在 `Github Page` 上取消勾选 `Enforce HTTPS` 服务

位置如下图：   
![取消Github HTTPS 勾选](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.20/articles/GithubPage建站心得及所踩的坑/01.png)

#### 2. 将买到的证书在阿里云上设置

如下图顺序设置便可完成：

![阿里云设置证书](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.20/articles/GithubPage建站心得及所踩的坑/01.png)

### 方法二：蹭 cloudfare 的免费证书
#### 1. 申请个 [cloudfare](https://www.cloudflare.com/zh-cn/) 账号，并添加DNS

[cloudfare](https://www.cloudflare.com/zh-cn/) 我的cloudfare是以前注册的，忘了一步步截图，但有中文界面跟着提示操作即可，不会很难，注册完成后，在以下界面操作：

找到github的ipv4的地址为以下几个，也不知道访问时会分配到哪个ip，索性A类地址全部添加上，像阿里云以下，把 `CHAME` 类地址也添加上，如下：   

![DNS添加记录](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.20/articles/GithubPage建站心得及所踩的坑/03.png)


![ SSL/TSL 选择完全 ](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.20/articles/GithubPage建站心得及所踩的坑/04.png)

### 2. 返回阿里云添加 [cloudfare](https://www.cloudflare.com/zh-cn/) 申请的DNS服务器


![阿里云的DNS修改操作](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.20/articles/GithubPage建站心得及所踩的坑/05.png)   
![从 cloudfare 复制DNS ](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.20/articles/GithubPage建站心得及所踩的坑/06.png)