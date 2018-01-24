---
title: "scrapy learning note 4"
layout: post
date: 2018-01-03 22:45
tag:
- scrapy
- scrapy engine
- spider
- scrapy framework principle
star: true
category: blog
author: jiaqixu
---

## Scrapy Framework Principle

Scrapy使用了Twisted异步网络库来处理网络通讯

他的整体架构大致如下：
<img src="/assets/images/blog/scrapy_framework.jpg">

### 基本介绍
<ol>
<li>Scrapy Engine:<br>
scrapy引擎是用来控制整个系统的数据处理流程，负责组件之间的流转，并进行事务处理的触发。
</li>
<br>
<li>Scheduler:<br>
调度程序从scrapy引擎接受请求并排序列入队列，以便后续的调度。
</li>
<br>
<li>Downloader:<br>
负责抓取网页，并传送给引擎，之后抓取结果将传给spider
</li>
<br>
<li>Spider:<br>
Spider是由Scrapy用户自己定义用来解析网页并抓取制定URL返回的内容的类，每个Spider都能处理一个域名或一组域名。换句话说就是用来定义特定网站的抓取和解析规则。
<br>
Spider的整个抓取流程（周期）是这样的:<br>
    <ol type='i'>
      <li>首先获取第一个URL的初始请求，当请求返回后调取一个回调函数。第一个请求是通过调用start_requests()方法。该方法默认从start_urls中的Url中生成请求，并执行解析来调用回调函数。</li>
      <li>在回调函数中，你可以解析网页响应并返回项目对象和请求对象或两者的迭代。这些请求也将包含一个回调，然后被Scrapy下载，然后有指定的回调处理。</li>
      <li>在回调函数中，你解析网站的内容，同程使用的是Xpath选择器（但是你也可以使用BeautifuSoup, lxml或其他任何你喜欢的程序），并生成解析的数据项。</li>
      <li>最后，从Spider返回的items通常会进驻到item pipeline。</li>
    </ol>
</li>
<br>
<li>Item Pipeline:<br>
Item Pipeline的主要责任是负责处理由Spider从网页中抽取的items，他的主要任务是清晰、验证和存储数据。当页面被Spider解析后，将被发送到item pipeline，并经过几个特定的次序处理数据。每个item Pipeline的组件都是由一个简单的方法组成的Python类。他们获取了item并执行他们的方法，同时他们还需要确定的是是否需要 在item pipeline中继续执行下一步或是直接丢弃掉不处理。
<br>
Item Pipeline通常执行的过程是:
    <ol type='i'>
      <li>清洗HTML数据</li>
      <li>验证解析到的数据(检查item是否包含必要的字段)</li>
      <li>检查是否是重复数据（如果重复就删除）</li>
      <li>将解析到的数据存储到数据库中</li>
    </ol>
</li>
<br>
<li>Downloader middlewares<br>
下载中间件是位于Scrapy引擎和下载器之间的钩子框架，主要是处理Scrapy引擎与下载器之间的请求及响应。它提供了一个自定义的代码的方式 来拓展Scrapy的功能。
</li>
<br>
<li>Spider middlewares<br>
Spider middlewares是介于Scrapy引擎和Spider之间的钩子框架，主要工作是处理蜘蛛的响应输入和请求输出。它提供一个自定义代码的方式来拓展Scrapy的功能,你可以插入自定义的代码来处理发送给蜘蛛的请求和返回蜘蛛获取的响应内容和项目。
</li>
</ol>

### 数据处理流程

Scrapy的整个数据处理流程由Scrapy引擎进行控制，其过程如下:
<ol>
<li>引擎打开一个网站(open a domain),找到处理该网站的Spider并向该spider请求第一个要爬取的URL(s) </li>
<li>引擎从Spider中获取到第一个要爬取的URL并在调度器（Scheduler）以Request调度</li>
<li>引擎向调度器请求下一个要爬取的URL</li>
<li>调度器返回下一个要爬取的URL给引擎，引擎将URL通过Downloader middlewares(request的方向)转发给Downloader</li>
<li>一旦页面下载完毕，Downloader生成一个该页面的Response，并将其通过downloader middleware（response的方向）发送给引擎 </li>
<li>引擎从downloader中接收到Response并通过Spider middlewares发送给Spider处理</li>
<li>Spider处理response并返回爬取的item及（跟进的）新的Request给引擎</li>
<li>引擎将（spider返回的)item给item pipeline,将（spider返回的）Request给调度器</li>
<li>（从第二步）重复直到调度器中没有更多地request,引擎将关闭该网站。</li>
</ol>
