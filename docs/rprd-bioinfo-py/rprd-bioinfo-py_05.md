# 第4章。创建斐波那契序列：编写、测试和基准算法

编写斐波那契序列的实现是成为编程英雄的旅程中的又一步。[Rosalind斐波那契描述](https://oreil.ly/7vkRw)指出，序列的起源是对一些重要（而不现实）假设进行数学模拟的兔子繁殖：

+   第一个月从一对新生兔子开始。

+   兔子可以在一个月后繁殖。

+   每个月，具有生育能力的每只兔子都与另一只具有生育能力的兔子交配。

+   兔子交配后一个月，它们产生与同等大小的一窝幼崽。

+   兔子是不朽的，永远不会停止交配。

序列始终以数字0和1开始。随后的数字可以通过在列表中添加前两个立即前值来生成*无限*，如[图4-1](#fig_4.1)所示。

![mpfb 0401](assets/mpfb_0401.png)

###### 图4-1。斐波那契序列的前八个数字——在初始的0和1之后，后续数字是通过将前两个数字相加而创建的

如果你搜索互联网上的解决方案，你会发现有几十种不同的生成序列的方式。我想专注于三种非常不同的方法。第一个解决方案使用*命令式*方法，其中算法严格定义了每一步。下一个解决方案使用*生成器*函数，最后一个将专注于*递归*解决方案。递归虽然有趣，但随着我尝试生成更多的序列，速度会显着减慢，但事实证明性能问题可以通过缓存来解决。

你将学到：

+   如何手动验证参数并抛出错误

+   如何使用列表作为堆栈

+   如何编写生成器函数

+   如何编写递归函数

+   递归函数为什么可能会慢，以及如何使用记忆化来修复这个问题

+   如何使用函数装饰器

# 入门

此章节的代码和测试位于*04_fib*目录中。首先将第一个解决方案复制到`fib.py`：

```py
$ cd 04_fib/
$ cp solution1_list.py fib.py
```

要求使用情况以查看参数的定义。你可以使用`n`和`k`，但我选择使用名称`generations`和`litter`：

```py
$ ./fib.py -h
usage: fib.py [-h] generations litter

Calculate Fibonacci

positional arguments:
  generations  Number of generations
  litter       Size of litter per generation

optional arguments:
  -h, --help   show this help message and exit
```

这将是第一个接受非字符串参数的程序。Rosalind挑战指出该程序应该接受两个正整数值：

+   `n` ≤ 40 代表代数的数量

+   `k` ≤ 5 代表配对产生的一窝幼崽的大小

尝试传递非整数值并注意程序的失败：

```py
$ ./fib.py foo
usage: fib.py [-h] generations litter
fib.py: error: argument generations: invalid int value: 'foo'
```

你无法察觉，但除了打印简要的使用说明和有用的错误消息外，该程序还生成了一个非零的退出值。在Unix命令行上，退出值为`0`表示成功。我把这看作是“零错误”。在`bash` shell中，我可以检查`$?`变量来查看最近进程的退出状态。例如，命令`echo Hello`应该以值`0`退出，确实如此：

```py
$ echo Hello
Hello
$ echo $?
0
```

再次尝试之前失败的命令，然后检查 `$?`：

```py
$ ./fib.py foo
usage: fib.py [-h] generations litter
fib.py: error: argument generations: invalid int value: 'foo'
$ echo $?
2
```

退出状态为 `2` 并不像数值非零那样重要。这是一个表现良好的程序，因为它拒绝了无效的参数，打印了有用的错误消息，并以非零状态退出。如果该程序是数据处理步骤管道的一部分（比如*Makefile*，见附录 [A](app01.html#app1_makefiles)），非零的退出值会导致整个流程停止，这是一件好事。接受无效值并悄悄失败或根本不失败的程序可能会导致无法重现的结果。程序正确验证参数并在无法继续时明确失败非常重要。

程序对接受的数字类型非常严格。值必须是整数。它还会排斥任何浮点数值：

```py
$ ./fib.py 5 3.2
usage: fib.py [-h] generations litter
fib.py: error: argument litter: invalid int value: '3.2'
```

程序接收的所有命令行参数在技术上都作为字符串接收。即使命令行上的 `5` 看起来像*数字* 5，实际上是*字符*`5`。在这种情况下，我依赖 `argparse` 尝试将值从字符串转换为整数。当这种转换失败时，`argparse` 生成这些有用的错误消息。

此外，程序还会拒绝不在允许范围内的 `generations` 和 `litter` 参数的值。请注意，错误消息包括参数的名称和违规值，以提供足够的反馈，以便用户修复它：

```py
$ ./fib.py -3 2
usage: fib.py [-h] generations litter
fib.py: error: generations "-3" must be between 1 and 40 ![1](assets/1.png)
$ ./fib.py 5 10
usage: fib.py [-h] generations litter
fib.py: error: litter "10" must be between 1 and 5 ![2](assets/2.png)
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO1-1)

`-3` 的 `generations` 参数不在指定的数值范围内。

[![2](assets/2.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO1-2)

`litter` 参数为 `10` 太高了。

查看解决方案的第一部分以了解如何使其工作：

```py
import argparse
from typing import NamedTuple

class Args(NamedTuple):
    """ Command-line arguments """
    generations: int ![1](assets/1.png)
    litter: int ![2](assets/2.png)

def get_args() -> Args:
    """ Get command-line arguments """

    parser = argparse.ArgumentParser(
        description='Calculate Fibonacci',
        formatter_class=argparse.ArgumentDefaultsHelpFormatter)

    parser.add_argument('gen', ![3](assets/3.png)
                        metavar='generations',
                        type=int, ![4](assets/4.png)
                        help='Number of generations')

    parser.add_argument('litter', ![5](assets/5.png)
                        metavar='litter',
                        type=int,
                        help='Size of litter per generation')

    args = parser.parse_args() ![6](assets/6.png)

    if not 1 <= args.gen <= 40: ![7](assets/7.png)
        parser.error(f'generations "{args.gen}" must be between 1 and 40') ![8](assets/8.png)

    if not 1 <= args.litter <= 5: ![9](assets/9.png)
        parser.error(f'litter "{args.litter}" must be between 1 and 5') ![10](assets/10.png)

    return Args(generations=args.gen, litter=args.litter) ![11](assets/11.png)
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO2-1)

`generations` 字段必须是 `int` 类型。

[![2](assets/2.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO2-2)

`litter` 字段也必须是 `int` 类型。

[![3](assets/3.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO2-3)

`gen` 位置参数首先定义，因此将接收第一个位置值。

[![4](assets/4.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO2-4)

`type=int` 表示所需数值的类别。请注意，`int` 表示的是类别本身，而不是类别的名称。

[![5](assets/5.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO2-5)

`litter` 位置参数其次定义，因此将接收第二个位置值。

[![6](assets/6.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO2-6)

尝试解析参数。任何失败都将导致错误消息，并以非零值退出程序。

[![7](assets/7.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO2-7)

`args.gen`的值现在是一个实际的`int`值，因此可以对其进行数值比较。检查它是否在可接受的范围内。

[![8](assets/8.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO2-8)

使用`parser.error()`函数生成错误并退出程序。

[![9](assets/9.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO2-9)

同样检查`args.litter`参数的值。

[![10](assets/10.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO2-10)

生成一个错误，其中包含用户需要修复问题的信息。

[![11](assets/11.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO2-11)

如果程序成功运行到这一点，则参数是接受范围内的有效整数值，因此返回`Args`。

我可以在`main()`函数中检查`generations`和`litter`值是否在正确的范围内，但我更喜欢尽可能在`get_args()`函数内进行尽可能多的参数验证，以便我可以使用`parser.error()`函数生成有用的消息并以非零值退出程序。

删除`fib.py`程序，然后使用**`new.py`**或您喜欢的方法重新开始创建程序：

```py
$ new.py -fp 'Calculate Fibonacci' fib.py
Done, see new script "fib.py".
```

您可以将`get_args()`定义替换为前面的代码，然后像这样修改您的`main()`函数：

```py
def main() -> None:
    args = get_args()
    print(f'generations = {args.generations}')
    print(f'litter = {args.litter}')
```

使用无效输入运行您的程序，并验证您是否看到早期显示的错误消息类型。使用可接受的值尝试您的程序，并验证您是否看到此类输出：

```py
$ ./fib.py 1 2
generations = 1
litter = 2
```

运行**`pytest`**来查看您的程序通过和未通过的测试。您应该通过前四个测试，并未通过第五个：

```py
$ pytest -xv
========================== test session starts ==========================
...
tests/fib_test.py::test_exists PASSED                             [ 14%]
tests/fib_test.py::test_usage PASSED                              [ 28%]
tests/fib_test.py::test_bad_generations PASSED                    [ 42%]
tests/fib_test.py::test_bad_litter PASSED                         [ 57%]
tests/fib_test.py::test_1 FAILED                                  [ 71%] ![1](assets/1.png)

=============================== FAILURES ================================
________________________________ test_1 _________________________________

    def test_1():
        """runs on good input"""

        rv, out = getstatusoutput(f'{RUN} 5 3') ![2](assets/2.png)
        assert rv == 0
>       assert out == '19' ![3](assets/3.png)
E       AssertionError: assert 'generations = 5\nlitter = 3' == '19' ![4](assets/4.png)
E         - 19    ![5](assets/5.png)
E         + generations = 5 ![6](assets/6.png)
E         + litter = 3

tests/fib_test.py:60: AssertionError
======================== short test summary info ========================
FAILED tests/fib_test.py::test_1 - AssertionError: assert 'generations...
!!!!!!!!!!!!!!!!!!!!!!! stopping after 1 failures !!!!!!!!!!!!!!!!!!!!!!!
====================== 1 failed, 4 passed in 0.38s ======================
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO3-1)

第一个失败的测试。由于`-x`标志的存在，测试在此处停止。

[![2](assets/2.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO3-2)

该程序以`5`作为生成数量和`3`作为每胎大小来运行。

[![3](assets/3.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO3-3)

输出应为`19`。

[![4](assets/4.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO3-4)

这显示了两个字符串的比较结果不相等。

[![5](assets/5.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO3-5)

预期值为`19`。

[![6](assets/6.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO3-6)

这是接收到的输出。

`pytest`的输出非常努力地指出了发生了什么问题。它显示了程序的运行方式及期望的结果与实际产生的结果之间的差异。程序应该打印19，这是使用每胎数量为3时斐波那契数列的第五个数字。如果您想独自完成程序，请直接开始。您应该使用**`pytest`**验证是否通过了所有测试。另外，运行**`make test`**以使用`pylint`、`flake8`和`mypy`检查您的程序。如果您需要一些指导，我将介绍我描述的第一种方法。

## 一种命令式方法

[图 4-2](#fig_4.2)描述了斐波那契数列的增长。较小的兔子表示必须成熟为较大的繁殖对的非繁殖对。

![mpfb 0402](assets/mpfb_0402.png)

###### 图 4-2\. 使用每胎数量为1的兔子配对作为斐波那契数列增长的可视化

您可以看到，要生成前两个数之后的任何数字，我需要知道前两个数。我可以使用这个公式来描述斐波那契数列（*F*）的任意位置*n*的值：

<math alttext="upper F Subscript n Baseline equals upper F Subscript n minus 1 Baseline plus upper F Subscript n minus 2" display="block"><mrow><msub><mi>F</mi> <mi>n</mi></msub> <mo>=</mo> <msub><mi>F</mi> <mrow><mi>n</mi><mo>-</mo><mn>1</mn></mrow></msub> <mo>+</mo> <msub><mi>F</mi> <mrow><mi>n</mi><mo>-</mo><mn>2</mn></mrow></msub></mrow></math>

在Python中，哪种数据结构可以让我按顺序保留一系列数字并按其位置引用它们？列表。我将从*F*[1] = 0和*F*[2] = 1开始：

```py
>>> fib = [0, 1]
```

*F*[3]值为*F*[2] + *F*[1] = 1 + 0 = 1。在生成下一个数字时，我将始终引用序列的*最后两个*元素。使用负索引最容易指示从列表*末尾*的位置。列表中的最后一个值始终位于位置`-1`：

```py
>>> fib[-1]
1
```

倒数第二个值是`-2`：

```py
>>> fib[-2]
0
```

我需要将这个值乘以每胎数量来计算该代产生的后代数量。首先，我将考虑每胎数量为1：

```py
>>> litter = 1
>>> fib[-2] * litter
0
```

我想要将这两个数字加在一起，并将结果附加到列表中：

```py
>>> fib.append((fib[-2] * litter) + fib[-1])
>>> fib
[0, 1, 1]
```

如果我再做一次，我可以看到正确的序列正在出现：

```py
>>> fib.append((fib[-2] * litter) + fib[-1])
>>> fib
[0, 1, 1, 2]
```

我需要重复这个动作`generations`次。（从技术上讲，实际上是`generations`-1次，因为Python使用基于0的索引。）我可以使用Python的`range()`函数生成从`0`到结束值但不包括结束值的数字列表。我调用这个函数仅仅是为了迭代特定次数，所以不需要`range()`函数生成的值。习惯上使用下划线（`_`）变量表示忽略某个值的意图：

```py
>>> fib = [0, 1]
>>> litter = 1
>>> generations = 5
>>> for _ in range(generations - 1):
...     fib.append((fib[-2] * litter) + fib[-1])
...
>>> fib
[0, 1, 1, 2, 3, 5]
```

这应该足够让您创建一个通过测试的解决方案。在下一节中，我将介绍另外两个解决方案，突出Python的一些非常有趣的部分。

# 解决方案

所有以下解决方案都共享相同的`get_args()`如前所示。

## 解决方案1：使用列表作为堆栈的命令式解决方案

这是我写的命令式解决方案。我使用列表作为*栈*来跟踪过去的值。我不需要所有的值，只需要最后两个，但是保持列表不断增长并引用最后两个值是相当容易的：

```py
def main() -> None:
    args = get_args()

    fib = [0, 1] ![1](assets/1.png)
    for _ in range(args.generations - 1): ![2](assets/2.png)
        fib.append((fib[-2] * args.litter) + fib[-1]) ![3](assets/3.png)

    print(fib[-1]) ![4](assets/4.png)
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO4-1)

从`0`和`1`开始。

[![2](assets/2.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO4-2)

使用`range()`函数创建正确数量的循环。

[![3](assets/3.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO4-3)

将下一个值附加到序列中。

[![4](assets/4.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO4-4)

打印序列的最后一个数字。

在`for`循环中使用`_`变量名表明我不打算使用该变量。下划线是一个有效的Python标识符，并且也是一种约定，用于表示*丢弃*值。例如，代码检查工具可能会看到我给变量赋了一个值但从未使用它，这通常会被视为可能的错误。下划线变量表明我不打算使用该值。在这种情况下，我仅仅使用`range()`函数是为了产生所需的循环次数。

这被认为是一个*命令式*解决方案，因为代码直接编码了算法的每一个指令。当你阅读递归解决方案时，你会看到算法可以以更声明性的方式编写，这也带来了我必须处理的意外后果。

稍微变化的方式是将这段代码放在一个我称之为`fib()`的函数中。请注意，在Python中可以在另一个函数内部声明函数，例如我将在`main()`内部创建`fib()`。我这样做是为了可以引用`args.litter`参数，因为函数正在捕获垃圾的运行时值，从而创建了一个*闭包*：

```py
def main() -> None:
    args = get_args()

    def fib(n: int) -> int: ![1](assets/1.png)
        nums = [0, 1] ![2](assets/2.png)
        for _ in range(n - 1): ![3](assets/3.png)
            nums.append((nums[-2] * args.litter) + nums[-1]) ![4](assets/4.png)
        return nums[-1] ![5](assets/5.png)

    print(fib(args.generations)) ![6](assets/6.png)
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO5-1)

创建一个名为`fib()`的函数，接受一个整数参数`n`并返回一个整数。

[![2](assets/2.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO5-2)

这与之前的代码相同。请注意，此列表称为`nums`，以避免与函数名冲突。

[![3](assets/3.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO5-3)

使用`range()`函数迭代生成。

[![4](assets/4.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO5-4)

函数引用`args.litter`参数，因此创建了一个闭包。

[![5](assets/5.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO5-5)

使用`return`将最终值发送回调用者。

[![6](assets/6.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO5-6)

使用`args.generations`参数调用`fib()`函数。

在前面的示例中，`fib()`函数的作用域仅限于`main()`函数。*作用域*指的是程序中特定函数名称或变量可见或合法的部分。

我不必使用闭包。以下是我如何使用标准函数表达相同的想法：

```py
def main() -> None:
    args = get_args()

    print(fib(args.generations, args.litter)) ![1](assets/1.png)

def fib(n: int, litter: int) -> int: ![2](assets/2.png)
    nums = [0, 1]
    for _ in range(n - 1):
        nums.append((nums[-2] * litter) + nums[-1])

    return nums[-1]
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO6-1)

函数`fib()`必须使用两个参数调用。

[![2](assets/2.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO6-2)

函数需要世代数和窝大小。函数体本质上相同。

在上述代码中，您可以看到我必须向`fib()`传递两个参数，而闭包只需要一个参数，因为捕获了`litter`。绑定值并减少参数数量是创建闭包的有效原因之一。另一个编写闭包的原因是限制函数的作用域。`fib()`函数的闭包定义仅在`main()`函数内有效，而前一个版本在整个程序中都可见。将一个函数隐藏在另一个函数中会使测试变得更加困难。在本例中，`fib()`函数几乎是整个程序，因此测试已在*tests/fib_test.py*中编写。

## 解决方案2：创建生成器函数

在之前的解决方案中，我生成了请求值之前的斐波那契数列，然后停止；但是，这个序列是无限的。我能否创建一个可以生成*所有*序列数的函数？从技术上讲，可以，但它永远不会完成，毕竟它是无限的。

Python有一种方法可以挂起生成可能无限序列的函数。我可以使用`yield`从函数中返回一个值，稍后再从函数中暂时离开，以在请求下一个值时恢复到相同状态。这种函数称为*生成器*，以下是我如何使用它生成序列：

```py
def fib(k: int) -> Generator[int, None, None]: ![1](assets/1.png)
    x, y = 0, 1 ![2](assets/2.png)
    yield x ![3](assets/3.png)

    while True: ![4](assets/4.png)
        yield y ![5](assets/5.png)
        x, y = y * k, x + y ![6](assets/6.png)
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO7-1)

类型签名表明该函数接受参数`k`（窝大小），必须是`int`类型。它返回一种`Generator`类型的特殊函数，该函数生成`int`值且没有发送或返回类型。

[![2](assets/2.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO7-2)

我只需要跟踪最后两代，我将它们初始化为`0`和`1`。

[![3](assets/3.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO7-3)

生成`0`。

[![4](assets/4.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO7-4)

创建一个无限循环。

[![5](assets/5.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO7-5)

生成最后一代。

[![6](assets/6.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO7-6)

将`x`（前两代）设置为当前代乘以幼崽数量。将`y`（前一代）设置为两个当前代的和。

生成器像迭代器一样工作，根据代码请求生成值，直到耗尽。由于这个生成器只生成yield值，因此发送和返回类型为`None`。除此之外，这段代码完全与程序的第一个版本相同，只是内部使用了一个花哨的生成器函数。查看[图4-3](#fig_4.3)以考虑该函数如何适用于两种不同的幼崽数量。

![mpfb 0403](assets/mpfb_0403.png)

###### 图4-3\. 展示了`fib()`生成器在时间上如何变化（`n`=5），适用于两个不同的幼崽数量（`k`=1 和 `k`=3）。

`Generator`的类型签名看起来有点复杂，因为它定义了yield、send和return的类型。我不需要在这里进一步深入，但建议您阅读关于[typing模块的文档](https://oreil.ly/Oir3d)。

这里是如何使用的：

```py
def main() -> None:
    args = get_args()
    gen = fib(args.litter) ![1](assets/1.png)
    seq = [next(gen) for _ in range(args.generations + 1)] ![2](assets/2.png)
    print(seq[-1]) ![3](assets/3.png)
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO8-1)

`fib()`函数接受幼崽数量作为参数并返回一个生成器。

[![2](assets/2.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO8-2)

使用`next()`函数从生成器中获取下一个值。使用列表推导来正确次数生成序列，直至请求的值。

[![3](assets/3.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO8-3)

打印序列中的最后一个数字。

`range()`函数的功能不同，因为第一个版本已经包含了`0`和`1`。在这里，我必须额外调用两次生成器才能产生这些值。

尽管我更喜欢列表推导，但我不需要整个列表。我只关心最后的值，所以可以这样写：

```py
def main() -> None:
    args = get_args()
    gen = fib(args.litter)
    answer = 0 ![1](assets/1.png)
    for _ in range(args.generations + 1): ![2](assets/2.png)
        answer = next(gen) ![3](assets/3.png)
    print(answer) ![4](assets/4.png)
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO9-1)

将答案初始化为`0`。

[![2](assets/2.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO9-2)

创建正确数量的循环。

[![3](assets/3.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO9-3)

获取当前代的值。

[![4](assets/4.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO9-4)

打印答案。

恰好，经常多次调用函数生成列表，所以有一个函数来为我们做这个。`itertools.islice()` 函数将“生成一个迭代器，从可迭代对象中返回选定的元素。”这是我如何使用它的方法：

```py
def main() -> None:
    args = get_args()
    seq = list(islice(fib(args.litter), args.generations + 1)) ![1](assets/1.png)
    print(seq[-1]) ![2](assets/2.png)
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO10-1)

`islice()` 的第一个参数是将被调用的函数，第二个参数是调用它的次数。该函数是惰性的，因此我使用 `list()` 强制生成值。

[![2](assets/2.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO10-2)

打印最后一个值。

由于我只使用 `seq` 变量一次，我可以避免那个赋值。如果基准测试证明以下是性能最佳的版本，我可能愿意写成一行：

```py
def main() -> None:
    args = get_args()
    print(list(islice(fib(args.litter), args.generations + 1))[-1])
```

聪明的代码很有趣，但可能变得难以阅读。^([1](ch04.html#idm45963635588216)) 你已经被警告过了。

生成器很酷，但比生成列表更复杂。它们是生成非常大或潜在无限序列值的适当方式，因为它们是惰性的，在你的代码需要时才计算下一个值。

## 解决方案 3：使用递归和记忆化

虽然有许多更有趣的方法来编写生成无限数列的算法，但我只展示使用*递归*的另一种方法：

```py
def main() -> None:
    args = get_args()

    def fib(n: int) -> int: ![1](assets/1.png)
        return 1 if n in (1, 2) \ ![2](assets/2.png)
            else fib(n - 2) * args.litter + fib(n - 1) ![3](assets/3.png)

    print(fib(args.generations)) ![4](assets/4.png)
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO11-1)

定义一个名为 `fib()` 的函数，它接受所需代数作为 `int` 并返回 `int`。

[![2](assets/2.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO11-2)

如果代数为 `1` 或 `2`，返回 `1`。这是非常重要的基础情况，不会进行递归调用。

[![3](assets/3.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO11-3)

对于所有其他情况，调用 `fib()` 函数两次，一次用于前两代，另一次用于前一代。与以往一样考虑每胎的数量。

[![4](assets/4.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO11-4)

打印给定代数的 `fib()` 函数的结果。

这里又是一个例子，我在 `main()` 函数内部定义了一个作为闭包的 `fib()` 函数，以便在 `fib()` 函数内使用 `args.litter` 值。这样可以将 `args.litter` 绑定到函数中。如果我在 `main()` 函数外定义了该函数，我将不得不在递归调用时传递 `args.litter` 参数。

这是一个非常优雅的解决方案，在几乎每个计算机科学的入门课程中都会教授。研究起来很有趣，但事实证明它非常慢，因为我最终需要调用该函数很多次。也就是说，`fib(5)`需要调用`fib(4)`和`fib(3)`来添加这些值。而`fib(4)`需要调用`fib(3)`和`fib(2)`，依此类推。图 4-4 显示，`fib(5)`导致14次函数调用以产生5个不同的值。例如，`fib(2)`被计算了三次，但我们只需要计算一次。

![mpfb 0404](assets/mpfb_0404.png)

###### 图 4-4\. 调用`fib(5)`的调用堆栈导致多次递归调用函数，随着输入值的增加，递归调用次数增加大约呈指数级增长

为了说明问题，我将采样这个程序在最大`n`为40时完成所需的时间。同样，我将使用`bash`中的`for`循环来展示如何在命令行中常见地对这样一个程序进行基准测试：

```py
$ for n in 10 20 30 40;
> do echo "==> $n <==" && time ./solution3_recursion.py $n 1
> done
==> 10 <==
55

real	0m0.045s
user	0m0.032s
sys	0m0.011s
==> 20 <==
6765

real	0m0.041s
user	0m0.031s
sys	0m0.009s
==> 30 <==
832040

real	0m0.292s
user	0m0.281s
sys	0m0.009s
==> 40 <==
102334155

real	0m31.629s
user	0m31.505s
sys	0m0.043s
```

从`n`=`30`的0.29秒到`n`=`40`的31秒的跳跃是巨大的。想象一下去到50及以上。我需要找到一种方法来加快这个过程或放弃递归的所有希望。解决方法是缓存先前计算的结果。这被称为*记忆化*，有许多实现方法。以下是一种方法。注意，您需要导入`typing.Callable`：

```py
def memoize(f: Callable) -> Callable: ![1](assets/1.png)
    """ Memoize a function """

    cache = {} ![2](assets/2.png)

    def memo(x): ![3](assets/3.png)
        if x not in cache: ![4](assets/4.png)
            cache[x] = f(x) ![5](assets/5.png)
        return cache[x] ![6](assets/6.png)

    return memo ![7](assets/7.png)
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO12-1)

定义一个接受函数（即*可调用对象*）并返回函数的函数。

[![2](assets/2.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO12-2)

使用字典来存储缓存值。

[![3](assets/3.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO12-3)

将`memo()`定义为对缓存的闭包。在调用时，该函数将接受某些参数`x`。

[![4](assets/4.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO12-4)

检查缓存中是否存在参数值。

[![5](assets/5.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO12-5)

如果没有，使用参数调用函数，并将该参数值的缓存设置为结果。

[![6](assets/6.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO12-6)

返回参数的缓存值。

[![7](assets/7.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO12-7)

返回新函数。

注意，`memoize()`函数返回一个新函数。在Python中，函数被认为是*一级对象*，意味着它们可以像其他类型的变量一样使用——你可以将它们作为参数传递并覆盖它们的定义。`memoize()`函数是*高阶函数*（HOF）的一个例子，因为它接受其他函数作为参数。我将在本书中使用其他HOF，如`filter()`和`map()`。

要使用`memoize()`函数，我将定义`fib()`，然后用记忆化版本*重新定义*它。如果你运行这个程序，无论`n`多大，你都会看到几乎瞬间的结果：

```py
def main() -> None:
    args = get_args()

    def fib(n: int) -> int:
        return 1 if n in (1, 2) else fib(n - 2) * args.litter + fib(n - 1)

    fib = memoize(fib) ![1](assets/1.png)

    print(fib(args.generations))
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO13-1)

用记忆化函数覆盖现有的`fib()`定义。

实现此目标的首选方法是使用*装饰器*，即修改其他函数的函数：

```py
def main() -> None:
    args = get_args()

    @memoize ![1](assets/1.png)
    def fib(n: int) -> int:
        return 1 if n in (1, 2) else fib(n - 2) * args.litter + fib(n - 1)

    print(fib(args.generations))
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO14-1)

使用`memoize()`函数装饰`fib()`函数。

尽管编写记忆化函数很有趣，但事实证明这是一个常见的需求，其他人已经为我们解决了。我可以移除`memoize()`函数，而是导入`functools.lru_cache`（最近最少使用缓存）函数：

```py
from functools import lru_cache
```

使用`lru_cache()`函数装饰`fib()`函数以实现记忆化，尽量减少干扰：

```py
def main() -> None:
    args = get_args()

    @lru_cache() ![1](assets/1.png)
    def fib(n: int) -> int:
        return 1 if n in (1, 2) else fib(n - 2) * args.litter + fib(n - 1)

    print(fib(args.generations))
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO15-1)

通过`lru_cache()`函数装饰`fib()`函数进行记忆化。注意，Python 3.6需要括号，但3.8及更高版本不需要。

# 测试解决方案的性能

哪个是最快的解决方案？我已经向你展示了如何在`bash`中使用`for`循环和`time`命令来比较命令的运行时间：

```py
$ for py in ./solution1_list.py ./solution2_generator_islice.py \
./solution3_recursion_lru_cache.py; do echo $py && time $py 40 5; done
./solution1_list.py
148277527396903091

real	0m0.070s
user	0m0.043s
sys	0m0.016s
./solution2_generator_islice.py
148277527396903091

real	0m0.049s
user	0m0.033s
sys	0m0.013s
./solution3_recursion_lru_cache.py
148277527396903091

real	0m0.041s
user	0m0.030s
sys	0m0.010s
```

看起来使用LRU缓存的递归解决方案是最快的，但我只有很少的数据——每个程序只运行一次。此外，我需要凭眼观察数据并确定哪个最快。

有更好的方法。我安装了一个名为[`hyperfine`](https://oreil.ly/shqOS)的工具来多次运行每个命令并比较结果：

```py
$ hyperfine -L prg ./solution1_list.py,./solution2_generator_islice.py,\
./solution3_recursion_lru_cache.py '{prg} 40 5' --prepare 'rm -rf __pycache__'
Benchmark #1: ./solution1_list.py 40 5
  Time (mean ± σ):      38.1 ms ±   1.1 ms    [User: 28.3 ms, System: 8.2 ms]
  Range (min … max):    36.6 ms …  42.8 ms    60 runs

Benchmark #2: ./solution2_generator_islice.py 40 5
  Time (mean ± σ):      38.0 ms ±   0.6 ms    [User: 28.2 ms, System: 8.1 ms]
  Range (min … max):    36.7 ms …  39.2 ms    66 runs

Benchmark #3: ./solution3_recursion_lru_cache.py 40 5
  Time (mean ± σ):      37.9 ms ±   0.6 ms    [User: 28.1 ms, System: 8.1 ms]
  Range (min … max):    36.6 ms …  39.4 ms    65 runs

Summary
  './solution3_recursion_lru_cache.py 40 5' ran
    1.00 ± 0.02 times faster than './solution2_generator_islice.py 40 5'
    1.01 ± 0.03 times faster than './solution1_list.py 40 5'
```

看起来`hyperfine`运行了每个命令60-66次，取平均结果，并发现`solution3_recursion_lru_cache.py`程序可能稍快。另一个你可能找到有用的基准工具是[`bench`](https://oreil.ly/FKnmd)，但你可以在互联网上搜索其他更适合你口味的基准工具。无论你使用什么工具，基准测试以及测试对挑战代码假设至关重要。

我使用了 `--prepare` 选项告诉 `hyperfine` 在运行命令之前删除 *pycache* 目录。这是 Python 创建的目录，用于缓存程序的 *bytecode*。如果程序的源代码自上次运行以来没有改变，那么 Python 可以跳过编译，直接使用 *pycache* 目录中存在的 bytecode 版本。我需要删除它，因为 `hyperfine` 在运行命令时检测到统计异常值，可能是缓存效应导致的。

# 测试好的、坏的和丑陋的

对于每一个挑战，希望你花些时间阅读测试内容。学习如何设计和编写测试与我展示的任何其他内容同样重要。如我之前提到的，我的第一个测试检查预期的程序是否存在，并且在需要时产生使用说明。接着，我通常会输入无效的数据来确保程序失败。我想强调的是针对不良 `n` 和 `k` 参数的测试。它们本质上是相同的，所以我只展示第一个作为示例，演示如何随机选择一个无效的整数值，例如可能是负数或者太大：

```py
def test_bad_n():
    """ Dies when n is bad """

    n = random.choice(list(range(-10, 0)) + list(range(41, 50))) ![1](assets/1.png)
    k = random.randint(1, 5) ![2](assets/2.png)
    rv, out = getstatusoutput(f'{RUN} {n} {k}') ![3](assets/3.png)
    assert rv != 0 ![4](assets/4.png)
    assert out.lower().startswith('usage:') ![5](assets/5.png)
    assert re.search(f'n "{n}" must be between 1 and 40', out) ![6](assets/6.png)
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO16-1)

将两个无效数字范围列表连接起来，并随机选择一个值。

[![2](assets/2.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO16-2)

选择一个从 `1` 到 `5`（包括边界）的随机整数。

[![3](assets/3.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO16-3)

运行带有参数的程序，并捕获输出。

[![4](assets/4.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO16-4)

确保程序报告了失败（非零退出值）。

[![5](assets/5.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO16-5)

检查输出是否以使用说明开头。

[![6](assets/6.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO16-6)

寻找描述 `n` 参数问题的错误消息。

我经常喜欢在测试时使用随机选择的无效值。这在某种程度上是为了给学生编写测试，以防止他们在单个坏输入上失败，但我也发现这有助于我避免意外地编写特定输入值的代码。我还未介绍 `random` 模块，但它可以让你进行伪随机选择。首先，你需要导入该模块：

```py
>>> import random
```

例如，你可以使用 `random.randint()` 来从给定范围内选择一个整数：

```py
>>> random.randint(1, 5)
2
>>> random.randint(1, 5)
5
```

或者使用 `random.choice()` 函数从某个序列中随机选择一个值。在这里，我想构建一个不连续的负数范围，与一个正数范围分开：

```py
>>> random.choice(list(range(-10, 0)) + list(range(41, 50)))
46
>>> random.choice(list(range(-10, 0)) + list(range(41, 50)))
-1
```

接下来的测试提供了程序的有效输入。例如：

```py
def test_2():
    """ Runs on good input """

    rv, out = getstatusoutput(f'{RUN} 30 4') ![1](assets/1.png)
    assert rv == 0 ![2](assets/2.png)
    assert out == '436390025825' ![3](assets/3.png)
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO17-1)

这些是我在尝试解决Rosalind挑战时收到的值。

[![2](assets/2.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO17-2)

程序不应在此输入上失败。

[![3](assets/3.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO17-3)

这是Rosalind的正确答案。

测试，就像文档一样，是写给未来自己的一封情书。尽管测试看起来可能很乏味，但当您尝试添加功能并意外破坏以前工作的东西时，您会感激失败的测试。认真地编写和运行测试可以防止您部署损坏的程序。

# 在所有解决方案上运行测试套件

您已经看到，在每章中，我会写多个解决方案来探索解决问题的各种方法。我完全依赖我的测试来确保我的程序是正确的。您可能会好奇看看我如何自动化测试每个解决方案的过程。查看*Makefile*并找到`all`目标：

```py
$ cat Makefile
.PHONY: test

test:
	python3 -m pytest -xv --flake8 --pylint --mypy fib.py tests/fib_test.py

all:
    ../bin/all_test.py fib.py
```

程序`all_test.py`会在运行测试套件之前用每个解决方案覆盖程序`fib.py`。这可能会覆盖您的解决方案。在运行`make all`之前，请确保您将您的版本提交到Git或至少复制一份，否则可能会丢失您的工作。

下面是由`all`目标运行的`all_test.py`程序。我将其分成两部分，从第一部分到`get_args()`。大部分内容现在应该已经很熟悉了：

```py
#!/usr/bin/env python3
""" Run the test suite on all solution*.py """

import argparse
import os
import re
import shutil
import sys
from subprocess import getstatusoutput
from functools import partial
from typing import NamedTuple

class Args(NamedTuple):
    """ Command-line arguments """
    program: str ![1](assets/1.png)
    quiet: bool ![2](assets/2.png)

# --------------------------------------------------
def get_args() -> Args:
    """ Get command-line arguments """

    parser = argparse.ArgumentParser(
        description='Run the test suite on all solution*.py',
        formatter_class=argparse.ArgumentDefaultsHelpFormatter)

    parser.add_argument('program', metavar='prg', help='Program to test') ![3](assets/3.png)

    parser.add_argument('-q', '--quiet', action='store_true', help='Be quiet') ![4](assets/4.png)

    args = parser.parse_args()

    return Args(args.program, args.quiet)
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO18-1)

要测试的程序名称，在本例中为`fib.py`。

[![2](assets/2.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO18-2)

一个布尔值`True`或`False`来创建更多或更少的输出。

[![3](assets/3.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO18-3)

默认类型是`str`。

[![4](assets/4.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO18-4)

`action='store_true'`使其成为布尔标志。如果标志存在，则值为`True`；否则为`False`。

`main()`函数是进行测试的地方：

```py
def main() -> None:
    args = get_args()
    cwd = os.getcwd() ![1](assets/1.png)
    solutions = list( ![2](assets/2.png)
        filter(partial(re.match, r'solution.*\.py'), os.listdir(cwd))) ![3](assets/3.png)

    for solution in sorted(solutions): ![4](assets/4.png)
        print(f'==> {solution} <==')
        shutil.copyfile(solution, os.path.join(cwd, args.program)) ![5](assets/5.png)
        subprocess.run(['chmod', '+x', args.program], check=True) ![6](assets/6.png)
        rv, out = getstatusoutput('make test') ![7](assets/7.png)
        if rv != 0: ![8](assets/8.png)
            sys.exit(out) ![9](assets/9.png)

        if not args.quiet: ![10](assets/10.png)
            print(out)

    print('Done.') ![11](assets/11.png)
```

[![1](assets/1.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO19-1)

获取当前工作目录，如果您在运行命令时位于该目录中，则为*04_fib*目录。

[![2](assets/2.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO19-2)

查找当前目录中的所有`solution*.py`文件。

[![3](assets/3.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO19-3)

`filter()`和`partial()`都是高阶函数；接下来我将解释它们。

[![4](assets/4.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO19-4)

文件名将以随机顺序排列，因此需要遍历排序后的文件。

[![5](assets/5.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO19-5)

将`solution*.py`文件复制到测试文件名。

[![6](assets/6.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO19-6)

让程序可执行。

[![7](assets/7.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO19-7)

运行`make test`命令，并捕获返回值和输出。

[![8](assets/8.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO19-8)

检查返回值是否不为`0`。

[![9](assets/9.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO19-9)

退出此程序时，打印测试输出并返回非零值。

[![10](assets/10.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO19-10)

除非程序需要静默运行，否则打印测试输出。

[![11](assets/11.png)](#co_creating_the_fibonacci_sequence__writing__testing__and_benchmarking_algorithms_CO19-11)

让用户知道程序正常结束。

在前面的代码中，我使用`sys.exit()`来立即停止程序，打印错误消息，并返回一个非零的退出值。如果查阅文档，您会发现可以用没有参数、一个整数值或像字符串这样的对象调用`sys.exit()`，这正是我所使用的：

```py
exit(status=None, /)
    Exit the interpreter by raising SystemExit(status).

    If the status is omitted or None, it defaults to zero (i.e., success).
    If the status is an integer, it will be used as the system exit status.
    If it is another kind of object, it will be printed and the system
    exit status will be one (i.e., failure).
```

前面的程序还使用了`filter()`或`partial()`函数，这两者都是高阶函数。我将解释我如何以及为什么使用它们。首先，`os.listdir()`函数将返回目录的全部内容，包括文件和子目录：

```py
>>> import os
>>> files = os.listdir()
```

这里有很多内容，所以我将从`pprint`模块导入`pprint()`函数以*漂亮打印*它：

```py
>>> from pprint import pprint
>>> pprint(files)
['solution3_recursion_memoize_decorator.py',
 'solution2_generator_for_loop.py',
 '.pytest_cache',
 'Makefile',
 'solution2_generator_islice.py',
 'tests',
 '__pycache__',
 'fib.py',
 'README.md',
 'solution3_recursion_memoize.py',
 'bench.html',
 'solution2_generator.py',
 '.mypy_cache',
 '.gitignore',
 'solution1_list.py',
 'solution3_recursion_lru_cache.py',
 'solution3_recursion.py']
```

我想要过滤出以*solution*开头且以*.py*结尾的文件名。在命令行中，我可以使用`solution*.py`这样的*文件通配符*模式，其中`*`表示*任意数量的任何字符*，`.`是一个字面上的点。这个模式的正则表达式版本稍微复杂一些，是`solution.*\.py`，其中`.`（点）是正则表达式元字符，代表*任何字符*，`*`（星号）表示*零个或多个*（见[图 4-5](#fig_4.5)）。为了表示字面上的点，我需要用反斜杠进行转义（`\.`）。注意，最好使用r字符串（*原始*字符串）来包围这个模式。

![mpfb 0405](assets/mpfb_0405.png)

###### 图 4-5\. 用于找到与文件通配符`solution*.py`匹配的文件的正则表达式

当匹配成功时，将返回一个`re.Match`对象：

```py
>>> import re
>>> re.match(r'solution.*\.py', 'solution1.py')
<re.Match object; span=(0, 12), match='solution1.py'>
```

当匹配失败时，将返回`None`值。我必须在这里使用`type()`，因为在REPL中不显示`None`值：

```py
>>> type(re.match(r'solution.*\.py', 'fib.py'))
<class 'NoneType'>
```

我想将此匹配应用于`os.listdir()`返回的所有文件。我可以使用`filter()`和`lambda`关键字来创建一个*匿名*函数。将`files`中的每个文件名作为`name`参数传递给匹配。`filter()`函数将只返回给定函数中返回真值的元素，因此那些无法匹配时返回`None`的文件名将被排除：

```py
>>> pprint(list(filter(lambda name: re.match(r'solution.*\.py', name), files)))
['solution3_recursion_memoize_decorator.py',
 'solution2_generator_for_loop.py',
 'solution2_generator_islice.py',
 'solution3_recursion_memoize.py',
 'solution2_generator.py',
 'solution1_list.py',
 'solution3_recursion_lru_cache.py',
 'solution3_recursion.py']
```

你可以看到`re.match()`函数接受两个参数——模式和要匹配的字符串。`partial()`函数允许我*部分应用*函数，并返回一个新函数。例如，`operator.add()`函数期望两个值并返回它们的和：

```py
>>> import operator
>>> operator.add(1, 2)
3
```

我可以创建一个函数，用于将任何值加`1`，就像这样：

```py
>>> from functools import partial
>>> succ = partial(op.add, 1)
```

`succ()`函数需要一个参数，并返回其后继：

```py
>>> succ(3)
4
>>> succ(succ(3))
5
```

同样，我可以创建一个函数`f()`，部分应用`re.match()`函数的第一个参数，即正则表达式模式：

```py
>>> f = partial(re.match, r'solution.*\.py')
```

`f()`函数正在等待一个字符串来应用匹配：

```py
>>> type(f('solution1.py'))
<class 're.Match'>
>>> type(f('fib.py'))
<class 'NoneType'>
```

如果你不带参数调用它，将会收到一个异常：

```py
>>> f()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: match() missing 1 required positional argument: 'string'
```

我可以用部分应用函数替换`lambda`作为`filter()`的第一个参数：

```py
>>> pprint(list(filter(f, files)))
['solution3_recursion_memoize_decorator.py',
 'solution2_generator_for_loop.py',
 'solution2_generator_islice.py',
 'solution3_recursion_memoize.py',
 'solution2_generator.py',
 'solution1_list.py',
 'solution3_recursion_lru_cache.py',
 'solution3_recursion.py']
```

我的编程风格在很大程度上倾向于纯函数式编程思想。我发现这种风格就像玩乐高积木一样——小而明确定义的测试函数可以组合成运行良好的更大程序。

# 深入了解

编程有许多不同的风格，如过程化、函数式、面向对象等等。即使在像Python这样的面向对象语言中，我也可以使用非常不同的编程方法。第一种解决方案可以被认为是*动态规划*方法，因为你首先通过解决较小的问题来解决更大的问题。如果你觉得递归函数有趣，汉诺塔问题是另一个经典练习。像Haskell这样的纯函数式语言大多避免像`for`循环这样的构造，而是严重依赖递归和高阶函数。口语和编程语言塑造了我们思考问题的方式，我鼓励你尝试使用你了解的其他语言解决这个问题，看看你可能会写出不同的解决方案。

# 复习

本章的要点：

+   在`get_args()`函数内部，您可以对参数进行手动验证，并使用`parser.error()`函数手动生成`argparse`错误。

+   您可以通过推送和弹出元素来使用列表作为堆栈。

+   在函数中使用`yield`将其转换为生成器。当函数生成一个值时，该值被返回，并保留函数的状态，直到下一个值被请求。生成器可用于创建潜在的无限值流。

+   递归函数调用自身，递归可能导致严重的性能问题。一种解决方法是使用记忆化来缓存值并避免重新计算。

+   高阶函数是接受其他函数作为参数的函数。

+   Python 的函数装饰器将高阶函数应用于其他函数。

+   基准测试是确定最佳性能算法的重要技术。`hyperfine` 和 `bench` 工具允许您比较多次迭代的命令运行时间。

+   `random` 模块提供了许多用于伪随机选择值的函数。

^([1](ch04.html#idm45963635588216-marker)) 正如传奇般的大卫·圣·哈宾斯和奈杰尔·塔夫内尔所观察到的，“愚蠢和聪明之间只有一线之隔。”
