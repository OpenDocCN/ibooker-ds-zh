第 9 章：Excel 自动化

到目前为止，我们已经学会了如何用 pandas 替换典型的 Excel 任务（[第 II 部分](index_split_013.html#filepos433190)），以及如何将 Excel 文件作为数据源和报告文件格式（[第 III 部分](index_split_018.html#filepos863198)）。本章开启了[第 IV 部分](index_split_023.html#filepos1235617)，在这一部分中，我们不再使用读者和写者包来操作 Excel 文件，而是开始使用 xlwings 自动化 Excel 应用程序。

xlwings 的主要用途是构建交互式应用程序，其中 Excel 电子表格充当用户界面，允许您通过单击按钮或调用用户定义函数来调用 Python —— 这种功能不被读取器和写入器包覆盖。但这并不意味着 xlwings 不能用于读写文件，只要您在 macOS 或 Windows 上安装了 Excel。xlwings 在这方面的一个优势是能够真正编辑 Excel 文件，而不改变或丢失任何现有内容或格式。另一个优势是，您可以从 Excel 工作簿中读取单元格值，而无需先保存它。然而，将 Excel 读取器/写入器包和 xlwings 结合使用也是完全合理的，正如我们将在[第 7 章](index_split_019.html#filepos863345)的报告案例研究中看到的那样。

本章将首先介绍 Excel 对象模型以及 xlwings：我们将首先学习如何连接工作簿、读写单元格数值等基础知识，然后深入了解转换器和选项是如何允许我们处理 pandas 数据帧和 NumPy 数组的。我们还将看看如何与图表、图片和定义名称进行交互，然后转向最后一节，解释 xlwings 在幕后的工作原理：这将为您提供所需的知识，使您的脚本性能更高，并解决缺少功能的问题。从本章开始，您需要在 Windows 或 macOS 上运行代码示例，因为它们依赖于本地安装的 Microsoft Excel。[1](index_split_025.html#filepos1437419)

开始使用 xlwings

xlwings的一个目标是作为VBA的替代品，允许您在Windows和macOS上从Python与Excel进行交互。由于Excel的网格是显示Python数据结构（如嵌套列表、NumPy数组和pandas DataFrame）的理想布局，xlwings的核心特性之一是尽可能地简化从Excel读取和写入这些数据结构。我将从介绍Excel作为数据查看器开始这一节——当您在Jupyter笔记本中与DataFrame交互时，这非常有用。然后我将解释Excel对象模型，然后使用xlwings进行交互式探索。最后，我将向您展示如何调用可能仍在遗留工作簿中的VBA代码。由于xlwings是Anaconda的一部分，我们不需要手动安装它。

使用Excel作为数据查看器

在前几章中，您可能已经注意到，默认情况下，Jupyter笔记本会隐藏更大的DataFrame的大部分数据，仅显示顶部和底部的行以及前几列和最后几列。了解数据的更好方法之一是绘制它——这使您能够发现异常值或其他不规则情况。然而，有时，能够滚动查看数据表确实非常有帮助。在阅读[第7章](index_split_019.html#filepos863345)之后，您已经了解如何在DataFrame上使用`to_excel`方法。虽然这样做可以实现，但可能有些繁琐：您需要为Excel文件命名，找到它在文件系统中的位置，打开它，在对DataFrame进行更改后，您需要关闭Excel文件，并重新运行整个过程。更好的方法可能是运行`df.to_clipboard()`，它将DataFrame `df`复制到剪贴板，使您可以将其粘贴到Excel中，但更简单的方法是使用xlwings提供的`view`函数：

> `In``[``1``]:``# 首先，让我们导入本章将要使用的包``import``datetime``as``dt``import``xlwings``as``xw``import``pandas``as``pd``import``numpy``as``np`
> 
> `In``[``2``]:``# 让我们创建一个基于伪随机数的DataFrame，并且有足够的行数，以至于只显示头部和尾部``df``=``pd``.``DataFrame``(``data``=``np``.``random``.``randn``(``100``,``5``),``columns``=``[``f``"试验 {i}"``for``i``in``range``(``1``,``6``)])``df``
> 
> `Out[2]:      Trial 1   Trial 2   Trial 3   Trial 4   Trial 5         0  -1.313877  1.164258 -1.306419 -0.529533 -0.524978         1  -0.854415  0.022859 -0.246443 -0.229146 -0.005493         2  -0.327510 -0.492201 -1.353566 -1.229236  0.024385         3  -0.728083 -0.080525  0.628288 -0.382586 -0.590157         4  -1.227684  0.498541 -0.266466  0.297261 -1.297985         ..       ...       ...       ...       ...       ...         95 -0.903446  1.103650  0.033915  0.336871  0.345999         96 -1.354898 -1.290954 -0.738396 -1.102659  0.115076         97 -0.070092 -0.416991 -0.203445 -0.686915 -1.163205         98 -1.201963  0.471854 -0.458501 -0.357171  1.954585         99  1.863610  0.214047 -1.426806  0.751906 -2.338352          [100 rows x 5 columns]`
> 
> `In``[``3``]:``# 在Excel中查看DataFrame``xw``.``view``(``df``)`

`view` 函数接受所有常见的 Python 对象，包括数字、字符串、列表、字典、元组、NumPy 数组和 pandas 数据框。默认情况下，它会打开一个新工作簿，并将对象粘贴到第一个工作表的 A1 单元格中——甚至可以使用 Excel 的自动调整功能调整列宽。您还可以通过将 xlwings 的 `sheet` 对象作为第二个参数提供给 `view` 函数，以重复使用同一个工作簿：`xw.view(df, mysheet)`。如何获取这样的 `sheet` 对象以及它如何适配到 Excel 对象模型中，这就是我接下来将要解释的内容。[2](index_split_025.html#filepos1437838)

> MACOS：权限和偏好
> 
> 在 macOS 上，请确保从 Anaconda Prompt（即通过终端）运行 Jupyter 笔记本和 VS Code，如 [第 2 章](index_split_008.html#filepos96824) 所示。这样可以确保第一次使用 xlwings 时会弹出两个弹窗：第一个是“终端想要控制系统事件”，第二个是“终端想要控制 Microsoft Excel”。您需要确认这两个弹窗以允许 Python 自动化 Excel。理论上，任何从中运行 xlwings 代码的应用程序都应该触发这些弹窗，但实际上，通常并非如此，因此通过终端运行它们可以避免麻烦。此外，您需要打开 Excel 的偏好设置，并取消“打开 Excel 时显示工作簿库”选项，该选项在“常规”类别下。这样可以直接在空工作簿中打开 Excel，而不是首先打开库，这样当您通过 xlwings 打开新的 Excel 实例时就不会受到干扰。

Excel对象模型

当你以编程方式使用 Excel 时，你会与其组件进行交互，比如工作簿或工作表。这些组件在 Excel 对象模型中组织，这是一个层次结构，代表了 Excel 的图形用户界面（见[图 9-1](#filepos1253700)）。Microsoft 在所有官方支持的编程语言中基本上使用相同的对象模型，无论是 VBA、Office 脚本（Excel 在 Web 上的 JavaScript 接口）还是 C#。与第[8章](index_split_020.html#filepos959867)中的读写包相比，xlwings 非常紧密地遵循了 Excel 对象模型，只是稍微有所创新：例如，xlwings 使用`app`代替`application`，`book`代替`workbook`：

+   > > > > 一个`app`包含`books`集合
+   > > > > 
+   > > > > 一个`book`包含`sheets`集合
+   > > > > 
+   > > > > 一个`sheet`提供对`range`对象和集合（如`charts`）的访问
+   > > > > 
+   > > > > 一个`range`包含一个或多个连续的单元格作为其项目

虚线框是集合，包含同一类型的一个或多个对象。一个`app`对应于一个 Excel 实例，即运行为单独进程的 Excel 应用程序。高级用户有时会并行使用多个 Excel 实例打开同一工作簿，例如，为了并行计算带有不同输入的工作簿。在更近期的 Excel 版本中，Microsoft 稍微增加了手动打开多个 Excel 实例的复杂性：启动 Excel，然后在 Windows 任务栏中右键单击其图标。在出现的菜单中，同时按住 Alt 键单击 Excel 条目（确保在释放鼠标按钮之后继续按住 Alt 键）——一个弹出窗口会询问是否要启动新的 Excel 实例。在 macOS 上，没有手动启动多个相同程序实例的方式，但是可以通过 xlwings 在编程方式下启动多个 Excel 实例，稍后我们将看到。总之，Excel 实例是一个隔离环境，这意味着一个实例无法与另一个实例通信。[3](index_split_025.html#filepos1438450) `sheet`对象让您访问诸如图表、图片和定义名称等集合——这些是我们将在本章第二部分中探讨的主题。

![](images/00006.jpg)

图 9-1\. 由 xlwings 实现的 Excel 对象模型（节选）

> 语言和区域设置
> 
> 本书基于 Excel 的美国英语版本。我偶尔会提到默认名称如“Book1”或“Sheet1”，如果你使用其他语言的 Excel，则名称会不同。例如，法语中的“Sheet1”称为“Feuille1”，西班牙语中称为“Hoja1”。此外，列表分隔符，即 Excel 在单元格公式中使用的分隔符，取决于您的设置：我将使用逗号，但您的版本可能需要分号或其他字符。例如，不是写`=SUM(A1, A2)`，而是在具有德国区域设置的计算机上写`=SUMME(A1; A2)`。
> 
> 在 Windows 上，如果您想将列表分隔符从分号更改为逗号，需要在 Excel 之外的 Windows 设置中更改它：点击 Windows 开始按钮，搜索“设置”（或点击齿轮图标），然后转到“时间与语言” > “地区与语言” > “附加日期、时间和区域设置”，最后点击“区域” > “更改位置”。在“列表分隔符”下，您将能够将其从分号更改为逗号。请注意，仅当您的“小数符号”（在同一菜单中）不是逗号时，此设置才有效。要覆盖系统范围内的小数和千位分隔符（但不更改列表分隔符），请在 Excel 中转到“选项” > “高级”，在“编辑选项”下找到相关设置。
> 
> 在 macOS 上，操作类似，不过你无法直接更改列表分隔符：在 macOS 的系统偏好设置（而非 Excel）中，选择“语言与地区”。在那里，为 Excel（在“应用程序”选项卡下）或全局（在“常规”选项卡下）设置特定的地区。

要熟悉 Excel 对象模型，通常最好是通过互动方式来操作。让我们从 `Book` 类开始：它允许您创建新工作簿并连接到现有工作簿；参见[表格 9-1](#filepos1256373)以获取概述。

表格 9-1\. 使用 Excel 工作簿

|  命令  |  描述  |
| --- | --- |
|   `xw.Book()` |  返回一个表示活动 Excel 实例中新 Excel 工作簿的`book`对象。如果没有活动实例，Excel 将会启动。 |
|   `xw.Book("Book1")` |  返回一个表示未保存的名为 Book1 的工作簿的`book`对象（不带文件扩展名）。 |
|   `xw.Book("Book1.xlsx")` |  返回一个表示已保存的名为 Book1.xlsx 的工作簿的`book`对象（带有文件扩展名）。文件必须是打开的或者在当前工作目录中。 |
|   `xw.Book(r"C:\path\Book1.xlsx")` |  返回一个表示已保存的完整文件路径的工作簿的`book`对象。文件可以是打开的或关闭的。使用前缀 `r` 将字符串转换为原始字符串，使得 Windows 下的反斜杠（`\`）被直接解释（我在[第五章](index_split_015.html#filepos482650)介绍了原始字符串）。在 macOS 上，不需要 `r` 前缀，因为文件路径使用正斜杠而不是反斜杠。 |
|   `xw.books.active` |  返回活动 Excel 实例中活动工作簿的`book`对象。 |

让我们看看如何从 `book` 对象逐步遍历对象模型层次结构到 `range` 对象：

> `In``[``4``]:``# 创建一个新的空工作簿并打印其名称。这是我们将在本章中大多数代码示例中使用的`book`。``book``=``xw``.``Book``()``book``.``name`
> 
> `Out[4]: 'Book2'`
> 
> `In``[``5``]:``# 访问工作表集合``book``.``sheets`
> 
> `Out[5]: Sheets([<Sheet [Book2]Sheet1>])`
> 
> `In``[``6``]:``# 通过索引或名称获取工作表对象。如果您的工作表名称不同，您需要调整`Sheet1`。``sheet1``=``book``.``sheets``[``0``]``sheet1``=``book``.``sheets``[``"Sheet1"``]`
> 
> `In``[``7``]:``sheet1``.``range``(``"A1"``)`
> 
> `Out[7]: <Range [Book2]Sheet1!$A$1>`

通过 `range` 对象，我们已经到达了层次结构的底部。尖括号中打印的字符串为您提供有关该对象的有用信息，但通常要使用具有属性的对象，如下一个示例所示：

> `In``[``8``]:``# 最常见的任务：写入值...``sheet1``.``range``(``"A1"``)``.``value``=``[[``1``,``2``],``[``3``,``4``]]``sheet1``.``range``(``"A4"``)``.``value``=``"Hello!"`
> 
> `In``[``9``]:``# ...和读取值``sheet1``.``range``(``"A1:B2"``)``.``value`
> 
> `Out[9]: [[1.0, 2.0], [3.0, 4.0]]`
> 
> `In``[``10``]:``sheet1``.``range``(``"A4"``)``.``value`
> 
> `Out[10]: 'Hello!'`

正如您所见，xlwings 的 `range` 对象的 `value` 属性默认接受和返回两维范围的嵌套列表和单个单元格的标量。到目前为止，我们几乎与 VBA 完全一致：假设 `book` 分别是 VBA 或 xlwings 工作簿对象，这是如何从 A1 到 B2 的单元格访问 `value` 属性的方法：

> `book``.``Sheets``(``1``)``.``Range``(``"A1:B2"``)``.``Value``# VBA``book``.``sheets``[``0``]``.``range``(``"A1:B2"``)``.``value``# xlwings`

差异在于：

属性

> > Python 使用小写字母，可能带有下划线，如 PEP 8 所建议的 Python 样式指南，我在 [第 3 章](index_split_010.html#filepos178328) 中介绍过。

索引

> > Python 使用方括号和从零开始的索引来访问 `sheets` 集合中的元素。

[表 9-2](#filepos1273710) 提供了 xlwings `range` 接受的字符串的概述。

表 9-2\. 使用 A1 表示法定义范围的字符串

|  引用  |  描述  |
| --- | --- |
|   `"A1"` |  单个单元格  |
|   `"A1:B2"` |  从 A1 到 B2 的单元格  |
|   `"A:A"` |  A 列  |
|   `"A:B"` |  A 到 B 列  |
|   `"1:1"` |  第 1 行  |
|   `"1:2"` |  1 到 2 行  |

索引和切片适用于 xlwings 的 `range` 对象 — 注意尖括号中的地址（打印的对象表示）以查看您最终使用的单元格范围：

> `In``[``11``]:``# 索引``sheet1``.``range``(``"A1:B2"``)[``0``,``0``]`
> 
> `Out[11]: <Range [Book2]Sheet1!$A$1>`
> 
> `In``[``12``]:``# 切片``sheet1``.``range``(``"A1:B2"``)[:,``1``]`
> 
> `Out[12]: <Range [Book2]Sheet1!$B$1:$B$2>`

索引对应于在 VBA 中使用 `Cells` 属性：

> `book``.``Sheets``(``1``)``.``Range``(``"A1:B2"``)``.``Cells``(``1``,``1``)``# VBA``book``.``sheets``[``0``]``.``range``(``"A1:B2"``)[``0``,``0``]``# xlwings`

相反地，您也可以通过索引和切片`sheet`对象来获取`range`对象，而不是显式地使用`range`作为`sheet`对象的属性。使用A1表示法可以减少输入，使用整数索引可以使Excel工作表感觉像NumPy数组：

> `In``[``13``]:``# 单个单元格：A1表示法``sheet1``[``"A1"``]`
> 
> `Out[13]: <Range [Book2]Sheet1!$A$1>`
> 
> `In``[``14``]:``# 多个单元格：A1表示法``sheet1``[``"A1:B2"``]`
> 
> `Out[14]: <Range [Book2]Sheet1!$A$1:$B$2>`
> 
> `In``[``15``]:``# 单个单元格：索引``sheet1``[``0``,``0``]`
> 
> `Out[15]: <Range [Book2]Sheet1!$A$1>`
> 
> `In``[``16``]:``# 多个单元格：切片``sheet1``[:``2``,``:``2``]`
> 
> `Out[16]: <Range [Book2]Sheet1!$A$1:$B$2>`

有时，通过引用范围的左上角和右下角单元格来定义范围可能更直观。下面的示例分别引用了单元格范围D10和D10:F11，使您可以理解索引/切片`sheet`对象与处理`range`对象之间的区别：

> `In``[``17``]:``# 通过工作表索引访问D10``sheet1``[``9``,``3``]`
> 
> `Out[17]: <Range [Book2]Sheet1!$D$10>`
> 
> `In``[``18``]:``# 通过range对象访问D10``sheet1``.``range``((``10``,``4``))`
> 
> `Out[18]: <Range [Book2]Sheet1!$D$10>`
> 
> `In``[``19``]:``# 通过sheet切片访问D10:F11``sheet1``[``9``:``11``,``3``:``6``]`
> 
> `Out[19]: <Range [Book2]Sheet1!$D$10:$F$11>`
> 
> `In``[``20``]:``# 通过range对象访问D10:F11``sheet1``.``range``((``10``,``4``),``(``11``,``6``))`
> 
> `Out[20]: <Range [Book2]Sheet1!$D$10:$F$11>`

使用元组定义`range`对象与VBA中的`Cells`属性非常相似，如下面的比较所示——假设`book`再次是VBA工作簿对象或xlwings的`book`对象。让我们首先看看VBA版本：

> `With``book``.``Sheets``(``1``)``myrange``=``.``Range``(.``Cells``(``10``,``4``),``.``Cells``(``11``,``6``))``End``With`

这与以下xlwings表达式等效：

> `myrange``=``book``.``sheets``[``0``]``.``range``((``10``,``4``),``(``11``,``6``))`
> 
> 零索引与一索引
> 
> 作为Python包，xlwings在通过Python的索引或切片语法访问元素时始终使用零索引。然而，xlwings的`range`对象使用Excel的一索引行和列索引。与Excel用户界面具有相同的行/列索引有时可能是有益的。如果您希望仅使用Python的零索引，请简单地使用`sheet[row_selection, column_selection]`语法。

下面的示例向您展示如何从`range`对象(`sheet1["A1"]`)获取到`app`对象。请记住，`app`对象代表一个Excel实例（尖括号中的输出表示Excel的进程ID，因此在您的机器上可能会有所不同）：

> `In``[``21``]:``sheet1``[``"A1"``]``.``sheet``.``book``.``app`
> 
> `Out[21]: <Excel App 9092>`

已经到达Excel对象模型的顶端，现在是时候看看如何处理多个Excel实例了。如果您想在多个Excel实例中打开相同的工作簿，或者特别是出于性能原因希望在不同实例中分配您的工作簿，那么您需要明确使用`app`对象。使用`app`对象的另一个常见用例是在隐藏的Excel实例中打开工作簿：这使得您可以在后台运行xlwings脚本，同时又不会阻碍您在Excel中进行其他工作：

> `In``[``22``]:``# 从打开的工作簿获取一个应用对象``# 并创建一个额外的不可见应用实例``visible_app``=``sheet1``.``book``.``app``invisible_app``=``xw``.``App``(``visible``=``False``)`
> 
> `In``[``23``]:``# 使用列表推导列出每个实例中打开的书名``[``visible_app``.``books``中的`book.name`]
> 
> `Out[23]: ['Book1', 'Book2']`
> 
> `In``[``24``]:``[``invisible_app``.``books``中的`book.name`]
> 
> `Out[24]: ['Book3']`
> 
> `In``[``25``]:``# 应用密钥表示进程ID（PID）``xw``.``apps``.``keys``()``
> 
> `Out[25]: [5996, 9092]`
> 
> `In``[``26``]:``# 也可以通过pid属性访问``xw``.``apps``.``active``.``pid``
> 
> `Out[26]: 5996`
> 
> `In``[``27``]:``# 在不可见的Excel实例中处理工作簿``invisible_book``=``invisible_app``.``books``[``0``]``invisible_book``.``sheets``[``0``][``"A1"``]``.``value``=``"由不可见应用程序创建。"``
> 
> `In``[``28``]:``# 将Excel工作簿保存在xl目录中``invisible_book``.``save``(``"xl/invisible.xlsx"``)`
> 
> `In``[``29``]:``# 退出不可见的Excel实例``invisible_app``.``quit``()`
> 
> MACOS：以编程方式访问文件系统
> 
> 如果您在macOS上运行`save`命令，Excel会弹出授权文件访问的提示窗口，您需要点击“选择”按钮确认，然后再点击“授权访问”。在macOS上，Excel是沙盒化的，这意味着您的程序只能通过确认此提示才能访问Excel应用程序外的文件和文件夹。确认后，Excel将记住位置，在下次运行脚本时再次运行时不会再打扰您。

如果您在两个Excel实例中打开相同的工作簿，或者想要指定在哪个Excel实例中打开工作簿，就不能再使用`xw.Book`了。相反，您需要使用在[表 9-3](#filepos1310515)中描述的`books`集合。请注意，`myapp`代表一个xlwings的`app`对象。如果您用`xw.books`替换`myapp.books`，xlwings将使用活动的`app`。

表 9-3\. 使用`books`集合操作

|  命令  |  描述  |
| --- | --- |
|   `myapp.books.add()` |  在`myapp`所引用的Excel实例中创建一个新的Excel工作簿，并返回相应的`book`对象。 |
|   `myapp.books.open(r"C:\path\Book.xlsx")` |  如果该书已打开，则返回`book`，否则首先在`myapp`引用的Excel实例中打开。请记住，前导的`r`将文件路径转换为原始字符串，以字面上的方式解释反斜杠。 |
|   `myapp.books["Book1.xlsx"]` |  如果该书已打开，则返回`book`对象。如果尚未打开，则会引发`KeyError`。如果你需要知道工作簿在Excel中是否已打开，请使用这个。请确保使用名称而不是完整路径。 |

在我们深入探讨xlwings如何替代你的VBA宏之前，让我们看看xlwings如何与你现有的VBA代码交互：如果你有大量的遗留代码，没有时间将所有内容迁移到Python，这可能非常有用。

运行VBA代码

如果你有大量的带有VBA代码的遗留Excel项目，将所有内容迁移到Python可能需要很多工作。在这种情况下，你可以使用Python来运行你的VBA宏。下面的示例使用了伴随库的xl文件夹中的vba.xlsm文件。它在Module1中包含以下代码：

> `Function``MySum``(``x``As``Double``,``y``As``Double``)``As``Double``MySum``=``x``+``y``End``Function`
> 
> `Sub``ShowMsgBox``(``msg``As``String``)``MsgBox``msg``End``Sub`

要通过Python调用这些函数，你首先需要实例化一个xlwings `macro`对象，然后调用它，使其感觉像是本地Python函数：

> `In``[``30``]:``vba_book``=``xw``.``Book``(``"xl/vba.xlsm"``)`
> 
> `In``[``31``]:``# 用VBA函数实例化宏对象``mysum``=``vba_book``.``macro``(``"Module1.MySum"``)``# 调用VBA函数``mysum``(``5``,``4``)`
> 
> `Out[31]: 9.0`
> 
> `In``[``32``]:``# 使用VBA Sub过程同样有效``show_msgbox``=``vba_book``.``macro``(``"Module1.ShowMsgBox"``)``show_msgbox``(``"Hello xlwings!"``)`
> 
> `In``[``33``]:``# 再次关闭该书（确保先关闭MessageBox）``vba_book``.``close``()`
> 
> 不要将VBA函数存储在工作表和此工作簿模块中
> 
> 如果你将VBA函数`MySum`存储在工作簿模块`ThisWorkbook`或工作表模块（例如`Sheet1`）中，你必须将其称为`ThisWorkbook.MySum`或`Sheet1.MySum`。然而，你将无法从Python访问函数的返回值，所以请确保将VBA函数存储在通过在VBA编辑器中右键单击模块文件夹插入的标准VBA代码模块中。

现在你知道如何与现有的VBA代码交互了，我们可以继续探索xlwings的使用方法，看看如何与数据框、NumPy数组和图表、图片以及已定义名称等集合一起使用它。

转换器、选项和集合

在本章的介绍性代码示例中，我们已经通过使用 xlwings 的 `range` 对象的 `value` 属性来读取和写入 Excel 中的字符串和嵌套列表。在深入研究允许我们影响 xlwings 读取和写入值的 `options` 方法之前，我将向您展示如何使用 pandas DataFrames 进行操作。我们继续处理图表、图片和已定义名称，这些通常可以从 `sheet` 对象访问。掌握这些 xlwings 基础知识后，我们将再次审视第 [7 章](index_split_019.html#filepos863345) 中的报告案例。

处理数据框

将数据框写入 Excel 与将标量或嵌套列表写入 Excel 没有任何区别：只需将数据框分配给 Excel 范围的左上角单元格：

> `In``[``34``]:``data``=``[[``"Mark"``,``55``,``"Italy"``,``4.5``,``"Europe"``],``[``"John"``,``33``,``"USA"``,``6.7``,``"America"``]]``df``=``pd``.``DataFrame``(``data``=``data``,``columns``=``[``"name"``,``"age"``,``"country"``,``"score"``,``"continent"``],``index``=``[``1001``,``1000``])``df``.``index``.``name``=``"user_id"``df`
> 
> `Out[34]:          name  age country  score continent          user_id          1001     Mark   55   Italy    4.5    Europe          1000     John   33     USA    6.7   America`
> 
> `In``[``35``]:``sheet1``[``"A6"``]``.``value``=``df`

如果您想抑制列标题和/或索引，则使用以下`options`方法：

> `In``[``36``]:``sheet1``[``"B10"``]``.``options``(``header``=``False``,``index``=``False``)``.``value``=``df`

将 Excel 范围作为数据框读取要求您在 `options` 方法中将 `DataFrame` 类作为 `convert` 参数提供。默认情况下，它期望您的数据具有标题和索引，但您可以再次使用 `index` 和 `header` 参数进行更改。而不是使用转换器，您还可以首先将值读取为嵌套列表，然后手动构建数据框，但使用转换器可以更轻松地处理索引和标题。

> THE EXPAND METHOD
> 
> 在下面的代码示例中，我将介绍 `expand` 方法，该方法使得读取一个连续的单元格块变得简单，提供与在 Excel 中执行 Shift+Ctrl+Down-Arrow+Right-Arrow 相同的范围，不同之处在于 `expand` 会跳过左上角的空单元格。
> 
> `In``[``37``]:``df2``=``sheet1``[``"A6"``]``.``expand``()``.``options``(``pd``.``DataFrame``)``.``value``df2`
> 
> `Out[37]:          name   age country  score continent          user_id          1001.0   Mark  55.0   Italy    4.5    Europe          1000.0   John  33.0     USA    6.7   America`
> 
> `In``[``38``]:``# 如果您希望索引是整数索引，则可以更改其数据类型``df2``.``index``=``df2``.``index``.``astype``(``int``)``df2`
> 
> `Out[38]:       name   age country  score continent          1001  Mark  55.0   Italy    4.5    Europe          1000  John  33.0     USA    6.7   America`
> 
> `In``[``39``]:``# 通过设置 index=False，它将把所有从 Excel 中获取的值放入 DataFrame 的数据部分，并使用默认索引``sheet1``[``"A6"``]``.``expand``()``.``options``(``pd``.``DataFrame``,``index``=``False``)``.``value`
> 
> `Out[39]:    user_id  name   age country  score continent          0   1001.0  Mark  55.0   Italy    4.5    Europe          1   1000.0  John  33.0     USA    6.7   America`

读取和写入 DataFrame 是转换器和选项如何工作的第一个示例。接下来我们将看一下它们是如何正式定义以及如何在其他数据结构中使用的。

转换器和选项

正如我们刚才所看到的，xlwings `range` 对象的 `options` 方法允许您影响从Excel读取和写入值的方式。也就是说，只有在调用 `range` 对象的 `value` 属性时才会评估 `options`。语法如下（`myrange` 是一个 xlwings 的 `range` 对象）：

> `myrange``.``options``(``convert``=``None``,``option1``=``value1``,``option2``=``value2``,``...``)``.``value`

[表 9-4](#filepos1343026) 显示了内置转换器，即 `convert` 参数接受的值。它们被称为内置，因为 xlwings 提供了一种方法来编写自己的转换器，如果需要重复应用额外的转换（例如在写入或读取值之前）时，这将非常有用——要了解它的工作原理，请参阅 [xlwings 文档](https://oreil.ly/Ruw8v)。

表 9-4\. 内置转换器

|  转换器  |  描述  |
| --- | --- |
|   `dict` |  简单的无嵌套字典，即 `{key1: value1, key2: value2, ...}` 的形式 |
|   `np.array` |  NumPy 数组，需要  `import numpy as np` |
|   `pd.Series` |  pandas Series，需要  `import pandas as pd` |
|   `pd.DataFrame` |  pandas DataFrame，需要  `import pandas as pd` |

我们已经在 DataFrame 示例中使用了 `index` 和 `header` 选项，但还有更多的选项可用，如 [表 9-5](#filepos1345335) 所示。

表 9-5\. 内置选项

|  选项  |  描述  |
| --- | --- |
|   `empty` |  默认情况下，空单元格被读取为  `None`。通过为 `empty` 提供值来更改这一点。 |
|   `date` |  接受应用于日期格式单元格值的函数。  |
|   `number` |  接受应用于数字的函数。  |
|   `ndim` |   维度数：在读取时，使用 `ndim` 强制将范围的值按特定维度到达。必须是 `None`、`1` 或 `2`。在读取值作为列表或 NumPy 数组时可用。 |
|   `transpose` |  转置值，即将列转换为行或反之。  |
|   `index` |  用于pandas的DataFrame和Series：在读取时，用于定义Excel范围是否包含索引。可以是`True`/`False`或整数。整数定义将多少列转换为`MultiIndex`。例如，`2`将使用最左边的两列作为索引。在写入时，可以通过将`index`设置为`True`或`False`来决定是否写出索引。 |
|   `header` |  与`index`相同，但应用于列标题。 |

让我们更仔细地看看`ndim`：默认情况下，从Excel读取单个单元格时，您会得到一个标量（例如，浮点数或字符串）；当从列或行读取时，您会得到一个简单的列表；最后，当从二维范围读取时，您会得到一个嵌套的（即二维的）列表。这不仅在自身上是一致的，而且等同于NumPy数组中切片的工作方式，正如在[第4章](index_split_014.html#filepos433313)中所见。一维情况是特例：有时，列可能只是否则是二维范围的边缘案例。在这种情况下，通过使用`ndim=2`强制范围始终以二维列表形式到达是有意义的：

> `In``[``40``]:``# 水平范围（一维）``sheet1``[``"A1:B1"``]``.``value`
> 
> `Out[40]: [1.0, 2.0]`
> 
> `In``[``41``]:``# 垂直范围（一维）``sheet1``[``"A1:A2"``]``.``value`
> 
> `Out[41]: [1.0, 3.0]`
> 
> `In``[``42``]:``# 水平范围（二维）``sheet1``[``"A1:B1"``]``.``options``(``ndim``=``2``)``.``value`
> 
> `Out[42]: [[1.0, 2.0]]`
> 
> `In``[``43``]:``# 垂直范围（二维）``sheet1``[``"A1:A2"``]``.``options``(``ndim``=``2``)``.``value`
> 
> `Out[43]: [[1.0], [3.0]]`
> 
> `In``[``44``]:``# 使用NumPy数组转换器的行为相同：``# 垂直范围导致一维数组``sheet1``[``"A1:A2"``]``.``options``(``np``.``array``)``.``value`
> 
> `Out[44]: array([1., 3.])`
> 
> `In``[``45``]:``# 保留列的方向``sheet1``[``"A1:A2"``]``.``options``(``np``.``array``,``ndim``=``2``)``.``value`
> 
> `Out[45]: array([[1.],                 [3.]])`
> 
> `In``[``46``]:``# 如果需要垂直写出列表，则`transpose`选项非常方便``sheet1``[``"D1"``]``.``options``(``transpose``=``True``)``.``value``=``[``100``,``200``]`

使用`ndim=1`强制将单个单元格的值读取为列表而不是标量。在pandas中，不需要`ndim`，因为DataFrame始终是二维的，Series始终是一维的。这里还有一个例子，展示了`empty`、`date`和`number`选项的工作方式：

> `In``[``47``]:``# 写入一些示例数据``sheet1``[``"A13"``]``.``value``=``[``dt``.``datetime``(``2020``,``1``,``1``),``None``,``1.0``]`
> 
> `In``[``48``]:``# 使用默认选项读取它``sheet1``[``"A13:C13"``]``.``value`
> 
> `Out[48]: [datetime.datetime(2020, 1, 1, 0, 0), None, 1.0]`
> 
> `In``[``49``]:``# 使用非默认选项将其读取回来``sheet1``[``"A13:C13"``]``.``options``(``empty``=``"NA"``,``dates``=``dt``.``date``,``numbers``=``int``)``.``value`
> 
> `Out[49]: [datetime.date(2020, 1, 1), 'NA', 1]`

到目前为止，我们已经使用了`book`、`sheet`和`range`对象。现在让我们继续学习如何处理从`sheet`对象访问的图表等集合！

图表、图片和定义名称

在本节中，我将向您展示如何处理通过`sheet`或`book`对象访问的三个集合：图表、图片和定义名称。[4](#filepos1438843) xlwings 仅支持最基本的图表功能，但由于您可以使用模板工作，您可能甚至不会错过太多内容。而且，为了补偿，xlwings 允许您将 Matplotlib 绘图嵌入为图片——您可能还记得来自[第 5 章](index_split_015.html#filepos482650)的信息，Matplotlib 是 pandas 的默认绘图后端。让我们从创建第一个 Excel 图表开始吧！

Excel 图表

要添加新图表，请使用`charts`集合的`add`方法，然后设置图表类型和源数据：

> `In``[``50``]:``sheet1``[``"A15"``]``.``value``=``[[``无``,``"北"``,``"南"``],``[``"上年度"``,``2``,``5``],``[``"今年"``,``3``,``6``]]`
> 
> `In``[``51``]:``chart``=``sheet1``.``charts``.``add``(``top``=``sheet1``[``"A19"``]``.``top``,``left``=``sheet1``[``"A19"``]``.``left``)``chart``.``chart_type``=``"column_clustered"``chart``.``set_source_data``(``sheet1``[``"A15"``]``.``expand``())`

这将生成左侧显示的图表，位于[图 9-2](#filepos1388244)。要查看可用的图表类型，请参阅[xlwings 文档](https://oreil.ly/2B58q)。如果你更喜欢使用 pandas 绘图而不是 Excel 图表，或者想使用 Excel 中没有的图表类型，xlwings 已经为你准备好了——让我们看看吧！

图片：Matplotlib 绘图

当您使用 pandas 的默认绘图后端时，您正在创建一个 Matplotlib 绘图。要将这样的绘图移至 Excel，您首先需要获取其`figure`对象，然后将其作为参数提供给`pictures.add`——这将把绘图转换为图片并发送至 Excel：

> `In``[``52``]:``# 将图表数据读取为 DataFrame``df``=``sheet1``[``"A15"``]``.``expand``()``.``options``(``pd``.``DataFrame``)``.``value``df`
> 
> `Out[52]:            北      南        上年度    2.0    5.0        今年    3.0    6.0`
> 
> `In``[``53``]:``# 通过使用 notebook 魔术命令启用 Matplotlib，并切换到`"seaborn"`风格``%``matplotlib``inline``import``matplotlib.pyplot``as``plt``plt``.``style``.``use``(``"seaborn"``)
> 
> `In``[``54``]:``# pandas 绘图方法返回一个“axis”对象，您可以从中获取图表。"T" 转置 DataFrame 以使绘图达到所需方向``ax``=``df``.``T``.``plot``.``bar``()``fig``=``ax``.``get_figure``()``
> 
> `In``[``55``]:``# 将图表发送到Excel``plot``=``sheet1``.``pictures``.``add``(``fig``,``name``=``"SalesPlot"``,``top``=``sheet1``[``"H19"``]``.``top``,``left``=``sheet1``[``"H19"``]``.``left``)``# 让我们将图表缩放到70%``plot``.``width``,``plot``.``height``=``plot``.``width``*``0.7``,``plot``.``height``*``0.7`

要使用新图表更新图片，只需使用`update`方法和另一个`figure`对象——这实际上将替换Excel中的图片，但会保留其所有属性，如位置、大小和名称：

> `In``[``56``]:``ax``=``(``df``+``1``)``.``T``.``plot``.``bar``()``plot``=``plot``.``update``(``ax``.``get_figure``())`

![](images/00059.jpg)

图 9-2\. Excel图表（左）和Matplotlib图表（右）

[图 9-2](#filepos1388244) 显示了Excel图表和Matplotlib图表在更新调用后的比较。

> 确保安装了PILLOW
> 
> 在处理图片时，请确保安装了[Pillow](https://oreil.ly/3HYkf)，Python的图片处理库：这将确保图片以正确的大小和比例到达Excel中。Pillow是Anaconda的一部分，因此如果您使用其他发行版，则需要通过运行`conda install pillow`或`pip install pillow`来安装它。请注意，`pictures.add`还可以接受磁盘上图片的路径，而不是Matplotlib图表。

图表和图片是通过`sheet`对象访问的集合。下面我们将看看如何访问定义名称集合，可以从`sheet`或`book`对象中访问。让我们看看这样做有什么区别！

定义名称

在Excel中，通过为范围、公式或常量分配名称来创建定义名称。[5](#filepos1439272) 将名称分配给范围可能是最常见的情况，称为命名范围。使用命名范围，您可以在公式和代码中使用描述性名称而不是形如`A1:B2`的抽象地址来引用Excel范围。与xlwings一起使用它们可以使您的代码更加灵活和稳固：从命名范围读取和写入值使您能够重新组织工作簿而无需调整Python代码：名称会粘附在单元格上，即使您通过插入新行等操作移动它。定义名称可以设置为全局工作簿范围或局部工作表范围。工作表范围的名称优势在于，您可以复制工作表而无需担心重复命名范围的冲突。在Excel中，您可以通过转到公式 > 定义名称或选择范围，然后在名称框中写入所需名称来手动添加定义名称——名称框位于公式栏左侧，默认显示单元格地址。以下是如何使用xlwings管理定义名称的方法：

> `In``[``57``]:``# 默认作用域为工作簿范围``sheet1``[``"A1:B2"``]``.``name``=``"matrix1"`
> 
> `In``[``58``]:``# 对于工作表范围，请使用叹号将工作表名称前缀``sheet1``[``"B10:E11"``]``.``name``=``"Sheet1!matrix2"`
> 
> `In``[``59``]:``# 现在你可以通过名字访问范围``sheet1``[``"matrix1"``]`
> 
> `Out[59]: <Range [Book2]Sheet1!$A$1:$B$2>`
> 
> `In``[``60``]:``# 如果您通过"sheet1"对象访问名称集合，``# 它仅包含该工作表范围内的名称``sheet1``.``names`
> 
> `Out[60]: [<Name 'Sheet1!matrix2': =Sheet1!$B$10:$E$11>]`
> 
> `In``[``61``]:``# 如果您通过"book"对象访问名称集合，``# 它包含所有名称，包括书籍和工作表范围``book``.``names`
> 
> `Out[61]: [<Name 'matrix1': =Sheet1!$A$1:$B$2>, <Name 'Sheet1!matrix2':           =Sheet1!$B$10:$E$11>]`
> 
> `In``[``62``]:``# 名称具有各种方法和属性。例如，您可以获取相应的范围对象。``book``.``names``[``"matrix1"``]``.``refers_to_range`
> 
> `Out[62]: <Range [Book2]Sheet1!$A$1:$B$2>`
> 
> `In``[``63``]:``# 如果您想要为常量或公式分配名称，请使用"add"方法``book``.``names``.``add``(``"EURUSD"``,``"=1.1151"``)`
> 
> `Out[63]: <Name 'EURUSD': =1.1151>`

查看通过公式 > 名称管理器打开的 Excel 中生成的定义名称（见 [Figure 9-3](#filepos1400210)）。请注意，macOS 上的 Excel 没有名称管理器，而是转到公式 > 定义名称，在那里你将看到现有的名称。

![](images/00014.jpg)

图 9-3\. 在 xlwings 添加了几个定义名称后的 Excel 名称管理器

现在，您知道如何使用 Excel 工作簿的最常用组件。这意味着我们可以再次从 [Chapter 7](index_split_019.html#filepos863345) 看看报告案例研究：让我们看看当我们引入 xlwings 时会发生什么变化！

Case Study (Re-Revisited): Excel Reporting

能够通过 xlwings 真正编辑 Excel 文件使我们能够处理模板文件，无论其多么复杂或存储在何种格式中，都将完全保留，例如，您可以轻松编辑 xlsb 文件，这是当前所有之前章节中的写入包都不支持的情况。当您查看配套存储库中的 sales_report_openpxyl.py 时，您将看到在准备 `summary` DataFrame 后，我们需要编写将近四十行代码来创建一个图表并使用 OpenPyXL 样式化一个 DataFrame。而使用 xlwings，您只需六行代码即可实现相同效果，如 [Example 9-1](#filepos1403182) 所示。能够处理 Excel 模板中的格式将为您节省大量工作。然而，这也是有代价的：xlwings 需要安装 Excel 才能运行——如果您需要在自己的机器上偶尔创建这些报告，这通常是可以接受的，但如果您试图作为 Web 应用程序的一部分在服务器上创建报告，则可能不太理想。

首先，您需要确保您的 Microsoft Office 许可证覆盖了服务器上的安装，其次，Excel 并不适用于无人值守自动化，这意味着您可能会遇到稳定性问题，尤其是在短时间内需要生成大量报告时。话虽如此，我见过不止一个客户成功地做到了这一点，因此，如果由于某种原因不能使用写入包，将 xlwings 运行在服务器上可能是一个值得探索的选择。只需确保通过 `app = xw.App()` 在新的 Excel 实例中运行每个脚本，以规避典型的稳定性问题。

您可以在附属存储库中的 sales_report_xlwings.py 中找到完整的 xlwings 脚本（前半部分与我们使用的 OpenPyXL 和 XlsxWriter 相同）。它也是一个完美的示例，展示了如何将读取包与 xlwings 结合使用：尽管 pandas（通过 OpenPyXL 和 xlrd）在从磁盘读取多个文件时更快，但 xlwings 更容易填充预格式化的模板。

示例 9-1\. sales_report_xlwings.py（仅第二部分）

`# 打开模板，粘贴数据，调整列宽``# 并调整图表数据源。然后以不同的名称保存。``template``=``xw``.``Book``(``this_dir``/``"xl"``/``"sales_report_template.xlsx"``)``sheet``=``template``.``sheets``[``"Sheet1"``]``sheet``[``"B3"``]``.``value``=``summary``sheet``[``"B3"``]``.``expand``()``.``columns``.``autofit``()``sheet``.``charts``[``"Chart 1"``]``.``set_source_data``(``sheet``[``"B3"``]``.``expand``()[:``-``1``,``:``-``1``])``template``.``save``(``this_dir``/``"sales_report_xlwings.xlsx"``)`

当您在 macOS 上首次运行此脚本（例如通过在 VS Code 中打开并点击“运行文件”按钮），您将再次确认弹出窗口以授予文件系统访问权限，这是本章早些时候已经遇到的内容。

使用格式化的 Excel 模板，你可以非常快速地创建漂亮的 Excel 报告。你还可以使用 `autofit` 等方法，这是写入包（如 writer packages）所不具备的功能，因为它依赖 Excel 应用程序进行的计算：这使得你可以根据单元格内容适当设置它们的宽度和高度。[图 9-4](#filepos1410076) 展示了由 xlwings 生成的销售报告的上部分，其中包括自定义表头以及应用了 `autofit` 方法的列。

当你开始使用 xlwings 不仅仅是填充模板中的几个单元格时，了解其内部机制会对你有所帮助：接下来的部分将深入探讨 xlwings 在幕后的工作原理。

![](images/00073.jpg)

图 9-4\. 基于预格式化模板的销售报告表格

高级 xlwings 主题

本节将向您展示如何使您的 xlwings 代码更高效，并解决缺少功能的问题。不过，要理解这些主题，我们首先需要简要介绍 xlwings 与 Excel 通信的方式。

xlwings 基础知识

xlwings 依赖于其他 Python 包来与操作系统的自动化机制进行通信：

Windows

> > 在 Windows 上，xlwings 依赖于 COM 技术，即组件对象模型。COM 是一种允许两个进程进行通信的标准——在我们的案例中是 Excel 和 Python。xlwings 使用 Python 包 [pywin32](https://oreil.ly/tm7sK) 处理 COM 调用。

macOS

> > 在 macOS 上，xlwings 依赖于 AppleScript。AppleScript 是苹果的脚本语言，用于自动化可脚本化的应用程序——幸运的是，Excel 就是这样一个可脚本化的应用程序。为了运行 AppleScript 命令，xlwings 使用 Python 包 [appscript](https://oreil.ly/tIsDd)。
> > 
> WINDOWS：如何避免僵尸进程
> 
> 在 Windows 上使用 xlwings 时，有时会注意到 Excel 看起来完全关闭了，但是当您打开任务管理器（右键单击 Windows 任务栏，然后选择任务管理器）时，在进程选项卡的背景进程下会看到 Microsoft Excel。如果您没有看到任何选项卡，请首先点击“更多详情”。或者，转到详细信息选项卡，在那里您将看到 Excel 列为“EXCEL.EXE”。要终止僵尸进程，请右键单击相应行，然后选择“结束任务”以强制关闭 Excel。
> 
> 因为这些进程是未终止的不死进程，通常被称为僵尸进程。保留它们会消耗资源，并可能导致不良行为：例如，当您打开新的 Excel 实例时，可能会出现文件被阻塞或加载项未能正确加载的情况。Excel 有时无法正常关闭的原因在于只有在没有 COM 引用（例如 xlwings 的 `app` 对象形式）时，进程才能被终止。通常，在终止 Python 解释器后，您会遇到 Excel 僵尸进程，因为这会阻止它正确清理 COM 引用。在 Anaconda Prompt 中考虑以下示例：
> 
> `(base)>` `python` `>>>` `import xlwings as xw` `>>>` `app = xw.App()`
> 
> 一旦新的 Excel 实例正在运行，请通过 Excel 用户界面再次退出它：虽然 Excel 关闭了，但任务管理器中的 Excel 进程将继续运行。如果您通过运行 `quit()` 或使用 Ctrl+Z 快捷键来正确关闭 Python 会话，Excel 进程最终会被关闭。然而，如果您在关闭 Excel 之前杀死 Anaconda Prompt，您会注意到该进程作为僵尸进程存在。如果在运行 Jupyter 服务器并在其中一个 Jupyter 笔记本单元格中保持了 xlwings 的 `app` 对象时杀死 Anaconda Prompt，情况也是如此。为了最小化出现 Excel 僵尸进程的可能性，这里有几个建议：
> 
+   > > > > 从 Python 中运行 `app.quit()` 而不是手动关闭 Excel。这样可以确保引用被正确清理。
+   > > > > 
+   > > > > 当你使用xlwings时，不要关闭交互式Python会话，例如，如果你在Anaconda Prompt上运行Python REPL，请通过运行`quit()`或使用Ctrl+Z快捷键来正确关闭Python解释器。当你使用Jupyter笔记本时，通过在网页界面上点击退出来关闭服务器。
+   > > > > 
+   > > > > 在交互式Python会话中，避免直接使用`app`对象是有帮助的，例如，可以使用`xw.Book()`代替`myapp.books.add()`。即使Python进程被终止，这样做也应该能正确地终止Excel。

现在你对xlwings的基础技术有了了解，让我们看看如何加快慢脚本的速度！

提高性能

为了保持xlwings脚本的性能，有几种策略：最重要的是尽量减少跨应用程序调用。使用原始值可能是另一种选择，最后，设置正确的`app`属性也可能有所帮助。让我们逐个讨论这些选项！

尽量减少跨应用程序调用

至关重要的是要知道，从Python到Excel的每个跨应用程序调用都是“昂贵的”，即很慢。因此，应该尽可能减少此类调用。最简单的方法是通过读取和写入整个Excel范围而不是遍历单个单元格来实现这一点。在以下示例中，我们首先通过遍历每个单元格，然后通过一次调用处理整个范围，读取和写入150个单元格：

> `In``[``64``]:``# 添加一个新工作表并写入150个值``# 以便有点东西可以操作``sheet2``=``book``.``sheets``.``add``()``sheet2``[``"A1"``]``.``value``=``np``.``arange``(``150``)``.``reshape``(``30``,``5``)`
> 
> `In``[``65``]:``%%``time``# 这进行了150次跨应用程序调用``for``cell``in``sheet2``[``"A1:E30"``]:``cell``.``value``+=``1`
> 
> `Wall time: 909 ms`
> 
> `In``[``66``]:``%%``time``# 这只进行了两次跨应用程序调用``values``=``sheet2``[``"A1:E30"``]``.``options``(``np``.``array``)``.``value``sheet2``[``"A1"``]``.``value``=``values``+``1`
> 
> `Wall time: 97.2 ms`

在macOS上，这些数字甚至更加极端，第二个选项比我的机器上的第一个选项快大约50倍。

原始值

xlwings 主要设计用于方便使用，而不是速度。但是，如果处理大型单元格范围，可能会遇到可以通过跳过 xlwings 数据清理步骤来节省时间的情况：例如，在读写数据时，xlwings 会遍历每个值，以在 Windows 和 macOS 之间对齐数据类型。通过在 `options` 方法中使用字符串 `raw` 作为转换器，可以跳过此步骤。尽管这应该使所有操作更快，但除非在 Windows 上写入大数组，否则差异可能不显著。但是，使用原始值意味着你不能再直接使用 DataFrame 进行工作。相反，你需要将你的值提供为嵌套的列表或元组。此外，你还需要提供写入范围的完整地址——仅提供左上角的单元格不再足够：

> `In``[``67``]:``# 使用原始值时，必须提供完整的目标范围，sheet["A35"] 不再有效``sheet1``[``"A35:B36"``]``.``options``(``"raw"``)``.``value``=``[[``1``,``2``],``[``3``,``4``]]`

应用程序属性

根据工作簿的内容，更改 `app` 对象的属性也可以帮助加快代码运行速度。通常，你需要查看以下属性（`myapp` 是 xlwings 的 `app` 对象）：

+   > > > > `myapp.screen_updating = False`
+   > > > > 
+   > > > > `myapp.calculation = "manual"`
+   > > > > 
+   > > > > `myapp.display_alerts = False`

在脚本末尾，确保将属性设置回它们的原始状态。如果你在 Windows 上，通过 `xw.App(visible=False)` 在隐藏的 Excel 实例中运行脚本，可能还会稍微提高性能。

现在你知道如何控制性能了，让我们看看如何扩展 xlwings 的功能。

如何解决缺失功能

xlwings 为最常用的 Excel 命令提供了 Pythonic 接口，并使其在 Windows 和 macOS 上都能正常工作。然而，Excel 对象模型中有许多方法和属性目前尚未被 xlwings 原生支持，但并非没有办法！xlwings 通过在任何 xlwings 对象上使用 `api` 属性，让你可以访问 Windows 上的 pywin32 对象和 macOS 上的 appscript 对象。这样一来，你就可以访问整个 Excel 对象模型，但也失去了跨平台兼容性。例如，假设你想清除单元格的格式。下面是如何操作：

+   > > > > 检查 xlwings `range` 对象上是否有可用的方法，例如，在 Jupyter notebook 中在 `range` 对象的末尾加上点后使用 Tab 键，通过运行 `dir(sheet["A1"])` 或搜索 [xlwings API 参考](https://oreil.ly/EiXBc)。在 VS Code 中，可用方法应自动显示在工具提示中。
+   > > > > 
+   > > > > 如果所需功能不可用，请使用 `api` 属性获取底层对象：在 Windows 上，`sheet["A1"].api` 将给出一个 pywin32 对象，在 macOS 上，将得到一个 appscript 对象。
+   > > > > 
+   > > > > 查看 [Excel VBA 参考](https://oreil.ly/UILPo) 中的 Excel 对象模型。要清除范围的格式，您将最终进入 [Range.ClearFormats](https://oreil.ly/kcEsw)。
+   > > > > 
+   > > > > 在 Windows 上，在大多数情况下，您可以直接使用 VBA 方法或属性与您的 `api` 对象。如果是方法，请确保在 Python 中加上括号：`sheet["A1"].api.ClearFormats()`。如果您在 macOS 上进行此操作，则更复杂，因为 appscript 使用的语法可能很难猜测。您最好的方法是查看作为 [xlwings 源代码](https://oreil.ly/YSS0Y) 一部分的开发者指南。然而，清除单元格格式很容易：只需按照 Python 的语法规则使用小写字符和下划线处理方法名称：`sheet["A1"].api.clear_formats()`。

如果您需要确保 `ClearFormats` 在两个平台上都能正常工作，可以按以下方式执行（`darwin` 是 macOS 的核心，并由 `sys.platform` 用作其名称）：

> `import``sys``if``sys``.``platform``.``startswith``(``"darwin"``):``sheet``[``"A10"``]``.``api``.``clear_formats``()``elif``sys``.``platform``.``startswith``(``"win"``):``sheet``[``"A10"``]``.``api``.``ClearFormats``()`

无论如何，值得在 xlwings 的 [GitHub 仓库](https://oreil.ly/kFkD0) 上提一个问题，以便在将来的版本中包含该功能。

结论

本章向您介绍了 Excel 自动化的概念：通过 xlwings，您可以使用 Python 完成传统上在 VBA 中完成的任务。我们了解了 Excel 对象模型以及 xlwings 如何允许您与其组件如 `sheet` 和 `range` 对象交互。掌握了这些知识，我们回到了第 [7 章](index_split_019.html#filepos863345) 的报告案例研究，并使用 xlwings 填写了一个预先格式化的报告模板；这展示了您可以在读取器包和 xlwings 并行使用的情况。我们还了解了 xlwings 底层使用的库，以了解如何改进性能并解决缺少功能的问题。我最喜欢的 xlwings 功能是它在 macOS 和 Windows 上同样出色。这更令人兴奋，因为 macOS 上的 Power Query 还没有 Windows 版本的所有功能：无论缺少什么功能，您都应该能够轻松用 pandas 和 xlwings 的组合替代它。

现在您已经了解了 xlwings 的基础知识，可以准备好进入下一章了：在那里，我们将迈出下一步，并从 Excel 本身调用 xlwings 脚本，使您能够构建由 Python 驱动的 Excel 工具。

> [1  ](index_split_024.html#filepos1238292) 在 Windows 上，您至少需要 Excel 2007，在 macOS 上，您至少需要 Excel 2016。或者，您可以安装 Excel 的桌面版，这是 Microsoft 365 订阅的一部分。查看您的订阅以获取有关如何执行此操作的详细信息。
> 
> [2  ](index_split_024.html#filepos1248734) 注意，xlwings 0.22.0 引入了 `xw.load` 函数，它类似于 `xw.view`，但工作方向相反：它允许您轻松将 Excel 范围加载到 Jupyter 笔记本中作为 pandas DataFrame，详见 [文档](https://oreil.ly/x7sTR)。
> 
> [3  ](index_split_024.html#filepos1253391) 有关单独的 Excel 实例以及其重要性的更多信息，请参见 [“什么是 Excel 实例，以及为什么这很重要？”](https://oreil.ly/L2FDT)。
> 
> [4  ](#filepos1367425) 另一个流行的集合是 `tables`。要使用它们，至少需要 xlwings 0.21.0；请参阅 [文档](https://oreil.ly/H2Imd)。
> 
> [5  ](#filepos1390175) 带有公式的定义名称也用于 lambda 函数，这是一种在不使用 VBA 或 JavaScript 的情况下定义用户自定义函数的新方法，微软在 2020 年 12 月宣布为 Microsoft 365 订阅用户的新功能。
