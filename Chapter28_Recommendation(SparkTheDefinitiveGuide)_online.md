---
title: 翻译 Chapter 28 Recommendation
date: 2019-08-31
copyright: true
categories: English,中文
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;" />

# Chapter 28 Recommendation 推荐

<center><strong>译者</strong>：<u><a style="color:#0879e3" href="https://snaildove.github.io">https://snaildove.github.io</a></u></center>
The task of recommendation is one of the most intuitive. By studying people’s explicit preferences (through ratings) or implicit preferences (through observed behavior), you can make recommendations on what one user may like by drawing similarities between the user and other users, or between the products they liked and other products. Using the underlying similarities, recommendation engines can make new recommendations to other users.

推荐任务是最直观的。通过研究人们的显性偏好（通过评分）或隐含偏好（通过观察到的行为），您可以通过绘制用户与其他用户之间或他们喜欢的产品与其他产品之间的相似性来对用户可能喜欢的内容提出建议。利用潜在的相似性，推荐引擎可以向其他用户提出新的推荐。

## <font color="#9a161a">Use Cases 使用案例</font>

Recommendation engines are one of the best use cases for big data. It’s fairly easy to collect training data about users’ past preferences at scale, and this data can be used in many domains to connect users with new content. Spark is an open source tool of choice used across a variety of companies for large-scale recommendations:

推荐引擎是大数据的最佳用例之一。可以相当容易地大规模收集有关用户过去偏好的训练数据，并且可以在许多域中使用此数据来将用户与新内容连接起来。 Spark是一种开源工具，可供各种公司使用，用于大规模推荐：

### <font color="#00000">Movie recommendations 电影推荐</font>

Amazon, Netflix, and HBO all want to provide relevant film and TV content to their users. Netflix utilizes Spark, to make large scale movie recommendations to their users.

亚马逊，Netflix 和 HBO 都希望向用户提供相关的电影和电视内容。 Netflix 利用 Spark 为其用户制作大型电影推荐。

### <font color="#00000">Course recommendations 课程建议</font>

A school might want to recommend courses to students by studying what courses similar students have liked or taken. Past enrollment data makes for a very easy to collect training dataset for this task.

学校可能希望通过研究类似学生喜欢或去上的课程向学生推荐课程。过去的注册数据可以很容易地收集此任务的训练数据集。

In Spark, there is one workhorse recommendation algorithm, Alternating Least Squares (ALS). This algorithm leverages a technique called collaborative filtering, which makes recommendations based only on which items users interacted with in the past. That is, it does not require or use any additional features about the users or the items. It supports several ALS variants (e.g., explicit or implicit feedback). Apart from ALS, Spark provides Frequent Pattern Mining for finding association rules in market basket analysis. Finally, [Spark’s RDD API](http://spark.apache.org/docs/latest/mllib-collaborative-filtering.html) also includes a lower-level matrix factorization method that will not be covered in this book.

在Spark中，有一种主力推荐算法，即交替最小二乘（Alternating Least Squares: ALS）。该算法利用称为协同过滤的技术，该技术仅基于用户过去互动的商品进行推荐。也就是说，它不需要或使用关于用户或商品的任何额外特征。它支持几种ALS变体（例如，显式或隐式反馈）。除了ALS之外，Spark还提供频繁模式挖掘，用于在市场购物篮分析中查找关联规则。最后，[Spark 的 RDD API](http://spark.apache.org/docs/latest/mllib-collaborative-filtering.html) 还包括一个较低级别的矩阵分解方法，本书不会介绍。

## <font color="#9a161a">Collaborative Filtering with Alternating Least Squares 使用交替最小二乘法进行协同过滤</font>

ALS finds a -dimensional feature vector for each user and item such that the dot product of each user’s feature vector with each item’s feature vector approximates the user’s rating for that item. Therefore this only requires an input dataset of existing ratings between user-item pairs, with three columns: a user ID column, an item ID column (e.g., a movie), and a rating column. The ratings can either be explicit—a numerical rating that we aim to predict directly—or implicit—in which case each rating represents the strength of interactions observed between a user and item (e.g., number of visits to a particular page), which measures our level of confidence in the user’s preference for that item. Given this input DataFrame, the model will produce feature vectors that you can use to predict users’ ratings for items they have not yet rated.

ALS 为每个用户和商品找到一个维度特征向量，使得每个用户的特征向量与每个商品的特征向量的点积近似于该用户对该商品的评分。因此，这仅需要用户—商品对之间的现有评分的输入数据集，具有三列：用户ID列，商品ID列（例如，电影）和评分列。评分可以是显性的——我们旨在直接或隐含地预测的数字评分——或者隐形的——在这种情况下，每个评分表示在用户和商品之间观察到的互动的强度（例如，对特定页面的访问次数），这度量我们对用户对该商品的偏好的信心程度。给定此输入DataFrame，模型将生成特征向量，您可以使用这些特征向量来预测用户对尚未评分的商品的评分。 

One issue to note in practice is that this algorithm does have a preference for serving things that are very common or that it has a lot of information on. If you’re introducing a new product that no users have expressed a preference for, the algorithm isn’t going to recommend it to many people. Additionally, if new users are onboarding onto the platform, they may not have any ratings in the training set. Therefore, the algorithm won’t know what to recommend them. These are examples of what we call the <font color="#32cd32"><strong><i>cold start problem</i></strong></font>, which we discuss later on in the chapter.

在实践中需要注意的一个问题是，该算法确实倾向于提供非常常见的事物或者具有大量信息的事物。如果您正在推出一种没有用户表示偏好的新产品，该算法不会向许多人推荐它。此外，如果新用户加入平台，他们可能在训练集中没有任何评分。因此，算法将不知道推荐它们的内容。这些是我们称之为冷启动问题的例子，我们将在本章后面讨论。

In terms of scalability, one reason for Spark’s popularity for this task is that the algorithm and implementation in MLlib can scale to millions of users, millions of items, and billions of ratings.

在可扩展性方面，Spark在此任务中受欢迎的一个原因是MLlib中的算法和实现可以扩展到数百万用户，数百万项和数十亿的评分。

### <font color="#00000">Model Hyperparameters 模型超参数</font>

These are configurations that we can specify to determine the structure of the model as well as the specific collaborative filtering problem we wish to solve : 

这些是我们可以指定的配置，用于确定模型的结构以及我们希望解决的特定协同过滤问题：

<font face="constant-width" color="#000000" size=4>rank 排名（评分排序）</font>

The rank term determines the dimension of the feature vectors learned for users and items. This should normally be tuned through experimentation. The core trade-off is that by specifying too high a rank, the algorithm may overfit the training data; but by specifying a low rank, then it may not make the best possible predictions. The default value is 10.

排名（评分排序）术语确定为用户和商品学习的特征向量的维度。这通常应该通过实验来调整。核心权衡是通过指定过高的等级，算法可能过度拟合训练数据；但是通过指定低等级，则可能无法做出最佳预测。默认值为10。

<font face="constant-width" color="#000000" size=4>alpha</font>

When training on implicit feedback (behavioral observations), the alpha sets a baseline confidence for preference. This has a default of 1.0 and should be driven through experimentation.

在对隐式反馈（行为观察）进行训练时，alpha设置了偏好的基准置信度。它的默认值为1.0，应该通过实验来驱动。

<font face="constant-width" color="#000000" size=4>regParam</font>

Controls regularization to prevent overfitting. You should test out different values for the regularization parameter to find the optimal value for your problem. The default is 0.1.

控制正规化以防止过度拟合。您应该测试正则化参数的不同值，以找到问题的最优值。默认值为0.1。

<font face="constant-width" color="#000000" size=4>implicitPrefs</font>

This Boolean value specifies whether you are training on implicit (`true`) or explicit (`false`) (refer back to the preceding discussion for an explanation of the difference between explicit and implicit). This value should be set based on the data that you’re using as input to the model. If the data is based off passive endorsement of a product (say, via a click or page visit), then you should use implicit preferences. In contrast, if the data is an explicit rating (e.g., the user gave this restaurant 4/5 stars), you should use explicit preferences. Explicit preferences are the default.

此布尔值指定您是在训练隐式（`true`）还是显式（`false`）（请参阅前面的讨论，以解释显式和隐式之间的区别）。应根据您用作模型输入的数据设置此值。如果数据基于产品的被动认可（例如，通过点击或页面访问），那么您应该使用隐式偏好。相反，如果数据是显示的评分（例如，用户给这家餐厅 4/5 星），您应该使用显示的偏好。显式偏好是默认值。

<font face="constant-width" color="#000000" size=4>nonnegative</font>

If set to true, this parameter configures the model to place non-negative constraints on the leastsquares problem it solves and only return non-negative feature vectors. This can improve performance in some applications. The default value is false.

如果设置为 `true`，则此参数将模型配置为对其解决的最小二乘问题设置非负约束，并仅返回非负特征向量。这可以提高某些应用程序的性能。默认值为false。

### <font color="#00000">Training Parameters 训练参数</font>

The training parameters for alternating least squares are a bit different from those that we have seen in other models. That’s because we’re going to get more low-level control over how the data is distributed across the cluster. The groups of data that are distributed around the cluster are called blocks. Determining how much data to place in each block can have a significant impact on the time it takes to train the algorithm (but not the final result). A good rule of thumb is to aim for approximately one to five million ratings per block. If you have less data than that in each block, more blocks will not improve the algorithm’s performance.

交替最小二乘的训练参数与我们在其他模型中看到的训练参数略有不同。那是因为我们将对数据在集群中的分布方式进行更多的低级控制。围绕群集分布的数据组称为块。确定每个块中放置多少数据会对训练算法所花费的时间产生重大影响（但不是最终结果）。一个好的经验法则是每块大约有一到五百万的评分。如果数据少于每个块中的数据，则更多的块不会提高算法的性能。

<font face="constant-width" color="#000000" size=4>numUserBlocks</font>

This determines how many blocks to split the users into. The default is 10.

这决定了将用户分成多少个块。默认值为10。

<font face="constant-width" color="#000000" size=4>numItemBlocks</font>

This determines how many blocks to split the items into. The default is 10.

这决定了将商品拆分的块数。默认值为10。

<font face="constant-width" color="#000000" size=4>maxIter</font>

Total number of iterations over the data before stopping. Changing this probably won’t change your results a ton, so this shouldn’t be the first parameter you adjust. The default is 10. An example of when you might want to increase this is that after inspecting your objective history and noticing that it doesn’t flatline after a certain number of training iterations.

停止前数据的迭代总数。改变这个可能不会剧烈改变你的结果，所以这不应该是你调整的第一个参数。默认值为10。 您可能希望增加此值的一个示例是检查您的目标历史记录并注意到在一定数量的训练迭代后它不会变平。

<font face="constant-width" color="#000000" size=4>checkpointInterval</font>

Checkpointing allows you to save model state during training to more quickly recover from node failures. You can set a checkpoint directory using `SparkContext.setCheckpointDir`.

检查点允许您在训练期间保存模型状态，以便更快地从节点故障中恢复。您可以使用`SparkContext.setCheckpointDir` 设置检查点目录。

<font face="constant-width" color="#000000" size=4>seed</font>

Specifying a random seed can help you replicate your results.

指定随机种子可以帮助您复制结果。

### <font color="#00000">Prediction Parameters 预测参数</font>

Prediction parameters determine how a trained model should actually make predictions. In our case, there’s one parameter: the cold start strategy (set through `coldStartStrategy`). This setting determines what the model should predict for users or items that did not appear in the training set. The cold start challenge commonly arises when you’re serving a model in production, and new users and/or items have no ratings history, and therefore the model has no recommendation to make. It can also occur when using simple random splits as in Spark’s `CrossValidator` or `TrainValidationSplit`, where it is very common to encounter users and/or items in the evaluation set that are not in the training set. 

预测参数确定训练模型应如何实际进行预测。在我们的例子中，有一个参数：冷启动策略（通过 `coldStartStrategy` 设置）。此设置确定模型应为未出现在训练集中的用户或商品预测的内容。当您在生产中为模型提供服务时，通常会出现冷启动挑战，并且新用户和/或商品没有评分历史记录，因此该模型无需建议。当使用 Spark 的 `CrossValidator` 或  `TrainValidationSplit` 中的简单随机拆分时也会发生这种情况，在这种情况下，在评估集中遇到不在训练集中的用户和/或商品是很常见的。

By default, Spark will assign `NaN` prediction values when it encounters a user and/or item that is not present in the actual model. This can be useful because you design your overall system to fall back to some default recommendation when a new user or item is in the system. However, this is undesirable during training because it will ruin the ability for your evaluator to properly measure the success of your model. This makes model selection impossible. Spark allows users to set the `coldStartStrategy` parameter to `drop` in order to drop any rows in the DataFrame of predictions that contain `NaN` values. The evaluation metric will then be computed over the non-NaN data and will be valid. drop and nan (the default) are the only currently supported cold-start strategies.

默认情况下，Spark 会在遇到实际模型中不存在的用户和/或商品时分配 `NaN` 预测值。这可能很有用，因为您将整个系统设计为在系统中有新用户或商品时回退到某个默认建议。但是，这在训练期间是不合需要的，因为它会破坏评估者正确测量模型成功的能力。这使得模型选择不可能。 Spark 允许用户将 `coldStartStrategy` 参数设置为`drop`，以便删除包含 `NaN` 值的预测的DataFrame中的任何行。然后将根据非NaN数据计算评估度量并且该评估度量将是有效的。 drop和nan（默认值）是目前唯一支持的冷启动策略。

### <font color="#00000">Example</font>

This example will make use of a dataset that we have not used thus far in the book, the MovieLens movie rating dataset. This dataset, naturally, has information relevant for making movie recommendations. We will first use this dataset to train a model: 

此示例将使用我们迄今为止尚未使用的数据集，即 MovieLens 电影评分数据集。当然，该数据集具有与制作电影推荐相关的信息。我们将首先使用此数据集来训练模型：

```scala 
// in Scala
import org.apache.spark.ml.recommendation.ALS
val ratings = spark.read.textFile("/data/sample_movielens_ratings.txt")
.selectExpr("split(value , '::') as col")
.selectExpr(
"cast(col[0] as int) as userId",
"cast(col[1] as int) as movieId",
"cast(col[2] as float) as rating",
"cast(col[3] as long) as timestamp")
val Array(training, test) = ratings.randomSplit(Array(0.8, 0.2))
val als = new ALS()
.setMaxIter(5)
.setRegParam(0.01)
.setUserCol("userId")
.setItemCol("movieId")
.setRatingCol("rating")
println(als.explainParams())
val alsModel = als.fit(training)
val predictions = alsModel.transform(test)
```

```python
# in Python
from pyspark.ml.recommendation import ALS
from pyspark.sql import Row
ratings = spark.read.text("/data/sample_movielens_ratings.txt")\
.rdd.toDF()\
.selectExpr("split(value , '::') as col")\
.selectExpr(
"cast(col[0] as int) as userId",
"cast(col[1] as int) as movieId",
"cast(col[2] as float) as rating",
"cast(col[3] as long) as timestamp")
training, test = ratings.randomSplit([0.8, 0.2])
als = ALS()\.setMaxIter(5)\
.setRegParam(0.01)\
.setUserCol("userId")\
.setItemCol("movieId")\
.setRatingCol("rating")
print als.explainParams()
alsModel = als.fit(training)
predictions = alsModel.transform(test)
```

We can now output the top recommendations for each user or movie. The model’s `recommendForAllUsers` method returns a DataFrame of a `userId`, an array of recommendations, as well as a rating for each of those movies. `recommendForAllItems` returns a DataFrame of a `movieId`, as well as the top users for that movie: 

我们现在可以为每个用户或电影输出最佳推荐。该模型的 `suggestForAllUsers` 方法返回 `userId` 的`DataFrame`，推荐的数组，以及每部电影的评分。 `suggestForAllItems` 返回 `movieId` 的 `DataFrame`，以及该影片的排名靠前用户：

```scala
// in Scala
alsModel.recommendForAllUsers(10)
.selectExpr("userId", "explode(recommendations)").show()
alsModel.recommendForAllItems(10)
.selectExpr("movieId", "explode(recommendations)").show()
```

```python
# in Python
alsModel.recommendForAllUsers(10)\
.selectExpr("userId", "explode(recommendations)").show()
alsModel.recommendForAllItems(10)\
.selectExpr("movieId", "explode(recommendations)").show()
```

## <font color="#9a161a">Evaluators for Recommendation 推荐的评估器</font>

When covering the cold-start strategy, we can set up an automatic model evaluator when working with ALS. One thing that may not be immediately obvious is that this recommendation problem is really just a kind of regression problem. Since we’re predicting values (ratings) for given users, we want to optimize for reducing the total difference between our users’ ratings and the true values. We can do this using the same `RegressionEvaluator` that we saw in Chapter 27. You can place this in a pipeline to automate the training process. When doing this, you should also set the cold-start strategy to be `drop` instead of `NaN` and then switch it back to `NaN` when it comes time to actually make predictions in your production system:

在涵盖冷启动策略时，我们可以在使用 ALS 时设置自动模型评估程序。有一件事可能不是很明显，这个推荐问题实际上只是一种回归问题。由于我们正在预测给定用户的价值（评分），因此我们希望优化以减少用户评分与真实值之间的总差异。我们可以使用我们在第27章中看到的相同的 `RegressionEvaluator` 来完成此操作。您可以将其置于管道中以自动化训练过程。执行此操作时，您还应将冷启动策略设置为 `drop` 而不是 `NaN`，然后在生产系统中实际进行预测时将其切换回 `NaN`：

```scala
// in Scala
import org.apache.spark.ml.evaluation.RegressionEvaluator
val evaluator = new RegressionEvaluator()
.setMetricName("rmse")
.setLabelCol("rating")
.setPredictionCol("prediction")
val rmse = evaluator.evaluate(predictions)
println(s"Root-mean-square error = $rmse")
```

```python
# in Python
from pyspark.ml.evaluation import RegressionEvaluator
evaluator = RegressionEvaluator()\
.setMetricName("rmse")\
.setLabelCol("rating")\
.setPredictionCol("prediction")
rmse = evaluator.evaluate(predictions)
print("Root-mean-square error = %f" % rmse)
```

## <font color="#9a161a">Metrics 衡量指标</font>

Recommendation results can be measured using both the standard regression metrics and some recommendation-specific metrics. It should come as no surprise that there are more sophisticated ways of measuring recommendation success than simply evaluating based on regression. These metrics are particularly useful for evaluating your final model.

可以使用标准回归衡量指标和一些特定于推荐的指标来衡量推荐结果。毫无疑问，有更多复杂的方法来衡量推荐成功，而不仅仅是基于回归进行评估。这些指标对于评估最终模型特别有用。

### <font color="#00000">Regression Metrics 回归的衡量指标</font>

We can recycle the regression metrics for recommendation. This is because we can simply see how close each prediction is to the actual rating for that user and item:

我们可以重复利用回归指标以进行推荐。这是因为我们可以简单地看到每个预测与该用户和商品的实际评分的接近程度：

```scala
// in Scala
import org.apache.spark.mllib.evaluation.{RankingMetrics, RegressionMetrics}
val regComparison = predictions.select("rating", "prediction")
.rdd.map(x => (x.getFloat(0).toDouble,x.getFloat(1).toDouble))
val metrics = new RegressionMetrics(regComparison)
```

```python
# in Python
from pyspark.mllib.evaluation import RegressionMetrics
regComparison = predictions.select("rating", "prediction")\
.rdd.map(lambda x: (x(0), x(1)))
metrics = RegressionMetrics(regComparison)
```

### <font color="#00000">Ranking Metrics 评分的衡量指标</font>

More interestingly, we also have another tool: ranking metrics. A `RankingMetric` allows us to compare our recommendations with an actual set of ratings (or preferences) expressed by a given user. `RankingMetric` does not focus on the value of the rank but rather whether or not our algorithm recommends an already ranked item again to a user. This does require some data preparation on our part. You may want to refer to Part II for a refresher on some of the methods. First, we need to collect a set of highly ranked movies for a given user. In our case, we’re going to use a rather low threshold: movies ranked above 2.5. Tuning this value will largely be a business decision : 

更有趣的是，我们还有另一个工具：排名指标。 `RankingMetric` 允许我们将我们的推荐与给定用户表达的实际评分（或偏好）进行比较。 `RankingMetric` 不关注评分的值，而是关注我们的算法是否再次向用户推荐已经评分的商品。 这确实需要我们做一些数据准备。 您可能需要参考第二部分来了解一些方法。首先，我们需要为给定用户收集一组评分很高的电影。在我们的例子中，我们将使用相当低的门槛：电影评分高于2.5。调整此值很大程度上取决于业务决策：

```scala
// in Scala
import org.apache.spark.mllib.evaluation.{RankingMetrics, RegressionMetrics}
import org.apache.spark.sql.functions.{col, expr}
val perUserActual = predictions
.where("rating > 2.5")
.groupBy("userId")
.agg(expr("collect_set(movieId) as movies"))
```

```python
# in Python
from pyspark.mllib.evaluation import RankingMetrics, RegressionMetrics
from pyspark.sql.functions import col, expr
perUserActual = predictions\
.where("rating > 2.5")\
.groupBy("userId")\
.agg(expr("collect_set(movieId) as movies"))
```

At this point, we have a collection of users, along with a truth set of previously ranked movies for each user. Now we will get our top 10 recommendations from our algorithm on a per-user basis. We will then see if the top 10 recommendations show up in our truth set. If we have a well-trained model, it will correctly recommend the movies a user already liked. If it doesn’t, it may not have learned enough about each particular user to successfully reflect their preferences:

此时，我们有一组用户，以及对每个用户进行过评分的电影的真值集合。现在，我们将根据每个用户的算法获得我们的十大推荐。然后我们将看看前十条推荐是否出现在我们的真实集中。如果我们有一个训练有素的模型，它将正确推荐用户已经喜欢过的电影。如果没有，则可能没有充分了解每个特定用户去成功反映他们的偏好：

```scala
// in Scala
val perUserPredictions = predictions
.orderBy(col("userId"), col("prediction").desc)
.groupBy("userId")
.agg(expr("collect_list(movieId) as movies"))
```

```python
# in Python
perUserPredictions = predictions\
.orderBy(col("userId"), expr("prediction DESC"))\
.groupBy("userId")\
.agg(expr("collect_list(movieId) as movies"))
```

Now we have two DataFrames, one of predictions and another the top-ranked items for a particular user. We can pass them into the `RankingMetrics` object. This object accepts an RDD of these combinations, as you can see in the following join and RDD conversion:

现在我们有两个 DataFrame，一个是预测，另一个是特定用户评分靠前的商品。我们可以将它们传递给`RankingMetrics` 对象。此对象接受这些组合的 RDD，如以下连接和 RDD 转换中所示：

```scala 
// in Scala
val perUserActualvPred = perUserActual.join(perUserPredictions, Seq("userId")).
map(row => (
	row(1).asInstanceOf[Seq[Integer]].toArray,
	row(2).asInstanceOf[Seq[Integer]].toArray.take(15)
))
val ranks = new RankingMetrics(perUserActualvPred.rdd)
```

```python
# in Python
perUserActualvPred = perUserActual.join(perUserPredictions, ["userId"]).rdd\
.map(lambda row: (row[1], row2))
ranks = RankingMetrics(perUserActualvPred)
```

Now we can see the metrics from that ranking. For instance, we can see how precise our algorithm is with the mean average precision. We can also get the precision at certain ranking points, for instance, to see where the majority of the positive recommendations fall : 

现在我们可以看到该评分的指标。例如，我们可以看到我们的算法与平均精度的精确度。例如，我们还可以获得某些评分的精确度，以查看大多数积极建议的落点：

```scala
// in Scala
ranks.meanAveragePrecision
ranks.precisionAt(5)
```

```python
# in Python
ranks.meanAveragePrecision
ranks.precisionAt(5)
```

## <font color="#9a161a">Frequent Pattern Mining 频繁模式挖掘</font>

In addition to ALS, another tool that MLlib provides for creating recommendations is frequent pattern mining. Frequent pattern mining, sometimes referred to as market basket analysis, looks at raw data and finds association rules. For instance, given a large number of transactions it might identify that users who buy hot dogs almost always purchase hot dog buns. This technique can be applied in the recommendation context, especially when people are filling shopping carts (either on or offline). Spark implements the FP-growth algorithm for frequent pattern mining. See [the Spark documentation](https://spark.apache.org/docs/latest/ml-frequent-pattern-mining.html#fp-growth) and ESL 14.2 for more information about this algorithm.

除了 ALS 之外，MLlib 为创建推荐提供的另一个工具是频繁的模式挖掘。 频繁模式挖掘（有时称为市场购物篮分析）会查看原始数据并查找关联规则。 例如，鉴于大量交易，它可能确定购买热狗的用户几乎总是购买热狗面包。 此技术可应用于在推荐背景，尤其是当人们填充购物车（在线或离线）时。 Spark实现了用于频繁模式挖掘的 FP 增长算法。 有关此算法的更多信息，请参阅[Spark文档](https://spark.apache.org/docs/latest/ml-frequent-pattern-mining.html#fp-growth)和 ESL 14.2。

## <font color="#9a161a">Conclusion 结论</font>

In this chapter, we discussed one of Spark’s most popular machine learning algorithms in practice—alternating least squares for recommendation. We saw how we can train, tune, and evaluate this model. In the next chapter, we’ll move to unsupervised learning and discuss clustering. 

在本章中，我们讨论了Spark在实践中最受欢迎的机器学习算法之一 ——用于推荐的交替最小二乘法。 我们看到了如何训练，调整和评估这个模型。 在下一章中，我们将转向无监督学习并讨论聚类。
