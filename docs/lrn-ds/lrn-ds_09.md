# 第7章。使用SQL处理关系

在[第6章](ch06.html#ch-pandas)中，我们使用数据框表示数据表。本章介绍了*关系*，另一种广泛使用的表示数据表的方式。我们还介绍了SQL，这是处理关系的标准编程语言。以下是一个关于流行狗品种信息的关系示例。

像数据框一样，关系中的每一行表示一个单狗品种记录。每一列表示记录的一个特征，例如，`grooming`列表示每个狗品种需要多频繁地梳理。

关系和数据框都为表中的每一列都有标签。但是，一个关键区别在于关系中的行没有标签，而数据框中的行有。

本章中，我们演示使用SQL进行常见的关系操作。我们首先解释SQL查询的结构。然后展示如何使用SQL执行常见的数据操作任务，如切片、过滤、排序、分组和连接。

###### 注意

本章复制了[第6章](ch06.html#ch-pandas)中的数据分析，但使用的是关系和SQL，而不是数据框和Python。两章的数据集、数据操作和结论几乎相同，以便于在使用`pandas`和SQL执行数据操作时进行比较。

# 子集化

要使用关系，我们将介绍一种称为*SQL*（Structured Query Language）的领域特定编程语言。我们通常将“SQL”发音为“sequel”，而不是拼写首字母缩略词。SQL是一种专门用于处理关系的语言，因此，与Python在操作关系数据时相比，SQL具有不同的语法。

在本章中，我们将在Python程序中使用SQL查询。这展示了一个常见的工作流程——数据科学家经常在SQL中处理和子集化数据，然后将数据加载到Python中进行进一步分析。与`pandas`程序相比，SQL数据库使处理大量数据变得更加容易。但是，将数据加载到`pandas`中使得可视化数据和构建统计模型变得更加容易。

###### 注意

为什么SQL系统往往更适合处理大型数据集？简而言之，SQL系统具有用于管理存储在磁盘上的数据的复杂算法。例如，当处理大型数据集时，SQL系统会透明地一次加载和操作小部分数据；相比之下，在`pandas`中做到这一点可能会更加困难。我们将在[第8章](ch08.html#ch-files)中更详细地讨论这个主题。

## SQL基础知识：SELECT和FROM

我们将使用`pd.read_sql`函数运行SQL查询，并将输出存储在`pandas`数据框中。使用此函数需要一些设置。我们首先导入`pandas`和`sqlalchemy` Python包：

```py
`import` `pandas` `as` `pd`
`import` `sqlalchemy`

```

我们的数据库存储在名为*babynames.db*的文件中。这个文件是一个[SQLite](https://oreil.ly/sGYWE)数据库，因此我们将设置一个可以处理这种格式的`sqlalchemy`对象：

```py
`db` `=` `sqlalchemy``.``create_engine``(``'``sqlite:///babynames.db``'``)`

```

###### 注

在本书中，我们使用 SQLite，这是一个非常有用的本地数据存储数据库系统。其他系统做出了不同的权衡，适用于不同的领域。例如，PostgreSQL 和 MySQL 是更复杂的系统，适用于大型 Web 应用程序，在这些应用程序中，许多最终用户同时写入数据。虽然每个 SQL 系统有细微的差异，但它们提供相同的核心 SQL 功能。读者可能还知道 Python 在其标准 `sqlite3` 库中提供了对 SQLite 的支持。我们选择使用 `sqlalchemy` 是因为它更容易重用代码，以适用于 SQLite 之外的其他 SQL 系统。

现在我们可以使用 `pd.read_sql` 在这个数据库上运行 SQL 查询。这个数据库有两个关系：`baby` 和 `nyt`。这是一个读取整个 `baby` 关系的简单示例。我们将 SQL 查询作为 Python 字符串编写，并传递给 `pd.read_sql`：

```py
`query` `=` `'''` 
`SELECT *`
`FROM baby;`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | Name | Sex | Count | Year |
| --- | --- | --- | --- | --- |
| **0** | Liam | M | 19659 | 2020 |
| **1** | Noah | M | 18252 | 2020 |
| **2** | Oliver | M | 14147 | 2020 |
| **...** | ... | ... | ... | ... |
| **2020719** | Verona | F | 5 | 1880 |
| **2020720** | Vertie | F | 5 | 1880 |
| **2020721** | Wilma | F | 5 | 1880 |

```py
2020722 rows × 4 columns
```

变量 `query` 内的文本包含 SQL 代码。`SELECT` 和 `FROM` 是 SQL 关键字。我们读取前述查询如下：

```py
SELECT  *  -- Get all the columns...
FROM  baby;  -- ...from the baby relation

```

`baby` 关系包含与 [第六章](ch06.html#ch-pandas) 中 `baby` 数据帧相同的数据：所有由美国社会安全管理局注册的婴儿姓名。

## 什么是关系？

让我们更详细地检查 `baby` 关系。一个关系有行和列。每一列都有一个标签，如 [Figure 7-1](#fig-relation-labels) 所示。不像数据帧，然而，关系中的个别行没有标签。也不像数据帧，关系的行不是有序的。

![relation-labels](assets/leds_0701.png)

###### 图 7-1\. `baby` 关系具有列的标签（用框框起来）

关系有着悠久的历史。对关系的更正式处理使用术语 *元组* 来指代关系的行，*属性* 来指代列。还有一种严格的方式使用关系代数来定义数据操作，它源自数学集合代数。

## 切片

*切片* 是通过从另一个关系中取出部分行或列来创建新关系的操作。想象切番茄——切片可以垂直和水平进行。要对关系的列进行切片，我们给 `SELECT` 语句传递我们想要的列：

```py
`query` `=` `'''` 
`SELECT Name`
`FROM baby;`
`'''` 

`pd``.``read_sql``(``query``,` `db``)`

```

|   | Name |
| --- | --- |
| **0** | Liam |
| **1** | Noah |
| **2** | Oliver |
| **...** | ... |
| **2020719** | Verona |
| **2020720** | Vertie |
| **2020721** | Wilma |

```py
2020722 rows × 1 columns
```

```py
`query` `=` `'''` 
`SELECT Name, Count`
`FROM baby;`
`'''` 

`pd``.``read_sql``(``query``,` `db``)`

```

|   | Name | Count |
| --- | --- | --- |
| **0** | Liam | 19659 |
| **1** | Noah | 18252 |
| **2** | Oliver | 14147 |
| **...** | ... | ... |
| **2020719** | Verona | 5 |
| **2020720** | Vertie | 5 |
| **2020721** | Wilma | 5 |

```py
2020722 rows × 2 columns
```

要切片出特定数量的行，请使用 `LIMIT` 关键字：

```py
`query` `=` `'''` 
`SELECT Name`
`FROM baby`
`LIMIT 10;`
`'''` 

`pd``.``read_sql``(``query``,` `db``)`

```

|   | 名称 |
| --- | --- |
| **0** | Liam |
| **1** | Noah |
| **2** | Oliver |
| **...** | ... |
| **7** | Lucas |
| **8** | Henry |
| **9** | Alexander |

```py
10 rows × 1 columns
```

总之，我们使用 `SELECT` 和 `LIMIT` 关键字来切片关系的列和行。

## 过滤行

现在我们转向*过滤*行—使用一个或多个条件取子集的行。在 `pandas` 中，我们使用布尔系列对象切片数据帧。在 SQL 中，我们使用带有谓词的 `WHERE` 关键字。以下查询将 `baby` 关系过滤为仅包含 2020 年的婴儿姓名：

```py
`query` `=` `'''` 
`SELECT *`
`FROM baby`
`WHERE Year = 2020;`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | 名称 | 性别 | 计数 | 年份 |
| --- | --- | --- | --- | --- |
| **0** | Liam | M | 19659 | 2020 |
| **1** | Noah | M | 18252 | 2020 |
| **2** | Oliver | M | 14147 | 2020 |
| **...** | ... | ... | ... | ... |
| **31267** | Zylynn | F | 5 | 2020 |
| **31268** | Zynique | F | 5 | 2020 |
| **31269** | Zynlee | F | 5 | 2020 |

```py
31270 rows × 4 columns
```

###### 警告

请注意，在比较相等性时，SQL 使用单等号：

```py
SELECT  *
FROM  baby
WHERE  Year  =  2020;
--         ↑
--         Single equals sign

```

然而，在 Python 中，单等号用于变量赋值。语句 `Year = 2020` 将值 `2020` 赋给变量 `Year`。要进行相等比较，Python 代码使用双等号：

```py
`# Assignment`
`my_year` `=` `2021`

`# Comparison, which evaluates to False`
`my_year` `==` `2020`

```

要向过滤器添加更多谓词，使用 `AND` 和 `OR` 关键字。例如，要查找在 2020 年或 2019 年出生的超过 10,000 名婴儿的姓名，我们写道：

```py
`query` `=` `'''` 
`SELECT *`
`FROM baby`
`WHERE Count > 10000`
 `AND (Year = 2020`
 `OR Year = 2019);`
`-- Notice that we use parentheses to enforce evaluation order`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | 名称 | 性别 | 计数 | 年份 |
| --- | --- | --- | --- | --- |
| **0** | Liam | M | 19659 | 2020 |
| **1** | Noah | M | 18252 | 2020 |
| **2** | Oliver | M | 14147 | 2020 |
| **...** | ... | ... | ... | ... |
| **41** | Mia | F | 12452 | 2019 |
| **42** | Harper | F | 10464 | 2019 |
| **43** | Evelyn | F | 10412 | 2019 |

```py
44 rows × 4 columns
```

最后，要查找 2020 年最常见的 10 个名字，我们可以使用 `ORDER BY` 关键字和 `DESC` 选项（DESC 表示 DESCending）按 `Count` 降序排序数据框：

```py
`query` `=` `'''` 
`SELECT *`
`FROM baby`
`WHERE Year = 2020`
`ORDER BY Count DESC`
`LIMIT 10;`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | 名称 | 性别 | 计数 | 年份 |
| --- | --- | --- | --- | --- |
| **0** | Liam | M | 19659 | 2020 |
| **1** | Noah | M | 18252 | 2020 |
| **2** | Emma | F | 15581 | 2020 |
| **...** | ... | ... | ... | ... |
| **7** | Sophia | F | 12976 | 2020 |
| **8** | Amelia | F | 12704 | 2020 |
| **9** | William | M | 12541 | 2020 |

```py
10 rows × 4 columns
```

我们看到，Liam、Noah 和 Emma 是 2020 年最受欢迎的婴儿名字。

## 例如：Luna 何时成为流行名字？

正如我们在[第 6 章](ch06.html#ch-pandas)中提到的，*纽约时报*文章提到，Luna 这个名字在 2000 年之前几乎不存在，但此后已成为女孩们非常流行的名字。Luna 何时变得流行？我们可以使用 SQL 中的切片和过滤来检查：

```py
`query` `=` `'''` 
`SELECT *`
`FROM baby`
`WHERE Name =` `"``Luna``"`
 `AND Sex =` `"``F``"``;`
`'''`

`luna` `=` `pd``.``read_sql``(``query``,` `db``)`
`luna`

```

|   | 名称 | 性别 | 计数 | 年份 |
| --- | --- | --- | --- | --- |
| **0** | Luna | F | 7770 | 2020 |
| **1** | Luna | F | 7772 | 2019 |
| **2** | Luna | F | 6929 | 2018 |
| **...** | ... | ... | ... | ... |
| **125** | Luna | F | 17 | 1883 |
| **126** | Luna | F | 18 | 1881 |
| **127** | Luna | F | 15 | 1880 |

```py
128 rows × 4 columns
```

`pd.read_sql` 返回一个 `pandas.DataFrame` 对象，我们可以用它来绘制图表。这展示了一个常见的工作流程 —— 使用 SQL 处理数据，将其加载到 `pandas` 数据框中，然后可视化结果：

```py
`px``.``line``(``luna``,` `x``=``'``Year``'``,` `y``=``'``Count``'``,` `width``=``350``,` `height``=``250``)`

```

![](assets/leds_07in01.png)

在本节中，我们介绍了数据科学家对关系进行子集处理的常见方法 —— 使用列标签进行切片和使用布尔条件进行过滤。在下一节中，我们将解释如何将行聚合在一起。

# 聚合

本节介绍了 SQL 中的分组和聚合。我们将使用与前一节相同的婴儿名数据：

```py
`import` `sqlalchemy`
`db` `=` `sqlalchemy``.``create_engine``(``'``sqlite:///babynames.db``'``)`

```

```py
`query` `=` `'''` 
`SELECT *`
`FROM baby`
`LIMIT 10`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | Name | Sex | Count | Year |
| --- | --- | --- | --- | --- |
| **0** | Liam | M | 19659 | 2020 |
| **1** | Noah | M | 18252 | 2020 |
| **2** | Oliver | M | 14147 | 2020 |
| **...** | ... | ... | ... | ... |
| **7** | Lucas | M | 11281 | 2020 |
| **8** | Henry | M | 10705 | 2020 |
| **9** | Alexander | M | 10151 | 2020 |

```py
10 rows × 4 columns
```

## 基本的分组聚合使用 GROUP BY

假设我们想找出记录在此数据中的总出生婴儿数。这只是 `Count` 列的总和。SQL 提供了我们在 `SELECT` 语句中使用的函数，比如 `SUM`：

```py
`query` `=` `'''` 
`SELECT SUM(Count)`
`FROM baby`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | SUM(Count) |
| --- | --- |
| **0** | 352554503 |

在 [第 6 章](ch06.html#ch-pandas) 中，我们使用分组和聚合来判断随时间是否有上升趋势的美国出生率。我们使用 `.groupby()` 按年份对数据集进行分组，然后使用 `.sum()` 在每个组内对计数进行求和。

在 SQL 中，我们使用 `GROUP BY` 子句进行分组，然后在 `SELECT` 中调用聚合函数：

```py
`query` `=` `'''` 
`SELECT Year, SUM(Count)`
`FROM baby`
`GROUP BY Year`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | Year | SUM(Count) |
| --- | --- | --- |
| **0** | 1880 | 194419 |
| **1** | 1881 | 185772 |
| **2** | 1882 | 213385 |
| **...** | ... | ... |
| **138** | 2018 | 3487193 |
| **139** | 2019 | 3437438 |
| **140** | 2020 | 3287724 |

```py
141 rows × 2 columns
```

与数据框分组一样，注意 `Year` 列包含唯一的 `Year` 值 —— 因为我们将它们分组在一起，所以不再有重复的 `Year` 值。在 `pandas` 中进行分组时，分组列成为结果数据框的索引。但是，关系没有行标签，所以 `Year` 值只是结果关系的一列。

这是在 `SQL` 中进行分组的基本步骤：

```py
SELECT
  col1,  -- column used for grouping
  SUM(col2)  -- aggregation of another column
FROM  table_name  -- relation to use
GROUP  BY  col1  -- the column(s) to group by

```

请注意，SQL 语句中子句的顺序很重要。为了避免语法错误，`SELECT` 需要首先出现，然后是 `FROM`，接着是 `WHERE`，最后是 `GROUP BY`。

在使用 `GROUP BY` 时，我们需要注意给 `SELECT` 的列。通常情况下，只有在使用这些列进行分组时，才能包括未经聚合的列。例如，在上述示例中我们按 `Year` 列进行分组，因此可以在 `SELECT` 子句中包括 `Year`。所有其他包含在 `SELECT` 中的列应该进行聚合，就像我们之前用 `SUM(Count)` 所做的那样。如果我们包含一个未使用于分组的“裸”列如 `Name`，SQLite 不会报错，但其他 SQL 引擎会报错，因此建议避免这样做。

## 多列分组

我们将多列传递给 `GROUP BY`，以便一次性按多列进行分组。当我们需要进一步细分我们的分组时，这是非常有用的。例如，我们可以按年份和性别分组，以查看随时间变化出生的男女婴儿数量：

```py
`query` `=` `'''` 
`SELECT Year, Sex, SUM(Count)`
`FROM baby`
`GROUP BY Year, Sex`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | Year | Sex | SUM(Count) |
| --- | --- | --- | --- |
| **0** | 1880 | F | 83929 |
| **1** | 1880 | M | 110490 |
| **2** | 1881 | F | 85034 |
| **...** | ... | ... | ... |
| **279** | 2019 | M | 1785527 |
| **280** | 2020 | F | 1581301 |
| **281** | 2020 | M | 1706423 |

```py
282 rows × 3 columns
```

请注意，上述代码与仅按单列进行分组非常相似，唯一的区别在于它为 `GROUP BY` 提供了多列，以便按 `Year` 和 `Sex` 进行分组。

###### 注意

与 `pandas` 不同，SQLite 没有提供简单的方法来对关系表进行透视。相反，我们可以在 SQL 中对两列使用 `GROUP BY`，将结果读取到数据框中，然后使用 `unstack()` 数据框方法。

## 其他聚合函数

SQLite 除了 `SUM` 外，还有几个内置的聚合函数，例如 `COUNT`、`AVG`、`MIN` 和 `MAX`。有关完整的函数列表，请参阅[SQLite 网站](https://oreil.ly/ALtjb)。

要使用其他聚合函数，我们在 `SELECT` 子句中调用它。例如，我们可以使用 `MAX` 替代 `SUM`：

```py
`query` `=` `'''` 
`SELECT Year, MAX(Count)`
`FROM baby`
`GROUP BY Year`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | Year | MAX(Count) |
| --- | --- | --- |
| **0** | 1880 | 9655 |
| **1** | 1881 | 8769 |
| **2** | 1882 | 9557 |
| **...** | ... | ... |
| **138** | 2018 | 19924 |
| **139** | 2019 | 20555 |
| **140** | 2020 | 19659 |

```py
141 rows × 2 columns
```

###### 注意

内置的聚合函数是数据科学家可能在 SQL 实现中首次遇到差异的地方之一。例如，SQLite 拥有相对较少的聚合函数，而[PostgreSQL 拥有更多](https://oreil.ly/gqYoK)。尽管如此，几乎所有 SQL 实现都提供 `SUM`、`COUNT`、`MIN`、`MAX` 和 `AVG`。

此部分涵盖了使用 SQL 中的 `GROUP BY` 关键字以一个或多个列对数据进行聚合的常见方法。在接下来的部分中，我们将解释如何将关系表进行连接。

# 连接

要连接两个数据表之间的记录，可以像数据框一样使用SQL关系进行连接。在本节中，我们介绍SQL连接以复制我们对婴儿姓名数据的分析。回想一下，[第6章](ch06.html#ch-pandas)提到了一篇*纽约时报*文章，讨论了某些姓名类别（如神话和婴儿潮时期的姓名）随着时间的推移变得更受欢迎或不受欢迎的情况。

我们已将*NYT*文章中的姓名和类别放入名为`nyt`的小关系中。首先，代码建立了与数据库的连接，然后运行SQL查询以显示`nyt`关系：

```py
`import` `sqlalchemy`
`db` `=` `sqlalchemy``.``create_engine``(``'``sqlite:///babynames.db``'``)`

```

```py
`query` `=` `'''` 
`SELECT *`
`FROM nyt;`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | nyt_name | 类别 |
| --- | --- | --- |
| **0** | Lucifer | forbidden |
| **1** | Lilith | forbidden |
| **2** | Danger | forbidden |
| **...** | ... | ... |
| **20** | Venus | celestial |
| **21** | Celestia | celestial |
| **22** | Skye | celestial |

```py
23 rows × 2 columns
```

###### 注

注意到前面的代码在`babynames.db`上运行查询，这个数据库包含前几节中较大的`baby`关系。SQL数据库可以容纳多个关系，当我们需要同时处理多个数据表时非常有用。另一方面，CSV文件通常只包含一个数据表——如果我们执行一个使用20个数据表的数据分析，可能需要跟踪20个CSV文件的名称、位置和版本。相反，将所有数据表存储在单个文件中的SQLite数据库中可能更简单。

要查看姓名类别的受欢迎程度，我们将`nyt`关系与`baby`关系连接，以从`baby`中获取姓名计数。

## 内连接

就像在[第6章](ch06.html#ch-pandas)中一样，我们制作了`baby`和`nyt`表的较小版本，以便更容易地查看在表合并时发生的情况。这些关系被称为`baby_small`和`nyt_small`：

```py
`query` `=` `'''` 
`SELECT *`
`FROM baby_small;`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | 姓名 | 性别 | 数量 | 年份 |
| --- | --- | --- | --- | --- |
| **0** | Noah | M | 18252 | 2020 |
| **1** | Julius | M | 960 | 2020 |
| **2** | Karen | M | 6 | 2020 |
| **3** | Karen | F | 325 | 2020 |
| **4** | Noah | F | 305 | 2020 |

```py
`query` `=` `'''` 
`SELECT *`
`FROM nyt_small;`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | nyt_name | 类别 |
| --- | --- | --- |
| **0** | Karen | boomer |
| **1** | Julius | mythology |
| **2** | Freya | mythology |

要在SQL中连接关系，我们使用`INNER JOIN`子句来指定要连接的表，并使用`ON`子句来指定表连接的条件。这里是一个示例：

```py
`query` `=` `'''` 
`SELECT *`
`FROM baby_small INNER JOIN nyt_small`
 `ON baby_small.Name = nyt_small.nyt_name`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | 姓名 | 性别 | 数量 | 年份 | nyt_name | 类别 |
| --- | --- | --- | --- | --- | --- | --- |
| **0** | Julius | M | 960 | 2020 | Julius | mythology |
| **1** | Karen | M | 6 | 2020 | Karen | boomer |
| **2** | Karen | F | 325 | 2020 | Karen | boomer |

注意到这个结果与在`pandas`中进行内连接的结果相同：新表具有`baby_small`和`nyt_small`表的列。Noah的行已消失，并且剩余的行具有它们在`nyt_small`中的匹配`category`。

要将两个表连接在一起，我们告诉 SQL 我们想要使用的每个表的列，并使用带有 `ON` 关键字的谓词进行连接。当连接列中的值满足谓词时，SQL 将行进行匹配，如 [Figure 7-2](#fig-sql-inner-join) 所示。

与 `pandas` 不同，SQL 在行连接方面提供了更大的灵活性。`pd.merge()` 方法只能使用简单的相等条件进行连接，但是 `ON` 子句中的谓词可以是任意复杂的。例如，在 [“Finding Collocated Sensors”](ch12.html#sec-pa-collocated) 中，我们利用了这种额外的多样性。

![sql-inner-join](assets/leds_0702.png)

###### 图 7-2\. 使用 SQL 将两个表连接在一起

## 左连接和右连接

类似于 `pandas`，SQL 也支持左连接。我们使用 `LEFT JOIN` 而不是 `INNER JOIN`：

```py
`query` `=` `'''` 
`SELECT *`
`FROM baby_small LEFT JOIN nyt_small`
 `ON baby_small.Name = nyt_small.nyt_name`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | Name | Sex | Count | Year | nyt_name | category |
| --- | --- | --- | --- | --- | --- | --- |
| **0** | Noah | M | 18252 | 2020 | None | None |
| **1** | Julius | M | 960 | 2020 | Julius | mythology |
| **2** | Karen | M | 6 | 2020 | Karen | 潮妈 |
| **3** | Karen | F | 325 | 2020 | Karen | 潮妈 |
| **4** | Noah | F | 305 | 2020 | None | None |

如我们所料，连接的“左”侧指的是出现在 `LEFT JOIN` 关键字左侧的表。我们可以看到即使 `Noah` 行在右侧关系中没有匹配时，它们仍然会保留在结果关系中。

请注意，SQLite 不直接支持右连接，但是我们可以通过交换关系的顺序，然后使用 `LEFT JOIN` 来执行相同的连接：

```py
`query` `=` `'''` 
`SELECT *`
`FROM nyt_small LEFT JOIN baby_small`
 `ON baby_small.Name = nyt_small.nyt_name`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | nyt_name | category | Name | Sex | Count | Year |
| --- | --- | --- | --- | --- | --- | --- |
| **0** | Karen | 潮妈 | Karen | F | 325.0 | 2020.0 |
| **1** | Karen | 潮妈 | Karen | M | 6.0 | 2020.0 |
| **2** | Julius | mythology | Julius | M | 960.0 | 2020.0 |
| **3** | Freya | mythology | None | None | NaN | NaN |

SQLite 没有内置的外连接关键字。在需要外连接的情况下，我们可以使用不同的 SQL 引擎或通过 `pandas` 执行外连接。然而，在我们（作者）的经验中，与内连接和左连接相比，外连接在实践中很少使用。

## 示例：NYT 姓名类别的流行度

现在让我们返回到完整的 `baby` 和 `nyt` 关系。

我们想知道 `nyt` 中姓名类别的流行程度随时间的变化。要回答这个问题，我们应该：

1.  使用 `ON` 关键字中指定的列内连接 `baby` 和 `nyt`，匹配姓名相等的行。

1.  按 `category` 和 `Year` 对表进行分组。

1.  使用求和对计数进行聚合：

```py
`query` `=` `'''` 
`SELECT`
 `category,`
 `Year,`
 `SUM(Count) AS count           -- [3]`
`FROM baby INNER JOIN nyt        -- [1]`
 `ON baby.Name = nyt.nyt_name   -- [1]`
`GROUP BY category, Year         -- [2]`
`'''`

`cate_counts` `=` `pd``.``read_sql``(``query``,` `db``)`
`cate_counts`

```

|   | category | Year | count |
| --- | --- | --- | --- |
| **0** | 潮妈 | 1880 | 292 |
| **1** | 潮妈 | 1881 | 298 |
| **2** | 潮妈 | 1882 | 326 |
| **...** | ... | ... | ... |
| **647** | mythology | 2018 | 2944 |
| **648** | mythology | 2019 | 3320 |
| **649** | mythology | 2020 | 3489 |

```py
650 rows × 3 columns
```

在上述查询中方括号中的数字（`[1]`、`[2]`、`[3]`）显示了我们计划中的每个步骤如何映射到 SQL 查询的部分。代码重新创建了来自 [第 6 章](ch06.html#ch-pandas) 的数据框，我们在其中创建了图表以验证 *纽约时报* 文章的主张。为简洁起见，我们在此省略了重复绘制图表的部分。

###### 注意

请注意，在此示例中的 SQL 代码中，数字的顺序看起来是不正确的——`[3]`、`[1]`，然后是`[2]`。对于首次学习 SQL 的人来说，我们通常可以将 `SELECT` 语句看作查询的*最后*执行的部分，即使它首先出现。

在本节中，我们介绍了用于关系的连接。当将关系连接在一起时，我们使用 `INNER JOIN` 或 `LEFT JOIN` 关键字以及布尔谓词来匹配行。在下一节中，我们将解释如何转换关系中的值。

# 转换和公共表达式（CTE）

在本节中，我们展示了如何调用内置 SQL 函数来转换数据列。我们还演示了如何使用公共表达式（CTE）从简单查询构建复杂查询。与往常一样，我们首先加载数据库：

```py
`# Set up connection to database`
`import` `sqlalchemy`
`db` `=` `sqlalchemy``.``create_engine``(``'``sqlite:///babynames.db``'``)`

```

## SQL 函数

SQLite 提供了多种*标量函数*，即用于转换单个数据值的函数。当在数据列上调用时，SQLite 将对列中的每个值应用这些函数。相比之下，像 `SUM` 和 `COUNT` 这样的聚合函数以数据列作为输入，并计算单个值作为输出。

SQLite 在其[在线文档](https://oreil.ly/kznBO)中提供了内置标量函数的详尽列表。例如，要找出每个名称中的字符数，我们使用 `LENGTH` 函数：

```py
`query` `=` `'''` 
`SELECT Name, LENGTH(Name)`
`FROM baby`
`LIMIT 10;`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | Name | LENGTH(Name) |
| --- | --- | --- |
| **0** | Liam | 4 |
| **1** | Noah | 4 |
| **2** | Oliver | 6 |
| **...** | ... | ... |
| **7** | Lucas | 5 |
| **8** | Henry | 5 |
| **9** | Alexander | 9 |

```py
10 rows × 2 columns
```

请注意，`LENGTH` 函数应用于 `Name` 列中的每个值。

###### 注意

像聚合函数一样，每个 SQL 实现都提供了不同的标量函数集。SQLite 提供的函数集相对较少，而 [PostgreSQL 则更多](https://oreil.ly/i2KIA)。尽管如此，几乎所有 SQL 实现都提供了与 SQLite 的 `LENGTH`、`ROUND`、`SUBSTR` 和 `LIKE` 函数相当的功能。

虽然标量函数与聚合函数使用相同的语法，但它们的行为不同。如果在单个查询中混合使用这两种函数，可能会导致输出结果混乱：

```py
`query` `=` `'''` 
`SELECT Name, LENGTH(Name), AVG(Count)`
`FROM baby`
`LIMIT 10;`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | Name | LENGTH(Name) | AVG(Count) |
| --- | --- | --- | --- |
| **0** | Liam | 4 | 174.47 |

在这里，`AVG(Name)` 计算了整个 `Count` 列的平均值，但输出结果令人困惑——读者很容易会认为平均值与名字 Liam 有关。因此，当标量函数和聚合函数同时出现在 `SELECT` 语句中时，我们必须格外小心。

要提取每个名称的首字母，我们可以使用 `SUBSTR` 函数（缩写为*substring*）。如文档中所述，`SUBSTR` 函数接受三个参数。第一个是输入字符串，第二个是开始子字符串的位置（从 1 开始索引），第三个是子字符串的长度：

```py
`query` `=` `'''` 
`SELECT Name, SUBSTR(Name, 1, 1)`
`FROM baby`
`LIMIT 10;`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | 名称 | SUBSTR(名称, 1, 1) |
| --- | --- | --- |
| **0** | Liam | L |
| **1** | Noah | N |
| **2** | Oliver | O |
| **...** | ... | ... |
| **7** | Lucas | L |
| **8** | Henry | H |
| **9** | Alexander | A |

```py
10 rows × 2 columns
```

我们可以使用 `AS` 关键字重命名列：

```py
`query` `=` `'''` 
`SELECT *, SUBSTR(Name, 1, 1) AS Firsts`
`FROM baby`
`LIMIT 10;`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | 名称 | 性别 | 数量 | 年份 | 首字母 |
| --- | --- | --- | --- | --- | --- |
| **0** | Liam | M | 19659 | 2020 | L |
| **1** | Noah | M | 18252 | 2020 | N |
| **2** | Oliver | M | 14147 | 2020 | O |
| **...** | ... | ... | ... | ... | ... |
| **7** | Lucas | M | 11281 | 2020 | L |
| **8** | Henry | M | 10705 | 2020 | H |
| **9** | Alexander | M | 10151 | 2020 | A |

```py
10 rows × 5 columns
```

在计算每个名称的首字母后，我们的分析旨在了解不同时间段内首字母的流行度。为此，我们希望将这个 SQL 查询的输出用作更长操作链中的单个步骤。

SQL 提供了几种选项来将查询分解为较小的步骤，这在像这样更复杂的分析中非常有帮助。最常见的选项是使用 `CREATE TABLE` 语句创建新关系，使用 `CREATE VIEW` 创建新视图，或者使用 `WITH` 创建临时关系。每种方法都有不同的用例。为简单起见，我们在本节仅描述 `WITH` 语句，并建议读者查阅 SQLite 文档以获取详细信息。

## 使用 `WITH` 子句进行多步查询

`WITH` 子句允许我们为任何 `SELECT` 查询指定一个名称。然后我们可以把该查询视为数据库中的一个关系，仅在查询的持续时间内存在。SQLite 将这些临时关系称为*公共表达式*。例如，我们可以取之前计算每个名称的首字母的查询，并称其为 `letters`：

```py
`query` `=` `'''` 
`-- Create a temporary relation called letters by calculating`
`-- the first letter for each name in baby`
`WITH letters AS (`
 `SELECT *, SUBSTR(Name, 1, 1) AS Firsts`
 `FROM baby`
`)`
`-- Then, select the first ten rows from letters`
`SELECT *`
`FROM letters`
`LIMIT 10;`
`'''`

`pd``.``read_sql``(``query``,` `db``)`

```

|   | 名称 | 性别 | 数量 | 年份 | 首字母 |
| --- | --- | --- | --- | --- | --- |
| **0** | Liam | M | 19659 | 2020 | L |
| **1** | Noah | M | 18252 | 2020 | N |
| **2** | Oliver | M | 14147 | 2020 | O |
| **...** | ... | ... | ... | ... | ... |
| **7** | Lucas | M | 11281 | 2020 | L |
| **8** | Henry | M | 10705 | 2020 | H |
| **9** | Alexander | M | 10151 | 2020 | A |

```py
10 rows × 5 columns
```

`WITH` 语句非常有用，因为它们可以链接在一起。我们可以在 `WITH` 语句中创建多个临时关系，每个关系对前一个结果执行一些工作，这样可以逐步构建复杂的查询。

## 例如：“L” 名字的流行度

我们可以使用`WITH`语句来查看以字母L开头的名字随时间的流行度。我们将临时`letters`关系按首字母和年份分组，然后使用求和聚合`Count`列，然后筛选只获取字母L开头的名字：

```py
`query` `=` `'''` 
`WITH letters AS (`
 `SELECT *, SUBSTR(Name, 1, 1) AS Firsts`
 `FROM baby`
`)`
`SELECT Firsts, Year, SUM(Count) AS Count`
`FROM letters`
`WHERE Firsts =` `"``L``"`
`GROUP BY Firsts, Year;`
`'''`

`letter_counts` `=` `pd``.``read_sql``(``query``,` `db``)`
`letter_counts`

```

|   | 首字母 | 年份 | 数量 |
| --- | --- | --- | --- |
| **0** | L | 1880 | 12799 |
| **1** | L | 1881 | 12770 |
| **2** | L | 1882 | 14923 |
| **...** | ... | ... | ... |
| **138** | L | 2018 | 246251 |
| **139** | L | 2019 | 249315 |
| **140** | L | 2020 | 239760 |

```py
141 rows × 3 columns
```

这个关系包含与[第6章](ch06.html#ch-pandas)相同的数据。在那一章中，我们制作了`Count`列随时间变化的图表，这里为了简洁起见省略了。

在这一节中，我们介绍了数据转换。为了转换关系中的值，我们通常使用SQL函数如`LENGTH()`或`SUBSTR()`。我们还解释了如何使用`WITH`子句构建复杂查询。

# 概要

在本章中，我们解释了什么是关系，它们为何有用，以及如何使用SQL代码处理它们。SQL数据库在许多现实世界的场景中非常有用。例如，SQL数据库通常具有强大的数据恢复机制——如果计算机在SQL操作中崩溃，数据库系统可以尽可能恢复数据而不会损坏。如前所述，SQL数据库还能处理更大规模的数据；组织使用SQL数据库来存储和查询那些用`pandas`代码无法在内存中分析的大型数据库。这些只是SQL成为数据科学工具箱中重要一环的几个原因，我们预计许多读者很快会在工作中遇到SQL代码。
