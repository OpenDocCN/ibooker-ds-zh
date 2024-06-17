# 第十九章：合并数据集：merge 和 join

Pandas 提供的一个重要功能是其高性能、内存中的连接和合并操作，如果你曾经使用过数据库，可能对此有所了解。主要接口是`pd.merge`函数，我们将看到几个示例，说明其实际操作方式。

为方便起见，在通常的导入之后，我们再次定义从上一章中定义的`display`函数：

```py
In [1]: import pandas as pd
        import numpy as np

        class display(object):
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

# 关系代数

`pd.merge`中实现的行为是所谓的*关系代数*的一个子集，这是一组操作关系数据的正式规则，形成了大多数数据库中可用操作的概念基础。关系代数方法的优势在于它提出了几个基本操作，这些操作成为任何数据集上更复杂操作的基础。通过在数据库或其他程序中高效实现这些基本操作的词汇，可以执行广泛范围的相当复杂的组合操作。

Pandas 在`pd.merge`函数和`Series`和`DataFrame`对象的相关`join`方法中实现了几个这些基本构建块。正如你将看到的，这些功能让你能够有效地链接来自不同来源的数据。

# 合并的类别

`pd.merge`函数实现了几种类型的连接：*一对一*、*多对一*和*多对多*。通过对`pd.merge`接口进行相同的调用来访问这三种连接类型；所执行的连接类型取决于输入数据的形式。我们将从三种合并类型的简单示例开始，并稍后讨论详细的选项。

## 一对一连接

或许最简单的合并类型是一对一连接，这在许多方面类似于你在第十八章中看到的逐列串联。作为具体示例，请考虑以下两个包含公司几名员工信息的`DataFrame`对象：

```py
In [2]: df1 = pd.DataFrame({'employee': ['Bob', 'Jake', 'Lisa', 'Sue'],
                            'group': ['Accounting', 'Engineering',
                                      'Engineering', 'HR']})
        df2 = pd.DataFrame({'employee': ['Lisa', 'Bob', 'Jake', 'Sue'],
                            'hire_date': [2004, 2008, 2012, 2014]})
        display('df1', 'df2')
Out[2]: df1                         df2
          employee        group       employee  hire_date
        0      Bob   Accounting     0     Lisa       2004
        1     Jake  Engineering     1      Bob       2008
        2     Lisa  Engineering     2     Jake       2012
        3      Sue           HR     3      Sue       2014
```

要将这些信息合并到一个`DataFrame`中，我们可以使用`pd.merge`函数：

```py
In [3]: df3 = pd.merge(df1, df2)
        df3
Out[3]:   employee        group  hire_date
        0      Bob   Accounting       2008
        1     Jake  Engineering       2012
        2     Lisa  Engineering       2004
        3      Sue           HR       2014
```

`pd.merge`函数会识别每个`DataFrame`都有一个`employee`列，并自动使用该列作为键进行连接。合并的结果是一个新的`DataFrame`，它合并了两个输入的信息。请注意，每列中的条目顺序不一定保持一致：在这种情况下，`df1`和`df2`中的`employee`列顺序不同，`pd.merge`函数能够正确处理这一点。此外，请记住，一般情况下合并会丢弃索引，除非是通过索引进行合并（参见`left_index`和`right_index`关键字，稍后讨论）。

## 多对一连接

多对一连接是其中一个键列包含重复条目的连接。对于多对一情况，结果的`DataFrame`将适当地保留这些重复条目。考虑以下多对一连接的示例：

```py
In [4]: df4 = pd.DataFrame({'group': ['Accounting', 'Engineering', 'HR'],
                            'supervisor': ['Carly', 'Guido', 'Steve']})
        display('df3', 'df4', 'pd.merge(df3, df4)')
Out[4]: df3                                   df4
          employee        group  hire_date              group supervisor
        0      Bob   Accounting       2008      0   Accounting      Carly
        1     Jake  Engineering       2012      1  Engineering      Guido
        2     Lisa  Engineering       2004      2           HR      Steve
        3      Sue           HR       2014

        pd.merge(df3, df4)
          employee        group  hire_date supervisor
        0      Bob   Accounting       2008      Carly
        1     Jake  Engineering       2012      Guido
        2     Lisa  Engineering       2004      Guido
        3      Sue           HR       2014      Steve
```

结果的`DataFrame`具有一个额外的列，其中“supervisor”信息重复出现在一个或多个位置，根据输入的要求。

## 多对多连接

多对多连接在概念上可能有点混乱，但仍然定义良好。如果左侧和右侧数组中的键列包含重复项，则结果是多对多合并。通过一个具体的例子可能更清楚。考虑以下例子，其中我们有一个显示特定组与一个或多个技能相关联的`DataFrame`。

通过执行多对多连接，我们可以恢复与任何个人相关联的技能：

```py
In [5]: df5 = pd.DataFrame({'group': ['Accounting', 'Accounting',
                                      'Engineering', 'Engineering', 'HR', 'HR'],
                            'skills': ['math', 'spreadsheets', 'software', 'math',
                                       'spreadsheets', 'organization']})
        display('df1', 'df5', "pd.merge(df1, df5)")
Out[5]: df1                       df5
        employee        group              group        skills
      0      Bob   Accounting     0   Accounting          math
      1     Jake  Engineering     1   Accounting  spreadsheets
      2     Lisa  Engineering     2  Engineering      software
      3      Sue           HR     3  Engineering          math
                                  4           HR  spreadsheets
                                  5           HR  organization

      pd.merge(df1, df5)
        employee        group        skills
      0      Bob   Accounting          math
      1      Bob   Accounting  spreadsheets
      2     Jake  Engineering      software
      3     Jake  Engineering          math
      4     Lisa  Engineering      software
      5     Lisa  Engineering          math
      6      Sue           HR  spreadsheets
      7      Sue           HR  organization
```

这三种类型的连接可以与其他 Pandas 工具一起使用，实现广泛的功能。但实际上，数据集很少像我们这里使用的那样干净。在下一节中，我们将考虑由`pd.merge`提供的一些选项，这些选项使您能够调整连接操作的工作方式。

# 指定合并键

我们已经看到了`pd.merge`的默认行为：它查找两个输入之间一个或多个匹配的列名，并将其用作键。然而，通常列名不会那么匹配，`pd.merge`提供了多种选项来处理这种情况。

## 关键字 on

最简单的方法是使用`on`关键字明确指定键列的名称，该关键字接受列名或列名列表：

```py
In [6]: display('df1', 'df2', "pd.merge(df1, df2, on='employee')")
Out[6]: df1                         df2
          employee        group       employee  hire_date
        0      Bob   Accounting     0     Lisa       2004
        1     Jake  Engineering     1      Bob       2008
        2     Lisa  Engineering     2     Jake       2012
        3      Sue           HR     3      Sue       2014

        pd.merge(df1, df2, on='employee')
          employee        group  hire_date
        0      Bob   Accounting       2008
        1     Jake  Engineering       2012
        2     Lisa  Engineering       2004
        3      Sue           HR       2014
```

此选项仅在左侧和右侧的`DataFrame`都具有指定的列名时有效。

## 关键字 left_on 和 right_on

有时您可能希望合并两个具有不同列名的数据集；例如，我们可能有一个数据集，其中员工姓名标记为“name”而不是“employee”。在这种情况下，我们可以使用`left_on`和`right_on`关键字来指定这两个列名：

```py
In [7]: df3 = pd.DataFrame({'name': ['Bob', 'Jake', 'Lisa', 'Sue'],
                            'salary': [70000, 80000, 120000, 90000]})
        display('df1', 'df3', 'pd.merge(df1, df3, left_on="employee",
                 right_on="name")')
Out[7]: df1                         df3
          employee        group        name  salary
        0      Bob   Accounting     0   Bob   70000
        1     Jake  Engineering     1  Jake   80000
        2     Lisa  Engineering     2  Lisa  120000
        3      Sue           HR     3   Sue   90000

        pd.merge(df1, df3, left_on="employee", right_on="name")
          employee        group  name  salary
        0      Bob   Accounting   Bob   70000
        1     Jake  Engineering  Jake   80000
        2     Lisa  Engineering  Lisa  120000
        3      Sue           HR   Sue   90000
```

如果需要，可以使用`DataFrame.drop()`方法删除多余的列：

```py
In [8]: pd.merge(df1, df3, left_on="employee", right_on="name").drop('name', axis=1)
Out[8]:   employee        group  salary
        0      Bob   Accounting   70000
        1     Jake  Engineering   80000
        2     Lisa  Engineering  120000
        3      Sue           HR   90000
```

## 左索引和右索引关键字

有时，而不是在列上进行合并，您可能希望在索引上进行合并。例如，您的数据可能如下所示：

```py
In [9]: df1a = df1.set_index('employee')
        df2a = df2.set_index('employee')
        display('df1a', 'df2a')
Out[9]: df1a                      df2a
                        group               hire_date
        employee                  employee
        Bob        Accounting     Lisa           2004
        Jake      Engineering     Bob            2008
        Lisa      Engineering     Jake           2012
        Sue                HR     Sue            2014
```

您可以通过在`pd.merge()`中指定`left_index`和/或`right_index`标志，将索引用作合并的键：

```py
In [10]: display('df1a', 'df2a',
                 "pd.merge(df1a, df2a, left_index=True, right_index=True)")
Out[10]: df1a                       df2a
                         group                hire_date
         employee                   employee
         Bob        Accounting      Lisa           2004
         Jake      Engineering      Bob            2008
         Lisa      Engineering      Jake           2012
         Sue                HR      Sue            2014

         pd.merge(df1a, df2a, left_index=True, right_index=True)
                         group  hire_date
         employee
         Bob        Accounting       2008
         Jake      Engineering       2012
         Lisa      Engineering       2004
         Sue                HR       2014
```

为方便起见，Pandas 包括`DataFrame.join()`方法，它执行基于索引的合并而无需额外的关键字：

```py
In [11]: df1a.join(df2a)
Out[11]:                 group  hire_date
         employee
         Bob        Accounting       2008
         Jake      Engineering       2012
         Lisa      Engineering       2004
         Sue                HR       2014
```

如果您希望混合索引和列，可以将`left_index`与`right_on`或`left_on`与`right_index`结合使用，以获得所需的行为：

```py
In [12]: display('df1a', 'df3', "pd.merge(df1a, df3, left_index=True,
                  right_on='name')")
Out[12]: df1a                       df3
                         group      name  salary
         employee                   0   Bob   70000
         Bob        Accounting      1  Jake   80000
         Jake      Engineering      2  Lisa  120000
         Lisa      Engineering      3   Sue   90000
         Sue                HR

         pd.merge(df1a, df3, left_index=True, right_on='name')
                  group  name  salary
         0   Accounting   Bob   70000
         1  Engineering  Jake   80000
         2  Engineering  Lisa  120000
         3           HR   Sue   90000
```

所有这些选项也适用于多个索引和/或多个列；这种行为的界面非常直观。有关更多信息，请参阅[Pandas 文档中的“Merge, Join, and Concatenate”部分](https://oreil.ly/ffyAp)。

# 指定连接的集合算术

在所有前面的示例中，我们忽略了在执行连接时的一个重要考虑因素：连接中使用的集合算术类型。当一个值出现在一个键列中而不出现在另一个键列中时，就会出现这种情况。考虑这个例子：

```py
In [13]: df6 = pd.DataFrame({'name': ['Peter', 'Paul', 'Mary'],
                             'food': ['fish', 'beans', 'bread']},
                            columns=['name', 'food'])
         df7 = pd.DataFrame({'name': ['Mary', 'Joseph'],
                             'drink': ['wine', 'beer']},
                            columns=['name', 'drink'])
         display('df6', 'df7', 'pd.merge(df6, df7)')
Out[13]: df6                  df7
             name   food           name drink
         0  Peter   fish      0    Mary  wine
         1   Paul  beans      1  Joseph  beer
         2   Mary  bread

         pd.merge(df6, df7)
            name   food drink
         0  Mary  bread  wine
```

在这里，我们已经合并了两个仅具有一个共同“name”条目的数据集：Mary。默认情况下，结果包含输入集合的*交集*；这称为*内连接*。我们可以使用`how`关键字显式指定为`"inner"`：

```py
In [14]: pd.merge(df6, df7, how='inner')
Out[14]:    name   food drink
         0  Mary  bread  wine
```

`how`关键字的其他选项包括`'outer'`、`'left'`和`'right'`。*外连接*返回输入列的并集并用 NA 填充缺失值：

```py
In [15]: display('df6', 'df7', "pd.merge(df6, df7, how='outer')")
Out[15]: df6                  df7
             name   food           name drink
         0  Peter   fish      0    Mary  wine
         1   Paul  beans      1  Joseph  beer
         2   Mary  bread

         pd.merge(df6, df7, how='outer')
              name   food drink
         0   Peter   fish   NaN
         1    Paul  beans   NaN
         2    Mary  bread  wine
         3  Joseph    NaN  beer
```

*左连接*和*右连接*分别返回左输入和右输入的连接。例如：

```py
In [16]: display('df6', 'df7', "pd.merge(df6, df7, how='left')")
Out[16]: df6                  df7
             name   food           name drink
         0  Peter   fish      0    Mary  wine
         1   Paul  beans      1  Joseph  beer
         2   Mary  bread

         pd.merge(df6, df7, how='left')
             name   food drink
         0  Peter   fish   NaN
         1   Paul  beans   NaN
         2   Mary  bread  wine
```

现在输出行对应于左输入中的条目。使用`how='right'`的方式也类似工作。

所有这些选项都可以直接应用于之前的任何连接类型。

# 重叠的列名：后缀关键字

最后，您可能会遇到两个输入`DataFrame`具有冲突列名的情况。考虑这个例子：

```py
In [17]: df8 = pd.DataFrame({'name': ['Bob', 'Jake', 'Lisa', 'Sue'],
                             'rank': [1, 2, 3, 4]})
         df9 = pd.DataFrame({'name': ['Bob', 'Jake', 'Lisa', 'Sue'],
                             'rank': [3, 1, 4, 2]})
         display('df8', 'df9', 'pd.merge(df8, df9, on="name")')
Out[17]: df8                df9
            name  rank         name  rank
         0   Bob     1      0   Bob     3
         1  Jake     2      1  Jake     1
         2  Lisa     3      2  Lisa     4
         3   Sue     4      3   Sue     2

         pd.merge(df8, df9, on="name")
            name  rank_x  rank_y
         0   Bob       1       3
         1  Jake       2       1
         2  Lisa       3       4
         3   Sue       4       2
```

因为输出将具有两个冲突的列名，`merge`函数会自动附加后缀`_x`和`_y`以使输出列唯一。如果这些默认值不合适，可以使用`suffixes`关键字指定自定义后缀：

```py
In [18]: pd.merge(df8, df9, on="name", suffixes=["_L", "_R"])
Out[18]:    name  rank_L  rank_R
         0   Bob       1       3
         1  Jake       2       1
         2  Lisa       3       4
         3   Sue       4       2
```

这些后缀适用于可能的任何连接模式，并且在多个重叠列的情况下也适用。

在第二十章中，我们将深入探讨关系代数。有关更多讨论，请参阅 Pandas 文档中的[“Merge, Join, Concatenate and Compare”](https://oreil.ly/l8zZ1)部分。

# 示例：美国各州数据

在合并数据来自不同来源时，合并和连接操作经常出现。在这里，我们将考虑一些关于[美国各州及其人口数据](https://oreil.ly/aq6Xb)的数据示例：

```py
In [19]: # Following are commands to download the data
         # repo = "https://raw.githubusercontent.com/jakevdp/data-USstates/master"
         # !cd data && curl -O {repo}/state-population.csv
         # !cd data && curl -O {repo}/state-areas.csv
         # !cd data && curl -O {repo}/state-abbrevs.csv
```

让我们使用 Pandas 的`read_csv`函数查看这三个数据集：

```py
In [20]: pop = pd.read_csv('data/state-population.csv')
         areas = pd.read_csv('data/state-areas.csv')
         abbrevs = pd.read_csv('data/state-abbrevs.csv')

         display('pop.head()', 'areas.head()', 'abbrevs.head()')
Out[20]: pop.head()
           state/region     ages  year  population
         0           AL  under18  2012   1117489.0
         1           AL    total  2012   4817528.0
         2           AL  under18  2010   1130966.0
         3           AL    total  2010   4785570.0
         4           AL  under18  2011   1125763.0

         areas.head()
                 state  area (sq. mi)
         0     Alabama          52423
         1      Alaska         656425
         2     Arizona         114006
         3    Arkansas          53182
         4  California         163707

         abbrevs.head()
                 state abbreviation
         0     Alabama           AL
         1      Alaska           AK
         2     Arizona           AZ
         3    Arkansas           AR
         4  California           CA
```

根据这些信息，假设我们想要计算一个相对简单的结果：按照 2010 年人口密度对美国各州和领地进行排名。显然，我们在这里有数据来找到这个结果，但我们需要合并数据集来实现这一点。

我们将从一个多对一的合并开始，这将使我们在人口`DataFrame`中得到完整的州名。我们要基于`pop`的`state/region`列和`abbrevs`的`abbreviation`列进行合并。我们将使用`how='outer'`以确保由于标签不匹配而丢失数据：

```py
In [21]: merged = pd.merge(pop, abbrevs, how='outer',
                           left_on='state/region', right_on='abbreviation')
         merged = merged.drop('abbreviation', axis=1) # drop duplicate info
         merged.head()
Out[21]:   state/region     ages  year  population    state
         0           AL  under18  2012   1117489.0  Alabama
         1           AL    total  2012   4817528.0  Alabama
         2           AL  under18  2010   1130966.0  Alabama
         3           AL    total  2010   4785570.0  Alabama
         4           AL  under18  2011   1125763.0  Alabama
```

让我们再次仔细检查是否存在任何不匹配，可以通过查找具有空值的行来完成：

```py
In [22]: merged.isnull().any()
Out[22]: state/region    False
         ages            False
         year            False
         population       True
         state            True
         dtype: bool
```

一些`population`值为 null；让我们找出它们是哪些！

```py
In [23]: merged[merged['population'].isnull()].head()
Out[23]:      state/region     ages  year  population state
         2448           PR  under18  1990         NaN   NaN
         2449           PR    total  1990         NaN   NaN
         2450           PR    total  1991         NaN   NaN
         2451           PR  under18  1991         NaN   NaN
         2452           PR    total  1993         NaN   NaN
```

所有空值人口数据似乎来自于 2000 年之前的波多黎各；这很可能是因为原始来源中没有这些数据。

更重要的是，我们看到一些新的`state`条目也为空，这意味着在`abbrevs`键中没有相应的条目！让我们找出哪些地区缺少这种匹配：

```py
In [24]: merged.loc[merged['state'].isnull(), 'state/region'].unique()
Out[24]: array(['PR', 'USA'], dtype=object)
```

我们可以快速推断问题所在：我们的人口数据包括了波多黎各（PR）和整个美国（USA）的条目，而这些条目在州缩写键中并未出现。我们可以通过填写适当的条目来快速修复这些问题：

```py
In [25]: merged.loc[merged['state/region'] == 'PR', 'state'] = 'Puerto Rico'
         merged.loc[merged['state/region'] == 'USA', 'state'] = 'United States'
         merged.isnull().any()
Out[25]: state/region    False
         ages            False
         year            False
         population       True
         state           False
         dtype: bool
```

`state`列中不再有空值：我们已经准备就绪！

现在我们可以使用类似的过程将结果与区域数据合并。检查我们的结果时，我们将希望在`state`列上进行连接：

```py
In [26]: final = pd.merge(merged, areas, on='state', how='left')
         final.head()
Out[26]:   state/region     ages  year  population    state  area (sq. mi)
         0           AL  under18  2012   1117489.0  Alabama        52423.0
         1           AL    total  2012   4817528.0  Alabama        52423.0
         2           AL  under18  2010   1130966.0  Alabama        52423.0
         3           AL    total  2010   4785570.0  Alabama        52423.0
         4           AL  under18  2011   1125763.0  Alabama        52423.0
```

再次，让我们检查是否存在空值以查看是否存在任何不匹配：

```py
In [27]: final.isnull().any()
Out[27]: state/region     False
         ages             False
         year             False
         population        True
         state            False
         area (sq. mi)     True
         dtype: bool
```

`area`列中有空值；我们可以查看这里被忽略的地区是哪些：

```py
In [28]: final['state'][final['area (sq. mi)'].isnull()].unique()
Out[28]: array(['United States'], dtype=object)
```

我们发现我们的`areas` `DataFrame`中并不包含整个美国的面积。我们可以插入适当的值（例如使用所有州面积的总和），但在这种情况下，我们将仅删除空值，因为整个美国的人口密度与我们当前的讨论无关：

```py
In [29]: final.dropna(inplace=True)
         final.head()
Out[29]:   state/region     ages  year  population    state  area (sq. mi)
         0           AL  under18  2012   1117489.0  Alabama        52423.0
         1           AL    total  2012   4817528.0  Alabama        52423.0
         2           AL  under18  2010   1130966.0  Alabama        52423.0
         3           AL    total  2010   4785570.0  Alabama        52423.0
         4           AL  under18  2011   1125763.0  Alabama        52423.0
```

现在我们已经拥有了所有需要的数据。为了回答我们感兴趣的问题，让我们首先选择与 2010 年对应的数据部分和总人口。我们将使用`query`函数来快速完成这一点（这需要安装 NumExpr 包，请参阅第二十四章）：

```py
In [30]: data2010 = final.query("year == 2010 & ages == 'total'")
         data2010.head()
Out[30]:     state/region   ages  year  population       state  area (sq. mi)
         3             AL  total  2010   4785570.0     Alabama        52423.0
         91            AK  total  2010    713868.0      Alaska       656425.0
         101           AZ  total  2010   6408790.0     Arizona       114006.0
         189           AR  total  2010   2922280.0    Arkansas        53182.0
         197           CA  total  2010  37333601.0  California       163707.0
```

现在让我们计算人口密度并按顺序显示结果。我们将首先根据州重新索引我们的数据，然后计算结果：

```py
In [31]: data2010.set_index('state', inplace=True)
         density = data2010['population'] / data2010['area (sq. mi)']
```

```py
In [32]: density.sort_values(ascending=False, inplace=True)
         density.head()
Out[32]: state
         District of Columbia    8898.897059
         Puerto Rico             1058.665149
         New Jersey              1009.253268
         Rhode Island             681.339159
         Connecticut              645.600649
         dtype: float64
```

结果是美国各州，以及华盛顿特区和波多黎各，按照其 2010 年人口密度（每平方英里居民数）的排名。我们可以看到，数据集中迄今为止最密集的地区是华盛顿特区（即哥伦比亚特区）；在各州中，密度最大的是新泽西州。

我们还可以检查列表的末尾：

```py
In [33]: density.tail()
Out[33]: state
         South Dakota    10.583512
         North Dakota     9.537565
         Montana          6.736171
         Wyoming          5.768079
         Alaska           1.087509
         dtype: float64
```

我们看到迄今为止最稀疏的州是阿拉斯加，平均每平方英里略高于一名居民。

当尝试使用真实数据源回答问题时，这种数据合并是一项常见任务。希望这个例子给您提供了一些想法，展示了如何结合我们涵盖的工具来从数据中获取洞察！
