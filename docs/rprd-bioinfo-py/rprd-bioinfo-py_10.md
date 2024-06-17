# 第9章\. 重叠图：使用共享的K-mers进行序列组装

*图* 是用于表示对象间两两关系的结构。如[罗莎琳德 GRPH 挑战](https://oreil.ly/kDu52)所述，本练习的目标是找到可以通过从一个序列末端到另一个序列开端的重叠来连接的序列对。这在实践中可以应用于将短的DNA读取序列连接成更长的连续序列（*contigs*）甚至整个基因组。起初，我只关注连接两个序列，但程序的第二个版本将使用可以连接任意数量序列的图结构来近似完成组装。在此实现中，用于连接序列的重叠区域必须是精确匹配的。真实世界的组装器必须允许重叠序列的大小和组成的变化。

你将学到：

+   如何使用k-mers创建重叠图

+   如何将运行时消息记录到文件中

+   如何使用`collections.defaultdict()`

+   如何使用集合交集来查找集合之间的共同元素

+   如何使用`itertools.product()`来创建列表的笛卡尔积

+   如何使用`iteration_utilities.starfilter()`函数

+   如何使用Graphviz来建模和可视化图结构

# 入门指南

此练习的代码和测试位于*09_grph*目录中。首先复制程序`grph.py`的解决方案，并请求用法：

```py
$ cd 09_grph/
$ cp solution1.py grph.py
$ ./grph.py -h
usage: grph.py [-h] [-k size] [-d] FILE

Overlap Graphs

positional arguments:
  FILE                  FASTA file ![1](assets/1.png)

optional arguments:
  -h, --help            show this help message and exit
  -k size, --overlap size
                        Size of overlap (default: 3) ![2](assets/2.png)
  -d, --debug           Debug (default: False) ![3](assets/3.png)
```

[![1](assets/1.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO1-1)

位置参数是一个必需的FASTA格式序列文件。

[![2](assets/2.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO1-2)

`-k`选项控制重叠字符串的长度，默认为`3`。

[![3](assets/3.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO1-3)

这是一个*标志*或布尔参数。当参数存在时，其值为`True`，否则为`False`。

Rosalind 页面上显示的示例输入也是第一个示例输入文件的内容：

```py
$ cat tests/inputs/1.fa
>Rosalind_0498
AAATAAA
>Rosalind_2391
AAATTTT
>Rosalind_2323
TTTTCCC
>Rosalind_0442
AAATCCC
>Rosalind_5013
GGGTGGG
```

罗莎琳德问题始终假设一个三个碱基的重叠窗口。我认为没有理由将此参数硬编码，因此我的版本包含一个`k`参数，用于指示重叠窗口的大小。例如，默认值为`3`时，可以连接三对序列：

```py
$ ./grph.py tests/inputs/1.fa
Rosalind_2391 Rosalind_2323
Rosalind_0498 Rosalind_2391
Rosalind_0498 Rosalind_0442
```

[图 9-1](#fig_9.1)显示了这些序列如何通过三个共同碱基重叠。

![mpfb 0901](assets/mpfb_0901.png)

###### 图 9-1\. 当连接 3-mers 时，三对序列形成重叠图

如图 9-2所示，当重叠窗口增加到四个碱基时，只有这些序列对中的一对可以连接。

```py
$ ./grph.py -k 4 tests/inputs/1.fa
Rosalind_2391 Rosalind_2323
```

![mpfb 0902](assets/mpfb_0902.png)

###### 图 9-2\. 当连接 4-mers 时，只有一对序列形成重叠图

最后，`--debug` 选项是一个*标志*，当参数存在时具有 `True` 值，否则为 `False`。当存在时，此选项指示程序将运行时日志消息打印到当前工作目录中名为 *.log* 的文件中。这不是 Rosalind 挑战的要求，但我认为你了解如何记录消息很重要。要看它如何运作，运行带有该选项的程序：

```py
$ ./grph.py tests/inputs/1.fa --debug
Rosalind_2391 Rosalind_2323
Rosalind_0498 Rosalind_2391
Rosalind_0498 Rosalind_0442
```

`--debug` 标志可以放在位置参数之前或之后，`argparse` 将正确解释其含义。其他参数解析器要求所有选项和标志在位置参数之前。*Vive la différence.*

现在应该有一个名为 *.log* 的文件，其内容如下，其含义将在稍后变得更加明显：

```py
$ cat .log
DEBUG:root:STARTS
defaultdict(<class 'list'>,
            {'AAA': ['Rosalind_0498', 'Rosalind_2391', 'Rosalind_0442'],
             'GGG': ['Rosalind_5013'],
             'TTT': ['Rosalind_2323']})
DEBUG:root:ENDS
defaultdict(<class 'list'>,
            {'AAA': ['Rosalind_0498'],
             'CCC': ['Rosalind_2323', 'Rosalind_0442'],
             'GGG': ['Rosalind_5013'],
             'TTT': ['Rosalind_2391']})
```

一旦你理解了你的程序应该如何工作，从一个新的 `grph.py` 程序重新开始：

```py
$ new.py -fp 'Overlap Graphs' grph.py
Done, see new script "grph.py".
```

这是我定义和验证参数的方法：

```py
from typing import List, NamedTuple, TextIO

class Args(NamedTuple): ![1](assets/1.png)
    """ Command-line arguments """
    file: TextIO
    k: int
    debug: bool

# --------------------------------------------------
def get_args() -> Args:
    """ Get command-line arguments """

    parser = argparse.ArgumentParser(
        description='Overlap Graphs',
        formatter_class=argparse.ArgumentDefaultsHelpFormatter)

    parser.add_argument('file', ![2](assets/2.png)
                        metavar='FILE',
                        type=argparse.FileType('rt'),
                        help='FASTA file')

    parser.add_argument('-k', ![3](assets/3.png)
                        '--overlap',
                        help='Size of overlap',
                        metavar='size',
                        type=int,
                        default=3)

    parser.add_argument('-d', '--debug', help='Debug', action='store_true') ![4](assets/4.png)

    args = parser.parse_args()

    if args.overlap < 1: ![5](assets/5.png)
        parser.error(f'-k "{args.overlap}" must be > 0') ![6](assets/6.png)

    return Args(args.file, args.overlap, args.debug) ![7](assets/7.png)
```

[![1](assets/1.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO2-1)

`Args` 类包含三个字段：一个是文件句柄 `file`；一个是应为正整数的 `k`；还有一个是布尔值 `debug`。

[![2](assets/2.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO2-2)

使用 `argparse.FileType` 确保这是一个可读的文本文件。

[![3](assets/3.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO2-3)

定义一个默认为 `3` 的整数参数。

[![4](assets/4.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO2-4)

定义一个布尔标志，当存在时将存储一个 `True` 值。

[![5](assets/5.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO2-5)

检查 `k`（重叠）值是否为负数。

[![6](assets/6.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO2-6)

使用 `parser.error()` 终止程序并生成一个有用的错误消息。

[![7](assets/7.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO2-7)

返回经过验证的参数。

我想强调这些行中发生了多少事情，以确保程序的参数是正确的。参数值应该在程序启动后尽快验证。我遇到过太多的程序，例如从不验证文件参数，然后在程序深处尝试打开一个不存在的文件，结果抛出一个晦涩难解的异常，没有人能够调试。如果你想要*可重现*的程序，第一要务是记录和验证所有的参数。

修改你的 `main()` 如下：

```py
def main() -> None:
    args = get_args()
    print(args.file.name)
```

运行你的程序与第一个测试输入文件，并验证你是否看到了这个：

```py
$ ./grph.py tests/inputs/1.fa
tests/inputs/1.fa
```

尝试用无效的`k`值和文件输入运行你的程序，然后运行`**pytest**`来验证你的程序是否通过了前四个测试。失败的测试期望有三对可以连接的序列ID，但程序输出了输入文件的名称。在我讲解如何创建重叠图之前，我想先介绍一下*日志记录*，因为这对于调试程序可能会很有用。

## 使用STDOUT、STDERR和日志管理运行时消息

我已经展示了如何将字符串和数据结构打印到控制台。你刚刚通过打印输入文件名来验证程序是否正常工作。在编写和调试程序时打印这样的消息可能会被戏称为*基于日志的开发*。这是几十年来调试程序的一种简单有效的方法。^([1](ch09.html#idm45963633006072))

默认情况下，`print()`会将消息发送到`STDOUT`（*标准输出*），Python使用`sys.stdout`来表示。我可以使用`print()`函数的`file`选项将其更改为`STDERR`（*标准错误*），方法是指定`sys.stderr`。考虑以下Python程序：

```py
$ cat log.py
#!/usr/bin/env python3

import sys

print('This is STDOUT.') ![1](assets/1.png)
print('This is also STDOUT.', file=sys.stdout) ![2](assets/2.png)
print('This is STDERR.', file=sys.stderr) ![3](assets/3.png)
```

[![1](assets/1.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO3-1)

默认的`file`值是`STDOUT`。

[![2](assets/2.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO3-2)

我可以使用`file`选项来指定标准输出。

[![3](assets/3.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO3-3)

这将会将消息打印到标准错误。

当我运行时，所有输出似乎都打印到标准输出：

```py
$ ./log.py
This is STDOUT.
This is also STDOUT.
This is STDERR.
```

在`bash` shell中，我可以使用文件重定向和`>`来分隔和捕获这两个流。标准输出可以使用文件句柄1捕获，标准错误可以使用2捕获。如果你运行以下命令，你应该在控制台上看不到任何输出：

```py
$ ./log.py 1>out 2>err
```

现在应该有两个新文件，一个名为*out*，其中包含打印到标准输出的两行：

```py
$ cat out
This is STDOUT.
This is also STDOUT.
```

另一个称为*err*的文件，其中只有一行打印到标准错误输出：

```py
$ cat err
This is STDERR.
```

仅了解如何打印和捕获这两个文件句柄可能足以满足你的调试需求。然而，有时候你可能需要比两个更多级别的打印，并且你可能希望通过代码控制这些消息被写入的位置，而不是使用shell重定向。这就是*日志记录*的作用，一种控制运行时消息何时、如何、何地打印的方法。Python的`logging`模块处理所有这些，所以首先导入该模块：

```py
import logging
```

对于这个程序，如果存在`--debug`标志，我将调试消息打印到一个名为*.log*的文件中（在当前工作目录中）。将你的`main()`修改为以下内容：

```py
def main() -> None:
    args = get_args()

    logging.basicConfig( ![1](assets/1.png)
        filename='.log', ![2](assets/2.png)
        filemode='w', ![3](assets/3.png)
        level=logging.DEBUG if args.debug else logging.CRITICAL) ![4](assets/4.png)

    logging.debug('input file = "%s"', args.file.name) ![5](assets/5.png)
```

[![1](assets/1.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO4-1)

这将*全局*影响所有后续对`logging`模块函数的调用。

[![2](assets/2.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO4-2)

所有输出将写入当前工作目录中的*.log*文件。我选择了以点开头的文件名，因此通常不会显示在视图中。

[![3](assets/3.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO4-3)

输出文件将使用`w`（*写入*）选项打开，意味着每次运行时都将*覆盖*它。使用`a`模式来*追加*，但要注意每次运行文件都会增长，除非您手动截断或删除它。

[![4](assets/4.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO4-4)

这设置了最低的日志记录级别（参见[表 9-1](#table_9.1)）。低于设置级别的消息将被忽略。

[![5](assets/5.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO4-5)

使用`logging.debug()`函数在日志级别设置为`DEBUG`或更高时将消息打印到日志文件中。

在上一个示例中，我使用了旧式的`printf()`格式化风格来调用`logging.debug()`。占位符用符号如`%s`表示字符串，需要替换的值作为参数传递。您也可以使用`str.format()`和f-strings进行日志消息格式化，但`pylint`可能建议您使用`printf()`风格。

日志记录的一个关键概念是日志级别的概念。如[表 9-1](#table_9.1)所示，*严重*级别是最高的，*调试*级别是最低的（*未设置*级别具有特定特性）。要了解更多信息，建议您在REPL中阅读**`help(logging)`**或访问[模块的在线文档](https://oreil.ly/bWgOp)。对于此程序，我只会使用最低的（调试）设置。当存在`--debug`标志时，日志级别设置为`logging.DEBUG`，所有通过`logging.debug()`记录的消息都会打印在日志文件中。当标志不存在时，日志级别设置为`logging.CRITICAL`，只有通过`logging.critical()`记录的消息才会通过。您可能认为我应该使用`logging.NOTSET`值，但请注意，这比`logging.DEBUG`低，因此所有调试消息都会通过。

表 9-1\. Python的`logging`模块中可用的日志级别

| 等级 | 数值 |
| --- | --- |
| 严重 | 50 |
| 错误 | 40 |
| 警告 | 30 |
| 信息 | 20 |
| 调试 | 10 |
| 未设置 | 0 |

要查看实际操作，请按以下步骤运行您的程序：

```py
$ ./grph.py --debug tests/inputs/1.fa
```

似乎程序什么都没做，但现在应该有一个*.log*文件，内容如下：

```py
$ cat .log
DEBUG:root:input file = "tests/inputs/1.fa"
```

再次运行程序，但不带`--debug`标志，并注意*.log*文件为空，因为它在打开时被覆盖，但从未记录任何内容。如果您使用典型的基于打印的调试技术，则必须找到并删除（或注释掉）程序中的所有`print()`语句以关闭调试。相反，如果您使用`logging.debug()`，则可以在调试级别记录并部署程序以仅记录关键消息。此外，您可以根据环境将日志消息写入不同的位置，所有这些操作都是*程序化*地在您的代码中完成，而不是依赖于shell重定向将日志消息放入正确的位置。

没有测试来确保您的程序创建日志文件。这只是向您展示如何使用日志记录。请注意，对`logging.critical()`和`logging.debug()`等函数的调用由`logging`模块的*全局*范围控制。我通常不喜欢程序受全局设置控制，但这是我唯一的例外，主要是因为我别无选择。我鼓励您在代码中大量使用`logging.debug()`调用，以查看您可以生成的各种输出。考虑在您在笔记本电脑上编写程序和在远程计算集群上部署它以无人值守运行时如何使用日志记录。

## 寻找重叠部分

接下来的任务是读取输入的FASTA文件。我首先在[第5章](ch05.html#ch05)展示了如何做到这一点。再次，我将使用`Bio.SeqIO`模块通过添加以下导入来完成：

```py
from Bio import SeqIO
```

我可以将`main()`修改为以下内容（省略任何日志调用）：

```py
def main() -> None:
    args = get_args()

    for rec in SeqIO.parse(args.file, 'fasta'):
        print(rec.id, rec.seq)
```

然后在第一个输入文件上运行以确保程序正常工作：

```py
$ ./grph.py tests/inputs/1.fa
Rosalind_0498 AAATAAA
Rosalind_2391 AAATTTT
Rosalind_2323 TTTTCCC
Rosalind_0442 AAATCCC
Rosalind_5013 GGGTGGG
```

在每个练习中，我尝试逐步逻辑地展示如何编写程序。我希望您学会进行非常小的更改以达到某个最终目标，然后运行您的程序以查看输出。您应该经常运行测试来查看需要修复的问题，并根据需要添加自己的测试。此外，请考虑在程序运行良好时频繁提交程序，以便在破坏程序时可以还原。采取小步骤并经常运行您的程序是学习编程的关键要素。

现在考虑如何从每个序列中获取第一个和最后一个*k*个碱基。您可以使用我在[第7章](ch07.html#ch07)中首次展示的提取k-mer的代码吗？例如，尝试让您的程序打印出这个：

```py
$ ./grph.py tests/inputs/1.fa
Rosalind_0498 AAATAAA first AAA last AAA
Rosalind_2391 AAATTTT first AAA last TTT
Rosalind_2323 TTTTCCC first TTT last CCC
Rosalind_0442 AAATCCC first AAA last CCC
Rosalind_5013 GGGTGGG first GGG last GGG
```

想想哪些*起始*字符串匹配哪些*结束*字符串。例如，序列 0498 以*AAA*结尾，而序列 0442 以*AAA*开头。这些序列可以连接成重叠图。

将`k`的值更改为`4`：

```py
$ ./grph.py tests/inputs/1.fa -k 4
Rosalind_0498 AAATAAA first AAAT last TAAA
Rosalind_2391 AAATTTT first AAAT last TTTT
Rosalind_2323 TTTTCCC first TTTT last TCCC
Rosalind_0442 AAATCCC first AAAT last TCCC
Rosalind_5013 GGGTGGG first GGGT last TGGG
```

现在你可以看到，只有两个序列，2391和2323，可以通过它们的重叠序列*TTTT*连接起来。从`1`到`10`变化`k`并检查第一个和最后一个区域。你有足够的信息来编写解决方案吗？如果没有，让我们继续思考这个问题。

## 通过重叠来分组序列

`for`循环逐个读取序列。在读取任何一个序列以查找起始和结束重叠区域时，我必须没有足够的信息来说哪些其他序列可以连接起来。我将不得不创建一些数据结构来保存*所有*序列的重叠区域。只有这样我才能回过头来弄清楚哪些序列可以连接起来。这涉及到序列组装器的关键元素——大多数需要大量内存来收集来自所有输入序列的所有所需信息，这可能会有数百万到数十亿。

我选择使用两个字典，一个用于*起始*区域，一个用于*结束*区域。我决定键将是*k*长度的序列，例如在`k`为`3`时为*AAA*，而值将是共享此区域的序列ID列表。我可以使用值`k`的字符串切片来提取这些前导和尾随序列：

```py
>>> k = 3
>>> seq = 'AAATTTT'
>>> seq[:k] ![1](assets/1.png)
'AAA'
>>> seq[-k:] ![2](assets/2.png)
'TTT'
```

[![1](assets/1.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO5-1)

取前`k`个碱基片段。

[![2](assets/2.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO5-2)

取最后`k`个碱基片段，使用负索引从序列末尾开始。

这些是k-mer，在上一章中我用过。它们一直出现，因此编写一个`find_kmers()`函数来从序列中提取k-mer是合理的。我将从定义函数的签名开始：

```py
def find_kmers(seq: str, k: int) -> List[str]: ![1](assets/1.png)
    """ Find k-mers in string """

    return [] ![2](assets/2.png)
```

[![1](assets/1.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO6-1)

此函数将接受一个字符串（序列）和一个整数值`k`，并返回一个字符串列表。

[![2](assets/2.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO6-2)

目前返回空列表。

现在我写一个测试来想象如何使用这个函数：

```py
def test_find_kmers() -> None:
    """Test find_kmers"""

    assert find_kmers('', 1) == [] ![1](assets/1.png)
    assert find_kmers('ACTG', 1) == ['A', 'C', 'T', 'G'] ![2](assets/2.png)
    assert find_kmers('ACTG', 2) == ['AC', 'CT', 'TG']
    assert find_kmers('ACTG', 3) == ['ACT', 'CTG']
    assert find_kmers('ACTG', 4) == ['ACTG']
    assert find_kmers('ACTG', 5) == [] ![3](assets/3.png)
```

[![1](assets/1.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO7-1)

将空字符串作为序列传递，以确保函数返回空列表。

[![2](assets/2.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO7-2)

检查所有使用短序列的`k`值。

[![3](assets/3.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO7-3)

一个长度为4的字符串没有5-mers。

在继续阅读之前尝试编写你自己的版本。这是我写的函数：

```py
def find_kmers(seq: str, k: int) -> List[str]:
    """Find k-mers in string"""

    n = len(seq) - k + 1 ![1](assets/1.png)
    return [] if n < 1 else [seq[i:i + k] for i in range(n)] ![2](assets/2.png)
```

[![1](assets/1.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO8-1)

找到字符串 `seq` 中 *k* 长度子字符串的数量 `n`。

[![2](assets/2.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO8-2)

如果 `n` 是负数，返回空列表；否则，使用列表推导返回 k-mer。

现在我有了从序列中获取前导和尾随 k-mer 的便利方法：

```py
>>> from grph import find_kmers
>>> kmers = find_kmers('AAATTTT', 3)
>>> kmers
['AAA', 'AAT', 'ATT', 'TTT', 'TTT']
>>> kmers[0] ![1](assets/1.png)
'AAA'
>>> kmers[-1] ![2](assets/2.png)
'TTT'
```

[![1](assets/1.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO9-1)

第一个元素是前导 k-mer。

[![2](assets/2.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO9-2)

最后一个元素是尾随 k-mer。

第一个和最后一个 k-mer 给我所需的键的重叠序列。我希望字典的值是共享这些 k-mer 的序列 ID 列表。我在 [第一章](ch01.html#ch01) 中介绍的 `collections.defaultdict()` 函数很适合用于此，因为它允许我轻松地用空列表实例化每个字典条目。为了日志记录目的，我需要导入它和 `pprint.pformat()` 函数，所以我添加如下内容：

```py
from collections import defaultdict
from pprint import pformat
```

这里是我如何应用这些想法：

```py
def main() -> None:
    args = get_args()

    logging.basicConfig(
        filename='.log',
        filemode='w',
        level=logging.DEBUG if args.debug else logging.CRITICAL)

    start, end = defaultdict(list), defaultdict(list) ![1](assets/1.png)
    for rec in SeqIO.parse(args.file, 'fasta'): ![2](assets/2.png)
        if kmers := find_kmers(str(rec.seq), args.k): ![3](assets/3.png)
            start[kmers[0]].append(rec.id) ![4](assets/4.png)
            end[kmers[-1]].append(rec.id) ![5](assets/5.png)

    logging.debug(f'STARTS\n{pformat(start)}') ![6](assets/6.png)
    logging.debug(f'ENDS\n{pformat(end)}')
```

[![1](assets/1.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO10-1)

创建起始和结束区域的字典，其默认值为列表。

[![2](assets/2.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO10-2)

迭代 FASTA 记录。

[![3](assets/3.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO10-3)

强制 `Seq` 对象转换为字符串并找到 k-mer。`:=` 语法将返回值分配给 `kmers`，然后 `if` 评估 `kmers` 是否为真值。如果函数没有返回 kmers，则以下块不会执行。

[![4](assets/4.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO10-4)

使用第一个 k-mer 作为 `start` 字典的键，并将此序列 ID 添加到列表中。

[![5](assets/5.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO10-5)

类似地，使用最后一个 k-mer 对 `end` 字典进行操作。

[![6](assets/6.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO10-6)

使用 `pprint.pformat()` 函数为日志格式化字典。

我之前在较早的章节中使用了 `pprint.pprint()` 函数来打印复杂的数据结构，以比默认的 `print()` 函数更漂亮的格式输出。我不能在这里使用 `pprint()`，因为它会打印到 `STDOUT`（或 `STDERR`）。相反，我需要为 `logging.debug()` 函数格式化数据结构以进行日志记录。

现在再次运行程序，使用第一个输入和 `--debug` 标志，然后检查日志文件：

```py
$ ./grph.py tests/inputs/1.fa -d
$ cat .log
DEBUG:root:STARTS ![1](assets/1.png)
defaultdict(<class 'list'>,
            {'AAA': ['Rosalind_0498', 'Rosalind_2391', 'Rosalind_0442'], ![2](assets/2.png)
             'GGG': ['Rosalind_5013'],
             'TTT': ['Rosalind_2323']})
DEBUG:root:ENDS ![3](assets/3.png)
defaultdict(<class 'list'>,
            {'AAA': ['Rosalind_0498'], ![4](assets/4.png)
             'CCC': ['Rosalind_2323', 'Rosalind_0442'],
             'GGG': ['Rosalind_5013'],
             'TTT': ['Rosalind_2391']})
```

[![1](assets/1.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO11-1)

各种起始序列和其 ID 的字典。

[![2](assets/2.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO11-2)

三个序列以 *AAA* 开始：0498、2391 和 0442。

[![3](assets/3.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO11-3)

各种结束序列及其 ID 的字典。

[![4](assets/4.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO11-4)

仅有一个以 *AAA* 结尾的序列，即 0498。

对于这个输入文件和重叠的 3-mers，正确的序对如下：

+   Rosalind_0498, Rosalind_2391: *AAA*

+   Rosalind_0498, Rosalind_0442: *AAA*

+   Rosalind_2391, Rosalind_2323: *TTT*

例如，当你将以 *AAA* 结尾的序列（0498）与以该序列开头的序列（0498、2391、0442）结合时，你会得到以下配对：

+   Rosalind_0498, Rosalind_0498

+   Rosalind_0498, Rosalind_2391

+   Rosalind_0498, Rosalind_0442

由于我无法将一个序列与自身连接，所以第一对被取消资格。找到下一个共同的 *end* 和 *start* 序列，然后迭代所有序列对。我留给你完成这个练习，找出所有共同的起始和结束键，然后组合所有序列 ID 以打印可以连接的序对。序对可以以任何顺序排列仍然通过测试。我只想祝你好运。我们都在依赖你。

# 解决方案

我有两种变体与你分享。第一种解决了 Rosalind 问题，展示了如何组合任意两个序列。第二种扩展了图形以创建所有序列的完整组装。

## 解决方案 1：使用集合交集查找重叠部分

在以下解决方案中，我介绍如何使用集合交集来找到起始和结束字典之间共享的 k-mer：

```py
def main() -> None:
    args = get_args()

    logging.basicConfig(
        filename='.log',
        filemode='w',
        level=logging.DEBUG if args.debug else logging.CRITICAL)

    start, end = defaultdict(list), defaultdict(list)
    for rec in SeqIO.parse(args.file, 'fasta'):
        if kmers := find_kmers(str(rec.seq), args.k):
            start[kmers[0]].append(rec.id)
            end[kmers[-1]].append(rec.id)

    logging.debug('STARTS\n{}'.format(pformat(start)))
    logging.debug('ENDS\n{}'.format(pformat(end)))

    for kmer in set(start).intersection(set(end)): ![1](assets/1.png)
        for pair in starfilter(op.ne, product(end[kmer], start[kmer])): ![2](assets/2.png)
            print(*pair) ![3](assets/3.png)
```

[![1](assets/1.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO12-1)

找到 `start` 和 `end` 字典之间的公共键。

[![2](assets/2.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO12-2)

迭代不相等的结束和起始序列对。

[![3](assets/3.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO12-3)

打印序列对。

最后三行让我尝试了几次才写出来，所以让我解释一下我是如何做到的。鉴于这些字典：

```py
>>> from pprint import pprint
>>> from Bio import SeqIO
>>> from collections import defaultdict
>>> from grph import find_kmers
>>> k = 3
>>> start, end = defaultdict(list), defaultdict(list)
>>> for rec in SeqIO.parse('tests/inputs/1.fa', 'fasta'):
...     if kmers := find_kmers(str(rec.seq), k):
...         start[kmers[0]].append(rec.id)
...         end[kmers[-1]].append(rec.id)
...
>>> pprint(start)
{'AAA': ['Rosalind_0498', 'Rosalind_2391', 'Rosalind_0442'],
 'GGG': ['Rosalind_5013'],
 'TTT': ['Rosalind_2323']}
>>> pprint(end)
{'AAA': ['Rosalind_0498'],
 'CCC': ['Rosalind_2323', 'Rosalind_0442'],
 'GGG': ['Rosalind_5013'],
```

我从这个想法开始：

```py
>>> for kmer in end: ![1](assets/1.png)
...     if kmer in start: ![2](assets/2.png)
...         for seq_id in end[kmer]: ![3](assets/3.png)
...             for other in start[kmer]: ![4](assets/4.png)
...                 if seq_id != other: ![5](assets/5.png)
...                     print(seq_id, other) ![6](assets/6.png)
...
Rosalind_0498 Rosalind_2391
Rosalind_0498 Rosalind_0442
Rosalind_2391 Rosalind_2323
```

[![1](assets/1.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO13-1)

迭代 `end` 字典的 k-mer（即 *keys*）。

[![2](assets/2.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO13-2)

检查这个 k-mer 是否在`start`字典中。

[![3](assets/3.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO13-3)

遍历每个 k-mer 的结束序列 ID。

[![4](assets/4.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO13-4)

遍历每个 k-mer 的起始序列 ID。

[![5](assets/5.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO13-5)

确保序列不相同。

[![6](assets/6.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO13-6)

打印序列 ID。

虽然这很有效，但我让它静置一段时间，然后回来，问自己我究竟想要做什么。前两行试图找到两个字典之间共同的键。使用集合交集可以更轻松地实现这一点。如果我在字典上使用`set()`函数，它将使用字典的键创建一个集合：

```py
>>> set(start)
{'TTT', 'GGG', 'AAA'}
>>> set(end)
{'TTT', 'CCC', 'AAA', 'GGG'}
```

然后，我可以调用`set.intersection()`函数来查找共同的键：

```py
>>> set(start).intersection(set(end))
{'TTT', 'GGG', 'AAA'}
```

在上面的代码中，下一行找到所有结束和起始序列 ID 的组合。使用`itertools.product()`函数可以更容易地完成这个任务，它将创建任意数量列表的笛卡尔积。例如，考虑在 k-mer *AAA* 上重叠的序列：

```py
>>> from itertools import product
>>> kmer = 'AAA'
>>> pairs = list(product(end[kmer], start[kmer]))
>>> pprint(pairs)
[('Rosalind_0498', 'Rosalind_0498'),
 ('Rosalind_0498', 'Rosalind_2391'),
 ('Rosalind_0498', 'Rosalind_0442')]
```

我想排除任何两个值相同的对。我可以为此编写一个`filter()`：

```py
>>> list(filter(lambda p: p[0] != p[1], pairs)) ![1](assets/1.png)
[('Rosalind_0498', 'Rosalind_2391'), ('Rosalind_0498', 'Rosalind_0442')]
```

[![1](assets/1.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO14-1)

`lambda`接收对`p`并检查零th和第一个元素不相等。

这个方法可以，但我对这段代码不满意。我真的很讨厌不能在`lambda`中解包元组值。我立即开始思考在第[6](ch06.html#ch06)和第[8](ch08.html#ch08)章中使用的`itertools.starmap()`函数可以做到这一点，所以我搜索了互联网找到了`Python starfilter`函数[`iteration_utilities.starfilter()`](https://oreil.ly/c6KKV)。我安装了这个模块并导入了该函数：

```py
>>> from iteration_utilities import starfilter
>>> list(starfilter(lambda a, b: a != b, pairs))
[('Rosalind_0498', 'Rosalind_2391'), ('Rosalind_0498', 'Rosalind_0442')]
```

这是一个改进，但我可以通过使用`operator.ne()`（不相等）函数来使它更清晰，这将消除`lambda`：

```py
>>> import operator as op
>>> list(starfilter(op.ne, pairs))
[('Rosalind_0498', 'Rosalind_2391'), ('Rosalind_0498', 'Rosalind_0442')]
```

最后，我展开每个对，使`print()`看到单独的字符串而不是列表容器：

```py
>>> for pair in starfilter(op.ne, pairs):
...     print(*pair)
...
Rosalind_0498 Rosalind_2391
Rosalind_0498 Rosalind_0442
```

我本可以进一步缩短这段代码，但我担心这会变得太密集：

```py
>>> print('\n'.join(map(' '.join, starfilter(op.ne, pairs))))
Rosalind_0498 Rosalind_2391
Rosalind_0498 Rosalind_0442
```

最后，在`main()`函数中有相当多的代码，在一个更大的程序中，我可能会将其移到一个带有单元测试的函数中。在这种情况下，集成测试覆盖了所有功能，所以这样做可能过于复杂。

## 解决方案2：使用图形查找所有路径

下一个解决方案通过图形近似地装配所有序列，以链接所有重叠的序列。虽然不是原始挑战的一部分，但思考起来非常有趣，而且实现起来甚至很简单。由于*GRPH*是挑战名称，探索如何在Python代码中表示图形是有意义的。

我可以手动对齐所有序列，如[图9-3](#fig_9.3)所示。这揭示了一个图结构，其中序列 Rosalind_0498 可以连接到 Rosalind_2391 或 Rosalind_0442，然后从 Rosalind_0498 到 Rosalind_2391 到 Rosalind_2323 形成链。

![mpfb 0903](assets/mpfb_0903.png)

###### 图9-3。第一个输入文件中的所有序列都可以使用3-mer连接。

为了编码这一点，我使用[Graphviz工具](https://graphviz.org)来表示和可视化图形结构。请注意，您需要在计算机上安装Graphviz才能使用此功能。例如，在macOS上，您可以使用Homebrew软件包管理器（**`brew install graphviz`**），而在Ubuntu Linux上，您可以使用**`apt install graphviz`**。

Graphviz 的输出将是一个文本文件，格式为[Dot语言](https://graphviz.org/doc/info/lang.html)，可以通过Graphviz的`dot`工具转换为图形图。仓库中的第二个解决方案具有控制输出文件名和是否打开图像的选项：

```py
$ ./solution2_graph.py -h
usage: solution2_graph.py [-h] [-k size] [-o FILE] [-v] [-d] FILE

Overlap Graphs

positional arguments:
  FILE                  FASTA file

optional arguments:
  -h, --help            show this help message and exit
  -k size, --overlap size
                        Size of overlap (default: 3)
  -o FILE, --outfile FILE
                        Output filename (default: graph.txt) ![1](assets/1.png)
  -v, --view            View outfile (default: False) ![2](assets/2.png)
  -d, --debug           Debug (default: False)
```

[![1](assets/1.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO15-1)

默认的输出文件名是*graph.txt*。还会自动生成一个*.pdf*文件，这是图形的可视化呈现。

[![2](assets/2.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO15-2)

此选项控制程序完成后是否自动打开PDF。

如果您在第一个测试输入上运行此程序，您将看到与之前相同的输出，以便通过测试套件：

```py
$ ./solution2_graph.py tests/inputs/1.fa -o 1.txt
Rosalind_2391 Rosalind_2323
Rosalind_0498 Rosalind_2391
Rosalind_0498 Rosalind_0442
```

现在应该还有一个新的输出文件叫做*1.txt*，其中包含使用Dot语言编码的图结构：

```py
$ cat 1.txt
digraph {
	Rosalind_0498
	Rosalind_2391
	Rosalind_0498 -> Rosalind_2391
	Rosalind_0498
	Rosalind_0442
	Rosalind_0498 -> Rosalind_0442
	Rosalind_2391
	Rosalind_2323
	Rosalind_2391 -> Rosalind_2323
}
```

您可以使用`dot`程序将其转换为可视化。以下是一个将图形保存为PNG文件的命令：

```py
$ dot -O -Tpng 1.txt
```

[图9-4](#fig_9.4)展示了连接第一个FASTA文件中所有序列的图形的结果可视化，重新概括了从[图9-3](#fig_9.3)的手动对齐。

![mpfb 0904](assets/mpfb_0904.png)

###### 图9-4。`dot`程序的输出，显示了在连接3-mer时第一个输入文件中序列的装配。

如果您使用`-v|--view`标志运行程序，此图像应该会自动显示。在图的术语中，每个序列是一个*节点*，两个序列之间的关系是一条*边*。

图可能具有方向性，也可能没有。[图 9-4](#fig_9.4) 包括箭头，表明存在从一个节点到另一个节点的关系；因此，这是一个*有向图*。以下代码显示了我如何创建和可视化此图。请注意，我导入 `graphiz.Digraph` 来创建有向图，而此代码省略了实际解决方案中的日志记录代码：

```py
def main() -> None:
    args = get_args()
    start, end = defaultdict(list), defaultdict(list)
    for rec in SeqIO.parse(args.file, 'fasta'):
        if kmers := find_kmers(str(rec.seq), args.k):
            start[kmers[0]].append(rec.id)
            end[kmers[-1]].append(rec.id)

    dot = Digraph() ![1](assets/1.png)
    for kmer in set(start).intersection(set(end)): ![2](assets/2.png)
        for s1, s2 in starfilter(op.ne, product(end[kmer], start[kmer])): ![3](assets/3.png)
            print(s1, s2) ![4](assets/4.png)
            dot.node(s1) ![5](assets/5.png)
            dot.node(s2)
            dot.edge(s1, s2) ![6](assets/6.png)

    args.outfile.close() ![7](assets/7.png)
    dot.render(args.outfile.name, view=args.view) ![8](assets/8.png)
```

[![1](assets/1.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO16-1)

创建一个有向图。

[![2](assets/2.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO16-2)

迭代共享的 k-mer。

[![3](assets/3.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO16-3)

查找共享 k-mer 的序列对，并将两个序列 ID 解包为 `s1` 和 `s2`。

[![4](assets/4.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO16-4)

打印测试的输出。

[![5](assets/5.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO16-5)

为每个序列添加节点。

[![6](assets/6.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO16-6)

添加连接节点的边缘。

[![7](assets/7.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO16-7)

关闭输出文件句柄，以便图形可以写入文件名。

[![8](assets/8.png)](#co_overlap_graphs__sequence_assembly__span_class__keep_together__using_shared_k_mers__span__CO16-8)

将图结构写入输出文件名。根据 `args.view` 选项，使用 `view` 选项打开图像。

这几行代码对程序的输出有很大影响。例如，[图 9-5](#fig_9.5) 显示，该程序基本上可以创建第二个输入文件中 100 个序列的完整组装。

![mpfb 0905](assets/mpfb_0905.png)

###### 图 9-5\. 第二个测试输入文件的图形

这张图像（虽然已缩小以适合页面）充分展示了数据的复杂性和完整性；例如，右上角的序列对——Rosalind_1144 和 Rosalind_2208——无法与任何其他序列连接。我鼓励您尝试将 `k` 增加到 `4` 并检查生成的图形，看看会有非常不同的结果。

图是真正强大的数据结构。正如本章介绍中指出的，图编码了成对关系。通过如此少量的代码看到 [图 9-5](#fig_9.5) 中 100 个序列的组装是令人惊讶的。虽然可能滥用 Python 列表和字典来表示图，但 Graphviz 工具使这一切变得更简单。

我在这个练习中使用了有向图，但这不一定是必需的。这也可以是无向图，但我喜欢箭头。我要注意的是，你可能会遇到术语 *有向无环图* (DAG)，表示没有循环的有向图，当一个节点指向自身时则形成循环。在线性基因组的情况下，循环可能指向不正确的组装，但在细菌的环形基因组中可能是必需的。如果你对这些想法感兴趣，应该调查一下 De Bruijn 图，它们通常由重叠的 k-mer 构建。

# 进一步探索

添加一个汉明距离选项，允许重叠序列具有指定的编辑距离。也就是说，距离为 1 允许具有单个碱基差异的序列重叠。

# 复习

本章要点：

+   要找到重叠区域，我使用了 k-mer 来找到每个序列的前 *k* 个和最后 *k* 个碱基。

+   `logging` 模块使得可以轻松地控制运行时消息的记录与关闭。

+   我使用 `defaultdict(list)` 创建了一个字典，当键不存在时会自动创建一个空列表作为默认值。

+   集合交集可以找到两个字典之间共享的键等公共元素。

+   `itertools.product()` 函数找到了所有可能的序列对。

+   `iteration_utilities.starfilter()` 函数将会解构 `filter()` 的参数，就像 `itertools.starmap()` 函数对 `map()` 一样。

+   Graphviz 工具能高效地表示和可视化复杂的图结构。

+   图可以用 Dot 语言进行文本表示，而 `dot` 程序可以生成各种格式的图形化显示。

+   重叠图可用于创建两个或多个序列的完整组装。

^([1](ch09.html#idm45963633006072-marker)) 想象一下在没有控制台的情况下调试程序。在1950年代，克劳德·香农访问了艾伦·图灵在英国实验室的实验室。在他们的对话期间，一只喇叭开始定期发声。图灵说这表明他的代码陷入了循环。在没有控制台的情况下，这是他监视程序进展的方式。
