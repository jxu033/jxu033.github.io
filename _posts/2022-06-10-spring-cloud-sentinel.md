---
title: "SpringCloud Sentinel"
layout: post
date: 2022-06-01 22:44
tag:
- spring cloud
- sentinel
- 限流
- 熔断
star: true
category: blog
author: jiaqixu
description: note for SpringCloud Alibaba Sentinel
---


### 目录
* TOC
{:toc}


## 服务雪崩效应
![image](/assets/images/blog/springcloud/服务雪崩效应.png) 

解决办法：

1. 限流: 基于qps

2. 仓壁模式(想象一下船的结构)：基于线程数/池

3. 断路器：统计是否有大量耗时的请求，如果有则熔断掉，可以参考家用电器的熔断器


## Sentinel服务哨兵

### 1.1 sentinel是什么
> 随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
> 

#### 1.1.1 sentinel具有以下特征
* **丰富的应用场景**：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀(即突发流量控制在系统容量可以承受的范围)、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
* 
![image](/assets/images/blog/springcloud/sentinel-应用场景.png)
* **完备的实时监控**: Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的 单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
* **广泛的开源生态**: Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC的整合。您只需要引入相应的依赖并进行简单的配置即可 快速地接入 Sentinel。
* **完善的SPI扩展点**: Sentinel 提供简单易用、完善的SPI扩展接口。您可以通过实现扩展 接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

#### 1.1.2 sentinel分为两个部分
* **核心库(Java客户端)**不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
* **控制台(Dashboard)**基于Spring Boot开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

### 1.2 sentinel和hystrix对比
![image](/assets/images/blog/springcloud/sentinel和hystrix对比.png)

### 1.3 sentinel架构
![image](/assets/images/blog/springcloud/sentinel架构.png)

```
#接口概览
http://127.0.0.1:8720/api
#获取资源的metrics信息 
http://127.0.0.1:8720/cnode?id=/jifen/{jifenId} 
#获取流控规则接口 http://127.0.0.1:8720/getRules?type=flow 
#设置规则接口
http://127.0.0.1:8720/setRules
```

### 1.4 sentinel控制台安装
下载地址：https://github.com/alibaba/Sentinel/releases/tag/v1.8.0
> 注意:启动 Sentinel 控制台需要 JDK 版本为 1.8 及以上版本。
> 
> 功能
> 
> 1:实时观看每个资源的访问情况(qps)
> 
> 2:通过控制台，设置流控规则(qps、线程数)、还可以设置熔断降级的规则(慢调用比例、异常比例、异常数)
> 

运行sentinel控制台
`java -jar sentinel-dashboard-1.8.0.jar --server.port=8888`

### 1.5 微服务整合sentinel
1. 添加依赖

	```
	<dependency>
	    <groupId>com.alibaba.cloud</groupId>
	    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
	</dependency>
	```
	
	![image](/assets/images/blog/springcloud/sentinel-dependency.png)
	
	可以看到依赖了aspectJ，即表明sentinel也利用了aop的思想去进行相关统计。
	
2. 配置
	
	```
	#默认是8719，如果8719被占用 +1，一直到没有被占用 
	spring.cloud.sentinel.transport.port=8719
	spring.cloud.sentinel.transport.dashboard=127.0.0.1:8888
	```
	```
	#开启饥额加载，当项目启动立即完成初始化，默认是懒加载 
	spring.cloud.sentinel.eager=true
	```
	
	![image](/assets/images/blog/springcloud/sentinel-config.png)



## sentinel服务流控
> Sentinel通过流量控制(flow control)以及熔断降级来保护系统资源
> 

### 2.1 QPS超过阈值直接失败
> 流量控制(flow control)，其原理是监控应用流量的 QPS 或并发线程数等
> 指标，当达到指定的阈值时对流量进行控制，以避免被瞬时的流量高峰冲
> 垮，从而保障应用的高可用性。

```
QPS:Queries-per-second,每秒资源被访问的次数
被访问资源的QPS超过阈值，直接进行限流(限制访问)，从而保护资源，
```

> 触发条件: /order/test5 当qps>1时，会触发流控

![image](/assets/images/blog/springcloud/qps超过阈值直接失败.png)


### 2.2 线程数超过阈值直接失败
> 触发条件:当/order/test5 正在工作的线程数>1

![image](/assets/images/blog/springcloud/线程数超过阈值直接失败.png) 

#### 2.2.1 JMeter压测工具
1. 新建测试计划
2. 添加线程组
3. 添加http请求
4. 添加结果观察
5. 发送请求

### 2.3 QPS超过阈值关联失败
> 触发条件: 当关联资源(/order/test6)访问qps>1 ,那么/order/test5就会被流控。

![image](/assets/images/blog/springcloud/qps超过阈值关联失败.png) 

### 2.4 线程数超过阈值关联失败
> 触发条件: 当关联资源(/order/test6)正在工作的线程数(并发数)>1 ,那么/order/test5 就会被流控。

![image](/assets/images/blog/springcloud/线程超过阈值关联失败.png) 

### 2.5 QPS超过阈值链路失败

![image](/assets/images/blog/springcloud/链路限流.png) 

```
feign.sentinel.enabled=true # 显示完整链路
spring.cloud.sentinel.web-context-unify=false 
# 官方在CommonFilter 引入了WEB_CONTEXT_UNIFY 参数，用于控制是否收敛context。
# 将其配置为 false 即可根据不同的URL进行链路限流。
```

**OrderController**

```
    @RequestMapping("test5")
    public String test5() {
        // jifen save
        orderService.saveOrder();

        return "test5";
    }

    @RequestMapping("test6")
    public String test6() {
        // jifen save
        orderService.saveOrder();
        return "test6";
    }
```

**OrderService**

```
@Service
public class OrderService {
    @Autowired
    private JifenApi jifenApi;

    @SentinelResource("saveOrder[jifen save]") # sentinel默认只扫控制器层，所以这里要加这个注解让其去扫描
    public void saveOrder() {
        //发送远程调用 jifen save
        Map save = jifenApi.save(new Jifen(1, 1 , "2"));

        System.out.println(save);
        System.out.println("order save success");
    }
}
```

> 触发条件: saveOrder[jifen save]有两个入口，分别为/order/test5和/order/test6 
> 
> 当/order/test5访问qps>1 那么/order/test5就不能访问受保护资源,其他入口照常访问

![image](/assets/images/blog/springcloud/qps超过阈值链路失败.png) 


### 2.6 流控效果-Warm-up
> Warm Up(`RuleConstant.CONTROL_BEHAVIOR_WARM_UP`)方式，即预热/冷启动方式。当系统长期处于低水位的情况下，当流量突然增加时，直接把系统拉升到高水位可能瞬间把系统压垮。通过“冷启动”，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间，避免冷系统被压垮。
> 
> 默认`coldFactor`为 3，即请求 QPS 从`threshold / 3`开始，经预热时⻓逐渐升至设定的QPS阈值。

![image](/assets/images/blog/springcloud/流控warm-up配置.png) 

![image](/assets/images/blog/springcloud/流控效果warm-up.png) 

### 2.6.1 场景
![image](/assets/images/blog/springcloud/限流冷启动.png) 


### 2.7 流控效果-匀速排队
> 匀速排队(`RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER`)方式，会严格控制请求通过的间隔时间，也即是让请求以均匀的速度通过，对应的漏桶算法。(削峰填谷)
> 
> 阈值类型必须设置为QPS，否则无效。

![image](/assets/images/blog/springcloud/流控-排队等待.png) 

上图表明： 假设`/order/test5`资源以每秒1次的请求处理能力处理数据，如果QPS超过1，则排队等待， 排队的请求最多只有10秒的预计处理时长。 如果后面再来的请求计算预计等待时长超过10秒，则拒绝该请求。

这种方式主要用于处理间隔性突发的流量，例如消息队列。想象一下这样的场景，在某一秒有大量的请求到来，而接下来的几秒则处于空闲状态，我们希望系统能够在接下来的空闲期间逐渐处理这些请求，而不是在第一秒直接拒绝多余的请求。

> 注意：匀速排队模式暂时不支持 QPS > 1000 的场景。


## Sentinel服务熔断降级
> 除了流量控制以外，对调用链路中不稳定的资源进行熔断降级也是保障高
> 可用的重要措施之一。一个服务常常会调用别的模块，可能是另外的一个
> 远程服务、数据库，或者第三方API等。例如，支付的时候，可能需要远
> 程调用银联提供的API; 查询某个商品的价格，可能需要进行数据库查询。
> 然而，这个被依赖服务的稳定性是不能保证的。如果依赖的服务出现了不
> 稳定的情况，请求的响应时间变⻓，那么调用服务的方法的响应时间也会
> 变⻓，线程会产生堆积，最终可能耗尽业务自身的线程池，服务本身也变
> 得不可用。

### 3.1 熔断降级—慢调用比例
> 慢调用比例(SLOW_REQUEST_RATIO): 选择以慢调用比例作为阈值，需
> 要设置允许的慢调用RT(即最大的响应时间)，请求的响应时间大于该值
> 则统计为慢调用。当单位统计时⻓( statIntervalMs )内请求数目大于设置
> 的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时⻓内请
> 求会自动被熔断。经过熔断时⻓后熔断器会进入探测恢复状态(HALF-
> OPEN状态)，若接下来的一个请求响应时间小于设置的慢调用 RT 则结束 
> 熔断，若大于设置的慢调用RT则会再次被熔断。

<br>
> 断路器的工作流程: 
> 
> 一旦熔断，断路器的状态是Open(所有的请求都不能进来) 
> 
> 当熔断时⻓结束，断路器的状态是half-Open(可以允许一个请求进来) 
> 
> 如果接下来的请求正常，断路器的状态是close(资源就自恢复) 
> 
> 如果接下来的请求不正常，断路器的状态是open

<br>
> 触发条件
> 
> 1s最小请求数必须大于5
> 
> 慢请求/总请求 > 0.6

![image](/assets/images/blog/springcloud/熔断降级-慢调用比例.png) 

演示代码：

```
@RequestMapping("test5")
public String test5(String flag)throws  Exception{
    if (flag == null) {
        Thread.sleep(1800);
    }
    return "test5";
}
```

### 3.2 熔断降级—异常比例

> 异常比例 ( ERROR_RATIO ): 当单位统计时⻓( statIntervalMs )内请求数
> 目大于设置的 最小请求数目，并且异常的比例大于阈值，则接下来的熔断
> 时⻓内请求会自动被熔断。 经过熔断时⻓后熔断器会进入探测恢复状态
> (HALF-OPEN 状态)，若接下来的一个请求成功完成(没有错误)则结束熔
> 断，否则会再次被熔断。异常比率的阈值范围是 [0.0, 1.0] ，代表 0% - 
> 100%。

<br>
> 断路器的工作流程: 
> 
> 一旦熔断，断路器的状态是Open(所有的请求都不能进来) 
> 
> 当熔断时⻓结束，断路器的状态是half-Open(可以允许一个请求进来) 
> 
> 如果接下来的请求正常，断路器的状态是close(资源就自恢复) 
> 
> 如果接下来的请求不正常，断路器的状态是open

<br>
> 触发条件
> 
> 1s最小请求数必须大于5
> 
> 1s内异常请求/总请求 > 0.6

![image](/assets/images/blog/springcloud/熔断降级-异常比例.png) 

演示代码：
```
@RequestMapping("test6")
public String test6(String flag)throws  Exception{
	if (flag == null) {
		throw new IllegalArgumentException("参数异常");
	}
   	 return "test6";
}
```

### 3.3 熔断降级—异常数

> 异常数 ( ERROR_COUNT ): 当单位统计时⻓内的异常数目超过阈值之后
> 会自动进行熔断。经过熔断时⻓后熔断器会进入探测恢复状态(HALF-
> OPEN 状态)，若接下来的一个请求成功完成(没有错误)则结束熔断，否则
> 会再次被熔断。

<br>
> 触发条件：
> 
> 每秒请求数必须大于5
>
> 每秒的异常数必须大于10(设置的阈值)

![image](/assets/images/blog/springcloud/熔断降级-异常数.png) 

## Sentinel热点key规则

### 4.1 热点
何为热点？热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：

* 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
* 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制

热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。

![image](/assets/images/blog/springcloud/热点限流.png) 

Sentinel 利用 LRU 策略统计最近最常访问的热点参数，结合令牌桶算法来进行参数级别的流控。

### 4.2 热点key参数规则
> 触发条件
> 
> 1. 必须有一个参数，参数的值可以任意。 e.g. `http://localhost:9002/order/test7?name=jack`
> 
> 2. 接口访问qps > 1

![image](/assets/images/blog/springcloud/热点key参数配置-1.png) 

![image](/assets/images/blog/springcloud/热点key参数配置-2.png) 

### 4.3 热点key参数规则—例外项

> 触发条件
> 
> 1. 第一个参数的值如果为jack，那么qps阈值为1
>  
> 2. 第一个参数的值如果为rose，那么qps阈值为5
> 
> 3. 第一个参数的值如果为其他，那么qps阈值为1
> 

![image](/assets/images/blog/springcloud/热点key参数配置-3.png) 

注意事项：
使用@SentinelResource注解使热点参数限流功能生效。

```
@RequestMapping("test7")
@SentinelResource("hotkey-test7")
public String test7(String name, Integer age) {
      System.out.println("name" + name);
      System.out.println("age" + age);
      return "test7";
}
```

## Sentinel-权限规则
很多时候，我们需要根据调用方来限制资源是否通过，这时候可以使用 Sentinel 的黑白名单控制的功能。黑白名单根据资源的请求来源（origin）限制资源是否通过，若配置白名单则只有请求来源位于白名单内时才可通过；若配置黑名单则请求来源位于黑名单时不通过，其余的请求通过。

> 调用方信息通过 ContextUtil.enter(resourceName, origin) 方法中的 origin 参数传入。

![image](/assets/images/blog/springcloud/sentinel权限规则流程.png) 

### 示例

> /goods/findById/{id}对于cloud-order不能调用

![image](/assets/images/blog/springcloud/黑名单规则设置.png) 


**从请求头获取source**

```
# 在cloud-goods微服务中设置
package com.jq.sentinel;

import com.alibaba.csp.sentinel.adapter.spring.webmvc.callback.RequestOriginParser;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;

@Component
public class CustomerRequestOriginParser implements RequestOriginParser {

    @Override
    public String parseOrigin(HttpServletRequest httpServletRequest) {
        System.out.println("come in");
        
        //获取请求头的source
        String source1 = httpServletRequest.getHeader("source");
        System.out.println(source1);
        return source1;
    }
}

```

**openFeign拦截器统一设置请求头**

```
# 在cloud-order中使用openFeign发请求时进行设置
package com.jq.feign;

import feign.RequestInterceptor;
import feign.RequestTemplate;
import org.springframework.stereotype.Component;

@Component
public class CustomerRequestInterceptor implements RequestInterceptor {
    public void apply(RequestTemplate requestTemplate) {
        //统一设置请求头
        requestTemplate.header("source", "cloud-order");
    }
}
```

**Order微服务调用Goods微服务**

```
@Autowired
private GoodsApi goodsApi;

@RequestMapping("test8")
public String test8() {
      Goods byId = goodsApi.findById("1");
      System.out.println(byId);
      return "test8";
}
```

![image](/assets/images/blog/springcloud/黑名单block.png) 


## Sentinel-针对来源

![image](/assets/images/blog/springcloud/sentinel-针对来源.png) 

```
@Component
public class CustomerRequestOriginParser  implements
RequestOriginParser {
    public String parseOrigin(HttpServletRequest httpServletRequest) {
        String source = httpServletRequest.getHeader("source");
        System.out.println("come in");
        System.out.println("source"+source);
        if (source != null) {
            return  source;
	    }
        return null;
    }
}
```

## 系统规则

Sentinel 系统自适应限流从整体维度对应用入口流量进行控制，结合应用的 Load、CPU 使用率、总体平均 RT、入口 QPS 和并发线程数等几个维度的监控指标，通过自适应的流控策略，让系统的入口流量和系统的负载达到一个平衡，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

系统保护规则是从应用级别的入口流量进行控制，从单台机器的 load、CPU 使用率、平均 RT、入口 QPS 和并发线程数等几个维度监控应用指标，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

系统保护规则是应用整体维度的，而不是资源维度的，并且仅对入口流量生效。入口流量指的是进入应用的流量（`EntryType.IN`），比如 Web 服务或 Dubbo 服务端接收的请求，都属于入口流量。

**系统规则支持以下的模式**：

* **Load 自适应**（仅对 Linux/Unix-like 机器生效）：系统的 load1 作为启发指标，进行自适应系统保护。当系统 load1 超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR 阶段）。系统容量由系统的 maxQps * minRt 估算得出。设定参考值一般是 CPU cores * 2.5。
* **CPU usage**（1.5.0+ 版本）：当系统 CPU 使用率超过阈值即触发系统保护（取值范围 0.0-1.0），比较灵敏。
* **平均 RT**：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒。
* **并发线程数**：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护。
* **入口 QPS**：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护。

![image](/assets/images/blog/springcloud/sentinel-系统规则.png) 

##  @Sentinel规则持久化

### 8.1 流控自定义异常处理

![image](/assets/images/blog/springcloud/sentinel-自定义异常处理-1.png) 

```
// 主逻辑
@RequestMapping("test9")
@SentinelResource(value = "test9", blockHandler = "")
public String test9(String name) {
      return "test9";
}

// 备选逻辑
public String test9Handler(String name, BlockException exception) {
      return "系统繁忙，请稍后重试";
}
```


调用test9接口，点击簇点链路并给资源名为test9的资源(因为现在使用SentinelResource注解)设置流控规则，如下图所示

![image](/assets/images/blog/springcloud/sentinel-自定义异常处理-2.png) 

### 8.2 流控自定义异常优化

```
//主逻辑
@RequestMapping("test9")
@SentinelResource(value = "test9", blockHandler = "testBlockHandler", blockHandlerClass = BlockException.class)
public Map test9(String name) {
      return new HashMap() {{
          put("code", 200);
          put("msg", "test9");
      }};
}
```

```
package com.jq.sentinel;

import com.alibaba.csp.sentinel.slots.block.BlockException;

import java.util.HashMap;
import java.util.Map;

public class BlockExceptionHandler {
    public static Map test9Handler(String name, BlockException exception) {
        System.out.println(name);
        return new HashMap() {{
           put("code", 200);
           put("msg", "系统繁忙，请稍后重试");
        }};
    }
}

```

### 8.3 熔断降级自定义处理

```
@RequestMapping("test10")
@SentinelResource(value = "test10", fallbackClass = DegradeExceptionHandler.class, fallback = "testFallBack")
public String test10(String flag){
     if (flag == null) {
         throw new IllegalArgumentException("参数异常");
     }
     return "test10";
}
```

```
package com.jq.sentinel;

import com.alibaba.csp.sentinel.slots.block.degrade.DegradeException;

public class DegradeExceptionHandler {
    public static String testFallBack(String flag, Throwable e) {
        if (e instanceof DegradeException) {
            return "触发了熔断系统";
        } else {
            return "其他异常系统";
        }
    }
}
```

### 8.5 全局异常处理

```
package com.jq.exception;

import com.alibaba.csp.sentinel.slots.block.authority.AuthorityException;
import com.alibaba.csp.sentinel.slots.block.degrade.DegradeException;
import com.alibaba.csp.sentinel.slots.block.flow.FlowException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.HashMap;
import java.util.Map;

@ControllerAdvice
public class CustomExceptionHandler {

    // 流控规则，全局异常处理
    @ExceptionHandler(FlowException.class)
    @ResponseBody
    public Map handlerFlowException() {
        return new HashMap() {{
           put("code", "5xx");
           put("msg", "系统繁忙，稍后重试");
        }};
    }

    // 熔断降级规则，全局异常处理
    @ExceptionHandler(DegradeException.class)
    @ResponseBody
    public Map handlerDegradeException() {
        return new HashMap() {{
            put("code", "5xx");
            put("msg", "系统开小差，稍后重试");
        }};
    }

    // 权限规则，全局异常处理
    @ExceptionHandler(AuthorityException.class)
    @ResponseBody
    public Map handlerAuthorityException() {
        return new HashMap() {{
            put("code", "5xx");
            put("msg", "没有权限访问");
        }};
    }

}

```

## Sentinel规则持久化

### 9.1 原始模式

如果不做任何修改，Dashboard的推送规则方式是通过API将规则推送至客户端并直接更新到内存中。

这种做法的好处是简单，无依赖;

坏处是应用重启，规则就会消失，仅用于简单测试，不能用于生产环境。

![image](/assets/images/blog/springcloud/sentinel-规则持久化-原始模式.png) 

### 9.2 Pull模式

`FileRefreshableDataSource`定时从指定文件中读取规则JSON文件【图中的本地文件】，如果发现文件发生变化，就更新规则缓存。

`FileWritableDataSource`接收控制台规则推送，并根据配置，修改规则JSON文件【图中的本地文件】。

![image](/assets/images/blog/springcloud/sentinel-规则持久化-pull模式.png) 

#### 9.2.1 pom依赖
```
<dependency>
  <groupId>com.alibaba.csp</groupId>
  <artifactId>sentinel-datasource-extension</artifactId>
</dependency>
```

#### 9.2.2 核心代码

```
package com.jq.sentinel;

import com.alibaba.csp.sentinel.command.handler.ModifyParamFlowRulesCommandHandler;
import com.alibaba.csp.sentinel.datasource.*;
import com.alibaba.csp.sentinel.init.InitFunc;
import com.alibaba.csp.sentinel.slots.block.authority.AuthorityRule;
import com.alibaba.csp.sentinel.slots.block.authority.AuthorityRuleManager;
import com.alibaba.csp.sentinel.slots.block.degrade.DegradeRule;
import com.alibaba.csp.sentinel.slots.block.degrade.DegradeRuleManager;
import com.alibaba.csp.sentinel.slots.block.flow.FlowRule;
import com.alibaba.csp.sentinel.slots.block.flow.FlowRuleManager;
import com.alibaba.csp.sentinel.slots.block.flow.param.ParamFlowRule;
import com.alibaba.csp.sentinel.slots.block.flow.param.ParamFlowRuleManager;
import com.alibaba.csp.sentinel.slots.system.SystemRule;
import com.alibaba.csp.sentinel.slots.system.SystemRuleManager;
import com.alibaba.csp.sentinel.transport.util.WritableDataSourceRegistry;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.TypeReference;

import java.io.File;
import java.io.IOException;
import java.util.List;

/**
 * @Author jiaqi.xu
 * @Date 2022/11/06 18:01
 */
public class FileDataSourceInit implements InitFunc {
    @Override
    public void init() throws Exception {
        // TIPS: 如果你对这个路径不喜欢，可修改为你喜欢的路径
        String ruleDir = System.getProperty("user.dir") +"/sentinel/rules";
        String flowRulePath = ruleDir + "/flow-rule.json";
        String degradeRulePath = ruleDir + "/degrade-rule.json";
        String systemRulePath = ruleDir + "/system-rule.json";
        String authorityRulePath = ruleDir + "/authority-rule.json";
        String paramFlowRulePath = ruleDir + "/param-flow-rule.json";
        this.mkdirIfNotExits(ruleDir);
        this.createFileIfNotExits(flowRulePath);
        this.createFileIfNotExits(degradeRulePath);
        this.createFileIfNotExits(systemRulePath);
        this.createFileIfNotExits(authorityRulePath);
        this.createFileIfNotExits(paramFlowRulePath);

        // 流控规则
        ReadableDataSource<String, List<FlowRule>> flowRuleRDS = new FileRefreshableDataSource<>(
                flowRulePath,
                flowRuleListParser
        );
        // 将可读数据源注册至FlowRuleManager
        // 这样当规则文件发生变化时，就会更新规则到内存
        FlowRuleManager.register2Property(flowRuleRDS.getProperty());
        WritableDataSource<List<FlowRule>> flowRuleWDS = new FileWritableDataSource<>(
                flowRulePath,
                this::encodeJson
        );
        // 将可写数据源注册至transport模块的WritableDataSourceRegistry中
        // 这样收到控制台推送的规则时，Sentinel会先更新到内存，然后将规则写入到 文件中
        WritableDataSourceRegistry.registerFlowDataSource(flowRuleWDS);

        // 降级规则
        ReadableDataSource<String, List<DegradeRule>> degradeRuleRDS = new FileRefreshableDataSource<>(
                        degradeRulePath,
                        degradeRuleListParser
                );
        DegradeRuleManager.register2Property(degradeRuleRDS.getProperty());
        WritableDataSource<List<DegradeRule>> degradeRuleWDS = new FileWritableDataSource<>(
                degradeRulePath,
                this::encodeJson
        );
        WritableDataSourceRegistry.registerDegradeDataSource(degradeRuleWDS);

        // 系统规则
        ReadableDataSource<String, List<SystemRule>> systemRuleRDS = new FileRefreshableDataSource<>(
                        systemRulePath,
                        systemRuleListParser
                );
        SystemRuleManager.register2Property(systemRuleRDS.getProperty());
        WritableDataSource<List<SystemRule>> systemRuleWDS = new FileWritableDataSource<>(
                systemRulePath,
                this::encodeJson
        );
        WritableDataSourceRegistry.registerSystemDataSource(systemRuleWDS);

        // 授权规则
        ReadableDataSource<String, List<AuthorityRule>> authorityRuleRDS = new FileRefreshableDataSource<>(
                authorityRulePath,
                authorityRuleListParser
        );
        AuthorityRuleManager.register2Property(authorityRuleRDS.getProperty()
        );
        WritableDataSource<List<AuthorityRule>> authorityRuleWDS = new
                FileWritableDataSource<>(
                authorityRulePath,
                this::encodeJson
        );
        WritableDataSourceRegistry.registerAuthorityDataSource(authorityRuleWDS);

        // 热点参数规则
        ReadableDataSource<String, List<ParamFlowRule>> paramFlowRuleRDS = new FileRefreshableDataSource<>(
                paramFlowRulePath,
                paramFlowRuleListParser
        );
        ParamFlowRuleManager.register2Property(paramFlowRuleRDS.getProperty());
        WritableDataSource<List<ParamFlowRule>> paramFlowRuleWDS = new FileWritableDataSource<>(
                paramFlowRulePath,
                this::encodeJson
        );
        ModifyParamFlowRulesCommandHandler.setWritableDataSource(paramFlowRuleWDS);
    }

    private Converter<String, List<FlowRule>> flowRuleListParser = source -> JSON.parseObject(
                    source,
                    new TypeReference<List<FlowRule>>() {}
            );

    private Converter<String, List<DegradeRule>> degradeRuleListParser = source -> JSON.parseObject(
            source,
            new TypeReference<List<DegradeRule>>() {}
    );

    private Converter<String, List<SystemRule>> systemRuleListParser = source -> JSON.parseObject(
            source,
            new TypeReference<List<SystemRule>>() {}
    );

    private Converter<String, List<AuthorityRule>>
            authorityRuleListParser = source -> JSON.parseObject(
            source,
            new TypeReference<List<AuthorityRule>>() {}
    );

    private Converter<String, List<ParamFlowRule>> paramFlowRuleListParser = source -> JSON.parseObject(
            source,
            new TypeReference<List<ParamFlowRule>>() {}
    );

    private void mkdirIfNotExits(String filePath) throws IOException {
        File file = new File(filePath);
        if (!file.exists()) {
            file.mkdirs();
        }
    }
    private void createFileIfNotExits(String filePath) throws
            IOException {
        File file = new File(filePath);
        if (!file.exists()) {
            file.createNewFile();
        }
    }
    private <T> String encodeJson(T t) {
        return JSON.toJSONString(t);
    }
}

```

### 9.2.3 配置

在项目的 resources/META-INF/services 目录下创建文件，

文件名: com.alibaba.csp.sentinel.init.InitFunc，内容为如下:

```
# 改成上面FileDataSourceInit的包名类名全路径即可。
com.jq.sentinel.FileDataSourceInit
```

### 9.2.4 优缺点分析

* 优点：简单易懂；没有多余依赖(如配置中心、缓存等)
* 缺点：
	1. 由于规则是用 FileRefreshableDataSource 定时更新的，所以规则更新会有延迟。如果FileRefreshableDataSource定时时间过大，可能⻓时间延迟; 如果FileRefreshableDataSource过小，又会影响性能;
	2. 规则存储在本地文件，如果有一天需要迁移微服务，那么需要把规则文件一起迁移， 否则规则会丢失。

### 9.3 Push模式

生产环境下一般更常用的是 push 模式的数据源。对于 push 模式的数据源, 如远程配置中心(ZooKeeper, Nacos, Apollo等等), 推送的操作不应由 Sentinel 客户端进行，而应该经控制台统一进行管理，直接进行推送，数据源仅负责获取配置中心推送的配置并更新到本地。

因此推送规则正确做法应该是 配置中心控制台/Sentinel 控制台 → 配置中心 → Sentinel 数据源 → Sentinel，而不是经 Sentinel 数据源推送至配置中心。流程如下：

![image](/assets/images/blog/springcloud/sentinel-规则持久化-push模式.png) 

### 9.3.1 控制台改造
需要改造sentinel源码，具体是sentinel-dashboard模块的代码，因为dashboard默认情况下是将配置的规则往微服务端的内存中推送的，并不是往配置中心去推。

### 9.3.2 微服务端
```
<!-- sentinel -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
<!-- sentinel的nacos持久化 --> <dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

### 9.3.3 添加配置
```
spring:
  cloud:
    sentinel:
      transport:
port: 8721
dashboard: localhost:8888 eager: true #饥饿加载 web-context-unify: false datasource:
flow: nacos:
            server-addr: ${nacos.server-addr}
            username: ${nacos.username}
            password: ${nacos.password}
            namespace: ${nacos.namespace}
            groupId: SENTINEL_GROUP
            dataId: ${spring.application.name}-flow-rules
            rule-type: flow
        degrade:
          nacos:
            server-addr: ${nacos.server-addr}
            username: ${nacos.username}
            password: ${nacos.password}
            namespace: ${nacos.namespace}
            groupId: SENTINEL_GROUP
            dataId: ${spring.application.name}-degrade-rules
            rule-type: degrade
            param-flow:
          nacos:
            server-addr: ${nacos.server-addr}
            username: ${nacos.username}
            password: ${nacos.password}
            namespace: ${nacos.namespace}
            groupId: SENTINEL_GROUP
            dataId: ${spring.application.name}-param-rules
            rule-type: param-flow
        system:
          nacos:
            server-addr: ${nacos.server-addr}
            username: ${nacos.username}
            password: ${nacos.password}
            namespace: ${nacos.namespace}
            groupId: SENTINEL_GROUP
            dataId: ${spring.application.name}-system-rules
            rule-type: system
        authority:
          nacos:
            server-addr: ${nacos.server-addr}
            username: ${nacos.username}
            password: ${nacos.password}
            namespace: ${nacos.namespace}
            groupId: SENTINEL_GROUP
            dataId: ${spring.application.name}-authority-rules
            rule-type: authority
nacos:
  server-addr: localhost:8848
  username: nacos
  password: nacos
  namespace: sentinel
```
