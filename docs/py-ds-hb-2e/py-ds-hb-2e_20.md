# 第十七章：分层索引

到目前为止，我们主要关注存储在 Pandas `Series` 和 `DataFrame` 对象中的一维和二维数据。通常，超出这些维度存储更高维度的数据是有用的——也就是说，数据由超过一个或两个键索引。早期的 Pandas 版本提供了 `Panel` 和 `Panel4D` 对象，可以视为二维 `DataFrame` 的三维或四维类比，但在实践中使用起来有些笨拙。处理更高维数据的更常见模式是利用*分层索引*（也称为*多重索引*），在单个索引中包含多个索引*级别*。通过这种方式，高维数据可以在熟悉的一维 `Series` 和二维 `DataFrame` 对象中紧凑地表示。（如果你对带有 Pandas 风格灵活索引的真正的 *N* 维数组感兴趣，可以查看优秀的[Xarray 包](https://xarray.pydata.org)。）

在本章中，我们将探讨直接创建 `MultiIndex` 对象；在多重索引数据中进行索引、切片和计算统计信息时的考虑；以及在简单索引和分层索引数据表示之间进行转换的有用程序。

我们从标准导入开始：

```py
In [1]: import pandas as pd
        import numpy as np
```

# 一个多重索引的系列

让我们首先考虑如何在一维 `Series` 中表示二维数据。为了具体起见，我们将考虑一个数据系列，其中每个点都有一个字符和数值键。

## 不好的方法

假设你想要跟踪两个不同年份的州数据。使用我们已经介绍过的 Pandas 工具，你可能会简单地使用 Python 元组作为键：

```py
In [2]: index = [('California', 2010), ('California', 2020),
                 ('New York', 2010), ('New York', 2020),
                 ('Texas', 2010), ('Texas', 2020)]
        populations = [37253956, 39538223,
                       19378102, 20201249,
                       25145561, 29145505]
        pop = pd.Series(populations, index=index)
        pop
Out[2]: (California, 2010)    37253956
        (California, 2020)    39538223
        (New York, 2010)      19378102
        (New York, 2020)      20201249
        (Texas, 2010)         25145561
        (Texas, 2020)         29145505
        dtype: int64
```

使用这种索引方案，你可以直接根据这个元组索引或切片系列：

```py
In [3]: pop[('California', 2020):('Texas', 2010)]
Out[3]: (California, 2020)    39538223
        (New York, 2010)      19378102
        (New York, 2020)      20201249
        (Texas, 2010)         25145561
        dtype: int64
```

但便利性到此为止。例如，如果你需要选择所有 2010 年的值，你将需要做一些混乱的（可能是缓慢的）整理来实现它：

```py
In [4]: pop[[i for i in pop.index if i[1] == 2010]]
Out[4]: (California, 2010)    37253956
        (New York, 2010)      19378102
        (Texas, 2010)         25145561
        dtype: int64
```

这会产生期望的结果，但不如我们在 Pandas 中已经喜爱的切片语法那样清晰（或对于大型数据集来说不够高效）。

## 更好的方法：Pandas 多重索引

幸运的是，Pandas 提供了更好的方法。我们基于元组的索引本质上是一个简单的多重索引，而 Pandas 的 `MultiIndex` 类型给了我们希望拥有的操作类型。我们可以从元组创建一个多重索引，如下所示：

```py
In [5]: index = pd.MultiIndex.from_tuples(index)
```

`MultiIndex` 表示多个*索引级别*——在这种情况下，州名和年份——以及每个数据点的多个*标签*，这些标签编码了这些级别。

如果我们使用这个 `MultiIndex` 重新索引我们的系列，我们将看到数据的分层表示：

```py
In [6]: pop = pop.reindex(index)
        pop
Out[6]: California  2010    37253956
                    2020    39538223
        New York    2010    19378102
                    2020    20201249
        Texas       2010    25145561
                    2020    29145505
        dtype: int64
```

这里 Series 表示法的前两列显示了多个索引值，而第三列显示了数据。请注意，第一列中有些条目缺失：在这种多重索引表示中，任何空白条目表示与上一行相同的值。

现在，要访问所有第二个索引为 2020 的数据，我们可以使用 Pandas 切片表示法：

```py
In [7]: pop[:, 2020]
Out[7]: California    39538223
        New York      20201249
        Texas         29145505
        dtype: int64
```

结果是一个仅包含我们感兴趣键的单索引 Series。这种语法比我们最初使用的基于元组的多重索引解决方案更方便（并且操作效率更高！）。接下来我们将进一步讨论在具有分层索引数据上进行此类索引操作。

## 作为额外维度的 MultiIndex

您可能会注意到这里还有另一点：我们本可以使用带有索引和列标签的简单`DataFrame`存储相同的数据。实际上，Pandas 就是考虑到这种等价性而构建的。`unstack`方法将快速将一个多重索引的`Series`转换为传统索引的`DataFrame`：

```py
In [8]: pop_df = pop.unstack()
        pop_df
Out[8]:                 2010      2020
        California  37253956  39538223
        New York    19378102  20201249
        Texas       25145561  29145505
```

自然，`stack`方法提供了相反的操作：

```py
In [9]: pop_df.stack()
Out[9]: California  2010    37253956
                    2020    39538223
        New York    2010    19378102
                    2020    20201249
        Texas       2010    25145561
                    2020    29145505
        dtype: int64
```

看到这些，您可能会想知道为什么我们要费心处理分层索引。原因很简单：正如我们能够使用多重索引来操作一个维度的`Series`中的二维数据一样，我们也可以用它来操作`Series`或`DataFrame`中的三维或更高维度的数据。多重索引中的每个额外级别代表了数据的一个额外维度；利用这个特性使我们在能够表示的数据类型上有了更大的灵活性。具体来说，我们可能希望为每个州在每年的人口（例如 18 岁以下人口）添加另一列人口统计数据；使用`MultiIndex`，这就像在`DataFrame`中添加另一列数据那样简单：

```py
In [10]: pop_df = pd.DataFrame({'total': pop,
                                'under18': [9284094, 8898092,
                                            4318033, 4181528,
                                            6879014, 7432474]})
         pop_df
Out[10]:                     total  under18
         California 2010  37253956  9284094
                    2020  39538223  8898092
         New York   2010  19378102  4318033
                    2020  20201249  4181528
         Texas      2010  25145561  6879014
                    2020  29145505  7432474
```

此外，所有在第十五章讨论的 ufunc 和其他功能也适用于层次索引。在此我们计算按年龄小于 18 岁人口的比例，给定上述数据：

```py
In [11]: f_u18 = pop_df['under18'] / pop_df['total']
         f_u18.unstack()
Out[11]:                 2010      2020
         California  0.249211  0.225050
         New York    0.222831  0.206994
         Texas       0.273568  0.255013
```

这使我们能够轻松快速地操作和探索甚至是高维数据。

# MultiIndex 创建方法

构建一个多重索引的`Series`或`DataFrame`最直接的方法是简单地将两个或更多索引数组列表传递给构造函数。例如：

```py
In [12]: df = pd.DataFrame(np.random.rand(4, 2),
                           index=[['a', 'a', 'b', 'b'], [1, 2, 1, 2]],
                           columns=['data1', 'data2'])
         df
Out[12]:         data1     data2
         a 1  0.748464  0.561409
           2  0.379199  0.622461
         b 1  0.701679  0.687932
           2  0.436200  0.950664
```

创建`MultiIndex`的工作是在后台完成的。

类似地，如果您传递了适当的元组作为键的字典，Pandas 将自动识别并默认使用`MultiIndex`：

```py
In [13]: data = {('California', 2010): 37253956,
                 ('California', 2020): 39538223,
                 ('New York', 2010): 19378102,
                 ('New York', 2020): 20201249,
                 ('Texas', 2010): 25145561,
                 ('Texas', 2020): 29145505}
         pd.Series(data)
Out[13]: California  2010    37253956
                     2020    39538223
         New York    2010    19378102
                     2020    20201249
         Texas       2010    25145561
                     2020    29145505
         dtype: int64
```

尽管如此，有时明确创建`MultiIndex`也是有用的；我们将看看几种方法来完成这个操作。

## 显式 MultiIndex 构造器

为了更灵活地构建索引，您可以使用`pd.MultiIndex`类中提供的构造方法。例如，就像我们之前做的那样，您可以从给定每个级别索引值的简单数组列表构造一个`MultiIndex`：

```py
In [14]: pd.MultiIndex.from_arrays([['a', 'a', 'b', 'b'], [1, 2, 1, 2]])
Out[14]: MultiIndex([('a', 1),
                     ('a', 2),
                     ('b', 1),
                     ('b', 2)],
                    )
```

或者可以通过提供每个点的多重索引值的元组列表来构建它：

```py
In [15]: pd.MultiIndex.from_tuples([('a', 1), ('a', 2), ('b', 1), ('b', 2)])
Out[15]: MultiIndex([('a', 1),
                     ('a', 2),
                     ('b', 1),
                     ('b', 2)],
                    )
```

甚至可以通过单个索引的笛卡尔积构建它：

```py
In [16]: pd.MultiIndex.from_product([['a', 'b'], [1, 2]])
Out[16]: MultiIndex([('a', 1),
                     ('a', 2),
                     ('b', 1),
                     ('b', 2)],
                    )
```

同样地，可以直接使用其内部编码通过传递`levels`（包含每个级别可用索引值的列表的列表）和`codes`（引用这些标签的列表的列表）构造`MultiIndex`：

```py
In [17]: pd.MultiIndex(levels=[['a', 'b'], [1, 2]],
                       codes=[[0, 0, 1, 1], [0, 1, 0, 1]])
Out[17]: MultiIndex([('a', 1),
                     ('a', 2),
                     ('b', 1),
                     ('b', 2)],
                    )
```

在创建`Series`或`DataFrame`时，可以将任何这些对象作为`index`参数传递，或者将其传递给现有`Series`或`DataFrame`的`reindex`方法。

## 多重索引级别名称

有时候给`MultiIndex`的级别命名会很方便。可以通过在任何先前讨论过的`MultiIndex`构造函数中传递`names`参数来实现，或者在事后设置索引的`names`属性来完成：

```py
In [18]: pop.index.names = ['state', 'year']
         pop
Out[18]: state       year
         California  2010    37253956
                     2020    39538223
         New York    2010    19378102
                     2020    20201249
         Texas       2010    25145561
                     2020    29145505
         dtype: int64
```

对于更复杂的数据集，这是一种跟踪各种索引值意义的有用方法。

## 列的多重索引

在`DataFrame`中，行和列是完全对称的，就像行可以具有多级索引一样，列也可以具有多级索引。考虑以下内容，这是一些（有些逼真的）医疗数据的模拟：

```py
In [19]: # hierarchical indices and columns
         index = pd.MultiIndex.from_product([[2013, 2014], [1, 2]],
                                            names=['year', 'visit'])
         columns = pd.MultiIndex.from_product([['Bob', 'Guido', 'Sue'],
                                              ['HR', 'Temp']],
                                              names=['subject', 'type'])

         # mock some data
         data = np.round(np.random.randn(4, 6), 1)
         data[:, ::2] *= 10
         data += 37

         # create the DataFrame
         health_data = pd.DataFrame(data, index=index, columns=columns)
         health_data
Out[19]: subject      Bob       Guido         Sue
         type          HR  Temp    HR  Temp    HR  Temp
         year visit
         2013 1      30.0  38.0  56.0  38.3  45.0  35.8
              2      47.0  37.1  27.0  36.0  37.0  36.4
         2014 1      51.0  35.9  24.0  36.7  32.0  36.2
              2      49.0  36.3  48.0  39.2  31.0  35.7
```

这基本上是四维数据，维度包括主题、测量类型、年份和访问次数。有了这个设置，例如，我们可以通过人名索引顶级列，并获得一个只包含该人信息的完整`DataFrame`：

```py
In [20]: health_data['Guido']
Out[20]: type          HR  Temp
         year visit
         2013 1      56.0  38.3
              2      27.0  36.0
         2014 1      24.0  36.7
              2      48.0  39.2
```

# 对`MultiIndex`进行索引和切片

在`MultiIndex`上进行索引和切片设计得很直观，如果将索引视为增加的维度会有所帮助。我们首先看一下如何对多重索引的序列进行索引，然后再看如何对多重索引的数据框进行索引。

## 多重索引的序列

考虑我们之前看到的州人口的多重索引`Series`：

```py
In [21]: pop
Out[21]: state       year
         California  2010    37253956
                     2020    39538223
         New York    2010    19378102
                     2020    20201249
         Texas       2010    25145561
                     2020    29145505
         dtype: int64
```

我们可以通过使用多个项进行索引来访问单个元素：

```py
In [22]: pop['California', 2010]
Out[22]: 37253956
```

`MultiIndex`还支持*部分索引*，即仅对索引中的一个级别进行索引。结果是另一个`Series`，保留较低级别的索引：

```py
In [23]: pop['California']
Out[23]: year
         2010    37253956
         2020    39538223
         dtype: int64
```

可以进行部分切片，只要`MultiIndex`是排序的（参见“排序和未排序索引”的讨论）：

```py
In [24]: poploc['california':'new york']
Out[24]: state       year
         california  2010    37253956
                     2020    39538223
         new york    2010    19378102
                     2020    20201249
         dtype: int64
```

对排序索引来说，可以通过在第一个索引中传递空切片来在较低级别执行部分索引：

```py
In [25]: pop[:, 2010]
Out[25]: state
         california    37253956
         new york      19378102
         texas         25145561
         dtype: int64
```

其他类型的索引和选择（在第十四章中讨论）同样适用；例如，基于布尔掩码的选择：

```py
In [26]: pop[pop > 22000000]
Out[26]: state       year
         California  2010    37253956
                     2020    39538223
         Texas       2010    25145561
                     2020    29145505
         dtype: int64
```

基于花式索引的选择也是有效的：

```py
In [27]: pop[['California', 'Texas']]
Out[27]: state       year
         California  2010    37253956
                     2020    39538223
         Texas       2010    25145561
                     2020    29145505
         dtype: int64
```

## 多重索引的数据框

多重索引的`DataFrame`表现方式类似。考虑之前的医疗玩具`DataFrame`：

```py
In [28]: health_data
Out[28]: subject      Bob       Guido         Sue
         type          HR  Temp    HR  Temp    HR  Temp
         year visit
         2013 1      30.0  38.0  56.0  38.3  45.0  35.8
              2      47.0  37.1  27.0  36.0  37.0  36.4
         2014 1      51.0  35.9  24.0  36.7  32.0  36.2
              2      49.0  36.3  48.0  39.2  31.0  35.7
```

记住，在`DataFrame`中，列是主要的，用于多重索引的`Series`的语法适用于列。例如，我们可以通过简单的操作恢复 Guido 的心率数据：

```py
In [29]: health_data['Guido', 'HR']
Out[29]: year  visit
         2013  1        56.0
               2        27.0
         2014  1        24.0
               2        48.0
         Name: (Guido, HR), dtype: float64
```

正如单索引情况一样，我们还可以使用在第十四章介绍的`loc`、`iloc`和`ix`索引器。例如：

```py
In [30]: health_data.iloc[:2, :2]
Out[30]: subject      Bob
         type          HR  Temp
         year visit
         2013 1      30.0  38.0
              2      47.0  37.1
```

这些索引器提供了底层二维数据的类似数组的视图，但每个`loc`或`iloc`中的单个索引可以传递多个索引的元组。例如：

```py
In [31]: health_data.loc[:, ('Bob', 'HR')]
Out[31]: year  visit
         2013  1        30.0
               2        47.0
         2014  1        51.0
               2        49.0
         Name: (Bob, HR), dtype: float64
```

在这些索引元组内部工作切片并不特别方便；尝试在元组内创建切片将导致语法错误：

```py
In [32]: health_data.loc[(:, 1), (:, 'HR')]
SyntaxError: invalid syntax (3311942670.py, line 1)
```

您可以通过使用 Python 的内置`slice`函数来明确构建所需的切片，但在这种情况下更好的方法是使用`IndexSlice`对象，Pandas 专门为此提供。例如：

```py
In [33]: idx = pd.IndexSlice
         health_data.loc[idx[:, 1], idx[:, 'HR']]
Out[33]: subject      Bob Guido   Sue
         type          HR    HR    HR
         year visit
         2013 1      30.0  56.0  45.0
         2014 1      51.0  24.0  32.0
```

如您所见，在多重索引的`Series`和`DataFrame`中与数据交互的方式有很多，并且与本书中的许多工具一样，熟悉它们的最佳方法是尝试它们！

# 重新排列多重索引

处理多重索引数据的关键之一是知道如何有效地转换数据。有许多操作将保留数据集中的所有信息，但为各种计算目的重新排列数据。我们在`stack`和`unstack`方法中看到了一个简短的示例，但在控制数据在层次索引和列之间重新排列方面，还有许多其他方法，我们将在这里探讨它们。

## 排序和未排序的索引

我之前简要提到过一个警告，但我应该在这里更加强调。*如果索引未排序，则许多`MultiIndex`切片操作将失败*。让我们仔细看看。

我们将从创建一些简单的多重索引数据开始，其中索引*未按词典顺序排序*：

```py
In [34]: index = pd.MultiIndex.from_product([['a', 'c', 'b'], [1, 2]])
         data = pd.Series(np.random.rand(6), index=index)
         data.index.names = ['char', 'int']
         data
Out[34]: char  int
         a     1      0.280341
               2      0.097290
         c     1      0.206217
               2      0.431771
         b     1      0.100183
               2      0.015851
         dtype: float64
```

如果我们尝试对这个索引进行部分切片，将导致错误：

```py
In [35]: try:
             data['a':'b']
         except KeyError as e:
             print("KeyError", e)
KeyError 'Key length (1) was greater than MultiIndex lexsort depth (0)'
```

尽管从错误消息中不完全清楚，但这是由于`MultiIndex`未排序造成的。出于各种原因，部分切片和其他类似操作要求`MultiIndex`中的级别按排序（即词典）顺序排列。Pandas 提供了许多方便的例程来执行这种类型的排序，例如`DataFrame`的`sort_index`和`sortlevel`方法。我们在这里将使用最简单的`sort_index`：

```py
In [36]: data = data.sort_index()
         data
Out[36]: char  int
         a     1      0.280341
               2      0.097290
         b     1      0.100183
               2      0.015851
         c     1      0.206217
               2      0.431771
         dtype: float64
```

当索引以这种方式排序时，部分切片将按预期工作：

```py
In [37]: data['a':'b']
Out[37]: char  int
         a     1      0.280341
               2      0.097290
         b     1      0.100183
               2      0.015851
         dtype: float64
```

## 堆叠和展开索引

正如我们之前简要看到的，可以将数据集从堆叠的多重索引转换为简单的二维表示，可选地指定要使用的级别：

```py
In [38]: pop.unstack(level=0)
Out[38]: year            2010      2020
         state
         California  37253956  39538223
         New York    19378102  20201249
         Texas       25145561  29145505
```

```py
In [39]: pop.unstack(level=1)
Out[39]: state       year
         California  2010    37253956
                     2020    39538223
         New York    2010    19378102
                     2020    20201249
         Texas       2010    25145561
                     2020    29145505
         dtype: int64
```

`unstack`的相反操作是`stack`，在这里可以用它来恢复原始系列：

```py
In [40]: pop.unstack().stack()
Out[40]: state       year
         California  2010    37253956
                     2020    39538223
         New York    2010    19378102
                     2020    20201249
         Texas       2010    25145561
                     2020    29145505
         dtype: int64
```

## 索引设置和重置

重新排列分层数据的另一种方法是将索引标签转换为列；这可以通过`reset_index`方法来实现。对人口字典调用此方法将导致一个`DataFrame`，其中包含`state`和`year`列，这些列保存了以前在索引中的信息。为了清晰起见，我们可以选择指定数据的名称作为列的表示方式：

```py
In [41]: pop_flat = pop.reset_index(name='population')
         pop_flat
Out[41]:         state  year  population
         0  California  2010    37253956
         1  California  2020    39538223
         2    New York  2010    19378102
         3    New York  2020    20201249
         4       Texas  2010    25145561
         5       Texas  2020    29145505
```

一个常见的模式是从列值构建一个`MultiIndex`。这可以通过`DataFrame`的`set_index`方法来实现，该方法返回一个多重索引的`DataFrame`：

```py
In [42]: pop_flat.set_index(['state', 'year'])
Out[42]:                  population
         state      year
         California 2010    37253956
                    2020    39538223
         New York   2010    19378102
                    2020    20201249
         Texas      2010    25145561
                    2020    29145505
```

在实践中，这种重新索引的方式是探索实际数据集时最有用的模式之一。
