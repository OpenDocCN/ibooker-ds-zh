# 第六章：NumPy 数组上的计算：通用函数

到目前为止，我们已经讨论了 NumPy 的一些基本要点。在接下来的几章中，我们将深入探讨 NumPy 在 Python 数据科学世界中如此重要的原因：因为它提供了一个简单灵活的接口来优化数据数组的计算。

NumPy 数组上的计算可能非常快，也可能非常慢。使其快速的关键在于使用向量化操作，通常通过 NumPy 的*通用函数*（ufuncs）实现。本章阐述了使用 NumPy ufuncs 的必要性，它可以使对数组元素的重复计算更加高效。然后介绍了 NumPy 包中许多常见和有用的算术 ufuncs。

# 循环的缓慢性

Python 的默认实现（称为 CPython）有时会非常慢地执行某些操作。这在一定程度上是由于语言的动态解释性质造成的；类型灵活，因此无法像 C 和 Fortran 语言那样将操作序列编译成高效的机器码。近年来，有各种尝试来解决这一弱点：著名的例子有 [PyPy 项目](http://pypy.org)，一个即时编译的 Python 实现；[Cython 项目](http://cython.org)，它将 Python 代码转换为可编译的 C 代码；以及 [Numba 项目](http://numba.pydata.org)，它将 Python 代码片段转换为快速的 LLVM 字节码。每种方法都有其优势和劣势，但可以肯定的是，这三种方法都还没有超越标准 CPython 引擎的普及度和影响力。

Python 的相对缓慢通常在需要重复执行许多小操作的情况下显现出来；例如，循环遍历数组以对每个元素进行操作。例如，假设我们有一个值数组，并且想要计算每个值的倒数。一个直接的方法可能如下所示：

```py
In [1]: import numpy as np
        rng = np.random.default_rng(seed=1701)

        def compute_reciprocals(values):
            output = np.empty(len(values))
            for i in range(len(values)):
                output[i] = 1.0 / values[i]
            return output

        values = rng.integers(1, 10, size=5)
        compute_reciprocals(values)
Out[1]: array([0.11111111, 0.25      , 1.        , 0.33333333, 0.125     ])
```

这种实现对于来自 C 或 Java 背景的人来说可能感觉相当自然。但是如果我们测量这段代码在大输入下的执行时间，我们会发现这个操作非常慢——也许出乎意料地慢！我们将使用 IPython 的 `%timeit` 魔术命令（在“分析和计时代码”中讨论）进行基准测试：

```py
In [2]: big_array = rng.integers(1, 100, size=1000000)
        %timeit compute_reciprocals(big_array)
Out[2]: 2.61 s ± 192 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```

计算这些百万次操作并存储结果需要几秒钟！即使是手机的处理速度也以亿次数的每秒运算来衡量，这看起来几乎是荒谬地慢。事实证明，这里的瓶颈不是操作本身，而是 CPython 在每个循环周期中必须进行的类型检查和函数调度。每次计算倒数时，Python 首先检查对象的类型，并动态查找正确的函数来使用该类型。如果我们在编译代码中工作，这种类型规范将在代码执行之前已知，并且结果可以更有效地计算。

# 引入 Ufuncs

对于许多类型的操作，NumPy 提供了一个便利的接口，用于这种静态类型化的、编译的例程。这被称为*向量化*操作。对于像这里的逐元素除法这样的简单操作，向量化操作只需直接在数组对象上使用 Python 算术运算符即可。这种向量化方法旨在将循环推入 NumPy 底层的编译层，从而实现更快的执行。

比较以下两个操作的结果：

```py
In [3]: print(compute_reciprocals(values))
        print(1.0 / values)
Out[3]: [0.11111111 0.25       1.         0.33333333 0.125     ]
        [0.11111111 0.25       1.         0.33333333 0.125     ]
```

查看我们大数组的执行时间，我们看到它完成的速度比 Python 循环快了几个数量级：

```py
In [4]: %timeit (1.0 / big_array)
Out[4]: 2.54 ms ± 383 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
```

NumPy 中的向量化操作是通过 ufuncs 实现的，其主要目的是在 NumPy 数组中快速执行重复操作。Ufuncs 非常灵活——我们之前看到了标量和数组之间的操作，但我们也可以在两个数组之间进行操作：

```py
In [5]: np.arange(5) / np.arange(1, 6)
Out[5]: array([0.        , 0.5       , 0.66666667, 0.75      , 0.8       ])
```

并且 ufunc 操作不仅限于一维数组。它们也可以作用于多维数组：

```py
In [6]: x = np.arange(9).reshape((3, 3))
        2 ** x
Out[6]: array([[  1,   2,   4],
               [  8,  16,  32],
               [ 64, 128, 256]])
```

通过 ufunc 的向量化计算几乎总是比使用 Python 循环实现的对应计算更有效率，特别是数组增长时。在 NumPy 脚本中看到这样的循环时，您应该考虑是否可以用向量化表达式替代它。

# 探索 NumPy 的 Ufuncs

Ufuncs 有两种类型：*一元 ufuncs*，作用于单个输入，和*二元 ufuncs*，作用于两个输入。我们将在这里看到这两种类型的函数示例。

## 数组算术

NumPy 的 ufunc 使用起来非常自然，因为它们利用了 Python 的本机算术运算符。可以使用标准的加法、减法、乘法和除法：

```py
In [7]: x = np.arange(4)
        print("x      =", x)
        print("x + 5  =", x + 5)
        print("x - 5  =", x - 5)
        print("x * 2  =", x * 2)
        print("x / 2  =", x / 2)
        print("x // 2 =", x // 2)  # floor division
Out[7]: x      = [0 1 2 3]
        x + 5  = [5 6 7 8]
        x - 5  = [-5 -4 -3 -2]
        x * 2  = [0 2 4 6]
        x / 2  = [0.  0.5 1.  1.5]
        x // 2 = [0 0 1 1]
```

还有一个一元 ufunc 用于求反，一个`**`运算符用于指数运算，一个`%`运算符用于求模：

```py
In [8]: print("-x     = ", -x)
        print("x ** 2 = ", x ** 2)
        print("x % 2  = ", x % 2)
Out[8]: -x     =  [ 0 -1 -2 -3]
        x ** 2 =  [0 1 4 9]
        x % 2  =  [0 1 0 1]
```

此外，这些操作可以随意组合在一起，而且遵循标准的操作顺序：

```py
In [9]: -(0.5*x + 1) ** 2
Out[9]: array([-1.  , -2.25, -4.  , -6.25])
```

所有这些算术操作都只是围绕着 NumPy 中特定 ufunc 的方便包装。例如，`+`运算符是`add` ufunc 的一个包装器。

```py
In [10]: np.add(x, 2)
Out[10]: array([2, 3, 4, 5])
```

表 6-1 列出了 NumPy 中实现的算术运算符。

表 6-1\. NumPy 中实现的算术运算符

| 运算符 | 等效 ufunc | 描述 |
| --- | --- | --- |
| `+` | `np.add` | 加法（例如，`1 + 1 = 2`） |
| `-` | `np.subtract` | 减法（例如，`3 - 2 = 1`） |
| `-` | `np.negative` | 一元取反（例如，`-2`） |
| `*` | `np.multiply` | 乘法（例如，`2 * 3 = 6`） |
| `/` | `np.divide` | 除法（例如，`3 / 2 = 1.5`） |
| `//` | `np.floor_divide` | 地板除法（例如，`3 // 2 = 1`） |
| `**` | `np.power` | 指数运算（例如，`2 ** 3 = 8`） |
| `%` | `np.mod` | 取模/取余数（例如，`9 % 4 = 1`） |

此外，还有布尔/位运算符；我们将在第九章中探索这些。

## 绝对值

就像 NumPy 理解 Python 内置的算术运算符一样，它也理解 Python 内置的绝对值函数：

```py
In [11]: x = np.array([-2, -1, 0, 1, 2])
         abs(x)
Out[11]: array([2, 1, 0, 1, 2])
```

对应的 NumPy ufunc 是`np.absolute`，也可以用别名`np.abs`调用：

```py
In [12]: np.absolute(x)
Out[12]: array([2, 1, 0, 1, 2])
```

```py
In [13]: np.abs(x)
Out[13]: array([2, 1, 0, 1, 2])
```

这个 ufunc 也可以处理复杂数据，此时它返回幅度：

```py
In [14]: x = np.array([3 - 4j, 4 - 3j, 2 + 0j, 0 + 1j])
         np.abs(x)
Out[14]: array([5., 5., 2., 1.])
```

## 三角函数

NumPy 提供了大量有用的 ufunc，对于数据科学家来说，其中一些最有用的是三角函数。我们将从定义一组角度开始：

```py
In [15]: theta = np.linspace(0, np.pi, 3)
```

现在我们可以在这些值上计算一些三角函数了：

```py
In [16]: print("theta      = ", theta)
         print("sin(theta) = ", np.sin(theta))
         print("cos(theta) = ", np.cos(theta))
         print("tan(theta) = ", np.tan(theta))
Out[16]: theta      =  [0.         1.57079633 3.14159265]
         sin(theta) =  [0.0000000e+00 1.0000000e+00 1.2246468e-16]
         cos(theta) =  [ 1.000000e+00  6.123234e-17 -1.000000e+00]
         tan(theta) =  [ 0.00000000e+00  1.63312394e+16 -1.22464680e-16]
```

这些值计算精度达到了机器精度，这就是为什么应该是零的值并不总是确切为零。反三角函数也是可用的：

```py
In [17]: x = [-1, 0, 1]
         print("x         = ", x)
         print("arcsin(x) = ", np.arcsin(x))
         print("arccos(x) = ", np.arccos(x))
         print("arctan(x) = ", np.arctan(x))
Out[17]: x         =  [-1, 0, 1]
         arcsin(x) =  [-1.57079633  0.          1.57079633]
         arccos(x) =  [3.14159265 1.57079633 0.        ]
         arctan(x) =  [-0.78539816  0.          0.78539816]
```

## 指数和对数

在 NumPy 的 ufunc 中还有其他常见的操作，如指数函数：

```py
In [18]: x = [1, 2, 3]
         print("x   =", x)
         print("e^x =", np.exp(x))
         print("2^x =", np.exp2(x))
         print("3^x =", np.power(3., x))
Out[18]: x   = [1, 2, 3]
         e^x = [ 2.71828183  7.3890561  20.08553692]
         2^x = [2. 4. 8.]
         3^x = [ 3.  9. 27.]
```

逆指数函数，对数函数也是可用的。基本的`np.log`给出自然对数；如果你偏好计算以 2 为底或以 10 为底的对数，这些也是可用的：

```py
In [19]: x = [1, 2, 4, 10]
         print("x        =", x)
         print("ln(x)    =", np.log(x))
         print("log2(x)  =", np.log2(x))
         print("log10(x) =", np.log10(x))
Out[19]: x        = [1, 2, 4, 10]
         ln(x)    = [0.         0.69314718 1.38629436 2.30258509]
         log2(x)  = [0.         1.         2.         3.32192809]
         log10(x) = [0.         0.30103    0.60205999 1.        ]
```

还有一些专门用于保持极小输入精度的版本：

```py
In [20]: x = [0, 0.001, 0.01, 0.1]
         print("exp(x) - 1 =", np.expm1(x))
         print("log(1 + x) =", np.log1p(x))
Out[20]: exp(x) - 1 = [0.         0.0010005  0.01005017 0.10517092]
         log(1 + x) = [0.         0.0009995  0.00995033 0.09531018]
```

当`x`非常小时，这些函数比使用原始的`np.log`或`np.exp`给出更精确的值。

## 专用的 Ufuncs

NumPy 还有许多其他的 ufunc 可用，包括双曲三角函数，位运算，比较操作，弧度转角度的转换，取整和余数等等。查阅 NumPy 文档会发现许多有趣的功能。

另一个更专业的 ufunc 来源是子模块`scipy.special`。如果你想在数据上计算一些不常见的数学函数，很可能它已经在`scipy.special`中实现了。这里有太多函数无法一一列出，但以下代码片段展示了在统计上可能会遇到的一些函数：

```py
In [21]: from scipy import special
```

```py
In [22]: # Gamma functions (generalized factorials) and related functions
         x = [1, 5, 10]
         print("gamma(x)     =", special.gamma(x))
         print("ln|gamma(x)| =", special.gammaln(x))
         print("beta(x, 2)   =", special.beta(x, 2))
Out[22]: gamma(x)     = [1.0000e+00 2.4000e+01 3.6288e+05]
         ln|gamma(x)| = [ 0.          3.17805383 12.80182748]
         beta(x, 2)   = [0.5        0.03333333 0.00909091]
```

```py
In [23]: # Error function (integral of Gaussian),
         # its complement, and its inverse
         x = np.array([0, 0.3, 0.7, 1.0])
         print("erf(x)  =", special.erf(x))
         print("erfc(x) =", special.erfc(x))
         print("erfinv(x) =", special.erfinv(x))
Out[23]: erf(x)  = [0.         0.32862676 0.67780119 0.84270079]
         erfc(x) = [1.         0.67137324 0.32219881 0.15729921]
         erfinv(x) = [0.         0.27246271 0.73286908        inf]
```

NumPy 和`scipy.special`中还有许多其他的 ufunc 可用。由于这些包的文档可以在线获取，因此通过类似“gamma function python”的网络搜索通常可以找到相关信息。

# 高级 Ufunc 功能

许多 NumPy 用户在不完全了解它们的全部特性的情况下就使用了 ufuncs。我在这里概述一些 ufunc 的专门特性。

## 指定输出

对于大型计算，有时指定计算结果存储的数组是很有用的。对于所有 ufunc，这可以通过函数的`out`参数来完成：

```py
In [24]: x = np.arange(5)
         y = np.empty(5)
         np.multiply(x, 10, out=y)
         print(y)
Out[24]: [ 0. 10. 20. 30. 40.]
```

这甚至可以与数组视图一起使用。例如，我们可以将计算结果写入指定数组的每隔一个元素：

```py
In [25]: y = np.zeros(10)
         np.power(2, x, out=y[::2])
         print(y)
Out[25]: [ 1.  0.  2.  0.  4.  0.  8.  0. 16.  0.]
```

如果我们改为写成`y[::2] = 2 ** x`，这将导致创建一个临时数组来保存`2 ** x`的结果，然后进行第二个操作将这些值复制到`y`数组中。对于这样一个小的计算来说，这并不会有太大的区别，但是对于非常大的数组来说，通过谨慎使用`out`参数可以节省内存。

## 聚合

对于二元 ufunc，聚合可以直接从对象中计算。例如，如果我们想要使用特定操作*减少*一个数组，我们可以使用任何 ufunc 的`reduce`方法。reduce 会重复地将给定操作应用于数组的元素，直到只剩下一个单一的结果。

例如，对`add` ufunc 调用`reduce`将返回数组中所有元素的总和：

```py
In [26]: x = np.arange(1, 6)
         np.add.reduce(x)
Out[26]: 15
```

类似地，对`multiply` ufunc 调用`reduce`将得到数组所有元素的乘积：

```py
In [27]: np.multiply.reduce(x)
Out[27]: 120
```

如果我们想要存储计算的所有中间结果，我们可以使用`accumulate`：

```py
In [28]: np.add.accumulate(x)
Out[28]: array([ 1,  3,  6, 10, 15])
```

```py
In [29]: np.multiply.accumulate(x)
Out[29]: array([  1,   2,   6,  24, 120])
```

请注意，对于这些特定情况，有专门的 NumPy 函数来计算结果（`np.sum`，`np.prod`，`np.cumsum`，`np.cumprod`），我们将在第七章中探讨。

## 外积

最后，任何 ufunc 都可以使用`outer`方法计算两个不同输入的所有对的输出。这使您可以在一行中执行诸如创建乘法表之类的操作：

```py
In [30]: x = np.arange(1, 6)
         np.multiply.outer(x, x)
Out[30]: array([[ 1,  2,  3,  4,  5],
                [ 2,  4,  6,  8, 10],
                [ 3,  6,  9, 12, 15],
                [ 4,  8, 12, 16, 20],
                [ 5, 10, 15, 20, 25]])
```

`ufunc.at`和`ufunc.reduceat`方法同样也是非常有用的，我们将在第十章中探讨它们。

我们还将遇到 ufunc 能够在不同形状和大小的数组之间执行操作的能力，这一组操作被称为*广播*。这个主题非常重要，我们将专门为其设立一整章（参见第八章）。

# Ufuncs：了解更多

更多有关通用函数的信息（包括可用函数的完整列表）可以在[NumPy](http://www.numpy.org)和[SciPy](http://www.scipy.org)文档网站上找到。

请回忆，您还可以通过在 IPython 中导入包并使用 IPython 的 tab 补全和帮助（`?`）功能来直接访问信息，如第一章中所述。
