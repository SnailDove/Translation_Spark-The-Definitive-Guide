---
title: 翻译 Chapter 26 Classification
date: 2019-09-01
copyrig ht: true
categories: English
tags: [Spark]
mathjax: true
mathjax2: true
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 26 Classification 分类

Classification is the task of predicting a label, category, class, or discrete variable given some input features. The key difference from other ML tasks, such as regression, is that the output label has a finite set of possible values (e.g., three classes).

分类是在给定一些输入特征的情况下预测标签，类别，类或离散变量的任务。与其他ML任务（例如回归）的主要区别在于输出标签具有一组有限的可能值（例如，三个类）。

## <font color="#9a161a">Use Cases 用户案例</font>

Classification has many use cases, as we discussed in Chapter 24. Here are a few more to consider as a reinforcement of the multitude of ways classification can be used in the real world.

分类有许多用户案例，正如我们在第24章中讨论的那样。这里还有一些需要考虑的因素，可以加强分类在现实世界中的使用方式。

**Predicting credit risk 预测信用风险** 

A financing company might look at a number of variables before offering a loan to a company or individual. Whether or not to offer the loan is a binary classification problem. 

在向公司或个人提供贷款之前，融资公司可能会考虑许多变量。是否提供贷款是二元分类问题。

**News classification 新闻分类** 

An algorithm might be trained to predict the topic of a news article (sports, politics, business, etc.).

可以训练算法来预测新闻文章（体育，政治，商业等）的主题。

**Classifying human activity 对人类活动进行分类**

By collecting data from sensors such as a phone accelerometer or smart watch, you can predict the person’s activity. The output will be one of a finite set of classes (e.g., walking, sleeping, standing, or running).

通过从传感器（如手机加速度计或智能手表）收集数据，您可以预测人员的活动。输出将是一组有限的类（例如，步行，睡觉， 站立或跑步）。

## <font color="#9a161a">Types of Classification 分类算法的类别</font>

Before we continue, let’s review several different types of classification. 

在继续之前，让我们回顾几种不同类型的分类。

### <font color="#00000">Binary Classification 二元分类</font>

The simplest example of classification is binary classification, where there are only two labels you can predict. One example is fraud analytics, where a given transaction can be classified as fraudulent or not; or email spam, where a given email can be classified as spam or not spam.

最简单的分类示例是二元分类，其中只有两个标签可以预测。一个例子是欺诈分析，其中给定的交易可以被分类为欺诈性或非欺诈性;或电子邮件垃圾邮件，其中给定的电子邮件可分类为垃圾邮件或非垃圾邮件。

### <font color="#00000">Multiclass Classification 多类别的分类</font>

Beyond binary classification lies multiclass classification, where one label is chosen from more than two distinct possible labels. A typical example is Facebook predicting the people in a given photo or a meterologist predicting the weather (rainy, sunny, cloudy, etc.). Note how there is always a finite set of classes to predict; it’s never unbounded. This is also called multinomial classification.

除了二元分类之外，还有多类分类，其中一个标签是从两个以上不同的可能标签中选择的。一个典型的例子是Facebook预测给定照片中的人或预测天气（雨天，晴天，阴天等）的气象学家。注意如何去预测总是有一组有限的类来；它永远不会无限制。这也称为多项分类。

### <font color="#00000">Multilabel Classification 多标签的分类</font>

Finally, there is multilabel classification, where a given input can produce multiple labels. For example, you might want to predict a book’s genre based on the text of the book itself. While this could be multiclass, it’s probably better suited for multilabel because a book may fall into multiple genres. Another example of multilabel classification is identifying the number of objects that appear in an image. Note that in this example, the number of output predictions is not necessarily fixed, and could vary from image to image.

最后，存在多标签分类，其中给定输入可以产生多个标签。例如，您可能希望根据书本身的文本来预测书籍的类型。虽然这可能是多类的，但它可能更适合多标签，因为一本书可能属于多种类型。多标签分类的另一个例子是识别出现在图像中的对象的数量。请注意，在此示例中，输出预测的数量不一定是固定的，并且可能因图像而异。

## <font color="#9a161a">Classification Models in MLlib 在MLlib中的分类模型</font>

Spark has several models available for performing binary and multiclass classification out of the box. The following models are available for classification in Spark : 

Spark有几种可用于执行二元和多分类的模型，这些模型开箱即用。Spark中可以使用以下模型进行分类：

- Logistic regression 逻辑回归
- Decision trees 决策树
- Random forests 随机森林
- Gradient-boosted trees 梯度提升树

Spark does not support making multilabel predictions natively. In order to train a multilabel model, you must train one model per label and combine them manually. Once manually constructed, there are built-in tools that support measuring these kinds of models (discussed at the end of the chapter). 

Spark不支持原生进行多标签预测。为了训练多标签模型， 您必须为每个标签训练一个模型并手动组合它们。一旦手动构建，就有内置工具支持测量这些模型（在本章末尾讨论）。

This chapter will cover the basics of each of these models by providing:

本章将通过提供以下内容介绍每种模型的基础知识：

- A simple explanation of the model and the intuition behind it 

    模型的简单解释及其背后的直觉

- Model hyperparameters (the different ways we can initialize the model)

    模型超参数（我们可以初始化模型的不同方式）

- Training parameters (parameters that affect how the model is trained)

    训练参数（影响模型训练方式的参数）

- Prediction parameters (parameters that affect how predictions are made)

    预测参数（影响预测方式的参数）

You can set the hyperparameters and training parameters in a `ParamGrid` as we saw in Chapter 24.

您可以在第24章中看到，在 `ParamGrid` 中设置超参数和训练参数。

### <font color="#00000">Model Scalability 模型伸缩性</font>

Model scalability is an important consideration when choosing your model. In general, Spark has great support for training large-scale machine learning models (note, these are large scale; on single node workloads there are a number of other tools that also perform well). Table 26-1 is a simple model scalability scorecard to use to find the best model for your particular task (if scalability is your core consideration). The actual scalability will depend on your configuration, machine size, and other specifics but should make for a good heuristic.

选择模型时，模型可伸缩性是一个重要的考虑因素。总的来说，Spark非常支持训练大型机器学习模型（注意，这些是大规模的;在单节点工作负载上，还有许多其他工具也表现良好）。表26-1是一个简单的模型可伸缩性记分卡，用于查找特定任务的最佳模型（如果可扩展性是您的核心考虑因素）。实际的可扩展性将取决于您的配置，机器数量和其他细节，但应该是一个良好的启发式。

Table 26-1. Model scalability reference 模型伸缩性参考表

| Model<br />模型                        | Features count<br />特征计数 | Training examples<br />训练例子 | Output classes<br />输出类别    |
| -------------------------------------- | ---------------------------- | ------------------------------- | ------------------------------- |
| Logistic regression<br />逻辑回归      | 1 to 10 million              | No limit                        | Features x Classes < 10 million |
| Decision trees<br />决策树             | 1,000s                       | No limit                        | Features x Classes < 10,000s    |
| Random forest 10,000s<br />随机森林    | 10,000s                      | No limit                        | Features x Classes < 100,000s   |
| Gradient-boosted trees<br />梯度提升树 | 1,000s                       | No limit                        | Features x Classes < 10,000s    |

We can see that nearly all these models scale to large collections of input data and there is ongoing work to scale them even further. The reason no limit is in place for the number of training examples is because these are trained using methods like stochastic gradient descent and L-BFGS. These methods are optimized specifically for working with massive datasets and to remove any constraints that might exist on the number of training examples you would hope to learn on.

我们可以看到，几乎所有这些模型都可以扩展到输入数据的大量集合，并且正在进行进一步扩展它们的工作。 对训练样本数量没有限制的原因是因为这些是使用随机梯度下降和 L-BFGS 等方法训练的。 这些方法专门优化用于处理大量数据集，并移除可能存在于您希望学习的训练示例数量上的任何约束。

Let’s start looking at the classification models by loading in some data: 

让我们开始通过加载一些数据来查看分类模型：

```scala
// in Scala
val bInput = spark.read.format("parquet").load("/data/binary-classification").
selectExpr("features", "cast(label as double) as label")
```

```python
# in Python
bInput = spark.read.format("parquet").load("/data/binary-classification")\
.selectExpr("features", "cast(label as double) as label")
```

---

<center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong></font></center>
Like our other advanced analytics chapters, this one cannot teach you the mathematical underpinnings of every model. See Chapter 4 in [ISL](http://faculty.marshall.usc.edu/gareth-james/) and [ESL](http://statweb.stanford.edu/~tibs/ElemStatLearn/) for a review of classification. 

像我们的其他高级分析章节一样，这个章节不能教你每个模型的数学基础。 有关分类的评论，请参阅 [ISL](http://faculty.marshall.usc.edu/gareth-james/) 和 [ESL](http://statweb.stanford.edu/~tibs/ElemStatLearn/) 中的第4章。

---

## <font color="#9a161a">Logistic Regression Logistic回归</font>

Logistic regression is one of the most popular methods of classification. It is a linear method that combines each of the individual inputs (or features) with specific weights (these weights are generated during the training process) that are then combined to get a probability of belonging to a particular class. These weights are helpful because they are good representations of feature importance; if you have a large weight, you can assume that variations in that feature have a significant effect on the outcome (assuming you performed normalization). A smaller weight means the feature is less likely to be important.

逻辑回归是最流行的分类方法之一。它是一种线性方法，将每个单独的输入（或特征）与特定权重（这些权重在训练过程中生成）组合在一起，然后将这些权重组合起来以获得属于特定类别的概率。这些权重是有用的，因为它们是特征重要性的良好表示；如果你的权重很大，你可以假设该特征的变化对结果有显着影响（假设你进行了标准化）。较小的权重意味着该特征不太可能重要。

See [ISL 4.3](http://faculty.marshall.usc.edu/gareth-james/) and [ESL 4.4](http://statweb.stanford.edu/~tibs/ElemStatLearn/) for more information.

有关更多信息，请参阅 [ISL 4.3](http://faculty.marshall.usc.edu/gareth-james/) 和 [ESL 4.4](http://statweb.stanford.edu/~tibs/ElemStatLearn/)。

### <font color="#00000">Model Hyperparameters 模型超参数</font>

Model hyperparameters are configurations that determine the basic structure of the model itself. The following hyperparameters are available for logistic regression:

模型超参数是确定模型本身的基本结构的配置项。以下超参数可用于逻辑回归：

<font face="constant-width" color="#000000" size=4>family</font>

Can be multinomial (two or more distinct labels; multiclass classification) or binary (only two distinct labels; binary classification).

可以是多项式（两个或多个不同的标签;多类分类）或二元（仅两个不同的标签;二元分类）。

<font face="constant-width" color="#000000" size=4>elasticNetParam</font>

A floating-point value from 0 to 1. This parameter specifies the mix of L1 and L2 regularization according to elastic net regularization (which is a linear combination of the two). Your choice of L1 or L2 depends a lot on your particular use case but the intuition is as follows: L1 regularization (a value of 1) will create sparsity in the model because certain feature weights will become zero (that are of little consequence to the output). For this reason, it can be used as a simple feature-selection method. On the other hand, L2 regularization (a value of 0) does not create sparsity because the corresponding weights for particular features will only be driven toward zero, but will never completely reach zero. ElasticNet gives us the best of both worlds—we can choose a value between 0 and 1 to specify a mix of L1 and L2 regularization. For the most part, you should be tuning this by testing different values.

从0到1的浮点值。该参数根据弹性网络正则化（两者的线性组合）指定L1和L2正则化的混合。您对L1或L2的选择很大程度上取决于您的特定使用案例，但直觉如下：L1正则化（值为1）将在模型中产生稀疏性，因为某些特征权重将变为零（这对于输出）。因此，它可以用作简单的特征选择方法。另一方面，L2正则化（值为0）不会产生稀疏性，因为特定特征的相应权重将仅被驱动为零，但永远不会完全达到零。 ElasticNet 为我们提供了两全其美的优势——我们可以选择0到1之间的值来指定L1和L2正则化的混合。在大多数情况下，您应该通过测试不同的值来调整它。

<font face="constant-width" color="#000000" size=4>fitIntercept</font>

Can be true or false. This hyperparameter determines whether or not to fit the intercept or the arbitrary number that is added to the linear combination of inputs and weights of the model. Typically you will want to fit the intercept if we haven’t normalized our training data.

可以是真是假。该超参数确定是否拟合截距或添加到模型的输入和权重的线性组合的任意数。通常，如果我们没有标准化训练数据，您将需要拟合截距。

<font face="constant-width" color="#000000" size=4>regParam</font>

A value ≥ 0. that determines how much weight to give to the regularization term in the objective function. Choosing a value here is again going to be a function of noise and dimensionality in our dataset. In a pipeline, try a wide range of values (e.g., 0, 0.01, 0.1, 1).

值≥0，用于确定目标函数中正则化项的权重。在这里选择一个值将再次成为我们数据集中噪声和维度的函数。在管道中，尝试各种值（例如，0,0.01,0.1,1）。

<font face="constant-width" color="#000000" size=4>standardization</font>

Can be true or false, whether or not to standardize the inputs before passing them into the model.
See Chapter 25 for more information.

无论是否在将输入传递到模型之前对输入进行标准化，都可以是真或假。有关更多信息，请参见第25章。

### <font color="#00000">Training Parameters 训练参数</font>

Training parameters are used to specify how we perform our training. Here are the training parameters for logistic regression.

训练参数用于指定我们如何执行训练。以下是逻辑回归的训练参数。

<font face="constant-width" color="#000000" size=4>maxIter</font>

Total number of iterations over the data before stopping. Changing this parameter probably won’t change your results a ton, so it shouldn’t be the first parameter you look to adjust. The default is 100.

停止前数据的迭代总数。更改此参数可能不会大幅度改变您的结果，因此它不应该是您要调整的第一个参数。默认值为100。

<font face="constant-width" color="#000000" size=4>tol</font>

This value specifies a threshold by which changes in parameters show that we optimized our weights enough, and can stop iterating. It lets the algorithm stop before `maxIter` iterations. The default value is $1.0E-6$. This also shouldn’t be the first parameter you look to tune.

此值指定一个阈值，通过该阈值，参数的变化表明我们已经足够优化了权重，并且可以停止迭代。它允许算法在`maxIter` 迭代之前停止。默认值为$1.0E-6$。这也不应该是你想要调整的第一个参数。

<font face="constant-width" color="#000000" size=4>weightCol</font>

The name of a weight column used to weigh certain rows more than others. This can be a useful tool if you have some other measure of how important a particular training example is and have a weight associated with it. For example, you might have 10,000 examples where you know that some labels are more accurate than others. You can weigh the labels you know are correct more than the ones you don’t.

用于比其他行加更多权重的一些行的权重列名称（**译者注**：每个样本的权重不一样，这里权重列指的是样本的权重向量）。如果您对特定训练示例的重要程度以及与之相关的权重有其他衡量标准，那么这可能是一个有用的工具。例如，您可能有10,000个示例，其中您知道某些标签比其他标签更准确。您可以对您知道的准确的标签比您不知道的标签加更多权重。

### <font color="#00000">Prediction Parameters 预测参数</font>

These parameters help determine how the model should actually be making predictions at prediction time, but do not affect training. Here are the prediction parameters for logistic regression:

这些参数有助于确定模型在预测时应该如何实际进行预测，但不会影响训练。以下是逻辑回归的预测参数：

<font face="constant-width" color="#000000" size=4>threshold</font>

A Double in the range of 0 to 1. This parameter is the probability threshold for when a given class should be predicted. You can tune this parameter according to your requirements to balance between false positives and false negatives. For instance, if a mistaken prediction would be costly—you might want to make its prediction threshold very high.

双精度范围为0到1，此参数是应该预测给定类的概率阈值。您可以根据您的要求调整此参数，以平衡预测阳性但是预测错误的样本（假阳）和预测阴性的但是预测错误的样本（假阴）。例如，如果错误的预测成本很高——您可能希望将其预测阈值设置得非常高。

<font face="constant-width" color="#000000" size=4>thresholds</font>

This parameter lets you specify an array of threshold values for each class when using multiclass classification. It works similarly to the single threshold parameter described previously.

使用多类分类时，此参数允许您为每个类指定阈值数组。它的工作方式类似于前面描述的单个阈值参数。

### <font color="#00000">Example</font>

Here’s a simple example using the `LogisticRegression` model. Notice how we didn’t specify any parameters because we’ll leverage the defaults and our data conforms to the proper column naming. In practice, you probably won’t need to change many of the parameters: 

这是使用 `LogisticRegression` 模型的简单示例。请注意我们没有指定任何参数，因为我们将利用默认值并且我们的数据符合正确的列命名。实际上，您可能不需要更改许多参数：

```scala
// in Scala
import org.apache.spark.ml.classification.LogisticRegression
val lr = new LogisticRegression()
println(lr.explainParams()) // see all parameters
val lrModel = lr.fit(bInput)
```

```python
# in Python
from pyspark.ml.classification import LogisticRegression
lr = LogisticRegression()
print lr.explainParams() # see all parameters
lrModel = lr.fit(bInput)
```

Once the model is trained you can get information about the model by taking a look at the coefficients and the intercept. The coefficients correspond to the individual feature weights (each feature weight is multiplied by each respective feature to compute the prediction) while the intercept is the value of the italics-intercept (if we chose to fit one when specifying the model). Seeing the coefficients can be helpful for inspecting the model that you built and comparing how features affect the prediction:

训练模型后，您可以通过查看系数和截距来获得有关模型的信息。系数对应于各个特征权重（每个特征权重乘以每个相应的特征以计算预测），而截距是斜体截距的值（如果我们在指定模型时选择拟合一个）。查看系数有助于检查您构建的模型并比较特征如何影响预测：

```scala
// in Scala
println(lrModel.coefficients)
println(lrModel.intercept)
```

```python
# in Python
print lrModel.coefficients
print lrModel.intercept
```

For a multinomial model (the current one is binary), `lrModel.coefficientMatrix` and `lrModel.interceptVector` can be used to get the coefficients and intercept. These will return Matrix and Vector types representing the values or each of the given classes.

对于多项模型（当前的二元模型），可以使用 `lrModel.coefficientMatrix` 和 `lrModel.interceptVector` 来获取系数和截距。这些将返回表示值或每个给定类的 Matrix 和 Vector 类型。

### <font color="#00000">Model Summary 模型摘要</font>

Logistic regression provides a model summary that gives you information about the final, trained model. This <font color="#32cd32"><strong>is analogous to</strong></font> the same types of summaries we see in many R language machine learning packages. The model summary is currently only available for binary logistic regression problems, but multiclass summaries will likely be added in the future. Using the binary summary, we can get all sorts of information about the model itself including the area under the ROC curve, the f measure by threshold, the precision, the recall, the recall by thresholds, and the ROC curve. Note that for the area under the curve, instance weighting is not taken into account, so if you wanted to see how you performed on the values you weighed more highly, you’d have to do that manually. This will probably change in future Spark versions. You can see the summary using the following APIs:

Logistic 回归提供了一个模型摘要，为您提供有关最终训练模型的信息。这类似于我们在许多R语言机器学习包中看到的相同类型的摘要。模型摘要目前仅适用于二元逻辑回归问题，但将来可能会添加多类摘要。使用二元汇总，我们可以获得有关模型本身的各种信息，包括 ROC 曲线下的面积，阈值的 f 度量，精度，召回率，阈值召回率和ROC 曲线。请注意，对于曲线下方的区域，不考虑实例权重，因此如果您想要查看权重更高的值的执行情况，则必须手动执行此操作。这可能会在未来的Spark版本中发生变化。您可以使用以下API查看摘要：

```scala
// in Scala
import org.apache.spark.ml.classification.BinaryLogisticRegressionSummary
val summary = lrModel.summary
val bSummary = summary.asInstanceOf[BinaryLogisticRegressionSummary]
println(bSummary.areaUnderROC)
bSummary.roc.show()
bSummary.pr.show()
```

```python
# in Python
summary = lrModel.summary
print summary.areaUnderROC
summary.roc.show()
summary.pr.show()
```

The speed at which the model descends to the final result is shown in the objective history. We can access this through the objective history on the model summary :

模型下降到最终结果的速度显示在目标历史中。我们可以通过模型摘要的目标历史来访问它：

```scala
summary.objectiveHistory
```

This is an array of doubles that specify how, over each training iteration, we are performing with respect to our objective function. This information is helpful to see if we have sufficient iterations or need to be tuning other parameters. 

这是一个双精度数组，用于指定在每次训练迭代中我们对目标函数的执行方式。此信息有助于查看是否有足够的迭代或需要调整其他参数。

## <font color="#9a161a">Decision Trees</font>

Decision trees are one of the more friendly and interpretable models for performing classification because they’re similar to simple decision models that humans use quite often. For example, if you have to predict whether or not someone will eat ice cream when offered, a good feature might be whether or not that individual likes ice cream. In pseudocode, if `person.likes(“ice_cream”)`, they will eat ice cream; otherwise, they won’t eat ice cream. A decision tree creates this type of structure with all the inputs and follows a set of branches when it comes time to make a prediction. This makes it a great starting point model because it’s easy to reason about, easy to inspect, and makes very few assumptions about the structure of the data. In short, rather than trying to train coeffiecients in order to model a function, it simply creates a big tree of decisions to follow at prediction time. This model also supports multiclass classification and provides outputs as predictions and probabilities in two different columns.

决策树是用于执行分类的更友好且可解释的模型之一，因为它们类似于人类经常使用的简单决策模型。 例如，如果您必须预测某人是否会在提供冰淇淋时吃冰淇淋，那么一个好的特征可能就是这个人是否喜欢冰淇淋。在伪代码中，如果是 `person.likes(“ice_cream”)`，他们会吃冰淇淋; 否则，他们不会吃冰淇淋。决策树使用所有输入创建此类结构，并在进行预测时遵循一组分支。 这使它成为一个很好的起点模型，因为它易于推理，易于检查，并且对数据结构做出很少的假设。简而言之，它不是试图训练系数来模拟一个函数，而是简单地在预测时创建一个大的决策树。 该模型还支持多类分类，并在两个不同的列中提供输出作为预测和概率。

While this model is usually a great start, it does come at a cost. It can overfit data extremely quickly. By that we mean that, unrestrained, the decision tree will create a pathway from the start based on every single training example. That means it encodes all of the information in the training set in the model. This is bad because then the model won’t generalize to new data (you will see poor test set prediction performance). However, there are a number of ways to try and rein in the model by limiting its branching structure (e.g., limiting its height) to get good predictive power.

虽然这种模式通常是一个很好的开始，但确实需要付出代价。它可以非常快速地过度拟合数据。我们的意思是，无拘无束，决策树将从一开始就根据每个训练样例创建一条路径。这意味着它会对模型中训练集中的所有信息进行编码。这很糟糕，因为那时模型不会泛化到新数据（您将看到不良的测试集预测性能）。然而，有许多方法可以通过限制其分支结构（例如，限制其高度）来尝试并控制模型以获得良好的预测能力。

See [ISL 8.1](http://faculty.marshall.usc.edu/gareth-james/) and [ESL 9.2](https://web.stanford.edu/~hastie/ElemStatLearn/) for more information.

有关更多信息，请参见 [ISL](http://faculty.marshall.usc.edu/gareth-james/)  8.1 和 [ESL](https://web.stanford.edu/~hastie/ElemStatLearn/) 9.2。

### <font color="#00000">Model Hyperparameters 模型超参数</font>

There are many different ways to configure and train decision trees. Here are the hyperparameters that Spark’s implementation supports : 

有许多不同的方法来配置和训练决策树。以下是Spark实现支持的超参数：

<font face="constant-width" color="#000000" size=4>maxDepth</font>

Since we’re training a tree, it can be helpful to specify a max depth in order to avoid overfitting to the dataset (in the extreme, every row ends up as its own leaf node). The default is 5.

由于我们正在训练树，因此指定最大深度以避免过度拟合数据集会很有帮助（在极端情况下，每一行最终都是自己的叶节点）。默认值为 5。

<font face="constant-width" color="#000000" size=4>maxBins</font>

In decision trees, continuous features are converted into categorical features and maxBins determines how many bins should be created from continous features. More bins gives a higher level of granularity. The value must be greater than or equal to 2 and greater than or equal to the number of categories in any categorical feature in your dataset. The default is 32.

在决策树中，连续特征将转换为分类特征，maxBins 将确定应从连续特征创建的分箱数（分桶数）。更多的箱（桶）提供更高级别的粒度。该值必须大于或等于2且大于或等于数据集中任何分类特征中的类别数。默认值为32.

<font face="constant-width" color="#000000" size=4>impurity</font>

To build up a “tree” you need to configure when the model should branch. Impurity represents the metric (information gain) to determine whether or not the model should split at a particular leaf node. This parameter can be set to either be “entropy” or “gini” (default), two commonly used impurity metrics.

要构建“树”，您需要配置模型何时应该分支。杂质表示用于确定模型是否应在特定叶节点处拆分的衡量指标（信息增益）。此参数可以设置为“信息熵”或“基尼系数”（默认），两个常用的不纯度的衡量指标。

<font face="constant-width" color="#000000" size=4>minInfoGain</font>

This parameter determines the minimum information gain that can be used for a split. A higher value can prevent overfitting. This is largely something that needs to be determined from testing out different variations of the decision tree model. The default is zero.

此参数确定可用于拆分的最小信息增益。较高的值可以防止过度拟合。这很大程度上需要通过测试决策树模型的不同变体来确定。默认值为零。

<font face="constant-width" color="#000000" size=4>minInstancePerNode</font>

This parameter determines the minimum number of training instances that need to end in a particular node. Think of this as another manner of controlling max depth. We can prevent overfitting by limiting depth or we can prevent it by specifying that at minimum a certain number of training values need to end up in a particular leaf node. If it’s not met we would “prune” the tree until that requirement is met. A higher value can prevent overfitting. The default is 1, but this can be any value greater than 1.

此参数确定需要在特定节点中结束的最小训练实例数。可以将其视为控制最大深度的另一种方式。我们可以通过限制深度来防止过度拟合，或者我们可以通过指定至少一定数量的训练值需要在特定叶节点中结束防止过度拟合。如果不满足，我们将“修剪”树，直到满足该要求。较高的值可以防止过度拟合。默认值为1，但这可以是大于1的任何值。

### <font color="#00000">Training Parameters 训练参数</font>

These are configurations we specify in order to manipulate how we perform our training. Here is the training parameter for decision trees:

这些是我们指定的配置，以便控制我们如何执行训练。 以下是决策树的训练参数：

<font face="constant-width" color="#000000" size=4>checkpointInterval 检查点间隔时间</font>

Checkpointing is a way to save the model’s work over the course of training so that if nodes in the cluster crash for some reason, you don’t lose your work. A value of `10` means the model will get checkpointed every 10 iterations. Set this to `-1` to turn off checkpointing. This parameter needs to be set together with a `checkpointDir` (a directory to checkpoint to) and with `useNodeIdCache=true`. Consult the Spark documentation for more information on checkpointing.

检查点是一种在训练过程中保存模型工作的方法，这样如果群集中的节点由于某种原因而崩溃，您就不会丢失工作。 值 `10` 表示模型将每10次迭代检查一次。 将此值设置为 `-1` 可关闭检查点。 此参数需要与`checkpointDir`（检查点的目录）和 `useNodeIdCache=true` 一起设置。 有关检查点的更多信息，请参阅Spark文档。

### <font color="#00000">Prediction Parameters 预测参数</font>

There is only one prediction parameter for decision trees: thresholds. Refer to the explanation for thresholds under “Logistic Regression”.

决策树只有一个预测参数：阈值。 请参阅“Logistic 回归”下的阈值（thresholds ）说明。

Here’s a minimal but complete example of using a decision tree classifier:

这是使用决策树分类器的最小但完整的示例：

```scala
// in Scala
import org.apache.spark.ml.classification.DecisionTreeClassifier
val dt = new DecisionTreeClassifier()
println(dt.explainParams())
val dtModel = dt.fit(bInput)
```

```python
# in Python
from pyspark.ml.classification import DecisionTreeClassifier
dt = DecisionTreeClassifier()
print dt.explainParams()
dtModel = dt.fit(bInput) 
```

## <font color="#9a161a">Random Forest and Gradient-Boosted Trees 随机森林与梯度提升树</font>

These methods are extensions of the decision tree. Rather than training one tree on all of the data, you train multiple trees on varying subsets of the data. The intuition behind doing this is that various decision trees will become “experts” in that particular domain while others become experts in others. By combining these various experts, you then get a “wisdom of the crowds” effect, where the group’s performance exceeds any individual. In addition, these methods can help prevent overfitting.

这些方法是决策树的扩展。您可以在不同的数据子集上训练多个树，而不是在所有数据上训练一棵树。这样做的直觉是，各种决策树将成为该特定领域的“专家”，而其他决策树则成为其他领域的专家。通过将这些不同的专家结合起来，您可以获得“群众智慧”的效果，即群体的表现超过任何个体。此外，这些方法可以帮助防止过度拟合。

Random forests and gradient-boosted trees are two distinct methods for combining decision trees. In random forests, we simply train a lot of trees and then average their response to make a prediction. With gradient-boosted trees, each tree makes a weighted prediction (such that some trees have more predictive power for some classes than others). They have largely the same parameters, which we note below. One current limitation is that gradient-boosted trees currently only support binary labels.

随机森林和梯度提升树是组合决策树的两种不同方法。在随机森林中，我们只是训练了很多树，然后平均他们的反馈来做出预测。对于梯度提升树，每棵树都进行加权预测（这样一些树对某些类具有比其他树更强的预测能力）。它们的参数大致相同，我们在下面说明。目前的一个限制是梯度提升树目前仅支持二元标签。

---

<center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong></font></center>
There are several popular tools for learning tree-based models. For example, the [XGBoost](https://xgboost.readthedocs.io/en/latest/) library provides an integration package for Spark that can be used to run it on Spark.

有几种流行的工具可用于学习基于树的模型。例如，[XGBoost](https://xgboost.readthedocs.io/en/latest/) 库为Spark提供了一个集成包，可用于在Spark上运行它。

---

See [ISL 8.2](http://faculty.marshall.usc.edu/gareth-james/) and [ESL 10.1](https://web.stanford.edu/~hastie/ElemStatLearn/) for more information on these tree ensemble models.

有关这些树集合模型的更多信息，请参见 [ISL](http://faculty.marshall.usc.edu/gareth-james/)  8.2 和 [ESL](https://web.stanford.edu/~hastie/ElemStatLearn/) 10.1。

### <font color="#00000">Model Hyperparameters 模型超参数</font>

Random forests and gradient-boosted trees provide all of the same model hyperparameters supported by decision trees. In addition, they add several of their own.

随机森林和梯度提升树提供决策树支持的所有相同的模型超参数。此外，他们还添加了几个自己的。

#### <font color="#3399cc">Random forest only 只有随机森林</font>

<font face="constant-width" color="#000000" size=4>numTrees 树的数量</font>

The total number of trees to train. 

要训练的树木总数。

<font face="constant-width" color="#000000" size=4>featureSubsetStrategy 特征子集的策略</font> 

This parameter determines how many features should be considered for splits. This can be a variety of different values including “auto”, “all”, “sqrt”, “log2”, or a number “n.” When your input is “n” the model will use n * number of features during training. When n is in the range (1, number of features), the model will use n features during training. There’s no one-size-fits-all solution here, so it’s worth experimenting with different values in your pipeline.

此参数确定要为拆分考虑的功能数量。这可以是各种不同的值，包括“auto”，“all”，“sqrt”，“log2”或数字“n”。当您的输入为“n”时，模型将在训练期间使用：n乘以特征数量。当n在范围（1，特征数量）范围内时，模型将在训练期间使用n个特征。这里没有一个通用的解决方案，因此值得在您的管道中试验不同的值。

#### <font color="#3399cc">Gradient-boosted trees (GBT) only 仅梯度提升树（GBT）</font>

<font face="constant-width" color="#000000" size=4>lossType 损失函数类型</font>

This is the loss function for gradient-boosted trees to minimize during training. Currently, only logistic loss is supported.

这是梯度提升树在训练期间最小化的损失函数。目前，仅支持logistic损失。

<font face="constant-width" color="#000000" size=4>maxIter 最大迭代次数</font>

Total number of iterations over the data before stopping. Changing this probably won’t change your results a ton, so it shouldn’t be the first parameter you look to adjust. The default is 100.

停止前数据的迭代总数。改变这个可能不会明显改变你的结果，所以它不应该是你想要调整的第一个参数。默认值为100。

<font face="constant-width" color="#000000" size=4>stepSize 每步的大小（即学习率的大小）</font>

This is the learning rate for the algorithm. A larger step size means that larger jumps are made between training iterations. This can help in the optimization process and is something that should be tested in training. The default is 0.1 and this can be any value from 0 to 1.

这是算法的学习率。较大的步长意味着在训练迭代之间进行较大的跳跃。这有助于优化过程，并且应该在训练中进行测试。默认值为 0.1，可以是 0 到  1 之间的任何值。

### <font color="#00000">Training Parameters 训练参数</font>

There is only one training parameter for these models, `checkpointInterval`. Refer back to the explanation under “Decision Trees” for details on checkpointing.

这些模型只有一个训练参数，`checkpointInterval`。有关检查点的详细信息，请参阅“决策树”下的说明。

### <font color="#00000">Prediction Parameters 预测参数</font>

These models have the same prediction parameters as decision trees. Consult the prediction parameters under that model for more information.

这些模型具有与决策树相同的预测参数。有关更多信息，请参阅该模型下的预测参数。

Here’s a short code example of using each of these classifiers:

这是使用每个分类器的简短代码示例：

```scala 
// in Scala
import org.apache.spark.ml.classification.RandomForestClassifier
val rfClassifier = new RandomForestClassifier()
println(rfClassifier.explainParams())
val trainedModel = rfClassifier.fit(bInput)

// in Scala
import org.apache.spark.ml.classification.GBTClassifier
val gbtClassifier = new GBTClassifier()
println(gbtClassifier.explainParams())
val trainedModel = gbtClassifier.fit(bInput)
```

```python
# in Python
from pyspark.ml.classification import RandomForestClassifier
rfClassifier = RandomForestClassifier()
print rfClassifier.explainParams()
trainedModel = rfClassifier.fit(bInput)

# in Python
from pyspark.ml.classification import GBTClassifier
gbtClassifier = GBTClassifier()
print gbtClassifier.explainParams()
trainedModel = gbtClassifier.fit(bInput) 
```

## <font color="#9a161a">Naive Bayes 朴素贝叶斯</font>

Naive Bayes classifiers are a collection of classifiers based on Bayes’ theorem. The core assumption behind the models is that all features in your data are independent of one another. Naturally, strict independence is a bit naive, but even if this is violated, useful models can still be produced. Naive Bayes classifiers are commonly used in text or document classification, although it can be used as a more general-purpose classifier as well. There are two different model types: either a multivariate Bernoulli model, where indicator variables represent the existence of a term in a document; or the multinomial model, where the total counts of terms are used.

朴素贝叶斯分类器是基于贝叶斯定理的分类器集合。模型背后的核心假设是数据中的所有特征都是相互独立的。当然，严格的独立性有点天真，但即使违反了这一点，仍然可以制作出有用的模型。朴素贝叶斯分类器通常用于文本或文档分类，尽管它也可以用作更通用的分类器。有两种不同的模型类型：多变量伯努利模型，其中指标变量（indicator variable）表示文档中术语（terms）的存在；或多项式模型，其中使用术语（terms）的总计数。

One important note when it comes to Naive Bayes is that all input features must be non-negative. See [ISL 4.4](http://faculty.marshall.usc.edu/gareth-james/) and [ESL 6.6](https://web.stanford.edu/~hastie/ElemStatLearn/) for more background on these models.

朴素贝叶斯的一个重要注意事项是所有输入功能必须是非负的。有关这些内容的更多背景信息，请参阅 [ISL](http://faculty.marshall.usc.edu/gareth-james/)  4.4 和[ESL](https://web.stanford.edu/~hastie/ElemStatLearn/)  6.6 楷模。

### <font color="#00000">Model Hyperparameters 模型超参数</font>

These are configurations we specify to determine the basic structure of the models:
这些是我们指定的配置，用于决定模型的基本结构：


<font face="constant-width" color="#000000" size=4>modelType 模型类型</font>

Either “bernoulli” or “multinomial.” See the previous section for more information on this choice.

“伯努利”或“多项式”。有关此选择的更多信息，请参阅上一节。

<font face="constant-width" color="#000000" size=4>weightCol 权重列</font>

Allows weighing different data points differently. Refer back to “Training Parameters” for the explanation of this hyperparameter.

允许对不同的数据点进行不一样的加权。有关此超参数的说明，请参阅“训练参数”。

### <font color="#00000">Training Parameters 训练参数</font>

These are configurations that specify how we perform our training:
这些是指定我们如何执行训练的配置：

<font face="constant-width" color="#000000" size=4>smoothing 平滑</font>

This determines the amount of regularization that should take place using [additive smoothing](https://en.wikipedia.org/wiki/Additive_smoothing). This helps smooth out categorical data and avoid overfitting on the training data by changing the expected probability for certain classes. The default value is 1.

这决定了应使用[加法平滑](https://en.wikipedia.org/wiki/Additive_smoothing)进行正则化的数量。这有助于平滑分类数据，并通过改变某些类的预期概率来避免过度拟合训练数据。默认值为1。

### <font color="#00000">Prediction Parameters 预测参数</font>

Naive Bayes shares the same prediction parameter, thresholds, as all of our other models. Refer back to the previous explanation for threshold to see how to use this. Here’s an example of using a Naive Bayes classifier.

朴素贝叶斯与我们所有其他模型共享相同的预测参数，阈值。请参阅前面的阈值（threshold ）说明，了解如何使用它。这是使用朴素贝叶斯分类器的示例。

```scala
// in Scala
import org.apache.spark.ml.classification.NaiveBayes
val nb = new NaiveBayes()
println(nb.explainParams())
val trainedModel = nb.fit(bInput.where("label != 0"))
```

```python
# in Python
from pyspark.ml.classification import NaiveBayes
nb = NaiveBayes()
print nb.explainParams()
trainedModel = nb.fit(bInput.where("label != 0"))
```

---

<center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong></font></center>
Note that in this example dataset, we have features that have negative values. In this case, the rows with negative features correspond to rows with label “0”. Therefore we’re just going to filter them out (via the label) instead of processing them further to demonstrate the naive bayes API. 

请注意，在此示例数据集中，我们具有负值的特征。 在这种情况下，具有负特征的行对应于标记为“0”的行。 因此，我们只是将它们过滤掉（通过标签），而不是进一步处理它们以演示朴素的贝叶斯API。

---

## <font color="#9a161a">Evaluators for Classification and Automating Model Tuning 分类和自动化模型调整评估器</font>

As we saw in Chapter 24, evaluators allow us to specify the metric of success for our model. An evaluator doesn’t help too much when it stands alone; however, when we use it in a pipeline, we can automate a grid search of our various parameters of the models and transformers—trying all combinations of the parameters to see which ones perform the best. Evaluators are most useful in this pipeline and parameter grid context. For classification, there are two evaluators, and they expect two columns: a predicted label from the model and a true label. For binary classification we use the `BinaryClassificationEvaluator`. This supports optimizing for two different metrics “areaUnderROC” and areaUnderPR.” For multiclass classification, we need to use the MulticlassClassificationEvaluator, which supports optimizing for “f1”, “weightedPrecision”, “weightedRecall”, and “accuracy”.

正如我们在第24章中看到的那样，评估器（evaluator）允许我们为模型指定成功的衡量。评估器（evaluator）独立时并没有太多帮助；然而，当我们在管道中使用它时，我们可以自动对模型和转换器的各种参数进行网格搜索——尝试参数的所有组合以查看哪些参数表现最佳。评估器（evaluator）在此管道和参数网格上下文中最有用。对于分类，有两个评估器（evaluator），他们期望有两列：来自模型的预测标签和真实标签。对于二元分类，我们使用 `BinaryClassificationEvaluator`。这支持优化两个不同的衡量指标 “areaUnderROC” 和 “areaUnderPR“。对于多类分类，我们需要使用`MulticlassClassificationEvaluator`，它支持优化 ”f1“，”weightedPrecision“，”weightedRecall“ 和 ”accuracy“。

To use evaluators, we build up our pipeline, specify the parameters we would like to test, and then run it and see the results. See Chapter 24 for a code example.

要使用评估器（evaluator），我们构建我们的管道，指定我们想要测试的参数，然后运行它并查看结果。有关代码示例，请参见第24章。

## <font color="#9a161a">Detailed Evaluation Metrics 详细的评估指标</font>

MLlib also contains tools that let you evaluate multiple classification metrics at once. Unfortunately, these metrics classes have not been ported over to Spark’s DataFrame-based ML package from the underlying RDD framework. So, at the time of this writing, you still have to create an RDD to use these. In the future, this functionality will likely be ported to DataFrames and the following may no longer be the best way to see metrics (although you will still be able to use these APIs).

MLlib 还包含一些工具，可让您一次评估多个分类指标。遗憾的是，这些衡量标准类尚未从基础RDD框架移植到Spark的基于DataFrame的ML包。因此，在撰写本文时，您仍然需要创建一个RDD来使用它们。将来，此功能可能会移植到DataFrames，以下可能不再是查看指标的最佳方式（尽管您仍然可以使用这些API）。


There are three different classification metrics we can use:

我们可以使用三种不同的分类指标：

- Binary classification metrics
二进制分类指标
- Multiclass classification metrics
多类分类指标
- Multilabel classification metrics
多标签分类指标

All of these measures follow the same approximate style. We’ll compare generated outputs with true values and the model calculates all of the relevant metrics for us. Then we can query the object for the values for each of the metrics:

所有这些措施都遵循相同的近似风格。我们将生成的输出与真值进行比较，模型为我们计算所有相关指标。然后我们可以在对象中查询每个指标的值：

```scala
// in Scala
import org.apache.spark.mllib.evaluation.BinaryClassificationMetrics
val out = model.transform(bInput)
.select("prediction", "label")
.rdd.map(x => (x(0).asInstanceOf[Double], x(1).asInstanceOf[Double]))
val metrics = new BinaryClassificationMetrics(out)
```

```python
# in Python
from pyspark.mllib.evaluation import BinaryClassificationMetrics
out = model.transform(bInput)\
.select("prediction", "label")\
.rdd.map(lambda x: (float(x[0]), float(x[1])))
metrics = BinaryClassificationMetrics(out)
```

Once we’ve done that, we can see typical classification success metrics on this metric’s object using a similar API to the one we saw with logistic regression :

完成后，我们可以使用与逻辑回归看到的类似的API，在此衡量标准对象上看到典型的分类成功衡量标准：

```scala
// in Scala
metrics.areaUnderPR
metrics.areaUnderROC
println("Receiver Operating Characteristic")
metrics.roc.toDF().show()
```

```python
# in Python
print metrics.areaUnderPR
print metrics.areaUnderROC
print "Receiver Operating Characteristic"
metrics.roc.toDF().show()
```

## <font color="#9a161a">One-vs-Rest Classifier 1对其余的分类</font>

There are some MLlib models that don’t support multiclass classification. In these cases, users can leverage a one-vs-rest classifier in order to perform multiclass classification given only a binary classifier. The intuition behind this is that for every class you hope to predict, the one-vs-rest classifier will turn the problem into a binary classification problem by isolating one class as the target class and grouping all of the other classes into one. Thus the prediction of the class becomes binary (is it this class or not this class?).

有些MLlib模型不支持多类分类。在这些情况下，用户可以利用 one-vs-rest 分类器，以便仅在给定二元分类器的情况下执行多类分类。这背后的直觉是，对于您希望预测的每个类，one-vs-rest 分类器将把一个类隔离为目标类并将所有其他类分组为一个，从而将问题转化为二元分类问题。因此，类的预测变为二元（这个类是否是这个类？）。

One-vs-rest is implemented as an estimator. For the base classifier it takes instances of the classifier and creates a binary classification problem for each of the classes. The classifier for class i is trained to predict whether the label is i or not, distinguishing class i from all other classes. Predictions are done by evaluating each binary classifier and the index of the most confident classifier is output as the label.

One-vs-rest 实现为估计器（estimator）。对于基类分类器，它接受分类器的实例并为每个类创建二元分类问题。训练 i 类的分类器来预测标签是否为 i ，将类 i 与所有其他类区分开来。通过评估每个二元分类器来完成预测，并且输出最自信的分类器的下标作为标签。

See the Spark documentation for a nice example of the use of [one-vs-rest](http://spark.apache.org/docs/latest/ml-classification-regression.html#one-vs-rest-classifier-aka-one-vs-all).

请参阅Spark文档，了解使用 [one-vs-rest](http://spark.apache.org/docs/latest/ml-classification-regression.html#one-vs-rest-classifier-aka-one-vs-all) 的一个很好的例子）。

## <font color="#9a161a">Multilayer Perceptron 多层感知器</font>

The multilayer perceptron is a classifier based on neural networks with a configurable number of layers (and layer sizes). We will discuss it in Chapter 31.

多层感知机是基于具有可配置数量的层（和层大小）的神经网络的分类器。我们将在第31章讨论它。

## <font color="#9a161a">Conclusion 结论</font>

In this chapter we covered the majority of tools Spark provides for classification: predicting one of a finite set of labels for each data point based on its features. In the next chapter, we’ll look at regression, where the required output is continuous instead of categorical. 

在本章中，我们介绍了Spark为分类提供的大多数工具：根据每个数据点的特征为每个数据点预测一组有限标签。在下一章中，我们将看回归，其中所需的输出是连续的而不是分类的。
