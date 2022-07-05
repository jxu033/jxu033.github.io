---
title: "Flink Time Syntax And Watermark"
layout: 
date: 2022-02-02 22:45
tag:
- flink
- time-syntax
- watermark
star: true
category: 
author: jiaqixu
---

- [Flink中的时间语义](#flink中的时间语义)
- [EventTime的引入](#eventtime的引入)
- [Watermark](#watermark)
	- [基本概念](#基本概念) 
	- [Watermark的引入](#watermark的引入)
		- [Assigner with periodic watermarks](#assigner-with-periodic-watermarks)
		- [Assigner with punctuated watermarks](#assigner-with-punctuated-watermarks)
- [EventTime在window中的使用](#eventtime在window中的使用)
	-	




### Flink中的时间语义

在 Flink 的流式处理中，会涉及到时间的不同概念，如下图所示:

![image](/assets/images/blog/flink/flink时间概念.png)

**Event Time**：是事件创建的时间。它通常由事件中的时间戳描述，例如采集的日志数据中，每一条日志都会记录自己的生成时间，Flink 通过时间戳分配器访问事件时间戳。

**Ingestion Time**: 是数据进入 Flink 的时间。

**Processing Time:** 是每一个执行基于时间操作的算子的本地系统时间，与机器相关，默认的时间属性就是 Processing Time。

例如，一条日志进入 Flink 的时间为 2017-11-12 10:00:00.123，到达 Window 的 系统时间为 2017-11-12 10:00:01.234，日志的内容如下:

`2017-11-02 18:37:15.624 INFO Fail over to rm2`

对于业务来说，要统计 1min 内的故障日志个数，哪个时间是最有意义的?—— eventTime，因为我们要根据日志的生成时间进行统计。

### EventTime的引入

在 Flink 的流式处理中，绝大部分的业务都会使用 eventTime，一般只在 `eventTime` 无法使用时，才会被迫使用 `ProcessingTime` 或者 `IngestionTime`。

如果要使用 `EventTime`，那么需要引入` EventTime` 的时间属性，引入方式如下所示:

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment
// 从调用时刻开始给 env 创建的每一个 stream 追加时间特征env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
```

### Watermark

#### 基本概念

我们知道，流处理从事件产生，到流经 source，再到 operator，中间是有一个过程和时间的，虽然大部分情况下，流到 operator 的数据都是按照事件产生的时间顺序来的，但是也不排除由于网络、分布式等原因，导致乱序的产生，所谓乱序，就是指 Flink 接收到的事件的先后顺序不是严格按照事件的 Event Time 顺序排列的。

![image](/assets/images/blog/flink/数据的乱序.png)

那么此时出现一个问题，一旦出现乱序，如果只根据 eventTime 决定 window 的 运行，我们不能明确数据是否全部到位，但又不能无限期的等下去，此时必须要有个机制来保证一个特定的时间后，必须触发 window 去进行计算了，这个特别的机制，就是 Watermark。

* Watermark 是一种衡量 Event Time 进展的机制。
* Watermark是用于处理乱序事件的，而正确的处理乱序事件，通常用 Watermark 机制结合 window 来实现。
* 数据流中的 Watermark 用于表示 timestamp 小于 Watermark 的数据，都已经 到达了，因此，window 的执行也是由 Watermark 触发的。
* Watermark 可以理解成一个延迟触发机制，我们可以设置 Watermark 的延时 时长 t，每次系统会校验已经到达的数据中最大的 maxEventTime，然后认定 eventTime 小于 maxEventTime - t 的所有数据都已经到达，如果有窗口的停止时间等于 maxEventTime – t，那么这个窗口被触发执行。

有序流的 Watermarker 如下图所示: (Watermark 设置为 0)

![image](/assets/images/blog/flink/有序数据的watermark.png)

乱序流的 Watermarker 如下图所示: (Watermark 设置为 2)

![image](/assets/images/blog/flink/无序数据的watermark.png)

当 Flink 接收到数据时，会按照一定的规则去生成 Watermark，这条 Watermark 就等于当前所有到达数据中的 maxEventTime - 延迟时长，也就是说，Watermark 是 基于数据携带的时间戳生成的，一旦 Watermark 比当前未触发的窗口的停止时间要 晚，那么就会触发相应窗口的执行。由于 event time 是由数据携带的，因此，如果 运行过程中无法获取新的数据，那么没有被触发的窗口将永远都不被触发。

上图中，我们设置的允许最大延迟到达时间为 2s，所以时间戳为 7s 的事件对应 的 Watermark 是 5s，时间戳为 12s 的事件的 Watermark 是 10s，如果我们的窗口 1 是 1s~5s，窗口 2 是 6s~10s，那么时间戳为 7s 的事件到达时的 Watermarker 恰好触 发窗口 1，时间戳为 12s 的事件到达时的 Watermark 恰好触发窗口 2。

Watermark 就是触发前一窗口的“关窗时间”，一旦触发关门那么以当前时刻 为准在窗口范围内的所有所有数据都会收入窗中。

只要没有达到水位那么不管现实中的时间推进了多久都不会触发关窗。

#### Watermark的引入

watermark 的引入很简单，对于乱序数据，最常见的引用方式如下:

```java
dataStream.assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<SensorReading>(Time.milliseconds(1000))
{
    @Override
    public long extractTimestamp (SensorReading element){
	    return element.getTimestamp() * 1000L;
    }
} );
```

Event Time 的使用一定要指定数据源中的时间戳。否则程序无法知道事件的事件时间是什么(数据源里的数据没有时间戳的话，就只能使用 Processing Time 了)。

我们看到上面的例子中创建了一个看起来有点复杂的类，这个类实现的其实就是分配时间戳的接口。Flink 暴露了 TimestampAssigner 接口供我们实现，使我们可以自定义如何从事件数据中抽取时间戳。

```
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
// 设置事件时间语义
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
DataStream<SensorReading> dataStream = env.addSource(new SensorSource())
        .assignTimestampsAndWatermarks(new MyAssigner());
```

MyAssigner 有两种类型

1）AssignerWithPeriodicWatermarks<br>
2）AssignerWithPunctuatedWatermarks <br>
以上两个接口都继承自 TimestampAssigner。

##### Assigner with periodic watermarks

周期性的生成 watermark: 系统会周期性的将 watermark 插入到流中(水位线也是一种特殊的事件!)。默认周期是 200 毫秒。可以使用 `ExecutionConfig.setAutoWatermarkInterval()`方法进行设置。

```
// 每隔 5 秒产生一个 watermark 
env.getConfig.setAutoWatermarkInterval(5000);
```

**产生 watermark 的逻辑**: 每隔 5 秒钟，Flink 会调用 `AssignerWithPeriodicWatermarks`的 `getCurrentWatermark()`方法。如果方法返回一个时间戳大于之前水位的时间戳，新的 watermark 会被插入到流中。这个检查保证了 水位线是单调递增的。如果方法返回的时间戳小于等于之前水位的时间戳，则不会产生新的 watermark。 

例子，自定义一个周期性的时间戳抽取:

```java
// 自定义周期性时间戳分配器
public static class MyPeriodicAssigner implements AssignerWithPeriodicWatermarks<SensorReading>{
    private Long bound = 60 * 1000L; // 延迟一分钟
    private Long maxTs = Long.MIN_VALUE; // 当前最大时间戳
    @Nullable
    @Override
    public Watermark getCurrentWatermark() {
        return new Watermark(maxTs - bound); }
    @Override
    public long extractTimestamp(SensorReading element, long previousElementTimestamp) {
        maxTs = Math.max(maxTs, element.getTimestamp());
        return element.getTimestamp(); 
    }
}
```

一种简单的特殊情况是，如果我们事先得知数据流的时间戳是单调递增的，也 就是说没有乱序，那我们可以使用 AscendingTimestampExtractor，这个类会直接使 用数据的时间戳生成 watermark。

```java
DataStream<SensorReading> dataStream = ...

dataStream.assignTimestampsAndWatermarks(
	new AscendingTimestampExtractor<SensorReading>() {
		@Override
		public long extractAscendingTimestamp(SensorReading element) { 
			return element.getTimestamp() * 1000;
		} 
});
```

而对于乱序数据流，如果我们能大致估算出数据流中的事件的最大延迟时间， 就可以使用如下代码:

```java
DataStream<SensorReading> dataStream = ...

dataStream.assignTimestampsAndWatermarks(
	new BoundedOutOfOrdernessTimestampExtractor<SensorReading>(Time.seconds(1)) {
		@Override
		public long extractTimestamp(SensorReading element) { 
			return element.getTimestamp() * 1000L;
		}
});
```

##### Assigner with punctuated watermarks

间断式地生成 watermark。和周期性生成的方式不同，这种方式不是固定时间的， 而是可以根据需要对每条数据进行筛选和处理。直接上代码来举个例子，我们只给 sensor_1 的传感器的数据流插入watermark

```java
public static class MyPunctuatedAssigner implements AssignerWithPunctuatedWatermarks<SensorReading>{
    private Long bound = 60 * 1000L; // 延迟一分钟
    @Nullable
    @Override
    public Watermark checkAndGetNextWatermark(SensorReading lastElement, long
            extractedTimestamp) {
        if(lastElement.getId().equals("sensor_1"))
            return new Watermark(extractedTimestamp - bound);
        else
            return null;
    }
    @Override
    public long extractTimestamp(SensorReading element, long previousElementTimestamp)
    {
        return element.getTimestamp();
    }
}
```

### EventTime在window中的使用

#### 滚动窗口(TumblingEventTimeWindows)

```java
package com.dongda.window;
 
import com.dongda.beans.SensorReading;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor;
import org.apache.flink.streaming.api.windowing.time.Time;
 
public class WindowTest1_EventTimeWindow {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);//为了方便观察打印出来的结果，将全局并行度设置为1
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        env.getConfig().setAutoWatermarkInterval(100);
 
        //从文件里面读取数据
        DataStream<String> inputStream = env.readTextFile("/Users/jiaqi/flink/src/main/resources/sensor.txt");
 
        //转成SensorReading类型,分配了时间戳和watermark
        DataStream<SensorReading> dataStream = inputStream.map(line -> {
            String[] fields = line.split(",");
            return new SensorReading(fields[0], new Long(fields[1]), new Double(fields[2]));
        })
                //乱序数据设置时间戳和watermark
                .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<SensorReading>(Time.seconds(1000L)) {
                    @Override
                    public long extractTimestamp(SensorReading sensorReading) {
                        return sensorReading.getTimestamp() * 1000L;
                    }
                });
        //基于事件时间的开窗聚合,统计15秒内温度的最小值
        SingleOutputStreamOperator<SensorReading> minTempStream = dataStream.keyBy("id")
                .timeWindow(Time.seconds(15))
                .minBy("temperature");
 
        minTempStream.print("minTemp");
        env.execute();
    }
}
```

这里第一个窗口的起始点是：如果不加offset的话是windowSize的整数倍，这里是15的整数倍；可以加offset用来处理不同时区的时间。

那么对于乱序程度很大的数据，虽然watermark会解决一部分，另一部分需要用allowedLateness来处理，例如我们设置1分钟之内，只要来了就更新一次，另外我们还有一个兜底的方法，加入1分钟之后还有迟到的数据怎么办，使用sideOutputLateData方法来处理，但是 延迟的时间都是以watermark为准，watermark对什么都有影响。代码如下：

```java
package com.dongda.window;
 
import com.dongda.beans.SensorReading;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.OutputTag;
 
public class WindowTest1_EventTimeWindow {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);//为了方便观察打印出来的结果，将全局并行度设置为1
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        env.getConfig().setAutoWatermarkInterval(100);
 
        //从文件里面读取数据
        DataStream<String> inputStream = env.readTextFile("/Users/haitaoyou/developer/flink/src/main/resources/sensor.txt");
 
        //转成SensorReading类型,分配了时间戳和watermark
        DataStream<SensorReading> dataStream = inputStream.map(line -> {
            String[] fields = line.split(",");
            return new SensorReading(fields[0], new Long(fields[1]), new Double(fields[2]));
        })
                //乱序数据设置时间戳和watermark
                .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<SensorReading>(Time.seconds(1000L)) {
                    @Override
                    public long extractTimestamp(SensorReading sensorReading) {
                        return sensorReading.getTimestamp() * 1000L;
                    }
                });
        //基于事件时间的开窗聚合,统计15秒内温度的最小值
        //设置allowedLateness 和兜底的sideOutputLateData
        OutputTag<SensorReading> outputTag=new OutputTag<SensorReading>("late");
        SingleOutputStreamOperator<SensorReading> minTempStream = dataStream.keyBy("id")
                .timeWindow(Time.seconds(15))
                .allowedLateness(Time.minutes(1))
                .sideOutputLateData(outputTag)
                .minBy("temperature");
 
        minTempStream.print("minTemp");
        minTempStream.getSideOutput(outputTag).print();
        env.execute();
    }
}
```

#### 滑动窗口(SlidingEventTimeWindows)
#### 会话窗口(EventTimeSessionWindows)
