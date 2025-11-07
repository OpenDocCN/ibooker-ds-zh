# 9 GroupBy 对象

本章涵盖

+   使用`groupby`方法按组拆分`DataFrame`

+   从`GroupBy`对象中的组中提取第一行和最后一行

+   在`GroupBy`组上执行聚合操作

+   遍历`GroupBy`对象中的`DataFrame`

pandas 库的`GroupBy`对象是一个用于将`DataFrame`行分组到桶中的存储容器。它提供了一套方法来聚合和分析集合中的每个独立组。它允许我们在每个组中提取特定索引位置的行。它还提供了一个方便的方式来遍历行的组。`GroupBy`对象中包含了很多强大的功能，所以让我们看看它能做什么。

## 9.1 从头创建一个 GroupBy 对象

让我们创建一个新的 Jupyter Notebook 并导入 pandas 库：

```
In  [1] import pandas as pd
```

我们将从一个小的例子开始，并在第 9.2 节中深入更多技术细节。让我们首先创建一个`DataFrame`，它存储了超市中水果和蔬菜的价格：

```
In  [2] food_data = {
          "Item": ["Banana", "Cucumber", "Orange", "Tomato", "Watermelon"],
          "Type": ["Fruit", "Vegetable", "Fruit", "Vegetable", "Fruit"],
          "Price": [0.99, 1.25, 0.25, 0.33, 3.00]
        }

        supermarket = pd.DataFrame(data = food_data)

        supermarket

Out [2]

 **Item       Type  Price**
0      Banana      Fruit   0.99
1    Cucumber  Vegetable   1.25
2      Orange      Fruit   0.25
3      Tomato  Vegetable   0.33
4  Watermelon      Fruit   3.00
```

类型列标识一个项目所属的组。在超市数据集中有两种项目组：水果和蔬菜。我们可以使用诸如*组*、*桶*和*簇*等术语来互换地描述相同的概念。多行可能属于同一类别。

`GroupBy`对象根据列中的共享值将`DataFrame`行组织到桶中。假设我们感兴趣的是水果的平均价格和蔬菜的平均价格。如果我们能将`"Fruit"`行和`"Vegetable"`行隔离到不同的组中，那么进行计算会更容易。

让我们从在超市`DataFrame`上调用`groupby`方法开始。我们需要传递给它 pandas 将用于创建组的列。下一个示例提供了类型列。该方法返回一个我们尚未见过的对象：一个`DataFrameGroupBy`。`DataFrameGroupBy`对象与`DataFrame`是分开且独立的：

```
In  [3] groups = supermarket.groupby("Type")
        groups

Out [3] <pandas.core.groupby.generic.DataFrameGroupBy object at
        0x114f2db90>
```

类型列有两个唯一值，因此`GroupBy`对象将存储两个组。`get_group`方法接受一个组名并返回一个包含相应行的`DataFrame`。让我们提取出`"Fruit"`行：

```
In  [4] groups.get_group("Fruit")

Out [4]

 **Item   Type  Price**
0      Banana  Fruit   0.99
2      Orange  Fruit   0.25
4  Watermelon  Fruit   3.00
```

我们还可以提取出`"Vegetable"`行：

```
In  [5] groups.get_group("Vegetable")

Out [5]

 **Item       Type  Price**
1  Cucumber  Vegetable   1.25
3    Tomato  Vegetable   0.33
```

`GroupBy`对象在聚合操作方面表现出色。我们的原始目标是计算超市中水果和蔬菜的平均价格。我们可以在`groups`上调用`mean`方法来计算每个组内项目的平均价格。通过几行代码，我们已经成功拆分、聚合和分析了一个数据集：

```
In  [6] groups.mean()

Out [6]

              Price
**Type** 
Fruit      1.413333
Vegetable  0.790000
```

在掌握了基础知识之后，让我们继续到一个更复杂的数据集。

## 9.2 从数据集中创建一个 GroupBy 对象

《财富》1000 强是美国按收入排名的前 1,000 家大公司的名单。《财富》杂志每年更新一次这个名单。fortune1000.csv 文件是 2018 年的《财富》1000 强公司的集合。每一行包括公司的名称、收入、利润、员工人数、行业和子行业：

```
In  [7] fortune = pd.read_csv("fortune1000.csv")
        fortune

Out [7]

 **Company  Revenues  Profits  Employees        Sector      Industry**
0         Walmart  500343.0   9862.0    2300000     Retailing  General M...
1     Exxon Mobil  244363.0  19710.0      71200        Energy  Petroleum...
2    Berkshire...  242137.0  44940.0     377000    Financials  Insurance...
3           Apple  229234.0  48351.0     123000    Technology  Computers...
4    UnitedHea...  201159.0  10558.0     260000   Health Care  Health Ca...
...       ...         ...       ...       ...           ...             ...
995  SiteOne L...    1862.0     54.6       3664   Wholesalers  Wholesale...
996  Charles R...    1858.0    123.4      11800   Health Care  Health Ca...
997     CoreLogic    1851.0    152.2       5900  Business ...  Financial...
998  Ensign Group    1849.0     40.5      21301   Health Care  Health Ca...
999           HCP    1848.0    414.2        190    Financials   Real estate

1000 rows × 6 columns
```

一个行业可以拥有许多公司。例如，苹果和亚马逊都属于“技术”行业。

行业是某个行业内的子类别。例如，“管道”和“石油精炼”行业属于“能源”行业。

“行业”列包含 21 个独特的行业。假设我们想要找到每个行业内的公司的平均收入。在我们使用`GroupBy`对象之前，让我们通过一个替代方法来解决这个问题。第五章向我们展示了如何创建一个布尔`Series`来从`DataFrame`中提取子集。下一个示例提取了所有“零售”行业的公司：

```
In  [8] in_retailing = fortune["Sector"] == "Retailing"
        retail_companies = fortune[in_retailing]
        retail_companies.head()

Out [8]

 **Company  Revenues  Profits  Employees     Sector           Industry**
0      Walmart  500343.0   9862.0    2300000  Retailing  General Mercha...
7   Amazon.com  177866.0   3033.0     566000  Retailing  Internet Servi...
14      Costco  129025.0   2679.0     182000  Retailing  General Mercha...
22  Home Depot  100904.0   8630.0     413000  Retailing  Specialty Reta...
38      Target   71879.0   2934.0     345000  Retailing  General Mercha...
```

我们可以通过使用方括号从子集中提取出收入列：

```
In  [9] retail_companies["Revenues"].head()

Out [9] 0     500343.0
        7     177866.0
        14    129025.0
        22    100904.0
        38     71879.0
        Name: Revenues, dtype: float64
```

最后，我们可以通过在收入列上调用`mean`方法来计算“零售”行业的平均收入：

```
In  [10] retail_companies["Revenues"].mean()

Out [10] 21874.714285714286
```

上述代码适用于计算一个行业的平均收入。然而，我们需要编写大量的额外代码，才能将相同的逻辑应用于《财富》中的其他 20 个行业。代码的可扩展性不高。Python 可以自动化一些重复性工作，但`GroupBy`对象提供了现成的最佳解决方案。pandas 开发者已经为我们解决了这个问题。

让我们在`fortune`的`DataFrame`上调用`groupby`方法。该方法接受一个列，pandas 将使用该列的值来分组行。如果列存储了行的分类数据，则列是分组的良好候选。确保有多个行属于其下的父类别。例如，数据集有 1,000 个独特的公司，但只有 21 个独特的行业，因此行业列非常适合进行汇总分析：

```
In  [11] sectors = fortune.groupby("Sector")
```

让我们输出`sectors`变量，看看我们正在处理什么类型的对象：

```
In  [12] sectors

Out [12] <pandas.core.groupby.generic.DataFrameGroupBy object at
         0x1235b1d10>
```

`DataFrameGroupBy`对象是一组`DataFrame`。在幕后，pandas 重复了我们用于“零售”行业的提取过程，但针对“行业”列中的所有 21 个值。

我们可以通过将`GroupBy`对象传递给 Python 的内置`len`函数来计算`sectors`中的组数：

```
In  [13] len(sectors)

Out [13] 21
```

`sectors` `GroupBy`对象有 21 个`DataFrame`。这个数字等于`fortune`的“行业”列中唯一值的数量，我们可以通过调用`nunique`方法来发现这一点：

```
In  [14] fortune["Sector"].nunique()

Out [14] 21
```

有哪些 21 个行业，以及《财富》杂志中每个行业有多少家公司？在`GroupBy`对象上的`size`方法返回一个包含按字母顺序排列的组和它们行数的`Series`。以下输出告诉我们，有 25 家《财富》公司属于“航空航天与国防”行业，14 家属于“服装”行业，等等：

```
In  [15] sectors.size()

Out [15] Sector
         Aerospace & Defense               25
         Apparel                           14
         Business Services                 53
         Chemicals                         33
         Energy                           107
         Engineering & Construction        27
         Financials                       155
         Food &  Drug Stores               12
         Food, Beverages & Tobacco         37
         Health Care                       71
         Hotels, Restaurants & Leisure     26
         Household Products                28
         Industrials                       49
         Materials                         45
         Media                             25
         Motor Vehicles & Parts            19
         Retailing                         77
         Technology                       103
         Telecommunications                10
         Transportation                    40
         Wholesalers                       44
         dtype: int64
```

现在我们已经将财富行分桶，让我们探索我们可以用`GroupBy`对象做什么。

## 9.3 GroupBy 对象的属性和方法

将我们的`GroupBy`对象可视化为字典，将 21 个行业映射到每个行业所属的财富行集合。`groups`属性存储一个字典，其中包含这些组到行的关联；其键是行业名称，其值是存储来自财富`DataFrame`的行索引位置的`Index`对象。该字典共有 21 个键值对，但我已将以下输出限制在前两个对以节省空间：

```
In  [16] sectors.groups

Out [16]

'Aerospace &  Defense': Int64Index([ 26,  50,  58,  98, 117, 118, 207, 224,
                                     275, 380, 404, 406, 414, 540, 660,
                                     661, 806, 829, 884, 930, 954, 955,
                                     959, 975, 988], dtype='int64'),
 'Apparel': Int64Index([88, 241, 331, 420, 432, 526, 529, 554, 587, 678,
                        766, 774, 835, 861], dtype='int64'),
```

输出告诉我们，索引位置为 26、50、58、98 等行的财富中的“Sector”列的值为`"Aerospace & Defense"`。

第四章介绍了用于通过索引标签提取`DataFrame`行和列的`loc`访问器。其第一个参数是行索引标签，第二个参数是列索引标签。让我们提取一个样本财富行以确认 pandas 是否将其拉入正确的行业组。我们将尝试`26`，这是`"Aerospace & Defense"`组中列出的第一个索引位置：

```
In  [17] fortune.loc[26, "Sector"]

Out [17] 'Aerospace &  Defense'
```

如果我们想找到每个行业中表现最好的公司（按收入计算）怎么办？`GroupBy`对象的`first`方法提取了财富中每个行业的第一个列表。因为我们的财富`DataFrame`按收入排序，所以每个行业提取出的第一家公司将是该行业表现最好的公司。`first`的返回值是一个 21 行的`DataFrame`（每个行业一家公司）：

```
In  [18] sectors.first()

Out [18]

                      Company  Revenues  Profits  Employees       Industry
**Sector** 
Aerospace &...         Boeing   93392.0   8197.0     140800  Aerospace ...
Apparel                  Nike   34350.0   4240.0      74400        Apparel
Business Se...  ManpowerGroup   21034.0    545.4      29000  Temporary ...
Chemicals           DowDuPont   62683.0   1460.0      98000      Chemicals
Energy            Exxon Mobil  244363.0  19710.0      71200  Petroleum ...
 ...                ...          ...        ...        ...             ...
Retailing             Walmart  500343.0   9862.0    2300000  General Me...
Technology              Apple  229234.0  48351.0     123000  Computers,...
Telecommuni...           AT&T  160546.0  29450.0     254000  Telecommun...
Transportation            UPS   65872.0   4910.0     346415  Mail, Pack...
Wholesalers          McKesson  198533.0   5070.0      64500  Wholesaler...
```

相补的`last`方法从属于每个行业的财富中提取最后一家公司。同样，pandas 按照它们在`DataFrame`中出现的顺序提取行。因为财富按收入降序排列公司，所以以下结果揭示了每个行业中收入最低的公司：

```
In  [19] sectors.last()

Out [19]

                      Company  Revenues  Profits  Employees       Industry
**Sector** 
Aerospace &...  Aerojet Ro...    1877.0     -9.2       5157  Aerospace ...
Apparel         Wolverine ...    2350.0      0.3       3700        Apparel
Business Se...      CoreLogic    1851.0    152.2       5900  Financial ...
Chemicals              Stepan    1925.0     91.6       2096      Chemicals
Energy          Superior E...    1874.0   -205.9       6400  Oil and Ga...
 ...                      ...      ...      ...         ...            ...
Retailing       Childrens ...    1870.0     84.7       9800  Specialty ...
Technology      VeriFone S...    1871.0   -173.8       5600  Financial ...
Telecommuni...  Zayo Group...    2200.0     85.7       3794  Telecommun...
Transportation  Echo Globa...    1943.0     12.6       2453  Transporta...
Wholesalers     SiteOne La...    1862.0     54.6       3664  Wholesaler...
```

`GroupBy`对象为每个行业组中的行分配索引位置。`"Aerospace & Defense"`行业的第一个财富行在其组中的索引位置为 0。同样，`"Apparel"`行业的第一个财富行在其组中的索引位置为 0。索引位置在组之间是独立的。

第`n`种方法从其组中提取给定索引位置的行。如果我们用`0`作为参数调用`nth`方法，我们将得到每个行业中的第一家公司。下一个`DataFrame`与`first`方法返回的相同：

```
In  [20] sectors.nth(0)

Out [20]

                      Company  Revenues  Profits  Employees       Industry
**Sector** 
Aerospace &...         Boeing   93392.0   8197.0     140800  Aerospace ...
Apparel                  Nike   34350.0   4240.0      74400        Apparel
Business Se...  ManpowerGroup   21034.0    545.4      29000  Temporary ...
Chemicals           DowDuPont   62683.0   1460.0      98000      Chemicals
Energy            Exxon Mobil  244363.0  19710.0      71200  Petroleum ...
 ...              ...            ...       ...          ...            ...
Retailing             Walmart  500343.0   9862.0    2300000  General Me...
Technology              Apple  229234.0  48351.0     123000  Computers,...
Telecommuni...           AT&T  160546.0  29450.0     254000  Telecommun...
Transportation            UPS   65872.0   4910.0     346415  Mail, Pack...
Wholesalers          McKesson  198533.0   5070.0      64500  Wholesaler...
```

下一个示例将`3`作为参数传递给`nth`方法，以从`fortune` `DataFrame`中的每个行业提取第四行。结果包括在其行业中按收入排名第四的 21 家公司：

```
In  [21] sectors.nth(3)

Out [21]

                      Company  Revenues  Profits  Employees       Industry
**Sector** 
Aerospace &...  General Dy...   30973.0   2912.0      98600  Aerospace ...
Apparel          Ralph Lauren    6653.0    -99.3      18250        Apparel
Business Se...        Aramark   14604.0    373.9     215000  Diversifie...
Chemicals            Monsanto   14640.0   2260.0      21900      Chemicals
Energy          Valero Energy   88407.0   4065.0      10015  Petroleum ...
 ...               ...            ...       ...        ...             ...
Retailing          Home Depot  100904.0   8630.0     413000  Specialty ...
Technology                IBM   79139.0   5753.0     397800  Informatio...
Telecommuni...  Charter Co...   41581.0   9895.0      94800  Telecommun...
Transportation  Delta Air ...   41244.0   3577.0      86564       Airlines
Wholesalers             Sysco   55371.0   1142.5      66500  Wholesaler...
```

注意到`"Apparel"`行业的值是`"Ralph Lauren"`。我们可以通过在财富中过滤`"Apparel"`行来确认输出是否正确。注意`"Ralph Lauren"`是第四行：

```
In  [22] fortune[fortune["Sector"] == "Apparel"].head()

Out [22]

 **Company  Revenues  Profits  Employees   Sector Industry**
88           Nike   34350.0   4240.0      74400  Apparel  Apparel
241            VF   12400.0    614.9      69000  Apparel  Apparel
331           PVH    8915.0    537.8      28050  Apparel  Apparel
420  Ralph Lauren    6653.0    -99.3      18250  Apparel  Apparel
432   Hanesbrands    6478.0     61.9      67200  Apparel  Apparel
```

`head`方法从每个组中提取多行。在下一个示例中，`head(2)`提取每个部门在财富中的前两行。结果是包含 42 行的`DataFrame`（21 个独特部门，每个部门两行）。不要将`GroupBy`对象上的此`head`方法与`DataFrame`对象上的`head`方法混淆：

```
In  [23] sectors.head(2)

Out [23]

 **Company  Revenues  Profits  Employees        Sector      Industry**
0         Walmart  500343.0   9862.0    2300000     Retailing  General M...
1     Exxon Mobil  244363.0  19710.0      71200        Energy  Petroleum...
2    Berkshire...  242137.0  44940.0     377000    Financials  Insurance...
3           Apple  229234.0  48351.0     123000    Technology  Computers...
4    UnitedHea...  201159.0  10558.0     260000   Health Care  Health Ca...
  ...         ...     ...       ...        ...          ...             ...
160          Visa   18358.0   6699.0      15000  Business ...  Financial...
162  Kimberly-...   18259.0   2278.0      42000  Household...  Household...
163         AECOM   18203.0    339.4      87000  Engineeri...  Engineeri...
189  Sherwin-W...   14984.0   1772.3      52695     Chemicals     Chemicals
241            VF   12400.0    614.9      69000       Apparel       Apparel
```

相补的`tail`方法从每个组中提取最后一行。例如，`tail(3)`提取每个部门的最后三行。结果是 63 行的`DataFrame`（21 个部门 x 3 行）：

```
In  [24] sectors.tail(3)

Out [24]

 **Company  Revenues  Profits  Employees        Sector      Industry**
473  Windstrea...    5853.0  -2116.6      12979  Telecommu...  Telecommu...
520  Telephone...    5044.0    153.0       9900  Telecommu...  Telecommu...
667  Weis Markets    3467.0     98.4      23000  Food &  D...  Food and ...
759  Hain Cele...    2853.0     67.4       7825  Food, Bev...  Food Cons...
774  Fossil Group    2788.0   -478.2      12300       Apparel       Apparel
  ...         ...      ...      ...        ...          ...             ...
995  SiteOne L...    1862.0     54.6       3664   Wholesalers  Wholesale...
996  Charles R...    1858.0    123.4      11800   Health Care  Health Ca...
997     CoreLogic    1851.0    152.2       5900  Business ...  Financial...
998  Ensign Group    1849.0     40.5      21301   Health Care  Health Ca...
999           HCP    1848.0    414.2        190    Financials   Real estate

63 rows × 6 columns
```

我们可以使用`get_group`方法提取给定组中的所有行。该方法返回包含行的`DataFrame`。下一个示例显示`"Energy"`部门中的所有公司：

```
In  [25] sectors.get_group("Energy").head()

Out [25]

 **Company  Revenues  Profits  Employees  Sector        Industry**
1      Exxon Mobil  244363.0  19710.0      71200  Energy  Petroleum R...
12         Chevron  134533.0   9195.0      51900  Energy  Petroleum R...
27     Phillips 66   91568.0   5106.0      14600  Energy  Petroleum R...
30   Valero Energy   88407.0   4065.0      10015  Energy  Petroleum R...
40  Marathon Pe...   67610.0   3432.0      43800  Energy  Petroleum R...
```

现在我们已经了解了`GroupBy`对象的机制，让我们讨论如何对每个嵌套组中的值进行聚合。

## 9.4 聚合操作

我们可以通过在`GroupBy`对象上调用方法来对每个嵌套组应用聚合操作。例如，`sum`方法将每个组中的列值相加。默认情况下，pandas 针对原始`DataFrame`中的所有数值列。在下一个示例中，`sum`方法计算了财富`DataFrame`中三个数值列（收入、利润和员工）的每个部门的总和。我们在`GroupBy`对象上调用`sum`方法：

```
In  [26] sectors.sum().head(10)

Out [26]

                             Revenues   Profits  Employees
**Sector** 
Aerospace & Defense          383835.0   26733.5    1010124
Apparel                      101157.3    6350.7     355699
Business Services            316090.0   37179.2    1593999
Chemicals                    251151.0   20475.0     474020
Energy                      1543507.2   85369.6     981207
Engineering & Construction   172782.0    7121.0     420745
Financials                  2442480.0  264253.5    3500119
Food &  Drug Stores          405468.0    8440.3    1398074
Food, Beverages & Tobacco    510232.0   54902.5    1079316
Health Care                 1507991.4   92791.1    2971189
```

让我们检查一个样本计算。Pandas 将公司在`"Aerospace & Defense"`中的收入总和列示为$383,835。我们可以使用`get_group`方法检索嵌套的`"Aerospace & Defense"` `DataFrame`，针对其收入列，并使用`sum`方法计算其总和：

```
In  [27] sectors.get_group("Aerospace & Defense").head()

Out [27]

 **Company  Revenues  Profits  Employees        Sector      Industry**
26         Boeing   93392.0   8197.0     140800  Aerospace...  Aerospace...
50   United Te...   59837.0   4552.0     204700  Aerospace...  Aerospace...
58   Lockheed ...   51048.0   2002.0     100000  Aerospace...  Aerospace...
98   General D...   30973.0   2912.0      98600  Aerospace...  Aerospace...
117  Northrop ...   25803.0   2015.0      70000  Aerospace...  Aerospace...

In  [28] sectors.get_group("Aerospace & Defense").loc[:,"Revenues"].head()

Out [28] 26     93392.0
         50     59837.0
         58     51048.0
         98     30973.0
         117    25803.0
         Name: Revenues, dtype: float64

In  [29] sectors.get_group("Aerospace & Defense").loc[:, "Revenues"].sum()

Out [29] 383835.0
```

值是相等的。Pandas 是正确的！通过单个`sum`方法调用，库将计算逻辑应用于`sectors` `GroupBy`对象中的每个嵌套`DataFrame`。我们用最少的代码对列的所有分组进行了聚合分析。

`GroupBy`对象支持许多其他聚合方法。下一个示例调用`mean`方法计算每个部门的收入、利润和员工列的平均值。同样，pandas 只包括其计算中的数值列：

```
In  [30] sectors.mean().head()

Out [30]

                         Revenues      Profits     Employees
**Sector** 
Aerospace & Defense  15353.400000  1069.340000  40404.960000
Apparel               7225.521429   453.621429  25407.071429
Business Services     5963.962264   701.494340  30075.452830
Chemicals             7610.636364   620.454545  14364.242424
Energy               14425.300935   805.373585   9170.158879
```

我们可以通过在`GroupBy`对象后面传递其名称并在方括号内传递来针对单个财富列。Pandas 返回一个新的对象，一个`SeriesGroupBy`：

```
In  [31] sectors["Revenues"]

Out [31] <pandas.core.groupby.generic.SeriesGroupBy object at 0x114778210>
```

在底层，`DataFrameGroupBy`对象存储了一个`SeriesGroupBy`对象的集合。`SeriesGroupBy`对象可以对财富中的单个列执行聚合操作。Pandas 将结果按部门组织。下一个示例按部门计算收入总和：

```
In  [32] sectors["Revenues"].sum().head()

Out [32] Sector
         Aerospace & Defense     383835.0
         Apparel                 101157.3
         Business Services       316090.0
         Chemicals               251151.0
         Energy                 1543507.2
         Name: Revenues, dtype: float64
```

下一个示例计算每个部门的平均员工数量：

```
In  [33] sectors["Employees"].mean().head()

Out [33] Sector
         Aerospace & Defense    40404.960000
         Apparel                25407.071429
         Business Services      30075.452830
         Chemicals              14364.242424
         Energy                  9170.158879
         Name: Employees, dtype: float64
```

`max`方法从给定列返回最大值。在下一个示例中，我们提取每个部门的最高利润列值。在`"Aerospace & Defense"`部门表现最好的公司利润为$8,197：

```
In  [34] sectors["Profits"].max().head()

Out [34] Sector
         Aerospace & Defense     8197.0
         Apparel                 4240.0
         Business Services       6699.0
         Chemicals               3000.4
         Energy                 19710.0
         Name: Profits, dtype: float64
```

相补的 `min` 方法返回给定列中的最小值。下一个示例显示每个部门的最低员工人数。在 `"Aerospace & Defense"` 部门中，公司最少的员工人数是 5,157：

```
In  [35] sectors["Employees"].min().head()

Out [35] Sector
         Aerospace & Defense    5157
         Apparel                3700        
         Business Services      2338
         Chemicals              1931
         Energy                  593
         Name: Employees, dtype: int64
```

`agg` 方法将多个聚合操作应用于不同的列，并接受一个字典作为其参数。在每一对键值中，键表示 `DataFrame` 的列，而值指定要应用于该列的聚合操作。下一个示例提取每个部门的最低收入、最高利润和平均员工人数：

```
In  [36] aggregations = {
             "Revenues": "min",
             "Profits": "max",
             "Employees": "mean"
         }

         sectors.agg(aggregations).head()

Out [36]

                     Revenues  Profits     Employees
**Sector** 
Aerospace & Defense    1877.0   8197.0  40404.960000
Apparel                2350.0   4240.0  25407.071429
Business Services      1851.0   6699.0  30075.452830
Chemicals              1925.0   3000.4  14364.242424
Energy                 1874.0  19710.0   9170.158879
```

Pandas 返回一个 `DataFrame`，其中聚合字典的键作为列标题。部门仍然是索引标签。

## 9.5 对所有组应用自定义操作

假设我们想要对 `GroupBy` 对象中的每个嵌套组应用一个自定义操作。在第 9.4 节中，我们使用了 `GroupBy` 对象的 `max` 方法来找到每个部门的最高收入。假设我们想要识别每个部门收入最高的公司。我们之前已经解决了这个问题，但现在假设财富是无序的。

`DataFrame` 的 `nlargest` 方法提取给定列中值最大的行。这里有一个快速回顾。下一个示例返回利润列中值最大的五个财富行：

```
In  [37] fortune.nlargest(n = 5, columns = "Profits")

Out [37]

 **Company  Revenues  Profits  Employees        Sector      Industry**
3          Apple  229234.0  48351.0     123000    Technology  Computers...
2   Berkshire...  242137.0  44940.0     377000    Financials  Insurance...
15       Verizon  126034.0  30101.0     155400  Telecommu...  Telecommu...
8           AT&T  160546.0  29450.0     254000  Telecommu...  Telecommu...
19  JPMorgan ...  113899.0  24441.0     252539    Financials  Commercia...
```

如果我们能在 `sectors` 中的每个嵌套 `DataFrame` 上调用 `nlargest` 方法，我们就能得到我们想要的结果。我们将在每个部门中得到收入最高的公司。

我们可以使用 `GroupBy` 对象的 `apply` 方法在这里。该方法期望一个函数作为参数。它对 `GroupBy` 对象中的每个组调用一次函数。然后它收集函数调用的返回值，并将它们以新的 `DataFrame` 的形式返回。

首先，让我们定义一个 `get_largest_row` 函数，它接受一个参数：一个 `DataFrame`。该函数将返回 Revenues 列中值最大的 `DataFrame` 行。该函数是动态的；只要它有一个 Revenues 列，它就可以对任何 `DataFrame` 执行逻辑：

```
In  [38] def get_largest_row(df):
             return df.nlargest(1, "Revenues")
```

接下来，我们可以调用 `apply` 方法，并传入未调用的 `get_largest_row` 函数。Pandas 对每个部门调用一次 `get_largest_row`，并返回一个 `DataFrame`，其中包含每个部门收入最高的公司：

```
In  [39] sectors.apply(get_largest_row).head()

Out [39]

                        Company  Revenues  Profits  Employees      Industry
**Sector** 
Aerospace ... 26         Boeing   93392.0   8197.0     140800  Aerospace...
Apparel       88           Nike   34350.0   4240.0      74400       Apparel
Business S... 142  ManpowerG...   21034.0    545.4      29000  Temporary...
Chemicals     46      DowDuPont   62683.0   1460.0      98000     Chemicals
Energy        1     Exxon Mobil  244363.0  19710.0      71200  Petroleum...
```

当 Pandas 不支持您想要应用于每个嵌套组的自定义聚合时，请使用 `apply` 方法。

## 9.6 按多个列分组

我们可以使用来自多个 `DataFrame` 列的值来创建一个 `GroupBy` 对象。当列值的组合是最佳标识符时，此操作是最佳的。下一个示例将两个字符串的列表传递给 `groupby` 方法。Pandas 首先按 Sector 列的值分组，然后按 Industry 列的值分组。请记住，公司的行业是更大部门内的一个子类别：

```
In  [40] sector_and_industry = fortune.groupby(by = ["Sector", "Industry"])
```

`GroupBy`对象的`size`方法现在返回一个包含每个内部组行数的`MultiIndex` `Series`。这个`GroupBy`对象长度为 82，这意味着`fortune`有 82 个独特的部门与行业组合：

```
In  [41] sector_and_industry.size()

Out [41]

**Sector               Industry** 
Aerospace & Defense  Aerospace and Defense                            25
Apparel              Apparel                                          14
Business Services    Advertising, marketing                            2
                     Diversified Outsourcing Services                 14
                     Education                                         2
                                                                      ..
Transportation       Trucking, Truck Leasing                          11
Wholesalers          Wholesalers: Diversified                         24
                     Wholesalers: Electronics and Office Equipment     8
                     Wholesalers: Food and Grocery                     6
                     Wholesalers: Health Care                          6
Length: 82, dtype: int64
```

`get_group`方法需要一个值元组来从`GroupBy`集合中提取嵌套`DataFrame`。下一个示例针对部门为`"Business Services"`和行业为`"Education"`的行：

```
In  [42] sector_and_industry.get_group(("Business Services", "Education"))

Out [42]

 **Company  Revenues  Profits  Employees        Sector   Industry**
567  Laureate ...    4378.0     91.5      54500  Business ...  Education
810  Graham Ho...    2592.0    302.0      16153  Business ...  Education
```

对于所有聚合操作，pandas 返回一个包含计算的`MultiIndex DataFrame`。下一个示例计算了`fortune`中三个数值列（收入、利润和员工人数）的总和，首先按部门分组，然后按每个部门内的行业分组：

```
In  [43] sector_and_industry.sum().head()

Out [43]

                                          Revenues  Profits  Employees
**Sector              Industry** 
Aerospace & Defense Aerospace and Def...  383835.0  26733.5    1010124
Apparel             Apparel               101157.3   6350.7     355699
Business Services   Advertising, mark...   23156.0   1667.4     127500
                    Diversified Outso...   74175.0   5043.7     858600
                    Education               6970.0    393.5      70653
```

我们可以使用与第 9.5 节相同的语法来针对单个`fortune`列进行聚合。在`GroupBy`对象后输入列名，然后调用聚合方法。下一个示例计算了每个部门/行业组合中公司的平均收入：

```
In  [44] sector_and_industry["Revenues"].mean().head(5)

Out [44]

**Sector               Industry** 
Aerospace & Defense  Aerospace and Defense               15353.400000
Apparel              Apparel                              7225.521429
Business Services    Advertising, marketing              11578.000000
                     Diversified Outsourcing Services     5298.214286
                     Education                            3485.000000
Name: Revenues, dtype: float64
```

总结来说，`GroupBy`对象是一个用于分割、组织和聚合`DataFrame`值的最佳数据结构。如果你需要使用多个列来识别分组，可以将列的列表传递给`groupby`方法。

## 9.7 编码挑战

这个编码挑战的数据集，cereals.csv，是 80 种流行早餐谷物的列表。每一行包括谷物的名称、制造商、类型、卡路里、纤维克数和糖克数。让我们看一下：

```
In  [45] cereals = pd.read_csv("cereals.csv")
         cereals.head()

Out [45]

 **Name    Manufacturer  Type  Calories  Fiber  Sugars**
0            100% Bran         Nabisco  Cold        70   10.0       6
1    100% Natural Bran     Quaker Oats  Cold       120    2.0       8
2             All-Bran       Kellogg's  Cold        70    9.0       5
3  All-Bran with Ex...       Kellogg's  Cold        50   14.0       0
4       Almond Delight  Ralston Purina  Cold       110    1.0       8
```

祝你好运！

### 9.7.1 问题

这里是挑战：

1.  使用制造商列的值对谷物进行分组。

1.  确定总组数和每组中的谷物数量。

1.  提取属于制造商/组`"Nabisco"`的谷物。

1.  计算每个制造商卡路里、纤维和糖列值的平均值。

1.  找到每个制造商糖列中的最大值。

1.  找到每个制造商在纤维列中的最小值。

1.  从每个制造商中提取糖含量最低的谷物到一个新的`DataFrame`中。

### 9.7.2 解决方案

让我们深入解决方案：

1.  要按制造商对谷物进行分组，我们可以在谷物的`DataFrame`上调用`groupby`方法，并传入制造商列。Pandas 将使用列的唯一值来组织分组：

    ```
    In  [46] manufacturers = cereals.groupby("Manufacturer")
    ```

1.  要找到组/制造商的总数，我们可以将`GroupBy`对象传递给 Python 的内置`len`函数：

    ```
    In  [47] len(manufacturers)

    Out [47] 7
    ```

    如果你好奇，`GroupBy`对象的`size`方法返回一个包含每组分谷物数量的`Series`：

    ```
    In  [48] manufacturers.size()

    Out [48] Manufacturer
             American Home Food Products     1
             General Mills                  22
             Kellogg's                      23
             Nabisco                         6
             Post                            9
             Quaker Oats                     8
             Ralston Purina                  8
             dtype: int64
    ```

1.  要识别属于`"Nabisco"`组的谷物，我们可以在我们的`GroupBy`对象上调用`get_group`方法。Pandas 将返回包含`"Nabisco"`行的嵌套`DataFrame`：

    ```
    In  [49] manufacturers.get_group("Nabisco")

    Out [49]

     **Name Manufacturer  Type Calories  Fiber Sugars**
    0                  100% Bran      Nabisco  Cold       70   10.0      6
    20    Cream of Wheat (Quick)      Nabisco   Hot      100    1.0      0
    63            Shredded Wheat      Nabisco  Cold       80    3.0      0
    64    Shredded Wheat 'n'Bran      Nabisco  Cold       90    4.0      0
    65  Shredded Wheat spoon ...      Nabisco  Cold       90    3.0      0
    68   Strawberry Fruit Wheats      Nabisco  Cold       90    3.0      5
    ```

1.  要计算`cereals`中数值列的平均值，我们可以在`manufacturers`的`GroupBy`对象上调用`mean`方法。Pandas 默认会聚合`cereals`中的所有数值列：

    ```
    In  [50] manufacturers.mean()

    Out [50]

                                   Calories     Fiber    Sugars
    **Manufacturer** 
    American Home Food Products  100.000000  0.000000  3.000000
    General Mills                111.363636  1.272727  7.954545
    Kellogg's                    108.695652  2.739130  7.565217
    Nabisco                       86.666667  4.000000  1.833333
    Post                         108.888889  2.777778  8.777778
    Quaker Oats                   95.000000  1.337500  5.250000
    Ralston Purina               115.000000  1.875000  6.125000
    ```

1.  接下来，我们的任务是找到每个制造商的最大糖分值。我们可以在`GroupBy`对象后面使用方括号来标识要聚合的列的值。然后我们提供正确的聚合方法，在这种情况下是`max`：

    ```
    In  [51] manufacturers["Sugars"].max()

    Out [51] Manufacturer
             American Home Food Products     3
             General Mills                  14
             Kellogg's                      15
             Nabisco                         6
             Post                           15
             Quaker Oats                    12
             Ralston Purina                 11
             Name: Sugars, dtype: int64
    ```

1.  要找到每个制造商的最小纤维值，我们可以将列交换为纤维并调用`min`方法：

    ```
    In  [52] manufacturers["Fiber"].min()

    Out [52] Manufacturer
             American Home Food Products    0.0
             General Mills                  0.0
             Kellogg's                      0.0
             Nabisco                        1.0
             Post                           0.0
             Quaker Oats                    0.0
             Ralston Purina                 0.0
             Name: Fiber, dtype: float64
    ```

1.  最后，我们需要为每个制造商识别出糖分列中值最低的谷物行。我们可以通过使用`apply`方法和自定义函数来解决这个问题。`smallest_sugar_row`函数使用`nsmallest`方法来提取糖分列中值最小的`DataFrame`行。然后我们使用`apply`在每一个`GroupBy`组上调用自定义函数：

    ```
    In  [53] def smallest_sugar_row(df):
                 return df.nsmallest(1, "Sugars")

    In  [54] manufacturers.apply(smallest_sugar_row)

    Out [54]

                              Name  Manufacturer Type Calories Fiber Sugars
    **Manufacturer** 
    American H... 43             Maypo  American ...   Hot       100   0.0      3
    General Mills 11          Cheerios  General M...  Cold       110   2.0      0
    Nabisco       20      Cream of ...       Nabisco   Hot       100   1.0      0
    Post          33        Grape-Nuts          Post  Cold       110   3.0      3
    Quaker Oats   57      Quaker Oa...   Quaker Oats   Hot       100   2.7     -1
    Ralston Pu... 61         Rice Chex  Ralston P...  Cold       110   0.0      2
    ```

恭喜你完成了编码挑战！

## 摘要

+   `GroupBy`对象是一个`DataFrame`的容器。

+   Pandas 通过使用一个或多个列的值将行划分到`GroupBy` `DataFrame`中。

+   `first`和`last`方法从每个`GroupBy`组中返回第一行和最后一行。原始`DataFrame`中的行顺序决定了每个组中的行顺序。

+   `head`和`tail`方法根据行在原始`DataFrame`中的位置从`GroupBy`对象中的每个组中提取多行。

+   `nth`方法通过索引位置从每个`GroupBy`组中提取一行。

+   Pandas 可以通过`GroupBy`对象对每个组执行聚合计算，如求和、平均值、最大值和最小值。

+   `agg`方法将不同的聚合操作应用于不同的列。我们传递一个字典，其中列作为键，聚合作为值。

+   `apply`方法在`GroupBy`对象中的每个`DataFrame`上调用一个函数。
