# 第二十九章：其他分布式实时流处理系统

正如本书所示，流处理对于每个数据导向型企业都是至关重要的技术。在处理流数据的任务中，有许多流处理堆栈可供选择，既有专有的也有开源的。它们在功能、API 和延迟与吞吐量之间的平衡中提供不同的权衡。

遵循“适合工作的正确工具”的原则，应当对每个新项目的要求进行比较和对比，以作出正确的选择。

此外，云的演进重要性不仅仅是作为基础设施提供者，还创建了一种新的服务类别，将系统功能作为托管服务（软件即服务 [SAAS]）提供。

在本章中，我们将简要概述当前维护的最相关的开源流处理器，如 Apache Storm、Apache Flink、Apache Beam 和 Kafka Streams，并概述主要云提供商在流处理领域的提供情况。

# Apache Storm

[Apache Storm](http://storm.apache.org/) 是由 Nathan Marz 在 BackType 创建的开源项目。后来在 Twitter 上使用并于 2011 年开源，由 Java 和 Closure 代码组成。它是一个开源的、分布式的、实时计算系统。它是第一个快速、可扩展且部分容错的“大数据”流引擎，当时被视为“流式处理中的 Hadoop”。凭借这一背景，它还激发了许多流处理系统和引擎，包括 Apache Spark Streaming。Storm 是与编程语言无关的，并保证数据至少会被处理一次，尽管在容错性方面存在边缘案例的限制。

Storm 是专为大规模实时分析和流处理、在线机器学习和持续计算而设计的，其非常灵活的编程模型使其能够胜任各种任务。它是第一个广受欢迎的实时分布式流处理系统。Storm 有其特定的术语，需要先介绍其基础知识，以便能够直观地理解其编程 API 的层次。特别是，编写一个 Storm 任务体现了部署拓扑的概念。但在我们深入讨论拓扑之前，需要了解 Storm 中流的传输内容。

## 处理模型

在 Storm 中，流是元组的流，这是其主要数据结构。Storm 元组是一个具有命名值列表的列表，其中每个值可以是任何类型。元组是动态类型的：字段的类型无需声明。这些无界序列的元组代表了我们感兴趣的流。该流的数据流由拓扑的边表示，我们将定义顶点的确切内容。为了使这些元组具有意义，流中的内容通过模式（schema）进行定义。该模式跟踪在一个图中，这个图就是 Storm 拓扑，它代表计算。在该拓扑中，图的节点大致可以分为两种类型：

*喷口*

喷口是流的来源，也是数据的起源。这是我们表示拓扑的有向图始终开始的地方。喷口的一个例子可以是重播的日志文件，一个消息系统，一个 Kafka 服务器等。该喷口创建元组，这些元组被发送到其他节点。其中一个其他节点可以是一个螺栓。

*螺栓*

一个螺栓处理输入流，可能产生新的输出流。它可以执行任何操作，包括过滤，流连接或累积中间结果。它还可以接收多个流，进行聚合，从数据库读取和写入等操作。特别地，它可以是流系统计算的终点，这意味着没有其他节点消耗该螺栓的输出。在这种情况下，该最终节点被称为汇点（sink）。因此，我们可以说拓扑中的所有汇点都是螺栓，但并非所有螺栓都是汇点。

## Storm 拓扑

在 Storm 中，拓扑结构是由喷口（spouts）和螺栓（bolts）组成的网络，其中最后一层的螺栓是汇点（sinks）。它们是应用逻辑的容器，类似于 Spark 中的作业，但会持续运行。因此，一个 Storm 拓扑可能包括几个日志文件生成器，如 Web 服务器，每个都可以发送到一个分割螺栓，该螺栓通过一些基本的 ETL 规则进行预处理和数据消息化，选择感兴趣的元素。在这两个螺栓中，我们可以进行连接，统计不规则元素，并通过时间戳将它们连接起来，以了解这些值 Web 服务器事件的时间顺序。这些可以发送到一个最终螺栓，将其副作用发送到特定的仪表板，指示可能在分布式 Web 服务中发生的错误和事件，如警报。

## Storm 集群

在这个上下文中，一个 Storm 集群由三个元素管理：拓扑的定义，也就是作业的定义，被传递给一个名为 Nimbus 的调度器，它处理部署拓扑的 Supervisor。Nimbus 守护进程是集群的调度器，负责管理集群上存在的拓扑。这与 YARN API 中的 JobTracker 概念类似。此外，Supervisor 守护进程是一个生成工作者的组件。它类似于 Hadoop 的 TaskTracker 概念，并且由 Supervisor 生成的工作者可以接收 Storm 拓扑中特定元素的实例。

## 与 Spark 比较

当比较 Apache Spark 和 Apache Storm 时，我们可以看到 Spark Streaming 受到 Apache Storm 及其组织的影响，尤其是在将资源管理器推迟到跨池的 Spark 流执行器之中的目的。

Storm 有一个优势，它处理元组是*逐个处理*，而不是按时间索引的*微批处理*。当它们被接收时，它们立即被推送到螺栓和拓扑的直接计算图中的新工作者。另一方面，要管理一个拓扑，我们需要描述我们预期的螺栓复制中的并行性。通过这种方式，我们在直接指定我们希望在图的每个阶段中实现的分布时，会有更多的工作。

对于 Spark Streaming 作业的更简单和直接的*分布*范例是其优势，其中 Spark Streaming（特别是在其动态分配模式下）将尽力以一种对非常高吞吐量有意义的方式部署我们程序的连续阶段。这种范例上的差异经常作为与其他系统比较 Apache Spark 流处理方法的主要亮点，但需要注意的是 Spark 正在进行连续处理。

这也是为什么几个基准测试（[[Chintapallu2015]](app01.xhtml#Chintapalli2015)）表明，尽管 Storm 通常提供比 Spark Streaming 更低的延迟，但总体而言，等效的 Spark Streaming 应用的吞吐量更高。

# Apache Flink

[Apache Flink](https://flink.apache.org/)，最初称为[StratoSphere](http://stratosphere.eu/)（[[Alexandrov2014]](app05.xhtml#Alexandrov2014)），是一种来自柏林工业大学及其附属大学的流处理框架。

Flink（[[Carbone2017]](app05.xhtml#Carbone2017)）是第一个支持无序处理的开源引擎，考虑到 MillWheel 和 Cloud Dataflow 的实现是 Google 的私有产品。它提供了 Java 和 Scala API，使其看起来与 Apache Spark 的 RDD/DStream 功能 API 类似：

```
val counts: DataStream[(String, Int)] = text
      // split up the lines in pairs (2-tuples) containing: (word,1)
      .flatMap(_.toLowerCase.split("\\W+"))
      .filter(_.nonEmpty)
      .map((_, 1))
      // group by the tuple field "0" and sum up tuple field "1"
      .keyBy(0)
      .sum(1)
```

类似 Google Cloud Dataflow（在本章后面提到），它使用*数据流编程模型*。

# 数据流编程

[数据流编程](http://bit.ly/2Wrnt3q) 是一种将计算建模为数据流图的编程类型的概念名称。它与函数式编程密切相关，强调数据的移动，并将程序建模为一系列连接的操作。明确定义的输入和输出通过操作连接在一起，操作之间视为彼此的黑盒子。

这是由 MIT 在 1960 年代的 Jack Dennis 及其研究生首创的。Google 随后将此名称重用于其云编程 API。

## 流优先框架

[Flink](http://bit.ly/2Wrnt3q) 是一种逐条流处理框架，它还提供快照来保护计算免受故障影响，尽管它缺乏同步批处理边界。如今，这个非常完整的框架比结构化流处理提供了更低级别的 API，但如果低延迟是关注点的话，它是对 Spark Streaming 的一个引人注目的替代选择。Flink 在 Scala 和 Java 中提供 API。¹

## 与 Spark 比较

与这些替代框架相比，Apache Spark 保持其主要优势，即紧密集成的高级数据处理 API，批处理和流处理之间的变化最小。

随着结构化流处理的发展，Apache Spark 已经赶上了时间查询代数的丰富功能（事件时间，触发器等），这些功能 Dataflow 拥有而 Spark Streaming 曾经缺乏。然而，结构化流处理保持了与 Apache Spark 中已经确立的批 DataSet 和 Dataframe API 的高度兼容性。

结构化流处理通过无缝扩展 Spark 的 DataSet 和 Dataframe API，添加了流处理功能，这是其主要价值所在：可以在流数据上进行计算，几乎不需要专门的培训，并且认知负荷很小。

这种集成的一个最有趣的方面是，通过 Catalyst 的查询规划器在结构化流处理中运行流数据集查询，从而优化用户查询，并使流计算比使用类似数据流系统编写的计算 less error prone。同时请注意，Flink 有一个类似于 Apache Spark Tungsten 的系统，允许它管理其自己的堆外内存段，利用强大的低级 JIT 优化。([[Hueske2015]](app05.xhtml#Hueske2015), [[Ewen2015]](app05.xhtml#Ewen2015))

最后，请注意，Apache Spark 也是关于调度研究的对象，这表明对于像 Spark 这样的系统，未来将会有更好的低延迟。它可以重复使用跨微批次的调度决策。([[Venkataraman2016]](app01.xhtml#Venkataraman2016))

总结一下，作为生态系统，Apache Spark 在继续流处理性能方面表现出非常强的论点，特别是在与批处理分析交换代码相关的背景下，而 Apache Beam 作为与其他流计算开发方式接口的平台，似乎是一个有趣的平台，用于开发“一次编写，任意集群运行”的这种开发方式。

# Kafka Streams

要在其他处理系统中继续此导览，我们必须提到年轻的 Kafka Streams 库。

[Kafka Streams](https://kafka.apache.org/documentation/streams/)（Kreps2016），于 2016 年推出，是 Apache Kafka 项目内集成的流处理引擎。

Kafka Streams 提供 Java 和 Scala API，我们可以使用这些 API 编写具有流处理功能的客户端应用程序。而 Spark、Storm 和 Flink 是*框架*，它们接受作业定义并在集群上管理其执行，而 Kafka Streams 则是一个库。它作为依赖项包含在应用程序中，并提供 API 供开发人员使用，以增加流处理功能。Kafka Streams 还为名为 KSQL 的流式 SQL 查询语言提供后端支持。

## Kafka Streams 编程模型

Kafka Streams 通过提供由有状态数据存储支持的流视图，利用了流-表二元性。它提出了表是聚合流的观点。这一观点根植于我们在第二部分中看到的 Structured Streaming 模型中的同一基本概念。使用 Kafka Streams，您可以从一次处理一个事件中受益。其处理支持包括分布式连接、聚合和有状态处理。窗口和事件时间处理也可用，以及 Kafka 提供的丰富的分布式处理和容错保证，使用偏移回放（重新处理）。

## 与 Spark 比较

Spark 模型和 Kafka Streams 之间的关键区别在于，Kafka Streams 被用作客户端应用程序范围内的客户端库，而 Spark 是一个分布式框架，负责在集群中的工作协调。在现代应用架构方面，Kafka Streams 应用可以被视为使用 Kafka 作为数据后端的微服务。它们的可伸缩性模型基于副本——运行应用程序的多个副本——并且它绑定于正在消费的主题的分区数。

Kafka Streams 的主要用途是为客户端应用程序“增加”流处理功能，或创建简单的流处理应用程序，这些应用程序使用 Kafka 作为它们的数据源和接收端。

然而，Kafka Streams 的劣势在于没有围绕 Apache Spark 开发的围绕流处理的丰富生态系统，如具有 MLlib 的机器学习能力或与广泛数据源支持的外部交互。此外，Kafka Streams 也不具备与 Spark 提供的批处理库和批处理处理的丰富互动。因此，单纯依赖 Kafka Streams 会难以构想未来复杂的机器学习管道，无法充分利用大量科学计算库的优势。

# 在云上

Spark 的表达性编程模型和先进的分析能力可以在云上使用，包括主要厂商的提供：亚马逊、微软和谷歌。在本节中，我们简要介绍了 Spark 流处理能力在云基础设施上及其与本地云功能的结合方式，以及在相关情况下与云提供商自有的专有流处理系统的比较。

## 亚马逊 Kinesis 在 AWS 上

[亚马逊 Kinesis](https://aws.amazon.com/kinesis) 是亚马逊网络服务（AWS）的流处理传输平台。它具有丰富的语义来定义流数据的生产者和消费者，以及用于与这些流端点创建的流水线连接器。我们在 第十九章 中提到了 Kinesis，在那里我们描述了 Kinesis 与 Spark Streaming 之间的连接器。

Kinesis 和 Structured Streaming 之间有连接器，提供两种方式：

+   Databricks 版本的 Spark 原生提供给用户，可在 AWS 和微软 Azure 云上使用。

+   在 JIRA 下的开源连接器 [Spark-18165](https://issues.apache.org/jira/browse/SPARK-18165)，提供了一种轻松从 Kinesis 流式传输数据的方式。

这些连接器是必要的，因为按设计，Kinesis 除了在 [AWS 分析](https://amzn.to/2QMPPyY) 上覆盖较简单的基于 SQL 的查询外，并未提供全面的流处理范式。因此，Kinesis 的价值在于让客户从经过实战验证的 Kinesis SDK 客户端生成的强大源和汇聚中实现自己的处理。使用 Kinesis，可以利用 AWS 平台的监控和限流工具，获得在 AWS 云上即可用的生产就绪流传输。更多详细信息可以在 [[Amazon2017]](app05.xhtml#Amazon2017) 中找到。

亚马逊 Kinesis 与 Structured Streaming 之间的开源连接器是 Qubole 工程师的贡献，您可以在 [[Georgiadis2018]](app05.xhtml#Georgiadis2018) 找到。该库已在 Spark 2.2 上进行开发和测试，允许 Kinesis 成为 Spark 生态系统的完整成员，让 Spark 用户定义任意复杂的分析处理。

最后，请注意，尽管 Kinesis 连接器用于 Spark Streaming 是基于旧的接收器模型，这带来了一些性能问题。而这个 Structured Streaming 客户端在其实现上更加现代化，但尚未迁移到 Spark 2.3.0 引入的数据源 API 的第 2 版本。Kinesis 是 Spark 生态系统中一个欢迎易于贡献更新其实现质量的区域。

总结一下，AWS 中的 Kinesis 是一种流传输机制，引入了生产和消费流，并将它们连接到特定的端点。但是，它的内置分析能力有限，这使得它与流分析引擎（如 Spark 的流模块）互补。

## Microsoft Azure Stream Analytics

Azure Streaming Analytics（[[Chiu2014]](app05.xhtml#Chiu2014)）是 Microsoft Azure 上的云平台，灵感来源于 DryadLINQ，这是一个 Microsoft 研究项目，用于使用逻辑处理计划编译语言查询，适用于流处理。它提供了一个高级 SQL-like 语言来描述用户查询，同时也可通过实验性 JavaScript 函数让用户定义一些自定义处理。其目标是使用这种类 SQL 语言表达高级的面向时间的流查询。

就这一点而言，它与 Structured Streaming 类似，支持多种时间处理模式。除了通常的分析函数——包括聚合、最后和第一个元素、转换、日期和时间函数——Azure Stream Analytics 还支持窗口聚合和时间连接。

时间连接是 SQL 连接，其中包含对匹配事件的时间约束。在连接时使用的谓词可以允许用户表达，例如，两个连接的事件必须具有按时间延迟有限的时间戳。这种丰富的查询语言得到了 Microsoft 的大力支持，他们尝试在 2016 年左右（[[Chen2016]](app05.xhtml#Chen2016)）在 Spark Streaming 1.4 上重新实现它。

这项工作尚未完成，因此在今天的生产环境中，它在 Azure Streaming Analytics 中的整合还不够完善。Structured Streaming 已经追赶上了这些特性，现在作为其内部连接设施的一部分，也提供了时间连接作为本地特性。

因此，Azure Stream Analytics 曾经在实现复杂的基于时间的查询方面具有优势，但现在提供的本地设施比 Structured Streaming 少，后者除了类 SQL 查询外，在其 Streaming Dataset API 中还提供了丰富的处理能力。

因此，对于在 Microsoft Cloud 上进行高级流处理项目而言，部署 Spark 在 Azure 上似乎是更为健壮的方法。选项包括 HDInsight 管理的 Spark、Azure 上的 Databricks，或使用 Azure 的本地资源配置能力，提供托管的 Kubernetes（AKS）以及虚拟机。

## Apache Beam/Google Cloud Dataflow

现代流处理系统有许多种，包括但不限于 Apache Flink，Apache Spark，Apache Apex，Apache Gearpump，Hadoop MapReduce，JStorm，IBM Streams 和 Apache Samza。Apache Beam 是由谷歌领导的开源项目，旨在管理这个流处理系统行业的存在，同时提供与谷歌云数据流计算引擎的良好集成。让我们解释一下这是如何实现的。

在 2013 年，谷歌拥有另一个名为 MillWheel 的内部云流处理系统[[Akidau2013]](app05.xhtml#Akidau2013)。当时机成熟，要给它一次翻新，并将其与成熟的云服务进行连接，以便向公众开放，MillWheel 就变成了谷歌云数据流（Google Cloud Dataflow）[[Akidau2014]](app05.xhtml#Akidau2014)，在容错性和事件触发领域增加了几个关键的流处理概念。关于此还有更多内容，你可以在[[Akidau2017]](app05.xhtml#Akidau2017)中找到。

但是，当我们已经列出了所有这些其他替代方案之后，为什么要提供另一种选择呢？是否可能在一个 API 下实现一个系统，它在所有这些计算引擎下都可以运行流处理？

那个 API 变成了一个开源项目，Apache Beam，旨在提供一个供流处理使用的单一编程 API，可以连接到我们之前提到的所有流计算引擎中的任何一个，以及 Apex，Gearpump，MapReduce，JStorm，IBM Streams 和 Samza。所有这些计算引擎都作为 Apache Beam 的后端插件（或"runners"）公开，旨在成为流处理的*通用语言*。

例如，要计算 30 分钟窗口的整数索引总和，我们可以使用以下方式：

```
PCollection<KV<String, Integer>> output = input
  .apply(Window
  .into(FixedWindows.of(Duration.standardMinutes(30)))
  .triggering(AfterWatermark.pastEndOfWindow()))
  .apply(Sum.integersPerKey());
```

在这里，我们通过关键字对整数进行求和，使用一个固定的 30 分钟窗口，并在水印通过窗口末尾时触发`sum`输出，这反映了"我们估计窗口已完成"的时刻。

###### 注意

需要注意的一点是，与结构化流式处理不同，触发器和输出与查询本身并不独立。在 Dataflow（以及 Beam 中），窗口还必须选择输出模式和触发器，这使标识符的语义与其运行时特性混淆在一起。在结构化流式处理中，即使对于逻辑上不使用窗口的查询，也可以拥有这些，从而使概念的分离更加简单。

Apache Beam 提供了 Java SDK 中的 API，以及 Python SDK 和一个更小但仍然显著的 Go SDK，以及一些 SQL 原始查询。它允许一种单一的语言支持一系列在流处理中通用可适应的概念，并希望可以运行在几乎每个流计算引擎上。除了我们在之前章节中看到的经典聚合操作，例如基于窗口的分片和事件时间处理，beam API 还允许在处理时间中包含触发器，包括计数触发器，允许延迟，以及事件时间触发器。

但 Beam 的亮点在于提供不同流处理系统之间的可移植性，集中一个单一的 API（遗憾的是没有 Scala SDK）用于几乎可以在任何地方运行的流处理程序。这是令人兴奋的，但请注意，当在特定计算引擎下执行时，它只具备此计算引擎及其“运行器”插件在实现 Apache Beam API 全部功能方面的能力。

特别是，Apache Spark 的计算引擎暴露了 Spark Streaming 的能力，而不是 Structured Streaming 的能力。截至本文撰写时，它尚未完全实现有状态流或任何事件时间处理，因为这些功能在仅限于 Spark Streaming 中或者“运行器”插件尚未跟上 Spark Streaming 自身变化的情况下受到限制。因此，用 Apache Beam 表达您的程序通常是一个稍微落后于 Structured Streaming 表达能力的游戏，同时 Apache Beam 的 Spark 运行器正在追赶。

当然，Spark 生态系统仅通过与其他相关流处理项目的协作才变得更强大，因此我们当然鼓励您这样做，以帮助为 [Apache Beam](https://github.com/apache/beam) 的 Spark 运行器贡献力量，以便使用 Beam API 的项目能够从 Spark 流处理引擎的效率提升中受益。

总之，Apache Beam 是一个旨在提供非常表现力强大的流处理模型的开源项目。它是一个在 Google Cloud Dataflow 中高效实现的 API，允许您在 Google Cloud 上运行此类程序以及众多相关流处理系统，但需要注意它们并非都具备相同的能力。建议参考 [Apache Beam 能力矩阵](https://beam.apache.org/documentation/runners/capability-matrix/)，了解差异概览。

但请注意，Google Cloud 也允许在节点或 Kubernetes 上运行本机 Apache Spark，在实践中，如果您知道将在支持轻松部署 Apache Spark 的系统上运行程序，则可能不需要切换到 Beam API。如果需要支持 Google Cloud 和其他流处理系统作为部署系统，Apache Beam 可能是个不错的选择。

¹ 为了保持 Scala 和 Java API 之间的一定一致性，一些允许在 Scala 中表达高级表现力的特性已从批处理和流处理的标准 API 中剔除。如果您希望享受完整的 Scala 经验，可以选择通过隐式转换选择扩展功能。
