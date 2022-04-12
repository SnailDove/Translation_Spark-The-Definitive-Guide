---
title: 翻译 Chapter 5 Basic Structured Operations
date: 2019-08-05
copyright: true
categories: English
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 5 Basic Structured Operations 基本结构的操作
<center><strong>译者</strong>：<u><a style="color:#0879e3" href="https://snaildove.github.io">https://snaildove.github.io</a></u></center>

In Chapter 4, we introduced the core abstractions of the Structured API. This chapter moves away from the architectural concepts and toward the tactical tools you will use to manipulate DataFrames and the data within them. This chapter focuses exclusively on fundamental DataFrame operations and avoids aggregations, window functions, and joins. These are discussed in subsequent chapters. Definitionally, a DataFrame consists of a series of records (like rows in a table), that are of type Row, and a number of columns (like columns in a spreadsheet) that represent a computation expression that can be performed on each individual record in the Dataset. Schemas define the name as well as the type of data in each column. Partitioning of the DataFrame defines the layout of the DataFrame or Dataset’s physical distribution across the cluster. The partitioning scheme defines how that is allocated. You can set this to be based on values in a certain column or nondeterministically.

在第4章中，我们介绍了结构化API的核心抽象。 本章从体系结构概念转向使用战术工具，您将使用这些工具来操纵DataFrame及其中的数据。 本章专门介绍DataFrame的基本操作，并避免聚合，窗口函数和连接。 这些将在后续章节中讨论。 从定义上讲，DataFrame由一系列记录（如表中的行）组成，这些记录的类型为Row，以及许多列（如电子表格中的列），他们可以表示成对Dataset中的每个单独记录执行的计算表达式。 模式定义每列中的名称和数据类型。 DataFrame的分区定义了整个集群中DataFrame或Dataset物理分布的布局。 分区方案定义了如何分配。 您可以将其设置为基于特定列中的值或不确定地。

Let’s create a DataFrame with which we can work:

让我们创建一个可以使用的DataFrame：

```scala
// in Scala
val df = spark.read.format("json")
.load("/data/flight-data/json/2015-summary.json")
```

```python
# in Python
df = spark.read.format("json").load("/data/flight-data/json/2015-summary.json")
```

We discussed that a DataFame will have columns, and we use a schema to define them. Let’s take a look at the schema on our current DataFrame:

我们讨论了DataFame将具有列，并使用一种模式来定义它们。让我们看一下当前DataFrame上的模式：

```scala
df.printSchema()
```

Schemas tie everything together, so they’re worth belaboring.

模式将所有内容捆绑在一起，因此值得反复强调。

## <font color="#9a161a">Schemas 模式</font>

A schema defines the column names and types of a DataFrame. We can either let a data source define the schema (called schema-on-read) or we can define it explicitly ourselves.

模式定义了DataFrame的列名和类型。我们可以让数据源定义模式（称为读取时的模式schema-on-read），也可以自己显示定义。

---

<p><center><font face="constant-width" color="#c67171" size=3><strong>WARNING 警告</strong></font></center></p>
Deciding whether you need to define a schema prior to reading in your data depends on your use case. For ad hoc analysis, schema-on-read usually works just fine (although at times it can be a bit slow with plain-text file formats like CSV or JSON). However, this can also lead to precision issues like a long type incorrectly set as an integer when reading in a file. When using Spark for production Extract, Transform, and Load (ETL), it is often a good idea to define your schemas manually, especially when working with untyped data sources like CSV and JSON because schema inference can vary depending on the type of data that you read in.

在读取数据之前决定是否需要定义模式取决于您的用例。对于临时分析，读取时的模式（schema-on-read）通常效果很好（尽管有时使用CSV或JSON等纯文本文件格式可能会有点慢）。但是，这也可能导致精度问题，例如在读取文件时将long类型错误地设置为整数。当使用Spark进行生产提取，转换和加载（ETL）时，手动定义模式通常是一个好主意，尤其是在使用CSV和JSON等无类型数据源时，因为模式推断会根据你所读取的数据类型的不同而有所不同。

---

Let’s begin with a simple file, which we saw in Chapter 4, and let the semi-structured nature of line delimited JSON define the structure. This is flight data from the United States Bureau of Transportation statistics:

让我们从第4章中看到的简单文件开始，让以行分隔JSON的半结构化性质定义结构。这是来自美国运输局统计数据的航班数据：

```scala
// in Scala
spark.read.format("json").load("/data/flight-data/json/2015-summary.json").schema
Scala returns the following:
org.apache.spark.sql.types.StructType = ...
StructType(StructField(DEST_COUNTRY_NAME,StringType,true),
StructField(ORIGIN_COUNTRY_NAME,StringType,true),
StructField(count,LongType,true))
```

```python
# in Python
spark.read.format("json").load("/data/flight-data/json/2015-summary.json").schema
Python returns the following:
StructType(List(StructField(DEST_COUNTRY_NAME,StringType,true),
StructField(ORIGIN_COUNTRY_NAME,StringType,true),
StructField(count,LongType,true)))
```

A schema is a `StructType` made up of a number of fields, `StructFields`, that have a name, type, a Boolean flag which specifies whether that column can contain missing or null values, and, finally, users can optionally specify associated metadata with that column. The metadata is a way of storing information about this column (Spark uses this in its machine learning library).

模式是一种 `StructType`，由许多字段，`StructFields` 组成，这些字段具有名称，类型，布尔值标志，用于指定该列可以包含缺失值还是null值，最后，用户可以选择指定与该列关联的元数据 。 元数据是一种存储有关此列的信息的方式（Spark在其机器学习库中使用此信息）。

Schemas can contain other `StructTypes` (Spark’s complex types). We will see this in Chapter 6 when we discuss working with complex types. If the types in the data (at runtime) do not match the schema, Spark will throw an error. The example that follows shows how to create and enforce a specific schema on a DataFrame.

模式可以包含其他 `StructType`（Spark的复杂类型）。当我们讨论使用复杂类型时，我们将在第6章中看到这一点。如果数据中的类型（在运行时）与模式不匹配，Spark将引发错误。下面的示例演示如何在DataFrame上创建和实施特定的模式。

```scala
// in Scala
import org.apache.spark.sql.types.{StructField, StructType, StringType, LongType}
import org.apache.spark.sql.types.Metadata
val myManualSchema = StructType(Array(
StructField("DEST_COUNTRY_NAME", StringType, true),
StructField("ORIGIN_COUNTRY_NAME", StringType, true),
StructField("count", LongType, false,
Metadata.fromJson("{"hello":"world"}"))
))

val df = spark.read.format("json").schema(myManualSchema)
.load("/data/flight-data/json/2015-summary.json")
```

Here’s how to do the same in Python：

在Python中执行以下操作的方法如下：

```scala
# in Python

from pyspark.sql.types import StructField, StructType, StringType, LongType
myManualSchema = StructType([
StructField("DEST_COUNTRY_NAME", StringType(), True),
StructField("ORIGIN_COUNTRY_NAME", StringType(), True),
StructField("count", LongType(), False, metadata={"hello":"world"})
])
df = spark.read.format("json").schema(myManualSchema)\
.load("/data/flight-data/json/2015-summary.json")
```

As discussed in Chapter 4, we cannot simply set types via the per-language types because Spark maintains its own type information. Let’s now discuss what schemas define: columns.

如第4章所述，我们不能简单地通过每种语言类型设置类型，因为Spark维护自己的类型信息。现在让我们讨论一下模式定义：列。

## <font color="#9a161a">Columns and Expressions</font>

Columns in Spark are similar to columns in a spreadsheet, R dataframe, or pandas DataFrame. You can select, manipulate, and remove columns from DataFrames and these operations are represented as expressions.

Spark中的列与电子表格，R dataframe 或 pandas DataFrame 中的列相似。您可以从DataFrames中选择，操作和删除列，并且这些操作表示为表达式。

To Spark, columns are logical constructions that simply represent a value computed on a per-record basis by means of an expression. This means that to have a real value for a column, we need to have a row; and to have a row, we need to have a DataFrame. You cannot manipulate an individual column outside the context of a DataFrame; you must use Spark transformations within a DataFrame to modify the contents of a column. 

对Spark而言，列是逻辑构造，仅表示通过表达式基于每个记录计算的值。这意味着要有一个列的实际值，我们需要有一行；要有一行，我们需要有一个DataFrame。您不能在DataFrame上下文之外操作单个列；您必须在DataFrame中使用Spark转换来修改列的内容。

```python
// in Scala
import org.apache.spark.sql.functions.{col, column}
col("someColumnName")
column("someColumnName")
```

```python
# in Python
from pyspark.sql.functions import col, column
col("someColumnName")
column("someColumnName")
```

We will stick to using col throughout this book. As mentioned, this column might or might not exist in our DataFrames. Columns are not resolved until we compare the column names with those we are maintaining in the catalog. Column and table resolution happens in the analyzer phase, as discussed in Chapter 4.

在本书中，我们将坚持使用col。如前所述，此列可能存在也可能不存在于我们的DataFrame中。在将列名与目录中维护的列名进行比较之前，不会解析列。列和表的解析发生在分析器阶段，如第4章所述。

---

<p><center><font face="constant-width" color="#737373" size=3><strong>NOTE 注意</strong></font></center></P>
We just mentioned two different ways of referring to columns. Scala has some unique language features that allow for more shorthand ways of referring to columns. The following bits of syntactic sugar perform the exact same thing, namely creating a column, but provide no performance improvement:

我们刚刚提到了两种不同的列引用方式。 Scala具有一些独特的语言功能，允许使用更简便的方式引用列。语法糖的以下几部分执行的操作完全相同，即创建一个列，但没有改善性能：

```scala
// in Scala
$"myColumn"
'myColumn
```

The allows us to designate a string as a special string that should refer to an expression. The tick mark (') is a special thing called a symbol; this is a Scala-specific construct of referring to some identifier. They both perform the same thing and are shorthand ways of referring to columns by name. You’ll likely see all of the aforementioned references when you read different people’s Spark code. We leave it to you to use whatever is most comfortable and maintainable for you and those with whom you work.

允许我们将字符串指定为应引用表达式的特殊字符串。记号（'）是一种特殊的东西，称为符号；这是引用某些标识符的Scala专门构建的一种方式。它们都执行相同的操作，并且是通过名称引用列的简便方法。阅读其他人的Spark代码时，您可能会看到所有上述参考。我们留给您，使用对您和与您一起工作的人来说最舒适和可维护的任何一种方式。

---

#### <font color="#3399cc">Explicit column references 显示的列引用</font>

If you need to refer to a specific DataFrame’s column, you can use the `col` method on the specific DataFrame. This can be useful when you are performing a join and need to refer to a specific column in one DataFrame that might share a name with another column in the joined DataFrame. We will see this in Chapter 8. As an added benefit, Spark does not need to resolve this column itself (during the analyzer phase) because we did that for Spark:

如果您需要引用特定DataFrame的列，则可以在特定DataFrame上使用 `col` 方法。当您执行连接并需要引用一个DataFrame中的特定列，而该列可能与连接的DataFrame中的另一列共享名称时，此功能很有用。我们将在第8章中看到这一点。作为一个额外的好处，Spark不需要自己解决此列（在分析器阶段），因为我们为Spark做到了：

```scala
df.col("count")
```

### <font color="#00000">Expressions 表达式</font>

We mentioned earlier that columns are expressions, but what is an expression? An expression is a set of transformations on one or more values in a record in a DataFrame. Think of it like a function that takes as input one or more column names, resolves them, and then potentially applies more expressions to create a single value for each record in the dataset. Importantly, this “single value” can actually be a complex type like a Map or Array. We’ll see more of the complex types in Chapter 6. In the simplest case, an expression, created via the expr function, is just a DataFrame column reference. In the simplest case, `expr("someCol")` is equivalent to `col("someCol")`.

前面我们提到列是表达式，但是什么是表达式？表达式是对DataFrame中记录中一个或多个值的一组转换。可以将其视为一个函数，该函数将一个或多个列名作为输入，进行解析，然后可能应用更多表达式为数据集中的每个记录创建单个值。重要的是，此“单个值”实际上可以是诸如Map或Array之类的复杂类型。我们将在第6章中看到更多复杂类型。在最简单的情况下，通过expr函数创建的表达式仅是DataFrame列引用。在最简单的情况下，`expr("someCol")`等同于`col("someCol")`。

#### <font color="#3399cc">Columns as expressions 列作为表达式</font>

Columns provide a subset of expression functionality. If you use col() and want to perform transformations on that column, you must perform those on that column reference. When using an expression, the expr function can actually parse transformations and column references from a string and can subsequently be passed into further transformations. Let’s look at some examples. `expr("someCol - 5")` is the same transformation as performing `col("someCol") - 5`, or even `expr("someCol") - 5`. 

列提供了表达式功能的子集。如果使用 `col()` 并想在该列上执行转换，则必须在该列引用上执行那些转换。使用表达式时，expr函数实际上可以解析字符串中的转换和列引用，并且可以随后将其传递到其他转换中。让我们看一些例子。`expr("someCol - 5")` 是与执行 `col("someCol") - 5` 相同的转换，或者甚至与：`expr("someCol") - 5` 相同。

That’s because Spark compiles these to a logical tree specifying the order of operations. This might be a bit confusing at first, but remember a couple of key points:

这是因为Spark将这些内容编译为指定操作顺序的逻辑树。刚开始时这可能有点令人困惑，但请记住几个要点：

- Columns are just expressions. 

    列只是表达式。

- Columns and transformations of those columns compile to the same logical plan as parsed expressions. 

    列和这些列的转换编译为与经过解析的表达式拥有相同的逻辑计划。

Let’s ground this with an example:

让我们以一个示例为基础：

```scala
(((col("someCol") + 5) * 200) - 6) < col("otherCol")
```
Figure 5-1 shows an overview of that logical tree. 

图 5-1 展示一个逻辑树的整体概述。

![1572143049723](C:/Users/ruito/AppData/Roaming/Typora/typora-user-images/1572143049723.png)

This might look familiar because it’s a directed acyclic graph. This graph is represented equivalently by the following code:

这可能看起来很熟悉，因为它是有向无环图。此图由以下代码等效表示：

```scala
// in Scala
import org.apache.spark.sql.functions.expr
expr("(((someCol + 5) * 200) - 6) < otherCol")
```

```python
# in Python
from pyspark.sql.functions import expr
expr("(((someCol + 5) * 200) - 6) < otherCol")
```

This is an extremely important point to reinforce. Notice how the previous expression is actually valid SQL code, as well, just like you might put in a SELECT statement? That’s because this SQL expression and the previous DataFrame code compile to the same underlying logical tree prior to execution. This means that you can write your expressions as DataFrame code or as SQL expressions and get the exact same performance characteristics. This is discussed in Chapter 4.

这是必须加强的极为重要的一点。注意前面的表达式实际上是多么有效的SQL代码，就像您可能在SELECT语句中一样吗？这是因为该SQL表达式和之前的DataFrame代码在执行之前会编译为相同的基础逻辑树。这意味着您可以将表达式编写为DataFrame代码或SQL表达式，并获得完全相同的性能特性。这将在第4章中讨论。

#### <font color="#3399cc">Accessing a DataFrame’s columns 访问DataFrame的列</font>

Sometimes, you’ll need to see a DataFrame’s columns, which you can do by using something like `printSchema`; however, if you want to programmatically access columns, you can use the columns property to see all columns on a DataFrame:

有时，您需要查看DataFrame的列，您可以使用诸如`printSchema`之类的方法来完成；但是，如果要以编程方式访问列，则可以使用columns属性查看DataFrame上的所有列：

```scala
spark.read.format("json").load("/data/flight-data/json/2015-summary.json")
.columns
```

## <font color="#9a161a">Records and Rows 记录和行</font>

In Spark, each row in a DataFrame is a single record. Spark represents this record as an object of type Row. Spark manipulates Row objects using column expressions in order to produce usable values. Row objects internally represent arrays of bytes. The byte array interface is never shown to users because we only use column expressions to manipulate them.

在Spark中，DataFrame中的每一行都是一条记录。 Spark将此记录表示为Row类型的对象。 Spark使用列表达式操纵Row对象，以产生可用的值。 行对象在内部表示字节数组。 字节数组接口从未显示给用户，因为我们仅使用列表达式来操作它们。

You’ll notice commands that return individual rows to the driver will always return one or more Row types when we are working with DataFrames.

您会注意到，当我们使用DataFrames时，将单个行返回给驱动程序的命令将始终返回一种或多种行类型。

---

<p><center><font face="constant-width" color="#737373" size=3><strong>NOTE 注意</strong></font></center></P>
We use lowercase “row” and “record” interchangeably in this chapter, with a focus on the latter. A capitalized Row refers to the Row object.

在本章中，我们将小写的“行”和“记录”互换使用，重点是后者。大写的行是指Row对象。

---

Let’s see a row by calling first on our DataFrame:

让我们先在DataFrame上调用以下行：

```scala
df.first()
```

### <font color="#00000">Creating Rows 创建行</font>

You can create rows by manually instantiating a Row object with the values that belong in each column. It’s important to note that only DataFrames have schemas. Rows themselves do not have schemas. This means that if you create a Row manually, you must specify the values in the same order as the schema of the DataFrame to which they might be appended (we will see this when we discuss creating DataFrames):

您可以通过手动实例化具有每个列中的值的Row对象来创建行。 请务必注意，只有 DataFrame 具有模式。 行本身没有模式。 这意味着，如果您手动创建Row，则必须以与可能被附加的DataFrame模式相同的顺序指定值（在讨论创建DataFrame时将看到此值）：

```scala
// in Scala
import org.apache.spark.sql.Row
val myRow = Row("Hello", null, 1, false)
```

```python
# in Python
from pyspark.sql import Row
myRow = Row("Hello", None, 1, False)
```

Accessing data in rows is equally as easy: you just specify the position that you would like. In Scala or Java, you must either use the helper methods or explicitly coerce the values. However, in Python or R, the value will automatically be coerced into the correct type : 

访问行中的数据同样容易：只需指定所需的位置即可。 在Scala或Java中，必须使用辅助方法或显式强制值。 但是，在Python或R中，该值将自动强制为正确的类型：

```scala
// in Scala
myRow(0) // type Any
myRow(0).asInstanceOf[String] // String
myRow.getString(0) // String
myRow.getInt(2) // Int
```

```python
# in Python
myRow[0]
myRow[2]
```

You can also explicitly return a set of Data in the corresponding Java Virtual Machine (JVM) objects by using the Dataset APIs. This is covered in Chapter 11.

您还可以使用Dataset API在相应的Java虚拟机（JVM）对象中显式返回一组数据。 这将在第11章中介绍。

## <font color="#9a161a">DataFrame Transformations DataFrame转换</font>

Now that we briefly defined the core parts of a DataFrame, we will move onto manipulating DataFrames. When working with individual DataFrames there are some fundamental objectives. These break down into several core operations, as depicted in Figure 5-2:

现在，我们简要定义了DataFrame的核心部分，我们将继续操作DataFrame。使用单个DataFrame时，有一些基本目标。这些细分为几个核心操作，如图5-2所示：

- We can add rows or columns
我们可以添加行或列
- We can remove rows or columns
我们可以删除行或列
- We can transform a row into a column (or vice versa)
我们可以将一行转换成一列（反之亦然）
- We can change the order of rows based on the values in columns 
我们可以根据列中的值更改行的顺序

![1572143485630](C:/Users/ruito/AppData/Roaming/Typora/typora-user-images/1572143485630.png)

Luckily, we can translate all of these into simple transformations, the most common being those that take one column, change it row by row, and then return our results.

幸运的是，我们可以将所有这些转换为简单的转换，最常见的转换是采用一列，逐行更改然后返回结果的转换。

### <font color="#00000">Creating DataFrames 创建数据框</font>

As we saw previously, we can create DataFrames from raw data sources. This is covered extensively in Chapter 9; however, we will use them now to create an example DataFrame (for illustration purposes later in this chapter, we will also register this as a temporary view so that we can query it with SQL and show off basic transformations in SQL, as well) : 

如前所述，我们可以从原始数据源创建DataFrame。第9章对此进行了广泛讨论。但是，我们现在将使用它们创建一个示例DataFrame（出于本章稍后的说明目的，我们还将其注册为一个临时视图，以便我们可以使用SQL查询它，并展示SQL中的基本转换）:

```scala
// in Scala
val df = spark.read.format("json")
.load("/data/flight-data/json/2015-summary.json")
df.createOrReplaceTempView("dfTable")
```
```python
# in Python
df = spark.read.format("json").load("/data/flight-data/json/2015-summary.json")
df.createOrReplaceTempView("dfTable")
```

We can also create DataFrames on the fly by taking a set of rows and converting them to a DataFrame. 

我们还可以通过获取一组行并将其转换为 DataFrame 来动态（程序运行时）创建 DataFrame。

```scala
// in Scala
import org.apache.spark.sql.Row
import org.apache.spark.sql.types.{StructField, StructType, StringType, LongType}
val myManualSchema = new StructType(Array(
new StructField("some", StringType, true),
new StructField("col", StringType, true),
new StructField("names", LongType, false)))
val myRows = Seq(Row("Hello", null, 1L))
val myRDD = spark.sparkContext.parallelize(myRows)
val myDf = spark.createDataFrame(myRDD, myManualSchema)
myDf.show()
```

---

<p><center><font face="constant-width" color="#737373" size=3><strong>NOTE 注意</strong></font></center></P>
In Scala, we can also take advantage of Spark’s implicits in the console (and if you import them in your JAR code) by running toDF on a Seq type. This does not play well with null types, so it’s not necessarily recommended for production use cases. 

在 Scala 中，我们还可以通过在 Seq 类型上运行 `toDF` 来利用控制台中 Spark 的隐式内容（如果您将其导入JAR代码中）。 此方法不适用于null类型，因此不一定建议在生产用例中使用。

---

```scala
// in Scala
val myDF = Seq(("Hello", 2, 1L)).toDF("col1", "col2", "col3")
```

```python
# in Python
from pyspark.sql import Row
from pyspark.sql.types import StructField, StructType, StringType, LongType

myManualSchema = StructType([
StructField("some", StringType(), True),
StructField("col", StringType(), True),
StructField("names", LongType(), False)
])

myRow = Row("Hello", None, 1)
myDf = spark.createDataFrame([myRow], myManualSchema)
myDf.show()
```
Giving an output of:

提供以下输出：

```shell
+-----+----+-----+
| some| col|names|
+-----+----+-----+
|Hello|null|  1  |
+-----+----+-----+
```

Now that you know how to create DataFrames, let’s take a look at their most useful methods that you’re going to be using: the `select` method when you’re working with columns or expressions, and the `selectExpr` method when you’re working with expressions in strings. Naturally some transformations are not specified as methods on columns; therefore, there exists a group of functions found in the `org.apache.spark.sql.functions` package.

现在您已经知道如何创建 DataFrames，下面让我们看一下它们将要使用的最有用的方法：使用<u>列或表达式</u>时的`select` 方法，以及当你处理在字符串中的表达式的时候用 `selectExpr` 方法。自然，某些转换未指定为列上的方法；因此，在`org.apache.spark.sql.functions` 包中可以找到一组函数。

With these three tools, you should be able to solve the vast majority of transformation challenges that you might encounter in DataFrames.

使用这三个工具，您应该能够解决DataFrame中可能遇到的绝大多数转换问题。

### <font color="#00000">select and selectExpr</font>

`select` and `selectExpr` allow you to do the DataFrame equivalent of SQL queries on a table of data: 

`select` 和 `selectExpr` 允许您与数据表上执行SQL查询等效的 DataFrame 操作：

```sql
-- in SQL
SELECT * FROM dataFrameTable
SELECT columnName FROM dataFrameTable
SELECT columnName * 10, otherColumn, someOtherCol as c FROM dataFrameTable
```

In the simplest possible terms, you can use them to manipulate columns in your DataFrames. Let’s walk through some examples on DataFrames to talk about some of the different ways of approaching this problem. The easiest way is just to use the select method and pass in the column names as strings with which you would like to work :  

用最简单的术语来说，您可以使用它们来操作 DataFrame 中的列。 让我们来看一下 DataFrame 上的一些示例，以讨论解决此问题的一些不同方法。 最简单的方法是使用 select 方法并将列名作为您要使用的字符串传递：

```scala
// in Scala
df.select("DEST_COUNTRY_NAME").show(2)
```

```python
# in Python
df.select("DEST_COUNTRY_NAME").show(2)
```

```sql
-- in SQL
SELECT DEST_COUNTRY_NAME FROM dfTable LIMIT 2
```

Giving an output of : 

提供以下输出：

```shell
+-----------------+
|DEST_COUNTRY_NAME|
+-----------------+
|  United States  |
|  United States  |
+-----------------+
```

You can select multiple columns by using the same style of query, just add more column name strings to your select method call: 

您可以使用相同的查询样式选择多个列，只需将更多列名称字符串添加到 select 方法调用中即可：

```scala
// in Scala
df.select("DEST_COUNTRY_NAME", "ORIGIN_COUNTRY_NAME").show(2)
```

```python
# in Python
df.select("DEST_COUNTRY_NAME", "ORIGIN_COUNTRY_NAME").show(2)
```

```sql
-- in SQL
SELECT DEST_COUNTRY_NAME, ORIGIN_COUNTRY_NAME FROM dfTable LIMIT 2
```

Giving an output of:

提供以下输出：

```shell
+-----------------+-------------------+
|DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|
+-----------------+-------------------+
|  United States  |     Romania       |
|  United States  |     Croatia       |
+-----------------+-------------------+
```

As discussed in “Columns and Expressions”, you can refer to columns in a number of different ways; all you need to keep in mind is that you can use them interchangeably: 

如“列和表达式”中所述，您可以通过多种不同的方式引用列。 您需要记住的是可以互换使用它们：

```scala
// in Scala
import org.apache.spark.sql.functions.{expr, col, column}

df.select(
df.col("DEST_COUNTRY_NAME"),
col("DEST_COUNTRY_NAME"),
column("DEST_COUNTRY_NAME"),
'DEST_COUNTRY_NAME,
$"DEST_COUNTRY_NAME",
expr("DEST_COUNTRY_NAME")
).show(2)
```

```python
# in Python
from pyspark.sql.functions import expr, col, column

df.select(
expr("DEST_COUNTRY_NAME"),
col("DEST_COUNTRY_NAME"),
column("DEST_COUNTRY_NAME")
).show(2)
```

One common error is attempting to mix Column objects and strings. For example, the following code will result in a compiler error:

一种常见的错误是尝试混合Column对象和字符串。 例如，以下代码将导致编译器错误：

```scala
df.select(col("DEST_COUNTRY_NAME"), "DEST_COUNTRY_NAME")) 
```

As we’ve seen thus far, expr is the most flexible reference that we can use. It can refer to a plain column or a string manipulation of a column. To illustrate, let’s change the column name, and then change it back by using the AS keyword and then the alias method on the column:

到目前为止，我们已经看到，expr是我们可以使用的最灵活的引用。它可以引用普通列或列的字符串操作。为了说明这一点，让我们更改列名，然后使用AS关键字然后在该列上使用alias方法将其更改回：

```scala
// in Scala
df.select(expr("DEST_COUNTRY_NAME AS destination")).show(2)
```

```python
# in Python
df.select(expr("DEST_COUNTRY_NAME AS destination")).show(2)
```

```sql
-- in SQL
SELECT DEST_COUNTRY_NAME as destination FROM dfTable LIMIT 2
```

This changes the column name to “destination.” You can further manipulate the result of your expression as another expression:

这会将列名更改为“ destination”。您可以进一步将表达式的结果作为另一个表达式来处理：

```scala
// in Scala
df.select(expr("DEST_COUNTRY_NAME as destination").alias("DEST_COUNTRY_NAME"))
.show(2)
```

```python
# in Python
df.select(expr("DEST_COUNTRY_NAME as destination").alias("DEST_COUNTRY_NAME"))\
.show(2) 
```

The preceding operation changes the column name back to its original name. Because select followed by a series of expr is such a common pattern, Spark has a shorthand for doing this efficiently: `selectExpr`. This is probably the most convenient interface for everyday use: 

前面的操作将列名称更改回其原始名称。因为select后跟一系列的expr是一种常见的模式，所以Spark有一个有效执行此操作的简写：selectExpr。这可能是日常使用中最方便的界面：

```scala
// in Scala
df.selectExpr("DEST_COUNTRY_NAME as newColumnName", "DEST_COUNTRY_NAME").show(2)
```

```python
# in Python
df.selectExpr("DEST_COUNTRY_NAME as newColumnName", "DEST_COUNTRY_NAME").show(2)
```

This opens up the true power of Spark. We can treat `selectExpr` as a simple way to build up complex expressions that create new DataFrames. In fact, we can add any valid non-aggregating SQL statement, and as long as the columns resolve, it will be valid! Here’s a simple example that adds a new column `withinCountry` to our DataFrame that specifies whether the destination and origin are the same: 

这打开了Spark的真正力量。我们可以将 selectExpr 视为构建可创建新DataFrame的复杂表达式的简单方法。实际上，我们可以添加任何有效的非聚合SQL语句，并且只要这些列能够解析，它就会有效！这是一个简单的示例，在Country中向我们的DataFrame添加了一个新列，用于指定目的地和起点是否相同：

```python
// in Scala
df.selectExpr(
"*", // include all original columns
"(DEST_COUNTRY_NAME = ORIGIN_COUNTRY_NAME) as withinCountry"
).show(2)
```

```python
# in Python
df.selectExpr(
"*", # all original columns
"(DEST_COUNTRY_NAME = ORIGIN_COUNTRY_NAME) as withinCountry"
).show(2)
```

```sql
-- in SQL
SELECT *, (DEST_COUNTRY_NAME = ORIGIN_COUNTRY_NAME) as withinCountry FROM dfTable LIMIT 2
```

Giving an output of:

提供以下输出：

```scala
+-----------------+-------------------+-----+-------------+
|DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|withinCountry|
+-----------------+-------------------+-----+-------------+
|  United States  |       Romania     |  15 |   false     |
|  United States  |       Croatia     |  1  |   false     |
+-----------------+-------------------+-----+-------------+
```

With select expression, we can also specify aggregations over the entire DataFrame by taking advantage of the functions that we have. These look just like what we have been showing so far: 

使用select表达式，我们还可以利用我们拥有的函数在整个DataFrame上指定聚合，这些看起来就像我们到目前为止所展示的：

```scala
// in Scala
df.selectExpr("avg(count)", "count(distinct(DEST_COUNTRY_NAME))").show(2)
```

```python
# in Python
df.selectExpr("avg(count)", "count(distinct(DEST_COUNTRY_NAME))").show(2)
```

```sql
-- in SQL
SELECT avg(count), count(distinct(DEST_COUNTRY_NAME)) FROM dfTable LIMIT 2
```

Giving an output of :

提供以下输出：

```scala
+-----------+---------------------------------+
| avg(count)|count(DISTINCT DEST_COUNTRY_NAME)|
+-----------+---------------------------------+
|1770.765625|             132                 |
+-----------+---------------------------------+
```

### <font color="#00000">Converting to Spark Types (Literals) 转换为Spark类型（字面量）</font>

Sometimes, we need to pass explicit values into Spark that are just a value (rather than a new column). This might be a constant value or something we’ll need to compare to later on. The way we do this is through *literals*. This is basically a translation from a given programming language’s literal value to one that Spark understands. Literals are expressions and you can use them in the same way: 

有时，我们需要将仅作为值（而不是新列）的显式值传递给Spark。这可能是一个恒定值，或者是以后我们需要比较的值。我们这样做的方法是通过文字。这基本上是从给定编程语言的字面值到Spark可以理解的一种转换。文字是表达式，您可以按照相同的方式使用它们：

```scala
// in Scala
import org.apache.spark.sql.functions.lit
df.select(expr("*"), lit(1).as("One")).show(2)
```

```python
# in Python
from pyspark.sql.functions import lit
df.select(expr("*"), lit(1).alias("One")).show(2)
```

In SQL, literals are just the specific value : 

在SQL中，文字只是特定的值：

```sql
-- in SQL
SELECT *, 1 as One FROM dfTable LIMIT 2
```

Giving an output of:

提供以下输出：

```shell
+-----------------+-------------------+-----+---+
|DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|One|
+-----------------+-------------------+-----+---+
|  United States  |    Romania        | 15  | 1 |
|  United States  |    Croatia        | 1   | 1 |
+-----------------+-------------------+-----+---+
```

This will come up when you might need to check whether a value is greater than some constant or other programmatically created variable. 

当您可能需要检查某个值是否大于某个常量或其他以编程方式创建的变量时，就会出现这种情况。

### <font color="#00000">Adding Columns 添加列</font>

There’s also a more formal way of adding a new column to a DataFrame, and that’s by using the `withColumn` method on our DataFrame. For example, let’s add a column that just adds the number one as a column:

还有一种更正式的方法，可以在DataFrame中添加新列，即使用DataFrame上的withColumn方法。例如，让我们添加一列，将数字一添加为一列：

```scala
// in Scala
df.withColumn("numberOne", lit(1)).show(2)
```

```python
# in Python
df.withColumn("numberOne", lit(1)).show(2)
```

```sql
-- in SQL
SELECT *, 1 as numberOne FROM dfTable LIMIT 2
```

Giving an output of : 

提供以下输出：

```shell
+-----------------+-------------------+-----+---------+
|DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|numberOne|
+-----------------+-------------------+-----+---------+
|  United States  |      Romania      | 15  |   1     |
|  United States  |      Croatia      | 1   |   1     |
+-----------------+-------------------+-----+---------+
```

Let’s do something a bit more interesting and make it an actual expression. In the next example, we’ll set a Boolean flag for when the origin country is the same as the destination country:

让我们做一些更有趣的事情，并将其变为实际的表达方式。在下一个示例中，我们将为原籍国与目的地国相同时设置一个布尔值标志：

```scala
// in Scala
df.withColumn("withinCountry", expr("ORIGIN_COUNTRY_NAME == DEST_COUNTRY_NAME"))
.show(2)
```

```scala
# in Python
df.withColumn("withinCountry", expr("ORIGIN_COUNTRY_NAME == DEST_COUNTRY_NAME"))\
.show(2)
```

Notice that the withColumn function takes two arguments: the column name and the expression that will create the value for that given row in the DataFrame. Interestingly, we can also rename a column this way. The SQL syntax is the same as we had previously, so we can omit it in this example:

请注意，withColumn函数带有两个参数：列名和将为DataFrame中给定行创建值的表达式。有趣的是，我们也可以用这种方式重命名列。 SQL语法与之前的语法相同，因此在此示例中可以省略它：

```scala
df.withColumn("Destination", expr("DEST_COUNTRY_NAME")).columns
```

Resulting in:

导致：

```sql
... DEST_COUNTRY_NAME, ORIGIN_COUNTRY_NAME, count, Destination
```

### <font color="#00000">Renaming Columns 重命名列</font>

Although we can rename a column in the manner that we just described, another alternative is to use the `withColumnRenamed` method. This will rename the column with the name of the string in the first argument to the string in the second argument:

尽管我们可以按照刚才描述的方式重命名列，但是另一种替代方法是使用 `withColumnRenamed` 方法。这会将第一个参数中的字符串名称重命名为第二个参数中的字符串名称：

```scala
// in Scala
df.withColumnRenamed("DEST_COUNTRY_NAME", "dest").columns
```

```python
# in Python
df.withColumnRenamed("DEST_COUNTRY_NAME", "dest").columns
```

```shell
... dest, ORIGIN_COUNTRY_NAME, count
```

### <font color="#00000">Reserved Characters and Keywords 保留字符和关键字</font>

One thing that you might come across is reserved characters like spaces or dashes in column names. Handling these means escaping column names appropriately. In Spark, we do this by using backtick (`) characters. Let’s use  withColumn, which you just learned about to create a column with reserved characters. We’ll show two examples—in the one shown here, we don’t need escape characters, but in the next one, we do:

您可能遇到的一件事是保留的字符，例如列名中的空格或破折号。处理这些意味着适当地转义了列名。在Spark中，我们通过使用反引号（`）字符来做到这一点。让我们使用 withColumn，您刚刚学习了如何创建带有保留字符的列。我们将显示两个示例，在这里显示的一个示例中，我们不需要转义符，但是在下一个示例中，我们这样做：

```scala
// in Scala
import org.apache.spark.sql.functions.expr
val dfWithLongColName = df.withColumn(
"This Long Column-Name",
expr("ORIGIN_COUNTRY_NAME"))
```

```python
# in Python
dfWithLongColName = df.withColumn("This Long Column-Name", expr("ORIGIN_COUNTRY_NAME"))
```

We don’t need escape characters here because the first argument to withColumn is just a string for the new column name. In this example, however, we need to use backticks because we’re referencing a column in an expression:

我们在这里不需要转义字符，因为 withColumn 的第一个参数只是新列名称的字符串。 但是，在此示例中，我们需要使用反引号，因为我们要引用表达式中的列：

```scala
// in Scala
dfWithLongColName.selectExpr(
"`This Long Column-Name`",
"`This Long Column-Name` as `new col`")
.show(2)
```

```python
# in Python
dfWithLongColName.selectExpr(
"`This Long Column-Name`",
"`This Long Column-Name` as `new col`")\
.show(2)
dfWithLongColName.createOrReplaceTempView("dfTableLong")
```

```sql
-- in SQL
SELECT This Long Column-Name, This Long Column-Name as new col
FROM dfTableLong LIMIT 2
```

We can refer to columns with reserved characters (and not escape them) if we’re doing an explicit string-to-column reference, which is interpreted as a literal instead of an expression. We only need to escape expressions that use reserved characters or keywords. The following two examples both result in the same DataFrame:

如果我们正在执行显式的字符串到列引用，则可以引用带有保留字符的列（而不是对它们进行转义），该引用被解释为文字而不是表达式。 我们只需要转义使用保留字符或关键字的表达式。 以下两个示例均导致相同的DataFrame：

```scala
// in Scala
dfWithLongColName.select(col("This Long Column-Name")).columns
```

```python
# in Python
dfWithLongColName.select(expr("`This Long Column-Name`")).columns
```

### <font color="#00000">Case Sensitivity 区分大小写</font>

By default Spark is case insensitive; however, you can make Spark case sensitive by setting the configuration:

默认情况下，Spark不区分大小写；但是，可以通过设置配置使Spark区分大小写：

```sql
-- in SQL
set spark.sql.caseSensitive true
```

### <font color="#00000">Removing Columns 删除列</font>

Now that we’ve created this column, let’s take a look at how we can remove columns from DataFrames. You likely already noticed that we can do this by using select. However, there is also a dedicated method called drop:

现在我们已经创建了此列，让我们看一下如何从DataFrames中删除列。您可能已经注意到我们可以通过使用select来做到这一点。但是，还有一个专用的方法称为drop：

```sql
df.drop("ORIGIN_COUNTRY_NAME").columns
```

We can drop multiple columns by passing in multiple columns as arguments:

我们可以通过传递多个列作为参数来删除多个列：

```scala
dfWithLongColName.drop("ORIGIN_COUNTRY_NAME", "DEST_COUNTRY_NAME")
```

### <font color="#00000">Changing a Column’s Type (cast) 更改列的类型（广播）</font>

Sometimes, we might need to convert from one type to another; for example, if we have a set of StringType that should be integers. We can convert columns from one type to another by casting the column from one type to another. For instance, let’s convert our count column from an integer to a type Long:

有时，我们可能需要从一种类型转换为另一种类型。例如，如果我们有一组应该为整数的StringType。通过将列从一种类型转换为另一种类型，我们可以将列从一种类型转换为另一种类型。例如，让我们将count列从整数转换为Long类型：

```scala
df.withColumn("count2", col("count").cast("long"))
```

```sql
-- in SQL
SELECT *, cast(count as long) AS count2 FROM dfTable
```

### <font color="#00000">Filtering Rows</font>

To filter rows, we create an expression that evaluates to true or false. You then filter out the rows with an expression that is equal to false. The most common way to do this with DataFrames is to create either an expression as a String or build an expression by using a set of column manipulations. There are two methods to perform this operation: you can use where or filter and they both will perform the same operation and accept the same argument types when used with DataFrames. We will stick to where because of its familiarity to SQL; however, filter is valid as well.

为了过滤行，我们创建一个表达式，其结果为true或false。 然后，使用等于false的表达式过滤掉行。 使用DataFrames执行此操作的最常见方法是将表达式创建为String或通过使用一组列操作来构建表达式。 有两种方法可以执行此操作：您可以使用where或filter，它们与DataFrames一起使用时将执行相同的操作并接受相同的参数类型。 由于对SQL的熟悉，我们将坚持到底。 但是，过滤器也有效。

---

<p><center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong></font></center></P>
When using the Dataset API from either Scala or Java, filter also accepts an arbitrary function that Spark will apply to each record in the Dataset. See Chapter 11 for more information. 

当从Scala或Java使用Dataset API时，filter还接受Spark应用于 Dataset 中每个记录的任意函数。有关更多信息，请参见第11章。

---

The following filters are equivalent, and the results are the same in Scala and Python:

以下过滤器是等效的，并且在Scala和Python中结果是相同的：

```scala
df.filter(col("count") < 2).show(2)
df.where("count < 2").show(2)
```

```sql
-- in SQL
SELECT * FROM dfTable WHERE count < 2 LIMIT 2
```

Giving an output of:

给出以下输出：

```shell
+-----------------+-------------------+-----+
|DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|
+-----------------+-------------------+-----+
|  United States  |      Croatia      |  1  |
|  United States  |     Singapore     |  1  |
+-----------------+-------------------+-----+
```

Instinctually, you might want to put multiple filters into the same expression. Although this is possible, it is not always useful, because Spark automatically performs all filtering operations at the same time regardless of the filter ordering. This means that if you want to specify multiple AND filters, just chain them sequentially and let Spark handle the rest:

本能地，您可能希望将多个过滤器放入同一表达式中。 尽管这是可行的，但并不总是有用的，因为Spark会自动同时同时执行所有过滤操作，而不考虑过滤器的顺序。 这意味着，如果您要指定多个AND过滤器，只需按顺序将它们链接起来，然后让Spark处理其余的过滤器：

```scala
// in Scala
df.where(col("count") < 2).where(col("ORIGIN_COUNTRY_NAME") =!= "Croatia")
.show(2)
```

```python
# in Python
df.where(col("count") < 2).where(col("ORIGIN_COUNTRY_NAME") != "Croatia")\
.show(2)
```

```sql
-- in SQL
SELECT * FROM dfTable WHERE count < 2 AND ORIGIN_COUNTRY_NAME != "Croatia"
LIMIT 2
```

Giving an output of:

给出以下输出：

```shell
+-----------------+-------------------+-----+
|DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|
+-----------------+-------------------+-----+
|  United States  |    Singapore      |  1  |
|     Moldova     |     United States |  1  |
+-----------------+-------------------+-----+
```

### <font color="#00000">Getting Unique Rows 获取非重复的行</font>

A very common use case is to extract the unique or distinct values in a DataFrame. These values can be in one or more columns. The way we do this is by using the distinct method on a DataFrame, which allows us to deduplicate any rows that are in that DataFrame. For instance, let’s get the unique origins in our dataset. This, of course, is a transformation that will return a new DataFrame with only unique rows:

一个非常常见的用例是在DataFrame中提取唯一或不同的值。 这些值可以在一列或多列中。 我们这样做的方法是对DataFrame使用不同的方法，该方法使我们能够对DataFrame中的所有行进行重复数据删除。 例如，让我们获取数据集中的唯一来源。 当然，这是一个转换，将返回仅具有唯一行的新DataFrame：

```scala
// in Scala
df.select("ORIGIN_COUNTRY_NAME", "DEST_COUNTRY_NAME").distinct().count()
```

```python
# in Python
df.select("ORIGIN_COUNTRY_NAME", "DEST_COUNTRY_NAME").distinct().count()
```

```sql
-- in SQL
SELECT COUNT(DISTINCT(ORIGIN_COUNTRY_NAME, DEST_COUNTRY_NAME)) FROM dfTable
```
Results in 256.

结果是256。

```scala
// in Scala
df.select("ORIGIN_COUNTRY_NAME").distinct().count()
```

```python
# in Python
df.select("ORIGIN_COUNTRY_NAME").distinct().count()
```

```sql
-- in SQL
SELECT COUNT(DISTINCT ORIGIN_COUNTRY_NAME) FROM dfTable
```

Results in 125.

结果是125。

### <font color="#00000">Random Samples 随机抽样</font>

Sometimes, you might just want to sample some random records from your DataFrame. You can do this by using the sample method on a DataFrame, which makes it possible for you to specify a fraction of rows to extract from a DataFrame and whether you’d like to sample with or without replacement:

有时，您可能只想从DataFrame中抽取一些随机记录。 您可以通过在DataFrame上使用sample方法来执行此操作，这使您可以指定要从DataFrame提取的行的一部分，以及是否要替换或不替换地进行采样：

```scala
val seed = 5
val withReplacement = false
val fraction = 0.5
df.sample(withReplacement, fraction, seed).count()
```

```python
# in Python
seed = 5
withReplacement = False
fraction = 0.5
df.sample(withReplacement, fraction, seed).count()
```

Giving an output of 126.

给出输出：126。

### <font color="#00000">Random Splits 随机分割</font>

Random splits can be helpful when you need to break up your DataFrame into a random “splits” of the original DataFrame. This is often used with machine learning algorithms to create training, validation, and test sets. In this next example, we’ll split our DataFrame into two different DataFrames by setting the weights by which we will split the DataFrame (these are the arguments to the function). Because this method is designed to be randomized, we will also specify a seed (just replace seed with a number of your choosing in the code block). It’s important to note that if you don’t specify a proportion for each DataFrame that adds up to one, they will be normalized so that they do:

当您需要将DataFrame分解成原始DataFrame的随机“拆分”时，随机拆分会很有帮助。这通常与机器学习算法一起使用以创建训练，验证和测试集。在下一个示例中，我们将通过设置将DataFrame分割的权重（这些是函数的参数），将DataFrame分为两个不同的DataFrame。因为此方法是随机设计的，所以我们还将指定一个种子（只需在代码块中用您选择的数量替换种子）。重要的是要注意，如果您没有为每个总计为1的DataFrame指定比例，则将它们标准化，这样就可以了：

```scala
// in Scala
val dataFrames = df.randomSplit(Array(0.25, 0.75), seed)
dataFrames(0).count() > dataFrames(1).count() // False
```

```python
# in Python
dataFrames = df.randomSplit([0.25, 0.75], seed)
dataFrames[0].count() > dataFrames[1].count() # False
```

### <font color="#00000">Concatenating and Appending Rows (Union) 串联和附加行（联合）</font>

As you learned in the previous section, DataFrames are immutable. This means users cannot append to DataFrames because that would be changing it. To append to a DataFrame, you must union the original DataFrame along with the new DataFrame. This just concatenates the two DataFrames. To union two DataFrames, you must be sure that they have the same schema and number of columns; otherwise, the union will fail.

如上一节所述，DataFrame是不可变的。这意味着用户无法附加到DataFrame，因为这将对其进行更改。要附加到DataFrame，必须将原始DataFrame与新DataFrame合并在一起。这只是连接两个DataFrame。要合并两个DataFrame，必须确保它们具有相同的模式和列数。否则，联合将失败。

---

<p><center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong></font></center></p>
Unions are currently performed based on location, not on the schema. This means that columns will not automatically line up the way you think they might. 

当前，联合是基于位置而不是基于模式执行的。这意味着列将不会自动按照您认为的方式排列。

---

```scala
// in Scala
import org.apache.spark.sql.Row
val schema = df.schema
val newRows = Seq(
Row("New Country", "Other Country", 5L),
Row("New Country 2", "Other Country 3", 1L)) 
val parallelizedRows = spark.sparkContext.parallelize(newRows)
val newDF = spark.createDataFrame(parallelizedRows, schema)
df.union(newDF)
.where("count = 1")
.where($"ORIGIN_COUNTRY_NAME" =!= "United States")
.show() // get all of them and we'll see our new rows at the end
```

```python
# in Python
from pyspark.sql import Row
schema = df.schema
newRows = [
Row("New Country", "Other Country", 5L),
Row("New Country 2", "Other Country 3", 1L)
] 
parallelizedRows = spark.sparkContext.parallelize(newRows)
newDF = spark.createDataFrame(parallelizedRows, schema)
df.union(newDF)\
.where("count = 1")\
.where(col("ORIGIN_COUNTRY_NAME") != "United States")\
.show()
```

In Scala, you must use the operator so that you don’t just compare the unevaluated column expression to a string but instead to the evaluated one: 

在Scala中，您必须使用  =!=  运算符，以便您不只是将未求值的列表达式与字符串进行比较，而是与已求值的表达式进行比较：

Giving the output of:

给出输出：

```shell
+-----------------+-------------------+-----+
|DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|
+-----------------+-------------------+-----+
|  United States  |      Croatia      |  1  |
+-----------------+-------------------+-----+
```

As expected, you’ll need to use this new DataFrame reference in order to refer to the DataFrame with the newly appended rows. A common way to do this is to make the DataFrame into a view or register it as a table so that you can reference it more dynamically in your code.

如预期的那样，您将需要使用这个新的 DataFrame 引用，以引用带有新添加的行的 DataFrame。 一种常见的方法是将 DataFrame 放入视图或将其注册为表，以便您可以在代码中更动态地引用它。

### <font color="#00000">Sorting Rows 排序行</font>

When we sort the values in a DataFrame, we always want to sort with either the largest or smallest values at the top of a DataFrame. There are two equivalent operations to do this sort and orderBy that work the exact same way. They accept both column expressions and strings as well as multiple columns. The default is to sort in ascending order: 

当我们对一个DataFrame中的值进行排序时，我们总是希望使用DataFrame顶部的最大值或最小值进行排序。 有两种等效的操作可以执行完全相同的排序和 orderBy 操作。 它们接受列表达式和字符串以及多列。 默认是按升序排序：

```scala
// in Scala
df.sort("count").show(5)
df.orderBy("count", "DEST_COUNTRY_NAME").show(5)
df.orderBy(col("count"), col("DEST_COUNTRY_NAME")).show(5)
```

```python
# in Python
df.sort("count").show(5)
df.orderBy("count", "DEST_COUNTRY_NAME").show(5)
df.orderBy(col("count"), col("DEST_COUNTRY_NAME")).show(5)
```

To more explicitly specify sort direction, you need to use the `asc` and `desc` functions if operating on a column. These allow you to specify the order in which a given column should be sorted: 

要更明确地指定排序方向，如果对列进行操作，则需要使用 `asc` 和 `desc` 函数。 这些允许您指定给定列的排序顺序：

```scala
// in Scala
import org.apache.spark.sql.functions.{desc, asc}
df.orderBy(expr("count desc")).show(2)
df.orderBy(desc("count"), asc("DEST_COUNTRY_NAME")).show(2)
```

```python
# in Python
from pyspark.sql.functions import desc, asc
df.orderBy(expr("count desc")).show(2)
df.orderBy(col("count").desc(), col("DEST_COUNTRY_NAME").asc()).show(2)
```

```sql
-- in SQL
SELECT * FROM dfTable ORDER BY count DESC, DEST_COUNTRY_NAME ASC LIMIT 2
```

An advanced tip is to use `asc_nulls_first`, `desc_nulls_first`, `asc_nulls_last`, or `desc_nulls_last` to specify where you would like your null values to appear in an ordered  DataFrame.

一个高级技巧是使用 `asc_nulls_first`，`desc_nulls_first`，`asc_nulls_last` 或 `desc_nulls_last` 指定您希望空值出现在有序DataFrame中的位置。

For optimization purposes, it’s sometimes advisable to sort within each partition before another set of transformations. You can use the `sortWithinPartitions` method to do this: 

出于优化目的，有时建议在每个分区内进行另一组转换之前进行排序。 您可以使用 `sortWithinPartitions` 方法执行此操作：

```scala
// in Scala
spark.read.format("json").load("/data/flight-data/json/*-summary.json")
.sortWithinPartitions("count")
```

```python
# in Python
spark.read.format("json").load("/data/flight-data/json/*-summary.json")\
.sortWithinPartitions("count")
```

We will discuss this more when we look at tuning and optimization in Part III.

当我们在第三部分中讨论调优和优化时，我们将对此进行更多讨论。

### <font color="#00000">Limit 限制</font>

Oftentimes, you might want to restrict what you extract from a DataFrame; for example, you might want just the top ten of some DataFrame. You can do this by using the limit method: 

通常，您可能希望限制从DataFrame中提取的内容； 例如，您可能只需要某些DataFrame的前十名。 您可以通过使用limit方法来做到这一点：

```scala
// in Scala
df.limit(5).show()
```

```python
# in Python
df.limit(5).show()
```

```sql
-- in SQL
SELECT * FROM dfTable LIMIT 6
```

```scala
// in Scala
df.orderBy(expr("count desc")).limit(6).show()
```

```python
# in Python
df.orderBy(expr("count desc")).limit(6).show()
```

```sql
-- in SQL
SELECT * FROM dfTable ORDER BY count desc LIMIT 6
```

### <font color="#00000">Repartition and Coalesce 分区与合并</font>

Another important optimization opportunity is to partition the data according to some frequently filtered columns, which control the physical layout of data across the cluster including the partitioning scheme and the number of partitions. Repartition will incur a full shuffle of the data, regardless of whether one is necessary. This means that you should typically only repartition when the future number of partitions is greater than your current number of partitions or when you are looking to partition by a set of columns: 

另一个重要的优化机会是根据一些频繁过滤的列对数据进行分区，这些列控制整个群集中数据的物理布局，包括分区方案和分区数。 无论是否需要重新分区，重新分区都会导致数据的完全数据再分配（shuffle）。 这意味着您通常仅应在将来的分区数大于当前的分区数时或在按一组列进行分区时重新分区：

```scala
// in Scala
df.rdd.getNumPartitions // 1
```

```python
# in Python
df.rdd.getNumPartitions() # 1
```

```scala
// in Scala
df.repartition(5)
```

```python
# in Python
df.repartition(5)
```

If you know that you’re going to be filtering by a certain column often, it can be worth repartitioning based on that column: 

如果您知道经常要按某个列进行过滤，则值得根据该列进行重新分区：

```python
// in Scala
df.repartition(col("DEST_COUNTRY_NAME"))
```

```python
# in Python
df.repartition(col("DEST_COUNTRY_NAME"))
```

You can optionally specify the number of partitions you would like, too: 

您也可以选择指定所需的分区数：

```scala
// in Scala
df.repartition(5, col("DEST_COUNTRY_NAME"))
```

```python
# in Python
df.repartition(5, col("DEST_COUNTRY_NAME"))
```

Coalesce, on the other hand, will not incur a full shuffle and will try to combine partitions. This operation will shuffle your data into five partitions based on the destination country name, and then coalesce them (without a full shuffle): 

另一方面，合并将不会引起全量数据再分配，并会尝试合并分区。 此操作将根据目标国家/地区名称将数据随机分为五个分区，然后将它们合并（不进行全量数据再分配）：

```scala
// in Scala
df.repartition(5, col("DEST_COUNTRY_NAME")).coalesce(2)
```

```python
# in Python
df.repartition(5, col("DEST_COUNTRY_NAME")).coalesce(2)
```

### <font color="#00000">Collecting Rows to the Driver 将行收集到驱动程序中</font>

As discussed in previous chapters, Spark maintains the state of the cluster in the driver. There are times when you’ll want to collect some of your data to the driver in order to manipulate it on your local machine.

如前几章所述，Spark在驱动程序中维护集群的状态。有时候，您希望将一些数据收集到驱动程序以便在本地计算机上进行操作。

Thus far, we did not explicitly define this operation. However, we used several different methods for doing so that are effectively all the same. collect gets all data from the entire DataFrame, take selects the first N rows, and show prints out a number of rows nicely. 

到目前为止，我们尚未明确定义此操作。但是，我们使用了几种不同的方法来进行操作，这些方法实际上都是相同的。 collect从整个DataFrame中获取所有数据，take选择前N行，然后show很好地打印出多行。


```scala
// in Scala
val collectDF = df.limit(10)
collectDF.take(5) // take works with an Integer count
collectDF.show() // this prints it out nicely
collectDF.show(5, false)
collectDF.collect()
```

```python
# in Python
collectDF = df.limit(10)
collectDF.take(5) # take works with an Integer count
collectDF.show() # this prints it out nicely
collectDF.show(5, False)
collectDF.collect()
```

There’s an additional way of collecting rows to the driver in order to iterate over the entire dataset. The method `toLocalIterator` collects partitions to the driver as an iterator. This method allows you to iterate over the entire dataset partition-by-partition in a serial manner: 

还有另一种收集行到驱动程序的方法，以便遍历整个数据集。方法 `toLocalIterator` 将分区作为迭代器收集到驱动程序。此方法允许您以串行方式逐分区遍历整个数据集：

```scala
collectDF.toLocalIterator()
```

---

<p><center><font face="constant-width" color="#c67171" size=3><strong>WARNING 警告</strong></font></center></p>
Any collection of data to the driver can be a very expensive operation! If you have a large dataset and call collect, you can crash the driver. If you use `toLocalIterator` and have very large partitions, you can easily crash the driver node and lose the state of your application. This is also expensive because we can operate on a one-by-one basis, instead of running computation in parallel.

向驱动程序收集任何数据都是非常昂贵的操作！如果您有一个很大的数据集并调用 `collect`，则可能会使驱动程序崩溃。如果使用 `toLocalIterator` 并具有很大的分区，则很容易使驱动程序节点崩溃并失去应用程序的状态。这也很昂贵，因为我们可以一对一地操作，而不是并行运行计算。

---

## <font color="#9a161a">Conclusion 总结</font>

This chapter covered basic operations on DataFrames. You learned the simple concepts and tools that you will need to be successful with Spark DataFrames. Chapter 6 covers in much greater detail all of the different ways in which you can manipulate the data in those DataFrames. 

本章介绍了DataFrame的基本操作。您了解了Spark DataFrame成功所需的简单概念和工具。第6章更加详细地介绍了可以在这些DataFrame中操作数据的所有不同方式。



