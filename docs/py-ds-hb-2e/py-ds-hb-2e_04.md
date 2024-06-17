# 第三章：调试和性能分析

除了前一章中讨论的增强交互工具外，Jupyter 还提供了许多探索和理解正在运行的代码的方式，例如通过跟踪逻辑错误或意外的慢执行。本章将讨论其中一些工具。

# 错误和调试

代码开发和数据分析始终需要一些试验和错误，而 IPython 包含了简化此过程的工具。本节将简要介绍控制 Python 异常报告的一些选项，然后探讨代码中调试错误的工具。

## 控制异常：`%xmode`

大多数情况下，当 Python 脚本失败时，它会引发异常。当解释器遇到这些异常时，可以从 Python 内部访问导致错误的原因，该信息可以在 *traceback* 中找到。通过 `%xmode` 魔术函数，IPython 允许您控制引发异常时打印的信息量。考虑以下代码：

```py
In [1]: def func1(a, b):
            return a / b

        def func2(x):
            a = x
            b = x - 1
            return func1(a, b)

In [2]: func2(1)
ZeroDivisionError                         Traceback (most recent call last)
<ipython-input-2-b2e110f6fc8f> in <module>()
----> 1 func2(1)

<ipython-input-1-d849e34d61fb> in func2(x)
      5     a = x
      6     b = x - 1
----> 7     return func1(a, b)

<ipython-input-1-d849e34d61fb> in func1(a, b)
      1 def func1(a, b):
----> 2     return a / b
      3
      4 def func2(x):
      5     a = x

ZeroDivisionError: division by zero
```

调用 `func2` 导致错误，并阅读打印的跟踪让我们能够准确地看到发生了什么。在默认模式下，此跟踪包括几行显示导致错误的每个步骤的上下文。使用 `%xmode` 魔术函数（缩写为 *exception mode*），我们可以更改打印的信息内容。

`%xmode` 接受一个参数，即模式，有三种可能性：`Plain`、`Context` 和 `Verbose`。默认是 `Context`，给出类似刚才显示的输出。`Plain` 更紧凑，提供较少信息：

```py
In [3]: %xmode Plain
Out[3]: Exception reporting mode: Plain

In [4]: func2(1)
Traceback (most recent call last):

  File "<ipython-input-4-b2e110f6fc8f>", line 1, in <module>
    func2(1)

  File "<ipython-input-1-d849e34d61fb>", line 7, in func2
    return func1(a, b)

  File "<ipython-input-1-d849e34d61fb>", line 2, in func1
    return a / b

ZeroDivisionError: division by zero
```

`Verbose` 模式添加了一些额外信息，包括调用的任何函数的参数：

```py
In [5]: %xmode Verbose
Out[5]: Exception reporting mode: Verbose

In [6]: func2(1)
ZeroDivisionError                         Traceback (most recent call last)
<ipython-input-6-b2e110f6fc8f> in <module>()
----> 1 func2(1)
        global func2 = <function func2 at 0x103729320>

<ipython-input-1-d849e34d61fb> in func2(x=1)
      5     a = x
      6     b = x - 1
----> 7     return func1(a, b)
        global func1 = <function func1 at 0x1037294d0>
        a = 1
        b = 0

<ipython-input-1-d849e34d61fb> in func1(a=1, b=0)
      1 def func1(a, b):
----> 2     return a / b
        a = 1
        b = 0
      3
      4 def func2(x):
      5     a = x

ZeroDivisionError: division by zero
```

这些额外信息可以帮助您更准确地找出异常发生的原因。那么为什么不始终使用 `Verbose` 模式？随着代码变得复杂，这种回溯可能会变得非常长。根据情况，有时 `Plain` 或 `Context` 模式的简洁性更容易处理。

## 调试：当仅仅阅读跟踪不足以解决问题时

用于交互式调试的标准 Python 工具是 `pdb`，即 Python 调试器。此调试器允许用户逐行步进代码，以查看可能导致更复杂错误的原因。其 IPython 增强版本是 `ipdb`，即 IPython 调试器。

启动和使用这两个调试器有许多方法；我们在此处不会全面涵盖它们。请参考这两个实用工具的在线文档以了解更多信息。

在 IPython 中，也许最方便的调试接口是 `%debug` 魔术命令。如果在异常发生后调用它，它将自动在异常点打开一个交互式调试提示符。`ipdb` 提示符允许您查看堆栈的当前状态，探索可用变量，甚至运行 Python 命令！

让我们查看最近的异常，然后执行一些基本任务。我们将打印 `a` 和 `b` 的值，然后输入 `quit` 退出调试会话：

```py
In [7]: %debug <ipython-input-1-d849e34d61fb>(2)func1()
      1 def func1(a, b):
----> 2     return a / b
      3

ipdb> print(a)
1
ipdb> print(b)
0
ipdb> quit
```

交互式调试器提供的远不止这些——我们甚至可以在堆栈中上下跳转，并探索那里的变量值：

```py
In [8]: %debug <ipython-input-1-d849e34d61fb>(2)func1()
      1 def func1(a, b):
----> 2     return a / b
      3

ipdb> up <ipython-input-1-d849e34d61fb>(7)func2()
      5     a = x
      6     b = x - 1
----> 7     return func1(a, b)

ipdb> print(x)
1
ipdb> up <ipython-input-6-b2e110f6fc8f>(1)<module>()
----> 1 func2(1)

ipdb> down <ipython-input-1-d849e34d61fb>(7)func2()
      5     a = x
      6     b = x - 1
----> 7     return func1(a, b)

ipdb> quit
```

这使我们不仅可以快速找出错误的原因，还可以查看导致错误的函数调用。

如果您希望调试器在引发异常时自动启动，可以使用 `%pdb` 魔术函数来打开这种自动行为：

```py
In [9]: %xmode Plain
        %pdb on
        func2(1)
Exception reporting mode: Plain
Automatic pdb calling has been turned ON
ZeroDivisionError: division by zero <ipython-input-1-d849e34d61fb>(2)func1()
      1 def func1(a, b):
----> 2     return a / b
      3

ipdb> print(b)
0
ipdb> quit
```

最后，如果您有一个希望以交互模式从头开始运行的脚本，可以使用命令 `%run -d` 运行它，并使用 `next` 命令逐行交互式地执行代码。

除了我在这里展示的常见和有用的命令外，还有许多可用于交互式调试的命令。表 3-1 包含了一些更多的描述。

表 3-1\. 调试命令的部分列表

| 命令 | 描述 |
| --- | --- |
| `l(ist)` | 显示文件中的当前位置 |
| `h(elp)` | 显示命令列表，或查找特定命令的帮助信息 |
| `q(uit)` | 退出调试器和程序 |
| `c(ontinue)` | 退出调试器，继续程序执行 |
| `n(ext)` | 进入程序的下一步 |
| `<enter>` | 重复上一个命令 |
| `p(rint)` | 打印变量 |
| `s(tep)` | 进入子例程 |
| `r(eturn)` | 从子例程中返回 |

欲了解更多信息，请在调试器中使用 `help` 命令，或查看 `ipdb` 的[在线文档](https://oreil.ly/TVSAT)。

# 代码分析和计时

在开发代码和创建数据处理流水线的过程中，通常可以在各种实现之间做出权衡。在开发算法的早期阶段，过于担心这些事情可能会适得其反。正如唐纳德·克努特所说，“我们应该忘记小的效率，大约 97%的时间：过早优化是一切邪恶的根源。”

但是一旦您的代码运行正常，深入了解其效率可能会有所帮助。有时检查特定命令或一系列命令的执行时间很有用；其他时候，检查多行过程并确定复杂操作系列中的瓶颈位置也很有用。IPython 提供了广泛的功能来进行此类代码的时间和性能分析。这里我们将讨论以下 IPython 魔术命令：

`%time`

时间单个语句的执行时间

`%timeit`

多次执行单个语句以获得更准确的时间

`%prun`

使用分析器运行代码

`%lprun`

使用逐行分析器运行代码

`%memit`

测量单个语句的内存使用情况

`%mprun`

使用逐行内存分析器运行代码

最后四个命令未捆绑在 IPython 中；要使用它们，您需要获取`line_profiler`和`memory_profiler`扩展，我们将在以下部分讨论它们。

## 计时代码片段：`%timeit`和`%time`

我们在“IPython 魔法命令”的介绍中看到了`%timeit`行魔法和`%%timeit`单元魔法；这些可以用来计时代码片段的重复执行。

```py
In [1]: %timeit sum(range(100))
1.53 µs ± 47.8 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)
```

请注意，由于此操作非常快，`%timeit`会自动执行大量重复。对于较慢的命令，`%timeit`会自动调整并执行较少的重复：

```py
In [2]: %%timeit
        total = 0
        for i in range(1000):
            for j in range(1000):
                total += i * (-1) ** j
536 ms ± 15.9 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```

有时重复操作并不是最佳选择。例如，如果我们有一个需要排序的列表，我们可能会被重复的操作误导；排序一个已经排序好的列表比排序一个未排序的列表快得多，因此重复操作会扭曲结果：

```py
In [3]: import random
        L = [random.random() for i in range(100000)]
        %timeit L.sort()
Out[3]: 1.71 ms ± 334 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
```

对于此操作，`%time`魔法函数可能是一个更好的选择。当命令运行时间较长时，短暂的系统相关延迟不太可能影响结果。让我们计时一个未排序和一个已排序列表的排序过程：

```py
In [4]: import random
        L = [random.random() for i in range(100000)]
        print("sorting an unsorted list:")
        %time L.sort()
Out[4]: sorting an unsorted list:
        CPU times: user 31.3 ms, sys: 686 µs, total: 32 ms
        Wall time: 33.3 ms
```

```py
In [5]: print("sorting an already sorted list:")
        %time L.sort()
Out[5]: sorting an already sorted list:
        CPU times: user 5.19 ms, sys: 268 µs, total: 5.46 ms
        Wall time: 14.1 ms
```

注意排序好的列表比无序列表排序快得多，但请注意，即使对于排序好的列表，使用`%time`比`%timeit`花费的时间也更长！这是因为`%timeit`在幕后做了一些聪明的事情，防止系统调用干扰计时。例如，它阻止了未使用的 Python 对象的清理（称为*垃圾收集*），否则可能会影响计时。因此，`%timeit`的结果通常明显比`%time`的结果快。

对于`%time`，与`%timeit`一样，使用`%%`单元魔法语法允许对多行脚本进行计时：

```py
In [6]: %%time
        total = 0
        for i in range(1000):
            for j in range(1000):
                total += i * (-1) ** j
CPU times: user 655 ms, sys: 5.68 ms, total: 661 ms
Wall time: 710 ms
```

要获取有关`%time`和`%timeit`以及它们可用选项的更多信息，请使用 IPython 帮助功能（例如，在 IPython 提示符处键入`%time?`）。

## **全脚本分析**: %prun

一个程序由许多单个语句组成，有时在上下文中计时这些语句比单独计时它们更重要。Python 包含一个内置的代码分析器（可以在 Python 文档中了解），但 IPython 提供了一个更方便的方式来使用此分析器，即魔法函数`%prun`。

举例来说，我们定义一个简单的函数来进行一些计算：

```py
In [7]: def sum_of_lists(N):
            total = 0
            for i in range(5):
                L = [j ^ (j >> i) for j in range(N)]
                total += sum(L)
            return total
```

现在我们可以调用一个函数调用来查看`%prun`的分析结果：

```py
In [8]: %prun sum_of_lists(1000000)
14 function calls in 0.932 seconds
Ordered by: internal time
ncalls  tottime  percall  cumtime  percall filename:lineno(function)
     5    0.808    0.162    0.808  0.162 <ipython-input-7-f105717832a2>:4(<listcomp>)
     5    0.066    0.013    0.066  0.013 {built-in method builtins.sum}
     1    0.044    0.044    0.918  0.918 <ipython-input-7-f105717832a2>:1
     > (sum_of_lists)
     1    0.014    0.014    0.932  0.932 <string>:1(<module>)
     1    0.000    0.000    0.932  0.932 {built-in method builtins.exec}
     1    0.000    0.000    0.000  0.000 {method 'disable' of '_lsprof.Profiler'
     > objects}
```

结果是一个表，按每次函数调用的总时间顺序显示执行在哪里花费了最多的时间。在这种情况下，大部分执行时间都花费在`sum_of_lists`内的列表推导式中。从这里开始，我们可以考虑如何改进算法的性能。

要获取有关`%prun`以及其可用选项的更多信息，请使用 IPython 帮助功能（即，在 IPython 提示符处键入`%prun?`）。

## 逐行分析：`%lprun`

`%prun`的函数级性能分析很有用，但有时逐行分析报告更方便。这不是 Python 或 IPython 内置的功能，但可以通过安装`line_profiler`包来实现这一点。首先使用 Python 的包装工具`pip`安装`line_profiler`包：

```py
$ pip install line_profiler

```

接下来，您可以使用 IPython 来加载`line_profiler` IPython 扩展，这是该软件包的一部分：

```py
In [9]: %load_ext line_profiler
```

现在，`%lprun`命令将对任何函数进行逐行分析。在这种情况下，我们需要明确告诉它我们感兴趣的是哪些函数：

```py
In [10]: %lprun -f sum_of_lists sum_of_lists(5000)
Timer unit: 1e-06 s

Total time: 0.014803 s
File: <ipython-input-7-f105717832a2>
Function: sum_of_lists at line 1

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
      1                                           def sum_of_lists(N):
      2         1          6.0      6.0      0.0      total = 0
      3         6         13.0      2.2      0.1      for i in range(5):
      4         5      14242.0   2848.4     96.2          L = [j ^ (j >> i) for j
      5         5        541.0    108.2      3.7          total += sum(L)
      6         1          1.0      1.0      0.0      return total
```

顶部的信息为我们提供了阅读结果的关键：时间以微秒为单位报告，我们可以看到程序在哪里花费了最多的时间。此时，我们可以使用这些信息来修改脚本的某些方面，使其在我们所需的用例中表现更好。

要获取关于`%lprun`及其可用选项的更多信息，请使用 IPython 的帮助功能（例如，在 IPython 提示符处键入`%lprun?`）。

## 内存使用分析：`%memit`和`%mprun`

分析的另一个方面是操作使用的内存量。可以使用另一个 IPython 扩展`memory_profiler`来评估这一点。与`line_profiler`一样，我们首先使用`pip`安装该扩展：

```py
$ pip install memory_profiler

```

然后我们可以使用 IPython 来加载它：

```py
In [11]: %load_ext memory_profiler
```

内存分析扩展包含两个有用的魔术函数：`%memit`（提供了`%timeit`的内存测量等价物）和`%mprun`（提供了`%lprun`的内存测量等价物）。`%memit`魔术函数可以非常简单地使用：

```py
In [12]: %memit sum_of_lists(1000000)
peak memory: 141.70 MiB, increment: 75.65 MiB
```

我们看到这个函数使用了约 140 MB 的内存。

对于逐行描述内存使用情况，我们可以使用`%mprun`魔术函数。不幸的是，这仅适用于在单独模块中定义的函数，而不适用于笔记本本身，因此我们将首先使用`%%file`单元格魔术创建一个简单的模块`mprun_demo.py`，其中包含我们的`sum_of_lists`函数，并添加一个额外的功能，以使我们的内存分析结果更清晰：

```py
In [13]: %%file mprun_demo.py
         def sum_of_lists(N):
             total = 0
             for i in range(5):
                 L = [j ^ (j >> i) for j in range(N)]
                 total += sum(L)
                 del L # remove reference to L
             return total
Overwriting mprun_demo.py
```

现在我们可以导入这个函数的新版本并运行内存行分析：

```py
In [14]: from mprun_demo import sum_of_lists
         %mprun -f sum_of_lists sum_of_lists(1000000)

Filename: /Users/jakevdp/github/jakevdp/PythonDataScienceHandbook/notebooks_v2/
> m prun_demo.py

Line #    Mem usage    Increment  Occurrences   Line Contents
============================================================
     1     66.7 MiB     66.7 MiB           1   def sum_of_lists(N):
     2     66.7 MiB      0.0 MiB           1       total = 0
     3     75.1 MiB      8.4 MiB           6       for i in range(5):
     4    105.9 MiB     30.8 MiB     5000015           L = [j ^ (j >> i) for j
     5    109.8 MiB      3.8 MiB           5           total += sum(L)
     6     75.1 MiB    -34.6 MiB           5           del L # remove reference to L
     7     66.9 MiB     -8.2 MiB           1       return total
```

在这里，`Increment`列告诉我们每行如何影响总内存预算：注意，当我们创建和删除列表`L`时，我们增加了约 30 MB 的内存使用量。这是在 Python 解释器本身的后台内存使用量之上的额外内存使用。

要获取关于`%memit`和`%mprun`及其可用选项的更多信息，请使用 IPython 的帮助功能（例如，在 IPython 提示符处键入`%memit?`）。

# 更多 IPython 资源

在这一系列章节中，我们只是初步介绍了使用 IPython 来支持数据科学任务的表面知识。更多信息可以在印刷品和网络上找到，这里我将列出一些其他可能对你有帮助的资源。

## 网络资源

[IPython 网站](http://ipython.org)

IPython 网站提供了文档、示例、教程以及各种其他资源的链接。

[nbviewer 网站](http://nbviewer.jupyter.org)

此网站显示任何可在互联网上找到的 Jupyter 笔记本的静态渲染。首页展示了一些示例笔记本，您可以浏览看看其他人是如何使用 IPython 的！

[Jupyter 笔记本的精选集](https://github.com/jupyter/jupyter/wiki)

由 nbviewer 提供支持的这个不断增长的笔记本列表展示了您可以用 IPython 进行的数值分析的深度和广度。它涵盖了从简短的示例和教程到完整的课程和笔记本格式的书籍！

视频教程

在互联网上搜索，您会发现许多关于 IPython 的视频教程。我特别推荐从 PyCon、SciPy 和 PyData 大会上获取教程，由 IPython 和 Jupyter 的两位主要创作者和维护者 Fernando Perez 和 Brian Granger 提供。

## 书籍

[*Python 数据分析*（O’Reilly）](https://oreil.ly/ik2g7)

Wes McKinney 的书包括一章介绍如何将 IPython 用作数据科学家的工具。尽管其中很多内容与我们在这里讨论的内容重叠，但另一个视角总是有帮助的。

*学习 IPython 进行交互计算与数据可视化*（Packt）

Cyrille Rossant 编写的这本简短书籍，为使用 IPython 进行数据分析提供了良好的入门。

*IPython 交互计算与可视化食谱*（Packt）

同样由 Cyrille Rossant 编写，这本书是更长、更高级的 IPython 数据科学使用指南。尽管书名中提到的是 IPython，但它也深入探讨了广泛的数据科学主题。

最后，提醒您可以自己寻找帮助：IPython 基于 `?` 的帮助功能（详见第一章）在您善加利用且经常使用时非常有用。当您浏览这里和其他地方的示例时，可以用它来熟悉 IPython 提供的所有工具。
