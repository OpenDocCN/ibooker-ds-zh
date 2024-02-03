# 1 初步

> 原文：[https://wesmckinney.com/book/preliminaries](https://wesmckinney.com/book/preliminaries)

*这本《Python数据分析第三版》的开放获取网络版本现在作为[印刷版和数字版](https://amzn.to/3DyLaJc)的伴侣可用。如果您发现任何勘误，请[在此处报告](https://oreilly.com/catalog/0636920519829/errata)。请注意，由Quarto制作的本站的某些方面与O'Reilly的印刷版和电子书版本的格式不同。

如果您发现这本书的在线版本有用，请考虑[订购纸质版](https://amzn.to/3DyLaJc)或[无DRM的电子书](https://www.ebooks.com/en-us/book/210644288/python-for-data-analysis/wes-mckinney/?affId=WES398681F)以支持作者。本网站的内容不得复制或复制。代码示例采用MIT许可证，可在GitHub或Gitee上找到。*## 1.1 这本书是关于什么的？

这本书关注的是在Python中操纵、处理、清理和处理数据的基本原理。我的目标是为Python编程语言及其面向数据的库生态系统和工具提供指南，使您能够成为一名有效的数据分析师。虽然书名中有“数据分析”一词，但重点特别放在Python编程、库和工具上，而不是数据分析方法论。这是您进行数据分析所需的Python编程。

在我2012年首次出版这本书之后不久，人们开始使用“数据科学”这个术语作为从简单的描述性统计到更高级的统计分析和机器学习等各种内容的总称。自那时以来，用于进行数据分析（或数据科学）的Python开源生态系统也显著扩展。现在有许多其他专门关注这些更高级方法的书籍。我希望这本书能够作为足够的准备，使您能够转向更具领域特定性的资源。

*注* *有些人可能将本书的大部分内容描述为“数据操纵”，而不是“数据分析”。我们还使用*整理*或*整理*这些术语来指代数据操纵。*### 什么样的数据？

当我说“数据”时，我确切指的是什么？主要关注的是*结构化数据*，这是一个故意模糊的术语，包括许多不同形式的常见数据，例如：

+   表格或类似电子表格的数据，其中每列可能是不同类型（字符串、数字、日期或其他）。这包括通常存储在关系数据库或制表符或逗号分隔文本文件中的各种数据。

+   多维数组（矩阵）。

+   由关键列相互关联的多个数据表（对SQL用户来说可能是主键或外键）。

+   均匀或不均匀间隔的时间序列。

这绝不是一个完整的列表。即使可能并不总是明显，大部分数据集都可以转换为更适合分析和建模的结构化形式。如果不行，可能可以从数据集中提取特征到结构化形式。例如，一组新闻文章可以处理成一个词频表，然后用于执行情感分析。

像Microsoft Excel这样的电子表格程序的大多数用户，可能是世界上最广泛使用的数据分析工具，对这些数据类型并不陌生。*## 1.2 为什么选择Python进行数据分析？

对许多人来说，Python编程语言具有很强的吸引力。自1991年首次亮相以来，Python已成为最受欢迎的解释性编程语言之一，与Perl、Ruby等一起。自2005年左右以来，Python和Ruby特别受欢迎，用于构建网站，使用它们众多的Web框架，如Rails（Ruby）和Django（Python）。这些语言通常被称为“脚本”语言，因为它们可以用于快速编写小程序或脚本来自动化其他任务。我不喜欢“脚本语言”这个术语，因为它带有一种暗示，即它们不能用于构建严肃的软件。出于各种历史和文化原因，在解释性语言中，Python已经发展成一个庞大而活跃的科学计算和数据分析社区。在过去的20年里，Python已经从一个前沿或“自担风险”的科学计算语言发展成为学术界和工业界数据科学、机器学习和通用软件开发中最重要的语言之一。

对于数据分析、交互式计算和数据可视化，Python不可避免地会与其他广泛使用的开源和商业编程语言和工具进行比较，如R、MATLAB、SAS、Stata等。近年来，Python改进的开源库（如pandas和scikit-learn）使其成为数据分析任务的热门选择。结合Python在通用软件工程方面的整体实力，它是构建数据应用程序的主要语言的绝佳选择。

### Python作为胶水

Python在科学计算中的成功部分在于轻松集成C、C++和FORTRAN代码。大多数现代计算环境共享一组类似的传统FORTRAN和C库，用于进行线性代数、优化、积分、快速傅里叶变换等算法。许多公司和国家实验室使用Python将几十年的传统软件粘合在一起的故事也是如此。

许多程序由小部分代码组成，其中大部分时间都花在其中，大量“胶水代码”很少运行。在许多情况下，胶水代码的执行时间微不足道；最有价值的努力是在优化计算瓶颈上，有时通过将代码移动到像C这样的低级语言来实现。

### 解决“双语言”问题

在许多组织中，通常使用更专门的计算语言如SAS或R进行研究、原型设计和测试新想法，然后将这些想法移植为更大的生产系统的一部分，比如Java、C#或C++。人们越来越发现Python不仅适合用于研究和原型设计，也适合用于构建生产系统。当一个开发环境足够时，为什么要维护两个呢？我相信越来越多的公司会选择这条道路，因为让研究人员和软件工程师使用相同的编程工具集通常会带来重大的组织效益。

在过去的十年里，一些解决“双语言”问题的新方法出现了，比如Julia编程语言。在许多情况下，充分利用Python将需要使用低级语言如C或C++编程，并创建Python绑定到该代码。也就是说，像Numba这样的“即时”（JIT）编译器技术提供了一种在Python编程环境中实现出色性能的方法，而无需离开Python编程环境。

### 为什么不用Python？

虽然Python是构建许多种分析应用程序和通用系统的优秀环境，但也有一些用途不太适合Python。

由于Python是一种解释性编程语言，通常大多数Python代码运行速度会比像Java或C++这样的编译语言编写的代码慢得多。由于*程序员时间*通常比*CPU时间*更有价值，许多人愿意做出这种权衡。然而，在具有非常低延迟或对资源利用要求苛刻的应用程序中（例如高频交易系统），花费时间以低级语言（但也低生产力）如C++编程，以实现可能的最大性能，可能是值得的。

Python可能是一个具有挑战性的语言，用于构建高度并发、多线程的应用程序，特别是具有许多CPU绑定线程的应用程序。造成这种情况的原因是它具有所谓的*全局解释器锁*（GIL），这是一种机制，防止解释器一次执行多个Python指令。GIL存在的技术原因超出了本书的范围。虽然在许多大数据处理应用中，可能需要一组计算机集群来在合理的时间内处理数据集，但仍然存在一些情况，其中单进程、多线程系统是可取的。

这并不是说Python不能执行真正的多线程、并行代码。使用本地多线程（在C或C++中）的Python C扩展可以在不受GIL影响的情况下并行运行代码，只要它们不需要经常与Python对象交互。

## 1.3必要的Python库

对于那些对Python数据生态系统和本书中使用的库不太熟悉的人，我将简要介绍其中一些。

### NumPy

[NumPy](https://numpy.org)，简称Numerical Python，长期以来一直是Python中数值计算的基石。它提供了大多数涉及Python中数值数据的科学应用所需的数据结构、算法和库粘合剂。NumPy包含，除其他内容外：

+   快速高效的多维数组对象*ndarray*

+   执行数组元素计算或数组之间的数学运算的函数

+   用于读取和写入基于数组的数据集到磁盘的工具

+   线性代数运算、傅里叶变换和随机数生成

+   成熟的C API，用于使Python扩展和本地C或C++代码能够访问NumPy的数据结构和计算功能

除了NumPy为Python增加的快速数组处理功能外，它在数据分析中的主要用途之一是作为数据容器，在算法和库之间传递数据。对于数值数据，NumPy数组比其他内置Python数据结构更有效地存储和操作数据。此外，使用低级语言（如C或FORTRAN）编写的库可以在NumPy数组中存储的数据上操作，而无需将数据复制到其他内存表示中。因此，许多Python的数值计算工具要么将NumPy数组作为主要数据结构，要么针对与NumPy的互操作性。

### pandas

[pandas](https://pandas.pydata.org)提供了高级数据结构和函数，旨在使处理结构化或表格数据变得直观和灵活。自2010年出现以来，它已经帮助Python成为一个强大和高效的数据分析环境。本书中将使用的pandas中的主要对象是DataFrame，这是一个表格化的、以列为导向的数据结构，具有行和列标签，以及Series，这是一个一维带标签的数组对象。

pandas将NumPy的数组计算思想与电子表格和关系数据库（如SQL）中发现的数据操作能力相结合。它提供了方便的索引功能，使您能够重新塑造、切片、执行聚合操作和选择数据子集。由于数据操作、准备和清理在数据分析中是如此重要，pandas是本书的主要关注点之一。

作为背景，我在2008年初在AQR Capital Management期间开始构建pandas，这是一家量化投资管理公司。当时，我有一套明确的要求，任何单一工具都无法很好地满足：

+   具有带有标签轴的数据结构，支持自动或显式数据对齐——这可以防止由于数据不对齐和来自不同来源的不同索引数据而导致的常见错误

+   集成的时间序列功能

+   相同的数据结构处理时间序列数据和非时间序列数据

+   保留元数据的算术操作和减少

+   灵活处理缺失数据

+   在流行数据库（例如基于SQL的数据库）中找到的合并和其他关系操作

我希望能够在一个地方完成所有这些事情，最好是在一种适合通用软件开发的语言中。Python是这方面的一个很好的候选语言，但当时并不存在一个集成了这些功能的数据结构和工具集。由于最初构建是为了解决金融和业务分析问题，pandas具有特别深入的时间序列功能和适用于处理由业务流程生成的时间索引数据的工具。

我在2011年和2012年的大部分时间里与我以前的AQR同事Adam Klein和Chang She一起扩展了pandas的功能。2013年，我停止了日常项目开发的参与，pandas自那时起已成为一个完全由社区拥有和维护的项目，全球范围内有超过两千名独特贡献者。

对于使用R语言进行统计计算的用户，DataFrame这个名字将是熟悉的，因为该对象是根据类似的R `data.frame`对象命名的。与Python不同，数据框内置于R编程语言及其标准库中。因此，pandas中许多功能通常要么是R核心实现的一部分，要么是由附加包提供的。

pandas这个名字本身来源于*panel data*，这是一个描述多维结构化数据集的计量经济学术语，也是对*Python数据分析*这个短语的一种变换。

### matplotlib

[matplotlib](https://matplotlib.org)是用于生成图表和其他二维数据可视化的最流行的Python库。最初由John D. Hunter创建，现在由一个庞大的开发团队维护。它专为创建适合出版的图表而设计。虽然Python程序员可以使用其他可视化库，但matplotlib仍然被广泛使用，并且与生态系统的其他部分相当好地集成。我认为它是默认可视化工具的一个安全选择。

### IPython和Jupyter

[IPython项目](https://ipython.org)始于2001年，是Fernando Pérez的一个副业项目，旨在打造更好的交互式Python解释器。在随后的20年里，它已成为现代Python数据堆栈中最重要的工具之一。虽然它本身不提供任何计算或数据分析工具，但IPython旨在用于交互式计算和软件开发工作。它鼓励*执行-探索*工作流程，而不是许多其他编程语言的典型*编辑-编译-运行*工作流程。它还提供了对操作系统的shell和文件系统的集成访问；这在许多情况下减少了在终端窗口和Python会话之间切换的需求。由于许多数据分析编码涉及探索、试错和迭代，IPython可以帮助您更快地完成工作。

2014年，Fernando和IPython团队宣布了[Jupyter项目](https://jupyter.org)，这是一个更广泛的倡议，旨在设计与语言无关的交互式计算工具。IPython网络笔记本变成了Jupyter笔记本，现在支持超过40种编程语言。IPython系统现在可以作为使用Python与Jupyter的*内核*（编程语言模式）。

IPython本身已成为更广泛的Jupyter开源项目的组成部分，为交互式和探索性计算提供了一个高效的环境。它最古老和最简单的“模式”是作为一个增强的Python shell，旨在加速Python代码的编写、测试和调试。您还可以通过Jupyter笔记本使用IPython系统。

Jupyter笔记本系统还允许您在Markdown和HTML中编写内容，为您提供了一种创建包含代码和文本的丰富文档的方式。

我个人经常在我的Python工作中使用IPython和Jupyter，无论是运行、调试还是测试代码。

在[GitHub上的附带书籍材料](https://github.com/wesm/pydata-book)中，您将找到包含每章代码示例的Jupyter笔记本。如果您无法访问GitHub，您可以尝试[Gitee上的镜像](https://gitee.com/wesmckinn/pydata-book)。

### SciPy

[SciPy](https://scipy.org)是一个解决科学计算中一些基础问题的包集合。以下是它在各个模块中包含的一些工具：

`scipy.integrate`

数值积分例程和微分方程求解器

`scipy.linalg`

线性代数例程和矩阵分解，扩展到`numpy.linalg`提供的范围之外

`scipy.optimize`

函数优化器（最小化器）和根查找算法

`scipy.signal`

信号处理工具

`scipy.sparse`

稀疏矩阵和稀疏线性系统求解器

`scipy.special`

SPECFUN的包装器，一个实现许多常见数学函数（如`gamma`函数）的FORTRAN库

`scipy.stats`

标准连续和离散概率分布（密度函数、采样器、连续分布函数）、各种统计检验和更多描述性统计

NumPy和SciPy共同构成了许多传统科学计算应用的相当完整和成熟的计算基础。

### scikit-learn

自2007年项目开始以来，[scikit-learn](https://scikit-learn.org)已成为Python程序员的首选通用机器学习工具包。截至撰写本文时，超过两千名不同的个人为该项目贡献了代码。它包括用于以下模型的子模块：

+   分类：SVM、最近邻、随机森林、逻辑回归等

+   回归：Lasso、岭回归等

+   聚类：*k*-means、谱聚类等

+   降维：PCA、特征选择、矩阵分解等

+   模型选择：网格搜索、交叉验证、度量

+   预处理：特征提取、归一化

除了pandas、statsmodels和IPython之外，scikit-learn对于使Python成为一种高效的数据科学编程语言至关重要。虽然我无法在本书中包含对scikit-learn的全面指南，但我将简要介绍一些其模型以及如何将其与本书中提供的其他工具一起使用。

### statsmodels

[statsmodels](https://statsmodels.org)是一个统计分析包，由斯坦福大学统计学教授Jonathan Taylor的工作启发而来，他实现了R编程语言中流行的一些回归分析模型。Skipper Seabold和Josef Perktold于2010年正式创建了新的statsmodels项目，自那时以来，该项目已经发展成为一群积极参与的用户和贡献者。Nathaniel Smith开发了Patsy项目，该项目提供了一个受R公式系统启发的用于statsmodels的公式或模型规范框架。

与scikit-learn相比，statsmodels包含用于经典（主要是频率主义）统计和计量经济学的算法。这包括诸如：

+   回归模型：线性回归、广义线性模型、鲁棒线性模型、线性混合效应模型等

+   方差分析（ANOVA）

+   时间序列分析：AR、ARMA、ARIMA、VAR和其他模型

+   非参数方法：核密度估计、核回归

+   统计模型结果的可视化

statsmodels更专注于统计推断，为参数提供不确定性估计和*p*-值。相比之下，scikit-learn更注重预测。

与scikit-learn一样，我将简要介绍statsmodels以及如何与NumPy和pandas一起使用它。

### 其他包

在2022年，有许多其他Python库可能会在关于数据科学的书中讨论。这包括一些较新的项目，如TensorFlow或PyTorch，这些项目已经成为机器学习或人工智能工作中流行的工具。现在有其他更专注于这些项目的书籍，我建议使用本书来建立通用Python数据处理的基础。然后，您应该准备好转向更高级的资源，这些资源可能假定一定水平的专业知识。

## 1.4 安装和设置

由于每个人都在不同的应用中使用Python，因此设置Python并获取必要的附加包没有单一的解决方案。许多读者可能没有完整的Python开发环境，适合跟随本书，因此我将在每个操作系统上提供详细的设置说明。我将使用Miniconda，这是conda软件包管理器的最小安装，以及[conda-forge](https://conda-forge.org)，这是一个基于conda的社区维护的软件分发。本书始终使用Python 3.10，但如果您是在未来阅读，欢迎安装更新版本的Python。

如果由于某种原因，这些说明在您阅读时已过时，您可以查看[我的书籍网站](https://wesmckinney.com/book)，我将努力保持最新安装说明的更新。

### Windows上的Miniconda

要在Windows上开始，请从[*https://conda.io*](https://conda.io)下载最新Python版本（目前为3.9）的Miniconda安装程序。我建议按照conda网站上提供的Windows安装说明进行安装，这些说明可能在本书出版时和您阅读时之间发生了变化。大多数人会想要64位版本，但如果这在您的Windows机器上无法运行，您可以安装32位版本。

当提示是否仅为自己安装还是为系统上的所有用户安装时，请选择最适合您的选项。仅为自己安装将足以跟随本书。它还会询问您是否要将Miniconda添加到系统PATH环境变量中。如果选择此选项（我通常会这样做），则此Miniconda安装可能会覆盖您已安装的其他Python版本。如果不这样做，那么您将需要使用安装的Window开始菜单快捷方式才能使用此Miniconda。此开始菜单条目可能称为“Anaconda3 (64位)”。

我假设您还没有将Miniconda添加到系统路径中。要验证配置是否正确，请在“开始”菜单下的“Anaconda3 (64位)”中打开“Anaconda Prompt (Miniconda3)”条目。然后尝试通过输入`python`来启动Python解释器。您应该会看到类似以下的消息：

```py
(base) C:\Users\Wes>python
Python 3.9 [MSC v.1916 64 bit (AMD64)] :: Anaconda, Inc. on win32
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

要退出Python shell，请输入`exit()`并按Enter键。

### GNU/Linux

Linux的详细信息会根据您的Linux发行版类型有所不同，但在这里我提供了Debian、Ubuntu、CentOS和Fedora等发行版的详细信息。设置与macOS类似，唯一的区别是Miniconda的安装方式。大多数读者会想要下载默认的64位安装程序文件，这是针对x86架构的（但未来可能会有更多用户使用基于aarch64的Linux机器）。安装程序是一个必须在终端中执行的shell脚本。然后您将会得到一个类似*Miniconda3-latest-Linux-x86_64.sh*的文件。要安装它，请使用`bash`执行此脚本：

```py
$ bash Miniconda3-latest-Linux-x86_64.sh
```

*注意* *一些Linux发行版在其软件包管理器中具有所有所需的Python软件包（在某些情况下是过时版本），可以使用类似apt的工具进行安装。这里描述的设置使用Miniconda，因为它在各种发行版中都很容易重现，并且更简单地升级软件包到最新版本。* *您可以选择将Miniconda文件放在哪里。我建议将文件安装在您的主目录中的默认位置；例如，*/home/$USER/miniconda*（自然包括您的用户名）。

安装程序会询问您是否希望修改您的shell脚本以自动激活Miniconda。我建议这样做（选择“是”）以方便起见。

安装完成后，启动一个新的终端进程并验证您是否已经安装了新的Miniconda：

```py
(base) $ python
Python 3.9 | (main) [GCC 10.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

要退出Python shell，请输入`exit()`并按Enter键或按Ctrl-D。*### macOS上的Miniconda

下载macOS Miniconda安装程序，应该命名为*Miniconda3-latest-MacOSX-arm64.sh*，适用于2020年以后发布的基于Apple Silicon的macOS计算机，或者*Miniconda3-latest-MacOSX-x86_64.sh*，适用于2020年之前发布的基于Intel的Mac。在macOS中打开终端应用程序，并通过使用`bash`执行安装程序（很可能在您的`Downloads`目录中）来安装：

```py
$ bash $HOME/Downloads/Miniconda3-latest-MacOSX-arm64.sh
```

当安装程序运行时，默认情况下会自动在默认shell环境和默认shell配置文件中配置Miniconda。这可能位于*/Users/$USER/.zshrc*。我建议让它这样做；如果您不想让安装程序修改默认的shell环境，您需要查阅Miniconda文档以便继续。

要验证一切是否正常工作，请尝试在系统shell中启动Python（打开终端应用程序以获取命令提示符）：

```py
$ python
Python 3.9 (main) [Clang 12.0.1 ] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

要退出shell，请按Ctrl-D或输入`exit()`并按Enter键。

### 安装必要的软件包

现在我们已经在您的系统上设置了Miniconda，是时候安装本书中将要使用的主要软件包了。第一步是通过在shell中运行以下命令将conda-forge配置为您的默认软件包渠道：

```py
(base) $ conda config --add channels conda-forge
(base) $ conda config --set channel_priority strict
```

现在，我们将使用Python 3.10使用`conda create`命令创建一个新的conda“环境”：

```py
(base) $ conda create -y -n pydata-book python=3.10
```

安装完成后，请使用`conda activate`激活环境：

```py
(base) $ conda activate pydata-book
(pydata-book) $
```

*注意* *每次打开新终端时，都需要使用`conda activate`来激活您的环境。您可以随时通过在终端中运行`conda info`来查看有关活动conda环境的信息。*  *现在，我们将使用`conda install`安装整本书中使用的基本软件包（以及它们的依赖项）：

```py
(pydata-book) $ conda install -y pandas jupyter matplotlib
```

我们还将使用其他软件包，但这些软件包可以在需要时稍后安装。有两种安装软件包的方法：使用`conda install`和`pip install`。在使用Miniconda时，应始终优先使用`conda install`，但某些软件包无法通过conda获得，因此如果`conda install $package_name`失败，请尝试`pip install $package_name`。

*注意* *如果您想安装本书其余部分使用的所有软件包，现在可以通过运行：

```py
conda install lxml beautifulsoup4 html5lib openpyxl \
               requests sqlalchemy seaborn scipy statsmodels \
               patsy scikit-learn pyarrow pytables numba
```

在Windows上，将`^`替换为Linux和macOS上使用的行继续符`\`。*  *您可以使用`conda` `update`命令更新软件包：

```py
conda update package_name
```

pip还支持使用`--upgrade`标志进行升级：

```py
pip install --upgrade package_name
```

您将有机会在整本书中尝试这些命令。

*注意* *虽然您可以使用conda和pip来安装软件包，但应避免使用pip更新最初使用conda安装的软件包（反之亦然），因为这样做可能会导致环境问题。我建议尽可能使用conda，并仅在无法使用`conda install`安装软件包时才回退到pip。***  ***### 集成开发环境和文本编辑器

当被问及我的标准开发环境时，我几乎总是说“IPython加上文本编辑器”。我通常会在IPython或Jupyter笔记本中编写程序，并逐步测试和调试每个部分。交互式地玩弄数据并直观验证特定数据操作是否正确也是很有用的。像pandas和NumPy这样的库旨在在shell中使用时提高生产力。

然而，在构建软件时，一些用户可能更喜欢使用功能更丰富的集成开发环境（IDE），而不是像Emacs或Vim这样的编辑器，后者在开箱即用时提供了更简洁的环境。以下是一些您可以探索的内容：

+   PyDev（免费），基于Eclipse平台构建的IDE

+   来自JetBrains的PyCharm（面向商业用户的订阅制，对于开源开发者免费）

+   Visual Studio的Python工具（适用于Windows用户）

+   Spyder（免费），目前与Anaconda捆绑的IDE

+   Komodo IDE（商业版）

由于Python的流行，大多数文本编辑器，如VS Code和Sublime Text 2，都具有出色的Python支持。****  ***## 1.5 社区和会议

除了通过互联网搜索外，各种科学和数据相关的Python邮件列表通常对问题有帮助并且响应迅速。一些可以参考的包括：

+   pydata：一个Google Group列表，用于与Python数据分析和pandas相关的问题

+   pystatsmodels：用于statsmodels或与pandas相关的问题

+   scikit-learn邮件列表（*scikit-learn@python.org*）和Python中的机器学习，一般

+   numpy-discussion：用于与NumPy相关的问题

+   scipy-user：用于一般SciPy或科学Python问题

我故意没有发布这些URL，以防它们发生变化。它们可以通过互联网搜索轻松找到。

每年举办许多全球各地的Python程序员会议。如果您想与其他分享您兴趣的Python程序员联系，我鼓励您尽可能参加其中一个。许多会议为那些无法支付入场费或旅行费的人提供财政支持。以下是一些可以考虑的会议：

+   PyCon和EuroPython：分别是在北美和欧洲举办的两个主要的一般Python会议

+   SciPy和EuroSciPy：分别是在北美和欧洲举办的面向科学计算的会议

+   PyData：面向数据科学和数据分析用例的全球系列区域会议

+   国际和地区 PyCon 会议（请参阅 [https://pycon.org](https://pycon.org) 获取完整列表）

## 1.6 浏览本书

如果您以前从未在 Python 中编程过，您可能需要花一些时间阅读 [第 2 章：Python 语言基础、IPython 和 Jupyter Notebooks](/book/python-basics) 和 [第 3 章：内置数据结构、函数和文件](/book/python-builtin)，我在这里放置了有关 Python 语言特性、IPython shell 和 Jupyter notebooks 的简明教程。这些内容是本书其余部分的先决知识。如果您已经有 Python 经验，您可以选择略读或跳过这些章节。

接下来，我简要介绍了 NumPy 的关键特性，将更高级的 NumPy 使用留给 [附录 A：高级 NumPy](/book/advanced-numpy)。然后，我介绍了 pandas，并将本书的其余部分专注于应用 pandas、NumPy 和 matplotlib 进行数据分析主题（用于可视化）。我以递增的方式组织了材料，尽管在章节之间偶尔会有一些轻微的交叉，有些概念可能尚未介绍。

尽管读者可能对他们的工作有许多不同的最终目标，但通常所需的任务大致可以分为许多不同的广泛组别：

与外部世界互动

使用各种文件格式和数据存储进行读写

准备

清理、整理、合并、规范化、重塑、切片和切块以及转换数据以进行分析

转换

对数据集组应用数学和统计操作以派生新数据集（例如，通过组变量对大表进行聚合）

建模和计算

将您的数据连接到统计模型、机器学习算法或其他计算工具

演示

创建交互式或静态图形可视化或文本摘要

### 代码示例

本书中的大多数代码示例都显示了输入和输出，就像在 IPython shell 或 Jupyter notebooks 中执行时一样：

```py
In [5]: CODE EXAMPLE
Out[5]: OUTPUT
```

当您看到像这样的代码示例时，意图是让您在编码环境中的 `In` 区块中键入示例代码，并通过按 Enter 键（或在 Jupyter 中按 Shift-Enter）执行它。您应该看到类似于 `Out` 区块中显示的输出。

我已更改了 NumPy 和 pandas 的默认控制台输出设置，以提高本书的可读性和简洁性。例如，您可能会看到在数字数据中打印更多位数的精度。要完全匹配书中显示的输出，您可以在运行代码示例之前执行以下 Python 代码：

```py
import numpy as np
import pandas as pd
pd.options.display.max_columns = 20
pd.options.display.max_rows = 20
pd.options.display.max_colwidth = 80
np.set_printoptions(precision=4, suppress=True)
```

### 示例数据

每一章的示例数据集都托管在 [GitHub 仓库](https://github.com/wesm/pydata-book) 中（如果无法访问 GitHub，则可以在 [Gitee 上的镜像](https://gitee.com/wesmckinn/pydata-book)）。您可以通过使用 Git 版本控制系统在命令行上下载这些数据，或者通过从网站下载仓库的 zip 文件来获取数据。如果遇到问题，请转到 [书籍网站](https://wesmckinney.com/book) 获取有关获取书籍材料的最新说明。

如果您下载包含示例数据集的 zip 文件，则必须完全提取 zip 文件的内容到一个目录，并在终端中导航到该目录，然后才能继续运行本书的代码示例：

```py
$ pwd
/home/wesm/book-materials

$ ls
appa.ipynb  ch05.ipynb  ch09.ipynb  ch13.ipynb  README.md
ch02.ipynb  ch06.ipynb  ch10.ipynb  COPYING     requirements.txt
ch03.ipynb  ch07.ipynb  ch11.ipynb  datasets
ch04.ipynb  ch08.ipynb  ch12.ipynb  examples
```

我已尽一切努力确保 GitHub 仓库包含重现示例所需的一切，但可能会出现一些错误或遗漏。如果有的话，请发送邮件至：*book@wesmckinney.com*。报告书中错误的最佳方式是在 [O'Reilly 网站上的勘误页面](https://oreil.ly/kmhmQ)上。

### 导入约定

Python 社区已经采用了许多常用模块的命名约定：

```py
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns
import statsmodels as sm
```

这意味着当你看到`np.arange`时，这是对NumPy中`arange`函数的引用。这样做是因为在Python软件开发中，从像NumPy这样的大型包中导入所有内容（`from numpy import *`）被认为是不良实践。
