# 前言

# 您将从本书中学到什么

本书将教会您有关使用 Apache Flink 进行流处理的一切。它由 11 章组成，希望能够讲述一个连贯的故事。虽然一些章节是描述性的，旨在介绍高级设计概念，其他章节更注重实操，并包含许多代码示例。

虽然我们在撰写时希望按章节顺序阅读本书，但熟悉某一章内容的读者可能会跳过它。其他更感兴趣立即编写 Flink 代码的读者可能会先阅读实用章节。以下简要描述了每章内容，以便您直接跳转到最感兴趣的章节。

+   第一章概述了有状态流处理、数据处理应用程序架构、应用程序设计以及流处理相对传统方法的优势。它还让您简要了解了在本地 Flink 实例上运行第一个流式应用程序的情况。

+   第二章讨论了流处理的基本概念和挑战，独立于 Flink。

+   第三章描述了 Flink 的系统架构和内部工作原理。它讨论了分布式架构、流式应用程序中的时间和状态处理，以及 Flink 的容错机制。

+   第四章详细说明如何设置开发和调试 Flink 应用程序的环境。

+   第五章向您介绍了 Flink DataStream API 的基础知识。您将学习如何实现 DataStream 应用程序以及支持的流转换、函数和数据类型。

+   第六章讨论了 DataStream API 的基于时间的运算符。这包括窗口操作符、基于时间的连接，以及处理函数，在处理流应用程序中处理时间时提供了最大的灵活性。

+   第七章介绍了如何实现有状态函数，并讨论了与此主题相关的一切，例如性能、健壮性以及有状态函数的演变。它还展示了如何使用 Flink 的可查询状态。

+   第八章介绍了 Flink 最常用的源和接收器连接器。它讨论了 Flink 实现端到端应用一致性的方法，以及如何实现自定义连接器从外部系统摄取数据并发送数据。

+   第九章讨论了如何在各种环境中设置和配置 Flink 集群。

+   第十章涵盖了全天候运行的流式应用程序的运维、监控和维护。

+   最后，第十一章包含资源，您可以使用这些资源提问，参加与 Flink 相关的活动，并了解 Flink 当前的使用情况。

# 本书使用的约定

本书使用以下排版约定：

*斜体*

表示新术语、网址、电子邮件地址、文件名和文件扩展名。

`Constant width`

用于程序清单，以及段落中引用程序元素，如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。还用于模块和包名称，以及显示用户应逐字键入和命令输出的文本或其他文本。

*`Constant width italic`*

显示应由用户提供值或由上下文确定值替换的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般说明。

###### 警告

此元素表示警告或注意事项。

# 使用代码示例

可以下载补充材料（Java 和 Scala 中的代码示例）[*https://github.com/streaming-with-flink*](https://github.com/streaming-with-flink)。

本书旨在帮助您完成工作。通常情况下，如果本书提供示例代码，您可以在您的程序和文档中使用它。除非您复制了代码的大部分，否则无需征得我们的许可。例如，编写一个使用本书多个代码片段的程序不需要许可。出售或分发包含 O’Reilly 书籍示例的 CD-ROM 需要许可。引用本书并引用示例代码回答问题不需要许可。将本书大量示例代码整合到产品文档中需要许可。

我们感谢但不要求署名。署名通常包括标题、作者、出版商和 ISBN 号。例如：“*使用 Apache Flink 进行流处理*，作者 Fabian Hueske 和 Vasiliki Kalavri（O’Reilly）。版权所有 2019 年 Fabian Hueske 和 Vasiliki Kalavri，978-1-491-97429-2。”

如果您认为使用示例代码超出了公平使用或上述许可的范围，请随时通过*permissions@oreilly.com*联系我们。

# O’Reilly 在线学习

###### 注意

近 40 年来，[*O’Reilly*](http://oreilly.com)为技术和商业培训提供了知识和见解，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章、会议以及我们的在线学习平台分享他们的知识和专长。O’Reilly 的在线学习平台为您提供按需访问的实时培训课程、深入学习路径、交互式编码环境以及来自 O’Reilly 和其他 200 多家出版商的大量文本和视频。欲了解更多信息，请访问[*http://oreilly.com*](http://www.oreilly.com)。

# 如何联系我们

有关本书的评论和问题，请联系出版商：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们有一本关于这本书的网页，上面列出了勘误、示例和任何额外信息。你可以访问这个页面：[*http://bit.ly/stream-proc*](http://shop.oreilly.com/product/0636920057321.do)。

如需对这本书进行评论或提出技术问题，请发送电子邮件至*bookquestions@oreilly.com*。

获取更多关于我们的书籍、课程、会议和新闻的信息，请访问我们的网站：[*http://www.oreilly.com*](http://www.oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)。

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)。

在 YouTube 上关注我们：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)。

在 Twitter 上关注作者们：[*@fhueske*](http://twitter.com/fhueske) 和 [*@vkalavri*](http://twitter.com/vkalavri)。

# 致谢

这本书要感谢和承蒙一些了不起的人的帮助和支持。我们在这里要特别感谢和承认其中的一些人。

这本书总结了 Apache Flink 社区多年来在设计、开发和测试中获得的知识。我们感谢所有通过代码、文档、审阅、错误报告、功能请求、邮件列表讨论、培训、会议演讲、聚会组织和其他活动为 Flink 做出贡献的人们。

特别感谢我们的同事 Flink 提交者们：Alan Gates, Aljoscha Krettek, Andra Lungu, ChengXiang Li, Chesnay Schepler, Chiwan Park, Daniel Warneke, Dawid Wysakowicz, Gary Yao, Greg Hogan, Gyula Fóra, Henry Saputra, Jamie Grier, Jark Wu, Jincheng Sun, Konstantinos Kloudas, Kostas Tzoumas, Kurt Young, Márton Balassi, Matthias J. Sax, Maximilian Michels, Nico Kruber, Paris Carbone, Robert Metzger, Sebastian Schelter, Shaoxuan Wang, Shuyi Chen, Stefan Richter, Stephan Ewen, Theodore Vasiloudis, Thomas Weise, Till Rohrmann, Timo Walther, Tzu-Li (Gordon) Tai, Ufuk Celebi, Xiaogang Shi, Xiaowei Jiang, Xingcan Cui。通过这本书，我们希望能够接触到全世界的开发者、工程师和流处理爱好者，并且扩大 Flink 社区的规模。

我们还要感谢我们的技术审阅人员，他们提出了无数宝贵的建议，帮助我们改进内容的呈现。感谢你们，Adam Kawa, Aljoscha Krettek, Kenneth Knowles, Lea Giordano, Matthias J. Sax, Stephan Ewen, Ted Malaska 和 Tyler Akidau。

最后，我们要衷心感谢 O'Reilly 公司的所有同仁，在我们漫长的两年半旅程中与我们同行，帮助我们将这个项目推向成功。谢谢你们，Alicia Young, Colleen Lobner, Christina Edwards, Katherine Tozer, Marie Beaugureau 和 Tim McGovern。
