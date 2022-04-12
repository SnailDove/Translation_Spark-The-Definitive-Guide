---
title: 翻译 Charpter 19 Performance Tuning
date: 2019-08-13
copyright: true
categories: English,中文
tags: [Spark]
mathjax: true
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 19 Performance Tuning 性能调优

Chapter 18 covered the Spark user interface (UI) and basic first-aid for your Spark Application. Using the tools outlined in that chapter, you should be able to ensure that your jobs run reliably. However, sometimes you’ll also need them to run faster or more efficiently for a variety of reasons. That’s what this chapter is about. Here, we present a discussion of some of the performance choices that are available to make your jobs run faster.

第18章介绍了Spark用户界面（UI）和 Spark应用程序的基本急救。使用这一章中概述的工具，您应该能够确保工作可靠地运行。但是，有时出于各种原因，您还需要它们更快或更高效地运行。这就是本章的内容。在这里，我们将讨论一些性能选择，这些选择可以使您的工作运行得更快。

Just as with monitoring, there are a number of different levels that you can try to tune at. For instance, if you had an extremely fast network, that would make many of your Spark jobs faster because shuffles are so often one of the costlier steps in a Spark job. Most likely, you won’t have much ability to control such things; therefore, we’re going to discuss the things you can control through code choices or configuration.

正如监视一样，您可以尝试调到许多不同的级别。例如，如果你有一个非常快速的网络，这将使你的许多 Spark 工作更快，因为 shuffle（数据再分配）往往是 Spark 工作成本更高的步骤之一。最有可能的是，您没有太多的能力来控制这些事情；因此，我们将讨论通过代码选择或配置可以控制的事情

There are a variety of different parts of Spark jobs that you might want to optimize, and it’s valuable to be specific. Following are some of the areas:

Spark作业有许多不同的部分，您可能希望对其进行优化，具体来说很有价值。以下是一些领域：

- Code-level design choices (e.g., RDDs versus DataFrames)

    代码级设计选择（例如，RDD与数据帧）

- Data at rest

    非运行时数据（未流通的数据）

- Joins

    连接

- Aggregations

    聚合

- Data in flight

    程序运行中的数据

- Individual application properties

    单个应用程序属性

- Inside of the Java Virtual Machine (JVM) of an executor

    执行器的Java虚拟机（JVM）的内部

- Worker nodes

    工作节点

- Cluster and deployment properties

    集群和部署属性

This list is by no means exhaustive, but it does at least ground the conversation and the topics that we cover in this chapter. Additionally, there are two ways of trying to achieve the execution characteristics that we would like out of Spark jobs. We can either do so indirectly by setting configuration values or changing the runtime environment. These should improve things across Spark Applications or across Spark jobs. Alternatively, we can try to directly change execution characteristic or design choices at the individual Spark job, stage, or task level. These kinds of fixes are very specific to that one area of our application and therefore have limited overall impact. There are numerous things that lie on both sides of the indirect versus direct divide, and we will draw lines in the sand accordingly.

这个列表决不是详尽的，但它至少为我们在本章中讨论的话题打下了基础。此外，有两种方法可以尝试实现我们希望的无 Spark 作业的执行特性。我们可以通过设置配置值或更改运行时环境来间接地这样做。这些应该可以改善Spark应用程序或Spark作业的性能。或者，我们可以尝试在单个Spark作业、阶段或任务级别直接更改执行特性或设计选择。这些类型的修复程序非常特定于我们应用程序的某个领域，因此总体影响有限。间接与直接之间分水岭的两边有许多东西，我们将区分出来。

One of the best things you can do to figure out how to improve performance is to implement good monitoring and job history tracking. Without this information, it can be difficult to know whether you’re really improving job performance. 

要想了解如何提高绩效，最好的方法之一就是实施良好的监控和工作历史跟踪。没有这些信息，很难知道你是否真的在提高工作绩效。

## <font color="#8e0012">Indirect Performance Enforcement 间接的性能加强</font>

As discussed, there are a number of indirect enhancements that you can perform to help your Spark jobs run faster. We’ll skip the obvious ones like “improve your hardware” and focus more on the things within your control.

如前所述，您可以执行一些间接增强，以帮助 Spark 作业运行得更快。我们将跳过“改进硬件”这类明显的问题，更多地关注您控制范围内的事情。

### <font color="#00000">Design Choices 设计选择</font>

Although good design choices seem like a somewhat obvious way to optimize performance, we often don’t  prioritize this step in the process. When designing your applications, making good design choices is very important because it not only helps you to write better Spark applications but also to get them to run in a more stable and consistent manner over time and in the face of external changes or variations. We’ve already discussed some of these topics earlier in the book, but we’ll summarize some of the fundamental ones again here.

尽管良好的设计选择似乎是优化性能的一种比较明显的方法，但我们通常不会在流程中优先考虑这一步骤。在设计应用程序时，做出良好的设计选择是非常重要的，因为这不仅有助于编写更好的Spark应用程序，而且有助于使它们在一段时间内以更稳定和一致的方式运行，并在面对外部变化运行。我们已经在书的前面讨论了其中的一些主题，但是我们将在这里再次总结一些基本的主题。

### <font color="#000000">Scala versus Java versus Python versus R 语言的选择</font>

This question is nearly impossible to answer in the general sense because a lot will depend on your use case. For instance, if you want to perform some single-node machine learning after performing a large ETL job, we might recommend running your Extract, Transform, and Load (ETL) code as SparkR code and then using R’s massive machine learning ecosystem to run your single-node machine learning algorithms. This gives you the best of both worlds and takes advantage of the strength of R as well as the strength of Spark without sacrifices. As we mentioned numerous times, Spark’s Structured APIs are consistent across languages in terms of speed and stability. That means that you should code with whatever language you are most comfortable using or is best suited for your use case.

这个问题几乎不可能从一般情况下回答，因为很大程度上取决于您的使用案例。例如，如果您想在执行大型ETL作业后执行一些单节点机器学习，我们可能建议运行提取、转换和加载（ETL）代码作为 SparkR 代码，然后使用R的大型机器学习生态系统来运行单节点机器学习算法。这给了你两个世界中最好的资源，利用 R 和 Spark 的强大力量且不带（性能上的）牺牲。正如我们多次提到的，Spark的结构化API在速度和稳定性方面跨语言保持一致。这意味着您应该使用最适合您使用或最适合您的用例的任何语言进行编码。

Things do get a bit more complicated when you need to include custom transformations that cannot be created in the Structured APIs. These might manifest themselves as RDD transformations or user defined functions (UDFs). If you’re going to do this, R and Python are not necessarily the best choice simply because of how this is actually executed. It’s also more difficult to provide stricter guarantees of types and manipulations when you’re defining functions that jump across languages. We find that using Python for the majority of the application, and porting some of it to Scala or writing specific UDFs in Scala as your application evolves, is a powerful technique—it allows for a nice balance between overall usability, maintainability, and performance.

当需要包含无法在结构化API中创建的自定义转换时，情况确实会变得更加复杂。这些可能表现为RDD转换或用户自定义函数（UDF）。如果要这样做，R 和 Python不一定是最佳选择，仅仅因为它实际上是如何执行的。在定义跨语言的函数时，更难对类型和操作提供更严格的保证。我们发现，对大多数应用程序使用 Python，随着应用程序的发展，将其中的一部分移植到 Scala 或者在 Scala 中编写特定的 UDF，这是一种强大的技术，它允许在总体可用性、可维护性和性能之间实现良好的平衡。

### <font color="#000000">DataFrames versus SQL versus Datasets versus RDDs 不同层级的API选择</font>

This question also comes up frequently. The answer is simple. Across all languages, DataFrames, Datasets, and SQL are equivalent in speed. This means that if you’re using DataFrames in any of these languages, performance is equal. However, if you’re going to be defining UDFs, you’ll take a performance hit writing those in Python or R, and to some extent a lesser performance hit in Java and Scala. If you want to optimize for pure performance, it would behoove you to try and get back to DataFrames and SQL as quickly as possible. Although all DataFrame, SQL, and Dataset code compiles down to RDDs, Spark’s optimization engine will write “better” RDD code than you can manually and certainly do it with orders of magnitude less effort. Additionally, you will lose out on new optimizations that are added to Spark’s SQL engine every release.

这个问题也经常出现。答案很简单。在所有语言中，DataFrames、Datasets 和 SQL 在速度上是等效的。这意味着，如果您使用这些语言中的任何一种，那么性能都是相同的。但是，如果你要定义UDFs，你将在Python或R中进行性能损失，在一定程度上，Java和Scala的性能下降更少一些。如果您希望优化以获得纯粹的性能，那么应该尝试尽快返回到 DataFrames 和 SQL。尽管所有的数据框架、SQL和数据集代码都编译成RDD，但Spark的优化引擎将编写“更好”的RDD代码，这比您可以手动编写的代码要好，而且肯定要花费更少的工作量。此外，在每一个版本中，您都将失去添加到Spark的SQL引擎中的新优化。

Lastly, if you want to use RDDs, we definitely recommend using Scala or Java. If that’s not possible, we recommend that you restrict the “surface area” of RDDs in your application to the bare minimum. That’s because when Python runs RDD code, it’s serializes a lot of data to and from the Python process. This is very expensive to run over very big data and can also decrease stability.

最后，如果您想使用RDDs，我们绝对推荐使用Scala或Java。如果这不可能，我们建议您将应用程序中RDD的使用范围限制为最小值。这是因为当Python运行RDD代码时，它会在Python进程之间序列化大量数据。这是非常昂贵的运行非常大的数据，也可以降低稳定性。

Although it isn’t exactly relevant to performance tuning, it’s important to note that there are also some gaps in what functionality is supported in each of Spark’s languages. We discussed this in Chapter 16. 

尽管它与性能调优并不完全相关，但需要注意的是，在Spark的每种语言中，在支持哪些功能方面也存在一些差距。我们在第16章讨论了这一点。

### <font color="#000000">Shuffle Configurations 数据再分配的配置</font>

Configuring Spark’s external shuffle service (discussed in Chapters 16 and 17) can often increase performance because it allows nodes to read shuffle data from remote machines even when the executors on those machines are busy (e.g., with garbage collection). This does come at the cost of complexity and maintenance, however, so it might not be worth it in your deployment. Beyond configuring this external service, there are also a number of configurations for shuffles, such as the number of concurrent connections per executor, although these usually have good defaults. In addition, for RDD-based jobs, the serialization format has a large impact on shuffle performance—always prefer Kryo over Java serialization, as described in “Object Serialization in RDDs”.

配置Spark的外部shuffle服务（在第16和17章中讨论）通常可以提高性能，因为它允许节点从远程机器读取shuffle数据，即使这些机器上的执行器很忙（例如，垃圾回收）。然而，这是以复杂性和维护为代价的，因此在您的部署中可能不值得这样做。除了配置这个外部服务之外，还有许多配置用于 shuffle，例如每个 executor 的并发连接数，尽管这些配置通常具有良好的默认值。此外，对于基于 RDD 的作业，序列化格式对 shuffle 性能的影响很大，在 Java 序列化中总是首选 Kryo，如“RDDS中的对象序列化”中所描述的。

Furthermore, for all jobs, the number of partitions of a shuffle matters. If you have too few partitions, then too few nodes will be doing work and there may be skew, but if you have too many partitions, there is an overhead to launching each one that may start to dominate. Try to aim for at least a few tens of megabytes of data per output partition in your shuffle.

此外，对于所有作业，shuffle 的分区数都很重要。如果分区太少，那么节点就太少，可能会出现数据倾斜，但是如果分区太多，那么启动每个分区的开销就会开始占主导地位。试着在shuffle时，每个输出分区至少要有几十兆的数据。

### <font color="#000000">Memory Pressure and Garbage Collection 性能压力和垃圾回收</font>

During the course of running Spark jobs, the executor or driver machines may struggle to complete their tasks because of a lack of sufficient memory or “memory pressure.” This may occur when an application takes up too much memory during execution or when garbage collection runs too frequently or is slow to run as large numbers of objects are created in the JVM and subsequently garbage collected as they are no longer used. One strategy for easing this issue is to ensure that you’re using the Structured APIs as much as possible. These will not only increase the efficiency with which your Spark jobs will execute, but it will also greatly reduce memory pressure because JVM objects are never realized and Spark SQL simply performs the computation on its internal format. 

在运行spark作业的过程中，由于内存不足或“内存压力”，executor 或 driver 机器可能难以完成其任务。当应用程序在执行期间占用太多内存或垃圾回收运行太频繁时，可能会发生这种情况。或者运行起来很慢，因为在JVM中创建了大量的对象，然后由于不再使用这些对象而被垃圾回收。缓解这个问题的一个策略是确保尽可能多地使用结构化API。这些不仅可以提高Spark作业的执行效率，而且还可以极大地降低内存压力，因为JVM对象从未实现，Spark SQL只是对其内部格式执行计算。

The Spark documentation includes some great pointers on tuning garbage collection for RDD and
UDF based applications, and we paraphrase the following sections from that information. 

spark文档包含一些基于RDD和UDF的应用程序优化垃圾回收的重要建议，我们从这些信息中解释了以下部分。

#### <font color="0a7ae5">Measuring the impact of garbage collection 衡量垃圾回收的影响</font>

The first step in garbage collection tuning is to gather statistics on how frequently garbage collection occurs and the amount of time it takes. You can do this by adding `-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps` to Spark’s JVM options using the `spark.executor.extraJavaOptions` configuration parameter. The next time you run your Spark job, you will see messages printed in the worker’s logs each time a garbage collection occurs. These logs will be on your cluster’s worker nodes (in the stdout files in their work directories), not in the driver. 

GC调优的第一步是垃圾回收发生的频率和 GC 花费的时间。你可以通过配置参数：`spark.executor.extraJavaOptions` 添加 `-verbose：gc -XX:+ PrintGCDetails -XX:+ PrintGCTimeStamps` 到 Spark 的 Java 虚拟机选项来完成。下次你运行Spark作业时，每次发生垃圾回收时都会在 worker 的日志中看到消息。请注意，这些日志将位于集群的 worker 节点上（在其工作目录中的 stdout 文件中），而不是在驱动程序上。

**译者注**：[官网例子：](https://spark.apache.org/docs/latest/configuration.html#Dynamically-Loading-Spark-Properties)

```shell
./bin/spark-submit --name "My app" --master local[4] 
--conf spark.eventLog.enabled=false
--conf "spark.executor.extraJavaOptions=-XX:+PrintGCDetails -XX:+PrintGCTimeStamps" myApp.jar
```

#### <font color="0a7ae5">Garbage collection tuning 垃圾回收的调试</font>

To further tune garbage collection, you first need to understand some basic information about memory management in the JVM: Java heap space is divided into two regions: Young and Old. The Young generation is meant to hold short-lived objects whereas the Old generation is intended for objects with longer lifetimes.

为了进一步调整垃圾回收，首先需要了解 JVM 中内存管理的一些基本信息：Java 堆空间被分为两个区域：Young 和 Old。年轻一代的目的是持有生命周期短的对象，而老一代的目的在于生命周期较长的对象。

The Young generation is further divided into three regions: Eden, Survivor1, and Survivor2. Here’s a simplified description of the garbage collection procedure:

年轻一代被进一步划分为三个区域：Eden, Survivor1  和 Survivor2。以下是垃圾回收过程的简化描述：

1. When Eden is full, a minor garbage collection is run on Eden and objects that are alive from
    Eden and Survivor1 are copied to Survivor2.
    
    当 Eden 已满时，会在 Eden 上运行一个小量的垃圾回收，来自 Eden  和 Survivor1 的活动对象会被复制到Survivor2。

2. The Survivor regions are swapped.

    Survivor 区域交换。

3. If an object is old enough or if Survivor2 is full, that object is moved to Old.

    如果对象足够旧，或者 Survivor2 已满，则该对象将移动到 Old。

4. Finally, when Old is close to full, a full garbage collection is invoked. This involves tracing through all the objects on the heap, deleting the unreferenced ones, and moving the others to fill up unused space, so it is generally the slowest garbage collection operation. 

    最后，当 Old 接近满的时候，将调用完全的垃圾回收 。这涉及到跟踪堆上的所有对象，删除未引用的对象，并移动其他对象以填充未使用的空间，因此它通常是最慢的垃圾回收操作。

The goal of garbage collection tuning in Spark is to ensure that only long-lived cached datasets are stored in the Old generation and that the Young generation is sufficiently sized to store all short-lived objects. This will help avoid full garbage collections to collect temporary objects created during task execution. Here are some steps that might be useful. 

Spark中垃圾回收调优的目标是确保只有生命周期长的缓存数据集存储在老年代中，并且年轻一代的大小足以存储所有生命周期短的对象。这将有助于避免垃圾回收，以回收在任务执行期间创建的临时对象。 以下是一些可能有用的步骤。

Gather garbage collection statistics to determine whether it is being run too often. If a full garbage collection is invoked multiple times before a task completes, it means that there isn’t enough memory available for executing tasks, so you should decrease the amount of memory Spark uses for caching (`spark.memory.fraction`).

收集垃圾回收的统计信息以确定它是否运行得太频繁。如果在任务完成之前多次调用完全的垃圾回收，这意味着没有足够的内存用于执行任务，因此应该减少Spark用于缓存的内存量（`spark.memory.fraction`）。

If there are too many minor collections but not many major garbage collections, allocating more memory for Eden would help. You can set the size of the Eden to be an over-estimate of how much memory each task will need. If the size of Eden is determined to be E, you can set the size of the Young generation using the option `-Xmn=4/3*E`. (The scaling up by 4/3 is to account for space used by survivor regions, as well.)

如果有太多的小量的垃圾回收，但没有太多大量的垃圾回收，为 Eden 分配更多的内存会有所帮助。您可以将 Eden 的大小设置为对每个任务需要多少内存的过度估计（有冗余空间）。如果 Eden（伊甸园）的大小确定为 `E`，您可以使用选项 `-xmn=4/3*E` 设置年轻一代的大小（放大4/3也是为了说明 survivor 区域使用的空间）。

As an example, if your task is reading data from HDFS, the amount of memory used by the task can be estimated by using the size of the data block read from HDFS. Note that the size of a decompressed block is often two or three times the size of the block. So if you want to have three or four tasks’ worth of working space, and the HDFS block size is 128 MB, we can estimate size of Eden to be $4*3*128$ MB.

**例如**，如果您的任务是从 HDFS 读取数据，那么可以使用从 HDFS 读取的数据块的大小来估计任务使用的内存量。请注意，解压块的大小通常是块大小的两到三倍。因此，如果您想要三个或四个任务的工作空间，并且 HDFS块大小为 $128$ MB，我们可以估计 Eden 的大小为 $4 * 3 * 128$ MB。

Try the G1GC garbage collector with `-XX:+UseG1GC`. It can improve performance in some situations in which garbage collection is a bottleneck and you don’t have a way to reduce it further by sizing the generations. Note that with large executor heap sizes, it can be important to increase the G1 region size with `-XX:G1HeapRegionSize` .

使用 ` -xx:+useg1gc` 尝试 G1GC 垃圾回收器。在垃圾回收是一个瓶颈的情况下，并且您没有办法通过调整代的大小进一步降低它，这样做可以提高性能。请注意，拥有大的 executor 堆大小，使用
 `-xx:g1HeapRegionSize` 增加G1区大小可能很重要。

Monitor how the frequency and time taken by garbage collection changes with the new settings. Our experience suggests that the effect of garbage collection tuning depends on your application and the amount of memory available. There are many more tuning options described online, but at a high level, managing how frequently full garbage collection takes place can help in reducing the overhead. You can specify garbage collection tuning flags for executors by setting `spark.executor.extraJavaOptions` in a job’s configuration. 

监视垃圾回收所用的频率和时间如何随新设置的变化而变化。我们的经验表明，垃圾回收调优的效果取决于您的应用程序和可用内存量。线上描述了更多的调优选项，但从较高的层次来说，管理完全的垃圾回收的频率有助于减少开销。通过在作业配置中设置 `spark.executor.extraJavaOptions`，可以为 executor 指定垃圾回收的优化标志。

## <font color="#8e0012">Direct Performance Enhancements 直接性能增强</font>

In the previous section, we touched on some general performance enhancements that apply to all jobs. Be sure to skim the previous couple of pages before jumping to this section and the solutions here. These solutions here are intended as “band-aids” of sorts for issues with specific stages or jobs, but they require inspecting and optimizing each stage or job separately.

在前一节中，我们讨论了一些适用于所有作业的通用的性能增强。在跳到本节和这里的解决方案之前，一定要浏览前面的几页。这里的这些解决方案是针对特定阶段或工作的各种问题的“创可贴”，但它们需要分别检查和优化每个阶段或工作。

### <font color="#000000">Parallelism 并行主义</font>

The first thing you should do whenever trying to speed up a specific stage is to increase the degree of parallelism. In general, we recommend having at least two or three tasks per CPU core in your cluster if the stage processes a large amount of data. You can set this via the `spark.default.parallelism` property as well as tuning the `spark.sql.shuffle.partitions` according to the number of cores in your cluster.

当您试图加速某个特定阶段时，首先应该做的是提高并行度。通常，如果阶段处理大量数据，我们建议集群中每个CPU核心至少有两到三个任务。您可以通过` spark.default.parallelism `属性进行设置，也可以根据集群中核心的数量来调整 ` spark.sql.shuffle.partitions`。

### <font color="#000000">Improved Filtering 改进过滤</font>

Another frequent source of performance enhancements is moving filters to the earliest part of your Spark job that you can. Sometimes, these filters can be pushed into the data sources themselves and this means that you can avoid reading and working with data that is irrelevant to your end result. Enabling partitioning and bucketing also helps achieve this. Always look to be filtering as much data as you can early on, and you’ll find that your Spark jobs will almost always run faster.

性能增强的另一个常见来源是将过滤器移动到Spark作业的最早部分。有时，这些过滤器可以被推入数据源本身，这意味着您可以避免读取和处理与最终结果无关的数据。启用分区和bucketing也有助于实现这一点。 总是尽可能早地过滤数据，你会发现你的Spark作业几乎总是运行得更快。

### <font color="#000000">Repartitioning and Coalescing 重分区和联合</font>

Repartition calls can incur a shuffle. However, doing some can optimize the overall execution of a job by balancing data across the cluster, so they can be worth it. In general, you should try to shuffle the least amount of data possible. For this reason, if you’re reducing the number of overall partitions in a DataFrame or RDD, first try coalesce method, which will not perform a shuffle but rather merge partitions on the same node into one partition. The slower repartition method will also shuffle data across the network to achieve even load balancing. Repartitions can be particularly helpful when performing joins or prior to a cache call. Remember that repartitioning is not free, but it can improve overall application performance and parallelism of your jobs.

重新分区调用可能导致混乱。但是，通过在集群中平衡数据，可以优化作业的整体执行，因此它们是值得的。一般来说，您应该尽量减少数据量。因此，如果要减少 DataFrame 或 RDD 中的整体分区数，请首先尝试 `coalesce` 方法，它不会执行数据再分配（shuffle），而是将同一节点上的分区合并到一个分区中。较慢的重新分区方法还将在网络上数据再分配数据，以实现均匀的负载平衡。重新分区在执行连接或在缓存调用之前特别有用。记住，重新分区不是免费的，但它可以提高应用程序的整体性能和作业的并行性。

### <font color="#000000">Custom partitioning 自定义分区器</font>

If your jobs are still slow or unstable, you might want to explore performing custom partitioning at the RDD level. This allows you to define a custom partition function that will organize the data across the cluster to a finer level of precision than is available at the DataFrame level. This is very rarely necessary, but it is an option. For more information, see Part III. 

如果您的作业仍然很慢或不稳定，您可能希望探索在RDD级别执行自定义分区。这允许您定义一个自定义分区函数，该函数将跨集群组织数据，使其达到比 DataFrame 级别更高的精度级别。这是很少必要的，但它是一种选择。更多信息，见第三部分。

### <font color="#000000">User-Defined Functions (UDFs) 用户定义函数（UDF）</font>

In general, avoiding UDFs is a good optimization opportunity. UDFs are expensive because they force representing data as objects in the JVM and sometimes do this multiple times per record in a query. You should try to use the Structured APIs as much as possible to perform your manipulations simply because they are going to perform the transformations in a much more efficient manner than you can do in a high-level language. There is also ongoing work to make data available to UDFs in batches, such as the Vectorized UDF extension for Python that gives your code multiple records at once using a Pandas data frame. We discussed UDFs and their costs in Chapter 18.

一般来说，避免UDF是一个很好的优化机会。UDF之所以昂贵，是因为它们强制将数据表示为JVM中的对象，并且有时在查询中对每条记录执行多次这样的操作。您应该尽可能多地使用结构化API来执行操作，因为它们将以比高级语言更高效的方式执行转换。还有一些正在进行的工作是批量向UDF提供数据，例如针对Python的矢量化UDF扩展，它使用PANDAS数据帧一次为代码提供多个记录。我们在第18章讨论了UDF及其成本。

### <font color="#000000">Temporary Data Storage (Caching) 临时数据存储</font>

In applications that reuse the same datasets over and over, one of the most useful optimizations is caching. Caching will place a DataFrame, table, or RDD into temporary storage (either memory or disk) across the executors in your cluster, and make subsequent reads faster. Although caching might sound like something we should do all the time, it’s not always a good thing to do. That’s because caching data incurs a serialization, deserialization, and storage cost. For example, if you are only going to process a dataset once (in a later transformation), caching it will only slow you down. The use case for caching is simple: as you work with data in Spark, either within an interactive session or a standalone application, you will often want to reuse a certain dataset (e.g., a DataFrame or RDD). For example, in an interactive data science session, you might load and clean your data and then reuse it to try multiple statistical models. Or in a standalone application, you might run an iterative algorithm that reuses the same dataset. You can tell Spark to cache a dataset using the cache method on DataFrames or RDDs.

在反复重用相同数据集的应用程序中，最有用的优化之一是缓存。缓存将把一个DataFrame、表或RDD放到集群中执行器的临时存储器（内存或磁盘）中，并使后续的读取速度更快。尽管缓存听起来像是我们一直应该做的事情，但这并不总是一件好事。这是因为缓存数据会导致序列化、反序列化和存储成本。 例如，如果您只处理一次数据集（在以后的转换中），缓存它只会减慢您的速度。缓存的用例很简单：当您在Spark中处理数据时，无论是在交互会话中还是在独立的应用程序中，您通常都希望重用某个数据集（例如，DataFrame 或 RDD）。 例如，在交互式数据科学会话中，您可以加载和清理数据，然后重新使用它来尝试多个统计模型。或者在独立的应用程序中，您可以运行一个重复使用相同数据集的迭代算法。您可以告诉Spark在 DataFrame 或 RDD 上使用cache方法缓存数据集。

Caching is a lazy operation, meaning that things will be cached only as they are accessed. The RDD API and the Structured API differ in how they actually perform caching, so let’s review the gory details before going over the storage levels. When we cache an RDD, we cache the actual, physical data (i.e., the bits). The bits. When this data is accessed again, Spark returns the proper data. This is done through the RDD reference. However, in the Structured API, caching is done based on the physical plan. This means that we effectively store the physical plan as our key (as opposed to the object reference) and perform a lookup prior to the execution of a Structured job. This can cause confusion because sometimes you might be expecting to access raw data but because someone else already cached the data, you’re actually accessing their cached version. Keep that in mind when using this feature.

缓存是一个懒惰的操作，这意味着只有数据被访问时才会对其进行缓存。RDD API和结构化API在实际执行缓存的方式上有所不同，  因此，在讨论存储级别之前，让我们先回顾一下详细信息。<font color="red">当我们缓存一个RDD时，我们缓存实际的物理数据（也就是：比特）。当再次访问此数据时，spark返回正确的数据。这是通过RDD引用完成的。但是，在结构化API中，缓存是基于物理计划完成的。这意味着我们有效地将物理计划存储为键（而不是对象引用），并在执行结构化作业之前执行查找。这可能会导致混淆，因为有时您可能希望访问原始数据，但因为其他人已经缓存了数据，所以实际上您正在访问他们的缓存版本。使用此功能时请记住这一点。</font>

There are different storage levels that you can use to cache your data, specifying what type of storage to use. Table 19-1 lists the levels. 

您可以使用不同的存储级别来缓存数据，指定要使用的存储类型。表19-1列出了各等级。

Table 19-1. Data cache storage levels 数据缓存级别

| Storage level Meaning | Meaning                                                      |
| --------------------- | ------------------------------------------------------------ |
| MEMORY_ONLY           | Store RDD as deserialized Java objects in the JVM. If the RDD does not fit in memory, some partitions will not be cached and will be recomputed on the fly each time they’re needed. This is the default level. <br />将RDD存储为JVM中的反序列化Java对象。如果RDD不在内存中，那么某些分区将不会被缓存，并且将在每次需要时即时重新计算。这是默认级别。 |
|MEMORY_AND_DISK|Store RDD as deserialized Java objects in the JVM. If the RDD does not fit in memory, store the partitions that don’t fit on disk, and read them from there when they’re needed.<br />将RDD存储为JVM中的反序列化Java对象。如果RDD不适合内存，请将不适合的分区存储在磁盘上，并在需要时从磁盘上读取它们。|
|MEMORY_ONLY_SER<br />(Java and Scala)|Store RDD as serialized Java objects (one byte array per partition). This is generally more space-efficient than deserialized objects, especially when using a fast serializer, but more CPU-intensive to read.<br />将RDD存储为序列化的Java对象（每个分区的一个字节数组）。这通常比反序列化对象更节省空间，尤其是在使用快速序列化程序时，但读取时CPU占用更大。|
|DISK_ONLY|Store the RDD partitions only on disk.<br />仅将RDD分区存储在磁盘上。|
|MEMORY_ONLY_2,<br/>MEMORY_AND_DISK_2,<br/>etc.|Same as the previous levels, but replicate each partition on two cluster nodes.<br />与前面的级别相同，但在两个集群节点上复制每个分区。|
|OFF_HEAP<br/>(experimental)|Similar to MEMORY_ONLY_SER, but store the data in off-heap memory. This requires off-heap memory to be enabled.<br />类似于只存储数据，但将数据存储在堆外内存中。这需要启用堆外内存。|

For more information on these options, take a look at “Configuring Memory Management”.

有关这些选项的详细信息，请参阅“配置内存管理”。

Figure 19-1 presents a simple illustrations of the process. 

图19-1给出了该过程的简单说明。

We load an initial DataFrame from a CSV file and then derive some new DataFrames from it using transformations. We can avoid having to recompute the original DataFrame (i.e., load and parse the CSV file) many times by adding a line to cache it along the way.

我们从csv文件加载一个初始数据帧，然后使用转换从中派生一些新的数据帧。我们可以避免多次重新计算原始数据帧（即加载和解析csv文件），方法是添加一行缓存它。

![1564413229643](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter19/1564413229643.png)

Now let’s walk through the code:

现在让我们完整地学习一个代码：

```python
# in Python
# Original loading code that does *not* cache DataFrame
DF1 = spark.read.format("csv")\
.option("inferSchema", "true")\
.option("header", "true")\
.load("/data/flight-data/csv/2015-summary.csv")
DF2 = DF1.groupBy("DEST_COUNTRY_NAME").count().collect()
DF3 = DF1.groupBy("ORIGIN_COUNTRY_NAME").count().collect()
DF4 = DF1.groupBy("count").count().collect()
```

You’ll see here that we have our “lazily” created DataFrame (DF1), along with three other DataFrames that access data in DF1. All of our downstream DataFrames share that common parent (DF1) and will repeat the same work when we perform the preceding code. In this case, it’s just reading and parsing the raw CSV data, but that can be a fairly intensive process, especially for large datasets.

在这里，您将看到我们的“惰性”创建的数据帧（df1），以及其他三个访问df1中数据的数据帧。我们所有的下游数据帧都共享这个公共父级（DF1），并且在执行前面的代码时将重复相同的工作。在这种情况下，它只是读取和解析原始的csv数据，但这可能是一个相当密集的过程，特别是对于大型数据集。

On my machine, those commands take a second or two to run. Luckily caching can help speed things up. When we ask for a DataFrame to be cached, Spark will save the data in memory or on disk the first time it computes it. Then, when any other queries come along, they’ll just refer to the one stored in memory as opposed to the original file. You do this using the DataFrame’s cache method:

在我的机器上，这些命令需要一两秒钟才能运行。幸运的是，缓存可以帮助加快速度。当我们请求缓存一个 DataFrame 时，spark将在第一次计算数据时将数据保存在内存或磁盘上。然后，当出现任何其他查询时，它们只引用存储在内存中的查询，而不是原始文件。使用 DataFrame 的缓存方法执行此操作：

```python
DF1.cache()
DF1.count()
```

We used the count above to eagerly cache the data (basically perform an action to force Spark to store it in memory), because caching itself is lazy—the data is cached only on the first time you run an action on the DataFrame. Now that the data is cached, the previous commands will be faster, as we can see by running the following code:

我们使用上面的计数来急切地缓存数据（基本上是执行一个操作来强制 spark 将其存储在内存中），因为缓存本身是懒惰的，数据只在您第一次在 DataFrame 上运行一个操作时缓存。现在缓存了数据，前面的命令将更快，我们可以通过运行以下代码看到这一点：

```python
# in Python
DF2 = DF1.groupBy("DEST_COUNTRY_NAME").count().collect()
DF3 = DF1.groupBy("ORIGIN_COUNTRY_NAME").count().collect()
DF4 = DF1.groupBy("count").count().collect()
```

When we ran this code, it cut the time by more than half! This might not seem that wild, but picture a large dataset or one that requires a lot of computation to create (not just reading in a file). The savings can be immense. It’s also great for iterative machine learning workloads because they’ll often need to access the same data a number of times, which we’ll see shortly. 

当我们运行这个代码时，它将时间缩短了一半以上！这看起来并不是那么疯狂，但想象一下一个大型数据集或需要大量计算才能创建的数据集（不仅仅是在文件中读取）。节省的钱可能是巨大的。这对于迭代机器学习工作负载也很好，因为它们通常需要多次访问相同的数据，稍后我们将看到。

The cache command in Spark always places data in memory by default, caching only part of the dataset if the cluster’s total memory is full. For more control, there is also a persist method that takes a StorageLevel object to specify where to cache the data: in memory, on disk, or both. 

Spark中的cache命令在默认情况下总是将数据放在内存中，如果集群的总内存已满，则只缓存数据集的一部分。为了获得更多的控制权，还有一个持久化方法，它使用一个 StorageLevel 对象来指定数据的缓存位置：内存中、磁盘上，或者两者兼而有之。

### <font color="#000000">Joins 连接</font>

Joins are a common area for optimization. The biggest weapon you have when it comes to optimizing joins is simply educating yourself about what each join does and how it’s performed. This will help you the most. Additionally, equi-joins are the easiest for Spark to optimize at this point and therefore should be preferred wherever possible. Beyond that, simple things like trying to use the filtering ability of inner joins by changing join ordering can yield large speedups. Additionally, using broadcast join hints can help Spark make intelligent planning decisions when it comes to creating query plans, as described in Chapter 8. Avoiding Cartesian joins or even full outer joins is often low-hanging fruit for stability and optimizations because these can often be optimized into different filtering style joins when you look at the entire data flow instead of just that one particular job area.

连接是优化的一个常见领域。在优化连接时，您拥有的最大武器就是简单地向自己介绍每个连接的作用和执行方式。这对你的帮助最大。此外，equi-joins 对于spark来说是最容易在这一点上进行优化的，因此在可能的情况下应首选。除此之外，通过改变连接顺序来尝试使用内部连接的过滤能力等简单的事情可以产生很大的加速。此外，使用广播连接提示可以帮助Spark在创建查询计划时做出智能规划决策，如第8章所述。避免笛卡尔连接，甚至是完全的外部连接，对于稳定性和优化来说通常都是容易获得的成果，因为当您查看整个数据流而不仅仅是一个特定的工作区域时，这些连接常常可以优化为不同的过滤类型的连接。

Lastly, following some of the other sections in this chapter can have a significant effect on joins. For example, collecting statistics on tables prior to a join will help Spark make intelligent join decisions. Additionally, bucketing your data appropriately can also help Spark avoid large shuffles when joins are performed.

最后，遵循本章中的一些其他部分可以对连接产生显著的影响。例如，在连接之前收集表的统计信息将有助于Spark 做出智能连接决策。此外，适当地将数据进行分桶还可以帮助 Spark 在执行连接时避免大的数据再分配（shuffle）。

### <font color="#000000">Aggregations 聚合</font>

For the most part, there are not too many ways that you can optimize specific aggregations beyond filtering data before the aggregation having a sufficiently high number of partitions. However, if you’re using RDDs, controlling exactly how these aggregations are performed ( e.g., using `reduceByKey `when possible over `groupByKey` ) can be very helpful and improve the speed and stability of your code.

在大多数情况下，除了在聚合具有足够多的分区之前过滤数据之外，没有太多方法可以优化特定聚合。但是，如果您使用的是 RDD，那么准确地控制这些聚合的执行方式（例如，在可能的情况下使用 `reduceByKey` 而不是`groupByKey`）会非常有帮助，并且可以提高代码的速度和稳定性。

### <font color="#000000">Broadcast Variables 广播变量</font>

We touched on broadcast joins and variables in previous chapters, and these are a good option for optimization. The basic premise is that if some large piece of data will be used across multiple UDF calls in your program, you can broadcast it to save just a single read-only copy on each node and avoid re-sending this data with each job. For example, broadcast variables may be useful to save a lookup table or a machine learning model. You can also broadcast arbitrary objects by creating broadcast variables using your SparkContext, and then simply refer to those variables in your tasks, as we discussed in Chapter 14.

我们在前面的章节中讨论了广播连接和变量，这些是一个很好的优化选择。基本前提是，如果在程序中的多个UDF调用之间使用一些大的数据块，您可以广播它以在每个节点上只保存一个只读副本，并避免在每个作业中重新发送这些数据。例如，广播变量对于保存查找表或机器学习模型可能很有用。您还可以通过使用 SparkContext 创建广播变量来广播任意对象，然后在任务中简单地引用这些变量，如我们在第14章中所讨论的。

### <font color="#000000">Conclusion 结论</font>

There are many different ways to optimize the performance of your Spark Applications and make them run faster and at a lower cost. In general, the main things you’ll want to prioritize are (1) reading as little data as possible through partitioning and efficient binary formats, (2) making sure there is sufficient parallellism and no data skew on the cluster using partitioning, and (3) using high-level APIs such as the Structured APIs as much as possible to take already optimized code. As with any other software optimization work, you should also make sure you are optimizing the right operations for your job: the Spark monitoring tools described in Chapter 18 will let you see which stages are taking the longest time and focus your efforts on those. Once you have identified the work that you believe can be optimized, the tools in this chapter will cover the most important performance optimization opportunities for the majority of users.

有许多不同的方法来优化Spark应用程序的性能，使其以更低的成本更快地运行。一般来说，您要优先考虑的主要事情是

（1）通过分区和有效的二进制格式读取尽可能少的数据

（2）确保在使用分区的集群上有足够的并行性和没有数据倾斜

（3）尽可能多地使用 high-level APIs去采用已经优化过的代码，例如：结构化的API

与任何其他软件优化工作一样，您还应该确保为您的工作优化了正确的操作：第18章中描述的 Spark 监控工具将让您了解哪些阶段花费的时间最长，并将您的精力集中在这些阶段上。一旦确定了您认为可以优化的工作，本章中的工具将为大多数用户提供最重要的性能优化机会。