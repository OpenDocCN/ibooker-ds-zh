# 第 8 章。pandas 简介

pandas 是我们数据可视化工具链中的关键组成部分，因为我们将使用它来清理和探索我们最近抓取的数据集（参见 [第 6 章](ch06.xhtml#chapter_heavy_scraping)）。上一章介绍了 NumPy，这是 Python 的数组处理库，也是 pandas 的基础。在我们应用 pandas 之前，本章将介绍其关键概念，并展示它如何与现有数据文件和数据库表进行交互。接下来的几章将继续在实际工作中学习 pandas。

# pandas 为何专为数据可视化定制

无论是基于网络还是印刷的任何数据可视化，很有可能被可视化的数据最初都存储在类似 Excel、CSV 文件或 HDF5 的行列式电子表格中。当然，也有一些可视化方式，如网络图，对于行列式数据不是最佳形式，但它们属于少数。pandas 专为操作行列式数据表而设计，其核心数据类型是 DataFrame，最好将其视为非常快速的编程电子表格。

# pandas 的开发动机

由 Wes Kinney 在 2008 年首次公开，pandas 是为解决一个特定问题而构建的——即虽然 Python 在数据操作方面表现出色，但在数据分析和建模方面相对较弱，尤其是与 R 等强大工具相比。

pandas 设计用于处理类似行列式电子表格中发现的异构^([1](ch08.xhtml#idm45607780439264))数据，但巧妙地利用了 NumPy 的同质数值数组的一些速度优势，这些数组被数学家、物理学家、计算机图形学等广泛使用。结合 Jupyter 笔记本和 Matplotlib 绘图库（以及像 seaborn 这样的辅助库），pandas 是一款一流的交互式数据分析工具。作为 NumPy 生态系统的一部分，它的数据建模可以轻松地通过诸如 SciPy、statsmodels 和 scikit-learn 等库进行增强。

# 数据与测量的分类

在接下来的章节中，我将介绍 pandas 的核心概念，重点讨论 DataFrame 以及如何通过常见的数据存储方式（如 CSV 文件和 SQL 数据库）将数据导入和导出。但首先，让我们稍作偏离，思考一下 pandas 设计用来处理的异构数据集的真正含义，这也是数据可视化的重要基础。

有可能会有一些可视化，比如柱状图或线图用来说明文章或现代网络仪表板中测量结果的变化，商品价格随时间的变化，一年中降雨量的变化，不同族裔的投票意向等。这些测量结果大致可以分为两组，数值型和分类型。数值型可以分为区间和比率尺度，分类值则可以进一步分为名义和有序测量。这样数据可视化者可以得到四种广泛的观察类别。

让我们以一组推文为例，以便提取这些测量类别。每条推文都有各种数据字段：

```py
{
  "text": "#Python and #JavaScript sitting in a tree...", ![1](assets/1.png)
  "id": 2103303030333004303, ![1](assets/1.png)
  "favorited": true, ![2](assets/2.png)
  "filter_level":"medium", ![3](assets/3.png)
  "created_at": "Wed Mar 23 14:07:43 +0000 2015", ![4](assets/4.png)
  "retweet_count":23, ![5](assets/5.png)
  "coordinates":[-97.5, 45.3] ![6](assets/6.png)
  ...
}
```

[![1](assets/1.png)](#co_introduction_to_pandas_CO1-1)

`text`和`id`字段是唯一的标识符。前者可能包含分类信息（比如包含#Python标签的推文类别），而后者可能用于创建一个类别（比如所有转发该推文的用户集合），但它们本身不是可视化字段。

[![2](assets/2.png)](#co_introduction_to_pandas_CO1-3)

`favorited`是布尔值，分类信息，将推文分为两组。这算作是*名义*类别，因为可以计数但不能排序。

[![3](assets/3.png)](#co_introduction_to_pandas_CO1-4)

`filter_level`也是分类信息，但它是有序的。过滤级别有低→中→高的顺序。

[![4](assets/4.png)](#co_introduction_to_pandas_CO1-5)

`created_at`字段是时间戳，数值尺度上的区间值。我们可能希望按照这个尺度对推文进行排序，这是pandas会自动完成的，并且可能会划分为更广泛的间隔，比如按天或者按周。再次强调，pandas使这变得非常简单。

[![5](assets/5.png)](#co_introduction_to_pandas_CO1-6)

`retweet_count`同样是数值尺度，但是是比率尺度。比率尺度与区间尺度相反，有一个有意义的零点——在这种情况下是没有转发。而我们的`created_at`时间戳则可以有一个任意的基线（比如unix时间或公历的年份0），就像温度尺度一样，摄氏度的0度等同于开尔文的273.15度。

[![6](assets/6.png)](#co_introduction_to_pandas_CO1-7)

`coordinates`如果有的话，有两个经纬度数值尺度。两者都是区间尺度，虽然讨论角度的比率没有太多意义。

因此，我们的简单推文字段的小子集包含了覆盖所有通常接受的测量分区的异质信息。而NumPy数组通常用于同质化的数值计算，pandas则设计用于处理分类数据、时间序列和反映现实世界数据异质性的项目。这使其非常适合数据可视化。

现在我们知道 pandas 设计用于处理的数据类型，让我们看看它使用的数据结构。

# DataFrame

在 pandas 会话中的第一步通常是将一些数据加载到 DataFrame 中。我们将在后面的部分中介绍我们可以做到这一点的各种方法。现在，让我们从文件中读取我们的 *nobel_winners.json* JSON 数据。`read_json` 返回一个从指定的 JSON 文件解析的 DataFrame。按照惯例，DataFrame 变量以 `df` 开头：

```py
import pandas as pd

df = pd.read_json('data/nobel_winners.json')
```

有了我们的 DataFrame，让我们检查其内容。获取 DataFrame 的行列结构的快速方法是使用其 `head` 方法显示（默认情况下）前五个项目。[图 8-1](#pandas_dataframe) 显示了来自 [Jupyter 笔记本](https://jupyter.org) 的输出，突出显示了 DataFrame 的关键元素。

![dpj2 0801](assets/dpj2_0801.png)

###### 图 8-1\. pandas DataFrame 的关键元素

## 索引

DataFrame 的列通过 `columns` 属性进行索引，这是一个 pandas `index` 实例。让我们选择 [图 8-1](#pandas_dataframe) 中的列：

```py
In [0]: df.columns
Out[0]: Index(['born_in', 'category', ... ], dtype='object')
```

最初，pandas 行具有一个单一的数值索引（如果需要，pandas 可以处理多个索引），可以通过 `index` 属性访问。默认情况下，这是一个节省内存的 [`RangeIndex`](https://oreil.ly/7Qzia)：

```py
In [1]: df.index
Out[1]: RangeIndex(start=0, stop=1052, step=1)
```

除了整数外，行索引还可以是字符串、`DatetimeIndex` 或 `PeriodIndex` 用于基于时间的数据，等等。通常，为了帮助选择，DataFrame 的一列将通过 `set_index` 方法设置为索引。在以下代码中，我们首先使用 `set_index` 方法将我们的 Nobel DataFrame 的索引设置为名称列，然后使用 `loc` 方法按索引标签选择一行（在本例中为 `name`）：

```py
In [2] df = df.set_index('name') ![1](assets/1.png)
In [3] df.loc['Albert Einstein'] ![2](assets/2.png)
Out[3]:
                born_in category      country date_of_birth date_of_death  \ name
Albert Einstein          Physics  Switzerland    1879-03-14    1955-04-18
Albert Einstein          Physics      Germany    1879-03-14    1955-04-18
[...]

df = df.reset_index() ![3](assets/3.png)
```

[![1](assets/1.png)](#co_introduction_to_pandas_CO2-1)

将索引设置为名称列。

[![2](assets/2.png)](#co_introduction_to_pandas_CO2-2)

现在您可以按 `name` 标签选择一行。

[![3](assets/3.png)](#co_introduction_to_pandas_CO2-3)

将索引返回到原始基于整数的状态。

## 行和列

DataFrame 的行和列存储为 [pandas Series](https://oreil.ly/z7PF4)，这是 NumPy 数组的异构对应物。这些本质上是带有标签的一维数组，可以包含从整数、字符串和浮点数到 Python 对象和列表的任何数据类型。

有两种方法可以从 DataFrame 中选择一行。我们已经看到了 `loc` 方法，它通过标签进行选择。还有一个 `iloc` 方法，它通过位置进行选择。因此，要选择 [图 8-1](#pandas_dataframe) 中的行，我们获取第二行：

```py
In [4] df.iloc[2]
Out[4]:
name                                              Vladimir Prelog *
born_in                                      Bosnia and Herzegovina
category                                                  Chemistry
country
date_of_birth                                         July 23, 1906
...
year                                                           1975
Name: 2, dtype: object
```

您可以使用点符号^([2](ch08.xhtml#idm45607779978240))或传统的关键字字符串数组访问方法获取 DataFrame 的列。这将返回一个 pandas Series，其中包含所有列字段，并保留其 DataFrame 索引：

```py
In [9] gender_col = df.gender # or df['gender']
In [10] type(gender_col)
Out[10] pandas.core.series.Series
In [11] gender_col.head() # grab the Series' first five items
Out[11]:
0    male #index, object
1    male
2    male
3    None
4    male
Name: gender, dtype: object
```

## 选择分组

有各种方法可以选择我们DataFrame的组（或行子集）。通常我们想要选择具有特定列值的所有行（例如，所有物理学类别的行）。一种方法是使用DataFrame的`groupby`方法对列（或列列表）进行分组，然后使用`get_group`方法选择所需的组。让我们使用这两种方法选择所有诺贝尔物理学奖获得者：

```py
cat_groups = df.groupby('category')
cat_groups
#Out[-] <pandas.core.groupby.generic.DataFrameGroupBy object ...>

cat_groups.groups.keys()
#Out[-]: dict_keys(['', 'Chemistry', 'Economics', 'Literature',\
#                  'Peace', 'Physics', 'Physiology or Medicine'])
 ...

In [14] phy_group = cat_groups.get_group('Physics')
In [15] phy_group.head()
Out[15]:
                 name born_in category  country    date_of_birth  \
13   François Englert          Physics  Belgium  6 November 1932
19         Niels Bohr          Physics  Denmark   7 October 1885
23  Ben Roy Mottelson          Physics  Denmark     July 9, 1926
24          Aage Bohr          Physics  Denmark     19 June 1922
47     Alfred Kastler          Physics   France       3 May 1902
...
```

另一种选择行子集的方法是使用布尔掩码来创建新的DataFrame。你可以像在NumPy数组中对所有成员应用布尔运算符一样，对DataFrame中的所有行应用布尔运算符：

```py
In [16] df.category == 'Physics'
Out[16]:
0     False
1     False
...
1047   True
...
```

然后，可以将生成的布尔掩码应用于原始DataFrame，以选择其行的子集：

```py
In [17]: df[df.category == 'Physics']
Out[17]:
                          name    born_in category    country  \
13            François Englert             Physics    Belgium
19                  Niels Bohr             Physics    Denmark
23           Ben Roy Mottelson             Physics    Denmark
24                   Aage Bohr             Physics    Denmark
...
1047          Brian P. Schmidt             Physics  Australia   ...
```

在接下来的章节中，我们将介绍更多关于数据选择的例子。现在，让我们看看如何从现有数据创建DataFrame以及如何保存我们数据框架操作的结果。

# 创建和保存DataFrame

创建DataFrame最简单的方法是使用Python字典。这也是您可能不经常使用的一种方法，因为您可能会从文件或数据库访问数据。尽管如此，它有其用途。

默认情况下，我们分别指定列，在以下示例中创建三行名称和类别列：

```py
df = pd.DataFrame({
     'name': ['Albert Einstein', 'Marie Curie',\
     'William Faulkner'],
     'category': ['Physics', 'Chemistry', 'Literature']
     })
```

我们可以使用`from_dict`方法允许我们使用我们首选的基于记录的对象数组。`from_dict`有一个`orient`参数，允许我们指定类似记录的数据，但pandas足够智能，可以解析出数据形式：

```py
df = pd.DataFrame.from_dict([ ![1](assets/1.png)
     {'name': 'Albert Einstein', 'category':'Physics'},
     {'name': 'Marie Curie', 'category':'Chemistry'},
     {'name': 'William Faulkner', 'category':'Literature'}
    ])
```

[![1](assets/1.png)](#co_introduction_to_pandas_CO3-1)

在这里，我们传入一个对象数组，每个对象对应我们DataFrame中的一行。

刚才展示的方法产生了一个相同的DataFrame：

```py
df.head()
Out:
               name    category
0   Albert Einstein     Physics
1       Marie Curie   Chemistry
2  William Faulkner  Literature
```

正如提到的，您可能不会直接从Python容器创建DataFrame。相反，您可能会使用pandas数据读取方法之一。

pandas拥有令人印象深刻的`read_[format]`/`to_[format]`方法，涵盖了大多数可想象的数据加载用例，从CSV到二进制HDF5再到SQL数据库。我们将介绍与数据可视化工作最相关的子集。有关完整列表，请参阅[pandas文档](https://oreil.ly/b3VFR)。

默认情况下，pandas会尝试合理地转换加载的数据。`convert_axes`（尝试将轴转换为适当的`dtype`）、`dtype`（猜测数据类型）和`convert_dates`参数在读取方法中默认都是`True`。请参阅[pandas文档](https://oreil.ly/MkmIx)以获取可用选项的示例，本例是为了将JSON文件读入DataFrame。

让我们首先涵盖基于文件的DataFrame，然后看看如何与（非）SQL数据库交互。

## JSON

在pandas中轻松加载我们首选的JSON格式数据：

```py
df = pd.read_json('file.json')
```

JSON文件可以采用各种形式，由可选的`orient`参数指定，其中之一为[`split`, `records`, `index`, `columns`, `values`]。我们的标准形式是一个记录数组，会被检测到：

```py
[{"name":"Albert Einstein", "category":"Physics", ...},
{"name":"Marie Curie", "category":"Chemistry", ... } ... ]
```

JSON 对象的默认格式是 `columns`，形式如下：

```py
{"name":{"0":"Albert Einstein","1":"Marie Curie" ... },
"category":{"1","Physics","2":"Chemistry" ... }}
```

正如讨论的那样，对于基于 Web 的可视化工作，特别是 D3，基于记录的 JSON 数组是将行列数据传递给浏览器的最常见方式。

###### 注意

请注意，您需要有效的 JSON 文件才能使用 pandas 工作，因为 `read_json` 方法和 Python 的 JSON 解析器通常不够宽容，并且异常信息也不够详细。一个常见的 JSON 错误是未将键用双引号括起来，或者在期望双引号的地方使用单引号。后者对于那些来自单双引号可以互换的语言的人来说尤为常见，这也是为什么您永远不应该自己构建 JSON 文档的原因——总是使用官方或备受尊重的库。

有各种方法可以将 DataFrame 存储为 JSON，但最适合与任何数据可视化工作协作的格式是记录数组。这是 D3 数据的最常见形式，也是我建议从 pandas 输出的形式。将 DataFrame 作为记录写入 JSON 只需在 `to_json` 方法中指定 `orient` 字段即可：

```py
df = pd.read_json('data.json')
# ... Perform data-cleaning operations
json = df.to_json('data_cleaned.json', orient='records') ![1](assets/1.png)
Out:
[{"name":"Albert Einstein", "category":"Physics", ...},
{"name":"Marie Curie", "category":"Chemistry", ... } ... ]
```

[![1](assets/1.png)](#co_introduction_to_pandas_CO4-1)

覆盖默认保存以将 JSON 存储为适合数据可视化的记录。

此外，我们还有参数 `date_format`（*epoch* 时间戳，*iso* 表示 ISO8601 等）、`double_precision` 和 `default_handler`，用于在对象无法使用 pandas 解析器转换为 JSON 时调用。详见[pandas 文档](https://oreil.ly/wqnI0)。

## CSV

符合 pandas 数据表精神的是，它对 CSV 文件的处理足够复杂，可以处理几乎所有可想象的数据。常规的 CSV 文件，也就是大多数情况下，会在没有参数的情况下加载：

```py
# data.csv:
# name,category
# "Albert Einstein",Physics
# "Marie Curie",Chemistry

df = pd.read_csv('data.csv')
df
Out:
              name   category
0  Albert Einstein    Physics
1      Marie Curie  Chemistry
```

尽管您可能希望所有 CSV 文件都是逗号分隔的，但您经常会发现文件名以 CSV 结尾，但使用分号或竖线 (`|`) 等不同的分隔符。它们可能还会对包含空格或特殊字符的字符串使用特定的引用方式。在这种情况下，我们可以在读取请求中指定任何非标准元素。我们将使用 Python 方便的 `StringIO` 模块来模拟从文件中读取：^([5](ch08.xhtml#idm45607779004224))

```py
from io import StringIO

data = " `Albert Einstein`| Physics \n`Marie Curie`|  Chemistry"

df = pd.read_csv(StringIO(data),
   sep='|', ![1](assets/1.png)
   names=['name', 'category'], ![2](assets/2.png)
   skipinitialspace=True, quotechar="`")

df
Out:
              name   category
0  Albert Einstein   Physics
1      Marie Curie  Chemistry
```

[![1](assets/1.png)](#co_introduction_to_pandas_CO5-1)

这些字段是以管道符分隔的，而不是默认的逗号分隔。

[![2](assets/2.png)](#co_introduction_to_pandas_CO5-2)

这里我们提供了缺失的列标题。

当保存 CSV 文件时，我们同样具有相同的灵活性，这里将编码设置为 Unicode `utf-8`：

```py
df.to_csv('data.csv', encoding='utf-8')
```

想要了解 CSV 选项的详细内容，请查阅[pandas 文档](https://oreil.ly/QPCs1)。

## Excel 文件

pandas 使用 Python 的 `xlrd` 模块来读取 Excel 2003 (*.xls*) 文件，使用 `openpyxl` 模块来读取 Excel 2007+ (*.xlsx*) 文件。后者是一个可选依赖项，需要安装：

```py
$ pip install openpyxl
```

Excel文档有多个命名工作表，每个工作表都可以传递给DataFrame。有两种方法将数据表读入DataFrame中。第一种方法是创建然后解析`ExcelFile`对象：

```py
dfs = {}
xls = pd.ExcelFile('data/nobel_winners.xlsx') # load Excel file
dfs['WinnersSheet1'] = xls.parse('WinnersSheet1', na_values=['NA']) ![1](assets/1.png)
dfs['WinnersSheet2'] = xls.parse('WinnersSheet2',
    index_col=1, ![2](assets/2.png)
    na_values=['-'], ![3](assets/3.png)
    skiprows=3 ![4](assets/4.png)
    )
```

[![1](assets/1.png)](#co_introduction_to_pandas_CO6-1)

按名称获取一个工作表并保存到一个字典中。

[![2](assets/2.png)](#co_introduction_to_pandas_CO6-2)

指定要用作DataFrame行标签的列位置。

[![3](assets/3.png)](#co_introduction_to_pandas_CO6-3)

识别为NaN的其他字符串列表。

[![4](assets/4.png)](#co_introduction_to_pandas_CO6-4)

在处理之前要跳过的行数（例如，元数据）。

或者，您可以使用`read_excel`方法，它是加载多个电子表格的便捷方法：

```py
dfs = pd.read_excel('data/nobel_winners.xlsx', ['WinnersSheet1','WinnersSheet2'],
                      index_col=None, na_values=['NA'])
```

使用生成的DataFrame检查第二个Excel表的内容：

```py
In: dfs['WinnersSheet2'].head()
Out:
     category             nationality  year                name    gender
0       Peace                American  1906  Theodore Roosevelt      male
1  Literature           South African  1991     Nadine Gordimer    female
2   Chemistry  Bosnia and Herzegovina  1975     Vladamir Prelog      male
```

不使用`read_excel`的唯一原因是如果您需要不同的参数来读取每个Excel工作表。

可以使用第二个（`sheetname`）参数按索引或名称指定工作表。`sheetname`可以是单个名称字符串或索引（从0开始）或混合列表。默认情况下，`sheetname`为`0`，返回第一个工作表。示例 8-1显示了一些变化。将`sheetname`设置为`None`将返回一个以工作表名称为键的DataFrame字典。

##### 示例 8-1\. 加载Excel工作表

```py
# return the first datasheet
df = pd.read_excel('nobel_winners.xls')

# return a named sheet
df = pd.read_excel('nobel_winners.xls', 'WinnersSheet3')

# first sheet and sheet named 'WinnersSheet3'
df = pd.read_excel('nobel_winners.xls', [0, 'WinnersSheet3'])

# all sheets loaded into a name-keyed dictionary
dfs = pd.read_excel('nobel_winners.xls', sheetname=None)
```

`parse_cols`参数允许您选择要解析的工作表列。将`parse_cols`设置为整数值将选择到该序数的所有列。将`parse_cols`设置为整数列表允许您选择特定的列：

```py
# parse up to the fifth column
pd.read_excel('nobel_winners.xls', 'WinnersSheet1', parse_cols=4)

# parse the second and fourth columns
pd.read_excel('nobel_winners.xls', 'WinnersSheet1', parse_cols=[1, 3])
```

有关`read_excel`的更多信息，请参阅[pandas文档](https://oreil.ly/Js7Le)。

您可以使用`to_excel`方法将DataFrame保存到Excel文件的工作表中，给出Excel文件名和工作表名称，本例中分别为 `'nobel_winners'` 和 `'WinnersSheet1'`：

```py
df.to_excel('nobel_winners.xlsx', sheet_name='WinnersSheet1')
```

有许多与`to_csv`类似的选项，都在[pandas文档](https://oreil.ly/g15Al)中有详细介绍。因为pandas `Panel`s 和 Excel文件可以存储多个DataFrame，所以有一个`Panel` `to_excel`方法可以将其所有的DataFrame写入Excel文件中。

如果您需要选择要写入共享Excel文件的多个DataFrame，可以使用`ExcelWriter`对象：

```py
with pd.ExcelWriter('nobel_winners.xlsx') as writer:
    df1.to_excel(writer, sheet_name='WinnersSheet1')
    df2.to_excel(writer, sheet_name='WinnersSheet2')
```

## SQL

根据偏好，pandas使用Python的`SQLAlchemy`模块来进行数据库抽象化。如果使用`SQLAlchemy`，还需要数据库的驱动程序库。

使用`read_sql`方法加载数据库表或SQL查询结果是最简单的方法。让我们使用我们首选的SQLite数据库，并将其获奖者表读入DataFrame中：

```py
import sqlalchemy

engine = sqlalchemy.create_engine(
                'sqlite:///data/nobel_winners.db') ![1](assets/1.png)
df = pd.read_sql('winners', engine) ![2](assets/2.png)
df
Out:
     index                category    country date_of_birth
0     4                   Peace    Belgium    1829-07-26
...
                      name                          place_of_birth
0    Auguste Beernaert  Ostend ,  Netherlands  (now  Belgium )
...
```

[![1](assets/1.png)](#co_introduction_to_pandas_CO7-1)

在这里，我们使用一个现有的SQLite（基于文件的）数据库。SQLAlchemy可以为所有常用的数据库创建引擎，例如 *mysql://USER:PASSWORD@localhost/db*。

[![2](assets/2.png)](#co_introduction_to_pandas_CO7-2)

将`'nobel_winners'`SQL 表的内容读入 DataFrame。`read_sql`是`read_sql_table`和`read_sql_query`方法的便捷包装器，根据其第一个参数将执行正确的操作。

将 DataFrame 写入 SQL 数据库非常简单。使用刚刚创建的引擎，我们可以将获奖者表的副本添加到我们的 SQLite 数据库中：

```py
# save DataFrame df to nobel_winners SQL table
df.to_sql('winners_copy', engine, if_exists='replace')
```

如果遇到由于数据包大小限制而导致的错误，可以使用`chunksize`参数设置每次写入的行数：

```py
# write 500 rows at a time
df.to_sql('winners_copy', engine, chunksize=500)
```

pandas 将会尝试将数据映射到适当的 SQL 类型，推断对象的数据类型。如果需要，可以在加载调用中覆盖默认类型：

```py
from sqlalchemy.types import String
df.to_sql('winners_copy', engine, dtype={'year': String}) ![1](assets/1.png)
```

[![1](assets/1.png)](#co_introduction_to_pandas_CO8-1)

覆盖 pandas 的推断，并指定年份作为`String`列。

进一步的 pandas-SQL 交互细节可以在[pandas 文档](https://oreil.ly/kiLyQ)中找到。

## MongoDB

对于数据可视化工作，像 MongoDB 这样的文档型 NoSQL 数据库非常方便。在 MongoDB 的情况下，情况更好，因为它使用一种名为 BSON 的 JSON 二进制形式作为其数据存储格式，即二进制 JSON。由于 JSON 是我们选择的数据粘合剂，连接我们的 Web 数据可视化和其后端服务器，所以有足够的理由考虑将数据集存储在 Mongo 中。它还与 pandas 很好地协作。

正如我们所见，pandas 的 DataFrame 可以很好地转换到 JSON 格式，并且可以将 Mongo 文档集合轻松转换为 pandas DataFrame：

```py
import pandas as pd
from pymongo import MongoClient

client = MongoClient() ![1](assets/1.png)

db = client.nobel_prize ![2](assets/2.png)
cursor = db.winners.find() ![3](assets/3.png)
df = pd.DataFrame(list(cursor)) ![4](assets/4.png)
df ![5](assets/5.png)
# _
```

[![1](assets/1.png)](#co_introduction_to_pandas_CO9-1)

创建一个 Mongo 客户端，使用默认的主机和端口。

[![2](assets/2.png)](#co_introduction_to_pandas_CO9-2)

获取`nobel_prize`数据库。

[![3](assets/3.png)](#co_introduction_to_pandas_CO9-3)

在`winner`集合中找到所有文档。

[![4](assets/4.png)](#co_introduction_to_pandas_CO9-4)

将游标中的所有文档加载到列表中，并用于创建 DataFrame。

[![5](assets/5.png)](#co_introduction_to_pandas_CO9-5)

此时 winners 集合为空—让我们用一些 DataFrame 数据填充它。

将 DataFrame 记录插入 MongoDB 数据库同样简单。在这里，我们使用我们在[示例 3-5](ch03.xhtml#data_get_mongo)中定义的`get_mongo_database`方法获取我们的`nobel_prize`数据库，并将 DataFrame 保存到其 winners 集合中：

```py
db = get_mongo_database('nobel_prize')

records = df.to_dict('records') ![1](assets/1.png)
db[collection].insert_many(records) ![2](assets/2.png)
```

[![1](assets/1.png)](#co_introduction_to_pandas_CO10-1)

将 DataFrame 转换为`dict`，使用`records`参数将行转换为单独的对象。

[![2](assets/2.png)](#co_introduction_to_pandas_CO10-2)

对于 PyMongo 版本 2，请使用`insert`方法。

pandas 没有像`to_csv`或`read_csv`那样的 MongoDB 便利方法，但是很容易编写几个实用函数来在 MongoDB 和 DataFrame 之间进行转换：

```py
def mongo_to_dataframe(db_name, collection, query={},\
                       host='localhost', port=27017,\
                       username=None, password=None,\
                        no_id=True):
    """ create a DataFrame from mongodb collection """

    db = get_mongo_database(db_name, host, port, username,\
     password)
    cursor = db[collection].find(query)
    df =  pd.DataFrame(list(cursor))

    if no_id: ![1](assets/1.png)
        del df['_id']

    return df

def dataframe_to_mongo(df, db_name, collection,\
                       host='localhost', port=27017,\
                       username=None, password=None):
    """ save a DataFrame to mongodb collection """
    db = get_mongo_database(db_name, host, port, username,\
     password)

    records = df.to_dict('records')
    db[collection].insert_many(records)
```

[![1](assets/1.png)](#co_introduction_to_pandas_CO11-1)

Mongo 的 `_id` 字段将包含在 DataFrame 中。默认情况下，删除该列。

将 DataFrame 的记录插入 Mongo 后，让我们确保它们已经成功存储：

```py
db = get_mongo_database('nobel_prize')
list(db.winners.find()) ![1](assets/1.png)
[{'_id': ObjectId('62fcf2fb0e7fe50ac4393912'),
  'id': 1,
  'category': 'Physics',
  'name': 'Albert Einstein',
  'nationality': 'Swiss',
  'year': 1921,
  'gender': 'male'},
 {'_id': ObjectId('62fcf2fb0e7fe50ac4393913'),
  'id': 2,
  'category': 'Physics',
  'name': 'Paul Dirac',
  'nationality': 'British',
  'year': 1933,
  'gender': 'male'},
 {'_id': ObjectId('62fcf2fb0e7fe50ac4393914'),
  'id': 3,
  'category': 'Chemistry',
  'name': 'Marie Curie',
  'nationality': 'Polish',
  'year': 1911,
  'gender': 'female'}]
```

[![1](assets/1.png)](#co_introduction_to_pandas_CO12-1)

集合的 `find` 方法返回一个游标，我们将其转换为 Python 列表以查看内容。

另一种创建 DataFrame 的方法是从一系列 Series 构建它们。让我们来看看这个过程，并借此机会更详细地探讨 Series。

# Series 转为 DataFrame

Series 是 pandas 的 DataFrame 的构建模块。它们可以独立地使用与 DataFrame 镜像的方法进行操作，并且它们可以组合成 DataFrame，正如我们将在子节中看到的那样。

pandas Series 的关键思想是索引。这些索引作为标签，用于包含在一行数据中的异构数据。当 pandas 操作多个数据对象时，这些索引用于对齐字段。

Series 可以通过三种方式之一创建。第一种是从 Python 列表或 NumPy 数组创建：

```py
s = pd.Series([1, 2, 3, 4]) # Series(np.arange(4))
Out:
0    1 # index, value
1    2
2    3
3    4
dtype: int64
```

注意，我们的 Series 自动创建了整数索引。如果我们要向 DataFrame（表）添加一行数据，我们将希望通过将它们作为整数或标签的列表传递来指定列索引：

```py
s = pd.Series([1, 2, 3, 4], index=['a', 'b', 'c', 'd'])
s
Out:
a    1
b    2
c    3
d    4
dtype: int64
```

注意，索引数组的长度应与数据数组的长度匹配。

我们可以使用 Python `dict` 指定数据和索引：

```py
s = pd.Series({'a':1, 'b':2, 'c':3})
Out:
a    1
b    2
c    3
dtype: int64
```

如果我们连同 `dict` 传递一个索引数组，pandas 将做出明智的事情，将索引与数据数组进行匹配。任何不匹配的索引将设置为 `NaN`（不是数字），并且任何不匹配的数据将被丢弃。注意，元素少于索引的一个后果是 series 被转换为 `float64` 类型：

```py
s = pd.Series({'a':1, 'b':2}, index=['a', 'b', 'c'])
Out:
a    1.0
b    2.0
c    NaN
dtype: float64

s = pd.Series({'a':1, 'b':2, 'c':3}, index=['a', 'b'])
Out:
a 1
b 2
dtype: int64
```

最后，我们可以将单个标量值作为数据传递给 Series，只要我们还指定了一个索引。然后，该标量值将应用于所有索引：

```py
pd.Series(9, {'a', 'b', 'c'})
Out:
a    9
b    9
c    9
dtype: int64
```

Series 就像 NumPy 数组（`ndarray`）一样，这意味着它们可以传递给大多数 NumPy 函数：

```py
s = pd.Series([1, 2, 3, 4], ['a', 'b', 'c', 'd'])
np.sqrt(s)
Out:
a    1.000000
b    1.414214
c    1.732051
d    2.000000
dtype: float64
```

切片操作的工作方式与 Python 列表或 `ndarray` 相同，但请注意索引标签会被保留：

```py
s[1:3]
Out:
b  2
c  3
dtype: int64
```

与 NumPy 的数组不同，pandas 的 series 可以接受多种类型的数据。通过添加两个 series 来演示此实用性，其中数字被加在一起，而字符串被串联：

```py
pd.Series([1, 2.1, 'foo']) + pd.Series([2, 3, 'bar'])
Out:
0         3 # 1 + 2
1       5.1 # 2.1 + 3
2    foobar # strings correctly concatenated
dtype: object
```

在与 NumPy 生态系统交互、操作来自 DataFrame 的数据或在 pandas 的 Matplotlib 封装器之外创建可视化时，创建和操作单独的 Series 尤为重要。

由于 Series 是 DataFrame 的构建模块，因此可以使用 pandas 的 `concat` 方法将它们连接起来创建 DataFrame：

```py
names = pd.Series(['Albert Einstein', 'Marie Curie'],\
 name='name') ![1](assets/1.png)
categories = pd.Series(['Physics', 'Chemistry'],\
 name='category')

df = pd.concat([names, categories], axis=1) ![2](assets/2.png)

df.head()
Out:
               name    category
0   Albert Einstein     Physics
1       Marie Curie   Chemistry
```

[![1](assets/1.png)](#co_introduction_to_pandas_CO13-1)

我们使用 `names` 和 `categories` series 为 DataFrame 提供数据和列名（series 的 `name` 属性）。

[![2](assets/2.png)](#co_introduction_to_pandas_CO13-2)

使用 `axis` 参数为 `1` 将两个 Series 连接起来，以指示它们是列。

除了刚刚讨论的从文件和数据库创建DataFrame的多种方法之外，现在你应该已经掌握了从DataFrame中获取数据的坚实基础。

# 摘要

本章奠定了接下来两章基于pandas的基础。讨论了pandas的核心概念——DataFrame、Index和Series，我们看到了为什么pandas非常适合处理数据可视化者处理的现实世界数据类型，通过允许存储异构数据并添加强大的索引系统来扩展NumPy `ndarray`。

有了pandas的核心数据结构的基础，接下来的几章将向你展示如何使用它们来清理和处理你的诺贝尔奖获得者数据集，扩展你对pandas工具包的了解，并演示如何在数据可视化环境中应用它。

现在我们知道如何将数据输入和输出DataFrame，是时候看看pandas可以做什么了。我们将首先了解如何确保你的数据无懈可击，发现并修复诸如重复行、丢失字段和损坏数据等异常。

^([1](ch08.xhtml#idm45607780439264-marker)) 典型电子表格中的列通常具有不同的数据类型（dtypes），如浮点数、日期时间、整数等。

^([2](ch08.xhtml#idm45607779978240-marker)) 只有列名是不带空格的字符串时。

^([3](ch08.xhtml#idm45607779218800-marker)) 如果你遇到问题，可以尝试在[JSONLint的验证器](https://jsonlint.com)中验证你的数据，以获得更好的反馈。

^([4](ch08.xhtml#idm45607779160128-marker)) D3支持多种其他数据格式，如分层（树状）数据或节点和链接图格式。这里有一个[a tree hierarchy specified in JSON](https://oreil.ly/WsBCI)的示例。

^([5](ch08.xhtml#idm45607779004224-marker)) 如果你想感受一下CSV或JSON解析器的使用，我建议你采用这种方法。这比管理本地文件要方便得多。
