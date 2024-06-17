# 第二版序言

我对《从零开始的数据科学》第一版感到异常自豪。它确实成为了我想要的那本书。但是，在数据科学的几年发展中，在Python生态系统的进步以及作为开发者和教育者的个人成长中，改变了我认为第一本数据科学入门书应该是什么样子的。

在生活中，没有重来的机会。但在写作中，有第二版。

因此，我重新编写了所有代码和示例，使用了Python 3.6（以及其新引入的许多功能，如类型注解）。我强调编写清晰代码的理念贯穿整本书。我用“真实”数据集替换了第一版中的一些玩具示例。我添加了关于深度学习、统计学和自然语言处理等主题的新材料，这些都是今天的数据科学家可能正在处理的内容。（我还删除了一些似乎不太相关的内容。）我仔细检查了整本书，修复了错误，重写了不够清晰的解释，并更新了一些笑话。

第一版是一本很棒的书，而这一版更好。享受吧！

+   Joel Grus

+   华盛顿州西雅图

+   2019

# 本书使用的约定

本书使用以下排版约定：

*斜体*

指示新术语、网址、电子邮件地址、文件名和文件扩展名。

`等宽字体`

用于程序清单，以及段落内指代程序元素（如变量或函数名、数据库、数据类型、环境变量、语句和关键字）。

**`等宽粗体`**

显示用户应该按照字面意思输入的命令或其他文本。

*`等宽斜体`*

显示应该用用户提供的值或上下文确定的值替换的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般注释。

###### 警告

此元素表示警告或注意事项。

# 使用代码示例

补充材料（代码示例、练习等）可在[*https://github.com/joelgrus/data-science-from-scratch*](https://github.com/joelgrus/data-science-from-scratch)下载。

本书旨在帮助您完成工作。一般情况下，如果本书提供示例代码，您可以在自己的程序和文档中使用它们。除非您要复制大量代码，否则无需征得我们的许可。例如，编写一个使用本书多个代码片段的程序不需要许可。销售或分发包含O'Reilly书籍示例的CD-ROM需要许可。通过引用本书并引用示例代码回答问题不需要许可。将本书大量示例代码纳入产品文档中需要许可。

我们感谢您的评价，但不要求。署名通常包括标题、作者、出版商和ISBN。例如：“*从零开始的数据科学*，第二版，作者 Joel Grus（奥莱利）。版权所有 2019 年 Joel Grus，978-1-492-04113-9。”

如果您认为您使用的代码示例超出了合理使用范围或上述权限，请随时与我们联系：[*permissions@oreilly.com*](mailto:permissions@oreilly.com)。

# [奥莱利在线学习](http://oreilly.com)

###### 注意

近 40 年来，[*奥莱利媒体*](http://oreilly.com)一直为企业提供技术和商业培训、知识和见解，帮助它们取得成功。

我们独特的专家和创新者网络通过书籍、文章、会议以及我们的在线学习平台分享他们的知识和专业知识。奥莱利的在线学习平台为您提供按需访问的实时培训课程、深度学习路径、交互式编程环境以及来自奥莱利和其他 200 多家出版商的大量文本和视频内容。欲了解更多信息，请访问 [*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送给出版社：

+   奥莱利媒体公司

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为这本书建立了一个网页，列出勘误、示例和任何其他信息。您可以访问 [*http://bit.ly/data-science-from-scratch-2e*](http://bit.ly/data-science-from-scratch-2e)。

如有关于本书的评论或技术问题，请发送电子邮件至 [*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com)。

有关我们的图书、课程、会议和新闻的更多信息，请访问我们的网站：[*http://www.oreilly.com*](http://www.oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

观看我们的 YouTube 视频：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)

# 致谢

首先，我要感谢 Mike Loukides 接受了我关于这本书的提案（并坚持让我将其减少到一个合理的大小）。他完全可以说：“这个人一直给我发样章，我怎么让他走开？”我很感激他没有那样做。我还要感谢我的编辑 Michele Cronin 和 Marie Beaugureau，他们指导我完成出版过程，使这本书达到了一个比我一个人写得更好的状态。

如果不是因为 Dave Hsu、Igor Tatarinov、John Rauser 及 Farecast 团队的影响，我可能永远不会学习数据科学，也就无法写出这本书。（当时甚至还没有称之为数据科学！）Coursera 和 DataTau 的好人们也值得称赞。

我也非常感谢我的试读者和评论者。Jay Fundling 找出了大量错误，并指出了许多不清晰的解释，多亏他，这本书变得更好（也更正确）。Debashis Ghosh 对我所有的统计进行了合理性检查，他是个英雄。Andrew Musselman 建议我减少书中“喜欢 R 而不喜欢 Python 的人是道德败类”的内容，我觉得这个建议非常中肯。Trey Causey、Ryan Matthew Balfanz、Loris Mularoni、Núria Pujol、Rob Jefferson、Mary Pat Campbell、Zach Geary、Denise Mauldin、Jimmy O’Donnell 和 Wendy Grus 也提供了宝贵的反馈。感谢所有读过第一版并帮助改进这本书的人。当然，任何剩下的错误都是我个人的责任。

我非常感谢 Twitter 的 #datascience 社区，让我接触到大量新概念，认识了很多优秀的人，让我感觉自己像个低成就者，于是我写了一本书来弥补。特别感谢 Trey Causey（再次），他（无意间地）提醒我要包括一个线性代数章节，以及 Sean J. Taylor，他（无意间地）指出了“处理数据”章节中的一些重大空白。

最重要的是，我要无比感谢 Ganga 和 Madeline。写书比写书的人更难，没有他们的支持我无法完成这本书。
