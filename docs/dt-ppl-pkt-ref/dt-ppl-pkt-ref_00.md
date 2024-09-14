# 序言

数据管道是数据分析和机器学习成功的基础。从多个不同来源移动数据并处理以提供上下文，这是拥有数据与从中获取价值之间的区别。

我在数据分析领域担任数据分析师、数据工程师和领导者已有 10 多年。在此期间，我见证了该领域的迅速变化和增长。云基础设施的出现，特别是云数据仓库，为重新思考数据管道的设计和实施创造了机会。

本书描述了我认为是在现代构建数据管道的基础和最佳实践。我基于自己的经验以及我认识和关注的行业领袖的意见和观察。

我的目标是本书既作为蓝图，又作为参考书。虽然您的需求特定于您的组织及您设定解决的问题，但我已多次通过这些基础的变体取得成功。希望您在构建和维护支撑数据组织的数据管道的旅程中找到它是一个有价值的资源。

# 本书适合的读者

本书的主要受众是现有和有抱负的数据工程师以及希望了解数据管道是什么以及如何实施的分析团队成员。他们的职称包括数据工程师、技术负责人、数据仓库工程师、分析工程师、商业智能工程师以及分析领导者/副总裁级别。

我假设您对数据仓库概念有基本的了解。要实施讨论的示例，您应该熟悉 SQL 数据库、REST API 和 JSON。您应该精通一种脚本语言，如 Python。基本了解 Linux 命令行和至少一种云计算平台也是理想的。

所有代码示例都是用 Python 和 SQL 编写，并使用许多开源库。我使用 Amazon Web Services (AWS)演示本书中描述的技术，许多代码示例中使用了 AWS 服务。在可能的情况下，我还注意到其他主要云提供商（如 Microsoft Azure 和 Google Cloud Platform (GCP)）上的类似服务。所有代码示例都可以根据您选择的云提供商进行修改，也可以用于本地使用。

# 本书中使用的约定

本书使用以下排版约定：

*斜体*

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`常量宽度`

用于程序列表，以及段落内引用程序元素，如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

**`常量宽度粗体`**

显示用户应按字面意思输入的命令或其他文本。

*`常量宽度斜体`*

显示应由用户提供的值或由上下文确定的值替换的文本。

# 使用示例代码

补充材料（示例代码、练习等）可在[*https://oreil.ly/datapipelinescode*](https://oreil.ly/datapipelinescode)下载。

如果您有技术问题或使用代码示例中的问题，请发送电子邮件至*bookquestions@oreilly.com*。

本书的目的是帮助您完成工作。一般情况下，如果本书提供示例代码，您可以在您的程序和文档中使用它。除非您复制了代码的大部分内容，否则无需征得我们的许可。例如，编写一个使用本书多个代码片段的程序无需许可。销售或分发 O'Reilly 图书的示例代码则需要许可。引用本书回答问题并引用示例代码无需许可。将本书的大量示例代码合并到您产品的文档中则需要许可。

我们感谢您的支持，但通常不需要您署名。署名通常包括标题、作者、出版商和 ISBN。例如：“*数据管道口袋参考*，作者 James Densmore（O’Reilly）。版权所有 2021 James Densmore，978-1-492-08783-0。”

如果您认为您使用的代码示例超出了公平使用范围或上述许可，请随时与我们联系：*permissions@oreilly.com*。

# O’Reilly 在线学习

###### 注意

40 多年来，[*O'Reilly Media*](http://oreilly.com)提供技术和商业培训、知识和见解，帮助企业取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专长。O’Reilly 的在线学习平台为您提供按需访问的现场培训课程、深入学习路径、交互式编码环境以及来自 O’Reilly 和 200 多家其他出版商的广泛的文本和视频内容。更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送给出版商：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（在美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们有这本书的网页，列出勘误、示例和任何额外信息。您可以访问[*https://oreil.ly/data-pipelines-pocket-ref*](https://oreil.ly/data-pipelines-pocket-ref)查看此页面。

通过电子邮件*bookquestions@oreilly.com* 发表评论或提出有关本书的技术问题。

关于我们的图书和课程的新闻和信息，请访问[*http://oreilly.com*](http://oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

观看我们的 YouTube 频道：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)

# 致谢

感谢所有在 O’Reilly 帮助这本书变为可能的人，特别是 Jessica Haberman 和 Corbin Collins。三位出色的技术审阅者 Joy Payton、Gordon Wong 和 Scott Haines 提供了宝贵的反馈，对整本书进行了关键改进。最后，感谢我的妻子阿曼达在书提出时的鼓励，以及我的狗伊齐在无数小时写作过程中一直陪伴在我身边。