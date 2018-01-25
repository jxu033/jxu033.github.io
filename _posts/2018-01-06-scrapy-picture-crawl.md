---
title: "scrapy learning note 6"
layout: post
date: 2018-01-06 22:45
tag:
- scrapy
- crawl
- picture
- download
star: true
category: blog
author: jiaqixu
---

## 图片抓取 picture url crawl and download

Based on leaning note 5, we add a new feature of crawling goods picture and save it locally.

### requirement analysis

1. target website -- 天猫（Tmall)
2. content to crawl -- 天猫商城销量前60的商品的情况（商品价格，商品名称，商品url）、店铺的情况（店铺名称，店铺url,公司名称，公司地址）以及商品的图片
3. saving format -- 图片存储到本地文件夹，数据存储到csv文件中

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
from scrapy.pipelines import images

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

            # Picture Url
            try:
                import ipdb; ipdb.set_trace()
                picture_url = div.xpath('//div[@class="productImg-wrap"]/a/img/@src')[0].extract()
                # have to be a list of pircture urls
                item['PICTURE_URL'] = ['http:' + picture_url]
            except Exception, e:
                print 'ERROR:', e

            yield scrapy.Request(
                url=item['GOODS_URL'], meta={'item':item}, callback=self.parse_detail,
                dont_filter=True
            )

            break # avoid getting block, just crawl one product

    def parse_detail(self, response):
        div = response.xpath('//div[@class="slogo"]/a[@class="slogo-shopname"]')
        if not div:
            self.log('List page error %s'%response.url)

        item = response.meta['item']
        item['SHOP_NAME'] = div.xpath('.//text()')[0].extract()
        pre_shop_url = div.xpath('@href')[0].extract()
        item['SHOP_URL'] = pre_shop_url if "http:" in pre_shop_url else ("http:" + pre_shop_url)
        yield item

{% endhighlight %}


### settings for image pipelines
{% highlight python %}
ITEM_PIPELINES = {
    'scrapy.pipelines.images.ImagesPipeline' : 1
}
IMAGES_URLS_FIELD = 'PICTURE_URL' # IMAGES_URL_FIELD mistake
IMAGES_STORE = 'images'
{% endhighlight %}

### 运行所需要的包
想要使用ImagesPipelines，需要安装PIL (Python Imaging Library)
The Images Pipeline uses Pillow for thumbnailing and normalizing images to JPEG/RGB format, so you need to install this library in order to use it.
安装完之后，运行下列命令即可：
{% highlight python %}
scrapy crawl tmall_spider -o results.csv
{% endhighlight %}
