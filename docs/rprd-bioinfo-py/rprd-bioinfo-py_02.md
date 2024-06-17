# 第一章。四核苷酸频率：计数事物

在生物信息学中，计算DNA中的碱基可能是“Hello, World!”。[Rosalind DNA挑战](https://oreil.ly/maR31)描述了一个程序，它将获取一段DNA序列并打印出发现的*A*、*C*、*G*和*T*的计数。在Python中计数事物有很多令人惊讶的方式，我将探索这种语言提供的内容。我还将演示如何编写结构良好、有文档化的程序，验证其参数以及编写和运行测试以确保程序正常工作。

在本章中，您将学到：

+   如何使用`new.py`开始一个新程序

+   如何定义和验证命令行参数使用`argparse`

+   如何使用`pytest`运行测试套件

+   如何迭代字符串的字符

+   计算集合中元素的方法

+   如何使用`if`/`elif`语句创建决策树

+   如何格式化字符串

# 开始使用

在开始之前，请确保您已阅读了[“获取代码和测试”](preface01.html#gettingCodeandTests)部分。一旦您有了代码仓库的本地副本，请切换到*01_dna*目录：

```py
$ cd 01_dna
```

这里您会找到几个`solution*.py`程序以及您可以用来查看程序是否正确工作的测试和输入数据。要了解您的程序应如何工作的概念，请从第一个解决方案复制到一个名为`dna.py`的程序：

```py
$ cp solution1_iter.py dna.py
```

现在以无参数或使用`-h`或`--help`标志运行程序。它将打印使用文档（注意*usage*是输出的第一个词）：

```py
$ ./dna.py
usage: dna.py [-h] DNA
dna.py: error: the following arguments are required: DNA
```

如果您遇到“权限被拒绝”的错误，请尝试运行**`chmod +x dna.py`**以添加可执行权限。

这是复制性的首要元素之一。*程序应提供关于其运行方式的文档。*虽然通常有类似于*README*文件或甚至是论文来描述一个程序，但程序本身必须提供关于其参数和输出的文档。我将向您展示如何使用`argparse`模块定义和验证参数，并生成文档，这意味着由程序生成的使用说明不可能是错误的。与*README*文件和更改日志等可能很快与程序开发脱节的情况形成对比，希望您能欣赏到这种文档的效果。

您可以从使用行看出，程序期望类似`DNA`的参数，因此让我们给它一个序列。正如在Rosalind页面上描述的那样，该程序按照顺序和用单个空格分隔的方式打印每个碱基*A*、*C*、*G*和*T*的计数：

```py
$ ./dna.py ACCGGGTTTT
1 2 3 4
```

当你前往解决**Rosalind.info**网站上的挑战时，程序的输入将作为下载文件提供；因此，我将编写程序以便它也能读取文件内容。我可以使用**`cat`**（*concatenate*的缩写）命令来打印*tests/inputs*目录中一个文件的内容。

```py
$ cat tests/inputs/input2.txt
AGCTTTTCATTCTGACTGCAACGGGCAATATGTCTCTGTGTGGATTAAAAAAAGAGTGTCTGATAGCAGC
```

这是在网站示例中显示的相同序列。因此，我知道程序的输出应该是这样的：

```py
$ ./dna.py tests/inputs/input2.txt
20 12 17 21
```

在整本书中，我将使用`pytest`工具来运行确保程序按预期工作的测试。当我运行**`pytest`**命令时，它将递归搜索当前目录以寻找看起来像测试的测试和函数。请注意，如果你在Windows上，你可能需要运行**`python3 -m pytest`**或**`pytest.exe`**。现在运行它，你应该会看到类似以下的东西，表明程序通过了*tests/dna_test.py*文件中的所有四个测试：

```py
$ pytest
=========================== test session starts ===========================
...
collected 4 items

tests/dna_test.py ....                                              [100%]

============================ 4 passed in 0.41s ============================
```

软件测试的关键要素是*使用已知的输入运行程序并验证它是否产生正确的输出*。尽管这似乎是一个显而易见的想法，但我曾经反对仅仅运行程序而不验证其正确行为的“测试”方案。

## 使用`new.py`创建程序

如果你复制了前面部分显示的解决方案之一，那么删除该程序，以便你可以从头开始：

```py
$ rm dna.py
```

在查看我的解决方案之前，请尝试解决这个问题。如果你认为你已经获得了所有需要的信息，请随意提前编写你自己版本的`dna.py`，使用`pytest`来运行提供的测试。如果你想一步步地与我学习如何编写程序和运行测试，请继续阅读。

本书中的每个程序都将接受一些命令行参数并创建一些输出，如命令行上的文本或新文件。我总是使用前言中描述的`new.py`程序来启动，但这不是必需的。你可以按照自己的喜好编写程序，从任何你想要的地方开始，但是你的程序应该具有相同的特性，如生成使用说明和正确验证参数。

在*01_dna*目录中创建你的`dna.py`程序，因为这包含程序的测试文件。这是我如何启动`dna.py`程序的方式。`--purpose`参数将用于程序的文档：

```py
$ new.py --purpose 'Tetranucleotide frequency' dna.py
Done, see new script "dna.py."
```

如果你运行新的`dna.py`程序，你会看到它定义了许多与命令行程序常见的不同类型的参数：

```py
$ ./dna.py --help
usage: dna.py [-h] [-a str] [-i int] [-f FILE] [-o] str

Tetranucleotide frequency ![1](assets/1.png)

positional arguments:
  str                   A positional argument ![2](assets/2.png)

optional arguments:
  -h, --help            show this help message and exit ![3](assets/3.png)
  -a str, --arg str     A named string argument (default: ) ![4](assets/4.png)
  -i int, --int int     A named integer argument (default: 0) ![5](assets/5.png)
  -f FILE, --file FILE  A readable file (default: None) ![6](assets/6.png)
  -o, --on              A boolean flag (default: False) ![7](assets/7.png)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO1-1)

这里使用`new.py`的`--purpose`来描述程序。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO1-2)

该程序接受一个单一的位置字符串参数。

[![3](assets/3.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO1-3)

`-h` 和 `--help` 标志是由 `argparse` 自动添加的，将触发使用说明。

[![4](assets/4.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO1-4)

这是一个命名选项，具有短 (`-a`) 和长 (`--arg`) 名称，用于字符串值。

[![5](assets/5.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO1-5)

这是一个命名选项，具有短 (`-i`) 和长 (`--int`) 名称，用于整数值。

[![6](assets/6.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO1-6)

这是一个命名选项，具有短 (`-f`) 和长 (`--file`) 名称，用于文件参数。

[![7](assets/7.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO1-7)

这是一个布尔标志，当 `-o` 或 `--on` 存在时将为 `True`，当它们不存在时为 `False`。

这个程序只需要 `str` 位置参数，并且你可以使用 `DNA` 作为 `metavar` 值，以便向用户指示参数的含义。删除所有其他参数。请注意，永远不要定义 `-h` 和 `--help` 标志，因为 `argparse` 在内部使用它们来响应使用请求。看看你是否可以修改你的程序，直到它生成以下的使用情况（如果你暂时无法生成使用情况，请不要担心，我将在下一节中展示这个）：

```py
$ ./dna.py -h
usage: dna.py [-h] DNA

Tetranucleotide frequency

positional arguments:
  DNA         Input DNA sequence

optional arguments:
  -h, --help  show this help message and exit
```

如果你能够使其正常工作，我想指出这个程序将只接受一个位置参数。如果尝试使用其他数量的参数运行它，程序将立即停止并打印错误消息：

```py
$ ./dna.py AACC GGTT
usage: dna.py [-h] DNA
dna.py: error: unrecognized arguments: GGTT
```

同样，该程序将拒绝任何未知的标志或选项。只需几行代码，你就建立了一个文档良好的程序，用于验证程序的参数。这是实现可重现性的一个非常基本且重要的步骤。

## 使用 argparse

`new.py` 创建的程序使用 `argparse` 模块来定义程序的参数，验证参数的正确性，并为用户创建使用文档。`argparse` 模块是 Python 的*标准*模块，这意味着它总是存在的。其他模块也可以执行这些操作，你可以自由选择任何方法来处理程序的这个方面。只需确保你的程序能够通过测试。

我为 *Tiny Python Projects* 写了一个 `new.py` 的版本，你可以在[该书的 GitHub 仓库的 *bin* 目录](https://oreil.ly/7romb)找到。那个版本比我要求你使用的版本要简单一些。我将首先向你展示使用这个早期版本 `new.py` 创建的 `dna.py` 的一个版本：

```py
#!/usr/bin/env python3 ![1](assets/1.png)
""" Tetranucleotide frequency """ ![2](assets/2.png)

import argparse ![3](assets/3.png)

# --------------------------------------------------
def get_args(): ![4](assets/4.png)
    """ Get command-line arguments """ ![5](assets/5.png)

    parser = argparse.ArgumentParser( ![6](assets/6.png)
        description='Tetranucleotide frequency',
        formatter_class=argparse.ArgumentDefaultsHelpFormatter)

    parser.add_argument('dna', metavar='DNA', help='Input DNA sequence') ![7](assets/7.png)

    return parser.parse_args() ![8](assets/8.png)

# --------------------------------------------------
def main(): ![9](assets/9.png)
    """ Make a jazz noise here """

    args = get_args() ![10](assets/10.png)
    print(args.dna)   ![11](assets/11.png)

# --------------------------------------------------
if __name__ == '__main__': ![12](assets/12.png)
    main()
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO2-1)

俗称的*shebang*（`#!`）告诉操作系统使用`env`命令（*environment*）来找到`python3`以执行程序的其余部分。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO2-2)

这是整个程序或模块的*文档字符串*（documentation string）。

[![3](assets/3.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO2-3)

我导入`argparse`模块来处理命令行参数。

[![4](assets/4.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO2-4)

我总是定义一个`get_args()`函数来处理`argparse`代码。

[![5](assets/5.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO2-5)

这是一个函数的文档字符串。

[![6](assets/6.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO2-6)

`parser`对象用于定义程序的参数。

[![7](assets/7.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO2-7)

我定义了一个`dna`参数，它将是位置参数，因为名称`dna`*不*以破折号开头。`metavar`是参数的简短描述，将出现在短使用说明中。不需要其他参数。

[![8](assets/8.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO2-8)

该函数返回解析参数的结果。帮助标志或参数错误将导致`argparse`打印使用说明或错误消息并退出程序。

[![9](assets/9.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO2-9)

本书中的所有程序都将始于`main()`函数。

[![10](assets/10.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO2-10)

`main()`中的第一步始终是调用`get_args()`。如果此调用成功，则参数必定是有效的。

[![11](assets/11.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO2-11)

`DNA`值可以通过`args.dna`属性获得，因为这是参数的名称。

[![12](assets/12.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO2-12)

这是Python程序中的常见习语，用于检测程序是否正在执行（而不是被导入），并执行`main()`函数。

当以`./dna.py`这样的形式调用程序时，Unix shell会使用shebang行。在Windows上，你需要运行**`python.exe dna.py`**来执行该程序。

虽然这段代码完全正常工作，但从`get_args()`返回的值是一个在程序运行时*动态生成*的`argparse.Namespace`对象。也就是说，我正在使用像`parser.add_argument()`这样的代码在运行时修改这个对象的结构，因此Python无法在*编译时*确定解析参数中会有哪些属性或它们的类型。虽然你可能很明显只有一个必需的字符串参数，但代码中没有足够的信息让Python能够辨别这一点。

编译程序是将其转换为计算机可以执行的机器代码。某些语言（如C）必须在运行之前单独编译。Python程序通常在一步中编译并运行，但仍有编译阶段。有些错误可以在编译时捕获，而其他错误则直到运行时才会出现。例如，语法错误会阻止编译。最好在编译时有错误，而不是在运行时有错误。

要了解为什么这可能是个问题，我将修改`main()`函数，引入一个类型错误。也就是说，我会故意误用`args.dna`值的*类型*。除非另有说明，通过`argparse`从命令行返回的所有参数值都是字符串。如果我试图将字符串`args.dna`除以整数值2，Python将引发异常并在运行时崩溃程序：

```py
def main():
    args = get_args()
    print(args.dna / 2) ![1](assets/1.png)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO3-1)

将字符串除以整数会产生一个异常。

如果我运行程序，它会如预期地崩溃：

```py
$ ./dna.py ACGT
Traceback (most recent call last):
  File "./dna.py", line 30, in <module>
    main()
  File "./dna.py", line 25, in main
    print(args.dna / 2)
TypeError: unsupported operand type(s) for /: 'str' and 'int'
```

我们的大脑明白这是一个不可避免的错误，但Python看不到这个问题。我需要的是在程序运行时无法修改的*静态*参数定义。继续阅读，了解类型注解和其他工具如何检测这类错误。

## 查找代码中错误的工具

这里的目标是在Python中编写正确、可复现的程序。有没有办法发现并避免像在数值运算中误用字符串这样的问题？`python3`解释器没有找到阻止我运行代码的问题。也就是说，程序在语法上是正确的，因此在前一节中的代码产生了一个*运行时错误*，因为只有当我执行程序时才会出错。多年前我曾在一个团队工作，我们开玩笑说：“如果能编译通过，就发布吧！”这显然是一种短视的编码方式。

我可以使用诸如linters和类型检查器之类的工具来找出代码中的一些问题。*Linters*是一种检查程序风格和许多种错误的工具，超越了简单的语法错误。[`pylint`工具](https://www.pylint.org)是一个流行的Python linter，我几乎每天都在使用。它能找到这个问题吗？显然不能，因为它给出了最大的赞扬：

```py
$ pylint dna.py

-------------------------------------------------------------------
Your code has been rated at 10.00/10 (previous run: 9.78/10, +0.22)
```

[`flake8`](https://oreil.ly/b3Qtj)工具是另一个我经常与`pylint`结合使用的检查器，因为它会报告不同类型的错误。当我运行`flake8 dna.py`时，没有输出，这意味着它没有发现要报告的错误。

[`mypy`](http://mypy-lang.org)工具是 Python 的静态*类型检查器*，意味着它旨在发现诸如试图将字符串除以数字等错误使用类型的问题。`pylint`和`flake8`都不会捕获类型错误，所以我不能合理地惊讶它们错过了这个错误。那么`mypy`有什么要说的呢？

```py
$ mypy dna.py
Success: no issues found in 1 source file
```

嗯，这有点令人失望；然而，你必须理解，`mypy`未能报告问题*是因为没有类型信息*。也就是说，`mypy`没有信息来表明将`args.dna`除以 2 是错误的。我很快就会修复这个问题。

## 引入命名元组

为了避免动态生成对象带来的问题，本书中的所有程序都将使用命名元组数据结构来静态定义从`get_args()`获取的参数。*元组*本质上是不可变的列表，通常用于表示 Python 中的记录型数据结构。这其中有很多内容需要理解，所以让我们先回到列表。

首先，*列表*是有序的项目序列。项目可以是异构的；理论上来说，这意味着所有项目可以是不同类型的，但实际上，混合类型通常是一个坏主意。我将使用`python3` REPL来演示列表的一些方面。我建议你使用**`help(list)`**阅读文档。

使用空方括号（`[]`）创建一个空列表，用于保存一些序列：

```py
>>> seqs = []
```

`list()`函数还将创建一个新的空列表：

```py
>>> seqs = list()
```

使用`type()`函数返回变量的类型来验证这是一个列表：

```py
>>> type(seqs)
<class 'list'>
```

列表有一些方法可以在列表末尾添加值，比如`list.append()`来添加一个值：

```py
>>> seqs.append('ACT')
>>> seqs
['ACT']
```

和`list.extend()`来添加多个值：

```py
>>> seqs.extend(['GCA', 'TTT'])
>>> seqs
['ACT', 'GCA', 'TTT']
```

如果在 REPL 中仅键入变量本身，它将被评估并转换为文本表示形式：

```py
>>> seqs
['ACT', 'GCA', 'TTT']
```

这基本上就是当你`print()`一个变量时发生的事情：

```py
>>> print(seqs)
['ACT', 'GCA', 'TTT']
```

可以使用索引*原地*修改任何值。请记住，Python 中的所有索引都是从 0 开始的，因此 0 是第一个元素。将第一个序列改为`TCA`：

```py
>>> seqs[0] = 'TCA'
```

验证已更改：

```py
>>> seqs
['TCA', 'GCA', 'TTT']
```

与列表类似，元组是一种可能异构对象的有序序列。当你在一系列项目之间放置逗号时，你就创建了一个元组：

```py
>>> seqs = 'TCA', 'GCA', 'TTT'
>>> type(seqs)
<class 'tuple'>
```

典型的做法是在元组值周围加上括号，以使其更加明确：

```py
>>> seqs = ('TCA', 'GCA', 'TTT')
>>> type(seqs)
<class 'tuple'>
```

不像列表，元组一旦创建就无法更改。如果你阅读`help(tuple)`，你会看到元组是一个*内置的不可变序列*，所以我无法添加值：

```py
>>> seqs.append('GGT')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'tuple' object has no attribute 'append'
```

或修改现有的值：

```py
>>> seqs[0] = 'TCA'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
```

在 Python 中，使用元组表示记录是相当常见的。例如，我可以表示一个`Sequence`，它有一个唯一的 ID 和一个碱基字符串：

```py
>>> seq = ('CAM_0231669729', 'GTGTTTATTCAATGCTAG')
```

虽然可以像使用列表一样使用索引从元组中获取值，但这样做很麻烦且容易出错。*命名元组*允许我为字段指定名称，这使得它们更加易于使用。要使用命名元组，可以从`collections`模块导入`namedtuple()`函数：

```py
>>> from collections import namedtuple
```

如[图1-2](#fig_1.2)所示，我使用`namedtuple()`函数创建了一个具有`id`和`seq`字段的`Sequence`的概念：

```py
>>> Sequence = namedtuple('Sequence', ['id', 'seq'])
```

![mpfb 0102](assets/mpfb_0102.png)

###### 图1-2\. `namedtuple()`函数生成一种方法，用于创建具有`id`和`seq`字段的`Sequence`类对象

这里的`Sequence`究竟是什么？

```py
>>> type(Sequence)
<class 'type'>
```

我刚创造了一个新的类型。你可以称`Sequence()`函数为*工厂*，因为它是用来生成`Sequence`类的新对象的函数。这是这些工厂函数和类名常见的命名约定，用以区分它们。

就像我可以使用`list()`函数创建一个新的列表一样，我可以使用`Sequence()`函数创建一个新的`Sequence`对象。我可以按位置传递`id`和`seq`值，以匹配它们在类中定义的顺序：

```py
>>> seq1 = Sequence('CAM_0231669729', 'GTGTTTATTCAATGCTAG')
>>> type(seq1)
<class '__main__.Sequence'>
```

或者我可以使用字段名称，并按任意顺序将它们作为键/值对传递：

```py
>>> seq2 = Sequence(seq='GTGTTTATTCAATGCTAG', id='CAM_0231669729')
>>> seq2
Sequence(id='CAM_0231669729', seq='GTGTTTATTCAATGCTAG')
```

尽管可以使用索引访问ID和序列：

```py
>>> 'ID = ' + seq1[0]
'ID = CAM_0231669729'
>>> 'seq = ' + seq1[1]
'seq = GTGTTTATTCAATGCTAG'
```

…命名元组的整个意义在于使用字段名称：

```py
>>> 'ID = ' + seq1.id
'ID = CAM_0231669729'
>>> 'seq = ' + seq1.seq
'seq = GTGTTTATTCAATGCTAG'
```

记录的值保持不可变：

```py
>>> seq1.id = 'XXX'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: can't set attribute
```

我经常希望在我的代码中保证一个值不会被意外改变。Python没有声明变量为*常量*或不可变的方式。元组默认是不可变的，我认为用一个不能被修改的数据结构来表示程序的输入是有道理的。这些输入是神圣的，几乎不应该被修改。

## 为命名元组添加类型

尽管`namedtuple()`很好用，但通过从`typing`模块导入`NamedTuple`类，可以使其变得更好。此外，可以使用此语法为字段分配*类型*。请注意，在REPL中需要使用空行来指示块已完成：

```py
>>> from typing import NamedTuple
>>> class Sequence(NamedTuple):
...     id: str
...     seq: str
...
```

您看到的`...`是行继续符。REPL显示到目前为止输入的内容不是完整的表达式。您需要输入一个空行来告诉REPL您已经完成了代码块。

与`namedtuple()`方法一样，`Sequence`是一个新类型：

```py
>>> type(Sequence)
<class 'type'>
```

实例化一个新的`Sequence`对象的代码是相同的：

```py
>>> seq3 = Sequence('CAM_0231669729', 'GTGTTTATTCAATGCTAG')
>>> type(seq3)
<class '__main__.Sequence'>
```

我仍然可以通过名称访问字段：

```py
>>> seq3.id, seq3.seq
('CAM_0231669729', 'GTGTTTATTCAATGCTAG')
```

由于我定义了两个字段都是`str`类型，你可能会认为这样*不*会起作用：

```py
>>> seq4 = Sequence(id='CAM_0231669729', seq=3.14)
```

很抱歉告诉你，Python本身会忽略类型信息。你可以看到我希望是`str`的`seq`字段实际上是一个`float`：

```py
>>> seq4
Sequence(id='CAM_0231669729', seq=3.14)
>>> type(seq4.seq)
<class 'float'>
```

那么这对我们有什么帮助呢？它在REPL中对我没有帮助，但是在我的源代码中添加类型将允许像`mypy`这样的类型检查工具找到这些错误。

## 使用命名元组表示参数

我希望表示程序参数的数据结构包括类型信息。与`Sequence`类一样，我可以定义一个派生自`NamedTuple`类型的类，在其中可以*静态定义带有类型的数据结构*。我喜欢称这个类为`Args`，但你可以根据喜好更改。我知道这可能看起来有点大材小用，但相信我，这种细节以后会有所回报。

最新的`new.py`使用了来自`typing`模块的`NamedTuple`类。我建议你这样定义和表示参数：

```py
#!/usr/bin/env python3
"""Tetranucleotide frequency"""

import argparse
from typing import NamedTuple ![1](assets/1.png)

class Args(NamedTuple): ![2](assets/2.png)
    """ Command-line arguments """
    dna: str ![3](assets/3.png)

# --------------------------------------------------
def get_args() -> Args: ![4](assets/4.png)
    """ Get command-line arguments """

    parser = argparse.ArgumentParser(
        description='Tetranucleotide frequency',
        formatter_class=argparse.ArgumentDefaultsHelpFormatter)

    parser.add_argument('dna', metavar='DNA', help='Input DNA sequence')

    args = parser.parse_args() ![5](assets/5.png)

    return Args(args.dna) ![6](assets/6.png)

# --------------------------------------------------
def main() -> None: ![7](assets/7.png)
    """ Make a jazz noise here """

    args = get_args()
    print(args.dna / 2) ![8](assets/8.png)

# --------------------------------------------------
if __name__ == '__main__':
    main()
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO4-1)

从`typing`模块导入`NamedTuple`类。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO4-2)

为参数定义一个基于`NamedTuple`类的`class`。请参阅以下注释。

[![3](assets/3.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO4-3)

类中有一个名为`dna`的单一字段，类型为`str`。

[![4](assets/4.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO4-4)

`get_args()`函数的类型注解显示它返回一个`Args`类型的对象。

[![5](assets/5.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO4-5)

像以前一样解析参数。

[![6](assets/6.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO4-6)

返回一个包含`args.dna`单一值的新`Args`对象。

[![7](assets/7.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO4-7)

`main()`函数没有`return`语句，因此返回默认的`None`值。

[![8](assets/8.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO4-8)

这是早期程序的类型错误。

如果你在这个程序上运行`pylint`，可能会遇到错误“继承*NamedTuple*，这不是一个类（inherit-non-class）”和“太少的公共方法（0/2）（too-few-public-methods）”。你可以通过将“inherit-non-class”和“too-few-public-methods”添加到你的*pylintrc*文件的“disable”部分来禁用这些警告，或者使用GitHub仓库根目录中包含的*pylintrc*文件。

如果你运行这个程序，你会看到它仍然会创建相同的未捕获异常。`flake8`和`pylint`都会继续报告程序看起来很好，但现在看看`mypy`告诉我的是什么：

```py
$ mypy dna.py
dna.py:32: error: Unsupported operand types for / ("str" and "int")
Found 1 error in 1 file (checked 1 source file)
```

错误消息显示，在第 32 行存在问题，涉及除法 (`/`) 操作符的操作数。我混合了字符串和整数值。如果没有类型注解，`mypy` 将无法找到此错误。如果没有`mypy` 的警告，我将不得不运行我的程序来找到它，确保执行包含错误的代码分支。在这种情况下，这一切都显而易见且微不足道，但在一个具有数百或数千行代码（LOC）、许多函数和逻辑分支（如 `if`/`else`）的更大程序中，我可能不会偶然发现这个错误。我依赖于类型和像 `mypy`（还有 `pylint`、`flake8` 等）这样的程序来修正这些类型的错误，而不是仅依赖于测试，更糟糕的是等待用户报告错误。

## 从命令行或文件读取输入

当您试图证明您的程序在 Rosalind.info 网站上运行时，您将下载一个包含程序输入的数据文件。通常，此数据要比问题描述中的示例数据大得多。例如，此问题的示例 DNA 字符串长度为 70 个碱基，但我下载的一个尝试的输入数据文件长度为 910 个碱基。

让程序同时从命令行和文本文件中读取输入，这样您就不必从下载的文件中复制和粘贴内容。这是我常用的一种模式，我更喜欢在 `get_args()` 函数内处理此选项，因为这涉及处理命令行参数。

首先，修正程序，以便打印 `args.dna` 的值而不进行除法操作：

```py
def main() -> None:
    args = get_args()
    print(args.dna) ![1](assets/1.png)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO5-1)

修复除法类型错误。

检查它是否工作：

```py
$ ./dna.py ACGT
ACGT
```

对于接下来的部分，您需要引入 `os` 模块以与操作系统交互。在其他 `import` 语句顶部添加 `import os`，然后将这两行代码添加到您的 `get_args()` 函数中：

```py
def get_args() -> Args:
    """ Get command-line arguments """

    parser = argparse.ArgumentParser(
        description='Tetranucleotide frequency',
        formatter_class=argparse.ArgumentDefaultsHelpFormatter)

    parser.add_argument('dna', metavar='DNA', help='Input DNA sequence')

    args = parser.parse_args()

    if os.path.isfile(args.dna):  ![1](assets/1.png)
        args.dna = open(args.dna).read().rstrip()   ![2](assets/2.png)

    return Args(args.dna)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO6-1)

检查 `dna` 值是否是一个文件。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO6-2)

调用 `open()` 打开文件句柄，然后链式调用 `fh.read()` 方法返回一个字符串，接着链式调用 `str.rstrip()` 方法删除尾部空白。

`fh.read()` 函数将整个文件读入一个变量中。在这种情况下，输入文件很小，所以这应该没问题，但在生物信息学中处理数千兆字节大小的文件是非常普遍的。在大文件上使用 `read()` 可能会导致程序崩溃甚至整个计算机崩溃。稍后我将展示如何逐行读取文件以避免这种情况。

现在运行您的程序，以确保它能处理字符串值：

```py
$ ./dna.py ACGT
ACGT
```

然后将文本文件用作参数：

```py
$ ./dna.py tests/inputs/input2.txt
AGCTTTTCATTCTGACTGCAACGGGCAATATGTCTCTGTGTGGATTAAAAAAAGAGTGTCTGATAGCAGC
```

现在你有一个灵活的程序，可以从两个来源读取输入。运行**`mypy dna.py`**以确保没有问题。

## 测试你的程序

你从罗莎琳描述中了解到，给定输入`ACGT`，程序应该打印`1 1 1 1`，因为这是*A*、*C*、*G*和*T*各自的数量。在*01_dna/tests*目录中，有一个名为*dna_test.py*的文件，其中包含对`dna.py`程序的测试。我为你编写了这些测试，这样你就可以看到使用一种方法开发程序，并能相当确信地告诉你程序是否正确的情况。这些测试非常基础——给定一个输入字符串，程序应该打印四种核苷酸的正确计数。当程序报告正确的数字时，它就正常工作了。

在*01_dna*目录中，我想让你运行**`pytest`**（或在Windows上运行**`python3 -m pytest`**或**`pytest.exe`**）。程序将递归搜索所有以*test_*开头或以*_test.py*结尾的文件，然后运行这些文件中以*test_*开头命名的函数。

当你运行**`pytest`**时，会看到大量输出，其中大多数是失败的测试。要理解为什么这些测试失败，让我们看一下*tests/dna_test.py*模块：

```py
""" Tests for dna.py """ ![1](assets/1.png)

import os ![2](assets/2.png)
import platform ![3](assets/3.png)
from subprocess import getstatusoutput ![4](assets/4.png)

PRG = './dna.py' ![5](assets/5.png)
RUN = f'python {PRG}' if platform.system() == 'Windows' else PRG ![6](assets/6.png)
TEST1 = ('./tests/inputs/input1.txt', '1 2 3 4') ![7](assets/7.png)
TEST2 = ('./tests/inputs/input2.txt', '20 12 17 21')
TEST3 = ('./tests/inputs/input3.txt', '196 231 237 246')
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO7-1)

这是该模块的文档字符串。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO7-2)

标准的`os`模块将与操作系统进行交互。

[![3](assets/3.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO7-3)

使用`platform`模块来确定是否在Windows上运行。

[![4](assets/4.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO7-4)

我从`subprocess`模块导入一个函数，用于运行`dna.py`程序并捕获输出和状态。

[![5](assets/5.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO7-5)

这些是程序的全局变量。我倾向于在我的测试中避免使用全局变量。在这里，我想定义一些将在函数中使用的值。我喜欢使用大写名称来突出显示全局可见性。

[![6](assets/6.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO7-6)

`RUN`变量确定如何运行`dna.py`程序。在Windows上，必须使用`python`命令来运行Python程序，但在Unix平台上，可以直接执行`dna.py`程序。

[![7](assets/7.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO7-7)

`TEST*`变量是元组，定义了包含DNA字符串的文件以及该字符串的预期输出。

`pytest`模块将按照测试文件中定义的顺序运行测试函数。我经常这样组织我的测试，从最简单的情况逐步进行，因此在失败后通常没有继续进行的必要。例如，第一个测试始终是检查要测试的程序是否存在。如果不存在，则没有继续运行更多测试的意义。我建议您在运行`pytest`时使用`-x`标志以在第一个失败的测试处停止，并使用`-v`标志以获取详细输出。

让我们看看第一个测试。函数名为`test_exists()`，这样`pytest`就能找到它。在函数体中，我使用一个或多个`assert`语句来检查某个条件是否*真实*。^([1](ch01.html#idm45963636356152)) 这里我断言程序`dna.py`存在。这就是为什么您的程序必须存在于此目录中——否则测试将找不到它的原因：

```py
def test_exists(): ![1](assets/1.png)
    """ Program exists """

    assert os.path.exists(PRG) ![2](assets/2.png)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO8-1)

函数名必须以`test_`开头，以便`pytest`能够找到它。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO8-2)

`os.path.exists()`函数如果给定的参数是文件则返回`True`。如果返回`False`，则断言失败，此测试将失败。

我编写的下一个测试总是检查程序是否会为`-h`和`--help`标志生成使用语句。`subprocess.getstatusoutput()`函数将使用短和长帮助标志运行`dna.py`程序。在每种情况下，我希望看到程序打印以*usage:*开头的文本。这不是一个完美的测试。它不检查文档是否准确，只是看起来像是可能是使用语句。我不认为每个测试都需要完全详尽。以下是测试内容：

```py
def test_usage() -> None:
    """ Prints usage """

    for arg in ['-h', '--help']: ![1](assets/1.png)
        rv, out = getstatusoutput(f'{RUN} {arg}') ![2](assets/2.png)
        assert rv == 0 ![3](assets/3.png)
        assert out.lower().startswith('usage:') ![4](assets/4.png)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO9-1)

迭代短和长帮助标志。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO9-2)

使用参数运行程序并捕获返回值和输出。

[![3](assets/3.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO9-3)

验证程序报告的成功退出值为0。

[![4](assets/4.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO9-4)

断言程序输出的小写结果以*usage:*开头。

命令行程序通常通过返回非零值向操作系统指示错误。如果程序成功运行，应该返回`0`。有时这个非零值可能与某些内部错误代码相关联，但通常它只是表示出现了问题。我编写的程序也会力求在成功运行时报告`0`，在出现错误时报告某个非零值。

接下来，我希望确保当没有给出参数时程序会退出：

```py
def test_dies_no_args() -> None:
    """ Dies with no arguments """

    rv, out = getstatusoutput(RUN) ![1](assets/1.png)
    assert rv != 0 ![2](assets/2.png)
    assert out.lower().startswith('usage:') ![3](assets/3.png)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO10-1)

捕获运行程序时没有参数时的返回值和输出。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO10-2)

验证返回值为非零的失败代码。

[![3](assets/3.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO10-3)

检查输出是否像一个用法说明。

在测试到这一点时，我知道我有一个程序，它的名称正确，可以运行以生成文档。这意味着程序至少在语法上是正确的，这是一个不错的起点进行测试。如果您的程序存在拼写错误，则必须在达到这一点之前进行更正。

## 运行程序以测试输出

现在我需要看看程序是否按预期运行。测试程序有许多方法，我喜欢使用称为*inside-out*和*outside-in*的两种基本方法。*Inside-out*方法从测试程序内部的各个函数开始。这通常被称为*单元*测试，因为函数可以被认为是计算的基本单位，我将在解决方案部分详细介绍。我将从*outside-in*方法开始。这意味着我将像用户一样从命令行运行程序。这是一种整体方法，用来检查代码片段是否能够协同工作以创建正确的输出，因此有时被称为*集成*测试。

第一个这样的测试将DNA字符串作为命令行参数传递，并检查程序是否产生正确格式的计数：

```py
def test_arg():
    """ Uses command-line arg """

    for file, expected in [TEST1, TEST2, TEST3]: ![1](assets/1.png)
        dna = open(file).read() ![2](assets/2.png)
        retval, out = getstatusoutput(f'{RUN} {dna}') ![3](assets/3.png)
        assert retval == 0 ![4](assets/4.png)
        assert out == expected ![5](assets/5.png)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO11-1)

解压元组到包含DNA字符串和程序运行时此输入的`expected`值的`file`中。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO11-2)

打开文件并从内容中读取`dna`。

[![3](assets/3.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO11-3)

使用函数`subprocess.getstatusoutput()`运行给定的DNA字符串的程序，该函数给出程序的返回值和文本输出（也称为`STDOUT`，发音为*standard out*）。

[![4](assets/4.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO11-4)

断言返回值为`0`，这表示成功（或0个错误）。

[![5](assets/5.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO11-5)

断言程序的输出是预期的数字字符串。

下一个测试几乎相同，但这次我将文件名作为程序的参数传递，以验证它是否正确地从文件中读取DNA：

```py
def test_file():
    """ Uses file arg """

    for file, expected in [TEST1, TEST2, TEST3]:
        retval, out = getstatusoutput(f'{RUN} {file}') ![1](assets/1.png)
        assert retval == 0
        assert out == expected
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO12-1)

与第一个测试的唯一区别是，我传递的是文件名而不是文件的内容。

现在你已经查看了测试，请返回并再次运行测试。这次使用**`pytest -xv`**，其中`-v`标志用于详细输出。由于`-x`和`-v`都是短标志，你可以像`-xv`或`-vx`一样组合它们。仔细阅读输出并注意它试图告诉你程序正在打印DNA序列，但测试期望一个数字序列：

```py
$ pytest -xv
============================= test session starts ==============================
...

tests/dna_test.py::test_exists PASSED                                    [ 25%]
tests/dna_test.py::test_usage PASSED                                     [ 50%]
tests/dna_test.py::test_arg FAILED                                       [ 75%]

=================================== FAILURES ===================================
___________________________________ test_arg ___________________________________

    def test_arg():
        """ Uses command-line arg """

        for file, expected in [TEST1, TEST2, TEST3]:
            dna = open(file).read()
            retval, out = getstatusoutput(f'{RUN} {dna}')
            assert retval == 0
>           assert out == expected  ![1](assets/1.png)
E           AssertionError: assert 'ACCGGGTTTT' == '1 2 3 4'  ![2](assets/2.png)
E             - 1 2 3 4
E             + ACCGGGTTTT

tests/dna_test.py:36: AssertionError
=========================== short test summary info ============================
FAILED tests/dna_test.py::test_arg - AssertionError: assert 'ACCGGGTTTT' == '...
!!!!!!!!!!!!!!!!!!!!!!!!!! stopping after 1 failures !!!!!!!!!!!!!!!!!!!!!!!!!!!
========================= 1 failed, 2 passed in 0.35s ==========================
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO13-1)

此行开头的`>`显示这是错误的来源。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO13-2)

程序的输出是字符串`ACCGGGTTTT`，但期望值是`1 2 3 4`。由于它们不相等，引发了一个`AssertionError`异常。

让我们修复一下。如果你认为你知道如何完成程序，请立即采用你的解决方案。首先，也许尝试运行你的程序来验证它是否会报告正确数量的*A*：

```py
$ ./dna.py A
1 0 0 0
```

接着是*C*：

```py
$ ./dna.py C
0 1 0 0
```

以及继续进行*G*和*T*。然后运行**`pytest`**，看看是否通过了所有测试。

完成一份可用版本后，请考虑尝试尽可能多地寻找得到相同答案的不同方法。这被称为*重构*程序。你需要从能够正确工作的东西开始，然后尝试改进它。改进可以通过多种方式衡量。也许你找到了用更少的代码写相同想法的方法，或者也许你找到了运行更快的解决方案。无论你使用什么标准，都要继续运行**`pytest`**来确保程序是正确的。

# 解决方案1：迭代并计算字符串中的字符数

如果你不知道从哪里开始，我将与你一起解决第一个解决方案。目标是遍历DNA字符串中的所有碱基。因此，首先我需要通过在REPL中分配一些值来创建一个名为`dna`的变量：

```py
>>> dna = 'ACGT'
```

注意，任何用引号括起来的值，无论是单引号还是双引号，都是字符串。在Python中，即使是单个字符也被视为字符串。我经常使用`type()`函数来验证变量的类型，这里我看到`dna`是`str`类（字符串类）：

```py
>>> type(dna)
<class 'str'>
```

在REPL中键入`help(str)`以查看你可以在字符串上执行的所有精彩操作。在基因组学中，字符串作为数据的重要组成部分，这一数据类型尤为重要。

在Python术语中，我想*迭代*字符串的字符，这些字符在这种情况下是DNA的核苷酸。使用`for`循环可以做到这一点。Python将字符串视为有序的字符序列，`for`循环将从头到尾访问每个字符：

```py
>>> for base in dna: ![1](assets/1.png)
...     print(base)  ![2](assets/2.png)
...
A
C
G
T
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO14-1)

`dna`字符串中的每个字符都将复制到`base`变量中。你可以称其为`char`，或者`c`表示*character*，或者其他你喜欢的名字。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO14-2)

每次调用`print()`都会以换行符结束，因此你会看到每个碱基占据一行。

后面你会看到`for`循环可以与列表、字典、集合和文件中的行一起使用——基本上任何可迭代的数据结构。

## 统计核苷酸

现在我知道如何访问序列中的每个碱基后，我需要计算每个碱基的数量而不是仅打印它们。这意味着我需要一些变量来跟踪每种四个核苷酸的数量。一种方法是创建四个整数计数的变量，每个变量对应一个碱基。我将通过将它们的初始值设置为`0`来*初始化*这四个计数变量：

```py
>>> count_a = 0
>>> count_c = 0
>>> count_g = 0
>>> count_t = 0
```

我可以使用我之前展示的元组解包语法来一行写完：

```py
>>> count_a, count_c, count_g, count_t = 0, 0, 0, 0
```

我需要查看每个碱基并确定要*增加*的变量，使其值增加1。例如，如果当前的`base`是*C*，那么我应该增加`count_c`变量。我可以这样写：

```py
for base in dna:
    if base == 'C': ![1](assets/1.png)
        count_c = count_c + 1 ![2](assets/2.png)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO15-1)

`==`运算符用于比较两个值是否相等。这里我想知道当前的`base`是否等于字符串`C`。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO15-2)

将`count_c`设置为当前值加1。

`==`运算符用于比较两个值是否相等。它用于比较两个字符串或两个数字。前面我展示了，如果混合字符串和数字，使用`/`会引发异常。如果你混合类型使用这个运算符会发生什么，例如 `'3' == 3`？在未比较类型的情况下，这是一个安全的运算符吗？

如图1-3所示，使用`+=`运算符来增加变量的值是一种更短的方式，它将右侧的值（通常标记为RHS）添加到表达式的左侧（或LHS）的内容中。

![mpfb 0103](assets/mpfb_0103.png)

###### 图1-3\. `+=`运算符将在左侧的变量上加上右侧的值

由于我有四个核苷酸要检查，我需要一种方法来结合另外三个`if`表达式。Python中的语法是使用`elif`来表示*else if*，`else`表示最后的或默认情况。以下是一个可以输入到程序或REPL中的代码块，实现了一个简单的决策树：

```py
dna = 'ACCGGGTTTT'
count_a, count_c, count_g, count_t = 0, 0, 0, 0
for base in dna:
    if base == 'A':
        count_a += 1
    elif base == 'C':
        count_c += 1
    elif base == 'G':
        count_g += 1
    elif base == 'T':
        count_t += 1
```

我应该得到每个排序后的碱基的计数，分别为1、2、3和4：

```py
>>> count_a, count_c, count_g, count_t
(1, 2, 3, 4)
```

现在我需要向用户报告结果：

```py
>>> print(count_a, count_c, count_g, count_t)
1 2 3 4
```

这就是程序期望的确切输出。注意，`print()`函数接受多个值作为参数并在每个值之间插入一个空格。如果在REPL中阅读`help(print)`，你会发现可以使用`sep`参数来改变这个行为：

```py
>>> print(count_a, count_c, count_g, count_t, sep='::')
1::2::3::4
```

`print()`函数还会在输出的末尾加上一个换行符，同样可以使用`end`选项来进行更改：

```py
>>> print(count_a, count_c, count_g, count_t, end='\n-30-\n')
1 2 3 4
-30-
```

## 编写和验证解决方案

使用上述代码，你应该能够创建一个通过所有测试的程序。在编写过程中，我建议你定期运行`pylint`、`flake8`和`mypy`来检查源代码中的潜在错误。我甚至建议你为这些工具安装`pytest`扩展，以便能够定期执行此类测试：

```py
$ python3 -m pip install pytest-pylint pytest-flake8 pytest-mypy
```

或者，我已经将*requirements.txt*文件放在GitHub仓库的根目录中，列出了我在整本书中将要使用的各种依赖项。你可以使用以下命令安装所有这些模块：

```py
$ python3 -m pip install -r requirements.txt
```

有了这些扩展，你可以运行以下命令来执行不仅在*tests/dna_test.py*文件中定义的测试，还包括使用这些工具进行linting和类型检查的测试：

```py
$ pytest -xv --pylint --flake8 --mypy tests/dna_test.py
========================== test session starts ===========================
...
collected 7 items

tests/dna_test.py::FLAKE8 SKIPPED                                  [ 12%]
tests/dna_test.py::mypy PASSED                                     [ 25%]
tests/dna_test.py::test_exists PASSED                              [ 37%]
tests/dna_test.py::test_usage PASSED                               [ 50%]
tests/dna_test.py::test_dies_no_args PASSED                        [ 62%]
tests/dna_test.py::test_arg PASSED                                 [ 75%]
tests/dna_test.py::test_file PASSED                                [ 87%]
::mypy PASSED                                                      [100%]
================================== mypy ==================================

Success: no issues found in 1 source file
====================== 7 passed, 1 skipped in 0.58s ======================
```

当缓存版本表明自上次测试以来没有任何更改时，某些测试将被跳过。使用`--cache-clear`选项强制运行测试。此外，如果代码格式不正确或缩进不正确，您可能会发现无法通过linting测试。您可以使用`yapf`或`black`自动格式化代码。大多数IDE和编辑器都会提供自动格式化选项。

这么多要打字，所以我在该目录中创建了一个*Makefile*的快捷方式供你使用：

```py
$ cat Makefile
.PHONY: test

test:
	python3 -m pytest -xv --flake8 --pylint --pylint-rcfile=../pylintrc \
    --mypy dna.py tests/dna_test.py

all:
	../bin/all_test.py dna.py
```

你可以通过阅读[附录 A](app01.html#app1_makefiles)来了解这些文件的更多信息。现在，了解到如果你的系统安装了`make`，你可以使用命令**`make test`**来运行*Makefile*中的`test`目标。如果你没有安装`make`或者不想使用它，也没关系，但我建议你探索一下*Makefile*如何用于文档化和自动化流程。

有许多方法可以编写`dna.py`的通过版本，我想鼓励你在阅读解决方案之前继续探索。最重要的是，我希望你习惯于修改你的程序然后运行测试来验证其工作。这就是*测试驱动开发*的循环，我首先创建某些度量标准来判断程序何时正确工作。在这种情况下，就是由`pytest`运行的*dna_test.py*程序。

测试确保我不偏离目标，它们还让我知道何时满足程序的要求。它们是作为我可以执行的程序而具体化的规格（也称为*规格*）。否则，我怎么知道程序何时工作或何时完成呢？或者，正如路易斯·斯里格利所说的，“没有需求或设计，编程就是在一个空白文本文件中添加错误的艺术。”

测试对于创建可复制的程序至关重要。除非您能绝对自动地证明在运行良好和坏数据时程序的正确性和可预测性，否则您不是在编写优秀的软件。

# 其他解决方案

我在本章早些时候写过的程序是GitHub仓库中的*solution1_iter.py*版本，所以我不会再复习那个版本了。我想向你展示几个替代方案，从简单到复杂的想法。请不要误以为它们从差到好递进。所有版本都通过了测试，所以它们都同样有效。重点是探索Python在解决常见问题时的各种可能性。请注意，我将省略它们共有的代码，例如`get_args()`函数。

## 解决方案2：创建一个`count()`函数并添加一个单元测试。

我想展示的第一个变体将把所有计数代码从`main()`函数中移到一个`count()`函数中。你可以在程序的任何地方定义这个函数，但我通常喜欢先写`get_args()`，然后是`main()`，然后是其他函数，最后是调用`main()`的最后一对语句之前。

对于以下函数，您还需要导入`typing.Tuple`值：

```py
def count(dna: str) -> Tuple[int, int, int, int]: ![1](assets/1.png)
    """ Count bases in DNA """

    count_a, count_c, count_g, count_t = 0, 0, 0, 0 ![2](assets/2.png)
    for base in dna:
        if base == 'A':
            count_a += 1
        elif base == 'C':
            count_c += 1
        elif base == 'G':
            count_g += 1
        elif base == 'T':
            count_t += 1

    return (count_a, count_c, count_g, count_t) ![3](assets/3.png)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO16-1)

类型显示该函数接受一个字符串，并返回一个包含四个整数值的元组。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO16-2)

这是从`main()`中进行计数的代码。

[![3](assets/3.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO16-3)

返回一个包含四个计数的元组。

有许多理由将此代码移入函数中。首先，这是一个*计算单元*——给定一个DNA字符串，返回四核苷酸频率——因此封装它是有意义的。这将使`main()`更短更易读，并允许我为函数编写单元测试。由于函数名为`count()`，我喜欢将单元测试命名为`test_count()`。我将此函数放在了`dna.py`程序中，正好在`count()`函数后面，而不是放在`dna_test.py`程序中，这只是为了方便起见。对于简短的程序，我倾向于将函数和单元测试放在源代码中的一起，但随着项目变大，我会将单元测试分离到单独的模块中。以下是测试函数：

```py
def test_count() -> None: ![1](assets/1.png)
    """ Test count """

    assert count('') == (0, 0, 0, 0) ![2](assets/2.png)
    assert count('123XYZ') == (0, 0, 0, 0)
    assert count('A') == (1, 0, 0, 0) ![3](assets/3.png)
    assert count('C') == (0, 1, 0, 0)
    assert count('G') == (0, 0, 1, 0)
    assert count('T') == (0, 0, 0, 1)
    assert count('ACCGGGTTTT') == (1, 2, 3, 4)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO17-1)

函数名必须以 `test_` 开头，以便被 `pytest` 找到。这里的类型显示该测试不接受参数，并且因为没有 `return` 语句，返回默认值 `None`。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO17-2)

我喜欢用预期和意外的值来测试函数，以确保它们返回合理的结果。空字符串应该返回全部零值。

[![3](assets/3.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO17-3)

其余的测试确保每个碱基在正确的位置上报告。

为了验证我的函数是否有效，我可以在 `dna.py` 程序上使用 `pytest`：

```py
$ pytest -xv dna.py
=========================== test session starts ===========================
...

dna.py::test_count PASSED                                           [100%]

============================ 1 passed in 0.01s ============================
```

第一个测试传递空字符串，并期望得到所有零的计数。这是一个判断调用，说实话。你可能决定你的程序应该向用户抱怨没有输入。也就是说，可以使用空字符串作为输入运行程序，而这个版本将报告如下：

```py
$ ./dna.py ""
0 0 0 0
```

同样，如果我传递一个空文件，我会得到相同的答案。使用 **`touch`** 命令创建一个空文件：

```py
$ touch empty
$ ./dna.py empty
0 0 0 0
```

在 Unix 系统上，`/dev/null` 是一个特殊的文件句柄，返回空值：

```py
$ ./dna.py /dev/null
0 0 0 0
```

你可能觉得没有输入是一个错误，并报告它。测试的重要之处在于它迫使我考虑这个问题。例如，如果给定空字符串，`count()` 函数应该返回零还是引发异常？如果输入为空，程序应该崩溃并退出，还是以非零状态结束？这些都是你为程序需要做出的决定。

现在我在 `dna.py` 代码中有了一个单元测试，我可以在该文件上运行 `pytest` 看它是否通过：

```py
$ pytest -v dna.py
============================ test session starts =============================
...
collected 1 item

dna.py::test_count PASSED                                              [100%]

============================= 1 passed in 0.01s ==============================
```

当我编写代码时，我喜欢编写只做一件事情的函数，尽可能少的参数。然后我喜欢在源代码中函数的后面写一个类似 `test_` 加上函数名的命名测试。如果我发现我有很多这样的单元测试，我可能决定将它们移到一个单独的文件中，并让 `pytest` 执行该文件。

要使用这个新函数，修改 `main()` 如下：

```py
def main() -> None:
    args = get_args()
    count_a, count_c, count_g, count_t = count(args.dna) ![1](assets/1.png)
    print('{} {} {} {}'.format(count_a, count_c, count_g, count_t)) ![2](assets/2.png)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO18-1)

将从 `count()` 返回的四个值解包到单独的变量中。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO18-2)

使用 `str.format()` 创建输出字符串。

让我们稍微关注一下Python的`str.format()`。如图[1-4](#fig_1.4)所示，字符串`'{} {} {} {}'`是我想要生成的输出的模板，并且我直接在字符串字面值上调用`str.format()`函数。这是Python中的一个常见习惯用法，您还将在`str.join()`函数中看到。重要的是要记住，在Python中，即使是字面字符串（在引号中直接存在于您的源代码中的字符串），也是您可以调用方法的*对象*。

![mpfb 0104](assets/mpfb_0104.png)

###### 图1-4\. `str.format()`函数使用花括号定义占位符，这些占位符将用参数的值填充。

每个字符串模板中的`{}`都是函数参数提供的某个值的占位符。使用这个函数时，您需要确保占位符的数量与参数的数量相同。参数按照它们提供的顺序插入。稍后我会详细介绍`str.format()`函数。

我不必展开`count()`函数返回的元组。如果我在元组前面加上一个星号(`*`)来*splat*它，我可以将整个元组作为参数传递给`str.format()`函数。这告诉Python将元组扩展为其值：

```py
def main() -> None:
    args = get_args()
    counts = count(args.dna) ![1](assets/1.png)
    print('{} {} {} {}'.format(*counts)) ![2](assets/2.png)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO20-1)

`counts`变量是包含整数基数计数的4元组。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO19-2)

`*counts`语法将元组扩展为格式字符串所需的四个值；否则，元组将被解释为单个值。

因为我只使用`counts`变量一次，我可以跳过赋值，将代码缩短为一行：

```py
def main() -> None:
    args = get_args()
    print('{} {} {} {}'.format(*count(args.dna))) ![1](assets/1.png)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO20-1)

将`count()`函数的返回值直接传递给`str.format()`方法。

第一种解决方案可能更易于阅读和理解，并且像`flake8`这样的工具将能够发现`{}`占位符数量与变量数量不匹配的情况。简单、冗长、明显的代码通常比紧凑、聪明的代码更好。尽管如此，了解元组解包和扩展变量的技术是很有用的，我将在后续程序中使用这些技术。

## 解决方案3：使用`str.count()`

前面的`count()`函数结果相当冗长。我可以使用`str.count()`方法将这个函数写成单行代码。此函数将计算一个字符串在另一个字符串中出现的次数。让我在 REPL 中演示给你看：

```py
>>> seq = 'ACCGGGTTTT'
>>> seq.count('A')
1
>>> seq.count('C')
2
```

如果未找到字符串，它将报告`0`，使其能够安全地计算所有四个核苷酸，即使输入序列缺少一个或多个碱基：

```py
>>> 'AAA'.count('T')
0
```

这是使用这个思想的`count()`函数的新版本：

```py
def count(dna: str) -> Tuple[int, int, int, int]: ![1](assets/1.png)
    """ Count bases in DNA """

    return (dna.count('A'), dna.count('C'), dna.count('G'), dna.count('T')) ![2](assets/2.png)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO21-1)

签名与之前相同。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO21-2)

对每个四个碱基调用 `dna.count()` 方法。

这段代码更加简洁，我可以使用相同的单元测试来验证它的正确性。这是一个关键点：*函数应该像黑匣子一样运行*。也就是说，我不知道或不关心盒子里面发生了什么。输入一些东西，得到一个答案，我只关心答案是否正确。只要外部的约定——参数和返回值保持不变，我可以自由地改变盒子里面发生的事情。

这里是使用 Python 的 f-string 语法在 `main()` 函数中创建输出字符串的另一种方式：

```py
def main() -> None:
    args = get_args()
    count_a, count_c, count_g, count_t = count(args.dna) ![1](assets/1.png)
    print(f'{count_a} {count_c} {count_g} {count_t}') ![2](assets/2.png)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO22-1)

将元组解包为四个计数。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO22-2)

使用 f-string 执行变量插值。

它被称为 *f-string*，因为在引号之前有一个 `f`。我使用 *format* 这个助记符来提醒自己这是用来格式化字符串的。Python 还有一个 *raw* 字符串，以 `r` 开头，稍后我会讨论它。Python 中的所有字符串——裸字符串、f-字符串或r-字符串——都可以用单引号或双引号括起来。这没有区别。

使用 f-string，`{}` 占位符可以执行 *变量插值*，这是一个术语，意味着将变量转换为其内容。这些大括号甚至可以执行代码。例如，`len()` 函数将返回字符串的长度，并且可以在大括号内执行：

```py
>>> seq = 'ACGT'
>>> f'The sequence "{seq}" has {len(seq)} bases.'
'The sequence "ACGT" has 4 bases.'
```

我通常发现使用 `str.format()` 相比等效的代码更易读。选择哪种方式主要是风格上的决定。我建议选择能使你的代码更易读的那种方式。

## 解决方案 4：使用字典计算所有字符的数量

到目前为止，我讨论了 Python 的字符串、列表和元组。下一个解决方案介绍了 *字典*，它们是键/值存储。我想展示一个内部使用字典的 `count()` 函数版本，这样我可以强调一些重要的理解点：

```py
def count(dna: str) -> Tuple[int, int, int, int]: ![1](assets/1.png)
    """ Count bases in DNA """

    counts = {} ![2](assets/2.png)
    for base in dna: ![3](assets/3.png)
        if base not in counts: ![4](assets/4.png)
            counts[base] = 0 ![5](assets/5.png)
        counts[base] += 1 ![6](assets/6.png)

    return (counts.get('A', 0), ![7](assets/7.png)
            counts.get('C', 0),
            counts.get('G', 0),
            counts.get('T', 0))
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO23-1)

在内部，我会使用一个字典，但函数签名不会改变。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO23-2)

初始化一个空字典来存储`counts`。

[![3](assets/3.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO23-3)

使用`for`循环遍历序列。

[![4](assets/4.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO23-4)

检查字典中是否尚不存在这个碱基。

[![5](assets/5.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO23-5)

将这个碱基的值初始化为`0`。

[![6](assets/6.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO23-6)

增加这个碱基的计数1。

[![7](assets/7.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO23-7)

使用`dict.get()`方法获取每个碱基的计数或默认值`0`。

再次强调，这个函数的契约——类型签名——没有改变。输入仍然是字符串，输出仍然是4个整数的元组。在函数内部，我将使用一个我将使用空花括号初始化的字典：

```py
>>> counts = {}
```

我也可以使用`dict()`函数。两者都没有优势：

```py
>>> counts = dict()
```

我可以使用`type()`函数来检查这是否是一个字典：

```py
>>> type(counts)
<class 'dict'>
```

`isinstance()`函数是检查变量类型的另一种方式：

```py
>>> isinstance(counts, dict)
True
```

我的目标是创建一个字典，其中每个碱基都作为*键*，其出现次数作为*值*。例如，对于序列`ACCGGGTTT`，我希望`counts`看起来像这样：

```py
>>> counts
{'A': 1, 'C': 2, 'G': 3, 'T': 4}
```

我可以使用方括号和键名访问任何一个值，如下所示：

```py
>>> counts['G']
3
```

如果尝试访问一个不存在的字典键，Python将引发`KeyError`异常：

```py
>>> counts['N']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'N'
```

我可以使用`in`关键字来查看字典中是否存在一个键：

```py
>>> 'N' in counts
False
>>> 'T' in counts
True
```

当我遍历序列中的每个碱基时，我需要查看`counts`字典中是否存在该碱基。如果不存在，我需要将其初始化为`0`。然后，我可以安全地使用`+=`操作符将碱基的计数增加1：

```py
>>> seq = 'ACCGGGTTTT'
>>> counts = {}
>>> for base in seq:
...     if not base in counts:
...         counts[base] = 0
...     counts[base] += 1
...
>>> counts
{'A': 1, 'C': 2, 'G': 3, 'T': 4}
```

最后，我想返回每个碱基的4元组计数。你可能会认为这会起作用：

```py
>>> counts['A'], counts['C'], counts['G'], counts['T']
(1, 2, 3, 4)
```

但是请问自己，如果序列中缺少其中一个碱基会发生什么？这段代码能通过我编写的单元测试吗？肯定不行。因为空字符串的第一个测试将生成`KeyError`异常。安全地询问字典值的方法是使用`dict.get()`方法。如果键不存在，则返回`None`：

```py
>>> counts.get('T')
4
>>> counts.get('N')
```

`dict.get()`方法接受一个可选的第二个参数，即在键不存在时返回的默认值，因此这是返回4个碱基计数的最安全方法：

```py
>>> counts.get('A', 0), counts.get('C', 0), counts.get('G', 0),
    counts.get('T', 0)
(1, 2, 3, 4)
```

无论你在`count()`函数内写什么，确保它能通过`test_count()`单元测试。

## 解决方案5：仅计算所需的碱基

前面的解决方案将计算输入序列中的每个字符，但如果我只想计算四个核苷酸呢？在这个解决方案中，我将初始化一个只包含所需碱基且值为`0`的字典。我还需要引入`typing.Dict`来运行这段代码：

```py
def count(dna: str) -> Dict[str, int]: ![1](assets/1.png)
    """ Count bases in DNA """

    counts = {'A': 0, 'C': 0, 'G': 0, 'T': 0} ![2](assets/2.png)
    for base in dna: ![3](assets/3.png)
        if base in counts: ![4](assets/4.png)
            counts[base] += 1 ![5](assets/5.png)

    return counts ![6](assets/6.png)
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO24-1)

现在的签名表明我将返回一个具有字符串键和整数值的字典。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO24-2)

使用四个碱基作为键和值为 `0` 的方式初始化 `counts` 字典。

[![3](assets/3.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO24-3)

遍历碱基。

[![4](assets/4.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO24-4)

检查碱基是否作为键存在于 `counts` 字典中。

[![5](assets/5.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO24-5)

如果是这样，则将此碱基的 `counts` 增加 1。

[![6](assets/6.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO24-6)

返回 `counts` 字典。

由于 `count()` 函数现在返回的是一个字典而不是元组，需要更改 `test_count()` 函数：

```py
def test_count() -> None:
    """ Test count """

    assert count('') == {'A': 0, 'C': 0, 'G': 0, 'T': 0} ![1](assets/1.png)
    assert count('123XYZ') == {'A': 0, 'C': 0, 'G': 0, 'T': 0} ![2](assets/2.png)
    assert count('A') == {'A': 1, 'C': 0, 'G': 0, 'T': 0}
    assert count('C') == {'A': 0, 'C': 1, 'G': 0, 'T': 0}
    assert count('G') == {'A': 0, 'C': 0, 'G': 1, 'T': 0}
    assert count('T') == {'A': 0, 'C': 0, 'G': 0, 'T': 1}
    assert count('ACCGGGTTTT') == {'A': 1, 'C': 2, 'G': 3, 'T': 4}
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO25-1)

返回的字典将始终具有键 `A`、`C`、`G` 和 `T`。即使对于空字符串，这些键也会存在并设置为 `0`。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO25-2)

所有其他测试具有相同的输入，但现在我检查答案作为一个字典返回。

在编写这些测试时，请注意字典中键的顺序不重要。以下代码中的两个字典内容相同，即使定义方式不同：

```py
>>> counts1 = {'A': 1, 'C': 2, 'G': 3, 'T': 4}
>>> counts2 = {'T': 4, 'G': 3, 'C': 2, 'A': 1}
>>> counts1 == counts2
True
```

我想指出 `test_count()` 函数测试函数以确保其正确性，并作为文档。阅读这些测试帮助我看到可能输入和函数预期输出的结构。

这是我需要更改 `main()` 函数以使用返回的字典的方式：

```py
def main() -> None:
    args = get_args()
    counts = count(args.dna) ![1](assets/1.png)
    print('{} {} {} {}'.format(counts['A'], counts['C'], counts['G'], ![2](assets/2.png)
                               counts['T']))
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO26-1)

`counts` 现在是一个字典。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO26-2)

使用 `str.format()` 方法使用字典中的值创建输出。

## 解决方案 6：使用 `collections.defaultdict()`

通过使用 `collections` 模块中的 `defaultdict()` 函数，可以摆脱所有之前初始化字典和检查键等的工作：

```py
>>> from collections import defaultdict
```

当我使用 `defaultdict()` 函数创建一个新字典时，我告诉它值的默认类型。我不再需要在使用之前检查键，因为 `defaultdict` 类型将自动使用默认类型的代表值创建我引用的任何键。对于计数核苷酸的情况，我希望使用 `int` 类型：

```py
>>> counts = defaultdict(int)
```

默认的`int`值将是`0`。对不存在的键的任何引用将导致它被创建为值`0`：

```py
>>> counts['A']
0
```

这意味着我可以一步实例化并增加任何碱基：

```py
>>> counts['C'] += 1
>>> counts
defaultdict(<class 'int'>, {'A': 0, 'C': 1})
```

这里是我如何使用这个思路重写`count()`函数的方式：

```py
def count(dna: str) -> Dict[str, int]:
    """ Count bases in DNA """

    counts: Dict[str, int] = defaultdict(int) ![1](assets/1.png)

    for base in dna:
        counts[base] += 1 ![2](assets/2.png)

    return counts
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO27-1)

`counts`将是一个带有整数值的`defaultdict`。这里的类型注释是`mypy`所需的，以确保返回的值是正确的。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO27-2)

我可以安全地增加这个碱基的`counts`。

函数`test_count()`看起来相当不同。我一眼就能看出答案与先前版本非常不同：

```py
def test_count() -> None:
    """ Test count """

    assert count('') == {} ![1](assets/1.png)
    assert count('123XYZ') == {'1': 1, '2': 1, '3': 1, 'X': 1, 'Y': 1, 'Z': 1} ![2](assets/2.png)
    assert count('A') == {'A': 1} ![3](assets/3.png)
    assert count('C') == {'C': 1}
    assert count('G') == {'G': 1}
    assert count('T') == {'T': 1}
    assert count('ACCGGGTTTT') == {'A': 1, 'C': 2, 'G': 3, 'T': 4}
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO28-1)

给定一个空字符串，将返回一个空字典。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO28-2)

注意字符串中的每个字符都是字典中的一个键。

[![3](assets/3.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO28-3)

只有`A`存在，计数为1。

鉴于返回的字典可能不包含所有碱基，`main()`中的代码需要使用`count.get()`方法来检索每个碱基的频率：

```py
def main() -> None:
    args = get_args()
    counts = count(args.dna) ![1](assets/1.png)
    print(counts.get('A', 0), counts.get('C', 0), counts.get('G', 0), ![2](assets/2.png)
          counts.get('T', 0))
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO29-1)

`counts`将是一个可能不包含所有核苷酸的字典。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO29-2)

使用`dict.get()`方法并设置默认值为`0`是最安全的。

## 解决方案7：使用`collections.Counter()`

> 完美是在无需添加任何内容时实现的，而是在无需再去除任何内容时实现的。
> 
> 安托万·德·圣埃克絮佩里

实际上我并不太喜欢最后三个解决方案，但我需要逐步介绍如何手动使用字典和`defaultdict()`，以及如何欣赏使用`collections.Counter()`的简便性：

```py
>>> from collections import Counter
>>> Counter('ACCGGGTTT')
Counter({'G': 3, 'T': 3, 'C': 2, 'A': 1})
```

最佳代码是你永远不必写的代码，而`Counter()`是一个预打包的函数，它将返回一个包含你传递的可迭代对象中各项频率的字典。你可能也听说过这被称为*包*或*多集*。这里的可迭代对象是由字符组成的字符串，所以我得到的字典与前两个解决方案相同，但*没有编写任何代码*。

它非常简单，你几乎可以避开`count()`和`test_count()`函数，直接集成到你的`main()`中：

```py
def main() -> None:
    args = get_args()
    counts = Counter(args.dna) ![1](assets/1.png)
    print(counts.get('A', 0), counts.get('C', 0), counts.get('G', 0), ![2](assets/2.png)
          counts.get('T', 0))
```

[![1](assets/1.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO30-1)

`counts` 将是一个包含 `args.dna` 中字符频率的字典。

[![2](assets/2.png)](#co_tetranucleotide_frequency___span_class__keep_together__counting_things__span__CO30-2)

使用 `dict.get()` 仍然是最安全的，因为我不能确定所有碱基都存在。

我可以说这段代码应该放在一个 `count()` 函数中并保留测试，但 `Counter()` 函数已经经过测试并具有良好定义的接口。我认为直接使用这个函数更有意义。

# 进一步探讨

这些解决方案仅处理以大写文本提供的 DNA 序列。看到这些序列用小写字母提供并不罕见。例如，在植物基因组学中，常用小写字母表示重复 DNA 区域。通过以下操作修改程序以处理大小写输入：

1.  添加一个新的输入文件，混合大小写。

1.  在 *tests/dna_test.py* 中添加一个测试，使用这个新文件并指定对大小写不敏感的预期计数。

1.  运行新的测试，并确保您的程序失败。

1.  修改程序，直到它通过新的测试和所有以前的测试。

使用字典计算所有可用字符的解决方案似乎更灵活。也就是说，某些测试仅考虑 *A*、*C*、*G* 和 *T* 作为碱基，但如果输入序列使用 [IUPAC 代码](https://oreil.ly/qGfsO) 表示测序中的可能歧义，那么程序将需要完全重写。一个仅硬编码查看四个核苷酸的程序也对使用不同字母表的蛋白质序列无用。考虑编写一个程序的版本，它将以每个找到的字符作为第一列并在第二列中打印字符的频率输出两列。允许用户按任一列升序或降序排序。

# 复习

这一章有点庞大。接下来的章节将会稍短，因为我将在这里覆盖的许多基本思想上继续构建：

+   你可以使用 `new.py` 程序创建一个基本的 Python 程序结构，使用 `argparse` 接受和验证命令行参数。

+   `pytest` 模块将运行所有以 `test_` 开头的函数，并报告通过了多少个测试。

+   单元测试用于函数，集成测试检查程序作为整体是否工作。

+   类似 `pylint`、`flake8` 和 `mypy` 的程序可以找出代码中的各种错误。你还可以让 `pytest` 自动运行测试，看看你的代码是否通过了这些检查。

+   复杂的命令可以存储为 *Makefile* 中的一个目标，并使用 `make` 命令执行。

+   您可以使用一系列 `if`/`else` 语句创建决策树。

+   有多种方法可以计算字符串中所有字符的数量。使用 `collections.Counter()` 函数可能是创建字母频率字典的最简单方法。

+   你可以用类型注释变量和函数，并使用`mypy`来确保类型的正确使用。

+   Python REPL是一个交互式工具，用于执行代码示例和阅读文档。

+   Python社区通常遵循诸如PEP8之类的风格指南。像`yapf`和`black`这样的工具可以根据这些建议自动格式化代码，而`pylint`和`flake8`等工具将报告与指南不符的偏差。

+   Python的字符串、列表、元组和字典是非常强大的数据结构，每种都有有用的方法和丰富的文档。

+   你可以创建一个自定义的、不可变的、基于命名元组的有类型`class`。

也许你正在想哪一个是七种解决方案中最好的。像生活中的许多事情一样，这取决于情况。有些程序写起来更短，更容易理解，但当面对大数据集时可能表现不佳。在[第2章](ch02.html#ch02)中，我将向你展示如何对比程序，使用大输入进行多次运行以确定哪一个性能最佳。

^([1](ch01.html#idm45963636356152-marker)) 布尔类型是`True`或`False`，但许多其他数据类型是*truthy*或反之*falsey*。空的`str`(`""`)是falsey，所以任何非空字符串是truthy。数字`0`是falsey，所以任何非零值是truthy。空的`list`、`set`或`dict`是falsey，所以任何非空的这些都是truthy。
