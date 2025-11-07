# 附录 F. 处理大型数据集

R 将其所有对象存储在虚拟内存中。对于我们大多数人来说，这个设计决策导致了快速的交互式体验，但对于处理大型数据集的分析师来说，它可能导致程序执行缓慢和内存相关错误。

内存限制主要取决于 R 的构建版本（32 位与 64 位）以及涉及的操作系统版本。以“无法分配大小为”开头的错误信息通常表明无法获得足够的连续内存，而以“无法分配长度为”开头的错误信息则表明已超出地址限制。当处理大型数据集时，如果可能的话，尽量使用 64 位构建版本。查看`help(Memory)`获取更多信息。

当处理大型数据集时，有三个问题需要考虑：提高编程效率以加快执行速度、外部存储数据以限制内存问题，以及使用专门设计的统计程序以有效地分析大量数据。首先，我们将考虑每个问题的简单解决方案。然后，我们将转向更全面（且更复杂）的解决方案，以处理*大数据*。

## F.1 高效编程

几个编程技巧可以帮助你在处理大型数据集时提高性能：

+   当可能时，将计算向量化。使用 R 内置的用于操作向量、矩阵和列表的函数（例如，`ifelse`、`colMeans`和`rowSums`），并在可行的情况下避免使用循环（`for`和`while`）。

+   使用矩阵而不是数据框（它们的开销更小）。

+   当导入分隔符文本文件时，使用来自`data.table`包的优化函数`fread()`或来自`vroom`包的`vroom()`。它们比基础 R 的`read.table()`函数快得多。

+   初始时正确设置对象的大小，而不是通过追加值从小对象增长。

+   对于重复的、独立的和数值密集型任务，使用并行化。

+   在尝试在完整数据集上运行之前，在数据样本上测试程序以优化代码并删除错误。

+   删除临时对象和不再需要的对象。调用`rm(list=ls())`将从内存中删除所有对象，提供一个干净的起点。可以使用`rm(object)`删除特定对象。删除大对象后，调用`gc()`将启动垃圾回收，确保对象从内存中删除。

+   使用 Jeromy Anglim 在其博客条目“R 中的内存管理：一些技巧和窍门”中描述的函数`.ls.objects()`来按大小（MB）排序列出所有工作空间对象（[`mng.bz/xGpq`](http://mng.bz/xGpq)）。此函数将帮助你找到并处理内存消耗者。

+   分析程序以查看每个函数花费了多少时间。你可以使用`Rprof()`和`summaryRprof()`函数完成此操作。`system.time()`函数也可以有所帮助。`profr`和`prooftools`包提供了可以帮助你分析分析输出的函数。

+   使用编译的外部例程来加速程序执行。当需要更优化的子例程时，您可以使用`Rcpp`包将 R 对象传输到 C++函数并返回。

第 20.5 节提供了向量化、高效数据输入、正确设置对象大小和并行化的示例。

对于大型数据集，提高代码效率只能走这么远。当您遇到内存限制或代码执行缓慢时，请考虑升级您的硬件。您还可以将数据存储在外部，并使用专门的分析例程。

## F.2 在 RAM 之外存储数据

有几个包可用于在 R 的主内存之外存储数据。该策略涉及将数据存储在外部数据库或磁盘上的二进制平面文件中，然后按需访问。表 F.1 列出了几个有用的包。

表 F.1 R 包用于访问大型数据集

| 包 | 描述 |
| --- | --- |
| `bigmemory` | 支持创建、存储、访问和操作庞大矩阵。矩阵分配到共享内存和内存映射文件。 |
| `disk.frame` | 一个基于磁盘的数据操作框架，用于处理大于 RAM 的数据集 |
| `ff` | 提供了存储在磁盘上的数据结构，但表现得像它们在 RAM 中一样 |
| `filehash` | 实现了一个简单的键值数据库，其中字符串键与存储在磁盘上的数据值相关联 |
| `ncdf`、`ncdf4` | 提供了访问 Unidata NetCDF 数据文件的接口 |
| `RODBC`、`RMySQL`、`ROracle`、`RPostgreSQL`、`RSQLite` | 每个都提供了访问外部关系型数据库管理系统的接口。 |

这些包有助于克服 R 在数据存储上的内存限制。但是，当您尝试在合理的时间内分析大型数据集时，您还需要专门的方法。以下是一些最有用的方法。

## F.3 内存外数据分析包

R 提供了几个用于分析大型数据集的包：

+   `biglm`和`speedglm`包以内存高效的方式对大型数据集进行线性模型和广义线性模型的拟合。这为处理大型数据集提供了`lm()`和`glm()`类型的功能。

+   几个包提供了用于处理由`bigmemory`包生成的庞大矩阵的分析函数。`biganalytics`包提供了 k-means 聚类、列统计以及`biglm`的包装器。`bigrf`包可用于拟合分类和回归森林。`bigtabulate`包提供了`table()`、`split()`和`tapply()`功能，而`bigalgebra`包提供了高级线性代数函数。

+   当与`ff`包一起使用时，`biglars`包为太大而无法存储在内存中的数据集提供了最小角回归、lasso 和逐步回归。

+   `data.table` 包提供了一个 `data.frame` 的增强版本，包括更快的聚合；更快的有序和重叠范围连接；以及通过引用按组快速添加、修改和删除列（无需复制）。您可以使用 `data.table` 结构处理大型数据集（例如，RAM 中 100 GB），并且它与任何期望数据框的 R 函数兼容。

这些包针对特定目的适应大型数据集，并且相对容易使用。接下来将描述用于分析千兆级数据的更全面解决方案。

## F.4 与海量数据集一起工作的综合解决方案

当我撰写这本书的第一版时，在这个部分我能说的最多就是，“好吧，我们正在尝试。”从那时起，结合高性能计算（HPC）和 R 语言的项目的数量激增。本节提供了使用 R 处理千兆级数据集的一些更流行方法的指针。每种方法都需要对 HPC 和其他软件平台（如 Hadoop，一个用于在分布式计算环境中处理大型数据集的免费 Java 软件框架）的使用有一定了解。

表 F.2 描述了使用 R 处理大规模数据集的开源方法。最流行的方法是 RHadoop 和 sparklyr。

表 F.2 R 开源大数据平台

| 方法 | 描述 |
| --- | --- |
| RHadoop | 在 R 环境中使用 Hadoop 管理和分析数据的软件。由五个相互连接的包组成：`rhbase`、`ravro`、`rhdfs`、`plyrmr` 和 `rmr2`。有关详细信息，请参阅 [`github.com/RevolutionAnalytics/RHadoop/wiki`](https://github.com/RevolutionAnalytics/RHadoop/wiki)。 |
| RHIPE | *R 和 Hadoop 集成编程环境*。一个 R 包，允许用户在 R 中运行 Hadoop MapReduce 作业。有关信息，请参阅 [`github.com/delta-rho/RHIPE`](https://github.com/delta-rho/RHIPE)。 |
| Hadoop Streaming | Hadoop streaming ([`hadoop.apache.org/docs/r1.2.1/streaming.html`](https://hadoop.apache.org/docs/r1.2.1/streaming.html)) 是一个用于创建和运行以任何语言作为映射器或/或归约器的 Map/Reduce 作业的实用程序。`HadoopStreaming` 包支持使用 R 编写这些脚本。有关信息，请参阅 [`cran.rstudio.com/web/packages/HadoopStreaming/`](https://cran.rstudio.com/web/packages/HadoopStreaming/)。 |
| RHIVE | 一个通过 HIVE 查询促进分布式计算的 R 扩展。RHIVE 支持在 R 中轻松使用 HIVE SQL，以及在 Hive 中使用 R 对象和 R 函数。有关信息，请参阅 [`github.com/nexr/RHive`](https://github.com/nexr/RHive)。 |
| pbdR | “使用 R 进行大数据编程。”通过简单界面访问可扩展、高性能库（如 MPI、ZeroMQ、ScaLAPACK 和 netCDF4 以及 PAPI）的包，实现 R 中的数据并行。pbdR 软件还支持在大型计算集群上使用单程序多数据（SPMD）模型。有关详细信息，请参阅 [`r-pbd.org`](http://r-pbd.org)。 |
| sparklyr | 一个提供 Apache Spark R 接口的包。支持连接到本地和远程 Apache Spark 集群，提供 `'dplyr'` 兼容的后端和 Spark 机器学习算法的接口。有关详细信息，请参阅 [`cran.r-project.org/web/packages/sparklyr`](https://cran.r-project.org/web/packages/sparklyr)。 |

云服务提供了一种现成的、可扩展的基础设施，具有可能巨大的内存和存储资源。为 R 用户最受欢迎的云服务由亚马逊、微软和谷歌提供。虽然不是云解决方案，但甲骨文也向 R 用户提供了大数据计算（见表 F.3）。

表 F.3 大数据项目商业平台

| 方法 | 描述 |
| --- | --- |
| 亚马逊网络服务 (AWS) | 使用 R 与 AWS 有几种方法。R 包 `paws`（R 中 AWS 的包）提供了从 R 内部访问 AWS 服务的完整套件（见 [`paws-r.github.io/`](https://paws-r.github.io/)）。`aws.ec2` 包是 AWS EC2 REST API 的简单客户端包（[`github.com/cloudyr/aws.ec2`](https://github.com/cloudyr/aws.ec2)）。Louis Aslett 维护的亚马逊机器镜像使得将 RStudio 服务器部署到亚马逊 EC2 服务变得相对简单（[`www.louisaslett.com/RStudio_AMI/`](https://www.louisaslett.com/RStudio_AMI/)）。 |
| 微软 Azure | 数据科学虚拟机是 Azure 云平台上的虚拟机镜像，专为数据科学构建。它们包括 Microsoft R Open、Microsoft 机器学习服务器、RStudio 桌面和 RStudio 服务器。有关详细信息，请参阅 [`azure.microsoft.com/en-us/ services/virtual-machines/data-science-virtual-machines/`](https://azure.microsoft.com/en-us/services/virtual-machines/data-science-virtual-machines/)。另外，请参阅 `AzureR`，这是一系列用于从 R 操作 Azure 的包（[`github.com/Azure/AzureR`](https://github.com/Azure/AzureR)）。 |
| 谷歌云服务 | `bigrquery` 包提供了对谷歌 BigQuery API 的接口（[`github.com/r-dbi/bigrquery`](https://github.com/r-dbi/bigrquery)）。`googleComputeEngineR` 包提供了 R 对 Google Cloud Compute Engine API 的接口。它使得为 R（包括 RStudio 和 Shiny）部署云资源变得尽可能简单（[`cloudyr.github.io/googleComputeEngineR/`](https://cloudyr.github.io/googleComputeEngineR/)）。 |
| Oracle R 高级分析 Hadoop | 这是一个 R 包集合，提供了对 HIVE 表、Hadoop 基础设施、Oracle 数据库表和本地 R 环境的接口。它还包括一系列预测分析技术。请参阅 Oracle 帮助文件（[`mng.bz/K4jn`](http://mng.bz/K4jn)）。 |

除了表 F.3 中的资源外，考虑 cloudyr 项目（[`cloudyr.github.io/`](https://cloudyr.github.io/）），这是一个旨在使使用 R 进行云计算更容易的倡议。它包含了一系列用于融合 R 和云服务优势的包。

在任何语言中处理从千兆到太字节范围的数据库集都可能具有挑战性。每种方法都伴随着一个显著的学习曲线。书籍《使用 R 和 Hadoop 进行大数据分析》（Prajapati，2013）是使用 R 与 Hadoop 的一个有用资源。对于 Spark，阅读《精通 Spark 与 R》（[https://therinspark.com/](https://therinspark.com/））和《使用任意代码从 R 中利用 Spark 进行性能优化》（[https://sparkfromr.com/](https://sparkfromr.com/））都非常有帮助。最后，查看 CRAN 任务视图：“*使用 R 进行高性能和并行计算*”（[https://cran.r-project.org/web/views/HighPerformanceComputing.html](https://cran.r-project.org/web/views/HighPerformanceComputing.html)）。这是一个快速变化和发展的领域，所以请确保经常检查任务视图。
