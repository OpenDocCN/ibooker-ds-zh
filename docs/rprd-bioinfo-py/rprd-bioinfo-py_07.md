# 第6章\. 找到海明距离：计算点突变

海明距离，以前言中提到的理查德·哈明的名字命名，是将一个字符串转变为另一个所需的编辑次数。这是衡量序列相似性的一种度量。我为此编写了几个其他度量标准，从[第1章](ch01.html#ch01)开始使用四核苷酸频率，继续到[第5章](ch05.html#ch05)使用GC含量。尽管后者在实际中可能是有信息量的，因为编码区域倾向于富含GC，但四核苷酸频率远远不能称得上是有用的。例如，序列*AAACCCGGGTTT*和*CGACGATATGTC*完全不同，但产生相同的碱基频率：

```py
$ ./dna.py AAACCCGGGTTT
3 3 3 3
$ ./dna.py CGACGATATGTC
3 3 3 3
```

单独观察，四核苷酸频率使这些序列看起来相同，但很明显它们会产生完全不同的蛋白质序列，因此在功能上是不同的。[图 6-1](#fig_6.1)显示了这2个序列的对齐，表明只有12个碱基中的3个是相同的，这意味着它们只有25%的相似性。

![mpfb 0601](assets/mpfb_0601.png)

###### 图6-1\. 显示匹配碱基的垂直条的两个序列的对齐

另一种表达这个概念的方式是说，12个碱基中有9个需要改变才能将其中一个序列变成另一个。这就是海明距离，它在生物信息学中与单核苷酸多态性（SNP，发音为*snips*）或单核苷酸变异（SNVs，发音为*snivs*）有些类似。该算法只考虑将一个碱基更改为另一个值，并且远远不能像序列对齐那样识别插入和删除。例如，[图 6-2](#fig_6.2)显示，当对齐时序列*AAACCCGGGTTT*和*AACCCGGGTTTA*是92%相似的（在左边），因为它们仅相差一个碱基。然而，海明距离（在右边）只显示有8个碱基是相同的，这意味着它们只有66%的相似性。

![mpfb 0602](assets/mpfb_0602.png)

###### 图6-2\. 这些序列的对齐显示它们几乎相同，而海明距离发现它们只有66%的相似性

该程序将始终严格比较字符串从它们的开头开始，这限制了实际应用于真实世界生物信息学的可能性。尽管如此，这个天真的算法事实证明是衡量序列相似性的一个有用的度量，并且编写实现在Python中提出了许多有趣的解决方案。

在本章中，您将学到：

+   如何使用`abs()`和`min()`函数

+   如何组合两个可能长度不等的列表的元素

+   如何使用`lambda`或现有函数编写`map()`

+   如何使用`operator`模块中的函数

+   如何使用`itertools.starmap()`函数

# 入门

您应该在存储库的*06_hamm*目录中工作。我建议您先了解这些解决方案的工作原理，然后将其中一个复制到`hamm.py`程序中并请求帮助：

```py
$ cp solution1_abs_iterate.py hamm.py
$ ./hamm.py -h
usage: hamm.py [-h] str str

Hamming distance

positional arguments:
  str         Sequence 1
  str         Sequence 2

optional arguments:
  -h, --help  show this help message and exit
```

该程序需要两个位置参数，即要比较的两个序列，并应打印汉明距离。例如，要将其中一个序列更改为另一个序列，我需要进行七次编辑：

```py
$ ./hamm.py GAGCCTACTAACGGGAT CATCGTAATGACGGCCT
7
```

运行测试（使用**`pytest`**或**`make test`**）以查看通过的测试套件。一旦您了解了预期的内容，请删除此文件并从头开始： 

```py
$ new.py -fp 'Hamming distance' hamm.py
Done, see new script "hamm.py".
```

定义参数，以便程序需要两个位置参数，这两个参数是字符串值：

```py
import argparse
from typing import NamedTuple

class Args(NamedTuple): ![1](assets/1.png)
    """ Command-line arguments """
    seq1: str
    seq2: str

# --------------------------------------------------
def get_args():
    """ Get command-line arguments """

    parser = argparse.ArgumentParser(
        description='Hamming distance',
        formatter_class=argparse.ArgumentDefaultsHelpFormatter)

    parser.add_argument('seq1', metavar='str', help='Sequence 1') ![2](assets/2.png)

    parser.add_argument('seq2', metavar='str', help='Sequence 2')

    args = parser.parse_args()

    return Args(args.seq1, args.seq2) ![3](assets/3.png)
```

[![1](assets/1.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO1-1)

程序参数将包含两个字符串值，用于这两个序列。

[![2](assets/2.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO1-2)

这两个序列是必需的位置字符串值。

[![3](assets/3.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO1-3)

实例化`Args`对象，使用这两个序列。

定义位置参数的顺序必须与在命令行上提供参数的顺序相匹配。也就是说，第一个位置参数将保存第一个位置参数，第二个位置参数将匹配第二个位置参数，依此类推。定义可选参数的顺序无关紧要，可选参数可以在位置参数之前或之后定义。

更改`main()`函数以打印这两个序列：

```py
def main():
    args = get_args()
    print(args.seq1, args.seq2)
```

此时，您应该有一个打印用法的程序，验证用户提供了两个序列，并打印这些序列：

```py
$ ./hamm.py GAGCCTACTAACGGGAT CATCGTAATGACGGCCT
GAGCCTACTAACGGGAT CATCGTAATGACGGCCT
```

如果您运行**`pytest -xvv`**（两个`v`增加输出的详细程度），您应该会发现程序通过了前三个测试。它应该在`test_input1`测试中失败，并显示类似以下的消息：

```py
=================================== FAILURES ===================================
_________________________________ test_input1 __________________________________

    def test_input1() -> None:
        """ Test with input1 """

>       run(INPUT1)

tests/hamm_test.py:47:
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _

file = './tests/inputs/1.txt' ![1](assets/1.png)

    def run(file: str) -> None:
        """ Run with input """

        assert os.path.isfile(file)
        seq1, seq2, expected = open(file).read().splitlines() ![2](assets/2.png)

        rv, out = getstatusoutput(f'{RUN} {seq1} {seq2}') ![3](assets/3.png)
        assert rv == 0
>       assert out.rstrip() == expected ![4](assets/4.png)
E       AssertionError: assert 'GAGCCTACTAACGGGAT CATCGTAATGACGGCCT' == '7' ![5](assets/5.png)
E         - 7
E         + GAGCCTACTAACGGGAT CATCGTAATGACGGCCT

tests/hamm_test.py:40: AssertionError
=========================== short test summary info ============================
FAILED tests/hamm_test.py::test_input1 - AssertionError: assert 'GAGCCTACTAAC...
!!!!!!!!!!!!!!!!!!!!!!!!!! stopping after 1 failures !!!!!!!!!!!!!!!!!!!!!!!!!!!
========================= 1 failed, 3 passed in 0.27s ==========================
```

[![1](assets/1.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO2-1)

测试的输入来自文件*./tests/inputs/1.txt*。

[![2](assets/2.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO2-2)

打开文件并读取两个序列和预期结果。

[![3](assets/3.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO2-3)

使用这两个序列运行程序。

[![4](assets/4.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO2-4)

当程序的输出与预期答案不匹配时，`assert`会失败。

[![5](assets/5.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO2-5)

具体来说，当程序应该打印两个序列时，它却打印了`7`。

## 迭代两个字符串的字符。

现在来找到这两个序列之间的汉明距离。首先，考虑这两个序列：

```py
>>> seq1, seq2 = 'AC', 'ACGT'
```

距离为2，因为你需要在第一个序列中添加*GT*或从第二个序列中删除*GT*使它们相同。我建议基线距离是它们长度的差异。请注意，Rosalind挑战假设两个相等长度的字符串，但我想使用这个练习来考虑长度不同的字符串。

根据做减法的顺序，可能会得到一个负数：

```py
>>> len(seq1) - len(seq2)
-2
```

使用`abs()`函数获取绝对值：

```py
>>> distance = abs(len(seq1) - len(seq2))
>>> distance
2
```

现在我将考虑如何迭代它们共有的字符。我可以使用`min()`函数找到较短序列的长度：

```py
>>> min(len(seq1), len(seq2))
2
```

我可以使用`range()`函数与这个一起，以获取相同字符的索引：

```py
>>> for i in range(min(len(seq1), len(seq2))):
...     print(seq1[i], seq2[i])
...
A A
C C
```

当这两个字符*不*相等时，应增加`distance`变量，因为我必须更改一个值以使其匹配另一个。请记住，Rosalind挑战总是从它们的开头比较这两个序列。例如，序列*ATTG*和*TTG*在一个碱基上不同，因为我可以从第一个中删除*A*或将其添加到第二个中以使它们匹配，但这个特定挑战的规则会说正确答案是3：

```py
$ ./hamm.py ATTG TTG
3
```

我相信这些信息足以帮助您编写一个通过测试套件的解决方案。一旦您有了可用的解决方案，请探索一些其他编写算法的方法，并使用测试套件不断检查您的工作。除了通过**`pytest`**运行测试外，确保使用**`make test`**选项验证您的代码也通过各种linting和类型检查测试。

# 解决方案

本节将介绍如何计算汉明距离的八种变体，从完全手动计算几行代码到将几个函数合并在一行的解决方案。

## 解决方案1：迭代和计数

第一个解决方案来自前一节的建议：

```py
def main():
    args = get_args()
    seq1, seq2 = args.seq1, args.seq2 ![1](assets/1.png)

    l1, l2 = len(seq1), len(seq2) ![2](assets/2.png)
    distance = abs(l1 - l2) ![3](assets/3.png)

    for i in range(min(l1, l2)): ![4](assets/4.png)
        if seq1[i] != seq2[i]: ![5](assets/5.png)
            distance += 1 ![6](assets/6.png)

    print(distance) ![7](assets/7.png)
```

[![1](assets/1.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO3-1)

将两个序列复制到变量中。

[![2](assets/2.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO3-2)

由于我会多次使用长度，我将它们存储在变量中。

[![3](assets/3.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO3-3)

基础距离是两个长度之间的差异。

[![4](assets/4.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO3-4)

使用较短的长度来找到共有的索引。

[![5](assets/5.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO3-5)

检查每个位置的字母。

[![6](assets/6.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO3-6)

将距离增加1。

[![7](assets/7.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO3-7)

打印距离。

此解决方案非常明确，列出了比较两个字符串所有字符所需的每个单独步骤。接下来的解决方案将开始缩短许多步骤，所以请确保你对这里展示的内容非常熟悉。

## 解决方案2：创建单元测试

第一个解决方案让我感到有些不舒服，因为计算汉明距离的代码应该在一个带有测试的函数中。我将首先创建一个名为`hamming()`的函数，放在`main()`函数之后。就风格而言，我喜欢先放`get_args()`，这样我一打开程序就能立即看到。我的`main()`函数总是其次，其他所有函数和测试都在其后。

我将从想象函数的输入和输出开始：

```py
def hamming(seq1: str, seq2: str) -> int: ![1](assets/1.png)
    """ Calculate Hamming distance """

    return 0 ![2](assets/2.png)
```

[![1](assets/1.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO4-1)

该函数将接受两个字符串作为位置参数，并返回一个整数。

[![2](assets/2.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO4-2)

要开始，该函数将始终返回`0`。

我想强调的是，该函数并不*打印*答案，而是*作为结果返回*。如果你写了这个函数来`print()`距离，你将无法编写单元测试。你必须完全依赖于集成测试来查看程序是否打印了正确的答案。尽可能地，我鼓励你编写纯函数，它们只对参数进行操作，没有副作用。打印是副作用，尽管程序最终确实需要打印答案，但这个函数的任务仅仅是在给定两个字符串时返回一个整数。

我已经展示了一些测试用例，你可以自行添加其他测试：

```py
def test_hamming() -> None:
    """ Test hamming """

    assert hamming('', '') == 0 ![1](assets/1.png)
    assert hamming('AC', 'ACGT') == 2 ![2](assets/2.png)
    assert hamming('GAGCCTACTAACGGGAT', 'CATCGTAATGACGGCCT') == 7 ![3](assets/3.png)
```

[![1](assets/1.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO5-1)

我总是认为将空字符串发送给字符串输入是一种良好的实践。

[![2](assets/2.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO5-2)

差异仅仅是长度的不同。

[![3](assets/3.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO5-3)

这是文档中的示例。

我知道这可能看起来有点极端，因为这个函数本质上就是整个程序。我几乎是在复制集成测试，我知道，但我用它来指出编写程序的最佳实践。`hamming()`函数是一个很好的代码单元，并且应该与测试一起放入函数中。在一个更大的程序中，这可能是数十到数百个其他函数之一，每个函数都应该*封装*、*文档化*和*测试*。

遵循测试驱动的原则，在程序上运行**`pytest`**以确保测试失败：

```py
$ pytest -v hamm.py
========================== test session starts ==========================
...

hamm.py::test_hamming FAILED                                      [100%]

=============================== FAILURES ================================
_____________________________ test_hamming ______________________________

    def test_hamming() -> None:
        """ Test hamming """

        assert hamming('', '') == 0
>       assert hamming('AC', 'ACGT') == 2
E       assert 0 == 2
E         +0
E         -2

hamm.py:69: AssertionError
======================== short test summary info ========================
FAILED hamm.py::test_hamming - assert 0 == 2
=========================== 1 failed in 0.13s ===========================
```

现在从`main()`中复制代码以修复函数：

```py
def hamming(seq1: str, seq2: str) -> int:
    """ Calculate Hamming distance """

    l1, l2 = len(seq1), len(seq2)
    distance = abs(l1 - l2)

    for i in range(min(l1, l2)):
        if seq1[i] != seq2[i]:
            distance += 1

    return distance
```

验证函数的正确性：

```py
$ pytest -v hamm.py
========================== test session starts ==========================
...

hamm.py::test_hamming PASSED                                      [100%]

=========================== 1 passed in 0.02s ===========================
```

您可以像这样将其整合到您的`main()`函数中：

```py
def main():
    args = get_args()
    print(hamming(args.seq1, args.seq2))  ![1](assets/1.png)
```

[![1](assets/1.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO6-1)

打印给定序列的函数返回值。

这将程序的复杂性隐藏在一个命名、文档化和测试过的单元中，缩短了程序的主体并提高了可读性。

## 解决方案3：使用`zip()`函数

下面的解决方案使用`zip()`函数将两个序列的元素组合起来。结果是一个包含每个位置字符的元组列表（见图6-3）。请注意，`zip()`是另一个惰性函数，因此我将使用`list()`在REPL中强制执行值：

```py
>>> list(zip('ABC', '123'))
[('A', '1'), ('B', '2'), ('C', '3')]
```

![mpfb 0603](assets/mpfb_0603.png)

###### 图6-3。元组由共同位置的字符组成。

如果我使用*AC*和*ACGT*序列，您将注意到`zip()`会停在较短的序列处，如图6-4所示：

```py
>>> list(zip('AC', 'ACGT'))
[('A', 'A'), ('C', 'C')]
```

![mpfb 0604](assets/mpfb_0604.png)

###### 图6-4。`zip()`函数将在最短序列处停止

我可以使用`for`循环遍历每一对。到目前为止，在我的`for`循环中，我使用单个变量表示列表中的每个元素，就像这样：

```py
>>> for tup in zip('AC', 'ACGT'):
...     print(tup)
...
('A', 'A')
('C', 'C')
```

在[第1章](ch01.html#ch01)，我展示了如何将元组中的值*解包*为单独的变量。Python的`for`循环允许我将每个元组解包为两个字符，如下所示：

```py
>>> for char1, char2 in zip('AC', 'ACGT'):
...     print(char1, char2)
...
A A
C C
```

`zip()`函数省去了第一个实现中的几行：

```py
def hamming(seq1: str, seq2: str) -> int:
    """ Calculate Hamming distance """

    distance = abs(len(seq1) - len(seq2)) ![1](assets/1.png)

    for char1, char2 in zip(seq1, seq2): ![2](assets/2.png)
        if char1 != char2: ![3](assets/3.png)
            distance += 1 ![4](assets/4.png)

    return distance
```

[![1](assets/1.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO7-1)

从长度的绝对差开始。

[![2](assets/2.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO7-2)

使用`zip()`将两个字符串的字符配对。

[![3](assets/3.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO7-3)

检查两个字符是否不相等。

[![4](assets/4.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO7-4)

增加距离。

## 解决方案4：使用`zip_longest()`函数

下一个解决方案从 `itertools` 模块导入 `zip_longest()` 函数。顾名思义，它将把列表压缩到最长列表的长度。[Figure 6-5](#fig_6.5) 显示当较短的序列已用尽时，该函数将插入 `None` 值：

```py
>>> from itertools import zip_longest
>>> list(zip_longest('AC', 'ACGT'))
[('A', 'A'), ('C', 'C'), (None, 'G'), (None, 'T')]
```

![mpfb 0605](assets/mpfb_0605.png)

###### 图 6-5\. `zip_longest()` 函数将在最长序列处停止

我不再需要从序列长度开始减去。相反，我将初始化一个 `distance` 变量为 `0`，然后使用 `zip_longest()` 创建要比较的碱基元组：

```py
def hamming(seq1: str, seq2: str) -> int:
    """ Calculate Hamming distance """

    distance = 0 ![1](assets/1.png)
    for char1, char2 in zip_longest(seq1, seq2): ![2](assets/2.png)
        if char1 != char2: ![3](assets/3.png)
            distance += 1 ![4](assets/4.png)

    return distance
```

[![1](assets/1.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO8-1)

初始化距离为 `0`。

[![2](assets/2.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO8-2)

将序列压缩到最长的长度。

[![3](assets/3.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO8-3)

比较字符。

[![4](assets/4.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO8-4)

增加计数器。

## Solution 5: 使用列表推导式

到目前为止，所有的解决方案都使用了 `for` 循环。我希望你开始预见我接下来会展示如何将其转换为列表推导式。当目标是创建一个新列表或将值列表缩减为某个答案时，通常使用列表推导式会更短和更可取。

第一个版本将使用一个 `if` 表达式，如果两个字符相同则返回 `1`，如果它们不同则返回 `0`：

```py
>>> seq1, seq2, = 'GAGCCTACTAACGGGAT', 'CATCGTAATGACGGCCT'
>>> [1 if c1 != c2 else 0 for c1, c2 in zip_longest(seq1, seq2)]
[1, 0, 1, 0, 1, 0, 0, 1, 0, 1, 0, 0, 0, 0, 1, 1, 0]
```

然后，汉明距离是这些的总和：

```py
>>> sum([1 if c1 != c2 else 0 for c1, c2 in zip_longest(seq1, seq2)])
7
```

另一种表达这个想法的方式是通过使用 *保护* 子句，即在列表推导式末尾的条件语句，决定是否允许特定元素：

```py
>>> ones = [1 for c1, c2 in zip_longest(seq1, seq2) if c1 != c2] ![1](assets/1.png)
>>> ones
[1, 1, 1, 1, 1, 1, 1]
>>> sum(ones)
7
```

[![1](assets/1.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO9-1)

`if` 语句是保护子句，如果两个字符不相等，则产生值 `1`。

您还可以使用我在 [Chapter 5](ch05.html#ch05) 中展示的布尔值/整数强制转换，其中每个 `True` 值将被视为 `1`，而 `False` 是 `0`：

```py
>>> bools = [c1 != c2 for c1, c2 in zip_longest(seq1, seq2)]
>>> bools
[True, False, True, False, True, False, False, True, False, True, False,
False, False, False, True, True, False]
>>> sum(bools)
7
```

任何这些想法都将函数简化为一行代码，以通过测试：

```py
def hamming(seq1: str, seq2: str) -> int:
    """ Calculate Hamming distance """

    return sum([c1 != c2 for c1, c2 in zip_longest(seq1, seq2)])
```

## Solution 6: 使用 `filter()` 函数

章节 [4](ch04.html#ch04) 和 [5](ch05.html#ch05) 表明，带有保护子句的列表推导式也可以使用 `filter()` 函数来表达。语法有些难看，因为Python不允许将 `zip_longest()` 中的元组解包为单独的变量。也就是说，我想编写一个 `lambda` 函数，将 `char1` 和 `char2` 解包为单独的变量，但这是不可能的：

```py
>>> list(filter(lambda char1, char2: char1 != char2, zip_longest(seq1, seq2)))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: <lambda>() missing 1 required positional argument: 'char2'
```

相反，我通常会将`lambda`变量称为`tup`或`t`，以提醒我这是一个元组。我将使用位置元组表示法将零位置的元素与一位置的元素进行比较。`filter()`只会生成那些元素不同的元组：

```py
>>> seq1, seq2 = 'AC', 'ACGT'
>>> list(filter(lambda t: t[0] != t[1], zip_longest(seq1, seq2)))
[(None, 'G'), (None, 'T')]
```

然后，汉明距离就是这个列表的长度。请注意，`len()`函数不会促使`filter()`生成值：

```py
>>> len(filter(lambda t: t[0] != t[1], zip_longest(seq1, seq2)))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: object of type 'filter' has no len()
```

这是代码必须使用`list()`来强制惰性`filter()`函数生成结果的一个例子。以下是我如何将这些思想整合到一起的方式：

```py
def hamming(seq1: str, seq2: str) -> int:
    """ Calculate Hamming distance """

    distance = filter(lambda t: t[0] != t[1], zip_longest(seq1, seq2)) ![1](assets/1.png)
    return len(list((distance))) ![2](assets/2.png)
```

[![1](assets/1.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO10-1)

使用`filter()`查找不同字符的元组对。

[![2](assets/2.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO10-2)

返回结果列表的长度。

## 解决方案 7：使用`map()`函数和`zip_longest()`函数

此解决方案使用`map()`而不是`filter()`，只是为了向您展示元组无法解包的相同情况。我想使用`map()`来生成一个布尔值列表，指示字符对是否匹配：

```py
>>> seq1, seq2 = 'AC', 'ACGT'
>>> list(map(lambda t: t[0] != t[1], zip_longest(seq1, seq2)))
[False, False, True, True]
```

这个`lambda`与用作*谓词*来确定哪些元素被允许通过的`filter()`中的lambda完全相同。在这里，代码*转换*元素成为应用`lambda`函数到参数后的结果，如[Figure 6-6](#fig_6.6)所示。记住，`map()`将始终返回与其消耗相同数量的元素，但`filter()`可能返回较少或根本不返回。

![mpfb 0606](assets/mpfb_0606.png)

###### 图 6-6\. `map()`函数将每个元组转换为表示两个元素不等的布尔值

我可以将这些布尔值求和，得到不匹配对的数量：

```py
>>> seq1, seq2, = 'GAGCCTACTAACGGGAT', 'CATCGTAATGACGGCCT'
>>> sum(map(lambda t: t[0] != t[1], zip_longest(seq1, seq2)))
7
```

这是带有此想法的函数：

```py
def hamming(seq1: str, seq2: str) -> int:
    """ Calculate Hamming distance """

    return sum(map(lambda t: t[0] != t[1], zip_longest(seq1, seq2)))
```

尽管这些函数已经从10多行代码减少到一行，但将其作为具有描述性名称和测试的函数仍然是有意义的。最终，您将开始创建可在项目间共享的可重用代码模块。

## 解决方案 8：使用`starmap()`和`operator.ne()`函数

我承认，我展示最后几个解决方案只是为了建立到这个最后一个解决方案。让我首先展示如何将`lambda`分配给一个变量：

```py
>>> not_same = lambda t: t[0] != t[1]
```

这不是推荐的语法，并且`pylint`肯定会在此处失败并推荐使用`def`代替：

```py
def not_same(t):
    return t[0] != t[1]
```

两者都将创建一个名为`not_same()`的函数，接受一个元组并返回两个元素是否相同：

```py
>>> not_same(('A', 'A'))
False
>>> not_same(('A', 'T'))
True
```

但是，如果我编写函数来接受两个位置参数，那么之前看到的相同错误将会出现：

```py
>>> not_same = lambda a, b: a != b
>>> list(map(not_same, zip_longest(seq1, seq2)))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: <lambda>() missing 1 required positional argument: 'b'
```

我需要的是`map()`的一个版本，它可以展开传入的元组（正如我在[第1章](ch01.html#ch01)中首次展示的那样），通过在元组前添加`*`（星号、星号、星形）来将其展开为其元素，这正是`itertools.starmap()`函数所做的（参见[图6-7](#fig_6.7)）：

```py
>>> from itertools import zip_longest, starmap
>>> seq1, seq2 = 'AC', 'ACGT'
>>> list(starmap(not_same, zip_longest(seq1, seq2)))
[False, False, True, True]
```

![mpfb 0607](assets/mpfb_0607.png)

###### 图6-7. `starmap()`函数对传入的元组应用星号，将其转换为`lambda`所期望的两个值

但等等，还有更多！我甚至不需要编写自己的`not_same()`函数，因为我已经有`operator.ne()`（不等于），通常使用`!=`操作符来编写：

```py
>>> import operator
>>> operator.ne('A', 'A')
False
>>> operator.ne('A', 'T')
True
```

*运算符*是一种特殊的二元函数（接受两个参数），函数名称通常是一些符号，如`+`，它位于参数之间。对于`+`，Python必须决定这是否意味着`operator.add()`：

```py
>>> 1 + 2
3
>>> operator.add(1, 2)
3
```

或者`operator.concat()`：

```py
>>> 'AC' + 'GT'
'ACGT'
>>> operator.concat('AC', 'GT')
'ACGT'
```

关键在于我已经有一个现有的函数，期望两个参数并返回它们是否相等，并且我可以使用`starmap()`来正确地将元组扩展为所需的参数：

```py
>>> seq1, seq2 = 'AC', 'ACGT'
>>> list(starmap(operator.ne, zip_longest(seq1, seq2)))
[False, False, True, True]
```

与之前一样，汉明距离是不匹配对的总和：

```py
>>> seq1, seq2, = 'GAGCCTACTAACGGGAT', 'CATCGTAATGACGGCCT'
>>> sum(starmap(operator.ne, zip_longest(seq1, seq2)))
7
```

看它如何运作：

```py
def hamming(seq1: str, seq2: str) -> int:
    """ Calculate Hamming distance """

    return sum(starmap(operator.ne, zip_longest(seq1, seq2))) ![1](assets/1.png)
```

[![1](assets/1.png)](#co_finding_the_hamming_distance___span_class__keep_together__counting_point_mutations__span__CO11-1)

将序列压缩，将元组转换为布尔比较，并求和。

此最终解决方案完全依赖于适合的四个我没有编写的函数。我相信最好的代码是你不编写（或测试或文档化）的代码。虽然我更喜欢这个纯函数解决方案，但你可能认为这段代码过于巧妙。你应该使用一年后你能理解的版本。

# 更进一步

+   不查看源代码，编写`zip_longest()`的一个版本。确保从测试开始，然后编写满足测试的函数。

+   扩展你的程序以处理超过两个输入序列。使你的程序打印每对序列之间的汉明距离。这意味着程序将打印*n*选择*k*个数，即*n*！/（*k*！（*n* - *k*）！）。对于三个序列，你的程序将打印3！/（2！（3 - 2）！）= 6 / 2 = 3距离对。

+   尝试编写一个序列对齐算法，该算法将显示例如序列*AAACCCGGGTTT*和*AACCCGGGTTTA*之间仅有一个差异。

# 回顾

这是一个相当深的兔子洞，只为了找到汉明距离，但它突出了关于Python函数的许多有趣细节：

+   内置的`zip()`函数将两个或更多个列表合并为元组列表，将相同位置的元素分组。它会停在最短的序列处，因此如果要处理最长的序列，请使用`itertools.zip_longest()`函数。

+   `map()` 和 `filter()` 都会将函数应用于某些可迭代的值。`map()` 函数将由函数转换的新序列返回，而 `filter()` 仅在应用函数时返回那些返回真值的元素。

+   传递给 `map()` 和 `filter()` 的函数可以是由 `lambda` 创建的匿名函数，也可以是现有函数。

+   `operator` 模块包含许多像 `ne()`（不等于）这样的函数，可以与 `map()` 和 `filter()` 一起使用。

+   `functools.starmap()` 函数的工作方式类似于 `map()`，但会将函数的传入值展开成值列表。
