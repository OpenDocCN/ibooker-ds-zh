# 附录 B：其他 Kafka 工具

Apache Kafka 社区已经创建了一个强大的工具和平台生态系统，使运行和使用 Kafka 的任务变得更加容易。虽然这并不是一个详尽的列表，但这里介绍了一些较受欢迎的工具，以帮助您入门。

# 买方注意

尽管作者与列入此列表的一些公司和项目有所关联，但他们和 O'Reilly 并没有明确支持某个工具。请务必自行研究这些平台和工具是否适合您需要做的工作。

# 全面的平台

一些公司提供了完全集成的平台，用于处理 Apache Kafka。这包括所有组件的托管部署，这样您就可以专注于使用 Kafka，而不是运行它的方式。这可以为资源不足（或者您不想将资源用于学习如何正确操作 Kafka 及其所需的基础设施）的用例提供理想的解决方案。一些还提供工具，如模式管理、REST 接口，有时还提供客户端库支持，以确保组件正确地进行交互。

| **标题** | Confluent Cloud |
| --- | --- |
| **URL** | [*https://www.confluent.io/confluent-cloud*](https://www.confluent.io/confluent-cloud) |
| **描述** | 由一些最初的开发者创建并支持 Kafka 的公司提供了受管理的解决方案，这是非常合适的。Confluent Cloud 将许多必备工具（包括模式管理、客户端、RESTful 接口和监控）合并为一个单一的产品。它在所有三个主要的云平台（AWS、Microsoft Azure 和 Google Cloud Platform）上都可用，并得到了 Confluent 雇佣的核心 Apache Kafka 贡献者的支持。该平台包括的许多组件，如模式注册表和 REST 代理，都可以作为独立的工具使用，受[Confluent 社区许可证](https://oreil.ly/lAFga)的支持，但限制了一些使用情况。|
| **标题** | Aiven |
| **URL** | [*https://aiven.io*](https://aiven.io) |
| **描述** | Aiven 为许多数据平台提供了受管理的解决方案，包括 Kafka。为了支持这一点，它开发了[Karapace](https://karapace.io)，这是一个与 Confluent 的组件 API 兼容的模式注册表和 REST 代理，但在[Apache 2.0 许可证](https://oreil.ly/a96F0)下受支持，不限制使用情况。除了三个主要的云提供商，Aiven 还支持[DigitalOcean](https://www.digitalocean.com)和[UpCloud](https://upcloud.com)。|
| **标题** | CloudKarafka |
| **URL** | [*https://www.cloudkarafka.com*](https://www.cloudkarafka.com) |
| **描述** | CloudKarafka 专注于提供一种受管理的 Kafka 解决方案，并集成了流行的基础设施服务（如 DataDog 或 Splunk）。它支持使用 Confluent 的 Schema Registry 和 REST 代理与其平台，但仅支持 Confluent 在许可证更改之前的 5.0 版本。CloudKarafka 在 AWS 和 Google Cloud Platform 上提供其服务。|
| **标题** | 亚马逊托管的 Apache Kafka 流式处理（Amazon MSK） |
| **URL** | [*https://aws.amazon.com/msk*](https://aws.amazon.com/msk) |
| **描述** | 亚马逊还提供了自己的托管 Kafka 平台，仅在 AWS 上受支持。通过与[AWS Glue](https://oreil.ly/hvjoV)集成提供模式支持，而 REST 代理不受直接支持。亚马逊推广使用社区工具（如 Cruise Control、Burrow 和 Confluent 的 REST 代理），但不直接支持它们。因此，MSK 的集成性比其他提供的要低一些，但仍然可以提供核心 Kafka 集群。|
| **标题** | Azure HDInsight |
| **URL** | [*https://azure.microsoft.com/en-us/services/hdinsight*](https://azure.microsoft.com/en-us/services/hdinsight) |
| **描述** | Microsoft 还为 HDInsight 中的 Kafka 提供了托管平台，该平台还支持 Hadoop、Spark 和其他大数据组件。与 MSK 类似，HDInsight 专注于核心 Kafka 集群，许多其他组件（包括模式注册表和 REST 代理）需要用户提供。一些第三方提供了执行这些部署的模板，但它们不受 Microsoft 支持。 |
| **标题** | Cloudera |
| **URL** | [*https://www.cloudera.com/products/open-source/apache-hadoop/apache-kafka.html*](https://www.cloudera.com/products/open-source/apache-hadoop/apache-kafka.html) |
| **Description** | Cloudera 自早期以来一直是 Kafka 社区的一部分，并将托管 Kafka 作为其整体客户数据平台（CDP）产品的流数据组件。CDP 不仅专注于 Kafka，而且在公共云环境中运行，并提供私有选项。 |

# 集群部署和管理

在托管平台之外运行 Kafka 时，您需要一些辅助工具来正确运行集群。这包括帮助进行供应和部署、平衡数据以及可视化您的集群。

- **Title** - Strimzi

- **URL** - [*https://strimzi.io*](https://strimzi.io)

- **Description** - Strimzi 提供了用于在 Kubernetes 环境中部署 Kafka 集群的 Kubernetes 操作员，使您更容易在云中（无论是公共云还是私有云）启动和运行 Kafka。它还提供了 Strimzi Kafka Bridge，这是一个在[Apache 2.0 许可证](https://oreil.ly/a96F0)下受支持的 REST 代理实现。目前，Strimzi 不支持模式注册表，因为存在许可证方面的顾虑。

- **Title** - AKHQ

- **URL** - [*https://akhq.io*](https://akhq.io)

- **Description** - AKHQ 是用于管理和与 Kafka 集群交互的图形用户界面。它支持配置管理，包括用户和 ACL，并为诸如模式注册表和 Kafka Connect 等组件提供一些支持。它还提供了用于与集群中数据交互的工具，作为控制台工具的替代方案。

- **Title** - JulieOps

- **URL** - [*https://github.com/kafka-ops/julie*](https://github.com/kafka-ops/julie)

- **Description** - JulieOps（前身为 Kafka 拓扑生成器）采用 GitOps 模型提供自动化管理主题和 ACL 的功能。JulieOps 不仅可以查看当前配置的状态，还可以提供声明性配置和随时间变化的主题、模式、ACL 等的变更控制。

- **Title** - Cruise Control

- **URL** - [*https://github.com/linkedin/cruise-control*](https://github.com/linkedin/cruise-control)

- **Description** - Cruise Control 是 LinkedIn 对如何管理数百个集群和数千个代理的答案。这个工具最初是为了自动重新平衡集群中的数据而诞生的，但已经发展到包括异常检测和管理操作，如添加和删除代理。对于任何不止是测试集群的情况，这对于任何 Kafka 操作员来说都是必不可少的。

- **Title** - Conduktor

- **URL** - [*https://www.conduktor.io*](https://www.conduktor.io)

- **Description** - 虽然不是开源的，但 Conduktor 是一个流行的桌面工具，用于管理和与 Kafka 集群交互。它支持许多托管平台（包括 Confluent、Aiven 和 MSK）和许多不同的组件（如 Connect、kSQL 和 Streams）。它还允许您与集群中的数据交互，而不是使用控制台工具。提供用于开发使用的免费许可证，可与单个集群一起使用。

# 监控和数据探索

运行 Kafka 的关键部分是确保您的集群和客户端健康。与许多应用程序一样，Kafka 公开了许多指标和其他遥测数据，但理解它可能是具有挑战性的。许多较大的监控平台（如[Prometheus](https://prometheus.io)）可以轻松地从 Kafka 代理和客户端获取指标。还有许多可用的工具可帮助理解所有数据。

| **标题** | Xinfra 监视器 |
| **URL** | [*https://github.com/linkedin/kafka-monitor*](https://github.com/linkedin/kafka-monitor) |
| **描述** | Xinfra 监视器（前身为 Kafka 监视器）是 LinkedIn 开发的，用于监视 Kafka 集群和代理的可用性。它通过使用一组主题通过集群生成合成数据并测量延迟、可用性和完整性来实现这一点。它是一个有价值的工具，可以测量 Kafka 部署的健康状况，而无需直接与客户端进行交互。 |
| **标题** | Burrow |
| **URL** | [*https://github.com/linkedin/burrow*](https://github.com/linkedin/burrow) |
| **描述** | Burrow 是 LinkedIn 最初创建的另一个工具，它提供了对 Kafka 集群中消费者滞后的全面监控。它可以查看消费者的健康状况，而无需直接与它们进行交互。Burrow 得到社区的积极支持，并拥有自己的[工具生态系统](https://oreil.ly/yNPRQ)来将其与其他组件连接起来。 |
| **标题** | Kafka 仪表板 |
| **URL** | [*https://www.datadoghq.com/dashboards/kafka-dashboard*](https://www.datadoghq.com/dashboards/kafka-dashboard) |
| **描述** | 对于那些使用 DataDog 进行监控的人，它提供了一个出色的 Kafka 仪表板，可以帮助您将 Kafka 集群整合到您的监控堆栈中。它旨在提供对 Kafka 集群的单一视图，简化了许多指标的视图。 |
| **标题** | 流资源浏览器 |
| **URL** | [*https://github.com/bakdata/streams-explorer*](https://github.com/bakdata/streams-explorer) |
| **描述** | Streams Explorer 是一个用于可视化数据在 Kubernetes 部署中的应用程序和连接器流动的工具。虽然它在很大程度上依赖于使用 bakdata 的工具结构化部署，但它可以提供这些应用程序及其指标的易于理解的视图。 |
| **标题** | kcat |
| **URL** | [*https://github.com/edenhill/kafkacat*](https://github.com/edenhill/kafkacat) |
| **描述** | Kcat（前身为 kafkacat）是 Apache Kafka 核心项目中的控制台生产者和消费者的备选方案。它体积小，速度快，用 C 语言编写，因此没有 JVM 开销。它还通过显示集群的元数据输出来支持对集群状态的有限视图。 |

# 客户端库

Apache Kafka 项目为 Java 应用程序提供了客户端库，但一种语言永远不够。市面上有许多 Kafka 客户端的实现，流行的语言如 Python、Go 和 Ruby 都有几种选择。此外，REST 代理（例如 Confluent、Strimzi 或 Karapace）可以涵盖各种用例。以下是一些经得住时间考验的客户端实现。

| **标题** | librdkafka |
| **URL** | [*https://github.com/edenhill/librdkafka*](https://github.com/edenhill/librdkafka) |
| **描述** | librdkafka 是 Kafka 客户端的 C 库实现，被认为是性能最佳的库之一。事实上，Confluent 支持 Go、Python 和.NET 客户端，这些客户端是围绕 librdkafka 创建的包装器。它仅在[两条款的 BSD 许可证](https://oreil.ly/dLoe8)下获得许可，这使得它可以轻松用于任何应用程序。 |
| **标题** | Sarama |
| **URL** | [*https://github.com/Shopify/sarama*](https://github.com/Shopify/sarama) |

Shopify 创建了 Sarama 客户端作为原生的 Golang 实现。它是根据[MIT 许可证](https://oreil.ly/sajdS)发布的。

kafka-python

[*https://github.com/dpkp/kafka-python*](https://github.com/dpkp/kafka-python)

kafka-python 是另一个原生的客户端实现，这次是用 Python 实现的。它是根据[Apache 2.0 许可证](https://oreil.ly/a96F0)发布的。

# 流处理

虽然 Apache Kafka 项目包括 Kafka Streams 用于构建应用程序，但对于从 Kafka 处理数据的流处理来说，并不是唯一的选择。

Samza

[*https://samza.apache.org*](https://samza.apache.org)

Apache Samza 是一个专为 Kafka 设计的流处理框架。虽然它早于 Kafka Streams，但它是由许多相同的人开发的，因此两者共享许多概念。然而，与 Kafka Streams 不同，Samza 在 Yarn 上运行，并为应用程序提供了一个完整的运行框架。

Spark

[*https://spark.apache.org*](https://spark.apache.org)

Spark 是另一个面向数据批处理的 Apache 项目。它通过将流视为快速微批处理来处理流。这意味着延迟略高，但容错性通过重新处理批次来简单处理，并且 Lambda 架构很容易。它还有广泛的社区支持。

Flink

[*https://flink.apache.org*](https://flink.apache.org)

Apache Flink 专门面向流处理，并具有非常低的延迟。与 Samza 一样，它支持 Yarn，但也可以与 Mesos、Kubernetes 或独立集群一起工作。它还支持 Python 和 R，并提供了高级 API。

Beam

[*https://beam.apache.org*](https://beam.apache.org)

Apache Beam 并不直接提供流处理，而是将自己宣传为批处理和流处理的统一编程模型。它利用像 Samza、Spark 和 Flink 这样的平台作为整体处理管道中组件的运行器。
