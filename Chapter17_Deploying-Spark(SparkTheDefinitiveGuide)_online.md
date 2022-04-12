---
title: 翻译 Chapter 17 Deploying Spark
date: 2019-08-07
copyright: true
categories: English,中文
tags: [Spark]
mathjax: false
mathjax2: false
toc: true
---

<img src="http://r91f3wdjg.hn-bkt.clouddn.com/SparkTheDefinitiveGuide/Spark权威指南封面.jpg" alt="1" style="zoom:68%;"/>

# Chapter 17 Deploying Spark

This chapter explores the infrastructure you need in place for you and your team to be able to run Spark Applications: 

本章将探讨您和您的团队能够运行Spark应用程序所需的基础架构：

- Cluster deployment choices 

  集群部署选择

- Spark’s different cluster managers 

  Spark的不同集群管理器

- Deployment considerations and configuring deployments 

  部署考虑事项和配置部署

For the most, part Spark should work similarly with all the supported cluster managers ; however, customizing the setup means understanding the intricacies of each of the cluster management systems. The hard part is deciding on the cluster manager (or choosing a managed service). Although we would be happy to include all the minute details about how you can configure different cluster with different cluster managers, it’s simply impossible for this book to provide hyper-specific details for every situation in every single environment. The goal of this chapter, therefore, is not to discuss each of the cluster managers in full detail, but rather to look at their fundamental differences and to provide a reference for a lot of the material already available on [the Spark website](http://spark.apache.org/docs/latest/cluster-overview.html). Unfortunately, there is no easy answer to “which is the easiest cluster manager to run” because it varies so much by use case, experience, and resources. The Spark documentation site offers a lot of detail about deploying Spark with actionable examples. We do our best to discuss the most relevant points. 

<p>对于大多数人来说，Spark应该与所有受支持的集群管理器类似地工作；但是，自定义设置意味着要了解每个集群管理系统的复杂性。困难的部分是决定集群管理器（或选择托管服务）。虽然我们很乐意提供有关如何使用不同集群管理器配置不同集群的所有细节，但本书根本不可能为每个环境中的每种情况提供超特定的详细信息。因此，本章的目标不是详细讨论每个集群管理器，而是要查看它们的基本差异，并为Spark网站上已有的许多资料提供参考。不幸的是，“哪个是最容易运行的集群管理器”没有简单的答案，因为它因用例，经验和资源而变化很大。 Spark文档站点提供了有关使用可操作示例部署 Spark 的大量详细信息。我们会尽力讨论最相关的观点。</p>
As of this writing, Spark has three officially supported cluster managers : 

在开始撰写本文时，Spark有三个官方支持的集群管理器：


- Standalone mode  独立模式
- Hadoop YARN 
- Apache Mesos 

These cluster managers maintain a set of machines onto which you can deploy Spark Applications. Naturally, each of these cluster managers has an opinionated  view toward management, and so there are trade-offs and semantics that you will need to keep in mind. However, they all run Spark applications the same way (as covered in Chapter 16). Let’s begin with the first point: where to deploy your cluster.

这些集群管理器维护一组可以部署Spark应用程序的计算机。当然，这些集群管理者中的每一个都对管理有一种自以为是的观点，因此需要记住权衡和语义。但是，它们都以相同的方式运行 Spark 应用程序（如第16章所述）。让我们从第一点开始：部署集群的位置。

## <font color="#9a161a" >Where to Deploy Your Cluster to Run Spark Applications 在何处部署集群以运行Spark应用程序</font>

There are two high-level options for where to deploy Spark clusters: deploy in an on-premises cluster or in the public cloud. This choice is consequential  and is therefore worth discussing.

在哪里部署Spark集群有两个高级选项：在本地部署集群或公共云中部署。这种选择是重要的，因此值得讨论。

### <font color="#000000" >On-Premises Cluster Deployments 本地部署集群部署</font>

Deploying Spark to an on-premises cluster is sometimes a reasonable option, especially for organizations that already manage their own datacenters. As with everything else, there are trade-offs to this approach. An on-premises cluster gives you full control over the hardware used, meaning you can optimize performance for your specific workload. However, it also introduces some challenges, especially when it comes to data analytics workloads like Spark. <font style="color:#EA7500">First</font>, with on-premises deployment, your cluster is fixed in size, whereas the resource demands of data analytics workloads are often elastic. If you make your cluster too small, it will be hard to launch the occasional very large analytics query or training job for a new machine learning model, whereas if you make it large, you will have resources sitting idle. <font style="color:#EA7500">Second</font>, for on-premises clusters, you need to select and operate your own storage system, such as a Hadoop file system or scalable key-value store. This includes setting up georeplication and disaster recovery if required.

<p>将 Spark 部署到本地部署集群有时是一种合理的选择，特别是对于已经管理自己的数据中心的组织。与其他一切一样，这种方法存在权衡取舍。本地部署集群使您可以完全控制所使用的硬件，这意味着您可以针对特定工作负载优化性能。但是，它也带来了一些挑战，特别是在Spark等数据分析工作负载方面。<font style="color:#EA7500">首先</font>，通过本地部署，您的集群的大小是固定的，而数据分析工作负载的资源需求通常是弹性的。如果您的集群太小，则很难为新的机器学习模型启动偶尔的非常大的分析查询或训练工作，而如果您将其扩大，则会使资源闲置。<font style="color:#EA7500">其次</font>，对于本地部署集群，您需要选择并运行自己的存储系统，例如 Hadoop 文件系统或可伸缩键值存储。这包括在需要时设置地理复制和灾难恢复。</p>
If you are going to deploy on-premises, the best way to combat the resource utilization problem is to use a cluster manager that allows you to run many Spark applications and dynamically reassign resources between them, or even allows non-Spark applications on the same cluster. All of Spark’s supported cluster managers allow multiple concurrent applications, but YARN and Mesos have better support for dynamic sharing and also additionally support non-Spark workloads. Handling on-premisesresource sharing is likely going to be the biggest difference your users see day to day with Spark on-premises versus in the cloud: in public clouds, it’s easy to give each application its own cluster of exactly the required size for just the duration of that job.

<p>如果要本地部署，解决资源利用率问题的最佳方法是使用集群管理器，它允许您运行许多 Spark 应用程序并在它们之间动态重新分配资源，甚至允许在相同集群上使用非 Spark 应用程序。所有 Spark 支持的集群管理器都允许多个并发应用程序，但 YARN 和 Mesos 可以更好地支持动态共享，还可以支持非Spark工作负载。处理资源共享可能是您的用户每天使用Spark本地部署与云中看到的最大差异：在公共云中，很容易为每个应用程序提供自己的集群，其中包含完全所需的大小那份工作。</p>
For storage, you have several different options, but covering all the trade-offs and operational details in depth would probably require its own book. The most common storage systems used for Spark are distributed file systems such as Hadoop’s HDFS and key-value stores such as Apache Cassandra. Streaming message bus systems such as Apache Kafka are also often used for ingesting data. All these systems have varying degrees of support for management, backup, and georeplication, sometimes built into the system and sometimes only through third-party commercial tools. Before choosing a storage option, we recommend evaluating the performance of its Spark connector and evaluating the available management tools. 

<p>对于存储，您有几种不同的选择，但是深入讨论所有权衡和操作细节可能需要它自己的书。用于 Spark 的最常见存储系统是分布式文件系统，例如 Hadoop 的 HDFS 和键值存储，例如 Apache Cassandra。诸如 Apache Kafka之类的流式消息总线系统也经常用于摄取数据。所有这些系统都对管理，备份和地理复制有不同程度的支持，有时内置于系统中，有时仅通过第三方商业工具。在选择存储选项之前，我们建议您评估其Spark连接器的性能并评估可用的管理工具。</p>

###  Spark in the Cloud 云端Spark</font>

While early big data systems were designed for on-premises deployment, the cloud is now an increasingly common platform for deploying Spark. The public cloud has several advantages when it comes to big data workloads. <font style="color:#EA7500">First</font>, resources can be launched and shut down elastically, so you can run that occasional “monster” job that takes hundreds of machines for a few hours without having to pay for them all the time. Even for normal operation, you can choose a different type of machine and cluster size for each application to optimize its cost performance—for example, launch machines with Graphics Processing Units (GPUs) just for your deep learning jobs. <font style="color:#EA7500">Second</font>, public clouds include low-cost, georeplicated storage that makes it easier to manage large amounts of data. 

<p>虽然早期的大数据系统是为本地部署而设计的，但云现在是部署Spark的日益普遍的平台。在涉及大数据工作负载时，公共云有几个优点。<font style="color:#EA7500">首先</font>，资源可以弹性地启动和关闭，因此您可以运行偶尔的“怪物”工作，这需要数百台机器几个小时，而无需一直为它们付费</font>。即使是正常操作，您也可以为每个应用程序选择不同类型的计算机和群集大小，以优化其性价比——例如，仅为您的深度学习作业启动具有图形处理单元（GPU）的计算机。<font style="color:#EA7500">其次</font>，公共云包括低成本，地理复制的存储，可以更轻松地管理大量数据。</p>

Many companies looking to migrate to the cloud imagine they’ll run their applications in the same way that they run their on-premises clusters. All the major cloud providers (Amazon Web Services [AWS], Microsoft Azure, Google Cloud Platform [GCP], and IBM Bluemix) include managed Hadoop clusters for their customers, which provide HDFS for storage as well as Apache Spark. This is actually not a great way to run Spark in the cloud, however, because by using a fixed-size cluster and file system, you are not going to be able to take advantage of elasticity. Instead, it is generally a better idea to use global storage systems that are decoupled from a specific cluster, such as Amazon S3, Azure Blob Storage, or Google Cloud Storage and spin up machines dynamically for each Spark workload. With decoupled compute and storage, you will be able to pay for computing resources only when needed, scale them up dynamically, and mix different hardware types. Basically, keep in mind that running Spark in the cloud need not mean migrating an on-premises installation to virtual machines: you can run Spark natively against  cloud storage to take full advantage of the cloud’s elasticity, cost-saving benefit, and management tools without having to manage an on-premise computing stack within your cloud environment.

<p>许多希望迁移到云的公司想象他们将以运行其本地部署集群的方式运行其应用程序。所有主要云提供商（Amazon Web Services [AWS]，Microsoft Azure，Google Cloud Platform [GCP]和IBM Bluemix）都为其客户提供托管Hadoop集群，这些集群为存储和Apache Spark提供HDFS。然而，这实际上不是在云中运行Spark的好方法，因为通过使用固定大小的集群和文件系统，您将无法利用弹性。而是，通常最好使用与特定集群（例如Amazon S3，Azure Blob存储或Google云存储）分离的全局存储系统，并为每个Spark工作负载动态启动计算机。通过分离计算和存储，您将能够仅在需要时为计算资源付费，动态扩展计算资源，并混合使用不同的硬件类型。基本上，请记住，在云中运行Spark并不意味着将本地安装迁移到虚拟机：您可以针对云存储本地运行Spark以充分利用云的弹性，成本节约优势和管理工具，而无需必须在云环境中管理本地部署计算堆栈。</p>
Several companies provide “cloud-native” Spark-based services, and all installations of Apache Spark can of course connect to cloud storage. Databricks, the company started by the Spark team from UC Berkeley, is one example of a service provider built specifically for Spark in the cloud. Databricks provides a simple way to run Spark workloads without the heavy baggage of a Hadoop installation. The company provides a number of features for running Spark more efficiently in the cloud, such as auto-scaling, auto-termination of clusters, and optimized connectors to cloud storage, as well as a collaborative environment for working on notebooks and standalone jobs. The company also provides a free Community Edition for learning Spark where you can run notebooks on a small cluster and share them live with others. A fun fact is that this entire book was written using the free Community Edition of Databricks, because we found the integrated Spark notebooks, live collaboration, and cluster management the easiest way to produce and test this content.

<p>有几家公司提供“基于云原生”的基于 Spark 的服务，Apache Spark的所有安装当然都可以连接到云存储</font>。由加州大学伯克利分校的 Spark 团队发起的公司 Databricks 是专门为云中的 Spark 构建的服务提供商的一个例子。  Databricks 提供了一种运行Spark工作负载的简单方法，而无需承担 Hadoop 安装的沉重负担。该公司提供了许多功能，可以在云中更有效地运行Spark，例如自动扩展，集群自动终止，云存储的优化连接器，以及用于处理 notebook 和独立作业的协作环境。该公司还提供免费的社区版学习 Spark，您可以在小型集群上运行 notebook 并与他人分享。一个有趣的事实是，整本书是使用免费的 Databricks 社区版编写的，因为我们发现集成的 Spark notebook，实时协作和集群管理是生成和测试此内容的最简单方法。</p>

If you run Spark in the cloud, much of the content in this chapter might not be relevant because you can often create a separate, short-lived Spark cluster for each job you execute. In that case, the standalone cluster manager is likely the easiest to use. However, you may still want to read this content if you’d like to share a longer-lived cluster among many applications, or to install Spark on virtual machines yourself.

如果您在云中运行Spark，本章中的大部分内容可能都不相关，因为您通常可以为您执行的每个作业创建一个单独的，短期的Spark集群。在这种情况下，独立集群管理器可能是最容易使用的。但是，如果您希望在许多应用程序之间共享一个较长期的集群，或者您自己在虚拟机上安装Spark，则可能仍希望阅读此内容。

## <font color="#9a161a" >Cluster Managers 集群管理器</font>

Unless you are using a high-level managed service, you will have to decide on the cluster manager to use for Spark. Spark supports three aforementioned cluster managers: standalone clusters, Hadoop YARN, and Mesos. Let’s review each of these.

除非您使用的是高级托管服务，否则您必须决定要用于Spark的集群管理器。Spark 支持三个上述集群管理器：独立集群（standalone clusters），Hadoop YARN 和 Mesos。让我们回顾一下这些。

### <font color="#000000" >Standalone Mode 独立模式</font>

Spark’s standalone cluster manager is a lightweight platform built specifically for Apache Spark workloads. Using it, you can run multiple Spark Applications on the same cluster. It also provides simple interfaces for doing so but can scale to large Spark workloads. The main disadvantage of the standalone mode is that it’s more limited than the other cluster managers—in particular, your cluster can only run Spark. It’s probably the best starting point if you just want to quickly get Spark running on a cluster, however, and you do not have experience using YARN or Mesos.

Spark 的独立集群管理器是专为 Apache Spark 工作负载构建的轻量级平台。使用它，您可以在同一个集群上运行多个 Spark 应用程序。它还提供了简单的界面，但可以扩展到大型 Spark 工作负载。独立模式的主要缺点是它比其他集群管理器更有限——特别是，您的集群只能运行 Spark。如果您只想快速让 Spark 在群集上运行，那么这可能是最好的起点，但是您没有使用 YARN 或 Mesos 的经验。

#### <font color="#3399cc">Starting a standalone cluster 启动独立集群</font>

Starting a standalone cluster requires provisioning the machines for doing so. That means starting them up, ensuring that they can talk to one another over the network, and getting the version of Spark you would like to run on those sets of machines. After that, there are two ways to start the cluster: by hand or using built-in launch scripts.

启动独立群集需要配置计算机。这意味着启动它们，确保它们可以通过网络相互通信，并获得您希望在这些机器上运行的 Spark 版本。之后，有两种方法可以启动集群：手动或使用内置启动脚本。

Let’s first launch a cluster by hand. The first step is to start the master process on the machine that we want that to run on, using the following command :  

让我们首先手动启动一个集群。第一步是使用以下命令在我们希望运行的机器上启动主进程：

```shell
$SPARK_HOME/sbin/start-master.sh
```

When we run this command, the cluster manager master process will start up on that machine. Once started, the master prints out a <font face="constant-width" color="#000000" size=3>spark://HOST:PORT URI</font>. You use this when you start each of the worker nodes of the cluster, and you can use it as the master argument to your <font face="constant-width" color="#000000" size=3>SparkSession</font> on application initialization. You can also find this URI on the master’s web UI, which is http://masterip-address:8080 by default. With that URI, start the worker nodes by logging in to each machine and running the following script using the URI you just received from the master node. The master machine must be available on the network of the worker nodes you are using, and the port must be open on the master node, as well:

运行此命令时，集群管理器主进程将在该计算机上启动。一旦启动，master 就打印出一个 spark://HOST:PORT URI。在启动集群的每个工作节点时使用它，并且可以在应用程序初始化时将其用作 SparkSession 的主参数。您还可以在 master 的 Web UI 上找到此 URI，默认情况下为 http://masterip-address:8080 。使用该 URI，通过登录到每台计算机并使用刚从主节点收到的 URI 运行以下脚本来启动工作节点。主机必须在您正在使用的工作节点的网络上可用，并且端口必须在主节点上打开，以及 ：

```shell
$SPARK_HOME/sbin/start-slave.sh <master-spark-URI>
```

As soon as you’ve run that on another machine, you have a Spark cluster running! This process is naturally a bit manual; thankfully there are scripts that can help to automate this process.

只要你在另一台机器上运行它，就会运行一个Spark集群！这个过程自然需要一点手动; 谢天谢地，有些脚本可以帮助自动化这个过程。

#### <font color="#3399cc">Cluster launch scripts 集群启动脚本</font>

You can configure cluster launch scripts that can automate the launch of standalone clusters. To do this, create a file called *conf/slaves* in your Spark directory that will contain the hostnames of all the machines on which you intend to start Spark workers, one per line. If this file does not exist, everything will launch locally. When you go to actually start the cluster, the master machine will access each of the worker machines via <font face="constant-width" color="#000000" size=3>Secure Shell (SSH)</font>. By default, SSH is run in parallel and requires that you configure password-less (using a private key) access. If you do not have a password-less setup, you can set the environment variable <font face="constant-width" color="#000000" size=3>SPARK_SSH_FOREGROUND</font> and serially provide a password for each worker.

您可以配置可以自动启动独立集群的集群启动脚本。为此，在Spark目录中创建一个名为 conf/slaves 的文件，该文件将包含要在其上启动 Spark 工作的所有计算机的主机名，每行一个。如果此文件不存在，则所有内容都将在本地启动。当您真正启动集群时，主计算机将通过 Secure Shell(SSH) 访问每个工作计算机。默认情况下，SSH 是并行运行的，需要您配置无密码（使用私钥）访问。如果您没有无密码设置，则可以设置环境变量SPARK_SSH_FOREGROUND 并为每个工作人员连续提供密码。

After you set up this file, you can launch or stop your cluster by using the following shell scripts, based on Hadoop’s deploy scripts, and available in <font face="constant-width" color="#000000" size=3>$SPARK_HOME/sbin</font>: 

设置此文件后，可以使用以下基于 Hadoop 部署脚本的 shell 脚本启动或停止集群，并在 $SPARK_HOME/sbin 中提供：

<font face="constant-width" color="#000000" size=3>$SPARK_HOME/sbin/start-master.sh</font>

Starts a master instance on the machine on which the script is executed.

在执行脚本的计算机上启动主实例。

<font face="constant-width" color="#000000" size=3>$SPARK_HOME/sbin/start-slaves.sh</font>

Starts a slave instance on each machine specified in the conf/slaves file.

在conf / slaves文件中指定的每台计算机上启动从属实例。

<font face="constant-width" color="#000000" size=3>$SPARK_HOME/sbin/start-slave.sh</font>

Starts a slave instance on the machine on which the script is executed.

在执行脚本的计算机上启动从属实例。

<font face="constant-width" color="#000000" size=3>$SPARK_HOME/sbin/start-all.sh</font>

Starts both a master and a number of slaves as described earlier.

如前所述，启动主服务器和多个从服务器。

<font face="constant-width" color="#000000" size=3>$SPARK_HOME/sbin/stop-master.sh</font>

Stops the master that was started via the bin/start-master.sh script.

停止通过bin / start-master.sh脚本启动的主服务器。

<font face="constant-width" color="#000000" size=3>$SPARK_HOME/sbin/stop-slaves.sh</font>

Stops all slave instances on the machines specified in the conf/slaves file.

停止conf / slaves文件中指定的计算机上的所有从属实例。

<font face="constant-width" color="#000000" size=3>$SPARK_HOME/sbin/stop-all.sh</font>

Stops both the master and the slaves as described earlier.

如前所述，停止主站和从站。

#### <font color="#3399cc">Standalone cluster configurations 独立集群配置</font>

Standalone clusters have a number of configurations that you can use to tune your application. These control everything from what happens to old files on each worker for terminated applications to the worker’s core and memory resources. These are controlled via environment variables or via application properties. Due to space limitations, we cannot include the entire configuration set here. Refer to the relevant table on [Standalone Environment Variables](http://spark.apache.org/docs/latest/spark-standalone.html#cluster-launch-scripts) in [the Spark documentation](http://spark.apache.org/docs/latest/index.html). 

独立集群具有许多可用于调整应用程序的配置。它们控制从终止应用程序的每个工作程序上的旧文件发生到工作程序的核心和内存资源的所有内容。这些是通过环境变量或应用程序属性控制的。由于篇幅限制，我们无法在此处包含整个配置集。请参阅[Spark文档](http://spark.apache.org/docs/latest/index.html)中有关[独立环境变量的相关表](http://spark.apache.org/docs/latest/spark-standalone.html#cluster-launch-scripts)。

#### <font color="#3399cc">Submitting applications 提交申请</font>

After you create the cluster, you can submit applications to it using the <font face="constant-width" color="#000000" size=3>spark://URI</font> of the master. You can do this either on the master node itself or another machine using <font face="constant-width" color="#000000" size=3>spark-submit</font>. There are some specific command-line arguments for standalone mode, which we covered in “Launching Applications”. 

创建集群后，可以使用主服务器的 spark://URI 向其提交应用程序。您可以在主节点本身或使用 spark-submit 的其他计算机上执行此操作。独立模式有一些特定的命令行参数，我们在“启动应用程序”中介绍了这些参数。

### <font color="#000000" >Spark on YARN 运行在YARN上Spark</font>

Hadoop YARN is a framework for job scheduling and cluster resource management. Even though Spark is often (mis)classified as a part of the “Hadoop Ecosystem,” in reality, Spark has little to do with Hadoop. Spark does natively support the Hadoop YARN cluster manager but it requires nothing from Hadoop itself.

Hadoop YARN 是一个用于作业调度和集群资源管理的框架。尽管 Spark 经常（错误地）归类为 “Hadoop生态系统” 的一部分，但事实上，Spark与Hadoop几乎没有关系。 Spark 本身支持 Hadoop YARN 集群管理器，但它不需要Hadoop本身。

You can run your Spark jobs on Hadoop YARN by specifying the master as YARN in the spark-submit command-line arguments. Just like with standalone mode, there are a number of knobs that you are able to tune according to what you would like the cluster to do. The number of knobs is naturally larger than that of Spark’s standalone mode because Hadoop YARN is a generic scheduler for a large number of different execution frameworks.

您可以通过在 spark-submit 命令行参数中将 master 指定为 YARN 来在 Hadoop YARN 上运行 Spark 作业。就像独立模式一样，有许多旋钮可以根据您希望集群执行的操作进行调整。旋钮的数量自然大于 Spark 的独立模式，因为 Hadoop YARN 是大量不同执行框架的通用调度程序。

Setting up a YARN cluster is beyond the scope of this book, but there are some great books on the topic as well as managed services that can simplify this experience.

设置 YARN 集群超出了本书的范围，但是有一些关于该主题的优秀书籍以及可以简化此体验的托管服务。

#### <font color="#3399cc">Submitting applications 提交申请</font>

When submitting applications to YARN, the core difference from other deployments is that `--master` will become yarn as opposed the master node IP, as it is in standalone mode. Instead, Spark will find the YARN configuration files using the environment variable `HADOOP_CONF_DIR` or `YARN_CONF_DIR`. Once you have set those environment variables to your Hadoop installation’s configuration directory, you can just run spark-submit like we saw in Chapter 16.

在向 YARN 提交申请时，与其他部署的核心区别在于 `--master` 将成为 YARN 而不是主节点 IP，正如它处于独立模式的那样。相反，Spark 将使用环境变量 `HADOOP_CONF_DIR` 或 `YARN_CONF_DIR` 找到 YARN 配置文件。一旦将这些环境变量设置到 Hadoop 安装的配置目录中，就可以像我们在第16章中看到的那样运行 `spark-submit`。

---

<center><font face="constant-width" color="#737373" size=4><strong>NOTE 注意</strong><font></center>
There are two deployment modes that you can use to launch Spark on YARN. As discussed in previous chapters, cluster mode has the spark driver as a process managed by the YARN cluster, and the client can exit after creating the application. In client mode, the driver will run in the client process and therefore YARN will be responsible only for granting executor resources to the application, not maintaining the master node. Also of note is that in cluster mode, Spark doesn’t necessarily run on the same machine on which you’re executing. Therefore libraries and external jars must be distributed manually or through the `--jars` command-line argument.

您可以使用两种部署模式在 YARN 上启动 Spark。如前几章所述，集群模式将 Spark 驱动程序作为 YARN 集群管理的进程，客户端可以在创建应用程序后退出。在客户端模式下，驱动程序将在客户端进程中运行，因此 YARN 仅负责向应用程序授予执行器（executor）资源，而不是维护主节点。另外值得注意的是，在集群模式下，Spark不一定在您正在执行的同一台机器上运行。因此，必须手动或通过 `--jars` 命令行参数分发库和外部jar。

---

There are a few YARN-specific properties that you can set by using spark-submit. These allow you to control priority queues and things like keytabs for security. We covered these in “Launching Applications” in Chapter 16.

您可以使用 spark-submit 设置一些特定于 YARN 的属性。这些允许您控制优先级队列和诸如 keytabs 之类的东西以确保安全性。我们在第16章的“启动应用程序”中介绍了这些内容。

#### <font color="#3399cc">Configuring Spark on YARN Applications 在YARN应用程序上配置Spark</font>
Deploying Spark as YARN applications requires you to understand the variety of different configurations and their implications for your Spark applications. This section covers some best practices for basic configurations and includes references to some of the important configuration for running your Spark applications.

将Spark部署为YARN应用程序需要您了解各种不同的配置及其对 Spark 应用程序的影响。本节介绍了一些基本配置的最佳实践，并包括对运行Spark应用程序的一些重要配置的引用。

#### <font color="#3399cc">Hadoop configurations Hadoop配置</font>
If you plan to read and write from HDFS using Spark, you need to include two Hadoop configuration files on Spark’s classpath: `hdfs-site.xml`, which provides default behaviors for the HDFS client; and `core-site.xml`, which sets the default file system name. The location of these configuration files varies across Hadoop versions, but a common location is inside of `/etc/hadoop/conf`. Some tools create these configurations on the fly, as well, so it’s important to understand how your managed service might be deploying these, as well.

如果您计划使用 Spark 从 HDFS 读取和写入，则需要在 Spark 的类路径中包含两个Hadoop配置文件：`hdfs-site.xml`，它为 HDFS 客户端提供默认行为; 和 `core-site.xml`，它设置默认文件系统名称。这些配置文件的位置因 Hadoop 版本而异，但常见位置在 `/etc/hadoop/conf` 中。有些工具也可以动态创建这些配置，因此了解托管服务如何部署这些配置也很重要。

To make these files visible to Spark, set `HADOOP_CONF_DIR` in `$SPARK_HOME/spark-env.sh` to a location containing the configuration files or as an environment variable when you go to spark–submit your application.

<p>要使这些文件对 Spark 可见，请将 `$SPARK_HOME/spark-env.sh` 中的 `HADOOP_CONF_DIR` 设置为包含配置文件的位置，或者当您转到 spark-submit 应用程序时将其设置为环境变量。</p>
#### <font color="#3399cc">Application properties for YARN YARN的应用程序属性</font>

There are a number of Hadoop-related configurations and things that come up that largely don’t have much to do with Spark, just running or securing YARN in a way that influences how Spark runs. Due to space limitations, we cannot include the configuration set here. Refer to the relevant table on [YARN Configurations](http://spark.apache.org/docs/latest/running-on-yarn.html#configuration) in the [Spark documentation](http://spark.apache.org/docs/latest/index.html).

有许多与 Hadoop 相关的配置和出现的东西很大程度上与 Spark 没什么关系，只是以影响 Spark 运行或保护 YARN 的方式。 由于空间限制，我们不能在此处包含配置集。 请参阅 [Spark文档](http://spark.apache.org/docs/latest/index.html) 中有关 [YARN配置的相关表](http://spark.apache.org/docs/latest/running-on-yarn.html#configuration)。

### <font color="#000000" >Spark on Mesos 在Mesos运行的Spark</font>

Apache Mesos is another clustering system that Spark can run on. A fun fact about Mesos is that the project was also started by many of the original authors of Spark, including one of the authors of this book. In the Mesos project’s own words :

Apache Mesos 是 Spark 可以运行的另一个集群系统。关于 Mesos 的一个有趣的事实是该项目也是由Spark的许多原作者创建的，包括本书的作者之一。在Mesos项目中用自己的话来说 ：

Apache Mesos abstracts CPU, memory, storage, and other compute resources away from machines (physical or virtual), enabling fault-tolerant and elastic distributed systems to easily be built and run effectively.

<p>Apache Mesos将CPU，内存，存储和其他计算资源从机器（物理或虚拟）中抽象出来，使容错和弹性分布式系统能够轻松构建并有效运行。</p>
For the most part, Mesos intends to be a datacenter scale-cluster manager that manages not just short-lived applications like Spark, but long-running applications like web applications or other resource interfaces. Mesos is the heaviest-weight cluster manager, simply because you might choose this cluster manager only if your organization already has a large-scale deployment of Mesos, but it makes for a good cluster manager nonetheless.

<p>在大多数情况下，Mesos打算成为一个数据中心规模集群管理器，它不仅管理像Spark这样的短期应用程序，而且管理长期运行的应用程序，如Web应用程序或其他资源接口。 Mesos是最重的集群管理器，仅仅因为您可能只在您的组织已经大规模部署Mesos时才选择此集群管理器，但它仍然是一个优秀的集群管理器。</p>
Mesos is a large piece of infrastructure, and unfortunately there’s simply too much information for us to cover how to deploy and maintain Mesos clusters. There are many great books on the subject for that, including Dipa Dubhashi and Akhil Das’s *Mastering Mesos* (O’Reilly, 2016). The goal here is to bring up some of the considerations that you’ll need to think about when running Spark Applications on Mesos.

Mesos 是一个很大的基础架构，不幸的是，我们有太多的信息来介绍如何部署和维护 Mesos 集群。有很多关于这个主题的好书，包括 Dipa Dubhashi 和 Akhil Das 的 《Mastering Mesos》（O'Reilly，2016）。这里的目标是提出在 Mesos 上运行 Spark 应用程序时需要考虑的一些注意事项。

For instance, one common thing you will hear about Spark on Mesos is fine-grained versus coarse-grained mode. Historically Mesos supported a variety of different modes (fine-grained and coarse-grained), but at this point, it supports only coarse-grained scheduling (fine-grained has been deprecated). Coarse-grained mode means that each Spark executor runs as a single Mesos task. Spark executors are sized according to the following application properties :

例如，你会听到关于 Spark on Mesos 的一个常见的事情是细粒度和粗粒度模式。从历史上看，Mesos支持各种不同的模式（细粒度和粗粒度），但此时，它仅支持粗粒度调度（细粒度已被弃用）。粗粒度模式意味着每个Spark执行程序作为单个Mesos任务运行。 Spark执行程序根据以下应用程序属性调整大小：

```shell
spark.executor.memory
spark.executor.cores
spark.cores.max/spark.executor.cores
```

#### <font color="#3399cc">Submitting applications</font>
Submitting applications to a Mesos cluster is similar to doing so for Spark’s other cluster managers. For the most part you should favor cluster mode when using Mesos. Client mode requires some extra configuration on your part, especially with regard to distributing resources around the cluster. For instance, in client mode, the driver needs extra configuration information in spark-env.sh to work with Mesos.

<p>将应用程序提交到 Mesos 集群与 Spark 的其他集群管理器类似。 在大多数情况下，您应该在使用 Mesos 时使用集群模式。 客户端模式需要您进行一些额外配置，尤其是在集群上分配资源方面。 例如，在客户端模式下，驱动程序需要 spark-env.sh 中的额外配置信息才能使用Mesos。</p>
In spark-env.sh set some environment variables : 

在spark-env.sh中设置一些环境变量：

```shell
export MESOS_NATIVE_JAVA_LIBRARY=<path to libmesos.so>
This path is typically <prefix>/lib/libmesos.so where the prefix is /usr/local by default. On Mac OS
X, the library is called libmesos.dylib instead of libmesos.so:
export SPARK_EXECUTOR_URI=<URL of spark-2.2.0.tar.gz uploaded above>
```

Finally, set the Spark Application property `spark.executor.uri` to <URL of spark-2.2.0.tar.gz>. Now, when starting a Spark application against the cluster, pass a `mesos://URL` as the master when creating a `SparkContex` , and set that property as a parameter in your `SparkConf` variable or the initialization of a `SparkSession` :

最后，将 Spark Application 属性 `spark.executor.uri` 设置为 <URL of spark-2.2.0.tar.gz>。 现在，在针对集群启动 Spark 应用程序时，在创建 `SparkContext`  时将 `mesos://URL` 作为 master 传递，并将该属性设置为 `SparkConf` 变量中的参数或 `SparkSession` 的初始化：

```shell
// in Scala
import org.apache.spark.sql.SparkSession
val spark = SparkSession.builder
.master("mesos://HOST:5050")
.appName("my app")
.config("spark.executor.uri", "<path to spark-2.2.0.tar.gz uploaded above>")
.getOrCreate()
```

Submitting cluster mode applications is fairly straightforward and follows the same spark-submit structure you read about before. We covered these in “Launching Applications”.

提交集群模式应用程序非常简单，并遵循您之前阅读的相同的 spark-submit 结构。 我们在“启动应用程序”中介绍了这些内容。

#### <font color="#3399cc">Configuring Mesos 配置Mesos</font>

Just like any other cluster manager, there are a number of ways that we can configure our Spark Applications when they’re running on Mesos. Due to space limitations, we cannot include the entire configuration set here. Refer to the relevant table on [Mesos Configurations](http://spark.apache.org/docs/latest/running-on-mesos.html#configuration) in [the Spark documentation](http://spark.apache.org/docs/latest/index.html). 

就像任何其他集群管理器一样，我们可以通过多种方式在 Spark 应用程序运行时配置它们。 由于篇幅限制，我们无法在此处包含整个配置集。 请参阅 [Spark文档](http://spark.apache.org/docs/latest/index.html) 中有关 [Mesos配置的相关表](http://spark.apache.org/docs/latest/running-on-mesos.html#configuration) 。

### <font color="#000000" >Secure Deployment Configurations 安全部署配置</font>
Spark also provides some low-level ability to make your applications run more securely, especially in untrusted environments. Note that the majority of this setup will happen outside of Spark. These configurations are primarily network-based to help Spark run in a more secure manner. This means authentication, network encryption, and setting TLS and SSL configurations. Due to space limitations, we cannot include the entire configuration set here. Refer to the relevant table on [Security Configurations](http://spark.apache.org/docs/latest/security.html) in [the Spark documentation](http://spark.apache.org/docs/latest/index.html).

Spark 还提供了一些低阶功能，使您的应用程序运行更安全，尤其是在不受信任的环境中。请注意，此设置的大部分将在 Spark 之外发生。这些配置主要基于网络，以帮助 Spark 以更安全的方式运行。这意味着身份验证，网络加密以及设置 TLS 和 SSL 配置。由于篇幅限制，我们无法在此处包含整个配置集。请参阅 [Spark文档](http://spark.apache.org/docs/latest/index.html) 中有关 [安全配置的相关表](http://spark.apache.org/docs/latest/security.html) 。

### <font color="#000000" >Cluster Networking Configurations 集群网络配置</font>
Just as shuffles are important, there can be some things worth tuning on the network. This can also be helpful when performing custom deployment configurations for your Spark clusters when you need to use proxies in between certain nodes. If you’re looking to increase Spark’s performance, these should not be the first configurations you go to tune, but may come up in custom deployment scenarios. Due to space limitations, we cannot include the entire configuration set here. Refer to the relevant table on [Networking Configurations](http://spark.apache.org/docs/latest/configuration.html#networking) in [the Spark documentation](http://spark.apache.org/docs/latest/index.html).

<p>正如数据再分配（shuffle）很重要，网络上可能会有一些值得调整的东西。当您需要在某些节点之间使用代理时，在为Spark 集群执行自定义部署配置时，这也很有用。如果您希望提高Spark的性能，这些不应该是您要调整的第一个配置，但可能会出现在自定义部署方案中。由于篇幅限制，我们无法在此处包含整个配置集。请参阅[Spark文档](http://spark.apache.org/docs/latest/index.html) 中的[网络配置相关表](http://spark.apache.org/docs/latest/configuration.html#networking)。</p>

### <font color="#000000" >Application Scheduling 应用程序调度</font>

Spark has several facilities for scheduling resources between computations. <font style="color:#EA7500">First</font>, recall that, as described earlier in the book, each Spark Application runs an independent set of executor processes. Cluster managers provide the facilities for scheduling across Spark applications. <font style="color:#EA7500">Second</font>, within each Spark application, multiple jobs (i.e., Spark actions) may be running concurrently if they were submitted by different threads. This is common if your application is serving requests over the network. Spark includes a fair scheduler to schedule resources within each application. We introduced this topic in the previous chapter.

Spark 有几种用于在计算之间调度资源的工具。<font style="color:#EA7500">首先</font>，回想一下，如本书前面所述，每个 Spark 应用程序都运行一组独立的执行器（executor）进程。集群管理器提供跨Spark应用程序进行调度的工具。<font style="color:#EA7500">其次</font>，在每个Spark应用程序中，如果它们是由不同的线程提交的，则多个作业（即 Spark actions ）可以同时运行。如果您的应用程序通过网络提供请求，这种情况很常见。 Spark包含一个公平的调度程序（fair scheduler）来安排每个应用程序中的资源。我们在前一章介绍了这个主题。

If multiple users need to share your cluster and run different Spark Applications, there are different options to manage allocation, depending on the cluster manager. The simplest option, available on all cluster managers, is static partitioning of resources. With this approach, each application is given a maximum amount of resources that it can use, and holds onto those resources for the entire duration. In spark-submit there are a number of properties that you can set to control the resource allocation of a particular application. Refer to Chapter 16 for more information. In addition, dynamic allocation (described next) can be turned on to let applications scale up and down dynamically based on their current number of pending tasks. If, instead, you want users to be able to share memory and executor resources in a fine-grained manner, you can launch a single Spark Application and use thread scheduling within it to serve multiple requests in parallel.

<p>如果多个用户需要共享您的集群并运行不同的Spark应用程序，则可以使用不同的选项来管理分配，具体取决于集群管理器。所有集群管理器上都可以使用的最简单的选项是资源的静态分区（static partitioning of resources）。通过这种方法，每个应用程序都可以使用最多的资源，并在整个持续时间内保留这些资源。在spark-submit中，您可以设置许多属性来控制特定应用程序的资源分配。有关更多信息，请参阅第16章。此外，可以打开动态分配（下面描述），让应用程序根据当前挂起的任务数量动态扩展和缩小。相反，如果您希望用户能够以细粒度的方式共享内存和执行程序资源，则可以启动单个Spark应用程序并在其中使用线程调度来并行处理多个请求。</p>

### <font color="#000000" >Dynamic allocation 动态分配</font>
If you would like to run multiple Spark Applications on the same cluster, Spark provides a mechanism to dynamically adjust the resources your application occupies based on the workload. This means that your application can give resources back to the cluster if they are no longer used, and request them again later when there is demand. This feature is particularly useful if multiple applications share resources in your Spark cluster.

<p>如果您希望在同一个集群上运行多个Spark应用程序，Spark会提供一种机制，根据工作负载动态调整应用程序占用的资源。这意味着如果不再使用，您的应用程序可以将资源提供给集群，并在需要时稍后再次请求它们。如果多个应用程序共享Spark集群中的资源，则此功能特别有用。</p>
This feature is disabled by default and available on all coarse-grained cluster managers; that is, standalone mode, YARN mode, and Mesos coarse-grained mode. There are two requirements for using this feature. <font style="color:#EA7500">First</font>, your application must set `spark.dynamicAllocation.enabled` to true. <font style="color:#EA7500">Second</font>, you must set up an external shuffle service on each worker node in the same cluster and set `spark.shuffle.service.enabled` to true in your application. The purpose of the external shuffle service is to allow executors to be removed without deleting shuffle files written by them. This is set up differently for each cluster manager and is described in [the job scheduling configuration](http://spark.apache.org/docs/latest/job-scheduling.html#configuration-and-setup). Due to space limitations, we cannot include the configuration set for dynamic allocation. Refer to the relevant table on [Dynamic Allocation Configurations](http://spark.apache.org/docs/latest/job-scheduling.html#dynamic-resource-allocation). 

<p>默认情况下禁用此功能，并且所有粗粒度集群管理器均可使用此功能; 也就是独立模式，YARN模式和Mesos粗粒度模式。使用此功能有两个要求。<font style="color:#EA7500">首先</font>，您的应用程序必须将 `spark.dynamicAllocation.enabled` 设置为true。<font style="color:#EA7500">其次</font>，必须在同一群集中的每个工作节点上设置外部数据再分配（shuffle） 服务，并在应用程序中将 `spark.shuffle.service.enabled` 设置为 true。外部 数据再分配shuffle 服务的目的是允许删除执行程序而不删除它们写入的shuffle文件。对于每个集群管理器，此设置不同，并在<a href="http://spark.apache.org/docs/latest/job-scheduling.html#configuration-and-setup" target="_blank">作业调度配置</a>中进行了描述。由于空间限制，我们不能包含动态分配的配置集。请参阅<a href="http://spark.apache.org/docs/latest/job-scheduling.html#dynamic-resource-allocation" target="_blank">动态分配配置的相关表</a>。</p>

## <font color="#9a161a" >Miscellaneous Considerations 各种各样的考虑因素</font>
There several other topics to consider when deploying Spark applications that may affect your choice of cluster manager and its setup. These are just things that you should think about when comparing different deployment options.

在部署可能影响您选择的集群管理器及其设置的 Spark 应用程序时，还需要考虑其他几个主题。在比较不同的部署选项时，您应该考虑这些事项。

One of the more important considerations is the number and type of applications you intend to be running. For instance, YARN is great for HDFS-based applications but is not commonly used for much else. Additionally, it’s not well designed to support the cloud, because it expects information to be available on HDFS. Also, compute and storage is largely coupled together, meaning that scaling your cluster involves scaling both storage and compute instead of just one or the other. Mesos does improve on this a bit conceptually, and it supports a wide range of application types, but it still requires pre-provisioning machines and, in some sense, requires buy-in at a much larger scale. For instance, it doesn’t really make sense to have a Mesos cluster for only running Spark Applications. Spark standalone mode is the lightest-weight cluster manager and is relatively simple to understand and take advantage of, but then you’re going to be building more application management infrastructure that you could get much more easily by using YARN or Mesos.

<p>其中一个更重要的考虑因素是您打算运行的应用程序的数量和类型</font>。例如，YARN 非常适合基于 HDFS 的应用程序，但并不常用于其他许多应用程序。此外，它还没有很好地支持云，因为它希望在 HDFS 上提供信息。此外，计算和存储在很大程度上是耦合在一起的，这意味着扩展集群涉及扩展存储和计算，而不仅仅是一个或另一个。 Mesos 在概念上确实有所改进，它支持多种应用类型，但它仍然需要预配置机器，从某种意义上说，需要更大规模的支持。例如，只运行 Spark 应用程序的 Mesos 集群没有真正发挥它的价值。Spark 独立模式是最轻量级的集群管理器，并且相对易于理解和利用，但随后您将构建更多应用程序管理基础架构，使用 YARN 或 Mesos 可以更轻松地获得这些基础架构。</p>

Another challenge is managing different Spark versions. Your hands are largely tied if you want to try to run a variety of different applications running different Spark versions, and unless you use a well-managed service, you’re going to need to spend a fair amount of time either managing different setup scripts for different Spark services or removing the ability for your users to use a variety of different Spark applications.

<p>另一个挑战是管理不同的 Spark 版本。如果你想尝试运行运行不同 Spark 版本的各种不同应用程序，你的手很大程度上是捆绑在一起的，除非你使用管理良好的服务，否则你需要花费相当多的时间来管理不同的设置脚本用于不同的 Spark 服务或删除用户使用各种不同 Spark 应用程序的能力。</p>
Regardless of the cluster manager that you choose, you’re going to want to consider how you’re going to set up logging, store logs for future reference, and allow end users to debug their applications. These are more “out of the box” for YARN or Mesos and might need some tweaking if you’re using standalone.

<p>无论您选择哪个集群管理器，您都会想要考虑如何设置日志记录，存储日志以供将来参考，以及允许最终用户调试其应用程序。对于 YARN 或 Mesos 来说，这些更“开箱即用”，如果您使用独立的话，可能需要进行一些调整。</p>
One thing you might want to consider—or that might influence your decision making—is maintaining a metastore in order to maintain metadata about your stored datasets, such as a table catalog. We saw how this comes up in Spark SQL when we are creating and maintaining tables. Maintaining an Apache Hive metastore, a topic beyond the scope of this book, might be something that’s worth doing to facilitate more productive, cross-application referencing to the same datasets. Depending on your workload, it might be worth considering using Spark’s external shuffle service. Typically Spark stores shuffle blocks (shuffle output) on a local disk on that particular node. An external shuffle service allows for storing those shuffle blocks so that they are available to all executors, meaning that you can arbitrarily kill executors and still have their shuffle outputs available to other applications.

<p>您可能想要考虑的一件事——或者可能影响您决策的一件事——是维护一个 Metastore，以维护有关您存储的数据集的元数据，例如表目录。我们在创建和维护表时看到了如何在 Spark SQL 中出现这种情况。维护 Apache Hive Metastore 是一个超出本书范围的主题，可能值得做些什么来促进对同一数据集的更高效，跨应用程序的引用。根据您的工作量，可能值得考虑使用 Spark 的外部 shuffle 服务。通常，Spark 将 shuffle 块（ shuffle 输出）存储在该特定节点上的本地磁盘上。外部 shuffle 服务允许存储这些 shuffle 块，以便它们可供所有执行程序使用，这意味着您可以任意杀死执行程序，并且仍然可以将其 shuffle 输出提供给其他应用程序。</p>
Finally, you’re going to need to configure at least some basic monitoring solution and help users debug their Spark jobs running on their clusters. This is going to vary across cluster management options and we touch on some of the things that you might want to set up in Chapter 18.

<p>最后，您将需要至少配置一些基本监视解决方案，并帮助用户调试在其集群上运行的 Spark 作业。这在集群管理选项中会有所不同，我们会讨论您可能希望在第18章中设置的一些内容。</p>
## <font color="#9a161a" >Conclusion 结论</font>
This chapter looked at the world of configuration options that you have when choosing how to deploy Spark. Although most of the information is irrelevant to the majority of users, it is worth mentioning if you’re performing more advanced use cases. It might seem fallacious, but there are other configurations that we have omitted that control even lower-level behavior. You can find these in the Spark documentation or in the Spark source code. Chapter 18 talks about some of the options that we have when monitoring Spark Applications. 

本章介绍了在选择部署 Spark 时所具有的配置选项的世界。虽然大多数信息与大多数用户无关，但如果您正在执行更高级的用户案例，则值得一提。它可能看起来很荒谬，但是我们已经省略了其他配置来控制甚至更低阶的行为。您可以在 Spark 文档或 Spark 源代码中找到它们。第18章讨论了监视Spark应用程序时的一些选项。
