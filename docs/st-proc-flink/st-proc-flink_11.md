# 第十一章：从这里去哪里？

这是一个漫长的旅程，你已经完成了本书的阅读！但是你的 Flink 之旅才刚刚开始，本章节将指引你从这里可以走的可能路径。我们将简要介绍一些未包含在本书中的额外 Flink 功能，并提供一些指向更多 Flink 资源的指引。Flink 周围存在着一个充满活力的社区，我们鼓励你与其他用户建立联系，开始贡献，或者了解正在使用 Flink 构建什么样的公司，以激发你自己的工作。

# Flink 生态系统的其他部分

尽管本书特别关注流处理，但实际上 Flink 是一个通用的分布式数据处理框架，也可以用于其他类型的数据分析。此外，Flink 提供了领域特定的库和 API，用于关系查询、复杂事件处理（CEP）和图处理。

## 用于批处理的 DataSet API

Flink 是一个完整的批处理处理器，可用于实现对有界输入数据进行一次性或定期查询的用例。DataSet 程序被指定为一系列转换，就像 DataStream 程序一样，不同之处在于 DataSet 是有界数据集合。DataSet API 提供运算符来执行过滤、映射、选择、连接和分组，以及用于从外部系统（如文件系统和数据库）读取和写入数据集的连接器。使用 DataSet API，您还可以定义迭代 Flink 程序，该程序执行固定步数的循环函数，或者直到满足收敛标准为止。

批处理作业在内部被表示为数据流程程序，并在与流处理作业相同的底层执行运行时上运行。目前，这两个 API 使用单独的执行环境，并且不能混合使用。然而，Flink 社区已经在努力统一这两者，并提供单一 API 以分析有界和无界数据流，这是 Flink 未来路线图中的一个重点。

## Table API 和 SQL 用于关系分析

尽管底层的 DataStream 和 DataSet API 是分离的，但是您可以使用其更高级别的关系 API（Table API 和 SQL）在 Flink 中实现统一的流和批量分析。

Table API 是 Scala 和 Java 的语言集成查询（LINQ）API。查询可以在批处理或流分析中执行，无需修改。它提供了常见的运算符来编写关系查询，包括选择、投影、聚合和连接，并且还具有用于自动完成和语法验证的 IDE 支持。

Flink SQL 遵循 ANSI SQL 标准，并利用[Apache Calcite](https://calcite.apache.org/)进行查询解析和优化。Flink 为批处理和流处理查询提供统一的语法和语义。由于对用户定义函数的广泛支持，SQL 可以涵盖各种用例。您可以将 SQL 查询嵌入到常规的 Flink DataSet 和 DataStream 程序中，或者直接使用 SQL CLI 客户端向 Flink 集群提交 SQL 查询。CLI 客户端允许您在命令行中检索和可视化查询结果，这使其成为尝试和调试 Flink SQL 查询或在流处理或批处理数据上运行探索性查询的强大工具。此外，您还可以使用 CLI 客户端提交分离的查询，直接将结果写入外部存储系统。

## 用于复杂事件处理和模式匹配的 FlinkCEP

FlinkCEP 是用于复杂事件模式检测的高级 API 和库。它建立在 DataStream API 之上，允许您指定要在流中检测的模式。常见的 CEP 用例包括金融应用程序、欺诈检测、复杂系统中的监控和警报，以及检测网络入侵或可疑用户行为。

## 图处理的 Gelly

Gelly 是 Flink 的图处理 API 和库。它建立在 DataSet API 和 Flink 对高效批处理迭代的支持之上。Gelly 在 Java 和 Scala 中提供高级编程抽象，用于执行图转换、聚合以及诸如顶点为中心和收集-求和-应用等迭代处理。它还包括一组常见的图算法，可供直接使用。

###### 注意

Flink 的高级 API 和接口与 DataStream 和 DataSet API 以及彼此之间良好集成，因此您可以轻松混合它们，并在同一程序中在库和 API 之间切换。例如，您可以使用 CEP 库从 DataStream 中提取模式，然后使用 SQL 分析提取的模式，或者您可以使用 Table API 将表筛选和投影为图，然后使用 Gelly 库中的图算法进行分析。

# 一个热情好客的社区

Apache Flink 拥有一个不断增长且热情好客的社区，全球贡献者和用户遍布各地。以下是一些资源，您可以使用这些资源提问、参加与 Flink 相关的活动，并了解人们如何使用 Flink：

### 邮件列表

+   user@flink.apache.org：用户支持和问题

+   dev@flink.apache.org：开发、发布和社区讨论

+   community@flink.apache.org：社区新闻和见面会

### 博客

+   [*https://flink.apache.org/blog*](https://flink.apache.org/blog/)

+   [*https://www.ververica.com/blog*](https://www.ververica.com/blog)

### 见面会和会议

+   [*https://flink-forward.org*](https://flink-forward.org/)

+   [*https://www.meetup.com/topics/apache-flink*](https://www.meetup.com/topics/apache-flink/)

再次，我们希望您通过本书更好地了解 Apache Flink 的能力和可能性。我们鼓励您成为其社区的积极一员。
