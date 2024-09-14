# 前言

十年前我启动 Apache Spark 项目时，我的主要目标之一是让广泛范围的用户更容易实现并行算法。新的大规模数据算法对计算的各个领域都产生了深远影响，我希望帮助开发人员实现这样的算法，并在不必从头开始构建分布式系统的情况下推理其性能。

因此，我对马哈茂德·帕尔西安博士关于 Spark 数据算法的新书感到非常兴奋。帕尔西安博士在大规模数据并行算法方面拥有广泛的研究和实践经验，包括作为 Illumina 大数据团队负责人开发生物信息学新算法。在这本书中，他通过其 Python API，PySpark，介绍了 Spark，并展示了如何使用 Spark 的分布式计算原语高效地实现各种有用的算法。他还解释了底层 Spark 引擎的工作原理，以及如何通过控制数据分区等技术来优化您的算法。无论是想要以可扩展的方式实现现有算法的读者，还是正在使用 Spark 开发新的定制算法的读者，这本书都将是一个很好的资源。

我也非常激动，因为 Parsian 博士在他讨论的所有算法中都包含了实际工作的代码示例，尽可能使用真实世界的问题。这些将为希望实现类似计算的读者提供一个很好的起点。无论您是打算直接使用这些算法还是利用 Spark 构建自己的定制算法，我希望您享受这本书作为开源引擎、其内部工作以及对计算产生广泛影响的现代并行算法的介绍。

Matei Zaharia

斯坦福大学计算机科学助理教授

Databricks 首席技术专家

Apache Spark 的创始人