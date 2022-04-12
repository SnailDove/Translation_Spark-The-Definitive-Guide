---
title: 翻译 Chapter18_Monitoring-and-Debugging
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

# Chapter 18 Monitoring and Debugging 监控与调试

This chapter covers the key details you need to monitor and debug your Spark Applications. To do this, we will walk through the Spark UI with an example query designed to help you understand how to trace your own jobs through the execution life cycle. The example we’ll look at will also help you understand how to debug your jobs and where errors are likely to occur.

本章介绍了监视和调试Spark应用程序所需的关键详细信息。为此，我们将使用一个示例查询来浏览Spark UI，该查询旨在帮助您了解如何在执行生命周期中跟踪自己的作业。我们将看到的示例还将帮助您了解如何调试作业以及可能发生错误的位置。

## <font color="#9a161a">The Monitoring Landscape 监控的宏观图</font>

At some point, you’ll need to monitor your Spark jobs to understand where issues are occuring in them. It’s worth reviewing the different things that we can actually monitor and outlining some of the options for doing so. Let’s review the components we can monitor (see Figure 18-1).

在某些时候，您需要监视您的Spark作业，以了解其中发生问题的位置。值得回顾一下我们可以实际监控的不同内容，并概述了这样做的一些选项。让我们回顾一下我们可以监控的组件（参见图18-1，在下文）。

**Spark Applications and Jobs Spark应用程序和作业**

The first thing you’ll want to begin monitoring when either debugging or just understanding better how your application executes against the cluster is the Spark UI and the Spark logs. These report information about the applications currently running at the level of concepts in Spark, such as RDDs and query plans. We talk in detail about how to use these Spark monitoring tools throughout this chapter.

在调试或只是更好地了解在集群背景下应用程序执行的方式时，您首先想要开始监视的是Spark UI和Spark日志。这些报告有关当前在Spark中概念级别运行的应用程序的信息，例如RDD和查询计划。我们将在本章中详细讨论如何使用这些Spark监视工具。

**JVM**

Spark runs the executors in individual Java Virtual Machines (JVMs). Therefore, the next level of detail would be to monitor the individual virtual machines (VMs) to better understand how your code is running. JVM utilities such as *jstack* for providing stack traces, *jmap* for creating heapdumps, *jstat* for reporting time–series statistics, and *jconsole* for visually exploring various JVM properties are useful for those comfortable with JVM internals. You can also use a tool like *jvisualvm* to help profile Spark jobs. Some of this information is provided in the Spark UI, but for very low-level debugging, the aforementioned  tools can come in handy.

Spark 在各个 Java 虚拟机（JVM）中运行执行程序（executor）。因此，下一级详细信息将是监视单个虚拟机（VM）以更好地了解代码的运行方式。 JVM实用程序（如用于提供堆栈跟踪的 jstack ，用于创建 heapdumps 的 jmap，用于报告时间序列统计信息的 jstat）以及用于可视化地探索各种 JVM 属性的 jconsole， 这些对于那些熟悉 JVM 内部的人来说非常有用。您还可以使用 jvisualvm 之类的工具来帮助分析 Spark 作业。其中一些信息在 Spark UI 中提供，但对于非常低级的调试，上述工具可以派上用场。

**OS/Machine**

The JVMs run on a host operating system (OS) and it’s important to monitor the state of those machines ensure that they are healthy. This includes monitoring things like CPU, network, and I/O. These are often reported in cluster-level monitoring solutions; however, there are more specific tools that you can use, including *dstat*, *iostat*, and *iotop*.

JVM 在主机操作系统（OS）上运行，监控这些机器的状态以确保它们是健康的非常重要。这包括监视 CPU，网络和 I/O 等内容。这些通常在集群级监控解决方案中报告；但是，您可以使用更多特定工具，包括 dstat，iostat 和iotop。

**Cluster**

Naturally, you can monitor the cluster on which your Spark Application(s) will run. This might be a YARN, Mesos, or standalone cluster. Usually it’s important to have some sort of monitoring solution here because, somewhat obviously, if your cluster is not working, you should probably know pretty quickly. Some popular cluster-level monitoring tools include *Ganglia* and *Prometheus*. 

当然，您可以监视运行 Spark 应用程序的集群。这可能是 YARN，Mesos 或独立集群。通常，在这里使用某种监控解决方案很重要，因为很明显，如果您的集群不工作，您应该很快就会知道。一些流行的集群级监控工具包括Ganglia 和 Prometheus。

![1566897517782](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter18/1566897517782.png)

## <font color="#9a161a">What to Monitor 要监控什么</font>
After that brief tour of the monitoring landscape, let’s discuss how we can go about monitoring and debugging our Spark Applications. There are two main things you will want to monitor: the processes running your application (at the level of CPU usage, memory usage, etc.), and the query execution inside it (e.g., jobs and tasks).

在简要介绍了监控环境之后，让我们讨论如何监控和调试我们的 Spark 应用程序。您需要监视两个主要内容：运行应用程序的进程（在 CPU 使用情况，内存使用情况的等级）以及在其中执行的查询（例如，作业和任务）。

### <font color="#000000">Driver and Executor Processes 驱动程序和执行程序进程</font>
When you’re monitoring a Spark application, you’re definitely going to want to keep an eye on the driver. This is where all of the state of your application lives, and you’ll need to be sure it’s running in a stable manner. If you could monitor only one machine or a single JVM, it would definitely be the driver. With that being said, understanding the state of the executors is also extremely important for monitoring individual Spark jobs. To help with this challenge, Spark has a configurable metrics system based on the Dropwizard Metrics Library. The metrics system is configured via a configuration file that Spark expects to be present at `$SPARK_HOME/conf/metrics.properties`. A custom file location can be specified by changing the `spark.metrics.conf` configuration property. These metrics can be output to a variety of different sinks, including cluster monitoring solutions like *Ganglia*.

当您监控Spark应用程序时，您肯定会想要关注驱动程序（driver）。这是您的应用程序的所有状态所在的位置，您需要确保它以稳定的方式运行。如果您只能监控一台机器或一台 JVM，那肯定是驱动程序（driver）。话虽如此，了解执行程序（executor）的状态对于监视各个 Spark 作业也非常重要。为了应对这一挑战，Spark 拥有一个基于 Dropwizard Metrics 库的可配置衡量系统。衡量系统通过 Spark 预期出现在 `$SPARK_HOME/conf/metrics.properties` 中的配置文件进行配置。可以通过更改 `spark.metrics.conf` 配置属性来指定自定义文件位置。这些指标可以输出到各种不同的接收器，包括像 Ganglia 这样的集群监控解决方案。

### <font color="#000000">Queries, Jobs, Stages, and Tasks 查询，作业，阶段和任务</font>

Although the driver and executor processes are important to monitor, sometimes you need to debug what’s going on at the level of a specific query. Spark provides the ability to dive into queries, jobs, stages, and tasks. (We learned about these in Chapter 15.) This information allows you to know exactly what’s running on the cluster at a given time. When looking for performance tuning or debugging, this is where you are most likely to start. Now that we know what we want to monitor, let’s look at the two most common ways of doing so: the Spark logs and the Spark UI. 

虽然驱动程序（driver）和执行程序（executor）进程对于监视很重要，但有时您需要调试特定查询级别的进程。 Spark 提供了深入查询，工作，阶段和任务的能力。 （我们在第15章中了解了这些内容。）此信息可让您准确了解在给定时间情况下集群上正在运行的内容。在寻找性能调优或调试时，这是您最有可能开始的地方。现在我们知道了我们想要监控的内容，让我们看看这两种最常见的方式：Spark 日志和 Spark UI 。

## <font color="#9a161a">Spark Logs Spark日志</font>

One of the most detailed ways to monitor Spark is through its log files. Naturally, strange events in Spark’s logs, or in the logging that you added to your Spark Application, can help you take note of exactly where jobs are failing or what is causing that failure. If you use the application template provided with the book, the logging framework we set up in the template will allow your application logs to show up along Spark’s own logs, making them very easy to correlate. One challenge, however, is that Python won’t be able to integrate directly with Spark’s Java-based logging library. Using Python’s logging module or even simple print statements will still print the results to standard error, however, and make them easy to find.

监视 Spark 的最详细方法之一是通过其日志文件。当然，Spark 的日志中或您添加到 Spark 应用程序的日志记录中的奇怪事件可以帮助您准确记录作业失败的原因或导致失败的原因。如果您使用本书提供的应用程序模板，我们在模板中设置的日志记录框架将允许您的应用程序日志显示在 Spark 自己的日志中，使它们非常容易关联。然而，一个挑战是 Python 无法直接与 Spark 的基于 Java 的日志库集成。但是，使用 Python 的日志记录模块甚至简单的打印语句仍然会将结果打印到标准错误，并使它们易于查找。

To change Spark’s log level, simply run the following command : 

要更改 Spark 的日志级别，只需运行以下命令 : 

    spark.sparkContext.setLogLevel("INFO")

This will allow you to read the logs, and if you use our application template, you can log your own relevant information along with these logs, allowing you to inspect both your own application and Spark. The logs themselves will be printed to standard error when running a local mode application, or saved to files by your cluster manager when running Spark on a cluster. Refer to each cluster manager’s documentation about how to find them—typically, they are available through the cluster manager’s web UI.

这将允许您阅读日志，如果您使用我们的应用程序模板，您可以记录您自己的相关信息以及这些日志，允许您检查自己的应用程序和 Spark。运行本地模式应用程序时，日志本身将打印为标准错误，或者在集群上运行 Spark 时由集群管理器保存到文件。请参阅每个集群管理器的文档，了解如何查找它们——通常，它们可通过集群管理器的Web UI 获得。

You won’t always find the answer you need simply by searching logs, but it can help you pinpoint the given problem that you’re encountering and possibly add new log statements in your application to better understand it. It’s also convenient to collect logs over time in order to reference them in the future. For instance, if your application crashes, you’ll want to debug why, without access to the now crashed application. You may also want to ship logs off the machine they were written on to hold onto them if a machine crashes or gets shut down (e.g., if running in the cloud).

您不会总是通过搜索日志找到所需的答案，但它可以帮助您查明您遇到的给定问题，并可能在您的应用程序中添加新的日志语句以更好地理解它。随着时间的推移收集日志以便将来引用它们也很方便。例如，如果您的应用程序崩溃，您将需要调试原因，而无需访问现在崩溃的应用程序。如果计算机崩溃或关闭（例如，如果在云中运行），您可能还希望将日志从他们写入的计算机上发送到其上。

## <font color="#9a161a">The Spark UI</font>

The Spark UI provides a visual way to monitor applications while they are running as well as metrics about your Spark workload, at the Spark and JVM level. Every SparkContext running launches a web UI, by default on port 4040, that displays useful information about the application. When you run Spark in local mode, for example, just navigate to http://localhost:4040 to see the UI when running a Spark Application on your local machine. If you’re running multiple applications, they will launch web UIs on increasing port numbers (4041, 4042, …). Cluster managers will also link to each application’s web UI from their own UI.

Spark UI 提供了一种可视化方式，用于在运行时监视应用程序，以及 Spark 和 JVM 级别的 Spark 工作负载指标。每个运行的 SparkContext 都会在端口 4040 上默认启动 Web UI，该UI显示有关应用程序的有用信息。例如，在本地模式下运行 Spark 时，只需导航到 http://localhost:4040，即可在本地计算机上运行 Spark 应用程序时查看UI 。如果您正在运行多个应用程序，他们将在增加端口号（4041,4042，...）时启动Web UI。集群管理器还将从其自己的 UI 链接到每个应用程序的 Web UI。

Figure 18-2 shows all of the tabs available in the Spark UI. 

图18-2 显示了 Spark UI 中可用的所有选项卡。

![1566904042989](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter18/1566904042989.png)

These tabs are accessible for each of the things that we’d like to monitor. For the most part, each of these should be self-explanatory : 

这些选项卡可供我们要监控的每个事项访问。在大多数情况下，每一个都应该是不言自明的：

- The Jobs tab refers to Spark jobs.

  “作业”选项卡指的是Spark作业。

- The Stages tab pertains to individual stages (and their relevant tasks).

  阶段选项卡适用于各个阶段（及其相关任务）。

- The Storage tab includes information and the data that is currently cached in our Spark Application.

  “存储”选项卡包含当前在我们的 Spark 应用程序中缓存的信息和数据。

- The Environment tab contains relevant information about the configurations and current settings of the Spark application.

  “环境”选项卡包含有关 Spark 应用程序的配置和当前设置的相关信息。

- The SQL tab refers to our Structured API queries (including SQL and DataFrames).

  SQL选项卡引用我们的结构化API查询（包括 SQL 和 DataFrames）。

- The Executors tab provides detailed information about each executor running our application.

  Executors选项卡提供有关运行我们的应用程序的每个执行程序（executor）的详细信息。

Let’s walk through an example of how you can drill down into a given query. Open a new Spark shell, run the following code, and we will trace its execution through the Spark UI: 

让我们来看一个如何深入查看给定查询的示例。打开一个新的 Spark shell，运行以下代码，我们将通过 Spark UI 跟踪它的执行：

```scala
# in Python
spark.read\
.option("header", "true")\
.csv("/data/retail-data/all/online-retail-dataset.csv")\
.repartition(2)\
.selectExpr("instr(Description, 'GLASS') >= 1 as is_glass")\
.groupBy("is_glass")\
.count()\
.collect()
```

This results in three rows of various values. The code kicks off a SQL query, so let’s navigate to the SQL tab, where you should see something similar to Figure 18-3.

这导致不同值的三行。 代码开始启动 SQL 查询，所以让我们导航到 SQL 选项卡，在那里你应该看到类似于图 18-3 的内容。

![1566904811641](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter18/1566904811641.png)

The first thing you see is aggregate statistics about this query:

您看到的第一件事是关于此查询的汇总统计信息：

<p style="font-family:Consolas;color:#000000;">
    Submitted Time: 2017/04/08 16:24:41</br>
Duration: 2 s</br>
Succeeded Jobs: 2
</p>

These will become important in a minute, but first let’s take a look at the Directed Acyclic Graph (DAG) of Spark stages. Each blue box in these tabs represent a stage of Spark tasks. The entire group of these stages represent our Spark job. Let’s take a look at each stage in detail so that we can better understand what is going on at each level, starting with Figure 18-4. 

这些将马上变得重要，但首先让我们来看看 Spark 阶段的有向无环图（DAG）。 这些选项卡中的每个蓝色框表示 Spark 任务的一个阶段。 这些阶段的整个组代表我们的 Spark 工作。 让我们详细了解每个阶段，以便我们可以更好地了解每个级别的情况，从图 18-4 开始。

![1566907520858](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter18/1566907520858.png)

The box on top, labeled WholeStateCodegen, represents a full scan of the CSV file. The box below that represents a shuffle that we forced when we called repartition. This turned our original dataset (of a yet to be specified number of partitions) into two partitions.

标记为 WholeStateCodegen 的顶部框表示 CSV 文件的完整扫描。 下面的框表示我们在调用重新分区时强制进行的数据再分配(shuffle)。 这将我们的原始数据集（尚未指定的分区数）转换为两个分区。

The next step is our projection (selecting/adding/filtering columns) and the aggregation. Notice that in Figure 18-5 the number of output rows is six. This conveniently lines up with the number of output rows multiplied by the number of partitions at aggregation time. This is because Spark performs an aggregation for each partition (in this case a hash-based aggregation) before shuffling the data around  in preparation for the final stage.

下一步是我们的投影（选择/添加/过滤列）和聚合。 请注意，在图18-5中，输出行数为6（final output * 2 = 6）。 这方便与输出行的数量乘以聚合时的分区数对齐。  这是因为 Spark 在为最终阶段做准备之前对数据进行再分配(shuffle)之前，为每个分区执行聚合（在这种情况下是基于散列（hash-based） 的聚合）。

![1566907630723](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter18/1566907630723.png)

The last stage is the aggregation of the subaggregations that we saw happen on a per-partition basis in the previous stage. We combine those two partitions in the final three rows that are the output of our total query (Figure 18-6)。

最后一个阶段是我们在前一阶段基于每个分区发生的子聚合的聚合。 我们将这两个分区组合在最后三行中，这三行是我们总查询的输出（图18-6）。

![1566907691417](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter18/1566907691417.png)

Let’s look further into the job’s execution. On the Jobs tab, next to Succeeded Jobs, click 2. As Figure 18-7 demonstrates, our job breaks down into three stages (which corresponds to what we saw on the SQL tab). 

让我们进一步了解作业的执行情况。 在 Jobs 选项卡上，单击 Succeeded Jobs，单击2. 如图18-7所示，我们的工作分为三个阶段（与我们在SQL选项卡上看到的相对应）。

![1566907752419](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter18/1566907752419.png)

These stages have more or less the same information as what’s shown in Figure 18-6, but clicking the label for one of them will show the details for a given stage. In this example, three stages ran, with eight, two, and then two hundred tasks each. Before diving into the stage detail, let’s review why this is the case.

这些阶段或多或少与图18-6中显示的信息相同，但单击其中一个阶段的标签将显示给定阶段的详细信息。 在这个例子中，运行了三个阶段，每个阶段分别有八个，两个，然后是两百个任务。 在深入了解阶段细节之前，让我们回顾一下为什么会这样。

The first stage has eight tasks. CSV files are splittable, and Spark broke up the work to be distributed relatively evenly between the different cores on the machine. This happens at the cluster level and points to an important optimization: how you store your files. The following stage has two tasks because we explicitly called a repartition to move the data into two partitions. The last stage has 200 tasks because the default shuffle partitions value is 200.

第一阶段有八个任务。 CSV 文件是可拆分的，Spark 分解了工作使其相对均匀分布在机器上不同核心之间。 这发生在集群级别，并指向一个重要的优化：如何存储文件。 下一个阶段有两个任务，因为我们显式调用了重新分区以将数据移动到两个分区中。 最后一个阶段有200个任务，因为默认的 shuffle 分区值为 200。

Now that we reviewed how we got here, click the stage with eight tasks to see the next level of detail, as shown in Figure 18-8.

现在我们回顾了我们如何到达这里，单击具有八个任务的阶段以查看下一个详细级别，如图18-8所示。

![1566908039098](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter18/1566908039098.png)

Spark provides a lot of detail about what this job did when it ran. Toward the top, notice the Summary Metrics section. This provides a synopsis of statistics regarding various metrics. What you want to be on the lookout for is uneven distributions of the values (we touch on this in Chapter 19). In this case, everything looks very consistent; there are no wide swings in the distribution of values. In the table at the bottom, we can also examine on a per-executor basis (one for every core on this particular machine, in this case). This can help identify whether a particular executor is struggling with its workload.

Spark 提供了很多关于这项工作在运行时所做些什么的细节。在顶部，请注意“摘要衡量标准（Summary Metrics）”部分。这提供了有关各种指标的统计数据的概要。你想要注意的是值的不均匀分布（我们在第19章中讨论）。在这种情况下，一切看起来都非常一致; 值的分布没有大幅波动。在底部的表中，我们还可以基于每个执行程序（executor）进行检查（在这种情况下，该特定计算机上的每个核心都有一个）。这有助于确定特定执行程序（executor）是否在努力应对其工作量。

Spark also makes available a set of more detailed metrics, as shown in Figure 18-8, which are probably not relevant to the large majority of users. To view those, click Show Additional Metrics, and then either choose (De)select All or select individual metrics, depending on what you want to see.

Spark还提供了一组更详细的指标，如图18-8所示，这些指标可能与绝大多数用户无关。要查看这些，请单击“显示其他衡量标准”，然后选择（取消）选择“全部”或选择单个衡量标准，具体取决于您要查看的内容。

You can repeat this basic analysis for each stage that you want to analyze. We leave that as an exercise for the reader.

您可以为要分析的每个阶段重复此基本分析。我们把它作为读者的练习。

### <font color="#000000">Other Spark UI tabs 其他Spark UI选项卡</font>

The remaining Spark tabs, Storage, Environment, and Executors, are fairly self-explanatory. The Storage tab shows information about the cached RDDs/DataFrames on the cluster. This can help you see if certain data has been evicted from the cache over time. The Environment tab shows you information about the Runtime Environment, including information about Scala and Java as well as the various Spark Properties that you configured on your cluster.

其余的 Spark 选项卡，存储，环境和执行程序（executor），都是不言自明的。 “存储”选项卡显示有关集群上缓存的 RDD / DataFrame 的信息。这可以帮助您查看某些数据是否随着时间的推移从缓存中逐出。 “环境”选项卡显示有关运行时环境的信息，包括有关 Scala 和 Java 的信息以及您在集群上配置的各种Spark属性。

### <font color="#000000">Configuring the Spark user interface 配置Spark用户界面</font>

There are a number of configurations that you can set regarding the Spark UI. Many of them are networking configurations such as enabling access control. Others let you configure how the Spark UI will behave (e.g., how many jobs, stages, and tasks are stored). Due to space limitations, we cannot include the entire configuration set here. Consult the relevant table on [Spark UI Configurations](http://spark.apache.org/docs/latest/monitoring.html#spark-configuration-options) in the Spark documentation. 

您可以设置有关 Spark UI 的许多配置。其中许多是网络配置，例如启用访问控制。其他允许您配置 Spark UI 的行为方式（例如，存储了多少个作业，阶段和任务）。由于篇幅限制，我们无法在此处包含整个配置集。请参阅Spark文档中 [Spark UI配置](http://spark.apache.org/docs/latest/monitoring.html#spark-configuration-options) 的相关表。

### <font color="#000000">Spark REST API</font>

In addition to the Spark UI, you can also access Spark’s status and metrics via a REST API. This is is available at http://localhost:4040/api/v1 and is a way of building visualizations and monitoring tools on top of Spark itself. For the most part this API exposes the same information presented in the web UI, except that it doesn’t include any of the SQL-related information. This can be a useful tool if you would like to build your own reporting solution based on the information available in the Spark UI. Due to space limitations, we cannot include the list of API endpoints here. Consult the relevant table on [REST API Endpoints](http://spark.apache.org/docs/latest/monitoring.html#rest-api) in the Spark documentation. 

除了 Spark UI，您还可以通过 REST API 访问Spark的状态和指标。这可以在 http://localhost:4040/api/v1 上获得，它是一种在Spark本身之上构建可视化和监视工具的方法。在大多数情况下，此API公开Web UI中显示的相同信息，但它不包含任何与SQL相关的信息。如果您希望根据Spark UI中提供的信息构建自己的报告解决方案，这可能是一个有用的工具。由于篇幅限制，我们无法在此处包含API端点列表。请参阅Spark文档中有关[REST API端点](http://spark.apache.org/docs/latest/monitoring.html#rest-api)的相关表。

### <font color="#000000">Spark UI History Server SparkUI历史记录服务器</font>

Normally, the Spark UI is only available while a SparkContext is running, so how can you get to it after your application crashes or ends? To do this, Spark includes a tool called the Spark History Server that allows you to reconstruct the Spark UI and REST API, provided that the application was configured to save an event log. You can find up-to-date information about how to use this tool in [the Spark documentation](https://spark.apache.org/docs/latest/monitoring.html).

通常，Spark UI 仅在 SparkContext 运行时可用，因此在应用程序崩溃或结束后如何才能访问它？为此，Spark包含一个名为 Spark History Server 的工具，允许您重建 Spark UI 和 REST API，前提是应用程序已配置为保存事件日志。您可以在[Spark文档](https://spark.apache.org/docs/latest/monitoring.html)中找到有关如何使用此工具的最新信息。

To use the history server, you first need to configure your application to store event logs to a certain location. You can do this by by enabling and the event log location with the configuration `spark.eventLog.dir`. Then, once you have stored the events, you can run the history server as a standalone application, and it will automatically reconstruct the web UI based on these logs. Some cluster managers and cloud services also configure logging automatically and run a history server by default.

要使用历史记录服务器，首先需要配置应用程序以将事件日志存储到特定位置。您可以通过启用spark.eventLog.enabled 和配置 spark.eventLog.dir 的事件日志位置来完成此操作。然后，一旦存储了事件，就可以将历史服务器作为独立应用程序运行，它将根据这些日志自动重建Web UI。某些集群管理器和云服务还会自动配置日志记录并默认运行历史记录服务器。

There are a number of other configurations for the history server. Due to space limitations, we cannot include the entire configuration set here. Refer to the relevant table on [Spark History Server Configurations](https://spark.apache.org/docs/latest/monitoring.html#spark-history-server-configuration-options) in the Spark documentation. 

历史服务器还有许多其他配置。由于篇幅限制，我们无法在此处包含整个配置集。请参阅Spark文档中有关[Spark History Server配置](https://spark.apache.org/docs/latest/monitoring.html#spark-history-server-configuration-options)的相关表。

## <font color="#9a161a">Debugging and Spark First Aid 调试和Spark急救</font>

The previous sections defined some core “vital signs”—that is, things that we can monitor to check the health of a Spark Application. For the remainder of the chapter we’re going to take a “first aid” approach to Spark debugging: We’ll review some signs and symptoms of problems in your Spark jobs, including signs that you might observe (e.g., slow tasks) as well as symptoms from Spark itself (e.g., OutOfMemoryError). There are many issues that may affect Spark jobs, so it’s impossible to cover everything. But we will discuss some of the more common Spark issues you may encounter. In addition to the signs and symptoms, we’ll also look at some potential treatments for these issues. 

前面的部分定义了一些核心的“生命体征” ——也就是说，我们可以监视以检查Spark应用程序运行状况的事情。对于本章的其余部分，我们将对Spark调试采取“急救”方法： 我们将审查Spark作业中的一些问题迹象和症状，包括您可能观察到的迹象（例如，缓慢的任务）以及来自Spark本身的症状（例如，OutOfMemoryError）。有许多问题可能会影响Spark作业，因此无法涵盖所有内容。但我们将讨论您可能遇到的一些更常见的Spark问题。除了症状和体征外，我们还将研究这些问题的一些潜在治疗方法。

Most of the recommendations about fixing issues refer to the configuration tools discussed in Chapter 16.

有关修复问题的大多数建议都参考了第16章中讨论的配置工具。

### <font color="#000000">Spark Jobs Not Starting Spark工作没有开始</font>

This issue can arise frequently, especially when you’re just getting started with a fresh deployment or environment.

这个问题可能经常出现，特别是当您刚刚开始全新部署或环境。

#### <font color="#3399cc">Signs and symptoms 迹象和症状</font>

- Spark jobs don’t start.

  Spark工作无法启动。

- The Spark UI doesn’t show any nodes on the cluster except the driver.

  除驱动程序（driver）外，Spark UI不显示集群上的任何节点。

- The Spark UI seems to be reporting incorrect information.

  Spark UI 似乎报告了错误的信息。

#### <font color="#3399cc">Potential treatments 可能的解决方法</font>

This mostly occurs when your cluster or your application’s resource demands are not configured properly. Spark, in a distributed setting, does make some assumptions about networks, file systems, and other resources. During the process of setting up the cluster, you likely configured something incorrectly, and now the node that runs the driver cannot talk to the executors. This might be because you didn’t specify what IP and port is open or didn’t open the correct one. This is most likely a cluster level, machine, or configuration issue. Another option is that your application requested more resources per executor than your cluster manager currently has free, in which case the driver will be waiting forever for executors to be launched.

当您的集群或应用程序的资源需求未正确配置时，通常会发生这种情况。 Spark 在分布式设置中确实对网络，文件系统和其他资源做出了一些假设。在设置集群的过程中，您可能错误地配置了某些内容，现在运行驱动程序（driver）的节点无法与执行程序（executor）通信。这可能是因为您未指定打开的IP和端口或未打开正确的IP和端口。这很可能是集群级别，计算机或配置问题。另一个选择是，您的应用程序为每个执行程序（executor）请求的资源比您的集群管理器当前有空的资源要多，在这种情况下，驱动程序（driver）将永远等待执行程序（executor）启动。

- Ensure that machines can communicate with one another on the ports that you expect. Ideally, you should open up all ports between the worker nodes unless you have more stringent security constraints.

    确保计算机可以在您期望的端口上相互通信。理想情况下，您应该打开工作节点之间的所有端口，除非您有更严格的安全约束。

- Ensure that your Spark resource configurations are correct and that your cluster manager is properly set up for Spark. Try running a simple application first to see if that works. One common issue may be that you requested more memory per executor than the cluster manager has free to allocate, so check how much it is reporting free (in its UI) and your sparksubmit memory configuration.

  确保Spark资源配置正确并且已正确设置集群管理器以用于 Spark。尝试先运行一个简单的应用程序，看看是否有效。一个常见问题可能是您为每个执行程序（executor）请求的内存多于集群管理器可以自由分配的内存，因此请检查它是报告空闲的多少（在其 UI 中）和 sparksubmit 内存配置。

### <font color="#000000">Errors Before Execution 执行前的错误</font>

This can happen when you’re developing a new application and have previously run code on this cluster, but now some new code won’t work.

当您开发新应用程序并且之前在此集群上运行代码时，可能会发生这种情况，但现在某些新代码将无法运行。

Signs and symptoms 迹象和症状

- Commands don’t run at all and output large error messages. 

  命令根本不运行并输出大的错误消息。

- You check the Spark UI and no jobs, stages, or tasks seem to run.

  您检查Spark UI并且似乎没有任何作业，阶段或任务运行。

#### <font color="#3399cc">Potential treatments 潜在的治疗方法</font>

After checking and confirming that the Spark UI environment tab shows the correct information for your application, it’s worth double-checking your code. Many times, there might be a simple typo or incorrect column name that is preventing the Spark job from compiling into its underlying Spark plan (when using the DataFrame API).

在检查并确认 Spark UI 环境选项卡显示应用程序的正确信息后，值得仔细检查您的代码。很多时候，可能会出现一个简单的拼写错误或不正确的列名，导致Spark作业无法编译到其基础 Spark 计划中（使用 DataFrame API 时）。

You should take a look at the error returned by Spark to confirm that there isn’t an issue in your code, such as providing the wrong input file path or field name. Double-check to verify that the cluster has the network connectivity that you expect between your driver, your workers, and the storage system you are using.

您应该查看Spark返回的错误，以确认代码中没有问题，例如提供错误的输入文件路径或字段名称。仔细检查以验证集群是否具有您期望的驱动程序（driver），工作人员和正在使用的存储系统之间的网络连接。

There might be issues with libraries or classpaths that are causing the wrong version of a library to be loaded for accessing storage. Try simplifying your application until you get a smaller version that reproduces the issue (e.g., just reading one dataset).

库或类路径可能存在导致加载库的错误版本以访问存储的问题。尝试简化您的应用程序，直到您获得重现问题的较小版本（例如，只读取一个数据集）。

### <font color="#000000">Errors During Execution 执行期间的错误</font>

- This kind of issue occurs when you already are working on a cluster or parts of your Spark 

  当您已经在集群或部分Spark上工作时，会出现此类问题。

- Application run before you encounter an error. This can be a part of a scheduled job that runs at some interval or a part of some interactive exploration that seems to fail after some time.

    在遇到错误之前运行应用程序。这可以是某个时间间隔运行的已经安排作业的一部分，也可能是某些时间后似乎失败的某些交互式探索的一部分。

#### <font color="#3399cc">Signs and symptoms 迹象和症状</font>

One Spark job runs successfully on the entire cluster but the next one fails.

一个 Spark 作业在整个集群上成功运行，但下一个失败。

- A step in a multistep query fails.

  多步查询中的步骤失败。

- A scheduled job that ran yesterday is failing today.

  昨天运行的预定工作今天失败了。

- Difficult to parse error message.

  难以解析错误消息。

#### <font color="#3399cc">Potential treatments 可能的疗法</font>

- Check to see if your data exists or is in the format that you expect. This can change over time or some upstream change may have had unintended consequences on your application. If an error quickly pops up when you run a query (i.e., before tasks are launched), it is most likely an analysis error while planning the query. This means that you likely misspelled a column name referenced in the query or that a column, view, or table you referenced does not exist.

    检查您的数据是否存在或是否符合您的预期格式。这可能会随着时间的推移而改变，或者某些上游更改可能会对您的应用程序产生意外后果。如果在运行查询时（即，在启动任务之前）快速弹出错误，则在计划查询时很可能是分析错误。这意味着您可能拼错了查询中引用的列名称，或者您引用的列，视图或表不存在>。

- Read through the stack trace to try to find clues about what components are involved (e.g., what operator and stage it was running in). Try to isolate the issue by progressively double-checking input data and ensuring the data conforms to your expectations. Also try removing logic until you can isolate the problem in a smaller version of your application.

    读取堆栈跟踪以尝试查找涉及哪些组件的线索（例如，运行的算子和阶段）。尝试通过逐步检查输入数据并确保数据符合您的期望来隔离问题。还可以尝试删除逻辑，直到您可以在较小版本的应用程序中隔离问题。

- If a job runs tasks for some time and then fails, it could be due to a problem with the input data itself, wherein the schema might be specified incorrectly or a particular row does not conform to the expected schema. For instance, sometimes your schema might specify that the data contains no nulls but your data does actually contain nulls, which can cause certain transformations to fail.

    如果作业运行任务一段时间然后失败，则可能是由于输入数据本身存在问题，其中可能未正确指定模式或特定行不符合预期模式。例如，有时您的模式可能指定数据不包含空值，但您的数据确实包含空值，这可能导致某些转换失败。

- It’s also possible that your own code for processing the data is crashing, in which case Spark will show you the exception thrown by your code. In this case, you will see a task marked as “failed” on the Spark UI, and you can also view the logs on that machine to understand what it was doing when it failed. Try adding more logs inside your code to figure out which data record was being processed.

    您自己的处理数据的代码也可能崩溃，在这种情况下，Spark会向您显示代码抛出的异常。在这种情况下，您将在Spark UI上看到标记为“失败”的任务，您还可以查看该计算机上的日志以了解失败时正在执行的操作。尝试在代码中添加更多日志，以确定正在处理哪些数据记录。

### <font color="#000000">Slow Tasks or Stragglers 缓慢的任务或Stragglers</font>

This issue is quite common when optimizing applications, and can occur either due to work not being evenly distributed across your machines (“skew”), or due to one of your machines being slower than the others (e.g., due to a hardware problem).

在优化应用程序时，此问题非常常见，并且可能由于工作不均匀分布在您的计算机上（“倾斜”），或者由于您的某台计算机比其他计算机慢（例如，由于硬件问题）而发生。

#### <font color="#3399cc">Signs and symptoms 迹象和症状</font>

Any of the following are appropriate symptoms of the issue : 

以下任何一种都是该问题的适当症状：

- Spark stages seem to execute until there are only a handful of tasks left. Those tasks then take a long time.

  Spark阶段似乎执行，直到只剩下少数任务。那些任务需要很长时间。

- These slow tasks show up in the Spark UI and occur consistently on the same dataset(s).

  这些缓慢的任务显示在 Spark UI 中，并在相同的数据集上一致地发生。

- These occur in stages, one after the other.

  这些是分阶段发生的，一个接一个。

- Scaling up the number of machines given to the Spark Application doesn’t really help—some tasks still take much longer than others.

  扩大提供给 Spark 应用程序的机器数量并没有多大帮助——某些任务仍然需要比其他任务更长的时间。

- In the Spark metrics, certain executors are reading and writing much more data than others.

  在Spark指标中，某些执行程序（executor）正在读取和写入比其他数据更多的数据。

#### <font color="#3399cc">Potential treatments 可能的治疗方法</font>
- Slow tasks are often called “stragglers.” There are many reasons they may occur, but most often the source of this issue is that your data is partitioned unevenly into DataFrame or RDD partitions. When this happens, some executors might need to work on much larger amounts of work than others. One particularly common case is that you use a group-by-key operation and one of the keys just has more data than others. In this case, when you look at the Spark UI, you might see that the shuffle data for some nodes is much larger than for others. 

    缓慢的任务通常被称为“落后者”。它们可能出现的原因很多，但大多数情况下，这个问题的根源是您的数据被不均匀地划分为 DataFrame 或 RDD 分区。当发生这种情况时，一些执行程序（executor）可能需要处理比其他工作量大得多的工作。一个特别常见的情况是您使用逐个键操作，其中一个键只是比其他键更多的数据。在这种情况下，当您查看 Spark UI 时，您可能会看到某些节点的 shuffle 数据比其他节点大得多。

- Try increasing the number of partitions to have less data per partition.

    尝试增加分区数，以使每个分区的数据更少。

- Try repartitioning by another combination of columns. For example, stragglers can come up when you partition by a skewed ID column, or a column where many values are null. In the latter case, it might make sense to first filter out the null values. Try increasing the memory allocated to your executors if possible.

    尝试通过另一个列组合重新分区。例如，当您通过倾斜的 ID 列或许多值为 null 的列进行分区时，straggler 可能会出现。在后一种情况下，首先过滤掉空值可能是有意义的。如果可能，尝试增加分配给执行程序（executor）的内存。

- Monitor the executor that is having trouble and see if it is the same machine across jobs; you might also have an unhealthy executor or machine in your cluster—for example, one whose disk is nearly full. If this issue is associated with a join or an aggregation, see “Slow Joins” or “Slow Aggregations”.

    监视有问题的执行程序（executor），看看它是否是跨不同作业的同一台机器；您的集群中可能还有一个不健康的执行程序（executor）或计算机——例如，磁盘几乎已满的计算机。如果此问题与连接或聚合相关联，请参阅“慢速连接”或“慢速聚合”。

- Check whether your user-defined functions (UDFs) are wasteful in their object allocation or business logic. Try to convert them to DataFrame code if possible. Ensure that your UDFs or User-Defined Aggregate Functions (UDAFs) are running on a small enough batch of data. Oftentimes an aggregation can pull a lot of data into memory for a common key, leading to that executor having to do a lot more work than others.

    检查用户定义的函数（UDF）在对象分配或业务逻辑中是否浪费。如果可能，尝试将它们转换为 DataFrame 代码。确保您的 UDF 或用户定义的聚合函数（UDAF）在足够小的数据批量上运行。通常，聚合可以将大量数据拉入内存以用于普通的键，从而导致执行程序（executor）必须比其他人执行更多的工作。

- Turning on speculation, which we discuss in “Slow Reads and Writes”, will have Spark run a second copy of tasks that are extremely slow. This can be helpful if the issue is due to a faulty node because the task will get to run on a faster one. Speculation does come at a cost, however, because it consumes additional resources. In addition, for some storage systems that use eventual consistency, you could end up with duplicate output data if your writes are not idempotent . (We discussed speculation configurations in Chapter 17.)

    打开我们在“慢速读取和写入”中讨论的推测（speculation），将使 Spark 运行极其缓慢的第二个任务副本。如果问题是由于故障节点引起的，这可能会有所帮助，因为任务将以更快的速度运行。然而，推测（speculation）确实需要付出代价，因为它消耗了额外的资源。此外，对于某些使用最终一致性的存储系统，如果写入不是幂等的，则最终可能会出现重复的输出数据。 （我们在第17章讨论了推测(speculation)配置）

- Another common issue can arise when you’re working with Datasets. Because Datasets perform a lot of object instantiation to convert records to Java objects for UDFs, they can cause a lot of garbage collection. If you’re using Datasets, look at the garbage collection metrics in the Spark UI to see if they’re consistent with the slow tasks. 

    当您使用 Datasets 时，可能会出现另一个常见问题。由于 Datasets 执行大量对象实例化以将记录转换为UDF 的 Java 对象，因此它们可能导致大量垃圾回收。如果您正在使用 Datasets，请查看Spark UI中的垃圾收集指标，以查看它们是否与缓慢的任务一致。

- Stragglers can be one of the most difficult issues to debug, simply because there are so many possible causes. However, in all likelihood, the cause will be some kind of data skew, so definitely begin by checking the Spark UI for imbalanced amountsimbalanced amounts of data across tasks.

    Stragglers可能是最难调试的问题之一，因为有很多可能的原因。但是，很有可能，原因将是某种数据偏差，因此必须首先检查Spark UI以查找跨任务的不平衡数据量。

### <font color="#000000">Slow Aggregations 慢的聚合</font>
If you have a slow aggregation, start by reviewing the issues in the “Slow Tasks” section before proceeding. Having tried those, you might continue to see the same problem.

如果您的聚合速度较慢，请先继续查看“慢速任务”部分中的问题，然后再继续。尝试过这些后，您可能会继续看到同样的问题。

#### <font color="#3399cc">Signs and symptoms 迹象和症状</font>
- Slow tasks during a `groupBy` call.

  在 groupBy 调用期间缓慢执行任务。

- Jobs after the aggregation are slow, as well.

  聚合后的工作也很慢。
#### <font color="#3399cc">Potential treatments 可能的疗法</font>

Unfortunately, this issue can’t always be solved. Sometimes, the data in your job just has some skewed keys, and the operation you want to run on them needs to be slow. Increasing the number of partitions, prior to an aggregation, might help by reducing the number of different keys processed in each task.

不幸的是，这个问题并不总能解决。有时，作业中的数据只有一些倾斜的键，您想要在它们上运行的操作需要很慢。在聚合之前增加分区数可能有助于减少每个任务中处理的不同键的数量。

- Increasing executor memory can help alleviate this issue, as well. If a single key has lots of data, this will allow its executor to spill to disk less often and finish faster, although it may still be much slower than executors processing other keys.

  增加执行程序（executor）内存也有助于缓解此问题。如果单个键有大量数据，这将允许其执行程序（executor）更少地溢出到磁盘并更快地完成，尽管它可能仍然比处理其他键的执行程序（executor）慢得多。

- If you find that tasks after the aggregation are also slow, this means that your dataset might have remained unbalanced after the aggregation. Try inserting a repartition call to partition it randomly.

  如果您发现聚合后的任务也很慢，这意味着聚合后您的数据集可能仍然不平衡。尝试插入重新分区调用以随机分区。

- Ensuring that all filters and SELECT statements that can be are above the aggregation can help to ensure that you’re working only on the data that you need to be working on and nothing else. Spark’s query optimizer will automatically do this for the structured APIs.

  确保可以在聚合之上的所有过滤器和 SELECT 语句可以帮助确保您仅处理您需要处理的数据而不是其他任何内容。 Spark的查询优化器将自动为结构化API执行此操作。

- Ensure null values are represented correctly (using Spark’s concept of null) and not as some default value like " " or "EMPTY". Spark often optimizes for skipping nulls early in the job when possible, but it can’t do so for your own placeholder values.

  确保正确表示空值（使用Spark的null概念）而不是像“”或“EMPTY”那样的默认值。 Spark通常会尽可能优化在作业的早期跳过空值，但是对于您自己的占位符值，它不能这样做。

- Some aggregation functions are also just inherently slower than others. For instance, `collect_list` and `collect_set` are very slow aggregation functions because they must return all the matching objects to the driver, and should be avoided in performance-critical code.

  某些聚合函数本身也比其他函数慢。例如，collect_list和collect_set是非常慢的聚合函数，因为它们必须将所有匹配的对象返回给驱动程序（driver），并且应该在性能关键代码中避免使用。

### <font color="#000000">Slow Joins 慢加入</font>

Joins and aggregations are both shuffles, so they share some of the same general symptoms as well as treatments.

连接和聚合都是数据再分配(shuffle)，因此它们共享一些相同的症状和应对方法。

#### <font color="#3399cc">Signs and symptoms 迹象和症状</font>
- A join stage seems to be taking a long time. This can be one task or many tasks.

  l连接阶段似乎需要很长时间。这可以是一个任务或许多任务。

- Stages before and after the join seem to be operating normally.

  连接之前和之后的阶段似乎正常运行。

#### <font color="#3399cc">Potential treatments 可能的疗法</font>
- Many joins can be optimized (manually or automatically) to other types of joins. We covered how to select different join types in Chapter 8.

  许多连接可以优化（手动或自动）到其他类型的连接。我们在第8章介绍了如何选择不同的连接类型。

- Experimenting with different join orderings can really help speed up jobs, especially if some of those joins filter out a large amount of data; do those first.

  尝试不同的连接顺序可以真正帮助加快工作，特别是如果其中一些连接过滤掉大量数据; 先做那些。

- Partitioning a dataset prior to joining can be very helpful for reducing data movement across the cluster, especially if the same dataset will be used in multiple join operations. It’s worth experimenting with different prejoin partitioning. Keep in mind, again, that this isn’t “free” and does come at the cost of a shuffle.

  在连接之前对数据集进行分区对于减少集群中的数据移动非常有用，尤其是在多个连接操作中将使用相同的数据集时。值得尝试不同的预连接（prejoin）分区。请记住，这不是“免费”，而是以数据再分配(shuffle)为代价。

- Slow joins can also be caused by data skew. There’s not always a lot you can do here, but sizing up the Spark application and/or increasing the size of executors can help, as described in earlier sections.

  数据倾斜也可能导致慢连接。你可以在这里做很多事情，但是调整 Spark 应用程序和/或增加执行程序（executor）的数量可以提供帮助，如前面部分所述。

- Ensuring that all filters and select statements that can be are above the join can help to ensure that you’re working only on the data that you need for the join.

  确保可以在连接之上的所有筛选器和选择语句可以帮助确保您仅处理连接所需的数据。

- Ensure that null values are handled correctly (that you’re using null) and not some default value like " " or "EMPTY", as with aggregations.

  确保正确处理空值（您使用的是null），而不是像聚合一样处理某些默认值，如“”或“EMPTY”。

- Sometimes Spark can’t properly plan for a broadcast join if it doesn’t know any statistics about the input DataFrame or table. If you know that one of the tables that you are joining is small, you can try to force a broadcast (as discussed in Chapter 8), or use Spark’s statistics collection commands to let it analyze the table.

  如果 Spark 不知道有关输入DataFrame或表的任何统计信息，Spark有时无法正确规划广播连接（broadcast join）。如果您知道要加入的其中一个表很小，则可以尝试强制广播（如第8章中所述），或使用Spark的统计信息收集命令让它分析表。

### <font color="#000000">Slow Reads and Writes 慢的读和写</font>

Slow I/O can be difficult to diagnose, especially with networked file systems.

慢速 I/O 可能难以诊断，尤其是对于网络文件系统。

#### <font color="#3399cc">Signs and symptoms 迹象和症状</font>
Slow reading of data from a distributed file system or external system. Slow writes from network file systems or Blob storage.

从分布式文件系统或外部系统缓慢读取数据。从网络文件系统或 Blob 存储缓慢写入。

#### <font color="#3399cc">Potential treatments 潜在的治疗</font>
Turning on speculation (set `spark.speculation` to `true`) can help with slow reads and writes. This will launch additional tasks with the same operation in an attempt to see whether it’s just some transient issue in the first task. Speculation is a powerful tool and works well with consistent file systems. However, it can cause duplicate data writes with some eventually consistent cloud services, such as Amazon S3, so check whether it is supported by the storage system connector you are using.

打开推测（将 `spark.speculation` 设置为 `true`）可以帮助减慢读取和写入。这将使用相同的操作启动其他任务，以尝试查看它是否只是第一个任务中的一些短暂问题。推测(speculation)是一种功能强大的工具，适用于一致的文件系统。但是，它可能导致重复数据写入与一些最终一致的云服务（如Amazon S3），因此请检查您使用的存储系统连接器是否支持它。

Ensuring sufficient network connectivity can be important—your Spark cluster may simply not have enough total network bandwidth to get to your storage system.

确保足够的网络连接非常重要——您的 Spark 集群可能根本没有足够的总网络带宽来访问您的存储系统。

For distributed file systems such as HDFS running on the same nodes as Spark, make sure Spark sees the same hostnames for nodes as the file system. This will enable Spark to do locality-aware scheduling, which you will be able to see in the “locality” column in the Spark UI. We’ll talk about locality a bit more in the next chapter.

对于与Spark在相同节点上运行的分布式文件系统（如HDFS），请确保Spark看到与文件系统相同的节点主机名。这将使Spark能够进行关注局部性的调度，您可以在Spark UI的“locality”列中看到该调度。我们将在下一章中讨论一下局部性。

### <font color="#000000">Driver OutOfMemoryError or Driver Unresponsive 驱动程序OutOfMemoryError或驱动程序无响应</font> 
This is usually a pretty serious issue because it will crash your Spark Application. It often happens due to collecting too much data back to the driver, making it run out of memory.

这通常是一个相当严重的问题，因为它会使您的Spark应用程序崩溃。它经常发生，因为收集了太多的数据回到驱动程序，使其耗尽内存。

#### <font color="#3399cc">Signs and symptoms 迹象和症状</font>

- Spark Application is unresponsive or crashed. OutOfMemoryErrors or garbage collection messages in the driver logs. 
    Spark应用程序无响应或崩溃。驱动程序日志中的OutOfMemoryErrors或垃圾回收消息。
- Commands take a very long time to run or don’t run at all. 
命令需要很长时间才能运行或根本不运行。
- Interactivity is very low or non-existent.
交互性很低或根本不存在。
- Memory usage is high for the driver JVM.
驱动程序JVM的内存使用率很高。

#### <font color="#3399cc">Potential treatments 可能的治疗方法</font>

There are a variety of potential reasons for this happening, and diagnosis is not always straightforward.

这种情况有多种可能的原因，诊断并不总是直截了当的。

- Your code might have tried to collect an overly large dataset to the driver node using operations such as collect.

    您的代码可能尝试使用诸如collect之类的操作将过大的数据集收集到驱动程序节点。

- You might be using a broadcast join where the data to be broadcast is too big. Use Spark’s maximum broadcast join configuration to better control the size it will broadcast.

  您可能正在使用广播连接，其中要广播的数据太大。使用Spark的最大广播连接配置可以更好地控制它将广播的大小。

- A long-running application generated a large number of objects on the driver and is unable to release them. Java’s jmap tool can be useful to see what objects are filling most of the memory of your driver JVM by printing a histogram of the heap. However, take note that jmap will pause that JVM while running. Increase the driver’s memory allocation if possible to let it work with more data.

  长时间运行的应用程序在驱动程序上生成了大量对象，无法释放它们。 Java 的 jmap 工具可以通过打印堆的直方图来查看哪些对象填充了驱动程序 JVM 的大部分内存。但请注意，jmap 会在运行时暂停该JVM。如果可能的话，增加驱动程序的内存分配，让它可以处理更多数据。

- Issues with JVMs running out of memory can happen if you are using another language binding, such as Python, due to data conversion between the two requiring too much memory in the JVM. Try to see whether your issue is specific to your chosen language and bring back less data to the driver node, or write it to a file instead of bringing it back as in-memory objects.

    如果您使用其他语言绑定（如Python），JVM内存不足会出现问题，因为两者之间的数据转换需要JVM中的内存过多。尝试查看您的问题是否特定于您选择的语言，并将较少的数据带回驱动程序节点，或将其写入文件而不是将其作为内存中对象重新引入。

- If you are sharing a SparkContext with other users (e.g., through the SQL JDBC server and some notebook environments), ensure that people aren’t trying to do something that might be causing large amounts of memory allocation in the driver (like working overly large arrays in their code or collecting large datasets).

    如果您与其他用户共享 SparkContext（例如，通过SQL JDBC服务器和某些 notebook  环境），请确保人们不会尝试执行可能导致驱动程序中大量内存分配的操作（例如，过度工作代码中的大型数组或收集大型数据集）。

### <font color="#000000">Executor OutOfMemoryError or Executor Unresponsive Executor OutOfMemoryError或Executor无响应</font>

Spark applications can sometimes recover from this automatically, depending on the true underlying
issue.

Spark应用程序有时可以自动从中恢复，具体取决于真正的底层问题。

#### <font color="#3399cc">Signs and symptoms 迹象和症状</font>

- OutOfMemoryErrors or garbage collection messages in the executor logs. You can find these in the Spark UI.

    执行程序(executor)日志中的OutOfMemoryErrors或垃圾回收消息。您可以在Spark UI中找到它们。

- Executors that crash or become unresponsive. 

    崩溃或无响应的执行程序（executor）。

- Slow tasks on certain nodes that never seem to recover.

  某些节点上的缓慢任务似乎永远无法恢复。

#### <font color="#3399cc">Potential treatments 潜在的疗法</font>

- Try increasing the memory available to executors and the number of executors.

    尝试增加执行程序(executor)可用的内存和执行程序(executor)的数量。

- Try increasing PySpark worker size via the relevant Python configurations.

  尝试通过相关的Python配置增加PySpark工作者大小。

- Look for garbage collection error messages in the executor logs. Some of the tasks that are running, especially if you’re using UDFs, can be creating lots of objects that need to be garbage collected. Repartition your data to increase parallelism, reduce the amount of records per task, and ensure that all executors are getting the same amount of work.

  在执行程序(executor)日志中查找垃圾收集错误消息。正在运行的某些任务（尤其是在使用UDF时）可能会创建大量需要进行垃圾回收的对象。重新分区数据以增加并行度，减少每个任务的记录数量，并确保所有执行程序(executor)获得相同的工作量。

- Ensure that null values are handled correctly (that you’re using null) and not some default value like " " or "EMPTY", as we discussed earlier.

  确保正确处理空值（您正在使用null）而不是像我们之前讨论的那样的默认值，如“”或“EMPTY”。

- This is more likely to happen with RDDs or with Datasets because of object instantiations.

  由于对象实例化，这更有可能发生在 RDD 或 Datasets  中。

- Try using fewer UDFs and more of Spark’s structured operations when possible.

  尽可能尝试使用更少的UDF和更多Spark的结构化操作。

- Use Java monitoring tools such as *jmap* to get a histogram of heap memory usage on your executors, and see which classes are taking up the most space.

  使用 jmap 等Java监视工具获取执行程序(executor)堆内存使用情况的直方图，并查看哪些类占用的空间最多。

- If executors are being placed on nodes that also have other workloads running on them, such as a key-value store, try to isolate your Spark jobs from other jobs.

  如果将执行程序(executor)放置在也运行其他工作负载的节点上（例如键值存储），请尝试将Spark作业与其他作业隔离开来。

### <font color="#000000">Unexpected Nulls in Results 结果中出现意外空白</font>

#### <font color="#3399cc">Signs and symptoms 迹象和症状</font>

- Unexpected null values after transformations.

  转换后出现意外的空值。

- Scheduled production jobs that used to work no longer work, or no longer produce the right results.

  过去工作安排的生产作业不再起作用，或者不再产生正确的结果。

#### <font color="#3399cc">Potential treatments 潜在的治疗</font>
- It’s possible that your data format has changed without adjusting your business logic. This means that code that worked before is no longer valid.

  您的数据格式可能已更改，而无需调整业务逻辑。这意味着以前工作的代码不再有效。

- Use an accumulator to try to count records or certain types, as well as parsing or processing errors where you skip a record. This can be helpful because you might think that you’re parsing data of a certain format, but some of the data doesn’t. Most often, users will place the accumulator in a UDF when they are parsing their raw data into a more controlled format and perform the counts there. This allows you to count valid and invalid records and then operate accordingly after the fact.

  使用累加器尝试计算记录或某些类型，以及解析或处理跳过记录的错误。这可能很有用，因为您可能认为您正在解析某种格式的数据，但有些数据却没有。大多数情况下，用户在将原始数据解析为更受控制的格式并在那里执行计数时，会将累加器放在 UDF 中。这允许您计算有效和无效的记录，然后在事后进行相应的操作。

- Ensure that your transformations actually result in valid query plans. Spark SQL sometimes does implicit type coercions that can cause confusing results. For instance, the SQL expression SELECT 5*"23" results in 115 because the string “25” converts to an the value 25 as an integer, but the expression SELECT 5 * " " results in null because casting the empty string to an integer gives null. Make sure that your intermediate datasets have the schema you expect them to (try using printSchema on them), and look for any CAST operations in the final query plan.

    确保您的转换实际上产生有效的查询计划。 Spark SQL有时会执行隐式类型强制，这可能会导致混乱的结果。例如，SQL表达式SELECT 5 *“23”导致115，因为字符串“25”将整数转换为值25，但表达式SELECT 5 *“”导致null，因为将空字符串转换为整数给出null。确保您的中间数据集具有您期望的模式（尝试对它们使用printSchema），并在最终查询计划中查找任何CAST操作。

​    

### <font color="#000000">No Space Left on Disk Errors 磁盘错误没有剩余空间</font>

#### <font color="#3399cc">Signs and symptoms 迹象和症状</font>
- You see “no space left on disk” errors and your jobs fail.

  您看到“磁盘上没有剩余空间”错误，您的作业失败。

#### <font color="#3399cc">Potential treatments 潜在的治疗</font>

- The easiest way to alleviate this, of course, is to add more disk space. You can do this by sizing up the nodes that you’re working on or attaching external storage in a cloud environment.

    当然，减轻这种情况的最简单方法是添加更多磁盘空间。您可以通过调整正在处理的节点或在云环境中连接外部存储来实现此目的。

- If you have a cluster with limited storage space, some nodes may run out first due to skew.

  如果您的集群存储空间有限，则某些节点可能会由于数据倾斜而首先耗尽。

- Repartitioning the data as described earlier may help here.

  如前所述重新分区数据可能对此有所帮助。

- There are also a number of storage configurations with which you can experiment. Some of these determine how long logs should be kept on the machine before being removed. For more information, see the Spark executor logs rolling configurations in Chapter 16.

  您还可以使用许多存储配置进行试验。其中一些决定了在移除之前应该在机器上保留多长时间的日志。有关更多信息，请参阅第16章中的Spark执行程序(executor)日志滚动配置。

- Try manually removing some old log files or old shuffle files from the machine(s) in question. This can help alleviate some of the issue although obviously it’s not a permanent fix.

  尝试从相关机器手动删除一些旧的日志文件或旧的数据再分配文件。这可以帮助减轻一些问题，虽然显然它不是永久性的修复。

### <font color="#000000">Serialization Errors 序列化错误</font>
#### <font color="#3399cc">Signs and symptoms 迹象和症状</font>
- You see serialization errors and your jobs fail.
您看到序列化错误，您的作业失败。

#### <font color="#3399cc">Potential treatments潜在的治疗方法</font>

- This is very uncommon when working with the Structured APIs, but you might be trying to perform some custom logic on executors with UDFs or RDDs and either the task that you’re trying to serialize to these executors or the data you are trying to share cannot be serialized. This often happens when you’re working with either some code or data that cannot be serialized into a UDF or function, or if you’re working with strange data types that cannot be serialized. If you are using (or intend to be using Kryo serialization), verify that you’re actually registering your classes so that they are indeed serialized.

  这在使用结构化API时非常罕见，但您可能尝试使用UDF或RDD在执行程序(executor)上执行某些自定义逻辑，以及您尝试序列化到这些执行程序(executor)的任务或您尝试共享的数据无法序列化。当您使用某些无法序列化为UDF或函数的代码或数据时，或者您正在使用无法序列化的奇怪数据类型时，通常会发生这种情况。如果您正在使用（或打算使用Kryo序列化），请验证您实际上是在注册类，以便它们确实是序列化的。

- Try not to refer to any fields of the enclosing object in your UDFs when creating UDFs inside a Java or Scala class. This can cause Spark to try to serialize the whole enclosing object, which may not be possible. Instead, copy the relevant fields to local variables in the same scope as closure and use those.

  在Java或Scala类中创建UDF时，尽量不要引用UDF中封闭对象的任何字段。这可能导致Spark尝试序列化整个封闭对象，这可能是不可能的。相反，将相关字段复制到与闭包相同的范围内的局部变量并使用它们。


## <font style="color:#9a161a">Conclusion 结论</font>

This chapter covered some of the main tools that you can use to monitor and debug your Spark jobs and applications, as well as the most common issues we see and their resolutions. As with debugging any complex software, we recommend taking a principled, step-by-step approach to debug issues. Add logging statements to figure out where your job is crashing and what type of data arrives at each stage, try to isolate the problem to the smallest piece of code possible, and work up from there. For data skew issues, which are unique to parallel computing, use Spark’s UI to get a quick overview of how much work each task is doing. In Chapter 19, we discuss performance tuning in particular and various tools you can use for that. 

本章介绍了一些可用于监视和调试Spark作业和应用程序的主要工具，以及我们看到的最常见问题及其解决方案。与调试任何复杂软件一样，我们建议采用有原则的逐步方法来调试问题。添加日志记录语句以确定作业崩溃的位置以及每个阶段到达的数据类型，尝试将问题隔离到可能的最小代码段，并从那里开始工作。对于并行计算所特有的数据偏差问题，请使用Spark的UI快速了解每项任务的工作量。在第19章中，我们特别讨论了性能调优以及可以使用的各种工具。