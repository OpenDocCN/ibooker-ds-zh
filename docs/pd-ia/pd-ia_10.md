# 8 重塑和交叉

本章涵盖

+   比较宽格式和窄格式数据

+   从 `DataFrame` 生成交叉表

+   通过求和、平均值、计数等方式聚合值

+   堆叠和取消堆叠 `DataFrame` 索引级别

+   将 `DataFrame` 熔化

一个数据集可能以不适合我们对其进行分析的格式出现。有时，问题局限于特定的列、行或单元格。一列可能有错误的数据类型，一行可能有缺失值，或一个单元格可能有错误的字符大小写。在其他时候，数据集可能存在更大的结构问题，这些问题超出了数据本身。也许数据集存储其值的方式使得提取单行变得容易，但聚合数据变得困难。

*重塑* 数据集意味着将其操纵成不同的形状，这种形状可以讲述一个从其原始展示中无法获取的故事。重塑提供了对数据的新视角或观点。这项技能至关重要；一项研究估计，80% 的数据分析都包括清理数据和将其扭曲成正确的形状。¹

在本章中，我们将探索新的 pandas 技巧，以将数据集塑造成我们想要的形状。首先，我们将看看如何通过简洁的交叉表来总结更大的数据集。然后我们将朝相反的方向前进，学习如何拆分聚合数据集。到那时，你将成为一个能够将数据扭曲成最适合你分析展示的大师。

## 8.1 宽格式与窄格式数据

在我们深入探讨更多方法之前，让我们简要地谈谈数据集结构。数据集可以以宽格式或窄格式存储其值。一个窄数据集也被称为长数据集或高数据集。这些名称反映了当我们向数据集中添加更多值时数据集扩展的方向。宽数据集在宽度上增加；它向外扩展。窄/长/高数据集在高度上增加；它向下扩展。

看一下以下表格，它测量了两个城市在两天内的温度：

```
 **Weekday  Miami  New York**
0   Monday    100        65
1  Tuesday    105        70
```

考虑到 *变量*，即变化的测量值。有人可能会认为这个数据集中的唯一变量是星期几和温度。但还有一个额外的变量隐藏在列名中：城市。这个数据集在两列中存储了相同的变量——温度，而不是一列。迈阿密和纽约的标题并没有描述它们所存储的数据——也就是说，`100` 并不是 `Miami` 的一种类型，就像 `Monday` 是 `Weekday` 的一种类型一样。数据集通过将变量存储在列标题中隐藏了变化的城市的变量。我们可以将这个表格归类为宽数据集。宽数据集在水平方向上扩展。

假设我们为另外两个城市引入了温度测量值。我们不得不为相同的变量添加两个新列：温度。注意数据集扩展的方向。数据变宽了，而不是变高：

```
 **Weekday  Miami  New York  Chicago  San Francisco**
0   Monday    100        65       50             60
1  Tuesday    105        70       58             62
```

水平扩展是坏事吗？不一定。宽数据集非常适合查看汇总图——完整的故事。如果我们关心的是周一和周二的温度，数据集很容易阅读和理解。但宽格式也有其缺点。随着我们添加更多列，数据集变得难以处理。假设我们编写了计算所有天数平均温度的代码。现在温度存储在四个列中。如果我们添加另一个城市列，我们就必须修改我们的计算逻辑以包含它。设计变得不那么灵活。

窄数据集垂直增长。窄格式使得操作现有数据并添加新记录变得更容易。每个变量都被隔离到单个列中。比较本节中的第一个表格和以下表格：

```
 **Weekday      City  Temperature**
0   Monday     Miami          100
1   Monday  New York           65
2  Tuesday     Miami          105
3  Tuesday  New York           70
```

要包括两个更多城市的温度，我们应该添加行而不是列。数据会变得更长，而不是更宽：

```
 **Weekday           City  Temperature**
0   Monday          Miami          100
1   Monday       New York           65
2   Monday        Chicago           50
3   Monday  San Francisco           60
4  Tuesday          Miami          105
5  Tuesday       New York           70
6  Tuesday        Chicago           58
7  Tuesday  San Francisco           62
```

在周一定位城市的温度是否更容易？我会说不是，因为现在数据分散在四行中。但计算平均温度更容易，因为我们已经将温度值隔离到单个列中。随着我们添加更多行，平均计算逻辑保持不变。

数据集的最佳存储格式取决于我们试图从中获得的洞察力。Pandas 提供了将 `DataFrame`s 从窄格式转换为宽格式以及相反的工具。我们将在本章的其余部分学习如何应用这两种转换。

## 8.2 从 DataFrame 创建透视表

我们的第一份数据集，sales_by_employee.csv，是一家虚构公司的业务交易列表。每一行包括销售的日期、销售人员的姓名、客户以及交易的收入和支出：

```
In  [1] import pandas as pd

In  [2] pd.read_csv("sales_by_employee.csv").head()

Out [2]

 **Date   Name       Customer  Revenue  Expenses**
0  1/1/20  Oscar  Logistics XYZ     5250       531
1  1/1/20  Oscar    Money Corp.     4406       661
2  1/2/20  Oscar     PaperMaven     8661      1401
3  1/3/20  Oscar    PaperGenius     7075       906
4  1/4/20  Oscar    Paper Pound     2524      1767
```

为了方便起见，让我们使用 `read_csv` 函数的 `parse_dates` 参数将日期列中的字符串转换为 datetime 对象。在此更改之后，此导入看起来很好。我们可以将 `DataFrame` 赋值给 `sales` 变量：

```
In  [3] sales = pd.read_csv(
            "sales_by_employee.csv", parse_dates = ["Date"]
        )

        sales.tail()

Out [3]

 **Date   Name           Customer  Revenue  Expenses**
21 2020-01-01  Creed        Money Corp.     4430       548
22 2020-01-02  Creed  Average Paper Co.     8026      1906
23 2020-01-02  Creed  Average Paper Co.     5188      1768
24 2020-01-04  Creed         PaperMaven     3144      1314
25 2020-01-05  Creed        Money Corp.      938      1053
```

在加载数据集后，让我们探索如何使用透视表聚合其数据。

### 8.2.1 pivot_table 方法

*透视表* 通过聚合列的值并使用其他列的值对结果进行分组。*聚合* 一词描述了涉及多个值的汇总计算。示例聚合包括平均、总和、中位数和计数。Pandas 中的透视表类似于 Microsoft Excel 中的透视表功能。

像往常一样，一个例子最有帮助，所以让我们解决我们的第一个挑战。多个销售人员在同一天完成了交易。此外，相同的销售人员在同一天完成了多个交易。如果我们想按日期汇总收入并查看每个销售人员对每日总量的贡献是多少？

我们遵循四个步骤来创建透视表：

1.  选择我们想要聚合的列（s）。

1.  选择要应用到列（s）上的聚合操作。

1.  选择将分组聚合数据为类别的列（s）。

1.  确定是否将组放在行轴、列轴或两个轴上。

我们一步一步来。首先，我们需要在我们的现有`sales` `DataFrame`上调用`pivot_table`方法。该方法`index`参数接受将构成数据透视表索引标签的列。Pandas 将使用该列的唯一值来分组结果。

下一个示例使用日期列的值作为数据透视表的索引标签。日期列包含五个唯一的日期。Pandas 对其`sales`中的所有数值列（支出和收入）应用其默认的聚合操作，即平均值：

```
In  [4] sales.pivot_table(index = "Date")

Out [4]

               Expenses      Revenue
**Date** 
2020-01-01   637.500000  4293.500000
2020-01-02  1244.400000  7303.000000
2020-01-03  1313.666667  4865.833333
2020-01-04  1450.600000  3948.000000
2020-01-05  1196.250000  4834.750000
```

该方法返回一个常规的`DataFrame`对象。它可能有点令人失望，但这个 DataFrame 是一个数据透视表！该表显示了按日期列中的五个唯一日期组织的平均支出和平均收入。

我们使用`aggfunc`参数声明聚合函数；其默认参数是`"mean"`。以下代码产生与前面代码相同的结果：

```
In  [5] sales.pivot_table(index = "Date", aggfunc = "mean")

Out [5]

               Expenses      Revenue
**Date** 
2020-01-01   637.500000  4293.500000
2020-01-02  1244.400000  7303.000000
2020-01-03  1313.666667  4865.833333
2020-01-04  1450.600000  3948.000000
2020-01-05  1196.250000  4834.750000
```

我们需要修改一些方法参数以达到我们的原始目标：按销售人员组织每一天的收入总和。首先，让我们将`aggfunc`参数的参数更改为`"sum"`以在支出和收入中添加值：

```
In  [6] sales.pivot_table(index = "Date", aggfunc = "sum")

Out [6]

            Expenses  Revenue
**Date** 
2020-01-01      3825    25761
2020-01-02      6222    36515
2020-01-03      7882    29195
2020-01-04      7253    19740
2020-01-05      4785    19339
```

目前，我们只关心对收入列中的值求和。`values`参数接受 pandas 将聚合的`DataFrame`列（s）。要仅聚合一个列的值，我们可以传递一个包含列名的字符串参数：

```
In  [7] sales.pivot_table(
            index = "Date", values = "Revenue", aggfunc = "sum"
        )

Out [7]

            Revenue
**Date** 
2020-01-01    25761
2020-01-02    36515
2020-01-03    29195
2020-01-04    19740
2020-01-05    19339
```

要跨多列聚合值，我们可以将`values`传递一个列的列表。

我们有一个按日期分组的收入总和。我们的最终步骤是传达每位销售人员对每日总收入的贡献。似乎最优的展示方式是将每位销售人员的名字放在单独的列中。换句话说，我们希望使用名称列的唯一值作为数据透视表的列标题。让我们在方法调用中添加一个`columns`参数，并传递参数`"Name"`：

```
In  [8] sales.pivot_table(
            index = "Date",
            columns = "Name",
            values = "Revenue",
            aggfunc = "sum"
        )

Out [8]

Name          Creed   Dwight     Jim  Michael   Oscar
**Date** 
2020-01-01   4430.0   2639.0  1864.0   7172.0  9656.0
2020-01-02  13214.0      NaN  8278.0   6362.0  8661.0
2020-01-03      NaN  11912.0  4226.0   5982.0  7075.0
2020-01-04   3144.0      NaN  6155.0   7917.0  2524.0
2020-01-05    938.0   7771.0     NaN   7837.0  2793.0
```

就这样！我们已经按照日期和销售人员组织了收入的总和。注意数据集中存在`NaN`。`NaN`表示销售人员在某一天没有`sales`中的行，并且没有收入值。例如，Dwight 没有 2020-01-02 日期的任何`sales`行。数据透视表需要存在 2020-01-02 的索引标签，以便为那天有收入值的四位销售人员。Pandas 用`NaN`填充缺失的空白。`NaN`值的存在还迫使整数转换为浮点数。

我们可以使用`fill_value`参数将所有数据透视表中的`NaN`替换为固定值。让我们用零来填补数据空白：

```
In  [9] sales.pivot_table(
            index = "Date",
            columns = "Name",
            values = "Revenue",
            aggfunc = "sum",
            fill_value = 0
        )

Out [9]

Name        Creed  Dwight   Jim  Michael  Oscar
**Date** 
2020-01-01   4430    2639  1864     7172   9656
2020-01-02  13214       0  8278     6362   8661
2020-01-03      0   11912  4226     5982   7075
2020-01-04   3144       0  6155     7917   2524
2020-01-05    938    7771     0     7837   2793
```

我们还可能想查看每个日期和销售人员的收入小计。我们可以将`margins`参数的参数设置为`True`来为每一行和每一列添加总计：

```
In  [10] sales.pivot_table(
             index = "Date",
             columns = "Name",
             values = "Revenue",
             aggfunc = "sum",
             fill_value = 0,
             margins = True
         )

Out [10]

Name                 Creed  Dwight    Jim  Michael  Oscar     All
**Date** 
2020-01-01 00:00:00   4430    2639   1864     7172   9656   25761
2020-01-02 00:00:00  13214       0   8278     6362   8661   36515
2020-01-03 00:00:00      0   11912   4226     5982   7075   29195
2020-01-04 00:00:00   3144       0   6155     7917   2524   19740
2020-01-05 00:00:00    938    7771      0     7837   2793   19339
All                  21726   22322  20523    35270  30709  130550
```

注意，将`"All"`包含在行标签中会改变日期的可视表示，现在包括小时、分钟和秒。Pandas 需要支持日期和字符串索引标签。字符串是唯一可以表示日期或文本值的数据类型。因此，库将索引从日期的`DatetimeIndex`转换为字符串的普通`Index`。当将日期对象转换为字符串表示时，Pandas 包括时间；它还假设没有时间的日期从一天的开始。

我们可以使用`margins_name`参数来自定义小计标签。以下示例将标签从`"All"`更改为`"Total"`：

```
In  [11] sales.pivot_table(
             index = "Date",
             columns = "Name",
             values = "Revenue",
             aggfunc = "sum",
             fill_value = 0,
             margins = True,
             margins_name = "Total"
         )

Out [11]

Name                 Creed  Dwight    Jim  Michael  Oscar   Total
**Date** 
2020-01-01 00:00:00   4430    2639   1864     7172   9656   25761
2020-01-02 00:00:00  13214       0   8278     6362   8661   36515
2020-01-03 00:00:00      0   11912   4226     5982   7075   29195
2020-01-04 00:00:00   3144       0   6155     7917   2524   19740
2020-01-05 00:00:00    938    7771      0     7837   2793   19339
Total                21726   22322  20523    35270  30709  130550
```

理想情况下，Excel 用户会感到非常熟悉这些选项。

### 8.2.2 交叉表的附加选项

交叉表支持各种聚合操作。假设我们感兴趣的是每天完成的企业交易数量。我们可以将`aggfunc`的参数设置为`"count"`来计算每个日期和员工组合的`sales`行数：

```
In  [12] sales.pivot_table(
             index = "Date",
             columns = "Name",
             values = "Revenue",
             aggfunc = "count"
         )

Out [12]

Name        Creed  Dwight  Jim  Michael  Oscar
**Date** 
2020-01-01    1.0     1.0  1.0      1.0    2.0
2020-01-02    2.0     NaN  1.0      1.0    1.0
2020-01-03    NaN     3.0  1.0      1.0    1.0
2020-01-04    1.0     NaN  2.0      1.0    1.0
2020-01-05    1.0     1.0  NaN      1.0    1.0
```

再次强调，一个`NaN`值表示销售人员在某一天没有进行销售。例如，Creed 在 2020-01-03 没有完成任何销售，而 Dwight 完成了三次。以下表格列出了`aggfunc`参数的一些其他选项：

| 参数 | 描述 |
| --- | --- |
| `max` | 分组中的最大值 |
| `min` | 分组中的最小值 |
| `std` | 分组中值的标准差 |
| `median` | 分组中值的中间值（中位数） |
| `size` | 分组中的值的数量（等同于`count`） |

我们还可以向`pivot_table`函数的`aggfunc`参数传递一个聚合函数的列表。交叉表将在列轴上创建一个`MultiIndex`，并将聚合存储在其最外层级别。以下示例聚合了按日期的收入的总和以及按日期的收入计数：

```
In  [13] sales.pivot_table(
             index = "Date",
             columns = "Name",
             values = "Revenue",
             aggfunc = ["sum", "count"],
             fill_value = 0
         )

Out [13]

              sum                            count
Name        Creed Dwight   Jim Michael Oscar  Creed Dwight Jim Michael Oscar
**Date** 
2020-01-01   4430   2639  1864    7172  9656      1      1   1       1     2
2020-01-02  13214      0  8278    6362  8661      2      0   1       1     1
2020-01-03      0  11912  4226    5982  7075      0      3   1       1     1
2020-01-04   3144      0  6155    7917  2524      1      0   2       1     1
2020-01-05    938   7771     0    7837  2793      1      1   0       1     1
```

我们可以通过向`aggfunc`参数传递一个字典来对不同列应用不同的聚合操作。使用字典的键来识别`DataFrame`列，并使用值来设置聚合操作。以下示例提取了每个日期和销售人员的组合的最小收入和最大支出：

```
In  [14] sales.pivot_table(
             index = "Date",
             columns = "Name",
             values = ["Revenue", "Expenses"],
             fill_value = 0,
             aggfunc = { "Revenue": "min", "Expenses": "max" }
         )

Out [14]

      Expenses                           Revenue
Name     Creed Dwight Jim Michael Oscar  Creed  Dwight    Jim Michael Oscar
**Date** 
20...   548      368  1305    412   531   4430    2639   1864    7172  5250
20...  1768        0   462    685  1401   8026       0   8278    6362  8661
20...     0      758  1923   1772   906      0    4951   4226    5982  7075
20...  1314        0   426   1857  1767   3144       0   3868    7917  2524
20...  1053     1475     0   1633   624    938    7771      0    7837  2793
```

我们还可以通过将列的列表传递给`index`参数，在单个轴上堆叠多个分组。以下示例在行轴上聚合了按销售人员和日期的支出总和。Pandas 返回一个具有两级`MultiIndex`的`DataFrame`：

```
In  [15] sales.pivot_table(
             index = ["Name", "Date"], values = "Revenue", aggfunc = "sum"
         ).head(10)

Out [15]

                   Revenue
**Name   Date** 
Creed  2020-01-01     4430
       2020-01-02    13214
       2020-01-04     3144
       2020-01-05      938
Dwight 2020-01-01     2639
       2020-01-03    11912
       2020-01-05     7771
Jim    2020-01-01     1864
       2020-01-02     8278
       2020-01-03     4226
```

通过切换`index`列表中字符串的顺序来重新排列交叉表`MultiIndex`中的级别。以下示例交换了名称和日期的位置：

```
In  [16] sales.pivot_table(
             index = ["Date", "Name"], values = "Revenue", aggfunc = "sum"
         ).head(10)

Out [16]

                    Revenue
**Date       Name** 
2020-01-01 Creed       4430
           Dwight      2639
           Jim         1864
           Michael     7172
           Oscar       9656
2020-01-02 Creed      13214
           Jim         8278
           Michael     6362
           Oscar       8661
2020-01-03 Dwight     11912
```

交叉表首先组织和排序日期值，然后在每个日期内组织和排序名称值。

## 8.3 索引级别的堆叠和取消堆叠

这里是当前销售情况的提醒：

```
In  [17] sales.head()

Out [17]

 **Date   Name       Customer  Revenue  Expenses**
0 2020-01-01  Oscar  Logistics XYZ     5250       531
1 2020-01-01  Oscar    Money Corp.     4406       661
2 2020-01-02  Oscar     PaperMaven     8661      1401
3 2020-01-03  Oscar    PaperGenius     7075       906
4 2020-01-04  Oscar    Paper Pound     2524      1767
```

让我们将销售数据按照员工姓名和日期来组织收入。我们将日期放在列轴上，姓名放在行轴上：

```
In  [18] by_name_and_date = sales.pivot_table(
             index = "Name",
             columns = "Date",
             values = "Revenue",
             aggfunc = "sum"
         )

         by_name_and_date.head(2)

Out [18]

Date    2020-01-01  2020-01-02  2020-01-03  2020-01-04  2020-01-05
**Name** 
Creed       4430.0     13214.0         NaN      3144.0       938.0
Dwight      2639.0         NaN     11912.0         NaN      7771.0
```

有时，我们可能想要将一个索引级别从一个轴移动到另一个轴。这种变化提供了数据的不同展示方式，我们可以决定我们更喜欢哪种视图。

`stack` 方法将一个索引级别从列轴移动到行轴。接下来的示例将日期索引级别从列轴移动到行轴。Pandas 创建一个 `MultiIndex` 来存储两个行级别：姓名和日期。因为只剩下一列的值，pandas 返回一个 `Series`：

```
In  [19] by_name_and_date.stack().head(7)

Out [19]

Name    Date
Creed   2020-01-01     4430.0
        2020-01-02    13214.0
        2020-01-04     3144.0
        2020-01-05      938.0
Dwight  2020-01-01     2639.0
        2020-01-03    11912.0
        2020-01-05     7771.0
dtype: float64
```

注意到 `DataFrame` 中的 `NaN` 值在 `Series` 中不存在。Pandas 在 `by_name_and_date` 交叉表中保留了 `NaN` 值的单元格，以保持行和列的结构完整性。这个 `MultiIndex Series` 的形状允许 pandas 丢弃 `NaN` 值。

相补的 `unstack` 方法将一个索引级别从行轴移动到列轴。考虑以下交叉表，它按客户和销售人员分组收入。行轴有一个两级的 `MultiIndex`，列轴有一个常规索引：

```
In  [20] sales_by_customer = sales.pivot_table(
             index = ["Customer", "Name"],
             values = "Revenue",
             aggfunc = "sum"
         )

         sales_by_customer.head()

Out [20]

                           Revenue
**Customer          Name** 
Average Paper Co. Creed      13214
                  Jim         2287
Best Paper Co.    Dwight      2703
                  Michael    15754
Logistics XYZ     Dwight      9209
```

`unstack` 方法将行索引的最内层级别移动到列索引：

```
In  [21] sales_by_customer.unstack()

Out [21]

                   Revenue
Name                 Creed  Dwight     Jim  Michael   Oscar
**Customer** 
Average Paper Co.  13214.0     NaN  2287.0      NaN     NaN
Best Paper Co.         NaN  2703.0     NaN  15754.0     NaN
Logistics XYZ          NaN  9209.0     NaN   7172.0  5250.0
Money Corp.         5368.0     NaN  8278.0      NaN  4406.0
Paper Pound            NaN  7771.0  4226.0      NaN  5317.0
PaperGenius            NaN  2639.0  1864.0  12344.0  7075.0
PaperMaven          3144.0     NaN  3868.0      NaN  8661.0
```

在新的 `DataFrame` 中，列轴现在有一个两级的 `MultiIndex`，行轴有一个常规的一级索引。

## 8.4 熔化数据集

交叉表聚合数据集中的值。在本节中，我们将学习如何做相反的事情：将聚合的数据集合分解为非聚合的数据集合。

让我们将宽格式与窄格式框架应用于销售 `DataFrame`。这里有一个有效的策略来确定数据集是否为窄格式：导航到一行值，询问每个单元格其值是否是列标题所描述变量的单个测量值。以下是销售的第一行：

```
In  [22] sales.head(1)

Out [22]

 **Date   Name       Customer  Revenue  Expenses**
0 2020-01-01  Oscar  Logistics XYZ     5250       531
```

在前面的例子中，`"2020-01-01"` 是日期，`"Oscar"` 是姓名，`"Logistics XYZ"` 是客户，`5250` 是收入金额，`531` 是支出金额。销售 `DataFrame` 是一个窄数据集的例子。每一行的值代表一个给定变量的单个观察结果。没有变量在多列中重复。

当我们在宽格式或窄格式中操作数据时，我们经常需要在灵活性和可读性之间做出选择。我们可以将最后四列（姓名、客户、收入、支出）表示为单个类别列中的字段（以下示例），但并没有真正的益处，因为四个变量是独立且分开的。当数据以这种格式存储时，聚合数据会更困难：

```
 **Date  Category          Value**
0 2020-01-01      Name          Oscar
1 2020-01-01  Customer  Logistics XYZ
2 2020-01-01   Revenue           5250
3 2020-01-01  Expenses            531
```

下一个数据集，video_game_sales.csv，是超过 16,000 个视频游戏区域销售的列表。每一行包括游戏名称以及在美国（NA）、欧洲（EU）、日本（JP）和其他（其他）地区销售的单位数量（以百万计）：

```
In  [23] video_game_sales = pd.read_csv("video_game_sales.csv")
         video_game_sales.head()

Out [23]

 **Name     NA     EU     JP  Other**
0           Wii Sports  41.49  29.02   3.77   8.46
1    Super Mario Bros.  29.08   3.58   6.81   0.77
2       Mario Kart Wii  15.85  12.88   3.79   3.31
3    Wii Sports Resort  15.75  11.01   3.28   2.96
4  Pokemon Red/Poke...  11.27   8.89  10.22   1.00
```

再次，让我们遍历一个样本行，并询问每个单元格是否包含正确的信息。这是 video_game_sales 的第一行：

```
In  [24] video_game_sales.head(1)

Out [24]

 **Name     NA     EU    JP  Other**
0  Wii Sports  41.49  29.02  3.77   8.46
```

第一个单元格是好的；"Wii Sports"是名称的一个例子。接下来的四个单元格有问题。41.49 不是 NA 的类型或 NA 的度量。NA（北美）不是一个其值在其列中变化的变量。NA 列的实际变量数据是销售数字。NA 代表这些销售数字的区域——一个单独且不同的变量。

因此，video_game_sales 以宽格式存储其数据。四个列（NA、EU、JP 和其他）存储相同的数据点：销售数量。如果我们添加更多的区域销售列，数据集将水平增长。如果我们可以在一个共同类别中分组多个列标题，那么这是一个提示，表明数据集正在以宽格式存储其数据。

假设我们将值"NA"、"EU"、"JP"和"Other"移动到新的区域列中。将前面的表示与以下表示进行比较：

```
 **Name Region  Sales**
0  Wii Sports     NA  41.49
1  Wii Sports     EU  29.02
2  Wii Sports     JP   3.77
3  Wii Sports  Other   8.46
```

从某种意义上说，我们正在对 video_game_sales `DataFrame`进行逆透视。我们正在将数据的汇总、概览视图转换为每个列存储一个变量信息的视图。

Pandas 使用`melt`方法熔化`DataFrame`。（*熔化*是将宽数据集转换为窄数据集的过程。）该方法接受两个主要参数：

+   `id_vars`参数设置标识符列，宽数据集聚合数据的列。Name 是 video_game_sales 中的标识符列。数据集按视频游戏汇总销售。

+   `value_vars`参数接受 pandas 将熔化和存储在新列中的值所在的列（s）。

让我们从简单开始，仅熔化 NA 列的值。在下一个示例中，pandas 遍历每个 NA 列的值，并将其分配给新`DataFrame`中的单独一行。库将前一个列名（NA）存储在一个新的变量列中：

```
In  [25] video_game_sales.melt(id_vars = "Name", value_vars = "NA").head()

Out [25]

 **Name variable  value**
0                Wii Sports       NA  41.49
1         Super Mario Bros.       NA  29.08
2            Mario Kart Wii       NA  15.85
3         Wii Sports Resort       NA  15.75
4  Pokemon Red/Pokemon Blue       NA  11.27
```

接下来，让我们熔化所有四个区域销售列。下一个代码示例将`value_vars`参数传递给来自 video_game_sales 的四个区域销售列的列表：

```
In  [26] regional_sales_columns = ["NA", "EU", "JP", "Other"]

         video_game_sales.melt(
             id_vars = "Name", value_vars = regional_sales_columns
         )

Out [26]

 **Name   variable  value**
0                                            Wii Sports       NA  41.49
1                                     Super Mario Bros.       NA  29.08
2                                        Mario Kart Wii       NA  15.85
3                                     Wii Sports Resort       NA  15.75
4                              Pokemon Red/Pokemon Blue       NA  11.27
 ...                                            ...          ...    ...
66259                Woody Woodpecker in Crazy Castle 5    Other   0.00
66260                     Men in Black II: Alien Escape    Other   0.00
66261  SCORE International Baja 1000: The Official Game    Other   0.00
66262                                        Know How 2    Other   0.00
66263                                  Spirits & Spells    Other   0.00

66264 rows × 3 columns
```

`melt`方法返回一个包含 66,264 行的`DataFrame`！相比之下，video_game_sales 有 16,566 行。新的数据集是原来的四倍长，因为它为 video_games_sales 中的每一行有四行数据。数据集存储

+   每个视频游戏及其相应的北美销售数量为 16,566 行

+   每个视频游戏及其相应的欧洲销售数量为 16,566 行

+   每个视频游戏及其相应的日本销售数量为 16,566 行

+   每个视频游戏及其相应的其他销售数量为 16,566 行

变量列包含来自 video_game_sales 的四个区域列名。值列包含来自这四个区域销售列的值。在上一个输出中，数据显示视频游戏"Woodpecker in Crazy Castle 5"在 video_game_sales 的其他列中的值为`0.00`。

我们可以通过传递`var_name`和`value_name`参数的参数来自定义熔融`DataFrame`的列名。下一个示例使用“区域”作为变量列和“销售额”作为值列：

```
In  [27] video_game_sales_by_region = video_game_sales.melt(
             id_vars = "Name",
             value_vars = regional_sales_columns,
             var_name = "Region",
             value_name = "Sales"
         )

         video_game_sales_by_region.head()

Out [27]

 **Name Region  Sales**
0                Wii Sports     NA  41.49
1         Super Mario Bros.     NA  29.08
2            Mario Kart Wii     NA  15.85
3         Wii Sports Resort     NA  15.75
4  Pokemon Red/Pokemon Blue     NA  11.27
```

窄数据比宽数据更容易聚合。假设我们想要找出所有地区每款视频游戏销售额的总和。给定熔融数据集，我们可以使用`pivot_table`方法通过几行代码来完成这个任务：

```
In  [28] video_game_sales_by_region.pivot_table(
             index = "Name", values = "Sales", aggfunc = "sum"
         ).head()

Out [28]

                               Sales
**Name** 
'98 Koshien                     0.40
.hack//G.U. Vol.1//Rebirth      0.17
.hack//G.U. Vol.2//Reminisce    0.23
.hack//G.U. Vol.3//Redemption   0.17
.hack//Infection Part 1         1.26
```

数据集的窄形状简化了转置过程。

## 8.5 爆炸值列表

有时，数据集会在同一单元格中存储多个值。我们可能想要拆分数据簇，以便每行只存储一个值。考虑 recipes.csv，这是一个包含三个菜谱的集合，每个菜谱都有一个名称和成分列表。成分存储在一个逗号分隔的字符串中：

```
In  [29] recipes = pd.read_csv("recipes.csv")
         recipes

Out [29]

 **Recipe                              Ingredients**
0   Cashew Crusted Chicken  Apricot preserves, Dijon mustard, cu...
1      Tomato Basil Salmon  Salmon filets, basil, tomato, olive ...
2  Parmesan Cheese Chicken  Bread crumbs, Parmesan cheese, Itali...
```

你还记得我们在第六章中介绍的`str.split`方法吗？此方法使用分隔符将字符串拆分为子字符串。我们可以通过逗号的存在来拆分每个“成分”字符串。在下一个示例中，pandas 返回一个列表的`Series`。每个列表存储该行的成分：

```
In  [30] recipes["Ingredients"].str.split(",")

Out [30]

0    [Apricot preserves,  Dijon mustard,  curry pow...
1    [Salmon filets,  basil,  tomato,  olive oil,  ...
2    [Bread crumbs,  Parmesan cheese,  Italian seas...
Name: Ingredients, dtype: object
```

让我们用新的列覆盖原始的“成分”列：

```
In  [31] recipes["Ingredients"] = recipes["Ingredients"].str.split(",")
         recipes

Out [31]

 **Recipe                              Ingredients**
0   Cashew Crusted Chicken  [Apricot preserves,  Dijon mustard, ...
1      Tomato Basil Salmon  [Salmon filets,  basil,  tomato,  ol...
2  Parmesan Cheese Chicken  [Bread crumbs,  Parmesan cheese,  It...
```

现在，我们如何将每个列表的值分散到多行中？`explode`方法为`Series`中的每个列表元素创建一个单独的行。我们在`DataFrame`上调用此方法，并传入包含列表的列：

```
In  [32] recipes.explode("Ingredients")

Out [32]

 **Recipe         Ingredients**
0  Cashew Crusted Chicken   Apricot preserves
0  Cashew Crusted Chicken       Dijon mustard
0  Cashew Crusted Chicken        curry powder
0  Cashew Crusted Chicken     chicken breasts
0  Cashew Crusted Chicken             cashews
1     Tomato Basil Salmon       Salmon filets
1     Tomato Basil Salmon               basil
1     Tomato Basil Salmon              tomato
1     Tomato Basil Salmon           olive oil
1     Tomato Basil Salmon     Parmesan cheese
2  Simply Parmesan Cheese        Bread crumbs
2  Simply Parmesan Cheese     Parmesan cheese
2  Simply Parmesan Cheese   Italian seasoning
2  Simply Parmesan Cheese                 egg
2  Simply Parmesan Cheese     chicken breasts
```

真棒！我们已经将每个成分隔离到单独的一行。请注意，`explode`方法需要一个列表的`Series`才能正常工作。

## 8.6 编码挑战

这是一个练习本章中介绍的重塑、转置和熔融概念的机会。

### 8.6.1 问题

我们为你提供了两个数据集进行操作。used_cars.csv 文件是 Craigslist 分类网站上出售的二手车的列表。每一行包括汽车的制造商、生产年份、燃料类型、传动类型和价格：

```
In  [33] cars = pd.read_csv("used_cars.csv")
         cars.head()

Out [33]

 **Manufacturer  Year Fuel Transmission  Price**
0        Acura  2012  Gas    Automatic  10299
1       Jaguar  2011  Gas    Automatic   9500
2        Honda  2004  Gas    Automatic   3995
3    Chevrolet  2016  Gas    Automatic  41988
4          Kia  2015  Gas    Automatic  12995
```

minimum_wage.csv 数据集是美国最低工资的集合。数据集有一个“州”列和多个年份列：

```
In  [34] min_wage = pd.read_csv("minimum_wage.csv")
         min_wage.head()

Out [34]

 **State  2010  2011  2012  2013  2014  2015   2016   2017**
0     Alabama  0.00  0.00  0.00  0.00  0.00  0.00   0.00   0.00
1      Alaska  8.90  8.63  8.45  8.33  8.20  9.24  10.17  10.01
2     Arizona  8.33  8.18  8.34  8.38  8.36  8.50   8.40  10.22
3    Arkansas  7.18  6.96  6.82  6.72  6.61  7.92   8.35   8.68
4  California  9.19  8.91  8.72  8.60  9.52  9.51  10.43  10.22
```

这里有一些挑战：

1.  对 cars 中的汽车价格总和进行汇总。按燃料类型在行轴上对结果进行分组。

1.  对 cars 中的汽车数量进行汇总。按索引轴上的制造商和列轴上的传动类型对结果进行分组。显示行和列的子总计。

1.  对 cars 中的汽车价格的平均值进行汇总。按年份和燃料类型在索引轴上，按传动类型在列轴上对结果进行分组。

1.  给定前一个挑战中的`DataFrame`，将传输级别从列轴移动到行轴。

1.  将`min_wage`从宽格式转换为窄格式。换句话说，将数据从八个年份的列（2010-17）移动到单个列。

### 8.6.2 解决方案

让我们逐一解决这些问题：

1.  `pivot_table` 方法是添加价格列中的值并按燃料类型组织总计的最佳解决方案。我们可以使用方法的 `index` 参数设置数据透视表的索引标签；我们将传递一个 `"Fuel"` 的参数。我们将指定聚合操作为 `"sum"`，使用 `aggfunc` 参数：

    ```
    In  [35] cars.pivot_table(
                 values = "Price", index = "Fuel", aggfunc = "sum"
             )

    Out [35]

                    Price
    **Fuel** 
    Diesel      986177143
    Electric     18502957
    Gas       86203853926
    Hybrid       44926064
    Other       242096286
    ```

1.  我们还可以使用 `pivot_table` 方法按制造商和传动类型计数汽车。我们将使用 `columns` 参数设置传动列的值作为数据透视表的列标签。记得传递 `margins` 参数一个 `True` 的参数以显示行和列的小计：

    ```
    In  [36] cars.pivot_table(
                 values = "Price",
                 index = "Manufacturer",
                 columns = "Transmission",
                 aggfunc = "count",
                 margins = True
             ).tail()

    Out [36]

    Transmission  Automatic   Manual    Other     All
    **Manufacturer** 
    Tesla             179.0      NaN     59.0     238
    Toyota          31480.0   1367.0   2134.0   34981
    Volkswagen       7985.0   1286.0    236.0    9507
    Volvo            2665.0    155.0     50.0    2870
    All            398428.0  21005.0  21738.0  441171
    ```

1.  要按年份和燃料类型在数据透视表的行轴上组织平均汽车价格，我们可以将一个字符串列表传递给 `pivot_table` 函数的 `index` 参数：

    ```
       In  [37] cars.pivot_table(
                    values = "Price",
                    index = ["Year", "Fuel"],
                    columns = ["Transmission"],
                    aggfunc = "mean"
                )

    Out [37]

    Transmission      Automatic        Manual         Other
    **Year Fuel** 
    2000 Diesel    11326.176962  14010.164021  11075.000000
         Electric   1500.000000           NaN           NaN
         Gas        4314.675996   6226.140327   3203.538462
         Hybrid     2600.000000   2400.000000           NaN
         Other     16014.918919  11361.952381  12984.642857
     ...  ...          ...           ...           ...
    2020 Diesel    63272.595930      1.000000   1234.000000
         Electric   8015.166667   2200.000000  20247.500000
         Gas       34925.857933  36007.270833  20971.045455
         Hybrid    35753.200000           NaN   1234.000000
         Other     22210.306452           NaN   2725.925926

    102 rows × 3 columns
    ```

    让我们将之前的透视表分配给 `report` 变量以进行下一个挑战：

    ```
    In  [38] report = cars.pivot_table(
                 values = "Price",
                 index = ["Year", "Fuel"],
                 columns = ["Transmission"],
                 aggfunc = "mean"
             )
    ```

1.  下一个练习是将传动类型从列索引移动到行索引。这里 `stack` 方法派上用场。该方法返回一个 `MultiIndex` `Series`。`Series` 有三个级别：年份、燃料和新增的传动：

    ```
    In  [39] report.stack()

    Out [39]

    Year  Fuel      Transmission
    2000  Diesel    Automatic       11326.176962
                    Manual          14010.164021
                    Other           11075.000000
          Electric  Automatic        1500.000000
          Gas       Automatic        4314.675996
                                        ...
    2020  Gas       Other           20971.045455
          Hybrid    Automatic       35753.200000
                    Other            1234.000000
          Other     Automatic       22210.306452
                    Other            2725.925926
    Length: 274, dtype: float64 
    ```

1.  接下来，我们希望将 `min_wage` 数据集从宽格式转换为窄格式。八个列存储相同的变量：工资本身。解决方案是 `melt` 方法。我们可以将 State 列声明为标识列，将八个年份列声明为变量列：

    ```
    In  [40] year_columns = [
                 "2010", "2011", "2012", "2013",
                 "2014", "2015", "2016", "2017"
             ]

             min_wage.melt(id_vars = "State", value_vars = year_columns)

    Out [40]

     **State variable  value**
    0          Alabama     2010   0.00
    1           Alaska     2010   8.90
    2          Arizona     2010   8.33
    3         Arkansas     2010   7.18
    4       California     2010   9.19
     ...        ...        ...     ...
    435       Virginia     2017   7.41
    436     Washington     2017  11.24
    437  West Virginia     2017   8.94
    438      Wisconsin     2017   7.41
    439        Wyoming     2017   5.26

    440 rows × 3 columns
    ```

    这里有一个额外的技巧：我们可以在调用 `melt` 方法时移除 `value_vars` 参数，仍然得到相同的 `DataFrame`。默认情况下，pandas 从除了我们传递给 `id_vars` 参数的列之外的所有列熔化数据：

    ```
    In  [41] min_wage.melt(id_vars = "State")

    Out [41]

     **State variable  value**
    0          Alabama     2010   0.00
    1           Alaska     2010   8.90
    2          Arizona     2010   8.33
    3         Arkansas     2010   7.18
    4       California     2010   9.19
     ...        ...        ...     ...
    435       Virginia     2017   7.41
    436     Washington     2017  11.24
    437  West Virginia     2017   8.94
    438      Wisconsin     2017   7.41
    439        Wyoming     2017   5.26

    440 rows × 3 columns
    ```

    我们还可以使用 `var_name` 和 `value_name` 参数自定义列名。下一个示例使用 `"Year"` 和 `"Wage"` 来更好地解释每一列代表的内容：

    ```
    In  [42] min_wage.melt(
                 id_vars = "State", var_name = "Year", value_name = "Wage"
             )

    Out [42]

     **State  Year   Wage**
    0          Alabama  2010   0.00
    1           Alaska  2010   8.90
    2          Arizona  2010   8.33
    3         Arkansas  2010   7.18
    4       California  2010   9.19
     ...         ...     ...    ...
    435       Virginia  2017   7.41
    436     Washington  2017  11.24
    437  West Virginia  2017   8.94
    438      Wisconsin  2017   7.41
    439        Wyoming  2017   5.26

    440 rows × 3 columns
    ```

恭喜你完成了编码挑战！

## 摘要

+   `pivot_table` 方法聚合 `DataFrame` 的数据。

+   数据透视表的聚合包括求和、计数和平均值。

+   我们可以自定义数据透视表的行标签和列标签。

+   我们可以使用一个或多个列的值作为数据透视表的索引标签。

+   `stack` 方法将一个索引级别从列索引移动到行索引。

+   `unstack` 方法将一个索引级别从行索引移动到列索引。

+   `melt` 方法通过将数据分布到各个单独的行来“取消透视”一个汇总表。这个过程将宽数据集转换为窄数据集。

+   `explode` 方法为列表中的每个元素创建一个单独的行条目；它需要一个列表的 `Series`。

* * *

¹ 参见 Hadley Wickham 的文章“Tidy Data”，发表在《统计软件杂志》上，[`vita.had.co.nz/papers/tidy-data.pdf`](https://vita.had.co.nz/papers/tidy-data.pdf)。
