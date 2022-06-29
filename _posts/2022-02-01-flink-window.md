---
title: "Flink Window"
layout: post
date: 2022-02-01 22:45
tag:
- flink
- window
- api
star: false
category: blog
author: jiaqixu
---

### 目录

- [window概述](#window概述)
- [window类型](#window类型)
	- [TimeWindow](#timewindow)
		- [滚动窗口(Tumbling Windows)](#滚动窗口(tumbling-windows))
		- [滑动窗口(Sliding Windows)](#滑动窗口(sliding-windows))
		- [会话窗口(Session Windows)](#会话窗口(session-windows))
- [window API](#window-api)
	- [TimeWindow](#timewindow)
		- [滚动窗口](#滚动窗口)
		- [滑动窗口](#滑动窗口)
	- [CountWindow](#countwindow)
		- [滚动窗口](#滚动窗口)
		- [滑动窗口](#滑动窗口)
	- [其他可选API](#其他可选API)
	


### window概述

![image](/assets/images/blog/flink/stream.png)

* 一般真实的流都是无界的，怎样处理无界的数据?
*  可以把无限的数据流进行切分，得到有限的数据集进行处理 —— 也就是得到有界流
*  窗口(window)就是将无限流切割为有限流的一种方式，它会将流数据分发到有限大小的桶(bucket)中进行分析

### window类型
Window 可以分成两类:

* TimeWindow: 按照时间生成 Window。
* CountWindow: 按照指定的数据条数生成一个 Window，与时间无关。

#### TimeWindow 
对于TimeWindow，可以根据窗口实现原理的不同分成三类: 滚动窗口(Tumbling Window)、滑动窗口(Sliding Window)和会话窗口(Session Window)。

##### 滚动窗口(Tumbling Windows)
将数据依据固定的窗口长度对数据进行切片。

**特点: 时间对齐，窗口长度固定，没有重叠。一个数据只可能属于一个窗口，可以定义为左闭右开**

滚动窗口分配器将每个元素分配到一个指定窗口大小的窗口中，滚动窗口有一 个固定的大小，并且不会出现重叠。例如:如果你指定了一个 5 分钟大小的滚动窗 口，窗口的创建如下图所示：

![image](/assets/images/blog/flink/tumbling-window.png)

适用场景: 适合做 BI 统计等(做每个时间段的聚合计算)。

##### 滑动窗口(Sliding Windows)

滑动窗口是固定窗口的更广义的一种形式，滑动窗口由固定的**窗口长度**和**滑动间隔**组成。

**特点: 时间对齐，窗口长度固定，可以有重叠。一个数据可能属于多个窗口，取决于步长（slide）**

`一个数据最多可以属于的窗口数量=window size / window slide`

滑动窗口分配器将元素分配到固定长度的窗口中，与滚动窗口类似，窗口的大 小由窗口大小参数来配置，另一个窗口滑动参数控制滑动窗口开始的频率。因此， 滑动窗口如果滑动参数小于窗口大小的话，窗口是可以重叠的，在这种情况下元素 会被分配到多个窗口中。

例如，你有 10 分钟的窗口和 5 分钟的滑动，那么每个窗口中 5 分钟的窗口里包 含着上个 10 分钟产生的数据，如下图所示:

![image](/assets/images/blog/flink/sliding-window.png)

适用场景: 对最近一个时间段内的统计(求某接口最近 5min 的失败率来决定是否要报警)。

##### 会话窗口(Session Windows)
由一系列事件组合一个指定时间长度的 timeout 间隙组成，类似于 web 应用的session，也就是一段时间没有接收到新数据就会生成新的窗口。

**特点:时间无对齐。**

session 窗口分配器通过 session 活动来对元素进行分组，session 窗口跟滚动窗口和滑动窗口相比，不会有重叠和固定的开始时间和结束时间的情况，相反，当它 在一个固定的时间周期内不再收到元素，即非活动间隔产生，那个这个窗口就会关闭。一个 session 窗口通过一个 session 间隔来配置，这个 session 间隔定义了非活跃周期的长度，当这个非活跃周期产生，那么当前的 session 将关闭并且后续的元素将被分配到新的 session 窗口中去。

![image](/assets/images/blog/flink/session-window.png)



### Window API

#### TimeWindow

TimeWindow 是将指定时间范围内的所有数据组成一个 window，一次对一个 window 里面的所有数据进行计算。

##### 滚动窗口
Flink 默认的时间窗口根据 Processing Time 进行窗口的划分，将 Flink 获取到的数据根据进入 Flink 的时间划分到不同的窗口中。

```
DataStream<Tuple2<String, Double>> minTempPerWindowStream = dataStream
	.map(new MapFunction<SensorReading, Tuple2<String, Double>>() {
		@Override
		public Tuple2<String, Double> map(SensorReading value) throws
Exception {
			return new Tuple2<>(value.getId(), value.getTemperature());
		}
	})
	.keyBy(data -> data.f0)
	.timeWindow( Time.seconds(15) )
	.minBy(1);
```

时间间隔可以通过 Time.milliseconds(x)，Time.seconds(x)，Time.minutes(x)等其中的一个来指定。

##### 滑动窗口

滑动窗口(SlidingEventTimeWindows)和滚动窗口的函数名是完全一致的，只是在传参数时需要传入两个参数，一个是 window_size，一个是 sliding_size。

下面代码中的 sliding_size 设置为了 5s，也就是说，每 5s 就计算输出结果一次， 每一次计算的 window 范围是 15s 内的所有元素。

```
DataStream<SensorReading> minTempPerWindowStream = dataStream
	.keyBy(SensorReading::getId)
	.timeWindow( Time.seconds(15), Time.seconds(5) )
	.minBy("temperature");
```

时间间隔可以通过 Time.milliseconds(x)，Time.seconds(x)，Time.minutes(x)等其中的一个来指定。

#### CountWindow

CountWindow 根据窗口中相同 key 元素的数量来触发执行，执行时只计算元素数量达到窗口大小的 key 对应的结果。

注意: CountWindow 的 window_size 指的是相同 Key 的元素的个数，不是输入的所有元素的总数。

##### 滚动窗口

默认的 CountWindow 是一个滚动窗口，只需要指定窗口大小即可，当元素数量达到窗口大小时，就会触发窗口的执行。

```
DataStream<SensorReading> minTempPerWindowStream = dataStream
	.keyBy(SensorReading::getId)
	.countWindow( 5 )
	.minBy("temperature");
```

##### 滑动窗口

滑动窗口和滚动窗口的函数名是完全一致的，只是在传参数时需要传入两个参 数，一个是 window_size，一个是 sliding_size。

下面代码中的 sliding_size 设置为了 2，也就是说，每收到两个相同 key 的数据 就计算一次，每一次计算的 window 范围是 10 个元素。

```
DataStream<SensorReading> minTempPerWindowStream = dataStream
	.keyBy(SensorReading::getId)
	.countWindow( 10, 2 )
	.minBy("temperature");
```


#### window function

window function 定义了要对窗口中收集的数据做的计算操作，主要可以分为两类:

* 增量聚合函数(incremental aggregation functions)
	
	每条数据到来就进行计算，保持一个简单的状态。典型的增量聚合函数有 ReduceFunction, AggregateFunction。
	
* 全窗口函数(full window functions)

	先把窗口所有数据收集起来，等到计算的时候会遍历所有数据。 ProcessWindowFunction 就是一个全窗口函数。
	
#### 其他可选API
*  .trigger() —— 触发器: 定义 window 什么时候关闭，触发计算并输出结果
*  .evitor() —— 移除器: 定义移除某些数据的逻辑
*  .allowedLateness() —— 允许处理迟到的数据, 同时保证快速和最后结果的正确性，给一个相对允许的时间就可以了
*  .sideOutputLateData() —— 将迟到的数据放入侧输出流 
*  .getSideOutput() —— 获取侧输出流

![image](/assets/images/blog/flink/window-api.png)


