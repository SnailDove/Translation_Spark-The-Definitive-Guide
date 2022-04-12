---
title: 翻译 Chapter 12 Resilient Distributed Datasets (RDDs)
date: 2019-11-07
copyright: true
categories: English
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 12. Resilient Distributed Datasets (RDDs)

The previous part of the book covered Spark’s Structured APIs. You should heavily favor these APIs in almost all scenarios. That being said, there are times when higher-level manipulation will not meet the business or engineering problem you are trying to solve. For those cases, you might need to use Spark’s lower-level APIs, specifically the Resilient Distributed Dataset (RDD), the `SparkContext`, and distributed shared variables like accumulators and broadcast variables. The chapters that follow in this part cover these APIs and how to use them.

本书的上一部分介绍了Spark的结构化API。在几乎所有情况下，您都应该大力支持这些API。话虽这么说，有时更高层别的操作无法满足您要解决的业务或工程问题。在这种情况下，您可能需要使用Spark的较底层API，特别是弹性分布式数据集（RDD），`SparkContext`和分布式共享变量（distributed shared variable），例如累加器（accumulator）和广播变量（broadcast variable）。本部分后面的章节介绍了这些API以及如何使用它们。

---

<p><center><font face="constant-width" color="#c67171" size=3><strong>WARNING 警告</strong></font></center></p>
If you are brand new to Spark, this is not the place to start. Start with the Structured APIs, you'll be more productive more quickly!

如果您是Spark的新手，那么这不是一个开始的地方。 从结构化API开始，您将更快地提高生产力！

---

## <font color="#9a161a">What Are the Low-Level APIs?</font>

There are two sets of low-level APIs: there is one for manipulating distributed data (RDDs), and another for distributing and manipulating distributed shared variables (broadcast variables and accumulators).

有两组底层API：一组用于处理分布式数据（RDD），另一组用于分发和处理分布式共享变量（广播变量和累加器）。

### <font color="#00000">When to Use the Low-Level APIs? </font>

You should generally use the lower-level APIs in three situations:

通常，您应在以下三种情况下使用较底层的API：

-  You need some functionality that you cannot find in the higher-level APIs; for example, if you need very tight control over physical data placement across the cluster.

    您需要一些在高层API中找不到的功能。 例如，如果您需要非常严格地控制整个集群中的物理数据放置。

- You need to maintain some legacy codebase written using RDDs.

    您需要维护一些使用RDD编写的旧代码库。

- You need to do some custom shared variable manipulation. We will discuss shared variables more in Chapter 14.

    您需要执行一些自定义共享变量操作。我们将在第14章中讨论共享变量。

Those are the reasons why you should use these lower-level tools, buts it’s still helpful to understand these tools because all Spark workloads compile down to these fundamental primitives. When you’re calling a DataFrame transformation, it actually just becomes a set of RDD transformations. This understanding can make your task easier as you begin debugging more and more complex workloads.

这就是为什么您应该使用这些底层工具的原因，但是了解这些工具仍然有帮助，因为所有Spark工作负载均会编译为这些基本原语（fundamental primitives）。当您调用DataFrame转换时，它实际上只是一组RDD转换。当您开始调试越来越复杂的工作负载时，这种理解可以使您的任务更轻松。

Even if you are an advanced developer hoping to get the most out of Spark, we still recommend focusing on the Structured APIs. However, there are times when you might want to “drop down” to some of the lower-level tools to complete your task. You might need to drop down to these APIs to use some legacy code, implement some custom partitioner, or update and track the value of a variable over the course of a data pipeline’s execution. These tools give you more fine-grained control at the expense of safeguarding you from shooting yourself in the foot.

即使您是希望充分利用Spark的高级开发人员，我们仍然建议您专注于结构化API。但是，有时您可能需要“下拉”一些较底层的工具来完成任务。您可能需要使用这些API来使用一些旧代码，实现一些自定义分区程序，或者在数据管道执行过程中更新和跟踪变量的值。这些工具可为您提供更细粒度的控制，但以保护您伤害到自己为代价。

### <font color="#00000">How to Use the Low-Level APIs?</font>

A `SparkContext` is the entry point for low-level API functionality. You access it through the `SparkSession`, which is the tool you use to perform computation across a Spark cluster. We discuss this further in Chapter 15 but for now, you simply need to know that you can access a `SparkContext` via the following call:

`SparkContext`是底层API功能的入口点。您可以通过 `SparkSession` 访问它，`SparkSession` 是用于跨 Spark 集群执行计算的工具。我们将在第15章中对此进行进一步讨论，但是现在，您只需要知道可以通过以下调用访问`SparkContext` ：

```scala
spark.sparkContext
```

## <font color="#9a161a">About RDDs</font>

RDDs were the primary API in the Spark 1.X series and are still available in 2.X, but they are not as commonly used. However, as we’ve pointed out earlier in this book, virtually all Spark code you run, whether DataFrames or Datasets, compiles down to an RDD. The Spark UI, covered in the next part of the book, also describes job execution in terms of RDDs. Therefore, it will behoove you to have at least a basic understanding of what an RDD is and how to use it.

RDD是Spark 1.X系列中的主要API，并且在2.X中仍然可用，但是并不常用。但是，正如我们在本书前面所指出的那样，您运行的几乎所有Spark代码（无论是DataFrames还是Datasets）都可以编译为RDD。本书下一部分介绍的Spark UI还以RDD来描述作业执行。因此，您应该至少对RDD是什么以及如何使用它有基本的了解。

In short, an RDD represents an immutable, partitioned collection of records that can be operated on in parallel. Unlike DataFrames though, where each record is a structured row containing fields with a known schema, in RDDs the records are just Java, Scala, or Python objects of the programmer’s choosing.

简而言之，RDD表示一个不变的，分区的记录集合，可以并行操作。不过，与DataFrames不同的是，每个记录都是一个结构化的行，其中包含具有已知模式的字段，而在RDD中，记录只是程序员选择的Java，Scala或Python对象。

RDDs give you complete control because every record in an RDD is a just a Java or Python object. You can store anything you want in these objects, in any format you want. This gives you great power, but not without potential issues. Every manipulation and interaction between values must be defined by hand, meaning that you must “reinvent the wheel” for whatever task you are trying to carry out. Also, optimizations are going to require much more manual work, because Spark does not understand the inner structure of your records as it does with the Structured APIs. For instance, Spark’s Structured APIs automatically store data in an optimized, compressed binary format, so to achieve the same space-efficiency and performance, you’d also need to implement this type of format inside your objects and all the low-level operations to compute over it. Likewise, optimizations like reordering filters and aggregations that occur automatically in Spark SQL need to be implemented by hand. For this reason and others, we highly recommend using the Spark Structured APIs when possible.

RDD提供了完全的控制权，因为RDD中的每条记录都只是一个Java或Python对象。您可以以任何所需的格式将所需的任何内容存储在这些对象中。这将为您提供强大的功能，但并非没有潜在的问题。值之间的每个操作和交互都必须手动定义，这意味着您必须“重新发明轮子”才能完成您要执行的任何任务。而且，优化将需要更多的人工工作，因为Spark无法像使用结构化API那样理解记录的内部结构。例如，Spark的结构化API自动以优化的压缩二进制格式存储数据，因此，要实现相同的空间效率和性能，还需要在对象内部以及所有底层操作中实现这种格式计算它。同样，需要手动执行在Spark SQL中自动进行的优化（例如重新排序过滤器和聚合）。因此，我们强烈建议您尽可能使用Spark结构化API。

The RDD API is similar to the Dataset, which we saw in the previous part of the book, except that RDDs are not stored in, or manipulated with, the structured data engine. However, it is trivial to convert back and forth between RDDs and Datasets, so you can use both APIs to take advantage of each API’s strengths and weaknesses. We’ll show how to do this throughout this part of the book.

RDD API 与 Dataset 类似，我们在本书的上半部分中看到了，除了 RDD 不存储在结构化数据引擎中或不使用结构化数据引擎操纵之外。但是，在 RDD 和 Datasets 之间来回转换很简单，因此您可以使用这两个API来利用每个API的优点和缺点。在本书的这一部分中，我们将展示如何执行此操作。

### <font color="#00000">Types of RDDs</font>

If you look through Spark’s API documentation, you will notice that there are lots of subclasses of RDD. For the most part, these are internal representations that the DataFrame API uses to create optimized physical execution plans. As a user, however, you will likely only be creating two types of RDDs: the “generic” RDD type or a key-value RDD that provides additional functions, such as aggregating by key. For your purposes, these will be the only two types of RDDs that matter. Both just represent a collection of objects, but key-value RDDs have special operations as well as a concept of custom partitioning by key.

查看Spark的API文档时，您会发现RDD有很多子类。在大多数情况下，这些是DataFrame API用于创建优化的物理执行计划的内部表示。但是，作为用户，您可能只会创建两种类型的RDD：**“通用” RDD类型**或提供附加功能（例如，按键聚合）的**键值RDD**。就您的目的而言，这将是仅有的两种重要的RDD类型。两者都仅表示对象的集合，但是键值RDD具有特殊的操作以及按键自定义分区的概念。

Let’s formally define RDDs. Internally, each RDD is characterized by five main properties :

让我们正式定义RDD。在内部，每个RDD具有五个主要属性：

1. A list of partitions 分区列表
2. A function for computing each split  用于计算每个拆分的函数
3. A list of dependencies on other RDDs  对其他RDD的依赖关系列表
4. Optionally, a `Partitioner` for key-value RDDs (e.g., to say that the RDD is hash-partitioned) （可选）一个键值RDD的分区程序（例如，说RDD是哈希分区的）
5. Optionally, a list of preferred locations on which to compute each split (e.g., block locations for a Hadoop Distributed File System [HDFS] file)  （可选）在其上计算每个拆分的首选位置的列表（例如，Hadoop分布式文件系统[HDFS]文件的块位置）

---

<p><center><font face="constant-width" color="#737373" size=3><strong>NOTE 注意</strong></font></center></P>
The `Partitioner` is probably one of the core reasons why you might want to use RDDs in your code. Specifying your own custom `Partitioner` can give you significant performance and stability improvements if you use it correctly. This is discussed in more depth in Chapter 13 when we introduce Key–Value Pair RDDs.

分区程序可能是您可能想在代码中使用RDD的核心原因之一。如果正确使用自定义分区程序，则可以显著提高性能和稳定性。当我们介绍键值对RDD时，将在第13章中对此进行更深入的讨论。

----

These properties determine all of Spark’s ability to schedule and execute the user program. Different kinds of RDDs implement their own versions of each of the aforementioned properties, allowing you to define new data sources.

这些属性决定了Spark安排和执行用户程序的全部能力。不同种类的RDD会为每个上述属性实现各自的版本，从而允许您定义新的数据源。

RDDs follow the exact same Spark programming paradigms that we saw in earlier chapters. They provide transformations, which evaluate lazily, and actions, which evaluate eagerly, to manipulate data in a distributed fashion. These work the same way as transformations and actions on DataFrames and Datasets. However, there is no concept of “rows” in RDDs; individual records are just raw Java/Scala/Python objects, and you manipulate those manually instead of tapping into the repository of functions that you have in the structured APIs.

RDD遵循我们在前几章中看到的完全相同的Spark编程范例。它们提供了延迟求值（evaluate lazily）的转换和迫切求值（evaluate eagerly）的动作，以分布式方式处理数据。这些工作方式与对DataFrame和Dataset进行转换和操作相同。但是，RDD中没有“行”的概念；单个记录只是原始的 Java/Scala/Python 对象，您可以手动操作它们，而不必进入结构化API中具有的函数存储库。

The RDD APIs are available in Python as well as Scala and Java. For Scala and Java, the performance is for the most part the same, the large costs incurred in manipulating the raw objects. Python, however, can lose a substantial amount of performance when using RDDs. Running Python RDDs equates to running Python user-defined functions (UDFs) row by row. Just as we saw inChapter 6. We serialize the data to the Python process, operate on it in Python, and then serialize it back to the Java Virtual Machine (JVM). This causes a high overhead for Python RDD manipulations. Even though many people ran production code with them in the past, we recommend building on the Structured APIs in Python and only dropping down to RDDs if absolutely necessary.

RDD API在Python以及Scala和Java中均可用。对于Scala和Java，性能在很大程度上是相同的，这是操作原始对象所产生的巨大成本。但是，使用RDD时，Python可能会损失大量性能。运行Python RDD等同于逐行运行Python用户定义函数（UDF）。就像在第6章中看到的那样。我们将数据序列化到Python进程，在Python中对其进行操作，然后将其序列化回Java虚拟机（JVM）。这会导致Python RDD操作的开销很大。即使过去有很多人使用生产代码来运行它们，我们还是建议在Python中基于结构化API进行构建，并且仅在绝对必要时才使用RDD。

### <font color="#00000">When to Use RDDs?</font>

In general, you should not manually create RDDs unless you have a very, very specific reason for doing so. They are a much lower-level API that provides a lot of power but also lacks a lot of the optimizations that are available in the Structured APIs. For the vast majority of use cases, DataFrames will be more efficient, more stable, and more expressive than RDDs.

通常，除非有非常特殊的原因，否则不应手动创建RDD。它们是一个底层的API，它提供了很多功能，但缺乏结构化API中可用的许多优化。在绝大多数用例中，DataFrames将比RDDs更高效，更稳定和更具表现力。 

The most likely reason for why you’ll want to use RDDs is because you need fine-grained control over the physical distribution of data (custom partitioning of data).

之所以要使用RDD，最可能的原因是因为您需要对数据的物理分布（数据的自定义分区）进行细粒度的控制。

### <font color="#00000">Datasets and RDDs of Case Classes</font>

We noticed this question on the web and found it to be an interesting one: what is the difference between RDDs of Case Classes and Datasets? The difference is that Datasets can still take advantage of the wealth of functions and optimizations that the Structured APIs have to offer. With Datasets, you do not need to choose between only operating on JVM types or on Spark types, you can choose whatever is either easiest to do or most flexible. You get the both of best worlds.

我们在网上注意到了这个问题，发现这是一个有趣的问题：案例类和 Datasets 的RDD有什么区别？区别在于，Datasets 仍然可以利用结构化API必须提供的丰富功能和优化。使用 Datasets ，您不需要是仅在JVM类型上的操作或是仅在Spark类型上的操作进行选择，可以选择最容易执行或最灵活的操作。你们两全其美。

## <font color="#9a161a">Creating RDDs</font>

Now that we discussed some key RDD properties, let’s begin applying them so that you can better understand how to use them.

现在，我们讨论了一些RDD关键属性，让我们开始应用它们，以便您可以更好地了解如何使用它们。

### <font color="#00000">Interoperating Between DataFrames, Datasets, and RDDs</font>

One of the easiest ways to get RDDs is from an existing DataFrame or Dataset. Converting these to an RDD is simple: just use the `rdd` method on any of these data types. You’ll notice that if you do a conversion from a Dataset[T] to an RDD, you’ll get the appropriate native type T back (remember this applies only to Scala and Java):

获取RDD的最简单方法之一是从现有的DataFrame或Dataset中获取。将它们转换为RDD很简单：只需对任何这些数据类型使用 `rdd` 方法。您会注意到，如果您进行了从 Dataset[T] 到 RDD 的转换，则会获得适当的本地类型T（请记住，这仅适用于 Scala 和 Java）：

```scala
// in Scala: converts a Dataset[Long] to RDD[Long]
spark.range(500).rdd
```

Because Python doesn’t have Datasets—it has only DataFrames—you will get an RDD of type Row:

由于 Python 没有 Datasets——它只有DataFrames——您将获得Row类型的RDD：

```scala
# in Python
spark.range(10).rdd
```

To operate on this data, you will need to convert this Row object to the correct data type or extract values out of it, as shown in the example that follows. This is now an RDD of type Row:

要对该数据进行操作，您将需要将该Row对象转换为正确的数据类型或从中提取值，如以下示例所示。现在是Row类型的RDD：

```scala
// in Scala
spark.range(10).toDF().rdd.map(rowObject => rowObject.getLong(0))
```

```python
# in Python
spark.range(10).toDF("id").rdd.map(lambda row: row[0])
```

You can use the same methodology to create a DataFrame or Dataset from an RDD. All you need to do is call the `toDF` method on the RDD:

您可以使用相同的方法从RDD创建DataFrame或Dataset。您需要做的就是在RDD上调用`toDF`方法：

```scala
// in Scala
spark.range(10).rdd.toDF()
```

```python
# in Python
spark.range(10).rdd.toDF()
```

This command creates an RDD of type Row. This row is the internal Catalyst format that Spark uses to represent data in the Structured APIs. This functionality makes it possible for you to jump between the Structured and low-level APIs as it suits your use case. (We talk about this in Chapter 13.)

该命令创建Row类型的RDD。此行是Spark用来表示Structured API中的数据的内部Catalyst格式。此功能使您可以在适合您的用例的情况下在结构化API和底层API之间进行切换。（我们将在第13章中讨论这一点。）

The RDD API will feel quite similar to the Dataset API in Chapter 11 because they are extremely similar to each other (RDDs being a lower-level representation of Datasets) that do not have a lot of the convenient functionality and interfaces that the Structured APIs do.

RDD API与第11章中的Dataset API非常相似，因为它们彼此非常相似（RDD 是 Datasets 的底层表示），并且没有结构化API所具有的许多便利功能和接口。

### <font color="#00000">From a Local Collection</font>

To create an RDD from a collection, you will need to use the `parallelize` method on a `SparkContext` (within a `SparkSession`). This turns a single node collection into a parallel collection.

要从集合创建RDD，您将需要在 `SparkContext` 上（在 `SparkSession` 中）使用 `parallelize` 方法。这会将单个节点集合变成并行集合。

When creating this parallel collection, you can also explicitly state the number of partitions into which you would like to distribute this array. In this case, we are creating two partitions:

创建此并行集合时，您还可以明确声明要将此数组分发到的分区数。在这种情况下，我们将创建两个分区：  

```scala
// in Scala
val myCollection = "Spark The Definitive Guide : Big Data Processing Made Simple".split(" ")
val words = spark.sparkContext.parallelize(myCollection, 2)
```

```python
# in Python
myCollection = "Spark The Definitive Guide : Big Data Processing Made Simple"\
.split(" ")
words = spark.sparkContext.parallelize(myCollection, 2)
```

An additional feature is that you can then name this RDD to show up in the Spark UI according to a given name:

另一个功能是，您可以根据给定的名称将该RDD命名为显示在Spark UI中：

```scala
// in Scala
words.setName("myWords")
words.name // myWords
```

```python
# in Python
words.setName("myWords")
words.name() # myWords
```

### <font color="#00000">From Data Sources</font>

Although you can create RDDs from data sources or text files, it’s often preferable to use the Data Source APIs. RDDs do not have a notion of “Data Source APIs” like DataFrames do; they primarily define their dependency structures and lists of partitions. The Data Source API that we saw in Chapter 9 is almost always a better way to read in data. That being said, you can also read data as RDDs using  `sparkContext`. For example, let’s read a text file line by line:

尽管您可以从数据源或文本文件创建RDD，但通常最好使用数据源API。RDD不像DataFrames那样具有“数据源API”的概念。它们主要定义其依赖关系结构和分区列表。我们在第9章中看到的数据源API几乎总是一种读取数据的更好方法。话虽如此，您也可以使用 `sparkContext` 将数据读取为RDD。例如，让我们逐行阅读一个文本文件：

 ```scala
spark.sparkContext.textFile("/some/path/withTextFiles")
 ```

This creates an RDD for which each record in the RDD represents a line in that text file or files. Alternatively, you can read in data for which each text file should become a single record. The use case here would be where each file is a file that consists of a large JSON object or some document that you will operate on as an individual:

这将创建一个RDD，RDD中的每个记录都代表该文本文件中的一行。或者，您可以读取每个文本文件应成为单个记录的数据。这里的用例是，每个文件都是一个由大型JSON对象或您将单独处理的文档组成的文件：

```scala
spark.sparkContext.wholeTextFiles("/some/path/withTextFiles")
```

In this RDD, the name of the file is the first object and the value of the text file is the second string object.

在此RDD中，文件名是第一个对象，文本文件的值是第二个字符串对象。

## <font color="#9a161a">Manipulating RDDs</font>

You manipulate RDDs in much the same way that you manipulate DataFrames. As mentioned, the core difference being that you manipulate raw Java or Scala objects instead of Spark types. There is also a dearth of “helper” methods or functions that you can draw upon to simplify calculations. Rather, you must define each filter, map functions, aggregation, and any other manipulation that you want as a function.

处理RDD的方式与处理DataFrames的方式几乎相同。如前所述，核心区别在于您可以操纵原始Java或Scala对象而不是Spark类型。缺少用于简化计算的“辅助”方法或函数，您必须定义每个过滤器，映射函数，聚合以及要作为函数进行的任何其他操作。

To demonstrate some data manipulation, let’s use the simple RDD (words) we created previously to define some more details.

为了演示一些数据操作，让我们使用之前创建的简单RDD（单词）来定义更多细节。

## <font color="#9a161a">Transformations</font>

For the most part, many transformations mirror the functionality that you find in the Structured APIs. Just as you do with DataFrames and Datasets, you specify transformations on one RDD to create another. In doing so, we define an RDD as a dependency to another along with some manipulation of the data contained in that RDD.

在大多数情况下，许多转换都反映了您在结构化API中找到的功能。就像使用DataFrames和Datasets一样，您可以在一个RDD上指定转换以创建另一个。为此，我们将RDD定义为对另一个的依赖，并对该RDD中包含的数据进行一些操作。

### <font color="#00000">distinct</font>

A distinct method call on an RDD removes duplicates from the RDD:

在RDD上进行不同的方法调用可从RDD中删除重复项：

```scala
words.distinct().count()
```

This gives a result of 10.

结果为10。

### <font color="#00000">filter</font>

Filtering is equivalent to creating a SQL-like where clause. You can look through our records in the RDD and see which ones match some predicate function. This function just needs to return a Boolean type to be used as a filter function. The input should be whatever your given row is. In this next example, we filter the RDD to keep only the words that begin with the letter “S”:

过滤等效于创建类似SQL的where子句。您可以在RDD中浏览我们的记录，看看哪些与某些谓词函数匹配。该函数只需要返回一个布尔类型即可用作过滤器函数。输入应为您给定的行。在下一个示例中，我们对RDD进行过滤，以仅保留以字母“ S”开头的单词：

```scala
// in Scala
def startsWithS(individual:String) = {
	individual.startsWith("S")
} 
```

```python
# in Python
def startsWithS(individual):
return individual.startswith("S")
```

Now that we defined the function, let’s filter the data. This should feel quite familiar if you read Chapter 11 because we simply use a function that operates record by record in the RDD. The function is defined to work on each record in the RDD individually:

现在我们定义了函数，让我们过滤数据。如果您阅读第11章，应该会感到非常熟悉，因为我们仅使用了一个函数来操作RDD中的记录。该函数被定义为分别在RDD中的每个记录上工作：

```scala
// in Scala
words.filter(word => startsWithS(word)).collect()
```

```python
# in Python
words.filter(lambda word: startsWithS(word)).collect()
```

This gives a result of Spark and Simple. We can see, like the Dataset API, that this returns native types. That is because we never coerce our data into type Row, nor do we need to convert the data after collecting it.

这给出了Spark和Simple的结果。我们可以看到，就像Dataset API一样，这将返回本地类型。那是因为我们从不将数据强制转换为Row类型，也不需要在收集数据后转换数据。

### <font color="#00000">map</font>

Mapping is again the same operation that you can read about in Chapter 11. You specify a function that returns the value that you want, given the correct input. You then apply that, record by record. Let’s perform something similar to what we just did. In this example, we’ll map the current word to the word, its starting letter, and whether the word begins with “S.”

映射同样是您在第11章中可以了解的相同操作。给定正确的输入，您可以指定一个函数，该函数返回所需的值。然后，您将其应用，逐条记录。让我们执行与我们刚做的类似的事情。在此示例中，我们将当前单词映射到该单词，其起始字母以及该单词是否以 “S” 开头。

Notice in this instance that we define our functions completely inline using the relevant lambda syntax:

注意在这种情况下，我们使用相关的lambda语法完全内联定义了我们的函数：

```scala
// in Scala
val words2 = words.map(word => (word, word(0), word.startsWith("S")))
```

```python
# in Python
words2 = words.map(lambda word: (word, word[0], word.startswith("S")))
```

 You can subsequently filter on this by selecting the relevant Boolean value in a new function:

随后，您可以通过在新函数中选择相关的布尔值来对此进行过滤：

```scala
// in Scala
words2.filter(record => record._3).take(5)
```

```python
# in Python
words2.filter(lambda record: record[2]).take(5)
```

This returns a tuple of “Spark,” “S,” and “true,” as well as “Simple,” “S,” and “True.”

这将返回“ Spark”，“ S”和“ true”以及“ Simple”，“ S”和“ True”的元组。

#### <font color="#3399cc">flatMap</font>

flatMap provides a simple extension of the map function we just looked at. Sometimes, each current row should return multiple rows, instead. For example, you might want to take your set of words and flatMap it into a set of characters. Because each word has multiple characters, you should use flatMap to expand it. flatMap requires that the ouput of the map function be an iterable that can be expanded:

`flatMap`提供了我们刚刚看过的map函数的简单扩展。有时，每个当前行应该返回多个行。例如，您可能想将一组单词和 `flatMap` 转换成一组字符。由于每个单词都有多个字符，因此应使用 `flatMap` 对其进行扩展。`flatMap` 要求map函数的输出是可迭代的，可以扩展：

```scala
// in Scala
words.flatMap(word => word.toSeq).take(5)
```

```python
# in Python
words.flatMap(lambda word: list(word)).take(5)
```

This yields S, P, A, R, K.

这产生S，P，A，R，K。

### <font color="#00000">sort</font>

To sort an RDD you must use the `sortBy` method, and just like any other RDD operation, you do this by specifying a function to extract a value from the objects in your RDDs and then sort based on that. For instance, the following example sorts by word length from longest to shortest:

要对RDD进行排序，必须使用 `sortBy` 方法，就像其他任何RDD操作一样，您可以通过指定一个函数来从RDD中的对象中提取值，然后基于该函数进行排序。例如，以下示例按单词长度从最长到最短排序：

```scala
// in Scala
words.sortBy(word => word.length() * -1).take(2)
```
```python
# in Python
words.sortBy(lambda word: len(word) * -1).take(2)
```

### <font color="#00000">Random Splits</font>

We can also randomly split an RDD into an Array of RDDs by using the `randomSplit` method, which accepts an Array of weights and a random seed:

我们还可以使用 `randomSplit` 方法将RDD随机分为RDD数组，该方法接受权重数组和随机种子：

```scala
// in Scala
val fiftyFiftySplit = words.randomSplit(Array[Double](0.5, 0.5))
```
```python
# in Python
fiftyFiftySplit = words.randomSplit([0.5, 0.5])
```

This returns an array of RDDs that you can manipulate individually.

这将返回可以单独操作的RDD数组。

## <font color="#9a161a">Actions</font>

Just as we do with DataFrames and Datasets, we specify actions to kick off our specified transformations. Actions either collect data to the driver or write to an external data source.

就像处理DataFrames和Datasets一样，我们指定action（动作/算子）来启动我们指定的转换。action（动作/算子）要么将数据收集到驱动程序，要么写入外部数据源。

### <font color="#000000">reduce</font>  

You can use the reduce method to specify a function to “reduce” an RDD of any kind of value to one value. For instance, given a set of numbers, you can reduce this to its sum by specifying a function that takes as input two values and reduces them into one. If you have experience in functional programming, this should not be a new concept:

您可以使用reduce方法来指定一个函数，以将任何类型的RDD“reduce”为一个值。例如，给定一组数字，您可以通过指定一个将两个值作为输入并减小为一个的函数来将其减少为总和。如果您具有函数式编程的经验，那么这不是一个新概念：

```scala
// in Scala
spark.sparkContext.parallelize(1 to 20).reduce(_ + _) // 210
```

```python
# in Python
spark.sparkContext.parallelize(range(1, 21)).reduce(lambda x, y: x + y) # 210
```

You can also use this to get something like the longest word in our set of words that we defined a moment ago. The key is just to define the correct function:

您也可以使用它来获得类似我们刚才定义的单词集中最长的单词。关键只是定义正确的功能：

```scala
// in Scala
def wordLengthReducer(leftWord:String, rightWord:String): String = {
    if (leftWord.length > rightWord.length)
        return leftWord
    else
		return rightWord
} 

words.reduce(wordLengthReducer)
```

```python
# in Python
def wordLengthReducer(leftWord, rightWord):
	if len(leftWord) > len(rightWord):
		return leftWord
	else:
		return rightWord

words.reduce(wordLengthReducer)
```

This reducer is a good example because you can get one of two outputs. Because the reduce operation on the partitions is not deterministic, you can have either “definitive” or “processing” (both of length 10) as the “left” word. This means that sometimes you can end up with one, whereas other times you end up with the other.

这个reducer是一个很好的例子，因为您可以获得两个输出之一。由于对分区的reduce操作不是确定性的，因此可以将“definitive”或“processing”（长度均为10）作为“左”字。这意味着有时候您可以以一个结局，而其他时候则以另一个结局。

### <font color="#00000">count</font>

This method is fairly self-explanatory. Using it, you could, for example, count the number of rows in the RDD:

这种方法是不言自明的。使用它，例如，您可以计算RDD中的行数：

```scala
words.count()
```

 #### <font color="#3399cc">countApprox</font>

Even though the return signature for this type is a bit strange, it’s quite sophisticated. This is an approximation of the count method we just looked at, but it must execute within a timeout (and can return incomplete results if it exceeds the timeout).

即使此类型的返回签名有些奇怪，也相当复杂。这是我们刚刚看过的count方法的近似值，但是它必须在超时内执行（如果超过超时，则可能返回不完整的结果）。

 The confidence is the probability that the error bounds of the result will contain the true value. That is, if `countApprox` were called repeatedly with confidence 0.9, we would expect 90% of the results to contain the true count. The confidence must be in the range [0,1], or an exception will be thrown:

置信度是结果的误差范围包含真实值的概率。也就是说，如果以0.9的置信度重复调用 `countApprox`，则我们期望90％的结果包含真实计数。置信度必须在[0,1]范围内，否则将引发异常：

```scala
val confidence = 0.95
val timeoutMilliseconds = 400
words.countApprox(timeoutMilliseconds, confidence)
```

#### <font color="#3399cc">countApproxDistinct</font>

There are two implementations of this, both based on streamlib’s implementation of “HyperLogLog in Practice: Algorithmic Engineering of a State-of-the-Art Cardinality Estimation Algorithm.” 

此方法有两种实现，均基于 streamlib 的“HyperLogLog in
Practice: Algorithmic Engineering of a State-of-the-Art Cardinality Estimation Algorithm” 的实现。

In the first implementation, the argument we pass into the function is the relative accuracy. Smaller values create counters that require more space. The value must be greater than 0.000017:

在第一种实现中，我们传递给函数的参数是相对精度。较小的值会创建需要更多空间的计数器。该值必须大于0.000017： 

```scala
words.countApproxDistinct(0.05)
```

With the other implementation you have a bit more control; you specify the relative accuracy based on two parameters: one for “regular” data and another for a sparse representation.

使用其他实现，您可以控制更多。您可以根据两个参数指定相对精度：一个用于“常规”数据，另一个用于稀疏表示。

The two arguments are p and sp where p is precision and sp is sparse precision. The relative accuracy is approximately 1.054 / sqrt(2 ). Setting a nonzero (sp > p) can reduce the memory consumption and increase accuracy when the cardinality is small. Both values are integers:

两个参数是p和sp，其中p是精度，而sp是稀疏精度。相对精度约为 1.054/sqrt(2) 。当基数较小时，将非零值设置为 (sp> p)可以减少内存消耗并提高准确性。这两个值都是整数：

```scala
words.countApproxDistinct(4, 10)
```

#### <font color="#3399cc">countByValue</font>

This method counts the number of values in a given RDD. However, it does so by finally loading the result set into the memory of the driver. You should use this method only if the resulting map is expected to be small because the entire thing is loaded into the driver’s memory. Thus, this method makes sense only in a scenario in which either the total number of rows is low or the number of distinct items is low:

此方法计算给定RDD中值的数量。但是，它是通过将结果集最终加载到驱动程序的内存中来实现的。仅在预期生成的 map较小的情况下才应使用此方法，因为整个 map 都已加载到驱动程序的内存中。因此，此方法仅在行总数少或不同项目数少的情况下才有意义：

```scala
words.countByValue()
```

#### <font color="#3399cc">countByValueApprox</font>

This does the same thing as the previous function, but it does so as an approximation. This must execute within the specified timeout (first parameter) (and can return incomplete results if it exceeds the timeout).

该功能与先前的功能相同，但仅作为近似值。此操作必须在指定的超时（第一个参数）内执行（如果超过超时，则可能返回不完整的结果）。

The confidence is the probability that the error bounds of the result will contain the true value. That is, if countApprox were called repeatedly with confidence 0.9, we would expect 90% of the results to contain the true count. The confidence must be in the range [0,1], or an exception will be thrown:

置信度是结果的误差范围包含真实值的概率。也就是说，如果以0.9的置信度重复调用countApprox，则我们期望90％的结果包含真实计数。置信度必须在[0,1]范围内，否则将引发异常：

```scala
words.countByValueApprox(1000, 0.95)
```

### <font color="#00000">first</font>

The first method returns the first value in the dataset:

第一个方法返回数据集中的第一个值：

```scala
words.first()
```

### <font color="#00000">max and min</font>

max and min return the maximum values and minimum values, respectively:

max和min分别返回最大值和最小值：

```scala
spark.sparkContext.parallelize(1 to 20).max()
spark.sparkContext.parallelize(1 to 20).min()
```

### <font color="#00000">take</font>

`take` and its derivative methods take a number of values from your RDD. This works by first scanning one partition and then using the results from that partition to estimate the number of additional partitions needed to satisfy the limit.

`take`及其派生方法从RDD中获取许多值。该方法是这样工作的：通过首先扫描一个分区，然后使用该分区的结果来估计满足该限制（“限制”指的是方法参数指定的值）所需的其他分区的数量。

There are many variations on this function, such as `takeOrdered`, `takeSample`, and top. You can use `takeSample` to specify a fixed-size random sample from your RDD. You can specify whether this should be done by using `withReplacement`, the number of values, as well as the random seed. top is effectively the opposite of `takeOrdered` in that it selects the top values according to the implicit ordering:

此函数有很多变体，例如`takeOrdered`，`takeSample`和`top`。您可以使用`takeSample`从RDD中指定一个固定大小的随机样本。您可以使用`withReplacement`，值的数量以及随机种子来指定是否应该这样做。`top` 实际上与`takeOrdered`相反，它根据隐式顺序选择顶部值：

```scala
words.take(5)
words.takeOrdered(5)
words.top(5)

val withReplacement = true
val numberToTake = 6
val randomSeed = 100L

words.takeSample(withReplacement, numberToTake, randomSeed)
```

## <font color="#9a161a">Saving Files</font>

Saving files means writing to plain-text files. With RDDs, you cannot actually “save” to a data source in the conventional sense. You must iterate over the partitions in order to save the contents of each partition to some external database. This is a low-level approach that reveals the underlying operation that is being performed in the higher-level APIs. Spark will take each partition, and write that out to the destination.

保存文件意味着写入纯文本文件。使用RDD，您实际上无法按照传统意义上的“保存”到数据源。您必须遍历分区才能将每个分区的内容保存到某个外部数据库。这是一种低层方法，它揭示了高层API中正在执行的基础操作。Spark将获取每个分区，并将其写出到目标位置。

### <font color="#00000">saveAsTextFile</font>

To save to a text file, you just specify a path and optionally a compression codec:

要保存到文本文件，只需指定路径和压缩编解码器即可：

```scala
words.saveAsTextFile("file:/tmp/bookTitle")
```

To set a compression codec, we must import the proper codec from Hadoop. You can find these in the `org.apache.hadoop.io.compress` library:

要设置压缩编解码器，我们必须从Hadoop导入正确的编解码器。您可以在 `org.apache.hadoop.io.compress` 库中找到这些：

```scala
// in Scala
import org.apache.hadoop.io.compress.BZip2Codec
words.saveAsTextFile("file:/tmp/bookTitleCompressed", classOf[BZip2Codec])
```

### <font color="#00000">SequenceFiles</font>

Spark originally grew out of the Hadoop ecosystem, so it has a fairly tight integration with a variety of Hadoop tools. A `sequenceFile` is a flat file consisting of binary key–value pairs. It is extensively used in `MapReduce` as input/output formats.

Spark最初起源于Hadoop生态系统，因此与各种Hadoop工具紧密集成。`sequenceFile` 是一个扁平结构的文件（flat file），由二进制键值对组成。它在MapReduce中广泛用作输入/输出格式。

><center><strong>译者附</strong></center>
>a flat file : A file consisting of records of a single record type in which there is no embedded structure information that governs relationships between records.
>
>扁平结构的文件：由单一记录类型的记录组成的文件，其中没有控制记录之间关系的嵌入式结构信息。

Spark can write to `sequenceFiles` using the `saveAsObjectFile` method or by explicitly writing key–value pairs, as described in Chapter 13:

如第13章所述，Spark可以使用 `saveAsObjectFile` 方法或通过显式编写键值对来写入 `sequenceFiles`：

```scala
words.saveAsObjectFile("/tmp/my/sequenceFilePath")
```

### <font color="#00000">Hadoop Files</font>

There are a variety of different Hadoop file formats to which you can save. These allow you to specify classes, output formats, Hadoop configurations, and compression schemes. (For information on these formats, read Hadoop: The Definitive Guide [O’Reilly, 2015].) These formats are largely irrelevant except if you’re working deeply in the Hadoop ecosystem or with some legacy `mapReduce` jobs.

您可以保存多种不同的Hadoop文件格式。这些允许您指定类，输出格式，Hadoop配置和压缩方案。（有关这些格式的信息，请阅读 O’Reilly 2015年出版的《Hadoop权威指南》这些格式在很大程度上无关紧要，除非您正在Hadoop生态系统中深入工作或使用一些旧的 `mapReduce` 作业。

## <font color="#9a161a">Caching</font>

The same principles apply for caching RDDs as for DataFrames and Datasets. You can either cache or persist an RDD. By default, cache and persist only handle data in memory. We can name it if we use the `setName` function that we referenced previously in this chapter:

缓存RDD的原理与DataFrame和Dataset的原理相同。您可以缓存或保留RDD。默认情况下，缓存和持久性仅处理内存中的数据。如果使用本章前面引用的`setName`函数，则可以为它命名：

```scala
words.cache()
```

We can specify a storage level as any of the storage levels in the singleton object: `org.apache.spark.storage.StorageLevel`, which are combinations of memory only; disk only; and separately, off heap.

我们可以将存储级别指定为单例对象中的任何存储级别：`org.apache.spark.storage.StorageLevel`，它们是仅在内存，仅在磁盘以及内存和磁盘的组合存储。

 We can subsequently query for this storage level (we talk about storage levels when we discuss persistence in Chapter 20):

随后，我们可以查询该存储级别（在第20章中讨论持久性时，我们将讨论存储级别）：

```scala
// in Scala
words.getStorageLevel
```

```python
# in Python
words.getStorageLevel()
```

## <font color="#9a161a">Checkpointing</font>

One feature not available in the DataFrame API is the concept of checkpointing. Checkpointing is the act of saving an RDD to disk so that future references to this RDD point to those intermediate partitions on disk rather than recomputing the RDD from its original source. This is similar to caching except that it’s not stored in memory, only disk. This can be helpful when performing iterative computation, similar to the use cases for caching:

DataFrame API中不可用的一项功能是检查点的概念。检查点是将RDD保存到磁盘的行为，以便将来对该RDD的引用指向磁盘上的那些中间分区，而不是从其原始源重新计算RDD。除了不存储在内存中，仅存储在磁盘中，这与缓存相似。这在执行迭代计算时可能会有所帮助，类似于缓存的用例：

```scala
spark.sparkContext.setCheckpointDir("/some/path/for/checkpointing")
words.checkpoint()
```

Now, when we reference this RDD, it will derive from the checkpoint instead of the source data. This can be a helpful optimization.

现在，当我们引用此RDD时，它将从检查点而不是源数据派生。这可能是有用的优化。

## <font color="#9a161a">Pipe RDDs to System Commands</font>

The pipe method is probably one of Spark’s more interesting methods. With pipe, you can return an RDD created by piping elements to a forked external process. The resulting RDD is computed by executing the given process once per partition. All elements of each input partition are written to a process’s stdin as lines of input separated by a newline. The resulting partition consists of the process’s `stdout` output, with each line of `stdout` resulting in one element of the output partition. A process is invoked even for empty partitions.

管道方法可能是Spark更有趣的方法之一。使用管道，可以将通过将元素传递到分叉的外部过程来创建的RDD。通过对每个分区执行一次给定的过程来计算得出的RDD。每个输入分区的所有元素都以换行符分隔的形式输入到进程的stdin中。结果分区由进程的 `stdout` 输出组成，每行 `stdout`产生输出分区的一个元素。甚至为空分区调用一个进程。

The print behavior can be customized by providing two functions.

可以通过提供两个函数来自定义打印行为。

 We can use a simple example and pipe each partition to the command wc. Each row will be passed in as a new line, so if we perform a line count, we will get the number of lines, one per partition:

我们可以使用一个简单的示例，并将每个分区通过管道传递给命令wc。每行将作为新行传递，因此，如果执行行计数，我们将获得行数，每个分区一个：

```scala
words.pipe("wc -l").collect()
```

In this case, we got five lines per partition.

在这种情况下，每个分区有五行。

### <font color="#00000">mapPartitions</font>

The previous command revealed that Spark operates on a per-partition basis when it comes to actually executing code. You also might have noticed earlier that the return signature of a map function on an RDD is actually `MapPartitionsRDD`. This is because map is just a row-wise alias for `mapPartitions`, which makes it possible for you to map an individual partition (represented as an iterator). That’s because physically on the cluster we operate on each partition individually (and not a specific row). A simple example creates the value “1” for every partition in our data, and the sum of the following expression will count the number of partitions we have:

上一条命令显示，Spark在实际执行代码时会按分区运行。您之前可能还已经注意到，RDD上的映射函数的返回签名实际上是 `MapPartitionsRDD`。这是因为map只是 `mapPartitions`  的行别名，这使您可以映射单个分区（表示为迭代器）。这是因为从物理上讲，我们在集群上分别对每个分区（而不是特定的行）进行操作。一个简单的示例：为数据中的每个分区创建值“ 1”，以下表达式的总和将计算我们拥有的分区数：

 ```scala
// in Scala
words.mapPartitions(part => Iterator[Int](1)).sum() // 2
 ```

```python
# in Python
words.mapPartitions(lambda part: [1]).sum() # 2
```

Naturally, this means that we operate on a per-partition basis and allows us to perform an operation on that entire partition. This is valuable for performing something on an entire subdataset of your RDD. You can gather all values of a partition class or group into one partition and then operate on that entire group using arbitrary functions and controls. An example use case of this would be that you could pipe this through some custom machine learning algorithm and train an individual model for that company’s portion of the dataset. A Facebook engineer has an interesting demonstration of their particular implementation of <u><a style="color:#0879e3" href="https://databricks.com/session/experiences-with-sparks-rdd-apis-for-complex-custom-applications">the pipe operator</a></u> with a similar use case <u><a style="color:#0879e3" href="https://spark-summit.org/east-2017/events/experiences-with-sparks-rdd-apis-for-complex-custom-applications/">demonstrated at Spark Summit East 2017</a></u>.

自然地，这意味着我们在每个分区的基础上进行操作，并允许我们在整个分区上执行操作。这对于在RDD的整个子数据集上执行某些操作非常有用。您可以将一个分区类或组的所有值收集到一个分区中，然后使用任意函数和控制（动作和转换）对该整个组进行操作。一个示例是，您可以通过一些自定义的机器学习算法对此进行处理，并为该公司的数据集部分训练一个单独的模型。一位Facebook工程师通过在Spark Spark East 2017上展示了一个类似的<u><a style="color:#0879e3" href="https://spark-summit.org/east-2017/events/experiences-with-sparks-rdd-apis-for-complex-custom-applications/">用例</a></u>，有趣地演示了他们对<u><a style="color:#0879e3" href="https://databricks.com/session/experiences-with-sparks-rdd-apis-for-complex-custom-applications">管道算子</a></u>的特定实现。

Other functions similar to `mapPartitions` include `mapPartitionsWithIndex`. With this you specify a function that accepts an index (within the partition) and an iterator that goes through all items within the partition. The partition index is the partition number in your RDD, which identifies where each record in our dataset sits (and potentially allows you to debug). You might use this to test whether your map functions are behaving correctly:

其他类似于 `mapPartitions` 的功能包括 `mapPartitionsWithIndex`。使用此功能，您可以指定一个接受索引（在分区内）和一个迭代器的函数，该迭代器遍历该分区内的所有项。分区索引是RDD中的分区号，它标识数据集中每个记录的位置（并可能允许您调试）。您可以使用它来测试您的 map 函数是否行为正确：

```scala
// in Scala
def indexedFunc(partitionIndex:Int, withinPartIterator: Iterator[String]) = {
    withinPartIterator.toList.map(value => s"Partition: $partitionIndex => $value").iterator
} 

words.mapPartitionsWithIndex(indexedFunc).collect()
```

```python
# in Python
def indexedFunc(partitionIndex, withinPartIterator):
	return ["partition: {} => {}".format(partitionIndex, x) for x in withinPartIterator]
words.mapPartitionsWithIndex(indexedFunc).collect()
```

```txt
Array[String] = Array(Partition: 0 => Spark, Partition: 0 => The, Partition: 0 => Definitive, Partition: 0 => Guide, Partition: 0 => :, Partition: 1 => Big, Partition: 1 => Data, Partition: 1 => Processing, Partition: 1 => Made, Partition: 1 => Simple)
```

### <font color="#00000">foreachPartition</font>

Although `mapPartitions` needs a return value to work properly, this next function does not. `foreachPartition` simply iterates over all the partitions of the data. The difference is that the function has no return value. This makes it great for doing something with each partition like writing it out to a database. In fact, this is how many data source connectors are written. You can create our own text file source if you want by specifying outputs to the temp directory with a random ID:

尽管 `mapPartitions` 需要一个返回值才能正常工作，但是下一个函数不需要。`foreachPartition` 只是简单地遍历数据的所有分区。区别在于该函数没有返回值。这非常适合对每个分区执行操作，例如将其写到数据库中。实际上，这就是写入的数据源连接器数量。如果需要，可以通过使用随机ID将输出指定到temp目录来创建自己的文本文件源：

 ```scala
words.foreachPartition { iter =>
    import java.io._
    import scala.util.Random
    val randomFileName = new Random().nextInt()
    val pw = new PrintWriter(new File(s"/tmp/random-file-${randomFileName}.txt"))
    while (iter.hasNext) {
    pw.write(iter.next())
    } 
    pw.close()
}
 ```

You’ll find these two files if you scan your `/tmp` directory.

如果您扫描 `/tmp` 目录，则会找到这两个文件。

### <font color="#00000">glom</font>

glom is an interesting function that takes every partition in your dataset and converts them to arrays. This can be useful if you’re going to collect the data to the driver and want to have an array for each partition. However, this can cause serious stability issues because if you have large partitions or a large number of partitions, it’s simple to crash the driver.

glom是一个有趣的函数，它获取数据集中的每个分区并将其转换为数组。如果您要将数据收集到驱动程序，并希望每个分区都有一个数组，这将很有用。但是，这可能会导致严重的稳定性问题，因为如果您具有较大的分区或大量的分区，则很容易使驱动程序崩溃。

In the following example, you can see that we get two partitions and each word falls into one partition each:

在下面的示例中，您可以看到我们得到两个分区，每个单词都落入一个分区：

```scala
# in Scala
spark.sparkContext.parallelize(Seq("Hello", "World"), 2).glom().collect()
// Array(Array(Hello), Array(World))
```

```python
# in Python
spark.sparkContext.parallelize(["Hello", "World"], 2).glom().collect()
# [['Hello'], ['World']]
```

## <font color="#9a161a">Conclusion</font>

In this chapter, you saw the basics of the RDD APIs, including single RDD manipulation. Chapter 13 touches on more advanced RDD concepts, such as joins and key-value RDDs.

在本章中，您了解了RDD API的基础知识，包括单个RDD操作。第13章介绍了更高层的RDD概念，例如连接接和键值RDD。

 
