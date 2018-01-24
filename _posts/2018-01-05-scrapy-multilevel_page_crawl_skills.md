---
title: "scrapy learning note 5"
layout: post
date: 2018-01-04 22:45
tag:
- scrapy
- crawl
- mutiple-level page crawl
star: true
category: blog
author: jiaqixu
---


## 多级页面抓取技巧

掌握在spider的不同parse函数中通过meta传递参数

### requirement analysis

1. target website -- 天猫（Tmall)
2. content to crawl -- 天猫商城销量前60的商品的情况（商品价格，商品名称，商品url）、店铺的情况（店铺名称，店铺url,公司名称，公司地址）
3. saving format -- excel表格（.csv）


### item 编写
{% highlight python %}
import scrapy


class TmallGoodsItem(scrapy.Item):
    # define the fields for your item here like:
    GOODS_PRICE = scrapy.Field()
    GOODS_NAME = scrapy.Field()
    GOODS_URL = scrapy.Field()
    SHOP_NAME = scrapy.Field()
    SHOP_URL = scrapy.Field()
    COMPANY_NAME = scrapy.Field()
    COMPANY_ADDRESS =scrapy.Field()
{% endhighlight %}

### spider 编写
{% highlight python %}
# -*- coding: utf-8 -*-
import scrapy
from tmallspider.items import TmallGoodsItem


class TmallSpider(scrapy.Spider):
    name = 'tmall_spider'
    allowed_domains = ['www.tmall.com/']
    start_urls = ['https://list.tmall.com/search_product.htm?spm=a220m.1000858.1000724.4.4aa3d905RKuMMl&cat=50025135&q=%C5%AE%D7%B0&sort=d&style=g&from=mallfp..pc_1_searchbutton']

    #记录处理的页数
    count = 0

    def parse(self, response):

        TmallSpider.count += 1
        divs = response.xpath('//div[@id="J_ItemList"]//div[@class="product-iWrap"]')

        for div in divs:
            item = TmallGoodsItem()
            item['GOODS_PRICE'] = div.xpath('p[@class="productPrice"]/em/@title')[0].extract()
            item['GOODS_NAME'] = div.xpath('p[@class="productTitle"]/a/@title')[0].extract()
            pre_goods_url = div.xpath('p[@class="productTitle"]/a/@href')[0].extract()
            item['GOODS_URL'] = pre_goods_url if "http:" in pre_goods_url else ("http:" + pre_goods_url)

            break # avoid getting block, just crawl one product

            yield scrapy.Request(
                url=item['GOODS_URL'], meta={'item':item}, callback=self.parse_detail,
                dont_filter=True
            )

    def parse_detail(self, response):
        div = response.xpath('//div[@class="shop-intro"]//div[@class="name"]')
        if not div:
            self.log('List page error %s'%response.url)

        item = response.meta['item']
        item['SHOP_NAME'] = div.xpath('a/text()')
        item['SHOP_URL'] = div.xpath('a/@href')

        yield item

{% endhighlight %}

### 输出到csv文件
{% highlight python %}
scrapy crawl  -o tmall_spider results.csv
{% endhighlight %}
