前言

Microsoft正在运行一个Excel的反馈论坛[UserVoice](https://oreil.ly/y1XwU)，每个人都可以提交新的想法供其他人投票。最受欢迎的功能请求是“将Python作为Excel脚本语言”，它的票数大约是第二名请求的两倍。尽管自2015年提出这个想法以来并没有真正发生什么，但在2020年底，当Python的创始人Guido van Rossum [发推文](https://oreil.ly/N1_7N)说他的“退休很无聊”，并且他将加入Microsoft时，Excel用户们又充满了新的希望。他的举动是否对Excel和Python的整合有任何影响，我不知道。但我确实知道，这种组合的魅力何在，以及你如何可以今天就开始使用Excel和Python。这本书就是关于这个的简要概述。

Python背后的主要推动力是我们生活在一个数据世界中的事实。如今，巨大的数据集对每个人都是可用的，而且无所不包。通常，这些数据集非常庞大，无法再适应电子表格。几年前，这可能被称为大数据，但如今，几百万行的数据集真的不算什么了。Excel已经发展到可以应对这一趋势：它引入了Power Query来加载和清理那些无法适应电子表格的数据集，并且引入了Power Pivot，一个用于对这些数据集进行数据分析并呈现结果的附加组件。Power Query基于Power Query M公式语言（M），而Power Pivot则使用数据分析表达式（DAX）定义公式。如果你也想在Excel文件中自动化一些事情，那么你会使用Excel内置的自动化语言Visual Basic for Applications（VBA）。也就是说，对于相对简单的任务，你最终可能会使用VBA、M和DAX。其中一个问题是，所有这些语言只能在Microsoft世界中为你提供服务，尤其是在Excel和Power BI中（我将在[第1章](index_split_007.html#filepos32075)中简要介绍Power BI）。

Python，另一方面，是一种通用的编程语言，已经成为分析师和数据科学家中最受欢迎的选择之一。如果你在Excel中使用Python，你可以使用一种擅长于各个方面的编程语言，无论是自动化Excel、访问和准备数据集，还是执行数据分析和可视化任务。最重要的是，你可以在Excel之外重复使用你的Python技能：如果你需要扩展计算能力，你可以轻松地将你的定量模型、仿真或机器学习应用移至云端，那里几乎没有限制的计算资源在等待着你。

为什么我写了这本书

通过我在 xlwings 上的工作，这是我们将在本书的[第四部分](index_split_023.html#filepos1235617)中见到的 Excel 自动化包，我与许多使用 Python 处理 Excel 的用户保持密切联系 —— 无论是通过 GitHub 上的[问题跟踪器](https://oreil.ly/ZJQkB)，还是在 [StackOverflow](https://stackoverflow.com) 上的问题，或者像会议和聚会这样的实际活动中。

我经常被要求推荐入门 Python 的资源。虽然 Python 的介绍确实不少，但它们往往要么太泛泛（没有关于数据分析的内容），要么太专业（完全是科学的介绍）。然而，Excel 用户往往处于中间地带：他们确实处理数据，但完全科学的介绍可能太技术化。他们也经常有特定的需求和问题，在现有的材料中找不到答案。其中一些问题是：

+   > > > > 我需要哪个 Python-Excel 包来完成哪些任务？
+   > > > > 
+   > > > > 如何将我的 Power Query 数据库连接迁移到 Python？
+   > > > > 
+   > > > > Python 中的 AutoFilter 或数据透视表等于 Excel 中的什么？

我写这本书是为了帮助你从零开始学习 Python，以便能够自动化你与 Excel 相关的任务，并利用 Python 的数据分析和科学计算工具在 Excel 中进行操作，而不需要任何绕道。

本书的受众

如果你是一个高级的 Excel 用户，希望用现代编程语言突破 Excel 的限制，那么这本书适合你。最典型的情况是，你每个月花数小时下载、清理和复制/粘贴大量数据到关键的电子表格中。

你应该具备基本的编程理解：如果你已经写过函数或者 for 循环（无论是哪种编程语言），并且知道什么是整数或字符串，那么会有所帮助。即使你习惯编写复杂的单元格公式或有调整过的记录的 VBA 宏经验，你也可以掌握本书。不过，并不要求你有任何 Python 特定的经验，因为我们将会介绍我们将使用的所有工具，包括 Python 自身的介绍。

如果你是一个经验丰富的 VBA 开发者，你会发现 Python 和 VBA 之间的常见比较，这将帮助你避开常见的陷阱，快速上手。

如果你是一名 Python 开发者，并且需要了解 Python 处理 Excel 应用程序和 Excel 文件的不同方式，以便根据业务用户的需求选择正确的包，那么这本书也会对你有所帮助。

本书的组织结构

在本书中，我将展示 Python 处理 Excel 的所有方面，分为四个部分：

[第一部分：Python 简介](index_split_006.html#filepos31953)

> > 本部分首先探讨了为什么Python是Excel的理想伴侣的原因，然后介绍了本书中将要使用的工具：Anaconda Python发行版、Visual Studio Code和Jupyter笔记本。本部分还将教会你足够的Python知识，以便能够掌握本书的其余内容。

[第二部分：pandas介绍](index_split_013.html#filepos433190)

> > pandas是Python的数据分析首选库。我们将学习如何使用Jupyter笔记本和pandas组合来替换Excel工作簿。通常，pandas代码既更容易维护又更高效，而且您可以处理不适合电子表格的数据集。与Excel不同，pandas允许您在任何地方运行您的代码，包括云中。

[第三部分：不使用Excel读写Excel文件](index_split_018.html#filepos863198)

> > 本部分介绍了使用以下Python包之一操纵Excel文件的方法：pandas、OpenPyXL、XlsxWriter、pyxlsb、xlrd和xlwt。这些包能够直接在磁盘上读取和写入Excel工作簿，并因此取代了Excel应用程序：由于不需要安装Excel，因此它们适用于Python支持的任何平台，包括Windows、macOS和Linux。阅读器包的典型用例是从外部公司或系统每天早晨收到的Excel文件中读取数据，并将其内容存储在数据库中。写入器包的典型用例是为您在几乎每个应用程序中都可以找到的着名的“导出到Excel”按钮提供功能。

[第四部分：使用xlwings编程Excel应用程序](index_split_023.html#filepos1235617)

> > 在本部分中，我们将看到如何使用xlwings包将Python与Excel应用程序自动化，而不是在磁盘上读写Excel文件。因此，本部分需要您在本地安装Excel。我们将学习如何打开Excel工作簿并在我们眼前操纵它们。除了通过Excel读写文件之外，我们还将构建交互式Excel工具：这些工具允许我们点击按钮，使Python执行您以前可能使用VBA宏执行的一些计算机密集型计算。我们还将学习如何在Python中编写用户定义的函数[1](#filepos31598)（UDFs），而不是VBA中。

非常重要的是要理解阅读和写入Excel文件（[第三部分](index_split_018.html#filepos863198)）与编程Excel应用程序（[第四部分](index_split_023.html#filepos1235617)）之间的根本区别，如[图P-1](#filepos18540)所示。

![](images/00027.jpg)

图P-1\. 读写Excel文件（第三部分）与编程Excel（第四部分）

由于[第三部分](index_split_018.html#filepos863198)不需要安装Excel，因此在Python支持的所有平台上都可以工作，主要是Windows、macOS和Linux。然而，[第四部分](index_split_023.html#filepos1235617)只能在Microsoft Excel支持的平台上工作，即Windows和macOS，因为代码依赖于本地安装的Microsoft Excel。

Python和Excel版本

本书基于Python 3.8，这是Anaconda Python发行版的最新版本附带的Python版本。如果你想使用更新的Python版本，请按照[书籍首页](https://xlwings.org/book)上的说明操作，但要确保不使用旧版本。如果Python 3.9有变化，我会偶尔进行评论。

本书还期望你使用现代版本的Excel，至少是Windows上的Excel 2007和macOS上的Excel 2016。配有Microsoft 365订阅的本地安装版本的Excel也可以完美地工作——事实上，我甚至推荐使用它，因为它具有其他Excel版本中找不到的最新功能。这也是我撰写本书时使用的版本，所以如果你使用其他版本的Excel，可能会偶尔看到菜单项名称或位置有轻微差异。

本书使用的约定

本书使用以下排版约定：

斜体

> > 表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`常规宽度`

> > 用于程序清单，以及段落内引用的程序元素，如变量或函数名、数据库、数据类型、环境变量、语句和关键字。

`常规宽度粗体`

> > 显示用户应该按照字面意思输入的命令或其他文本。

`常规斜体`

> > 显示应由用户提供的值或由上下文确定的值替换的文本。
> > 
> 提示
> 
> 此元素表示提示或建议。
> 
> 注意
> 
> 此元素表示一般说明。
> 
> 警告
> 
> 此元素表示警告或注意事项。

使用代码示例

我在[网页](https://xlwings.org/book)上提供了额外的信息，帮助你理解这本书。确保查看，特别是如果你遇到问题时。

补充材料（代码示例、练习等）可以从[https://github.com/fzumstein/python-for-excel](https://github.com/fzumstein/python-for-excel)下载。要下载这个配套的仓库，请点击绿色的“Code”按钮，然后选择下载ZIP。下载后，在Windows上右键单击文件并选择“解压缩全部”以解压缩文件到文件夹中。在macOS上，只需双击文件即可解压缩。如果你知道如何使用Git，也可以使用Git将仓库克隆到本地硬盘上。你可以将文件夹放在任何位置，但在本书中我偶尔会按照以下方式提及它：

> `C:\Users\``username``\python-for-excel`

在 Windows 上简单下载并解压 ZIP 文件后，您会得到一个类似于以下结构的文件夹（请注意重复的文件夹名称）：

> `C:\...\Downloads\python-for-excel-1st-edition\python-for-excel-1st-edition`

将此文件夹的内容复制到您在 C:\Users\<username>\python-for-excel 下创建的文件夹中，可能会使您更容易跟进。对于 macOS，即将文件复制到 /Users/<username>/python-for-excel。

如果您有技术问题或在使用代码示例时遇到问题，请发送电子邮件至 [bookquestions@oreilly.com](mailto:bookquestions@oreilly.com)。

本书旨在帮助您完成工作任务。一般而言，如果本书提供了示例代码，您可以在自己的程序和文档中使用它。除非您要复制大部分代码，否则无需事先联系我们以获取许可。例如，编写一个使用本书多个代码片段的程序并不需要许可。出售或分发 O’Reilly 图书的示例代码则需要许可。通过引用本书并引用示例代码回答问题无需许可。将本书大量示例代码整合到产品文档中则需要许可。

我们感谢您的使用，但通常不需要署名。署名通常包括标题、作者、出版商和 ISBN。例如：“Python for Excel by Felix Zumstein (O’Reilly). Copyright 2021 Zoomer Analytics LLC, 978-1-492-08100-5.”

如果您认为您使用的代码示例超出了公平使用范围或上述许可的限制，请随时联系我们，邮箱为 [permissions@oreilly.com](mailto:permissions@oreilly.com)。

O’Reilly 在线学习

> 注意
> 
> 40 多年来，[O’Reilly Media](http://oreilly.com) 提供技术和商业培训、知识和见解，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专业知识。O’Reilly 的在线学习平台为您提供按需访问实时培训课程、深度学习路径、交互式编码环境以及来自 O’Reilly 和其他 200 多家出版商的大量文本和视频。欲了解更多信息，请访问 [http://oreilly.com](http://oreilly.com)。

如何联系我们

请就本书的评论和问题联系出版商：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（在美国或加拿大）

+   707-829-0515（国际或当地）

+   707-829-0104（传真）

我们为本书设有一个网页，其中列出勘误、示例和任何额外信息。您可以访问 [https://oreil.ly/py4excel](https://oreil.ly/py4excel)。

发送电子邮件至 [bookquestions@oreilly.com](mailto:bookquestions@oreilly.com) 以评论或提出关于本书的技术问题。

欲了解更多关于我们的书籍、课程、会议和新闻的信息，请访问我们的网站：[http://www.oreilly.com](http://www.oreilly.com)。

在Facebook上找到我们：[http://facebook.com/oreilly](http://facebook.com/oreilly)。

在Twitter上关注我们：[http://twitter.com/oreillymedia](http://twitter.com/oreillymedia)。

在YouTube上观看我们：[http://www.youtube.com/oreillymedia](http://www.youtube.com/oreillymedia)。

致谢

作为初次撰写书籍的作者，我非常感激沿途得到的许多人的帮助，他们让这段旅程对我来说变得更加轻松！

在O'Reilly，我要感谢我的编辑Melissa Potter，在保持我积极进度和帮助我将这本书变得可读方面做得很好。我还要感谢Michelle Smith，她与我一起工作在初步书籍提案上，以及Daniel Elfanbaum，他从不厌倦回答我的技术问题。

非常感谢所有投入大量时间阅读我初稿的同事、朋友和客户。他们的反馈对于使书籍更易理解至关重要，一些案例研究受到了他们与我分享的真实Excel问题的启发。我的感谢送给Adam Rodriguez、Mano Beeslar、Simon Schiegg、Rui Da Costa、Jürg Nager和Christophe de Montrichard。

我还从O'Reilly在线学习平台的早期发布版本的读者那里得到了有用的反馈。感谢Felipe Maion、Ray Doue、Kolyu Minevski、Scott Drummond、Volker Roth和David Ruggles！

我非常幸运，这本书得到了高素质的技术审阅人员的审查，我真的很感谢他们在很大时间压力下所付出的辛勤工作。感谢你们所有的帮助，Jordan Goldmeier、George Mount、Andreas Clenow、Werner Brönnimann和Eric Moreira！

特别感谢Björn Stiel，他不仅是技术审阅人员，还是我在这本书中学到许多知识的导师。这些年来，很高兴能和你一起工作！

最后但并非最不重要的，我要感谢Eric Reynolds，在2016年将他的ExcelPython项目合并到xlwings代码库中。他还从头重新设计了整个包，使我早期可怕的API成为过去。非常感谢你！

> [1  ](#filepos18058) Microsoft已开始使用术语自定义函数而不是UDF。在本书中，我将继续称它们为UDF。
