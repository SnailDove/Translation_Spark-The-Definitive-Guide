---
title: 翻译 Chapter 13 Advanced RDDs
date: 2019-11-07
copyright: true
categories: English
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 13. Advanced RDDs 

Chapter 12 explored the basics of single RDD manipulation. You learned how to create RDDs and why you might want to use them. In addition, we discussed map, filter, reduce, and how to create functions to transform single RDD data. This chapter covers the advanced RDD operations and focuses on key–value RDDs, a powerful abstraction for manipulating data. We also touch on some more advanced topics like custom partitioning, a reason you might want to use RDDs in the first place. With a custom partitioning function, you can control exactly how data is laid out on the cluster and manipulate that individual partition accordingly. Before we get there, let’s summarize the key topics we will cover:

第12章探讨了单个RDD操作的基础。您学习了如何创建RDD，以及为什么要使用它们。此外，我们讨论了 map，filter，reduce以及如何创建函数来转换单个RDD数据。本章介绍高级RDD操作，并重点介绍键值RDD，这是用于处理数据的强大抽象。我们还涉及一些更高级的主题，例如自定义分区，这是您可能首先要使用RDD的原因。使用自定义分区功能，您可以精确控制数据在群集上的布局方式，并相应地操作该单个分区。在到达那里之前，让我们总结一下我们将涉及的关键主题：

- Aggregations and key–value RDDs 聚合以及键值对RDD
- Custom partitioning 定制分区
- RDD joins RDD连接

---

<p><center><font face="constant-width" color="#737373" size=3><strong>NOTE 注意</strong></font></center></P>
This set of APIs has been around since, essentially, the beginning of Spark, and there are a ton of examples all across the web on this set of APIs. This makes it trivial to search and find examples that will show you how to use these operations.

从本质上讲，这是从Spark诞生以来就存在的这组API，并且在这组API上的网络上都有大量示例。这使搜索和查找示例向您展示如何使用这些操作变得很简单。

---

Let’s use the same dataset we used in the last chapter:

我们使用与上一章相同的数据集：

```scala
// in Scala
val myCollection = "Spark The Definitive Guide : Big Data Processing Made Simple".
split(" ")
val words = spark.sparkContext.parallelize(myCollection, 2)
```

```python
# in Python
myCollection = "Spark The Definitive Guide : Big Data Processing Made Simple"\
.split(" ")
words = spark.sparkContext.parallelize(myCollection, 2)
```

## <font color="#9a161a">Key-Value Basics (Key-Value RDDs)</font>

There are many methods on RDDs that require you to put your data in a key–value format. A hint that this is required is that the method will include `<some-operation>ByKey` . Whenever you see `ByKey` in a method name, it means that you can perform this only on a `PairRDD` type. The easiest way is to just map over your current RDD to a basic key–value structure. This means having two values in each record of your RDD:

RDD上有许多方法要求您将数据以键值格式存储。需要的提示是该方法将包含 `<some-operation>ByKey`。只要在方法名称中看到`ByKey`，就意味着您只能在`PairRDD`类型上执行此操作。最简单的方法是将当前的RDD映射到基本键值结构。这意味着在RDD的每个记录中都有两个值：

```scala
// in Scala
words.map(word => (word.toLowerCase, 1))
```

```python
# in Python
words.map(lambda word: (word.lower(), 1))
```

 ### <font color="#00000">keyBy</font>

The preceding example demonstrated a simple way to create a key. However, you can also use the `keyBy` function to achieve the same result by specifying a function that creates the key from your current value. In this case, you are keying by the first letter in the word. Spark then keeps the record as the value for the keyed RDD:

前面的示例演示了一种创建 key 的简单方法。但是，您还可以通过指定从当前值创建 key 的函数，使用 `keyBy` 函数获得相同的结果。在这种情况下，您将按单词中的第一个字母作为 key。然后，Spark将记录保留为键控RDD的值：

 ```scala
// in Scala
val keyword = words.keyBy(word => word.toLowerCase.toSeq(0).toString)
 ```

```python
# in Python
keyword = words.keyBy(lambda word: word.lower()[0])
```

### <font color="#00000">Mapping over Values</font>

After you have a set of key–value pairs, you can begin manipulating them as such. If we have a tuple, Spark will assume that the first element is the key, and the second is the value. When in this format, you can explicitly choose to map-over the values (and ignore the individual keys). Of course, you could do this manually, but this can help prevent errors when you know that you are just going to modify the values:

在拥有一组键值对之后，您就可以像这样操作它们。如果我们有一个元组，Spark将假定第一个元素是键，第二个是值。采用这种格式时，您可以明确选择映射值（并忽略各个键）。当然，您可以手动执行此操作，但是当您知道将要修改值时，这可以帮助防止错误：

```scala
// in Scala
keyword.mapValues(word => word.toUpperCase).collect()
```

```python
# in Python
keyword.mapValues(lambda word: word.upper()).collect()
```

Here’s the output in Python:

这是Python的输出：

```txt
[('s', 'SPARK'),
('t', 'THE'),
('d', 'DEFINITIVE'),
('g', 'GUIDE'),
(':', ':'),
('b', 'BIG'),
('d', 'DATA'),
('p', 'PROCESSING'),
('m', 'MADE'),
('s', 'SIMPLE')]
```

(The values in Scala are the same but omitted for brevity.)

Scala中的值相同，但为简洁起见，省略了它们。

You can `flatMap` over the rows, as we saw in Chapter 12, to expand the number of rows that you have to make it so that each row represents a character. In the following example, we will omit the output, but it would simply be each character as we converted them into arrays:

您可以对行进行 `flatMap`，如我们在第12章中看到的那样，以扩展必须包含的行数，以便每行代表一个字符。在下面的示例中，我们将省略输出，但是在将它们转换为数组时，将只是每个字符：

```scala
// in Scala
keyword.flatMapValues(word => word.toUpperCase).collect()
```

```python
# in Python
keyword.flatMapValues(lambda word: word.upper()).collect()
```

### <font color="#00000">Extracting Keys and Values</font>

When we are in the key–value pair format, we can also extract the specific keys or values by using the following methods:

当采用键值对格式时，我们还可以使用以下方法提取特定的键或值：

```scala
// in Scala
keyword.keys.collect()
keyword.values.collect()
```
```
# in Python
keyword.keys().collect()
keyword.values().collect()
```

### <font color="#00000">lookup</font>

One interesting task you might want to do with an RDD is look up the result for a particular key. Note that there is no enforcement mechanism with respect to there being only one key for each input, so if we lookup “s”, we are going to get both values associated with that—“Spark” and “Simple”:

您可能想对RDD进行的一项有趣的任务是查找特定 key 的结果。请注意，没有针对每个输入只有一个键的强制机制，因此，如果我们查找“ s”，我们将获得与此相关的两个值：“ Spark”和“ Simple”：

```scala
keyword.lookup("s")
```

```scala
Seq[String] = WrappedArray(Spark, Simple)
```

### <font color="#00000">sampleByKey</font>

There are two ways to sample an RDD by a set of keys. We can do it via an approximation or exactly. Both operations can do so with or without replacement as well as sampling by a fraction by a given key. This is done via simple random sampling with one pass over the RDD, which produces a sample of size that’s approximately equal to the sum of `math.ceil(numItems * samplingRate)` over all key values:

有两种方法可以通过一组 key 对RDD进行采样。我们可以通过近似或精确地做到这一点。两种操作都可以进行替换，也可以不进行替换，也可以通过给定的 key 按比例采样。这是通过对RDD进行一遍的简单随机抽样完成的，该抽样会产生一个样本，其大小近似等于所有关键值的 `math.ceil(numItems * samplingRate)` 之和：

```scala
// in Scala
val distinctChars = words.flatMap(word => word.toLowerCase.toSeq).distinct.collect()
import scala.util.Random
val sampleMap = distinctChars.map(c => (c, new Random().nextDouble())).toMap
words.map(word => (word.toLowerCase.toSeq(0), word))
.sampleByKey(true, sampleMap, 6L)
.collect()
```

```python
# in Python
import random
distinctChars = words.flatMap(lambda word: list(word.lower())).distinct().collect()
sampleMap = dict(map(lambda c: (c, random.random()), distinctChars))
words.map(lambda word: (word.lower()[0], word))\
.sampleByKey(True, sampleMap, 6).collect()
```

This method differs from `sampleByKey` in that you make additional passes over the RDD to create a sample size that’s exactly equal to the sum of `math.ceil(numItems * samplingRate)` over all key values with a 99.99% confidence. When sampling without replacement, you need one additional pass over the RDD to guarantee sample size; when sampling with replacement, you need two additional passes:

此方法 `sampleByKeyExact` 与 `sampleByKey` 的不同之处在于，您在RDD上进行了额外的遍历，以创建一个样本大小，该样本大小在99.99％的置信度下完全等于所有键值的 `math.ceil(numItems * samplingRate)` 之和。在不更换样本的情况下，您需要在RDD上再进行一次传递以确保样本量。在进行替换采样时，您需要另外两次传递：

```scala
// in Scala
words.map(word => (word.toLowerCase.toSeq(0), word))
.sampleByKeyExact(true, sampleMap, 6L).collect()
```

><center><strong>译者附</strong></center>
>这一块内容本书并没有讲清楚，属于分层抽样，详询：https://spark.apache.org/docs/latest/mllib-statistics.html#stratified-sampling

## <font color="#9a161a">Aggregations</font>

You can perform aggregations on plain RDDs or on PairRDDs, depending on the method that you are using. Let’s use some of our datasets to demonstrate this:

您可以在纯RDD或PairRDD上执行聚合，具体取决于您使用的方法。让我们使用一些数据集来证明这一点：

```scala
// in Scala
val chars = words.flatMap(word => word.toLowerCase.toSeq)
val KVcharacters = chars.map(letter => (letter, 1))
def maxFunc(left:Int, right:Int) = math.max(left, right)
def addFunc(left:Int, right:Int) = left + right
val nums = sc.parallelize(1 to 30, 5)
```

```python
# in Python
chars = words.flatMap(lambda word: word.lower())
KVcharacters = chars.map(lambda letter: (letter, 1))
def maxFunc(left, right):
	return max(left, right)
def addFunc(left, right):
	return left + right
nums = sc.parallelize(range(1,31), 5)
```

After you have this, you can do something like `countByKey`, which counts the items per each key.

完成此操作后，您可以执行诸如`countByKey`之类的操作，该操作对每个键的项目进行计数。

### <font color="#00000">countByKey</font>

You can count the number of elements for each key, collecting the results to a local Map. You can also do this with an approximation, which makes it possible for you to specify a timeout and confidence when using Scala or Java:

您可以计算每个键的元素数量，然后将结果收集到本地Map中。您也可以使用近似值来执行此操作，这使您可以在使用Scala或Java时指定超时和置信度：

```scala
// in Scalaval timeout = 1000L //milliseconds
val confidence = 0.95
KVcharacters.countByKey()
KVcharacters.countByKeyApprox(timeout, confidence)
```

```python
# in Python
KVcharacters.countByKey()
```

### <font color="#00000">Understanding Aggregation Implementations</font>

There are several ways to create your key–value `PairRDDs`; however, the implementation is actually quite important for job stability. Let’s compare the two fundamental choices, `groupBy` and `reduce`. We’ll do these in the context of a key, but the same basic principles apply to the `groupBy` and `reduce` methods.

有多种方法可以创建键值`PairRDD`。但是，实施对于提高工作稳定性实际上非常重要。让我们比较两个基本选择`groupBy`和`reduce`。我们将在键的上下文中进行这些操作，但是相同的基本原理也适用于`groupBy`和`reduce`方法。

#### <font color="#3399cc">groupByKey</font>

Looking at the API documentation, you might think `groupByKey` with a map over each grouping is the best way to sum up the counts for each key:

查看API文档，您可能会认为`groupByKey`和每个分组的映射是求和每个 key 计数的最佳方法：

```scala
// in Scala
KVcharacters.groupByKey().map(row => (row._1, row._2.reduce(addFunc))).collect()
```
```python
# in Python
KVcharacters.groupByKey().map(lambda row: (row[0], reduce(addFunc, row[1])))\
.collect()
# note this is Python 2, reduce must be imported from functools in Python 3
```

However, this is, for the majority of cases, the wrong way to approach the problem. The fundamental issue here is that each executor must hold all values for a given key in memory before applying the function to them. Why is this problematic? If you have massive key skew, some partitions might be completely overloaded with a ton of values for a given key, and you will get `OutOfMemoryErrors`. This obviously doesn’t cause an issue with our current dataset, but it can cause serious problems at scale. This is not guaranteed to happen, but it can happen.

但是，在大多数情况下，这是解决问题的错误方法。此处的根本问题是，每个执行程序应用于函数之前，必须在内存中保存给定键的所有值。为什么这有问题？如果您有大量的键偏斜，则某些分区可能会因给定键的大量值而完全过载，并且您将获得`OutOfMemoryErrors`。显然，这不会导致我们当前的数据集出现问题，但可能会导致严重的大规模问题。不能保证会发生这种情况，但是有可能发生。

There are use cases when `groupByKey` does make sense. If you have consistent value sizes for each key and know that they will fit in the memory of a given executor, you’re going to be just fine. It’s just good to know exactly what you’re getting yourself into when you do this. There is a preferred approach for additive use cases: `reduceByKey`.

在某些情况下，`groupByKey`确实有意义。如果每个键的值大小都一致，并且知道它们的大小将适合给定执行器的内存，那么就可以了。最好能确切地知道自己在进行此操作时会遇到什么。对于用例，有一种首选方法：`reduceByKey`。

#### <font color="#3399cc">reduceByKey</font>

Because we are performing a simple count, a much more stable approach is to perform the same `flatMap` and then just perform a map to map each letter instance to the number one, and then perform a `reduceByKey` with a summation function in order to collect back the array. This implementation is much more stable because the reduce happens within each partition and doesn’t need to put everything in memory. Additionally, there is no incurred shuffle during this operation; everything happens at each worker individually before performing a final reduce. This greatly enhances the speed at which you can perform the operation as well as the stability of the operation:

因为我们执行的是简单计数，所以一种更稳定的方法是执行相同的 `flatMap`，然后执行映射以将每个字母实例映射到数字，然后执行带有 sum 函数的 `reduceByKey` 以便回收数组。这种实现更加稳定，因为减少操作发生在每个分区内，不需要将所有内容都放在内存中。此外，在此操作过程中不会发生数据再分配；在执行最终归约（reduce）之前，每件事都会在每个 worker 上发生。这极大地提高了您执行操作的速度以及操作的稳定性：

 ```scala
 KVcharacters.reduceByKey(addFunc).collect()
 ```

 Here’s the result of the operation:

操作结果如下：

```txt
Array((d,4), (p,3), (t,3), (b,1), (h,1), (n,2),
...
(a,4), (i,7), (k,1), (u,1), (o,1), (g,3), (m,2), (c,1))
```

The `reduceByKey` method returns an RDD of a group (the key) and sequence of elements that are not guaranteed to have an ordering. Therefore this method is completely appropriate when our workload is associative but inappropriate when the order matters.

`reduceByKey`方法返回一组（键）的RDD以及不保证具有顺序的元素序列。因此，当我们的工作量具有关联性时，此方法是完全合适的，但是当订单重要时，此方法是不合适的。

### <font color="#00000">Other Aggregation Methods</font>

There exist a number of advanced aggregation methods. For the most part these are largely implementation details depending on your specific workload. We find it very rare that users come across this sort of workload (or need to perform this kind of operation) in modern-day Spark. There just aren’t that many reasons for using these extremely low-level tools when you can perform much simpler aggregations using the Structured APIs. These functions largely allow you very specific, very low-level control on exactly how a given aggregation is performed on the cluster of machines.

存在许多高级聚合方法。在大多数情况下，这些主要是实现细节，具体取决于您的特定工作负载。我们发现用户很少在现代Spark中遇到这种工作负载（或需要执行这种操作）。当您可以使用结构化API执行更简单的汇总时，使用这些极底层工具的原因并不多。这些功能很大程度上允许您非常精确，非常底层地控制如何在计算机群集上执行给定聚合。

#### <font color="#3399cc">aggregate</font>

Another function is aggregate. This function requires a null and start value and then requires you to specify two different functions. The first aggregates within partitions, the second aggregates across partitions. The start value will be used at both aggregation levels:

另一个功能是聚合。该函数需要一个null和起始值，然后需要您指定两个不同的函数。第一个聚集（aggregate）在分区内，第二个聚集（aggregates ）跨分区之间。起始值将在两个聚合级别上使用：  

```scala
// in Scala
nums.aggregate(0)(maxFunc, addFunc)
```

```python
# in Python
nums.aggregate(0, maxFunc, addFunc)
```

aggregate does have some performance implications because it performs the final aggregation on the driver. If the results from the executors are too large, they can take down the driver with an `OutOfMemoryError`. There is another method, `treeAggregate` that does the same thing as aggregate (at the user level) but does so in a different way. It basically “pushes down” some of the subaggregations (creating a tree from executor to executor) before performing the final aggregation on the driver. Having multiple levels can help you to ensure that the driver does not run out of memory in the process of the aggregation. These tree-based implementations are often to try to improve stability in certain operations:

聚合确实会影响性能，因为它在驱动程序上执行最终聚合。如果执行程序的结果太大，则它们可能会因`OutOfMemoryError`而关闭驱动程序。还有另一种方法，`treeAggregate`，它与聚合（在用户级别）执行相同的操作，但是执行方式不同。在对驱动程序执行最终聚合之前，它基本上是“下推”某些子聚合（在 executor 之间创建树）。具有多个级别可以帮助您确保驱动程序在聚合过程中不会耗尽内存。这些基于树的实现通常是为了尝试提高某些操作的稳定性：

```scala
// in Scala
val depth = 3
nums.treeAggregate(0)(maxFunc, addFunc, depth)
```

```python
# in Python
depth = 3
nums.treeAggregate(0, maxFunc, addFunc, depth)
```

#### <font color="#3399cc">aggregateByKey</font>

This function does the same as aggregate but instead of doing it partition by partition, it does it by key. The start value and functions follow the same properties:

此功能与聚合功能相同，但不是一个分区一个分区地进行，而是按键进行。起始值和函数具有相同的属性：

```scala
// in Scala
KVcharacters.aggregateByKey(0)(addFunc, maxFunc).collect()
```

```python
# in Python
KVcharacters.aggregateByKey(0, addFunc, maxFunc).collect()
```

#### <font color="#3399cc">combineByKey</font>

Instead of specifying an aggregation function, you can specify a combiner. This combiner operates on a given key and merges the values according to some function. It then goes to merge the different outputs of the combiners to give us our result. We can specify the number of output partitions as a custom output partitioner as well:

您可以指定组合器，而不是指定聚合函数。该组合器对给定的键进行操作，并根据某些函数合并值。然后将合并器的不同输出进行合并，以得到我们的结果。我们还可以将输出分区的数量指定为自定义输出分区程序：

```scala
// in Scala
val valToCombiner = (value:Int) => List(value)
val mergeValuesFunc = (vals:List[Int], valToAppend:Int) => valToAppend :: vals
val mergeCombinerFunc = (vals1:List[Int], vals2:List[Int]) => vals1 ::: vals2
// now we define these as function variables
val outputPartitions = 6
KVcharacters.combineByKey(
    valToCombiner,
    mergeValuesFunc,
    mergeCombinerFunc,
    outputPartitions
).collect()
```

```python
# in Python
def valToCombiner(value):
	return [value]
def mergeValuesFunc(vals, valToAppend):
	vals.append(valToAppend)
	return vals
def mergeCombinerFunc(vals1, vals2):
	return vals1 + vals2
outputPartitions = 6
KVcharacters.combineByKey(
    valToCombiner,
    mergeValuesFunc,
    mergeCombinerFunc,
	outputPartitions
).collect()
```

#### <font color="#3399cc">foldByKey</font>

`foldByKey` merges the values for each key using an associative function and a neutral “zero value,” which can be added to the result an arbitrary number of times, and must not change the result (e.g., 0 for addition, or 1 for multiplication):

`foldByKey` 使用关联函数和中性的“零值”合并每个键的值，该值可以任意多次添加到结果中，并且不得更改结果（例如，0表示加法，1表示乘法）：

```scala
// in Scala
KVcharacters.foldByKey(0)(addFunc).collect()
```

```python
# in Python
KVcharacters.foldByKey(0, addFunc).collect()
```

## <font color="#9a161a">CoGroups</font>

CoGroups give you the ability to group together up to three key–value RDDs together in Scala and two in Python. This joins the given values by key. This is effectively just a group-based join on an RDD. When doing this, you can also specify a number of output partitions or a custom partitioning function to control exactly how this data is distributed across the cluster (we talk about partitioning functions later on in this chapter):

借助 CoGroup，您可以在Scala中将最多三个键值RDD组合在一起，而在Python中将它们组合在一起。这通过键将给定值连接在一起。实际上，这只是RDD上基于组的连接。在执行此操作时，您还可以指定多个输出分区或自定义分区函数，以精确控制该数据在整个集群中的分布方式（我们将在本章稍后讨论分区功能）：

```scala
// in Scala
import scala.util.Random
val distinctChars = words.flatMap(word => word.toLowerCase.toSeq).distinct
val charRDD = distinctChars.map(c => (c, new Random().nextDouble()))
val charRDD2 = distinctChars.map(c => (c, new Random().nextDouble()))
val charRDD3 = distinctChars.map(c => (c, new Random().nextDouble()))
charRDD.cogroup(charRDD2, charRDD3).take(5)
```

```python
# in Python
import random
distinctChars = words.flatMap(lambda word: word.lower()).distinct()
charRDD = distinctChars.map(lambda c: (c, random.random()))
charRDD2 = distinctChars.map(lambda c: (c, random.random()))
charRDD.cogroup(charRDD2).take(5)
```

The result is a group with our key on one side, and all of the relevant values on the other side.

结果是一组，我们的 key 在一侧，而所有相关值在另一侧。

```txt
Array[(Char, (Iterable[Double], Iterable[Double], Iterable[Double]))] = Array((d,(CompactBuffer(0.12833684521321143),CompactBuffer(0.5229399319461184),CompactBuffer(0.39412761081641534))), (p,(CompactBuffer(0.5563787512207469),CompactBuffer(0.8482281764275303),CompactBuffer(0.05654936603265115))), (t,(CompactBuffer(0.8063968912600572),CompactBuffer(0.8059552537188721),CompactBuffer(0.4538221779361298))), (b,(CompactBuffer(0.19635385859609022),CompactBuffer(0.15376521889330752),CompactBuffer(0.07330965327320438))), (h,(CompactBuffer(0.1639926173875862),CompactBuffer(0.139685392942837),CompactBuffer(0.10124445377925972))))
```

## <font color="#9a161a">Joins</font>

RDDs have much the same joins as we saw in the Structured API, although RDDs are much more involved for you. They all follow the same basic format: the two RDDs we would like to join, and, optionally, either the number of output partitions or the customer partition function to which they should output. We’ll talk about partitioning functions later on in this chapter.

尽管RDD对您来说涉及更多，但RDD的连接与结构化API中的连接几乎相同。它们都遵循相同的基本格式：我们要加入的两个RDD，以及可选的输出分区的数量或它们应输出到的自定义分区函数。我们将在本章稍后讨论分区函数。

### <font color="#00000">Inner Join</font>

We’ll demonstrate an inner join now. Notice how we are setting the number of output partitions we would like to see:

现在，我们将演示内部连接。注意我们如何设置我们希望看到的输出分区数： 

```scala
// in Scala
val keyedChars = distinctChars.map(c => (c, new Random().nextDouble()))
val outputPartitions = 10
KVcharacters.join(keyedChars).count()
KVcharacters.join(keyedChars, outputPartitions).count()
```

```python
# in Python
keyedChars = distinctChars.map(lambda c: (c, random.random()))
outputPartitions = 10
KVcharacters.join(keyedChars).count()
KVcharacters.join(keyedChars, outputPartitions).count()
```

We won’t provide an example for the other joins, but they all follow the same basic format. You can learn about the following join types at the conceptual level in Chapter 8:

我们不会提供其他连接的示例，但是它们都遵循相同的基本格式。您可以在第8章的概念层次上了解以下连接类型：

- fullOuterJoin
- leftOuterJoin
- rightOuterJoin
- cartesian (This, again, is very dangerous! It does not accept a join key and can have a massive output.) 笛卡尔（再次，这非常危险！它不接受连接键，并且可以产生大量输出。）

### <font color="#00000">zips</font>

The final type of join isn’t really a join at all, but it does combine two RDDs, so it’s worth labeling it as a join. zip allows you to “zip” together two RDDs, assuming that they have the same length. This creates a PairRDD. The two RDDs must have the same number of partitions as well as the same number of elements:

最终的连接类型根本不是连接，但它确实结合了两个RDD，因此值得将其标记为连接。zip允许您将两个RDD“压缩”在一起，前提是它们具有相同的长度。这将创建一个PairRDD。两个RDD必须具有相同数量的分区和相同数量的元素：

```scala
// in Scala
val numRange = sc.parallelize(0 to 9, 2)
words.zip(numRange).collect()
```

```python
# in Python
numRange = sc.parallelize(range(10), 2)
words.zip(numRange).collect()
```

This gives us the following result, an array of keys zipped to the values:

这为我们提供了以下结果，将键数组压缩为值：

```txt
[('Spark', 0),
('The', 1),
('Definitive', 2),
('Guide', 3),
(':', 4),
('Big', 5),
('Data', 6),
('Processing', 7),
('Made', 8),
('Simple', 9)]
```

## <font color="#9a161a">Controlling Partitions</font>

With RDDs, you have control over how data is exactly physically distributed across the cluster. Some of these methods are basically the same from what we have in the Structured APIs but the key addition (that does not exist in the Structured APIs) is the ability to specify a partitioning function (formally a custom Partitioner, which we discuss later when we look at basic methods).

使用RDD，您可以控制如何在整个群集中准确地物理分布数据。其中一些方法与结构化API中的方法基本相同，但关键的附加功能（结构化API中不存在）具有指定分区函数（通常是自定义分区程序）的能力，稍后我们将在后面讨论看看基本方法）。

### <font color="#00000">coalesce</font>

`coalesce` effectively collapses partitions on the same worker in order to avoid a shuffle of the data when repartitioning. For instance, our words RDD is currently two partitions, we can collapse that to one partition by using `coalesce` without bringing about a shuffle of the data:

`coalesce` 有效地折叠了同一 worker 上的分区，以避免在重新分区时数据数据再分配。例如，我们的词RDD当前是两个分区，我们可以使用 `coalesce` 将其折叠为一个分区，而不会造成数据再分配：

```scala
// in Scala
words.coalesce(1).getNumPartitions // 1
```
```python
# in Python
words.coalesce(1).getNumPartitions() # 1
```

### <font color="#00000">repartition</font>

The repartition operation allows you to repartition your data up or down but performs a shuffle across nodes in the process. Increasing the number of partitions can increase the level of parallelism when operating in map- and filter-type operations:

使用重新分区操作，您可以向上或向下重新分区数据，但可以在进程中的各个节点之间执行数据再分配。在map和filter类型的操作中进行操作时，增加分区数量可以提高并行度：

```scala
words.repartition(10) // gives us 10 partitions
```

### <font color="#00000">repartitionAndSortWithinPartitions</font>

This operation gives you the ability to repartition as well as specify the ordering of each one of those output partitions. We’ll omit the example because the documentation for it is good, but both the partitioning and the key comparisons can be specified by the user.

此操作使您能够重新分区以及指定这些输出分区中每个分区的顺序。我们将省略该示例，因为该示例的文档不错，但是分区和键比较都可以由用户指定。

### <font color="#00000">Custom Partitioning</font>

This ability is one of the primary reasons you’d want to use RDDs. Custom partitioners are not available in the Structured APIs because they don’t really have a logical counterpart. They’re a lowlevel, implementation detail that can have a significant effect on whether your jobs run successfully. The canonical example to motivate custom partition for this operation is PageRank whereby we seek to control the layout of the data on the cluster and avoid shuffles. In our shopping dataset, this might mean partitioning by each customer ID (we’ll get to this example in a moment).

此功能是您要使用RDD的主要原因之一。自定义分区程序在结构化API中不可用，因为它们实际上并没有逻辑上的对应关系。它们是底层的实施细节，可能会对您的作业是否成功运行产生重大影响。鼓励对此操作进行自定义分区的典型示例是PageRank，据此，我们试图控制集群上数据的布局并避免乱序。在我们的购物数据集中，这可能意味着按每个客户ID进行分区（我们稍后将转到此示例）。

In short, the sole goal of custom partitioning is to even out the distribution of your data across the cluster so that you can work around problems like data skew.

简而言之，自定义分区的唯一目标是使数据在集群中的分布均匀，以便您可以解决数据倾斜等问题。  

If you’re going to use custom partitioners, you should drop down to RDDs from the Structured APIs, apply your custom partitioner, and then convert it back to a DataFrame or Dataset. This way, you get the best of both worlds, only dropping down to custom partitioning when you need to.

如果要使用自定义分区程序，则应从结构化API降到RDD，应用自定义分区程序，然后将其转换回DataFrame或Dataset。这样，您可以兼得两全其美，只在需要时才使用自定义分区。

To perform custom partitioning you need to implement your own class that extends Partitioner. You need to do this only when you have lots of domain knowledge about your problem space—if you’re just looking to partition on a value or even a set of values (columns), it’s worth just doing it in the DataFrame API.

要执行自定义分区，您需要实现自己的扩展Partitioner的类。只有在对问题空间有很多领域知识的情况下，才需要执行此操作。如果您只是想对一个值甚至一组值（列）进行分区，那么只需在DataFrame API中进行操作即可。

Let’s dive into an example:

让我们投入到一个例子：

```scala
// in Scala
val df = spark.read.option("header", "true").option("inferSchema", "true")
.csv("/spark/The-Definitive-Guide/data/retail-data/all/")
val rdd = df.coalesce(10).rdd
```

```python
# in Python
df = spark.read.option("header", "true").option("inferSchema", "true")\
.csv("/data/retail-data/all/")
rdd = df.coalesce(10).rdd
```

```scala
df.printSchema()
```

Spark has two built-in Partitioners that you can leverage off in the RDD API, a `HashPartitioner` for discrete values and a `RangePartitioner`. These two work for discrete values and continuous values, respectively. Spark’s Structured APIs will already use these, although we can use the same thing in RDDs:

Spark具有两个可在RDD API中使用的内置分区程序，一个用于离散值的 `HashPartitioner` 和一个 `RangePartitioner`。这两个分别适用于离散值和连续值。尽管我们可以在RDD中使用相同的东西，但Spark的结构化API已经使用了它们：

```scala
// in Scala
import org.apache.spark.HashPartitioner
rdd.map(r => r(6)).take(5).foreach(println)
val keyedRDD = rdd.keyBy(row => row(6).asInstanceOf[Int].toDouble)
keyedRDD.partitionBy(new HashPartitioner(10)).take(10)
```

Although the hash and range partitioners are useful, they’re fairly rudimentary. At times, you will need to perform some very low-level partitioning because you’re working with very large data and large key skew. Key skew simply means that some keys have many, many more values than other keys. You want to break these keys as much as possible to improve parallelism and prevent `OutOfMemoryErrors` during the course of execution.

尽管哈希和范围分区器很有用，但还很初级。有时，由于要处理非常大的数据和较大的键倾斜，因此您将需要执行一些非常底层的分区。键倾斜只是意味着某些键比其他键具有更多很多的值。您希望尽可能地拆分这些键，以提高并行度并在执行过程中防止 `OutOfMemoryErrors`。

One instance might be that you need to partition more keys if and only if the key matches a certain format. For instance, we might know that there are two customers in your dataset that always crash your analysis and we need to break them up further than other customer IDs. In fact, these two are so skewed that they need to be operated on alone, whereas all of the others can be lumped into large groups. This is obviously a bit of a caricatured example, but you might see similar situations in your data, as well:

一个实例可能是，当且仅当键与某种格式匹配时，才需要对更多键进行分区。例如，我们可能知道您的数据集中有两个客户总是使您的分析崩溃，因此我们需要比其他客户ID进一步细分它们。实际上，这两个倾斜了，需要单独操作，而其他所有都可以分成大组。这显然是一个讽刺的例子，但是您可能还会在数据中看到类似的情况：

```scala
// in Scala
import org.apache.spark.Partitioner
class DomainPartitioner extends Partitioner {
    def numPartitions = 3
    def getPartition(key: Any): Int = {
    	val customerId = key.asInstanceOf[Double].toInt
        if (customerId == 17850.0 || customerId == 12583.0) {
            return 0
        } else {
            return new java.util.Random().nextInt(2) + 1
        }
	}
} 

keyedRDD
.partitionBy(new DomainPartitioner).map(_._1).glom().map(_.toSet.toSeq.length)
.take(5)
```


After you run this, you will see the count of results in each partition. The second two numbers will vary, because we’re distributing them randomly (as you will see when we do the same in Python) but the same principles apply:

运行此命令后，您将看到每个分区中的结果计数。后两个数字会有所不同，因为我们是随机分配它们（如您在Python中进行相同操作时所见），但是适用相同的原理：

```scala
# in Python
def partitionFunc(key):
    import random
    if key == 17850 or key == 12583:
        return 0
    else:
        return random.randint(1,2)
```

```scala
keyedRDD = rdd.keyBy(lambda row: row[6])
keyedRDD\
.partitionBy(3, partitionFunc)\
.map(lambda x: x[0])\
.glom()\
.map(lambda x: len(set(x)))\
.take(5)
```

This custom key distribution logic is available only at the RDD level. Of course, this is a simple example, but it does show the power of using arbitrary logic to distribute the data around the cluster in a physical manner.

此自定义 key 分发逻辑仅在RDD级别可用。当然，这是一个简单的示例，但是它确实显示了使用任意逻辑以物理方式在集群中分布数据的强大能力。

## <font color="#9a161a">Custom Serialization</font>

The last advanced topic that is worth talking about is the issue of Kryo serialization. Any object that you hope to parallelize (or function) must be serializable: 

值得讨论的最后一个高级主题是Kryo序列化问题。您希望并行化（或函数化）的任何对象都必须可序列化：

```scala
// in Scala
class SomeClass extends Serializable {
var someValue = 0
	def setSomeValue(i:Int) = {
		someValue = i
		this
	}
} 
sc.parallelize(1 to 10).map(num => new SomeClass().setSomeValue(num))
```

The default serialization can be quite slow. Spark can use the Kryo library (version 2) to serialize objects more quickly. Kryo is significantly faster and more compact than Java serialization (often as much as 10x), but does not support all serializable types and requires you to register the classes you’ll use in the program in advance for best performance.

默认序列化可能会很慢。Spark可以使用Kryo库（版本2）更快地序列化对象。与Java序列化（通常多达10倍）相比，Kryo显着更快，更紧凑，但是它不支持所有可序列化的类型，并且需要您预先注册要在程序中使用的类才能获得最佳性能。

You can use Kryo by initializing your job with a `SparkConf` and setting the value of "spark.serializer" to "org.apache.spark.serializer.KryoSerializer" (we discuss this in the next part of the book). This setting configures the serializer used for shuffling data between worker nodes and serializing RDDs to disk. The only reason Kryo is not the default is because of the custom registration requirement, but we recommend trying it in any . Since Spark 2.0.0, we internally use Kryo serializer when shuffling RDDs with simple types, arrays of simple types, or string type. 

您可以通过使用`SparkConf`初始化作业并将 “`spark.serializer`” 的值设置为 “`org.apache.spark.serializer.KryoSerializer`”来使用 Kryo（我们将在本书的下一部分中讨论）。此设置配置序列化器用于在 worker 节点之间对数据进行再分配以及序列化RDD到磁盘。Kryo 不是默认值的唯一原因是由于自定义注册要求，但是我们建议在任何网络密集型应用程序中尝试使用它。从Spark 2.0.0开始，在对具有简单类型，简单类型数组或字符串类型的RDD进行改组时，我们在内部使用Kryo序列化程序。

Spark automatically includes Kryo serializers for the many commonly used core Scala classes covered in the AllScalaRegistrar from the Twitter chill library.

对于许多来自twitter chill库中的属于AllScalaRegistrar的常用核心Scala类，Spark自动为它们包括Kryo序列化器。

To register your own custom classes with Kryo, use the registerKryoClasses method:

要向 Kryo 注册您自己的自定义类，请使用registerKryoClasses方法：

```scala
// in Scala
val conf = new SparkConf().setMaster(...).setAppName(...)
conf.registerKryoClasses(Array(classOf[MyClass1], classOf[MyClass2]))
val sc = new SparkContext(conf)
```

## <font color="#9a161a">Conclusion 小结</font>

In this chapter we discussed many of the more advanced topics regarding RDDs. Of particular note was the section on custom partitioning, which allows you very specific functions to layout your data.In Chapter 14, we discuss another of Spark’s low-level tools: distributed variables.

在本章中，我们讨论了有关RDD的许多更高级的主题。特别值得注意的是有关自定义分区的部分，该节允许您使用非常特定的功能来布局数据。在第14章中，我们讨论了Spark的另一个底层工具：分布式变量。 