# 序言

# 本书适合谁阅读？

我们为那些对数据有亲和力并希望在流处理领域改进知识和技能的软件专业人士创建了这本书，他们已经熟悉或希望使用 Apache Spark 进行他们的流应用程序。

我们已经包含了一个全面介绍流处理背后概念的部分。这些概念是理解 Apache Spark 提供的两个流 API：结构化流处理和 Spark 流处理的基础。

我们深入探讨了这些 API，并提供了从我们的经验中得出的关于其特性、应用和实际建议的见解。

除了覆盖 API 及其实际应用外，我们还讨论了每位流处理从业者工具箱中应包含的几种高级技术。

所有级别的读者都将从本书的入门部分中受益，而更有经验的专业人士将从覆盖的高级技术中获得新的见解，并将获得关于如何进一步学习的指导。

我们没有对您对 Spark 所需的知识做任何假设，但不熟悉 Spark 数据处理能力的读者应该注意，在本书中，我们专注于其流处理能力和 API。如果您希望了解 Spark 的更广泛能力和生态系统，我们推荐 Bill Chambers 和 Matei Zaharia（O’Reilly）的 *Spark: The Definitive Guide*。

本书使用的编程语言是 Scala。虽然 Spark 提供了 Scala、Java、Python 和 R 的绑定，但我们认为 Scala 是流应用程序的首选语言。尽管许多代码示例可以转换为其他语言，但某些领域，如复杂的状态计算，最好使用 Scala 编程语言。

# 安装 Spark

Spark 是由 Apache 基金会正式托管的开源项目，但主要在 [GitHub](http://bit.ly/2J5cWCL) 上进行开发。您也可以通过以下地址作为二进制预编译包下载：[*https://spark.apache.org/downloads.html*](https://spark.apache.org/downloads.html)。

从那里开始，您可以在一个或多个机器上运行 Spark，我们将在稍后解释。各大 Linux 发行版都提供了包，这些包应该有助于安装。

出于本书的目的，我们使用的示例和代码与 Spark 2.4.0 兼容，除了输出和格式细节的小改动外，这些示例应该与未来的 Spark 版本保持兼容。

请注意，Spark 是一个在 Java 虚拟机（JVM）上运行的程序，您应该在任何 Spark 组件将运行的每台机器上安装并访问它。

要安装 Java 开发工具包（JDK），[我们建议使用 OpenJDK](http://bit.ly/2GK7ym0)，它在许多系统和架构上都有打包。

您也可以 [安装 Oracle JDK](http://bit.ly/2ZMMkwe)。

Spark 和任何 Scala 程序一样，可以在安装了 JDK 6 或更高版本的任何系统上运行。Spark 的推荐 Java 运行时版本取决于其版本：

+   对于低于 2.0 版本的 Spark，建议使用 Java 7。

+   对于 2.0 及以上版本的 Spark，建议使用 Java 8。

# 学习 Scala

本书中的示例均使用 Scala 语言编写。这是核心 Spark 的实现语言，但远非唯一可用的语言；截至本文撰写时，Spark 还提供了 Python、Java 和 R 的 API。

Scala 是当今功能最完整的编程语言之一，它既提供了函数式特性又提供了面向对象的特性。然而，它的简洁性和类型推断使其语法的基本元素易于理解。

> 作为初学者语言，Scala 具有许多优点，从教学的角度来看，其正则的语法和语义是最重要的之一。
> 
> 伯恩·雷格内尔，隆德大学

因此，我们希望示例足够清晰，以便读者能够理解它们的含义。然而，对于希望通过书籍来学习语言并更为舒适的读者，我们建议使用*Atomic Scala* [[Eckel2013]](#Eckel2013)。

# 《前路》

本书分为五个部分：

+   第一部分详细阐述和深化了我们在前言中讨论的概念。我们涵盖了流处理的基本概念，实现流处理的一般架构蓝图，并深入研究了 Spark。

+   在第二部分中，我们学习了结构化流处理，其编程模型以及如何实现从相对简单的无状态转换到高级有状态操作的流应用程序。我们还讨论了它与支持全天候运营的监控工具的集成，并探讨了当前正在开发中的实验领域。

+   在第三部分中，我们学习了 Spark Streaming。与结构化流处理类似，我们学习了如何创建流应用程序，操作 Spark Streaming 作业，并将其与 Spark 中的其他 API 集成。我们在这一部分结束时还提供了性能调优的简要指南。

+   第四部分介绍了高级流处理技术。我们讨论了使用概率数据结构和近似技术来解决流处理挑战，并研究了在线机器学习在 Spark Streaming 中的有限空间。

+   最后，在第五部分中，我们将进入超越 Apache Spark 的流处理领域。我们将调查其他可用的流处理器，并提供进一步学习 Spark 和流处理的步骤。

我们建议您通过第一部分来理解支持流处理的概念。这将有助于在本书的其余部分使用统一的语言和概念。

第二部分，结构化流处理，和第三部分，Spark 流处理，遵循一致的结构。您可以选择先学习其中一个，以符合您的兴趣和最紧迫的优先事项：

+   也许您正在启动一个新项目，想了解结构化流处理？可以查看！从第二部分开始。

+   或者您可能正在进入使用 Spark 流处理的现有代码库，想更好地理解它？从第三部分开始。

第四部分 最初会深入探讨理解所讨论的概率结构所需的一些数学背景。我们喜欢把它称为“前路险峻，风景优美”。

第五部分 将把使用 Spark 进行流处理与其他可用的框架和库放在一起进行比较。这可能会帮助您在决定采用特定技术之前尝试一个或多个替代方案。

本书的在线资源通过提供笔记本和代码来补充您的学习体验，您可以自己使用和实验。或者，您甚至可以借用一段代码来启动自己的项目。在线资源位于[*https://github.com/stream-processing-with-spark*](https://github.com/stream-processing-with-spark)。

我们真诚地希望您享受阅读本书的乐趣，就像我们享受整理所有信息和捆绑体验一样。

# 参考文献

+   [Eckel2013] Eckel, Bruce 和 Dianne Marsh，*Atomic Scala*（Mindview LLC, 2013）。

+   [Odersky2016] Odersky, Martin, Lex Spoon, and Bill Venners，*Scala 编程*，第三版（Artima Press, 2016）。

# 本书使用的约定

本书采用以下排版约定：

*斜体*

表示新术语、网址、电子邮件地址、文件名和文件扩展名。

`固定宽度`

用于程序清单，以及在段落中引用程序元素，如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

**`固定宽度粗体`**

显示用户需要按照字面意思输入的命令或其他文本。

*`固定宽度斜体`*

显示应由用户提供值或根据上下文确定值替换的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般说明。

###### 警告

此元素表示警告或注意事项。

# 使用代码示例

本书的在线库包含了增强学习体验的补充材料，包括交互式笔记本、工作代码示例和一些项目，让您可以实验并获得有关所涵盖主题和技术的实用见解。它可以在[*https://github.com/stream-processing-with-spark*](https://github.com/stream-processing-with-spark)*找到。

所包含的笔记本运行在 Spark Notebook 上，这是一个专注于使用 Scala 处理 Apache Spark 的开源、基于 Web 的交互式编码环境。其实时小部件非常适合处理流应用程序，因为我们可以将数据可视化，看到它在系统中的传递过程中发生的情况。

Spark Notebook 可以在[*https://github.com/spark-notebook/spark-notebook*](https://github.com/spark-notebook/spark-notebook)*找到，并且预构建版本可以直接从它们的分发站点[*http://spark-notebook.io*](http://spark-notebook.io)*下载。

本书旨在帮助您完成工作。一般情况下，如果本书提供示例代码，您可以在您的程序和文档中使用它。除非您复制了代码的大部分，否则无需联系我们以获得许可。例如，编写一个使用本书多个代码块的程序不需要许可。出售或分发奥莱利书籍示例的 CD-ROM 需要许可。引用本书并引用示例代码回答问题不需要许可。将本书大量示例代码整合到产品文档中需要许可。

我们感谢，但不要求署名。署名通常包括标题、作者、出版商和 ISBN。例如：“*使用 Apache Spark 进行流处理*，作者 Gerard Maas 和 François Garillot（奥莱利）。版权 2019 年 François Garillot 和 Gerard Maas Images，978-1-491-94424-0。”

如果您觉得您使用的代码示例超出了公平使用或以上授予的权限，请随时通过*permissions@oreilly.com*联系我们。

# 奥莱利在线学习

###### 注意

近 40 年来，[*奥莱利传媒*](http://oreilly.com)已经为企业提供技术和商业培训、知识和洞察，帮助它们取得成功。

我们独特的专家和创新者网络通过书籍、文章、会议和我们的在线学习平台分享他们的知识和专业知识。奥莱利的在线学习平台为您提供按需访问实时培训课程、深入学习路径、交互式编码环境以及来自奥莱利和其他 200 多个出版商的大量文本和视频内容。有关更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将关于本书的评论和问题发送给出版商：

+   奥莱利传媒，公司

+   北格拉文斯坦高速公路 1005 号

+   加利福尼亚州塞巴斯托波尔 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们有一个专门为本书设立的网页，上面列出了勘误、示例和任何额外信息。您可以访问这个网页，地址是[*http://bit.ly/stream-proc-apache-spark*](http://bit.ly/stream-proc-apache-spark)。

要对本书提出评论或询问技术问题，请发送电子邮件至*bookquestions@oreilly.com*。

获取有关我们的图书、课程、会议和新闻的更多信息，请访问我们的网站：[*http://www.oreilly.com*](http://www.oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

在 YouTube 上观看我们：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)

# 致谢

这本书从最初的 Spark Streaming 学习手册，发展成为 Apache Spark 流处理能力的全面资源。我们要感谢审阅人员提供的宝贵反馈，这些反馈帮助我们将本书引导到现在的形式。我们特别感谢来自 Datastax 的 Russell Spitzer、Facebook 的 Serhat Yilmaz 和 Klarrio 的 Giselle Van Dongen。

我们要感谢 Holden Karau 在起草初期为我们提供的帮助和建议，以及 Bill Chambers 在我们增加结构化流处理覆盖范围时的持续支持。

我们在 O’Reilly 的编辑 Jeff Bleiel，在从初期想法和版本的草稿进展到您手中的内容完成过程中，一直是耐心、反馈和建议的坚强支柱。我们也要感谢 Shannon Cutt，她是我们在 O’Reilly 的第一位编辑，在启动这个项目时提供了大量帮助。O’Reilly 的其他人员在许多阶段都帮助过我们，推动我们向前迈进。

我们感谢 Tathagata Das 在 Spark Streaming 早期阶段的许多交流，特别是当我们在推动框架能够实现的极限时。

## 来自 Gerard

我要感谢我的同事在 Lightbend 的支持和理解，他们在我同时兼顾书写和工作职责时给予了支持。特别感谢 Ray Roestenburg 在困难时刻的鼓励；Dean Wampler 一直支持我在这本书中的努力；以及 Ruth Stento 在写作风格上的出色建议。

特别感谢 Kurt Jonckheer、Patrick Goemaere 和 Lieven Gesquière，他们创造了这个机会，并给了我深入了解 Spark 的空间；还要感谢 Andy Petrella 创建了 Spark Notebook，更重要的是，他的热情和激情影响了我继续探索编程和数据交汇的领域。

最重要的是，我要无限感谢我的妻子 Ingrid，我的女儿 Layla 和 Juliana，以及我的母亲 Carmen。没有她们的爱、关怀和理解，我无法完成这个项目。

## 来自 François

在撰写这本书期间，我非常感激在瑞士电信和 Facebook 的同事们的支持；感谢 Chris Fregly、Paco Nathan 和 Ben Lorica 的建议和支持；感谢我的妻子 AJung 的一切。
