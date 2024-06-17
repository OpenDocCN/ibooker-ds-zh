# 第10章。Python初学者的第一步

Python是由Guido van Rossum于1991年创建的编程语言，与R一样，是自由且开源的。当时，van Rossum正在阅读《蒙提·派森的飞行马戏团》的剧本，并决定以这部英国喜剧命名这种语言。与专门为数据分析设计的R不同，Python被开发为一种通用语言，用于与操作系统交互、处理处理错误等任务。这对Python如何“思考”和处理数据有一些重要的影响。例如，在[第7章](ch07.html#data-structures-in-r)中你看到，R有一个内置的表格数据结构。而Python并非如此；我们将更多地依赖外部包来处理数据。

这并不一定是一个问题：与R一样，Python有数千个由充满活力的贡献者社区维护的包。你会发现Python被用于从移动应用开发到嵌入式设备再到数据分析等各个领域。它的多样化用户群体正在迅速增长，Python不仅成为了最受欢迎的分析编程语言之一，也成为了计算领域的一种重要工具。

###### 注意

Python最初被设计为一种通用编程语言，而R则是专门用于统计分析。

# 下载Python

[Python软件基金会](https://python.org)维护着“官方”的Python源代码。由于Python是开源的，任何人都可以获取、添加和重新分发Python代码。Anaconda是其中一种Python发行版，也是本书建议的安装方式。它由同名的盈利公司维护，提供了付费版本；我们将使用免费的个人版本。Python现在已经到了第三个版本，Python 3。您可以在[Anaconda的网站](https://oreil.ly/3RYeQ)下载Python 3的最新版本。

除了简化的Python安装外，Anaconda还提供了额外的内容，包括一些我们在本书中稍后会使用的流行包。它还附带了一个Web应用程序，我们将用它来处理Python：Jupyter Notebook。

# 开始使用Jupyter

如[第6章](ch06.html#first-steps-r)中所述，R是在EDA程序S的基础上建模的。由于EDA的迭代性质，语言的预期工作流程是执行和探索所选代码行的输出。这使得直接从R脚本*.r进行数据分析变得容易。我们使用了RStudio IDE，为我们的编程会话提供了额外的支持，例如专门用于帮助文档和环境中对象信息的窗格。

相比之下，Python 在某些方面更像是“低级”编程语言，其中代码首先需要编译成机器可读文件，*然后*再运行。这使得从 Python 脚本 *.py* 进行分步数据分析相对更加棘手。物理学家兼软件开发者费尔南多·佩雷斯及其同事在 2001 年启动了 IPython 项目，以使 Python 更加交互式（IPython 是对“交互式 Python”的俏皮简称）。这一倡议的一个结果是一种新类型的文件，即 *IPython Notebook*，以 *.ipynb* 文件扩展名表示。

该项目逐渐受到关注，并于2014年发展成更广泛的 Jupyter 项目，这是一个与语言无关的倡议，旨在开发交互式、开源的计算软件。因此，IPython Notebook 成为了 Jupyter Notebook，同时保留了 *.ipynb* 扩展名。Jupyter 笔记本作为交互式网络应用程序运行，允许用户将代码与文本、方程式等结合起来，创建媒体丰富的交互式文档。事实上，Jupyter 在一定程度上是为了向伽利略用于记录其发现木星卫星的笔记本致敬而命名的。*内核* 在幕后用于执行笔记本的代码。通过下载 Anaconda，您已经设置好了执行 Python 的 Jupyter 笔记本所需的所有必要部分：现在您只需要启动一个会话。

启动 Jupyter 笔记本的步骤因 Windows 和 Mac 计算机而异。在 Windows 上，打开“开始”菜单，然后搜索并启动 `Anaconda Prompt`。这是一个用于处理您的 Anaconda 发行版的命令行工具，也是与 Python 代码交互的另一种方式。在 Anaconda 提示符内部，输入 `jupyter notebook` 并按 `Enter` 键。您的命令将类似于以下内容，但主目录路径不同：

```py
(base) C:\Users\User> jupyter notebook
```

在 Mac 上，打开 Launchpad，然后搜索并启动 Terminal。这是随 Mac 一起提供的命令行界面，可用于与 Python 进行通信。在终端提示符内部，输入 `jupyter notebook` 并按 `Enter` 键。您的命令行将类似于以下内容，但主目录路径不同：

```py
user@MacBook-Pro ~ % jupyter notebook
```

在任一系统上执行此操作后，将会发生几件事情：首先，会在您的计算机上启动一个类似终端的额外窗口。*请勿关闭此窗口。* 这将建立与内核的连接。此外，Jupyter 笔记本界面应会自动在您的默认网络浏览器中打开。如果没有自动打开，终端窗口中会包含一个您可以复制并粘贴到浏览器中的链接。图 10-1 中展示了您在浏览器中应该看到的内容。Jupyter 启动时会显示一个类似文件浏览器的界面。现在，您可以导航至您希望保存笔记本的文件夹。

![Jupyter 起始页面](assets/aina_1001.png)

###### 图 10-1\. Jupyter 起始页面

要打开一个新的笔记本，请转至浏览器窗口右上角，选择 New → Notebook → Python 3\. 一个新标签页将打开，显示一个空白的 Jupyter 笔记本。与 RStudio 类似，Jupyter 提供的功能远远超出了我们在介绍中所能覆盖的范围；我们将重点介绍一些关键要点，以帮助您入门。Jupyter 笔记本的四个主要组成部分在 [Figure 10-2](#jupyter-interface) 中进行了标注；让我们逐一了解一下。

![Jupyter 笔记本界面截图](assets/aina_1002.png)

###### 图 10-2\. Jupyter 界面的组成元素

首先是笔记本的名称：这是我们的 *.ipynb* 文件的名称。您可以通过点击并在当前名称上进行编辑来重新命名笔记本。

接下来是菜单栏。这包含了不同的操作，用于处理您的笔记本。例如，在 File 下，您可以打开和关闭笔记本。保存它们并不是什么大问题，因为 Jupyter 笔记本会每两分钟自动保存一次。如果您需要将笔记本转换为 *.py* Python 脚本或其他常见文件类型，可以通过 File → Download as 完成。还有一个 Help 部分，包含了几个指南和链接到参考文档。您可以从此菜单了解 Jupyter 的许多键盘快捷键。

之前，我提到过 *内核* 是 Jupyter 在幕后与 Python 交互的方式。菜单栏中的 *Kernel* 选项包含了与之配合工作的有用操作。由于计算机的工作特性，有时候使您的 Python 代码生效所需的仅仅是重新启动内核。您可以通过 Kernel → Restart 来执行此操作。

立即位于菜单栏下方的是工具栏。这包含了一些有用的图标，用于处理您的笔记本，比通过菜单进行导航更加方便：例如，这里的几个图标与与内核的交互相关。

在 Jupyter 中，您会花费大部分时间，您可以在笔记本中插入和重新定位 *单元格*。要开始，请使用工具栏进行最后一项操作：您会在那里找到一个下拉菜单，当前设置为 Code；将其更改为 Markdown。

现在，导航到您的第一个代码单元格，并输入短语 `Hello, Jupyter!` 然后返回工具栏并选择运行图标。会发生几件事情。首先，您会看到您的 `Hello, Jupyter!` 单元格渲染成为类似文字处理文档的样子。接下来，您会看到一个新的代码单元格放置在前一个单元格下面，并且已经设置好让您输入更多信息。您的笔记本应该类似于 [Figure 10-3](#hello-world-jupyter)。

![Jupyter keyboard shortcuts](assets/aina_1003.png)

###### 图 10-3\. “Hello, Jupyter!”

现在，返回工具栏并再次选择“Markdown”从下拉菜单中选择。正如您开始了解的那样，Jupyter 笔记本由可以是不同类型的模块单元格组成。我们将专注于两种最常见的类型：Markdown 和 Code。Markdown 是一种使用常规键盘字符来格式化文本的纯文本标记语言。

将以下文本插入您的空白单元格中：

```py
# Big Header 1
## Smaller Header 2
### Even smaller headers
#### Still more

*Using one asterisk renders italics*

**Using two asterisks renders bold**

- Use dashes to...
- Make bullet lists

Refer to code without running it as `fixed-width text`
```

现在运行该单元格：您可以从工具栏或使用快捷键 Alt + Enter（Windows）、Option + Return（Mac）来完成。您的选择将呈现如 [Figure 10-4](#jupyter-markdown) 所示。

![Markdown in Jupyter](assets/aina_1004.png)

###### 图 10-4\. Jupyter 中 Markdown 格式化的示例

要了解更多关于 Markdown 的信息，请返回菜单栏中的帮助部分。值得学习，以构建优雅的笔记本，可以包括图像、方程式等。但在本书中，我们将专注于 *code* 块，因为那里可以执行 Python 代码。您现在应该在第三个代码单元格上；您可以将此保留为代码格式。最后，我们将在 Python 中进行编码。

Python 可以像 Excel 和 R 一样用作高级计算器。[Table 10-1](#python-arithmetic) 列出了 Python 中一些常见的算术运算符。

表 10-1\. Python 中常见的算术运算符

| 运算符 | 描述 |
| --- | --- |
| `+` | 加法 |
| `-` | 减法 |
| `*` | 乘法 |
| `/` | 除法 |
| `**` | 指数 |
| `%%` | 取模 |
| `//` | 地板除法 |

输入以下算术，然后运行单元格：

```py
In [1]: 1 + 1
Out[1]: 2
```

```py
In [2]: 2 * 4
Out[2]: 8
```

当执行 Jupyter 代码块时，它们分别用 `In []` 和 `Out []` 标记其输入和输出。

Python 也遵循操作顺序；让我们试着在同一个单元中运行几个例子：

```py
In [3]: # Multiplication before addition
        3 * 5 + 6
        2 / 2 - 7 # Division before subtraction
Out[3]: -6.0
```

默认情况下，Jupyter 笔记本仅返回单元格中最后一次运行代码的输出，所以我们将其分成两部分。您可以使用键盘快捷键 Ctrl + Shift + -（减号）在 Windows 或 Mac 上在光标处分割单元格：

```py
In [4]:  # Multiplication before addition
         3 * 5 + 6

Out[4]: 21
```

```py
In [5]:  2 / 2 - 7 # Division before subtraction

Out[5]: -6.0
```

是的，Python 也使用代码注释。与 R 类似，它们以井号开头，并且最好将它们保持在单独的行上。

像 Excel 和 R 一样，Python 包括许多用于数字和字符的函数：

```py
In [6]: abs(-100)

Out[6]: 100
```

```py
In [7]: len('Hello, world!')

Out[7]: 13
```

与 Excel 不同，但类似于 R，Python 是区分大小写的。这意味着 *只有* `abs()` 起作用，而不是 `ABS()` 或 `Abs()`。

```py
In [8]:  ABS(-100)

        ------------------------------------------------------------------------
        NameError                              Traceback (most recent call last)
        <ipython-input-20-a0f3f8a69d46> in <module>
        ----> 1 print(ABS(-100))
            2 print(Abs(-100))

        NameError: name 'ABS' is not defined
```

与 R 类似，你可以使用 `?` 运算符获取有关函数、包等的信息。会打开一个窗口，就像 [图 10-5](#jupyter-help) 中展示的那样，你可以展开它或在新窗口中打开。

![在 Jupyter 中的帮助文档](assets/aina_1005.png)

###### 图 10-5\. 在 Jupyter 笔记本中打开帮助文档

比较运算符在 Python 中与 R 和 Excel 大多数情况下工作方式相同；在 Python 中，结果要么返回 `True` 要么返回 `False`。

```py
In [10]: # Is 3 greater than 4?
         3 > 4

Out[10]: False
```

与 R 一样，你可以用 `==` 检查两个值是否相等；单个等号 `=` 用于赋值对象。我们在 Python 中会一直使用 `=` 来赋值对象。

```py
In [11]:  # Assigning an object in Python
          my_first_object = abs(-100)
```

你可能注意到第 11 格单元中没有 `Out []` 组件。那是因为我们只是 *赋值* 了对象；我们没有打印任何内容。现在让我们来做一下：

```py
In [12]: my_first_object

Out[12]: 100
```

Python 中的对象名称必须以字母或下划线开头，其余部分只能包含字母、数字或下划线。还有一些禁用的关键字。同样，你可以广泛命名 Python 对象，但并不意味着你应该把对象命名为 `scooby_doo`。

就像在 R 中一样，我们的 Python 对象可以由不同的数据类型组成。[表 10-2](#data-types-python) 展示了一些基本的 Python 类型。你能看出与 R 的相似性和差异吗？

表 10-2\. Python 的基本对象类型

| 数据类型 | 示例 |
| --- | --- |
| 字符串 | `'Python'`, `'G. Mount'`, `'Hello, world!'` |
| 浮点数 | `6.2`, `4.13`, `3.1` |
| 整数 | `3`, `-1`, `12` |
| 布尔值 | `True`, `False` |

让我们给一些对象赋值。我们可以用 `type()` 函数找出它们的类型：

```py
In [13]:  my_string = 'Hello, world'
          type(my_string)

Out[13]: str

In [14]: # Double quotes work for strings, too
         my_other_string = "We're able to code Python!"
         type(my_other_string)

Out[14]: str

In [15]: my_float = 6.2
         type(my_float)

Out[15]: float

In [16]: my_integer = 3
         type(my_integer)

Out[16]: int

In [17]: my_bool = True
         type(my_bool)

Out[17]: bool
```

你已经在 R 中使用对象了，所以可能不会对它们作为 Python 操作的一部分感到意外。

```py
In [18]:  # Is my_float equal to 6.1?
          my_float == 6.1

Out[18]: False

In [19]:  # How many characters are in my_string?
          # (Same function as Excel)
          len(my_string)

Out[19]: 12
```

Python 中与函数密切相关的是 *方法*。方法通过一个点与对象关联，并对该对象执行某些操作。例如，要将字符串对象中的所有字母大写，我们可以使用 `upper()` 方法：

```py
In [20]:  my_string.upper()

Out[20]: 'HELLO, WORLD'
```

函数和方法都用于对对象执行操作，在本书中我们将同时使用两者。就像你可能希望的那样，Python 和 R 一样，可以在单个对象中存储多个值。但在深入讨论之前，让我们先来看看 *模块* 在 Python 中是如何工作的。

# Python 模块

Python 被设计为通用编程语言，因此甚至一些最简单的用于处理数据的函数也不是默认可用的。例如，我们找不到一个可以计算平方根的函数：

```py
In [21]:  sqrt(25)

        ----------------------------------------------------------
        NameError              Traceback (most recent call last)
        <ipython-input-18-1bf613b64533> in <module>
        ----> 1 sqrt(25)

        NameError: name 'sqrt' is not defined
```

此函数在 Python 中确实存在。但要访问它，我们需要引入一个 *模块*，它类似于 R 中的包。Python 标准库中预装了许多模块；例如，`math` 模块包含许多数学函数，包括 `sqrt()`。我们可以通过 `import` 语句将此模块引入到我们的会话中：

```py
In [22]:  import math
```

语句是告诉解释器要做什么的指令。我们刚刚告诉 Python，嗯，*导入* `math` 模块。现在 `sqrt()` 函数应该对我们可用了；试试看吧：

```py
In [23]:  sqrt(25)

        ----------------------------------------------------------
        NameError              Traceback (most recent call last)
        <ipython-input-18-1bf613b64533> in <module>
        ----> 1 sqrt(25)

        NameError: name 'sqrt' is not defined
```

说实话，我并没有在 `sqrt()` 函数上说谎。我们仍然遇到错误的原因是我们需要明确告诉 Python *这个* 函数来自哪里。我们可以通过在函数前加上模块名来做到这一点，像这样：

```py
In [24]:  math.sqrt(25)

Out[24]: 5.0
```

标准库中充满了有用的模块。然后是数千个第三方模块，捆绑成*包*并提交到 Python 包索引中。`pip` 是标准的包管理系统；它可以用于从 Python 包索引以及外部源安装。

Anaconda 发行版在处理包时已经做了很多工作。首先，一些最受欢迎的 Python 包是预安装的。此外，Anaconda 包括功能以确保您计算机上的所有包都是兼容的。因此，最好直接从 Anaconda 安装包而不是从 `pip` 安装。通常情况下，Python 包的安装是通过命令行完成的；您之前在 Anaconda Prompt（Windows）或 Terminal（Mac）中已经使用过命令行。不过，我们可以通过在 Jupyter 前加上感叹号来执行命令行代码。让我们从 Anaconda 安装一个流行的数据可视化包 `plotly`。使用的语句是 `conda install`：

```py
In [25]:  !conda install plotly
```

并非所有包都可以从 Anaconda 下载；在这种情况下，我们可以通过 `pip` 进行安装。让我们为 `pyxlsb` 包做这个，它可以用来将二进制 `.xlsb` Excel 文件读取到 Python 中：

```py
In [26]:  !pip install pyxlsb
```

尽管直接从 Jupyter 下载包很方便，但对于其他人来说，尝试运行您的笔记本只会遇到漫长或不必要的下载可能是一个不愉快的惊喜。这就是为什么通常注释掉安装命令是一个约定，我在书的代码库中也遵循这个约定。

###### 注意

如果您使用 Anaconda 运行 Python，最好首先通过 `conda` 安装东西，只有在包不可用时才通过 `pip` 安装。

# 升级 Python、Anaconda 和 Python 包

[Table 10-3](#python-update-options) 列出了维护您的 Python 环境的几个其他有用的命令。您还可以使用 Anaconda Navigator 通过点和点击界面安装和维护 Anaconda 包，Anaconda Navigator 与 Anaconda 个人版一起安装。要开始，请在计算机上启动该应用程序，然后转到帮助菜单以阅读更多文档。

表 10-3\. 维护 Python 包的有用命令

| 命令 | 描述 |
| --- | --- |
| `conda update anaconda` | 更新 Anaconda 发行版 |
| `conda update python` | 更新 Python 版本 |
| `conda update -- all` | 更新通过 `conda` 下载的所有可能的包 |
| `pip list -- outdated` | 列出所有可以更新的通过 `pip` 下载的包 |

# 结论

在本章中，你学会了如何在Python中处理对象和包，并且掌握了使用Jupyter笔记本的技巧。

# 练习：

以下练习提供了关于这些主题的额外练习和深入见解：

1.  从一个新的Jupyter笔记本开始，做以下操作：

    +   将1和-4的和分配给变量`a`。

    +   将变量`a`的绝对值分配给变量`b`。

    +   将`b`减1的结果分配给变量`d`。

    +   变量`d`是否大于2？

1.  Python标准库包括一个名为`random`的模块，其中包含一个名为`randint()`的函数。这个函数的用法类似于Excel中的`RANDBETWEEN()`；例如，`randint(1, 6)`会返回一个介于1和6之间的整数。使用这个函数找到一个介于0和36之间的随机数。

1.  Python标准库还包括一个名为`this`的模块。当你导入该模块时会发生什么？

1.  从Anaconda下载`xlutils`包，然后使用`?`操作符检索可用的文档。

我再次鼓励你尽快在日常工作中开始使用这种语言，即使一开始只是作为计算器。你也可以尝试在R和Python中执行相同的任务，然后进行比较和对比。如果你是通过将R与Excel相关联来学习的，那么将Python与R相关联也是一样的道理。
