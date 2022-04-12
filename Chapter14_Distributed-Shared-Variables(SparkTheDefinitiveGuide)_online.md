---
title: 翻译 Chapter 14 Distributed Shared Variables
date: 2019-11-07
copyright: true
categories: English
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 14 Distributed Shared Variables

In addition to the Resilient Distributed Dataset (RDD) interface, the second kind of low-level API in Spark is two types of “distributed shared variables”: broadcast variables and accumulators. These are variables you can use in your user-defined functions (e.g., in a map function on an RDD or a DataFrame) that have special properties when running on a cluster. Specifically, accumulators let you add together data from all the tasks into a shared result (e.g., to implement a counter so you can see how many of your job’s input records failed to parse), while broadcast variables let you save a large value on all the worker nodes and reuse it across many Spark actions without re-sending it to the cluster. This chapter discusses some of the motivation for each of these variable types as well as how to use them.

除了弹性分布式数据集（RDD）接口外，Spark中的第二种底层API是两种“分布式共享变量”：广播变量（broadcast variable）和累加器（accumulator）。这些是您可以在用户定义的函数（例如，在RDD或DataFrame上的映射函数）中使用的变量，这些变量在集群上运行时具有特殊的属性。具体来说，累加器使您可以将所有任务的数据加到一个共享的结果中（例如，实现一个计数器，以便您可以查看有多少作业的输入记录无法解析），而广播变量使您可以在所有工作节点上保存较大的值，并在许多Spark action中重复使用它，而无需将其重新发送到集群。本章讨论了每种变量类型的一些动机以及如何使用它们。

## <font color="#9a161a">Broadcast Variables</font>

Broadcast variables are a way you can share an immutable value efficiently around the cluster without encapsulating that variable in a function closure. The normal way to use a variable in your driver node inside your tasks is to simply reference it in your function closures (e.g., in a map operation), but this can be inefficient, especially for large variables such as a lookup table or a machine learning model. The reason for this is that when you use a variable in a closure, it must be deserialized on the worker nodes many times (one per task). Moreover, if you use the same variable in multiple Spark actions and jobs, it will be re-sent to the workers with every job instead of once.

广播变量是一种无需在函数闭包中封装该变量就可以有效地在集群中共享不可变值的方法。在任务的驱动程序节点中使用变量的通常方法是在函数闭包中（例如在映射操作中）简单地引用它，但这可能效率不高，尤其是对于较大的变量，例如查找表或机器学习模型。这样做的原因是，当在闭包中使用变量时，必须在 worker 上多次对它进行反序列化（每个任务一个）。而且，如果您在多个Spark action和作业中使用相同的变量，则它将随每个作业重新发送给 worker，而不是一次。

This is where broadcast variables come in. Broadcast variables are shared, immutable variables that are cached on every machine in the cluster instead of serialized with every single task. The canonical use case is to pass around a large lookup table that fits in memory on the executors and use that in a function, as illustrated in Figure 14-1.

这就是广播变量的用处。广播变量是共享的，不可变的变量，它们缓存在集群中的每台计算机上，而不是与每个任务序列化。规范的用例是传递一个大的查找表，该表的大小适合 executor 的内存，并在函数中使用它，如图14-1所示。

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter14/1574393608504.png" alt="1" style="zoom:80%;" />

For example, suppose that you have a list of words or values:

例如，假设您有一个单词或值的列表：

```scala
// in Scala
val myCollection = "Spark The Definitive Guide : Big Data Processing Made Simple"
.split(" ")
val words = spark.sparkContext.parallelize(myCollection, 2)
```

```python
# in Python
my_collection = "Spark The Definitive Guide : Big Data Processing Made Simple"\
.split(" ")
words = spark.sparkContext.parallelize(my_collection, 2)
```

You would like to supplement your list of words with other information that you have, which is many kilobytes, megabytes, or potentially even gigabytes in size. This is technically a right join if we thought about it in terms of SQL:

您想用其他信息来补充单词列表，这些信息的大小可能为千字节，兆字节甚至是千兆字节。如果从SQL角度考虑，从技术上讲这是一个正确的连接：

```scala
// in Scala
val supplementalData = Map("Spark" -> 1000, "Definitive" -> 200,
"Big" -> -300, "Simple" -> 100)
```

```python
# in Python
supplementalData = {"Spark":1000, "Definitive":200,
"Big":-300, "Simple":100}
```

We can broadcast this structure across Spark and reference it by using `suppBroadcast`. This value is immutable and is lazily replicated across all nodes in the cluster when we trigger an action:

我们可以在Spark上广播此结构，并使用`suppBroadcast`引用它。当我们触发一个 action 时，该值是不可变的，并且会在集群中的所有节点之间惰性复制：

```scala
// in Scala
val suppBroadcast = spark.sparkContext.broadcast(supplementalData)
```

```python
# in Python
suppBroadcast = spark.sparkContext.broadcast(supplementalData)
```

We reference this variable via the value method, which returns the exact value that we had earlier. This method is accessible within serialized functions without having to serialize the data. This can save you a great deal of serialization and deserialization costs because Spark transfers data more efficiently around the cluster using broadcasts:

我们通过value方法引用此变量，该方法返回我们之前的确切值。 此方法可在序列化函数内访问，而不必序列化数据。这可以为您节省大量的序列化和反序列化成本，因为Spark使用广播在集群中更高效地传输数据：

```scala
// in Scala
suppBroadcast.value
```

```python
# in Python
suppBroadcast.value
```

Now we could transform our RDD using this value. In this instance, we will create a key–value pair according to the value we might have in the map. If we lack the value, we will simply replace it with 0:

现在，我们可以使用此值转换RDD。在这种情况下，我们将根据映射中可能具有的值创建一个键值对。如果缺少该值，则只需将其替换为0：

```scala
// in Scala
words.map(word => (word, suppBroadcast.value.getOrElse(word, 0)))
.sortBy(wordPair => wordPair._2)
.collect()
```

```python
# in Python
words.map(lambda word: (word, suppBroadcast.value.get(word, 0)))\
.sortBy(lambda wordPair: wordPair[1])\
.collect()
```

This returns the following value in Python and the same values in an array type in Scala:

这将在Python中返回以下值，在Scala中返回数组类型中的相同值：

```txt
[('Big', -300),
('The', 0),
...
('Definitive', 200),
('Spark', 1000)]
```

The only difference between this and passing it into the closure is that we have done this in a much more efficient manner (Naturally, this depends on the amount of data and the number of executors. For very small data (low KBs) on small clusters, it might not be). Although this small dictionary probably is not too large of a cost, if you have a much larger value, the cost of serializing the data for every task can be quite significant.

 此操作与将其传递给闭包之间的唯一区别是，我们以一种更加高效的方式完成了此操作（自然，这取决于数据量和 executor 的数量。对于小型集群中的非常小的数据（低KB）而言），可能不是）。尽管这个小词典的开销可能不会太大，但是如果您拥有更大的价值，则为每个任务序列化数据的开销可能会非常大。

One thing to note is that we used this in the context of an RDD; we can also use this in a UDF or in a Dataset and achieve the same result.

 需要注意的一件事是，我们在RDD的上下文中使用了它。我们也可以在 UDF 或 Dataset 中使用它来达到相同的结果。

## <font color="#9a161a">Accumulators</font>

Accumulators (Figure 14-2), Spark’s second type of shared variable, are a way of updating a value inside of a variety of transformations and propagating that value to the driver node in an efficient and fault-tolerant way.

 累加器（图14-2）是Spark的第二种共享变量，是一种在各种转换中更新值并将该值以有效且容错的方式传播到驱动程序节点的方法。

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter14/1574401245639.png" alt="1" style="zoom:80%;" />

 

 Accumulators provide a mutable variable that a Spark cluster can safely update on a per-row basis. You can use these for debugging purposes (say to track the values of a certain variable per partition in order to intelligently use it over time) or to create low-level aggregation. Accumulators are variables that are “added” to only through an associative and commutative operation and can therefore be efficiently supported in parallel. You can use them to implement counters (as in MapReduce) or sums. Spark natively supports accumulators of numeric types, and programmers can add support for new types.

 累加器提供一个可变变量，Spark集群可以在每行的基础上安全地对其进行更新。您可以将它们用于调试目的（例如跟踪每个分区中某个变量的值，以便随着时间的推移智能地使用它）或创建底层聚合。累加器是仅通过关联和交换操作“添加”的变量，因此可以并行有效地得到支持。您可以使用它们来实现计数器（如MapReduce）或总和。Spark原生支持数字类型的累加器，程序员可以添加对新类型的支持。

For accumulator updates performed inside actions only, Spark guarantees that each task’s update to the accumulator will be applied only once, meaning that restarted tasks will not update the value. In transformations, you should be aware that each task’s update can be applied more than once if tasks or job stages are reexecuted.

 对于仅在 action 内部执行的累加器更新，Spark保证每个任务对累加器的更新将仅应用一次，这意味着重新启动的任务将不会更新该值。在转换中，您应该意识到，如果重新执行任务或作业阶段，则可以多次应用每个任务的更新。

Accumulators do not change the lazy evaluation model of Spark. If an accumulator is being updated within an operation on an RDD, its value is updated only once that RDD is actually computed (e.g., when you call an action on that RDD or an RDD that depends on it). Consequently, accumulator updates are not guaranteed to be executed when made within a lazy transformation like `map()`.

 累加器不会更改Spark的惰性求值模型（lazy evaluation model）。如果在RDD上的操作中正在更新累加器，则仅在实际计算RDD之后才更新其值（例如，当您对该RDD或依赖于该RDD的RDD调用操作时）。因此，当在诸如 `map()` 的惰性转换中进行累加器更新时，不能保证执行更新。

Accumulators can be both named and unnamed. Named accumulators will display their running results in the Spark UI, whereas unnamed ones will not.

 累加器可以命名也可以不命名。命名累加器将在Spark UI中显示其运行结果，而未命名累加器则不会。

### <font color="#00000">Basic Example</font>

Let’s experiment by performing a custom aggregation on the Flight dataset that we created earlier in the book. In this example, we will use the Dataset API as opposed to the RDD API, but the extension is quite similar:

让我们通过对我们在本书前面创建的 Flight 数据集执行自定义汇总来进行实验。在此示例中，我们将使用Dataset API而不是RDD API，但扩展名非常相似： 

```scala
// in Scala
case class Flight(DEST_COUNTRY_NAME: String,
ORIGIN_COUNTRY_NAME: String, count: BigInt)
val flights = spark.read
.parquet("/data/flight-data/parquet/2010-summary.parquet")
.as[Flight]
```

```python
# in Python
flights = spark.read\
.parquet("/data/flight-data/parquet/2010-summary.parquet")
```

Now let’s create an accumulator that will count the number of flights to or from China. Even though we could do this in a fairly straightfoward manner in SQL, many things might not be so straightfoward. Accumulators provide a programmatic way of allowing for us to do these sorts of counts. The following demonstrates creating an unnamed accumulator:

现在，我们创建一个累加器，该累加器将计算往返中国的航班数量。即使我们可以在SQL中以相当直截了当的方式执行此操作，但许多事情可能并不那么直截了当。 累加器提供了一种编程方式，使我们能够进行此类计数。下面演示了如何创建未命名的累加器：

```scala
// in Scala
import org.apache.spark.util.LongAccumulator
val accUnnamed = new LongAccumulator
val acc = spark.sparkContext.register(accUnnamed)
```

```PYTHON
# in Python
accChina = spark.sparkContext.accumulator(0)
```

Our use case fits a named accumulator a bit better. There are two ways to do this: a short-hand method and a long-hand one. The simplest is to use the SparkContext. Alternatively, we can instantiate the accumulator and register it with a name:

我们的用例更适合命名的累加器。 有两种方法可以做到这一点：一种简便方法和一种常规方法。最简单的是使用SparkContext。另外，我们可以实例化累加器并使用名称注册它：

```SCALA
// in Scala
val accChina = new LongAccumulator
val accChina2 = spark.sparkContext.longAccumulator("China")
spark.sparkContext.register(accChina, "China")		
```

We specify the name of the accumulator in the string value that we pass into the function, or as the second parameter into the register function. Named accumulators will display in the Spark UI, whereas unnamed ones will not.

 我们在传递给函数的字符串值中指定累加器的名称，或者将其指定为寄存器函数的第二个参数。已命名的累加器将显示在Spark UI中，而未命名的累加器则不会显示。

The next step is to define the way we add to our accumulator. This is a fairly straightforward function:

下一步是定义添加到累加器中的方式。这是一个相当简单的功能： 

```scala
// in Scala
def accChinaFunc(flight_row: Flight) = {
    val destination = flight_row.DEST_COUNTRY_NAME
    val origin = flight_row.ORIGIN_COUNTRY_NAME
    if (destination == "China") {
            accChina.add(flight_row.count.toLong)
    } 
    if (origin == "China") {
            accChina.add(flight_row.count.toLong)
    }
}
```

```python
# in Python
def accChinaFunc(flight_row):
    destination = flight_row["DEST_COUNTRY_NAME"]
    origin = flight_row["ORIGIN_COUNTRY_NAME"]
    if destination == "China":
        accChina.add(flight_row["count"])
    if origin == "China":
		accChina.add(flight_row["count"])
```

Now, let’s iterate over every row in our flights dataset via the `foreach` method. The reason for this is because `foreach` is an action, and Spark can provide guarantees that perform only inside of actions.

 现在，让我们通过`foreach`方法遍历 flight 数据集中的每一行。这样做的原因是因为`foreach`是一个动作，Spark可以提供仅在 action 内部执行的保证。

The `foreach` method will run once for each row in the input DataFrame (assuming that we did not filter it) and will run our function against each row, incrementing the accumulator accordingly:

`foreach`方法将对输入DataFrame中的每一行运行一次（假设我们没有对其进行过滤），并将针对每一行运行我们的函数，从而相应地增加累加器：

```scala
// in Scala
flights.foreach(flight_row => accChinaFunc(flight_row))
```

```python
# in Python
flights.foreach(lambda flight_row: accChinaFunc(flight_row))
```

This will complete fairly quickly, but if you navigate to the Spark UI, you can see the relevant value, on a per-Executor level, even before querying it programmatically, as demonstrated in Figure 14-3.

这将很快完成，但是 如果导航到Spark UI，则即使在以编程方式查询它之前，也可以在每个执行器级别上看到相关值，如图14-3所示。

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter14/1574402832000.png" alt="1122" style="zoom:80%;" />

Of course, we can query it programmatically, as well. To do this, we use the value property:

当然，我们也可以通过编程方式查询它。为此，我们使用value属性：

```scala
// in Scala
accChina.value // 953
```

```python
# in Python
accChina.value # 953
```

### <font color="#00000">Custom Accumulators</font>

Although Spark does provide some default accumulator types, sometimes you might want to build your own custom accumulator. In order to do this you need to subclass the AccumulatorV2 class. There are several abstract methods that you need to implement, as you can see in the example that follows. In this example, you we will add only values that are even to the accumulator. Although this is again simplistic, it should show you how easy it is to build up your own accumulators:

尽管Spark确实提供了一些默认的累加器类型，但有时您可能想要构建自己的自定义累加器。为此，您需要对AccumulatorV2类进行子类化。您需要实现多种抽象方法，如以下示例所示。在此示例中，您将甚至仅将值加到累加器。尽管这再简单不过了，但它应该告诉您建立自己的累加器有多么容易： 

```scala
// in Scala
import scala.collection.mutable.ArrayBuffer
import org.apache.spark.util.AccumulatorV2
val arr = ArrayBuffer[BigInt]()
class EvenAccumulator extends AccumulatorV2[BigInt, BigInt] {
	private var num:BigInt = 0
    
	def reset(): Unit = {
		this.num = 0
	} 
    
    def add(intValue: BigInt): Unit = {
        if (intValue % 2 == 0) {
            this.num += intValue
        }
    } 

    def merge(other: AccumulatorV2[BigInt,BigInt]): Unit = {
        this.num += other.value
    } 

    def value():BigInt = {
        this.num
    } 

    def copy(): AccumulatorV2[BigInt,BigInt] = {
        new EvenAccumulator
    } 

    def isZero():Boolean = {
        this.num == 0
    }

} 
val acc = new EvenAccumulator
val newAcc = sc.register(acc, "evenAcc")
```

```scala
// in Scala
acc.value // 0
flights.foreach(flight_row => acc.add(flight_row.count))
acc.value // 31390
```

If you are predominantly a Python user, you can also create your own custom accumulators by subclassing <u><a style="color:#0879e3" href="https://spark.apache.org/docs/1.1.0/api/python/pyspark.accumulators.AccumulatorParam-class.html">AccumulatorParam</a></u> and using it as we saw in the previous example.

如果您主要是Python用户，则也可以通过将 <u><a style="color:#0879e3" href="https://spark.apache.org/docs/1.1.0/api/python/pyspark.accumulators.AccumulatorParam-class.html">AccumulatorParam</a></u> 子类化并使用它来创建自己的自定义累加器，如上例所示。

## <font color="#9a161a">Conclusion</font>

In this chapter, we covered distributed variables. These can be helpful tools for optimizations or for debugging. In Chapter 15, we define how Spark runs on a cluster to better understand when these can be helpful.

在本章中，我们介绍了分布式变量。这些对于优化或调试可能是有用的工具。在第15章中，我们定义了Spark如何在集群上运行，以更好地了解何时可以提供帮助。
