# 第十八章：组合数据集：concat 和 append

一些最有趣的数据研究来自于结合不同的数据源。这些操作可以涉及从两个不同数据集的非常简单的连接到更复杂的数据库风格的联接和合并，正确处理数据集之间的任何重叠。`Series`和`DataFrame`是专为这类操作而构建的，Pandas 包含使这种数据处理快速和简单的函数和方法。

在这里，我们将使用`pd.concat`函数查看`Series`和`DataFrame`的简单连接；稍后我们将深入探讨 Pandas 中实现的更复杂的内存合并和连接。

我们从标准导入开始：

```py
In [1]: import pandas as pd
        import numpy as np
```

为方便起见，我们将定义这个函数，它创建一个特定形式的`DataFrame`，在接下来的示例中将非常有用：

```py
In [2]: def make_df(cols, ind):
            """Quickly make a DataFrame"""
            data = {c: [str(c) + str(i) for i in ind]
                    for c in cols}
            return pd.DataFrame(data, ind)

        # example DataFrame
        make_df('ABC', range(3))
Out[2]:     A   B   C
        0  A0  B0  C0
        1  A1  B1  C1
        2  A2  B2  C2
```

另外，我们将创建一个快速的类，允许我们将多个`DataFrame`并排显示。该代码利用了特殊的`_repr_html_`方法，IPython/Jupyter 用它来实现其丰富的对象显示：

```py
In [3]: class display(object):
            """Display HTML representation of multiple objects"""
            template = """<div style="float: left; padding: 10px;">
 <p style='font-family:"Courier New", Courier, monospace'>{0}{1}
 """
            def __init__(self, *args):
                self.args = args

            def _repr_html_(self):
                return '\n'.join(self.template.format(a, eval(a)._repr_html_())
                                 for a in self.args)

            def __repr__(self):
                return '\n\n'.join(a + '\n' + repr(eval(a))
                                   for a in self.args)
```

随着我们在以下部分继续讨论，使用这个将会更加清晰。

# 回顾：NumPy 数组的连接

`Series`和`DataFrame`对象的连接行为与 NumPy 数组的连接类似，可以通过`np.concatenate`函数完成，如第五章中所讨论的那样。记住，您可以使用它将两个或多个数组的内容合并为单个数组：

```py
In [4]: x = [1, 2, 3]
        y = [4, 5, 6]
        z = [7, 8, 9]
        np.concatenate([x, y, z])
Out[4]: array([1, 2, 3, 4, 5, 6, 7, 8, 9])
```

第一个参数是要连接的数组的列表或元组。此外，在多维数组的情况下，它接受一个`axis`关键字，允许您指定沿其进行连接的轴：

```py
In [5]: x = [[1, 2],
             [3, 4]]
        np.concatenate([x, x], axis=1)
Out[5]: array([[1, 2, 1, 2],
               [3, 4, 3, 4]])
```

# 使用`pd.concat`进行简单连接

`pd.concat`函数提供了与`np.concatenate`类似的语法，但包含我们稍后将讨论的多个选项：

```py
# Signature in Pandas v1.3.5
pd.concat(objs, axis=0, join='outer', ignore_index=False, keys=None,
          levels=None, names=None, verify_integrity=False,
          sort=False, copy=True)
```

`pd.concat`可用于简单连接`Series`或`DataFrame`对象，就像`np.concatenate`可用于数组的简单连接一样：

```py
In [6]: ser1 = pd.Series(['A', 'B', 'C'], index=[1, 2, 3])
        ser2 = pd.Series(['D', 'E', 'F'], index=[4, 5, 6])
        pd.concat([ser1, ser2])
Out[6]: 1    A
        2    B
        3    C
        4    D
        5    E
        6    F
        dtype: object
```

它还可以用于连接更高维度的对象，如`DataFrame`：

```py
In [7]: df1 = make_df('AB', [1, 2])
        df2 = make_df('AB', [3, 4])
        display('df1', 'df2', 'pd.concat([df1, df2])')
Out[7]: df1           df2           pd.concat([df1, df2])
            A   B         A   B         A   B
        1  A1  B1     3  A3  B3     1  A1  B1
        2  A2  B2     4  A4  B4     2  A2  B2
                                    3  A3  B3
                                    4  A4  B4
```

其默认行为是在`DataFrame`内按行连接（即`axis=0`）。与`np.concatenate`类似，`pd.concat`允许指定沿其进行连接的轴。考虑以下示例：

```py
In [8]: df3 = make_df('AB', [0, 1])
        df4 = make_df('CD', [0, 1])
        display('df3', 'df4', "pd.concat([df3, df4], axis='columns')")
Out[8]: df3           df4           pd.concat([df3, df4], axis='columns')
            A   B         C   D         A   B   C   D
        0  A0  B0     0  C0  D0     0  A0  B0  C0  D0
        1  A1  B1     1  C1  D1     1  A1  B1  C1  D1
```

我们也可以等效地指定`axis=1`；这里我们使用了更直观的`axis='columns'`。

## 重复的索引

`np.concatenate`和`pd.concat`之间的一个重要区别是，Pandas 的连接*保留索引*，即使结果会有重复的索引！考虑以下简单示例：

```py
In [9]: x = make_df('AB', [0, 1])
        y = make_df('AB', [2, 3])
        y.index = x.index  # make indices match
        display('x', 'y', 'pd.concat([x, y])')
Out[9]: x             y             pd.concat([x, y])
            A   B         A   B         A   B
        0  A0  B0     0  A2  B2     0  A0  B0
        1  A1  B1     1  A3  B3     1  A1  B1
                                    0  A2  B2
                                    1  A3  B3
```

注意结果中的重复索引。虽然这在`DataFrame`中是有效的，但结果通常不理想。`pd.concat`提供了几种处理方法。

### 将重复的索引视为错误处理

如果你想简单地验证`pd.concat`的结果中的索引是否重叠，可以包含`verify_integrity`标志。将其设置为`True`，如果存在重复索引，连接将引发异常。以下是一个示例，为了清晰起见，我们将捕获并打印错误消息：

```py
In [10]: try:
             pd.concat([x, y], verify_integrity=True)
         except ValueError as e:
             print("ValueError:", e)
ValueError: Indexes have overlapping values: Int64Index([0, 1], dtype='int64')
```

### 忽略索引

有时索引本身并不重要，你更希望它被简单地忽略。可以使用`ignore_index`标志指定此选项。将其设置为`True`，连接将为结果的`DataFrame`创建一个新的整数索引：

```py
In [11]: display('x', 'y', 'pd.concat([x, y], ignore_index=True)')
Out[11]: x              y             pd.concat([x, y], ignore_index=True)
             A   B          A   B         A   B
         0  A0  B0      0  A2  B2     0  A0  B0
         1  A1  B1      1  A3  B3     1  A1  B1
                                      2  A2  B2
                                      3  A3  B3
```

### 添加 MultiIndex 键

另一个选项是使用`keys`选项指定数据源的标签；结果将是一个具有层次索引的系列，其中包含数据：

```py
In [12]: display('x', 'y', "pd.concat([x, y], keys=['x', 'y'])")
Out[12]: x              y             pd.concat([x, y], keys=['x', 'y'])
             A   B          A   B           A   B
         0  A0  B0      0  A2  B2     x 0  A0  B0
         1  A1  B1      1  A3  B3       1  A1  B1
                                      y 0  A2  B2
                                        1  A3  B3
```

我们可以使用第十七章中讨论的工具将这个多重索引的 `DataFrame` 转换为我们感兴趣的表示形式。

## 使用连接进行连接

在我们刚刚查看的短示例中，我们主要是连接具有共享列名的 `DataFrame`。在实践中，来自不同来源的数据可能具有不同的列名集，`pd.concat` 在这种情况下提供了几个选项。考虑以下两个 `DataFrame` 的连接，它们具有一些（但不是全部！）共同的列：

```py
In [13]: df5 = make_df('ABC', [1, 2])
         df6 = make_df('BCD', [3, 4])
         display('df5', 'df6', 'pd.concat([df5, df6])')
Out[13]: df5                df6                pd.concat([df5, df6])
             A   B   C          B   C   D         A   B   C    D
         1  A1  B1  C1      3  B3  C3  D3      1   A1  B1  C1  NaN
         2  A2  B2  C2      4  B4  C4  D4      2   A2  B2  C2  NaN
                                               3  NaN  B3  C3   D3
                                               4  NaN  B4  C4   D4
```

默认行为是用 NA 值填充无可用数据的条目。要更改这一点，可以调整`concat`函数的`join`参数。默认情况下，连接是输入列的并集（`join='outer'`），但我们可以使用`join='inner'`将其更改为列的交集：

```py
In [14]: display('df5', 'df6',
                 "pd.concat([df5, df6], join='inner')")
Out[14]: df5                df6
             A   B   C          B   C   D
         1  A1  B1  C1      3  B3  C3  D3
         2  A2  B2  C2      4  B4  C4  D4

         pd.concat([df5, df6], join='inner')
             B   C
         1  B1  C1
         2  B2  C2
         3  B3  C3
         4  B4  C4
```

另一个有用的模式是在连接之前使用`reindex`方法对要丢弃的列进行更精细的控制：

```py
In [15]: pd.concat([df5, df6.reindex(df5.columns, axis=1)])
Out[15]:      A   B   C
         1   A1  B1  C1
         2   A2  B2  C2
         3  NaN  B3  C3
         4  NaN  B4  C4
```

## `append` 方法

因为直接数组连接是如此常见，`Series` 和 `DataFrame` 对象具有一个`append`方法，可以用更少的按键完成相同的操作。例如，可以使用 `df1.append(df2)` 替代 `pd.concat([df1, df2])`：

```py
In [16]: display('df1', 'df2', 'df1.append(df2)')
Out[16]: df1            df2           df1.append(df2)
             A   B          A   B         A   B
         1  A1  B1      3  A3  B3     1  A1  B1
         2  A2  B2      4  A4  B4     2  A2  B2
                                      3  A3  B3
                                      4  A4  B4
```

请注意，与 Python 列表的 `append` 和 `extend` 方法不同，Pandas 中的 `append` 方法不会修改原始对象；相反，它会创建一个包含组合数据的新对象。它也不是一种非常有效的方法，因为它涉及到新索引的创建 *以及* 数据缓冲区。因此，如果你计划进行多个 `append` 操作，通常最好建立一个 `DataFrame` 对象的列表，并一次性将它们全部传递给 `concat` 函数。

在下一章中，我们将介绍一种更强大的方法来组合来自多个来源的数据：`pd.merge` 中实现的数据库风格的合并/连接。有关 `concat`、`append` 和相关功能的更多信息，请参阅[Pandas 文档中的“Merge, Join, Concatenate and Compare”](https://oreil.ly/cY16c)。
