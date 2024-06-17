# 第8章\. R中的数据操作和可视化

美国统计学家罗纳德·蒂斯特德曾经说过：“原始数据就像生土豆一样，通常在使用之前需要清理。” 数据操作需要时间，如果你曾经做过以下工作，你可能感受到了痛苦：

+   选择、删除或创建计算列

+   排序或筛选行

+   按组汇总和总结类别

+   通过公共字段连接多个数据集

很可能，你在Excel中已经*大量*进行了所有这些操作，并且可能深入研究了`VLOOKUP()`和透视表等著名功能来完成它们。在本章中，你将学习这些技术的R语言等效方法，特别是借助`dplyr`。

数据操作通常与可视化紧密相关：正如前文所述，人类在视觉处理信息方面非常擅长，因此这是评估数据集的一种绝佳方式。您将学习如何使用美丽的`ggplot2`包可视化数据，该包与`dplyr`一样属于`tidyverse`。这将使您在使用R探索和测试数据关系时站稳脚跟，这将在[第9章](ch09.html#r-capstone)中介绍。让我们通过调用相关的包开始。在本章中，我们还将使用书籍的伴随存储库中的*star*数据集（https://oreil.ly/lmZb7）。

```py
library(tidyverse)
library(readxl)

star <- read_excel('datasets/star/star.xlsx')
head(star)
#> # A tibble: 6 x 8
#>   tmathssk treadssk classk            totexpk sex   freelunk race  schidkn
#>      <dbl>    <dbl> <chr>               <dbl> <chr> <chr>    <chr>   <dbl>
#> 1      473      447 small.class             7 girl  no       white      63
#> 2      536      450 small.class            21 girl  no       black      20
#> 3      463      439 regular.with.aide       0 boy   yes      black      19
#> 4      559      448 regular                16 boy   no       white      69
#> 5      489      447 small.class             5 boy   yes      white      79
#> 6      454      431 regular                 8 boy   yes      white       5
```

# 使用dplyr进行数据操作

`dplyr`是一个用于操作表格数据结构的流行包。它的许多函数，或者*动词*，工作方式类似，并且可以轻松地一起使用。[表 8-1](#dplyr-grammar)列出了一些常见的`dplyr`函数及其用途；本章将详细介绍每一个。

表8-1\. `dplyr`经常使用的动词

| 功能 | 它的作用 |
| --- | --- |
| `select()` | 选择给定列 |
| `mutate()` | 根据现有列创建新列 |
| `rename()` | 对给定列进行重命名 |
| `arrange()` | 根据条件重新排序行 |
| `filter()` | 根据条件选择行 |
| `group_by()` | 根据给定列分组行 |
| `summarize()` | 汇总每个组的值 |
| `left_join()` | 将表B中匹配的记录连接到表A；如果在表B中找不到匹配项，则结果为`NA` |

为了简洁起见，我不会涵盖`dplyr`的所有函数，甚至不会涵盖我们所涵盖的函数的所有使用方式。要了解更多关于该包的信息，请查看Hadley Wickham和Garrett Grolemund（O’Reilly）的《*R for Data Science*》。您还可以通过在RStudio中导航到帮助→ 速查表→ 使用`dplyr`进行数据转换来访问一张有用的速查表，总结`dplyr`的许多函数是如何一起工作的。

## 列操作

在 Excel 中选择和删除列通常需要隐藏或删除它们。这可能会导致审计或重现困难，因为隐藏的列容易被忽略，而删除的列不容易恢复。`select()` 函数可用于从 R 的数据框中选择指定列。对于 `select()`，以及每个这样的函数，第一个参数将是要处理的数据框。然后，提供其他参数来操作该数据框中的数据。例如，我们可以像这样从 `star` 中选择 *tmathssk*、*treadssk* 和 *schidkin*：

```py
select(star, tmathssk, treadssk, schidkn)
#> # A tibble: 5,748 x 3
#>    tmathssk treadssk schidkn
#>       <dbl>    <dbl>   <dbl>
#>  1      473      447      63
#>  2      536      450      20
#>  3      463      439      19
#>  4      559      448      69
#>  5      489      447      79
#>  6      454      431       5
#>  7      423      395      16
#>  8      500      451      56
#>  9      439      478      11
#> 10      528      455      66
#> # ... with 5,738 more rows
```

我们也可以在 `select()` 函数中使用 `-` 运算符来*删除*指定列：

```py
select(star, -tmathssk, -treadssk, -schidkn)
#> # A tibble: 5,748 x 5
#>    classk            totexpk sex   freelunk race
#>    <chr>               <dbl> <chr> <chr>    <chr>
#>  1 small.class             7 girl  no       white
#>  2 small.class            21 girl  no       black
#>  3 regular.with.aide       0 boy   yes      black
#>  4 regular                16 boy   no       white
#>  5 small.class             5 boy   yes      white
#>  6 regular                 8 boy   yes      white
#>  7 regular.with.aide      17 girl  yes      black
#>  8 regular                 3 girl  no       white
#>  9 small.class            11 girl  no       black
#> 10 small.class            10 girl  no       white
```

在这里的一个更加优雅的替代方法是将所有不需要的列传递给一个向量，*然后*将其删除：

```py
select(star, -c(tmathssk, treadssk, schidkn))
#> # A tibble: 5,748 x 5
#>    classk            totexpk sex   freelunk race
#>    <chr>               <dbl> <chr> <chr>    <chr>
#>  1 small.class             7 girl  no       white
#>  2 small.class            21 girl  no       black
#>  3 regular.with.aide       0 boy   yes      black
#>  4 regular                16 boy   no       white
#>  5 small.class             5 boy   yes      white
#>  6 regular                 8 boy   yes      white
#>  7 regular.with.aide      17 girl  yes      black
#>  8 regular                 3 girl  no       white
#>  9 small.class            11 girl  no       black
#> 10 small.class            10 girl  no       white
```

请记住，在前面的示例中，我们只是调用函数而没有实际将输出分配给对象。

对于 `select()` 的另一个简写方法是使用 `:` 运算符选择两个列之间的所有内容，包括两列。这一次，我将选择从 *tmathssk* 到 *totexpk* 的所有内容，再将结果分配回 `star`：

```py
star <- select(star, tmathssk:totexpk)
head(star)
#> # A tibble: 6 x 4
#>   tmathssk treadssk classk            totexpk
#>      <dbl>    <dbl> <chr>               <dbl>
#> 1      473      447 small.class             7
#> 2      536      450 small.class            21
#> 3      463      439 regular.with.aide       0
#> 4      559      448 regular                16
#> 5      489      447 small.class             5
#> 6      454      431 regular                 8
```

您可能在 Excel 中创建了计算列；`mutate()` 将在 R 中执行相同的操作。让我们创建一个 *new_column* 列，其中包括阅读和数学分数的组合。使用 `mutate()`，我们首先提供新列的名称 *first*，然后是等号，最后是要使用的计算公式。我们可以在公式中引用其他列：

```py
star <- mutate(star, new_column = tmathssk + treadssk)
head(star)
#> # A tibble: 6 x 5
#>   tmathssk treadssk classk            totexpk new_column
#>      <dbl>    <dbl> <chr>               <dbl>      <dbl>
#> 1      473      447 small.class             7        920
#> 2      536      450 small.class            21        986
#> 3      463      439 regular.with.aide       0        902
#> 4      559      448 regular                16       1007
#> 5      489      447 small.class             5        936
#> 6      454      431 regular                 8        885
```

`mutate()` 函数使得派生比如对数变换或者滞后变量等相对复杂的计算列变得更加容易；详见帮助文档获取更多信息。

*new_column* 对于总分并不是特别有帮助的名称。幸运的是，`rename()` 函数就像其名字所暗示的那样工作。我们将指定如何命名新列，取代旧列：

```py
star <- rename(star, ttl_score = new_column)
head(star)
#> # A tibble: 6 x 5
#>   tmathssk treadssk classk            totexpk ttl_score
#>      <dbl>    <dbl> <chr>               <dbl>     <dbl>
#> 1      473      447 small.class             7       920
#> 2      536      450 small.class            21       986
#> 3      463      439 regular.with.aide       0       902
#> 4      559      448 regular                16      1007
#> 5      489      447 small.class             5       936
#> 6      454      431 regular                 8       885
```

## 逐行操作

到目前为止，我们一直在*列*上操作。现在让我们专注于*行*；具体来说是排序和过滤。在 Excel 中，我们可以使用自定义排序菜单按多列排序。例如，如果我们希望按升序排序此数据框的 *classk*，然后按 *treadssk* 排序。在 Excel 中进行此操作的菜单看起来会像 [图 8-1](#custom-sort-excel)。

![Excel 中的自定义排序菜单](assets/aina_0801.png)

###### 图 8-1\. Excel 中的自定义排序菜单

我们可以通过使用 `arrange()` 函数在 `dplyr` 中复制这一过程，按照希望数据框排序的顺序包括每一列：

```py
arrange(star, classk, treadssk)
#> # A tibble: 5,748 x 5
#>    tmathssk treadssk classk  totexpk ttl_score
#>       <dbl>    <dbl> <chr>     <dbl>     <dbl>
#>  1      320      315 regular       3       635
#>  2      365      346 regular       0       711
#>  3      384      358 regular      20       742
#>  4      384      358 regular       3       742
#>  5      320      360 regular       6       680
#>  6      423      376 regular      13       799
#>  7      418      378 regular      13       796
#>  8      392      378 regular      13       770
#>  9      392      378 regular       3       770
#> 10      399      380 regular       6       779
#> # ... with 5,738 more rows
```

如果我们希望某列按降序排序，可以将 `desc()` 函数传递给该列。

```py
# Sort by classk descending, treadssk ascending
arrange(star, desc(classk), treadssk)
#> # A tibble: 5,748 x 5
#>    tmathssk treadssk classk      totexpk ttl_score
#>       <dbl>    <dbl> <chr>         <dbl>     <dbl>
#>  1      412      370 small.class      15       782
#>  2      434      376 small.class      11       810
#>  3      423      378 small.class       6       801
#>  4      405      378 small.class       8       783
#>  5      384      380 small.class      19       764
#>  6      405      380 small.class      15       785
#>  7      439      382 small.class       8       821
#>  8      384      384 small.class      10       768
#>  9      405      384 small.class       8       789
#> 10      423      384 small.class      21       807
```

Excel 表包括有助于根据给定条件过滤任何列的下拉菜单。要在 R 中过滤数据框，我们将使用名为 `filter()` 的函数。让我们过滤 `star`，仅保留 `classk` 等于 `small.class` 的记录。请记住，因为我们检查的是相等性而不是分配对象，所以在这里我们必须使用 `==` 而不是 `=`：

```py
filter(star, classk == 'small.class')
#> # A tibble: 1,733 x 5
#>    tmathssk treadssk classk      totexpk ttl_score
#>       <dbl>    <dbl> <chr>         <dbl>     <dbl>
#>  1      473      447 small.class       7       920
#>  2      536      450 small.class      21       986
#>  3      489      447 small.class       5       936
#>  4      439      478 small.class      11       917
#>  5      528      455 small.class      10       983
#>  6      559      474 small.class       0      1033
#>  7      494      424 small.class       6       918
#>  8      478      422 small.class       8       900
#>  9      602      456 small.class      14      1058
#> 10      439      418 small.class       8       857
#> # ... with 1,723 more rows
```

从 tibble 输出中我们可以看到，我们的 `filter()` 操作 *仅* 影响了行数，*而不是* 列数。现在让我们找出 `treadssk` 至少为 `500` 的记录：

```py
filter(star, treadssk >= 500)
#> # A tibble: 233 x 5
#>    tmathssk treadssk classk            totexpk ttl_score
#>       <dbl>    <dbl> <chr>               <dbl>     <dbl>
#>  1      559      522 regular                 8      1081
#>  2      536      507 regular.with.aide       3      1043
#>  3      547      565 regular.with.aide       9      1112
#>  4      513      503 small.class             7      1016
#>  5      559      605 regular.with.aide       5      1164
#>  6      559      554 regular                14      1113
#>  7      559      503 regular                10      1062
#>  8      602      518 regular                12      1120
#>  9      536      580 small.class            12      1116
#> 10      626      510 small.class            14      1136
#> # ... with 223 more rows
```

可以使用 `&` 运算符进行多条件过滤，“与” 和 `|` 运算符进行“或”运算。让我们将之前的两个条件用 `&` 结合起来：

```py
# Get records where classk is small.class and
# treadssk is at least 500
filter(star, classk == 'small.class' & treadssk >= 500)
#> # A tibble: 84 x 5
#>    tmathssk treadssk classk      totexpk ttl_score
#>       <dbl>    <dbl> <chr>         <dbl>     <dbl>
#>  1      513      503 small.class       7      1016
#>  2      536      580 small.class      12      1116
#>  3      626      510 small.class      14      1136
#>  4      602      518 small.class       3      1120
#>  5      626      565 small.class      14      1191
#>  6      602      503 small.class      14      1105
#>  7      626      538 small.class      13      1164
#>  8      500      580 small.class       8      1080
#>  9      489      565 small.class      19      1054
#> 10      576      545 small.class      19      1121
#> # ... with 74 more rows
```

## 聚合和连接数据

我喜欢称透视表为 Excel 的“WD-40”，因为它们允许我们将数据“旋转”到不同的方向，以便进行简单的分析。例如，让我们重新创建 [Figure 8-2](#excel-pivot-table) 中显示的按班级规模的平均数学分数的透视表：

![Excel 透视表的工作原理](assets/aina_0802.png)

###### Figure 8-2\. Excel 透视表的工作原理

如 [Figure 8-2](#excel-pivot-table) 所示，此透视表包括两个要素。首先，我按变量 *classk* 进行了数据聚合。然后，我通过计算 *tmathssk* 的平均值进行了总结。在 R 中，这些是离散的步骤，使用了不同的 `dplyr` 函数。首先，我们将使用 `group_by()` 进行数据聚合。我们的输出包括一行 `# Groups: classk [3]`，表示 `star_grouped` 根据 `classk` 变量分为三组：

```py
star_grouped <- group_by(star, classk)
head(star_grouped)
#> # A tibble: 6 x 5
#> # Groups:   classk [3]
#>   tmathssk treadssk classk            totexpk ttl_score
#>      <dbl>    <dbl> <chr>               <dbl>     <dbl>
#> 1      473      447 small.class             7       920
#> 2      536      450 small.class            21       986
#> 3      463      439 regular.with.aide       0       902
#> 4      559      448 regular                16      1007
#> 5      489      447 small.class             5       936
#> 6      454      431 regular                 8       885
```

我们通过一个变量进行了 *分组*，现在让我们用另一个变量来 *汇总* 它，使用 `summarize()` 函数（`summarise()` 也可以）。在这里，我们将指定结果列的名称以及如何计算它。[Table 8-2](#dplyr-agg-types) 列出了一些常见的聚合函数。

Table 8-2\. `dplyr` 的有用聚合函数

| 函数 | 聚合类型 |
| --- | --- |
| `sum()` | 总和 |
| `n()` | 计数 |
| `mean()` | 平均值 |
| `max()` | 最高值 |
| `min()` | 最低值 |
| `sd()` | 标准差 |

我们可以通过在我们的分组数据框上运行 `summarize()` 来获取按班级规模的平均数学分数：

```py
summarize(star_grouped, avg_math = mean(tmathssk))
#> `summarise()` ungrouping output (override with `.groups` argument)
#> # A tibble: 3 x 2
#>   classk            avg_math
#>   <chr>                <dbl>
#> 1 regular               483.
#> 2 regular.with.aide     483.
#> 3 small.class           491.
```

`` `summarise()` ungrouping output `` 错误是一个警告，说明您通过聚合操作取消了分组 tibble 的分组。减去一些格式上的差异，我们得到了与 [Figure 8-2](#excel-pivot-table) 相同的结果。

如果透视表是 Excel 的 WD-40，那么 `VLOOKUP()` 就是胶带，可以轻松地从多个来源合并数据。在我们最初的 *star* 数据集中，*schidkin* 是一个学区指示器。我们在本章早些时候删除了此列，所以让我们重新读取它。但是，如果除了指示器编号外，我们还想知道这些区域的 *名称* 怎么办？幸运的是，书库中的 *districts.csv* 包含了这些信息，所以让我们将它们一起读取，并制定一个组合策略：

```py
star <- read_excel('datasets/star/star.xlsx')
head(star)
#> # A tibble: 6 x 8
#>   tmathssk treadssk classk            totexpk sex   freelunk race  schidkn
#>      <dbl>    <dbl> <chr>               <dbl> <chr> <chr>    <chr>   <dbl>
#> 1      473      447 small.class             7 girl  no       white      63
#> 2      536      450 small.class            21 girl  no       black      20
#> 3      463      439 regular.with.aide       0 boy   yes      black      19
#> 4      559      448 regular                16 boy   no       white      69
#> 5      489      447 small.class             5 boy   yes      white      79
#> 6      454      431 regular                 8 boy   yes      white       5

districts <- read_csv('datasets/star/districts.csv')

#> -- Column specification -----------------------------------------------------
#> cols(
#>   schidkn = col_double(),
#>   school_name = col_character(),
#>   county = col_character()
#> )

head(districts)
#> # A tibble: 6 x 3
#>   schidkn school_name     county
#>     <dbl> <chr>           <chr>
#> 1       1 Rosalia         New Liberty
#> 2       2 Montgomeryville Topton
#> 3       3 Davy            Wahpeton
#> 4       4 Steelton        Palestine
#> 5       6 Tolchester      Sattley
#> 6       7 Cahokia         Sattley
```

看起来需要的类似于`VLOOKUP()`的功能：我们希望从*districts*中“读入”*school_name*（可能还包括*county*）变量到*star*，给定共享的*schidkn*变量。要在R中实现这一点，我们将使用*joins*的方法，它来自关系数据库，这是在[第5章](ch05.html#data-analytics-stack)中讨论过的主题。与`VLOOKUP()`最接近的是左外连接，在`dplyr`中可以使用`left_join()`函数实现。我们将首先提供“基本”表(*star*)，然后是“查找”表(*districts*)。该函数将在*star*的每条记录中查找并返回*districts*中的匹配项，如果没有找到匹配项则返回`NA`。为了减少控制台输出的过多信息，我将仅保留来自*star*的一些列：

```py
# Left outer join star on districts
left_join(select(star, schidkn, tmathssk, treadssk), districts)
#> Joining, by = "schidkn"
#> # A tibble: 5,748 x 5
#>    schidkn tmathssk treadssk school_name     county
#>      <dbl>    <dbl>    <dbl> <chr>           <chr>
#>  1      63      473      447 Ridgeville      New Liberty
#>  2      20      536      450 South Heights   Selmont
#>  3      19      463      439 Bunnlevel       Sattley
#>  4      69      559      448 Hokah           Gallipolis
#>  5      79      489      447 Lake Mathews    Sugar Mountain
#>  6       5      454      431 NA              NA
#>  7      16      423      395 Calimesa        Selmont
#>  8      56      500      451 Lincoln Heights Topton
#>  9      11      439      478 Moose Lake      Imbery
#> 10      66      528      455 Siglerville     Summit Hill
#> # ... with 5,738 more rows
```

`left_join()`非常智能：它知道在`schidkn`上进行连接，并“查找”不仅*school_name*还包括*county*。要了解更多关于数据连接的信息，请查看帮助文档。

在R中，缺失的观察结果表示为特殊值`NA`。例如，似乎没有找到第5区的地区名的匹配项。在`VLOOKUP()`中，这将导致`#N/A`错误。`NA`并*不*意味着观察结果等于零，只是其值缺失。在编程R时，你可能会看到其他特殊值，例如`NaN`或`NULL`；要了解更多信息，请查看帮助文档。

## dplyr与管道操作符（%>%）

正如你所看到的，`dplyr`函数对于任何曾在Excel中处理数据的人来说都非常强大且直观。并且，任何曾处理过数据的人都知道，仅需一步就能准备好数据是非常罕见的。例如，你可能想用*star*进行典型的数据分析任务：

> 按班级类型查找平均阅读分数，从高到低排序。

根据我们对数据处理的了解，我们可以将其分解为三个明确的步骤：

1.  按班级类型分组我们的数据。

1.  找到每个组的平均阅读分数。

1.  将这些结果从高到低排序。

我们可以在`dplyr`中执行类似以下操作：

```py
star_grouped <- group_by(star, classk)
star_avg_reading <- summarize(star_grouped, avg_reading = mean(treadssk))
#> `summarise()` ungrouping output (override with `.groups` argument)
#>
star_avg_reading_sorted <- arrange(star_avg_reading, desc(avg_reading))
star_avg_reading_sorted
#>
#> # A tibble: 3 x 2
#>   classk            avg_reading
#>   <chr>                   <dbl>
#> 1 small.class              441.
#> 2 regular.with.aide        435.
#> 3 regular                  435.
```

这让我们得到了一个答案，但需要相当多的步骤，并且很难跟踪各种函数和对象名称。另一种方法是使用管道运算符`%>%`将这些函数链接在一起。这使我们能够将一个函数的输出直接传递给另一个函数的输入，因此我们能够避免不断重命名我们的输入和输出。此运算符的默认键盘快捷键为Windows下的Ctrl+Shift+M，Mac下的Cmd-Shift-M。

让我们重新创建上述步骤，这次使用管道操作符。我们将每个函数放在自己的一行上，用`%>%`将它们组合起来。虽然不必将每个步骤放在自己的行上，但通常出于可读性考虑。使用管道操作符时，无需突出显示整个代码块即可运行它；只需将光标放在所需选择中的任何位置并执行即可：

```py
 star %>%
   group_by(classk) %>%
   summarise(avg_reading = mean(treadssk)) %>%
   arrange(desc(avg_reading))
#> `summarise()` ungrouping output (override with `.groups` argument)
#> # A tibble: 3 x 2
#>   classk            avg_reading
#>   <chr>                   <dbl>
#> 1 small.class              441.
#> 2 regular.with.aide        435.
#> 3 regular                  435.
```

起初不再在每个函数中显式包含数据源作为参数可能会令人困惑。但是将最后一个代码块与之前的比较，你会看到这种方法效率有多高。此外，管道运算符可以与非`dplyr`函数一起使用。例如，让我们只将操作结果的前几行分配给`head()`：

```py
# Average math and reading score
# for each school district
star %>%
   group_by(schidkn) %>%
   summarise(avg_read = mean(treadssk), avg_math = mean(tmathssk)) %>%
   arrange(schidkn) %>%
   head()
#> `summarise()` ungrouping output (override with `.groups` argument)
#> # A tibble: 6 x 3
#>   schidkn avg_read avg_math
#>     <dbl>    <dbl>    <dbl>
#> 1       1     444\.     492.
#> 2       2     407\.     451.
#> 3       3     441      491.
#> 4       4     422\.     468.
#> 5       5     428\.     460.
#> 6       6     428\.     470.
```

## 使用tidyr重塑数据

尽管在R中，`group_by()`和`summarize()`与Excel中的透视表类似，但这些函数不能做到Excel透视表所能做的一切。如果你想要*重塑*数据，或者改变行和列的设置怎么办？例如，我们的*star*数据框有两列分别用于数学和阅读成绩，*tmathssk*和*treadssk*。我想将它们合并为一个名为*score*的列，并且用另一个名为*test_type*的列指示每个观察是数学还是阅读。我还想保留学校指示符*schidkn*作为分析的一部分。

[图8-3](#excel-reshape)展示了在Excel中的样子；请注意，我将*Values*字段从*tmathssk*和*treadssk*改名为*math*和*reading*。如果你想进一步查看此透视表，它在[书籍仓库中作为*ch-8.xlsx*可供查阅](https://oreil.ly/Kq93s)。在这里我再次使用了索引列；否则，透视表将尝试通过*schidkn*“汇总”所有值。

![Excel中的重塑](assets/aina_0803.png)

###### 图8-3\. 在Excel中重塑*star*

我们可以使用`tidyr`，这是`tidyverse`核心包，来重塑*star*。在R中重塑时，添加一个索引列也会很有帮助，就像在Excel中一样。我们可以用`row_number()`函数来创建一个：

```py
star_pivot <- star %>%
                select(c(schidkn, treadssk, tmathssk)) %>%
                mutate(id = row_number())
```

要重塑数据框，我们将使用`tidyr`中的`pivot_longer()`和`pivot_wider()`。在你的脑海中和[图8-3](#excel-reshape)中考虑一下，如果我们将*tmathssk*和*treadssk*的分数合并到一列中，数据集会发生什么变化。数据集会变得更长，因为我们在这里添加了行。要使用`pivot_longer()`，我们将使用`cols`参数指定要拉长的列，并使用`values_to`来命名结果列。我们还将使用`names_to`来命名指示每个分数是数学还是阅读的列：

```py
star_long <- star_pivot %>%
                 pivot_longer(cols = c(tmathssk, treadssk),
                              values_to = 'score', names_to = 'test_type')
head(star_long)
#> # A tibble: 6 x 4
#>   schidkn    id test_type score
#>     <dbl> <int> <chr>     <dbl>
#> 1      63     1 tmathssk    473
#> 2      63     1 treadssk    447
#> 3      20     2 tmathssk    536
#> 4      20     2 treadssk    450
#> 5      19     3 tmathssk    463
#> 6      19     3 treadssk    439
```

很好。但是是否有一种方法可以将*tmathssk*和*treadssk*重命名为*math*和*reading*呢？有的，使用`recode()`，这是`dplyr`中另一个有用的函数，可以与`mutate()`一起使用。`recode()`与包中的其他函数有些不同，因为我们在等号之前包括“旧”值的名称，然后是新值。`dplyr`中的`distinct()`函数将确认所有行都已命名为*math*或*reading*：

```py
# Rename tmathssk and treadssk as math and reading
star_long <- star_long %>%
   mutate(test_type = recode(test_type,
                            'tmathssk' = 'math', 'treadssk' = 'reading'))

distinct(star_long, test_type)
#> # A tibble: 2 x 1
#>   test_type
#>   <chr>
#> 1 math
#> 2 reading
```

现在我们的数据框已经延长了，我们可以   现在我们的数据框已经拉长，我们可以使用`pivot_wider()`将其扩展回来。这次，我将指定哪些列包含其行中的值应该作为列，使用`values_from`，以及结果列的名称使用`names_from`：

```py
star_wide <- star_long %>%
                pivot_wider(values_from = 'score', names_from = 'test_type')
head(star_wide)
#> # A tibble: 6 x 4
#>   schidkn    id  math reading
#>     <dbl> <int> <dbl>   <dbl>
#> 1      63     1   473     447
#> 2      20     2   536     450
#> 3      19     3   463     439
#> 4      69     4   559     448
#> 5      79     5   489     447
#> 6       5     6   454     431
```

数据重塑在R中是相对复杂的操作，所以在疑惑时，问问自己：*我是在将数据做宽还是做长？在数据透视表中我该怎么做？* 如果你能逻辑地走通实现所需最终状态的过程，编码就会容易得多。

# 使用ggplot2进行数据可视化

`dplyr`还能做很多帮助我们操作数据的事情，但现在让我们将注意力转向数据可视化。具体来说，我们将关注另一个`tidyverse`包，`ggplot2`。该包的名称和模型是根据计算机科学家利兰·威尔金森（Leland Wilkinson）设计的“图形语法”来命名和建模的，`ggplot2`提供了一种构建图形的有序方法。这种结构模仿了语音元素如何结合在一起形成句子，因此被称为“图形语法”。

我将在这里介绍一些`ggplot2`的基本元素和图表类型。更多关于该包的信息，请查阅包的原作者哈德利·威克汉姆（Hadley Wickham）所著的*ggplot2: Elegant Graphics for Data Analysis*（Springer）。你还可以通过在RStudio中导航到Help → Cheatsheets → Data Visualization with ggplot2来访问一个有用的速查表。一些`ggplot2`的基本元素可以在[表8-3](#elements-of-ggplot2)中找到。还有其他元素可用；更多信息，请查看前面提到的资源。

表8-3。`ggplot2`的基础元素

| 元素 | 描述 |
| --- | --- |
| `data` | 数据源 |
| `aes` | 从数据到视觉属性（x轴和y轴、颜色、大小等）的美学映射 |
| `geom` | 图中观察到的几何对象类型（线条、条形、点等） |

让我们从可视化*classk*每个级别的观测数量作为条形图开始。我们将从`ggplot()`函数开始，指定[表8-3](#elements-of-ggplot2)中的三个元素：

```py
ggplot(data = star, ![1](assets/1.png)
          aes(x = classk)) + ![2](assets/2.png)
   geom_bar() ![3](assets/3.png)
```

[![1](assets/1.png)](#co_data_manipulation_and_visualization_in_r_CO1-1)

数据源通过`data`参数指定。

[![2](assets/2.png)](#co_data_manipulation_and_visualization_in_r_CO1-2)

从数据到可视化的美学映射通过`aes()`函数指定。在这里，我们要求*classk*映射到最终图表的x轴。

[![3](assets/3.png)](#co_data_manipulation_and_visualization_in_r_CO1-3)

我们通过`geom_bar()`函数根据指定的数据和美学映射绘制一个几何对象。结果显示在[图8-4](#barplot-ggplot2)中。

![计数图](assets/aina_0804.png)

###### 图8-4。`ggplot2`中的条形图

类似于管道操作符，不必将每一层图形放在单独的行中，但通常为了可读性而更倾向于这样做。还可以通过将光标放置在代码块的任何位置并运行来执行整个绘图。

因其模块化方法，使用 `ggplot2` 迭代可视化很容易。例如，我们可以通过将 *x* 映射更改为 *treadssk* 并使用 `geom_histogram()` 绘制结果，将我们的绘图切换为直方图。这导致了图 [8-5](#histogram-ggplot2) 中显示的直方图：

```py
ggplot(data = star, aes(x = treadssk)) +
  geom_histogram()

#> `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![直方图](assets/aina_0805.png)

###### 图 8-5\. `ggplot2` 中的直方图

还有许多方法可以自定义 `ggplot2` 绘图。例如，您可能已经注意到，上一个绘图的输出消息指出直方图使用了 30 个柱。让我们将该数字更改为 25，并在 `geom_histogram()` 中使用粉色填充以及几个其他参数。这导致了图 [8-6](#custom-histogram-ggplot2) 中显示的直方图：

```py
ggplot(data = star, aes(x = treadssk)) +
  geom_histogram(bins = 25, fill = 'pink')
```

![自定义直方图](assets/aina_0806.png)

###### 图 8-6\. `ggplot2` 中的自定义直方图

使用 `geom_boxplot()` 创建箱线图，如 [图 8-7](#boxplot-ggplot2) 所示：

```py
ggplot(data = star, aes(x = treadssk)) +
  geom_boxplot()
```

![箱线图](assets/aina_0807.png)

###### 图 8-7\. 箱线图

到目前为止的任何情况下，我们都可以通过将兴趣变量包含在 `y` 映射中而不是 `x` 映射中来“翻转”图形。让我们尝试在我们的箱线图中进行此操作。图 [8-8](#reverse-boxplot-ggplot2) 展示了以下操作的结果：

```py
ggplot(data = star, aes(y = treadssk)) +
  geom_boxplot()
```

![翻转的箱线图](assets/aina_0808.png)

###### 图 8-8\. “翻转”的箱线图

现在让我们通过将 *classk* 映射到 x 轴和 *treadssk* 映射到 y 轴，为每个班级大小级别制作一个箱线图。这导致了图 [8-9](#grouped-boxplot-ggplot2) 中显示的箱线图：

```py
ggplot(data = star, aes(x = classk, y = treadssk)) +
  geom_boxplot()
```

同样，我们可以使用 `geom_point()` 在 x 和 y 轴上绘制 *tmathssk* 和 *treadssk* 的关系，作为散点图。这导致了图 [8-10](#scatterplot-ggplot2) 中显示的结果：

```py
ggplot(data = star, aes(x = tmathssk, y = treadssk)) +
  geom_point()
```

![分组箱线图](assets/aina_0809.png)

###### 图 8-9\. 按组绘制的箱线图

![散点图](assets/aina_0810.png)

###### 图 8-10\. 散点图

我们可以使用一些额外的 `ggplot2` 函数在 x 和 y 轴上叠加标签，以及一个绘图标题。图 [8-11](#labeled-plot-ggplot2) 展示了结果：

```py
ggplot(data = star, aes(x = tmathssk, y = treadssk)) +
  geom_point() +
  xlab('Math score') + ylab('Reading score') +
  ggtitle('Math score versus reading score')
```

![具有自定义标签和标题的散点图](assets/aina_0811.png)

###### 图 8-11\. 带有自定义轴标签和标题的散点图

# 结论

`dplyr` 和 `ggplot2` 还可以做更多事情，但这已足以让您开始真正的任务：探索和测试数据中的关系。这将是 [第9章](ch09.html#r-capstone) 的重点。

# 练习

[书籍库](https://oreil.ly/kBk3e)在 *datasets* 的 *census* 子文件夹中有两个文件，*census.csv* 和 *census-divisions.csv*。将它们读入 R 并执行以下操作：

1.  按地区升序、部门升序和人口降序对数据进行排序（您需要合并数据集来执行此操作）。将结果写入 Excel 工作表。

1.  从合并数据集中删除邮政编码字段。

1.  创建一个名为*density*的新列，计算人口除以土地面积的结果。

1.  可视化2015年所有观测点的土地面积与人口之间的关系。

1.  查找2015年每个地区的总人口。

1.  创建一个包含州名和人口的表格，每年2010年至2015年的人口分别放在一个单独的列中。
