---
title: "Redis note"
layout: post
date: 2019-02-28 22:44
tag:
- Redis
- NoSQL
star: true
category: blog
author: jiaqixu
description: note for Redis
---

### 目录
- [Redis的安装](#Redis的安装)
- [Redis五大数据类型](#Redis五大数据类型)
    * [Redis的Key的基本命令操作](#redis的Key的基本命令操作)
    * [String类型以及命令操作](#string类型以及命令操作)
    * [原子性](#原子性)  
    * [List类型以及命令操作](#list类型以及命令操作)
    * [Set类型以及命令操作](#set类型以及命令操作)
    * [Hash类型以及命令操作](#hash类型以及命令操作)
    * [zset类型以及命令操作](#zset类型以及命令操作)
- [Redis配置文件](#redis配置文件)
- [Redis事务](#redis事务)


### Redis的安装
```text
1. https://redis.io/download
2. 创建一个Redis文件夹，在terminal下cd到该文件夹下，curl -O http://download.redis.io/releases/redis-5.0.5.tar.gz
3. tar xzf redis-5.0.5.tar.gz
4. cd redis-5.0.5
5. make(需要gcc，因为redis源码是C语言)
6. make install 默认安装路径/usr/local/bin, 也可以在安装时指定安装目录:e.g. make install PREFIX=/opt/module/redis

redis-server: Redis服务器启动命令
redis-cli: 客户端，操作入口
```


### Redis五大数据类型
#### Redis的Key的基本命令操作
```text
keys *  查询当前库的所有键
keys ?  ？表示占位符，相当于模糊查询的下划线
keys *[1-2] 
exists <key> 判断某个键是否存在
type <key> 查看键的类型
del <key> 删除某个键
expire <key> <seconds> 为键值设置过期时间
ttl <key> 查看还有多少秒过期， -1表示用不过期，-2表示已过期
dbsize 查看当前数据库key的数量 (Redis可以处理2的32次方个keys， key和value最大都是512MB)
flushdb 清空当前数据库
flushall 通杀全部库
```

#### String类型以及命令操作
* String是Redis最基本的类型，可以理解成与Memcached一摸一样的模型，一个key对应一个value。
* String类型是二进制安全的。意味着Redis的string可以包含任何数据。比如jpg图片或者序列化对象。
* String类型是Redis最基本的数据类型，一个Redis中字符串value最多可以是512MB。

* 相关命令操作
  ```text
  set <key> <value> 添加键值对,重复设置会覆盖之前的值
  get <key> 查询对应键值
  append <key> <value> 将给定的value追加到原值的末尾,返回追加字符串之后新的字符串的长度
  strlen <key> 获得key所对应value值的长度
  setnx <key> 只有key不存在时，设置key的值；如果key已经存在，则什么都不做返回0。
  
  incr <key> 
  将key中存储的数字值增1
  只能对数字值操作，如果为空，新增值为1
  
  decr <key>
  将key中存储的数字值减1
  只能对数字值操作，如果为空，新增值为-1
  
  incrby/decrby <key> <步长>
  将key中存储的数字值增减。自定义步长。
  
  mset <key1> <value1> <key2> <value2> ...
  同时设置一个或多个key-value对 (如果有一个失败就都失败，比如mset k1 v1 k2, 参数错误)
  
  mget <key1> <key2> <key3> ...
  同时获取一个或者多个value，对于不存在的key会对应返回(nil)
  
  msetnx <key1> <value1> <key2> <value2> ...
  同时设置一个或者多个key-value对，当且仅当所有给定key都不存在(如果有一个失败，就都失败)
  
  getrange <key> <起始位置> <结束位置>
  获得值的范围(包括边界)，索引值如果超过范围贼返回空字符串。
  e.g. getrange k1 0 -1 返回整个字符串
  
  setrange <key> <起始位置> <value>
  用<value>覆盖着写<key>所存储的字符串值，从<起始位置>开始。
  
  setex <key> <过期时间> <value>
  设置键值的同时，设置过期时间，单位秒
  
  getset <key> <value>
  以新换旧，设置了新值的同时返回旧值。
  
  set <key> <value> [EX seconds] [PX milliseconds] [NX|XX]
  EX: 过期时间，单位秒
  PX: 过期时间，单位毫秒
  NX: 当有的时候，什么都不做
  XX: 有的时候，也做
  ```

#### 原子性
所谓原子操作是指不会被线程调度机制打断的操作；这种操作一旦开始，就一直运行到结束，中间
不会有任何context switch(切换到另一个线程)。

1）在单线程中，能够在单条指令中完成的操作都可以认为是"原子操作"，因为中断只能发生于指令之间。

2）在多线程中，不能被其他进程(线程)打断的操作就叫原子操作。

Redis单命令的原子性主要得益于Redis的单线程。

#### List类型以及命令操作
* 单键多值(也就是说key是一个，value是多个，其实合在一起还是字符串，只是结构类似于list，还是key-value对)。
* Redis列表是简单的字符串列表，按照插入顺序排序。可以添加一个元素到列表的头部(左边)后者尾部(右边)。
* 它的底层实际是个双向链表，对两端的操作性能很好，通过索引下标的操作中间的节点性能会较差。
  
    ![image](/assets/images/blog/redis-1.png)
* 相关命令操作
  ```text
  lpush/rpush <key> <value1> <value2> <value3> ...
  从左边/右边插入一个或者多个值
  e.g. lpush l1 v1 v2 v3 v4
  
  lpop/rpop <key>
  从左边/右边吐出一个值; 值在键在，值光键亡(换句话说，当value都pop光之后，key也会自动销毁).
  
  rpoplpush <key1> <key2>
  从<key1>列表右边吐出一个值，插到<key2>列表左边.
  
  lrange <key> <start> <stop>
  按照索引下标获得元素(从左到右)
  e.g. lpush l3 v1 v2 v3 v4 v5
       lrange l3 0 -1
  
  lindex <key> <index>
  按照索引下标获得元素(从左到右)
  e.g. lindex l3 0 # 输出"v5"
       lindex l3 -1 # 输出"v1"
       lindex l3 10 # 超过索引的话，不会报错，输出(nil)
       
  llen <key> 获取列表长度
  
  linsert <key> before/after <value> <new_value>
  在<value>的前面/后面插入<new_value>值
  
  lrem <key> <n> <value>
  当n>0, 从左到右删除n个value
  当n<0, 从右到左删除n个value
  当n=0, 将所有值为value的元素全删
  ```
  
#### Set类型以及命令操作
* Redis set对外提供的功能与list类似，它也是一个列表的功能。<br>
  特殊之处在于set是可以自动排重的，当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，<br>
  并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。
* Redis的Set是string类型的无序集合。<br>
  它底层其实是一个value为null的hash表,所以添加,删除，查找的复杂度都是O(1)。
  
* 相关命令操作
  ```text
  sadd <key> <value1> <value2> ...
  将一个或多个member元素加入到集合key当中，已经存在于集合的member元素将被忽略。
  e.g. sadd s1 v1 v2 v3 v2 v3 # 输出 (integer) 3
  
  smembers <key> 取出该集合的所有值
  e.g. smembers s1 # 输出 1) "v3" 2) "v2" 3) "v1" 看似有序，其实无序
       sadd s1 v4 v5 v6 v7 
       smembers v4 v5 v6 v7 # (integer) 4
       
  sismember <key> <value> 
  判断集合<key>是否为含有该<value>值，如果有返回1；没有返回0
  e.g. sismember s1 v8 # (integer) 0
       sismember s1 v7 # (integer) 1
       
  scard <key> 返回该集合的元素个数。
  srem <key> <value1> <value2> ... 删除集合中的某个元素
  spop <key> 随意从该集合吐出一个值
  srandmember <key> <n> 
  随机从该集合中取出n个值；不会从集合中删除
  
  sinter <key1> <key2> 返回两个集合的交集元素
  sunion <key1> <key2> 返回两个集合的并集元素
  sdiff  <key1> <key2> 返回两个集合的差集元素(返回在key1集合中，不在key2集合的元素的集合)
  ```
  
#### Hash类型以及命令操作
* Redis hash是一个键值对集合(指的是value是键值对形式)
* Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。
  
  ![image](/assets/images/blog/redis-2.png)

* 类似Java里面的Map<String, Object>
* 相关命令操作
  ```text
  hset <key> <field> <value>
  给<key>集合中的<field>键赋值<value>
  
  hget <key1> <field>
  从<key1>集合<field>取出<value>
  
  hmset <key1> <field1> <value1> <field2> <value2> ...
  批量设置hash的值
  
  hexists <key> <field> 查看哈希表key中，给定域field是否存在
  hkeys <key> 列出该hash集合的所有field
  hvals <key> 列出该hash集合的所有value
  
  hincrby <key> <field> <increment> 
  为哈希表key中的域field的值加上增量increment
  
  hsetnx <key> <value> 
  将哈希表key中的域field的值设为value，当且仅当域field不存在
  ```


#### zset类型以及命令操作
* Redis有序集合zset((sorted set))与普通集合set非常相似，是一个没有重复元素的字符串集合。<br>
  不同之处是有序集合的每个成员都关联了一个评分(score)，<br>
  这个评分被用来按照从最低分到最高分的方式排序集合中的成员。集合的成员是唯一的，但是评分可以重复。
* 因为元素是有序的，所以可以很快根据评分(score)或者次序(position)来获取一个范围的元素。<br>
  访问有序集合的中间元素也是非常快的，因此能够使用有序集合作为一个没有重复元素的智能列表。
* 相关命令操作
  ```text
  zadd <key> <score1> <value1> <score2> <value2> ...
  将一个或多个member元素及其score值加入到有序集key当中
  
  zrange <key> <start> <stop> [WITHSCORES]
  返回有序集key中，下标在<start><stop>之间的元素；
  带WITHSCORES，可以让分数一起和值返回到结果集。
  
  zrangebyscore <key> <min> <max> [WITHSCORES] [limit offset count]
  返回有序集key中，所有score的值介于min和max之间(包括等于min或者max)的成员。
  有序集成员按score的值递增(从小到大)次序排列。
  
  zrevrangebyscore <key> <max> <min> [WITHSCORES] [limit offset count]
  同上，改为从大到小排列。
  
  zincrby <key> <increment> <value> 为元素的score加上增量
  zrem <key> <value> 删除该集合下，指定值的元素
  zcount <key> <min> <max> 统计该集合，分数区间内的元素个数
  zrank <key> <value> 返回该值在集合中的排名，从0开始
  ```


### Redis配置文件
在启动Redis服务器时，我们需要为其指定一个配置文件，缺省情况下配置文件在Redis的源码目录下，文件名为**redis.conf**。

Redis配置文件使用 ################ 被分成了几大块区域。

主要有:
```text
1. 网络(network)
2. 通用(general)
3. 快照(snapshotting)
4. 复制(replication)
5. 安全(security)
6. 限制(limits)
7. 追加模式(append only mode)
8. LUA脚本(lua scripting)
9. REDIS集群(REDIS CLUSTER)
10. 慢日志(slow log)
11. 事件通知(event notification)
12. ADVANCED CONFIG
```

Redis的配置文件主要参数以及作用
```text
bind 127.0.0.1 
指定redis只接受来自于该IP地址(多个IP地址的话，用空格隔开)的请求。如果不进行设置，那么将处理所有的请求。
在生产环境中最好设置该项。 

port 6379
指定redis运行的端口，默认是6379；

tcp-backlog 511
在高并发环境下需要一个高backlog来避免慢客户端连接问题。
当服务器和redis建立连接之后，对于处理不了的请求，会排队等待。这个backlog就是允许排队的队列大小。
注意这个值的最终设定还要参考/proc/sys/net/core/somaxconn.

tcp-keepalive 300
当客户端和服务器建立连接之后，这个时间表示空闲多长时间后关闭连接。
如果指定为0，表示关闭该功能。

daemonize no
是否将Redis作为守护进程(后台运行)运行。注意配置成守护进程后Redis会将进程号写入文件/var/run/redis.pid。

pidfile /var/run/redis_6379.pid
当redis在后台运行的时候，Redis默认会把pid文件放在/var/run/redis.pid.我们可以配置到其他地址。
当运行多个redis服务时，需要指定不同的pid文件和端口。

loglevel notice
指定日志记录级别
Redis支持四个级别:DEBUG(记录很多信息，用于开发和测试), verbose(记录有用的信息，但不像debug会记录那么多), 
                notice(适度的的verbose，常用于生产环境), warning(只有非常重要或者严重的信息会被记录)，
                默认为notice。


logfile
用于指定日志文件名称。空字符串表示标准输出。注意如果用标准输出来作为记录但是又用后台启动redis，
日志会被记录到文件/dev/null.

databases 16
设置数据库的数量。默认使用第0个数据库，基于每个连接可以使用select <dbid>来选择一个不同的数据库。

maxclients 10000
设置最多同时可以连接的客户端数量。

maxmemory <bytes>
指定redis最大内存限制。

maxmemory-policy <方式>
当内存达到最大值的时候Redis会选择删除哪些数据呢，有五种方式可以供选择:
    1. volatile-lru: 利用LRU算法移除设置了过期时间的key
    2. allkeys-lru: 利用LRU算法移除任何key
    3. volatile-lfu: 利用LFU算法移除设置了过期时间的key
    4. allkeys-lfu: 利用LFU算法移除任何key
    5. volatile-random: 移除随机一个设置了过期时间的key
    6. allkeys-random: 移除随机任何一个key
    7. volatile-ttl: 移除即将过期的key(minor TTL)
    8. noeviction: 不移除任何key，返回一个错误当写操作的时候。
注意: 对于上面的策略，如果没有合适key可以移除，写的时候Redis会返回一个错误。
LRU(Least Recently Used), LFU(Least Frequently Used)
```

### Redis事务
Redis事务允许一组命令在单一的步骤中执行。事务有三个特性:
* Redis事务是一个单独的隔离操作:事务中的所有命令都会序列化，按顺序地执行。事务在执行过程中，
不会被其他客户端发送来的命令请求所打断。
* 没有隔离级别的概念: 队列中的命令没有提交之前都不会实际的被执行，因为事务提交前任何指令都不会被实际执行，
也就不存在"事务内的查询要看到事务里的更新，在事务外查询不能看到"这个让人万分头疼的问题。
* 不保证原子性: Redis同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚。

一个事务从开始到执行会经历以下三个阶段:
1. 开始事务
2. 命令入队
3. 执行事务

#### Multi, Exec, discard
* 从输入**Multi**命令开始，输入的命令都会依次进入命令队列中，但不会执行，直到输入**Exec**后，Redis会将之前的命令队列中的命令依次执行。
* 组队的过程中可以通过**discard**来放弃组队。

    ![image](/assets/images/blog/redis-3.png)


#### 事务的错误处理
* 组队中某个命令出现了报告错误，执行时整个的所有队列都会被取消。

    ![image](/assets/images/blog/redis-4.png)
    
    
* 如果执行阶段某个命令报出了错误，则只有报错的命令不会被执行，而其他的命令都会执行，不会回滚。
   
   ![image](/assets/images/blog/redis-5.png)

#### 事务冲突问题
假设有如下请求:
1. 一个请求想给金额减8000
2. 一个请求想给金额减5000
3. 一个请求想给金额减1000

![image](/assets/images/blog/redis-6.png)

对于以上的冲突问题，有两种解决方法:悲观锁(相当于mysql的行级锁，解决了不可重复读，mysql的隔离级别就叫可重复读)和乐观锁

![image](/assets/images/blog/redis-7png)

* 悲观锁(Pessimistic Lock)，顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，
这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是
在做操作之前先上锁。

* 乐观锁(Optimistic Lock), 顾名思义, 就是很乐观， 每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人
有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量。<br>
  **Redis**就是利用这种check-and-set机制实现事务的。

总结: 悲观锁效率低;乐观锁虽然效率高，但是有可能会失败,比方说淘宝双11，肯定会有失败的订单，但是没关系，100个订单失败1，2个没关系，这容错率是允许的在高效率下。

#### WATCH, UNWATCH
WATCH key \[key ...\]
* 在执行multi命令之前，先执行watch key1 \[key2\], 可以监视一个(或多个)key，如果在事务执行之前这个(或这些)key被其他命令所改动，
那么事务将被打断(会返回(nil)，表示被打断，打断的机制就是通过check-and-set)。

UNWATCH
* 取消WATCH命令对所有key的监视
* 如果在执行WATCH命令之后，EXEC命令或DISCARD命令先被执行了的化，那么就不需要再执行UNWATCH了。




