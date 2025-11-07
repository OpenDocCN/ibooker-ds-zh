# 3 个 `Series` 方法

本章涵盖

+   使用 `read_csv` 函数导入 CSV 数据集

+   按升序和降序排序 `Series` 值

+   在 `Series` 中检索最大和最小值

+   在 `Series` 中计数唯一值的出现次数

+   在 `Series` 的每个值上调用函数

在第二章中，我们开始探索 `Series` 对象，这是一个一维的、带有同质值的标签数组。我们从不同的来源填充了我们的 `Series`，包括列表、字典和 NumPy `ndarrays`。我们观察了 pandas 如何为每个 `Series` 值分配一个索引标签和一个索引位置。我们学习了如何对 `Series` 应用数学运算。

在掌握基础知识后，我们准备探索一些真实世界的数据集！在本章中，我们将介绍许多高级 `Series` 操作，包括排序、计数和分桶。我们还将开始看到这些方法如何帮助我们从数据中得出见解。让我们深入探讨。

## 3.1 使用 `read_csv` 函数导入数据集

*CSV* 是一个纯文本文件，它使用换行符分隔每行数据，使用逗号分隔每行值。文件的第一行包含数据的列标题。本章为我们提供了三个 CSV 文件来操作：

+   *pokemon.csv*—一个包含超过 800 种宝可梦的列表，这些宝可梦是任天堂流行媒体系列的卡通怪物。每种宝可梦都关联一个或多个类型，例如火、水、草。

+   *google_stock.csv*—从 2004 年 8 月上市到 2019 年 10 月的谷歌科技公司每日美元股价集合。

+   *revolutionary_war.csv*—美国独立战争期间战斗的记录。每次小冲突都与一个开始日期和一个美国州相关联。

让我们先导入数据集。在继续的过程中，我们将讨论我们可以进行的优化，以简化分析。 

我们的第一步是启动一个新的 Jupyter Notebook 并导入 pandas 库。确保在 CSV 文件所在的目录中创建笔记本：

```
In [1] import pandas as pd
```

Pandas 有十几个导入函数来加载各种文件格式。这些函数在库的最高级别可用，并以前缀 `read.` 开头。在我们的情况下，要导入 CSV，我们想要 `read_csv` 函数。该函数的第一个参数 `filepath_or_buffer` 期望一个包含文件名的字符串。确保该字符串包含 .csv 扩展名（例如 `"pokemon.csv"`，而不是 `"pokemon"`）。默认情况下，pandas 在笔记本所在的目录中查找文件：

```
In  [2] # The two lines below are equivalent
        pd.read_csv(filepath_or_buffer = "pokemon.csv")
        pd.read_csv("pokemon.csv")

Out [2]

 **Pokemon                Type**
0       Bulbasaur      Grass / Poison
1         Ivysaur      Grass / Poison
2        Venusaur      Grass / Poison
3      Charmander                Fire
4      Charmeleon                Fire
...           ...                 ...
804     Stakataka        Rock / Steel
805   Blacephalon        Fire / Ghost
806       Zeraora            Electric
807        Meltan               Steel
808      Melmetal               Steel

809 rows × 2 columns
```

无论数据集中有多少列，`read_csv` 函数总是将数据导入一个 `DataFrame`，这是一个支持多行多列的二维 pandas 数据结构。我们将在第四章中详细介绍这个对象。使用 `DataFrame` 没有问题，但我们想更多地练习 `Series`，所以让我们将 CSV 的数据存储在较小的数据结构中。

我们遇到的第一问题是数据集有两个列（宝可梦和类型），但 `Series` 只支持一列数据。一个简单的解决方案是将数据集的一个列设置为 `Series` 索引。我们可以使用 `index_col` 参数来设置索引列。请注意大小写敏感性：字符串必须与数据集中的标题匹配。让我们将 `"Pokemon"` 作为 `index_col` 的参数传递：

```
In  [3] pd.read_csv("pokemon.csv", index_col = "Pokemon")

Out [3]

                       Type
**Pokemon** 
Bulbasaur    Grass / Poison
Ivysaur      Grass / Poison
Venusaur     Grass / Poison
Charmander             Fire
Charmeleon             Fire
     ...                ...
Stakataka      Rock / Steel
Blacephalon    Fire / Ghost
Zeraora            Electric
Meltan                Steel
Melmetal              Steel

809 rows × 1 columns
```

我们已经成功将 Pokemon 列设置为 `Series` 索引，但 pandas 仍然默认将数据导入到 `DataFrame` 中。毕竟，一个能够容纳多列数据的容器在技术上也可以容纳一列数据。要强制 pandas 使用 `Series`，我们需要添加另一个名为 `squeeze` 的参数，并传递一个值为 `True` 的参数。`squeeze` 参数将一列 `DataFrame` 强制转换为 `Series`：

```
In  [4] pd.read_csv("pokemon.csv", index_col = "Pokemon", squeeze = True)

Out [4] Pokemon
        Bulbasaur      Grass / Poison
        Ivysaur        Grass / Poison
        Venusaur       Grass / Poison
        Charmander               Fire
        Charmeleon               Fire
                            ...
        Stakataka        Rock / Steel
        Blacephalon      Fire / Ghost
        Zeraora              Electric
        Meltan                  Steel
        Melmetal                Steel
        Name: Type, Length: 809, dtype: object
```

我们正式拥有了一个 `Series`。太好了！索引标签是宝可梦的名字，值是宝可梦的类型。

值下面的输出揭示了某些重要细节：

+   Pandas 将 `Series` 命名为 Type，这是 CSV 文件中的列名。

+   `Series` 有 809 个值。

+   `dtype:` `object` 告诉我们这是一个字符串值的 `Series`。`object` 是 pandas 对字符串和更复杂数据结构的内部术语。

最后一步是将 `Series` 赋值给一个变量。在这里 `pokemon` 感觉很合适：

```
In  [5] pokemon = pd.read_csv(
            "pokemon.csv", index_col = "Pokemon", squeeze = True
        )
```

剩下的两个数据集有一些额外的复杂性。让我们看一下 google_stock.csv：

```
In  [6] pd.read_csv("google_stocks.csv").head()

Out [6]

 **Date  Close**
0  2004-08-19  49.98
1  2004-08-20  53.95
2  2004-08-23  54.50
3  2004-08-24  52.24
4  2004-08-25  52.80
```

当导入数据集时，pandas 会推断每个列最适合的数据类型。有时，库会采取保守的做法，避免对我们的数据进行假设。例如，google_stocks.csv 包含一个日期列，其值为 YYYY-MM-DD 格式的日期时间（例如 2010-08-04）。除非我们告诉 pandas 将这些值视为日期时间，否则库默认将它们导入为字符串。字符串是一种更通用和灵活的数据类型；它可以表示任何值。

让我们明确告诉 pandas 将日期列中的值转换为日期时间。虽然我们不会在第十一章介绍日期时间，但将每列数据存储在最准确的数据类型中被认为是最佳实践。当 pandas 知道它有日期时间时，它将启用在普通字符串上不可用的额外方法，例如计算日期的星期几。

`read_csv` 函数的 `parse_dates` 参数接受一个字符串列表，表示 pandas 应将其文本值转换为日期时间的列。下一个示例传递了一个包含 `"Date"` 的列表：

```
In  [7] pd.read_csv("google_stocks.csv", parse_dates = ["Date"]).head()

Out [7]

 **Date  Close**
0  2004-08-19  49.98
1  2004-08-20  53.95
2  2004-08-23  54.50
3  2004-08-24  52.24
4  2004-08-25  52.80
```

输出中没有视觉差异，但 pandas 在底层为日期列存储了不同的数据类型。让我们使用 `index_col` 参数将日期列设置为 `Series` 索引；`Series` 与日期索引配合得很好。最后，让我们添加 `squeeze` 参数来强制使用 `Series` 对象而不是 `DataFrame`：

```
In  [8] pd.read_csv(
            "google_stocks.csv",
            parse_dates = ["Date"],
            index_col = "Date",
            squeeze = True
        ).head()

Out [8] Date
        2004-08-19    49.98
        2004-08-20    53.95
        2004-08-23    54.50
        2004-08-24    52.24
        2004-08-25    52.80
        Name: Close, dtype: float64
```

看起来不错。我们有一个由日期时间索引标签和浮点值组成的`Series`。让我们将这个`Series`保存到`google`变量中：

```
In  [9] google = pd.read_csv(
            "google_stocks.csv",
            parse_dates = ["Date"],
            index_col = "Date",
            squeeze = True
        )
```

我们还有一个数据集要导入：美国独立战争战役。这次，让我们在导入时预览最后五行。我们将`tail`方法链接到`read_csv`函数返回的`DataFrame`：

```
In  [10] pd.read_csv("revolutionary_war.csv").tail()

Out [10]

 **Battle  Start Date     State**
227         Siege of Fort Henry   9/11/1782  Virginia
228  Grand Assault on Gibraltar   9/13/1782       NaN
229   Action of 18 October 1782  10/18/1782       NaN
230   Action of 6 December 1782   12/6/1782       NaN
231   Action of 22 January 1783   1/22/1783  Virginia
```

看一下“州”列。哎呀——这个数据集有一些缺失值。提醒一下，pandas 使用`NaN`（不是一个数字）来标记缺失值。`NaN`是一个 NumPy 对象，用于表示无或值的缺失。这个数据集包含没有确定开始日期的战役或在美国领土外作战的战役的缺失/不存在值。

让我们将“开始日期”列设置为索引。我们再次使用`index_col`参数来设置索引，并使用`parse_dates`参数将开始日期字符串转换为日期时间值。Pandas 可以识别这个数据集的日期格式（M/D/YYYY）：

```
In  [11] pd.read_csv(
             "revolutionary_war.csv",
             index_col = "Start Date",
             parse_dates = ["Start Date"],
         ).tail()

Out [11]

                                Battle     State
**Start Date** 
1782-09-11         Siege of Fort Henry  Virginia
1782-09-13  Grand Assault on Gibraltar       NaN
1782-10-18   Action of 18 October 1782       NaN
1782-12-06   Action of 6 December 1782       NaN
1783-01-22   Action of 22 January 1783  Virginia
```

默认情况下，`read_csv`函数从 CSV 文件中导入所有列。如果我们想得到一个`Series`，我们必须限制导入到两列：一列作为索引，另一列作为值。在这种情况下，`squeeze`参数本身是不够的；如果有多于一列的数据，pandas 将忽略该参数。

`read_csv`函数的`usecols`参数接受一个列列表，pandas 应该导入这些列。让我们只包括开始日期和州：

```
In  [12] pd.read_csv(
             "revolutionary_war.csv",
             index_col = "Start Date",
             parse_dates = ["Start Date"],
             usecols = ["State", "Start Date"],
             squeeze = True
         ).tail()

Out [12] Start Date
         1782-09-11    Virginia
         1782-09-13         NaN
         1782-10-18         NaN
         1782-12-06         NaN
         1783-01-22    Virginia
         Name: State, dtype: object
```

完美！我们有一个由日期时间索引和字符串值组成的`Series`。让我们将其分配给一个`battles`变量：

```
In  [13] battles = pd.read_csv(
             "revolutionary_war.csv",
             index_col = "Start Date",
             parse_dates = ["Start Date"],
             usecols = ["State", "Start Date"],
             squeeze = True
         )
```

现在我们已经将数据集导入到`Series`对象中，让我们看看我们可以用它们做什么。

## 3.2 对 Series 进行排序

我们可以按值或索引对`Series`进行排序，按升序或降序排序。

### 3.2.1 使用`sort_values`方法按值排序

假设我们想知道谷歌公司最低和最高的股价。`sort_values`方法返回一个新`Series`，其中的值按升序排序。*升序*意味着大小增加——换句话说，从小到大。索引标签与其值对应移动：

```
In  [14] google.sort_values()

Out [14] Date
         2004-09-03      49.82
         2004-09-01      49.94
         2004-08-19      49.98
         2004-09-02      50.57
         2004-09-07      50.60
                        ...
         2019-04-23    1264.55
         2019-10-25    1265.13
         2018-07-26    1268.33
         2019-04-26    1272.18
         2019-04-29    1287.58
         Name: Close, Length: 3824, dtype: float64
```

Pandas 按字母顺序对字符串`Series`进行排序。*升序*意味着从字母表的开始到结束：

```
In  [15] pokemon.sort_values()

Out [15] Pokemon
         Illumise                Bug
         Silcoon                 Bug
         Pinsir                  Bug
         Burmy                   Bug
         Wurmple                 Bug
                           ...
         Tirtouga       Water / Rock
         Relicanth      Water / Rock
         Corsola        Water / Rock
         Carracosta     Water / Rock
         Empoleon      Water / Steel
         Name: Type, Length: 809, dtype: object
```

Pandas 在排序时将大写字母排在小写字母之前。因此，大写字母`"Z"`在小写字母`"a"`之前。在下一个例子中，请注意字符串`"adam"`出现在`"Ben"`之后：

```
In  [16] pd.Series(data = ["Adam", "adam", "Ben"]).sort_values()

Out [16] 0    Adam
         2     Ben
         1    adam
         dtype: object
```

`ascending`参数设置排序顺序，默认参数为`True`。要按降序（从大到小）排序`Series`值，将参数传递一个`False`的参数：

```
In  [17] google.sort_values(ascending = False).head()

Out [17] Date
         2019-04-29    1287.58
         2019-04-26    1272.18
         2018-07-26    1268.33
         2019-10-25    1265.13
         2019-04-23    1264.55
         Name: Close, dtype: float64
```

降序排序将字符串`Series`按逆字母顺序排列。*降序*意味着从字母表的末尾到开头：

```
In  [18] pokemon.sort_values(ascending = False).head()

Out [18] Pokemon
         Empoleon      Water / Steel
         Carracosta     Water / Rock
         Corsola        Water / Rock
         Relicanth      Water / Rock
         Tirtouga       Water / Rock
         Name: Type, dtype: object
```

`na_position` 参数配置返回的 `Series` 中 `NaN` 值的位置，其默认参数为 `"last"`。默认情况下，pandas 将缺失值放置在排序后的 `Series` 的末尾：

```
In  [19] # The two lines below are equivalent
         battles.sort_values()
         battles.sort_values(na_position = "last")

Out [19] Start Date
         1781-09-06    Connecticut
         1779-07-05    Connecticut
         1777-04-27    Connecticut
         1777-09-03       Delaware
         1777-05-17        Florida
                          ...
         1782-08-08            NaN
         1782-08-25            NaN
         1782-09-13            NaN
         1782-10-18            NaN
         1782-12-06            NaN
         Name: State, Length: 232, dtype: object
```

要首先显示缺失值，可以将 `na_position` 参数的值设置为 `"first"`。结果 `Series` 首先显示所有 `NaN`，然后是排序后的值：

```
In  [20] battles.sort_values(na_position = "first")

Out [20] Start Date
         1775-09-17         NaN
         1775-12-31         NaN
         1776-03-03         NaN
         1776-03-25         NaN
         1776-05-18         NaN
                         ...
         1781-07-06    Virginia
         1781-07-01    Virginia
         1781-06-26    Virginia
         1781-04-25    Virginia
         1783-01-22    Virginia
         Name: State, Length: 232, dtype: object
```

如果我们想移除 `NaN` 值呢？`dropna` 方法返回一个没有缺失值的 `Series`。注意，该方法仅针对 `Series` 中的 `NaN` 值，而不是索引。下一个示例过滤出具有当前位置的战斗：

```
In  [21] battles.dropna().sort_values()

Out [21] Start Date
         1781-09-06    Connecticut
         1779-07-05    Connecticut
         1777-04-27    Connecticut
         1777-09-03       Delaware
         1777-05-17        Florida
                             ...
         1782-08-19       Virginia
         1781-03-16       Virginia
         1781-04-25       Virginia
         1778-09-07       Virginia
         1783-01-22       Virginia
         Name: State, Length: 162, dtype: object
```

之前的 `Series` 比预期的 `battles` 要短。Pandas 从 `battles` 中移除了 70 个 `NaN` 值。

### 3.2.2 使用 sort_index 方法按索引排序

有时候，我们的关注点可能在于索引而不是值。幸运的是，我们可以使用 `sort_index` 方法按索引对 `Series` 进行排序。使用此选项，值会与其索引对应项一起移动。与 `sort_values` 类似，`sort_index` 也接受一个 `ascending` 参数，其默认参数也是 `True`：

```
In  [22] # The two lines below are equivalent
         pokemon.sort_index()
         pokemon.sort_index(ascending = True)

Out [22] Pokemon
         Abomasnow        Grass / Ice
         Abra                 Psychic
         Absol                   Dark
         Accelgor                 Bug
         Aegislash      Steel / Ghost
                           ...
         Zoroark                 Dark
         Zorua                   Dark
         Zubat        Poison / Flying
         Zweilous       Dark / Dragon
         Zygarde      Dragon / Ground
         Name: Type, Length: 809, dtype: object
```

当按升序对日期时间集合进行排序时，pandas 从最早的日期排序到最新的日期。`battles` `Series` 提供了一个很好的机会来观察这个排序的实际应用：

```
In  [23] battles.sort_index()

Out [23] Start Date
         1774-09-01    Massachusetts
         1774-12-14    New Hampshire
         1775-04-19    Massachusetts
         1775-04-19    Massachusetts
         1775-04-20         Virginia
                           ...
         1783-01-22         Virginia
         NaT              New Jersey
         NaT                Virginia
         NaT                     NaN
         NaT                     NaN
         Name: State, Length: 232, dtype: object
```

在排序后的 `Series` 的末尾，我们看到了一种新的值类型。Pandas 使用另一个 NumPy 对象 `NaT`（代表 not a time）来代替缺失的日期值。`NaT` 对象与索引的日期时间类型保持数据完整性。

`sort_index` 方法还包括 `na_position` 参数，用于改变 `NaN` 值的位置。下一个示例首先显示缺失值，然后是排序后的日期时间：

```
In  [24] battles.sort_index(na_position = "first").head()

Out [24] Start Date
         NaT              New Jersey
         NaT                Virginia
         NaT                     NaN
         NaT                     NaN
         1774-09-01    Massachusetts
         Name: State, dtype: object
```

要按降序排序，我们可以将 `ascending` 参数的值设置为 `False`。降序排序显示从最新到最早的日期：

```
In  [25] battles.sort_index(ascending = False).head()

Out [25] Start Date
         1783-01-22    Virginia
         1782-12-06         NaN
         1782-10-18         NaN
         1782-09-13         NaN
         1782-09-11    Virginia
         Name: State, dtype: object
```

数据集最早的战斗发生在 1783 年 1 月 22 日，在弗吉尼亚州。

### 3.2.3 使用 nsmallest 和 nlargest 方法检索最小和最大值

假设我们想找到谷歌股票表现最佳的五个日期。一个选项是将 `Series` 按降序排序，然后限制结果为前五行：

```
In  [26] google.sort_values(ascending = False).head()

Out [26] Date
         2019-04-29    1287.58
         2019-04-26    1272.18
         2018-07-26    1268.33
         2019-10-25    1265.13
         2019-04-23    1264.55
         Name: Close, dtype: float64
```

这个操作相当常见，因此 pandas 提供了一个辅助方法来节省我们一些字符。`nlargest` 方法返回 `Series` 中的最大值。它的第一个参数 `n` 设置要返回的记录数。`n` 参数的默认参数为 `5`。Pandas 在返回的 `Series` 中按降序排序值：

```
In  [27] # The two lines below are equivalent
         google.nlargest(n = 5)
         google.nlargest()

Out [27] Date
         2019-04-29    1287.58
         2019-04-26    1272.18
         2018-07-26    1268.33
         2019-10-25    1265.13
         2019-04-23    1264.55
         Name: Close, dtype: float64
```

补充的 `nsmallest` 方法返回 `Series` 中的最小值，并按升序排序。它的 `n` 参数也有一个默认参数 `5`：

```
In  [28] # The two lines below are equivalent
         google.nsmallest(n = 5)
         google.nsmallest(5)

Out [28] Date
         2004-09-03    49.82
         2004-09-01    49.94
         2004-08-19    49.98
         2004-09-02    50.57
         2004-09-07    50.60
         2004-08-30    50.81
         Name: Close, dtype: float64
```

注意，这两个方法都不适用于字符串 `Series`。

## 3.3 使用 inplace 参数覆盖 Series

我们在本章中调用的所有方法都会返回新的 `Series` 对象。我们用 `pokemon`、`google` 和 `battles` 变量引用的原始 `Series` 对象在我们的操作过程中始终保持未受影响。作为一个例子，让我们观察方法调用前后的 `battles`；`Series` 并没有改变：

```
In  [29] battles.head(3)

Out [29] Start Date
         1774-09-01    Massachusetts
         1774-12-14    New Hampshire
         1775-04-19    Massachusetts
         Name: State, dtype: object

In  [30] battles.sort_values().head(3)

Out [30] Start Date
         1781-09-06    Connecticut
         1779-07-05    Connecticut
         1777-04-27    Connecticut
         Name: State, dtype: object

In  [31] battles.head(3)

Out [31] Start Date
         1774-09-01    Massachusetts
         1774-12-14    New Hampshire
         1775-04-19    Massachusetts
         Name: State, dtype: object
```

如果我们想修改 `battles` `Series` 呢？pandas 中的许多方法包括一个 `inplace` 参数，当传递 `True` 作为参数时，它似乎会修改被调用的对象。

将上一个例子与下一个例子进行比较。在这里，我们再次调用 `sort_values` 方法，但这次我们传递 `True` 作为 `inplace` 参数的参数。如果我们使用 `inplace`，该方法将返回 `None`，导致 Jupyter Notebook 中没有输出。当我们输出 `battles` 时，我们可以看到它已经改变了：

```
In  [32] battles.head(3)

Out [32] Start Date
         1774-09-01    Massachusetts
         1774-12-14    New Hampshire
         1775-04-19    Massachusetts
         Name: State, dtype: object
In  [33] battles.sort_values(inplace = True)

In  [34] battles.head(3)

Out [34] Start Date
         1781-09-06    Connecticut
         1779-07-05    Connecticut
         1777-04-27    Connecticut
         Name: State, dtype: object
```

`inplace` 参数是一个常见的混淆点。它的名字暗示它修改或突变现有对象，而不是创建一个副本。开发者可能会被 `inplace` 诱惑，因为减少我们创建的副本数量可以减少内存使用。但即使有 `inplace` 参数，pandas 在我们调用方法时总是会创建一个对象的副本。库始终创建一个副本；`inplace` 参数将我们的现有变量重新分配给新对象。因此，与普遍看法相反，`inplace` 参数并不提供任何性能优势。这两行在技术上等效：

```
battles.sort_values(inplace = True)
battles = battles.sort_values()
```

为什么 pandas 开发者选择了这种实现？我们总是创建副本我们能获得什么优势？你可以在网上找到更详细的解释，但简短的答案是不可变数据结构往往导致更少的错误。记住，不可变对象无法改变。我们可以复制一个不可变对象并操作副本，但我们不能改变原始对象。Python 字符串就是一个例子。不可变对象不太可能进入损坏或无效的状态；它也更容易测试。

pandas 开发团队已经讨论过在未来的版本中从库中移除 `inplace` 参数。我的建议是如果可能的话避免使用它。替代方案是将方法的返回值重新分配给相同的变量或创建一个更具有描述性的变量。例如，我们可以将 `sort_values` 方法的返回值分配给一个如 `sorted_battles` 的变量。

## 3.4 使用 value_counts 方法计数值

这里是一个关于 `pokemon` `Series` 的提醒：

```
In  [35] pokemon.head()

Out [35] Pokemon
         Bulbasaur     Grass / Poison
         Ivysaur       Grass / Poison
         Venusaur      Grass / Poison
         Charmander              Fire
         Charmeleon              Fire
         Name: Type, dtype: object
```

我们如何找出最常见的宝可梦类型？我们需要将值分组到桶中，并计算每个桶中元素的数量。`value_counts` 方法，它计算每个 `Series` 值出现的次数，完美地解决了这个问题：

```
In  [36] pokemon.value_counts()

Out [36] Normal            65
         Water             61
         Grass             38
         Psychic           35
         Fire              30
                   ..
         Fire / Dragon      1
         Dark / Ghost       1
         Steel / Ground     1
         Fire / Psychic     1
         Dragon / Ice       1
         Name: Type, Length: 159, dtype: int64
```

`value_counts`方法返回一个新的`Series`对象。索引标签是`pokemon` `Series`的值，值是它们各自的计数。有 65 只宝可梦被归类为正常，61 只被归类为水，等等。对于那些好奇的人，“正常”宝可梦是那些在物理攻击方面表现出色的宝可梦。

`value_counts` `Series`的长度等于`pokemon` `Series`中唯一值的数量。作为提醒，`nunique`方法返回此信息：

```
In  [37] len(pokemon.value_counts())

Out [37] 159

In  [38] pokemon.nunique()

Out [38] 159
```

在这种情况下，数据完整性至关重要。额外的空格或字符的不同大小写会导致 pandas 认为两个值不相等，并将它们分别计数。我们将在第六章中讨论数据清理。

`value_counts`方法的`ascending`参数的默认参数为`False`。Pandas 按降序对值进行排序，从出现次数最多到最少。要按升序排序值，将`ascending`参数传递一个值为`True`：

```
In  [39] pokemon.value_counts(ascending = True)

Out [39] Rock / Poison        1
         Ghost / Dark         1
         Ghost / Dragon       1
         Fighting / Steel     1
         Rock / Fighting      1
                             ..
         Fire                30
         Psychic             35
         Grass               38
         Water               61
         Normal              65
```

我们可能对宝可梦类型相对于所有类型的比率更感兴趣。将`value_counts`方法的`normalize`参数设置为`True`以返回每个唯一值的频率。一个值的频率是该值在数据集中所占的比例：

```
In  [40] pokemon.value_counts(normalize = True).head()

Out [40] Normal            0.080346
         Water             0.075402
         Grass             0.046972
         Psychic           0.043263
         Fire              0.037083
```

我们可以将频率`Series`中的值乘以 100，以得到每种宝可梦类型对整体贡献的百分比。你还记得第二章中的语法吗？我们可以使用像乘号这样的普通数学运算符与`Series`一起使用。Pandas 将对每个值应用此操作：

```
In  [41] pokemon.value_counts(normalize = True).head() * 100

Out [41] Normal            8.034611
         Water             7.540173
         Grass             4.697157
         Psychic           4.326329
         Fire              3.708282
```

正常宝可梦占数据集的 8.034611%，水宝可梦占 7.540173%，等等。真有趣！

假设我们想要限制百分比的精度。我们可以使用`round`方法对`Series`的值进行四舍五入。该方法的第一参数`decimals`设置小数点后保留的位数。下一个示例将值四舍五入到两位数字；它将前一个示例中的代码用括号括起来以避免语法错误。我们想要确保 pandas 首先将每个值乘以 100，然后对结果`Series`调用`round`方法：

```
In  [42] (pokemon.value_counts(normalize = True) * 100).round(2)

Out [42] Normal              8.03
         Water               7.54
         Grass               4.70
         Psychic             4.33
         Fire                3.71
                             ...
         Rock / Fighting     0.12
         Fighting / Steel    0.12
         Ghost / Dragon      0.12
         Ghost / Dark        0.12
         Rock / Poison       0.12
         Name: Type, Length: 159, dtype: float64
```

`value_counts`方法在数值`Series`上操作相同。下一个示例计算`google` `Series`中每个唯一股票价格的出现次数。结果发现，数据集中没有股票价格出现超过三次：

```
In  [43] google.value_counts().head()

Out [43] 237.04     3
         288.92     3
         287.68     3
         290.41     3
         194.27     3
```

为了识别数值数据集中的趋势，将值分组到预定义的区间中可能比计数唯一值更有益。让我们首先确定`google` `Series`中最小值和最大值之间的差异。`Series`的`max`和`min`方法在这里工作得很好。另一个选择是将`Series`传递给 Python 内置的`max`和`min`函数：

```
In  [44] google.max()

Out [44] 1287.58

In  [45] google.min()

Out [45] 49.82
```

最小值和最大值之间有大约 1,250 的范围。让我们将股价分成 200 的桶，从 0 开始，一直工作到 1,400。我们可以将这些区间定义为列表中的值，并将列表传递给`value_counts`方法的`bins`参数。Pandas 将使用列表中每两个后续值作为区间的下限和上限：

```
In  [46] buckets = [0, 200, 400, 600, 800, 1000, 1200, 1400]
         google.value_counts(bins = buckets)

Out [46] (200.0, 400.0]      1568
         (-0.001, 200.0]      595
         (400.0, 600.0]       575
         (1000.0, 1200.0]     406
         (600.0, 800.0]       380
         (800.0, 1000.0]      207
         (1200.0, 1400.0]      93
         Name: Close, dtype: int64
```

输出告诉我们，在数据集中，谷歌的股价在$200 到$400 之间有 1,568 个值。

注意，pandas 按每个桶中的值的数量降序对之前的`Series`进行了排序。如果我们想按区间排序结果呢？我们只需要混合匹配几个 pandas 方法。这些区间是返回的`Series`中的索引标签，因此我们可以使用`sort_index`方法来排序它们。这种连续调用多个方法的技巧称为**方法链**：

```
In  [47] google.value_counts(bins = buckets).sort_index()

Out [47] (-0.001, 200.0]      595
         (200.0, 400.0]      1568
         (400.0, 600.0]       575
         (600.0, 800.0]       380
         (800.0, 1000.0]      207
         (1000.0, 1200.0]     406
         (1200.0, 1400.0]      93
         Name: Close, dtype: int64
```

我们可以通过将`False`传递给`value_counts`方法的`sort`参数来获得相同的结果：

```
In  [48] google.value_counts(bins = buckets, sort = False)

Out [48] (-0.001, 200.0]      595
         (200.0, 400.0]      1568
         (400.0, 600.0]       575
         (600.0, 800.0]       380
         (800.0, 1000.0]      207
         (1000.0, 1200.0]     406
         (1200.0, 1400.0]      93
         Name: Close, dtype: int64
```

注意，第一个区间包含-0.001 而不是 0。当 pandas 将`Series`的值组织到桶中时，它可能将任何桶的范围扩展到.1%的任一方向。区间周围的符号具有意义：

+   一个括号标记一个值作为区间外的**排除**值。

+   一个方括号标记一个值作为区间内的**包含**值。

考虑区间 `(-0.001, 200.0]`。-0.001 被排除，200 被包含。因此，该区间捕获了所有大于-0.001 且小于或等于 200.0 的值。

一个**闭区间**包含两个端点。例如 `[5, 10]`（大于等于 5，小于等于 10）。

一个**开区间**不包含两个端点。例如 `(5, 10)`（大于 5，小于 10）。

带有`bin`参数的`value_counts`方法返回**半开区间**。Pandas 将包含一个端点并排除另一个端点。

`value_counts`方法的`bins`参数还接受一个整数参数。Pandas 将自动计算`Series`中最大值和最小值之间的差值，并将范围分成指定的数量个桶。下一个示例将`google`中的股价分成六个桶。请注意，桶的大小可能不完全相等（由于任何方向上任何区间的可能.1%扩展），但将非常接近：

```
In  [49] google.value_counts(bins = 6, sort = False)

Out [49] (48.581, 256.113]      1204
         (256.113, 462.407]     1104
         (462.407, 668.7]        507
         (668.7, 874.993]        380
         (874.993, 1081.287]     292
         (1081.287, 1287.58]     337
         Name: Close, dtype: int64
```

我们的`battles`数据集怎么样了？我们有一段时间没看到它了：

```
In  [50] battles.head()

Out [50] Start Date
         1781-09-06    Connecticut
         1779-07-05    Connecticut
         1777-04-27    Connecticut
         1777-09-03       Delaware
         1777-05-17        Florida
         Name: State, dtype: object
```

我们可以使用`value_counts`方法查看在独立战争中哪个州发生了最多的战斗：

```
In  [51] battles.value_counts().head()

Out [51] South Carolina    31
         New York          28
         New Jersey        24
         Virginia          21
         Massachusetts     11
         Name: State, dtype: int64
```

Pandas 默认会从`value_counts` `Series`中排除`NaN`值。将`dropna`参数的值设置为`False`以将空值计为一个不同的类别：

```
In  [52] battles.value_counts(dropna = False).head()

Out [52] NaN               70
         South Carolina    31
         New York          28
         New Jersey        24
         Virginia          21
         Name: State, dtype: int64
```

`Series`索引也支持`value_counts`方法。在调用方法之前，我们必须通过`index`属性访问索引对象。让我们找出在独立战争中哪一天发生了最多的战斗：

```
In  [53] battles.index

Out [53]

DatetimeIndex(['1774-09-01', '1774-12-14', '1775-04-19', '1775-04-19',
               '1775-04-20', '1775-05-10', '1775-05-27', '1775-06-11',
               '1775-06-17', '1775-08-08',
               ...
               '1782-08-08', '1782-08-15', '1782-08-19', '1782-08-26',
               '1782-08-25', '1782-09-11', '1782-09-13', '1782-10-18',
               '1782-12-06', '1783-01-22'],
              dtype='datetime64[ns]', name='Start Date', length=232,
              freq=None)

In  [54] battles.index.value_counts()

Out [54] 1775-04-19    2
         1781-05-22    2
         1781-04-15    2
         1782-01-11    2
         1780-05-25    2
                      ..
         1778-05-20    1
         1776-06-28    1
         1777-09-19    1
         1778-08-29    1
         1777-05-17    1
         Name: Start Date, Length: 217, dtype: int64
```

看起来没有哪一天发生了超过两场战斗。

## 3.5 使用 apply 方法对每个 Series 值调用函数

在 Python 中，函数是一个 *一等对象*，这意味着语言将其视为任何其他数据类型。函数可能感觉像是一个更抽象的实体，但它与其他任何数据结构一样有效。

关于一等对象的简单思考方式是：你可以用数字做的任何事情，你都可以用函数做。例如，你可以做所有以下事情：

+   将函数存储在列表中。

+   将函数分配为字典键的值。

+   将一个函数作为参数传递给另一个函数。

+   从另一个函数中返回一个函数。

区分函数和函数调用非常重要。一个 *函数* 是一系列产生输出的指令；它是一个“食谱”，尚未烹饪。相比之下，一个 *函数调用* 是指令的实际执行；它是食谱的烹饪。

下一个示例声明了一个 `funcs` 列表，该列表存储了三个 Python 内置函数。`len`、`max` 和 `min` 函数在列表中没有被调用。列表存储了函数本身的引用：

```
In  [55] funcs = [len, max, min]
```

下一个示例使用 `for` 循环遍历 `funcs` 列表。在三次迭代中，`current_func` 迭代变量代表未调用的 `len`、`max` 和 `min` 函数。在每次迭代中，循环调用动态的 `current_func` 函数，传入 `google` `Series` 并打印返回值：

```
In  [56] for current_func in funcs:
             print(current_func(google))

Out [56] 3824
         1287.58
         49.82
```

输出包括三个函数的连续返回值：`Series` 的长度、`Series` 中的最大值和最小值。

这里的关键点是我们可以像对待 Python 中的任何其他对象一样对待函数。那么这个事实如何应用到 pandas 中呢？假设我们想要将 `google` `Series` 中的每个浮点值向上或向下舍入到最接近的整数。Python 有一个方便的 `round` 函数来完成这个任务。该函数将大于 0.5 的值向上舍入，将小于 0.5 的值向下舍入：

```
In  [57] round(99.2)

Out [57] 99

In  [58] round(99.49)

Out [58] 99

In  [59] round(99.5)

Out [59] 100
```

如果我们能够将这个 `round` 函数应用到我们的 `Series` 中的每个值上，那岂不是很好？我们很幸运。`Series` 有一个名为 `apply` 的方法，它为每个 `Series` 值调用一次函数，并返回一个由函数调用返回值组成的新 `Series`。`apply` 方法期望它将调用的函数作为其第一个参数 `func`。下一个示例传递了 Python 的内置 `round` 函数：

```
In  [60] # The two lines below are equivalent
         google.apply(func = round)
         google.apply(round)

Out [60] Date
         2004-08-19      50
         2004-08-20      54
         2004-08-23      54
         2004-08-24      52
         2004-08-25      53
                       ...
         2019-10-21    1246
         2019-10-22    1243
         2019-10-23    1259
         2019-10-24    1261
         2019-10-25    1265
         Name: Close, Length: 3824, dtype: int64
```

我们已经将每个 `Series` 的值都四舍五入过了！

再次请注意，我们正在将未调用的 `round` 函数传递给 `apply` 方法。我们传递的是食谱。在 pandas 的内部某个地方，`apply` 方法知道要在每个 `Series` 值上调用我们的函数。Pandas 抽象掉了操作的复杂性。

`apply`方法也接受自定义函数。定义一个函数，它接受一个参数并返回你希望 pandas 存储在聚合`Series`中的值。

假设我们想知道有多少宝可梦是单一类型的（例如火）以及有多少宝可梦是两种或更多类型的。我们需要将相同的逻辑，即宝可梦的分类，应用于每个`Series`值。函数是一个封装该逻辑的理想容器。让我们定义一个名为`single_or_multi`的实用函数，它接受一个宝可梦类型并确定它是一个或多个类型。如果一个宝可梦有多个类型，字符串会使用斜杠（`"Fire / Ghost"`）分隔它们。我们可以使用 Python 的`in`运算符来检查参数字符串中是否存在前斜杠。`if`语句仅在条件评估为`True`时执行一个块。在我们的情况下，如果存在`/`，则函数将返回字符串`"Multi"`；否则，它将返回`"Single"`：

```
In  [61] def single_or_multi(pokemon_type):
             if "/" in pokemon_type:
                 return "Multi"

             return "Single"
```

现在我们可以将`single_or_multi`函数传递给`apply`方法。以下是对`pokemon`外观的快速回顾：

```
In  [62] pokemon.head(4)

Out [62] Pokemon
         Bulbasaur     Grass / Poison
         Ivysaur       Grass / Poison
         Venusaur      Grass / Poison
         Charmander              Fire
         Name: Type, dtype: object
```

下一个示例调用`apply`方法，将`single_or_multi`函数作为其参数。Pandas 会对每个`Series`值调用`single_or_multi`函数：

```
In  [63] pokemon.apply(single_or_multi)

Out [63] Pokemon
         Bulbasaur       Multi
         Ivysaur         Multi
         Venusaur        Multi
         Charmander     Single
         Charmeleon     Single
                         ...
         Stakataka       Multi
         Blacephalon     Multi
         Zeraora        Single
         Meltan         Single
         Melmetal       Single
         Name: Type, Length: 809, dtype: object
```

我们的第一种样本，妙蛙种子，被归类为草/毒宝可梦，因此`single_or_multi`函数返回`"Multi"`。相比之下，我们的第四种样本，小火龙，被归类为火宝可梦，因此函数返回`"Single"`。对于剩余的`pokemon`值，相同的逻辑重复出现。

我们有一个新的`Series`对象！让我们通过调用`value_counts`来找出有多少宝可梦属于每个分类：

```
In  [64] pokemon.apply(single_or_multi).value_counts()

Out [64] Multi     405
         Single    404
         Name: Type, dtype: int64
```

单一能量和多能量宝可梦的分布相当均匀。我希望这些知识在某个时刻对你的生活有所帮助。

## 3.6 编程挑战

让我们解决一个结合了本章和第二章中介绍的一些想法的挑战。

### 3.6.1 问题

假设一位历史学家向我们求助，要求我们确定在革命战争中哪一天发生了最多的战斗。最终输出应该是一个`Series`，其中包含星期（星期日、星期一等）作为索引标签，以及每天战斗的数量作为值。从头开始，导入 revolutionary_war.csv 数据集，并执行必要的操作以获得以下数据：

```
Saturday     39
Friday       39
Wednesday    32
Thursday     31
Sunday       31
Tuesday      29
Monday       27
```

解决这个问题你需要额外的 Python 知识。如果你有一个单独的 datetime 对象，你可以使用`strftime`方法并传入参数`"%A"`来返回日期所在的星期几（例如`"Sunday"`）。参见以下示例和附录 B，以获取关于 datetime 对象的更全面概述：

```
In  [65] import datetime as dt
         today = dt.datetime(2020, 12, 26)
         today.strftime("%A")

Out [65] 'Saturday'
```

提示：声明一个自定义函数来计算日期的星期几可能很有帮助。

祝你好运！

### 3.6.2 解决方案

让我们重新导入 revolutionary_war.csv 数据集，并提醒自己其原始形状：

```
In  [66] pd.read_csv("revolutionary_war.csv").head()

Out [66]
 **Battle  Start Date          State**
0                       Powder Alarm    9/1/1774  Massachusetts
1  Storming of Fort William and Mary  12/14/1774  New Hampshire
2   Battles of Lexington and Concord   4/19/1775  Massachusetts
3                    Siege of Boston   4/19/1775  Massachusetts
4                 Gunpowder Incident   4/20/1775       Virginia
```

我们不需要分析中的战斗和状态列。你可以使用任一列作为索引，或者坚持使用默认的数字索引。

关键步骤是将开始日期列中的字符串值强制转换为 datetime。如果我们处理的是日期，我们可以调用与日期相关的函数，如 `strftime`。对于普通字符串，我们没有同样的能力。让我们使用 `usecols` 参数选择开始日期列，并使用 `parse_dates` 参数将其值转换为 datetime。最后，记得将 `True` 传递给 `squeeze` 参数以创建一个 `Series` 而不是 `DataFrame`：

```
In  [67] days_of_war = pd.read_csv(
             "revolutionary_war.csv",
             usecols = ["Start Date"],
             parse_dates = ["Start Date"],
             squeeze = True,
         )

         days_of_war.head()

Out [67] 0   1774-09-01
         1   1774-12-14
         2   1775-04-19
         3   1775-04-19
         4   1775-04-20
         Name: Start Date, dtype: datetime64[ns]
```

我们下一个挑战是提取每个日期的一周中的某一天。一个解决方案（仅使用我们目前所知的工具）是将每个 `Series` 值传递给一个函数，该函数将返回该日期的一周中的某一天。现在让我们声明这个函数：

```
In  [68] def day_of_week(date):
             return date.strftime("%A")
```

我们如何为每个 `Series` 值调用一次 `day_of_week` 函数？我们可以将 `day_of_week` 函数作为参数传递给 `apply` 方法。我们期望得到一周中的某一天，但结果却...

```
In  [69] days_of_war.apply(day_of_week)

---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-411-c133befd2940> in <module>
----> 1 days_of_war.apply(day_of_week)

ValueError: NaTType does not support strftime
```

哎呀——我们的开始日期列有缺失值。与 datetime 对象不同，`NaT` 对象没有 `strftime` 方法，所以当将其传递给 `day_of_week` 函数时，pandas 会遇到麻烦。简单的解决方案是在调用 `apply` 方法之前从 `Series` 中删除所有缺失的 datetime 值。我们可以使用 `dropna` 方法做到这一点：

```
In  [70] days_of_war.dropna().apply(day_of_week)

Out [70] 0       Thursday
         1      Wednesday
         2      Wednesday
         3      Wednesday
         4       Thursday
                  ...
         227    Wednesday
         228       Friday
         229       Friday
         230       Friday
         231    Wednesday
         Name: Start Date, Length: 228, dtype: object
```

现在我们有进展了！我们需要一种方法来计算每个工作日的出现次数。`value_counts` 方法就做到了这一点：

```
In  [71] days_of_war.dropna().apply(day_of_week).value_counts()

Out [71] Saturday     39
         Friday       39
         Wednesday    32
         Thursday     31
         Sunday       31
         Tuesday      29
         Monday       27
         Name: Start Date, dtype: int64
```

完美！结果是周五和周六打平。恭喜你完成编码挑战！

## 摘要

+   `read_csv` 函数将 CSV 的内容导入 pandas 数据结构中。

+   `read_csv` 函数的参数可以自定义导入的列、索引、数据类型等。

+   `sort_values` 方法按升序或降序对 `Series` 的值进行排序。

+   `sort_index` 方法按升序或降序对 `Series` 的索引进行排序。

+   我们可以使用 `inplace` 参数将方法返回的副本重新分配给原始变量。使用 `inplace` 没有性能上的好处。

+   `value_counts` 方法计算 `Series` 中每个唯一值的出现次数。

+   `apply` 方法在 `Series` 的每个值上调用一个函数，并将结果返回到一个新的 `Series` 中。
