---
title: 翻译 Chapter 22 Event-Time and Stateful Processing
date: 2019-07-28
copyright: true
categories: English,中文
tags: [Spark]
mathjax: true
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 22 Event-Time and Stateful Processing

Chapter 21 covered the core concepts and basic APIs; this chapter dives into event-time and stateful processing. Event-time processing is a hot topic because we analyze information with respect to the time that it was created, not processed. The key idea between this style of processing is that over the lifetime of the job, Spark will maintain relevant state that it can update over the course of the job before outputting it to the sink.

第21章介绍了核心概念和基本API；本章深入讨论事件时间（event-time）和状态处理（stateful processing）。事件时间处理是一个热门话题，因为我们分析与创建时间相关的信息，而不是与处理时间相关的信息。这种处理方式之间的关键思想是，在作业的整个生命周期中，Spark 将保持相关状态，以便在将作业输出到接收器之前在作业的整个过程中进行更新。

Let’s cover these concepts in greater detail before we begin working with code to show they work.

在我们开始使用代码来展示这些概念的工作之前，让我们更详细地介绍它们。

## <font color="#9a161a">Event Time 事件时间</font>

Event time is an important topic to cover discretely because Spark’s DStream API does not support processing information with respect to event-time. At a higher level, in stream-processing systems there are effectively two relevant times for each event: the time at which it actually occurred (event time), and the time that it was processed or reached the stream-processing system (processing time).

事件时间是一个重要的主题，需要离散地涵盖，因为 Spark 的 DStream API 不支持与事件时间相关的处理信息。在更高的层次上，在流处理系统中，每个事件实际上有两个相关的时间：它实际发生的时间（事件时间），以及它被处理或到达流处理系统的时间（处理时间）。

**Event time 事件时间**

Event time is the time that is embedded in the data itself. It is most often, though not required to be, the time that an event actually occurs. This is important to use because it provides a more robust way of comparing events against one another. The challenge here is that event data can be late or out of order. This means that the stream processing system must be able to handle out-oforder or late data.

事件时间是嵌入到数据本身中的时间。它通常是事件实际发生的时间，尽管不要求如此。这一点很重要，因为它提供了一种更强大的方法来比较事件。这里的挑战是事件数据可能会延迟或无序。这意味着流处理系统必须能够处理无序或延迟的数据。

**Processing time 处理时间**

Processing time is the time at which the stream-processing system actually receives data. This is usually less important than event time because when it’s processed is largely an implementation detail. This can’t ever be out of order because it’s a property of the streaming system at a certain time (not an external system like event time). Those explanations are nice and abstract, so let’s use a more tangible example. Suppose that we have a datacenter located in San Francisco. An event occurs in two places at the same time: one in Ecuador, the other in Virginia (see Figure 22-1)。

处理时间是流处理系统实际接收数据的时间。这通常不如事件时间重要，因为处理时间在很大程度上是一个实现细节。这永远不会有问题，因为它是流系统在特定时间（而不是外部系统，如事件时间）的属性。这些解释既好又抽象，所以让我们用一个更具体的例子。假设我们有一个位于旧金山的数据中心。一个事件同时发生在两个地方：一个在厄瓜多尔，另一个在弗吉尼亚（见图22-1：遍布世界的事件时间）。

![1565188361752](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter22/1565188361752.png)

Due to the location of the datacenter, the event in Virginia is likely to show up in our datacenter before the event in Ecuador. If we were to analyze this data based on processing time, it would appear that the event in Virginia occurred before the event in Ecuador: something that we know to be wrong. However, if we were to analyze the data based on event time (largely ignoring the time at which it’s processed), we would see that these events occurred at the same time. 

由于数据中心的位置，弗吉尼亚州的事件可能会在厄瓜多尔的事件之前出现在我们的数据中心。如果我们根据处理时间来分析这些数据，弗吉尼亚州的事件似乎发生在厄瓜多尔事件之前：我们知道这是错误的。但是，如果我们基于事件时间分析数据（主要忽略处理数据的时间），我们将看到这些事件同时发生。

As we mentioned, the fundamental idea is that the order of the series of events in the processing system does not guarantee an ordering in event time. This can be somewhat unintuitive, but is worth reinforcing. Computer networks are unreliable. That means that events can be dropped, slowed down, repeated, or be sent without issue. Because individual events are not guaranteed to suffer one fate orthe other, we must acknowledge that any number of things can happen to these events on the way from the source of the information to our stream processing system. For this reason, we need to operate on event time and look at the overall stream with reference to this information contained in the data rather than on when it arrives in the system. This means that we hope to compare events based on the time at which those events occurred. 

正如我们所提到的，基本思想是处理系统中一系列事件的顺序并不保证事件时间的顺序。这可能有点不切实际，但值得加强。计算机网络不可靠。这意味着事件可以被删除、减速、重复或毫无问题地发送。由于个别事件不一定会遭受一种或另一种命运，我们必须承认，在从信息源到我们的流处理系统的过程中，这些事件可能会发生任何数量的事情。因此，我们需要在事件时间上操作，并参考数据中包含的信息查看整个流，而不是在数据到达系统时查看。这意味着我们希望根据这些事件发生的时间来比较事件。

## <font color="#9a161a">Stateful Processing</font>

The other topic we need to cover in this chapter is stateful processing. Actually, we already demonstrated this many times in Chapter 21. Stateful processing is only necessary when you need to use or update intermediate information (state) over longer periods of time (in either a microbatch or a record-at-a-time approach). This can happen when you are using event time or when you are performing an aggregation on a key, whether that involves event time or not.

在本章中，我们需要讨论的另一个主题是状态处理。实际上，我们已经在第21章中演示过很多次了。只有在需要使用或更新较长时间的中间信息（状态）时才需要状态处理（通过微批次或记录一次处理方法）。无论是否涉及事件时间，当使用事件时间或对键执行聚合时，都可能发生这种情况。

For the most part, when you’re performing stateful operations. Spark handles all of this complexity for you. For example, when you specify a grouping, Structured Streaming maintains and updates the information for you. You simply specify the logic. When performing a stateful operation, Spark stores the intermediate information in a state store. Spark’s current state store implementation is an inmemory state store that is made fault tolerant by storing intermediate state to the checkpoint directory. 

在大多数情况下，当您执行有状态操作时。Spark 为您处理所有这些复杂性。 例如，当您指定分组时，Structured Streaming 将为您维护和更新信息。您只需指定逻辑。在执行状态操作时，Spark 将中间信息存储在状态存储中。Spark当前的状态存储实现是一个 InMemory 状态存储，通过将中间状态存储到检查点（checkpoint ）目录中，使其具有容错性。

## <font color="#9a161a">Arbitrary Stateful Processing 任意（或自定义）状态处理</font>

The stateful processing capabilities described above are sufficient to solve many streaming problems. However, there are times when you need fine-grained control over what state should be stored, how it is updated, and when it should be removed, either explicitly or via a time-out. This is called arbitrary (or custom) stateful processing and Spark allows you to essentially store whatever information you like over the course of the processing of a stream. This provides immense flexibility and power and allows for some complex business logic to be handled quite easily. Just as we did before, let’s ground this with some examples : 

上面描述的状态处理功能足以解决许多流问题。但是，有时需要对什么状态应该存储、如何更新状态以及何时删除状态进行细粒度控制，无论是显式还是通过超时。这被称为任意（或自定义）状态处理，Spark 允许您在处理流的过程中基本上存储您喜欢的任何信息。这提供了巨大的灵活性和能力，并允许非常容易地处理一些复杂的业务逻辑。正如我们之前所做的，让我们用一些例子来说明这一点：

 You’d like to record information about user sessions on an ecommerce site. For instance, you might want to track what pages users visit over the course of this session in order to provide recommendations in real time during their next session. Naturally, these sessions have completely arbitrary start and stop times that are unique to that user.

您希望在电子商务网站上记录有关用户会话的信息。例如，您可能希望跟踪用户在本会话过程中访问的页面，以便在下次会话期间实时提供建议。当然，这些会话的开始和停止时间完全是该用户独有的。

- Your company would like to report on errors in the web application but only if five events occur during a user’s session. You could do this with count-based windows that only emit a result if five events of some type occur.

    您的公司希望报告Web应用程序中的错误，但前提是在用户会话期间发生五个事件。您可以对基于计数的窗口执行此操作，该窗口仅在发生某种类型的五个事件时发出结果。

- You’d like to deduplicate records over time. To do so, you’re going to need to keep track of every record that you see before deduplicating it.

    您希望随着时间的推移消除重复记录。要做到这一点，您需要在消除重复数据之前跟踪您看到的每个记录。

Now that we’ve explained the core concepts that we’re going to need in this chapter, let’s cover all of this with some examples that you can follow along with and explain some of the important caveats that you need to consider when processing in this manner. 

既然我们已经在本章中解释了我们将需要的核心概念，那么让我们用一些例子来涵盖所有这些，您可以跟随这些例子，并解释在以这种方式处理时需要考虑的一些重要注意事项。

## <font color="#9a161a">Event-Time Basics 事件时间基础知识</font>

Let’s begin with the same dataset from the previous chapter. When working with event time, it’s just another column in our dataset, and that’s really all we need to concern ourselves with; we simply use that column, as demonstrated here : 

让我们从上一章的相同数据集开始。在处理事件时间时，它只是数据集中的另一个列，而这正是我们需要关注的全部内容；我们只需使用该列，如下所示：

```scala
// in Scala
spark.conf.set("spark.sql.shuffle.partitions", 5)
val static = spark.read.json("/data/activity-data")
val streaming = spark
.readStream
.schema(static.schema)
.option("maxFilesPerTrigger", 10)
.json("/data/activity-data")
```

```python
# in Python
spark.conf.set("spark.sql.shuffle.partitions", 5)
static = spark.read.json("/data/activity-data")
streaming = spark\
.readStream\
.schema(static.schema)\
.option("maxFilesPerTrigger", 10)\
.json("/data/activity-data")
```

```scala
streaming.printSchema()
```

```shell
root
|-- Arrival_Time: long (nullable = true)
|-- Creation_Time: long (nullable = true)
|-- Device: string (nullable = true)
|-- Index: long (nullable = true)
|-- Model: string (nullable = true)
|-- User: string (nullable = true)
|-- gt: string (nullable = true)
|-- x: double (nullable = true)
|-- y: double (nullable = true)
|-- z: double (nullable = true)
```

In this dataset, there are two time-based columns. The Creation_Time column defines when an event was created, whereas the Arrival_Time defines when an event hit our servers somewhere upstream. We will use Creation_Time in this chapter. This example reads from a file but, as we saw in the previous chapter, it would be simple to change it to Kafka if you already have a cluster up and running.

在这个数据集中，有两个基于时间的列。Creation_Time 列定义事件的创建时间，而 Arrival_Time 则定义事件在上游某个位置在我们的服务器检索到的时间。我们将在本章中使用 Creation_Time 。这个例子是从一个文件中读取的，但是，正如我们在上一章中看到的，如果已经启动并运行了集群，那么将其更改为 Kafka 将非常简单。

## <font color="#9a161a">Windows on Event Time 基于事件时间的窗口</font>

The first step in event-time analysis is to convert the timestamp column into the proper Spark SQL timestamp type. Our current column is unixtime nanoseconds (represented as a long), therefore we’re going to have to do a little manipulation to get it into the proper format : 

事件时间分析的第一步是将 timestamp 列转换为正确的 Spark SQL timestamp 类型。我们当前的列是 unixtime纳秒（表示为长列），因此我们需要做一些操作才能将其转换为正确的格式：

```scala
// in Scala
val withEventTime = streaming.selectExpr(
    "*",
    "cast(cast(Creation_Time as double)/1000000000 as timestamp) as event_time"
)
```

```python
# in Python
withEventTime = streaming\.selectExpr(
    "*",
    "cast(cast(Creation_Time as double)/1000000000 as timestamp) as event_time"
)
```

### <font color="#000000">Tumbling Windows 滚动的窗口</font>

The simplest operation is simply to count the number of occurrences of an event in a given window. Figure 22-2 depicts the process when performing a simple summation based on the input data and a key.

最简单的操作就是计算给定窗口中事件的发生次数。图22-2，描述了基于输入数据和键，执行简单求和时的过程。

![1565255555140](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter22/1565255555140.png)

We’re performing an aggregation of keys over a window of time. We update the result table(depending on the output mode) when every trigger runs, which will operate on the data received since the last trigger. In the case of our actual dataset (and Figure 22-2), we’ll do so in 10-minute windows without any overlap between them (each, and only one event can fall into one window). This will update in real time, as well, meaning that if new events were being added upstream to our system, Structured Streaming would update those counts accordingly. This is the complete output mode, Spark will output the entire result table regardless of whether we’ve seen the entire dataset : 

我们在一段时间内执行 keys 聚合。当每个触发器运行时，我们更新结果表（取决于输出模式），它将对自上一个触发器以来接收的数据进行操作。在实际数据集（和图22-2）的情况下，我们将在10分钟的窗口中完成这项工作，它们之间没有任何重叠（每个窗口中只能有一个事件落在一个窗口中）。这也将实时更新，这意味着如果在系统上游添加新事件，Structured Streaming 将相应地更新这些计数。这是完整的输出模式，Spark 将输出整个结果表，而不管我们是否看到了整个数据集 ：

```scala
// in Scala
import org.apache.spark.sql.functions.{window, col}
withEventTime.groupBy(window(col("event_time"), "10 minutes")).count()
.writeStream
.queryName("events_per_window")
.format("memory")
.outputMode("complete")
.start()
```

```python
# in Python
from pyspark.sql.functions import window, col
withEventTime.groupBy(window(col("event_time"), "10 minutes")).count()\
.writeStream\
.queryName("events_per_window")\
.format("memory")\
.outputMode("complete")\
.start()
```

Now we’re writing out to the in-memory sink for debugging, so we can query it with SQL after we have the stream running : 

现在，我们正在写入内存中的接收器进行调试，以便在流运行后使用 SQL 查询它：

```scala
spark.sql("SELECT * FROM events_per_window").show()
```

```sql
SELECT * FROM events_per_window
```

This shows us something like the following result, depending on the amount of data processed when you had run the query : 

根据运行查询时处理的数据量，这将向我们显示如下结果 ：

```text
+---------------------------------------------+-----+
|                    window                   |count|
+---------------------------------------------+-----+
|[2015-02-23 10:40:00.0,2015-02-23 10:50:00.0]|11035|
|[2015-02-24 11:50:00.0,2015-02-24 12:00:00.0]|18854|
...
|[2015-02-23 13:40:00.0,2015-02-23 13:50:00.0]|20870|
|[2015-02-23 11:20:00.0,2015-02-23 11:30:00.0]|9392 |
+---------------------------------------------+-----+
```

For reference, here’s the schema we get from the previous query: 

为了参考，这里我们从前一个查询获得的模式（schema）：

```text
root
|-- window: struct (nullable = false)
| |-- start: timestamp (nullable = true)| |-- end: timestamp (nullable = true)
|-- count: long (nullable = false)
```

Notice how window is actually a <font face="constant-width" size=3>struct</font> (a complex type). Using this we can query this <font face="constant-width" size=3>struct</font> for the start and end times of a particular window. Of importance is the fact that we can also perform an aggregation on multiple columns, including the event time column. Just like we saw in the previous chapter, we can even perform these aggregations using methods like cube. While we won’t repeat the fact that we can perform the multi-key aggregation below, this does apply to any window-style aggregation (or stateful computation) we would like : 

注意窗口实际上是一个结构（复杂类型）。使用这个，我们可以查询这个结构来获取特定窗口的开始和结束时间。重要的是，我们还可以对多个列（包括事件时间列）执行聚合。正如我们在上一章中看到的，我们甚至可以使用类似于 Cube 的方法来执行这些聚合。虽然我们不会重复这样一个事实，即我们可以执行下面的多键聚合，但这确实适用于我们想要的任何窗口式聚合（或状态计算）：

```scala
// in Scala
import org.apache.spark.sql.functions.{window, col}
withEventTime.groupBy(window(col("event_time"), "10 minutes"), "User").count()
.writeStream
.queryName("events_per_window")
.format("memory")
.outputMode("complete")
.start()
```

```python
# in Python
from pyspark.sql.functions import window, col
withEventTime.groupBy(window(col("event_time"), "10 minutes"), "User").count()\
.writeStream\
.queryName("pyevents_per_window")\
.format("memory")\
.outputMode("complete")\
.start()
```

#### <font color="0a7ae5">Sliding windows 滑动窗口</font>

The previous example was simple counts in a given window. Another approach is that we can decouple the window from the starting time of the window. Figure 22-3 illustrates what we mean.

前一个例子是给定窗口中的简单计数。另一种方法是我们可以将窗口与窗口的开始时间分离。图22-3说明了我们的意思。

![1565276939426](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter22/1565276939426.png)

In the figure, we are running a sliding window through which we look at an hour increment, but we’d like to get the state every 10 minutes. This means that we will update the values over time and will include the last hours of data. In this example, we have 10-minute windows, starting every five minutes. Therefore each event will fall into two different windows. You can tweak this further according to your needs:

在图中，我们运行一个滑动窗口，通过它我们可以看到一个小时增量，但是我们希望每10分钟获得一个状态。这意味着我们将随着时间的推移更新这些值，并包括最后几个小时的数据。在这个例子中，我们有10分钟的窗口，每5分钟启动一次。因此，每个事件将分为两个不同的窗口。您可以根据需要进一步调整：

```scala
// in Scala
import org.apache.spark.sql.functions.{window, col}
withEventTime.groupBy(window(col("event_time"), "10 minutes", "5 minutes"))
.count()
.writeStream()
.queryName("events_per_window")
.format("memory")
.outputMode("complete")
.start()
```

```python
# in Python
from pyspark.sql.functions import window, col
withEventTime.groupBy(window(col("event_time"), "10 minutes", "5 minutes"))\
.count()\
.writeStream\
.queryName("pyevents_per_window")\
.format("memory")\
.outputMode("complete")\
.start()
```

Naturally, we can query the in-memory table:

当然，我们可以查询内存中的表：

```sql
SELECT * FROM events_per_window
```

This query gives us the following result. Note that the starting times for each window are now in 5-minute intervals instead of 10, like we saw in the previous query:

此查询提供以下结果。注意，现在每个窗口的开始时间间隔为5分钟，而不是10分钟，就像我们在前面的查询中看到的那样：

```txt
+---------------------------------------------+-----+
|                  window                     |count|
+---------------------------------------------+-----+
|[2015-02-23 14:15:00.0,2015-02-23 14:25:00.0]|40375|
|[2015-02-24 11:50:00.0,2015-02-24 12:00:00.0]|56549|
...
|[2015-02-24 11:45:00.0,2015-02-24 11:55:00.0]|51898|
|[2015-02-23 10:40:00.0,2015-02-23 10:50:00.0]|33200|
+---------------------------------------------+-----+
```

### <font color="#000000">Handling Late Data with Watermarks 使用水印处理延迟数据</font>

The preceding examples are great, but they have a flaw. We never specified how late we expect to see data. This means that Spark is going to need to store that intermediate data forever because we never specified a watermark, or a time at which we don’t expect to see any more data. This applies to all stateful processing that operates on event time. We must specify this watermark in order to age-out data in the stream (and, therefore, state) so that we don’t overwhelm the system over a long period of time.

前面的例子很好，但是它们有一个缺陷。我们从未具体说明我们期望看到数据的时间有多晚。这意味着 Spark 需要永远存储中间数据，因为我们从未指定过水印，或者我们不希望看到更多数据的时间。这适用于对事件时间进行操作的所有状态处理。我们必须指定这个水印，以便消除流中的数据（因此，状态），以便在很长一段时间内不会压垮系统。

Concretely, a watermark is an amount of time following a given event or set of events after which we do not expect to see any more data from that time. We know this can happen due to delays on the network, devices that lose a connection, or any number of other issues. In the DStreams API, there was no robust way to handle late data in this way—if an event occurred at a certain time but did not make it to the processing system by the time the batch for a given window started, it would show up in other processing batches. Structured Streaming remedies this. In event time and stateful processing, a given window’s state or set of data is decoupled from a processing window. That means that as more events come in, Structured Streaming will continue to update a window with more information.

具体来说，水印是给定事件或一组事件之后的一段时间，在此时间之后，我们不希望看到更多的数据。我们知道这可能是由于网络延迟、设备断开连接或任何其他问题造成的。在 DStreams API 中，如果某个事件在某个时间发生，但在某个给定窗口的批处理开始时未到达处理系统，则没有可靠的方法以这种方式处理延迟数据，它将显示在其他批处理中。Structured Streaming 补救了这点。在事件时间和状态处理中，给定窗口的状态或数据集与处理窗口分离。这意味着，随着更多事件的到来，Structured Streaming 将继续更新包含更多信息的窗口。

Let’s return back to our event time example from the beginning of the chapter, shown now in Figure 22-4

让我们回到本章开头的事件时间示例，如图22-4所示。

![1565279336906](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter22/1565279336906.png)

In this example, let’s imagine that we frequently see some amount of delay from our customers in Latin America. Therefore, we specify a watermark of 10 minutes. When doing this, we instruct Spark that any event that occurs more than 10 “event-time” minutes past a previous event should be ignored. Conversely, this also states that we expect to see every event within 10 minutes. After that, Spark should remove intermediate state and, depending on the output mode, do something with the result. As mentioned at the beginning of the chapter, we need to specify watermarks because if we did not, we’d need to keep all of our windows around forever, expecting them to be updated forever. This brings us to the core question when working with event-time: “how late do I expect to see data?” The answer to this question will be the watermark that you’ll configure for your data.

在这个例子中，假设我们经常看到拉丁美洲客户的延迟。因此，我们指定10分钟的水印。在执行此操作时，我们指示 Spark，应忽略发生在上一个事件之后超过10“事件时间”分钟的任何事件。相反，这也说明我们期望在10分钟内看到每个事件。之后，Spark 应消除中间状态，并根据输出模式对结果进行处理。正如本章开头所提到的，我们需要指定水印，因为如果没有，我们将需要永远保留所有窗口，期待它们永远更新。在处理事件时间时，这就给我们带来了一个核心问题：“我希望看到数据有多晚？“此问题的答案将是为数据配置的水印。

Returning to our dataset, if we know that we typically see data as produced downstream in minutes but we have seen delays in events up to five hours after they occur (perhaps the user lost cell phone connectivity), we’d specify the watermark in the following way : 

返回到我们的数据集，如果我们知道我们通常在数分钟内看到下游生成的数据，但我们已经看到事件发生后长达5小时的延迟（可能是用户丢失了手机连接），我们将按以下方式指定水印 ：

```scala
// in Scala
import org.apache.spark.sql.functions.{window, col}

withEventTime
.withWatermark("event_time", "5 hours")
.groupBy(window(col("event_time"), "10 minutes", "5 minutes"))
.count()
.writeStream
.queryName("events_per_window")
.format("memory")
.outputMode("complete")
.start()
```

```python
# in Python
from pyspark.sql.functions import window, col

withEventTime\
.withWatermark("event_time", "30 minutes")\
.groupBy(window(col("event_time"), "10 minutes", "5 minutes"))\
.count()\
.writeStream\
.queryName("pyevents_per_window")\
.format("memory")\
.outputMode("complete")\
.start()
```

It’s pretty amazing, but almost nothing changed about our query. We essentially just added another configuration. Now, Structured Streaming will wait until 30 minutes after the final timestamp of this 10-minute rolling window before it finalizes the result of that window. We can query our table and see the intermediate results because we’re using complete mode—they’ll be updated over time. In append mode, this information won’t be output until the window closes.

这很神奇，但我们的查询几乎没有任何变化。我们实际上只是添加了另一个配置。现在，Structured Streaming 将在这个10分钟滚动窗口的最后时间戳之后等待30分钟，然后最终确定该窗口的结果。我们可以查询表并查看中间结果，因为我们使用的是完整模式，它们将随着时间的推移而更新。在附加模式下，在窗口关闭之前不会输出此信息。

```sql
SELECT * FROM events_per_window
```

```text
+---------------------------------------------+-----+
|                    window                   |count|
+---------------------------------------------+-----+
|[2015-02-23 14:15:00.0,2015-02-23 14:25:00.0]|9505 |
|[2015-02-24 11:50:00.0,2015-02-24 12:00:00.0]|13159|
...
|[2015-02-24 11:45:00.0,2015-02-24 11:55:00.0]|12021|
|[2015-02-23 10:40:00.0,2015-02-23 10:50:00.0]|7685 |
+---------------------------------------------+-----+
```

At this point, you really know all that you need to know about handling late data. Spark does all of the heavy lifting for you. Just to reinforce the point, if you do not specify how late you think you will see data, then Spark will maintain that data in memory forever. Specifying a watermark allows it to free those objects from memory, allowing your stream to continue running for a long time.

此时，您真正了解处理延迟数据所需的所有信息。Spark 为你做了所有的重担。为了强调这一点，如果您不指定您认为看到数据的时间有多晚，那么 Spark 将永远在内存中维护该数据。指定水印允许它从内存中释放这些对象，从而允许流长时间继续运行。

## <font color="#9a161a">Dropping Duplicates in a Stream 在流中删除重复项</font>

One of the more difficult operations in record-at-a-time systems is removing duplicates from the stream. Almost by definition, you must operate on a batch of records at a time in order to find duplicates—there’s a high coordination overhead in the processing system. Deduplication is an important tool in many applications, especially when messages might be delivered multiple times by upstream systems. A perfect example of this are Internet of Things (IoT) applications that have upstream producers generating messages in nonstable network environments, and the same message might end up being sent multiple times. Your downstream applications and aggregations should be able to assume that there is only one of each message. 

一次记录系统中比较困难的操作之一是从流中删除重复项。几乎按照定义，您必须一次对一批记录进行操作，以便找到重复的记录。处理系统中的协调开销很高。在许多应用程序中，重复数据消除是一个重要的工具，特别是当消息可能被上游系统多次传递时。一个很好的例子是物联网（IOT）应用程序，它让上游生产者在不稳定的网络环境中生成消息，而相同的消息最终可能会被多次发送。下游应用程序和聚合应该能够假定每条消息只有一条。

Essentially, Structured Streaming makes it easy to take message systems that provide at-least-once semantics, and convert them into exactly-once by dropping duplicate messages as they come in, based on arbitrary keys. To de-duplicate data, Spark will maintain a number of user specified keys and ensure that duplicates are ignored.

从本质上讲，Structured Streaming 使获取至少提供一次语义的消息系统变得容易，并根据任意键删除重复的消息从而将它们转换为恰好一次的语义。为了消除重复数据，Spark 将维护许多用户指定的 keys，并确保忽略重复的keys。

---

<center><font face="constant-width" color="#c67171" size=4><strong>WARNING</strong></font></center>
Just like other stateful processing applications, you need to specify a watermark to ensure that the maintained state does not grow infinitely over the course of your stream.

与其他状态处理应用程序一样，您需要指定一个水印，以确保维护状态不会在流过程中无限增长。

---

Let’s begin the de-duplication process. The goal here will be to de-duplicate the number of events per user by removing duplicate events. Notice how you need to specify the event time column as a duplicate column along with the column you should de-duplicate. The core assumption is that duplicate events will have the same timestamp as well as identifier. In this model, rows with two different timestamps are two different records:

让我们开始重复数据消除过程。这里的目标是通过删除重复事件来消除每个用户的事件数。请注意，您需要如何将事件时间列指定为重复的列以及应消除重复的列。核心假设是重复事件将具有相同的时间戳和标识符。在此模型中，具有两个不同时间戳的行是两个不同的记录：

```scala
// in Scala
import org.apache.spark.sql.functions.expr
withEventTime
.withWatermark("event_time", "5 seconds")
.dropDuplicates("User", "event_time")
.groupBy("User")
.count()
.writeStream
.queryName("deduplicated")
.format("memory")
.outputMode("complete")
.start()
```

```python
# in Python
from pyspark.sql.functions import expr
withEventTime\
.withWatermark("event_time", "5 seconds")\
.dropDuplicates(["User", "event_time"])\.groupBy("User")\
.count()\
.writeStream\
.queryName("pydeduplicated")\
.format("memory")\
.outputMode("complete")\
.start()
```

The result will be similar to the following and will continue to update over time as more data is read by your stream:

结果将类似于以下内容，随着流读取更多数据，结果将继续更新：

```txt
+----+-----+
|User|count|
+----+-----+
|  a | 8085|
|  b | 9123|
|  c | 7715|
|  g | 9167|
|  h | 7733|
|  e | 9891|
|  f | 9206|
|  d | 8124|
|  i | 9255|
+----+-----+
```

## <font color="#9a161a">Arbitrary Stateful Processing 任意状态处理</font>

The first section if this chapter demonstrates how Spark maintains information and updates windows based on our specifications. But things differ when you have more complex concepts of windows; this is, where arbitrary stateful processing comes in. This section includes several examples of different use cases along with examples that show you how you might go about setting up your business logic. Stateful processing is available only in Scala in Spark 2.2. This will likely change in the future.

第一节如果这一章演示 Spark 如何维护信息和更新基于我们的规范的窗口。但是，当您对窗口有更复杂的概念时，情况就不同了；这就是任意状态处理的出现之处。本节包括几个不同用户案例以及一些示例，这些示例向您展示了如何设置业务逻辑。状态处理仅在 Spark 2.3 中的 Scala 中可用。这在将来可能会改变。

When performing stateful processing, you might want to do the following:

在执行状态处理时，您可能需要执行以下操作：

- Create window based on counts of a given key
基于给定键的计数创建窗口。

- Emit an alert if there is a number of events within a certain time frame 
如果在某个时间范围内有多个事件，则发出警报。

- Maintain user sessions of an undetermined amount of time and save those sessions to perform some analysis on later.
保持不确定多长时间的用户会话，并保存这些会话以便稍后执行一些分析。

At the end of the day, there are two things you will want to do when performing this style of processing:

一天结束时，在执行这种类型的处理时，有两件事要做：

- Map over groups in your data, operate on each group of data, and generate at most a single row for each group. The relevant API for this use case is <font face="constant-width" size=3>mapGroupsWithState</font>.

    映射数据中的组，对每组数据进行操作，并为每组最多生成一行。此用户案例的相关API是`MapGroupsWithState` 。

- Map over groups in your data, operate on each group of data, and generate one or more rows for each group. The relevant API for this use case is <font face="constant-width"  size=3>flatMapGroupsWithState</font>.

    映射数据中的组，对每组数据进行操作，并为每组生成一行或多行。此用户案例的相关API是`FlatmapGroupsWithState` 。

When we say “operate” on each group of data, that means that you can arbitrarily update each group independent of any other group of data. This means that you can define arbitrary window types that don’t conform to tumbling or sliding windows like we saw previously in the chapter. One important benefit that we get when we perform this style of processing is control over configuring time-outs on state. With windows and watermarks, it’s very simple: you simply time-out a window when the watermark passes the window start. This doesn’t apply to arbitrary stateful processing, because you manage the state based on user-defined concepts. Therefore, you need to properly time-out your state. 

当我们对每一组数据说“操作”时，这意味着您可以独立于任何其他数据组任意更新每一组数据。这意味着您可以定义不符合滚动或滑动窗口的任意窗口类型，正如我们在本章前面看到的那样。当我们执行这种类型的处理时，我们得到的一个重要好处是对状态的超时配置进行控制。有窗口和水印，这非常简单：当水印（设置的时间）超过窗口时，只需超时即可开始。这不适用于任意状态处理，因为您是基于用户定义的概念来管理状态的。因此，您需要适当地超时您的状态。

Let’s discuss this a bit more.

我们再讨论一下。

### <font color="#000000">Time-Outs</font>

As mentioned in Chapter 21, a time-out specifies how long you should wait before timing-out some intermediate state. A time-out is a global parameter across all groups that is configured on a pergroup basis. Time-outs can be either based on processing time (<font face="constant-width" size=3>GroupStateTimeout.ProcessingTimeTimeout</font>) or event time (<font face="constant-width" size=3>GroupStateTimeout.EventTimeTimeout</font>). When using time-outs, check for time-out first before processing the values. You can get this information by checking the <font face="constant-width" size=3>state.hasTimedOut</font> flag or checking whether the values iterator is empty. You need to set some state (i.e., state must be defined, not removed) for time-outs to be set. With a time-out based on processing time, you can set the time-out duration by calling GroupState.setTimeoutDuration (we’ll see code examples of this later in this section of the chapter). The time-out will occur when the clock has advanced by the set duration. Guarantees provided by this time-out with a duration of D ms are as follows : 

如第21章所述，超时指定在超时某个中间状态之前应等待多长时间。超时是在每个组基础上配置的所有组的全局参数。超时可以基于处理时间（`GroupStateTimeout.ProcessingTimeTimeTimeout`）或事件时间（`GroupStateTimeout.EventTimeTimeTimeTimeout`）。使用超时时，请先检查超时，然后再处理值。您可以通过检查 `state.hasTimedOut`  标志或检查值迭代器是否为空来获取此信息。您需要设置一些状态（即必须定义状态，而不是删除状态），以便设置超时。使用基于处理时间的超时，可以通过调用`GroupState.SetTimeOutDuration` 来设置超时的持续时间（我们将在本章后面的部分中看到这方面的代码示例）。当时钟提前到设定的持续时间时，将发生超时。持续时间为 d ms 的超时提供的保证如下：

Time-out will never occur before the clock time has advanced by D ms。

在时钟时间提前 d ms 之前，不会发生超时。

Time-out will occur eventually when there is a trigger in the query (i.e., after D ms). So there is a no strict upper bound on when the time-out would occur. For example, the trigger interval of the query will affect when the time-out actually occurs. If there is no data in the stream (for any group) for a while, there won’t be any trigger and the time-out function call will not occur until there is data.

当查询中存在触发器时（即，在 d ms 之后），最终会发生超时。所以在什么时候会发生超时没有严格的上限。例如，查询的触发器间隔将影响什么时候实际发生超时。如果流中（对于任何组）有一段时间没有数据，则不会有任何触发器，并且在有数据之前不会发生超时函数调用。

Because the processing time time-out is based on the clock time, it is affected by the variations in the system clock. This means that time zone changes and clock skew are important variables to consider. 

由于处理超时基于时钟时间，因此它受系统时钟变化的影响。这意味着时区变化和时钟偏移是要考虑的重要变量。

With a time-out based on event time, the user also must specify the event-time watermark in the query using watermarks. When set, data older than the watermark is filtered out. As the developer, you can set the timestamp that the watermark should reference by setting a time-out timestamp using the <font face="constant-width" size=3>GroupState.setTimeoutTimestamp(...) API</font>. The time-out would occur when the watermark advances beyond the set timestamp. Naturally, you can control the time-out delay by either specifying longer watermarks or simply updating the time-out as you process your stream. Because you can do this in arbitrary code, you can do it on a per-group basis. The guarantee provided by this time-out is that it will never occur before the watermark has exceeded the set time-out. 

对于基于事件时间的超时，用户还必须使用水印在查询中指定事件时间水印。设置后，将过滤掉比水印早的数据。作为开发人员，可以通过使用 `Groupstate.setTimeoutTimestamp(…)` API 设置超时时间戳来设置水印应引用的时间戳。当水印超过设置的时间戳时，将发生超时。当然，您可以通过指定更长的水印或在处理流时简单地更新超时来控制超时延迟。因为您可以在任意代码中执行此操作，所以可以在每个组的基础上执行此操作。此超时提供的保证是，在水印超过设置的超时之前，它将永远不会发生。

Similar to processing-time time-outs, there is a no strict upper bound on the delay when the time-out actually occurs. The watermark can advance only when there is data in the stream, and the event time of the data has actually advanced.

与处理时间（processing-time）超时类似，实际发生超时时，延迟没有严格的上限。只有当流中有数据，并且数据的事件时间实际提前时，水印才能前进。

---

<center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong></font></center>
We mentioned this a few moments ago, but it’s worth reinforcing. Although time-outs are important, they might not always function as you expect. For instance, as of this writing, Structured Streaming does not have asynchronous job execution, which means that Spark will not output data (or time-out data) between the time that a epoch finishes and the next one starts, because it is not processing any data at that time. Also, if a processing batch of data has no records (keep in mind this is a batch, not a group), there are no updates and there cannot be an event-time time-out. This might change in future versions.

我们几分钟前提到过，但值得加强。尽管超时很重要，但它们可能并不总是如您所期望的那样工作。例如，在撰写本文时，Structured Streaming 没有异步作业执行，这意味着 Spark 不会在一个 epoch 完成和下一个 epoch 开始之间输出数据（或超时数据），因为它当时不处理任何数据。此外，如果一批处理数据没有记录（请记住，这是一批数据，而不是一个组），则不会有更新，也不会有事件超时。这在未来的版本中可能会改变。

---

### <font color="#000000">Output Modes</font>

One last “gotcha” when working with this sort of arbitrary stateful processing is the fact that not all output modes discussed in Chapter 21 are supported. This is sure to change as Spark continues to change, but, as of this writing, <font face="constant-width" color="##000000" size=3>mapGroupsWithState</font> supports only the update output mode, whereas <font face="constant-width" color="#000000" size=3>flatMapGroupsWithState</font> supports <font face="constant-width" color="#000000" size=3>append</font> and <font face="constant-width" color="#000000" size=3>update</font>. append mode means that only after the time-out (meaning the watermark has passed) will data show up in the result set. This does not happen automatically, it is your responsibility to output the proper row or rows. Please see Table 21-1 to see which output modes can be used when.

在处理这种任意状态处理时，最后一个“发现”是，并非第21章中讨论的所有输出模式都受支持。这肯定会随着 Spark 的不断变化而改变，但是，在本文中，mapGroupsWithState 只支持更新输出模式，而flatMapGroupsWithState  支持 append 和 update。附加（append）模式意味着只有在超时（即水印已通过）之后，数据才会显示在结果集中。这不会自动发生，您有责任输出正确的行。请参阅表21-1，了解在什么情况下可以使用哪些输出模式。

### <font color="#000000">mapGroupsWithState</font>

Our first example of stateful processing uses a feature called <font face="constant-width" color="#000000" size=3>mapGroupsWithState</font>. This is similar to a user-defined aggregation function that takes as input an update set of data and then resolves it down to a specific key with a set of values. There are several things you’re going to need to define along the way : 

我们的第一个状态处理示例使用了一个名为 mapGroupsWithState 的特性。这类似于一个用户定义的聚合函数，它将一组更新数据作为输入，然后用一组值将其解析为一个特定的键。 在这一过程中，您需要定义以下几个方面：

- Three class definitions: an input definition, a state definition, and optionally an output definition.

    三个类定义：输入定义、状态定义和输出定义（可选）。

- A function to update the state based on a key, an iterator of events, and a previous state.

    基于键、事件迭代器和前一状态更新状态的函数 。

- A time-out parameter (as described in the time-outs section).

    超时参数（如超时部分所述）。

With these objects and definitions, you can control arbitrary state by creating it, updating it over time, and removing it. Let’s begin with a example of simply updating the key based on a certain amount of state, and then move onto more complex things like sessionization. 

通过这些对象和定义，您可以通过创建、随时间更新和删除任意状态来控制它。让我们从一个简单的基于一定数量状态更新键的示例开始，然后转到更复杂的事情，比如会话化。

Because we’re working with sensor data, let’s find the first and last timestamp that a given user performed one of the activities in the dataset. This means that the key we will be grouping on (and mapping on) is a user and activity combination. 

因为我们正在处理传感器数据，所以让我们找到给定用户执行数据集中某个活动的第一个和最后一个时间戳。这意味着我们将分组（和映射）的key 是用户和活动的组合。

---------

<center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong></center></font></center>
When you use `mapGroupsWithState`, the output of the dream will contain only one row per key (or group) at all times. If you would like each group to have multiple outputs, you should use `flatMapGroupsWithState` (covered shortly). 

当您使用 `mapGroupsWithState` 时，梦想的输出将始终只包含每个键（或组）一行。如果希望每个组有多个输出，则应使用 `flatmapGroupsWithState`（稍后介绍）。

---

Let’s establish the input, state, and output definitions:

让我们建立输入、状态和输出定义 ： 

```scala
case class InputRow(user:String, timestamp:java.sql.Timestamp, activity:String)
case class UserState(user:String,
var activity:String,
var start:java.sql.Timestamp,
var end:java.sql.Timestamp)
```

For readability, set up the function that defines how you will update your state based on a given row:

为了便于阅读，请设置一个函数，该函数定义如何根据给定的行更新状态：

```scala
def updateUserStateWithEvent(state:UserState, input:InputRow):UserState = {
    if (Option(input.timestamp).isEmpty) {
        return state
    } 
    if (state.activity == input.activity) {
        if (input.timestamp.after(state.end)) {
         state.end = input.timestamp
        } 
        if (input.timestamp.before(state.start)) {
         state.start = input.timestamp
        }
    } else {
        if (input.timestamp.after(state.end)) {
            state.start = input.timestamp
            state.end = input.timestamp
            state.activity = input.activity
      }
    } 
    state
}
```

Now, write the function that defines the way state is updated based on an epoch of rows:

现在，编写一个函数，该函数定义了基于行的 epoch 更新状态的方式：

```scala
import org.apache.spark.sql.streaming.{GroupStateTimeout, OutputMode, GroupState}

def updateAcrossEvents(user:String,
	inputs: Iterator[InputRow],
	oldState: GroupState[UserState]):UserState = {
	var state:UserState = if (oldState.exists) oldState.get else UserState(user,
		"",
		new java.sql.Timestamp(6284160000000L),
		new java.sql.Timestamp(6284160L)
    )
	// we simply specify an old date that we can compare against and
	// immediately update based on the values in our data
	for (input <- inputs) {
		state = updateUserStateWithEvent(state, input)
		oldState.update(state)
	} 
    state
}
```

When we have that, it’s time to start your query by passing in the relevant information. The one thing that you’re going to have to add when you specify `mapGroupsWithState` is whether you need to timeout a given group’s state. This just gives you a mechanism to control what should be done with state that receives no update after a certain amount of time. In this case, you want to maintain state indefinitely, so specify that Spark should not time-out. Use the update output mode so that you get updates on the user activity:

当我们得到这些信息时，是时候通过传递相关信息来开始您的查询了。当您指定 `mapGroupsWithState` 时，您需要添加的一件事是您是否需要使给定组的状态超时。这只是为您提供了一种机制，用于控制在一定时间后不接收任何更新的状态应执行的操作。在这种情况下，您希望无限期地保持状态，因此请指定 Spark 不应超时。使用更新输出模式，以便获得用户活动的更新：

```scala
import org.apache.spark.sql.streaming.GroupStateTimeout

withEventTime.
selectExpr("User as user",
	"cast(Creation_Time/1000000000 as timestamp) as timestamp", "gt as activity").
as[InputRow].
groupByKey(_.user).
mapGroupsWithState(GroupStateTimeout.NoTimeout)(updateAcrossEvents).
writeStream.
queryName("events_per_window").
format("memory").
outputMode("update").
start()

SELECT * FROM events_per_window order by user, start
```

Here’s a sample of our result set:

以下是我们的结果集示例：

```shell
+----+--------+--------------------+--------------------+
|user|activity|        start       |        end         |
+----+--------+--------------------+--------------------+
| a  | bike   |2015-02-23 13:30:...|2015-02-23 14:06:...|
| a  | bike   |2015-02-23 13:30:...|2015-02-23 14:06:...|
 ...
| d  | bike   |2015-02-24 13:07:...|2015-02-24 13:42:...|
+----+--------+--------------------+--------------------+
```

An interesting aspect of our data is that the last activity performed at any given time is “bike.” This is related to how the experiment was likely run, in which they had each participant perform the same activities in order。

我们数据的一个有趣的方面是，在任何给定时间最后一次执行的活动是“自行车”。这与实验可能的运行方式有关，其中每个参与者依次执行相同的活动。

### <font color="#000000">EXAMPLE: COUNT-BASED WINDOWS</font>

Typical window operations are built from start and end times for which all events that fall in between those two points contribute to the counting or summation that you’re performing. However, there are times when instead of creating windows based on time, you’d rather create them based on a number of events regardless of state and event times, and perform some aggregation on that window of data. For example, we may want to compute a value for every 500 events received, regardless of when they are received. The next example analyzes the activity dataset from this chapter and outputs the average reading of each device periodically, creating a window based on the count of events and outputting it each time it has accumulated 500 events for that device. You define two case classes for this task: the input row format (which is simply a device and a timestamp); and the state and output rows (which contain the current count of records collected, device ID, and an array of readings for the events in the window).

典型的窗口操作是从开始和结束时间开始构建的，在这两个时间点之间的所有事件都有助于您正在执行的计数或求和。但是，有时您不希望基于时间创建窗口，而是基于许多事件创建窗口，而不管状态和事件时间如何，并对该数据窗口执行一些聚合。例如，我们可能希望为每接收500个事件计算一个值，而不管它们何时被接收。下一个示例分析本章中的活动数据集，定期输出每个设备的平均读数，根据事件计数创建一个窗口，并在该设备累计500个事件时输出该窗口。为此任务定义两个案例类（case class）：输入行格式（简单地说是一个设备和一个时间戳）；状态行和输出行（包含收集的记录的当前计数、设备ID和窗口中事件的读取数组）。

Here are our various, self-describing case class definitions :  

下面是我们的各种自我描述的案例类定义：

```scala
case class InputRow(device: String, timestamp: java.sql.Timestamp, x : Double)
case class DeviceState(device: String, var values: Array[Double], var count: Int)
case class OutputRow(device: String, previousAverage: Double)
```

Now, you can define the function to update the individual state based on a single input row. You could write this inline or in a number of other ways, but this example makes it easy to see exactly how you update based on a given row : 

现在，您可以定义函数来更新基于单个输入行的单个状态。您可以以内联或其他多种方式编写此代码，但此示例使您很容易确切地了解如何根据给定行进行更新：

```scala
def updateWithEvent(state:DeviceState, input:InputRow):DeviceState = {
	state.count += 1
	// maintain an array of the x-axis values
	state.values = state.values ++ Array(input.x)
	state
}
```

Now it’s time to define the function that updates across a series of input rows. Notice in the example that follows that we have a specific key, the iterator of inputs, and the old state, and we update that old state over time as we receive new events. This, in turn, will return our output rows with the updates on a per-device level based on the number of counts it sees. This case is quite straightforward, after a given number of events, you update the state and reset it. You then create an output row. You can see this row in the output table : 

现在是定义跨一系列输入行更新的函数的时候了。注意在下面的示例中，我们有一个特定的键、输入的迭代器和旧状态，在接收新事件时，我们会随着时间的推移更新旧状态。反过来，这将返回我们的输出行，并根据它看到的计数数在每个设备级别上进行更新。这种情况非常简单，在给定数量的事件之后，您将更新状态并重置它。然后创建一个输出行。您可以在输出表中看到此行：

```scala
import org.apache.spark.sql.streaming.{GroupStateTimeout, OutputMode,
GroupState}
def updateAcrossEvents(device:String, inputs: Iterator[InputRow], oldState : GroupState[DeviceState]) : Iterator[OutputRow] = {
    inputs.toSeq.sortBy(_.timestamp.getTime).toIterator.flatMap { input =>
        val state = if (oldState.exists) oldState.get else DeviceState(device, Array(), 0)
        val newState = updateWithEvent(state, input)
        if (newState.count >= 500) {
            // One of our windows is complete; replace our state with an empty
            // DeviceState and output the average for the past 500 items from
            // the old state
            oldState.update(DeviceState(device, Array(), 0))
            Iterator(OutputRow(device,
            newState.values.sum / newState.values.length.toDouble))
        } else {
            // Update the current DeviceState object in place and output no
            // records
            oldState.update(newState)
            Iterator()
        }
    }
}
```

Now you can run your stream. You will notice that you need to explicitly state the output mode, which is append. You also need to set a `GroupStateTimeout`. This time-out specifies the amount of time you want to wait before a window should be output as complete (even if it did not reach the required count). In that case, set an infinite time-out, meaning if a device never gets to that required 500 count threshold, it will maintain that state forever as “incomplete” and not output it to the result table. By specifying both of those parameters you can pass in the `updateAcrossEvents` function and start the stream:

现在你可以运行你的流。您将注意到您需要显式地声明输出模式，即 append。还需要设置 `GroupStateTimeout`。此超时指定在窗口输出完成之前要等待的时间量（即使它未达到所需的计数）。在这种情况下，设置一个无限的超时，这意味着如果一个设备永远无法达到所需的500计数阈值，它将永远保持该状态为“不完整”，而不会将其输出到结果表。通过指定这两个参数，可以在 `updateAcrossEvents` 函数中传递并启动流：

```scala
import org.apache.spark.sql.streaming.GroupStateTimeout
withEventTime
.selectExpr("Device as device",
"cast(Creation_Time/1000000000 as timestamp) as timestamp", "x")
.as[InputRow]
.groupByKey(_.device)
.flatMapGroupsWithState(OutputMode.Append, GroupStateTimeout.NoTimeout)(updateAcrossEvents)
.writeStream
.queryName("count_based_device")
.format("memory")
.outputMode("append")
.start()
```

After you start the stream, it’s time to query it. Here are the results:

启动流之后，是时候查询它了。结果如下：

```sql
SELECT * FROM count_based_device
```

```shell
+--------+--------------------+
| device |   previousAverage  |
+--------+--------------------+
|nexus4_1|   4.660034012E-4   |
|nexus4_1|0.001436279298199...|
|nexus4_1|1.049804683999999...|
|nexus4_1|-0.01837188737960...|
+--------+--------------------+
```

You can see the values change over each of those windows as you append new data to the result set.

当您向结果集附加新数据时，可以看到这些窗口中的每个窗口的值都发生了变化。

### <font color="#000000">flatMapGroupsWithState</font>

Our second example of stateful processing will use a feature called flatMapGroupsWithState. This is quite similar to mapGroupsWithState except that rather than just having a single key with at most one output, a single key can have many outputs. This can provide us a bit more flexibility and the same fundamental structure as mapGroupsWithState applies. Here’s what we’ll need to define. 

我们的第二个状态处理示例将使用名为 `flatmapGroupsWithState` 的功能。这与 `mapGroupsWithState` 非常相似，只是一个键最多只能有一个输出，而不是只有一个键可以有多个输出。这可以为我们提供更多的灵活性和与`mapGroupsWithState` 应用相同的基本结构。以下是我们需要定义的内容。

- Three class definitions: an input definition, a state definition, and optionally an output definition.
三个类定义：输入定义、状态定义和输出定义（可选）。
- A function to update the state based on a key, an iterator of events, and a previous state.
基于键、事件迭代器和前一状态更新状态的函数。
- A time-out parameter (as described in the time-outs section).
超时参数（如超时部分所述）。

With these objects and definitions, we can control arbitrary state by creating it, updating it over time, and removing it. Let’s start with an example of sessionization. 

通过这些对象和定义，我们可以通过创建、随时间更新和删除任意状态来控制它。让我们从会话化的例子开始。

### <font color="#000000">EXAMPLE: SESSIONIZATION</font>

Sessions are simply unspecified time windows with a series of events that occur. Typically, you want to record these different events in an array in order to compare these sessions to other sessions in the future. In a session, you will likely have arbitrary logic to maintain and update your state over time as well as certain actions to define when state ends (like a count) or a simple time-out. Let’s build on the previous example and define it a bit more strictly as a session. At times, you might have an explicit session ID that you can use in your function. This obviously makes it much easier because you can just perform a simple aggregation and might not even need your own stateful logic. In this case, you’re creating sessions on the fly from a user ID and some time information and if you see no new event from that user in five seconds, the session terminates. You’ll also notice that this code uses time-outs differently than we have in other examples.

会话仅是具有一系列事件的不明确时间的窗口。通常，您希望在数组中记录这些不同的事件，以便将来将这些会话与其他会话进行比较。在会话中，您可能拥有随时间去维护和更新状态的任意逻辑，以及定义状态何时结束（如计数）或简单超时的某些操作。让我们在前面的示例基础上进行构建，并将其更严格地定义为一个会话。有时，您可能有一个显式会话ID，可以在函数中使用。这显然使它变得更容易，因为您可以执行简单的聚合，甚至可能不需要自己的状态逻辑。在这种情况下，您将根据用户ID和一些时间信息动态创建会话，如果在五秒钟内没有看到该用户的新事件，会话将终止。您还将注意到，此代码使用超时的方式与其他示例中的不同。

You can follow the same process of creating your classes, defining our single event update function and then the multievent update function: 

您可以遵循创建类，定义单个事件更新函数，然后定义多事件更新函数的相同过程

```scala
case class InputRow(uid:String, timestamp:java.sql.Timestamp, x:Double,
activity:String)

case class UserSession(val uid:String, var timestamp:java.sql.Timestamp,
var activities: Array[String], var values: Array[Double])

case class UserSessionOutput(val uid:String, var activities: Array[String],
var xAvg:Double)

def updateWithEvent(state:UserSession, input:InputRow):UserSession = {
    // handle malformed dates
    if (Option(input.timestamp).isEmpty) {
        return state
    } state.timestamp = input.timestamp
        state.values = state.values ++ Array(input.x)
        if (!state.activities.contains(input.activity)) {
            state.activities = state.activities ++ Array(input.activity)
    } 
    state
} 

import org.apache.spark.sql.streaming.{GroupStateTimeout, OutputMode,
GroupState}
def updateAcrossEvents(uid:String, inputs: Iterator[InputRow],
oldState: GroupState[UserSession]):Iterator[UserSessionOutput] = {
    inputs.toSeq.sortBy(_.timestamp.getTime).toIterator.flatMap { input =>
    val state = if (oldState.exists) oldState.get else UserSession(uid,
        new java.sql.Timestamp(6284160000000L),Array(), Array())
    val newState = updateWithEvent(state, input)
    if (oldState.hasTimedOut) {
        val state = oldState.get
        oldState.remove()
        Iterator(UserSessionOutput(uid, state.activities,
           newState.values.sum / newState.values.length.toDouble))
     } else if (state.values.length > 1000) {
         val state = oldState.get
         oldState.remove()
         Iterator(UserSessionOutput(uid, state.activities, newState.values.sum / newState.values.length.toDouble))
     } else {
         oldState.update(newState)
         oldState.setTimeoutTimestamp(newState.timestamp.getTime(), "5 seconds")
         Iterator()
     }
   }
}
```

You’ll see in this one that we only expect to see an event at most five seconds late. Anything other than that and we will ignore it. We will use an EventTimeTimeout to set that we want to time-out based on the event time in this stateful operation : 

你会看到在这一个，我们只希望看到一个事件最迟迟五秒。除此之外，我们将忽略它。我们将使用EventTimeTimeout 来设置希望基于此状态操作中的事件时间超时：

```scala
import org.apache.spark.sql.streaming.GroupStateTimeout
withEventTime.where("x is not null")
.selectExpr("user as uid",
	"cast(Creation_Time/1000000000 as timestamp) as timestamp",
	"x", "gt as activity")
.as[InputRow]
.withWatermark("timestamp", "5 seconds")
.groupByKey(_.uid)
.flatMapGroupsWithState(OutputMode.Append,
GroupStateTimeout.EventTimeTimeout)(updateAcrossEvents)
.writeStream
.queryName("count_based_device")
.format("memory")
.start()
```

Querying this table will show you the output rows for each user over this time period:

查询此表将显示此时间段内每个用户的输出行：

As you might expect, sessions that have a number of activities in them have a higher x-axis gyroscope value than ones that have fewer activities. It should be trivial to extend this example to problem sets more relevant to your own domain, as well.

正如您可能期望的那样，具有许多活动的会话比具有较少活动的会话具有更高的X轴陀螺仪值。将这个示例扩展到与您自己的域更相关的问题集应该是很简单的。

## <font color="#9a161a">Conclusion 结论</font>

This chapter covered some of the more advanced topics in Structured Streaming, including event time and stateful processing. This is effectively the user guide to help you actually build out your application logic and turn it into something that provides value. Next, we will discuss what we’ll need to do in order to take this application to production and maintain and update it over time.

本章介绍结构化流中的一些更高级的主题，包括事件时间和状态处理。这实际上是一个用户指南，可以帮助您实际构建应用程序逻辑，并将其转化为提供价值的东西。接下来，我们将讨论需要做什么，以便将此应用程序投入生产，并随着时间的推移对其进行维护和更新。
