---
title: 翻译 Chapter 4 Structured API Overview
date: 2019-08-05
copyright: true
categories: English
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 4 Structured API Overview 

This part of the book will be a deep dive into Spark’s Structured APIs. The Structured APIs are a tool for manipulating all sorts of data, from unstructured log files to semi-structured CSV files and highly structured Parquet files. These APIs refer to three core types of distributed collection APIs:

本书的这一部分将深入探讨Spark的结构化API。结构化API是用于处理各种数据的工具，从非结构化日志文件到半结构化CSV文件以及高度结构化的Parquet文件。这些API指的是分布式集合API的三种核心类型：

- Datasets
- DataFrames
- SQL tables and views SQL表和视图

Although they are distinct parts of the book, the majority of the Structured APIs apply to both batch and streaming computation. This means that when you work with the Structured APIs, it should be simple to migrate from batch to streaming (or vice versa) with little to no effort. We’ll cover streaming in detail in Part V.

尽管它们是本书的不同部分，但大多数结构化API均适用于批处理和流计算。这意味着，当您使用结构化API时，不费吹灰之力就可以轻松地从批处理迁移到流式处理（反之亦然）。我们将在第五部分中详细介绍流式传输。

The Structured APIs are the fundamental abstraction that you will use to write the majority of your data flows. Thus far in this book, we have taken a tutorial-based approach, meandering our way through much of what Spark has to offer. This part offers a more in-depth exploration. In this chapter, we’ll introduce the fundamental concepts that you should understand: the typed and untyped APIs (and their differences); what the core terminology is; and, finally, how Spark actually takes your Structured API data flows and executes it on the cluster. We will then provide more specific task-based information for working with certain types of data or data sources.

结构化API是基本的抽象概念，可用于编写大多数数据流。到目前为止，在本书中，我们采用了基于手册的方法，蜿蜒地浏览过了Spark提供的许多功能。这部分提供了更深入的探索。在本章中，我们将介绍您应该了解的基本概念：类型化API和非类型化API（及其区别）；核心术语是什么；最后，Spark实际如何获取结构化API数据流并在集群上执行它。然后，我们将提供更具体的基于任务的信息，以处理某些类型的数据或数据源。

----

<p><center><font face="constant-width" color="#737373" size=3><strong>NOTE 注意</strong></font></center></p>

Before proceeding, let’s review the fundamental concepts and definitions that we covered in Part I. Spark is a distributed programming model in which the user specifies transformations. Multiple transformations build up a directed acyclic graph of instructions. An action begins the process of executing that graph of instructions, as a single job, by breaking it down into stages and tasks to execute across the cluster. The logical structures that we manipulate with transformations and actions are DataFrames and Datasets. To create a new DataFrame or Dataset, you call a transformation. To start computation or convert to native language types, you call an action.

在继续之前，让我们回顾一下在第一部分中介绍的基本概念和定义。Spark是一个分布式编程模型，用户可以在其中指定转换。多次转换建立了指令的有向无环图。动作 (action) 通过将指令图分解为要在整个集群中执行的阶段 (stage) 和任务 (task) 来开始，作为单个作业执行该指令图的过程。我们通过转换 (transformation) 和 动作(action)操作的逻辑结构是DataFrame 和 Dataset。要创建新的DataFrame或Dataset，请调用 transformation。要开始计算或转换为本地语言类型，请调用一个 action。

---

## <font color="#9a161a">DataFrames and Datasets</font>

Part I discussed DataFrames. Spark has two notions of structured collections: DataFrames and Datasets. We will touch on the (nuanced) differences shortly, but let’s define what they both represent first.

第一部分讨论了 DataFrame。 Spark有两个结构化集合的概念：DataFrames和Datasets。 我们将在短期内探讨（细微的）差异，但让我们先定义它们分别代表什么。

DataFrames and Datasets are (distributed) table-like collections with well-defined rows and columns. Each column must have the same number of rows as all the other columns (although you can use null to specify the absence of a value) and each column has type information that must be consistent for every row in the collection. To Spark, DataFrames and Datasets represent immutable, lazily evaluated plans that specify what operations to apply to data residing at a location to generate some output. When we perform an action on a DataFrame, we instruct Spark to perform the actual transformations and return the result. These represent plans of how to manipulate rows and columns to compute the user’s desired result.

DataFrame和Dataset是（分布式的）类表集合，具有明确定义的行和列。 每列必须具有与所有其他列相同的行数（尽管您可以使用null来指定不存在值），并且每一列的类型信息必须与集合中的每一行保持一致。 对于Spark，DataFrame和Dataset表示不可变，惰性求值的计划，这些计划指定对驻留在某个位置的数据进行哪些操作以生成某些输出。 当我们在DataFrame上执行 action 时，我们指示Spark执行实际的转换并返回结果。 这些代表如何操作行和列以计算用户期望结果的计划。

---

<p><center><font face="constant-width" color="#737373" size=3><strong>NOTE 注意</strong></font></center></p>

Tables and views are basically the same thing as DataFrames. We just execute SQL against them instead o DataFrame code. We cover all of this in Chapter 10, which focuses specifically on Spark SQL. To add a bit more specificity to these definitions, we need to talk about schemas, which are the way you define the types of data you’re storing in this distributed collection. 

表和视图与DataFrames基本相同。我们只是针对它们执行 SQL 而不是 DataFrame 代码。我们将在第10章中专门介绍Spark SQL。为了使这些定义更加具体，我们需要讨论模式（schema），这是定义存储在此分布式集合中的数据类型的方式。

---

## <font color="#9a161a">Schemas 模式</font>

A schema defines the column names and types of a DataFrame. You can define schemas manually or read a schema from a data source (often called schema on read). Schemas consist of types, meaning that you need a way of specifying what lies where.

模式定义了DataFrame的列名和类型。您可以手动定义模式，也可以从数据源读取模式（通常称为读取时模式）。模式由类型组成，这意味着您需要一个方式指定类型（数据列的具体类型）和相应的位置（数据列）。

## <font color="#9a161a">Overview of Structured Spark Types 结构化Spark类型概述</font>

Spark is effectively a programming language of its own. Internally, Spark uses an engine called Catalyst that maintains its own type information through the planning and processing of work. In doing so, this opens up a wide variety of execution optimizations that make significant differences. Spark types map directly to the different language APIs that Spark maintains and there exists a lookup table for each of these in Scala, Java, Python, SQL, and R. Even if we use Spark’s Structured APIs from Python or R, the majority of our manipulations will operate strictly on Spark types, not Python types. For example, the following code does not perform addition in Scala or Python; it actually performs addition purely in Spark:

Spark实际上是一种自己的编程语言。在内部，Spark使用一种称为Catalyst的引擎，该引擎通过计划和处理工作来维护自己的类型信息。这样一来，就可以开辟出各种各样的执行优化方案，从而产生显着差异。 Spark类型直接映射到Spark维护的不同语言API，并且在Scala，Java，Python，SQL和R中存在针对每种语言的查找表。即使我们使用来自Python或R的Spark的结构化API，我们大多数操作将严格针对Spark类型而不是Python类型进行操作。例如，以下代码在Scala或Python中不执行加法；它实际上仅在Spark中执行加法：

```scala
// in Scala
val df = spark.range(500).toDF("number")
df.select(df.col("number") + 10)
```

```python
# in Python
df = spark.range(500).toDF("number")
df.select(df["number"] + 10)
```

This addition operation happens because Spark will convert an expression written in an input language to Spark’s internal Catalyst representation of that same type information. It then will operate on that internal representation. We touch on why this is the case momentarily, but before we can, we need to discuss Datasets.

之所以进行这种加法操作，是因为Spark会将以一种输入语言编写的表达式转换为相同类型信息的Spark内部Catalyst表示形式，然后它将在该内部表示上运行。我们短暂地谈谈为什么会这样，但是在我们这样做之前，我们需要讨论数据集。

### <font color="#00000">DataFrames Versus Datasets DataFrames对比DataSet</font>

In essence, within the Structured APIs, there are two more APIs, the “untyped” DataFrames and the “typed” Datasets. To say that DataFrames are untyped is as lightly in accurate; they have types, but Spark maintains them completely and only checks whether those types line up to those specified in the schema at runtime. Datasets, on the other hand, check whether types conform to the specification at compile time. Datasets are only available to Java Virtual Machine (JVM)–based languages (Scala and Java) and we specify types with case classes or Java beans.

本质上，在结构化API中，还有另外两个API，即“无类型 ”DataFrames 和“有类型” Datasets。说DataFrame是未类型化的，这是不准确的。它们具有类型，但是Spark会完全维护它们，并且仅在运行时检查那些类型是否与模式中指定的类型一致。另一方面，Dataset在编译时检查类型是否符合规范。Dataset仅适用于基于Java虚拟机（JVM）的语言（Scala和Java），并且我们使用案例类或Java Bean指定类型。

For the most part, you’re likely to work with DataFrames. To Spark (in Scala), DataFrames are simply Datasets of Type Row. The “Row” type is Spark’s internal representation of its optimized in memory format for computation. This format makes for highly specialized and efficient computation because rather than using JVM types, which can cause high garbage-collection and object instantiation costs, Spark can operate on its own internal format without incurring any of those costs. To Spark (in Python or R), there is no such thing as a Dataset: everything is a DataFrame and therefore we always operate on that optimized format.

在大多数情况下，您可能会使用DataFrame。对于Spark（在Scala中），DataFrames只是类型为Row的数据集。 “Row”类型是Spark内部优化表示的内部表示形式，用于计算。这种格式可以进行高度专业化和高效的计算，因为Spark可以使用自己的内部格式运行，而不会产生任何这些代价，而不是使用JVM类型（后者可能导致高昂的垃圾收集和对象实例化成本）。对于Spark（在Python或R中），没有诸如Dataset之类的东西：一切都是DataFrame，因此我们始终以优化格式运行。

---

<p><center><font face="constant-width" color="#737373" size=3><strong>NOTE 注意</strong></font></center></p>

The internal Catalyst format is well covered in numerous Spark presentations. Given that this book is intended for a more general audience, we’ll refrain from going into the implementation. If you’re curious, there are some excellent talks by <u><a style="color:#0879e3" href="https://www.youtube.com/watch?v=5ajs8EIPWGI&feature=youtu.be">Josh Rosen</a></u> and <u><a style="color:#0879e3" href="https://www.youtube.com/watch?v=GDeePbbCz2g&feature=youtu.be">Herman van Hovell</a></u>, both of Databricks, about their work in the development of Spark’s Catalyst engine.

许多Spark演示都很好地介绍了内部Catalyst格式。 鉴于本书是为更广泛的读者准备的，我们将不着手实施。 如果您感到好奇，Databricks的 <u><a style="color:#0879e3" href="https://www.youtube.com/watch?v=5ajs8EIPWGI&feature=youtu.be">Josh Rosen</a></u> 和 <u><a style="color:#0879e3" href="https://www.youtube.com/watch?v=GDeePbbCz2g&feature=youtu.be">Herman van Hovell</a></u> 都会就他们在Spark的Catalyst引擎开发方面的工作进行精彩的演讲。

---

><center><strong>译者附</strong></center>
><center>为什么使用结构化API？</center>
>
>![1575778747016](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter4/1575778747016.png)
>
>截图来自<u><a style="color:#0879e3" href="https://www.youtube.com/watch?v=GDeePbbCz2g&feature=youtu.be">Herman van Hovell</a></u>视频的内容，非原书内容。

Understanding DataFrames, Spark Types, and Schemas takes some time to digest. What you need to know is that when you’re using DataFrames, you’re taking advantage of Spark’s optimized internal format. This format applies the same efficiency gains to all of Spark’s language APIs. If you need strict compile-time checking, read Chapter 11 to learn more about it.

了解DataFrame，Spark类型和模式需要一些时间来进行消化。您需要了解的是，在使用DataFrames时，您会利用Spark的优化内部格式。这种格式可将所有Spark语言API的效率提高相同效益。如果需要严格的编译时检查，请阅读第11章以了解更多信息。

Let’s move onto some friendlier and more approachable concepts: columns and rows.

让我们进入一些更友好，更平易近人的概念：列和行。

### <font color="#00000">Columns 列</font>

Columns represent a simple type like an integer or string, a complex type like an array or map, or a null value. Spark tracks all of this type information for you and offers a variety of ways, with which you can transform columns. Columns are discussed extensively in Chapter 5, but for the most part you can think about Spark Column types as columns in a table.

列表示简单类型（例如整数或字符串），复杂类型（例如数组或映射）或空值。 Spark会为您跟踪所有此类信息，并提供多种方式来转换列。列在第5章中进行了广泛讨论，但是在大多数情况下，您可以将Spark列类型视为表中的列。

### <font color="#00000">Rows 行</font>

A row is nothing more than a record of data. Each record in a DataFrame must be of type Row, as we can see when we `collect` the following DataFrames. We can create these rows manually from SQL, from Resilient Distributed Datasets (RDDs), from data sources, or manually from scratch. Here, we create one by using a `range`:

行只不过是数据记录。 DataFrame中的每个记录都必须是Row类型，正如我们在 `collect` 以下 DataFrame 时所看到的。我们可以从SQL，弹性分布式数据集（RDD），数据源或从头开始手动创建这些行。在这里，我们使用 `range` 创建一个：

```scala
// in Scala
spark.range(2).toDF().collect()
```

```python
# in Python
spark.range(2).collect()
```

These both result in an array of Row objects. 

这些都导致 Row 对象的数组。

### <font color="#00000">Spark Types</font>

We mentioned earlier that Spark has a large number of internal type representations. We include a handy reference table on the next several pages so that you can most easily reference what type, in your specific language, lines up with the type in Spark.

前面我们提到，Spark具有大量内部类型表示形式。 在接下来的几页中，我们将提供一个方便的参考表，以便您可以最轻松地参考特定语言与Spark中的类型对齐的类型。

Before getting to those tables, let’s talk about how we instantiate, or declare, a column to be of a certain type.

在进入这些表之前，让我们谈谈如何实例化或声明一列属于某种类型。

To work with the correct Scala types, use the following:

要使用正确的Scala类型，请使用以下命令：

```scala
import org.apache.spark.sql.types._
val b = ByteType
```

To work with the correct Java types, you should use the factory method in the following package:

要使用正确的Java类型，应使用以下软件包中的工厂方法：

```scala
import org.apache.spark.sql.types.DataTypes;
ByteType x = DataTypes.ByteType;
```

Python types at times have certain requirements, which you can see listed in Table 4-1, as do Scala and Java, which you can see listed in Tables 4-2 and 4-3, respectively. To work with the correct Python types, use the following:

有时，Python类型具有某些要求，表4-1中列出了这些要求，而Scala和Java则具有某些要求，分别在表4-2和表4-3中列出了。 要使用正确的Python类型，请使用以下命令：

```scala
from pyspark.sql.types import *
b = ByteType()
```

The following tables provide the detailed type information for each of Spark’s language bindings. 

下表提供了每种Spark语言绑定的详细类型信息。

*Table 4-1. Python type reference*

| **Data type** | **Value type in Python**                                     | **API to access or create a data type**                      |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ByteType      | int or long. Note: Numbers will be converted to 1-byte signed integer numbers at runtime. Ensure that numbers are within the range of-128 to 127.<br />int或long。注意：数字将在运行时转换为1字节有符号整数。确保数字在-128到127的范围内。 | ByteType()                                                   |
| ShortType     | int or long. Note: Numbers will be converted to 2-byte signed integer numbers at runtime. Ensure that numbers are within the range of-32768 to 32767.<br />int或long。注意：数字将在运行时转换为2字节有符号整数。确保数字在-32768到32767的范围内。 | ShortType()                                                  |
| IntegerType   | int or long. Note: Python has a lenient definition of "integer.” Numbers that are too large will be rejected by Spark SQL if you use the `IntegerType()`. It’s best practice to use LongType.<br />int或long。注意：Python的宽泛定义是“整数”。如果您使用`IntegerType()`，则太大的数字将被Spark SQL拒绝。最佳做法是使用LongType。 | Integerlype()                                                |
| LongType      | long. Note: Numbers will be converted to 8-byte signed integer numbers at runtime. Ensure that numbers are within the range of-9223372036854775808 to 9223372036854775807. Otherwise, convert data to decimaLDecimal and use DecimaFlype.<br />long。注意：数字将在运行时转换为8字节有符号整数。确保数字在-9223372036854775808到9223372036854775807之间。否则，将数据转换为decimaLDecimal并使用DecimaFlype。 | Longlype()                                                   |
| FloatType     | float. Note: Numbers will be converted to 4-byte single-precision floating-point numbers at runtime.<br />float。注意：数字将在运行时转换为4字节单精度浮点数。 | FloatType()                                                  |
| DoubleType    | float                                                        | DoubleType()                                                 |
| DecimalType   | decimalDecimal                                               | DecimalTypeO                                                 |
| StringType    | string                                                       | StringType()                                                 |
| BinaryType    | bytearray                                                    | BinaryType()                                                 |
| BooleanType   | bool                                                         | BooleanType()                                                |
| llmestamplype | datetime.datetime                                            | TlmestampTypeO                                               |
| DateType      | datetime.date                                                | DateType()                                                   |
| ArrayType     | list, tuple, or array                                        | ArrayType(elementType, [containsNull]). <br />Note: The default value of containsNull is True.<br />注意：containsNull的默认值为True。 |
| MapType       | diet                                                         | MapType(keyType, valueType, [valueContainsNull]). <br />Note: The default value of valueContainsNull is True.<br />注意：valueContainsNull的默认值为True。 |
| Structlype    | list or tuple                                                | StructType(fields). <br />Note: fields is a list of StructFields. Also, fields with the same name are not allowed.<br />注意：字段是StructFields的列表。同样，不允许使用具有相同名称的字段。 |
| StructField   | The value type in Python of the data type of this field (for example, Int for a StructField with the data type IntegerType) | StructField(name, datalype, [nullable]) <br />Note: The defaul value of nullable is True.<br />注意：nullable的默认值为True。 |

*Table 4-2. Scala type reference*

| **Data type** | **Value type in Scala**                                      | **API to access or create a data type**                      |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ByteType      | Byte                                                         | ByteType                                                     |
| ShortType     | Short                                                        | ShortType                                                    |
| IntegerType   | Int                                                          | IntegerType                                                  |
| LongType      | Long                                                         | LongType                                                     |
| FloatType     | Float                                                        | FloatType                                                    |
| DoubleType    | Double                                                       | DoubleType                                                   |
| DecimalType   | java.math.BigDecimal                                         | DecimalType                                                  |
| StringType    | String                                                       | StringType                                                   |
| BinaryType    | Array[Byte]                                                  | BinaryType                                                   |
| BooleanType   | Boolean                                                      | BooleanType                                                  |
| TimestampType | java.sql.Timestamp                                           | TimestampType                                                |
| DateType      | java.sql.Date                                                | DateType                                                     |
| ArrayType     | scala.collection.Seq                                         | ArrayType(elementType, [containsNull]). <br />Note: The default value of containsNull is true.<br />注意：containsNull的默认值为true。 |
| MapType       | scala.collection.Map                                         | MapType(keyType, valueType, [valueContainsNull]). <br />Note: The default value of valueContainsNull is true.<br />注意：valueContainsNull的默认值为true。 |
| StructType    | org.apache.spark.sql.Row                                     | StructType(fields). <br />Note: fields is an Array of StructFields. Also, fields with the same name are not allowed.<br />注意：字段是StructFields的数组。同样，不允许使用具有相同名称的字段。 |
| StructField   | The value type in Scala of the data type of this field (for example, Int for a StructField with the data type IntegerType)<br />Scala中此字段的数据类型的值类型（例如，对于数据类型为IntegerType的StructField为Int） | StructField(name, dataType, [nullable]). <br />Note: The default value of nullable is true.<br />注意：nullable的默认值为true。 |

*Table 4-3. Java type reference* 

| **Data type** | **Value type in Java**                                       | **API to access or create a data type**                      |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ByteType      | byte or Byte                                                 | DataTypes. ByteType                                          |
| ShortType     | short or Short                                               | DataTypes. ShortType                                         |
| IntegerType   | int or Integer                                               | DataTypes. IntegerType                                       |
| LongType      | long or Long                                                 | DataTypes. LongType                                          |
| FloatType     | float or Float                                               | DataTypes. FloatType                                         |
| DoubleType    | double or Double                                             | DataTypes. DoubleType                                        |
| DecinialType  | java .math.BigDecimal                                        | DataTypes.createDecinialType()<br />DataTypes.createDecinialType(precision, scale). |
| StringType    | String                                                       | DataTypes. StringType                                        |
| BmaryType     | byte[]                                                       | DataTypes. BinaryType                                        |
| BooleanType   | boolean or Boolean                                           | DataTypes. BooleanType                                       |
| TimestampType | java. sqL Timestamp                                          | DataTypes.TimestampType                                      |
| DateType      | java.sqLDate                                                 | DataTypes. DateType                                          |
| ArrayType     | java.utiLList                                                | DataTypes.createArrayType(elementType). Note: The value of contamsNull will be true.<br />DataTypes.createArrayType(elementType, contamsNull). |
| MapType       | java.util.Map                                                | DataTypes.createMapType(keyType, vahieType). <br />Note: The value of valueContainsNull will be true.<br />注意：valueContainsNull的值将为true。<br />DataTypes.createMapType(keyType, vahieType, vahieContainsNull) |
| StructType    | org.apache.spark.sql.Row                                     | DataTypes.createStructType(fieIds). <br />Note: fields is a List or an array of StructFields. Also, two fields with the same name are not alfowed.<br />注意：字段是StructField的列表或数组。同样，两个同名字段也不被允许。 |
| StructField   | The value type in Java of the data type of this field (for example, int for a StructField with the data type IntegerType)<br />Java中此字段的数据类型的值类型（例如，数据类型为IntegerType的StructField的int） | DataTypes.createStructField(name, dataType, nullable)        |

## <font color="#9a161a">Overview of Structured API Execution 结构化API执行概述</font>

This section will demonstrate how this code is actually executed across a cluster. This will help you understand (and potentially debug) the process of writing and executing code on clusters, so let’s walk through the execution of a single structured API query from user code to executed code. Here’s an overview of the steps:

本节将演示如何在整个集群中实际执行此代码。这将帮助您了解（并可能调试）在集群上编写和执行代码的过程，因此让我们逐步执行从用户代码到执行代码的单个结构化API查询的执行。以下是步骤概述：

1. Write DataFrame/Dataset/SQL Code. 

    编写DataFrame / Dataset / SQL代码。

2. If valid code, Spark converts this to a Logical Plan.

    如果是有效代码，Spark会将其转换为逻辑计划。

3. Spark transforms this Logical Plan to a Physical Plan, checking for optimizations along the way.

    Spark将此逻辑计划转换为物理计划，并按照方式检查优化。

4. Spark then executes this Physical Plan (RDD manipulations) on the cluster.

    然后，Spark在集群上执行此物理计划（RDD操作）。

To execute code, we must write code. This code is then submitted to Spark either through the console or via a submitted job. This code then passes through the Catalyst Optimizer, which decides how the code should be executed and lays out a plan for doing so before, finally, the code is run and the result is returned to the user. Figure 4-1 shows the process. 

要执行代码，我们必须编写代码。然后，此代码通过控制台或提交的作业提交给Spark。然后，此代码通过Catalyst Optimizer，后者确定应如何执行代码，并在此之前制定执行计划，最后，代码将运行并将结果返回给用户。流程如图4-1所示。

![1572090019513](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter4/1572090019513.png)

### <font color="#00000">Logical Planning 逻辑规划</font>

The first phase of execution is meant to take user code and convert it into a logical plan. Figure 4-2 illustrates this process. 

执行的第一阶段旨在获取用户代码并将其转换为逻辑计划。图4-2说明了此过程。

![1572090084025](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter4/1572090084025.png) 

This logical plan only represents a set of abstract transformations that do not refer to executors or drivers, it’s purely to convert the user’s set of expressions into the most optimized version. It does this by converting user code into an unresolved logical plan. This plan is unresolved because although your code might be valid, the tables or columns that it refers to might or might not exist. Spark uses the catalog, a repository of all table and DataFrame information, to resolve columns and tables in the analyzer. The analyzer might reject the unresolved logical plan if the required table or column name does not exist in the catalog. If the analyzer can resolve it, the result is passed through the Catalyst Optimizer, a collection of rules that attempt to optimize the logical plan by pushing down predicates or selections. Packages can extend the Catalyst to include their own rules for domain specific optimizations.

此逻辑计划仅代表一组抽象转换，这些转换不涉及 executor 或 driver，而仅仅是将用户的表达式集转换为最优化的版本。它通过将用户代码转换为未解析的逻辑计划来实现。该计划尚未解析，因为尽管您的代码可能有效，但它所引用的表或列可能存在或可能不存在。 Spark使用 catalog（所有表和DataFrame信息的存储库）解析分析器中的列和表。如果所需的表或列名称在目录中不存在，则分析器可能会拒绝未解析的逻辑计划。如果分析器可以解析问题，则结果将通过Catalyst Optimizer传递，Catalyst Optimizer是一组规则的集合，这些规则试图通过向下推理谓词（predicates）或选择（selections）来优化逻辑计划。软件包可以扩展Catalyst，以包括其用于特定领域优化的规则。

### <font color="#00000">Physical Planning 物理规划</font>

After successfully creating an optimized logical plan, Spark then begins the physical planning process. The physical plan, often called a Spark plan, specifies how the logical plan will execute on the cluster by generating different physical execution strategies and comparing them through a cost model, as depicted in Figure 4-3. An example of the cost comparison might be choosing how to perform a given join by looking at the physical attributes of a given table (how big the table is or how big its partitions are) 

成功创建优化的逻辑计划后，Spark然后开始物理计划过程。物理计划通常称为Spark计划，它通过生成不同的物理执行策略并通过成本模型进行比较来指定逻辑计划在集群上的执行方式，如图4-3所示。成本比较的一个示例可能是通过查看给定表的物理属性（表的大小或分区的大小）来选择如何执行给定的连接。

![1572090201269](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter4/1572090201269.png)

Physical planning results in a series of RDDs and transformations. This result is why you might have heard Spark referred to as a compiler—it takes queries in DataFrames, Datasets, and SQL and compiles them into RDD transformations for you.

物理规划会导致一系列的RDD和转换。这个结果就是为什么您可能听说过Spark称为编译器的原因——它接受DataFrames，Datasets和SQL中的查询，然后将它们编译为RDD转换。

### <font color="#00000">Execution 执行</font>

Upon selecting a physical plan, Spark runs all of this code over RDDs, the lower-level programming interface of Spark (which we cover in Part III). Spark performs further optimizations at runtime, generating native Java bytecode that can remove entire tasks or stages during execution. Finally the result is returned to the user.

选择了物理计划后，Spark将在RDD（Spark的较底层编程接口）上运行所有这些代码（我们将在第III部分中介绍）。 Spark在运行时执行进一步的优化，生成本地Java字节码，可以在执行过程中删除整个任务或阶段。最后，结果返回给用户。

## <font color="#9a161a">Conclusion 结论</font>

In this chapter, we covered Spark Structured APIs and how Spark transforms your code into what will physically execute on the cluster. In the chapters that follow, we cover core concepts and how to use the key functionality of the Structured APIs. 

在本章中，我们介绍了Spark结构化API，以及Spark如何将您的代码转换为将在集群上实际执行的代码。在接下来的章节中，我们将介绍核心概念以及如何使用结构化API的关键功能。
