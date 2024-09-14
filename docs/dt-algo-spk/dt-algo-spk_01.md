# 序言

Spark 已经成为大规模数据分析的事实标准。自其成立九年以来，我一直在使用和教授 Spark，并且在数据提取、转换、加载（ETL）过程、分布式算法开发以及大规模数据分析方面见证了巨大的改进。我最初使用 Java 开发 Spark，但我发现尽管代码非常稳定，但需要编写冗长的代码行，这可能导致代码难以阅读。因此，为了这本书，我决定使用 PySpark（Spark 的 Python API），因为用 Python 表达 Spark 的强大更为简单：代码简短、易读且易于维护。PySpark 功能强大但使用简单，您可以通过一组简单的转换和操作表达任何 ETL 或分布式算法。

# 我为什么写这本书

这是一本关于使用 PySpark 进行数据分析的入门书籍。本书提供了一系列指南和示例，旨在帮助软件和数据工程师以最简单的方式解决数据问题。如您所知，解决任何数据问题的方法有很多种：PySpark 可以让我们为复杂问题编写简单的代码。这是我在本书中尝试表达的座右铭：保持简单，使用参数，使您的解决方案可以被其他开发人员重用。我旨在教读者如何思考数据、理解其来源和最终预期形式，以及如何使用基本的数据转换模式解决各种数据问题。

# 这本书是为谁准备的

要有效使用本书，最好了解 Python 编程语言的基础知识，例如如何使用条件语句（`if-then-else`）、遍历列表以及定义和调用函数。然而，如果您的背景是其他编程语言（如 Java 或 Scala），并且不了解 Python，也可以使用本书，因为我已经提供了关于 Spark 和 PySpark 的合理介绍。

本书主要面向希望使用 Spark 引擎和 PySpark 分析大量数据并开发分布式算法的人群。我提供了简单的示例，展示如何在 PySpark 中执行 ETL 操作和编写分布式算法。代码示例编写方式简单，可以轻松地复制粘贴以完成工作。

[GitHub 提供的示例代码](https://github.com/mahmoudparsian/data-algorithms-with-spark) 是开始您自己数据项目的绝佳资源。

# 本书的组织方式

本书包含 12 章，分为三个部分：

第一部分，“基础”

前四章介绍了 Spark 和 PySpark 的基础知识，并介绍了数据转换，如映射器、过滤器和减少器。它们包含许多实际示例，可以帮助您开始自己的 PySpark 项目。本书的前四章引入了简单的 PySpark 数据转换（如`map()`、`flatMap()`、`filter()`和`reduceByKey()`），这些转换可以解决大约 95%的所有数据问题。以下是这里的详细内容：

+   第一章，“介绍 Spark 和 PySpark”，提供了数据算法的高级概述，并介绍了使用 Spark 和 PySpark 解决数据分析问题的方法。

+   第二章，“动作中的转换”，展示了如何使用 Spark 转换（映射器、过滤器和减少器）来解决真实的数据问题。

+   第三章，“映射器转换”，介绍了最常用的映射器转换：`map()`、`filter()`、`flatMap()`和`mapPartitions()`。

+   第四章，“Spark 中的减少”，重点介绍了减少转换（如`reduceByKey()`、`groupByKey()`和`combineByKey()`），它们在按键分组数据中起着非常重要的作用。提供了许多简单但实用的示例，确保您能有效地使用这些减少。

第二部分，“处理数据”

接下来的四章涵盖了数据分区、图算法、从/向多种不同数据源读取/写入数据以及排名算法：

+   第五章，“数据分区”，介绍了在特定数据列上物理分区数据的函数。这种分区将使您的 SQL 查询（例如在 Amazon Athena 或 Google BigQuery 中）能够分析数据的一个切片，而不是整个数据集，从而提高查询性能。

+   第六章，“图算法”，介绍了 Spark 中一个最重要的外部包，GraphFrames，它可以用于分析 Spark 分布式环境中的大型图形。

+   第七章，“与外部数据源交互”，展示了如何从各种数据源读取数据并将其写入。

+   第八章，“排名算法”，介绍了两种重要的排名算法，PageRank（用于搜索引擎）和 rank product（用于基因分析）。

第三部分，“数据设计模式”

最后四章介绍了实际数据设计模式，以实例形式呈现：

+   第九章，“经典数据设计模式”，介绍了一些基本数据设计模式或可重复使用的解决方案，通常用于解决各种数据问题。示例包括 Input-Map-Output 和 Input-Filter-Output。

+   第十章，“实用数据设计模式”，介绍了常见和实用的数据设计模式，用于组合、汇总、过滤和组织数据等任务。这些模式以实用示例的形式展示。

+   第十一章，“连接设计模式”，介绍了用于连接两个或多个数据集的简单模式；讨论了一些提高连接算法效率的性能标准。

+   第十二章，“PySpark 中的特征工程”，介绍了开发机器学习算法中最常用的特征工程技术。

附加章节

由于我不希望使本书变得过于臃肿，我在书的 [GitHub 仓库](https://github.com/mahmoudparsian/data-algorithms-with-spark) 中包含了有关 TF-IDF、相关性和 k-mer 等主题的额外材料。

# 本书使用的约定

本书使用以下排版约定：

*斜体*

指示新术语、URL、电子邮件地址、文件名和文件扩展名。

`等宽字体`

用于程序清单，以及在段落中引用程序元素，如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

**`等宽粗体`**

显示应由用户逐字输入的命令或其他文本。

*`等宽斜体`*

显示应由用户提供值或由上下文确定值的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般注释。

###### 警告

此元素表示警告或注意事项。

# 使用代码示例

补充材料（例如代码示例、练习等）可在 [*https://github.com/mahmoudparsian/data-algorithms-with-spark*](https://github.com/mahmoudparsian/data-algorithms-with-spark) 上下载。

如果您在使用代码示例时遇到技术问题或困难，请发送电子邮件至 *mahmoud.parsian@yahoo.com*。

本书旨在帮助您完成工作。一般来说，如果本书提供了示例代码，您可以在自己的程序和文档中使用它。除非您复制了大部分代码，否则无需征得我们的许可。例如，编写一个使用本书多个代码片段的程序无需许可。销售或分发 O’Reilly 书籍中的示例代码则需要许可。引用本书回答问题并引用示例代码无需许可。将本书中大量示例代码整合到产品文档中则需要许可。

我们感谢您的支持，但通常不要求归属。归属通常包括标题、作者、出版商和 ISBN。例如：“*Data Algorithms with Spark* by Mahmoud Parsian (O’Reilly). Copyright 2022 Mahmoud Parsian, 978-1-492-08238-5.”

如果您认为使用代码示例超出了合理使用范围或以上授权，请随时通过*permissions@oreilly.com*与我们联系。

# 奥莱利在线学习

###### 注意

40 多年来，[*奥莱利传媒*](http://oreilly.com)提供技术和商业培训、知识和洞见，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章、会议以及我们的在线学习平台分享他们的知识和专业知识。奥莱利的在线学习平台为您提供按需访问的实时培训课程、深入学习路径、交互式编码环境以及来自奥莱利和其他 200 多个出版商的大量文本和视频。有关更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将关于本书的评论和问题发送至出版商：

+   奥莱利传媒股份有限公司

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为本书创建了一个网页，列出勘误、示例和任何额外信息。您可以访问此页面：[*https://oreil.ly/data-algorithms-with-spark*](https://oreil.ly/data-algorithms-with-spark)。

发送电子邮件至*bookquestions@oreilly.com*以发表评论或提出关于本书的技术问题。

有关我们的图书、课程、会议和新闻的更多信息，请访问我们的网站：[*http://www.oreilly.com*](http://www.oreilly.com)。

在 LinkedIn 上找到我们：[*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media)

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

在 YouTube 上关注我们：[*http://youtube.com/oreillymedia*](http://youtube.com/oreillymedia)

# 致谢

这本书的构思是由 O'Reilly Media 的高级采编编辑 Jess Haberman 提出的。我非常感激她的联系 —— 非常感谢你，Jess！我感激 O'Reilly Media 的内容开发编辑 Melissa Potter，她从项目开始就与我密切合作，并帮助我大大改善了这本书。非常感谢你，Melissa！非常感谢编辑 Rachel Head，她在整本书的编辑工作中做得非常出色；如果你能读懂并理解这本书，那都是因为 Rachel。我要衷心感谢 Christopher Faucher（出版编辑），他做得非常好，确保了截稿时间并且一切都井然有序。推动一本书的出版绝非易事，但 Christopher 做得非常出色。

感谢技术审稿人 Parviz Deyhim 和 Benjamin Muskalla 非常仔细地审阅我的书，并就随之而来的评论、修正和建议表示感谢。我还要特别感谢我的博士导师和亲密的朋友，Ramachandran Krishnaswamy 博士，我从他身上学到了很多；我将永远珍惜与他的友谊。

为了增加 GitHub 上所有章节的 PySpark 解决方案，Deepak Kumar 和 Biman Mandal 提供了 Scala 解决方案，这对读者来说是一个很好的资源。非常感谢你们，Deepak 和 Biman。最后但同样重要的是，我要向 Apache Spark 的创始人 Matei Zaharia 博士表示巨大的感谢和敬意，他为我的书撰写了前言；我为他的友善言辞感到荣幸和自豪。
