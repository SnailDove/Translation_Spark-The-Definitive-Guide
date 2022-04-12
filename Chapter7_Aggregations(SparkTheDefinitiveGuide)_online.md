---
title: 翻译 Chapter 7. Aggregations
date: 2019-08-05
copyright: true
categories: English
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 7 Aggregations 聚合
<center><strong>译者</strong>：<u><a style="color:#0879e3" href="https://snaildove.github.io">https://snaildove.github.io</a></u></center>

Aggregating is the act of collecting something together and is a cornerstone of big data analytics. In an aggregation, you will specify a key or grouping and an aggregation function that specifies how you should transform one or more columns. This function must produce one result for each group, given multiple input values. Spark’s aggregation capabilities are sophisticated and mature, with a variety of different use cases and possibilities. In general, you use aggregations to summarize numerical data usually by means of some grouping. This might be a summation, a product, or simple counting. Also, with Spark you can aggregate any kind of value into an array, list, or map, as we will see in “Aggregating to Complex Types”.

聚合是将某物收集在一起的行为，是大数据分析的基石。在聚合中，您将指定一个键或分组以及一个聚合函数，该函数指定应如何转换一个或多个列。给定多个输入值，此函数必须为每个组产生一个结果。 Spark的聚合功能是复杂巧妙且成熟的，具有各种不同的用例和可能性。通常，通过分组使用聚合去汇总数值型数据。这可能是求和，乘积或简单的计数。另外，使用Spark可以将任何类型的值聚合到数组，列表或映射中，如我们在“聚合为复杂类型”中所见。

In addition to working with any type of values, Spark also allows us to create the following groupings types:

除了使用任何类型的值外，Sp​​ark还允许我们创建以下分组类型：


- The simplest grouping is to just summarize a complete DataFrame by performing an aggregation in a select statement.
最简单的分组是通过在select语句中执行聚合来汇总一个完整的DataFrame。
- A “group by” allows you to specify one or more keys as well as one or more aggregation functions to transform the value columns.
“分组依据”允许您指定一个或多个键以及一个或多个聚合函数来转换值列。
- A “window” gives you the ability to specify one or more keys as well as one or more aggregation functions to transform the value columns. However, the rows input to the function are somehow related to the current row.
“窗口”使您能够指定一个或多个键以及一个或多个聚合函数来转换值列。但是，输入到函数的行以某种方式与当前行相关。
- A “grouping set,” which you can use to aggregate at multiple different levels. Grouping sets are available as a primitive in SQL and via rollups and cubes in DataFrames.
一个“分组集”，可用于在多个不同级别进行汇总。分组集可作为SQL中的原语以及通过DataFrames中的 rollup 和 cube 使用。
- A “rollup” makes it possible for you to specify one or more keys as well as one or more aggregation functions to transform the value columns, which will be summarized hierarchically. 
“rollup”使您可以指定一个或多个键以及一个或多个聚合函数来转换值列，这些列将按层次进行汇总。
- A “cube” allows you to specify one or more keys as well as one or more aggregation functions to transform the value columns, which will be summarized across all combinations of columns.
“cube”允许您指定一个或多个键以及一个或多个聚合函数来转换值列，这些列将在所有列的组合中汇总。

Each grouping returns a `RelationalGroupedDataset` on which we specify our aggregations. 

每个分组都返回一个 `RelationalGroupedDataset`，在上面我们指定聚合。

---

<p><center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong></font></center></P>
An important thing to consider is how exact you need an answer to be. When performing calculations over big data, it can be quite expensive to get an exact answer to a question, and it’s often much cheaper to simply request an approximate to a reasonable degree of accuracy. You’ll note that we mention some approximation functions throughout the book and oftentimes this is a good opportunity to improve the speed and execution of your Spark jobs, especially for interactive and ad hoc analysis. 

要考虑的重要事项是您需要答案的精确程度。 在对大数据进行计算时，获得问题的准确答案可能会非常昂贵，而简单地请求一个近似的准确度通常会便宜得多。 您会注意到，我们在整本书中都提到了一些近似函数，通常这是一个提高Spark作业的速度和执行速度的好机会，尤其是对于交互式和临时安排的分析而言。

---

Let’s begin by reading in our data on purchases, repartitioning the data to have far fewer partitions (because we know it’s a small volume of data stored in a lot of small files), and caching the results for rapid access:

首先，我们读取购买数据，将数据重新分区为更少的分区（因为我们知道存储在许多小文件中的数据量很小），并缓存结果以进行快速访问： 

```python
// in Scala
val df = spark.read.format("csv")
.option("header", "true")
.option("inferSchema", "true")
.load("/data/retail-data/all/*.csv")
.coalesce(5)
df.cache()
df.createOrReplaceTempView("dfTable")
```

```python
# in Python
df = spark.read.format("csv")\
.option("header", "true")\
.option("inferSchema", "true")\
.load("/data/retail-data/all/*.csv")\
.coalesce(5)
df.cache()
df.createOrReplaceTempView("dfTable")
```

Here’s a sample of the data so that you can reference the output of some of the functions: 

这是数据示例，因此您可以引用某些函数的输出：

```txt
+---------+---------+--------------------+--------+--------------+---------+-----
|InvoiceNo|StockCode|    Description     |Quantity| InvoiceDate  |UnitPrice|Cu...
+---------+---------+--------------------+--------+--------------+---------+-----
|  536365 |  85123A | WHITE HANGING...   |   6    |12/1/2010 8:26|   2.55  | ...
|  536365 |  71053  | WHITE METAL...     |   6    |12/1/2010 8:26|   3.39  | ...
...
|  536367 |  21755  |LOVE BUILDING BLO...|   3    |12/1/2010 8:34|   5.95  | ...
|  536367 |  21777  |RECIPE BOX WITH M...|   4    |12/1/2010 8:34|   7.95  | ...
+---------+---------+--------------------+--------+--------------+---------+-----
```

As mentioned, basic aggregations apply to an entire DataFrame. The simplest example is the count method:

如前所述，基本聚合适用于整个DataFrame。 最简单的示例是count方法：

```scala
df.count() == 541909
```

If you’ve been reading this book chapter by chapter, you know that count is actually an action as opposed to a transformation, and so it returns immediately. You can use count to get an idea of the total size of your dataset but another common pattern is to use it to cache an entire DataFrame in memory, just like we did in this example.

如果您已经逐章阅读过本书，那么您就会知道，计数实际上是一种动作（action），而不是一种转换（transformation），因此它会立即返回。您可以使用count来了解数据集的总大小，但是另一个常见的模式是使用它来将整个DataFrame缓存在内存中，就像我们在本示例中所做的那样。


Now, this method is a bit of an outlier because it exists as a method (in this case) as opposed to a function and is eagerly evaluated instead of a lazy transformation. In the next section, we will see count used as a lazy function, as well.

现在，此方法有点离群值，因为它作为一种方法存在（在这种情况下）而不是函数，并且急切地被求值而不是延迟转换。在下一节中，我们还将看到count也用作惰性函数。

## <font color="#9a161a">Aggregation Functions 聚合函数</font>

All aggregations are available as functions, in addition to the special cases that can appear on DataFrames or via  `.stat` , like we saw in Chapter 6. You can find most aggregation functions in the `org.apache.spark.sql.functions` package. 

除了可以在DataFrames上或通过 `.stat` 出现的特殊情况（如我们在第6章中看到的）之外，所有聚合都可用作函数。您可以在 `org.apache.spark.sql.functions` 包中找到大多数聚合函数。

---

<p><center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong></font></center></P>
There are some gaps between the available SQL functions and the functions that we can import in Scala and Python. This changes every release, so it’s impossible to include a definitive list. This section covers the most common functions. 

可用的SQL函数与我们可以在Scala和Python中导入的函数之间存在一些差距。每个发行版都会更改，因此不可能包含确定的列表。本节介绍最常用的功能。

---

### <font color="#00000">count 计数</font>

The first function worth going over is count, except in this example it will perform as a transformation instead of an action. In this case, we can do one of two things: specify a specific column to count, or all the columns by using `count(*)` or `count(1)` to represent that we want to count every row as the literal one, as shown in this example:

值得复习的第一个功能是计数，除了在此示例中，它将作为转换而不是动作执行。 在这种情况下，我们可以执行以下两项操作之一：指定要计数的特定列，或者使用 `count(*)` 或 `count(1)` 表示要将每一行都视为文字行来指定所有列，如图所示 在此示例中：

```scala
// in Scala
import org.apache.spark.sql.functions.count
df.select(count("StockCode")).show() // 541909
```

```python
# in Python
from pyspark.sql.functions import count
df.select(count("StockCode")).show() # 541909
```

```sql
-- in SQL
SELECT COUNT(*) FROM dfTable
```

---

<p><center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong></font></center></P>
There are a number of gotchas when it comes to null values and counting. For instance, when performing a `count(*)`, Spark will count null values (including rows containing all nulls). However, when counting an individual column, Spark will not count the null values.

当涉及到空值和计数时，有很多陷阱。 例如，当执行 `count(*)` 时，Spark将计算空值（包括包含所有空值的行）。 但是，当计算单个列时，Spark将不计算空值。

---

### <font color="#00000">countDistinct</font>

Sometimes, the total number is not relevant; rather, it’s the number of unique groups that you want. To get this number, you can use the `countDistinct` function. This is a bit more relevant for individual columns:

有时，总数并不重要； 而是您想要的不重复的组的数目。 要获得此数字，可以使用 `countDistinct` 函数。 这与各个列更相关：

```scala
// in Scala
import org.apache.spark.sql.functions.countDistinct
df.select(countDistinct("StockCode")).show() // 4070
```
```python
# in Python
from pyspark.sql.functions import countDistinct
df.select(countDistinct("StockCode")).show() # 4070
```
```
-- in SQL
SELECT COUNT(DISTINCT *) FROM DFTABLE
```
### <font color="#00000">approx_count_distinct</font>

Often, we find ourselves working with large datasets and the exact distinct count is irrelevant. There are times when an approximation to a certain degree of accuracy will work just fine, and for that, you can use the `approx_count_distinct` function:

通常，我们发现自己正在处理大型数据集，而确切的不同数量无关紧要。 有时，达到某种程度的精确度就可以了，为此，您可以使用 `approx_count_distinct` 函数：

```scala
// in Scala
import org.apache.spark.sql.functions.approx_count_distinct
df.select(approx_count_distinct("StockCode", 0.1)).show() // 3364
```
```python
# in Python
from pyspark.sql.functions import approx_count_distinct
df.select(approx_count_distinct("StockCode", 0.1)).show() # 3364
```
```sql
-- in SQL
SELECT approx_count_distinct(StockCode, 0.1) FROM DFTABLE
```

You will notice that `approx_count_distinct` took another parameter with which you can specify the maximum estimation error allowed. In this case, we specified a rather large error and thus receive an answer that is quite far off but does complete more quickly than `countDistinct`. You will see much greater performance gains with larger datasets.

您会注意到，`approx_count_distinct` 使用了另一个参数，您可以使用该参数指定允许的最大估计误差。 在这种情况下，我们指定了一个相当大的错误，因此得到的答案相差很远，但是比 `countDistinct` 完成得更快。 使用更大的数据集，您将看到更大的性能提升。

### <font color="#00000">first and last</font>

You can get the first and last values from a DataFrame by using these two obviously named functions. This will be based on the rows in the DataFrame, not on the values in the DataFrame:

您可以通过使用这两个明显的命名函数得到从 DataFrame  的第一和最后一个值。这将基于在 DataFrame  中的行，而不是在 DataFrame  的值：

```scala
// in Scala
import org.apache.spark.sql.functions.{first, last}
df.select(first("StockCode"), last("StockCode")).show()
```

```scala
# in Python
from pyspark.sql.functions import first, last
df.select(first("StockCode"), last("StockCode")).show()
```

```sql
-- in SQL
SELECT first(StockCode), last(StockCode) FROM dfTable
```

```txt
+-----------------------+----------------------+
|first(StockCode, false)|last(StockCode, false)|
+-----------------------+----------------------+
|       85123A          |       22138          |
+-----------------------+----------------------+
```

### <font color="#00000">min and max</font>

To extract the minimum and maximum values from a DataFrame, use the min and max functions:

要从DataFrame中提取最小值和最大值，请使用min和max函数：

```scala
// in Scala
import org.apache.spark.sql.functions.{min, max}
df.select(min("Quantity"), max("Quantity")).show()
```

```python
# in Python
from pyspark.sql.functions import min, max
df.select(min("Quantity"), max("Quantity")).show()
```

```sql
-- in SQL
SELECT min(Quantity), max(Quantity) FROM dfTable
```

```txt
+-------------+-------------+
|min(Quantity)|max(Quantity)|
+-------------+-------------+
|    -80995   |    80995    |
+-------------+-------------+
```

### <font color="#00000">sum</font>

Another simple task is to add all the values in a row using the sum function:

另一个简单的任务是使用sum函数将所有值连续添加：

```scala
// in Scala
import org.apache.spark.sql.functions.sum
df.select(sum("Quantity")).show() // 5176450
```

```python
# in Python
from pyspark.sql.functions import sum
df.select(sum("Quantity")).show() # 5176450
```

```sql
-- in SQL
SELECT sum(Quantity) FROM dfTable
```

### <font color="#00000">sumDistinct</font>

In addition to summing a total, you also can sum a distinct set of values by using the `sumDistinct` function:

除了求和外，还可以使用 `sumDistinct` 函数对一组不同的值求和：

```scala
// in Scala
import org.apache.spark.sql.functions.sumDistinct
df.select(sumDistinct("Quantity")).show() // 29310
```

```python
# in Python
from pyspark.sql.functions import sumDistinct
df.select(sumDistinct("Quantity")).show() # 29310
```

```sql
-- in SQL
SELECT SUM(Quantity) FROM dfTable -- 29310
```

### <font color="#00000">avg</font>

Although you can calculate average by dividing sum by count, Spark provides an easier way to get that value via the `avg` or `mean` functions. In this example, we use `alias` in order to more easily reuse these columns later:

尽管您可以通过将总和除以计数来计算平均值，但是Spark提供了一种更简单的方法，可以通过 `avg`  或 `mean`  函数获取该值。 在此示例中，我们使用 `alias`，以便以后更轻松地重用这些列：

```scala
// in Scala
import org.apache.spark.sql.functions.{sum, count, avg, expr}
df.select(
count("Quantity").alias("total_transactions"),
sum("Quantity").alias("total_purchases"),
avg("Quantity").alias("avg_purchases"),
expr("mean(Quantity)").alias("mean_purchases"))
.selectExpr(
"total_purchases/total_transactions",
"avg_purchases",
"mean_purchases").show()
```

```python
# in Python
from pyspark.sql.functions import sum, count, avg, expr
df.select(
count("Quantity").alias("total_transactions"),
sum("Quantity").alias("total_purchases"),
avg("Quantity").alias("avg_purchases"),
expr("mean(Quantity)").alias("mean_purchases"))\
.selectExpr(
"total_purchases/total_transactions",
"avg_purchases",
"mean_purchases").show()
```

```txt
+--------------------------------------+----------------+----------------+
|(total_purchases / total_transactions)|  avg_purchases | mean_purchases |
+--------------------------------------+----------------+----------------+
|          9.55224954743324            |9.55224954743324|9.55224954743324|
+--------------------------------------+----------------+----------------+
```

---

<p><center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong></font></center></P>
You can also average all the distinct values by specifying distinct. In fact, most aggregate functions support doing so only on distinct values.

您还可以通过指定distinct将所有非重复值取平均值。 实际上，大多数聚合函数仅在不同的值上支持这样做。

---

### <font color="#00000">Variance and Standard Deviation 方差与标准差</font>

Calculating the mean naturally brings up questions about the variance and standard deviation. These are both measures of the spread of the data around the mean. The variance is the average of the squared differences from the mean, and the standard deviation is the square root of the variance. You can calculate these in Spark by using their respective functions. However, something to note is that Spark has both the formula for the sample standard deviation as well as the formula for the population standard deviation. These are fundamentally different statistical formulae, and we need to differentiate between them. By default, Spark performs the formula for the sample standard deviation or variance if you use the variance or `stddev` functions.

计算平均值自然会引起有关方差和标准偏差的问题。 这些都是衡量数据均值分布的方法。 方差是与平均值的平方差的平均值，标准差是方差的平方根。 您可以使用各自的功能在Spark中计算这些值。 但是，需要注意的是，Spark同时具有样本标准偏差的公式和总体标准偏差的公式。 这些是根本不同的统计公式，我们需要对其进行区分。 默认情况下，如果您使用方差或 `stddev` 函数，Spark将为样本标准差或方差执行公式。

You can also specify these explicitly or refer to the population standard deviation or variance:

 您还可以明确指定这些内容，或参考总体标准差或方差：
```scala
// in Scala
import org.apache.spark.sql.functions.{var_pop, stddev_pop}
import org.apache.spark.sql.functions.{var_samp, stddev_samp}
df.select(var_pop("Quantity"), var_samp("Quantity"),
stddev_pop("Quantity"), stddev_samp("Quantity")).show()
```

```python
# in Python
from pyspark.sql.functions import var_pop, stddev_pop
from pyspark.sql.functions import var_samp, stddev_samp
df.select(var_pop("Quantity"), var_samp("Quantity"),
stddev_pop("Quantity"), stddev_samp("Quantity")).show()
```

```sql
-- in SQL
SELECT var_pop(Quantity), var_samp(Quantity),
stddev_pop(Quantity), stddev_samp(Quantity)
FROM dfTable
```

```txt
+------------------+------------------+--------------------+-------------------+
| var_pop(Quantity)|var_samp(Quantity)|stddev_pop(Quantity)|stddev_samp(Quan...|
+------------------+------------------+--------------------+-------------------+
|47559.303646609056|47559.391409298754| 218.08095663447796 |  218.081157850... |
+------------------+------------------+--------------------+-------------------+
```

### <font color="#00000">skewness and kurtosis 偏度和峰度</font>

Skewness and kurtosis are both measurements of extreme points in your data. Skewness measures the asymmetry of the values in your data around the mean, whereas kurtosis is a measure of the tail of data. These are both relevant specifically when modeling your data as a probability distribution of a random variable. Although here we won’t go into the math behind these specifically, you can look up definitions quite easily on the internet. You can calculate these by using the functions:

偏度和峰度都是数据中极端值的度量。 偏斜度测量数据中均值前后的不对称性，而峰度则是对数据尾部的度量。 当将数据建模为随机变量的概率分布时，这两个都特别相关。 尽管这里我们不专门讨论这些内容，但您可以在因特网上轻松查找定义。 您可以使用以下函数来计算这些：

```scala
import org.apache.spark.sql.functions.{skewness, kurtosis}
df.select(skewness("Quantity"), kurtosis("Quantity")).show()
```

```python
# in Python
from pyspark.sql.functions import skewness, kurtosis
df.select(skewness("Quantity"), kurtosis("Quantity")).show()
```

```sql
-- in SQL
SELECT skewness(Quantity), kurtosis(Quantity) FROM dfTable
```

```scala
+-------------------+------------------+
| skewness(Quantity)|kurtosis(Quantity)|
+-------------------+------------------+
|-0.2640755761052562|119768.05495536952|
+-------------------+------------------+
```

### <font color="#00000">Covariance and Correlation 协方差和相关性</font>

We discussed single column aggregations, but some functions compare the interactions of the values in two difference columns together. Two of these functions are `cov` and `corr`, for covariance and correlation, respectively. Correlation measures the Pearson correlation coefficient, which is scaled between –1 and +1. The covariance is scaled according to the inputs in the data.

我们讨论了单列聚合，但是某些函数将两个不同列中的值的相互作用进行了比较。 其中两个函数分别是 `cov` 和 `corr`，分别用于协方差和相关性。 相关测量皮尔森相关系数，该系数在–1和+1之间缩放。 协方差根据数据中的输入进行缩放。


Like the `var` function, covariance can be calculated either as the sample covariance or the population covariance. Therefore it can be important to specify which formula you want to use. Correlation has no notion of this and therefore does not have calculations for population or sample. Here’s how they work:

像 `var` 函数一样，可以将协方差计算为样本协方差或总体协方差。因此，指定要使用的公式可能很重要。相关性不具有此概念，因此不具有总体或样本的计算。这里是他们是如何工作：

```scala
// in Scala
import org.apache.spark.sql.functions.{corr, covar_pop, covar_samp}
df.select(corr("InvoiceNo", "Quantity"), covar_samp("InvoiceNo", "Quantity"),
covar_pop("InvoiceNo", "Quantity")).show()
```

```python
# in Python
from pyspark.sql.functions import corr, covar_pop, covar_samp
df.select(corr("InvoiceNo", "Quantity"), covar_samp("InvoiceNo", "Quantity"),
covar_pop("InvoiceNo", "Quantity")).show()
```

```sql
-- in SQL
SELECT corr(InvoiceNo, Quantity), covar_samp(InvoiceNo, Quantity),
covar_pop(InvoiceNo, Quantity)
FROM dfTable
```

```txt
+-------------------------+-------------------------------+---------------------+
|corr(InvoiceNo, Quantity)|covar_samp(InvoiceNo, Quantity)|covar_pop(InvoiceN...|
+-------------------------+-------------------------------+---------------------+
|  4.912186085635685E-4   |      1052.7280543902734       |      1052.7...      |
+-------------------------+-------------------------------+---------------------+
```

### <font color="#00000">Aggregating to Complex Types 聚合为复杂类型</font>

In Spark, you can perform aggregations not just of numerical values using formulas, you can also perform them on complex types. For example, we can collect a list of values present in a given column or only the unique values by collecting to a set.

在Spark中，您不仅可以使用公式对数值进行汇总，还可以对复杂类型进行汇总。 例如，我们可以通过收集到一个集合来收集给定列中存在的值列表或仅收集唯一值。


You can use this to carry out some more programmatic access later on in the pipeline or pass the entire collection in a user-defined function (UDF):

您可以使用它在以后的管道中执行更多的编程访问，或者将整个集合传递给用户定义的函数（UDF）：

```scala
// in Scala
import org.apache.spark.sql.functions.{collect_set, collect_list}
df.agg(collect_set("Country"), collect_list("Country")).show()
```

```python
# in Python
from pyspark.sql.functions import collect_set, collect_list
df.agg(collect_set("Country"), collect_list("Country")).show()
```

```sql
-- in SQL
SELECT collect_set(Country), collect_set(Country) FROM dfTable
```

```txt
+--------------------+---------------------+
|collect_set(Country)|collect_list(Country)|
+--------------------+---------------------+
|[Portugal, Italy,...| [United Kingdom, ...|
+--------------------+---------------------+
```

## <font color="#9a161a">Grouping 分组</font>

Thus far, we have performed only DataFrame-level aggregations. A more common task is to perform calculations based on groups in the data. This is typically done on categorical data for which we group our data on one column and perform some calculations on the other columns that end up in that group.

到目前为止，我们仅执行了DataFrame级的聚合。 一个更常见的任务是基于数据中的组执行计算。 这通常是针对分类数据完成的，对于这些分类数据，我们将数据分组在一个列上，然后对最终归入该组的其他列执行一些计算。

The best way to explain this is to begin performing some groupings. The first will be a count, just as we did before. We will group by each unique invoice number and get the count of items on that invoice. Note that this returns another DataFrame and is lazily performed.

解释此问题的最佳方法是开始执行一些分组。就像我们之前所做的那样，第一个是计数。我们将按每个唯一的发票编号分组，并获取该发票上的项目数。请注意，这将返回另一个DataFrame并被延迟执行。

We do this grouping in two phases. First we specify the column(s) on which we would like to group, and then we specify the aggregation(s). The first step returns a `RelationalGroupedDataset`, and the second step returns a DataFrame.

我们分两个阶段进行分组。首先，我们指定要分组的列，然后指定聚合。第一步返回一个 `RelationalGroupedDataset`，第二步返回一个DataFrame。

As mentioned, we can specify any number of columns on which we want to group: 

如前所述，我们可以指定要分组的任意数量的列：

```scala
df.groupBy("InvoiceNo", "CustomerId").count().show()
```

```sql
-- in SQL
SELECT count(*) FROM dfTable GROUP BY InvoiceNo, CustomerId
```

```txt
+---------+----------+-----+
|InvoiceNo|CustomerId|count|
+---------+----------+-----+
|  536846 |  14573   |  76 |
...
| C544318 |  12989   |  1  |
+---------+----------+-----+
```

### <font color="#00000">Grouping with Expressions 用表达式分组</font>

As we saw earlier, counting is a bit of a special case because it exists as a method. For this, usually we prefer to use the count function. Rather than passing that function as an expression into a select statement, we specify it as within `agg`. This makes it possible for you to pass-in arbitrary expressions that just need to have some aggregation specified. You can even do things like alias a column after transforming it for later use in your data flow: 

如前所述，计数是一种特殊情况，因为它作为一种方法存在。 为此，通常我们更喜欢使用 `count` 函数。 与其将函数作为表达式传递到 select 语句中，不如在 `agg` 中指定它。 这使您可以传入只需要特定的一些聚合函数的任意表达式。 您甚至可以在对列进行转换后进行别名处理，以供以后在数据流中使用：

```scala
// in Scala
import org.apache.spark.sql.functions.count
df.groupBy("InvoiceNo").agg(
	count("Quantity").alias("quan"), expr("count(Quantity)")
).show()
```

```python
# in Python
from pyspark.sql.functions import count
df.groupBy("InvoiceNo").agg(
	count("Quantity").alias("quan"), expr("count(Quantity)")
).show()
```

```txt
+---------+----+---------------+
|InvoiceNo|quan|count(Quantity)|
+---------+----+---------------+
| 536596  |  6 |       6       |
...
| C542604 |  8 |       8       |
+---------+----+---------------+
```

### <font color="#00000">Grouping with Maps 用映射分组</font>

Sometimes, it can be easier to specify your transformations as a series of Maps for which the key is the column, and the value is the aggregation function (as a string) that you would like to perform. You can reuse multiple column names if you specify them inline, as well: 

有时，将转换指定为一系列 Map（以键为列，值是您要执行的聚合函数（作为字符串））会更容易。 如果您内联地（inline）指定了多个列名，则可以重用它们：

```scala
// in Scala
df.groupBy("InvoiceNo").agg("Quantity"->"avg", "Quantity"->"stddev_pop").show()
```

```python
# in Python
df.groupBy("InvoiceNo").agg(expr("avg(Quantity)"),expr("stddev_pop(Quantity)"))\
.show()
```

```sql
-- in SQL
SELECT avg(Quantity), stddev_pop(Quantity), InvoiceNo FROM dfTable
GROUP BY InvoiceNo
```

```txt
+---------+------------------+--------------------+
|InvoiceNo| avg(Quantity)    |stddev_pop(Quantity)|
+---------+------------------+--------------------+
| 536596  |      1.5         | 1.1180339887498947 |
...
| C542604 |      -8.0        | 15.173990905493518 |
+---------+------------------+--------------------+
```

## <font color="#9a161a">Window Functions 窗口函数</font>

You can also use window functions to carry out some unique aggregations by either computing some aggregation on a specific “window” of data, which you define by using a reference to the current data. This window specification determines which rows will be passed in to this function. Now this is a bit abstract and probably similar to a standard group-by, so let’s differentiate them a bit more. 

您还可以使用窗口函数来执行某些唯一的聚合，方法是在特定的数据“窗口”上计算某些聚合，您可以使用对当前数据的引用来定义这些聚合。此窗口规范确定哪些行将传递到此函数。现在，这有点抽象，可能类似于标准分组方式，因此让我们对其进行更多区分。


A group-by takes data, and every row can go only into one grouping. A window function calculates a return value for every input row of a table based on a group of rows, called a frame. Each row can fall into one or more frames. A common use case is to take a look at a rolling average of some value for which each row represents one day. If you were to do this, each row would end up in seven different frames. We cover defining frames a little later, but for your reference, Spark supports three kinds of window functions: ranking functions, analytic functions, and aggregate functions. Figure 7-1 illustrates how a given row can fall into multiple frames. 

分组依据可以获取数据，并且每一行只能分为一组。对于由一组行（称为帧）组成的表，窗口函数根据每个输入行计算返回值。每一行可以分为一个或多个帧。一个常见的用例是查看某个值的滚动平均值，该值的每一行代表一天。如果要这样做，每一行将以七个不同的帧结束。稍后我们将介绍定义帧的步骤，但仅供参考，Spark支持三种窗口函数：排名函数，分析函数和聚合函数。图7-1说明了给定的行如何分成多个帧。

![1572538707601](C:/Users/ruito/AppData/Roaming/Typora/typora-user-images/1572538707601.png)

To demonstrate, we will add a date column that will convert our invoice date into a column that contains only date information (not time information, too):

为了说明这一点，我们将添加一个日期列，该列会将发票日期转换为仅包含日期信息（也不包含时间信息）的列：

```scala
// in Scala
import org.apache.spark.sql.functions.{col, to_date}
val dfWithDate = df.withColumn("date", to_date(col("InvoiceDate"), "MM/d/yyyy H:mm"))
dfWithDate.createOrReplaceTempView("dfWithDate")
```

```python
# in Python
from pyspark.sql.functions import col, to_date
dfWithDate = df.withColumn("date", to_date(col("InvoiceDate"), "MM/d/yyyy H:mm"))
dfWithDate.createOrReplaceTempView("dfWithDate")
```

The first step to a window function is to create a window specification. Note that the partition by is unrelated to the partitioning scheme concept that we have covered thus far. It’s just a similar concept that describes how we will be breaking up our group. The ordering determines the ordering within a given partition, and, finally, the frame specification (the rowsBetween statement) states which rows will be included in the frame based on its reference to the current input row. In the following example, we look at all previous rows up to the current row:

窗口函数的第一步是创建窗口规范。 注意，partition by 与到目前为止我们所讨论的分区方案概念无关。 这只是一个类似的概念，描述了我们将如何拆分小组。 排序确定给定分区内的排序，最后，帧规范（rowsBetween语句）根据对当前输入行的引用，说明哪些行将包含在帧中。 在以下示例中，我们查看直到当前行的所有先前行：

```scala
// in Scala
import org.apache.spark.sql.expressions.Window
import org.apache.spark.sql.functions.col
val windowSpec = Window
.partitionBy("CustomerId", "date")
.orderBy(col("Quantity").desc)
.rowsBetween(Window.unboundedPreceding, Window.currentRow) 
```

```python
# in Python
from pyspark.sql.window import Window
from pyspark.sql.functions import desc
windowSpec = Window\
.partitionBy("CustomerId", "date")\
.orderBy(desc("Quantity"))\
.rowsBetween(Window.unboundedPreceding, Window.currentRow)
```

Now we want to use an aggregation function to learn more about each specific customer. An example might be establishing the maximum purchase quantity over all time. To answer this, we use the same aggregation functions that we saw earlier by passing a column name or expression. In addition, we indicate the window specification that defines to which frames of data this function will apply: 

现在，我们要使用聚合函数来了解有关每个特定客户的更多信息。 一个示例可能是一直以来的最大购买数量。 为了回答这个问题，我们使用相同的聚合函数。 另外，我们指出了窗口规范，该规范定义了此功能将应用哪些数据帧：

```scala
import org.apache.spark.sql.functions.max
val maxPurchaseQuantity = max(col("Quantity")).over(windowSpec)

```

```python
# in Python
from pyspark.sql.functions import max
maxPurchaseQuantity = max(col("Quantity")).over(windowSpec)
```

You will notice that this returns a column (or expressions). We can now use this in a DataFrame select statement. Before doing so, though, we will create the purchase quantity rank. To do that we use the  `dense_rank` function to determine which date had the maximum purchase quantity for every customer. We use `dense_rank`  as opposed to rank to avoid gaps in the ranking sequence when there are tied values (or in our case, duplicate rows): 

您会注意到，这将返回一个列（或表达式）。 现在，我们可以在DataFrame select语句中使用它。 但是，在此之前，我们将创建采购数量等级。 为此，我们使用 `dense_rank` 函数来确定哪个日期的每个客户的购买数量最多。 当存在绑定值（或在我们的示例中为重复的行）时，我们使用 `dense_rank`  而不是 `rank` 来避免排名序列中的空白：

```scala
// in Scala
import org.apache.spark.sql.functions.{dense_rank, rank}
val purchaseDenseRank = dense_rank().over(windowSpec)
val purchaseRank = rank().over(windowSpec)
```

```python
# in Python
from pyspark.sql.functions import dense_rank, rank
purchaseDenseRank = dense_rank().over(windowSpec)
purchaseRank = rank().over(windowSpec)
```

This also returns a column that we can use in select statements. Now we can perform a select to view the calculated window values: 

这还会返回一个可在select语句中使用的列。 现在，我们可以执行选择以查看计算出的窗口值：

```scala
// in Scala
import org.apache.spark.sql.functions.col
dfWithDate.where("CustomerId IS NOT NULL").orderBy("CustomerId")
.select(
col("CustomerId"),
col("date"),
col("Quantity"),
purchaseRank.alias("quantityRank"),
purchaseDenseRank.alias("quantityDenseRank"),
maxPurchaseQuantity.alias("maxPurchaseQuantity")).show()
```

```python
# in Python
from pyspark.sql.functions import col
dfWithDate.where("CustomerId IS NOT NULL").orderBy("CustomerId")\
.select(
col("CustomerId"),
col("date"),
col("Quantity"),
purchaseRank.alias("quantityRank"),
purchaseDenseRank.alias("quantityDenseRank"),
maxPurchaseQuantity.alias("maxPurchaseQuantity")).show()
```

```sql
-- in SQL
SELECT CustomerId, date, Quantity,
	rank(Quantity) OVER (PARTITION BY CustomerId, date
						ORDER BY Quantity DESC NULLS LAST
                        ROWS BETWEEN
                        UNBOUNDED PRECEDING AND
                        CURRENT ROW) as rank,
    dense_rank(Quantity) OVER (PARTITION BY CustomerId, date
						ORDER BY Quantity DESC NULLS LAST
                        ROWS BETWEEN
                        UNBOUNDED PRECEDING AND
                        CURRENT ROW) as dRank,
    max(Quantity) OVER (PARTITION BY CustomerId, date
                        ORDER BY Quantity DESC NULLS LAST
                        ROWS BETWEEN
                        UNBOUNDED PRECEDING AND
                        CURRENT ROW) as maxPurchase
FROM dfWithDate WHERE CustomerId IS NOT NULL ORDER BY CustomerId
```

```txt
+----------+----------+--------+------------+-----------------+---------------+
|CustomerId|    date  |Quantity|quantityRank|quantityDenseRank|maxP...Quantity|
+----------+----------+--------+------------+-----------------+---------------+
| 12346    |2011-01-18| 74215  |   1        |    1            |      74215    |
| 12346    |2011-01-18| -74215 |   2        |    2            |      74215    |
| 12347    |2010-12-07|   36   |   1        |    1            |        36     |
| 12347    |2010-12-07|   30   |   2        |    2            |        36     |
...
| 12347    |2010-12-07|   12   |   4        |    4            |        36     |
| 12347    |2010-12-07|    6   |   17       |    5            |        36     |
| 12347    |2010-12-07|    6   |   17       |    5            |        36     |
+----------+----------+--------+------------+-----------------+---------------+
```

## <font color="#9a161a">Grouping Sets 分组集</font>

Thus far in this chapter, we’ve seen simple group-by expressions that we can use to aggregate on a set of columns with the values in those columns. However, sometimes we want something a bit more complete—an aggregation across multiple groups. We achieve this by using grouping sets. Grouping sets are a low-level tool for combining sets of aggregations together. They give you the ability to create arbitrary aggregation in their group-by statements.

到目前为止，在本章中，我们已经看到了简单的分组表达式，可用于将一组列中的值聚合在一起。但是，有时我们想要一些更完整的东西——跨多个组的汇总。我们通过使用分组集来实现。分组集是用于将聚合集组合在一起的低阶工具。它们使您能够在group-by语句中创建任意聚合。


Let’s work through an example to gain a better understanding. Here, we would like to get the total quantity of all stock codes and customers. To do so, we’ll use the following SQL expression: 

让我们通过一个例子来获得更好的理解。在这里，我们想获得所有股票代码和客户的总数。为此，我们将使用以下SQL表达式：

```scala
// in Scala
val dfNoNull = dfWithDate.drop()
dfNoNull.createOrReplaceTempView("dfNoNull")
```

```python
# in Python
dfNoNull = dfWithDate.drop()
dfNoNull.createOrReplaceTempView("dfNoNull")
```

```sql
-- in SQL
SELECT CustomerId, stockCode, sum(Quantity) FROM dfNoNull
GROUP BY customerId, stockCode
ORDER BY CustomerId DESC, stockCode DESC
```

You can do the exact same thing by using a grouping set:

您可以使用分组集来做完全相同的事情：

```sql
-- in SQL
SELECT CustomerId, stockCode, sum(Quantity) FROM dfNoNull
GROUP BY customerId, stockCode GROUPING SETS((customerId, stockCode))
ORDER BY CustomerId DESC, stockCode DESC
```

```txt
+----------+---------+-------------+
|CustomerId|stockCode|sum(Quantity)|
+----------+---------+-------------+
| 18287    | 85173   |    48       |
| 18287    | 85040A  |    48       |
| 18287    | 85039B  |    120      |
...
| 18287    | 23269   |    36       |
+----------+---------+-------------+
```

---

<p><center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong></font></center></p>
Grouping sets depend on null values for aggregation levels. If you do not filter-out null values, you will get incorrect results.

分组集取决于聚合级别的空值。 如果不筛选出空值，则会得到不正确的结果。

---

This applies to cubes, rollups, and grouping sets. Simple enough, but what if you also want to include the total number of items, regardless of customer or stock code? With a conventional group-by statement, this would be impossible. But, it’s simple with grouping sets: we simply specify that we would like to aggregate at that level, as well, in our grouping set. This is, effectively, the union of several different groupings together: 

这适用于  cube，rollup 和分组集。 很简单，但是如果您还想包括项目总数，而不管客户或库存代码如何呢？ 使用常规的分组声明，这将是不可能的。 但是，使用分组集很简单：我们只需在分组集中指定要在该级别进行汇总。 实际上，这是几个不同分组的结合：

```sql
-- in SQL
SELECT CustomerId, stockCode, sum(Quantity) FROM dfNoNull
GROUP BY customerId, stockCode GROUPING SETS((customerId, stockCode),())
ORDER BY CustomerId DESC, stockCode DESC
```

```txt
+----------+---------+-------------+
|customerId|stockCode|sum(Quantity)|
+----------+---------+-------------+
|  18287   | 85173   |    48       |
|  18287   | 85040A  |    48       |
|  18287   | 85039B  |    120      |
...
|  18287   | 23269   |    36       |
+----------+---------+-------------+
```

The GROUPING SETS operator is only available in SQL. To perform the same in DataFrames, you use the rollup and cube operators—which allow us to get the same results. Let’s go through those.

GROUPING SETS 运算符仅在 SQL 中可用。 要在DataFrames中执行相同的操作，请使用 rollup 和 cube 运算符——允许我们获得相同的结果。 让我们来看看这些。

### <font color="#00000">Rollups 汇总</font>

Thus far, we’ve been looking at explicit groupings. When we set our grouping keys of multiple columns, Spark looks at those as well as the actual combinations that are visible in the dataset. A rollup is a multidimensional aggregation that performs a variety of group-by style calculations for us. Let’s create a rollup that looks across time (with our new Date column) and space (with the Country column) and creates a new DataFrame that includes the grand total over all dates, the grand total for each date in the DataFrame, and the subtotal for each country on each date in the DataFrame: 

到目前为止，我们一直在研究显式分组。设置多列的分组键时，Spark会查看这些键以及数据集中可见的实际组合。rollup 是一个多维聚合，可以为我们执行各种分组样式计算。让我们创建一个跨时间（使用新的Date列）和空间（使用Country列）查看的汇总，并创建一个新的DataFrame，其中包括所有日期的总计，DataFrame中每个日期的总计以及小计对于每个国家/地区在DataFrame中的每个日期：

```scala
val rolledUpDF = dfNoNull.rollup("Date", "Country").agg(sum("Quantity"))
.selectExpr("Date", "Country", "`sum(Quantity)` as total_quantity")
.orderBy("Date")
rolledUpDF.show()
```

```python
# in Python
rolledUpDF = dfNoNull.rollup("Date", "Country").agg(sum("Quantity"))\
.selectExpr("Date", "Country", "`sum(Quantity)` as total_quantity")\
.orderBy("Date")
rolledUpDF.show()
```

```txt
+----------+--------------+--------------+
|   Date   |   Country    |total_quantity|
+----------+--------------+--------------+
|    null  |     null     |   5176450    |
|2010-12-01|United Kingdom|   23949      |
|2010-12-01|  Germany     |   117        |
|2010-12-01|  France      |   449        |
...
|2010-12-03|  France      |   239        |
|2010-12-03|   Italy      |   164        |
|2010-12-03|  Belgium     |   528        |
+----------+--------------+--------------+
```

Now where you see the null values is where you’ll find the grand totals. A null in both rollup columns specifies the grand total across both of those columns:

现在，您可以在其中找到空值的地方找到总计。 两个汇总列中的null指定这两个列的总计：

```scala
rolledUpDF.where("Country IS NULL").show()
rolledUpDF.where("Date IS NULL").show()
```

```txt
+----+-------+--------------+
|Date|Country|total_quantity|
+----+-------+--------------+
|null|   null| 5176450      |
+----+-------+--------------+ 
```

### <font color="#00000">Cube 多维数据集</font>

A cube takes the rollup to a level deeper. Rather than treating elements hierarchically, a cube does the same thing across all dimensions. This means that it won’t just go by date over the entire time period, but also the country. To pose this as a question again, can you make a table that includes the following? 

cube 将 rollup 扩展到更深的层次。 cube 不是在层次上处理元素，而是在所有维度上执行相同的操作。 这意味着它不仅会在整个时间段内按日期显示，而且还会在整个国家/地区显示。 再次提出这个问题，您可以制作一个包含以下内容的表格吗？

- The total across all dates and countries
所有日期和国家/地区的总计
- The total for each date across all countries
所有国家/地区每个日期的总计
- The total for each country on each date
每个国家/地区在每个日期的总数
- The total for each country across all dates 
所有国家/地区在所有日期的总计


The method call is quite similar, but instead of calling rollup, we call cube: 

方法调用非常相似，但是我们不调用 rollup，而是调用 cube：

```scala
// in Scala
dfNoNull.cube("Date", "Country").agg(sum(col("Quantity")))
.select("Date", "Country", "sum(Quantity)").orderBy("Date").show()
```

```python
# in Python
from pyspark.sql.functions import sum
dfNoNull.cube("Date", "Country").agg(sum(col("Quantity")))\
.select("Date", "Country", "sum(Quantity)").orderBy("Date").show()
```

```txt
+----+--------------------+-------------+
|Date|       Country      |sum(Quantity)|
+----+--------------------+-------------+
|null|         Japan      |    25218    |
|null|        Portugal    |    16180    |
|null|       Unspecified  |    3300     |
|null|           null     |    5176450  |
|null|        Australia   |    83653    |
...
|null|        Norway      |    19247    |
|null|      Hong Kong     |    4769     |
|null|        Spain       |    26824    |
|null|      Czech Republic|    592      |
+----+--------------------+-------------+
```

This is a quick and easily accessible summary of nearly all of the information in our table, and it’s a great way to create a quick summary table that others can use later on.

这是对我们表中几乎所有信息的快速且易于访问的摘要，也是创建其他人以后可以使用的快速摘要表的好方法。

### <font color="#00000">Grouping Metadata 分组元数据</font> 

Sometimes when using cubes and rollups, you want to be able to query the aggregation levels so that you can easily filter them down accordingly. We can do this by using the grouping_id, which gives us a column specifying the level of aggregation that we have in our result set. The query in the example that follows returns four distinct grouping IDs: 

有时，在使用多维数据集和汇总时，您希望能够查询聚合级别，以便可以轻松地相应地对其进行过滤。 我们可以使用grouping_id做到这一点，它为我们提供了一列，用于指定结果集中的聚合级别。 以下示例中的查询返回四个不同的分组ID：

Table 7-1. Purpose of grouping IDs 对ID分组的目的

| Grouping | ID Description  |
| :----------------------------------------------------------: | ------------------------------------------------------------ |
| 3 | This will appear for the highest-level aggregation, which will gives us the total quantity regardless of `customerId` and `stockCode`.  这将显示在最高级别的汇总中，无论 `customerId` 和 `stockCode` 如何，都将为我们提供总量。|
| 2 | This will appear for all aggregations of individual stock codes. This gives us the total quantity per stock code, regardless of customer. 这将显示在单个股票代码的所有汇总中。 这给了我们每个股票代码的总数量，而与客户无关。 |
| 1 | This will give us the total quantity on a per-customer basis, regardless of item purchased. 这将为我们提供每个客户的总数量，而与购买的商品无关。 |
| 0 | This will give us the total quantity for individual `customerId` and `stockCode` combinations. 这将为我们提供单个“ customerId”和“ stockCode”组合的总数量。|

This is a bit abstract, so it’s well worth trying out to understand the behavior yourself : 

这有点抽象，因此值得尝试自己了解一下行为：

```scala
// in Scala
import org.apache.spark.sql.functions.{grouping_id, sum, expr}
dfNoNull.cube("customerId", "stockCode").agg(grouping_id(), sum("Quantity"))
.orderBy(expr("grouping_id()").desc)
.show()
```

```txt
+----------+---------+-------------+-------------+
|customerId|stockCode|grouping_id()|sum(Quantity)|
+----------+---------+-------------+-------------+
|   null   |  null   |    3        |   5176450   |
|   null   |  23217  |    2        |   1309      |
|   null   |  90059E |    2        |     19      |
...
+----------+---------+-------------+-------------+
```

### <font color="#00000">Pivot </font>

Pivots make it possible for you to convert a row into a column. For example, in our current data we have a Country column. With a pivot, we can aggregate according to some function for each of those given countries and display them in an easy-to-query way: 

Pivot 使您可以将行转换为列。 例如，在当前数据中，我们有一个“国家”列。 有了枢轴（Pivot），我们可以针对每个给定国家/地区按照某种功能进行汇总，并以易于查询的方式显示它们：

```scala
// in Scala
val pivoted = dfWithDate.groupBy("date").pivot("Country").sum()
```

```python
# in Python
pivoted = dfWithDate.groupBy("date").pivot("Country").sum()
```

This DataFrame will now have a column for every combination of country, numeric variable, and a column specifying the date. For example, for USA we have the following columns: 

现在，此 DataFrame 将为国家、数字变量的每种组合提供一列，并为指定日期提供一列。例如，对于美国，我们有以下几列：

<p><center><font face="constant-width" color="#000000" size=3>USA_sum(Quantity), USA_sum(UnitPrice), USA_sum(CustomerID)</font></center></p>
This represents one for each numeric column in our dataset (because we just performed an aggregation over all of them). 

这代表了数据集中每个数字列的一个（因为我们只是对所有它们进行了汇总）。


Here’s an example query and result from this data: 

这是查询示例，并根据这些数据得出结果：

```scala
pivoted.where("date > '2011-12-05'").select("date" ,"`USA_sum(Quantity)`").show()
```

```txt
+----------+-----------------+
|   date   |USA_sum(Quantity)|
+----------+-----------------+
|2011-12-06|      null       |
|2011-12-09|      null       |
|2011-12-08|      -196       |
|2011-12-07|      null       |
+----------+-----------------+ 
```

Now all of the columns can be calculated with single groupings, but the value of a pivot comes down to how you would like to explore the data. It can be useful, if you have low enough cardinality in a certain column to transform it into columns so that users can see the schema and immediately know what to query for.

现在，可以使用单个分组来计算所有列，但是数据透视表的价值取决于您希望如何浏览数据。如果您在某个列中的基数小的足够将其转换为列，以便用户可以看到模式并立即知道要查询的内容，则此方法很有用。

## <font color="#9a161a">User-Defined Aggregation Functions 用户定义的聚合函数</font>

User-defined aggregation functions (UDAFs) are a way for users to define their own aggregation functions based on custom formulae or business rules. You can use UDAFs to compute custom calculations over groups of input data (as opposed to single rows). Spark maintains a single `AggregationBuffer` to store intermediate results for every group of input data.

用户定义的聚合函数（UDAF）是用户根据自定义公式或业务规则定义自己的聚合函数的一种方式。您可以使用UDAF在输入数据组（而不是单行）上计算自定义计算。 Spark维护一个 `AggregationBuffer` 来存储每组输入数据的中间结果。


To create a UDAF, you must inherit from the `UserDefinedAggregateFunction` base class and implement the following methods:

要创建UDAF，您必须继承自基类 `UserDefinedAggregateFunction` 并实现以下方法：

- `inputSchema` represents input arguments as a `StructType`
`inputSchema` 将输入参数表示为 `StructType`
- `bufferSchema ` represents intermediate UDAF results as a `StructType`
`bufferSchema` 将中间的UDAF结果表示为 `StructType`
- `dataType` represents the return `DataType`
`dataType` 表示返回的 `DataType`
- `deterministic` is a Boolean value that specifies whether this UDAF will return the same result for a given input
  `deterministic`  是一个布尔值，它指定此UDAF对于给定的输入是否将返回相同的结果
- `initialize` allows you to initialize values of an aggregation buffer
`initialize`  允许您初始化聚合缓冲区的值
- `update` describes how you should update the internal buffer based on a given row
`update`  描述了如何根据给定的行更新内部缓冲区
- `merge` describes how two aggregation buffers should be merged
`merge`  描述了如何合并两个聚合缓冲区
- `evaluate` will generate the final result of the aggregation
`evaluate`  将生成聚合的最终结果

The following example implements a BoolAnd, which will inform us whether all the rows (for a given column) are true; if they’re not, it will return false: 

下面的示例实现了一个BoolAnd，它将通知我们所有行（对于给定列）是否为true；如果不是，它将返回false：

```scala
// in Scala
import org.apache.spark.sql.expressions.MutableAggregationBuffer
import org.apache.spark.sql.expressions.UserDefinedAggregateFunction
import org.apache.spark.sql.Row
import org.apache.spark.sql.types._
class BoolAnd extends UserDefinedAggregateFunction {
    def inputSchema: org.apache.spark.sql.types.StructType =
        StructType(StructField("value", BooleanType) :: Nil)
    def bufferSchema: StructType = StructType(
        StructField("result", BooleanType) :: Nil
    ) 
    def dataType: DataType = BooleanType
    def deterministic: Boolean = true
    def initialize(buffer: MutableAggregationBuffer): Unit = {
    	buffer(0) = true
    } 
    def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
    	buffer(0) = buffer.getAs[Boolean](0) && input.getAs[Boolean](0)
    } 
    def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
    	buffer1(0) = buffer1.getAs[Boolean](0) && buffer2.getAs[Boolean](0)
    } 
    def evaluate(buffer: Row): Any = {
        buffer(0)
    }
}
```

Now, we simply instantiate our class and/or register it as a function: 

现在，我们只需实例化我们的类和/或将其注册为一个函数：

```scala
// in Scala
val ba = new BoolAnd
spark.udf.register("booland", ba)
import org.apache.spark.sql.functions._
spark.range(1)
.selectExpr("explode(array(TRUE, TRUE, TRUE)) as t")
.selectExpr("explode(array(TRUE, FALSE, TRUE)) as f", "t")
.select(ba(col("t")), expr("booland(f)"))
.show()
```

```txt
+----------+----------+
|booland(t)|booland(f)|
+----------+----------+
|   true   |  false   |
+----------+----------+
```

UDAFs are currently available only in Scala or Java. However, in Spark 2.3, you will also be able to call Scala or Java UDFs and UDAFs by registering the function just as we showed in the UDF section in Chapter 6. For more information, go to [SPARK-19439](https://issues.apache.org/jira/browse/SPARK-19439).

UDAF当前仅在Scala或Java中可用。 但是，在Spark 2.3中，您也可以通过注册函数来调用Scala或Java UDF和UDAF，就像我们在第6章UDF部分中所显示的那样。有关更多信息，请转到[SPARK-19439](https://issues.apache.org/jira/browse/SPARK-19439)。

## <font color="#9a161a">Conclusion 小结</font>

This chapter walked through the different types and kinds of aggregations that you can perform in Spark. You learned about simple grouping-to window functions as well as rollups and cubes. Chapter 8 discusses how to perform joins to combine different data sources together. 

本章介绍了可以在Spark中执行的不同类型的聚合。 您了解了简单的分组到窗口函数以及rollup和cube。 第8章讨论如何执行 join 以将不同的数据源组合在一起。