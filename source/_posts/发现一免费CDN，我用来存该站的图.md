---
title: 发现一免费CDN，我用来存该站的图
date: 2021-04-22 10:25:24
tags:
    - CDN
cover: https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.17/articles/发现一免费CDN，我用来存该站的图/cover.png
---

先上网址：https://www.jsdelivr.com/?docs=gh 

<!-- more -->

说说我遇到的痛点，该站采用`hexo`生成，在网上搜了一圈，建议用`hexo-asset-image`工具，但`_post`文件夹里会多出一个文件夹专门用于存储图片，且该工具还有一点bug，虽然最后也解决了，但总有点不爽。思前想后，还是新建一个放图片的git repo最省事，如果直接贴仓库，可能速度有点慢，于是就得找CDN了。

CDN （Content Delivery Network/Content Distribution Network）内容分发网络。是指一种透过互联网互相连接的电脑网络系统，利用最靠近每位用户的服务器，更快、更可靠地将音乐、图片、影片、应用程序及其他文件发送给用户，来提供高性能、可扩展性及低成本的网络内容传递给用户。 

选择了GitHub版本，操作如下：

1. 首先，新建一个GitHub仓库：例如我自己的图片的仓库： `https://github.com/ys558/my-blog-imgs`
2. 将你要放网络的图片推到你自己仓库
3. 关键的一步：**给仓库版本打标签**，两个方法：  
  方法一、命令行, 例如：
    ```
    git tag -a v1.4 -m "my version 1.4"
    ```
    命令行更详细的操作可参考[这里](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%89%93%E6%A0%87%E7%AD%BE)

    方法二、推完图片后，直接在你github图片仓库上操作：

    ![`git tag`](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.13/git-version.png)
  
4. 将你放图片链接的位置替换为 `jsdelivr` 举例的CDN格式即可，以我头像的图片举例，用`GitHub`的格式为：
    
    ```bash
    https://cdn.jsdelivr.net/gh/user/repo@version/file   
    ```
    替换为：
    ```bash
    https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.17/avatar/宇航员.svg
    ```