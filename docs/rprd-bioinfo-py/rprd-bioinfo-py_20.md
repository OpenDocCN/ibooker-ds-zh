# 第18章. FASTX Sampler：随机子抽样序列文件

在基因组学和宏基因组学中，序列数据集可能会变得非常庞大，需要大量时间和计算资源来进行分析。许多测序仪每个样本可以生成数千万个读取，许多实验涉及数十到数百个样本，每个样本具有多个技术重复，导致数据达到几个GB到TB的量级。通过随机子抽样序列减少输入文件的大小，可以更快地探索数据。在本章中，我将展示如何使用Python的`random`模块从FASTA/FASTQ序列文件中选择部分读取。

您将学习到：

+   非确定性抽样

# 入门

此练习的代码和测试位于*18_fastx_sampler*目录中。首先复制名为`sampler.py`的程序解决方案：

```py
$ cd 18_fastx_sampler/
$ cp solution.py sampler.py
```

用于测试该程序的FASTA输入文件将由您在[第17章](ch17.html#ch17)中编写的`synth.py`程序生成。如果您没有完成编写该程序，请务必在执行**`make fasta`**以创建包含1K、10K和100K读取数的三个75到200 bp长度文件（分别命名为*n1k.fa*、*n10k.fa*和*n100k.fa*）之前将解决方案复制到该文件名中。使用`seqmagique.py`来验证文件的正确性：

```py
$ ../15_seqmagique/seqmagique.py tests/inputs/n1*
name                     min_len    max_len    avg_len    num_seqs
tests/inputs/n100k.fa         75        200     136.08      100000
tests/inputs/n10k.fa          75        200     136.13       10000
tests/inputs/n1k.fa           75        200     135.16        1000
```

运行`sampler.py`以选择最小文件中默认的10%序列。如果使用随机种子`1`，您应该得到95个读取：

```py
$ ./sampler.py -s 1 tests/inputs/n1k.fa
Wrote 95 sequences from 1 file to directory "out"
```

结果可以在名为*out*的输出目录中的*n1k.fa*文件中找到。验证方法之一是使用**`grep -c`**来计算每个记录开头的符号`>`出现的次数：

```py
$ grep -c '>' out/n1k.fa
95
```

注意，如果您忘记在`>`周围加上引号，则会等待您一个顽固的错误：

```py
$ grep -c > out/n1k.fa
usage: grep [-abcDEFGHhIiJLlmnOoqRSsUVvwxZ] [-A num] [-B num] [-C[num]]
	[-e pattern] [-f file] [--binary-files=value] [--color=when]
	[--context[=num]] [--directories=action] [--label] [--line-buffered]
	[--null] [pattern] [file ...]
```

等等，发生了什么？请记住`>`是`bash`中将`STDOUT`从一个程序重定向到文件的运算符。在前面的命令中，我没有足够的参数运行`grep`并将输出重定向到*out/n1k.fa*。您看到的输出是打印到`STDERR`的用法。什么也没有打印到`STDOUT`，所以这个空输出覆盖了*out/n1k.fa*文件，现在它是空的：

```py
$ wc out/n1k.fa
       0       0       0 out/n1k.fa
```

我特别指出这一点是因为我曾经由于这个问题丢失了几个序列文件。数据已经永久丢失，所以我必须重新运行之前的命令来重新生成文件。在这之后，我建议您使用`seqmagique.py`来验证内容：

```py
$ ../15_seqmagique/seqmagique.py out/n1k.fa
name          min_len    max_len    avg_len    num_seqs
out/n1k.fa         75        200     128.42          95
```

## 回顾程序参数

这是一个非常复杂的程序，具有许多选项。运行`sampler.py`程序请求帮助。请注意，唯一需要的参数是输入文件，因为所有选项都设置为合理的默认值：

```py
$ ./sampler.py -h
usage: sampler.py [-h] [-f format] [-p reads] [-m max] [-s seed] [-o DIR]
                  FILE [FILE ...]

Probabilistically subset FASTA files

positional arguments:
  FILE                  Input FASTA/Q file(s) ![1](assets/1.png)

optional arguments:
  -h, --help            show this help message and exit
  -f format, --format format
                        Input file format (default: fasta) ![2](assets/2.png)
  -p reads, --percent reads
                        Percent of reads (default: 0.1) ![3](assets/3.png)
  -m max, --max max     Maximum number of reads (default: 0) ![4](assets/4.png)
  -s seed, --seed seed  Random seed value (default: None) ![5](assets/5.png)
  -o DIR, --outdir DIR  Output directory (default: out) ![6](assets/6.png)
```

[![1](assets/1.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO1-1)

需要一个或多个*FASTA*或*FASTQ*文件。

[![2](assets/2.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO1-2)

输入文件的默认序列格式是*FASTA*。

[![3](assets/3.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO1-3)

默认情况下，程序将选择10%的读取。

[![4](assets/4.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO1-4)

这个选项会在达到指定的最大值时停止抽样。

[![5](assets/5.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO1-5)

此选项将随机种子设置为重现选择。

[![6](assets/6.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO1-6)

输出文件的默认目录是*out*。

与先前的程序一样，程序将拒绝无效或不可读的输入文件，随机种子参数必须是一个整数值。`-p|--percent`选项应该是介于0和1之间（不包括0和1）的浮点值，程序将拒绝超出此范围的任何内容。我手动验证此参数并使用`parser.error()`，就像第[4](ch04.html#ch04)章和第[9](ch09.html#ch09)章一样：

```py
$ ./sampler.py -p 3 tests/inputs/n1k.fa
usage: sampler.py [-h] [-f format] [-p reads] [-m max] [-s seed] [-o DIR]
                  FILE [FILE ...]
sampler.py: error: --percent "3.0" must be between 0 and 1
```

`-f|--format`选项只接受值`fasta`或`fastq`，默认为第一个。我使用`argparse`的`choices`选项，就像第[15](ch15.html#ch15)章和第[16](ch16.html#ch16)章一样，自动拒绝不需要的值。例如，程序将拒绝`fastb`的值：

```py
$ ./sampler.py -f fastb tests/inputs/n1k.fa
usage: sampler.py [-h] [-f format] [-p reads] [-m max] [-s seed] [-o DIR]
                  FILE [FILE ...]
sampler.py: error: argument -f/--format: invalid choice:
'fastb' (choose from 'fasta', 'fastq')
```

最后，`-m|--max`选项默认为`0`，意味着程序会无上限地抽样约`--percent`的读取。实际上，你可能有数千万个读取的输入文件，但你只希望每个文件最多有10万个。使用这个选项在达到所需数量时停止抽样。例如，我可以使用`-m 30`在30个读取时停止抽样：

```py
$ ./sampler.py -m 30 -s 1 tests/inputs/n1k.fa
  1: n1k.fa
Wrote 30 sequences from 1 file to directory "out"
```

当您认为理解程序应该如何工作时，请重新开始使用您的版本：

```py
$ new.py -fp 'Probabilistically subset FASTA files' sampler.py
Done, see new script "sampler.py".
```

## 定义参数

程序的参数包含许多不同的数据类型，我用以下类表示：

```py
class Args(NamedTuple):
    """ Command-line arguments """
    files: List[TextIO] ![1](assets/1.png)
    file_format: str ![2](assets/2.png)
    percent: float ![3](assets/3.png)
    max_reads: int ![4](assets/4.png)
    seed: Optional[int] ![5](assets/5.png)
    outdir: str ![6](assets/6.png)
```

[![1](assets/1.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO2-1)

[![2](assets/2.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO2-2)

文件是一组打开的文件句柄列表。

输入文件格式是一个字符串。

[![3](assets/3.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO2-3)

读取的百分比是一个浮点值。

[![4](assets/4.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO2-4)

最大读取数是一个整数。

[![5](assets/5.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO2-5)

随机种子值是一个可选的整数。

[![6](assets/6.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO2-6)

输出目录名称是一个字符串。

这是我如何使用`argparse`定义参数的方式：

```py
def get_args() -> Args:
    parser = argparse.ArgumentParser(
        description='Probabilistically subset FASTA files',
        formatter_class=argparse.ArgumentDefaultsHelpFormatter)

    parser.add_argument('file',
                        metavar='FILE',
                        type=argparse.FileType('r'), ![1](assets/1.png)
                        nargs='+',
                        help='Input FASTA/Q file(s)')

    parser.add_argument('-f',
                        '--format',
                        help='Input file format',
                        metavar='format',
                        type=str,
                        choices=['fasta', 'fastq'], ![2](assets/2.png)
                        default='fasta')

    parser.add_argument('-p',
                        '--percent',
                        help='Percent of reads',
                        metavar='reads',
                        type=float, ![3](assets/3.png)
                        default=.1)

    parser.add_argument('-m',
                        '--max',
                        help='Maximum number of reads',
                        metavar='max',
                        type=int,
                        default=0) ![4](assets/4.png)

    parser.add_argument('-s',
                        '--seed',
                        help='Random seed value',
                        metavar='seed',
                        type=int,
                        default=None) ![5](assets/5.png)

    parser.add_argument('-o',
                        '--outdir',
                        help='Output directory',
                        metavar='DIR',
                        type=str,
                        default='out') ![6](assets/6.png)

    args = parser.parse_args()

    if not 0 < args.percent < 1: ![7](assets/7.png)
        parser.error(f'--percent "{args.percent}" must be between 0 and 1')

    if not os.path.isdir(args.outdir): ![8](assets/8.png)
        os.makedirs(args.outdir)

    return Args(files=args.file, ![9](assets/9.png)
                file_format=args.format,
                percent=args.percent,
                max_reads=args.max,
                seed=args.seed,
                outdir=args.outdir)
```

[![1](assets/1.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO3-1)

将文件输入定义为一个或多个可读文本文件。

[![2](assets/2.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO3-2)

使用`choices`来限制文件格式，默认为`fasta`。

[![3](assets/3.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO3-3)

百分比参数是浮点数，默认为10%。

[![4](assets/4.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO3-4)

最大读取次数应为整数，默认为`0`。

[![5](assets/5.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO3-5)

随机种子值是可选的，但如果存在则应为有效整数。

[![6](assets/6.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO3-6)

输出目录是一个字符串，默认值为`out`。

[![7](assets/7.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO3-7)

验证百分比应在0到1之间。

[![8](assets/8.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO3-8)

如果输出目录不存在，则创建该输出目录。

[![9](assets/9.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO3-9)

返回`Args`对象。

注意，虽然程序接受FASTA和FASTQ输入，但应仅编写FASTA格式的输出文件。

## 非确定性采样

由于序列没有固有的排序，人们可能会试图获取用户指定的前几个序列。例如，我可以使用`head`从每个文件中选择一些行。这对FASTQ文件有效，只要该数字是四的倍数，但这种方法可能会为大多数其他序列格式（如多行FASTA或Swiss-Prot）创建无效文件。

我展示了几个从文件中读取并选择序列的程序，因此我可以重新使用其中一个来选择记录，直到达到所需的数量。当老板首次要求我编写这个程序时，我确实这样做了。然而，输出是无用的，因为我没有意识到输入是同事创建的合成数据集，用于模拟*宏基因组*，即由未知生物组成的环境样本。输入文件是通过连接来自已知基因组的各种读取而创建的，因此，例如，前10K读取来自细菌，接下来的10K来自另一种细菌，接下来的10K来自代表性古细菌，接下来的10K来自病毒，依此类推。仅获取前*N*条记录未能包含输入的多样性。编写这个程序不仅无聊，而且更糟糕的是，它总是生成相同的输出，因此无法用于生成不同的子样本。这是一个*确定性*程序的示例，因为给定相同的输入，输出始终相同。

因为我需要找到一种方法来随机选择一定百分比的读取，我的第一个想法是计算读取的数量，以便我可以弄清楚有多少是，例如，10%。为此，我将所有序列存储在一个列表中，使用`len()`来确定存在多少个，然后随机选择该范围内的10%的数字。虽然这种方法对于非常小的输入文件可能有效，但我希望您能看出它在任何有意义的方式上都无法扩展。遇到包含数千万读取的输入文件并不罕见。在类似Python列表的数据结构中保持所有这些数据可能需要比机器上可用内存更多的内存。

最终，我选择了一种方案，每次只读取一个序列，然后随机决定是选择还是拒绝它。也就是说，对于每个序列，我从一个*连续均匀分布*中随机选择一个数字，该分布在0到1之间，意味着这个范围内的所有值被选中的可能性是相等的。如果该数字小于或等于给定的百分比，我选择这个读取。这种方法一次只在内存中保持一个序列记录，因此应该至少是线性或*O(n)*可扩展的。

为了演示选择过程，我将导入`random`模块，并使用`random.random()`函数选择0到1之间的一个数字：

```py
>>> import random
>>> random.random()
0.465289867914331
```

很难让你得到和我一样的数字。我们必须就种子达成一致，才能产生相同的值。使用整数`1`，你应该得到这个数字：

```py
>>> random.seed(1)
>>> random.random()
0.13436424411240122
```

`random.random()`函数使用均匀分布。`random`模块还可以从其他分布中进行采样，如正态或高斯分布。查阅`help(random)`以了解这些其他函数及其使用方法。

当我遍历每个文件中的序列时，我使用此函数来选择一个数字。如果该数字小于或等于所选百分比，我希望将该序列写入输出文件。也就是说，`random.random()`函数应该大约有10%的时间产生小于或等于`.10`的数字。通过这种方式，我使用了一种*非确定性*的采样方法，因为每次运行程序时（假设我没有设置随机种子），所选的读取将会有所不同。这使我能够从同一输入文件中生成许多不同的子样本，这在生成技术重复用于分析时可能会很有用。

## 程序的结构化

您可能会对这个程序的复杂性感到有些不知所措，因此我将提供您可能会发现有帮助的伪代码：

```py
set the random seed
iterate through each input file
    set the output filename to output directory plus the input file's basename
    open the output filehandle
    initialize a counter for how many records have been taken

    iterate through each record of the input file
        select a random number between 0 and 1
        if this number is less than or equal to the percent
            write the sequence in FASTA format to the output filehandle
            increment the counter for taking records

        if there is a max number of records and the number taken is equal
            leave the loop

    close the output filehandle

print how many sequences were taken from how many files and the output location
```

我鼓励您在一个文件上运行`solution.py`程序，然后在多个文件上运行，并尝试逆向工程输出。继续运行测试套件以确保您的程序正常运行。

希望您能看到此程序在结构上与许多先前处理某些输入文件并创建某些输出的程序非常相似。例如，在[第2章](ch02.html#ch02)中，您处理了DNA序列文件以在输出目录中生成RNA序列文件。在[第15章](ch15.html#ch15)中，您处理了序列文件以生成统计摘要表。在[第16章](ch16.html#ch16)中，您处理了序列文件以选择符合某一模式的记录，并将选定的序列写入输出文件。第2章和第16章中的程序最接近您需要在这里做的事情，因此我建议您借鉴这些解决方案。

# 解决方案

我想分享两个解决方案的版本。第一个解决方案致力于解决所描述的确切问题。第二个解决方案超出了原始要求，因为我想向您展示如何解决处理大型生物信息数据集时可能面临的两个常见问题，即打开过多文件句柄和读取压缩文件。

## 解决方案 1：读取常规文件

如果您处理的是少量未压缩的输入文件，则以下解决方案是合适的：

```py
def main() -> None:
    args = get_args()
    random.seed(args.seed) ![1](assets/1.png)

    total_num = 0 ![2](assets/2.png)
    for i, fh in enumerate(args.files, start=1): ![3](assets/3.png)
        basename = os.path.basename(fh.name)
        out_file = os.path.join(args.outdir, basename)
        print(f'{i:3}: {basename}') ![4](assets/4.png)

        out_fh = open(out_file, 'wt') ![5](assets/5.png)
        num_taken = 0 ![6](assets/6.png)

        for rec in SeqIO.parse(fh, args.file_format): ![7](assets/7.png)
            if random.random() <= args.percent: ![8](assets/8.png)
                num_taken += 1
                SeqIO.write(rec, out_fh, 'fasta')

            if args.max_reads and num_taken == args.max_reads: ![9](assets/9.png)
                break

        out_fh.close() ![10](assets/10.png)
        total_num += num_taken

    num_files = len(args.files) ![11](assets/11.png)
    print(f'Wrote {total_num:,} sequence{"" if total_num == 1 else "s"} '
          f'from {num_files:,} file{"" if num_files == 1 else "s"} '
          f'to directory "{args.outdir}"')
```

[![1](assets/1.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO4-1)

设置随机种子（如果存在）。默认的`None`值与不设置种子相同。

[![2](assets/2.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO4-2)

初始化一个变量以记录选择的总序列数。

[![3](assets/3.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO4-3)

遍历每个输入文件句柄。

[![4](assets/4.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO4-4)

通过将输出目录与文件的基本名称连接来构造输出文件名。

[![5](assets/5.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO4-5)

打开输出文件句柄以写入文本。

[![6](assets/6.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO4-6)

初始化一个变量以记录从此文件中获取的序列数。

[![7](assets/7.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO4-7)

遍历输入文件中的每个序列记录。

[![8](assets/8.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO4-8)

如果记录是随机选择的，则递增计数器并将序列写入输出文件。

[![9](assets/9.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO4-9)

如果定义了最大限制并且选择的记录数等于此限制，则退出内部`for`循环。

[![10](assets/10.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO4-10)

关闭输出文件句柄并递增总记录数。

[![11](assets/11.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO4-11)

注意处理的文件数并告知用户最终状态。

## 解决方案 2：读取大量压缩文件

最初的问题不涉及读取压缩文件，但通常会发现数据以这种方式存储，以节省数据传输的带宽和存储数据的磁盘空间。Python可以直接读取使用诸如`zip`和`gzip`等工具压缩的文件，因此在处理之前不需要解压缩输入文件。

另外，如果您正在处理数百到数千个输入文件，您会发现使用`type=argparse.FileType()`将导致程序失败，因为您可能会超出操作系统允许的最大打开文件数。在这种情况下，您应将`Args.files`声明为`List[str]`并像这样创建参数：

```py
parser.add_argument('file',
                    metavar='FILE',
                    type=str, ![1](assets/1.png)
                    nargs='+',
                    help='Input FASTA/Q file(s)')
```

[![1](assets/1.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO5-1)

将参数类型设置为一个或多个字符串值。

这意味着您需要自行验证输入文件，可以在`get_args()`函数中执行，如下所示：

```py
if bad_files := [file for file in args.file if not os.path.isfile(file)]: ![1](assets/1.png)
    parser.error(f'Invalid file: {", ".join(bad_files)}') ![2](assets/2.png)
```

[![1](assets/1.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO6-1)

查找所有不是有效文件的参数。

[![2](assets/2.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO6-2)

使用`parser.error()`报告不良输入。

`main()`处理需要稍作更改，因为现在`args.files`将是一个字符串列表。您需要自行使用`open()`打开文件句柄，这是处理压缩文件所需的关键更改。我使用了一个简单的启发式方法来检查文件扩展名是否为`.gz`，以确定文件是否被压缩，并将使用`gzip.open()`函数来打开它：

```py
def main() -> None:
    args = get_args()
    random.seed(args.seed)

    total_num = 0
    for i, file in enumerate(args.files, start=1): ![1](assets/1.png)
        basename = os.path.basename(file)
        out_file = os.path.join(args.outdir, basename)
        print(f'{i:3}: {basename}')

        ext = os.path.splitext(basename)[1] ![2](assets/2.png)
        fh = gzip.open(file, 'rt') if ext == '.gz' else open(file, 'rt') ![3](assets/3.png)
        out_fh = open(out_file, 'wt')
        num_taken = 0

        for rec in SeqIO.parse(fh, args.file_format):
            if random.random() <= args.percent:
                num_taken += 1
                SeqIO.write(rec, out_fh, 'fasta')

            if args.max_reads and num_taken == args.max_reads:
                break

        out_fh.close()
        total_num += num_taken

    num_files = len(args.files)
    print(f'Wrote {total_num:,} sequence{"" if total_num == 1 else "s"} '
          f'from {num_files:,} file{"" if num_files == 1 else "s"} '
          f'to directory "{args.outdir}".')
```

[![1](assets/1.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO7-1)

`args.files`现在是一个字符串列表，而不是文件句柄。

[![2](assets/2.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO7-2)

获取文件扩展名。

[![3](assets/3.png)](#co_fastx_sampler__randomly_subsampling_sequence_files_CO7-3)

如果文件扩展名是`.gz`，则使用`gzip.open()`打开文件；否则，使用普通的`open()`函数。

最后，有时`nargs='+'`也不起作用。对于一个项目，我不得不下载超过35万个XML文件。将所有这些作为参数传递将导致命令行本身出现“Argument list too long”的错误。我的解决方法是将目录名称作为参数接受：

```py
parser.add_argument('-d',
                    '--dir',
                    metavar='DIR',
                    type=str,
                    nargs='+',
                    help='Input directories of FASTA/Q file(s)')
```

然后我使用Python递归搜索目录中的文件。对于这段代码，我添加了`from pathlib import Path`，以便我可以使用`Path.rglob()`函数：

```py
files = []
for dirname in args.dir:
    if os.path.isdir(dirname):
        files.extend(list(Path(dirname).rglob('*')))

if not files:
    parser.error('Found no files')

return Args(files=files,
            file_format=args.format,
            percent=args.percent,
            max_reads=args.max,
            seed=args.seed,
            outdir=args.outdir)
```

程序可以像以前一样继续运行，因为Python在列表中存储几十万项没有问题。

# 进一步探索

该程序始终产生FASTA格式的输出。添加一个`--outfmt`输出格式选项，以便您可以指定输出格式。考虑检测输入文件格式，并像您在[第16章](ch16.html#ch16)中所做的那样以相同的方式编写输出格式。务必添加适当的测试以验证程序的功能。

# **复习**

+   FASTA文件中的`>`记录标记也是`bash`中的重定向操作符，因此在命令行中必须谨慎引用此值。

+   确定性方法始终为给定输入产生相同的输出。非确定性方法对相同的输入产生不同的输出。

+   `random`模块具有从各种分布（如均匀分布和正态分布）中选择数字的函数。
