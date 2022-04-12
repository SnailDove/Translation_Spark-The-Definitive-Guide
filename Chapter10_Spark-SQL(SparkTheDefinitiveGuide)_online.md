---
title: 翻译 Chapter 10. Spark SQL
date: 2019-10-20
copyright: true
categories: English
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 10. Spark SQL
<center><strong>译者</strong>：<u><a style="color:#0879e3" href="https://snaildove.github.io">https://snaildove.github.io</a></u></center>

Spark SQL is arguably one of the most important and powerful features in Spark. This chapter introduces the core concepts in Spark SQL that you need to understand. This chapter will not rewrite the ANSI-SQL specification or enumerate every single kind of SQL expression. If you read any other parts of this book, you will notice that we try to include SQL code wherever we include DataFrame code to make it easy to cross-reference with code samples. Other examples are available in the appendix and reference sections.

Spark SQL可以说是Spark中最重要和最强大的功能之一。本章介绍您需要了解的Spark SQL核心概念。本章将不会重写ANSI-SQL规范或枚举每种SQL表达式。如果您阅读本书的其他部分，将会发现我们尝试在包含DataFrame代码的任何地方都包含SQL代码，以便于与代码示例进行交叉引用。附录和参考部分提供了其他示例。

In a nutshell, with Spark SQL you can run SQL queries against views or tables organized into databases. You also can use system functions or define user functions and analyze query plans in order to optimize their workloads. This integrates directly into the DataFrame and Dataset API, and as we saw in previous chapters, you can choose to express some of your data manipulations in SQL and others in DataFrames and they will compile to the same underlying code.

简而言之，使用Spark SQL，您可以对组织到数据库中的视图或表运行SQL查询。您还可以使用系统函数或定义用户函数并分析查询计划，以优化其工作量。它直接集成到DataFrame和Dataset API中，正如我们在前几章中所看到的，您可以选择在SQL中表达某些数据操作，在DataFrames中表达其他数据操作，它们将编译为相同的基础代码。

## <font color="#9a161a">What Is SQL?</font>

SQL or Structured Query Language is a domain-specific language for expressing relational operations over data. It is used in all relational databases, and many “NoSQL” databases create their SQL dialect in order to make working with their databases easier. SQL is everywhere, and even though tech pundits prophesized its death, it is an extremely resilient data tool that many businesses depend on. Spark implements a subset of ANSI SQL:2003. This SQL standard is one that is available in the majority of SQL databases and this support means that Spark successfully runs the popular benchmark TPC-DS.

SQL或结构化查询语言是一种特定领域的语言，用于表达对数据的关系操作。它在所有关系数据库中使用，许多“ NoSQL”数据库创建其SQL方言，以便于使用其数据库。SQL无处不在，即使技术专家预言了它的消亡，它还是许多企业所依赖的极其灵活的数据工具。Spark实现了ANSI SQL：2003的子集。此SQL标准是大多数SQL数据库中可用的标准，并且这种支持意味着Spark成功运行了流行的基准TPC-DS。

## <font color="#9a161a">Big Data and SQL: Apache Hive</font>

Before Spark’s rise, Hive was the de facto big data SQL access layer. Originally developed at Facebook, Hive became an incredibly popular tool across industry for performing SQL operations on big data. In many ways it helped propel Hadoop into different industries because analysts could run SQL queries. Although Spark began as a general processing engine with Resilient Distributed Datasets (RDDs), a large cohort of users now use Spark SQL.

在Spark崛起之前，Hive是事实上的大数据SQL访问层。Hive最初是在Facebook上开发的，已成为行业内非常流行的工具，用于对大数据执行SQL操作。它以多种方式帮助将Hadoop推向不同的行业，因为分析师可以运行SQL查询。尽管Spark最初是使用弹性分布式数据集（RDD）作为通用处理引擎，但现在有大量用户使用Spark SQL。

## <font color="#9a161a">Big Data and SQL: Spark SQL</font>

With the release of Spark 2.0, its authors created a superset of Hive’s support, writing a native SQL parser that supports both ANSI-SQL as well as HiveQL queries. This, along with its unique interoperability with DataFrames, makes it a powerful tool for all sorts of companies. For example, in late 2016, Facebook announced that it had begun running Spark workloads and seeing large benefits in doing so. In the words of the blog post’s authors:

随着Spark 2.0的发布，其作者创建了Hive支持的超集，编写了支持ANSI-SQL和HiveQL查询的本地SQL解析器。这以及它与DataFrames的独特互操作性，使其成为各种公司的强大工具。例如，在2016年末，Facebook宣布它已开始运行Spark工作负载，并看到这样做有很大的好处。用博客文章作者的话来说：

>*We challenged Spark to replace a pipeline that decomposed to hundreds of Hive jobs into a single Spark job. Through a series of performance and reliability improvements, we were able to scale Spark to handle one of our entity ranking data processing use cases in production…. The Spark-based pipeline produced significant performance improvements (4.5–6x CPU, 3–4x resource reservation, and ~5x latency) compared with the old Hive-based pipeline, and it has been running in production for several months.*
>
>我们向Spark提出挑战，要求将分解成数百个Hive作业的管道替换为单个Spark作业。通过一系列的性能和可靠性改进，我们能够扩展Spark以处理生产中我们对实体排名数据处理用例之一的需求。与基于Hive的旧管道相比，基于Spark的管道显着提高了性能（4.5-6倍CPU，3-4倍资源预留和约5倍延迟），并且已经在生产中运行了几个月。

The power of Spark SQL derives from several key facts: SQL analysts can now take advantage of Spark’s computation abilities by plugging into the Thrift Server or Spark’s SQL interface, whereas data engineers and scientists can use Spark SQL where appropriate in any data flow. This unifying API allows for data to be extracted with SQL, manipulated as a DataFrame, passed into one of Spark MLlibs’ large-scale machine learning algorithms, written out to another data source, and everything in between.

Spark SQL的强大功能来自几个关键事实：现在，SQL分析师可以通过插入Thrift Server或Spark的SQL接口来利用Spark的计算能力，而数据工程师和科学家可以在任何数据流中酌情使用Spark SQL。这个统一的API允许使用SQL提取数据，将其作为DataFrame进行操作，传递到Spark MLlibs的大型机器学习算法之一中，写出到另一个数据源中，以及介于两者之间的所有内容。

---

<p><center><font face="constant-width" color="#737373" size=3><strong>NOTE 注意</strong></font></center></P>
Spark SQL is intended to operate as an online analytic processing (OLAP) database, not an online transaction processing (OLTP) database. This means that it is not intended to perform extremely low-latency queries. Even though support for in-place modifications is sure to be something that comes up in the future, it’s not something that is currently available.

Spark SQL旨在用作在线分析处理（OLAP）数据库，而不是在线事务处理（OLTP）数据库。这意味着它不打算执行极低延迟的查询。即使将来肯定会支持就地修改，但目前尚不支持。

----

### <font color="#00000">Spark’s Relationship to Hive</font>

Spark SQL has a great relationship with Hive because it can connect to Hive metastores. The Hive metastore is the way in which Hive maintains table information for use across sessions. With Spark SQL, you can connect to your Hive metastore (if you already have one) and access table metadata to reduce file listing when accessing information. This is popular for users who are migrating from a legacy Hadoop environment and beginning to run all their workloads using Spark.

Spark SQL与Hive有着密切的关系，因为它可以连接到Hive元存储。Hive元存储库是Hive维护表信息以供跨会话使用的方式。使用Spark SQL，您可以连接到Hive元存储（如果已经拥有一个）并访问表元数据以减少访问信息时的文件列表。非常受遗留的Hadoop环境迁移并开始使用Spark运行其所有工作负载的用户欢迎。

#### <font color="#3399cc">The Hive metastore</font>

To connect to the Hive metastore, there are several properties that you’ll need. First, you need to set the Metastore version (`spark.sql.hive.metastore.version`) to correspond to the proper Hive metastore that you’re accessing. By default, this value is 1.2.1. You also need to set `spark.sql.hive.metastore.jars` if you’re going to change the way that the HiveMetastoreClient is initialized. Spark uses the default versions, but you can also specify Maven repositories or a classpath in the standard format for the Java Virtual Machine (JVM). In addition, you might need to supply proper class prefixes in order to communicate with different databases that store the Hive metastore. You’ll set these as shared prefixes that both Spark and Hive will share (`spark.sql.hive.metastore.sharedPrefixes`).

要连接到 Hive Metastore，您需要几个属性。首先，您需要将 Metastore 版本（`spark.sql.hive.metastore.version`）设置为与您正在访问的正确的 Hive Metastore 相对应。默认情况下，此值为1.2.1。如果您要更改 HiveMetastoreClient 的初始化方式，则还需要设置 `spark.sql.hive.metastore.jars` 。Spark使用默认版本，但是您也可以为Java虚拟机（JVM）以标准格式指定Maven存储库或类路径。此外，您可能需要提供适当的类前缀才能与存储Hive元存储库的其他数据库进行通信。您将这些设置为Spark和Hive都将共享的共享前缀（`spark.sql.hive.metastore.sharedPrefixes`）。

If you’re connecting to your own metastore, it’s worth checking <u><a style="color:#0879e3" href="http://spark.apache.org/docs/latest/sql-programming-guide.html#interacting-with-different-versions-of-hive-metastore">the documentation</a></u> for further updates and more information.

如果您要连接到自己的元存储库，则值得查看<u><a style="color:#0879e3" href="http://spark.apache.org/docs/latest/sql-programming-guide.html#interacting-with-different-versions-of-hive-metastore">文档</a></u>以获取更多更新和更多信息。

## <font color="#9a161a">How to Run Spark SQL Queries</font>

Spark provides several interfaces to execute SQL queries.

Spark提供了几个接口来执行SQL查询。

### <font color="#00000">Spark SQL CLI  </font>

The Spark SQL CLI is a convenient tool with which you can make basic Spark SQL queries in local mode from the command line. Note that the Spark SQL CLI cannot communicate with the Thrift JDBC server. To start the Spark SQL CLI, run the following in the Spark directory:

Spark SQL CLI是一种方便的工具，您可以使用它从命令行在本地模式下进行基本的Spark SQL查询。请注意，Spark SQL CLI无法与Thrift JDBC服务器通信。要启动Spark SQL CLI，请在Spark目录中运行以下命令：

```scala
./bin/spark-sql
```


You configure Hive by placing your *hive-site.xml*, *core-site.xml*, and *hdfs-site.xml* files in *conf/*. For a complete list of all available options, you can run `./bin/spark-sql --help`.

您可以通过将 hive-site.xml，core-site.xml 和 hdfs-site.xml 文件放在 conf/ 中来配置Hive。有关所有可用选项的完整列表，可以运行`./bin/spark-sql --help`。

### <font color="#00000">Spark’s Programmatic SQL Interface</font>

In addition to setting up a server, you can also execute SQL in an ad hoc manner via any of Spark’s language APIs. You can do this via the method sql on the SparkSession object. This returns a DataFrame, as we will see later in this chapter. For example, in Python or Scala, we can run the following:

除了设置服务器之外，您还可以通过任意Spark语言API以临时方式执行SQL。您可以通过SparkSession对象上的sql方法执行此操作。这将返回一个DataFrame，我们将在本章后面看到。例如，在Python或Scala中，我们可以运行以下命令：

 ```scala
spark.sql("SELECT 1 + 1").show()
 ```

The command `spark.sql("SELECT 1 + 1")` returns a DataFrame that we can then evaluate programmatically. Just like other transformations, this will not be executed eagerly but lazily. This is an immensely powerful interface because there are some transformations that are much simpler to express in SQL code than in DataFrames.

命令 `spark.sql("SELECT 1 + 1")` 返回一个DataFrame，然后我们可以通过编程对其求值。就像其他转换一样，这不会急于执行，而是懒惰地执行。这是一个非常强大的接口，因为在SQL代码中表达的某些转换比在DataFrames中表达的转换要简单得多。

You can express multiline queries quite simply by passing a multiline string into the function. For example, you could execute something like the following code in Python or Scala:

通过将多行字符串传递到函数中，可以非常简单地表达多行查询。例如，您可以在Python或Scala中执行类似以下代码的操作：

```scala
spark.sql("""SELECT user_id, department, first_name FROM professors WHERE department IN
(SELECT name FROM department WHERE created_date >= '2016-01-01')""")
```

Even more powerful, you can completely interoperate between SQL and DataFrames, as you see fit. For instance, you can create a DataFrame, manipulate it with SQL, and then manipulate it again as a DataFrame. It’s a powerful abstraction that you will likely find yourself using quite a bit:

更加强大的是，您可以根据需要在SQL和DataFrame之间完全进行互操作。例如，您可以创建一个DataFrame，使用SQL对其进行操作，然后再次将其作为DataFrame进行操作。这是一个强大的抽象，您可能会发现自己经常使用：

```scala
// in Scala
spark.read.json("/data/flight-data/json/2015-summary.json")
.createOrReplaceTempView("some_sql_view") // DF => SQL
spark.sql("""
SELECT DEST_COUNTRY_NAME, sum(count)FROM some_sql_view GROUP BY DEST_COUNTRY_NAME
""")
.where("DEST_COUNTRY_NAME like 'S%'").where("`sum(count)` > 10")
.count() // SQL => DF
```

```python
# in Python
spark.read.json("/data/flight-data/json/2015-summary.json")\
.createOrReplaceTempView("some_sql_view") # DF => SQL
spark.sql("""
SELECT DEST_COUNTRY_NAME, sum(count)
FROM some_sql_view GROUP BY DEST_COUNTRY_NAME
""")\
.where("DEST_COUNTRY_NAME like 'S%'").where("`sum(count)` > 10")\
.count() # SQL => DF
```

### <font color="#00000">SparkSQL Thrift JDBC/ODBC Server</font>

Spark provides a Java Database Connectivity (JDBC) interface by which either you or a remote program connects to the Spark driver in order to execute Spark SQL queries. A common use case might be a for a business analyst to connect business intelligence software like Tableau to Spark. The Thrift JDBC/Open Database Connectivity (ODBC) server implemented here corresponds to the HiveServer2 in Hive 1.2.1. You can test the JDBC server with the beeline script that comes with either Spark or Hive 1.2.1.

Spark提供了Java数据库连接（JDBC）接口，可通过该接口，您自己或远程程序连接到Spark驱动程序以执行Spark SQL查询。对于业务分析师来说，一个常见的用例可能是将Tableau之类的商业智能软件连接到Spark。此处实现的Thrift JDBC/开放数据库连接（ODBC）服务器对应于Hive 1.2.1中的HiveServer2。您可以使用Spark或Hive 1.2.1附带的beeline脚本测试JDBC服务器。

To start the JDBC/ODBC server, run the following in the Spark directory : 

要启动 JDBC/ODBC 服务器，请在Spark目录中运行以下命令：

```shell
./sbin/start-thriftserver.sh
```

This script accepts all `bin/spark-submit` command-line options. To see all available options for configuring this Thrift Server, run `./sbin/start-thriftserver.sh --help`. By default, the server listens on localhost:10000. You can override this through environmental variables or system properties

该脚本接受所有 `bin/spark-submit` 命令行选项。要查看用于配置此Thrift Server的所有可用选项，请运行`./sbin/start-thriftserver.sh --help`。默认情况下，服务器在 localhost:10000 上侦听。您可以通过环境变量或系统属性来覆盖它。

For environment configuration, use this:

对于环境配置，请使用以下命令：

```shell
export HIVE_SERVER2_THRIFT_PORT=<listening-port>
export HIVE_SERVER2_THRIFT_BIND_HOST=<listening-host>
./sbin/start-thriftserver.sh \
--master <master-uri> \
...
```

For system properties: 

对于系统属性：

```shell
./sbin/start-thriftserver.sh \
--hiveconf hive.server2.thrift.port=<listening-port> \
--hiveconf hive.server2.thrift.bind.host=<listening-host> \
--master <master-uri>
...
```

You can then test this connection by running the following commands:

然后，您可以通过运行以下命令来测试此连接：

```shell
./bin/beeline
beeline> !connect jdbc:hive2://localhost:10000
```

Beeline will ask you for a username and password. In nonsecure mode, simply type the username on your machine and a blank password. For secure mode, follow the instructions given in the <u><a style="color:#0879e3" href="https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients">beeline documentation</a></u>.

Beeline会要求您提供用户名和密码。在非安全模式下，只需在计算机上键入用户名和空白密码即可。对于安全模式，请遵循 beeline <u><a style="color:#0879e3" href="https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients">文档</a></u>中给出的说明。

## <font color="#9a161a">Catalog</font>

The highest level abstraction in Spark SQL is the Catalog. The Catalog is an abstraction for the storage of metadata about the data stored in your tables as well as other helpful things like databases, tables, functions, and views. The catalog is available in the `org.apache.spark.sql.catalog.Catalog` package and contains a number of helpful functions for doing things like listing tables, databases, and functions. We will talk about all of these things shortly. It’s very self-explanatory to users, so we will omit the code samples here but it’s really just another programmatic interface to Spark SQL. This chapter shows only the SQL being executed; thus, if you’re using the programmatic interface, keep in mind that you need to wrap everything in a `spark.sql` function call to execute the relevant code.

Spark SQL中最高级别的抽象是Catalog。Catalog是用于存储相关表中存储的数据以及其他有用信息（如数据库，表，函数和视图）的元数据的抽象。该目录可在`org.apache.spark.sql.catalog.Catalog`包中找到，并包含许多有用的函数，用于执行诸如列出表，数据库和函数之类的操作。我们很快将讨论所有这些事情。这对用户来说不言自明，因此我们在这里省略了代码示例，但实际上它只是Spark SQL的另一个编程接口。本章仅显示正在执行的SQL。因此，如果您使用的是编程接口，请记住，您需要将所有内容包装在`spark.sql`函数调用中以执行相关代码。

## <font color="#9a161a">Tables</font>

To do anything useful with Spark SQL, you first need to define tables. Tables are logically equivalent to a DataFrame in that they are a structure of data against which you run commands. We can join tables, filter them, aggregate them, and perform different manipulations that we saw in previous chapters. The core difference between tables and DataFrames is this: you define DataFrames in the scope of a programming language, whereas you define tables within a database. This means that when you create a table (assuming you never changed the database), it will belong to the default database. We discuss databases more fully later on in the chapter.

为了对Spark SQL做任何有用的事情，您首先需要定义表。表在逻辑上等效于DataFrame，因为它们是运行命令所依据的数据结构。我们可以联接表，对其进行过滤，对其进行汇总，并执行在上一章中看到的不同操作。表和DataFrames之间的核心区别在于：您可以在编程语言范围内定义DataFrames，而可以在数据库中定义表。这意味着在创建表时（假设您从未更改过数据库），该表将属于默认数据库。我们将在本章后面更全面地讨论数据库。 

An important thing to note is that in Spark 2.X, tables always contain data. There is no notion of a temporary table, only a view, which does not contain data. This is important because if you go to drop a table, you can risk losing the data when doing so.

需要注意的重要一点是，在Spark 2.X中，表始终包含数据。没有临时表的概念，只有一个不包含数据的视图。这很重要，因为如果要删除表，则可能会丢失数据。

### <font color="#00000">Spark-Managed Tables</font>

One important note is the concept of managed versus unmanaged tables. Tables store two important pieces of information. The data within the tables as well as the data about the tables; that is, the metadata. You can have Spark manage the metadata for a set of files as well as for the data. When you define a table from files on disk, you are defining an unmanaged table. When you use `saveAsTable` on a DataFrame, you are creating a managed table for which Spark will track of all of the relevant information.

重要说明之一是托管表与非托管表的概念。表存储两个重要的信息。表中的数据以及有关表的数据；即元数据。您可以让Spark管理一组文件和数据的元数据。当您从磁盘上的文件定义表时，就是在定义非托管表。在DataFrame上使用 `saveAsTable` 时，您将创建一个托管表，Spark会为其跟踪所有相关信息。

This will read your table and write it out to a new location in Spark format. You can see this reflected in the new explain plan. In the explain plan, you will also notice that this writes to the default Hive warehouse location. You can set this by setting the `spark.sql.warehouse.dir` configuration to the directory of your choosing when you create your `SparkSession`. By default Spark sets this to `/user/hive/warehouse`:  

这将读取您的表并将其以Spark格式写到新位置。您可以在新的计划说明（explain plan）中看到这一点。在计划说明（explain plan）中，您还将注意到这将写入默认的Hive仓库位置。您可以通过在创建`SparkSession`时将`spark.sql.warehouse.dir`配置设置为所选目录来进行设置。默认情况下，Spark将其设置为 `/user/hive/warehouse`：

you can also see tables in a specific database by using the query `show tables IN databaseName`,  where  `databaseName` represents the name of the database that you want to query.

您还可以使用查询：`show tables IN databaseName` 查看特定数据库中的表，其中 `databaseName` 代表要查询的数据库的名称。

If you are running on a new cluster or local mode, this should return zero results.

如果您在新的集群或本地模式上运行，则应返回零结果。

### <font color="#00000">Creating Tables</font>

You can create tables from a variety of sources. Something fairly unique to Spark is the capability of reusing the entire Data Source API within SQL. This means that you do not need to define a table and then load data into it; Spark lets you create one on the fly. You can even specify all sorts of sophisticated options when you read in a file. For example, here’s a simple way to read in the flight data we worked with in previous chapters:

您可以从多种来源创建表。Spark相当独特的功能是可以在SQL中重用整个数据源API。这意味着您无需定义表然后再将数据加载到表中。Spark可让您即时创建一个。读取文件时，甚至可以指定各种复杂的选项。例如，这是一种读取我们先前章节中使用的航班数据的简单方法：

```SPARQL
CREATE TABLE flights (
DEST_COUNTRY_NAME STRING, ORIGIN_COUNTRY_NAME STRING, count LONG)
USING JSON OPTIONS (path '/data/flight-data/json/2015-summary.json')
```

<div style="background:#f7f7f7">
<p><center><strong><font face="constant-width" color="#000000" size=3>USING AND STORED AS</font></strong></center><p>
    The specification of the USING syntax in the previous example is of significant importance. If you do not specify the format, Spark will default to a Hive SerDe configuration. This has performance implications for future readers and writers because Hive SerDes are much slower than Spark’s native serialization. Hive users can also use the STORED AS syntax to specify that this should be a Hive table.<br /><br />前面示例中的USING语法规范非常重要。如果未指定格式，Spark将默认为Hive SerDe配置。由于Hive SerDes比Spark的本地序列化要慢得多，因此这对将来的读取器（reader）和写入器（writer）都有性能影响。Hive用户还可以使用STORED AS语法来指定此表应为Hive表。
</div>

You can also add comments to certain columns in a table, which can help other developers understand the data in the tables:

您还可以将注释添加到表中的某些列，这可以帮助其他开发人员理解表中的数据：

```SPARQL
CREATE TABLE flights_csv (
DEST_COUNTRY_NAME STRING,
ORIGIN_COUNTRY_NAME STRING COMMENT "remember, the US will be most prevalent",
count LONG)
USING csv OPTIONS (header true, path '/data/flight-data/csv/2015-summary.csv')
```

It is possible to create a table from a query as well : 

也可以通过查询创建表：

```SPARQL
CREATE TABLE flights_from_select USING parquet AS SELECT * FROM flights
```

In addition, you can specify to create a table only if it does not currently exist:

此外，您可以指定仅在当前不存在的情况下创建表：

----

<p><center><font face="constant-width" color="#737373" size=3><strong>NOTE 注意</strong></font></center></P>
In this example, we are creating a Hive-compatible table because we did not explicitly specify the format via USING. We can also do the following

在此示例中，我们将创建一个兼容Hive的表，因为我们没有通过USING明确指定格式。我们还可以执行以下操作

```SPARQL
CREATE TABLE IF NOT EXISTS flights_from_select
AS SELECT * FROM flights
```

---

Finally, you can control the layout of the data by writing out a partitioned dataset, as we saw in Chapter 9:

最后，您可以通过写出分区的数据集来控制数据的布局，如我们在第9章中所看到的：

```SPARQL
CREATE TABLE partitioned_flights USING parquet PARTITIONED BY (DEST_COUNTRY_NAME)
AS SELECT DEST_COUNTRY_NAME, ORIGIN_COUNTRY_NAME, count FROM flights LIMIT 5
```

These tables will be available in Spark even through sessions; temporary tables do not currently exist in Spark. You must create a temporary view, which we demonstrate later in this chapter.

这些表甚至可以通过会话在Spark中使用；临时表目前在Spark中不存在。您必须创建一个临时视图，我们将在本章稍后进行演示。

### <font color="#00000">Creating External Tables</font>

As we mentioned in the beginning of this chapter, Hive was one of the first big data SQL systems, and Spark SQL is completely compatible with Hive SQL (HiveQL) statements. One of the use cases that you might encounter is to port your legacy Hive statements to Spark SQL. Luckily, you can, for the most part, just copy and paste your Hive statements directly into Spark SQL. For example, in the example that follows, we create an unmanaged table. Spark will manage the table’s metadata; however, the files are not managed by Spark at all. You create this table by using the CREATE EXTERNAL TABLE statement.

如本章开头所述，Hive是最早的大数据SQL系统之一，Spark SQL与Hive SQL（HiveQL）语句完全兼容。您可能会遇到的一种使用情况是将旧的Hive语句移植到Spark SQL。幸运的是，在大多数情况下，您只需将Hive语句直接复制并粘贴到Spark SQL中即可。例如，在下面的示例中，我们创建一个 非托管表。Spark将管理表格的元数据；但是，这些文件完全不受Spark管理。通过使用CREATE EXTERNAL TABLE语句创建此表。

You can view any files that have already been defined by running the following command:

您可以通过运行以下命令来查看任何已定义的文件：

```SPARQL
CREATE EXTERNAL TABLE hive_flights (
DEST_COUNTRY_NAME STRING, ORIGIN_COUNTRY_NAME STRING, count LONG)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LOCATION '/data/flight-data-hive/'
```

You can also create an external table from a select clause: 

您还可以从select子句创建外部表：

```SPARQL
CREATE EXTERNAL TABLE hive_flights_2
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','LOCATION '/data/flight-data-hive/' AS SELECT * FROM flights
```

### <font color="#00000">Inserting into Tables</font>

Insertions follow the standard SQL syntax:

插入遵循标准的SQL语法：

```SPARQL
INSERT INTO flights_from_select
SELECT DEST_COUNTRY_NAME, ORIGIN_COUNTRY_NAME, count FROM flights LIMIT 20
```

You can optionally provide a partition specification if you want to write only into a certain partition. Note that a write will respect a partitioning scheme, as well (which may cause the above query to run quite slowly); however, it will add additional files only into the end partitions:

如果您只想写入某个分区，则可以选择提供分区规范。注意，写操作也将遵循分区方案（这可能导致上述查询运行得很慢）。但是，它只会将其他文件添加到最终分区中：

```SPARQL
INSERT INTO partitioned_flights
PARTITION (DEST_COUNTRY_NAME="UNITED STATES")
SELECT count, ORIGIN_COUNTRY_NAME FROM flights
WHERE DEST_COUNTRY_NAME='UNITED STATES' LIMIT 12
```

### <font color="#00000">Describing Table Metadata</font>

We saw earlier that you can add a comment when creating a table. You can view this by describing the table metadata, which will show us the relevant comment:

前面我们看到，您可以在创建表时添加注释。您可以通过描述表的元数据来查看此信息，这将向我们显示相关注释：

 ```SPARQL
DESCRIBE TABLE flights_csv
 ```

You can also see the partitioning scheme for the data by using the following (note, however, that this works only on partitioned tables):

您还可以通过使用以下内容查看数据的分区方案（但是请注意，这仅适用于分区表）：

```sql
SHOW PARTITIONS partitioned_flights
```

### <font color="#00000">Refreshing Table Metadata</font>

Maintaining table metadata is an important task to ensure that you’re reading from the most recent set of data. There are two commands to refresh table metadata. REFRESH TABLE refreshes all cached entries (essentially, files) associated with the table. If the table were previously cached, it would be cached lazily the next time it is scanned:

维护表元数据是一项重要的任务，以确保您正在从最新的数据集中进行读取。有两个命令可以刷新表元数据。REFRESH TABLE刷新与该表关联的所有缓存条目（实质上是文件）。如果该表先前已被缓存，则下次扫描时将被延迟缓存：

```sql
REFRESH table partitioned_flights
```

Another related command is REPAIR TABLE, which refreshes the partitions maintained in the catalog for that given table. This command’s focus is on collecting new partition information—an example might be writing out a new partition manually and the need to repair the table accordingly:

另一个相关的命令是REPAIR TABLE，它刷新给定表在目录中维护的分区。该命令的重点是收集新的分区信息——例如，可能是手动写出新分区，并且需要相应地修复表：

```sql
MSCK REPAIR TABLE partitioned_flights
```

### <font color="#00000">Dropping Tables</font>

You cannot delete tables: you can only “drop” them. You can drop a table by using the DROP keyword. If you drop a managed table (e.g., flights_csv), both the data and the table definition will be removed:

您不能 delete 表：只能“drop”它们。您可以使用DROP关键字删除表。如果您删除托管表（例如，flights_csv），则数据和表定义都将被删除：

```SPARQL
DROP TABLE flights_csv;
```

---

<p><center><font face="constant-width" color="#c67171" size=3><strong>WARNING 警告</strong></font></center></p>
Dropping a table deletes the data in the table, so you need to be very careful when doing this.

删除表会删除表中的数据，因此在执行此操作时需要非常小心。

---

If you try to drop a table that does not exist, you will receive an error. To only delete a table if it already exists, use `DROP TABLE IF EXISTS`.

如果尝试删除不存在的表，则会收到错误消息。要仅删除已存在的表，请使用`DROP TABLE IF EXISTS`。

```SPARQL
DROP TABLE IF EXISTS flights_csv;
```

----

<p><center><font face="constant-width" color="#c67171" size=3><strong>WARNING 警告</strong></font></center></p>
This deletes the data in the table, so exercise caution when doing this.

这会删除表中的数据，因此请谨慎操作。

---

#### <font color="#3399cc">Dropping unmanaged tables</font>

If you are dropping an unmanaged table (e.g., hive_flights), no data will be removed but you will no longer be able to refer to this data by the table name.

如果要删除非托管表（例如hive_flights），则不会删除任何数据，但是您将不再能够通过表名引用该数据。

### <font color="#00000">Caching Tables</font>

Just like DataFrames, you can cache and uncache tables. You simply specify which table you would like using the following syntax:

就像DataFrames一样，您可以缓存和取消缓存表。您只需使用以下语法指定要使用的表：

```SQL
CACHE TABLE flights
```

Here’s how you uncache them:

解除缓存的方法如下：

```SQL
UNCACHE TABLE FLIGHTS
```

## <font color="#9a161a">Views</font>

Now that you created a table, another thing that you can define is a view. A view specifies a set of transformations on top of an existing table—basically just saved query plans, which can be convenient for organizing or reusing your query logic. Spark has several different notions of views. Views can be global, set to a database, or per session.

现在，您已经创建了一个表，您可以定义的另一件事是视图。视图在现有表的顶部指定一组转换（基本上只是保存的查询计划），可以方便地组织或重用查询逻辑。Spark有几种不同的视图概念。视图可以是全局视图，设置为数据库视图或每个会话。

### <font color="#00000">Creating Views</font>

To an end user, views are displayed as tables, except rather than rewriting all of the data to a new location, they simply perform a transformation on the source data at query time. This might be a filter, select, or potentially an even larger GROUP BY or ROLLUP. For instance, in the following example, we create a view in which the destination is United States in order to see only those flights:

对于末端用户，视图显示为表，除了将所有数据重写到新位置之外，它们只是在查询时对源数据执行转换。这可能是一个筛选器，一个选择，甚至可能是更大的`GROUP BY` 或 `ROLLUP`。例如，在以下示例中，我们创建一个目的地为美国的视图，以便仅查看那些航班： 

```SPARQL
CREATE VIEW just_usa_view AS
SELECT * FROM flights WHERE dest_country_name = 'United States'
```

Like tables, you can create temporary views that are available only during the current session and are not registered to a database:

像表一样，您可以创建仅在当前会话期间可用且未注册到数据库的临时视图：

```SPARQL
CREATE TEMP VIEW just_usa_view_temp AS
SELECT * FROM flights WHERE dest_country_name = 'United States'
```

Or, it can be a global temp view. Global temp views are resolved regardless of database and are viewable across the entire Spark application, but they are removed at the end of the session:

或者，它可以是全局临时视图。无论使用哪种数据库，都可以解析全局临时视图，并且可以在整个Spark应用程序中查看它们，但是在会话结束时将其删除：

```SPARQL
CREATE GLOBAL TEMP VIEW just_usa_global_view_temp AS
SELECT * FROM flights WHERE dest_country_name = 'United States'

SHOW TABLES
```

You can also specify that you would like to overwrite a view if one already exists by using the keywords shown in the sample that follows. We can overwrite both temp views and regular views:

您还可以使用下面的示例中显示的关键字，指定要覆盖的视图（如果已经存在）。我们可以覆盖临时视图和常规视图：

```SPARQL
CREATE OR REPLACE TEMP VIEW just_usa_view_temp AS
SELECT * FROM flights WHERE dest_country_name = 'United States'
```

Now you can query this view just as if it were another table:

现在，您可以查询此视图，就像它是另一个表一样：

```SPARQL
SELECT * FROM just_usa_view_temp
```

A view is effectively a transformation and Spark will perform it only at query time. This means that it will only apply that filter after you actually go to query the table (and not earlier). Effectively, views are equivalent to creating a new DataFrame from an existing DataFrame.

视图实际上是一种转换，Spark仅在查询时执行。这意味着它只会在您实际查询表之后（而不是更早）才应用该过滤器。实际上，视图等效于从现有DataFrame创建新DataFrame。

In fact, you can see this by comparing the query plans generated by Spark DataFrames and Spark SQL. In DataFrames, we would write the following:

实际上，您可以通过比较Spark DataFrames和Spark SQL生成的查询计划来看到这一点。在DataFrames中，我们将编写以下内容：

```SPARQL
val flights = spark.read.format("json")
.load("/data/flight-data/json/2015-summary.json")
val just_usa_df = flights.where("dest_country_name = 'United States'")
just_usa_df.selectExpr("*").explain
```

In SQL, we would write (querying from our view) this:

在SQL中，我们将这样编写（从我们的视图中查询）：

```SPARQL
EXPLAIN SELECT * FROM just_usa_view
```

Or, equivalently:

或者，等效地：

```SPARQL
EXPLAIN SELECT * FROM flights WHERE dest_country_name = 'United States'
```

Due to this fact, you should feel comfortable in writing your logic either on DataFrames or SQL— whichever is most comfortable and maintainable for you.

由于这个事实，在DataFrames或SQL上编写逻辑时应该感到很自在——无论哪种方法对您来说都是最舒适和可维护的。

### <font color="#00000">Dropping Views</font>

You can drop views in the same way that you drop tables; you simply specify that what you intend to drop is a view instead of a table. The main difference between dropping a view and dropping a table is that with a view, no underlying data is removed, only the view definition itself : 

您可以像删除表一样删除视图。您只需指定要删除的是视图而不是表。删除视图和删除表之间的主要区别在于，使用视图时，不会删除任何基础数据，只会删除视图定义本身：

```SPARQL
DROP VIEW IF EXISTS just_usa_view;
```

## <font color="#9a161a">Databases</font>

Databases are a tool for organizing tables. As mentioned earlier, if you do not define one, Spark will use the default database. Any SQL statements that you run from within Spark (including DataFrame commands) execute within the context of a database. This means that if you change the database, any user-defined tables will remain in the previous database and will need to be queried differently.

数据库是用于组织表的工具。如前所述，如果您未定义数据库，Spark将使用默认数据库。您在Spark中运行的所有SQL语句（包括DataFrame命令）都在数据库的上下文中执行。这意味着，如果您更改数据库，则任何用户定义的表都将保留在先前的数据库中，并且需要以其他方式查询。

---

<p><center><font face="constant-width" color="#c67171" size=3><strong>WARNING 警告</strong></font></center></p>
This can be a source of confusion, especially if you’re sharing the same context or session for your coworkers, so be sure to set your databases appropriately.

这可能会引起混乱，尤其是如果您要为同事共享相同的上下文或会话时，请确保正确设置数据库。 

---

You can see all databases by using the following command:

您可以使用以下命令查看所有数据库：

```SPARQL
SHOW DATABASES
```

### <font color="#00000">Creating Databases</font>

Creating databases follows the same patterns you’ve seen previously in this chapter; however, hereyou use the CREATE DATABASE keywords:

创建数据库的方式与本章前面介绍的相同。但是，您在这里使用CREATE DATABASE关键字：

```SPARQL
CREATE DATABASE some_db
```

### <font color="#00000">Setting the Database</font>

You might want to set a database to perform a certain query. To do this, use the USE keyword followed by the database name:

您可能需要设置数据库以执行特定查询。为此，请使用USE关键字，后跟数据库名称：

```SPARQL
USE some_db
```

After you set this database, all queries will try to resolve table names to this database. Queries that were working just fine might now fail or yield different results because you are in a different database:

设置该数据库后，所有查询将尝试将表名解析为该数据库。现在，运行良好的查询可能会失败或产生不同的结果，因为您位于其他数据库中：

```SPARQL
SHOW tables
SELECT * FROM flights -- fails with table/view not found
```

However, you can query different databases by using the correct prefix:

但是，您可以使用正确的前缀来查询其他数据库：

```SPARQL
SELECT * FROM default.flights
```

You can see what database you’re currently using by running the following command:

通过运行以下命令，您可以查看当前正在使用的数据库：

```SPARQL
SELECT current_database()
```

You can, of course, switch back to the default database:

您当然可以切换回默认数据库：

```SPARQL
USE default;
```

### <font color="#00000">Dropping Databases</font>

Dropping or removing databases is equally as easy: you simply use the DROP DATABASE keyword:

删除或删除数据库同样容易：您只需使用DROP DATABASE关键字：

```SPARQL
DROP DATABASE IF EXISTS some_db;
```

## <font color="#9a161a">Select Statements</font>

Queries in Spark support the following ANSI SQL requirements (here we list the layout of the SELECT expression):

Spark中的查询支持以下ANSI SQL要求（此处列出了SELECT表达式的布局）：

 ```SPARQL
SELECT [ALL|DISTINCT] named_expression[, named_expression, ...]
    FROM relation[, relation, ...][lateral_view[, lateral_view, ...]]
    [WHERE boolean_expression]
    [aggregation [HAVING boolean_expression]]
    [ORDER BY sort_expressions]
    [CLUSTER BY expressions]
    [DISTRIBUTE BY expressions]
    [SORT BY sort_expressions]
    [WINDOW named_window[, WINDOW named_window, ...]]
    [LIMIT num_rows]

named_expression:
: expression [AS alias]

relation:
	| join_relation
	| (table_name|query|relation) [sample] [AS alias]
	: VALUES (expressions)[, (expressions), ...]
		[AS (column_name[, column_name, ...])]

expressions:
	: expression[, expression, ...]

sort_expressions:
	: expression [ASC|DESC][, expression [ASC|DESC], ...]
 ```

### <font color="#00000">case…when…then Statements</font>

Oftentimes, you might need to conditionally replace values in your SQL queries. You can do this by using a case...when...then...end style statement. This is essentially the equivalent of programmatic if statements:

通常，您可能需要有条件地替换SQL查询中的值。您可以通过使用case ... when ... then ... end 类型语句来实现。从本质上讲，这等效于程序化if语句：

```SPARQL
SELECT
	CASE WHEN DEST_COUNTRY_NAME = 'UNITED STATES' THEN 1
		 WHEN DEST_COUNTRY_NAME = 'Egypt' THEN 0
		 ELSE -1 END
FROM partitioned_flights
```

## <font color="#9a161a">Advanced Topics</font>

Now that we defined where data lives and how to organize it, let’s move on to querying it. A SQL query is a SQL statement requesting that some set of commands be run. SQL statements can define manipulations, definitions, or controls. The most common case are the manipulations, which is the focus of this book.

现在我们定义了数据的存放位置以及如何组织数据，让我们继续进行数据查询。SQL查询是一条SQL语句，它要求运行某些命令集。SQL语句可以定义操作，进行定义或定义控制流（control）。最常见的情况是操作，这是本书的重点。

### <font color="#00000">Complex Types</font>

Complex types are a departure from standard SQL and are an incredibly powerful feature that does not exist in standard SQL. Understanding how to manipulate them appropriately in SQL is essential. There are three core complex types in Spark SQL: structs, lists, and maps.

复杂类型与标准SQL背道而驰，并且是标准SQL中不存在的强大功能。了解如何在SQL中适当地操作它们至关重要。 Spark SQL中存在三种核心复杂类型：结构，列表和映射。  

#### <font color="#3399cc">Structs</font>

Structs are more akin to maps. They provide a way of creating or querying nested data in Spark. To create one, you simply need to wrap a set of columns (or expressions) in parentheses:

结构更类似于映射。它们提供了一种在Spark中创建或查询嵌套数据的方法。要创建一个，只需要将一组列（或表达式）括在括号中：

```SPARQL
CREATE VIEW IF NOT EXISTS nested_data AS
SELECT (DEST_COUNTRY_NAME, ORIGIN_COUNTRY_NAME) as country, count FROM flights
```

Now, you can query this data to see what it looks like:

现在，您可以查询此数据以查看其外观：

```SPARQL
SELECT * FROM nested_data
```

You can even query individual columns within a struct—all you need to do is use dot syntax:

您甚至可以查询结构中的各个列——您所需要做的就是使用点语法：

```SPARQL
SELECT country.DEST_COUNTRY_NAME, count FROM nested_data
```

If you like, you can also select all the subvalues from a struct by using the struct’s name and select all of the subcolumns. Although these aren’t truly subcolumns, it does provide a simpler way to think about them because we can do everything that we like with them as if they were a column:

如果愿意，您还可以使用结构的名称从结构中选择所有子值，然后选择所有子列。尽管这些并不是真正的子列，但是它确实提供了一种更简单的方式来考虑它们，因为我们可以像对待专栏一样做我们喜欢的所有事情：

```SPARQL
SELECT country.*, count FROM nested_data
```

#### <font color="#3399cc">Lists</font>

If you’re familiar with lists in programming languages, Spark SQL lists will feel familiar. There are several ways to create an array or list of values. You can use the `collect_list` function, which creates a list of values. You can also use the function `collect_set`, which creates an array without duplicate values. These are both aggregation functions and therefore can be specified only in aggregations:

如果您熟悉编程语言中的列表，Spark SQL列表将很熟悉。有几种创建数组或值列表的方法。您可以使用`collect_list`函数创建一个值列表。您还可以使用函数`collect_set`创建一个没有重复值的数组。这些都是聚合函数，因此只能在聚合中指定：

```SPARQL
SELECT DEST_COUNTRY_NAME as new_name, collect_list(count) as flight_counts,
collect_set(ORIGIN_COUNTRY_NAME) as origin_set
FROM flights GROUP BY DEST_COUNTRY_NAME
```

You can, however, also create an array manually within a column, as shown here:

但是，您也可以在列中手动创建数组，如下所示：

```SPARQL
SELECT DEST_COUNTRY_NAME, ARRAY(1, 2, 3) FROM flights
```

You can also query lists by position by using a Python-like array query syntax:

您还可以使用类似Python的数组查询语法按位置查询列表：

```SPARQL
SELECT DEST_COUNTRY_NAME as new_name, collect_list(count)[0]
FROM flights GROUP BY DEST_COUNTRY_NAME
```

You can also do things like convert an array back into rows. You do this by using the explode function. To demonstrate, let’s create a new view as our aggregation:

您还可以执行将数组转换回行的操作。您可以通过使用展开函数来实现。为了演示，让我们创建一个新的视图作为汇总：

> <center><strong>译者附</strong></center>
> explode，直译成“爆炸”不合场景，因此此处意译为：展开，如有不当，欢迎指出。

```SPARQL
CREATE OR REPLACE TEMP VIEW flights_agg AS
SELECT DEST_COUNTRY_NAME, collect_list(count) as collected_counts
FROM flights GROUP BY DEST_COUNTRY_NAME
```

Now let’s explode the complex type to one row in our result for every value in the array. The DEST_COUNTRY_NAME will duplicate for every value in the array, performing the exact opposite of the original collect and returning us to the original DataFrame:

现在，对于数组中的每个值，让我们将复杂类型展开（explode）为一行。DEST_COUNTRY_NAME将为数组中的每个值重复，执行与原始collection相反的操作，并将返回到原始DataFrame：

```SPARQL
SELECT explode(collected_counts), DEST_COUNTRY_NAME FROM flights_agg
```

### <font color="#00000">Functions</font>

In addition to complex types, Spark SQL provides a variety of sophisticated functions. You can find most of these functions in the DataFrames function reference; however, it is worth understanding how to find these functions in SQL, as well. To see a list of functions in Spark SQL, you use the SHOW FUNCTIONS statement:

除了复杂的类型，Spark SQL还提供了各种复杂巧妙的函数。您可以在DataFrames函数参考中找到大多数这些函数。但是，也值得了解如何在SQL中找到这些函数。要查看Spark SQL中的函数列表，请使用SHOW FUNCTIONS语句：

```SPARQL
SHOW FUNCTIONS
```

You can also more specifically indicate whether you would like to see the system functions (i.e., those built into Spark) as well as user functions:

您还可以更具体地指出是否要查看系统函数（即Spark内置的函数）以及用户函数：

```SPARQL
SHOW SYSTEM FUNCTIONS
```

User functions are those defined by you or someone else sharing your Spark environment. These are the same user-defined functions that we talked about in earlier chapters (we will discuss how to create them later on in this chapter):

用户函数是您或共享您的Spark环境的其他人定义的函数。这些是与我们在前几章中讨论过的用户定义函数一样的函数（我们将在本章稍后讨论如何创建它们）：

```SPARQL
SHOW USER FUNCTIONS
```

You can filter all SHOW commands by passing a string with wildcard (*) characters. Here, we can see all functions that begin with “s”:

您可以通过传递带有通配符（*）字符的字符串来过滤所有SHOW命令。在这里，我们可以看到所有以“ s”开头的函数：

```SPARQL
SHOW FUNCTIONS "s*";
```

Optionally, you can include the LIKE keyword, although this is not necessary:

（可选）您可以包括LIKE关键字，尽管这不是必需的：

```SPARQL
SHOW FUNCTIONS LIKE "collect*";
```

Even though listing functions is certainly useful, often you might want to know more about specific functions themselves. To do this, use the DESCRIBE keyword, which returns the documentation for a specific function.

即使列出函数肯定有用，但通常您可能想进一步了解特定函数本身。为此，请使用DESCRIBE关键字，该关键字返回特定函数的文档。

><p><center><font face="constant-width" color="#737373" size=3><strong>译者附</strong></font></center></P>
>例子：
>
>```SPARQL
>DESCRIBE FUNCTION collect_list
>```

#### <font color="#3399cc">User-defined functions</font>

As we saw in Chapters 3 and 4, Spark gives you the ability to define your own functions and use them in a distributed manner. You can define functions, just as you did before, writing the function in the language of your choice and then registering it appropriately:

正如我们在第3章和第4章中看到的那样，Spark使您能够定义自己的函数并以分布式方式使用它们。您可以像以前一样定义函数，以您选择的语言编写函数，然后适当地注册它：

```SPARQL
def power3(number:Double):Double = number * number * number
spark.udf.register("power3", power3(_:Double):Double)

SELECT count, power3(count) FROM flights
```

You can also register functions through the Hive CREATE TEMPORARY FUNCTION syntax.

您还可以通过Hive CREATE TEMPORARY FUNCTION语法注册函数。

### <font color="#00000">Subqueries</font>

With subqueries, you can specify queries within other queries. This makes it possible for you to specify some sophisticated logic within your SQL. In Spark, there are two fundamental subqueries. Correlated subqueries use some information from the outer scope of the query in order to supplement information in the subquery. Uncorrelated subqueries include no information from the outer scope. Each of these queries can return one (scalar subquery) or more values. Spark also includes support for predicate subqueries, which allow for filtering based on values.

使用子查询，您可以在其他查询中指定查询。这使您可以在SQL中指定一些复杂的逻辑。在Spark中，有两个基本子查询。关联子查询使用查询外部范围中的某些信息来补充子查询中的信息。不相关的子查询不包含来自外部范围的信息。这些查询中的每个查询都可以返回一个（标量子查询）或多个值。Spark还包括对谓词子查询的支持，该谓词子查询允许基于值进行过滤。

#### <font color="#3399cc">Uncorrelated predicate subqueries</font>

For example, let’s take a look at a predicate subquery. In this example, this is composed of two uncorrelated queries. The first query is just to get the top five country destinations based on the data we have:

例如，让我们看一下谓词子查询。在此示例中，这由两个不相关的查询组成。第一个查询只是根据我们拥有的数据获取前五个国家/地区的目的地：

```SPARQL
SELECT dest_country_name FROM flights
GROUP BY dest_country_name ORDER BY sum(count) DESC LIMIT 5
```

This gives us the following result:

这给我们以下结果：

```scheme
+-----------------+
|dest_country_name|
+-----------------+
|   United States |
|   Canada        |
|   Mexico        |
|   United Kingdom|
|   Japan         |
+-----------------+
```

Now we place this subquery inside of the filter and check to see if our origin country exists in that list:

现在，我们将此子查询放入过滤器中，并检查该列表中是否存在我们的原籍国：

```SPARQL
SELECT * FROM flights
WHERE origin_country_name IN (SELECT dest_country_name FROM flights
GROUP BY dest_country_name ORDER BY sum(count) DESC LIMIT 5)
```

This query is uncorrelated because it does not include any information from the outer scope of the query. It’s a query that you can run on its own.

 该查询是不相关的，因为它不包含来自查询外部范围的任何信息。您可以单独运行该查询。

#### <font color="#3399cc">Correlated predicate subqueries</font>

Correlated predicate subqueries allow you to use information from the outer scope in your inner query. For example, if you want to see whether you have a flight that will take you back from your destination country, you could do so by checking whether there is a flight that has the destination country as an origin and a flight that had the origin country as a destination:

 相关谓词子查询使您可以在内部查询中使用外部作用域中的信息。例如，如果您想查看是否有将您从目的地国家带回国的航班，则可以通过检查是否有一个以目的地国家为出发地的航班以及是否有一个将国家作为出发地的航班来进行。作为目的地：

```SPARQL
SELECT * FROM flights f1
WHERE EXISTS (SELECT 1 FROM flights f2
	WHERE f1.dest_country_name = f2.origin_country_name)
AND EXISTS (SELECT 1 FROM flights f2
	WHERE f2.dest_country_name = f1.origin_country_name)
```

EXISTS just checks for some existence in the subquery and returns true if there is a value. You can flip this by placing the NOT operator in front of it. This would be equivalent to finding a flight to a destination from which you won’t be able to return!

EXISTS只是检查子查询中是否存在，如果有值，则返回true。您可以通过将NOT运算符放在其前面来翻转它。这等同于找到飞往您将无法返回的目的地的航班！

#### <font color="#3399cc">Uncorrelated scalar queries</font>

Using uncorrelated scalar queries, you can bring in some supplemental information that you might not have previously. For example, if you wanted to include the maximum value as its own column from the entire counts dataset, you could do this:

使用不相关的标量查询，您可以引入一些以前可能没有的补充信息。例如，如果要在整个计数数据集中将最大值作为自己的列包括在内，则可以执行以下操作：

```SPARQL
SELECT *, (SELECT max(count) FROM flights) AS maximum FROM flights
```

## <font color="#9a161a">Miscellaneous Features</font>

There are some features in Spark SQL that don’t quite fit in previous sections of this chapter, so we’re going to include them here in no particular order. These can be relevant when performing optimizations or debugging your SQL code.

Spark SQL中的某些函数与本章前面的部分不太吻合，因此我们将以不特定的顺序将其包含在此处。这些在执行优化或调试SQL代码时可能是相关的。

### <font color="#00000">Configurations</font>

There are several Spark SQL application configurations, which we list in Table 10-1. You can set these either at application initialization or over the course of application execution (like we have seen with shuffle partitions throughout this book).

有几种Spark SQL应用程序配置，我们在表10-1中列出。您可以在应用程序初始化时或在应用程序执行过程中进行设置（就像我们在本书中看到的随机排序分区一样）。

Table 10-1. Spark SQL configurations 

 

| Property Name                                | Default                 | Meaning                                                      |
| -------------------------------------------- | ----------------------- | ------------------------------------------------------------ |
| spark.sql.inMemoryColumnarStorage.compressed | true                    | When set to true, Spark SQL automatically selects a compression codec for each column based on statistics of the data.<br />设置为true时，Spark SQL根据数据的统计信息自动为每一列选择一个压缩编解码器。 |
| spark.sql.inMemoryColumnarStorage.batchSize  | 10000                   | Controls the size of batches for columnar caching. Larger batch sizes can improve memory utilization and compression, but risk OutOfMemoryErrors (OOMs) when caching data.<br />控制用于列式缓存的批处理的大小。较大的批处理大小可以提高内存利用率和压缩率，但是在缓存数据时会出现OutOfMemoryErrors（OOM）。 |
| spark.sql.files.maxPartitionBytes            | 134217728<br />(128 MB) | The maximum number of bytes to pack into a single partition when reading files.<br />读取文件时打包到单个分区中的最大字节数。 |
| spark.sql.files.openCostInBytes              | 4194304<br/>(4 MB)      | The estimated cost to open a file, measured by the number of bytes that could be scanned in the same time. This is used when putting multiple files into a partition. It is better to overestimate; that way the partitions with small files will be faster than partitions with bigger files (which is scheduled first).<br />打开文件的估计成本，用可以同时扫描的字节数来衡量。将多个文件放入一个分区时使用。最好高估一下；这样，具有较小文件的分区将比具有较大文件的分区（首先安排）更快。 |
| spark.sql.broadcastTimeout                   | 300                     | Timeout in seconds for the broadcast wait time in broadcast.<br />广播中的广播等待时间超时（以秒为单位）。 |
| spark.sql.autoBroadcastJoinThreshold         | 10485760<br/>(10 MB)    | Configures the maximum size in bytes for a table that will be broadcast to all worker nodes when performing a join. You can disable broadcasting by setting this value to -1. Note that currently statistics are supported only for Hive Metastore tables for which the command ANALYZE TABLE COMPUTE STATISTICS noscan has been run.<br />配置表的最大大小（以字节为单位），该表在执行连接时将广播到所有工作程序节点。您可以通过将此值设置为-1来禁用广播。请注意，当前仅对运行了ANALYZE TABLE COMPUTE STATISTICS noscan命令的Hive Metastore表支持统计信息。 |
| spark.sql.shuffle.partitions                 | 200                     | Configures the number of partitions to use when shuffling data for joins or aggregations.<br />配置在对连接或聚集进行数据再分配时要使用的分区数。 |

### <font color="#00000">Setting Configuration Values in SQL</font>

We talk about configurations in Chapter 15, but as a preview, it’s worth mentioning how to set configurations from SQL. Naturally, you can only set Spark SQL configurations that way, but here’s how you can set shuffle partitions:

我们将在第15章中讨论配置，但是作为预览，值得一提的是如何从SQL设置配置。当然，您只能以这种方式设置Spark SQL配置，但是这里是设置数据再分配（shuffle）分区的方法：

```SQL
SET spark.sql.shuffle.partitions=20  
```

## <font color="#9a161a">Conclusion</font>

It should be clear from this chapter that Spark SQL and DataFrames are very closely related and that you should be able to use nearly all of the examples throughout this book with only small syntactical tweaks. This chapter illustrated more of the Spark SQL–related specifics. Chapter 11 focuses on a new concept: Datasets that allow for type-safe structured transformations.

从本章中应该清楚地知道，Spark SQL和DataFrames是密切相关的，并且您应该能够通过很少的语法调整就可以使用本书中几乎所有的示例。本章说明了更多与Spark SQL相关的细节。第11章关注于一个新概念：允许类型安全的结构化转换的Dataset。

 