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
- [Redis的安装](#redis的安装)
- [Redis五大数据类型](#redis五大数据类型)
    * [Redis的key的基本命令操作](#redis的key的基本命令操作)
    * [String类型以及命令操作](#string类型以及命令操作)
    * [原子性](#原子性)  
    * [List类型以及命令操作](#list类型以及命令操作)
    * [Set类型以及命令操作](#set类型以及命令操作)
    * [Hash类型以及命令操作](#hash类型以及命令操作)
    * [zset类型以及命令操作](#zset类型以及命令操作)
- [Redis配置文件](#redis配置文件)
- [Redis订阅发布模式](#redis订阅发布模式)
- [Redis事务](#redis事务)
- [Redis数据备份与恢复](#redis数据备份与恢复)
- [Redis持久化](#redis持久化)
    * [RDB](#rdb)
    * [AOF](#aof)
- [Redis主从复制之读写分离](#redis主从复制之读写分离)
    * [主从配置实例](#主从配置实例)
    * [一主二仆模式演示](#一主二仆模式演示)
    * [主从复制原理](#主从复制原理)
    * [薪火相传](#薪火相传)
- [Redis主从复制之哨兵机制](#redis主从复制之哨兵机制)
    * [哨兵监听主机](#哨兵监听主机)
    * [配置哨兵实例](#配置哨兵实例)
    * [Redis主从复制之故障恢复](#redis主从复制之故障恢复)
- [Redis集群](#redis集群)
    * [问题](#问题)
    * [什么是集群](#什么是集群)
    * [安装ruby环境](#安装ruby环境)
    * [主从配置实例](#主从配置实例)
    * [什么是slots](#什么是slots)
    * [在集群中录入值](#在集群中录入值)
    * [查询集群中的值](#查询集群中的值)
    * [故障恢复](#故障恢复)
    * [Redis集群的优缺点](#redis集群的优缺点)
    
    
    
### redis的安装
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


### redis五大数据类型
#### redis的key的基本命令操作
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

#### string类型以及命令操作
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

#### list类型以及命令操作
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
  
#### hash类型以及命令操作
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


### redis配置文件
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

### redis订阅发布模式
Redis发布订阅(pub/sub)是一种消息通信模式:发送者(pub)发送消息，订阅者(sub)接收消息。
Redis客户端可以订阅任意数量的频道。下图展示了频道channel1，以及订阅这个频道的三个客户端 ---
client2,client5,client1之间的关系:

![image](/assets/images/blog/redis-8.png)

当有新消息通过PUBLISH命令发送给频道channel1时，这个消息就会被发送给订阅它的三个客户端:

![image](/assets/images/blog/redis-9.png)

```text
订阅给指定频道的信息: SUBSCRIBE channel [channel...]
将信息message发送到指定的频道channel: PUBLISH channel message
```
应用案例:
```text
Jiaqis-MacBook-Pro:jxu033.github.io jiaqi$ redis-cli
127.0.0.1:6379> SUBSCRIBE class:20191020
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "class:20191020"
3) (integer) 1
1) "message"
2) "class:20191020"
3) "I love u"
1) "message"
2) "class:20191020"
3) "I love her"


Jiaqis-MacBook-Pro:jxu033.github.io jiaqi$ redis-cli
127.0.0.1:6379> PUBLISH class:20191020 "I love u"
(integer) 1
127.0.0.1:6379> PUBLISH class:20191020 "I love her"
(integer) 1
127.0.0.1:6379> 
```

### redis事务
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

![image](/assets/images/blog/redis-7.png)

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

### redis数据备份与恢复
Redis SAVE命令用于创建当前数据库的备份。
```text
127.0.0.1:6379> save
OK
```
该命令将在redis安装目录(我感觉是启动redis-server的目录)中创建dump.rdb文件。

如果需要恢复数据,只需将备份文件(dump.rdb)移动到redis安装目录并启动服务即可。获取redis目录可以使用CONFIG命令:
```text
127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "/Users/jiaqi/personal_projects/jxu033.github.io"
```
以上命令CONFIG GET dir输出的redis安装目录为/Users/jiaqi/personal_projects/jxu033.github.io

创建redis备份文件也可以使用命令BGSAVE，该命令在后台执行。
```text
127.0.0.1:6379> bgsave
Background saving started
```
注意: 对于dump.rdb的存储的路径问题，可以通过修改redis.conf配置文件的snapshotting模块的dir变量来指定。默认为./，即运行redis-server的当前目录。

### Redis持久化
Redis提供了2个不同形式的持久化方式: RDB(Redis Database)和AOF(Append Of File)
持久化的数据的作用: 用于重启后的数据恢复。

#### RDB
在指定的时间间隔内将内存中的数据集**快照**写入磁盘，也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里。

**备份是如何执行的(对于bgsave来说)**:<br>
Redis会单独创建(fork)一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。
整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比
AOF方式更加的高效。RDB的缺点就是最后一次持久化之后的数据可能会丢失。

**关于fork**:<br>
在Linux程序中，fork()会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，Linux中引入了
**"写时复制技术"**（内核只为新生成的子进程创建虚拟空间结构，它们复制于父进程的虚拟空间结构，但是不为这些段分配物理内存，它们共享父进程的物理空间，当父子进程中有更改相应的段的行为发生时，再为子进程相应的段分配物理空间。）
，一般情况父进程和子进程会共用同一段物理内存，只有进程空间的各段内容要发生变化时，才会将父进程的内容复制一份给子进程。


**rdb的保存的文件**:<br>
在redis.conf中配置文件名称，默认为dump.rdb。
```text
# The filename where to dump the DB
dbfilename dump.rdb
```
rdb文件的保存路径，也可以修改。默认为redis启动时命令行所在的目录下。
```text
# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir ./
```

**rdb的保存策略**:<br>
```text
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""

save 900 1
save 300 10
save 60 10000
```

**手动保存快照**: **save**<br>
* 命令save: 只管保存，其他不管，全部阻塞(会阻塞客户端的写操作)
* save vs bgsave(会使用前面所说的备份原理，调用fork机制)

注意: Redis的RDB文件不会坏掉，因为其写操作是在一个新的进程中进行的。当生成一个新的RDB文件时，Redis生成的
子进程会先将数据写到一个临时文件中，然后通过原子性rename系统调用将临时文件重命名为RDB文件。
这样在任何时候出现故障，Redis的EDB文件都总是可用的。

以下是一些关于RDB持久化的其他配置:
```text
stop-writes-on-bgsave-error yes
当Redis无法写入磁盘的化，直接关掉Redis的写操作

rdbcompression yes
进行rdb保存时，将文件压缩

rdbchecksum yes
在存储快照后，还可以让redis使用CRC64算法来进行数据校验，但是这样会增大大约10%的性能消耗，如果希望
获取到最大的性能提升，可以关闭此功能
```

#### RDB的优缺点
优点:
* 节省磁盘空间
* 恢复速度快

缺点:
* 虽然redis在fork时使用了写时拷贝技术，但是如果数据庞大时还时比较消耗性能。
* 在备份周期在一定间隔时间做一次备份，所以如果redis意外宕掉的话，就会丢失最后一次快照
之后的所有修改。

#### AOF
**以日志的形式来记录每个写操作**(即不记录get)，将Redis执行过的所有写指令记录下来(读操作不记录)，
只许追加文件但不可以改写文件，Redis启动之初会去读取该文件重新构建数据，换言之，Redis重启的话就根据日志文件
的内容将写指令从前到后执行一次以完成数据的恢复工作。

* AOF默认不开启，需要手动在配置文件中配置(APPEND ONLY MODE)。

```text
############################## APPEND ONLY MODE ###############################

# By default Redis asynchronously dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage may result into a few minutes of writes lost (depending on
# the configured save points).
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check http://redis.io/topics/persistence for more information.

appendonly no
```

* 可以在redis.conf配置文件名称，默认为appendonly.aof

```text
# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof" 
# AOF文件名如上，其存取的路径和rdb共用一个路径，即dir指定的路径。
```

* AOF文件的保存路径，同RDB的路径一致，均由dir指定。
* AOF的保存策略

```text
# The fsync() call tells the Operating System to actually write data on disk
# instead of waiting for more data in the output buffer. Some OS will really flush
# data on disk, some other OS will just try to do it ASAP.
#
# Redis supports three different modes:
#
# no: don't fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log. Slow, Safest.
# everysec: fsync only one time every second. Compromise.
#
# The default is "everysec", as that's usually the right compromise between
# speed and data safety. It's up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with the idea of
# some data loss consider the default persistence mode that's snapshotting),
# or on the contrary, use "always" that's very slow but a bit safer than
# everysec.
#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".

# appendfsync always # 只要发送到server就一定往磁盘上存(即放到文件里)，效率低
appendfsync everysec # 秒对于我们来说和快，计算机来说其实还算长的，所以效率和存储各方面还是不错的
# appendfsync no # Redis不管了，操作系统自己决定什么时候往文件里放(即存到磁盘上)
```

* 如果AOF和RDB同时开启(两个文件都有appendonly.aof和dump.rdb)，听谁的

```text
听AOF的(读appendonly.aof,即AOF器作用)。这和保存周期有关系，因为RDB有时间间隔，在时间间隔当中如果出现问题，则数据就丢失了。
AOF也有自己的保存策略，相对于RDB来说，AOF保存的数据会多一些，所以恢复的时候用数据多的来恢复肯定会好一些。有人会说，那我将RDB保存策略时间设置短一些，但是
这样会影响效率。所以总的来说，当两者同时开启的时候，redis会使用AOF来进行数据恢复(或者启动)。
```

* AOF会丢失数据吗

![image](/assets/images/blog/redis-10.png)

```text
当然也会丢失数据。
appendfsync everysec 是指每条会去进行同步，但是数据的传递分好多层次，如上图：
1. 假设现在数据传到Server APP，准备往OS缓存传，但是还没传的时候，这时候server app服务器宕掉了，数据就会丢失
2. 假设现在数据传到了OS缓存，这时候server APP服务器宕掉了，这时候数据就不会丢失，因为现在已经和APP层没关系了，已经传到了OS层，操作系统级。
3. 假设现在数据传到了OS缓存，但是断电了，不仅是现在传到OS缓存的数据没了，之前的数据在缓存的数据也全没了。
```

* AOF文件故障备份
```text
AOF的备份机制和性能虽然和RDB不同，但是备份和恢复的操作同RDB一样，都是拷贝备份文件，需要恢复时再拷贝到Redis工作目录下，启动系统即加载。
AOF和RDB同时开启，系统默认取AOF的数据。
```

* AOF文件故障恢复
```text
如果遇到AOF文件损毁，可通过redis-check-aof --fix appendonly.aof进行恢复。
AOF文件是有格式的，如果你不按格式乱改该文件，然后启动redis-server，会报错，并且提示你使用./redis-check-aof --fix <filename>去检测修复，他会把不符合格式要求的数据清理掉。
其实AOF文件是不能随便手动修改的，就算你按格式改了，他也会通过比对dump.rdb文件来去除掉你直接加入的数据。
```

* Rewrite
```text
AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制，当AOF文件的大小超过所设定的阀值时，Redis
就是启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集。可以使用命令bgrewriteaof。
```

* Redis如何实现重写
```text
AOF文件持续增长而过大时，会fork初一条新进程来将文件重写(也是先写临时文件最后再rename)，遍历新进程的内存中数据，
每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方法重写
了一个新的aof文件，这点和快照有点类似。
```

* 何时重写
```text
重写虽然可以节约大量磁盘空间，减少恢复时间。但是每次重写还是有一定的负担的，因此设定redis要满足一定条件才会进行重写。
以下两个参数决定了何时进行重写:
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
系统载入时或者上次重写完毕时，Redis会记录当时的AOF大小，设为base_size，如果redis的
AOF当前大小>=base_size + base_size * 100% 并且当前大小>=64mb(默认)的情况 下，Redis会对AOF文件进行重写。
```

#### AOF的优缺点
优点:
* 备份机制更稳健，丢失数据概率更低。
* 可读的日志文件，通过操作AOF文件，可以处理错误操作。

缺点:
* 比起RDB，占用更多的磁盘空间
* 恢复备份速度要慢(因为数据多，而且相当于从头到尾执行一边aof文件中的命令)
* 每次读写都同步的话，有一定的性能压力
* 存在个别Bug，造成数据不能恢复


#### 用RDB还是用AOF
* 官方推荐两个都启动
* 如果对数据不敏感，可以单独选用RDB
* 不建议单独使用AOF，因为可能会出现Bug
* 如果只是做纯内存缓存，可以都不用(因为只是缓存，如果失效的话，直接从数据库读就可以了)


### Redis主从复制之读写分离
* 主从复制，就是主机数据更新后根据配置和策略，自动同步到备机的master/slaver机制，
**Master以写为主，Slave以读为主**

![image](/assets/images/blog/redis-11.png)

* 用处: 读写分离，性能扩展; 容灾快速恢复
* 配从(服务器)不配主(服务器)
    ```text
    拷贝多个redis.conf文件include
    开启daemonize yes
    pid文件名字pidfile
    指定端口port
    Log文件名字
    dump.rdb名字dbfilename
    appendonly 关掉或者换名字
    ```

#### 主从配置实例
首先将redis.conf中的daemonize变量设置为yes，将appendonly变量设置为no
1. 文件redis6379.conf
```text
include /opt/module/redis/conf/redis.conf
pidfile /var/run/redis_6379.pid
port 6379
dbfilename dump6379.rdb
```

2. 文件redis6380.conf
```text
include /opt/module/redis/conf/redis.conf
pidfile /var/run/redis_6380.pid
port 6380
dbfilename dump6380.rdb
```

3. 文件redis6381.conf
```text
include /opt/module/redis/conf/redis.conf
pidfile /var/run/redis_6381.pid
port 6381
dbfilename dump6381.rdb
```
配置完之后，启动三个服务器slave<br>
```text
redis-server conf/ms/redis6379.conf
redis-server conf/ms/redis6380.conf
redis-server conf/ms/redis6381.conf
```

客户端连接服务器<br>
```text
redis-cli # 默认连接到6379
redis-cli -p 6380 # 连接到6380
redis-cli -p 6381 # 连接到6381
```

通过输入命令**info replication**可以打印主从复制的相关信息

现在想让6379服务器作为master，其他两个作为slave，可以通过以下命令设置主从关系:
```text
#slaveof <ip> <port> # 成为某个实例的从服务器
slaveof localhost 6379
```
此时再打印主从复制关系可以看到主从服务器已经设置完毕。

#### 一主二仆模式演示
1. 切入点问题。 slave1，slave2是从头开始复制还是从切入点开始复制？比如说从k4进来(成为从机)，那之前的k1,k2,k3是否也可以复制
    ```text
    可以。
    ```
2. 从机是否可以写？set可否？
    ```text
    不可以。只读的。
    ```
3. 主机shutdown后情况如何？从机是上位还是原地待命
    ```text
    原地待命。
    ```
4. 主机又回来了之后，主机新增记录，从机还能否顺利复制？
   ```text
   当然可以。因为关系还在呢。
   ```
5. 其中一台从机down后情况如何？依照原有它能跟上大部队吗？
   ```text
   如果从机down掉之后，再启动，发现它已经跟不上大部队了，它自己的role变成了master。不再是down之前的从机了。
   为了使其down 掉之后再启动依然还是从机(跟上大部队)，可以在其配置文件中加上命令slaveof <主机IP> <port>，
   这样当其启动的时候，会成为从机(又建立关系)，就又会把主机的数据复制过来(可能会出现卡顿，因为在从master读取数据并加载)。
   ```

#### 主从复制原理
* 每次从机联通以后，都会给主机发送sync指令
* 主机立刻进行存盘操作，发送RDB文件给从机
* 从机收到rdb文件后，进行全盘加载
* 之后每次主机的写操作，都会立刻发送给从机，从机执行相同的命令

![image](/assets/images/blog/redis-12.png)


#### 薪火相传
* 上一个slave可以是下一个slave的Master，slave同样可以接收其他slaves的连接和同步请求，
那么该slave(该slave虽然作为链条的下一个master，但他的角色还是slave，所以还是不能进行写操作)作为链条中下一个的master，可以有效减轻master的写压力，去中心化降低风险。
* 用slaveof <ip> <port> 
* 中途变更转向(从最原先master的从机，变为其中一个slave的从机): 会清除之前的数据，重新建立关系拷贝最新的数据
* 风险是一旦某个slave宕机，其后面的slave都无法备份。

![image](/assets/images/blog/redis-13.png)

#### 反客为主
* 当一个master宕机后，后面的slave可以立刻升为master，其后面的slave不用做任何修改。
* 使用命令**slaveof no one**将从机变为主机。
```text
假设6379是6380的master，6380是6381的master，此时如果6379宕机了(shuntdown)，
因为6380的角色还是从机，所以不能写操作；但是可以通过slaveof no one将其变为主机；
此时通过info replication可以查看其角色已经从slave变为master，并且可以用set进行写操作。
```
但是此时有一个问题，如果发生以上问题，能否自动化完成以上转换，这时我们引入哨兵模式。


### Redis主从复制之哨兵机制
* 反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库(从机)变成主库(主机)。

![image](/assets/images/blog/redis-14.png)

#### 哨兵监听主机
哨兵(sentinel)通过"心跳"来监听主机，即时不时(比如说每隔30s或3mins)的ping一下主机，如果发现放完ping之后，主机没有
响应(即pong回来)，这个时候哨兵可能会觉得主机挂掉了。但是还不能确定肯定是挂掉了，得再发几个ping，因为有些情况下，如果网络
慢(网络慢的原因可能是由于大量的写操作占用了网络)也可能导致超时，从而没有返回响应。

所以一般情况下，每个从机上都会有一个哨兵，这就意味着当第一个哨兵去ping主机的时候ping不上(即没有返回响应)，他不确定主机是否肯定挂掉了，这时他会让别的
一些哨兵也去ping一下主机，比说别的哨兵ping了发现也没响应，就会认为确实挂了。哨兵的判断机制决定怎么样才认为主机真的挂掉了，比如说过可以设置一个哨兵认为你挂掉就算挂掉了，还是多个哨兵认为你挂掉了
才算挂掉了，可以通过设置redis.conf来配置。

如果一旦确定主机确实挂掉了，他就会从从机当中根据一些网络，优先级等机制来选择从机，使其成为主机。

#### 配置哨兵实例
* 调整为一主二仆模式

```text
启动3个redis服务器:
redis-server conf/ms/redis6379.conf
redis-server conf/ms/redis6380.conf
redis-server conf/ms/redis6381.conf

启动client分别连接三个服务器:
redis-cli -p 6379 # 连接后可用info replication查看信息
redis-cli -p 6380 # 连接后可用info replication查看信息
redis-cli -p 6381 # 连接后可用info replication查看信息

通过命令slaveof配置6379服务器为master，6380和6381服务器为master的2个slave
配置完成之后，通过info replication查看一主二仆是否正确配置。
```

* 自定义的/myredis目录下新建sentinel.conf文件

* 在sentinel.conf配置文件中填写内容
```text
sentinel monitor mymaster 127.0.0.1 6379 1
# 其中mymaster表示哨兵的名称
# 127.0.0.1 master的IP地址
# port master服务器的端口号
# 数字1表示至少有多少个哨兵同意迁移的数量(有几个哨兵认为主机挂了就真的挂了); 这里1就是说如果有一个哨兵认为你服务器挂掉了，就会进行主机切换。
```

* 启动哨兵命令
```text
redis-sentinel <哨兵配置文件路径>
# 启动后你可以看到服务器日志信息在控制台上提示:
+monitor master mymaster 127.0.0.1 6379 quorum 1
+slave slave 127.0.0.1:6380 @ mymaster 127.0.0.1 6379
+slave slave 127.0.0.1:6381 @ mymaster 127.0.0.1 6379
```

* shutdown主机6379

```text
现在我们shutdown了master服务器，过了一段时间后，我们可以在控制台上看到新的信息产生:
新产生的信息包括判断6379主机是否宕掉的信息，故障转移信息，切换主机信息等。
其说明当原先6379主机shutdown之后，通过哨兵机制，6381服务器被选举出来成为主机，6380变成6381的slave
```

* 将宕掉的6379服务器重新启动

```text
在6379启动后，不会马上变化，其启动后一开始还是master(哨兵还没来得及监控到他)，但是因为之前三台服务器之间彼此建立了关系(包括主从，哨兵等)，过了一会儿，哨兵
发现其活过来了，就会将其转换为6381的从机，这也就是我们会在控制台上看到下面信息的原因。
+convert-to-slave slave 127.0.0.1:6379 127.0.0.1 6379 @ mymaster 127.0.0.1 6381

所有过程都可以通过info replication命令查看变化之前和之后的信息，能帮助理解。
```

#### Redis主从复制之故障恢复

故障恢复换句话说就是当主机宕掉(出现了故障)之后，根据什么机制来迁移主机(将选举从机并将其转换为主机)。

![image](/assets/images/blog/redis-15.png)

1. 选择优先级靠前的, 通过以下配置实现:
```text
# The replica priority is an integer number published by Redis in the INFO output.
# It is used by Redis Sentinel in order to select a replica to promote into a
# master if the master is no longer working correctly.
#
# A replica with a low priority number is considered better for promotion, so
# for instance if there are three replicas with priority 10, 100, 25 Sentinel will
# pick the one with priority 10, that is the lowest.
#
# However a special priority of 0 marks the replica as not able to perform the
# role of master, so a replica with priority of 0 will never be selected by
# Redis Sentinel for promotion.
#
# By default the priority is 100.
replica-priority 100 # 数字越小优先级越高; 当设置为0的时候，则永远不会将这台服务器当主机(这样我们可以通过设置0排除一些不好的服务器，使其永远当不了主机)。
```

2. 选择偏移量最大的
```text
如果优先级是一样的情况下，则选择偏移量最大的。
偏移量指获得原主数据最多的。比如master现在同时向2个主机传数据，但是巧了，master给完第一个从机后挂了，则意味着另外一个从机
没有这个数据，相比这个从机，之前收到的数据的那个从机所获的数据多(即偏移量较大)。
```

3. 选择runid最小的从服务器
```text
和后面要说的集群有管，每个redis启动后都会随机生成一个40为的runid
```

### Redis集群

#### 问题
1. 容量不够，redis如何进行扩容<br>
使用集群的目的是可以扩容，因为redis容量不够，redis是基于内存的，内存不可能无限大。所以
可以使用集群来进行扩容，那么你机器越多，内存越多，则存的数据也就越多。

2. 并发写操作，redis如何分摊
前面的主从复制解决不了这个问题，因为写操作只能在master上进行，分摊不了压力。但是集群可以解决这个问题。


#### 什么是集群
* Redis集群实现了对Redis的水平扩容，即启动N个redis节点，将这个数据库分布存储在这N个节点中，
每个节点存储总数据的1/N。
* Redis集群通过分区(partition)来提供一定程度的可用性(availabilty):即使集群中有一部分节点失效
或者无法进行通讯，集群也可以继续处理命令请求。

#### 安装ruby环境
在配置集群之前需要安装ruby环境:
1. 安装ruby环境
```text
执行yum install ruby
执行yum install rubygems
```

2. 拷贝redis-3.2.0.gem到/opt目录下
3. 执行在opt目录下执行gem install --local redis-3.2.0.gem

#### 主从配置实例
你们公司的redis架构是什么样子的
```text
如果我们是读写分离，则至少是3台
如果redis是集群的话，则至少是6台。因为集群至少得有3个master，则每个master一个slave则总共至少6台。
```

先介绍一些有关安装redis cluster的配置修改
```text
cluster-enable yes # 默认是不打开的，得该成yes打开集群模式
cluster-config-file nodes-6379.conf # 设定节点配置文件名
cluster-node-timeout 15000 # 设定节点失联时间，超过该时间(毫秒), 集群自动进行主从切换
```


现在我们进行集群实例制作，将会制作6个实例，6379，6380，6381，6389，6390，6391，步骤如下:
1. 创建一个cluster文件夹，在其目录下创建以下文件并加入内容如下:
    1) 文件redis6379.conf
      ```text
      include /opt/module/redis/conf/redis.conf
      pidfile /var/run/redis_6379.pid
      port 6379
      dbfilename dump6379.rdb
      cluster-enable yes
      cluster-config-file nodes-6379.conf
      cluster-node-timeout 15000
      ```
    2) 文件redis6380.conf
      ```text
      include /opt/module/redis/conf/redis.conf
      pidfile /var/run/redis_6380.pid
      port 6380
      dbfilename dump6380.rdb
      cluster-enable yes
      cluster-config-file nodes-6380.conf
      cluster-node-timeout 15000
      ```   

    3) 文件redis6381.conf
      ```text
      include /opt/module/redis/conf/redis.conf
      pidfile /var/run/redis_6381.pid
      port 6381
      dbfilename dump6381.rdb
      cluster-enable yes
      cluster-config-file nodes-6381.conf
      cluster-node-timeout 15000
      ```   


    4) 文件redis6389.conf
      ```text
      include /opt/module/redis/conf/redis.conf
      pidfile /var/run/redis_6389.pid
      port 6389
      dbfilename dump6389.rdb
      cluster-enable yes
      cluster-config-file nodes-6389.conf
      cluster-node-timeout 15000
      ```   


    5) 文件redis6390.conf
      ```text
      include /opt/module/redis/conf/redis.conf
      pidfile /var/run/redis_6390.pid
      port 6390
      dbfilename dump6390.rdb
      cluster-enable yes
      cluster-config-file nodes-6390.conf
      cluster-node-timeout 15000
      ```  
      
    6) 文件redis6391.conf
      ```text
      include /opt/module/redis/conf/redis.conf
      pidfile /var/run/redis_6391.pid
      port 6391
      dbfilename dump6391.rdb
      cluster-enable yes
      cluster-config-file nodes-6391.conf
      cluster-node-timeout 15000
      ```        
      

2. 根据上面的配置文件，分别启动6台redis服务器
```text
redis-server conf/cluster/redis6379.conf
redis-server conf/cluster/redis6380.conf
redis-server conf/cluster/redis6381.conf
redis-server conf/cluster/redis6389.conf
redis-server conf/cluster/redis6390.conf
redis-server conf/cluster/redis6391.conf
# 启动后通过使用命令ps -ef | grep redis 可以发现每条信息后面都有[cluster]。
```

3. 将六个节点合成一个集群
    * 组合之前，请确保所有redis实例启动后，nodes-xxxx.conf文件都正常生成。
    * 合体(把每个redis启动好之后，通过该指令将其合体在一块形成一个集群):<br>
        ```text
        cd /opt/redis-3.2.5/src
        ./redis-trib.rb create --replicas 1
        192.168.31.211:6379 192.168.31.211:6380
        192.168.31.211:6381 192.168.31.211:6389
        192.168.31.211:6390 192.168.31.211:6391
        注意: 此处不要用127.0.0.1，请使用真实IP地址
        还有前面配置的ruby环境就是为了执行./redis-trib.rb文件
        create: 表示创建集群的意思
        参数replicas就是所谓的副本, 其值为1，则当我们给定6台的时候，其为了满足条件会自动
        产生3台master，3台slave，这样刚好满足设置的副本值。
        ```
        
4. 运行上面命令后，在标准输出上会出现以下配置信息<br>
    ![image](/assets/images/blog/redis-16.png)
    ```text
    M: 表示master，有三台6379，6380，6381
    S: 表示slave，有三台6389，6390，6391
    每个master后面都有为其分配的插槽信息
    每个slave的后面包含replicates <40位runid>，通过这个runid可以看出他是哪个master的备份
    这里6379，6380，6381分别是6389，6390，6391的备份
    最后他会问你是否接收上面的配置，接受这个配置之后，可以去dir指定的路径下看到产生的节点配置文件(nodes-xxx.conf)
    ```    

5. 接受上面的配置之后，会出现以下信息
    ![image](/assets/images/blog/redis-17.png)
    ```text
    所有16384个插槽被cover了，16384 = 16 * 1024
    现在集群已经配置完毕，我们可以尝试用客户端进行连接
    ```
    
6. 用集群的方式连接客户端
    * 首先我们尝试用以前的方式连接其中一个master服务器，然后进行写操作   
        ![image](/assets/images/blog/redis-18.png)
        ```text
        可以看到写操作发生错误
        使用命令CLUSTER nodes可以查看集群信息
        根据信息，可以知道当前6379服务器包含0-5460，而k1对应的插槽为12706，不在6379上，而是在6381上。
        但是因为我们当前采用的连接方式不是集群方式，导致他不会把所有服务器当成一个整体，从而不会将k1重定向到
        服务器6381上的12706插槽。
        ```
    * 所以我们应该使用集群的方式连接
        ![image](/assets/images/blog/redis-19.png)      
        ```text
        这时集群会自动将键k1转向过去，因为他认为所有服务器是个整体(因为使用集群方式连接)，redis集群会自动将其转向到
        能处理当前key的机器上去。
        ```

redis cluster如何分配这六个节点<br>
* 一个集群至少要有三个主节点(这是因为故障检测和故障转移过程都要求大多数主节点达成协议。如果我们只有2个主设备而一个设备发生故障，则另外一个主节点无法根据协议作出决定)
* 选项 --replicas 1 表示我们希望为集群中的每个主节点创建一个从节点
* 分配原则尽量保证每个主数据库运行在不同的IP地址，每个从库和主库不在一个IP地址上(我们的实例为了演示方便，都运行在一台机器上)。

#### 什么是slots
* 一个Redis集群包含16384个插槽(hash slot)，(换句话说，把整个redis内存分成16384块插槽,每个插槽就是用来放每个key-value的)，数据库中的每个键都属于这16384个插槽的其中一个，
集群使用公式**CRC16(key)%16384**来计算键key属于哪个槽，其中CRC16(key)语句用于计算键key的CRC16校验和。
* 集群中的每个节点负责处理一部分插槽。举个例子，如果一个集群可以有主节点，其中:
    ```text
    节点A负责处理0号至5500号插槽。
    节点B负责处理5501号至11000号插槽。
    节点C负责处理11001号至16383号插槽
    ```


#### 在集群中录入值
* 在redis-cli每次录入,查询键值，redis都会计算该key应该被送往的插槽,如果不是该客户端
对应服务器的插槽，redis会报错，并告知应前往的redis实例地址和端口
* redis-cli客户端提供了-c参数实现自动重定向。<br>
  （如redis-cli -c -p 6379登入后，再录入，查询键值对可以自动重定向）
* 不在一个slot下的键值，是不能使用mget,mset等多键操作。
* 可以通过{}来定义组的概念(可以认为是别名)，从而使key中{}内相同内容的键值对放到一个slot中去。
   ![image](/assets/images/blog/redis-20.png)      
    
#### 查询集群中的值
```text
CLUSTER KEYSLOT <key> 计算键key应该被放置在哪个槽上
CLUSTER COUNTKEYSLOT <slot> 返回槽slot目前包含的键值对数量
CLUSTER GETKEYSINSLOT <slot> <count> 返回count个slot槽中的键
```

#### 故障恢复
* 如果主节点下线，从节点能否自动升为主节点？<br>
    集群内部有机制进行处理，当主机宕掉之后，其从机会成为主机。
* 主节点恢复后，主从关系会如何?<br>
    以前的主节点，会成为成为主节点的从节点的从节点。 
   
   ![image](/assets/images/blog/redis-21.png)
   
   上图是主节点下线到重新启动后这一过程所对应的集群信息变化。
   
* 如果所有某一段插槽的主从节点都宕掉，redis服务是否还能继续?<br>
    看你的实际情况，来设定cluster-require-full-coverage参数从而决定是否继续当出现此情况。
* 可以通过设置redis.conf中的参数cluster-require-full-coverage来决定
    ```text
    # By default Redis Cluster nodes stop accepting queries if they detect there
    # is at least an hash slot uncovered (no available node is serving it).
    # This way if the cluster is partially down (for example a range of hash slots
    # are no longer covered) all the cluster becomes, eventually, unavailable.
    # It automatically returns available as soon as all the slots are covered again.
    #
    # However sometimes you want the subset of the cluster which is working,
    # to continue to accept queries for the part of the key space that is still
    # covered. In order to do so, just set the cluster-require-full-coverage
    # option to no.
    #
    # cluster-require-full-coverage yes
    ```

#### Redis集群的优缺点
Redis集群提供了以下好处:
* 实现扩容
* 分摊压力
* 无中心配置相对简单(每个节点的下面都有集群的配置，即nodes-xxx.conf文件，从而知道插槽在哪个位置，从而进行正确的重定向，即任何一个都是中心，就是无中心)

Redis集群的不足:
* 多键操作是不被支持的
* 多键的Redis事务是不被支持的。lua脚本不被支持
* 由于集群方案出现较晚，很多公司已经采用了其他的集群方案，而代理或者客户端分片的方案想要迁移至redis cluster，
需要整体迁移而不是逐步过渡，复杂度较大。
