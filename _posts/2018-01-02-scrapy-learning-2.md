---
title: "scrapy learning note 2"
layout: post
date: 2018-01-02 22:45
tag:
- scrapy
- basic
- spider
star: true
category: blog
author: jiaqixu
---

## Scrapy 内置服务介绍

### logging 模块

1. 简单使用方法
{% highlight python %}
import logging
logging.warning('This is a warning')
{% endhighlight %}

2. 通用的记录日志的方法，可加入日志的级别
{% highlight python %}
import logging
logging.log(logging.WARNING, 'This is a warning')
{% endhighlight %}

3. 通过logger记录日志
{% highlight python %}
import logging
logger = logging.getLogger(__name__)
logger.warning('This is a warning')
{% endhighlight %}

#### logging module in scrapy

1.Logging from Spiders：
{% highlight python %}
import scrapy

class MySpider(scrapy.Spider):

    name = 'myspider'
    start_urls = ['https://scrapinghub.com']

    def parse(self, response):
        self.logger.info('Parse function called on %s', response.url)
{% endhighlight %}
因为scrapy.Spider类中包含了logger，再者因为MySpider继承了Spider，所以可以直接调用内置logger

2. logger 创建默认是使用spider的名字， 但是我们可以使用任何自定义的python logger：
{% highlight python %}
import logging
import scrapy

logger = logging.getLogger('mycustomlogger')

class MySpider(scrapy.Spider):

    name = 'myspider'
    start_urls = ['https://scrapinghub.com']

    def parse(self, response):
        logger.info('Parse function called on %s', response.url)
{% endhighlight %}

3. 为了更方便的使用日志，我们可以在setttings.py中配置相关的参数，scrapy会自动帮我们去记录日志信息，这样对于我们程序的调试和信息的记录都是非常有帮助的

以下这些settings可以用来设置logging:
<ul>
<li>LOG_FILE：日志的名称</li>
<li>LOG_ENABLED:是否启用日志</li>
<li>LOG_ENCODING:设置日志所使用的编码，默认为‘utf-8’</li>
<li>LOG_LEVEL:日志最小记录级别（程序运行过程中出现小于这个级别的信息将不会被记录),默认为‘DEBUG’, Available levels are: CRITICAL, ERROR, WARNING, INFO, DEBUG</li>
<li>LOG_FORMAT: 日志的格式（String for formatting log messsages）， Default: '%(asctime)s [%(name)s] %(levelname)s: %(message)s'</li>
<li>OG_DATEFORMAT: 日志的格式， Default: '%Y-%m-%d %H:%M:%S'</li>
<li>LOG_STDOUT: 如何设置为True，则会将所有标准输出（如print 'hello',则会出现在日志中）和ERROR将会记录在日志中， Default: False</li>
</ul>

### Stats Collection

该模块用来收集在爬取数据的过程中相关的信息，比如说爬了多少条信息，爬了多少网页，产生了多少数据

#### 基本操作

```python
import logging
import scrapy

logger = logging.getLogger('mycustomlogger')

class MySpider(scrapy.Spider):

    name = 'myspider'
    start_urls = ['https://scrapinghub.com']

    def parse(self, response):
        logger.info('Parse function called on %s', response.url)   logger.info('Parse function called on %s', response.url) logger.info('Parse function called on %s', response.url)
```
