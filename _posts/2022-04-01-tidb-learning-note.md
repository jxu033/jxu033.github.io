---
title: "Tidb learning note"
layout: post
date: 2022-04-01 22:45
tag:
- flink
- window
- api
star: start
category: blog
author: jiaqixu
---

### 目录
* TOC
{:toc}

## 介绍

TiDB是一款定位于在线事务处理/在线分析处理(HTAP： Hybrid Transactional/Analytical Processing)的融合型数据库产品，实现了一键水平伸缩，强一致性的多副本数据安全，分布式事务，**实时**OLAP等重要特性。同时兼容Mysql协议和生态，迁移便捷，运维成本极低。
设计目标100% OLTP and 80% OLAP

OLTP: 
强调支持短时间内大量并发的事物操作(CRUD)能力，每个操作设计的数据量都很小(比如几十到几百字节)
强调事物的强一致性(银行的转账交易)

OLAP:
偏向复杂的只读查询，读取海量数据进行分析计算，查询时间往往很长
e.g. 双十一结束，淘宝运行人员对订单进行分析挖掘，找出一些市场规律等等。

这种分析可能需要读取所有的历史订单进行计算，耗时几十秒甚至几十分钟都有可能。


## TiDB架构特性
### TiDB整体架构

TiDB集群主要包括三个核心组件：TiDB Server， PD Server和TiKV Server.
此外，还有用于解决用户复杂OLAP需求的TiSpark组件和简化云上部署管理的TiDB Operator组件。

#### 架构图解
![image.png](https://cdn.nlark.com/yuque/0/2022/png/694242/1649949412992-84baf6e0-f3bf-4e1c-8cf6-cc24add644c2.png?x-oss-process=image/format,png#clientId=u532d4910-b2cf-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=413&id=u0a7629a5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=826&originWidth=1403&originalType=binary&ratio=1&rotation=0&showTitle=false&size=3481144&status=done&style=none&taskId=u87bf4e5c-2f56-4aa2-9e71-ae876bd4c83&title=&width=701.5)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/694242/1649949420861-e8201288-e95a-40e9-926c-5290158dd14b.png?x-oss-process=image/format,png#clientId=u532d4910-b2cf-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=307&id=uf379b5a1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=448&originWidth=1023&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1379294&status=done&style=none&taskId=u62a3fd03-92a2-4d95-b03d-7e740958128&title=&width=701.5)

### 核心特性

1. 高度兼容MySQL

大多数情况下，无需修改代码即可从MySQL轻松迁移至TiDB，分库分表后的MySQL集群亦可通过TiDB工具进行实时迁移。
对于用户使用的时候，可以透明地从MySQL切换到TiDB中，只是"新MySQL"的后端是存储"无限的"，不再受制于Local的磁盘容量。在运维使用时也可以将TiDB当作一个从库挂到MySQL主从架构中。

2. 分布式事务

TiDB 100% 支持标准的ACID事务。

3. 一站式HTAP解决方案

HTAP： Hybrid Transactional/Analytical Processing
TiDB作为典型的OLTP行存储数据库，同时兼具强大的OLAP性能，配合TiSpark，可提供一站式HTAP解决方案，一份存储同时处理OLTP & OLAP, 无需传统繁琐的ETL过程。

4. 云原生SQL数据库

TiDB是为云而设计的数据库，支持公有云、私有云和混合云，配合TiDB Operator项目可实现自动化运维，使部署、配置和维护变得十分简单。

5. 水平弹性扩展

通过简单地增加新节点即可实现TiDB的水平扩展，按需扩展吞吐或存储，轻松应对高并发、海量数据场景。

6. 真正金融级高可用

相比于传统主从(M-S)复制方案，基于Raft(数据一致性协议)的多数派选举协议可以提供金融级的100%数据强一致性保证，且在不丢失大多数副本的前期下，可以实现故障的自动恢复(auto-failover)，无需人工介入。

#### 水平扩展
无限水平扩展是TiDB的一大特点，这里说的水平扩展包括两方面：计算能力(TiDB)和存储能力(TiKV)。

**TiDB Server** 负责处理SQL请求，随着业务的增长，可以简单的添加TiDB Server节点，提高整体的处理能力，提供更高的吞吐。
**TiKV** 负责存储数据，随着数据量的增长，可以部署更多的TiKV Server 节点解决数据Scale的问题。
**PD** 会在TiKV节点之间以Region为单位做调度，将部分数据迁移到新加的节点上。

所以在业务的早期，可以只部署少量的服务实例(推荐至少部署3个TiKV，3个PD，2个TiDB)，随着业务量的增长，按照需求添加TiKV或者TiDB实例。

#### 高可用
高可用是TiDB的另一大特点，TiDB/TiKV/PD 这三个组件都能容忍部分实例失效，不影响整个集群的可用性。下面分别说明这三个组件的可用性、单个实例失效后的后果以及如何恢复。

- TiDB

TiDB是无状态的，推荐至少部署两个实例，前端通过负载均衡组件对外提供服务。当单个实例失效时，会影响正在这个实例上进行的Session，从应用的角度看，会出现单次请求失败的情况，重新连接后即可继续获得服务。单个实例失效后，可以重启这个实例或者部署一个新的实例。

- PD

PD是一个集群，通过Raft协议保持数据的一致性，单个实例失效时，如果这个实例不是Raft的leader，那么服务完全不受影响；如果这个实例时Raft的leader，会重新选出新的Raft leader， 自动恢复服务。PD在选举的过程中无法对外提供服务，这个时间大约是3秒钟。推荐至少部署三个PD实例，单个实例失效后，重启这个实例或者添加新的实例。

- TiKV

TiKV是一个集群，通过Raft协议保持数据的一致性(副本数量可配置，默认保存三副本)，并通过PD做负载均衡调度。单个节点失效时，会影响这个节点上存储的所有Region。对于Region中的Leader节点，会中断服务，等待重新选举；对于Region中的Follower节点，不会影响服务。当某个TiKV节点失效，并且在一段时间内(默认30分钟)无法恢复，PD会将其上的数据迁移到其他的TiKV节点上。

### TiDB存储和计算能力

#### 存储能力-TiKV-LSM
TiKV Server通常是3+的，TiDB每份数据缺省为3副本，这一点与HDFS有些相似，但是通过Raft协议进行数据复制，TiKV Server上的数据的是以Region为单位进行，由PD Server集群进行统一条度，类似HBASE的Region调度。
TiKV集群存储的数据格式是key-value的，在TiDB中，并不是将数据直接存储在HDD/SSD中，而是通过RocksDB实现了TB级别的本地化存储方案，着重提的一点是：RocksDB和HBASE一样，都是通过**LSM(**Log-Structured MergeTree**)**作为存储方案，避免了B+树叶子节点膨胀带来的大量随机读写。从而提高声了整体的吞吐量。

#### 计算能力-TiDB Server
TiDB Server本身是无状态的，意味着当计算能力成为瓶颈的时候，可以直接扩容机器，对用户是透明的。理论上TiDB Server的数量并没有上限限制。

#### 总结
TiDB 作为新一代的NewSQL数据库，在数据库领域已经逐渐站稳脚跟，结合了Etcd/MySQL/HDFS/HBase/Spark等技术的突出特点，随着TiDB的大面积推广，会逐渐若活OLTP/OLAP的界限，并简化目前冗杂的ETL流程，引起新一轮的技术浪潮。

## TiDB安装部署
![Screen Shot 2022-04-18 at 10.32.50 PM.png](https://cdn.nlark.com/yuque/0/2022/png/694242/1650294183574-67346d98-5de3-4eba-94d1-d6b8b5e1b75c.png#clientId=ubef25aed-def2-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=180&id=u1004e933&margin=%5Bobject%20Object%5D&name=Screen%20Shot%202022-04-18%20at%2010.32.50%20PM.png&originHeight=180&originWidth=630&originalType=binary&ratio=1&rotation=0&showTitle=false&size=35426&status=done&style=none&taskId=u6565b6bc-163e-4823-a6b5-1441820e11e&title=&width=630)

#### 

#### Linux单机部署测试集群
```shell
# 1. 下载安装TiUP
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh

# 2. 添加export PATH=/root/.tiup/bin:$PATH 至.bashrc文件

# 3. 启动集群，制定版本v5.4.0, 2个TiDB instance, 3个PD instance, 3个TiKV instatnce
# -T用来持久化数据
tiup playground v5.4.0 -T v0.1 --db 2 --pd 3 --kv 3 --host 0.0.0.0
```
```shell
# 查看TiDB最近版本
tiup list tidb
```


#### Docker-compose集群版部署

(只支持v4.0以及之前的版本)
1. 下载tidb-docker-compose

git clone https://github.com/pingcap/tidb-docker-compose.git

2. 创建并启动集群，获取最新Docker镜像：

cd tidb-docker-compose && docker-compose pull && docker-compose up -d

3. 访问集群

mysql -h 127.0.0.1 -P 4000 -u root

4. 访问集群 Grafana 监控页面

http://localhost:3000/

5. 集群数据可视化

http://localhost:8010

![Screen Shot 2022-04-18 at 11.12.26 PM.png](https://cdn.nlark.com/yuque/0/2022/png/694242/1650294776628-40b86b38-372c-4cc6-bb2d-b2e781c18c96.png#clientId=u1f51a9cc-341b-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=582&id=uf54430f0&margin=%5Bobject%20Object%5D&name=Screen%20Shot%202022-04-18%20at%2011.12.26%20PM.png&originHeight=582&originWidth=772&originalType=binary&ratio=1&rotation=0&showTitle=false&size=107356&status=done&style=none&taskId=u4084c4dc-377e-43ce-a5b9-06e32930ddb&title=&width=772)

reference: [https://docs.pingcap.com/zh/tidb/v4.0/deploy-test-cluster-using-docker-compose](https://docs.pingcap.com/zh/tidb/v4.0/deploy-test-cluster-using-docker-compose)


## TiDB-读取历史数据

接下来介绍TiDB如何读取历史版本数据，包括具体的操作流程以及历史数据的保存策略。

#### 功能说明
TiDB实现了通过标准SQL接口读取历史数据功能，无需特殊的client或者driver。当数据被更新、删除后，依然可以通过SQL接口将更新/删除前的数据读取出来。

#### 操作流程
为支持读取历史版本数据，引入了一个新的system variable：tidb_snapshot，这个变量是Session范围有效，可以通过标准的Set语句修改其值。其值为文本，能够存储TSO和日期时间。TSO即是全局授时的时间戳，是从PD端获取的；日期时间的格式可以为："2020-10-08 16:45:26.999", 一般来说可以只写到秒，比如"2020-10-08 16:45:26"。当这个变量被设置时，TiDB会用这个时间戳建立Snapshot(没有开销，只是创建数据结构)，随后所有的Select操作都会在这个Snapshot上读取数据。

注意：TiDB的事务是通过PD进行全局授时，所以存储的数据版本也是以PD所授时间戳为版本号。在生成Snapshot时，是以tidb_snapshot变量的值作为版本号，如果TiDB Server所在机器和PD Server所在机器的本地时间相差较大，需要以PD的时间为准。

当读取历史版本操作结束后，可以结束当前Session或者是通过Set语句将tidb_snapshot变量的值设为""，即可读取最新版本的数据。

#### 示例
```plsql
-- 初始化一个表，并插入几行数据
create table t (c int);
insert into t values (1),(2),(3)

-- 查看表中数据
select * from t; -- 返回 1 2 3

--查看当前时间
select now(); -- 2022-04-19 22:00:00

--更新某一行数据
update t set c = 22 where c = 2;

--确认数据已经被更新
select * from t; -- 返回1 22 3

--设置一个特殊的环境变量, 这是一个session scope的变量，其意义为读取这个时间之前的最新的一个版本
set @@tidb_snapshot="2022-04-19 21:59:00";
select * from t; -- 返回1 2 3

--清空这个变量后，即可读取最新版本数据
set @@tidb_snapshot="";
select * from t; --返回1 22 3

```

## 数据迁移 - TiDB Lightning
TiDB Lightning是一个将圈梁数据高速导入到TiDB的集群的工具，目前支持Mydumper或CSV输出格式的数据源。你可以在一下两种场景下使用Lightning，

1. 迅速导入大量新数据
1. 备份恢复所有数据

TiDB Lightning主要包含两个部分:

1. tidb-lightning("前端")：主要完成适配工作，通过读取数据源，在下游TiDB集群建表、将数据转换成键/值对(KV对)发送到tikv-importer、检查数据完整性等。
1. tikv-importer("后端")：主要完成将数据导入TiKV集群的工作，把tidb-lightning写入的KV对缓存、排序、切分并导入到TiKV集群。

#### TiDB Lightning整体架构
![image.png](https://cdn.nlark.com/yuque/0/2022/png/694242/1650608660118-1d051612-e438-467b-a9b2-ea97be6d2d50.png#clientId=ucd434c4b-7612-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=585&id=u0e8a8442&margin=%5Bobject%20Object%5D&name=image.png&originHeight=585&originWidth=864&originalType=binary&ratio=1&rotation=0&showTitle=false&size=199240&status=done&style=none&taskId=u6c3baba8-1701-4c51-8943-3e87d37472e&title=&width=864)
#### 准备迁移工具
**wget **[**https://download.pingcap.org/tidb-enterprise-tools-nightly-linux-amd64.tar.gz**](https://download.pingcap.org)** (包含Syncer, Loader, Mydumper)**
**wget **[**https://download.pingcap.org/tidb-toolkit-{version}-linux-amd64.tar.gz**](https://download.pingcap.org/tidb-toolkit-{version}-linux-amd64.tar.gz)** 包含TiDB Lightning**

reference: [https://docs.pingcap.com/zh/tidb/v4.0/download-ecosystem-tools#tidb-lightning](https://docs.pingcap.com/zh/tidb/v4.0/download-ecosystem-tools#tidb-lightning)


#### 导出数据
```bash
mkdir -p /data/atlas_aidb/

cd /export/servers/tidb-enterprise-tools-latest-linux-amd64/bin

./mydumper -h 127.0.0.1 -P 3306 -u <username> -p <password> -t 16 -F 256 -B atlas_aidb -T <table_1>,<table_2> --skip-tz-utc -o /data/atlas_aidb/

cd /data/atlas_aidb

其中:
-B : 从指定数据库导出
-T : 只导出指定的表
-t 16: 使用16个线程导出数据
-F 256: 将每张表切分成多个文件，每个文件大小约为256MB
--skip-tz-utc: 添加这个参数则会忽略掉TiDB与导数据的机器之间时区设置不一致的情况，禁止自动转换

这样全量备份数据就导出到了/data/atlas_aidb目录中。
```

#### 启动tikv-importer

1. cd /export/servers/tidb-toolkit-latest-linux-amd64/bin

2. vim tikv-importer.toml
```bash
# TiKV Importer 配置文件模版

# 日志文件。
log-file = "tikv-importer.log"
# 日志等级：trace、debug、info、warn、error、off。
log-level = "info"

[server]
# tikv-importer 监听的地址，tidb-lightning 需要连到这个地址进行数据写入。
addr = "10.0.20.7:8287"

[import]
# 存储引擎文档 (engine file) 的文件夹路径。
import-dir = "/mnt/ssd/data.import/"
```

3. 运行tikv-importer

nohup ./tikv-importer -C tikv-importer.toml > nohup.out &

#### 启动tidb-lightning
```bash
#!/bin/bash
nohup ./tidb-lightning \
            --importer 192.168.20.10:8287 \
            -d /data/atlas_aidb/ \
            --tidb-host 172.16.31.2 \
            --tidb-user root \
            --log-file tidb-lightning.log \
        > nohup.out &
```
#### 检查数据
导入完毕后，TiDB Lightning 会自动退出。若导入成功，日志的最后一行会显示 **tidb lightning exit**.

Reference: [https://docs.pingcap.com/zh/tidb/v3.0/get-started-with-tidb-lightning](https://docs.pingcap.com/zh/tidb/v3.0/get-started-with-tidb-lightning)

## TiDB技术原理

### 存储
#### 
#### 引言
数据库、操作系统和编译器并称为三大系统，可以说是整个计算机软件的基石。其中数据库更靠近应用层，是很多业务的支撑。这一领域经过了几十年的发展，不断的有新的进展。

很多人用过数据库，但是很少有人实现过一个数据库，特别是实现一个分布式数据库。
了解数据库的实现原理和细节，一方面可以提高个人技术，对构建起他系统有帮助，另外一方面也有利于用好数据库。

研究一门技术最好的方式是研究其中一个开源项目，数据库也不例外。单机数据库领域有很多很好的开源项目，其中MySQL和PostgreSQL是其中知名度最高的两个，不少同学都看过这两个项目的代码。但是分布式数据库方面，好的开源项目并不多。TiDB目前获得了广泛的关注，特别是一些技术爱好者，希望能够参与这个项目。由于分布式数据库自身的复杂性，很多人并不能很好的理解整个项目，所以我希望能写一些文章，自顶向下，由浅入深，讲述TiDB的一些技术原理，包括用户可见的技术以及大量隐藏在SQL界面后用户不可见的技术点。

#### 保存数据
数据库最根本的功能是能把数据存下来。
保存数据的方法很多，最简单的方法是直接在内存中建一个数据结构，保存用户发来的数据。比如用一个数组，每当收到一条数据就向数组中追加一条记录。这个方案十分简单，能满足基本，并且性能肯定会很好，但是除此之外却是漏洞百出，其中最大的问题是数据完全在内存中，一旦停机或者服务重启，数据就会永久丢失。

为了解决数据丢失问题，我们可以把数据放在非易失存储介质(比如硬盘)中。改进的方案是在磁盘上创建一个文件，收到一条数据，就在文件中Append一行。OK，我们现在有了一个能持久化存储数据的方案，但是还是不够好，假设这块磁盘出现了坏道呢？没事，我们可以做RAID(Redundant Array of Independent Disks), 提供单机冗余存储。如果整台机器都挂了呢？比如火灾，RAID也保不住这些数据。我们还可以将存储改用网络存储，或者是通过硬件或者软件进行存储复制。到这里似乎我们已经解决了数据安全问题，可以松一口气了。But，在做复制的过程中国呢是否能保证副本之间的一致性？ 也就是在保证数据不丢的前期下，还要保证数据不错。保证数据不丢不错只是一项最基本的要求，还有更多令人头疼的问题等待解决:

- 能否支持跨数据中心的容灾
- 写入速度是否够快
- 数据保存下来后，是否方便读取
- 保存的数据如何修改？如果支持并发的修改
- 如何原子地修改多条记录

这些问题每一项都非常难，但是要做一个优秀的数据存储系统，必须要解决上述的每一个难题。为了解决数据存储问题，我们开发了TiKV这个项目。接下来我向大家介绍一下TiKV的一些设计思想和基本概念。

#### Key-Value
作为保存数据的系统，首先要决定的是数据的存储模型，也就是数据以什么样的形式保存下来。TiKV 的选择是 Key-Value 模型，并且提供有序遍历方法。简单来讲，可以将**TiKV看作一个巨大的Map,** 其中Key和Value都是原始的Byte数组，在这个Map中, Key按照Byte数组总的原始二进制比特位比较顺序排列。

TiKV 数据存储的两个关键点：

1. 这是一个巨大的 Map（可以类比一下 C++ 的 std::map），也就是存储的是 Key-Value Pairs（键值对）
1. 这个 Map 中的 Key-Value pair 按照 Key 的二进制顺序有序，也就是可以 Seek 到某一个 Key 的位置，然后不断地调用 Next 方法以递增的顺序获取比这个 Key 大的 Key-Value。

#### 本地存储(RocksDB)
任何持久化的存储引擎，数据终归要保存在磁盘上，TiKV 也不例外。但是 TiKV 没有选择直接向磁盘上写数据，而是把数据保存在 RocksDB 中，具体的数据落地由 RocksDB 负责。这个选择的原因是开发一个单机存储引擎工作量很大，特别是要做一个高性能的单机引擎，需要做各种细致的优化，而 RocksDB 是由 Facebook 开源的一个非常优秀的单机 KV 存储引擎，可以满足 TiKV 对单机引擎的各种要求。这里可以简单的认为 RocksDB 是一个单机的持久化 Key-Value Map。

RocksDB底层使用LSM树将对数据的修改增量保存在内存中，达到指定大小限制之后批量把数据flush到磁盘中，磁盘中树定期可以做merge操作，合并成一棵大树，以优化性能。

#### Raft
我们已经为数据找到了一个高效可靠的本地存储方案。接下来 TiKV 的实现面临一件更难的事情：如何保证单机失效的情况下，数据不丢失，不出错？简单来说，我们需要想办法把数据复制到多台机器上，这样一台机器挂了，我们还有其他的机器上的副本；复杂来说，我们还需要这个复制方案是可靠、高效并且能处理副本失效的情况。听上去比较难，但是好在我们有Raft协议。Raft 是一个一致性协议(算法)，本文只会对 Raft 做一个简要的介绍，细节问题可以参考它的[论文](https://raft.github.io/raft.pdf)。Raft 提供几个重要的功能：

1. Leader(主副本)选举
1. 成员变更(如添加副本、删除副本、转移Leader等操作)
1. 日志复制

TiKV利用Raft来做数据复制，每个数据变更都会落地为一条Raft日志，通过Raft的日志复制功能，将数据安全可靠地同步到复制组Group的每一个节点中。不过在实际写入中，根据 Raft 的协议，只需要同步复制到多数节点，即可安全地认为数据写入成功。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/694242/1650466252734-28339c5c-13af-4b64-9697-44cdd2ce145b.png#clientId=u328ac80b-7dbf-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=295&id=uce57d893&margin=%5Bobject%20Object%5D&name=image.png&originHeight=281&originWidth=496&originalType=binary&ratio=1&rotation=0&showTitle=false&size=63358&status=done&style=none&taskId=u6b3dd645-7223-41b7-bb32-c401b7ed547&title=&width=521)
总结一下，通过单机的 RocksDB，TiKV 可以将数据快速地存储在磁盘上；通过 Raft，将数据复制到多台机器上，以防单机失效。数据的写入是通过 Raft 这一层的接口写入，而不是直接写 RocksDB。通过实现 Raft，TiKV 变成了一个分布式的 Key-Value 存储，少数几台机器宕机也能通过原生的 Raft 协议自动把副本补全，可以做到对业务无感知。

#### Region

**Region是一个非常重要的概念，**是理解后续一系列机制的基础。

前面提到，TiKV 可以看做是一个巨大的有序的 KV Map，那么为了实现存储的水平扩展，数据将被分散在多台机器上。这里提到的数据分散在多台机器上和Raft的数据复制不是一个概念，在这一节我们先忘记Raft。假设所有的数据都只有一个副本，这样更容易理解。

对于一个 KV 系统，将数据分散在多台机器上有两种比较典型的方案：

- Hash：按照 Key 做 Hash，根据 Hash 值选择对应的存储节点。
- Range：按照 Key 分 Range，某一段连续的 Key 都保存在一个存储节点上。

TiKV 选择了第二种方式，将整个 Key-Value 空间分成很多段，每一段是一系列连续的 Key，将每一段叫做一个 Region，并且会尽量保持每个 Region 中保存的数据不超过一定的大小(这个大小可以配置)，目前在 TiKV 中默认是 96MB(根据TiDB版本可能不同)。每一个 Region 都可以用 [StartKey，EndKey) 这样一个左闭右开区间来描述。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/694242/1650467172791-20121525-31da-4c78-afa4-3d452feeb1aa.png#clientId=u328ac80b-7dbf-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=237&id=u41002fa4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=235&originWidth=292&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19122&status=done&style=none&taskId=u02bffbb4-d291-478d-9cc8-45035777b92&title=&width=294)
将数据划分成 Region 后，TiKV 将会做两件重要的事情：

- 以 Region 为单位，将数据分散在集群中所有的节点上，并且尽量保证每个节点上服务的 Region 数量差不多。
- 以 Region 为单位做 Raft 的复制和成员管理。

**这两点非常重要**：

- 先看第一点，数据按照 Key 切分成很多 Region，每个 Region 的数据只会保存在一个节点上面（暂不考虑多副本）。TiDB 系统会有一个组件 (PD) 来负责将 Region 尽可能均匀的散布在集群中所有的节点上，这样一方面实现了存储容量的水平扩展（增加新的节点后，会自动将其他节点上的 Region 调度过来），另一方面也实现了负载均衡（不会出现某个节点有很多数据，其他节点上没什么数据的情况）。同时为了保证上层客户端能够访问所需要的数据，系统中也会有一个组件 (PD) 记录 Region 在节点上面的分布情况，也就是通过任意一个 Key 就能查询到这个 Key 在哪个 Region 中，以及这个 Region 目前在哪个节点上（即 Key 的位置路由信息）。至于负责这两项重要工作的组件 (PD)，会在后续介绍。
- 对于第二点，TiKV 是以 Region 为单位做数据的复制，也就是一个 Region 的数据会保存多个副本，TiKV 将每一个副本叫做一个 Replica。Replica 之间是通过 Raft 来保持数据的一致，一个 Region 的多个 Replica 会保存在不同的节点上，构成一个 Raft Group。其中一个 Replica 会作为这个 Group 的 Leader，其他的 Replica 作为 Follower。默认情况下，所有的读和写都是通过 Leader 进行，读操作在 Leader 上即可完成，而写操作再由 Leader 复制给 Follower。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/694242/1650467533278-2f363aff-0c04-4252-bab2-72e57e1602b1.png#clientId=u328ac80b-7dbf-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=376&id=u7b3b3c15&margin=%5Bobject%20Object%5D&name=image.png&originHeight=446&originWidth=796&originalType=binary&ratio=1&rotation=0&showTitle=false&size=151204&status=done&style=none&taskId=ub48d720f-7f46-4e01-98da-43487fc5b28&title=&width=671)
以 Region 为单位做数据的分散和复制，TiKV 就成为了一个分布式的具备一定容灾能力的 KeyValue 系统，不用再担心数据存不下，或者是磁盘故障丢失数据的问题。

#### MVCC
很多数据库都会实现多版本并发控制 (MVCC)，TiKV 也不例外。设想这样的场景：两个客户端同时去修改一个 Key 的 Value，如果没有数据的多版本控制，就需要对数据上锁，在分布式场景下，可能会带来性能以及死锁问题。TiKV 的 MVCC 实现是通过在 Key 后面添加版本号来实现，简单来说，没有 MVCC 之前，可以把 TiKV 看做这样的：
```bash
Key1 -> Value
Key2 -> Value
……
KeyN -> Value
```
有了 MVCC 之后，TiKV 的 Key 排列是这样的：
```bash
Key1_Version3 -> Value
Key1_Version2 -> Value
Key1_Version1 -> Value
……
Key2_Version4 -> Value
Key2_Version3 -> Value
Key2_Version2 -> Value
Key2_Version1 -> Value
……
KeyN_Version2 -> Value
KeyN_Version1 -> Value
……
```
注意，对于同一个 Key 的多个版本，版本号较大的会被放在前面，版本号小的会被放在后面（见 [Key-Value](https://docs.pingcap.com/zh/tidb/stable/tidb-storage#key-value-pairs%E9%94%AE%E5%80%BC%E5%AF%B9) 一节，Key 是有序的排列），这样当用户通过一个 Key + Version 来获取 Value 的时候，可以通过 Key 和 Version 构造出 MVCC 的 Key，也就是 Key_Version。然后可以直接通过 RocksDB 的 SeekPrefix(Key_Version) API，定位到第一个大于等于这个 Key_Version 的位置。

**Refernece:  **[**https://docs.pingcap.com/zh/tidb/stable/tidb-storage**](https://docs.pingcap.com/zh/tidb/stable/tidb-storage)
### 计算
#### 引言
TiDB 在 TiKV 提供的分布式存储能力基础上，构建了兼具优异的交易处理能力与良好的数据分析能力的计算引擎。本文首先从数据映射算法入手介绍 TiDB 如何将库表中的数据映射到 TiKV 中的 (Key, Value) 键值对，然后描述 TiDB 元信息管理方式，最后介绍 TiDB SQL 层的主要架构。
对于计算层依赖的存储方案，本文只介绍基于 TiKV 的行存储结构。针对分析型业务的特点，TiDB 推出了作为 TiKV 扩展的列存储方案 [TiFlash](https://docs.pingcap.com/zh/tidb/stable/tiflash-overview)。

#### 表数据与Key-Value模型的映射关系
本小节介绍TiDB中数据到(Key, Value)键值对的映射方案。这里的数据主要包括一下两个方面：

- 表中每一行的数据，以下简称表数据
- 表中所有索引的数据，以下简称索引数据

**表数据与 Key-Value 的映射关系**
在关系型数据库中，一个表可能有很多列。要将一行中各列数据映射成一个 (Key, Value) 键值对，需要考虑如何构造 Key。首先，**OLTP 场景下有大量针对单行或者多行的增、删、改、查等操作**，要求数据库具备快速读取一行数据的能力。因此，**对应的 Key 最好有一个唯一 ID**（显示或隐式的 ID），以方便快速定位。其次，很多 OLAP 型查询需要进行全表扫描。如果能够将一个表中所有行的 Key 编码到一个区间内，就可以通过范围查询高效完成全表扫描的任务。

基于上述考虑，TiDB 中的表数据与 Key-Value 的映射关系作了如下设计：

- 为了保证同一个表的数据放在一起，方便查找，TiDB 会为每个表分配一个表 ID，用 TableID 表示。表 ID 是一个整数，在整个集群内唯一。
- TiDB 会为表中每行数据分配一个行 ID，用 RowID 表示。行 ID 也是一个整数，在表内唯一。对于行 ID，TiDB 做了一个小优化，**如果某个表有整数型的主键，TiDB 会使用主键的值当做这一行数据的行 ID。**



每行数据按照如下规则编码成 (Key, Value) 键值对：
```sql
Key:   tablePrefix{TableID}_recordPrefixSep{RowID}
Value: [col1, col2, col3, col4]
```
其中 **tablePrefix** 和 **recordPrefixSep** 都是特定的字符串常量，用于在 Key 空间内区分其他数据。其具体值在后面的小结中给出。

**索引数据和 Key-Value 的映射关系**
TiDB 同时支持主键和二级索引（包括唯一索引和非唯一索引）。与表数据映射方案类似，TiDB 为表中每个索引分配了一个索引 ID，用 IndexID 表示。
对于主键和唯一索引，需要根据键值快速定位到对应的 RowID，因此，按照如下规则编码成 (Key, Value) 键值对：
```sql
Key:   tablePrefix{tableID}_indexPrefixSep{indexID}_indexedColumnsValue
Value: RowID
```
对于不需要满足唯一性约束的普通二级索引，一个键值可能对应多行，需要根据键值范围查询对应的 RowID。因此，按照如下规则编码成 (Key, Value) 键值对：
```sql
Key:   tablePrefix{TableID}_indexPrefixSep{IndexID}_indexedColumnsValue_{RowID}
Value: null
```
**映射关系小结**
```sql
tablePrefix     = []byte{'t'}
recordPrefixSep = []byte{'r'}
indexPrefixSep  = []byte{'i'}
```
另外请注意，上述方案中，无论是表数据还是索引数据的 Key 编码方案，一个表内所有的行都有相同的 Key 前缀，一个索引的所有数据也都有相同的前缀。这样具有相同的前缀的数据，在 TiKV 的 Key 空间内，是排列在一起的。因此只要小心地设计后缀部分的编码方案，保证编码前和编码后的比较关系不变，就可以将表数据或者索引数据有序地保存在 TiKV 中。采用这种编码后，**一个表的所有行数据会按照 RowID 顺序地排列在 TiKV 的 Key 空间中，某一个索引的数据也会按照索引数据的具体的值（编码方案中的 indexedColumnsValue）顺序地排列在 Key 空间内。**

**Key-Value 映射关系示例**
最后通过一个简单的例子，来理解 TiDB 的 Key-Value 映射关系。假设 TiDB 中有如下这个表：
```sql
CREATE TABLE User {
  ID int,
  Name varchar(20),
  Role varchar(20),
  Age int,
  PRIMARY KEY(ID),
  Key idxAge(age)
}
```
假设该表中有 3 行数据：
```sql
1, "TiDB", "SQL Layer", 10
2, "TiKV", "KV Engine", 20
3, "PD", "Manager", 30
```
首先每行数据都会映射为一个 (Key, Value) 键值对，同时该表有一个 int 类型的主键，所以 RowID 的值即为该主键的值。假设该表的 TableID 为 10，则其存储在 TiKV 上的表数据为：
```sql
t10_r1 --> ["TiDB", "SQL Layer", 10]
t10_r2 --> ["TiKV", "KV Engine", 20]
t10_r3 --> ["PD", "Manager", 30]
```
除了主键外，该表还有一个非唯一的普通二级索引 idxAge，假设这个索引的 IndexID 为 1，则其存储在 TiKV 上的索引数据为：
```sql
t10_i1_10_1 --> null
t10_i1_20_2 --> null
t10_i1_30_3 --> null
```
#### 元信息管理
TiDB 中每个 Database 和 Table 都有元信息，也就是其定义以及各项属性。这些信息也需要持久化，TiDB 将这些信息也存储在了 TiKV 中。
每个 Database/Table 都被分配了一个唯一的 ID，这个 ID 作为唯一标识，并且在编码为 Key-Value 时，这个 ID 都会编码到 Key 中，再加上 m_ 前缀。这样可以构造出一个 Key，Value 中存储的是序列化后的元信息。
除此之外，TiDB 还用一个专门的 (Key, Value) 键值对存储当前所有表结构信息的最新版本号。这个键值对是全局的，每次 DDL (data definition language)操作的状态改变时其版本号都会加 1。目前，TiDB 把这个键值对持久化存储在 PD Server 中，其 Key 是 "/tidb/ddl/global_schema_version"，Value 是类型为 int64 的版本号值。TiDB 采用 Online Schema 变更算法，有一个后台线程在不断地检查 PD Server 中存储的表结构信息的版本号是否发生变化，并且保证在一定时间内一定能够获取版本的变化。

#### SQL层
TiDB 的 SQL 层，即 TiDB Server，负责将 SQL 翻译成 Key-Value 操作，将其转发给共用的分布式 Key-Value 存储层 TiKV，然后组装 TiKV 返回的结果，最终将查询结果返回给客户端。
这一层的节点都是无状态的，节点本身并不存储数据，节点之间完全对等。

#### SQL运算
最简单的方案就是通过上一节所述的表数据与Key-Value的映射关系([#rYIMs](#rYIMs))方案，将 SQL 查询映射为对 KV 的查询，再通过 KV 接口获取对应的数据，最后执行各种计算。

比如 **select count(*) from user where name = "TiDB"** 这样一个 SQL 语句，它需要读取表中所有的数据，然后检查 name 字段是否是 TiDB，如果是的话，则返回这一行。具体流程如下：

1. **构造出 Key Range**：一个表中所有的 RowID 都在 [0, MaxInt64) 这个范围内，使用 0 和 MaxInt64 根据行数据的 Key 编码规则，就能构造出一个 [StartKey, EndKey)的左闭右开区间。
1. **扫描 Key Range**：根据上面构造出的 Key Range，读取 TiKV 中的数据。
1. **过滤数据**：对于读到的每一行数据，计算 name = "TiDB" 这个表达式，如果为真，则向上返回这一行，否则丢弃这一行数据。
1. **计算 Count(*)**：对符合要求的每一行，累计到 Count(*) 的结果上面。

**整个流程示意图如下：**
![image.png](https://cdn.nlark.com/yuque/0/2022/png/694242/1650882173774-18b6adb9-b410-49eb-a73f-51c8057cb301.png#clientId=u328170b5-efe1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=391&id=u00e4e2f6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=559&originWidth=719&originalType=binary&ratio=1&rotation=0&showTitle=false&size=112418&status=done&style=none&taskId=uf23d609d-7e35-4381-941e-bb4925cdd5e&title=&width=502.5)
这个方案是直观且可行的，但是在分布式数据库的场景下有一些显而易见的问题：

- 在扫描数据的时候，每一行都要通过 KV 操作从 TiKV 中读取出来，至少有一次 RPC 开销，如果需要扫描的数据很多，那么这个开销会非常大。
- 并不是所有的行都满足过滤条件 name = "TiDB"，如果不满足条件，其实可以不读取出来。
- 此查询只要求返回符合要求行的数量，不要求返回这些行的值。

#### 分布式SQL运算
为了解决上述问题，计算应该需要尽量靠近存储节点，以避免大量的 RPC 调用。首先，SQL 中的**谓词条件 name = "TiDB"** 应被下推到存储节点进行计算，这样只需要返回有效的行，避免无意义的网络传输。然后，**聚合函数 Count(*) **也可以被下推到存储节点，进行**预聚合**，每个节点只需要返回一个 Count(*) 的结果即可，再由 SQL 层将各个节点返回的 **Count(*) 的结果累加求和**。

以下是数据逐层返回的示意图：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/694242/1650882549521-98e3087d-ba79-4a87-81c2-108ea9631973.png#clientId=u328170b5-efe1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=313&id=u946fe758&margin=%5Bobject%20Object%5D&name=image.png&originHeight=377&originWidth=609&originalType=binary&ratio=1&rotation=0&showTitle=false&size=113569&status=done&style=none&taskId=uf534c988-b30c-4f84-9218-7c3fad853c3&title=&width=505.5)

#### SQL层架构
通过上面的例子，希望大家对 SQL 语句的处理有一个基本的了解。实际上 TiDB 的 SQL 层要复杂得多，模块以及层次非常多，下图列出了重要的模块以及调用关系：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/694242/1650882897081-2111c662-e82f-4156-af33-d35422d441e4.png#clientId=uf0d617f1-ac5c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=277&id=ua56de0fc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=330&originWidth=825&originalType=binary&ratio=1&rotation=0&showTitle=false&size=187094&status=done&style=none&taskId=u2b7ccc0d-3319-4bae-8d58-2d93adc54a9&title=&width=693.5)
用户的 SQL 请求会直接或者通过 **Load Balancer** 发送到 TiDB Server，TiDB Server 会解析 **MySQL Protocol Packet**，获取请求内容，对 SQL 进行语法解析和语义分析，制定和优化查询计划，执行查询计划并获取和处理数据。数据全部存储在 TiKV 集群中，所以在这个过程中 TiDB Server 需要和 TiKV 交互，获取数据。最后 TiDB Server 需要将查询结果返回给用户。


### 调度
PD(Placement Driver) 是 TiDB 集群的管理模块，同时也负责集群数据的实时调度。本节介绍一下 PD 的设计思想和关键概念。

#### 场景描述
TiKV 集群是 TiDB 数据库的分布式 KV 存储引擎，数据以 Region 为单位进行复制和管理，每个 Region 会有多个副本 (Replica)，这些副本会分布在不同的 TiKV 节点上，其中 Leader 负责读/写，Follower 负责同步 Leader 发来的 Raft log。

需要考虑以下场景：

- 为了提高集群的空间利用率，需要根据 Region 的空间占用对副本进行合理的分布。
- 集群进行跨机房部署的时候，要保证一个机房掉线，不会丢失 Raft Group 的多个副本。
- 添加一个节点进入 TiKV 集群之后，需要合理地将集群中其他节点上的数据搬到新增节点。
- 当一个节点掉线时，需要考虑快速稳定地进行容灾。
   - 从节点的恢复时间来看
      - 如果节点只是短暂掉线（重启服务），是否需要进行调度。
      - 如果节点是长时间掉线（磁盘故障，数据全部丢失），如何进行调度。
   - 假设集群需要每个 Raft Group 有 N 个副本，从单个 Raft Group 的副本个数来看
      - 副本数量不够（例如节点掉线，失去副本），需要选择适当的机器的进行补充。
      - 副本数量过多（例如掉线的节点又恢复正常，自动加入集群），需要合理的删除多余的副本。
- 读/写通过 Leader 进行，Leader 的分布只集中在少量几个节点会对集群造成影响。
- 并不是所有的 Region 都被频繁的访问，可能访问热点只在少数几个 Region，需要通过调度进行负载均衡。
- 集群在做负载均衡的时候，往往需要搬迁数据，这种数据的迁移可能会占用大量的网络带宽、磁盘 IO 以及 CPU，进而影响在线服务。

以上问题和场景如果多个同时出现，就不太容易解决，因为需要考虑全局信息。同时整个系统也是在动态变化的，因此需要一个中心节点，来对系统的整体状况进行把控和调整，所以有了 PD 这个模块。

#### 调度需求
对以上的问题和场景进行分类和整理，可归为以下两类：
**第一类：作为一个分布式高可用存储系统，必须满足的需求，包括几种**

- 副本数量不能多也不能少
- 副本需要根据拓扑结构分布在不同属性的机器上
- 节点宕机或异常能够自动合理快速地进行容灾
- 


**第二类：作为一个良好的分布式系统，需要考虑的地方包括**

- 维持整个集群的 Leader 分布均匀
- 维持每个节点的储存容量均匀
- 维持访问热点分布均匀
- 控制负载均衡的速度，避免影响在线服务
- 管理节点状态，包括手动上线/下线节点，以及自动下线失效节点

满足第一类需求后，整个系统将具备强大的容灾功能。满足第二类需求后，可以使得系统整体的资源利用率更高且合理，具备良好的扩展性。
为了满足这些需求，首先需要收集足够的信息，比如每个节点的状态、每个 Raft Group 的信息、业务访问操作的统计等；其次需要设置一些策略，PD 根据这些信息以及调度的策略，制定出尽量满足前面所述需求的调度计划；最后需要一些基本的操作，来完成调度计划。

#### 调度的基本操作
调度的基本操作指的是为了满足调度的策略。上述调度需求可整理为以下三个操作：

- 增加一个副本
- 删除一个副本
- 将 Leader 角色在一个 Raft Group 的不同副本之间 transfer（迁移）。

刚好** Raft 协议**通过 **AddReplica、RemoveReplica、TransferLeader** 这三个命令，可以支撑上述三种基本操作。

#### 信息收集
调度依赖于整个集群信息的收集，简单来说，调度需要知道每个 TiKV 节点的状态以及每个 Region 的状态。**TiKV 集群会向 PD 汇报两类消息，TiKV 节点信息和 Region 信息：**

**每个 TiKV 节点会定期向 PD 汇报节点的状态信息**
TiKV 节点 (Store) 与 PD 之间存在心跳包，一方面 PD 通过心跳包检测每个 Store 是否存活，以及是否有新加入的 Store；另一方面，心跳包中也会携带这个 Store的状态信息，主要包括：

- 总磁盘容量
- 可用磁盘容量
- 承载的 Region 数量
- 数据写入/读取速度
- 发送/接受的 Snapshot 数量（副本之间可能会通过 Snapshot 同步数据）
- 是否过载
- labels 标签信息（标签是具备层级关系的一系列 Tag，能够感知拓扑信息）

通过使用 pd-ctl 可以查看到 TiKV Store 的状态信息。TiKV Store 的状态具体分为 Up，Disconnect，Offline，Down，Tombstone。各状态的关系如下：

- **Up**：表示当前的 TiKV Store 处于提供服务的状态。
- **Disconnect**：当 PD 和 TiKV Store 的心跳信息丢失超过 20 秒后，该 Store 的状态会变为 Disconnect 状态，当时间超过** max-store-down-time** 指定的时间后，该 Store 会变为 **Down** 状态。
- **Down**：表示该 TiKV Store 与集群失去连接的时间已经超过了 max-store-down-time 指定的时间，默认 30 分钟。超过该时间后，对应的 Store 会变为 Down，并且开始在存活的 Store 上补足各个 Region 的副本。
- **Offline**：当对某个 TiKV Store 通过 PD Control 进行手动下线操作，该 Store 会变为 Offline 状态。该状态只是 Store 下线的中间状态，处于该状态的 Store 会将其上的所有 Region 搬离至其它满足搬迁条件的 Up 状态 Store。当该 Store 的 leader_count 和 region_count (在 PD Control 中获取) 均显示为 0 后，该 Store 会由 Offline 状态变为 Tombstone 状态。在 Offline 状态下，禁止关闭该 Store 服务以及其所在的物理服务器。下线过程中，如果集群里不存在满足搬迁条件的其它目标 Store（例如没有足够的 Store 能够继续满足集群的副本数量要求），该 Store 将一直处于 Offline 状态。
- Tombstone：表示该 TiKV Store 已处于完全下线状态，可以使用 remove-tombstone 接口安全地清理该状态的 TiKV。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/694242/1650883758669-24c60036-2bf2-473f-8631-673dfbb44dcb.png#clientId=uf0d617f1-ac5c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=297&id=uded306df&margin=%5Bobject%20Object%5D&name=image.png&originHeight=447&originWidth=816&originalType=binary&ratio=1&rotation=0&showTitle=false&size=63752&status=done&style=none&taskId=ud0264425-a98c-4f6a-8b4e-99071487da9&title=&width=542)

**每个 Raft Group 的 Leader 会定期向 PD 汇报 Region 的状态信息**
每个 Raft Group 的 Leader 和 PD 之间存在心跳包，用于汇报这个Region的状态，主要包括下面几点信息：

- Leader 的位置
- Followers 的位置
- 掉线副本的个数
- 数据写入/读取的速度
- 


PD 不断的通过这两类心跳消息收集整个集群的信息，再以这些信息作为决策的依据。
除此之外，PD 还可以通过扩展的接口接受额外的信息，用来做更准确的决策。比如当某个 Store 的心跳包中断的时候，PD 并不能判断这个节点是临时失效还是永久失效，只能经过一段时间的等待（默认是 30 分钟），如果一直没有心跳包，就认为该 Store 已经下线，再决定需要将这个 Store 上面的 Region 都调度走。
但是有的时候，是运维人员主动将某台机器下线，这个时候，可以通过 PD 的管理接口通知 PD 该 Store 不可用，PD 就可以马上判断需要将这个 Store 上面的 Region 都调度走。

#### 调度的策略
PD 收集了这些信息后，还需要一些策略来制定具体的调度计划。
**一个 Region 的副本数量正确**
当 PD 通过某个 Region Leader 的心跳包发现这个 Region 的副本数量不满足要求时，需要通过 Add/Remove Replica 操作调整副本数量。出现这种情况的可能原因是：

- 某个节点掉线，上面的数据全部丢失，导致一些 Region 的副本数量不足
- 某个掉线节点又恢复服务，自动接入集群，这样之前已经补足了副本的 Region 的副本数量过多，需要删除某个副本
- 管理员调整副本策略，修改了 **max-replicas**的配置

**一个 Raft Group 中的多个副本不在同一个位置**
注意这里用的是『同一个位置』而不是『同一个节点』。在一般情况下，PD 只会保证多个副本不落在一个节点上，以避免单个节点失效导致多个副本丢失。在实际部署中，还可能出现下面这些需求：

- 多个节点部署在同一台物理机器上
- TiKV 节点分布在多个机架上，希望单个机架掉电时，也能保证系统可用性
- TiKV 节点分布在多个 IDC 中，希望单个机房掉电时，也能保证系统可用性

这些需求本质上都是某一个节点具备共同的位置属性，构成一个最小的『容错单元』，希望这个单元内部不会存在一个 Region 的多个副本。这个时候，可以给节点配置 [labels](https://github.com/tikv/tikv/blob/v4.0.0-beta/etc/config-template.toml#L140) 并且通过在 PD 上配置 [location-labels](https://github.com/pingcap/pd/blob/v4.0.0-beta/conf/config.toml#L100) 来指名哪些 label 是位置标识，需要在副本分配的时候尽量保证一个 Region 的多个副本不会分布在具有相同的位置标识的节点上。

**副本在 Store 之间的分布均匀分配**
由于每个 Region 的副本中存储的数据容量上限是固定的，通过维持每个节点上面副本数量的均衡，使得各节点间承载的数据更均衡。

**Leader 数量在 Store 之间均匀分配**
Raft 协议要求读取和写入都通过 Leader 进行，所以计算的负载主要在 Leader 上面，PD 会尽可能将 Leader 在节点间分散开。

**访问热点数量在 Store 之间均匀分配**
每个 Store 以及 Region Leader 在上报信息时携带了当前访问负载的信息，比如 Key 的读取/写入速度。PD 会检测出访问热点，且将其在节点之间分散开。

**各个 Store 的存储空间占用大致相等**
每个 Store 启动的时候都会指定一个 Capacity 参数，表明这个 Store 的存储空间上限，PD 在做调度的时候，会考虑节点的存储空间剩余量。

**控制调度速度，避免影响在线服务**
调度操作需要耗费 CPU、内存、磁盘 IO 以及网络带宽，需要避免对线上服务造成太大影响。PD 会对当前正在进行的操作数量进行控制，默认的速度控制是比较保守的，如果希望加快调度（比如停服务升级或者增加新节点，希望尽快调度），那么可以通过调节 PD 参数动态加快调度速度。

**支持手动下线节点**
当通过**pd-ctl**手动下线节点后，PD会在一定的速率控制下，将节点上的数据调度走。当调度完成后，就会将这个节点置为下线状态。

#### 调度的实现
PD 不断地通过 Store 或者 Leader 的心跳包收集整个集群信息，并且根据这些信息以及调度策略生成调度操作序列。每次收到 Region Leader 发来的心跳包时，PD 都会检查这个 Region 是否有待进行的操作，然后通过心跳包的回复消息，将需要进行的操作返回给 Region Leader，并在后面的心跳包中监测执行结果。
注意这里的操作只是给 Region Leader 的建议，并不保证一定能得到执行，具体是否会执行以及什么时候执行，由 Region Leader 根据当前自身状态来定。


## 实践经验

### 项目数据迁移

#### 数据导出
```bash
mkdir -p /home/jiaqi/tidb-migration/atlas_aidb

/home/jiaqi/tidb-migration/tidb-enterprise-tools-nightly-linux-amd64/bin

./mydumper -h 127.0.0.1 -P 3306 -u <username> -p <password> -t 16 -F 256 -B atlas_aidb -T <table_1>,<table_2> --skip-tz-utc -o /home/jiaqi/tidb-migration/atlas_aidb/

cd /home/jiaqi/tidb-migration/atlas_aidb

其中:
-B : 从指定数据库导出
-T : 只导出指定的表
-t 16: 使用16个线程导出数据
-F 256: 将每张表切分成多个文件，每个文件大小约为256MB
--skip-tz-utc: 添加这个参数则会忽略掉TiDB与导数据的机器之间时区设置不一致的情况，禁止自动转换

这样全量备份数据就导出到了/home/jiaqi/tidb-migration/atlas_aidb目录中。
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/694242/1650534331066-fcb688fa-76f3-497a-893f-59619907ebdd.png#clientId=ub716174a-365b-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=532&id=u433a521f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=532&originWidth=2186&originalType=binary&ratio=1&rotation=0&showTitle=false&size=373612&status=done&style=none&taskId=u89397505-7feb-489c-8406-1898f725529&title=&width=2186)
可以发现导出过程中，部分数据导入失败，原因以及解决方案：

很可能是因为一次 select 太多导致 TiDB OOM 重启从而 mydumper 连接断掉。可以通过以下方式回避这个问题：
a. 设置 --rows 参数，开启表内并发，同时划分的语句可以减少单次 select 的数据量。
b. 使用 dumpling v4.0.8，设置 --tidb-mem-quota-query 为 8GB 甚至更小的数字，[https://docs.pingcap.com/zh/tidb/stable/dumpling-overview#dumpling-主要选项表。](https://docs.pingcap.com/zh/tidb/stable/dumpling-overview#dumpling-%E4%B8%BB%E8%A6%81%E9%80%89%E9%A1%B9%E8%A1%A8%E3%80%82)
c. 使用 dumpling v4.0.8，设置 --params “tidb_distsql_scan_concurrency=5”，减少 TiDB scan 数据时的并发数。

#### 数据导入
```bash
./bin/loader  -d ../../atlas_aidb  -h 127.0.0.1 -u root -P 4000
```

![image.png](https://cdn.nlark.com/yuque/0/2022/png/694242/1650791143544-42068ce2-f14d-4f4c-9611-846a22fa7f4c.png#clientId=ucb4a1781-2f51-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1636&id=uae751f4f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1636&originWidth=1520&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1781294&status=done&style=none&taskId=uc4fad79a-ac2c-421c-85b5-bc545140d65&title=&width=1520)
#### 项目启动

1. 修改数据库root用户密码为
```sql
update user set authentication_string = password('mysql1qaz@WSX') where user = 'root';
flush PRIVILEGES;
```

2. 修改项目启动配置文件的数据库连接信息

![image.png](https://cdn.nlark.com/yuque/0/2022/png/694242/1650792833213-f4ea3715-5b5b-47a1-abd0-4f2014eadc27.png#clientId=u7dbdf0dc-6cd4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=326&id=u347427c0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=326&originWidth=1528&originalType=binary&ratio=1&rotation=0&showTitle=false&size=347207&status=done&style=none&taskId=ua4886b3f-743d-4e5f-94b8-83b05cc523f&title=&width=1528)

3. 启动后发现报错
   - ![image.png](https://cdn.nlark.com/yuque/0/2022/png/694242/1650792883832-cbdefe88-119e-4367-954d-4789f475deef.png#clientId=u7dbdf0dc-6cd4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=298&id=u0dddc4c0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=298&originWidth=2174&originalType=binary&ratio=1&rotation=0&showTitle=false&size=183905&status=done&style=none&taskId=ucea18509-58d4-4274-99da-541761679a1&title=&width=2174)

问题解决： 
show variables like "%tidb_enable%";
set global tidb_enable_noop_functions=1;

   - 也可能碰到-- Unsupported multi schema change错误导致启动失败，原因是因为对应的tidb版本不支持在单个ALTER TABLE修改多个列。

### 


