# 第二十四章：高性能 Pandas：eval 和 query

正如我们在之前的章节中已经看到的，PyData 栈的强大建立在 NumPy 和 Pandas 将基本操作推送到低级编译代码中的能力上，通过直观的高级语法：例如 NumPy 中的向量化/广播操作，以及 Pandas 中的分组类型操作。虽然这些抽象对许多常见用例是高效和有效的，但它们经常依赖于临时中间对象的创建，这可能会导致计算时间和内存使用的不必要开销。

为了解决这个问题，Pandas 包括一些方法，允许您直接访问 C 速度操作，而无需昂贵地分配中间数组：`eval` 和 `query`，这些方法依赖于 [NumExpr 包](https://oreil.ly/acvj5)。

# 激励查询和 eval：复合表达式

我们之前已经看到，NumPy 和 Pandas 支持快速的向量化操作；例如，当添加两个数组的元素时：

```py
In [1]: import numpy as np
        rng = np.random.default_rng(42)
        x = rng.random(1000000)
        y = rng.random(1000000)
        %timeit x + y
Out[1]: 2.21 ms ± 142 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
```

如在 第六章 中讨论的，这比通过 Python 循环或理解式添加要快得多：

```py
In [2]: %timeit np.fromiter((xi + yi for xi, yi in zip(x, y)),
                            dtype=x.dtype, count=len(x))
Out[2]: 263 ms ± 43.4 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```

但是当计算复合表达式时，这种抽象可能变得不那么高效。例如，考虑以下表达式：

```py
In [3]: mask = (x > 0.5) & (y < 0.5)
```

因为 NumPy 评估每个子表达式，这大致等同于以下内容：

```py
In [4]: tmp1 = (x > 0.5)
        tmp2 = (y < 0.5)
        mask = tmp1 & tmp2
```

换句话说，*每个中间步骤都显式地分配在内存中*。如果 `x` 和 `y` 数组非常大，这可能导致显著的内存和计算开销。NumExpr 库使您能够逐个元素地计算这种复合表达式，而无需分配完整的中间数组。有关更多详细信息，请参阅 [NumExpr 文档](https://oreil.ly/acvj5)，但目前足以说，该库接受一个 *字符串*，该字符串给出您想计算的 NumPy 风格表达式：

```py
In [5]: import numexpr
        mask_numexpr = numexpr.evaluate('(x > 0.5) & (y < 0.5)')
        np.all(mask == mask_numexpr)
Out[5]: True
```

这里的好处在于，NumExpr 以避免可能的临时数组方式评估表达式，因此对于长数组上的长序列计算比 NumPy 要高效得多。我们将在这里讨论的 Pandas `eval` 和 `query` 工具在概念上类似，并且本质上是 NumExpr 功能的 Pandas 特定包装。

# pandas.eval 用于高效操作

Pandas 中的 `eval` 函数使用字符串表达式来高效地计算 `DataFrame` 对象上的操作。例如，考虑以下数据：

```py
In [6]: import pandas as pd
        nrows, ncols = 100000, 100
        df1, df2, df3, df4 = (pd.DataFrame(rng.random((nrows, ncols)))
                              for i in range(4))
```

要使用典型的 Pandas 方法计算所有四个 `DataFrame` 的总和，我们只需写出总和：

```py
In [7]: %timeit df1 + df2 + df3 + df4
Out[7]: 73.2 ms ± 6.72 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
```

可以通过构造字符串表达式来使用 `pd.eval` 计算相同的结果：

```py
In [8]: %timeit pd.eval('df1 + df2 + df3 + df4')
Out[8]: 34 ms ± 4.2 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
```

`eval` 版本的这个表达式大约快 50%（并且使用的内存要少得多），同时给出相同的结果：

```py
In [9]: np.allclose(df1 + df2 + df3 + df4,
                    pd.eval('df1 + df2 + df3 + df4'))
Out[9]: True
```

`pd.eval`支持广泛的操作。为了展示这些操作，我们将使用以下整数数据：

```py
In [10]: df1, df2, df3, df4, df5 = (pd.DataFrame(rng.integers(0, 1000, (100, 3)))
                                    for i in range(5))
```

下面是`pd.eval`支持的操作的总结：

算术运算符

`pd.eval`支持所有算术运算符。例如：

```py
In [11]: result1 = -df1 * df2 / (df3 + df4) - df5
         result2 = pd.eval('-df1 * df2 / (df3 + df4) - df5')
         np.allclose(result1, result2)
Out[11]: True
```

比较运算符

`pd.eval`支持所有比较运算符，包括链式表达式：

```py
In [12]: result1 = (df1 < df2) & (df2 <= df3) & (df3 != df4)
         result2 = pd.eval('df1 < df2 <= df3 != df4')
         np.allclose(result1, result2)
Out[12]: True
```

位运算符

`pd.eval`支持`&`和`|`位运算符：

```py
In [13]: result1 = (df1 < 0.5) & (df2 < 0.5) | (df3 < df4)
         result2 = pd.eval('(df1 < 0.5) & (df2 < 0.5) | (df3 < df4)')
         np.allclose(result1, result2)
Out[13]: True
```

此外，它还支持在布尔表达式中使用字面量`and`和`or`：

```py
In [14]: result3 = pd.eval('(df1 < 0.5) and (df2 < 0.5) or (df3 < df4)')
         np.allclose(result1, result3)
Out[14]: True
```

对象属性和索引

`pd.eval`支持通过`obj.attr`语法和`obj[index]`语法访问对象属性：

```py
In [15]: result1 = df2.T[0] + df3.iloc[1]
         result2 = pd.eval('df2.T[0] + df3.iloc[1]')
         np.allclose(result1, result2)
Out[15]: True
```

其他操作

其他操作，例如函数调用、条件语句、循环和其他更复杂的构造，目前*不*在`pd.eval`中实现。如果你想执行这些更复杂的表达式类型，可以使用 NumExpr 库本身。

# DataFrame.eval 进行按列操作

就像 Pandas 有一个顶级的`pd.eval`函数一样，`DataFrame`对象也有一个`eval`方法，功能类似。`eval`方法的好处是可以按名称引用列。我们将用这个带标签的数组作为示例：

```py
In [16]: df = pd.DataFrame(rng.random((1000, 3)), columns=['A', 'B', 'C'])
         df.head()
Out[16]:           A         B         C
         0  0.850888  0.966709  0.958690
         1  0.820126  0.385686  0.061402
         2  0.059729  0.831768  0.652259
         3  0.244774  0.140322  0.041711
         4  0.818205  0.753384  0.578851
```

使用前面一节中的`pd.eval`，我们可以像这样计算三个列的表达式：

```py
In [17]: result1 = (df['A'] + df['B']) / (df['C'] - 1)
         result2 = pd.eval("(df.A + df.B) / (df.C - 1)")
         np.allclose(result1, result2)
Out[17]: True
```

`DataFrame.eval`方法允许更简洁地评估列的表达式：

```py
In [18]: result3 = df.eval('(A + B) / (C - 1)')
         np.allclose(result1, result3)
Out[18]: True
```

请注意，在这里我们将*列名视为评估表达式中的变量*，结果正是我们希望的。

## 在 DataFrame.eval 中的赋值

除了刚才讨论的选项之外，`DataFrame.eval`还允许对任何列进行赋值。让我们使用之前的`DataFrame`，它有列`'A'`，`'B'`和`'C'`：

```py
In [19]: df.head()
Out[19]:           A         B         C
         0  0.850888  0.966709  0.958690
         1  0.820126  0.385686  0.061402
         2  0.059729  0.831768  0.652259
         3  0.244774  0.140322  0.041711
         4  0.818205  0.753384  0.578851
```

我们可以使用`df.eval`创建一个新的列`'D'`，并将其赋值为从其他列计算得到的值：

```py
In [20]: df.eval('D = (A + B) / C', inplace=True)
         df.head()
Out[20]:           A         B         C          D
         0  0.850888  0.966709  0.958690   1.895916
         1  0.820126  0.385686  0.061402  19.638139
         2  0.059729  0.831768  0.652259   1.366782
         3  0.244774  0.140322  0.041711   9.232370
         4  0.818205  0.753384  0.578851   2.715013
```

以同样的方式，任何现有的列都可以被修改：

```py
In [21]: df.eval('D = (A - B) / C', inplace=True)
         df.head()
Out[21]:           A         B         C         D
         0  0.850888  0.966709  0.958690 -0.120812
         1  0.820126  0.385686  0.061402  7.075399
         2  0.059729  0.831768  0.652259 -1.183638
         3  0.244774  0.140322  0.041711  2.504142
         4  0.818205  0.753384  0.578851  0.111982
```

## DataFrame.eval 中的本地变量

`DataFrame.eval`方法支持一种额外的语法，使其能够与本地 Python 变量一起使用。考虑以下内容：

```py
In [22]: column_mean = df.mean(1)
         result1 = df['A'] + column_mean
         result2 = df.eval('A + @column_mean')
         np.allclose(result1, result2)
Out[22]: True
```

这里的`@`字符标记的是*变量名*而不是*列名*，并且让你能够高效地评估涉及两个“命名空间”的表达式：列的命名空间和 Python 对象的命名空间。请注意，这个`@`字符只支持`DataFrame.eval`*方法*，而不支持`pandas.eval`*函数*，因为`pandas.eval`函数只能访问一个（Python）命名空间。

# `DataFrame.query`方法

`DataFrame`还有一个基于评估字符串的方法，叫做`query`。考虑以下内容：

```py
In [23]: result1 = df[(df.A < 0.5) & (df.B < 0.5)]
         result2 = pd.eval('df[(df.A < 0.5) & (df.B < 0.5)]')
         np.allclose(result1, result2)
Out[23]: True
```

正如我们在讨论`DataFrame.eval`时使用的示例一样，这是一个涉及`DataFrame`列的表达式。然而，它不能使用`DataFrame.eval`语法表示！相反，对于这种类型的筛选操作，你可以使用`query`方法：

```py
In [24]: result2 = df.query('A < 0.5 and B < 0.5')
         np.allclose(result1, result2)
Out[24]: True
```

与掩码表达式相比，这不仅是更有效的计算，而且更易于阅读和理解。请注意，`query`方法还接受`@`标志来标记本地变量：

```py
In [25]: Cmean = df['C'].mean()
         result1 = df[(df.A < Cmean) & (df.B < Cmean)]
         result2 = df.query('A < @Cmean and B < @Cmean')
         np.allclose(result1, result2)
Out[25]: True
```

# 性能：何时使用这些函数

在考虑是否使用`eval`和`query`时，有两个考虑因素：*计算时间*和*内存使用*。内存使用是最可预测的方面。正如前面提到的，涉及 NumPy 数组或 Pandas `DataFrame`的每个复合表达式都会导致临时数组的隐式创建。例如，这个：

```py
In [26]: x = df[(df.A < 0.5) & (df.B < 0.5)]
```

大致相当于这个：

```py
In [27]: tmp1 = df.A < 0.5
         tmp2 = df.B < 0.5
         tmp3 = tmp1 & tmp2
         x = df[tmp3]
```

如果临时`DataFrame`的大小与您可用的系统内存（通常为几个千兆字节）相比显著，则使用`eval`或`query`表达式是个好主意。您可以使用以下命令检查数组的大约大小（以字节为单位）：

```py
In [28]: df.values.nbytes
Out[28]: 32000
```

就性能而言，即使您没有使用完系统内存，`eval`可能会更快。问题在于您的临时对象与系统的 L1 或 L2 CPU 缓存大小（通常为几兆字节）相比如何；如果它们要大得多，那么`eval`可以避免在不同内存缓存之间移动值时可能出现的某些潜在缓慢。实际上，我发现传统方法与`eval`/`query`方法之间的计算时间差异通常不显著——如果有什么的话，对于较小的数组来说，传统方法更快！`eval`/`query`的好处主要在于节省内存，以及它们有时提供的更清晰的语法。

我们在这里已经涵盖了关于`eval`和`query`的大部分细节；有关更多信息，请参阅 Pandas 文档。特别是，可以为运行这些查询指定不同的解析器和引擎；有关详细信息，请参阅文档中的[“提升性能”部分](https://oreil.ly/DHNy8)。

# 更多资源

在本书的这一部分中，我们已经涵盖了有效使用 Pandas 进行数据分析的许多基础知识。但我们的讨论还有很多内容未涉及。要了解更多关于 Pandas 的信息，我推荐以下资源：

[Pandas 在线文档](http://pandas.pydata.org)

这是完整文档的首选来源。虽然文档中的示例通常基于小型生成的数据集，但选项的描述是全面的，并且通常非常有助于理解各种函数的使用。

[*Python for Data Analysis*](https://oreil.ly/0hdsf)

由 Pandas 的原始创建者 Wes McKinney 撰写，这本书包含了比我们在本章中有空间讨论的 Pandas 包更多的细节。特别是，McKinney 深入探讨了用于时间序列的工具，这些工具是他作为金融顾问的核心内容。这本书还包含许多将 Pandas 应用于从实际数据集中获得洞察的有趣例子。

[*Effective Pandas*](https://oreil.ly/cn1ls)

Pandas 开发者 Tom Augspurger 的这本简短电子书，简洁地概述了如何有效和惯用地使用 Pandas 库的全部功能。

[PyVideo 上的 Pandas](https://oreil.ly/mh4wI)

从 PyCon 到 SciPy 再到 PyData，许多会议都有 Pandas 开发者和高级用户提供的教程。特别是 PyCon 的教程通常由经过严格筛选的优秀演讲者提供。

结合这些资源，再加上这些章节中的详细介绍，我希望你能够准备好使用 Pandas 解决任何遇到的数据分析问题！
