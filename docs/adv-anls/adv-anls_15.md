# 第12章。Python中的数据操作和可视化

在[第8章](ch08.html#r-data-manipulation-visualization)中，你学习如何在数据操作和可视化方面使用`tidyverse`套件。在这里，我们将在Python中演示类似的技术，应用于相同的*star*数据集。特别是，我们将使用`pandas`和`seaborn`分别进行数据操作和可视化。这不是关于这些模块或Python在数据分析中的全部功能的详尽指南。相反，这足以让你自己探索。

尽可能地，我会模仿我们在[第8章](ch08.html#r-data-manipulation-visualization)中所做的步骤，并执行相同的操作。因为熟悉这些操作，我会更专注于如何在Python中执行数据操作和可视化，而不是为什么这样做。让我们加载必要的模块，并开始使用*star*。第三个模块`matplotlib`对你来说是新的，将用于辅助我们在`seaborn`中的工作。它已经随Anaconda安装。具体来说，我们将使用`pyplot`子模块，并将其别名为`plt`。

```py
In [1]:  import pandas as pd
         import seaborn as sns
         import matplotlib.pyplot as plt

         star = pd.read_excel('datasets/star/star.xlsx')
         star.head()
Out[1]:
   tmathssk  treadssk             classk  totexpk   sex freelunk   race  \
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

# 列操作

在[第11章](ch11.html#data-structures-in-python)中，你学习到`pandas`将尝试将一维数据结构转换为Series。当选择列时，这个看似微不足道的点将变得非常重要。让我们看一个例子：假设我们只想保留DataFrame中的*tmathssk*列。我们可以使用熟悉的单括号表示法来做到这一点，但从技术上讲，这会导致一个Series，而不是DataFrame：

```py
In [2]:  math_scores = star['tmathssk']
         type(math_scores)

Out[2]: pandas.core.series.Series
```

如果我们不能确定是否希望*math_scores*保持为一维结构，最好将其保留为DataFrame。为此，我们可以使用两组方括号而不是一组：

```py
In [3]: math_scores = star[['tmathssk']]
        type(math_scores)

Out[3]: pandas.core.frame.DataFrame
```

按照这个模式，我们可以在*star*中仅保留所需的列。我将使用`columns`属性来确认。

```py
In [4]:  star = star[['tmathssk','treadssk','classk','totexpk','schidkn']]
         star.columns

Out[4]: Index(['tmathssk', 'treadssk', 'classk',
             'totexpk', 'schidkn'], dtype='object')
```

要删除特定的列，请使用`drop()`方法。`drop()`可以用于删除列或行，所以我们需要通过使用`axis`参数来指定。在`pandas`中，行是轴`0`，列是轴`1`，正如[图12-1](#pandas-axes)所示。

![star数据集的轴](assets/aina_1201.png)

###### 图12-1。一个`pandas` DataFrame的轴

这里是如何删除*schidkn*列的方法：

```py
In [5]: star = star.drop('schidkn', axis=1)
        star.columns

Out[5]: Index(['tmathssk', 'treadssk',
            'classk', 'totexpk'], dtype='object')
```

现在让我们来看看如何从DataFrame中派生新的列。我们可以使用方括号表示法来做到这一点——这一次，我确实希望结果是一个Series，因为DataFrame的每一列实际上都是一个Series（就像R数据框的每一列实际上都是一个向量一样）。在这里，我将计算数学和阅读成绩的综合：

```py
In [6]: star['new_column'] = star['tmathssk'] + star['treadssk']
        star.head()

Out[6]:
   tmathssk  treadssk             classk  totexpk  new_column
0       473       447        small.class        7         920
1       536       450        small.class       21         986
2       463       439  regular.with.aide        0         902
3       559       448            regular       16        1007
4       489       447        small.class        5         936
```

再次，*new_column*不是一个非常描述性的变量名。让我们使用`rename()`函数来修复它。我们将使用`columns`参数，并以你可能不熟悉的格式传递数据：

```py
In [7]: star = star.rename(columns = {'new_column':'ttl_score'})
        star.columns

Out[7]: Index(['tmathssk', 'treadssk', 'classk', 'totexpk', 'ttl_score'],
           dtype='object')
```

上一个示例中使用的花括号表示法是 Python 的 *字典*。字典是 *键-值* 对的集合，每个元素的键和值由冒号分隔。这是 Python 的核心数据结构之一，是你继续学习该语言时要了解的内容。

# 按行进行操作

现在让我们转向按行常见的操作。我们将从排序开始，在 `pandas` 中可以使用 `sort_values()` 方法完成。我们将按照各列的指定顺序传递给 `by` 参数来排序：

```py
In [8]: star.sort_values(by=['classk', 'tmathssk']).head()

Out[8]:
     tmathssk  treadssk   classk  totexpk  ttl_score
309        320       360  regular        6        680
1470       320       315  regular        3        635
2326       339       388  regular        6        727
2820       354       398  regular        6        752
4925       354       391  regular        8        745
```

默认情况下，所有列都按升序排序。要修改这一点，我们可以包含另一个参数 `ascending`，其中包含 `True`/`False` 标志的列表。让我们按照班级大小（*classk*）升序和数学分数（*treadssk*）降序来排序 *star*。因为我们没有将此输出重新分配给 *star*，所以这种排序不会永久影响数据集。

```py
In [9]: # Sort by class size ascending and math score descending
        star.sort_values(by=['classk', 'tmathssk'],
         ascending=[True, False]).head()

Out[9]:
      tmathssk  treadssk   classk  totexpk  ttl_score
724        626       474  regular       15       1100
1466       626       554  regular       11       1180
1634       626       580  regular       15       1206
2476       626       538  regular       20       1164
2495       626       522  regular        7       1148
```

要筛选 DataFrame，我们首先使用条件逻辑创建一个 Series，其中包含每行是否符合某些条件的 `True`/`False` 标志。然后，我们仅保留 DataFrame 中 Series 中标记为 `True` 的记录。例如，让我们仅保留 `classk` 等于 `small.class` 的记录。

```py
In [10]: small_class = star['classk'] == 'small.class'
         small_class.head()

Out[10]:
        0     True
        1     True
        2    False
        3    False
        4     True
        Name: classk, dtype: bool
```

现在，我们可以使用括号过滤此结果的 Series。我们可以使用 `shape` 属性确认新 DataFrame 的行数和列数：

```py
In [11]: star_filtered = star[small_class]
         star_filtered.shape

Out[11]: (1733, 5)
```

`star_filtered` 将包含比 *star* 更少的行，但列数相同：

```py
In [12]: star.shape

Out[12]: (5748, 5)
```

让我们再试一次：找到 `treadssk` 至少为 `500` 的记录：

```py
In [13]: star_filtered = star[star['treadssk'] >= 500]
         star_filtered.shape

Out[13]: (233, 5)
```

还可以使用 and/or 语句根据多个条件进行过滤。就像在 R 中一样，`&` 和 `|` 分别表示 "和" 和 "或"。让我们将前面的两个条件放入括号中，并使用 `&` 将它们连接到一个语句中：

```py
In [14]: # Find all records with reading score at least 500 and in small class
         star_filtered = star[(star['treadssk'] >= 500) &
                   (star['classk'] == 'small.class')]
         star_filtered.shape

Out[14]: (84, 5)
```

# 聚合和连接数据

要在 DataFrame 中对观察结果进行分组，我们将使用 `groupby()` 方法。如果打印 `star_grouped`，你会看到它是一个 `DataFrameGroupBy` 对象：

```py
In [15]: star_grouped = star.groupby('classk')
         star_grouped

Out[15]: <pandas.core.groupby.generic.DataFrameGroupBy
            object at 0x000001EFD8DFF388>
```

现在我们可以选择其他字段来对这个分组的 DataFrame 进行聚合。[Table 12-1](#pandas-agg-types) 列出了一些常见的聚合方法。

表 12-1\. `pandas` 中有用的聚合函数

| 方法 | 聚合类型 |
| --- | --- |
| `sum()` | 总和 |
| `count()` | 计数值 |
| `mean()` | 平均值 |
| `max()` | 最高值 |
| `min()` | 最小值 |
| `std()` | 标准差 |

以下给出了每个班级大小的平均数学分数：

```py
In [16]: star_grouped[['tmathssk']].mean()

Out[16]:
                        tmathssk
    classk
    regular            483.261000
    regular.with.aide  483.009926
    small.class        491.470283
```

现在我们将找到每年教师经验的最高总分。因为这将返回相当多的行，我将包含 `head()` 方法以仅获取一些行。将多个方法添加到同一命令的这种做法称为方法 *链式调用*：

```py
In [17]: star.groupby('totexpk')[['ttl_score']].max().head()

Out[17]:
                ttl_score
        totexpk
        0             1171
        1             1133
        2             1091
        3             1203
        4             1229
```

[第8章](ch08.html#r-data-manipulation-visualization)回顾了Excel的`VLOOKUP()`和左外连接之间的相似性和差异。我将重新读入*star*和*districts*的最新副本；让我们使用`pandas`来合并这些数据集。我们将使用`merge()`方法将*school-districts*中的数据“查找”到*star*中。通过将`how`参数设置为`left`，我们将指定左外连接，这是最接近`VLOOKUP()`的连接类型：

```py
In [18]: star = pd.read_excel('datasets/star/star.xlsx')
         districts = pd.read_csv('datasets/star/districts.csv')
         star.merge(districts, how='left').head()

Out[18]:
   tmathssk  treadssk             classk  totexpk   sex freelunk   race  \
0       473       447        small.class        7 girl       no  white
1       536       450        small.class       21 girl       no  black
2       463       439  regular.with.aide        0   boy      yes  black
3       559       448            regular       16   boy       no  white
4       489       447        small.class        5 boy      yes  white

   schidkn    school_name          county
0       63     Ridgeville     New Liberty
1       20  South Heights         Selmont
2       19      Bunnlevel         Sattley
3       69          Hokah      Gallipolis
4       79   Lake Mathews  Sugar Mountain
```

Python像R一样对数据连接非常直观：它默认使用*schidkn*进行合并，并同时引入*school_name*和*county*。

# 数据重塑

让我们来看看如何在Python中扩展和延长数据集，再次使用`pandas`。首先，我们可以使用`melt()`函数将*tmathssk*和*treadssk*合并到一列中。为此，我将指定要使用`frame`参数操作的DataFrame，使用`id_vars`作为唯一标识符的变量，并使用`value_vars`将变量融合为单列。我还将指定如何命名生成的值和标签变量，分别为`value_name`和`var_name`：

```py
In [19]: star_pivot = pd.melt(frame=star, id_vars = 'schidkn',
            value_vars=['tmathssk', 'treadssk'], value_name='score',
            var_name='test_type')
         star_pivot.head()

Out[19]:
            schidkn test_type  score
         0       63  tmathssk    473
         1       20  tmathssk    536
         2       19  tmathssk    463
         3       69  tmathssk    559
         4       79  tmathssk    489
```

如何将*tmathssk*和*treadssk*重命名为*math*和*reading*？为此，我将使用一个Python字典来设置一个名为`mapping`的对象，它类似于一个“查找表”以重新编码这些值。我将把它传递给`map()`方法来重新编码*test_type*。我还将使用`unique()`方法确认*test_type*现在仅包含*math*和*reading*：

```py
In [20]: # Rename records in `test_type`
         mapping = {'tmathssk':'math','treadssk':'reading'}
         star_pivot['test_type'] = star_pivot['test_type'].map(mapping)

         # Find unique values in test_type
         star_pivot['test_type'].unique()

Out[20]: array(['math', 'reading'], dtype=object)
```

要将*star_pivot*扩展回单独的*math*和*reading*列，我将使用`pivot_table()`方法。首先，我将指定要使用`index`参数索引的变量，然后使用`columns`和`values`参数指定标签和值来自哪些变量：

在`pandas`中，可以设置唯一的索引列；默认情况下，`pivot_table()`将设置`index`参数中包含的任何变量。为了覆盖这一点，我将使用`reset_index()`方法。要了解更多关于`pandas`中自定义索引以及其他无数的数据操作和分析技术，请参阅《*Python for Data Analysis*》（O'Reilly出版社第2版）作者Wes McKinney的书籍[链接](https://oreil.ly/CrP7B)。

```py
In [21]: star_pivot.pivot_table(index='schidkn',
          columns='test_type', values='score').reset_index()

Out[21]:
         test_type  schidkn        math     reading
         0                1  492.272727  443.848485
         1                2  450.576923  407.153846
         2                3  491.452632  441.000000
         3                4  467.689655  421.620690
         4                5  460.084746  427.593220
         ..             ...         ...         ...
         74              75  504.329268  440.036585
         75              76  490.260417  431.666667
         76              78  468.457627  417.983051
         77              79  490.500000  434.451613
         78              80  490.037037  442.537037

         [79 rows x 3 columns]
```

# 数据可视化

现在让我们简要介绍一下Python中的数据可视化，特别是使用`seaborn`包。`seaborn`在统计分析和`pandas`数据框架中表现特别出色，因此在这里是一个很好的选择。类似于`pandas`是建立在`numpy`之上，`seaborn`利用了另一个流行的Python绘图包`matplotlib`的功能。

`seaborn`包括许多函数来构建不同类型的图表。我们将修改这些函数中的参数，以指定要绘制的数据集、沿x轴和y轴的变量、要使用的颜色等。我们首先通过使用`countplot()`函数来可视化每个*classk*水平的观察计数。

我们的数据集是*star*，我们将使用`data`参数指定。要将*classk*的水平放置在x轴上，我们将使用`x`参数。这导致了图 12-2 中所示的计数图：

```py
In [22]: sns.countplot(x='classk', data=star)
```

![Countplot](assets/aina_1202.png)

###### 图 12-2\. 计数图

现在对于*treadssk*的直方图，我们将使用`displot()`函数。同样，我们将指定`x`和`data`。[图 12-3](#seaborn-histogram)展示了结果：

```py
In [23]: sns.displot(x='treadssk', data=star)
```

![直方图](assets/aina_1203.png)

###### 图 12-3\. 直方图

`seaborn`函数包括许多可选参数，用于自定义图表的外观。例如，让我们将箱数更改为25并将绘图颜色更改为粉红色。这将产生[图 12-4](#seaborn-custom-histogram)：

```py
In [24]: sns.displot(x='treadssk', data=star, bins=25, color='pink')
```

![自定义直方图](assets/aina_1204.png)

###### 图 12-4\. 自定义直方图

要制作箱线图，使用`boxplot()`函数，如[图 12-5](#seaborn-boxplot)所示：

```py
In [25]: sns.boxplot(x='treadssk', data=star)
```

到目前为止，在所有这些情况下，我们都可以通过将感兴趣的变量包含在`y`参数中来“翻转”图表。让我们尝试使用我们的箱线图进行演示，我们将得到图 12-6 中显示的输出：

```py
In [26]: sns.boxplot(y='treadssk', data=star)
```

![箱线图](assets/aina_1205.png)

###### 图 12-5\. 箱线图

![箱线图](assets/aina_1206.png)

###### 图 12-6\. “翻转”的箱线图

要为每个班级大小水平制作箱线图，我们将在`classk`的x轴上包含一个额外的参数，这样就得到了图 12-7 中所示的按组的箱线图：

```py
In [27]: sns.boxplot(x='classk', y='treadssk', data=star)
```

![分组箱线图](assets/aina_1207.png)

###### 图 12-7\. 按组的箱线图

现在让我们使用`scatterplot()`函数在x轴上绘制*tmathssk*与y轴上的*treadssk*之间的关系。[图 12-8](#seaborn-scatterplot)是结果：

```py
In [28]: sns.scatterplot(x='tmathssk', y='treadssk', data=star)
```

![散点图](assets/aina_1208.png)

###### 图 12-8\. 散点图

假设我们想要与外部观众分享这个图表，他们可能不熟悉*treadssk*和*tmathssk*是什么。我们可以通过从`matplotlib.pyplot`中借用功能向该图表添加更多有用的标签。我们将与之前相同地运行`scatterplot()`函数，但这次我们还将调用`pyplot`中的函数来添加自定义的x和y轴标签，以及图表标题。这导致了图 12-9 中显示的带有标签的散点图：

```py
In [29]: sns.scatterplot(x='tmathssk', y='treadssk', data=star)
         plt.xlabel('Math score')
         plt.ylabel('Reading score')
         plt.title('Math score versus reading score')
```

![带有自定义标签和标题的散点图](assets/aina_1209.png)

###### 图 12-9\. 带有自定义轴标签和标题的散点图

`seaborn`包括许多功能，用于构建视觉上吸引人的数据可视化。要了解更多，请查看[官方文档](https://oreil.ly/2joMU)。

# 结论

`pandas`和`seaborn`能做的远不止这些，但这已足以让你开始真正的任务：探索和测试数据中的关系。这将是[第13章](ch13.html#python-for-data-analysis-capstone)的重点。

# 练习

[书籍存储库](https://oreil.ly/hFEOG)的*datasets*子文件夹*census*中有两个文件，*census.csv*和*census-divisions.csv*。将这些文件读入Python并执行以下操作：

1.  按地区升序、分区升序和人口降序对数据进行排序（你需要合并数据集才能做到这一点）。将结果写入Excel工作表。

1.  从合并的数据集中删除邮政编码字段。

1.  创建一个名为*density*的新列，该列是人口除以陆地面积的计算结果。

1.  可视化2015年所有观察中陆地面积和人口之间的关系。

1.  找出每个地区在2015年的总人口。

1.  创建一个包含州名和人口的表，每年2010年至2015年的人口分别保留在一个单独的列中。
