# 第二十一章：数据库

# 简介

大量数据存储在数据库中，因此你了解如何访问它是至关重要的。有时你可以请求某人为你下载一个 `.csv` 文件的快照，但这很快就变得痛苦：每次需要进行更改时，你都必须与另一个人沟通。你希望能够直接进入数据库，按需获取你所需的数据。

在本章中，你将首先学习 DBI 包的基础知识：如何使用它连接数据库，然后通过 SQL¹ 查询检索数据。*SQL*，即结构化查询语言，是数据库的通用语言，对所有数据科学家来说都是一种重要的语言。话虽如此，我们不会从 SQL 入手，而是教你 dbplyr，它可以将你的 dplyr 代码转换为 SQL。我们将以此方式教授你一些 SQL 最重要的特性。虽然在本章结束时你不会成为 SQL 大师，但你将能够识别最重要的组件并理解它们的作用。

## 先决条件

在本章中，我们将介绍 DBI 和 dbplyr。DBI 是一个低级接口，用于连接数据库并执行 SQL；dbplyr 是一个高级接口，将你的 dplyr 代码转换为 SQL 查询，然后使用 DBI 执行这些查询。

```
library(DBI)
library(dbplyr)
library(tidyverse)
```

# 数据库基础知识

在最简单的层面上，你可以把数据库看作是一个数据框的集合，在数据库术语中称为*表*。就像 `data.frame` 一样，数据库表是一组具有命名列的集合，其中每个列中的值都是相同类型的。数据框与数据库表之间有三个高级别的区别：

+   数据库表存储在磁盘上，可以任意大。数据框存储在内存中，并且基本上是有限的（尽管这个限制对于许多问题来说仍然很大）。

+   数据库表几乎总是有索引的。就像书的索引一样，数据库索引使得能够快速找到感兴趣的行，而无需查看每一行。数据框和 tibble 没有索引，但数据表有，这也是它们如此快速的原因之一。

+   大多数经典数据库优化于快速收集数据，而不是分析现有数据。这些数据库被称为*面向行*，因为数据是按行存储的，而不像 R 那样按列存储。近年来，出现了许多*面向列*数据库的发展，这使得分析现有数据变得更加快速。

数据库由数据库管理系统（*DBMS* 简称）管理，基本上有三种形式：

+   *客户端-服务器* 数据库管理系统运行在强大的中央服务器上，你可以从你的计算机（客户端）连接到它们。它们非常适合在组织中与多人共享数据。流行的客户端-服务器 DBMS 包括 PostgreSQL、MariaDB、SQL Server 和 Oracle。

+   *云* DBMS，如 Snowflake、Amazon 的 RedShift 和 Google 的 BigQuery，类似于客户端-服务器 DBMS，但运行在云端。这意味着它们可以轻松处理极大的数据集，并根据需要自动提供更多的计算资源。

+   *进程内* DBMS，如 SQLite 或 duckdb，在你的计算机上完全运行。它们非常适合处理大数据集，你是主要用户。

# 连接到数据库

要从 R 连接到数据库，你将使用一对包：

+   你总是会使用 DBI（数据库接口），因为它提供了一组通用函数，用于连接到数据库、上传数据、运行 SQL 查询等等。

+   你还会使用专门为你连接的 DBMS 定制的包。这个包将通用的 DBI 命令转换为特定于给定 DBMS 所需的命令。通常每个 DBMS 都有一个包，例如，RPostgres 用于 PostgreSQL，RMariaDB 用于 MySQL。

如果找不到特定 DBMS 的包，通常可以使用 odbc 包代替。这使用许多 DBMS 支持的 ODBC 协议。odbc 需要更多的设置，因为你还需要安装 ODBC 驱动程序并告诉 odbc 包在哪里找到它。

具体来说，你可以使用[`DBI::dbConnect()`](https://dbi.r-dbi.org/reference/dbConnect.xhtml)来创建数据库连接。第一个参数选择 DBMS，² 然后第二个及后续参数描述如何连接到它（即它的位置和需要访问它所需的凭据）。以下代码展示了几个典型的例子：

```
con <- DBI::dbConnect(
  RMariaDB::MariaDB(), 
  username = "foo"
)
con <- DBI::dbConnect(
  RPostgres::Postgres(), 
  hostname = "databases.mycompany.com", 
  port = 1234
)
```

从 DBMS 到 DBMS 连接的详细信息会有很大的不同，所以遗憾的是我们无法在这里涵盖所有的细节。这意味着你需要自己做一些研究。通常你可以向团队中的其他数据科学家询问，或者与你的数据库管理员（DBA）交流。最初的设置通常需要一点调试（也许需要一些搜索），但通常只需做一次。

## 在本书中

为了本书而设立客户端-服务器或云 DBMS 可能会很麻烦，因此我们将使用一个完全驻留在 R 包中的进程内 DBMS：duckdb。多亏了 DBI 的魔力，使用 duckdb 和任何其他 DBMS 之间唯一的区别在于如何连接到数据库。这使得它非常适合教学，因为你可以轻松运行这段代码，同时也可以轻松地将所学内容应用到其他地方。

连接到 duckdb 特别简单，因为默认情况下创建一个临时数据库，退出 R 时会删除。这对学习来说非常好，因为它保证每次重新启动 R 时都从一个干净的状态开始：

```
con <- DBI::dbConnect(duckdb::duckdb())
```

duckdb 是专为数据科学家需求设计的高性能数据库。我们在这里使用它是因为它很容易入门，但它也能以极快的速度处理千兆字节的数据。如果你想在真实的数据分析项目中使用 duckdb，你还需要提供 `dbdir` 参数来创建一个持久的数据库，并告诉 duckdb 在哪里保存它。假设你正在使用一个项目（第六章），把它存储在当前项目的 `duckdb` 目录中是合理的：

```
con <- DBI::dbConnect(duckdb::duckdb(), dbdir = "duckdb")
```

## 加载一些数据

因为这是一个新的数据库，我们需要首先添加一些数据。在这里，我们将使用 [`DBI::dbWriteTable()`](https://dbi.r-dbi.org/reference/dbWriteTable.xhtml) 添加 ggplot2 的 `mpg` 和 `diamonds` 数据集。[`dbWriteTable()`](https://dbi.r-dbi.org/reference/dbWriteTable.xhtml) 的最简单用法需要三个参数：一个数据库连接，要在数据库中创建的表的名称，以及一个数据框。

```
dbWriteTable(con, "mpg", ggplot2::mpg)
dbWriteTable(con, "diamonds", ggplot2::diamonds)
```

如果你在一个真实的项目中使用 duckdb，我们强烈推荐学习 `duckdb_read_csv()` 和 `duckdb_register_arrow()`。它们为你提供了强大和高效的方式，直接将数据快速加载到 duckdb 中，而无需先加载到 R 中。我们还将展示一种有用的技术，用于将多个文件写入数据库，详见 “Writing to a Database”。

## DBI 基础

你可以通过使用几个其他的 DBI 函数来检查数据是否加载正确：`dbListTable()` 列出数据库中的所有表，³ 和 [`dbReadTable()`](https://dbi.r-dbi.org/reference/dbReadTable.xhtml) 检索表的内容。

```
dbListTables(con)
#> [1] "diamonds" "mpg"

con |> 
  dbReadTable("diamonds") |> 
  as_tibble()
#> # A tibble: 53,940 × 10
#>   carat cut       color clarity depth table price     x     y     z
#>   <dbl> <fct>     <fct> <fct>   <dbl> <dbl> <int> <dbl> <dbl> <dbl>
#> 1  0.23 Ideal     E     SI2      61.5    55   326  3.95  3.98  2.43
#> 2  0.21 Premium   E     SI1      59.8    61   326  3.89  3.84  2.31
#> 3  0.23 Good      E     VS1      56.9    65   327  4.05  4.07  2.31
#> 4  0.29 Premium   I     VS2      62.4    58   334  4.2   4.23  2.63
#> 5  0.31 Good      J     SI2      63.3    58   335  4.34  4.35  2.75
#> 6  0.24 Very Good J     VVS2     62.8    57   336  3.94  3.96  2.48
#> # … with 53,934 more rows
```

[`dbReadTable()`](https://dbi.r-dbi.org/reference/dbReadTable.xhtml) 返回一个 `data.frame`，所以我们使用 [`as_tibble()`](https://tibble.tidyverse.org/reference/as_tibble.xhtml) 将其转换为 tibble，以便它可以漂亮地打印出来。

如果你已经了解 SQL，你可以使用 [`dbGetQuery()`](https://dbi.r-dbi.org/reference/dbGetQuery.xhtml) 来获取在数据库上运行查询的结果：

```
sql <- "
 SELECT carat, cut, clarity, color, price 
 FROM diamonds 
 WHERE price > 15000
"
as_tibble(dbGetQuery(con, sql))
#> # A tibble: 1,655 × 5
#>   carat cut       clarity color price
#>   <dbl> <fct>     <fct>   <fct> <int>
#> 1  1.54 Premium   VS2     E     15002
#> 2  1.19 Ideal     VVS1    F     15005
#> 3  2.1  Premium   SI1     I     15007
#> 4  1.69 Ideal     SI1     D     15011
#> 5  1.5  Very Good VVS2    G     15013
#> 6  1.73 Very Good VS1     G     15014
#> # … with 1,649 more rows
```

如果你以前没见过 SQL，别担心！你很快就会了解更多。但是如果你仔细阅读，你可能会猜到它选择了 `diamonds` 数据集的五列，并选择了所有 `price` 大于 15,000 的行。

# dbplyr 基础

现在我们已经连接到数据库并加载了一些数据，我们可以开始学习 dbplyr。dbplyr 是 dplyr 的一个 *后端*，这意味着你可以继续编写 dplyr 代码，但后端会以不同的方式执行它。在这里，dbplyr 转换为 SQL；其他后端包括 [dtplyr](https://oreil.ly/9Dq5p)，它转换为 [data.table](https://oreil.ly/k3EaP)，以及 [multidplyr](https://oreil.ly/gmDpk)，它可以在多个核心上执行你的代码。

要使用 dbplyr，你必须先使用 [`tbl()`](https://dplyr.tidyverse.org/reference/tbl.xhtml) 来创建一个表示数据库表的对象：

```
diamonds_db <- tbl(con, "diamonds")
diamonds_db
#> # Source:   table<diamonds> [?? x 10]
#> # Database: DuckDB 0.6.1 [root@Darwin 22.3.0:R 4.2.1/:memory:]
#>   carat cut       color clarity depth table price     x     y     z
#>   <dbl> <fct>     <fct> <fct>   <dbl> <dbl> <int> <dbl> <dbl> <dbl>
#> 1  0.23 Ideal     E     SI2      61.5    55   326  3.95  3.98  2.43
#> 2  0.21 Premium   E     SI1      59.8    61   326  3.89  3.84  2.31
#> 3  0.23 Good      E     VS1      56.9    65   327  4.05  4.07  2.31
#> 4  0.29 Premium   I     VS2      62.4    58   334  4.2   4.23  2.63
#> 5  0.31 Good      J     SI2      63.3    58   335  4.34  4.35  2.75
#> 6  0.24 Very Good J     VVS2     62.8    57   336  3.94  3.96  2.48
#> # … with more rows
```

###### 注意

与数据库互动的另外两种常见方法。首先，许多企业数据库非常庞大，因此您需要一些层次结构来组织所有的表。在这种情况下，您可能需要提供一个模式或者一个目录和模式，以选择您感兴趣的表：

```
diamonds_db <- tbl(con, in_schema("sales", "diamonds"))
diamonds_db <- tbl(
  con, in_catalog("north_america", "sales", "diamonds")
  )
```

有时您可能希望使用自己的 SQL 查询作为起点：

```
diamonds_db <- tbl(con, sql("SELECT * FROM diamonds"))
```

这个对象是*惰性*的；当你在其上使用 dplyr 动词时，dplyr 不会进行任何操作：它只记录你希望执行的操作序列，并在需要时执行它们。例如，考虑以下管道：

```
big_diamonds_db <- diamonds_db |> 
  filter(price > 15000) |> 
  select(carat:clarity, price)

big_diamonds_db
#> # Source:   SQL [?? x 5]
#> # Database: DuckDB 0.6.1 [root@Darwin 22.3.0:R 4.2.1/:memory:]
#>   carat cut       color clarity price
#>   <dbl> <fct>     <fct> <fct>   <int>
#> 1  1.54 Premium   E     VS2     15002
#> 2  1.19 Ideal     F     VVS1    15005
#> 3  2.1  Premium   I     SI1     15007
#> 4  1.69 Ideal     D     SI1     15011
#> 5  1.5  Very Good G     VVS2    15013
#> 6  1.73 Very Good G     VS1     15014
#> # … with more rows
```

您可以通过打印顶部的 DBMS 名称来确定这个对象表示一个数据库查询，尽管它告诉您列的数量，但通常不知道行的数量。这是因为找到总行数通常需要执行完整的查询，我们正试图避免这样做。

您可以使用 [`show_query()`](https://dplyr.tidyverse.org/reference/explain.xhtml) 函数查看 dplyr 生成的 SQL 代码。如果你了解 dplyr，这是学习 SQL 的好方法！编写一些 dplyr 代码，让 dbplyr 将其转换为 SQL，然后尝试弄清这两种语言如何匹配。

```
big_diamonds_db |>
  show_query()
#> <SQL>
#> SELECT carat, cut, color, clarity, price
#> FROM diamonds
#> WHERE (price > 15000.0)
```

要将所有数据带回 R，您可以调用[`collect()`](https://dplyr.tidyverse.org/reference/compute.xhtml)。在幕后，这会生成 SQL 代码，调用 [`dbGetQuery()`](https://dbi.r-dbi.org/reference/dbGetQuery.xhtml) 获取数据，然后将结果转换为 tibble：

```
big_diamonds <- big_diamonds_db |> 
  collect()
big_diamonds
#> # A tibble: 1,655 × 5
#>   carat cut       color clarity price
#>   <dbl> <fct>     <fct> <fct>   <int>
#> 1  1.54 Premium   E     VS2     15002
#> 2  1.19 Ideal     F     VVS1    15005
#> 3  2.1  Premium   I     SI1     15007
#> 4  1.69 Ideal     D     SI1     15011
#> 5  1.5  Very Good G     VVS2    15013
#> 6  1.73 Very Good G     VS1     15014
#> # … with 1,649 more rows
```

通常情况下，您将使用 dbplyr 从数据库中选择所需的数据，使用下面描述的翻译执行基本的过滤和聚合。然后，一旦准备用 R 特有的函数分析数据，您可以使用 [`collect()`](https://dplyr.tidyverse.org/reference/compute.xhtml) 收集数据到内存中的 tibble，并继续您的工作与纯 R 代码。

# SQL

本章的其余部分将通过 dbplyr 的视角向你介绍一点 SQL。这是一个非常非传统的 SQL 入门，但我们希望它能让你迅速掌握基础知识。幸运的是，如果你了解 dplyr，那你很容易掌握 SQL，因为许多概念是相同的。

我们将使用 nycflights13 包中的两个老朋友 `flights` 和 `planes` 来探索 dplyr 和 SQL 之间的关系。这些数据集很容易导入我们的学习数据库，因为 dbplyr 自带一个将 nycflights13 中的表复制到我们数据库中的函数：

```
dbplyr::copy_nycflights13(con)
#> Creating table: airlines
#> Creating table: airports
#> Creating table: flights
#> Creating table: planes
#> Creating table: weather
flights <- tbl(con, "flights")
planes <- tbl(con, "planes")
```

## SQL 基础知识

SQL 的顶层组件被称为*语句*。常见的语句包括用于定义新表的`CREATE`，用于添加数据的`INSERT`，以及用于检索数据的`SELECT`。我们将重点关注`SELECT`语句，也称为*查询*，因为这几乎是数据科学家经常使用的。

查询由 *子句* 组成。有五个重要的子句：`SELECT`、`FROM`、`WHERE`、`ORDER BY` 和 `GROUP BY`。每个查询必须包含 `SELECT`⁴ 和 `FROM`⁵ 子句，最简单的查询是 `SELECT * FROM table`，它从指定的表中选择所有列。这是未经修改的表时 `dbplyr` 生成的内容：

```
flights |> show_query()
#> <SQL>
#> SELECT *
#> FROM flights
planes |> show_query()
#> <SQL>
#> SELECT *
#> FROM planes
```

`WHERE` 和 `ORDER BY` 控制包含哪些行以及如何排序：

```
flights |> 
  filter(dest == "IAH") |> 
  arrange(dep_delay) |>
  show_query()
#> <SQL>
#> SELECT *
#> FROM flights
#> WHERE (dest = 'IAH')
#> ORDER BY dep_delay
```

`GROUP BY` 将查询转换为摘要，导致聚合操作发生：

```
flights |> 
  group_by(dest) |> 
  summarize(dep_delay = mean(dep_delay, na.rm = TRUE)) |> 
  show_query()
#> <SQL>
#> SELECT dest, AVG(dep_delay) AS dep_delay
#> FROM flights
#> GROUP BY dest
```

在 `dplyr` 动词和 `SELECT` 子句之间存在两个重要的区别：

+   在 SQL 中，大小写不重要：可以写成 `select`、`SELECT`，甚至 `SeLeCt`。本书中我们将坚持使用大写字母来写 SQL 关键字，以区分它们与表或变量名。

+   在 SQL 中，顺序很重要：必须按照 `SELECT`、`FROM`、`WHERE`、`GROUP BY` 和 `ORDER BY` 的顺序书写子句。令人困惑的是，这个顺序并不匹配子句实际执行的顺序，实际上是先执行 `FROM`，然后是 `WHERE`、`GROUP BY`、`SELECT` 和 `ORDER BY`。

下面的部分详细探讨了每个子句。

###### 注意

请注意，尽管 SQL 是一个标准，但它非常复杂，没有数据库完全遵循该标准。尽管我们在本书中将关注的主要组件在不同 DBMS 之间相似，但存在许多细小的变化。幸运的是，`dbplyr` 被设计来处理这个问题，并为不同的数据库生成不同的翻译。它并非完美，但在不断改进，如果遇到问题，您可以在 [GitHub](https://oreil.ly/xgmg8) 上提出问题以帮助我们做得更好。

## SELECT

`SELECT` 子句是查询的核心，执行与 [`select()`](https://dplyr.tidyverse.org/reference/select.xhtml)，[`mutate()`](https://dplyr.tidyverse.org/reference/mutate.xhtml)，[`rename()`](https://dplyr.tidyverse.org/reference/rename.xhtml)，[`relocate()`](https://dplyr.tidyverse.org/reference/relocate.xhtml) 和你将在下一节中学到的 [`summarize()`](https://dplyr.tidyverse.org/reference/summarise.xhtml) 相同的工作。

[`select()`](https://dplyr.tidyverse.org/reference/select.xhtml)，[`rename()`](https://dplyr.tidyverse.org/reference/rename.xhtml)，和 [`relocate()`](https://dplyr.tidyverse.org/reference/relocate.xhtml) 都有非常直接的对应 `SELECT` 的翻译，因为它们只影响列出现的位置（如果有的话）以及列的名称：

```
planes |> 
  select(tailnum, type, manufacturer, model, year) |> 
  show_query()
#> <SQL>
#> SELECT tailnum, "type", manufacturer, model, "year"
#> FROM planes

planes |> 
  select(tailnum, type, manufacturer, model, year) |> 
  rename(year_built = year) |> 
  show_query()
#> <SQL>
#> SELECT tailnum, "type", manufacturer, model, "year" AS year_built
#> FROM planes

planes |> 
  select(tailnum, type, manufacturer, model, year) |> 
  relocate(manufacturer, model, .before = type) |> 
  show_query()
#> <SQL>
#> SELECT tailnum, manufacturer, model, "type", "year"
#> FROM planes
```

该示例还展示了 SQL 如何进行重命名。在 SQL 术语中，重命名称为 *别名*，并使用 `AS` 进行操作。请注意，与 [`mutate()`](https://dplyr.tidyverse.org/reference/mutate.xhtml) 不同的是，旧名称在左侧，新名称在右侧。

###### 注意

在前面的例子中，请注意`"year"`和`"type"`被包裹在双引号中。这是因为它们在 duckdb 中是*保留词*，所以 dbplyr 将它们用引号括起来，以避免在列/表名和 SQL 操作符之间产生任何潜在混淆。

当与其他数据库一起工作时，你可能会看到每个变量名都被引用，因为只有少数客户端包（如 duckdb）知道所有的保留词，所以它们为了安全起见将所有内容都加了引号：

```
SELECT "tailnum", "type", "manufacturer", "model", "year"
FROM "planes"
```

其他一些数据库系统使用反引号而不是引号：

```
SELECT `tailnum`, `type`, `manufacturer`, `model`, `year`
FROM `planes`
```

对于[`mutate()`](https://dplyr.tidyverse.org/reference/mutate.xhtml)的翻译同样很简单：每个变量在`SELECT`中成为一个新的表达式：

```
flights |> 
  mutate(
    speed = distance / (air_time / 60)
  ) |> 
  show_query()
#> <SQL>
#> SELECT *, distance / (air_time / 60.0) AS speed
#> FROM flights
```

我们将在“函数翻译”中回顾个别组件（如`/`）的翻译。

## FROM

`FROM`子句定义了数据源。一开始它可能不是很有趣，因为我们只是使用单表。一旦涉及到连接函数，你会看到更复杂的例子。

## GROUP BY

[`group_by()`](https://dplyr.tidyverse.org/reference/group_by.xhtml)被翻译为`GROUP BY`⁶子句，而[`summarize()`](https://dplyr.tidyverse.org/reference/summarise.xhtml)则被翻译为`SELECT`子句：

```
diamonds_db |> 
  group_by(cut) |> 
  summarize(
    n = n(),
    avg_price = mean(price, na.rm = TRUE)
  ) |> 
  show_query()
#> <SQL>
#> SELECT cut, COUNT(*) AS n, AVG(price) AS avg_price
#> FROM diamonds
#> GROUP BY cut
```

我们将在“函数翻译”中回顾[`n()`](https://dplyr.tidyverse.org/reference/context.xhtml)和[`mean()`](https://rdrr.io/r/base/mean.xhtml)的翻译过程。

## WHERE

[`filter()`](https://dplyr.tidyverse.org/reference/filter.xhtml)被翻译为`WHERE`子句：

```
flights |> 
  filter(dest == "IAH" | dest == "HOU") |> 
  show_query()
#> <SQL>
#> SELECT *
#> FROM flights
#> WHERE (dest = 'IAH' OR dest = 'HOU')

flights |> 
  filter(arr_delay > 0 & arr_delay < 20) |> 
  show_query()
#> <SQL>
#> SELECT *
#> FROM flights
#> WHERE (arr_delay > 0.0 AND arr_delay < 20.0)
```

在这里需要注意几个重要的细节：

+   `|`变成`OR`，`&`变成`AND`。

+   SQL 使用`=`进行比较，而不是`==`。SQL 中没有赋值操作，所以在这方面没有混淆的可能性。

+   SQL 中只使用`''`表示字符串，而不是`""`。在 SQL 中，`""`用于标识变量，就像 R 的``` `` ```一样。

另一个有用的 SQL 操作符是`IN`，它与 R 的`%in%`相似：

```
flights |> 
  filter(dest %in% c("IAH", "HOU")) |> 
  show_query()
#> <SQL>
#> SELECT *
#> FROM flights
#> WHERE (dest IN ('IAH', 'HOU'))
```

SQL 使用`NULL`而不是`NA`。`NULL`在比较和算术运算中的行为类似于`NA`。主要的区别在于，当进行汇总时，它们会被静默地丢弃。当你第一次遇到时，dbplyr 会提醒你这种行为：

```
flights |> 
  group_by(dest) |> 
  summarize(delay = mean(arr_delay))
#> Warning: Missing values are always removed in SQL aggregation functions.
#> Use `na.rm = TRUE` to silence this warning
#> This warning is displayed once every 8 hours.
#> # Source:   SQL [?? x 2]
#> # Database: DuckDB 0.6.1 [root@Darwin 22.3.0:R 4.2.1/:memory:]
#>   dest   delay
#>   <chr>  <dbl>
#> 1 ATL   11.3 
#> 2 ORD    5.88 
#> 3 RDU   10.1 
#> 4 IAD   13.9 
#> 5 DTW    5.43 
#> 6 LAX    0.547
#> # … with more rows
```

如果你想了解更多关于`NULL`如何工作的信息，你可能会喜欢 Markus Winand 的[“SQL 的三值逻辑”](https://oreil.ly/PTwQz)。

总的来说，你可以使用在 R 中处理`NA`时使用的函数来处理`NULL`：

```
flights |> 
  filter(!is.na(dep_delay)) |> 
  show_query()
#> <SQL>
#> SELECT *
#> FROM flights
#> WHERE (NOT((dep_delay IS NULL)))
```

这个 SQL 查询展示了 dbplyr 的一个缺点：虽然 SQL 是正确的，但并不像你手写的那样简单。在这种情况下，你可以去掉括号，并使用一个更容易阅读的特殊操作符：

```
WHERE "dep_delay" IS NOT NULL
```

注意，如果你用`summarize`创建的变量进行[`filter()`](https://dplyr.tidyverse.org/reference/filter.xhtml)，`dbplyr`会生成`HAVING`子句，而不是`WHERE`子句。这是 SQL 的一个特殊之处：`WHERE`在`SELECT`和`GROUP BY`之前评估，所以 SQL 需要另一个在其后评估的子句。

```
diamonds_db |> 
  group_by(cut) |> 
  summarize(n = n()) |> 
  filter(n > 100) |> 
  show_query()
#> <SQL>
#> SELECT cut, COUNT(*) AS n
#> FROM diamonds
#> GROUP BY cut
#> HAVING (COUNT(*) > 100.0)
```

## ORDER BY

对行进行排序涉及从[`arrange()`](https://dplyr.tidyverse.org/reference/arrange.xhtml)到`ORDER BY`子句的直接翻译：

```
flights |> 
  arrange(year, month, day, desc(dep_delay)) |> 
  show_query()
#> <SQL>
#> SELECT *
#> FROM flights
#> ORDER BY "year", "month", "day", dep_delay DESC
```

注意[`desc()`](https://dplyr.tidyverse.org/reference/desc.xhtml)如何翻译成`DESC`：这是许多`dplyr`函数之一，其命名直接受到 SQL 启发。

## 子查询

有时候无法将一个`dplyr`管道翻译成单个`SELECT`语句，需要使用子查询。*子查询*就是作为`FROM`子句中数据源的查询，而不是通常的表格。

`dbplyr`通常使用子查询来解决 SQL 的限制。例如，`SELECT`子句中的表达式不能引用刚刚创建的列。这意味着以下（愚蠢的）`dplyr`管道需要分两步进行：第一个（内部）查询计算`year1`，然后第二个（外部）查询可以计算`year2`：

```
flights |> 
  mutate(
    year1 = year + 1,
    year2 = year1 + 1
  ) |> 
  show_query()
#> <SQL>
#> SELECT *, year1 + 1.0 AS year2
#> FROM (
#>   SELECT *, "year" + 1.0 AS year1
#>   FROM flights
#> ) q01
```

如果你尝试对刚创建的变量进行[`filter()`](https://dplyr.tidyverse.org/reference/filter.xhtml)，也会看到这种情况。记住，尽管`WHERE`在`SELECT`之后编写，但它在之前评估，因此在这个（愚蠢的）例子中我们需要一个子查询：

```
flights |> 
  mutate(year1 = year + 1) |> 
  filter(year1 == 2014) |> 
  show_query()
#> <SQL>
#> SELECT *
#> FROM (
#>   SELECT *, "year" + 1.0 AS year1
#>   FROM flights
#> ) q01
#> WHERE (year1 = 2014.0)
```

有时候`dbplyr`会创建不必要的子查询，因为它尚不知道如何优化这种转换。随着时间推移，`dbplyr`的改进会使这类情况变得更少，但可能永远不会消失。

## 连接

如果你熟悉`dplyr`的连接操作，SQL 的连接操作也是类似的。这里有一个简单的例子：

```
flights |> 
  left_join(planes |> rename(year_built = year), by = "tailnum") |> 
  show_query()
#> <SQL>
#> SELECT
#>   flights.*,
#>   planes."year" AS year_built,
#>   "type",
#>   manufacturer,
#>   model,
#>   engines,
#>   seats,
#>   speed,
#>   engine
#> FROM flights
#> LEFT JOIN planes
#>   ON (flights.tailnum = planes.tailnum)
```

这里需要注意的主要是语法：SQL 连接使用`FROM`子句的子子句引入额外的表格，并使用`ON`来定义表格之间的关系。

`dplyr`中这些函数的命名与 SQL 密切相关，因此你可以轻松推测[`inner_join()`](https://dplyr.tidyverse.org/reference/mutate-joins.xhtml)、[`right_join()`](https://dplyr.tidyverse.org/reference/mutate-joins.xhtml)和[`full_join()`](https://dplyr.tidyverse.org/reference/mutate-joins.xhtml)的等效 SQL 语句：

```
SELECT flights.*, "type", manufacturer, model, engines, seats, speed
FROM flights
INNER JOIN planes ON (flights.tailnum = planes.tailnum)

SELECT flights.*, "type", manufacturer, model, engines, seats, speed
FROM flights
RIGHT JOIN planes ON (flights.tailnum = planes.tailnum)

SELECT flights.*, "type", manufacturer, model, engines, seats, speed
FROM flights
FULL JOIN planes ON (flights.tailnum = planes.tailnum)
```

当从数据库中处理数据时，你可能需要许多连接。这是因为数据库表通常以高度规范化的形式存储，每个“事实”都存储在一个地方，为了保持完整的数据集进行分析，你需要浏览通过主键和外键连接的复杂网络表。如果遇到这种情况，Tobias Schieferdecker、Kirill Müller 和 Darko Bergant 的[dm 包](https://oreil.ly/tVS8h)会帮上大忙。它可以自动确定表之间的连接，使用 DBA 通常提供的约束条件可视化连接情况，生成你需要连接一个表到另一个表的连接。

## 其他动词

dbplyr 还翻译其他动词，如[`distinct()`](https://dplyr.tidyverse.org/reference/distinct.xhtml)，`slice_*()`，以及[`intersect()`](https://generics.r-lib.org/reference/setops.xhtml)，还有越来越多的 tidyr 函数，如[`pivot_longer()`](https://tidyr.tidyverse.org/reference/pivot_longer.xhtml)和[`pivot_wider()`](https://tidyr.tidyverse.org/reference/pivot_wider.xhtml)。查看目前提供的完整集合最简单的方法是访问[dbplyr 网站](https://oreil.ly/A8OGW)。

## 练习

1.  [`distinct()`](https://dplyr.tidyverse.org/reference/distinct.xhtml)被翻译成什么？[`head()`](https://rdrr.io/r/utils/head.xhtml)呢？

1.  解释下面每个 SQL 查询的作用，并尝试使用 dbplyr 重新创建它们：

    ```
    SELECT * 
    FROM flights
    WHERE dep_delay < arr_delay

    SELECT *, distance / (airtime / 60) AS speed
    FROM flights
    ```

# 函数翻译

到目前为止，我们已经集中讨论了 dplyr 动词如何翻译成查询语句的主要内容。现在我们将稍微深入一点，讨论与单个列一起工作的 R 函数的翻译；例如，在[`summarize()`](https://dplyr.tidyverse.org/reference/summarise.xhtml)中使用`mean(x)`时会发生什么？

为了帮助理解正在发生的事情，我们将使用几个小助手函数来运行[`summarize()`](https://dplyr.tidyverse.org/reference/summarise.xhtml)或[`mutate()`](https://dplyr.tidyverse.org/reference/mutate.xhtml)，并显示生成的 SQL。这将使探索几种变化，以及摘要和转换如何不同，变得更加容易。

```
summarize_query <- function(df, ...) {
  df |> 
    summarize(...) |> 
    show_query()
}
mutate_query <- function(df, ...) {
  df |> 
    mutate(..., .keep = "none") |> 
    show_query()
}
```

让我们通过一些摘要来深入研究！看看以下代码，你会注意到一些摘要函数，比如[`mean()`](https://rdrr.io/r/base/mean.xhtml)，其翻译相对简单，而像[`median()`](https://rdrr.io/r/stats/median.xhtml)这样的函数则复杂得多。这种复杂性通常更高，适用于统计学中常见但在数据库中不太常见的操作。

```
flights |> 
  group_by(year, month, day) |>  
  summarize_query(
    mean = mean(arr_delay, na.rm = TRUE),
    median = median(arr_delay, na.rm = TRUE)
  )
#> `summarise()` has grouped output by "year" and "month". You can override
#> using the `.groups` argument.
#> <SQL>
#> SELECT
#>   "year",
#>   "month",
#>   "day",
#>   AVG(arr_delay) AS mean,
#>   PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY arr_delay) AS median
#> FROM flights
#> GROUP BY "year", "month", "day"
```

当你在[`mutate()`](https://dplyr.tidyverse.org/reference/mutate.xhtml)中使用它们时，摘要函数的翻译会变得更加复杂，因为它们必须转变为所谓的*窗口*函数。在 SQL 中，你可以通过在普通聚合函数后添加`OVER`来将普通聚合函数转变为窗口函数：

```
flights |> 
  group_by(year, month, day) |>  
  mutate_query(
    mean = mean(arr_delay, na.rm = TRUE),
  )
#> <SQL>
#> SELECT
#>   "year",
#>   "month",
#>   "day",
#>   AVG(arr_delay) OVER (PARTITION BY "year", "month", "day") AS mean
#> FROM flights
```

在 SQL 中，`GROUP BY`子句专门用于汇总，因此您可以看到分组已从`PARTITION BY`参数移至`OVER`。

窗口函数包括所有向前或向后查找的函数，例如[`lead()`](https://dplyr.tidyverse.org/reference/lead-lag.xhtml)和[`lag()`](https://dplyr.tidyverse.org/reference/lead-lag.xhtml)，它们分别查看“前一个”或“后一个”值：

```
flights |> 
  group_by(dest) |>  
  arrange(time_hour) |> 
  mutate_query(
    lead = lead(arr_delay),
    lag = lag(arr_delay)
  )
#> <SQL>
#> SELECT
#>   dest,
#>   LEAD(arr_delay, 1, NULL) OVER (PARTITION BY dest ORDER BY time_hour) AS lead,
#>   LAG(arr_delay, 1, NULL) OVER (PARTITION BY dest ORDER BY time_hour) AS lag
#> FROM flights
#> ORDER BY time_hour
```

在这里，重要的是对数据进行[`arrange()`](https://dplyr.tidyverse.org/reference/arrange.xhtml)，因为 SQL 表没有固有的顺序。实际上，如果不使用[`arrange()`](https://dplyr.tidyverse.org/reference/arrange.xhtml)，每次都可能以不同的顺序返回行！请注意，对于窗口函数，排序信息是重复的：主查询的`ORDER BY`子句不会自动应用于窗口函数。

另一个重要的 SQL 函数是`CASE WHEN`。它被用作[`if_else()`](https://dplyr.tidyverse.org/reference/if_else.xhtml)和[`case_when()`](https://dplyr.tidyverse.org/reference/case_when.xhtml)的翻译，这两个是直接受其启发的 dplyr 函数。这里有几个简单的示例：

```
flights |> 
  mutate_query(
    description = if_else(arr_delay > 0, "delayed", "on-time")
  )
#> <SQL>
#> SELECT CASE WHEN 
#>   (arr_delay > 0.0) THEN 'delayed' 
#>   WHEN NOT (arr_delay > 0.0) THEN 'on-time' END AS description
#> FROM flights
flights |> 
  mutate_query(
    description = 
      case_when(
        arr_delay < -5 ~ "early", 
        arr_delay < 5 ~ "on-time",
        arr_delay >= 5 ~ "late"
      )
  )
#> <SQL>
#> SELECT CASE
#> WHEN (arr_delay < -5.0) THEN 'early'
#> WHEN (arr_delay < 5.0) THEN 'on-time'
#> WHEN (arr_delay >= 5.0) THEN 'late'
#> END AS description
#> FROM flights
```

`CASE WHEN`也用于一些从 R 到 SQL 没有直接翻译的其他函数。其中一个很好的例子是[`cut()`](https://rdrr.io/r/base/cut.xhtml)。

```
flights |> 
  mutate_query(
    description =  cut(
      arr_delay, 
      breaks = c(-Inf, -5, 5, Inf), 
      labels = c("early", "on-time", "late")
    )
  )
#> <SQL>
#> SELECT CASE
#> WHEN (arr_delay <= -5.0) THEN 'early'
#> WHEN (arr_delay <= 5.0) THEN 'on-time'
#> WHEN (arr_delay > 5.0) THEN 'late'
#> END AS description
#> FROM flights
```

dbplyr 还可以翻译常见的字符串和日期时间操作函数，您可以在[`vignette("translation-function", package = "dbplyr")`](https://dbplyr.tidyverse.org/articles/translation-function.xhtml)中了解这些。dbplyr 的翻译虽然不完美，但对于您大部分时间使用的函数来说，效果出奇的好。

# 摘要

在本章中，您学习了如何从数据库访问数据。我们专注于 dbplyr，这是一个 dplyr 的“后端”，允许您编写熟悉的 dplyr 代码，并自动将其转换为 SQL。我们利用这种转换教您了一些 SQL；学习一些 SQL 很重要，因为它是处理数据最常用的语言，掌握一些 SQL 会使您更容易与其他不使用 R 的数据专业人员交流。如果您完成了本章并希望了解更多关于 SQL 的知识，我们有两个推荐：

+   [*数据科学家的 SQL*](https://oreil.ly/QfAat)由 Renée M. P. Teate 编写，专为数据科学家的需求设计的 SQL 介绍，并包括您在真实组织中可能遇到的高度互联数据的示例。

+   [*实用 SQL*](https://oreil.ly/-0Usp)由安东尼·德巴罗斯（Anthony DeBarros）从数据记者的视角（一位专门讲述引人入胜故事的数据科学家）撰写，详细介绍了如何将数据导入数据库并运行自己的数据库管理系统（DBMS）。

在下一章中，我们将学习另一个用于处理大数据的 dplyr 后端：arrow。arrow 包设计用于处理磁盘上的大文件，是数据库的自然补充。

¹ SQL 可以发音为 “s”-“q”-“l” 或 “sequel”。

² 通常，这是你从客户端包中使用的唯一函数，因此我们建议使用 `::` 来提取那个函数，而不是用 [`library()`](https://rdrr.io/r/base/library.xhtml) 加载整个包。

³ 至少是你有权限查看的所有表。

⁴ 令人困惑的是，根据上下文，`SELECT` 可能是语句，也可能是子句。为避免混淆，我们通常使用 `SELECT` 查询而不是 `SELECT` 语句。

⁵ 严格来说，只需要 `SELECT`，因为你可以编写像 `SELECT 1+1` 这样的查询来进行基本计算。但如果你想处理数据（正如你总是需要的！），你还需要 `FROM` 子句。

⁶ 这并非巧合：dplyr 函数名受到了 SQL 语句的启发。
