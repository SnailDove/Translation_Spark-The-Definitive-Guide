---

title: 翻译 Chapter 27 Regression
date: 2019-09-01
copyrig ht: true
categories: English
tags: [Spark]
mathjax: true
mathjax2: true
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 27 Regression 回归
<center><strong>译者</strong>：<u><a style="color:#0879e3" href="https://snaildove.github.io">https://snaildove.github.io</a></u></center>

Regression is a logical extension of classification. Rather than just predicting a single value from a set of values, regression is the act of predicting a real number (or continuous variable) from a set of features (represented as numbers).

回归是分类的逻辑延伸。回归不是仅仅从一组值中预测单个值，而是从一组特征（表示为数字）预测实数（或连续变量）的行为。

Regression can be harder than classification because, from a mathematical perspective, there are an infinite number of possible output values. Furthermore, we aim to optimize some metric of error between the predicted and true value, as opposed to an accuracy rate. Aside from that, regression and classification are fairly similar. For this reason, we will see a lot of the same underlying concepts applied to regression as we did with classification.

回归可能比分类更难，因为从数学角度来看，存在无限数量的可能输出值。此外，我们的目标是优化预测值和真值之间的一些误差度量，而不是准确率。除此之外，回归和分类非常相似。出于这个原因，我们将看到许多与回归相同的基本概念应用于分类。

## <font color="#9a161a">Use Cases 使用案例</font>

The following is a small set of regression use cases that can get you thinking about potential regression problems in your own domain:

以下是一小组回归用例，可以让您考虑自己领域中潜在的回归问题：

**Predicting movie viewership 预测电影收视率**

Given information about a movie and the movie-going public, such as how many people have watched the trailer or shared it on social media, you might want to predict how many people are likely to watch the movie when it comes out.

如果给出有关电影和电影公众的信息，例如有多少人观看了预告片或在社交媒体上分享了预告片，您可能想要预测有多少人在电影上映时可能会看电影。

**Predicting company revenue 预测公司收入**

Given a current growth trajectory, the market, and seasonality, you might want to predict how much revenue a company will gain in the future.

如果给出目前的增长轨迹，市场和季节性，您可能希望预测公司未来将获得多少收入。

**Predicting crop yield 预测作物产量**

Given information about the particular area in which a crop is grown, as well as the current weather throughout the year, you might want to predict the total crop yield for a particular plot of land.

如果给出有关作物种植的特定区域以及全年当前天气的信息，您可能希望预测特定土地的总产量。


## <font color="#9a161a">Regression Models in MLlib 在MLlib中的回归模型</font>

There are several fundamental regression models in MLlib. Some of these models are carryovers from Chapter 26. Others are only relevant to the regression problem domain. This list is current as of Spark 2.2 but will grow :

MLlib 中有几个基本的回归模型。其中一些模型是第26章的遗留物。其他模型只与回归问题内的领域有关。此列表是 Spark 2.2 的最新列表，但会增长：

- Linear regression 线性回归

- Generalized linear regression 广义线性回归

- Isotonic regression 保序回归

- Decision trees 决策树

- Random forest 随机森林

- Gradient-boosted trees 梯度提升树

- Survival regression 生存回归

This chapter will cover the basics of each of these particular models by providing:

本章将通过提供以下内容来介绍每种特定模型的基础知识：

- A simple explanation of the model and the intuition behind the algorithm 模型的简单解释和算法背后的直觉
- Model hyperparameters (the different ways that we can initialize the model) 模型超参数（我们可以初始化模型的不同方式）
- Training parameters (parameters that affect how the model is trained) 训练参数（影响模型训练方式的参数）
- Prediction parameters (parameters that affect how predictions are made) 预测参数（影响预测方式的参数）

You can search over the hyperparameters and training parameters using a `ParamGrid`, as we saw in Chapter 24.

您可以使用 `ParamGrid` 搜索超参数和训练参数，如第24章所述。

### <font color="#00000">Model Scalability 模型可伸缩性</font>

The regression models in MLlib all scale to large datasets. Table 27-1 is a simple model scalability scorecard that will help you in choosing the best model for your particular task (if scalability is your core consideration). These will depend on your configuration, machine size, and other factors.

MLlib中的回归模型全都可以扩展到大型数据集。表27-1是一个简单的模型可伸缩性记分卡，可帮助您为特定任务选择最佳模型（如果可扩展性是您的核心考虑因素）。这些将取决于您的配置，机器大小和其他因素。


Table 27-1. Regression scalability reference

表27-1。回归可伸缩性参考 

| Model                         | Number features | Training examples |
| ----------------------------- | --------------- | ----------------- |
| Linear regression             | 1 to 10 million | No limit          |
| Generalized linear regression | 4,096           | No limit          |
| Isotonic regression           | N/A             | Millions          |
| Decision trees                | 1,000s          | No limit          |
| Random forest                 | 10,000s         | No limit          |
| Gradient-boosted              | trees 1,000s    | No limit          |
| Survival regression           | 1 to 10 million | No limit          |

---

<p><center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong></font></center></p>
<p>Like our other advanced analytics chapters, this one cannot teach you the mathematical underpinnings of every model. See Chapter 3 in ISL and ESL for a review of regression.<br /><br />像我们的其他高级分析章节一样，这个章节不能教你每个模型的数学基础。 有关回归的评论，请参阅ISL和ESL的第3章。</p>
---

Let’s read in some sample data that we will use throughout the chapter:

让我们读一下我们将在本章中使用的一些示例数据：

```scala
// in Scala
val df = spark.read.load("/data/regression")
```

```python
# in Python
df = spark.read.load("/data/regression") 
```

## <font color="#9a161a">Linear Regression 线性回归</font>

Linear regression assumes that a linear combination of your input features (the sum of each feature multiplied by a weight) results along with an amount of Gaussian error in the output. This linear assumption (along with Gaussian error) does not always hold true, but it does make for a simple, interpretable model that’s hard to overfit. Like logistic regression, Spark implements ElasticNet regularization for this, allowing you to mix L1 and L2 regularization.

线性回归假设输入特征的线性组合（每个特征的总和乘以权重）与输出中的高斯误差量一起产生。这种线性假设（连同高斯误差）并不总是成立，但它确实构成了一个难以过度拟合的简单，可解释的模型。与逻辑回归一样，Spark 为此实现了 ElasticNet 正则化，允许您混合L1和L2正则化。

See [ISL](http://www-bcf.usc.edu/~gareth/ISL/) 3.2 and [ESL](http://statweb.stanford.edu/~tibs/ElemStatLearn/) 3.2 for more information.

有关详细信息，请参阅 [ISL](http://www-bcf.usc.edu/~gareth/ISL/) 3.2和 [ESL](http://statweb.stanford.edu/~tibs/ElemStatLearn/) 3.2。


### <font color="#00000">Model Hyperparameters 模型超参数</font>

Linear regression has the same model hyperparameters as logistic regression. See Chapter 26 for more information.

线性回归具有与逻辑回归相同的模型超参数。有关更多信息，请参见第26章。

### <font color="#00000">Training Parameters 训练参数</font>

Linear regression also shares all of the same training parameters from logistic regression. Refer back to Chapter 26 for more on this topic.

线性回归还与逻辑回归共享所有相同的训练参数。有关此主题的更多信息，请参阅第26章。

### <font color="#00000">Example 示例</font>

Here’s a short example of using linear regression on our sample dataset:

以下是在样本数据集上使用线性回归的简短示例：


```scala
// in Scala
import org.apache.spark.ml.regression.LinearRegression
val lr = new LinearRegression().setMaxIter(10).setRegParam(0.3)\
.setElasticNetParam(0.8)
println(lr.explainParams())
val lrModel = lr.fit(df)
```

```python
# in Python

from pyspark.ml.regression import LinearRegression
lr = LinearRegression().setMaxIter(10).setRegParam(0.3).setElasticNetParam(0.8)
print lr.explainParams()
lrModel = lr.fit(df)
```

### <font color="#00000">Training Summary 训练摘要</font>

Just as in logistic regression, we get detailed training information back from our model. The code font method is a simple shorthand for accessing these metrics. It reports several conventional metrics for measuring the success of a regression model, allowing you to see how well your model is actually fitting the line.

就像在逻辑回归中一样，我们从模型中获取详细的训练信息。code font 方法是访问这些指标的简单简写。它报告了几个用于衡量回归模型成功的常规指标，使您可以看到模型实际拟合生产线的程度。

The summary method returns a summary object with several fields. Let’s go through these in turn. The residuals are simply the weights for each of the features that we input into the model. The objective history shows how our training is going at every iteration. The root mean squared error is a measure of how well our line is fitting the data, determined by looking at the distance between each predicted value and the actual value in the data. The R-squared variable is a measure of the proportion of the variance of the predicted variable that is captured by the model.

summary方法返回包含多个字段的摘要对象。让我们依次讨论这些问题。残差只是我们输入模型的每个特征的权重。目标历史显示了我们的训练在每次迭代中的进展情况。均方根误差是衡量我们的线拟合数据的程度，通过查看每个预测值与数据中实际值之间的距离来确定。 R平方变量是模型捕获的预测变量的方差比例的度量。

There are a number of metrics and summary information that may be relevant to your use case. This section demonstrates the API, but does not comprehensively cover every metric (consult the API documentation for more information).

有许多可能与您的使用案例相关的指标和摘要信息。本节演示API，但不全面涵盖每个指标（有关更多信息，请参阅API文档）。

Here are some of the attributes of the model summary for linear regression:

以下是线性回归模型摘要的一些属性：

```scala
// in Scala
val summary = lrModel.summary
summary.residuals.show()
println(summary.objectiveHistory.toSeq.toDF.show())
println(summary.rootMeanSquaredError)
println(summary.r2)
```

```python
# in Python
summary = lrModel.summary
summary.residuals.show()
print summary.totalIterations
print summary.objectiveHistory
print summary.rootMeanSquaredError
print summary.r2
```

## <font color="#9a161a">Generalized Linear Regression 广义线性回归</font>

The standard linear regression that we saw in this chapter is actually a part of a family of algorithms called generalized linear regression. Spark has two implementations of this algorithm. One is optimized for working with very large sets of features (the simple linear regression covered previously in this chapter), while the other is more general, includes support for more algorithms, and doesn’t currently scale to large numbers of features.

我们在本章中看到的标准线性回归实际上是一类称为广义线性回归的算法的一部分。 Spark有两种这种算法的实现。一个针对处理非常大的特征集（本章前面介绍的简单线性回归）进行了优化，而另一个则更为通用，包括对更多算法的支持，并且目前不能扩展到大量特征。

The generalized form of linear regression gives you more fine-grained control over what kind of regression model you use. For instance, these allow you to select the expected noise distribution from a variety of families, including Gaussian (linear regression), binomial (logistic regression), poisson (poisson regression), and gamma (gamma regression). The generalized models also support setting a link function that specifies the relationship between the linear predictor and the mean of the distribution function. Table 27-2 shows the available link functions for each family.

线性回归的广义形式使您可以更精细地控制所使用的回归模型。例如，这些允许您从各种族（family）中选择预期的噪声分布，包括高斯（线性回归），二项式（逻辑回归），泊松（泊松回归）和伽玛（伽马回归）。广义模型还支持设置连接函数（link function），该函数指定线性预测器与分布函数的均值之间的关系。表27-2显示了每个系列的可用连接函数（link function）。

Table 27-2. Regression families, response types, and link functions 

表27-2  回归族（families），响应类型和连接函数（link function）

| Family   | Response type | Supported links                |
| -------- | ------------- | ------------------------------ |
| Gaussian | Continuous    | Identity*, Log, Inverse        |
| Binomial | Binary        | Logit*, Probit, CLogLog        |
| Poisson  | Count         | Log*, Identity, Sqrt           |
| Gamma    | Continuous    | Inverse*, Idenity, Log         |
| Tweedie  | Zero-inflated | continuous Power link function |

The asterisk signifies the canonical link function for each family.

星号表示每个族（family）的规范（canonical）连接函数（link function）。

See [ISL](http://www-bcf.usc.edu/~gareth/ISL/)  3.2 and [ESL](http://statweb.stanford.edu/~tibs/ElemStatLearn/)  3.2 for more information on generalized linear models.

有关广义线性模型的更多信息，请参见 [ISL](http://www-bcf.usc.edu/~gareth/ISL/)  3.2 和 [ESL](http://statweb.stanford.edu/~tibs/ElemStatLearn/)  3.2。

---

<p><center><font face="constant-width" color="#c67171" size=3><strong>WARNING 警告</strong></font></center><p>
<p>A fundamental limitation as of Spark 2.2 is that generalized linear regression only accepts a maximum of 4,096 features for inputs. This will likely change for later versions of Spark, so be sure to refer to the documentation.<br /></p>Spark 2.2 的一个基本限制是广义线性回归仅接受最多4,096个输入特征。 对于更高版本的Spark，这可能会有所改变，因此请务必参考文档。</p>
---


### <font color="#00000">Model Hyperparameters 模型超参数</font>

These are configurations that we specify to determine the basic structure of the model itself. In addition to `fitIntercept` and `regParam` (mentioned in “Regression”), generalized linear regression includes several other hyperparameters:

这些是我们指定的配置，用于确定模型本身的基本结构。除了 `fitIntercept` 和 `regParam`（在“回归”中提到）之外，广义线性回归还包括其他几个超参数：

<p><font face="constant-width" color="#000000" size=4>family</font></P>
A description of the error distribution to be used in the model. Supported options are Poisson, binomial, gamma, Gaussian, and Tweedie.

要在模型中使用的错误分布的描述。支持的选项包括泊松，二项，伽马，高斯和特威迪（Tweedie）。

<p><font face="constant-width" color="#000000" size=4>link</font></p>
The name of link function which provides the relationship between the linear predictor and the mean of the distribution function. Supported options are cloglog, probit, logit, inverse, sqrt, identity, and log (default: identity).

连接函数的名称，它提供线性预测器与分布函数均值之间的关系。支持的选项包括 cloglog，probit，logit，inverse，sqrt，identity 和 log（默认值：identity）。

<p><font face="constant-width" color="#000000" size=4>solver</font></p>
The solver algorithm to be used for optimization. The only currently supported solver is irls (iteratively reweighted least squares).

<p>用于优化的解算器算法。目前唯一支持的解算器是 <font face="constant-width" color="#000000" size=3>irls</font>（迭代重新加权最小二乘）。</P>
<p><font face="constant-width" color="#000000" size=4>variancePower</font></p>
The power in the variance function of the Tweedie distribution, which characterizes the relationship between the variance and mean of the distribution. Only applicable to the Tweedie family. Supported values are 0 and [1, Infinity). The default is 0.

特威迪（Tweedie）分布的方差函数中的幂，其表征方差和分布均值之间的关系。仅适用于特威迪（Tweedie）分布族。支持的值为 0 和 $[1，\infty]$。默认值为0。

<p><font face="constant-width" color="#000000" size=4>linkPower</font></p>
The index in the power link function for the Tweedie family.

特威迪（Tweedie）分布族连接函数（link function）的幂的索引。


### <font color="#00000">Training Parameters 训练参数</font>

The training parameters are the same that you will find for logistic regression. Consult Chapter 26 for more information.

训练参数与逻辑回归相同。有关更多信息，请参阅第26章。

### <font color="#00000">Prediction Parameters 预测参数</font>

This model adds one prediction parameter:

该模型添加了一个预测参数：

<p><font face="constant-width" color="#000000" size=4>linkPredictionCol</font></P>
A column name that will hold the output of our link function for each prediction.

一个列名，用于保存每个预测的连接函数（link function）的输出。

### <font color="#00000">Example 例子</font>

Here’s an example of using `GeneralizedLinearRegression`:

以下是使用 `GeneralizedLinearRegression` 的示例：

```scala
// in Scala
import org.apache.spark.ml.regression.GeneralizedLinearRegression
val glr = new GeneralizedLinearRegression()
.setFamily("gaussian")
.setLink("identity")
.setMaxIter(10)
.setRegParam(0.3)
.setLinkPredictionCol("linkOut")
println(glr.explainParams())
val glrModel = glr.fit(df)
```

```python
# in Python
from pyspark.ml.regression import GeneralizedLinearRegression
glr = GeneralizedLinearRegression()\
.setFamily("gaussian")\
.setLink("identity")\
.setMaxIter(10)\
.setRegParam(0.3)\
.setLinkPredictionCol("linkOut")
print glr.explainParams()
glrModel = glr.fit(df)
```

### <font color="#00000">Training Summary 训练摘要</font>

As for the simple linear model in the previous section, the training summary provided by Spark for the generalized linear model can help you ensure that your model is a good fit for the data that you used as the training set. It is important to note that this does not replace running your algorithm against a proper test set, but it can provide more information. This information includes a number of different potential metrics for analyzing the fit of your algorithm, including some of the most common success metrics:

对于上一节中的简单线性模型，Spark为广义线性模型提供的训练摘要可以帮助您确保模型非常拟合您用作训练集的数据。重要的是要注意，这不会取代运行您的算法与正确的测试集，但它可以提供更多信息。此信息包括许多用于分析算法拟合的不同潜在指标，包括一些最常见的成功指标：

R squared

- The coefficient of determination; a measure of fit. 
决定系数；拟合的一种度量。

The residuals 残差

- The difference between the label and the predicted value. 
标签和预测值之间的差异。

Be sure to inspect the summary object on the model to see all the available methods. 

请务必检查模型上的摘要对象以查看所有可用方法。

## <font color="#9a161a">Decision Trees 决策树</font>

Decision trees as applied to regression work fairly similarly to decision trees applied to classification. The main difference is that decision trees for regression output a single number per leaf node instead of a label (as we saw with classification). The same interpretability properties and model structure still apply. In short, rather than trying to train coeffiecients to model a function, decision tree regression simply creates a tree to predict the numerical outputs. This is of significant consequence because unlike generalized linear regression, we can predict nonlinear functions in the input data. This also creates a significant risk of overfitting the data, so we need to be careful when tuning and evaluating these models.

应用于回归的决策树与应用于分类的决策树非常相似。主要区别在于回归的决策树每个叶节点输出一个数字而不是标签（正如我们在分类中看到的那样）。相同的可解释性属性和模型结构仍然适用。简而言之，决策树回归不是试图训练系数来模拟函数，而是简单地创建一个树来预测数值输出。这是重要的结果，因为与广义线性回归不同，我们可以预测输入数据中的非线性函数。这也会产生过度拟合数据的重大风险，因此在调整和评估这些模型时需要小心。

We also covered decision trees in Chapter 26 (refer to “Decision Trees”). For more information on this topic, consult [ISL](http://www-bcf.usc.edu/~gareth/ISL/)  8.1 and [ESL](http://statweb.stanford.edu/~tibs/ElemStatLearn/) 9.2 .

我们还在第26章介绍了决策树（参见“决策树”）。有关该主题的更多信息，请参阅 [ISL](http://www-bcf.usc.edu/~gareth/ISL/) 8.1和 [ESL](http://statweb.stanford.edu/~tibs/ElemStatLearn/) 9.2。

### <font color="#00000">Model Hyperparameters 模型超参数</font>

The model hyperparameters that apply decision trees for regression are the same as those for classification except for a slight change to the impurity parameter. See Chapter 26 for more information on the other hyperparameters:

应用决策树进行回归的模型超参数与分类相同，只是杂质参数略有变化。有关其他超参数的更多信息，请参见第26章：

<font face="constant-width" color="#000000" size=4>impurity 不纯度</font>

	The impurity parameter represents the metric (information gain) for whether or not the model should split at a particular leaf node with a particular value or keep it as is. The only metric currently supported for regression trees is “variance.”

不纯度参数表示模型是否应在具有特定值的特定叶节点处分割或保持原样的度量（信息增益）。目前支持回归树的唯一指标是“方差（variance）”。

### <font color="#00000">Training Parameters 训练参数</font>

In addition to hyperparameters, classification and regression trees also share the same training parameters. See “Training Parameters” in Chapter 26  for these parameters.

除了超参数，分类和回归树也共享相同的训练参数。有关这些参数，请参见第26章中的“训练参数”。

### <font color="#00000">Example 示例</font>

Here’s a short example of using a decision tree regressor:

以下是使用决策树回归程序的简短示例：


```scala
// in Scala
import org.apache.spark.ml.regression.DecisionTreeRegressor
val dtr = new DecisionTreeRegressor()
println(dtr.explainParams())
val dtrModel = dtr.fit(df)
```

```python
# in Python
from pyspark.ml.regression import DecisionTreeRegressor
dtr = DecisionTreeRegressor()
print dtr.explainParams()
dtrModel = dtr.fit(df)
```

## <font color="#9a161a">Random Forests and Gradient-Boosted Trees 随机森林和梯度提升树</font>

The random forest and gradient-boosted tree models can be applied to both classification and regression. As a review, these both follow the same basic concept as the decision tree, except rather than training one tree, many trees are trained to perform a regression. In the random forest model, many de-correlated trees are trained and then averaged. With gradient-boosted trees, each tree makes a weighted prediction (such that some trees have more predictive power for some classes over others). Random forest and gradient-boosted tree regression have the same model hyperparameters and training parameters as the corresponding classification models, except for the purity measure (as is the case with `DecisionTreeRegressor`).

随机森林和梯度提升树模型可以应用于分类和回归。作为回顾，这些都遵循与决策树相同的基本概念，除了训练一棵树，训练许多树进行回归。在随机森林模型中，训练去相关的树然后进行平均。使用梯度提升树，每棵树进行加权预测（这样一些树对某些类具有比其他树更多的预测能力）。随机森林和梯度提升树回归具有与相应分类模型相同的模型超参数和训练参数，除了纯度测量（如“`DecisionTreeRegressor`”的情况）。

See [ISL](http://www-bcf.usc.edu/~gareth/ISL/)  8.2 and [ESL](http://statweb.stanford.edu/~tibs/ElemStatLearn/) 10.1 for more information on tree ensembles.

有关树集合的更多信息，请参见 [ISL](http://www-bcf.usc.edu/~gareth/ISL/)  8.2 和 [ESL](http://statweb.stanford.edu/~tibs/ElemStatLearn/) 10.1。

### <font color="#00000">Model Hyperparameters 模型超参数</font>

These models share many of the same parameters as we saw in the previous chapter as well as for regression decision trees. Refer back to “Model Hyperparameters” in Chapter 26 for a thorough explanation of these parameters. As for a single regression tree, however, the only impurity metric currently supported is variance.

这些模型共享许多与我们在前一章中看到的相同的参数以及回归决策树。有关这些参数的详细说明，请参阅第26章中的“模型超参数”。但是，对于单个回归树，当前支持的唯一不纯度的度量（impurity metric）是方差。

### <font color="#00000">Training Parameters 训练参数</font>

These models support the same `checkpointInterval` parameter as classification trees, as described in Chapter 26.

这些模型支持与分类树相同的`checkpointInterval`参数，如第26章所述。
### <font color="#00000">Example</font>

Here’s a small example of how to use these two models to perform a regression: 

以下是如何使用这两个模型执行回归的一个小示例：

```scala 
// in Scala
import org.apache.spark.ml.regression.RandomForestRegressor
import org.apache.spark.ml.regression.GBTRegressor
val rf = new RandomForestRegressor()
println(rf.explainParams())
val rfModel = rf.fit(df)
val gbt = new GBTRegressor()
println(gbt.explainParams())
val gbtModel = gbt.fit(df)
```

```python
# in Python
from pyspark.ml.regression import RandomForestRegressor
from pyspark.ml.regression import GBTRegressor
rf = RandomForestRegressor()
print rf.explainParams()
rfModel = rf.fit(df)
gbt = GBTRegressor()
print gbt.explainParams()
gbtModel = gbt.fit(df)
```
## <font color="#9a161a">Advanced Methods 高级方法</font>

The preceding methods are highly general methods for performing a regression. The models are by no means exhaustive, but do provide the essential regression types that many folks use. This next section will cover some of the more specialized regression models that Spark includes. We omit code examples simply because they follow the same patterns as the other algorithms.

前述方法是用于执行回归的高度通用的方法。这些模型并非详尽无遗，但确实提供了许多人使用的基本回归类型。下一节将介绍Spark包含的一些更专业的回归模型。我们省略代码示例只是因为它们遵循与其他算法相同的模式。

### <font color="#00000">Survival Regression (Accelerated Failure Time) 生存回归（加速失败时间）</font>

Statisticians use survival analysis to understand the survival rate of individuals, typically in controlled experiments. Spark implements the accelerated failure time model, which, rather than describing the actual survival time, models the log of the survival time. This variation of survival regression is implemented in Spark because the more well-known Cox Proportional Hazard’s model is semi-parametric and does not scale well to large datasets. By contrast, accelerated failure time does because each instance (row) contributes to the resulting model independently. Accelerated failure time does have different assumptions than the Cox survival model and therefore one is not necessarily a drop-in replacement for the other. Covering these differing assumptions is outside of the scope of this book. See [L. J. Wei’s paper](https://onlinelibrary.wiley.com/doi/abs/10.1002/sim.4780111409) on accelerated failure time for more information.

统计学家使用生存分析来了解个体的存活率，通常是在对照实验中。Spark实现了加速失败时间模型，该模型不是描述实际生存时间，而是模拟生存时间的对数。这种生存回归的变体（variation ）在Spark中实现，因为更为人熟知的 Cox Proportional Hazard 模型是半参数的，并且不能很好地扩展到大型数据集。相比之下，加速失败时间确实存在，因为每个实例（行）都独立地对结果模型做出贡献。加速失败时间确实具有与 Cox 生存模型不同的假设，因此一个不一定是另一个的直接替代品。涵盖这些不同的假设超出了本书的范围。见 [L. J. Wei的论文](https://onlinelibrary.wiley.com/doi/abs/10.1002/sim.4780111409) 关于加速失败时间以获取更多信息。

The requirement for input is quite similar to that of other regressions. We will tune coefficients according to feature values. However, there is one departure, and that is the introduction of a censor variable column. A test subject censors during a scientific study when that individual drops out of a study, since their state at the end of the experiment may be unknown. This is important because we cannot assume an outcome for an individual that censors (doesn’t report that state to the researchers) at some intermediate point in a study.

输入要求与其他回归非常相似。我们将根据特征值调整系数。然而，有一个偏离，那就是引入一个检查变量列。当一个人退出研究时，测试对象在科学研究期间进行检查，因为他们在实验结束时的状态可能是未知的。这很重要，因为我们无法假设在研究的某个中间点审查（不向研究人员报告该状态）的个人的结果。

See more about survival regression with AFT in [the documentation](http://spark.apache.org/docs/latest/ml-classification-regression.html#survival-regression).

在[文档](http://spark.apache.org/docs/latest/ml-classification-regression.html#survival-regression)中查看有关AFT的生存回归的更多信息。

### <font color="#00000">Isotonic Regression 保序回归</font>

Isotonic regression is another specialized regression model, with some unique requirements. Essentially, isotonic regression specifies a piecewise linear function that is always monotonically increasing. It cannot decrease. This means that if your data is going up and to the right in a given plot, this is an appropriate model. If it varies over the course of input values, then this is not appropriate. The illustration of isotonic regression’s behavior in Figure 27-1 makes it much easier to understand. 

保序回归是另一种专门的回归模型，具有一些独特的要求。本质上，保序回归指定了一个单调递增的分段线性函数。它不能减少。这意味着如果您的数据在给定的图中向上和向右，这是一个合适的模型。如果它在输入值的过程中变化，那么这是不合适的。图27-1中保序回归的行为说明使其更容易理解。

![1568215916793](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter27/1568215916793.png)

Notice how this gets a better fit than the simple linear regression. See more about how to use this model in [the Spark documentation](http://spark.apache.org/docs/latest/ml-classification-regression.html#isotonic-regression). 

注意这比简单的线性回归更合适。在[Spark文档](http://spark.apache.org/docs/latest/ml-classification-regression.html#isotonic-regression)中查看有关如何使用此模型的更多信息。

## <font color="#9a161a">Evaluators and Automating Model Tuning 评估器和自动化模型调整</font>

Regression has the same core model tuning functionality that we saw with classification. We can specify an evaluator, pick a metric to optimize for, and then train our pipeline to perform that parameter tuning on our part. The evaluator for regression, unsurprisingly, is called the `RegressionEvaluator` and allows us to optimize for a number of common regression success metrics. Just like the classification evaluator, `RegressionEvaluator` expects two columns, a column representing the prediction and another representing the true label. The supported metrics to optimize for are the root mean squared error (“rmse”), the mean squared error (“mse”), the $r^2$ metric (“r2”), and the mean absolute error (“mae”).

回归具有与分类相同且关键的模型调整功能。我们可以指定一个评估器，选择一个要优化的度量，然后训练我们的管道来执行我们的参数调整。毫无疑问，回归评估器称为 `RegressionEvaluator`，它允许我们针对许多常见的回归成功度量进行优化。就像分类评估器一样，`RegressionEvaluator` 需要两列，一列代表预测值，另一列代表真实标签。要优化的支持度量是均方根误差（“rmse”），均方误差（“mse”），$ r ^ 2 $ metric（“r2”）和平均绝对误差（“mae”） ）。

To use `RegressionEvaluator`, we build up our pipeline, specify the parameters we would like to test, and then run it. Spark will automatically select the model that performs best and return this to us:

要使用 `RegressionEvaluator`，我们构建我们的管道，指定我们想要测试的参数，然后运行它。 Spark 会自动选择性能最佳的模型并将其返回给我们：
```scala
// in Scala
import org.apache.spark.ml.evaluation.RegressionEvaluator
import org.apache.spark.ml.regression.GeneralizedLinearRegression
import org.apache.spark.ml.Pipeline
import org.apache.spark.ml.tuning.{CrossValidator, ParamGridBuilder}
val glr = new GeneralizedLinearRegression()
.setFamily("gaussian")
.setLink("identity")
val pipeline = new Pipeline().setStages(Array(glr))
val params = new ParamGridBuilder().addGrid(glr.regParam, Array(0, 0.5, 1))
.build()
val evaluator = new RegressionEvaluator()
.setMetricName("rmse")
.setPredictionCol("prediction")
.setLabelCol("label")
val cv = new CrossValidator()
.setEstimator(pipeline)
.setEvaluator(evaluator)
.setEstimatorParamMaps(params)
.setNumFolds(2) // should always be 3 or more but this dataset is small
val model = cv.fit(df)
```

```python
# in Python
from pyspark.ml.evaluation import RegressionEvaluator
from pyspark.ml.regression import GeneralizedLinearRegression
from pyspark.ml import Pipeline
from pyspark.ml.tuning import CrossValidator, ParamGridBuilder
glr = GeneralizedLinearRegression().setFamily("gaussian").setLink("identity")
pipeline = Pipeline().setStages([glr])
params = ParamGridBuilder().addGrid(glr.regParam, [0, 0.5, 1]).build()
evaluator = RegressionEvaluator()\
.setMetricName("rmse")\
.setPredictionCol("prediction")\
.setLabelCol("label")
cv = CrossValidator()\
.setEstimator(pipeline)\
.setEvaluator(evaluator)\
.setEstimatorParamMaps(params)\
.setNumFolds(2) # should always be 3 or more but this dataset is small
model = cv.fit(df)
```

## <font color="#9a161a">Metrics 衡量指标</font>

Evaluators allow us to evaluate and fit a model according to one specific metric, but we can also access a number of regression metrics via the `RegressionMetrics` object. As for the classification metrics in the previous chapter, `RegressionMetrics` operates on RDDs of (prediction, label) pairs. For instance, let’s see how we can inspect the results of the previously trained model.

评估器允许我们根据一个特定指标评估和拟合模型，但我们也可以通过 `RegressionMetrics` 对象访问许多回归指标。 对于前一章中的分类度量，`RegressionMetrics` 对（预测，标签）数据对的RDD进行操作。 例如，让我们看看我们如何检查以前训练过的模型的结果。

```scala
// in Scala
import org.apache.spark.mllib.evaluation.RegressionMetrics
val out = model.transform(df)
.select("prediction", "label")
.rdd.map(x => (x(0).asInstanceOf[Double], x(1).asInstanceOf[Double]))
val metrics = new RegressionMetrics(out)
println(s"MSE = {metrics.meanSquaredError}")
println(s"RMSE = {metrics.rootMeanSquaredError}")
println(s"R-squared = {metrics.r2}")
println(s"MAE = {metrics.meanAbsoluteError}")
println(s"Explained variance = ${metrics.explainedVariance}")
```

```python
# in Python
from pyspark.mllib.evaluation import RegressionMetrics
out = model.transform(df)\
.select("prediction", "label").rdd.map(lambda x: (float(x[0]), float(x[1])))
metrics = RegressionMetrics(out)
print "MSE: " + str(metrics.meanSquaredError)
print "RMSE: " + str(metrics.rootMeanSquaredError)
print "R-squared: " + str(metrics.r2)
print "MAE: " + str(metrics.meanAbsoluteError)
print "Explained variance: " + str(metrics.explainedVariance)
```

Consult [the Spark documentation](http://spark.apache.org/docs/latest/mllib-evaluation-metrics.html) for the latest methods.

有关最新方法，请参阅[Spark文档](http://spark.apache.org/docs/latest/mllib-evaluation-metrics.html)。

## <font color="#9a161a">Conclusion 结论</font>

In this chapter, we covered the basics of regression in Spark, including how we train models and how we measure success. In the next chapter, we’ll take a look at recommendation engines, one of the more popular applications of MLlib. 

在本章中，我们介绍了Spark中回归的基础知识，包括我们如何训练模型以及如何衡量成功。 在下一章中，我们将介绍推荐引擎，这是 MLlib 更受欢迎的应用之一。
