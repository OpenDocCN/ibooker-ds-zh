# 前言

## 前言

我从青少年时期开始编程，从包含魔法师和乌龟漫画的有趣书籍中学习。我阅读了杂志，上面教我如何制作自己的简单游戏或在屏幕上产生愚蠢的效果。我玩得很开心。

但当我上大学时，我的书开始讨论银行账户、余额、销售部门、雇员和雇主。我怀疑我的程序员生活是否会意味着穿上灰色西装，编写处理工资系统的代码。哦，恐怖！

至少有一半的同学对编程充满热情地厌恶。我无法责怪他们。为什么编程书籍一定要如此无聊、功能性和合理？

冒险和乐趣在哪里？乐趣被低估了。如果一本书让你学习并享受学习，谁会在意它是否愚蠢和有愚蠢的笑话？

这是我写这本书的一个原因。我想让读者享受学习编程——不是通过讲笑话，而是通过解决有趣且令人愉快的编程示例。

我向你保证，不会有模拟销售部门的例子。相反，我们将做一些像模拟火箭发射、假装用古老的罗马加密技术给军队指挥官发送秘密信息，以及其他许多事情。

我想要写这本书的第二个重要原因是人们总是问我，“Julia？难道它只是一种只适合科学和科学家的语言吗？” Julia 在这个领域取得了重大成功，这就是为什么今天的 Julia 社区充满了聪明人，他们正在解决难题，比如开发新药、模拟传染病传播、气候变化或经济。

但不是的，你不需要是天才或科学家才能使用 Julia。Julia 是一种神奇的 *通用* 编程语言，适合每个人！我不是科学家，我已经享受了超过 9 年的使用它。使用 Julia，你会发现你可以比过去更快、更优雅地解决问题。而且作为额外的甜点，计算密集型代码将运行得非常快。

## 致谢

这本书经历了多次变化。曾经，它是一本自出版的书。后来，偶然的机会让我与 Manning Publications 接触，我们决定合作出版我的书。当时，我没有意识到自己将要投入多少工作。在我心中，我只是对现有的书进行一些小的修订，但根据我收到的所有反馈，我意识到我需要进行很多修订。

有时候我觉得要放弃。然而，尽管困难重重，我相信 Manning 为我们作者建立的大量辅助系统帮助我创作了一本显著更好的书。为此，我必须感谢 Nicole Butterfield，是她让我签约 Manning。我有两位 Manning 编辑：Lesley Trites，在本书早期阶段，以及 Marina Michaels，她凭借丰富的经验和稳健的手法帮助我完成了最后冲刺。我想对 Milan Ćurčić 表示感谢，他是我的技术发展编辑，他在确定材料是否对我的目标受众易懂（或不）方面提供了大量反馈。我的校对员 Christian Berk 作为非母语英语使用者对我非常宝贵，他纠正了我可能写下的任何古怪结构或语法错误。

此外，我想感谢在本书开发过程中不同阶段阅读我的手稿并提供宝贵反馈的审稿人：Alan Lenton、Amanda Debler、Andy Robinson、Chris Bailey、Daniel Kenney、Darrin Bishop、Eli Mayost、Emanuele Piccinelli、Ganesh Swaminathan、Geert Van Laethem、Geoff Barto、Ivo Balbaert、Jeremy Chen、John Zoetebier、Jonathan Owens、Jort Rodenburg、Katia Patkin、Kevin Cheung、Krzysztof Jȩdrzejewski、Louis Luangkesorn、Mark Thomas、Maura Wilder、Mike Baran、Nikos Kanakaris、Ninoslav Čerkez、Orlando Alejo Méndez Morales、Patrick Regan、Paul Silisteanu、Paul Verbeke、Samvid Mistry、Simone Sguazza、Steve Grey-Wilson、Timothy Wolodzko 和 Thomas Heiman。

特别感谢 Maurizio Tomasi，技术校对员，在本书进入生产前，他仔细地再次审查了代码。最后，感谢 Julia 的创造者。你们创造了面向未来的编程语言，我相信它将改变计算机行业。这听起来可能有些夸张，但我确实相信 Julia 是编程语言演变中的一个重要里程碑。

## 关于本书

《Julia 作为第二语言》是面向软件开发者的 Julia 编程语言入门。它不仅涵盖了语言的语法和语义，还试图通过在基于读取-评估-打印循环（REPL）的环境中进行大量交互式编码，教会读者如何像 Julia 开发者一样思考和操作。

### 谁应该阅读这本书？

《Julia 作为第二语言》是为对 Julia 编程语言感兴趣但可能没有科学或数学背景的开发者所写。对于想要探索数据科学或科学计算的人来说，本书也是一个很好的起点，因为 Julia 是为这类工作设计得非常好的语言。然而，这并不排除其他用途。任何希望用现代、高性能的语言编程以提高生产力的开发者都将从这本书中受益。

### 本书是如何组织的

本书分为五个部分，共包含 18 章。

第一部分涵盖了语言的基本知识。

+   第一章解释了 Julia 是什么样的语言，为什么它被创建，以及使用 Julia 编程语言的优势。

+   第二章讨论了在 Julia 中处理数字，展示了如何使用 Julia 的 REPL 环境作为一个非常复杂的计算器。

+   第三章通过实现三角函数和计算斐波那契数来解释控制流语句，如 if 语句、while 循环和 for 循环。

+   第四章解释了如何使用数组类型处理数字集合。读者将通过一个涉及比萨销售数据的示例来学习。

+   第五章是关于处理文本的。本章将指导你如何使用颜色制作漂亮的比萨销售数据展示，以及读取和写入比萨数据到文件。

+   第六章讨论了如何使用字典集合类型实现将罗马数字转换为十进制数字的程序。

第二部分更详细地介绍了 Julia 的类型系统。

+   第七章解释了 Julia 中的类型层次结构以及如何定义自己的复合类型。这是最重要的一章，因为它还解释了多重分派，这是 Julia 中最重要和独特的特点之一。

+   第八章介绍了一个火箭仿真代码示例，我们将在接下来的几章中使用它。本章的重点是定义不同火箭部件的类型。

+   第九章通过构建一个处理不同温度单位的代码示例，深入探讨了 Julia 中的数值转换和提升。本章有助于巩固对 Julia 多重分派系统的理解。

+   第十章解释了如何在 Julia 中表示不存在、缺失或未定义的对象。

第三部分回顾了第一部分中涵盖的集合类型，如数组、字典和字符串，但这次更深入地探讨了细节。

+   第十一章详细介绍了字符串，包括 Unicode 和 UTF-8 在 Julia 中的使用以及它们对字符串使用的影响。

+   第十二章解释了所有 Julia 集合共有的特性，例如遍历元素和构建自己的集合。

+   第十三章通过几个代码示例展示了如何在许多类型的应用程序中使用集合和集合操作来组织和搜索数据。

+   第十四章展示了如何处理和组合不同维度的数组，例如向量和矩阵。

第四部分专注于在多个级别组织代码的方法，包括从函数级别到包、文件和目录的模块化。

+   第十五章深入探讨了在 Julia 中使用函数，重点介绍了函数式编程与面向对象编程的区别。

+   第十六章是关于将代码组织到模块中，使用第三方包以及创建自己的包与他人共享代码的。

第五部分深入探讨了在缺乏前几章作为基础的情况下难以解释的细节。

+   第十七章基于第五章。通过阅读和将火箭引擎写入文件、套接字和管道（CSV 格式），你将深入了解 Julia 的 I/O 系统。

+   第十八章解释了如何定义参数化数据类型以及为什么参数化类型对性能、内存使用和正确性有益。

### 关于代码

本书包含许多源代码示例，无论是编号列表还是与正常文本并列。在这两种情况下，源代码都使用固定宽度字体进行格式化，如下所示。许多列表旁边都有代码注释，突出显示重要概念。

在许多情况下，原始源代码已经被重新格式化；我们添加了换行并重新调整了缩进，以适应书中的可用页面空间。在极少数情况下，即使这样也不够，列表中还包括了行续续标记（➥）。此外，当代码在文本中描述时，源代码中的注释通常也会从列表中删除。许多列表旁边都有代码注释，突出显示重要概念。

你编写的代码大多是在 Julia REPL（读取-评估-打印循环）环境或 Unix shell 中。在这些情况下，你会看到一个提示符，例如 julia>、shell>、help?> 或 $。当你输入时，这些提示符不应该被包含。然而，如果你将代码示例粘贴到终端窗口中，Julia 通常能够过滤掉这些提示符。

旨在写入文件的代码通常不会显示提示符。但是，如果你喜欢，通常可以将此代码粘贴到 Julia REPL 中。

你可以从本书的 liveBook（在线）版本中获取可执行的代码片段，网址为 [`livebook.manning.com/book/julia-as-a-second-language`](https://livebook.manning.com/book/julia-as-a-second-language)。本书中示例的完整代码可以从 Manning 网站 [`www.manning.com/books/julia-as-a-second-language`](https://www.manning.com/books/julia-as-a-second-language) 下载，也可以从 GitHub [`github.com/ordovician/code-samples-julia-second-language`](https://github.com/ordovician/code-samples-julia-second-language) 下载。

建议使用 Julia 版本 1.7 或更高版本来运行本书中的示例代码。

### liveBook 讨论论坛

购买 *Julia as a Second Language* 包括免费访问 liveBook，Manning 的在线阅读平台。使用 liveBook 的独家讨论功能，你可以对整本书或特定章节或段落附加评论。为自己做笔记、提问和回答技术问题以及从作者和其他用户那里获得帮助都非常简单。要访问论坛，请访问 [`livebook.manning.com/book/julia-as-a-second-language/discussion`](https://livebook.manning.com/book/julia-as-a-second-language/discussion)。你还可以了解更多关于 Manning 论坛和行为准则的信息，网址为 [`livebook.manning.com/discussion`](https://livebook.manning.com/discussion)。

曼宁对读者的承诺是提供一个场所，让读者之间以及读者与作者之间可以进行有意义的对话。这不是对作者参与特定数量活动的承诺，作者对论坛的贡献仍然是自愿的（且未付费）。我们建议你尝试向作者提出一些挑战性的问题，以免他的兴趣转移！只要这本书有售，论坛和先前讨论的存档将可通过出版社的网站访问。

### 其他在线资源

需要更多帮助？Julia 语言有一个活跃的 Slack 工作空间/社区，拥有超过 10,000 名成员，其中许多成员可以实时与你沟通。有关注册信息，请访问[`julialang.org/slack`](https://julialang.org/slack)。

+   Julia Discourse ([`discourse.julialang.org`](https://discourse.julialang.org))是关于 Julia 相关问题的首选之地。

+   [Julia 社区页面](https://julialang.org/community)提供了有关 YouTube 频道、即将举行的 Julia 活动、GitHub 和 Twitter 的信息。

+   Julia 语言和标准库的官方文档可以在[`docs.julialang.org/en/v1/`](https://docs.julialang.org/en/v1/)找到。

## 关于作者

![Engheim 作者](img/Engheim_author.png)

埃里克·恩海姆（Erik Engheim）是一位作家、会议演讲者、视频课程作者和软件开发者。他在挪威的油气行业中，大部分职业生涯都在开发用于储层建模和模拟的 3D 建模软件。埃里克还担任了多年的 iOS 和 Android 开发者。自 2013 年以来，埃里克一直在使用 Julia 编程，并撰写和制作关于 Julia 的视频。

## 关于封面插图

《Julia 作为第二语言》封面上的图像是“Paysanne Anglaise”，或“英国农妇”，取自雅克·格拉塞·德·圣索沃尔（Jacques Grasset de Saint-Sauveur）的作品集，该作品集于 1788 年出版。每一幅插图都是手工精细绘制和着色的。

在那些日子里，仅凭人们的着装就可以轻易地识别出他们的居住地以及他们的职业或社会地位。曼宁通过基于几个世纪前丰富多样的地域文化的书封面来庆祝计算机行业的创新精神和主动性，这些文化通过如这一系列图片般的生活化重现。
