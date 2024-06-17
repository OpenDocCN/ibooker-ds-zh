# 第十四章：数据索引和选择

在第二部分中，我们详细讨论了访问、设置和修改 NumPy 数组中的值的方法和工具。这些包括索引（例如`arr[2, 1]`）、切片（例如`arr[:, 1:5]`）、掩码（例如`arr[arr > 0]`）、花式索引（例如`arr[0, [1, 5]]`）以及它们的组合（例如`arr[:, [1, 5]]`）。在这里，我们将看一下类似的方法来访问和修改 Pandas `Series`和`DataFrame`对象中的值。如果你使用过 NumPy 模式，Pandas 中的相应模式会感觉非常熟悉，尽管有一些需要注意的怪癖。

我们将从一维`Series`对象的简单情况开始，然后转向更复杂的二维`DataFrame`对象。

# Series 中的数据选择

正如你在前一章中看到的，`Series`对象在许多方面都像一个一维 NumPy 数组，而在许多方面都像一个标准的 Python 字典。如果你记住这两个重叠的类比，将有助于你理解这些数组中的数据索引和选择模式。

## `Series`作为字典

像字典一样，`Series`对象提供了从一组键到一组值的映射：

```py
In [1]: import pandas as pd
        data = pd.Series([0.25, 0.5, 0.75, 1.0],
                         index=['a', 'b', 'c', 'd'])
        data
Out[1]: a    0.25
        b    0.50
        c    0.75
        d    1.00
        dtype: float64
```

```py
In [2]: data['b']
Out[2]: 0.5
```

我们还可以使用类似字典的 Python 表达式和方法来查看键/索引和值：

```py
In [3]: 'a' in data
Out[3]: True
```

```py
In [4]: data.keys()
Out[4]: Index(['a', 'b', 'c', 'd'], dtype='object')
```

```py
In [5]: list(data.items())
Out[5]: [('a', 0.25), ('b', 0.5), ('c', 0.75), ('d', 1.0)]
```

`Series`对象也可以用类似字典的语法进行修改。就像你可以通过分配给新键来扩展字典一样，你可以通过分配给新索引值来扩展`Series`：

```py
In [6]: data['e'] = 1.25
        data
Out[6]: a    0.25
        b    0.50
        c    0.75
        d    1.00
        e    1.25
        dtype: float64
```

这种对象的易变性是一个方便的特性：在幕后，Pandas 正在做出关于内存布局和数据复制的决策，这可能需要进行，而用户通常不需要担心这些问题。

## 一维数组中的 Series

`Series`建立在这种类似字典的接口上，并通过与 NumPy 数组相同的基本机制提供了数组样式的项目选择——即切片、掩码和花式索引。以下是这些的示例：

```py
In [7]: # slicing by explicit index
        data['a':'c']
Out[7]: a    0.25
        b    0.50
        c    0.75
        dtype: float64
```

```py
In [8]: # slicing by implicit integer index
        data[0:2]
Out[8]: a    0.25
        b    0.50
        dtype: float64
```

```py
In [9]: # masking
        data[(data > 0.3) & (data < 0.8)]
Out[9]: b    0.50
        c    0.75
        dtype: float64
```

```py
In [10]: # fancy indexing
         data[['a', 'e']]
Out[10]: a    0.25
         e    1.25
         dtype: float64
```

其中，切片可能是最容易混淆的来源。请注意，当使用显式索引进行切片（例如`data['a':'c']`）时，最终索引被*包括*在切片中，而当使用隐式索引进行切片（例如`data[0:2]`）时，最终索引被*排除*在切片之外。

## 索引器：loc 和 iloc

如果你的`Series`有一个明确的整数索引，那么像`data[1]`这样的索引操作将使用明确的索引，而像`data[1:3]`这样的切片操作将使用隐式的 Python 风格索引：

```py
In [11]: data = pd.Series(['a', 'b', 'c'], index=[1, 3, 5])
         data
Out[11]: 1    a
         3    b
         5    c
         dtype: object
```

```py
In [12]: # explicit index when indexing
         data[1]
Out[12]: 'a'
```

```py
In [13]: # implicit index when slicing
         data[1:3]
Out[13]: 3    b
         5    c
         dtype: object
```

由于整数索引可能会导致混淆，Pandas 提供了一些特殊的*索引器*属性，明确地暴露了某些索引方案。这些不是功能性方法，而是属性，它们向`Series`中的数据公开了特定的切片接口。

首先，`loc`属性允许始终引用显式索引的索引和切片：

```py
In [14]: data.loc[1]
Out[14]: 'a'
```

```py
In [15]: data.loc[1:3]
Out[15]: 1    a
         3    b
         dtype: object
```

`iloc`属性允许引用始终参考隐式 Python 样式索引的索引和切片：

```py
In [16]: data.iloc[1]
Out[16]: 'b'
```

```py
In [17]: data.iloc[1:3]
Out[17]: 3    b
         5    c
         dtype: object
```

Python 代码的一个指导原则是“明确优于隐式”。`loc`和`iloc`的显式特性使它们在保持代码清晰和可读性方面非常有帮助；特别是在整数索引的情况下，始终一致地使用它们可以防止由于混合索引/切片约定而导致的微妙错误。

# 数据框选择

回想一下，`DataFrame`在许多方面都像一个二维或结构化数组，而在其他方面则像一个共享相同索引的`Series`结构的字典。当我们探索在这种结构内进行数据选择时，这些类比可能会有所帮助。

## DataFrame 作为字典

我们首先考虑的类比是将`DataFrame`视为一组相关`Series`对象的字典。让我们回到我们州的面积和人口的例子：

```py
In [18]: area = pd.Series({'California': 423967, 'Texas': 695662,
                           'Florida': 170312, 'New York': 141297,
                           'Pennsylvania': 119280})
         pop = pd.Series({'California': 39538223, 'Texas': 29145505,
                          'Florida': 21538187, 'New York': 20201249,
                          'Pennsylvania': 13002700})
         data = pd.DataFrame({'area':area, 'pop':pop})
         data
Out[18]:                 area       pop
         California    423967  39538223
         Texas         695662  29145505
         Florida       170312  21538187
         New York      141297  20201249
         Pennsylvania  119280  13002700
```

组成`DataFrame`列的单个`Series`可以通过列名的字典样式索引进行访问：

```py
In [19]: data['area']
Out[19]: California      423967
         Texas           695662
         Florida         170312
         New York        141297
         Pennsylvania    119280
         Name: area, dtype: int64
```

类似地，我们可以使用列名为字符串的属性样式访问：

```py
In [20]: data.area
Out[20]: California      423967
         Texas           695662
         Florida         170312
         New York        141297
         Pennsylvania    119280
         Name: area, dtype: int64
```

尽管这是一个有用的简写，但请记住，并非所有情况下都适用！例如，如果列名不是字符串，或者列名与`DataFrame`的方法冲突，这种属性样式访问就不可能。例如，`DataFrame`有一个`pop`方法，所以`data.pop`将指向这个方法而不是`pop`列：

```py
In [21]: data.pop is data["pop"]
Out[21]: False
```

特别地，你应该避免尝试通过属性进行列赋值（即，使用`data['pop'] = z`而不是`data.pop = z`）。

像之前讨论过的`Series`对象一样，这种字典样式的语法也可以用来修改对象，比如在这种情况下添加一个新列：

```py
In [22]: data['density'] = data['pop'] / data['area']
         data
Out[22]:                 area       pop     density
         California    423967  39538223   93.257784
         Texas         695662  29145505   41.896072
         Florida       170312  21538187  126.463121
         New York      141297  20201249  142.970120
         Pennsylvania  119280  13002700  109.009893
```

这展示了`Series`对象之间按元素进行算术运算的简单语法预览；我们将在第十五章进一步深入探讨这个问题。

## DataFrame 作为二维数组

正如前面提到的，我们也可以将`DataFrame`视为增强的二维数组。我们可以使用`values`属性查看原始的底层数据数组：

```py
In [23]: data.values
Out[23]: array([[4.23967000e+05, 3.95382230e+07, 9.32577842e+01],
                [6.95662000e+05, 2.91455050e+07, 4.18960717e+01],
                [1.70312000e+05, 2.15381870e+07, 1.26463121e+02],
                [1.41297000e+05, 2.02012490e+07, 1.42970120e+02],
                [1.19280000e+05, 1.30027000e+07, 1.09009893e+02]])
```

在这个画面中，许多熟悉的类似数组的操作可以在`DataFrame`本身上完成。例如，我们可以转置整个`DataFrame`来交换行和列：

```py
In [24]: data.T
Out[24]:            California         Texas       Florida      New York  Pennsylvania
         area     4.239670e+05  6.956620e+05  1.703120e+05  1.412970e+05  1.192800e+05
         pop      3.953822e+07  2.914550e+07  2.153819e+07  2.020125e+07  1.300270e+07
         density  9.325778e+01  4.189607e+01  1.264631e+02  1.429701e+02  1.090099e+02
```

然而，当涉及到`DataFrame`对象的索引时，很明显，列的字典样式索引排除了我们简单将其视为 NumPy 数组的能力。特别是，将单个索引传递给数组会访问一行：

```py
In [25]: data.values[0]
Out[25]: array([4.23967000e+05, 3.95382230e+07, 9.32577842e+01])
```

并且将一个单独的“索引”传递给`DataFrame`会访问一列：

```py
In [26]: data['area']
Out[26]: California      423967
         Texas           695662
         Florida         170312
         New York        141297
         Pennsylvania    119280
         Name: area, dtype: int64
```

因此，对于数组样式的索引，我们需要另一种约定。在这里，Pandas 再次使用了前面提到的`loc`和`iloc`索引器。使用`iloc`索引器，我们可以像使用简单的 NumPy 数组一样索引底层数组（使用隐式的 Python 风格索引），但结果中保持了`DataFrame`的索引和列标签：

```py
In [27]: data.iloc[:3, :2]
Out[27]:               area       pop
         California  423967  39538223
         Texas       695662  29145505
         Florida     170312  21538187
```

同样地，使用`loc`索引器，我们可以以类似于数组的样式索引底层数据，但使用显式的索引和列名：

```py
In [28]: data.loc[:'Florida', :'pop']
Out[28]:               area       pop
         California  423967  39538223
         Texas       695662  29145505
         Florida     170312  21538187
```

在这些索引器中，可以使用任何熟悉的类似于 NumPy 的数据访问模式。例如，在`loc`索引器中，我们可以按以下方式组合遮罩和花式索引：

```py
In [29]: data.loc[data.density > 120, ['pop', 'density']]
Out[29]:                pop     density
         Florida   21538187  126.463121
         New York  20201249  142.970120
```

任何这些索引约定也可以用于设置或修改值；这是通过与您在使用 NumPy 工作时习惯的标准方式完成的：

```py
In [30]: data.iloc[0, 2] = 90
         data
Out[30]:                 area       pop     density
         California    423967  39538223   90.000000
         Texas         695662  29145505   41.896072
         Florida       170312  21538187  126.463121
         New York      141297  20201249  142.970120
         Pennsylvania  119280  13002700  109.009893
```

要提升您在 Pandas 数据操作中的熟练程度，我建议您花一些时间使用一个简单的`DataFrame`，并探索这些不同索引方法允许的索引、切片、遮罩和花式索引类型。

## 额外的索引约定

还有一些额外的索引约定，可能与前面的讨论看似不符，但在实践中仍然很有用。首先，*索引*指的是列，而*切片*指的是行：

```py
In [31]: data['Florida':'New York']
Out[31]:             area       pop     density
         Florida   170312  21538187  126.463121
         New York  141297  20201249  142.970120
```

这种切片也可以通过数字而不是索引来引用行。

```py
In [32]: data[1:3]
Out[32]:            area       pop     density
         Texas    695662  29145505   41.896072
         Florida  170312  21538187  126.463121
```

类似地，直接的遮罩操作是按行而不是按列进行解释。

```py
In [33]: data[data.density > 120]
Out[33]:             area       pop     density
         Florida   170312  21538187  126.463121
         New York  141297  20201249  142.970120
```

这两种约定在语法上与 NumPy 数组上的约定类似，虽然它们可能不完全符合 Pandas 的约定模式，但由于它们的实际实用性，它们被包含了进来。
