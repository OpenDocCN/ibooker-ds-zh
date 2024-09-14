# 第九章：比较、掩码和布尔逻辑

本章介绍了使用布尔掩码来检查和操作 NumPy 数组中的值。当你想要基于某些条件提取、修改、计数或以其他方式操作数组中的值时，掩码就会出现：例如，你可能希望计算大于某个特定值的所有值，或者删除所有超过某个阈值的异常值。在 NumPy 中，布尔掩码通常是实现这些任务的最高效方式。

# 示例：统计下雨天数

想象一下，你有一系列数据，代表了一个城市一年中每天的降水量。例如，在这里我们将加载西雅图市 2015 年的每日降雨统计数据，使用 Pandas（参见第三部分）：

```py
In [1]: import numpy as np
        from vega_datasets import data

        # Use DataFrame operations to extract rainfall as a NumPy array
        rainfall_mm = np.array(
            data.seattle_weather().set_index('date')['precipitation']['2015'])
        len(rainfall_mm)
Out[1]: 365
```

数组包含 365 个值，从 2015 年 1 月 1 日到 12 月 31 日的每日降雨量（以毫米为单位）。

首先快速可视化，让我们来看一下图 9-1 中的下雨天数直方图，这是使用 Matplotlib 生成的（我们将在第四部分中详细探讨这个工具）：

```py
In [2]: %matplotlib inline
        import matplotlib.pyplot as plt
        plt.style.use('seaborn-whitegrid')
```

```py
In [3]: plt.hist(rainfall_mm, 40);
```

![output 6 0](img/output_6_0.png)

###### 图 9-1\. 西雅图 2015 年降雨直方图

这个直方图给了我们一个关于数据外观的概括性的想法：尽管西雅图以多雨而闻名，2015 年西雅图大部分日子都几乎没有测得的降水量。但这并没有很好地传达一些我们想要看到的信息：例如，整年有多少天下雨？在那些下雨的日子里，平均降水量是多少？有多少天降水量超过 10 毫米？

其中一种方法是通过手工来回答这些问题：我们可以遍历数据，每次看到在某个期望范围内的值时增加一个计数器。但是出于本章讨论的原因，从编写代码的时间和计算结果的时间来看，这种方法非常低效。我们在第六章中看到，NumPy 的通用函数（ufuncs）可以用来替代循环，在数组上进行快速的逐元素算术操作；同样地，我们可以使用其他通用函数在数组上进行逐元素的*比较*，然后可以操作结果来回答我们的问题。暂且把数据放在一边，讨论一下 NumPy 中一些常用工具，使用*掩码*来快速回答这类问题。

# 比较运算符作为通用函数

第六章介绍了 ufuncs，特别关注算术运算符。我们看到，在数组上使用 `+`, `-`, `*`, `/` 和其他运算符会导致逐元素操作。NumPy 还实现了比较运算符，如 `<`（小于）和 `>`（大于）作为逐元素的 ufuncs。这些比较运算符的结果始终是一个布尔数据类型的数组。标准的六种比较操作都是可用的：

```py
In [4]: x = np.array([1, 2, 3, 4, 5])
```

```py
In [5]: x < 3  # less than
Out[5]: array([ True,  True, False, False, False])
```

```py
In [6]: x > 3  # greater than
Out[6]: array([False, False, False,  True,  True])
```

```py
In [7]: x <= 3  # less than or equal
Out[7]: array([ True,  True,  True, False, False])
```

```py
In [8]: x >= 3  # greater than or equal
Out[8]: array([False, False,  True,  True,  True])
```

```py
In [9]: x != 3  # not equal
Out[9]: array([ True,  True, False,  True,  True])
```

```py
In [10]: x == 3  # equal
Out[10]: array([False, False,  True, False, False])
```

还可以对两个数组进行逐元素比较，并包括复合表达式：

```py
In [11]: (2 * x) == (x ** 2)
Out[11]: array([False,  True, False, False, False])
```

就像算术运算符的情况一样，NumPy 中的比较运算符也被实现为 ufuncs；例如，当你写 `x < 3` 时，NumPy 内部使用 `np.less(x, 3)`。这里显示了比较运算符及其等效的 ufuncs 的摘要：

| 操作符 | 等效的 ufunc | 操作符 | 等效的 ufunc |
| --- | --- | --- | --- |
| `==` | `np.equal` | `!=` | `np.not_equal` |
| `<` | `np.less` | `<=` | `np.less_equal` |
| `>` | `np.greater` | `>=` | `np.greater_equal` |

就像算术 ufuncs 的情况一样，这些函数适用于任何大小和形状的数组。这里是一个二维数组的例子：

```py
In [12]: rng = np.random.default_rng(seed=1701)
         x = rng.integers(10, size=(3, 4))
         x
Out[12]: array([[9, 4, 0, 3],
                [8, 6, 3, 1],
                [3, 7, 4, 0]])
```

```py
In [13]: x < 6
Out[13]: array([[False,  True,  True,  True],
                [False, False,  True,  True],
                [ True, False,  True,  True]])
```

在每种情况下，结果都是一个布尔数组，NumPy 提供了一些简单的模式来处理这些布尔结果。

# 使用布尔数组

给定一个布尔数组，你可以进行许多有用的操作。我们将使用我们之前创建的二维数组 `x`：

```py
In [14]: print(x)
Out[14]: [[9 4 0 3]
          [8 6 3 1]
          [3 7 4 0]]
```

## 计数条目

要计算布尔数组中 `True` 条目的数量，可以使用 `np.count_nonzero`：

```py
In [15]: # how many values less than 6?
         np.count_nonzero(x < 6)
Out[15]: 8
```

我们看到有八个数组条目小于 6。获取这些信息的另一种方法是使用 `np.sum`；在这种情况下，`False` 被解释为 `0`，`True` 被解释为 `1`：

```py
In [16]: np.sum(x < 6)
Out[16]: 8
```

`np.sum` 的好处在于，与其他 NumPy 聚合函数一样，这种求和可以沿着行或列进行：

```py
In [17]: # how many values less than 6 in each row?
         np.sum(x < 6, axis=1)
Out[17]: array([3, 2, 3])
```

这会统计矩阵每行中小于 6 的值的数量。

如果我们有兴趣快速检查任何或所有的值是否为 `True`，我们可以使用（你猜对了）`np.any` 或 `np.all`：

```py
In [18]: # are there any values greater than 8?
         np.any(x > 8)
Out[18]: True
```

```py
In [19]: # are there any values less than zero?
         np.any(x < 0)
Out[19]: False
```

```py
In [20]: # are all values less than 10?
         np.all(x < 10)
Out[20]: True
```

```py
In [21]: # are all values equal to 6?
         np.all(x == 6)
Out[21]: False
```

`np.all` 和 `np.any` 也可以沿着特定的轴使用。例如：

```py
In [22]: # are all values in each row less than 8?
         np.all(x < 8, axis=1)
Out[22]: array([False, False,  True])
```

这里第三行的所有元素都小于 8，而其他行则不是这样。

最后，一个快速的警告：如第七章中提到的，Python 中有内置的 `sum`、`any` 和 `all` 函数。它们与 NumPy 版本的语法不同，特别是在多维数组上使用时可能会失败或产生意外的结果。确保在这些示例中使用 `np.sum`、`np.any` 和 `np.all`！

## 布尔运算符

我们已经看到如何计算例如所有降雨量小于 20 毫米的天数，或所有降雨量大于 10 毫米的天数。但是如果我们想知道降雨量大于 10 毫米且小于 20 毫米的天数有多少呢？我们可以用 Python 的*位逻辑运算符* `&`, `|`, `^`, 和 `~` 来实现这一点。与标准算术运算符一样，NumPy 将它们重载为 ufuncs，这些 ufuncs 在（通常是布尔）数组上逐元素工作。

例如，我们可以这样处理这种复合问题：

```py
In [23]: np.sum((rainfall_mm > 10) & (rainfall_mm < 20))
Out[23]: 16
```

这告诉我们，有 16 天的降雨量在 10 到 20 毫米之间。

这里的括号很重要。由于操作符优先级规则，如果去掉括号，这个表达式将按照以下方式进行评估，从而导致错误：

```py
rainfall_mm > (10 & rainfall_mm) < 20
```

让我们演示一个更复杂的表达式。使用德摩根定律，我们可以以不同的方式计算相同的结果：

```py
In [24]: np.sum(~( (rainfall_mm <= 10) | (rainfall_mm >= 20) ))
Out[24]: 16
```

在数组上结合比较操作符和布尔操作符，可以进行广泛的高效逻辑操作。

以下表格总结了位运算布尔操作符及其等效的 ufuncs：

| 运算符 | 等效 ufunc | 运算符 | 等效 ufunc |
| --- | --- | --- | --- |
| `&` | `np.bitwise_and` |  | `np.bitwise_or` |
| `^` | `np.bitwise_xor` | `~` | `np.bitwise_not` |

使用这些工具，我们可以开始回答关于我们天气数据的许多问题。以下是将布尔操作与聚合结合时可以计算的一些结果示例：

```py
In [25]: print("Number days without rain:  ", np.sum(rainfall_mm == 0))
         print("Number days with rain:     ", np.sum(rainfall_mm != 0))
         print("Days with more than 10 mm: ", np.sum(rainfall_mm > 10))
         print("Rainy days with < 5 mm:    ", np.sum((rainfall_mm > 0) &
                                                     (rainfall_mm < 5)))
Out[25]: Number days without rain:   221
         Number days with rain:      144
         Days with more than 10 mm:  34
         Rainy days with < 5 mm:     83
```

# 布尔数组作为掩码

在前面的部分中，我们看过直接在布尔数组上计算的聚合。更强大的模式是使用布尔数组作为掩码，选择数据本身的特定子集。让我们回到之前的`x`数组：

```py
In [26]: x
Out[26]: array([[9, 4, 0, 3],
                [8, 6, 3, 1],
                [3, 7, 4, 0]])
```

假设我们想要一个数组，其中所有值都小于，比如说，5。我们可以轻松地为这个条件获取一个布尔数组，就像我们已经看到的那样：

```py
In [27]: x < 5
Out[27]: array([[False,  True,  True,  True],
                [False, False,  True,  True],
                [ True, False,  True,  True]])
```

现在，要从数组中*选择*这些值，我们可以简单地在这个布尔数组上进行索引；这称为*掩码*操作：

```py
In [28]: x[x < 5]
Out[28]: array([4, 0, 3, 3, 1, 3, 4, 0])
```

返回的是一个填充有所有满足条件值的一维数组；换句话说，所有掩码数组为`True`位置上的值。

然后我们可以自由地按照我们的意愿操作这些值。例如，我们可以计算我们西雅图降雨数据的一些相关统计信息：

```py
In [29]: # construct a mask of all rainy days
         rainy = (rainfall_mm > 0)

         # construct a mask of all summer days (June 21st is the 172nd day)
         days = np.arange(365)
         summer = (days > 172) & (days < 262)

         print("Median precip on rainy days in 2015 (mm):   ",
               np.median(rainfall_mm[rainy]))
         print("Median precip on summer days in 2015 (mm):  ",
               np.median(rainfall_mm[summer]))
         print("Maximum precip on summer days in 2015 (mm): ",
               np.max(rainfall_mm[summer]))
         print("Median precip on non-summer rainy days (mm):",
               np.median(rainfall_mm[rainy & ~summer]))
Out[29]: Median precip on rainy days in 2015 (mm):    3.8
         Median precip on summer days in 2015 (mm):   0.0
         Maximum precip on summer days in 2015 (mm):  32.5
         Median precip on non-summer rainy days (mm): 4.1
```

通过组合布尔操作、掩码操作和聚合，我们可以非常快速地回答关于我们数据集的这些问题。

# 使用关键词 and/or 与操作符 &/|

一个常见的混淆点是关键词 `and` 和 `or` 与操作符 `&` 和 `|` 之间的区别。什么情况下会使用其中一个而不是另一个？

区别在于：`and` 和 `or` 在整个对象上操作，而 `&` 和 `|` 在对象内的元素上操作。

当你使用`and`或`or`时，相当于要求 Python 将对象视为单个布尔实体。在 Python 中，所有非零整数都将被评估为`True`。因此：

```py
In [30]: bool(42), bool(0)
Out[30]: (True, False)
```

```py
In [31]: bool(42 and 0)
Out[31]: False
```

```py
In [32]: bool(42 or 0)
Out[32]: True
```

当你在整数上使用`&`和`|`时，表达式将作用于元素的位表示，对构成数字的各个位应用*与*或*或*：

```py
In [33]: bin(42)
Out[33]: '0b101010'
```

```py
In [34]: bin(59)
Out[34]: '0b111011'
```

```py
In [35]: bin(42 & 59)
Out[35]: '0b101010'
```

```py
In [36]: bin(42 | 59)
Out[36]: '0b111011'
```

注意，要产生结果，需要按顺序比较二进制表示的相应位。

当你在 NumPy 中有一个布尔值数组时，可以将其视为一个比特串，其中`1 = True`，`0 = False`，`&`和`|`将类似于前面的示例中的操作：

```py
In [37]: A = np.array([1, 0, 1, 0, 1, 0], dtype=bool)
         B = np.array([1, 1, 1, 0, 1, 1], dtype=bool)
         A | B
Out[37]: array([ True,  True,  True, False,  True,  True])
```

但是，如果你在这些数组上使用`or`，它将尝试评估整个数组对象的真假，这不是一个明确定义的值：

```py
In [38]: A or B
ValueError: The truth value of an array with more than one element is
          > ambiguous.
          a.any() or a.all()
```

类似地，当对给定数组评估布尔表达式时，应该使用`|`或`&`而不是`or`或`and`：

```py
In [39]: x = np.arange(10)
         (x > 4) & (x < 8)
Out[39]: array([False, False, False, False, False,  True,  True,  True, False,
                False])
```

尝试评估整个数组的真假将会产生与我们之前看到的相同的`ValueError`：

```py
In [40]: (x > 4) and (x < 8)
ValueError: The truth value of an array with more than one element is
          > ambiguous.
          a.any() or a.all()
```

因此，请记住：`and`和`or`对整个对象执行单个布尔评估，而`&`和`|`对对象的内容（各个位或字节）执行多个布尔评估。对于布尔 NumPy 数组，后者几乎总是期望的操作。
