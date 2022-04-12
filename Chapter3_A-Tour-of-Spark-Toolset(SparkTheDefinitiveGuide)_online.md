---
title: 翻译 Chapter 3 A Tour of Spark’s Toolset 
date: 2019-11-07
copyright: true
categories: English
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 3. A Tour of Spark’s Toolset

<center><strong>译者</strong>：<u><a style="color:#0879e3" href="https://snaildove.github.io">https://snaildove.github.io</a></u></center>

In Chapter 2, we introduced Spark’s core concepts, like transformations and actions, in the context of Spark’s Structured APIs. These simple conceptual building blocks are the foundation of Apache Spark’s vast ecosystem of tools and libraries (Figure 3-1). Spark is composed of these primitives— the lower-level APIs and the Structured APIs—and then a series of standard libraries for additional functionality.

在第2章中，我们在Spark的结构化API中介绍了Spark的核心概念，例如转换和动作。这些简单的概念构建块是Apache Spark庞大的工具和库生态系统的基础（图3-1）。Spark由这些基础元素（低阶API和结构化API）以及一系列用于附加功能的标准库组成。

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter3/1574419267522.png" alt="1122" style="zoom:80%;" />

Spark’s libraries support a variety of different tasks, from graph analysis and machine learning to streaming and integrations with a host of computing and storage systems. This chapter presents a whirlwind tour of much of what Spark has to offer, including some of the APIs we have not yet covered and a few of the main libraries. For each section, you will find more detailed information in other parts of this book; our purpose here is provide you with an overview of what’s possible.

Spark的库支持各种不同的任务，从图形分析和机器学习到流以及与大量计算和存储系统的集成。本章介绍了Spark所提供的许多功能，包括一些我们尚未介绍的API和一些主要的库。对于每一部分，您都可以在本书的其他部分找到更详细的信息。我们的目的是为您提供可能的概览。

This chapter covers the following:

本章内容如下：

- Running production applications with `spark-submit` 

    通过 `spark-submit` 来运行生产应用程序

- Datasets: type-safe APIs for structured data Datasets：

    用于结构化数据的类型安全的API

- Structured Streaming 

    结构化流

- Machine learning and advanced analytics 

    机器学习和高级分析 

- Resilient Distributed Datasets (RDD): Spark’s low level APIs 

    弹性分布式数据集（RDD）：Spark的低阶API

- SparkR 

- The third-party package ecosystem 

    第三方软件包生态系统

After you’ve taken the tour, you’ll be able to jump to the corresponding parts of the book to find answers to your questions about particular topics.

游览之后，您可以跳到本书的相应部分，以找到有关特定主题的问题的答案。

## <font color="#9a161a">Running Production Application</font>

Spark makes it easy to develop and create big data programs. Spark also makes it easy to turn your interactive exploration into production applications with `spark-submit`, a built-in command-line tool. spark-submit does one thing: it lets you send your application code to a cluster and launch it to execute there. Upon submission, the application will run until it exits (completes the task) or encounters an error. You can do this with all of Spark’s support cluster managers including Standalone, Mesos, and YARN.

Spark使开发和创建大数据程序变得容易。使用内置的命令行工具 `spark-submit`，Spark还可以轻松地将交互式探索转变为生产应用程序。spark-submit做一件事：它使您可以将应用程序代码发送到集群并启动它以在其中执行。提交后，应用程序将运行，直到退出（完成任务）或遇到错误。您可以使用Spark的所有支持集群管理器（包括Standalone，Mesos和YARN）来执行此操作。

`spark-submit` offers several controls with which you can specify the resources your application needs as well as how it should be run and its command-line arguments.

`spark-submit` 提供了几个控制（选项），您可以使用这些控制（选项）指定应用程序所需的资源以及应如何运行该应用程序及其命令行参数。

You can write applications in any of Spark’s supported languages and then submit them for execution. The simplest example is running an application on your local machine. We’ll show this by running a sample Scala application that comes with Spark, using the following command in the directory where you downloaded Spark:

您可以使用Spark支持的任何语言编写应用程序，然后将其提交执行。最简单的示例是在本地计算机上运行应用程序。我们将通过运行Spark随附的示例Scala应用程序（在您下载Spark的目录中使用以下命令）来显示此信息：

```shell
./bin/spark-submit \
	--class org.apache.spark.examples.SparkPi \
	--master local \
	./examples/jars/spark-examples_2.11-2.2.0.jar 10
```

This sample application calculates the digits of pi to a certain level of estimation. Here, we’ve told `spark-submit` that we want to run on our local machine, which class and which JAR we would like to run, and some command-line arguments for that class.

该示例应用程序将 $\pi$ 的位数计算为一定程度的估计。在这里，我们告诉 `spark-submit`，我们要在本地计算机上运行，我们要运行哪个类和哪个JAR，以及该类的一些命令行参数。

We can also run a Python version of the application using the following command:

我们还可以使用以下命令运行该应用程序的Python版本：

```shell
./bin/spark-submit \
	--master local \
	./examples/src/main/python/pi.py 10
```

By changing the master argument of spark-submit, we can also submit the same application to a cluster running Spark’s standalone cluster manager, Mesos or YARN.

通过更改spark-submit的主参数，我们还可以将同一应用程序提交给运行Spark独立的集群管理器Mesos或YARN的集群。

`spark-submit` will come in handy to run many of the examples we’ve packaged with this book. In the rest of this chapter, we’ll go through examples of some APIs that we haven’t yet seen in our introduction to Spark.

在运行与本书一起打包的许多示例时，`spark-submit` 将派上用场。在本章的其余部分，我们将介绍在Spark简介中尚未见到的一些API的示例。

## <font color="#9a161a">Datasets: Type-Safe Structured APIs</font>

The first API we’ll describe is a type-safe version of Spark’s structured API called Datasets, for writing statically typed code in Java and Scala. The Dataset API is not available in Python and R, because those languages are dynamically typed.

我们将介绍的第一个API是Spark结构化API的类型安全版本，称为 Datasets，用于以Java和Scala编写静态类型的代码。Datasets API在Python和R中不可用，因为这些语言是动态类型的。

Recall that DataFrames, which we saw in the previous chapter, are a distributed collection of objects of type Row that can hold various types of tabular data. The Dataset API gives users the ability to assign a Java/Scala class to the records within a DataFrame and manipulate it as a collection of typed objects, similar to a Java ArrayList or Scala Seq. The APIs available on Datasets are type-safe, meaning that you cannot accidentally view the objects in a Dataset as being of another class than the class you put in initially. This makes Datasets especially attractive for writing large applications, with which multiple software engineers must interact through well-defined interfaces.

回想一下，我们在上一章中看到的DataFrames是Row类型的对象的分布式集合，这些对象可以保存各种类型的表格数据。Dataset API使用户能够将 Java/Scala 类分配给DataFrame中的记录，并将其作为类型化对象的集合进行操作，类似于Java ArrayList 或 Scala Seq。Datasets 上可用的API是类型安全的，这意味着您不能意外地将 Dataset 中的对象视为不同于最初放置的类的其他类。这使得 Datasets 对于编写大型应用程序特别有吸引力，多个软件工程师必须通过定义良好的接口与之进行交互。

The Dataset class is parameterized with the type of object contained inside: Dataset<T> in Java and Dataset[T] in Scala. For example, a Dataset[Person] will be guaranteed to contain objects of class Person. As of Spark 2.0, the supported types are classes following the JavaBean pattern in Java and case classes in Scala. These types are restricted because Spark needs to be able to automatically analyze the type T and create an appropriate schema for the tabular data within your Dataset.

使用内部包含对象的类型对Dataset类进行参数化：Java中的 `Dataset<T>` 和 Scala 中的 `Dataset[T]`。例如，将保证Dataset[Person] 包含Person类的对象。从Spark 2.0开始，支持的类型是Java中遵循JavaBean模式的类以及Scala中的案例类。这些类型受到限制，因为Spark需要能够自动分析类型T并为数据集中的表格数据创建适当的模式。

One great thing about Datasets is that you can use them only when you need or want to. For instance, in the following example, we’ll define our own data type and manipulate it via arbitrary map and filter functions. After we’ve performed our manipulations, Spark can automatically turn it back into a DataFrame, and we can manipulate it further by using the hundreds of functions that Spark includes. This makes it easy to drop down to lower level, perform type-safe coding when necessary, and move higher up to SQL for more rapid analysis. Here is a small example showing how you can use both type-safe functions and DataFrame-like SQL expressions to quickly write business logic:

Datasets 的一大优点是，只有在需要或想要时才可以使用它们。例如，在以下示例中，我们将定义自己的数据类型，并通过任意的map和filter函数对其进行操作。执行完操作后，Spark可以自动将其转换回DataFrame，并且可以使用Spark包含的数百种函数对其进行进一步操作。这样可以轻松地降低到较低阶（的API），在必要时执行类型安全的编码，并可以将其上移至SQL以进行更快的分析。这是一个小示例，展示了如何使用类型安全函数和类似DataFrame的SQL表达式来快速编写业务逻辑：

```scala
// in Scala
case class Flight(DEST_COUNTRY_NAME: String, ORIGIN_COUNTRY_NAME: String, count: BigInt)
val flightsDF = spark.read.parquet("/data/flight-data/parquet/2010-summary.parquet/")
val flights = flightsDF.as[Flight]
```

One final advantage is that when you call collect or take on a Dataset, it will collect objects of the proper type in your Dataset, not DataFrame Rows. This makes it easy to get type safety and securely perform manipulation in a distributed and a local manner without code changes:

最后一个优点是，当您调用收集或使用数据集时，它将收集数据集中正确类型的对象，而不是 DataFrame Rows。这使得轻松获得类型安全性并以分布式和本地方式安全地执行操作而无需更改代码：

```scala
// in Scala
flights
.filter(flight_row => flight_row.ORIGIN_COUNTRY_NAME != "Canada")
.map(flight_row => flight_row)
.take(5)

flights
.take(5)
.filter(flight_row => flight_row.ORIGIN_COUNTRY_NAME != "Canada")
.map(fr => Flight(fr.DEST_COUNTRY_NAME, fr.ORIGIN_COUNTRY_NAME, fr.count + 5))
```

We cover Datasets in depth in Chapter 11.

我们将在第11章中深入介绍 Datasets。

## <font color="#9a161a">Structured Streaming</font>

Structured Streaming is a high-level API for stream processing that became production-ready in Spark 2.2. With Structured Streaming, you can take the same operations that you perform in batch mode using Spark’s structured APIs and run them in a streaming fashion. This can reduce latency and allow for incremental processing. The best thing about Structured Streaming is that it allows you to rapidly and quickly extract value out of streaming systems with virtually no code changes. It also makes it easy to conceptualize because you can write your batch job as a way to prototype it and then you can convert it to a streaming job. The way all of this works is by incrementally processing that data.

结构化流是用于流处理的高层API，在 Spark 2.2 中已投入生产。借助结构化流，您可以执行与使用Spark的结构化API以批处理模式执行的相同操作，并以流式方式运行它们。这可以减少等待时间并允许增量处理。关于结构化流技术的最好之处在于，它使您能够快速地从流系统中提取价值，而几乎无需更改代码。这也使概念化变得容易，因为您可以编写批处理作业以将其原型化，然后将其转换为流式作业。所有这些工作的方式是通过逐步处理该数据。

Let’s walk through a simple example of how easy it is to get started with Structured Streaming. For this, we will use a <u><a style="color:#0879e3" href="https://github.com/databricks/Spark-The-Definitive-Guide/tree/master/data/retail-data">retail dataset</a></u>, one that has specific dates and times for us to be able to use. We will use the “by-day” set of files, in which one file represents one day of data.

让我们看一个简单的示例，说明开始使用结构化流技术有多么容易。为此，我们将使用<u><a style="color:#0879e3" href="https://github.com/databricks/Spark-The-Definitive-Guide/tree/master/data/retail-data">零售数据集</a></u>，其中包含可使用的特定日期和时间。我们将使用“按天”文件集，其中一个文件代表一天的数据。

We put it in this format to simulate data being produced in a consistent and regular manner by a different process. This is retail data so imagine that these are being produced by retail stores and sent to a location where they will be read by our Structured Streaming job.

我们将其以这种格式放置，以模拟通过不同过程以一致且规则的方式生成的数据。这是零售数据，因此可以想象这些是由零售商店生产的，并发送到我们的“结构化流”作业可以读取的位置。

It’s also worth sharing a sample of the data so you can reference what the data looks like:

还值得分享数据样本，以便您可以参考数据的外观：

```txt
InvoiceNo,StockCode,Description,Quantity,InvoiceDate,UnitPrice,CustomerID,Country
536365,   85123A,   WHITE HANGING HEART T-LIGHT HOLDER,6,2010-12-01 08:26:00,2.55,17...
536365,   71053,    WHITE METAL LANTERN,6,2010-12-01 08:26:00,3.39,17850.0,United Kin...
536365,   84406B,   CREAM CUPID HEARTS COAT HANGER,8,2010-12-01 08:26:00,2.75,17850...
```

To ground this, let’s first analyze the data as a static dataset and create a DataFrame to do so. We’ll also create a schema from this static dataset (there are ways of using schema inference with streaming that we will touch on in Part V):

为此，我们首先将数据分析为静态数据集，然后创建一个DataFrame进行分析。我们还将从此静态数据集中创建一个模式（在第五部分中将介绍用流进行模式推断的方法）：

```scala
// in Scala
val staticDataFrame = spark.read.format("csv")
.option("header", "true")
.option("inferSchema", "true")
.load("/data/retail-data/by-day/*.csv")
staticDataFrame.createOrReplaceTempView("retail_data")
val staticSchema = staticDataFrame.schema
```

```python
# in Python
staticDataFrame = spark.read.format("csv")\
.option("header", "true")\
.option("inferSchema", "true")\
.load("/data/retail-data/by-day/*.csv")
staticDataFrame.createOrReplaceTempView("retail_data")
staticSchema = staticDataFrame.schema
```

Because we’re working with time–series data, it’s worth mentioning how we might go along grouping and aggregating our data. In this example we’ll take a look at the sale hours during which a given customer (identified by `CustomerId`) makes a large purchase. For example, let’s add a total cost column and see on what days a customer spent the most.

由于我们正在处理时间序列数据，因此值得一提的是我们可能如何对数据进行分组和汇总。在此示例中，我们将查看特定客户（由 `CustomerId` 标识）进行大笔交易的销售时间。例如，让我们添加一个“总费用”列，然后查看客户花费最多的几天。

The window function will include all data from each day in the aggregation. It’s simply a window over the time–series column in our data. This is a helpful tool for manipulating date and timestamps because we can specify our requirements in a more human form (via intervals), and Spark will group all of them together for us : 

窗口函数将包括汇总中每天的所有数据。它只是我们数据中基于“时间序列”这一列的一个窗口。这是处理日期和时间戳的有用工具，因为我们可以以更人性化的形式（通过时间间隔）指定需求，Spark会为我们将所有需求分组在一起：

```scala
// in Scala
import org.apache.spark.sql.functions.{window, column, desc, col}
staticDataFrame
.selectExpr(
"CustomerId",
"(UnitPrice * Quantity) as total_cost",
"InvoiceDate")
.groupBy(
col("CustomerId"), window(col("InvoiceDate"), "1 day"))
.sum("total_cost")
.show(5)
```

```python
# in Python
from pyspark.sql.functions import window, column, desc, col
staticDataFrame\
.selectExpr(
"CustomerId",
"(UnitPrice * Quantity) as total_cost",
"InvoiceDate")\
.groupBy(
col("CustomerId"), window(col("InvoiceDate"), "1 day"))\.sum("total_cost")\
.show(5)
```

It’s worth mentioning that you can also run this as SQL code, just as we saw in the previous chapter. Here’s a sample of the output that you’ll see:

值得一提的是，您也可以将其作为SQL代码运行，就像在上一章中看到的那样。这是您将看到的输出示例：

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter3/1574422330532.png" alt="112" style="zoom:67%;" />

The null values represent the fact that we don’t have a customerId for some transactions.

空值表示以下事实：我们没有某些交易的customerId。

That’s the static DataFrame version; there shouldn’t be any big surprises in there if you’re familiar with the syntax.

那是静态的DataFrame版本；如果您熟悉语法，那么应该不会有什么大的惊喜。

Because you’re likely running this in local mode, it’s a good practice to set the number of shuffle partitions to something that’s going to be a better fit for local mode. This configuration specifies the number of partitions that should be created after a shuffle. By default, the value is 200, but because there aren’t many executors on this machine, it’s worth reducing this to 5. We did this same operation in Chapter 2, so if you don’t remember why this is important, feel free to flip back to review.

由于您可能会在本地模式下运行此程序，因此最好将数据再分配分区的数量设置为更适合本地模式的数量。此配置指定数据再分配后应创建的分区数。默认情况下，该值为200，但是由于这台机器上的执行程序（executor）并不多，因此值得将其减少为5。我们在第2章中进行了相同的操作，因此，如果您不记得为什么这样做很重要，请放心后退以进行查看。

```scala
spark.conf.set("spark.sql.shuffle.partitions", "5")
```

Now that we’ve seen how that works, let’s take a look at the streaming code! You’ll notice that very little actually changes about the code. The biggest change is that we used `readStream` instead of `read`, additionally you’ll notice the `maxFilesPerTrigger` option, which simply specifies the number of files we should read in at once. This is to make our demonstration more “streaming,” and in a production scenario this would probably be omitted.

现在我们已经了解了它的工作原理，下面让我们看一下流代码！您会注意到，实际上对代码所做的更改很少。最大的变化是我们使用`readStream`代替了`read`，此外，您还会注意到`maxFilesPerTrigger`选项，该选项仅指定我们一次应读取的文件数。这是为了使我们的演示更加“流畅”，在生产场景中，可能会省略。

```scala
val streamingDataFrame = spark.readStream
.schema(staticSchema)
.option("maxFilesPerTrigger", 1)
.format("csv")
.option("header", "true")
.load("/data/retail-data/by-day/*.csv")
```

```python
# in Python
streamingDataFrame = spark.readStream\
.schema(staticSchema)\
.option("maxFilesPerTrigger", 1)\
.format("csv")\
.option("header", "true")\
.load("/data/retail-data/by-day/*.csv")
```

Now we can see whether our DataFrame is streaming:

现在我们可以看看我们的DataFrame是否正在流式传输：

```scala
streamingDataFrame.isStreaming // returns true
```

Let’s set up the same business logic as the previous DataFrame manipulation. We’ll perform a summation in the process:

让我们设置与以前的DataFrame操作相同的业务逻辑。我们将在此过程中进行汇总：

```scala
// in Scala
val purchaseByCustomerPerHour = streamingDataFrame
.selectExpr(
"CustomerId",
"(UnitPrice * Quantity) as total_cost",
"InvoiceDate")
.groupBy(
$"CustomerId", window($"InvoiceDate", "1 day"))
.sum("total_cost")
```

```python
# in Python
purchaseByCustomerPerHour = streamingDataFrame\
.selectExpr(
"CustomerId",
"(UnitPrice * Quantity) as total_cost",
"InvoiceDate")\
.groupBy(
col("CustomerId"), window(col("InvoiceDate"), "1 day"))\
.sum("total_cost")
```

This is still a lazy operation, so we will need to call a streaming action to start the execution of this data flow.

这仍然是一个懒惰的操作，因此我们将需要调用流操作来开始执行此数据流。

Streaming actions are a bit different from our conventional static action because we’re going to be populating data somewhere instead of just calling something like count (which doesn’t make any sense on a stream anyways). The action we will use will output to an in-memory table that we will update after each *trigger*. In this case, each trigger is based on an individual file (the read option that we set). Spark will mutate the data in the in-memory table such that we will always have the highest value as specified in our previous aggregation:

流操作与常规的静态操作有所不同，因为我们将在某个地方填充数据，而不是仅仅调用诸如count之类的东西（无论如何，这对流没有任何意义）。我们将使用的操作将输出到一个内存表中，我们将在每次触发后对其进行更新。在这种情况下，每个触发器都基于一个单独的文件（我们设置的读取选项）。Spark将对内存表中的数据进行突变，以使我们将始终具有先前聚合中指定的最高值：

```scala
// in Scala
purchaseByCustomerPerHour.writeStream
.format("memory") // memory = store in-memory table
.queryName("customer_purchases") // the name of the in-memory table
.outputMode("complete") // complete = all the counts should be in the table
.start()
```

```python
# in Python
purchaseByCustomerPerHour.writeStream\
.format("memory")\
.queryName("customer_purchases")\
.outputMode("complete")\
.start()
```

When we start the stream, we can run queries against it to debug what our result will look like if we were to write this out to a production sink:

启动流时，可以对它运行查询进行调试将结果写到生产环境的接收器后的结果：

```scala
// in Scala
spark.sql("""
SELECT *
FROM customer_purchases
ORDER BY `sum(total_cost)` DESC
""")
.show(5)
```

```python
# in Python
spark.sql("""
SELECT *
FROM customer_purchases
ORDER BY `sum(total_cost)` DESC
""")\
.show(5)
```

You’ll notice that the composition of our table changes as we read in more data! With each file, the results might or might not be changing based on the data. Naturally, because we’re grouping customers, we hope to see an increase in the top customer purchase amounts over time (and do for a period of time!). Another option you can use is to write the results out to the console:

您会注意到，随着我们读取更多数据，表的组成也会发生变化！对于每个文件，结果可能会或可能不会根据数据而改变。自然，因为我们将客户分组，所以我们希望随着时间的推移（并持续一段时间！），客户的最大购买量会增加。您可以使用的另一个选项是将结果写到控制台：

```scala
purchaseByCustomerPerHour.writeStream
.format("console")
.queryName("customer_purchases_2")
.outputMode("complete")
.start()
```

You shouldn’t use either of these streaming methods in production, but they do make for convenient demonstration of Structured Streaming’s power. Notice how this window is built on event time, as well, not the time at which Spark processes the data. This was one of the shortcomings of Spark Streaming that Structured Streaming has resolved. We cover Structured Streaming in depth in Part V.

您不应该在生产中使用这两种流传输方法中的任何一种，但是它们确实可以方便地演示结构化流传输的功能。请注意，此窗口也是基于事件时间构建的，而不是基于Spark处理数据的时间。这是结构化流已解决的Spark流的缺点之一。我们将在第五部分深入介绍结构化流。

## <font color="#9a161a">Machine Learning and Advanced Analytics</font>

Another popular aspect of Spark is its ability to perform large-scale machine learning with a built-in library of machine learning algorithms called MLlib. MLlib allows for preprocessing, munging, training of models, and making predictions at scale on data. You can even use models trained in MLlib to make predictions in Strucutred Streaming. Spark provides a sophisticated machine learning API for performing a variety of machine learning tasks, from classification to regression, and clustering to deep learning. To demonstrate this functionality, we will perform some basic clustering on our data using a standard algorithm called -means.

Spark的另一个受欢迎的方面是它具有使用称为MLlib的内置机器学习算法库执行大规模机器学习的能力。MLlib允许进行预处理，修改，模型训练以及对数据进行大规模预测。您甚至可以使用在MLlib中训练的模型在Strucutred Streaming中进行预测。Spark提供了完善的机器学习API，可用于执行各种机器学习任务，从分类到回归，再到集群再到深度学习。为了演示此功能，我们将使用称为-means的标准算法对数据执行一些基本的聚类。

---

<p><center><font face="constant-width" color="#737373" size=3><strong>WHAT IS K-MEANS?</strong></font></center></P>
-means is a clustering algorithm in which “” centers are randomly assigned within the data. The points closest to that point are then “assigned” to a class and the center of the assigned points is computed. This center point is called the centroid. We then label the points closest to that centroid, to the centroid’s class, and shift the centroid to the new center of that cluster of points. We repeat this process for a finite set of iterations or until convergence (our center points stop changing).

-means是一种聚类算法，其中在数据内随机分配“”中心。然后，将最接近该点的点“分配”给一个类，并计算分配点的中心。该中心点称为质心（centroid）。然后，我们标记最接近该质心（centroid），质心类别的点，然后将质心移动到该点簇的新中心。我们对有限的一组迭代或直到收敛（我们的中心点停止更改）重复此过程。

---

Spark includes a number of preprocessing methods out of the box. To demonstrate these methods, we will begin with some raw data, build up transformations before getting the data into the right format, at which point we can actually train our model and then serve predictions:

Spark包括许多现成的预处理方法。为了演示这些方法，我们将从一些原始数据开始，在将数据转换为正确格式之前先进行转换，然后才能实际训练模型，然后进行预测：

```scala
staticDataFrame.printSchema()
```

```txt
root
|-- InvoiceNo: string (nullable = true)
|-- StockCode: string (nullable = true)
|-- Description: string (nullable = true)
|-- Quantity: integer (nullable = true)
|-- InvoiceDate: timestamp (nullable = true)
|-- UnitPrice: double (nullable = true)
|-- CustomerID: double (nullable = true)
|-- Country: string (nullable = true)
```

Machine learning algorithms in MLlib require that data is represented as numerical values. Our current data is represented by a variety of different types, including timestamps, integers, and strings. Therefore we need to transform this data into some numerical representation. In this instance, we’ll use several DataFrame transformations to manipulate our date data:

MLlib中的机器学习算法要求将数据表示为数值。我们当前的数据由各种不同的类型表示，包括时间戳，整数和字符串。因此，我们需要将此数据转换为某种数字表示形式。在这种情况下，我们将使用几种DataFrame转换来操纵日期数据：

```scala
// in Scala
import org.apache.spark.sql.functions.date_format
val preppedDataFrame = staticDataFrame
.na.fill(0)
.withColumn("day_of_week", date_format($"InvoiceDate", "EEEE"))
.coalesce(5)
```

```python
# in Python
from pyspark.sql.functions import date_format, col
preppedDataFrame = staticDataFrame\
.na.fill(0)\
.withColumn("day_of_week", date_format(col("InvoiceDate"), "EEEE"))\
.coalesce(5)
```

We are also going to need to split the data into training and test sets. In this instance, we are going to do this manually by the date on which a certain purchase occurred; however, we could also use MLlib’s transformation APIs to create a training and test set via train validation splits or cross validation (these topics are covered at length in Part VI):

我们还需要将数据分为训练集和测试集。在这种情况下，我们将在特定购买发生的日期之前手动执行此操作；但是，我们也可以使用MLlib的转换API通过训练验证拆分或交叉验证来创建训练和测试集（这些主题在第VI部分中进行了详细介绍）：

```scala
// in Scala
val trainDataFrame = preppedDataFrame
.where("InvoiceDate < '2011-07-01'")
val testDataFrame = preppedDataFrame
.where("InvoiceDate >= '2011-07-01'")
```

```python
# in Python
trainDataFrame = preppedDataFrame\
.where("InvoiceDate < '2011-07-01'")
testDataFrame = preppedDataFrame\
.where("InvoiceDate >= '2011-07-01'")
```

Now that we’ve prepared the data, let’s split it into a training and test set. Because this is a time– series set of data, we will split by an arbitrary date in the dataset. Although this might not be the optimal split for our training and test, for the intents and purposes of this example it will work just fine. We’ll see that this splits our dataset roughly in half:

现在我们已经准备好数据，让我们将其分为训练和测试集。因为这是一个时间序列数据集，所以我们将在数据集中按任意日期分割。尽管对于我们的训练和测试而言，这可能不是最佳选择，但出于本示例的目的，它仍然可以正常工作。我们将看到这将数据集大致分为两半：

```scala
trainDataFrame.count()
testDataFrame.count()
```

Note that these transformations are DataFrame transformations, which we cover extensively in Part II. Spark’s MLlib also provides a number of transformations with which we can automate some of our general transformations. One such transformer is a `StringIndexer`:

请注意，这些转换是DataFrame转换，我们将在第二部分中进行广泛讨论。Spark的MLlib还提供了许多转换，通过这些转换我们可以自动化一些常规转换。这样的转换器就是 `StringIndexer`：

```scala
// in Scala
import org.apache.spark.ml.feature.StringIndexer
val indexer = new StringIndexer()
.setInputCol("day_of_week")
.setOutputCol("day_of_week_index")
```

```python
# in Python
from pyspark.ml.feature import StringIndexer
indexer = StringIndexer()\
.setInputCol("day_of_week")\
.setOutputCol("day_of_week_index")
```

This will turn our days of weeks into corresponding numerical values. For example, Spark might represent Saturday as 6, and Monday as 1. However, with this numbering scheme, we are implicitly stating that Saturday is greater than Monday (by pure numerical values). This is obviously incorrect. To fix this, we therefore need to use a `OneHotEncoder` to encode each of these values as their own column. These Boolean flags state whether that day of week is the relevant day of the week:

这会将我们的星期几转换为相应的数值。例如，Spark可能将星期六表示为6，将星期一表示为1。但是，使用此编号方案，我们隐式地指出，星期六大于星期一（按纯数值）。这显然是不正确的。为了解决这个问题，因此我们需要使用`OneHotEncoder`将这些值中的每一个编码为自己的列。这些布尔标志说明星期几是否是星期几的相关日期：

```scala
// in Scala
import org.apache.spark.ml.feature.OneHotEncoder
val encoder = new OneHotEncoder()
.setInputCol("day_of_week_index")
.setOutputCol("day_of_week_encoded")
```

```python
# in Python
from pyspark.ml.feature import OneHotEncoder
encoder = OneHotEncoder()\
.setInputCol("day_of_week_index")\
.setOutputCol("day_of_week_encoded")
```

Each of these will result in a set of columns that we will “assemble” into a vector. All machine learning algorithms in Spark take as input a Vector type, which must be a set of numerical values:

这些中的每一个都会产生一组列，我们将这些列“组合”为向量。Spark中的所有机器学习算法都将Vector类型作为输入，该类型必须是一组数值：

```scala
// in Scala
import org.apache.spark.ml.feature.VectorAssembler
val vectorAssembler = new VectorAssembler()
.setInputCols(Array("UnitPrice", "Quantity", "day_of_week_encoded"))
.setOutputCol("features")
```

```python
# in Python
from pyspark.ml.feature import VectorAssembler
vectorAssembler = VectorAssembler()\
.setInputCols(["UnitPrice", "Quantity", "day_of_week_encoded"])\
.setOutputCol("features")
```

Here, we have three key features: the price, the quantity, and the day of week. Next, we’ll set this up into a pipeline so that any future data we need to transform can go through the exact same process:

在这里，我们具有三个主要功能：价格，数量和星期几。接下来，我们将其设置为管道，以便将来需要转换的所有数据都可以经过完全相同的过程：

```scala
// in Scala
import org.apache.spark.ml.Pipeline
val transformationPipeline = new Pipeline()
.setStages(Array(indexer, encoder, vectorAssembler))
```

```python
# in Python
from pyspark.ml import Pipeline
transformationPipeline = Pipeline()\
.setStages([indexer, encoder, vectorAssembler])
```

Preparing for training is a two-step process. We first need to fit our transformers to this dataset. We cover this in depth in Part VI, but basically our `StringIndexer` needs to know how many unique values there are to be indexed. After those exist, encoding is easy but Spark must look at all the distinct values in the column to be indexed in order to store those values later on:

准备训练是一个分为两个步骤的过程。我们首先需要使我们的转换器（transformer）拟合该数据集。我们将在第六部分中对此进行深入介绍，但是基本上我们的`StringIndexer`需要知道要索引多少个唯一值。这些值存在之后，编码就很容易了，但是Spark必须查看要索引的列中的所有不同值，以便以后存储这些值：

```scala
// in Scala
val fittedPipeline = transformationPipeline.fit(trainDataFrame)
```

```python
# in Python
fittedPipeline = transformationPipeline.fit(trainDataFrame)
```

After we fit the training data, we are ready to take that fitted pipeline and use it to transform all of our data in a consistent and repeatable way:

拟合训练数据后，我们准备采用已经拟合的管道，并使用这个管道以一致且可重复的方式转换所有数据：

```scala
// in Scala
val transformedTraining = fittedPipeline.transform(trainDataFrame)
```

```python
# in Python
transformedTraining = fittedPipeline.transform(trainDataFrame)
```

At this point, it’s worth mentioning that we could have included our model training in our pipeline. We chose not to in order to demonstrate a use case for caching the data. Instead, we’re going to perform some hyperparameter tuning on the model because we do not want to repeat the exact same transformations over and over again; specifically, we’ll use caching, an optimization that we discuss in more detail in Part IV. This will put a copy of the intermediately transformed dataset into memory, allowing us to repeatedly access it at much lower cost than running the entire pipeline again. If you’re curious to see how much of a difference this makes, skip this line and run the training without caching the data. Then try it after caching; you’ll see the results are significant:

在这一点上，值得一提的是，我们可以将我们的模型训练纳入我们的流程中。我们选择不这样做是为了演示用于缓存数据的用例。相反，我们将对模型执行一些超参数调整，因为我们不想一遍又一遍地重复完全相同的转换。具体来说，我们将使用缓存，该优化将在第四部分中详细讨论。这会将中间转换后的数据集的副本放入内存中，从而使我们能够以比再次运行整个管道更低的成本重复访问它。如果您想知道这有什么不同，请跳过此行并进行训练，而无需缓存数据。然后在缓存后尝试；您会看到效果显着：

```scala
transformedTraining.cache()
```

We now have a training set; it’s time to train the model. First we’ll import the relevant model that we’d like to use and instantiate it:

我们现在有一个训练集；是时候训练模型了。首先，我们导入要使用的相关模型并实例化：

```scala
// in Scala
import org.apache.spark.ml.clustering.KMeans
val kmeans = new KMeans()
.setK(20)
.setSeed(1L)
```

```python
# in Python
from pyspark.ml.clustering import KMeans
kmeans = KMeans()\
.setK(20)\
.setSeed(1L)
```

In Spark, training machine learning models is a two-phase process. First, we initialize an untrained model, and then we train it. There are always two types for every algorithm in MLlib’s DataFrame API. They follow the naming pattern of Algorithm, for the untrained version, and `AlgorithmModel` for the trained version. In our example, this is `KMeans` and then `KMeansModel`.

在Spark中，训练机器学习模型是一个分为两个阶段的过程。首先，我们初始化未训练的模型，然后训练它。MLlib的DataFrame API中的每种算法总是有两种类型。对于未训练的版本，它们遵循算法的命名模式，对于训练的版本，它们遵循 `AlgorithmModel` 的命名模式。在我们的示例中，这是 `KMeans`，然后是 `KMeansModel`。

Estimators in MLlib’s DataFrame API share roughly the same interface that we saw earlier with our preprocessing transformers like the `StringIndexer`. This should come as no surprise because it makes training an entire pipeline (which includes the model) simple. For our purposes here, we want to do things a bit more step by step, so we chose to not do this in this example:

MLlib的DataFrame API中的估计器（estimator）与我们之前使用 `StringIndexer` 等预处理转换器（transformer）看到的接口大致相同。这不足为奇，因为它使训练整个管道（包括模型）变得简单。出于此处的目的，我们希望一步一步地做一些事情，因此在此示例中我们选择不这样做：

```scala
// in Scala
val kmModel = kmeans.fit(transformedTraining)
```

```python
# in Python
kmModel = kmeans.fit(transformedTraining)
```

After we train this model, we can compute the cost according to some success merits on our training set. The resulting cost on this dataset is actually quite high, which is likely due to the fact that we did not properly preprocess and scale our input data, which we cover in depth in Chapter 25:

训练此模型后，我们可以根据训练集上的一些成功功绩来计算成本。该数据集上的最终成本实际上是很高的，这很可能是由于我们没有适当地预处理和缩放我们的输入数据这一事实，我们将在第25章中进行深入介绍：

```scala
kmModel.computeCost(transformedTraining)
```

```scala
// in Scala
val transformedTest = fittedPipeline.transform(testDataFrame)
```

```python
# in Python
transformedTest = fittedPipeline.transform(testDataFrame)
```

```scala
kmModel.computeCost(transformedTest)
```

Naturally, we could continue to improve this model, layering more preprocessing as well as performing hyperparameter tuning to ensure that we’re getting a good model. We leave that discussion for Part VI.

当然，我们可以继续改进此模型，对更多的预处理进行分层，并执行超参数调整，以确保获得好的模型。我们将讨论留给第六部分。

## <font color="#9a161a">Lower-Level APIs</font>

Spark includes a number of lower-level primitives to allow for arbitrary Java and Python object manipulation via Resilient Distributed Datasets (RDDs). Virtually everything in Spark is built on top of RDDs. As we will discuss in Chapter 4, DataFrame operations are built on top of RDDs and compile down to these lower-level tools for convenient and extremely efficient distributed execution. There are some things that you might use RDDs for, especially when you’re reading or manipulating raw data, but for the most part you should stick to the Structured APIs. RDDs are lower level than DataFrames because they reveal physical execution characteristics (like partitions) to end users.

Spark包含许多较低层次的原语，以允许通过弹性分布式数据集（RDD）进行任意Java和Python对象操作。实际上，Spark中的所有内容都建立在RDD之上。正如我们将在第4章中讨论的那样，DataFrame操作建立在RDD之上，并向下编译为这些较低层次的工具，以实现便捷而高效的分布式执行。在某些情况下，您可能会使用RDD，尤其是在读取或处理原始数据时，但在大多数情况下，您应该坚持使用结构化API。RDD比DataFrames低，因为它们向最终用户揭示了物理执行特征（如分区）。

One thing that you might use RDDs for is to parallelize raw data that you have stored in memory on the driver machine. For instance, let’s parallelize some simple numbers and create a DataFrame after we do so. We then can convert that to a DataFrame to use it with other DataFrames:

可能使用RDD的一件事是并行化存储在驱动程序计算机内存中的原始数据。例如，让我们并行处理一些简单的数字，然后创建一个DataFrame。然后，我们可以将其转换为DataFrame以与其他DataFrame一起使用：

```scala
// in Scala
spark.sparkContext.parallelize(Seq(1, 2, 3)).toDF()
```

```python
# in Python
from pyspark.sql import Row
spark.sparkContext.parallelize([Row(1), Row(2), Row(3)]).toDF()
```

RDDs are available in Scala as well as Python. However, they’re not equivalent. This differs from the DataFrame API (where the execution characteristics are the same) due to some underlying implementation details. We cover lower-level APIs, including RDDs in Part IV. As end users, you shouldn’t need to use RDDs much in order to perform many tasks unless you’re maintaining older Spark code. There are basically no instances in modern Spark, for which you should be using RDDs instead of the structured APIs beyond manipulating some very raw unprocessed and unstructured data.

RDD在Scala和Python中均可用。但是，它们并不相同。由于某些基础实现细节，这与DataFrame API（执行特性相同）不同。我们将介绍较低层次的API，包括第IV部分中的RDD。作为最终用户，除非您要维护较旧的Spark代码，否则无需为了执行许多任务而使用太多RDD。在现代Spark中，基本上没有实例，除了处理一些非常原始的未处理和非结构化数据外，您应该使用RDD而不是结构化API。

## <font color="#9a161a">SparkR</font>

SparkR is a tool for running R on Spark. It follows the same principles as all of Spark’s other language bindings. To use SparkR, you simply import it into your environment and run your code. It’s all very similar to the Python API except that it follows R’s syntax instead of Python. For the most part, almost everything available in Python is available in SparkR:

SparkR是用于在Spark上运行R的工具。它遵循与Spark所有其他语言绑定相同的原则。要使用SparkR，只需将其导入到环境中并运行代码。除了遵循R的语法而不是Python之外，所有其他方面都与Python API非常相似。在大多数情况下，SparkR提供了Python中几乎所有可用的功能：

```R
# in R
library(SparkR)
sparkDF <- read.df("/data/flight-data/csv/2015-summary.csv",
source = "csv", header="true", inferSchema = "true")
take(sparkDF, 5)

# in R
collect(orderBy(sparkDF, "count"), 20)
```

R users can also use other R libraries like the pipe operator in magrittr to make Spark transformations a bit more R-like. This can make it easy to use with other libraries like ggplot for more sophisticated plotting:

R用户还可以使用其他R库，例如magrittr中的管道运算符，使Spark转换更像R。这可以使它易于与ggplot等其他库一起使用，以进行更复杂的绘图：

```R
# in R
library(magrittr)
sparkDF %>%
orderBy(desc(sparkDF$count)) %>%
groupBy("ORIGIN_COUNTRY_NAME") %>%
count() %>%
limit(10) %>%
collect()
```

We will not include R code samples as we do in Python, because almost every concept throughout this book that applies to Python also applies to SparkR. The only difference will by syntax. We cover SparkR and sparklyr in Part VII.

我们不会像在Python中那样包含R代码示例，因为本书中适用于Python的几乎所有概念也都适用于SparkR。唯一的区别在于语法。我们将在第七部分介绍SparkR和sparklyr。

## <font color="#9a161a">Spark’s Ecosystem and Packages</font>

One of the best parts about Spark is the ecosystem of packages and tools that the community has created. Some of these tools even move into the core Spark project as they mature and become widely used. As of this writing, the list of packages is rather long, numbering over 300—and more are added frequently. You can find the largest index of Spark Packages at <u><a style="color:#0879e3" href="https://spark-packages.org/">spark-packages.org</a></u>, where any user can publish to this package repository. There are also various other projects and packages that you can find on the web; for example, on GitHub.

关于Spark的最好的部分之一是社区创建的软件包和工具的生态系统。随着这些工具的成熟和广泛使用，其中一些工具甚至进入了Spark核心项目。在撰写本文时，软件包列表相当长，超过300个，并且经常添加更多。您可以在 <u><a style="color:#0879e3" href="https://spark-packages.org/">spark-packages.org</a></u> 上找到Spark Packages的最新索引，任何用户都可以在其中将其发布到此软件包存储库。您还可以在网上找到其他各种项目和软件包。例如，在GitHub上。

## <font color="#9a161a">Conclusion</font>

We hope this chapter showed you the sheer variety of ways in which you can apply Spark to your own business and technical challenges. Spark’s simple, robust programming model makes it easy to apply to a large number of problems, and the vast array of packages that have crept up around it, created by hundreds of different people, are a true testament to Spark’s ability to robustly tackle a number of business problems and challenges. As the ecosystem and community grows, it’s likely that more and more packages will continue to crop up. We look forward to seeing what the community has in store! 

我们希望本章向您展示将Spark应用于自己的业务和技术挑战的多种方法。Spark的简单，健壮的编程模型使您可以轻松地将其应用于大量问题，并且由数百个不同的人创建的大量围绕它的软件包，这些都是Spark强大解决大量问题的能力的真实证明。业务问题和挑战。随着生态系统和社区的发展，越来越多的软件包可能会继续出现。

The rest of this book will provide deeper dives into the product areas in Figure 3-1.

我们期待看到社区中拥有的一切！本书的其余部分将更深入地研究图3-1中的产品区域。  

You may read the rest of the book any way that you prefer, we find that most people hop from area to area as they hear terminology or want to apply Spark to certain problems they’re facing.

您可以按照自己喜欢的任何方式阅读本书的其余部分，我们发现大多数人在听到术语或希望将Spark应用到他们所面临的某些问题时会跳来跳去。