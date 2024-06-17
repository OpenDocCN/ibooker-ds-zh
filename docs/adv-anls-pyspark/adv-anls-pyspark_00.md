# 序言

Apache Spark 的悠久前身，从 MPI（消息传递接口）到 MapReduce，使得编写能够利用大量资源并抽象掉分布式系统细节的程序成为可能。正如数据处理需求推动了这些框架的发展一样，在某种程度上，大数据领域与它们密切相关，其范围由这些框架能够处理的内容定义。Spark 最初的承诺是将这一点推向更远——使编写分布式程序感觉像编写常规程序一样。

Spark 的普及与 Python 数据（PyData）生态系统的普及同时发生。因此，Spark 的 Python API——PySpark，在过去几年中显著增长是合情合理的。尽管 PyData 生态系统最近推出了一些分布式编程选项，但 Apache Spark 仍然是跨行业和领域处理大型数据集的最受欢迎选择之一。由于最近努力将 PySpark 与其他 PyData 工具集成，学习这一框架可以显著提高数据科学从业者的生产力。

我们认为教授数据科学的最佳方式是通过示例。为此，我们编写了一本应用程序书籍，试图涵盖大规模分析中最常见的算法、数据集和设计模式之间的互动。本书并非用于全书阅读：请翻到一个看起来您想要完成的任务或仅仅激发您兴趣的章节或页面，从那里开始阅读。

# 为什么我们现在写这本书？

Apache Spark 在 2020 年经历了一次重大的版本升级——版本 3.0。其中最大的改进之一是引入了 Spark 自适应执行。这一特性大大简化了调优和优化的复杂性。尽管在书中我们没有提及它，因为在 Spark 3.2 及更高版本中，默认已启用，因此您将自动获得其好处。

生态系统的变化，加上 Spark 的最新主要发布，使得本版及时而至。与之前版本的《使用 Scala 进行高级分析与 Spark》不同，我们将使用 Python。我们将涵盖最佳实践，并在适当时与更广泛的 Python 数据科学生态系统集成。所有章节均已更新以使用最新的 PySpark API。新增了两个章节，并对多个章节进行了重大改写。我们不会涵盖 Spark 的流处理和图形库。随着 Spark 进入一个新的成熟和稳定的时代，我们希望这些变化能使本书作为多年来分析的有用资源得以保留。

# 本书的组织方式

[第1章](ch01.xhtml#Introduction)将Spark和PySpark置于数据科学和大数据分析的更广泛背景下。之后，每一章都包括使用PySpark的独立分析。[第2章](ch02.xhtml#introduction_to_data_anlysis_with_pyspark)通过数据清洗的用例介绍了PySpark和Python中的数据处理基础知识。接下来的几章深入探讨了使用Spark进行机器学习的核心内容，应用了一些最常见的算法于经典应用场景中。其余章节则更多地涉及稍微不同寻常的应用，例如通过文本中的潜在语义关系查询维基百科、分析基因组数据和识别相似图像。

本书不涉及PySpark的优缺点。它也不涉及其他几件事。它介绍了Spark编程模型以及Spark的Python API PySpark的基础知识。然而，它不试图成为Spark的参考书或提供所有Spark细节的全面指南。它也不试图成为机器学习、统计学或线性代数的参考书，尽管许多章节在使用它们之前提供了一些背景。

相反，这本书将帮助读者体会使用PySpark进行大型数据集上复杂分析的感觉，覆盖整个流程：不仅仅是建模和评估，还包括数据清洗、预处理和数据探索，特别关注将结果转化为生产应用程序。我们认为通过示例来教授这些是最好的方式。

这里是本书将解决的一些任务的示例：

预测森林覆盖

我们使用决策树（见[第4章](ch04.xhtml#making_predictions_with_decision_trees_and_decision_forests)）预测森林覆盖类型，使用相关特征如位置和土壤类型。

查询维基百科以查找类似条目

我们通过使用自然语言处理（NLP）技术（见[第6章](ch06.xhtml#understanding_wikipedia_with_lda_and_spark_nlp)）识别条目之间的关系并查询维基百科语料库。

理解纽约出租车的利用率

我们通过执行时间和地理空间分析（见[第7章](ch07.xhtml#geospatial_and_temporal_data_analysis_on_taxi_trip_data)）计算出租车等待时间的平均值作为位置的函数。

为投资组合减少风险

我们使用蒙特卡洛模拟（见[第9章](ch09.xhtml#analyzing_genomics_data_and_and_the_bdg_project)）为投资组合估计金融风险。

在可能的情况下，我们尝试不仅仅提供“解决方案”，而是演示完整的数据科学工作流程，包括所有的迭代、死胡同和重新启动。这本书将有助于您更熟悉Python、Spark以及机器学习和数据分析。然而，这些都是为了更大的目标服务，我们希望这本书最重要的是教会您如何处理类似前述任务。每一章，大约20页的内容，都会尽可能接近演示如何构建这些数据应用的一个部分。

# 本书使用约定

本书使用以下排版约定：

*斜体*

表示新术语、网址、电子邮件地址、文件名和文件扩展名。

`常数宽度`

用于程序清单，以及段落中引用程序元素，如变量或函数名、数据库、数据类型、环境变量、语句和关键字。

**`常数宽度粗体`**

显示用户应按字面意义键入的命令或其他文本。

*`常数宽度斜体`*

显示应使用用户提供的值或由上下文确定的值替换的文本。

此元素表示提示或建议。

此元素表示一般说明。

此元素表示警告或注意事项。

# 使用代码示例

可以下载补充材料（代码示例、练习等）位于[*https://github.com/sryza/aas*](https://github.com/sryza/aas)。

如果您在使用代码示例时遇到技术问题或问题，请发送电子邮件至[*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com)。

本书的目的是帮助您完成工作。一般情况下，如果本书提供了示例代码，您可以在自己的程序和文档中使用它。除非您要复制大量代码，否则无需事先联系我们请求许可。例如，编写一个使用本书多个代码片段的程序并不需要许可。销售或分发O’Reilly图书中的示例代码则需要许可。如果通过引用本书回答问题并引用示例代码，也不需要许可。将本书中大量示例代码整合到产品文档中，则需要许可。

我们感谢您，但不要求您进行署名。通常的署名包括标题、作者、出版社和ISBN。例如：“*使用PySpark进行高级分析* 由Akash Tandon、Sandy Ryza、Uri Laserson、Sean Owen和Josh Wills（O’Reilly）编写。版权2022 Akash Tandon，978-1-098-10365-1。”

如果您认为使用示例代码超出了合理使用或以上授权，请随时通过[*permissions@oreilly.com*](mailto:permissions@oreilly.com)联系我们。

# O’Reilly在线学习

40多年来，[*O’Reilly Media*](https://oreilly.com)一直为公司提供技术和业务培训、知识和见解，帮助它们取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专长。O’Reilly的在线学习平台为您提供按需访问实时培训课程、深入学习路径、交互式编码环境以及来自O’Reilly和其他200多个出版商的广泛文本和视频的机会。有关更多信息，请访问[*https://oreilly.com*](https://oreilly.com)。

# 如何联系我们

有关本书的评论和问题，请联系出版社：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为这本书创建了一个网页，列出勘误、示例和任何额外信息。您可以访问[*https://oreil.ly/adv-analytics-pyspark*](https://oreil.ly/adv-analytics-pyspark)获取此页面。

电子邮件[*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com)以评论或询问有关本书的技术问题。

有关我们的书籍和课程的新闻和信息，请访问[*https://oreilly.com*](https://oreilly.com)。

在LinkedIn上找到我们：[*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media)

在Twitter上关注我们：[*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia)

在YouTube上观看我们：[*https://youtube.com/oreillymedia*](https://youtube.com/oreillymedia)

# 致谢

毫无疑问，如果没有Apache Spark和MLlib的存在，您将不会阅读本书。我们都要感谢构建和开源它的团队以及为其增加功能的数百名贡献者。

我们要感谢所有花费大量时间审查本书前版本内容的专家：Michael Bernico，Adam Breindel，Ian Buss，Parviz Deyhim，Jeremy Freeman，Chris Fregly，Debashish Ghosh，Juliet Hougland，Jonathan Keebler，Nisha Muktewar，Frank Nothaft，Nick Pentreath，Kostas Sakellis，Tom White，Marcelo Vanzin和再次感谢Juliet Hougland。谢谢大家！我们欠你们一份感谢。这极大地改进了结果的结构和质量。

Sandy还要感谢Jordan Pinkus和Richard Wang对风险章节理论的帮助。

感谢Jeff Bleiel和O’Reilly为出版这本书和将其送入您手中提供的经验和大力支持。
