---
title: 翻译 Chapter 24 Advanced Analytics and Machine Learning Overview
date: 2019-08-20
copyright: true
categories: English,中文
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 24 Advanced Analytics and Machine Learning Overview

Thus far, we have covered fairly general data flow APIs. This part of the book will dive deeper into some of the more specific advanced analytics APIs available in Spark. Beyond large-scale SQL analysis and streaming, Spark also provides support for statistics, machine learning, and graph analytics. These encompass a set of workloads that we will refer to as advanced analytics. This part of the book will cover advanced analytics tools in Spark, including : 

到目前为止，我们已经涵盖了相当通用的数据流 API。本书的这一部分将深入探讨 Spark 中可用的一些更具体的高级分析 API。除了大规模的 SQL  分析和流媒体，Spark 还提供对统计，机器学习和图形分析的支持。这些包含一套工作负荷，我们将其称为高级分析。本书的这一部分将介绍Spark中的高级分析工具，包括：

- Preprocessing your data (cleaning data and feature engineering) 预处理数据（清理数据和特征工程）
- Supervised learning 监督学习
- Recommendation learning 推荐学习
- Unsupervised engines 无人监督引擎
- Graph analytics 图形分析
- Deep learning 深度学习

This chapter offers a basic overview of advanced analytics, some example use cases, and a basic advanced analytics workflow. Then we’ll cover the analytics tools just listed and teach you how to apply them.

本章提供高级分析的基本概述，一些用户案例的示例和基本的高级分析工作流。然后我们将介绍刚刚列出的分析工具，并教您如何应用它们。

---

<center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong><font></center>
This book is not intended to teach you everything you need to know about machine learning from scratch. We won’t go into strict mathematical definitions and formulations—not for lack of importance but simply because it’s too much information to include. This part of the book is not an algorithm guide that will teach you the mathematical underpinnings  of every available algorithm nor the in-depth implementation strategies used. The chapters included here serve as a guide for users, with the purpose of outlining what you need to know to use Spark’s advanced analytics APIs. 

本书无意向您介绍从头开始学习机器学习所需的所有知识。我们不会进入严格的数学定义和表述——不是因为缺乏重要性，而仅仅因为它包含太多的信息。本书的这一部分不是一个算法指南，它将教你每个可用算法的数学基础，也不会使用深入的实现策略。此处包含的章节可作为用户指南，旨在概述使用Spark的高级分析API需要了解的内容。

---



## <font color="#9a161a" >A Short Primer on Advanced Analytics 高级分析的简短入门</font>

Advanced analytics refers to a variety of techniques aimed at solving the core problem of deriving insights and making predictions or recommendations based on data. The best ontology for machine learning is structured based on the task that you’d like to perform. The most common tasks include:

高级分析是指旨在解决获取洞察力并根据数据进行预测或推荐的核心问题的各种技术。机器学习的最佳本体是基于您想要执行的任务而构建的。最常见的任务包括：

- Supervised learning, including classification and regression, where the goal is to predict a label for each data point based on various features.

  监督学习，包括分类和回归，其目标是基于各种特征预测每个数据点的标签。

- Recommendation engines to suggest products to users based on behavior.

  推荐引擎，根据行为向用户推荐产品。

- Unsupervised learning, including clustering, anomaly detection, and topic modeling, where the goal is to discover structure in the data.

  无监督学习，包括聚类，异常检测和主题建模，其目标是发现数据中的结构。

- Graph analytics tasks such as searching for patterns in a social network.

  图形分析任务，例如在社交网络中搜索模式。

Before discussing Spark’s APIs in detail, let’s review each of these tasks along with some common machine learning and advanced analytics use cases. While we have certainly tried to make this introduction as accessible as possible, at times you may need to consult other resources in order to fully understand the material. O’Reilly should we link to or mention any specific ones? Additionally, we will cite the following books throughout the next few chapters because they are great resources for learning more about the individual analytics (and, as a bonus, they are freely available on the web):

在详细讨论Spark的API之前，让我们回顾一下这些任务以及一些常见的机器学习和高级分析用户案例。虽然我们确实试图尽可能地使用这种介绍，但有时您可能需要咨询其他资源以便完全理解材料。 O'Reilly应该链接或提及任何特定的？此外，我们将在接下来的几章中引用以下书籍，因为它们是了解有关个人分析的更多资源（并且，作为奖励，它们可以在网上免费获得）：

- [An Introduction to Statistical Learning](http://www-bcf.usc.edu/~gareth/ISL/) by Gareth James, Daniela Witten, Trevor Hastie, and Robert Tibshirani. We refer to this book as “ISL.”

  Gareth James，Daniela Witten，Trevor Hastie 和 Robert Tibshirani 的[统计学习导论](http://www-bcf.usc.edu/~gareth/ISL/)简介。我们将这本书称为“ISL”。

- [Elements of Statistical Learning](https://web.stanford.edu/~hastie/ElemStatLearn/) by Trevor Hastie, Robert Tibshirani, and Jerome Friedman. We refer to this book as “ESL.”

  Trevor Hastie，Robert Tibshirani 和 Jerome Friedman 的[统计学习基础](https://web.stanford.edu/~hastie/ElemStatLearn/)。我们将这本书称为“ESL”。

- [Deep Learning](http://www.deeplearningbook.org/) by Ian Goodfellow, Yoshua Bengio, and Aaron Courville. We refer to this book as “DLB.”

  Ian Goodfellow，Yoshua Bengio 和 Aaron Courville 的[深度学习](http://www.deeplearningbook.org/)。我们将这本书称为“DLB”。

### <font color="#00000">Supervised Learning 监督学习</font>

Supervised learning is probably the most common type of machine learning. The goal is simple: using historical data that already has labels (often called the dependent variables), train a model to predict the values of those labels based on various features of the data points. One example would be to predict a person’s income (the dependent variables) based on age (a feature). This training process usually proceeds through an iterative optimization algorithm such as gradient descent. The training algorithm starts with a basic model and gradually improves it by adjusting various internal parameters (coefficients) during each training iteration. The result of this process is a trained model that you can use to make predictions on new data. There are a number of different tasks we’ll need to complete as part of the process of training and making predictions, such as measuring the success of trained models before using them in the field, but the fundamental principle is simple: train on historical data, ensure that it generalizes to data we didn’t train on, and then make predictions on new data.

有监督的学习可能是最常见的机器学习类型。目标很简单：使用已经有标签的历史数据（通常称为因变量），训练模型根据数据点的各种特征预测这些标签的值。一个例子是根据年龄（特征）预测一个人的收入（因变量）。该训练过程通常通过诸如梯度下降的迭代优化算法进行。训练算法从基本模型开始，并通过在每次训练迭代期间调整各种内部参数（系数）来逐步改进它。此过程的结果是经过训练的模型，您可以使用该模型对新数据进行预测。作为训练和预测过程的一部分，我们需要完成许多不同的任务，例如在现场使用之前测量训练模型的成功，但基本原则很简单：训练历史数据，确保它推广到我们没有训练的数据，然后对新数据进行预测。

We can further organize supervised learning based on the type of variable we’re looking to predict. We’ll get to that next. 

我们可以根据我们想要预测的变量类型进一步组织有监督的学习。我们接下来会谈到这一点。

#### <font color="#3399cc" >Classification 分类</font>

One common type of supervised learning is classification. Classification is the act of training an algorithm to predict a dependent variable that is categorical (belonging to a discrete, finite set of values). The most common case is binary classification, where our resulting model will make a prediction that a given item belongs to one of two groups. The canonical example is classifying email spam. Using a set of historical emails that are organized into groups of spam emails and not spam emails, we train an algorithm to analyze the words in, and any number of properties of, the historical emails and make predictions about them. Once we are satisfied with the algorithm’s performance, we use that model to make predictions about future emails the model has never seen before.

一种常见的监督学习类型是分类。分类是训练算法以预测分类的因变量（属于离散的，有限的值集）的行为。最常见的情况是二元分类，其中我们得到的模型将预测给定项目属于两个组中的一个。典型的例子是对垃圾邮件进行分类。使用一组历史电子邮件，这些电子邮件被组织成垃圾邮件组而不是垃圾邮件，我们训练一种算法来分析历史电子邮件中的单词和任意数量的属性，并对它们进行预测。一旦我们对算法的性能感到满意，我们就会使用该模型来预测模型以前从未见过的未来电子邮件。

When we classify items into more than just two categories, we call this multiclass classification. For example, we may have four different categories of email (as opposed to the two categories in the previous paragraph): spam, personal, work related, and other. There are many use cases for classification, including:

当我们将项目分类为两个以上的类别时，我们称之为多类分类。例如，我们可能有四种不同类别的电子邮件（与前一段中的两个类别相对）：垃圾邮件，个人电子邮件，与工作相关的邮件和其他类别。分类有很多用户案例，包括：

**Predicting disease 预测疾病**

A doctor or hospital might have a historical dataset of behavioral and physiological attributes of a set of patients. They could use this dataset to train a model on this historical data (and evaluate its success and ethical implications before applying it) and then leverage  it to predict whether or not a patient has heart disease or not. This is an example of binary classification (healthy heart, unhealthy heart) or multiclass classification (healthy heart, or one of several different diseases).
医生或医院可能具有一组患者的行为和生理属性的历史数据集。他们可以使用此数据集来训练该历史数据的模型（并在应用之前评估其成功和伦理影响），然后利用它来预测患者是否患有心脏病。这是二元分类（健康的心脏，不健康的心脏）或多类分类（健康的心脏，或几种不同的疾病之一）的例子。

**Classifying images 分类图像**

There are a number of applications from companies like Apple, Google, or Facebook that can predict who is in a given photo by running a classification model that has been trained on historical images of people in your past photos. Another common use case is to classify images or label the objects in images.

Apple，Google或Facebook等公司提供的许多应用程序可以通过运行已经过过去照片中人物历史图像训练的分类模型来预测给定照片中的人物。另一个常见用户案例是对图像进行分类或标记图像中的对象。

**Predicting customer churn 预测客户流失**

A more business-oriented use case might be predicting customer churn—that is, which customers are likely to stop using a service. You can do this by training a binary classifier on past customers that have churned (and not churned) and using it to try and predict whether or not current customers will churn.

更加面向业务的用户案例可能预测客户流失——也就是说，哪些客户可能会停止使用服务。您可以通过对过去已经搅动（而非搅动）的客户训练二元分类器并使用它来尝试预测当前客户是否会流失来实现此目的。

**Buy or won’t buy 买或不买**

Companies often want to predict whether visitors of their website will purchase a given product. They might use information about users’ browsing pattern or attributes such as location in order to drive this prediction.

公司通常希望预测其网站的访问者是否会购买特定产品。他们可能会使用有关用户浏览模式或属性（如位置）的信息来推动此预测。


There are many more use cases for classification beyond these examples. We will introduce more use cases, as well as Spark’s classification APIs, in Chapter 26.

除了这些示例之外，还有更多用于分类的用户案例。我们将在第26章介绍更多用户案例以及 Spark 的分类API。

#### <font color="#3399cc" >Regression 回归</font>

In classification, our dependent variable is a set of discrete values. In regression, we instead try to predict a continuous variable (a real number). In simplest terms, rather than predicting a category, we want to predict a value on a number line. The rest of the process is largely the same, which is why they’re both forms of supervised learning. We will train on historical data to make predictions about data we have never seen. Here are some typical examples:

在分类中，我们的因变量是一组离散值。在回归中，我们改为尝试预测连续变量（实数）。简单来说，我们希望预测数字行上的值，而不是预测类别。其余的过程基本相同，这就是为什么它们都是监督学习的形式。我们将对历史数据进行训练，以预测我们从未见过的数据。以下是一些典型示例：

**Predicting sales 预测销售额**

A store may want to predict total product sales on given data using historical sales data. There are a number of potential input variables, but a simple example might be using last week’s sales data to predict the next day’s data.

商店可能希望使用历史销售数据预测给定数据的总产品销售额。有许多潜在的输入变量，但一个简单的例子可能是使用上周的销售数据来预测第二天的数据。

**Predicting height 预测身高**

Based on the heights of two individuals, we might want to predict the heights of their potential children.

基于两个人的高度，我们可能想要预测他们可能孩子的高度。

**Predicting the number of viewers of a show 预测节目观众的数量**

A media company like Netflix might try to predict how many of their subscribers will watch a particular show.

像Netflix这样的媒体公司可能会试图预测他们有多少用户会观看特定的节目。

We will introduce more use cases, as well as Spark’s methods for regression, in Chapter 27.

我们将在第27章介绍更多用户案例以及Spark的回归方法。

#### <font color="#3399cc" >Recommendation 推荐</font>

Recommendation is one of the most intuitive applications of advanced analytics. By studying people’s explicit preferences (through ratings) or implicit ones (through observed behavior) for various products or items, an algorithm can make recommendations on what a user may like by drawing similarities between the users or items. By looking at these similarities, the algorithm makes recommendations to users based on what similar users liked, or what other products resemble the ones the user already purchased. Recommendation is a common use case for Spark and well suited to big data. Here are some example use cases:

推荐是高级分析最直观的应用之一。通过研究人们对各种产品或项目的明确偏好（通过评级）或隐性（通过观察到的行为），算法可以通过绘制用户或项目之间的相似性来对用户可能喜欢的内容做出推荐。通过查看这些相似性，该算法基于类似用户喜欢的内容或其他产品类似于用户已购买的产品向用户推荐。推荐是Spark的常见用户案例，非常适合大数据。以下是一些示例用户案例：

**Movie recommendations 电影推荐**

[Netflix uses Spark](https://www.youtube.com/watch?v=II8GlmbDg9M), although not necessarily its built-in libraries, to make large-scale movie recommendations to its users. It does this by studying what movies users watch and do not watch in the Netflix application. In addition, Netflix likely takes into consideration how similar a given user’s ratings are to other users’.

[Netflix使用Spark](https://www.youtube.com/watch?v=II8GlmbDg9M)，尽管不一定是其内置库，向用户提供大规模的电影推荐。它是通过研究用户在Netflix应用程序中观看和不观看的电影来实现的。此外，Netflix可能会考虑给定的用户评级与其他用户的相似程度。

**Product recommendations 产品推荐**

Amazon uses product recommendations as one of its main tools to increase sales. For instance, based on the items in our shopping cart, Amazon may recommend other items that were added to similar shopping carts in the past. Likewise, on every product page, Amazon shows similar products purchased by other users.

亚马逊将产品推荐作为增加销售额的主要工具之一。例如，根据购物车中的商品，亚马逊可能会推荐过去添加到类似购物车中的其他商品。同样，在每个产品页面上，亚马逊都会显示其他用户购买的类似产品。


We will introduce more recommendation use cases, as well as Spark’s methods for generating recommendations, in Chapter 28. 

我们将在第28章介绍更多推荐用户案例，以及 Spark 用于生成推荐的方法。

### <font color="#00000">Unsupervised Learning 无监督学习</font>

Unsupervised learning is the act of trying to find patterns or discover the underlying structure in a given set of data. This differs from supervised learning because there is no dependent variable (label) to predict.

无监督学习是试图在给定数据集中找到模式或发现底层结构的行为。这与监督学习不同，因为没有因变量（标签）来预测。

Some example use cases for unsupervised learning include:

用于无监督学习的一些示例用户案例包括：

**Anomaly detection 异常检测**

Given some standard event type often occuring over time, we might want to report when a nonstandard type of event occurs. For example, a security officer might want to receive notifications when a strange object (think vehicle, skater, or bicyclist) is observed on a pathway.

鉴于某些标准事件类型经常出现，我们可能希望报告何时发生非标准类型的事件。例如，当在道路上观察到奇怪的物体（想想车辆，滑板或骑自行车者）时，安保人员可能希望接收通知。

**User segmentation 用户细分**

Given a set of user behaviors, we might want to better understand what attributes certain users share with other users. For instance, a gaming company might cluster users based on properties like the number of hours played in a given game. The algorithm might reveal that casual players have very different behavior than hardcore gamers, for example, and allow the company to offer different recommendations or rewards to each player.

给定一组用户行为，我们可能希望更好地了解某些用户与其他用户共享的属性。例如，游戏公司可以基于诸如在给定游戏中玩的小时数之类的属性来聚集用户。例如，该算法可能揭示休闲玩家与硬核游戏玩家的行为非常不同，并允许公司向每个玩家提供不同的推荐或奖励。

**Topic modeling 主题建模**

Given a set of documents, we might analyze the different words contained therein to see if there is some underlying relation between them. For example, given a number of web pages on data analytics, a topic modeling algorithm can cluster them into pages about machine learning, SQL, streaming, and so on based on groups of words that are more common in one topic than in others. Intuitively,  it is easy to see how segmenting customers could help a platform cater better to each set of users. However, it may be hard to discover whether or not this set of user segments is “correct”. For this reason, it can be difficult to determine whether a particular model is good or not. We will discuss unsupervised learning in detail in Chapter 29.

给定一组文档，我们可以分析其中包含的不同单词，看看它们之间是否存在某种潜在的关系。例如，给定大量关于数据分析的网页，主题建模算法可以基于在一个主题中比在其他主题中更常见的单词组将它们聚类到关于机器学习，SQL，流等的页面中。直观地，很容易看出细分客户如何帮助平台更好地满足每组用户。但是，可能很难发现这组用户段是否“正确”。因此，可能难以确定特定模型是否良好。我们将在第29章详细讨论无监督学习。

### <font color="#00000">Graph Analytics 图表分析</font>

While less common than classification and regression, graph analytics is a powerful tool. Fundamentally, graph analytics is the study of structures in which we specify vertices (which are objects) and edges (which represent the relationships between those objects). For example, the vertices might represent people and products, and edges might represent a purchase. By looking at the properties of vertices and edges, we can better understand the connections between them and the overall structure of the graph. Since graphs are all about relationships, anything that specifies a relationship is a great use case for graph analytics. Some examples include:

虽然不如分类和回归常见，但图形分析是一种强大的工具。从根本上说，图形分析是对结构的研究，其中我们指定顶点（对象）和边（表示这些对象之间的关系）。例如，顶点可能代表人和产品，边可能代表购买。通过查看顶点和边的属性，我们可以更好地理解它们之间的连接以及图的整体结构。由于图表都是关系，因此指定关系的任何内容都是图表分析的一个很好的用户案例。一些例子包括：

**Fraud prediction 欺诈预测**

[Capital One uses Spark’s graph analytics capabilities](https://www.youtube.com/watch?v=q5HFMVoN_rc&feature=youtu.be) to better understand fraud networks. By using historical fraudulent information (like phone numbers, addresses, or names) they discover fraudulent credit requests or transactions. For instance, any user accounts within two hops of a fraudulent phone number might be considered suspicious.

[Capital One使用Spark的图形分析功能](https://www.youtube.com/watch?v=q5HFMVoN_rc&feature=youtu.be)来更好地了解欺诈网络。通过使用历史欺诈信息（如电话号码，地址或姓名），他们会发现欺诈性信用请求或交易。例如，欺诈性电话号码两跳内的任何用户帐户可能被视为可疑。

**Anomaly detection 异常检测**

By looking at how networks of individuals connect with one another, outliers and anomalies can be flagged for manual analysis. For instance, if typically in our data each vertex has ten edges associated with it and a given vertex only has one edge, that might be worth investigating as something strange.

通过观察个体网络如何相互连接，可以标记离群点和异常值以进行手动分析。例如，如果通常在我们的数据中，每个顶点都有10个与之关联的边，并且给定的顶点只有一个边，那么可能值得调查一些奇怪的东西。

**Classification 分类**

Given some facts about certain vertices in a network, you can classify other vertices according to their connection to the original node. For instance, if a certain individual is labeled as an influencer in a social network, we could classify other individuals with similar network structures as influencers.

给定关于网络中某些顶点的一些事实，您可以根据它们与原始节点的连接对其他顶点进行分类。例如，如果某个人被标记为社交网络中的影响者，我们可以将具有类似网络结构的其他个人归类为影响者。

**Recommendation 推荐**

Google’s original web recommendation algorithm, PageRank, is a graph algorithm that analyzes website relationships in order to rank the importance of web pages. For example, a web page that has a lot of links to it is ranked as more important than one with no links to it. We’ll discuss more examples of graph analytics in Chapter 30. 

Google 的原始网络推荐算法 PageRank 是一种图表算法，可以分析网站关系，以便对网页的重要性进行排名。例如，具有大量链接指向的网页被排名为比没有链接指向的网页更重要。我们将在第30章讨论更多图形分析的例子。

### <font color="#00000">The Advanced Analytics Process 高级分析流程</font>

You should have a firm grasp of some fundamental use cases for machine learning and advanced analytics. However, finding a use case is only a small part of the actual advanced analytics process. There is a lot of work in preparing your data for analysis, testing different ways of modeling it, and evaluating these models. This section will provide structure to the overall analytics process and the steps we have to take to not just perform one of the tasks just outlined, but actually evaluate success objectively in order to understand whether or not we should apply our model to the real world (Figure 24-1).

您应该牢牢掌握机器学习和高级分析的一些基本用户案例。但是，查找用户案例只是实际高级分析过程的一小部分。在准备分析数据，测试不同的建模方法以及评估这些模型方面，有很多工作要做。本节将提供整体分析过程的结构，以及我们必须采取的步骤，不仅要执行刚刚概述的任务之一，而且要客观地评估成功，以便了解我们是否应该将我们的模型应用于现实世界（图24-1）。

![1567256517488](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter24/1567256517488.png)

The overall process involves, the following steps (with some variation):

整个过程涉及以下步骤（有一些变化）：

1. Gathering and collecting the relevant data for your task.

  归拢和收集任务的相关数据。

2. Cleaning and inspecting the data to better understand it.

  清理和检查数据以更好地理解它。

3. Performing feature engineering to allow the algorithm to leverage the data in a suitable form
    (e.g., converting the data to numerical vectors).

  执行特征工程以允许算法以适当的形式利用数据（例如，将数据转换为数字向量）。

4. Using a portion of this data as a training set to train one or more algorithms to generate some candidate models.

  使用该数据的一部分作为训练集来训练一个或多个算法以生成一些候选模型。

5. Evaluating and comparing models against your success criteria by objectively measuring results on a subset of the same data that was not used for training. This allows you to better understand how your model may perform in the wild.

  通过客观地测量未用于训练的相同数据的子集的结果，评估和比较模型与您的成功标准。这使您可以更好地了解模型在新数据上的表现。

6. Leveraging the insights from the above process and/or using the model to make predictions, detect anomalies, or solve more general business challenges. These steps won’t be the same for every advanced analytics task. However, this workflow does serve as a general framework for what you’re going to need to be successful with advanced analytics. Just as we did with the various advanced analytics tasks earlier in the chapter, let’s break down the process to better understand the overall objective of each step.

  利用上述过程中的见解和/或使用模型进行预测，检测异常或解决更常见的业务挑战。每个高级分析任务的这些步骤都不相同。但是，此工作流程确实可作为使用高级分析成功所需的一般框架。正如我们在本章前面对各种高级分析任务所做的那样，让我们分解过程以更好地理解每个步骤的总体目标。

#### <font color="#3399cc" >Data collection 数据收集</font>

Naturally it’s hard to create a training set without first collecting data. Typically this means at least gathering the datasets you’ll want to leverage to train your algorithm. Spark is an excellent tool for this because of its ability to speak to a variety of data sources and work with data big and small.

当然，在没有首先收集数据的情况下很难创建训练集。通常，这意味着至少要收集您想要用来训练算法的数据集。 Spark 是一个很好的工具，因为它能够与各种数据源对话并处理大小数据。

#### <font color="#3399cc" >Data cleaning 数据清理</font>

After you’ve gathered the proper data, you’re going to need to clean and inspect it. This is typically done as part of a process called exploratory data analysis, or EDA. EDA generally means using interactive queries and visualization methods in order to better understand distributions, correlations, and other details in your data. During this process you may notice you need to remove some values that may have been misrecorded upstream  or that other values may be missing. Whatever the case, it’s always good to know what is in your data to avoid mistakes down the road. The multitude of Spark functions in the structured APIs will provide a simple way to clean and report on your data.

在收集了适当的数据后，您将需要清理并检查它。这通常作为称为探索性数据分析（EDA）的过程的一部分来完成。 EDA通常意味着使用交互式查询和可视化方法，以便更好地理解数据中的分布，相关性和其他详细信息。在此过程中，您可能会注意到需要删除可能在上游错误记录的某些值或者可能缺少其他值。无论如何，最好知道数据中的内容，以避免错误发生。结构化API中的众多 Spark 函数将提供一种清理和报告数据的简单方法。

#### <font color="#3399cc" >Feature engineering 特征工程</font>

Now that you collected and cleaned your dataset, it’s time to convert it to a form suitable for machine learning algorithms, which generally means numerical features. Proper feature engineering can often make or break a machine learning application, so this is one task you’ll want to do carefully. The process of feature engineering includes a variety of tasks, such as normalizing data, adding variables to represent the interactions of other variables, manipulating categorical variables, and converting them to the proper format to be input into our machine learning model. In MLlib, Spark’s machine learning library, all variables will usually have to be input as vectors of doubles (regardless of what they actually represent). We cover the process of feature engineering in great depth in Chapter 25. As you will see in that chapter, Spark provides the essentials you’ll need to manipulate your data using a variety of machine learning statistical techniques.

现在您已经收集并清理了数据集，现在是时候将其转换为适合机器学习算法的形式，这通常意味着数字特征。合适的特征工程通常可以创建或破坏机器学习应用程序，因此这是您需要仔细完成的一项任务。特征工程的过程包括各种任务，例如规范化数据，添加变量以表示其他变量的交互，控制分类变量，以及将它们转换为适当的格式以输入到我们的机器学习模型中。在 Spark 的机器学习库 MLlib 中，所有变量通常都必须作为双精度向量输入（无论它们实际代表什么）。我们将在第25章深入介绍特征工程的过程。正如您将在本章中看到的，Spark 提供了使用各种机器学习统计技术操作数据所需的基本知识。

---

<center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong></font></center>
The following few steps (training models, model tuning, and evaluation) are not relevant to all use cases. This is a general workflow that may vary significantly based on the end objective you would like to achieve.

以下几个步骤（训练模型，模型调优和评估）与所有用户案例无关。 这是一个通用工作流程，可能会根据您希望实现的最终目标而有很大差异。

---

#### <font color="#3399cc" >Training models 训练模型</font>

At this point in the process we have a dataset of historical information (e.g., spam or not spam emails) and a task we would like to complete (e.g., classifying spam emails). Next, we will want to train a model to predict the correct output, given some input. During the training process, the parameters inside of the model will change according to how well the model performed on the input data. For instance, to classify spam emails, our algorithm will likely find that certain words are better predictors of spam than others and therefore weight the parameters associated with those words higher. In the end, the trained model will find that certain words should have more influence (because of their consistent association with spam emails) than others. The output of the training process is what we call a model. Models can then be used to gain insights or to make future predictions. To make predictions, you will give the model an input and it will produce an output based on a mathematical manipulation of these inputs. Using the classification example, given the properties of an email, it will predict whether that email is spam or not by comparing to the historical spam and not spam emails that it was trained on.

在此过程中，我们拥有历史信息（例如，垃圾邮件或非垃圾邮件）的数据集以及我们想要完成的任务（例如，对垃圾邮件进行分类）。接下来，我们将需要训练模型来预测正确的输出，给定一些输入。在训练过程中，模型内部的参数将根据模型在输入数据上的执行情况而变化。例如，为了对垃圾邮件进行分类，我们的算法可能会发现某些单词比其他单词更好地预测垃圾邮件，因此将与这些单词相关联的参数加权更高。最后，受过训练的模型会发现某些单词应该比其他单词具有更大的影响力（因为它们与垃圾邮件的一致关联）。训练过程的输出就是我们所说的模型。然后可以使用模型来获得洞察力或进行未来预测。要进行预测，您将为模型提供输入，并根据这些输入的数学操作生成输出。使用分类这个例子，在给定电子邮件的属性的情况下，它将通过与历史垃圾邮件进行比较来预测该电子邮件是否为垃圾邮件，而不是通过与之进行过训练的垃圾邮件进行比较。


However, just training a model isn’t the objective—we want to leverage our model to produce insights. Thus, we must answer the question: how do we know our model is any good at what it’s supposed to do? That’s where model tuning and evaluation come in.

然而，仅仅训练模型并不是目标——我们希望利用我们的模型来产生洞察力。因此，我们必须回答这个问题：我们怎么知道我们的模型对它应该做的事情有多好？这就是模型调整和评估的用武之地。

#### <font color="#3399cc">Model tuning and evaluation 模型调整和评估</font>

You likely noticed earlier that we mentioned that you should split your data into multiple portions and use only one for training. This is an essential step in the machine learning process because when you build an advanced analytics model you want that model to generalize to data it has not seen before. Splitting our dataset into multiple portions allows us to objectively  test the effectiveness of the trained model against a set of data that it has never seen before. The objective is to see if your model understands something fundamental about this data process or whether or not it just noticed the things particular to only the training set (sometimes called overfitting). That’s why it is called a test set. In the process of training models, we also might take another, separate subset of data and treat that as another type of test set, called a validation set, in order to try out different hyperparameters (parameters that affect the training process) and compare different variations of the same model without overfitting to the test set.

您之前可能已经注意到我们提到您应该将数据分成多个部分，并且只使用一个部分进行训练。这是机器学习过程中必不可少的一步，因为当您构建高级分析模型时，您希望该模型能够泛化到（推广到）之前从未见过的数据。将我们的数据集拆分成多个部分使我们能够客观地测试训练模型对一组前所未有的数据的有效性。目标是看你的模型是否理解这个数据过程的基本内容，或者它是否只注意到训练集特有的东西（有时称为过度拟合）。这就是为什么它被称为测试集。在训练模型的过程中，我们也可以采用另一个单独的数据子集，并将其视为另一种类型的测试集，称为验证集，以便尝试不同的超参数（影响训练过程的参数）并比较相同模型的不同变体（比如：维度相同，超参数不同）而不会过度拟合测试集。

---

<center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong></font></center>
Following proper training, validation, and test set best practices is essential to successfully using machine learning. It’s easy to end up overfitting (training a model that does not generalize well to new data) if we do not properly isolate these sets of data. We cannot cover this problem in depth in this book, but almost any machine learning book will cover this topic. 

经过适当的训练，验证和测试集最佳实践对于成功使用机器学习至关重要。如果我们没有正确地隔离这些数据集，很容易最终过度拟合（训练一个不能很好地泛化到新数据的模型）。我们无法在本书中深入探讨这个问题，但几乎所有的机器学习书都将涵盖这一主题。

---

To continue with the classification example we referenced previously, we have three sets of data: a training set for training models, a validation set for testing different variations of the models that we’re training, and lastly, a test set we will use for the final evaluation of our different model variations to see which one performed the best. 

为了继续我们之前引用的分类示例，我们有三组数据：训练模型的训练集，用于测试我们正在训练的模型的不同变体的验证集，最后对于最后评估我们不同的模型变体，我们将使用测试集看看哪一个表现最好。

#### <font color="#3399cc" >Leveraging the model and/or insights 利用模型和/或洞悉力</font>

After running the model through the training process and ending up with a well-performing model, you are now ready to use it! Taking your model to production can be a significant challenge in and of itself. We will discuss some tactics later on in this chapter. 

在通过训练过程运行模型并最终获得性能良好的模型后，您现在可以使用它了！将您的模型投入生产可能是一项重大挑战。我们将在本章后面讨论一些策略。


## <font color="#9a161a" >Spark’s Advanced Analytics Toolkit Spark的高级分析工具包</font>

The previous overview is just an example workflow and doesn’t encompass all use cases or potential workflows. In addition, you probably noticed that we did not discuss Spark almost at all. This section will discuss Spark’s advanced analytics capabilities. Spark includes several core packages and many external packages for performing advanced analytics. The primary package is MLlib, which provides an interface for building machine learning pipelines.

前面的概述只是一个示例工作流程，并不包含所有用户案例或潜在的工作流程。另外，您可能已经注意到我们几乎没有讨论过 Spark。本节将讨论 Spark 的高级分析功能。 Spark 包含多个核心软件包和许多用于执行高级分析的外部软件包。主要包是 MLlib ，它提供了用于构建机器学习管道的接口。

### <font color="#000000">What Is MLlib? 什么是MLlib？</font>

MLlib is a package, built on and included in Spark, that provides interfaces for gathering and cleaning data, feature engineering and feature selection, training and tuning large-scale supervised and unsupervised machine learning models, and using those models in production.


MLlib 是一个基于 Spark 构建的软件包，它提供了用于收集和清理数据，特征工程和特征选择，训练和调优大规模监督和无监督机器学习模型以及在生产中使用这些模型的接口。

---

<center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong></font></center> 
MLlib actually consists of two packages that leverage different core data structures. The package `org.apache.spark.ml` includes an interface for use with DataFrames. This package also offers a high-level interface for building machine learning pipelines that help standardize the way in which you perform the preceding steps. The lower-level package, `org.apache.spark.mllib`, includes interfaces for Spark’s low-level RDD APIs. This book will focus exclusively on the DataFrame API. The RDD API is the lower-level interface, which is in maintenance mode (meaning it will only receive bug fixes, not new features) at this time. It has also been covered fairly extensively in older books on Spark and is therefore omitted here.

MLlib 实际上由两个利用不同核心数据结构的包组成。包 `org.apache.spark.ml` 包含一个使用 DataFrames 的接口。该软件包还提供了一个用于构建机器学习管道的高层次接口，有助于标准化执行上述步骤的方式。较低层次的包 `org.apache.spark.mllib` 包含 Spark 的低层次 RDD API 的接口。本书将专注于 DataFrame API。 RDD API 是较低层次的接口，它处于维护模式（意味着它目前只接收错误修复，而不是新功能）。它在Spark的旧书中也得到了相当广泛的介绍，因此在此省略。

---

#### <font color="#3399cc">When and why should you use MLlib (versus scikit-learn, TensorFlow, or foo package) 何时以及为什么要使用MLlib（与scikit-learn，TensorFlow或foo包相比）</font>

At a high level, MLlib might sound like a lot of other machine learning packages you’ve probably heard of, such as scikit-learn for Python or the variety of R packages for performing similar tasks. So why should you bother with MLlib at all? There are numerous tools for performing machine learning on a single machine, and while there are several great options to choose from, these single machine tools do have their limits either in terms of the size of data you can train on or the processing time.

在很高的层次上，MLlib 可能听起来像你可能听说过的许多其他机器学习包，例如用于 Python 的 scikit-learn 或用于执行类似任务的各种 R 包。那你为什么要操心 MLlib 呢？有许多工具可以在一台机器上进行机器学习，虽然有几个很好的选择可供选择，但这些单机在你可以训练的数据大小或处理时间方面都有其局限性。

This means single-machine tools are usually complementary to MLlib. When you hit those scalability issues, take advantage of Spark’s abilities.

这意味着单机工具通常是 MLlib 的补充。当您遇到这些可伸缩性问题时，请充分利用Spark的功能。


There are two key use cases where you want to leverage Spark’s ability to scale. <font style="color:#EA7500">First</font>, you want to leverage Spark for preprocessing and feature generation to reduce the amount of time it might take to produce training and test sets from a large amount of data. <font style="color:#EA7500">Then</font> you might leverage single-machine learning libraries to train on those given data sets. <font style="color:#EA7500">Second</font>, when your input data or model size become too difficult or inconvenient to put on one machine, use Spark to do the heavy lifting. Spark makes distributed machine learning very simple.

有两个关键用户案例，在这些案例中您希望利用Spark的伸缩能力。<font style="color:#EA7500">首先</font>，您希望利用Spark进行预处理和特征生成，以减少从大量数据生成训练和测试集所需的时间。<font style="color:#EA7500">然后</font>，您可以利用单机学习库来训练那些给定的数据集。<font style="color:#EA7500">其次</font>，当您的输入数据或模型大小变得太难或不方便放在一台机器上时，请使用Spark来完成繁重的工作。 Spark使分布式机器学习变得非常简单。

An important caveat to all of this is that while training and data preparation are made simple, there are still some complexities you will need to keep in mind, especially when it comes to deploying a trained model. For example, Spark does not provide a built-in way to serve low-latency predictions from a model, so you may want to export the model to another serving system or a custom application to do that. MLlib is generally designed to allow inspecting and exporting models to other tools where possible. 

所有这一切的一个重要警告是，虽然训练和数据准备工作变得简单，但仍需要记住一些复杂性，特别是在部署经过训练的模型时。例如，Spark 不提供从模型提供低延迟预测的内置方法，因此您可能希望将模型导出到另一个服务系统或自定义应用程序来执行此操作。 MLlib 通常设计为允许在可能的情况下检查和导出模型到其他工具。

## <font color="#9a161a" >High-Level MLlib Concepts</font>

In MLlib there are several fundamental “structural” types: transformers, estimators, evaluators, and pipelines. By structural, we mean you will think in terms of these types when you define an end-to-end machine learning pipeline. They’ll provide the common language for defining what belongs in what part of the pipeline. Figure 24-2 illustrates the overall workflow that you will follow when developing machine learning models in Spark. 

在 MLlib 中，有几种基本的“结构”类型：转换器（transformer），估计器（estimator），评估器（evaluator）和管道（pipeline）。 通过结构，我们的意思是当您定义端到端机器学习管道时，您将考虑这些类型。 它们将提供用于定义属于管道的哪些部分的通用语言。 图24-2说明了在 Spark 中开发机器学习模型时要遵循的整体工作流程。

![1567256299402](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter24/1567256299402.png)

***Transformers*** are functions that convert raw data in some way. This might be to create a new interaction variable (from two other variables), normalize a column, or simply change an Integer into a  `Double`  type to be input into a model. An example of a transformer is one that converts string categorical variables into numerical values that can be used in MLlib. Transformers are primarily used in preprocessing and feature engineering. Transformers take a DataFrame as input and produce a new DataFrame as output, as illustrated in Figure 24-3. 

**转换器（transformer）**是以某种方式转换原始数据的函数。 这可能是创建一个新的交互变量（来自另外两个变量），规范化一个列，或者只是将一个 Integer 更改为  `Double`  类型以输入到模型中。 转换器（transformer）的示例是将字符串分类变量转换为可在 MLlib 中使用的数值的转换器（transformer）。 转换器（transformer）主要用于预处理和特征工程。 转换器（transformer）将 DataFrame 作为输入并生成一个新的 DataFrame 作为输出，如图 24-3 所示。

![1567256344807](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter24/1567256344807.png)

***Estimators*** are one of two kinds of things. First, estimators can be a kind of transformer that is initialized with data. For instance, to normalize numerical data we’ll need to initialize our transformation with some information about the current values in the column we would like to normalize. This requires two passes over our data—the initial pass generates the initialization values and the second actually applies the generated function over the data. In the Spark’s nomenclature, algorithms that allow users to train a model from data are also referred to as estimators.

***估计器（estimator）***是两种事物之一。首先，估计器（estimator）可以是一种用数据初始化的转换器（transformer）。例如，为了标准化数值数据，我们需要使用有关我们想要标准化的列中的当前值的一些信息来初始化我们的转换。这需要对数据进行两次传递——初始传递生成初始化值，第二次实际应用于在数据上生成的函数。在Spark的命名法中，允许用户从数据训练模型的算法也称为估计器（estimator）。


An ***evaluator*** allows us to see how a given model performs according to criteria we specify like a [receiver operating characteristic (ROC) curve](http://mlwiki.org/index.php/ROC_Analysis). After we use an evaluator to select the best model from the ones we tested, we can then use that model to make predictions.

***评估器（evaluator ）***允许我们根据我们指定的标准（如[受试者特性（ROC）曲线](http://mlwiki.org/index.php/ROC_Analysis)）查看给定模型的执行情况。在我们使用评估器（evaluator ）从我们测试的模型中选择最佳模型之后，我们可以使用该模型进行预测。

From a high level we can specify each of the transformations, estimations, and evaluations one by one, but it is often easier to specify our steps as stages in a pipeline. This pipeline is similar to scikit-learn’s pipeline concept.

从高层次我们可以逐个指定每个转换（ transformation），估计（estimation）和评估（evaluation），但通常更容易将我们的步骤指定为管道中的阶段。此管道类似于 scikit-learn 的管道概念。

### <font color="#00000">Low-level data types 低层次数据类型</font>

In addition to the structural types for building pipelines, there are also several lower-level data types you may need to work with in MLlib (Vector being the most common). Whenever we pass a set of features into a machine learning model, we must do it as a vector that consists of Doubles. This vector can be either sparse (where most of the elements are zero) or dense (where there are many unique values). Vectors are created in different ways. To create a dense vector, we can specify an array of all the values. To create a sparse vector, we can specify the total size and the indices and values of the non-zero elements. Sparse is the best format, as you might have guessed, when the majority of values are zero as this is a more compressed representation. Here is an example of how to manually create a Vector: 

除了构建管道的结构类型之外，还有一些您可能需要在 MLlib 中使用的较低层次的数据类型（向量（Vector）是最常见的）。每当我们将一组特征传递给机器学习模型时，我们必须将其作为由双精度（ `Double` ）组成的向量。此向量可以是稀疏的（大多数元素为零）或密集（有许多唯一值）。向量（Vector）以不同方式创建。要创建密集向量，我们可以指定所有值的数组。要创建稀疏向量，我们可以指定总大小以及非零元素的索引和值。当大多数值为零时，稀疏是最好的格式，正如您可能已经猜到的那样，因为这是一种更加压缩的表示。以下是如何手动创建向量（Vector）的示例：

```scala
// in Scala
import org.apache.spark.ml.linalg.Vectorsval denseVec = Vectors.dense(1.0, 2.0, 3.0)
val size = 3
val idx = Array(1,2) // locations of non-zero elements in vector
val values = Array(2.0,3.0)
val sparseVec = Vectors.sparse(size, idx, values)
sparseVec.toDense
denseVec.toSparse
# in Python
from pyspark.ml.linalg import Vectors
denseVec = Vectors.dense(1.0, 2.0, 3.0)
size = 3
idx = [1, 2] # locations of non-zero elements in vector
values = [2.0, 3.0]
sparseVec = Vectors.sparse(size, idx, values)
```

---

<center><font face="constant-width" color="#c67171" size=4><strong>WARNING 警告</strong></font></center>
Confusingly, there are similar datatypes that refer to ones that can be used in DataFrames and others that can only be used in RDDs. The RDD implementations fall under the mllib package while the DataFrame implementations fall under ml. 

令人困惑的是，有类似的数据类型可以参考在 DataFrame 中使用的数据类型以及其他只能在RDD中使用的数据类型。 RDD 实现属于 mllib 包，而 DataFrame 实现属于 ml 。

----

## <font color="#9a161a" >MLlib in Action</font>

Now that we have described some of the core pieces you can expect to come across, let’s create a simple pipeline to demonstrate each of the components. We’ll use a small synthetic dataset that will help illustrate our point. Let’s read the data in and see a sample before talking about it further: 

现在我们已经描述了您可能会遇到的一些核心部分，让我们创建一个简单的管道来演示每个组件。 我们将使用一个小的合成数据集来帮助说明我们的观点。 在进一步讨论之前，让我们读取数据并查看示例：


Here’s a sample of the data: 

这是一个数据样本：

```scala
// in Scala
var df = spark.read.json("/data/simple-ml")
df.orderBy("value2").show()
```

```scala
# in Python
df = spark.read.json("/data/simple-ml")
df.orderBy("value2").show()
```

```text
+-----+----+------+------------------+
|color| lab|value1|            value2|
+-----+----+------+------------------+
|green|good|     1|14.386294994851129|
|green| bad|    16|14.386294994851129|
| blue| bad|     8|14.386294994851129|
| blue| bad|     8|14.386294994851129|
+-----+----+------+------------------+
only showing top 20 rows
```

This dataset consists of a categorical label with two values (good or bad), a categorical variable (color), and two numerical variables. While the data is synthetic, let’s imagine that this dataset represents a company’s customer health. The “color” column represents some categorical health rating made by a customer service representative. The “lab” column represents the true customer health. The other two values are some numerical measures of activity within an application (e.g., minutes spent on site and purchases). Suppose that we want to train a classification model where we hope to predict a binary variable—the label—from the other values. 

此数据集由具有两个值（好或坏），分类变量（颜色）和两个数值变量的分类标签组成。虽然数据是合成的，但我们假设这个数据集代表了公司的客户健康状况。 “color”列表示由客户服务代表做出的某些分类健康评级。 “lab”列代表真正的客户健康状况。其他两个值是应用程序内活动的一些数字度量（例如，在站点和购买上花费的分钟数）。假设我们想要训练一个分类模型，我们希望从其他值预测二进制变量——标签。

---

<center><font face="constant-width" color="#737373" size=4><strong>TIP 提示</strong></font></center>
Apart from JSON, there are some specific data formats commonly used for supervised learning, including LIBSVM. These formats have real valued labels and sparse input data. Spark can read and write for these formats using its data source API. Here’s an example of how to read in data from a `libsvm` file using that Data Source API.

除了 JSON 之外，还有一些通常用于监督学习的特定数据格式，包括 LIBSVM 。这些格式具有实值标签和稀疏输入数据。 Spark 可以使用其数据源 API 读取和写入这些格式。以下是如何使用该 Data Source API 从 `libsvm` 文件读取数据的示例。

```scala
spark.read.format("libsvm").load("/data/sample_libsvm_data.txt")
```
For more information on LIBSVM, see [the documentation](https://www.csie.ntu.edu.tw/~cjlin/libsvm/). 

有关 LIBSVM 的更多信息，请参阅[文档](https://www.csie.ntu.edu.tw/~cjlin/libsvm/)。

---

### <font color="#000000">Feature Engineering with Transformers 使用Transformer进行特征工程</font>

As already mentioned, transformers help us manipulate our current columns in one way or another. Manipulating these columns is often in pursuit of building features (that we will input into our model).

如前所述，转换器（transformer）帮助我们以某种方式操纵当前列。操纵这些列通常是为了追求构建特征（我们将输入到我们的模型中）。

 Transformers exist to either cut down the number of features, add more features, manipulate current ones, or simply to help us format our data correctly. Transformers add new columns to DataFrames. When we use MLlib, all inputs to machine learning algorithms (with several exceptions discussed in later chapters) in Spark must consist of type `Double` (for labels) and `Vector[Double]` (for features).

转换器（transformer）可以减少特征数量，添加更多特征，控制当前特征，或者只是帮助我们正确地格式化数据。转换器（transformer）向 DataFrames 添加新列。当我们使用 MLlib 时，Spark 中机器学习算法的所有输入（在后面的章节中讨论了几个例外）必须由类型 `Double`（用于标签）和 `Vector[Double]`（用于特征）。

The current dataset does not meet that requirement and therefore we need to transform it to the proper format. To achieve this in our example, we are going to specify an RFormula. This is a declarative language for specifying machine learning transformations and is simple to use once you understand the syntax. RFormula supports a limited subset of the R operators that in practice work quite well for simple models and manipulations (we demonstrate the manual approach to this problem in Chapter 25). The basic RFormula operators are: 

当前数据集不符合该要求，因此我们需要将其转换为适当的格式。为了在我们的示例中实现这一点，我们将指定一个 RFormula。这是一种用于指定机器学习转换的声明性语言，一旦理解了语法，就很容易使用。 RFormula 支持R运算符的有限子集，这些运算符实际上对于简单模型和操作非常有效（我们在第25章中演示了解决此问题的手动方法）。基本的 RFormula 运算符（operator）是：

| 运算符operator                                               | 含义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <font face="constant-width" color="#000000" size=4><strong>~</strong></font> | Separate target and terms<br/>分隔目标和项                   |
| <font face="constant-width" color="#000000" size=4><strong>+</strong></font> | Concat terms; “+ 0” means removing the intercept (this means that the y-intercept of the line that we will fit will be 0)<br/>拼接项；“+0” 意味着移除截距（这意思是我们将拟合的直线的 y轴截距将会是0） |
| <font face="constant-width" color="#000000" size=4><strong>-</strong></font> | Remove a term; “- 1” means removing the intercept (this means that the y-intercept of the line that we will fit will be 0—yes, this does the same thing as “+ 0”<br/>移除一个项；“-1” 与 “-0” 做同样的事情 |
| <font face="constant-width" color="#000000" size=4><strong>:</strong></font> | Interaction (multiplication for numeric values, or binarized categorical values)<br />交互（数值乘法或二进制分类值） |
| <font face="constant-width" color="#000000" size=4><strong>.</strong></font> | All columns except the target/dependent variable<br />除目标/因变量之外的所有列 |


In order to specify transformations with this syntax, we need to import the relevant class. Then we go through the process of defining our formula. In this case we want to use all available variables (the `.`) and also add in the interactions between value1 and color and value2 and color, treating those as new features: 

为了使用此语法指定转换（transformation），我们需要导入相关的类。 然后我们将完成定义公式的过程。 在这种情况下，我们想要使用所有可用的变量（`.`），并添加 `value1` 和 `color` 以及 `value2` 和 `color` 之间的交互，将它们视为新功能：

```scala
// in Scala
import org.apache.spark.ml.feature.RFormula
val supervised = new RFormula().setFormula("lab ~ . + color:value1 + color:value2")
```

```python
# in Python
from pyspark.ml.feature import RFormula
supervised = RFormula(formula="lab ~ . + color:value1 + color:value2")
```

At this point, we have declaratively specified how we would like to change our data into what we will train our model on. The next step is to fit the RFormula transformer to the data to let it discover the possible values of each column. Not all transformers have this requirement but because RFormula will automatically handle categorical variables for us, it needs to determine which columns are categorical and which are not, as well as what the distinct values of the categorical columns are. For this reason, we have to call the `fit` method. Once we call `fit`, it returns a “trained” version of our transformer we can then use to actually transform our data.

在这一点上，我们已声明指定我们如何将数据更改为我们将训练模型的内容。 下一步是将 RFormula 转换器（transformer）拟合数据，以便发现每列的可能值。 并非所有转换器（transformer）都有此要求，但由于RFormula 会自动为我们处理分类变量，因此需要确定哪些列是分类的，哪些不是，以及分类列的不同值是什么。 出于这个原因，我们必须调用 `fit` 方法。 一旦我们调用了 `fit`，它就会返回我们转换器（transformer）的“经过训练”的版本，然后我们可以使用它来实际转换我们的数据。

---

<center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong></font></center>
We’re using the RFormula transformer because it makes performing several transformations extremely easy to do. In Chapter 25, we’ll show other ways to specify a similar set of transformations and outline the component parts of the RFormula when we cover the specific transformers in MLlib.

我们正在使用 RFormula 转换器（transformer），因为它使得执行几次变换非常容易。 在第25章中，我们将展示指定一组类似的转换（transformation）的其他方法，并在我们覆盖 MLlib 中的特定转换器（transformer）时概述 RFormula 的组成部分。

---

Now that we covered those details, let’s continue on and prepare our DataFrame: 

现在我们已经介绍了这些细节，让我们继续并准备我们的 DataFrame：

```scala
// in Scala
val fittedRF = supervised.fit(df)
val preparedDF = fittedRF.transform(df)
preparedDF.show()
```

```python
# in Python
fittedRF = supervised.fit(df)
preparedDF = fittedRF.transform(df)
preparedDF.show()
```

Here’s the output from the training and transformation process: 

以下是训练和转换过程的输出：

```text
+-----+----+------+------------------+----------------------------------------------------------------------+-----+
|color|lab |value1|value2            |features                                                              |label|
+-----+----+------+------------------+----------------------------------------------------------------------+-----+
|green|good|1     |14.386294994851129|(10,[1,2,3,5,8],[1.0,1.0,14.386294994851129,1.0,14.386294994851129])  |1.0  |
|blue |bad |8     |14.386294994851129|(10,[2,3,6,9],[8.0,14.386294994851129,8.0,14.386294994851129])        |0.0  |
|blue |bad |12    |14.386294994851129|(10,[2,3,6,9],[12.0,14.386294994851129,12.0,14.386294994851129])      |0.0  |
+-----+----+------+------------------+----------------------------------------------------------------------+-----+
```

In the output we can see the result of our transformation—a column called features that has our previously raw data. What’s happening behind the scenes is actually pretty simple. RFormula inspects our data during the `fit` call and outputs an object that will transform our data according to the specified formula, which is called an RFormulaModel. This “trained” transformer always has the word Model in the type signature. 

在输出中，我们可以看到转换的结果——一个名为 features 的列，它包含我们以前的原始数据。幕后发生的事实上非常简单。 RFormula 在 `fit` 调用期间检查我们的数据，并输出一个对象，该对象将根据指定的公式（称为 RFormulaModel ）转换我们的数据。这种“训练有素”的转换器（transformer）在类型签名中始终具有单词Model。


When we use this transformer, Spark automatically converts our categorical variable to Doubles so that we can input it into a (yet to be specified) machine learning model. In particular, it assigns a numerical value to each possible color category, creates additional features for the interaction variables between colors and value1/value2, and puts them all into a single vector. We then call transform on that object in order to transform our input data into the expected output data.

当我们使用这个转换器时，Spark会自动将我们的分类变量转换为双精度变量，以便我们可以将它输入到（尚未指定的）机器学习模型中。 特别是，它为每个可能的颜色类别指定一个数值，为颜色和 value1/value2 之间的交互变量创建额外的特征，并将它们全部放入一个单独向量中。 然后，我们调用该对象的变换，以便将输入数据转换为预期的输出数据。


Thus far you (pre)processed the data and added some features along the way. Now it is time to actually train a model (or a set of models) on this dataset. In order to do this, you first need to prepare a test set for evaluation.

到目前为止，您（之前）处理过数据并在此过程中添加了一些特征。现在是时候在这个数据集上实际训练一个模型（或一组模型）了。为此，您首先需要准备一个用于评估的测试集。

---

<center><font face="constant-width" color="#737373" size=4><strong>TIP 提示</strong></font></center>
Having a good test set is probably the most important thing you can do to ensure you train a model you can actually use in the real world (in a dependable way). Not creating a representative test set or using your test set for hyperparameter tuning are surefire ways to create a model that does not perform well in real-world scenarios. Don’t skip creating a test set—it’s a requirement to know how well your model actually does!

拥有一个好的测试集可能是你可以做的最重要的事情，以确保你训练一个你可以在现实世界中实际使用的模型（以可靠的方式）。 不创建代表性测试集或使用测试集进行超参数调整是创建在实际场景中表现不佳的模型的绝对方法。 不要跳过创建测试集——这是要求了解模型的实际效果！

---

Let’s create a simple test set based off a random split of the data now (we’ll be using this test set throughout the remainder of the chapter): 

让我们现在根据数据的随机分割创建一个简单的测试集（我们将在本章的其余部分使用这个测试集）：

```scala
// in Scala
val Array(train, test) = preparedDF.randomSplit(Array(0.7, 0.3))
```

```python
# in Python
train, test = preparedDF.randomSplit([0.7, 0.3])
```

### <font color="#000000">Estimators 估计</font>

Now that we have transformed our data into the correct format and created some valuable features, it’s time to actually fit our model. In this case we will use a classification algorithm called logistic regression. To create our classifier we instantiate an instance of `LogisticRegression`, using the default configuration or hyperparameters. We then set the label columns and the feature columns; the column names we are setting—label and features—are actually the default labels for all estimators in Spark MLlib, and in later chapters we omit them: 

现在我们已经将数据转换为正确的格式并创建了一些有价值的特征，现在是时候实际拟合我们的模型了。 在这种情况下，我们将使用称为逻辑回归的分类算法。 要创建我们的分类器，我们使用默认配置或超参数来实例化`LogisticRegression ` 的实例。 然后我们设置标签列和特征列；我们设置的列名称——标签和特征——实际上是Spark MLlib 中所有估计器的默认标签，在后面的章节中我们忽略它们：


```scala
// in Scala
import org.apache.spark.ml.classification.LogisticRegression
val lr = new LogisticRegression().setLabelCol("label").setFeaturesCol("features")
```

```python
# in Python
from pyspark.ml.classification import LogisticRegression
lr = LogisticRegression(labelCol="label",featuresCol="features")
```

Before we actually go about training this model, let’s inspect the parameters. This is also a great way to remind yourself of the options available for each particular model: 

在我们真正开始训练这个模型之前，让我们检查参数。 这也是提醒自己每种特定模型可用选项的好方法：

```scala
// in Scala
println(lr.explainParams())
```

```python
# in Python
print lr.explainParams()
```

While the output is too large to reproduce here, it shows an explanation of all of the parameters for Spark’s implementation of logistic regression. The `explainParams` method exists on all algorithms available in MLlib.

虽然输出太大而无法在此重现，但它显示了 Spark 实现逻辑回归的所有参数的解释。 在 MLlib 中可用的所有算法都有 `explainParams` 方法。

Upon instantiating an untrained algorithm, it becomes time to fit it to data. In this case, this returns a `LogisticRegressionModel`: 

在实例化未经训练的算法时，就可以将它与数据相弥合。在这种情况下，这将返回 `LogisticRegressionModel `：

```scala
// in Scala
val fittedLR = lr.fit(train)
```

```python
# in Python
fittedLR = lr.fit(train)
```

This code will kick off a Spark job to train the model. As opposed to the transformations that you saw throughout the book, the fitting of a machine learning model is eager and performed immediately.

此代码将启动 Spark 作业以训练模型。 与您在本书中看到的转换相反，机器学习模型的拟合非常迫切并且立即执行。

Once complete, you can use the model to make predictions. Logically this means tranforming features into labels. We make predictions with the transform method. For example, we can transform our training dataset to see what labels our model assigned to the training data and how those compare to the true outputs. This, again, is just another DataFrame we can manipulate. Let’s perform that prediction with the following code snippet: 

完成后，您可以使用该模型进行预测。 从逻辑上讲，这意味着将特征转换为标签。 我们使用transform方法进行预测。 例如，我们可以转换训练数据集，以查看我们的模型分配给训练数据的标签以及这些标签与真实输出的比较。 这又是我们可以操作的另一个DataFrame。 让我们使用以下代码片段执行该预测：

```scala
fittedLR.transform(train).select("label", "prediction").show()
```

This results in:

结果：

```text
+-----+----------+
|label|prediction|
+-----+----------+
| 0.0 |    0.0   |
...
| 0.0 |    0.0   |
+-----+----------+ 
```

Our next step would be to manually evaluate this model and calculate performance metrics like the true positive rate, false negative rate, and so on. We might then turn around and try a different set of parameters to see if those perform better. However, while this is a useful process, it can also be quite tedious. Spark helps you avoid manually trying different models and evaluation criteria by allowing you to specify your workload as a declarative pipeline of work that includes all your transformations as well as tuning your hyperparameters.

我们的下一步是手动评估此模型并计算性能指标，如真阳性率，假阴性率等。 然后我们可以转去尝试一组不同的参数，看看这些参数是否表现更好。 然而，虽然这是一个有用的过程，但它也可能非常繁琐。 Spark允许您将工作负载指定为声明式的管道，其中包含所有转换以及调整超参数的工作，从而帮助您避免手动尝试不同的模型和评估标准。

<div style="background:#f7f7f7">
    <center><font face="constant-width" color="#000000" size=4><strong>A REVIEW OF HYPERPARAMETERS 超参数回顾</strong></font></center></br>
Although we mentioned them previously, let’s more formally define hyperparameters. Hyperparameters are configuration parameters that affect the training process, such as model architecture and regularization. They are set prior to starting training. For instance, logistic regression has a hyperparameter that determines how much regularization should be performed on our data through the training phase (regularization is a technique that pushes models against overfitting data). You’ll see in the next couple of pages that we can set up our pipeline to try different hyperparameter values (e.g., different regularization values) in order to compare different variations of the same model against one another.</br></br>
虽然我们之前提到过它们，但我们更正式地定义了超参数。 超参数是影响训练过程的配置参数，例如模型架构和正则化。 它们在开始训练之前设定。 例如，逻辑回归有一个超参数，用于确定应通过训练阶段对我们的数据执行多少正则化（正则化是一种推动模型对抗过度拟合数据的技术）。 您将在接下来的几页中看到，我们可以设置我们的管道以尝试不同的超参数值（例如，不同的正则化值），以便比较相同模型相对于彼此的不同变化。
</div>

### <font color="#00000">Pipelining Our Workflow 流水线我们的工作流程</font>


As you probably noticed, if you are performing a lot of transformations, writing all the steps and keeping track of DataFrames ends up being quite tedious. That’s why Spark includes the Pipeline concept. A pipeline allows you to set up a dataflow of the relevant transformations that ends with an estimator that is automatically tuned according to your specifications, resulting in a tuned model ready for use. Figure 24-4 illustrates this process. 

正如您可能已经注意到的，如果您正在执行大量转换，那么编写所有步骤并跟踪 DataFrames 最终会非常繁琐。 这就是 Spark 包含 Pipeline 概念的原因。 管道允许您设置相关转换的数据流，该数据流以估计器（estimator）结束，而估计器（estimator）根据您设置的规范自动调整，从而使得可以使用的调整模型。 图24-4说明了这个过程。

![1567261735442](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter24/1567261735442.png)

Note that it is essential that instances of transformers or models are not reused across different pipelines. Always create a new instance of a model before creating another pipeline.

请注意，转换器（transformer）或模型的实例不能在不同的管道中重复使用。 始终在创建另一个管道之前创建模型的新实例。

In order to make sure we don’t overfit, we are going to create a holdout test set and tune our hyperparameters based on a validation set (note that we create this validation set based on the original dataset, not the preparedDF used in the previous pages) :  

为了确保我们不过度拟合，我们将创建一个留出法（holdout） 测试集并基于验证集调整我们的超参数（请注意，我们基于原始数据集创建此验证集，而不是之前页面使用的 preparedDF）：


```scala
// in Scala
val Array(train, test) = df.randomSplit(Array(0.7, 0.3))
```

```python
# in Python
train, test = df.randomSplit([0.7, 0.3])
```

Now that you have a holdout set, let’s create the base stages in our pipeline. A stage simply represents a transformer or an estimator. In our case, we will have two estimators. The `RFomula` will first analyze our data to understand the types of input features and then transform them to create new features. Subsequently, the `LogisticRegression` object is the algorithm that we will train to produce a model: 

既然你有一个 留出法（holdout） 集合，让我们在我们的管道中创建基础阶段。 阶段只代表转换器（transformer）或估计器（estimator）。 在我们的例子中，我们将有两个估计器（estimator）。 `RFomula` 将首先分析我们的数据，以了解输入特征的类型，然后转换它们以创建新特征。 随后，`LogisticRegression` 对象是我们将训练生成模型的算法：

```scala
// in Scala
val rForm = new RFormula()
val lr = new LogisticRegression().setLabelCol("label").setFeaturesCol("features")
```

```python
# in Python
rForm = RFormula()
lr = LogisticRegression().setLabelCol("label").setFeaturesCol("features")
```

We will set the potential values for the RFormula in the next section. Now instead of manually using our transformations and then tuning our model we just make them stages in the overall pipeline, as in the following code snippet: 

我们将在下一节中设置 `RFormula` 的潜在值。 现在，不是手动使用我们的转换然后调整我们的模型，而是在整个管道中创建它们，如下面的代码片段所示：

```scala
// in Scala
import org.apache.spark.ml.Pipeline
val stages = Array(rForm, lr)
val pipeline = new Pipeline().setStages(stages)
```

```python
# in Python
from pyspark.ml import Pipeline
stages = [rForm, lr]
pipeline = Pipeline().setStages(stages)
```

### <font color="#00000">Training and Evaluation 训练和评估</font>

Now that you arranged the logical pipeline, the next step is training. In our case, we won’t train just one model (like we did previously); we will train several variations of the model by specifying different combinations of hyperparameters that we would like Spark to test. We will then select the best model using an `Evaluator` that compares their predictions on our validation data. We can test different hyperparameters in the entire pipeline, even in the RFormula that we use to manipulate the raw data. This code shows how we go about doing that: 

既然您已经安排了逻辑管道，那么下一步就是训练。 在我们的例子中，我们不会只训练一个模型（就像我们之前做过的那样）; 我们将通过指定我们希望  Spark  测试的超参数的不同组合来训练模型的几种变体。 然后，我们将使用评估器（`Evaluator`） 选择最佳模型，该评估器（evaluator ）将对其验证数据的预测进行比较。 我们可以在整个管道中测试不同的超参数，甚至可以在我们用来操作原始数据的 `RFormula` 中测试。 此代码显示了我们如何做到这一点：

```scala
// in Scala
import org.apache.spark.ml.tuning.ParamGridBuilder
val params = new ParamGridBuilder()
.addGrid(rForm.formula, Array(
"lab ~ . + color:value1",
"lab ~ . + color:value1 + color:value2"))
.addGrid(lr.elasticNetParam, Array(0.0, 0.5, 1.0))
.addGrid(lr.regParam, Array(0.1, 2.0))
.build()
```

```python
# in Python
from pyspark.ml.tuning import ParamGridBuilder
params = ParamGridBuilder()\
.addGrid(rForm.formula, [
"lab ~ . + color:value1",
"lab ~ . + color:value1 + color:value2"])\
.addGrid(lr.elasticNetParam, [0.0, 0.5, 1.0])\
.addGrid(lr.regParam, [0.1, 2.0])\
.build()
```

In our current paramter grid, there are three hyperparameters that will diverge from the defaults:
在我们当前的参数网格中，有三个超参数与默认值不同：

- Two different versions of the RFormula

  两种不同版本的 RFormula

- Three different options for the ElasticNet parameter

  ElasticNet 参数的三个不同选项

- Two different options for the regularization parameter

  正则化参数的两个不同选项

This gives us a total of 12 different combinations of these parameters, which means we will be training 12 different versions of logistic regression. We explain the ElasticNet parameter as well as the regularization options in Chapter 26.

这给了我们这些参数的12种不同组合，这意味着我们将训练12种不同版本的逻辑回归。我们将在第26章中解释ElasticNet参数以及正则化选项。

Now that the grid is built, it’s time to specify our evaluation process. The evaluator allows us to automatically and objectively compare multiple models to the same evaluation metric. There are evaluators for classification and regression, covered in later chapters, but in this case we will use the `BinaryClassificationEvaluator`, which has a number of potential evaluation metrics, as we’ll discuss in Chapter 26. In this case we will use `areaUnderROC`, which is the total area under the receiver operating characteristic, a common measure of classification performance: 

现在网格已经建成，是时候指定我们的评估过程了。评估器（evaluator ）允许我们自动客观地将多个模型在同一评估指标进行比较。有分类和回归的评估器（evaluator ），在后面的章节中有介绍，但在这种情况下，我们将使用 `BinaryClassificationEvaluator`，它有许多潜在的评估指标，我们将在第26章讨论。在这种情况下，我们将使用 `areaUnderROC`，是受试者特性曲线下的总面积，是分类性能的常用度量：


```scala
// in Scala
import org.apache.spark.ml.evaluation.BinaryClassificationEvaluator
val evaluator = new BinaryClassificationEvaluator()
.setMetricName("areaUnderROC")
.setRawPredictionCol("prediction")
.setLabelCol("label")
```

```python
# in Python
from pyspark.ml.evaluation import BinaryClassificationEvaluator
evaluator = BinaryClassificationEvaluator()\
.setMetricName("areaUnderROC")\
.setRawPredictionCol("prediction")\
.setLabelCol("label")
```

Now that we have a pipeline that specifies how our data should be transformed, we will perform model selection to try out different hyperparameters in our logistic regression model and measure success by comparing their performance using the `areaUnderROC` metric.

既然我们有一个管道来指定我们的数据应该如何转换，我们将执行模型选择以在我们的逻辑回归模型中尝试不同的超参数，并通过使用 `areaUnderROC`  指标比较它们的性能来衡量成功。

As we discussed, it is a best practice in machine learning to fit hyperparameters on a validation set (instead of your test set) to prevent overfitting. For this reason, we cannot use our holdout test set (that we created before) to tune these parameters. Luckily, Spark provides two options for performing hyperparameter tuning automatically. We can use `TrainValidationSplit`, which will simply perform an arbitrary random split of our data into two different groups, or `CrossValidator`, which performs K-fold cross-validation by splitting the dataset into k non-overlapping, randomly partitioned folds: 

正如我们所讨论的，机器学习中的最佳实践是在验证集（而不是测试集）上拟合超参数以防止过度拟合。出于这个原因，我们不能使用我们的留出法（holdout） 测试集（我们之前创建的）来调整这些参数。幸运的是，Spark 提供了两种自动执行超参数调整的选项。我们可以使用 `TrainValidationSplit`，它只是将我们的数据任意随机拆分成两个不同的组，或 `CrossValidator`，它通过将数据集拆分为 k 个非重叠，随机分区的部分来执行K折交叉验证：

```scala
// in Scala
import org.apache.spark.ml.tuning.TrainValidationSplit
val tvs = new TrainValidationSplit()
.setTrainRatio(0.75) // also the default.
.setEstimatorParamMaps(params)
.setEstimator(pipeline)
.setEvaluator(evaluator)
```

```python
# in Python
from pyspark.ml.tuning import TrainValidationSplit
tvs = TrainValidationSplit()\
.setTrainRatio(0.75)\
.setEstimatorParamMaps(params)\
.setEstimator(pipeline)\
.setEvaluator(evaluator)
```

Let’s run the entire pipeline we constructed. To review, running this pipeline will test out every version of the model against the validation set. Note the type of `tvsFitted` is `TrainValidationSplitModel`. Any time we fit a given model, it outputs a “model” type :  

让我们运行我们构建的整个管道。 要进行检查，运行此管道将根据验证集测试模型的每个版本。 请注意，`tvsFitted ` 的类型是 `TrainValidationSplitModel` 。 只要我们拟合一个给定的模型，它就会输出一个“模型”类型：

```scala
// in Scala
val tvsFitted = tvs.fit(train)
```

```python
# in Python
tvsFitted = tvs.fit(train)
```

And of course evaluate how it performs on the test set!

当然还要评估它在测试集上的表现！

```scala
evaluator.evaluate(tvsFitted.transform(test)) // 0.9166666666666667
```

We can also see a training summary for some models. To do this we extract it from the pipeline, cast it to the proper type, and print our results. The metrics available on each model are discussed throughout the next several chapters. Here’s how we can see the results:

我们还可以查看某些模型的训练摘要。 为此，我们从管道中提取它，将其转换为合适的类型，然后打印我们的结果。 在接下来的几章中将讨论每种模型上可用的衡量指标。 以下是我们如何看到结果：

```scala
// in Scala
import org.apache.spark.ml.PipelineModel
import org.apache.spark.ml.classification.LogisticRegressionModel
val trainedPipeline = tvsFitted.bestModel.asInstanceOf[PipelineModel]
val TrainedLR = trainedPipeline.stages(1).asInstanceOf[LogisticRegressionModel]
val summaryLR = TrainedLR.summary
summaryLR.objectiveHistory // 0.6751425885789243, 0.5543659647777687, 0.473776...
```

The objective history shown here provides details related to how our algorithm performed over each training iteration. This can be helpful because we can note the progress our algorithm is making toward the best model. Large jumps are typically expected at the beginning, but over time the values should become smaller and smaller, with only small amounts of variation between the values.

此处显示的目标历史记录提供了与我们的算法如何在每次训练迭代中执行的详细信息。 这可能有用，因为我们可以记录我们的算法朝向最佳模型所取得的进展。 通常在开始时预期会出现大跳跃，但随着时间的推移，值会变得越来越小，值之间只有少量变化。

### <font color="#00000">Persisting and Applying Models 持久化和应用模型</font>

Now that we trained this model, we can persist it to disk to use it for prediction purposes later on:

现在我们已经训练了这个模型，我们可以将它保存到磁盘，以便以后用于预测目的：


```scala
tvsFitted.write.overwrite().save("/tmp/modelLocation")
```

After writing out the model, we can load it into another Spark program to make predictions. To do this, we need to use a “model” version of our particular algorithm to load our persisted model from disk. If we were to use `CrossValidator`, we’d have to read in the persisted version as the `CrossValidatorModel`, and if we were to use `LogisticRegression` manually we would have to use `LogisticRegressionModel`. In this case, we use `TrainValidationSplit`, which outputs `TrainValidationSplitModel`:

在写出模型之后，我们可以将它加载到另一个 Spark 程序中进行预测。 为此，我们需要使用特定算法的“模型”版本从磁盘加载持久化模型。 如果我们使用 `CrossValidator`，我们必须在持久化版本中读取`CrossValidatorModel`，如果我们手动使用 `LogisticRegression`，我们将不得不使用`LogisticRegressionModel`。 在这个案例下，我们使用 `TrainValidationSplit`，它输出`TrainValidationSplitModel`：

```scala
// in Scala
import org.apache.spark.ml.tuning.TrainValidationSplitModel
val model = TrainValidationSplitModel.load("/tmp/modelLocation")
model.transform(test) 
```

## <font color="#9a161a">Deployment Patterns</font>

In Spark there are several different deployment patterns for putting machine learning models into production. Figure 24-5 illustrates common workflows. 

在 Spark 中，有几种不同的部署模式可用于将机器学习模型投入生产。 图 24-5 说明了常见的工作流程。

![1567262732536](http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Chapter24/1567262732536.png)

Here are the various options for how you might go about deploying a Spark model. These are the general options you should be able to link to the process illustrated in Figure 24-5.

以下是有关如何部署 Spark 模型的各种选项。这些是您应该能够链接到图24-5中所示流程的通用选项。

Train your machine learning (ML) model offline and then supply it with offline data. In this context, we mean offline data to be data that is stored for analysis, and not data that you need to get an answer from quickly. Spark is well suited to this sort of deployment.

离线训练您的机器学习（ML）模型，然后为其提供离线数据。在这种情况下，我们将离线数据称为存储用于分析的数据，而不是快速获得答案所需的数据。 Spark 非常适合这种部署。

Train your model offline and then put the results into a database (usually a key-value store). This works well for something like recommendation but poorly for something like classification or regression where you cannot just look up a value for a given user but must calculate one based on the input.

离线训练模型，然后将结果放入数据库（通常是键值存储）。这对于像推荐这样的东西很有效，但对于像分类或回归这样的东西来说效果很差，你不仅可以查找给定用户的值，而且必须根据输入计算一个值。

Train your ML algorithm offline, persist the model to disk, and then use that for serving. This is not a low-latency solution if you use Spark for the serving part, as the overhead of starting up a Spark job can be high, even if you’re not running on a cluster. Additionally this does not parallelize well, so you’ll likely have to put a load balancer in front of multiple model replicas and build out some REST API integration yourself. There are some interesting potential solutions to this problem, but no standards currently exist for this sort of model serving.

离线训练 ML 算法，将模型保存到磁盘，然后使用它进行服务。如果您将 Spark 用作服务部分，这不是一个低延迟的解决方案，因为即使您没有在集群上运行，启动Spark作业的开销也很高。此外，这并不能很好地并行化，因此您可能必须在多个模型副本之前放置一个负载均衡器，并自己构建一些REST API集成。这个问题有一些有趣的潜在解决方案，但目前没有这种模型服务的标准。

Manually (or via some other software) convert your distributed model to one that can run much more quickly on a single machine. This works well when there is not too much manipulation of the raw data in Spark but can be hard to maintain over time. Again, there are several solutions in progress. For example, MLlib can export some models to PMML, a common model interchange format.

手动（或通过其他软件）将您的分布式模型转换为可在单台机器上运行得更快的模型。当 Spark 中的原始数据没有太多操作但随着时间的推移很难维护时，这种方法很有效。同样，有几种解决方案正在进行中。例如，MLlib可以将一些模型导出为PMML，这是一种常见的模型交换格式。

Train your ML algorithm online and use it online. This is possible when used in conjunction with Structured Streaming, but can be complex for some models.

在线训练你的ML算法并在线使用它。当与结构化流结合使用时，这是可能的，但对于某些模型可能很复杂。


While these are some of the options, there are many other ways of performing model deployment and management. This is an area under heavy development and many potential innovations are currently being worked on.

虽然这些是一些选项，但还有许多其他方法可以执行模型部署和管理。这是一个正在蓬勃发展的领域，目前正在开展许多潜在的创新。

## <font color="#9a161a">Conclusion 结论</font>

In this chapter we covered the core concepts behind advanced analytics and MLlib. We also showed you how to use them. The next chapter will discuss preprocessing in depth, including Spark’s tools for feature engineering and data cleaning. Then we’ll move into detailed descriptions of each algorithm available in MLlib along with some tools for graph analytics and deep learning. 

在本章中，我们介绍了高级分析和MLlib背后的核心概念。我们还向您展示了如何使用它们。下一章将深入讨论预处理，包括Spark的特征工程和数据清理工具。然后，我们将详细介绍MLlib中可用的每种算法，以及一些用于图形分析和深度学习的工具。
