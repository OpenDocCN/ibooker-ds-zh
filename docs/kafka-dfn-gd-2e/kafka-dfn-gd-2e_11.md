# 第九章：构建数据管道

当人们讨论使用 Apache Kafka 构建数据管道时，他们通常指的是一些用例。第一种是构建一个数据管道，其中 Apache Kafka 是两个端点之一，例如，将数据从 Kafka 传输到 S3，或者将数据从 MongoDB 传输到 Kafka。第二种用例涉及在两个不同系统之间构建管道，但使用 Kafka 作为中间件。一个例子是通过首先将数据从 Twitter 发送到 Kafka，然后从 Kafka 发送到 Elasticsearch，从而将数据从 Twitter 传输到 Elasticsearch。

当我们在 Apache Kafka 的 0.9 版本中添加 Kafka Connect 时，是因为我们看到 Kafka 在 LinkedIn 和其他大型组织中都被用于这两种用例。我们注意到将 Kafka 集成到数据管道中存在特定的挑战，每个组织都必须解决这些挑战，因此决定向 Kafka 添加 API，以解决其中一些挑战，而不是强迫每个组织从头开始解决这些挑战。

Kafka 为数据管道提供的主要价值在于其作为管道中各个阶段之间的非常大的、可靠的缓冲区。这有效地解耦了管道内的数据生产者和消费者，并允许在多个目标应用程序和系统中使用相同数据源的数据，这些目标应用程序和系统具有不同的及时性和可用性要求。这种解耦，加上可靠性、安全性和效率，使 Kafka 非常适合大多数数据管道。

# 将数据集成放入背景中

一些组织认为 Kafka 是管道的*端点*。他们关注的问题是“我如何将数据从 Kafka 传输到 Elastic？”这是一个值得问的问题，特别是如果你需要 Elastic 中的数据，而它目前在 Kafka 中，我们将看看如何做到这一点。但我们将从在至少包括两个（可能更多）不是 Kafka 本身的端点的更大背景中开始讨论 Kafka 的使用。我们鼓励面临数据集成问题的任何人考虑更大的背景，而不仅仅关注于即时端点。专注于短期集成是导致复杂且昂贵的数据集成混乱的原因。

在本章中，我们将讨论构建数据管道时需要考虑的一些常见问题。这些挑战并不特定于 Kafka，而是一般的数据集成问题。尽管如此，我们将展示为什么 Kafka 非常适合数据集成用例，以及它如何解决许多这些挑战。我们将讨论 Kafka Connect API 与普通的生产者和消费者客户端的不同之处，以及何时应该使用每种客户端类型。然后我们将深入讨论 Kafka Connect 的一些细节。虽然本章不涉及 Kafka Connect 的全面讨论，但我们将展示基本用法的示例，以帮助您入门，并指导您在哪里学习更多。最后，我们将讨论其他数据集成系统以及它们如何与 Kafka 集成。

# 构建数据管道时的考虑因素

虽然我们不会在这里详细讨论构建数据管道的所有细节，但我们想强调在设计软件架构时需要考虑的一些最重要的事情，目的是集成多个系统。

## 及时性

一些系统期望它们的数据一天一次以大批量到达；其他系统期望数据在生成后的几毫秒内到达。大多数数据管道都处于这两个极端之间的某个位置。良好的数据集成系统可以支持不同管道的不同及时性要求，并且在业务需求变化时也可以更容易地迁移不同的时间表。Kafka 作为一个具有可扩展和可靠存储的流数据平台，可以用于支持从几乎实时的管道到每日批处理的任何需求。生产者可以根据需要频繁或不频繁地写入 Kafka，消费者也可以在事件到达时读取和传递最新的事件。或者消费者可以批量处理：每小时运行一次，连接到 Kafka，并读取前一个小时积累的事件。

在这种情况下，看待 Kafka 的一个有用方式是它充当了一个巨大的缓冲区，解耦了生产者和消费者之间的时间敏感性要求。生产者可以实时写入事件，而消费者可以处理事件批次，反之亦然。这也使得施加反压变得微不足道——Kafka 本身对生产者施加反压（在需要时延迟确认），因为消费速率完全由消费者驱动。

## 可靠性

我们希望避免单点故障，并允许从各种故障事件中快速且自动地恢复。数据管道通常是数据到达业务关键系统的方式；故障超过几秒钟可能会带来巨大的破坏，特别是当时效性要求更接近光谱的几毫秒端时。可靠性的另一个重要考虑因素是交付保证——一些系统可以承受数据丢失，但大多数情况下都需要至少一次交付，这意味着源系统的每个事件都将到达其目的地，但有时重试会导致重复。通常，甚至需要恰好一次的交付——源系统的每个事件都将无法丢失或重复地到达目的地。

我们在第七章中深入讨论了 Kafka 的可用性和可靠性保证。正如我们所讨论的，Kafka 可以单独提供至少一次的保证，并且当与具有事务模型或唯一键的外部数据存储结合时，可以提供恰好一次的保证。由于许多终点是提供恰好一次交付语义的数据存储，基于 Kafka 的管道通常可以实现恰好一次的交付。值得强调的是，Kafka 的 Connect API 通过提供与处理偏移量时与外部系统集成的 API，使连接器更容易构建端到端的恰好一次管道。事实上，许多可用的开源连接器支持恰好一次的交付。

## 高吞吐量和变化吞吐量

我们正在构建的数据管道应该能够扩展到非常高的吞吐量，这在现代数据系统中经常需要。更重要的是，如果吞吐量突然增加，它们应该能够适应。

有了 Kafka 作为生产者和消费者之间的缓冲区，我们不再需要将消费者的吞吐量与生产者的吞吐量耦合在一起。我们也不再需要实现复杂的反压机制，因为如果生产者的吞吐量超过消费者的吞吐量，数据将在 Kafka 中积累，直到消费者赶上。Kafka 通过独立添加消费者或生产者来扩展的能力，使我们能够动态地独立地扩展管道的任一侧，以满足不断变化的需求。

Kafka 是一个高吞吐量的分布式系统，即使在较小的集群上，也能每秒处理数百兆字节的数据，因此我们不必担心我们的管道在需求增长时无法扩展。此外，Kafka Connect API 专注于并行化工作，可以根据系统要求在单个节点上进行工作，也可以进行扩展。我们将在接下来的章节中描述平台如何允许数据源和目标在多个执行线程中分割工作，并在单台机器上运行时利用可用的 CPU 资源。

Kafka 还支持多种类型的压缩，允许用户和管理员在吞吐量要求增加时控制网络和存储资源的使用。

## 数据格式

数据管道中最重要的考虑之一是协调不同的数据格式和数据类型。不同数据库和其他存储系统支持的数据类型各不相同。您可能会将 XML 和关系数据加载到 Kafka 中，在 Kafka 中使用 Avro，然后在将数据写入 Elasticsearch 时需要将数据转换为 JSON，在将数据写入 HDFS 时需要将数据转换为 Parquet，在将数据写入 S3 时需要将数据转换为 CSV。

Kafka 本身和 Connect API 在数据格式方面完全是不可知的。正如我们在前几章中看到的，生产者和消费者可以使用任何序列化器来表示适合您的任何格式的数据。Kafka Connect 有自己的内存对象，包括数据类型和模式，但正如我们将很快讨论的那样，它允许可插拔的转换器来允许以任何格式存储这些记录。这意味着无论您使用 Kafka 的哪种数据格式，它都不会限制您选择的连接器。

许多来源和目标都有模式；我们可以从数据源中读取模式，存储它，并用它来验证兼容性，甚至在目标数据库中更新模式。一个经典的例子是从 MySQL 到 Snowflake 的数据管道。如果有人在 MySQL 中添加了一列，一个优秀的管道将确保该列也被添加到 Snowflake 中，因为我们正在向其中加载新数据。

此外，在将数据从 Kafka 写入外部系统时，接收连接器负责将数据写入外部系统的格式。一些连接器选择使此格式可插拔。例如，S3 连接器允许在 Avro 和 Parquet 格式之间进行选择。

支持不同类型的数据是不够的。通用数据集成框架还应处理各种来源和目标之间的行为差异。例如，Syslog 是一个推送数据的来源，而关系数据库需要框架从中提取数据。HDFS 是追加写入的，我们只能向其写入数据，而大多数系统允许我们追加数据和更新现有记录。

## 转换

转换比其他要求更有争议。通常有两种构建数据管道的方法：ETL 和 ELT。ETL 代表*提取-转换-加载*，意味着数据管道负责在数据通过时对数据进行修改。它被认为可以节省时间和存储空间，因为您不需要存储数据，修改数据，然后再次存储数据。根据转换的不同，这种好处有时是真实的，但有时会将计算和存储的负担转移到数据管道本身，这可能是不可取的。这种方法的主要缺点是，在管道中对数据进行的转换可能会限制希望在管道下游进一步处理数据的人的手段。如果在 MongoDB 和 MySQL 之间构建管道的人决定过滤某些事件或从记录中删除字段，那么所有访问 MySQL 中数据的用户和应用程序只能访问部分数据。如果他们需要访问缺失的字段，就需要重建管道，并且历史数据需要重新处理（假设可用）。

ELT 代表*提取-加载-转换*，意味着数据管道只进行最小的转换（主要是数据类型转换），目标是确保到达目标系统的数据尽可能与源数据相似。在这些系统中，目标系统收集“原始数据”，所有必需的处理都在目标系统中完成。这里的好处是系统为目标系统的用户提供了最大的灵活性，因为他们可以访问所有数据。这些系统也往往更容易进行故障排除，因为所有数据处理都限制在一个系统中，而不是在管道和其他应用程序之间分开。缺点是转换会在目标系统消耗 CPU 和存储资源。在某些情况下，这些系统是昂贵的，因此有强烈的动机尽可能将计算从这些系统中移出。

Kafka Connect 包括单条消息转换功能，它在从源到 Kafka 或从 Kafka 到目标系统的复制过程中转换记录。这包括将消息路由到不同的主题，过滤消息，更改数据类型，删除特定字段等。通常使用 Kafka Streams 进行涉及连接和聚合的更复杂的转换，我们将在单独的章节中详细探讨这些内容。

###### 警告

在使用 Kafka 构建 ETL 系统时，请记住，Kafka 允许您构建一对多的管道，其中源数据只写入 Kafka 一次，然后被多个应用程序消费，并写入多个目标系统。预期会进行一些预处理和清理，例如标准化时间戳和数据类型，添加谱系，以及可能删除个人信息 - 这些转换将使所有数据的消费者受益。但是不要在摄取时过早地清理和优化数据，因为它可能在其他地方需要不太精细的数据。

## 安全

安全始终应该是一个关注点。在数据管道方面，主要的安全问题通常是：

+   谁可以访问被摄入到 Kafka 中的数据？

+   我们能确保通过管道传输的数据是加密的吗？这主要是对跨数据中心边界的数据管道的关注。

+   谁有权对管道进行修改？

+   如果数据管道需要从受访控制的位置读取或写入，它能够正确进行身份验证吗？

+   我们处理的个人可识别信息（PII）是否符合有关其存储、访问和使用的法律和法规？

Kafka 允许在数据传输过程中对数据进行加密，从数据源到 Kafka，再从 Kafka 到目标系统。它还支持身份验证（通过 SASL）和授权，因此您可以确保敏感信息不会被未经授权的人传输到不太安全的系统中。Kafka 还提供审计日志以跟踪访问 - 无权和有权的访问。通过一些额外的编码，还可以跟踪每个主题中事件的来源和修改者，以便为每条记录提供完整的谱系。

Kafka 安全性在第十一章中有详细讨论。但是，Kafka Connect 及其连接器需要能够连接到外部数据系统，并对外部数据系统的连接进行身份验证，连接器的配置将包括用于与外部数据系统进行身份验证的凭据。

这些天不建议将凭据存储在配置文件中，因为这意味着配置文件必须小心处理并且受到限制的访问。一个常见的解决方案是使用外部秘密管理系统，比如[HashiCorp Vault](https://www.vaultproject.io)。Kafka Connect 包括对[外部秘密配置](https://oreil.ly/5eVRU)的支持。Apache Kafka 只包括允许引入可插拔外部配置提供程序的框架，一个从文件中读取配置的示例提供程序，还有[社区开发的外部配置提供程序](https://oreil.ly/ovntG)，可以与 Vault、AWS 和 Azure 集成。

## 故障处理

假设所有数据始终完美是危险的。提前规划故障处理是很重要的。我们能防止有错误的记录进入管道吗？我们能从无法解析的记录中恢复吗？坏记录可以被修复（也许由人类）并重新处理吗？如果坏事件看起来和正常事件完全一样，而你只在几天后发现了问题呢？

由于 Kafka 可以配置为长时间存储所有事件，因此可以在需要时回溯并从错误中恢复。这也允许重放存储在 Kafka 中的事件到目标系统，如果它们丢失了。

## 耦合和灵活性

数据管道实现的一个理想特征是解耦数据源和数据目标。意外耦合可能发生的多种方式：

特别的管道

一些公司最终为他们想要连接的每一对应用程序构建自定义管道。例如，他们使用 Logstash 将日志转储到 Elasticsearch，使用 Flume 将日志转储到 HDFS，使用 Oracle GoldenGate 从 Oracle 获取数据到 HDFS，使用 Informatica 从 MySQL 和 XML 获取数据到 Oracle 等。这将数据管道紧密耦合到特定的端点，并创建了一堆集成点，需要大量的部署、维护和监控工作。这也意味着公司采用的每个新系统都将需要构建额外的管道，增加了采用新技术的成本，并抑制了创新。

元数据丢失

如果数据管道不保留模式元数据并且不允许模式演变，你最终会将产生数据的软件和目标使用数据的软件紧密耦合在一起。没有模式信息，两个软件产品都需要包含如何解析数据和解释数据的信息。如果数据从 Oracle 流向 HDFS，而 DBA 在 Oracle 中添加了一个新字段而没有保留模式信息并允许模式演变，那么从 HDFS 读取数据的每个应用程序都会崩溃，或者所有开发人员都需要同时升级他们的应用程序。这两种选择都不够灵活。通过管道支持模式演变，每个团队都可以以自己的速度修改他们的应用程序，而不必担心以后会出现问题。

极端处理

正如我们在讨论数据转换时提到的，一些数据处理是数据管道固有的。毕竟，我们正在将数据在不同的系统之间移动，不同的数据格式是有意义的，并且支持不同的用例。然而，过多的处理会将所有下游系统都与构建管道时所做的决定联系起来，包括保留哪些字段，如何聚合数据等。这经常导致管道的不断变化，因为下游应用的需求变化，这不够灵活、高效或安全。更灵活的方式是尽可能保留原始数据，并允许下游应用，包括 Kafka Streams 应用，自行决定数据处理和聚合。

# 何时使用 Kafka Connect 与生产者和消费者

在写入 Kafka 或从 Kafka 读取时，您可以选择使用传统的生产者和消费者客户端，如第三章和第四章中所述，或者使用 Kafka Connect API 和连接器，我们将在接下来的章节中描述。在我们开始深入了解 Kafka Connect 的细节之前，您可能已经在想，“我应该在什么时候使用哪个呢？”

正如我们所见，Kafka 客户端是嵌入在您自己的应用程序中的客户端。它允许您的应用程序将数据写入 Kafka 或从 Kafka 读取数据。当您可以修改要将应用程序连接到的应用程序的代码，并且希望将数据推送到 Kafka 或从 Kafka 拉取数据时，请使用 Kafka 客户端。

您将使用 Connect 将 Kafka 连接到您没有编写并且无法或不会修改其代码或 API 的数据存储。Connect 将用于将数据从外部数据存储拉取到 Kafka，或者将数据从 Kafka 推送到外部存储。要使用 Kafka Connect，您需要连接到要连接的数据存储的连接器，如今这些连接器已经很丰富。这意味着在实践中，Kafka Connect 的用户只需要编写配置文件。

如果您需要将 Kafka 连接到数据存储，并且尚不存在连接器，您可以选择使用 Kafka 客户端或 Connect API 编写应用程序。推荐使用 Connect，因为它提供了诸如配置管理、偏移存储、并行化、错误处理、支持不同数据类型和标准管理 REST API 等开箱即用的功能。编写一个将 Kafka 连接到数据存储的小应用听起来很简单，但实际上需要处理许多关于数据类型和配置的细节，使任务变得复杂。此外，您还需要维护这个管道应用并对其进行文档化，您的团队成员需要学习如何使用它。Kafka Connect 是 Kafka 生态系统的标准部分，它为您处理了大部分工作，使您能够专注于在外部存储之间传输数据。

# Kafka Connect

Kafka Connect 是 Apache Kafka 的一部分，提供了一种可扩展和可靠的方式在 Kafka 和其他数据存储之间复制数据。它提供了 API 和运行时来开发和运行*连接器插件*——Kafka Connect 执行的库，负责移动数据。Kafka Connect 作为一组*worker 进程*运行。您可以在工作节点上安装连接器插件，然后使用 REST API 配置和管理*连接器*，这些连接器使用特定配置运行。*连接器*启动额外的*任务*以并行移动大量数据，并更有效地利用工作节点上的可用资源。源连接器任务只需要从源系统读取数据并向工作进程提供 Connect 数据对象。汇连接器任务从工作进程获取连接器数据对象，并负责将其写入目标数据系统。Kafka Connect 使用*转换器*来支持以不同格式存储这些数据对象——JSON 格式支持是 Apache Kafka 的一部分，Confluent Schema Registry 提供了 Avro、Protobuf 和 JSON Schema 转换器。这使用户可以选择在 Kafka 中存储数据的格式，而不受其使用的连接器的影响，以及如何处理数据的模式（如果有）。

本章无法涵盖 Kafka Connect 及其众多连接器的所有细节。这本身就可以填满一整本书。然而，我们将概述 Kafka Connect 及其使用方法，并指向其他参考资源。

## 运行 Kafka Connect

Kafka Connect 随 Apache Kafka 一起提供，因此无需单独安装。对于生产使用，特别是如果您计划使用 Connect 来移动大量数据或运行许多连接器，您应该将 Connect 运行在与 Kafka 代理不同的服务器上。在这种情况下，在所有机器上安装 Apache Kafka，并在一些服务器上启动代理，然后在其他服务器上启动 Connect。

启动 Connect worker 与启动代理非常相似-您使用属性文件调用启动脚本：

```java
 bin/connect-distributed.sh config/connect-distributed.properties
```

Connect worker 的一些关键配置：

`bootstrap.servers`

Kafka Connect 将与之一起工作的 Kafka 代理的列表。`连接器`将把它们的数据传输到这些代理中的一个或多个。您不需要指定集群中的每个代理，但建议至少指定三个。

`group.id`

具有相同组 ID 的所有 worker 都属于同一个 Connect 集群。在集群上启动的连接器将在任何 worker 上运行，其任务也将在任何 worker 上运行。

`plugin.path`

Kafka Connect 使用可插拔架构，其中连接器、转换器、转换和秘密提供者可以被下载并添加到平台上。为了做到这一点，Kafka Connect 必须能够找到并加载这些插件。

我们可以配置一个或多个目录作为连接器及其依赖项的位置。例如，我们可以配置`plugin.path=/opt/connectors,/home/gwenshap/connectors`。在这些目录中的一个中，我们通常会为每个连接器创建一个子目录，因此在前面的示例中，我们将创建`/opt/connectors/jdbc`和`/opt/​con⁠nec⁠tors/elastic`。在每个子目录中，我们将放置连接器 jar 本身及其所有依赖项。如果连接器作为`uberJar`发货并且没有依赖项，它可以直接放置在`plugin.path`中，不需要子目录。但请注意，将依赖项放在顶级路径将不起作用。

另一种方法是将连接器及其所有依赖项添加到 Kafka Connect 类路径中，但这并不推荐，如果您使用一个带有与 Kafka 的依赖项冲突的依赖项的连接器，可能会引入错误。推荐的方法是使用`plugin.path`配置。

`key.converter`和`value.converter`

Connect 可以处理存储在 Kafka 中的多种数据格式。这两个配置设置了将存储在 Kafka 中的消息的键和值部分的转换器。默认值是使用 Apache Kafka 中包含的`JSONConverter`的 JSON 格式。这些配置也可以设置为`AvroConverter`、`ProtobufConverter`或`JscoSchemaConverter`，这些都是 Confluent Schema Registry 的一部分。

一些转换器包括特定于转换器的配置参数。您需要使用`key.converter.`或`value.converter.`作为前缀来设置这些参数，具体取决于您是要将其应用于键还是值转换器。例如，JSON 消息可以包含模式或无模式。为了支持任一种情况，您可以分别设置`key.converter.schemas.enable=true`或`false`。相同的配置也可以用于值转换器，通过将`value.converter.schemas.enable`设置为`true`或`false`。Avro 消息也包含模式，但您需要使用`key.converter.schema.registry.url`和`value.converter.schema.​regis⁠try.url`来配置模式注册表的位置。

`rest.host.name`和`rest.port`

连接器通常通过 Kafka Connect 的 REST API 进行配置和监视。您可以为 REST API 配置特定的端口。

一旦工作人员上岗并且您有一个集群，请通过检查 REST API 确保其正常运行：

```java
$ curl http://localhost:8083/
{"version":"3.0.0-SNAPSHOT","commit":"fae0784ce32a448a","kafka_cluster_id":"pfkYIGZQSXm8RylvACQHdg"}%
```

访问基本 REST URI 应返回您正在运行的当前版本。我们正在运行 Kafka 3.0.0 的快照（预发布）。我们还可以检查可用的连接器插件：

```java
$ curl http://localhost:8083/connector-plugins

[
  {
    "class": "org.apache.kafka.connect.file.FileStreamSinkConnector",
    "type": "sink",
    "version": "3.0.0-SNAPSHOT"
  },
  {
    "class": "org.apache.kafka.connect.file.FileStreamSourceConnector",
    "type": "source",
    "version": "3.0.0-SNAPSHOT"
  },
  {
    "class": "org.apache.kafka.connect.mirror.MirrorCheckpointConnector",
    "type": "source",
    "version": "1"
  },
  {
    "class": "org.apache.kafka.connect.mirror.MirrorHeartbeatConnector",
    "type": "source",
    "version": "1"
  },
  {
    "class": "org.apache.kafka.connect.mirror.MirrorSourceConnector",
    "type": "source",
    "version": "1"
  }
]
```

我们正在运行纯粹的 Apache Kafka，因此唯一可用的连接器插件是文件源、文件接收器，以及 MirrorMaker 2.0 的一部分连接器。

让我们看看如何配置和使用这些示例连接器，然后我们将深入更高级的示例，这些示例需要设置外部数据系统来连接。

# 独立模式

请注意，Kafka Connect 还有一个独立模式。它类似于分布式模式——你只需要运行`bin/connect-standalone.sh`而不是`bin/connect-distributed.sh`。您还可以通过命令行传递连接器配置文件，而不是通过 REST API。在这种模式下，所有的连接器和任务都在一个独立的工作节点上运行。它用于需要在特定机器上运行连接器和任务的情况（例如，`syslog`连接器监听一个端口，所以你需要知道它在哪些机器上运行）。

## 连接器示例：文件源和文件接收器

这个例子将使用 Apache Kafka 的文件连接器和 JSON 转换器。要跟着做，请确保您的 ZooKeeper 和 Kafka 已经运行起来。

首先，让我们运行一个分布式的 Connect 工作节点。在真实的生产环境中，您至少需要运行两到三个这样的节点，以提供高可用性。在这个例子中，我们只会启动一个：

```java
bin/connect-distributed.sh config/connect-distributed.properties &
```

现在是时候启动一个文件源了。举个例子，我们将配置它来读取 Kafka 配置文件——基本上是将 Kafka 的配置导入到一个 Kafka 主题中：

```java
echo '{"name":"load-kafka-config", "config":{"connector.class":
"FileStreamSource","file":"config/server.properties","topic":
"kafka-config-topic"}}' | curl -X POST -d @- http://localhost:8083/connectors
-H "Content-Type: application/json"

{
  "name": "load-kafka-config",
  "config": {
    "connector.class": "FileStreamSource",
    "file": "config/server.properties",
    "topic": "kafka-config-topic",
    "name": "load-kafka-config"
  },
  "tasks": [
    {
      "connector": "load-kafka-config",
      "task": 0
    }
  ],
  "type": "source"
}
```

要创建一个连接器，我们编写了一个 JSON，其中包括一个连接器名称`load-kafka-config`，以及一个连接器配置映射，其中包括连接器类、我们要加载的文件和我们要将文件加载到的主题。

让我们使用 Kafka 控制台消费者来检查我们是否已经将配置加载到一个主题中：

```java
gwen$ bin/kafka-console-consumer.sh --bootstrap-server=localhost:9092
--topic kafka-config-topic --from-beginning
```

如果一切顺利，你应该会看到类似以下的内容：

```java
{"schema":{"type":"string","optional":false},"payload":"# Licensed to the Apache Software Foundation (ASF) under one or more"}

<more stuff here>

{"schema":{"type":"string","optional":false},"payload":"############################# Server Basics #############################"}
{"schema":{"type":"string","optional":false},"payload":""}
{"schema":{"type":"string","optional":false},"payload":"# The id of the broker. This must be set to a unique integer for each broker."}
{"schema":{"type":"string","optional":false},"payload":"broker.id=0"}
{"schema":{"type":"string","optional":false},"payload":""}

<more stuff here>
```

这实际上是*config/server.properties*文件的内容，因为它被逐行转换为 JSON 并放置在`kafka-config-topic`中。请注意，默认情况下，JSON 转换器在每条记录中放置一个模式。在这种特定情况下，模式非常简单——只有一个名为`payload`的列，类型为`string`，每条记录中包含一个文件的单行。

现在让我们使用文件接收器转换器将该主题的内容转储到一个文件中。生成的文件应该与原始的*server.properties*文件完全相同，因为 JSON 转换器将 JSON 记录转换回简单的文本行：

```java
echo '{"name":"dump-kafka-config", "config":{"connector.class":"FileStreamSink","file":"copy-of-server-properties","topics":"kafka-config-topic"}}' | curl -X POST -d @- http://localhost:8083/connectors --header "content-Type:application/json"

{"name":"dump-kafka-config","config":{"connector.class":"FileStreamSink","file":"copy-of-server-properties","topics":"kafka-config-topic","name":"dump-kafka-config"},"tasks":[]}
```

注意源配置的变化：我们现在使用的类是`FileStreamSink`而不是`FileStreamSource`。我们仍然有一个文件属性，但现在它指的是目标文件而不是记录的源，而且不再指定*topic*，而是指定*topics*。注意复数形式——你可以用 sink 将多个主题写入一个文件，而源只允许写入一个主题。

如果一切顺利，你应该有一个名为*copy-of-server-properties*的文件，它与我们用来填充`kafka-config-topic`的*config/server.properties*完全相同。

要删除一个连接器，您可以运行：

```java
curl -X DELETE http://localhost:8083/connectors/dump-kafka-config
```

###### 警告

这个例子使用了 FileStream 连接器，因为它们简单且内置于 Kafka 中，允许您在安装 Kafka 之外创建您的第一个管道。这些不应该用于实际的生产管道，因为它们有许多限制和没有可靠性保证。如果您想从文件中摄取数据，可以使用几种替代方案：[FilePulse Connector](https://oreil.ly/VLCf2), [FileSystem Connector](https://oreil.ly/Fcryw), 或 [SpoolDir](https://oreil.ly/qgsI4)。

## 连接器示例：MySQL 到 Elasticsearch

现在我们有一个简单的例子正在运行，让我们做一些更有用的事情。让我们将一个 MySQL 表流式传输到一个 Kafka 主题，然后从那里将其加载到 Elasticsearch 并索引其内容。

我们正在 MacBook 上运行测试。要安装 MySQL 和 Elasticsearch，只需运行：

```java
brew install mysql
brew install elasticsearch
```

下一步是确保您有这些连接器。有几个选项：

1.  使用[Confluent Hub 客户端](https://oreil.ly/c7S5z)下载和安装。

1.  从[Confluent Hub](https://www.confluent.io/hub)网站（或您感兴趣的连接器托管的任何其他网站）下载。

1.  从源代码构建。为此，您需要：

1.  克隆连接器源代码：

```java
        git clone https://github.com/confluentinc/kafka-connect-elasticsearch
        ```

1.  运行“mvn install -DskipTests”来构建项目。

1.  使用[JDBC 连接器](https://oreil.ly/yXg0S)重复。

现在我们需要加载这些连接器。创建一个目录，比如*/opt/connectors*，并更新*config/connect-distributed.properties*以包括`plugin.path=/opt/​con⁠nec⁠tors`。

然后将在构建每个连接器的“target”目录下创建的 jar 文件及其依赖项复制到“plugin.path”的适当子目录中：

```java
gwen$ mkdir /opt/connectors/jdbc
gwen$ mkdir /opt/connectors/elastic
gwen$ cp .../kafka-connect-jdbc/target/kafka-connect-jdbc-10.3.x-SNAPSHOT.jar /opt/connectors/jdbc
gwen$ cp ../kafka-connect-elasticsearch/target/kafka-connect-elasticsearch-11.1.0-SNAPSHOT.jar /opt/connectors/elastic
gwen$ cp ../kafka-connect-elasticsearch/target/kafka-connect-elasticsearch-11.1.0-SNAPSHOT-package/share/java/kafka-connect-elasticsearch/* /opt/connectors/elastic
```

此外，由于我们不仅需要连接到任何数据库，而是特别需要连接到 MySQL，因此您需要下载并安装 MySQL JDBC 驱动程序。出于许可证原因，驱动程序不随连接器一起提供。您可以从[MySQL 网站](https://oreil.ly/KZCPw)下载驱动程序，然后将 jar 文件放在*/opt/connectors/jdbc*中。

重新启动 Kafka Connect 工作程序，并检查新的连接器插件是否已列出：

```java
gwen$  bin/connect-distributed.sh config/connect-distributed.properties &

gwen$  curl http://localhost:8083/connector-plugins
[
  {
    "class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
    "type": "sink",
    "version": "11.1.0-SNAPSHOT"
  },
  {
    "class": "io.confluent.connect.jdbc.JdbcSinkConnector",
    "type": "sink",
    "version": "10.3.x-SNAPSHOT"
  },
  {
    "class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "type": "source",
    "version": "10.3.x-SNAPSHOT"
  }
```

我们可以看到我们现在在我们的“Connect”集群中有更多的连接器插件可用。

下一步是在 MySQL 中创建一个表，我们可以使用我们的 JDBC 连接器将其流式传输到 Kafka 中：

```java
gwen$ mysql.server restart
gwen$  mysql --user=root

mysql> create database test;
Query OK, 1 row affected (0.00 sec)

mysql> use test;
Database changed
mysql> create table login (username varchar(30), login_time datetime);
Query OK, 0 rows affected (0.02 sec)

mysql> insert into login values ('gwenshap', now());
Query OK, 1 row affected (0.01 sec)

mysql> insert into login values ('tpalino', now());
Query OK, 1 row affected (0.00 sec)
```

正如您所看到的，我们创建了一个数据库和一个表，并插入了一些行作为示例。

下一步是配置我们的 JDBC 源连接器。我们可以通过查看文档找出可用的配置选项，但我们也可以使用 REST API 来查找可用的配置选项：

```java
gwen$ curl -X PUT -d '{"connector.class":"JdbcSource"}' localhost:8083/connector-plugins/JdbcSourceConnector/config/validate/ --header "content-Type:application/json"

{
    "configs": [
        {
            "definition": {
                "default_value": "",
                "dependents": [],
                "display_name": "Timestamp Column Name",
                "documentation": "The name of the timestamp column to use
                to detect new or modified rows. This column may not be
                nullable.",
                "group": "Mode",
                "importance": "MEDIUM",
                "name": "timestamp.column.name",
                "order": 3,
                "required": false,
                "type": "STRING",
                "width": "MEDIUM"
            },
            <more stuff>
```

我们要求 REST API 验证连接器的配置，并向其发送了一个仅包含类名的配置（这是必需的最低配置）。作为响应，我们得到了所有可用配置的 JSON 定义。

有了这些信息，现在是时候创建和配置我们的 JDBC 连接器了：

```java
echo '{"name":"mysql-login-connector", "config":{"connector.class":"JdbcSourceConnector","connection.url":"jdbc:mysql://127.0.0.1:3306/test?user=root","mode":"timestamp","table.whitelist":"login","validate.non.null":false,"timestamp.column.name":"login_time","topic.prefix":"mysql."}}' | curl -X POST -d @- http://localhost:8083/connectors --header "content-Type:application/json"

{
  "name": "mysql-login-connector",
  "config": {
    "connector.class": "JdbcSourceConnector",
    "connection.url": "jdbc:mysql://127.0.0.1:3306/test?user=root",
    "mode": "timestamp",
    "table.whitelist": "login",
    "validate.non.null": "false",
    "timestamp.column.name": "login_time",
    "topic.prefix": "mysql.",
    "name": "mysql-login-connector"
  },
  "tasks": []
}
```

让我们通过从“mysql.login”主题中读取数据来确保它起作用：

```java
gwen$ bin/kafka-console-consumer.sh --bootstrap-server=localhost:9092 --topic mysql.login --from-beginning
```

如果您收到错误消息说主题不存在或看不到数据，请检查 Connect worker 日志以查找错误，例如：

```java
[2016-10-16 19:39:40,482] ERROR Error while starting connector mysql-login-connector (org.apache.kafka.connect.runtime.WorkerConnector:108)
org.apache.kafka.connect.errors.ConnectException: java.sql.SQLException: Access denied for user 'root;'@'localhost' (using password: NO)
       	at io.confluent.connect.jdbc.JdbcSourceConnector.start(JdbcSourceConnector.java:78)
```

其他问题可能涉及类路径中驱动程序的存在或读取表的权限。

一旦连接器运行，如果您在“login”表中插入了额外的行，您应该立即在“mysql.login”主题中看到它们的反映。

# 变更数据捕获和 Debezium 项目

我们正在使用的 JDBC 连接器使用 JDBC 和 SQL 来扫描数据库表中的新记录。它通过使用时间戳字段或递增的主键来检测新记录。这是一个相对低效且有时不准确的过程。所有关系型数据库都有一个事务日志（也称为重做日志、binlog 或预写日志）作为其实现的一部分，并且许多允许外部系统直接从其事务日志中读取数据——这是一个更准确和高效的过程，称为“变更数据捕获”。大多数现代 ETL 系统依赖于变更数据捕获作为数据源。[Debezium 项目](https://debezium.io)提供了一系列高质量的开源变更捕获连接器，适用于各种数据库。如果您计划将数据从关系型数据库流式传输到 Kafka，我们强烈建议如果您的数据库存在的话使用 Debezium 变更捕获连接器。此外，Debezium 文档是我们见过的最好的文档之一——除了记录连接器本身外，它还涵盖了与变更数据捕获相关的有用设计模式和用例，特别是在微服务的背景下。

将 MySQL 数据传输到 Kafka 本身就很有用，但让我们通过将数据写入 Elasticsearch 来增加乐趣。

首先，我们启动 Elasticsearch 并通过访问其本地端口来验证它是否正常运行：

```java
gwen$ elasticsearch &
gwen$ curl http://localhost:9200/
{
  "name" : "Chens-MBP",
  "cluster_name" : "elasticsearch_gwenshap",
  "cluster_uuid" : "X69zu3_sQNGb7zbMh7NDVw",
  "version" : {
    "number" : "7.5.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "8bec50e1e0ad29dad5653712cf3bb580cd1afcdf",
    "build_date" : "2020-01-15T12:11:52.313576Z",
    "build_snapshot" : false,
    "lucene_version" : "8.3.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

现在创建并启动连接器：

```java
echo '{"name":"elastic-login-connector", "config":{"connector.class":"ElasticsearchSinkConnector","connection.url":"http://localhost:9200","type.name":"mysql-data","topics":"mysql.login","key.ignore":true}}' | curl -X POST -d @- http://localhost:8083/connectors --header "content-Type:application/json"

{
  "name": "elastic-login-connector",
  "config": {
    "connector.class": "ElasticsearchSinkConnector",
    "connection.url": "http://localhost:9200",
    "topics": "mysql.login",
    "key.ignore": "true",
    "name": "elastic-login-connector"
  },
  "tasks": [
    {
      "connector": "elastic-login-connector",
      "task": 0
    }
  ]
}
```

这里有一些我们需要解释的配置。`connection.url`只是我们之前配置的本地 Elasticsearch 服务器的 URL。Kafka 中的每个主题默认情况下将成为一个单独的 Elasticsearch 索引，与主题名称相同。我们写入 Elasticsearch 的唯一主题是`mysql.login`。JDBC 连接器不会填充消息键。因此，Kafka 中的事件具有空键。因为 Kafka 中的事件缺少键，我们需要告诉 Elasticsearch 连接器使用主题名称、分区 ID 和偏移量作为每个事件的键。这是通过将`key.ignore`配置设置为`true`来完成的。

让我们检查一下是否已创建了具有`mysql.login`数据的索引：

```java
gwen$ curl 'localhost:9200/_cat/indices?v'
health status index       uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   mysql.login wkeyk9-bQea6NJmAFjv4hw   1   1          2            0      3.9kb          3.9kb
```

如果索引不存在，请查看 Connect worker 日志中的错误。缺少配置或库是错误的常见原因。如果一切正常，我们可以搜索索引以查找我们的记录：

```java
gwen$ curl -s -X "GET" "http://localhost:9200/mysql.login/_search?pretty=true"
{
  "took" : 40,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "mysql.login",
        "_type" : "_doc",
        "_id" : "mysql.login+0+0",
        "_score" : 1.0,
        "_source" : {
          "username" : "gwenshap",
          "login_time" : 1621699811000
        }
      },
      {
        "_index" : "mysql.login",
        "_type" : "_doc",
        "_id" : "mysql.login+0+1",
        "_score" : 1.0,
        "_source" : {
          "username" : "tpalino",
          "login_time" : 1621699816000
        }
      }
    ]
  }
}
```

如果您在 MySQL 表中添加新记录，它们将自动出现在 Kafka 的`mysql.login`主题中，并出现在相应的 Elasticsearch 索引中。

现在我们已经看到如何构建和安装 JDBC 源和 Elasticsearch 接收器，我们可以构建和使用任何适合我们用例的连接器对。Confluent 维护了一组自己预构建的连接器，以及来自社区和其他供应商的一些连接器，位于[Confluent Hub](https://www.confluent.io/hub)。您可以选择列表中的任何连接器进行尝试，下载它，配置它 - 要么基于文档，要么通过从 REST API 中提取配置 - 并在您的 Connect worker 集群上运行它。

# 构建您自己的连接器

连接器 API 是公开的，任何人都可以创建新的连接器。因此，如果您希望集成的数据存储没有现有的连接器，我们鼓励您编写自己的连接器。然后，您可以将其贡献给 Confluent Hub，以便其他人可以发现并使用它。本章的范围超出了讨论构建连接器所涉及的所有细节，但有多篇博客文章[解释了如何做到这一点](https://oreil.ly/WUqlZ)，以及来自[Kafka Summit NY 2019](https://oreil.ly/rV9RH)、[Kafka Summit London 2018](https://oreil.ly/Jz7XV)和[ApacheCon](https://oreil.ly/8QsOL)的精彩演讲。我们还建议查看现有的连接器作为起点，并可能使用[Apache Maven archtype](http://bit.ly/2sc9E9q)来加速。我们始终鼓励您在 Apache Kafka 社区邮件列表（users@kafka.apache.org）上寻求帮助或展示您最新的连接器，或者将其提交到 Confluent Hub，以便它们可以轻松被发现。

## 单消息转换

将记录从 MySQL 复制到 Kafka，然后再复制到 Elastic 本身就很有用，但 ETL 管道通常涉及转换步骤。在 Kafka 生态系统中，我们将转换分为单消息转换（SMT），它们是无状态的，以及流处理，它可以是有状态的。SMT 可以在 Kafka Connect 中完成，转换消息而无需编写任何代码。更复杂的转换通常涉及连接或聚合，将需要有状态的 Kafka Streams 框架。我们将在后面的章节中讨论 Kafka Streams。

Apache Kafka 包括以下 SMT：

转换

更改字段的数据类型。

MaskField

用 null 替换字段的内容。这对于删除敏感或个人识别数据非常有用。

过滤器

删除或包含符合特定条件的所有消息。内置条件包括匹配主题名称、特定标头，或消息是否为墓碑（即具有空值）。

扁平化

将嵌套数据结构转换为扁平结构。这是通过将路径中所有字段的名称连接到特定值来完成的。

HeaderFrom

将消息中的字段移动或复制到标头中。

插入标头

向每条消息的标头添加静态字符串。

插入字段

向消息添加一个新字段，可以使用其偏移等元数据的值，也可以使用静态值。

正则路由器

使用正则表达式和替换字符串更改目标主题。

替换字段

删除或重命名消息中的字段。

时间戳转换器

修改字段的时间格式 - 例如，从 Unix Epoch 到字符串。

时间戳路由器

根据消息时间戳修改主题。这在接收连接器中非常有用，当我们想要根据时间戳将消息复制到特定的表分区时，主题字段用于在目标系统中找到等效的数据集。

此外，转换是从主要 Apache Kafka 代码库之外的贡献者那里获得的。这些可以在 GitHub（[Lenses.io](https://oreil.ly/fWAyh)、[Aiven](https://oreil.ly/oQRG5)和[Jeremy Custenborder](https://oreil.ly/OdPHW)有有用的集合）或[Confluent Hub](https://oreil.ly/Up8dM)上找到。

要了解更多关于 Kafka Connect SMT 的信息，您可以阅读[“SMT 十二天”](https://oreil.ly/QnpQV)博客系列中的详细示例。此外，您还可以通过[教程和深入探讨](https://oreil.ly/rw4CU)来学习如何编写自己的转换。

例如，假设我们想要为之前创建的 MySQL 连接器生成的每条记录添加[记录标头](https://oreil.ly/ISiWs)。标头将指示该记录是由此 MySQL 连接器创建的，这在审计人员想要检查这些记录的渊源时非常有用。

为此，我们将用以下内容替换以前的 MySQL 连接器配置：

```java
echo '{
  "name": "mysql-login-connector",
  "config": {
    "connector.class": "JdbcSourceConnector",
    "connection.url": "jdbc:mysql://127.0.0.1:3306/test?user=root",
    "mode": "timestamp",
    "table.whitelist": "login",
    "validate.non.null": "false",
    "timestamp.column.name": "login_time",
    "topic.prefix": "mysql.",
    "name": "mysql-login-connector",
    "transforms": "InsertHeader",
    "transforms.InsertHeader.type":
      "org.apache.kafka.connect.transforms.InsertHeader",
    "transforms.InsertHeader.header": "MessageSource",
    "transforms.InsertHeader.value.literal": "mysql-login-connector"
  }}' | curl -X POST -d @- http://localhost:8083/connectors --header "content-Type:application/json"
```

现在，如果您向我们在上一个示例中创建的 MySQL 表中插入几条记录，您将能够看到`mysql.login`主题中的新消息具有标头（请注意，您需要 Apache Kafka 2.7 或更高版本才能在控制台消费者中打印标头）：

```java
bin/kafka-console-consumer.sh --bootstrap-server=localhost:9092 --topic mysql.login --from-beginning --property print.headers=true

NO_HEADERS	{"schema":{"type":"struct","fields":[{"type":"string","optional":true,"field":"username"},{"type":"int64","optional":true,"name":"org.apache.kafka.connect.data.Timestamp","version":1,"field":"login_time"}],"optional":false,"name":"login"},"payload":{"username":"tpalino","login_time":1621699816000}}
MessageSource:mysql-login-connector	{"schema":{"type":"struct","fields":

[{"type":"string","optional":true,"field":"username"},{"type":"int64","optional":true,"name":"org.apache.kafka.connect.data.Timestamp","version":1,"field":"login_time"}],"optional":false,"name":"login"},"payload":{"username":"rajini","login_time":1621803287000}}
```

如您所见，旧记录显示“NO_HEADERS”，而新记录显示“MessageSource:mysql-login-connector”。

# 错误处理和死信队列

转换是一个连接器配置的示例，它不特定于一个连接器，但可以在任何连接器的配置中使用。另一个非常有用的连接器配置是`error.tolerance` - 您可以配置任何连接器静默丢弃损坏的消息，或将它们路由到一个名为“死信队列”的特殊主题。您可以在[“Kafka Connect 深入探讨-错误处理和死信队列”博客文章](https://oreil.ly/935hH)中找到更多详细信息。

## 深入了解 Kafka Connect

要理解 Kafka Connect 的工作原理，您需要了解三个基本概念以及它们之间的相互作用。正如我们之前解释过并通过示例演示的那样，要使用 Kafka Connect，您需要运行一个工作节点集群并创建/删除连接器。我们之前没有深入探讨的一个额外细节是转换器处理数据的方式 - 这些是将 MySQL 行转换为 JSON 记录的组件，连接器将其写入 Kafka。

让我们更深入地了解每个系统以及它们之间的相互作用。

### 连接器和任务

连接器插件实现了连接器 API，其中包括两个部分：

连接器

连接器负责三件重要的事情：

+   确定连接器将运行多少任务

+   决定如何在任务之间分配数据复制工作

+   从工作节点获取任务的配置并将其传递

例如，JDBC 源连接器将连接到数据库，发现现有表以进行复制，并根据此决定需要多少任务——选择“tasks.max”配置和表的数量中的较小值。一旦决定了将运行多少任务，它将为每个任务生成一个配置——使用连接器配置（例如，“connection.url”）和为每个任务分配的表列表。 “taskConfigs（）”方法返回映射列表（即，我们要运行的每个任务的配置）。然后工作人员负责启动任务，并为每个任务提供其自己独特的配置，以便它将从数据库复制一组唯一的表。请注意，当您通过 REST API 启动连接器时，它可能在任何节点上启动，并且随后启动的任务也可能在任何节点上执行。

任务

任务实际上负责将数据进出 Kafka。所有任务都是通过从工作人员接收上下文来初始化的。源上下文包括一个允许源任务存储源记录的偏移量的对象（例如，在文件连接器中，偏移量是文件中的位置；在 JDBC 源连接器中，偏移量可以是表中的时间戳列）。汇接器的上下文包括允许连接器控制从 Kafka 接收的记录的方法，用于诸如施加反压、重试和外部存储偏移以实现精确一次交付等操作。任务初始化后，它们将使用包含任务的连接器为其创建的配置的“属性”对象启动。任务启动后，源任务轮询外部系统并返回记录列表，工作人员将这些记录发送到 Kafka 代理。汇任务通过工作人员从 Kafka 接收记录，并负责将记录写入外部系统。

### 工人

Kafka Connect 的工作进程是执行连接器和任务的“容器”进程。它们负责处理定义连接器及其配置的 HTTP 请求，以及将连接器配置存储在内部 Kafka 主题中，启动连接器及其任务，并传递适当的配置。如果工作进程停止或崩溃，Connect 集群中的其他工作人员将通过 Kafka 的消费者协议中的心跳检测到这一点，并将在该工作人员上运行的连接器和任务重新分配给剩余的工作人员。如果新的工作人员加入 Connect 集群，其他工作人员将注意到这一点，并分配连接器或任务给它，以确保所有工作人员之间的负载均衡。工作人员还负责自动将源和汇连接器的偏移量提交到内部 Kafka 主题，并在任务抛出错误时处理重试。

了解工作人员的最佳方法是意识到连接器和任务负责数据集成的“移动数据”部分，而工作人员负责 REST API、配置管理、可靠性、高可用性、扩展和负载平衡。

这种关注点的分离是使用 Connect API 与经典的消费者/生产者 API 的主要优势。有经验的开发人员知道，编写从 Kafka 读取数据并将其插入数据库的代码可能需要一两天，但如果需要处理配置、错误、REST API、监控、部署、扩展和处理故障，可能需要几个月才能做到一切正确。大多数数据集成管道不仅涉及一个源或目标。因此，考虑一下为数据库集成而花费的工作量，为其他技术重复多次。如果您使用连接器实现数据复制，您的连接器将插入处理一堆复杂操作问题的工作人员，这些问题您无需担心。

### 转换器和 Connect 的数据模型

Connect API 谜题的最后一部分是连接器数据模型和转换器。Kafka 的 Connect API 包括一个数据 API，其中包括数据对象和描述该数据的模式。例如，JDBC 源从数据库中读取列，并基于数据库返回的列的数据类型构造一个`Connect Schema`对象。然后使用模式构造包含数据库记录中所有字段的`Struct`。对于每一列，我们存储列名和该列中的值。每个源连接器都会做类似的事情——从源系统中读取事件并生成`Schema`和`Value`对。汇连接器则相反——获取`Schema`和`Value`对，并使用`Schema`解析值并将其插入目标系统。

尽管源连接器知道如何基于数据 API 生成对象，但仍然存在一个问题，即`Connect`工作程序如何将这些对象存储在 Kafka 中。这就是转换器发挥作用的地方。当用户配置工作程序（或连接器）时，他们选择要使用哪种转换器将数据存储在 Kafka 中。目前，可用的选择包括原始类型、字节数组、字符串、Avro、JSON、JSON 模式或 Protobufs。JSON 转换器可以配置为在结果记录中包含模式或不包含模式，因此我们可以支持结构化和半结构化数据。当连接器将 Data API 记录返回给工作程序时，工作程序然后使用配置的转换器将记录转换为 Avro 对象、JSON 对象或字符串，然后将结果存储到 Kafka 中。

对于汇连接器，相反的过程发生。当 Connect 工作程序从 Kafka 中读取记录时，它使用配置的转换器将记录从 Kafka 中的格式（即原始类型、字节数组、字符串、Avro、JSON、JSON 模式或 Protobufs）转换为 Connect 数据 API 记录，然后将其传递给汇连接器，汇连接器将其插入目标系统中。

这使得 Connect API 能够支持存储在 Kafka 中的不同类型的数据，而与连接器实现无关（即，只要有可用的转换器，任何连接器都可以与任何记录类型一起使用）。

### 偏移量管理

偏移量管理是工作程序为连接器执行的方便服务之一（除了通过 REST API 进行部署和配置管理）。其想法是，连接器需要知道它们已经处理了哪些数据，并且它们可以使用 Kafka 提供的 API 来维护已经处理的事件的信息。

对于源连接器来说，这意味着连接器返回给 Connect 工作程序的记录包括逻辑分区和逻辑偏移量。这些不是 Kafka 分区和 Kafka 偏移量，而是源系统中需要的分区和偏移量。例如，在文件源中，分区可以是一个文件，偏移量可以是文件中的行号或字符号。在 JDBC 源中，分区可以是数据库表，偏移量可以是表中记录的 ID 或时间戳。编写源连接器时涉及的最重要的设计决策之一是决定在源系统中对数据进行分区和跟踪偏移量的良好方式，这将影响连接器可以实现的并行级别以及是否可以提供至少一次或精确一次语义。

当源连接器返回包含每条记录的源分区和偏移量的记录列表时，工作程序将记录发送到 Kafka 代理。如果代理成功确认记录，工作程序将存储发送到 Kafka 的记录的偏移量。这允许连接器在重新启动或崩溃后从最近存储的偏移量开始处理事件。存储机制是可插拔的，通常是一个 Kafka 主题；您可以使用`offset.storage.topic`配置控制主题名称。此外，Connect 使用 Kafka 主题存储我们创建的所有连接器的配置和每个连接器的状态，这些主题的名称分别由`config.storage.topic`和`status.storage.topic`配置。

接收连接器具有相反但类似的工作流程：它们读取 Kafka 记录，这些记录已经有了主题、分区和偏移标识符。然后它们调用`connector`的`put()`方法，应该将这些记录存储在目标系统中。如果连接器报告成功，它们将提交它们给连接器的偏移量回到 Kafka，使用通常的消费者提交方法。

框架本身提供的偏移跟踪应该使开发人员更容易编写连接器，并在使用不同的连接器时保证一定程度的一致行为。

# Kafka Connect 的替代方案

到目前为止，我们已经非常详细地查看了 Kafka 的 Connect API。虽然我们喜欢 Connect API 提供的便利和可靠性，但这并不是将数据输入和输出 Kafka 的唯一方法。让我们看看其他替代方案以及它们通常在何时使用。

## 其他数据存储的摄入框架

虽然我们喜欢认为 Kafka 是宇宙的中心，但有些人不同意。有些人大部分的数据架构都是围绕 Hadoop 或 Elasticsearch 等系统构建的。这些系统有它们自己的数据摄入工具——Hadoop 的 Flume，Elasticsearch 的 Logstash 或 Fluentd。如果您实际上正在构建一个以 Hadoop 为中心或以 Elastic 为中心的系统，而 Kafka 只是该系统的许多输入之一，那么使用 Flume 或 Logstash 是有意义的。

## 基于 GUI 的 ETL 工具

像 Informatica 这样的老式系统，像 Talend 和 Pentaho 这样的开源替代方案，甚至像 Apache NiFi 和 StreamSets 这样的较新替代方案，都支持 Apache Kafka 作为数据源和目的地。如果您已经在使用这些系统，比如您已经使用 Pentaho 做了所有事情，您可能不会对为了 Kafka 而添加另一个数据集成系统感兴趣。如果您正在使用基于 GUI 的方法构建 ETL 管道，这也是有道理的。这些系统的主要缺点是它们通常是为复杂的工作流程而构建的，如果您只是想将数据输入和输出 Kafka，它们将是一个相当沉重和复杂的解决方案。我们认为数据集成应该专注于在所有条件下忠实地传递消息，而大多数 ETL 工具增加了不必要的复杂性。

我们鼓励您将 Kafka 视为一个可以处理数据集成（使用 Connect）、应用程序集成（使用生产者和消费者）和流处理的平台。Kafka 可能是 ETL 工具的可行替代，该工具仅集成数据存储。

## 流处理框架

几乎所有的流处理框架都包括从 Kafka 读取事件并将其写入其他几个系统的能力。如果您的目标系统得到支持，并且您已经打算使用该流处理框架来处理来自 Kafka 的事件，那么使用相同的框架进行数据集成似乎是合理的。这通常可以节省流处理工作流程中的一步（无需将处理过的事件存储在 Kafka 中，只需读取它们并将其写入另一个系统），但缺点是可能更难排除诸如丢失和损坏的消息等问题。

# 摘要

在本章中，我们讨论了使用 Kafka 进行数据集成。从使用 Kafka 进行数据集成的原因开始，我们涵盖了数据集成解决方案的一般考虑因素。我们展示了为什么我们认为 Kafka 及其 Connect API 是一个很好的选择。然后，我们给出了几个不同场景下如何使用 Kafka Connect 的示例，花了一些时间看 Connect 是如何工作的，然后讨论了一些 Kafka Connect 的替代方案。

无论您最终选择哪种数据集成解决方案，最重要的特性始终是其在所有故障条件下传递所有消息的能力。我们相信 Kafka Connect 非常可靠——基于其与 Kafka 经过验证的可靠性特性的集成——但重要的是您测试您选择的系统，就像我们一样。确保您选择的数据集成系统可以在停止进程、崩溃的机器、网络延迟和高负载的情况下生存而不会丢失消息。毕竟，在本质上，数据集成系统只有一个任务——传递这些消息。

当然，可靠性通常是集成数据系统时最重要的要求，但这只是一个要求。在选择数据系统时，首先审查您的要求是很重要的（参考“构建数据管道时的考虑因素”以获取示例），然后确保您选择的系统满足这些要求。但这还不够——您还必须充分了解您的数据集成解决方案，以确保您使用的方式支持您的要求。Kafka 支持至少一次语义是不够的；您必须确保您没有意外地配置它以确保完全的可靠性。
