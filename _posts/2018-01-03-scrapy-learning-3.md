---
title: "scrapy learning note 3"
layout: post
date: 2018-01-03 22:45
tag:
- scrapy
- basic
- spider
- pymongo
star: true
category: blog
author: jiaqixu
---

## Scrapy 实战学习笔记 - 3

### 将scrapy爬取的数据插入mongdb数据库中

以爬取西刺网站的代理（xici PROXY）为例,进行一个简单的演示：

#### 分析网站结构，根据要爬取的内容编写Item
{% highlight python %}
import scrapy

class CollectipsItem(scrapy.Item):
    # define the fields for your item here like:
    IP = scrapy.Field()
    PORT = scrapy.Field()
    POSITION = scrapy.Field()
    TYPE = scrapy.Field()
    SPEED = scrapy.Field()
    LAST_CHECK_TIME = scrapy.Field()
{% endhighlight %}

#### 建立spider爬取数据
{% highlight python %}
# -*- coding: utf-8 -*-
import scrapy
from collectips.items import CollectipsItem


class XicispiderSpider(scrapy.Spider):
    name = 'XiciSpider'
    allowed_domains = ['xicidaili.com']

    start_urls = ['http://xicidaili.com/']

    def start_requests(self):
        reqs = []
        headers = {
            'User-Agent': 'Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Mobile Safari/537.36'
        }
        for i in range(1,3): #to be improved
            yield scrapy.Request('http://www.xicidaili.com/nn/%s'%i, headers=headers)


    def parse(self, response):
        ip_list = response.xpath('//table[@id="ip_list"]')
        trs = ip_list[0].xpath('tr')
        items = []
        for ip in trs[1:]:
            pre_item = CollectipsItem()
            pre_item['IP'] = ip.xpath('td[2]/text()')[0].extract()
            pre_item['PORT'] = ip.xpath('td[3]/text()')[0].extract()
            pre_item['POSITION'] = ip.xpath('string(td[4])')[0].extract().strip()
            pre_item['TYPE'] = ip.xpath('td[6]/text()')[0].extract()
            pre_item['SPEED'] = ip.xpath('td[7]/div[@class="bar"]/@title') \
                                .re('\d{0,2}\.\d*')[0]
            pre_item['LAST_CHECK_TIME'] = ip.xpath('td[10]/text()')[0].extract()
            yield pre_item
{% endhighlight %}

#### 编写pipelines处理item, 包括连接数据库和插入items
{% highlight python %}
import pymongo
from scrapy.conf import settings
from scrapy.exceptions import DropItem
from scrapy import log

class MongoDBPipeline(object):

    def __init__(self):
        connection = pymongo.MongoClient(
            settings['MONGODB_SERVER'],
            settings['MONGODB_PORT']
        )
        db = connection[settings['MONGODB_DB']]
        self.collection = db[settings['MONGODB_COLLECTION']]

    def process_item(self, item, spider):
        valid = True
        for data in item:
            if not data:
                valid = False
                raise DropItem("Missing {0}!".format(data))
        if valid:
            self.collection.insert(dict(item))
            log.msg("Question added to MongoDB database!",
                    level=log.DEBUG, spider=spider)
        return item
{% endhighlight %}

#### 在settings.py中配置数据库（为上一步提供参数）和pipeline
{% highlight python %}
# mongodb database settings
ITEM_PIPELINES = ['collectips.pipelines.MongoDBPipeline', ]

MONGODB_SERVER = "localhost"
MONGODB_PORT = 27017
MONGODB_DB = "xici_proxy"
MONGODB_COLLECTION = "ips_collection"

# Configure item pipelines
ITEM_PIPELINES = {
    'collectips.pipelines.MongoDBPipeline': 300,
}
{% endhighlight %}

#### 运行 spider, 爬取的数据将会被出入数据库中.
<img src="/assets/images/blog/scraped_mongo_result.jpg">
