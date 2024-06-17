# 第十三章：介绍 Pandas 对象

在非常基本的层面上，Pandas 对象可以被认为是 NumPy 结构化数组的增强版本，其中的行和列被标签而不是简单的整数索引所标识。正如我们将在本章课程中看到的那样，Pandas 在基本数据结构之上提供了一系列有用的工具、方法和功能，但几乎所有接下来的内容都需要理解这些结构。因此，在我们继续之前，让我们看看这三种基本的 Pandas 数据结构：`Series`、`DataFrame` 和 `Index`。

我们将从标准的 NumPy 和 Pandas 导入开始我们的代码会话：

```py
In [1]: import numpy as np
        import pandas as pd
```

# Pandas Series 对象

Pandas `Series` 是一个索引数据的一维数组。它可以从列表或数组创建如下：

```py
In [2]: data = pd.Series([0.25, 0.5, 0.75, 1.0])
        data
Out[2]: 0    0.25
        1    0.50
        2    0.75
        3    1.00
        dtype: float64
```

`Series` 将一系列值与显式的索引序列结合在一起，我们可以使用 `values` 和 `index` 属性来访问。`values` 简单地是一个熟悉的 NumPy 数组：

```py
In [3]: data.values
Out[3]: array([0.25, 0.5 , 0.75, 1.  ])
```

`index` 是一个 `pd.Index` 类型的类似数组的对象，我们稍后会更详细地讨论：

```py
In [4]: data.index
Out[4]: RangeIndex(start=0, stop=4, step=1)
```

与 NumPy 数组类似，数据可以通过相关的索引使用熟悉的 Python 方括号符号进行访问：

```py
In [5]: data[1]
Out[5]: 0.5
```

```py
In [6]: data[1:3]
Out[6]: 1    0.50
        2    0.75
        dtype: float64
```

正如我们将看到的，Pandas `Series` 比它模拟的一维 NumPy 数组要更加一般化和灵活。

## 作为广义的 NumPy 数组的 Series

从目前我们所见，`Series` 对象可能看起来基本上可以与一维 NumPy 数组互换。本质上的区别在于，虽然 NumPy 数组具有用于访问值的*隐式定义*整数索引，但 Pandas `Series` 具有与值关联的*显式定义*索引。

这种显式索引定义赋予了 `Series` 对象额外的能力。例如，索引不必是整数，而可以是任何所需类型的值。所以，如果我们希望，我们可以使用字符串作为索引：

```py
In [7]: data = pd.Series([0.25, 0.5, 0.75, 1.0],
                         index=['a', 'b', 'c', 'd'])
        data
Out[7]: a    0.25
        b    0.50
        c    0.75
        d    1.00
        dtype: float64
```

并且项目访问按预期工作：

```py
In [8]: data['b']
Out[8]: 0.5
```

我们甚至可以使用非连续或非顺序的索引：

```py
In [9]: data = pd.Series([0.25, 0.5, 0.75, 1.0],
                         index=[2, 5, 3, 7])
        data
Out[9]: 2    0.25
        5    0.50
        3    0.75
        7    1.00
        dtype: float64
```

```py
In [10]: data[5]
Out[10]: 0.5
```

## 作为专用字典的 Series

以这种方式，你可以把 Pandas 的 `Series` 想象成 Python 字典的一个特殊版本。字典是一个将任意键映射到一组任意值的结构，而 `Series` 是一个将类型化键映射到一组类型化值的结构。这种类型化很重要：正如 NumPy 数组背后的特定类型的编译代码使其在某些操作上比 Python 列表更高效一样，Pandas `Series` 的类型信息使其在某些操作上比 Python 字典更高效。

`Series`-作为字典的类比可以通过直接从 Python 字典构造 `Series` 对象来更清晰地解释，例如根据 2020 年人口普查得到的五个最多人口的美国州：

```py
In [11]: population_dict = {'California': 39538223, 'Texas': 29145505,
                            'Florida': 21538187, 'New York': 20201249,
                            'Pennsylvania': 13002700}
         population = pd.Series(population_dict)
         population
Out[11]: California      39538223
         Texas           29145505
         Florida         21538187
         New York        20201249
         Pennsylvania    13002700
         dtype: int64
```

从这里，可以执行典型的字典式项目访问：

```py
In [12]: population['California']
Out[12]: 39538223
```

不同于字典，`Series` 也支持数组式的操作，比如切片：

```py
In [13]: population['California':'Florida']
Out[13]: California    39538223
         Texas         29145505
         Florida       21538187
         dtype: int64
```

我们将在第十四章讨论 Pandas 索引和切片的一些怪癖。

## 构建 Series 对象

我们已经看到了几种从头开始构建 Pandas `Series`的方法。所有这些方法都是以下版本的某种形式：

```py
pd.Series(data, index=index)
```

其中`index`是一个可选参数，`data`可以是多个实体之一。

例如，`data`可以是一个列表或 NumPy 数组，此时`index`默认为整数序列：

```py
In [14]: pd.Series([2, 4, 6])
Out[14]: 0    2
         1    4
         2    6
         dtype: int64
```

或者`data`可以是一个标量，它被重复以填充指定的索引：

```py
In [15]: pd.Series(5, index=[100, 200, 300])
Out[15]: 100    5
         200    5
         300    5
         dtype: int64
```

或者它可以是一个字典，此时`index`默认为字典的键：

```py
In [16]: pd.Series({2:'a', 1:'b', 3:'c'})
Out[16]: 2    a
         1    b
         3    c
         dtype: object
```

在每种情况下，都可以显式设置索引以控制使用的键的顺序或子集：

```py
In [17]: pd.Series({2:'a', 1:'b', 3:'c'}, index=[1, 2])
Out[17]: 1    b
         2    a
         dtype: object
```

# Pandas DataFrame 对象

Pandas 中的下一个基本结构是`DataFrame`。与前一节讨论的`Series`对象类似，`DataFrame`可以被视为 NumPy 数组的泛化，或者 Python 字典的特殊化。我们现在将看看每种观点。

## DataFrame 作为广义的 NumPy 数组

如果`Series`是具有显式索引的一维数组的类比，那么`DataFrame`就是具有显式行和列索引的二维数组的类比。正如你可能把二维数组看作一系列对齐的一维列的有序序列一样，你可以把`DataFrame`看作一系列对齐的`Series`对象。这里，“对齐”意味着它们共享相同的索引。

为了演示这一点，让我们首先构建一个新的`Series`，列出前一节讨论的五个州的面积（以平方公里为单位）：

```py
In [18]: area_dict = {'California': 423967, 'Texas': 695662, 'Florida': 170312,
                      'New York': 141297, 'Pennsylvania': 119280}
         area = pd.Series(area_dict)
         area
Out[18]: California      423967
         Texas           695662
         Florida         170312
         New York        141297
         Pennsylvania    119280
         dtype: int64
```

现在，我们已经有了与之前的`population`系列一起的信息，我们可以使用字典构造一个包含此信息的单个二维对象：

```py
In [19]: states = pd.DataFrame({'population': population,
                                'area': area})
         states
Out[19]:               population    area
         California      39538223  423967
         Texas           29145505  695662
         Florida         21538187  170312
         New York        20201249  141297
         Pennsylvania    13002700  119280
```

与`Series`对象类似，`DataFrame`还有一个`index`属性，用于访问索引标签：

```py
In [20]: states.index
Out[20]: Index(['California', 'Texas', 'Florida', 'New York', 'Pennsylvania'],
          > dtype='object')
```

此外，`DataFrame`还有一个`columns`属性，它是一个`Index`对象，保存列标签：

```py
In [21]: states.columns
Out[21]: Index(['population', 'area'], dtype='object')
```

因此，`DataFrame`可以被视为二维 NumPy 数组的泛化，其中行和列都有用于访问数据的泛化索引。

## DataFrame 作为特殊的字典

同样，我们也可以把`DataFrame`视为字典的特殊情况。在字典将键映射到值的情况下，`DataFrame`将列名映射到包含列数据的`Series`。例如，请求`'area'`属性将返回包含我们之前看到的面积的`Series`对象：

```py
In [22]: states['area']
Out[22]: California      423967
         Texas           695662
         Florida         170312
         New York        141297
         Pennsylvania    119280
         Name: area, dtype: int64
```

注意这里可能的混淆点：在一个二维 NumPy 数组中，`data[0]` 将返回第一行。对于 `DataFrame`，`data['col0']` 将返回第一列。因此，最好将 `DataFrame` 视为广义的字典，而不是广义的数组，尽管两种视角都是有用的。我们将在 第十四章 探讨更灵活的 `DataFrame` 索引方式。

## 构造 DataFrame 对象

Pandas `DataFrame` 可以以多种方式构建。这里我们将探讨几个例子。

### 从单个 Series 对象

`DataFrame` 是 `Series` 对象的集合，一个单列的 `DataFrame` 可以从一个单独的 `Series` 构建出来：

```py
In [23]: pd.DataFrame(population, columns=['population'])
Out[23]:              population
           California   39538223
                Texas   29145505
              Florida   21538187
             New York   20201249
         Pennsylvania   13002700
```

### 从字典列表

任何字典列表都可以转换成 `DataFrame`。我们将使用一个简单的列表推导来创建一些数据：

```py
In [24]: data = [{'a': i, 'b': 2 * i}
                 for i in range(3)]
         pd.DataFrame(data)
Out[24]:    a  b
         0  0  0
         1  1  2
         2  2  4
```

即使字典中有些键是缺失的，Pandas 也会用 `NaN` 值（即“Not a Number”；参见 第十六章）来填充它们：

```py
In [25]: pd.DataFrame([{'a': 1, 'b': 2}, {'b': 3, 'c': 4}])
Out[25]:      a  b    c
         0  1.0  2  NaN
         1  NaN  3  4.0
```

### 从字典的 Series 对象

正如我们之前所看到的，一个 `DataFrame` 也可以从一个字典的 `Series` 对象构建出来：

```py
In [26]: pd.DataFrame({'population': population,
                       'area': area})
Out[26]:               population    area
         California      39538223  423967
         Texas           29145505  695662
         Florida         21538187  170312
         New York        20201249  141297
         Pennsylvania    13002700  119280
```

### 从二维 NumPy 数组

给定一个二维数据数组，我们可以创建一个带有指定列和索引名称的 `DataFrame`。如果省略，将使用整数索引：

```py
In [27]: pd.DataFrame(np.random.rand(3, 2),
                      columns=['foo', 'bar'],
                      index=['a', 'b', 'c'])
Out[27]:         foo       bar
         a  0.471098  0.317396
         b  0.614766  0.305971
         c  0.533596  0.512377
```

### 从 NumPy 结构化数组

我们在 第十二章 中讨论了结构化数组。Pandas `DataFrame` 的操作方式与结构化数组非常相似，可以直接从结构化数组创建一个：

```py
In [28]: A = np.zeros(3, dtype=[('A', 'i8'), ('B', 'f8')])
         A
Out[28]: array([(0, 0.), (0, 0.), (0, 0.)], dtype=[('A', '<i8'), ('B', '<f8')])
```

```py
In [29]: pd.DataFrame(A)
Out[29]:    A    B
         0  0  0.0
         1  0  0.0
         2  0  0.0
```

# Pandas Index 对象

正如你所见，`Series` 和 `DataFrame` 对象都包含了一个明确的*索引*，让你可以引用和修改数据。这个 `Index` 对象本身就是一个有趣的结构，它可以被看作是一个*不可变数组*或者一个*有序集合*（技术上是一个多重集合，因为 `Index` 对象可能包含重复的值）。这些视角在 `Index` 对象上的操作上产生了一些有趣的后果。举个简单的例子，让我们从一个整数列表构造一个 `Index`：

```py
In [30]: ind = pd.Index([2, 3, 5, 7, 11])
         ind
Out[30]: Int64Index([2, 3, 5, 7, 11], dtype='int64')
```

## 作为不可变数组的 Index

`Index` 在许多方面都像一个数组。例如，我们可以使用标准的 Python 索引表示法来检索值或切片：

```py
In [31]: ind[1]
Out[31]: 3
```

```py
In [32]: ind[::2]
Out[32]: Int64Index([2, 5, 11], dtype='int64')
```

`Index` 对象也具有许多与 NumPy 数组相似的属性：

```py
In [33]: print(ind.size, ind.shape, ind.ndim, ind.dtype)
Out[33]: 5 (5,) 1 int64
```

`Index` 对象和 NumPy 数组之间的一个区别是索引是不可变的——也就是说，它们不能通过正常的方式修改：

```py
In [34]: ind[1] = 0
TypeError: Index does not support mutable operations
```

这种不可变性使得在多个 `DataFrame` 和数组之间共享索引更加安全，避免了因无意中修改索引而产生的副作用。

## 作为有序集合的 Index

Pandas 对象旨在简化诸如跨数据集连接等操作，这些操作依赖于集合算术的许多方面。`Index` 对象遵循 Python 内置 `set` 数据结构使用的许多约定，因此可以以熟悉的方式计算并集、交集、差集和其他组合：

```py
In [35]: indA = pd.Index([1, 3, 5, 7, 9])
         indB = pd.Index([2, 3, 5, 7, 11])
```

```py
In [36]: indA.intersection(indB)
Out[36]: Int64Index([3, 5, 7], dtype='int64')
```

```py
In [37]: indA.union(indB)
Out[37]: Int64Index([1, 2, 3, 5, 7, 9, 11], dtype='int64')
```

```py
In [38]: indA.symmetric_difference(indB)
Out[38]: Int64Index([1, 2, 9, 11], dtype='int64')
```
