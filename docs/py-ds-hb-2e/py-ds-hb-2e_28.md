# 第四部分：Matplotlib 可视化

现在我们将深入研究 Python 中用于可视化的 Matplotlib 包。Matplotlib 是一个建立在 NumPy 数组上的跨平台数据可视化库，旨在与更广泛的 SciPy 栈配合使用。它由 John Hunter 在 2002 年构思，最初作为 IPython 的补丁，用于通过 IPython 命令行从`gnuplot`实现交互式 MATLAB 风格的绘图。当时，IPython 的创始人 Fernando Perez 正在忙于完成他的博士论文，没有时间几个月内审查该补丁。John 将此视为自己行动的信号，于是 Matplotlib 包诞生了，版本 0.1 于 2003 年发布。当它被采纳为太空望远镜科学研究所（背后是哈勃望远镜的人们）首选的绘图包，并得到财政支持以及大幅扩展其功能时，Matplotlib 得到了早期的推广。

Matplotlib 最重要的特点之一是其与多种操作系统和图形后端的良好兼容性。Matplotlib 支持数十种后端和输出类型，这意味着无论您使用哪种操作系统或希望使用哪种输出格式，它都能正常工作。这种跨平台、面面俱到的方法一直是 Matplotlib 的一大优势。它导致了大量用户的使用，进而促使了活跃的开发者基础以及 Matplotlib 在科学 Python 社区中强大的工具和普及率。

近年来，然而，Matplotlib 的界面和风格开始显得有些过时。像 R 语言中的`ggplot`和`ggvis`以及基于 D3js 和 HTML5 canvas 的 Web 可视化工具包，常使 Matplotlib 感觉笨重和老旧。尽管如此，我认为我们不能忽视 Matplotlib 作为一个经过良好测试的跨平台图形引擎的优势。最近的 Matplotlib 版本使得设置新的全局绘图样式相对容易（参见 第三十四章），人们一直在开发新的包，利用其强大的内部机制通过更清晰、更现代的 API 驱动 Matplotlib，例如 Seaborn（在 第三十六章 讨论），[`ggpy`](http://yhat.github.io/ggpy)，[HoloViews](http://holoviews.org)，甚至 Pandas 本身可以作为 Matplotlib API 的封装器使用。即使有了这些封装器，深入了解 Matplotlib 的语法来调整最终的绘图输出仍然经常很有用。因此，我认为即使新工具意味着社区逐渐不再直接使用 Matplotlib API，Matplotlib 本身仍将保持数据可视化堆栈中不可或缺的一部分。
