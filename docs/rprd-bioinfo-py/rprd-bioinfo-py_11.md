# 第十章：找到最长共享子序列：找到 K-mers，编写函数和使用二分搜索

如[Rosalind LCSM 挑战](https://oreil.ly/SONgC)中所述，此练习的目标是在给定的 FASTA 文件中找到所有序列共享的最长子字符串。在第八章中，我正在搜索一些序列中的给定基序。在这个挑战中，我不知道*先验*地存在任何共享的基序——更不用说它的大小或组成——所以我将只是寻找每个序列中存在的任何长度的序列。这是一个具有挑战性的练习，汇集了我在早期章节中展示的许多想法。我将使用解决方案来探索算法设计、函数、测试和代码组织。

您将学习：

+   如何使用 K-mer 查找共享子序列

+   如何使用`itertools.chain()`连接列表的列表

+   如何以及为什么使用二分搜索

+   最大化函数的一种方式

+   如何在`min()`和`max()`中使用`key`选项

# 入门

所有与此挑战相关的代码和测试都在*10_lcsm*目录中。首先将第一个解决方案复制到`lcsm.py`程序，并请求帮助：

```py
$ cp solution1_kmers_imperative.py lcsm.py
$ ./lcsm.py -h
usage: lcsm.py [-h] FILE

Longest Common Substring

positional arguments:
  FILE        Input FASTA

optional arguments:
  -h, --help  show this help message and exit
```

唯一必需的参数是一个 FASTA 格式的 DNA 序列单文件。与接受文件的其他程序一样，程序将拒绝无效或不可读的输入。以下是我将要使用的第一个输入。这些序列中的最长共同子序列是*CA*，*TA*和*AC*，最后一个在输出中以粗体显示：

```py
$ cat tests/inputs/1.fa
>Rosalind_1
GATTACA
>Rosalind_2
TAGACCA
>Rosalind_3
ATACA
```

这些答案中的任何一个都是可以接受的。使用第一个测试输入运行程序，看看它随机选择了一个可接受的 2-mer：

```py
$ ./lcsm.py tests/inputs/1.fa
CA
```

第二个测试输入要大得多，您会注意到程序花费的时间明显更长。在我的笔记本电脑上，它几乎花了 40 秒。在解决方案中，我将展示一种使用二分搜索显著减少运行时间的方法：

```py
$ time ./lcsm.py tests/inputs/2.fa
GCCTTTTGATTTTAACGTTTATCGGGTGTAGTAAGATTGCGCGCTAATTCCAATAAACGTATGGAGGACATTCCCCGT

real	0m39.244s
user	0m33.708s
sys	0m6.202s
```

虽然这不是挑战的要求，但我已经包含了一个输入文件，其中不包含任何程序应该创建合适响应的共享子序列：

```py
$ ./lcsm.py tests/inputs/none.fa
No common subsequence.
```

从头开始运行`lcsm.py`程序：

```py
$ new.py -fp 'Longest Common Substring' lcsm.py
Done, see new script "lcsm.py".
```

定义参数如下：

```py
class Args(NamedTuple): ![1](img/1.png)
    """ Command-line arguments """
    file: TextIO

def get_args() -> Args:
    """ Get command-line arguments """

    parser = argparse.ArgumentParser(
        description='Longest Common Substring',
        formatter_class=argparse.ArgumentDefaultsHelpFormatter)

    parser.add_argument('file', ![2](img/2.png)
                        help='Input FASTA',
                        metavar='FILE',
                        type=argparse.FileType('rt'))

    args = parser.parse_args()

    return Args(args.file) ![3](img/3.png)
```

![1](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO1-1)

此程序的唯一输入是一个 FASTA 格式的文件。

![2](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO1-2)

定义一个单一的`file`参数。

![3](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO1-3)

返回包含打开文件句柄的`Args`对象。

然后更新`main()`函数以打印传入的文件名：

```py
def main() -> None:
    args = get_args()
    print(args.file.name)
```

确保您看到正确的用法，并且程序正确地打印了文件名：

```py
$ ./lcsm.py tests/inputs/1.fa
tests/inputs/1.fa
```

此时，您的程序应该通过前三个测试。如果您认为自己知道如何完成程序，请继续。如果您想要一些正确方向的提示，请继续阅读。

## 在 FASTA 文件中找到最短的序列

现在应该已经熟悉了读取 FASTA 文件的方法。我将像之前一样使用`Bio.SeqIO.parse()`。我在解决这个问题时的第一个想法是找到共享的 k-mer，同时最大化`k`。最长的子序列不能比文件中最短的序列更长，因此我决定从`k`等于最短的那个开始。找到最短的序列需要我首先扫描*所有*记录。要回顾如何做到这一点，`Bio.SeqIO.parse()`函数返回一个迭代器，让我可以访问每个 FASTA 记录：

```py
>>> from Bio import SeqIO
>>> fh = open('./tests/inputs/1.fa')
>>> recs = SeqIO.parse(fh, 'fasta')
>>> type(recs)
<class 'Bio.SeqIO.FastaIO.FastaIterator'>
```

我可以使用在第四章中首次展示的`next()`函数来强制迭代器生成下一个值，其类型为`SeqRecord`：

```py
>>> rec = next(recs)
>>> type(rec)
<class 'Bio.SeqRecord.SeqRecord'>
```

除了序列本身，FASTA 记录还包含元数据，如序列 ID、名称等：

```py
>>> rec
SeqRecord(seq=Seq('GATTACA'),
          id='Rosalind_1',
          name='Rosalind_1',
          description='Rosalind_1',
          dbxrefs=[])
```

读取信息包装在`Seq`对象中，该对象具有许多有趣和有用的方法，您可以在 REPL 中使用**`help(rec.seq)`**来探索。我只对原始序列感兴趣，因此可以使用`str()`函数将其强制转换为字符串：

```py
>>> str(rec.seq)
'GATTACA'
```

我需要将所有序列都存入列表中，以便可以找到最短序列的长度。由于我将多次使用它们，我可以使用列表推导式将整个文件读入列表：

```py
>>> fh = open('./tests/inputs/1.fa') ![1](img/1.png)
>>> seqs = [str(rec.seq) for rec in SeqIO.parse(fh, 'fasta')] ![2](img/2.png)
>>> seqs
['GATTACA', 'TAGACCA', 'ATACA']
```

![1](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO2-1)

重新打开文件句柄，否则现有文件句柄将从第二次读取开始。

![2](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO2-2)

创建一个列表，强制将每个记录的序列转换为字符串。

序列文件可能包含数百万个读取，并将它们存储在列表中可能会超出可用内存并导致机器崩溃。（问我为什么知道。）问题在于，我需要在下一步中找到所有这些序列中共同的子序列。我有几个*Makefile*目标，将使用*10_lcsm*目录中的`genseq.py`程序生成带有共同模体的大型 FASTA 输入，供您测试。该程序对 Rosalind 提供的数据集运行得很好。

可以使用`map()`函数表达相同的想法：

```py
>>> fh = open('./tests/inputs/1.fa')
>>> seqs = list(map(lambda rec: str(rec.seq), SeqIO.parse(fh, 'fasta')))
>>> seqs
['GATTACA', 'TAGACCA', 'ATACA']
```

要找到最短序列的长度，我需要找到所有序列的长度，可以使用列表推导式来完成：

```py
>>> [len(seq) for seq in seqs]
[7, 7, 5]
```

我更喜欢使用`map()`来写这个更短的方法：

```py
>>> list(map(len, seqs))
[7, 7, 5]
```

Python 内置了`min()`和`max()`函数，可以从列表中返回最小值或最大值：

```py
>>> min(map(len, seqs))
5
>>> max(map(len, seqs))
7
```

因此，最短序列等于长度的最小值：

```py
>>> shortest = min(map(len, seqs))
>>> shortest
5
```

## 从序列中提取 K-mer

最长的共享子序列不可能长于最短序列，并且必须被所有读取共享。 因此，我的下一步是找出所有序列中的所有`k-mer`，从最短序列的长度（`5`）开始。 在第九章中，我编写了一个`find_kmers()`函数和测试，所以我会把那段代码复制到这个程序中。 记得为此导入`typing.List`：

```py
def find_kmers(seq: str, k: int) -> List[str]:
    """ Find k-mers in string """

    n = len(seq) - k + 1
    return [] if n < 1 else [seq[i:i + k] for i in range(n)]

def test_find_kmers() -> None:
    """ Test find_kmers """

    assert find_kmers('', 1) == []
    assert find_kmers('ACTG', 1) == ['A', 'C', 'T', 'G']
    assert find_kmers('ACTG', 2) == ['AC', 'CT', 'TG']
    assert find_kmers('ACTG', 3) == ['ACT', 'CTG']
    assert find_kmers('ACTG', 4) == ['ACTG']
    assert find_kmers('ACTG', 5) == []
```

一个合乎逻辑的方法是从`k`的最大可能值开始，递减计数，直到找到所有序列共享的`k-mer`。 到目前为止，我只使用`range()`函数递增计数。 我可以反转起始和停止值来逆向计数吗？ 显然不行。 如果起始值大于停止值，则`range()`将生成一个空列表：

```py
>>> list(range(shortest, 0))
[]
```

当阅读第七章中的密码子时，我提到`range()`函数接受最多三个参数，最后一个是*步长*，我在那里用来每次跳三个碱基。 这里我需要使用步长`-1`来倒数。 记住停止值不包括在内：

```py
>>> list(range(shortest, 0, -1))
[5, 4, 3, 2, 1]
```

另一种逆向计数的方法是从头开始计数并反转结果：

```py
>>> list(reversed(range(1, shortest + 1)))
[5, 4, 3, 2, 1]
```

无论如何，我都想反复迭代`k`的减少值，直到找到所有序列共享的`k-mer`。 一个序列可能包含多个相同的`k-mer`，因此使用`set()`函数将结果唯一化非常重要：

```py
>>> from lcsm import find_kmers
>>> from pprint import pprint
>>> for k in range(shortest, 0, -1):
...     print(f'==> {k} <==')
...     pprint([set(find_kmers(s, k)) for s in seqs])
...
==> 5 <==
[{'TTACA', 'GATTA', 'ATTAC'}, {'TAGAC', 'AGACC', 'GACCA'}, {'ATACA'}]
==> 4 <==
[{'ATTA', 'TTAC', 'TACA', 'GATT'},
 {'GACC', 'AGAC', 'TAGA', 'ACCA'},
 {'TACA', 'ATAC'}]
==> 3 <==
[{'ACA', 'TAC', 'GAT', 'ATT', 'TTA'},
 {'AGA', 'TAG', 'CCA', 'ACC', 'GAC'},
 {'ACA', 'ATA', 'TAC'}]
==> 2 <==
[{'AC', 'AT', 'CA', 'TA', 'TT', 'GA'},
 {'AC', 'CA', 'CC', 'TA', 'AG', 'GA'},
 {'AC', 'AT', 'CA', 'TA'}]
==> 1 <==
[{'G', 'C', 'T', 'A'}, {'G', 'C', 'T', 'A'}, {'C', 'T', 'A'}]
```

你能看到如何使用这个想法来计算每个`k`值的所有`k-mer`吗？ 寻找频率与序列数匹配的`k-mer`。 如果找到多个，打印任意一个。

# 解决方案

该程序的两个变体使用相同的基本逻辑来查找最长的共享子序列。 第一个版本在输入规模增加时表现不佳，因为它使用逐步线性方法迭代每个可能的`k`长度的序列。 第二个版本引入了二分搜索来找到一个好的`k`的起始值，然后启动一个爬坡搜索来发现`k`的最大值。

## 解决方案 1：计算 K-mer 的频率

在前一节中，我已经找到了所有序列中值为`k`的`k-mer`，从最短序列开始，然后降到`1`。 这里我将从`k`等于`5`开始，这是第一个 FASTA 文件中最短序列的长度：

```py
>>> fh = open('./tests/inputs/1.fa')
>>> seqs = [str(rec.seq) for rec in SeqIO.parse(fh, 'fasta')]
>>> shortest = min(map(len, seqs))
>>> kmers = [set(find_kmers(seq, shortest)) for seq in seqs]
>>> kmers
[{'TTACA', 'GATTA', 'ATTAC'}, {'TAGAC', 'AGACC', 'GACCA'}, {'ATACA'}]
```

我需要一种方法来计算每个`k-mer`在所有序列中出现的次数。 一种方法是使用`collections.Counter()`，我在第一章中首次展示了它：

```py
>>> from collections import Counter
>>> counts = Counter()
```

我可以遍历每个序列的`k-mer`集合，并使用`Counter.update()`方法将它们添加起来：

```py
>>> for group in kmers:
...     counts.update(group)
...
>>> pprint(counts)
Counter({'TTACA': 1,
         'GATTA': 1,
         'ATTAC': 1,
         'TAGAC': 1,
         'AGACC': 1,
         'GACCA': 1,
         'ATACA': 1})
```

或者我可以使用`itertools.chain()`将许多`k-mer`列表连接成单个列表：

```py
>>> from itertools import chain
>>> list(chain.from_iterable(kmers))
['TTACA', 'GATTA', 'ATTAC', 'TAGAC', 'AGACC', 'GACCA', 'ATACA']
```

将此作为`Counter()`的输入会产生相同的集合，显示每个 5-mer 都是唯一的，每次出现一次：

```py
>>> counts = Counter(chain.from_iterable(kmers))
>>> pprint(counts)
Counter({'TTACA': 1,
         'GATTA': 1,
         'ATTAC': 1,
         'TAGAC': 1,
         'AGACC': 1,
         'GACCA': 1,
         'ATACA': 1})
```

`Counter()`在底层是一个常规字典，这意味着我可以访问所有字典方法。我想通过`dict.items()`方法迭代键和值作为一对，使用计数的 k-mers 等于序列数量：

```py
>>> n = len(seqs)
>>> candidates = []
>>> for kmer, count in counts.items():
...     if count == n:
...         candidates.append(kmer)
...
>>> candidates
[]
```

当`k`为`5`时，没有候选序列，因此我需要尝试较小的值。由于我知道正确的答案是`2`，所以我将使用`k=2`重新运行此代码以生成此字典：

```py
>>> k = 2
>>> kmers = [set(find_kmers(seq, k)) for seq in seqs]
>>> counts = Counter(chain.from_iterable(kmers))
>>> pprint(counts)
Counter({'CA': 3,
         'AC': 3,
         'TA': 3,
         'GA': 2,
         'AT': 2,
         'TT': 1,
         'AG': 1,
         'CC': 1})
```

从这里，我找到了三个候选的 2-mer，它们的频率为 3，这等于序列的数量：

```py
>>> candidates = []
>>> for kmer, count in counts.items():
...     if count == n:
...         candidates.append(kmer)
...
>>> candidates
['CA', 'AC', 'TA']
```

不管我选择哪个候选者，我都会使用`random.choice()`函数，该函数从一个选择列表中返回一个值：

```py
>>> import random
>>> random.choice(candidates)
'AC'
```

我喜欢这个方向，所以我想把它放到一个函数中，这样我就可以测试它：

```py
def common_kmers(seqs: List[str], k: int) -> List[str]:
    """ Find k-mers common to all sequences """

    kmers = [set(find_kmers(seq, k)) for seq in seqs]
    counts = Counter(chain.from_iterable(kmers))
    n = len(seqs) ![1](img/1.png)
    return [kmer for kmer, freq in counts.items() if freq == n] ![2](img/2.png)
```

![1](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO3-1)

找到序列的数量。

![2](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO3-2)

返回频率等于序列数量的 k-mers。

这使得`main()`非常易读：

```py
import random
import sys

def main() -> None:
    args = get_args()
    seqs = [str(rec.seq) for rec in SeqIO.parse(args.file, 'fasta')] ![1](img/1.png)
    shortest = min(map(len, seqs)) ![2](img/2.png)

    for k in range(shortest, 0, -1): ![3](img/3.png)
        if kmers := common_kmers(seqs, k): ![4](img/4.png)
            print(random.choice(kmers)) ![5](img/5.png)
            sys.exit(0) ![6](img/6.png)

    print('No common subsequence.') ![7](img/7.png)
```

![1](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO4-1)

将所有序列读入列表。

![2](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO4-2)

找到最短序列的长度。

![3](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO4-3)

从最短序列倒数。

![4](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO4-4)

使用这个`k`值找到所有共同的 k-mers。

![5](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO4-5)

如果找到任何 k-mers，则打印一个随机选择。

![6](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO4-6)

使用退出值为`0`（无错误）退出程序。

![7](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO4-7)

如果我到达这一点，请通知用户没有共享的序列。

在上述代码中，我再次使用了我在第五章介绍的海象运算符（`:=`）来首先将调用`common_kmers()`的结果赋给变量`kmers`，然后评估`kmers`的真实性。如果`kmers`是真实的，即意味着找到了这个`k`值的共同 k-mers，Python 将只在下一个块中进入。在引入这种语言特性之前，我必须写两行赋值和评估，如下所示：

```py
kmers = common_kmers(seqs, k)
if kmers:
    print(random.choice(kmers))
```

## 解决方案 2：使用二分搜索加快速度

正如本章开头所述，随着输入大小的增加，该解决方案的增长速度会变得更慢。跟踪程序进度的一种方法是在`for`循环的开头放置一个`print(k)`语句。用第二个输入文件运行此命令，你会看到它从 1,000 开始倒数，直到 k 达到 78 才达到正确的值。

逆向计数 1 太慢了。如果你的朋友让你猜一个介于 1 和 1,000 之间的数字，你不会从 1,000 开始，每次朋友说“太高”时就猜少 1。选择 500 会更快（也更有利于你们的友谊）。如果你的朋友选择了 453，他们会说“太高”，所以你聪明地选择了 250。他们会回答“太低”，然后你继续在你最后一次高低猜测的中间值之间分割差值，直到找到正确答案。这就是*二分查找*，它是快速从排序值列表中查找所需值位置的绝佳方法。

为了更好地理解这一点，我在 *10_lcsm* 目录中包含了一个名为`binsearch.py`的程序：

```py
$ ./binsearch.py -h
usage: binsearch.py [-h] -n int -m int

Binary Search

optional arguments:
  -h, --help         show this help message and exit
  -n int, --num int  The number to guess (default: None)
  -m int, --max int  The maximum range (default: None)
```

下面是程序的相关部分。如果你愿意，可以阅读参数定义的源代码。`binary_search()`函数是递归的，就像第四章中对斐波那契数列问题的一种解决方案一样。请注意，为了使二分查找起作用，搜索值必须是排序的，`range()`函数提供了这个功能：

```py
def main() -> None:
    args = get_args()
    nums = list(range(args.maximum + 1))
    pos = binary_search(args.num, nums, 0, args.maximum)
    print(f'Found {args.num}!' if pos > 0 else f'{args.num} not present.')

def binary_search(x: int, xs: List[int], low: int, high: int) -> int:
    print(f'{low:4} {high:4}', file=sys.stderr)

    if high >= low: ![1](img/1.png)
        mid = (high + low) // 2 ![2](img/2.png)

        if xs[mid] == x: ![3](img/3.png)
            return mid

        if xs[mid] > x: ![4](img/4.png)
            return binary_search(x, xs, low, mid - 1) ![5](img/5.png)

        return binary_search(x, xs, mid + 1, high) ![6](img/6.png)

    return -1 ![7](img/7.png)
```

![1](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO5-1)

退出递归的基本情况是当此条件为假时。

![2](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO5-2)

中点是`high`和`low`之间的中间值，使用地板除法。

![3](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO5-3)

如果元素在中间，则返回中点。

![4](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO5-4)

看看中点的值是否大于所需值。

![5](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO5-5)

搜索较低的值。

![6](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO5-6)

搜索较高的值。

![7](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO5-7)

未找到该值。

`binary_search()`函数中的`x`和`xs`名称意为单数和复数。在我的脑海中，我将它们发音为*ex*和*exes*。这种符号在纯函数式编程中很常见，因为我不试图描述`x`是什么类型的值。它可以是字符串、数字或其他任何类型的值。重要的是`xs`是某种类型的全部可比较值的集合。

我加入了一些`print()`语句，这样，使用前面的数字运行，您可以看到`low`和`high`在 10 步中最终收敛到目标数字：

```py
$ ./binsearch.py -n 453 -m 1000
   0 1000
   0  499
 250  499
 375  499
 438  499
 438  467
 453  467
 453  459
 453  455
 453  453
Found 453!
```

只需八次迭代即可确定数字不存在：

```py
$ ./binsearch.py -n 453 -m 100
   0  100
  51  100
  76  100
  89  100
  95  100
  98  100
 100  100
 101  100
453 not present.
```

二分搜索可以告诉我值是否存在于值列表中，但这并不是我的问题。虽然我相当确定大多数数据集中至少会有一个 2-mer 或 1-mer 的共同点，但我包含了一个没有这种共同点的文件：

```py
$ cat tests/inputs/none.fa
>Rosalind_1
GGGGGGG
>Rosalind_2
AAAAAAAA
>Rosalind_3
CCCC
>Rosalind_4
TTTTTTTT
```

如果有一个可接受的`k`值，那么我需要找到*最大*值。我决定使用二分搜索来找到一个起始点，以进行寻找最大值的爬坡搜索。首先我会展示`main()`，然后我会分解其他函数：

```py
def main() -> None:
    args = get_args()
    seqs = [str(rec.seq) for rec in SeqIO.parse(args.file, 'fasta')] ![1](img/1.png)
    shortest = min(map(len, seqs)) ![2](img/2.png)
    common = partial(common_kmers, seqs) ![3](img/3.png)
    start = binary_search(common, 1, shortest) ![4](img/4.png)

    if start >= 0: ![5](img/5.png)
        candidates = [] ![6](img/6.png)
        for k in range(start, shortest + 1): ![7](img/7.png)
            if kmers := common(k): ![8](img/8.png)
                candidates.append(random.choice(kmers)) ![9](img/9.png)
            else:
                break ![10](img/10.png)

        print(max(candidates, key=len)) ![11](img/11.png)
    else:
        print('No common subsequence.') ![12](img/12.png)
```

![1](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO6-1)

将序列列表作为字符串获取。

![2](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO6-2)

找到最短序列的长度。

![3](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO6-3)

部分应用`common_kmers()`函数与`seqs`输入。

![4](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO6-4)

使用二分搜索找到给定函数的起始点，使用`1`作为`k`的最小值，使用最短序列长度为最大值。

![5](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO6-5)

检查二分搜索是否找到了有用的东西。

![6](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO6-6)

初始化候选值列表。

![7](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO6-7)

以二分搜索结果开始爬坡搜索。

![8](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO6-8)

检查是否存在共同的 k-mer。

![9](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO6-9)

如果是这样，随机向候选列表添加一个。

![10](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO6-10)

如果没有共同的 k-mer，则退出循环。

![11](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO6-11)

选择具有最长长度的候选序列。

![12](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO6-12)

告诉用户没有答案。

虽然在上述代码中有许多要解释的事情，但我想强调对`max()`的调用。我之前展示了该函数将从列表中返回最大值。通常您可能会考虑在数字列表上使用它：

```py
>>> max([4, 2, 8, 1])
8
```

在上述代码中，我想在列表中找到最长的字符串。我可以使用`map()`函数映射`len()`函数来找到它们的长度：

```py
>>> seqs = ['A', 'CC', 'GGGG', 'TTT']
>>> list(map(len, seqs))
[1, 2, 4, 3]
```

这表明第三个序列*GGGG*是最长的。`max()`函数接受一个可选的`key`参数，该参数是在比较之前应用于每个元素的函数。如果我使用`len()`函数，那么`max()`可以正确地识别出最长的序列：

```py
>>> max(seqs, key=len)
'GGGG'
```

让我们看看我如何修改`binary_search()`函数以适应我的需求：

```py
def binary_search(f: Callable, low: int, high: int) -> int: ![1](img/1.png)
    """ Binary search """

    hi, lo = f(high), f(low) ![2](img/2.png)
    mid = (high + low) // 2 ![3](img/3.png)

    if hi and lo: ![4](img/4.png)
        return high

    if lo and not hi: ![5](img/5.png)
        return binary_search(f, low, mid)

    if hi and not lo: ![6](img/6.png)
        return binary_search(f, mid, high)

    return -1 ![7](img/7.png)
```

![1](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO7-1)

该函数接受另一个函数`f()`以及`low`和`high`值作为参数。在这个例子中，函数`f()`将返回共同的 k-mer，但该函数可以执行任何您想要的计算。

![2](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO7-2)

使用最高和最低的`k`值调用函数`f()`。

![3](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO7-3)

找到`k`的中点值。

![4](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO7-4)

如果函数`f()`对高`k`和低`k`值都找到了共同的 k-mer，则返回最高的 k。

![5](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO7-5)

如果高`k`找不到 k-mer，但低值找到了，则递归调用函数，在更低的`k`值中搜索。

![6](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO7-6)

如果低`k`找不到 k-mer，但高值找到了，则递归调用函数，在更高的`k`值中搜索。

![7](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO7-7)

返回`-1`以指示使用`high`和`low`参数调用`f()`时未找到 k-mer。

这是我为此编写的测试： 

```py
def test_binary_search() -> None:
    """ Test binary_search """

    seqs1 = ['GATTACA', 'TAGACCA', 'ATACA'] ![1](img/1.png)
    f1 = partial(common_kmers, seqs1) ![2](img/2.png)
    assert binary_search(f1, 1, 5) == 2 ![3](img/3.png)

    seqs2 = ['GATTACTA', 'TAGACTCA', 'ATACTA'] ![4](img/4.png)
    f2 = partial(common_kmers, seqs2)
    assert binary_search(f2, 1, 6) == 3 ![5](img/5.png)
```

![1](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO8-1)

这些是我一直在使用的具有三个共享 2-mer 的序列。

![2](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO8-2)

定义一个函数来找到第一组序列中的 k-mers。

![3](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO8-3)

搜索找到`k`为`2`是正确答案。

![4](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO8-4)

与以前相同的序列，但现在共享 3-mer。

![5](img/#co_finding_the_longest_shared_subsequence__finding_k_mers__writing_functions__and_using_binary_search_CO8-5)

搜索找到`k`为`3`。

与前面的二分搜索不同，我的版本不一定会返回*精确*答案，只是一个不错的起点。如果没有任何大小为`k`的共享序列，则我会告知用户：

```py
$ ./solution2_binary_search.py tests/inputs/none.fa
No common subsequence.
```

如果有共享子序列，则此版本运行速度显著提高，可能快 28 倍：

```py
$ hyperfine -L prg ./solution1_kmers_functional.py,./solution2_binary_search.py\
  '{prg} tests/inputs/2.fa'
Benchmark #1: ./solution1_kmers_functional.py tests/inputs/2.fa
  Time (mean ± σ):     40.686 s ±  0.443 s    [User: 35.208 s, System: 6.042 s]
  Range (min … max):   40.165 s … 41.349 s    10 runs

Benchmark #2: ./solution2_binary_search.py tests/inputs/2.fa
  Time (mean ± σ):      1.441 s ±  0.037 s    [User: 1.903 s, System: 0.255 s]
  Range (min … max):    1.378 s …  1.492 s    10 runs

Summary
  './solution2_binary_search.py tests/inputs/2.fa' ran
   28.24 ± 0.79 times faster than './solution1_kmers_functional.py
   tests/inputs/2.fa'
```

当我从最大的`k`值开始搜索并向下迭代时，我正在执行对所有可能值的*线性*搜索。这意味着搜索时间与值的数量*n*成比例增长（线性增长）。相比之下，二分搜索的增长率是对数*log n*。通常用*大 O*表示法来讨论算法的运行时增长，因此您可能会看到二分搜索描述为 O(log *n*)，而线性搜索是 O(*n*)，这要糟糕得多。

# 更进一步

与第九章中的建议一样，添加一个汉明距离选项，可以在决定共享 k-mer 时允许指定数量的差异。

# 复习

本章的关键点：

+   K-mer 可以用来找到序列的保守区域。

+   列表的列表可以使用`itertools.chain()`组合成单个列表。

+   可以在排序值上使用二分搜索来比线性搜索更快地找到值。

+   爬山法是最大化函数输入的一种方式。

+   `min()`和`max()`的`key`选项是在比较之前应用于值的函数。
