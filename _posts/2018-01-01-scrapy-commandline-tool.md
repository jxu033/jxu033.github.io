---
title: "scrapy basic commandline tool"
layout: post
date: 2018-01-01 22:44
tag:
- scrapy
- commandline
- spider
star: true
category: blog
author: jiaqixu
---

## Summary:

{% highlight raw %}
1. scrapy --help
{% endhighlight %}
是scrapy的基本命令， 用于查看帮助信息

{% highlight raw %}
2. scrapy version
{% endhighlight %}
输出scrapy的版本信息

{% highlight raw %}
3. scrapy version -v
{% endhighlight %}
除了scrapy的版本，还列出了scpray中各个组件的版本还有所使用的操作系统平台的信息等

{% highlight raw %}
4. scrapy startproject
{% endhighlight %}
用于新建一个工程

{% highlight raw %}
5. scrapy genspider
{% endhighlight %}
在工程中产生一个spider, 可以产生多个spiders, 不同spider要求名字不同

{% highlight raw %}
6. scrapy list
{% endhighlight %}
列出工程下所有spiders

{% highlight raw %}
7. scrapy view <webpage_url>
{% endhighlight %}
在浏览器中打开spider看到页面究竟是怎么样的

{% highlight raw %}
8. scrapy parse <webpage_url>
{% endhighlight %}
在工程中使用固定的parse函数解析某个页面,这个命令可以用来检查所写的Parse函数是否正确

{% highlight raw %}
9. scrapy shell <webpage_url>
{% endhighlight %}
一个非常有用的命令，可以用于调试数据、检测xpath，查看页面源代码，等等

{% highlight raw %}
10. scrapy runspider <spider_file.py>
{% endhighlight %}
运行整个spider文件,依旧能输出爬到的数据

{% highlight raw %}
11. scrapy bench
{% endhighlight %}
可以用来检测scrapy是否安装成功
