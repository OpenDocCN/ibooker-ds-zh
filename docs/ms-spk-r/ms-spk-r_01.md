# 前言

在信息呈指数级增长的世界中，领先的工具如 Apache Spark 提供支持，以解决今天我们面临的许多相关问题。从寻求基于数据驱动决策改进的公司，到在健康保健、金融、教育和能源领域解决问题的研究机构，Spark 使得分析更多信息比以往任何时候更快速、更可靠地实现。

已经有各种书籍写作用于学习 Apache Spark；例如，[*Spark: The Definitive Guide*](https://oreil.ly/gMaGP)是一个全面的资源，而[*Learning Spark*](https://oreil.ly/1-4CA)则是一本入门书籍，旨在帮助用户快速上手（均出自 O’Reilly）。然而，截至目前，尚无一本专门用于学习使用 R 语言进行 Apache Spark 的书籍，也没有专门为 R 用户或有意成为 R 用户的人设计的书籍。

有一些在线资源可供学习使用 R 语言学习 Apache Spark，尤其是[spark.rstudio.com](https://spark.rstudio.com)网站和 Spark 文档网站在[spark.apache.org](http://bit.ly/31H2nMl)。这两个网站都是很好的在线资源；然而，内容并不打算从头到尾阅读，并假定您作为读者对 Apache Spark、R 和集群计算有一定的了解。

本书的目标是帮助任何人开始使用 R 语言进行 Apache Spark 的学习。另外，由于 R 编程语言的创建是为了简化数据分析，我们相信本书为您提供了学习使用 Spark 解决数据分析问题的最简单途径。前几章提供了一个介绍，帮助任何人快速掌握这些概念，并介绍在您自己的计算机上处理这些问题所需的工具。然后我们迅速进入相关的数据科学主题、集群计算和即使对经验丰富的用户也感兴趣的高级主题。

因此，本书旨在成为广泛用户群的有用资源，从初学者对学习 Apache Spark 感兴趣，到有经验的读者希望了解为什么以及如何从 R 语言使用 Apache Spark。

本书的一般概述如下：

介绍

在前两章中，第一章，*介绍*，和第二章，*入门*，您将了解 Apache Spark、R 语言以及使用 Spark 和 R 进行数据分析的工具。

分析

在第三章，*分析*中，您将学习如何使用 R 语言在 Apache Spark 中分析、探索、转换和可视化数据。

建模

在第四章，*建模*和第五章，*管道*中，您将学习如何创建统计模型，目的是提取信息、预测结果，并在生产准备好的工作流程中自动化此过程。

扩展

截至目前，本书集中讨论了在个人计算机上执行操作以及使用有限数据格式。第六章，*集群*，第七章，*连接*，第八章，*数据* 和 第九章，*调整*，介绍了分布式计算技术，这些技术用于跨多台机器和数据格式执行分析和建模，以解决 Apache Spark 设计用于处理的大规模数据和计算问题。

扩展

第十章，*扩展*，描述了特定相关用例适用的可选组件和扩展功能。您将了解替代建模框架、图处理、深度学习预处理数据、地理空间分析和大规模基因组学。

高级

本书以一组高级章节结束，第十一章，*分布式 R*，第十二章，*流处理* 和 第十三章，*贡献*；这些章节对高级用户最感兴趣。然而，当您到达本节时，内容不会显得那么令人生畏；相反，这些章节与前面的章节一样相关、有用和有趣。

第一组章节，第一章–第五章，提供了在规模化数据科学和机器学习上执行温和介绍。如果您计划在阅读本书的同时按照代码示例进行操作，那么这些章节是考虑逐行执行代码的好选择。因为这些章节使用您的个人计算机教授所有概念，所以您不会利用 Spark 被设计用于使用的多台计算机。但不要担心：接下来的章节将详细教授这一点！

第二组章节，第六章–第九章，介绍了在使用 Spark 进行集群计算的令人兴奋的世界中的基本概念。说实话，它们也介绍了集群计算中一些不那么有趣的部分，但请相信我们，学习我们提出的概念是值得的。此外，每章的概述部分特别有趣、信息丰富且易于阅读，有助于您对集群计算的工作原理形成直观的理解。对于这些章节，我们实际上不建议逐行执行代码——特别是如果您试图从头到尾学习 Spark。在您拥有适当的 Spark 集群之后，您随时可以回来执行代码。然而，如果您在工作中已经有了一个集群，或者您真的很有动力想要得到一个集群，您可能希望使用 第六章 来选择一个，然后使用 第七章 连接到它。

第三组章节，10–13，介绍了对大多数读者都很有趣的工具，将有助于更好地跟踪内容。这些章节涵盖了许多高级主题，自然而然地，你可能对某些主题更感兴趣；例如，你可能对分析地理数据集感兴趣，或者你可能更喜欢处理实时数据集，甚至两者兼顾！根据你的个人兴趣或手头的问题，我们鼓励你执行最相关的代码示例。这些章节中的所有代码都是为了在你的个人计算机上执行而编写的，但是我们也鼓励你使用适当的 Spark 集群，因为你将拥有解决问题和调优大规模计算所需的工具。

# 格式化

从代码生成的表格的格式如下：

```
# A tibble: 3 x 2
  numbers text
    <dbl> <chr>
1       1 one
2       2 two
3       3 three
```

表格的尺寸（行数和列数）在第一行描述，接着是第二行的列名和第三行的列类型。此外，我们在整本书中还使用了 `tibble` 包提供的各种微小视觉改进。

大多数图表使用 `ggplot2` 包及附录中提供的自定义主题进行渲染；然而，由于本书不侧重数据可视化，我们仅提供了一个基本绘图的代码，可能与我们应用的格式不符合。如果你有兴趣学习更多关于 R 中可视化的内容，可以考虑专门的书籍，比如 [*R Graphics Cookbook*](https://oreil.ly/bIF4l)（O'Reilly）。

# 致谢

感谢使 Spark 与 R 集成的包作者们：Javier Luraschi、Kevin Kuo、Kevin Ushey 和 JJ Allaire（`sparklyr`）；Romain François 和 Hadley Wickham（`dbplyr`）；Hadley Wickham 和 Edgar Ruiz（`dpblyr`）；Kirill Mülller（`DBI`）；以及 Apache Spark 项目本身的作者及其原始作者 Matei Zaharia。

感谢那些发布扩展以丰富 Spark 和 R 生态系统的包作者们：Akhil Nair（`crassy`）；Harry Zhu（`geospark`）；Kevin Kuo（`graphframes`、`mleap`、`sparktf` 和 `sparkxgb`）；Jakub Hava、Navdeep Gill、Erin LeDell 和 Michal Malohlava（`rsparkling`）；Jan Wijffels（`spark.sas7bdat`）；Aki Ariga（`sparkavro`）；Martin Studer（`sparkbq`）；Matt Pollock（`sparklyr.nested`）；Nathan Eastwood（`sparkts`）；以及 Samuel Macêdo（`variantspark`）。

感谢我们出色的编辑 Melissa Potter，她为我们提供了指导、鼓励和无数小时的详细反馈，使这本书成为我们能够写出的最好的作品。

致谢 Bradley Boehmke、Bryan Adams、Bryan Jonas、Dusty Turner 和 Hossein Falaki，感谢你们的技术审查、时间和坦诚反馈，以及与我们分享专业知识。多亏了你们，许多读者将会有更愉快的阅读体验。

感谢 RStudio、JJ Allaire 和 Tareef Kawaf 支持这项工作，以及 R 社区本身对其持续的支持和鼓励。

Max Kuhn，感谢您在第四章中对模型的宝贵反馈，我们在此章节中经过他的允许，从他精彩的书籍*Feature Engineering and Selection: A Practical Approach for Predictive Models*（CRC Press）中改编了示例。

我们也要感谢那些间接参与但未在本节明确列出的每个人；我们确实站在巨人的肩膀上。

本书本身是使用`bookdown`（由谢益辉开发）、`rmarkdown`（由 JJ Allaire 和谢益辉开发）以及`knitr`（由谢益辉开发）编写的，我们使用`ggplot2`（由 Hadley Wickham 和 Winston Chang 开发）绘制了可视化，使用`nomnoml`（由 Daniel Kallin 和 Javier Luraschi 开发）创建了图表，并使用`pandoc`（由 John MacFarlane 开发）进行了文档转换。

# 本书使用的约定

本书使用了以下排版约定：

*斜体*

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`等宽`

用于程序清单以及段落内引用程序元素，例如变量或函数名、数据库、数据类型、环境变量、语句和关键字。

**`等宽粗体`**

显示用户应按原样输入的命令或其他文本。

*`等宽斜体`*

显示应替换为用户提供值或由上下文确定值的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般说明。

# 使用代码示例

补充材料（代码示例、练习等）可从*https://github.com/r-spark/the-r-in-spark*下载。

本书旨在帮助您完成工作。一般情况下，如果本书提供了示例代码，您可以在您的程序和文档中使用它。除非您要复制代码的大部分，否则无需征得我们的许可。例如，编写一个使用本书多个代码块的程序不需要许可。售卖或分发包含 O’Reilly 书籍示例的 CD-ROM 则需要许可。引用本书并引述示例代码以回答问题也不需要许可。将本书中大量示例代码整合到产品文档中则需要许可。

我们感谢，但不要求署名。署名通常包括书名、作者、出版商和 ISBN。例如：“*Mastering Spark with R* by Javier Luraschi, Kevin Kuo, and Edgar Ruiz (O’Reilly). Copyright 2020 Javier Luraschi, Kevin Kuo, and Edgar Ruiz, 978-1-492-04637-0.”

如果您认为您对代码示例的使用超出了合理使用范围或上述许可的限制，请随时通过*permissions@oreilly.com*联系我们。

# O’Reilly 在线学习

###### 注意

近 40 年来，[*O’Reilly Media*](http://oreilly.com)一直致力于为公司提供技术和商业培训、知识和见解，帮助它们取得成功。

我们独特的专家和创新者网络通过图书、文章、会议和我们的在线学习平台分享他们的知识和专业知识。O’Reilly 的在线学习平台让您随时访问现场培训课程、深度学习路径、交互式编码环境以及来自 O’Reilly 和其他 200 多家出版商的大量文本和视频。欲了解更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送至出版商：

+   O’Reilly Media，Inc.

+   Gravenstein Highway North，1005 号

+   加利福尼亚州，Sebastopol，95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为本书设有网页，列出勘误、示例和任何额外信息。您可以访问此页面[*https://oreil.ly/SparkwithR*](https://oreil.ly/SparkwithR)。

要就本书发表评论或提出技术问题，请发送电子邮件至*bookquestions@oreilly.com*。

关于我们的图书、课程、会议和新闻的更多信息，请访问我们的网站[*http://www.oreilly.com*](http://www.oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

在 YouTube 上关注我们：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)
