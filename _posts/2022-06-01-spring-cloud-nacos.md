---
title: "SpringCloud Nacos"
layout: post
date: 2022-06-01 22:44
tag:
- spring cloud
- nacos
- 注册中心
- 配置中心
star: true
category: blog
author: jiaqixu
description: note for SpringCloud Alibaba Nacos
---

### 目录
* TOC
{:toc}

## 服务注册与发现
### 2.1 服务注册与发现
> 服务器注册，就是将提供某个服务的模块信息(通常是这个服务的ip和端口)注册到1个公共的组件上去(比如: zookeeper\consul\eureka\nacos)。
> 
> 服务发现，就是新注册的这个服务模块能够及时的被其他调用者发现。不管是服务新增 和服务删减都能实现自动发现。

### 2.2 注册中心对比
**nacos**: 是阿里开源的，经过了阿里实践的。
**eureka**:netflix公司的，现在不维护了，不开源了。
**Consul** : HashiCorp 公司推出的开源产品，用于实现分布式系统的服务发现、服务隔离、服务配置。

![image](/assets/images/blog/springcloud/注册中心对比.png)

## nacos简介与安装
官网：https://nacos.io/zh-cn/docs/what-is-nacos.html

### 3.1 架构与功能

**nacos架构**

![image](/assets/images/blog/springcloud/nacos架构.png)

**nacos功能**

* 名字服务(Naming Service)
	`命名服务是指通过指定的名字来获取资源或者服务的地址，提供者的信息。`
	
	**官网的描述**
	`提供分布式系统中所有对象(Object)、实体(Entity)的“名字”到关联的元数据之间的映射管理服务，例如 ServiceName -> Endpoints Info, Distributed Lock Name -> Lock Owner/Status Info, DNS Domain Name -> IP List, 服务发现和 DNS 就是名字服务的2大场景。` 
	
* 服务配置(Configuration Service)
	`动态配置服务让您能够以中心化、外部化和动态化的方式管理所有环境的配置。动态配置消除 了配置变更时重新部署应用和服务的需要。配置中心化管理让实现无状态服务更简单，也让按 需弹性扩展服务更容易。`


### 3.2 安装
下载地址：https://github.com/alibaba/nacos/tags

1. 解压安装
`https://github.com/alibaba/nacos/releases/tag/1.4.1`

2. 配置
	* 进入目录 `/Users/jiaqi/personal_projects/springcloud-alibaba/nacos/conf/application.properties`
	* 修改数据配置
		
		```sql
      	### If use MySQL as datasource:
		spring.datasource.platform=mysql

		### Count of DB:
		db.num=1

		### Connect URL of DB:
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=root
db.password.0=123456
		```

3. 创建数据库以及表 
	* 创建数据库名为nacos的数据库，并执行`conf/nacos-mysql.sql`文件来创建对应数据库表。

4. 配置startup.sh，以standalone方式启动

	```	
	export SERVER="nacos-server"
	export MODE="standalone"
	export FUNCTION_MODE="all"
	export MEMBER_LIST=""
	export EMBEDDED_STORAGE=""
	while getopts ":m:f:s:c:p:" opt
	```

5. 启动 `./startup.sh -m standalone` ,  成功。

	![image](/assets/images/blog/springcloud/nacos-console.png)

### 3.3 注册中心工作流程
 
**服务注册**： Nacos Client会通过发送REST请求的方式向Nacos Server注册自己的服务，提供自身的元数据，比如ip地址、端口等信息。Nacos Server接收到注册请求后，就会把这些元数据信息存储在一个双层的内存Map中。

**服务心跳**： 在服务注册后，Nacos Client会维护一个定时心跳来持续通知Nacos Server，说明服务一直处于可用状态，防止被踢出。默认5s发送一次心跳。

**服务同步**： Nacos Server集群之间会互相同步服务实例，用来保证服务信息的一致性。

**服务发现**： 服务消费者(Nacos Client)在调用服务提供者的服务时，会发送一个REST请求给Nacos Server, 获取上面注册的服务清单，并且缓存在Nacos Client本地，同时会在Nacos Client本地开启一个定时任务定时拉去服务端最新的注册表信息更新到本地缓存。

**服务健康检查**：Nacos Server会开启一个定时任务用来检查注册服务实例的健康情况，对于超过15s没有收到客户端心跳的实例会将它的healthy属性设置为false(客户端服务发现时不会发现)，如果某个实例超过30s没有收到心跳，直接剔除该实例(被剔除的实例如果恢复发送心跳则会重新注册）

## 微服务入门案例

### 4.1 boot与cloud版本
```
springboot：提供了快速开发微服务的能力
springcloud提供了微服务治理的能力(服务注册与发现、服务降级、限流、熔断、网关、负载均衡、配置中心等)，为微服务开发提供了全家桶服务。
```

springboot的版本查看地址: https://spring.io/projects/spring-boot#learn<br>
springcloud的版本查看地址:https://spring.io/projects/spring-cloud#overview<br>
详细版本对应信息查看:https://start.spring.io/actuator/info

**注意**
```
如果采用springboot和springcloud(springcloud netflix)那么使用以上版本对应就ok了， 但是如果要使用alibaba的组件(nacos、sentinel、RocketMQ、Seata)必须使用springcloud alibaba
```

### 4.2 springcloud-alibaba
springcloud与springcloud-alibaba关系

>◆ 我们通常说的SpringCloud，泛指Spring Cloud Netflix，也是springcloud第一代
> 
> ◆ SpringCloud Alibaba是SpringCloud的子项目，是阿里巴巴结合自身微服务实践
> 
> ◆ SpringCloud Alibaba符合SpringCloud标准，依赖于springcloud

![image](/assets/images/blog/springcloud/springcloud组件对比.png)

### 4.3 确定版本
通过查看springcloud alibaba官网确定https://github.com/alibaba/spring-cloud-alibaba/wiki/

### 4.4 创建父工程

```
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.2.5.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR8</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

        </dependencies>
    </dependencyManagement>
```

### 4.5 服务提供者

#### 4.5.1 pom.xml
```
   <dependencies>
        <!-- Web场景依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 端点监控的场景依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!-- nacos场景依赖 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>
```

#### 4.5.2 application.xml
```
spring:
  application:
name: cloud-goods #服务名称，必须，保证唯一 cloud:
    nacos:
      discovery:
server-addr: localhost:8848 #指定nacos-server的地址 username: nacos
password: nacos
server:
  port: 9001
```

#### 4.5.3 启动类加注解
```
package com.jq;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient //开启服务注册与发现功能
public class GoodsApp {
    public static void main(String[] args) {
        SpringApplication.run(GoodsApp.class, args);
    }
}
```

#### 4.5.4 查询商品接口
```
package com.jq.controller;

import com.jq.Goods;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("goods")
public class GoodsController {
    @RequestMapping("findById/{id}")
    public Goods findById(@PathVariable String id) {
        System.out.println("id" + id);
        return new Goods("小米", 99);
    }
}
```

### 4.6 服务消费者

#### 4.6.1 application.yml
```
spring:
  application:
    name: cloud-order # 服务名称，必须，保证唯一
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 # 指定nacos-server的地址
        username: nacos
        password: nacos
server:
  port: 9002

```

#### 4.6.2 启动类加注解
```java
@SpringBootApplication
@EnableDiscoveryClient
public class OrderApp {
    public static void main(String[] args) {
        SpringApplication.run(OrderApp.class, args);
    }

    @Bean
    // 让ribbon拦截RestTemplate发出的所有的请求
    // ribbon获取url中的service name
    // 从nacos注册中心获取实例列表
    // 负责从实例列表中通过相应的负载均衡算法，获取一个实例
    // RestTemplate请求实例
    @LoadBalanced
    public RestTemplate initRestTemplate() {
        return new RestTemplate();
    }
}
```

#### 4.6.3 保存订单接口
```
@RestController
@RequestMapping("order")
public class OrderController {

    @Autowired
    private RestTemplate restTemplate;

    @RequestMapping("save")
    public Map save() {
        // 远程调用cloud-goods服务，获取goods信息，发送http请求(httpclient) => resttemplate
        String serviceName = "cloud-goods";
        String url = "http://" + serviceName + "/goods/findById/1";
        Goods goods = restTemplate.getForObject(url, Goods.class);
        System.out.println(goods);

        // 2. 保存订单(本地调用)
        System.out.println("save order success.");

        // 3. TODO: 扣库存

        return new HashMap() {{
            put("code", "200");
            put("msg", "success");
        }};
    }
}
```


### 4.8 领域模型
```
nacos的服务由三元组唯一确定 (namespace、group、servicename) 
nacos的配置由三元组唯一确定 (namespace、group、dataId)
不同的namespace是相互隔离的，相同namespace但是不同的group也是相互隔离的
默认的namespace是public ，不能删除 
默认的group是DEFAULT-GROUP
```

1. 创建namespace

![image](/assets/images/blog/springcloud/创建命名空间.png)

2. 发布服务到指定的namespace

![image](/assets/images/blog/springcloud/发布服务到指定命名空间.png)


## RestTemplate
> 实现服务间远程调用

### 5.1 简介
```
RestTemplate是java模拟浏览器发送http请求的工具类
RestTemplate基于`Apache`的`HttpClient`实现。HttpClient使用起来太过频繁。
spring提供了一种简单便捷的模版类来进行操作，这就是`RestTemplate`
```

### 5.2 ForObject
> 返回的是响应结果

get请求

```java
Map goods = restTemplate.getForObject(BaseURL+"findGoodsById? goodsId=12", Map.class);
System.out.println(goods.get("goodsName"));
```

post请求

```java
Map goods = restTemplate.postForObject(BaseURL + "/save", new Goods("huawei", 99.99), Map.class);
System.out.println(goods.get("code"));
```

### 5.3 ForEntity

> 返回的是响应体

get请求

```java
ResponseEntity<Goods> forEntity = restTemplate.getForEntity(BaseURL +
"findGoodsById?goodsId=12", Goods.class);
System.out.println("http status:"+forEntity.getStatusCode());
System.out.println("http response body:"+forEntity.getBody());
```

post请求

```java
ResponseEntity<Map> responseEntity = restTemplate.postForEntity(BaseURL
+ "/save", new Goods("huawei", 99.99), Map.class);
System.out.println("http status:"+responseEntity.getStatusCode());
System.out.println("http response body:"+responseEntity.getBody());
```

## 负载均衡器Ribbon
```
nacos:注册中心，解决服务的注册与发现
Ribbon:客户端的负载均衡器，解决的是服务实例列表的负载均衡的问题
```

### 6.1 简介
> Ribbon是Netflix公司开源的一个负载均衡的项目，是一个"客户端"负载均衡器，运行在客户端上

### 6.2 在项目中怎么使用

第一步： pom依赖
> springcloud alibaba对Ribbon做了兼容

![image](/assets/images/blog/springcloud/alibaba-ribbon.png)

第二步：@LoadBalanced注解
```
@Bean
@LoadBalanced
public RestTemplate restTemplate(){
    return new RestTemplate();
}
```

### 6.3 工作流程

![image](/assets/images/blog/springcloud/ribbon工作流程.png)

### 6.4 源码追踪

### 6.5 负载均衡策略

```
Ribbon核心组件IRule:根据特定算法从服务列表中选取一个需要访问的服务;
其中IRule是一个接口，有七个自带的落地实现类，可以实现不同的负载均衡算法规则:
```
![image](/assets/images/blog/springcloud/负载均衡策略.png)

**切换负载均衡策略**

1. 定义负载均衡策略
	
	```
	package com.rule;

	import com.netflix.loadbalancer.BestAvailableRule;
	import com.netflix.loadbalancer.IRule;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;

	@Configuration
	public class MyRule {
	    @Bean
	    public IRule getRule() {
      	 	return new BestAvailableRule();
    		}
	}
	```

2. 使用自定义的负载均衡策略，使生效

	```
	@SpringBootApplication
	@EnableDiscoveryClient
	@RibbonClient(name = "cloud-goods", configuration = {MyRule.class}) // 调用cloud-goods微服务	的时候，采用的负载均衡策略是MyRule
	public class OrderApp {
	    public static void main(String[] args) {
	        SpringApplication.run(OrderApp.class, args);
	    }

	    @Bean
	    // 让ribbon拦截RestTemplate发出的所有的请求
	    // ribbon获取url中的service name
	    // 从nacos注册中心获取实例列表
	    // 负责从实例列表中通过相应的负载均衡算法，获取一个实例
	    // RestTemplate请求实例
	    @LoadBalanced
	    public RestTemplate initRestTemplate() {
	        return new RestTemplate();
	    }
	}
	```


### 6.6 服务实例列表同步刷新

服务消费者会起一个定时任务去刷新实例列表
```
DynamicServerListLoadBalancer.updateListOfServers 
//从nacos server获取最新的实例列表 
NacosServerList.getServers
```

## nacos集群搭建

### 7.1 集群架构
http://nacos.com:port/openAPI<br>
域名 + SLB模式(内网SLB，不可暴露到公网，以免带来安全风险)，可读性好，而且换ip方便，推荐模式

![image](/assets/images/blog/springcloud/nacos集群架构.png)



### 7.2 集群搭建
> 伪集群:一台服务器搭建3台nacos 通过端口进行区分

#### 7.2.1 集群规划
![image](/assets/images/blog/springcloud/nacos集群规划.png)

#### 7.2.2 详细步骤
第一步: 上传nacos包到linux服务器并解压
`tar -zxvf nacos-server-1.4.1.tar.gz -C /export/server/`

第二步：修改nacos数据源
```
cd /export/server/nacos/conf/
vim application.properties
```

![image](/assets/images/blog/springcloud/修改nacos数据源.png)

创建数据库以及表

第三步: 修改/export/server/nacos/bin/startup.sh 的JAVA_OPT

![image](/assets/images/blog/springcloud/修改nacosJVM参数.png)


> 虚拟机内存调大至2G

```text
原设置
JAVA_OPT="${JAVA_OPT} -server -Xms2g -Xmx2g -Xmn1g - XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
修改后:
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m - XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=160m"
```

第四步: 配置/export/server/nacos/conf/cluster.conf配置文件

修改集群配置文件的文件名

`cp cluster.conf.example cluster.conf`

```
192.168.25.101:8848
192.168.25.101:8858
192.168.25.101:8868
```

第五步: 复制三份，同时修改监听端口

```
[root@zhuxm01 server]# cp nacos/ nacos8848 -r
[root@zhuxm01 server]# cp nacos/ nacos8858 -r
[root@zhuxm01 server]# cp nacos/ nacos8868 -r
```

第六步: 分别启动nacos实例

创建nacos-cluster-startup.sh

```
sh /export/server/nacos8848/bin/startup.sh
sh /export/server/nacos8858/bin/startup.sh
sh /export/server/nacos8868/bin/startup.sh
```

第七步：测试

```
spring.cloud.nacos.discovery.server-addr=www.nacos.com
```

第八步：配置nginx反向代理

```
upstream  nacos-cluster {
       server    192.168.25.101:8848;
       server    192.168.25.101:8858;
       server    192.168.25.101:8868;
} 

server {
	listen 80;
	server_name  www.nacos.com;
	#charset koi8-r;
	#access_log  logs/host.access.log  main;
	location / {
   		proxy_pass http://nacos-cluster/;
	}
}
```


## SpringCloud OpenFeign
>  作为Spring Cloud的子项目之一，Spring Cloud OpenFeign 是一种声明式、模板化的 HTTP 客户端，在 Spring Cloud 中使用 OpenFeign，可以做到使用 HTTP请求远程服务 时能与调用本地方法一样的编码体验，开发者完全感知不到这是远程方法，更感知不到 这是个 HTTP 请求。同时OpenFeign通过集成Ribbon实现客户端的负载均衡。
> 
>nacos-server : 注册中心，解决是服务的注册与发现 
>
>Ribbon:客户端负载均衡器，解决的是服务集群负载均衡的问题 
>
>OpenFeign:声明式 HTTP 客户端 、代替Resttemplate组件，实现远程调用
>

### 8.1 演示案例说明

>cloud-order为服务消费者、cloud-jifen为服务提供者 
>
>功能1:添加订单，生成一条积分记录 
>
>功能2:修改订单，修改积分记录 
>
>功能3:删除订单，删除积分记录 
>
>功能4:查询订单，获取积分记录

### 8.2 新建积分微服务

#### 8.2.1 pom依赖
```
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-
discovery</artifactId>
        </dependency>
    </dependencies>
```

#### 8.2.2 application.properties
```
spring:
  application:
    name: cloud-jifen # 服务名称，必须，保证唯一
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 # 指定nacos-server的地址
        username: nacos
        password: nacos
server:
  port: 9004
```

#### 8.2.3 启动类
```
@SpringBootApplication
@EnableDiscoveryClient
public class JifenApp {
    public static void main(String[] args) {
        SpringApplication.run(JifenApp.class, args);
    }
}

```

#### 8.2.4 暴露接口
```
@RestController
@RequestMapping("/jifen")
public class JifenController {
    @PostMapping(value = "/save")
    public Map save(@RequestBody Jifen jifen) {
        System.out.println("调用了积分保存接口");
        System.out.println(jifen);
        return new HashMap() {{
            put("isSuccess", true);
            put("msg", "save success");
        }};
    }

    @PostMapping(value = "/update")
    public Map update(@RequestBody Jifen jifen) {
        System.out.println(jifen);
        return new HashMap() {{
            put("isSuccess", true);
            put("msg", "update success");
        }};
    }

    @GetMapping(value = "/delete")
    public Map deleteById(Integer jifenId) {
        System.out.println("删除id为" + jifenId + "的积分信息");
        return new HashMap() {{
            put("isSuccess", true);
            put("msg", "delete success");
        }};
    }

    @GetMapping(value = "/{jifenId}")
    public Jifen findJifenById(@PathVariable Integer jifenId) {
        System.out.println("已经查询到" + jifenId + "积分数据");
        return new Jifen(jifenId, 12, jifenId + "号积分");
    }

    @GetMapping(value = "/search")
    public Jifen search(Integer uid, String type) {
        System.out.println("uid:" + uid + "type:" + type);
        return new Jifen(uid, 12, type);
    }

    @PostMapping(value = "/searchByEntity")
    public List<Jifen> searchMap(@RequestBody Jifen jifen) {
        System.out.println(jifen);
        List<Jifen> jifens = new ArrayList<Jifen>();
        jifens.add(new Jifen(110, 12, "下单积分"));
        jifens.add(new Jifen(111, 18, "支付积分"));
        return jifens;
    }
```

### 8.3 Openfeign使用

#### 8.3.1 openfeign依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

#### 8.3.2 开启openfeign
```
@SpringBootApplication
@EnableDiscoveryClient
//@RibbonClient(name = "cloud-goods", configuration = {MyRule.class}) // 调用cloud-goods微服务的时候，采用的负载均衡策略是MyRule
@EnableFeignClients(basePackages = {"com.api"})
public class OrderApp {
    public static void main(String[] args) {
        SpringApplication.run(OrderApp.class, args);
    }
}
```

#### 8.3.3 接口声明
```
@FeignClient("cloud-jifen") // 服务名称
@RequestMapping("/jifen")
public interface JifenApi {
    @PostMapping(value = "/save")
    public Map save(@RequestBody Jifen jifen);
}
```

#### 8.3.4 接口调用

**扫描openfeign接口**

```
@EnableFeignClients(basePackages = {"com.api"})
public class OrderApp {
    public static void main(String[] args) {
        SpringApplication.run(OrderApp.class, args);
    }
}
```

**调用**

```
@RestController
@RequestMapping("order")
public class OrderController {
    @Autowired
    private JifenApi jifenApi;

    @RequestMapping("save-test")
    public Map saveTest() {
        // 通过openfeign远程调用cloud-jifen服务的/jifen/save接口
        Jifen jifen = new Jifen(1, 10, "2");

        //url: http://cloud-jifen/jifen/save
        Map save = jifenApi.save(jifen);
        return save;
    }
}
```

### 8.4 Openfeign常用配置项
```
spring:
  application:
    name: cloud-order # 服务名称，必须，保证唯一
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 # 指定nacos-server的地址
        username: nacos
        password: nacos
server:
  port: 9002

ribbon:
  eager-load:
    clients: true # 开启饥饿加载，默认是懒加载
    client:  # 这里表明项目已启动的时候，就加载下面2个微服务的实例列表
      - cloud-jifen
      - cloud-goods
feign:
  client:
    config:
      cloud-jifen:
        connect-timeout: 1000
        read-timeut: 1 # 设置cloud-jifen相应的超时时间为1毫秒
      default: # 设置默认的超时时间
        connect-timeout: 1000
        read-timeout: 1

```

## Naocs配置中心

> 小结：
> 
> Nacos:注册中心，解决服务的注册与发现 
> 
> Ribbon:客户端的负载均衡器，服务集群的负载均衡 
> 
> OpenFeign:声明式的HTTP客户端，服务远程调用 
> 
> Nacos:配置中心，中心化管理配置文件
> 


### 9.1 为什么使用配置中心
![image](/assets/images/blog/springcloud/为什么使用配置中心.png)

### 9.2 主流配置中心对比
目前市面上用的比较多的配置中心有:Spring Cloud Config、Apollo、Nacos和Disconf等。
由于Disconf不再维护，下面主要对比一下Spring Cloud Config、Apollo和Nacos。

![image](/assets/images/blog/springcloud/配置中心对比.png)

```
从配置中心⻆度来看，性能方面Nacos的读写性能最高，Apollo次之，Spring Cloud Config 依赖Git场景不适合开放的大规模自动化运维API。 

功能方面Apollo最为完善，nacos具有Apollo大部分配置管理功能，而Spring CloudConfig 不带运维管理界面，需要自行开发。
 
Nacos的一大优势是整合了注册中心、配置中心功能，部署和操作相比Apollo都要直观简单，因 此它简化了架构复杂度，并减轻运维及部署工作。
```

### 9.3 配置管理领域模型

Namespace -> Group -> DataId

### 9.4 配置中心入门使用

1. 创建文件

	![image](/assets/images/blog/springcloud/配置中心使用-创建文件.png)

2. 新建配置

	![image](/assets/images/blog/springcloud/配置中心使用-新建配置.png)
	
	![image](/assets/images/blog/springcloud/配置中心使用-创建配置完成.png)
3. 服务端加载配置信息

	在cloud-jifen微服务的pom.xml中加入依赖
	
	```
	<dependency>
    	<groupId>com.alibaba.cloud</groupId>
	    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
	</dependency>
	```
	
	在resources目录下新建bootstrap.yml文件，内容如下
	
	```
	# 从配置中心加载配置文件
	# 文件名是通过公式来拼接 ${prefix}-${spring.profiles.active}.${file-extension}
	spring:
	  cloud:
	    nacos:
	      config:
	        server-addr: localhost:8848
	        namespace: test
	        group: DEFAULT_GROUP
	        username: nacos
	        password: nacos
	        prefix: cloud-jifen
	        file-extension: yml
	  profiles:
	    active: test
	```

4. 验证

	![image](/assets/images/blog/springcloud/配置中心使用-启动成功.png)
	


### 9.5 环境切换
**测试环境**

cloud-jifen-test.yml

```
spring:
  application:
    name: cloud-jifen # 服务名称，必须，保证唯一
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 # 指定nacos-server的地址
        username: nacos
        password: nacos
        namespace: test
        group: my-group
server:
  port: 9004
```

**生产环境**

cloud-jifen-product.yml

```
spring:
  application:
    name: cloud-jifen #服务名称，必须，保证唯一 
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #指定nacos-server的地址 
        username: nacos
        password: nacos
        namespace: pro
        group: my-group
server:
  port: 9008
```

**环境切换**

![image](/assets/images/blog/springcloud/nacos配置环境切换.png)


### 9.6 nacos配置动态刷新
> 动态刷新： 不停机动态修改配置，立即生效
> 

修改配置文件属性
![image](/assets/images/blog/springcloud/配置动态刷新.png)

```
@RestController
@RequestMapping("/jifen")
@RefreshScope
public class JifenController {
    @Value("${pic.url}")
    private String URL;

    @RequestMapping("test")
    public String test() {
        return URL;
    }
```

### 9.7 动态刷新连接池大小

**cloud-jifen**整合mybatis

```
        <!-- mybatis的起步依赖-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.3</version>
        </dependency>

        <!-- mysql驱动 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--   druid     -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>
```

**在nacos配置中心添加配置**

```
spring:
  datasource:
    druid:
      driver-class-name: com.mysql.jdbc.Driver
      username: root
      password: 123456
      url: jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=utf8&useSSL=false 
      max-active: 60#连接池配置
```

**启动cloud-jifen服务**

访问druid监控`http://localhost:9008/druid/datasource.html`，可以看到max-active为60; 修改配置为100，发现马上生效变成100.

### 9.8 nacos共享配置
1. 新建共享配置
	common.yml
	
	```
	spring:
	  datasource:
	    druid:
	      driver-class-name: com.mysql.jdbc.Driver
	      username: root
	      password: mysql1qaz@WSX
	      url: jdbc:mysql://127.0.0.1:3306/galaxy?useUnicode=true&characterEncoding=utf8&useSSL=false 
	      max-active: 100 #连接池配置
	  cloud:
	    nacos:
	      discovery:
	        server-addr: localhost:8848 #指定nacos-server的地址 
	        username: nacos
	        password: nacos
	        namespace: product
	        group: my-group
	```
2. cloud-jifen个性配置

	```
	spring:
	  application:
	    name: cloud-jifen #服务名称，必须，保证唯一 
	server:
	  port: 9008
	```
3. 加载共享配置

	```
	# 从配置中心加载配置文件
	# 文件名是通过公式来拼接 ${prefix}-${spring.profiles.active}.${file-extension}
	spring:
	  cloud:
	    nacos:
	      config:
	        server-addr: localhost:8848
	        namespace: product
	        group: DEFAULT_GROUP
	        username: nacos
	        password: nacos
	        prefix: cloud-jifen
	        file-extension: yml
	        shared-configs: # 加载共享配置文件 
	          - common.yml
	          - common1.yml
	        refreshable-dataids: common.yml, common1.yml
	  profiles:
	    active: product
	```
	
### 9.9 配置版本管理

![image](/assets/images/blog/springcloud/配置中心-版本管理.png)

	
	
