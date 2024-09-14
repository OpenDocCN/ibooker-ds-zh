# 第二章：增强交互功能

IPython 和 Jupyter 的大部分功能来自它们提供的额外交互工具。本章将涵盖其中一些工具，包括所谓的魔术命令，用于探索输入和输出历史记录的工具，以及与 Shell 交互的工具。

# IPython 魔术命令

前一章展示了 IPython 如何让您高效、交互地使用和探索 Python。在这里，我们将开始讨论 IPython 在正常 Python 语法之上添加的一些增强功能。这些在 IPython 中被称为*魔术命令*，以`%`字符为前缀。这些魔术命令设计精炼地解决标准数据分析中的各种常见问题。魔术命令有两种类型：*行魔术*，以单个`%`前缀表示，作用于单行输入；*单元格魔术*，以双`%%`前缀表示，作用于多行输入。我将在这里展示和讨论一些简短的示例，并稍后回到更专注地讨论几个有用的魔术命令。

## 运行外部代码：`%run`

随着您开始开发更多的代码，您可能会发现自己在 IPython 中进行交互式探索，以及在文本编辑器中存储希望重用的代码。与其在新窗口中运行此代码，不如在您的 IPython 会话中运行更方便。可以通过`%run`魔术命令完成此操作。

例如，假设您创建了一个*myscript.py*文件，其中包含以下内容：

```py
# file: myscript.py

def square(x):
    """square a number"""
    return x ** 2

for N in range(1, 4):
    print(f"{N} squared is {square(N)}")
```

你可以从你的 IPython 会话中执行以下操作：

```py
In [1]: %run myscript.py
1 squared is 1
2 squared is 4
3 squared is 9
```

还要注意，在运行此脚本后，其中定义的任何函数都可以在您的 IPython 会话中使用：

```py
In [2]: square(5)
Out[2]: 25
```

有几种选项可以微调代码的运行方式；您可以通过在 IPython 解释器中键入**`%run?`**来查看正常的文档。

## 代码执行时间：`%timeit`

另一个有用的魔术函数示例是`%timeit`，它将自动确定其后的单行 Python 语句的执行时间。例如，我们可能希望检查列表理解的性能：

```py
In [3]: %timeit L = [n ** 2 for n in range(1000)]
430 µs ± 3.21 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
```

`%timeit` 的好处在于对于短命令，它会自动执行多次运行以获得更稳健的结果。对于多行语句，添加第二个`%`符号将其转换为可以处理多行输入的单元格魔术。例如，这里是使用`for`循环的等效构造：

```py
In [4]: %%timeit
   ...: L = []
   ...: for n in range(1000):
   ...:     L.append(n ** 2)
   ...:
484 µs ± 5.67 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
```

我们可以立即看到，在这种情况下，列表理解比等效的`for`循环结构快大约 10%。我们将在“分析和计时代码”中探索`%timeit`和其他计时和分析代码的方法。

## 魔术函数帮助：？，%magic 和%lsmagic

与普通的 Python 函数类似，IPython 的魔术函数有文档字符串，可以以标准方式访问这些有用的文档。例如，要查阅 `%timeit` 魔术函数的文档，只需输入以下内容：

```py
In [5]: %timeit?
```

类似地，还可以访问其他函数的文档。要访问可用魔术函数的一般描述，包括一些示例，请输入以下内容：

```py
In [6]: %magic
```

要快速简单地列出所有可用的魔术函数，请输入以下内容：

```py
In [7]: %lsmagic
```

最后，我将提到，如果你愿意的话，定义自己的魔术函数非常简单。我不会在这里讨论它，但如果你有兴趣，可以参考 “更多 IPython 资源” 中列出的参考资料。

# 输入和输出历史

正如之前所见，IPython shell 允许你使用上下箭头键（或等效的 Ctrl-p/Ctrl-n 快捷键）访问先前的命令。此外，在 shell 和笔记本中，IPython 还提供了几种获取先前命令输出以及命令字符串版本的方法。我们将在这里探讨这些方法。

## IPython 的 In 和 Out 对象

现在我想象你已经开始熟悉 IPython 使用的 `In [1]:`/`Out[1]:` 样式的提示了。但事实证明，这些不仅仅是漂亮的装饰：它们提供了一种方法，让你可以访问当前会话中的先前输入和输出。假设我们开始一个看起来像这样的会话：

```py
In [1]: import math

In [2]: math.sin(2)
Out[2]: 0.9092974268256817

In [3]: math.cos(2)
Out[3]: -0.4161468365471424
```

我们已经导入了内置的 `math` 包，然后计算了数字 2 的正弦和余弦。这些输入和输出显示在带有 `In`/`Out` 标签的 shell 中，但是 IPython 实际上创建了一些名为 `In` 和 `Out` 的 Python 变量，这些变量会自动更新以反映这些历史记录：

```py
In [4]: In
Out[4]: ['', 'import math', 'math.sin(2)', 'math.cos(2)', 'In']

In [5]: Out
Out[5]:
{2: 0.9092974268256817,
 3: -0.4161468365471424,
 4: ['', 'import math', 'math.sin(2)', 'math.cos(2)', 'In', 'Out']}
```

`In` 对象是一个列表，按顺序保存命令（列表的第一项是一个占位符，以便 `In [1]` 可以引用第一条命令）：

```py
In [6]: print(In[1])
import math
```

`Out` 对象不是一个列表，而是一个将输入编号映射到它们的输出（如果有的话）的字典：

```py
In [7]: print(Out[2])
.9092974268256817
```

注意，并非所有操作都有输出：例如，`import` 语句和 `print` 语句不会影响输出。后者可能会令人惊讶，但如果考虑到 `print` 是一个返回 `None` 的函数，这是有道理的；为简洁起见，任何返回 `None` 的命令都不会添加到 `Out` 中。

当你想要与过去的结果交互时，这可能会很有用。例如，让我们使用先前计算的结果来检查 `sin(2) ** 2` 和 `cos(2) ** 2` 的总和：

```py
In [8]: Out[2] ** 2 + Out[3] ** 2
Out[8]: 1.0
```

结果是 `1.0`，这符合我们对这个著名的三角恒等式的预期。在这种情况下，使用这些先前的结果可能是不必要的，但如果你执行了一个非常昂贵的计算并且忘记将结果赋给一个变量，这可能会变得非常方便。

## 下划线快捷键和先前的输出

标准 Python shell 仅包含一种简单的快捷方式用于访问先前的输出：变量 `_`（即一个下划线）会随先前的输出更新。这在 IPython 中也适用：

```py
In [9]: print(_)
.0
```

但是 IPython 进一步扩展了这一点 — 您可以使用双下划线访问倒数第二个输出，并使用三个下划线访问倒数第三个输出（跳过任何没有输出的命令）：

```py
In [10]: print(__)
-0.4161468365471424

In [11]: print(___)
.9092974268256817
```

IPython 到此为止：超过三个下划线开始变得有点难以计数，此时通过行号更容易引用输出。

我还应该提一下另一个快捷方式 — `Out[*X*]` 的简写是 `_*X*`（即一个下划线后跟行号）：

```py
In [12]: Out[2]
Out[12]: 0.9092974268256817

In [13]: _2
Out[13]: 0.9092974268256817
```

## 禁止输出

有时候，您可能希望抑制语句的输出（这在我们将在第四部分探索的绘图命令中可能最常见）。或者，您执行的命令会产生一个您不希望存储在输出历史记录中的结果，也许是因为在其他引用被移除时它可以被释放。抑制命令的输出最简单的方法是在行尾添加分号：

```py
In [14]: math.sin(2) + math.cos(2);
```

结果将在不显示在屏幕上或存储在 `Out` 字典中的情况下计算：

```py
In [15]: 14 in Out
Out[15]: False
```

## 相关的魔术命令

要一次访问一批先前的输入，`%history` 魔术命令非常有帮助。以下是如何打印前四个输入：

```py
In [16]: %history -n 1-3
   1: import math
   2: math.sin(2)
   3: math.cos(2)
```

如往常一样，您可以输入 `%history?` 以获取更多信息并查看可用选项的描述（有关 `?` 功能的详细信息，请参见第一章）。其他有用的魔术命令包括 `%rerun`，它将重新执行命令历史记录的某些部分，以及 `%save`，它将命令历史记录的某个集合保存到文件中。

# IPython 和 Shell 命令

在与标准 Python 解释器的交互工作时，一个令人沮丧的地方是需要在多个窗口之间切换以访问 Python 工具和系统命令行工具。IPython 弥合了这一差距，并为您提供了在 IPython 终端内直接执行 shell 命令的语法。这是通过感叹号实现的：在 `!` 后出现的任何内容将不会由 Python 内核执行，而是由系统命令行执行。

以下讨论假定您正在使用类 Unix 系统，如 Linux 或 macOS。接下来的一些示例在 Windows 上会失败，因为 Windows 默认使用不同类型的 shell，但如果您使用[*Windows 子系统 for Linux*](https://oreil.ly/H5MEE)，这里的示例应该能正常运行。如果您对 shell 命令不熟悉，我建议您查看由始终优秀的软件教程基金会组织的[Unix shell 教程](https://oreil.ly/RrD2Y)。

## Shell 快速入门

对于如何使用 shell/终端/命令行的全面介绍远远超出了本章的范围，但对于未接触过的人，我将在这里进行一个快速介绍。shell 是一种与计算机进行文本交互的方式。自从 20 世纪 80 年代中期以来，当微软和苹果推出了他们现在无处不在的图形操作系统的第一个版本时，大多数计算机用户通过熟悉的菜单选择和拖放操作与他们的操作系统交互。但是操作系统在这些图形用户界面之前就存在，主要通过文本输入的序列来控制：在提示符下，用户会输入一个命令，计算机会执行用户告诉它做的事情。那些早期的提示系统是今天大多数数据科学家仍在使用的 shell 和终端的前身。

对于不熟悉 shell 的人可能会问，为什么要费这个劲，当很多相同的结果可以通过简单点击图标和菜单来实现呢？一个 shell 用户可能会反问：为什么要去寻找图标和菜单项，而不是通过输入命令来更轻松地完成任务呢？虽然这听起来像是典型的技术偏好僵局，但当超越基本任务时，很快就会明显地感觉到 shell 在控制高级任务方面提供了更多的控制权——尽管学习曲线确实可能令人望而生畏。

例如，这里是一个 Linux/macOS shell 会话的示例，用户在其系统上探索、创建和修改目录和文件（`osx:~ $`是提示符，`$`之后的所有内容是键入的命令；以`#`开头的文本仅用作描述，而不是您实际要键入的内容）：

```py
osx:~ $ echo "hello world"            # echo is like Python's print function
hello world

osx:~ $ pwd                            # pwd = print working directory
/home/jake                             # This is the "path" that we're sitting in

osx:~ $ ls                             # ls = list working directory contents
notebooks  projects

osx:~ $ cd projects/                   # cd = change directory

osx:projects $ pwd
/home/jake/projects

osx:projects $ ls
datasci_book   mpld3   myproject.txt

osx:projects $ mkdir myproject          # mkdir = make new directory

osx:projects $ cd myproject/

osx:myproject $ mv ../myproject.txt ./  # mv = move file. Here we're moving the
                                        # file myproject.txt from one directory
                                        # up (../) to the current directory (./).
osx:myproject $ ls
myproject.txt

```

请注意，所有这些只是通过键入命令而不是点击图标和菜单来执行熟悉操作（导航目录结构、创建目录、移动文件等）的一种紧凑方式。仅仅几个命令（`pwd`、`ls`、`cd`、`mkdir`和`cp`）就可以完成许多最常见的文件操作，但当您超越这些基础操作时，shell 方法真正显示其强大之处。

## IPython 中的 Shell 命令

任何标准的 shell 命令都可以通过在其前面加上`!`字符直接在 IPython 中使用。例如，`ls`、`pwd`和`echo`命令可以如下运行：

```py
In [1]: !ls
myproject.txt

In [2]: !pwd
/home/jake/projects/myproject

In [3]: !echo "printing from the shell"
printing from the shell
```

## 向 Shell 传递值和从 Shell 获取值

Shell 命令不仅可以从 IPython 中调用，还可以与 IPython 命名空间交互。例如，您可以使用赋值操作符`=`将任何 shell 命令的输出保存到 Python 列表中：

```py
In [4]: contents = !ls

In [5]: print(contents)
['myproject.txt']

In [6]: directory = !pwd

In [7]: print(directory)
['/Users/jakevdp/notebooks/tmp/myproject']
```

这些结果不会以列表的形式返回，而是以 IPython 中定义的特殊的 shell 返回类型返回：

```py
In [8]: type(directory)
IPython.utils.text.SList
```

这看起来和行为很像 Python 列表，但具有额外的功能，比如`grep`和`fields`方法以及允许您以方便的方式搜索、过滤和显示结果的`s`、`n`和`p`属性。有关这些信息，您可以使用 IPython 内置的帮助功能。

可以使用 `{*varname*}` 语法将 Python 变量传递到 shell 中，实现双向通信：

```py
In [9]: message = "hello from Python"

In [10]: !echo {message}
hello from Python
```

大括号中包含变量名，这个变量名在 shell 命令中会被替换为变量的内容。

## 与 Shell 相关的魔术命令

如果你在 IPython 的 shell 命令中尝试了一段时间，你可能会注意到无法使用 `!cd` 来导航文件系统：

```py
In [11]: !pwd
/home/jake/projects/myproject

In [12]: !cd ..

In [13]: !pwd
/home/jake/projects/myproject
```

这是因为笔记本中的 shell 命令是在一个临时子 shell 中执行的，这个 shell 并不保留命令之间的状态。如果你想更持久地改变工作目录，可以使用 `%cd` 魔术命令：

```py
In [14]: %cd ..
/home/jake/projects
```

实际上，默认情况下你甚至可以不带 `%` 符号使用这些功能：

```py
In [15]: cd myproject
/home/jake/projects/myproject
```

这被称为*自动魔术*函数，可以通过 `%automagic` 魔术函数切换执行这些命令时是否需要显式 `%` 符号。

除了 `%cd`，还有其他可用的类似 shell 的魔术函数，如 `%cat`、`%cp`、`%env`、`%ls`、`%man`、`%mkdir`、`%more`、`%mv`、`%pwd`、`%rm` 和 `%rmdir`，如果 `automagic` 打开，这些命令都可以不带 `%` 符号使用。这使得你几乎可以像处理普通 shell 一样处理 IPython 提示符：

```py
In [16]: mkdir tmp

In [17]: ls
myproject.txt  tmp/

In [18]: cp myproject.txt tmp/

In [19]: ls tmp
myproject.txt

In [20]: rm -r tmp
```

在同一个终端窗口中访问 shell，与你的 Python 会话结合得更自然，减少了上下文切换。
