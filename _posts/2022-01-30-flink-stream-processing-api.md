---
title: "Flink Stream Processing APIs"
layout: 
date: 2022-01-30 22:45
tag:
- flink
- stream
- api
star: true
category: 
author: jiaqixu
---

### 目录
- [Environment](#environment)
	- [getExecutionEnvironment](#getexecutionenvironment)
	- [createLocalEnvironment](#createlocalenvironment)
	- [createRemoteEnvironment](#createremoteenvironment)
- [Source](#source)
	- [从集合读取数据](#从集合读取数据)
	- [从文件读取数据](#从文件读取数据)
	- [以kafka消息队列的数据作为来源](#以kafka消息队列的数据作为来源)
	- [自定义Source](#自定义source)
- [Transform](#transform)
	- [map](#map)
	- [flatMap](#flatmap)
	- [filter](#filter)
	- [keyBy](#keyby)
	- [Reduce](#reduce)
	- [Split和Select](#split和select)
	- [Connect和CoMap](#connect和comap)
	- [Union](#union)
	- [总结](#总结)
- [支持的数据类型](#支持的数据类型)
	- [基础数据类型](#基础数据类型)
	- [Java和Scala元组(Tuples)](#java和scala元组(tuples))
	- [Scala样例类(case classes)](#scala样例类(case-classes))
	- [Java简单对象(POJOs)](#java简单对象(pojos))
	- [其他](#其他)
- [实现UDF函数 —— 更细粒度的控制流](#实现udf函数-——-更细粒度的控制流)
	- [函数类(Function Classes)](#函数类(function-classes))
	- [匿名函数(Lambda Functions)](#匿名函数(lambda-functions))
	- [富函数(Rich Functions)](#富函数(rich-functions))
- [Sink](#sink)
	- [Kafka](#kafka)
	- [Redis](#redis)
	- [ElasticSearch](#elasticsearch)
	- [JDBC自定义sink](#JDBC自定义sink)


### Flink流处理API
![image](/assets/images/blog/flink/流处理API.png)

#### Environment

##### getExecutionEnvironment
创建一个执行环境，表示当前执行程序的上下文。 如果程序是独立调用的，则此方法返回本地执行环境；如果从命令行客户端调用程序以提交到集群，则此方法返回此集群的执行环境，也就是说，`getExecutionEnvironment`会根据查询运行的方式决定返回什么样的运行环境，是最常用的一种创建执行环境的方式。

`ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();`

`StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();`

 如果没有设置并行度，会以 flink-conf.yaml 中的配置为准，默认是 1。
 
![image](/assets/images/blog/flink/default-parallelism.png)

##### createLocalEnvironment
返回本地执行环境，需要在调用时指定默认的并行度。
`LocalStreamEnvironment env = StreamExecutionEnvironment.createLocalEnvironment(1);`

##### createRemoteEnvironment
返回集群执行环境，将Jar提交到远程服务器。需要在调用时指定JobManager的IP和端口号，并指定要在集群中运行的Jar包。
`StreamExecutionEnvironment env =
StreamExecutionEnvironment.createRemoteEnvironment("jobmanage-hostname", 6123, "YOURPATH//WordCount.jar");`

#### Source
##### 从集合读取数据
```java
package com.dongda.source;
 
import com.dongda.beans.SensorReading;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
 
import java.util.Arrays;
 
public class SourceTest1_Collection {
    public static void main(String[] args) throws Exception {
        //1.Source:从集合中读取数据
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
 
        DataStream<SensorReading> sensorDataStream = env.fromCollection(Arrays.asList(
                new SensorReading("sensor_1", 1547718199L, 35.8),
                new SensorReading("sensor_6", 1547718201L, 15.4),
                new SensorReading("sensor_7", 1547718202L, 6.7),
                new SensorReading("sensor_10", 1547718205L, 38.1)
        ));
 
        //2.打印
        sensorDataStream.print();
 
        //3.执行
        env.execute();
    }
}
```

##### 从文件读取数据
在resource下创建sensor.txt文件，以便读取
```
sensor_1, 1547718199, 35.8
sensor_6", 1547718201, 15.4
sensor_7", 1547718202, 6.7
sensor_10", 1547718205, 38.1
```

```
package com.dongda.source;
 
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
 
 
public class SourceTest2_File {
    public static void main(String[] args) throws Exception {
        //1.Source:从集合中读取数据
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        //从文件读取数据
        DataStream<String> sensorDataStream = env.readTextFile("/Users/jiaqi/personal_projects/flinkTutorial/src/main/resources/sensor.text");
 
        //2.打印
        sensorDataStream.print();
 
        //3.执行
        env.execute();
    }
}
```

##### 以kafka消息队列的数据作为来源 

```
<dependency>
<groupId>org.apache.flink</groupId>
<artifactId>flink-connector-kafka-0.11_2.12</artifactId>
<version>1.10.1</version>
</dependency>
```

```java
package com.dongda.source;
 
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer011;
 
import java.util.Properties;
 
public class SourceTest3_Kafka {
    public static void main(String[] args) throws Exception {
        //1.Source:从集合中读取数据
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        // kafka 配置项
        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "localhost:9092");
        properties.setProperty("group.id", "consumer-group");
        properties.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("auto.offset.reset", "latest");
 
 
        //从kafka读取数据
        DataStream<String> sensorDataStream = env.addSource(new FlinkKafkaConsumer011<String>("sensor",new SimpleStringSchema(),properties));
 
        //2.打印
        sensorDataStream.print();
 
        //3.执行
        env.execute();
    }
 
}
```

##### 自定义Source

```java
package com.jq.daily;

import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import java.util.HashMap;
import java.util.Random;

public class StreamAPI {
    public static void main(String[] args) throws Exception {
        //1.Source:从集合中读取数据
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        // 从文件读取数据
        DataStream<SensorReading> sensorDataStream = env.addSource(
                new MySensorSource()
        );

        //2.打印
        sensorDataStream.print();

        //3.执行
        env.execute();
    }

    private static class MySensorSource implements SourceFunction<SensorReading> {
        //定义一个标识位，用来控制数据的产生
        private boolean running = true;

        public void run(SourceContext<SensorReading> sourceContext) throws Exception {
            //定义一个随机数发生器
            Random random = new Random();

            //设置10个传感器的初始温度
            HashMap<String, Double> sensorTempMap = new HashMap<String, Double>();
            for (int i = 0; i < 10; i++) {
                sensorTempMap.put("sensor_" + (i + 1), 60 + random.nextGaussian() * 20);
            }

            while (running) {
                for (String sensorId : sensorTempMap.keySet()) {
                    //在当前的温度基础上随机波动
                    double newtemp = sensorTempMap.get(sensorId) + random.nextGaussian();
                    sensorTempMap.put(sensorId, newtemp);
                    sourceContext.collect(new SensorReading(sensorId, System.currentTimeMillis(), newtemp));

                }
                Thread.sleep(1000L);
            }

        }

        public void cancel() {

        }
    }
}

```

#### Transform
转换算子

##### map
![image](/assets/images/blog/flink/map.png)

```
DataStream<Integer> mapStram = dataStream.map(new MapFunction<String, Integer>() {
	public Integer map(String value) throws Exception {
		return value.length();
	}
});
```

##### flatMap

```
DataStream<String> flatMapStream = dataStream.flatMap(new FlatMapFunction<String,
String>() {
	public void flatMap(String value, Collector<String> out) throws Exception {
	String[] fields = value.split(",");
	for( String field: fields )
		out.collect(field);
	}
});
```

##### filter
![image](/assets/images/blog/flink/filter.png)
```
DataStream<Interger> filterStream = dataStream.filter(new FilterFunction<String>()
{
	public boolean filter(String value) throws Exception {
		return value == 1;
	}
});
```

##### keyBy
![image](/assets/images/blog/flink/keyBy.png)

`DataStream → KeyedStream`: 逻辑地将一个流拆分成不相交的分区，每个分区包含具有相同 key 的元素，在内部以 hash 的形式实现的。

##### 滚动聚合算子(Rolling Aggregation)
这些算子可以针对 KeyedStream 的每一个支流做聚合。
	
*  sum()
*  min()
*  max()
*  minBy()
*  maxBy()

##### Reduce

`KeyedStream → DataStream`: 一个分组数据流的聚合操作，合并当前的元素和上次聚合的结果，产生一个新的值，返回的流中包含每一次聚合的结果，而不是只返回最后一次聚合的最终结果。

```
DataStream<String> inputStream = env.readTextFile("sensor.txt");
// 转换成 SensorReading 类型
DataStream<SensorReading> dataStream = inputStream.map(new
MapFunction<String, SensorReading>() {
	// 分组
	public SensorReading map(String value) throws Exception {
		String[] fileds = value.split(",");
		return new SensorReading(fileds[0], new Long(fileds[1]), new Double(fileds[2]));
	}
});

KeyedStream<SensorReading, Tuple> keyedStream = dataStream.keyBy("id");
// reduce 聚合，取最小的温度值，并输出当前的时间戳
DataStream<SensorReading> reduceStream = keyedStream.reduce(new
ReduceFunction<SensorReading>() {
	@Override
	public SensorReading reduce(SensorReading value1, SensorReading value2)
throws Exception {
	return new SensorReading(
		value1.getId(),
		value2.getTimestamp(),
		Math.min(value1.getTemperature(), value2.getTemperature()));
	}
});
```

微小结：
以上我们介绍了基本的转换计算，也学完了稍微复杂一点的聚合计算，其实我们做大数据一般就是map、reduce这两组操作，要不就是你只跟当前状态有关，做一个简单转换，要不就是和之前的某些数据、和状态有关，做一个聚合、做一个统计，那还可以做什么操作呢？接下来介绍的又可以归为一大类，操作的是多条流，所以我们往往会把他们总结起来，叫做多流转换算子！现在开始


##### Split和Select
![image](/assets/images/blog/flink/split.png)

`DataStream → SplitStream`: 根据某些特征把一个 DataStream 拆分成两个或者多个DataStream。

Split名义上是把一条流拆成两个，但事实上SplitStream还是一条流，那Split操作到底干了一件什么事情呢？它是按照一定的特征，把数据做一个划分，然后给他相当于盖上一个戳（相当于一个拣选的标志），就我当前还是放在同一个流里面，但是我已经根据他不同的特点，盖了不同的戳，那接下来下一步就是根据那个戳做一个拣选，就可以得到不同的流。也就是说，你做完Split操作后一定要跟上一个Select操作，这才是一个完整的分流操作。


![image](/assets/images/blog/flink/select.png)

`SplitStream→DataStream`: 从一个 SplitStream 中获取一个或者多个DataStream.

基于这个SplitStream调用一个Select方法，然后根据不同的戳去提取，然后就能得到不同的DataStream，这就是一个完整的流程。

```java
SplitStream<SensorReading> splitStream = dataStream.split(new
OutputSelector<SensorReading>() {
	@Override
	public Iterable<String> select(SensorReading value) {
		return (value.getTemperature() > 30) ? Collections.singletonList("high") :
				Collections.singletonList("low");
	}
});
DataStream<SensorReading> highTempStream = splitStream.select("high");
DataStream<SensorReading> lowTempStream = splitStream.select("low");
DataStream<SensorReading> allTempStream = splitStream.select("high", "low");
```

##### Connect和CoMap

![image](/assets/images/blog/flink/connect.png)

`DataStream,DataStream → ConnectedStreams`: 连接两个保持他们类型的数据流，两个数据流被 Connect 之后，只是被放在了一个同一个流中，内部依然保持各自的数据和形式不发生任何变化，两个流相互独立。

![image](/assets/images/blog/flink/coMap-coFlatMap.png)

`ConnectedStreams → DataStream`: 作用于 ConnectedStreams 上，功能与 map
和 flatMap 一样，对 ConnectedStreams 中的每一个 Stream 分别进行 map 和 flatMap 处理。

```java
// 合流 connect
DataStream<Tuple2<String, Double>> warningStream = highTempStream.map(new
MapFunction<SensorReading, Tuple2<String, Double>>() {
	@Override
	public Tuple2<String, Double> map(SensorReading value) throws Exception {
		return new Tuple2<>(value.getId(), value.getTemperature());
	}
});

ConnectedStreams<Tuple2<String, Double>, SensorReading> connectedStreams =
warningStream.connect(lowTempStream);

DataStream<Object> resultStream = connectedStreams.map(new
CoMapFunction<Tuple2<String,Double>, SensorReading, Object>() {
	@Override
	public Object map1(Tuple2<String, Double> value) throws Exception {
		return new Tuple3<>(value.f0, value.f1, "warning");
	}
	
	@Override
	public Object map2(SensorReading value) throws Exception {
		return new Tuple2<>(value.getId(), "healthy");
	}
});
```

##### Union

前面介绍了connect连接两条流合并的操作，大家会发现connect的操作是非常灵活的，因为两个流的数据类型可以不一样，可以非常灵活的将各种各样的数据整合到一起，然后统一做操作（有点像一国两制)。但是他也有局限，它只能连接两条流，我们可以发现，他做map的时候只是实现了map1、map2。那么假如说我要想多条流合在一起怎么做呢？Union!它也不需要像connect操作后还要map，但是它要求合并的流的数据类型要一致。

总结一下就是说：各有各的特点，union的特点是可以合并多条流，但是它们的数据类型必须一样；connect的特点是数据类型可以不一样，然后再做转换，它就非常的灵活。实际使用中connect还是多一些。

![image](/assets/images/blog/flink/union.png)

`DataStream → DataStream`: 对两个或者两个以上的DataStream进行union操作，产生一个包含所有 DataStream 元素的新 DataStream。

```
DataStream<SensorReading> unionStream = highTempStream.union(lowTempStream);
```

Connect 与 Union 区别

1.  Union 之前两个流的类型必须是一样，Connect 可以不一样，在之后的 coMap 中再去调整成为一样的。
2. Connect 只能操作两个流，Union 可以操作多个。

##### 总结

到目前为止我们可以稍微总结一下，为什么所有的转换算子，都叫DataStreamAPI呢？就是因为我们基础的数据结构就是DataStream,然后里面可能有范型定义了内部的数据结构，它在做转换的过程当中，数据类型有可能会发生改变。那这里面涉及到流本身的数据结构变化的有哪些呢？简单转换不改变。KeyBy之后就会由一个DataStream 转换成 KeyedStream，当然，KeyedStream本质我们说还是一个DataStream，但是大家会发现KeyedStream里面有很多DataStream本身调用不了的方法，比如说聚合操作。

#### 支持的数据类型

Flink 流应用程序处理的是以数据对象表示的事件流。所以在 Flink 内部，我们需要能够处理这些对象。它们需要被序列化和反序列化，以便通过网络传送它们; 或者从状态后端、检查点和保存点读取它们。为了有效地做到这一点，Flink 需要明确知道应用程序所处理的数据类型。Flink 使用类型信息的概念来表示数据类型，并为每个数据类型生成特定的序列化器、反序列化器和比较器。

Flink 还具有一个类型提取系统，该系统分析函数的输入和返回类型，以自动获取类型信息，从而获得序列化器和反序列化器。但是，在某些情况下，例如 lambda 函数或泛型类型，需要显式地提供类型信息，才能使应用程序正常工作或提高其性能。

Flink 支持 Java 和 Scala 中所有常见数据类型。使用最广泛的类型有以下几种。

##### 基础数据类型
Flink 支持所有的 Java 和 Scala 基础数据类型，Int, Double, Long, String, ...

```
DataStream<Integer> numberStream = env.fromElements(1, 2, 3, 4);
numberStream.map(data -> data * 2);
```

##### Java和Scala元组(Tuples)

```
DataStream<Tuple2<String, Integer>> personStream = env.fromElements(
new Tuple2("Adam", 17),
new Tuple2("Sarah", 23) );
personStream.filter(p -> p.f1 > 18);
```

##### Scala样例类(case classes)
```
case class Person(name: String, age: Int)
val persons: DataStream[Person] = env.fromElements(
	Person("Adam", 17), 
	Person("Sarah", 23) )
 persons.filter(p => p.age > 18)
```

##### Java简单对象(POJOs)

```
public class Person {
	public String name;
	public int age;
	public Person() {}
	public Person(String name, int age) {
       	this.name = name;
	      this.age = age;
     }
 }
    
 DataStream<Person> persons = env.fromElements(
	new Person("Alex", 42), 
	new Person("Wendy", 23));
```

##### 其他
Flink 对 Java 和 Scala 中的一些特殊目的的类型也都是支持的，比如 Java 的
ArrayList，HashMap，Enum 等等。

#### 实现UDF函数 —— 更细粒度的控制流

##### 函数类(Function Classes)

Flink 暴露了所有 udf 函数的接口(实现方式为接口或者抽象类)。例如 MapFunction, FilterFunction, ProcessFunction 等等。

下面例子实现了 FilterFunction 接口:

```
DataStream<String> flinkTweets = tweets.filter(new FlinkFilter());

public static class FlinkFilter implements FilterFunction<String> { 
	@Override
	public boolean filter(String value) throws Exception { 
		return value.contains("flink");
	}
}
```

还可以将函数实现成匿名类

```
DataStream<String> flinkTweets = tweets.filter(new FilterFunction<String>() {
	@Override
	public boolean filter(String value) throws Exception {
		return value.contains("flink");
	}
});
```

我们 filter 的字符串"flink"还可以当作参数传进去。

```
DataStream<String> tweets = env.readTextFile("INPUT_FILE ");
DataStream<String> flinkTweets = tweets.filter(new KeyWordFilter("flink"));
 
public static class KeyWordFilter implements FilterFunction<String> {
        private String keyWord;
 
        KeyWordFilter(String keyWord) {
            this.keyWord = keyWord;
        }
 
        @Override
        public boolean filter(String value) throws Exception {
            return value.contains(this.keyWord);
        }
    }
```

##### 匿名函数(Lambda Functions)
```
DataStream<String> tweets = env.readTextFile("INPUT_FILE");
DataStream<String> flinkTweets = tweets.filter( tweet -> tweet.contains("flink") );
```

##### 富函数(Rich Functions)
“富函数”是 DataStream API 提供的一个函数类的接口，所有 Flink 函数类都有其 Rich 版本。它与常规函数的不同在于，可以获取运行环境的上下文，并拥有一些生命周期方法，所以可以实现更复杂的功能。

* RichMapFunction
* RichFlatMapFunction
* RichFilterFunction
* ...

Rich Function 有一个生命周期的概念。典型的生命周期方法有:

* open()方法是 rich function 的初始化方法，当一个算子例如 map 或者 filter
被调用之前 open()会被调用。
* close()方法是生命周期中的最后一个调用的方法，做一些清理工作。
* getRuntimeContext()方法提供了函数的 RuntimeContext 的一些信息，例如函
数执行的并行度，任务的名字，以及 state 状态

```
public static class MyMapFunction extends RichMapFunction<SensorReading,
Tuple2<Integer, String>> {
	@Override
	public Tuple2<Integer, String> map(SensorReading value) throws Exception {
		return new Tuple2<>(getRuntimeContext().getIndexOfThisSubtask(), value.getId());
	}
	
	@Override
	public void open(Configuration parameters) throws Exception {
		System.out.println("my map open");
	// 以下可以做一些初始化工作，例如建立一个和 HDFS 的连接
	}
	
	@Override
	public void close() throws Exception {
		System.out.println("my map close");
	// 以下做一些清理工作，例如断开和 HDFS 的连接
	}
}
```

#### Sink
Flink 没有类似于 spark 中 foreach 方法，让用户进行迭代的操作。虽有对外的输出操作都要利用 Sink 完成。最后通过类似如下方式完成整个任务最终输出操作。

```
stream.addSink(new MySink(xxxx))
```
官方提供了一部分的框架的sink。除此之外，需要用户自定义实现sink。

![image](/assets/images/blog/flink/sink.png)

##### Kafka

pom.xml

```
<dependency>
<groupId>org.apache.flink</groupId>
<artifactId>flink-connector-kafka-0.11_2.12</artifactId>
<version>1.10.1</version>
</dependency>
```
主函数中添加 sink:

```
dataStream.addSink(new FlinkKafkaProducer011[String]("localhost:9092", "test", new SimpleStringSchema()))
```

##### Redis
pom.xml

```
<dependency>
<groupId>org.apache.bahir</groupId>
<artifactId>flink-connector-redis_2.11</artifactId>
<version>1.0</version>
</dependency>
```

定义一个 redis 的 mapper 类，用于定义保存到 redis 时调用的命令:

```
public static class MyRedisMapper implements RedisMapper<SensorReading>{
	// 保存到 redis 的命令，存成哈希表
	public RedisCommandDescription getCommandDescription() {
		return new RedisCommandDescription(RedisCommand.HSET, "sensor_tempe");
	}
	
	public String getKeyFromData(SensorReading data) {
		return data.getId();
	}
	
	public String getValueFromData(SensorReading data) {
		return data.getTemperature().toString();
	}
}
```

在主函数中调用:

```
FlinkJedisPoolConfig config = new FlinkJedisPoolConfig.Builder()
	.setHost("localhost")
	.setPort(6379)
	.build();
dataStream.addSink( new RedisSink<SensorReading>(config, new MyRedisMapper()) );
```

输出结果:

![image](/assets/images/blog/flink/redis-sink.png)

##### ElasticSearch

pom.xml

```
<dependency>
<groupId>org.apache.flink</groupId>
<artifactId>flink-connector-elasticsearch6_2.12</artifactId>
<version>1.10.1</version>
</dependency>
```

在主函数中调用

```
// es 的 httpHosts 配置
ArrayList<HttpHost> httpHosts = new ArrayList<>();
httpHosts.add(new HttpHost("localhost", 9200));
dataStream.addSink( new ElasticsearchSink.Builder<SensorReading>(httpHosts, new
MyEsSinkFunction()).build());
```

ElasitcsearchSinkFunction 的实现

```
public static class MyEsSinkFunction implements
ElasticsearchSinkFunction<SensorReading>{
	@Override
	public void process(SensorReading element, RuntimeContext ctx, RequestIndexer
indexer) {
	HashMap<String, String> dataSource = new HashMap<>();
	dataSource.put("id", element.getId());
	dataSource.put("ts", element.getTimestamp().toString());
	dataSource.put("temp", element.getTemperature().toString());
	IndexRequest indexRequest = Requests.indexRequest()
		.index("sensor")
		.type("readingData")
		.source(dataSource);
	indexer.add(indexRequest);
	}
}
```

##### JDBC自定义sink

```
<dependency>
<groupId>mysql</groupId>
<artifactId>mysql-connector-java</artifactId>
<version>5.1.44</version>
</dependency>
```

添加 MyJdbcSink
```
public class SinkTest4_Jdbc {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);//为了方便观察打印出来的结果，将全局并行度设置为1
 
        //从文件里面读取数据
        DataStream<String> inputStream = env.readTextFile("sensor.txt");
 
        //lamda表达式写法 转换成SensorReading类型
        DataStream<SensorReading> dataStream = inputStream.map(line -> {
            String[] fields = line.split(",");
            return new SensorReading(fields[0], new Long(fields[1]), new Double(fields[2]));
        });
 
        dataStream.addSink(new MyJdbcSink());
 
        env.execute();
    }
 
    //实现自定义的SinkFunction
    private static class MyJdbcSink extends RichSinkFunction<SensorReading> {
        //声明连接和预编译语句
        Connection connection=null;
        PreparedStatement insertStmt=null;
        PreparedStatement updateStmt=null;
        @Override
        public void open(Configuration parameters) throws Exception {
            connection= DriverManager.getConnection("jdbc:mysql://localhost:3306/test","root","123456");
            insertStmt=connection.prepareStatement("insert into sensor_temp(id,temp) values(?,?)");
            updateStmt=connection.prepareStatement("update sensor_temp set temp= ? where id = ?");
        }
 
 
        //每来一条数据，调用连接，执行SQL
        @Override
        public void invoke(SensorReading value, Context context) throws Exception {
            //直接执行更新语句，如果没有更新那么就插入
            updateStmt.setDouble(1,value.getTemperature());
            updateStmt.setString(2,value.getId());
            updateStmt.execute();
            if (updateStmt.getUpdateCount()==0){
                insertStmt.setString(1,value.getId());
                insertStmt.setDouble(2,value.getTemperature());
                insertStmt.execute();
            }
        }
 
        //最后有始有终，我们要进行关闭
        @Override
        public void close() throws Exception {
            insertStmt.close();
            updateStmt.close();
            connection.close();
        }
    }
}
```


