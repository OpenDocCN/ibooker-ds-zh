# 第二十二章：Arrow

# 介绍

CSV 文件设计成易于人类阅读。它们是一种很好的交换格式，因为它们简单且可以被各种工具读取。但 CSV 文件不够高效：您需要做相当多的工作才能将数据读入 R。在本章中，您将了解到一个强大的替代方案：[parquet 格式](https://oreil.ly/ClE7D)，这是一个基于开放标准的格式，在大数据系统中被广泛使用。

我们将 parquet 文件与 [Apache Arrow](https://oreil.ly/TGrH5) 配对使用，后者是一款专为高效分析和传输大数据集而设计的多语言工具包。我们将通过 [arrow 软件包](https://oreil.ly/g60F8) 使用 Apache Arrow，它提供了 dplyr 后端，允许您使用熟悉的 dplyr 语法分析超内存数据集。此外，arrow 运行速度极快；本章后面将展示一些示例。

arrow 和 dbplyr 都提供了 dplyr 后端，因此您可能会想知道何时使用每一个。在许多情况下，选择已经为您做出，例如数据已经在数据库中或者在 parquet 文件中，并且您希望按原样处理它。但如果您从自己的数据开始（也许是 CSV 文件），您可以将其加载到数据库中或将其转换为 parquet。总之，在早期分析阶段，很难知道哪种方法最有效，因此我们建议您尝试两种方法，并选择最适合您的那一种。

（特别感谢 Danielle Navarro 提供本章的初始版本。）

## 先决条件

在本章中，我们将继续使用 tidyverse，特别是 dplyr，但我们将与 arrow 软件包配对使用，该软件包专门设计用于处理大数据：

```
library(tidyverse)
library(arrow)
```

本章稍后我们还将看到 arrow 和 duckdb 之间的一些连接，因此我们还需要 dbplyr 和 duckdb。

```
library(dbplyr, warn.conflicts = FALSE)
library(duckdb)
#> Loading required package: DBI
```

# 获取数据

我们首先获取一个值得使用这些工具的数据集：来自西雅图公共图书馆的项目借阅数据集，可在线获取，网址为 [Seattle Open Data](https://oreil.ly/u56DR)。该数据集包含 41,389,465 行数据，告诉您每本书从 2005 年 4 月至 2022 年 10 月每月被借阅的次数。

以下代码将帮助您获取数据的缓存副本。数据是一个 9 GB 的 CSV 文件，因此下载可能需要一些时间。我强烈建议您使用 `curl::multidownload()` 来获取非常大的文件，因为它专为此目的而设计：它会提供进度条，并且如果下载中断，它可以恢复下载。

```
dir.create("data", showWarnings = FALSE)

curl::multi_download(
  "https://r4ds.s3.us-west-2.amazonaws.com/seattle-library-checkouts.csv",
  "data/seattle-library-checkouts.csv",
  resume = TRUE
)
```

# 打开数据集

让我们先来看一下数据。9 GB 的文件足够大，我们可能不希望将整个文件加载到内存中。一个好的经验法则是，您通常希望内存大小至少是数据大小的两倍，而许多笔记本电脑最大只能支持 16 GB。这意味着我们应避免使用 [`read_csv()`](https://readr.tidyverse.org/reference/read_delim.xhtml)，而应使用 [`arrow::open_dataset()`](https://arrow.apache.org/docs/r/reference/open_dataset.xhtml)：

```
seattle_csv <- open_dataset(
  sources = "data/seattle-library-checkouts.csv", 
  format = "csv"
)
```

当运行此代码时会发生什么？[`open_dataset()`](https://arrow.apache.org/docs/r/reference/open_dataset.xhtml)将扫描几千行以确定数据集的结构。然后记录它所找到的内容并停止；只有在您明确请求时才会继续读取更多行。这些元数据是我们在打印`seattle_csv`时看到的内容：

```
seattle_csv
#> FileSystemDataset with 1 csv file
#> UsageClass: string
#> CheckoutType: string
#> MaterialType: string
#> CheckoutYear: int64
#> CheckoutMonth: int64
#> Checkouts: int64
#> Title: string
#> ISBN: null
#> Creator: string
#> Subjects: string
#> Publisher: string
#> PublicationYear: string
```

输出的第一行告诉您`seattle_csv`作为单个 CSV 文件存储在本地磁盘上，只有在需要时才会加载到内存中。其余输出告诉您 Arrow 为每列推断的列类型。

我们可以通过[`glimpse()`](https://pillar.r-lib.org/reference/glimpse.xhtml)看到实际情况。这显示了大约 4100 万行和 12 列，并展示了一些值。

```
seattle_csv |> glimpse()
#> FileSystemDataset with 1 csv file
#> 41,389,465 rows x 12 columns
#> $ UsageClass      <string> "Physical", "Physical", "Digital", "Physical", "Ph…
#> $ CheckoutType    <string> "Horizon", "Horizon", "OverDrive", "Horizon", "Hor…
#> $ MaterialType    <string> "BOOK", "BOOK", "EBOOK", "BOOK", "SOUNDDISC", "BOO…
#> $ CheckoutYear     <int64> 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 20…
#> $ CheckoutMonth    <int64> 6, 6, 6, 6, 6, 6, 6, 6, 6, 6, 6, 6, 6, 6, 6, 6, 6,…
#> $ Checkouts        <int64> 1, 1, 1, 1, 1, 1, 1, 1, 4, 1, 1, 2, 3, 2, 1, 3, 2,…
#> $ Title           <string> "Super rich : a guide to having it all / Russell S…
#> $ ISBN            <string> "", "", "", "", "", "", "", "", "", "", "", "", ""…
#> $ Creator         <string> "Simmons, Russell", "Barclay, James, 1965-", "Tim …
#> $ Subjects        <string> "Self realization, Conduct of life, Attitude Psych…
#> $ Publisher       <string> "Gotham Books,", "Pyr,", "Random House, Inc.", "Di…
#> $ PublicationYear <string> "c2011.", "2010.", "2015", "2005.", "c2004.", "c20…
```

我们可以开始使用 dplyr 动词处理此数据集，使用[`collect()`](https://dplyr.tidyverse.org/reference/compute.xhtml)强制 Arrow 执行计算并返回一些数据。例如，此代码告诉我们每年的总借阅次数：

```
seattle_csv |> 
  count(CheckoutYear, wt = Checkouts) |> 
  arrange(CheckoutYear) |> 
  collect()
#> # A tibble: 18 × 2
#>   CheckoutYear       n
#>          <int>   <int>
#> 1         2005 3798685
#> 2         2006 6599318
#> 3         2007 7126627
#> 4         2008 8438486
#> 5         2009 9135167
#> 6         2010 8608966
#> # … with 12 more rows
```

由于 Arrow 的支持，无论基础数据集有多大，此代码都将有效运行。但目前速度较慢：在 Hadley 的计算机上运行大约需要 10 秒。考虑到我们有多少数据，这并不糟糕，但如果切换到更好的格式，可以使其运行速度更快。

# Parquet 格式

为了使这些数据更易于处理，让我们切换到 Parquet 文件格式，并将其拆分成多个文件。接下来的部分将首先介绍 Parquet 和分区，然后应用我们学到的内容到 Seattle 图书馆数据。

## Parquet 的优势

像 CSV 一样，Parquet 用于矩形数据，但不是可以用任何文件编辑器读取的文本格式，而是一个专门为大数据需求设计的自定义二进制格式。这意味着：

+   Parquet 文件通常比等价的 CSV 文件更小。Parquet 依赖于[高效编码](https://oreil.ly/OzpFo)来减少文件大小，并支持文件压缩。这有助于使 Parquet 文件快速，因为需要从磁盘移到内存的数据更少。

+   Parquet 文件具有丰富的类型系统。正如我们在“控制列类型”中讨论的那样，CSV 文件不提供任何关于列类型的信息。例如，CSV 读取器必须猜测`"08-10-2022"`应该被解析为字符串还是日期。相比之下，Parquet 文件以记录类型及其数据的方式存储数据。

+   Parquet 文件是“列导向”的。这意味着它们按列组织，类似于 R 的数据框架。与按行组织的 CSV 文件相比，这通常会导致数据分析任务的性能更好。

+   Parquet 文件是“分块”的，这使得可以同时处理文件的不同部分，并且如果幸运的话，甚至可以跳过一些块。

## 分区

随着数据集变得越来越大，将所有数据存储在单个文件中变得越来越痛苦，将大型数据集分割成许多文件通常非常有用。当这种结构化工作得聪明时，这种策略可以显著提高性能，因为许多分析只需要一部分文件的子集。

没有关于如何分区数据集的硬性规则：结果将取决于您的数据、访问模式和读取数据的系统。在找到适合您情况的理想分区之前，您可能需要进行一些实验。作为一个粗略的指南，Arrow 建议避免小于 20 MB 和大于 2 GB 的文件，并避免产生超过 10,000 个文件的分区。您还应尝试按照您进行过滤的变量进行分区；正如您马上将看到的那样，这样可以使 Arrow 跳过许多工作，仅读取相关文件。

## 重写西雅图图书馆数据

让我们将这些想法应用到西雅图图书馆的数据中，看看它们在实践中如何运作。我们将按照 `CheckoutYear` 进行分区，因为一些分析可能只想查看最近的数据，并且按年份分区得到了 18 个合理大小的块。

为了重写数据，我们使用 [`dplyr::group_by()`](https://dplyr.tidyverse.org/reference/group_by.xhtml) 定义分区，然后使用 [`arrow::write_dataset()`](https://arrow.apache.org/docs/r/reference/write_dataset.xhtml) 将分区保存到目录中。[`write_dataset()`](https://arrow.apache.org/docs/r/reference/write_dataset.xhtml) 有两个重要的参数：我们将创建文件的目录和我们将使用的格式。

```
pq_path <- "data/seattle-library-checkouts"
```

```
seattle_csv |>
  group_by(CheckoutYear) |>
  write_dataset(path = pq_path, format = "parquet")
```

这需要大约一分钟才能运行；正如我们马上将看到的，这是一个初始投资，通过使未来的操作更快来得到回报。

让我们看看我们刚刚产生的内容：

```
tibble(
  files = list.files(pq_path, recursive = TRUE),
  size_MB = file.size(file.path(pq_path, files)) / 1024²
)
#> # A tibble: 18 × 2
#>   files                            size_MB
#>   <chr>                              <dbl>
#> 1 CheckoutYear=2005/part-0.parquet    109.
#> 2 CheckoutYear=2006/part-0.parquet    164.
#> 3 CheckoutYear=2007/part-0.parquet    178.
#> 4 CheckoutYear=2008/part-0.parquet    195.
#> 5 CheckoutYear=2009/part-0.parquet    214.
#> 6 CheckoutYear=2010/part-0.parquet    222.
#> # … with 12 more rows
```

我们的单个 9 GB CSV 文件已经被重写为 18 个 parquet 文件。文件名使用了由 [Apache Hive 项目](https://oreil.ly/kACzC) 使用的“自描述”约定。Hive 风格的分区将文件夹命名为“key=value”的约定，因此您可能猜到，`CheckoutYear=2005` 目录包含所有 `CheckoutYear` 是 2005 年的数据。每个文件大小在 100 到 300 MB 之间，总大小现在约为 4 GB，略大于原始 CSV 文件大小的一半。这正如我们预期的那样，因为 parquet 是一种更高效的格式。

# 使用 dplyr 与 Arrow

现在我们已经创建了这些 parquet 文件，我们需要再次读取它们。我们再次使用 [`open_dataset()`](https://arrow.apache.org/docs/r/reference/open_dataset.xhtml)，但这次我们给它一个目录：

```
seattle_pq <- open_dataset(pq_path)
```

现在我们可以编写我们的 dplyr 流水线。例如，我们可以计算过去五年每个月借出的书籍总数：

```
query <- seattle_pq |> 
  filter(CheckoutYear >= 2018, MaterialType == "BOOK") |>
  group_by(CheckoutYear, CheckoutMonth) |>
  summarize(TotalCheckouts = sum(Checkouts)) |>
  arrange(CheckoutYear, CheckoutMonth)
```

为 Arrow 数据编写 dplyr 代码在概念上与 dbplyr 类似，如 第二十一章 中讨论的：你编写 dplyr 代码，它会自动转换为 Apache Arrow C++ 库理解的查询，当你调用 [`collect()`](https://dplyr.tidyverse.org/reference/compute.xhtml) 时执行。

```
query
#> FileSystemDataset (query)
#> CheckoutYear: int32
#> CheckoutMonth: int64
#> TotalCheckouts: int64
#> 
#> * Grouped by CheckoutYear
#> * Sorted by CheckoutYear [asc], CheckoutMonth [asc]
#> See $.data for the source Arrow object
```

调用 [`collect()`](https://dplyr.tidyverse.org/reference/compute.xhtml) 即可获取结果：

```
query |> collect()
#> # A tibble: 58 × 3
#> # Groups:   CheckoutYear [5]
#>   CheckoutYear CheckoutMonth TotalCheckouts
#>          <int>         <int>          <int>
#> 1         2018             1         355101
#> 2         2018             2         309813
#> 3         2018             3         344487
#> 4         2018             4         330988
#> 5         2018             5         318049
#> 6         2018             6         341825
#> # … with 52 more rows
```

与 dbplyr 类似，Arrow 仅理解某些 R 表达式，因此可能无法像平常那样编写完全相同的代码。不过，支持的操作和函数列表相当广泛，并且在不断增加中；可以在 [`?acero`](https://arrow.apache.org/docs/r/reference/acero.xhtml) 中找到目前支持的完整列表。

## 性能

让我们快速看一下从 CSV 切换到 Parquet 对性能的影响。首先，让我们计算将数据存储为单个大型 CSV 文件时，在 2021 年每个月的借阅书籍数量所需的时间：

```
seattle_csv |> 
  filter(CheckoutYear == 2021, MaterialType == "BOOK") |>
  group_by(CheckoutMonth) |>
  summarize(TotalCheckouts = sum(Checkouts)) |>
  arrange(desc(CheckoutMonth)) |>
  collect() |> 
  system.time()
#>    user  system elapsed 
#>  11.997   1.189  11.343
```

现在让我们使用我们的新版本数据集，在这个版本中，西雅图图书馆的借阅数据已经被分成了 18 个更小的 Parquet 文件：

```
seattle_pq |> 
  filter(CheckoutYear == 2021, MaterialType == "BOOK") |>
  group_by(CheckoutMonth) |>
  summarize(TotalCheckouts = sum(Checkouts)) |>
  arrange(desc(CheckoutMonth)) |>
  collect() |> 
  system.time()
#>    user  system elapsed 
#>   0.272   0.063   0.063
```

性能提升约 100 倍归因于两个因素：多文件分区和单个文件的格式：

+   分区可以提高性能，因为此查询使用 `CheckoutYear == 2021` 来过滤数据，而 Arrow 足够智能，能识别出它只需读取 18 个 Parquet 文件中的一个。

+   Parquet 格式通过以二进制格式存储数据来提高性能，可以更直接地读入内存。按列存储的格式和丰富的元数据意味着 Arrow 只需读取查询中实际使用的四列（`CheckoutYear`、`MaterialType`、`CheckoutMonth` 和 `Checkouts`）。

这种性能上的巨大差异是将大型 CSV 文件转换为 Parquet 格式是值得的！

## 使用 dbplyr 和 Arrow

Parquet 和 Arrow 的最后一个优势是很容易将 Arrow 数据集转换成 DuckDB 数据库（第二十一章）只需调用 [`arrow::to_duckdb()`](https://arrow.apache.org/docs/r/reference/to_duckdb.xhtml)：

```
seattle_pq |> 
  to_duckdb() |>
  filter(CheckoutYear >= 2018, MaterialType == "BOOK") |>
  group_by(CheckoutYear) |>
  summarize(TotalCheckouts = sum(Checkouts)) |>
  arrange(desc(CheckoutYear)) |>
  collect()
#> Warning: Missing values are always removed in SQL aggregation functions.
#> Use `na.rm = TRUE` to silence this warning
#> This warning is displayed once every 8 hours.
#> # A tibble: 5 × 2
#>   CheckoutYear TotalCheckouts
#>          <int>          <dbl>
#> 1         2022        2431502
#> 2         2021        2266438
#> 3         2020        1241999
#> 4         2019        3931688
#> 5         2018        3987569
```

[`to_duckdb()`](https://arrow.apache.org/docs/r/reference/to_duckdb.xhtml) 的好处在于转移过程中不涉及任何内存复制，并且符合 Arrow 生态系统的目标：实现从一个计算环境到另一个计算环境的无缝过渡。

# 摘要

在本章中，您已经初步了解了 arrow 包，它为处理大型磁盘数据集提供了 dplyr 后端支持。它可以处理 CSV 文件，并且如果您将数据转换为 parquet 格式，速度会快得多。Parquet 是一种二进制数据格式，专为在现代计算机上进行数据分析而设计。与 CSV 文件相比，能够处理 parquet 文件的工具较少，但是其分区、压缩和列式结构使其分析效率大大提高。

接下来，您将学习有关您的第一个非矩形数据源的内容，您将使用 tidyr 包提供的工具处理此类数据。我们将重点关注来自 JSON 文件的数据，但是无论数据源如何，一般的原则都适用于类似树形的数据。
