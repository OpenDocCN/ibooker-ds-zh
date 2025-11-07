# 6 处理文本数据

本章涵盖

+   从字符串中移除空白字符

+   将字符串转换为大写或小写

+   在字符串中查找和替换字符

+   通过字符索引位置切片字符串

+   通过分隔符拆分文本

文本数据可能会非常混乱。现实世界的数据集充满了错误的字符、不正确的字母大小写、空白字符等等。清理数据的过程被称为*整理*或*修补*。通常，我们的大部分数据分析都致力于修补。我们可能一开始就知道我们想要得出的见解，但困难在于将数据整理成适合操作的形式。幸运的是，pandas 背后的一个主要动机是简化清理格式不正确的文本值的过程。这个库经过实战检验且灵活。在本章中，我们将学习如何使用 pandas 来修复我们文本数据集中的各种缺陷。有很多内容要介绍，所以让我们直接进入正题。

## 6.1 字母大小写和空白字符

我们将首先在新的 Jupyter Notebook 中导入 pandas：

```
In  [1] import pandas as pd
```

本章的第一个数据集，chicago_food_inspections.csv，是芝加哥市进行的超过 150,000 次食品检查的列表。CSV 文件只包含两列：一列是机构的名称，另一列是风险评级。四个风险等级是风险 1（高）、风险 2（中）、风险 3（低），以及针对最严重违规者的特殊等级“所有”：

```
In  [2] inspections = pd.read_csv("chicago_food_inspections.csv")
        inspections

Out [2]

 **Name             Risk**
0                  MARRIOT MARQUIS CHICAGO    Risk 1 (High)
1                               JETS PIZZA  Risk 2 (Medium)
2                                ROOM 1520     Risk 3 (Low)
3                  MARRIOT MARQUIS CHICAGO    Risk 1 (High)
4                               CHARTWELLS    Risk 1 (High)
     ...                                  ...                 ...
153805                           WOLCOTT'S    Risk 1 (High)
153806        DUNKIN DONUTS/BASKIN-ROBBINS  Risk 2 (Medium)
153807                            Cafe 608    Risk 1 (High)
153808                         mr.daniel's    Risk 1 (High)
153809                          TEMPO CAFE    Risk 1 (High)

153810 rows × 2 columns
```

注意：chicago_food_inspections.csv 是芝加哥市提供的数据集的一个修改版本（[`mng.bz/9N60`](http://mng.bz/9N60)）。数据中存在拼写错误和不一致性；我们保留了它们，以便您可以看到现实世界中出现的各种数据不规则性。我鼓励您考虑如何使用本章中学习的技巧来优化这些数据。

我们立即在名称列中看到一个问题：字母大小写不一致。大多数行值是大写的，一些是小写的（`"mr.daniel's"`），还有一些是正常情况（`"Café 608"`）。

前面的输出没有显示`inspections`中隐藏的另一个问题：名称列的值被空白字符包围。如果我们使用方括号语法单独检查名称`Series`，我们可以更容易地发现额外的间隔。注意行尾没有对齐：

```
In  [3] inspections["Name"].head()

Out [3] 0     MARRIOT MARQUIS CHICAGO   
        1                    JETS PIZZA 
        2                     ROOM 1520 
        3      MARRIOT MARQUIS CHICAGO  
        4                  CHARTWELLS   
        Name: Name, dtype: object
```

我们可以使用`Series`上的`values`属性来获取存储值的底层 NumPy `ndarray`。空白字符出现在值的开始和结束处：

```
In  [4] inspections["Name"].head().values

Out [4] array([' MARRIOT MARQUIS CHICAGO   ', ' JETS PIZZA ',
               '   ROOM 1520 ', '  MARRIOT MARQUIS CHICAGO  ',
               ' CHARTWELLS   '], dtype=object)
```

让我们先关注空白字符。我们稍后会处理字母大小写问题。

`Series`对象的`str`属性暴露了一个`StringMethods`对象，这是一个强大的字符串处理方法工具箱：

```
In  [5] inspections["Name"].str
Out [5] <pandas.core.strings.StringMethods at 0x122ad8510>
```

每当我们想要执行字符串操作时，我们都会在`StringMethods`对象上调用一个方法，而不是在`Series`本身上。一些方法类似于 Python 的本地字符串方法，而其他方法则是 pandas 独有的。有关 Python 字符串方法的全面回顾，请参阅附录 B。

我们可以使用`strip`方法族来从字符串中删除空白字符。`lstrip`（左删除）方法从字符串的开始处删除空白字符。以下是一个基本示例：

```
In  [6] dessert = "  cheesecake  "
        dessert.lstrip()

Out [6] 'cheesecake  '
```

`rstrip`（右删除）方法从字符串的末尾删除空白字符：

```
In  [7] dessert.rstrip()

Out [7] '  cheesecake'
```

`strip`方法从字符串的两端删除空白字符：

```
In  [8] dessert.strip()

Out [8] 'cheesecake'
```

这三个`strip`方法都可在`StringMethods`对象上使用。每个方法都会返回一个新的`Series`对象，其中每个列值都应用了该操作。让我们调用它们：

```
In  [9] inspections["Name"].str.lstrip().head()

Out [9] 0    MARRIOT MARQUIS CHICAGO   
        1                   JETS PIZZA 
        2                    ROOM 1520 
        3     MARRIOT MARQUIS CHICAGO  
        4                 CHARTWELLS   
        Name: Name, dtype: object
In  [10] inspections["Name"].str.rstrip().head()

Out [10] 0      MARRIOT MARQUIS CHICAGO
         1                   JETS PIZZA
         2                    ROOM 1520
         3      MARRIOT MARQUIS CHICAGO
         4                   CHARTWELLS
         Name: Name, dtype: object

In  [11] inspections["Name"].str.strip().head()

Out [11] 0    MARRIOT MARQUIS CHICAGO
         1                 JETS PIZZA
         2                  ROOM 1520
         3    MARRIOT MARQUIS CHICAGO
         4                 CHARTWELLS
         Name: Name, dtype: object
```

现在，我们可以用没有额外空白的新`Series`覆盖现有的`Series`。在等号的右侧，我们将使用`strip`代码创建新的`Series`。在等号的左侧，我们将使用方括号语法来表示我们想要覆盖的列。Python 首先处理等号右侧的部分。总之，我们使用“名称”列创建一个没有空白的新`Series`，然后用这个新`Series`覆盖“名称”列：

```
In  [12] inspections["Name"] = inspections["Name"].str.strip()
```

这个一行解决方案适用于小数据集，但对于拥有大量列的数据集来说可能会很快变得繁琐。我们如何快速将相同的逻辑应用到所有`DataFrame`列上？你可能还记得`columns`属性，它公开了包含`DataFrame`列名的可迭代`Index`对象：

```
In  [13] inspections.columns

Out [13] Index(['Name', 'Risk'], dtype='object')
```

我们可以使用 Python 的`for`循环遍历每一列，从`DataFrame`中动态提取它，调用`str.strip`方法返回一个新的`Series`，并覆盖原始列。这个逻辑只需要两行代码：

```
In  [14] for column in inspections.columns:
             inspections[column] = inspections[column].str.strip()
```

Python 的所有字符大小写方法都可在`StringMethods`对象上使用。例如，`lower`方法会将所有字符串字符转换为小写：

```
In  [15] inspections["Name"].str.lower().head()

Out [15] 0    marriot marquis chicago
         1                 jets pizza
         2                  room 1520
         3    marriot marquis chicago
         4                 chartwells
         Name: Name, dtype: object
```

相反的`str.upper`方法返回一个包含大写字符串的`Series`。下一个示例在另一个`Series`上调用该方法，因为“名称”列已经大多是 uppercase 的：

```
In  [16] steaks = pd.Series(["porterhouse", "filet mignon", "ribeye"])
         steaks

Out [16] 0     porterhouse
         1    filet mignon
         2          ribeye
         dtype: object

In  [17] steaks.str.upper()

Out [17] 0     PORTERHOUSE
         1    FILET MIGNON
         2          RIBEYE
         dtype: object
```

假设我们想要以更标准、可读的格式获取机构的名称。我们可以使用`str.capitalize`方法将`Series`中每个字符串的第一个字母大写：

```
In  [18] inspections["Name"].str.capitalize().head()

Out [18] 0    Marriot marquis chicago
         1                 Jets pizza
         2                  Room 1520
         3    Marriot marquis chicago
         4                 Chartwells
         Name: Name, dtype: object
```

这是一个正确的步骤，但可能最好的方法是`str.title`，它将每个单词的第一个字母大写。Pandas 使用空格来识别一个单词的结束和下一个单词的开始：

```
In  [19] inspections["Name"].str.title().head()

Out [19] 0    Marriot Marquis Chicago
         1                 Jets Pizza
         2                  Room 1520
         3    Marriot Marquis Chicago
         4                 Chartwells
         Name: Name, dtype: object
```

`title`方法是一个处理地点、国家、城市和人们全名的绝佳选项。

## 6.2 字符串切片

让我们把注意力转向“风险”列。每一行的值都包含风险的数值和分类表示（例如 1 和`"高"`）。以下是该列的提醒：

```
In  [20] inspections["Risk"].head()

Out [20]

0      Risk 1 (High)
1    Risk 2 (Medium)
2       Risk 3 (Low)
3      Risk 1 (High)
4      Risk 1 (High)
Name: Risk, dtype: object
```

假设我们想从每一行中提取数值风险值。鉴于每行看似一致的格式，这个操作可能看起来很简单，但我们必须小心行事。在一个如此大的数据集中，总会有欺骗的空间：

```
In  [21] len(inspections)

Out [21] 153810
```

所有行都遵循`"风险" `数字` "(风险` `等级)"`格式吗？我们可以通过调用`unique`方法来找出答案，该方法返回一个包含列唯一值的 NumPy `ndarray`：

```
In  [22] inspections["Risk"].unique()

Out [22] array(['Risk 1 (High)', 'Risk 2 (Medium)', 'Risk 3 (Low)', 'All',
                nan], dtype=object)
```

我们必须考虑两个额外的值：缺失的`NaNs`和`'全部'`字符串。我们如何处理这些值最终取决于分析师和业务。这些值是否重要，或者可以丢弃？在这种情况下，让我们提出一个折衷方案：我们将删除缺失的`NaN`值，并将`"全部"`值替换为`"风险 4 (极端)"`。我们将选择这种方法以确保所有风险值具有一致的格式。

我们可以使用第五章中引入的`dropna`方法从`Series`中删除缺失值。我们将传递其`subset`参数一个包含`DataFrame`列的列表，pandas 将在这个列表中查找`NaN`s。下一个示例从`inspections`中删除风险列中包含`NaN`值的行：

```
In  [23] inspections = inspections.dropna(subset = ["Risk"])
```

让我们检查风险列中的唯一值：

```
In  [24] inspections["Risk"].unique()

Out [24] array(['Risk 1 (High)', 'Risk 2 (Medium)', 'Risk 3 (Low)', 'All'],
                dtype=object)
```

我们可以使用`DataFrame`的有用`replace`方法将一个值的所有出现替换为另一个值。该方法的第一参数`to_replace`设置要搜索的值，第二个参数`value`指定将每个出现替换为什么值。下一个示例将`"全部"`字符串值替换为`"风险 4 (极端)"`：

```
In  [25] inspections = inspections.replace(
             to_replace = "All", value = "Risk 4 (Extreme)"
         )
```

现在风险列中的所有值都有了一致的格式：

```
In  [26] inspections["Risk"].unique()

Out [26] array(['Risk 1 (High)', 'Risk 2 (Medium)', 'Risk 3 (Low)',
                'Risk 4 (Extreme)'], dtype=object)
```

接下来，让我们继续我们的原始目标，即提取每一行的风险数字。

## 6.3 字符串切片和字符替换

我们可以在`StringMethods`对象上使用`slice`方法通过索引位置提取字符串的子串。该方法接受起始索引和结束索引作为参数。下限（起点）是包含的，而上限（终点）是排除的。

我们的风险数字从每个字符串的索引位置 5 开始。下一个示例从索引位置 5 开始提取字符，直到（但不包括）索引位置 6：

```
In  [27] inspections["Risk"].str.slice(5, 6).head()

Out [27] 0    1
         1    2
         2    3
         3    1
         4    1
         Name: Risk, dtype: object
```

我们还可以用 Python 的列表切片语法（见附录 B）替换`slice`方法。以下代码返回与前面代码相同的结果：

```
In  [28] inspections["Risk"].str[5:6].head()

Out [28] 0    1
         1    2
         2    3
         3    1
         4    1
         Name: Risk, dtype: object
```

如果我们想从每一行中提取分类排名（`"高"`，`"中"`，`"低"`和`"全部"`)，这个挑战由于单词长度的不同而变得困难；我们不能从起始索引位置提取相同数量的字符。有几个解决方案可用。我们将讨论最健壮的选项，正则表达式，在第 6.7 节中。

现在，让我们一步一步地解决这个问题。我们可以从使用`slice`方法提取每行的风险类别开始。如果我们向`slice`方法传递一个单一值，pandas 将使用它作为下限，并提取到字符串的末尾。

以下示例从每个字符串的索引位置 8 开始提取字符，直到字符串的末尾。索引位置 8 的字符是每种风险类型的第一个字母（例如，`"High"`中的`"H"`，“Medium”中的`"M"`，`"Low"`中的`"L"`，以及`"Extreme"`中的`"E"`）：

```
In  [29] inspections["Risk"].str.slice(8).head()

Out [29] 0      High)
         1    Medium)
         2       Low)
         3      High)
         4      High)
         Name: Risk, dtype: object
```

我们也可以使用 Python 的列表切片语法。在方括号内，提供一个起始索引位置，后跟一个单冒号。结果是相同的：

```
In  [30] inspections["Risk"].str[8:].head()

Out [30] 0      High)
         1    Medium)
         2       Low)
         3      High)
         4      High)
         Name: Risk, dtype: object
```

我们仍然需要处理那些讨厌的闭合括号。这里有一个酷解决方案：将负数参数传递给 `str.slice` 方法。负数参数设置索引界限相对于字符串的末尾：-1 提取到最后一个字符，-2 提取到倒数第二个字符，依此类推。让我们从索引位置 8 提取到每个字符串的最后一个字符：

```
In  [31] inspections["Risk"].str.slice(8, -1).head()

Out [31] 0      High
         1    Medium
         2       Low
         3      High
         4      High
         Name: Risk, dtype: object
```

我们做到了！如果你更喜欢列表切片语法，你可以在方括号内冒号后面传递 -1：

```
In  [32] inspections["Risk"].str[8:-1].head()

Out [32] 0      High
         1    Medium
         2       Low
         3      High
         4      High
         Name: Risk, dtype: object
```

另一种移除闭合括号的策略是使用 `str.replace` 方法。我们可以将每个闭合括号替换为一个空字符串——一个没有字符的字符串。

每个 `str` 方法都会返回一个新的 `Series` 对象，并具有自己的 `str` 属性。这一特性允许我们在调用每个方法时引用 `str` 属性，从而按顺序链式调用多个字符串方法。以下示例展示了如何链式调用 `slice` 和 `replace` 方法：

```
In  [33] inspections["Risk"].str.slice(8).str.replace(")", "").head()

Out [33] 0      High
         1    Medium
         2       Low
         3      High
         4      High
         Name: Risk, dtype: object
```

通过从中间索引位置切片并移除结束括号，我们能够隔离每行的风险级别。

## 6.4 布尔方法

第 6.3 节介绍了 `upper` 和 `slice` 等返回字符串 `Series` 的方法。`StringMethods` 对象上可用的其他方法返回布尔值 `Series`。这些方法在过滤 `DataFrame` 时可能特别有用。

假设我们想要隔离所有名称中包含单词 `"Pizza"` 的机构。在纯 Python 中，我们使用 `in` 操作符来搜索字符串中的子字符串：

```
In  [34] "Pizza" in "Jets Pizza"

Out [34] True
```

字符串匹配的最大挑战是大小写敏感性。例如，Python 不会在 `"Jets Pizza"` 中找到字符串 `"pizza"`，因为 `"p"` 字符的大小写不匹配：

```
In  [35] "pizza" in "Jets Pizza"

Out [35] False
```

为了解决这个问题，我们需要在检查子字符串的存在之前确保所有列值的大小写一致。我们可以在全小写的 `Series` 中查找 `"pizza"`，或者在全部大写的 `Series` 中查找 `"PIZZA"`。让我们选择前者。

`contains` 方法检查每个 `Series` 值中是否包含子字符串。当 pandas 在行的字符串中找到方法参数时，该方法返回 `True`；如果没有找到，则返回 `False`。以下示例首先使用 `lower` 方法将名称列转换为小写，然后在每行中搜索 `"pizza"`：

```
In  [36] inspections["Name"].str.lower().str.contains("pizza").head()

Out [36] 0    False
         1     True
         2    False
         3    False
         4    False
         Name: Name, dtype: bool
```

我们有一个布尔 `Series`，我们可以用它来提取所有名称中包含 `"Pizza"` 的机构：

```
In  [37] has_pizza = inspections["Name"].str.lower().str.contains("pizza")
         inspections[has_pizza]

Out [37]

 **Name             Risk**
1                             JETS PIZZA  Risk 2 (Medium)
19         NANCY'S HOME OF STUFFED PIZZA    Risk 1 (High)
27            NARY'S GRILL & PIZZA ,INC.    Risk 1 (High)
29                   NARYS GRILL & PIZZA    Risk 1 (High)
68                         COLUTAS PIZZA    Risk 1 (High)
 ...                              ...                ...
153756       ANGELO'S STUFFED PIZZA CORP    Risk 1 (High)
153764                COCHIAROS PIZZA #2    Risk 1 (High)
153772  FERNANDO'S MEXICAN GRILL & PIZZA    Risk 1 (High)
153788            REGGIO'S PIZZA EXPRESS    Risk 1 (High)
153801        State Street Pizza Company    Risk 1 (High)

3992 rows × 2 columns
```

注意到 pandas 保留了姓名中的原始字母大小写。`inspections` `DataFrame` 从未改变。`lower` 方法返回一个新的 `Series`，而我们对其调用的 `contains` 方法返回另一个新的 `Series`，pandas 使用它来从原始 `DataFrame` 中过滤行。

如果我们想要在目标上更加精确，也许提取所有以字符串 `"tacos"` 开头的机构？现在我们关注子字符串在每个字符串中的位置。`str.startswith` 方法解决了这个问题，如果字符串以它的参数开头则返回 `True`：

```
In  [38] inspections["Name"].str.lower().str.startswith("tacos").head()

Out [38] 0    False
         1    False
         2    False
         3    False
         4    False
         Name: Name, dtype: bool

In  [39] starts_with_tacos = (
             inspections["Name"].str.lower().str.startswith("tacos")
         )

         inspections[starts_with_tacos]
Out [39]

 **Name           Risk**
69               TACOS NIETOS  Risk 1 (High)
556       TACOS EL TIO 2 INC.  Risk 1 (High)
675          TACOS DON GABINO  Risk 1 (High)
958       TACOS EL TIO 2 INC.  Risk 1 (High)
1036      TACOS EL TIO 2 INC.  Risk 1 (High)
...                       ...            ...
143587          TACOS DE LUNA  Risk 1 (High)
144026           TACOS GARCIA  Risk 1 (High)
146174        Tacos Place's 1  Risk 1 (High)
147810  TACOS MARIO'S LIMITED  Risk 1 (High)
151191            TACOS REYNA  Risk 1 (High)

105 rows × 2 columns
```

补充的 `str.endswith` 方法检查每个 `Series` 字符串的末尾是否有子字符串：

```
In  [40] ends_with_tacos = (
             inspections["Name"].str.lower().str.endswith("tacos")
         )

         inspections[ends_with_tacos]

Out [40]

 **Name           Risk**
382        LAZO'S TACOS  Risk 1 (High)
569        LAZO'S TACOS  Risk 1 (High)
2652       FLYING TACOS   Risk 3 (Low)
3250       JONY'S TACOS  Risk 1 (High)
3812       PACO'S TACOS  Risk 1 (High)
...                 ...            ...
151121      REYES TACOS  Risk 1 (High)
151318   EL MACHO TACOS  Risk 1 (High)
151801   EL MACHO TACOS  Risk 1 (High)
153087  RAYMOND'S TACOS  Risk 1 (High)
153504        MIS TACOS  Risk 1 (High)

304 rows × 2 columns
```

无论您是在寻找字符串的开头、中间还是结尾的文本，`StringMethods` 对象都有一个辅助方法来帮助您。

## 6.5 分割字符串

我们下一个数据集是一组虚构的客户。每一行包括客户的姓名和地址。让我们使用 `read_csv` 函数导入 customers.csv 文件，并将 `DataFrame` 赋值给 `customers` 变量：

```
In  [41] customers = pd.read_csv("customers.csv")
         customers.head()

Out [41]
 **Name                                              Address**
0        Frank Manning  6461 Quinn Groves, East Matthew, New Hampshire,166...
1    Elizabeth Johnson   1360 Tracey Ports Apt. 419, Kyleport, Vermont,319...
2      Donald Stephens   19120 Fleming Manors, Prestonstad, Montana, 23495
3  Michael Vincent III        441 Olivia Creek, Jimmymouth, Georgia, 82991
4       Jasmine Zamora     4246 Chelsey Ford Apt. 310, Karamouth, Utah, 76...
```

我们可以使用 `str.len` 方法来返回每行字符串的长度。例如，行 0 的 `"Frank Manning"` 值的长度为 13 个字符：

```
In  [42] customers["Name"].str.len().head()

Out [42] 0    13
         1    17
         2    15
         3    19
         4    14
         Name: Name, dtype: int64
```

假设我们想要将每个客户的第一个和最后一个名字分别放在两个单独的列中。您可能熟悉 Python 的 `split` 方法，该方法使用指定的分隔符来分割字符串。该方法返回一个由所有分割后的子字符串组成的列表。下一个示例使用连字符分隔符将电话号码分割成三个字符串列表：

```
In  [43] phone_number = "555-123-4567"
         phone_number.split("-")

Out [43] ['555', '123', '4567']
```

`str.split` 方法对 `Series` 中的每一行执行相同的操作；它的返回值是一个列表的 `Series`。我们将分隔符传递给方法的第一个参数，`pat`（代表 *pattern*）。下一个示例通过空格的存在来分割 `Name` 中的值：

```
In  [44] # The two lines below are equivalent
         customers["Name"].str.split(pat = " ").head()
         customers["Name"].str.split(" ").head()

Out [44] 0           [Frank, Manning]
         1       [Elizabeth, Johnson]
         2         [Donald, Stephens]
         3    [Michael, Vincent, III]
         4          [Jasmine, Zamora]
         Name: Name, dtype: object
```

接下来，让我们重新调用这个新的列表 `Series` 上的 `str.len` 方法来获取每个列表的长度。Pandas 会根据 `Series` 存储的数据类型动态反应：

```
In  [45] customers["Name"].str.split(" ").str.len().head()

Out [45] 0    2
         1    2
         2    2
         3    3
         4    2
         Name: Name, dtype: int64
```

我们有一个小问题。由于 `"MD"` 和 `"Jr"` 这样的后缀，一些名字有多于两个单词。我们可以在索引位置 3 看到一个例子：Michael Vincent III，pandas 会将其分割成三个元素的列表。为了确保每个列表有相同数量的元素，我们可以限制分割的数量。如果我们设置一个最大分割阈值为一次，pandas 将会在第一个空格处分割字符串并停止。然后我们将有一个由两个元素的列表组成的 `Series`。每个列表将包含客户的第一个名字以及随后的任何内容。

下一个示例将 `1` 作为参数传递给 `split` 方法的 `n` 参数，这设置了最大分割次数。看看 pandas 如何处理索引 3 的 `"Michael Vincent III"`：

```
In  [46] customers["Name"].str.split(pat = " ", n = 1).head()

Out [46] 0          [Frank, Manning]
         1      [Elizabeth, Johnson]
         2        [Donald, Stephens]
         3    [Michael, Vincent III]
         4         [Jasmine, Zamora]
         Name: Name, dtype: object
```

现在我们所有的列表长度都相等了。我们可以使用 `str.get` 来根据每行的列表的索引位置提取一个值。例如，我们可以定位到索引 0，来提取每个列表的第一个元素，也就是客户的第一个名字：

```
In  [47] customers["Name"].str.split(pat = " ", n = 1).str.get(0).head()

Out [47] 0        Frank
         1    Elizabeth
         2       Donald
         3      Michael
         4      Jasmine
         Name: Name, dtype: object
```

要从每个列表中提取姓氏，我们可以将 `get` 方法传递一个索引位置为 1：

```
In  [48] customers["Name"].str.split(pat = " ", n = 1).str.get(1).head()

Out [48] 0        Manning
         1        Johnson
         2       Stephens
         3    Vincent III
         4         Zamora
         Name: Name, dtype: object
```

`get` 方法也支持负参数。参数 `-1` 从每行的列表中提取最后一个元素，无论列表有多少元素。以下代码产生与前面代码相同的结果，并且在列表长度不同的情况下更灵活：

```
In  [49] customers["Name"].str.split(pat = " ", n = 1).str.get(-1).head()

Out [49] 0        Manning
         1        Johnson
         2       Stephens
         3    Vincent III
         4         Zamora
         Name: Name, dtype: object
```

到目前为止，一切顺利。我们已经使用两个单独的 `get` 方法调用来提取两个单独的 `Series` 中的第一个和最后一个名字。不是很好吗？在单个方法调用中执行相同的逻辑？幸运的是，`str.split` 方法接受一个 `expand` 参数，当我们传递一个 `True` 参数时，该方法返回一个新的 `DataFrame` 而不是列表的 `Series`：

```
In  [50] customers["Name"].str.split(
             pat = " ", n = 1, expand = True
         ).head()

Out [50]

 **0            1**
0      Frank      Manning
1  Elizabeth      Johnson
2     Donald     Stephens
3    Michael  Vincent III
4    Jasmine       Zamora
```

我们得到了一个新的 `DataFrame`！因为我们没有为列提供自定义名称，pandas 默认在列轴上使用数字索引。

在这些情况下要小心。如果我们不使用 `n` 参数限制拆分的次数，pandas 将在元素不足的行中放置 `None` 值：

```
In  [51] customers["Name"].str.split(pat = " ", expand = True).head()

Out [51]

 **0         1     2**
0      Frank   Manning  None
1  Elizabeth   Johnson  None
2     Donald  Stephens  None
3    Michael   Vincent   III
4    Jasmine    Zamora  None
```

现在我们已经隔离了客户的姓名，让我们将新的两列 `DataFrame` 附接到现有的客户 `DataFrame` 上。在等号的右边，我们将使用 `split` 代码来创建 `DataFrame`。在等号的左边，我们将提供一个列名列表，放在一对方括号内。Pandas 将将这些列附加到客户上。下一个示例添加了两个新列，即“First Name”和“Last Name”，并用 `split` 方法返回的 `DataFrame` 来填充它们：

```
In  [52] customers[["First Name", "Last Name"]] = customers[
             "Name"
         ].str.split(pat = " ", n = 1, expand = True)
```

让我们看看结果：

```
In  [53] customers

Out [53]

 **Name                   Address    First Name     Last Name**
0           Frank Manning  6461 Quinn Groves, E...       Frank      Manning
1       Elizabeth Johnson  1360 Tracey Ports Ap...   Elizabeth      Johnson
2         Donald Stephens  19120 Fleming Manors...      Donald     Stephens
3     Michael Vincent III  441 Olivia Creek, Ji...     Michael  Vincent III
4          Jasmine Zamora  4246 Chelsey Ford Ap...     Jasmine       Zamora
 ...              ...                 ...                ...            ...
9956        Dana Browning  762 Andrew Views Apt...        Dana     Browning
9957      Amanda Anderson  44188 Day Crest Apt ...      Amanda     Anderson
9958           Eric Davis  73015 Michelle Squar...        Eric        Davis
9959     Taylor Hernandez  129 Keith Greens, Ha...      Taylor    Hernandez
9960     Sherry Nicholson  355 Griffin Valley, ...      Sherry    Nicholson

9961 rows × 4 columns
```

太棒了！现在我们已经将客户的姓名提取到单独的列中，我们可以删除原始的姓名列。一种方法是使用我们客户的 `DataFrame` 上的 `drop` 方法。我们将传递列的名称到 `labels` 参数，并将 `"columns"` 作为 `axis` 参数的参数。我们需要包含 `axis` 参数来告诉 pandas 在列中而不是行中查找姓名标签：

```
In  [54] customers = customers.drop(labels = "Name", axis = "columns")
```

记住，突变操作在 Jupyter Notebook 中不会产生输出。我们必须打印 `DataFrame` 来查看结果：

```
In  [55] customers.head()

Out [55]

 **Address   First Name     Last Name**
0  6461 Quinn Groves, East Matthew, New Hampshire...       Frank      Manning
1   1360 Tracey Ports Apt. 419, Kyleport, Vermont...   Elizabeth      Johnson
2      19120 Fleming Manors, Prestonstad, Montana...      Donald     Stephens
3           441 Olivia Creek, Jimmymouth, Georgia...     Michael  Vincent III
4     4246 Chelsey Ford Apt. 310, Karamouth, Utah...     Jasmine       Zamora
```

好了，姓名列已经消失了，我们已经将其内容拆分到了两个新的列中。

## 6.6 编程挑战

现在是你练习本章引入的概念的机会。

### 6.6.1 问题

我们的客户数据集包括一个地址列。每个地址由街道、城市、州和邮政编码组成。你的挑战是将这四个值分开；将它们分配给新的街道、城市、州和邮政编码列；然后删除地址列。尝试解决这个问题，然后查看解决方案。

### 6.6.2 解决方案

我们的第一步是使用 `split` 方法通过分隔符拆分地址字符串。单独的逗号似乎是一个好的参数：

```
In  [56] customers["Address"].str.split(",").head()

Out [56] 0    [6461 Quinn Groves,  East Matthew,  New Hampsh...
         1    [1360 Tracey Ports Apt. 419,  Kyleport,  Vermo...
         2    [19120 Fleming Manors,  Prestonstad,  Montana,...
         3    [441 Olivia Creek,  Jimmymouth,  Georgia,  82991]
         4    [4246 Chelsey Ford Apt. 310,  Karamouth,  Utah...
         Name: Address, dtype: object
```

不幸的是，这个分割操作保留了逗号后面的空格。我们可以通过使用 `strip` 等方法进行额外的清理，但有一个更好的解决方案。如果我们仔细思考，地址的每一部分都是由逗号和空格分隔的。因此，我们可以将这两个字符作为分隔符传递给 `split` 方法：

```
In  [57] customers["Address"].str.split(", ").head()

Out [57] 0    [6461 Quinn Groves, East Matthew, New Hampshir...
         1    [1360 Tracey Ports Apt. 419, Kyleport, Vermont...
         2    [19120 Fleming Manors, Prestonstad, Montana, 2...
         3       [441 Olivia Creek, Jimmymouth, Georgia, 82991]
         4    [4246 Chelsey Ford Apt. 310, Karamouth, Utah, ...
         Name: Address, dtype: object
```

现在列表中的每个子字符串开头都没有多余的空格了。

默认情况下，`split` 方法返回一个列表的 `Series`。我们可以通过传递 `expand` 参数一个值为 `True` 的参数来使方法返回一个 `DataFrame`：

```
In  [58] customers["Address"].str.split(", ", expand = True).head()

Out [58]

 **0             1              2      3**
0           6461 Quinn Groves  East Matthew  New Hampshire  16656
1  1360 Tracey Ports Apt. 419      Kyleport        Vermont  31924
2        19120 Fleming Manors   Prestonstad        Montana  23495
3            441 Olivia Creek    Jimmymouth        Georgia  82991
4  4246 Chelsey Ford Apt. 310     Karamouth           Utah  76252
```

我们还有一些步骤要做。让我们将新的四列 `DataFrame` 添加到现有的客户 `DataFrame` 中。我们将定义一个包含新列名称的列表。这次，让我们将列表赋给一个变量以简化可读性。接下来，我们将在等号前传递列表，并在等号右侧使用前面的代码创建新的 `DataFrame`：

```
In  [59] new_cols = ["Street", "City", "State", "Zip"]

         customers[new_cols] = customers["Address"].str.split(
              pat = ", ", expand = True   
         )
```

最后一步是删除原始的 Address 列。`drop` 方法在这里是一个很好的解决方案。为了永久更改 `DataFrame`，请确保用返回的 `DataFrame` 覆盖 customers：

```
In  [60] customers.drop(labels = "Address", axis = "columns").head()

Out [60]

 **First Name    Last Name        Street          City         State    Zip**
0      Frank      Manning  6461 Quin...  East Matthew  New Hamps...  16656
1  Elizabeth      Johnson  1360 Trac...      Kyleport       Vermont  31924
2     Donald     Stephens  19120 Fle...   Prestonstad       Montana  23495
3    Michael  Vincent III  441 Olivi...    Jimmymouth       Georgia  82991
4    Jasmine       Zamora  4246 Chel...     Karamouth          Utah  76252
```

另一个选项是在目标列之前使用 Python 的内置 `del` 关键字。这个语法会修改 `DataFrame`：

```
In  [61] del customers["Address"]
```

让我们看看最终产品：

```
In  [62] customers.tail()

Out [62]

 **First Name  Last Name        Street         City            State    Zip**
9956       Dana   Browning  762 Andrew ...   North Paul     New Mexico  28889
9957     Amanda   Anderson  44188 Day C...  Lake Marcia          Maine  37378
9958       Eric      Davis  73015 Miche...  Watsonville  West Virginia  03933
9959     Taylor  Hernandez  129 Keith G...    Haleyfurt       Oklahoma  98916
9960     Sherry  Nicholson  355 Griffin...    Davidtown     New Mexico  17581
```

我们已经成功地将地址列的内容提取到了四个新的列中。恭喜你完成了编码挑战！

## 6.7 关于正则表达式的说明

任何关于处理文本数据的讨论如果没有提到正则表达式（也称为 RegEx）都是不完整的。正则表达式是一种搜索模式，用于在字符串中查找字符序列。

我们使用特殊的语法来声明正则表达式，该语法由符号和字符组成。例如，`\d` 匹配介于 0 和 9 之间的任何数字。通过正则表达式，我们可以通过针对小写字母、大写字母、数字、斜杠、空白、字符串边界等来定义复杂的搜索模式。

假设一个像 555-555-5555 这样的电话号码隐藏在一个更长的字符串中。我们可以使用正则表达式来定义一个搜索算法，提取由三个连续数字、一个破折号、三个连续数字、另一个破折号和四个更多连续数字组成的序列。这种粒度级别赋予了正则表达式其强大的功能。

下面是一个快速示例，展示了语法在实际中的应用。接下来的代码示例使用 Street 列上的 `replace` 方法，将所有连续四个数字替换为星号字符：

```
In  [63] customers["Street"].head()

Out [63]  0             6461 Quinn Groves
          1    1360 Tracey Ports Apt. 419
          2          19120 Fleming Manors
          3              441 Olivia Creek
          4    4246 Chelsey Ford Apt. 310
          Name: Street, dtype: object

In  [64] customers["Street"].str.replace(
             "\d{4,}", "*", regex = True
         ).head()

Out [64] 0             * Quinn Groves
         1    * Tracey Ports Apt. 419
         2           * Fleming Manors
         3           441 Olivia Creek
         4    * Chelsey Ford Apt. 310
         Name: Street, dtype: object
```

正则表达式是一个高度专业化的技术主题。关于正则表达式的复杂性，已经有许多书籍被撰写。目前，重要的是要注意，pandas 支持大多数字符串方法的正则表达式参数。你可以查看附录 E 以获得该领域的更全面介绍。

## 摘要

+   `str` 属性包含一个 `StringMethods` 对象，该对象具有对 `Series` 值执行字符串操作的方法。

+   `strip` 方法族用于从字符串的开始、结束或两侧移除空白字符。

+   `upper`、`lower`、`capitalize` 和 `title` 等方法用于修改字符串字符的大小写。

+   `contains` 方法用于检查另一个字符串中是否存在子字符串。

+   `startswith` 方法用于检查字符串开头是否存在子字符串。

+   补充的 `endswith` 方法用于检查字符串末尾是否存在子字符串。

+   `split` 方法通过使用指定的分隔符将字符串分割成列表。我们可以用它来分割 `DataFrame` 列中的文本，跨越多个 `Series`。
