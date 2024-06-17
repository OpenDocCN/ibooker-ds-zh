# 第十二章：结构化数据：NumPy 的结构化数组

虽然通常我们的数据可以用值的同类数组很好地表示，但有时情况并非如此。本章演示了 NumPy 的*结构化数组*和*记录数组*的使用，它们为复合异构数据提供了高效的存储。虽然这里展示的模式对于简单操作很有用，但像这样的情景通常适合使用 Pandas 的`DataFrame`，我们将在第三部分中探讨。

```py
In [1]: import numpy as np
```

假设我们有几类数据关于一些人（比如，姓名、年龄和体重），我们想要存储这些值以供在 Python 程序中使用。可以将它们分别存储在三个单独的数组中：

```py
In [2]: name = ['Alice', 'Bob', 'Cathy', 'Doug']
        age = [25, 45, 37, 19]
        weight = [55.0, 85.5, 68.0, 61.5]
```

但这有点笨拙。这里没有告诉我们这三个数组是相关的；NumPy 的结构化数组允许我们通过使用单一结构更自然地存储所有这些数据。

回顾之前我们用这样的表达式创建了一个简单的数组：

```py
In [3]: x = np.zeros(4, dtype=int)
```

我们可以类似地使用复合数据类型规范来创建结构化数组：

```py
In [4]: # Use a compound data type for structured arrays
        data = np.zeros(4, dtype={'names':('name', 'age', 'weight'),
                                  'formats':('U10', 'i4', 'f8')})
        print(data.dtype)
Out[4]: [('name', '<U10'), ('age', '<i4'), ('weight', '<f8')]
```

这里的`'U10'`被翻译为“最大长度为 10 的 Unicode 字符串”，`'i4'`被翻译为“4 字节（即 32 位）整数”，而`'f8'`被翻译为“8 字节（即 64 位）浮点数”。我们将在下一节讨论这些类型代码的其他选项。

现在我们创建了一个空的容器数组，我们可以用我们的值列表填充这个数组：

```py
In [5]: data['name'] = name
        data['age'] = age
        data['weight'] = weight
        print(data)
Out[5]: [('Alice', 25, 55. ) ('Bob', 45, 85.5) ('Cathy', 37, 68. )
         ('Doug', 19, 61.5)]
```

正如我们所希望的那样，数据现在方便地排列在一个结构化数组中。

结构化数组的方便之处在于，我们现在既可以按索引也可以按名称引用值：

```py
In [6]: # Get all names
        data['name']
Out[6]: array(['Alice', 'Bob', 'Cathy', 'Doug'], dtype='<U10')
```

```py
In [7]: # Get first row of data
        data[0]
Out[7]: ('Alice', 25, 55.)
```

```py
In [8]: # Get the name from the last row
        data[-1]['name']
Out[8]: 'Doug'
```

使用布尔遮罩，我们甚至可以进行一些更复杂的操作，例如按年龄进行过滤：

```py
In [9]: # Get names where age is under 30
        data[data['age'] < 30]['name']
Out[9]: array(['Alice', 'Doug'], dtype='<U10')
```

如果您想要进行比这些更复杂的操作，您可能应该考虑 Pandas 包，详细介绍请参阅第四部分。正如您将看到的，Pandas 提供了一个`DataFrame`对象，它是建立在 NumPy 数组上的结构，提供了许多有用的数据操作功能，类似于您在这里看到的，以及更多更多。

# 探索结构化数组的创建

结构化数组数据类型可以用多种方式指定。早些时候，我们看到了字典方法：

```py
In [10]: np.dtype({'names':('name', 'age', 'weight'),
                   'formats':('U10', 'i4', 'f8')})
Out[10]: dtype([('name', '<U10'), ('age', '<i4'), ('weight', '<f8')])
```

为了清晰起见，数值类型可以使用 Python 类型或 NumPy `dtype`来指定：

```py
In [11]: np.dtype({'names':('name', 'age', 'weight'),
                   'formats':((np.str_, 10), int, np.float32)})
Out[11]: dtype([('name', '<U10'), ('age', '<i8'), ('weight', '<f4')])
```

复合类型也可以作为元组列表来指定：

```py
In [12]: np.dtype([('name', 'S10'), ('age', 'i4'), ('weight', 'f8')])
Out[12]: dtype([('name', 'S10'), ('age', '<i4'), ('weight', '<f8')])
```

如果类型名称对您不重要，您可以单独在逗号分隔的字符串中指定这些类型：

```py
In [13]: np.dtype('S10,i4,f8')
Out[13]: dtype([('f0', 'S10'), ('f1', '<i4'), ('f2', '<f8')])
```

缩短的字符串格式代码可能并不直观，但它们基于简单的原则构建。第一个（可选）字符 `<` 或 `>`，表示“小端”或“大端”，分别指定了显著位的排序约定。接下来的字符指定数据类型：字符、字节、整数、浮点数等（参见表 12-1）。最后一个字符或多个字符表示对象的字节大小。

表 12-1\. NumPy 数据类型

| 字符 | 描述 | 示例 |
| --- | --- | --- |
| `'b'` | 字节 | `np.dtype('b')` |
| `'i'` | 有符号整数 | `np.dtype('i4') == np.int32` |
| `'u'` | 无符号整数 | `np.dtype('u1') == np.uint8` |
| `'f'` | 浮点数 | `np.dtype('f8') == np.int64` |
| `'c'` | 复数浮点数 | `np.dtype('c16') == np.complex128` |
| `'S'`, `'a'` | 字符串 | `np.dtype('S5')` |
| `'U'` | Unicode 字符串 | `np.dtype('U') == np.str_` |
| `'V'` | 原始数据（空） | `np.dtype('V') == np.void` |

# 更高级的复合类型

可以定义更高级的复合类型。例如，您可以创建每个元素包含值数组或矩阵的类型。在这里，我们将创建一个数据类型，其 `mat` 组件由一个 <math alttext="3 times 3"><mrow><mn>3</mn> <mo>×</mo> <mn>3</mn></mrow></math> 浮点矩阵组成：

```py
In [14]: tp = np.dtype([('id', 'i8'), ('mat', 'f8', (3, 3))])
         X = np.zeros(1, dtype=tp)
         print(X[0])
         print(X['mat'][0])
Out[14]: (0, [[0., 0., 0.], [0., 0., 0.], [0., 0., 0.]])
         [[0. 0. 0.]
          [0. 0. 0.]
          [0. 0. 0.]]
```

现在 `X` 数组中的每个元素包含一个 `id` 和一个 <math alttext="3 times 3"><mrow><mn>3</mn> <mo>×</mo> <mn>3</mn></mrow></math> 矩阵。为什么您会使用这个而不是简单的多维数组，或者可能是 Python 字典？一个原因是这个 NumPy `dtype` 直接映射到 C 结构定义，因此包含数组内容的缓冲区可以在适当编写的 C 程序中直接访问。如果您发现自己编写一个 Python 接口来操作结构化数据的传统 C 或 Fortran 库，结构化数组可以提供一个强大的接口。

# 记录数组：带有变化的结构化数组

NumPy 还提供了记录数组（`np.recarray` 类的实例），几乎与上述结构化数组相同，但有一个附加功能：可以将字段作为属性而不是字典键访问。回顾我们之前通过编写来访问样本数据集中的年龄：

```py
In [15]: data['age']
Out[15]: array([25, 45, 37, 19], dtype=int32)
```

如果我们将数据视为记录数组，我们可以用稍少的按键操作访问它：

```py
In [16]: data_rec = data.view(np.recarray)
         data_rec.age
Out[16]: array([25, 45, 37, 19], dtype=int32)
```

缺点在于，对于记录数组，即使使用相同的语法，访问字段时也涉及一些额外的开销：

```py
In [17]: %timeit data['age']
         %timeit data_rec['age']
         %timeit data_rec.age
Out[17]: 121 ns ± 1.4 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)
         2.41 µs ± 15.7 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)
         3.98 µs ± 20.5 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)
```

是否更便利的表示形式值得（轻微的）开销将取决于您自己的应用程序。

# 转向 Pandas

本章节关于结构化数组和记录数组被特意放置在本书的这一部分的末尾，因为它很好地过渡到我们将要介绍的下一个包：Pandas。结构化数组在某些情况下非常有用，比如当你使用 NumPy 数组来映射到 C、Fortran 或其他语言的二进制数据格式时。但对于日常使用结构化数据，Pandas 包是一个更好的选择；我们将在接下来的章节深入探讨它。
