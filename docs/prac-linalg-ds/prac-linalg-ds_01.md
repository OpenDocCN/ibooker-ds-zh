# 第一章：引言

# 什么是线性代数以及为什么要学它？

线性代数在数学中有着有趣的历史，起源可以追溯到西方 17 世纪，而在中国要早得多。矩阵——线性代数核心的数字表格——被用来提供一种紧凑的符号来存储像几何坐标这样的数字集合（这是笛卡尔对矩阵的最初运用），以及方程组（由高斯首创）。在 20 世纪，矩阵和向量被用于包括微积分、微分方程、物理学和经济学在内的多变量数学中。

但直到最近，大多数人都不需要关心矩阵。这里的关键是：计算机在处理矩阵方面非常高效。因此，现代计算引发了现代线性代数的兴起。现代线性代数是计算性的，而传统线性代数则是抽象的。现代线性代数最好通过代码和应用于图形、统计学、数据科学、人工智能和数值模拟来学习，而传统线性代数则是通过证明和思考无限维向量空间来学习。现代线性代数为几乎每个计算机实现的算法提供了结构性支柱，而传统线性代数则常常是高等数学大学生的智力食粮。

欢迎来到现代线性代数。

你应该学习线性代数吗？这取决于你是否想理解算法和过程，还是仅仅应用别人开发的方法。我并不是贬低后者——使用你不理解的工具没有本质上的错误（我正在一台我可以使用但不能从头开始构建的笔记本电脑上写这篇文章）。但考虑到你正在阅读这本标题为 O’Reilly 图书系列的书，我猜你要么（1）想要知道算法如何工作，要么（2）想要开发或调整计算方法。所以是的，你应该学习线性代数，而且你应该学习它的现代版本。

# 关于本书

本书的目的是教会你现代线性代数。但这不是记忆一些关键方程和苦苦钻研抽象证明的问题；目的是教会你如何*思考*矩阵、向量以及作用在它们上面的操作。你将会对为何线性代数如此重要有几何直觉。你将会理解如何在 Python 代码中实现线性代数概念，重点放在机器学习和数据科学的应用上。

许多传统的线性代数教科书为了泛化起见避免了数值示例，期望你自行推导困难的证明，并教授大量与计算机应用或实现无关的概念。我并不是在批评——抽象的线性代数是美丽而优雅的。但如果你的目标是将线性代数（以及数学更广泛地说）作为理解数据、统计、深度学习、图像处理等工具，那么传统的线性代数教科书可能会让你感到时间的浪费，让你感到困惑，并且担心你在技术领域的潜力。

这本书是为自学者而写的。也许你有数学、工程或物理学学位，但需要学习如何在代码中实现线性代数。或者你在大学没有学习数学，现在意识到线性代数对你的学习或工作的重要性。无论哪种方式，这本书都是一个独立的资源；它不仅仅是一门基于讲座的课程的补充（尽管它可以用于这个目的）。

如果你在阅读前面三段时点头表示赞同，那么这本书绝对适合你。

如果你想深入研究线性代数，包括更多的证明和探索，那么有几本优秀的书籍可以考虑，包括我的《线性代数：理论、直觉、代码》（Sincxpress BV）。¹

# 先决条件

我试图为那些背景知识较少但热情学习者写这本书。话虽如此，没有东西是真正从零开始学会的。

## 数学

你需要对高中数学感到自在。只要基本的代数和几何；没什么高深的东西。

这本书不需要任何微积分（尽管微积分对于线性代数经常用于的应用，如深度学习和优化，非常重要）。

但更重要的是，你需要对数学有一定的思考能力，能看方程和图形，并接受与学习数学相关的知识挑战。

## 态度

线性代数是数学的一个分支，因此这是一本数学书。学习数学，尤其是作为成年人，需要一些耐心、奉献精神和积极的态度。泡一杯咖啡，深呼吸，把手机放在另一个房间，然后开始深入研究。

在你脑海中会有一个声音告诉你，你太老了或太笨了，学习高级数学。有时这个声音会更响，有时更轻，但它总是在那里。而且不仅仅是你——每个人都有这样的声音。你不能抑制或摧毁那个声音；甚至别试了。接受一点不安全感和自我怀疑是人类的一部分。每次那个声音响起，都是你向它证明它错了的挑战。

## 编程

本书侧重于代码中的线性代数应用。我选择用 Python 写这本书，因为 Python 目前是数据科学、机器学习及相关领域中使用最广泛的语言。如果你更喜欢 MATLAB、R、C 或 Julia 等其他语言，那么我希望你能轻松地将 Python 代码翻译过去。

我试图尽可能简化 Python 代码，同时保持其适用性。第十六章 提供了 Python 编程的基本介绍。你是否需要阅读这一章取决于你的 Python 技能水平：

中级/高级（>1 年编码经验）

完全跳过第十六章，或者可能略读一下，了解一下在本书的其余部分中会出现的代码类型。

一些知识（<1 年经验）

如果有新的内容或需要复习的内容，请务必仔细阅读该章节。但你应该能够迅速通过它。

完全初学者

详细阅读本章节。请理解，这本书不是完整的 Python 教程，因此如果你在内容章节中遇到代码困难，可能需要放下这本书，通过专门的 Python 课程或书籍进行学习，然后再回到这本书。

# 数学证明与从编码中获得的直觉

学习数学的目的是理解数学。如何理解数学呢？我们来数数：

严格的证明

在数学中，证明是一系列陈述，表明一组假设导致了一个逻辑结论。证明在纯数学中无疑是非常重要的。

可视化和示例

清晰的解释、图表和数值示例帮助你理解线性代数中的概念和操作。大多数示例都在 2D 或 3D 中进行简单可视化，但这些原则也适用于更高维度。

这两者的区别在于，正式的数学证明提供了严谨性，但很少提供直觉，而可视化和示例通过实际操作提供持久的直觉，但可能根据不同的特定示例存在风险不准确性。

包括重要声明的证明，但我更注重通过解释、可视化和代码示例来建立直觉。

这使我想到了从编码中获得数学直觉（我有时称之为“软证明”）。这里的想法是：你假设 Python（以及诸如 NumPy 和 SciPy 这样的库）正确地实现了低级别的数值计算，而你通过探索许多数值示例来专注于原则。

一个快速的例子：我们将“软证明”乘法的交换性原理，即 <math alttext="a times b equals b times a"><mrow><mi>a</mi> <mo>×</mo> <mi>b</mi> <mo>=</mo> <mi>b</mi> <mo>×</mo> <mi>a</mi></mrow></math>：

```
a = np.random.randn()
b = np.random.randn()
a*b - b*a
```

这段代码生成两个随机数，并测试交换乘法顺序对结果无影响的假设。如果可交换原则成立，第三行将输出`0.0`。如果你多次运行此代码并始终得到`0.0`，那么你通过在许多不同的数值示例中看到相同的结果来获得了可交换性的直觉。

明确一点：从代码中获得的直觉不能替代严格的数学证明。关键是，“软证明”允许你理解数学概念，而不必担心抽象数学语法和论证的细节。这对缺乏高级数学背景的程序员特别有利。

底线是*你可以通过一点编程学到很多数学*。

# 书中和在线下载的代码

你可以阅读本书而不看代码或解决代码练习。这没问题，你肯定会学到一些东西。但如果你的知识是肤浅和短暂的，不要感到失望。如果你真的想*理解*线性代数，你需要解决问题。这就是为什么本书为每个数学概念都配有代码演示和练习的原因。

重要的代码直接打印在书中。我希望你阅读文本和方程式，查看图形，并同时*看到代码*。这将帮助你将概念和方程式与代码联系起来。

但在书中打印代码可能占用大量空间，而在计算机上手动复制代码很烦琐。因此，书中仅打印关键代码行；在线代码包含额外的代码、注释、图形装饰等。在线代码还包含编程练习的解决方案（所有练习，不仅仅是奇数问题！）。你应该在阅读本书的同时下载代码并进行学习。

所有代码都可以从 GitHub 网站[*https://github.com/mikexco⁠hen​/LinAlg4DataScienc*](https://github.com/mikexcohen/LinAlg4DataScience)获取。你可以克隆这个仓库或简单地下载整个仓库作为 ZIP 文件（无需注册、登录或付费下载代码）。

我使用 Google 的 Colab 环境在 Jupyter 笔记本中编写代码。我选择使用 Jupyter 因为它是一个友好且易于使用的环境。话虽如此，我鼓励你使用任何你喜欢的 Python 集成开发环境。在线代码也以原始的*.py*文件形式提供以方便使用。

# 代码练习

数学不是一项旁观者运动。大多数数学书籍都有无数的纸和笔问题要解决（老实说：没有人会解决所有这些问题）。但这本书完全是关于*应用*线性代数的，没有人会在纸上应用线性代数！相反，你要在代码中应用线性代数。因此，不再有手工解决的问题和让读者练习的乏味证明（数学教科书作者喜欢写的），这本书有大量的代码练习。

代码练习的难度各不相同。如果你是 Python 和线性代数的新手，你可能会发现一些练习非常具有挑战性。如果你遇到困难，这里有一个建议：快速浏览我的解决方案以获取灵感，然后将其放在一边，以便不再看到我的代码，然后继续努力完成你自己的代码。

在比较你的解决方案和我的时候，请记住在 Python 中解决问题有多种方式。找到正确答案很重要；你所采取的步骤通常取决于个人的编码风格。

# 如何使用本书（供教师和自学者使用）

这本书有三种使用环境：

自学者

我试图使这本书适合那些希望在非正式课堂环境外自学线性代数的读者。虽然当然还有无数其他书籍、网站、YouTube 视频和在线课程可能对学生有帮助。

数据科学课程的主要教材

这本书可以作为数据科学、机器学习、人工智能及相关主题的数学基础课程的主要教材。除此介绍和 Python 附录外，共有 14 个内容章节，学生每周可以预计学习一到两章。因为学生可以访问所有练习的解答，教师可以选择用额外的问题集来补充书中的练习。

数学课程中重点是线性代数的次要教材

这本书也可以作为数学课程的补充教材，重点是证明。在这种情况下，讲座将专注于理论和严格的证明，而这本书可以用于将概念转化为代码，关注数据科学和机器学习中的应用。正如我前面所写的，教师可以选择提供补充练习，因为所有书中练习的解答都可以在线找到。

¹ 抱歉自吹自擂；我保证这本书里我只会这么一次地对你做这样的纵容。