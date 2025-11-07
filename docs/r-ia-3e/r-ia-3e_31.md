# 附录 C. 从 R 导出数据

在第二章中，我们回顾了将数据导入 R 的多种方法。但有时，您可能想要走另一条路——从 R 导出数据——以便数据可以存档或导入到外部应用中。在本附录中，您将学习如何将 R 对象输出到分隔文本文件、Excel 电子表格或统计应用（如 SPSS、SAS 或 Stata）。

## C.1 分隔文本文件

您可以使用 `write.table()` 函数将 R 对象输出到分隔文本文件。格式如下

```
write.table(*X*, *outfile*, sep=*delimiter*, quote=TRUE, na="NA")
```

其中 `x` 是对象，`outfile` 是目标文件。例如，以下语句

```
write.table(mydata, "mydata.txt", sep=",")
```

将数据集 `mydata` 保存为当前工作目录中名为 mydata.txt 的逗号分隔文件。要保存输出文件到其他位置，请包含路径（例如，`"c:/myprojects/mydata.txt"`）。将 `sep=","` 替换为 `sep="\t"` 将数据保存为制表符分隔文件。默认情况下，字符串用引号（`""`）括起来，缺失值以 `NA` 写入。

## C.2 Excel 电子表格

`xlsx` 包中的 `write.xlsx()` 函数可用于将 R 数据框保存到 Excel 2007 工作簿。格式如下

```
library(xlsx)
write.xlsx(*X*, *outfile*, col.Names=TRUE, row.names=TRUE, 
           sheetName="Sheet 1", append=FALSE)
```

例如，以下语句

```
library(xlsx)
write.xlsx(mydata, "mydata.xlsx")
```

将数据框 `mydata` 导出为当前工作目录中名为 mydata.xlsx 的 Excel 工作簿中的工作表（默认为 Sheet 1）。默认情况下，数据集中的变量名用于创建电子表格中的列标题，行名放置在电子表格的第一列。如果 mydata.xlsx 已经存在，则会被覆盖。

`xlsx` 包是操作 Excel 工作簿的强大工具。有关更多详细信息，请参阅包文档。

## C.3 统计应用

`foreign` 包中的 `write.foreign()` 函数可用于将数据框导出到外部统计应用。创建了两个文件：一个包含数据的自由格式文本文件和一个包含将数据读入外部统计应用的指令的代码文件。格式如下

```
write.foreign(*dataframe*, *datafile*, *codefile*, package=*package*)
```

例如，以下代码

```
library(foreign)
write.foreign(mydata, "mydata.txt", "mycode.sps", package="SPSS")
```

将数据框 `mydata` 导出为当前工作目录中名为 mydata.txt 的自由格式文本文件，以及一个名为 mycode.sps 的 SPSS 程序，该程序可用于读取文本文件。`package` 的其他值包括 `"SAS"` 和 `"Stata"`。

要了解更多关于从 R 导出数据的信息，请参阅“R 数据导入/导出”文档，可在 [`cran.r-project.org/doc/manuals/R-data.pdf`](http://cran.r-project.org/doc/manuals/R-data.pdf) 获取。
