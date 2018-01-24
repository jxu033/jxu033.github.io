---
title: "scrapy learning note 1"
layout: post
date: 2018-01-01 22:45
tag:
- scrapy
- basic
- spider
star: true
category: blog
author: jiaqixu
---

## Spider 实战学习笔记 - 1

#### summary
- [Spider](#spider)
- [Selector](#selector)
- [Item](#item)
- [Item Pipeline](#item-pipeline)
- [Feed Exports](#feed-exports)
### Spider

#### 基本介绍
概念： Spider是一个类，它定义了怎样爬取一个网站，包括怎样跟踪连接、怎样提取数据。

循环执行流程:
<ol>
<li>generating the initial Requests</li>
<li>parse the response</li>
<li>using selector</li>
<li>store item</li>
</ol>

示例：
{% highlight python %}
import scrapy

class StackOverflowSpider(scrapy.Spider):
    name = 'stackoverflow'
    start_urls = ['https://stackoverflow.com/questions?sort=votes']

    def parse(self, response):
        for href in response.css('.question-summary h3 a::attr(href)'):
            import ipdb; ipdb.set_trace()
            full_url = response.urljoin(href.extract())
            yield scrapy.Request(full_url, callback=self.parse_question)

    def parse_question(self, response):
        yield {
            'title': response.css('h1 a::text').extract()[0],
            'votes': response.css('.question .vote-count-post::text').extract()[0],
            'body': response.css('.question .post-text').extract()[0],
            #'tags': response.css('.qestion .post-tag::text').extract(),
            'tags': response.xpath("//div[class='post-taglist']//a/text()").extract(),
            'link': response.url
        }
{% endhighlight %}

#### 基类（scrapy.Spider）介绍
属性：

name：spider的名称，要求唯一

allowed_domains: 允许的域名

start_urls: 初始urls

custom_settings: 个性化设置，会覆盖全局的设置

crawler: 抓取器，spider将绑定到它上面

settings: 配置实例，包含工程中所有的配置变量

logger: 日志实例

方法：

from_crawler(crawler, * args, ** kwargs): 类方法用于创建spiders

start_requests(): 生成初始的requests

make_requests_from_url(url): 根据url生成一个request

parse()：用于解析网页内容

log(message[, level, component]): 用来记录日志，这里请使用logger属性记录日志，
                                  self.logger.info('visit success')

close(reason): 当spider关闭的时候调用的方法

#### 子类介绍

CrawlSpider:

1. 最常用的spider，用于抓去普通网页
2. 增加了两个成员：1）rules:定义了一些抓取规则--连接怎么跟踪、使用哪一个parse函数解析此连接
                2）parse_start_url(response): 解析初始url的响应
3. 实例：
{% highlight python %}
import scrapy
from scrapy.spiders import CrawlSpider, Rule
from scrapy.linkextractors import LinkExtractor

class MySpider(CrawlSpider):
    name = 'example.com'
    allowed_domains = ['example.com']
    start_urls = ['http://www.example.com']

    rules = (
        # Extract links matching 'category.php' (but not matching 'subsection.php')
        # and follow links from them (since no callback means follow=True by default).
        Rule(LinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),

        # Extract links matching 'item.php' and parse them with the spider's method parse_item
        Rule(LinkExtractor(allow=('item\.php', )), callback='parse_item'),
    )

    def parse_item(self, response):
        self.logger.info('Hi, this is an item page! %s', response.url)
        item = scrapy.Item()
        item['id'] = response.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
        item['name'] = response.xpath('//td[@id="item_name"]/text()').extract()
        item['description'] = response.xpath('//td[@id="item_description"]/text()').extract()
        return item
{% endhighlight %}

其他的spider：<br>
XMLFeedSpider<br>
CSVFeedSpider<br>
SitemapSPider<br>
见 https://doc.scrapy.org/en/latest/topics/spiders.html


### Selector
用来解析网页的库有很多，比如beautifulsoup、 lxml，但在scrapy里面默认的使用的是selector，相对来说也是最好用的。

#### 基础介绍
实例化：<br>
{% highlight python %}
from scrapy.selector import Selector
from scrapy.http import HtmlResponse
{% endhighlight %}

1. 用text初始化 （Constructing from text):
{% highlight python %}
body = '<html><body><span>good</span></body></html>'
Selector(text=body).xpath('//span/text()').extract()
[u'good']
{% endhighlight %}

2. TextResponse对象初始化（Constructing from response):
{% highlight python %}
response = HtmlResponse(url='http://example.com', body=body)
Selector(response=response).xpath('//span/text()').extract()
[u'good']
{% endhighlight %}

常用方法:
<ol>
<li>xpath</li>
<li>css</li>
<li>re</li>
<li>extract</li>
</ol>

#### 应用实例
<a href='https://doc.scrapy.org/en/latest/topics/selectors.html'>见文档</a>


### Item

Items are declared using a simple class definition syntax and Field objects. Here is an example:
{% highlight python %}
import scrapy

class Product(scrapy.Item):
    name = scrapy.Field()
    price = scrapy.Field()
    stock = scrapy.Field()
    last_updated = scrapy.Field(serializer=str)
{% endhighlight %}

### Item Pipeline

#### 作用
Typical uses of item pipelines are:
<ul>
<li>cleansing HTML data</li>
<li>validating scraped data (checking that the items contain certain fields)</li>
<li>checking for duplicates (and dropping them)</li>
<li>storing the scraped item in a database</li>
</ul>

#### 编写方法

<a href='https://doc.scrapy.org/en/latest/topics/item-pipeline.html'>Item pipeline example</a>

#### 配置启用pipeline

To activate an Item Pipeline component you must add its class to the ITEM_PIPELINES setting, like in the following example:
{% highlight python %}
ITEM_PIPELINES = {
    'myproject.pipelines.PricePipeline': 300,
    'myproject.pipelines.JsonWriterPipeline': 800,
}
{% endhighlight %}
The integer values you assign to classes in this setting determine the order in which they run: items go through from lower valued to higher valued classes. It’s customary to define these numbers in the 0-1000 range.

### Feed Exports

<a href='https://doc.scrapy.org/en/latest/topics/feed-exports.html'>存储数据抓取结果</a>
