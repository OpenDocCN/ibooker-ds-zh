# 第五章：NumPy 数组的基础知识

在 Python 中对数据进行操作几乎等同于对 NumPy 数组进行操作：即使是像 Pandas（第 III 部分）这样的较新工具也是围绕 NumPy 数组构建的。本章将介绍使用 NumPy 数组操作来访问数据和子数组、拆分、重塑和连接数组的几个示例。虽然这里展示的操作类型可能看起来有些枯燥和迂腐，但它们构成了本书中许多其他示例的基础。好好地了解它们！

这里将涵盖一些基本数组操作的类别：

*数组的属性*

确定数组的大小、形状、内存消耗和数据类型

*数组的索引*

获取和设置单个数组元素的值

*数组的切片*

获取和设置大数组中的较小子数组

*数组的重塑*

更改给定数组的形状

*数组的连接和拆分*

将多个数组组合成一个数组，并将一个数组拆分成多个数组

# NumPy 数组属性

首先让我们讨论一些有用的数组属性。我们将从定义一维、二维和三维随机数组开始。我们将使用 NumPy 的随机数生成器，我们将使用一个固定值来*seed*，以确保每次运行此代码时都生成相同的随机数组：

```py
In [1]: import numpy as np
        rng = np.random.default_rng(seed=1701)  # seed for reproducibility

        x1 = rng.integers(10, size=6)  # one-dimensional array
        x2 = rng.integers(10, size=(3, 4))  # two-dimensional array
        x3 = rng.integers(10, size=(3, 4, 5))  # three-dimensional array
```

每个数组都有一些属性，包括`ndim`（维数）、`shape`（每个维度的大小）、`size`（数组的总大小）和`dtype`（每个元素的类型）：

```py
In [2]: print("x3 ndim: ", x3.ndim)
        print("x3 shape:", x3.shape)
        print("x3 size: ", x3.size)
        print("dtype:   ", x3.dtype)
Out[2]: x3 ndim:  3
        x3 shape: (3, 4, 5)
        x3 size:  60
        dtype:    int64
```

有关数据类型的更多讨论，请参阅第四章。

# 数组索引：访问单个元素

如果你熟悉 Python 标准列表的索引，那么在 NumPy 中进行索引将会感觉非常熟悉。在一维数组中，第*i*个值（从零开始计数）可以通过在方括号中指定所需索引来访问，就像在 Python 列表中一样：

```py
In [3]: x1
Out[3]: array([9, 4, 0, 3, 8, 6])
```

```py
In [4]: x1[0]
Out[4]: 9
```

```py
In [5]: x1[4]
Out[5]: 8
```

要从数组的末尾进行索引，可以使用负索引：

```py
In [6]: x1[-1]
Out[6]: 6
```

```py
In [7]: x1[-2]
Out[7]: 8
```

在多维数组中，可以使用逗号分隔的`(*行*, *列*)`元组来访问项目：

```py
In [8]: x2
Out[8]: array([[3, 1, 3, 7],
               [4, 0, 2, 3],
               [0, 0, 6, 9]])
```

```py
In [9]: x2[0, 0]
Out[9]: 3
```

```py
In [10]: x2[2, 0]
Out[10]: 0
```

```py
In [11]: x2[2, -1]
Out[11]: 9
```

值也可以使用任何前述的索引表示进行修改：

```py
In [12]: x2[0, 0] = 12
         x2
Out[12]: array([[12,  1,  3,  7],
                [ 4,  0,  2,  3],
                [ 0,  0,  6,  9]])
```

请记住，与 Python 列表不同，NumPy 数组具有固定的类型。这意味着，例如，如果你尝试将浮点值插入到整数数组中，该值将被静默截断。不要被这种行为所困扰！

```py
In [13]: x1[0] = 3.14159  # this will be truncated!
         x1
Out[13]: array([3, 4, 0, 3, 8, 6])
```

# 数组切片：访问子数组

正如我们可以使用方括号访问单个数组元素一样，我们也可以使用*切片*符号（由冒号（`:`）字符标记）来访问子数组。NumPy 的切片语法遵循标准 Python 列表的语法；要访问数组`x`的一个切片，使用以下方法：

```py
x[start:stop:step]
```

如果其中任何一个未指定，它们将默认为`start=0`、`stop=<size of dimension>`、`step=1`的值。让我们看一些在一维和多维数组中访问子数组的例子。

## 一维子数组

这里是一些访问一维子数组中元素的示例：

```py
In [14]: x1
Out[14]: array([3, 4, 0, 3, 8, 6])
```

```py
In [15]: x1[:3]  # first three elements
Out[15]: array([3, 4, 0])
```

```py
In [16]: x1[3:]  # elements after index 3
Out[16]: array([3, 8, 6])
```

```py
In [17]: x1[1:4]  # middle subarray
Out[17]: array([4, 0, 3])
```

```py
In [18]: x1[::2]  # every second element
Out[18]: array([3, 0, 8])
```

```py
In [19]: x1[1::2]  # every second element, starting at index 1
Out[19]: array([4, 3, 6])
```

一个可能令人困惑的情况是当`step`值为负时。在这种情况下，`start`和`stop`的默认值会被交换。这成为反转数组的便捷方式：

```py
In [20]: x1[::-1]  # all elements, reversed
Out[20]: array([6, 8, 3, 0, 4, 3])
```

```py
In [21]: x1[4::-2]  # every second element from index 4, reversed
Out[21]: array([8, 0, 3])
```

## 多维子数组

多维切片的工作方式相同，使用逗号分隔的多个切片。例如：

```py
In [22]: x2
Out[22]: array([[12,  1,  3,  7],
                [ 4,  0,  2,  3],
                [ 0,  0,  6,  9]])
```

```py
In [23]: x2[:2, :3]  # first two rows & three columns
Out[23]: array([[12,  1,  3],
                [ 4,  0,  2]])
```

```py
In [24]: x2[:3, ::2]  # three rows, every second column
Out[24]: array([[12,  3],
                [ 4,  2],
                [ 0,  6]])
```

```py
In [25]: x2[::-1, ::-1]  # all rows & columns, reversed
Out[25]: array([[ 9,  6,  0,  0],
                [ 3,  2,  0,  4],
                [ 7,  3,  1, 12]])
```

一个常见的例程是访问数组的单行或单列。这可以通过组合索引和切片来完成，使用一个冒号（`:`）标记的空切片：

```py
In [26]: x2[:, 0]  # first column of x2
Out[26]: array([12,  4,  0])
```

```py
In [27]: x2[0, :]  # first row of x2
Out[27]: array([12,  1,  3,  7])
```

在行访问的情况下，可以省略空切片以获得更紧凑的语法：

```py
In [28]: x2[0]  # equivalent to x2[0, :]
Out[28]: array([12,  1,  3,  7])
```

## 子数组作为非副本视图

与 Python 列表切片不同，NumPy 数组切片返回的是数组数据的*视图*而不是*副本*。考虑我们之前的二维数组：

```py
In [29]: print(x2)
Out[29]: [[12  1  3  7]
          [ 4  0  2  3]
          [ 0  0  6  9]]
```

让我们从中提取一个<math alttext="2 times 2"><mrow><mn>2</mn> <mo>×</mo> <mn>2</mn></mrow></math>子数组：

```py
In [30]: x2_sub = x2[:2, :2]
         print(x2_sub)
Out[30]: [[12  1]
          [ 4  0]]
```

现在，如果我们修改这个子数组，我们会看到原始数组也发生了改变！观察：

```py
In [31]: x2_sub[0, 0] = 99
         print(x2_sub)
Out[31]: [[99  1]
          [ 4  0]]
```

```py
In [32]: print(x2)
Out[32]: [[99  1  3  7]
          [ 4  0  2  3]
          [ 0  0  6  9]]
```

一些用户可能会觉得这很奇怪，但这可能是有利的：例如，在处理大型数据集时，我们可以访问和处理这些数据集的部分而无需复制底层数据缓冲区。

## 创建数组的副本

尽管数组视图具有这些特性，但有时将数据明确复制到数组或子数组中更有用。这可以使用`copy`方法最容易地完成：

```py
In [33]: x2_sub_copy = x2[:2, :2].copy()
         print(x2_sub_copy)
Out[33]: [[99  1]
          [ 4  0]]
```

如果我们现在修改这个子数组，原始数组不会受到影响：

```py
In [34]: x2_sub_copy[0, 0] = 42
         print(x2_sub_copy)
Out[34]: [[42  1]
          [ 4  0]]
```

```py
In [35]: print(x2)
Out[35]: [[99  1  3  7]
          [ 4  0  2  3]
          [ 0  0  6  9]]
```

# 数组的重塑

另一种有用的操作类型是数组的重塑，可以使用`reshape`方法完成。例如，如果你想将数字 1 到 9 放在一个<math alttext="3 times 3"><mrow><mn>3</mn> <mo>×</mo> <mn>3</mn></mrow></math>网格中，你可以这样做：

```py
In [36]: grid = np.arange(1, 10).reshape(3, 3)
         print(grid)
Out[36]: [[1 2 3]
          [4 5 6]
          [7 8 9]]
```

请注意，为了使其工作，初始数组的大小必须与重塑后数组的大小匹配，在大多数情况下，`reshape`方法将返回初始数组的非副本视图。

一个常见的重塑操作是将一维数组转换为二维行或列矩阵：

```py
In [37]: x = np.array([1, 2, 3])
         x.reshape((1, 3))  # row vector via reshape
Out[37]: array([[1, 2, 3]])
```

```py
In [38]: x.reshape((3, 1))  # column vector via reshape
Out[38]: array([[1],
                [2],
                [3]])
```

这样做的一个便利的简写是在切片语法中使用`np.newaxis`：

```py
In [39]: x[np.newaxis, :]  # row vector via newaxis
Out[39]: array([[1, 2, 3]])
```

```py
In [40]: x[:, np.newaxis]  # column vector via newaxis
Out[40]: array([[1],
                [2],
                [3]])
```

这是本书剩余部分我们经常会利用的模式。

# 数组连接和拆分

所有前述的例程都在单个数组上工作。NumPy 还提供了将多个数组合并成一个数组的工具，以及将单个数组拆分成多个数组的工具。

## 数组的连接

在 NumPy 中，数组的连接或组合主要通过`np.concatenate`、`np.vstack`和`np.hstack`这些例程来实现。`np.concatenate`将一个数组的元组或列表作为其第一个参数，如下所示：

```py
In [41]: x = np.array([1, 2, 3])
         y = np.array([3, 2, 1])
         np.concatenate([x, y])
Out[41]: array([1, 2, 3, 3, 2, 1])
```

你也可以一次连接多个数组：

```py
In [42]: z = np.array([99, 99, 99])
         print(np.concatenate([x, y, z]))
Out[42]: [ 1  2  3  3  2  1 99 99 99]
```

它也可以用于二维数组：

```py
In [43]: grid = np.array([[1, 2, 3],
                          [4, 5, 6]])
```

```py
In [44]: # concatenate along the first axis
         np.concatenate([grid, grid])
Out[44]: array([[1, 2, 3],
                [4, 5, 6],
                [1, 2, 3],
                [4, 5, 6]])
```

```py
In [45]: # concatenate along the second axis (zero-indexed)
         np.concatenate([grid, grid], axis=1)
Out[45]: array([[1, 2, 3, 1, 2, 3],
                [4, 5, 6, 4, 5, 6]])
```

对于处理混合维度数组，使用 `np.vstack`（垂直堆叠）和 `np.hstack`（水平堆叠）函数可能更清晰：

```py
In [46]: # vertically stack the arrays
         np.vstack([x, grid])
Out[46]: array([[1, 2, 3],
                [1, 2, 3],
                [4, 5, 6]])
```

```py
In [47]: # horizontally stack the arrays
         y = np.array([[99],
                       [99]])
         np.hstack([grid, y])
Out[47]: array([[ 1,  2,  3, 99],
                [ 4,  5,  6, 99]])
```

类似地，对于高维数组，`np.dstack` 将沿第三轴堆叠数组。

## 数组的分割

连接的反操作是分割，由函数 `np.split`、`np.hsplit` 和 `np.vsplit` 实现。对于每一个，我们可以传递一个给定分割点的索引列表：

```py
In [48]: x = [1, 2, 3, 99, 99, 3, 2, 1]
         x1, x2, x3 = np.split(x, [3, 5])
         print(x1, x2, x3)
Out[48]: [1 2 3] [99 99] [3 2 1]
```

注意，*N* 个分割点会导致 *N* + 1 个子数组。相关的函数 `np.hsplit` 和 `np.vsplit` 类似：

```py
In [49]: grid = np.arange(16).reshape((4, 4))
         grid
Out[49]: array([[ 0,  1,  2,  3],
                [ 4,  5,  6,  7],
                [ 8,  9, 10, 11],
                [12, 13, 14, 15]])
```

```py
In [50]: upper, lower = np.vsplit(grid, [2])
         print(upper)
         print(lower)
Out[50]: [[0 1 2 3]
          [4 5 6 7]]
         [[ 8  9 10 11]
          [12 13 14 15]]
```

```py
In [51]: left, right = np.hsplit(grid, [2])
         print(left)
         print(right)
Out[51]: [[ 0  1]
          [ 4  5]
          [ 8  9]
          [12 13]]
         [[ 2  3]
          [ 6  7]
          [10 11]
          [14 15]]
```

类似地，对于高维数组，`np.dsplit` 将沿第三轴分割数组。
