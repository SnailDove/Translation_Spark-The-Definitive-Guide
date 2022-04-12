---
title: 翻译 Chapter 29 Unsupervised Learning
date: 2019-08-05
copyright: true
categories: English
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 29 Unsupervised Learning

<center><strong>译者</strong>：<u><a style="color:#0879e3" href="https://snaildove.github.io">https://snaildove.github.io</a></u></center>

This chapter will cover the details of Spark’s available tools for unsupervised learning, focusing specifically on clustering. Unsupervised learning is, generally speaking, used less often than supervised learning because it’s usually harder to apply and measure success (from an end-result perspective). These challenges can become exacerbated at scale. For instance, clustering in high dimensional space can create odd clusters simply because of the properties of high-dimensional spaces, something referred to as *the curse of dimensionality* . The curse of dimensionality describes the fact that as a feature space expands in dimensionality, it becomes increasingly sparse. This means that the data needed to fill this space for statistically meaningful results increases rapidly with any increase in dimensionality. Additionally, with high dimensions comes more noise in the data. This, in turn, may cause your model to hone in on noise instead of the true factors causing a particular result or grouping. Therefore in the model scalability table, we include computational limits, as well as a set of statistical recommendations. These are heuristics and should be helpful guides, not requirements.

本章将详细介绍Spark的可用于无监督学习的工具，重点是集群。一般来说，无监督学习的使用频率比有监督学习的频率要低，因为无监督学习通常很难应用和衡量成功（从最终结果的角度来看）。这些挑战可能会在规模上加剧。例如，高维空间中的聚类可以仅仅由于高维空间的特性（称为维数的诅咒）而创建奇数簇。维度的诅咒描述了一个事实，即随着特征空间维度的扩展，它变得越来越稀疏。这意味着，填充该空间以获取具有统计意义的结果所需的数据会随着维度的增加而迅速增加。此外，尺寸越大，数据中的噪声越多。反过来，这可能会导致模型陷入噪音，而不是导致特定结果或分组的真实因素。因此，在模型可伸缩性表中，我们包括计算限制以及一组统计建议。这些是试探法，应该是有用的指南，而不是要求。

At its core, *unsupervised learning* is trying to discover patterns or derive a concise representation of the underlying structure of a given dataset.

本质上，无监督学习试图发现模式或派生给定数据集的基础结构的简洁表示。

## <font color="#9a161a">Use Cases 用户案例</font>

Here are some potential use cases. At its core, these patterns might reveal topics, anomalies, or groupings in our data that may not have been obvious beforehand:

这里是一些潜在的用例。从根本上讲，这些模式可能会揭示我们数据中可能事先不明显的主题，异常或分组：

Finding anomalies in data 在数据中找异常值

If the majority of values in a dataset cluster into a larger group with several small groups on the outside, those groups might warrant further investigation.
如果数据集中的大多数值聚集到一个较大的组中，外部又有几个小组，则这些组可能需要进一步调查。

Topic modeling 主题模型

By looking at large bodies of text, it is possible to find topics that exist across those different documents.

通过查看大量的正文，可以找到这些不同文档中存在的主题。

## <font color="#9a161a">Model Scalability 模型的伸缩性</font>

Just like with our other models, it’s important to mention the basic model scalability requirements along with statistical recommendations.

与其他模型一样，重要的是要提及基本模型可扩展性要求以及统计建议。

Table 29-1. Clustering model scalability reference 集群模型可伸缩性参考

| Model             | Statistical recommendation                | Computation limits                                           | Training examples |
| ----------------- | ----------------------------------------- | ------------------------------------------------------------ | ----------------- |
| k-means           | 50 to 100                                 | maximum Features x clusters < 10 million<br />最大特征群小于1000,0000万 | No limit          |
| Bisecting k-means | 50 to 100                                 | maximum Features x clusters < 10 million<br />最大特征群小于1000,0000万 | No limit          |
| GMM               | 50 to 100                                 | maximum Features x clusters < 10 million<br />最大特征群小于1000,0000万 | No limit          |
| LDA               | An interpretable number<br />可解释的数字 | 1,000s of topics<br />千数理级的主题                         | No limit          |

Let’s get started by loading some example numerical data:

让我们从加载一些示例数字数据开始：

```scala
// in Scala
import org.apache.spark.ml.feature.VectorAssembler
val va = new VectorAssembler()
.setInputCols(Array("Quantity", "UnitPrice"))
.setOutputCol("features")
val sales = va.transform(spark.read.format("csv")
.option("header", "true")
.option("inferSchema", "true")
.load("/data/retail-data/by-day/*.csv")
.limit(50)
.coalesce(1)
.where("Description IS NOT NULL"))
sales.cache()
```

```python
# in Python
from pyspark.ml.feature import VectorAssembler
va = VectorAssembler()\
.setInputCols(["Quantity", "UnitPrice"])\
.setOutputCol("features")
sales = va.transform(spark.read.format("csv")
.option("header", "true")
.option("inferSchema", "true")
.load("/data/retail-data/by-day/*.csv")
.limit(50)
.coalesce(1)
.where("Description IS NOT NULL"))
sales.cache()
```

## <font color="#9a161a">k-means K均值</font>

-*means* is one of the most popular clustering algorithms. In this algorithm, a user-specified number of clusters () are randomly assigned to different points in the dataset. The unassigned points are then “assigned” to a cluster based on their proximity (measured in Euclidean distance) to the previously assigned point. Once this assignment happens, the center of this cluster (called the *centroid*) is computed, and the process repeats. All points are assigned to a particular centroid, and a new centroid is computed. We repeat this process for a finite number of iterations or until convergence (i.e., when our centroid locations stop changing). This does not, however, mean that our clusters are always sensical. For instance, a given “logical” cluster of data might be split right down the middle simply because of the starting points of two distinct clusters. Thus, it is often a good idea to perform multiple runs of -means starting with different initializations.

-means是最流行的聚类算法之一。在此算法中，将用户指定数量的聚类（）随机分配给数据集中的不同点。然后根据未分配点与先前分配点的接近度（以欧几里德距离测量）将它们“分配”到聚类。一旦发生这种分配，就会计算出该簇的中心（称为质心），然后重复该过程。将所有点分配给特定的质心，并计算一个新的质心。我们将这个过程重复进行有限的迭代或直到收敛为止（即，当我们的质心位置停止更改时）。但是，这并不意味着我们的集群总是明智的。例如，一个给定的“逻辑”数据集群可能仅由于两个不同集群的起点而在中间被拆分。因此，从不同的初始化开始执行多次-means通常是一个好主意。

Choosing the right value for is an extremely important aspect of using this algorithm successfully, as well as a hard task. There’s no real prescription for the number of clusters you need, so you’ll likely have to experiment with different values and consider what you would like the end result to be. For more information on -means, see <u><a style="color:#0879e3" href="http://faculty.marshall.usc.edu/gareth-james/">ISL 10.3</a></u> and <u><a style="color:#0879e3" href="https://web.stanford.edu/~hastie/ElemStatLearn//">ESL 14.3</a></u>.

选择正确的值是成功使用此算法极为重要的方面，也是一项艰巨的任务。对于所需的群集数量并没有真正的规定，因此您可能必须尝试使用不同的值并考虑最终结果是什么。有关-means的更多信息，请参见 <u><a style="color:#0879e3" href="http://faculty.marshall.usc.edu/gareth-james/">ISL 10.3</a></u> 和 <u><a style="color:#0879e3" href="https://web.stanford.edu/~hastie/ElemStatLearn//">ESL 14.3</a></u>。

### <font color="#00000">Model Hyperparameters  模型的超参数</font>

These are configurations that we specify to determine the basic structure of the model: 

我们指定了以下这些配置来确定模型的基本结构：

This is the number of clusters that you would like to end up with.

这是您最终希望使用的集群数量。

### <font color="#00000">Training Parameters 训练参数</font>

<p><font face="constant-width" color="#000000" size=3>initMode</font></p>
The initialization mode is the algorithm that determines the starting locations of the centroids. The supported options are random and <u><a style="color:#0879e3" href="http://theory.stanford.edu/~sergei/papers/kMeansPP-soda.pdf">-means||</a></u> (the default). The latter is a parallelized variant of the <u><a style="color:#0879e3" href="http://theory.stanford.edu/~sergei/papers/kMeansPP-soda.pdf">-means||</a></u> method. While the details are not within the scope of this book, the thinking behind the latter method is that rather than simply choosing random initialization locations, the algorithm chooses cluster centers that are already well spread out to generate a better clustering.

初始化模式是确定质心起始位置的算法。支持的选项是random和 <u><a style="color:#0879e3" href="http://theory.stanford.edu/~sergei/papers/kMeansPP-soda.pdf">-means||</a></u>。（默认）。后者是 <u><a style="color:#0879e3" href="http://theory.stanford.edu/~sergei/papers/kMeansPP-soda.pdf">-means||</a></u> 的并行变体。方法。尽管细节不在本书的讨论范围之内，但后一种方法的思想是，该算法不仅选择随机初始化位置，还选择分布良好的聚类中心，以生成更好的聚类。

<p><font face="constant-width" color="#000000" size=3>initSteps</font></p> 
The number of steps for <u><a style="color:#0879e3" href="http://theory.stanford.edu/~sergei/papers/kMeansPP-soda.pdf">-means||</a></u> initialization mode. Must be greater than 0. (The default value is 2.)

<u><a style="color:#0879e3" href="http://theory.stanford.edu/~sergei/papers/kMeansPP-soda.pdf">-means||</a></u> 的步骤数初始化模式。必须大于0。（默认值为2。）

<p><font face="constant-width" color="#000000" size=3>maxIter</font></p>
Total number of iterations over the data before stopping. Changing this probably won’t change your results a ton, so don’t make this the first parameter you look to adjust. The default is 20.

停止之前，数据上的迭代总数。更改此设置可能不会使您的结果大为改变，因此请不要将其设为您要调整的第一个参数。默认值为20。

<p><font face="constant-width" color="#000000" size=3>tol</font>

Specifies a threshold by which changes in centroids show that we optimized our model enough, and can stop iterating early, before `maxIter` iterations. The default value is 0.0001. 

指定一个阈值，通过该阈值质心的变化可以表明我们已经充分优化了模型，并且可以在`maxIter`迭代之前尽早停止迭代。默认值为0.0001。

This algorithm is generally robust to these parameters, and the main trade-off is that running more initialization steps and iterations may lead to a better clustering at the expense of longer training time:

该算法通常对这些参数具有鲁棒性，并且主要的权衡是，运行更多的初始化步骤和迭代可能会导致更好的聚类，但需要更长的训练时间：

### <font color="#00000">Example</font>

```scala
// in Scala
import org.apache.spark.ml.clustering.KMeans
val km = new KMeans().setK(5)
println(km.explainParams())
val kmModel = km.fit(sales)
```

```python
# in Python
from pyspark.ml.clustering import KMeans
km = KMeans().setK(5)
print km.explainParams()
kmModel = km.fit(sales)
```

### <font color="#00000">k-means Metrics Summary k-means衡量指标简述</font>

-means includes a summary class that we can use to evaluate our model. This class provides some common measures for -means success (whether these apply to your problem set is another question). The -means summary includes information about the clusters created, as well as their relative sizes (number of examples).

-means包括一个摘要类，可用于评估模型。此类提供了一些用于-means成功的常用措施（这些措施是否适用于您的问题集是另一个问题）。-means摘要包括有关创建的集群及其相对大小（示例数）的信息。

We can also compute the *within set sum of squared errors*, which can help measure how close our values are from each cluster centroid, using `computeCost`. The implicit goal in -means is that we want to minimize the within set sum of squared error, subject to the given number of clusters:

我们还可以计算出平方误差的集合内总和，这可以使用`computeCost`帮助测量值与每个聚类质心的接近程度。-means中的隐式目标是，在给定数量的簇的情况下，我们希望最小化平方误差的设置和之内：

```scala
// in Scala
val summary = kmModel.summary
summary.clusterSizes // number of points
kmModel.computeCost(sales)
println("Cluster Centers: ")
kmModel.clusterCenters.foreach(println)
```

```python
# in Python
summary = kmModel.summary
print summary.clusterSizes # number of points
kmModel.computeCost(sales)
centers = kmModel.clusterCenters()
print("Cluster Centers: ")
for center in centers:
print(center)
```

## <font color="#9a161a">Bisecting k-means二等分K均值</font>

Bisecting -means is a variant of -means. The core difference is that instead of clustering points by starting “bottom-up” and assigning a bunch of different groups in the data, this is a top-down clustering method. This means that it will start by creating a single group and then splitting that groupinto smaller groups in order to end up with the number of clusters specified by the user. This is usually a faster method than -means and will yield different results.

均分-means是-means的变体。核心区别在于，这是一种通过自上而下的聚类方法，而不是通过“自下而上”开始并在数据中分配一堆不同的组来聚类点。这意味着它将首先创建一个组，然后将该组分成较小的组，以得到用户指定的群集数。这通常是比-means更快的方法，并且会产生不同的结果。

### <font color="#00000">Model Hyperparameters 模型参数</font>

These are configurations that we specify to determine the basic structure of the model:  

我们指定了以下这些配置来确定模型的基本结构：

​	This is the number of clusters that you would like to end up with

​	这是您最终希望使用的集群数量

### <font color="#00000">Training Parameters 训练参数</font>

<p><font face="constant-width" color="#000000" size=3>minDivisibleClusterSize</font></p> 
The minimum number of points (if greater than or equal to 1.0) or the minimum proportion of points (if less than 1.0) of a divisible cluster. The default is 1.0, meaning that there must be at least one point in each cluster.

可整类的最小点数（如果大于或等于1.0）或最小比例点（如果小于1.0）。默认值为1.0，这意味着每个群集中至少必须有一个点。

<p><font face="constant-width" color="#000000" size=3>maxIter</font></p> 
Total number of iterations over the data before stopping. Changing this probably won’t change your results a ton, so don’t make this the first parameter you look to adjust. The default is 20.

停止之前，数据上的迭代总数。更改此设置可能不会使您的结果大为改变，因此请不要将其设为您要调整的第一个参数。默认值为20。

Most of the parameters in this model should be tuned in order to find the best result. There’s no rule that applies to all datasets.

该模型中的大多数参数都应进行调整以找到最佳结果。没有适用于所有数据集的规则。

### <font color="#00000">Example</font>

```scala
// in Scala
import org.apache.spark.ml.clustering.BisectingKMeans
val bkm = new BisectingKMeans().setK(5).setMaxIter(5)
println(bkm.explainParams())
val bkmModel = bkm.fit(sales)
```

```python
# in Python
from pyspark.ml.clustering import BisectingKMeans
bkm = BisectingKMeans().setK(5).setMaxIter(5)
bkmModel = bkm.fit(sales)
```

### <font color="#00000">Bisecting k-means Summary 二等分k-means简述</font>

Bisecting -means includes a summary class that we can use to evaluate our model, that is largely the same as the -means summary. This includes information about the clusters created, as well as their relative sizes (number of examples):

二等分-means包括一个摘要类，我们可以使用该类来评估我们的模型，该类与-means摘要大致相同。这包括有关创建的集群及其相对大小（示例数）的信息：

```scala
// in Scala
val summary = bkmModel.summary
summary.clusterSizes // number of pointskmModel.computeCost(sales)
println("Cluster Centers: ")
kmModel.clusterCenters.foreach(println)
```

```python
# in Python
summary = bkmModel.summary
print summary.clusterSizes # number of points
kmModel.computeCost(sales)
centers = kmModel.clusterCenters()
print("Cluster Centers: ")
for center in centers:
print(center)
```

## <font color="#9a161a">Gaussian Mixture Models</font>

Gaussian mixture models (GMM) are another popular clustering algorithm that makes different assumptions than bisecting -means or -means do. Those algorithms try to group data by reducing the sum of squared distances from the center of the cluster. Gaussian mixture models, on the other hand, assume that each cluster produces data based upon random draws from a Gaussian distribution. This means that clusters of data should be less likely to have data at the edge of the cluster (reflected in the Guassian distribution) and much higher probability of having data in the center. Each Gaussian cluster can be of arbitrary size with its own mean and standard deviation (and hence a possibly different, ellipsoid shape). There are still user-specified clusters that will be created during training.

高斯混合模型（GMM）是另一种流行的聚类算法，与将-means或-means分为两等分相比，它做出了不同的假设。这些算法尝试通过减少距群集中心的平方距离之和来对数据进行分组。另一方面，高斯混合模型假设每个聚类基于来自高斯分布的随机抽取生成数据。这意味着数据集群在集群边缘的数据（在高斯分布中反映）的可能性应该较小，而在中心拥有数据的可能性则更高。每个高斯聚类可以具有任意大小，具有自己的均值和标准差（因此可能是不同的椭圆形）。在培训期间仍将创建用户指定的群集。 

A simplified way of thinking about Gaussian mixture models is that they’re like a soft version of means. -means creates very rigid clusters—each point is only within one cluster. GMMs allow for a more nuanced cluster associated with probabilities, instead of rigid boundaries.

考虑高斯混合模型的一种简化方法是，它们就像均值的软版本。-means创建非常严格的群集-每个点仅在一个群集内。GMM允许与概率相关的更细微的簇，而不是严格的边界。

For more information, see <u><a style="color:#0879e3" href="http://faculty.marshall.usc.edu/gareth-james/">ISL</a></u> 14.3. 

更多信息，请查看 <u><a style="color:#0879e3" href="http://faculty.marshall.usc.edu/gareth-james/">ISL</a></u> 14.3.

### <font color="#00000">Model Hyperparameters 模型参数</font>

These are configurations that we specify to determine the basic structure of the model:

我们指定这些配置来确定模型的基本结构： 

​	This is the number of clusters that you would like to end up with.

​	这是您最终希望使用的集群数量。

### <font color="#00000">Training Parameters 训练参数</font>

<p><font face="constant-width" color="#000000" size=3>maxIter</font></p>
Total number of iterations over the data before stopping. Changing this probably won’t change your results a ton, so don’t make this the first parameter you look to adjust. The default is 100. 
停止之前，数据上的迭代总数。更改此设置可能不会使您的结果大为改变，因此请不要将其设为您要调整的第一个参数。默认值为100。

<p><font face="constant-width" color="#000000" size=3>tol</font></p>
This value simply helps us specify a threshold by which changes in parameters show that we optimized our weights enough. A smaller value can lead to higher accuracy at the cost of performing more iterations (although never more than `maxIter`). 	The default value is 0.01. 
该值只是简单地帮助我们指定一个阈值，通过该阈值参数的变化表明我们已经充分优化了权重。较小的值可以以执行更	多迭代为代价提高精度（尽管绝不超过`maxIter`）。默认值为0.01。



As with our -means model, these training parameters are less likely to have an impact than the number of clusters, .

与我们的-means模型一样，这些训练参数产生影响的可能性要小于聚类的数量。

### <font color="#00000">Example</font>

```scala
// in Scala
import org.apache.spark.ml.clustering.GaussianMixture
val gmm = new GaussianMixture().setK(5)
println(gmm.explainParams())
val model = gmm.fit(sales)
```

```python
# in Python
from pyspark.ml.clustering import GaussianMixture
gmm = GaussianMixture().setK(5)
print gmm.explainParams()
model = gmm.fit(sales)
```

### <font color="#00000">Gaussian Mixture Model Summary 高斯混合模型简述</font>

Like our other clustering algorithms, Gaussian mixture models include a summary class to help with model evaluation. This includes information about the clusters created, like the weights, the means, and the covariance of the Gaussian mixture, which can help us learn more about the underlying structure inside of our data:

与我们的其他聚类算法一样，高斯混合模型包括摘要类，以帮助模型评估。这包括有关创建的聚类的信息，例如权重，均值和高斯混合的协方差，这些信息可以帮助我们进一步了解数据内部的基础结构：

```scala
// in Scala
val summary = model.summary
model.weights
model.gaussiansDF.show()
summary.cluster.show()
summary.clusterSizes
summary.probability.show()
```

```python
# in Python
summary = model.summary
print model.weights
model.gaussiansDF.show()
summary.cluster.show()
summary.clusterSizes
summary.probability.show()
```

## <font color="#9a161a">Latent Dirichlet Allocation 隐式狄利克雷分布</font>

*Latent Dirichlet Allocation* (LDA) is a hierarchical clustering model typically used to perform topic modelling on text documents. LDA tries to extract high-level topics from a series of documents and keywords associated with those topics. It then interprets each document as having a variable number of contributions from multiple input topics. There are two implementations that you can use: online LDA and expectation maximization. In general, online LDA will work better when there are more examples, and the expectation maximization optimizer will work better when there is a larger input vocabulary. This method is also capable of scaling to hundreds or thousands of topics.

潜在狄利克雷分配（LDA）是一种层次结构的聚类模型，通常用于对文本文档执行主题建模。LDA尝试从一系列与这些主题相关的文档和关键字中提取高级主题。然后，它将每个文档解释为具有来自多个输入主题的不同数量的文稿。您可以使用两种实现：在线LDA和期望最大化。通常，当有更多示例时，在线LDA会更好地工作；而在输入词汇量更大的情况下，期望最大化优化器也将更好地工作。此方法还可以扩展到数百或数千个主题。

To input our text data into LDA, we’re going to have to convert it into a numeric format. You can use the `CountVectorizer` to achieve this. 

要将文本数据输入到LDA中，我们必须将其转换为数字格式。您可以使用`CountVectorizer`来实现。

### <font color="#00000">Model Hyperparameters 模型参数</font>

These are configurations that we specify to determine the basic structure of the model:

我们指定这些配置来确定模型的基本结构：

​	The total number of topics to infer from the data. The default is 10 and must be a positive number.

​	从数据推断出的主题总数。默认值为10，并且必须为正数。

<p><font face="constant-width" color="#000000" size=3>docConcentration</font></p> 
Concentration parameter (commonly named “alpha”) for the prior placed on documents’ distributions over topics (“theta”). This is the parameter to a Dirichlet distribution, where larger values mean more smoothing (more regularization).
优先级的浓度参数（通常称为“ alpha”）放在文档的主题分布（“ theta”）上。这是Dirichlet分布的参数，其中较大的值表示更平滑（更规则化）。

If not set by the user, then docConcentration is set automatically. If set to singleton vector [alpha], then alpha is replicated to a vector of length k in fitting. Otherwise, the docConcentration vector must be length .
如果用户未设置，则将自动设置docConcentration。如果设置为单例向量α，则将α复制到拟合中长度为k的向量。否则，docConcentration向量必须为length。


如果用户未设置，则将自动设置docConcentration。如果设置为单例向量α，则将α复制到拟合中长度为k的向量。否则，docConcentration向量必须为length。

<p><font face="constant-width" color="#000000" size=3>topicConcentration</font></p> 
The concentration parameter (commonly named “beta” or “eta”) for the prior placed on a topic’s distributions over terms. This is the parameter to a symmetric Dirichlet distribution. If not set by the user, then topicConcentration is set automatically.
优先事项的浓度参数（通常称为“ beta”或“ eta”），位于主题的各个术语的分布中。这是对称Dirichlet分布的参数。如果用户未设置，则topicConcentration会自动设置。

### <font color="#00000">Training Parameters 训练参数</font>

<p><font face="constant-width" color="#000000" size=3>maxIter</font></p> 
Total number of iterations over the data before stopping. Changing this probably won’t change your results a ton, so don’t make this the first parameter you look to adjust. The default is 20.
停止之前，数据上的迭代总数。更改此设置可能不会使您的结果大为改变，因此请不要将其设为您要调整的第一个参数。默认值为20。

<p><font face="constant-width" color="#000000" size=3>optimizer</font></p> 
This determines whether to use EM or online training optimization to determine the LDA model. The default is online.
这确定是使用EM还是在线培训优化来确定LDA模型。默认为在线。

<p><font face="constant-width" color="#000000" size=3>learningDecay</font></p> 
Learning rate, set as an exponential decay rate. This should be between (0.5, 1.0] to guarantee asymptotic convergence. The default is 0.51 and only applies to the online optimizer.
学习率，设置为指数衰减率。此值应介于（0.5，1.0]之间以确保渐近收敛。默认值为0.51，仅适用于在线优化器。

<p><font face="constant-width" color="#000000" size=3>learningOffset</font></p> 
A (positive) learning parameter that downweights early iterations. Larger values make early iterations count less. The default is 1,024.0 and only applies to the online optimizer.
一个（正）学习参数，可以减轻早期迭代的负担。较大的值使早期迭代的计数减少。默认值为1,024.0，仅适用于在线优化器。

<p><font face="constant-width" color="#000000" size=3>optimizeDocConcentration</font></p> 
Indicates whether the docConcentration (Dirichlet parameter for document-topic distribution) will be optimized during training. The default is true but only applies to the online optimizer.

指示在培训期间是否将优化docConcentration（用于文档主题分发的Dirichlet参数）。默认值为true，但仅适用于在线优化器。

<p><font face="constant-width" color="#000000" size=3>subsamplingRate</font></p> 
The fraction of the corpus to be sampled and used in each iteration of mini-batch gradient descent, in range (0, 1]. The default is 0.5 and only applies to the online optimizer.
在小批量梯度下降的每次迭代中要采样和使用的语料库分数，范围为（0，1]。默认值为0.5，仅适用于在线优化器。

<p><font face="constant-width" color="#000000" size=3>seed</font></p> 
This model also supports specifying a random seed for reproducibility.
该模型还支持指定可重复性的随机种子。

<p><font face="constant-width" color="#000000" size=3>checkpointInterval</font></p> 
This is the same checkpoint feature that we saw in Chapter 26.

这是我们在第26章中看到的相同的检查点功能。

### <font color="#00000">Prediction Parameters 预测参数</font>

<p><font face="constant-width" color="#000000" size=3>topicDistributionCol</font></p>
The column that will hold the output of the topic mixture distribution for each document.
该列将保存每个文档的主题混合分布的输出。

### <font color="#00000">Example</font>

```scala
// in Scala
import org.apache.spark.ml.feature.{Tokenizer, CountVectorizer}
val tkn = new Tokenizer().setInputCol("Description").setOutputCol("DescOut")
val tokenized = tkn.transform(sales.drop("features"))
val cv = new CountVectorizer()
.setInputCol("DescOut")
.setOutputCol("features")
.setVocabSize(500)
.setMinTF(0)
.setMinDF(0)
.setBinary(true)
val cvFitted = cv.fit(tokenized)
val prepped = cvFitted.transform(tokenized)
```

```python
# in Python
from pyspark.ml.feature import Tokenizer, CountVectorizer
tkn = Tokenizer().setInputCol("Description").setOutputCol("DescOut")tokenized = tkn.transform(sales.drop("features"))
cv = CountVectorizer()\
.setInputCol("DescOut")\
.setOutputCol("features")\
.setVocabSize(500)\
.setMinTF(0)\
.setMinDF(0)\
.setBinary(True)
cvFitted = cv.fit(tokenized)
prepped = cvFitted.transform(tokenized)
```

```scala
// in Scala
import org.apache.spark.ml.clustering.LDA
val lda = new LDA().setK(10).setMaxIter(5)
println(lda.explainParams())
val model = lda.fit(prepped)
```

```python
# in Python
from pyspark.ml.clustering import LDA
lda = LDA().setK(10).setMaxIter(5)
print lda.explainParams()
model = lda.fit(prepped)
```

After we train the model, you will see some of the top topics. This will return the term indices, and we’ll have to look these up using the `CountVectorizerModel` that we trained in order to find out the true words. For instance, when we trained on the data our top 3 topics were hot, home, and brown after looking them up in our vocabulary:

训练模型后，您将看到一些热门话题。这将返回术语索引，我们必须使用我们训练的`CountVectorizerModel`来查找这些索引，以便找出真实的单词。例如，当我们对数据进行培训时，在我们的词汇表中查找它们之后，我们的前3个主题是热门，家和棕色：

```scala
// in Scala
model.describeTopics(3).show()
cvFitted.vocabulary
```

```python
# in Python
model.describeTopics(3).show()
cvFitted.vocabulary
```

These methods result in detailed information about the vocabulary used as well as the emphasis on particular terms. These can be helpful for better understanding the underlying topics. Due to space constraints, we can’t show this output. Using similar APIs, we can get some more technical measures like the log likelihood and perplexity. The goal of these tools is to help you optimize the number of topics, based on your data. When using perplexity in your success criteria, you should apply these metrics to a holdout set to reduce the overall perplexity of the model. Another option is to optimize to increase the log likelihood value on the holdout set. We can calculate each of these by passing a dataset into the following functions: `model.logLikelihood` and `model.logPerplexity`.

这些方法可提供有关所用词汇的详细信息以及对特定术语的强调。这些有助于更好地理解基础主题。由于篇幅所限，我们无法显示此输出。使用类似的API，我们可以获得更多技术指标，例如对数可能性和困惑度。这些工具的目的是帮助您根据数据优化主题数。在成功标准中使用困惑度时，应将这些指标应用于保留集，以减少模型的总体困惑度。另一个选择是优化以增加保留集上的对数似然值。我们可以通过将数据集传递给以下函数来计算每个参数：`model.logLikelihood`和`model.logPerplexity`。

## <font color="#9a161a">Conclusion 总结</font>

This chapter covered the most popular algorithms that Spark includes for unsupervised learning. The next chapter will bring us out of MLlib and talk about some of the advanced analytics ecosystem that has grown outside of Spark. 

本章介绍了Spark包含的用于无监督学习的最受欢迎的算法。下一章将使我们脱离MLlib，并讨论一些Spark以外的高级分析生态系统。
