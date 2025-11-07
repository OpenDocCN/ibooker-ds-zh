# 5. 过滤 DataFrame

本章涵盖

+   减少 DataFrame 的内存使用

+   通过一个或多个条件提取`DataFrame`行

+   过滤包含或排除空值的`DataFrame`行

+   选择介于某个范围内的列值

+   从`DataFrame`中删除重复和空值

在第四章中，我们学习了如何通过使用`loc`和`iloc`访问器从`DataFrame`中提取行、列和单元格值。这些访问器在我们知道要针对的行/列的索引标签和位置时工作得很好。有时，我们可能想要通过条件或标准而不是标识符来定位行。例如，我们可能想要提取一个列包含特定值的行子集。

在本章中，我们将学习如何声明包含和排除`DataFrame`行的逻辑条件。我们将看到如何通过使用`AND`和`OR`逻辑结合多个条件。最后，我们将介绍一些 pandas 实用方法，这些方法简化了过滤过程。前方有很多乐趣，让我们开始吧。

## 5.1 优化数据集以减少内存使用

在我们过渡到过滤之前，让我们快速谈谈在 pandas 中减少内存使用。每次导入数据集时，考虑每个列是否以最优化类型存储其数据都很重要。“最佳”数据类型是消耗最少内存或提供最多效用的一种类型。例如，在大多数计算机上，整数比浮点数占用更少的内存，所以如果你的数据集包含整数，理想的情况是将它们导入为整数而不是浮点数。作为另一个例子，如果你的数据集包含日期，理想的情况是将它们导入为日期时间而不是字符串，这允许进行日期时间特定的操作。在本节中，我们将学习一些技巧和窍门，通过将列数据转换为不同类型来减少内存消耗，这将有助于后续的快速过滤。让我们从导入我们最喜欢的数据分析库的常规操作开始：

```
In  [1] import pandas as pd
```

本章的 employees.csv 数据集是一个虚构的公司员工集合。每条记录包括员工的姓氏、性别、在公司的工作开始日期、薪水、经理状态（`True`或`False`）和团队。让我们用`read_csv`函数看一下数据集：

```
In  [2] pd.read_csv("employees.csv")

Out [2]

 **First Name  Gender Start Date    Salary   Mgmt          Team**
0       Douglas    Male     8/6/93       NaN   True     Marketing
1        Thomas    Male    3/31/96   61933.0   True           NaN
2         Maria  Female        NaN  130590.0  False       Finance
3         Jerry     NaN     3/4/05  138705.0   True       Finance
4         Larry    Male    1/24/98  101004.0   True            IT
...         ...     ...        ...       ...    ...           ...
996     Phillip    Male    1/31/84   42392.0  False       Finance
997     Russell    Male    5/20/13   96914.0  False       Product
998       Larry    Male    4/20/13   60500.0  False  Business Dev
999      Albert    Male    5/15/12  129949.0   True         Sales
1000        NaN     NaN        NaN       NaN    NaN           NaN

1001 rows × 6 columns

```

请花点时间注意输出中散布的`NaNs`。每一列都有缺失值。实际上，最后一行只包含`NaNs`。在现实世界中，像这样的不完整数据很常见。数据集可能包含空白行、空白列等。

我们如何提高数据集的效用？我们的第一个优化是我们现在应该感到舒适的。我们可以使用`parse_dates`参数将开始日期列中的文本值转换为日期时间：

```
In  [3] pd.read_csv("employees.csv", parse_dates = ["Start Date"]).head()

Out [3]

 **First Name  Gender  Start Date    Salary   Mgmt       Team**
0    Douglas    Male  1993-08-06       NaN   True  Marketing
1     Thomas    Male  1996-03-31   61933.0   True        NaN
2      Maria  Female         NaT  130590.0  False    Finance
3      Jerry     NaN  2005-03-04  138705.0   True    Finance
4      Larry    Male  1998-01-24  101004.0   True         IT

```

CSV 导入进展顺利，所以让我们将`DataFrame`对象分配给一个描述性的变量，例如`employees`：

```
In  [4] employees = pd.read_csv(
            "employees.csv", parse_dates = ["Start Date"]
        )
```

有几种选项可以提高`DataFrame`操作的速度和效率。首先，让我们总结当前的数据集。我们可以调用`info`方法来查看列列表、它们的数据类型、缺失值的计数以及`DataFrame`的总内存消耗：

```
In  [5] employees.info()

Out [5]

<class 'pandas.core.frame.DataFrame'>
RangeIndex: 1001 entries, 0 to 1000
Data columns (total 6 columns):
 #   Column      Non-Null Count  Dtype
---  ------      --------------  -----
 0   First Name  933 non-null    object
 1   Gender      854 non-null    object
 2   Start Date  999 non-null    datetime64[ns]
 3   Salary      999 non-null    float64
 4   Mgmt        933 non-null    object
 5   Team        957 non-null    object
dtypes: datetime64ns, float64(1), object(4)
message usage: 47.0+ KB

```

让我们从上到下浏览输出结果。我们有一个包含 1,001 行的`DataFrame`，从索引 0 开始，到索引 1000 结束。有四个字符串列，一个日期时间列和一个浮点列。所有六列都有缺失数据。

当前内存使用量约为 47 KB——对于现代计算机来说是一个小数字，但让我们尝试将其数量减少。在阅读以下示例时，请更多地关注百分比减少而不是数值减少。随着数据集的增长，性能改进将更加显著。

### 5.1.1 使用 astype 方法转换数据类型

你注意到 pandas 将 Mgmt 列的值作为字符串导入了吗？该列只存储两个值：`True`和`False`。我们可以通过将值转换为更轻量级的布尔数据类型来减少内存使用。

`astype`方法将`Series`的值转换为不同的数据类型。它接受一个参数：新的数据类型。我们可以传递数据类型或其名称的字符串。

下一个示例从`employees`中提取 Mgmt `Series`并使用`bool`参数调用其`astype`方法。Pandas 返回一个新的布尔`Series`对象。请注意，库将`NaNs`转换为`True`值。我们将在 5.5.4 节中讨论删除缺失值。

```
In  [6] employees["Mgmt"].astype(bool)

Out [6] 0        True
        1        True
        2       False
        3        True
        4        True
                ...
        996     False
        997     False
        998     False
        999      True
        1000     True
        Name: Mgmt, Length: 1001, dtype: bool
```

看起来不错！现在我们已经预览了`Series`将如何显示，我们可以覆盖`employees`中现有的 Mgmt 列。更新`DataFrame`列的工作方式与在字典中设置键值对类似。如果存在具有指定名称的列，pandas 会用新的`Series`覆盖它。如果不存在具有该名称的列，pandas 会创建一个新的`Series`并将其追加到`DataFrame`的右侧。库通过共享索引标签来匹配`Series`和`DataFrame`中的行。

下一个代码示例使用我们的新布尔`Series`覆盖 Mgmt 列。作为提醒，Python 首先评估赋值运算符（`=`）的右侧。首先，我们创建一个新的`Series`，然后我们覆盖现有的 Mgmt 列：

```
In  [7] employees["Mgmt"] = employees["Mgmt"].astype(bool)
```

列赋值不会产生返回值，因此在 Jupyter Notebook 中代码不会输出任何内容。让我们再次查看`DataFrame`以查看结果：

```
In  [8] employees.tail()

Out [8]

 **First Name Gender Start Date    Salary   Mgmt          Team**
996     Phillip   Male 1984-01-31   42392.0  False       Finance
997     Russell   Male 2013-05-20   96914.0  False       Product
998       Larry   Male 2013-04-20   60500.0  False  Business Dev
999      Albert   Male 2012-05-15  129949.0   True         Sales
1000        NaN    NaN        NaT       NaN   True           NaN

```

除了缺失值最后一行的`True`之外，`DataFrame`看起来没有不同。但我们的内存使用量如何呢？让我们再次调用`info`方法来查看差异：

```
In  [9] employees.info()

Out [9]

<class 'pandas.core.frame.DataFrame'>
RangeIndex: 1001 entries, 0 to 1000
Data columns (total 6 columns):
 #   Column      Non-Null Count  Dtype
---  ------      --------------  -----
 0   First Name  933 non-null    object
 1   Gender      854 non-null    object
 2   Start Date  999 non-null    datetime64[ns]
 3   Salary      999 non-null    float64
 4   Mgmt        1001 non-null   bool
 5   Team        957 non-null    object
dtypes: bool(1), datetime64ns, float64(1), object(3)
memory usage: 40.2+ KB

```

我们将`employees`的内存使用量减少了近 15%，从 47 KB 降至 40.2 KB。这是一个相当不错的开始！

接下来，让我们过渡到 Salary 列。如果我们打开原始 CSV 文件，我们可以看到其值存储为整数：

```
First Name,Gender,Start Date,Salary,Mgmt,Team
Douglas,Male,8/6/93,,True,Marketing
Thomas,Male,3/31/96,61933,True,
Maria,Female,,130590,False,Finance
Jerry,,3/4/05,138705,True,Finance

```

在`employees`中，然而，pandas 将 Salary 值存储为浮点数。为了在整个列中支持`NaNs`，pandas 将整数转换为浮点数——这是我们在前面的章节中观察到的库的技术要求。

在我们之前的布尔示例之后，我们可能会尝试使用`astype`方法将列的值强制转换为整数。不幸的是，pandas 会引发一个`ValueError`异常：

```
In  [10] employees["Salary"].astype(int)

---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-99-b148c8b8be90> in *<module>*
----> 1 employees["Salary"].astype(int)

ValueError: Cannot convert non-finite values (NA or inf) to integer

```

Pandas 无法将`NaN`值转换为整数。我们可以通过用常数值替换`NaN`值来解决这个问题。`fillna`方法用我们传入的参数替换`Series`的空值。下一个示例提供了一个填充值为 0。请注意，您选择的价值可能会扭曲数据；0 仅用于示例。

我们知道原始的 Salary 列在其最后一行有一个缺失值。让我们在调用`fillna`方法后查看最后一行：

```
In  [11] employees["Salary"].fillna(0).tail()

Out [11] 996      42392.0
         997      96914.0
         998      60500.0
         999     129949.0
         1000         0.0
         Name: Salary, dtype: float64

```

太好了。现在，由于 Salary 列没有缺失值，我们可以使用`astype`方法将其值转换为整数：

```
In  [12] employees["Salary"].fillna(0).astype(int).tail()

Out [12] 996      42392
         997      96914
         998      60500
         999     129949
         1000         0
         Name: Salary, dtype: int64

```

接下来，我们可以覆盖`employees`中现有的 Salary `Series`：

```
In  [13] employees["Salary"] = employees["Salary"].fillna(0).astype(int)
```

我们可以做出一个额外的优化。Pandas 包括一个称为*类别*的特殊数据类型，这对于相对于其总大小具有少量唯一值的列来说非常理想。一些具有有限数量值的日常数据点示例包括性别、工作日、血型、行星和收入群体。在幕后，pandas 只为每个分类值存储一个副本，而不是在行之间存储重复的副本。

`nunique`方法可以揭示每个`DataFrame`列中唯一值的数量。请注意，它默认排除计数中的缺失值（`NaN`）：

```
In  [14] employees.nunique()

Out [14] First Name    200
         Gender          2
         Start Date    971
         Salary        995
         Mgmt            2
         Team           10
         dtype: int64

```

Gender 和 Team 列是存储分类值的良好候选。在 1,001 行数据中，Gender 只有两个唯一值，而 Team 只有十个唯一值。

让我们再次使用`astype`方法。首先，我们将通过将`"category"`作为参数传递给方法将 Gender 列的值转换为类别：

```
In  [15] employees["Gender"].astype("category")

Out [15] 0         Male
         1         Male
         2       Female
         3          NaN
         4         Male
                  ...
         996       Male
         997       Male
         998       Male
         999       Male
         1000       NaN
         Name: Gender, Length: 1001, dtype: category
         Categories (2, object): [Female, Male]

```

Pandas 已经识别出两个独特的类别：`"Female"`和`"Male"`。我们可以覆盖现有的 Gender 列：

```
In  [16] employees["Gender"] = employees["Gender"].astype("category")
```

让我们通过调用`info`方法检查内存使用情况。由于 pandas 只需要跟踪两个值而不是 1,001 个，内存使用量再次显著下降：

```
In  [17] employees.info()

Out [17]

<class 'pandas.core.frame.DataFrame'>
RangeIndex: 1001 entries, 0 to 1000
Data columns (total 6 columns):
 #   Column      Non-Null Count  Dtype
---  ------      --------------  -----
 0   First Name  933 non-null    object
 1   Gender      854 non-null    category
 2   Start Date  999 non-null    datetime64[ns]
 3   Salary      1001 non-null   int64
 4   Mgmt        1001 non-null   bool
 5   Team        957 non-null    object
dtypes: bool(1), category(1), datetime64ns, int64(1), object(2)
memory usage: 33.5+ KB

```

让我们为只有十个唯一值的 Team 列重复相同的流程：

```
In  [18] employees["Team"] = employees["Team"].astype("category")

In  [19] employees.info()

Out [19]

<class 'pandas.core.frame.DataFrame'>
RangeIndex: 1001 entries, 0 to 1000
Data columns (total 6 columns):
 #   Column      Non-Null Count  Dtype
---  ------      --------------  -----
 0   First Name  933 non-null    object
 1   Gender      854 non-null    category
 2   Start Date  999 non-null    datetime64[ns]
 3   Salary      1001 non-null   int64
 4   Mgmt        1001 non-null   bool
 5   Team        957 non-null    category
dtypes: bool(1), category(2)
memory usage: 27.0+ KB

```

在不到十行代码的情况下，我们已经将`DataFrame`的内存消耗减少了 40%以上。想象一下对拥有数百万行数据集的影响！

## 5.2 通过单个条件进行过滤

提取数据子集可能是数据分析中最常见的操作。一个*子集*是符合某种条件的大数据集的一部分。

假设我们想要生成所有名为`"Maria"`的员工的列表。为了完成这个任务，我们需要根据“First Name”列中的值过滤我们的员工数据集。名为 Maria 的员工列表是所有员工的一个子集。

首先，简要回顾一下 Python 中相等性的工作原理。在 Python 中，相等运算符（`==`）比较两个对象是否相等，如果对象相等则返回`True`，如果不相等则返回`False`。（有关详细解释，请参阅附录 B。）以下是一个简单的示例：

```
In  [20] "Maria" == "Maria"

Out [20] True

In  [21] "Maria" == "Taylor"

Out [21] False
```

要比较每个`Series`条目与一个常量值，我们将`Series`放在等号的一侧，而将值放在另一侧：

```
Series == value
```

有些人可能会认为这种语法会导致错误，但 pandas 足够智能，能够识别出我们想要比较的是每个`Series`值与指定字符串的相等性，而不是与`Series`本身进行比较。我们在第二章中探讨了类似的想法，当时我们将`Series`与数学运算符（如加号）配对。

当我们将`Series`与相等运算符结合时，pandas 返回一个布尔`Series`。下一个示例比较每个“First Name”列的值与`"Maria"`。`True`值表示字符串`"Maria"`确实出现在该索引处，而`False`值表示没有。以下输出表明索引 2 存储的值是`"Maria"`：

```
In  [22] employees["First Name"] == "Maria"

Out [22] 0       False
         1       False
         2        True
         3       False
         4       False
                 ...
         996     False
         997     False
         998     False
         999     False
         1000    False
         Name: First Name, Length: 1001, dtype: bool
```

如果我们只能提取具有 True 值的行，那么我们的 employees DataFrame 将包含数据集中所有的"Maria"记录。幸运的是，pandas 提供了一个方便的语法，通过布尔`Series`提取行。要过滤行，我们在 DataFrame 后提供布尔`Series`，并用方括号括起来：

```
In  [23] employees[employees["First Name"] == "Maria"]

Out [23]

 **First Name    Gender   Start Date   Salary     Mgmt            Team**
2        Maria  Female        NaT  130590  False       Finance
198      Maria  Female 1990-12-27   36067   True       Product
815      Maria     NaN 1986-01-18  106562  False            HR
844      Maria     NaN 1985-06-19  148857  False         Legal
936      Maria  Female 2003-03-14   96250  False  Business Dev
984      Maria  Female 2011-10-15   43455  False   Engineering
```

大获成功！我们已经使用布尔`Series`过滤了“First Name”列中值为`"Maria"`的行。

如果使用多个方括号令人困惑，您可以将布尔`Series`分配给一个描述性变量，然后通过方括号传递该变量。以下代码产生与前面代码相同的行子集：

```
In  [24] marias = employees["First Name"] == "Maria"
         employees[marias]

Out [24]

 **First      Name  Gender   Start  Date   Salary    Mgmt            Team**
2        Maria  Female        NaT  130590  False       Finance
198      Maria  Female 1990-12-27   36067   True       Product
815      Maria     NaN 1986-01-18  106562  False            HR
844      Maria     NaN 1985-06-19  148857  False         Legal
936      Maria  Female 2003-03-14   96250  False  Business Dev
984      Maria  Female 2011-10-15   43455  False   Engineering
```

初学者在比较值相等性时最常见的错误是使用一个等号而不是两个。请记住，单个等号将对象分配给变量，而两个等号检查对象之间的相等性。如果我们在这个例子中不小心使用了单个等号，我们将所有“First Name”列的值覆盖为字符串`"Maria"`。这可不是什么好事。

让我们再举一个例子。如果我们想提取不属于财务团队的员工子集，协议保持不变，但略有变化。我们需要生成一个布尔`Series`，检查团队列的哪些值不等于`"Finance"`。然后我们可以使用布尔`Series`来过滤`employees`。Python 的不等运算符在两个值不相等时返回`True`，在相等时返回`False`：

```
In  [25] "Finance" != "Engineering"

Out [25] True
```

`Series` 对象也与不等式运算符友好地配合。以下示例比较团队列中的值与字符串 `"Finance"`。`True` 表示给定索引的 `Team` 值不是 `"Finance"`，而 `False` 表示 `Team` 值是 `"Finance"`：

```
In  [26] employees["Team"] != "Finance"

Out [26] 0        True
         1        True
         2       False
         3       False
         4        True
                 ...
         996     False
         997      True
         998      True
         999      True
         1000     True
         Name: Team, Length: 1001, dtype: bool
```

`现在我们有了布尔 `Series`，我们可以将其放入方括号中，以提取值为 `True` 的 DataFrame 行。在以下输出中，我们看到 pandas 排除了索引 2 和 3 的行，因为那里的 `Team` 值为 `"Finance"`：

```
In  [27] employees[employees["Team"] != "Finance"]

Out [27]

 **First Name  Gender Start Date  Salary   Mgmt          Team**
0       Douglas    Male 1993-08-06       0   True     Marketing
1        Thomas    Male 1996-03-31   61933   True           NaN
4         Larry    Male 1998-01-24  101004   True            IT
5        Dennis    Male 1987-04-18  115163  False         Legal
6          Ruby  Female 1987-08-17   65476   True       Product
...       ...      ...     ...        ...      ...         ...
995       Henry     NaN 2014-11-23  132483  False  Distribution
997     Russell    Male 2013-05-20   96914  False       Product
998       Larry    Male 2013-04-20   60500  False  Business Dev
999      Albert    Male 2012-05-15  129949   True         Sales
1000        NaN     NaN        NaT       0   True           NaN

899 rows × 6 columns
```

注意，结果包括具有缺失值的行。我们可以在索引 1000 处看到一个例子。在这种情况下，pandas 将 `NaN` 视为不等于字符串 `"Finance"`。

如果我们想要检索公司中的所有经理？经理在 `Mgmt` 列中的值为 `True`。我们可以执行 `employees["Mgmt"] == True`，但不需要，因为 Mgmt 已经是一个布尔 `Series`。`True` 值和 `False` 值已经表明 pandas 应该保留还是丢弃一行。因此，我们可以将 `Mgmt` 列本身放入方括号中：

```
In  [28] employees[employees["Mgmt"]].head()

Out [28]

 **First Name  Gender Start Date  Salary  Mgmt       Team**
0    Douglas    Male 1993-08-06       0  True  Marketing
1     Thomas    Male 1996-03-31   61933  True        NaN
3      Jerry     NaN 2005-03-04  138705  True    Finance
4      Larry    Male 1998-01-24  101004  True         IT
6       Ruby  Female 1987-08-17   65476  True    Product
```

我们也可以使用算术运算符根据数学条件来过滤列。以下示例生成一个布尔 `Series`，用于筛选薪资值大于 $100,000 的记录（有关此语法的更多信息，请参阅第二章）：

```
In  [29] high_earners = employees["Salary"] > 100000
         high_earners.head()

Out [29] 0    False
         1    False
         2     True
         3     True
         4     True
         Name: Salary, dtype: bool
```

让我们看看哪些员工的薪资超过 $100,000：

```
In  [30] employees[high_earners].head()

Out [30]

 **First Name  Gender Start Date  Salary   Mgmt          Team**
2      Maria  Female        NaT  130590  False       Finance
3      Jerry     NaN 2005-03-04  138705   True       Finance
4      Larry    Male 1998-01-24  101004   True            IT
5     Dennis    Male 1987-04-18  115163  False         Legal
9    Frances  Female 2002-08-08  139852   True  Business Dev
```

尝试在 `employees` 的其他列上练习语法。只要提供布尔 `Series`，pandas 就能够过滤 `DataFrame`。

## 5.3 多条件过滤

我们可以通过创建两个独立的布尔 `Series` 并声明 pandas 应在它们之间应用的逻辑标准来使用多个条件过滤 `DataFrame`。

### 5.3.1 AND 条件

假设我们想要找到所有在业务发展团队工作的女性员工。现在 pandas 必须查找两个条件来选择行：性别列中的值为 `"Female"`，团队列中的值为 `"Business Dev"`。这两个标准是独立的，但都必须满足。以下是一个关于如何使用 `AND` 逻辑与两个条件快速提醒的例子：

| 条件 1 | 条件 2 | 评估 |
| --- | --- | --- |
| `True` | `True` | `True` |
| `True` | `False` | `False` |
| `False` | `True` | `False` |
| `False` | `False` | `False` |

让我们一次构建一个布尔 `Series`。我们可以从隔离性别列中的 `"Female"` 值开始：

```
In  [31] is_female = employees["Gender"] == "Female"
```

接下来，我们将针对所有在 `"Business Dev"` 团队工作的员工：

```
In  [32] in_biz_dev = employees["Team"] == "Business Dev"
```

最后，我们需要计算两个 `Series` 的交集，即 `is_female` 和 `in_biz_dev` `Series` 都有 `True` 值的行。将两个 `Series` 都放入方括号中，并在它们之间放置一个 `&` 符号。`&` 符号声明了一个 `AND` 逻辑标准。`is_female` `Series` 必须为 `True`，并且 `in_biz_dev` `Series` 也必须为 `True`：

```
In  [33] employees[is_female & in_biz_dev].head()

Out [33]

 **First Name  Gender Start Date  Salary   Mgmt          Team**
9     Frances  Female 2002-08-08  139852   True  Business Dev
33       Jean  Female 1993-12-18  119082  False  Business Dev
36     Rachel  Female 2009-02-16  142032  False  Business Dev
38  Stephanie  Female 1986-09-13   36844   True  Business Dev
61     Denise  Female 2001-11-06  106862  False  Business Dev
```

只要我们用`&`符号分隔连续的两个`Series`，我们就可以在方括号内包含任意数量的`Series`。以下示例添加了一个第三个标准来识别业务发展团队的女性经理：

```
In  [34] is_manager = employees["Mgmt"]
         employees[is_female & in_biz_dev & is_manager].head()

Out [34]

 **First Name  Gender Start Date  Salary  Mgmt          Team**
9      Frances  Female 2002-08-08  139852  True  Business Dev
38   Stephanie  Female 1986-09-13   36844  True  Business Dev
66       Nancy  Female 2012-12-15  125250  True  Business Dev
92       Linda  Female 2000-05-25  119009  True  Business Dev
111     Bonnie  Female 1999-12-17   42153  True  Business Dev

```

总结来说，`&`符号选择符合所有条件的行。声明两个或多个布尔`Series`，然后使用`&`符号将它们编织在一起。

### 5.3.2 OR 条件

我们也可以提取符合几个条件之一的行。并非所有条件都必须为真，但至少有一个条件必须满足。以下是一个关于如何使用两个条件进行`OR`逻辑的快速提醒：

| 条件 1 | 条件 2 | 评估 |
| --- | --- | --- |
| `True` | `True` | `True` |
| `True` | `False` | `True` |
| `False` | `True` | `True` |
| `False` | `False` | `False` |

假设我们想要识别所有薪资低于 4 万美元或开始日期在 2015 年 1 月 1 日之后的员工。我们可以使用数学运算符如<和>来得到这两个条件的两个单独的布尔`Series`：

```
In  [35] earning_below_40k = employees["Salary"] < 40000
 started_after_2015 = employees["Start Date"] > "2015-01-01"
```

我们在布尔`Series`之间使用管道符号（`|`）来声明`OR`标准。在下一个示例中，我们选择任一布尔`Series`持有`True`值的行：

```
In  [36] employees[earning_below_40k | started_after_2015].tail()

Out [36]

 **First Name  Gender Start Date  Salary   Mgmt         Team**
958      Gloria  Female 1987-10-24   39833  False  Engineering
964       Bruce    Male 1980-05-07   35802   True        Sales
967      Thomas    Male 2016-03-12  105681  False  Engineering
989      Justin     NaN 1991-02-10   38344  False        Legal
1000        NaN     NaN        NaT       0   True          NaN

```

索引位置 958、964、989 和 1000 的行符合薪资条件，索引位置 967 的行符合开始日期条件。Pandas 还会包括符合这两个条件的行。

### 5.3.3 使用 ~ 进行反转

波浪线符号（`~`）反转布尔`Series`中的值。所有`True`值变为`False`，所有`False`值变为`True`。以下是一个简单的示例，使用了一个小的`Series`：

```
In  [37] my_series = pd.Series([True, False, True])
         my_series

Out [37] 0     True
         1    False
         2     True
         dtype: bool

In  [38] ~my_series

Out [38] 0    False
         1     True
         2    False
         dtype: bool
```

当我们想要反转一个条件时，反转非常有用。假设我们想要识别薪资低于 10 万美元的员工。我们可以使用两种方法，第一种是编写`employees["Salary"] < 100000`：

```
In  [39] employees[employees["Salary"] < 100000].head()

Out [39]

 **First Name  Gender Start Date  Salary  Mgmt         Team**
0    Douglas    Male 1993-08-06       0  True    Marketing
1     Thomas    Male 1996-03-31   61933  True          NaN
6       Ruby  Female 1987-08-17   65476  True      Product
7        NaN  Female 2015-07-20   45906  True      Finance
8     Angela  Female 2005-11-22   95570  True  Engineering
```

或者，我们可以反转收入超过或等于 10 万美元的员工的结果集。生成的`DataFrames`将是相同的。在下一个示例中，我们将大于操作放在括号内。这种语法确保 pandas 在反转其值之前生成布尔`Series`。一般来说，当评估顺序可能对 pandas 不清楚时，你应该使用括号：

```
In  [40] employees[~(employees["Salary"] >= 100000)].head()

Out [40]

 **First Name  Gender Start Date  Salary  Mgmt         Team**
0    Douglas    Male 1993-08-06       0  True    Marketing
1     Thomas    Male 1996-03-31   61933  True          NaN
6       Ruby  Female 1987-08-17   65476  True      Product
7        NaN  Female 2015-07-20   45906  True      Finance
8     Angela  Female 2005-11-22   95570  True  Engineering
```

TIP 对于像这样复杂的提取，考虑将布尔`Series`分配给一个描述性变量。

### 5.3.4 布尔方法

Pandas 为喜欢方法而不是运算符的分析师提供了一个替代语法。以下表格显示了相等、不等式和其他算术运算的方法替代方案：

| 操作 | 算术语法 | 方法语法 |
| --- | --- | --- |
| 相等 | `employees["Team"] == "Marketing"` | `employees["Team"].eq("Marketing")` |
| 不等式 | `employees["Team"] != "Marketing"` | `employees["Team"].ne("Marketing")` |
| 小于 | `employees["Salary"] < 100000` | `employees["Salary"].lt(100000)` |
| 小于或等于 | `employees["Salary"] <= 100000` | `employees["Salary"].le(100000)` |
| 大于 | `employees["Salary"] > 100000` | `employees["Salary"].gt(100000)` |
| 大于等于 | `employees["Salary"] >= 100000` | `employees["Salary"].ge(100000)` |

关于使用`&`和`|`符号进行`AND`/`OR`逻辑的规则同样适用。

## 5.4 通过条件过滤

一些过滤操作比简单的相等或不等式检查更复杂。幸运的是，pandas 提供了许多辅助方法，这些方法为这些类型的提取生成布尔`Series`。

### 5.4.1 `isin`方法

如果我们想隔离属于销售、法律或市场营销团队的员工呢？我们可以在方括号内提供三个单独的布尔`Series`，并添加`|`符号来声明`OR`条件：

```
In  [41] sales = employees["Team"] == "Sales"
         legal = employees["Team"] == "Legal"
         mktg  = employees["Team"] == "Marketing"
         employees[sales | legal | mktg].head()

Out [41]

 **First Name  Gender Start Date  Salary   Mgmt       Team**
0     Douglas    Male 1993-08-06       0   True  Marketing
5      Dennis    Male 1987-04-18  115163  False      Legal
11      Julie  Female 1997-10-26  102508   True      Legal
13       Gary    Male 2008-01-27  109831  False      Sales
20       Lois     NaN 1995-04-22   64714   True      Legal
```

虽然这个解决方案是可行的，但它并不具有可扩展性。如果我们的下一个报告要求从 15 个团队而不是三个团队中获取员工呢？为每个条件声明一个`Series`是费时的。

一个更好的解决方案是`isin`方法，它接受一个元素的可迭代序列（列表、元组、`Series`等）并返回一个布尔`Series`。`True`表示 pandas 在可迭代序列的值中找到了行的值，而`False`表示没有找到。当我们有了`Series`，我们可以用它以通常的方式过滤`DataFrame`。下一个示例达到相同的结果集：

```
In  [42] all_star_teams = ["Sales", "Legal", "Marketing"]
         on_all_star_teams = employees["Team"].isin(all_star_teams)
         employees[on_all_star_teams].head()

Out [42]

 **First Name  Gender Start Date  Salary   Mgmt       Team**
0     Douglas    Male 1993-08-06       0   True  Marketing
5      Dennis    Male 1987-04-18  115163  False      Legal
11      Julie  Female 1997-10-26  102508   True      Legal
13       Gary    Male 2008-01-27  109831  False      Sales
20       Lois     NaN 1995-04-22   64714   True      Legal
```

使用`isin`方法的最佳情况是我们事先不知道比较集合，例如当它是动态生成的时候。

### 5.4.2 `between`方法

当处理数字或日期时，我们经常想要提取位于某个范围内的值。假设我们想要识别所有薪资在$80,000 和$90,000 之间的员工。我们可以创建两个布尔`Series`，一个用于声明下限，一个用于声明上限。然后我们可以使用`&`运算符强制两个条件都为`True`：

```
In  [43] higher_than_80 = employees["Salary"] >= 80000
         lower_than_90 = employees["Salary"] < 90000
         employees[higher_than_80 & lower_than_90].head()

Out [43]

 **First Name  Gender Start Date  Salary   Mgmt         Team**
19      Donna  Female 2010-07-22   81014  False      Product
31      Joyce     NaN 2005-02-20   88657  False      Product
35    Theresa  Female 2006-10-10   85182  False        Sales
45      Roger    Male 1980-04-17   88010   True        Sales
54       Sara  Female 2007-08-15   83677  False  Engineering
```

一个稍微更简洁的解决方案是使用一个名为`between`的方法，它接受一个下限和一个上限；它返回一个布尔`Series`，其中`True`表示行的值位于指定的区间内。请注意，第一个参数，即下限，是包含的，而第二个参数，即上限，是排除的。以下代码返回与前面代码相同的`DataFrame`，过滤出在$80,000 和$90,000 之间的薪资：

```
In  [44] between_80k_and_90k = employees["Salary"].between(80000, 90000)
         employees[between_80k_and_90k].head()

Out [44]

 **First Name  Gender Start Date  Salary   Mgmt         Team**
19      Donna  Female 2010-07-22   81014  False      Product
31      Joyce     NaN 2005-02-20   88657  False      Product
35    Theresa  Female 2006-10-10   85182  False        Sales
45      Roger    Male 1980-04-17   88010   True        Sales
54       Sara  Female 2007-08-15   83677  False  Engineering
```

`between`方法也适用于其他数据类型的列。要过滤日期时间，我们可以传递时间范围的起始和结束日期的字符串。该方法的第一个和第二个参数的关键字参数是`left`和`right`。在这里，我们找到所有在 20 世纪 80 年代开始与公司合作的员工：

```
In  [45] eighties_folk = employees["Start Date"].between(
             left = "1980-01-01",
             right = "1990-01-01"
         )

         employees[eighties_folk].head()

Out [45]

 **First Name  Gender Start Date  Salary   Mgmt     Team**
5      Dennis    Male 1987-04-18  115163  False    Legal
6        Ruby  Female 1987-08-17   65476   True  Product
10     Louise  Female 1980-08-12   63241   True      NaN
12    Brandon    Male 1980-12-01  112807   True       HR
17      Shawn    Male 1986-12-07  111737  False  Product
```

我们还可以将`between`方法应用于字符串列。让我们提取所有名字以字母`"R"`开头的员工。我们将以大写字母`"R"`作为包含的下限，并向上到非包含上限`"S"`：

```
In  [46] name_starts_with_r = employees["First Name"].between("R", "S")
         employees[name_starts_with_r].head()

Out [46]

 **First Name  Gender Start Date  Salary   Mgmt          Team**
6        Ruby  Female 1987-08-17   65476   True       Product
36     Rachel  Female 2009-02-16  142032  False  Business Dev
45      Roger    Male 1980-04-17   88010   True         Sales
67     Rachel  Female 1999-08-16   51178   True       Finance
78      Robin  Female 1983-06-04  114797   True         Sales
```

像往常一样，在处理字符和字符串时，请注意大小写敏感性。

### 5.4.3 `isnull`和`notnull`方法

员工数据集包含大量的缺失值。我们可以在前五行中看到一些缺失值：

```
In  [47] employees.head()

Out [47]

 **First Name  Gender Start Date  Salary   Mgmt       Team**
0    Douglas    Male 1993-08-06       0   True  Marketing
1     Thomas    Male 1996-03-31   61933   True        NaN
2      Maria  Female        NaT  130590  False    Finance
3      Jerry     NaN 2005-03-04  138705   True    Finance
4      Larry    Male 1998-01-24  101004   True         IT
```

Pandas 用`NaN`（不是一个数字）标记缺失的文本值和缺失的数值，并用`NaT`（不是一个时间）标记缺失的日期时间值。我们可以在开始日期列的索引位置 2 看到一个示例。

我们可以使用几个 pandas 方法来隔离给定列中具有 null 或现有值的行。`isnull`方法返回一个布尔`Series`，其中`True`表示某行的值缺失：

```
In  [48] employees["Team"].isnull().head()

Out [48] 0    False
         1     True
         2    False
         3    False
         4    False
         Name: Team, dtype: bool
```

Pandas 将`NaT`和`None`值视为 null。下一个示例在开始日期列上调用`isnull`方法：

```
In  [49] employees["Start Date"].isnull().head()

Out [49] 0    False
         1    False
         2     True
         3    False
         4    False
         Name: Start Date, dtype: bool
```

`notnull`方法返回其逆`Series`，其中`True`表示某行的值存在。以下输出表明索引 0、2、3 和 4 没有缺失值：

```
In  [50] employees["Team"].notnull().head()

Out [50] 0     True
         1    False
         2     True
         3     True
         4     True
         Name: Team, dtype: bool
```

我们可以通过反转`isnull`方法返回的`Series`来产生相同的结果集。提醒一下，我们使用波浪符号（`~`）来反转布尔`Series`：

```
In  [51] (~employees["Team"].isnull()).head()

Out [51] 0     True
         1    False
         2     True
         3     True
         4     True
         Name: Team, dtype: bool

```

两种方法都有效，但`notnull`更具有描述性，因此推荐使用。

和往常一样，我们可以使用这些布尔`Series`来提取特定的`DataFrame`行。在这里，我们提取了所有缺少团队值的员工：

```
In  [52] no_team = employees["Team"].isnull()
         employees[no_team].head()

Out [52]

 **First Name  Gender Start Date  Salary   Mgmt Team**
1      Thomas    Male 1996-03-31   61933   True  NaN
10     Louise  Female 1980-08-12   63241   True  NaN
23        NaN    Male 2012-06-14  125792   True  NaN
32        NaN    Male 1998-08-21  122340   True  NaN
91      James     NaN 2005-01-26  128771  False  NaN

```

下一个示例提取了具有现有姓氏值的员工：

```
In  [53] has_name = employees["First Name"].notnull()
         employees[has_name].tail()

Out [53]

 **First Name Gender Start Date  Salary   Mgmt          Team**
995      Henry    NaN 2014-11-23  132483  False  Distribution
996    Phillip   Male 1984-01-31   42392  False       Finance
997    Russell   Male 2013-05-20   96914  False       Product
998      Larry   Male 2013-04-20   60500  False  Business Dev
999     Albert   Male 2012-05-15  129949   True         Sales
```

`isnull`和`notnull`方法是快速过滤一个或多个行中现有和缺失值的最优方式。

### 5.4.4 处理 null 值

在讨论缺失值的话题上，让我们讨论一些处理它们的方法。在 5.2 节中，我们学习了如何使用`fillna`方法用常数值替换`NaNs`。我们也可以删除它们。

让我们通过将数据集恢复到其原始形状来开始本节。我们将使用`read_csv`函数重新导入 CSV 文件：

```
In  [54] employees = pd.read_csv(
             "employees.csv", parse_dates = ["Start Date"]
         )

```

这里是一个提醒，它看起来是这样的：

```
In  [55] employees

Out [55]

 **First Name  Gender Start Date    Salary   Mgmt          Team**
0       Douglas    Male 1993-08-06       NaN   True     Marketing
1        Thomas    Male 1996-03-31   61933.0   True           NaN
2         Maria  Female        NaT  130590.0  False       Finance
3         Jerry     NaN 2005-03-04  138705.0   True       Finance
4         Larry    Male 1998-01-24  101004.0   True            IT
 ...       ...     ...     ...          ...     ...           ...
996     Phillip    Male 1984-01-31   42392.0  False       Finance
997     Russell    Male 2013-05-20   96914.0  False       Product
998       Larry    Male 2013-04-20   60500.0  False  Business Dev
999      Albert    Male 2012-05-15  129949.0   True         Sales
1000        NaN     NaN        NaT       NaN    NaN           NaN

1001 rows × 6 columns

```

`dropna`方法删除包含任何`NaN`值的`DataFrame`行。一行缺失多少个值无关紧要；如果存在单个`NaN`，则方法会排除该行。员工`DataFrame`在薪资列的索引 0、团队列的索引 1、开始日期列的索引 2 和性别列的索引 3 处有缺失值。注意，pandas 在以下输出中排除了所有这些行：

```
In  [56] employees.dropna()

Out [56]

 **First Name  Gender Start Date    Salary   Mgmt          Team**
4        Larry    Male 1998-01-24  101004.0   True            IT
5       Dennis    Male 1987-04-18  115163.0  False         Legal
6         Ruby  Female 1987-08-17   65476.0   True       Product
8       Angela  Female 2005-11-22   95570.0   True   Engineering
9      Frances  Female 2002-08-08  139852.0   True  Business Dev
 ...      ...     ...     ...          ...     ...           ...
994     George    Male 2013-06-21   98874.0   True     Marketing
996    Phillip    Male 1984-01-31   42392.0  False       Finance
997    Russell    Male 2013-05-20   96914.0  False       Product
998      Larry    Male 2013-04-20   60500.0  False  Business Dev
999     Albert    Male 2012-05-15  129949.0   True         Sales

761 rows × 6 columns

```

我们可以将`how`参数传递一个`"all"`的参数来删除所有值都缺失的行。数据集中只有最后一行满足这个条件：

```
In  [57] employees.dropna(how = "all").tail()

Out [57]

 **First Name Gender Start Date    Salary   Mgmt          Team**
995      Henry    NaN 2014-11-23  132483.0  False  Distribution
996    Phillip   Male 1984-01-31   42392.0  False       Finance
997    Russell   Male 2013-05-20   96914.0  False       Product
998      Larry   Male 2013-04-20   60500.0  False  Business Dev
999     Albert   Male 2012-05-15  129949.0   True         Sales

```

`how`参数的默认参数是`"any"`。一个`"any"`的参数会在某行的任何值缺失时删除该行。注意，在先前的输出中，索引标签 995 的性别列中有`NaN`。将此输出与以下输出进行比较，其中第 995 行不存在；pandas 仍然删除了最后一行，因为它至少有一个`NaN`值：

```
In  [58] employees.dropna(how = "any").tail()

Out [58]

 **First Name Gender Start Date    Salary   Mgmt          Team**
994     George   Male 2013-06-21   98874.0   True     Marketing
996    Phillip   Male 1984-01-31   42392.0  False       Finance
997    Russell   Male 2013-05-20   96914.0  False       Product
998      Larry   Male 2013-04-20   60500.0  False  Business Dev
999     Albert   Male 2012-05-15  129949.0   True         Sales

```

我们可以使用 `subset` 参数来针对具有特定列中缺失值的行。下一个示例删除性别列中具有缺失值的行：

```
In  [59] employees.dropna(subset = ["Gender"]).tail()

Out [59]

 **First Name Gender Start Date    Salary   Mgmt          Team**
994     George   Male 2013-06-21   98874.0   True     Marketing
996    Phillip   Male 1984-01-31   42392.0  False       Finance
997    Russell   Male 2013-05-20   96914.0  False       Product
998      Larry   Male 2013-04-20   60500.0  False  Business Dev
999     Albert   Male 2012-05-15  129949.0   True         Sales
```

我们还可以将 `subset` 参数传递给列的列表。如果行在指定的任何列中具有缺失值，pandas 将删除该行。下一个示例删除具有缺失值的开始日期列、薪资列或两者的行：

```
In  [60] employees.dropna(subset = ["Start Date", "Salary"]).head()

Out [60]

 **First Name  Gender Start Date    Salary   Mgmt     Team**
1     Thomas    Male 1996-03-31   61933.0   True      NaN
3      Jerry     NaN 2005-03-04  138705.0   True  Finance
4      Larry    Male 1998-01-24  101004.0   True       IT
5     Dennis    Male 1987-04-18  115163.0  False    Legal
6       Ruby  Female 1987-08-17   65476.0   True  Product

```

`thresh` 参数指定一行必须具有的最小非空值阈值，以便 pandas 保留该行。下一个示例筛选 `employees` 以获取至少有四个现有值的行：

```
In  [61] employees.dropna(how = "any", thresh = 4).head()

Out [61]

 **First Name  Gender Start Date    Salary   Mgmt       Team**
0    Douglas    Male 1993-08-06       NaN   True  Marketing
1     Thomas    Male 1996-03-31   61933.0   True        NaN
2      Maria  Female        NaT  130590.0  False    Finance
3      Jerry     NaN 2005-03-04  138705.0   True    Finance
4      Larry    Male 1998-01-24  101004.0   True         IT

```

当一定数量的缺失值使行对分析无用时，`thresh` 参数非常出色。

## 5.5 处理重复项

缺失值在杂乱的数据集中很常见，重复值也是如此。幸运的是，pandas 包含了几个用于识别和排除重复值的方法。

### 5.5.1 重复项方法

首先，这是一个关于团队列前五行的快速提醒。注意，值 `"Finance"` 出现在索引位置 2 和 3：

```
In  [62] employees["Team"].head()

Out [62] 0    Marketing
         1          NaN
         2      Finance
         3      Finance
         4           IT
         Name: Team, dtype: object
```

`duplicated` 方法返回一个布尔 `Series`，用于识别列中的重复值。Pandas 在 `Series` 中遇到之前遇到的任何值时返回 `True`。考虑下一个示例。`duplicated` 方法将团队列中 `"Finance"` 的第一次出现标记为非重复（`False`）。它将 `"Finance"` 的所有后续出现标记为重复（`True`）。相同的逻辑适用于所有其他团队值：

```
In  [63] employees["Team"].duplicated().head()

Out [63] 0    False
         1    False
         2    False
         3     True
         4    False
         Name: Team, dtype: bool

```

`duplicated` 方法的 `keep` 参数告诉 pandas 保留哪个重复出现的值。它的默认参数 `"first"` 保留每个重复值的第一次出现。以下代码与前面的代码等价：

```
In  [64] employees["Team"].duplicated(keep = "first").head()

Out [64] 0    False
         1    False
         2    False
         3     True
         4    False
         Name: Team, dtype: bool

```

我们还可以要求 pandas 将列中值的最后一次出现标记为非重复。将 `"last"` 字符串传递给 `keep` 参数：

```
In  [65] employees["Team"].duplicated(keep = "last")

Out [65] 0        True
         1        True
         2        True
         3        True
         4        True
                 ...
         996     False
         997     False
         998     False
         999     False
         1000    False
         Name: Team, Length: 1001, dtype: bool

```

假设我们想要从每个团队中提取一名员工。我们可以使用的策略是提取每个独特团队在团队列中的第一行。我们现有的 `duplicated` 方法返回一个布尔 `Series`；`True` 识别第一次出现后的所有重复值。如果我们反转这个 `Series`，我们将得到一个 `Series`，其中 `True` 表示 pandas 首次遇到该值：

```
In  [66] (~employees["Team"].duplicated()).head()

Out [66] 0     True
         1     True
         2     True
         3    False
         4     True
         Name: Team, dtype: bool

```

现在，我们可以通过传递方括号内的布尔 `Series` 来按团队提取一名员工。Pandas 将包含具有团队列中值第一次出现的行。请注意，库将 `NaNs` 视为唯一值：

```
In  [67] first_one_in_team = ~employees["Team"].duplicated()
         employees[first_one_in_team]

Out [67]

 **First Name  Gender Start Date    Salary   Mgmt          Team**
0     Douglas    Male 1993-08-06       NaN   True     Marketing
1      Thomas    Male 1996-03-31   61933.0   True           NaN
2       Maria  Female        NaT  130590.0  False       Finance
4       Larry    Male 1998-01-24  101004.0   True            IT
5      Dennis    Male 1987-04-18  115163.0  False         Legal
6        Ruby  Female 1987-08-17   65476.0   True       Product
8      Angela  Female 2005-11-22   95570.0   True   Engineering
9     Frances  Female 2002-08-08  139852.0   True  Business Dev
12    Brandon    Male 1980-12-01  112807.0   True            HR
13       Gary    Male 2008-01-27  109831.0  False         Sales
40    Michael    Male 2008-10-10   99283.0   True  Distribution

```

这个输出告诉我们，Douglas 是数据集中营销团队的第一个员工，Thomas 是第一个缺少团队的人，Maria 是财务团队的第一个员工，等等。

### 5.5.2 删除重复项方法

`DataFrame` 的 `drop_duplicates` 方法提供了一个方便的快捷方式来完成 5.5.1 节中的操作。默认情况下，该方法会删除所有值都等于之前遇到的一行中的值的行。没有所有六个行值都相等的 `employees` 行，所以使用标准调用方法，该方法对我们来说并没有做什么：

```
In  [68] employees.drop_duplicates()

Out [68]

 **First Name  Gender Start Date    Salary   Mgmt          Team**
0       Douglas    Male 1993-08-06       NaN   True     Marketing
1        Thomas    Male 1996-03-31   61933.0   True           NaN
2         Maria  Female        NaT  130590.0  False       Finance
3         Jerry     NaN 2005-03-04  138705.0   True       Finance
4         Larry    Male 1998-01-24  101004.0   True            IT
 ...       ...     ...     ...          ...     ...           ...
996     Phillip    Male 1984-01-31   42392.0  False       Finance
997     Russell    Male 2013-05-20   96914.0  False       Product
998       Larry    Male 2013-04-20   60500.0  False  Business Dev
999      Albert    Male 2012-05-15  129949.0   True         Sales
1000        NaN     NaN        NaT       NaN    NaN           NaN

1001 rows × 6 columns

```

但我们可以向该方法传递一个 `subset` 参数，其中包含 pandas 应该用来确定行唯一性的列的列表。下一个示例找到 Team 列中每个唯一值的第一个出现。换句话说，pandas 只保留具有 Team 值（例如 `"Marketing"`）的第一个出现的行。它排除了第一个之后具有重复 Team 值的所有行：

```
In  [69] employees.drop_duplicates(subset = ["Team"])

Out [69]

 **First Name  Gender Start Date    Salary   Mgmt          Team**
0     Douglas    Male 1993-08-06       NaN   True     Marketing
1      Thomas    Male 1996-03-31   61933.0   True           NaN
2       Maria  Female        NaT  130590.0  False       Finance
4       Larry    Male 1998-01-24  101004.0   True            IT
5      Dennis    Male 1987-04-18  115163.0  False         Legal
6        Ruby  Female 1987-08-17   65476.0   True       Product
8      Angela  Female 2005-11-22   95570.0   True   Engineering
9     Frances  Female 2002-08-08  139852.0   True  Business Dev
12    Brandon    Male 1980-12-01  112807.0   True            HR
13       Gary    Male 2008-01-27  109831.0  False         Sales
40    Michael    Male 2008-10-10   99283.0   True  Distribution

```

`drop_duplicates` 方法还接受一个 `keep` 参数。我们可以传递一个 `"last"` 参数来保留每个重复值的最后出现行。这些行可能更接近数据集的末尾。在下面的示例中，Alice 是数据集中 HR 团队中的最后一名员工，Justin 是 Legal 团队中的最后一名员工，依此类推：

```
In  [70] employees.drop_duplicates(subset = ["Team"], keep = "last")

Out [70]

 **First Name  Gender Start Date    Salary   Mgmt          Team**
988       Alice  Female 2004-10-05   47638.0  False            HR
989      Justin     NaN 1991-02-10   38344.0  False         Legal
990       Robin  Female 1987-07-24  100765.0   True            IT
993        Tina  Female 1997-05-15   56450.0   True   Engineering
994      George    Male 2013-06-21   98874.0   True     Marketing
995       Henry     NaN 2014-11-23  132483.0  False  Distribution
996     Phillip    Male 1984-01-31   42392.0  False       Finance
997     Russell    Male 2013-05-20   96914.0  False       Product
998       Larry    Male 2013-04-20   60500.0  False  Business Dev
999      Albert    Male 2012-05-15  129949.0   True         Sales
1000        NaN     NaN        NaT       NaN    NaN           NaN
```

对于 `keep` 参数还有一个额外的选项。我们可以传递一个 `False` 参数来排除所有具有重复值的行。如果存在任何其他具有相同值的行，Pandas 将拒绝该行。下一个示例筛选出 `employees` 中在 First Name 列中具有唯一值的行。换句话说，这些名字在 `DataFrame` 中只出现一次：

```
In  [71] employees.drop_duplicates(subset = ["First Name"], keep = False)

Out [71]

 **First Name  Gender Start Date    Salary   Mgmt          Team**
5       Dennis    Male 1987-04-18  115163.0  False         Legal
8       Angela  Female 2005-11-22   95570.0   True   Engineering
33        Jean  Female 1993-12-18  119082.0  False  Business Dev
190      Carol  Female 1996-03-19   57783.0  False       Finance
291      Tammy  Female 1984-11-11  132839.0   True            IT
495     Eugene    Male 1984-05-24   81077.0  False         Sales
688      Brian    Male 2007-04-07   93901.0   True         Legal
832      Keith    Male 2003-02-12  120672.0  False         Legal
887      David    Male 2009-12-05   92242.0  False         Legal

```

假设我们想要通过多个列的组合来识别重复项。我们可能想要找到数据集中具有唯一 First Name 和 Gender 组合的每个员工的第一个出现，例如。为了参考，这里是一个具有 `"Douglas"` 名字和 `"Male"` 性别的所有员工的子集：

```
In  [72] name_is_douglas = employees["First Name"] == "Douglas"
         is_male = employees["Gender"] == "Male"
         employees[name_is_douglas & is_male]

Out [72]

 **First Name Gender Start Date    Salary   Mgmt         Team**
0      Douglas   Male 1993-08-06       NaN   True    Marketing
217    Douglas   Male 1999-09-03   83341.0   True           IT
322    Douglas   Male 2002-01-08   41428.0  False      Product
835    Douglas   Male 2007-08-04  132175.0  False  Engineering

```

我们可以将列的列表传递给 `drop_duplicates` 方法的 `subset` 参数。Pandas 将使用这些列来确定重复项的存在。下一个示例使用性别和 Team 列中的值的组合来识别重复项：

```
In  [73] employees.drop_duplicates(subset = ["Gender", "Team"]).head()

Out [73]

 **First Name  Gender Start Date    Salary   Mgmt       Team**
0    Douglas    Male 1993-08-06       NaN   True  Marketing
1     Thomas    Male 1996-03-31   61933.0   True        NaN
2      Maria  Female        NaT  130590.0  False    Finance
3      Jerry     NaN 2005-03-04  138705.0   True    Finance
4      Larry    Male 1998-01-24  101004.0   True         IT
```

让我们浏览一下输出。索引为 0 的行持有 employees 数据集中 `"Douglas"` 和 `"Male"` 性别的第一个出现。Pandas 将排除任何具有相同两个值的其他行。为了澄清，如果某行具有 `"Douglas"` 名字和不同的 `"Male"` 性别，库仍然会包括该行。同样，它也会包括具有 `"Male"` 性别和不同的 `"Douglas"` 名字的行。Pandas 使用两个列的值组合来识别重复项。

## 5.6 编程挑战

这是您练习本章引入的概念的机会。

### 5.6.1 问题

netflix.csv 数据集是 Netflix 视频流媒体服务在 2019 年 11 月可观看的近 6,000 个标题的集合。它包括四个列：视频的标题、导演、Netflix 添加它的日期以及它的类型/类别。导演和 date_added 列包含缺失值。我们可以在以下输出的索引位置 0、2 和 5836 中看到示例：

```
In  [74] pd.read_csv("netflix.csv")

Out [74]

 **title        director date_added     type**
0               Alias Grace             NaN   3-Nov-17  TV Show
1            A Patch of Fog  Michael Lennox  15-Apr-17    Movie
2                  Lunatics             NaN  19-Apr-19  TV Show
3                 Uriyadi 2     Vijay Kumar   2-Aug-19    Movie
4         Shrek the Musical     Jason Moore  29-Dec-13    Movie
   ...            ...               ...          ...        ...
5832            The Pursuit     John Papola   7-Aug-19    Movie
5833       Hurricane Bianca   Matt Kugelman   1-Jan-17    Movie
5834           Amar's Hands  Khaled Youssef  26-Apr-19    Movie
5835  Bill Nye: Science Guy  Jason Sussberg  25-Apr-18    Movie
5836           Age of Glory             NaN        NaN  TV Show

5837 rows × 4 columns

```

使用本章学到的技能，解决以下挑战：

1.  优化数据集以减少内存使用并最大化实用性。

1.  查找所有标题为`"Limitless"`的行。

1.  查找所有导演为`"Robert Rodriguez"`且类型为`"Movie"`的行。

1.  查找所有`date_added`值为`"2019-07-31"`或导演为`"Robert"` `Altman`的行。

1.  查找所有导演为`"Orson Welles"`、`"Aditya"` `Kripalani`或`"`Sam Raimi"`的行。

1.  查找所有`date_added`值在 2019 年 5 月 1 日至 2019 年 6 月 1 日之间的行。

1.  删除导演列中所有包含`NaN`值的行。

1.  确定 Netflix 在其目录中仅添加了一部电影的日子。

### 5.6.2 解决方案

让我们解决这些问题！

1.  为了优化数据集以节省内存和提高实用性，我们首先可以将`date_added`列的值转换为日期时间格式。我们可以在导入时通过将`parse_dates`参数传递给`read_csv`函数来强制类型转换：

    ```
    In  [75] netflix = pd.read_csv("netflix.csv", parse_dates = ["date_added"])
    ```

    保持基准很重要，让我们看看当前的内存使用情况：

    ```
    In  [76] netflix.info()

    Out [76]

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 5837 entries, 0 to 5836
    Data columns (total 4 columns):
     #   Column      Non-Null Count  Dtype
    ---  ------      --------------  -----
     0   title       5837 non-null   object
     1   director    3936 non-null   object
     2   date_added  5195 non-null   datetime64[ns]
     3   type        5837 non-null   object
    dtypes: datetime64ns, object(3)
    memory usage: 182.5+ KB

    ```

    我们能否将任何列的值转换为不同的数据类型？比如分类值？让我们使用`nunique`方法来计算每列的唯一值数量：

    ```
    In  [77] netflix.nunique()

    Out [77] title         5780
             director      3024
             date_added    1092
             type             2
             dtype: int64

    ```

    类型列是分类值的完美候选者。在一个包含 5,837 行的数据集中，它只有两个唯一值：`"Movie"`和`"TV Show"`。我们可以使用`astype`方法转换其值。请记住覆盖原始`Series`：

    ```
    In  [78] netflix["type"] = netflix["type"].astype("category")
    ```

    将数据转换为分类数据后，我们的内存使用减少了多少？惊人的 22%：

    ```
    In  [79] netflix.info()

    Out [79]

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 5837 entries, 0 to 5836
    Data columns (total 4 columns):
     #   Column      Non-Null Count  Dtype
    ---  ------      --------------  -----
     0   title       5837 non-null   object
     1   director    3936 non-null   object
     2   date_added  5195 non-null   datetime64[ns]
     3   type        5837 non-null   category
    dtypes: category(1), datetime64ns, object(2)
    memory usage: 142.8+ KB

    ```

1.  我们需要使用等号运算符来比较每个标题列的值与字符串`"Limitless"`。之后，我们可以使用布尔`Series`从`netflix`中提取返回`True`的行：

    ```
    In  [80] netflix[netflix["title"] == "Limitless"]

    Out [80]

     **title         director date_added     type**
    1559  Limitless      Neil Burger 2019-05-16    Movie
    2564  Limitless              NaN 2016-07-01  TV Show
    4579  Limitless  Vrinda Samartha 2019-10-01    Movie

    ```

1.  为了提取由 Robert Rodriguez 执导的电影，我们需要两个布尔`Series`，一个比较导演列的值与`"Robert Rodriguez"`，另一个比较类型列的值与`"Movie"`。`&`符号应用于两个布尔`Series`的`AND`逻辑：

    ```
    In  [81] directed_by_robert_rodriguez = (
                 netflix["director"] == "Robert Rodriguez"
             )
             is_movie = netflix["type"] == "Movie"
             netflix[directed_by_robert_rodriguez & is_movie]

    Out [81]

     **title          director date_added     type**
    1384    Spy Kids: All the Time in the ...  Robert Rodriguez 2019-02-19  Movie
    1416                  Spy Kids 3: Game...  Robert Rodriguez 2019-04-01  Movie
    1460  Spy Kids 2: The Island of Lost D...  Robert Rodriguez 2019-03-08  Movie
    2890                           Sin City    Robert Rodriguez 2019-10-01  Movie
    3836                             Shorts    Robert Rodriguez 2019-07-01  Movie
    3883                           Spy Kids    Robert Rodriguez 2019-04-01  Movie

    ```

1.  下一个问题要求所有标题为`"2019-07-31"`或导演为`"Robert Altman"`的标题。这个问题与上一个问题类似，但需要`|`符号进行`OR`逻辑：

    ```
    In  [82] added_on_july_31 = netflix["date_added"] == "2019-07-31"
             directed_by_altman = netflix["director"] == "Robert Altman"
             netflix[added_on_july_31 | directed_by_altman]

    Out [82]
     **title       director date_added    type**
    611                            Popeye  Robert Altman 2019-11-24   Movie
    1028        The Red Sea Diving Resort    Gideon Raff 2019-07-31   Movie
    1092                     Gosford Park  Robert Altman 2019-11-01   Movie
    3473  Bangkok Love Stories: Innocence            NaN 2019-07-31 TV Show
    5117                       Ramen Shop      Eric Khoo 2019-07-31   Movie

    ```

1.  下一个挑战要求找到导演为`"Orson Welles"`、`"Aditya"` `Kripalani`或`"`Sam Raimi"`的条目。一个选项是创建三个布尔`Series`，每个导演一个，然后使用`|`运算符。但生成布尔`Series`的更简洁和可扩展的方法是在导演列上调用`isin`方法并传入导演列表：

    ```
    In  [83] directors = ["Orson Welles", "Aditya Kripalani", "Sam Raimi"]
             target_directors = netflix["director"].isin(directors)
             netflix[target_directors]

    Out [83]

     **title          director date_added   type**
    946                 The Stranger      Orson Welles 2018-07-19  Movie
    1870                    The Gift         Sam Raimi 2019-11-20  Movie
    3706                Spider-Man 3         Sam Raimi 2019-11-01  Movie
    4243        Tikli and Laxmi Bomb  Aditya Kripalani 2018-08-01  Movie
    4475  The Other Side of the Wind      Orson Welles 2018-11-02  Movie
    5115    Tottaa Pataaka Item Maal  Aditya Kripalani 2019-06-25  Movie

    ```

1.  要找到所有`date_added`值在 2019 年 5 月 1 日和 2019 年 6 月 1 日之间的行，最简洁的方法是使用`between`方法。我们可以提供这两个日期作为下限和上限。这种方法消除了需要两个单独的布尔`Series`的需求：

    ```
    In  [84] may_movies = netflix["date_added"].between(
                 "2019-05-01", "2019-06-01"
             )

             netflix[may_movies].head()

    Out [84]

     **title      director date_added     type**
    29            Chopsticks  Sachin Yardi 2019-05-31    Movie
    60        Away From Home           NaN 2019-05-08  TV Show
    82   III Smoking Barrels    Sanjib Dey 2019-06-01    Movie
    108            Jailbirds           NaN 2019-05-10  TV Show
    124              Pegasus       Han Han 2019-05-31    Movie

    ```

1.  `dropna`方法删除具有缺失值的`DataFrame`行。我们必须包含`subset`参数以限制 pandas 应查找空值的列。对于这个问题，我们将针对导演列中的`NaN`值`:`。

    ```
    In  [85] netflix.dropna(subset = ["director"]).head()

    Out [85]

     **title        director date_added   type**
    1                      A Patch of Fog  Michael Lennox 2017-04-15  Movie
    3                           Uriyadi 2     Vijay Kumar 2019-08-02  Movie
    4                   Shrek the Musical     Jason Moore 2013-12-29  Movie
    5                    Schubert In Love     Lars Büchel 2018-03-01  Movie
    6  We Have Always Lived in the Castle   Stacie Passon 2019-09-14  Movie

    ```

1.  最后的挑战要求识别 Netflix 仅在服务中添加一部电影的日子。一个解决方案是认识到`date_added`列包含同一日添加的标题的重复日期值。我们可以使用`drop_duplicates`方法，并设置`subset`为`date_added`以及`keep`参数为`False`。Pandas 将删除`date_added`列中任何重复条目的行。结果`DataFrame`将包含在其各自日期上仅添加的标题：

    ```
    In  [86] netflix.drop_duplicates(subset = ["date_added"], keep = False)

    Out [86]

     **title         director date_added   type**
    4                      Shrek the Musical      Jason Moore 2013-12-29  Movie
    12                         Without Gorky   Cosima Spender 2017-05-31  Movie
    30            Anjelah Johnson: Not Fancy        Jay Karas 2015-10-02  Movie
    38                        One Last Thing      Tim Rouhana 2019-08-25  Movie
    70    Marvel's Iron Man & Hulk: Heroes ...      Leo Riley 2014-02-16  Movie
     ...                              ...             ...          ...      ...
    5748                             Menorca     John Barnard 2017-08-27  Movie
    5749                          Green Room  Jeremy Saulnier 2018-11-12  Movie
    5788     Chris Brown: Welcome to My Life   Andrew Sandler 2017-10-07  Movie
    5789             A Very Murray Christmas    Sofia Coppola 2015-12-04  Movie
    5812            Little Singham in London    Prakash Satam 2019-04-22  Movie

    391 rows × 4 columns

    ```

恭喜你完成了编码挑战！

## 摘要

+   `astype`方法将`Series`的值转换为另一种数据类型。

+   当一个`Series`具有少量唯一值时，`category`数据类型是理想的。

+   Pandas 可以根据一个或多个条件从`DataFrame`中提取数据子集。

+   将一个布尔`Series`放在方括号内以提取`DataFrame`的子集。

+   使用相等、不等和数学运算符将每个`Series`条目与一个常数值进行比较。

+   `&`符号强制要求满足多个条件才能提取一行。

+   `|`符号强制要求满足任一条件才能提取一行。

+   像`isnull`、`notnull`、`between`和`duplicated`这样的辅助方法返回布尔`Series`，我们可以使用它们来过滤数据集。

+   `fillna`方法用常数值替换`NaNs`。

+   `dropna`方法删除具有空值的行。我们可以自定义其参数以针对所有或某些列中的缺失值。
