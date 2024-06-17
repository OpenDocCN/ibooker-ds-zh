# 第一章：在 IPython 和 Jupyter 中开始

在编写数据科学的 Python 代码时，我通常会在三种工作模式之间切换：我使用 IPython shell 尝试短命令序列，使用 Jupyter Notebook 进行更长时间的交互分析和与他人共享内容，并使用诸如 Emacs 或 VSCode 的交互式开发环境（IDE）创建可重复使用的 Python 包。本章重点介绍前两种模式：IPython shell 和 Jupyter Notebook。虽然软件开发中使用 IDE 是数据科学家工具箱中的重要第三工具，但我们在此不会直接讨论它。

# 启动 IPython Shell

此书大部分的文本，包括本部分，都不是设计用于被被动吸收的。我建议您在阅读时跟随并尝试所涵盖的工具和语法：通过这样做建立的肌肉记忆将比仅仅阅读要有用得多。首先通过在命令行上键入 **`ipython`** 来启动 IPython 解释器；或者，如果您安装了像 Anaconda 或 EPD 这样的发行版，可能会有一个特定于您系统的启动器。

一旦完成此操作，您应该看到如下提示：

```py
Python 3.9.2 (v3.9.2:1a79785e3e, Feb 19 2021, 09:06:10)
Type 'copyright', 'credits' or 'license' for more information
IPython 7.21.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]:
```

准备好了，现在可以跟着进行。

# 启动 Jupyter Notebook

Jupyter Notebook 是一个基于浏览器的图形界面，用于 IPython shell，并在此基础上提供了丰富的动态显示功能。除了执行 Python/IPython 语句外，笔记本还允许用户包括格式化文本、静态和动态可视化、数学方程、JavaScript 小部件等。此外，这些文档可以以一种方式保存，让其他人能够打开它们并在自己的系统上执行代码。

尽管您将通过 Web 浏览器窗口查看和编辑 Jupyter 笔记本，但它们必须连接到正在运行的 Python 进程以执行代码。您可以通过在系统 shell 中运行以下命令来启动此进程（称为“内核”）：

```py
$ jupyter lab

```

此命令启动一个本地 Web 服务器，该服务器将对您的浏览器可见。它立即输出一个显示其正在执行的日志；该日志看起来会像这样：

```py
$ jupyter lab
[ServerApp] Serving notebooks from local directory: /Users/jakevdp/ \
PythonDataScienceHandbook
[ServerApp] Jupyter Server 1.4.1 is running at:
[ServerApp] http://localhost:8888/lab?token=dd852649
[ServerApp] Use Control-C to stop this server and shut down all kernels
(twice to skip confirmation).

```

执行命令后，默认浏览器应自动打开并导航到列出的本地 URL；确切的地址将取决于您的系统。如果浏览器未自动打开，您可以手动打开一个窗口并输入此地址（*http://localhost:8888/lab/* 作为示例）。

# IPython 中的帮助和文档

如果您在本章中不阅读其他部分，请阅读此部分：我发现在我的日常工作流程中，讨论的工具是 IPython 最具变革性的贡献。

当技术上熟悉的人被要求帮助朋友、家人或同事解决计算机问题时，大多数时候这不是知道答案的问题，而是知道如何快速找到未知答案的问题。在数据科学中也是如此：可搜索的网络资源，如在线文档、邮件列表线程和 Stack Overflow 答案，包含了丰富的信息，甚至（尤其是？）关于您之前搜索过的主题。成为数据科学的有效从业者，不仅仅是记住每种可能情况下应该使用的工具或命令，更重要的是学会如何有效地查找您不知道的信息，无论是通过网络搜索引擎还是其他方式。

IPython/Jupyter 最有用的功能之一是缩小用户与文档之间的差距，帮助用户有效地完成工作。尽管网络搜索在回答复杂问题方面仍然发挥着作用，但仅通过 IPython 就能找到大量信息。IPython 可以在几个按键中帮助回答的问题的一些示例包括：

+   我如何调用这个函数？它有哪些参数和选项？

+   Python 对象的源代码是什么样子的？

+   我导入的这个包里面有什么？

+   这个对象有哪些属性或方法？

在 IPython shell 和 Jupyter Notebook 中提供的工具可以快速访问这些信息，主要包括使用 `?` 字符查看文档、使用 `??` 字符查看源代码，以及使用 Tab 键进行自动完成。

## 使用 `?` 访问文档

Python 语言及其数据科学生态系统是为用户设计的，其中一个重要部分是访问文档。每个 Python 对象都包含对称为 *docstring* 的字符串的引用，大多数情况下将包含对象的简明摘要及其使用方法。Python 有一个内置的 `help` 函数，可以访问这些信息并打印结果。例如，要查看内置 `len` 函数的文档，请执行以下操作：

```py
In [1]: help(len)
Help on built-in function len in module builtins:

len(obj, /)
    Return the number of items in a container.
```

根据您的解释器不同，此信息可能会显示为内联文本或在单独的弹出窗口中。

因为查找对象帮助是如此常见和有用，IPython 和 Jupyter 引入了 `?` 字符作为访问此文档和其他相关信息的简写：

```py
In [2]: len?
Signature: len(obj, /)
Docstring: Return the number of items in a container.
Type:      builtin_function_or_method
```

此表示法适用于几乎任何内容，包括对象方法：

```py
In [3]: L = [1, 2, 3]
In [4]: L.insert?
Signature: L.insert(index, object, /)
Docstring: Insert object before index.
Type:      builtin_function_or_method
```

或者甚至是对象本身，具有其类型的文档：

```py
In [5]: L?
Type:        list
String form: [1, 2, 3]
Length:      3
Docstring:
Built-in mutable sequence.

If no argument is given, the constructor creates a new empty list.
The argument must be an iterable if specified.
```

更重要的是，即使是您自己创建的函数或其他对象，这也同样适用！在这里，我们将定义一个带有文档字符串的小函数：

```py
In [6]: def square(a):
  ....:     """Return the square of a."""
  ....:     return a ** 2
  ....:
```

请注意，为我们的函数创建文档字符串时，我们只需将字符串字面值放在第一行。由于文档字符串通常是多行的，按照惯例，我们使用了 Python 的三引号符号来表示多行字符串。

现在我们将使用 `?` 来找到这个文档字符串：

```py
In [7]: square?
Signature: square(a)
Docstring: Return the square of a.
File:      <ipython-input-6>
Type:      function
```

通过 docstrings 快速访问文档是你应该养成的习惯之一，这是你应该始终将这样的内联文档添加到你编写的代码中的原因之一。

## 使用 ?? 访问源代码

因为 Python 语言非常易读，通过阅读你感兴趣的对象的源代码通常可以获得更深入的见解。IPython 和 Jupyter 通过双问号 (`??`) 提供了直接查看源代码的快捷方式：

```py
In [8]: square??
Signature: square(a)
Source:
def square(a):
    """Return the square of a."""
    return a ** 2
File:      <ipython-input-6>
Type:      function
```

对于像这样的简单函数，双问号可以快速了解其底层细节。

如果你经常使用这个功能，你会注意到有时 `??` 后缀不会显示任何源代码：这通常是因为所讨论的对象不是用 Python 实现的，而是用 C 或其他编译扩展语言实现的。如果是这种情况，`??` 后缀会给出与 `?` 后缀相同的输出。你会在许多 Python 内置对象和类型中特别发现这种情况，包括前面提到的 `len` 函数：

```py
In [9]: len??
Signature: len(obj, /)
Docstring: Return the number of items in a container.
Type:      builtin_function_or_method
```

使用 `?` 和/或 `??` 是查找任何 Python 函数或模块功能的强大快速方式。

## 使用 Tab 补全探索模块

另一个有用的界面是使用 Tab 键自动完成和探索对象、模块和命名空间的内容。在接下来的示例中，我将使用 `<TAB>` 来指示何时按 Tab 键。

### 对象内容的 Tab 补全

每个 Python 对象都有与之关联的各种属性和方法。与前面提到的 `help` 函数类似，Python 有一个内置的 `dir` 函数，返回这些属性和方法的列表，但实际使用中，Tab 补全接口更容易使用。要查看对象的所有可用属性列表，可以输入对象名称，后跟句点 (`.`) 字符和 Tab 键：

```py
In [10]: L.<TAB>
            append() count    insert   reverse
            clear    extend   pop      sort
            copy     index    remove
```

要缩小列表，可以输入名称的第一个或几个字符，然后按 Tab 键查找匹配的属性和方法：

```py
In [10]: L.c<TAB>
             clear() count()
             copy()

In [10]: L.co<TAB>
              copy()  count()
```

如果只有一个选项，按 Tab 键将为您完成该行。例如，以下内容将立即替换为 `L.count`：

```py
In [10]: L.cou<TAB>
```

尽管 Python 没有严格强制区分公共/外部属性和私有/内部属性的区别，按照惯例，前置下划线用于表示后者。为了清晰起见，默认情况下省略了这些私有方法和特殊方法，但可以通过显式输入下划线来列出它们：

```py
In [10]: L._<TAB>
           __add__             __delattr__     __eq__
           __class__           __delitem__     __format__()
           __class_getitem__() __dir__()       __ge__            >
           __contains__        __doc__         __getattribute__
```

为简洁起见，我只显示了输出的前几列。大多数都是 Python 的特殊双下划线方法（通常被昵称为“dunder”方法）。

### 导入时的 Tab 补全

在从包中导入对象时，Tab 补全也非常有用。在这里，我们将用它来查找以 `co` 开头的 `itertools` 包中的所有可能导入：

```py
In [10]: from itertools import co<TAB>
         combinations()                  compress()
         combinations_with_replacement() count()
```

同样，您可以使用 tab 补全来查看系统上可用的导入（这将根据哪些第三方脚本和模块对您的 Python 会话可见而变化）：

```py
In [10]: import <TAB>
            abc                 anyio
            activate_this       appdirs
            aifc                appnope        >
            antigravity         argon2

In [10]: import h<TAB>
            hashlib html
            heapq   http
            hmac
```

### 超出 tab 补全：通配符匹配

如果您知道对象或属性名称的前几个字符，tab 补全是有用的，但如果您想要匹配名称中间或末尾的字符，则帮助不大。对于这种用例，IPython 和 Jupyter 提供了使用`*`字符进行名称通配符匹配的方法。

例如，我们可以使用这个来列出命名空间中名称以`Warning`结尾的每个对象：

```py
In [10]: *Warning?
BytesWarning                  RuntimeWarning
DeprecationWarning            SyntaxWarning
FutureWarning                 UnicodeWarning
ImportWarning                 UserWarning
PendingDeprecationWarning     Warning
ResourceWarning
```

注意，`*` 字符匹配任何字符串，包括空字符串。

同样，假设我们正在寻找一个字符串方法，其中包含单词`find`在其名称中的某处。我们可以这样搜索：

```py
In [11]: str.*find*?
str.find
str.rfind
```

我发现这种灵活的通配符搜索可以帮助我在了解新包或重新熟悉熟悉的包时找到特定的命令很有用。

# IPython Shell 中的键盘快捷键

如果您在计算机上花费了任何时间，您可能已经发现在工作流程中使用键盘快捷键的用途。最熟悉的可能是 Cmd-c 和 Cmd-v（或 Ctrl-c 和 Ctrl-v），用于在各种程序和系统中复制和粘贴。高级用户往往会走得更远：流行的文本编辑器如 Emacs、Vim 和其他编辑器通过复杂的按键组合为用户提供了一系列不可思议的操作。

IPython shell 不会走这么远，但在输入命令时提供了许多快速导航的键盘快捷键。虽然其中一些快捷键确实在基于浏览器的笔记本中起作用，但本节主要讨论 IPython shell 中的快捷键。

一旦您习惯了这些，它们可以非常有用，可以快速执行某些命令，而无需将手从“home”键盘位置移开。如果您是 Emacs 用户或者有 Linux 风格 shell 的使用经验，则以下内容将非常熟悉。我将这些快捷键分为几个类别：*导航快捷键*、*文本输入快捷键*、*命令历史快捷键*和*其他快捷键*。

## 导航快捷键

虽然使用左右箭头键在行内向前向后移动是很明显的，但还有其他选项不需要移动手从键盘的“home”位置：

| 按键 | 动作 |
| --- | --- |
| Ctrl-a | 将光标移动到行首 |
| Ctrl-e | 将光标移动到行尾 |
| Ctrl-b 或左箭头键 | 将光标向后移动一个字符 |
| Ctrl-f 或右箭头键 | 将光标向前移动一个字符 |

## 文本输入快捷键

虽然每个人都习惯使用退格键删除前一个字符，但经常需要稍微扭动手指才能按到该键，而且它一次只能删除一个字符。在 IPython 中，有几个快捷键可用于删除您正在输入的文本的某些部分；其中最有用的是删除整行文本的命令。如果你发现自己使用 Ctrl-b 和 Ctrl-d 的组合来删除前一个字符，而不是按退格键，那么你将会知道这些快捷键已经变得本能了！

| 按键 | 动作 |
| --- | --- |
| 退格键 | 删除行内前一个字符 |
| Ctrl-d | 删除行内下一个字符 |
| Ctrl-k | 从光标处剪切文本到行尾 |
| Ctrl-u | 从行首剪切文本到光标处 |
| Ctrl-y | 拷贝（即粘贴）之前被剪切的文本 |
| Ctrl-t | 转置（即交换）前两个字符 |

## 命令历史快捷键

或许在这里讨论的最有影响力的快捷键是 IPython 提供的用于导航命令历史记录的快捷键。此命令历史记录超出了您当前的 IPython 会话：您的整个命令历史记录存储在 IPython 配置文件目录中的 SQLite 数据库中。

访问先前的命令最简单的方法是使用上箭头和下箭头键来浏览历史记录，但还有其他选项：

| 按键 | 动作 |
| --- | --- |
| Ctrl-p（或向上箭头键） | 访问历史记录中的上一个命令 |
| Ctrl-n（或向下箭头键） | 访问历史记录中的下一个命令 |
| Ctrl-r | 通过命令历史记录进行逆向搜索 |

逆向搜索选项可能特别有用。回想一下，早些时候我们定义了一个名为 `square` 的函数。让我们从一个新的 IPython shell 中反向搜索我们的 Python 历史记录，并再次找到这个定义。当您在 IPython 终端中按 Ctrl-r 时，您将看到以下提示：

```py
In [1]:
(reverse-i-search)`':
```

如果您在此提示符下开始键入字符，IPython 将自动填充最近的命令（如果有），与这些字符匹配的：

```py
In [1]:
(reverse-i-search)`sqa': square??
```

在任何时候，您都可以添加更多字符以完善搜索，或者再次按 Ctrl-r 以进一步搜索与查询匹配的另一个命令。如果您之前跟随操作，再按两次 Ctrl-r 会得到以下结果：

```py
In [1]:
(reverse-i-search)`sqa': def square(a):
    """Return the square of a"""
    return a ** 2
```

找到您要查找的命令后，按回车键搜索将结束。然后，您可以使用检索到的命令并继续会话：

```py
In [1]: def square(a):
    """Return the square of a"""
    return a ** 2

In [2]: square(2)
Out[2]: 4
```

请注意，您可以使用 Ctrl-p/Ctrl-n 或上/下箭头键以类似的方式搜索历史记录，但只能通过匹配行开头的字符来搜索。也就是说，如果您键入 **`def`** 然后按 Ctrl-p，它将找到您历史记录中以字符 `def` 开头的最近的命令（如果有的话）。

## 杂项快捷键

最后，还有一些杂项快捷键不属于前面提到的任何类别，但仍然值得知道：

| 按键 | 动作 |
| --- | --- |
| Ctrl-l | 清除终端屏幕 |
| Ctrl-c | 中断当前 Python 命令 |
| Ctrl-d | 退出 IPython 会话 |

特别是 Ctrl-c 快捷键在你不小心启动了一个运行时间非常长的任务时非常有用。

虽然这里讨论的一些快捷键起初可能显得有点晦涩，但是通过实践，它们很快就会变得自动化。一旦你培养了那种肌肉记忆，我相信你甚至会希望它们在其他情境中也能用到。
