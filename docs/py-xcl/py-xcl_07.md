第 5 章\. 使用 pandas 进行数据分析

本章将为您介绍 pandas，即 Python 数据分析库，或者——我喜欢这样说——具有超能力的基于 Python 的电子表格。它非常强大，以至于我曾与一些公司合作时，他们完全放弃了 Excel，而是用 Jupyter 笔记本和 pandas 的组合来替代它。然而，作为本书的读者，我假设您会继续使用 Excel，在这种情况下，pandas 将作为在电子表格中获取数据的接口。pandas 使在 Excel 中特别痛苦的任务变得更加简单、快速和少出错。其中一些任务包括从外部源获取大型数据集以及处理统计数据、时间序列和交互式图表。pandas 最重要的超能力是向量化和数据对齐。正如我们在上一章中看到的使用 NumPy 数组一样，向量化使您能够编写简洁的基于数组的代码，而数据对齐则确保在处理多个数据集时不会出现数据不匹配的情况。

这一章涵盖了整个数据分析过程：从清洗和准备数据开始，然后通过聚合、描述统计和可视化来理解更大的数据集。在本章末尾，我们将看到如何使用 pandas 导入和导出数据。但首先，让我们从介绍 pandas 的主要数据结构开始：DataFrame 和 Series！

DataFrame 和 Series

DataFrame 和 Series 是 pandas 中的核心数据结构。在本节中，我将介绍它们，并重点介绍 DataFrame 的主要组成部分：索引、列和数据。DataFrame 类似于二维 NumPy 数组，但它带有列和行标签，每列可以容纳不同的数据类型。通过从 DataFrame 中提取单列或单行，您会得到一个一维 Series。同样，Series 类似于带有标签的一维 NumPy 数组。当您查看 [图 5-1](#filepos485286) 中 DataFrame 的结构时，您会很容易想象到，DataFrame 就是您基于 Python 的电子表格。

![](images/00069.jpg)

图 5-1\. 一个 pandas Series 和 DataFrame

要展示从电子表格转换到 DataFrame 有多容易，请考虑下面的 Excel 表格 [图 5-2](#filepos485969)，它显示了在线课程的参与者及其分数。您可以在伴随仓库的 xl 文件夹中找到相应的文件 course_participants.xlsx。

![](images/00077.jpg)

图 5-2\. course_participants.xlsx

要在 Python 中使用这个 Excel 表格，首先导入 pandas，然后使用它的`read_excel`函数，该函数返回一个 DataFrame：

> `In``[``1``]:``import``pandas``as``pd`
> 
> `In``[``2``]:``pd``.``read_excel``(``"xl/course_participants.xlsx"``)`
> 
> `Out[2]:    user_id   name  age  country  score continent         0     1001   Mark   55    Italy    4.5    Europe         1     1000   John   33      USA    6.7   America         2     1002    Tim   41      USA    3.9   America         3     1003  Jenny   12  Germany    9.0    Europe`
> 
> PYTHON 3.9下的READ_EXCEL函数
> 
> 如果你在使用Python 3.9或更高版本运行`pd.read_excel`，请确保至少使用pandas 1.2，否则在读取xlsx文件时会出错。

如果你在Jupyter笔记本中运行这段代码，DataFrame将以HTML表格的形式进行漂亮的格式化，这使得它与Excel中的表格更加接近。我将在[第7章](index_split_019.html#filepos863345)中详细介绍使用pandas读写Excel文件，因此这只是一个介绍性的示例，展示电子表格和DataFrame确实非常相似。现在让我们从头开始重新创建这个DataFrame，而不是从Excel文件中读取它：创建DataFrame的一种方法是提供数据作为嵌套列表，并为`columns`和`index`提供值：

> `In``[``3``]:``data``=``[[``"Mark"``,``55``,``"Italy"``,``4.5``,``"Europe"``],``[``"John"``,``33``,``"USA"``,``6.7``,``"America"``],``[``"Tim"``,``41``,``"USA"``,``3.9``,``"America"``],``[``"Jenny"``,``12``,``"Germany"``,``9.0``,``"Europe"``]]``df``=``pd``.``DataFrame``(``data``=``data``,``columns``=``[``"name"``,``"age"``,``"country"``,``"score"``,``"continent"``],``index``=``[``1001``,``1000``,``1002``,``1003``])``df`
> 
> `Out[3]:        name  age  country  score continent         1001   Mark   55    Italy    4.5    Europe         1000   John   33      USA    6.7   America         1002    Tim   41      USA    3.9   America         1003  Jenny   12  Germany    9.0    Europe`

通过调用`info`方法，您将获得一些基本信息，最重要的是数据点的数量和每列的数据类型：

> `In``[``4``]:``df``.``info``()`
> 
> `<class 'pandas.core.frame.DataFrame'> Int64Index: 4 entries, 1001 to 1003 Data columns (total 5 columns): #   Column     Non-Null Count  Dtype ---  ------     --------------  ----- 0   name       4 non-null      object 1   age        4 non-null      int64 2   country    4 non-null      object 3   score      4 non-null      float64 4   continent  4 non-null      object dtypes: float64(1), int64(1), object(3) memory usage: 192.0+ bytes`

如果你只对列的数据类型感兴趣，请运行`df.dtypes`。字符串或混合数据类型的列将具有数据类型`object`。[1](index_split_016.html#filepos767133) 现在让我们更详细地看一下DataFrame的索引和列。

索引

DataFrame的行标签称为索引。如果没有有意义的索引，请在构造DataFrame时将其省略。pandas将自动创建从零开始的整数索引。我们在从Excel文件读取DataFrame的第一个示例中看到了这一点。索引将允许pandas更快地查找数据，并对许多常见操作至关重要，例如合并两个DataFrame。您可以像以下方式访问索引对象：

> `In``[``5``]:``df``.``index`
> 
> `Out[5]: Int64Index([1001, 1000, 1002, 1003], dtype='int64')`

如果有意义，给索引起个名字。让我们按照Excel表格的方式，并给它命名为`user_id`：

> `In``[``6``]:``df``.``index``.``name``=``"user_id"``df`
> 
> `Out[6]:           name  age  country  score continent         user_id         1001      Mark   55    Italy    4.5    Europe         1000      John   33      USA    6.7   America         1002       Tim   41      USA    3.9   America         1003     Jenny   12  Germany    9.0    Europe`

与数据库的主键不同，DataFrame索引可以具有重复项，但在这种情况下查找值可能较慢。要将索引转换为常规列，请使用`reset_index`，要设置新索引，请使用`set_index`。如果您不想在设置新索引时丢失现有索引，请确保首先重置它：

> `In``[``7``]:``# "reset_index"将索引转换为列，用默认索引替换``# 从最开始加载的DataFrame对应的数据框``df``.``reset_index``()`
> 
> `Out[7]:    user_id   name  age  country  score continent         0     1001   Mark   55    Italy    4.5    Europe         1     1000   John   33      USA    6.7   America         2     1002    Tim   41      USA    3.9   America         3     1003  Jenny   12  Germany    9.0    Europe`
> 
> `In``[``8``]:``# "reset_index"将"user_id"转换为常规列，而``# "set_index"将列"name"转换为索引``df``.``reset_index``()``.``set_index``(``"name"``)`
> 
> `Out[8]:        user_id  age  country  score continent         name         Mark      1001   55    Italy    4.5    Europe         John      1000   33      USA    6.7   America         Tim       1002   41      USA    3.9   America         Jenny     1003   12  Germany    9.0    Europe`

使用`df.reset_index().set_index("name")`时，您正在使用方法链接：因为`reset_index()`返回一个DataFrame，所以您可以直接调用另一个DataFrame方法，而不必先编写中间结果。

> DATAFRAME METHODS RETURN COPIES
> 
> 每当您在DataFrame上调用形式为`df.method_name()`的方法时，您将获得一个应用了该方法的DataFrame副本，保留原始DataFrame不变。我们刚刚通过调用`df.reset_index()`做到了这一点。如果您想要更改原始DataFrame，您需要将返回值分配回原始变量，如下所示：
> 
> > `df = df.reset_index()`
> > 
> 由于我们没有这样做，这意味着我们的变量 `df` 仍然保留其原始数据。接下来的示例也调用了 DataFrame 方法，即不更改原始 DataFrame。

要更改索引，请使用 `reindex` 方法：

> `In``[``9``]:``df``.``reindex``([``999``,``1000``,``1001``,``1004``])`
> 
> `Out[9]:          name   age country  score continent         user_id         999       NaN   NaN     NaN    NaN       NaN         1000     John  33.0     USA    6.7   America         1001     Mark  55.0   Italy    4.5    Europe         1004      NaN   NaN     NaN    NaN       NaN`

这是数据对齐的第一个示例：`reindex` 将接管所有匹配新索引的行，并将在不存在信息的地方引入带有缺失值（`NaN`）的行。你留下的索引元素将被删除。稍后在本章中，我将适当地介绍 `NaN`。最后，要对索引进行排序，请使用 `sort_index` 方法：

> `In``[``10``]:``df``.``sort_index``()`
> 
> `Out[10]:           name  age  country  score continent          user_id          1000      John   33      USA    6.7   America          1001      Mark   55    Italy    4.5    Europe          1002       Tim   41      USA    3.9   America          1003     Jenny   12  Germany    9.0    Europe`

如果你想按一列或多列对行进行排序，请使用 `sort_values`：

> `In``[``11``]:``df``.``sort_values``([``"continent"``,``"age"``])`
> 
> `Out[11]:           name  age  country  score continent          user_id          1000      John   33      USA    6.7   America          1002       Tim   41      USA    3.9   America          1003     Jenny   12  Germany    9.0    Europe          1001      Mark   55    Italy    4.5    Europe`

该示例展示了如何首先按 `continent`，然后按 `age` 进行排序。如果只想按一列排序，也可以将列名作为字符串提供：

> `df.sort_values("continent")`

这涵盖了索引如何工作的基础知识。现在让我们将注意力转向其水平对应物，即 DataFrame 的列！

列

要获取 DataFrame 的列信息，请运行以下代码：

> `In``[``12``]:``df``.``columns``
> 
> `Out[12]: Index(['name', 'age', 'country', 'score', 'continent'], dtype='object')`

如果在构建 DataFrame 时没有提供任何列名，pandas 将使用从零开始的整数为列编号。然而，对于列来说，这几乎从来不是一个好主意，因为列表示变量，因此易于命名。你可以像设置索引一样给列头分配一个名称：

> `In``[``13``]:``df``.``columns``.``name``=``"properties"``df`
> 
> `Out[13]: properties   name  age  country  score continent          user_id          1001         Mark   55    Italy    4.5    Europe          1000         John   33      USA    6.7   America          1002          Tim   41      USA    3.9   America          1003        Jenny   12  Germany    9.0    Europe`

如果你不喜欢列名，可以重命名它们：

> `In``[``14``]:``df``.``rename``(``columns``=``{``"name"``:``"First Name"``,``"age"``:``"Age"``})`
> 
> `Out[14]: properties First Name  Age  country  score continent          user_id          1001             Mark   55    Italy    4.5    Europe          1000             John   33      USA    6.7   America          1002              Tim   41      USA    3.9   America          1003            Jenny   12  Germany    9.0    Europe`

如果要删除列，请使用以下语法（示例显示如何同时删除列和索引）：

> `In``[``15``]:``df``.``drop``(``columns``=``[``"name"``,``"country"``],``index``=``[``1000``,``1003``])`
> 
> `Out[15]: properties  age  score continent          user_id          1001         55    4.5    Europe          1002         41    3.9   America`

DataFrame的列和索引都由一个`Index`对象表示，因此可以通过转置DataFrame来将列变为行，反之亦然：

> `In``[``16``]:``df``.``T``# df.transpose()的快捷方式`
> 
> `Out[16]: user_id       1001     1000     1002     1003          properties          name          Mark     John      Tim    Jenny          age             55       33       41       12          country      Italy      USA      USA  Germany          score          4.5      6.7      3.9        9          continent   Europe  America  America   Europe`

值得在这里记住的是，我们的DataFrame `df` 仍然没有改变，因为我们从未将方法调用返回的DataFrame重新分配给原始的 `df` 变量。如果想要重新排列DataFrame的列，可以使用我们与索引一起使用的`reindex`方法，但通常更直观的是按所需顺序选择列：

> `In``[``17``]:``df``.``loc``[:,``[``"continent"``,``"country"``,``"name"``,``"age"``,``"score"``]]`
> 
> `Out[17]: properties continent  country   name  age  score          user_id          1001          Europe    Italy   Mark   55    4.5          1000         America      USA   John   33    6.7          1002         America      USA    Tim   41    3.9          1003          Europe  Germany  Jenny   12    9.0`

最后这个例子需要解释的地方很多：关于`loc`以及数据选择工作方式的所有内容都是下一节的主题。

数据操作

现实世界中的数据很少是一成不变的，因此在处理数据之前，您需要清理数据并将其转换为可消化的形式。我们将从查找如何从DataFrame中选择数据开始，如何更改数据，以及如何处理缺失和重复数据。然后，我们将对DataFrame执行几个计算，并查看如何处理文本数据。最后，我们将了解pandas在返回数据视图与副本时的情况。本节中有许多概念与我们在上一章中使用NumPy数组时所见的概念相关。

数据选择

让我们首先在查看其他方法之前，通过标签和位置访问数据，包括布尔索引和使用 MultiIndex 选择数据。

按标签选择

访问 DataFrame 数据的最常见方法是引用其标签。使用属性 `loc`，代表位置，指定要检索的行和列：

> `df``.``loc``[``row_selection``,``column_selection``]`

`loc` 支持切片表示法，因此可以接受冒号来分别选择所有行或列。另外，您还可以提供标签列表以及单个列或行名称。请查看[表格 5-1](#filepos526726)以查看从我们的样本 DataFrame `df` 中选择不同部分的几个示例。

表 5-1\. 按标签选择数据

|  选择  |  返回数据类型  |  示例  |
| --- | --- | --- |
|  单个值  |  标量  |   `df.loc[1000, "country"]` |
|  单列（1d）  |  Series  |   `df.loc[:, "country"]` |
|  单列（2d）  |  DataFrame  |   `df.loc[:, ["country"]]` |
|  多列  |  DataFrame  |   `df.loc[:, ["country", "age"]]` |
|  列范围  |  DataFrame  |   `df.loc[:, "name":"country"]` |
|  单行（1d）  |  Series  |   `df.loc[1000, :]` |
|  单行（2d）  |  DataFrame  |   `df.loc[[1000], :]` |
|  多行  |  DataFrame  |   `df.loc[[1003, 1000], :]` |
|  行范围  |  DataFrame  |   `df.loc[1000:1002, :]` |

> 标签切片具有闭合间隔
> 
> 使用标签的切片表示法与 Python 和 pandas 中其他一切的工作方式不一致：它们包括上限端点。

应用我们从[表格 5-1](#filepos526726)中获得的知识，让我们使用 `loc` 来选择标量、Series 和 DataFrames：

> `In``[``18``]:``# 对行和列选择使用标量返回标量``df``.``loc``[``1001``,``"name"``]`
> 
> `Out[18]: 'Mark'`
> 
> `In``[``19``]:``# 在行或列选择上使用标量返回 Series``df``.``loc``[[``1001``,``1002``],``"age"``]`
> 
> `Out[19]: user_id          1001    55          1002    41          Name: age, dtype: int64`
> 
> `In``[``20``]:``# 选择多行和多列返回 DataFrame``df``.``loc``[:``1002``,``[``"name"``,``"country"``]]`
> 
> `Out[20]: properties  name country          user_id          1001        Mark   Italy          1000        John     USA          1002         Tim     USA`

重要的是，您要理解 DataFrame 与 Series 之间的区别：即使有单个列，DataFrame 也是二维的，而 Series 是一维的。DataFrame 和 Series 都有索引，但只有 DataFrame 有列标题。当您将列选择为 Series 时，列标题将成为 Series 的名称。许多函数或方法将同时适用于 Series 和 DataFrame，但在执行算术计算时，行为会有所不同：对于 DataFrame，pandas 根据列标题对齐数据—稍后在本章中会详细介绍。

> 列选择的快捷方式
> 
> 由于选择列是一个如此常见的操作，pandas 提供了一个快捷方式。而不是：
> 
> > `df``.``loc``[:,``column_selection``]`
> > 
> 你可以这样写：
> 
> > `df``[``column_selection``]`
> > 
> 例如，`df["country"]` 从我们的示例 DataFrame 返回一个 Series，而 `df[["name", "country"]]` 返回一个包含两列的 DataFrame。

按位置选择

通过位置选择 DataFrame 的子集对应于我们在本章开始时使用 NumPy 数组所做的事情。但是，对于 DataFrame，你必须使用 `iloc` 属性，它代表整数位置：

> `df``.``iloc``[``row_selection``,``column_selection``]`

在使用切片时，你要处理标准的半开区间。[表 5-2](#filepos540243) 给出了与我们之前在 [表 5-1](#filepos526726) 中查看的相同案例。

表 5-2\. 按位置选择数据

|  选择  |  返回数据类型  |  示例  |
| --- | --- | --- |
|  单个值  |  Scalar  |   `df.iloc[1, 2]` |
|  一列（1d）  |  Series  |   `df.iloc[:, 2]` |
|  一列（2d）  |  DataFrame  |   `df.iloc[:, [2]]` |
|  多列  |  DataFrame  |   `df.iloc[:, [2, 1]]` |
|  列范围  |  DataFrame  |   `df.iloc[:, :3]` |
|  一行（1d）  |  Series  |   `df.iloc[1, :]` |
|  一行（2d）  |  DataFrame  |   `df.iloc[[1], :]` |
|  多行  |  DataFrame  |   `df.iloc[[3, 1], :]` |
|  行范围  |  DataFrame  |   `df.iloc[1:3, :]` |

这是如何使用 `iloc` 的方法——与之前使用 `loc` 的样本相同：

> `In``[``21``]:``df``.``iloc``[``0``,``0``]``# 返回一个 Scalar`
> 
> `Out[21]: 'Mark'`
> 
> `In``[``22``]:``df``.``iloc``[[``0``,``2``],``1``]``# 返回一个 Series`
> 
> `Out[22]: user_id          1001    55          1002    41          Name: age, dtype: int64`
> 
> `In``[``23``]:``df``.``iloc``[:``3``,``[``0``,``2``]]``# 返回一个 DataFrame`
> 
> `Out[23]: properties  name country          user_id          1001        Mark   Italy          1000        John     USA          1002         Tim     USA`

按标签或位置选择数据并不是访问 DataFrame 子集的唯一方式。另一种重要的方式是使用布尔索引；让我们看看它是如何工作的！

按布尔索引选择

布尔索引是指使用仅包含 `True` 或 `False` 的 Series 或 DataFrame 来选择 DataFrame 的子集。布尔 Series 用于选择 DataFrame 的特定列和行，而布尔 DataFrame 用于选择整个 DataFrame 中的特定值。最常见的用法是用布尔索引来过滤 DataFrame 的行。可以把它看作是 Excel 中的自动筛选功能。例如，这是如何筛选只显示住在美国且年龄超过 40 岁的人的 DataFrame 的方法：

> `In``[``24``]:``tf``=``(``df``[``"age"``]``>``40``)``&``(``df``[``"country"``]``==``"USA"``)``tf``# 这是一个仅包含 True/False 的 Series`
> 
> `Out[24]: user_id          1001    False          1000    False          1002     True          1003    False          dtype: bool`
> 
> `In``[``25``]:``df``.``loc``[``tf``,``:]`
> 
> `Out[25]: properties name  age country  score continent          user_id          1002        Tim   41     USA    3.9   America`

这里有两件事需要解释。首先，由于技术限制，你不能在数据框（DataFrames）中像[第三章](index_split_010.html#filepos178328)中那样使用Python的布尔运算符。相反，你需要使用如[表5-3](#filepos554254)所示的符号。

表5-3\. 布尔运算符

|  基本Python数据类型  |  数据框和Series  |
| --- | --- |
|   `and` |   `&` |
|   `or` |   `&#124;` |
|   `not` |   `~` |

其次，如果你有多个条件，请确保将每个布尔表达式放在括号中，以避免运算符优先级成为问题：例如，`&`的运算符优先级高于`==`。因此，如果没有括号，示例中的表达式将被解释为：

> `df``[``"age"``]``>``(``40``&``df``[``"country"``])``==``"USA"`

如果你想要过滤索引，你可以引用它作为`df.index`：

> `In``[``26``]:``df``.``loc``[``df``.``index``>``1001``,``:]`
> 
> `Out[26]: properties   name  age  country  score continent          user_id          1002          Tim   41      USA    3.9   America          1003        Jenny   12  Germany    9.0    Europe`

如果你想在基本的Python数据结构（如列表）中使用`in`操作符，那么在Series中使用`isin`来过滤你的数据框（DataFrame）以选择来自意大利和德国的参与者：

> `In``[``27``]:``df``.``loc``[``df``[``"country"``]``.``isin``([``"Italy"``,``"Germany"``]),``:]`
> 
> `Out[27]: properties   name  age  country  score continent          user_id          1001         Mark   55    Italy    4.5    Europe          1003        Jenny   12  Germany    9.0    Europe`

当你使用`loc`来提供一个布尔Series时，数据框提供了一个特殊的语法，无需`loc`即可选择给定完整布尔DataFrame的值：

> `df``[``boolean_df``]`

如果你的数据框（DataFrame）仅包含数字，这将特别有帮助。提供一个布尔DataFrame将在布尔DataFrame为`False`时在数据框中返回`NaN`。稍后将更详细地讨论`NaN`。让我们从创建一个名为`rainfall`的新样本数据框开始，其中只包含数字：

> `In``[``28``]:``# 这可能是以毫米为单位的年降雨量``rainfall``=``pd``.``DataFrame``(``data``=``{``"City 1"``:``[``300.1``,``100.2``],``"City 2"``:``[``400.3``,``300.4``],``"City 3"``:``[``1000.5``,``1100.6``]})``rainfall`
> 
> `Out[28]:    City 1  City 2  City 3          0   300.1   400.3  1000.5          1   100.2   300.4  1100.6`
> 
> `In``[``29``]:``rainfall``<``400`
> 
> `Out[29]:    City 1  City 2  City 3          0    True   False   False          1    True    True   False`
> 
> `In``[``30``]:``rainfall``[``rainfall``<``400``]`
> 
> `Out[30]:    City 1  City 2  City 3          0   300.1     NaN     NaN          1   100.2   300.4     NaN`

注意，在这个例子中，我使用了字典来构建一个新的DataFrame——如果数据已经以这种形式存在，这通常很方便。以这种方式使用布尔值通常用于过滤特定值，如异常值。

结束数据选择部分之前，我将介绍一种特殊类型的索引称为MultiIndex。

使用MultiIndex进行选择

一个**MultiIndex**是一个具有多个层级的索引。它允许你按层次分组数据，并轻松访问子集。例如，如果将我们示例DataFrame `df` 的索引设置为 `continent` 和 `country` 的组合，你可以轻松选择特定大陆的所有行：

> `In[31]:` # MultiIndex 需要排序`df_multi`=`df`.`reset_index`()`.`set_index`([``"continent"``,``"country"``])`df_multi`=`df_multi`.`sort_index`()``df_multi`
> 
> `Out[31]: properties         user_id   name  age  score          continent country          America   USA         1000   John   33    6.7                    USA         1002    Tim   41    3.9          Europe    Germany     1003  Jenny   12    9.0                    Italy       1001   Mark   55    4.5`
> 
> `In[32]:` `df_multi``.``loc``[``"Europe"``,``:]`
> 
> `Out[32]: properties  user_id   name  age  score          country          Germany        1003  Jenny   12    9.0          Italy          1001   Mark   55    4.5`

请注意，pandas通过不重复左侧索引级别（大陆）来美化MultiIndex的输出。而是在每行更改时仅打印大陆。通过提供元组来选择多个索引级别：

> `In[33]:` `df_multi`.`loc``[(``"Europe"``,``"Italy"``),``:]`
> 
> `Out[33]: properties         user_id  name  age  score          continent country          Europe    Italy       1001  Mark   55    4.5`

如果要选择性地重置MultiIndex的部分，请提供级别作为参数。从左侧开始，零是第一列：

> `In[34]:` `df_multi`.`reset_index`(``level``=``0``)`
> 
> `Out[34]: properties continent  user_id   name  age  score          country          USA          America     1000   John   33    6.7          USA          America     1002    Tim   41    3.9          Germany       Europe     1003  Jenny   12    9.0          Italy         Europe     1001   Mark   55    4.5`

虽然我们在本书中不会手动创建MultiIndex，但像`groupby`这样的某些操作将导致pandas返回带有MultiIndex的DataFrame，因此了解它是很好的。我们将在本章后面介绍`groupby`。

现在你知道了各种选择数据的方法，现在是时候学习如何更改数据了。

数据设置

更改DataFrame数据的最简单方法是使用`loc`或`iloc`属性为特定元素分配值。这是本节的起点，在转向操作现有DataFrame的其他方法之前：替换值和添加新列。

通过标签或位置设置数据

正如本章前面所指出的，当你调用`df.reset_index()`等数据框方法时，该方法总是应用于一个副本，保持原始数据框不变。然而，通过`loc`和`iloc`属性赋值会改变原始数据框。由于我想保持我们的数据框`df`不变，因此在这里我使用了一个称为`df2`的副本。如果你想改变单个值，请按照以下步骤操作：

> `In``[``35``]:``# 先复制数据框以保留原始数据不变``df2``=``df``.``copy``()`
> 
> `In``[``36``]:``df2``.``loc``[``1000``,``"name"``]``=``"JOHN"``df2`
> 
> `Out[36]: properties   name  age  country  score continent          user_id          1001         Mark   55    Italy    4.5    Europe          1000         JOHN   33      USA    6.7   America          1002          Tim   41      USA    3.9   America          1003        Jenny   12  Germany    9.0    Europe`

你也可以同时更改多个值。改变ID为1000和1001的用户的分数的一种方法是使用列表：

> `In``[``37``]:``df2``.``loc``[[``1000``,``1001``],``"score"``]``=``[``3``,``4``]``df2`
> 
> `Out[37]: properties   name  age  country  score continent          user_id          1001         Mark   55    Italy    4.0    Europe          1000         JOHN   33      USA    3.0   America          1002          Tim   41      USA    3.9   America          1003        Jenny   12  Germany    9.0    Europe`

通过位置使用`iloc`来改变数据的方式与此相同。现在我们继续看看如何通过布尔索引来改变数据。

通过布尔索引设置数据

布尔索引，我们用来过滤行的方式，也可以用来在数据框中赋值。想象一下，你需要匿名化所有年龄低于20岁或来自美国的人的姓名：

> `In``[``38``]:``tf``=``(``df2``[``"age"``]``<``20``)``|``(``df2``[``"country"``]``==``"USA"``)``df2``.``loc``[``tf``,``"name"``]``=``"xxx"``df2`
> 
> `Out[38]: properties  name  age  country  score continent          user_id          1001        Mark   55    Italy    4.0    Europe          1000         xxx   33      USA    3.0   America          1002         xxx   41      USA    3.9   America          1003         xxx   12  Germany    9.0    Europe`

有时，你有一个数据集，需要跨整个数据框替换某些值，即不特定于某些列。在这种情况下，再次使用特殊语法，并像这样提供整个数据框与布尔值（此示例再次使用`rainfall`数据框）。

> `In``[``39``]:``# 先复制数据框以保留原始数据不变``rainfall2``=``rainfall``.``copy``()``rainfall2`
> 
> `Out[39]:    城市 1  城市 2  城市 3          0   300.1   400.3  1000.5          1   100.2   300.4  1100.6`
> 
> `In``[``40``]:``# 将低于400的值设为0``rainfall2``[``rainfall2``<``400``]``=``0``rainfall2`
> 
> `Out[40]:    City 1  City 2  City 3          0     0.0   400.3  1000.5          1     0.0     0.0  1100.6`

如果只想用另一个值替换一个值，有一种更简单的方法，我将在下面展示给你。

通过替换值设置数据

如果要在整个DataFrame或选定列中替换某个值，请使用`replace`方法：

> `In``[``41``]:``df2``.``replace``(``"USA"``,``"U.S."``)`
> 
> `Out[41]: properties  name  age  country  score continent          user_id          1001        Mark   55    Italy    4.0    Europe          1000         xxx   33     U.S.    3.0   America          1002         xxx   41     U.S.    3.9   America          1003         xxx   12  Germany    9.0    Europe`

如果您只想在`country`列上执行操作，您可以改用以下语法：

> `df2``.``replace``({``"country"``:``{``"USA"``:``"U.S."``}})`

在这种情况下，由于`USA`只出现在`country`列中，它产生了与前一个示例相同的结果。让我们看看如何向DataFrame添加额外列，以结束这一节。

通过添加新列设置数据

要向DataFrame添加新列，请为新列名称分配值。例如，您可以使用标量或列表向DataFrame添加新列：

> `In``[``42``]:``df2``.``loc``[:,``"discount"``]``=``0``df2``.``loc``[:,``"price"``]``=``[``49.9``,``49.9``,``99.9``,``99.9``]``df2`
> 
> `Out[42]: properties  name  age  country  score continent  discount  price          user_id          1001        Mark   55    Italy    4.0    Europe         0   49.9          1000         xxx   33      USA    3.0   America         0   49.9          1002         xxx   41      USA    3.9   America         0   99.9          1003         xxx   12  Germany    9.0    Europe         0   99.9`

添加新列通常涉及矢量化计算：

> `In``[``43``]:``df2``=``df``.``copy``()``# 让我们从头开始复制``df2``。``df2``.``loc``[:,``"birth year"``]``=``2021``-``df2``[``"age"``]``df2`
> 
> `Out[43]: properties   name  age  country  score continent  birth year          user_id          1001         Mark   55    Italy    4.5    Europe        1966          1000         John   33      USA    6.7   America        1988          1002          Tim   41      USA    3.9   America        1980          1003        Jenny   12  Germany    9.0    Europe        2009`

我稍后会向你展示更多关于DataFrame计算的内容，但在我们到达那之前，请记住我已经多次使用了`NaN`吗？下一节将为您提供有关缺失数据主题的更多背景。

缺失数据

缺失数据可能会影响数据分析结果的偏差，从而使你的结论不够健壮。然而，在数据集中有空白是非常常见的，你需要处理它们。在 Excel 中，通常需要处理空单元格或 `#N/A` 错误，但是 pandas 使用 NumPy 的 `np.nan` 表示缺失数据，显示为 `NaN`。`NaN` 是浮点数的标准表示为“非数字”。对于时间戳，使用 `pd.NaT`，对于文本，pandas 使用 `None`。使用 `None` 或 `np.nan`，你可以引入缺失值：

> `In``[``44``]:``df2``=``df``.``copy``()``# 让我们从一个新的副本开始``df2``.``loc``[``1000``,``"score"``]``=``None``df2``.``loc``[``1003``,``:]``=``None``df2`
> 
> `Out[44]: properties  name   age country  score continent          user_id          1001        Mark  55.0   Italy    4.5    Europe          1000        John  33.0     USA    NaN   America          1002         Tim  41.0     USA    3.9   America          1003        None   NaN    None    NaN      None`

清理 DataFrame，通常需要删除具有缺失数据的行。这很简单：

> `In``[``45``]:``df2``.``dropna``()`
> 
> `Out[45]: properties  name   age country  score continent          user_id          1001        Mark  55.0   Italy    4.5    Europe          1002         Tim  41.0     USA    3.9   America`

然而，如果你只想删除所有值都缺失的行，请使用 `how` 参数：

> `In``[``46``]:``df2``.``dropna``(``how``=``"all"``)`
> 
> `Out[46]: properties  name   age country  score continent          user_id          1001        Mark  55.0   Italy    4.5    Europe          1000        John  33.0     USA    NaN   America          1002         Tim  41.0     USA    3.9   America`

要获得一个布尔 DataFrame 或 Series，根据是否存在 `NaN`，使用 `isna`：

> `In``[``47``]:``df2``.``isna``()`
> 
> `Out[47]: properties   name    age  country  score  continent          user_id          1001        False  False    False  False      False          1000        False  False    False   True      False          1002        False  False    False  False      False          1003         True   True     True   True       True`

要填充缺失值，使用 `fillna`。例如，将分数列中的 `NaN` 替换为其平均值（稍后我将介绍描述统计信息如 `mean`）：

> `In``[``48``]:``df2``.``fillna``({``"score"``:``df2``[``"score"``]``.``mean``()})`
> 
> `Out[48]: properties  name   age country  score continent          user_id          1001        Mark  55.0   Italy    4.5    Europe          1000        John  33.0     USA    4.2   America          1002         Tim  41.0     USA    3.9   America          1003        None   NaN    None    4.2      None`

缺失数据不是唯一需要清理数据集的条件。对于重复数据也是如此，所以让我们看看我们的选择！

重复数据

像缺失数据一样，重复项会对分析的可靠性产生负面影响。要删除重复行，请使用`drop_duplicates`方法。您可以选择提供一列子集作为参数：  

> `In``[``49``]:``df``.``drop_duplicates``([``"country"``,``"continent"``])`  
> 
> `Out[49]: properties   name  age  country  score continent          user_id          1001         Mark   55    Italy    4.5    Europe          1000         John   33      USA    6.7   America          1003        Jenny   12  Germany    9.0    Europe`  

默认情况下，这将保留第一次出现。要查找某列是否包含重复项或获取其唯一值，请使用以下两个命令（如果您希望在索引上运行此操作，请使用`df.index`而不是`df["country"]`）：  

> `In``[``50``]:``df``[``"country"``]``.``is_unique`  
> 
> `Out[50]: False`  
> 
> `In``[``51``]:``df``[``"country"``]``.``unique``()  
> 
> `Out[51]: array(['Italy', 'USA', 'Germany'], dtype=object)`  

最后，要了解哪些行是重复的，请使用`duplicated`方法，它返回一个布尔值系列：默认情况下，它使用参数`keep="first"`，保留第一次出现并仅标记重复为`True`。通过设置参数`keep=False`，它将对所有行返回`True`，包括第一次出现，从而轻松获取包含所有重复行的DataFrame。在以下示例中，我们查看`country`列是否有重复，但实际上，您经常查看索引或整个行。在这种情况下，您必须使用`df.index.duplicated()`或`df.duplicated()`代替：  

> `In``[``52``]:``# 默认情况下，它仅标记重复项为True，即没有第一次出现``df``[``"country"``]``.``duplicated``()`  
> 
> `Out[52]: user_id          1001    False          1000    False          1002     True          1003    False          Name: country, dtype: bool`  
> 
> `In``[``53``]:``# 要获取所有重复`"country"`的行，请使用``keep=False``df``.``loc``[``df``[``"country"``]``.``duplicated``(``keep``=``False``),``:]`  
> 
> `Out[53]: properties  name  age country  score continent          user_id          1000        John   33     USA    6.7   America          1002         Tim   41     USA    3.9   America`  

一旦您通过删除缺失和重复数据清理了您的DataFrame，您可能希望执行一些算术运算-下一节将为您介绍如何执行这些操作。  

算术运算  

像NumPy数组一样，DataFrame和Series利用向量化。例如，要将数字添加到`rainfall` DataFrame中的每个值，只需执行以下操作：  

> `In``[``54``]:``rainfall`  
> 
> `Out[54]:    City 1  City 2  City 3          0   300.1   400.3  1000.5          1   100.2   300.4  1100.6`  
> 
> `In``[``55``]:``rainfall``+``100`  
> 
> `Out[55]:    City 1  City 2  City 3          0   400.1   500.3  1100.5          1   200.2   400.4  1200.6`

然而，pandas 的真正力量在于其自动数据对齐机制：当你使用多个 DataFrame 进行算术运算时，pandas 会自动根据它们的列和行索引进行对齐。让我们创建第二个具有一些相同行和列标签的 DataFrame，然后进行求和操作：

> `In``[``56``]:``more_rainfall``=``pd``.``DataFrame``(``data``=``[[``100``,``200``],``[``300``,``400``]],``index``=``[``1``,``2``],``columns``=``[``"City 1"``,``"City 4"``])``more_rainfall`
> 
> `Out[56]:    City 1  City 4          1     100     200          2     300     400`
> 
> `In``[``57``]:``rainfall``+``more_rainfall`
> 
> `Out[57]:    City 1  City 2  City 3  City 4          0     NaN     NaN     NaN     NaN          1   200.2     NaN     NaN     NaN          2     NaN     NaN     NaN     NaN`

结果 DataFrame 的索引和列是两个 DataFrame 索引和列的并集：那些两个 DataFrame 中都有值的字段显示其和，而其余的 DataFrame 显示 `NaN`。如果你来自 Excel，这可能是你需要适应的地方，因为在 Excel 中，当你在算术运算中使用空单元格时，它们会自动转换为零。要获得与 Excel 中相同的行为，请使用 `add` 方法并使用 `fill_value` 替换 `NaN` 值为零：

> `In``[``58``]:``rainfall``.``add``(``more_rainfall``,``fill_value``=``0``)`
> 
> `Out[58]:    City 1  City 2  City 3  City 4          0   300.1   400.3  1000.5     NaN          1   200.2   300.4  1100.6   200.0          2   300.0     NaN     NaN   400.0`

这对于其他算术运算符也适用，如表 [Table 5-4](https://example.org/filepos625873) 所示。

Table 5-4\. 算术运算符

|  Operator  |  Method  |
| --- | --- |
|   `*` |   `mul` |
|   `+` |   `add` |
|   `-` |   `sub` |
|   `/` |   `div` |
|   `**` |   `pow` |

当你在计算中有一个 DataFrame 和一个 Series 时，默认情况下 Series 会沿着索引进行广播：

> `In``[``59``]:``# 从一行中提取的 Series``rainfall``.``loc``[``1``,``:]`
> 
> `Out[59]: City 1     100.2          City 2     300.4          City 3    1100.6          Name: 1, dtype: float64`
> 
> `In``[``60``]:``rainfall``+``rainfall``.``loc``[``1``,``:]`
> 
> `Out[60]:    City 1  City 2  City 3          0   400.3   700.7  2101.1          1   200.4   600.8  2201.2`

因此，要按列添加一个 Series，你需要使用 `add` 方法并显式指定 `axis` 参数：

> `In``[``61``]:``# 从一列中提取的 Series``rainfall``.``loc``[:,``"City 2"``]`
> 
> `Out[61]: 0    400.3          1    300.4          Name: City 2, dtype: float64`
> 
> `In``[``62``]:``rainfall``.``add``(``rainfall``.``loc``[:,``"City 2"``],``axis``=``0``)`
> 
> `Out[62]:    City 1  City 2  City 3          0   700.4   800.6  1400.8          1   400.6   600.8  1401.0`

虽然本节讨论的是带有数字的 DataFrame 在算术运算中的行为，但下一节将展示你在处理 DataFrame 中文本时的选项。

处理文本列

正如我们在本章开头所见，包含文本或混合数据类型的列具有数据类型`object`。要对包含文本字符串的列执行操作，请使用`str`属性，该属性使您可以访问Python的字符串方法。我们在[第3章](index_split_010.html#filepos178328)中已经了解了一些字符串方法，但查看一下[Python文档](https://oreil.ly/-e7SC)中提供的方法也无妨。例如，要去除前导和尾随空格，请使用`strip`方法；要将所有首字母大写，可以使用`capitalize`方法。将这些方法链在一起将清理手动输入数据产生的混乱文本列：

> `In``[``63``]:``# 让我们创建一个新的DataFrame``users``=``pd``.``DataFrame``(``data``=``[``" mArk "``,``"JOHN  "``,``"Tim"``,``" jenny"``],``columns``=``[``"name"``])``users`
> 
> `Out[63]:      name          0   mArk          1  JOHN          2     Tim          3   jenny`
> 
> `In``[``64``]:``users_cleaned``=``users``.``loc``[:,``"name"``]``.``str``.``strip``()``.``str``.``capitalize``()``users_cleaned`
> 
> `Out[64]: 0     Mark          1     John          2      Tim          3    Jenny          Name: name, dtype: object`

或者，要查找所有以“J”开头的名称：

> `In``[``65``]:``users_cleaned``.``str``.``startswith``(``"J"``)`
> 
> `Out[65]: 0    False          1     True          2    False          3     True          Name: name, dtype: bool`

字符串方法很容易使用，但有时您可能需要以不内置的方式操作DataFrame。在这种情况下，创建自己的函数并将其应用于DataFrame，如下一节所示。

应用函数

数据框提供了`applymap`方法，该方法将应用于每个单独的元素，如果没有NumPy ufuncs可用，则非常有用。例如，没有用于字符串格式化的ufuncs，因此我们可以像这样格式化DataFrame的每个元素：

> `In``[``66``]:``rainfall`
> 
> `Out[66]:    City 1  City 2  City 3          0   300.1   400.3  1000.5          1   100.2   300.4  1100.6`
> 
> `In``[``67``]:``def``format_string``(``x``):``return``f``"{x:,.2f}"`
> 
> `In``[``68``]:``# 注意，我们传递函数时不要调用它，即format_string而不是format_string()！``rainfall``.``applymap``(``format_string``)`
> 
> `Out[68]:    City 1  City 2    City 3          0  300.10  400.30  1,000.50          1  100.20  300.40  1,100.60`

要分解这个过程：下面的f-string将`x`返回为一个字符串：`f"{x}"`。要添加格式化，将冒号附加到变量后面，然后是格式化字符串`,.2f`。逗号是千位分隔符，`.2f`表示小数点后两位的固定点表示法。要获取有关如何格式化字符串的更多详细信息，请参阅[格式规范迷你语言](https://oreil.ly/NgsG8)，它是Python文档的一部分。

对于这种用例，lambda 表达式（见侧边栏）被广泛使用，因为它们允许你在一行内写出同样的内容，而无需定义单独的函数。利用 lambda 表达式，我们可以将前面的例子重写为以下形式：

> `In``[``69``]:``降雨量``.``applymap``(``lambda``x``:``f``"{x:,.2f}"``)`
> 
> `Out[69]:    城市 1  城市 2    城市 3          0  300.10  400.30  1,000.50          1  100.20  300.40  1,100.60`
> 
> LAMBDA 表达式
> 
> Python 允许你通过 lambda 表达式在一行内定义函数。Lambda 表达式是匿名函数，这意味着它是一个没有名称的函数。考虑这个函数：
> 
> `def``函数名``(``参数1``,``参数2``,``...``):``return``返回值`
> 
> 这个函数可以重写为如下的 lambda 表达式：
> 
> `lambda``参数1``,``参数2``,``...``:``返回值`
> 
> 本质上，你用 lambda 替换 `def`，省略 `return` 关键字和函数名，并把所有内容放在一行上。就像我们在 `applymap` 方法中看到的那样，在这种情况下，这样做非常方便，因为我们不需要为仅被使用一次的事情定义一个函数。

我已经提到了所有重要的数据操作方法，但在我们继续之前，理解 pandas 何时使用数据框的视图和何时使用副本是很重要的。

视图 vs. 副本

你可能还记得上一章节中，切片 NumPy 数组返回一个视图。但是对于数据框而言，情况更加复杂：`loc` 和 `iloc` 是否返回视图或副本往往难以预测，这使得它成为比较令人困惑的话题之一。因为改变视图和数据框副本是很大的区别，当 pandas 认为你以不合适的方式设置数据时，它经常会提出如下警告：`SettingWithCopyWarning`。为了避免这种颇为神秘的警告，这里有一些建议：

+   > > > > 在原始数据框上设置值，而不是在从另一个数据框切片得到的数据框上设置值
+   > > > > 
+   > > > > 如果你想要在切片后得到一个独立的数据框，那么要显式地复制：
+   > > > > 
    > > > > `选择``=``df``.``loc``[:,``[``"国家"``,``"大陆"``]]``.``copy``()`

虽然在处理 `loc` 和 `iloc` 时情况复杂，但值得记住的是，所有数据框方法如 `df.dropna()` 或 `df.sort_values("列名")` 总是返回一个副本。

到目前为止，我们大多数时间都是在处理一个数据框。接下来的章节将展示多种将多个数据框合并为一个的方法，这是 pandas 提供的一个非常常见的强大工具。

合并数据框

在Excel中组合不同的数据集可能是一个繁琐的任务，通常涉及大量的`VLOOKUP`公式。幸运的是，pandas的合并DataFrame功能是其杀手级功能之一，其数据对齐能力将极大地简化你的生活，从而大大减少引入错误的可能性。合并和连接DataFrame可以通过各种方式进行；本节只讨论使用`concat`、`join`和`merge`的最常见情况。虽然它们有重叠之处，但每个函数都使特定任务变得非常简单。我将从`concat`函数开始，然后解释使用`join`的不同选项，最后介绍最通用的`merge`函数。

连接

简单地将多个DataFrame粘合在一起，`concat`函数是你的好帮手。正如函数名称所示，这个过程有一个技术名字叫做连接。默认情况下，`concat`沿着行将DataFrame粘合在一起，并自动对齐列。在下面的示例中，我创建了另一个DataFrame `more_users`，并将其附加到我们样本DataFrame `df`的底部：

> `In``[``70``]:``data``=``[[``15``,``"法国"``,``4.1``,``"贝基"``],``[``44``,``"加拿大"``,``6.1``,``"莉安"``]]``more_users``=``pd``.``DataFrame``(``data``=``data``,``columns``=``[``"年龄"``,``"国家"``,``"得分"``,``"姓名"``],``index``=``[``1000``,``1011``])``more_users`
> 
> `Out[70]:       年龄 国家  得分    姓名          1000   15  法国    4.1   贝基          1011   44  加拿大    6.1  莉安`
> 
> `In``[``71``]:``pd``.``concat``([``df``,``more_users``],``axis``=``0``)`
> 
> `Out[71]:         姓名  年龄  国家  得分 大陆          1001    马克   55    意大利    4.5    欧洲          1000    约翰   33      美国    6.7   美洲          1002     蒂姆   41      美国    3.9   美洲          1003   珍妮   12  德国    9.0    欧洲          1000   贝基   15   法国    4.1       NaN          1011  莉安   44   加拿大    6.1       NaN`

现在你注意到，由于`concat`在指定轴（行）上将数据粘合在一起，并且仅在另一个轴（列）上对齐数据，所以你现在有重复的索引元素！即使两个DataFrame中的列名不同序，它们也会自动匹配列名！如果你想沿着列将两个DataFrame粘合在一起，请设置`axis=1`：

> `In``[``72``]:``data``=``[[``3``,``4``],``[``5``,``6``]]``more_categories``=``pd``.``DataFrame``(``data``=``data``,``columns``=``[``"测验"``,``"登录"``],``index``=``[``1000``,``2000``])``more_categories`
> 
> `Out[72]:       测验  登录          1000        3       4          2000        5       6`
> 
> `In``[``73``]:``pd``.``concat``([``df``,``more_categories``],``axis``=``1``)`
> 
> `Out[73]:        name   age  country  score continent  quizzes  logins          1000   John  33.0      USA    6.7   America      3.0     4.0          1001   Mark  55.0    Italy    4.5    Europe      NaN     NaN          1002    Tim  41.0      USA    3.9   America      NaN     NaN          1003  Jenny  12.0  Germany    9.0    Europe      NaN     NaN          2000    NaN   NaN      NaN    NaN       NaN      5.0     6.0`

`concat` 的特殊且非常有用的特性是它可以接受超过两个 DataFrame。我们将在下一章节中使用它将多个 CSV 文件合并成一个单独的 DataFrame：

> `pd``.``concat``([``df1``,``df2``,``df3``,``...``])`

另一方面，`join` 和 `merge` 仅适用于两个 DataFrame，下面我们将看到。

连接与合并

当你连接两个 DataFrame 时，你将每个 DataFrame 的列合并到一个新的 DataFrame 中，同时根据集合理论决定行的处理方式。如果你之前有过与关系数据库的工作经验，那么这与 SQL 查询中的 `JOIN` 子句是相同的概念。[图 5-3](#filepos668317) 显示了内连接、左连接、右连接和外连接四种连接类型如何通过使用两个示例 DataFrame `df1` 和 `df2` 进行操作。

![](images/00002.jpg)

图 5-3\. 连接类型

使用 `join` 方法，pandas 使用两个 DataFrame 的索引来对齐行。内连接返回仅在索引重叠的行的 DataFrame。左连接获取左侧 DataFrame `df1` 的所有行，并在右侧 DataFrame `df2` 上匹配索引。在 `df2` 中没有匹配行的地方，pandas 将填充 `NaN`。左连接对应于 Excel 中的 `VLOOKUP` 情况。右连接获取右表 `df2` 的所有行，并将它们与 `df1` 的行在索引上匹配。最后，全外连接（即完全外连接）获取两个 DataFrame 的索引的并集，并在可能的情况下匹配值。[表 5-5](#filepos669531) 是文本形式中 [图 5-3](#filepos668317) 的等效内容。

表 5-5\. 连接类型

|  类型  |  描述  |
| --- | --- |
|   `inner` |  仅包含索引存在于两个 DataFrame 中的行  |
|   `left` |  从左 DataFrame 中获取所有行，匹配右 DataFrame 的行  |
|   `right` |  从右 DataFrame 中获取所有行，匹配左 DataFrame 的行  |
|   `outer` |  从两个 DataFrame 中获取所有行的并集  |

让我们看看实际操作中的情况，将 [图 5-3](#filepos668317) 中的示例活现出来：

> `In``[``74``]:``df1``=``pd``.``DataFrame``(``data``=``[[``1``,``2``],``[``3``,``4``],``[``5``,``6``]],``columns``=``[``"A"``,``"B"``])``df1`
> 
> `Out[74]:    A  B          0  1  2          1  3  4          2  5  6`
> 
> `In``[``75``]:``df2``=``pd``.``DataFrame``(``data``=``[[``10``,``20``],``[``30``,``40``]],``columns``=``[``"C"``,``"D"``],``index``=``[``1``,``3``])``df2`
> 
> `Out[75]:     C   D          1  10  20          3  30  40`
> 
> `In``[``76``]:``df1``.``join``(``df2``,``how``=``"inner"``)`
> 
> `Out[76]:    A  B   C   D          1  3  4  10  20`
> 
> `In``[``77``]:``df1``.``join``(``df2``,``how``=``"left"``)`
> 
> `Out[77]:    A  B     C     D          0  1  2   NaN   NaN          1  3  4  10.0  20.0          2  5  6   NaN   NaN`
> 
> `In``[``78``]:``df1``.``join``(``df2``,``how``=``"right"``)`
> 
> `Out[78]:      A    B   C   D          1  3.0  4.0  10  20          3  NaN  NaN  30  40`
> 
> `In``[``79``]:``df1``.``join``(``df2``,``how``=``"outer"``)`
> 
> `Out[79]:      A    B     C     D          0  1.0  2.0   NaN   NaN          1  3.0  4.0  10.0  20.0          2  5.0  6.0   NaN   NaN          3  NaN  NaN  30.0  40.0`

如果你想要根据一个或多个DataFrame列进行连接而不是依赖索引，使用`merge`而不是`join`。`merge`接受`on`参数作为连接条件：这些列必须存在于两个DataFrame中，并用于匹配行：

> `In``[``80``]:``# 给两个DataFrame都添加一个名为"category"的列``df1``[``"category"``]``=``[``"a"``,``"b"``,``"c"``]``df2``[``"category"``]``=``[``"c"``,``"b"``]`
> 
> `In``[``81``]:``df1`
> 
> `Out[81]:    A  B category          0  1  2        a          1  3  4        b          2  5  6        c`
> 
> `In``[``82``]:``df2`
> 
> `Out[82]:     C   D category          1  10  20        c          3  30  40        b`
> 
> `In``[``83``]:``df1``.``merge``(``df2``,``how``=``"inner"``,``on``=``[``"category"``])`
> 
> `Out[83]:    A  B category   C   D          0  3  4        b  30  40          1  5  6        c  10  20`
> 
> `In``[``84``]:``df1``.``merge``(``df2``,``how``=``"left"``,``on``=``[``"category"``])`
> 
> `Out[84]:    A  B category     C     D          0  1  2        a   NaN   NaN          1  3  4        b  30.0  40.0          2  5  6        c  10.0  20.0`

由于`join`和`merge`接受许多可选参数来适应更复杂的场景，我建议你查看[官方文档](https://oreil.ly/OZ4WV)以了解更多信息。

你现在知道如何操作一个或多个DataFrame，这将引导我们数据分析旅程的下一步：理解数据。

描述性统计和数据聚合

在理解大数据集的一种方法是计算描述统计，如总和或平均值，可以针对整个数据集或有意义的子集。本节首先介绍了在pandas中如何进行这种操作，然后介绍了两种数据聚合到子集的方式：`groupby`方法和`pivot_table`函数。

描述性统计

描述性统计允许你通过使用定量的方法对数据集进行总结。例如，数据点的数量是一个简单的描述性统计量。像均值、中位数或众数这样的平均数也是其他流行的例子。DataFrame和Series允许你通过诸如`sum`、`mean`和`count`之类的方法方便地访问描述性统计信息。在本书中你将会遇到许多这样的方法，并且完整的列表可以通过[pandas文档](https://oreil.ly/t2q9Q)获取。默认情况下，它们返回沿着`axis=0`的Series，这意味着你得到了列的统计信息：

> `In``[``85``]:``rainfall`
> 
> `Out[85]:    City 1  City 2  City 3          0   300.1   400.3  1000.5          1   100.2   300.4  1100.6`
> 
> `In``[``86``]:``rainfall``.``mean``()`
> 
> `Out[86]: City 1     200.15          City 2     350.35          City 3    1050.55          dtype: float64`

如果需要每行的统计信息，请提供`axis`参数：

> `In``[``87``]:``rainfall``.``mean``(``axis``=``1``)`
> 
> `Out[87]: 0    566.966667          1    500.400000          dtype: float64`

默认情况下，描述性统计不包括缺失值，这与Excel处理空单元格的方式一致，因此在带有空单元格的范围上使用Excel的`AVERAGE`公式将给出与在具有相同数字和`NaN`值而不是空单元格的Series上应用`mean`方法相同的结果。

有时仅仅获取DataFrame所有行的统计数据是不够的，你需要更详细的信息——例如每个类别的平均值。让我们看看如何实现！

分组

使用我们的示例DataFrame `df`，让我们再次找出每个大陆的平均分数！为此，首先按大陆分组行，然后应用`mean`方法，该方法将计算每个组的平均值。所有非数字列将自动排除：

> `In``[``88``]:``df``.``groupby``([``"continent"``])``.``mean``()`
> 
> `Out[88]: properties   age  score          continent          America     37.0   5.30          Europe      33.5   6.75`

如果包括多于一个列，生成的DataFrame将具有层次化索引——我们之前遇到的MultiIndex：

> `In``[``89``]:``df``.``groupby``([``"continent"``,``"country"``])``.``mean``()`
> 
> `Out[89]: properties         age  score          continent country          America   USA       37    5.3          Europe    Germany   12    9.0                    Italy     55    4.5`

你可以使用大多数由pandas提供的描述性统计量，如果想使用自己的函数，可以使用`agg`方法。例如，这是如何获取每个组最大值与最小值之差的方法：

> `In``[``90``]:``df``.``groupby``([``"continent"``])``.``agg``(``lambda``x``:``x``.``max``()``-``x``.``min``())`
> 
> `Out[90]: properties  age  score          continent          America       8    2.8          Europe       43    4.5`

在 Excel 中，获取每个分组的统计数据的一种流行方式是使用数据透视表。它们引入了第二个维度，非常适合从不同角度查看数据。pandas 也有数据透视表功能，我们接下来会看到。

数据透视和数据融合

如果您在Excel中使用数据透视表，可以毫不费力地应用 pandas 的`pivot_table`函数，因为它的使用方式基本相同。下面的 DataFrame 中的数据组织方式与通常在数据库中存储记录的方式类似；每一行显示了特定水果在特定地区的销售交易：

> `In``[``91``]:``data``=``[[``"Oranges"``,``"North"``,``12.30``],``[``"Apples"``,``"South"``,``10.55``],``[``"Oranges"``,``"South"``,``22.00``],``[``"Bananas"``,``"South"``,``5.90``],``[``"Bananas"``,``"North"``,``31.30``],``[``"Oranges"``,``"North"``,``13.10``]]``sales``=``pd``.``DataFrame``(``data``=``data``,``columns``=``[``"Fruit"``,``"Region"``,``"Revenue"``])``sales`
> 
> `Out[91]:      Fruit Region  Revenue          0  Oranges  North    12.30          1   Apples  South    10.55          2  Oranges  South    22.00          3  Bananas  South     5.90          4  Bananas  North    31.30          5  Oranges  North    13.10`

要创建数据透视表，将 DataFrame 作为`pivot_table`函数的第一个参数提供。`index`和`columns`定义了 DataFrame 的哪一列将成为数据透视表的行和列标签。`values`将根据`aggfunc`聚合到生成的 DataFrame 的数据部分中，`aggfunc`是一个可以作为字符串或 NumPy ufunc 提供的函数。最后，`margins`对应于 Excel 中的`Grand Total`，即如果不指定`margins`和`margins_name`，则不会显示`Total`列和行：

> `In``[``92``]:``pivot``=``pd``.``pivot_table``(``sales``,``index``=``"Fruit"``,``columns``=``"Region"``,``values``=``"Revenue"``,``aggfunc``=``"sum"``,``margins``=``True``,``margins_name``=``"Total"``)``pivot`
> 
> `Out[92]: Region   North  South  Total          Fruit          Apples     NaN  10.55  10.55          Bananas   31.3   5.90  37.20          Oranges   25.4  22.00  47.40          Total     56.7  38.45  95.15`

总之，对数据进行透视意味着获取某一列的唯一值（在我们的例子中是`Region`），并将其转换为数据透视表的列标题，从而聚合另一列的值。这样可以轻松地查看所关心维度的汇总信息。在我们的数据透视表中，您立即可以看到北部地区没有销售苹果，而南部地区的大部分收入来自橙子。如果您希望反过来，将列标题转换为单列的值，请使用`melt`。从这个意义上说，`melt`是`pivot_table`函数的反义词：

> `In``[``93``]:``pd``.``melt``(``pivot``.``iloc``[:``-``1``,:``-``1``]``.``reset_index``(),``id_vars``=``"Fruit"``,``value_vars``=``[``"North"``,``"South"``],``value_name``=``"Revenue"``)`
> 
> `Out[93]:      Fruit Region  Revenue          0   Apples  North      NaN          1  Bananas  North    31.30          2  Oranges  North    25.40          3   Apples  South    10.55          4  Bananas  South     5.90          5  Oranges  South    22.00`

在这里，我将我们的透视表作为输入提供，但我使用`iloc`来去除总行和列。我还重置索引，以便所有信息都作为常规列可用。然后，我提供`id_vars`以指示标识符，并提供`value_vars`以定义我要“解构”的列。如果您希望准备数据以便将其存储回预期以此格式存储的数据库中，熔断可能会很有用。

使用聚合统计数据有助于理解数据，但没有人喜欢阅读满篇数字。为了使信息易于理解，创建可视化效果最有效，这是我们接下来要讨论的内容。虽然Excel使用术语图表，但pandas通常称之为图。在本书中，我会交替使用这些术语。

绘图

绘图允许您可视化数据分析的发现，可能是整个过程中最重要的一步。对于绘图，我们将使用两个库：首先是Matplotlib，pandas的默认绘图库，然后是Plotly，这是一个现代绘图库，在Jupyter笔记本中提供更交互式的体验。

Matplotlib

Matplotlib是一个长期存在的绘图包，包含在Anaconda发行版中。您可以使用它生成多种格式的图表，包括高质量打印的矢量图形。当您调用DataFrame的`plot`方法时，pandas默认会生成一个Matplotlib图。

要在Jupyter笔记本中使用Matplotlib，您需要首先运行两个魔术命令中的一个（参见侧边栏[“魔术命令”](#filepos726690)）：`%matplotlib inline`或`%matplotlib notebook`。它们配置笔记本以便在笔记本本身中显示图表。后者命令增加了一些交互性，允许您更改图表的大小或缩放系数。让我们开始，并使用pandas和Matplotlib创建第一个图表（见[图5-4](#filepos726089)）：

> `In``[``94``]:``import``numpy``as``np``%``matplotlib``inline``# Or %matplotlib notebook`
> 
> `In``[``95``]:``data``=``pd``.``DataFrame``(``data``=``np``.``random``.``rand``(``4``,``4``)``*``100000``,``index``=``[``"Q1"``,``"Q2"``,``"Q3"``,``"Q4"``],``columns``=``[``"East"``,``"West"``,``"North"``,``"South"``])``data``.``index``.``name``=``"Quarters"``data``.``columns``.``name``=``"Region"``data`
> 
> `Out[95]: Region            East          West         North         South          Quarters          Q1        23254.220271  96398.309860  16845.951895  41671.684909          Q2        87316.022433  45183.397951  15460.819455  50951.465770          Q3        51458.760432   3821.139360  77793.393899  98915.952421          Q4        64933.848496   7600.277035  55001.831706  86248.512650`
> 
> `In``[``96``]:``data``.``plot``()``# 快捷方式用于 data.plot.line()``
> 
> `Out[96]: <AxesSubplot:xlabel='Quarters'>`

![](images/00008.jpg)

图 5-4\. Matplotlib 绘图

请注意，在此示例中，我使用了 NumPy 数组来构建 pandas DataFrame。提供 NumPy 数组允许你利用上一章介绍的 NumPy 构造函数；在这里，我们使用 NumPy 根据伪随机数生成了一个 pandas DataFrame。因此，当你在自己的环境中运行示例时，会得到不同的值。

> 魔术命令
> 
> 我们在 Jupyter 笔记本中使用的 `%matplotlib inline` 命令是一个魔术命令。魔术命令是一组简单的命令，可以让 Jupyter 笔记本单元格以特定方式运行，或者使繁琐的任务变得异常简单，几乎像魔术一样。你可以像写 Python 代码一样在单元格中编写这些命令，但它们以 `%%` 或 `%` 开头。影响整个单元格的命令以 `%%` 开头，而仅影响单行的命令以 `%` 开头。
> 
> 我们将在接下来的章节中看到更多的魔术命令，但如果你想列出当前所有可用的魔术命令，请运行 `%lsmagic`，并运行 `%magic` 获取详细说明。

即使使用 `%matplotlib notebook` 魔术命令，你可能会注意到 Matplotlib 最初设计用于静态绘图，而不是在网页上进行交互体验。这就是为什么接下来我们将使用 Plotly，这是一个专为 Web 设计的绘图库。

Plotly

Plotly 是基于 JavaScript 的库，自版本 4.8.0 起，可以作为 pandas 的绘图后端，具有出色的交互性：你可以轻松缩放，单击图例以选择或取消选择类别，并获取有关悬停数据点的更多信息。Plotly 不包含在 Anaconda 安装中，因此如果尚未安装，请通过运行以下命令安装：

> `(base)>` `conda install plotly`

运行此单元格后，整个笔记本的绘图后端将设置为 Plotly，如果重新运行之前的单元格，它也将呈现为 Plotly 图表。对于 Plotly，你只需在能够绘制 [5-5](#filepos731078) 和 [5-6](#filepos732496) 图形之前将其设置为后端：

> `In``[``97``]:``# 将绘图后端设置为 Plotly``pd``.``options``.``plotting``.``backend``=``"plotly"`
> 
> `In``[``98``]:``data``.``plot``()``

![](images/00016.jpg)

图 5-5\. Plotly 折线图

> `In``[``99``]:``# 显示相同数据的条形图``data``.``plot``.``bar``(``barmode``=``"group"``)`

![](images/00021.jpg)

图 5-6\. Plotly 条形图

> 绘图后端的差异
> 
> 如果您使用 Plotly 作为绘图后端，您需要直接在 Plotly 文档中检查绘图方法的接受参数。例如，您可以查看 [Plotly 条形图文档](https://oreil.ly/Ekurd) 中的 `barmode=group` 参数。

pandas 和底层绘图库提供了丰富的图表类型和选项，可以以几乎任何期望的方式格式化图表。也可以将多个图表安排到子图系列中。总览见 [表 5-6](#filepos733705) 显示了可用的绘图类型。

表 5-6\. pandas 绘图类型

|  类型  |  描述  |
| --- | --- |
|   `line` |  线性图表，默认运行 `df.plot()` 时的默认设置 |
|   `bar` |  垂直条形图  |
|   `barh` |  水平条形图  |
|   `hist` |  直方图  |
|   `box` |  箱线图  |
|   `kde` |  密度图，也可以通过 `density` 使用 |
|   `area` |  面积图  |
|   `scatter` |  散点图  |
|   `hexbin` |  六角形箱形图  |
|   `pie` |  饼图  |

此外，pandas 还提供了一些由多个独立组件组成的高级绘图工具和技术。详情请见 [pandas 可视化文档](https://oreil.ly/FxYg9)。

> 其他绘图库
> 
> Python 的科学可视化领域非常活跃，除了 Matplotlib 和 Plotly 外，还有许多其他高质量的选择，可能更适合特定用例：
> 
> Seaborn
> 
> > [Seaborn](https://oreil.ly/a3U1t) 建立在 Matplotlib 之上。它改进了默认样式，并添加了额外的绘图，如热图，通常简化了您的工作：您可以仅使用几行代码创建高级统计图。
> > 
> Bokeh
> 
> > [Bokeh](https://docs.bokeh.org) 类似于 Plotly 在技术和功能上：它基于 JavaScript，因此在 Jupyter 笔记本中也非常适合交互式图表。Bokeh 包含在 Anaconda 中。
> > 
> Altair
> 
> > [Altair](https://oreil.ly/t06t7) 是一个基于 [Vega 项目](https://oreil.ly/RN6A7) 的统计可视化库。Altair 也是基于 JavaScript 的，提供一些像缩放这样的交互功能。
> > 
> HoloViews
> 
> > [HoloViews](https://holoviews.org) 是另一个基于 JavaScript 的包，专注于使数据分析和可视化变得简单。只需几行代码，您就可以实现复杂的统计图。

我们将在下一章节创建更多的图表来分析时间序列，但在此之前，让我们通过学习如何使用 pandas 导入和导出数据来结束本章节！

导入和导出数据帧

到目前为止，我们使用嵌套列表、字典或NumPy数组从头构建了 DataFrame。了解这些技术很重要，但通常数据已经可用，您只需将其转换为 DataFrame。为此，pandas 提供了各种读取函数。但即使您需要访问 pandas 不提供内置读取器的专有系统，您通常也可以使用 Python 包连接到该系统，一旦获取数据，将其转换为 DataFrame 就很容易。在 Excel 中，数据导入通常是使用 Power Query 处理的工作类型。

分析和更改数据集后，您可能希望将结果推回数据库或将其导出为 CSV 文件或者——考虑到本书的标题——将其呈现在 Excel 工作簿中供您的经理查看。要导出 pandas DataFrame，请使用DataFrame提供的导出器方法之一。[表格 5-7](#filepos741567)显示了最常见的导入和导出方法概述。

表格 5-7\. 导入和导出数据框

|  数据格式/系统  |  导入：pandas（pd）函数  |  导出：DataFrame（df）方法  |
| --- | --- | --- |
|  CSV 文件  |   `pd.read_csv` |   `df.to_csv` |
|  JSON  |   `pd.read_json` |   `df.to_json` |
|  HTML  |   `pd.read_html` |   `df.to_html` |
|  剪贴板  |   `pd.read_clipboard` |   `df.to_clipboard` |
|  Excel 文件  |   `pd.read_excel` |   `df.to_excel` |
|  SQL 数据库  |   `pd.read_sql` |   `df.to_sql` |

我们将在[第 11 章](index_split_027.html#filepos1487255)中遇到 `pd.read_sql` 和 `pd.to_sql`，在那里我们将把它们作为案例研究的一部分使用。并且由于我打算在整个[第 7 章](index_split_019.html#filepos863345)中专门讨论使用 pandas 读取和写入 Excel 文件的主题，因此在本节中我将重点讨论导入和导出 CSV 文件。让我们从导出现有的 DataFrame 开始！

导出 CSV 文件

如果您需要将 DataFrame 传递给可能不使用 Python 或 pandas 的同事，以 CSV 文件的形式传递通常是个好主意：几乎每个程序都知道如何导入它们。要将我们的示例 DataFrame `df` 导出到 CSV 文件，使用 `to_csv` 方法：

> `In``[``100``]:``df``.``to_csv``(``"course_participants.csv"``)`

如果您想将文件存储在不同的目录中，请提供完整路径作为原始字符串，例如，`r"C:\path\to\desired\location\msft.csv"`。

> 在 Windows 上使用原始字符串处理文件路径
> 
> 在字符串中，反斜杠用于转义某些字符。这就是为什么在 Windows 上处理文件路径时，您需要使用双反斜杠（`C:\\path\\to\\file.csv`）或在字符串前加上`r`来将其转换为原始字符串，以字面上解释字符。这在 macOS 或 Linux 上不是问题，因为它们在路径中使用正斜杠。

通过仅提供文件名如我所做的那样，它将在与笔记本相同的目录中生成文件`course_participants.csv`，内容如下：

> `user_id,name,age,country,score,continent 1001,Mark,55,Italy,4.5,Europe 1000,John,33,USA,6.7,America 1002,Tim,41,USA,3.9,America 1003,Jenny,12,Germany,9.0,Europe`

现在您已经了解如何使用`df.to_csv`方法，让我们看看如何导入 CSV 文件！

导入 CSV 文件

导入本地 CSV 文件就像将其路径提供给`read_csv`函数一样简单。MSFT.csv 是我从 Yahoo! Finance 下载的 CSV 文件，包含了微软的日常历史股价 —— 你可以在伴随的存储库中的 csv 文件夹找到它：

> `In``[``101``]:``msft``=``pd``.``read_csv``(``"csv/MSFT.csv"``)`

经常需要向`read_csv`提供比仅文件名更多的参数。例如，`sep`参数允许您告诉 pandas CSV 文件使用的分隔符或分隔符，以防它不是默认的逗号。在下一章中，我们将使用更多的参数，但是为了全面了解，请查看[pandas文档](https://oreil.ly/2GMhW)。

现在我们正在处理具有数千行的大型 DataFrame，通常第一步是运行`info`方法以获取 DataFrame 的摘要信息。接下来，您可能希望使用`head`和`tail`方法查看 DataFrame 的前几行和最后几行。这些方法默认返回五行，但可以通过提供所需行数作为参数来更改。您还可以运行`describe`方法获取一些基本统计信息：

> `In``[``102``]:``msft``.``info``()`
> 
> `<class 'pandas.core.frame.DataFrame'> RangeIndex: 8622 entries, 0 to 8621 Data columns (total 7 columns): #   Column     Non-Null Count  Dtype ---  ------     --------------  ----- 0   Date       8622 non-null   object 1   Open       8622 non-null   float64 2   High       8622 non-null   float64 3   Low        8622 non-null   float64 4   Close      8622 non-null   float64 5   Adj Close  8622 non-null   float64 6   Volume     8622 non-null   int64 dtypes: float64(5), int64(1), object(1) memory usage: 471.6+ KB`
> 
> `In``[``103``]:``# 由于空间问题，我只选择了几列``# 您也可以直接运行：msft.head()``msft``.``loc``[:,``[``"Date"``,``"Adj Close"``,``"Volume"``]]``.``head``()`
> 
> `Out[103]:          Date  Adj Close      Volume           0  1986-03-13   0.062205  1031788800           1  1986-03-14   0.064427   308160000           2  1986-03-17   0.065537   133171200           3  1986-03-18   0.063871    67766400           4  1986-03-19   0.062760    47894400`
> 
> `In``[``104``]:``msft``.``loc``[:,``[``"Date"``,``"Adj Close"``,``"Volume"``]]``.``tail``(``2``)`
> 
> `Out[104]:             Date   Adj Close    Volume           8620  2020-05-26  181.570007  36073600           8621  2020-05-27  181.809998  39492600`
> 
> `In``[``105``]:``msft``.``loc``[:,``[``"Adj Close"``,``"Volume"``]]``.``describe``()`
> 
> `Out[105]:          Adj Close        Volume           count  8622.000000  8.622000e+03           mean     24.921952  6.030722e+07           std      31.838096  3.877805e+07           min       0.057762  2.304000e+06           25%       2.247503  3.651632e+07           50%      18.454313  5.350380e+07           75%      25.699224  7.397560e+07           max     187.663330  1.031789e+09`

`Adj Close` 代表调整后的收盘价格，校正了股票价格如股票拆分等公司行动。 `Volume` 是交易的股票数量。我总结了本章中看到的各种 DataFrame 探索方法在 [表 5-8](#filepos758607) 中。

表格 5-8\. DataFrame 探索方法和属性

|  DataFrame（df）方法/属性  |  描述  |
| --- | --- |
|   `df.info()` |  提供数据点数量、索引类型、数据类型和内存使用情况。  |
|   `df.describe()` |  提供基本统计信息，包括计数、均值、标准差、最小值、最大值和百分位数。  |
|   `df.head(n=5)` |  返回DataFrame的前  n 行。 |
|   `df.tail(n=5)` |  返回DataFrame的最后  n 行。 |
|   `df.dtypes` |  返回每列的数据类型。  |

`read_csv` 函数还可以接受 URL 而不是本地 CSV 文件。以下是直接从配套仓库读取 CSV 文件的方法：

> `In``[``106``]:``# URL 中的换行符仅是为了使其适合页面``url``=``(``"https://raw.githubusercontent.com/fzumstein/"``"python-for-excel/1st-edition/csv/MSFT.csv"``)``msft``=``pd``.``read_csv``(``url``)`
> 
> `In``[``107``]:``msft``.``loc``[:,``[``"Date"``,``"Adj Close"``,``"Volume"``]]``.``head``(``2``)`
> 
> `Out[107]:          Date  Adj Close      Volume           0  1986-03-13   0.062205  1031788800           1  1986-03-14   0.064427   308160000`

在下一章关于时间序列的章节中，我们将继续使用此数据集和 `read_csv` 函数，将 `Date` 列转换为 `DatetimeIndex`。

结论

本章充满了分析 pandas 数据集的新概念和工具。我们学会了如何加载 CSV 文件，如何处理缺失或重复数据，以及如何利用描述性统计信息。我们还看到了如何将 DataFrame 转换为交互式图表。虽然消化这些内容可能需要一些时间，但加入 pandas 工具箱后，您将很快理解其强大之处。在此过程中，我们将 pandas 与以下 Excel 功能进行了比较：

自动筛选功能

> > 参见 [“通过布尔索引进行选择”](index_split_015.html#filepos549613)。

VLOOKUP 公式

> > 参见 [“连接和合并”](#filepos667627)。

数据透视表

> > 参见 [“数据透视和融合”](#filepos701398)。

Power Query

> > 这是 [“导入和导出数据框”](#filepos740294)、[“数据操作”](index_split_015.html#filepos524268) 和 [“合并数据框”](#filepos652519) 的结合。

下一章讲述的是时间序列分析，这一功能使得 pandas 在金融行业广泛应用。让我们看看为什么 pandas 的这一部分比 Excel 更具优势！

> [1  ](index_split_015.html#filepos498666) pandas 1.0.0 引入了专门的`string`数据类型，以使某些操作更容易且与文本更一致。由于它仍处于实验阶段，我在本书中不打算使用它。
