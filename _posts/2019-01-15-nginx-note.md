---
title: "Nginx note"
layout: post
date: 2019-01-15 22:44
tag:
- nginx
- server
- linux
star: true
category: blog
author: jiaqixu
description: note for nginx
---

### 目录

通过学习[nginx官方文档](http://nginx.org/en/docs)做出的笔记.

#### 配置文件的结构
nginx由模块组成,模块由配置文件中指定的指令控制. 指令被分为简单指令(simple directives)和块指令(block directives).

放在配置文件中但又在任何其他`context`之外的指令被认为是在`main context`中. `events` 和 `http`指令存在于`main` context, `server`在`http` context, `location`在`server`.

在`#` 标记后的语句被认为是注释(comment).

#### 服务静态内容
通常来说, 配置文件包含几个`server`块,这些`server`块由它们监听的接口和服务器名称来区分.
一旦nginx决定用哪个服务器来处理请求,它就根据服务器块中定义的`location`指令的参数来测试请求头中指定的URI.

对于匹配的请求,URI将会被加到由`root`指令指定的路径下, 从而形成在一个新的路径.这条新的路径指向了一个本地文件系统上的文件.
如果有多个匹配的`location`块,则nginx会选择匹配最长前缀的那个`location`块.

如果有一些不期望的的错误或者其他的什么发生,可以尝试从(/usr/local/nginx或者/var/log/nginx)下的`access.log`和`error.log`中寻找原因.


#### 设置简单的代理服务器
note:这里注意proxy server 和 proxied server在文中的意思.
nginx的一个常见的用途是将其设置为代理服务器(proxy server),这意味着服务器(proxy server)接收请求,将请求传递给proxied servers, 并从其那里检索响应(response) 并把response发还给用户.

下面的例子,我们配置了一个基本的proxy server, 这个proxy server用本地文件夹下的文件来服务(serve)图片请求;对于其他的请求,它将其发送到一个proxied server.
```
server {
    listen 8080;
    root /data/up1;

    location / {
    }
}

server {
    location / {
        proxy_pass http://localhost:8080/;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```
说明:
1. 第一个`server`块定义了以个proxied server, 这是一个简单的`server`, 它监听8080端口.(如果没有指定默认是80端口).
2. 在第一个`server`块中,`root`指令被放置在`server` context中. 在这种情况下,如果`location`块中没有包括其自己的`root`,则会使用`server`块下的`root`.
3. 第二个`server`块是一个代理服务器(`proxy server`), 在它的第一个`location`块中,用了`proxy_pass`指令(参数指定为`http://localhost:8080`,包含了proxied server的protocol,name和端口).
4. 第二个`server`块的第二个`location`中,参数是正则表达式(由`~`来表明者是一个正则表达式).
5. 因此第二个`server`块会把由`.gif`, `.jpg` 和`.png`为结尾的请求让第二个`location`块处理,其他的请求会被第一个`location`块处理,转发给proxied server(第一个`server`块).


#### 配置FastCGI Proxying
nginx可用于将请求路由到FastCGI服务器, FastCGI服务器运行各种框架和编程语言(例如php)构建的应用程序.

使用FastCGI服务器的最基本的nginx配置包括使用`fastcgi_pass`指令来代替`proxy_pass`指令, 以及使用`fastcgi_para`指令来设置传递给FastCGI服务器的参数.

假设FastCGI服务器可以被访问通过`localhost:9000`,在前面的例子的基础上将第二个`proxy server`做以下修改即可:
```
server {
    location / {
        fastcgi_pass  localhost:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param QUERY_STRING    $query_string;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```
这就将设置完一个服务器,通过FastCGI协议将除静态图像请求外的所有请求路由到在`localhost:9000`上运行的`proxied server`.
