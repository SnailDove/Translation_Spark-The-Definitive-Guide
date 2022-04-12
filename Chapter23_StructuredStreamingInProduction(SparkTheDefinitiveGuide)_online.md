---
title: 翻译 Chapter 23 Structured Streaming in Production
subtitle: Spark-The Definitive Guide
date: 2019-08-10
copyright: true
categories: English,中文
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---
<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 23 Structured Streaming in Production<br/>生产环境中的结构化流

The previous chapters of this part of the book have covered Structured Streaming from a user’s perspective. Naturally this is the core of your application. This chapter covers some of the operational tools needed to run Structured Streaming robustly in production after you’ve developed an application.

本书这一部分的前几章从用户的角度介绍了结构化流。当然，这是应用程序的核心。**本章介绍在开发应用程序之后，在生产环境中可靠地运行结构化流所需的一些操作工具。**

Structured Streaming was marked as production-ready in Apache Spark 2.2.0, meaning that this release has all the features required for production use and stabilizes the API. Many organizations are already using the system in production because, frankly, it’s not much different from running other production Spark applications. Indeed, through features such as transactional sources/sinks and exactly-once processing, the Structured Streaming designers sought to make it as easy to operate as possible. This chapter will walk you through some of the key operational tasks specific to Structured Streaming. This should supplement everything we saw and learned about Spark operations in Part II.

结构化流在 Apache Spark 2.2.0中被标记为生产就绪，这意味着该版本具有生产使用所需的所有功能，并稳定了API。许多组织已经在生产中使用该系统，因为坦率地说，**它与运行其他生产Spark应用程序没有太大区别。事实上，通过事务性源/接收器和一次性处理等功能，结构化流设计人员力求使其尽可能易于操作。本章将引导您完成特定于结构化流的一些关键操作任务**。这应该补充我们在第二部分中看到和学到的关于 Spark 操作的所有知识。

## <font style="color:#9a161a">Fault Tolerance and Checkpointing<br/>容错和检查点</font>

The most important operational concern for a streaming application is failure recovery. Faults are inevitable: you’re going to lose a machine in the cluster, a schema will change by accident without a proper migration, or you may even intentionally restart the cluster or application. In any of these cases, Structured Streaming allows you to recover an application by just restarting it. To do this, you must configure the application to use checkpointing and write-ahead logs, both of which are handled automatically by the engine. Specifically, you must configure a query to write to a checkpoint location on a reliable file system (e.g., HDFS, S3, or any compatible filesystem). Structured Streaming will then periodically save all relevant progress information (for instance, the range of offsets processed in a given trigger) as well as the current intermediate state values to the checkpoint location. In a failure scenario, you simply need to restart your application, making sure to point to the same checkpoint location, and it will automatically recover its state and start processing data where it left off. You do not have to manually manage this state on behalf of the application—Structured Streaming does it for you.

流应用程序最重要的操作问题是故障恢复。错误是不可避免的：您将失去集群中的一台机器，一个模式将在没有适当迁移的情况下意外更改，或者您甚至可能有意重新启动集群或应用程序。在这些情况下，**结构化流允许您通过重新启动应用程序来恢复应用程序。为此，必须将应用程序配置为使用检查点和提前写入日志，这两个日志都由引擎自动处理**。具体来说，您必须配置一个查询，以写入可靠文件系统（例如，HDFS、S3或任何兼容的文件系统）上的检查点位置。结构化流将定期将所有相关的进度信息（例如，在给定触发器中处理的偏移范围）以及当前中间状态值保存到检查点位置。**在失败的情况下，只需重新启动应用程序，确保指向相同的检查点位置，它将自动恢复其状态，并在停止的地方开始处理数据。您不必代表应用程序手动管理此状态，结构化流为您做到了这一点**。

To use checkpointing, specify your checkpoint location *before* starting your application through the <font face="constant-width" color="#000000" size=4>checkpointLocation</font> option on writeStream. You can do this as follows: 

**要使用检查点，请在通过 writeStream 上的检查点位置选项启动应用程序之前指定检查点位置**。您可以这样做：

```scala
// in Scala
val static = spark.read.json("/data/activity-data")
val streaming = spark
.readStream
.schema(static.schema).option("maxFilesPerTrigger", 10)
.json("/data/activity-data")
.groupBy("gt")
.count()
val query = streaming
.writeStream
.outputMode("complete")
.option("checkpointLocation", "/some/location/")
.queryName("test_stream")
.format("memory")
.start()
```

```python
# in Python
static = spark.read.json("/data/activity-data")
streaming = spark\
.readStream\
.schema(static.schema)\
.option("maxFilesPerTrigger", 10)\
.json("/data/activity-data")\
.groupBy("gt")\
.count()
query = streaming\
.writeStream\
.outputMode("complete")\
.option("checkpointLocation", "/some/python/location/")\
.queryName("test_python_stream")\
.format("memory")\
.start()
```

If you lose your checkpoint directory or the information inside of it, your application will not be able to recover from failures and you will have to restart your stream from scratch.

**如果丢失了检查点目录或其中的信息，应用程序将无法从失败中恢复，您必须从头开始重新启动流。**

<font style="color:#9a161a"><strong>Updating Your Application<br/>更新应用程序</strong></font>

Checkpointing is probably the most important thing to enable in order to run your applications in production. This is because the checkpoint will store all of the information about what your stream has processed thus far and what the intermediate state it may be storing is. However, checkpointing does come with a small catch—you’re going to have to reason about your old checkpoint data when you update your streaming application. When you update your application, you’re going to have to ensure that your update is not a breaking change. Let’s cover these in detail when we review the two types of updates: either an update to your application code or running a new Spark version.

**为了在生产环境中运行应用程序，检查点可能是最重要的。这是因为检查点将存储到目前为止您的流处理的内容以及它可能存储的中间状态的所有信息。但是，检查点的出现只是一个小问题，当您更新流应用程序时，您必须考虑旧的检查点数据。当你更新你的应用程序时，你必须确保你的更新不是一个突破性的改变。**当我们回顾这两种类型的更新时，让我们详细介绍一下这些：**应用程序代码的更新或者运行一个新的 Spark 版本**。

### <font style="color:#000000">Updating Your Streaming Application Code<br/>更新流应用程序代码</font>

Structured Streaming is designed to allow certain types of changes to the application code between application restarts. Most importantly, you are allowed to change user-defined functions (UDFs) as long as they have the same type signature. This feature can be very useful for bug fixes. For example, imagine that your application starts receiving a new type of data, and one of the data parsing functions in your current logic crashes. With Structured Streaming, you can recompile the application with a new version of that function and pick up at the same point in the stream where it crashed earlier. While small adjustments like adding a new column or changing a UDF are not breaking changes and do not require a new checkpoint directory, there are larger changes that do require an entirely new checkpoint directory. For example, if you update your streaming application to add a new aggregation key or fundamentally change the query itself, Spark cannot construct the required state for the new query from an old checkpoint directory. In these cases, Structured Streaming will throw an exception saying it cannot begin from a checkpoint directory, and you must start from scratch with a new (empty) directory as your checkpoint location.

**结构化流的设计允许在应用程序重新启动之间对应用程序代码进行某些类型的更改。最重要的是，您可以更改用户定义函数（UDF），只要它们具有相同的类型签名。这个特性对于错误修复非常有用**。例如，假设应用程序开始接收新类型的数据，并且当前逻辑崩溃时的一个数据解析函数。使用结构化流，您可以使用该函数的新版本重新编译应用程序，并在流中之前崩溃的同一点上继续进行。**虽然诸如添加新列或更改UDF之类的小调整不是突破性的改变，也不需要新的检查点目录，但仍有较大的更改需要全新的检查点目录。例如，如果更新流应用程序以添加新的聚合键或从根本上更改查询本身，Spark将无法从旧的检查点目录构造新查询所需的状态。在这些情况下，结构化流将抛出一个异常，说明它不能从检查点目录开始，并且必须从头开始，使用一个新的（空）目录作为检查点位置。**

### <font style="color:#000000">Updating Your Spark Version<br/>更新Spark版本</font>

Structured Streaming applications should be able to restart from an old checkpoint directory across patch version updates to Spark (e.g., moving from Spark 2.2.0 to 2.2.1 to 2.2.2). The checkpoint format is designed to be forward-compatible, so the only way it may be broken is due to critical bug fixes. If a Spark release cannot recover from old checkpoints, this will be clearly documented in its release notes. The Structured Streaming developers also aim to keep the format compatible across minor version updates (e.g., Spark 2.2.x to 2.3.x), but you should check the release notes to see whether this is supported for each upgrade. In either case, if you cannot start from a checkpoint, you will need to start your application again using a new checkpoint directory.

结构化流应用程序应该能够从旧的检查点目录跨补丁版本更新重新启动到 Spark（例如，从 Spark 2.2.0迁移到2.2.1到2.2.2）。**检查点格式设计为向前兼容，因此唯一可能被破坏的方法是修复关键的错误。如果Spark发行版不能从旧的检查点恢复，那么它的发行说明中会清楚地记录这一点。结构化流式开发人员还致力于保持格式在次要版本更新（例如spark 2.2.x到2.3.x）之间的兼容性，但是您应该检查发行说明，以查看是否支持每次升级。在这两种情况下，如果无法从检查点启动，则需要使用新的检查点目录重新启动应用程序**。

### <font style="color:#000000">Sizing and Rescaling Your Application<br/>调整应用程序的大小和重新缩放</font>

In general, the size of your cluster should be able to comfortably handle bursts above your data rate. The key metrics you should be monitoring in your application and cluster are discussed as follows. In general, if you see that your input rate is much higher than your processing rate (elaborated upon momentarily), it’s time to scale up your cluster or application. Depending on your resource manager and deployment, you may just be able to dynamically add executors to your application. When it comes time, you can scale-down your application in the same way—remove executors (potentially through your cloud provider) or restart your application with lower resource counts. These changes will likely incur some processing delay (as data is recomputed or partitions are shuffled around when executors are removed). In the end, it’s a business decision as to whether it’s worthwhile to create a system with more sophisticated resource management capabilities.

一般来说，集群的大小应该能够轻松地处理高于数据速率的突发事件。您应该在应用程序和集群中监控的关键指标讨论如下。**一般来说，如果您看到您的输入速率远远高于您的处理速率（马上详细描述），那么是时候扩展集群或应用程序了**。根据您的资源管理器和部署，您可能只能动态地向应用程序添加执行器。**当遇到这种情况时，您可以用同样的方法缩小应用程序的规模，删除执行者（可能通过云提供商）或以较低的资源计数重新启动应用程序。这些更改可能会导致一些处理延迟（当执行器被删除时，数据会重新计算或分区会四处移动）**。最后，对于是否值得创建一个具有更复杂资源管理功能的系统，这是一个业务决策。

While making underlying infrastructure changes to the cluster or application are sometimes necessary, other times a change may only require a restart of the application or stream with a new configuration. For instance, changing `spark.sql.shuffle.partitions` is not supported while a stream is currently running (it won’t actually change the number of shuffle partitions). This requires restarting the actual stream, not necessarily the entire application. Heavier weight changes, like changing arbitrary Spark application configurations, will likely require an application restart. 

虽然有时需要对集群或应用程序进行基础结构更改，但在其他情况下，更改可能只需要用新配置重新启动应用程序或流。例如，当流当前正在运行时，不支持更改 `spark.sql.shuffle.partitions`（它实际上不会更改shuffle分区的数目）。这需要重新启动实际流，而不一定是整个应用程序。更重的重量变化，如改变任意的 Spark 应用程序配置，可能需要重新启动应用程序。

## <font style="color:#9a161a">Metrics and Monitoring<br/>量化指标和监控</font>

Metrics and monitoring in streaming applications is largely the same as for general Spark applications using the tools described in Chapter 18. However, Structured Streaming does add several more specifics in order to help you better understand the state of your application. There are two key APIs you can leverage to query the status of a streaming query and see its recent execution progress. With these two APIs, you can get a sense of whether or not your stream is behaving as expected.

流应用程序中的量化指标和监控与使用第18章中描述的工具的一般 Spark 应用程序基本相同。但是，结构化流确实添加了更多的细节，以帮助您更好地了解应用程序的状态。您可以利用两个关键API来查询流式查询的状态并查看其最近的执行进度。通过这两个API，您可以了解流是否按预期运行。

### <font style="color:#000000">Query Status<br/>查询状态</font>

The query status is the most basic monitoring API, so it’s a good starting point. It aims to answer the question, “What processing is my stream performing right now?” This information is reported in the <font face="constant-width" color="#000000" size=4>status</font> field of the query object returned by <font face="constant-width" color="#000000" size=4>startStream</font>. For example, you might have a simple counts stream that provides counts of IOT devices defined by the following query (here we’re just using the same query from the previous chapter without the initialization code) :

查询状态是最基本的监控API，所以它是一个很好的起点。它的目的是回答这个问题，“我的流现在正在执行什么处理？”“此信息在 startStream 返回的查询对象的 status 字段中报告。例如，您可能有一个简单的计数流，它提供由以下查询定义的物联网设备计数（这里我们只使用上一章中的相同查询，而不使用初始化代码）：

```scala
query.status
```

To get the status of a given query, simply running the command <font face="constant-width" color="#000000" size=4>query.status</font> will return the current status of the stream. This gives us details about what is happening at that point in time in the stream. Here’s a sample of what you’ll get back when querying this status:

要获取给定查询的状态，只需运行命令 query.status 即可返回流的当前状态。这为我们提供了有关流中那个时间点发生的事情的详细信息。以下是查询此状态时将返回的示例：

```text
{
	"message" : "Getting offsets from ...",
	"isDataAvailable" : true,
	"isTriggerActive" : true
}
```

The above snippet describes getting the offsets from a Structured Streaming data source (hence the message describing getting offsets). There are a variety of messages to describe the stream’s status.

上面的代码段描述了从结构化流数据源获取偏移量（因此描述获取偏移量的消息）。有各种各样的消息来描述流的状态。

---

<center><font face="constant-width" color="#73737b" size=4><strong>NOTE</strong></font></center>

We have shown the status command inline here the way you would call it in a Spark shell. However, for a standalone application, you may not have a shell attached to run arbitrary code inside your process. In that case, you can expose its status by implementing a monitoring server, such as a small HTTP server that listens on a port and returns query.status when it gets a request. Alternatively, you can use the richer `StreamingQueryListener` API described later to listen to more events.

**我们已经在这里显示了 status 命令，您可以在 Spark shell中调用它。但是，对于独立的应用程序，可能没有附加 shell 到进程内运行任意代码。在这种情况下，您可以通过实现监控服务器来公开其状态，例如在端口上侦听并在收到请求时返回 query.status 的小型 HTTP 服务器。或者，您可以使用后面描述的更丰富的 streamingQueryListener API来监听更多的事件。**

---

### <font style="color:#000000">Recent Progress<br/>最新进度</font>

While the query’s current status is useful to see, equally important is an ability to view the query’s progress. The progress API allows us to answer questions like “At what rate am I processing tuples?” or “How fast are tuples arriving from the source?” By running query.recentProgress, you’ll get access to more time-based information like the processing rate and batch durations. The streaming query progress also includes information about the input sources and output sinks behind your stream. 

虽然查询的当前状态很有用，但查看查询进度的能力同样重要。progress API 允许我们回答“我以什么速率处理元组（tuples）？”或者“元组（tuples）从源文件到达的速度有多快？”“通过运行 query.recentProgress，您可以访问更多基于时间的信息，如处理速率和批处理持续时间。流查询进度还包括有关流后面的输入源和输出接收器的信息。

```scala
query.recentProgress
```

Here’s the result of the Scala version after we ran the code from before; the Python one will be similar:

下面是 Scala 版本在运行之前的代码之后的结果；Python 版本将类似：

```text
Array({
    "id" : "d9b5eac5-2b27-4655-8dd3-4be626b1b59b",
    "runId" : "f8da8bc7-5d0a-4554-880d-d21fe43b983d",
    "name" : "test_stream",
    "timestamp" : "2017-08-06T21:11:21.141Z",
    "numInputRows" : 780119,
    "processedRowsPerSecond" : 19779.89350912779,
    "durationMs" : {
        "addBatch" : 38179,
        "getBatch" : 235,
        "getOffset" : 518,
        "queryPlanning" : 138,
        "triggerExecution" : 39440,
        "walCommit" : 312
    },
    "stateOperators" : [ {
        "numRowsTotal" : 7,
        "numRowsUpdated" : 7
    } ],
    "sources" : [ {
        "description" : "FileStreamSource[/some/stream/source/]",
        "startOffset" : null,
        "endOffset" : {
        	"logOffset" : 0
        },
        "numInputRows" : 780119,
        "processedRowsPerSecond" : 19779.89350912779
    } ],
    "sink" : {
    "description" : "MemorySink"
    }
})
```

As you can see from the output just shown, this includes a number of details about the state of the stream. It is important to note that this is a snapshot in time (according to when we asked for the query progress). In order to consistently get output about the state of the stream, you’ll need to query this API for the updated state repeatedly. The majority of the fields in the previous output should be selfexplanatory. However, let’s review some of the more consequential fields in detail.

正如您从刚刚显示的输出中看到的那样，这包括一些关于流状态的详细信息。**需要注意的是，这是一个及时的快照（根据我们何时请求查询进度）。为了一致地获得有关流状态的输出，您需要反复查询此API以获取更新状态**。上一个输出中的大多数字段都应该是一目了然的。但是，让我们详细回顾一些更重要的字段。

##### <font style="color:#3399cc">Input rate and processing rate<br/>输入速率和处理速率</font>

The input rate specifies how much data is flowing into Structured Streaming from our input source. The processing rate is how quickly the application is able to analyze that data. In the ideal case, the input and processing rates should vary together. Another case might be when the input rate is much greater than the processing rate. When this happens, the stream is falling behind and you will need to scale the cluster up to handle the larger load.

输入速率指定从输入源流入结构化流的数据量。处理速度是应用程序分析数据的速度。**在理想情况下，输入和处理速率应该同时变化。另一种情况可能是输入速率远远大于处理速率。当这种情况发生时，流将落在后面，您需要向上扩展集群以处理更大的负载。**

##### <font style="color:#3399cc">Batch duration<br/>批处理持续时间</font>

Nearly all streaming systems utilize batching to operate at any reasonable throughput (some have an option of high latency in exchange for lower throughput). Structured Streaming achieves both. As it operates on the data, you will likely see batch duration oscillate as Structured Streaming processes varying numbers of events over time. Naturally, this metric will have little to no relevance when the continuous processing engine is made an execution option.

几乎所有的流系统都利用批处理以任何合理的吞吐量运行（有些系统可以选择高延迟，以换取较低的吞吐量）。结构化流实现了这两个目标。当它对数据进行操作时，您可能会看到批处理持续时间随着结构化流处理时间的变化而波动。当然，当连续处理引擎成为一个执行选项时，这个量化指标几乎没有相关性。

---

<center><font face="constant-width" color="#737373" size=4><strong>TIP 提示</strong></font></center>

Generally it’s a best practice to visualize the changes in batch duration and input and processing rates. It’s much more helpful than simply reporting changes over time. 

**一般来说，将批处理持续时间、输入和处理速率的变化可视化是最佳实践。它比简单地报告随时间变化更有用**。

---

### <font style="color:#000000">Spark UI<br/>Spark用户界面</font>

The Spark web UI, covered in detail in Chapter 18, also shows tasks, jobs, and data processing metrics for Structured Streaming applications. On the Spark UI, each streaming application will appear as a sequence of short jobs, one for each trigger. However, you can use the same UI to see metrics, query plans, task durations, and logs from your application. One departure of note from the DStream API is that the Streaming Tab is not used by Structured Streaming.

第18章详细介绍了Spark Web 用户界面，它还显示了结构化流应用程序的任务、作业和数据处理指标。**在Spark用户界面上，每个流式应用程序将显示为一系列短作业，每个触发器一个。但是，您可以使用同一个UI查看来自应用程序的量化指标、查询计划、任务工期和日志。**与 DStream API 不同的一点是，结构化流不使用流选项卡。

## <font style="color:#9a161a">Alerting<br/>告警</font>

Understanding and looking at the metrics for your Structured Streaming queries is an important first step. However, this involves constantly watching a dashboard or the metrics in order to discover potential issues. You’re going to need robust automatic alerting to notify you when your jobs are failing or not keeping up with the input data rate without monitoring them manually. There are several ways to integrate existing alerting tools with Spark, generally building on the recent progress API we covered before. For example, you may directly feed the metrics to a monitoring system such as the open source Coda Hale Metrics library or Prometheus, or you may simply log them and use a log aggregation system like Splunk. In addition to monitoring and alerting on queries, you’re also going to want to monitor and alert on the state of the cluster and the overall application (if you’re running multiple queries together).

**了解和查看结构化流式查询的指标是重要的第一步。但是，这需要不断观察仪表盘或指标，以发现潜在的问题。当你的工作失败或者没有手动监控就不能跟上输入数据速率时，你需要强大的自动警报来通知你。有几种方法可以将现有的警报工具与Spark集成在一起，通常基于我们之前介绍的新近发展的API。例如，您可以直接将量化指标输入监控系统**，如开源 Coda Hale Metrics 库或 Prometheus ，**也可以简单地将其记录并使用日志聚合系统，如Splunk。除了对查询进行监控和警报之外，您还需要对集群和整个应用程序的状态进行监控和发出警报（如果您一起运行多个查询）。**

## <font style="color:#9a161a">Advanced Monitoring with the Streaming Listener<br/>使用流式侦听器进行高级监控</font>

We already touched on some of the high-level monitoring tools in Structured Streaming. With a bit of glue logic, you can use the status and queryProgress APIs to output monitoring events into your organization’s monitoring platform of choice (e.g., a log aggregation system or Prometheus dashboard). Beyond these approaches, there is also a lower-level but more powerful way to observe an application’s execution: the <font face="constant-width" color="#000000" size=4>StreamingQueryListener</font> class. 

我们已经讨论了结构化流中的一些高级监控工具。使用一些粘合逻辑，您可以使用状态和 queryProgress API将监控事件输出到组织的监控平台（例如，日志聚合系统或 Prometheus 仪表板）。除了这些方法之外，还有一种更低阶但更强大的方法来观察应用程序的执行：StreamingQueryListener 类。

The <font face="constant-width" color="#000000" size=4>StreamingQueryListener</font> class will allow you to receive asynchronous updates from the streaming query in order to automatically output this information to other systems and implement robust monitoring and alerting mechanisms. You start by developing your own object to extend <font face="constant-width" color="#000000" size=4>StreamingQueryListener</font>, then attach it to a running <font face="constant-width" color="#000000" size=4>SparkSession</font>. Once you attach your custom listener with <font face="constant-width" color="#000000" size=4>sparkSession.streams.addListener()</font>, your class will receive notifications when a query is started or stopped, or progress is made on an active query. Here’s a simple example of a listener from the Structured Streaming documentation: 

**StreamingQueryListener 类将允许您从流查询接收异步更新，以便自动将此信息输出到其他系统，并实现可靠的监控和警报机制。首先开发自己的对象来扩展 StreamingQueryListener，然后将其附加到正在运行的SparkSession。使用 sparkSession.streams.addListener（）附加自定义侦听器后，当查询启动或停止，或在活动查询上取得进展时，类将收到通知。**以下是结构化流文档中侦听器的简单示例：

```scala
val spark: SparkSession = ...
    spark.streams.addListener(new StreamingQueryListener() {
        
    override def onQueryStarted(queryStarted: QueryStartedEvent): Unit = {
        println("Query started: " + queryStarted.id)
	} 
        
    override def onQueryTerminated(queryTerminated: QueryTerminatedEvent): Unit = {
        println("Query terminated: " + queryTerminated.id)
	} 
        
    override def onQueryProgress(queryProgress: QueryProgressEvent): Unit = {
		println("Query made progress: " + queryProgress.progress)
	}
})
```

Streaming listeners allow you to process each progress update or status change using custom code and pass it to external systems. For example, the following code for a StreamingQueryListener that will <font color="#32cd32">forward</font> all query progress information <font color="#32cd32">to</font> Kafka. You’ll have to parse this JSON string once you read data from Kafka in order to access the actual metrics: 

**流式侦听器（streaming listeners）允许您使用自定义代码处理每个进度更新或状态更改，并将其传递给外部系统**。例如，下面的代码用于将所有查询进度信息转发到 Kafka 的  StreamingQueryListener。从Kafka读取数据后，必须解析这个JSON字符串，才能访问实际的量化指标：

```scala
class KafkaMetrics(servers: String) extends StreamingQueryListener {
    val kafkaProperties = new Properties()
    kafkaProperties.put("bootstrap.servers", servers)
    kafkaProperties.put( "key.serializer", "kafkashaded.org.apache.kafka.common.serialization.StringSerializer")
    kafkaProperties.put( "value.serializer", "kafkashaded.org.apache.kafka.common.serialization.StringSerializer")
    val producer = new KafkaProducer[String, String](kafkaProperties)
    
    import org.apache.spark.sql.streaming.StreamingQueryListener
    import org.apache.kafka.clients.producer.KafkaProduceroverride 
    
    def onQueryProgress(event : StreamingQueryListener.QueryProgressEvent): Unit = {
        producer.send(new ProducerRecord("streaming-metrics",
        event.progress.json))
    } 
	override def onQueryStarted(event:
                    StreamingQueryListener.QueryStartedEvent) : Unit = {}
    override def onQueryTerminated(event:
                    StreamingQueryListener.QueryTerminatedEvent) : Unit = {}
}
```

Using the StreamingQueryListener interface, you can even monitor Structured Streaming applications on one cluster by running a Structured Streaming application on that same (or another) cluster. You could also manage multiple streams in this way.

使用streamingquerylistener接口，您甚至可以通过在同一个（或另一个）集群上运行结构化流应用程序来监控一个集群上的结构化流应用程序。您还可以用这种方式管理多个流。

## <font style="color:#9a161a">Conclusion<br/>小结</font>

In this chapter, we covered the main tools needed to run Structured Streaming in production: checkpoints for fault tolerance and various monitoring APIs that let you observe how your application is running. Lucky for you, if you’re running Spark in production already, many of the concepts and tools are similar, so you should be able to reuse a lot of your existing knowledge. Be sure to check Part IV to see some other helpful tools for monitoring Spark Applications. 

**在本章中，我们介绍了在生产环境中运行结构化流所需的主要工具：容错检查点和各种监控API，这些API允许您观察应用程序的运行情况。幸运的是，如果您已经在生产中运行了Spark，那么许多概念和工具都是类似的，因此您应该能够重用大量现有的知识。一定要检查第四部分，看看其他一些有助于监测 Spark 应用的工具**。
