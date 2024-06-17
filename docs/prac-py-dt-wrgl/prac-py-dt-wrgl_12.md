# 附录 A. 更多 Python 编程资源

正如本书希望揭示的那样，Python 是一种强大而灵活的编程语言，具有广泛的应用。虽然在前几章中我介绍了许多关键概念和流行库，但我创建了这个附录，为你提供一些有用的资源和参考资料，帮助你将 Python 工作提升到更高水平。

# 官方 Python 文档

是的，总是有搜索引擎和[StackOverflow](https://stackoverflow.com/questions/tagged/python)，但习惯阅读官方文档确实有其价值 — 无论是针对 Python 语言还是流行的库如 `pandas`、`matplotlib` 或 `seaborn`。虽然我不建议你坐下来逐页阅读 *任何* 编程文档，但浏览你想要使用的数据类型或函数的参数和选项，可以让你更好地理解它（通常）可以做什么，以及其机制如何组织。当你想做全新的事情时，这尤其有帮助，因为它会指引你寻找前进的路径。

例如，我在写这本书时就知道 `seaborn` 和 `pandas` 都是基于 `matplotlib` 构建的，我也曾尝试通过它们来制作和定制图形。然而，直到我仔细查阅后者的文档时，我才明白我经常在示例代码中看到的 `figure` 和 `axes` 对象之间的区别，这种理解帮助我在实验不同方式全面定制可视化时更快地找到解决方案。几乎同样重要的是，官方文档通常保持更新，而关于某个主题的最流行论坛帖子往往可以过时数月甚至数年 — 这意味着它们中包含的建议有时可能已经大大过时。

我还建议定期查阅[Python 标准库](https://docs.python.org/3/library/index.html)，因为你可能会惊讶于它提供的大量内置功能。你在使用库时熟悉的许多方法都是基于（或模仿）存在于“纯” Python 中的函数。虽然库通常是独特和有用的，但并不能保证它们会继续开发或维护。如果你可以通过使用“纯” Python 获得所需的功能，那么你的代码依赖于不再更新的库的风险就越小。

# 安装 Python 资源

根据你的编程环境，有很多种方法可以安装 Python 包。无论你是在 macOS 上使用 Homebrew，还是在 Windows 机器上工作，或者使用 Colab，安装 Python 包的最可靠方法[几乎总是使用某个版本的 `pip`](https://packaging.python.org/tutorials/installing-packages)。事实上，你甚至可以使用 `pip` 在 Google Colab 笔记本上安装包（如果你找到了尚未安装的包）[使用以下语法](https://colab.research.google.com/notebooks/snippets/importing_libraries.ipynb)：

```py
!pip install *librarynamehere*

```

无论你选择什么，我建议你做出选择并坚持下去——如果你开始使用多个工具来安装和更新你的 Python 环境，事情可能会变得非常不可预测。

## 寻找库的位置

大部分情况下，我建议你安装那些在[Python Package Index](https://pypi.org)（也称为 PyPI）上可用的 Python 包。 PyPI 有清晰的结构和强大的标签和搜索界面，使得查找有用的 Python 包变得很容易，而且PyPI 包的文档通常具有标准的布局（如[图 A-1](#pypi_docs_example)所示），如果你要浏览大量选项，这将极大地节省你的时间。

一些项目（如 Beautiful Soup `lxml`）可能仍然将大部分文档保存在独立的位置，但它们的 PyPI 页面通常仍然包含项目功能的有益摘要，甚至一些入门提示。我个人喜欢看的其中一件事是“发布历史”部分，显示项目的首次创建时间，以及包的更新频率和最近更新时间。当然，项目的长期存在并不是评估给定包可靠性的完美指标——因为任何人都可以发布 Python 包（并将其添加到 PyPI 中）——但那些存在时间较长和/或最近更新频繁的包通常是一个很好的起点。

![示例 PyPI 包页面。](assets/ppdw_aa01.png)

###### 图 A-1\. 示例 PyPI 包页面

# 保持你的工具锋利

编程工具一直在更新，因为社区发现（并且大多数情况下修复）新问题，或者达成共识，某个软件工具的某些方面（甚至Python本身！）应该有所不同。如果您按照[第1章](ch01.html#chapter1)中提供的安装Python和Jupyter Notebook的说明进行操作，那么您可以使用`conda`命令来更新这两者。您只需偶尔运行`conda update python`和`conda update jupyter`。与此同时，因为本书面向初学者，我在[第1章](ch01.html#chapter1)中没有涉及Python *环境*的问题。在这个背景下，一个给定的*环境*主要由默认使用的Python主版本（例如3.9与3.8）来定义，当您使用`python`命令时。虽然运行`conda update python`将更新，比如，版本3.8.6到3.8.11，它 *不会* 自动将您的版本更新到不同的主要发布版本（例如3.9）。通常情况下，这不会给您造成问题，除非您的Python安装已经过时了几年（因此是主要版本）。如果发生这种情况，您将需要为新版本的Python创建一个新的环境，主要是因为否则您的计算机可能很难保持清晰。幸运的是，当您升级Python到下一个主要版本时，您可以从[`conda`文档页面](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html)了解如何操作。

# 如何获取更多信息

因为Python是如此受欢迎的编程语言，因此在线资源和图书馆中有数千种资源，可帮助您进入下一个学习阶段，无论是完成特定项目还是仅仅想了解更多关于机器学习和自然语言处理等高级主题。

对于更深入的数据科学主题的实用、简洁介绍，我的首选推荐会是[*Python数据科学手册*](https://www.oreilly.com/library/view/python-data-science/9781491912126/)，作者是Jake VanderPlas（O’Reilly）——这是我在试图理解Python可以进行的更高级机器学习工作时自己使用过的资源，Jake清晰的写作风格和简洁的示例提供了一种既能够概述Python机器学习，又能学习具体方法（如k-means聚类）的易于理解的方式。更好的是，你可以在[网上免费获取整本书](https://jakevdp.github.io/PythonDataScienceHandbook)——虽然如果可以的话，我强烈建议购买一本实体书。

或许比找到合适的书籍、教程或课程更有价值的是，在你继续进行数据整理的旅程中找到一个可以与之共同学习和工作的人群社区。不论是通过学校、社区组织还是聚会，找到一个小团体，你可以与他们讨论你的项目，寻求建议（有时也是共鸣！），可能是扩展你在Python和数据整理方面技能最宝贵的资源。还有许多无费的项目可以支持并推动你的工作，尤其是来自技术领域中代表性不足的群体。看看像[Ada Developers Academy](https://adadevelopersacademy.org)或[Free Code Camp](https://freecodecamp.org)这样的团体。
