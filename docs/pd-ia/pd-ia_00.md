# 前置材料

## 前言

说实话，我是完全偶然发现 pandas 的。

2015 年，我在世界上最大的招聘网站 Indeed.com 应聘数据运营分析师职位。在最后的技能挑战中，我被要求使用微软 Excel 电子表格软件从内部数据集中提取洞察。为了给对方留下深刻印象，我从我的数据分析工具箱中拉出了我能想到的所有技巧：列排序、文本操作、交叉表，当然还有标志性的 `VLOOKUP` 函数。（好吧，也许“标志性的”有点夸张。）

虽然听起来可能有些奇怪，但当时我并没有意识到除了 Excel 之外还有其他数据分析工具。Excel 几乎无处不在：我的父母用它，我的老师用它，我的同事也用它。它感觉像是一个既定的标准。所以当我收到一份工作邀请时，我立刻购买了价值约 100 美元的 Excel 书籍并开始学习。是时候成为一名电子表格专家了！

我带着打印出来的 50 个最常用 Excel 函数的清单开始了我的第一天工作。在我几乎完成登录工作电脑后不久，我的经理把我拉进会议室，告诉我优先级已经改变。团队的数据集已经膨胀到 Excel 无法支持的大小。我的队友们也在寻找自动化他们日常和每周报告中重复步骤的方法。幸运的是，我的经理已经找到了解决这两个问题的方案。他问我是否听说过 pandas。

“毛茸茸的动物？”我困惑地问道。

“不，”他说。“Python 数据分析库。”

在做好所有准备之后，是时候从头开始学习一项新技术了。我有点紧张；我以前从未写过一行代码。我不是一个 Excel 的人吗？我能够做到这一点吗？只有一个方法可以找到答案。我开始深入研究 pandas 的官方文档，YouTube 视频教程，书籍，研讨会，Stack Overflow 问题和我能接触到的任何数据集。我很高兴地发现，开始使用 pandas 是如此简单和愉快。代码感觉直观且直接。这个库运行速度快。功能开发得很好，而且非常全面。有了 pandas，我可以用很少的代码完成大量的数据处理。

我的故事在 Python 社区中很常见。过去十年中，这种语言天文数字的增长通常归因于新开发者能够轻松上手。我坚信，如果你处于与我类似的位置，你也能同样学好 pandas。如果你想要将数据分析技能扩展到 Excel 表格之外，这本书就是你的邀请函。

当我对 pandas 感到舒适时，我继续探索 Python，然后是其他编程语言。在许多方面，pandas 引领了我向全职软件工程师的转变。我非常感激这个强大的库，我很高兴将知识的火炬传递给你。我希望你能发现代码为你带来的魔法。

## 致谢

要让《Pandas in Action》完成需要付出很多努力，我想对在它的两年写作过程中支持我的人表示最深切的感激。

首先，我要向我的美好女友 Meredith 表示衷心的感谢。从第一句话开始，她就坚定不移地支持我。她是一个充满活力、幽默和善良的灵魂，总是在我遇到困难时支持我。这本书因为有她而更加出色。谢谢你，Merbear。

感谢我的父母 Irina 和 Dmitriy，他们为我提供了一个温馨的家，我总能在这里找到慰藉。

感谢我的双胞胎姐妹 Mary 和 Alexandra。她们在她们这个年龄非常聪明、好奇和勤奋，我为她们感到无比自豪。祝你们在大学好运！

感谢我们的金毛犬沃森。他并不是一个真正的 Python 专家，但他的风趣和友好弥补了这一点。

非常感谢我的编辑 Sarah Miller，和她一起工作是一种绝对的享受。我非常感激她在整个过程中的耐心和洞察力。她是真正的船长，她让一切顺利航行。

如果没有 Indeed 提供的机会，我就不会成为一名软件工程师。我想向我的前经理 Srdjan Bodruzic 表达衷心的感谢，感谢他的慷慨和指导（以及雇佣我！）。感谢我的 CX 团队成员——Tommy Winschel、Danny Moncada、JP Schultz 和 Travis Wright——他们的智慧和幽默。感谢在我任职期间伸出援手的其他 Indeed 员工：Matthew Morin、Chris Hatton、Chip Borsi、Nicole Saglimbene、Danielle Scoli、Blairr Swayne 和 George Improglou。感谢我在 Sophie 的古巴菜餐厅共进晚餐的每一个人！

我是以软件工程师的身份在 Stride Consulting 开始写这本书的。我想感谢许多 Stride 员工在整个过程中的支持：David “The Dominator” DiPanfilo、Min Kwak、Ben Blair、Kirsten Nordine、Michael “Bobby” Nunez、Jay Lee、James Yoo、Ray Veliz、Nathan Riemer、Julia Berchem、Dan Plain、Nick Char、Grant Ziolkowski、Melissa Wahnish、Dave Anderson、Chris Aporta、Michael Carlson、John Galioto、Sean Marzug-McCarthy、Travis Vander Hoop、Steve Solomon 和 Jan Mlčoch。

感谢我作为软件工程师和顾问有机会与之共事的友好面孔：Francis Hwang、Inhak Kim、Liana Lim、Matt Bambach、Brenton Morris、Ian McNally、Josh Philips、Artem Kochnev、Andrew Kang、Andrew Fader、Karl Smith、Bradley Whitwell、Brad Popiolek、Eddie Wharton、Jen Kwok 以及我最喜欢的咖啡团队：Adam McAmis 和 Andy Fritz。

感谢以下这些人为我的生活增添了价值：Nick Bianco、Cam Stier、Keith David、Michael Cheung、Thomas Philippeau、Nicole DiAndrea 和 James Rokeach。

感谢我最喜欢的乐队 New Found Glory，他们为许多写作会议提供了背景音乐。流行朋克并未死去！

感谢帮助该项目顺利完成并协助营销工作的 Manning 团队：Jennifer Houle、Aleksandar Dragosavljević、Radmila Ercegovac、Candace Gillhoolley、Stjepan Jureković和 Lucas Weber。还要感谢负责内容的 Manning 团队：Sarah Miller，我的发展编辑；Deirdre Hiam，我的生产编辑；Keir Simpson，我的校对编辑；以及 Jason Everett，我的校对员。

感谢帮助我消除技术问题的技术审稿人：Al Pezewski、Alberto Ciarlanti、Ben McNamara、Björn Neuhaus、Christopher Kottmyer、Dan Sheikh、Dragos Manailoiu、Erico Lendzian、Jeff Smith、Jérôme Bâton、Joaquin Beltran、Jonathan Sharley、Jose Apablaza、Ken W. Alger、Martin Czygan、Mathijs Affourtit、Matthias Busch、Mike Cuddy、Monica E. Guimaraes、Ninoslav Cerkez、Rick Prins、Syed Hasany、Viton Vitanis 和 Vybhavreddy Kammireddy Changalreddy。感谢你们的努力，使我成为了一名更好的作家和教育者。

最后，感谢过去六年我居住的城市霍布肯。我在其公共图书馆、当地咖啡馆和珍珠奶茶店写下了这本书的许多部分。我在这个城镇取得了许多人生上的进步，它永远刻在了我的历史中。谢谢，霍布肯！

## 关于本书

### 适合阅读本书的人群

《Pandas 实战》是 pandas 库数据分析的全面介绍。Pandas 使您能够轻松执行多种数据操作：排序、连接、转换、清理、去重、聚合等。本书逐步介绍主题，从其较小的构建块开始，逐步过渡到较大的数据结构。

《Pandas 实战》是为那些对电子表格软件（如 Microsoft Excel、Google Sheets 和 Apple Numbers）有中级经验的数据分析师以及/或对替代数据分析工具（如 R 和 SAS）有经验的人编写的。它也适合那些对数据分析感兴趣的 Python 开发者。

### 本书组织结构：路线图

《Pandas 实战》由两大部分组成，共 14 章。

第一部分“核心 pandas”以递增的方式介绍了 pandas 库的基本机制：

+   第一章使用 pandas 分析样本数据集，以展示库的功能概述。

+   第二章介绍了`Series`对象，这是 pandas 的核心数据结构，用于存储有序数据集合。

+   第三章深入探讨了`Series`对象。我们探讨了各种`Series`操作，包括排序值、删除重复项、提取最小值和最大值等。

+   第四章介绍了`DataFrame`，这是一个二维数据表。我们将前几章的概念应用于新的数据结构，并介绍了额外的操作。

+   第五章展示了如何使用各种逻辑条件（如相等、不等、比较、包含、排除等）从`DataFrame`中筛选行子集。

第二部分，“应用 pandas”，专注于更高级的 pandas 功能和它们在现实世界数据集中解决的问题：

+   第六章教你如何在 pandas 中处理不完美的文本数据。我们讨论了如何解决诸如删除空白字符、修复字符大小写以及从一个单独的列中提取多个值等问题。

+   第七章讨论了`MultiIndex`，它允许我们将多个列值组合成一个数据行的唯一标识符。

+   第八章描述了如何在交叉表中聚合我们的数据，将标题从行轴移到列轴，并将我们的数据从宽格式转换为窄格式。

+   第九章探讨了如何将行分组到桶中，并通过`GroupBy`对象聚合结果集合。

+   第十章通过使用各种连接将多个数据集合并成一个。

+   第十一章演示了如何在 pandas 中处理日期和时间。它涵盖了诸如排序日期、计算持续时间以及确定日期是否位于月份或季度的开始等主题。

+   第十二章展示了如何将额外的文件类型导入到 pandas 中，包括 Excel 和 JSON。我们还学习了如何从 pandas 导出数据。

+   第十三章专注于配置库的设置。我们深入探讨了如何修改显示的行数、改变浮点数的精度、四舍五入低于阈值的值等等。

+   第十四章探讨了使用 matplotlib 库进行数据可视化。我们看到了如何使用 pandas 数据创建折线图、条形图、饼图等等。

每一章都是基于前一章构建的。对于那些从头开始学习 pandas 的人来说，我建议按线性顺序阅读章节。同时，为了确保本书作为参考指南的有用性，我已将每一章编写为具有自己数据集的独立教程。我们在每一章的开始从零开始编写代码，这样你可以从任何你喜欢的章节开始。

大多数章节都以一个编码挑战结束，这个挑战允许你练习其概念。我强烈建议尝试这些练习。

Pandas 建立在 Python 编程语言之上，建议在开始之前对语言机制有基本了解。对于那些 Python 经验有限的人来说，附录 B 提供了对该语言的详细介绍。

### 关于代码

本书包含许多源代码示例，这些示例以固定宽度字体`like this`格式化，以将其与普通文本区分开来。

本书示例的源代码可在以下 GitHub 仓库中找到：[`github.com/paskhaver/pandas-in-action`](https://github.com/paskhaver/pandas-in-action)。对于 Git 和 GitHub 新手，请在仓库页面上寻找下载 ZIP 按钮。对于熟悉 Git 和 GitHub 的用户，欢迎从命令行克隆仓库。

仓库还包括文本的完整数据集。当我学习 pandas 时，我最大的挫折之一是教程总是喜欢依赖于随机生成数据。没有一致性，没有上下文，没有故事，没有乐趣。在这本书中，我们将使用许多真实世界的数据集，涵盖从篮球运动员的薪水到宝可梦类型再到餐馆卫生检查的各个方面。数据无处不在，pandas 是今天可用的最佳工具之一，可以帮助我们理解这些数据。我希望您喜欢数据集的轻松焦点。

### liveBook 讨论论坛

购买《Pandas in Action》包括免费访问由曼宁出版社运行的私人网络论坛，您可以在论坛中就本书发表评论，提出技术问题，并从作者和其他用户那里获得帮助。要访问论坛，请访问[`livebook.manning.com/#!/book/pandas-in-action/discussion`](https://livebook.manning.com/#!/book/pandas-in-action/discussion)。您还可以在[`live book.manning.com/#!/discussion`](https://livebook.manning.com/#!/discussion)了解更多关于曼宁论坛和行为的规则。

曼宁出版社对读者的承诺是提供一个场所，让读者之间以及读者与作者之间可以进行有意义的对话。这并不是对作者参与特定数量活动的承诺，作者对论坛的贡献仍然是自愿的（且未付费）。我们建议您尝试向作者提出一些挑战性的问题，以免他们的兴趣转移！只要这本书有售，论坛和以前讨论的存档将可通过出版社的网站访问。

### 其他在线资源

+   官方的 pandas 文档可在[`pandas.pydata.org /docs`](https://pandas.pydata.org/docs)找到。

+   在我的业余时间，我在 Udemy 上创建技术视频课程。您可以在[`www.udemy.com/user/borispaskhaver`](https://www.udemy.com/user/borispaskhaver)找到这些课程；它们包括一个 20 小时的 pandas 课程和一个 60 小时的 Python 课程。

+   您可以通过 Twitter ([`twitter.com/borispaskhaver`](https://twitter.com/borispaskhaver))或 LinkedIn ([`www.linkedin.com/in/boris-paskhaver`](https://www.linkedin.com/in/boris-paskhaver))联系我。

## 关于作者

博里斯·帕斯哈弗（Boris Paskhaver）是一位位于纽约市的全栈软件工程师、顾问和在线教育者。他在 e-learning 平台 Udemy 上有六门课程，超过 140 小时的视频，300,000 名学生，20,000 条评论，以及每月消耗 1,000 万分钟的课程内容。在成为软件工程师之前，博里斯曾担任数据分析师和系统管理员。他于 2013 年毕业于纽约大学，主修商业经济学和市场营销。

## 关于封面插图

《熊猫行动》封面上的插图被标注为“加莱夫人”，或称“加莱女士”。这幅插图取自雅克·格拉塞·德·圣索沃尔（1757–1810）的作品集，名为《不同国家的服饰》，于 1797 年在法国出版。每一幅插图都是手工精心绘制和着色的。格拉塞·德·圣索沃尔收藏中的丰富多样性生动地提醒我们，200 年前世界的城镇和地区在文化上是如何截然不同的。他们彼此孤立，说着不同的方言和语言。在街道或乡村，仅凭他们的服饰就能轻易识别他们居住的地方以及他们的职业或社会地位。

自那以后，我们的着装方式已经改变，而当时区域间的多样性已经消失。现在很难区分不同大陆的居民，更不用说不同的城镇、地区或国家了。也许我们用文化多样性换取了更加丰富多彩的个人生活——当然，是更加丰富多彩和快节奏的技术生活。

在难以区分一本计算机书和另一本计算机书的今天，曼宁通过基于两百年前区域生活的深刻多样性设计的书封面，庆祝了计算机行业的创新精神和主动性，这些多样性被格拉塞·德·圣索沃尔的图画重新赋予了生命。
