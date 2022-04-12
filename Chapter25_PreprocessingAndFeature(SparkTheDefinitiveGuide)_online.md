---
title: 翻译 Chapter 25  Preprocessing and Feature Engineering
date: 2019-08-26
copyright: true
categories: English,中文
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 25 Preprocessing and Feature Engineering 预处理和特征工程

Any data scientist worth her salt knows that one of the biggest challenges (and time sinks) in advanced analytics is preprocessing. It’s not that it’s particularly complicated programming, but rather that it requires deep knowledge of the data you are working with and an understanding of what your model needs in order to successfully leverage this data. This chapter covers the details of how you can use Spark to perform preprocessing and feature engineering. We’ll walk through the core requirements you’ll need to meet in order to train an MLlib model in terms of how your data is structured. We will then discuss the different tools Spark makes available for performing this kind of work. 

值得任何数据科学家都知道的是高级分析中最大的挑战之一（和时间汇集）是预处理。这不是特别复杂的编程，而是需要深入了解您正在使用的数据，并了解您的模型需要什么才能成功利用这些数据。本章介绍了如何使用 Spark  执行预处理和功能工程的详细信息。我们将逐步介绍您需要满足的核心要求，以便根据数据的结构来训练 MLlib 模型。然后，我们将讨论 Spark 可用于执行此类工作的不同工具。

## <font color="#9a161a">Formatting Models According to Your Use Case 根据您的使用案例格式化模型</font>

To preprocess data for Spark’s different advanced analytics tools, you must consider your end objective. The following list walks through the requirements for input data structure for each advanced analytics task in MLlib:

要为 Spark 的不同高级分析工具预处理数据，您必须考虑最终目标。以下列表详细研究了 MLlib 中每个高级分析任务的输入数据结构要求：

- In the case of most classification and regression algorithms, you want to **get** your data into a column of type Double to represent the label and a column of type Vector (either dense or sparse) to represent the features. 

    对于大多数分类和回归算法，您希望将数据放入 Double 类型的列中以表示标签，并使用Vector类型（密集或稀疏）来表示特征。

- In the case of recommendation, you want to get your data into a column of users, a column of
    items (say movies or books), and a column of ratings. 

    在推荐的案例中，你想将数据放入到一列用户，一列项目（比如电影或书籍）和一列评分中去。

- In the case of unsupervised learning, a column of type Vector (either dense or sparse) is needed to represent the features. 

    在无监督学习的情况下，需要一个Vector类型（密集或稀疏）来表示特征。

- In the case of graph analytics, you will want a DataFrame of vertices and a DataFrame of edges. 

    在图形分析的情况下，您将需要顶点的 DataFrame 和边的 DataFrame 。

The best way to get your data in these formats is through transformers. Transformers are functions that
accept a DataFrame as an argument and return a new DataFrame as a response. This chapter will
focus on what transformers are relevant for particular use cases rather than attempting to enumerate
every possible transformer. 

以这些格式获取数据的最佳方式是通过转换器。转换器是接受DataFrame作为参数并返回一个新的DataFrame的函数。本章将重点介绍转换器与特定用户案例相关的内容，而不是试图列举所有可能的转换器。

---

<center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong></font></center>
Spark provides a number of transformers as part of the `org.apache.spark.ml.feature` package. The corresponding package in Python is `pyspark.ml.feature`. New transformers are constantly popping up in Spark MLlib and therefore it is impossible to include a definitive list in this book. The most up-to-date information can be found on the [Spark documentation site](http://spark.apache.org/docs/latest/ml-features.html). 

Spark 提供了许多转换器作为 `org.apache.spark.ml.feature` 包的一部分。 Python 中相应的包是`pyspark.ml.feature` 。新的转换器不断出现在 Spark MLlib 中，因此不可能在本书中包含明确的列表。可以在[Spark文档站点](http://spark.apache.org/docs/latest/ml-features.html)上找到最新信息。

---

Before we proceed, we’re going to read in several different sample datasets, each of which has different properties we will manipulate in this chapter : 

在我们继续之前，我们将阅读几个不同的样本数据集，每个样本数据集都有不同的属性，我们将在本章中操作：

```scala
// in Scala
val sales = spark.read.format("csv")
.option("header", "true")
.option("inferSchema", "true")
.load("/data/retail-data/by-day/*.csv")
.coalesce(5)
.where("Description IS NOT NULL")

val fakeIntDF = spark.read.parquet("/data/simple-ml-integers")
var simpleDF = spark.read.json("/data/simple-ml")
val scaleDF = spark.read.parquet("/data/simple-ml-scaling")
```

```python
# in Python
sales = spark.read.format("csv")\
.option("header", "true")\
.option("inferSchema", "true")\
.load("/data/retail-data/by-day/*.csv")\
.coalesce(5)\
.where("Description IS NOT NULL")
fakeIntDF = spark.read.parquet("/data/simple-ml-integers")
simpleDF = spark.read.json("/data/simple-ml")
scaleDF = spark.read.parquet("/data/simple-ml-scaling")
```

In addition to this realistic sales data, we’re going to use several simple synthetic datasets as well. `FakeIntDF`, `simpleDF`, and `scaleDF` all have very few rows. This will give you the ability to focus on the exact data manipulation we are performing instead of the various inconsistencies of an particular dataset. Because we’re going to be accessing the sales data a number of times, we’re going to cache it so we can read it efficiently from memory as opposed to reading it from disk every time we need it. Let’s also check out the first several rows of data in order to better understand what’s in the dataset: 

除了这些现实生活中的销售数据，我们还将使用几个简单的合成数据集。 `FakeIntDF`，`simpleDF` 和 `scaleDF` 都只有很少的行。 这将使您能够专注于我们正在执行的确切数据操作，而不是特定数据集的各种不一致性。 因为我们将多次访问销售数据，所以我们将对其进行缓存，以便我们可以从内存中有效地读取它，而不是每次需要时从磁盘读取它。 我们还要查看前几行数据，以便更好地了解数据集中的内容：

```scala
sales.cache()
sales.show()
```

```text
+---------+---------+--------------------+--------+-------------------+---------
|InvoiceNo|StockCode|     Description    |Quantity|    InvoiceDate    |UnitPr...
+---------+---------+--------------------+--------+-------------------+---------
|  580538 |  23084  | RABBIT NIGHT LIGHT |   48   |2011-12-05 08:38:00| 1...
...
|  580539 |  22375  |AIRLINE BAG VINTA...|   4    |2011-12-05 08:39:00| 4...
+---------+---------+--------------------+--------+-------------------+---------
```

---

<center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong></font></center>
It is important to note that we filtered out null values here. MLlib does not always play nicely with null values at this point in time. This is a frequent cause for problems and errors and a great first step when you are debugging. Improvements are also made with every Spark release to improve algorithm handling of null values. 

请务必注意，我们在此处过滤掉了空值。 在这个时间点，MLlib并不总能很好地处理空值。 这是导致问题和错误的常见原因，也是调试时的第一步。 每个Spark版本都进行了改进，以改进空值的算法处理。

---

## <font color="#9a161a">Transformers 转换器</font>

We discussed transformers in the previous chapter, but it’s worth reviewing them again here. Transformers are functions that convert raw data in some way. This might be to create a new interaction variable (from two other variables), to normalize a column, or to simply turn it into a Double to be input into a model. Transformers are primarily used in preprocessing or feature generation.

我们在前一章讨论过转换器，但值得再次回顾一下。转换器是以某种方式转换原始数据的函数。这可能是创建一个新的交互变量（来自其他两个变量），标准化一个列，或者简单地将其转换为 Double 以输入到模型中。转换器主要用于预处理或特征生成。

Spark’s transformer only includes a transform method. This is because it will not change based on the input data. Figure 25-1 is a simple illustration. On the left is an input DataFrame with the column to be manipulated. On the right is the input DataFrame with a new column representing the output transformation.

Spark 的转换器只包含一种转换方法。这是因为它不会根据输入数据而改变。图25-1是一个简单的图示。左侧是输入 DataFrame，其中包含要操作的列。右侧是输入DataFrame，其中一个新列表示输出转换。

![1567564236559](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter25/1567564236559.png)

The Tokenizer is an example of a transformer. It tokenizes a string, splitting on a given character, and has nothing to learn from our data; it simply applies a function. We’ll discuss the tokenizer in more depth later in this chapter, but here’s a small code snippet  showing how a tokenizer is built to accept the input column, how it transforms the data, and then the output from that transformation: 

Tokenizer是转换器的一个例子。它符号化一个字符串，分裂给定的文本，并且没有任何东西可以从我们的数据中学习；它只是应用一个功能。我们将在本章后面更深入地讨论符号生成器，但这里有一个小代码片段，显示如何构建标记生成器以接受输入列，如何转换数据，然后来自该转换的输出：

```scala
// in Scala
import org.apache.spark.ml.feature.Tokenizer
val tkn = new Tokenizer().setInputCol("Description")
tkn.transform(sales.select("Description")).show(false)
```

```text
+-----------------------------------+------------------------------------------+
|           Description             |          tok_7de4dfc81ab7__output        |
+-----------------------------------+------------------------------------------+
|          RABBIT NIGHT LIGHT       |           [rabbit, night, light]         |
|          DOUGHNUT LIP GLOSS       |           [doughnut, lip, gloss]         |
...
|AIRLINE BAG VINTAGE WORLD CHAMPION | [airline, bag, vintage, world, champion] |
|AIRLINE BAG VINTAGE JET SET BROWN  | [airline, bag, vintage, jet, set, brown] |
+-----------------------------------+------------------------------------------+
```

## <font color="#9a161a">Estimators for Preprocessing 预处理的估记器</font>

Another tool for preprocessing are estimators. An estimator is necessary when a transformation you would like to perform must be initialized with data or information about the input column (often derived by doing a pass over the input column itself). For example, if you wanted to scale the values in our column to have mean zero and unit variance, you would need to perform a pass over the entire data in order to calculate the values you would use to normalize the data to mean zero and unit variance. In effect, an estimator can be a transformer configured according to your particular input data. In simplest terms, you can either blindly apply a transformation (a “regular” transformer type) or perform a transformation based on your data (an estimator type). Figure 25-2 is a simple illustration of an estimator fitting to a particular input dataset, generating a transformer that is then applied to the input dataset to append a new column (of the transformed data). 

预处理的另一个工具是估记器。当您想要执行的转换必须使用有关输入列的数据或信息进行初始化时（通常通过对输入列本身进行传递），必须使用估计器。例如，如果要将列中的值缩放为具有均值零和单位方差，则需要对整个数据执行传递，以便计算用于将数据标准化为零和单位方差的值。实际上，估计器可以是根据您的特定输入数据配置的转换器。简单来说，您可以盲目地应用转换（“常规”转换器类型）或根据您的数据执行转换（估计器类型）。图25-2是拟合特定输入数据集的估计器的简单图示，生成转换器，然后将其应用于输入数据集以附加（已经转换的数据的）新列。

![1567564499836](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter25/1567564499836.png)

An example of this type of estimator is the `StandardScaler`, which scales your input column according to the range of values in that column to have a zero mean and a variance of 1 in each dimension. For that reason it must first perform a pass over the data to create the transformer. Here’s a sample code snippet showing the entire process, as well as the output: 

此类估计器的一个示例是 `StandardScaler`，它根据该列中的值范围缩放输入列，使其在每个维度中具有零均值和方差1。因此，它必须首先对数据执行传递以创建变换器。这是一个示例代码片段，显示整个过程以及输出：

```scala
// in Scala
import org.apache.spark.ml.feature.StandardScaler
val ss = new StandardScaler().setInputCol("features")
ss.fit(scaleDF).transform(scaleDF).show(false)
```

```text
+---+--------------+------------------------------------------------------------+
|id |   features   |             stdScal_d66fbeac10ea__output                   |
+---+--------------+------------------------------------------------------------+
| 0 |[1.0,0.1,-1.0]|[1.1952286093343936,0.02337622911060922,-0.5976143046671968]|
...
| 1 |[3.0,10.1,3.0]| [3.5856858280031805,2.3609991401715313,1.7928429140015902] |
+---+--------------+------------------------------------------------------------+
```

We will use both estimators and transformers throughout and cover more about these particular estimators (and add examples in Python) later on in this chapter. 

我们将在整个过程中使用估计器和转换器，并在本章后面详细介绍这些特定的估计器（并在Python中添加示例）。

### <font color="#00000">Transformer Properties 转换器属性</font>

All transformers require you to specify, at a minimum, the `inputCol`  and the `outputCol`, which represent the column name of the input and output, respectively. You set these with `setInputCol` and `setOutputCol`. There are some defaults (you can find these in the documentation), but it is a best practice to manually specify them yourself for clarity. In addition to input and output columns, all transformers have different parameters that you can tune (whenever we mention a parameter in this chapter you must set it with a `set()` method). In Python, we also have another method to set these values with keyword arguments to the object’s constructor. We exclude these from the examples in the next chapter for consistency. Estimators require you to fit the transformer to your particular dataset and then call transform on the resulting object.

所有转换器都要求您至少指定 `inputCol` 和 `outputCol`，它们分别表示输入和输出的列名。您可以使用`setInputCol` 和 `setOutputCol` 设置它们。有一些默认值（您可以在文档中找到这些默认值），但为了清晰起见，最好自己手动指定它们。除了输入和输出列之外，所有转换器都有不同的参数可以调整（每当我们在本章中提到参数时，必须使用 `set()` 方法设置它）。在 Python 中，我们还有另一种方法可以使用关键字参数为对象的构造函数设置这些值。为了保持一致性，我们将在下一章的示例中排除这些。估计器要求您将转换器拟合特定数据集，然后对结果对象调用transform。

---

<center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong></font></center>
Spark MLlib stores metadata about the columns it uses in each DataFrame as an attribute on the column itself. This allows  it to properly store (and annotate) that a column of Doubles may actually represent a series of categorical variables instead of continuous values. However, metadata won’t show up when you print the schema or the DataFrame.

Spark MLlib 将有关其在每个DataFrame中使用的列的元数据存储为列本身的属性。 这允许它正确地存储（和注释）双列列实际上可以表示一系列分类变量而不是连续值。 但是，打印模式（schema）或 DataFrame 时，元数据不会显示。

---

## <font color="#9a161a">High-Level Transformers 高层（接口的）转换器</font>

High-level transformers, such as the RFormula we saw in the previous chapter, allow you to concisely specify a number of transformations in one. These operate at a “high level”, and allow yow to avoid doing data manipulations or transformations one by one. In general, you should try to use the highest level transformers you can, in order to minimize the risk of error and help you focus on the business problem instead of the smaller details of implementation. While this is not always possible, it’s a good objective.

高级变换器，例如我们在前一章中看到的RFormula，允许您在一个变换器中简明地指定多个变换。它们在“高级别”运行，并允许您避免逐个进行数据操作或转换。通常，您应该尝试使用最高级别的变换器，以便最大限度地降低出错风险，并帮助您专注于业务问题而不是较小的实现细节。虽然这并非总是可行，但这是一个很好的目标。

### <font color="#00000">RFormula</font>

The `RFormula` is the easiest transfomer to use when you have “conventionally” formatted data. Spark borrows this transformer from the R language to make it simple to declaratively specify a set of transformations for your data. With this transformer, values can be either numerical or categorical and you do not need to extract values from strings or manipulate them in any way. The `RFormula` will automatically handle categorical inputs (specified as strings) by performing something called one-hot encoding. In brief, one-hot encoding converts a set of values into a set of binary columns specifying whether or not the data point has each particular value (we’ll discuss one-hot encoding in more depth later in the chapter). With the `RFormula`, numeric columns will be cast to `Double` but will not be one-hot encoded. If the label column is of type `String`, it will be first transformed to `Double` with `StringIndexer `.

当您拥有“常规”格式的数据时，`RFormula` 是最容易使用的变换器。 Spark 从 R 语言借用了这个转换器，使得声明性地为数据指定一组转换变得简单。使用此转换器，值可以是数值类型或类别类型，您不需要从字符串中提取值或以任何方式操纵它们。 `RFormula` 将通过执行称为独热编码的操作自动处理类别类型输入（指定为字符串）。简而言之，独热编码将一组值转换为一组二进制列，指定数据点是否具有每个特定值（我们将在本章后面更深入地讨论独热编码）。使用 `RFormula`，数字列将转换为 `Double`，但不会进行独热编码。如果 `label` 列的类型为 `String`，则它将首先使用 `StringIndexer` 转换为 `Double`。

---

<center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong></font></center>
Automatic casting of numeric columns to `Double` without one-hot encoding has some important implications. If you have numerically valued categorical variables, they will only be cast to `Double`, implicitly specifying an order. It is important to ensure the input types correspond to the expected conversion. If you have categorical variables that really have no order relation, they should be cast to `String`. You can also manually index columns (see “Working with Categorical Features”).

将数字列自动转换为 `Double` 而不使用独热编码具有一些重要作用。如果您有数值分类变量，它们将仅转换为`Double`，隐式指定顺序。确保输入类型与预期转换相对应非常重要。如果您的分类变量确实没有顺序关系，则应将它们强制转换为 `String`。您也可以手动索引列（请参阅本文的“使用分类功能”小节）。

---

The RFormula allows you to specify your transformations in declarative syntax. It is simple to use once you understand the syntax. Currently, RFormula supports a limited subset of the R operators that in practice work quite well for simple transformations. The basic operators are : 

`RFormula` 允许您在声明性语法中指定转换。一旦理解了语法，它就很容易使用。目前，`RFormula` 支持 R 运算符的有限子集，这些运算符在实际上对于简单变换非常有效。基本运算符是：

| 运算符operator                                               | 含义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <font face="constant-width" color="#000000" size=4><strong>~</strong></font> | Separate target and terms<br/><br/>分隔目标和项              |
| <font face="constant-width" color="#000000" size=4><strong>+</strong></font> | Concat terms; “+ 0” means removing the intercept (this means that the y-intercept of the line that we will fit will be 0)<br/><br/>拼接项；“+0” 意味着移除截距（这意思是我们将拟合的直线的 y轴截距将会是0） |
| <font face="constant-width" color="#000000" size=4><strong>-</strong></font> | Remove a term; “- 1” means removing the intercept (this means that the y-intercept of the line that we will fit will be 0—yes, this does the same thing as “+ 0”<br/><br/>移除一个项；“-1” 与 “-0” 做同样的事情 |
| <font face="constant-width" color="#000000" size=4><strong>:</strong></font> | Interaction (multiplication for numeric values, or binarized categorical values)<br /><br/>交互（数值乘法或二进制分类值） |
| <font face="constant-width" color="#000000" size=4><strong>.</strong></font> | All columns except the target/dependent variable<br /><br/>除目标/因变量之外的所有列 |

RFormula also uses default columns of label and features to label, you guessed it, the label and the set of features that it outputs (for supervised machine learning). The models covered later on in this chapter by default require those column names, making it easy to pass the resulting transformed DataFrame into a model for training. If this doesn’t make sense yet, don’t worry—it’ll become clear once we actually start using models in later chapters.

RFormula还使用标签和特征的默认列来标记，您猜对了，标签和它输出的特征集（用于监督机器学习）。 默认情况下，本章后面介绍的模型需要这些列名称，以便将生成的转换后的 DataFrame 传递到训练模型中。 如果这还没有意义，请不要担心——一旦我们真正开始在后面的章节中使用模型，它就会变得清晰。

Let’s use RFormula in an example. In this case, we want to use all available variables (the <font face="constant-width" color="#000000" size=4><strong>.</strong></font>) and then specify an interaction between value1 and color and value2 and color as additional features to generate: 

我们在一个例子中使用 RFormula。 在这种案例下，我们希望使用所有可用变量（<font face="constant-width" color="#000000" size=4><strong>.</strong></font>），然后指定 value1 和 color 以及 value2 和 color 之间的交互作为生成的附加功能：

```scala
// in Scala
import org.apache.spark.ml.feature.RFormula
val supervised = new RFormula().setFormula("lab ~ . + color:value1 + color:value2")
supervised.fit(simpleDF).transform(simpleDF).show()
```

```python
# in Python
from pyspark.ml.feature import RFormula
supervised = RFormula(formula="lab ~ . + color:value1 + color:value2")
supervised.fit(simpleDF).transform(simpleDF).show()
```

```text
+-----+----+------+------------------+--------------------+-----+
|color| lab|value1|       value2     |      features      |label|
+-----+----+------+------------------+--------------------+-----+
|green|good|  1   |14.386294994851129|(10,[1,2,3,5,8],[...| 1.0 |
| blue| bad|  8   |14.386294994851129|(10,[2,3,6,9],[8....| 0.0 |
...
| red | bad|  1   | 38.97187133755819|(10,[0,2,3,4,7],[...| 0.0 |
| red | bad|  2   |14.386294994851129|(10,[0,2,3,4,7],[...| 0.0 |
+-----+----+------+------------------+--------------------+-----+
```

### <font color="#00000">SQL Transformers</font>

A SQLTransformer allows you to leverage Spark’s vast library of SQL-related manipulations just as you would a MLlib transformation. Any SELECT statement you can use in SQL is a valid transformation. The only thing you need to change is that instead of using the table name, you should just use the keyword THIS. You might want to use SQLTransformer if you want to formally codify some DataFrame manipulation as a preprocessing step, or try different SQL expressions for features during hyperparameter tuning. Also note that the output of this transformation will be appended as a column to the output DataFrame.

`SQLTransformer` 允许您像使用 MLlib 转换一样利用 Spark 庞大的 SQL 相关操作库。 在 SQL 中可以使用的任何 SELECT 语句都是有效的转换。 您需要更改的唯一事情是，您应该只使用关键字 THIS，而不是使用表名。 如果要将某些 DataFrame 操作正式编码为预处理步骤，或者在超参数调整期间尝试使用不同的 SQL 表达式，则可能需要使用 `SQLTransformer`。 另请注意，此转换的输出将作为列附加到输出 DataFrame。

You might want to use an `SQLTransformer` in order to represent all of your manipulations on the very rawest form of your data so you can version different variations of manipulations as transformers. This gives you the benefit of building and testing varying pipelines, all by simply swapping out transformers. The following is a basic example of using `SQLTransformer`: 

您可能希望使用  `SQLTransformer` 来表示对最新形式的数据的所有操作，以便您可以将不同的操作变体版本作为变换器。 通过简单地更换转换器，这为您提供了构建和测试不同管道的利好。 以下是使用 `SQLTransformer` 的基本示例：

```scala
// in Scala
import org.apache.spark.ml.feature.SQLTransformer
val basicTransformation = new SQLTransformer().
setStatement("""
SELECT sum(Quantity), count(*), CustomerID
FROM __THIS__
GROUP BY CustomerID
""")
basicTransformation.transform(sales).show()
```

```python
# in Python
from pyspark.ml.feature import SQLTransformer
basicTransformation = SQLTransformer()\
.setStatement("""
SELECT sum(Quantity), count(*), CustomerID
FROM __THIS__
GROUP BY CustomerID
""")
basicTransformation.transform(sales).show()
```

Here’s a sample of the output:

这是样本的输出：

```text
-------------+--------+----------+
|sum(Quantity)|count(1)|CustomerID|
+-------------+--------+----------+
|    119      |   62   | 14452.0  |
...
|    138      |   18   | 15776.0  |
+-------------+--------+----------+
```

For extensive samples of these transformations, refer back to Part II. 

有关这些转换的大量示例，请参阅第II部分。

### <font color="#00000">VectorAssembler</font>

The VectorAssembler is a tool you’ll use in nearly every single pipeline you generate. It helps concatenate all your features into one big vector you can then pass into an estimator. It’s used typically in the last step of a machine learning pipeline and takes as input a number of columns of Boolean, Double, or Vector. This is particularly helpful if you’re going to perform a number of manipulations using a variety of transformers and need to gather all of those results together.

VectorAssembler是您将在几乎每个生成的管道中使用的工具。 它有助于将所有特征连接成一个大向量，然后传递给估计器。 它通常用于机器学习管道的最后一步，并将多列Boolean，Double或Vector作为输入。 如果您要使用各种转换器执行大量操作并且需要将所有这些结果收集在一起，这将特别有用。

The output from the following code snippet will make it clear how this works: 

以下代码段的输出将清楚说明其工作原理：

```scala
// in Scala
import org.apache.spark.ml.feature.VectorAssembler
val va = new VectorAssembler().setInputCols(Array("int1", "int2", "int3"))
va.transform(fakeIntDF).show()
```
```python
# in Python
from pyspark.ml.feature import VectorAssembler
va = VectorAssembler().setInputCols(["int1", "int2", "int3"])va.transform(fakeIntDF).show()
```

```text
+----+----+----+--------------------------------------------+
|int1|int2|int3|VectorAssembler_403ab93eacd5585ddd2d__output|
+----+----+----+--------------------------------------------+
|  1 |  2 |  3 |            [1.0,2.0,3.0]                   |
|  4 |  5 |  6 |            [4.0,5.0,6.0]                   |
|  7 |  8 |  9 |            [7.0,8.0,9.0]                   |
+----+----+----+--------------------------------------------+
```

## <font color="#9a161a">Working with Continuous Features 使用连续特征</font>

Continuous features are just values on the number line, from positive infinity to negative infinity. There are two common transformers for continuous features. First, you can convert continuous features into categorical features via a process called bucketing, or you can scale and normalize your features according to several different requirements. These transformers will only work on Double types, so make sure you’ve turned any other numerical values to Double: 

连续特征只是数字线上的值，从正无穷大到负无穷大。 连续功能有两种常见的变压器。 首先，您可以通过称为bucketing 的过程将连续要素转换为分类要素，或者您可以根据多种不同要求对要素进行缩放和规范化。 这些变换器只适用于Double类型，因此请确保您已将任何其他数值转换为Double：

```scala
// in Scala
val contDF = spark.range(20).selectExpr("cast(id as double)")
```

```python
# in Python
contDF = spark.range(20).selectExpr("cast(id as double)")
```

### <font color="#00000">Bucketing 分桶</font>

The most straightforward approach to bucketing or binning is using the `Bucketizer`. This will split a given continuous feature into the buckets of your designation. You specify how buckets should be created via an array or list of `Double` values. This is useful because you may want to simplify the features in your dataset or simplify their representations for interpretation later on. For example, imagine you have a column that represents a person’s weight and you would like to predict some value based on this information. In some cases, it might be simpler to create three buckets of “overweight,” “average,” and “underweight.”

最直接的分组或分级方法是使用Bucketizer。这会将给定的连续特征分成您指定的桶。您可以指定如何通过数组或Double值列表创建存储区。这很有用，因为您可能希望简化数据集中的功能，或者稍后简化其表示以进行解释。例如，假设您有一个代表一个人体重的列，并且您希望根据此信息预测某些值。在某些情况下，创建三个“超重”，“平均”和“体重不足”的桶可能更简单。

To specify the bucket, set its borders. For example, setting splits to 5.0, 10.0, 250.0 on our `contDF` will actually fail because we don’t cover all possible input ranges. When specifying your bucket points, the values you pass into `splits`  must satisfy three requirements:

要指定桶，请设置其边界。例如，在我们的 contDF 上设置拆分为5.0,10.0,250.0实际上会失败，因为我们没有涵盖所有可能的输入范围。指定桶点时，传递给拆分的值必须满足三个要求：

- The minimum value in your splits array must be less than the minimum value in your DataFrame.

    `splits` 数组中的最小值必须小于DataFrame中的最小值。

- The maximum value in your splits array must be greater than the maximum value in your DataFrame.

    `splits` 数组中的最大值必须大于DataFrame中的最大值。

- You need to specify at a minimum three values in the splits array, which creates two buckets.

    您需要在 `splits` 数组中至少指定三个值，这会创建两个桶。

---

<center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong></font></center>
The `Bucketizer` can be confusing because we specify bucket borders via the `splits` method, but these are not actually splits.

`Bucketizer` 可能会让人感到困惑，因为我们通过 `splits` 方法指定了bucket边界，但这些实际上并不是分裂。

---

To cover all possible ranges, `scala.Double.NegativeInfinity` might be another split option, with `scala.Double.PositiveInfinity` to cover all possible ranges outside of the inner splits. In Python we specify this in the following way: `float("inf")`, `float("-inf")`.

为了覆盖所有可能的范围，`scala.Double.NegativeInfinity` 可能是另一个分裂选项，`scala.Double.PositiveInfinity` 可以覆盖内部分裂之外的所有可能范围。在 Python 中，我们通过以下方式指定它：`float(“inf”)`，`float(“-inf”)`。

In order to handle `null `or `NaN` values, we must specify the `handleInvalid` parameter as a certain value. We can either keep those values (`keep`), `error` or `null` , or skip those rows. Here’s an example of using bucketing:

为了处理 `null` 或 `NaN` 值，我们必须将 `handleInvalid` 参数指定为特定值。我们可以保留这些值（`keep`），`error` 或 `null`，或跳过这些行。以下是使用 `bucketing` 的示例： 

```scala
// in Scala
import org.apache.spark.ml.feature.Bucketizer
val bucketBorders = Array(-1.0, 5.0, 10.0, 250.0, 600.0)
val bucketer = new Bucketizer().setSplits(bucketBorders).setInputCol("id")
bucketer.transform(contDF).show()
```

```python
# in Python
from pyspark.ml.feature import Bucketizer
bucketBorders = [-1.0, 5.0, 10.0, 250.0, 600.0]
bucketer = Bucketizer().setSplits(bucketBorders).setInputCol("id")
bucketer.transform(contDF).show()
```

```text
+----+---------------------------------------+
| id |Bucketizer_4cb1be19f4179cc2545d__output|
+----+---------------------------------------+
| 0.0|                   0.0                 |
...
|10.0|                   2.0                 |
|11.0|                   2.0                 |
...
+----+---------------------------------------+
```

In addition to splitting based on hardcoded values, another option is to split based on percentiles in our data. This is done with `QuantileDiscretizer`, which will bucket the values into user-specified buckets with the splits being determined by approximate quantiles values. For instance, the 90th quantile is the point in your data at which 90% of the data is below that value. You can control how finely the buckets should be split by setting the relative error for the approximate quantiles calculation using `setRelativeError`. Spark does this is by allowing you to specify the number of buckets you would like out of the data and it will split up your data accordingly. The following is an example:

除了基于硬编码值进行分裂外，另一种选择是根据数据中的百分位数进行分裂。 这是通过 `QuantileDiscretizer` 完成的，它将值存储到用户指定的桶中，分裂由近似的分位数值确定。 例如，第90个分位数是数据中90％的数据低于该值的点。 您可以通过使用 `setRelativeError` 设置近似分位数计算的相对误差来控制分割桶的精确程度。 Spark 这样做是为了让你能够指定数据中你想要的桶数，它会相应地分割你的数据。 以下是一个例子：

```scala
// in Scala
import org.apache.spark.ml.feature.QuantileDiscretizer
val bucketer = new QuantileDiscretizer().setNumBuckets(5).setInputCol("id")
val fittedBucketer = bucketer.fit(contDF)
fittedBucketer.transform(contDF).show()
```

```python
# in Python
from pyspark.ml.feature import QuantileDiscretizer
bucketer = QuantileDiscretizer().setNumBuckets(5).setInputCol("id")
fittedBucketer = bucketer.fit(contDF)
fittedBucketer.transform(contDF).show()
```

```text
+----+----------------------------------------+
| id |quantileDiscretizer_cd87d1a1fb8e__output|
+----+----------------------------------------+
| 0.0|              0.0                       |
...
| 6.0|              1.0                       |
| 7.0|              2.0                       |
...
|14.0|              3.0                       |
|15.0|              4.0                       |
...
+----+----------------------------------------+
```

#### <font color="#3399cc">Advanced bucketing techniques</font>

The techniques descriubed here are the most common ways of bucketing data, but there are a number of other ways that exist in Spark today. All of these processes are the same from a data flow perspective: start with continuous data and place them in buckets so that they become categorical. Differences arise depending on the algorithm used to compute these buckets. The simple examples we just looked at are easy to intepret and work with, but more advanced techniques such as locality sensitivity hashing (LSH) are also available in MLlib.

这里描述的技术是最常见的数据分桶方式，但今天Spark中还有许多其他方法。从数据流的角度来看，所有这些过程都是相同的：从连续数据开始并将它们放在桶中，以便它们有分类。根据用于计算这些桶的算法而产生差异。我们刚看到的简单示例很容易解释和使用，但MLlib中也提供了更高级的技术，如局部敏感哈希（LSH）。

### <font color="#00000">Scaling and Normalization</font>

We saw how we can use bucketing to create groups out of continuous variables. Another common task is to scale and normalize continuous data. While not always necessary, doing so is usually a best practice. You might want to do this when your data contains a number of columns based on different scales. For instance, say we have a DataFrame with two columns: weight (in ounces) and height (in feet). If you don’t scale or normalize, the algorithm will be less sensitive to variations in height because height values in feet are much lower than weight values in ounces. That’s an example where you should scale your data.

我们看到了如何使用分桶从连续变量中创建组。另一个常见任务是缩放和标准化连续数据。虽然并非总是必要，但这样做通常是最佳做法。当您的数据包含基于不同比例的多个列时，您可能希望这样做。例如，假设我们有一个包含两列的DataFrame：weight（以盎司为单位）和height（以英尺为单位）。如果不进行缩放或标准化，则算法对高度变化不太敏感，因为以英尺为单位的高度值远低于以盎司为单位的重量值。这是您应该缩放数据的示例。

An example of normalization might involve transforming the data so that each point’s value is a representation of its distance from the mean of that column. Using the same example from before, we might want to know how far a given individual’s height is from the mean height. Many algorithms assume that their input data is normalized.

标准化的一个示例可能涉及转换数据，以便每个点的值表示其与该列平均值的距离。使用之前相同的示例，我们可能想知道给定个体的高度与平均高度的距离。许多算法假设他们的输入数据被标准化。

As you might imagine, there are a multitude of algorithms we can apply to our data to scale or normalize it. Enumerating them all is unnecessary here because they are covered in many other texts and machine learning libraries. If you’re unfamiliar with the concept in detail, check out any of the books referenced in the previous chapter. Just keep in mind the fundamental goal—we want our data on the same scale so that values can easily be compared to one another in a sensible way. In MLlib, this is always done on columns of type Vector. MLlib will look across all the rows in a given column (of type Vector) and then treat every dimension in those vectors as its own particular column. It will then apply the scaling or normalization function on each dimension separately. A simple example might be the following vectors in a column: 

正如您可能想象的那样，我们可以将大量算法应用于我们的数据来扩展或规范化它。在这里列举所有这些都是不必要的，因为它们包含在许多其他文本和机器学习库中。如果您不熟悉这个概念，请查看上一章中引用的任何书籍。请记住基本目标——我们希望我们的数据具有相同的比例，以便可以以合理的方式轻松地将值进行比较。在MLlib中，这总是在Vector类型的列上完成。 MLlib 将查看给定列（Vector类型）中的所有行，然后将这些向量中的每个维度视为其自己的特定列。然后，它将分别在每个维度上应用缩放或归一化功能。一个简单的例子可能是列中的以下向量：

1,2
3,4

When we apply our scaling (but not normalization) function, the “3” and the “1” will be adjusted according to those two values while the “2” and the “4” will be adjusted according to one another. This is commonly referred to as component-wise comparisons. 

当我们应用我们的缩放（但不是归一化）功能时，将根据这两个值调整“3”和“1”，而“2”和“4”将根据彼此进行调整。这通常被称为分量比较。

### <font color="#00000">StandardScaler</font>

The `StandardScaler` standardizes a set of features to have zero mean and a standard deviation of 1. The flag `withStd` will scale the data to unit standard deviation while the flag withMean (false by default) will center the data prior to scaling it.

`StandardScaler` 将一组特征标准化为零均值和标准差为1. 标志 `withStd` 将数据缩放到单位标准差，而标志`withMean`（默认为false）将在缩放之前将数据中心化。

---

<center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong></font></center>
Centering can be very expensive on sparse vectors because it generally turns them into dense vectors, so be careful before centering your data.

在稀疏向量上居中可能非常昂贵，因为它通常会将它们变成密集矢量，因此在对数据中心化之前要小心。

---

Here’s an example of using a `StandardScaler`: 

以下是使用 `StandardScaler` 的示例：

```scala
// in Scala
import org.apache.spark.ml.feature.StandardScaler
val sScaler = new StandardScaler().setInputCol("features")
sScaler.fit(scaleDF).transform(scaleDF).show()
```

```python
# in Python
from pyspark.ml.feature import StandardScaler
sScaler = StandardScaler().setInputCol("features")
sScaler.fit(scaleDF).transform(scaleDF).show()
```

The output is shown below:

```text
+---+--------------+------------------------------------------------------------+
|id |   features   |         StandardScaler_41aaa6044e7c3467adc3__output        |
+---+--------------+------------------------------------------------------------+
|0  |[1.0,0.1,-1.0]|[1.1952286093343936,0.02337622911060922,-0.5976143046671968]|
...
|1  |[3.0,10.1,3.0]|[3.5856858280031805,2.3609991401715313,1.7928429140015902]  |
+---+--------------+------------------------------------------------------------+ 
```

#### <font color="#3399cc">MinMaxScaler</font>

The `MinMaxScaler` will scale the values in a vector (component wise) to the proportional values on a scale from a given min value to a max value. If you specify the minimum value to be 0 and the maximum value to be 1, then all the values will fall in between 0 and 1: 

`MinMaxScaler` 会将向量（基于元素）中的值按照从给定最小值到最大值的比例缩放到比例值。如果将最小值指定为0并将最大值指定为1，则所有值都将介于0和1之间：

```scala
// in Scala
import org.apache.spark.ml.feature.MinMaxScaler
val minMax = new MinMaxScaler().setMin(5).setMax(10).setInputCol("features")
val fittedminMax = minMax.fit(scaleDF)
fittedminMax.transform(scaleDF).show()
```

```python
# in Python
from pyspark.ml.feature import MinMaxScaler
minMax = MinMaxScaler().setMin(5).setMax(10).setInputCol("features")
fittedminMax = minMax.fit(scaleDF)
fittedminMax.transform(scaleDF).show()
```

```text
+---+--------------+----------------------------------------------------------+
|id |   features   |           MaxAbsScaler_402587e1d9b6f268b927__output      |
+---+--------------+----------------------------------------------------------+
|0  |[1.0,0.1,-1.0]|[0.3333333333333333,0.009900990099009901,-0.3333333333333]|
... 
|1  |[3.0,10.1,3.0]|                     [1.0,1.0,1.0]                        |
+---+--------------+----------------------------------------------------------+
```

#### <font color="#3399cc">ElementwiseProduct</font>

The `ElementwiseProduct` allows us to scale each value in a vector by an arbitrary value. For example, given the vector below and the row “1, 0.1, -1” the output will be “10, 1.5, -20.” Naturally the dimensions of the scaling vector must match the dimensions of the vector inside the relevant column:

`ElementwiseProduct` 允许我们通过任意值缩放向量中的每个值。例如，给定下面的向量和行“1,0.1，-1”，输出将是“10,1.5，-20。”当然，缩放向量的尺寸必须与相关列内的向量尺寸相匹配：

 ```scala
// in Scala
import org.apache.spark.ml.feature.ElementwiseProduct
import org.apache.spark.ml.linalg.Vectors
val scaleUpVec = Vectors.dense(10.0, 15.0, 20.0)
val scalingUp = new ElementwiseProduct()
.setScalingVec(scaleUpVec)
.setInputCol("features")
scalingUp.transform(scaleDF).show()
 ```
```python
# in Python
from pyspark.ml.feature import ElementwiseProduct
from pyspark.ml.linalg import Vectors
scaleUpVec = Vectors.dense(10.0, 15.0, 20.0)
scalingUp = ElementwiseProduct()\
.setScalingVec(scaleUpVec)\
.setInputCol("features")
scalingUp.transform(scaleDF).show()
```


```text
+---+--------------+-----------------------------------------------+
| id|   features   |ElementwiseProduct_42b29ea5a55903e9fea6__output|
+---+--------------+-----------------------------------------------+
| 0 |[1.0,0.1,-1.0]|             [10.0,1.5,-20.0]                  |
...
| 1 |[3.0,10.1,3.0]|             [30.0,151.5,60.0]                 |
+---+--------------+-----------------------------------------------+
```

#### <font color="#3399cc">Normalizer</font>

The `normalizer` allows us to scale multidimensional vectors using one of several power norms, set through the parameter “p”. For example, we can use the Manhattan norm (or Manhattan distance) with p = 1, Euclidean norm with p = 2, and so on. The Manhattan distance is a measure of distance where you can only travel from point to point along the straight lines of an axis (like the streets in Manhattan).

`Normalizer`（归一化器又称：标准化）允许我们使用几个强大的范数之一来缩放多维向量，通过参数“p”设置。例如，我们可以使用曼哈顿范数（或曼哈顿距离），其中p = 1，欧几里德范数，p = 2，依此类推。曼哈顿距离是距离的度量，您只能沿着轴的直线（如曼哈顿的街道）从一个点到另一个点行进。

Here’s an example of using the `Normalizer`: 

以下是使用 `Normalizer` 的示例：

```scala
// in Scala
import org.apache.spark.ml.feature.Normalizer
val manhattanDistance = new Normalizer().setP(1).setInputCol("features")
manhattanDistance.transform(scaleDF).show()
```

```python
# in Python
from pyspark.ml.feature import Normalizer
manhattanDistance = Normalizer().setP(1).setInputCol("features")
manhattanDistance.transform(scaleDF).show()
```

```text
+---+--------------+-------------------------------+
| id|   features   |normalizer_1bf2cd17ed33__output|
+---+--------------+-------------------------------+
| 0 |[1.0,0.1,-1.0]|     [0.47619047619047...      |
| 1 | [2.0,1.1,1.0]|     [0.48780487804878...      |
| 0 |[1.0,0.1,-1.0]|     [0.47619047619047...      |
| 1 | [2.0,1.1,1.0]|     [0.48780487804878...      |
| 1 |[3.0,10.1,3.0]|     [0.18633540372670...      |
+---+--------------+-------------------------------+
```

## <font color="#9a161a">Working with Categorical Features</font>

The most common task for categorical features is indexing. Indexing converts a categorical variable in a column to a numerical one that you can plug into machine learning algorithms. While this is conceptually simple, there are some catches that are important to keep in mind so that Spark can do this in a stable and repeatable manner.

分类特征的最常见任务是索引。索引将列中的分类变量转换为可插入机器学习算法的数字变量。虽然这在概念上很简单，但仍有一些隐患事项要记住，以便 Spark 可以以稳定和可重复的方式执行此操作。

In general, we recommend re-indexing every categorical variable when pre-processing just for consistency’s sake. This can be helpful in maintaining your models over the long run as your encoding practices may change over time.

通常，我们建议在预处理时为每个分类变量重新编制索引以保持一致性。从长远来看，这有助于维护模型，因为编码实践可能会随着时间的推移而发生变化。

### <font color="#00000">StringIndexer</font>

The simplest way to index is via the StringIndexer, which maps strings to different numerical IDs. Spark’s StringIndexer also creates metadata attached to the DataFrame that specify what inputs correspond to what outputs. This allows us later to get inputs back from their respective index values: 

索引的最简单方法是通过 StringIndexer，它将字符串映射到不同的数字 ID。 Spark 的 StringIndexer 还创建附加到 DataFrame 的元数据，用于指定哪些输入对应于哪些输出。这允许我们稍后从各自的索引值获取输入：

```scala
// in Scala
import org.apache.spark.ml.feature.StringIndexer
val lblIndxr = new StringIndexer().setInputCol("lab").setOutputCol("labelInd")
val idxRes = lblIndxr.fit(simpleDF).transform(simpleDF)
idxRes.show()
```

```python
# in Python
from pyspark.ml.feature import StringIndexer
lblIndxr = StringIndexer().setInputCol("lab").setOutputCol("labelInd")
idxRes = lblIndxr.fit(simpleDF).transform(simpleDF)
idxRes.show()
```

```text
+-----+----+------+------------------+--------+
|color| lab|value1|        value2    |labelInd|
+-----+----+------+------------------+--------+
|green|good|  1   |14.386294994851129|  1.0   |
...
| red | bad|  2   |14.386294994851129|  0.0   |
+-----+----+------+------------------+--------+
```

We can also apply `StringIndexer` to columns that are not strings, in which case, they will be converted to strings before being indexed: 

我们也可以将 `StringIndexer` 应用于非字符串的列，在这种情况下，它们将在被索引之前转换为字符串：

```scala
// in Scala
val valIndexer = new StringIndexer()
.setInputCol("value1")
.setOutputCol("valueInd")
valIndexer.fit(simpleDF).transform(simpleDF).show()
```

```python
# in Python
valIndexer = StringIndexer().setInputCol("value1").setOutputCol("valueInd")
valIndexer.fit(simpleDF).transform(simpleDF).show()
```

```text
+-----+----+------+------------------+--------+
|color| lab|value1|      value2      |valueInd|
+-----+----+------+------------------+--------+
|green|good|  1   |14.386294994851129|  1.0   |
...
| red | bad|  2   |14.386294994851129|  0.0   |
+-----+----+------+------------------+--------+
```

Keep in mind that the `StringIndexer` is an estimator that must be fit on the input data. This means it must see all inputs to select a mapping of inputs to IDs. If you train a `StringIndexer` on inputs “a,” “b,” and “c” and then go to use it against input “d,” it will throw an error by default. Another option is to skip the entire row if the input value was not a value seen during training. Going along with the previous example, an input value of “d” would cause that row to be skipped entirely. We can set this option before or after training the indexer or pipeline. More options may be added to this feature in the future but as of Spark 2.2, you can only skip or throw an error on invalid inputs. 

请记住，`StringIndexer` 是一个必须适合输入数据的估计器（estimator）。这意味着它必须查看所有输入以选择输入到 ID 的映射。如果你在输入“a”，“b”和“c”上训练一个 `StringIndexer`，然后对输入“d”的背景下使用它，它默认会抛出一个错误。如果输入值不是训练期间看到的值，则另一个选项是跳过整行。沿用前面的示例，输入值“d”将导致完全跳过该行。我们可以在训练索引器或管道之前或之后设置此选项。将来可能会向此功能添加更多选项，但从Spark 2.2开始，您只能跳过或在无效输入上抛出错误。

```scala
valIndexer.setHandleInvalid("skip")
valIndexer.fit(simpleDF).setHandleInvalid("skip")
```

### <font color="#00000">Converting Indexed Values Back to Text</font>

When inspecting your machine learning results, you’re likely going to want to map back to the original values. Since MLlib classification models make predictions using the indexed values, this conversion is useful for converting model predictions (indices) back to the original categories. We can do this with `IndexToString`. You’ll notice that we do not have to input our value to the String key; Spark’s MLlib maintains this metadata for you. You can optionally specify the outputs. 

检查机器学习结果时，您可能希望映射回原始值。由于 MLlib 分类模型使用索引值进行预测，因此此转换对于将模型预测（索引）转换回原始类别非常有用。我们可以使用 `IndexToString` 来做到这一点。您会注意到我们不必将我们的值输入String键; Spark的MLlib为您维护这个元数据。您可以选择指定输出。

```scala
// in Scala
import org.apache.spark.ml.feature.IndexToString
val labelReverse = new IndexToString().setInputCol("labelInd")
labelReverse.transform(idxRes).show()
```

```python
# in Python
from pyspark.ml.feature import IndexToString
labelReverse = IndexToString().setInputCol("labelInd")
labelReverse.transform(idxRes).show()
```

```text
+-----+----+------+------------------+--------+--------------------------------+
|color| lab|value1|      value2      |labelInd|IndexToString_415...2a0d__output|
+-----+----+------+------------------+--------+--------------------------------+
|green|good|  1   |14.386294994851129|   1.0  |             good               |
...
| red | bad|  2   |14.386294994851129|   0.0  |             bad                |
+-----+----+------+------------------+--------+--------------------------------+
```

### <font color="#00000">Indexing in Vectors</font>

VectorIndexer is a helpful tool for working with categorical variables that are already found inside of vectors in your dataset. This tool will automatically find categorical features inside of your input vectors and convert them to categorical features with zero-based category indices. For example, in the following DataFrame, the first column in our Vector is a categorical variable with two different categories while the rest of the variables are continuous. By setting maxCategories to 2 in our VectorIndexer, we are instructing Spark to take any column in our vector with two or less distinct values and convert it to a categorical variable. This can be helpful when you know how many unique values there are in your largest category because you can specify this and it will automatically index the values accordingly.  Conversely, Spark changes the data based on this parameter, so if you have continuous variables that don’t appear particularly continuous (lots of repeated values) these can be unintentionally converted to categorical variables if there are too few unique values. 

`VectorIndexer` 是一个有用的工具，用于处理已在数据集中的向量中找到的分类变量。此工具将自动查找输入向量内的分类特征，并将其转换为具有从零开始的类别索引的分类特征。例如，在以下 DataFrame 中，Vector 中的第一列是具有两个不同类别的分类变量，而其余变量是连续的。通过在我们的 `VectorIndexer` 中将 `maxCategories` 设置为2，我们指示 Spark 在我们的向量中使用两个或更少不同的值并将其转换为分类变量。当您知道最大类别中有多少个唯一值时，这会很有用，因为您可以指定它，并相应地自动索引值。相反，Spark 会根据此参数更改数据，因此如果连续变量看起来不是特别连续（许多重复值），如果唯一值太少，这些变量可能会无意中转换为分类变量。

```scala
// in Scala
import org.apache.spark.ml.feature.VectorIndexer
import org.apache.spark.ml.linalg.Vectors
val idxIn = spark.createDataFrame(Seq(
(Vectors.dense(1, 2, 3),1),
(Vectors.dense(2, 5, 6),2),
(Vectors.dense(1, 8, 9),3)
)).toDF("features", "label")
val indxr = new VectorIndexer()
.setInputCol("features")
.setOutputCol("idxed")
.setMaxCategories(2)
indxr.fit(idxIn).transform(idxIn).show()
```

```python
# in Python
from pyspark.ml.feature import VectorIndexer
from pyspark.ml.linalg import Vectors
idxIn = spark.createDataFrame([
(Vectors.dense(1, 2, 3),1),
(Vectors.dense(2, 5, 6),2),
(Vectors.dense(1, 8, 9),3)
]).toDF("features", "label")
indxr = VectorIndexer()\
.setInputCol("features")\.setOutputCol("idxed")\
.setMaxCategories(2)
indxr.fit(idxIn).transform(idxIn).show()
```

```text
+-------------+-----+-------------+
|   features  |label|   idxed     |
+-------------+-----+-------------+
|[1.0,2.0,3.0]|  1  |[0.0,2.0,3.0]|
|[2.0,5.0,6.0]|  2  |[1.0,5.0,6.0]|
|[1.0,8.0,9.0]|  3  |[0.0,8.0,9.0]|
+-------------+-----+-------------+
```

### <font color="#00000">One-Hot Encoding 独热编码</font>

Indexing categorical variables is only half of the story. One-hot encoding is an extremely common data transformation performed after indexing categorical variables. This is because indexing does not always represent our categorical variables in the correct way for downstream models to process. For instance, when we index our “color” column, you will notice that some colors have a higher value (or index number) than others (in our case, blue is 1 and green is 2).

索引分类变量只是故事的一半。独热编码是在对分类变量建立索引之后执行的极其常见的数据转换。这是因为索引并不总是以下游模型处理的正确方式表示我们的分类变量。例如，当我们索引 “color” 列时，您会注意到某些颜色的值（或索引号码）高于其他颜色（在我们的例子中，蓝色为1，绿色为2）。

This is incorrect because it gives the mathematical appearance that the input to the machine learning algorithm seems to specify that green > blue, which makes no sense in the case of the current categories. To avoid this, we use `OneHotEncoder`, which will convert each distinct value to a `Boolean` flag (1 or 0) as a component in a vector. When we encode the color value, then we can see these are no longer ordered, making them easier for downstream models (e.g., a linear model) to process: 

这是不正确的，因为它套上了数学的外衣，机器学习算法的输入似乎指定绿色>蓝色，这在当前类别的情况下没有意义。为避免这种情况，我们使用 `OneHotEncoder`，它将每个不同的值转换为布尔（Boolean）标志（1或0）作为向量中的元素。当我们对颜色值进行编码时，我们可以看到它们不再有序，这使得下游模型（例如，线性模型）更容易处理：

```scala
// in Scala
import org.apache.spark.ml.feature.{StringIndexer, OneHotEncoder}
val lblIndxr = new StringIndexer().setInputCol("color").setOutputCol("colorInd")
val colorLab = lblIndxr.fit(simpleDF).transform(simpleDF.select("color"))
val ohe = new OneHotEncoder().setInputCol("colorInd")
ohe.transform(colorLab).show()
```

```python
# in Python
from pyspark.ml.feature import OneHotEncoder, StringIndexer
lblIndxr = StringIndexer().setInputCol("color").setOutputCol("colorInd")
colorLab = lblIndxr.fit(simpleDF).transform(simpleDF.select("color"))
ohe = OneHotEncoder().setInputCol("colorInd")
ohe.transform(colorLab).show()
```

```text
+-----+--------+------------------------------------------+
|color|colorInd|OneHotEncoder_46b5ad1ef147bb355612__output|
+-----+--------+------------------------------------------+
|green|  1.0   |            (2,[1],[1.0])                 |
| blue|  2.0   |            (2,[],[])                     |
...
| red |  0.0   |            (2,[0],[1.0])                 |
| red |  0.0   |            (2,[0],[1.0])                 |
+-----+--------+------------------------------------------+
```

### <font color="#00000">Text Data Transformers 文本数据转换器</font>

Text is always tricky input because it often requires lots of manipulation to map to a format that a machine learning model will be able to use effectively. There are generally two kinds of texts you’ll see: free-form text and string categorical variables. This section primarily focuses on free-form text because we already discussed categorical variables.

文本总是很棘手的输入，因为它经常需要大量的操作才能映射到机器学习模型能够有效使用的格式。您将看到通常有两种文本：自由格式文本和字符串分类变量。本节主要关注自由格式文本，因为我们已经讨论了分类变量。

### <font color="#00000">Tokenizing Text 文本符号化</font>

Tokenization is the process of converting free-form text into a list of “tokens” or individual words. The easiest way to do this is by using the `Tokenizer` class. This transformer will take a string of words, separated by whitespace, and convert them into an array of words. For example, in our dataset we might want to convert the Description field into a list of tokens. 

符号化是将自由格式文本转换为“符号”或单个单词列表的过程。最简单的方法是使用 `Tokenizer` 类。这个转换器将采用一串由空格分隔的单词，并将它们转换为单词数组。例如，在我们的数据集中，我们可能希望将Description 字段转换为标记列表。

```scala
// in Scala
import org.apache.spark.ml.feature.Tokenizer
val tkn = new Tokenizer().setInputCol("Description").setOutputCol("DescOut")
val tokenized = tkn.transform(sales.select("Description"))
tokenized.show(false)
```

```python
# in Python
from pyspark.ml.feature import Tokenizer
tkn = Tokenizer().setInputCol("Description").setOutputCol("DescOut")
tokenized = tkn.transform(sales.select("Description"))
tokenized.show(20, False)
```

```text
+-----------------------------------+------------------------------------------+
|        Description DescOut        |
+-----------------------------------+------------------------------------------+
|        RABBIT NIGHT LIGHT         |              [rabbit, night, light]      |
|        DOUGHNUT LIP GLOSS         |              [doughnut, lip, gloss]      |
...
|AIRLINE BAG VINTAGE WORLD CHAMPION | [airline, bag, vintage, world, champion] |
|AIRLINE BAG VINTAGE JET SET BROWN  | [airline, bag, vintage, jet, set, brown] |
+-----------------------------------+------------------------------------------+
```

We can also create a `Tokenizer` that is not just based white space but a regular expression with the `RegexTokenizer`. The format of the regular expression should conform to the Java Regular Expression (RegEx) syntax: 

我们还可以创建一个 `Tokenizer`，它不仅仅是基于空格，而是使用 `RegexTokenizer` 的正则表达式。正则表达式的格式应符合 Java 正则表达式（RegEx）语法：

```scala
// in Scala
import org.apache.spark.ml.feature.RegexTokenizer
val rt = new RegexTokenizer()
.setInputCol("Description")
.setOutputCol("DescOut")
.setPattern(" ") // simplest expression
.setToLowercase(true)
rt.transform(sales.select("Description")).show(false)
```

```python
# in Python
from pyspark.ml.feature import RegexTokenizer
rt = RegexTokenizer()\
.setInputCol("Description")\
.setOutputCol("DescOut")\
.setPattern(" ")\
.setToLowercase(True)
rt.transform(sales.select("Description")).show(20, False)
```

```text
+-----------------------------------+------------------------------------------+
|         Description DescOut       |
+-----------------------------------+------------------------------------------+
|         RABBIT NIGHT LIGHT        |         [rabbit, night, light]           |
|         DOUGHNUT LIP GLOSS        |         [doughnut, lip, gloss]           |
...
|AIRLINE BAG VINTAGE WORLD CHAMPION | [airline, bag, vintage, world, champion] |
|AIRLINE BAG VINTAGE JET SET BROWN  | [airline, bag, vintage, jet, set, brown] |
+-----------------------------------+------------------------------------------+
```

Another way of using the `RegexTokenizer` is to use it to output values matching the provided pattern instead of using it as a gap. We do this by setting the `gaps` parameter to `false`. Doing this with a space as a pattern returns all the spaces, which is not too useful, but if we made our pattern capture individual words, we could return those: 

使用 `RegexTokenizer` 的另一种方法是使用它来输出与提供的模式匹配的值，而不是将其用作间隔。我们通过将`gaps` 参数设置为 `false` 来完成此操作。使用空格作为模式执行此操作将返回所有空格，这不是太有用，但如果我们使模式捕获单个单词，我们可以返回这些：

```scala
// in Scala
import org.apache.spark.ml.feature.RegexTokenizer
val rt = new RegexTokenizer()
.setInputCol("Description")
.setOutputCol("DescOut")
.setPattern(" ")
.setGaps(false)
.setToLowercase(true)
rt.transform(sales.select("Description")).show(false)
```

```python
# in Python
from pyspark.ml.feature import RegexTokenizer
rt = RegexTokenizer()\
.setInputCol("Description")\
.setOutputCol("DescOut")\
.setPattern(" ")\
.setGaps(False)\
.setToLowercase(True)
rt.transform(sales.select("Description")).show(20, False)
```

```text
+-----------------------------------+------------------+
|      Description DescOut          |
+-----------------------------------+------------------+
|      RABBIT NIGHT LIGHT           |      [ , ]       |
|      DOUGHNUT LIP GLOSS           |      [ , , ]     |
...
|AIRLINE BAG VINTAGE WORLD CHAMPION |     [ , , , , ]  |
|AIRLINE BAG VINTAGE JET SET BROWN  |     [ , , , , ]  |
+-----------------------------------+------------------+
```

### <font color="#00000">Removing Common Words 移除常见词</font>

A common task after tokenization is to filter stop words, common words that are not relevant in many kinds of analysis and should thus be removed. Frequently occurring stop words in English include “the,” “and,” and “but”. Spark contains a list of default stop words you can see by calling the following method, which can be made case insensitive if necessary (as of Spark 2.2, supported languages for stopwords are “danish,” “dutch,” “english,” “finnish,” “french,” “german,” “hungarian,” “italian,” “norwegian,” “portuguese,” “russian,” “spanish,” “swedish,” and “turkish”): 

符号化（tokenization ）后的一个常见任务是过滤停用词，这些词在多种分析中不相关，因此应该被删除。英语中经常出现的停用词包括“the”，“and”和“but”。Spark包含一个默认停止词列表，您可以通过调用以下方法查看，如果需要，可以使其不区分大小写（从Spark 2.2开始 ，支持的停用词语言是“丹麦语”，“荷兰语”，“英语”，“芬兰语”，“法语”，“德语”，“匈牙利语”，“意大利语”，“挪威语”，“葡萄牙语”，“俄语”，“西班牙语”，“瑞典语 ，“和”土耳其语“）: 

```scala
// in Scala
import org.apache.spark.ml.feature.StopWordsRemover
val englishStopWords = StopWordsRemover.loadDefaultStopWords("english")
val stops = new StopWordsRemover()
.setStopWords(englishStopWords)
.setInputCol("DescOut")
stops.transform(tokenized).show()
```

```python
# in Python
from pyspark.ml.feature import StopWordsRemover
englishStopWords = StopWordsRemover.loadDefaultStopWords("english")
stops = StopWordsRemover()\
.setStopWords(englishStopWords)\
.setInputCol("DescOut")
stops.transform(tokenized).show()
```

The following output shows how this works: 

下面的输出展示了这是如何工作的：

```text
+--------------------+--------------------+------------------------------------+
|     Description    |      DescOut       |StopWordsRemover_4ab18...6ed__output|
+--------------------+--------------------+------------------------------------+
...
|SET OF 4 KNICK KN...|[set, of, 4, knic...|        [set, 4, knick, k...        |
...
+--------------------+--------------------+------------------------------------+
```

Notice how the word of is removed in the output column. That’s because it’s such a common word that it isn’t relevant to any downstream manipulation and simply adds noise to our dataset.

注意如何在输出列中删除单词。这是因为它是一个常见的词，它与任何下游操作无关，只是简单地为我们的数据集添加噪声。

### <font color="#00000">Creating Word Combinations 创建词的组合</font>

Tokenizing our strings and filtering stop words leaves us with a clean set of words to use as features. It is often of interest to look at combinations of words, usually by looking at colocated words. Word combinations are technically referred to as n-grams—that is, sequences of words of length n. An ngram of length 1 is called a unigrams; those of length 2 are called bigrams, and those of length 3 are called trigrams (anything above those are just four-gram, five-gram, etc.), Order matters with n-gram creation, so converting a sentence with three words into bigram representation would result in two bigrams. The goal when creating n-grams is to better capture sentence structure and more information than can be gleaned by simply looking at all words individually. Let’s create some n-grams to illustrate this concept.

对字符串进行符号化（Tokenizing）并过滤停用词会给我们留下一组简洁的用作特征的单词。通常通过查看共现的单词来查看单词的组合通常是有意义的。单词组合在技术上被称为 n-gram，即长度为n的单词序列。长度为1的 ngram 称为单元组（unigram）；长度为2的那些被称为二元组（bigram），而长度为3的那些被称为三元组（trigram）（任何高于那些只有 four-gram， five-gram等），顺序与 n-gram 创建有关，所以将一个带三个单词的句子转换成 bigram 代表将产生两个 bigram。创建 n-gram 时的目标是更好地捕获句子结构和更多信息，而不是通过简单地单独查看所有单词来收集信息。让我们创建一些 n-gram 来说明这个概念。

The bigrams of “Big Data Processing Made Simple” are: 

“大数据处理变得简单”的2元组是：

- “Big Data”
- “Data Processing”
- “Processing Made”
- “Made Simple”

While the trigrams are:

而三元组是：

- “Big Data Processing”
- “Data Processing Made”
- “Procesing Made Simple”

With n-grams, we can look at sequences of words that commonly co-occur and use them as inputs to a machine learning algorithm. These can create better features than simply looking at all of the words individually (say, tokenized on a space character): 

使用 n-gram，我们可以查看通常共同出现的单词序列，并将它们用作机器学习算法的输入。 这些可以创建比单独查看所有单词更好的特征（例如，在空格字符上符号化）：

```scala
// in Scala
import org.apache.spark.ml.feature.NGram
val unigram = new NGram().setInputCol("DescOut").setN(1)
val bigram = new NGram().setInputCol("DescOut").setN(2)
unigram.transform(tokenized.select("DescOut")).show(false)
bigram.transform(tokenized.select("DescOut")).show(false)
```

```python
# in Python
from pyspark.ml.feature import NGram
unigram = NGram().setInputCol("DescOut").setN(1)
bigram = NGram().setInputCol("DescOut").setN(2)
unigram.transform(tokenized.select("DescOut")).show(False)
bigram.transform(tokenized.select("DescOut")).show(False)
```

```text
+-----------------------------------------+-------------------------------------
           DescOut                        |       ngram_104c4da6a01b__output ...
+-----------------------------------------+-------------------------------------
|           [rabbit, night, light]        |       [rabbit, night, light] ...
|           [doughnut, lip, gloss]        |       [doughnut, lip, gloss] ...
...
|[airline, bag, vintage, world, champion] |[airline, bag, vintage, world, cha...
|[airline, bag, vintage, jet, set, brown] |[airline, bag, vintage, jet, set, ...
+-----------------------------------------+-------------------------------------
```

And the result for bigrams: 

二元组的结果：

```text
+------------------------------------------+------------------------------------
              DescOut                      |      ngram_6e68fb3a642a__output ...
+------------------------------------------+------------------------------------
|        [rabbit, night, light]            | [rabbit night, night light] ...
|        [doughnut, lip, gloss]            | [doughnut lip, lip gloss] ...
...
|[airline, bag, vintage, world, champion]  | [airline bag, bag vintage, vintag...
|[airline, bag, vintage, jet, set, brown]  | [airline bag, bag vintage, vintag...
+------------------------------------------+------------------------------------
```

### <font color="#00000">Converting Words into Numerical Representations 将单词转换为数字表示</font>

Once you have word features, it’s time to start counting instances of words and word combinations for use in our models. The simplest way is just to include binary counts of a word in a given document (in our case, a row). Essentially, we’re measuring whether or not each row contains a given word. This is a simple way to normalize for document sizes and occurrence counts and get numerical features that allow us to classify documents based on content. In addition, we can count words using a `CountVectorizer`, or reweigh them according to the prevalence of a given word in all the documents using a TF–IDF transformation (discussed next).

一旦你有了单词功能，就可以开始计算单词和单词组合的实例，以便在我们的模型中使用。最简单的方法是在给定文档中包含单词的二进制计数（在我们的例子中是一行）。基本上，我们测量每行是否包含给定的单词。这是一种标准化文档大小和出现次数的简单方法，并获得允许我们根据内容对文档进行分类的数值特征。此外，我们可以使用 `CountVectorizer` 对单词进行计数，或者使用 TF-IDF 转换根据所有文档中给定单词的普遍程度对它们进行重新加权（下面将讨论）。

A `CountVectorizer` operates on our tokenized data and does two things:

`CountVectorizer` 对我们的符号化数据进行操作，并做两件事：

1. During the fit process, it finds the set of words in all the documents and then counts the occurrences of those words in those documents.

    在拟合过程中，它在所有文档中找到一组单词，然后计算这些单词在这些文档中的出现次数。

2. It then counts the occurrences of a given word in each row of the DataFrame column during the transformation process and outputs a vector with the terms that occur in that row. 

    然后，它在转换过程中计算DataFrame列的每一行中给定单词的出现次数，并输出带有该行中出现的词语（term）的向量。

Conceptually this tranformer treats every row as a document and every word as a term and the total collection of all terms as the vocabulary. These are all tunable parameters, meaning we can set the minimum term frequency (minTF) for the term to be included in the vocabulary (effectively removing rare words from the vocabulary); minimum number of documents a term must appear in (minDF) before being included in the vocabulary (another way to remove rare words from the vocabulary); and finally, the total maximum vocabulary size (vocabSize). Lastly, by default the `CountVectorizer` will output the counts of a term in a document. To just return whether or not a word exists in a document, we can use setBinary(true). Here’s an example of using CountVectorizer: 

从概念上讲，这个转换器将每一行视为一个文档，将每个单词（word）视为一个术语（term），并将所有术语的总集合视为词汇（vocabulary）。这些都是可调参数，这意味着我们可以设置词汇中包含的术语的最小术语频率（minTF）（有效地从词汇表中删除稀有词）; 术语在被包含在词汇表中之前必须出现的次数满足（minDF）中的最小文档数量（从词汇表中删除稀有词汇的另一种方式）；最后，总的最大词汇量大小（vocabSize）。最后，默认情况下，`CountVectorizer` 将输出文档中术语的计数。要返回文档中是否存在单词，我们可以使用setBinary（true）。以下是使用` CountVectorizer` 的示例：

```scala
// in Scala
import org.apache.spark.ml.feature.CountVectorizer
val cv = new CountVectorizer()
.setInputCol("DescOut")
.setOutputCol("countVec")
.setVocabSize(500)
.setMinTF(1)
.setMinDF(2)
val fittedCV = cv.fit(tokenized)
fittedCV.transform(tokenized).show(false)
```

```python
# in Python
from pyspark.ml.feature import CountVectorizer
cv = CountVectorizer()\.setInputCol("DescOut")\
.setOutputCol("countVec")\
.setVocabSize(500)\
.setMinTF(1)\
.setMinDF(2)
fittedCV = cv.fit(tokenized)
fittedCV.transform(tokenized).show(False)
```

While the output looks a little complicated, it’s actually just a sparse vector that contains the total vocabulary size, the index of the word in the vocabulary, and then the counts of that particular word: 

虽然输出看起来有点复杂，但它实际上只是一个稀疏向量，它包含总词汇量大小，词汇表中单词的索引，然后是特定单词的计数：

```text
+---------------------------------+--------------------------------------------+
             DescOut              |                 countVec                   |
+---------------------------------+--------------------------------------------+
|       [rabbit, night, light]    |         (500,[150,185,212],[1.0,1.0,1.0])  |
|       [doughnut, lip, gloss]    |         (500,[462,463,492],[1.0,1.0,1.0])  |
...
|[airline, bag, vintage, world,...|         (500,[2,6,328],[1.0,1.0,1.0])      |
|[airline, bag, vintage, jet, s...|(500,[0,2,6,328,405],[1.0,1.0,1.0,1.0,1.0]) |
+---------------------------------+--------------------------------------------+
```

#### <font color="#3399cc">Term frequency–inverse document frequency 词频—逆文档频率</font>

Another way to approach the problem of converting text into a numerical representation is to use term frequency–inverse document frequency (TF–IDF). In simplest terms, TF–IDF measures how often a word occurs in each document, weighted according to how many documents that word occurs in. The result is that words that occur in a few documents are given more weight than words that occur in many documents. In practice, a word like “the” would be weighted very low because of its prevalence while a more specialized word like “streaming” would occur in fewer documents and thus would be weighted higher. In a way, TF–IDF helps find documents that share similar topics.

解决将文本转换为数字表示的问题的另一种方法是使用词频 - 逆文档频率（TF-IDF）。简单来说，TF-IDF测量每个文档中单词出现的频率，根据单词出现的文档数加权。结果是，少数文档中出现的单词比许多文档中出现的单词更重要。 在实践中，像“the”这样的单词由于其普遍性而被加权得非常低，而像“streaming”这样的更专业的单词将在更少的文档中出现，因此将被加权更高。在某种程度上，TF-IDF有助于查找共享相似主题的文档。

Let’s take a look at an example—first, we’ll inspect some of the documents in our data containing the word “red”: 

让我们看一个例子——首先，我们将检查包含单词“red”的数据中的一些文档：

```scala
// in Scala
val tfIdfIn = tokenized
.where("array_contains(DescOut, 'red')")
.select("DescOut")
.limit(10)
tfIdfIn.show(false)
```

```python
# in Python
tfIdfIn = tokenized\
.where("array_contains(DescOut, 'red')")\
.select("DescOut")\
.limit(10)
tfIdfIn.show(10, False)
```

```text
+---------------------------------------+
               DescOut                  |
+---------------------------------------+
|[gingham, heart, , doorstop, red]      |
...
|[red, retrospot, oven, glove]          |
|[red, retrospot, plate]                |
+---------------------------------------+
```

We can see some overlapping words in these documents, but these words provide at least a rough topic-like representation. Now let’s input that into TF–IDF. To do this, we’re going to hash each word and convert it to a numerical representation, and then weigh each word in the voculary according to the inverse document frequency. Hashing is a similar process as CountVectorizer, but is irreversible—that is, from our output index for a word, we cannot get our input word (multiple words might map to the same output index): 

我们可以在这些文档中看到一些重叠的单词，但这些单词至少提供了一个粗略的主题表示。现在让我们输入TF-IDF。为此，我们将对每个单词进行哈希并将其转换为数字表示，然后根据逆文档频率对单词中的每个单词进行加权。哈希是与 `CountVectorizer` 类似的过程，但是不可逆转——也就是说，从单词的输出索引，我们无法得到输入词（多个单词可能映射到相同的输出索引）：

```scala
// in Scala
import org.apache.spark.ml.feature.{HashingTF, IDF}
val tf = new HashingTF()
.setInputCol("DescOut")
.setOutputCol("TFOut")
.setNumFeatures(10000)
val idf = new IDF()
.setInputCol("TFOut")
.setOutputCol("IDFOut")
.setMinDocFreq(2)
```

```python
# in Python
from pyspark.ml.feature import HashingTF, IDF
tf = HashingTF()\
.setInputCol("DescOut")\
.setOutputCol("TFOut")\
.setNumFeatures(10000)
idf = IDF()\
.setInputCol("TFOut")\
.setOutputCol("IDFOut")\
.setMinDocFreq(2)
```

```scala
// in Scala
idf.fit(tf.transform(tfIdfIn)).transform(tf.transform(tfIdfIn)).show(false)
```

```python
# in Python
idf.fit(tf.transform(tfIdfIn)).transform(tf.transform(tfIdfIn)).show(10, False)
```

While the output is too large to include here, notice that a certain value is assigned to “red” and that this value appears in every document. Also note that this term is weighted extremely low because it appears in every document. The output format is a sparse Vector we can subsequently input into a machine learning model in a form like this:

虽然输出太大而不能包含在此处，但请注意某个值被指定为 “red”，并且该值出现在每个文档中。另请注意，此术语（term）的权重极低，因为它出现在每个文档中。输出格式是一个稀疏的 Vector，我们可以随后以这样的形式输入到机器学习模型中：

```text
(10000,[2591,4291,4456],[1.0116009116784799,0.0,0.0])
```

This vector is represented using three different values: the total vocabulary size, the hash of every word appearing in the document, and the weighting of each of those terms. This is similar to the `CountVectorizer` output. 

该向量使用三个不同的值表示：总词汇量大小，文档中出现的每个单词的哈希值，以及每个术语的权重。这类似于 `CountVectorizer` 输出。

### <font color="#00000">Word2Vec</font>

Word2Vec is a deep learning–based tool for computing a vector representation of a set of words. The goal is to have similar words close to one another in this vector space, so we can then make generalizations about the words themselves. This model is easy to train and use, and has been shown to be useful in a number of natural language processing applications, including entity recognition, disambiguation, parsing, tagging, and machine translation.

Word2Vec 是一种基于深度学习的工具，用于计算一组单词的向量表示。目标是在这个向量空间中使相似的单词彼此接近，这样我们就可以对单词本身进行概括。该模型易于训练和使用，并且已被证明在许多自然语言处理应用中是有用的，包括实体识别，消歧，解析，标记和机器翻译。

Word2Vec is notable for capturing relationships between words based on their semantics. For example, if v~king, v~queen, v~man, and v~women represent the vectors for those four words, then we will often get a representation where v~king - v~man + v~woman ~= v~queen. To do this, Word2Vec uses a technique called “skip-grams” to convert a sentence of words into a vector representation (optionally of a specific size). It does this by building a vocabulary, and then for every sentence, it removes a token and trains the model to predict the missing token in the "n-gram” representation. Word2Vec works best with continuous, free-form text in the form of tokens.

Word2Vec以基于语义捕获单词之间的关系而着称。例如，如果v~king，v~ques，v~man和v~woman代表这四个单词的向量，那么我们经常会得到一个表示 v~king  -  v~man + v~woman~ = v 〜女王。为此，Word2Vec使用一种名为“skip-gram”的技术将单词的句子转换为向量表示（可选地具有特定大小）。它通过构建词汇表来实现这一点，然后对于每个句子，它会删除一个子并训练模型以预测“n-gram”表示中的丢失子. Word2Vec 最适用于连续的自由格式文本，其形式为：子。

Here’s a simple example from the documentation: 

以下是文档中的一个简单示例：

```scala
// in Scala
import org.apache.spark.ml.feature.Word2Vec
import org.apache.spark.ml.linalg.Vector
import org.apache.spark.sql.Row

// Input data: Each row is a bag of words from a sentence or document.
val documentDF = spark.createDataFrame(Seq(
	"Hi I heard about Spark".split(" "),
	"I wish Java could use case classes".split(" "),
	"Logistic regression models are neat".split(" ")
).map(Tuple1.apply)).toDF("text")

// Learn a mapping from words to Vectors.
val word2Vec = new Word2Vec().
setInputCol("text").
setOutputCol("result").
setVectorSize(3).
setMinCount(0)

val model = word2Vec.fit(documentDF)
val result = model.transform(documentDF)
result.collect().foreach { 
    case Row(text: Seq[_], features: Vector) =>	
	println(s"Text: [${text.mkString(", ")}] => \nVector: $features\n")
}
```

```python
 #in Python
from pyspark.ml.feature import Word2Vec
# Input data: Each row is a bag of words from a sentence or document.
documentDF = spark.createDataFrame(
    [("Hi I heard about Spark".split(" "), ),	
	 ("I wish Java could use case classes".split(" "), ),
	 ("Logistic regression models are neat".split(" "), )
	], 
    ["text"]
)
# Learn a mapping from words to Vectors.word2Vec = Word2Vec(vectorSize=3, minCount=0, inputCol="text",
outputCol="result")
model = word2Vec.fit(documentDF)
result = model.transform(documentDF)
for row in result.collect():
	text, vector = row
	print("Text: [%s] => \nVector: %s\n" % (", ".join(text), str(vector)))
```

```
Text: [Hi, I, heard, about, Spark] =>
Vector: [-0.008142343163490296,0.02051363289356232,0.03255096450448036]
Text: [I, wish, Java, could, use, case, classes] =>
Vector: [0.043090314205203734,0.035048123182994974,0.023512658663094044]
Text: [Logistic, regression, models, are, neat] =>
Vector: [0.038572299480438235,-0.03250147425569594,-0.01552378609776497]
```

Spark’s Word2Vec implementation includes a variety of tuning parameters that can be found in [the documentation](http://spark.apache.org/docs/latest/mllib-feature-extraction.html#word2vec).

Spark 的 Word2Vec 实现包括各种调整参数，这可以在[文档](http://spark.apache.org/docs/latest/mllib-feature-extraction.html#word2vec)中找到。

## <font color="#9a161a">Feature Manipulation特征操作</font>

While nearly every transformer in ML manipulates the feature space in some way, the following algorithms and tools are automated means of either expanding the input feature vectors or reducing them to a lower number of dimensions.

虽然 ML 中的几乎每个转换器都以某种方式操纵特征空间，但以下算法和工具是扩展输入特征向量或将它们减少到较低维数的自动化方法。

### <font color="#00000">PCA 主成分分析</font>

Principal Components Analysis (PCA) is a mathematical technique for finding the most important aspects of our data (the principal components). It changes the feature representation of our data by creating a new set of features (“aspects”). Each new feature is a combination of the original features. The power of PCA is that it can create a smaller set of more meaningful features to be input into your model, at the potential cost of interpretability.

主成分分析（PCA）是一种用于查找数据最重要层面（主要成分）的数学技术。它通过创建一组新特征（“方面”）来更改数据的特征表示。每个新特征都是原始特征的组合。PCA的强大之处在于它可以创建一组更小的更有意义的特征，以便以可解释的潜在成本输入到您的模型中。

You’d want to use PCA if you have a large input dataset and want to reduce the total number of features you have. This frequently comes up in text analysis where the entire feature space is massive and many of the features are largely irrelevant. Using PCA, we can find the most important combinations of features and only include those in our machine learning model. PCA takes a parameter , specifying the number of output features to create. Generally, this should be much smaller than your input vectors’ dimension.

如果您有大量输入数据集并希望减少所拥有的特征总数，则需要使用 PCA。这经常出现在文本分析中，其中整个特征空间是巨大的，并且许多特征在很大程度上是无关紧要的。使用 PCA，我们可以找到最重要的特征组合，并且只包括我们的机器学习模型中的特征组合。 PCA 接受一个参数，指定要创建的输出要素的数量。通常，这应该比输入向量的维度小得多。

---

<center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong></font></center>
Picking the right is nontrivial and there’s no prescription we can give. Check out the relevant chapters in [ESL](https://web.stanford.edu/~hastie/ElemStatLearn//) and [ISL](http://faculty.marshall.usc.edu/gareth-james/) for more information.

挑选对的特征是非常重要的，我们无法给予处方。有关更多信息，请查看 [ESL](https://web.stanford.edu/~hastie/ElemStatLearn//) 和 [ISL](http://faculty.marshall.usc.edu/gareth-james/) 中的相关章节。

---

Let’s train PCA with a of 2: 

让我们训练 PCA 的 2：

```scala
// in Scala
import org.apache.spark.ml.feature.PCA
val pca = new PCA().setInputCol("features").setK(2)
pca.fit(scaleDF).transform(scaleDF).show(false)
```

```python
# in Python
from pyspark.ml.feature import PCA
pca = PCA().setInputCol("features").setK(2)
pca.fit(scaleDF).transform(scaleDF).show(20, False)
```

```text
+---+--------------+------------------------------------------+
|id |     features |           pca_7c5c4aa7674e__output       |
+---+--------------+------------------------------------------+
|0  |[1.0,0.1,-1.0]|[0.0713719499248418,-0.4526654888147822]  |
...
|1  |[3.0,10.1,3.0]|[-10.872398139848944,0.030962697060150646]|
+---+--------------+------------------------------------------+
```

### <font color="#00000">Interaction 相互作用</font>

In some cases, you might have domain knowledge about specific variables in your dataset. For example, you might know that a certain interaction between the two variables is an important variable to include in a downstream estimator. The feature transformer `Interaction` allows you to create an interaction between two variables manually. It just multiplies the two features together—something that a typical linear model would not do for every possible pair of features in your data. This transformer is currently only available directly in Scala but can be called from any language using the RFormula. We recommend users just use `RFormula` instead of manually creating interactions.

在某些情况下，您可能拥有有关数据集中特定变量的领域知识。例如，您可能知道两个变量之间的某种相互作用是包含在下游估算器中的重要变量。特征转换器 `Interaction` 允许您手动创建两个变量之间的交互。它只是将两个特征相乘——这是典型的线性模型不能为数据中的每个可能的特征对做的事情。此转换器目前只能在 Scala 中直接使用，但可以使用 `RFormula` 从任何语言调用。我们推荐用户只使用 `RFormula` 而不是手动创建交互。

### <font color="#00000">Polynomial Expansion 多项式扩展</font>

Polynomial expansion is used to generate interaction variables of all the input columns. With polynomial expansion, we specify to what degree we would like to see various interactions. For example, for a degree-2 polynomial, Spark takes every value in our feature vector, multiplies it by every other value in the feature vector, and then stores the results as features. For instance, if we have two input features, we’ll get four output features if we use a second degree polynomial (2x2). If we have three input features, we’ll get nine output features (3x3). If we use a third-degree polynomial, we’ll get 27 output features (3x3x3) and so on. This transformation is useful when you want to see interactions between particular features but aren’t necessarily sure about which interactions to consider.

多项式展开用于生成所有输入列的交互变量。通过多项式展开，我们指定了我们希望看到各种交互的维度。例如，对于维度为2的多项式，Spark 会获取特征向量中的每个值，将其乘以特征向量中的每个其他值，然后将结果存储为特征。例如，如果我们有两个输入特征，如果我们使用二次多项式（2x2），我们将得到四个输出特征。如果我们有三个输入特征，我们将获得九个输出特征（3x3）。如果我们使用三次多项式，我们将获得27个输出特征（3x3x3），依此类推。当您想要查看特定之间的交互但不一定确定要考虑哪些交互时，此转换很有用。

---

<center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong></font></center>
Polynomial expansion can greatly increase your feature space, leading to both high computational costs and overfitting. Use it with caution, especially for higher degrees.

多项式扩展可以极大地增加您的特征空间，从而导致高计算成本和过度拟合。请谨慎使用，特别是对于更高的度数。

---

Here’s an example of a second degree polynomial: 

这是二次多项式的一个例子：

```scala
// in Scala
import org.apache.spark.ml.feature.PolynomialExpansion
val pe = new PolynomialExpansion().setInputCol("features").setDegree(2)
pe.transform(scaleDF).show(false)
```

```python
# in Python
from pyspark.ml.feature import PolynomialExpansion
pe = PolynomialExpansion().setInputCol("features").setDegree(2)
pe.transform(scaleDF).show()
```

```text
+---+--------------+-----------------------------------------------------------+
|id |   features   |                 poly_9b2e603812cb__output                 |
+---+--------------+-----------------------------------------------------------+
| 0 |[1.0,0.1,-1.0]|[1.0,1.0,0.1,0.1,0.010000000000000002,-1.0,-1.0,-0.1,1.0]  |
...
| 1 |[3.0,10.1,3.0]|[3.0,9.0,10.1,30.299999999999997,102.00999999999999,3.0... |
+---+--------------+-----------------------------------------------------------+
```

## <font color="#9a161a">Feature Selection</font>

Often, you will have a large range of possible features and want to select a smaller subset to use for training. For example, many features might be correlated, or using too many features might lead to overfitting. This process is called feature selection. There are a number of ways to evaluate feature importance once you’ve trained a model but another option is to do some rough filtering beforehand. Spark has some simple options for doing that, such as `ChiSqSelector`.

通常，您将拥有大量可能的特征，并希望选择较小的子集用于训练。例如，许多特征可能是相关的，或者使用太多特征可能会导致过拟合。此过程称为特征选择。一旦您训练了模型，有很多方法可以评估特征重要性，但另一种方法是事先进行粗略过滤。 Spark有一些简单的选项，比如 `ChiSqSelector` （卡方选择器）。

### <font color="#00000">ChiSqSelector 卡方选择器</font>

`ChiSqSelector` leverages a statistical test to identify features that are not independent from the label we are trying to predict, and drop the uncorrelated features. It’s often used with categorical data in order to reduce the number of features you will input into your model, as well as to reduce the dimensionality of text data (in the form of frequencies or counts). Since this method is based on the Chi-Square test, there are several different ways we can pick the “best” features. The methods are `numTopFeatures`, which is ordered by <font face="constant-width" color="#000000" size=3>p-value</font>; <font face="constant-width" color="#000000" size=3>percentile</font>, which takes a proportion of the input features (instead of just the top N features); and <font face="constant-width" color="#000000" size=3>fpr</font>, which sets a cut off <font face="constant-width" color="#000000" size=3>p-value</font>.

`ChiSqSelector` 利用统计测试来识别与我们试图预测的标签无关的特征，并删除不相关的特征。它通常与分类数据一起使用，以减少您将输入到模型中的特征数量，以及减少文本数据的维度（以频率或计数的形式）。由于此方法基于卡方检验，因此有几种不同的方法可以选择“最佳”特征。方法是 `numTopFeatures`，按p值排序；百分位数，它占用一部分输入特征（而不仅仅是前N个特征）; 和 <font face="constant-width" color="#000000" size=3>fpr</font>，它设置了一个截止的p值。

We will demonstrate this with the output of the `CountVectorizer` created earlier in this chapter: 

我们将使用本章前面创建的 `CountVectorizer` 的输出来演示这一点：

```scala
// in Scala
import org.apache.spark.ml.feature.{ChiSqSelector, Tokenizer}
val tkn = new Tokenizer().setInputCol("Description").setOutputCol("DescOut")
val tokenized = tkn.
transform(sales.select("Description", "CustomerId")).
where("CustomerId IS NOT NULL")
val prechi = fittedCV.transform(tokenized)
val chisq = new ChiSqSelector().
setFeaturesCol("countVec").
setLabelCol("CustomerId").
setNumTopFeatures(2)
chisq.fit(prechi).transform(prechi)
.drop("customerId", "Description", "DescOut").show()
```

```python
# in Python
from pyspark.ml.feature import ChiSqSelector, Tokenizer
tkn = Tokenizer().setInputCol("Description").setOutputCol("DescOut")
tokenized = tkn\
.transform(sales.select("Description", "CustomerId"))\
.where("CustomerId IS NOT NULL")
prechi = fittedCV.transform(tokenized)\
.where("CustomerId IS NOT NULL")
chisq = ChiSqSelector()\
.setFeaturesCol("countVec")\
.setLabelCol("CustomerId")\
.setNumTopFeatures(2)
chisq.fit(prechi).transform(prechi)\
.drop("customerId", "Description", "DescOut").show()
```

## <font color="#9a161a">Advanced Topics 高级主题</font>

There are several advanced topics surrounding transformers and estimators. Here we touch on the two most common, persisting transformers as well as writing custom ones. Persisting Transformers Once you’ve used an estimator to configure a transformer, it can be helpful to write it to disk and simply load it when necessary (e.g., for use in another Spark session). We saw this in the previous chapter when we persisted an entire pipeline. To persist a transformer individually, we use the write method on the fitted transformer (or the standard transformer) and specify the location : 

围绕转换器和估计器有几个高级主题。 在这里，我们触及两个最常见的，持久化的转换器以及编写自定义的转换器。 持久化的转换器一旦使用了估计器来配置转换器，将其写入磁盘并在必要时简单地加载（例如，用于另一个Spark会话）会很有帮助。 我们在上一章中看到了这一点，当时我们持久化整个管道。 为了单独持久化转换器，我们在已经拟合的转换器（或标准转换器）上使用写入方法并指定位置：

```scala
// in Scala
val fittedPCA = pca.fit(scaleDF)
fittedPCA.write.overwrite().save("/tmp/fittedPCA")
```

```python
# in Python
fittedPCA = pca.fit(scaleDF)
fittedPCA.write().overwrite().save("/tmp/fittedPCA")
```

We can then load it back in:

我们可以加载回来：

```scala
// in Scala
import org.apache.spark.ml.feature.PCAModel
val loadedPCA = PCAModel.load("/tmp/fittedPCA")
loadedPCA.transform(scaleDF).show()
```

```python
# in Python
from pyspark.ml.feature import PCAModel
loadedPCA = PCAModel.load("/tmp/fittedPCA")
loadedPCA.transform(scaleDF).show() 
```

## <font color="#9a161a">Writing a Custom Transformer 编写自定义转换器</font>

Writing a custom transformer can be valuable when you want to encode some of your own business logic in a form that you can fit into an ML Pipeline, pass on to hyperparameter search, and so on. In general you should try to use the built-in modules (e.g., SQLTransformer) as much as possible because they are optimized to run efficiently. But sometimes we do not have that luxury. Let’s create a simple tokenizer to demonstrate: 

当您想要以适合ML管道的形式编码自己的一些业务逻辑，传递给超参数搜索等时，编写自定义转换器可能很有价值。 通常，您应该尝试尽可能多地使用内置模块（例如，SQLTransformer），因为它们经过优化可以高效运行。 但有时我们没有那么奢侈。 让我们创建一个简单的符号化器来演示：

```scala
import org.apache.spark.ml.UnaryTransformer
import org.apache.spark.ml.util.{DefaultParamsReadable, DefaultParamsWritable,
Identifiable}
import org.apache.spark.sql.types.{ArrayType, StringType, DataType}
import org.apache.spark.ml.param.{IntParam, ParamValidators}

class MyTokenizer(override val uid: String) extends UnaryTransformer[String, Seq[String], MyTokenizer] with DefaultParamsWritable {
	def this() = this(Identifiable.randomUID("myTokenizer"))
	
    val maxWords: IntParam = new IntParam(this, "maxWords",
	"The max number of words to return.",
	ParamValidators.gtEq(0))

    def setMaxWords(value: Int): this.type = set(maxWords, value)
	def getMaxWords: Integer = $(maxWords)
	override protected def createTransformFunc: String => Seq[String] = (
		inputString: String) => {
		inputString.split("\\s").take($(maxWords))
	} 

	override protected def validateInputType(inputType: DataType): Unit = {
		require(inputType == StringType, s"Bad input type: $inputType. Requires 	String.")
	} 

	override protected def outputDataType: DataType = new ArrayType(StringType,true)
} 

// this will allow you to read it back in by using this object.object MyTokenizer extends DefaultParamsReadable[MyTokenizer]
val myT = new MyTokenizer().setInputCol("someCol").setMaxWords(2)
myT.transform(Seq("hello world. This text won't show.").toDF("someCol")).show()
```

It is also possible to write a custom estimator where you must customize the transformation based on the actual input data. However, this isn’t as common as writing a standalone transformer and is therefore not included in this book. A good way to do this is to look at one of the simple estimators we saw before and modify the code to suit your use case. A good place to start might be the StandardScaler.

也可以编写自定义估计器，您必须根据实际输入数据自定义转换。但是，这并不像编写独立转换器那么常见，因此不包含在本书中。这样做的一个好方法是查看我们之前看到的一个简单估计器并修改代码以适合您的用例。一个好的起点可能是StandardScaler。

## <font color="#9a161a">Conclusion 结论</font>

This chapter gave a whirlwind tour of many of the most common preprocessing transformations Spark has available. There are several domain-specific ones we did not have enough room to cover (e.g., Discrete Cosine Transform), but you can find more information in [the documentation](http://spark.apache.org/docs/latest/ml-features.html). This area of Spark is also constantly growing as the community develops new ones.

本章对Spark提供的许多最常见的预处理转换进行了快速之旅。有几个特定于某些领域的，我们没有足够的空间来覆盖（例如，离散余弦变换），但您可以在[文档](http://spark.apache.org/docs/latest/ml-features.html)中找到更多信息。随着社区发展新领域，Spark的这一领域也在不断发展。

Another important aspect of this feature engineering toolkit is consistency. In the previous chapter we covered the pipeline concept, an essential tool to package and train end-to-end ML workflows. In the next chapter we will start going through the variety of machine learning tasks you may have and what algorithms are available for each one. 

此特征工程工具包的另一个重要方面是一致性。在上一章中，我们介绍了管道概念，它是打包和训练端到端ML工作流程的重要工具。在下一章中，我们将开始介绍您可能拥有的各种机器学习任务以及每种机器可用的算法。
