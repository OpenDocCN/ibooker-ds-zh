# 第十一章. Python 中的数据结构

在第十章中，您学习了简单的 Python 对象类型，如字符串、整数和布尔值。现在让我们来看看如何将多个值组合在一起，称为*集合*。Python 默认提供了几种集合对象类型。我们将从*列表*开始这一章节。我们可以通过用逗号分隔每个条目并将结果放在方括号内来将值放入列表中：

```py
In [1]: my_list = [4, 1, 5, 2]
        my_list

Out[1]: [4, 1, 5, 2]
```

此对象包含所有整数，但本身不是整数数据类型：它是一个*列表*。

```py
In [2]: type(my_list)

Out[2]: list
```

实际上，我们可以在列表中包含各种不同类型的数据……甚至其他列表。

```py
In [3]: my_nested_list = [1, 2, 3, ['Boo!', True]]
        type(my_nested_list)

Out[3]: list
```

正如您所见，列表非常适合存储数据。但是现在，我们真正感兴趣的是能够像 Excel 范围或 R 向量一样工作的东西，然后进入表格数据。一个简单的列表是否符合要求？让我们试试把`my_list`乘以二。

```py
In [4]:  my_list * 2

Out[4]: [4, 1, 5, 2, 4, 1, 5, 2]
```

这可能*不*是您要找的内容：Python 字面上接受了您的请求，并且，嗯，将*列表*加倍，而不是*列表内部的*数字。有办法在这里自己获取我们想要的结果：如果您之前使用过循环，您可以在这里设置一个循环来将每个元素乘以二。如果您以前没有使用过循环，也没关系：更好的选择是导入一个模块，使在 Python 中执行计算变得更容易。为此，我们将使用包含在 Anaconda 中的`numpy`。

# NumPy 数组

```py
In [5]:  import numpy
```

正如其名称所示，`numpy`是 Python 中用于数值计算的模块，并且已经成为 Python 作为分析工具流行的基础。要了解更多关于`numpy`的信息，请访问 Jupyter 菜单栏的帮助部分并选择“NumPy 参考”。现在我们将专注于`numpy` *数组*。这是一个包含所有相同类型项目并且可以在任意数量，或*n*维度中存储数据的集合。我们将专注于一个*一维*数组，并使用`array()`函数从列表中转换我们的第一个数组：

```py
In [6]:  my_array = numpy.array([4, 1, 5, 2])
         my_array

Out[6]: array([4, 1, 5, 2])
```

乍一看，`numpy`数组看起来*很像*一个列表；毕竟，我们甚至是从一个列表*中*创建了它。但我们可以看到它实际上是一个不同的数据类型：

```py
In [7]: type(my_list)

Out[7]: list

In [8]: type(my_array)

Out[8]: numpy.ndarray
```

具体而言，它是一个`ndarray`，或者*n*维数组。因为它是一个不同的数据结构，所以在操作时它可能会表现出不同的行为。例如，当我们将一个`numpy`数组与另一个数组相乘时会发生什么？

```py
In [9]: my_list * 2

Out[9]: [4, 1, 5, 2, 4, 1, 5, 2]

In [10]: my_array * 2

Out[10]: array([ 8,  2, 10,  4])
```

在许多方面，此行为应该让您想起 Excel 范围或 R 向量。事实上，就像 R 向量一样，`numpy`数组将会*强制*数据成为相同类型：

```py
In [11]: my_coerced_array = numpy.array([1, 2, 3, 'Boo!'])
         my_coerced_array

Out[11]: array(['1', '2', '3', 'Boo!'], dtype='<U11')
```

正如您所看到的，`numpy`对于在 Python 中处理数据非常有用。计划*经常*导入它……这意味着经常输入它。幸运的是，您可以通过*别名*减轻负担。我们将使用`as`关键字为`numpy`指定其常用别名`np`：

```py
In [12]:  import numpy as np
```

这为模块提供了一个临时且更易管理的名称。现在，在我们的 Python 会话期间，每次我们想要调用`numpy`中的代码时，我们可以引用其别名。

```py
In [13]: import numpy as np
         # numpy also has a sqrt() function:
         np.sqrt(my_array)

Out[13]: array([2.        , 1.        , 2.23606798, 1.41421356])
```

###### 注意

请记住，别名对于您的 Python 会话是*临时*的。如果重新启动内核或启动新笔记本，则别名将不再有效。

# 索引和子集选择 NumPy 数组

让我们花点时间来探索如何从`numpy`数组中提取单个项，我们可以通过在对象名称旁边直接加上方括号及其索引号来完成：

```py
In [14]: # Get second element... right?
         my_array[2]

Out[14]: 5
```

例如，我们刚刚从数组中提取了第二个元素……*或者我们是这样做的吗？* 让我们重新审视`my_array`；第二个位置*真的*显示了什么？

```py
In [15]: my_array

Out[15]: array([4, 1, 5, 2])
```

看起来`1`位于第二个位置，而`5`实际上在*第三*个位置。是什么解释了这种差异？事实证明，这是因为 Python 计数方式与我们通常的计数方式不同。

作为对这种奇怪概念的预热，想象一下，您因为兴奋地获取新数据集而多次下载它。匆忙行事会让您得到一系列文件，命名如下：

+   *dataset.csv*

+   *dataset (1).csv*

+   *dataset (2).csv*

+   *dataset (3).csv*

作为人类，我们倾向于从一开始计算事物。但计算机通常从*零*开始计数。多个文件下载就是一个例子：我们的第二个文件实际上命名为`dataset (1)`，而不是`dataset (2)`。这称为*零起始索引*，在 Python 中*到处都有*。

这一切都是说，对于 Python 来说，用数字`1`索引的内容返回的是*第二*位置的值，用`2`索引的是第三个，依此类推。

```py
In [16]: # *Now* let's get the second element
         my_array[1]

Out[16]: 1
```

在 Python 中，还可以对一系列连续的值进行子集选择，称为 Python 中的*切片*。让我们尝试找出第二到第四个元素。我们已经了解了零起始索引的要点；这有多难呢？

```py
In [17]: # Get second through fourth elements... right?
         my_array[1:3]

Out[17]: array([1, 5])
```

*但等等，还有更多。* 除了零起始索引外，切片*不包括*结束元素。这意味着我们需要“加 1”到第二个数字以获得我们想要的范围：

```py
In [18]: # *Now* get second through fourth elements
         my_array[1:4]

Out[18]: array([1, 5, 2])
```

在 Python 中，您可以进行更多的切片操作，例如从对象的*末尾*开始或选择从开头到指定位置的所有元素。现在，记住的重要事情是*Python 使用零起始索引*。

二维`numpy`数组可以作为 Python 的表格数据结构，但所有元素必须是相同的数据类型。在业务环境中分析数据时，这很少发生，因此我们将转向`pandas`以满足此要求。

# 引入 Pandas DataFrames

根据计量经济学中*面板数据*的命名，`pandas`在操作和分析表格数据方面特别有帮助。像`numpy`一样，它与 Anaconda 一起安装。典型的别名是`pd`：

```py
In [19]: import pandas as pd
```

`pandas`模块在其代码库中利用了`numpy`，您将看到两者之间的一些相似之处。`pandas`包括一个称为*Series*的一维数据结构。但它最广泛使用的结构是二维*DataFrame*（听起来熟悉吗？）。使用`DataFrame`函数可以从其他数据类型（包括`numpy`数组）创建 DataFrame：

```py
In [20]: record_1 = np.array(['Jack', 72, False])
         record_2 = np.array(['Jill', 65, True])
         record_3 = np.array(['Billy', 68, False])
         record_4 = np.array(['Susie', 69, False])
         record_5 = np.array(['Johnny', 66, False])

         roster = pd.DataFrame(data = [record_1,
             record_2, record_3, record_4, record_5],
               columns = ['name', 'height', 'injury'])

         roster

Out[20]:
              name height injury
         0    Jack     72  False
         1    Jill     65   True
         2   Billy     68  False
         3   Susie     69  False
         4  Johnny     66  False
```

DataFrame 通常包括每列的命名*标签*。行下方还将有一个*索引*，默认情况下从 0 开始。这是一个相当小的数据集要探索，所以让我们找点别的。不幸的是，Python 不会自带任何 DataFrame，但我们可以使用`seaborn`包找到一些。`seaborn`也随 Anaconda 一起安装，并经常被别名为`sns`。`get_dataset_names()`函数将返回可用于使用的 DataFrame 列表：

```py
In [21]: import seaborn as sns
         sns.get_dataset_names()

Out[21]:
        ['anagrams', 'anscombe', 'attention', 'brain_networks', 'car_crashes',
        'diamonds', 'dots', 'exercise', 'flights', 'fmri', 'gammas',
        'geyser', 'iris', 'mpg', 'penguins', 'planets', 'tips', 'titanic']
```

*鸢尾花*听起来熟悉吗？我们可以使用`load_dataset()`函数将其加载到我们的 Python 会话中，并使用`head()`方法打印前五行。

```py
In [22]: iris = sns.load_dataset('iris')
         iris.head()

Out[22]:
                sepal_length  sepal_width  petal_length  petal_width species
        0           5.1          3.5           1.4          0.2  setosa
        1           4.9          3.0           1.4          0.2  setosa
        2           4.7          3.2           1.3          0.2  setosa
        3           4.6          3.1           1.5          0.2  setosa
        4           5.0          3.6           1.4          0.2  setosa
```

# Python 中的数据导入

与 R 一样，最常见的是从外部文件中读取数据，我们需要处理目录来做到这一点。Python 标准库包含了用于处理文件路径和目录的`os`模块：

```py
In [23]: import os
```

对于接下来的部分，请将您的笔记本保存在书籍存储库的主文件夹中。默认情况下，Python 将当前工作目录设置为您的活动文件所在的位置，因此我们不必像在 R 中那样担心更改目录。您仍然可以使用`os`的`getcwd()`和`chdir()`函数分别检查和更改它。

Python 遵循与 R 相同的相对和绝对文件路径的一般规则。让我们看看是否可以使用`os`的`path`子模块中的`isfile()`函数在存储库中定位*test-file.csv*：

```py
In [24]: os.path.isfile('test-file.csv')

Out[24]: True
```

现在我们想要找到该文件，它包含在*test-folder*子文件夹中。

```py
In [25]: os.path.isfile('test-folder/test-file.csv')

Out[25]: True
```

接下来，尝试将此文件的副本放在当前位置上一级的文件夹中。您应该能够使用此代码找到它：

```py
In [26]:  os.path.isfile('../test-file.csv')

Out[26]: True
```

与 R 一样，您通常会从外部来源读取数据以在 Python 中进行操作，而这个来源几乎可以是任何想得到的东西。`pandas`包含从*.xlsx*和*.csv*文件等文件中读取数据到 DataFrame 的功能。为了演示，我们将从书籍存储库中读取我们可靠的*star.xlsx*和*districts.csv*数据集。使用`read_excel()`函数来读取 Excel 工作簿：

```py
In [27]: star = pd.read_excel('datasets/star/star.xlsx')
         star.head()

Out[27]:
   tmathssk  treadssk             classk  totexpk   sex freelunk   race
0       473       447        small.class        7 girl       no  white
1       536       450        small.class       21 girl       no  black
2       463       439  regular.with.aide        0   boy      yes  black
3       559       448            regular       16   boy       no  white
4       489       447        small.class        5 boy      yes  white

   schidkn
0       63
1       20
2       19
3       69
4       79
```

类似地，我们可以使用`pandas`中的`read_csv()`函数来读取*.csv*文件：

```py
In [28]: districts = pd.read_csv('datasets/star/districts.csv')
         districts.head()

Out[28]:
             schidkn      school_name       county
          0        1          Rosalia  New Liberty
          1        2  Montgomeryville       Topton
          2        3             Davy     Wahpeton
          3        4         Steelton    Palestine
          4        6       Tolchester      Sattley
```

如果您想要读取其他 Excel 文件类型或特定范围和工作表，例如，请查看`pandas`文档。

# 探索 DataFrame

让我们继续评估*star* DataFrame。`info()`方法会告诉我们一些重要的信息，比如它的维度和列的类型：

```py
In [29]: star.info()

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 5748 entries, 0 to 5747
    Data columns (total 8 columns):
    #   Column    Non-Null Count  Dtype
    ---  ------    --------------  -----
    0   tmathssk  5748 non-null   int64
    1   treadssk  5748 non-null   int64
    2   classk    5748 non-null   object
    3   totexpk   5748 non-null   int64
    4   sex       5748 non-null   object
    5   freelunk  5748 non-null   object
    6   race      5748 non-null   object
    7   schidkn   5748 non-null   int64
    dtypes: int64(4), object(4)
    memory usage: 359.4+ KB
```

我们可以使用`describe()`方法检索描述性统计信息：

```py
In [30]: star.describe()

Out[30]:
                tmathssk     treadssk      totexpk      schidkn
        count  5748.000000  5748.000000  5748.000000  5748.000000
        mean    485.648051   436.742345     9.307411    39.836639
        std      47.771531    31.772857     5.767700    22.957552
        min     320.000000   315.000000     0.000000     1.000000
        25%     454.000000   414.000000     5.000000    20.000000
        50%     484.000000   433.000000     9.000000    39.000000
        75%     513.000000   453.000000    13.000000    60.000000
        max     626.000000   627.000000    27.000000    80.000000
```

默认情况下，`pandas`仅包括数值变量的描述统计信息。我们可以使用`include='all'`来覆盖这个设置。

```py
In [31]: star.describe(include = 'all')

Out[31]:
           tmathssk     treadssk             classk      totexpk   sex  \
count   5748.000000  5748.000000               5748  5748.000000  5748
unique          NaN          NaN                  3          NaN     2
top             NaN          NaN  regular.with.aide          NaN   boy
freq            NaN          NaN               2015          NaN  2954
mean     485.648051   436.742345                NaN     9.307411   NaN
std       47.771531    31.772857                NaN     5.767700   NaN
min      320.000000   315.000000                NaN     0.000000   NaN
25%      454.000000   414.000000                NaN     5.000000   NaN
50%      484.000000   433.000000                NaN     9.000000   NaN
75%      513.000000   453.000000                NaN    13.000000   NaN
max      626.000000   627.000000                NaN    27.000000   NaN

       freelunk   race      schidkn
count      5748   5748  5748.000000
unique        2      3          NaN
top          no  white          NaN
freq       2973   3869          NaN
mean        NaN    NaN    39.836639
std         NaN    NaN    22.957552
min         NaN    NaN     1.000000
25%         NaN    NaN    20.000000
50%         NaN    NaN    39.000000
75%         NaN    NaN    60.000000
max         NaN    NaN    80.000000
```

`NaN`是一个特殊的`pandas`值，表示缺失或不可用的数据，例如分类变量的标准差。

## DataFrame 的索引和子集

让我们回到小型 *roster* DataFrame，通过它们的行和列位置访问各种元素。要对 DataFrame 进行索引，我们可以使用 `iloc` 方法，或称为 *integer location*。方括号表示法对您来说可能已经很熟悉了，但这次我们需要通过行 *和* 列来进行索引（再次从零开始）。让我们在前面创建的 *roster* DataFrame 上演示一下。

```py
In [32]:  # First row, first column of DataFrame
          roster.iloc[0, 0]

Out[32]: 'Jack'
```

在此处也可以使用切片来捕获多行和列：

```py
In [33]: # Second through fourth rows, first through third columns
         roster.iloc[1:4, 0:3]

Out[33]:
     name height injury
 1   Jill     65   True
 2  Billy     68  False
 3  Susie     69  False
```

要按名称索引整个列，我们可以使用相关的 `loc` 方法。我们将在第一个索引位置留一个空切片以捕获所有行，然后命名感兴趣的列：

```py
In [34]:  # Select all rows in the name column
          roster.loc[:, 'name']

Out[34]:
        0      Jack
        1      Jill
        2     Billy
        3     Susie
        4    Johnny
        Name: name, dtype: object
```

## 写入 DataFrame

`pandas` 还包括函数，可以分别使用 `write_csv()` 和 `write_xlsx()` 方法将 DataFrame 写入到 *.csv* 文件和 *.xlsx* 工作簿中：

```py
In [35]: roster.to_csv('output/roster-output-python.csv')
         roster.to_excel('output/roster-output-python.xlsx')
```

# 结论

在短时间内，您能够从单元素对象进展到列表、`numpy` 数组，最终到 `pandas` DataFrames。希望您能看到这些数据结构之间的演变和联系，并欣赏到所介绍包的附加优势。接下来的 Python 章节将大量依赖于 `pandas`，但您已经看到 `pandas` 本身依赖于 `numpy` 和 Python 的基本规则，例如从零开始的索引。

# 练习

在本章中，您学会了如何在 Python 中使用几种不同的数据结构和集合类型。以下练习提供了关于这些主题的额外实践和见解：

1.  对以下数组进行切片，以便保留第三到第五个元素。

    ```py
    practice_array = ['I', 'am', 'having', 'fun', 'with', 'Python']
    ```

1.  从 `seaborn` 加载 `tips` DataFrame。

    +   打印有关此 DataFrame 的一些信息，例如观测数和每列的类型。

    +   打印此 DataFrame 的描述统计信息。

1.  书籍存储库（[`oreil.ly/RKmg0`](https://oreil.ly/RKmg0)）包含了 *datasets* 文件夹中 *ais* 子文件夹中的 *ais.xlsx* 文件。将其读取为 Python 的 DataFrame。

    +   打印此 DataFrame 的前几行。

    +   仅将此 DataFrame 的 *sport* 列写回 Excel 为 *sport.xlsx*。
