---
title: 翻译 Chapter 9 Data Sources
date: 2019-10-20
copyright: true
categories: English
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 9 Data Sources 数据源
<center><strong>译者</strong>：<u><a style="color:#0879e3" href="https://snaildove.github.io">https://snaildove.github.io</a></u></center>

This chapter formally introduces the variety of other data sources that you can use with Spark out of the box as well as the countless other sources built by the greater community. Spark has six “core” data sources and hundreds of external data sources written by the community. The ability to read and write from all different kinds of data sources and for the community to create its own contributions is arguably one of Spark’s greatest strengths. Following are Spark’s core data sources:

本章正式介绍了可与Spark一起开箱即用的各种其他数据源，以及更大的社区构建的无数其他数据源。Spark有六个“核心”数据源和社区编写的数百个外部数据源。从所有不同类型的数据源读取和写入数据以及使社区自行做出贡献的能力可以说是Spark的最大优势之一。以下是Spark的核心数据源：

- CSV
- JSON
- Parquet
- ORC
- JDBC/ODBC connections
- Plain-text files 纯文本文件

As mentioned, Spark has numerous community-created data sources. Here’s just a small sample: 

如前所述，Spark具有大量社区创建的数据源。这只是一个小样本：

- <u><a href="https://spark-packages.org/package/datastax/spark-cassandra-connector" style="color:#0879e3">Cassandra</a></u>
- <u><a href="https://bit.ly/2FkKN5A" style="color:#0879e3">HBase</a></u>
- <u><a href="https://bit.ly/2FkKN5A" style="color:#0879e3">MongoDB</a></u>
- <u><a href="https://bit.ly/2GlMsJE" style="color:#0879e3">AWS Redshift</a></u>
- <u><a href="https://bit.ly/2GitGCK" style="color:#0879e3">XML</a></u>
- And many, many others

The goal of this chapter is to give you the ability to read and write from Spark’s core data sources and know enough to understand what you should look for when integrating with third-party data sources. To achieve this, we will focus on the core concepts that you need to be able to recognize and understand.

本章的目的是使您能够从Spark的核心数据源进行读写，并且足够了解与第三方数据源集成时应寻找的内容。 为此，我们将重点关注您需要能够识别和理解的核心概念。

## <font color="#9a161a">The Structure of the Data Sources API 数据源API的结构</font>

Before proceeding with how to read and write from certain formats, let’s visit the overall organizational structure of the data source APIs.

在继续进行某些格式的读取和写入之前，让我们先访问数据源API的总体组织结构。

### <font color="#00000">Read API Structure 读取数据的API的结构</font>

The core structure for reading data is as follows: 

```scala
DataFrameReader.format(...).option("key", "value").schema(...).load()
```

We will use this format to read from all of our data sources. format is optional because by default Spark will use the Parquet format. option allows you to set key-value configurations to parameterize how you will read data. Lastly, schema is optional if the data source provides a schema or if you intend to use schema inference. Naturally, there are some required options for each format, which we will discuss when we look at each format.

我们将使用这种格式来读取所有数据源。格式是可选的，因为默认情况下，Spark将使用Parquet格式。选项允许您设置键值配置，以参数化如何读取数据。最后，如果数据源提供了模式，或者您打算使用模式推断，则模式是可选的。自然，每种格式都有一些必需的选项，我们将在讨论每种格式时进行讨论。

---

<p><center><font face="constant-width" color="#737373" size=3><strong>NOTE 注意</strong></font></center></P>
There is a lot of shorthand notation in the Spark community, and the data source read API is no exception. We try to be
consistent throughout the book while still revealing some of the shorthand notation along the way. 

Spark社区中有很多速记符号，并且数据源读取API也不例外。我们试图在整本书中保持一致，同时仍然沿途揭示一些速记符号。

----

### <font color="#00000">Basics of Reading Data 读取数据的基础要素</font>

The foundation for reading data in Spark is the `DataFrameReader`. We access this through the `SparkSession` via the read attribute:

在Spark中读取数据的基础是 `DataFrameReader`。我们通过 `SparkSession ` 的 `read` 属性来使用这个：

```scala
spark.read
```

After we have a DataFrame reader, we specify several values: 

有了DataFrame读取器后，我们指定几个值：

- The format
- The schema
- The read mode
- A series of options 

The format, options, and schema each return a `DataFrameReader` that can undergo further transformations and are all optional, except for one option. Each data source has a specific set of options that determine how the data is read into Spark (we cover these options shortly). At a minimum, you must supply the `DataFrameReader` a path to from which to read.

格式，选项和模式每个都返回一个`DataFrameReader`，该对象可以进行进一步的转换，并且都是可选的，除了一个选项。每个数据源都有一组特定的选项，这些选项决定了如何将数据读入Spark（稍后将介绍这些选项）。至少必须为`DataFrameReader`提供读取的路径。

Here’s an example of the overall layout:

这是整体布局的示例： 

```scala
spark.read.format("csv")
.option("mode", "FAILFAST")
.option("inferSchema", "true")
.option("path", "path/to/file(s)")
.schema(someSchema)
.load()
```

There are a variety of ways in which you can set options; for example, you can build a map and pass in your configurations. For now, we’ll stick to the simple and explicit way that you just saw.

您可以通过多种方式设置选项。例如，您可以构建映射并传递配置。目前，我们将继续使用您刚才看到的简单明了的方式。

#### <font color="#3399cc">Read modes 读取模式</font>

Reading data from an external source naturally entails encountering malformed data, especially when working with only semi-structured data sources. Read modes specify what will happen when Spark does come across malformed records. Table 9-1 lists the read modes.

从外部源读取数据自然会遇到格式错误的数据，尤其是在仅使用半结构化数据源时。读取方式指定当Spark遇到格式错误的记录时将发生的情况。表9-1列出了读取方式。

Table 9-1. Spark’s read modes

| Read mode       | Description                                                  |
| --------------- | ------------------------------------------------------------ |
| `permissive`    | Sets all fields to `null` when it encounters a corrupted record and places all corrupted records in a string column `called _corrupt_record`<br />遇到损坏的记录并将所有损坏的记录放在 `called _corrupt_record` 的字符串列中时，将所有字段设置为 `null` 。 |
| `dropMalformed` | Drops the row that contains malformed records<br />删除包含格式错误的记录的行。 |
| `failFast`      | Fails immediately upon encountering malformed records<br />遇到格式错误的记录后立即失败。 |

The default is `permissive`. 

默认是 `permissive`。

### <font color="#00000"> Write API Structure 写入数据的API的结构</font>

The core structure for writing data is as follows: 

写入数据的核心结构如下：

```scala
DataFrameWriter.format(...).option(...).partitionBy(...).bucketBy(...).sortBy(
...).save()
```

We will use this format to write to all of our data sources. `format` is optional because by default, Spark will use the Parquet format. option, again, allows us to configure how to write out our given data. `PartitionBy`, `bucketBy`, and `sortBy `work only for file-based data sources; you can use them to control the specific layout of files at the destination.

### <font color="#00000">Basics of Writing Data 写入数据的基础要素</font>

The foundation for writing data is quite similar to that of reading data. Instead of the `DataFrameReader`, we have the `DataFrameWriter`. Because we always need to write out some given data source, we access the `DataFrameWriter `on a per-DataFrame basis via the write attribute:

写入数据的基础要素与读取数据的基础非常相似。代替了`DataFrameReader`，我们有了`DataFrameWriter`。因为我们总是需要写出某些给定的数据源，所以我们通过write属性在每个DataFrame的基础上访问`DataFrameWriter`：

```scala
// in Scala
dataFrame.write
```

After we have a `DataFrameWriter`, we specify three values: the format, a series of options, and the save mode. At a minimum, you must supply a path. We will cover the potential for options, which vary from data source to data source, shortly.

有了 `DataFrameWriter` 之后，我们指定三个值：格式，一系列选项和保存方式。至少必须提供一条路径。不久之后，我们将介绍各种选项的潜力，这些选项因数据源而异。

```scala
// in Scala
dataframe.write.format("csv")
.option("mode", "OVERWRITE")
.option("dateFormat", "yyyy-MM-dd")
.option("path", "path/to/file(s)")
.save()
```

#### <font color="#3399cc">Save modes 保存方式</font>

Save modes specify what will happen if Spark finds data at the specified location (assuming all else equal). Table 9-2 lists the save modes. 

保存方式指定如果Spark在指定位置找到数据（假设所有其他条件相等）将发生的情况。表9-2列出了保存方式。

 Table 9-2. Spark’s save modes

| Save mode       | Description                                                  |
| --------------- | ------------------------------------------------------------ |
| `append`        | Appends the output files to the list of files that already exist at that location<br />将输出文件追加到该位置已存在的文件列表中 |
| `overwrite`     | Will completely overwrite any data that already exists there<br />将完全覆盖那里已经存在的任何数据 |
| `errorIfExists` | Throws an error and fails the write if data or files already exist at the specified location<br />如果指定位置已经存在数据或文件，则会引发错误并导致写入失败 |
| `ignore`        | If data or files exist at the location, do nothing with the current DataFrame<br />如果该位置存在数据或文件，请对当前DataFrame不执行任何操作 |

 The default is `errorIfExists`. This means that if Spark finds data at the location to which you’re writing, it will fail the write immediately.

默认值为`errorIfExists`。这意味着，如果Spark在您要写入的位置找到数据，它将立即导致写入失败。

We’ve largely covered the core concepts that you’re going to need when using data sources, so now let’s dive into each of Spark’s native data sources.

我们已经在很大程度上涵盖了使用数据源时需要的核心概念，因此现在让我们深入研究Spark的每个本地数据源。

## <font color="#9a161a">CSV Files </font>

CSV stands for commma-separated values. This is a common text file format in which each line represents a single record, and commas separate each field within a record. CSV files, while seeming well structured, are actually one of the trickiest file formats you will encounter because not many assumptions can be made in production scenarios about what they contain or how they are structured. For this reason, the CSV reader has a large number of options. These options give you the ability to work around issues like certain characters needing to be escaped—for example, commas inside of columns when the file is also comma-delimited or null values labeled in an unconventional way.

CSV代表以逗号分隔的值。这是一种常见的文本文件格式，其中每一行代表一个记录，并用逗号分隔记录中的每个字段。CSV文件虽然看起来结构良好，但实际上是您将遇到的最棘手的文件格式之一，因为在生产方案中无法对其包含的内容或结构进行很多假设。因此，CSV读取器具有大量选项。这些选项使您能够解决某些需要转义的字符等问题，例如，文件也是逗号分隔时的列内逗号，或者以非常规方式标记的空值。

### <font color="#00000">CSV Options</font>

Table 9-3 presents the options available in the CSV reader. 

表9-3列出了CSV阅读器中可用的选项。

Table 9-3. CSV data source options  CSV数据源选项

| **Read/write** | **Key**                     | **Potential values**                                         | **Default**                  | **Description**                                              |
| -------------- | --------------------------- | ------------------------------------------------------------ | ---------------------------- | ------------------------------------------------------------ |
| Both           | sep                         | Any single string character                                  | ,                            | The single character that is used as separator for each field and value.<br />用作每个字段和值的分隔符的单个字符。 |
| Both           | header                      | true, false                                                  | false                        | A Boolean flag that declares whether the first line in the file(s) are the names of the columns.<br />布尔值标志，用于声明文件中的第一行是否为列名。 |
| Read           | escape                      | Any string character                                         | \                            | The character Spark should use to escape other characters in the file.<br />字符Spark应该用于转义文件中的其他字符。 |
| Read           | inferSchema                 | true, false                                                  | false                        | Specifies whether Spark should infer column types when reading the file.<br />指定在读取文件时Spark是否应推断列类型。 |
| Read           | IgnoreLeadingWhiteSpace     | true, false                                                  | false                        | Declares whether leading spaces from values being read should be skipped.<br />声明是否应跳过读取值的前导空格。 |
| Read           | IgnoreTrailingWhiteSpace    | true, false                                                  | false                        | Declares whether trailing spaces from values being read should be skipped.<br />声明是否应跳过读取值的尾随空格。 |
| Both           | nullValue                   | Any string character                                         | “”                           | Declares what character represents a null value in the file.<br/>声明什么字符代表文件中的空值。 |
| Both           | nanValue                    | Any string character                                         | NaN                          | Declares what character represents a NaN or missing character in the CSV file.<br />在CSV文件中声明代表NaN或缺少字符的字符。 |
| Both           | positivelnf                 | Any string or character                                      | Inf                          | Declares what character(s) represent a positive infinite value.<br />声明哪些字符表示正无穷大。 |
| Both           | negativelnf                 | Any string or character                                      | -Inf                         | Declares what character(s) represent a negative infinite value.<br />声明哪些字符表示负无穷大。 |
| Both           | compression or codec        | None, uncompressed. bzip2, deflate, gzip, Iz4, or snappy     | none                         | Declares what compression codec Spark should use to read or write the file.<br />声明Spark应当使用哪种压缩编解码器读取或写入文件。 |
| Both           | dateFormat                  | Any string or character that conforms to java’s `SimpleDataFormat`. | yyyy-MM-dd                   | Declares the date format for any columns that are date type.<br />声明任何日期类型列的日期格式。 |
| Both           | timestampFormat             | Any string or character that conforms to java's `SimpleDataFormat`. | yyyy-MM-dd’T’HH:mm :ss.SSSZZ | Declares the timestamp format for any columns that are timestamp type.<br />声明所有属于时间戳类型的列的时间戳格式。 |
| Read           | maxColumns                  | Any integer                                                  | 20480                        | Declares the maximum number of columns in the file.<br />声明文件中的最大列数。 |
| Read           | maxCharsPerColunn           | Any integer                                                  | 1000000                      | Declares the maximum number of characters in a column.<br />声明一列中的最大字符数。 |
| Read           | escapeQuotes                | true, false                                                  | true                         | Declares whether Spark should escape quotes that are found in lines.<br />声明Spark是否应该转义在行中找到的引号。 |
| Read           | maxMalformedLogPerPartition | Any integer                                                  | 10                           | Sets the maximum number of malformed rows Spark will log for each partition. Malformed records beyond this number will be ignored.<br />设置Spark将为每个分区记录的格式错误的最大行数。超出此数字的格式错误的记录将被忽略。 |
| Write          | quoteAll                    | true, false                                                  | false                        | Specifies whether all values should be enclosed in quotes, as opposed to just escaping values that have a quote character.<br />指定是否所有值都应该用引号引起来，而不是仅转义具有引号字符的值。 |
| Read           | multiLine                   | true, false                                                  | false                        | This option allows you to read multiline CSV files where each logical row in the CSV file might span multiple rows in the file itself.<br />此选项使您可以读取多行CSV文件，其中CSV文件中的每个逻辑行都可能跨越文件本身中的多个行。 |

### <font color="#00000">Reading CSV Files</font>

To read a CSV file, like any other format, we must first create a `DataFrameReader` for that specific format. Here, we specify the format to be CSV: 

要读取CSV文件，就像其他任何格式一样，我们必须首先为该特定格式创建一个`DataFrameReader`。在这里，我们将格式指定为CSV：

```scala
spark.read.format("csv")
```

After this, we have the option of specifying a schema as well as modes as options. Let’s set a couple of options, some that we saw from the beginning of the book and others that we haven’t seen yet.

此后，我们可以选择指定模式（schema）以及方式（mode）作为选项。让我们设置几个选项，其中一些是我们从本书开始就看到的，还有一些我们还没有看到的。

We’ll set the header to true for our CSV file, the mode to be `FAILFAST`, and `inferSchema` to true:

我们将CSV文件的标头设置为true，将方式（mode）设置为 `FAILFAST`，将 `inferSchema` 设置为true： 

```scala
// in Scala
spark.read.format("csv")
.option("header", "true")
.option("mode", "FAILFAST")
.option("inferSchema", "true")
.load("some/path/to/file.csv")
```

As mentioned, we can use the mode to specify how much tolerance we have for malformed data. For example, we can use these modes and the schema that we created in Chapter 5 to ensure that our file(s) conform to the data that we expected:

如前所述，我们可以使用该方式（mode）来指定对畸形数据的容忍度。例如，我们可以使用这些方式（mode）和我们在第5章中创建的模式（schema）来确保我们的文件符合我们期望的数据：

```scala
// in Scala
import org.apache.spark.sql.types.{StructField, StructType, StringType, LongType}

val myManualSchema = new StructType(Array(
new StructField("DEST_COUNTRY_NAME", StringType, true),
new StructField("ORIGIN_COUNTRY_NAME", StringType, true),
new StructField("count", LongType, false)
))

spark.read.format("csv")
.option("header", "true")
.option("mode", "FAILFAST").schema(myManualSchema)
.load("/data/flight-data/csv/2010-summary.csv")
.show(5)
```

Things get tricky when we don’t expect our data to be in a certain format, but it comes in that way, anyhow. For example, let’s take our current schema and change all column types to `LongType`. This does not match the actual schema, but Spark has no problem with us doing this. The problem will only manifest itself when Spark actually reads the data. As soon as we start our Spark job, it will immediately fail (after we execute a job) due to the data not conforming to the specified schema:

当我们不希望数据采用某种特定格式时，事情就会变得棘手，但无论如何都是这样。例如，让我们采用当前的模式（schema）并将所有列类型更改为`LongType`。这与实际的模式（schema）不匹配，但是Spark对此没有问题。仅当Spark实际读取数据时，问题才会显现出来。一旦开始执行Spark作业，由于数据不符合指定的模式，它将立即失败（在执行作业之后）：

```scala
// in Scala
val myManualSchema = new StructType(Array(
new StructField("DEST_COUNTRY_NAME", StringType, true),
new StructField("ORIGIN_COUNTRY_NAME", StringType, true),
new StructField("count", LongType, false) ))
spark.read.format("csv")
.option("header", "true")
.option("mode", "FAILFAST")
.schema(myManualSchema)
.load("/data/flight-data/csv/2010-summary.csv")
.take(5)
```

In general, Spark will fail only at job execution time rather than DataFrame definition time—even if, for example, we point to a file that does not exist. This is due to lazy evaluation, a concept we learned about in Chapter 2.

通常，Spark仅在作业执行时失败，而不是在DataFrame定义时失败，即使例如，我们指向的文件不存在。这是由于我们在第2章中学到了惰性求值（lazy evaluation）。

### <font color="#00000">Writing CSV Files</font>

Just as with reading data, there are a variety of options (listed in Table 9-3) for writing data when we write CSV files. This is a subset of the reading options because many do not apply when writing data (like `maxColumns` and `inferSchema`). Here’s an example: 

就像读取数据一样，当我们编写CSV文件时，有多种选项（表9-3中列出）用于写数据。这是读取选项的子集，因为在写入数据时，许多选项均不适用（例如`maxColumns`和`inferSchema`）。这是一个例子：

```scala
// in Scala
val csvFile = spark.read.format("csv")
.option("header", "true").option("mode", "FAILFAST").schema(myManualSchema)
.load("/data/flight-data/csv/2010-summary.csv")
# in Python
csvFile = spark.read.format("csv")\
.option("header", "true")\
.option("mode", "FAILFAST")\
.option("inferSchema", "true")\
.load("/data/flight-data/csv/2010-summary.csv")
```

For instance, we can take our CSV file and write it out as a TSV file quite easily: 

例如，我们可以轻松提取CSV文件并将其作为TSV文件写出：

```scala
// in Scala
csvFile.write.format("csv").mode("overwrite").option("sep", "\t").save("/tmp/my-tsv-file.tsv")
```

```python
# in Python
csvFile.write.format("csv").mode("overwrite").option("sep", "\t")\
.save("/tmp/my-tsv-file.tsv")
```

When you list the destination directory, you can see that my-tsv-file is actually a folder with numerous files within it:

当您列出目标目录时，您可以看到my-tsv-file实际上是一个文件夹，其中包含许多文件：

```shell
$ ls /tmp/my-tsv-file.tsv/
/tmp/my-tsv-file.tsv/part-00000-35cf9453-1943-4a8c-9c82-9f6ea9742b29.csv
```

This actually reflects the number of partitions in our DataFrame at the time we write it out. If we were to repartition our data before then, we would end up with a different number of files. We discuss this trade-off at the end of this chapter.

实际上，这反映了我们在写出DataFrame时分区的数量。如果要在此之前对数据进行重新分区，最终将获得不同数量的文件。我们将在本章末尾讨论这种权衡。

## <font color="#9a161a">JSON Files</font>

Those coming from the world of JavaScript are likely familiar with JavaScript Object Notation, or JSON, as it’s commonly called. There are some catches when working with this kind of data that are worth considering before we jump in. In Spark, when we refer to JSON files, we refer to line-delimited JSON files. This contrasts with files that have a large JSON object or array per file. 

那些来自JavaScript世界的人可能熟悉JavaScript Object Notation，即JSON（通常称为JSON）。使用此类数据时，有一些陷阱值得我们跳入之前考虑。在Spark中，当我们引用JSON文件时，我们引用的是行分隔JSON文件。这与每个文件具有较大JSON对象或数组的文件形成对比。

The line-delimited versus multiline trade-off is controlled by a single option: `multiLine`. When you set this option to true, you can read an entire file as one json object and Spark will go through the work of parsing that into a DataFrame. Line-delimited JSON is actually a much more stable format because it allows you to append to a file with a new record (rather than having to read in an entire file and then write it out), which is what we recommend that you use. Another key reason for the popularity of line-delimited JSON is because JSON objects have structure, and JavaScript (on which JSON is based) has at least basic types. This makes it easier to work with because Spark can make more assumptions on our behalf about the data. You’ll notice that there are significantly less options than we saw for CSV because of the objects.

行定界与多行权衡由一个选项控制：`multiLine`。当将此选项设置为true时，您可以将整个文件作为一个json对象读取，Spark将完成将其解析为DataFrame的工作。行分隔的JSON实际上是一种更加稳定的格式，因为它允许您将具有新记录的文件追加到文件中（而不是必须读取整个文件然后将其写出），这是我们建议您使用的格式。行分隔JSON流行的另一个关键原因是因为JSON对象具有结构，而JavaScript（基于JSON的JavaScript）至少具有基本类型。这使使用起来更容易，因为Spark可以代表我们对数据做出更多假设。您会注意到，由于对象的原因，选项比我们看到的要少得多。

### <font color="#00000">JSON Options</font>

Table 9-4 lists the options available for the JSON object, along with their descriptions.

表9-4列出了可用于JSON对象的选项及其说明。

Table 9-4. JSON data source options JSON数据源选项

| Read/write | Key                                     | Potential values                                             | Default                                                      | Description                                                  |
| ---------- | --------------------------------------- | :----------------------------------------------------------- | ------------------------------------------------------------ | :----------------------------------------------------------- |
| Both       | compression or codec                    | None,<br/>uncompressed,<br/>bzip2, deflate,<br />gzip, lz4, or<br/>snappy | none                                                         | Declares what compression codec Spark should use to read or write the file.<br />声明当Spark读取或写入文件的压缩编解码器。 |
| Both       | dateFormat                              | Any string or<br/>character that<br/>conforms to Java’s<br/>SimpleDataFormat. | yyyy-MM-dd                                                   | Declares the date format for any columns that are date type.<br />声明任何日期类型列的日期格式。 |
| Both       | timestampFormat                         | Any string or<br/>character that<br/>conforms to Java’s<br/>SimpleDataFormat. | yyyy-MM-dd’T’<br />HH:mm:ss.SSSZZ                            | Declares the timestamp format for any columns that are timestamp type.<br />声明任何日期类型列的日期格式。 |
| Read       | primitiveAsString                       | true, false                                                  | false                                                        | Infers all primitive values as string type.<br />将所有原始值推断为字符串类型。 |
| Read       | allowComments                           | true, false                                                  | false                                                        | Ignores Java/C++ style comment in JSON records.<br />忽略JSON记录中的Java / C ++样式注释。 |
| Read       | allowUnquoted<br />FieldNames           | true, false                                                  | false                                                        | Allows unquoted JSON field names.<br />允许不带引号的JSON字段名称 |
| Read       | allowSingleQuotes                       | true, false                                                  | true                                                         | Allows single quotes in addition to double quotes.<br />除双引号外，还允许单引号。 |
| Read       | allowNumeric<br />LeadingZeros          | true, false                                                  | false                                                        | Allows leading zeroes in numbers (e.g., 00012).<br/>允许数字前导零（例如00012）。 |
| Read       | allowBackslash<br/>EscapingAnyCharacter | true, false                                                  | false                                                        | Allows accepting quoting of all characters using backslash quoting mechanism.<br/>允许使用反斜杠引用机制接受所有字符的引用。 |
| Read       | columnName<br />OfCorruptRecord         | Any string                                                   | Value of<br/>spark.sql.column<br/>&<br />NameOfCorruptRecord | new field having a malformed string created by `permissive` mode. This will override the configuration value.<br/>由 `permissive` 方式（mode）创建的字符串格式错误的新字段。这将覆盖配置值。 |
| Read       | multiLine                               | true, false                                                  | false                                                        | Allows for reading in non-line-delimited JSON files.<br/>允许读取非行分隔的JSON文件。 |

Now, reading a line-delimited JSON file varies only in the format and the options that we specify:

现在，读取以行分隔的JSON文件仅在格式和我们指定的选项上有所不同：

```scala
spark.read.format("json")
```

### <font color="#00000">Reading JSON Files</font>

Let’s look at an example of reading a JSON file and compare the options that we’re seeing: 

让我们看一个读取JSON文件并比较我们看到的选项的示例：

```scala
// in Scala
spark.read.format("json").option("mode", "FAILFAST").schema(myManualSchema)
.load("/data/flight-data/json/2010-summary.json").show(5)
```

```python
# in Python
spark.read.format("json").option("mode", "FAILFAST")\
.option("inferSchema", "true")\
.load("/data/flight-data/json/2010-summary.json").show(5)
```

### <font color="#00000">Writing JSON Files</font>

 Writing JSON files is just as simple as reading them, and, as you might expect, the data source does not matter. Therefore, we can reuse the CSV DataFrame that we created earlier to be the source for our JSON file. This, too, follows the rules that we specified before: one file per partition will be written out, and the entire DataFrame will be written out as a folder. It will also have one JSON object per line:

编写JSON文件就像读取它们一样简单，而且，正如您可能期望的那样，数据源无关紧要。因此，我们可以重用我们先前创建的CSV DataFrame作为JSON文件的源。这也遵循我们之前指定的规则：每个分区将写入一个文件，而整个DataFrame将作为一个文件夹写入。每行还将有一个JSON对象：

```scala
// in Scala
csvFile.write.format("json").mode("overwrite").save("/tmp/my-json-file.json")
```

```python
# in Python
csvFile.write.format("json").mode("overwrite").save("/tmp/my-json-file.json")
```

```shell
$ ls /tmp/my-json-file.json//tmp/my-json-file.json/part-00000-tid-543....json
```

## <font color="#9a161a">Parquet Files</font>

Parquet is an open source column-oriented data store that provides a variety of storage optimizations, especially for analytics workloads. It provides columnar compression, which saves storage space and allows for reading individual columns instead of entire files. It is a file format that works exceptionally well with Apache Spark and is in fact the default file format. We recommend writing data out to Parquet for long-term storage because reading from a Parquet file will always be more efficient than JSON or CSV. Another advantage of Parquet is that it supports complex types. This means that if your column is an array (which would fail with a CSV file, for example), map, or struct, you’ll still be able to read and write that file without issue. Here’s how to specify Parquet as the read format:

Parquet是面向列的开源数据存储，可提供各种存储优化，尤其是针对分析工作负载。它提供了列压缩，从而节省了存储空间，并允许读取单个列而不是整个文件。它是一种文件格式，可与Apache Spark配合使用，并且实际上是默认文件格式。我们建议将数据写到Parquet中进行长期存储，因为从Parquet文件中读取数据总是比JSON或CSV更有效。Parquet的另一个优点是它支持复杂类型。这意味着，如果您的列是数组（例如，CSV文件会失效），映射或结构，那么您仍然可以毫无问题地读写该文件。以下是将Parquet指定为读取格式的方法：

```scala
spark.read.format("parquet")
```

### <font color="#00000"> Reading Parquet Files</font>

Parquet has very few options because it enforces its own schema when storing data. Thus, all you need to set is the format and you are good to go. We can set the schema if we have strict requirements for what our DataFrame should look like. Oftentimes this is not necessary because we can use schema on read, which is similar to the `inferSchema` with CSV files. However, with Parquet files, this method is more powerful because the schema is built into the file itself (so no inference needed).

Parquet具有很少的选项，因为它在存储数据时会强制执行自己的模式。因此，您只需要设置格式就可以了。如果我们对DataFrame有严格的要求，则可以设置模式。通常，这不是必需的，因为我们可以在读取时使用模式，这与带有CSV文件的 `inferSchema` 相似。但是，对于Parquet文件，此方法功能更强大，因为该模式内置在文件本身中（因此无需进行推断）。

 Here are some simple examples reading from parquet : 

 以下是从 parquet 上读取的一些简单示例： 

 ```scala
spark.read.format("parquet")
 ```

```python
// in Scala
spark.read.format("parquet")
.load("/data/flight-data/parquet/2010-summary.parquet").show(5)
```

```python
# in Python
spark.read.format("parquet")\
.load("/data/flight-data/parquet/2010-summary.parquet").show(5)
```

#### <font color="#3399cc">Parquet options</font> 

As we just mentioned, there are very few Parquet options—precisely two, in fact—because it has a well-defined specification that aligns closely with the concepts in Spark. Table 9-5 presents the options.

正如我们刚才提到的，Parquet选项很少，实际上只有两个，因为它具有定义明确的规范，可以与Spark中的概念紧密结合。表9-5列出了这些选项。

---

<p><center><font face="constant-width" color="#c67171" size=3><strong>WARNING 警告</strong></font></center></p>
Even though there are only two options, you can still encounter problems if you’re working with incompatible Parquet files. Be careful when you write out Parquet files with different versions of Spark (especially older ones) because this can cause significant headache.

即使只有两个选项，但如果使用不兼容的Parquet文件，仍然会遇到问题。用不同版本的Spark（尤其是较旧的Spark）写出Parquet文件时要小心，因为这会引起严重的问题。

---

Table 9-5. Parquet data source options 

### <font color="#00000">Writing Parquet Files</font>

Writing Parquet is as easy as reading it. We simply specify the location for the file. The same partitioning rules apply: 

编写 Parquet 就像阅读它一样容易。 我们只需指定文件的位置。 相同的分区规则适用：

```scala
// in Scala
csvFile.write.format("parquet").mode("overwrite")
.save("/tmp/my-parquet-file.parquet")
```

```python
# in Python
csvFile.write.format("parquet").mode("overwrite")\
.save("/tmp/my-parquet-file.parquet")
```

## <font color="#9a161a">ORC Files</font>

ORC is a self-describing, type-aware columnar file format designed for Hadoop workloads. It is optimized for large streaming reads, but with integrated support for finding required rows quickly. ORC actually has no options for reading in data because Spark understands the file format quite well. An often-asked question is: What is the difference between ORC and Parquet? For the most part, they’re quite similar; the fundamental difference is that Parquet is further optimized for use with Spark, whereas ORC is further optimized for Hive.

ORC是一种专为Hadoop工作负载设计的自我描述、注意类型的列式文件格式。它针对大型流读取进行了优化，但是集成了对快速查找所需行的支持。ORC实际上没有读取数据的选项，因为Spark非常了解文件格式。一个经常问到的问题是：ORC和Parquet有什么区别？在大多数情况下，它们非常相似； 根本的区别在于Parquet进一步优化了与Spark一起使用，而ORC进一步优化了针对Hive。

### <font color="#00000">Reading Orc Files</font>

Here’s how to read an ORC file into Spark:  

以下是将ORC文件读入Spark的方法：

```scala
// in Scala
spark.read.format("orc").load("/data/flight-data/orc/2010-summary.orc").show(5)
```

```python
# in Python
spark.read.format("orc").load("/data/flight-data/orc/2010-summary.orc").show(5)
```

### <font color="#00000">Writing Orc Files</font>

At this point in the chapter, you should feel pretty comfortable taking a guess at how to write ORC files. It really follows the exact same pattern that we have seen so far, in which we specify the format and then save the file: 

在本章的这一点上，您应该对如何编写ORC文件进行猜测感到很自在。它实际上遵循我们到目前为止所看到的完全相同的模式，在该模式中，我们指定格式然后保存文件：

```scala
// in Scala
csvFile.write.format("orc").mode("overwrite").save("/tmp/my-json-file.orc")
```

```python
# in Python
csvFile.write.format("orc").mode("overwrite").save("/tmp/my-json-file.orc")
```

## <font color="#9a161a">SQL Databases</font>

SQL data sources are one of the more powerful connectors because there are a variety of systems to which you can connect (as long as that system speaks SQL). For instance you can connect to a MySQL database, a PostgreSQL database, or an Oracle database. You also can connect to SQLite, which is what we’ll do in this example. Of course, databases aren’t just a set of raw files, so there are more options to consider regarding how you connect to the database. Namely you’re going to need to begin considering things like authentication and connectivity (you’ll need to determine whether the network of your Spark cluster is connected to the network of your database system).

SQL数据源是功能更强大的连接器之一，因为可以连接多种系统（只要该系统使用SQL即可）。例如，您可以连接到MySQL数据库，PostgreSQL数据库或Oracle数据库。您还可以连接到SQLite，这是我们在此示例中所做的。当然，数据库不仅是一组原始文件，因此，关于如何连接数据库，还有更多选项可供考虑。即您将需要开始考虑诸如身份验证和连接之类的事情（您需要确定Spark集群的网络是否已连接到数据库系统的网络）。

To avoid the distraction of setting up a database for the purposes of this book, we provide a reference sample that runs on SQLite. We can skip a lot of these details by using SQLite, because it can work with minimal setup on your local machine with the limitation of not being able to work in a distributed setting. If you want to work through these examples in a distributed setting, you’ll want to connect to another kind of database.

为了避免为了本书而设置数据库，我们提供了一个在SQLite上运行的参考示例。通过使用SQLite，我们可以跳过很多这些详细信息，因为它可以在本地计算机上以最少的设置工作，并且不能在分布式设置中工作。如果要在分布式环境中浏览这些示例，则需要连接到另一种数据库。

<div style="background:#f7f7f7;">
    <P><center><font face="constant-width" color="#000000" size=3><strong>A PRIMER ON SQLITE</strong></font></center></P>
<p>
<u><a style="color:#0879e3" href="https://sqlite.org/index.html">SQLite is the most used database engine in the entire world</a></u>, and for good reason. It’s powerful, fast, and easy to understand. This is because a SQLite database is just a file. That’s going to make it very easy for you to get up and running because we include the source file in <u><a style="color:#0879e3;" href="https://github.com/databricks/Spark-The-Definitive-Guide/tree/master/data/flight-data/jdbc">the official repository</a></u> for this book. Simply download that file to your local machine, and you will be able to read from it and write to it. We’re using SQLite, but all of the code here works with more traditional relational databases, as well, like MySQL. The primary difference is in the properties that you include when you connect to the database. When we’re working with SQLite, there’s no notion of user or password.
<br /><br />
有充分的理由，<u><a style="color:#0879e3" href="https://sqlite.org/index.html">SQLite是全世界使用最广泛的数据库引擎</a></u>。它功能强大，快速且易于理解。这是因为SQLite数据库只是一个文件。这将使您非常容易地启动和运行，因为我们在本书的<u><a style="color:#0879e3;" href="https://github.com/databricks/Spark-The-Definitive-Guide/tree/master/data/flight-data/jdbc">官方资源库</a></u>中包含了源文件。只需将该文件下载到您的本地计算机上，您就可以对其进行读取和写入。我们使用的是SQLite，但此处的所有代码也适用于更传统的关系数据库，例如MySQL。主要区别在于连接数据库时所包含的属性。当我们使用SQLite时，没有用户或密码的概念。
</p>
</div>

---

<p><center><font face="constant-width" color="#c67171" size=3><strong>WARNING 警告</strong></font></center></p>
Although SQLite makes for a good reference example, it’s probably not what you want to use in production. Also, SQLite will not necessarily work well in a distributed setting because of its requirement to lock the entire database on write. The example we present here will work in a similar way using MySQL or PostgreSQL, as well.

尽管SQLite提供了很好的参考示例，但它并不是您想在生产中使用的功能。另外，由于需要在写入时锁定整个数据库，因此SQLite在分布式设置中不一定会很好地工作。我们在此提供的示例也可以使用MySQL或PostgreSQL以类似的方式工作。

---

To read and write from these databases, you need to do two things: include the Java Database Connectivity (JDBC) driver for you particular database on the spark classpath, and provide the proper JAR for the driver itself. For example, to be able to read and write from PostgreSQL, you might run something like this:

要从这些数据库读取和写入，您需要做两件事：在spark类路径上包含用于您的特定数据库的Java数据库连接（JDBC）驱动程序，并为驱动程序本身提供适当的JAR。例如，为了能够从PostgreSQL进行读取和写入，您可以运行以下命令：

```shell
./bin/spark-shell \
--driver-class-path postgresql-9.4.1207.jar \
--jars postgresql-9.4.1207.jar
```

Just as with our other sources, there are a number of options that are available when reading from and writing to SQL databases. Only some of these are relevant for our current example, but Table 9-6 lists all of the options that you can set when working with JDBC databases.

就像我们的其他来源一样，在读取和写入SQL数据库时，有许多可用的选项。其中只有一些与我们当前的示例相关，但是表9-6列出了在使用JDBC数据库时可以设置的所有选项。

Table 9-6. JDBC data source options 

| Property Name                                    | Meaning                                                      |
| ------------------------------------------------ | ------------------------------------------------------------ |
| url                                              | The JDBC URL to which to connect. The source-specific connection properties can be specified in the URL; for example, `jdbc:postgresql://localhost/test?user=fred&password=secret`.<br />要连接的JDBC URL。可以在URL中指定特定于源的连接属性。例如，`jdbc:postgresql://localhost/test?user=fred&password=secret` |
| dbtable                                          | The JDBC table to read. Note that anything that is valid in a FROM clause of a SQL query can be used. For example, instead of a full table you could also use a subquery in parentheses.<br/>要读取的JDBC表。注意，可以使用在SQL查询的FROM子句中有效的任何东西。例如，除了完整表之外，您还可以在括号中使用子查询。 |
| partitionColumn,<br/>lowerBound, <br/>upperBound | If any one of these options is specified, then all others must be set as well. In addition, `numPartitions` must be specified. These properties describe how to partition the table when reading in parallel from multiple workers. `partitionColumn` must be a numeric column from the table in question. Notice that `lowerBound` and `upperBound` are used only to decide the partition stride, not for filtering the rows in the table. Thus, all rows in the table will be partitioned and returned. This option applies only to reading.<br />如果指定了这些选项中的任何一个，则还必须设置所有其他选项。另外，必须指定`numPartitions`。这些属性描述了从多个 workers 并行读取时如何对表进行分区。`partitionColumn`必须是相关查询表的数值列。请注意，`lowerBound`和`upperBound`仅用于确定分区步幅，而不用于过滤表中的行。因此，表中的所有行都将被分区并返回。此选项仅适用于阅读。 |
| numPartitions                                    | The maximum number of partitions that can be used for parallelism in table reading and writing. This also determines the maximum number of concurrent JDBC connections. If the number of partitions to write exceeds this limit, we decrease it to this limit by calling coalesce(`numPartitions`) before writing.<br />表读写中可用于并行处理的最大分区数。这也确定了并发JDBC连接的最大数量。如果要写入的分区数超过了此限制，我们可以通过在写入之前调用Coalesce（`numPartitions`）来将其降至此限制。 |
| fetchsize                                        | The JDBC fetch size, which determines how many rows to fetch per round trip. This can help performance on JDBC drivers, which default to low fetch size (e.g., Oracle with 10 rows). This option applies only to reading.<br />JDBC的获取大小，它确定每轮要获取多少行。这可以帮助提高JDBC驱动程序的性能，该驱动程序默认为较小的获取大小（例如，具有10行的Oracle）。此选项仅适用于读取数据。 |
| batchsize                                        | The JDBC batch size, which determines how many rows to insert per round trip. This can help performance on JDBC drivers. This option applies only to writing. The default is 1000.<br/>JDBC批处理大小，它确定每个回合要插入多少行。这可以帮助提高JDBC驱动程序的性能。此选项仅适用于写入数据。默认值为1000。 |
| isolationLevel                                   | The transaction isolation level, which applies to current connection. It can be one of `NONE`, `READ_COMMITTED`, `READ_UNCOMMITTED`, `REPEATABLE_READ`, or `SERIALIZABLE`, corresponding to standard transaction isolation levels defined by JDBC’s Connection object. The default is `READ_UNCOMMITTED`. This option applies only to writing. For more information, refer to the documentation in `java.sql.Connection`.<br/>事务隔离级别，适用于当前连接。它可以是`NONE`，`READ_COMMITTED`，`READ_UNCOMMITTED`，`REPEATABLE_READ`或`SERIALIZABLE`之一，对应于JDBC的Connection对象定义的标准事务隔离级别。默认值为`READ_UNCOMMITTED`。此选项仅适用于写入数据。有关更多信息，请参考`java.sql.Connection`中的文档。 |
| truncate                                         | This is a JDBC writer-related option. When `SaveMode.Overwrite` is enabled, Spark truncates an existing table instead of dropping and re-creating it. This can be more efficient, and it prevents the table metadata (e.g., indices) from being removed. However, it will not work in some cases, such as when the new data has a different schema. The default is false. This option applies only to writing.<br/>这是与JDBC写入器相关的选项。启用`SaveMode.Overwrite`时，Spark会截断现有表，而不是删除并重新创建它。这样可以更有效，并且可以防止删除表元数据（例如索引）。但是，在某些情况下（例如，新数据具有不同的模式时），它将不起作用。默认为false。此选项仅适用于写作。 |
| createTableOptions                               | This is a JDBC writer-related option. If specified, this option allows setting of database-specific table and partition options when creating a table (e.g., `CREATE TABLE t (name string) ENGINE=InnoDB`). This option applies only to writing.<br/>这是与JDBC写入器相关的选项。如果指定，则此选项允许在创建表时设置特定于数据库的表和分区选项（例如 `CREATE TABLE t (name string) ENGINE=InnoDB` ）。此选项仅适用于写入数据。 |
| createTableColumnTypes                           | The database column data types to use instead of the defaults, when creating the table. Data type in formation should be specified in the same format as CREATE TABLE columns syntax (e.g., “`name CHAR(64), comments VARCHAR(1024)`”). The specified types should be valid Spark SQL data types. This option applies only to writing.<br/>创建表时要使用的数据库列数据类型，而不是缺省值。格式中的数据类型应以与CREATE TABLE列语法相同的格式指定（例如，“`name CHAR(64), comments VARCHAR(1024)`”）。指定的类型应该是有效的Spark SQL数据类型。此选项仅适用于写作。 |

### <font color="#00000">Reading from SQL Databases</font>

When it comes to reading a file, SQL databases are no different from the other data sources that we looked at earlier. As with those sources, we specify the format and options, and then load in the data:

在读取文件时，SQL数据库与我们之前看过的其他数据源没有什么不同。与这些源一样，我们指定格式和选项，然后加载数据：

```scala
// in Scala
val driver = "org.sqlite.JDBC"
val path = "/data/flight-data/jdbc/my-sqlite.db"
val url = s"jdbc:sqlite:/${path}"
val tablename = "flight_info"
```

```python
# in Python
driver = "org.sqlite.JDBC"
path = "/data/flight-data/jdbc/my-sqlite.db"
url = "jdbc:sqlite:" + path
tablename = "flight_info"
```

After you have defined the connection properties, you can test your connection to the database itself to ensure that it is functional. This is an excellent troubleshooting technique to confirm that your database is available to (at the very least) the Spark driver. This is much less relevant for SQLite because that is a file on your machine but if you were using something like MySQL, you could test the connection with the following:

定义连接属性后，可以测试与数据库本身的连接以确保其正常运行。这是一种出色的故障排除技术，可确保您的数据库可用于（至少）Spark驱动程序。这与SQLite无关紧要，因为这是您计算机上的文件，但是如果您使用的是类似MySQL的文件，则可以使用以下命令测试连接：

```scala
import java.sql.DriverManager
val connection = DriverManager.getConnection(url)
connection.isClosed()
connection.close()
```

If this connection succeeds, you’re good to go. Let’s go ahead and read the DataFrame from the SQL table:

如果此连接成功，那就很好了。让我们继续阅读SQL表中的DataFrame：

```scala
// in Scala
val dbDataFrame = spark.read.format("jdbc").option("url", url)
.option("dbtable", tablename).option("driver", driver).load()
```

```python
# in Python
dbDataFrame = spark.read.format("jdbc").option("url", url)\
.option("dbtable", tablename).option("driver", driver).load()
```

SQLite has rather simple configurations (no users, for example). Other databases, like PostgreSQL, require more configuration parameters. Let’s perform the same read that we just performed, except using PostgreSQL this time:

SQLite具有相当简单的配置（例如，没有用户）。其他数据库（例如PostgreSQL）需要更多配置参数。让我们执行与刚刚执行的读取相同的操作，除了这次使用PostgreSQL：

```scala
// in Scala
val pgDF = spark.read
.format("jdbc")
.option("driver", "org.postgresql.Driver")
.option("url", "jdbc:postgresql://database_server")
.option("dbtable", "schema.tablename")
.option("user", "username").option("password","my-secret-password").load()
```

```python
# in Python
pgDF = spark.read.format("jdbc")\
.option("driver", "org.postgresql.Driver")\
.option("url", "jdbc:postgresql://database_server")\
.option("dbtable", "schema.tablename")\
.option("user", "username").option("password", "my-secret-password").load()
```

As we create this DataFrame, it is no different from any other: you can query it, transform it, and join it without issue. You’ll also notice that there is already a schema, as well. That’s because Spark gathers this information from the table itself and maps the types to Spark data types. Let’s get only the distinct locations to verify that we can query it as expected:

在创建此DataFrame时，它与其他任何对象都没有什么不同：您可以对其进行查询，转换和加入，而不会出现问题。您还会注意到，也已经有一个模式。那是因为Spark会从表格本身收集此信息，然后将类型映射为Spark数据类型。让我们仅获取不同的位置，以验证我们可以按预期查询它：

```scala
dbDataFrame.select("DEST_COUNTRY_NAME").distinct().show(5)
```

```txt
+-----------------+
|DEST_COUNTRY_NAME|
+-----------------+
| Anguilla        |
| Russia          |
| Paraguay        |
| Senegal         |
| Sweden          |
+-----------------+
```

Awesome, we can query the database! Before we proceed, there are a couple of nuanced details that are worth understanding.

太好了，我们可以查询数据库了！在我们继续之前，有一些细微的细节值得理解。

### <font color="#00000">Query Pushdown 查询向下推导</font>

First, Spark makes a best-effort attempt to filter data in the database itself before creating the DataFrame. For example, in the previous sample query, we can see from the query plan that it selects only the relevant column name from the table: 

首先，Spark会尽最大努力在创建DataFrame之前过滤数据库本身中的数据。例如，在上一个示例查询中，我们可以从查询计划中看到它仅从表中选择相关的列名：

```scala
dbDataFrame.select("DEST_COUNTRY_NAME").distinct().explain
```

```txt
== Physical Plan ==
*HashAggregate(keys=[DEST_COUNTRY_NAME#8108], functions=[])
+- Exchange hashpartitioning(DEST_COUNTRY_NAME#8108, 200)
   +- *HashAggregate(keys=[DEST_COUNTRY_NAME#8108], functions=[])
	   +- *Scan JDBCRelation(flight_info) [numPartitions=1] ...
```

Spark can actually do better than this on certain queries. For example, if we specify a filter on our DataFrame, Spark will push that filter down into the database. We can see this in the explain plan under `PushedFilters`.

在某些查询中，Spark实际上比这更好。例如，如果我们在DataFrame上指定一个过滤器，Spark将把该过滤器下推到数据库中。我们可以在 `PushedFilters` 下的说明计划中看到这一点。

```scala
// in Scala
dbDataFrame.filter("DEST_COUNTRY_NAME in ('Anguilla', 'Sweden')").explain
```

```python
# in Python
dbDataFrame.filter("DEST_COUNTRY_NAME in ('Anguilla', 'Sweden')").explain()
```

```txt
== Physical Plan ==
*Scan JDBCRel... PushedFilters: [*In(DEST_COUNTRY_NAME, [Anguilla,Sweden])],
...
```

Spark can’t translate all of its own functions into the functions available in the SQL database in which you’re working. Therefore, sometimes you’re going to want to pass an entire query into your SQL that will return the results as a DataFrame. Now, this might seem like it’s a bit complicated, but it’s actually quite straightforward. Rather than specifying a table name, you just specify a SQL query. Of course, you do need to specify this in a special way; you must wrap the query in parenthesis and rename it to something—in this case, I just gave it the same table name:

Spark无法将其所有功能转换为您正在使用的SQL数据库中可用的功能。因此，有时您想要将整个查询传递到SQL中，该查询会将结果作为DataFrame返回。现在，这似乎有些复杂，但实际上非常简单。您只需指定一个SQL查询，而不是指定表名。当然，您确实需要以一种特殊的方式指定它。您必须将查询括在括号中并将其重命名为某种东西——在这种情况下，我只是给了它相同的表名：

```scala
// in Scala
val pushdownQuery = """(SELECT DISTINCT(DEST_COUNTRY_NAME) FROM flight_info) AS flight_info"""
val dbDataFrame = spark.read.format("jdbc")
.option("url", url).option("dbtable", pushdownQuery).option("driver", driver)
.load()
```

```python
# in Python
pushdownQuery = """(SELECT DISTINCT(DEST_COUNTRY_NAME) FROM flight_info) AS flight_info"""
dbDataFrame = spark.read.format("jdbc")\
.option("url", url).option("dbtable", pushdownQuery).option("driver", driver)\
.load()
```

Now when you query this table, you’ll actually be querying the results of that query. We can see this in the explain plan. Spark doesn’t even know about the actual schema of the table, just the one that results from our previous query:

现在，当您查询该表时，您实际上将在查询该查询的结果。我们可以在解释计划中看到这一点。Spark甚至不知道表的实际模式，仅知道我们先前查询的结果：

```scala
dbDataFrame.explain()
```

```txt
== Physical Plan ==
*Scan JDBCRelation(
(SELECT DISTINCT(DEST_COUNTRY_NAME)
	FROM flight_info) as flight_info
) [numPartitions=1] [DEST_COUNTRY_NAME#788] ReadSchema: ...
```

#### <font color="#3399cc">Reading from databases in parallel</font>

All throughout this book, we have talked about partitioning and its importance in data processing. Spark has an underlying algorithm that can read multiple files into one partition, or conversely, read multiple partitions out of one file, depending on the file size and the “splitability” of the file type and compression. The same flexibility that exists with files, also exists with SQL databases except that you must configure it a bit more manually. What you can configure, as seen in the previous options, is the ability to specify a maximum number of partitions to allow you to limit how much you are reading and writing in parallel:

在本书中，我们都谈到了分区及其在数据处理中的重要性。Spark具有一种基础算法，可以根据文件大小以及文件类型和压缩的“可拆分性”将多个文件读取到一个分区中，或者反之，可以从一个文件读取多个分区。文件具有相同的灵活性，SQL数据库也具有相同的灵活性，只是您必须手动进行一些配置。如前面的选项所示，您可以配置的功能是指定最大分区数，以限制并行读取和写入的数量：  

```scala
// in Scala
val dbDataFrame = spark.read.format("jdbc")
.option("url", url).option("dbtable", tablename).option("driver", driver)
.option("numPartitions", 10).load()
```

```python
# in Python
dbDataFrame = spark.read.format("jdbc")\
.option("url", url).option("dbtable", tablename).option("driver", driver)\
.option("numPartitions", 10).load()
```

In this case, this will still remain as one partition because there is not too much data. However, this configuration can help you ensure that you do not overwhelm the database when reading and writing data:

在这种情况下，由于没有太多数据，因此仍将保留为一个分区。但是，此配置可以帮助您确保在读写数据时不会使数据库不知所措：

```scala
dbDataFrame.select("DEST_COUNTRY_NAME").distinct().show()
```

There are several other optimizations that unfortunately only seem to be under another API set. You can explicitly push predicates down into SQL databases through the connection itself. This optimization allows you to control the physical location of certain data in certain partitions by specifying predicates. That’s a mouthful, so let’s look at a simple example. We only need data from two countries in our data: Anguilla and Sweden. We could filter these down and have them pushed into the database, but we can also go further by having them arrive in their own partitions in Spark.

不幸的是，还有其他一些优化似乎只是在另一个API集之下。您可以通过连接本身将谓词显式向下推入SQL数据库。通过这种优化，您可以通过指定谓词来控制某些分区中某些数据的物理位置。这是一个大问题，所以让我们看一个简单的例子。我们只需要两个国家/地区的数据：安圭拉和瑞典。我们可以过滤掉它们并将它们推送到数据库中，但是我们也可以进一步通过将它们放入Spark中自己的分区中。

><center><strong>译者附</strong></center>
>**谓词（predicate）**——通常来说是函数的一种，是需要满足特定条件的函数。该条件就是“返回值是真值”，即返回的值必须为TRUE/FALSE/UNKNOWN）

We do that by specifying a list of predicates when we create the data source:

我们通过在创建数据源时指定谓词列表来做到这一点： 

```scala
// in Scala
val props = new java.util.Properties
props.setProperty("driver", "org.sqlite.JDBC")
val predicates = Array(
"DEST_COUNTRY_NAME = 'Sweden' OR ORIGIN_COUNTRY_NAME = 'Sweden'",
"DEST_COUNTRY_NAME = 'Anguilla' OR ORIGIN_COUNTRY_NAME = 'Anguilla'")
spark.read.jdbc(url, tablename, predicates, props).show()
spark.read.jdbc(url, tablename, predicates, props).rdd.getNumPartitions // 2
```

```python
# in Python
props = {"driver":"org.sqlite.JDBC"}
predicates = [
"DEST_COUNTRY_NAME = 'Sweden' OR ORIGIN_COUNTRY_NAME = 'Sweden'",
"DEST_COUNTRY_NAME = 'Anguilla' OR ORIGIN_COUNTRY_NAME = 'Anguilla'"]
spark.read.jdbc(url, tablename, predicates=predicates, properties=props).show()
spark.read.jdbc(url,tablename,predicates=predicates,properties=props)\
.rdd.getNumPartitions() # 2
```

```txt
+-----------------+-------------------+-----+
|DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|
+-----------------+-------------------+-----+
|    Sweden       |   United States   | 65  |
|  United States  |      Sweden       | 73  |
|     Anguilla    |   United States   | 21  |
|  United States  |      Anguilla     | 20  |
+-----------------+-------------------+-----+
```

If you specify predicates that are not disjoint, you can end up with lots of duplicate rows. Here’s an example set of predicates that will result in duplicate rows:

如果您指定不互斥的谓词，则最终可能会出现很多重复的行。这是一组导致重复行的谓词示例：

```scala
// in Scala
val props = new java.util.Properties
props.setProperty("driver", "org.sqlite.JDBC")
val predicates = Array(
"DEST_COUNTRY_NAME != 'Sweden' OR ORIGIN_COUNTRY_NAME != 'Sweden'",
"DEST_COUNTRY_NAME != 'Anguilla' OR ORIGIN_COUNTRY_NAME != 'Anguilla'")
spark.read.jdbc(url, tablename, predicates, props).count() // 510
```

```python
# in Python
props = {"driver":"org.sqlite.JDBC"}
predicates = [
"DEST_COUNTRY_NAME != 'Sweden' OR ORIGIN_COUNTRY_NAME != 'Sweden'",
"DEST_COUNTRY_NAME != 'Anguilla' OR ORIGIN_COUNTRY_NAME != 'Anguilla'"]
spark.read.jdbc(url, tablename, predicates=predicates, properties=props).count()
```

#### <font color="#3399cc">Partitioning based on a sliding window</font> 

Let’s take a look to see how we can partition based on predicates. In this example, we’ll partition based on our numerical count column. Here, we specify a minimum and a maximum for both the first partition and last partition. Anything outside of these bounds will be in the first partition or final partition. Then, we set the number of partitions we would like total (this is the level of parallelism).

让我们看一下如何基于谓词进行分区。在此示例中，我们将基于数值计数列进行分区。在此，我们为第一个分区和最后一个分区都指定了最小值和最大值。这些范围之外的任何内容都将位于第一个分区或最终分区中。然后，我们设置希望的分区总数（这是并行度）。

Spark then queries our database in parallel and returns `numPartitions` partitions. We simply modify the upper and lower bounds in order to place certain values in certain partitions. No filtering is taking place like we saw in the previous example:

然后，Spark并行查询我们的数据库，并返回 `numPartitions` 分区。我们只需修改上限和下限，以便将某些值放置在某些分区中。不会像前面的示例中那样进行过滤： 

```scala
// in Scala
val colName = "count"
val lowerBound = 0L
val upperBound = 348113L // this is the max count in our database
val numPartitions = 10
```

```python
# in Python
colName = "count"
lowerBound = 0L
upperBound = 348113L # this is the max count in our database
numPartitions = 10
```

This will distribute the intervals equally from low to high:

这将从低到高平均分配时间间隔：

```scala
// in Scala
spark.read.jdbc(url,tablename,colName,lowerBound,upperBound,numPartitions,props)
.count() // 255
```

```python
# in Python
spark.read.jdbc(url, tablename, column=colName, properties=props,
lowerBound=lowerBound, upperBound=upperBound,
numPartitions=numPartitions).count() # 255
```

### <font color="#000000">Writing to SQL Databases</font>

 Writing out to SQL databases is just as easy as before. You simply specify the URI and write out the data according to the specified write mode that you want. In the following example, we specify overwrite, which overwrites the entire table. We’ll use the CSV DataFrame that we defined earlier in order to do this:

写入SQL数据库就像以前一样容易。您只需指定URI并根据所需的指定写方式（mode）写出数据。在下面的示例中，我们指定覆盖，该覆盖将覆盖整个表。我们将使用我们先前定义的 CSV DataFrame 来做到这一点：

```scala
// in Scala
val newPath = "jdbc:sqlite://tmp/my-sqlite.db"
csvFile.write.mode("overwrite").jdbc(newPath, tablename, props)
```

```python
# in Python
newPath = "jdbc:sqlite://tmp/my-sqlite.db"
csvFile.write.jdbc(newPath, tablename, mode="overwrite", properties=props)
```

Let’s look at the results:

我们来看一下结果：

```scala
// in Scala
spark.read.jdbc(newPath, tablename, props).count() // 255
```

```python
# in Python
spark.read.jdbc(newPath, tablename, properties=props).count() # 255
```

Of course, we can append to the table this new table just as easily:

当然，我们可以轻松地将此新表添加到表中：

```scala
// in Scala
csvFile.write.mode("append").jdbc(newPath, tablename, props)
```

```python
# in Python
csvFile.write.jdbc(newPath, tablename, mode="append", properties=props)
```

Notice that count increases:

请注意，计数增加了：

```scala
// in Scala
spark.read.jdbc(newPath, tablename, props).count() // 765
```

```python
# in Python
spark.read.jdbc(newPath, tablename, properties=props).count() # 765
```

## <font color="#9a161a">Text Files </font>

Spark also allows you to read in plain-text files. Each line in the file becomes a record in the DataFrame. It is then up to you to transform it accordingly. As an example of how you would do this,suppose that you need to parse some Apache log files to some more structured format, or perhaps you want to parse some plain text for natural-language processing. Text files make a great argument for the Dataset API due to its ability to take advantage of the flexibility of native types.

Spark还允许您读取纯文本文件。文件中的每一行都成为DataFrame中的一条记录。然后由您自己进行相应的转换。作为如何执行此操作的示例，假设您需要将某些Apache日志文件解析为某种更结构化的格式，或者您可能想解析一些纯文本以进行自然语言处理。文本文件因其能够利用本地类型（native type）的灵活性而成为Dataset API的重要论据。

### <font color="#00000">Reading Text Files </font>

Reading text files is straightforward: you simply specify the type to be textFile. With textFile, partitioned directory names are ignored. To read and write text files according to partitions, you should use text, which respects partitioning on reading and writing:

读取文本文件非常简单：您只需将类型指定为 textFile。使用 textFile，分区目录名将被忽略。要根据分区读取和写入文本文件，您应该使用文本，这在读写时要考虑分区：

```scala
spark.read.textFile("/data/flight-data/csv/2010-summary.csv")
.selectExpr("split(value, ',') as rows").show()
```

```txt
+--------------------+
|         rows       |
+--------------------+
|[DEST_COUNTRY_NAM...|
|[United States, R...|
...
|[United States, A...|
|[Saint Vincent an...|
|[Italy, United St...|
+--------------------+
```

### <font color="#00000">Writing Text Files</font>

When you write a text file, you need to be sure to have only one string column; otherwise, the write will fail: 

在写入文本文件时，您需要确保只有一个字符串列。否则，写入将失败：

```scala
csvFile.select("DEST_COUNTRY_NAME").write.text("/tmp/simple-text-file.txt")
```

If you perform some partitioning when performing your write (we’ll discuss partitioning in the next couple of pages), you can write more columns. However, those columns will manifest as directories in the folder to which you’re writing out to, instead of columns on every single file : 

如果您在执行写入操作时进行了分区（我们将在接下来的几页中讨论分区），则可以写入更多列。但是，这些列将显示为您要写入的文件夹中的目录，而不是每个文件的列：

```scala
// in Scala
csvFile.limit(10).select("DEST_COUNTRY_NAME", "count")
.write.partitionBy("count").text("/tmp/five-csv-files2.csv")
```

```python
# in Python
csvFile.limit(10).select("DEST_COUNTRY_NAME", "count")\
.write.partitionBy("count").text("/tmp/five-csv-files2py.csv")
```

## <font color="#9a161a">Advanced I/O Concepts </font>

We saw previously that we can control the parallelism of files that we write by controlling the partitions prior to writing. We can also control specific data layout by controlling two things: bucketing and partitioning (discussed momentarily).

先前我们看到，可以通过在写入之前控制分区来控制所写入文件的并行性。我们还可以通过控制两件事来控制特定的数据布局：存储和分区（暂时讨论）。

### <font color="#00000">Splittable File Types and Compression</font>

Certain file formats are fundamentally “splittable.” This can improve speed because it makes it possible for Spark to avoid reading an entire file, and access only the parts of the file necessary to satisfy your query. Additionally if you’re using something like Hadoop Distributed File System (HDFS), splitting a file can provide further optimization if that file spans multiple blocks. In conjunction with this is a need to manage compression. Not all compression schemes are splittable. How you store your data is of immense consequence when it comes to making your Spark jobs run smoothly. We recommend Parquet with gzip compression.

某些文件格式从根本上讲是“可拆分的”。这可以提高速度，因为它使Spark可以避免读取整个文件，而仅访问满足查询所需的文件部分。此外，如果您使用的是Hadoop分布式文件系统（HDFS）之类的文件，则如果文件跨越多个块，则拆分文件可以提供进一步的优化。与此相关，需要管理压缩。并非所有压缩方案都是可拆分的。当使Spark作业平稳运行时，如何存储数据将产生巨大的后果。我们建议使用gzip压缩的Parquet。

### <font color="#00000">Reading Data in Parallel</font>

Multiple executors cannot read from the same file at the same time necessarily, but they can read different files at the same time. In general, this means that when you read from a folder with multiple files in it, each one of those files will become a partition in your DataFrame and be read in by available executors in parallel (with the remaining queueing up behind the others).

多个 worker 不一定必须同时读取同一文件，但是他们可以同时读取不同的文件。通常，这意味着当您从其中包含多个文件的文件夹中读取时，这些文件中的每个文件都将成为DataFrame中的一个分区，并由可用的 executor 并行读取（其余文件排在其他文件后面）。

### <font color="#00000">Writing Data in Parallel</font>

The number of files or data written is dependent on the number of partitions the DataFrame has at the time you write out the data. By default, one file is written per partition of the data. This means that although we specify a “file,” it’s actually a number of files within a folder, with the name of the specified file, with one file per each partition that is written.

写入的文件或数据的数量取决于您写出数据时DataFrame拥有的分区数量。默认情况下，每个数据分区写入一个文件。这意味着尽管我们指定了一个“文件”，但实际上它是一个文件夹中的许多文件，具有指定文件的名称，每个写入的分区每个文件一个。

For example, the following code:

例如下面的代码 ：

```txt
csvFile.repartition(5).write.format("csv").save("/tmp/multiple.csv")
```

will end up with five files inside of that folder. As you can see from the list call:

最终将在该文件夹中包含五个文件。从列表调用中可以看到：

```shell
ls /tmp/multiple.csv

/tmp/multiple.csv/part-00000-767df509-ec97-4740-8e15-4e173d365a8b.csv
/tmp/multiple.csv/part-00001-767df509-ec97-4740-8e15-4e173d365a8b.csv
/tmp/multiple.csv/part-00002-767df509-ec97-4740-8e15-4e173d365a8b.csv
/tmp/multiple.csv/part-00003-767df509-ec97-4740-8e15-4e173d365a8b.csv
/tmp/multiple.csv/part-00004-767df509-ec97-4740-8e15-4e173d365a8b.csv
```

#### <font color="#3399cc">Partitioning</font>

Partitioning is a tool that allows you to control what data is stored (and where) as you write it. When you write a file to a partitioned directory (or table), you basically encode a column as a folder. What this allows you to do is skip lots of data when you go to read it in later, allowing you to read in only the data relevant to your problem instead of having to scan the complete dataset. These are supported for all file-based data sources:

分区是一种工具，可让您在写入数据时控制要存储的数据（以及存储在何处）。当您将文件写入分区目录（或表）时，基本上将一列编码为文件夹。这允许您执行的操作是稍后读入时跳过许多数据，从而仅读取与问题相关的数据，而不必扫描整个数据集。所有基于文件的数据源均支持以下功能：

```scala
// in Scala
csvFile.limit(10).write.mode("overwrite").partitionBy("DEST_COUNTRY_NAME")
.save("/tmp/partitioned-files.parquet")
```
```python
# in Python
csvFile.limit(10).write.mode("overwrite").partitionBy("DEST_COUNTRY_NAME")\
.save("/tmp/partitioned-files.parquet")
```

Upon writing, you get a list of folders in your Parquet “file”:

编写后，您会在Parquet“文件”中获得一个文件夹列表：

```shell
$ ls /tmp/partitioned-files.parquet
```

```txt
...
DEST_COUNTRY_NAME=Costa Rica/
DEST_COUNTRY_NAME=Egypt/
DEST_COUNTRY_NAME=Equatorial Guinea/
DEST_COUNTRY_NAME=Senegal/
DEST_COUNTRY_NAME=United States/
```

Each of these will contain Parquet files that contain that data where the previous predicate was true: 

其中每个将包含Parquet文件，这些文件包含先前谓词为 true 的数据：

```shell
$ ls /tmp/partitioned-files.parquet/DEST_COUNTRY_NAME=Senegal/
part-00000-tid.....parquet
```

This is probably the lowest-hanging optimization that you can use when you have a table that readers frequently filter by before manipulating. For instance, date is particularly common for a partition because, downstream, often we want to look at only the previous week’s data (instead of scanning the entire list of records). This can provide massive speedups for readers.

当您拥有一个表且读取器（reader）在操作这个表之前经常对其进行过滤，那么这可能是您可以使用的最容易优化。例如，日期在分区中尤为常见，因为在下游，我们通常只希望查看前一周的数据（而不是扫描整个记录列表）。这可以为读者提供巨大的提速。

#### <font color="#3399cc">Bucketing </font>

Bucketing is another file organization approach with which you can control the data that is specifically written to each file. This can help avoid shuffles later when you go to read the data because data with the same bucket ID will all be grouped together into one physical partition. This means that the data is prepartitioned according to how you expect to use that data later on, meaning you can avoid expensive shuffles when joining or aggregating.

存储桶是另一种文件组织方法，可用于控制专门写入每个文件的数据。这样可以避免以后再读取数据时发生数据再分配（shuffle），因为具有相同存储区ID的数据将全部分组到一个物理分区中。这意味着将根据您以后使用数据的方式对数据进行预分区，这意味着您可以避免在合并或聚合时进行代价很高的数据再分配（shuffle）。

Rather than partitioning on a specific column (which might write out a ton of directories), it’s probably worthwhile to explore bucketing the data instead. This will create a certain number of files and organize our data into those “buckets”:

与其在特定的列上进行分区（可能会写出大量的目录），不如探索对数据进行存储桶化。这将创建一定数量的文件，并将我们的数据组织到这些“存储桶”中： 

```scala
val numberBuckets = 10
val columnToBucketBy = "count"
csvFile.write.format("parquet").mode("overwrite")
.bucketBy(numberBuckets, columnToBucketBy).saveAsTable("bucketedFiles")
```

```shell
$ ls /user/hive/warehouse/bucketedfiles/

part-00000-tid-1020575097626332666-8....parquet
part-00000-tid-1020575097626332666-8....parquet
part-00000-tid-1020575097626332666-8....parquet
...
```

Bucketing is supported only for Spark-managed tables. For more information on bucketing and partitioning, watch [<font color="#0879e3"><u>this talk</u></font>](https://spark-summit.org/2017/events/why-you-should-care-about-data-layout-in-the-filesystem/) from Spark Summit 2017.

仅受Spark托管的表支持存储桶。有关存储分区和分区的更多信息，请观看Spark Summit 2017上的[<font color="#0879e3"><u>讨论</u></font>](https://spark-summit.org/2017/events/why-you-should-care-about-data-layout-in-the-filesystem/)。

><center><strong>译者附</strong></center>
>摘录： 本书第10章：《What Is SQL?》
>
>One important note is the concept of managed versus unmanaged tables. Tables store two important pieces of information. The data within the tables as well as the data about the tables; that is, the metadata. You can have Spark manage the metadata for a set of files as well as for the data. When you define a table from files on disk, you are defining an unmanaged table. When you use `saveAsTable` on a DataFrame, you are creating a managed table for which Spark will track of all of the relevant information.
>
>重要说明之一是托管表与非托管表的概念。表存储两个重要的信息。表中的数据以及有关表的数据；即元数据。您可以让Spark管理一组文件和数据的元数据。当您从磁盘上的文件定义表时，就是在定义非托管表。在DataFrame上使用 `saveAsTable` 时，您将创建一个托管表，Spark会为其跟踪所有相关信息。
>
>This will read your table and write it out to a new location in Spark format. You can see this reflected in the new explain plan. In the explain plan, you will also notice that this writes to the default Hive warehouse location. You can set this by setting the `spark.sql.warehouse.dir` configuration to the directory of your choosing when you create your `SparkSession`. By default Spark sets this to `/user/hive/warehouse`:  
>
>这将读取您的表并将其以Spark格式写到新位置。您可以在新的计划说明（explain plan）中看到这一点。在计划说明（explain plan）中，您还将注意到这将写入默认的Hive仓库位置。您可以通过在创建`SparkSession`时将`spark.sql.warehouse.dir`配置设置为所选目录来进行设置。默认情况下，Spark将其设置为 `/user/hive/warehouse`：
>
>you can also see tables in a specific database by using the query `show tables IN databaseName`,  where  `databaseName` represents the name of the database that you want to query.
>
>您还可以使用查询：`show tables IN databaseName` 查看特定数据库中的表，其中 `databaseName` 代表要查询的数据库的名称。

### <font color="#00000">Writing Complex Types</font>

As we covered in Chapter 6, Spark has a variety of different internal types. Although Spark can work with all of these types, not every single type works well with every data file format. For instance, CSV files do not support complex types, whereas Parquet and ORC do.

如第6章所述，Spark具有多种不同的内部类型。尽管Spark可以使用所有这些类型，但是并非每种类型都能很好地适用于每种数据文件格式。例如，CSV文件不支持复杂类型，而Parquet和ORC支持。

### <font color="#00000">Managing File Size </font>

Managing file sizes is an important factor not so much for writing data but reading it later on. When you’re writing lots of small files, there’s a significant metadata overhead that you incur managing all of those files. Spark especially does not do well with small files, although many file systems (like HDFS) don’t handle lots of small files well, either. You might hear this referred to as the “small file problem.” The opposite is also true: you don’t want files that are too large either, because it becomes inefficient to have to read entire blocks of data when you need only a few rows.

相对于写数据，管理文件大小对于稍后读取是一个十分重要的因素。当您写很多小文件时，管理所有这些文件会产生相当大的元数据开销。尽管许多文件系统（例如HDFS）也不能很好地处理许多小文件，但Spark尤其不适用于小文件。您可能会听到被称为“小文件问题”的情况。相反的情况也是如此：您也不想太大的文件，因为当您只需要几行数据时，不得不读取整个数据块是低效的。

Spark 2.2 introduced a new method for controlling file sizes in a more automatic way. We saw previously that the number of output files is a derivative of the number of partitions we had at write time (and the partitioning columns we selected). Now, you can take advantage of another tool in order to limit output file sizes so that you can target an optimum file size. You can use the `maxRecordsPerFile` option and specify a number of your choosing. This allows you to better control file sizes by controlling the number of records that are written to each file. For example, if you set an option for a writer as `df.write.option("maxRecordsPerFile", 5000)`, Spark will ensure that files will contain at most 5,000 records.

Spark 2.2引入了一种新方法，可以更自动地控制文件大小。先前我们看到输出文件的数量是写入时我们拥有的分区数量（以及我们选择的分区列）的派生数。现在，您可以利用另一个工具来限制输出文件的大小，以便您可以确定最佳的文件大小。您可以使用`maxRecordsPerFile`选项指定一个数字。这样可以通过控制写入每个文件的记录数来更好地控制文件大小。例如，如果您将写入器（writer）的选项设置为 `df.write.option("maxRecordsPerFile", 5000)`，Spark将确保文件最多包含5,000条记录。

## <font color="#9a161a">Conclusion</font>

In this chapter we discussed the variety of options available to you for reading and writing data in Spark. This covers nearly everything you’ll need to know as an everyday user of Spark. For the curious, there are ways of implementing your own data source; however, we omitted instructions for how to do this because the API is currently evolving to better support Structured Streaming. If you’re interested in seeing how to implement your own custom data sources, the [<font color="#0879e3"><u>Cassandra Connector</u></font>](https://github.com/datastax/spark-cassandra-connector) is well organized and maintained and could provide a reference for the adventurous. 

在本章中，我们讨论了可用于在Spark中读写数据的各种选项。这几乎涵盖了您作为Spark的日常用户需要了解的所有内容。出于好奇，有几种方法可以实现您自己的数据源。但是，我们省略了有关如何执行此操作的说明，因为API正在不断发展以更好地支持结构化流。如果您有兴趣了解如何实现自己的自定义数据源，则[<font color="#0879e3"><u>Cassandra Connector</u></font>](https://github.com/datastax/spark-cassandra-connector)的组织和维护良好，可以为喜欢冒险的人提供参考。

In Chapter 10, we discuss Spark SQL and how it interoperates with everything else we’ve seen so far in the Structured APIs.

在第10章中，我们将讨论Spark SQL以及它如何与到目前为止在结构化API中看到的所有其他事物进行交互。 

