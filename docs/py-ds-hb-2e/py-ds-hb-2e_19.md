# 第十六章：处理缺失数据

许多教程中找到的数据与现实世界中的数据之间的区别在于，现实世界的数据很少是干净且同质的。特别是，许多有趣的数据集会有一些数据缺失。更复杂的是，不同的数据源可能以不同的方式指示缺失数据。

在本章中，我们将讨论一些有关缺失数据的一般考虑事项，看看 Pandas 如何选择表示它，并探索一些处理 Python 中缺失数据的内置 Pandas 工具。在本书中，我将通常将缺失数据总称为*null*、`*NaN*`或*NA*值。

# 缺失数据约定中的权衡

已开发出许多方法来跟踪表格或`DataFrame`中缺失数据的存在。通常，它们围绕两种策略之一展开：使用*掩码*全局指示缺失值，或选择指示缺失条目的*哨兵值*。

在掩码方法中，掩码可以是一个完全独立的布尔数组，也可以涉及在数据表示中占用一个比特来局部指示值的空状态。

在哨兵方法中，哨兵值可以是一些特定于数据的约定，比如用 -9999 表示缺失的整数值或一些罕见的比特模式，或者可以是一个更全局的约定，比如用`NaN`（不是数字）表示缺失的浮点值，这是 IEEE 浮点规范的一部分。

这两种方法都不是没有权衡的。使用单独的掩码数组需要分配额外的布尔数组，这在存储和计算上都增加了开销。哨兵值会减少可以表示的有效值范围，并且可能需要额外的（通常是非优化的）CPU 和 GPU 算术逻辑，因为常见的特殊值如`NaN`并不适用于所有数据类型。

就像大多数情况下没有普遍适用的最佳选择一样，不同的语言和系统使用不同的约定。例如，R 语言使用每种数据类型内的保留比特模式作为指示缺失数据的哨兵值，而 SciDB 系统使用附加到每个单元的额外字节来指示 NA 状态。

# Pandas 中的缺失数据

Pandas 处理缺失值的方式受其对 NumPy 包的依赖限制，后者对非浮点数据类型没有内置的 NA 值概念。

也许 Pandas 本可以效仿 R 在指定每个单独数据类型的位模式以指示空值方面的领先地位，但是这种方法事实证明相当笨拙。虽然 R 只有 4 种主要数据类型，但是 NumPy 支持的数据类型远远超过这个数字：例如，虽然 R 只有一个整数类型，但是考虑到可用的位宽、符号性和编码的字节顺序，NumPy 支持 14 种基本整数类型。在所有可用的 NumPy 类型中保留特定的位模式将导致在各种类型的操作中特殊处理大量操作，很可能甚至需要新的 NumPy 软件包的分支。此外，对于较小的数据类型（如 8 位整数），牺牲一位用作掩码将显著减少其可以表示的值的范围。

由于这些限制和权衡，Pandas 有两种存储和操作空值的“模式”：

+   默认模式是使用基于哨兵值的缺失数据方案，哨兵值为`NaN`或`None`，具体取决于数据类型。

+   或者，您可以选择使用 Pandas 提供的可空数据类型（dtypes）（稍后在本章讨论），这将导致创建一个伴随的掩码数组来跟踪缺失的条目。然后，这些缺失的条目将被呈现给用户作为特殊的`pd.NA`值。

无论哪种情况，Pandas API 提供的数据操作和操作将以可预测的方式处理和传播这些缺失的条目。但是为了对为什么会做出这些选择有一些直觉，让我们快速探讨一下`None`、`NaN`和`NA`中固有的权衡。像往常一样，我们将从导入 NumPy 和 Pandas 开始：

```py
In [1]: import numpy as np
        import pandas as pd
```

## `None`作为哨兵值

对于某些数据类型，Pandas 使用`None`作为哨兵值。`None`是一个 Python 对象，这意味着包含`None`的任何数组必须具有`dtype=object`，即它必须是 Python 对象的序列。

例如，观察将`None`传递给 NumPy 数组会发生什么：

```py
In [2]: vals1 = np.array([1, None, 2, 3])
        vals1
Out[2]: array([1, None, 2, 3], dtype=object)
```

这个`dtype=object`意味着 NumPy 可以推断出数组内容的最佳公共类型表示是 Python 对象。以这种方式使用`None`的缺点是数据的操作将在 Python 级别完成，其开销比通常对具有本地类型的数组所见的快速操作要大得多：

```py
In [3]: %timeit np.arange(1E6, dtype=int).sum()
Out[3]: 2.73 ms ± 288 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
```

```py
In [4]: %timeit np.arange(1E6, dtype=object).sum()
Out[4]: 92.1 ms ± 3.42 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
```

此外，因为 Python 不支持与`None`的算术运算，像`sum`或`min`这样的聚合通常会导致错误：

```py
In [5]: vals1.sum()
TypeError: unsupported operand type(s) for +: 'int' and 'NoneType'
```

因此，Pandas 在其数值数组中不使用`None`作为哨兵值。

## `NaN`：缺失的数值数据

另一个缺失数据哨兵，`NaN`，是不同的；它是一种特殊的浮点值，在所有使用标准 IEEE 浮点表示的系统中都被识别：

```py
In [6]: vals2 = np.array([1, np.nan, 3, 4])
        vals2
Out[6]: array([ 1., nan,  3.,  4.])
```

请注意，NumPy 为此数组选择了本地的浮点类型：这意味着与之前的对象数组不同，此数组支持快速操作并推入编译代码中。请记住，`NaN`有点像数据病毒——它会感染到任何它接触到的其他对象。

不管进行何种操作，带有`NaN`的算术运算的结果都将是另一个`NaN`：

```py
In [7]: 1 + np.nan
Out[7]: nan
```

```py
In [8]: 0 * np.nan
Out[8]: nan
```

这意味着对值的聚合是定义良好的（即，它们不会导致错误），但并不总是有用的：

```py
In [9]: vals2.sum(), vals2.min(), vals2.max()
Out[9]: (nan, nan, nan)
```

也就是说，NumPy 确实提供了对`NaN`敏感的聚合函数版本，将忽略这些缺失值：

```py
In [10]: np.nansum(vals2), np.nanmin(vals2), np.nanmax(vals2)
Out[10]: (8.0, 1.0, 4.0)
```

`NaN`的主要缺点是它是特定的浮点值；对于整数、字符串或其他类型，都没有等价的`NaN`值。

## Pandas 中的 NaN 和 None

`NaN`和`None`都有它们的用途，而且 Pandas 几乎可以在它们之间自由转换，视情况而定：

```py
In [11]: pd.Series([1, np.nan, 2, None])
Out[11]: 0    1.0
         1    NaN
         2    2.0
         3    NaN
         dtype: float64
```

对于没有可用哨兵值的类型，当存在 NA 值时，Pandas 会自动进行类型转换。例如，如果我们将整数数组中的一个值设置为`np.nan`，它将自动提升为浮点类型以容纳 NA：

```py
In [12]: x = pd.Series(range(2), dtype=int)
         x
Out[12]: 0    0
         1    1
         dtype: int64
```

```py
In [13]: x[0] = None
         x
Out[13]: 0    NaN
         1    1.0
         dtype: float64
```

请注意，除了将整数数组转换为浮点数外，Pandas 还会自动将`None`转换为`NaN`值。

虽然这种类型的魔法操作可能与像 R 这样的特定领域语言中的 NA 值的更统一方法相比显得有些投机，但是 Pandas 的哨兵/转换方法在实践中运作得非常好，并且据我经验，很少引起问题。

表 16-1 列出了引入 NA 值时 Pandas 中的提升转换规则。

表 16-1\. Pandas 按类型处理 NA 值

| 类型 | 存储 NA 时的转换 | NA 哨兵值 |
| --- | --- | --- |
| `floating` | 无变化 | `np.nan` |
| `object` | 无变化 | `None`或`np.nan` |
| `integer` | 转换为`float64` | `np.nan` |
| `boolean` | 转换为`object` | `None`或`np.nan` |

请记住，在 Pandas 中，字符串数据始终以`object`类型存储。

# Pandas 可空数据类型

在早期版本的 Pandas 中，`NaN`和`None`作为哨兵值是唯一可用的缺失数据表示。这引入的主要困难是隐式类型转换：例如，无法表示真正的整数数组带有缺失数据。

为了解决这个问题，Pandas 后来添加了*可空数据类型*，它们通过名称的大写区分于常规数据类型（例如，`pd.Int32`与`np.int32`）。为了向后兼容，只有在明确请求时才会使用这些可空数据类型。

例如，这是一个带有缺失数据的整数`Series`，由包含所有三种可用缺失数据标记的列表创建：

```py
In [14]: pd.Series([1, np.nan, 2, None, pd.NA], dtype='Int32')
Out[14]: 0       1
         1    <NA>
         2       2
         3    <NA>
         4    <NA>
         dtype: Int32
```

这种表示可以在本章剩余的所有操作中与其他表示方法交替使用。

# 操作空值

正如我们所见，Pandas 将 `None`、`NaN` 和 `NA` 视为基本可以互换，用于指示缺失或空值。为了促进这一约定，Pandas 提供了几种方法来检测、删除和替换 Pandas 数据结构中的空值。它们包括：

`isnull`

生成一个指示缺失值的布尔掩码

`notnull`

`isnull` 的反操作

`dropna`

返回数据的过滤版本

`fillna`

返回填充或插补了缺失值的数据副本

我们将以对这些程序的简要探索和演示来结束本章。

## 检测空值

Pandas 数据结构有两个有用的方法来检测空数据：`isnull` 和 `notnull`。任何一个都将返回数据的布尔掩码。例如：

```py
In [15]: data = pd.Series([1, np.nan, 'hello', None])
```

```py
In [16]: data.isnull()
Out[16]: 0    False
         1     True
         2    False
         3     True
         dtype: bool
```

正如在 第十四章 中提到的那样，布尔掩码可以直接用作 `Series` 或 `DataFrame` 的索引：

```py
In [17]: data[data.notnull()]
Out[17]: 0        1
         2    hello
         dtype: object
```

对于 `DataFrame` 对象，`isnull()` 和 `notnull()` 方法生成类似的布尔结果。

## 删除空值

除了这些掩码方法之外，还有方便的方法 `dropna`（用于删除 NA 值）和 `fillna`（用于填充 NA 值）。对于 `Series`，结果是直接的：

```py
In [18]: data.dropna()
Out[18]: 0        1
         2    hello
         dtype: object
```

对于 `DataFrame`，有更多的选择。考虑以下 `DataFrame`：

```py
In [19]: df = pd.DataFrame([[1,      np.nan, 2],
                            [2,      3,      5],
                            [np.nan, 4,      6]])
         df
Out[19]:      0    1  2
         0  1.0  NaN  2
         1  2.0  3.0  5
         2  NaN  4.0  6
```

我们不能从 `DataFrame` 中删除单个值；我们只能删除整行或整列。根据应用程序的不同，您可能需要其中一个，因此 `dropna` 包含了一些 `DataFrame` 的选项。

默认情况下，`dropna` 将删除*任何*存在空值的行：

```py
In [20]: df.dropna()
Out[20]:      0    1  2
         1  2.0  3.0  5
```

或者，您可以沿不同的轴删除 NA 值。使用 `axis=1` 或 `axis='columns'` 将删除包含空值的所有列：

```py
In [21]: df.dropna(axis='columns')
Out[21]:    2
         0  2
         1  5
         2  6
```

但是这样会丢掉一些好数据；您可能更感兴趣的是删除具有*所有* NA 值或大多数 NA 值的行或列。这可以通过 `how` 或 `thresh` 参数进行指定，这些参数允许对允许通过的空值数量进行精细控制。

默认值为 `how='any'`，这样任何包含空值的行或列都将被删除。您还可以指定 `how='all'`，这样只会删除包含*所有*空值的行/列：

```py
In [22]: df[3] = np.nan
         df
Out[22]:      0    1  2   3
         0  1.0  NaN  2 NaN
         1  2.0  3.0  5 NaN
         2  NaN  4.0  6 NaN
```

```py
In [23]: df.dropna(axis='columns', how='all')
Out[23]:      0    1  2
         0  1.0  NaN  2
         1  2.0  3.0  5
         2  NaN  4.0  6
```

对于更精细的控制，`thresh` 参数允许您指定保留行/列的最小非空值数：

```py
In [24]: df.dropna(axis='rows', thresh=3)
Out[24]:      0    1  2   3
         1  2.0  3.0  5 NaN
```

在这里，第一行和最后一行已被删除，因为它们各自只包含两个非空值。

## 填充空值

有时候，您不想丢弃 NA 值，而是希望用有效值替换它们。这个值可以是一个单独的数字，比如零，或者可能是一些从好的值中插补或插值出来的值。您可以使用 `isnull` 方法作为掩码来原地进行这个操作，但是因为这是一个常见的操作，Pandas 提供了 `fillna` 方法，它返回一个替换了空值的数组的副本。

考虑以下 `Series`：

```py
In [25]: data = pd.Series([1, np.nan, 2, None, 3], index=list('abcde'),
                           dtype='Int32')
         data
Out[25]: a       1
         b    <NA>
         c       2
         d    <NA>
         e       3
         dtype: Int32
```

我们可以用一个单一的值（如零）填充 NA 条目：

```py
In [26]: data.fillna(0)
Out[26]: a    1
         b    0
         c    2
         d    0
         e    3
         dtype: Int32
```

我们可以指定向前填充以向前传播上一个值：

```py
In [27]: # forward fill
         data.fillna(method='ffill')
Out[27]: a    1
         b    1
         c    2
         d    2
         e    3
         dtype: Int32
```

或者我们可以指定向后填充以向后传播下一个值：

```py
In [28]: # back fill
         data.fillna(method='bfill')
Out[28]: a    1
         b    2
         c    2
         d    3
         e    3
         dtype: Int32
```

对于`DataFrame`，选项类似，但我们还可以指定填充应该沿着的`axis`：

```py
In [29]: df
Out[29]:      0    1  2   3
         0  1.0  NaN  2 NaN
         1  2.0  3.0  5 NaN
         2  NaN  4.0  6 NaN
```

```py
In [30]: df.fillna(method='ffill', axis=1)
Out[30]:      0    1    2    3
         0  1.0  1.0  2.0  2.0
         1  2.0  3.0  5.0  5.0
         2  NaN  4.0  6.0  6.0
```

如果在向前填充时前一个值不可用，NA 值将保留。
