# 第九章：使用 Apache Spark 构建可靠的数据湖

在前几章中，您学习了如何轻松有效地使用 Apache Spark 构建可扩展和高性能的数据处理流水线。然而，在实践中，仅仅表达处理逻辑只解决了构建流水线端到端问题的一半。对于数据工程师、数据科学家或数据分析师来说，构建流水线的最终目标是查询处理后的数据并从中获取见解。存储解决方案的选择决定了从原始数据到见解（即端到端）的数据流水线的稳健性和性能。

在本章中，我们将首先讨论您需要关注的存储解决方案的关键特性。然后，我们将讨论两类广泛的存储解决方案，即数据库和数据湖，以及如何与它们一起使用 Apache Spark。最后，我们将介绍存储解决方案的下一波发展，称为湖仓库，并探讨这个领域中一些新的开源处理引擎。

# **优化存储解决方案的重要性**

以下是存储解决方案中所需的一些属性：

可扩展性和性能

存储解决方案应能够扩展到所需的数据量，并提供工作负载所需的读/写吞吐量和延迟。

事务支持

复杂的工作负载通常同时读写数据，因此支持[ACID 事务](https://oreil.ly/6Jn97)对于确保最终结果的质量至关重要。

支持多种数据格式

存储解决方案应能够存储非结构化数据（例如原始日志文本文件）、半结构化数据（例如 JSON 数据）和结构化数据（例如表格数据）。

支持多样化的工作负载

存储解决方案应能够支持多样化的业务工作负载，包括：

+   像传统 BI 分析这样的 SQL 工作负载

+   像传统 ETL 作业处理原始非结构化数据的批处理工作负载

+   像实时监控和报警这样的流式工作负载

+   像推荐和流失预测这样的 ML 和 AI 工作负载

开放性

支持广泛的工作负载通常要求数据以开放数据格式存储。标准 API 允许从各种工具和引擎访问数据。这使得企业可以针对每种类型的工作负载使用最优的工具，并做出最佳的业务决策。

随着时间的推移，不同类型的存储解决方案被提出，每种解决方案在这些属性方面都有其独特的优缺点。在本章中，我们将探讨可用的存储解决方案是如何从*数据库*发展到*数据湖*，以及如何与每种解决方案一起使用 Apache Spark。然后我们将转向下一代存储解决方案，通常被称为数据*湖仓库*，它们可以提供数据湖的可扩展性和灵活性，同时具备数据库的事务性保证。

# 数据库

多年来，数据库一直是构建存储业务关键数据的最可靠解决方案。在本节中，我们将探讨数据库及其工作负载的架构，以及如何在数据库上使用 Apache Spark 进行分析工作负载。我们将结束本节讨论数据库在支持现代非 SQL 工作负载方面的限制。

## 数据库简介

数据库旨在以表格形式存储结构化数据，可以使用 SQL 查询进行读取。数据必须遵循严格的模式，这允许数据库管理系统在数据存储和处理方面进行高度协同优化。也就是说，它们紧密地将数据和索引的内部布局与高度优化的查询处理引擎耦合在磁盘文件中，因此能够在存储的数据上提供非常快速的计算，并在所有读写操作上提供强大的事务 ACID 保证。

数据库上的 SQL 工作负载可以广泛分类为两类，如下所示：

[在线事务处理 (OLTP) 工作负载](https://oreil.ly/n94tD)

像银行账户交易一样，OLTP 工作负载通常是高并发、低延迟、简单查询，每次读取或更新少量记录。

[在线分析处理 (OLAP)](https://oreil.ly/NJQ2m)

OLAP 工作负载，例如周期性报告，通常是涉及聚合和连接的复杂查询，需要高吞吐量的扫描许多记录。

值得注意的是，Apache Spark 是专为 OLAP 工作负载而设计的查询引擎，而不是 OLTP 工作负载。因此，在本章的其余部分，我们将专注于分析工作负载的存储解决方案讨论。接下来，让我们看看如何使用 Apache Spark 读写数据库。

## 使用 Apache Spark 读写数据库

由于日益增长的连接器生态系统，Apache Spark 能够连接多种数据库进行数据读写。对于具有 JDBC 驱动程序的数据库（例如 PostgreSQL、MySQL），您可以使用内置的 JDBC 数据源以及适当的 JDBC 驱动程序 jar 包访问数据。对于许多其他现代数据库（例如 Azure Cosmos DB、Snowflake），还有专用连接器，可以使用适当的格式名称调用。本书的第五章详细讨论了几个示例，这使得基于 Apache Spark 的数据仓库和数据库的工作负载和用例扩展变得非常简单。

## 数据库的限制

自上个世纪以来，数据库和 SQL 查询被认为是构建 BI 工作负载的重要解决方案。然而，过去十年中出现了两大新的分析工作负载趋势：

数据规模的增长

随着大数据的出现，全球工业界出现了一种趋势，即为了理解趋势和用户行为，衡量和收集一切（页面访问量、点击等）。因此，任何公司或组织收集的数据量从几十年前的几吉字节增加到今天的几百或几千吉字节。

分析多样性的增长

随着数据收集量的增加，深入洞察的需求也在增加。这导致了像机器学习和深度学习这样的复杂分析技术的爆炸性增长。

数据库已被证明在适应这些新趋势方面相当不足，原因如下：

数据库在横向扩展方面成本极高

虽然数据库在单机上处理数据非常高效，但数据量的增长速度远远超过了单机性能的增长。处理引擎前进的唯一途径是横向扩展，即使用多台机器并行处理数据。然而，大多数数据库，尤其是开源数据库，并未设计成能够横向扩展执行分布式处理。少数能够满足处理需求的工业级数据库解决方案，往往是运行在专用硬件上的专有解决方案，因此非常昂贵，无论是获取还是维护。

数据库对非 SQL 基础分析的支持不够好

数据库以复杂（通常是专有的）格式存储数据，这些格式通常被高度优化，只能由该数据库的 SQL 处理引擎有效读取。这意味着其他处理工具，如机器学习和深度学习系统，无法有效地访问数据（除非通过从数据库中低效地读取所有数据）。数据库也不能轻易扩展以执行像机器学习这样的非 SQL 基础分析。

正是由于这些数据库的限制，才引发了一种完全不同的存储数据方法的发展，即*数据湖*。

# 数据湖

与大多数数据库相反，数据湖是一种分布式存储解决方案，运行在通用硬件上，可以轻松实现横向扩展。在本节中，我们将从讨论数据湖如何满足现代工作负载的需求开始，然后看看 Apache Spark 如何与数据湖集成，使工作负载能够处理任意规模的数据。最后，我们将探讨数据湖为实现可扩展性而做出的架构上的牺牲所带来的影响。

## 数据湖简介

数据湖架构与数据库的架构不同，它将分布式存储系统与分布式计算系统解耦。这使得每个系统可以根据工作负载的需求进行横向扩展。此外，数据以开放格式的文件保存，因此任何处理引擎都可以使用标准 API 读取和写入它们。这个想法在 2000 年代后期由 Apache Hadoop 项目中的 Hadoop 文件系统（HDFS）推广开来，该项目本身深受 Sanjay Ghemawat、Howard Gobioff 和 Shun-Tak Leung 的研究论文《Google 文件系统》的启发。

组织通过独立选择以下内容来构建他们的数据湖：

存储系统

他们选择在机器群集上运行 HDFS 或使用任何云对象存储（例如 AWS S3、Azure Data Lake Storage 或 Google Cloud Storage）。

文件格式

根据下游工作负载的不同，数据以文件形式存储，可以是结构化（例如 Parquet、ORC）、半结构化（例如 JSON）或有时甚至是非结构化格式（例如文本、图像、音频、视频）。

处理引擎

再次，根据要执行的分析工作负载的类型，选择处理引擎。这可以是批处理引擎（例如 Spark、Presto、Apache Hive）、流处理引擎（例如 Spark、Apache Flink）或机器学习库（例如 Spark MLlib、scikit-learn、R）。

这种灵活性——能够选择最适合当前工作负载的存储系统、开放数据格式和处理引擎——是数据湖比数据库更大的优势。总体而言，对于相同的性能特征，数据湖通常提供比数据库更便宜的解决方案。这一关键优势导致了大数据生态系统的爆炸式增长。在接下来的部分中，我们将讨论如何使用 Apache Spark 在任何存储系统上读写常见文件格式。

## 使用 Apache Spark 读写数据湖

当构建自己的数据湖时，Apache Spark 是其中一种最佳处理引擎，因为它提供了他们需要的所有关键功能：

支持多样化的工作负载

Spark 提供了处理多种工作负载所需的所有必要工具，包括批处理、ETL 操作、使用 Spark SQL 进行 SQL 工作负载、使用结构化流进行流处理（在第八章中讨论）、以及使用 MLlib 进行机器学习（在第十章中讨论），等等。

支持多样化的文件格式

在第四章中，我们详细探讨了 Spark 对非结构化、半结构化和结构化文件格式的内置支持。

支持多样化的文件系统

Spark 支持从支持 Hadoop FileSystem API 的任何存储系统访问数据。由于这个 API 已成为大数据生态系统的事实标准，大多数云和本地存储系统都为其提供了实现——这意味着 Spark 可以读取和写入大多数存储系统。

然而，对于许多文件系统（特别是基于云存储的文件系统，如 AWS S3），您必须配置 Spark 以便以安全的方式访问文件系统。此外，云存储系统通常没有标准文件系统期望的相同文件操作语义（例如，S3 的最终一致性），如果不按照 Spark 的配置进行配置，可能会导致不一致的结果。有关详细信息，请参阅[云集成文档](https://oreil.ly/YncTL)。

## 数据湖的局限性

数据湖并非没有缺陷，其中最严重的是缺乏事务性保证。具体来说，数据湖无法提供 ACID 保证：

原子性和隔离

处理引擎以分布方式写入数据湖中的数据。如果操作失败，就没有机制回滚已经写入的文件，从而可能留下潜在的损坏数据（当并发工作负载修改数据时，提供跨文件的隔离是非常困难的，因此问题进一步恶化）。

一致性

失败写入的缺乏原子性进一步导致读者获取数据的不一致视图。事实上，即使成功写入数据，也很难确保数据质量。例如，数据湖的一个非常常见的问题是意外地以与现有数据不一致的格式和架构写出数据文件。

为了解决数据湖的这些局限性，开发人员采用各种技巧。以下是几个示例：

+   数据湖中的大量数据文件通常根据列的值（例如，一个大型的 Parquet 格式的 Hive 表按日期分区）“分区”到子目录中。为了实现对现有数据的原子修改，通常整个子目录会被重写（即写入临时目录，然后交换引用），以便更新或删除少量记录。

+   数据更新作业（例如，每日 ETL 作业）和数据查询作业（例如，每日报告作业）的调度通常错开，以避免对数据的并发访问及由此引起的任何不一致性。

试图消除这些实际问题的努力导致了新系统的开发，例如湖岸。

# 湖岸：存储解决方案演进的下一个步骤

*湖岸*是一种新范式，它结合了数据湖和数据仓库的最佳元素，用于 OLAP 工作负载。湖岸由一个新的系统设计驱动，提供了类似数据库直接在用于数据湖的低成本可扩展存储上的数据管理功能。更具体地说，它们提供以下功能：

事务支持

类似于数据库，湖仓在并发工作负载下提供 ACID 保证。

模式强制执行和治理

湖仓防止将带有错误模式的数据插入表格，必要时，可以显式演变表格模式以适应不断变化的数据。系统应能够理解数据完整性，并具备强大的治理和审计机制。

支持开放格式中多样化的数据类型

与数据库不同，但类似于数据湖，湖仓可以存储、精炼、分析和访问所有类型的数据，以支持许多新数据应用程序所需的结构化、半结构化或非结构化数据。为了让各种工具能够直接和高效地访问它，数据必须以开放格式存储，并具备标准化的 API 来读取和写入。

支持多样化工作负载

借助各种工具的驱动，使用开放 API 读取数据，湖仓使得多样化工作负载能够在单一仓库中处理数据。打破孤立的数据孤岛（即针对不同数据类别的多个仓库），使开发人员更轻松地构建从传统 SQL 和流式分析到机器学习的多样化和复杂数据解决方案。

支持插入更新和删除操作

像[变更数据捕获(CDC)](https://oreil.ly/eEj_m)和[缓慢变化维度(SCD)](https://oreil.ly/13zll)这样的复杂用例需要对表格中的数据进行持续更新。湖仓允许数据在具有事务保证的情况下进行并发删除和更新。

数据治理

湖仓提供工具，帮助您理解数据完整性，并审计所有数据变更以符合政策要求。

目前有几个开源系统，如 Apache Hudi、Apache Iceberg 和 Delta Lake，可以用来构建具备这些特性的湖仓。在非常高的层次上，这三个项目都受到了著名数据库原理的启发，具有类似的架构。它们都是开放的数据存储格式，具有以下特点：

+   在可扩展的文件系统中以结构化文件格式存储大量数据。

+   维护事务日志记录数据的原子变更时间线（类似数据库）。

+   使用日志定义表数据的版本，并在读写者之间提供快照隔离保证。

+   支持使用 Apache Spark 读写表格数据。

在这些大致描述的框架内，每个项目在 API、性能以及与 Apache Spark 数据源 API 集成程度上都具有独特特征。我们将在接下来探讨它们。请注意，所有这些项目都在快速发展，因此在阅读时可能有些描述已经过时。请参考每个项目的在线文档获取最新信息。

## Apache Hudi

最初由 [Uber Engineering](https://eng.uber.com/hoodie) 创建，[Apache Hudi](https://hudi.apache.org)——Hadoop 更新删除和增量的首字母缩写——是一种专为 key/value 数据的增量 upserts 和删除而设计的数据存储格式。数据存储为列式格式的组合（例如 Parquet 文件）和基于行的格式（例如用于在 Parquet 文件上记录增量变更的 Avro 文件）。除了前面提到的常见功能外，它还支持：

+   快速、可插拔索引的 upsert

+   原子发布数据，支持回滚

+   读取表的增量更改

+   数据恢复的保存点

+   文件大小和布局管理使用统计信息

+   异步压缩行和列式数据

## Apache Iceberg

最初在 [Netflix](https://github.com/Netflix/iceberg) 创建，[Apache Iceberg](https://iceberg.apache.org) 是另一种用于大数据集的开放存储格式。但与专注于 upserting 键/值数据的 Hudi 不同，Iceberg 更专注于通用数据存储，可在单个表中扩展到 PB 级并具有模式演化特性。具体来说，它提供以下附加功能（除了常见功能）：

+   通过添加、删除、更新、重命名和重新排序列、字段和/或嵌套结构来进行模式演化

+   隐藏分区，它在表中为行创建分区值

+   分区演化，根据数据量或查询模式的变化自动执行元数据操作以更新表布局

+   时间旅行，允许您通过 ID 或时间戳查询特定表快照

+   回滚到以前的版本以更正错误

+   可串行化隔离，即使在多个并发写入者之间也是如此

## Delta Lake

[Delta Lake](https://delta.io/) 是由 Linux Foundation 托管的开源项目，由 Apache Spark 的原始创建者构建。与其他类似的项目一样，它是一种提供事务保证并支持模式强制和演化的开放数据存储格式。它还提供几个其他有趣的特性，其中一些是独特的。Delta Lake 支持：

+   使用结构化流源和接收器进行表的流式读取和写入

+   即使在 Java、Scala 和 Python API 中也支持更新、删除和合并（用于 upsert 操作）

+   通过显式更改表模式或在 DataFrame 写入期间将 DataFrame 的模式隐式合并到表的模式中进行模式演化。（事实上，Delta Lake 中的合并操作支持条件更新/插入/删除的高级语法，同时更新所有列等，正如您稍后在本章中将看到的。）

+   时间旅行，允许您通过 ID 或时间戳查询特定表快照

+   回滚到以前的版本以更正错误

+   多个并发写入者之间的可序列化隔离，执行任何 SQL、批处理或流操作

在本章的其余部分，我们将探讨如何使用 Apache Spark 系统来构建提供上述属性的 lakehouse。在这三个系统中，到目前为止 Delta Lake 与 Apache Spark 数据源（用于批处理和流处理工作负载）以及 SQL 操作（例如`MERGE`）的整合最紧密。因此，我们将使用 Delta Lake 进一步探索。

###### 注意

这个项目被称为 Delta Lake，因为它类似于流。溪流流入海洋形成三角洲，这里是所有沉积物积累的地方，因此也是有价值的农作物生长的地方。朱尔斯·S·达姆吉（我们的合著者之一）提出了这个比喻！

# 使用 Apache Spark 和 Delta Lake 构建 Lakehouse

在本节中，我们将快速了解 Delta Lake 和 Apache Spark 如何用于构建 lakehouse。具体来说，我们将探索以下内容：

+   使用 Apache Spark 读写 Delta Lake 表格

+   Delta Lake 如何允许并发批处理和流式写入，并提供 ACID 保证

+   Delta Lake 如何通过在所有写操作上强制执行模式并允许显式模式演变来确保更好的数据质量

+   使用更新、删除和合并操作构建复杂的数据管道，所有操作均保证 ACID 保证

+   审计修改 Delta Lake 表格的操作历史，并通过查询早期版本的表格实现时间旅行

本节中使用的数据是公共[Lending Club Loan Data](https://oreil.ly/P7AR-)的修改版本（Parquet 格式中的列子集）。¹ 它包括了 2012 年至 2017 年间所有资助的贷款记录。每条贷款记录包括申请人提供的申请信息以及当前贷款状态（当前、逾期、已完全还清等）和最新的付款信息。

## 使用 Apache Spark 配置 Delta Lake

您可以通过以下任一方式配置 Apache Spark 以链接到 Delta Lake 库：

设置一个交互式 shell

如果您正在使用 Apache Spark 3.0，可以通过以下命令行参数启动 PySpark 或 Scala shell，并与 Delta Lake 一起使用：

```
--packages io.delta:delta-core_2.12:0.7.0
```

例如：

```
pyspark --packages io.delta:delta-core_2.12:0.7.0
```

如果您正在运行 Spark 2.4，则必须使用 Delta Lake 0.6.0。

使用 Maven 坐标设置一个独立的 Scala/Java 项目

如果要使用 Maven 中央仓库中的 Delta Lake 二进制文件构建项目，可以将以下 Maven 坐标添加到项目依赖项中：

```
  <dependency>
  <groupId>io.delta</groupId>
  <artifactId>delta-core_2.12</artifactId>
  <version>0.7.0</version>
</dependency>
```

同样，如果您正在运行 Spark 2.4，则必须使用 Delta Lake 0.6.0。

###### 注意

参见[Delta Lake 文档](https://oreil.ly/MmlC3)获取最新信息。

## 加载数据到 Delta Lake 表格中

如果您习惯于使用 Apache Spark 和任何结构化数据格式（比如 Parquet）构建数据湖，那么很容易将现有工作负载迁移到使用 Delta Lake 格式。您只需将所有 DataFrame 读写操作更改为使用`format("delta")`而不是`format("parquet")`。让我们试试将一些前述的贷款数据，作为[Parquet 文件](https://oreil.ly/7pP1y)，首先读取这些数据并保存为 Delta Lake 表格：

```
// In Scala
// Configure source data path
val sourcePath = "/databricks-datasets/learning-spark-v2/loans/
 loan-risks.snappy.parquet"

// Configure Delta Lake path
val deltaPath = "/tmp/loans_delta"

// Create the Delta table with the same loans data
spark
  .read
  .format("parquet")
  .load(sourcePath)
  .write
  .format("delta")
  .save(deltaPath)

// Create a view on the data called loans_delta
spark
 .read
 .format("delta")
 .load(deltaPath)
 .createOrReplaceTempView("loans_delta")
```

```
# In Python
# Configure source data path
sourcePath = "/databricks-datasets/learning-spark-v2/loans/
  loan-risks.snappy.parquet"

# Configure Delta Lake path
deltaPath = "/tmp/loans_delta"

# Create the Delta Lake table with the same loans data
(spark.read.format("parquet").load(sourcePath) 
  .write.format("delta").save(deltaPath))

# Create a view on the data called loans_delta
spark.read.format("delta").load(deltaPath).createOrReplaceTempView("loans_delta")
```

现在，我们可以像处理任何其他表格一样轻松读取和探索数据：

```
// In Scala/Python

// Loans row count
spark.sql("SELECT count(*) FROM loans_delta").show()

+--------+
|count(1)|
+--------+
|   14705|
+--------+

// First 5 rows of loans table
spark.sql("SELECT * FROM loans_delta LIMIT 5").show()

+-------+-----------+---------+----------+
|loan_id|funded_amnt|paid_amnt|addr_state|
+-------+-----------+---------+----------+
|      0|       1000|   182.22|        CA|
|      1|       1000|   361.19|        WA|
|      2|       1000|   176.26|        TX|
|      3|       1000|   1000.0|        OK|
|      4|       1000|   249.98|        PA|
+-------+-----------+---------+----------+
```

## 将数据流加载到 Delta Lake 表格中

与静态 DataFrame 一样，您可以通过将格式设置为`"delta"`轻松修改现有的结构化流作业以写入和读取 Delta Lake 表格。假设您有一个名为`newLoanStreamDF`的 DataFrame，其模式与表格相同。您可以如下追加到表格中：

```
// In Scala
import org.apache.spark.sql.streaming._

val newLoanStreamDF = ...   // Streaming DataFrame with new loans data
val checkpointDir = ...     // Directory for streaming checkpoints
val streamingQuery = newLoanStreamDF.writeStream
  .format("delta")
  .option("checkpointLocation", checkpointDir)
  .trigger(Trigger.ProcessingTime("10 seconds"))
  .start(deltaPath)
```

```
# In Python
newLoanStreamDF = ...   # Streaming DataFrame with new loans data
checkpointDir = ...     # Directory for streaming checkpoints
streamingQuery = (newLoanStreamDF.writeStream 
    .format("delta") 
    .option("checkpointLocation", checkpointDir) 
    .trigger(processingTime = "10 seconds") 
    .start(deltaPath))
```

使用此格式，就像任何其他格式一样，结构化流提供端到端的幂等保证。但是，Delta Lake 相比于传统格式（如 JSON、Parquet 或 ORC）具有一些额外的优势：

允许批处理和流作业向同一表格写入

使用其他格式，从结构化流作业写入表格的数据将覆盖表格中的任何现有数据。这是因为表格中维护的元数据用于确保流写入的幂等性，并不考虑其他非流写入。Delta Lake 的高级元数据管理允许同时写入批处理和流数据。

允许多个流作业向同一表格追加数据

与其他格式的元数据相同限制也会阻止多个结构化流查询向同一表格追加数据。Delta Lake 的元数据为每个流查询维护事务信息，从而使任意数量的流查询能够并发写入具有幂等保证的表格。

即使在并发写入的情况下也提供 ACID 保证

与内置格式不同，Delta Lake 允许并发批处理和流操作写入具有 ACID 保证的数据。

## 写入时强制执行模式以防止数据损坏

使用 Spark 管理数据时的常见问题是使用 JSON、Parquet 和 ORC 等常见格式时由于错误格式的数据写入而导致的意外数据损坏。由于这些格式定义了单个文件的数据布局而不是整个表格的布局，因此没有机制防止任何 Spark 作业将具有不同模式的文件写入到现有表中。这意味着对于由多个 Parquet 文件组成的整个表格，不存在一致性保证。

Delta Lake 格式将模式记录为表级元数据。因此，对 Delta Lake 表的所有写操作都可以验证正在写入的数据是否与表的模式兼容。如果不兼容，Spark 将在写入和提交数据之前抛出错误，从而防止此类意外数据损坏。让我们通过尝试写入带有额外列`closed`的一些数据来测试这一点，该列表示贷款是否已终止。请注意，该列在表中不存在：

```
// In Scala
val loanUpdates = Seq(
    (1111111L, 1000, 1000.0, "TX", false), 
    (2222222L, 2000, 0.0, "CA", true))
  .toDF("loan_id", "funded_amnt", "paid_amnt", "addr_state", "closed")

loanUpdates.write.format("delta").mode("append").save(deltaPath)
```

```
# In Python
from pyspark.sql.functions import *

cols = ['loan_id', 'funded_amnt', 'paid_amnt', 'addr_state', 'closed']
items = [
(1111111, 1000, 1000.0, 'TX', True), 
(2222222, 2000, 0.0, 'CA', False)
]

loanUpdates = (spark.createDataFrame(items, cols)
  .withColumn("funded_amnt", col("funded_amnt").cast("int")))
loanUpdates.write.format("delta").mode("append").save(deltaPath)
```

此写入将失败，并显示以下错误消息：

```
org.apache.spark.sql.AnalysisException: A schema mismatch detected when writing 
  to the Delta table (Table ID: 48bfa949-5a09-49ce-96cb-34090ab7d695).
To enable schema migration, please set:
'.option("mergeSchema", "true")'.

Table schema:
root
-- loan_id: long (nullable = true)
-- funded_amnt: integer (nullable = true)
-- paid_amnt: double (nullable = true)
-- addr_state: string (nullable = true)

Data schema:
root
-- loan_id: long (nullable = true)
-- funded_amnt: integer (nullable = true)
-- paid_amnt: double (nullable = true)
-- addr_state: string (nullable = true)
-- closed: boolean (nullable = true)
```

这说明了 Delta Lake 如何阻止不匹配表模式的写入。然而，它也提供了如何使用选项 `mergeSchema` 实际演进表模式的提示，接下来将进行讨论。

## 适应变化数据的演进模式

在我们这个变化不断的世界中，我们可能希望将这个新列添加到表中。可以通过设置选项 `"mergeSchema"` 为 `"true"` 显式地添加这个新列：

```
// In Scala
loanUpdates.write.format("delta").mode("append")
  .option("mergeSchema", "true")
  .save(deltaPath)
```

```
# In Python
(loanUpdates.write.format("delta").mode("append")
  .option("mergeSchema", "true")
  .save(deltaPath))
```

通过这个操作，列 `closed` 将被添加到表模式中，并且新数据将被追加。当读取现有行时，新列的值将被视为 `NULL`。在 Spark 3.0 中，您还可以使用 SQL DDL 命令 `ALTER TABLE` 来添加和修改列。

## 转换现有数据

Delta Lake 支持 `UPDATE`、`DELETE` 和 `MERGE` 等 DML 命令，允许您构建复杂的数据管道。这些命令可以使用 Java、Scala、Python 和 SQL 调用，使用户能够使用他们熟悉的任何 API，无论是使用 DataFrames 还是表。此外，每个数据修改操作都确保 ACID 保证。

让我们通过几个实际用例来探索这一点。

### 更新数据以修复错误

在管理数据时，一个常见的用例是修复数据中的错误。假设在查看数据时，我们意识到所有分配给 `addr_state = 'OR'` 的贷款都应该分配给 `addr_state = 'WA'`。如果贷款表是 Parquet 表，那么要执行这样的更新，我们需要：

1.  将所有未受影响的行复制到一个新表中。

1.  将所有受影响的行复制到一个 DataFrame 中，然后执行数据修改。

1.  将之前提到的 DataFrame 的行插入到新表中。

1.  删除旧表并将新表重命名为旧表名。

在 Spark 3.0 中，直接支持像 `UPDATE`、`DELETE` 和 `MERGE` 这样的 DML SQL 操作，而不是手动执行所有这些步骤，您可以简单地运行 SQL `UPDATE` 命令。然而，对于 Delta Lake 表，用户也可以通过使用 Delta Lake 的编程 API 来运行此操作，如下所示：

```
// In Scala
import io.delta.tables.DeltaTable
import org.apache.spark.sql.functions._

val deltaTable = DeltaTable.forPath(spark, deltaPath)
deltaTable.update(
  col("addr_state") === "OR",
  Map("addr_state" -> lit("WA")))
```

```
# In Python
from delta.tables import *

deltaTable = DeltaTable.forPath(spark, deltaPath)
deltaTable.update("addr_state = 'OR'",  {"addr_state": "'WA'"})
```

### 删除与用户相关的数据

随着类似欧盟[通用数据保护条例（GDPR）](https://oreil.ly/hOdBE)这样的数据保护政策的实施，现在比以往任何时候都更重要能够从所有表中删除用户数据。假设您必须删除所有已完全偿还贷款的数据。使用 Delta Lake，您可以执行以下操作：

```
// In Scala
val deltaTable = DeltaTable.forPath(spark, deltaPath)
deltaTable.delete("funded_amnt >= paid_amnt")
```

```
# In Python
deltaTable = DeltaTable.forPath(spark, deltaPath)
deltaTable.delete("funded_amnt >= paid_amnt")
```

与更新类似，在 Delta Lake 和 Apache Spark 3.0 中，您可以直接在表上运行`DELETE` SQL 命令。

### 使用 merge()向表中插入变更数据

一个常见的用例是变更数据捕获，您必须将 OLTP 表中的行更改复制到另一个表中，以供 OLAP 工作负载使用。继续我们的贷款数据示例，假设我们有另一张新贷款信息表，其中一些是新贷款，另一些是对现有贷款的更新。此外，假设此`changes`表与`loan_delta`表具有相同的架构。您可以使用基于`MERGE` SQL 命令的`DeltaTable.merge()`操作将这些变更插入表中：

```
// In Scala
deltaTable
  .alias("t")
  .merge(loanUpdates.alias("s"), "t.loan_id = s.loan_id")
  .whenMatched.updateAll()
  .whenNotMatched.insertAll()
  .execute()
```

```
# In Python
(deltaTable
  .alias("t")
  .merge(loanUpdates.alias("s"), "t.loan_id = s.loan_id") 
  .whenMatchedUpdateAll() 
  .whenNotMatchedInsertAll() 
  .execute())
```

作为提醒，您可以将此作为 SQL `MERGE`命令在 Spark 3.0 中运行。此外，如果您有这类捕获变更的流，您可以使用 Structured Streaming 查询连续应用这些变更。查询可以从任何流源中的微批次（参见第八章）读取变更，并使用`foreachBatch()`将每个微批次中的变更应用于 Delta Lake 表中。

### 使用仅插入合并去重数据

Delta Lake 中的合并操作支持比 ANSI 标准指定的更多扩展语法，包括以下高级特性：

删除操作

例如，`MERGE ... WHEN MATCHED THEN DELETE`。

子句条件

例如，`MERGE ... WHEN MATCHED AND *<condition>* THEN ...`。

可选操作

所有`MATCHED`和`NOT MATCHED`子句都是可选的。

星号语法

例如，`UPDATE *`和`INSERT *`以将源数据集中匹配列的所有列更新/插入目标表中。等效的 Delta Lake API 是`updateAll()`和`insertAll()`，我们在前一节中看到了。

这使您能够用很少的代码表达更多复杂的用例。例如，假设您希望为`loan_delta`表回填历史数据。但是，一些历史数据可能已经插入了表中，您不希望更新这些记录，因为它们可能包含更为更新的信息。您可以通过`loan_id`进行插入时进行去重，运行以下仅包含`INSERT`操作的合并操作（因为`UPDATE`操作是可选的）：

```
// In Scala
deltaTable
  .alias("t")
  .merge(historicalUpdates.alias("s"), "t.loan_id = s.loan_id")
  .whenNotMatched.insertAll()
  .execute()
```

```
# In Python
(deltaTable
  .alias("t")
  .merge(historicalUpdates.alias("s"), "t.loan_id = s.loan_id") 
  .whenNotMatchedInsertAll() 
  .execute())
```

还有更复杂的用例，例如包括删除和 SCD 表的 CDC，使用扩展合并语法变得简单。请参阅[文档](https://oreil.ly/XBag7)获取更多详细信息和示例。

## 通过操作历史审核数据更改

所有对 Delta Lake 表的更改都记录在表的事务日志中作为提交。当你向 Delta Lake 表或目录写入时，每个操作都会自动进行版本控制。你可以像以下代码片段中所示查询表的操作历史记录：

```
// In Scala/Python
deltaTable.history().show()
```

默认情况下，这将显示一个包含许多版本和大量列的巨大表格。我们可以打印最后三个操作的一些关键列：

```
// In Scala
deltaTable
  .history(3)
  .select("version", "timestamp", "operation", "operationParameters")
  .show(false)
```

```
# In Python
(deltaTable
  .history(3)
  .select("version", "timestamp", "operation", "operationParameters")
  .show(truncate=False))
```

这将生成以下输出：

```
+-------+-----------+---------+-------------------------------------------+
|version|timestamp  |operation|operationParameters                        |
+-------+-----------+---------+-------------------------------------------+
|5      |2020-04-07 |MERGE    |[predicate -> (t.`loan_id` = s.`loan_id`)] |
|4      |2020-04-07 |MERGE    |[predicate -> (t.`loan_id` = s.`loan_id`)] |
|3      |2020-04-07 |DELETE   |predicate -> ["(CAST(`funded_amnt` ...    |
+-------+-----------+---------+-------------------------------------------+
```

注意对审核更改有用的 `operation` 和 `operationParameters`。

## 使用时间旅行查询表的先前快照

你可以使用 `DataFrameReader` 选项 `"versionAsOf"` 和 `"timestampAsOf"` 查询表的以前版本化的快照。以下是几个例子：

```
// In Scala
spark.read
  .format("delta")
  .option("timestampAsOf", "2020-01-01")  // timestamp after table creation
  .load(deltaPath)

spark.read.format("delta")
  .option("versionAsOf", "4")
  .load(deltaPath)
```

```
# In Python
(spark.read
  .format("delta")
  .option("timestampAsOf", "2020-01-01")  # timestamp after table creation
  .load(deltaPath))

(spark.read.format("delta")
  .option("versionAsOf", "4")
  .load(deltaPath))
```

这在各种情况下都很有用，例如：

+   通过重新运行特定表版本上的作业复现机器学习实验和报告

+   比较不同版本之间的数据变化以进行审核

+   通过读取前一个快照作为 DataFrame 并用其覆盖表格来回滚不正确的更改

# 摘要

本章探讨了使用 Apache Spark 构建可靠数据湖的可能性。简而言之，数据库长期以来解决了数据问题，但未能满足现代用例和工作负载的多样化需求。数据湖的建立旨在减轻数据库的一些限制，而 Apache Spark 是构建它们的最佳工具之一。然而，数据湖仍然缺乏数据库提供的一些关键特性（例如 ACID 保证）。Lakehouse 是数据解决方案的下一代，旨在提供数据库和数据湖的最佳功能，并满足多样化用例和工作负载的所有要求。

我们简要探讨了几个开源系统（Apache Hudi 和 Apache Iceberg），可以用来构建 Lakehouse，然后更详细地了解了 Delta Lake，这是一个基于文件的开源存储格式，与 Apache Spark 一起是构建 Lakehouse 的优秀基础模块。正如你所看到的，它提供以下功能：

+   像数据库一样的事务保证和架构管理

+   可伸缩性和开放性，如数据湖

+   支持具有 ACID 保证的并发批处理和流处理工作负载

+   支持使用更新、删除和合并操作对现有数据进行转换，以确保 ACID 保证。

+   支持版本控制、操作历史的审计和查询以前的版本

在下一章中，我们将探讨如何开始使用 Spark 的 MLlib 构建 ML 模型。

^([1) 可以在[此 Excel 文件](https://oreil.ly/Rgtn1)中查看完整的数据视图。
