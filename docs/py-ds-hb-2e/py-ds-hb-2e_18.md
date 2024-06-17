# 第十五章：在 Pandas 中操作数据

NumPy 的一个优点是它允许我们执行快速的逐元素操作，包括基本算术（加法、减法、乘法等）和更复杂的操作（三角函数、指数和对数函数等）。Pandas 从 NumPy 继承了许多这些功能，并且在第六章介绍的 ufuncs 对此至关重要。

然而，Pandas 还包括一些有用的技巧：对于像否定和三角函数这样的一元操作，这些 ufuncs 将在输出中 *保留索引和列标签*；对于像加法和乘法这样的二元操作，当将对象传递给 ufunc 时，Pandas 将自动 *对齐索引*。这意味着保持数据的上下文和组合来自不同来源的数据（这两个任务对于原始 NumPy 数组来说可能是错误的）在 Pandas 中基本上变得十分简单。我们还将看到在一维 `Series` 结构和二维 `DataFrame` 结构之间存在着明确定义的操作。

# Ufuncs：索引保留

因为 Pandas 是设计用于与 NumPy 协作的，任何 NumPy 的 ufunc 都可以在 Pandas 的 `Series` 和 `DataFrame` 对象上使用。让我们先定义一个简单的 `Series` 和 `DataFrame` 来演示这一点：

```py
In [1]: import pandas as pd
        import numpy as np
```

```py
In [2]: rng = np.random.default_rng(42)
        ser = pd.Series(rng.integers(0, 10, 4))
        ser
Out[2]: 0    0
        1    7
        2    6
        3    4
        dtype: int64
```

```py
In [3]: df = pd.DataFrame(rng.integers(0, 10, (3, 4)),
                          columns=['A', 'B', 'C', 'D'])
        df
Out[3]:    A  B  C  D
        0  4  8  0  6
        1  2  0  5  9
        2  7  7  7  7
```

如果我们在这些对象中的任一对象上应用 NumPy 的 ufunc，结果将是另一个 Pandas 对象 *并保留索引*：

```py
In [4]: np.exp(ser)
Out[4]: 0       1.000000
        1    1096.633158
        2     403.428793
        3      54.598150
        dtype: float64
```

对于更复杂的操作序列，情况也是如此：

```py
In [5]: np.sin(df * np.pi / 4)
Out[5]:               A             B         C         D
        0  1.224647e-16 -2.449294e-16  0.000000 -1.000000
        1  1.000000e+00  0.000000e+00 -0.707107  0.707107
        2 -7.071068e-01 -7.071068e-01 -0.707107 -0.707107
```

任何在第六章中讨论过的 ufunc 都可以以类似的方式使用。

# Ufuncs：索引对齐

对于两个 `Series` 或 `DataFrame` 对象的二元操作，Pandas 将在执行操作的过程中对齐索引。这在处理不完整数据时非常方便，我们将在接下来的一些示例中看到。

## Series 中的索引对齐

例如，假设我们正在结合两个不同的数据源，并希望仅找到按 *面积* 排名前三的美国州和按 *人口* 排名前三的美国州：

```py
In [6]: area = pd.Series({'Alaska': 1723337, 'Texas': 695662,
                          'California': 423967}, name='area')
        population = pd.Series({'California': 39538223, 'Texas': 29145505,
                                'Florida': 21538187}, name='population')
```

现在让我们来看看在进行人口密度计算时会发生什么：

```py
In [7]: population / area
Out[7]: Alaska              NaN
        California    93.257784
        Florida             NaN
        Texas         41.896072
        dtype: float64
```

结果数组包含两个输入数组的索引的 *并集*，这可以直接从这些索引中确定：

```py
In [8]: area.index.union(population.index)
Out[8]: Index(['Alaska', 'California', 'Florida', 'Texas'], dtype='object')
```

任何其中一个没有条目的项目都标记有 `NaN`，即“不是数字”，这是 Pandas 标记缺失数据的方式（详见第十六章对缺失数据的进一步讨论）。对于 Python 内置的任何算术表达式，都会实现这种索引匹配；任何缺失值都将被 `NaN` 标记：

```py
In [9]: A = pd.Series([2, 4, 6], index=[0, 1, 2])
        B = pd.Series([1, 3, 5], index=[1, 2, 3])
        A + B
Out[9]: 0    NaN
        1    5.0
        2    9.0
        3    NaN
        dtype: float64
```

如果不希望使用`NaN`值，可以使用适当的对象方法修改填充值，而不是使用操作符。例如，调用`A.add(B)`等效于调用`A + B`，但允许可选地显式指定`A`或`B`中可能缺失元素的填充值：

```py
In [10]: A.add(B, fill_value=0)
Out[10]: 0    2.0
         1    5.0
         2    9.0
         3    5.0
         dtype: float64
```

## 数据帧中的索引对齐

当对`DataFrame`对象进行操作时，*同时*在列和索引上进行类似的对齐：

```py
In [11]: A = pd.DataFrame(rng.integers(0, 20, (2, 2)),
                          columns=['a', 'b'])
         A
Out[11]:     a  b
         0  10  2
         1  16  9
```

```py
In [12]: B = pd.DataFrame(rng.integers(0, 10, (3, 3)),
                          columns=['b', 'a', 'c'])
         B
Out[12]:    b  a  c
         0  5  3  1
         1  9  7  6
         2  4  8  5
```

```py
In [13]: A + B
Out[12]:       a     b   c
         0  13.0   7.0 NaN
         1  23.0  18.0 NaN
         2   NaN   NaN NaN
```

请注意，无论这两个对象中的顺序如何，索引都正确对齐，并且结果中的索引是排序的。与`Series`一样，我们可以使用关联对象的算术方法，并传递任何希望用于替代缺失条目的`fill_value`。这里我们将用`A`中所有值的平均值填充：

```py
In [14]: A.add(B, fill_value=A.values.mean())
Out[14]:        a      b      c
         0  13.00   7.00  10.25
         1  23.00  18.00  15.25
         2  17.25  13.25  14.25
```

表 15-1 列出了 Python 运算符及其相应的 Pandas 对象方法。

表 15-1。Python 运算符与 Pandas 方法的映射

| Python 运算符 | Pandas 方法 |
| --- | --- |
| `+` | `add` |
| `-` | `sub`, `subtract` |
| `*` | `mul`, `multiply` |
| `/` | `truediv`, `div`, `divide` |
| `//` | `floordiv` |
| `%` | `mod` |
| `**` | `pow` |

# Ufuncs：DataFrame 与 Series 之间的操作

当对`DataFrame`和`Series`进行操作时，索引和列的对齐方式类似地保持，并且结果类似于二维数组和一维 NumPy 数组之间的操作。考虑一种常见的操作，即查找二维数组与其一行之间的差异：

```py
In [15]: A = rng.integers(10, size=(3, 4))
         A
Out[15]: array([[4, 4, 2, 0],
                [5, 8, 0, 8],
                [8, 2, 6, 1]])
```

```py
In [16]: A - A[0]
Out[16]: array([[ 0,  0,  0,  0],
                [ 1,  4, -2,  8],
                [ 4, -2,  4,  1]])
```

根据 NumPy 的广播规则（参见第八章），二维数组与其一行之间的减法操作是逐行应用的。

在 Pandas 中，默认情况下也是逐行操作的约定：

```py
In [17]: df = pd.DataFrame(A, columns=['Q', 'R', 'S', 'T'])
         df - df.iloc[0]
Out[17]:    Q  R  S  T
         0  0  0  0  0
         1  1  4 -2  8
         2  4 -2  4  1
```

如果您希望以列为单位进行操作，可以使用前面提到的对象方法，并指定`axis`关键字：

```py
In [18]: df.subtract(df['R'], axis=0)
Out[18]:    Q  R  S  T
         0  0  0 -2 -4
         1 -3  0 -8  0
         2  6  0  4 -1
```

注意，像前面讨论过的操作一样，这些`DataFrame`/`Series`操作会自动对齐两个元素之间的索引：

```py
In [19]: halfrow = df.iloc[0, ::2]
         halfrow
Out[19]: Q    4
         S    2
         Name: 0, dtype: int64
```

```py
In [20]: df - halfrow
Out[20]:      Q   R    S   T
         0  0.0 NaN  0.0 NaN
         1  1.0 NaN -2.0 NaN
         2  4.0 NaN  4.0 NaN
```

这种索引和列的保留与对齐意味着在 Pandas 中对数据进行的操作将始终保持数据上下文，这可以防止在原始 NumPy 数组中处理异构和/或不对齐数据时可能出现的常见错误。
