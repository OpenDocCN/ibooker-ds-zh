# 第15章。Seqmagique：创建和格式化报告

在生物信息学项目中，你经常会发现自己盯着一个目录，里面充满了序列文件，通常是 FASTA 或 FASTQ 格式。你可能希望首先了解文件中序列的分布情况，例如每个文件中有多少序列以及序列的平均长度、最小长度和最大长度。你需要知道是否有任何文件损坏了，也许它们从测序中心传输时没有完全完成，或者是否有任何样本读数远低，可能表明需要重新进行糟糕的测序运行。在本章中，我将介绍使用哈希检查序列文件的一些技术，并编写一个小型实用程序来模拟 Seqmagick 的部分功能，以说明如何创建格式化的文本表格。这个程序可以作为处理给定文件集中所有记录并生成汇总统计表格的任何程序的模板。

你将学到：

+   如何安装 `seqmagick` 工具

+   如何使用 MD5 哈希

+   如何在 `argparse` 中使用 `choices` 限制参数

+   如何使用 `numpy` 模块

+   如何模拟文件句柄

+   如何使用 `tabulate` 和 `rich` 模块来格式化输出表格

# 使用 Seqmagick 分析序列文件

`seqmagick` 是一个处理序列文件的实用命令行工具。如果按照前言中的设置说明进行安装，它应该已经和其他 Python 模块一起安装好了。如果没有安装，你可以使用 `pip` 安装它：

```py
$ python3 -m pip install seqmagick
```

如果你运行 **`seqmagick --help`**，你会看到这个工具提供了许多选项。我只想关注 `info` 子命令。我可以在 *15_seqmagique* 目录中的测试输入 FASTA 文件上运行它，如下所示：

```py
$ cd 15_seqmagique
$ seqmagick info tests/inputs/*.fa
name                  alignment    min_len   max_len   avg_len  num_seqs
tests/inputs/1.fa     FALSE             50        50     50.00         1
tests/inputs/2.fa     FALSE             49        79     64.00         5
tests/inputs/empty.fa FALSE              0         0      0.00         0
```

在这个练习中，你将创建一个名为 `seqmagique.py` 的程序（应该用夸张的法语口音发音），它将模拟这个输出。该程序的目的是提供给定文件集中序列的基本概述，以便你可以发现例如截断或损坏的文件。

首先将解决方案复制到 `seqmagique.py` 并请求使用说明：

```py
$ cp solution1.py seqmagique.py
$ ./seqmagique.py -h
usage: seqmagique.py [-h] [-t table] FILE [FILE ...]

Mimic seqmagick

positional arguments:
  FILE                  Input FASTA file(s) ![1](assets/1.png)

optional arguments:
  -h, --help            show this help message and exit
  -t table, --tablefmt table ![2](assets/2.png)
                        Tabulate table style (default: plain)
```

[![1](assets/1.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO1-1)

程序接受一个或多个应该是 FASTA 格式的输入文件。

[![2](assets/2.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO1-2)

这个选项控制输出表格的格式。

在相同的文件上运行这个程序，并注意输出几乎相同，只是省略了 `alignment` 列：

```py
$ ./seqmagique.py tests/inputs/*.fa
name                     min_len    max_len    avg_len    num_seqs
tests/inputs/1.fa             50         50      50.00           1
tests/inputs/2.fa             49         79      64.00           5
tests/inputs/empty.fa          0          0       0.00           0
```

`--tablefmt` 选项控制输出表格的格式。这是你将编写的第一个程序，它限制了给定列表中的值。要看看它如何运行，请使用一个像 `blargh` 这样的虚假值：

```py
$ ./seqmagique.py -t blargh tests/inputs/1.fa
usage: seqmagique.py [-h] [-t table] FILE [FILE ...]
seqmagique.py: error: argument -t/--tablefmt: invalid choice: 'blargh'
(choose from 'plain', 'simple', 'grid', 'pipe', 'orgtbl', 'rst',
 'mediawiki', 'latex', 'latex_raw', 'latex_booktabs')
```

然后尝试不同的表格格式，例如 `simple`：

```py
$ ./seqmagique.py -t simple tests/inputs/*.fa
name                     min_len    max_len    avg_len    num_seqs
---------------------  ---------  ---------  ---------  ----------
tests/inputs/1.fa             50         50      50.00           1
tests/inputs/2.fa             49         79      64.00           5
tests/inputs/empty.fa          0          0       0.00           0
```

运行程序与其他表格样式，然后尝试测试套件。接下来，我将讨论如何获取我们程序分析的数据。

# 使用 MD5 哈希检查文件

大多数基因组学项目的第一步将是将序列文件传输到某个可以分析它们的位置，并且防止数据损坏的第一道防线是确保文件完整复制。文件的来源可能是测序中心或像[GenBank](https://oreil.ly/2eaMj)或[Sequence Read Archive (SRA)](https://oreil.ly/kGNCv)这样的公共存储库。文件可能会通过存储卡送达，或者你可以从互联网下载它们。如果是后者，你可能会发现你的连接中断，导致一些文件被截断或损坏。如何找到这些类型的错误？

一个检查文件完整性的方法是在本地与服务器上的文件大小进行比较。例如，你可以使用**`ls -l`**命令查看文件的*长*列表，其中显示以字节为单位的文件大小。对于大型序列文件，这将是一个非常大的数字，你将不得不手动比较源和目的地的文件大小，这很繁琐且容易出错。

另一种技术涉及使用文件的*哈希*或*消息摘要*，这是由一种单向加密算法生成的文件内容签名，为每个可能的输入创建唯一的输出。尽管有许多工具可以用来创建哈希，我将专注于使用 MD5 算法的工具。这种算法最初是在密码学和安全领域开发的，但研究人员后来发现了许多缺陷，现在只适合用于验证数据完整性等目的。

在 macOS 上，我可以使用**`md5`**来从第一个测试输入文件的内容生成一个 128 位的哈希值，如下所示：

```py
$ md5 -r tests/inputs/1.fa
c383c386a44d83c37ae287f0aa5ae11d tests/inputs/1.fa
```

我还可以使用**`openssl`**：

```py
$ openssl md5 tests/inputs/1.fa
MD5(tests/inputs/1.fa)= c383c386a44d83c37ae287f0aa5ae11d
```

在 Linux 上，我使用**`md5sum`**：

```py
$ md5sum tests/inputs/1.fa
c383c386a44d83c37ae287f0aa5ae11d  tests/inputs/1.fa
```

正如你所看到的，无论是什么工具或平台，对于相同的输入文件，哈希值都是相同的。如果我改变输入文件的任何一个比特，将会生成一个不同的哈希值。相反地，如果我找到另一个生成相同哈希值的文件，那么这两个文件的内容是相同的。例如，*empty.fa* 文件是我为测试创建的一个零长度文件，它具有以下哈希值：

```py
$ md5 -r tests/inputs/empty.fa
d41d8cd98f00b204e9800998ecf8427e tests/inputs/empty.fa
```

如果我使用**`touch foo`**命令创建另一个空文件，我会发现它具有相同的签名：

```py
$ touch foo
$ md5 -r foo
d41d8cd98f00b204e9800998ecf8427e foo
```

数据提供者通常会创建一个校验和文件，以便你可以验证数据的完整性。我创建了一个*tests/inputs/checksums.md5*如下所示：

```py
$ cd tests/inputs
$ md5 -r *.fa > checksums.md5
```

它具有以下内容：

```py
$ cat checksums.md5
c383c386a44d83c37ae287f0aa5ae11d 1.fa
863ebc53e28fdfe6689278e40992db9d 2.fa
d41d8cd98f00b204e9800998ecf8427e empty.fa
```

`md5sum`工具有一个`--check`选项，我可以使用它来自动验证文件与给定文件中的校验和是否匹配。macOS 的`md5`工具没有这个选项，但你可以使用**`brew install md5sha1sum`**来安装一个等效的`md5sum`工具来实现这个功能。

```py
$ md5sum --check checksums.md5
1.fa: OK
2.fa: OK
empty.fa: OK
```

MD5校验和提供了比手动检查文件大小更完整和更简便的数据完整性验证方式。虽然文件摘要不直接属于本练习的一部分，但在开始任何分析之前了解如何验证数据的完整性和未损坏性非常重要。

# 入门指南

你应该在*15_seqmagique*目录下进行此练习。我将像往常一样启动程序：

```py
$ new.py -fp 'Mimic seqmagick' seqmagique.py
Done, see new script "seqmagique.py".
```

首先，我需要使程序接受一个或多个文本文件作为位置参数。我还想创建一个选项来控制输出表格的格式。以下是相应的代码：

```py
import argparse
from typing import NamedTuple, TextIO, List

class Args(NamedTuple):
    """ Command-line arguments """
    files: List[TextIO]
    tablefmt: str

def get_args() -> Args:
    """Get command-line arguments"""

    parser = argparse.ArgumentParser(
        description='Argparse Python script',
        formatter_class=argparse.ArgumentDefaultsHelpFormatter)

    parser.add_argument('file', ![1](assets/1.png)
                        metavar='FILE',
                        type=argparse.FileType('rt'),
                        nargs='+',
                        help='Input FASTA file(s)')

    parser.add_argument('-t',
                        '--tablefmt',
                        metavar='table',
                        type=str,
                        choices=[ ![2](assets/2.png)
                            'plain', 'simple', 'grid', 'pipe', 'orgtbl', 'rst',
                            'mediawiki', 'latex', 'latex_raw', 'latex_booktabs'
                        ],
                        default='plain',
                        help='Tabulate table style')

    args = parser.parse_args()

    return Args(args.file, args.tablefmt)
```

[![1](assets/1.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO2-1)

为一个或多个可读文本文件定义一个位置参数。

[![2](assets/2.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO2-2)

定义一个选项，使用`choices`来限制参数为列表中的值，确保定义一个合理的`default`值。

使用`choices`选项来控制`--tablefmt`参数确实可以大大减少验证用户输入的工作量。如在[“使用Seqmagick分析序列文件”](#UsingSeqmagick)，对于表格格式选项的错误值会触发有用的错误消息。

修改`main()`函数以打印输入文件名：

```py
def main() -> None:
    args = get_args()
    for fh in args.files:
        print(fh.name)
```

并验证这是否有效：

```py
$ ./seqmagique.py tests/inputs/*.fa
tests/inputs/1.fa
tests/inputs/2.fa
tests/inputs/empty.fa
```

目标是遍历每个文件并打印以下内容：

`name`

文件名

`min_len`

最短序列的长度

`max_len`

最长序列的长度

`avg_len`

所有序列的平均/均值长度

`num_seqs`

序列的数量

如果你想为程序准备一些真实的输入文件，可以使用[`fastq-dump`工具](https://oreil.ly/Vmb0w)从NCBI下载来自研究[“北太平洋亚热带环流中的浮游微生物群落”](https://oreil.ly/aAGUA)的序列：

```py
$ fastq-dump --split-3 SAMN00000013 ![1](assets/1.png)
```

[![1](assets/1.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO3-1)

`--split-3`选项将确保成对端读取正确分割为正向/反向/未配对的读取。`SAMN00000013`字符串是实验中[一个样本](https://oreil.ly/kBCQU)的访问号。

## 使用tabulate()格式化文本表格

程序的输出将是使用该模块的`tabulate()`函数格式化的文本表格。确保阅读文档：

```py
>>> from tabulate import tabulate
>>> help(tabulate)
```

我需要为表格定义标题，并决定使用与Seqmagick相同的标题（减去`alignment`列）：

```py
>>> hdr = ['name', 'min_len', 'max_len', 'avg_len', 'num_seqs']
```

第一个测试文件*tests/inputs/1.fa*只有一个长度为50的序列，因此其列如下：

```py
>>> f1 = ['tests/inputs/1.fa', 50, 50, 50.00, 1]
```

第二个测试文件*tests/inputs/2.fa*有五个序列，长度从49个碱基到79个碱基，平均长度为64个碱基：

```py
>>> f2 = ['tests/inputs/2.fa', 49, 79, 64.00, 5]
```

`tabulate()`函数期望将表格数据作为一个列表的列表按位置传递，并且可以指定`headers`作为关键字参数：

```py
>>> print(tabulate([f1, f2], headers=hdr))
name                 min_len    max_len    avg_len    num_seqs
-----------------  ---------  ---------  ---------  ----------
tests/inputs/1.fa         50         50         50           1
tests/inputs/2.fa         49         79         64           5
```

或者，我可以将标题放在数据的第一行，并指示这是标题的位置：

```py
>>> print(tabulate([hdr, f1, f2], headers='firstrow'))
name                 min_len    max_len    avg_len    num_seqs
-----------------  ---------  ---------  ---------  ----------
tests/inputs/1.fa         50         50         50           1
tests/inputs/2.fa         49         79         64           5
```

注意，`tabulate()` 函数的默认表格样式是 `simple`，但我需要匹配 Seqmagick 的输出格式，所以我可以使用 `tablefmt` 选项设置为 `plain`：

```py
>>> print(tabulate([f1, f2], headers=hdr, tablefmt='plain'))
name                 min_len    max_len    avg_len    num_seqs
tests/inputs/1.fa         50         50         50           1
tests/inputs/2.fa         49         79         64           5
```

还有一点需要注意的是，`avg_len` 列中的值显示为整数，但应格式化为两位小数的浮点数。`floatfmt` 选项控制这一点，使用类似于我之前展示过的 f-string 数字格式化的语法：

```py
>>> print(tabulate([f1, f2], headers=hdr, tablefmt='plain', floatfmt='.2f'))
name                 min_len    max_len    avg_len    num_seqs
tests/inputs/1.fa         50         50      50.00           1
tests/inputs/2.fa         49         79      64.00           5
```

你的任务是处理每个文件中的所有序列，找到统计数据并打印最终表格。这应该足以帮助你解决问题。在能够通过所有测试之前，请不要提前阅读。

# 解决方案

我将展示两种解决方案，它们都显示文件统计信息，但输出格式不同。第一个解决方案使用 `tabulate()` 函数创建 ASCII 文本表格，第二个使用 `rich` 模块创建更精美的表格，可以让你的实验室同事和主要研究员（PI）印象深刻。

## 解决方案 1：使用 tabulate() 进行格式化

对于我的解决方案，我首先决定编写一个 `process()` 函数来处理每个输入文件。每当我面对需要处理某些列表项的问题时，我更喜欢专注于如何处理其中的一个项。也就是说，我不是试图找出所有文件的所有统计信息，而是首先想弄清楚如何找到一个文件的信息。

我的函数需要返回文件名和四个指标：最小值、最大值、平均序列长度以及序列数。就像 `Args` 类一样，我喜欢为此创建一个基于 `NamedTuple` 的类型，这样我就可以拥有一个静态类型的数据结构，可以由 `mypy` 进行验证：

```py
class FastaInfo(NamedTuple):
    """ FASTA file information """
    filename: str
    min_len: int
    max_len: int
    avg_len: float
    num_seqs: int
```

现在我可以定义一个函数来返回这个数据结构。请注意，我使用 `numpy.mean()` 函数来获取平均长度。`numpy` 模块提供了许多强大的数学操作来处理数值数据，特别适用于多维数组和线性代数函数。在导入依赖项时，通常使用 `np` 别名导入 `numpy` 模块：

```py
import numpy as np
from tabulate import tabulate
from Bio import SeqIO
```

在 REPL 中运行 **`help(np)`** 可以查看文档。这是我编写此函数的方式：

```py
def process(fh: TextIO) -> FastaInfo: ![1](assets/1.png)
    """ Process a file """

    if lengths := [len(rec.seq) for rec in SeqIO.parse(fh, 'fasta')]: ![2](assets/2.png)
        return FastaInfo(filename=fh.name, ![3](assets/3.png)
                         min_len=min(lengths), ![4](assets/4.png)
                         max_len=max(lengths), ![5](assets/5.png)
                         avg_len=round(float(np.mean(lengths)), 2), ![6](assets/6.png)
                         num_seqs=len(lengths)) ![7](assets/7.png)

    return FastaInfo(filename=fh.name, ![8](assets/8.png)
                     min_len=0,
                     max_len=0,
                     avg_len=0,
                     num_seqs=0)
```

[![1](assets/1.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO4-1)

该函数接受一个文件句柄，并返回一个 `FastaInfo` 对象。

[![2](assets/2.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO4-2)

使用列表推导式从文件句柄中读取所有序列。使用 `len()` 函数返回每个序列的长度。

[![3](assets/3.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO4-3)

文件的名称可以通过 `fh.name` 属性获得。

[![4](assets/4.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO4-4)

`min()`函数将返回最小值。

[![5](assets/5.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO4-5)

`max()`函数将返回最大值。

[![6](assets/6.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO4-6)

`np.mean()`函数将返回值列表的平均值。`round()`函数用于将此浮点值四舍五入为两位有效数字。

[![7](assets/7.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO4-7)

序列的数量是列表的长度。

[![8](assets/8.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO4-8)

如果没有序列，对所有值返回零。

无论何时，我都想为此编写一个单元测试。尽管我编写的集成测试覆盖了程序的这部分，但我想展示如何编写一个读取文件的函数的单元测试。而不是依赖实际文件，我将创建一个*模拟*或假文件句柄。

第一个测试文件如下所示：

```py
$ cat tests/inputs/1.fa
>SEQ0
GGATAAAGCGAGAGGCTGGATCATGCACCAACTGCGTGCAACGAAGGAAT
```

我可以使用`io.StringIO()`函数创建一个类似文件句柄的对象：

```py
>>> import io
>>> f1 = '>SEQ0\nGGATAAAGCGAGAGGCTGGATCATGCACCAACTGCGTGCAACGAAGGAAT\n' ![1](assets/1.png)
>>> fh = io.StringIO(f1) ![2](assets/2.png)
>>> for line in fh: ![3](assets/3.png)
...     print(line, end='') ![4](assets/4.png)
...
>SEQ0
GGATAAAGCGAGAGGCTGGATCATGCACCAACTGCGTGCAACGAAGGAAT
```

[![1](assets/1.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO5-1)

这是来自第一个输入文件的数据。

[![2](assets/2.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO5-2)

创建一个模拟文件句柄。

[![3](assets/3.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO5-3)

遍历模拟文件句柄的各行。

[![4](assets/4.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO5-4)

打印具有换行符（`\n`）的行，因此使用`end=''`以避免额外的换行符。

不过，有一个小问题，因为`process()`函数调用`fh.name`属性以获取输入文件名，这将引发异常：

```py
>>> fh.name
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: '_io.StringIO' object has no attribute 'name'
```

幸运的是，还有另一种方法可以使用Python的标准`unittest`模块创建模拟文件句柄。虽然我几乎在所有写作中更喜欢使用`pytest`模块，但`unittest`模块已存在很长时间，是另一个能够编写和运行测试的功能强大的框架。在这种情况下，我需要导入[`uni⁠t​test.mock.mock_open()`函数。](https://oreil.ly/EGvXh)这是我如何使用来自第一个测试文件的数据创建模拟文件句柄。我使用`read_data`来定义将由`fh.read()`方法返回的数据：

```py
>>> from unittest.mock import mock_open
>>> fh = mock_open(read_data=f1)()
>>> fh.read()
'>SEQ0\nGGATAAAGCGAGAGGCTGGATCATGCACCAACTGCGTGCAACGAAGGAAT\n'
```

在测试的上下文中，我不关心文件名，只要它返回一个字符串且不抛出异常即可：

```py
>>> fh.name
<MagicMock name='open().name' id='140349116126880'>
```

虽然我经常将单元测试放在与它们测试的函数相同的模块中，但在这种情况下，我更愿意将其放在一个单独的`unit.py`模块中，以使主程序更短。我编写了测试来处理一个空文件，一个带有一个序列的文件和一个带有多个序列的文件（这也反映在三个输入测试文件中）。假设如果函数对这三种情况有效，则对于所有其他情况都应有效：

```py
from unittest.mock import mock_open ![1](assets/1.png)
from seqmagique import process ![2](assets/2.png)

def test_process() -> None:
    """ Test process """

    empty = process(mock_open(read_data='')()) ![3](assets/3.png)
    assert empty.min_len == 0
    assert empty.max_len == 0
    assert empty.avg_len == 0
    assert empty.num_seqs == 0

    one = process(mock_open(read_data='>SEQ0\nAAA')()) ![4](assets/4.png)
    assert one.min_len == 3
    assert one.max_len == 3
    assert one.avg_len == 3
    assert one.num_seqs == 1

    two = process(mock_open(read_data='>SEQ0\nAAA\n>SEQ1\nCCCC')()) ![5](assets/5.png)
    assert two.min_len == 3
    assert two.max_len == 4
    assert two.avg_len == 3.5
    assert two.num_seqs == 2
```

[![1](assets/1.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO6-1)

导入`mock_open()`函数。

[![2](assets/2.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO6-2)

导入我正在测试的`process()`函数。

[![3](assets/3.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO6-3)

一个模拟空文件句柄，所有值应该为零。

[![4](assets/4.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO6-4)

一个具有三个碱基的单一序列。

[![5](assets/5.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO6-5)

一个带有两个序列的文件句柄，一个有三个碱基，一个有四个碱基。

使用**`pytest`**运行测试：

```py
$ pytest -xv unit.py
============================= test session starts ==============================
...

unit.py::test_process PASSED                                             [100%]

============================== 1 passed in 2.55s ===============================
```

这是我如何在`main()`中使用我的`process()`函数的方法：

```py
def main() -> None:
    args = get_args()
    data = [process(fh) for fh in args.files] ![1](assets/1.png)
    hdr = ['name', 'min_len', 'max_len', 'avg_len', 'num_seqs'] ![2](assets/2.png)
    print(tabulate(data, tablefmt=args.tablefmt, headers=hdr, floatfmt='.2f')) ![3](assets/3.png)
```

[![1](assets/1.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO7-1)

将所有输入文件处理成一个`FastaInfo`对象（元组）列表。

[![2](assets/2.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO7-2)

定义表头。

[![3](assets/3.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO7-3)

使用`tabulate()`函数打印格式化的输出表格。

为了测试这个程序，我使用以下输入来运行它：

+   空文件

+   带有一个序列的文件

+   具有两个序列的文件

+   所有输入文件

要开始，我使用默认表格样式运行所有这些。然后我需要验证所有10个表格样式是否正确创建。将所有可能的测试输入与所有表格样式结合起来会产生很高的*圈复杂度*——参数可以组合的不同方式的数量。

要测试这个，我首先需要手动验证我的程序是否正确运行。然后我需要为我打算测试的每个组合生成样本输出。我编写了以下`bash`脚本来为给定的输入文件和可能的表格样式创建一个*out*文件：

```py
$ cat mk-outs.sh
#!/usr/bin/env bash

PRG="./seqmagique.py" ![1](assets/1.png)
DIR="./tests/inputs" ![2](assets/2.png)
INPUT1="${DIR}/1.fa" ![3](assets/3.png)
INPUT2="${DIR}/2.fa"
EMPTY="${DIR}/empty.fa"

$PRG $INPUT1 > "${INPUT1}.out" ![4](assets/4.png)
$PRG $INPUT2 > "${INPUT2}.out"
$PRG $EMPTY > "${EMPTY}.out"
$PRG $INPUT1 $INPUT2 $EMPTY > "$DIR/all.fa.out"

STYLES="plain simple grid pipe orgtbl rst mediawiki latex latex_raw
 latex_booktabs"

for FILE in $INPUT1 $INPUT2; do ![5](assets/5.png)
    for STYLE in $STYLES; do
        $PRG -t $STYLE $FILE > "$FILE.${STYLE}.out"
    done
done

echo Done.
```

[![1](assets/1.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO8-1)

正在测试的程序。

[![2](assets/2.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO8-2)

输入文件的目录。

[![3](assets/3.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO8-3)

输入文件。

[![4](assets/4.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO8-4)

使用三个输入文件和默认的表格样式运行程序。

[![5](assets/5.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO8-5)

使用两个输入文件和所有表格样式运行程序。

在*tests/seqmagique_test.py*中的测试将使用给定文件运行程序，并将输出与*tests/inputs*目录中的*out*文件之一进行比较。在此模块的顶部，我定义了输入和输出文件，如下所示：

```py
TEST1 = ('./tests/inputs/1.fa', './tests/inputs/1.fa.out')
```

我在模块中定义了一个`run()`函数，用于使用输入文件运行程序，并将实际输出与预期输出进行比较。这是一个基本模式，你可以用来测试任何程序的输出：

```py
def run(input_file: str, expected_file: str) -> None:
    """ Runs on command-line input """

    expected = open(expected_file).read().rstrip() ![1](assets/1.png)
    rv, out = getstatusoutput(f'{RUN} {input_file}') ![2](assets/2.png)
    assert rv == 0 ![3](assets/3.png)
    assert out == expected ![4](assets/4.png)
```

[![1](assets/1.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO9-1)

从文件中读取预期的输出。

[![2](assets/2.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO9-2)

使用默认的表格样式运行给定的输入文件。

[![3](assets/3.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO9-3)

检查返回值是否为`0`。

[![4](assets/4.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO9-4)

检查输出是否符合预期值。

我是这样使用它的：

```py
def test_input1() -> None:
    """ Runs on command-line input """

    run(*TEST1) ![1](assets/1.png)
```

[![1](assets/1.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO10-1)

将元组展开以将两个值按位置传递给`run()`函数。

测试套件还检查表格样式：

```py
def test_styles() -> None:
    """ Test table styles """

    styles = [ ![1](assets/1.png)
        'plain', 'simple', 'grid', 'pipe', 'orgtbl', 'rst', 'mediawiki',
        'latex', 'latex_raw', 'latex_booktabs'
    ]

    for file in [TEST1[0], TEST2[0]]: ![2](assets/2.png)
        for style in styles: ![3](assets/3.png)
            expected_file = file + '.' + style + '.out' ![4](assets/4.png)
            assert os.path.isfile(expected_file) ![5](assets/5.png)
            expected = open(expected_file).read().rstrip() ![6](assets/6.png)
            flag = '--tablefmt' if random.choice([0, 1]) else '-t' ![7](assets/7.png)
            rv, out = getstatusoutput(f'{RUN} {flag} {style} {file}') ![8](assets/8.png)
            assert rv == 0 ![9](assets/9.png)
            assert out == expected
```

[![1](assets/1.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO11-1)

定义一个包含所有可能样式的列表。

[![2](assets/2.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO11-2)

使用两个非空文件。

[![3](assets/3.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO11-3)

遍历每种样式。

[![4](assets/4.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO11-4)

输出文件是输入文件名加上样式和扩展名*.out*。

[![5](assets/5.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO11-5)

检查文件是否存在。

[![6](assets/6.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO11-6)

从文件中读取预期值。

[![7](assets/7.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO11-7)

随机选择短或长标志进行测试。

[![8](assets/8.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO11-8)

使用标志选项、样式和文件运行程序。

[![9](assets/9.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO11-9)

确保程序无错误运行，并生成正确的输出。

如果我做了改动以致于程序不再创建与之前相同的输出，这些测试应该能够捕捉到。这是一种*回归*测试，我正在比较程序现在的工作方式与之前的工作方式。也就是说，无法生成相同输出将被视为一种回归。虽然我的测试套件并非完全详尽，但它涵盖了足够多的组合，我对程序的正确性感到有信心。

## 解决方案 2：使用 rich 进行格式化

在这第二个解决方案中，我想展示另一种创建输出表格的方法，使用 `rich` 模块来跟踪输入文件的处理并创建一个更漂亮的输出表格。[图 15-1](#fig_15.1) 展示了输出的外观。

![mpfb 1501](assets/mpfb_1501.png)

###### 图 15-1\. 使用 `rich` 模块的进度指示器和输出表格更加华丽

我仍然以相同的方式处理文件，所不同的只是创建输出的方式。我首先需要导入所需的函数：

```py
from rich.console import Console
from rich.progress import track
from rich.table import Table, Column
```

这是我使用它们的方式：

```py
def main() -> None:
    args = get_args()

    table = Table('Name', ![1](assets/1.png)
                  Column(header='Min. Len', justify='right'),
                  Column(header='Max. Len', justify='right'),
                  Column(header='Avg. Len', justify='right'),
                  Column(header='Num. Seqs', justify='right'),
                  header_style="bold black")

    for fh in track(args.file): ![2](assets/2.png)
        file = process(fh) ![3](assets/3.png)
        table.add_row(file.filename, str(file.min_len), str(file.max_len), ![4](assets/4.png)
                      str(file.avg_len), str(file.num_seqs))

    console = Console() ![5](assets/5.png)
    console.print(table)
```

[![1](assets/1.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO12-1)

创建表格来保存数据。名称列是一个标准的左对齐字符串字段。所有其他列都需要右对齐，并需要一个自定义的 `Column` 对象。

[![2](assets/2.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO12-2)

使用 `track()` 函数迭代每个文件句柄，为用户创建进度条。

[![3](assets/3.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO12-3)

处理文件以获取统计信息。

[![4](assets/4.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO12-4)

将文件的统计信息添加到表格中。注意，所有值必须是字符串。

[![5](assets/5.png)](#co_seqmagique__creating__span_class__keep_together__and_formatting_reports__span__CO12-5)

创建一个 `Console` 对象，并用它来打印输出。

# 更进一步

`seqmagick` 工具还有许多其他有用的选项。尽量实现您能实现的所有版本。

# 复习

本章要点：

+   `seqmagick` 工具提供了许多检查序列文件的方法。

+   有许多方法可以验证您的输入文件是否完整且未损坏，从检查文件大小到使用 MD5 哈希等消息摘要。

+   `argparse` 参数的 `choices` 选项将强制用户从给定列表中选择一个值。

+   `tabulate` 和 `rich` 模块可以创建数据的文本表格。

+   `numpy` 模块对许多数学操作都很有用。

+   `io.StringIO()` 和 `unittest.mock.mock_open()` 函数提供了两种模拟文件句柄进行测试的方法。

+   回归测试验证程序继续像之前一样工作。
