---
title: "Nginx learning note"
layout: post
date: 2019-10-30 22:44
tag:
- nginx
star: true
category: blog
author: jiaqixu
description: note for nginx
---

### 目录
- [Nginx安装-源码方式](#nginx安装-源码方式)
- [Nginx架构](#nginx架构)
- [rewrite配置if指令](#rewrite配置if指令)
- [rewrite中的break和last](#rewrite中的break和last)
    * [break和last在location外部](#break和last在location外部)
    * [当break和last在location里面](#当break和last在location里面)
    * [结论](#结论)
- [return指令](#return指令)
    * [直接返回状态码](#直接返回状态码)
    * [返回字符串](#返回字符串)
    * [返回url](#返回url)
    * [生成场景实战](#生成场景实战)
- [rewrite规则](#rewrite规则)
- [nginx常用全局变量](#nginx常用全局变量)
- [rewrite实战](#rewrite实战)
    * [域名重定向](#域名重定向)
    * [防盗链](#防盗链)
    * [伪静态](#伪静态)
    * [rewrite多个条件的并且](#rewrite多个条件的并且)
- [location配置](#location配置)
    * [安装echo-nginx-module模块](#安装echo-nginx-module模块)
    * [规则优先级](#规则优先级)
    * [规则示例](#规则示例)
    * [小常识](#小常识)
- [nginx代理](#nginx代理)
    * [示意图](#示意图)
    * [Nginx正向代理](#nginx正向代理)
       - [Nginx正向代理配置文件](#nginx正向代理配置文件)
       - [Nginx正向代理配置执行说明](#nginx正向代理配置执行说明)
    * [Nginx反向代理](#nginx反向代理)
        - [配置说明](#配置说明)
        - [Nginx反向代理buffer和proxy_cache](#nginx反向代理buffer和proxy_cache)
- [Nginx负载均衡](#nginx负载均衡)
- [Nginx访问控制](#nginx访问控制)
    * [deny和allow](#deny和allow)
    * [基于location的访问控制](#基于location的访问控制)
    * [Nginx基于document_uri的访问控制](#nginx基于document_uri的访问控制)
    * [Nginx基于request_uri访问控制](#nginx基于request_uri访问控制)
    * [Nginx基于user_agent的访问控制](#nginx基于user_agent的访问控制)
    * [Nginx基于http_referer的访问控制](#nginx基于http_referer的访问控制)
    * [Nginx的限速](#nginx的限速)
- [Nginx用户认证](#nginx用户认证)
- [Nginx之SSL配置](#nginx之ssl配置)
    * [CA证书](#ca证书)
    * [SSL原理](#ssl原理)
    * [自制ca证书](#自制ca证书)
    * [Nginx单向ssl配置](#nginx单向ssl配置)
    * [Nginx配置双向认证](#nginx配置双向认证)
- [Nginx日志配置](#nginx日志配置)
    * [Nginx的错误日志](#nginx的错误日志)
    * [Nginx访问日志格式](#nginx访问日志格式)
    * [Nginx访问日志配置](#nginx访问日志配置)
    * [Nginx访问日志过滤](#nginx访问日志过滤)
    * [Nginx访问日志切割](#nginx访问日志切割)
- [Nginx优化](#nginx优化)
    * [Nginx配置参数优化](#nginx配置参数优化)
    * [调整Linux内核参数](#调整linux内核参数)
- [Nginx服务监控](#nginx服务监控)
    * [系统级别监控](#系统级别监控)
    * [配置Nginx状态](#配置nginx状态)
    
       

### Nginx安装-源码方式
    curl -O http://nginx.org/download/nginx-1.14.0.tar.gz
    tar zxf nginx-1.14.0.tar.gz
    cd nginx-1.14.0; ./configure --prefix=/usr/local/nginx (编译的参数可以用./config --help查看)
    make; make install; 运行完之后，在/usr/local/nginx目录下就可以看到conf(配置文件目录),html(默认页目录)，sbin(可执行文件目录),logs(日志文件目录)
    
    一些基本命令:
    /usr/local/nginx/sbin/nginx; 启动
    pkill nginx; 杀死nginx进程，停止nginx服务
    /usr/local/nginx/sbin/nginx -t 检测配置文件语法错误
    /usr/local/nginx/sbin/nginx -s reload 重载配置
    
    使用源码的方式安装nginx，可以使以后安装nginx第三方模块的时候更加简单方便。
    
    
#### 服务管理脚本
上面关于nginx命令有些长，以下脚本可以方便我们操作，我将这个脚本放在/etc/init.d/nginx下。
```bash
#!/bin/bash
# chkconfig: - 30 21
# description: http service.
# Source Function Library
. /etc/init.d/functions
# Nginx Settings

NGINX_SBIN="/usr/local/nginx/sbin/nginx"
NGINX_CONF="/usr/local/nginx/conf/nginx.conf"
NGINX_PID="/usr/local/nginx/logs/nginx.pid"
RETVAL=0
prog="Nginx"

start() 
{
    echo -n $"Starting $prog: "
    mkdir -p /dev/shm/nginx_temp
    daemon $NGINX_SBIN -c $NGINX_CONF
    RETVAL=$?
    echo
    return $RETVAL
}

stop() 
{
    echo -n $"Stopping $prog: "
    killproc -p $NGINX_PID $NGINX_SBIN -TERM
    rm -rf /dev/shm/nginx_temp
    RETVAL=$?
    echo
    return $RETVAL
}

reload()
{
    echo -n $"Reloading $prog: "
    killproc -p $NGINX_PID $NGINX_SBIN -HUP
    RETVAL=$?
    echo
    return $RETVAL
}

restart()
{
    stop
    start
}

configtest()
{
    $NGINX_SBIN -c $NGINX_CONF -t
    return 0
}

case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  reload)
        reload
        ;;
  restart)
        restart
        ;;
  configtest)
        configtest
        ;;
  *)
        echo $"Usage: $0 {start|stop|reload|restart|configtest}"
        RETVAL=1
esac

exit $RETVAL

```


### Nginx架构

Nginx服务器使用master/work 多进程模式。

主进程启动后，会接收和处理外部信号；主进程启动后通过fork()函数产生一个或多个子进程，每个子进程
会进行进程初始化，模块调用以及对事件的接收和处理等工作。

    ![image](/assets/images/blog/nginx-1.png)
    
**主进程**

主要功能是和外界通信和对内部其他进程进行管理，具体来说有以下几点:
* 读取nginx配置文件并验证其有效性和正确性
* 建立，绑定和关闭socket
* 按照配置生成，管理工作进程
* 接收外界指令，比如重启，关闭，重载服务等指令
* 日志文件管理

**子进程**

是由主进程生成，生成数量可以在配置文件中定义。该进程主要工作有:

* 接收客户端请求
* 将请求依次送入各个功能模块进行过滤处理
* IO调用，获取响应数据
* 与后端服务器通信，接收后段服务器处理结果
* 数据缓存，访问缓存索引，查询和调用缓存数据
* 发送请求结果，响应客户端请求
* 接收主进程指令，如重启，重载，退出等


### rewrite配置if指令
    格式：if (条件判断) { 具体的rewrite规则 }
    
#### 条件举例

    条件判断语句由Nginx内置变量、逻辑判断符号和目标字符串三部分组成。
    其中，内置变量是Nginx固定的非自定义的变量，如，$request_method, $request_uri等。
    逻辑判断符号，有=, !=, ~, ~*, !~, !~*
    !表示相反的意思，~为匹配符号，它右侧为正则表达式，区分大小写，而~*为不区分大小写匹配。
    目标字符串可以是正则表达式，通常不用加引号，但表达式中有特殊符号时，比如空格、花括号、分号等，需要用单引号引起来。

#### 示例1

    if ($request_method = POST)  #当请求的方法为POST时，直接返回405状态码
    {
	    return 405; #在该示例中并未用到rewrite规则，if中支持用return指令。
    }

#### 示例2

    if ($http_user_agent ~ MSIE) #user_agent带有MSIE字符的请求，直接返回403状态码
    {
	    return 403;
    }

    如果想同时限制多个user_agent，还可以写成这样

    if ($http_user_agent ~ "MSIE|firefox|spider")
    {
	    return 403;
    }

#### 示例3

    if(!-f $request_filename)  #当请求的文件不存在，将会执行下面的rewrite规则
    {
        rewrite 语句;
    }
    
#### 示例4

    if($request_uri ~* 'gid=\d{9,12}/')  #\d表示数字，{9,12}表示数字出现的次数是9到12次，如gid=123456789/就是符合条件的。
    {
        rewrite 语句;
    }
    

### rewrite中的break和last
    两个指令用法相同，但含义不同，需要放到rewrite规则的末尾，用来控制重写后的链接是否继续被nginx配置执行(主要是rewrite、return指令)。
    
    示例1（连续两条rewrite规则）：
    server{
        listen 80; 
        server_name test.com;
	    root /tmp/123.com;

	    rewrite /1.html /2.html ;
	    rewrite /2.html /3.html ;
        
    }
    当我们请求1.html时，最终访问到的是3.html，两条rewrite规则先后执行。
    

#### break和last在location外部

    格式：rewrite xxxxx  break;
    
    示例2（增加break）：
    server{
        listen 80; 
        server_name test.com;
	    root /tmp/123.com;

	    rewrite /1.html /2.html break;
	    rewrite /2.html /3.html;
    }
    当我们请求1.html时，最终访问到的是2.html
    说明break在此示例中，作用是不再执行break以下的rewrite规则。
    
    但，当配置文件中有location时，它还会去执行location{}段的配置（请求要匹配该location）。
    
    示例3（break后面还有location段）：
    server{
        listen 80; 
        server_name test.com;
	    root /tmp/123.com;

	    rewrite /1.html /2.html break;
	    rewrite /2.html /3.html;
        location /2.html {
            return 403;
        }
    }
    当请求1.html时，最终会返回403状态码，说明它去匹配了break后面的location{}配置。
    
    以上2个示例中，可以把break替换为last，它们两者起到的效果一模一样。
    
#### 当break和last在location里面
    
    示例4（什么都不加）：
    server{
        listen 80; 
        server_name test.com;
        root /tmp/123.com;
        
        location / {
            rewrite /1.html /2.html;
            rewrite /2.html /3.html;
        }
        location /2.html
        {
            rewrite /2.html /a.html;
        }
        location /3.html
        {
            rewrite /3.html /b.html;
        }
    }
    当请求/1.html，最终将会访问/b.html，连续执行location /下的两次rewrite，跳转到了/3.html，然后又匹配location /3.html
    
    示例5（增加break）：
    server{
        listen 80; 
        server_name test.com;
        root /tmp/123.com;
        
        location / {
            rewrite /1.html /2.html break;
            rewrite /2.html /3.html;
        }
        location /2.html
        {
            rewrite /2.html /a.html;
        }
        location /3.html
        {
            rewrite /3.html /b.html;
        }
    }
    当请求/1.html，最终会访问/2.html
    在location{}内部，遇到break，本location{}内以及后面的所有location{}内的所有指令都不再执行。

    
    示例6（增加last）:
    server{
        listen 80; 
        server_name test.com;
        root /tmp/123.com;
        
        location / {
            rewrite /1.html /2.html last;
            rewrite /2.html /3.html;
        }
        location /2.html
        {
            rewrite /2.html /a.html;
        }
        location /3.html
        {
            rewrite /3.html /b.html;
        }
    }
    当请求/1.html，最终会访问/a.html
    在location{}内部，遇到last，本location{}内后续指令不再执行，而重写后的url再次从头开始，从头到尾匹配一遍规则。
    

#### 结论

* 当rewrite规则在location{}外，break和last作用一样，遇到break或last后，其后续的rewrite/return语句不再执行。但后续有location{}的话，还会近一步执行location{}里面的语句,当然前提是请求必须要匹配该location。
* 当rewrite规则在location{}里，遇到break后，本location{}与其他location{}的所有rewrite/return规则都不再执行。
* 当rewrite规则在location{}里，遇到last后，本location{}里后续rewrite/return规则不执行，但重写后的url再次从头开始执行所有规则，哪个匹配执行哪个。
    
### return指令

    该指令一般用于对请求的客户端直接返回响应状态码。在该作用域内return后面的所有nginx配置都是无效的。
    可以在server、location以及if配置中使用。
    
    除了支持跟状态码，还可以跟字符串或者url链接。

#### 直接返回状态码

    示例1：
    server{
        listen 80;
        server_name www.aming.com;
        return 403;
        rewrite /(.*) /abc/$1;  #该行配置不会被执行。
    }
    
    示例2：
    server {
    .....
    
    if ($request_uri ~ "\.htpasswd|\.bak")
    {
        return 404;
        rewrite /(.*) /aaa.txt;  #该行配置不会被执行。
    }
    //如果下面还有其他配置，会被执行。
    .....
    }


#### 返回字符串

    示例3：
    server{
        listen 80;
        server_name www.aming.com;
        return 200 "hello";
    }
    说明：如果要想返回字符串，必须要加上状态码，否则会报错。
    还可以支持json数据
    
    示例4：
	location ^~ /aming {
        default_type application/json ;
        return 200  '{"name":"aming","id":"100"}';
    }
    
    也支持写一个变量
    
    示例5：
    location /test {
	    return 200 "$host $request_uri";
	}
	
	
#### 返回url

    示例6：
    server{
        listen 80;
        server_name www.aming.com;
        return http://www.baidu.com;
        rewrite /(.*) /abc/$1;  //该行配置不会被执行。
    }
    注意：return后面的url必须是以http://或者https://开头的
    

#### 生成场景实战

    背景：网站被黑了，凡是在百度点击到本网站的请求，全部都跳转到了一个赌博网站。
    通过nginx解决：
    if ($http_referer ~ 'baidu.com') 
    {
        return 200 "<html><script>window.location.href='//$host$request_uri';</script></html>";
    }
    
    如果写成：
    return http://$host$request_uri; 在浏览器中会提示“重定向的次数过多”。


### rewrite规则
    格式：rewrite  regex replacement [flag] 
    
    * rewrite配置可以在server、location以及if配置段内生效
    
    * regex是用于匹配URI的正则表达式，其不会匹配到$host（域名）
    
    * replacement是目标跳转的URI，可以以http://或者https://开头，也可以省略掉$host，直接写$request_uri部分（即请求的链接）
    
    * flag，用来设置rewrite对URI的处理行为，其中有break、last、rediect、permanent，其中break和last在前面已经介绍过，
    rediect和permanent的区别在于，前者为临时重定向(302)，而后者是永久重定向(301)，对于用户通过浏览器访问，这两者的效果是一致的。
    但是，对于搜索引擎蜘蛛爬虫来说就有区别了，使用301更有利于SEO。所以，建议replacemnet是以http://或者https://开头的flag使用permanent。

#### 示例1

    location / {
        rewrite /(.*) http://www.aming.com/$1 permanent;
    }
    说明：.*为正则表达式，用()括起来，在后面的URI中可以调用它，第一次出现的()用$1调用，第二次出现的()用$2调用，以此类推。


#### 示例2

    location / {
        rewrite /.* http://www.aming.com$request_uri permanent;
    }
    说明：在replacement中，支持变量，这里的$request_uri就是客户端请求的链接
    
#### 示例3

    server{
        listen 80;
        server_name www.123.com;
        root /tmp/123.com;
        index index.html;
        rewrite /(.*) /abc/$1 redirect;
    }
    说明：本例中的rewrite规则有问题，会造连续循环，最终会失败，解决该问题有两个方案。
    关于循环次数，经测试发现，curl 会循环50次，chrome会循环80次，IE会循环120次，firefox会循环20次。

#### 示例4

    server{
        listen 80;
        server_name www.123.com;
        root /tmp/123.com;
        index index.html;
        rewrite /(.*) /abc/$1 break;
    }
    说明：在rewrite中使用break，会避免循环。
    
#### 示例5

    server{
        listen 80;
        server_name www.123.com;
        root /tmp/123.com;
        index index.html;
        if ($request_uri !~ '^/abc/')
        {
            rewrite /(.*) /abc/$1 redirect;
        }
    }
    说明：加一个条件限制，也可以避免产生循环
    
    
### nginx常用全局变量

| 变量       | 说明    |
| :--------   | :-----   | 
|$args       |请求中的参数，如www.123.com/1.php?a=1&b=2的$args就是a=1&b=2 |
|$content_length |HTTP请求信息里的"Content-Length" |
|$conten_type    |  HTTP请求信息里的"Content-Type"   |
|$document_root|nginx虚拟主机配置文件中的root参数对应的值|
|$document_uri|当前请求中不包含指令的URI，如www.123.com/1.php?a=1&b=2的$document_uri就是1.php,不包含后面的参数|
|$host|主机头，也就是域名|
|$http_user_agent|客户端的详细信息，也就是浏览器的标识，用curl -A可以指定|
|$http_cookie|客户端的cookie信息|
|$limit_rate|如果nginx服务器使用limit_rate配置了显示网络速率，则会显示，如果没有设置， 则显示0|
|$remote_addr|客户端的公网ip|
|$remote_port|客户端的port|
|$remote_user|如果nginx有配置认证，该变量代表客户端认证的用户名|
|$request_body_file|做反向代理时发给后端服务器的本地资源的名称|
|$request_method|请求资源的方式，GET/PUT/DELETE等|
|$request_filename|当前请求的资源文件的路径名称，相当于是$document_root/$document_uri的组合|
|$request_uri|请求的链接，包括$document_uri和$args|
|$scheme|请求的协议，如ftp,http,https|
|$server_protocol|客户端请求资源使用的协议的版本，如HTTP/1.0，HTTP/1.1，HTTP/2.0等|
|$server_addr|服务器IP地址|
|$server_name|服务器的主机名|
|$server_port|服务器的端口号|
|$uri|和$document_uri相同|
|$http_referer|客户端请求时的referer，通俗讲就是该请求是通过哪个链接跳过来的，用curl -e可以指定|

### rewrite实战

    本部分内容为nginx生产环境中使用的场景示例。
    
#### 域名重定向

    示例1（不带条件的）：
    server{
        listen 80;
        server_name www.2.com;
        root /data/wwwroot/www.2.com;
        rewrite /(.*) http://www.1.com/$1 permanent;
        .......
        
    }
    
    示例2（带条件的）：
    server{
        listen 80;
        server_name www.2.com 2.com;
        root /data/wwwroot/www.2.com;
        if ($host != 'www.2.com')
        {
            rewrite /(.*) http://www.2.com/$1 permanent;
        }
        .......
        
    }
    
    示例3（http跳转到https）：
    server{
        listen 80;
        server_name www.2.com;
        root /data/wwwroot/www.2.com;
        rewrite /(.*) https://www.2.com/$1 permanent;
        .......
        
    }
    
    示例4（域名访问二级目录）
    server{
        listen 80;
        server_name bbs.2.com;
        root /data/wwwroot/www.2.com;
        rewrite /(.*) http://www.2.com/bbs/$1 last;
        .......
        
    }
    
    示例5（静态请求分离）
    server{
        listen 80;
        server_name www.2.com;
        root /data/wwwroot/www.2.com;
        location ~* ^.+.(jpg|jpeg|gif|css|png|js)$
        {
            rewrite /(.*) http://img.2.com/$1 permanent;
        }

        .......
        
    }
    或者：
    server{
        listen 80;
        server_name www.2.com;
        root /data/wwwroot/www.2.com;
        if ( $uri ~* 'jpg|jpeg|gif|css|png|js$')
        {
            rewrite /(.*) http://img.2.com/$1 permanent;
        }

        .......
        
    }
      

#### 防盗链

    示例6
    server{
        listen 80;
        server_name www.2.com;
        location ~* ^.+.(jpg|jpeg|gif|css|png|js|rar|zip|flv)$
        {
            valid_referers none blocked server_names *.2.com 2.com;
            if ($invalid_referer)
            {
                rewrite /(.*) http://img.2.com/images/forbidden.png;
            }
        }

        .......
        
    }
    说明：*这里是通配，跟正则里面的*不是一个意思，none指的是referer不存在的情况（curl -e 测试），
          blocked指的是referer头部的值被防火墙或者代理服务器删除或者伪装的情况，
          该情况下，referer头部的值不以http://或者https://开头（curl -e 后面跟的referer不以http://或者https://开头）。
    或者：
        location ~* ^.+.(jpg|jpeg|gif|css|png|js|rar|zip|flv)$
        {
            valid_referers none blocked server_names *.2.com 2.com;
            if ($invalid_referer)
            {
                return 403;
            }
        }
      
#### 伪静态

    示例7(discuz伪静态)：
    location /  {
        rewrite ^([^\.]*)/topic-(.+)\.html$ $1/portal.php?mod=topic&topic=$2 last;
        rewrite ^([^\.]*)/forum-(\w+)-([0-9]+)\.html$ $1/forum.php?mod=forumdisplay&fid=$2&page=$3 last;
        rewrite ^([^\.]*)/thread-([0-9]+)-([0-9]+)-([0-9]+)\.html$ $1/forum.php?mod=viewthread&tid=$2&extra=page%3D$4&page=$3 last;
        rewrite ^([^\.]*)/group-([0-9]+)-([0-9]+)\.html$ $1/forum.php?mod=group&fid=$2&page=$3 last;
        rewrite ^([^\.]*)/space-(username|uid)-(.+)\.html$ $1/home.php?mod=space&$2=$3 last;
        rewrite ^([^\.]*)/(fid|tid)-([0-9]+)\.html$ $1/index.php?action=$2&value=$3 last;
    }
    
#### rewrite多个条件的并且

    示例8：(nginx不支持if条件的逻辑与/逻辑或运算 ，并且不支持if的嵌套语法，但我们可以用变量的方式来实现)
    location /{
        set $rule 0;
        if ($document_uri !~ '^/abc')
        {
            set $rule "${rule}1";
        }
        if ($http_user_agent ~* 'ie6|firefox')
        {
           set $rule "${rule}2";
        }
        if ($rule = "012")
        {
            rewrite /(.*) /abc/$1 redirect;
        }
    }
    

### location配置
    nginx location语法规则：location [=|~|~*|^~] /uri/ { … }
    nginx的location匹配的变量是$uri

| 符号       | 说明    |
| :--------   | :-----   | 
| = |   表示精确匹配|
| ^~|  表示uri以指定字符或字符串开头|
| ~ |  表示区分大小写的正则匹配|
| ~*|  表示不区分大小写的正则匹配|
|/|  通用匹配，任何请求都会匹配到|


#### 安装echo-nginx-module模块
    为了方便测试location配置，我们安装第三方模块echo-nginx-module，这样就可以在nginx配置中使用echo指令了。
    1. git clone https://github.com/openresty/echo-nginx-module.git
    2. cd /etc/local/src/nginx-1.14.0目录下, 运行make clean。
    3. 通过/usr/local/nginx/sbin/nginx -V 可以查看已有的参数，在此基础上运行
       ./config --add-module=<上面克隆模块的路径> (e.g. ./config --add-module=/usr/local/src/echo-nginx-module)
    4. make; make install完成编译。
    
#### 规则优先级

    =  高于  ^~  高于  ~* 等于 ~  高于  /
    对于等同的优先级，谁写在前面就用谁的
    
#### 规则示例

    location = "/12.jpg" { ... }
    如：
    www.2.com/12.jpg 匹配
    www.2.com/abc/12.jpg 不匹配
    
    location ^~ "/abc/" { ... }
    如：
    www.2.com/abc/123.html 匹配
    www.2.com/a/abc/123.jpg 不匹配
    
    location ~ "png" { ... }
    如：
    www.2.com/aaa/bbb/ccc/123.png 匹配
    www.2.com/aaa/png/123.html 匹配
    
    location ~* "png" { ... }
    如：
    www.2.com/aaa/bbb/ccc/123.PNG 匹配
    www.2.com/aaa/png/123.html 匹配
    
    
    location /admin/ { ... }
    如：
    www.2.com/admin/aaa/1.php 匹配
    www.2.com/123/admin/1.php 不匹配
    
#### 小常识

    有些资料上介绍location支持不匹配 !~，
    如： location !~ 'png'{ ... }
    这是错误的，location不支持 !~
    
    如果有这样的需求，可以通过if来实现，
    如： if ($uri !~ 'png') { ... }
    
    注意：location优先级小于if



### Nginx代理

#### 示意图
![image](/assets/images/blog/nginx-2.png)

![image](/assets/images/blog/nginx-3.png)

![image](/assets/images/blog/nginx-4.png)

![image](/assets/images/blog/nginx-5.png)

#### Nginx正向代理
    Nginx正向代理使用场景并不多见。
    需求场景1：
    如果在机房中，只有一台机器可以联网，其他机器只有内网，内网的机器想用使用yum安装软件包，在能联网的机器上配置一个正向代理即可。
 
 
#### Nginx正向代理配置文件

```
server {
    listen 80 default_server;
    resolver 119.29.29.29; # DNS的IP地址，可以从dns.lisect.com查看
    location /
    {
        proxy_pass http://$host$request_uri;
    }
}
``` 
 
#### Nginx正向代理配置执行说明

* resolver

```    
语法：resolver address;

address为DNS服务器的地址，国内通用的DNS 119.29.29.29为dnspod公司提供。 国际通用DNS 8.8.8.8或者8.8.4.4为google提供。
其他可以参考 http://dns.lisect.com/
    
示例：resolver 119.29.29.29;
```

* default_server

```
之所以要设置为默认虚拟主机，是因为这样就不用设置server_name了，任何域名解析过来都可以正常访问。
```
    
* proxy_pass

```
该指令用来设置要代理的目标url，正向代理服务器设置就保持该固定值即可。关于该指令的详细解释在反向代理配置中。
```
 

#### Nginx反向代理

```
Nginx反向代理在生产环境中使用很多的。

场景1：
域名没有备案，可以把域名解析到香港一台云主机上，在香港云主机做个代理，而网站数据是在大陆的服务器上。

示例1：
server 
{
    listen 80;
    server_name www.apelearn.com;
    
    location /
    {
        proxy_pass http://47.104.7.242/;
        proxy_set_header Host   $host;
        proxy_set_header X-Real-IP      $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

```
示例2：

fp.conf内容:
server {
   listen 80;
   server_name www.test.com;

   location /
   {
       proxy_pass http://127.0.0.1:8080/;
       proxy_set_header Host   $host;
       proxy_set_header X-Real-IP      $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   }
}

default_8080.conf内容:
server {
    listen 8080 default_server;
    root /Users/jiaqi/data/wwwroot/test.com;
    index index.html;
    location /
    {
	echo "8080 default";
    }
}


test.com.conf内容:
server {
    listen 8080;
    server_name www.test.com;
    root /Users/jiaqi/data/wwwroot/test.com;
    index index.html;
}

运行结果:
Jiaqis-MacBook-Pro:nginx-1.14.0 jiaqi$ curl -x127.0.0.1:80 www.test.com
test.com_8080

注释掉fp.conf中的 proxy_set_header Host   $host;结果如下:
Jiaqis-MacBook-Pro:nginx-1.14.0 jiaqi$ sudo /usr/local/nginx/sbin/nginx -s reload
Jiaqis-MacBook-Pro:nginx-1.14.0 jiaqi$ curl -x127.0.0.1:80 www.test.com
8080 default

从示例2可以得出：
如果没有proxy_set_header Host，则访问的就直接是http://127.0.0.1:8080/；
如果设置了proxy_set_header Host &host，则访问的是&host域名，指定的IP是http://127.0.0.1:8080/，就好比
是curl -x 127.0.0.1:8080 &host

如果你想直接不用proxy_set_header并且在proxy_pass后直接写域名如http://www.test.com:8080/,则需要配置/etc/hosts从而解析这个域名。
```

#### 配置说明
##### 1. proxy_pass

    在正向代理中，已经使用过该指令。
    格式很简单： proxy_pass  URL;
    其中URL包含：传输协议（http://, https://等）、主机名（域名或者IP:PORT）、uri。

    示例如下：
    proxy_pass http://www.test.com/;
    proxy_pass http://192.168.200.101:8080/uri;
    proxy_pass unix:/tmp/www.sock;
    
    对于proxy_pass的配置有几种情况需要注意。
    示例2：
    location /aming/
    {
        proxy_pass http://192.168.1.10;
        ...
    }
    
    
    示例3：
    location /aming/
    {
        proxy_pass http://192.168.1.10/;
        ...
    }
    
    示例4：
    location /aming/
    {
        proxy_pass http://192.168.1.10/linux/;
        ...
    }
    
    示例5：
    location /aming/
    {
        proxy_pass http://192.168.1.10/linux;
        ...
    }
    
    假设server_name为www.test.com
    当请求http://www.test.com/aming/a.html的时候，以上示例2-5分别访问的结果是
    
    示例2：http://192.168.1.10/aming/a.html
    
    示例3：http://192.168.1.10/a.html
    
    示例4：http://192.168.1.10/linux/a.html
    
    示例5：http://192.168.1.10/linuxa.html
    


##### 2. proxy_set_header
```
proxy_set_header用来设定被代理服务器接收到的header信息。

语法：proxy_set_header field value;
field为要更改的项目，也可以理解为变量的名字，比如host
value为变量的值

如果不设置proxy_set_header，则默认host的值为proxy_pass后面跟的那个域名或者IP（一般写IP），
比如示例4，请求到后端的服务器上时，完整请求uri为：http://192.168.1.10/linux/a.html

如果设置proxy_set_header，如 proxy_set_header host $host;
比如示例4，请求到后端的服务器完整uri为：http://www.test.com/linux/a.html

proxy_set_header X-Real-IP $remote_addr;和proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
用来设置被代理端接收到的远程客户端IP，如果不设置，则header信息中并不会透传远程真实客户端的IP地址。
可以用如下示例来测试：

示例6(被代理端)
server{
	listen 8080;
	server_name www.test.com;
	root /Users/jiaqi/data/wwwroot/test.com;
	index index.html;
    location / {
	    echo "$host";
	    echo $remote_addr;
	    echo $proxy_add_x_forwarded_for;
	}
}

示例7（代理服务器上）
server {
    listen 80;
    server_name www.test.com;

    location /
    {
	proxy_pass http://127.0.0.1:8080/;
	proxy_set_header host $host;
	proxy_set_header X-Real-IP      $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

Jiaqis-MacBook-Pro:nginx-1.14.0 jiaqi$  curl -x192.168.2.21:80 www.test.com
www.test.com
127.0.0.1
192.168.2.21, 127.0.0.1
```
 
##### 3. proxy_redirect
```
该指令用来修改被代理服务器返回的响应头中的Location头域和“refresh”头域。
语法结构为：
proxy_redirect redirect replacement;
proxy_redirect default;
proxy_redirect off;

示例8：
server {
    listen 80;
    server_name www.test.com;
    index  index.html;

    location /
    {
	proxy_pass http://127.0.0.1:8080;
	proxy_set_header host $host;
	proxy_set_header X-Real-IP      $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

当请求的链接为 http://www.test.com/aming
结果会返回301，定向到了 http://www.test.com:8080/aming/

注意：返回301有几个先决条件
1. location后面必须是/; 
2. proxy_pass后面的URL不能加uri,只能是IP或者IP:port结尾，并不能以/结尾；
3. 访问的uri必须是一个真实存在的目录，如，这里的aming必须是存在的
4. 访问的时候，不能以/结尾，只能是 www.test.com/aming

虽然，这4个条件挺苛刻，但确实会遇到类似的请求。解决方法是，加一行proxy_redirect http://$host:8080/ /;

示例9：
server {
    listen 80;
    server_name www.test.com;
    index  index.html;

    location /
    {
	proxy_pass http://127.0.0.1:8080;
	proxy_set_header host $host;
	proxy_redirect http://$host:8080/ /;
	proxy_set_header X-Real-IP      $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}


```


#### Nginx反向代理buffer和proxy_cache
```
两个都是nginx代理中内存设置相关的参数。
```

##### proxy_buffering设置
```
proxy_buffering主要是实现被代理服务器的数据和客户端的请求异步。
为了方便理解，我们定义三个角色，A为客户端，B为代理服务器，C为被代理服务器。

当proxy_buffering开启，A发起请求到B，B再到C，C反馈的数据先到B的buffer上，
然后B会根据proxy_busy_buffer_size来决定什么时候开始把数据传输给A。在此过程中，如果所有的buffer被写满，
数据将会写入到temp_file中。

相反，如果proxy_buffering关闭，C反馈的数据实时地通过B传输给A。
```

##### 以下配置，都是针对每一个http请求的。
```
1. proxy_buffering  on;
该参数设置是否开启proxy的buffer功能，参数的值为on或者off。
如果这个设置为off，那么proxy_buffers和proxy_busy_buffers_size这两个指令将会失效。 
但是无论proxy_buffering是否开启，proxy_buffer_size都是生效的

2. proxy_buffer_size  4k;
该参数用来设置一个特殊的buffer大小的。
从被代理服务器（C）上获取到的第一部分响应数据内容到代理服务器（B）上，通常是header，就存到了这个buffer中。 
如果该参数设置太小，会出现502错误码，这是因为这部分buffer不够存储header信息。建议设置为4k。

3. proxy_buffers  8  4k;
这个参数设置存储被代理服务器上的数据所占用的buffer的个数和每个buffer的大小。
所有buffer的大小为这两个数字的乘积。

4. proxy_busy_buffer_size 16k;
在所有的buffer里，我们需要规定一部分buffer把自己存的数据传给A，这部分buffer就叫做busy_buffer。
proxy_busy_buffer_size参数用来设置处于busy状态的buffer有多大。

作用：proxy_busy_buffers_size不是独立的空间，他是proxy_buffers和proxy_buffer_size的一部分。
nginx会在没有完全读完后端响应就开始向客户端传送数据，所以它会划出一部分busy状态的buffer来专门向客户端传送数据(建议为proxy_buffers中单个缓冲区的2倍)，
然后它继续从后端取数据。
    
对于B上buffer里的数据何时传输给A，我个人的理解是这样的：
1）如果完整数据大小小于busy_buffer大小，当数据传输完成后，马上传给A；
2）如果完整数据大小不少于busy_buffer大小，则装满busy_buffer后，马上传给A；

5. proxy_temp_path
语法：proxy_temp_path  path [level1 level2 level3]
定义proxy的临时文件存在目录以及目录的层级。

例：proxy_temp_path /usr/local/nginx/proxy_temp 1 2;
其中/usr/local/nginx/proxy_temp为临时文件所在目录，1表示层级1的目录名为一个数字(0-9),2表示层级2目录名为2个数字(00-99)

6. proxy_max_temp_file_size
设置临时文件的总大小，例如 proxy_max_temp_file_size 100M;

7. proxy_temp_file_wirte_size
设置同时写入临时文件的数据量的总大小。通常设置为8k或者16k。

```

##### proxy_buffer示例
```
server
{
    listen 80;
    server_name www.test.com;
    proxy_buffering on;
	proxy_buffer_size 4k;
    proxy_buffers 2 4k;
    proxy_busy_buffers_size 4k;
    proxy_temp_path /tmp/nginx_proxy_tmp 1 2;
	proxy_max_temp_file_size 20M;
	proxy_temp_file_write_size 8k;
	
	location /
	{
	    proxy_pass      http://192.168.10.110:8080/;
        proxy_set_header Host   $host;
        proxy_set_header X-Real-IP      $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}

}
```

##### proxy_cache设置
```
proxy_cache将从C上获取到的数据根据预设规则存放到B上（内存+磁盘）留着备用，
A请求B时，B会把缓存的这些数据直接给A，而不需要再去向C去获取。

proxy_cache相关功能生效的前提是，需要设置proxy_buffering on;

```


##### proxy_cache主要参数
```
1. proxy_cache
语法：proxy_cache zone|off

默认为off，即关闭proxy_cache功能，zone为用于存放缓存的内存区域名称。
例：proxy_cache my_zone;

从nginx 0.7.66版本开始，proxy_cache机制开启后会检测被代理端的HTTP响应头中的"Cache-Control"、"Expire"头域。
如，Cache-Control为no-cache时，是不会缓存数据的。

2. proxy_cache_bypass 
语法：proxy_cache_bypass string;

该参数设定，什么情况下的请求不读取cache而是直接从后端的服务器上获取资源。
这里的string通常为nginx的一些变量。

例：proxy_cahce_bypass $cookie_nocache $arg_nocache $arg_comment;
意思是，如果$cookie_nocache $arg_nocache$arg_comment这些变量的值只要任何一个不为0或者不为空时，
则响应数据不从cache中获取，而是直接从后端的服务器上获取。

3. proxy_no_cache
语法：proxy_no_cache string;

该参数和proxy_cache_bypass类似，用来设定什么情况下不缓存。

例：proxy_no_cache $cookie_nocache $arg_nocache $arg_comment;
表示，如果$cookie_nocache $arg_nocache $arg_comment的值只要有一项不为0或者不为空时，不缓存数据。

4. proxy_cache_key
语法：proxy_cache_key string;

定义cache key，如： proxy_cache_key $scheme$proxy_host$uri$is_args$args; （该值为默认值，一般不用设置）

5. proxy_cache_path
语法：proxy_cache_path path [levels=levels] keys_zone=name:size  [inactive=time] [max_size=size] 

path设置缓存数据存放的路径；

levels设置目录层级，如levels=1:2，表示有两级子目录,第一个目录名取md5值的倒数第一个值，第二个目录名取md5值的倒数第3和倒数第2个值。如下图：
```
![image](/assets/images/blog/nginx-6.png)
```
keys_zone设置内存zone的名字和大小，如keys_zone=my_zone:10m

inactive设置缓存多长时间就失效，当硬盘上的缓存数据在该时间段内没有被访问过，就会失效了，该数据就会被删除，默认为10s。

max_size设置硬盘中最多可以缓存多少数据，当到达该数值时，nginx会删除最少访问的数据。

例：proxy_cache_path /data/nginx_cache/ levels=1:2 keys_zone=my_zone:10m inactive=300s max_size=5g

```

##### proxy_cache示例
```
http 
{
    ...;
    
    proxy_cache_path /data/nginx_cache/ levels=1:2 keys_zone=my_zone:10m inactive=300s max_size=5g;
    
    ...;
    
    server
    {
        listen 80;
        server_name www.test.com;
        proxy_buffering on;
    	proxy_buffer_size 4k;
        proxy_buffers 2 4k;
        proxy_busy_buffers_size 4k;
        proxy_temp_path /tmp/nginx_proxy_tmp 1 2;
    	proxy_max_temp_file_size 20M;
    	proxy_temp_file_write_size 8k;
	
	
	
	    location /
	    {
	        proxy_cache my_zone;
	        proxy_pass      http://127.0.0.1:8080/;
            proxy_set_header Host   $host;
            proxy_set_header X-Real-IP      $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

	    }

    }
}

说明：核心配置为proxy_cache_path那一行。
```

### Nginx负载均衡

```
Nginx通过upstream和proxy_pass实现了负载均衡。本质上也是Nginx的反向代理功能，只不过后端的server为多个。
```

#### 案例一（简单的轮询）
```
upstream www {
    server 172.37.150.109:80;
    server 172.37.150.101:80;
    server 172.37.150.110:80;
}

server {
    listen 80;
    server_name www.test.com;
    location / {
        proxy_pass http://www/;
        proxy_set_header Host   $host;
        proxy_set_header X-Real-IP      $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

说明：当被代理的机器有多台时，需要使用upstream来定义一个服务器组，
其中www名字可以自定义，在后面的proxy_pass那里引用。
这样nginx会将请求均衡地轮询发送给www组内的三台服务器。

```

#### 案例二（带权重轮询+ip_hash算法）
```
upstream www {
    server 172.37.150.109:80 weight=50;
    server 172.37.150.101:80 weight=100;
    server 172.37.150.110:80 weight=50;
    ip_hash;
}

server {
    listen 80;
    server_name www.test.com;
    location / {
        proxy_pass http://www/;
        proxy_set_header Host   $host;
        proxy_set_header X-Real-IP      $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

说明：可以给www组内的三台机器配置权重，权重越高，则分配到的请求越多。
ip_hash为nginx负载均衡算法，原理很简单，它根据请求所属的客户端IP计算得到一个数值，然后把请求发往该数值对应的后端。
所以同一个客户端的请求，都会发往同一台后端，除非该后端不可用了。ip_hash能够达到保持会话的效果。

```

#### 案例三（upstream其他配置）
```
upstream www {
        server 172.37.150.109:80 weight=50 max_fails=3 fail_timeout=30s;
        server 172.37.150.101:80 weight=100;
        server 172.37.150.110:80 down;
        server 172.37.150.110:80 backup;
}
server
{
    listen 80;
    server_name www.test.com;
    location / {
        proxy_next_upstream off;
        proxy_pass http://www/;
        proxy_set_header Host   $host;
        proxy_set_header X-Real-IP      $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

说明：down，表示当前的server不参与负载均衡；
backup，为预留的机器，当其他的server（非backup）出现故障或者忙的时候，才会请求backup机器;
max_fails，允许请求失败的次数，默认为1。当失败次数达到该值，就认为该机器down掉了。 失败的指标是由proxy_next_upstream模块定义，其中404状态码不认为是失败。
fail_timeout，定义失败的超时时间，也就是说在该时间段内达到max_fails，才算真正的失败。默认是10秒。

proxy_next_upstream(Specifies in which cases a request should be passed to the next server)，
通过后端服务器返回的响应状态码，表示服务器死活，可以灵活控制后端机器是否加入分发列表。
语法: proxy_next_upstream error | timeout | invalid_header | http_500 | http_502 | http_503 | http_504 |http_404 | off ...; 
默认值: proxy_next_upstream error timeout

error      # 和后端服务器建立连接时，或者向后端服务器发送请求时，或者从后端服务器接收响应头时，出现错误
timeout    # 和后端服务器建立连接时，或者向后端服务器发送请求时，或者从后端服务器接收响应头时，出现超时
invalid_header  # 后端服务器返回空响应或者非法响应头
http_500   # 后端服务器返回的响应状态码为500
http_502   # 后端服务器返回的响应状态码为502
http_503   # 后端服务器返回的响应状态码为503
http_504   # 后端服务器返回的响应状态码为504
http_404   # 后端服务器返回的响应状态码为404
off        # 停止将请求发送给下一台后端服务器

```

#### 案例四（根据不同的uri）
```
    upstream aa.com {         
                      server 192.168.0.121;
                      server 192.168.0.122;  
     }
    upstream bb.com {  
                       server 192.168.0.123;
                       server 192.168.0.124;
    }
    server {
        listen       80;
        server_name  www.test.com;
        location ~ aa.php
        {
            proxy_pass http://aa.com/;
            proxy_set_header Host   $host;
            proxy_set_header X-Real-IP      $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        location ~ bb.php
        {
              proxy_pass http://bb.com/;
              proxy_set_header Host   $host;
              proxy_set_header X-Real-IP      $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        location /
        {
              proxy_pass http://bb.com/;
              proxy_set_header Host   $host;
              proxy_set_header X-Real-IP      $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}

说明：请求aa.php的，会到aa.com组，请求bb.php的会到bb.com，其他请求全部到bb.com。

```

#### 案例五（根据不同的目录）
```
upstream aaa.com
{
            server 192.168.111.6;
}
upstream bbb.com
{
            server 192.168.111.20;
}
server {
        listen 80;
        server_name www.test.com;
        location /aaa/
        {
            proxy_pass http://aaa.com/aaa/;
            proxy_set_header Host   $host;
            proxy_set_header X-Real-IP      $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        location /bbb/
        {
            proxy_pass http://bbb.com/bbb/;
            proxy_set_header Host   $host;
            proxy_set_header X-Real-IP      $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        location /
        {
            proxy_pass http://bbb.com/;
            proxy_set_header Host   $host;
            proxy_set_header X-Real-IP      $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}
```

### Nginx访问控制

#### deny和allow
```
Nginx的deny和allow指令是由ngx_http_access_module模块提供，Nginx安装默认内置了该模块。
除非在安装时有指定 --without-http_access_module。
```

##### 语法
```
语法：allow/deny address | CIDR | unix: | all

它表示，允许/拒绝某个ip或者一个ip段访问.如果指定unix:,那将允许socket的访问。
注意：unix在1.5.1中新加入的功能。

在nginx中，allow和deny的规则是按顺序执行的。

```

##### 示例
```
示例1：
location /
{
    allow 192.168.0.0/24;
    allow 127.0.0.1;
    deny all;
}

说明：这段配置值允许192.168.0.0/24网段和127.0.0.1的请求，其他来源IP全部拒绝。

示例2：
location ~ "admin"
{
    allow 110.21.33.121;
    deny all
}
说明：访问的uri中包含admin的请求，只允许110.21.33.121这个IP的请求。

```

#### 基于location的访问控制
```
在生产环境中，我们会对某些特殊的请求进行限制，比如对网站的后台进行限制访问。
这就用到了location配置。
```

##### 示例1
```
location /test/
{
    deny all;
}

说明：针对/test/目录，全部禁止访问，这里的deny all可以改为return 403.
```

##### 示例2
```
location ~ ".bak|\.ht"
{
    return 403;
}
说明：访问的uri中包含.bak字样的或者包含.ht的直接返回403状态码。

测试链接举例：
1. www.test.com/123.bak
2. www.test.com/aming/123/.htalskdjf
```

##### 示例3
```
location ~ (data|cache|tmp|image|attachment).*\.php$
{
    deny all;
}

说明：请求的uri中包含data、cache、tmp、image、attachment并且以.php结尾的，全部禁止访问。

测试链接举例：
1. www.test.com/aming/cache/1.php
2. www.test.com/image/123.phps
3. www.test.com/aming/datas/1.php

```

### Nginx基于document_uri的访问控制
```
这就用到了变量$document_uri，根据前面所学内容，该变量等价于$uri，其实也等价于location匹配。
```

##### 示例1
```
if ($document_uri ~ "/admin/")
{
    return 403;
}

说明：当请求的uri中包含/admin/时，直接返回403.

if结构中不支持使用allow和deny。

测试链接：
1. www.test.com/123/admin/1.html 匹配
2. www.test.com/admin123/1.html  不匹配
3. www.test.com/admin.php  不匹配
```

##### 示例2
```
if ($document_uri = /admin.php)
{
    return 403;
}

说明：请求的uri为/admin.php时返回403状态码。

测试链接：
1. www.test.com/admin.php 匹配
2. www.test.com/123/admin.php  不匹配
```

##### 示例3
```
if ($document_uri ~ '/data/|/cache/.*\.php$')
{
    return 403;
}

说明：请求的uri包含data或者cache目录，并且是php时，返回403状态码。

测试链接：
1. www.test.com/data/123.php  匹配
2. www.test.com/cache1/123.php 不匹配
```

### Nginx基于request_uri访问控制
```
$request_uri比$docuemnt_uri多了请求的参数。
主要是针对请求的uri中的参数进行控制。
```

##### 示例
```
if ($request_uri ~ "gid=\d{9,12}")
{
    return 403;
}

说明：\d{9,12}是正则表达式，表示9到12个数字，例如gid=1234567890就符号要求。

测试链接：
1. www.test.com/index.php?gid=1234567890&pid=111  匹配
2. www.test.com/gid=123  不匹配

背景知识：
曾经有一个客户的网站cc攻击，对方发起太多类似这样的请求：/read-123405150-1-1.html
实际上，这样的请求并不是正常的请求，网站会抛出一个页面，提示帖子不存在。
所以，可以直接针对这样的请求，return 403状态码。

```


### Nginx基于user_agent的访问控制
```
user_agent大家并不陌生，可以简单理解成浏览器标识，包括一些蜘蛛爬虫都可以通过user_agent来辨识。
通过观察访问日志，可以发现一些搜索引擎的蜘蛛对网站访问特别频繁，它们并不友好。
为了减少服务器的压力，其实可以把除主流搜索引擎蜘蛛外的其他蜘蛛爬虫全部封掉。
另外，一些cc攻击，我们也可以通过观察它们的user_agent找到规律。

```
##### 示例
```
if ($user_agent ~ 'YisouSpider|MJ12bot/v1.4.2|YoudaoBot|Tomato')
{
    return 403;
}
说明：user_agent包含以上关键词的请求，全部返回403状态码。

测试：
1. curl -A "123YisouSpider1.0"
2. curl -A "MJ12bot/v1.4.1"

```

### Nginx基于http_referer的访问控制
```
在前面讲解rewrite时，曾经用过该变量，当时实现了防盗链功能。
其实基于该变量，我们也可以做一些特殊的需求。

```

##### 示例
```
背景：网站被黑挂马，搜索引擎收录的网页是有问题的，当通过搜索引擎点击到网站时，却显示一个博彩网站。
由于查找木马需要时间，不能马上解决，为了不影响用户体验，可以针对此类请求做一个特殊操作。
比如，可以把从百度访问的链接直接返回404状态码，或者返回一段html代码。

if ($http_referer ~ 'baidu.com')
{
    return 404;
}

或者

if ($http_referer ~ 'baidu.com')
{
    return 200 "<html><script>window.location.href='//$host$request_uri';</script></html>";
}
```


#### Nginx的限速
```
可以通过ngx_http_limit_conn_module和ngx_http_limit_req_module模块来实现限速的功能。
```

##### ngx_http_limit_conn_module
```
该模块主要限制下载速度。
```
###### 1. 并发限制
```
配置示例
http
{
    ...
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    limit_conn_status 503;
    limit_conn_log_level error;
    ...
    server
    {
        listen 80;
        server_name www.2.com;
        root /data/wwwroot/www.2.com;
        access_log /tmp/2.log;
        location /
        {
            limit_conn addr 1;
        }  
    }
}
说明：首先用limit_conn_zone定义了一个内存区块索引aming，大小为10m，它以$binary_remote_addr作为key。
该配置只能在http里面配置，不支持在server里配置。

limit_conn 定义针对aming这个zone，并发连接为10个。在这需要注意一下，这个10指的是单个IP的并发最多为10个。
测试: 可以通过ab -c 1 -n 10 http://www.2.com/来测试，然后查看2.log日志文件。
```
###### 2. 速度限制
```
server {
    listen 80;
    server_name www.2.com;
    root /Users/jiaqi/data/wwwroot/www.2.com;
    access_log /Users/jiaqi/tmp/2.log;
    location /
    {
       limit_rate_after 512k;
       limit_rate 150k;
    }
}
说明：limit_rate_after定义当一个文件下载到指定大小（本例中为512k）之后开始限速；
limit_rate 定义下载速度为150k/s。

注意：这两个参数针对每个请求限速。
测试:可以在www.2.com目录下加一个123.tar.gz包，然后通过google浏览器下载，查看下载速度。
```

##### ngx_http_limit_req_module

```
该模块主要用来限制请求数。
```
###### 1. limit_req_zone
```
语法: limit_req_zone $variable zone=name:size rate=rate;
默认值: none
配置段: http

设置一块共享内存限制域用来保存键值的状态参数。 特别是保存了当前超出请求的数量。 
键的值就是指定的变量（空值不会被计算）。
如limit_req_zone $binary_remote_addr zone=two:10m rate=2r/s;

说明：区域名称为one，大小为10m，平均处理的请求频率不能超过每秒一次,键值是客户端IP。
使用$binary_remote_addr变量， 可以将每条状态记录的大小减少到64个字节，这样1M的内存可以保存大约1万6千个64字节的记录。
如果限制域的存储空间耗尽了，对于后续所有请求，服务器都会返回 503 (Service Temporarily Unavailable)错误。
速度可以设置为每秒处理请求数和每分钟处理请求数，其值必须是整数，
所以如果你需要指定每秒处理少于1个的请求，2秒处理一个请求，可以使用 “30r/m”。

```


###### 2. limit_req
```
语法: limit_req zone=name [burst=number] [nodelay];
默认值: —
配置段: http, server, location

设置对应的共享内存限制域和允许被处理的最大请求数阈值。 
如果请求的频率超过了限制域配置的值，请求处理会被延迟，所以所有的请求都是以定义的频率被处理的。 
超过频率限制的请求会被延迟，直到被延迟的请求数超过了定义的阈值，
这时，这个请求会被终止，并返回503 (Service Temporarily Unavailable) 错误。

这个阈值的默认值为0。如：
limit_req_zone $binary_remote_addr zone=two:10m rate=2r/s;
server {
    listen 80;
    server_name www.2.com;
    root /Users/jiaqi/data/wwwroot/www.2.com;
    access_log /Users/jiaqi/tmp/2.log;
    limit_req zone=two burst=5;
}

运行ab -n 10 -c 10  http://www.2.com/，查看2.log日志:
127.0.0.1 - - [14/Nov/2019:21:54:51 -0500] "GET / HTTP/1.0" 200 10 "-" "ApacheBench/2.3"
127.0.0.1 - - [14/Nov/2019:21:54:51 -0500] "GET / HTTP/1.0" 503 213 "-" "ApacheBench/2.3"
127.0.0.1 - - [14/Nov/2019:21:54:51 -0500] "GET / HTTP/1.0" 503 213 "-" "ApacheBench/2.3"
127.0.0.1 - - [14/Nov/2019:21:54:51 -0500] "GET / HTTP/1.0" 503 213 "-" "ApacheBench/2.3"
127.0.0.1 - - [14/Nov/2019:21:54:51 -0500] "GET / HTTP/1.0" 503 213 "-" "ApacheBench/2.3"
127.0.0.1 - - [14/Nov/2019:21:54:52 -0500] "GET / HTTP/1.0" 200 10 "-" "ApacheBench/2.3"
127.0.0.1 - - [14/Nov/2019:21:54:52 -0500] "GET / HTTP/1.0" 200 10 "-" "ApacheBench/2.3"
127.0.0.1 - - [14/Nov/2019:21:54:53 -0500] "GET / HTTP/1.0" 200 10 "-" "ApacheBench/2.3"
127.0.0.1 - - [14/Nov/2019:21:54:53 -0500] "GET / HTTP/1.0" 200 10 "-" "ApacheBench/2.3"
127.0.0.1 - - [14/Nov/2019:21:54:54 -0500] "GET / HTTP/1.0" 200 10 "-" "ApacheBench/2.3"

限制平均每秒不超过一个请求，同时允许超过频率限制的请求数不多于5个。

如果不希望超过的请求被延迟，可以用nodelay参数,如：

limit_req zone=aming burst=5 nodelay;
```

##### 设定白名单IP
```
如果是针对公司内部IP或者lo（127.0.0.1）不进行限速，如何做呢？这就要用到geo模块了。

假如，预把127.0.0.1和192.168.100.0/24网段设置为白名单，需要这样做。
在http { }里面增加：
geo $limited {
    default 1;
    127.0.0.1/32 0;
    192.168.100.0/24 0;
}

map $limited $limit {
	1 $binary_remote_addr;
    0 "";
}

原来的 “limit_req_zone $binary_remote_addr ” 改为“limit_req_zone $limit”

完整示例：

http {
	geo $limited {
		default 1;
		127.0.0.1/32 0;
		192.168.100.0/24 0;
	}

	map $limited $limit {
		1 $binary_remote_addr;
		0 "";
	}
    
    limit_req_zone $limit zone=aming:10m rate=1r/s;

    server {
        location  ^~ /download/ {  
            limit_req zone=aming burst=5;
        }
    }
}
```

### Nginx用户认证
```
当访问一些私密资源时，最好配置用户认证，增加安全性。
```
##### 步骤和示例
* 安装httpd
```
yum install -y httpd
```

* 使用htpasswd生产密码文件
```
htpasswd -c /usr/local/nginx/conf/htpasswd user1
```

* 配置nginx用户认证
```
server {
    listen 80;
    server_name www.1.com;
    root /Users/jiaqi/data/wwwroot/www.1.com;

    location /admin/
    {
	auth_basic "Auth";
	auth_basic_user_file /usr/local/nginx/conf/htpasswd;
    }
}
```

* 测试
```
curl -uuser:passwd www.aminglinux.com/admin/1.html
```

### Nginx之SSL配置

#### CA证书
##### 先来一个例子
```
A公司的小明被派到B公司办事情。B公司如何信任小明是A公司派来的呢？
```
###### 普通介绍信
```
为了让B公司信任小明，A公司特意给小明开了一封介绍信，在信件中详细说明了小明的特征以及小明过来的目的，
并且声明这个小明确实是A公司派来的，除此之外还要有一个A公司的公章。
这样B公司前台小姐姐拿到介绍信后，通过信件内容和A公司公章就能判断出小明确实是A公司派来的员工。 

那万一A公司公章是假的呢？毕竟公章伪造太容易了，这样岂不是会存在问题。咱们就暂且认为公章这种东西很难伪造，
否则故事无法继续喽。
```
###### 引入第三方中介公司
```
好，回到刚才的话题。如果和B公司有业务往来的公司很多，每个公司的公章都不同，那B公司的前台小姐姐就要懂得分辨各种公章，
非常滴麻烦。所以，有某个中介公司C，发现了这个商机。C公司专门开设了一项“代理公章”的业务。
　　
于是今后，A公司的业务员去B公司，需要带2个介绍信：
介绍信1（含有C公司的公章及A公司的公章。并且特地注明：C公司信任A公司。）
介绍信2（仅含有A公司的公章，然后写上：兹有xxx先生/女士前往贵公司办理业务，请给予接洽......。）

这样不是增加麻烦了吗？有啥好处呢？
主要的好处在于，对于接待公司的前台，就不需要记住各个公司的公章分别是啥样子的，她只要记住中介公司C的公章即可。
当她拿到两份介绍信之后，先对介绍信1的C公章验明正身，确认无误之后，再比对“介绍信1”和“介绍信2”的两个A公章是否一致。
如果是一样的，那就可以证明“介绍信2”是可以信任的了。
```
##### 相关专业术语的解释
```
下面就着上面的例子，把相关的名词，作一些解释。
```
###### 什么是证书？
```
证书，英文也叫“digital certificate”或“public key certificate”。
它是用来证明某某东西确实是某某的东西，通俗地说，证书就好比例子里面的公章。通过公章，
可以证明该介绍信确实是对应的公司发出的。
理论上，人人都可以找个证书工具，自己做一个证书。那如何防止坏人自己制作证书出来骗人呢？
```
###### 什么是CA？
```
CA 是“Certificate Authority”的缩写，也叫“证书授权中心”。它是负责管理和签发证书的第三方机构，
就好比例子里面的中介C公司。
一般来说，CA必须是所有行业和所有公众都信任的、认可的。因此它必须具有足够的权威性。
就好比A、B两公司都必须信任C公司，才会找C公司作为公章的中介。
```
###### 什么是CA证书？
```
CA证书，顾名思义，就是CA颁发的证书。

前面已经说了，人人都可以找工具制作证书。但是你一个小破孩制作出来的证书是没啥用处的。
因为你不是权威的CA机关，你自己搞的证书不具有权威性。
这就好比上述的例子里，某个坏人自己刻了一个公章，盖到介绍信上。但是别人一看，
不是受信任的中介公司的公章，就不予理睬。
```
###### 什么是证书之间的信任关系？
```
在开篇的例子里谈到，引入中介后，业务员要同时带两个介绍信。第一个介绍信包含了两个公章，并注明，公章C信任公章A。
证书间的信任关系，就和这个类似。就是用一个证书来证明另一个证书是真实可信滴。
```

###### 什么是证书信任链？
```
实际上，证书之间的信任关系，是可以嵌套的。
比如，C信任A1，A1信任A2，A2信任A3......这个叫做证书的信任链。
只要你信任链上的头一个证书，那后续的证书，都是可以信任滴。
```

###### 什么是根证书？
```
根证书的英文叫“root certificate”，为了说清楚根证书是咋回事，再来看个稍微复杂点的例子。
假设C证书信任A和B；然后A信任A1和A2；B信任B1和B2。则它们之间，构成如下的一个树形关系（一个倒立的树）。
```
![image](/assets/images/blog/nginx-7.png)
```
处于最顶上的树根位置的那个证书，就是“根证书”。除了根证书，其它证书都要依靠上一级的证书，来证明自己。
那谁来证明“根证书”可靠呢？
实际上，根证书自己证明自己是可靠滴（或者换句话说，根证书是不需要被证明滴）。
聪明的同学此刻应该意识到了：根证书是整个证书体系安全的根本。
所以，如果某个证书体系中，根证书出了问题（不再可信了），那么所有被根证书所信任的其它证书，也就不再可信了。
```

##### 证书有啥用？

CA证书的作用有很多，只列出常用的几个。

* 验证网站是否可信（针对HTTPS）
```
通常，我们如果访问某些敏感的网页（比如用户登录的页面），其协议都会使用HTTPS而不是HTTP,因为HTTP协议是明文的，
一旦有坏人在偷窥你的网络通讯，他/她就可以看到网络通讯的内容（比如你的密码、银行帐号、等）。
而 HTTPS 是加密的协议，可以保证你的传输过程中，坏蛋无法偷窥。
但是，千万不要以为，HTTPS协议有了加密，就可高枕无忧了。
假设有一个坏人，搞了一个假的网银的站点，然后诱骗你上这个站点。
假设你又比较单纯，一不留神，就把你的帐号，口令都输入进去了。那这个坏蛋的阴谋就得逞了。
为了防止坏人这么干，HTTPS 协议除了有加密的机制，还有一套证书的机制。通过证书来确保，某个站点确实就是某个站点。
有了证书之后，当你的浏览器在访问某个HTTPS网站时，会验证该站点上的CA证书（类似于验证介绍信的公章）。
如果浏览器发现该证书没有问题（证书被某个根证书信任、证书上绑定的域名和该网站的域名一致、证书没有过期），
那么页面就直接打开，否则的话，浏览器会给出一个警告，告诉你该网站的证书存在某某问题，是否继续访问该站点。
```
* 验证文件是否可信


#### SSL原理
```
要想弄明白SSL认证原理，首先要对CA有有所了解，它在SSL认证过程中有非常重要的作用。
说白了，CA就是一个组织，专门为网络服务器颁发证书的，国际知名的CA机构有VeriSign、Symantec，国内的有GlobalSign。
每一家CA都有自己的根证书，用来对它所签发过的服务器端证书进行验证。

如果服务器提供方想为自己的服务器申请证书，它就需要向CA机构提出申请。
服务器提供方向CA提供自己的身份信息，CA判明申请者的身份后，就为它分配一个公钥，
并且CA将该公钥和服务器身份绑定在一起，并为之签字，这就形成了一个服务器端证书。

如果一个用户(浏览器)想鉴别另一个证书的真伪，他就用CA的公钥对那个证书上的签字进行验证，一旦验证通过，该证书就被认为是有效的。
证书实际是由证书签证机关（CA）签发的对用户的公钥的认证。

证书的内容包括：电子签证机关的信息、公钥用户信息、公钥、权威机构的签字和有效期等等。
目前，证书的格式和验证方法普遍遵循X.509国际标准。
```

##### 申请证书过程
![image](/assets/images/blog/nginx-8.png)
```
首先要有一个CA根证书，然后用CA根证书来签发用户证书。
用户进行证书申请：
1. 先生成一个私钥
2. 用私钥生成证书请求(证书请求里应含有公钥信息)
3. 利用证书服务器的CA根证书来签发证书

这样最终拿到一个由CA根证书签发的证书，其实证书里仅有公钥，而私钥是在用户手里的。
```

##### SSL工作流程（单向）
![image](/assets/images/blog/nginx-9.png)
```
1.客户端say hello 服务端
2.服务端将证书、公钥等发给客户端
3.客户端CA验证证书，成功继续、不成功弹出选择页面
4.客户端告知服务端所支持的加密算法
5.服务端选择最高级别加密算法明文通知客户端
6.客户端生成随机对称密钥key，使用服务端公钥加密发送给服务端
7.服务端使用私钥解密，获取对称密钥key
8.后续客户端与服务端使用该密钥key进行加密通信
```

##### SSL工作流程（双向）
单向认证，仅仅是客户端需要检验服务端证书是否是正确的，而服务端不会检验客户端证书是否是正确的。
双向认证，指客户端验证服务器端证书，而服务器也需要通过CA的公钥证书来验证客户端证书。

双向验证的过程：
```
1.客户端say hello 服务端
2.服务端将证书、公钥等发给客户端
3.客户端CA验证证书，成功继续、不成功弹出选择页面
4.客户端将自己的证书和公钥发送给服务端
5.服务端验证客户端证书，如不通过直接断开连接
6.客户端告知服务端所支持的加密算法
7.服务端选择最高级别加密算法使用客户端公钥加密后发送给客户端
8.客户端收到后使用私钥解密并生成随机对称密钥key，使用服务端公钥加密发送给服务端
9.服务端使用私钥解密，获取对称密钥key
10.后续客户端与服务端使用该密钥key进行加密通信
```

#### 自制ca证书
##### CA根证书
```
# mkdir /etc/pki/ca_test //创建CA更证书的目录

# cd /etc/pki/ca_test

# mkdir root server client newcerts  //创建几个相关的目录

# echo 01 > serial   //定义序列号为01

# echo 01 > crlnumber  //定义crl号为01

# touch index.txt  //创建index.txt

# cd ..

# vi tls/openssl.cnf  //改配置文件
default_ca     = CA_default 改为 default_ca     = CA_test
[ CA_default ] 改为 [ CA_test ]
dir             = /etc/pki/CA  改为  dir             = /etc/pki/ca_test
certificate	= $dir/cacert.pem  改为 certificate	= $dir/root/ca.crt
private_key	= $dir/private/cakey.pe 改为  private_key	= $dir/root/ca.key

# openssl genrsa -out /etc/pki/ca_test/root/ca.key  //生成私钥

# openssl req -new -key /etc/pki/ca_test/root/ca.key -out /etc/pki/ca_test/root/ca.csr   
//生成请求文件，会让我们填写一些指标,这里要注意：如果在这一步填写了相应的指标，
比如Country Name、State or Province Name、hostname。

# openssl x509 -req -days 3650 -in /etc/pki/ca_test/root/ca.csr -signkey /etc/pki/ca_test/root/ca.key -out /etc/pki/ca_test/root/ca.crt 
//生成crt文件
```

##### 生成server端证书
```
# cd /etc/pki/ca_test/server

# openssl genrsa -out server.key   //生成私钥文件

# openssl req -new -key server.key -out server.csr//生成证书请求文件，填写信息需要和ca.csr中的Organization Name保持一致

# openssl ca -in server.csr -cert /etc/pki/ca_test/root/ca.crt -keyfile /etc/pki/ca_test/root/ca.key -out server.crt -days 3650  
//用根证书签名server.csr，最后生成公钥文件server.crt，此步骤会有两个地方需要输入y
Sign the certificate? [y/n]:y
1 out of 1 certificate requests certified, commit? [y/n]y

```

##### 生成客户端证书
```
如果做ssl的双向认证，还需要给客户端生成一个证书，步骤和上面的基本一致
# cd /etc/pki/ca_test/client

# openssl genrsa -out  client.key  //生成私钥文件

# openssl req -new  -key client.key -out client.csr  //生成请求文件，填写信息需要和ca.csr中的Organization Name保持一致

# openssl ca -in client.csr -cert /etc/pki/ca_test/root/ca.crt -keyfile /etc/pki/ca_test/root/ca.key -out client.crt -days 3650 
//签名client.csr, 生成client.crt，此步如果出现
failed to update database
TXT_DB error number 2

需执行：
# sed -i 's/unique_subject = yes/unique_subject = no/' /etc/pki/ca_test/index.txt.attr

执行完，再次重复执行签名client.csr那个操作
```

#### Nginx单向ssl配置
配置示例:
```
cp /etc/pki/ca_test/server/server.* /usr/local/nginx/conf/
{
    listen 443 ssl;
    server_name www.123.com;
    index index.html index.php;
    root /data/wwwroot/123.com;
    ssl on;
    ssl_certificate server.crt;
    ssl_certificate_key server.key;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ALL:!DH:!EXPORT:!RC4:+HIGH:+MEDIUM:!eNULL;
    ssl_prefer_server_ciphers on;
    ...
}
```
##### 配置说明
```
1. 443端口为ssl监听端口。
2. ssl on表示打开ssl支持。
3. ssl_certificate指定crt文件所在路径，如果写相对路径，必须把该文件和nginx.conf文件放到一个目录下。
4. ssl_certificate_key指定key文件所在路径。
5. ssl_protocols指定SSL协议。
6. ssl_ciphers配置ssl加密算法，多个算法用:分隔，ALL表示全部算法，!表示不启用该算法，+表示将该算法排到最后面去。
7. ssl_prefer_server_ciphers 如果不指定默认为off，当为on时，在使用SSLv3和TLS协议时，服务器加密算法将优于客户端加密算法。
```

#### Nginx配置双向认证
```
cp /etc/pki/ca_test/root/ca.crt /usr/local/nginx/conf/
配置示例：
{
    listen 443 ssl;
    server_name www.aminglinux.com;
    index index.html index.php;
    root /data/wwwroot/aminglinux.com;
    ssl on;
    ssl_certificate server.crt;
    ssl_certificate_key server.key;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ALL:!DH:!EXPORT:!RC4:+HIGH:+MEDIUM:!eNULL;
    ssl_prefer_server_ciphers on;
    ssl_client_certificate ca.crt; //这里的ca.crt是根证书公钥文件
    ssl_verify_client on;
    ...
}

```

##### 客户端（浏览器）操作
```
如果不进行以下操作，浏览器会出现400错误。400 Bad Request（No required SSL certificate was sent）
首先需要将client.key转换为pfx(p12)格式

# cd /etc/pki/ca_test/client
# openssl pkcs12 -export -inkey client.key -in client.crt -out client.pfx  //这一步需要输入一个自定义密码，一会在windows上安装的时候要用到，需要记一下。

然后将client.pfx拷贝到windows下，双击即可安装。

也可以直接curl测试：
curl -k --cert /etc/pki/ca_test/client/client.crt  --key /etc/pki/ca_test/client/client.key https://www.123.com/index.html
```

### Nginx日志配置

#### Nginx的错误日志
```
Nginx错误日志平时不用太关注，但是一旦出了问题，就需要借助错误日志来判断问题所在。

配置参数格式：error_log /path/to/log level;
```

##### Nginx错误日志级别
```
常见的错误日志级别有debug | info | notice | warn | error | crit | alert | emerg
级别越高记录的信息越少，如果不定义，默认级别为error.

它可以配置在main、http、server、location段里。

如果在配置文件中定义了两个error_log，在同一个配置段里的话会产生冲突，所以同一个段里只允许配置一个error_log。
但是，在不同的配置段中出现是没问题的。
```

##### Nginx错误日志示例
```
error_log  /var/log/nginx/error.log crit;

如果要想彻底关闭error_log，需要这样配置
error_log /dev/null;

在location和main中都定义了error_log(debug级别)，然后重新加载nginx，发现main中定义的日志文件有新的内容，而location中定义的日志没有内容；
如果通过访问匹配某个location的uri，发现location对应的日志出现内容，而main对应的没有。
```

#### Nginx访问日志格式
```
Nginx访问日志可以设置自定义的格式，来满足特定的需求。
```

##### 访问日志格式示例
```
示例1
    log_format combined_realip '$remote_addr $http_x_forwarded_for [$time_local]'
    '$host "$request_uri" $status'
    '"$http_referer" "$http_user_agent"';

示例2
    log_format main '$remote_addr [$time_local] '
    '$host "$request_uri" $status "$request"'
    '"$http_referer" "$http_user_agent" "$request_time"';

若不配置log_format或者不在access_log配置中指定log_format，则默认格式为：
    '$remote_addr - $remote_user [$time_local] "$request" '
    '$status $body_bytes_sent "$http_referer" '
    '"$http_user_agent";

```
##### 常见变量
| 变量       | 说明    |
| :--------   | :-----   | 
|$time_local|  通用日志格式下的本地时间；（服务器时间）|
|$remote_addr|客户端（用户）IP地址|
|$status| 请求状态码，如200，404，301，302等|
|$body_bytes_sent |发送给客户端的字节数，不包括响应头的大小|
|$bytes_sent|发送给客户端的总字节数|
|$request_length|请求的长度（包括请求行，请求头和请求正文）|
|$request_time |请求处理时间，单位为秒，小数的形式|
|$upstream_addr|集群轮询地址|
|$upstream_response_time|指从Nginx向后端（php-cgi)建立连接开始到接受完数据然后关闭连接为止的时间|
|$remote_user|用来记录客户端用户名称|
|$request |请求方式（GET或者POST等）+URL（包含$request_method,$host,$request_uri）|
|$http_user_agent|用户浏览器标识|
|$http_host |请求的url地址（目标url地址）的host|
|$host|等同于$http_host|
|$http_referer|来源页面，即从哪个页面转到本页，如果直接在浏览器输入网址来访问，则referer为空|
|$uri |请求中的当前URI(不带请求参数，参数位于$args)，不同于浏览器传递的$request_uri的值，它可以通过内部重定向，或者使用index指令进行修改。|
|$document_uri|等同于$uri|
|$request_uri|比$uri多了参数，即$uri+$args|
|$http_x_forwarded_for|如果使用了代理，这个参数会记录代理服务器的ip和客户端的ip|


#### Nginx访问日志配置
```
web服务器的访问日志是非常重要的，我们可以通过访问日志来分析用户的访问情况，
也可以通过访问日志发现一些异常访问，比如cc攻击。

格式： access_log /path/to/logfile format;

access_log可以配置到http, server, location配置段中。

```

##### 配置示例
```
server 
{
    listen 80;
    server_name www.test.com;
    root /data/wwwroot/www.test.com;
    index index.html index.php;
    access_log /data/logs/www.test.com_access.log main;
}
说明：若不指定log_format，则按照默认的格式写日志。

```


#### Nginx访问日志过滤
```
一个网站，会包含很多元素，尤其是有大量的图片、js、css等静态元素。
这样的请求其实可以不用记录日志。
```
##### 配置示例
```
location ~* ^.+\.(gif|jpg|png|css|js)$ 
{
    access_log off;
}

或
location ~* ^.+\.(gif|jpg|png|css|js)$                                      
{
    access_log /dev/null;
}
```


#### Nginx访问日志切割
```
如果任由访问日志写下去，日志文件会变得越来越大，甚至是写满磁盘。
所以，我们需要想办法把日志做切割，比如每天生成一个新的日志，旧的日志按规定时间删除即可。

实现日志切割可以通过写shell脚本或者系统的日志切割机制实现。
```
##### shell脚本切割Nginx日志(将所有日志定义在同一目录下可以方便切割)
```
切割脚本内容：
#!/bin/bash
logdir=/var/log/nginx  //定义日志路径(所有日志都已经在配置文件中定义在此路径下)
prefix=`date -d "-1 day" +%y%m%d`  //定义切割后的日志前缀
cd $logdir  
for f in `ls *.log`
do
   mv $f $f-$prefix  //把日志改名
done
/bin/kill -USR1 $(cat /usr/local/nginx/logs/nginx.pid 2>/dev/null) 2>/dev/null  //生成新的日志
bzip2 *$prefix  //压缩日志
find . -type f -mtime +180 |xargs /bin/rm -f  //删除超过180天的老日志

```

##### 系统日志切割机制
```
在/etc/logrotate.d/下创建nginx文件，内容为：
/data/logs/*log {
    daily
    rotate 30
    missingok
    notifempty
    compress
    sharedscripts
    postrotate
        /bin/kill -USR1 $(cat /usr/local/nginx/logs/nginx.pid 2>/dev/null) 2>/dev/null || :
    endscript
}

说明：
1 nginx日志在/data/logs/目录下面，日志名字以log结尾
2 daily表示每天切割
3 rotate 30表示日志保留30天
4 missingok表示忽略错误
5 notifempty表示如果日志为空，不切割
6 compress表示压缩
7 sharedscripts和endscript中间可以引用系统的命令
8 postrotate表示当切割之后要执行的命令
```


### Nginx优化

#### Nginx配置参数优化

```
Nginx作为高性能web服务器，即使不特意调整配置参数也可以处理大量的并发请求。
以下的配置参数是借鉴网上的一些调优参数，仅作为参考，不见得适于你的线上业务。
```

##### worker进程

* worker_processes
```
该参数表示启动几个工作进程，建议和本机CPU核数保持一致，每一核CPU处理一个进程。
```

* worker_rlimit_nofile
```
它表示Nginx最大可用的文件描述符个数，需要配合系统的最大描述符，建议设置为102400。
还需要在系统里执行ulimit -n 102400才可以。
也可以直接修改配置文件/etc/security/limits.conf修改
增加：
#* soft nofile 655350 (去掉前面的#)
#* hard nofile 655350 (去掉前面的#)
```
* worker_connections 
```
该参数用来配置每个Nginx worker进程最大处理的连接数，这个参数也决定了该Nginx服务器最多能处理多少客户端请求
（worker_processes * worker_connections)，建议把该参数设置为10240，不建议太大。
```

##### http和tcp连接

* use epoll
```
使用epoll模式的事件驱动模型，该模型为Linux系统下最优方式。
```
* multi_accept on
```
使每个worker进程可以同时处理多个客户端请求。
```
* sendfile on
```
使用内核的FD文件传输功能，可以减少user mode和kernel mode的切换，从而提升服务器性能。
```
* tcp_nopush on
```
当tcp_nopush设置为on时，会调用tcp_cork方法进行数据传输。
使用该方法会产生这样的效果：当应用程序产生数据时，内核不会立马封装包，而是当数据量积累到一定量时才会封装，然后传输。
```
* tcp_nodelay on
```
不缓存data-sends（关闭 Nagle 算法），这个能够提高高频发送小数据报文的实时性。
(关于Nagle算法)
【假如需要频繁的发送一些小包数据，比如说1个字节，以IPv4为例的话，则每个包都要附带40字节的头，
也就是说，总计41个字节的数据里，其中只有1个字节是我们需要的数据。
为了解决这个问题，出现了Nagle算法。
它规定：如果包的大小满足MSS，那么可以立即发送，否则数据会被放到缓冲区，等到已经发送的包被确认了之后才能继续发送。
通过这样的规定，可以降低网络里小包的数量，从而提升网络性能。】
```

* keepalive_timeout
```
定义长连接的超时时间，建议30s，太短或者太长都不一定合适，当然，最好是根据业务自身的情况来动态地调整该参数。
```
* keepalive_requests
```
定义当客户端和服务端处于长连接的情况下，每个客户端最多可以请求多少次，可以设置很大，比如50000.
```
* reset_timeout_connection on
```
设置为on的话，当客户端不再向服务端发送请求时，允许服务端关闭该连接。
```
* client_body_timeout 
```
客户端如果在该指定时间内没有加载完body数据，则断开连接，单位是秒，默认60，可以设置为10。
```
* send_timeout
```
这个超时时间是发送响应的超时时间，即Nginx服务器向客户端发送了数据包，但客户端一直没有去接收这个数据包。
如果某个连接超过send_timeout定义的超时时间，那么Nginx将会关闭这个连接。单位是秒，可以设置为3。
```

##### buffer和cache(以下配置都是针对单个请求)

* client_body_buffer_size
```
当客户端以POST方法提交一些数据到服务端时，会先写入到client_body_buffer中，如果buffer写满会写到临时文件里，建议调整为128k。
```
* client_max_body_size
```
浏览器在发送含有较大HTTP body的请求时，其头部会有一个Content-Length字段，client_max_body_size是用来限制Content-Length所示值的大小的。
这个限制body的配置不用等Nginx接收完所有的HTTP包体，就可以告诉用户请求过大不被接受。会返回413状态码。
例如，用户试图上传一个1GB的文件，Nginx在收完包头后，发现Content-Length超过client_max_body_size定义的值，
就直接发送413(Request Entity Too Large)响应给客户端。
将该数值设置为0，则禁用限制，建议设置为10m。
```
* client_header_buffer_size
```
设置客户端header的buffer大小，建议4k。
```
* large_client_header_buffers
```
对于比较大的header（超过client_header_buffer_size）将会使用该部分buffer，两个数值，第一个是个数，第二个是每个buffer的大小。
建议设置为4 8k
```
* open_file_cache 
```
该参数会对以下信息进行缓存：
打开文件描述符的文件大小和修改时间信息;
存在的目录信息;
搜索文件的错误信息（文件不存在无权限读取等信息）。
格式：open_file_cache max=size inactive=time;
max设定缓存文件的数量，inactive设定经过多长时间文件没被请求后删除缓存。
建议设置 open_file_cache max=102400 inactive=20s;
```

* open_file_cache_valid
```
指多长时间检查一次缓存的有效信息。建议设置为30s。
```

* open_file_cache_min_uses
```
open_file_cache指令中的inactive参数时间内文件的最少使用次数，
如,将该参数设置为1，则表示，如果文件在inactive时间内一次都没被使用，它将被移除。
建议设置为2。
```

##### 压缩

```
对于纯文本的内容，Nginx是可以使用gzip压缩的。使用压缩技术可以减少对带宽的消耗。
由ngx_http_gzip_module模块支持

配置如下：
gzip on; //开启gzip功能
gzip_min_length 1024; //设置请求资源超过该数值才进行压缩，单位字节
gzip_buffers 16 8k; //设置压缩使用的buffer大小，第一个数字为数量，第二个为每个buffer的大小
gzip_comp_level 6; //设置压缩级别，范围1-9,9压缩级别最高，也最耗费CPU资源
gzip_types text/plain application/x-javascript text/css application/xml image/jpeg image/gif image/png; //指定哪些类型的文件需要压缩
gzip_disable "MSIE 6\."; //IE6浏览器不启用压缩

测试：
curl -I -H "Accept-Encoding: gzip, deflate" http://www.test.com/1.css

```


##### 日志
* 错误日志级别调高，比如crit级别，尽量少记录无关紧要的日志。
* 对于访问日志，如果不要求记录日志，可以关闭，
* 静态资源的访问日志关闭


##### 静态文件过期
```
对于静态文件，需要设置一个过期时间，这样可以让这些资源缓存到客户端浏览器，
在缓存未失效前，客户端不再向服务器请求相同的资源，从而节省带宽和资源消耗。

配置示例如下：
location ~* ^.+\.(gif|jpg|png|css|js)$                                      
{
    expires 1d; //1d表示1天，也可以用24h表示一天。
}
```

##### 作为代理服务器
```
Nginx绝大多数情况下都是作为代理或者负载均衡的角色。
因为前面章节已经介绍过以下参数的含义，在这里只提供对应的配置参数：
http
{
    proxy_cache_path /data/nginx_cache/ levels=1:2 keys_zone=my_zone:10m inactive=300s max_size=5g;
    ...;
    server
    {
	proxy_buffering on;
	proxy_buffer_size 4k;
	proxy_buffers 2 4k;
	proxy_busy_buffers_size 4k;
	proxy_temp_path /tmp/nginx_proxy_tmp 1 2;
	proxy_max_temp_file_size 20M;
	proxy_temp_file_write_size 8k;
	
	location /
	{
	    proxy_cache my_zone;
	    ...;
	}
    }
}

```

##### SSL优化
* 适当减少worker_processes数量，因为ssl功能需要使用CPU的计算。
* 使用长连接，因为每次建立ssl会话，都会耗费一定的资源（加密、解密）
* 开启ssl缓存，简化服务端和客户端的“握手”过程。
```
ssl_session_cache   shared:SSL:10m; //缓存为10M
ssl_session_timeout 10m; //会话超时时间为10分钟
```

#### 调整Linux内核参数
```
作为高性能WEB服务器，只调整Nginx本身的参数是不行的，因为Nginx服务依赖于高性能的操作系统。
以下为常见的几个Linux内核参数优化方法。
```
* 1 net.ipv4.tcp_max_tw_buckets 
```
对于tcp连接，服务端和客户端通信完后状态变为timewait，假如某台服务器非常忙，连接数特别多的话，那么这个timewait数量就会越来越大。
毕竟它也是会占用一定的资源，所以应该有一个最大值，当超过这个值，系统就会删除最早的连接，这样始终保持在一个数量级。
这个数值就是由net.ipv4.tcp_max_tw_buckets这个参数来决定的。
CentOS7系统，你可以使用sysctl -a |grep tw_buckets来查看它的值，默认为32768，
你可以适当把它调低，比如调整到8000，毕竟这个状态的连接太多也是会消耗资源的。
但你不要把它调到几十、几百这样，因为这种状态的tcp连接也是有用的，
如果同样的客户端再次和服务端通信，就不用再次建立新的连接了，用这个旧的通道，省时省力。
```
* 2 net.ipv4.tcp_tw_recycle = 1
```
该参数的作用是快速回收timewait状态的连接。上面虽然提到系统会自动删除掉timewait状态的连接，但如果把这样的连接重新利用起来岂不是更好。
所以该参数设置为1就可以让timewait状态的连接快速回收，它需要和下面的参数配合一起使用。
```

* 3 net.ipv4.tcp_tw_reuse = 1
```
该参数设置为1，将timewait状态的连接重新用于新的TCP连接，要结合上面的参数一起使用。
```
* 4 net.ipv4.tcp_syncookies = 1
```
tcp三次握手中，客户端向服务端发起syn请求，服务端收到后，也会向客户端发起syn请求同时连带ack确认，
假如客户端发送请求后直接断开和服务端的连接，不接收服务端发起的这个请求，服务端会重试多次，
这个重试的过程会持续一段时间（通常高于30s），当这种状态的连接数量非常大时，服务器会消耗很大的资源，从而造成瘫痪，
正常的连接进不来，这种恶意的半连接行为其实叫做syn flood攻击。
设置为1，是开启SYN Cookies，开启后可以避免发生上述的syn flood攻击。
开启该参数后，服务端接收客户端的ack后，再向客户端发送ack+syn之前会要求client在短时间内回应一个序号，
如果客户端不能提供序号或者提供的序号不对则认为该客户端不合法，于是不会发ack+syn给客户端，更涉及不到重试。
```
* 5 net.ipv4.tcp_max_syn_backlog
```
该参数定义系统能接受的最大半连接状态的tcp连接数。客户端向服务端发送了syn包，服务端收到后，会记录一下，
该参数决定最多能记录几个这样的连接。在CentOS7，默认是256，当有syn flood攻击时，这个数值太小则很容易导致服务器瘫痪，
实际上此时服务器并没有消耗太多资源（cpu、内存等），所以可以适当调大它，比如调整到30000。
```
* 6 net.ipv4.tcp_syn_retries
```
该参数适用于客户端，它定义发起syn的最大重试次数，默认为6，建议改为2。
```
* 7 net.ipv4.tcp_synack_retries
```
该参数适用于服务端，它定义发起syn+ack的最大重试次数，默认为5，建议改为2，可以适当预防syn flood攻击。
```
* 8 net.ipv4.ip_local_port_range
```
该参数定义端口范围，系统默认保留端口为1024及以下，以上部分为自定义端口。这个参数适用于客户端，
当客户端和服务端建立连接时，比如说访问服务端的80端口，客户端随机开启了一个端口和服务端发起连接，
这个参数定义随机端口的范围。默认为32768    61000，建议调整为1025 61000。
```
* 9 net.ipv4.tcp_fin_timeout
```
tcp连接的状态中，客户端上有一个是FIN-WAIT-2状态，它是状态变迁为timewait前一个状态。
该参数定义不属于任何进程的该连接状态的超时时间，默认值为60，建议调整为6。
```
* 10 net.ipv4.tcp_keepalive_time
```
tcp连接状态里，有一个是established状态，只有在这个状态下，客户端和服务端才能通信。正常情况下，当通信完毕，
客户端或服务端会告诉对方要关闭连接，此时状态就会变为timewait，如果客户端没有告诉服务端，
并且服务端也没有告诉客户端关闭的话（例如，客户端那边断网了），此时需要该参数来判定。
比如客户端已经断网了，但服务端上本次连接的状态依然是established，服务端为了确认客户端是否断网，
就需要每隔一段时间去发一个探测包去确认一下看看对方是否在线。这个时间就由该参数决定。它的默认值为7200秒，建议设置为30秒。
```
* 11 net.ipv4.tcp_keepalive_intvl
```
该参数和上面的参数是一起的，服务端在规定时间内发起了探测，查看客户端是否在线，如果客户端并没有确认，
此时服务端还不能认定为对方不在线，而是要尝试多次。该参数定义重新发送探测的时间，即第一次发现对方有问题后，过多久再次发起探测。
默认值为75秒，可以改为3秒。
```
* 12 net.ipv4.tcp_keepalive_probes
```
第10和第11个参数规定了何时发起探测和探测失败后再过多久再发起探测，但并没有定义一共探测几次才算结束。
该参数定义发起探测的包的数量。默认为9，建议设置2。
```

##### 设置和范例
```
在Linux下调整内核参数，可以直接编辑配置文件/etc/sysctl.conf，然后执行sysctl -p命令生效

结合以上分析的各内核参数，范例如下
net.ipv4.tcp_fin_timeout = 6
net.ipv4.tcp_keepalive_time = 30
net.ipv4.tcp_max_tw_buckets = 8000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 30000
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_synack_retries = 2
net.ipv4.ip_local_port_range = 1025 61000
net.ipv4.tcp_keepalive_intvl = 3
net.ipv4.tcp_keepalive_probes = 2


```

### Nginx服务监控

#### 系统级别监控
```
TOP
ps aux | grep nginx
netstat -lnp | grep nginx
ss -an 
lsof -c nginx
日志
```

#### 配置Nginx状态
```
Nginx有内置一个状态页，需要在编译的时候指定参数--with-http_stub_status_module参数方可打开。
也就是说，该功能是由http_stub_status_module模块提供，默认没有加载。
```

##### Nginx配置文件示例
```
server{
	listen 80;
	server_name www.test.com;
	
	location /status/ {
	    stub_status on;
	    access_log off;
	    allow 127.0.0.1;
	    allow 192.168.10.0/24;
	    deny all;
	}
}

```

##### 配置说明
* location /status/这样当访问/status/时即可访问到状态页内容。
* stub_status on即打开了状态页。
* access_log off不记录日志
* allow和deny只允许指定IP和IP段访问，因为这个页面需要保护起来，并不公开，当然也可以做用户认证。

##### 测试和结果说明
```
测试命令：curl -x127.0.0.1:80 www.test.com/status/

结果如下：
Active connections: 1 
server accepts handled requests
 11 11 11 
Reading: 0 Writing: 1 Waiting: 0 

说明：
active connections – 活跃的连接数量
server accepts handled requests — 总共处理的连接数、成功创建的握手次数、总共处理的请求次数
需要注意，一个连接可以有多次请求。
reading — 读取客户端的连接数.
writing — 响应数据到客户端的数量
waiting — 开启 keep-alive 的情况下,这个值等于 active – (reading+writing), 意思就是 Nginx 已经处理完正在等候下一次请求指令的驻留连接.
```


