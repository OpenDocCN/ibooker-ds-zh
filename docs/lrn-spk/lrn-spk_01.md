# 前言

欢迎您阅读第二版的*Learning Spark*。自 2015 年首版由 Holden Karau、Andy Konwinski、Patrick Wendell 和 Matei Zaharia 共同撰写以来已经过去五年。这个新版已经更新，以反映 Apache Spark 经过 Spark 2.x 和 Spark 3.0 的演变，包括其扩展的内置和外部数据源生态系统，机器学习和流处理技术，与 Spark 紧密集成。

自其首个 1.x 版本发布以来的多年间，Spark 已成为事实上的大数据统一处理引擎。在此过程中，它已扩展其范围以支持各种分析工作负载。我们的目的是为读者捕捉和整理这一演变过程，展示不仅可以如何使用 Spark，而且它如何适应大数据和机器学习的新时代。因此，我们设计每一章节都在前一章节奠定的基础上逐步建设，确保内容适合我们的目标读者群体。

# 本书适合对象

大多数处理大数据的开发人员是数据工程师、数据科学家或机器学习工程师。本书旨在为那些希望使用 Spark 来扩展其应用程序以处理大量数据的专业人士提供帮助。

特别是数据工程师将学习如何使用 Spark 的结构化 API 执行复杂的数据探索和分析，无论是批量还是流式数据；使用 Spark SQL 进行交互式查询；使用 Spark 的内置和外部数据源在不同文件格式中读取、精炼和写入数据，作为其提取、转换和加载（ETL）任务的一部分；以及使用 Spark 和开源 Delta Lake 表格格式构建可靠的数据湖。

对于数据科学家和机器学习工程师，Spark 的 MLlib 库提供了许多常见算法来构建分布式机器学习模型。我们将介绍如何使用 MLlib 构建管道，分布式机器学习的最佳实践，如何使用 Spark 扩展单节点模型，以及如何使用开源库 MLflow 管理和部署这些模型。

虽然本书着重于将 Spark 作为多样工作负载的分析引擎进行学习，但我们不会涵盖 Spark 支持的所有语言。大多数章节中的示例都是用 Scala、Python 和 SQL 编写的。在必要时，我们也会添加一些 Java。对于有意通过 R 学习 Spark 的人士，我们推荐 Javier Luraschi、Kevin Kuo 和 Edgar Ruiz 的[*Mastering Spark with R*](http://shop.oreilly.com/product/0636920223764.do)（O’Reilly）。

最后，由于 Spark 是一个分布式引擎，建立对 Spark 应用概念的理解至关重要。我们将指导您了解您的 Spark 应用程序如何与 Spark 的分布式组件进行交互，以及如何将执行分解为集群上的并行任务。我们还将涵盖支持的部署模式及其适用环境。

尽管有许多我们选择涵盖的主题，但我们选择不专注于的主题也有几个。这些包括旧版的低级别 Resilient Distributed Dataset (RDD) API 和 GraphX，即用于图形和图形并行计算的 Spark API。我们也没有涵盖高级主题，比如如何扩展 Spark 的 Catalyst 优化器以实现自己的操作，如何实现自己的目录，或者如何编写自己的 DataSource V2 数据接收器和数据源。虽然这些都是 Spark 的一部分，但它们超出了您第一本关于学习 Spark 的书籍的范围。

相反，我们已经将书籍围绕 Spark 的结构化 API 进行了重点和组织，跨其所有组件，并展示您如何使用 Spark 在规模上处理结构化数据以执行数据工程或数据科学任务。

# 书籍的组织方式

我们按照一种引导您从章节到章节的方式组织了这本书，通过介绍概念，通过示例代码片段演示这些概念，并在书的[GitHub 存储库](https://github.com/databricks/LearningSparkV2)中提供完整的代码示例或笔记本。

第一章，*Apache Spark 简介：统一分析引擎*

介绍大数据的演变，并提供 Apache Spark 的高级概述及其在大数据中的应用。

第二章，*下载 Apache Spark 并入门*

引导您下载并在本地计算机上设置 Apache Spark。

第三章，*Apache Spark 的结构化 APIs* 到 第六章，*Spark SQL 和数据集*

这些章节专注于使用 DataFrame 和 Dataset 结构化 API 从内置和外部数据源中摄取数据，应用内置和自定义函数，并利用 Spark SQL。这些章节构成后续章节的基础，包括所有最新的 Spark 3.0 变更。

第七章，*优化和调优 Spark 应用程序*

通过 Spark UI 提供了调优、优化、调试和检查 Spark 应用程序的最佳实践，以及可以调整以提高性能的配置细节。

第八章，*结构化流处理*

指导您通过 Spark 流处理引擎的演变和结构化流处理编程模型。它检查典型流查询的解剖，并讨论转换流数据的不同方法——有状态聚合、流连接和任意有状态聚合——同时指导您如何设计高性能的流查询。

第九章，*使用 Apache Spark 构建可靠的数据湖*

调查了三种开源表格式存储解决方案，作为 Spark 生态系统的一部分，它们利用 Apache Spark 构建具有事务保证的可靠数据湖。由于 Delta Lake 与 Spark 在批处理和流处理工作负载中的紧密集成，我们重点关注该解决方案，并探讨其如何促进数据管理新范式，即“lakehouse”。

第十章，*MLlib 机器学习*

介绍了 MLlib，Spark 的分布式机器学习库，并通过端到端示例演示了如何构建机器学习流水线，包括特征工程、超参数调优、评估指标以及模型的保存和加载。

第十一章，*管理、部署和扩展机器学习流水线与 Apache Spark*

讲解如何使用 MLflow 跟踪和管理 MLlib 模型，比较和对比不同的模型部署选项，探讨如何利用 Spark 进行非 MLlib 模型的分布式模型推断、特征工程和/或超参数调优。

第十二章，*尾声：Apache Spark 3.0*

尾声突出了 Spark 3.0 的显著特性和变化。虽然增强和特性的全面范围太广泛而无法适应单一章节，但我们突出了您应该了解的主要变化，并建议您在 Spark 3.0 正式发布时查看发布说明。

在这些章节中，我们根据需要引入或记录了 Spark 3.0 的功能，并针对 Spark 3.0.0-preview2 测试了所有的代码示例和 notebooks。

# 如何使用代码示例

本书中的代码示例从简短的代码片段到完整的 Spark 应用程序和端到端 notebooks，涵盖 Scala、Python、SQL 以及必要时的 Java。

虽然某些章节中的短代码片段是自包含的，可以复制并粘贴到 Spark shell（`pyspark`或`spark-shell`）中运行，其他则是来自独立的 Scala、Python 或 Java Spark 应用程序或端到端 notebooks 的片段。要在 Scala、Python 或 Java 中运行独立的 Spark 应用程序，请阅读本书 GitHub 存储库的各自章节的 README 文件中的说明。

至于 notebooks，要运行这些内容，您需要注册一个免费的[Databricks 社区版](https://community.cloud.databricks.com/)帐户。我们详细介绍了如何导入这些 notebooks，并在 Spark 3.0 中创建一个集群，详见[README](https://github.com/databricks/LearningSparkV2/tree/master/notebooks)。

# 使用的软件和配置

本书中大部分代码及其相关 notebooks 均针对 Apache Spark 3.0.0-preview2 编写和测试，这是我们在编写最终章节时可用的版本。

当本书出版时，Apache Spark 3.0 将已发布并可供社区普遍使用。我们建议您根据您操作系统的以下配置 [下载](https://oreil.ly/WFX48) 并使用官方版本：

+   Apache Spark 3.0（为 Apache Hadoop 2.7 预构建）

+   Java 开发工具包（JDK）1.8.0

如果您只打算使用 Python，则可以简单地运行 `pip install pyspark`。

# 本书中使用的约定

本书使用以下排版约定：

*斜体*

表示新术语、网址、电子邮件地址、文件名和文件扩展名。

`固定宽度`

用于程序清单，以及段落内引用程序元素，如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

`**粗体固定宽度**`

显示用户应直接输入的命令或其他文本。

`*斜体固定宽度*`

显示应由用户提供的值或由上下文确定的值。

###### 注意

此元素表示一般注释。

# 使用代码示例

如果您有技术问题或使用代码示例时遇到问题，请发送电子邮件至 *bookquestions@oreilly.com*。

本书旨在帮助您完成工作。一般情况下，如果本书提供示例代码，则您可以在您的程序和文档中使用它。除非您要复制本书的大部分代码，否则您无需联系我们以获得许可。例如，编写一个使用本书多个代码片段的程序不需要许可。销售或分发 O'Reilly 书籍中的示例代码需要许可。引用本书并引用示例代码回答问题不需要许可。将本书中大量示例代码整合到您产品的文档中需要许可。

我们赞赏，但通常不要求署名。署名通常包括标题、作者、出版商和 ISBN。例如：“*学习 Spark*，第二版，作者 Jules S. Damji、Brooke Wenig、Tathagata Das 和 Denny Lee。版权所有 2020 Databricks, Inc.，978-1-492-05004-9。”

如果您认为您对代码示例的使用超出了合理使用范围或上述许可，请随时与我们联系，邮箱为 *permissions@oreilly.com*。

# O'Reilly 在线学习

###### 注意

40 多年来，[*O'Reilly Media*](http://oreilly.com) 提供技术和商业培训、知识和见解，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专长。O’Reilly 的在线学习平台为您提供按需访问的实时培训课程、深入的学习路径、交互式编码环境以及来自 O’Reilly 和其他 200 多家出版商的大量文本和视频。有关更多信息，请访问 [*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送给出版商：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104 (传真)

访问我们的书页，我们在那里列出勘误、示例和任何额外信息，网址为 [*https://oreil.ly/LearningSpark2*](https://oreil.ly/LearningSpark2)。

电子邮件 *bookquestions@oreilly.com*，您可以评论或询问有关本书的技术问题。

有关我们的书籍和课程的新闻和信息，请访问 [*http://oreilly.com*](http://oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

在 YouTube 上关注我们：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)

# 致谢

这个项目真正是众多人的团队努力，没有他们的支持和反馈，我们将无法完成这本书，尤其是在当今前所未有的 COVID-19 时代。

首先，我们要感谢我们的雇主，Databricks，支持我们并在工作中分配了专门的时间来完成这本书。特别感谢 Matei Zaharia、Reynold Xin、Ali Ghodsi、Ryan Boyd 和 Rick Schultz，他们鼓励我们写第二版。

第二，我们要感谢我们的技术审阅者：Adam Breindel、Amir Issaei、Jacek Laskowski、Sean Owen 和 Vishwanath Subramanian。他们通过他们在社区和行业的技术专业知识，提供了勤奋和建设性的反馈，使本书成为了宝贵的资源，用来学习 Spark。

除了正式的书评者外，我们还从其他具有特定主题和章节知识的人士那里获得了宝贵的反馈，我们要感谢他们的贡献。特别感谢：Conor Murphy、Hyukjin Kwon、Maryann Xue、Niall Turbitt、Wenchen Fan、Xiao Li 和 Yuanjian Li。

最后，我们要感谢我们在 Databricks 的同事（他们对我们错过或忽视项目截止日期的包容）、我们的家人和爱人（他们在我们白天或周末深夜写作时的耐心与同情）、以及整个开源 Spark 社区。没有他们的持续贡献，Spark 就不会取得今天的成就——我们这些作者也没有什么可写的。

谢谢大家！
