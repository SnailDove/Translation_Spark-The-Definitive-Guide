---
title: 翻译 Chapter 6. Working with Different Types of Data
date: 2019-08-05
copyright: true
categories: English
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 6. Working with Different Types of Data 处理不同类型的数据
<center><strong>译者</strong>：<u><a style="color:#0879e3" href="https://snaildove.github.io">https://snaildove.github.io</a></u></center>

Chapter 5 presented basic DataFrame concepts and abstractions. This chapter covers building expressions, which are the bread and butter of Spark’s structured operations. We also review working with a variety of different kinds of data, including the following:

第5章介绍了基本的DataFrame概念和抽象。本章涵盖了构建表达式，它们是Spark结构化操作的基础。我们还将回顾使用各种不同类型的数据的工作，包括以下内容：

- Booleans
- Numbers
- Strings
- Dates and timestamps
- Handling null 处理空
- Complex types 复杂类型
- User-defined functions 用户定义的函数

## <font color="#9a161a">Where to Look for APIs 在何处查找API </font>

Before we begin, it’s worth explaining where you as a user should look for transformations. Spark is a growing project, and any book (including this one) is a snapshot in time. One of our priorities in this book is to teach where, as of this writing, you should look to find functions to transform your data. Following are the key places to look : 

在开始之前，值得解释一下您作为用户应该在哪里寻求转换。 Spark是一个正在发展的项目，任何书籍（包括本书）都是及时的快照。在本书中，我们的优先重点之一是自本书开始教您应该在哪里寻找用于转换数据的函数。 以下是查找的主要地方：

**DataFrame (Dataset) Methods**

This is actually a bit of a trick because a DataFrame is just a Dataset of Row types, so you’ll actually end up looking at the Dataset methods, which are available at this [link](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Dataset).

这实际上是一个技巧，因为DataFrame只是行类型的数据集，因此您实际上最终将查看Dataset方法，该方法在此[链接](http://spark.apache.org /docs/latest/api/scala/index.html#org.apache.spark.sql.Dataset)可以得到。

Dataset submodules like [DataFrameStatFunctions](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrameStatFunctions) and [DataFrameNaFunctions](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrameNaFunctions) have more methods that solve specific sets of problems. [DataFrameStatFunctions](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrameStatFunctions), for example, holds a variety of statistically related functions, whereas [DataFrameNaFunctions](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrameNaFunctions) refers to functions that are relevant when working with null data.

Dataset 子模块，例如 [DataFrameStatFunctions](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrameStatFunctions) 和 [DataFrameNaFunctions](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrameNaFunctions) 具有更多解决特定问题集的方法。例如，[DataFrameStatFunctions](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrameStatFunctions) 拥有各种与统计相关的功能，而 [DataFrameNaFunctions](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrameNaFunctions) 指的是在处理空数据时相关的函数。

**Column Methods 列方法**

These were introduced for the most part in Chapter 5. They hold a variety of general column related methods like `alias` or `contains`. You can find [the API Reference for Column methods here](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Column).

这些在第5章中进行了大部分介绍。它们具有与列相关的各种常规方法，如“ alias”或“ contains”。您可以在[这里](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Column)找到Column方法的API参考链接。

`org.apache.spark.sql.functions` contains a variety of functions for a range of different data types. Often, you’ll see the entire package imported because they are used so frequently. You can find [SQL and DataFrame functions here](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.functions$).

`org.apache.spark.sql.functions` 包含用于各种不同数据类型的各种功能。通常，您会看到导入的整个程序包，因为它们是如此频繁地使用。您可以在[此处找到SQL和DataFrame函数](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.functions$)。

Now this may feel a bit overwhelming but have no fear, the majority of these functions are ones that you will find in SQL and analytics systems. All of these tools exist to achieve one purpose, to transform rows of data in one format or structure to another. This might create more rows or reduce the number of rows available. To begin, let’s read in the DataFrame that we’ll be using for this analysis: 

现在，这可能会让人感到有些不知所措，但请放心，这些功能大多数都是您可以在SQL和分析系统中找到的。存在所有这些工具以实现一个目的，即将一种格式或结构的数据行转换为另一种格式或结构。这可能会创建更多的行或减少可用的行数。首先，让我们阅读将用于此分析的 DataFrame ：
```python
// in Scala
val df = spark.read.format("csv")
.option("header", "true")
.option("inferSchema", "true")
.load("/data/retail-data/by-day/2010-12-01.csv")
df.printSchema()
df.createOrReplaceTempView("dfTable")
```

```python
# in Python
df = spark.read.format("csv")\
.option("header", "true")\
.option("inferSchema", "true")\
.load("/data/retail-data/by-day/2010-12-01.csv")
df.printSchema()
df.createOrReplaceTempView("dfTable") 
```

Here’s the result of the schema and a small sample of the data:

这是模式的结果和一小部分数据示例：

```shell
root

|-- InvoiceNo: string (nullable = true)
|-- StockCode: string (nullable = true)
|-- Description: string (nullable = true)
|-- Quantity: integer (nullable = true)
|-- InvoiceDate: timestamp (nullable = true)
|-- UnitPrice: double (nullable = true)
|-- CustomerID: double (nullable = true)
|-- Country: string (nullable = true)

+---------+---------+--------------------+--------+-------------------+---- 
|InvoiceNo|StockCode|      Description   |Quantity|    InvoiceDate    |Unit 
+---------+---------+--------------------+--------+-------------------+---- 
| 536365  | 85123A  |WHITE HANGING HEA...|    6   |2010-12-01 08:26:00| ...
| 536365  | 71053   | WHITE METAL LANTERN|    6   |2010-12-01 08:26:00| ...
...
| 536367  |  21755  |LOVE BUILDING BLO...|    3   |2010-12-01 08:34:00| ...
| 536367  |  21777  |RECIPE BOX WITH M...|    4   |2010-12-01 08:34:00| ...
+---------+---------+--------------------+--------+-------------------+----  
```

## <font color="#9a161a">Converting to Spark Types 转换为Spark类型</font>

One thing you’ll see us do throughout this chapter is convert native types to Spark types. We do this by using the first function that we introduce here, the `lit` function. This function converts a type in another language to its corresponding Spark representation. Here’s how we can convert a couple of different kinds of Scala and Python values to their respective Spark types:

在本章中，您将看到我们要做的一件事是将本地类型转换为Spark类型。我们通过使用此处介绍的第一个函数 “`lit`” 函数来实现此目的。此函数将另一种语言的类型转换为其对应的Spark表示形式。我们将如何将几种不同的Scala和Python值转换为各自的Spark类型：

```scala
// in Scala
import org.apache.spark.sql.functions.lit
df.select(lit(5), lit("five"), lit(5.0))
```

```python
# in Python
from pyspark.sql.functions import lit
df.select(lit(5), lit("five"), lit(5.0))
```
There’s no equivalent function necessary in SQL, so we can use the values directly:

SQL中没有等效的功能，因此我们可以直接使用这些值：

```sql
-- in SQL
SELECT 5, "five", 5.0 
```

## <font color="#9a161a">Working with Booleans 使用布尔值</font>

Booleans are essential when it comes to data analysis because they are the foundation for all filtering. Boolean statements consist of four elements: and, or, true, and false. We use these simple structures to build logical statements that evaluate to either true or false. These statements are often used as conditional requirements for when a row of data must either pass the test (evaluate to true) or else it will be filtered out.

在数据分析中，布尔是必不可少的，因为它们是所有过滤的基础。布尔语句由四个元素组成：and, or, true, 和 false。我们使用这些简单的结构来构建评估为true或false的逻辑语句。当一行数据必须通过测试（评估为true）或将其过滤掉时，这些语句通常用作条件要求。

Let’s use our retail dataset to explore working with Booleans. We can specify equality as well as less-than or greater-than:

让我们使用零售数据集探索使用布尔值的方法。我们可以指定相等以及小于或大于：
```scala
// in Scala
import org.apache.spark.sql.functions.col
df.where(col("InvoiceNo").equalTo(536365))
.select("InvoiceNo", "Description")
.show(5, false)
```

---

<p><center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong></font></center></p>
Scala has some particular semantics regarding the use of == and ===. In Spark, if you want to filter by equality you should use === (equal) or =!= (not equal). You can also use the not function and the equal To method. 

Scala对于==和===的使用具有一些特殊的语义。 在Spark中，如果要按相等过滤，则应使用 ===（等于）或 =!= （不等于）。 您还可以使用not函数和equal To方法。

---

```scala
// in Scala
import org.apache.spark.sql.functions.col
df.where(col("InvoiceNo") === 536365)
.select("InvoiceNo", "Description")
.show(5, false)
```

Python keeps a more conventional notation :

Python保留了一个更常规的符号：

```python
# in Python
from pyspark.sql.functions import col
df.where(col("InvoiceNo") != 536365)\
.select("InvoiceNo", "Description")\
.show(5, False)
```

```shell
+---------+-----------------------------+
|InvoiceNo|       Description           |
+---------+-----------------------------+
|  536366 |    HAND WARMER UNION JACK   |
...
|  536367 |   POPPY'S PLAYHOUSE KITCHEN |
+---------+-----------------------------+
```

Another option—and probably the cleanest—is to specify the predicate as an expression in a string. This is valid for Python or Scala. Note that this also gives you access to another way of expressing “does not equal”: 

另一个选择（可能是最简洁的选择）是将谓词指定为字符串中的表达式。 这对Python或Scala有效。 请注意，这还使您可以使用另一种表示“不相等”的方式：

```scala
df.where("InvoiceNo = 536365").show(5, false)
```

```python
df.where("InvoiceNo <> 536365").show(5, false)
```

We mentioned that you can specify Boolean expressions with multiple parts when you use and or or. In Spark, you should always chain together and filters as a sequential filter.

我们提到过，当您使用and或or时，可以指定包含多个部分的布尔表达式。在Spark中，您应始终链接在一起并将过滤器作为顺序过滤器。

The reason for this is that even if Boolean statements are expressed serially (one after the other), Spark will flatten all of these filters into one statement and perform the filter at the same time, creating the and statement for us. Although you can specify your statements explicitly by using and if you like, they’re often easier to understand and to read if you specify them serially. or statements need to be specified in the same statement: 

原因是，即使布尔语句以串行方式（一个接一个地表达），Spark也会将所有这些过滤器展平为一个语句并同时执行过滤器，从而为我们创建了and语句。尽管您可以使用和根据需要明确指定语句，但是如果您依次指定它们，通常更易于理解和阅读。或需要在同一条语句中指定的语句：
```scala
// in Scala
val priceFilter = col("UnitPrice") > 600
val descripFilter = col("Description").contains("POSTAGE")
df.where(col("StockCode").isin("DOT")).where(priceFilter.or(descripFilter))
.show()
```

```python
# in Python
from pyspark.sql.functions import instr
priceFilter = col("UnitPrice") > 600
descripFilter = instr(df.Description, "POSTAGE") >= 1
df.where(df.StockCode.isin("DOT")).where(priceFilter | descripFilter).show()
```

```sql
-- in SQL
SELECT * FROM dfTable WHERE StockCode in ("DOT") AND(UnitPrice > 600 OR
instr(Description, "POSTAGE") >= 1)
```

```shell
+---------+---------+--------------+--------+-------------------+---------+...
|InvoiceNo|StockCode| Description  |Quantity|     InvoiceDate   |UnitPrice|...
+---------+---------+--------------+--------+-------------------+---------+...
|  536544 |   DOT   |DOTCOM POSTAGE|    1   |2010-12-01 14:32:00| 569.77  |...
|  536592 |   DOT   |DOTCOM POSTAGE|    1   |2010-12-01 17:06:00| 607.49  |...
+---------+---------+--------------+--------+-------------------+---------+...
```

Boolean expressions are not just reserved to filters. To filter a DataFrame, you can also just specify a Boolean column: 

布尔表达式不仅保留给过滤器。 要过滤DataFrame，您还可以仅指定一个布尔列：

```scala
// in Scala
val DOTCodeFilter = col("StockCode") === "DOT"
val priceFilter = col("UnitPrice") > 600
val descripFilter = col("Description").contains("POSTAGE")
df.withColumn("isExpensive", DOTCodeFilter.and(priceFilter.or(descripFilter)))
.where("isExpensive")
.select("unitPrice", "isExpensive").show(5)
```

```python
# in Python
from pyspark.sql.functions import instr
DOTCodeFilter = col("StockCode") == "DOT"
priceFilter = col("UnitPrice") > 600
descripFilter = instr(col("Description"), "POSTAGE") >= 1
df.withColumn("isExpensive", DOTCodeFilter & (priceFilter | descripFilter))\
.where("isExpensive")\
.select("unitPrice", "isExpensive").show(5)
```

```sql
-- in SQL
SELECT UnitPrice, (StockCode = 'DOT' AND
(UnitPrice > 600 OR instr(Description, "POSTAGE") >= 1)) as isExpensive
FROM dfTable
WHERE (StockCode = 'DOT' AND
(UnitPrice > 600 OR instr(Description, "POSTAGE") >= 1))
```

Notice how we did not need to specify our filter as an expression and how we could use a column name without any extra work.

请注意，我们如何不需要将过滤器指定为表达式，以及如何无需任何额外工作就可以使用列名。

If you’re coming from a SQL background, all of these statements should seem quite familiar. Indeed, all of them can be expressed as a where clause. In fact, it’s often easier to just express filters as SQL statements than using the programmatic DataFrame interface and Spark SQL allows us to do this without paying any performance penalty. For example, the following two statements are equivalent: 

如果您来自SQL背景，那么所有这些语句似乎都应该很熟悉。 实际上，所有这些都可以表示为where子句。 实际上，仅将过滤器表示为SQL语句通常比使用程序化DataFrame接口更容易，并且Spark SQL允许我们执行此操作而无需付出任何性能损失。 例如，以下两个语句是等效的：

```scala
// in Scala
import org.apache.spark.sql.functions.{expr, not, col}

df.withColumn("isExpensive", not(col("UnitPrice").leq(250)))
.filter("isExpensive")
.select("Description", "UnitPrice").show(5)

df.withColumn("isExpensive", expr("NOT UnitPrice <= 250"))
.filter("isExpensive")
.select("Description", "UnitPrice").show(5)
```

Here’s our state definition:

这是我们的语句定义：

```python
# in Python
from pyspark.sql.functions import expr
df.withColumn("isExpensive", expr("NOT UnitPrice <= 250"))\
.where("isExpensive")\
.select("Description", "UnitPrice").show(5)
```

---

<p><center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong></font></center></p>
One “gotcha” that can come up is if you’re working with null data when creating Boolean expressions. If there is a null in your data, you’ll need to treat things a bit differently. Here’s how you can ensure that you perform a null-safe equivalence test:

一个可能出现的“陷阱”是创建布尔表达式时是否使用空数据。 如果您的数据为空，则需要对数据进行一些不同的处理。 您可以通过以下方式确保执行空值安全等效测试：

```scala
df.where(col("Description").eqNullSafe("hello")).show() 
```

---

Although not currently available (Spark 2.2), `IS [NOT] DISTINCT FROM` will be coming in Spark 2.3 to do the same thing in SQL. 

尽管目前尚不可用（Spark 2.2），但 `IS [NOT] DISTINCT FROM` 将在Spark 2.3中使用SQL进行相同的操作。

## <font color="#9a161a">Working with Numbers 使用数字</font>

When working with big data, the second most common task you will do after filtering things is counting things. For the most part, we simply need to express our computation, and that should be valid assuming that we’re working with numerical data types.

在处理大数据时，过滤事物后要做的第二个最常见的任务是计数。 在大多数情况下，我们只需要表达我们的计算量，并且假设我们使用的是数值数据类型，那么这应该是有效的。

To fabricate a contrived example, let’s imagine that we found out that we mis-recorded the quantity in our retail dataset and the true quantity is equal to (the current quantity * the unit price) + 5. This will introduce our first numerical function as well as the `pow` function that raises a column to the expressed power: 

为了构造一个人为的示例，让我们想象一下，我们发现我们在零售数据集中错误地记录了数量，真实数量等于（当前数量*单价）+5。这将引入我们的第一个数值函数为以及 `pow` 函数，该函数将列提高到表示的功效：

```scala
// in Scala
import org.apache.spark.sql.functions.{expr, pow}
val fabricatedQuantity = pow(col("Quantity") * col("UnitPrice"), 2) + 5
df.select(expr("CustomerId"), fabricatedQuantity.alias("realQuantity")).show(2)
```

```python
# in Python
from pyspark.sql.functions import expr, pow
fabricatedQuantity = pow(col("Quantity") * col("UnitPrice"), 2) + 5
df.select(expr("CustomerId"), fabricatedQuantity.alias("realQuantity")).show(2)
```

```sql
+----------+------------------+
|CustomerId|   realQuantity   |
+----------+------------------+
| 17850.0  |239.08999999999997|
| 17850.0  |     418.7156     |
+----------+------------------+
```

Notice that we were able to multiply our columns together because they were both numerical. Naturally we can add and subtract as necessary, as well. In fact, we can do all of this as a SQL expression, as well: 

注意，我们能够将列相乘，因为它们都是数值。 当然，我们也可以根据需要添加和减去。 实际上，我们也可以将所有这些操作都作为SQL表达式来完成：

```scala
// in Scala
df.selectExpr(
"CustomerId",
"(POWER((Quantity * UnitPrice), 2.0) + 5) as realQuantity").show(2)
```

```python
# in Python
df.selectExpr(
"CustomerId",
"(POWER((Quantity * UnitPrice), 2.0) + 5) as realQuantity").show(2)
```

```sql
-- in SQL
SELECT customerId, (POWER((Quantity * UnitPrice), 2.0) + 5) as realQuantity
FROM dfTable 
```

Another common numerical task is rounding. If you’d like to just round to a whole number, oftentimes you can cast the value to an integer and that will work just fine. However, Spark also has more detailed functions for performing this explicitly and to a certain level of precision. In the following example, we round to one decimal place:

另一个常见的数字任务是四舍五入。 如果您想四舍五入为整数，通常可以将值转换为整数，这样就可以正常工作。 但是，Spark还具有更详细的功能，可以显式执行此操作并达到一定的精度。 在下面的示例中，我们四舍五入到小数点后一位：

```scala
// in Scala
import org.apache.spark.sql.functions.{round, bround}
df.select(round(col("UnitPrice"), 1).alias("rounded"), col("UnitPrice")).show(5)
```

By default, the `round` function rounds up if you’re exactly in between two numbers. You can round down by using the `bround`:

默认情况下，如果您恰好在两个数字之间，用 `round` 函数会四舍五入。 您可以使用 `bround` 向下取整：

```scala
// in Scala
import org.apache.spark.sql.functions.lit
df.select(round(lit("2.5")), bround(lit("2.5"))).show(2)
```

```python
# in Python
from pyspark.sql.functions import lit, round, bround
df.select(round(lit("2.5")), bround(lit("2.5"))).show(2)
```

```sql
-- in SQL
SELECT round(2.5), bround(2.5)
```

```shell
+-------------+--------------+
|round(2.5, 0)|bround(2.5, 0)|
+-------------+--------------+
|    3.0      |    2.0       |
|    3.0      |    2.0       |
+-------------+--------------+ 
```

Another numerical task is to compute the correlation of two columns. For example, we can see the Pearson correlation coefficient for two columns to see if cheaper things are typically bought in greater quantities. We can do this through a function as well as through the DataFrame statistic methods: 

另一个数字任务是计算两列的相关性。 例如，我们可以看到两列的Pearson相关系数，以查看是否通常会更大量地购买便宜的东西。 我们可以通过一个函数以及通过DataFrame统计方法来做到这一点：

```scala
// in Scala
import org.apache.spark.sql.functions.{corr}
df.stat.corr("Quantity", "UnitPrice")
df.select(corr("Quantity", "UnitPrice")).show()
```

```python
# in Python
from pyspark.sql.functions import corr
df.stat.corr("Quantity", "UnitPrice")
df.select(corr("Quantity", "UnitPrice")).show()
```

```sql
-- in SQL
SELECT corr(Quantity, UnitPrice) FROM dfTable
```

```shell
+-------------------------+
|corr(Quantity, UnitPrice)|
+-------------------------+
|   -0.04112314436835551  |
+-------------------------+
```

Another common task is to compute summary statistics for a column or set of columns. We can use the describe method to achieve exactly this. This will take all numeric columns and calculate the count, mean, standard deviation, min, and max. You should use this primarily for viewing in the console because the schema might change in the future:

另一个常见任务是为一列或一组列计算摘要统计信息。我们可以使用describe方法来实现这一目标。这将占用所有数字列，并计算计数，平均值，标准偏差，最小值和最大值。您应该主要在控制台中使用它，因为模式（schema）将来可能会更改：

```scala
// in Scala
df.describe().show()
```

```python
# in Python
df.describe().show()
```

```shell
+-------+------------------+------------------+------------------+
|summary|    Quantity      | UnitPrice        |   CustomerID     |
+-------+------------------+------------------+------------------+
| count |       3108       |   3108           |         1968     |
| mean  | 8.627413127413128|4.151946589446603 |15661.388719512195|
| stddev|26.371821677029203|15.638659854603892|1854.4496996893627|
| min   |        -24       |     0.0          |     12431.0      |
| max   |        600       |     607.49       |     18229.0      |
+-------+------------------+------------------+------------------+
```

If you need these exact numbers, you can also perform this as an aggregation yourself by importing the functions and applying them to the columns that you need: 

如果您需要这些确切的数字，也可以通过导入函数并将其应用于所需的列来执行作为聚合：

```scala
// in Scala
import org.apache.spark.sql.functions.{count, mean, stddev_pop, min, max}
```

```python
# in Python
from pyspark.sql.functions import count, mean, stddev_pop, min, max
```

There are a number of statistical functions available in the `StatFunctions` Package (accessible using stat as we see in the code block below). These are DataFrame methods that you can use to calculate a variety of different things. For instance, you can calculate either exact or approximate quantiles of your data using the `approxQuantile` method:

`StatFunctions` 程序包中提供了许多统计功能（如下面的代码块所示，可以使用stat访问）。这些是DataFrame方法，可用于计算各种不同的事物。例如，您可以使用 `approxQuantile`  方法计算数据的精确或近似分位数：

```scala
// in Scala
val colName = "UnitPrice"
val quantileProbs = Array(0.5)
val relError = 0.05
df.stat.approxQuantile("UnitPrice", quantileProbs, relError) // 2.51
```

```python
# in Python
colName = "UnitPrice"
quantileProbs = [0.5]
relError = 0.05
df.stat.approxQuantile("UnitPrice", quantileProbs, relError) # 2.51
```

You also can use this to see a cross-tabulation or frequent item pairs (be careful, this output will be large and is omitted for this reason):

您还可以使用它来查看交叉列表或出现频率很高的的项目对（请注意，此输出将很大，因此被省略）：

```scala
// in Scala
df.stat.crosstab("StockCode", "Quantity").show()
```

```python
# in Python
df.stat.crosstab("StockCode", "Quantity").show()
```

```scala
// in Scala
df.stat.freqItems(Seq("StockCode", "Quantity")).show()
```

```python
# in Python
df.stat.freqItems(["StockCode", "Quantity"]).show()
```

As a last note, we can also add a unique ID to each row by using the function `monotonically_increasing_id`. This function generates a unique value for each row, starting with 0:

最后，我们还可以通过使用 `monotonically_increasing_id` 函数向每行添加唯一的ID。此函数为每一行生成一个唯一值，从0开始：

```scala
// in Scala
import org.apache.spark.sql.functions.monotonically_increasing_id
df.select(monotonically_increasing_id()).show(2)
```

```python
# in Python
from pyspark.sql.functions import monotonically_increasing_id
df.select(monotonically_increasing_id()).show(2)
```

There are functions added with every release, so check the documentation for more methods. For instance, there are some random data generation tools (e.g., `rand(), randn()`) with which you can randomly generate data; however, there are potential determinism issues when doing so. (You can find discussions about these challenges on the Spark mailing list.) There are also a number of more advanced tasks like bloom filtering and sketching algorithms available in the stat package that we mentioned (and linked to) at the beginning of this chapter. Be sure to search the API documentation for more information and functions. 

每个发行版中都添加了功能，因此请查看文档以了解更多方法。例如，有些随机数据生成工具（例如`rand(), randn()`）可用来随机生成数据；但是，这样做时存在潜在的确定性问题。 （您可以在Spark邮件列表中找到有关这些挑战的讨论）在本章的开头，我们还提到了（并链接到）stat包中还有许多更高级的任务，例如布隆过滤和草图绘制算法(sketching algorithm)。确保搜索API文档以获取更多信息和功能。

## <font color="#9a161a">Working with Strings 使用字符串</font>

String manipulation shows up in nearly every data flow, and it’s worth explaining what you can do with strings. You might be manipulating log files performing regular expression extraction or substitution, or checking for simple string existence, or making all strings uppercase or lowercase.

字符串操作几乎出现在每个数据流中，值得解释如何使用字符串。您可能正在操纵执行正则表达式提取或替换的日志文件，或者检查是否存在简单的字符串，或者将所有字符串都设置为大写或小写。

Let’s begin with the last task because it’s the most straightforward. The `initcap` function will capitalize every word in a given string when that word is separated from another by a space. 

让我们从最后一个任务开始，因为它是最简单的。当一个给定的字符串中每个单词之间用空格隔开时，`initcap`函数会将每个单词的首字母大写。

```scala
// in Scala
import org.apache.spark.sql.functions.{initcap}
df.select(initcap(col("Description"))).show(2, false)
```

```python
# in Python
from pyspark.sql.functions import initcap
df.select(initcap(col("Description"))).show()
```

```sql
-- in SQL
SELECT initcap(Description) FROM dfTable
```

```shell
+----------------------------------+
|       initcap(Description)       |
+----------------------------------+
|White Hanging Heart T-light Holder|
|        White Metal Lantern       |
+----------------------------------+
```

As just mentioned, you can cast strings in uppercase and lowercase, as well: 

如前所述，您还可以将字符串转换为大写和小写形式：

```scala
// in Scala
import org.apache.spark.sql.functions.{lower, upper}
df.select(col("Description"),
lower(col("Description")),
upper(lower(col("Description")))).show(2)
```

```python
# in Python
from pyspark.sql.functions import lower, upper
df.select(col("Description"),
lower(col("Description")),
upper(lower(col("Description")))).show(2)
```
```sql
-- in SQL
SELECT Description, lower(Description), Upper(lower(Description)) FROM dfTable
```

```shell
+--------------------+--------------------+-------------------------+
|    Description     | lower(Description) |upper(lower(Description))|
+--------------------+--------------------+-------------------------+
|WHITE HANGING HEA...|white hanging hea...|   WHITE HANGING HEA...  |
| WHITE METAL LANTERN| white metal lantern|   WHITE METAL LANTERN   |
+--------------------+--------------------+-------------------------+
```

Another trivial task is adding or removing spaces around a string. You can do this by using `lpad`, `ltrim`, `rpad `and `rtrim`, `trim`: 

另一个琐碎的任务是在字符串周围添加或删除空格。 您可以使用`lpad`，`ltrim`，`rpad` 和 `rtrim`，`trim`来做到这一点：

```scala
// in Scala
import org.apache.spark.sql.functions.{lit, ltrim, rtrim, rpad, lpad, trim}
df.select(
ltrim(lit(" HELLO ")).as("ltrim"),
rtrim(lit(" HELLO ")).as("rtrim"),
trim(lit(" HELLO ")).as("trim"),
lpad(lit("HELLO"), 3, " ").as("lp"),
rpad(lit("HELLO"), 10, " ").as("rp")).show(2)
```

```python
# in Python
from pyspark.sql.functions import lit, ltrim, rtrim, rpad, lpad, trim
df.select(
ltrim(lit(" HELLO ")).alias("ltrim"),
rtrim(lit(" HELLO ")).alias("rtrim"),
trim(lit(" HELLO ")).alias("trim"),
lpad(lit("HELLO"), 3, " ").alias("lp"),
rpad(lit("HELLO"), 10, " ").alias("rp")).show(2)
```

```sql
-- in SQL
SELECT
ltrim(' HELLLOOOO '),
rtrim(' HELLLOOOO '),
trim(' HELLLOOOO '),
lpad('HELLOOOO ', 3, ' '),
rpad('HELLOOOO ', 10, ' ')
FROM dfTable
```

```shell
+---------+---------+-----+---+----------+
|   ltrim |  rtrim  | trim| lp|    rp    |
+---------+---------+-----+---+----------+
|  HELLO  |  HELLO  |HELLO| HE|HELLO     |
|  HELLO  |  HELLO  |HELLO| HE|HELLO     |
+---------+---------+-----+---+----------+
```

Note that if lpad or rpad takes a number less than the length of the string, it will always remove values from the right side of the string.

请注意，如果lpad或rpad的数字小于字符串的长度，它将始终从字符串的右侧删除值。

### <font color="#00000">Regular Expressions 正则表达式</font>

Probably one of the most frequently performed tasks is searching for the existence of one string in another or replacing all mentions of a string with another value. This is often done with a tool called regular expressions that exists in many programming languages. Regular expressions give the user an ability to specify a set of rules to use to either extract values from a string or replace them with some other values.

可能是执行最频繁的任务之一是在另一个字符串中查找一个字符串的存在，或用另一个值替换所有提及的字符串。通常使用许多编程语言中存在的称为正则表达式的工具来完成此操作。正则表达式使用户能够指定一组规则，以用于从字符串中提取值或将其替换为其他值。

Spark takes advantage of the complete power of Java regular expressions. The Java regular expression syntax departs slightly from other programming languages, so it is worth reviewing before putting anything into production. There are two key functions in Spark that you’ll need in order to perform regular expression tasks: regexp_extract and regexp_replace. These functions extract values and replace values, respectively.

Spark利用了Java正则表达式的全部功能。 Java正则表达式语法与其他编程语言略有不同，因此在将任何产品投入生产之前，值得回顾一下。为了执行正则表达式任务，Spark中需要两个关键功能：regexp_extract和regexp_replace。这些函数分别提取值和替换值。

Let’s explore how to use the `regexp_replace` function to replace substitute color names in our description column: 

让我们探索一下如何使用 `regexp_replace` 函数替换描述列中的替代颜色名称：
```scala
// in Scala
import org.apache.spark.sql.functions.regexp_replace
val simpleColors = Seq("black", "white", "red", "green", "blue")
val regexString = simpleColors.map(_.toUpperCase).mkString("|")
// the | signifies `OR` in regular expression syntax
df.select(
regexp_replace(col("Description"), regexString, "COLOR").alias("color_clean"),
col("Description")).show(2)
```

```python
# in Python
from pyspark.sql.functions import regexp_replace
regex_string = "BLACK|WHITE|RED|GREEN|BLUE"
df.select(
regexp_replace(col("Description"), regex_string, "COLOR").alias("color_clean"),
col("Description")).show(2)
```

```sql
-- in SQL
SELECT
regexp_replace(Description, 'BLACK|WHITE|RED|GREEN|BLUE', 'COLOR') as
color_clean, Description
FROM dfTable
```

```shell
+--------------------+--------------------+
|    color_clean     |     Description    |
+--------------------+--------------------+
|COLOR HANGING HEA...|WHITE HANGING HEA...|
| COLOR METAL LANTERN| WHITE METAL LANTERN|
+--------------------+--------------------+
```

Another task might be to replace given characters with other characters. Building this as a regular expression could be tedious, so Spark also provides the translate function to replace these values. This is done at the character level and will replace all instances of a character with the indexed character in the replacement string: 

另一个任务可能是用其他字符替换给定的字符。 将其构建为正则表达式可能很繁琐，因此Spark还提供了 `translation` 函数来替换这些值。 这是在字符级别完成的，它将用替换字符串中的索引字符替换字符的所有实例：

```scala
// in Scalaimport org.apache.spark.sql.functions.translate
df.select(translate(col("Description"), "LEET", "1337"), col("Description"))
.show(2)
```

```python
# in Python
from pyspark.sql.functions import translate
df.select(translate(col("Description"), "LEET", "1337"),col("Description"))\
.show(2)
```

```sql
-- in SQL
SELECT translate(Description, 'LEET', '1337'), Description FROM dfTable
```

```shell
+----------------------------------+--------------------+
|translate(Description, LEET, 1337)|    Description     |
+----------------------------------+--------------------+
|   WHI73 HANGING H3A...           |WHITE HANGING HEA...|
|   WHI73 M37A1 1AN73RN            | WHITE METAL LANTERN|
+----------------------------------+--------------------+
```

We can also perform something similar, like pulling out the first mentioned color:

我们还可以执行类似的操作，例如拉出第一个提到的颜色：

```scala
// in Scala
import org.apache.spark.sql.functions.regexp_extract
val regexString = simpleColors.map(_.toUpperCase).mkString("(", "|", ")")
// the | signifies OR in regular expression syntax
df.select(
regexp_extract(col("Description"), regexString, 1).alias("color_clean"),
col("Description")).show(2)
```

```python
# in Python
from pyspark.sql.functions import regexp_extract
extract_str = "(BLACK|WHITE|RED|GREEN|BLUE)"
df.select(
regexp_extract(col("Description"), extract_str, 1).alias("color_clean"),
col("Description")).show(2)
```

```sql
-- in SQL
SELECT regexp_extract(Description, '(BLACK|WHITE|RED|GREEN|BLUE)', 1),
Description FROM dfTable
```

```shell
+-------------+--------------------+
| color_clean |     Description    |
+-------------+--------------------+
|    WHITE    |WHITE HANGING HEA...|
|    WHITE    | WHITE METAL LANTERN|
+-------------+--------------------+
```

Sometimes, rather than extracting values, we simply want to check for their existence. We can do this with the contains method on each column. This will return a Boolean declaring whether the value you specify is in the column’s string: 

有时，我们只是想检查它们的存在，而不是提取值。 我们可以在每一列上使用contains方法来做到这一点。 这将返回一个布尔值，该布尔值声明您指定的值是否在该列的字符串中：

```scala
// in Scala
val containsBlack = col("Description").contains("BLACK")
val containsWhite = col("DESCRIPTION").contains("WHITE")
df.withColumn("hasSimpleColor", containsBlack.or(containsWhite))
.where("hasSimpleColor")
.select("Description").show(3, false)
```

In Python and SQL, we can use the `instr` function: 

在Python和SQL中，我们可以使用 `instr` 函数：

```python
# in Python
from pyspark.sql.functions import instr
containsBlack = instr(col("Description"), "BLACK") >= 1
containsWhite = instr(col("Description"), "WHITE") >= 1
df.withColumn("hasSimpleColor", containsBlack | containsWhite)\
.where("hasSimpleColor")\
.select("Description").show(3, False)
```

```sql
-- in SQL
SELECT Description FROM dfTable
WHERE instr(Description, 'BLACK') >= 1 OR instr(Description, 'WHITE') >= 1
```

```shell
+----------------------------------+
|            Description           |
+----------------------------------+
|WHITE HANGING HEART T-LIGHT HOLDER|
|WHITE METAL LANTERN               |
|RED WOOLLY HOTTIE WHITE HEART.    |
+----------------------------------+
```

This is trivial with just two values, but it becomes more complicated when there are values. Let’s work through this in a more rigorous way and take advantage of Spark’s ability to accept a dynamic number of arguments. When we convert a list of values into a set of arguments and pass them into a function, we use a language feature called `varargs`. Using this feature, we can effectively unravel an array of arbitrary length and pass it as arguments to a function. This, coupled with select makes it possible for us to create arbitrary numbers of columns dynamically: 

仅使用两个值，这是微不足道的，但是当存在值时，它变得更加复杂。 让我们以更严格的方式进行研究，并利用Spark接受动态数量的参数的能力。 当我们将值列表转换为一组参数并将其传递给函数时，我们使用一种称为`varargs` 的语言功能。 使用此功能，我们可以有效地解开任意长度的数组并将其作为参数传递给函数。 结合select，我们可以动态创建任意数量的列：

```scala
// in Scala
val simpleColors = Seq("black", "white", "red", "green", "blue")
val selectedColumns = simpleColors.map(color => {
col("Description").contains(color.toUpperCase).alias(s"is_$color")
}):+expr("*") // could also append this value

df.select(selectedColumns:_*).where(col("is_white").or(col("is_red")))
.select("Description").show(3, false)
```

```shell
+----------------------------------+
|            Description           |
+----------------------------------+
|WHITE HANGING HEART T-LIGHT HOLDER|
|WHITE METAL LANTERN               |
|RED WOOLLY HOTTIE WHITE HEART.    |
+----------------------------------+
```

We can also do this quite easily in Python. In this case, we’re going to use a different function, `locate` , that returns the integer location (1 based location). We then convert that to a Boolean before using it as the same basic feature: 

我们也可以在Python中很容易地做到这一点。在这种情况下，我们将使用另一个函数 `locate`，该函数返回整数位置（从1开始的位置）。然后，在将其用作相同的基本功能之前，将其转换为布尔值：

```scala
# in Python
from pyspark.sql.functions import expr, locate
simpleColors = ["black", "white", "red", "green", "blue"]
def color_locator(column, color_string):
return locate(color_string.upper(), column).cast("boolean")\
.alias("is_" + c)

selectedColumns = [color_locator(df.Description, c) for c in simpleColors]
selectedColumns.append(expr("*")) # has to a be Column type
df.select(*selectedColumns).where(expr("is_white OR is_red"))\
.select("Description").show(3, False)
```

This simple feature can often help you programmatically generate columns or Boolean filters in a way that is simple to understand and extend. We could extend this to calculating the smallest common denominator for a given input value, or whether a number is a prime.

这个简单的功能通常可以帮助您以易于理解和扩展的方式以编程方式生成列或布尔过滤器。我们可以将其扩展为计算给定输入值或数字是否为质数的最小公分母。

## <font color="#9a161a">Working with Dates and Timestamps 使用日期和时间戳记</font>

Dates and times are a constant challenge in programming languages and databases. It’s always necessary to keep track of timezones and ensure that formats are correct and valid. Spark does its best to keep things simple by focusing explicitly on two kinds of time-related information. There are dates, which focus exclusively on calendar dates, and timestamps, which include both date and time information. Spark, as we saw with our current dataset, will make a best effort to correctly identify column types, including dates and timestamps when we enable `inferSchema`. We can see that this worked quite well with our current dataset because it was able to identify and read our date format without us having to provide some specification for it.

在编程语言和数据库中，日期和时间一直是一个挑战。始终需要跟踪时区并确保格式正确和有效。 Spark通过明确关注两种与时间相关的信息来尽力使事情变得简单。有一些日期（仅专注于日历日期）和时间戳（包括日期和时间信息）。正如我们在当前数据集中看到的那样，当启用“ inferSchema”时，Spark将尽最大努力正确识别列类型，包括日期和时间戳。我们可以看到，这对于我们当前的数据集非常有效，因为它能够识别和读取我们的日期格式，而无需我们提供一些规范。

As we hinted earlier, working with dates and timestamps closely relates to working with strings because we often store our timestamps or dates as strings and convert them into date types at runtime. This is less common when working with databases and structured data but much more common when we are working with text and CSV files. We will experiment with that shortly. 

正如我们之前所暗示的，使用日期和时间戳与使用字符串紧密相关，因为我们经常将时间戳或日期存储为字符串并将其在运行时转换为日期类型。在使用数据库和结构化数据时，这种情况不太常见，但是在处理文本和CSV文件时，这种情况更为常见。我们将很快对此进行试验。

---

<p><center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong></font></center></p>
There are a lot of caveats, unfortunately, when working with dates and timestamps, especially when it comes to timezone handling. In version 2.1 and before, Spark parsed according to the machine’s timezone if timezones are not explicitly specified in the value that you are parsing. You can set a session local timezone if necessary by setting `spark.conf.sessionLocalTimeZone` in the SQL configurations. This should be set according to the Java TimeZone format.

不幸的是，在处理日期和时间戳时，尤其是在时区处理方面，有很多警告。 在2.1版及更高版本中，如果未在要解析的值中明确指定时区，则Spark将根据计算机的时区进行解析。 您可以根据需要通过在SQL配置中设置`spark.conf.sessionLocalTimeZone` 来设置会话本地时区。 应该根据Java TimeZone格式进行设置。

```scala
df.printSchema()
```

```shell
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

---

Although Spark will do read dates or times on a best-effort basis. However, sometimes there will be no getting around working with strangely formatted dates and times. The key to understanding the transformations that you are going to need to apply is to ensure that you know exactly what type and format you have at each given step of the way. Another common “gotcha” is that Spark’s `TimestampType` class supports only second-level precision, which means that if you’re going to be working with milliseconds or microseconds, you’ll need to work around this problem by potentially operating on them as longs. Any more precision when coercing to a `TimestampType` will be removed.

尽管Spark会尽最大努力读取日期或时间。但是，有时无法解决格式和日期格式异常的问题。理解将要应用的转换的关键是确保您确切地知道在此过程中的每个给定步骤中所具有的类型和格式。另一个常见的“陷阱”是Spark的`TimestampType` 类仅支持二级精度，这意味着如果您要使用毫秒或微秒，则可能需要长时间对其进行操作来解决此问题。强制转换为 `TimestampType` 时，将删除任何更高的精度。

Spark can be a bit particular about what format you have at any given point in time. It’s important to be explicit when parsing or converting to ensure that there are no issues in doing so. At the end of the day, Spark is working with Java dates and timestamps and therefore conforms to those standards.

Spark可能会在任何给定时间点上对您使用哪种格式有些特殊。 解析或转换时必须明确，以确保这样做没有问题。 归根结底，Spark正在使用Java日期和时间戳，因此符合这些标准。

Let’s begin with the basics and get the current date and the current timestamps: 

让我们从基础开始，获取当前日期和当前时间戳：

```scala
// in Scala
import org.apache.spark.sql.functions.{current_date, current_timestamp}
val dateDF = spark.range(10)
.withColumn("today", current_date())
.withColumn("now", current_timestamp())
dateDF.createOrReplaceTempView("dateTable")
```

```python
# in Python
from pyspark.sql.functions import current_date, current_timestamp
dateDF = spark.range(10)\
.withColumn("today", current_date())\
.withColumn("now", current_timestamp())
dateDF.createOrReplaceTempView("dateTable")dateDF.printSchema()
```

```shell
root
|-- id: long (nullable = false)
|-- today: date (nullable = false)
|-- now: timestamp (nullable = false)
```

Now that we have a simple DataFrame to work with, let’s add and subtract five days from today. These functions take a column and then the number of days to either add or subtract as the arguments:

现在我们有了一个简单的DataFrame，让我们从今天开始增加和减少5天。 这些函数使用一列，然后加上要加或减的天数作为参数：

```scala
// in Scala
import org.apache.spark.sql.functions.{date_add, date_sub}
dateDF.select(date_sub(col("today"), 5), date_add(col("today"), 5)).show(1)
```

```python
# in Python
from pyspark.sql.functions import date_add, date_sub
dateDF.select(date_sub(col("today"), 5), date_add(col("today"), 5)).show(1)
```

```sql
-- in SQL
SELECT date_sub(today, 5), date_add(today, 5) FROM dateTable
```

```shell
+------------------+------------------+
|date_sub(today, 5)|date_add(today, 5)|
+------------------+------------------+
|    2017-06-12    |   2017-06-22     |
+------------------+------------------+
```

Another common task is to take a look at the difference between two dates. We can do this with the `datediff` function that will return the number of days in between two dates. Most often we just care about the days, and because the number of days varies from month to month, there also exists a function, months_between, that gives you the number of months between two dates: 

另一个常见的任务是查看两个日期之间的差异。 我们可以使用 `datediff` 函数来执行此操作，该函数将返回两个日期之间的天数。 大多数情况下，我们只关心日期，并且由于天数每个月都不同，因此还存在一个 `months_between` 函数，该函数可为您提供两个日期之间的月数：

```scala
// in Scala
import org.apache.spark.sql.functions.{datediff, months_between, to_date}
dateDF.withColumn("week_ago", date_sub(col("today"), 7))
.select(datediff(col("week_ago"), col("today"))).show(1)
dateDF.select(
to_date(lit("2016-01-01")).alias("start"),
to_date(lit("2017-05-22")).alias("end"))
.select(months_between(col("start"), col("end"))).show(1)
```

```python
# in Python
from pyspark.sql.functions import datediff, months_between, to_date
dateDF.withColumn("week_ago", date_sub(col("today"), 7))\
.select(datediff(col("week_ago"), col("today"))).show(1)
dateDF.select(
to_date(lit("2016-01-01")).alias("start"),
to_date(lit("2017-05-22")).alias("end"))\
.select(months_between(col("start"), col("end"))).show(1)-- in SQL
SELECT to_date('2016-01-01'), months_between('2016-01-01', '2017-01-01'),
datediff('2016-01-01', '2017-01-01')
FROM dateTable
```

```shell
+-------------------------+
|datediff(week_ago, today)|
+-------------------------+
|          -7             |
+-------------------------+
+--------------------------+
|months_between(start, end)|
+--------------------------+
|         -16.67741935     |
+--------------------------+
```

Notice that we introduced a new function: the to_date function. The to_date function allows you to convert a string to a date, optionally with a specified format. We specify our format in the [Java SimpleDateFormat](https://docs.oracle.com/javase/tutorial/i18n/format/simpleDateFormat.html) which will be important to reference if you use this function: 

注意，我们引入了一个新函数：`to_date` 函数。 `to_date` 函数允许您将字符串转换为日期，可以选择使用指定格式。 我们在 [Java SimpleDateFormat](https://docs.oracle.com/javase/tutorial/i18n/format/simpleDateFormat.html) 中指定我们的格式，如果您使用此函数，这对于引用这个函数很重要：

 ```scala
// in Scala
import org.apache.spark.sql.functions.{to_date, lit}
spark.range(5).withColumn("date", lit("2017-01-01"))
.select(to_date(col("date"))).show(1)
 ```

```python
# in Python
from pyspark.sql.functions import to_date, lit
spark.range(5).withColumn("date", lit("2017-01-01"))\
.select(to_date(col("date"))).show(1)
```

Spark will not throw an error if it cannot parse the date; rather, it will just return null. This can be a bit tricky in larger pipelines because you might be expecting your data in one format and getting it in another. To illustrate, let’s take a look at the date format that has switched from year-month-day to year-day-month. Spark will fail to parse this date and silently return null instead: 

如果无法解析日期，Spark不会抛出错误。相反，它将仅返回null。在较大的管道中，这可能会有些棘手，因为您可能期望数据是一种格式并以另一种格式获取。为了说明这一点，让我们看一下从 year-month-day 转换为year-day-month的日期格式。 Spark将无法解析此日期，而是静默返回null：

```scala
dateDF.select(to_date(lit("2016-20-12")),to_date(lit("2017-12-11"))).show(1)
+-------------------+-------------------+
|to_date(2016-20-12)|to_date(2017-12-11)|
+-------------------+-------------------+
|        null       |      2017-12-11   |
+-------------------+-------------------+
```

We find this to be an especially tricky situation for bugs because some dates might match the correct format, whereas others do not. In the previous example, notice how the second date appears as Decembers 11th instead of the correct day, November 12th. Spark doesn’t throw an error because it cannot know whether the days are mixed up or that specific row is incorrect.

我们发现这种情况对于bug来说尤其棘手，因为某些日期可能与正确的格式匹配，而另一些则不匹配。 在上一个示例中，请注意第二个日期如何显示为12月11日，而不是正确的日期11月12日。 Spark不会引发错误，因为它不知道这些日期是混合的还是特定的行不正确。


Let’s fix this pipeline, step by step, and come up with a robust way to avoid these issues entirely. The first step is to remember that we need to specify our date format according to the [Java SimpleDateFormat standard](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html).

让我们逐步解决此问题，并提出一种健壮的方法来完全避免这些问题。第一步是要记住，我们需要根据[Java SimpleDateFormat标准](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html) 指定日期格式。

We will use two functions to fix this: `to_date` and `to_timestamp`. The former optionally expects a format, whereas the latter requires one: 

我们将使用两个函数来解决此问题：`to_date` 和 `to_timestamp`。前者可以选择一种格式，而后者则需要一种格式：

```scala
// in Scala
import org.apache.spark.sql.functions.to_date
val dateFormat = "yyyy-dd-MM"
val cleanDateDF = spark.range(1).select(
to_date(lit("2017-12-11"), dateFormat).alias("date"),
to_date(lit("2017-20-12"), dateFormat).alias("date2"))
cleanDateDF.createOrReplaceTempView("dateTable2")
```

```python
# in Python
from pyspark.sql.functions import to_date
dateFormat = "yyyy-dd-MM"
cleanDateDF = spark.range(1).select(
to_date(lit("2017-12-11"), dateFormat).alias("date"),
to_date(lit("2017-20-12"), dateFormat).alias("date2"))
cleanDateDF.createOrReplaceTempView("dateTable2")
```

```sql
-- in SQL
SELECT to_date(date, 'yyyy-dd-MM'), to_date(date2, 'yyyy-dd-MM'), to_date(date)
FROM dateTable2
```

```txt
+----------+----------+
|    date  | date2    |
+----------+----------+
|2017-11-12|2017-12-20|
+----------+----------+
```

Now let’s use an example of to_timestamp, which always requires a format to be specified: 

现在，我们使用 `to_timestamp` 的示例，该示例始终需要指定一种格式：

```scala
// in Scala
import org.apache.spark.sql.functions.to_timestamp
cleanDateDF.select(to_timestamp(col("date"), dateFormat)).show()
```

```python
# in Python
from pyspark.sql.functions import to_timestamp
cleanDateDF.select(to_timestamp(col("date"), dateFormat)).show()
```

```sql
-- in SQL
SELECT to_timestamp(date, 'yyyy-dd-MM'), to_timestamp(date2, 'yyyy-dd-MM')
FROM dateTable2
```

```txt
+----------------------------------+
|to_timestamp(`date`, 'yyyy-dd-MM')|
+----------------------------------+
|        2017-11-12 00:00:00       |
+----------------------------------+
```

Casting between dates and timestamps is simple in all languages—in SQL, we would do it in the following way: 

在所有语言中，日期和时间戳之间的转换都很简单——在SQL中，我们可以通过以下方式进行：

```sql
-- in SQL
SELECT cast(to_date("2017-01-01", "yyyy-dd-MM") as timestamp)
```

After we have our date or timestamp in the correct format and type, comparing between them is actually quite easy. We just need to be sure to either use a date/timestamp type or specify our string according to the right format of yyyy-MM-dd if we’re comparing a date: 

在以正确的格式和类型获得日期或时间戳后，实际上比较起来很容易。 如果要比较日期，我们只需要确保使用日期/时间戳类型或根据yyyy-MM-dd的正确格式指定我们的字符串即可：

```scala
cleanDateDF.filter(col("date2") > lit("2017-12-12")).show()
```

One minor point is that we can also set this as a string, which Spark parses to a literal:

一小点是，我们还可以将其设置为字符串，Spark将其解析为文字：

```scala
cleanDateDF.filter(col("date2") > "'2017-12-12'").show()
```

---

<p><center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong></font></center></p>
Implicit type casting is an easy way to shoot yourself in the foot, especially when dealing with null values or dates in different timezones or formats. We recommend that you parse them explicitly instead of relying on implicit conversions.

隐式类型转换是一种使自己步履蹒跚的简便方法，尤其是在处理具有不同时区或格式的空值或日期时。 我们建议您显式解析它们，而不要依赖隐式转换。

---

## <font color="#9a161a">Working with Nulls in Data 在数据中的空值</font>

As a best practice, you should always use nulls to represent missing or empty data in your DataFrames. Spark can optimize working with null values more than it can if you use empty strings or other values. The primary way of interacting with null values, at DataFrame scale, is to use the `.na` subpackage on a DataFrame. There are also several functions for performing operations and explicitly specifying how Spark should handle null values. For more information, see Chapter 5 (where we discuss ordering), and also refer back to “Working with Booleans”.

最佳做法是，应始终使用null来表示DataFrame中丢失或为空的数据。与使用空字符串或其他值相比，Spark可以优化使用null的工作。在DataFrame规模上，与null进行交互的主要方式是在DataFrame上使用 `.na` 子包。还有一些函数可以执行操作并明确指定Spark应该如何处理null。有关更多信息，请参见第5章（我们将在其中讨论排序），另请参考“使用布尔值”。

---

<p><center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong></font></center></p>
Nulls are a challenging part of all programming, and Spark is no exception. In our opinion, being explicit is always better than being implicit when handling null values. For instance, in this part of the book, we saw how we can define columns as having null types. However, this comes with a catch. When we declare a column as not having a null time, that is not actually enforced. To reiterate, when you define a schema in which all columns are declared to not have null values, Spark will not enforce that and will happily let null values into that column. The nullable signal is simply to help Spark SQL optimize for handling that column. If you have null values in columns that should not have null values, you can get an incorrect result or see strange exceptions that can be difficult to debug.

null 是所有编程中具有挑战性的一部分，Spark也不例外。我们认为，在处理null时，显式总是比隐式好。例如，在本书的这一部分中，我们看到了如何将列定义为具有null类型。但是，这有一个陷阱。当我们声明一列不具有 null 时，实际上并没有强制执行。重申一下，当您定义一个模式，在该模式中声明所有列都不具有null时，Spark将不强制执行该操作，并且会很乐意让null进入该列。可为空的信号仅是为了帮助Spark SQL优化处理该列。如果不含null的列中包含null，则可能会得到错误的结果，或者会看到难以调试的奇怪异常。

---

There are two things you can do with null values: you can explicitly drop nulls or you can fill them with a value (globally or on a per-column basis). Let’s experiment with each of these now.

使用空值可以做两件事：您可以显式删除空值，也可以用一个值（全局或基于每个列）填充空值。让我们现在尝试其中的每一个。

### <font color="#00000">Coalesce 合并</font>

Spark includes a function to allow you to select the first non-null value from a set of columns by using the `coalesce` function. In this case, there are no null values, so it simply returns the first column:

Spark包含一个函数，该函数允许您使用合并（`coalesce`）函数从一组列中选择第一个非空值。 在这种情况下，没有空值，因此它仅返回第一列：

```scala
// in Scala
import org.apache.spark.sql.functions.coalesce
df.select(coalesce(col("Description"), col("CustomerId"))).show()
```

```python
# in Python
from pyspark.sql.functions import coalesce
df.select(coalesce(col("Description"), col("CustomerId"))).show()
```

### <font color="#00000">ifnull, nullIf, nvl, and nvl2</font>

There are several other SQL functions that you can use to achieve similar things. `ifnull` allows you to select the second value if the first is null, and defaults to the first. Alternatively, you could use `nullif`, which returns null if the two values are equal or else returns the second if they are not. `nvl` returns the second value if the first is null, but defaults to the first. Finally, nvl2 returns the second value if the first is not null; otherwise, it will return the last specified value (else_value in the following example):

您还可以使用其他几个SQL函数来实现类似的功能。 如果第一个为 `null`，则 `ifnull` 允许您选择第二个值，默认为第一个。 或者，您可以使用 `nullif`，如果两个值相等，则返回 `null`；否则，返回第二个值。 如果第一个为空，则 `nvl` 返回第二个值，但默认为第一个。 最后，如果第一个不为 `null`，则`nvl2`返回第二个值。 否则，它将返回最后指定的值（在以下示例中为 else_value）：

```sql
-- in SQL
SELECT
ifnull(null, 'return_value'),
nullif('value', 'value'),
nvl(null, 'return_value'),
nvl2('not_null', 'return_value', "else_value")
FROM dfTable LIMIT 1
```

```shell
+------------+----+------------+------------+
|    a       |  b |     c      |     d      |
+------------+----+------------+------------+
|return_value|null|return_value|return_value|
+------------+----+------------+------------+
```

Naturally, we can use these in select expressions on DataFrames, as well.

自然，我们也可以在DataFrames的select表达式中使用它们。

### <font color="#00000">drop 删除</font>

The simplest function is drop, which removes rows that contain nulls. The default is to drop any row in which any value is null:

最简单的函数是drop，它删除包含空值的行。 缺省值为删除任何值为null的行：

```scala
df.na.drop()
df.na.drop("any")
```

In SQL, we have to do this column by column:

在SQL中，我们必须逐列进行此操作：

```shell
-- in SQL
SELECT * FROM dfTable WHERE Description IS NOT NULL
```

Specifying "any" as an argument drops a row if any of the values are null. Using “all” drops the row only if all values are null or NaN for that row:

如果任何值均为空，则将“ any”指定为参数将删除一行。 仅当该行的所有值均为null或NaN时，才使用“ all”删除该行：

```shell
df.na.drop("all")
```

We can also apply this to certain sets of columns by passing in an array of columns:

我们还可以通过传递一个列数组来将其应用于某些列集：

```scala
// in Scala
df.na.drop("all", Seq("StockCode", "InvoiceNo"))
```

```python
# in Python
df.na.drop("all", subset=["StockCode", "InvoiceNo"])
```

### <font color="#00000">fill</font>

Using the fill function, you can fill one or more columns with a set of values. This can be done by specifying a map—that is a particular value and a set of columns. For example, to fill all null values in columns of type String, you might specify the following:

使用填充功能，可以用一组值填充一个或多个列。 这可以通过指定一个映射来完成，该映射是一个特定的值和一组列。 例如，要填充字符串类型的列中的所有空值，可以指定以下内容：

```scala 
df.na.fill("All Null values become this string")
```

We could do the same for columns of type Integer by using `df.na.fill(5:Integer)`, or for Doubles `df.na.fill(5:Double)`. To specify columns, we just pass in an array of column names like we did in the previous example:

我们可以使用 `df.na.fill(5:Integer)` 对Integer类型的列执行相同的操作，也可以对 `df.na.fill(5:Double)` 进行Double操作。 要指定列，我们只需要像上一个示例一样传递一个列名数组：

```scala
// in Scala
df.na.fill(5, Seq("StockCode", "InvoiceNo"))
```

```python
# in Python
df.na.fill("all", subset=["StockCode", "InvoiceNo"])
```

We can also do this with with a Scala Map, where the key is the column name and the value is the value we would like to use to fill null values:

我们也可以使用Scala Map来做到这一点，其中的键是列名，值是我们想要用来填充空值的值：

```scala
// in Scala
val fillColValues = Map("StockCode" -> 5, "Description" -> "No Value")
df.na.fill(fillColValues)
```

```python
# in Python
fill_cols_vals = {"StockCode": 5, "Description" : "No Value"}
df.na.fill(fill_cols_vals)
```

### <font color="#00000">replace 替换</font>

In addition to replacing null values like we did with drop and fill, there are more flexible options that you can use with more than just null values. Probably the most common use case is to replace all values in a certain column according to their current value. The only requirement is that this value be the same type as the original value:

除了像用drop and fill替换空值那样，您还可以使用更多灵活的选项，而不仅仅是空值。可能最常见的用例是根据其当前值替换特定列中的所有值。唯一的要求是该值必须与原始值具有相同的类型：

```scala
// in Scala
df.na.replace("Description", Map("" -> "UNKNOWN"))
```

```python
# in Python
df.na.replace([""], ["UNKNOWN"], "Description") 
```

## <font color="#9a161a">Ordering 排序</font>

As we discussed in Chapter 5, you can use `asc_nulls_first`, `desc_nulls_first`, `asc_nulls_last`, or `desc_nulls_last` to specify where you would like your null values to appear in an ordered DataFrame.

正如我们在第5章中讨论的那样，您可以使用 `asc_nulls_first`，`desc_nulls_first `，`asc_nulls_last` 或`desc_nulls_last` 来指定希望空值出现在有序DataFrame中的位置。

## <font color="#9a161a">Working with Complex Types 使用复杂类型</font>

Complex types can help you organize and structure your data in ways that make more sense for the problem that you are hoping to solve. There are three kinds of complex types: structs, arrays, and maps.

复杂类型可以帮助您以对希望解决的问题更有意义的方式组织和构造数据。 复杂类型共有三种：结构（struct），数组（array）和映射（map）。

### <font color="#00000">Structs 结构</font>

You can think of structs as DataFrames within DataFrames. A worked example will illustrate this more clearly. We can create a struct by wrapping a set of columns in parenthesis in a query:

您可以将结构视为DataFrame中的DataFrame。一个可行的示例将更清楚地说明这一点。我们可以通过在查询中用括号括起一组列来创建结构：

```scala
df.selectExpr("(Description, InvoiceNo) as complex", "")

df.selectExpr("struct(Description, InvoiceNo) as complex", "")

// in Scala
import org.apache.spark.sql.functions.struct
val complexDF = df.select(struct("Description", "InvoiceNo").alias("complex"))
complexDF.createOrReplaceTempView("complexDF")
```

```python
# in Python

from pyspark.sql.functions import struct
complexDF = df.select(struct("Description", "InvoiceNo").alias("complex"))
complexDF.createOrReplaceTempView("complexDF")
```

We now have a DataFrame with a column complex. We can query it just as we might another DataFrame, the only difference is that we use a dot syntax to do so, or the column method `getField`:

现在，我们有了一个带有列复合体的DataFrame。 我们可以像查询另一个DataFrame一样查询它，唯一的区别是我们使用点语法或列方法getField进行查询：

```scala
complexDF.select("complex.Description")
complexDF.select(col("complex").getField("Description"))
```

We can also query all values in the struct by using *. This brings up all the columns to the top-level DataFrame:

我们还可以使用*查询结构中的所有值。 这将所有列调到顶级DataFrame：

```scala
complexDF.select("complex.*")
```

```sql
-- in SQL
SELECT complex.* FROM complexDF
```

### <font color="#00000">Arrays 数组</font>

To define arrays, let’s work through a use case. With our current data, our objective is to take every single word in our Description column and convert that into a row in our DataFrame. The first task is to turn our Description column into a complex type, an array.

要定义数组，让我们研究一下用例。 使用我们当前的数据，我们的目标是获取Description列中的每个单词，并将其转换为DataFrame中的一行。 第一个任务是将我们的Description列转换为复杂类型，即数组。

### <font color="#00000">split 拆分</font>

We do this by using the split function and specify the delimiter:

我们通过使用split函数并指定定界符来做到这一点：

```scala
// in Scala
import org.apache.spark.sql.functions.split
df.select(split(col("Description"), " ")).show(2)
```

```python
# in Python
from pyspark.sql.functions import split
df.select(split(col("Description"), " ")).show(2)
```

```sql
-- in SQL
SELECT split(Description, ' ') FROM dfTable
```

```shell
+---------------------+
|split(Description, ) |
+---------------------+
| [WHITE, HANGING, ...|
| [WHITE, METAL, LA...|
+---------------------+
```

This is quite powerful because Spark allows us to manipulate this complex type as another column. We can also query the values of the array using Python-like syntax:

这非常强大，因为Spark允许我们将这种复杂类型作为另一列进行操作。 我们还可以使用类似Python的语法查询数组的值：

```scala
// in Scala
df.select(split(col("Description"), " ").alias("array_col"))
.selectExpr("array_col[0]").show(2)
```

```python
# in Python
df.select(split(col("Description"), " ").alias("array_col"))\
.selectExpr("array_col[0]").show(2)
```

```sql
-- in SQL
SELECT split(Description, ' ')[0] FROM dfTable
```

This gives us the following result:

这给我们以下结果：

```shell
+------------+
|array_col[0]|
+------------+
|   WHITE    |
|   WHITE    |
+------------+
```

### <font color="#00000">Array Length 数组长度</font>

We can determine the array’s length by querying for its size:

我们可以通过查询数组的大小来确定数组的长度：

```scala
// in Scala
import org.apache.spark.sql.functions.size
df.select(size(split(col("Description"), " "))).show(2) // shows 5 and 3
```

```python
# in Python
from pyspark.sql.functions import size
df.select(size(split(col("Description"), " "))).show(2) # shows 5 and 3
```

### <font color="#00000">array_contains</font>

We can also see whether this array contains a value:

我们还可以查看此数组是否包含值：

```scala
// in Scala
import org.apache.spark.sql.functions.array_contains
df.select(array_contains(split(col("Description"), " "), "WHITE")).show(2)
```

```scala
# in Python
from pyspark.sql.functions import array_contains
.select(array_contains(split(col("Description"), " "), "WHITE")).show(2)
```

```sql
-- in SQL
SELECT array_contains(split(Description, ' '), 'WHITE') FROM dfTable
```

This gives us the following result:

这给我们以下结果：

```shell
+--------------------------------------------+
|array_contains(split(Description, ), WHITE) |
+--------------------------------------------+
|                  true                      |
|                  true                      |
+--------------------------------------------+
```

However, this does not solve our current problem. To convert a complex type into a set of rows (one per value in our array), we need to use the explode function.

但是，这不能解决我们当前的问题。 要将复杂类型转换为一组行（数组中的每个值一个），我们需要使用explode函数。

### <font color="#00000">explode 展开</font>

The explode function takes a column that consists of arrays and creates one row (with the rest of the values duplicated) per value in the array. Figure 6-1 illustrates the process. 

explode函数采用由数组组成的列，并为数组中的每个值创建一行（其余值重复）。 图6-1说明了该过程。

![1572357568223](C:/Users/ruito/AppData/Roaming/Typora/typora-user-images/1572357568223.png)

```scala
// in Scala
import org.apache.spark.sql.functions.{split, explode}
df.withColumn("splitted", split(col("Description"), " "))
.withColumn("exploded", explode(col("splitted")))
.select("Description", "InvoiceNo", "exploded").show(2)
```

```python
# in Python
from pyspark.sql.functions import split, explode
df.withColumn("splitted", split(col("Description"), " "))\
.withColumn("exploded", explode(col("splitted")))\
.select("Description", "InvoiceNo", "exploded").show(2)
```

```sql
-- in SQL
SELECT Description, InvoiceNo, exploded
FROM (SELECT *, split(Description, " ") as splitted FROM dfTable)
LATERAL VIEW explode(splitted) as exploded
```
This gives us the following result:

这给我们以下结果：

```shell
+--------------------+---------+--------+
|    Description     |InvoiceNo|exploded|
+--------------------+---------+--------+
|WHITE HANGING HEA...|  536365 |  WHITE |
|WHITE HANGING HEA...|  536365 | HANGING|
+--------------------+---------+--------+
```

### <font color="#00000">Maps 映射</font>

Maps are created by using the map function and key-value pairs of columns. You then can select them just like you might select from an array: 

使用映射函数和列的键值对创建映射。 然后，您可以像从数组中选择一样选择它们：

```scala
// in Scala
import org.apache.spark.sql.functions.map
df.select(map(col("Description"), col("InvoiceNo")).alias("complex_map")).show(2)
```

```python
# in Python
from pyspark.sql.functions import create_map
df.select(create_map(col("Description"), col("InvoiceNo")).alias("complex_map"))\
.show(2)
```

```sql
-- in SQL
SELECT map(Description, InvoiceNo) as complex_map FROM dfTable
WHERE Description IS NOT NULL 
```

This produces the following result: 

这将产生以下结果：

```shell
+--------------------+
|    complex_map     |
+--------------------+
|Map(WHITE HANGING...|
|Map(WHITE METAL L...|
+--------------------+
```

You can query them by using the proper key. A missing key returns null:

您可以使用适当的键查询它们。 缺少键将返回null：

```scala
// in Scala
df.select(map(col("Description"), col("InvoiceNo")).alias("complex_map"))
.selectExpr("complex_map['WHITE METAL LANTERN']").show(2)
```

```python
# in Python
df.select(map(col("Description"), col("InvoiceNo")).alias("complex_map"))\
.selectExpr("complex_map['WHITE METAL LANTERN']").show(2)
```

This gives us the following result:

这给我们以下结果：

```shell
+--------------------------------+
|complex_map[WHITE METAL LANTERN]|
+--------------------------------+
|          null                  |
|          536365                |
+--------------------------------+
```

You can also explode map types, which will turn them into columns:

您还可以展开映射类型，这会将它们转换为列：

```scala
// in Scala
df.select(map(col("Description"), col("InvoiceNo")).alias("complex_map"))
.selectExpr("explode(complex_map)").show(2)
```

```python
# in Python
df.select(map(col("Description"), col("InvoiceNo")).alias("complex_map"))\
.selectExpr("explode(complex_map)").show(2)
```

This gives us the following result:

这给我们以下结果：

```shell
+--------------------+------+
|         key        | value|
+--------------------+------+
|WHITE HANGING HEA...|536365|
| WHITE METAL LANTERN|536365|
+--------------------+------+
```

### <font color="#00000">Working with JSON 使用JSON</font>

Spark has some unique support for working with JSON data. You can operate directly on strings of JSON in Spark and parse from JSON or extract JSON objects. Let’s begin by creating a JSON column:

Spark对使用JSON数据提供了一些独特的支持。 您可以直接在Spark中对JSON字符串进行操作，并从JSON进行解析或提取JSON对象。 首先创建一个JSON列：

```scala
// in Scala
val jsonDF = spark.range(1).selectExpr("""
'{"myJSONKey" : {"myJSONValue" : [1, 2, 3]}}' as jsonString""")
```

```python
# in Python
jsonDF = spark.range(1).selectExpr("""
'{"myJSONKey" : {"myJSONValue" : [1, 2, 3]}}' as jsonString""")
```

You can use the get_json_object to inline query a JSON object, be it a dictionary or array. You can use json_tuple if this object has only one level of nesting:

您可以使用 `get_json_object` 内联查询 `JSON` 对象（无论是字典还是数组）。 如果此对象只有一层嵌套，则可以使用 `json_tuple`：

```scala
// in Scala
import org.apache.spark.sql.functions.{get_json_object, json_tuple}
jsonDF.select(
get_json_object(col("jsonString"), "$.myJSONKey.myJSONValue[1]") as "column",
json_tuple(col("jsonString"), "myJSONKey")).show(2)
```

```python
# in Python
from pyspark.sql.functions import get_json_object, json_tuple
jsonDF.select(
get_json_object(col("jsonString"), ".myJSONKey.myJSONValue[1]") as "column",
json_tuple(col("jsonString"), "myJSONKey")).show(2)
```

Here’s the equivalent in SQL : 

```sql
jsonDF.selectExpr(
"json_tuple(jsonString, '.myJSONKey.myJSONValue[1]') as column").show(2)
```

This results in the following table:

结果如下表所示：

```txt
+------+--------------------+
|column|        c0          |
+------+--------------------+
|  2   |{"myJSONValue":[1...|
+------+--------------------+
```

You can also turn a StructType into a JSON string by using the to_json function:

您还可以使用 `to_json` 函数将 `StructType` 转换为 `JSON` 字符串：

```scala
// in Scala
import org.apache.spark.sql.functions.to_json
df.selectExpr("(InvoiceNo, Description) as myStruct")
.select(to_json(col("myStruct")))
```

```python
# in Python
from pyspark.sql.functions import to_json
df.selectExpr("(InvoiceNo, Description) as myStruct")\
.select(to_json(col("myStruct")))
```

This function also accepts a dictionary (map) of parameters that are the same as the JSON data source. You can use the from_json function to parse this (or other JSON data) back in. This naturally requires you to specify a schema, and optionally you can specify a map of options, as well:

此函数还接受与JSON数据源相同的参数字典（映射）。 您可以使用from_json函数将其（或其他JSON数据）解析回去。这自然要求您指定一个模式，并且还可以指定一个的选项映射：

```scala
// in Scala
import org.apache.spark.sql.functions.from_json
import org.apache.spark.sql.types._
val parseSchema = new StructType(Array(
new StructField("InvoiceNo",StringType,true),
new StructField("Description",StringType,true)))
df.selectExpr("(InvoiceNo, Description) as myStruct")
.select(to_json(col("myStruct")).alias("newJSON"))
.select(from_json(col("newJSON"), parseSchema), col("newJSON")).show(2)
```

```python
# in Python
from pyspark.sql.functions import from_json
from pyspark.sql.types import *
parseSchema = StructType((
StructField("InvoiceNo",StringType(),True),
StructField("Description",StringType(),True)))
df.selectExpr("(InvoiceNo, Description) as myStruct")\
.select(to_json(col("myStruct")).alias("newJSON"))\
.select(from_json(col("newJSON"), parseSchema), col("newJSON")).show(2)
```

This gives us the following result:

这给我们以下结果：

```shell
+----------------------+--------------------+
|jsontostructs(newJSON)|       newJSON      |
+----------------------+--------------------+
| [536365,WHITE HAN... |{"InvoiceNo":"536...|
| [536365,WHITE MET... |{"InvoiceNo":"536...|
+----------------------+--------------------+
```

## <font color="#9a161a">User-Defined Functions 用户定义的函数</font>

One of the most powerful things that you can do in Spark is define your own functions. These user-defined functions (UDFs) make it possible for you to write your own custom transformations using Python or Scala and even use external libraries. UDFs can take and return one or more columns as input. Spark UDFs are incredibly powerful because you can write them in several different programming languages; you do not need to create them in an esoteric format or domain-specific language. They’re just functions that operate on the data, record by record. By default, these functions are registered as temporary functions to be used in that specific SparkSession or Context.

您可以在Spark中执行的最强大的功能之一就是定义自己的函数。这些用户定义函数（UDF）使您可以使用Python或Scala甚至使用外部库来编写自己的自定义转换。 UDF可以接受并返回一列或多列作为输入。 Spark UDF非常强大，因为您可以用几种不同的编程语言编写它们。您无需以深奥的格式或特定于域的语言创建它们。它们只是对数据进行操作的功能，逐条记录。**默认情况下，这些功能被注册为在该特定SparkSession或Context中使用的临时功能**。

Although you can write UDFs in Scala, Python, or Java, there are performance considerations that you should be aware of. To illustrate this, we’re going to walk through exactly what happens when you create UDF, pass that into Spark, and then execute code using that UDF. 

尽管您可以使用Scala，Python或Java编写UDF，但仍应注意一些性能注意事项。为了说明这一点，我们将详细介绍创建UDF，将其传递给Spark并使用该UDF执行代码时发生的情况。

The first step is the actual function. We’ll create a simple one for this example. Let’s write a power3 function that takes a number and raises it to a power of three:

第一步是实际功能。我们将为此示例创建一个简单的示例。让我们编写一个power3函数，该函数接受一个数字并将其提高为三的幂：

```scala
// in Scala
val udfExampleDF = spark.range(5).toDF("num")
def power3(number:Double):Double = number * number * number
power3(2.0)
```

```python
# in Python
udfExampleDF = spark.range(5).toDF("num")
def power3(double_value):
return double_value ** 3
power3(2.0)
```

In this trivial example, we can see that our functions work as expected. We are able to provide an individual input and produce the expected result (with this simple test case). Thus far, our expectations for the input are high: it must be a specific type and cannot be a null value (see “Working with Nulls in Data”).

在这个简单的示例中，我们可以看到我们的功能按预期工作。我们能够提供单独的输入并产生预期的结果（使用这个简单的测试用例）。到目前为止，我们对输入的期望很高：它必须是特定类型，不能为空值（请参阅“在数据中使用空值”）。

Now that we’ve created these functions and tested them, we need to register them with Spark so that we can use them on all of our worker machines. Spark will serialize the function on the driver and transfer it over the network to all executor processes. This happens regardless of language.

现在我们已经创建了这些功能并对其进行了测试，我们需要在Spark上注册它们，以便可以在所有工作计算机上使用它们。 Spark将序列化驱动程序上的函数，并将其通过网络传输到所有执行程序进程。无论使用哪种语言，都会发生这种情况。

When you use the function, there are essentially two different things that occur. If the function is written in Scala or Java, you can use it within the Java Virtual Machine (JVM). This means that there will be little performance penalty aside from the fact that you can’t take advantage of code generation capabilities that Spark has for built-in functions. There can be performance issues if you create or use a lot of objects; we cover that in the section on optimization in Chapter 19.

使用该函数时，实际上会发生两种不同的情况。如果该函数是用Scala或Java编写的，则可以在Java虚拟机（JVM）中使用它。这意味着除了您无法利用Spark内置函数的代码生成功能之外，几乎没有性能损失。如果创建或使用很多对象，可能会出现性能问题；我们将在第19章中的“优化”部分中进行介绍。

If the function is written in Python, something quite different happens. Spark starts a Python process on the worker, serializes all of the data to a format that Python can understand (remember, it was in the JVM earlier), executes the function row by row on that data in the Python process, and then finally returns the results of the row operations to the JVM and Spark. Figure 6-2 provides an overview of the process. 

如果该函数是用Python编写的，则会发生完全不同的事情。 Spark在工作程序上启动一个Python进程，将所有数据序列化为Python可以理解的格式（请记住，它早先在JVM中），在Python进程中逐行对该数据执行函数，然后最终返回将行操作的结果传递给JVM和Spark。图6-2概述了该过程。
![1572361414482](C:/Users/ruito/AppData/Roaming/Typora/typora-user-images/1572361414482.png)

---

<p><center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong></font></center></p>
Starting this Python process is expensive, but the real cost is in serializing the data to Python. This is costly for two reasons: it is an expensive computation, but also, after the data enters Python, Spark cannot manage the memory of the worker. This means that you could potentially cause a worker to fail if it becomes resource constrained (because both the JVM and Python are competing for memory on the same machine). We recommend that you write your UDFs in Scala or Java—the small amount of time it should take you to write the function in Scala will always yield significant speed ups, and on top of that, you can still use the function from Python! 

启动此Python进程非常昂贵，但实际成本是将数据序列化为Python。 这是昂贵的，原因有两个：这是昂贵的计算，而且，在数据输入Python之后，Spark无法管理工作程序的内存。 这意味着，如果工作程序受到资源限制，则有可能导致它失败（因为JVM和Python都在同一台机器上争夺内存）。 我们建议您使用Scala或Java编写UDF——用少量时间在Scala中编写函数将始终能够显着提高速度，最重要的是，您仍然可以使用Python中的函数！

---

Now that you have an understanding of the process, let’s work through an example. First, we need to register the function to make it available as a DataFrame function: 

现在您已经了解了该过程，下面以一个示例为例。首先，我们需要注册该函数以使其可用作DataFrame函数：

```scala
// in Scala
import org.apache.spark.sql.functions.udf
val power3udf = udf(power3(_:Double):Double)
```

We can use that just like any other DataFrame function: 

```scala
// in Scala
udfExampleDF.select(power3udf(col("num"))).show() 
```

The same applies to Python—first, we register it:

同样适用于Python——首先，我们注册它：

```python
# in Python
from pyspark.sql.functions import udf
power3udf = udf(power3)
```

Then, we can use it in our DataFrame code:

然后，我们可以在DataFrame代码中使用它：

```python
# in Python
from pyspark.sql.functions import col
udfExampleDF.select(power3udf(col("num"))).show(2)
```

```shell
+-----------+
|power3(num)|
+-----------+
|    0      |
|    1      |
+-----------+
```

At this juncture, we can use this only as a DataFrame function. That is to say, we can’t use it within a string expression, only on an expression. However, we can also register this UDF as a Spark SQL function. This is valuable because it makes it simple to use this function within SQL as well as across languages. Let’s register the function in Scala:

目前，我们只能将其用作DataFrame函数。 也就是说，我们**不能在字符串表达式中使用它，而只能在表达式中使用它。 但是，我们也可以将此UDF注册为Spark SQL函数。 这很有价值，因为它使在SQL以及跨语言中使用此功能变得简单**。 让我们在Scala中注册该功能：

```scala
// in Scala
spark.udf.register("power3", power3(_:Double):Double)
udfExampleDF.selectExpr("power3(num)").show(2)
```
Because this function is registered with Spark SQL—and we’ve learned that any Spark SQL function or expression is valid to use as an expression when working with DataFrames—we can turn around and use the UDF that we wrote in Scala, in Python. However, rather than using it as a DataFrame function, we use it as a SQL expression:

由于此函数已在Spark SQL中注册——并且我们了解到，在使用DataFrames时，任何Spark SQL函数或表达式都可有效地用作表达式——我们可以转而使用Scala或用Python编写的UDF。 但是，不是将其用作DataFrame函数，而是将其用作SQL表达式：

```python
# in Python
udfExampleDF.selectExpr("power3(num)").show(2)
# registered in Scala
```

We can also register our Python function to be available as a SQL function and use that in any language, as well.

我们还可以注册Python函数以将其作为SQL函数使用，也可以在任何语言中使用它。

One thing we can also do to ensure that our functions are working correctly is specify a return type. As we saw in the beginning of this section, Spark manages its own type information, which does not align exactly with Python’s types. Therefore, it’s a best practice to define the return type for your function when you define it. It is important to note that specifying the return type is not necessary, but it is a best practice.

为了确保我们的功能正常运行，我们还可以做的一件事就是指定返回类型。 正如我们在本节开头所看到的，Spark管理自己的类型信息，该信息与Python的类型不完全一致。 因此，最佳做法是在定义函数时定义返回类型。 重要的是要注意，没有必要指定返回类型，但这是最佳实践。

If you specify the type that doesn’t align with the actual type returned by the function, Spark will not throw an error but will just return null to designate a failure. You can see this if you were to switch the return type in the following function to be a DoubleType:

如果您指定的类型与该函数返回的实际类型不符，Spark将不会抛出错误，而只会返回null来表示失败。如果要在以下函数中将返回类型切换为DoubleType，则可以看到此信息：

```python
# in Python
from pyspark.sql.types import IntegerType, DoubleType
spark.udf.register("power3py", power3, DoubleType())
```
```python
# in Python
udfExampleDF.selectExpr("power3py(num)").show(2)
# registered via Python
```

This is because the `range` creates integers. When integers are operated on in Python, Python won’t convert them into floats (the corresponding type to Spark’s double type), therefore we see null. We can remedy this by ensuring that our Python function returns a float instead of an integer and the function will behave correctly.

这是因为 `range` 创建整数。 当在Python中对整数进行运算时，Python不会将其转换为浮点数（与Spark的double类型相对应的类型），因此我们会看到null。 我们可以通过确保Python函数返回浮点数而不是整数来补救此问题，并且该函数将正常运行。

Naturally, we can use either of these from SQL, too, after we register them:

自然地，在注册它们之后，我们也可以在SQL中使用它们之一：


```sql
-- in SQL
SELECT power3(12), power3py(12) --doesn't work because of return type
```

When you want to optionally return a value from a UDF, you should return None in Python and an Option type in Scala:

当您希望从UDF返回值时，应在Python中返回None，在Scala中返回Option类型：

## Hive UDFs

As a last note, you can also use UDF/UDAF creation via a Hive syntax. To allow for this, first you must enable Hive support when they create their `SparkSession` (via `SparkSession.builder().enableHiveSupport()` ). Then you can register UDFs in SQL. This is only supported with precompiled Scala and Java packages, so you’ll need to specify them as a dependency:

最后，您还可以通过Hive语法使用 UDF / UDAF 创建。 为此，首先在创建 SparkSession 时必须启用Hive的支持（通过 `SparkSession.builder().enableHiveSupport()` ）。 然后，您可以在SQL中注册UDF。 仅预编译的Scala和Java软件包支持此功能，因此您需要将它们指定为依赖项：

```sql
-- in SQL
CREATE TEMPORARY FUNCTION myFunc AS 'com.organization.hive.udf.FunctionName'
```

Additionally, you can register this as a permanent function in the Hive Metastore by removing TEMPORARY.

此外，您可以通过删除TEMPORARY将其注册为Hive Metastore中的永久函数。

## <font color="#9a161a">Conclusion 结论</font>

This chapter demonstrated how easy it is to extend Spark SQL to your own purposes and do so in a way that is not some esoteric, domain-specific language but rather simple functions that are easy to test and maintain without even using Spark! This is an amazingly powerful tool that you can use to specify sophisticated business logic that can run on five rows on your local machines or on terabytes of data on a 100-node cluster! 

本章展示了将Spark SQL扩展到自己的目的有多么容易，并且这样做不是某种深奥的，特定于领域的语言，而是一种简单的函数，即使不使用Spark也不容易测试和维护！ 这是一个非常强大的工具，可用于指定复杂的业务逻辑，这些逻辑可以在本地计算机上的五行上运行，也可以在100节点群集上的TB级数据上运行！

