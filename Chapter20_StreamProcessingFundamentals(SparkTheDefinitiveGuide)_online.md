---
title: 翻译 Chapter 20 Stream Processing Fundamentals
date: 2019-06-02
copyright: true
categories: English,中文
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 20 Stream Processing Fundamentals 流处理基础

Stream processing is a key requirement in many big data applications. As soon as an application computes something of value—say, a report about customer activity, or a new machine learning model —an organization will want to compute this result continuously in a production setting. As a result, organizations of all sizes are starting to incorporate stream processing, often even in the first version of a new application.

流处理是许多大数据应用程序的关键要求。一旦一个应用程序计算出一些有价值的东西，比如关于客户活动的报告，或者一个新的机器学习模型，一个组织就会想要在生产环境中连续计算这个结果。因此，各种规模的组织都开始合并流处理，甚至在新应用程序的第一个版本中也是如此。

Luckily, Apache Spark has a long history of high-level support for streaming. In 2012, the project incorporated Spark Streaming and its DStreams API, one of the first APIs to enable stream processing using high-level functional operators like map and reduce. Hundreds of organizations now use DStreams in production for large real-time applications, often processing terabytes of data per hour. Much like the Resilient Distributed Dataset (RDD) API, however, the DStreams API is based on relatively low-level operations on Java/Python objects that limit opportunities for higher-level optimization. Thus, in 2016, the Spark project added Structured Streaming, a new streaming API built directly on DataFrames that supports both rich optimizations and significantly simpler integration with other DataFrame and Dataset code. The Structured Streaming API was marked as stable in Apache Spark 2.2, and has also seen swift adoption throughout the Spark community.

幸运的是，Apache Spark 有很长的高级流支持历史。2012年，该项目整合了 Spark Streaming 及其 DStreams API，这是第一批使用诸如 Map 和 Reduce 之类的高级功能操作符实现流处理的 API 之一。数百个组织现在在生产中使用数据流来处理大型实时应用程序，通常每小时处理数兆字节的数据。**与弹性分布式数据集（RDD）API非常相似，但是，DStreams API 是基于 Java/Python 对象上相对较低级别的操作，这些对象限制了更高级别优化的机会。因此，在2016年，Spark项目添加了结构化流式处理（Structured Streaming），这是一种直接在 DataFrames 上构建的新流式API，它既支持丰富的优化，也支持与其他 DataFrame  和 Dataset 代码的显著简化集成**。结构化流式API在ApacheSpark2.2中被标记为稳定的，并且在整个Spark社区中也得到了迅速的采用。

In this book, we will focus only on the Structured Streaming API, which integrates directly with the DataFrame and Dataset APIs we discussed earlier in the book and is the framework of choice for writing new streaming applications. If you are interested in DStreams, many other books cover that API, including several dedicated books on Spark Streaming only, such as Learning Spark Streaming by Francois Garillot and Gerard Maas (O’Reilly, 2017). Much as with RDDs versus DataFrames, however, Structured Streaming offers a superset of the majority of the functionality of DStreams, and will often perform better due to code generation and the Catalyst optimizer. Before we discuss the streaming APIs in Spark, let’s more formally define streaming and batch processing. This chapter will discuss some of the core concepts in this area that we will need throughout this part of the book. It won’t be a dissertation on this topic, but will cover enough of the concepts to let you make sense of systems in this space. 

在本书中，我们将只关注结构化流式API（Structured Streaming API），它直接与本书前面讨论的 DataFrame  和 Dataset API 集成，是编写新流式应用程序的首选框架。如果您对 DStream 感兴趣，那么许多其他书籍都涉及该 API，其中包括一些专门的关于 Spark Streaming 的书籍，例如 Francois Garillot和Gerard Maas的 *Learning Spark Streaming（O'Reilly，2017）*。然而，**与RDD与 DataFrame  相比，结构化流提供了数据流大部分功能的超集，并且由于代码生成和Catalyst优化器，通常性能会更好。** 在讨论Spark中的流式API之前，让我们更正式地定义流式处理和批处理。本章将讨论本书这一部分中我们需要的这一领域的一些核心概念。这不是一篇关于这个主题的论文，但将涵盖足够多的概念，使您能够理解这个空间中的系统。

## <font color="#8e0012">What Is Stream Processing 什么是流处理? </font>

*Stream processing* is the act of continuously incorporating new data to compute a result. In stream processing, the input data is unbounded and has no predetermined beginning or end. It simply forms a series of events that arrive at the stream processing system (e.g., credit card transactions, clicks on a website, or sensor readings from Internet of Things [IoT] devices). User applications can then compute various queries over this stream of events (e.g., tracking a running count of each type of event or aggregating them into hourly windows). The application will output multiple versions of the result as it runs, or perhaps keep it up to date in an external “sink” system such as a key-value store. 

**流处理是不断合并新数据以计算结果的行为。在流处理中，输入数据是无边界的，没有预先确定的开始或结束。它只是形成一系列到达流处理系统的事件**（例如，信用卡交易、网站点击或物联网设备的传感器读数）。然后，**用户应用程序可以计算对该事件流的各种查询**（例如，跟踪每种类型事件的运行计数，或将其聚合到每小时一次的窗口中）。**应用程序将在运行时输出结果的多个版本，或者在外部的“接收器”系统（如键值存储）中使其事件保持最新。**

Naturally, we can compare streaming to *batch processing*, in which the computation runs on a fixedinput dataset. Oftentimes, this might be a large-scale dataset in a data warehouse that contains all the historical events from an application (e.g., all website visits or sensor readings for the past month). Batch processing also takes a query to compute, similar to stream processing, but only computes the result once.

当然，我们可以将流式处理与批处理进行比较，**在批处理中，计算运行在固定的输入数据集上。通常，这可能是数据仓库中的大型数据集，其中包含应用程序的所有历史事件**（例如，过去一个月的所有网站访问或传感器读数）。**批处理也需要一个查询来计算，类似于流处理，但只计算一次结果。**

Although streaming and batch processing sound different, in practice, they often need to work together. For example, streaming applications often need to *join* input data against a dataset written periodically by a batch job, and the *output* of streaming jobs is often files or tables that are queried in batch jobs. Moreover, any business logic in your applications needs to work consistently across streaming and batch execution: for example, if you have a custom code to compute a user’s billing amount, it would be harmful to get a different result when running it in a streaming versus batch fashion! To handle these needs, Structured Streaming was designed from the beginning to *interoperate easily* with the rest of Spark, including batch applications. Indeed, the Structured Streaming developers coined the term *continuous applications* to capture [end-to-end applications](https://databricks.com/blog/2016/07/28/continuous-applications-evolving-streaming-in-apache-spark-2-0.html) that consist of streaming, batch, and interactive jobs all working on the same data to deliver an end product. Structured Streaming is focused on making it simple to build such applications in an end-to-end fashion instead of only handling stream-level per-record processing. 

**虽然流式处理和批处理听起来不同，但在实践中，它们通常需要一起工作。例如，流式处理应用程序通常需要将输入数据与批处理作业定期写入的数据集连接起来，而流式处理作业的输出通常是批处理作业中查询的文件或表**。此外，<font color="red">应用程序中的任何业务逻辑都需要在流式处理和批处理执行之间始终如一地工作：例如，如果您有一个自定义代码来计算用户的账单金额，那么以流式处理与批处理方式运行时获得不同的结果将是有害的！</font>为了处理这些需求，从一开始就设计了结构化流（Structured Streaming），以便与Spark的其余部分（包括批处理应用程序）轻松地进行互操作。**实际上，结构化流式开发人员创造了术语“连续应用程序”，以捕获由流式、批处理和交互式作业组成的端到端应用程序，这些作业都处理相同的数据以交付最终产品。结构化流的重点是使端到端的方式构建此类应用程序变得简单，而不是只应对流级别的每个记录处理**。

### <font color="#f68a1e">Stream Processing Use Cases 流处理使用案例</font>

We defined stream processing as the incremental processing of unbounded datasets, but that’s a strange way to motivate a use case. Before we get into advantages and disadvantages of streaming, let’s explain why you might want to use streaming. We’ll describe six common use cases with varying requirements from the underlying stream processing system.

我们将流处理定义为对无边界数据集的增量处理，但这是一种激活使用案例的奇怪方式。在我们讨论流的优点和缺点之前，**让我们解释一下为什么您可能想要使用流。我们将描述来自底层流处理系统的具有不同需求的六个常见用户案例。**

#### <font color="0a7ae5">Notifications and alerting 通知和警报</font>

Probably the most obvious streaming use case involves notifications and alerting. Given some series of events, a notification or alert should be triggered if some sort of event or series of events occurs. This doesn’t necessarily imply autonomous or preprogrammed decision making; alerting can also be used to notify a human counterpart of some action that needs to be taken. An example might be driving an alert to an employee at a fulfillment center that they need to get a certain item from a location in the warehouse and ship it to a customer. In either case, the notification needs to happen quickly.

可能最明显的流式用户案例涉及通知和警报。**对于某些事件系列，如果发生某种事件或事件系列，则应触发通知或警报。这并不一定意味着自主或预先编程的决策；警报也可以用来通知人类对应方需要采取的某些行动。例如，向履行中心的员工发出警报，提醒他们需要从仓库中的某个位置获取某个项目并将其发送给客户。在这两种情况下，通知都需要快速发生**。

#### <font color="0a7ae5">Real-time reporting 实时报告</font>

Many organizations use streaming systems to run real-time dashboards that any employee can look at. For example, this book’s authors leverage Structured Streaming every day to run real-time reporting dashboards throughout Databricks (where both authors of this book work). We use these dashboards to monitor total platform usage, system load, uptime, and even usage of new features as they are rolled out, among other applications.

**许多组织使用流媒体系统来运行任何员工都可以查看的实时仪表盘。**例如，本书的作者利用结构化流媒体每天在整个 Databricks（本书的两位作者都在这里工作）中运行实时报告仪表盘。**我们使用这些仪表盘来监控平台的总使用量、系统负载、正常运行时间，甚至在推出新功能时对它们的使用情况，以及其他应用程序**。

#### <font color="0a7ae5">Incremental ETL 增量的ETL</font>

One of the most common streaming applications is to reduce the latency companies must endure while retreiving information into a data warehouse—in short, “my batch job, but streaming.” Spark batch jobs are often used for Extract, Transform, and Load (ETL) workloads that turn raw data into a structured format like Parquet to enable efficient queries. Using Structured Streaming, these jobs can incorporate new data within seconds, enabling users to query it faster downstream. In this use case, it is critical that data is processed exactly once and in a fault-tolerant manner: we don’t want to lose any input data before it makes it to the warehouse, and we don’t want to load the same data twice. Moreover, the streaming system needs to make updates to the data warehouse transactionally so as not to confuse the queries running on it with partially written data.

**最常见的流式应用程序之一是减少公司在将信息检索到数据仓库（简而言之，“我的批处理作业，但流式处理”）时必须忍受的延迟。Spark批处理作业通常用于提取、转换和加载（ETL）的工作，将原始数据转换为类似 Parquet的结构化格式，以实现高效查询**。使用结构化流，这些作业可以在几秒钟内合并新数据，使用户能够更快地向下游查询数据。**在这个用户案例中，以一种容错的方式处理数据且有且仅且处理一次是非常关键的：我们不希望在数据到达仓库之前丢失任何输入数据，也不希望加载相同的数据两次。**此外，流媒体系统需要对数据仓库进行事务性更新，以免将运行在数据仓库上的查询与部分写入的数据混淆。

#### <font color="0a7ae5">Update data to serve in real time 更新数据以提供实时服务</font>

Streaming systems are frequently used to compute data that gets served interactively by another application. For example, a web analytics product such as Google Analytics might continuously track the number of visits to each page, and use a streaming system to keep these counts up to date. When users interact with the product’s UI, this web application queries the latest counts. Supporting this use case requires that the streaming system can perform incremental updates to a key–value store (or other serving system) as a sync, and often also that these updates are transactional, as in the ETL case, to avoid corrupting the data in the application. 

**流系统通常用于计算由其他应用程序交互提供服务的数据**。例如，像Google Analytics这样的Web Analytics产品可能会持续跟踪每个页面的访问次数，并使用流式系统保持这些计数的最新。当用户与产品的UI交互时，此Web应用程序查询最新计数。**支持此用户案例要求流系统可以同步对键值存储（或其他服务系统）执行增量更新，并且通常这些更新是事务性的，如ETL情况，以避免损坏应用程序中的数据**。

#### <font color="0a7ae5">Real-time decision making 实时决策</font>

Real-time decision making on a streaming system involves analyzing new inputs and responding to them automatically using business logic. An example use case would be a bank that wants to automatically verify whether a new transaction on a customer’s credit card represents fraud based on their recent history, and deny the transaction if the charge is determined fradulent. This decision needs to be made in real-time while processing each transaction, so developers could implement this business logic in a streaming system and run it against the stream of transactions. This type of application will likely need to maintain a significant amount of state about each user to track their current spending patterns, and automatically compare this state against each new transaction.

**流媒体系统的实时决策涉及到分析新的输入并使用业务逻辑自动响应它们**。例如，一家银行希望根据客户最近的历史自动验证其信用卡上的新交易是否代表欺诈行为，并在确定费用过期时拒绝该交易。这个决策需要在处理每个事务时实时做出，因此开发人员可以在流系统中实现这个业务逻辑，并针对事务流运行它。**这种类型的应用程序可能需要维护每个用户的大量状态**，以跟踪他们当前的支出模式，**并自动将这种状态与每个新事务进行比较**。

#### <font color="0a7ae5">Online machine learning 在线机器学习</font>

A close derivative of the real-time decision-making use case is online machine learning. In this scenario, you might want to train a model on a combination of streaming and historical data from multiple users. An example might be more sophisticated than the aforementioned credit card transaction use case: rather than reacting with hardcoded rules based on one customer’s behavior, the company may want to continuously update a model from all customers’ behavior and test each transaction against it. This is the most challenging use case of the bunch for stream processing systems because it requires aggregation across multiple customers, joins against static datasets, integration with machine learning libraries, and low-latency response times.

**实时决策用户案例的一个密切派生是在线机器学习**。在这个场景中，您可能希望对来自多个用户的流数据和历史数据组合的模型进行训练。一个例子可能比前面提到的信用卡交易用户案例更复杂：公司可能希望从所有客户的行为中不断更新一个模型，并根据它测试每个交易，而不是根据一个客户的行为对硬编码规则做出反应。**对于流处理系统来说，这是最具挑战性的用户案例，因为它需要跨多个客户进行聚合、针对静态数据集进行连接、与机器学习库集成以及低延迟响应时间。**

### <font color="#f68a1e">Advantages of Stream Processing</font>

Now that we’ve seen some use cases for streaming, let’s crystallize some of the advantages of stream processing. For the most part, batch is much simpler to understand, troubleshoot, and write applications in for the majority of use cases. Additionally, the ability to process data in batch allows for vastly higher data processing throughput than many streaming systems. However, stream processing is essential in two cases. First, stream processing enables lower latency: when your application needs to respond quickly (on a timescale of minutes, seconds, or milliseconds), you will need a streaming system that can keep state in memory to get acceptable performance. Many of the decision making and alerting use cases we described fall into this camp. Second, stream processing can also be more efficient in updating a result than repeated batch jobs, because it automatically incrementalizes the computation. For example, if we want to compute web traffic statistics over the past 24 hours, a naively implemented batch job might scan all the data each time it runs, always processing 24 hours’ worth of data. In contrast, a streaming system can remember state from the previous computation and only count the new data. If you tell the streaming system to update your report every hour, for example, it would only need to process 1 hour’s worth of data each time (the new data since the last report). In a batch system, you would have to implement this kind of incremental computation by hand to get the same performance, resulting in a lot of extra work that the streaming system will automatically give you out of the box.

既然我们已经看到了流的一些用例，那么让我们具体说明流处理的一些优势。在大多数情况下，对于大多数用例，批处理更容易理解、排除故障和编写应用程序。此外，**批量处理数据的能力使得数据处理吞吐量大大高于许多流系统。然而，流处理在两种情况下是必要的。首先，流处理可以降低延迟：当应用程序需要快速响应（以分钟、秒或毫秒为时间刻度）时，您将需要一个流系统，它可以在内存中保持状态以获得可接受的性能。我们描述的许多决策和警报用例都属于这个阵营。第二，流处理在更新结果方面也比重复的批处理作业更有效，因为它会自动增加计算量。**例如，如果我们想计算过去24小时内的Web流量统计数据，一个简单实现的批处理作业可能会在每次运行时扫描所有数据，总是处理24小时的数据。相反，**流系统可以记住以前计算的状态，并且只计算新数据**。例如，如果您告诉流系统每小时更新一次报告，则每次只需要处理1小时的数据（自上次报告以来的新数据）。**在批处理系统中，您必须手工实现这种增量计算，以获得相同的性能，从而导致流系统自动提供的大量额外工作**。

### <font color="#f68a1e">Challenges of Stream Processing</font>

We discussed motivations and advantages of stream processing, but as you likely know, there’s never a free lunch. Let’s discuss some of the challenges of operating on streams. To ground this example, let’s imagine that our application receives input messages from a sensor (e.g., inside a car) that report its value at different times. We then want to search within this stream for certain values, or certain patterns of values. One specific challenge is that the input records might arrive to our application out-of-order: due to delays and retransmissions, for example, we might receive the following sequence of updates in order, where the time field shows the time when the value was actually measured:

我们讨论了流处理的动机和优势，但正如您可能知道的，从来没有免费的午餐。让我们讨论在流上操作的一些挑战。为了使这个例子更加简单，让我们假设我们的应用程序从一个传感器（例如，在车内）接收输入消息，该传感器在不同的时间报告其值。然后我们希望在这个流中搜索特定的值或值的特定模式。**一个具体的挑战是，输入记录可能会无序到达我们的应用程序**：例如，由于延迟和重新传输，我们可能会按顺序接收以下更新序列，其中时间字段显示实际测量值的时间：

```json
{value: 1, time: "2017-04-07T00:00:00"}
{value: 2, time: "2017-04-07T01:00:00"}
{value: 5, time: "2017-04-07T02:00:00"}
{value: 10, time: "2017-04-07T01:30:00"}
{value: 7, time: "2017-04-07T03:00:00"}
```

In any data processing system, we can construct logic to perform some action based on receiving the single value of “5.” In a streaming system, we can also respond to this individual event quickly. However, things become more complicated if you want only to trigger some action based on a specific sequence of values received, say, 2 then 10 then 5. In the case of batch processing, this is not particularly difficult because we can simply sort all the events we have by time field to see that 10 did come between 2 and 5. However, this is harder for stream processing systems. The reason is that the streaming system is going to receive each event individually, and will need to track some state across events to remember the 2 and 5 events and realize that the 10 event was between them. The need to remember such state over the stream creates more challenges. For instance, what if you have a massive data volume (e.g., millions of sensor streams) and the state itself is massive? What if a machine in the sytem fails, losing some state? What if the load is imbalanced and one machine is slow? And how can your application signal downstream consumers when analysis for some event is “done” (e.g., the pattern 2-10-5 did not occur)? Should it wait a fixed amount of time or remember some state indefinitely? All of these challenges and others—such as making the input and the output of the system transactional—can come up when you want to deploy a streaming application.

在任何数据处理系统中，我们都可以在接收到单个值“5”的基础上构造逻辑来执行某些操作。在流系统中，我们还可以快速响应这个单独的事件。但是，**如果您只想根据接收到的特定值序列（例如2，然后10，然后5）触发一些操作，那么事情会变得更加复杂**。在批处理的情况下，这并不特别困难，因为我们可以简单地按时间字段对所有事件进行排序，以查看10是否在2到5之间。然而，对于流处理系统来说，这是很困难的。**原因是流系统将单独接收每个事件，并且需要跨事件跟踪一些状态，以记住值为2和5的事件，并认识到值为10的事件介于两者之间。需要记住这样的状态会带来更多的挑战。例如，如果你有一个巨大的数据量（例如，数百万个传感器流），而状态本身也是巨大的呢？如果系统中的一台机器发生故障，失去某种状态，会怎么样？如果负载不平衡，一台机器运行缓慢怎么办？当某些事件的分析“完成”时（例如，模式2-10-5没有发生），您的应用程序如何向下游消费者发出信号？它应该等待固定的时间还是无限期地记住某个状态？当您想要部署流式应用程序时，所有这些挑战和其他挑战（如使系统事务性的输入和输出）都会出现。

To summarize, the challenges we described in the previous paragraph and a couple of others, are as follows:

**综上所述，我们在上一段和其他几段中描述的挑战如下**：

- Processing out-of-order data based on application timestamps (also called event time)

    基于应用程序时间戳（也称为事件时间）处理无序数据

- Maintaining large amounts of state

    维持大量的状态

- Supporting high-data throughput

    支持大数据吞吐量

- Processing each event exactly once despite machine failures

    尽管机器出现故障，但每个事件处理一次

- Handling load imbalance and straggler

    处理负载不平衡和散乱

- Responding to events at low latency

    响应低延迟事件

- Joining with external data in other storage systems

    与其他存储系统中的外部数据连接

- Determining how to update output sinks as new events arrive

    确定新事件到达时如何更新输出接收器

- Writing data transactionally to output systems

    以事务方式将数据写入输出系统

- Updating your application’s business logic at runtime

    在运行时更新应用程序的业务逻辑

Each of these topics are an active area of research and development in large-scale streaming systems. To understand how different streaming systems have tackled these challenges, we describe a few of the most common design concepts you will see across them. 

**这些课题中的每一个都是大规模流系统研究和开发的活跃领域。**为了了解不同的流系统如何应对这些挑战，我们将介绍一些您将看到的最常见的设计概念。

## <font color="#8e0012">Stream Processing Design Points 流处理设计点</font>

To support the stream processing challenges we described, including high throughput, low latency, and out-of-order data, there are multiple ways to design a streaming system. We describe the most common design options here, before describing Spark’s choices in the next section.

为了支持我们描述的流处理挑战，包括高吞吐量、低延迟和无序数据，设计流系统有多种方法。在下一节描述Spark的选择之前，我们先在这里描述最常见的设计选项。

### <font color="#f68a1e">Record-at-a-Time Versus Declarative APIs 记录一次与声明性API</font>

The simplest way to design a streaming API would be to just pass each event to the application and let it react using custom code. This is the approach that many early streaming systems, such as Apache Storm, implemented, and it has an important place when applications need full control over the processing of data. Streaming that provide this kind of record-at-a-time API just give the user a collection of “plumbing” to connect together into an application. However, the downside of these systems is that most of the complicating factors we described earlier, such as maintaining state, are solely governed by the application. For example, with a record-at-a-time API, you are responsible for tracking state over longer time periods, dropping it after some time to clear up space, and responding differently to duplicate events after a failure. Programming these systems correctly can be quite challenging. At its core, low-level APIs require deep expertise to be develop and maintain.

**设计流式API的最简单方法是将每个事件传递给应用程序，并让它使用自定义代码进行响应。这是许多早期流媒体系统（如 Apache Storm）实现的方法，在应用程序需要完全控制数据处理时**，它具有重要的地位。**流式处理提供了这种一次记录的API，它只给用户一个“管道”集合，将它们连接到一个应用程序中。但是，这些系统的缺点是，我们前面描述的大多数复杂因素（如维护状态）都是由应用程序单独控制的**。例如，对于记录一次的API，您负责在较长的时间段内跟踪状态，在一段时间后将其丢弃以清除空间，并对失败后的重复事件做出不同的响应。对这些系统进行正确的编程是非常有挑战性的。核心层面来说，低阶API需要深厚的专业知识去执行开发和维护。

As a result, many newer streaming systems provide declarative APIs, where your application specifies what to compute but not how to compute it in response to each new event and how to recover from failure. Spark’s original DStreams API, for example, offered functional API based on operations like map, reduce and filter on streams. Internally, the DStream API automatically tracked how much data each operator had processed, saved any relevant state reliably, and recovered the computation from failure when needed. Systems such as Google Dataflow and Apache Kafka Streams provide similar, functional APIs. Spark’s Structured Streaming actually takes this concept even further, switching from functional operations to relational (SQL-like) ones that enable even richer automatic optimization of the execution without programming effort.

因此，**许多较新的流系统都提供声明性API（declarative APIs），其中应用程序指定要计算什么，而不是如何响应每个新事件以及如何从故障中恢复**。例如，Spark 最初的 DStreams API 提供了基于 map、reduce 和 filter 等操作的函数式 API。在内部，DStream API自动跟踪每个操作员处理了多少数据，可靠地保存了任何相关状态，并在需要时从失败中恢复计算。 **Google Dataflow 和 Apache Kafka 流等系统提供类似的功能性API。Spark的结构化流实际上更进一步地采用了这一概念，从功能操作转换为关系操作（类似于SQL），从而在不需要编程的情况下实现更丰富的自动执行优化。**

### <font color="#f68a1e">Event Time Versus Processing Time 事件时间与处理时间</font>

For the systems with declarative APIs, a second concern is whether the system natively supports event time. Event time is the idea of processing data based on timestamps inserted into each record at the source, as opposed to the time when the record is received at the streaming application (which is called processing time). In particular, when using event time, records may arrive to the system out of order (e.g., if they traveled back on different network paths), and different sources may also be out of sync with each other (some records may arrive later than other records for the same event time). If your application collects data from remote sources that may be delayed, such as mobile phones or IoT devices, event-time processing is crucial: without it, you will miss important patterns when some data is late. In contrast, if your application only processes local events (e.g., ones generated in the same datacenter), you may not need sophisticated event-time processing.

对于具有声明性API的系统，第二个问题是系统本身是否支持事件时间。**事件时间是基于在源位置插入到每个记录中的时间戳来处理数据的概念，而不是在流应用程序接收记录的时间（称为处理时间）**。特别是，在使用事件时间时，记录可能会无序到达系统（例如，如果它们返回到不同的网络路径上），并且不同的源也可能不同步（某些记录可能比同一事件时间的其他记录晚到达）。**如果您的应用程序从可能延迟的远程源（如移动电话或物联网设备）收集数据，事件时间处理至关重要：如果没有它，当某些数据延迟时，您将错过重要的模式。相反，如果应用程序只处理本地事件（例如，在同一个数据中心生成的事件），则可能不需要复杂的事件时间处理**。

When using event-time, several issues become common concerns across applications, including tracking state in a manner that allows the system to incorporate late events, and determining when it is safe to output a result for a given time window in event time (i.e., when the system is likely to have received all the input up to that point). Because of this, many declarative systems, including Structured Streaming, have “native” support for event time integrated into all their APIs, so that these concerns can be handled automatically across your whole program.

**当使用事件时间时，几个问题成为应用程序中常见的问题，包括以允许系统合并延迟事件的方式跟踪状态，以及在给定的事件时间窗口内，确定安全输出结果的时间**（即，系统可能具有接收到该点之前的所有输入）。因此，**许多声明性系统（包括结构化流）都支持将事件时间集成到其所有API中，以便在整个程序中自动处理这些问题**。

### <font color="#f68a1e">Continuous Versus Micro-Batch Execution 连续与微批量执行</font>

The final design decision you will often see come up is about continuous versus micro-batch execution. In continuous processing-based systems, each node in the system is continually listening to messages from other nodes and outputting new updates to its child nodes. For example, suppose that your application implements a map-reduce computation over several input streams. In a continuous processing system, each of the nodes implementing map would read records one by one from an input source, compute its function on them, and send them to the appropriate reducer. The reducer would then update its state whenever it gets a new record. The key idea is that this happens on each individual record, as illustrated in Figure 20-1. 

您经常会看到的最终设计决策是关于连续与微批量执行的。**在基于连续处理的系统中，系统中的每个节点都在不断地监听来自其他节点的消息，并将新的更新输出到其子节点。**例如，假设您的应用程序在多个输入流上实现了一个map reduce计算。在连续处理系统中，实现映射的每个节点将从输入源中逐个读取记录，计算其功能，并将其发送到相应的reducer。然后，每当有新的记录时，reducer就会更新其状态。关键的想法是这发生在每个单独的记录上，如图20-1所示。

![1564654336845](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter20/1564654336845.png)

Continuous processing has the advantage of offering the lowest possible latency when the total input rate is relatively low, because each node responds immediately to a new message. However, continuous processing systems generally have lower maximum throughput, because they incur a significant amount of overhead per-record (e.g., calling the operating system to send a packet to a downstream node). In addition, continous systems generally have a fixed topology of operators that cannot be moved at runtime without stopping the whole system, which can introduce load balancing issues.

**当总输入速率相对较低时，连续处理的优点是提供尽可能低的延迟，因为每个节点都会立即响应新消息。但是，连续处理系统通常具有较低的最大吞吐量，因为它们在每条记录上都会产生大量开销**（例如，调用操作系统向下游节点发送数据包）。**此外，连续系统通常具有固定的算子（ operator: [mathematics] a symbol or function which represents an operation in mathematics）拓扑结构，如果不停止整个系统，就无法在运行时移动这些算子，这会引入负载平衡问题**。

In contrast, micro-batch systems wait to accumulate small batches of input data (say, 500 ms’ worth), then process each batch in parallel using a distributed collection of tasks, similar to the execution of a batch job in Spark. Micro-batch systems can often achieve high throughput per node because they leverage the same optimizations as batch systems (e.g., vectorized processing), and do not incur any extra per-record overhead, as illustrated in Figure 20-2.

**相反，微批处理系统等待积累小批量的输入数据（比如500毫秒的值），然后使用分布式任务集合并行处理每一批，类似于Spark中批处理作业的执行。微批处理系统通常可以实现每个节点的高吞吐量，因为它们利用与批处理系统相同的优化（例如，矢量化处理），并且不会产生任何额外的每记录开销**，如图20-2所示。

![1564655645193](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter20/1564655645193.png)

Thus, they need fewer nodes to process the same rate of data. Micro-batch systems can also use dynamic load balancing techniques to handle changing workloads (e.g., increasing or decreasing the number of tasks). The downside, however, is a higher base latency due to waiting to accumulate a micro-batch. In practice, the streaming applications that are large-scale enough to need to *distribute* their computation tend to prioritize throughput, so Spark has traditionally implemented micro-batch processing. In Structured Streaming, however, there is an active development effort to also support a continuous processing mode beneath the same API.  

**因此，它们需要更少的节点来处理相同的数据速率。微批处理系统还可以使用动态负载平衡技术来处理不断变化的工作负载（例如，增加或减少任务数量）。但是，缺点是，由于等待积累一个微批处理，所以基本延迟更高。**在实践中，**流应用程序的规模足够大，需要分配它们的计算（资源），倾向于优先考虑吞吐量，因此Spark传统上已经实现了微批处理**。然而，**在结构化流中，也有一项积极的开发工作来支持同一API下的连续处理模式**。

When choosing between these two execution modes, the main factors you should keep in mind are your desired latency and total cost of operation (TCO). Micro-batch systems can comfortably deliver latencies from 100 ms to a second, depending on the application. Within this regime, they will generally require fewer nodes to achieve the same throughput, and hence lower operational cost (including lower maintenance cost due to less frequent node failures). For much lower latencies, you should consider a continuous processing system, or using a micro-batch system in conjunction with a fast serving layer to provide low-latency queries (e.g., outputting data into MySQL or Apache Cassandra, where it can be served to clients in milliseconds).

**在这两种执行模式之间进行选择时，您应该记住的主要因素是所需的延迟和总操作成本（TCO: total cost of operation）。根据应用程序的不同，微批处理系统可以轻松地将延迟从100毫秒传递到一秒钟。在这种情况下，它们通常需要更少的节点来实现相同的吞吐量，从而降低运营成本（包括由于节点故障频率较低而导致的维护成本）。对于较低的延迟，您应该考虑使用连续处理系统，或者将微批处理系统与快速服务层结合使用，以提供低延迟查询**（例如，将数据输出到 MySQL 或 Apache Cassandra 中，在那里它可以以毫秒为单位提供给客户机）。

In general, Structured Streaming is meant to be an easier-to use and higher-performance evolution of Spark Streaming’s DStream API, so we will focus solely on this new API in this book. Many of the concepts, such as building a computation out of a graph of transformations, also apply to DStreams, but we leave the exposition of that to other books.

## <font color="#8e0012">Spark’s Streaming APIs Spark的流式API</font>

We covered some high-level design approaches to stream processing, but thus far we have not discussed Spark’s APIs in detail. Spark includes two streaming APIs, as we discussed at the beginning of this chapter. The earlier DStream API in Spark Streaming is purely micro-batch oriented. It has a declarative (functional-based) API but no support for event time. The newer Structured Streaming API adds higher-level optimizations, event time, and support for continuous processing.

我们介绍了流处理的一些高级设计方法，但到目前为止，我们还没有详细讨论Spark的API。**Spark包含两个流式API**，正如我们在本章开头所讨论的。**Spark Streaming 中早期的 DStream API 纯粹是面向微批处理的。它有一个声明性（基于函数的）API，但不支持事件时间。更新的结构化流式API增加了更高级的优化、事件时间和对连续处理的支持。**

### <font color="#f68a1e">The DStream API 数据流API</font>

Spark’s original DStream API has been used broadly for stream processing since its first release in 2012. For example, DStreams was the most widely used processing engine in Datanami’s 2016 survey. Many companies use and operate Spark Streaming at scale in production today due to its highlevel API interface and simple exactly-once semantics. Interactions with RDD code, such as joins with static data, are also natively supported in Spark Streaming. Operating Spark Streaming isn’t much more difficult than operating a normal Spark cluster. However, the DStreams API has several limitations. First, it is based purely on Java/Python objects and functions, as opposed to the richer concept of structured tables in DataFrames and Datasets. This limits the engine’s opportunity to perform optimizations. Second, the API is purely based on processing time—to handle event-time operations, applications need to implement them on their own. Finally, DStreams can only operate in a micro-batch fashion, and exposes the duration of micro-batches in some parts of its API, making it difficult to support alternative execution modes.

Spark 的原始 DStream API 自2012年首次发布以来，已广泛用于流处理。例如，DStreams是Datanami 2016年调查中使用最广泛的处理引擎。由于其高级API接口和简单的一次性语义，许多公司现在在生产中大规模使用和操作Spark流。与 RDD 代码的交互（如与静态数据的连接）也在Spark流中得到原生支持。操作 Spark 流并不比操作普通的 Spark 集群更困难。但是，**DStreams API有几个限制。首先，它纯粹基于Java/Python 对象和函数，而不是 DataFrames 和 Datasets 中结构化表的更丰富的概念。这限制了引擎执行优化的机会。第二，API 纯粹是基于处理时间来处理事件时间操作，应用程序需要自己实现它们。最后，DStream 只能以微批量方式操作，并在其API的某些部分公开微批量的持续时间，这使得支持替代执行模式变得困难**。

### <font color="#f68a1e">Structured Streaming 结构化流</font>

Structured Streaming is a higher-level streaming API built from the ground up on Spark’s Structured APIs. It is available in all the environments where structured processing runs, including Scala, Java, Python, R, and SQL. Like DStreams, it is a declarative API based on high-level operations, but by building on the structured data model introduced in the previous part of the book, Structured Streaming can perform more types of optimizations automatically. However, unlike DStreams, Structured Streaming has native support for event time data (all of its the windowing operators automatically support it). As of Apache Spark 2.2, the system only runs in a micro-batch model, but the Spark team at Databricks has announced an effort called Continuous Processing to add a continuous execution mode. This should become an option for users in Spark 2.3.

**结构化流是在Spark的结构化API基础上构建的更高阶的流式API。它可以在结构化处理运行的所有环境中使用，包括Scala、Java、Python、R和SQL。与数据流一样，它是一个基于高阶操作的声明性API，但是通过构建本书上一部分介绍的结构化数据模型，结构化流可以自动执行更多类型的优化。但是，与数据流不同，结构化流具有对事件时间数据的原生支持（它的所有窗口化算子都自动支持它）。从 Apache Spark 2.2开始，系统只运行在一个微批量模型中，但 Databricks 的 Spark团队已经宣布了一项称为“连续处理”的工作，以添加连续执行模式。这应该成为 Spark 2.3 中用户的一个选项。**

More fundamentally, beyond simplifying stream processing, Structured Streaming is also designed to make it easy to build end-to-end continuous applications using Apache Spark that combine streaming, batch, and interactive queries. For example, Structured Streaming does not use a separate API from DataFrames: you simply write a normal DataFrame (or SQL) computation and launch it on a stream. Structured Streaming will automatically update the result of this computation in an incremental fashion as data arrives. This is a major help when writing end-to-end data applications: developers do not need to maintain a separate streaming version of their batch code, possibly for a different execution system, and risk having these two versions of the code fall out of sync. As another example, Structured Streaming can output data to standard sinks usable by Spark SQL, such as Parquet tables, making it easy to query your stream state from another Spark applications. In future versions of Apache Spark, we expect more and more components of the project to integrate with Structured Streaming, including online learning algorithms in MLlib.

**更重要的是，除了简化流处理之外，结构化流还设计为使用结合流、批处理和交互式查询的Apache Spark轻松构建端到端连续应用程序**。例如，**结构化流不使用不同于 DataFrames 的API：您只需编写一个普通的 DataFrame（或SQL）计算并在流上启动它。当数据到达时，结构化流将以增量方式自动更新计算结果。在编写端到端数据应用程序时**，主要的帮助是：可能是针对不同的执行系统，开发人员不需要维护其批处理代码的不同的流式版本，并且拥有两个版本的代码将有不同步的风险。作为另一个例子，结构化流可以将数据输出到 Spark Sql 可用的标准接收器，例如 Parquet 表，这样就可以很容易地从另一个Spark应用程序查询流状态。**在 Apache Spark的未来版本中，我们期望项目中越来越多的组件与结构化流集成，包括MLLIB中的在线学习算法**。

### <font color="#f68a1e">Conclusion 总结</font>

This chapter covered the basic concepts and ideas that you’re going to need to understand stream processing. The design approaches introduced in this chapter should clarify how you can evaluate streaming systems for a given application. You should also feel comfortable understanding what trade-offs the authors of DStreams and Structured Streaming have made, and why the direct support for DataFrame programs is a big help when using Structured Streaming: there is no need to duplicate your application logic. In the upcoming chapters, we’ll dive right into Structured Streaming to understand how to use it.

**本章介绍了理解流处理所需的基本概念和想法。本章介绍的设计方法应该阐明如何评估给定应用程序的流系统。您还应该理解数据流和结构化流的作者所做的权衡，以及为什么在使用结构化流时直接支持 DataFrame 程序是一个很大的帮助：不需要复制应用程序逻辑。**在接下来的章节中，我们将直接深入到结构化流，了解如何使用它。
