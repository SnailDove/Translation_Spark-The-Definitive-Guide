---
title: 翻译 Chapter 21 Structured Streaming Basics
date: 2019-06-28
copyright: true
categories: English,中文
tags: [Spark]
mathjax: true
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 21 Structured Streaming Basics 结构化流基础

Now that we have covered a brief overview of stream processing, let’s dive right into Structured Streaming. In this chapter, we will, again, state some of the key concepts behind Structured Streaming and then apply them with some code examples that show how easy the system is to use.

既然我们已经介绍了流处理的简要概述，那么让我们直接深入到 Structured Streaming 中。在本章中，我们将再次说明 Structured Streaming 背后的一些关键概念，然后将它们与一些代码示例一起应用，这些代码示例说明系统的易用性。

## <font color="#9a161a">Structured Streaming Basics 结构化流基础</font>

Structured Streaming, as we discussed at the end of Chapter 20, is a stream processing framework built on the Spark SQL engine. Rather than introducing a separate API, Structured Streaming uses the existing structured APIs in Spark (DataFrames, Datasets, and SQL), meaning that all the operations you are familiar with there are supported. Users express a streaming computation in the same way they’d write a batch computation on static data. Upon specifying this, and specifying a streaming destination, the Structured Streaming engine will take care of running your query incrementally and continuously as new data arrives into the system. These logical instructions for the computation are then executed using the same Catalyst engine discussed in Part II of this book, including query optimization, code generation, etc. Beyond the core structured processing engine, Structured Streaming includes a number of features specifically for streaming. For instance, Structured Streaming ensures end-to-end, exactly-once processing as well as fault-tolerance through checkpointing and write-ahead logs.

Structured Streaming，正如我们在第20章末尾所讨论的，是基于Spark SQL引擎构建的流处理框架。Structured Streaming 处理不引入单独的API，而是使用Spark中现有的结构化API（DataFrames、Datasets 和 SQL），这意味着支持您熟悉的所有操作。用户表达流计算的方式与对静态数据编写批处理计算的方式相同。在指定了这一点并指定了流目的地之后，Structured Streaming 引擎将负责在新数据到达系统时以增量和连续的方式运行查询。然后使用本书第二部分中讨论的相同Catalyst引擎执行这些计算逻辑指令，包括查询优化、代码生成等。除了核心结构化处理引擎之外，Structured Streaming 还包括一些专门与流相关的特性。例如，Structured Streaming 通过检查点和提前写入日志确保端到端、一次性处理以及容错性。


The main idea behind Structured Streaming is to treat a stream of data as a table to which data is continuously appended. The job then periodically checks for new input data, process it, updates some internal state located in a state store if needed, and updates its result. A cornerstone of the API is that you should not have to change your query’s code when doing batch or stream processing—you should have to specify only whether to run that query in a batch or streaming fashion. Internally, Structured Streaming will automatically figure out how to “incrementalize” your query, i.e., update its result efficiently whenever new data arrives, and will run it in a fault-tolerant fashion. 

Structured Streaming 背后的主要思想是将数据流视为一个连续追加数据的表。然后，该作业会定期检查新的输入数据、处理它、根据需要更新位于状态存储中的某些内部状态，并更新其结果。API的一个基石是，在进行批处理或流处理时，不必更改查询的代码，只需指定是以批处理还是流方式运行该查询。在内部，Structured Streaming 将自动计算出如何“增量化”查询，即在新数据到达时高效地更新其结果，并以容错的方式运行它。

![1564674106272](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter21/1564674106272.png)

In simplest terms, Structured Streaming is “your DataFrame, but streaming.” This makes it very easy to get started using streaming applications. You probably already have the code for them! There are some limits to the types of queries Structured Streaming will be able to run, however, as well as some new concepts you have to think about that are specific to streaming, such as event-time and out of-order data. We will discuss these in this and the following chapters.

简单来说，Structured Streaming 是“你的DataFrame，却是流”，这使得开始使用流应用程序变得非常容易。你可能已经有了他们的代码了！但是，结构化流能够运行的查询类型有一些限制，以及一些您必须考虑的特定于流媒体的新概念，例如事件时间和无序数据。我们将在本章和以下章节中讨论这些问题。

Finally, by integrating with the rest of Spark, Structured Streaming enables users to build what we call continuous applications. A continuous application is an end-to-end application that reacts to data in real time by combining a variety of tools: streaming jobs, batch jobs, joins between streaming and offline data, and interactive ad-hoc queries. Because most streaming jobs today are deployed within the context of a larger continuous application, the Spark developers sought to make it easy to specify the whole application in one framework and get consistent results across these different portions of it. For example, you can use Structured Streaming to continuously update a table that users query interactively with Spark SQL, serve a machine learning model trained by MLlib, or join streams with offline data in any of Spark’s data sources—applications that would be much more complex to build using a mix of different tools.

最后，通过与Spark的其余部分集成，Structured Streaming 使用户能够构建我们所称的连续应用程序。连续应用程序是一个端到端的应用程序，它通过组合各种工具实时响应数据：流式作业、批处理作业、流式和离线数据之间的连接以及交互式临时查询。由于目前大多数流作业都部署在一个更大的连续应用程序的环境中，Spark开发人员试图让在一个框架中指定整个应用程序这件事变得容易，并在其中的不同部分获得一致的结果。例如，您可以使用 Structured Streaming 来连续更新用户使用 Spark Sql 进行交互查询的表，提供由 MLlib 训练的机器学习模型，或者连接流和在spark的任何数据源应用程序中使用的离线数据，混合使用不同的工具来构建这些应用程序会复杂得多。

## <font color="#9a161a">Core Concepts 核心概念</font>

Now that we introduced the high-level idea, let’s cover some of the important concepts in a Structured Streaming job. One thing you will hopefully find is that there aren’t many. That’s because Structured Streaming is designed to be simple. Read some other big data streaming books and you’ll notice that they begin by introducing terminology like distributed stream processing topologies for skewed data reducers (a caricature, but accurate) and other complex verbiage. Spark’s goal is to handle these concerns automatically and give users a simple way to run any Spark computation on a stream. Transformations and Actions Structured Streaming maintains the same concept of transformations and actions that we have seen throughout this book. The transformations available in Structured Streaming are, with a few restrictions, the exact same transformations that we saw in Part II. The restrictions usually involve some types of queries that the engine cannot incrementalize yet, although some of the limitations are being lifted in new versions of Spark. There is generally only one action available in Structured Streaming: that of starting a stream, which will then run continuously and output results.

既然我们介绍了高级概念，那么让我们来介绍 Structured Streaming 作业中的一些重要概念。希望你能发现的一件事是没有太多。这是因为 Structured Streaming 的设计很简单。阅读其他一些大数据流书籍，您会注意到，它们首先介绍了一些术语，如用于倾斜的数据的分布式数据流处理拓扑（一个漫画，但准确）和其他复杂的措辞。Spark的目标是自动处理这些问题，并为用户提供一种在流上运行任何Spark计算的简单方法。转换（Transformations ）和动作（Actions ）Structured Streaming 保持了与我们在本书中看到的转换和操作的相同概念。Structured Streaming 中可用的转换，有一些限制，与第二部分中看到的转换完全相同。这些限制通常涉及到一些类型的查询，但是引擎还不能增加这些查询，尽管在新版本的spark中正在解除一些限制。在Structured Streaming 中通常只有一个操作可用：启动流，然后该流将连续运行并输出结果。

### <font color="#000000">Input Sources 输入源</font>

Structured Streaming supports several input sources for reading in a streaming fashion. As of Spark 2.2, the supported input sources are as follows:

Structured Streaming 支持以流方式读取的多个输入源。从Spark 2.2 开始，支持的输入源如下：


- Apache Kafka 0.10 

- Files on a distributed file system like HDFS or S3 (Spark will continuously read new files in a directory)

  分布式文件系统上的文件，如 hdfs 或 s3（Spark将连续读取目录中的新文件）。

- A socket source for testing

  用于测试的套接字源。

- We discuss these in depth later in this chapter, but it’s worth mentioning that the authors of Spark are working on a stable source API so that you can build your own streaming connectors.

    在本章后面我们将深入讨论这些内容，但值得一提的是，Spark的作者正在开发一个稳定的源API，以便您可以构建自己的流式连接器。


### <font color="#000000">Sinks 接收器</font>

Just as sources allow you to get data into Structured Streaming, sinks specify the destination for the result set of that stream. Sinks and the execution engine are also responsible for reliably tracking the exact progress of data processing. Here are the supported output sinks as of Spark 2.2 :

正如源允许您将数据获取到 Structured Streaming 中一样，接收器为该流的结果集指定目标。接收器和执行引擎还负责可靠地跟踪数据处理的准确进度。以下是从 spark 2.2 开始支持的输出接收器：

- Apache Kafka 0.10

    

- Almost any file format

  几乎所有文件格式

- A foreach sink for running arbitary computation on the output records

  用于对输出记录进行任意计算的 foreach 接收器

- A console sink for testing

  用于测试的控制台接收器

- A memory sink for debugging

  用于调试的内存接收器

We discuss these in more detail later in the chapter when we discuss sources.

在本章后面讨论来源时，我们将更详细地讨论这些内容。

### <font color="#000000">Output Modes 输出模式</font>

Defining a sink for our Structured Streaming job is only half of the story. We also need to define how we want Spark to write data to that sink. For instance, do we only want to append new information? Do we want to update rows as we receive more information about them over time (e.g., updating the click count for a given web page)? Do we want to completely overwrite the result set every single time (i.e. always write a file with the complete click counts for all pages)? To do this, we define an output mode, similar to how we define output modes in the static Structured APIs.

为我们的 Structured Streaming 作业定义一个接收器只是故事的一半。我们还需要定义如何希望 Spark 将数据写入该接收器。例如，我们只想附加新信息吗？我们是否希望随着时间的推移更新有关行的更多信息（例如，更新给定网页的单击计数）？是否每次都要完全覆盖结果集（即始终为所有页面编写具有完整单击计数的文件）？为此，我们定义了一个输出模式，类似于我们如何在静态结构化API中定义输出模式。

The supported output modes are as follows:

支持的输出模式如下：

- Append (only add new records to the output sink)

追加（仅向输出接收器添加新记录）

- Update (update changed records in place)

更新（更新已更改的记录）

- Complete (rewrite the full output)
完整（重写完整输出）

One important detail is that certain queries, and certain sinks, only support certain output modes, as we will discuss later in the book. For example, suppose that your job is just performing a map on a stream. The output data will grow indefinitely as new records arrive, so it would not make sense to use Complete mode, which requires writing all the data to a new file at once. In contrast, if you are doing an aggregation into a limited number of keys, Complete and Update modes would make sense, but Append would not, because the values of some keys’ need to be updated over time.

一个重要的细节是，某些查询和某些接收器只支持某些输出模式，正如我们将在本书后面讨论的那样。例如，假设您的工作只是在流上执行映射。当新记录到达时，输出数据将无限增长，因此使用完整模式是没有意义的，这需要立即将所有数据写入新文件。相反，如果您将聚合到有限数量的键中，则完整和更新模式是有意义的，但追加模式是没有意义的，因为某些键的值需要随着时间的推移而更新。

### <font color="#000000">Triggers 触发器</font>

Whereas output modes define how data is output, triggers define when data is output—that is, when Structured Streaming should check for new input data and update its result. By default, Structured Streaming will look for new input records as soon as it has finished processing the last group of input data, giving the lowest latency possible for new results. However, this behavior can lead to writing many small output files when the sink is a set of files. Thus, Spark also supports triggers based on processing time (only look for new data at a fixed interval). In the future, other types of triggers may also be supported.

虽然输出模式定义了数据的输出方式，但触发器定义了何时输出数据，也就是说，Structured Streaming 应该检查新的输入数据并更新其结果。默认情况下，Structured Streaming 将在处理完最后一组输入数据后立即查找新的输入记录，从而为新结果提供尽可能低的延迟。但是，当接收器是一组文件时，这种行为会导致写入许多小的输出文件。因此，Spark 还支持基于处理时间的触发器（只在固定的时间间隔内查看新数据）。将来，还可能支持其他类型的触发器。

### <font color="#000000">Event-Time Processing 事件时间处理</font>

Structured Streaming also has support for event-time processing (i.e., processing data based on timestamps included in the record that may arrive out of order). There are two key ideas that you will need to understand here for the moment; we will talk about both of these in much more depth in the next chapter, so don’t worry if you’re not perfectly clear on them at this point.

Structured Streaming 还支持事件时间处理（即基于记录中可能出现无序的时间戳处理数据）。现在有两个关键的想法需要你理解；我们将在下一章更深入地讨论这两个问题，因此，如果你在这一点上不完全清楚，不要担心。

#### <font color="0a7ae5">Event-time data 事件-时间数据</font>

Event-time means time fields that are embedded in your data. This means that rather than processing data according to the time it reaches your system, you process it according to the time that it was generated, even if records arrive out of order at the streaming application due to slow uploads or network delays. Expressing event-time processing is simple in Structured Streaming. Because the system views the input data as a table, the event time is just another field in that table, and your application can do grouping, aggregation, and windowing using standard SQL operators. However, under the hood, Structured Streaming can take some special actions when it knows that one of your columns is an event time field, including optimizing query execution or determining when it is safe to forget state about a time window. Many of these actions can be controlled using watermarks.

事件-时间表示嵌入在数据中的时间字段。这意味着，您不必根据数据到达系统的时间来处理数据，而是根据数据生成的时间来处理数据，即使由于上传速度慢或网络延迟，记录在流式应用程序中出现无序。在 Structured Streaming 中，表示事件时间处理很简单。因为系统将输入数据视为一个表，所以事件时间只是该表中的另一个字段，您的应用程序可以使用标准的 SQL 运算符进行分组、聚合和窗口化。然而，在底层，当结构化流知道某个列是事件时间字段时，它可以采取一些特殊的操作，包括优化查询执行或确定何时可以安全地忘记时间窗口的状态。其中许多操作可以使用水印进行控制。

#### <font color="0a7ae5">Watermarks 水印</font>

Watermarks are a feature of streaming systems that allow you to specify how late they expect to see data in event time. For example, in an application that processes logs from mobile devices, one might expect logs to be up to 30 minutes late due to upload delays. Systems that support event time, including Structured Streaming, usually allow setting watermarks to limit how long they need to remember old data. Watermarks can also be used to control when to output a result for a particular event time window (e.g., waiting until the watermark for it has passed).

水印是流系统的一个特性，它允许您指定它们期望在事件时间内看到数据的时间有多晚。例如，在处理来自移动设备的日志的应用程序中，由于上载延迟，日志可能会延迟30分钟。支持事件时间（包括 Structured Streaming ）的系统通常允许设置水印以限制它们需要记住旧数据的时间。水印还可用于控制何时为特定事件时间窗口输出结果（例如，等待水印通过）。

## <font color="#9a161a">Structured Streaming in Action 结构化流动作</font>

Let’s get to an applied example of how you might use Structured Streaming. For our examples, we’re going to be working with the Heterogeneity Human Activity Recognition Dataset. The data consists of smartphone and smartwatch sensor readings from a variety of devices—specifically, the accelerometer and gyroscope, sampled at the highest possible frequency supported by the devices. Readings from these sensors were recorded while users performed activities like biking, sitting, standing, walking, and so on. There are several different smartphones and smartwatches used, and nine total users. You can download the data here, in the activity data folder.

让我们来看一个如何使用结构化流的应用示例。对于我们的例子，我们将使用 Heterogeneity Human Activity Recognition（多种不同人类活动识别）数据集。数据由智能手机和智能手表传感器组成，这些传感器的读数来自各种设备，特别是加速度计和陀螺仪，它们以设备支持的最高频率进行采样。当用户进行诸如骑自行车、坐、站、走等活动时，这些传感器的读数被记录下来。使用了几种不同的智能手机和智能手表，总共有九个用户。您可以在“activity data”文件夹中的此处下载数据。

---

<center><font color="#737373"><strong>TIP 提示</strong></font></center>
This Dataset is fairly large. If it’s too large for your machine, you can remove some of the files and it will work just fine. 

这个数据集相当大。如果它对您的机器来说太大，您可以删除一些文件，它将正常工作。

---

Let’s read in the static version of the dataset as a DataFrame :

让我们将数据集的静态版本作为 DataFrame 读取

```scala
// in Scala
val static = spark.read.json("/data/activity-data/")
val dataSchema = static.schema
```

```python
//in Python
static = spark.read.json("/data/activity-data/")
dataSchema = static.schema
Here’s the schema: 
```

Here’s the schema:

这是数据模式：

```shell
root
|-- Arrival_Time: long (nullable = true)
|-- Creation_Time: long (nullable = true)
|-- Device: string (nullable = true)
|-- Index: long (nullable = true)
|-- Model: string (nullable = true)
|-- User: string (nullable = true)
|-- corrupt_record: string (nullable = true)
|-- gt: string (nullable = true)
|-- x: double (nullable = true)
|-- y: double (nullable = true)
|-- z: double (nullable = true)
```

Here’s a sample of the DataFrame:

这是 DataFrame 的一个样例

```shell
+-------------+------------------+--------+-----+------+----+--------+-----+-----
| Arrival_Time| Creation_Time    | Device |Index| Model|User|_c...ord|. gt | x
|1424696634224|142469663222623685|nexus4_1| 62  |nexus4| a  | null   |stand|-0...
...
|1424696660715|142469665872381726|nexus4_1| 2342|nexus4| a  | null   |stand|-0...
+-------------+------------------+--------+-----+------+----+--------+-----+-----
```

You can see in the preceding example, which includes a number of timestamp columns, models, user, and device information. The gt field specifies what activity the user was doing at that time. 

您可以在前面的示例中看到，其中包括许多时间戳列、模型、用户和设备信息。gt 字段指定用户当时正在执行的活动。

Next, let’s create a streaming version of the same Dataset, which will read each input file in the dataset one by one as if it was a stream. 

接下来，让我们创建同一个数据集的流式版本，它将逐个读取数据集中的每个输入文件，就像它是一个流一样。

Streaming DataFrames are largely the same as static DataFrames. We create them within Spark applications and then perform transformations on them to get our data into the correct format. Basically, all of the transformations that are available in the static Structured APIs apply to Streaming DataFrames. However, one small difference is that Structured Streaming does not let you perform schema inference without explicitly enabling it. You can enable schema inference for this by setting the configuration `spark.sql.streaming.schemaInference`  to true. Given that fact, we will read the schema from one file (that we know has a valid schema) and pass the data Schema object from our static DataFrame to our streaming DataFrame. As mentioned, you should avoid doing this in a production scenario where your data may (accidentally) change out from under you : 

Streaming DataFrames 在很大程度上与静态 DataFrames 相同。我们在 Spark 应用程序中创建它们，然后对它们进行转换，以将数据转换为正确的格式。基本上，静态结构化 API 中可用的所有转换都应用于  Streaming DataFrames。然而，一个小的区别是结构化流不允许您在没有显式启用模式推断的情况下执行模式推断。您可以通过设置配置为此启用模式推断 `spark.sql.streaming.schemaInterrusion` 为true。鉴于这个事实，我们将从一个文件（我们知道有一个有效的模式）中读取模式，并将数据模式对象从静态 DataFrame 传递到 streaming DataFrame。如前所述，在生产场景中，您应该避免这样做，因为您的数据可能（意外）从您的下面更改：

```scala
// in Scala
val streaming = spark.readStream.schema(dataSchema)
.option("maxFilesPerTrigger", 1).json("/data/activity-data")
```

```python
# in Python
streaming = spark.readStream.schema(dataSchema).option("maxFilesPerTrigger", 1)\
.json("/data/activity-data")
```
---

<center><font color="#737373"><strong>NOTE 注释</strong></font></center>
We discuss `maxFilesPerTrigger` a little later on in this chapter but essentially it allows you to control how quickly Spark will read all of the files in the folder. By specifying this value lower, we’re artificially limiting the flow of the stream to one file per trigger. This helps us demonstrate how Structured Streaming runs incrementally in our example, but probably isn’t something you’d use in production.

我们稍后在本章中讨论 `MaxFilePertrigger`，但实际上它允许您控制Spark读取文件夹中所有文件的速度。通过将该值指定得更低，我们人为地将流限制为每个触发器读取一个文件。这有助于我们演示 Structured Streaming 在我们的示例中是如何增量运行的，但可能不是您在生产中使用的。

---

Just like with other Spark APIs, streaming DataFrame creation and execution is lazy. In particular, wecan now specify transformations on our streaming DataFrame before finally calling an action to start the stream. In this case, we’ll show one simple transformation—we will group and count data by the gt column, which is the activity being performed by the user at that point in time:

与其他Spark API一样，streaming DataFrame 的创建和执行也是惰性的。特别是，我们现在可以在最后调用一个动作来启动流之前，在 streaming DataFrame 上指定转换。在这种情况下，我们将展示一个简单的转换，我们将按gt列对数据进行分组和计数，这是用户在该时间点执行的活动：

```scala
// in Scala
val activityCounts = streaming.groupBy("gt").count()
```

```python
# in Python
activityCounts = streaming.groupBy("gt").count()
```

Because this code is being written in local mode on a small machine, we are going to set the shuffle partitions to a small value to avoid creating too many shuffle partitions:

因为这段代码是在一台小型机器上以本地模式写入的，所以我们要将数据再分配（shuffle）分区设置为一个较小的值，以避免创建过多的数据再分配（shuffle）分区：

```scala
spark.conf.set("spark.sql.shuffle.partitions", 5)
```

Now that we set up our transformation, we need only to specify our action to start the query. As mentioned previously in the chapter, we will specify an output destination, or output sink for our result of this query. For this basic example, we are going to write to a memory sink which keeps an in-memory table of the results.

既然我们已经设置了转换，我们只需要指定我们的操作来启动查询。如前一章所述，我们将为这个查询的结果指定一个输出目的地或输出接收器（sink）。对于这个基本的例子，我们将数据写到一个内存接收器，它保存一个结果的内存表。

In the process of specifying this sink, we’re going to need to define how Spark will output that data. In this example, we use the complete output mode. This mode rewrites all of the keys along with their counts after every trigger:

在指定这个接收器（sink）的过程中，我们需要定义Spark将如何输出这些数据。在这个例子中，我们使用完整的输出模式。此模式在每次触发后重写所有键及其计数：

```scala
// in Scala
val activityQuery = activityCounts.writeStream.queryName("activity_counts")
.format("memory").outputMode("complete")
.start()
```

```python
# in Python
activityQuery = activityCounts.writeStream.queryName("activity_counts")\
.format("memory").outputMode("complete")\
.start()
```

We are now writing out our stream! You’ll notice that we set a *unique* query name to represent this stream, in this case `activity_counts`. We specified our format as an in-memory table and we set the output mode.

我们现在正在写我们的流！您会注意到，我们设置了一个唯一的查询名称来表示这个流，在本例中是 `activity_counts` 。我们将格式指定为内存中的表，并设置输出模式。

When we run the preceding code, we also want to include the following line:

当我们运行上述代码时，我们还希望包括以下行：

```scala
activityQuery.awaitTermination()
```

After this code is executed, the streaming computation will have started in the background. The query object is a handle to that active streaming query, and we must specify that we would like to wait for the termination of the query using `activityQuery.awaitTermination() ` to prevent the driver process from exiting while the query is active. We will omit this from our future parts of the book for readability, but it must be included in your production applications; otherwise, your stream won’t be able to run.

执行此代码后，流计算将在后台启动。查询对象是该活动流查询的句柄（handle），我们必须指定要使用 `activityquery.wait termination()`等待查询的终止，以防止查询（query）还在激活时驱动程序进程退出。为了可读性，我们将在书的未来部分中省略这一点，但它必须包含在您的生产应用程序中；否则，流将无法运行。

Spark lists this stream, and other active ones, under the active streams in our SparkSession. We can see a list of those streams by running the following:

Spark 在 SparkSession 中的活动流下列出此流和其他活动流。我们可以通过运行以下命令来查看这些流的列表：

```scala
spark.streams.active
```

Spark also assigns each stream a UUID, so if need be you could iterate through the list of running streams and select the above one. In this case, we assigned it to a variable, so that’s not necessary.

Spark 还为每个流分配一个 UUID，因此如果需要，您可以遍历正在运行的流列表并选择上面的流。在这种情况下，我们把它赋给一个变量，这样就不需要了。

Now that this stream is running, we can experiment with the results by querying the in-memory table it is maintaining of the current output of our streaming aggregation. This table will be called `activity_counts`, the same as the stream. To see the current data in this output table, we simply need to query it! We’ll do this in a simple loop that will print the results of the streaming query every second:

现在这个流正在运行，我们可以通过查询内存中的表来试验结果，该表维护流聚合的当前输出。此表将被称为 `activity_counts` ，与流相同。要查看这个输出表中的当前数据，我们只需要查询它！我们将在一个简单的循环中执行此操作，该循环将每秒打印流式查询的结果：

```scala
// in Scala
for( i <- 1 to 5 ) {
spark.sql("SELECT * FROM activity_counts").show()
Thread.sleep(1000)
} 
```

```python
#in Python
from time import sleep
for x in range(5):
spark.sql("SELECT * FROM activity_counts").show()
sleep(1)
```

As the preceding queries run, you should see the counts for each activity change over time. For instance, the first show call displays the following result (because we queried it while the stream was reading the first file):

当前面的查询运行时，您应该看到每个活动的计数随着时间的推移而变化。例如，第一个show调用显示以下结果（因为我们在流读取第一个文件时查询了它）：

```shell
+---+-----+
| gt|count|
+---+-----+
+---+-----+
```

The previous show call shows the following result—note that the result will probably vary when you’re running this code personally because you will likely start it at a different time:

上一个 show 调用显示以下结果注意，当您亲自运行此代码时，结果可能会有所不同，因为您可能会在不同的时间启动它：

```shell
+----------+-----+
|    gt    |count|
+----------+-----+
|    sit   | 8207|
...
|    null  | 6966|
|    bike  | 7199|
+----------+-----+
```

With this simple example, the power of Structured Streaming should become clear. You can take the same operations that you use in batch and run them on a stream of data with very few code changes (essentially just specifying that it’s a stream). The rest of this chapter touches on some of the details about the various manipulations, sources, and sinks that you can use with Structured Streaming

通过这个简单的例子，Structured Streaming 的力量应该变得清晰。您可以采用批处理中使用的相同操作，并在代码更改很少的数据流上运行它们（本质上只是指定它是一个流）。本章的其余部分将详细介绍可用于 Structured Streaming 的各种操作、源和接收器。

## <font color="#9a161a">Transformations on Streams 流上的转换</font>

Streaming transformations, as we mentioned, include almost all static DataFrame transformations that you already saw in Part II. All select, filter, and simple transformations are supported, as are all DataFrame functions and individual column manipulations. The limitations arise on transformations that do not make sense in context of streaming data. For example, as of Apache Spark 2.2, users cannot sort streams that are not aggregated, and cannot perform multiple levels of aggregation without using Stateful Processing (covered in the next chapter). These limitations may be lifted as Structured Streaming continues to develop, so we encourage you to check the documentation of your version of Spark for updates.

正如我们所提到的，流式转换几乎包括了您在第二部分中已经看到的所有静态 DataFrame 转换。支持所有选择、筛选和简单转换，以及所有 DataFrame 函数和单个列操作。在流式数据环境中，转换会产生一些不合理的限制。例如，从 Apache Spark 2.2 开始，用户不能对未聚合的流进行排序，并且不能在不使用状态处理（Stateful Processing）的情况下执行多个级别的聚合（在下一个章中介绍）。随着结构化流的不断发展，这些限制可能会被解除，因此我们鼓励您检查 Spark 版本的文档以获取更新。

### <font color="#000000">Selections and Filtering 选择和筛选</font>

All select and filter transformations are supported in Structured Streaming, as are all DataFrame functions and individual column manipulations. We show a simple example using selections and filtering below. In this case, because we are not updating any keys over time, we will use the Append output mode, so that new results are appended to the output table:

Structured Streaming 中支持所有选择和筛选转换，以及所有 DataFrame 函数和单个列操作。我们用下面的选择和过滤来展示一个简单的例子。在这种情况下，由于我们不会随时间更新任何键，因此我们将使用附加（Append ）输出模式，以便将新结果附加到输出表：

```scala
// in Scala
import org.apache.spark.sql.functions.expr
val simpleTransform = streaming.withColumn("stairs", expr("gt like '%stairs%'"))
.where("stairs")
.where("gt is not null")
.select("gt", "model", "arrival_time", "creation_time")
.writeStream
.queryName("simple_transform")
.format("memory")
.outputMode("append")
.start()
```

```python
# in Python
from pyspark.sql.functions import expr
simpleTransform = streaming.withColumn("stairs", expr("gt like '%stairs%'"))\
.where("stairs")\
.where("gt is not null")\
.select("gt", "model", "arrival_time", "creation_time")\
.writeStream\
.queryName("simple_transform")\
.format("memory")\
.outputMode("append")\
.start()
```

### <font color="#000000">Aggregations 聚合</font>

Structured Streaming has excellent support for aggregations. You can specify arbitrary aggregations, as you saw in the Structured APIs. For example, you can use a more exotic aggregation, like a cube, on the phone model and activity and the average x, y, z accelerations of our sensor (jump back to Chapter 7 in order to see potential aggregations that you can run on your stream):

Structured Streaming 对聚合有极好的支持。您可以指定任意聚合，正如您在结构化API中看到的那样。例如，您可以在电话模型和活动中使用更奇特的聚合，如 `cube`，以及传感器的平均 x、y、z 加速度（请跳回到第7章，以查看您可以在流中运行的潜在聚合）：

```scala
// in Scala
val deviceModelStats = streaming.cube("gt", "model").avg()
.drop("avg(Arrival_time)")
.drop("avg(Creation_Time)")
.drop("avg(Index)")
.writeStream.queryName("device_counts").format("memory").outputMode("complete")
.start()
```

```python
# in Python
deviceModelStats = streaming.cube("gt", "model").avg()\
.drop("avg(Arrival_time)")\
.drop("avg(Creation_Time)")\
.drop("avg(Index)")\
.writeStream.queryName("device_counts").format("memory")\
.outputMode("complete")\
.start()
```

Querying that table allows us to see the results:

查询该表可以查看结果：

```sql
SELECT * FROM device_counts
```

```shell
+----------+------+------------------+--------------------+--------------------+
| gt       | model| avg(x)           | avg(y)             | avg(z)             |
+----------+------+------------------+--------------------+--------------------+
| sit      | null |-3.682775300344...|1.242033094787975...|-4.22021191297611...|
| stand    | null |-4.415368069618...|-5.30657295890281...|2.264837548081631...|
...
| walk     |nexus4|-0.007342235359...|0.004341030525168...|-6.01620400184307...|
|stairsdown|nexus4|0.0309175199508...|-0.02869185568293...| 0.11661923308518365|
...
+----------+------+------------------+--------------------+--------------------+
```

In addition to these aggregations on raw columns in the dataset, Structured Streaming has special support for columns that represent event time, including watermark support and windowing. We will discuss these in more detail in Chapter 22.

除了对数据集中原始列的聚合之外，Structured Streaming 还特别支持表示事件时间的列，包括水印支持和窗口化。我们将在第22章中更详细地讨论这些问题。

---

<center><font color="#737373"><strong>NOTE 注意</strong></font></center>
As of Spark 2.2, the one limitation of aggregations is that multiple “chained” aggregations (aggregations on streaming aggregations) are not supported at this time. However, you can achieve this by writing out to an intermediate sink of data, like Kafka or a file sink. This will change in the future as the Structured Streaming community adds this functionality.

从 Spark 2.2 开始，聚合的一个限制是此时不支持多个“链接（chained）”聚合（流聚合上的聚合）。但是，您可以通过将数据写到中间的接收器（如 Kafka 或 文件接收器）来实现这一点。随着 Structured Streaming 社区添加了这一功能，这一点将来会有所改变。

---

### <font color="#000000">Joins</font>

As of Apache Spark 2.2, Structured Streaming supports joining streaming DataFrames to static DataFrames. Spark 2.3 will add the ability to join multiple streams together. You can do multiple column joins and supplement streaming data with that from static data sources:

从 Apache Spark 2.2 开始，Structured Streaming 支持将 streaming DataFrames 连接到 static DataFrames。Spark 2.3 将添加将多个流连接在一起的能力。您可以执行多个列连接，并使用静态数据源中的数据补充流数据：

```scala
// in Scala
val historicalAgg = static.groupBy("gt", "model").avg()
val deviceModelStats = streaming.drop("Arrival_Time", "Creation_Time", "Index")
.cube("gt", "model").avg()
.join(historicalAgg, Seq("gt", "model"))
.writeStream.queryName("device_counts").format("memory").outputMode("complete")
.start()
```

```python
# in Python
historicalAgg = static.groupBy("gt", "model").avg()
deviceModelStats = streaming.drop("Arrival_Time", "Creation_Time", "Index")\
.cube("gt", "model").avg()\
.join(historicalAgg, ["gt", "model"])\
.writeStream.queryName("device_counts").format("memory")\
.outputMode("complete")\
.start()
```

In Spark 2.2, full outer joins, left joins with the stream on the right side, and right joins with the stream on the left are not supported. Structured Streaming also does not yet support stream-to-stream joins, but this is also a feature under active development.

在 Spark 2.2 中，不支持完全外部连接、左侧连接到右侧的流、右侧连接到左侧的流。Structured Streaming 还不支持流到流的连接，但这也是一个正在积极开发的特性。

## <font color="#9a161a">Input and Output 输入和输出</font>

This section dives deeper into the details of how sources, sinks, and output modes work in Structured Streaming. Specifically, we discuss how, when, and where data flows into and out of the system. As of this writing, Structured Streaming supports several sources and sinks, including Apache Kafka, files, and several sources and sinks for testing and debugging. More sources may be added over time, so be sure to check the documentation for the most up-to-date information. We discuss the source and sink for a particular storage system together in this chapter, but in reality you can mix and match them (e.g., use a Kafka input source with a file sink).

本节深入介绍了源、汇和输出模式在结构化流中的工作方式。具体来说，我们将讨论数据如何、何时和在何处流入和流出系统。在撰写本文时，Structured Streaming 支持多个源和接收器，包括 Apache Kafka、文件以及用于测试和调试的多个源和接收器。随着时间的推移，可能会添加更多的源，因此一定要检查文档以获取最新的信息。在本章中，我们一起讨论特定存储系统的源和接收器，但实际上，您可以混合和匹配它们（例如，使用 Kafka 输入源和文件接收器）。

### <font color="#000000">Where Data Is Read and Written (Sources and Sinks)</font>

Structured Streaming supports several production sources and sinks (files and Apache Kafka), as well as some debugging tools like the memory table sink. We mentioned these at the beginning of the chapter, but now let’s cover the details of each one.

Structured Streaming 支持多个生产源和接收器（文件和Apache Kafka），以及一些调试工具，如内存表接收器。我们在本章的开头提到了这些，但现在让我们来介绍每一个的细节。

#### <font color="0a7ae5">File source and sink 文件源和寄接收器</font>

Probably the simplest source you can think of is the simple file source. It’s easy to reason about and understand. While essentially any file source should work, the ones that we see in practice are Parquet, text, JSON, and CSV. The only difference between using the file source/sink and Spark’s static file source is that with streaming, we can control the number of files that we read in during each trigger via the · option that we saw earlier. Keep in mind that any files you add into an input directory for a streaming job need to appear in it atomically. Otherwise, Spark will process partially written files before you have finished. On file systems that show partial writes, such as local files or HDFS, this is best done by writing the file in an external directory and moving it into the input directory when finished. On Amazon S3, objects normally only appear once fully written.

您能想到的最简单的源可能是简单的文件源。这很容易解释和理解。虽然基本上任何文件源都可以工作，但我们在实践中看到的是 Parquet、文本、JSON 和 CSV。使用文件源/接收器（sink ）和 Spark 的静态文件源之间的唯一区别是，使用流式处理，我们可以通过前面看到的 `maxFilesPerTrigger ` 选项控制在每个触发器期间读取的文件数。请记住，为流式作业添加到输入目录中的任何文件都需要以原子（atomically）的方式出现在文件夹里。否则，Spark将在完成之前处理部分写入的文件。在显示部分写入（如本地文件或HDF）的文件系统上，最好将文件写入外部目录，并在完成后将其移动到输入目录。在Amazon S3 上，对象通常只出现一次完全写入。

#### <font color="0a7ae5">Kafka source and sink Kafka源和接收器</font>

Apache Kafka is a distributed publish-and-subscribe system for streams of data. Kafka lets you publish and subscribe to streams of records like you might do with a message queue—these are stored as streams of records in a fault-tolerant way. Think of Kafka like a distributed buffer. Kafka lets you store streams of records in categories that are referred to as topics. Each record in Kafka consists of a key, a value, and a timestamp. Topics consist of immutable sequences of records for which the position of a record in a sequence is called an offset. Reading data is called subscribing to a topic and writing data is as simple as publishing to a topic. 

Apache Kafka 是一个分布式数据流发布和订阅系统。Kafka 允许您发布和订阅记录流，就像处理消息队列一样。这些记录以容错方式存储为记录流。把 Kafka 想象成一个分布式缓冲区。Kafka 允许您将记录流存储在称为主题（topic）的类别中。Kafka 中的每个记录都由一个键（key）、一个值（value）和一个时间戳（timestamp）组成。主题由不可变的记录序列组成，对于这些记录序列中的记录位置称为偏移量（offset）。读数据被称为订阅主题，写数据和发布主题一样简单。


Spark allows you to read from Kafka with both batch and streaming DataFrames.

Spark允许您使用批处理和 streaming DataFrames 从Kafka中读取数据。

As of Spark 2.2, Structured Streaming supports Kafka version 0.10. This too is likely to expand in the future, so be sure to check the documentation for more information about the Kafka versions available. There are only a few options that you need to specify when you read from Kafka.

从Spark 2.2开始，Structured Streaming 支持Kafka版本 0.10。这在将来也可能会扩展，因此请务必查看文档以获取有关可用 Kafka 版本的更多信息。当您从 Kafka 阅读时，只有几个选项需要指定。

### <font color="#000000">Reading from the Kafka Source 从Kafka源读取数据</font>

To read, you first need to choose one of the following options: <font face="constant-width" size=4>assign</font>, <font face="constant-width " size=4>subscribe</font>, or <font face="constant-width" size=4>subscribePattern</font>. Only one of these can be present as an option when you go to read from Kafka. <font face="constant-width" size=4>Assign</font> is a fine-grained way of specifying not just the topic but also the topic partitions from which you would like to read. This is specified as a JSON string <font face="constant-width" size=4>{"topicA":[0,1],"topicB":[2,4]}</font>. <font face="constant-width" size=4>subscribe</font> and <font face="constant-width" size=4>subscribePattern</font> are ways of subscribing to one or more topics either by specifying a list of topics (in the former) or via a pattern (via the latter). Second, you will need to specify the `kafka.bootstrap.servers` that Kafka provides to connect to the service.

要从 Kafka 读取数据，首先需要选择以下选项之一：assign、subscribe 或 subscribePattern 。当你从 Kafka 的读取数据，只有其中一个可以作为选项出现。assign 是一种细粒度的方法，它不仅指定主题，还指定要从中读取的主题分区。这被指定为JSON字符串 `{"topicA":[0,1],"topicB":[2,4]}`。subscribe  和 subscribePattern 是通过指定主题列表（前者）或通过模式（后者）订阅一个或多个主题的方法。其次，您需要指定 Kafka 提供的 `Kafka.bootstrap.servers` 来连接到服务。

After you have specified your options, you have several other options to specify:

在指定了选项之后，还可以指定其他几个选项：

<font face="constant-width" size=4>startingOffsets</font> and <font face="constant-width" size=4>endingOffsets</font> **起始偏移量和结束偏移量**

The start point when a query is started, either <font face="constant-width" size=3>earliest</font>, which is from the earliest offsets; <font face="constant-width" size=3>latest</font>, which is 	just from the latest offsets; or a JSON string specifying a starting offset for each <font face="constant-width" size=3>TopicPartition</font> . In the                     	JSON, -2 as an offset can be used to refer to earliest, -1 to latest. For example, the JSON specification 		  	could  be <font face="constant-width" size=3>{"topicA":{"0":23,"1":-1},"topicB": {"0":-2}}</font>. This applies only when a new Streaming query               	is started, and that resuming will always pick up from where the query left off. Newly discovered               	partitions during a query will start at earliest. The ending offsets for a given query.
	
开始查询的起始点，可以是最早的，也可以是最早的偏移量；最迟的，只不过是最新的偏移量；或者是指定每个 TopicPartition 的起始偏移量的 JSON 字符串。在 JSON 中，可以使用 -2 作为偏移量来引用最早的，-1表示引用最晚的。例如，JSON 规范可以是 `{"topicA":{"0":23,"1":-1},"topicB": {"0":-2}}` 。这仅在启动新的流式查询时适用，并且恢复将始终从查询停止的位置开始。查询期间新发现的分区将最早开始。给定查询的结束偏移量。

<font face="constant-width" size=4> failOnDataLoss</font> **故障数据丢失**

Whether to fail the query when it’s possible that data is lost (e.g., topics are deleted, or offsets are out of 	range). This might be a false alarm. You can disable it when it doesn’t work as you expected. The default 	is true.
	
当数据可能丢失时（例如，删除主题或偏移量超出范围），是否使查询失败。这可能是虚惊一场。当它不能按预期工作时，您可以禁用它。默认值为true。

<font face="constant-width" size=4>maxOffsetsPerTrigger</font>

The total number of offsets to read in a given trigger. 
	
在给定触发器中要读取的偏移量总数。

There are also options for setting Kafka consumer timeouts, fetch retries, and intervals. 

还有设置 Kafka 消费者（consumer）超时、获取重试和间隔的选项。

To read from Kafka, do the following in Structured Streaming: 

要读取Kafka，请在 Structured Streaming 处理中执行以下操作：

```scala
// in Scala
// Subscribe to 1 topic
val ds1 = spark.readStream.format("kafka")
.option("kafka.bootstrap.servers", "host1:port1,host2:port2")
.option("subscribe", "topic1")
.load()
// Subscribe to multiple topics
val ds2 = spark.readStream.format("kafka")
.option("kafka.bootstrap.servers", "host1:port1,host2:port2")
.option("subscribe", "topic1,topic2")
.load()
// Subscribe to a pattern of topics
val ds3 = spark.readStream.format("kafka")
.option("kafka.bootstrap.servers", "host1:port1,host2:port2")
.option("subscribePattern", "topic.*")
.load()
```
Python is quite similar:  Python是十分的类似

```python
# in Python
# Subscribe to 1 topic
df1 = spark.readStream.format("kafka")\
.option("kafka.bootstrap.servers", "host1:port1,host2:port2")\
.option("subscribe", "topic1")\
.load()
# Subscribe to multiple topics
df2 = spark.readStream.format("kafka")\
.option("kafka.bootstrap.servers", "host1:port1,host2:port2")\
.option("subscribe", "topic1,topic2")\
.load()
# Subscribe to a pattern
df3 = spark.readStream.format("kafka")\.option("kafka.bootstrap.servers", "host1:port1,host2:port2")\
.option("subscribePattern", "topic.*")\
.load()
```

Each row in the source will have the following schema:
源中的每一行将具有以下模式

- key: binary
- value: binary
- topic: string
- partition: int
- offset: long
- timestamp: long

Each message in Kafka is likely to be serialized in some way. Using native Spark functions in the Structured APIs, or a User-Defined Function (UDF), you can parse the message into a more structured format analysis. A common pattern is to use JSON or Avro to read and write to Kafka.

Kafka 的每个消息都可能以某种方式序列化。使用 Structured API 中的本地 Spark 函数或用户定义函数（UDF），您可以将消息解析为更结构化的格式便于分析。一种常见的模式是使用 JSON 或 AVRO 读写 Kafka。

### <font color="#000000">Writing to the Kafka Sink 给 Kafka 接收器写消息</font>

Writing to Kafka queries is largely the same as reading from them except for fewer parameters. You’ll still need to specify the Kafka bootstrap servers, but the only other option you will need to supply is either a column with the topic specification or supply that as an option. For example, the following writes are equivalent:

写入Kafka查询与从中读取基本相同，只是参数较少。您仍然需要指定 `kafka.bootstrap.servers`，但是您将需要提供的唯一其他选项要么是具有主题规范的列，要么是将其作为选项提供。例如，以下写操作是等效的：

```scala
// in Scala
ds1.selectExpr("topic", "CAST(key AS STRING)", "CAST(value AS STRING)")
.writeStream.format("kafka")
.option("checkpointLocation", "/to/HDFS-compatible/dir")
.option("kafka.bootstrap.servers", "host1:port1,host2:port2")
.start()

ds1.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
.writeStream.format("kafka")
.option("kafka.bootstrap.servers", "host1:port1,host2:port2")
.option("checkpointLocation", "/to/HDFS-compatible/dir")\
.option("topic", "topic1")
.start()
```

```python
# in Python
df1.selectExpr("topic", "CAST(key AS STRING)", "CAST(value AS STRING)")\
.writeStream\
.format("kafka")\
.option("kafka.bootstrap.servers", "host1:port1,host2:port2")\
.option("checkpointLocation", "/to/HDFS-compatible/dir")\
.start()

df1.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")\
.writeStream\
.format("kafka")\.option("kafka.bootstrap.servers", "host1:port1,host2:port2")\
.option("checkpointLocation", "/to/HDFS-compatible/dir")\
.option("topic", "topic1")\
.start()
```

#### <font color="0a7ae5">Foreach sink</font>

The <font face="constant-width" size=4>foreach</font> sink is akin to <font face="constant-width" size=4>foreachPartitions</font> in the Dataset API. This operation allows arbitrary operations to be computed on a per-partition basis, in parallel. This is available in Scala and Java initially, but it will likely be ported to other languages in the future. To use the <font face="constant-width" size=4>foreach</font> sink, you must implement the <font face="constant-width" size=4>ForeachWriter</font> interface, which is available in the Scala/Java documents, which contains three methods: <font face="constant-width" size=4>open</font>, <font face="constant-width" size=4>process</font>, and <font face="constant-width" size=4>close</font>. The relevant methods will be called whenever there is a sequence of rows generated as output after a trigger.

Foreach 接收器类似于数据集 API 中的 Foreach 分区。此操作允许以每个分区为基础并行计算任意操作。这在Scala 和 Java 最初是可用的，但它将来可能会移植到其他语言。要使用 Foreach ，必须实现 ForeachWriter 接口，该接口在 Scala/Java 文档中可用，它包含三种方法：打开（`open`）、进程（`process`）和关闭（`close`）。每当有一系列行在触发器之后作为输出生成时，都将调用相关方法。

Here are some important details:

以下是一些重要的细节：

- The writer must be Serializable, as it were a UDF or a Dataset map function.

    编写器（writer ）必须是可序列化的，因为它是一个 UDF 或 数据集映射函数。

- The three methods (<font face="constant-width" size=4>open, process, close</font>) will be called on each executor.

    将对每个执行器调用三个方法（`open`、`process`、`close`）。

- The writer must do all its initialization, like opening connections or starting transactions only in the <font face="constant-width" size=4>open</font> method. A common source of errors is that if initialization occurs outside of the <font face="constant-width" size=4>open</font> method (say in the class that you’re using), that happens on the driver instead of the executor.
  
    编写器（writer ）必须执行其所有初始化，例如只在 `open`方法中打开连接或启动事务。一个常见的错误源是：如果初始化发生在 `open` 方法之外（比如在您正在使用的类中），则会发生在驱动程序上，而不是执行器上。

Because the Foreach sink runs arbitrary user code, one key issue you must consider when using it is fault tolerance. If Structured Streaming asked your sink to write some data, but then crashed, it cannot know whether your original write succeeded. Therefore, the API provides some additional parameters to help you achieve exactly-once processing.

因为 Foreach 接收器运行任意用户代码，所以在使用它时必须考虑的一个关键问题是容错。如果 Structured Streaming 请求接收器写入一些数据，但随后崩溃，则无法知道原始写入是否成功。因此，API提供了一些额外的参数来帮助您实现精确的一次性处理。

First, the open call on your · receives two parameters that uniquely identify the set of rows that need to be acted on. The version parameter is a monotonically increasing ID that increases on a per-trigger basis, and partitionId is the ID of the partition of the output in your task. Your open method should return whether to process this set of rows. If you track your sink’s output externally and see that this set of rows was already output (e.g., you wrote the last version and partitionId written in your storage system), you can return false from open to skip processing this set of rows. Otherwise, return true. Your ForeachWriter will be opened again for each trigger’s worth of data to write.

首先，Foreachwriter 上的 `open` 调用接收两个参数，它们唯一标识了需要对其执行操作的行集。version参数是一个单调递增的ID，它在每个触发器的基础上递增，而 partitionId 是任务中输出分区的ID。您的 `open`方法应该返回是否处理这行集。如果从外部跟踪接收器的输出，并看到这行集已被输出（例如，您编写了存储系统中写入的最后一个版本和 partitionId ），则可以从 `open `返回 false 以跳过处理这组行。否则，返回true。您的ForEachWriter将再次打开，以便为每个触发器的数据量进行写入。

Next, the process method will be called for each record in the data, assuming your open method returned true. This is fairly straightforward—just process or write your data.

接下来，将为数据中的每个记录调用 process 方法，假定open方法返回true。这相当简单，只需处理或编写数据即可。

Finally, whenever open is called, the close method is also called (unless the node crashed before that), regardless of whether open returned true. If Spark witnessed an error during processing, the close method receives that error. It is your responsibility to clean up any open resources duringclose.

最后，无论何时调用 open，也会调用 close 方法（除非节点在此之前崩溃），无论 open 是否返回 true。如果Spark 在处理过程中发现错误，close方法将接收该错误。您有责任在关闭期间清理任何打开的资源。

Together, the ForeachWriter interface effectively lets you implement your own sink, including your own logic for tracking which triggers’ data has been written or safely overwriting it on failures. We show an example of passing a ForeachWriter below:

总之，Foreachwriter 接口有效地让您实现自己的接收器，包括跟踪哪些触发器的数据已被写入或在失败时安全覆盖的逻辑。下面是传递 ForEachWriter 的示例：

```scala
//in Scala
datasetOfString.write.foreach(new ForeachWriter[String] {
    def open(partitionId: Long, version: Long): Boolean = {
    // open a database connection
    } def process(record: String) = {
    // write string to connection
    } def close(errorOrNull: Throwable): Unit = {
    // close the connection
    }
  }
)
```

#### <font color="0a7ae5">Sources and sinks for testing 用于测试的源和接收器</font>

Spark also includes several test sources and sinks that you can use for prototyping or debugging your streaming queries (these should be used only during development and not in production scenarios, because they do not provide end-to-end fault tolerance for your application):

Spark 还包括几个用于测试的源和接收器，您可以使用它们来进行原型设计或调试流式查询（这些测试源和接收器只应在开发期间使用，而不应在生产场景中使用，因为它们不为您的应用程序提供端到端的容错性）：

**Socket source 套接字源**

The socket source allows you to send data to your Streams via TCP sockets. To start one, specify a host and port to read data from. Spark will open a new TCP connection to read from that address. The socket source should not be used in production because the socket sits on the driver and does not provide end-to-end fault-tolerance guarantees. 

套接字源允许您通过TCP套接字向流发送数据。要启动一个，请指定要从中读取数据的主机和端口。Spark 将打开一个新的TCP连接来读取该地址。在生产中不应使用套接字源，因为套接字位于驱动程序上，并且不提供端到端的容错保证。 

Here is a short example of setting up this source to read from localhost:9999: 

以下是设置此源以从 localhost:9999 读取的简短示例：

```scala
// in Scala
val socketDF = spark.readStream.format("socket")
.option("host", "localhost").option("port", 9999).load()
```

```python
# in Python
socketDF = spark.readStream.format("socket")\
.option("host", "localhost").option("port", 9999).load()
```

If you’d like to actually write data to this application, you will need to run a server that listens on port 9999. On Unix-like systems, you can do this using the NetCat utility, which will let you type text into the first connection that is opened to port 9999. Run the command below before starting your Spark application, then write into it: 

```shell
nc -lk 9999
```

如果您真的想向这个应用程序写入数据，您需要运行一个在端口 9999 上监听的服务器。在类似Unix的系统上，您可以使用 NetCat 工具来执行此操作，该工具允许您在打开到端口 9999 的第一个连接中键入文本。在启动Spark应用程序之前运行下面的命令，然后写入它：

```shell
nc -lk 9999 
```

The socket source will return a table of text strings, one per line in the input data.

套接字源将返回一个文本字符串表，在输入数据中每行一个。

**Console sink 控制台接收器**

The console sink allows you to write out some of your streaming query to the console. This is useful for debugging but is not fault-tolerant. Writing out to the console is simple and only prints some rows of your streaming query to the console. This supports both append and complete output modes: 

控制台接收器允许您向控制台写出一些流式查询。这对调试很有用，但不是容错的。写入控制台很简单，只将流式查询的一些行打印到控制台。这支持附加和完整输出模式：

```scala
activityCounts.format("console").write()
```

**Memory sink 内存接收器**

The memory sink is a simple source for testing your streaming system. It’s similar to the console sink except that rather than printing to the console, it collects the data to the driver and then makes the data available as an in-memory table that is available for interactive querying. This sink is not fault tolerant, and you shouldn’t use it in production, but is great for testing and querying your stream during development. This supports both append and complete output modes:

内存接收器是测试流系统的简单源。它类似于控制台接收器，只是它不是打印到控制台，而是将数据收集到驱动程序，然后将数据作为内存中的表提供，以用于交互式查询。这个接收器不是容错的，您不应该在生产中使用它，但是对于开发期间测试和查询流非常有用。这支持附加和完整输出模式：

If you do want to output data to a table for interactive SQL queries in production, the authors recommend using the Parquet file sink on a distributed file system (e.g., S3). You can then query the data from any Spark application.

如果您确实希望将数据输出到生产中用于交互式 SQL 查询的表中，那么作者建议在分布式文件系统（如S3）上使用 Parquet 文件接收器。然后您可以从任何 Spark 应用程序查询数据。

### <font color="#000000">How Data Is Output (Output Modes) 如何输出数据（输出模式）</font>

Now that you know where your data can go, let’s discuss how the result Dataset will look when it gets there. This is what we call the output mode. As we mentioned, they’re the same concept as save modes on static DataFrames. There are three modes supported by Structured Streaming. Let’s look at each of them.

既然您已经知道了数据的去向，那么让我们讨论一下当结果数据集到达那里时它将如何显示。这就是我们所说的输出模式。正如我们提到的，它们与静态数据帧上的保存模式是相同的概念。Structured Streaming 支持三种模式。让我们看看每一个。

#### <font color="0a7ae5">Append mode 附加模式</font>

Append mode is the default behavior and the simplest to understand. When new rows are added to the result table, they will be output to the sink based on the trigger (explained next) that you specify. This mode ensures that each row is output once (and only once), assuming that you have a fault-tolerant sink. When you use append mode with event-time and watermarks (covered in Chapter 22), only the final result will output to the sink.

附加模式是默认行为，也是最容易理解的。当新的行添加到结果表中时，它们将根据您指定的触发器（解释如下）输出到接收器。此模式确保每行输出一次（并且仅输出一次），前提是您有一个容错接收器。在事件时间和水印（见第22章）中使用附加模式时，只有最终结果才会输出到接收器。

#### <font color="0a7ae5">Complete mode 完整模式</font>

Complete mode will output the entire state of the result table to your output sink. This is useful when you’re working with some stateful data for which all rows are expected to change over time or thesink you are writing does not support row-level updates. Think of it like the state of a stream at the time the previous batch had run.

完成模式将结果表的整个状态输出到输出接收器。当您处理的某些状态数据的所有行都会随着时间的推移而改变，或者您正在写入的链接不支持行级更新时，这非常有用。把它想象成前一批处理运行时的流状态。

#### <font color="0a7ae5">Update mode 更新模式</font>

Update mode is similar to complete mode except that only the rows that are different from the previous write are written out to the sink. Naturally, your sink must support row-level updates to support this mode. If the query doesn’t contain aggregations, this is equivalent to append mode.

更新模式与完成模式类似，除了不同于上一次写入的行会被写入接收器。当然，您的接收器必须支持行级更新才能支持此模式。如果查询不包含聚合，则这相当于追加模式。

#### <font color="0a7ae5">When can you use each mode? 你什么时候可以使用每种模式？</font>

Structured Streaming limits your use of each mode to queries where it makes sense. For example, if your query just does a map operation, Structured Streaming will not allow complete mode, because this would require it to remember all input records since the start of the job and rewrite the whole output table. This requirement is bound to get prohibitively expensive as the job runs. We will discuss when each mode is supported in more detail in the next chapter, once we also cover event-time processing and watermarks. If your chosen mode is not available, Spark Streaming will throw an exception when you start your stream.

Structured Streaming 将每种模式的使用限制为有意义的查询。例如，如果查询只是执行映射操作，Structured Streaming 将不允许使用完整模式，因为这将要求它记住自作业开始以来的所有输入记录并重写整个输出表。随着作业的运行，这一要求必然会变得非常昂贵。在下一章中，我们将更详细地讨论支持每种模式的时间，我们也将讨论事件时间处理和水印。如果您选择的模式不可用，当您启动流时，Spark流将抛出一个异常。

Here’s a handy table from the documentation that lays all of this out. Keep in mind that this will change in the future, so you’ll want to check the documentation for the most up-to-date version.

这是文档中列出所有这些内容的一个方便查询的表。请记住，这将在将来发生变化，因此您需要检查文档中的最新版本。

Table 21-1 shows when you can use each output mode.

表21-1显示了您何时可以使用每个输出模式。

| Query Type                              | Query type<br/>(continued)                                   | Supported<br/>Output<br/>Modes   | Notes                                                        |
| --------------------------------------- | ------------------------------------------------------------ | -------------------------------- | ------------------------------------------------------------ |
| Queries with<br/>aggregation            | Aggregation on event-time with watermark<br />按照有水印的事件时间来聚合 | Append,<br/>Update,<br/>Complete | Append mode uses watermark to drop old aggregation state. This means that as new rows are brought into the table, Spark will only keep around rows that are below the “watermark”. <br />Update mode also uses the watermark to remove old aggregation state. <br />By definition, complete mode does not drop old aggregation state since this mode preserves all data in the Result Table.<br />附加模式使用水印删除旧的聚合状态。这意味着当新的行被放入表中时，spark将只保留在“水印”下的行。更新模式还使用水印删除旧的聚合状态。根据定义，完整模式不会删除旧的聚合状态，因为此模式将保留结果表中的所有数据。 |
|                                         | Other<br/>aggregations                                       | Complete,<br/>Update             | Since no watermark is defined (only defined in other category), old aggregation state is not dropped. Append mode is not supported as aggregates can update thus violating the semantics of this mode.<br />由于未定义水印（仅在其他类别中定义），因此不会删除旧的聚合状态。不支持追加模式，因为聚合可能更新，从而违反了此模式的语义。 |
| Queries with<br/>mapGroupsWithState     |                                                              | Update                           |                                                              |
| Queries with<br/>flatMapGroupsWithState | Append<br/>operation<br/>mode                                | Append                           | Aggregations are allowed after flatMapGroupsWithState.       |
|                                         | Update<br/>operation<br/>mode                                | Update                           | Aggregations not allowed after flatMapGroupsWithState.       |
| Other queries                           |                                                              | Append,<br/>Update               | Complete mode not supported as it is infeasible to keep all unaggregated data in the Result Table.<br />不支持完整模式，因为无法将所有未聚合的数据保留在结果表中。 |

### <font color="#000000">When Data Is Output (Triggers) 数据输出时（触发器）</font>

To control when data is output to our sink, we set a *trigger*. By default, Structured Streaming will start data as soon as the previous trigger completes processing. You can use triggers to ensure that you do not overwhelm your output sink with too many updates or to try and control file sizes in the output. Currently, there is one periodic trigger type, based on processing time, as well as a “once” trigger to manually run a processing step once. More triggers will likely be added in the future.

为了控制数据何时输出到接收器，我们设置了一个触发器。默认情况下，Structured Streaming 将在上一个触发器完成处理后立即启动数据。您可以使用触发器来确保不会用太多的更新覆盖了输出接收器，或者尝试控制输出中的文件大小。目前，有一种基于处理时间的定期触发器类型，以及一种“一次”触发器，用于手动运行一次处理步骤。未来可能会增加更多的触发器。

#### <font color="0a7ae5">Processing time trigger 处理时间触发器</font>

For the processing time trigger, we simply specify a duration as a string (you may also use a Duration in Scala or TimeUnit in Java). We’ll show the string format below.

对于处理时间触发器，我们只需将一个持续时间指定为一个字符串（您也可以在 Java 中使用 TimeUnit 或 在 Scala 中使用 Duration）。我们将在下面显示字符串格式。

```scala
// in Scala
import org.apache.spark.sql.streaming.Trigger
activityCounts.writeStream.trigger(Trigger.ProcessingTime("100 seconds"))
.format("console").outputMode("complete").start()
```

```python
# in Python
activityCounts.writeStream.trigger(processingTime='5 seconds')\.format("console").outputMode("complete").start()
```

The <font face="constant-width" size=4>ProcessingTime</font> trigger will wait for multiples of the given duration in order to output data. For example, with a trigger duration of one minute, the trigger will fire at 12:00, 12:01, 12:02, and so on. If a trigger time is missed because the previous processing has not yet completed, then Spark will wait until the next trigger point (i.e., the next minute), rather than firing immediately after the previous processing completes.

ProcessingTime 触发器将等待数倍的给定的持续时间以输出数据。例如，触发器持续时间为一分钟，触发器将在12:00、12:01、12:02等时间触发。如果由于前一个处理尚未完成而错过触发时间，则 Spark 将等待下一个触发点（即下一分钟），而不是在前一个处理完成后立即触发。

#### <font color="0a7ae5">Once trigger 一次触发</font>

You can also just run a streaming job once by setting that as the trigger. This might seem like a weird case, but it’s actually extremely useful in both development and production. During development, you can test your application on just one trigger’s worth of data at a time. During production, the Once trigger can be used to run your job manually at a low rate (e.g., import new data into a summary table just occasionally). Because Structured Streaming still fully tracks all the input files processed and the state of the computation, this is easier than writing your own custom logic to track this in a batch job, and [saves a lot of resources over running a continuous job 24/7](https://bit.ly/2BuQUSR) :

您也可以通过将其设置为触发器来运行一次流作业。这看起来是一个奇怪的使用案例，但实际上在开发和生产中都非常有用。在开发过程中，一次只能在一个触发器的数据上测试应用程序。在生产过程中，可以使用ONCE触发器以较低的速度手动运行作业（例如，有时将新数据导入摘要表）。由于 Structured Streaming 处理仍然完全跟踪所有处理的输入文件和计算状态，因此这比在批处理作业中编写自己的自定义逻辑来跟踪这一点要容易得多，并且[在每天24小时每周七天的连续作业的运行过程中节省了大量资源](https://bit.ly/2BuQUSR)：

```scala
// in Scala
import org.apache.spark.sql.streaming.Trigger
activityCounts.writeStream.trigger(Trigger.Once())
.format("console").outputMode("complete").start()
```

```python
# in Python
activityCounts.writeStream.trigger(once=True)\
.format("console").outputMode("complete").start()
```

## <font color="#9a161a">Streaming Dataset API 流数据集API</font>

One final thing to note about Structured Streaming is that you are not limited to just the DataFrame API for streaming. You can also use Datasets to perform the same computation but in type-safe manner. You can turn a streaming DataFrame into a Dataset the same way you did with a static one. As before, the Dataset’s elements need to be Scala case classes or Java bean classes. Other than that, the DataFrame and Dataset operators work as they did in a static setting, and will also turn into a streaming execution plan when run on a stream. 

关于 Structured Streaming 的最后一点需要注意的是，您不仅限于流的 DataFrame  API。您也可以使用 Datasets 以类型安全的方式执行相同的计算。您可以像处理静态 Datasets 一样，将流式 DataFrame  转换为 Dataset 。如前所述，Dataset  的元素需要是 Scala case 类或 Java bean 类。除此之外，DataFrame 和 Dataset 算子（operators）的工作方式与静态设置中的相同，并且在流上运行时也将转变为流执行计划。

Here’s an example using the same dataset that we used in Chapter 11:

下面是使用第11章中使用的相同数据集的示例：

```scala
// in Scala
case class Flight(DEST_COUNTRY_NAME: String, ORIGIN_COUNTRY_NAME: String,
count: BigInt)

val dataSchema = spark.read
.parquet("/data/flight-data/parquet/2010-summary.parquet/")
.schema

val flightsDF = spark.readStream.schema(dataSchema)
.parquet("/data/flight-data/parquet/2010-summary.parquet/")
val flights = flightsDF.as[Flight]

def originIsDestination(flight_row: Flight): Boolean = {
	return flight_row.ORIGIN_COUNTRY_NAME == flight_row.DEST_COUNTRY_NAME
} 

flights.filter(flight_row => originIsDestination(flight_row))
.groupByKey(x => x.DEST_COUNTRY_NAME).count()
.writeStream.queryName("device_counts").format("memory").outputMode("complete")
.start()
```

## <font color="#9a161a">Conclusion 结论</font>

It should be clear that Structured Streaming presents a powerful way to write streaming applications. Taking a batch job you already run and turning it into a streaming job with almost no code changes is both simple and extremely helpful from an engineering standpoint if you need to have this job interact closely with the rest of your data processing application. Chapter 22 dives into two advanced streaming-related concepts: event-time processing and stateful processing. Then, after that, Chapter 23 addresses what you need to do to run Structured Streaming in production.

很明显，Structured Streaming 为编写流应用程序提供了一种强大的方式。如果您需要在与数据处理应用程序的其余部分紧密联系，从工程的角度来看，将已经运行的批处理作业转换为几乎没有代码更改的流作业既简单又非常有用。第22章深入介绍了两个与流相关的高级概念：事件时间处理和状态处理。然后，在这之后，第23章介绍了在生产环境中运行 Structured Streaming 处理需要做什么。