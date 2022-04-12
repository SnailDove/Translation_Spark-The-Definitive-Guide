---
title: 翻译 Chapter 8 Joins
date: 2019-08-05
copyright: true
categories: English
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 8. Joins 连接
<center><strong>译者</strong>：<u><a style="color:#0879e3" href="https://snaildove.github.io">https://snaildove.github.io</a></u></center>

Chapter 7 covered aggregating single datasets, which is helpful, but more often than not, your Spark applications are going to bring together a large number of different datasets. For this reason, joins are an essential part of nearly all Spark workloads. Spark’s ability to talk to different data means that you gain the ability to tap into a variety of data sources across your company. This chapter covers not just what joins exist in Spark and how to use them, but some of the basic internals so that you can think about how Spark actually goes about executing the join on the cluster. This basic knowledge can help you avoid running out of memory and tackle problems that you could not solve before.

第7章介绍了聚合单个数据集的方法，这很有用，但通常，您的Spark应用程序将把大量不同的数据集组合在一起。因此，连接几乎是所有Spark工作负载中必不可少的一部分。 Spark能够处理不同数据的能力意味着您可以利用公司中的各种数据源。本章不仅涵盖Spark中存在的连接及其使用方式，还涵盖了一些基本的内部原理，以便您可以考虑Spark实际如何在集群上执行连接。这些基础知识可以帮助您避免内存不足，并解决以前无法解决的问题。

## <font color="#9a161a">Join Expressions 连接表达式</font>

A join brings together two sets of data, the *left* and the *right*, by comparing the value of one or more *keys* of the left and right and evaluating the result of a *join expression* that determines whether Spark should bring together the left set of data with the right set of data. The most common join expression, an <font face="constant-width" color="#000000" size=2>equi-join</font>, compares whether the specified keys in your left and right datasets are equal. If they are equal, Spark will combine the left and right datasets. The opposite is true for keys that do not match; Spark discards the rows that do not have matching keys. Spark also allows for much more sophsticated join policies in addition to <font face="constant-width" color="#000000" size=2>equi-joins</font>. We can even use complex types and perform something like checking whether a key exists within an array when you perform a join.

连接通过比较左右键中一个或多个键的值并评估连接表达式的结果来确定左右两个数据集，该连接表达式确定Spark是否应将左数据集与右数据集放在一起。最常见的连接表达式，equi-join，比较您左右数据集中的指定键是否相等。如果它们相等，Spark将合并左右数据集。对于不匹配的键则相反。 Spark丢弃没有匹配键的行。除 equi-join 外，Spark还允许更多复杂的连接策略。我们甚至可以使用复杂的类型并执行类似的操作，例如在执行连接时检查数组中是否存在键。

## <font color="#9a161a">Join Types 连接类型</font>

Whereas the *join expression* determines whether two rows should join, the *join type* determines what should be in the result set. There are a variety of different join types available in Spark for you to use: 

连接表达式确定是否应连接两行，连接类型确定结果集中应包含的内容。 Spark提供了多种不同的连接类型供您使用：

- Inner joins (keep rows with keys that exist in the left and right datasets)
内部连接（保持行包含在左右数据集中的键）
- Outer joins (keep rows with keys in either the left or right datasets)
外部连接（在左侧或右侧数据集中保留带有键的行）
- Left outer joins (keep rows with keys in the left dataset)
左外部连接（在左数据集中保留带有键的行）
- Right outer joins (keep rows with keys in the right dataset)
右外部连接（在右数据集中保留带有键的行）
- Left semi joins (keep the rows in the left, and only the left, dataset where the key appears in the right dataset)
左半连接（保留键在右侧数据集中出现的左侧数据集，并且仅保留左侧数据集）
- Left anti joins (keep the rows in the left, and only the left, dataset where they do not appear in the right dataset)
左反连接（保留左侧的行，并且仅保留左侧的数据集，而这些行未出现在右侧的数据集中）
- Natural joins (perform a join by implicitly matching the columns between the two datasets with the same names)
自然连接（通过隐式匹配两个具有相同名称的数据集之间的列来执行连接）
- Cross (or Cartesian) joins (match every row in the left dataset with every row in the right dataset) 
交叉（或笛卡尔）连接（将左数据集中的每一行与右数据集中的每一行匹配）

If you have ever interacted with a relational database system, or even an Excel spreadsheet, the concept of joining different datasets together should not be too abstract. Let’s move on to showing examples of each join type. This will make it easy to understand exactly how you can apply these to your own problems. To do this, let’s create some simple datasets that we can use in our examples: 

如果您曾经与关系数据库系统甚至是Excel电子表格进行过交互，那么将不同数据集连接在一起的概念就不会太抽象。让我们继续展示每种连接类型的示例。这将使您容易准确地理解如何将其应用于自己的问题。为此，让我们创建一些可在示例中使用的简单数据集：
```scala
// in Scala
val person = Seq(
(0, "Bill Chambers", 0, Seq(100)),
(1, "Matei Zaharia", 1, Seq(500, 250, 100)),
(2, "Michael Armbrust", 1, Seq(250, 100)))
.toDF("id", "name", "graduate_program", "spark_status")
val graduateProgram = Seq(
(0, "Masters", "School of Information", "UC Berkeley"),
(2, "Masters", "EECS", "UC Berkeley"),
(1, "Ph.D.", "EECS", "UC Berkeley"))
.toDF("id", "degree", "department", "school")
val sparkStatus = Seq(
(500, "Vice President"),
(250, "PMC Member"),
(100, "Contributor"))
.toDF("id", "status")
```

```python
# in Python
person = spark.createDataFrame([
(0, "Bill Chambers", 0, [100]),
(1, "Matei Zaharia", 1, [500, 250, 100]),
(2, "Michael Armbrust", 1, [250, 100])])\
.toDF("id", "name", "graduate_program", "spark_status")
graduateProgram = spark.createDataFrame([
(0, "Masters", "School of Information", "UC Berkeley"),
(2, "Masters", "EECS", "UC Berkeley"),
(1, "Ph.D.", "EECS", "UC Berkeley")])\
.toDF("id", "degree", "department", "school")
sparkStatus = spark.createDataFrame([
(500, "Vice President"),
(250, "PMC Member"),
(100, "Contributor")])\
.toDF("id", "status")
```

Next, let’s register these as tables so that we use them throughout the chapter: 

接下来，让我们将它们注册为表格，以便在本章中使用它们：

```scala
person.createOrReplaceTempView("person")
graduateProgram.createOrReplaceTempView("graduateProgram")
sparkStatus.createOrReplaceTempView("sparkStatus")
```

## <font color="#9a161a">Inner Joins 内连</font>

Inner joins evaluate the keys in both of the DataFrames or tables and include (and join together) only the rows that evaluate to true. In the following example, we join the `graduateProgram` DataFrame with the `person` DataFrame to create a new DataFrame: 

内部连接评估两个DataFrame或表中的键，并且仅包括（并连接）评估结果为true的行。 在以下示例中，我们将 `graduateProgram` DataFrame与  `person`  DataFrame一起创建一个新的DataFrame：

```scala
// in Scala
val joinExpression = person.col("graduate_program") === graduateProgram.col("id")
```

```python
# in Python
joinExpression = person["graduate_program"] == graduateProgram['id']
```

Keys that do not exist in both DataFrames will not show in the resulting DataFrame. For example, the following expression would result in zero values in the resulting DataFrame: 

两个 DataFrame 中都不存在的键将不会显示在结果 Dataframe 中。 例如，以下表达式将在结果 DataFrame 中导致零值：

```scala
// in Scala
val wrongJoinExpression = person.col("name") === graduateProgram.col("school")
```

```python
# in Python
wrongJoinExpression = person["name"] == graduateProgram["school"]
```

Inner joins are the default join, so we just need to specify our left DataFrame and join the right in the JOIN expression:

内部连接是默认连接，因此我们只需要指定左侧的DataFrame并在JOIN表达式中连接右侧：

```scala
person.join(graduateProgram, joinExpression).show()
```

```sql
-- in SQL
SELECT * FROM person JOIN graduateProgram
ON person.graduate_program = graduateProgram.id 
```


| id| name|graduate_program| spark_status| id| degree|department|...|
|---|----------------|----------------|---------------|---|-------|----------|---|
| 0| Bill Chambers| 0| [100]| 0|Masters| School...|...|
| 1| Matei Zaharia| 1|[500, 250, 100]| 1| Ph.D.| EECS|...|
| 2|Michael Armbrust| 1| [250, 100]| 1| Ph.D.| EECS|...|


We can also specify this explicitly by passing in a third parameter, the `joinType`:

我们还可以通过传入第三个参数 `joinType` 来明确指定此名称：

```scala
// in Scala
var joinType = "inner"
```

```python
# in Python
joinType = "inner"
person.join(graduateProgram, joinExpression, joinType).show()
```

```sql
-- in SQL
SELECT * FROM person INNER JOIN graduateProgram
ON person.graduate_program = graduateProgram.id 
```

| id| name|graduate_program| spark_status| id| degree| department...|
|---|----------------|----------------|---------------|---|-------|--------------|
| 0| Bill Chambers| 0| [100]| 0|Masters| School...|
| 1| Matei Zaharia| 1|[500, 250, 100]| 1| Ph.D.| EECS...|
| 2|Michael Armbrust| 1| [250, 100]| 1| Ph.D.| EECS...|


## <font color="#9a161a">Outer Joins 外连接</font>

Outer joins evaluate the keys in both of the DataFrames or tables and includes (and joins together) the rows that evaluate to true or false. If there is no equivalent row in either the left or right DataFrame, Spark will insert null:

外部连接评估两个DataFrames或表中的键，并包括（并连接在一起）评估为true或false的行。 如果左侧或右侧DataFrame中没有等效行，Spark将插入null：

```scala
joinType = "outer"
person.join(graduateProgram, joinExpression, joinType).show()
```

```sql
-- in SQL
SELECT * FROM person FULL OUTER JOIN graduateProgram
ON graduate_program = graduateProgram.id 
```

| id| name|graduate_program| spark_status| id| degree| departmen...|
|----|----------------|----------------|---------------|---|-------|-------------|
| 1| Matei Zaharia| 1|[500, 250, 100]| 1| Ph.D.| EEC...|
| 2|Michael Armbrust| 1| [250, 100]| 1| Ph.D.| EEC...|null| null| null| null| 2|Masters| EEC...|
| 0| Bill Chambers| 0| [100]| 0|Masters| School...|

## <font color="#9a161a">Left Outer Joins 左外连接</font>

Left outer joins evaluate the keys in both of the DataFrames or tables and includes all rows from the left DataFrame as well as any rows in the right DataFrame that have a match in the left DataFrame. If there is no equivalent row in the right DataFrame, Spark will insert null:

左外部连接会评估两个DataFrame或表中的键，并包括左DataFrame中的所有行以及右DataFrame中与左DataFrame中具有匹配项的所有行。 如果右侧DataFrame中没有等效的行，Spark将插入null：
```scala
joinType = "left_outer"
graduateProgram.join(person, joinExpression, joinType).show()
```

```sql
-- in SQL
SELECT * FROM graduateProgram LEFT OUTER JOIN person
ON person.graduate_program = graduateProgram.id 
```

| id| degree|department| school| id| name|graduate_program|...|
|---|-------|----------|-----------|----|----------------|----------------|---|
| 0|Masters| School...|UC Berkeley| 0| Bill Chambers| 0|...|
| 2|Masters| EECS|UC Berkeley|null| null| null|...|
| 1| Ph.D.| EECS|UC Berkeley| 2|Michael Armbrust| 1|...|
| 1| Ph.D.| EECS|UC Berkeley| 1| Matei Zaharia| 1|...|


## <font color="#9a161a">Right Outer Joins 右外连接</font>

Right outer joins evaluate the keys in both of the DataFrames or tables and includes all rows from the right DataFrame as well as any rows in the left DataFrame that have a match in the right DataFrame. If there is no equivalent row in the left DataFrame, Spark will insert null:

右外部连接会评估两个DataFrame或表中的键，并包括右DataFrame中的所有行以及左DataFrame中与右DataFrame中具有匹配项的所有行。如果左侧DataFrame中没有等效的行，Spark将插入null：

```scala
joinType = "right_outer"
person.join(graduateProgram, joinExpression, joinType).show()
```

```sql
-- in SQL
SELECT * FROM person RIGHT OUTER JOIN graduateProgram
ON person.graduate_program = graduateProgram.id 
```

| id| name|graduate_program| spark_status| id| degree| department|
|----|----------------|----------------|---------------|---|-------|------------|
| 0| Bill Chambers| 0| [100]| 0|Masters|School of...|
|null| null| null| null| 2|Masters| EECS|
| 2|Michael Armbrust| 1| [250, 100]| 1| Ph.D.| EECS|
| 1| Matei Zaharia| 1|[500, 250, 100]| 1| Ph.D.| EECS|

## <font color="#9a161a">Left Semi Joins 左半连接</font>

Semi joins are a bit of a departure from the other joins. They do not actually include any values from the right DataFrame. They only compare values to see if the value exists in the second DataFrame. If the value does exist, those rows will be kept in the result, even if there are duplicate keys in the left DataFrame. Think of left semi joins as filters on a DataFrame, as opposed to the function of a conventional join: 

半连接与其他连接有些偏离。它们实际上并不包含来自右DataFrame的任何值。他们仅比较值以查看该值是否存在于第二个DataFrame中。如果该值确实存在，则即使左侧DataFrame中存在重复的键，这些行也将保留在结果中。可以将左半连接视为DataFrame上的过滤器，这与常规连接的功能相反：

```scala
joinType = "left_semi"
graduateProgram.join(person, joinExpression, joinType).show()
```

| id| degree| department| school|
|---|-------|--------------------|-----------|
| 0|Masters|School of Informa...|UC Berkeley|
| 1| Ph.D.| EECS|UC Berkeley|

```scala
// in Scala
val gradProgram2 = graduateProgram.union(Seq(
(0, "Masters", "Duplicated Row", "Duplicated School")).toDF())
gradProgram2.createOrReplaceTempView("gradProgram2")
```

```python
# in Python
gradProgram2 = graduateProgram.union(spark.createDataFrame([
(0, "Masters", "Duplicated Row", "Duplicated School")]))
gradProgram2.createOrReplaceTempView("gradProgram2")
gradProgram2.join(person, joinExpression, joinType).show()
```

```sql
-- in SQL
SELECT * FROM gradProgram2 LEFT SEMI JOIN person
ON gradProgram2.id = person.graduate_program
```

| id| degree| department| school|
|---|-------|--------------------|-----------------|
| 0|Masters|School of Informa...| UC Berkeley|
| 1| Ph.D.| EECS| UC Berkeley|
| 0|Masters| Duplicated Row|Duplicated School|

## <font color="#9a161a">Left Anti Joins 左反连接</font>

Left anti joins are the opposite of left semi joins. Like left semi joins, they do not actually include any values from the right DataFrame. They only compare values to see if the value exists in the second DataFrame. However, rather than keeping the values that exist in the second DataFrame, they keep only the values that do not have a corresponding key in the second DataFrame. Think of anti joins as a NOT IN SQL-style filter: 

左反连接与左半连接相反。像左半连接一样，它们实际上不包括右DataFrame中的任何值。他们仅比较值以查看该值是否存在于第二个DataFrame中。但是，与其保留第二个DataFrame中存在的值，不如保留第二个DataFrame中没有相应键的值。将反连接视为NOT IN SQL样式的过滤器：

```scala
joinType = "left_anti"
graduateProgram.join(person, joinExpression, joinType).show()
```

```sql
-- in SQL
SELECT * FROM graduateProgram LEFT ANTI JOIN person
ON graduateProgram.id = person.graduate_program 
```

| id| degree|department| school|
|---|-------|----------|-----------|
| 2|Masters| EECS|UC Berkeley|

## <font color="#9a161a">Natural Joins 自然连接</font>

Natural joins make implicit guesses at the columns on which you would like to join. It finds matching columns and returns the results. Left, right, and outer natural joins are all supported.

自然连接对要连接的列进行隐式猜测。它找到匹配的列并返回结果。左，右和外部的自然连接均受到这样的支持。

---

<p><center><font face="constant-width" color="#c67171" size=3><strong>WARNING 警告</strong></font></center></p>
Implicit is always dangerous! The following query will give us incorrect results because the two DataFrames/tables share a column name (id), but it means different things in the datasets. You should always use this join with caution.

隐式总是危险的！以下查询将为我们提供不正确的结果，因为两个DataFrame或表共享一个列名（id），但这意味着数据集中的内容有所不同。您应始终谨慎使用此连接。

```sql
-- in SQL
SELECT * FROM graduateProgram NATURAL JOIN person 
```

---

## <font color="#9a161a">Cross (Cartesian) Joins 交叉（笛卡尔）连接</font>

The last of our joins are cross-joins or cartesian products. Cross-joins in simplest terms are inner joins that do not specify a predicate. Cross joins will join every single row in the left DataFrame to ever single row in the right DataFrame. This will cause an absolute explosion in the number of rows contained in the resulting DataFrame. If you have 1,000 rows in each DataFrame, the cross-join of these will result in 1,000,000 (1,000 x 1,000) rows. For this reason, you must very explicitly state that you want a cross-join by using the cross join keyword : 

我们的最后一个连接是交叉连接或笛卡尔积。 用最简单的术语来说，交叉连接是不指定谓词的内部连接。 交叉连接会将左侧DataFrame中的每一行连接到右侧DataFrame中的每一行。 这将导致结果DataFrame中包含的行数发生绝对爆炸。 如果每个DataFrame中有1,000行，则这些交叉连接将导致1,000,000（1,000 x 1,000）行。 因此，您必须使用cross join关键字非常显示地声明要进行交叉连接：

```scala
joinType = "cross"
graduateProgram.join(person, joinExpression, joinType).show()
```

```sql
-- in SQL
SELECT * FROM graduateProgram CROSS JOIN person
ON graduateProgram.id = person.graduate_program
```

| id| degree|department| school| id| name|graduate_program|spar...|
|---|-------|----------|-----------|---|----------------|----------------|-------|
| 0|Masters| School...|UC Berkeley| 0| Bill Chambers| 0| ...|
| 1| Ph.D.| EECS|UC Berkeley| 2|Michael Armbrust| 1| [2...|
| 1| Ph.D.| EECS|UC Berkeley| 1| Matei Zaharia| 1|[500...|

If you truly intend to have a cross-join, you can call that out explicitly : 

如果您确实打算进行交叉连接，则可以显示调用：

```scala
person.crossJoin(graduateProgram).show()
```

```sql
-- in SQL
SELECT * FROM graduateProgram CROSS JOIN person
```

| id| name|graduate_program| spark_status| id| degree| departm...|
|---|----------------|----------------|---------------|---|-------|-------------|
| 0| Bill Chambers| 0| [100]| 0|Masters| School...|
|...|...|...|...|...|...|...|
| 1| Matei Zaharia| 1|[500, 250, 100]| 0|Masters| School...|
|...|...|...|...|...|...|...|
| 2|Michael Armbrust| 1| [250, 100]| 0|Masters| School...|
|...|...|...|...|...|...|...|

---

<p><center><font face="constant-width" color="#c67171" size=3><strong>WARNING 警告</strong></font></center></p>
You should use cross-joins only if you are absolutely, 100 percent sure that this is the join you need. There is a reason why you need to be explicit when defining a cross-join in Spark. They’re dangerous! Advanced users can set the session-level configuration `spark.sql.crossJoin.enable` to true in order to allow cross-joins without warnings or without Spark trying to perform another join for you. 

仅在绝对必要时才应使用交叉连接，100％确保这是您需要的连接。 有一个原因为什么在Spark中定义交叉连接时需要明确： 他们很危险！ 高级用户可以将会话级别的配置 `spark.sql.crossJoin.enable` 设置为true，以允许交叉连接而不会发出警告或Spark不会尝试为您执行另一个连接。

---

## <font color="#9a161a">Challenges When Using Joins 使用连接时的挑战</font> 

When performing joins, there are some specific challenges and some common questions that arise. The rest of the chapter will provide answers to these common questions and then explain how, at a high level, Spark performs joins. This will hint at some of the optimizations that we are going to cover in later parts of this book. 

执行连接时，会出现一些特定的挑战和一些常见的问题。 本章的其余部分将提供对这些常见问题的解答，然后从较高的角度解释Spark如何执行连接。 这将暗示我们将在本书的后面部分中介绍的一些优化。


### <font color="#00000">Joins on Complex Types 连接复杂类型</font>

Even though this might seem like a challenge, it’s actually not. Any expression is a valid join expression, assuming that it returns a Boolean: 

尽管这似乎是一个挑战，但实际上并非如此。 假定它返回一个布尔值，则任何表达式都是有效的连接表达式：

```scala
import org.apache.spark.sql.functions.expr
person.withColumnRenamed("id", "personId")
.join(sparkStatus, expr("array_contains(spark_status, id)")).show()
```

```python
# in Python
from pyspark.sql.functions import expr
person.withColumnRenamed("id", "personId")\
.join(sparkStatus, expr("array_contains(spark_status, id)")).show()
```

```sql
-- in SQL
SELECT * FROM
(select id as personId, name, graduate_program, spark_status FROM person)
INNER JOIN sparkStatus ON array_contains(spark_status, id)
```

|personId| name|graduate_program| spark_status| id| status|
|--------|----------------|----------------|---------------|---|--------------|
| 0| Bill Chambers| 0| [100]|100| Contributor|
| 1| Matei Zaharia| 1|[500, 250, 100]|500|Vice President|
| 1| Matei Zaharia| 1|[500, 250, 100]|250| PMC Member|
| 1| Matei Zaharia| 1|[500, 250, 100]|100| Contributor|
| 2|Michael Armbrust| 1| [250, 100]|250| PMC Member|
| 2|Michael Armbrust| 1| [250, 100]|100| Contributor|

### <font color="#00000">Handling Duplicate Column Names 处理重复的列名</font>

One of the tricky things that come up in joins is dealing with duplicate column names in your results DataFrame. In a DataFrame, each column has a unique ID within Spark’s SQL Engine, Catalyst. This unique ID is purely internal and not something that you can directly reference. This makes it quite difficult to refer to a specific column when you have a DataFrame with duplicate column names.

连接中棘手的事情之一是处理结果DataFrame中的重复列名。 在DataFrame中，Spark的SQL引擎Catalyst中的每一列都有唯一的ID。 此唯一ID纯粹是内部的，不能直接引用。 当您的DataFrame具有重复的列名时，这使得引用特定的列变得非常困难。

This can occur in two distinct situations:

这可能在两种不同的情况下发生：

- The join expression that you specify does not remove one key from one of the input DataFrames and the keys have the same column name 
您指定的连接表达式不会从输入DataFrame之一中删除一个键，并且这些键具有相同的列名。
- Two columns on which you are not performing the join have the same name
没有执行连接的两列拥有相同的名称。 

Let’s create a problem dataset that we can use to illustrate these problems: 

让我们创建一个问题数据集，以用来说明这些问题：

```scala
val gradProgramDupe = graduateProgram.withColumnRenamed("id", "graduate_program")
val joinExpr = gradProgramDupe.col("graduate_program") === person.col(
"graduate_program")
```

Note that there are now two graduate_program columns, even though we joined on that key:

请注意，即使我们加入了该键，现在也有两个Graduate_program列：

```scala
person.join(gradProgramDupe, joinExpr).show()
```

The challenge arises when we refer to one of these columns:

当我们引用这些列之一时，挑战就出现了：

```scala
person.join(gradProgramDupe, joinExpr).select("graduate_program").show()
```

Given the previous code snippet, we will receive an error. In this particular example, Spark generates this message:

给定前面的代码片段，我们将收到一个错误。 在此特定示例中，Spark生成以下消息：

```scala
org.apache.spark.sql.AnalysisException: Reference 'graduate_program' is
ambiguous, could be: graduate_program#40, graduate_program#1079.; 
```

#### <font color="#3399cc">Approach 1: Different join expression 方法1：不同的连接表达式</font>

When you have two keys that have the same name, probably the easiest fix is to change the join expression from a Boolean expression to a string or sequence. This automatically removes one of the columns for you during the join:

当您有两个名称相同的键时，最简单的解决方法可能是将连接表达式从布尔表达式更改为字符串或序列。 这会在连接过程中自动为您删除其中一列：

```scala
person.join(gradProgramDupe,"graduate_program").select("graduate_program").show()
```

 #### <font color="#3399cc">Approach 2: Dropping the column after the join 方法2：加入后删除列</font>

Another approach is to drop the offending column after the join. When doing this, we need to refer to the column via the original source DataFrame. We can do this if the join uses the same key names or if the source DataFrames have columns that simply have the same name : 

另一种方法是在连接后删除有问题的列。这样做时，我们需要通过源DataFrame引用该列。如果连接使用相同的键名，或者源DataFrame的列仅具有相同的名称，则可以执行以下操作：

```scala
person.join(gradProgramDupe, joinExpr).drop(person.col("graduate_program"))
.select("graduate_program").show()

val joinExpr = person.col("graduate_program") === graduateProgram.col("id")
person.join(graduateProgram, joinExpr).drop(graduateProgram.col("id")).show()
```

This is an artifact of Spark’s SQL analysis process in which an explicitly referenced column will pass analysis because Spark has no need to resolve the column. Notice how the column uses the `.col` method instead of a column function. That allows us to implicitly specify that column by its specific ID.

这是Spark的SQL分析过程的产物，其中显示引用的列将传递到分析过程，因为Spark不需要解析该列。 注意该列如何使用 `.col` 方法而不是列函数。 这使我们可以通过其特定ID隐式指定该列。

#### <font color="#3399cc">Approach 3: Renaming a column before the join 方法3：在连接之前重命名列</font>

We can avoid this issue altogether if we rename one of our columns before the join:

如果我们在连接之前重命名其中一个列，则可以完全避免此问题：

```scala
val gradProgram3 = graduateProgram.withColumnRenamed("id", "grad_id")
val joinExpr = person.col("graduate_program") === gradProgram3.col("grad_id")
person.join(gradProgram3, joinExpr).show() 
```

## <font color="#9a161a">How Spark Performs Joins Spark如何执行连接</font> 

To understand how Spark performs joins, you need to understand the two core resources at play: the *node-to-node communication strategy* and *per node computation strategy*. These internals are likely irrelevant to your business problem. However, comprehending how Spark performs joins can mean the difference between a job that completes quickly and one that never completes at all.

要了解Spark如何执行连接，您需要了解两个核心资源：节点对节点通信策略和每个节点计算策略。这些内部因素可能与您的业务问题无关。但是，了解Spark如何执行连接可能意味着快速完成的工作与根本没有完成的工作之间的区别。


### <font color="#00000">Communication Strategies 通信策略</font>

Spark approaches cluster communication in two different ways during joins. It either incurs a shuffle join, which results in an all-to-all communication or a broadcast join. Keep in mind that there is a lot more detail than we’re letting on at this point, and that’s intentional. Some of these internal optimizations are likely to change over time with new improvements to the cost-based optimizer and improved communication strategies. For this reason, we’re going to focus on the high-level examples to help you understand exactly what’s going on in some of the more common scenarios, and let you take advantage of some of the low-hanging fruit that you can use right away to try to speed up some of your workloads.

在连接期间，Spark以两种不同的方式处理集群通信。它会导致数据再分配连接（shuffle join），从而导致进行所有节点之间（all-to-all）相互通信或广播连接（broadcast join）。请记住，这时我们要提供的细节比我们要多得多，这是故意的。随着基于成本的优化器的新改进和改进的通信策略，其中一些内部优化可能会随时间而变化。因此，我们将集中在高阶示例上，以帮助您准确了解某些较常见的情况下发生的事情，并让您充分利用一些可以直接用的容易的方法去尝试加快一些工作量。

The core foundation of our simplified view of joins is that in Spark you will have either a big table or a small table. Although this is obviously a spectrum (and things do happen differently if you have a “medium-sized table”), it can help to be binary about the distinction for the sake of this explanation. 

我们简化的连接视图的核心基础是，在Spark中，您将有一个大表或一个小表。尽管这显然是一个范围（如果您有“中型表”，事情的发生会有所不同），但是为了便于说明，将其区分成2个范围可能会有所帮助。

#### <font color="#3399cc">Big table–to–big table 大表与大表的连接</font>
When you join a big table to another big table, you end up with a shuffle join, such as that illustrates in Figure 8-1. 

当将一个大表连接到另一个大表时，最终将进行随机组合，如图8-1所示。

![1572709612960](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter8/Figure8.1-JoiningTwoBigTables.jpg) 

In a shuffle join, every node talks to every other node and they share data according to which node has a certain key or set of keys (on which you are joining). These joins are expensive because the network can become congested with traffic, especially if your data is not partitioned well. 

在数据再分配连接中，每个节点都与其他每个节点通信，并根据哪个节点具有某个键或一组键（要加入的键）共享数据。这些连接代价非常高，因为网络可能会变得拥塞，特别是如果您的数据没有很好地分区的时候。

This join describes taking a big table of data and joining it to another big table of data. An example of this might be a company that receives billions of messages every day from the Internet of Things, and needs to identify the day-over-day changes that have occurred. The way to do this is by joining on deviceId, messageType, and date in one column, and date - 1 day in the other column. In Figure 8-1, DataFrame 1 and DataFrame 2 are both large DataFrames. This means that all worker nodes (and potentially every partition) will need to communicate with one another during the entire join process (with no intelligent partitioning of data).

此连接描述获取一个大数据表并将其连接到另一个大数据表。例如，一家公司每天从物联网接收数十亿条消息，并且需要确定每天发生的变化，这就是一个例子。要做到这一点，方法是在一列中加入deviceId，messageType和date，在另一列中加入date-1天。在图8-1中，DataFrame 1和DataFrame 2都是大型DataFrame。这意味着所有工作节点（以及可能的每个分区）在整个连接过程中都需要相互通信（没有智能分区数据）。

#### <font color="#3399cc">Big table–to–small table 大表连接小表</font>

When the table is small enough to fit into the memory of a single worker node, with some breathing room of course, we can optimize our join. Although we can use a big table–to–big table communication strategy, it can often be more efficient to use a broadcast join. What this means is that we will replicate our small DataFrame onto every worker node in the cluster (be it located on one machine or many). Now this sounds expensive. However, what this does is prevent us from performing the all-to-all communication during the entire join process. Instead, we perform it only once at the beginning and then let each individual worker node perform the work without having to wait or communicate with any other worker node, as is depicted in Figure 8-2. 

当表足够小到融入单个工作节点的内存时，当然还有一些喘息的空间，我们可以优化连接。尽管我们可以使用大表对大表的通信策略，但使用广播连接通常会更有效。这意味着我们将把小型DataFrame复制到集群中的每个工作节点上（无论它位于一台计算机上还是多台计算机上）。现在听起来代价很高。但是，这样做会阻止我们在整个连接过程中执行所有节点之间相互的通信。相反，我们仅在开始时执行一次，然后让每个单独的工作程序节点执行工作，而不必等待或与任何其他工作程序节点通信，如图8-2所示。

![1572709687893](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter8/Figure8.2-ABroadcastJoin.jpg)

At the beginning of this join will be a large communication, just like in the previous type of join. However, immediately after that first, there will be no further communication between nodes. This means that joins will be performed on every single node individually, making CPU the biggest bottleneck. For our current set of data, we can see that Spark has automatically set this up as a broadcast join by looking at the explain plan: 

与以前的连接类型一样，此连接的开始将进行大量通信。但是，紧接在那之后，节点之间将不再有进一步的通信。这意味着连接将在每个单个节点上单独执行，这使CPU成为最大的瓶颈。对于我们当前的数据集，我们可以通过查看解释计划来看到Spark已**自动**将其设置为广播连接：

```scala
val joinExpr = person.col("graduate_program") === graduateProgram.col("id")
person.join(graduateProgram, joinExpr).explain()
```

```txt
== Physical Plan ==
*BroadcastHashJoin [graduate_program#40], [id#5....
:- LocalTableScan [id#38, name#39, graduate_progr...+- BroadcastExchange HashedRelationBroadcastMode(....
   +- LocalTableScan [id#56, degree#57, departmen....
```

With the DataFrame API, we can also explicitly give the optimizer a hint that we would like to use a broadcast join by using the correct function around the small DataFrame in question. In this example, these result in the same plan we just saw; however, this is not always the case:

借助DataFrame API，我们还可以通过对所讨论的小型DataFrame使用正确的函数，为优化程序明确提示我们要使用广播连接。在此示例中，这些结果与我们刚刚看到的计划相同；然而，这**并非总是如此**：

```scala
import org.apache.spark.sql.functions.broadcast
val joinExpr = person.col("graduate_program") === graduateProgram.col("id")
person.join(broadcast(graduateProgram), joinExpr).explain()
```

The SQL interface also includes the ability to provide *hints* to perform joins. These are not enforced, however, so the optimizer might choose to ignore them. You can set one of these hints by using a special comment syntax. `MAPJOIN`, `BROADCAST`, and `BROADCASTJOIN ` all do the same thing and are all supported : 

SQL接口还提供执行连接提示的功能。但是，这些功能不是强制性的，因此优化程序可能会选择忽略它们。您可以使用特殊的注释语法设置这些提示之一。 `MAPJOIN`，`BROADCAST `和 `BROADCASTJOIN` 都做相同的事情，并且都受支持：

```sql
-- in SQL
SELECT /*+ MAPJOIN(graduateProgram) */ * FROM person JOIN graduateProgram
ON person.graduate_program = graduateProgram.id
```

This doesn’t come for free either: if you try to broadcast something too large, you can crash your driver node (because that collect is expensive). This is likely an area for optimization in the future. 

这也不是免费的：如果您尝试广播太大的内容，可能会导致驱动程序节点崩溃（因为收集代价很高）。这可能是将来需要优化的领域。


#### <font color="#3399cc">Little table–to–little table 小表连接小表</font>

When performing joins with small tables, it’s usually best to let Spark decide how to join them. You can always force a broadcast join if you’re noticing strange behavior.

在执行小表之间的连接时，通常最好让Spark决定如何连接它们。如果您发现异常行为，可以随时强制加入广播连接。

## <font color="#9a161a">Conclusion 结论</font>

In this chapter, we discussed joins, probably one of the most common use cases. One thing we did not mention but is important to consider is if you partition your data correctly prior to a join, you can end up with much more efficient execution because even if a shuffle is planned, if data from two different DataFrames is already located on the same machine, Spark can avoid the shuffle. Experiment with some of your data and try partitioning beforehand to see if you can notice the increase in speed when performing those joins. In Chapter 9, we will discuss Spark’s data source APIs. There are additional implications when you decide what order joins should occur in. Because some joins act as filters, this can be a low-hanging improvement in your workloads, as you are guaranteed to reduce data exchanged over the network.

在本章中，我们讨论了连接（可能是最常见的用例之一）。我们没有提到但要考虑的一件事是，如果在连接之前正确地对数据进行了分区，则可以提高执行效率，因为即使数据再分配是有计划得，如果来自两个不同DataFrames的数据位于同一台机器上，Spark可以避免数据再分配。试用一些数据，然后尝试进行分区，以查看执行这些连接时是否可以注意到速度的提高。 在第9章中，我们将讨论Spark的数据源API。当您决定应按什么顺序进行连接时，还存在其他含义。由于某些连接充当过滤器，因此可以保证在网络上交换的数据减少，这对您的工作负载而言是微不足道的改进。

The next chapter will depart from user manipulation, as we’ve seen in the last several chapters, and touch on reading and writing data using the Structured APIs. 

正如我们在前几章中所看到的那样，下一章将脱离用户操作，并介绍使用结构化API读写数据。