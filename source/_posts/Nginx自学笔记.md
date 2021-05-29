---
title: Nginx自学笔记
date: 2021-05-28 09:36:33
tags:
    - 后端
    - Nginx
    - 反向代理
cover: https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/cover.png
---
Ngix这几年太火了，在不久的将来会超过Apache，由于他的底层用epoll，现在多用于服务器端的反向代理和负载均衡 ，强大的并发性能使其基本上没有项目不用他。
本篇把 Ngix 的基础配置撸了一遍，记下来以免以后忘了。
<!-- more -->

## 安装

可查看[Nginx官网](http://nginx.org/en/download.html)

### 连接远程服务器
```shell
ssh root@8.134.120.206
```
输入密码后见到登录了
![ssh 登录远程服务器](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/00.png)

### 安装些必要的工具
```shell
yum -y install gcc gcc-c++ autoconf pcre-devel make automake
yum -y install wget httpd-tools vim
```

### 返回家目创建Nginx主要文件夹
```shell
cd ~
mkdir nginx-learn && cd nginx-learn
mkdir app backup download logs work
```
app: 放程序源码
backup: 备份文件
download: 下载文件夹
logs: 日志
work: 放一些项目文档

### 安装最新版Nginx

需查看Nginx的yum源的版本，运行以下命令：

```shell
yum list | grep nginx 
```

![创建nginx主目录并查看yum源的版本](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/01.png)

yum 源的更新配置，复制Nginx官网[这里](https://nginx.org/en/linux_packages.html#RHEL-CentOS)的配置，我把他粘出来

```text
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

```shell
vim /etc/yum.repos.d/nginx.repo
```

其中 `baseurl=http://nginx.org/packages/centos/$releasever/$basearch/` 的 `$releasever` 我用的是centOS 7的版本，所以我直接改成7，如下：
![改动nginx.repo文件](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/02.png)

这次再安装nginx
```shell
yum install nginx
nginx -v
# or
nginx -V
```

可以发现更新到最新的nginx版本
![改动yum源后，安装了nginx最新版1.20.1](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/03.png)


查看所有 nginx 目录及配置文件的命令：

```shell
rpm -ql nginx
```
![所有nginx配置文件](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/04.png)

### Nginx配置项详解
配置项：大部分工作在以下两个文件执行修改：
/etc/nginx/nginx.conf
cd /etc/nginx
cat nginx.conf
![所有nginx配置文件](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/05.png)

下面解释一下各配置的作用：
```text
user  nginx; # 默认用户
worker_processes  auto; # 进程数，即cpu数

error_log  /var/log/nginx/error.log notice; # 错误日志
pid        /var/run/nginx.pid; # 进程


events {
    worker_connections  1024; # 允许最大连接数
}


http {
    include       /etc/nginx/mime.types; # 文件扩展名的映射表
    default_type  application/octet-stream; # 

    # 设置日志格式：
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    # 访问日志存放位置
    access_log  /var/log/nginx/access.log  main;

    # 开启默认传输模式
    sendfile        on;

    # 减少网络报文数量
    #tcp_nopush     on;

    # 超时时间
    keepalive_timeout  65;

    # 开启压缩
    #gzip  on;

    # 子配置项
    include /etc/nginx/conf.d/*.conf;
}
```

```shell
cd conf.d/
vim default.conf
```

```
server {
    listen       80; # 监听端口号
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

### 网站根目录
我们改动一点根目录的内容：
```shell
vim /usr/share/nginx/html/index.html
```
![利用vim对index.html的内容进行一点改动](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/001.png)


## 运行Nginx
### 开启
```shell
nginx
```

### 关闭
```shell
# 正常关停
nginx -s quit

# 几种强制关停
nginx -s stop
killall nginx
```

### 重载
每次改完配置文件后，需要重载nginx，运行一下命令：
```shell
nginx -s reload
```

### Linux通用命令执行开启，关闭和重载
当然，也可以用Linux的通用命令进行关停或重启：
```shell
# 或 Linux通用命令：
systemctl start nginx.service

# Linux通用命令关停：
systemctl stop nginx.service

# 重启：
systemctl restart nginx.service
```

### 查看进程
通过查看nginx所有进程确认其已经运行：
```shell
ps aux | grep nginx
```
![查看所有进程](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/10.png)

### 查看所有开启的端口号

```shell
netstat -tlnp
```

![查看所有开启的端口号](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/11.png)


配置一个404页面，修改一个配置如下：
```shell
vim /etc/nginx/conf.d/default.conf
```
增加一行，改成这样：   

![添加自定义404页面](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/12.png)
 
做一个404页面：   
```shell
vim /usr/share/nginx/html/404_error.html
```

![自己写的404页面](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/20.png)

写完页面后，地址栏随便输入一地址会发现出现了404页面：

![查看404页面效果](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/21.png)


也可以重定向到其他页面，如：
```shell
vim /etc/nginx/conf.d/default.conf
```
![查看所有进程](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/13.png)
 

## 权限控制

### 禁止某些ip不能访问

```shell
vim /etc/nginx/conf.d/default.conf
```

- 在default.conf加上 `deny  113.65.137.13;` ，如下，禁止了我本机不可以访问    
- 如果要禁止某一网段的IP不能访问，则写：`deny  113.65.137.13/200;` ，表示 13 到 200 的所有IP号都不能访问   
- `allow` 允许哪些IP可以访问   
- 注意！！！`default.conf` 文件是从上往下执行，如 `deny  all;` 写在 `allow  113.65.137.13;`之前，那么`113.65.137.13`仍不能访问  


![禁止某些IP访问](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/22.png)      

再次在页面打开，就会发现不能访问页面，出现403页面：

![禁止某些IP访问后出现403页面](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/23.png)   

### 精确匹配路径访问权限

- 在 `location` 配置某些路径允许哪些IP 进行访问，添加如下：
- 都是正则匹配，除了路径，还可匹配不能访问的文件，如 `.php` 结尾的文件   

```text
    location =/img {
        allow all;
    }
    location =/admin {
        deny all;
    }
    location ~\.php$ {
        deny all;
    }
```

## 虚拟主机

配置虚拟主机的好处，可以省钱，一台物理服务器可划分多个空间，每个独立空间可配置独立的虚拟主机，彼此相互隔离，每个虚拟主机可以独立配置web服务

### 基于端口号配置虚拟主机

在 `/etc/nginx/conf.d/` 下新建 `8001.conf` 文件，用于配置 `8001` 端口的虚拟主机

```shell
vim /etc/nginx/conf.d/8001.conf
```

![基于8001配置新的虚拟主机](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/25.png)   

基于改虚拟主机配置一个 `index.html` 文件：   

```shell
touch /usr/share/nginx/html/html8001/index.html
```

写下
```html
<h1>welcome to port 8001</h1>
```

我这里用的是阿里云做演示，所以返回阿里云界面配置8001端口：
![阿里云配置8001端口步骤1](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/26.png)  
![阿里云配置8001端口步骤2](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/27.png)  
![阿里云配置8001端口步骤3](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/28.png)  
![阿里云配置8001端口步骤4](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/29.png) 


### 基于IP配置虚拟主机

同样是改变 `8001.conf` 文件的 `localhost` 项的配置
```shell
vim /etc/nginx/conf.d/8001.conf
```

如：

```text
server{
    listen 80;
    server_name 192.168.1.200;
    root /usr/share/nginx/html/html8001;
    index index.html;
}
```

### 基于域名配置虚拟主机

在阿里云上申请域名，这里不演示，现在 `.top` 结尾的域名在做促销，起个稍微长点的域名注册只需9元就能注册一年， 我这里注册了 `service-learn.top` 花了9元

点击域名解析：   

![阿里云配置域名解析1](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/32.png)

输入ip地址，类型选择A类型：

![阿里云配置域名解析2](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/33.png)

在nginx 配置

```shell
vim /etc/nginx/conf.d/default.conf
```

将 `server_name  localhost;` 改为 `server_name  niginx.service-learn.top;`

同样的原来的基于端口的虚拟机8001端口也修改：   
先去阿里云配置解析 `nginx2.service-learn.top` 地址

```shell
vim /etc/nginx/conf.d/8001.conf
```


将 `server_name  localhost;` 改为  `server_name nginx2.service-learn.top;`，改后如下：

```text
server{
        listen 80;
        server_name nginx2.service-learn.top;
        root /usr/share/nginx/html/html8001;
        index index.html;
}
```

## 反向代理


### 正向代理（Proxy）

为客户端做的代理服务器，最形象的解释就是，我们平时用vpn翻墙上外网，

![正向代理](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/正向代理图.jpg)

### 反向代理（Reverse Proxy)

为服务端做的代理服务，反向代理最大的好处是

- 安全性，使用反向代理客户端用户只能通过外来网来访问代理服务器，并且用户并不知道自己访问的真实服务器是那一台，可以很好的提供安全保护。在遭受网络攻击时，会被停留在反向代理上，攻击不到我们真实的服务器。

- 功能性，反向代理的主要用途是为多个服务器提供负债均衡、缓存等功能。负载均衡就是一个网站的内容被部署在若干服务器上，可以把这些机子看成一个集群，那Nginx可以将接收到的客户端请求“均匀地”分配到这个集群中所有的服务器上，从而实现服务器压力的平均分配，也叫负载均衡。

![反向代理](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/反向代理图.jpg)


### Nginx的反向代理实操

这里我们用 8001 的配置来做反向代理，设置也很简单，修改 `vim /etc/nginx/conf.d/8001.conf` 如下

我们要访问 8.134.120.206:8001 端口的网站，反向代理到 [我的博客](http://zyzy.info) 这个网站上

```text
server{
        listen 8001;
        server_name 8.134.120.206;
        location / {
                proxy_pass http://zyzy.info;
        }
}
```

打开浏览器显示如下，反向代理设置成功：
![反向代理](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/36.png)

### 反向代理其他配置参数

proxy_set_header :在将客户端请求发送给后端服务器之前，更改来自客户端的请求头信息。
proxy_connect_timeout:配置Nginx与后端代理服务器尝试建立连接的超时时间。
proxy_read_timeout : 配置Nginx向后端服务器组发出read请求后，等待相应的超时时间。
proxy_send_timeout：配置Nginx向后端服务器组发出write请求后，等待相应的超时时间。
proxy_redirect :用于修改后端服务器返回的响应头中的Location和Refresh。

## Nginx适配移动端

这里和前端的调整页面布局的自适应不是一个概念，现在的移动端适配，是由于不同域名的切换，引起的适配，所以Nginx在这块就能发挥作用，   
这种通过Nginx的适配，体验效果会更好，包括淘宝京东在内的国内大型网站都是采用Nginx的方案

我们可以打开浏览器打开京东看到这种适配：

![京东的Nginx移动端适配](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/37.gif)

我们来做类似的功能：   

建立pc和移动端两个文件夹

```shell
mkdir /usr/share/nginx/pc /usr/share/nginx/mobile
```

分别编辑 `vim /usr/share/nginx/mobile/pc/index.html` 和 `vim /usr/share/nginx/mobile/mobile/index.html`，随便写点内容

编辑Nginx设置如下即可   

```text
server{
        listen 8001;
        server_name 8.134.120.206;
        location / {
                root /usr/share/nginx/pc;
                if ($http_user_agent ~* 'Android|webOS|iPhone|BlackBerry') {
                        root /usr/share/nginx/mobile;
                }
                index index.html;
        }
}
```

打开浏览器看看效果

![京东的Nginx移动端适配](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/38.gif)


## Gzip压缩

网页的一种压缩技术，经过Gzip压缩过后的网页，会缩小到原来30%的大小

### 查看一个网站是否有gzip压缩

在[站长工具](http://tool.chinaz.com/Gzips/Default.aspx?q=zyzy.info) 里查看，直接输入域名即可，例如我们查询 `8.134.120.206`
还没配置gzip压缩   

![没有配置gzip的网站](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/39.gif)


Nginx的gzip压缩功能非常丰富，配置也很简单

```shell
 vim /etc/nginx/nginx.conf
```

添加两行配置如下：   

```text
gzip on;
gzip_types text/plain application/javascript text/css;
```

![Nginx添加gzip配置](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/40.gif)

改完后重载Nginx服务，再用站长工具查询可看到

![配置了gzip的网站](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.35/articles/Nginx自学笔记/41.gif)