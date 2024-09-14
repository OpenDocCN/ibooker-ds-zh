# 第七章：数据导入

# 简介

使用 R 包提供的数据是学习数据科学工具的好方法，但您想要将所学应用于自己的数据。在本章中，您将学习将数据文件读入 R 的基础知识。

具体来说，本章将重点介绍读取纯文本矩形文件。我们将从处理列名、类型和缺失数据的实用建议开始。然后，您将了解如何一次从多个文件读取数据，并将数据从 R 写入文件。最后，您将学习如何在 R 中手工制作数据框架。

## 先决条件

在本章中，您将学习如何使用 readr 包在 R 中加载平面文件，该包是核心 tidyverse 的一部分：

```
library(tidyverse)
```

# 从文件中读取数据

首先，我们将重点放在最常见的矩形数据文件类型上：CSV，即“逗号分隔值”。这是一个简单的 CSV 文件的示例。第一行，通常称为*标题行*，给出列名，接下来的六行提供数据。列由逗号分隔。

```
Student ID,Full Name,favourite.food,mealPlan,AGE
1,Sunil Huffmann,Strawberry yoghurt,Lunch only,4
2,Barclay Lynn,French fries,Lunch only,5
3,Jayendra Lyne,N/A,Breakfast and lunch,7
4,Leon Rossini,Anchovies,Lunch only,
5,Chidiegwu Dunkel,Pizza,Breakfast and lunch,five
6,Güvenç Attila,Ice cream,Lunch only,6
```

表 7-1 表示相同的数据作为表格。

表 7-1\. students.csv 文件中的数据表

| 学生 ID | 全名 | 最喜欢的食物 | 餐饮计划 | 年龄 |
| --- | --- | --- | --- | --- |
| 1 | Sunil Huffmann | Strawberry yoghurt | 仅午餐 | 4 |
| 2 | Barclay Lynn | French fries | 仅午餐 | 5 |
| 3 | Jayendra Lyne | N/A | 早餐和午餐 | 7 |
| 4 | Leon Rossini | Anchovies | 仅午餐 | NA |
| 5 | Chidiegwu Dunkel | Pizza | 早餐和午餐 | five |
| 6 | Güvenç Attila | Ice cream | 仅午餐 | 6 |

我们可以使用[`read_csv()`](https://readr.tidyverse.org/reference/read_delim.xhtml)将这个文件读入 R。第一个参数是最重要的：文件的路径。你可以把路径看作是文件的地址：文件名为`students.csv`，存放在`data`文件夹中。

```
students <- read_csv("data/students.csv")
#> Rows: 6 Columns: 5
#> ── Column specification ─────────────────────────────────────────────────────
#> Delimiter: ","
#> chr (4): Full Name, favourite.food, mealPlan, AGE
#> dbl (1): Student ID
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

如果您在项目的`data`文件夹中有`students.csv`文件，前面的代码将起作用。您可以下载[`students.csv`文件](https://oreil.ly/GDubb)，或者直接从该 URL 读取它：

```
students <- read_csv("https://pos.it/r4ds-students-csv")
```

当您运行[`read_csv()`](https://readr.tidyverse.org/reference/read_delim.xhtml)时，它会输出一条消息，告诉您数据的行数和列数，使用的分隔符以及列规格（按列包含的数据类型命名的列名）。它还打印出一些有关检索完整列规格及如何静音此消息的信息。这条消息是 readr 的一个重要部分，我们将在“控制列类型”中返回它。

## 实用建议

读取数据后，通常的第一步是以某种方式转换数据，以便在后续分析中更容易处理。让我们再次查看`students`数据：

```
students
#> # A tibble: 6 × 5
#>   `Student ID` `Full Name`      favourite.food     mealPlan            AGE 
#>          <dbl> <chr>            <chr>              <chr>               <chr>
#> 1            1 Sunil Huffmann   Strawberry yoghurt Lunch only          4 
#> 2            2 Barclay Lynn     French fries       Lunch only          5 
#> 3            3 Jayendra Lyne    N/A                Breakfast and lunch 7 
#> 4            4 Leon Rossini     Anchovies          Lunch only          <NA> 
#> 5            5 Chidiegwu Dunkel Pizza              Breakfast and lunch five 
#> 6            6 Güvenç Attila    Ice cream          Lunch only          6
```

在`favourite.food`列中，有一堆食物项目，以及字符字符串`N/A`，本应是 R 识别的真正`NA`，表示“不可用”。我们可以使用`na`参数来解决这个问题。默认情况下，[`read_csv()`](https://readr.tidyverse.org/reference/read_delim.xhtml)仅识别此数据集中的空字符串（`""`）为`NA`；我们希望它也能识别字符字符串`"N/A"`：

```
students <- read_csv("data/students.csv", na = c("N/A", ""))

students
#> # A tibble: 6 × 5
#>   `Student ID` `Full Name`      favourite.food     mealPlan            AGE 
#>          <dbl> <chr>            <chr>              <chr>               <chr>
#> 1            1 Sunil Huffmann   Strawberry yoghurt Lunch only          4 
#> 2            2 Barclay Lynn     French fries       Lunch only          5 
#> 3            3 Jayendra Lyne    <NA>               Breakfast and lunch 7 
#> 4            4 Leon Rossini     Anchovies          Lunch only          <NA> 
#> 5            5 Chidiegwu Dunkel Pizza              Breakfast and lunch five 
#> 6            6 Güvenç Attila    Ice cream          Lunch only          6
```

您可能还注意到`Student ID`和`Full Name`列被反引号包围。这是因为它们包含空格，违反了 R 变量名称的常规规则；它们是*nonsyntactic*名称。要引用这些变量，您需要使用反引号 `` ` `` 包围它们：

```
students |> 
  rename(
    student_id = `Student ID`,
    full_name = `Full Name`
  )
#> # A tibble: 6 × 5
#>   student_id full_name        favourite.food     mealPlan            AGE 
#>        <dbl> <chr>            <chr>              <chr>               <chr>
#> 1          1 Sunil Huffmann   Strawberry yoghurt Lunch only          4 
#> 2          2 Barclay Lynn     French fries       Lunch only          5 
#> 3          3 Jayendra Lyne    <NA>               Breakfast and lunch 7 
#> 4          4 Leon Rossini     Anchovies          Lunch only          <NA> 
#> 5          5 Chidiegwu Dunkel Pizza              Breakfast and lunch five 
#> 6          6 Güvenç Attila    Ice cream          Lunch only          6
```

另一种方法是使用[`janitor::clean_names()`](https://rdrr.io/pkg/janitor/man/clean_names.xhtml)一次性使用一些启发式方法将它们全部转换为蛇形命名:¹

```
students |> janitor::clean_names()
#> # A tibble: 6 × 5
#>   student_id full_name        favourite_food     meal_plan           age 
#>        <dbl> <chr>            <chr>              <chr>               <chr>
#> 1          1 Sunil Huffmann   Strawberry yoghurt Lunch only          4 
#> 2          2 Barclay Lynn     French fries       Lunch only          5 
#> 3          3 Jayendra Lyne    <NA>               Breakfast and lunch 7 
#> 4          4 Leon Rossini     Anchovies          Lunch only          <NA> 
#> 5          5 Chidiegwu Dunkel Pizza              Breakfast and lunch five 
#> 6          6 Güvenç Attila    Ice cream          Lunch only          6
```

读取数据后的另一个常见任务是考虑变量类型。例如，`meal_plan`是一个具有已知可能值集合的分类变量，在 R 中应表示为因子：

```
students |>
  janitor::clean_names() |>
  mutate(meal_plan = factor(meal_plan))
#> # A tibble: 6 × 5
#>   student_id full_name        favourite_food     meal_plan           age 
#>        <dbl> <chr>            <chr>              <fct>               <chr>
#> 1          1 Sunil Huffmann   Strawberry yoghurt Lunch only          4 
#> 2          2 Barclay Lynn     French fries       Lunch only          5 
#> 3          3 Jayendra Lyne    <NA>               Breakfast and lunch 7 
#> 4          4 Leon Rossini     Anchovies          Lunch only          <NA> 
#> 5          5 Chidiegwu Dunkel Pizza              Breakfast and lunch five 
#> 6          6 Güvenç Attila    Ice cream          Lunch only          6
```

注意，`meal_plan` 变量中的值保持不变，但是在变量名下标有所不同，从字符（`<chr>`）变为因子（`<fct>`）。有关因子的详细信息请参见第十六章。

在分析这些数据之前，您可能希望修复`age`和`id`列。当前，`age`是一个字符变量，因为其中一条观察结果被输入为`five`而不是数字`5`。我们将在第二十章讨论解决此问题的详细信息。

```
students <- students |>
  janitor::clean_names() |>
  mutate(
    meal_plan = factor(meal_plan),
    age = parse_number(if_else(age == "five", "5", age))
  )

students
#> # A tibble: 6 × 5
#>   student_id full_name        favourite_food     meal_plan             age
#>        <dbl> <chr>            <chr>              <fct>               <dbl>
#> 1          1 Sunil Huffmann   Strawberry yoghurt Lunch only              4
#> 2          2 Barclay Lynn     French fries       Lunch only              5
#> 3          3 Jayendra Lyne    <NA>               Breakfast and lunch     7
#> 4          4 Leon Rossini     Anchovies          Lunch only             NA
#> 5          5 Chidiegwu Dunkel Pizza              Breakfast and lunch     5
#> 6          6 Güvenç Attila    Ice cream          Lunch only              6
```

新的函数是[`if_else()`](https://dplyr.tidyverse.org/reference/if_else.xhtml)，它有三个参数。第一个参数`test`应该是一个逻辑向量。当`test`为`TRUE`时，结果将包含第二个参数`yes`的值；当`test`为`FALSE`时，结果将包含第三个参数`no`的值。在这里我们说，如果`age`是字符字符串`"five"`，则将其变为`"5"`，否则保持为`age`。有关[`if_else()`](https://dplyr.tidyverse.org/reference/if_else.xhtml)和逻辑向量的更多信息，请参见第十二章。

## 其他参数

还有几个重要的参数需要提到，如果我们首先向您展示一个方便的技巧将更容易演示：[`read_csv()`](https://readr.tidyverse.org/reference/read_delim.xhtml)可以读取您创建并格式化为 CSV 文件样式的文本字符串：

```
read_csv(
  "a,b,c
 1,2,3
 4,5,6"
)
#> # A tibble: 2 × 3
#>       a     b     c
#>   <dbl> <dbl> <dbl>
#> 1     1     2     3
#> 2     4     5     6
```

通常，[`read_csv()`](https://readr.tidyverse.org/reference/read_delim.xhtml)使用数据的第一行作为列名，这是一种常见的约定。但是在文件顶部可能包含几行元数据并不罕见。您可以使用`skip = n`跳过前`n`行，或使用`comment = "#"`删除以`#`开头的所有行，例如：

```
read_csv(
  "The first line of metadata
 The second line of metadata
 x,y,z
 1,2,3",
  skip = 2
)
#> # A tibble: 1 × 3
#>       x     y     z
#>   <dbl> <dbl> <dbl>
#> 1     1     2     3

read_csv(
  "# A comment I want to skip
 x,y,z
 1,2,3",
  comment = "#"
)
#> # A tibble: 1 × 3
#>       x     y     z
#>   <dbl> <dbl> <dbl>
#> 1     1     2     3
```

在其他情况下，数据可能没有列名。您可以使用 `col_names = FALSE` 来告诉 [`read_csv()`](https://readr.tidyverse.org/reference/read_delim.xhtml) 不将第一行视为标题，而是按顺序从 `X1` 到 `Xn` 进行标记：

```
read_csv(
  "1,2,3
 4,5,6",
  col_names = FALSE
)
#> # A tibble: 2 × 3
#>      X1    X2    X3
#>   <dbl> <dbl> <dbl>
#> 1     1     2     3
#> 2     4     5     6
```

或者，您可以将 `col_names` 传递为字符向量，用作列名：

```
read_csv(
  "1,2,3
 4,5,6",
  col_names = c("x", "y", "z")
)
#> # A tibble: 2 × 3
#>       x     y     z
#>   <dbl> <dbl> <dbl>
#> 1     1     2     3
#> 2     4     5     6
```

这些参数是你在实践中大部分会遇到的 CSV 文件所需知道的全部内容。（对于其余部分，你需要仔细检查你的 `.csv` 文件并阅读 [`read_csv()`](https://readr.tidyverse.org/reference/read_delim.xhtml) 的文档中的许多其他参数。）

## 其他文件类型

一旦掌握了 [`read_csv()`](https://readr.tidyverse.org/reference/read_delim.xhtml)，使用 readr 的其他函数就很简单了；关键是知道要使用哪个函数：

[`read_csv2()`](https://readr.tidyverse.org/reference/read_delim.xhtml)

读取以分号分隔的文件。这些文件使用 `;` 而不是 `,` 来分隔字段，是在使用 `,` 作为小数点分隔符的国家中很常见的。

[`read_tsv()`](https://readr.tidyverse.org/reference/read_delim.xhtml)

读取制表符分隔的文件。

[`read_delim()`](https://readr.tidyverse.org/reference/read_delim.xhtml)

读取具有任何分隔符的文件，如果不指定分隔符，则尝试自动猜测分隔符。

[`read_fwf()`](https://readr.tidyverse.org/reference/read_fwf.xhtml)

读取固定宽度文件。您可以使用 [`fwf_widths()`](https://readr.tidyverse.org/reference/read_fwf.xhtml) 指定字段的宽度，或者使用 [`fwf_positions()`](https://readr.tidyverse.org/reference/read_fwf.xhtml) 指定字段的位置。

[`read_table()`](https://readr.tidyverse.org/reference/read_table.xhtml)

读取常见的固定宽度文件变体，其中列由空格分隔。

[`read_log()`](https://readr.tidyverse.org/reference/read_log.xhtml)

读取 Apache 风格的日志文件。

## 练习

1.  用什么函数可以读取以 | 分隔字段的文件？

1.  除了 `file`、`skip` 和 `comment` 外，[`read_csv()`](https://readr.tidyverse.org/reference/read_delim.xhtml) 和 [`read_tsv()`](https://readr.tidyverse.org/reference/read_delim.xhtml) 还有哪些共同的参数？

1.  是 [`read_fwf()`](https://readr.tidyverse.org/reference/read_fwf.xhtml) 最重要的参数是什么？

1.  有时 CSV 文件中的字符串包含逗号。为了防止它们引起问题，它们需要用引号字符（如 `"` 或 `'`）括起来。默认情况下，[`read_csv()`](https://readr.tidyverse.org/reference/read_delim.xhtml) 假定引号字符是 `"`. 要将以下文本读入数据框，您需要指定 [`read_csv()`](https://readr.tidyverse.org/reference/read_delim.xhtml) 的哪个参数？

    ```
    "x,y\n1,'a,b'"
    ```

1.  确定以下内联 CSV 文件中的问题。运行代码时会发生什么？

    ```
    read_csv("a,b\n1,2,3\n4,5,6")
    read_csv("a,b,c\n1,2\n1,2,3,4")
    read_csv("a,b\n\"1")
    read_csv("a,b\n1,2\na,b")
    read_csv("a;b\n1;3")
    ```

1.  在以下数据框中，通过练习引用非语法名称：

    1.  提取名为 `1` 的变量。

    1.  绘制 `1` 对 `2` 的散点图。

    1.  创建一个名为 `3` 的新列，它是 `2` 除以 `1`。

    1.  将列重命名为 `one`，`two` 和 `three`：

    ```
    annoying <- tibble(
      `1` = 1:10,
      `2` = `1` * 2 + rnorm(length(`1`))
    )
    ```

# 控制列类型

CSV 文件不包含每个变量的类型信息（即它是逻辑值、数字、字符串等），因此 readr 将尝试猜测类型。本节描述了猜测过程的工作方式，如何解决一些导致猜测失败的常见问题，以及如果需要的话如何自己提供列类型。最后，我们将提及一些一般策略，如果 readr 失败严重，您需要获取有关文件结构的更多信息时可以使用。

## 猜测类型

readr 使用一种启发式方法来确定列类型。对于每一列，它从第一行到最后一行均匀地抽取 1,000² 个值，忽略缺失值。然后它按以下问题逐步进行：

+   它只包含 `F`，`T`，`FALSE` 或 `TRUE`（忽略大小写）吗？如果是，那么它是逻辑值。

+   它只包含数字（例如，`1`，`-4.5`，`5e6`，`Inf`）吗？如果是，那么它是一个数字。

+   它是否符合 ISO8601 标准？如果是，那么它是日期或日期时间（我们将在“创建日期/时间”中详细讨论日期时间）。

+   否则，它必须是一个字符串。

您可以在这个简单的例子中看到这种行为：

```
read_csv("
 logical,numeric,date,string
 TRUE,1,2021-01-15,abc
 false,4.5,2021-02-15,def
 T,Inf,2021-02-16,ghi
")
#> # A tibble: 3 × 4
#>   logical numeric date       string
#>   <lgl>     <dbl> <date>     <chr> 
#> 1 TRUE        1   2021-01-15 abc 
#> 2 FALSE       4.5 2021-02-15 def 
#> 3 TRUE      Inf   2021-02-16 ghi
```

如果您有一个干净的数据集，这种启发式方法效果很好，但是在实际生活中，您会遇到各种奇怪而美丽的失败。

## 缺失值、列类型和问题

列检测最常见的失败方式是，一列包含意外值，导致您得到一个字符列而不是更具体的类型。其中最常见的原因之一是缺失值，使用的不是 readr 期望的 `NA`。

以这个简单的单列 CSV 文件为例：

```
simple_csv <- "
 x
 10
 .
 20
 30"
```

如果我们没有添加任何额外的参数读取它，`x` 将成为一个字符列：

```
read_csv(simple_csv)
#> # A tibble: 4 × 1
#>   x 
#>   <chr>
#> 1 10 
#> 2 . 
#> 3 20 
#> 4 30
```

在这个小例子中，您可以轻松看到缺失值 `.`。但是，如果您有成千上万行中间夹杂着几个 `.` 表示的缺失值会发生什么？一种方法是告诉 readr `x` 是一个数字列，然后查看它失败的地方。您可以使用 `col_types` 参数来做到这一点，该参数接受一个命名列表，其中名称与 CSV 文件中的列名匹配：

```
df <- read_csv(
  simple_csv, 
  col_types = list(x = col_double())
)
#> Warning: One or more parsing issues, call `problems()` on your data frame for
#> details, e.g.:
#>   dat <- vroom(...)
#>   problems(dat)
```

现在[`read_csv()`](https://readr.tidyverse.org/reference/read_delim.xhtml) 报告了问题，并告诉我们可以通过[`problems()`](https://readr.tidyverse.org/reference/problems.xhtml)了解更多信息：

```
problems(df)
#> # A tibble: 1 × 5
#>     row   col expected actual file 
#>   <int> <int> <chr>    <chr>  <chr> 
#> 1     3     1 a double .      /private/tmp/RtmpAYlSop/file392d445cf269
```

这告诉我们，在第 3 行第 1 列出现了问题，readr 期望是一个 double，但得到了一个 `.`。这表明该数据集使用 `.` 表示缺失值。因此，我们设置 `na = "."`，自动猜测成功，给我们了我们想要的数字列：

```
read_csv(simple_csv, na = ".")
#> # A tibble: 4 × 1
#>       x
#>   <dbl>
#> 1    10
#> 2    NA
#> 3    20
#> 4    30
```

## 列类型

readr 提供了总共九种列类型供您使用：

+   [`col_logical()`](https://readr.tidyverse.org/reference/parse_atomic.xhtml) 和 [`col_double()`](https://readr.tidyverse.org/reference/parse_atomic.xhtml) 分别读取逻辑值和实数。它们相对较少使用（除非像前面展示的那样），因为 readr 通常会为你猜测这些类型。

+   [`col_integer()`](https://readr.tidyverse.org/reference/parse_atomic.xhtml) 读取整数。在这本书中我们很少区分整数和双精度浮点数，因为它们在功能上是等效的，但明确读取整数有时也很有用，因为它们只占双精度浮点数内存的一半。

+   [`col_character()`](https://readr.tidyverse.org/reference/parse_atomic.xhtml) 读取字符串。当你有一个列是数字标识符，即长序列的数字，用来标识一个对象但不适合进行数学运算时，这将会很有用。例如电话号码、社会保险号、信用卡号等。

+   [`col_factor()`](https://readr.tidyverse.org/reference/parse_factor.xhtml), [`col_date()`](https://readr.tidyverse.org/reference/parse_datetime.xhtml), 和 [`col_datetime()`](https://readr.tidyverse.org/reference/parse_datetime.xhtml) 分别创建因子、日期和日期时间；当我们讨论到这些数据类型时（请见第十六章 和 第十七章），你会进一步学习这些内容。

+   [`col_number()`](https://readr.tidyverse.org/reference/parse_number.xhtml) 是一种宽松的数值解析器，它会忽略非数值组件，特别适用于货币。你将在第十三章中进一步了解它。

+   [`col_skip()`](https://readr.tidyverse.org/reference/col_skip.xhtml) 跳过一个列，使其不包含在结果中，如果你有一个大的 CSV 文件，并且只想使用部分列，这将会很有用。

也可以通过从 [`list()`](https://rdrr.io/r/base/list.xhtml) 切换到 [`cols()`](https://readr.tidyverse.org/reference/cols.xhtml) 并指定 `.default` 来覆盖默认列。

```
another_csv <- "
x,y,z
1,2,3"

read_csv(
  another_csv, 
  col_types = cols(.default = col_character())
)
#> # A tibble: 1 × 3
#>   x     y     z 
#>   <chr> <chr> <chr>
#> 1 1     2     3
```

另一个有用的辅助函数是 [`cols_only()`](https://readr.tidyverse.org/reference/cols.xhtml)，它将只读取你指定的列：

```
read_csv(
  another_csv,
  col_types = cols_only(x = col_character())
)
#> # A tibble: 1 × 1
#>   x 
#>   <chr>
#> 1 1
```

# 从多个文件读取数据

有时候你的数据分布在多个文件中，而不是单个文件。例如，你可能有多个月份的销售数据，每个月份的数据保存在单独的文件中：`01-sales.csv` 表示一月，`02-sales.csv` 表示二月，`03-sales.csv` 表示三月。使用 [`read_csv()`](https://readr.tidyverse.org/reference/read_delim.xhtml)，你可以一次性读取这些数据，并将它们堆叠在一个单一的数据框中。

```
sales_files <- c("data/01-sales.csv", "data/02-sales.csv", "data/03-sales.csv")
read_csv(sales_files, id = "file")
#> # A tibble: 19 × 6
#>   file              month    year brand  item     n
#>   <chr>             <chr>   <dbl> <dbl> <dbl> <dbl>
#> 1 data/01-sales.csv January  2019     1  1234     3
#> 2 data/01-sales.csv January  2019     1  8721     9
#> 3 data/01-sales.csv January  2019     1  1822     2
#> 4 data/01-sales.csv January  2019     2  3333     1
#> 5 data/01-sales.csv January  2019     2  2156     9
#> 6 data/01-sales.csv January  2019     2  3987     6
#> # … with 13 more rows
```

如果您的 CSV 文件位于项目的 `data` 文件夹中，前面的代码将再次起作用。您可以从 [*https://oreil.ly/jVd8o*](https://oreil.ly/jVd8o)、[*https://oreil.ly/RYsgM*](https://oreil.ly/RYsgM) 和 [*https://oreil.ly/4uZOm*](https://oreil.ly/4uZOm) 下载这些文件，或直接读取它们：

```
sales_files <- c(
  "https://pos.it/r4ds-01-sales",
  "https://pos.it/r4ds-02-sales",
  "https://pos.it/r4ds-03-sales"
)
read_csv(sales_files, id = "file")
```

`id` 参数会向结果数据框中添加一个名为 `file` 的新列，用于标识数据来自哪个文件。在文件没有可帮助您追溯观察结果到原始来源的标识列的情况下，这尤为有用。

如果您要读取许多文件，逐个写出它们的名称作为列表可能会很麻烦。相反，您可以使用基础函数 [`list.files()`](https://rdrr.io/r/base/list.files.xhtml) 通过匹配文件名中的模式来查找文件。您将在 第十五章 中更多了解这些模式。

```
sales_files <- list.files("data", pattern = "sales\\.csv$", full.names = TRUE)
sales_files
#> [1] "data/01-sales.csv" "data/02-sales.csv" "data/03-sales.csv"
```

# 写入文件

readr 还提供了两个将数据写入磁盘的实用函数：[`write_csv()`](https://readr.tidyverse.org/reference/write_delim.xhtml) 和 [`write_tsv()`](https://readr.tidyverse.org/reference/write_delim.xhtml)。这些函数的最重要参数是 `x`（要保存的数据框）和 `file`（保存位置）。您还可以使用 `na` 指定如何写入缺失值，以及是否要 `append` 到现有文件中。

```
write_csv(students, "students.csv")
```

现在让我们再次读取该 CSV 文件。请注意，由于从普通文本文件重新开始读取，您刚刚设置的变量类型信息将丢失：

```
students
#> # A tibble: 6 × 5
#>   student_id full_name        favourite_food     meal_plan             age
#>        <dbl> <chr>            <chr>              <fct>               <dbl>
#> 1          1 Sunil Huffmann   Strawberry yoghurt Lunch only              4
#> 2          2 Barclay Lynn     French fries       Lunch only              5
#> 3          3 Jayendra Lyne    <NA>               Breakfast and lunch     7
#> 4          4 Leon Rossini     Anchovies          Lunch only             NA
#> 5          5 Chidiegwu Dunkel Pizza              Breakfast and lunch     5
#> 6          6 Güvenç Attila    Ice cream          Lunch only              6
write_csv(students, "students-2.csv")
read_csv("students-2.csv")
#> # A tibble: 6 × 5
#>   student_id full_name        favourite_food     meal_plan             age
#>        <dbl> <chr>            <chr>              <chr>               <dbl>
#> 1          1 Sunil Huffmann   Strawberry yoghurt Lunch only              4
#> 2          2 Barclay Lynn     French fries       Lunch only              5
#> 3          3 Jayendra Lyne    <NA>               Breakfast and lunch     7
#> 4          4 Leon Rossini     Anchovies          Lunch only             NA
#> 5          5 Chidiegwu Dunkel Pizza              Breakfast and lunch     5
#> 6          6 Güvenç Attila    Ice cream          Lunch only              6
```

这使得 CSV 在缓存中间结果时略显不可靠 — 每次加载时都需要重新创建列规范。有两种主要替代方法：

+   [`write_rds()`](https://readr.tidyverse.org/reference/read_rds.xhtml) 和 [`read_rds()`](https://readr.tidyverse.org/reference/read_rds.xhtml) 是围绕基础函数 [`readRDS()`](https://rdrr.io/r/base/readRDS.xhtml) 和 [`saveRDS()`](https://rdrr.io/r/base/readRDS.xhtml) 的统一包装器。这些函数使用 R 的自定义二进制格式 RDS 存储数据。这意味着当重新加载对象时，您加载的是*完全相同的* R 对象。

    ```
    write_rds(students, "students.rds")
    read_rds("students.rds")
    #> # A tibble: 6 × 5
    #>   student_id full_name        favourite_food     meal_plan             age
    #>        <dbl> <chr>            <chr>              <fct>               <dbl>
    #> 1          1 Sunil Huffmann   Strawberry yoghurt Lunch only              4
    #> 2          2 Barclay Lynn     French fries       Lunch only              5
    #> 3          3 Jayendra Lyne    <NA>               Breakfast and lunch     7
    #> 4          4 Leon Rossini     Anchovies          Lunch only             NA
    #> 5          5 Chidiegwu Dunkel Pizza              Breakfast and lunch     5
    #> 6          6 Güvenç Attila    Ice cream          Lunch only              6
    ```

+   arrow 包允许您读取和写入 parquet 文件，这是一种快速的二进制文件格式，可以在多种编程语言之间共享。我们将在 第二十二章 中更深入地讨论 arrow。

    ```
    library(arrow)
    write_parquet(students, "students.parquet")
    read_parquet("students.parquet")
    #> # A tibble: 6 × 5
    #>   student_id full_name        favourite_food     meal_plan             age
    #>        <dbl> <chr>            <chr>              <fct>               <dbl>
    #> 1          1 Sunil Huffmann   Strawberry yoghurt Lunch only              4
    #> 2          2 Barclay Lynn     French fries       Lunch only              5
    #> 3          3 Jayendra Lyne    NA                 Breakfast and lunch     7
    #> 4          4 Leon Rossini     Anchovies          Lunch only             NA
    #> 5          5 Chidiegwu Dunkel Pizza              Breakfast and lunch     5
    #> 6          6 Güvenç Attila    Ice cream          Lunch only              6
    ```

Parquet 比 RDS 更快，并且可在 R 之外使用，但需要 arrow 包。

# 数据输入

有时您需要在 R 脚本中手动组装 tibble 进行一些数据输入。有两个有用的函数可以帮助您完成此操作，这两个函数在按列或按行布局 tibble 方面有所不同。[`tibble()`](https://tibble.tidyverse.org/reference/tibble.xhtml) 按列工作：

```
tibble(
  x = c(1, 2, 5), 
  y = c("h", "m", "g"),
  z = c(0.08, 0.83, 0.60)
)
#> # A tibble: 3 × 3
#>       x y         z
#>   <dbl> <chr> <dbl>
#> 1     1 h      0.08
#> 2     2 m      0.83
#> 3     5 g      0.6
```

按列排列数据可能会使行之间的关系难以看清，因此另一种选择是[`tribble()`](https://tibble.tidyverse.org/reference/tribble.xhtml)，即*tr*ansposed t*ibble*，它允许你逐行布置数据。[`tribble()`](https://tibble.tidyverse.org/reference/tribble.xhtml)专为在代码中进行数据输入而定制：列标题以`~`开头，条目之间用逗号分隔。这使得可以以易于阅读的形式布置少量数据：

```
tribble(
  ~x, ~y, ~z,
  1, "h", 0.08,
  2, "m", 0.83,
  5, "g", 0.60
)
#> # A tibble: 3 × 3
#>   x         y     z
#>   <chr> <dbl> <dbl>
#> 1        1 h  0.08
#> 2        2 m  0.83
#> 3        5 g  0.6
```

# 摘要

在本章中，你学会了如何使用[`read_csv()`](https://readr.tidyverse.org/reference/read_delim.xhtml)加载 CSV 文件，并使用[`tibble()`](https://tibble.tidyverse.org/reference/tibble.xhtml)和[`tribble()`](https://tibble.tidyverse.org/reference/tribble.xhtml)进行自己的数据输入。你已经了解了 CSV 文件的工作原理，可能会遇到的一些问题以及如何克服它们。我们将在本书中多次涉及数据导入：第二十章将向你展示如何从 Excel 和 Google Sheets 加载数据，第二十一章从数据库中加载，第二十二章从 Parquet 文件中加载，第二十三章从 JSON 中加载，以及第二十四章从网站中加载。

我们几乎已经完成了本书章节的结尾，但还有一个重要的主题需要讨论：如何获取帮助。因此，在下一章中，你将学习一些寻求帮助的好方法，如何创建一个示范性代码以最大化获取良好帮助的机会，以及一些关于跟上 R 世界的一般建议。

¹ [janitor 包](https://oreil.ly/-J8GX)不属于 tidyverse 的一部分，但它提供了方便的数据清理功能，并且在使用`|>`的数据管道中运行良好。

² 你可以使用`guess_max`参数覆盖默认的 1,000。
