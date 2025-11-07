# 前言

## 前言

现在，世界上充斥着大量的数据分析软件工具。读者可能会想，为什么是 *Julia for Data Analysis*？这本书回答了“为什么”和“如何”的问题。

由于读者可能不熟悉我，我想自我介绍一下。我是 Julia 语言的创造者之一，同时也是 Julia Computing 的联合创始人和首席执行官。我们创立 Julia 语言的想法很简单——创建一种既像 C 语言一样快速，又像 R 和 Python 一样容易使用的语言。这个简单的想法在许多不同的领域产生了巨大的影响，因为 Julia 社区围绕它建立了一套精彩的抽象和基础设施。Bogumił 与许多共同贡献者一起，为数据分析构建了一个高性能且易于使用的包生态系统。

现在，你可能想知道，为什么还需要另一个库？Julia 的数据分析生态系统是从底层构建的，利用了 Julia 本身的一些基本思想。这些库是“完全 Julia”，意味着它们完全用 Julia 实现——用于处理数据的 DataFrames.jl 库、用于读取数据的 CSV.jl 库、用于统计分析的 JuliaStats 生态系统等等。这些库基于在 R 中开发的思想并进一步发展。例如，在 Julia 中处理缺失数据的基础设施是 Julia 生态系统的一个核心部分。它花费了多年时间才得到正确处理，并使 Julia 编译器高效，以减少处理缺失数据的开销。完全本地的 DataFrames.jl 库意味着你不再需要局限于矢量化编码风格来实现高性能数据分析。你可以简单地编写针对多吉字节数据集的 for 循环，使用多线程进行并行数据处理，与 Julia 生态系统中的计算库集成，甚至将这些作为 Web API 部署，供其他系统使用。本书中展示了所有这些功能。我在这本书中真正喜欢的一点是，Bogumił 向读者介绍的一些例子不仅整洁、小巧、表格化，而且是真实世界的数据——例如，一个包含 200 万行的棋盘游戏集！

本书分为两部分。第一部分介绍了 Julia 语言的基礎概念，包括类型系统、多重分派、数据结构等。第二部分在此基础上进一步阐述，并介绍数据分析——读取数据、选择、创建 DataFrame、分割-应用-组合、排序、连接和重塑，最后以一个完整的应用程序结束。书中还讨论了 Arrow 数据交换格式，它允许 Julia 程序与 R、Python 和 Spark 等数据分析工具共存。书中所有章节中的代码模式都教授读者良好的实践，这些实践有助于实现高性能数据分析。

Bogumił 不仅是对 Julia 数据分析和统计生态系统的重大贡献者，而且还构建了几个课程（如 JuliaAcademy 上的课程）并广泛地博客关于这些包的内部结构。因此，他是介绍 Julia 如何有效用于数据分析的最佳作者之一。

——Viral Shah，Julia Computing 的联合创始人兼首席执行官

## 前言

我从 2014 年开始使用 Julia 语言。在此之前，我主要使用 R 进行数据分析（那时 Python 在该领域还不够成熟）。然而，除了探索数据和构建机器学习模型之外，我经常需要实现定制的计算密集型代码，这需要花费数天时间来完成计算。我主要使用 C 或 Java 来处理这类应用。不断在编程语言之间切换是一件痛苦的事情。

在我了解到 Julia 之后，我立刻感觉到它是一项符合我需求、令人兴奋的技术。即使在它的早期阶段（在 1.0 版本发布之前），我也能成功地在我的项目中使用它。然而，就像每一样新工具一样，它仍然需要被完善。

然后，我决定开始为 Julia 语言及其与数据管理功能相关的包做出贡献。多年来，我的关注点发生了变化，最终我成为了 DataFrames.jl 包的主要维护者之一。我相信 Julia 现在已经准备好用于严肃的应用，DataFrames.jl 已经达到了稳定状态，并且功能丰富。因此，我决定写这本书，分享我使用 Julia 进行数据分析的经验。

我一直认为，软件不仅要提供出色的功能，还要提供足够的文档。因此，多年来我一直在维护这些在线资源：Julia 快车 ([`github.com/bkamins/The-Julia-Express`](https://github.com/bkamins/The-Julia-Express))，这是一份关于 Julia 语言的快速入门教程；DataFrames.jl 简介 ([`github.com/bkamins/Julia-DataFrames-Tutorial`](https://github.com/bkamins/Julia-DataFrames-Tutorial))，一组 Jupyter 笔记本集合；以及关于 Julia 的每周博客 ([`bkamins.github.io/`](https://bkamins.github.io/))。此外，去年 Manning 邀请我准备 *Hands-On Data Science with Julia* liveProject ([`www.manning.com/liveprojectseries/data-science-with-julia-ser`](https://www.manning.com/liveprojectseries/data-science-with-julia-ser))，这是一套涵盖常见数据科学任务的练习。

在编写了所有这些教学材料之后，我强烈感觉到拼图中还缺少一块。那些想要用 Julia 开始进行数据科学的人很难找到一本能够逐步介绍他们所需基础知识的书籍，以便使用 Julia 进行数据分析。这本书填补了这一空白。

Julia 生态系统有数百个包可用于您的数据科学项目，并且每天都有新的包被注册。这本书的目标是教授 Julia 的最重要的特性和一些用户在进行数据分析时会发现有用的精选流行包。在阅读本书后，您应该能够独立完成以下任务：

+   使用 Julia 进行数据分析。

+   学习由专业包提供的功能，这些功能超越了数据分析，并在进行数据科学项目时很有用。附录 C 提供了我推荐的 Julia 生态系统中的工具概述，按应用领域分类。

+   舒适地学习 Julia 的更高级方面，这些方面对包开发者来说相关重要。

+   从社交媒体上关于 Julia 的讨论中受益，例如 Discourse ([`discourse.julialang.org/`](https://discourse.julialang.org/))、Slack ([`julialang.org/slack/`](https://julialang.org/slack/)) 和 Zulip (https://julialang.zulipchat.com/register/)），自信地理解其他用户在评论中引用的关键概念和术语。

## 致谢

这本书是我与 Julia 语言旅程的重要部分。因此，我想感谢许多帮助过我的人。

让我先感谢那些我从他们那里学到很多并从中获得灵感的 Julia 社区成员。他们的名字太多，难以一一列举，所以我不得不艰难地选择其中几位。在我早期，Stefan Karpinski 在我支持他塑造 Julia 字符串处理功能时，帮助我很多，让我作为 Julia 贡献者入门。在数据科学生态系统中，Milan Bouchet-Valat 已经是我多年的重要合作伙伴。他对 Julia 数据和统计生态系统的维护工作是无价的。我从他那里学到最重要的东西是对细节的关注以及考虑包维护者做出的设计决策的长期后果。下一个关键人物是 Jacob Quinn，他设计了并实现了我在本书中讨论的许多功能。最后，我想提到 Peter Deffebach 和 Frames Catherine White，他们都是 Julia 数据分析生态系统的重大贡献者，并且总是愿意从包用户的视角提供宝贵的评论和建议。

我还想感谢我的编辑 Marina Michaels，技术编辑 Chad Scherrer，以及技术校对 German Gonzalez-Morris，以及那些在本书开发的不同阶段花时间阅读我的手稿并提供宝贵反馈的审稿人：Ben McNamara，Carlos Aya-Moreno，Clemens Baader，David Cronkite，Dr. Mike Williams，Floris Bouchot，Guillaume Alleon，Joel Holmes，Jose Luis Manners，Kai Gellien，Kay Engelhardt，Kevin Cheung，Laud Bentil，Marco Carnini，Marvin Schwarze，Mattia Di Gangi，Maureen Metzger，Maxim Volgin，Milan Mulji，Neumann Chew，Nikos Tzortzis Kanakaris，Nitin Gode，Orlando Méndez Morales，Patrice Maldague，Patrick Goetz，Peter Henstock，Rafael Guerra，Samuel Bosch，Satej Kumar Sahu，Shiroshica Kulatilake，Sonja Krause-Harder，Stefan Pinnow，Steve Rogers，Tom Heiman，Tony Dubitsky，Wei Luo，Wolf Thomsen，以及 Yongming Han。最后，感谢整个 Manning 团队，他们在本书的生产和推广过程中与我合作：Deirdre Hiam，我的项目经理；Sharon Wilkey，我的校对编辑；以及 Melody Dolab，我的页面校对员。

最后，我想对我的科学合作者表示感激，特别是 Tomasz Olczak，Paweł Prałat，Przemysław Szufel，以及 François Théberge，我们共同使用 Julia 语言发表了多篇论文。

## 关于本书

本书分为两部分，旨在帮助你开始使用 Julia 进行数据分析。它首先解释了 Julia 在此类应用中最重要的一些特性。接下来，它讨论了在数据科学项目中使用的选定核心包的功能。

这份材料围绕完整的数据分析项目构建，从数据收集开始，经过数据转换，最终以可视化和构建基本预测模型结束。我的目标是教会你在任何数据科学项目中都有用的基本概念和技能。

本书不需要你具备高级机器学习算法的先验知识。这些知识对于理解 Julia 中数据分析的基础并不必要，而且我在本书中也没有讨论此类模型。我假设你了解基本的数据科学工具和技术，例如广义线性回归或 LOESS 回归。同样，从数据工程的角度来看，我涵盖了最常用的操作，包括从网络获取数据、编写网络服务、处理压缩文件和使用基本的数据存储格式。我排除了需要额外复杂配置（与 Julia 无关）或专业软件工程知识的特性。

附录 C 回顾了在数据工程和数据分析领域提供高级功能的 Julia 包。通过本书中你获得的知识，你应该能够自信地独立学习使用这些包。

### 应该阅读本书的人

本书面向希望了解如何使用 Julia 进行数据分析的数据科学家或数据工程师。我假设你有一些使用 R、Python 或 MATLAB 等编程语言进行数据分析的经验。

### 本书是如何组织的：路线图

本书分为两部分，共有 14 章和三个附录。

第一章概述了 Julia，并解释了为什么它是数据科学项目的优秀语言。

第一部分的章节如下，教授你在数据分析项目中非常有用的基本 Julia 技能。这些章节对于不太熟悉 Julia 语言的读者来说是必不可少的。然而，我预计即使是使用 Julia 的人也会在这里找到有用的信息，因为我选择讨论的主题是基于常见困难问题的。这部分的目的不是提供一个完整的 Julia 语言介绍，而是从数据科学项目的实用性角度来编写的。第一部分的章节包括：

+   第二章介绍了 Julia 的语法基础和常见的语言结构，以及变量作用域规则最重要的方面。

+   第三章介绍了 Julia 的类型系统和方法。它还介绍了如何使用包和模块，最后讨论了宏的使用。

+   第四章涵盖了处理数组、字典、元组和命名元组。

+   第五章讨论了与 Julia 中集合操作相关的高级主题，包括参数化类型的广播和子类型规则。它还涵盖了将 Julia 与 Python 集成的相关内容。

+   第六章教你如何在 Julia 中处理字符串。此外，它还涵盖了使用符号、处理固定宽度字符串以及使用 PooledArrays.jl 包压缩向量的主题。

+   第七章专注于处理时间序列数据和缺失值。它还涵盖了使用 HTTP 查询获取数据以及解析 JSON 数据。

在第二部分，你将学习如何在 DataFrames.jl 包的帮助下构建数据分析管道。虽然，在一般情况下，你可以仅使用第一部分中学习的数据结构进行数据分析，但通过使用数据表构建你的数据分析工作流程将更加容易，同时也能确保你的代码效率。以下是第二部分你将学习的内容：

+   第八章教你如何从 CSV 文件创建数据表，并在数据表上执行基本操作。它还展示了如何在 Apache Arrow 和 SQLite 数据库中处理数据，处理压缩文件以及进行基本的数据可视化。

+   第九章展示了如何从数据表中选择行和列，你还将学习如何构建和可视化局部估计散点图平滑（LOESS）回归模型。

+   第十章涵盖了创建新数据帧以及用新数据填充现有数据帧的各种方法。它讨论了 Tables.jl 接口，这是一个表概念的实现无关抽象。你还将学习如何将 Julia 与 R 集成以及序列化 Julia 对象。

+   第十一章教你如何将数据帧转换为其他类型的对象。其中一种基本类型是分组数据帧。你还将了解类型稳定的代码和类型盗用等重要通用概念。

+   第十二章专注于数据帧对象的转换和变异——特别是使用拆分-应用-组合策略。此外，本章还涵盖了使用 Graphs.jl 包处理图数据的基础知识。

+   第十三章讨论了 DataFrames.jl 包提供的先进数据帧转换选项，以及数据帧排序、连接和重塑。它还教你如何在数据处理管道中链式执行多个操作。从数据科学的角度来看，本章展示了如何在 Julia 中处理分类数据并评估分类模型。

+   第十四章展示了如何在 Julia 中构建一个提供由分析算法生成数据的 Web 服务。此外，它还展示了如何通过利用 Julia 的多线程功能来实施蒙特卡洛模拟并使其运行更快。

本书以三个附录结束。附录 A 提供了关于 Julia 的安装和配置以及与 Julia 工作相关的常见任务的基本信息，特别是包管理。附录 B 包含了章节中提出的练习的答案。附录 C 对 Julia 包生态系统进行了回顾，这对于你的数据科学和数据工程项目将非常有用。

### 关于代码

本书包含许多源代码示例，既有编号列表，也有与普通文本混排。在两种情况下，源代码都以固定宽度字体格式化，如这样，以将其与普通文本区分开来。有时代码也会被**加粗**，以突出显示本章中从先前步骤中更改的代码，例如当新功能添加到现有代码行时。

此外，当在文本中描述代码时，源代码中的注释通常已从列表中删除。代码注释伴随着许多列表，突出显示重要概念。

本书使用的所有代码都可在 GitHub 上找到，网址为[`github.com/bkamins/JuliaForDataAnalysis`](https://github.com/bkamins/JuliaForDataAnalysis)。代码示例旨在在终端的交互会话中执行。因此，在书中，在大多数情况下，代码块都显示了带有 julia>提示符的 Julia 输入和命令下面的输出。这种风格与你的终端显示相匹配。以下是一个示例：

```
julia> 1 + 2       ❶
3                  ❷
```

❶ 1 + 2 是用户执行的 Julia 代码。

❷ 3 是 Julia 在终端中打印的输出。

本书展示的所有材料都可以在 Windows、macOS 或 Linux 上运行。你应该能够在拥有 8GB RAM 的机器上运行所有示例。然而，一些代码列表需要更多的 RAM；在这些情况下，我在书中给出了警告。

如何运行本书中展示的代码

为了确保书中展示的所有代码在你的机器上正确运行，首先遵循附录 A 中描述的配置步骤是至关重要的。

本书是用 Julia 1.7 编写的并进行了测试。

一个特别重要的点是，在运行示例代码之前，你应该始终激活书中 GitHub 仓库提供的项目环境，网址为[`github.com/bkamins/JuliaForDataAnalysis`](https://github.com/bkamins/JuliaForDataAnalysis)。

尤其重要的是，在运行示例代码之前，你应该始终激活书中 GitHub 仓库提供的项目环境，网址为[`github.com/bkamins/JuliaForDataAnalysis`](https://github.com/bkamins/JuliaForDataAnalysis)。

书中展示的代码不是通过复制粘贴到你的 Julia 会话中执行。始终使用你在本书 GitHub 仓库中可以找到的代码。对于每一章，仓库都有一个单独的文件，包含该章的所有代码。

### liveBook 讨论论坛

购买《Julia for Data Analysis》包括对 Manning 在线阅读平台 liveBook 的免费访问。使用 liveBook 的独家讨论功能，你可以对整本书、特定章节或段落添加评论。为自己做笔记、提出和回答技术问题、从作者和其他用户那里获得帮助都非常简单。要访问论坛，请访问[`livebook.manning.com/book/julia-for-data-analysis/discussion`](https://livebook.manning.com/book//julia-for-data-analysis/discussion)。你还可以在[`livebook.manning.com/discussion`](https://livebook.manning.com/discussion)了解更多关于 Manning 论坛和行为准则的信息。

Manning 对我们读者的承诺是提供一个场所，在那里个人读者之间以及读者与作者之间可以进行有意义的对话。这不是对作者参与特定数量承诺的承诺，作者对论坛的贡献仍然是自愿的（且未付费）。我们建议你尝试向作者提出一些挑战性的问题，以免他的兴趣转移！只要本书有售，论坛和以前讨论的存档将从出版社的网站提供访问。

### 其他在线资源

以下是在阅读本书时你可能觉得有用的精选在线资源列表：

+   DataFrames.jl 文档([`dataframes.juliadata.org/stable/`](https://dataframes.juliadata.org/stable/))，包含教程链接

+   *使用 Julia 进行动手数据科学* liveProject ([`www.manning.com/liveprojectseries/data-science-with-julia-ser`](https://www.manning.com/liveprojectseries/data-science-with-julia-ser))，这是一项设计为阅读本书后的后续资源，你可以用它来测试你的技能，并学习如何使用 Julia 进行高级机器学习模型。

+   我的每周博客 ([`bkamins.github.io/`](https://bkamins.github.io/))，我在那里写有关 Julia 语言的文章

此外，还有许多关于 Julia 的一般信息的有价值来源。以下是其中一些最受欢迎的选择：

+   Julia 语言网站 ([`julialang.org`](https://julialang.org))

+   JuliaCon 会议 ([`juliacon.org`](https://juliacon.org))

+   Discourse ([`discourse.julialang.org`](https://discourse.julialang.org))

+   Slack ([`julialang.org/slack/`](https://julialang.org/slack/))

+   Zulip ([`julialang.zulipchat.com/register/`](https://julialang.zulipchat.com/register/))

+   Forem ([`forem.julialang.org`](https://forem.julialang.org))

+   Stack Overflow ([`stackoverflow.com/questions/tagged/julia`](https://stackoverflow.com/questions/tagged/julia))

+   Julia YouTube 频道 ([www.youtube.com/user/julialanguage](https://www.youtube.com/user/julialanguage))

+   Talk Julia 播客 ([www.talkjulia.com](https://www.talkjulia.com/))

+   JuliaBloggers 博客聚合器 ([`www.juliabloggers.com`](https://www.juliabloggers.com))

## 关于作者

![Kaminski](img/Kaminski.png)

Bogumił Kamiński 是 DataFrames.jl 的主要开发者，这是 Julia 生态系统中进行数据处理的核心包。他在为商业客户提供数据科学项目方面拥有超过 20 年的经验。Bogumił 在本科和研究生层次上教授数据科学也超过 20 年。

## 关于封面插图

《Julia for Data Analysis》封面上的图像是“Prussienne de Silésie”，或“西里西亚的普鲁士人”，取自 Jacques Grasset de Saint-Sauveur 的收藏，该收藏于 1797 年出版。每一幅插图都是手工精细绘制和着色的。

在那些日子里，人们通过他们的服饰很容易就能识别出他们住在哪里，他们的职业或社会地位是什么。Manning 通过基于几个世纪前丰富多样的地区文化的书封面来庆祝计算机行业的创新和主动性，这些文化通过像这样的收藏品中的图片被重新带回生活。
