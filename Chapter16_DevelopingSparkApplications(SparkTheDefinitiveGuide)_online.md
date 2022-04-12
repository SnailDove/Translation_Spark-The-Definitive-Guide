---
title: 翻译 Chapter 16 Developing Spark Applications
date: 2019-08-05
copyright: true
categories: English
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 16 Developing Spark Applications

## <font color="#9a161a" >Testing Spark Applications 测试Spark应用程序</font>

You now know what it takes to write and run a Spark Application, so let’s move on to a less exciting but still very important topic: testing. Testing Spark Applications relies on a couple of key principles and tactics that you should keep in mind as you’re writing your applications.

您现在知道编写和运行Spark应用程序需要什么，所以让我们继续讨论一个不太令人兴奋但仍然非常重要的主题：测试。测试Spark应用程序依赖于在编写应用程序时应该记住的几个关键原则和策略。

### <font color="#000000" >Strategic Principles 战略原则</font>
Testing your data pipelines and Spark Applications is just as important as actually writing them. This is because you want to ensure that they are resilient to future change, in data, logic, and output. In this section, we’ll first discuss what you might want to test in a typical Spark Application, then discuss *how* to organize your code for easy testing.

测试数据管道和 Spark 应用程序与实际编写它们一样重要。这是因为您希望确保它们能够适应未来的变化，包括数据，逻辑和输出。在本节中，我们将首先讨论您可能希望在典型的 Spark 应用程序中测试的内容，然后讨论如何组织代码以便于轻松测试。

#### <font color="#3399cc">Input data resilience 输入数据弹性</font>
Being resilient to different kinds of input data is something that is quite fundamental to how you write your data pipelines. The data will change because the business needs will change. Therefore your Spark Applications and pipelines should be resilient to at least some degree of change in the input data or otherwise ensure that these failures are handled in a graceful and resilient way. For the most part this means being smart about writing your tests to handle those edge cases of different inputs and making sure that the pager  only goes off when it’s something that is truly important.

对不同类型的输入数据具有弹性对于编写数据管道非常重要。数据将发生变化，因为业务需求将发生变化。因此，您的Spark应用程序和管道应该能够适应输入数据的至少某种程度的变化，或者确保以优雅和弹性的方式处理这些故障。在大多数情况下，这意味着要聪明地编写测试来处理不同输入的边缘情况（极端情况），并确保报警器仅在真正重要的事情发生时才会发出警报。

#### <font color="#3399cc">Business logic resilience and evolution 业务逻辑弹性和演变</font>

The business logic in your pipelines will likely change as well as the input data. Even more importantly, you want to be sure that what you’re deducing from the raw data is what you actually think that you’re deducing. This means that you’ll need to do robust logical testing with realistic data to ensure that you’re actually getting what you want out of it. One thing to be wary of here is trying to write a bunch of “Spark Unit Tests” that just test Spark’s functionality. You don’t want to be doing that; instead, you want to be testing your business logic and ensuring that the complex business pipeline that you set up is actually doing what you think it should be doing.

管道中的业务逻辑可能会随输入数据而变化。更重要的是，您希望确保从原始数据中推断出的内容是您实际认为正在推导的内容。这意味着您需要使用真实数据进行强大的逻辑测试，以确保您实际上从中获得了您想要的内容。需要注意的一件事是尝试编写一组只测试Spark功能的“Spark Unit Tests”。你不想这样做;相反，您希望测试业务逻辑并确保您设置的复杂业务管道实际上正在执行您认为应该执行的操作。

#### <font color="#3399cc">Resilience in output and atomicity 输出的弹性和原子性</font>

Assuming that you’re prepared for departures in the structure of input data and that your business logic is well tested, you now want to ensure that your output structure is what you expect. This means you will need to gracefully handle output schema resolution. It’s not often that data is simply dumped in some location, never to be read again—most of your Spark pipelines are probably feeding other Spark pipelines. For this reason you’re going to want to make certain that your downstream consumers understand the “state” of the data—this could mean how frequently it’s updated as well as whether the data is “complete” (e.g., there is no late data) or that there won’t be any last-minute corrections to the data. All of the aforementioned issues are principles that you should be thinking about as you build your data pipelines (actually, regardless of whether you’re using Spark). This strategic thinking is important for laying down the foundation for the system that you would like to build.

假设您已准备好在输入数据结构中离开，并且您的业务逻辑已经过充分测试，那么您现在需要确保您的输出结构符合您的预期。这意味着您需要优雅地处理输出模式解析。通常不会将数据简单地转储到某个位置，永远不会再次读取——大多数Spark管道可能正在为其他 Spark 管道提供服务。因此，您需要确保下游消费者了解数据的“状态”——这可能意味着更新频率以及数据是否“完整”（例如，没有后期数据）或者不会对数据进行任何最后修正。所有上述问题都是您在构建数据管道时应该考虑的原则（实际上，无论您是否使用Spark）。这种战略思维对于为您希望构建的系统奠定基础非常重要。

### <font color="#000000" >Tactical Takeaways 战术外卖</font>
Although strategic thinking is important, let’s talk a bit more in detail about some of the tactics that you can actually use to make your application easy to test. The highest value approach is to verify that your business logic is correct by employing proper unit testing and to ensure that you’re resilient to changing input data or have structured it so that schema evolution will not become unwielding  in the future. The decision for how to do this largely falls on you as the developer because it will vary according to your business domain and domain expertise.

虽然战略思维很重要，但让我们更详细地谈谈您可以实际使用的一些策略，以使您的应用程序易于测试。最有价值的方法是通过采用适当的单元测试来验证您的业务逻辑是否正确，并确保您能够适应不断变化的输入数据，或者对其进行结构化，以便模式演变在未来不会变得无法使用。关于如何做到这一点的决定很大程度上取决于您作为开发人员，因为它将根据您的业务领域和领域专业知识而有所不同。

#### <font color="#3399cc">Managing SparkSessions 管理SparkSessions</font>
Testing your Spark code using a unit test framework like `JUnit ` or `ScalaTest` is relatively easy because of Spark’s local mode—just create a local mode `SparkSession` as part of your test harness to run it. However, to make this work well, you should try to perform dependency injection as much as possible when managing SparkSessions in your code. That is, initialize the `SparkSession` only once and pass it around to relevant functions and classes at runtime in a way that makes it easy to substitute during testing. This makes it much easier to test each individual function with a dummy `SparkSession` in unit tests.

使用单元测试框架（如 JUnit 或 ScalaTest ）测试Spark代码相对容易，因为Spark的本地模式——只需创建一个本地模式 SparkSession 作为测试工具的一部分来运行它。但是，为了使这项工作更好，您应该在代码中管理SparkSessions 时尽可能多地执行依赖项注入。也就是说，只将 SparkSession 初始化一次并在运行时将其传递给相关的函数和类，以便在测试期间轻松替换。这使得在单元测试中使用仿真的SparkSession测试每个单独的函数变得更加容易。

#### <font color="#3399cc">Which Spark API to Use? 使用哪种Spark API？</font>
Spark offers several choices of APIs, ranging from SQL to DataFrames and Datasets, and each of these can have different impacts for maintainability   and testability    of your application. To be perfectly honest, the right API depends on your team and its needs: some teams and projects will need the less strict SQL and DataFrame APIs for speed of development, while others will want to use type-safe Datasets or RDDs.

Spark提供了多种API选择，从 SQL 到 DataFrames 和 Datasets，每种API都会对应用程序的可维护性和可测试性产生不同的影响。说实话，正确的API取决于您的团队及其需求：一些团队和项目需要不太严格的SQL和DataFrame API来提高开发速度，而其他团队和项目则需要使用类型安全的 DataSet 或 RDD 。


In general, we recommend documenting and testing the input and output types of each function regardless of which API you use. The type-safe API automatically enforces a minimal contract for your function that makes it easy for other code to build on it. If your team prefers to use DataFrames or SQL, then spend some time to document and test what each function returns and what types of inputs it accepts to avoid surprises later, as in any dynamically typed programming language. While the lower-level RDD API is also statically typed, we recommend going into it only if you need low-level features such as partitioning that are not present in Datasets, which should not be very common; the Dataset API allows more performance optimizations and is likely to provide even more of them in the future.

通常，我们建议记录和测试每个函数的输入和输出类型，无论您使用哪个API。类型安全的API会自动为您的函数强制执行最小的约定（可以理解为：可使用的条款少，灵活度低），以便在其上构建其他代码。如果您的团队更喜欢使用DataFrames或SQL，那么花一些时间来记录和测试每个函数返回的内容以及它接受哪些类型的输入以避免以后出现意外，就像在任何动态类型编程语言中一样。虽然较低阶的RDD API也是静态类型的，但我们建议只有在需要低阶功能（例如数据集中不存在的分区）时才进入它，这应该不常见; DataSet API允许更多性能优化，并且可能在将来提供更多性能优化。

A similar set of considerations applies to which programming language to use for your application: there certainly is no right answer for every team, but depending on your needs, each language will provide different benefits. We generally recommend using statically typed languages like Scala and Java for larger applications or those where you want to be able to drop into low-level code to fully control performance, but Python and R may be significantly better in other cases—for example, if you need to use some of their other libraries. Spark code should easily be testable in the standard unit testing frameworks in every language. 

一组类似的注意事项适用于您的应用程序使用哪种编程语言：每个团队肯定没有正确的答案，但根据您的需求，每种语言都会提供不同的好处。我们通常建议对大型应用程序使用静态类型语言（如Scala和Java），或者希望能够放入低阶代码以完全控制性能的语言，但在其他情况下，Python和R可能会明显更好——例如，如果你需要使用他们的一些其他库。 Spark代码应该可以在每种语言的标准单元测试框架中轻松测试。

### <font color="#000000" >Connecting to Unit Testing Frameworks 连接到单元测试框架</font>
To unit test your code, we recommend using the standard frameworks in your langage (e.g., `JUnit` or `ScalaTest` ), and setting up your test harnesses to create and clean up a `SparkSession` for each test. Different frameworks offer different mechanisms to do this, such as “before” and “after” methods. We have included some sample unit testing code in the application templates for this chapter.

要对代码进行单元测试，我们建议您使用标准框架（例如，`JUnit` 或 `ScalaTest`），并设置测试工具来为每个测试创建和清理 `SparkSession`。不同的框架提供了不同的机制来实现这一点，例如“之前”和“之后”方法。我们在本章的应用程序模板中包含了一些示例单元测试代码。

### <font color="#000000" >Connecting to Data Sources 连接到数据源</font>
As much as possible, you should make sure your testing code does not connect to production data sources, so that developers can easily run it in isolation if these data sources change. One easy way to make this happen is to have all your business logic functions take DataFrames or Datasets as input instead of directly connecting to various sources; after all, subsequent code will work the same way no matter what the data source was. If you are using the structured APIs in Spark, another way to make this happen is named tables: you can simply register some dummy datasets (e.g., loaded from small text file or from in-memory objects) as various table names and go from there. 

您应尽可能确保测试代码不会连接到生产数据源，以便开发人员可以在这些数据源发生更改时轻松地单独运行它。实现这一目标的一种简单方法是让所有业务逻辑功能将DataFrames或Datasets作为输入，而不是直接连接到各种源; 毕竟，无论数据源是什么，后续代码都将以相同的方式工作。如果您在Spark中使用结构化API，另一种实现此目的的方法是命名表：您只需将一些仿真数据集（例如，从小文本文件或内存中对象加载）注册为各种表名，然后从那里开始。

## <font color="#9a161a" >The Development Process 开发过程</font>
The development process with Spark Applications is similar to development workflows that you have probably already used. First, you might maintain a scratch space, such as an interactive notebook or some equivalent thereof, and then as you build key components and algorithms, you move them to a more permanent location like a library or package. The notebook experience is one that we often recommend (and are using to write this book) because of its simplicity in experimentation. There are also some tools, such as Databricks, that allow you to run notebooks as production applications as well.

Spark应用程序的开发过程类似于您可能已经使用过的开发工作流程。首先，您可以维护一个临时空间，例如交互式笔记本或其等效物，然后在构建关键组件和算法时，将它们移动到更永久的位置，如库或包。notebook（与jupter notebook类似）的体验是我们经常推荐的（并且正在用来编写本书），因为它的实验非常简单。还有一些工具，如 Databricks，允许您将笔记本作为生产应用程序运行。

When running on your local machine, the <font face="constant-width" color="#000000" size=3>spark-shell</font> and its various language-specific implementations are probably the best way to develop applications. For the most part, the shell is for interactive applications, whereas <font face="constant-width" color="#000000" size=3>spark-submit</font> is for production applications on your Spark cluster. You can use the shell to interactively run Spark, just as we showed you at the beginning of this book. This is the mode with which you will run <font face="constant-width" color="#000000" size=3>PySpark</font>, <font face="constant-width" color="#000000" size=3>Spark SQL</font>, and <font face="constant-width" color="#000000" size=3>SparkR</font>. In the bin folder, when you download Spark, you will find the various ways of starting these shells. Simply run <font face="constant-width" color="#000000" size=3>sparkshell</font>(for Scala), <font face="constant-width" color="#000000" size=3>spark-sql</font>, <font face="constant-width" color="#000000" size=3>pyspark</font>, and <font face="constant-width" color="#000000" size=3>sparkR</font>. After you’ve finished your application and created a package or script to run, spark-submit will become your best friend to submit this job to a cluster. 

在本地计算机上运行时，spark-shell 及其各种特定于语言的实现可能是开发应用程序的最佳方式。在大多数情况下，shell用于交互式应用程序，而 spark-submit 用于Spark集群上的生产应用程序。您可以使用shell以交互方式运行Spark，就像我们在本书开头部分向您展示的那样。这是运行 PySpark，Spark SQL 和 SparkR 的模式。在bin文件夹中，当您下载 Spark 时，您将找到启动这些 shell 的各种方法。只需运行 sparkshell（适用于Scala），spark-sql，pyspark 和 sparkR。在您完成应用程序并创建要运行的包或脚本后，spark-submit 将成为您将此作业提交到群集的最佳朋友。

## <font color="#9a161a" >Launching Applications 启动应用程序</font>

The most common way for running Spark Applications is through <font face="constant-width" color="#000000" size=3>spark-submit</font>. Previously in this chapter, we showed you how to run <font face="constant-width" color="#000000" size=3>spark-submit</font>; you simply specify your options, the application <font face="constant-width" color="#000000" size=3>JAR</font> or script, and the relevant arguments: 

运行Spark应用程序的最常用方法是通过spark-submit。 在本章的前面，我们向您展示了如何运行spark-submit; 您只需指定选项，应用程序 JAR 或脚本以及相关参数：

```shell
./bin/spark-submit \
--class <main-class> \
--master <master-url> \
--deploy-mode <deploy-mode> \
--conf <key>=<value> \
... # other options
<application-jar-or-script> \
[application-arguments]
```

You can always specify whether to run in client or cluster mode when you submit a Spark job with 
<font face="constant-width" color="#000000" size=3>spark-submit</font>. However, you should almost always favor running in cluster mode (or in client mode on the cluster itself) to reduce latency between the executors and the driver.

当您使用 spark-submit 提交 Spark 作业时，您始终可以指定是以客户端还是群集模式运行。 但是，您几乎总是倾向于在群集模式下运行（或在集群本身的客户端模式下）以减少执行程序和驱动程序之间的延迟。

When submitting applciations, pass a <font face="constant-width" color="#000000" size=3>.py</font> file in the place of a <font face="constant-width" color="#000000" size=3>.jar</font>, and add Python <font face="constant-width" color="#000000" size=3>.zip</font>, <font face="constant-width" color="#000000" size=3>.egg</font>, or <font face="constant-width" color="#000000" size=3>.py</font> to the search path with <font face="constant-width" color="#000000" size=3>--py-files</font>. 

提交 applciations 时，在 .jar 的位置传递 .py 文件，并使用 --py-files 将 Python .zip，.egg 或 .py 添加到搜索路径。

For reference, Table 16-1 lists all of the available <font face="constant-width" color="#000000" size=3>spark-submit</font> options, including those that are particular to some cluster managers. To enumerate all these options yourself, run <font face="constant-width" color="#000000" size=3>spark-submit</font> with <font face="constant-width" color="#000000" size=3>--help</font>. 

作为参考，表16-1列出了所有可用的 spark-submit 选项，包括某些集群管理器特有的选项。 要自己枚举所有这些选项，请使用 --help 运行 spark-submit。

Table 16-1. [Spark submit help text](http://spark.apache.org/docs/latest/submitting-applications.html) 

| Parameter                    | Description                                                  |
| ---------------------------- | ------------------------------------------------------------ |
| --master<br/>MASTER_URL      | spark://host:port, mesos://host:port, yarn, or local<br />Spark 连接的资源管理器 |
| --deploymode<br/>DEPLOY_MODE | Whether to launch the driver program locally (“client”) or on one of the worker machines inside the cluster (“cluster”) (Default: client)<br />是在本地（“client”）还是在集群内（“cluster”）的某个工作机器上启动驱动程序（默认值：client） |
| --class<br/>CLASS_NAME       | Your application’s main class (for Java / Scala apps).<br />您的应用程序的主类（适用于Java / Scala应用程序）。 |
| --name NAME                  | A name of your application.<br />你的应用程序主类。          |
| --jars JARS                  | Comma-separated list of local JARs to include on the driver and executor classpaths.<br />以逗号分隔的本地 JAR 列表，包含在驱动程序和执行程序类路径中。 |
| --packages                   | search the local Maven repo, then Maven Central and any additional remote repositories given by --<font face="constant-width" color="#000000" size=3>repositories</font>. The format for the coordinates should be <font face="constant-width" color="#000000" size=3>groupId:artifactId:version</font>.<br />搜索本地 Maven 仓库，然后搜索 Maven Central 以及 `--repositories` 给出的任何其他远程仓库。 坐标的格式应为 `groupId:artifactId:version`。 |
| --exclude-packages           | Comma-separated list of <font face="constant-width" color="#000000" size=3>groupId:artifactId</font>, to exclude while resolving the dependencies provided in --<font face="constant-width" color="#000000" size=3>packages</font> to avoid dependency conflicts.<br />逗号分隔的 `groupId:artifactId` 列表，在解析 `--packages` 中提供的依赖项时排除，以避免依赖性冲突。 |
| --repositories               | Comma-separated list of additional remote repositories to search for the Maven coordinates given with --<font face="constant-width" color="#000000" size=3>packages</font>.<br />以逗号分隔的其他远程仓库列表，用于搜索 `--packages` 给出的Maven坐标 |
| --py-files<br/>PY_FILES      | Comma-separated list of *.zip*, *.egg*, or *.py* files to place on the <font face="constant-width" color="#000000" size=3>PYTHONPATH</font> for Python apps.<br />以逗号分隔的 .zip，.egg 或 .py 文件列表，放在 Python 应用程序的 PYTHONPATH 上。 |
| --files<br/>FILES            | Comma-separated list of files to be placed in the working directory of each executor.<br />以逗号分隔的文件列表，放在每个执行程序的工作目录中。 |
| --conf<br/>PROP=VALUE        | Arbitrary Spark configuration property.<br />任意Spark配置属性 |
| --<br/>propertiesfile FILE   | Path to a file from which to load extra properties. If not specified, this will look for <font face="constant-width" color="#000000" size=3>conf/spark-defaults.conf</font>.<br />从中加载额外属性的文件的路径。 如果未指定，则会查找 `conf/spark-defaults.conf`。 |
| --driver-memory MEM          | Memory for driver (e.g., 1000M, 2G) (Default: 1024M).<br />驱动器的内存（默认：1024M）。 |
| --driver-java-options        | Extra Java options to pass to the driver.<br />传递给驱动器的额外 Java 选项。 |
| --driver-library-path        | Extra library path entries to pass to the driver.<br />要传递给驱动程序的额外库路径条目。 |
| --driver-class-path          | Extra class path entries to pass to the driver. Note that JARs added with --<font face="constant-width" color="#000000" size=3>jars</font> are automatically included in the classpath.<br />要传递给驱动程序的额外类路径条目。请注意，添加了 `--jars` 的 JAR 会自动包含在类路径中。 |
| --executor-memory MEM        | Memory per executor (e.g., 1000M, 2G) (Default: 1G).<br />每个执行器的内存（例如，1000M，2G）（默认值：1G）。 |
| --proxy-user NAME            | User to impersonate when submitting the application. This argument does not work with --<font face="constant-width" color="#000000" size=3>principal</font> / <font face="constant-width" color="#000000" size=3>keytab</font>.<br />用户在提交申请时进行冒充。此参数不适用于 `--principal/keytab`。 |
| --help, -h                   | Show this help message and exit.<br />显示此帮助消息并退出。 |
| --verbose, -v                | Print additional debug output.<br />打印其他调试输出。       |
| --version                    | Print the version of current Spark.<br />打印当前Spark的版本。 |


There are some deployment-specific configurations as well (see Table 16-2).

还有一些特定于部署的配置（参见表16-2）。

| Cluster Managers | Modes   | Conf                        | Description                                                  |
| ---------------- | ------- | --------------------------- | ------------------------------------------------------------ |
| Standalone       | Cluster | --driver-cores NUM          | Cores for driver (Default: 1)<br />驱动核心（默认值：1）     |
| Standalone/Mesos | Cluster | --supervise                 | If given, restarts the driver on failure.<br />如果给定，则在失败时重新启动驱动程序。 |
| Standalone/Mesos | Cluster | --kill<br/>SUBMISSION_ID    | If given, kills the driver specified.<br />如果给定，则杀死指定的驱动程序。 |
| Standalone/Mesos | Cluster | --status<br/>SUBMISSION_ID  | If given, requests the status of the driver specified.<br />如果给定，请求指定的驱动程序的状态。 |
| Standalone/Mesos | Either  | --total-executor-cores NUM  | Total cores for all executors.<br />所有执行器（executor）的核心总数。 |
| Standalone/YARN  | Either  | --total-executor-cores NUM1 | Number of cores per executor. (Default: 1 in YARN mode or all available cores on the worker in standalone mode)<br />每个执行器（executor）的核心数。 （默认值：YARN模式下为1或独立模式下工作线程上的所有可用核心） |
| YARN             | Either  | --driver-cores NUM          | Number of cores used by the driver, only in cluster mode (Default: 1).<br />驱动程序使用的核心数，仅在集群模式下（默认值：1）。 |
| YARN             | Either  | queue QUEUE_NAME            | The YARN queue to submit to (Default: “default”).<br />要提交的YARN队列（默认值：“默认”）。 |
| YARN             | Either  | --num-executors NUM         | Number of executors to launch (Default: 2). If dynamic allocation is enabled, the initial number of executors will be at least NUM.<br />要启动的执行程序数（默认值：2）。如果启用了动态分配，则执行程序的初始数量将至少为NUM。 |
| YARN             | Either  | --archives<br/>ARCHIVES     | Comma-separated list of archives to be extracted into the working directory of each executor.<br />以逗号分隔的档案列表，提取到每个执行程序的工作目录中。 |
| YARN             | Either  | --principal <br/>PRINCIPAL  | Principal to be used to log in to KDC, while running on secure HDFS.<br />Principal 用于在安全 HDFS 上运行时登录 KDC。 |
| YARN             | Either  | --keytab<br/>KEYTAB         | The full path to the file that contains the keytab for the principal specified above. This keytab will be copied to the node running the Application Master via the Secure Distributed Cache, for renewing the login tickets and the delegation tokens periodically.<br />包含上面指定的主体的keytab的文件的完整路径。此密钥表将通过安全分布式缓存复制到运行Application Master的节点，以定期更新登录票证和委派令牌。 |

[最新的 Spark 应用程序提交帮助文档](http://spark.apache.org/docs/latest/submitting-applications.html) 

### <font color="#000000" >Application Launch Examples</font>
We already covered some local-mode application examples previously in this chapter, but it’s worth looking at how we use some of the aforementioned options, as well. Spark also includes several examples and demonstration applications in the *examples* directory that is included when you download Spark. If you’re stuck on how to use certain parameters, simply try them first on your local machine and use the SparkPi class as the main class:

我们已经介绍了本章前面的一些本地模式应用程序示例，但是值得一看的是我们如何使用上述一些选项。 Spark还包含下载Spark时包含的示例目录中的几个示例和演示应用程序。 如果你坚持使用某些参数，只需先在本地机器上尝试它们，然后使用SparkPi类作为主类：

```scala 
./bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://207.184.161.138:7077 \
--executor-memory 20G \
--total-executor-cores 100 \
replace/with/path/to/examples.jar \
1000
```

The following snippet does the same for Python. You run it from the Spark directory and this will allow you to submit a Python application (all in one script) to the standalone cluster manager. You can also set the same executor limits as in the preceding example:

以下代码段对Python也是如此。 您可以从Spark目录运行它，这将允许您将Python应用程序（所有在一个脚本中）提交给独立的集群管理器。 您还可以设置与前面示例中相同的执行器（executor）限制：

```scala
./bin/spark-submit \
--master spark://207.184.161.138:7077 \
examples/src/main/python/pi.py \
1000
```

You can change this to run in local mode as well by setting the master to <font face="constant-width" color="#000000" size=3>local</font> or <font face="constant-width" color="#000000" size=3>local[*]</font> to run on all the cores on your machine. You will also need to change the <font face="constant-width" color="#000000" size=3>/path/to/examples.jar</font> to the relevant Scala and Spark versions you are running.

您可以将此更改为在本地模式下运行，方法是将主服务器设置为 local  或 local[*] 以在计算机上的所有核心上运行。 您还需要将 /path/to/examples.jar 更改为您正在运行的相关Scala和Spark版本。

## <font color="#9a161a" >Configuring Applications 配置应用程序</font>
Spark includes a number of different configurations, some of which we covered in Chapter 15. There are many different configurations, depending on what you’re hoping to achieve. This section covers those very details. For the most part, this information is included for reference and is probably worth skimming only, unless you’re looking for something in particular. The majority of configurations fall into the following categories:

Spark包含许多不同的配置，其中一些我们在第15章中介绍过。根据您希望实现的目标，有许多不同的配置。本节介绍了这些细节。在大多数情况下，这些信息仅供参考，可能仅值得略读，除非您特别寻找某些内容。大多数配置分为以下几类：

- Application properties 应用属性

- Runtime environment 运行环境

- Shuffle behavior 数据再分配行为

- Spark UI

- Compression and serialization 解压缩

- Memory management 内存管理

- Execution behavior 执行行为

- Networking 网络

- Scheduling 调度

- Dynamic allocation 动态分配

- Security 安全

- Encryption 加密

- Spark SQL

- Spark streaming Spark流

- SparkR

Spark provides three locations to configure the system:

Spark提供三个位置来配置系统：

- Spark properties control most application parameters and can be set by using a <font face="constant-width" color="#000000" size=3>SparkConf</font> object Spark

    属性控制大多数应用程序参数，可以使用 SparkConf 对象进行设置

- Java system properties 

    Java系统属性

- Hardcoded configuration files

    硬编码配置文件

There are several templates that you can use, which you can find in the `/conf` directory available in the root of the Spark home folder. You can set these properties as hardcoded variables in your applications or by specifying them at runtime. You can use environment variables to set per-machine settings, such as the IP address, through the `conf/spark-env.sh` script on each node. Lastly, you can configure logging through <font face="constant-width" color="#000000" size=3>log4j.properties</font>.

您可以使用几个模板，您可以在Spark主文件夹的根目录中的 `/conf` 目录中找到这些模板。您可以在应用程序中将这些属性设置为硬编码变量，也可以在运行时指定它们。您可以使用环境变量通过每个节点上的 `conf/spark-env.sh` 脚本设置每台计算机设置，例如IP地址。最后，您可以通过 log4j.properties 配置日志记录。

### <font color="#000000">The SparkConf</font>
The <font face="constant-width" color="#000000" size=3>SparkConf</font> manages all of our application configurations. You create one via the import statement, as shown in the example that follows. After you create it, the <font face="constant-width" color="#000000" size=3>SparkConf</font> is immutable  for that specific Spark Application: 

SparkConf 管理我们的所有应用程序配置。您可以通过 import 语句创建一个，如下面的示例所示。创建它之后，SparkConf 对于特定的Spark应用程序是不可变的：

```scala
// in Scala
import org.apache.spark.SparkConf
val conf = new SparkConf().setMaster("local[2]").setAppName("DefinitiveGuide")
.set("some.conf", "to.some.value")
```

```python
# in Python
from pyspark import SparkConf
conf = SparkConf().setMaster("local[2]").setAppName("DefinitiveGuide")\
.set("some.conf", "to.some.value")
```

You use the <font face="constant-width" color="#000000" size=3>SparkConf</font> to configure individual Spark Applications with Spark properties. These Spark properties control how the Spark Application runs and how the cluster is configured. The example that follows configures the local cluster to have two threads and specifies the application name that shows up in the Spark UI.

您可以使用 SparkConf 使用 Spark 属性配置各个 Spark 应用程序。这些 Spark 属性控制 Spark 应用程序的运行方式以及集群的配置方式。以下示例将本地集群配置为具有两个线程，并指定在 Spark UI 中显示的应用程序名称。

You can configure these at runtime, as you saw previously in this chapter through command-line arguments. This is helpful when starting a Spark Shell that will automatically include a basic Spark Application for you; for instance:

您可以在运行时配置它们，如本章前面通过命令行参数所见。这在启动Spark Shell时非常有用，它将自动包含一个基本的Spark应用程序；例如：

```shell
./bin/spark-submit --name "DefinitiveGuide" --master local[4] ...
```

Of note is that when setting time duration-based properties, you should use the following format:

值得注意的是，在设置基于持续时间的属性时，您应该使用以下格式：

- 25ms (milliseconds 毫秒)
- 5s (seconds 秒)
- 10m or 10min (minutes 分钟)
- 3h (hours 小时)
- 5d (days 天)
- 1y (years 年)

### <font color="#000000" >Application Properties 应用属性</font>

Application properties are those that you set either from spark-submit or when you create your Spark Application. They define basic application metadata as well as some execution characteristics. Table 16-3 presents a list of current application properties. 

应用程序属性是您通过 spark-submit 或创建 Spark 应用程序时设置的属性。它们定义了基本的应用程序元数据以及一些执行特性。表 16-3 列出了当前的应用程序属性。

Table 16-3. [Application properties](http://spark.apache.org/docs/latest/configuration.html#application-properties)

| Property name <br />属性名 | Default<br />默认值 | Meaning 意思                                                 |
| -------------------------- | ------------------- | ------------------------------------------------------------ |
| spark.app.name             | (none)              | The name of your application. This will appear in the UI and in log data.<br />您的应用程序的名称。 这将显示在UI和日志数据中。 |
| spark.driver.cores         | 1                   | Number of cores to use for the driver process, only in cluster mode.<br />仅在集群模式下用于驱动程序进程的核心数。 |
| spark.driver.maxResultSize | 1g                  | Limit of total size of serialized results of all partitions for each Spark action (e.g., collect). Should be at least 1M, or 0 for unlimited. Jobs will be aborted if the total size exceeds this limit. Having a high limit can cause OutOfMemoryErrors in the driver (depends on spark.driver.memory and memory overhead of objects in JVM). Setting a proper limit can protect the driver from OutOfMemoryErrors.<br />每个Spark操作的所有分区的序列化结果的总大小限制（例如，collect）。 应至少为1M，或0为无限制。 如果总大小超过此限制，则将中止作业。 具有上限可能会导致驱动程序中的OutOfMemoryErrors（取决于spark.driver.memory和JVM中对象的内存开销）。 设置适当的限制可以保护驱动程序免受OutOfMemoryErrors的影响。 |
| spark.driver.memory        | 1g                  | Amount of memory to use for the driver process, where SparkContext is initialized. (e.g. 1g, 2g). Note: in client mode, this must not be set through the SparkConf directly in your application, because the driver JVM has already started at that point. Instead, set this through the --driver-memory command-line option or in your default properties file.<br />用于初始化 SparkContext 的驱动程序进程的内存量。 （例如1g，2g）。 注意：在客户端模式下，不能直接在应用程序中通过 SparkConf 设置，因为驱动程序JVM已在此时启动。 而是通过--driver-memory命令行选项或在默认属性文件中设置它。 |
| spark.executor.memory      | 1g                  | Amount of memory to use per executor process (e.g., 2g, 8g).<br />每个执行程序进程使用的内存量（例如，2g，8g）。 |
| spark.extraListeners       | (none)              | A comma-separated list of classes that implement SparkListener; when initializing SparkContext, instances of these classes will be created and registered with Spark’s listener bus. If a class has a single-argument constructor that accepts a SparkConf, that constructor will be called; otherwise, a zero-argument constructor will be called. If no valid constructor can be found, the SparkContext creation will fail with an exception.<br />以逗号分隔的实现SparkListener的类列表; 在初始化SparkContext时，将创建这些类的实例并使用Spark的侦听器总线进行注册。 如果一个类有一个接受SparkConf的单参数构造函数，那么将调用该构造函数; 否则，将调用零参数构造函数。 如果找不到有效的构造函数，SparkContext创建将失败并出现异常。 |
| spark.logConf              | (false)             | Logs the effective SparkConf as INFO when a SparkContext is started.<br />启动 SparkContext 时，将有效的 SparkConf 记录为INFO。 |
| spark.master               | (none)              | The cluster manager to connect to. See the list of allowed master URLs.<br />要连接的集群管理器。 请参阅允许的主URL列表。 |
| spark.submit.deployMode    | (none)              | The deploy mode of the Spark driver program, either “client” or “cluster,” which means to launch driver program locally (“client”) or remotely (“cluster”) on one of the nodes inside the cluster.<br />Spark驱动程序的部署模式，“客户端”或“集群”，这意味着在集群内的一个节点上本地（“客户端”）或远程（“集群”）启动驱动程序。 |
| spark.log.callerContext    | (none)              | Application information that will be written into Yarn RM log/HDFS audit log when running on Yarn/HDFS. Its length depends on the Hadoop configuration hadoop.caller.context.max.size. It should be concise, and typically can have up to 50 characters.<br />在 Yarn/HDFS 上运行时将写入Yarn RM log / HDFS 审核日志的应用程序信息。 它的长度取决于Hadoop配置hadoop.caller.context.max.size。 它应该简洁，通常最多可包含50个字符。 |
| spark.driver.supervise     | (false)             | If true, restarts the driver automatically if it fails with a non-zero exit status. Only has effect in Spark standalone mode or Mesos cluster deploy mode.<br />如果为true，则在失败且退出状态为非零时自动重新启动驱动程序。 仅在Spark独立模式或Mesos集群部署模式下有效。 |

You can ensure that you’ve correctly set these values by checking the application’s web UI on port 4040 of the driver on the “Environment” tab. Only values explicitly specified through sparkdefaults.conf, SparkConf, or the command line will appear. For all other configuration properties, you can assume the default value is used.

您可以通过在“环境”选项卡上检查驱动程序的端口4040上的应用程序的Web UI来确保您正确设置了这些值。仅显示那些通过 sparkdefaults.conf，SparkConf或命令行显式指定的值。对于所有其他配置属性，您可以假设使用默认值。

### <font color="#000000" >Runtime Properties 运行时属性</font>
Although less common, there are times when you might also need to configure the runtime environment of your application. Due to space limitations, we cannot include the entire configuration set here. Refer to the relevant table on [the Runtime Environment](https://bit.ly/2FlsX2i) in [the Spark documentation](https://bit.ly/1qnQ26w). These properties allow you to configure extra classpaths and python paths for both drivers and executors, Python worker configurations, as well as miscellaneous  logging properties.

虽然不太常见，但有时您可能还需要配置应用程序的运行时环境。由于篇幅限制，我们无法在此处包含整个配置集。请参阅[Spark文档](https://bit.ly/1qnQ26w)中的 [Runtime Environment](https://bit.ly/1qnQ26w) 上的相关表。这些属性允许您对驱动程序（driver）和执行器（excutor），Python工作程序进行配置以及各种日志记录属性配置额外的类路径和python路径。

### <font color="#000000" >Execution Properties 执行属性</font>
These configurations are some of the most relevant for you to configure because they give you finer-grained control on actual execution. Due to space limitations, we cannot include the entire configuration set here. Refer to the relevant table on [Execution Behavior](http://spark.apache.org/docs/latest/configuration.html#execution-behavior) in [the Spark documentation](http://spark.apache.org/docs/latest/index.html). The most common configurations to change are <font face="constant-width" color="#000000" size=3>spark.executor.cores</font> (to control the number of available cores) and <font face="constant-width" color="#000000" size=3>spark.files.maxPartitionBytes</font> (maximum partition size when reading files).

这些配置与您配置最相关，因为它们可以为您提供对实际执行的精细控制。由于篇幅限制，我们无法在此处包含整个配置集。请参阅[Spark文档](http://spark.apache.org/docs/latest/index.html)中的[执行行为相关表](http://spark.apache.org/docs/latest/configuration.html#execution-behavior)。要更改的最常见配置是 spark.executor.cores（用于控制可用内核的数量）和 spark.files.maxPartitionBytes（读取文件时的最大分区大小）。

### <font color="#000000" >Configuring Memory Management 配置内存管理</font>
There are times when you might need to manually manage the memory options to try and optimize your applications. Many of these are not particularly relevant for end users because they involve a lot of legacy concepts or fine-grained controls that were obviated in Spark 2.X because of automatic memory management. Due to space limitations, we cannot include the entire configuration set here. Refer to the relevant table on [Memory Management](http://spark.apache.org/docs/latest/configuration.html#memory-management) in [the Spark documentation](http://spark.apache.org/docs/latest/index.html).

有时您可能需要手动管理内存选项以尝试和优化您的应用程序。 其中许多与最终用户并不特别相关，因为它们涉及很多遗留概念或由于自动内存管理而在Spark 2.X中避免的细粒度控制。 由于篇幅限制，我们无法在此处包含整个配置集。 请参阅[Spark文档](http://spark.apache.org/docs/latest/index.html)中的[内存管理相关表](http://spark.apache.org/docs/latest/configuration.html#memory-management)。

### <font color="#000000" >Configuring Shuffle Behavior 配置数据再分配行为</font>
We’ve emphasized how shuffles can be a bottleneck in Spark jobs because of their high communication overhead. Therefore there are a number of low-level configurations for controlling shuffle behavior. Due to space limitations, we cannot include the entire configuration set here. Refer to the relevant table on [Shuffle Behavior](http://spark.apache.org/docs/latest/configuration.html#shuffle-behavior) in [the Spark documentation](http://spark.apache.org/docs/latest/index.html).

我们已经强调了数据再分配如何成为Spark工作的瓶颈，因为它们的通信开销很高。因此，存在许多用于控制数据再分配行为的低阶配置。由于篇幅限制，我们无法在此处包含整个配置集。请参阅[Spark文档](http://spark.apache.org/docs/latest/index.html)中有关[数据再分配行为的相关表](http://spark.apache.org/docs/latest/configuration.html#shuffle-behavior)。

### <font color="#000000" >Environmental Variables 环境变量</font>
You can configure certain Spark settings through environment variables, which are read from the *conf/spark-env.sh* script in the directory where Spark is installed (or *conf/spark-env.cmd* on Windows). In Standalone and Mesos modes, this file can give machine-specific information such as hostnames. It is also sourced when running local Spark Applications or submission scripts. Note that *conf/spark-env.sh* does not exist by default when Spark is installed. However, you can copy *conf/spark-env.sh.template* to create it. Be sure to make the copy executable. 

您可以通过环境变量配置某些Spark设置，这些环境变量是从安装Spark的目录中的 conf/spark-env.sh 脚本（或Windows上的 conf/spark-env.cmd）中读取的。 在Standalone和Mesos模式下，此文件可以提供特定于机器的信息，例如主机名。 它还在运行本地Spark应用程序或提交脚本时获取。 请注意，安装Spark时默认情况下不存在conf/spark-env.sh。 但是，您可以复制 conf/spark-env.sh.template 来创建它。 务必使副本可执行。

The following variables can be set in spark-env.sh:

可以在spark-env.sh中设置以下变量：

<font face="constant-width" color="#000000" size=3>JAVA_HOME</font>

​	Location where Java is installed (if it’s not on your default PATH).
​	安装Java的位置（如果它不在您的默认PATH上）。

<font face="constant-width" color="#000000" size=3>PYSPARK_PYTHON</font>

​	Python binary executable to use for PySpark in both driver and workers (default is python2.7 if available; otherwise, python). Property spark.pyspark.python takes precedence if it is set.
​	在驱动程序和工作程序中用于 PySpark 的 Python 二进制可执行文件（如果可用，默认为 python2.7; 否则为python）。 如果设置了属性 spark.pyspark.python，则优先级。

<font face="constant-width" color="#000000" size=3>PYSPARK_DRIVER_PYTHON</font>

​	Python binary executable to use for PySpark in driver only (default is PYSPARK_PYTHON). Property spark.pyspark.driver.python takes precedence if it is set.
​	Python二进制可执行文件仅用于驱动程序中的PySpark（默认为PYSPARK_PYTHON）。 如果设置了属性spark.pyspark.driver.python，则优先级。

<font face="constant-width" color="#000000" size=3>SPARKR_DRIVER_R</font>

​	R binary executable to use for SparkR shell (default is R). Property spark.r.shell.command takes precedence if it is set.
​	用于SparkR shell的R二进制可执行文件（默认为R）。 如果设置了属性spark.r.shell.command优先。

<font face="constant-width" color="#000000" size=3>SPARK_LOCAL_IP</font>

​	IP address of the machine to which to bind.
​	要绑定的计算机的IP地址。

<font face="constant-width" color="#000000" size=3>SPARK_PUBLIC_DNS</font>

​	Hostname your Spark program will advertise to other machines. 
​	您的Spark程序的主机名将通告给其他计算机。

In addition to the variables ust listed, there are also options for setting up the Spark standalone cluster scripts, such as number of cores to use on each machine and maximum memory. Because spark-env.sh is a shell script, you can set some of these programmatically; for example, you might compute SPARK_LOCAL_IP by looking up the IP of a specific network interface.

除了列出的变量之外，还有用于设置Spark独立集群脚本的选项，例如每台计算机上使用的核心数和最大内存。因为spark-env.sh是一个shell脚本，你可以通过编程方式设置其中一些;例如，您可以通过查找特定网络接口的IP来计算SPARK_LOCAL_IP。

---

<center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong><font></center>
When running Spark on YARN in cluster mode, you need to set environment variables by using the `spark.yarn.appMasterEnv.[EnvironmentVariableName]` property in your `conf/spark-defaults.conf` file. Environment variables that are set in spark-env.sh will not be reflected in the YARN Application Master process in cluster mode. See the YARN-related Spark Properties for more information.

在集群模式下在 YARN 上运行Spark时，需要使用 `conf/spark-defaults.conf` 文件中的`spark.yarn.appMasterEnv.[EnvironmentVariableName]` 属性设置环境变量。在 `spark-env.sh` 中设置的环境变量不会在集群模式下反映在 YARN Application Master 进程中。有关更多信息，请参阅与 YARN 相关的 Spark属性。

---

### <font color="#000000" >Job Scheduling Within an Application 应用程序内的作业调度</font>
Within a given Spark Application, multiple parallel jobs can run simultaneously if they were submitted from separate threads. By job, in this section, we mean a Spark action and any tasks that need to run to evaluate that action. Spark’s scheduler is fully thread-safe and supports this use case to enable applications that serve multiple requests (e.g., queries for multiple users). By default, Spark’s scheduler runs jobs in <font face="constant-width" color="#000000" size=4>FIFO</font> fashion. If the jobs at the head of the queue don’t need to use the entire cluster, later jobs can begin to run right away, but if the jobs at the head of the queue are large, later jobs might be delayed significantly.

在给定的 Spark 应用程序中，如果从不同的的线程提交多个并行作业，则它们可以同时运行。按照作业，在本节中，我们指的是 Spark 操作以及需要运行以评估该操作的任何任务。 Spark 的调度程序是完全线程安全的，并支持此用户案例以支持提供多个请求的应用程序（例如，为多个用户进行查询）。默认情况下，Spark 的调度程序以<font face="constant-width" color="#000000" size=4>FIFO</font>方式运行作业。如果队列头部的作业不需要使用整个集群，则以后的作业可以立即开始运行，但如果队列头部的作业很大，则后续作业可能会显着延迟。

It is also possible to configure fair sharing between jobs. Under fair sharing, Spark assigns tasks between jobs in a round-robin fashion so that all jobs get a roughly equal share of cluster resources. This means that short jobs submitted while a long job is running can begin receiving resources right away and still achieve good response times without waiting for the long job to finish. This mode is best for multiuser settings.

也可以在作业之间配置公平共享。在公平共享下，Spark以循环方式在作业之间分配任务，以便所有作业获得大致相等的集群资源份额。这意味着当长期运行的工作正在执行时所提交的短期工作可以立即开始接收资源，并且仍然可以实现良好的响应时间，而无需等待长时间的工作完成。此模式最适合多用户设置。

To enable the fair scheduler, set the `spark.scheduler.mode` property to <font face="constant-width" color="#000000" size=4>FAIR</font> when configuring a <font face="constant-width" color="#000000" size=3>SparkContext</font>.

要启用公平调度器，请在配置 SparkContext 时将 spark.scheduler.mode 属性设置为 <font face="constant-width" color="#000000" size=4>FAIR</font>。

```scala
val conf = new SparkConf().setMaster(...).setAppName(...)
conf.set("spark.scheduler.mode", "FAIR")
val sc = new SparkContext(conf)
```

The fair scheduler also supports grouping jobs into pools, and setting different scheduling options, or weights, for each pool. This can be useful to create a high-priority pool for more important jobs or to group the jobs of each user together and give users equal shares  regardless of how many concurrent jobs they have instead of giving jobs equal shares. This approach is modeled after the Hadoop Fair Scheduler.

公平调度器还支持将作业分组到池中，并为每个池设置不同的调度选项或权重。这对于为更重要的作业创建高优先级池或将每个用户的作业组合在一起并为用户提供相同的份额非常有用，无论他们有多少并发作业而不是给予作业相等的份额。此方法模拟Hadoop Fair Scheduler。

Without any intervention, newly submitted jobs go into a default pool, but jobs pools can be set by adding the <font face="constant-width" color="#000000" size=3>spark.scheduler.pool</font> local property to the SparkContext in the thread that’s submitting them. This is done as follows (assuming sc is your SparkContext )： 

在没有任何干预的情况下，新提交的作业将进入默认池，但可以通过将 `spark.scheduler.pool` 本地属性添加到提交它们的线程中的 SparkContext 来设置作业池。这是完成如下（假设sc是你的 SparkContext )：

```scala
sc.setLocalProperty("spark.scheduler.pool", "pool1")
```

After setting this local property, all jobs submitted within this thread will use this pool name. The setting is per-thread to make it easy to have a thread run multiple jobs on behalf of the same user. If you’d like to clear the pool that a thread is associated with, set it to null.

设置此本地属性后，此线程中提交的所有作业都将使用此池名称。该设置是每个线程，以便让线程代表同一个用户运行多个作业变得容易。如果要清除与线程关联的池，请将其设置为null。

## <font color="#9a161a" >Conclusion 结论</font>

This chapter covered a lot about Spark Applications; we learned how to write, test, run, and configure them in all of Spark’s languages. In Chapter 17, we talk about deploying and the cluster management options you have when it comes to running Spark Applications. 

本章介绍了 Spark 应用程序；我们学习了如何使用Spark的所有语言编写，测试，运行和配置它们。在第17章中，我们将讨论在运行 Spark 应用程序时的部署和集群管理选项。