---
title: 翻译 Chapter 11 Datasets
date: 2019-11-07
copyright: true
categories: English
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 11 Datasets
<center><strong>译者</strong>：<u><a style="color:#0879e3" href="https://snaildove.github.io">https://snaildove.github.io</a></u></center>

Datasets are the foundational type of the Structured APIs. We already worked with DataFrames, which are Datasets of type Row, and are available across Spark’s different languages. Datasets are a strictly Java Virtual Machine (JVM) language feature that work only with Scala and Java. Using Datasets, you can define the object that each row in your Dataset will consist of. In Scala, this will be a case class object that essentially defines a schema that you can use, and in Java, you will define a Java Bean. Experienced users often refer to Datasets as the “typed set of APIs” in Spark. For more information, see Chapter 4.

Dataset 是结构化 API 的基本类型。我们已经使用了 DataFrames，它们是 Row 类型的 Dataset，可在Spark的不同语言中使用。Dataset 是严格的 Java 虚拟机（JVM）语言特性（feature），只能使用 Scala 和 Java。使用 Dataset，您可以定义 Dataset 中每一行将组成的对象。在 Scala 中，这将是一个 case 类对象，该对象本质上定义了可以使用的模式，而在Java中，您将定义 Java Bean。有经验的用户通常将 Dataset 称为Spark中的 “API的类型集”。有关更多信息，请参见第4章。

In Chapter 4, we discussed that Spark has types like `StringType`, `BigIntType`, `StructType`, and so on. Those Spark-specific types map to types available in each of Spark’s languages like String, Integer, and Double. When you use the DataFrame API, you do not create strings or integers, but Spark manipulates the data for you by manipulating the Row object. In fact, if you use Scala or Java, all “DataFrames” are actually Datasets of type Row. To efficiently support domain-specific objects, a special concept called an “Encoder” is required. The encoder maps the domain-specific type T to Spark’s internal type system.

在第4章中，我们讨论了Spark具有`StringType`，`BigIntType`，`StructType`等类型。这些特定于Spark的类型映射到每种Spark语言中可用的类型，例如String，Integer和Double。使用DataFrame API时，您不会创建字符串或整数，但是Spark通过操纵Row对象为您操纵数据。实际上，如果使用Scala或Java，则所有 “DataFrame” 实际上都是Row类型的Dataset。为了有效地支持特定于域的对象，需要一个称为“编码器”的特殊概念。编码器将特定于域的类型T映射到Spark的内部类型系统。

For example, given a class Person with two fields, name (string) and age (int), an encoder directs Spark to generate code at runtime to serialize the Person object into a binary structure. When using DataFrames or the “standard” Structured APIs, this binary structure will be a Row. When we want to create our own domain-specific objects, we specify a case class in Scala or a JavaBean in Java. Spark will allow us to manipulate this object (in place of a Row) in a distributed manner.

例如，给定Person类具有 name (string) 和 age (int) 两个字段，编码器指示Spark在运行时生成代码以将Person对象序列化为二进制结构。当使用 DataFrame 或“标准”结构化API时，此二进制结构将为行。当我们要创建自己的特定于域的对象时，我们在Scala中指定一个案例类，在Java中指定一个JavaBean。Spark将允许我们以分布式方式操纵该对象（代替Row）。

When you use the Dataset API, for every row it touches, this domain specifies type, Spark converts the Spark Row format to the object you specified (a case class or Java class). This conversion slows down your operations but can provide more flexibility. You will notice a hit in performance but this is a far different order of magnitude from what you might see from something like a user-defined function (UDF) in Python, because the performance costs are not as extreme as switching programming languages, but it is an important thing to keep in mind.

使用 Dataset  API时，该域为它遇到的每一行指定类型，Spark将Spark Row格式转换为您指定的对象（案例类或Java类）。这种转换会减慢您的操作速度，但可以提供更大的灵活性。您会注意到性能受到了影响，但这与您在Python中的用户定义函数（UDF）之类的看到的结果数量级相差很大，因为性能成本并不像切换编程语言那样极端，但是是一件重要的事情要牢记。

## <font color="#9a161a">When to Use Datasets</font>

You might ponder, if I am going to pay a performance penalty when I use Datasets, why should I use them at all? If we had to condense this down into a canonical list, here are a couple of reasons:

您可能会思考，如果我在使用 Dataset 时要付出性能损失，那为什么还要使用它们呢？如果我们必须将其简化为规范列表，则有以下两个原因：

- When the operation(s) you would like to perform cannot be expressed using DataFrame manipulations.

  当您要执行的操作无法使用DataFrame操作表示时。

- When you want or need type-safety, and you’re willing to accept the cost of performance to achieve it。
    当您想要或需要类型安全性时，您愿意接受性能成本来实现它。

Let’s explore these in more detail. There are some operations that cannot be expressed using the Structured APIs we have seen in the previous chapters. Although these are not particularly common, you might have a large set of business logic that you’d like to encode in one specific function instead of in SQL or DataFrames. This is an appropriate use for Datasets. Additionally, the Dataset API is type-safe. Operations that are not valid for their types, say subtracting two string types, will fail at compilation time not at runtime. If correctness and bulletproof code is your highest priority, at the cost of some performance, this can be a great choice for you. This does not protect you from malformed data but can allow you to more elegantly handle and organize it.

让我们更详细地探讨这些。有些操作无法使用我们在前几章中看到的结构化API来表达。尽管这些并不是特别常见，但是您可能想使用一个特定的功能而不是SQL或DataFrames进行编码的大量业务逻辑。这是 Datasets 的适当用法。此外，Dataset API是类型安全的。对于其类型无效的操作（例如减去两个字符串类型）**将在编译时而不是在运行时失败**。如果正确性和安全代码是您的最高优先级，而以牺牲性能为代价，那么这对于您来说是个不错的选择。这不能保护您免受格式错误的数据的侵害，但可以使您更优雅地处理和组织数据。

Another potential time for which you might want to use Datasets is when you would like to reuse a variety of transformations of entire rows between single-node workloads and Spark workloads. If you have some experience with Scala, you might notice that Spark’s APIs reflect those of Scala Sequence Types, but they operate in a distributed fashion. In fact, Martin Odersky, the inventor of Scala, said just that in 2015 at Spark Summit Europe. Due to this, one advantage of using Datasets is that if you define all of your data and transformations as accepting case classes it is trivial to reuse them for both distributed and local workloads. Additionally, when you collect your DataFrames to local disk, they will be of the correct class and type, sometimes making further manipulation easier.

您可能希望使用 Dataset 的另一个潜在时间是，您想在单节点工作负载和Spark工作负载之间重用整个行的各种转换时。如果您有使用Scala的经验，您可能会注意到Spark的API反映了Scala序列类型的API，但是它们以分布式方式运行。实际上，Scala的发明者马丁·奥德斯基（Martin Odersky）在2015年欧洲Spark峰会上就这样说过。因此，使用 Dataset 的一个优势是，如果您将所有数据和转换定义为接受案例类，那么将它们重用于分布式和本地工作负载就都很简单。另外，当您将DataFrame收集到本地磁盘时，它们将具有正确的类和类型，有时使进一步的操作变得容易。

Probably the most popular use case is to use DataFrames and Datasets in tandem, manually trading off between performance and type safety when it is most relevant for your workload. This might be at the end of a large, DataFrame-based extract, transform, and load (ETL) transformation when you’d like to collect data to the driver and manipulate it by using single-node libraries, or it might be at the beginning of a transformation when you need to perform per-row parsing before performing filtering and further manipulation in Spark SQL.

可能最流行的用例是串联使用DataFrame和Dataset，在与您的工作负载最相关的性能和类型安全之间进行手动权衡。当您想将数据收集到驱动程序（driver）并使用单节点库对其进行操作时，这可能是在大型的，基于DataFrame的提取，转换和加载（ETL）转换的结尾，或者可能在需要在Spark SQL中执行过滤和进一步处理之前需要执行每行解析的转换的开始。

## <font color="#9a161a">Creating Datasets</font>

Creating Datasets is somewhat of a manual operation, requiring you to know and define the schemas ahead of time.

创建数据集有些是手动操作，需要您提前了解和定义模式。

### <font color="#00000">In Java: Encoders</font>

Java Encoders are fairly simple, you simply specify your class and then you’ll encode it when you come upon your DataFrame (which is of type `Dataset<Row>`):

Java编码器非常简单，只需指定您的类，然后在遇到DataFrame（类型为 `Dataset<Row>` ）时对它进行编码：

```java
import org.apache.spark.sql.Encoders;

public class Flight implements Serializable{
    String DEST_COUNTRY_NAME;
    String ORIGIN_COUNTRY_NAME;
    Long DEST_COUNTRY_NAME;
}

Dataset<Flight> flights = spark.read
.parquet("/data/flight-data/parquet/2010-summary.parquet/")
.as(Encoders.bean(Flight.class));
```

### <font color="#00000">In Scala: Case Classes</font>

To create Datasets in Scala, you define a Scala case class. A case class is a regular class that has the following characteristics:

要在Scala中创建数据集，您需要定义一个Scala案例类。案例类是具有以下特征的常规类：

- Immutable 不可更改
- Decomposable through pattern matching 通过模式匹配可分解
- Allows for comparison based on structure instead of reference  允许基于结构而不是引用进行比较
- Easy to use and manipulate  易于使用和操作

These traits make it rather valuable for data analysis because it is quite easy to reason about a case class. Probably the most important feature is that case classes are immutable and allow for comparison by structure instead of value.

这些特征使其对于数据分析非常有价值，因为对于案例类进行推理非常容易。可能最重要的特征是案例类是不可变的，并允许按结构而不是值进行比较。

Here’s how the Scala documentation describes it:

以下是Scala文档的描述方式：

- immutability frees you from needing to keep track of where and when things are mutated.  不可变性使您无需跟踪发生突变的位置和时间

- Comparison-by-value allows you to compare instances as if they were primitive values—no more uncertainty regarding whether instances of a class are compared by value or reference. 按值比较允许您将实例视为原始值进行比较——不再不确定是否通过值或引用比较类的实例。

- Pattern matching simplifies branching logic, which leads to less bugs and more readable code.

    模式匹配可简化分支逻辑，从而减少错误并提高可读性。

These advantages carry over to their usage within Spark, as well.

这些优势也可以延续到Spark中。

To begin creating a Dataset, let’s define a case class for one of our datasets:

要开始创建数据集，请为我们的一个数据集定义一个案例类：

```scala
case class Flight(DEST_COUNTRY_NAME: String,
	ORIGIN_COUNTRY_NAME: String, count: BigInt)
```

Now that we defined a case class, this will represent a single record in our dataset. More succinctly, we now have a Dataset of Flights. This doesn’t define any methods for us, simply the schema. When we read in our data, we’ll get a DataFrame. However, we simply use the as method to cast it to our specified row type:

现在我们定义了一个案例类，它将代表我们 dataset 中的一条记录。更简洁地说，我们现在有了一个航班数据集。这并没有为我们定义任何方法，仅是模式。读取数据后，我们将获得一个DataFrame。但是，我们仅使用as方法将其强制转换为指定的行类型：

```scala
val flightsDF = spark.read.parquet("/data/flight-data/parquet/2010-summary.parquet/")
val flights = flightsDF.as[Flight]
```

## <font color="#9a161a">Actions</font>

Even though we can see the power of Datasets, what’s important to understand is that actions like collect, take, and count apply to whether we are using Datasets or DataFrames:

即使我们可以看到 Datasets 的强大能力，但重要的是要了解，诸如  collect, take,  和 count 的 action（动作，算子） 适用于我们使用的不管是 Datasets 还是 DataFrame：

```scala
flights.show(2)
```

```txt
+-----------------+-------------------+-----+
|DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|
+-----------------+-------------------+-----+
| United States   |    Romania        | 1   |
| United States   |    Ireland        | 264 |
+-----------------+-------------------+-----+
```

You’ll also notice that when we actually go to access one of the case classes, we don’t need to do any type coercion, we simply specify the named attribute of the case class and get back, not just the expected value but the expected type, as well:

您还会注意到，当我们实际上要访问其中一个案例类时，我们不需要执行任何类型强制转换，我们只需指定案例类已经命名的属性并获取，不仅返回期望值，还返回预期类型：

```scala
flights.first.DEST_COUNTRY_NAME // United States
```

## <font color="#9a161a">Transformations</font>

Transformations on Datasets are the same as those that we saw on DataFrames. Any transformation that you read about in this section is valid on a Dataset, and we encourage you to look through the specific sections on relevant aggregations or joins.

Datasets 的转换与我们在 DataFrame 上看到的转换相同。您在本节中了解的任何转换都对 Dataset 有效，我们建议您仔细阅读有关聚合或连接的特定部分。

In addition to those transformations, Datasets allow us to specify more complex and strongly typed transformations than we could perform on DataFrames alone because we manipulate raw Java Virtual Machine (JVM) types. To illustrate this raw object manipulation, let’s filter the Dataset that you just created.

除了这些转换之外，数据集还允许我们指定比单独在 DataFrames 上执行的更复杂，类型更强的转换，因为我们可以处理原始的Java虚拟机（JVM）类型。为了说明这种原始对象的操作，让我们过滤刚刚创建的数据集。

### <font color="#00000">Filtering</font>

 Let’s look at a simple example by creating a simple function that accepts a Flight and returns a Boolean value that describes whether the origin and destination are the same. This is not a UDF (at least, in the way that Spark SQL defines UDF) but a generic function.

让我们看一个简单的例子，创建一个简单的函数，该函数接受一个Flight并返回一个布尔值，该值描述起点和终点是否相同。这不是UDF（至少以Spark SQL定义UDF的方式），而是通用函数。

---

<p><center><font face="constant-width" color="#737373" size=3><strong>TIP 提示</strong></font></center></P>
You’ll notice in the following example that we’re going to create a function to define this filter. This is an important difference from what we have done thus far in the book. By specifying a function, we are forcing Spark to evaluate this function on every row in our Dataset. This can be very resource intensive. For simple filters it is always preferred to write SQL expressions. This will greatly reduce the cost of filtering out the data while still allowing you to manipulate it as a Dataset later on:

在以下示例中，您会注意到我们将创建一个函数来定义此过滤器。这与到目前为止我们在书中所做的是一个重要的区别。通过指定一个函数，我们迫使Spark在数据集中的每一行上计算这个函数。这可能会占用大量资源。对于简单的过滤器，总是首选编写SQL表达式。这将大大降低过滤数据的成本，同时仍允许您稍后将其作为 Dataset 进行操作：

---

```scala
def originIsDestination(flight_row: Flight): Boolean = {
	return flight_row.ORIGIN_COUNTRY_NAME == flight_row.DEST_COUNTRY_NAME
}
```

We can now pass this function into the filter method specifying that for each row it should verify that this function returns true and in the process will filter our Dataset down accordingly:

现在，我们可以将此函数传递到filter方法中，指定它应针对每一行验证该函数返回true，并在此过程中相应地过滤掉我们的数据集：

```scala
flights.filter(flight_row => originIsDestination(flight_row)).first()
```

The result is:

结果是：

```txt
Flight = Flight(United States,United States,348113)
```

As we saw earlier, this function does not need to execute in Spark code at all. Similar to our UDFs, we can use it and test it on data on our local machines before using it within Spark.

如我们先前所见，此功能根本不需要在Spark代码中执行。与我们的UDF类似，在Spark中使用它之前，我们可以使用它并在本地计算机上的数据上对其进行测试。

For example, this dataset is small enough for us to collect to the driver (as an Array of Flights) on which we can operate and perform the exact same filtering operation:

例如，此数据集足够小，我们可以收集给驱动程序（作为航班数组），在该驱动程序上我们可以进行操作并执行完全相同的过滤操作：

```scala
flights.collect().filter(flight_row => originIsDestination(flight_row))
```

The result is:

结果是：

```scala
Array[Flight] = Array(Flight(United States,United States,348113))
```

We can see that we get the exact same answer as before.

我们可以看到我们得到了与以前完全相同的答案。

### <font color="#00000">Mapping</font>

Filtering is a simple transformation, but sometimes you need to map one value to another value. We did this with our function in the previous example: it accepts a flight and returns a Boolean, but other times we might actually need to perform something more sophisticated like extract a value, compare a set of values, or something similar.

过滤是一个简单的转换，但是有时您需要将一个值映射到另一个值。我们在上一个示例中使用函数进行了此操作：它接受一个 flight 并返回一个布尔值，但是有时我们实际上可能需要执行更复杂的操作，例如提取值，比较一组值或类似操作。

The simplest example is manipulating our Dataset such that we extract one value from each row. This is effectively performing a DataFrame like select on our Dataset. Let’s extract the destination:

最简单的示例是处理 Dataset ，以便从每一行提取一个值。这实际上是在我们的 Dataset 上执行类似于select的DataFrame。让我们提取目的地：

```scala
val destinations = flights.map(f => f.DEST_COUNTRY_NAME)
```

Notice that we end up with a Dataset of type String. That is because Spark already knows the JVM type that this result should return and allows us to benefit from compile-time checking if, for some reason, it is invalid.

注意，我们最终得到的是String类型的 Dataset。这是因为Spark已经知道该结果应返回的JVM类型，并允许我们从编译时检查中受益（如果出于某种原而无效）。

We can collect this and get back an array of strings on the driver:

我们可以收集这些并获取驱动程序上的字符串数组：

```scala
val localDestinations = destinations.take(5)
```

This might feel trivial and unnecessary; we can do the majority of this right on DataFrames. We in fact recommend that you do this because you gain so many benefits from doing so. You will gain advantages like code generation that are simply not possible with arbitrary user-defined functions. However, this can come in handy with much more sophisticated row-by-row manipulation.

这可能是琐碎且不必要的。我们可以在DataFrames上行使大部分权利。实际上，我们建议您这样做，因为这样做会带来很多好处。您将获得诸如代码生成之类的优势，而这些优势是任意用户定义函数根本无法实现的。但是，这可以通过更复杂的逐行操作来派上用场。

## <font color="#9a161a">Joins </font>

Joins, as we covered earlier, apply just the same as they did for DataFrames. However Datasets also provide a more sophisticated method, the `joinWith` method. `joinWith` is roughly equal to a co-group (in RDD terminology) and you basically end up with two nested Datasets inside of one. Each column represents one Dataset and these can be manipulated accordingly. This can be useful when you need to maintain more information in the join or perform some more sophisticated manipulation on the entire result, like an advanced map or filter.

如前所述，连接的应用方式与对 DataFrame 的应用方式相同。但是，数据集还提供了更复杂的方法 `joinWith` 方法。`joinWith` 大致等于一个 co-group（在RDD术语中），您基本上在一个内部拥有两个嵌套的数据集。每一列代表一个数据集，可以相应地对其进行操作。当您需要在连接中维护更多信息或对整个结果执行一些更复杂的操作（例如高级映射或过滤器）时，这将很有用。

Let’s create a fake flight metadata dataset to demonstrate `joinWith`:

我们创建一个假的航班元数据数据集来演示 `joinWith`：

```scala
case class FlightMetadata(count: BigInt, randomData: BigInt)

val flightsMeta = spark.range(500).map(x => (x, scala.util.Random.nextLong))
.withColumnRenamed("_1", "count").withColumnRenamed("_2", "randomData")
.as[FlightMetadata]

val flights2 = flights
.joinWith(flightsMeta, flights.col("count") === flightsMeta.col("count"))
```

Notice that we end up with a Dataset of a sort of key-value pair, in which each row represents a Flight and the Flight Metadata. We can, of course, query these as a Dataset or a DataFrame with complex types:

请注意，我们最后得到的是一种键值对的数据集，其中每一行代表一个Flight和Flight Metadata。当然，我们可以将它们查询为具有复杂类型的 Dataset 或 DataFrame：

```scala
flights2.selectExpr("_1.DEST_COUNTRY_NAME")
```

We can collect them just as we did before:

我们可以像以前一样收集它们：

```scala
flights2.take(2)

Array[(Flight, FlightMetadata)] = Array((Flight(United States,Romania,1),...
```

```scala
val flights2 = flights.join(flightsMeta, Seq("count"))
```

We can always define another Dataset to gain this back. It’s also important to note that there are no problems joining a DataFrame and a Dataset—we end up with the same result:

我们总是可以定义另一个数据集来获得回报。同样重要的是要注意，将DataFrame和Dataset连接起来没有问题——我们最终得到了相同的结果：

```scala
val flights2 = flights.join(flightsMeta.toDF(), Seq("count"))
```

## <font color="#9a161a">Grouping and Aggregations</font>

Grouping and aggregations follow the same fundamental standards that we saw in the previous aggregation chapter, so `groupBy` `rollup` and `cube` still apply, but these return DataFrames instead of Datasets (you lose type information):

分组和聚合遵循在上一聚合章中看到的相同基本标准，因此 `groupBy`, `rollup`和 `cube` 仍然适用，但是它们返回DataFrames而不是Datasets（您会丢失类型信息）：

 ```scala
flights.groupBy("DEST_COUNTRY_NAME").count()
 ```

This often is not too big of a deal, but if you want to keep type information around there are other groupings and aggregations that you can perform. An excellent example is the `groupByKey` method. This allows you to group by a specific key in the Dataset and get a typed Dataset in return. This function, however, doesn’t accept a specific column name but rather a function. This makes it possible for you to specify more sophisticated grouping functions that are much more akin to something like this:

这通常没什么大不了的，但是如果您想保留类型信息，则可以执行其他分组和聚合。一个很好的例子是`groupByKey`方法。这使您可以按数据集中的特定键进行分组，并获取返回的类型化数据集。但是，此函数不接受特定的列名，而是接受一个函数。这使您可以指定更复杂的分组功能，这些功能类似于以下内容： 

```scala
flights.groupByKey(x => x.DEST_COUNTRY_NAME).count()
```

Although this provides flexibility, it’s a trade-off because now we are introducing JVM types as well as functions that cannot be optimized by Spark. This means that you will see a performance difference and we can see this when we inspect the explain plan. In the following, you can see that we are effectively appending a new column to the DataFrame (the result of our function) an d then performing the grouping on that:

尽管这提供了灵活性，但是这是一个折衷，因为现在我们引入了JVM类型以及Spark无法优化的功能。这意味着您将看到性能差异，并且在检查解释计划时可以看到此差异。在下面的内容中，您可以看到我们正在有效地向DataFrame追加新列（我们函数的结果），然后对该分组执行分组：

```scala
flights.groupByKey(x => x.DEST_COUNTRY_NAME).count().explain
```

```txt
== Physical Plan ==
*HashAggregate(keys=[value#1396], functions=[count(1)])
	+- Exchange hashpartitioning(value#1396, 200)
	   +- *HashAggregate(keys=[value#1396], functions=[partial_count(1)])
		  +- *Project [value#1396]
			 +- AppendColumns <function1>, newInstance(class ...
			 [staticinvoke(class org.apache.spark.unsafe.types.UTF8String, ...
			 +- *FileScan parquet [D...
```

After we perform a grouping with a key on a Dataset, we can operate on the Key Value Dataset with functions that will manipulate the groupings as raw objects:

在对 Dataset 上的键执行分组之后，我们可以对键值数据集进行操作，该函数具有将分组作为原始对象进行操作的功能：

```scala
def grpSum(countryName:String, values: Iterator[Flight]) = {
	values.dropWhile(_.count < 5).map(x => (countryName, x))
} 

flights.groupByKey(x => x.DEST_COUNTRY_NAME).flatMapGroups(grpSum).show(5)
```

```txt
+--------+--------------------+
|   _1   |        _2          |
+--------+--------------------+
|Anguilla|[Anguilla,United ...|
|Paraguay|[Paraguay,United ...|
| Russia |[Russia,United St...|
| Senegal|[Senegal,United S...|
| Sweden |[Sweden,United St...|
+--------+--------------------+
```

```scala
def grpSum2(f:Flight):Integer = {
	1
}

flights.groupByKey(x => x.DEST_COUNTRY_NAME).mapValues(grpSum2).count().take(5)
```

We can even create new manipulations and define how groups should be reduced:

我们甚至可以创建新的操作并定义应如何减少组：

```scala
def sum2(left:Flight, right:Flight) = {
	Flight(left.DEST_COUNTRY_NAME, null, left.count + right.count)
} 

flights.groupByKey(x => x.DEST_COUNTRY_NAME).reduceGroups((l, r) => sum2(l, r))
.take(5)
```

It should be straightfoward enough to understand that this is a more expensive process than aggregating immediately after scanning, especially because it ends up in the same end result:

应该足够直观地了解到，与扫描后立即进行聚合相比，这是一个更昂贵的过程，尤其是因为它最终会达到相同的最终结果：

```txt
flights.groupBy("DEST_COUNTRY_NAME").count().explain
== Physical Plan ==
*HashAggregate(keys=[DEST_COUNTRY_NAME#1308], functions=[count(1)])
+- Exchange hashpartitioning(DEST_COUNTRY_NAME#1308, 200)
   +- *HashAggregate(keys=[DEST_COUNTRY_NAME#1308], functions=[partial_count(1)])
	  +- *FileScan parquet [DEST_COUNTRY_NAME#1308] Batched: tru...
```

This should motivate using Datasets only with user-defined encoding surgically and only where it makes sense. This might be at the beginning of a big data pipeline or at the end of one.

这应该仅通过外科手术并且仅在有意义的地方激发使用 Datasets 的动机。这可能是在大数据管道的开始或结束时。

## <font color="#9a161a">Conclusion</font>

In this chapter, we covered the basics of Datasets and provided some motivating examples. Although short, this chapter actually teaches you basically all that you need to know about Datasets and how to use them. It can be helpful to think of them as a blend between the higher-level Structured APIs and the low-level RDD APIs, which is the topic of Chapter 12.

在本章中，我们介绍了 Datasets  的基础知识，并提供了一些激励性的示例。尽管简短，但本章实际上教会了您基本上需要了解的有关数据集以及如何使用它们的所有知识。将它们视为高级结构化API和低级RDD API之间的混合会很有帮助，这是第12章的主题。

 
