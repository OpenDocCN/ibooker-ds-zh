# 第四章\. 在Python中处理基于文件和基于Feed的数据

在[第三章](ch03.html#chapter3)中，我们着重讨论了对数据质量贡献巨大的许多特征——从数据*完整性*、一致性和清晰度到数据*适应性*的可靠性、有效性和代表性。我们讨论了清理和标准化数据的必要性，以及通过与其他数据集的结合来增强数据的必要性。但在实践中，我们如何实现这些目标呢？

很显然，要评估数据集的*质量*，首先需要审查其内容，但这有时说起来比做起来更容易。几十年来，数据整理一直是一项高度专业化的追求，导致公司和组织创建了各种不同（有时是专有的）数字数据格式，以满足其特定需求。通常，这些格式带有自己的文件扩展名——你可能见过其中的一些：*xls*、*csv*、*dbf* 和 *spss* 都是通常与“数据”文件相关联的文件格式。^([1](ch04.html#idm45143426969488)) 尽管它们的具体结构和细节各不相同，但所有这些格式都可以被描述为*基于文件*的——也就是说，它们包含（或多或少）静态文件中的历史数据，可以从数据库下载、通过同事的电子邮件发送，或者通过文件共享站点访问。最重要的是，基于文件的数据集在今天打开和一周、一个月或一年后打开时，大部分情况下会包含相同的信息。

如今，这些基于文件的格式与过去20年中伴随实时网络服务而出现的数据格式和接口形成鲜明对比。网络数据今天涵盖从新闻到天气监测再到社交媒体站点的一切，这些Feed风格的数据源具有其独特的格式和结构。像*xml*、*json* 和 *rss* 这样的扩展表示这种实时数据类型，通常需要通过专门的应用程序接口或API访问。与基于文件的格式不同，通过API访问同一网络数据位置或“端点”将始终显示出当前可用的最*新*数据——而这些数据可能在几天、几小时甚至几秒钟内发生变化。

当然，这些并不是完美的区分。许多组织（特别是政府部门）提供可以下载的基于文件的数据，但当源数据更新时，会覆盖这些文件，并使用相同的名称创建新的文件。同时，Feed风格的数据格式*可以*被下载并保存以供将来参考——但在线的源位置通常不提供访问旧版本的权限。尽管每种数据格式类别有时会有非常不寻常的用途，但在大多数情况下，您可以利用基于文件和基于Feed的数据格式之间的高级别区别，来帮助选择适合特定数据整理项目的最合适的数据源。

如何知道你需要基于文件还是基于源的数据？在许多情况下，你没有选择权。例如，社交媒体公司通过其 API 提供对其数据源的即时访问，但通常不提供回顾性数据。其他类型的数据——特别是那些自其他来源综合或在发布前经过大量审查的数据——更有可能以基于文件的格式提供。如果你确实可以在基于文件和基于源的格式之间进行选择，那么你的选择将取决于你的数据整理问题的性质：如果关键在于拥有最近的可用数据，那么可能基于源的格式更可取。但是，如果你关注的是趋势，那么基于文件的数据，更有可能包含随时间收集的信息，可能是你的最佳选择。尽管如此，即使两种格式都可用，也不能保证它们包含相同的字段，这可能再次为你做出决定。

在本章中，我们将通过实际示例，从几种最常见的基于文件和基于源的数据格式中整理数据，目标是使它们更容易审查、清洗、增强和分析。我们还将查看一些可能需要出于必要性而处理的更难处理的数据格式。在这些过程中，我们将大量依赖 Python 社区为这些目的开发的出色的各种库，包括用于处理从电子表格到图像等各种内容的专业库和程序。到我们完成时，你将具备处理各种数据整理项目所需的技能和示例脚本，为你的下一个数据整理项目铺平道路！

# 结构化数据与非结构化数据

在我们深入编写代码和整理数据之前，我想简要讨论另一个可能影响数据整理项目方向（和速度）的数据源的关键属性——处理*结构化*数据与*非结构化*数据。

大多数数据整理项目的目标是产生见解，并且通常是使用数据做出更好的决策。但决策是时间敏感的，因此我们与数据的工作也需要权衡取舍：我们可能不会等待“完美”的数据集，而是可能结合两个或三个不太完美的数据集，以便建立我们正在调查的现象的有效近似，或者我们可能寻找具有共同标识符（例如邮政编码）的数据集，即使这意味着我们需要稍后推导出真正感兴趣的特定维度结构（比如街区）。只要我们能在不牺牲太多数据质量的情况下获得这些效益，提高我们数据工作的及时性也可以增加其影响力。

使我们的数据处理更有效的最简单方法之一是寻找易于Python和其他计算工具访问和理解的数据格式。尽管计算机视觉、自然语言处理和机器学习的进步使计算机能够分析几乎不考虑其底层结构或格式的数据变得更加容易，但事实上，*结构化*、*机器可读*的数据仍然——或许不足为奇地——是最直接的数据类型。事实上，虽然从访谈到图像再到书籍文本都可以用作数据源，但当我们讨论“数据”时，我们通常想到的更多是结构化的、数值化的数据。

*结构化*数据是任何已按某种方式组织和分类的数据类型，通常以记录和字段的形式存在。在基于文件的格式中，通常是行和列；在基于数据源的格式中，它们通常是（本质上是）对象列表或*字典*。

相比之下，*非结构化*数据可能由不同数据类型的混合组成，包括文本、数字，甚至照片或插图。例如，杂志或小说的内容，或歌曲的波形通常被认为是非结构化数据。

如果你现在在想，“等等，小说有结构啊！章节呢？”那么恭喜你：你已经像数据处理者一样思考了。我们可以通过收集关于世界的信息并对其应用结构来创建几乎任何事物的数据。[^4] 事实上，这就是*所有*数据的生成方式：我们通过文件和数据源访问的数据集都是某人关于如何收集和组织信息的决定的产物。换句话说，有多种方式可以组织信息，但所选择的结构影响了其分析方法。这就是为什么认为数据可以某种方式“客观”有点荒谬；毕竟，它是（固有主观的）人类选择的产物。

例如，尝试进行这个小实验：想象一下如何组织你的某个收藏（可以是音乐、书籍、游戏或茶叶的各种品种——你说了算）。现在问问朋友他们如何组织他们自己的这类收藏物品。你们的方式一样吗？哪种“更好”？再问问其他人，甚至第三个人。虽然你可能会发现你和朋友们用于组织音乐收藏等系统之间的相似之处，但我会非常惊讶，如果你们中有任何两个人做法完全相同的话。事实上，你可能会发现每个人都有点不同的方式，但*也*坚信自己的方式是“最好”的。而且确实如此，对*他们*来说是最好的。

如果这让您想起了我们在[“如何？以及为谁？”](ch03.html#how_for_whom)中的讨论，那并非巧合，因为您的数据整理问题和努力的结果最终将是——你猜对了！——另一个数据集，它将反映*您*的兴趣和优先级。它也将是有结构和组织的，这使得以某些方式处理它比其他方式更容易。但这里的要点不是任何给定的方式是对还是错，而是每个选择都涉及权衡。识别和承认这些权衡是诚实和负责任地使用数据的关键部分。

因此，在使用*结构化*数据时的一个关键权衡是，这要求依赖他人在组织基础信息时的判断和优先考量。显然，如果这些数据是按照一个开放透明的流程，并涉及到合格的专家进行结构化的，那么这可能是一件好事——甚至是一件伟大的事情！像这样经过深思熟虑的数据结构可以让我们对一个我们可能完全不了解的主题有早期的洞察。另一方面，也存在着我们可能会继承他人的偏见或设计不良选择的可能性。

当然，*非结构化*数据给予了我们完全的自由，可以将信息组织成最适合我们需求的数据结构。不足为奇的是，这要求*我们*负责参与一个强大的数据质量过程，这可能既复杂又耗时。

我们如何事先知道特定数据集是结构化的还是非结构化的？在这种情况下，文件扩展名肯定可以帮助我们。基于提要的数据格式始终具有至少*某种*结构，即使它们包含“自由文本”块，如社交媒体帖子。因此，如果您看到文件扩展名*.json*、*.xml*、*.rss*或*.atom*，这些数据至少有某种记录和字段结构，正如我们将在[“基于提要的数据——网络驱动的实时更新”](#feed_based_data)中探讨的那样。以*.csv*、*.tsv*、*.txt*、*.xls(x)*或*.ods*结尾的基于文件的数据往往遵循表格类型、行和列的结构，正如我们将在下一节中看到的那样。而真正的非结构化数据，则最有可能以*.doc(x)*或*.pdf*的形式出现。

现在，我们对可能遇到的不同类型数据源有了很好的掌握，甚至对如何定位它们有了一些了解，让我们开始处理吧！

# 使用结构化数据

自数字计算的早期以来，*表格*一直是结构化数据的最常见方式之一。即使在今天，许多最常见且易于处理的数据格式仍然不过是表格或表格的集合。事实上，在[第二章](ch02.html#chapter2)中，我们已经使用了一种非常常见的表格类型数据格式：*.csv*或逗号分隔值格式。

## 基于文件、表格类型数据——从定界到实现

一般来说，你通常会遇到的所有表格类型数据格式都是所谓的*分隔*文件的示例：每个数据记录都在自己的行或行上，数据值的字段或列之间的边界由特定的文本字符指示或*分隔*。通常，文件中使用的*分隔符*的指示被纳入到数据集的文件扩展名中。例如，*.csv*文件扩展名代表*逗号分隔值*，因为这些文件使用逗号字符（`,`）作为分隔符；*.tsv*文件扩展名代表*制表符分隔值*，因为数据列是由制表符分隔的。以下是常见与分隔数据相关的文件扩展名列表：

*.csv*

*逗号分隔值*文件是你可能会遇到的最常见的表格类型结构化数据文件之一。几乎任何处理表格数据的软件系统（例如政府或公司数据系统、电子表格程序，甚至专门的商业数据程序）都可以将数据输出为*.csv*，正如我们在[第二章](ch02.html#chapter2)中看到的，Python中有方便的库可以轻松处理这种数据类型。

*.tsv*

*制表符分隔值*文件已经存在很长一段时间，但是描述性的*.tsv*扩展名直到最近才变得普遍。虽然数据提供商通常不解释为什么选择一个分隔符而不是另一个分隔符，但是对于需要包含逗号的值的数据集，例如邮政地址，制表符分隔文件可能更常见。

*.txt*

具有此扩展名的结构化数据文件通常是伪装成*.tsv*文件的文件；旧的数据系统通常使用*.txt*扩展名标记制表符分隔的数据。正如您将在接下来的示例中看到的，最好在编写任何代码之前使用基本文本程序（或类似Atom的代码编辑器）打开和查看*任何*您想要处理的数据文件，因为查看文件内容是了解您正在处理的分隔符的唯一可靠方法。

*.xls(x)*

这是使用Microsoft Excel生成的电子表格的文件扩展名。因为这些文件除了公式、格式和其他简单分隔文件无法复制的特性之外，还可以包含多个“工作表”，所以它们需要更多的内存来存储相同数量的数据。它们还具有其他限制（如只能处理一定数量的行），这可能会对数据集的完整性产生影响。

*.ods*

*开放文档电子表格*文件是由许多开源软件套件（如[LibreOffice](https://libreoffice.org)和[OpenOffice](https://openoffice.org/download/index.html)）生成的电子表格的默认扩展名，具有类似于*.xls(x)*文件的限制和功能。

在我们深入研究如何在Python中处理这些文件类型之前，值得花一点时间考虑一下何时我们可能*想要*处理表格类型数据以及何时找到它。

### 何时处理表格类型数据

大多数情况下，我们在源数据格式方面并没有太多选择。事实上，我们需要进行数据整理的主要原因之一是因为我们拥有的数据不完全符合我们的需求。尽管如此，了解您希望能够处理的数据格式仍然是有价值的，这样您可以利用它来指导您对初始数据的搜索。

在 [“结构化数据与非结构化数据”](#structured_vs_unstructured) 中，我们讨论了结构化数据的优缺点，现在我们知道，表格类型数据是最古老和最常见的机器可读数据形式之一。这一历史部分意味着多年来许多形式的源数据已经被塞进表格中，尽管它们可能并不一定适合于类似表格的表示。然而，这种格式对于回答关于时间趋势和模式的问题特别有用。例如，在我们的 Citi Bike 练习中，来自 [第二章](ch02.html#chapter2) 的“顾客”与“订阅者”在一个月内骑行 Citi Bike 的次数。如果我们愿意，我们可以对每个可用的 Citi Bike 骑行月份执行相同的计算，以了解该比例随时间的任何模式。

当然，表格类型的数据通常不适合实时数据或者每个观察结果不包含相同可能值的数据。这类数据通常更适合我们在 [“基于Feed的数据—网络驱动的实时更新”](#feed_based_data) 中讨论的基于feed的数据格式。

### 在哪里找到表格类型数据

由于绝大多数机器可读数据仍然以表格类型数据格式存在，因此这是最容易定位的数据格式之一。电子表格在各个学科中很常见，并且许多政府和商业信息系统依赖于能够以此方式组织数据的软件。无论何时你从专家或组织请求数据，你可能都会得到表格类型的格式。这同样适用于几乎所有在线开放数据门户和数据共享站点。正如我们在 [“智能搜索特定数据类型”](#smart_searching) 中讨论的，即使是通过搜索引擎，你也可以找到表格类型数据（和其他特定文件格式），只要你知道如何查找。

## 用 Python 整理表格类型数据

为了帮助说明在 Python 中处理表格类型数据有多简单，我们将逐步介绍如何从本节提到的所有文件类型中读取数据，再加上一些其他类型，只是为了确保。虽然在后面的章节中，我们将看看如何进行更多的数据清理、转换和数据质量评估，但我们目前的重点仅仅是访问每种数据文件中的数据并使用 Python 与之交互。

### 从 CSV 中读取数据

如果你没有在 [Chapter 2](ch02.html#chapter2) 中跟随进行，这里是一个从 *.csv* 文件中读取数据的复习，使用了来自 Citi Bike 数据集的示例（[Example 4-1](#csv_parsing)）。和往常一样，在我的脚本顶部的注释中，我包含了程序正在做的描述以及任何源文件的链接。由于我们之前已经使用过这种数据格式，所以现在我们只关心打印出前几行数据以查看它们的样子。

##### Example 4-1\. csv_parsing.py

```py
# a simple example of reading data from a .csv file with Python
# using the "csv" library.
# the source data was sampled from the Citi Bike system data:
# https://drive.google.com/file/d/17b461NhSjf_akFWvjgNXQfqgh9iFxCu_/
# which can be found here:
# https://s3.amazonaws.com/tripdata/index.html

# import the `csv` library ![1](assets/1.png)
import csv

# open the `202009CitibikeTripdataExample.csv` file in read ("r") mode
# this file should be in the same folder as our Python script or notebook
source_file = open("202009CitibikeTripdataExample.csv","r") ![2](assets/2.png)

# pass our `source_file` as an ingredient to the `csv` library's
# DictReader "recipe".
# store the result in a variable called `citibike_reader`
citibike_reader = csv.DictReader(source_file)

# the DictReader method has added some useful information to our data,
# like a `fieldnames` property that lets us access all the values
# in the first or "header" row
print(citibike_reader.fieldnames) ![3](assets/3.png)

# let's just print out the first 5 rows
for i in range(0,5): ![4](assets/4.png)
    print (next(citibike_reader))
```

[![1](assets/1.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO1-1)

这是我们处理表格类型数据时的得力工具库。

[![2](assets/2.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO1-2)

`open()` 是一个内置函数，接受文件名和“模式”作为参数。在这个例子中，目标文件（`202009CitibikeTripdataExample.csv`）应该与我们的 Python 脚本或笔记本位于同一个文件夹中。模式的值可以是 `r` 表示“读取”，也可以是 `w` 表示“写入”。

[![3](assets/3.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO1-3)

通过打印出 `citibike_reader.fieldnames` 的值，我们可以看到“User Type”列的确切标签是 `usertype`。

[![4](assets/4.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO1-4)

`range()` 函数提供了一种执行某个代码片段特定次数的方法，从第一个参数的值开始，直到第二个参数的值之前结束。例如，下面缩进的代码将执行五次，遍历 `i` 的值为 `0`、`1`、`2`、`3` 和 `4`。有关 `range()` 函数的更多信息，请参见 [“添加迭代器：range 函数”](#add_iterators)。

运行后的输出应该是这样的：

```py
['tripduration', 'starttime', 'StartDate', 'stoptime', 'start station id',
'start station name', 'start station latitude', 'start station longitude', 'end
station id', 'end station name', 'end station latitude', 'end station
longitude', 'bikeid', 'usertype', 'birth year', 'gender']
{'tripduration': '4225', 'starttime': '2020-09-01 00:00:01.0430', 'StartDate':
'2020-09-01', 'stoptime': '2020-09-01 01:10:26.6350', 'start station id':
'3508', 'start station name': 'St Nicholas Ave & Manhattan Ave', 'start station
latitude': '40.809725', 'start station longitude': '-73.953149', 'end station
id': '116', 'end station name': 'W 17 St & 8 Ave', 'end station latitude': '40.
74177603', 'end station longitude': '-74.00149746', 'bikeid': '44317',
'usertype': 'Customer', 'birth year': '1979', 'gender': '1'}
 ...
{'tripduration': '1193', 'starttime': '2020-09-01 00:00:12.2020', 'StartDate':
'2020-09-01', 'stoptime': '2020-09-01 00:20:05.5470', 'start station id':
'3081', 'start station name': 'Graham Ave & Grand St', 'start station
latitude': '40.711863', 'start station longitude': '-73.944024', 'end station
id': '3048', 'end station name': 'Putnam Ave & Nostrand Ave', 'end station
latitude': '40.68402', 'end station longitude': '-73.94977', 'bikeid': '26396',
'usertype': 'Customer', 'birth year': '1969', 'gender': '0'}
```

### 从 TSV 和 TXT 文件中读取数据

尽管其名称如此，但 Python 的 *csv* 库基本上是 Python 中处理表格类型数据的一站式解决方案，这要归功于 `DictReader` 函数的 `delimiter` 选项。除非你另有指示，否则 `DictReader` 会假定逗号字符（`,`）是它应该查找的分隔符。然而，覆盖这一假设很容易：你只需在调用函数时指定不同的字符即可。在 [Example 4-2](#tsv_parsing) 中，我们指定了制表符 (`\t`)，但我们也可以轻松地替换为我们喜欢的任何分隔符（或者源文件中出现的分隔符）。

##### Example 4-2\. tsv_parsing.py

```py
# a simple example of reading data from a .tsv file with Python, using
# the `csv` library. The source data was downloaded as a .tsv file
# from Jed Shugerman's Google Sheet on prosecutor politicians: ![1](assets/1.png)
# https://docs.google.com/spreadsheets/d/1E6Z-jZWbrKmit_4lG36oyQ658Ta6Mh25HCOBaz7YVrA

# import the `csv` library
import csv

# open the `ShugermanProsecutorPoliticians-SupremeCourtJustices.tsv` file
# in read ("r") mode.
# this file should be in the same folder as our Python script or notebook
tsv_source_file = open("ShugermanProsecutorPoliticians-SupremeCourtJustices.tsv","r")

# pass our `tsv_source_file` as an ingredient to the csv library's
# DictReader "recipe."
# store the result in a variable called `politicians_reader`
politicians_reader = csv.DictReader(tsv_source_file, delimiter='\t')

# the DictReader method has added some useful information to our data,
# like a `fieldnames` property that lets us access all the values
# in the first or "header" row
print(politicians_reader.fieldnames)

# we'll use the `next()` function to print just the first row of data
print (next(politicians_reader))
```

[![1](assets/1.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO2-1)

此数据集在 Jeremy Singer-Vine 的《Data Is Plural》通讯中列出（[*https://data-is-plural.com*](https://data-is-plural.com)）。

这应该产生类似以下的输出：

```py
['', 'Justice', 'Term Start/End', 'Party', 'State', 'Pres Appt', 'Other Offices
Held', 'Relevant Prosecutorial Background']
{'': '40', 'Justice': 'William Strong', 'Term Start/End': '1870-1880', 'Party':
'D/R', 'State': 'PA', 'Pres Appt': 'Grant', 'Other Offices Held': 'US House,
Supr Court of PA, elect comm for elec of 1876', 'Relevant Prosecutorial
Background': 'lawyer'}
```

尽管*.tsv*文件扩展名如今变得相对常见，但许多由旧数据库生成的*实际上*是制表符分隔的文件可能会以*.txt*文件扩展名的形式传送到您手中。幸运的是，正如前文中所述的那样，在我们指定正确的分隔符的情况下，这并不会影响我们如何处理文件，正如您可以在[示例 4-3](#txt_parsing)中看到的那样。

##### 示例 4-3\. txt_parsing.py

```py
# a simple example of reading data from a .tsv file with Python, using
# the `csv` library. The source data was downloaded as a .tsv file
# from Jed Shugerman's Google Sheet on prosecutor politicians:
# https://docs.google.com/spreadsheets/d/1E6Z-jZWbrKmit_4lG36oyQ658Ta6Mh25HCOBaz7YVrA
# the original .tsv file was renamed with a file extension of .txt

# import the `csv` library
import csv

# open the `ShugermanProsecutorPoliticians-SupremeCourtJustices.txt` file
# in read ("r") mode.
# this file should be in the same folder as our Python script or notebook
txt_source_file = open("ShugermanProsecutorPoliticians-SupremeCourtJustices.txt","r")

# pass our txt_source_file as an ingredient to the csv library's DictReader
# "recipe" and store the result in a variable called `politicians_reader`
# add the "delimiter" parameter and specify the tab character, "\t"
politicians_reader = csv.DictReader(txt_source_file, delimiter='\t') ![1](assets/1.png)

# the DictReader function has added useful information to our data,
# like a label that shows us all the values in the first or "header" row
print(politicians_reader.fieldnames)

# we'll use the `next()` function to print just the first row of data
print (next(politicians_reader))
```

[![1](assets/1.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO3-1)

正如在[“不要留空格！”](ch01.html#no_spaces)中讨论的那样，在我们使用代码时，空白字符必须被*转义*。在这里，我们使用了一个`tab`的转义字符，即`\t`。另一个常见的空白字符代码是`\n`表示`换行`（或者根据您的设备是`\r`表示`回车`）。

如果一切顺利，这个脚本的输出应该与[示例 4-2](#tsv_parsing)中的输出完全相同。

此时您可能会问自己一个问题：“我如何知道我的文件有什么分隔符？”虽然有程序化的方法可以帮助检测这一点，但简单的答案是：看！每当您开始处理（或考虑处理）新数据集时，请首先在您的设备上最基本的文本程序中打开它（任何代码编辑器也是一个可靠的选择）。特别是如果文件很大，使用尽可能简单的程序将使您的设备能够将最大的内存和处理能力用于实际读取数据，从而减少程序挂起或设备崩溃的可能性（同时关闭其他程序和多余的浏览器标签页也会有所帮助）！

虽然我稍后会谈论一些检查*真正*大型文件中小部分的方法，但现在是开始练习评估数据质量的关键技能的时候——所有这些技能都需要审查您的数据并对其做出评判。因此，虽然确实有“自动化”识别数据正确分隔符等任务的方法，但在文本编辑器中用眼睛查看它通常不仅更快更直观，而且还会帮助您更加熟悉数据的其他重要方面。

# 实际数据整理：理解失业情况

我们将用于探索一些复杂表格类型数据格式的基础数据集是关于美国失业情况的数据。为什么选择这个数据集？因为失业以某种方式影响到我们大多数人，近几十年来，美国经历了一些特别高的失业率。美国的失业数据每月由劳工统计局（BLS）发布，尽管它们经常被一般新闻来源报道，但通常被视为对“经济”状况的某种抽象指标。这些数字真正代表的含义很少被深入讨论。

当我2007年首次加入*华尔街日报*时，为探索月度经济指标数据（包括失业率）构建交互式仪表板是我的第一个重要项目。我在这个过程中学到的更有趣的事情之一是每个月并不只计算“一个”失业率，而是*几个*（确切地说是六个）。通常由新闻来源报告的是所谓的“U3”失业率，这是美国劳工统计局描述的：

> 总失业人数，作为民用劳动力的百分比（官方失业率）。

表面上看，这似乎是对失业的一个简单定义：所有合理可能在工作的人中，有多少百分比不在工作？

然而，真实情况稍微复杂一些。什么是“就业”或被计算为“劳动力”的一部分意味着什么？查看不同的失业数据更清楚地说明了“U3”数字没有考虑到的内容。 “U6”失业率的定义如下：

> 总失业人数，加上所有边缘附属于劳动力的人员，再加上因经济原因而只能兼职工作的总人数，作为民用劳动力加上所有边缘附属于劳动力的人员的百分比。

当我们阅读附带的注释时，这个较长的定义开始清晰起来：^([5](ch04.html#idm45143422908112))

> 注意：边缘附属于劳动力的人员是指目前既不工作也不寻找工作，但表示他们想要工作并且有时间，并且在过去12个月内曾经寻找过工作。较少附属的人员是边缘附属的一个子集，他们没有目前寻找工作的原因。经济原因而只能兼职工作的人员是那些希望全职工作并且有时间但不得不接受兼职安排的人员。每年都会随着一月份的数据发布而引入更新的人口控制。

换句话说，如果你想要一份工作（并且在过去一年内曾经寻找过工作），但最近没有寻找过工作——或者你有一份兼职工作，但希望全职工作——那么在U3的定义中你不被正式计算为“失业”。这意味着美国人工作多份工作的经济现实（更可能是女性并且有更多孩子的人）^([6](ch04.html#idm45143422903792))，以及可能的“零工”工作者（最近估计占美国劳动力的30%），^([7](ch04.html#idm45143422901552))并不一定反映在U3数字中。毫不奇怪，U6率通常比U3率每月高出几个百分点。

为了看到这些比率随时间的变化如何，我们可以从圣路易斯联邦储备银行的网站上下载它们，该网站提供了成千上万的经济数据集供下载，包括表格类型的*.xls(x)*文件以及如我们稍后将在[示例 4-12](#xml_parsing)中看到的，还有提供提要型格式。

您可以从 [联邦储备经济数据库 (FRED) 网站](https://fred.stlouisfed.org/series/U6RATE) 下载这些练习的数据。它展示了自上世纪90年代初创建以来的当前 U6 失业率。

要在这张图表上添加 U3 率，请在右上角选择“编辑图表” → “添加线条”。在搜索字段中，输入 **`UNRATE`** 然后在搜索栏下方出现时选择“失业率”。最后，点击“添加系列”。使用右上角的 X 关闭此侧窗口，然后选择“下载”，确保选择第一个选项，即 Excel。^([8](ch04.html#idm45143422892896)) 这将是一个 *.xls* 文件，我们将在最后处理它，因为尽管它仍然广泛可用，但这是一个相对过时的文件格式（自 [2007 年起被 *.xlsx* 取代成为 [Microsoft Excel 电子表格的默认格式](https://en.wikipedia.org/wiki/Microsoft_Excel#File_formats)）。

要获取我们需要的其他文件格式，只需用电子表格程序如 Google Sheets 打开您下载的文件，选择“另存为”，然后选择 *.xlsx*，然后重复该过程选择 *.ods*。现在您应该有以下三个包含相同信息的文件：*fredgraph.xlsx*、*fredgraph.ods* 和 *fredgraph.xls*。^([9](ch04.html#idm45143422886480))

###### 注意

如果你打开了原始的 *fredgraph.xls* 文件，你可能注意到它包含的不仅仅是失业数据；它还包含一些关于数据来源以及 U3 和 U6 失业率定义的头部信息。在分析这些文件中的失业率时，需要进一步将这些元数据与表格类型的数据分开。但是请记住，目前我们的目标只是将所有不同格式的文件转换为 *.csv* 格式。我们将在 [第7章](ch07.html#chapter7) 处理涉及移除这些元数据的数据清洗过程。

## XLSX、ODS 和其他所有格式

大多数情况下，最好避免直接处理保存为 *.xlsx*、*.ods* 和大多数其他非文本表格类型数据格式的数据。如果您只是在探索数据集阶段，我建议您简单地使用您喜欢的电子表格程序打开它们，然后将它们保存为 *.csv* 或 *.tsv* 文件格式，然后再在 Python 中访问它们。这不仅会使它们更容易处理，还可以让您实际查看数据文件的内容并了解其包含的内容。

将 *.xls(x)* 和类似的数据格式重新保存和审查为 *.csv* 或等效的基于文本的文件格式，既能减小文件大小，*又*能让你更好地了解“真实”数据的样子。由于电子表格程序中的格式选项，有时屏幕上看到的内容与实际文件中存储的原始值有很大不同。例如，在电子表格程序中以百分比形式显示的值（例如，10%）实际上可能是小数（.1）。如果你试图基于电子表格中看到的内容而不是像 *.csv* 这样的基于文本的数据格式进行 Python 处理或分析，这可能会导致问题。

但是，肯定会有一些情况，你需要直接使用 Python 访问 *.xls(x)* 和类似的文件类型。^([10](ch04.html#idm45143422842640)) 例如，如果有一个 *.xls* 数据集，你需要定期处理（比如，每个月），每次手动重新保存文件都会变得不必要地耗时。

幸运的是，我们在 [“社区”](ch01.html#community) 中谈到的活跃 Python 社区已经创建了可以轻松处理各种数据格式的库。为了彻底了解这些库如何与更复杂的源数据（以及数据格式）配合工作，以下代码示例读取指定的文件格式，然后创建一个包含相同数据的 *新* *.csv* 文件。

然而，要使用这些库，你首先需要在设备上安装它们，方法是在终端窗口中逐个运行以下命令：^([11](ch04.html#idm45143422837216))

```py
pip install openpyxl
pip install pyexcel-ods
pip install xlrd==2.0.1
```

在以下代码示例中，我们将使用 *openpyxl* 库访问（或*解析*）*.xlsx* 文件，使用 *pyexcel-ods* 库处理 *.ods* 文件，并使用 *xlrd* 库从 *.xls* 文件中读取数据（有关查找和选择 Python 库的更多信息，请参见 [“查找库的地方”](app01.html#where_to_find_libraries)）。

为了更好地说明这些不同文件格式的特殊性，我们将做类似于我们在 [示例 4-3](#txt_parsing) 中所做的事情：我们将获取作为 *.xls* 文件提供的示例数据，并使用电子表格程序将该源文件重新保存为其他格式，创建包含*完全相同数据*的 *.xlsx* 和 *.ods* 文件。在此过程中，我认为你会开始感受到这些非文本格式如何使数据处理过程变得更加（我会说，是不必要地）复杂。

我们将从 *\ref*（[示例 4-4](#xlsx_parsing)）开始，通过一个 *.xlsx* 文件进行工作，使用从 FRED 下载的失业数据的一个版本。这个示例说明了处理基于文本的表格型数据文件和非文本格式之间的首个主要区别之一：由于非文本格式支持多个“工作表”，我们需要在脚本顶部包含一个 `for` 循环，在其中放置用于创建各自输出文件的代码（每个工作表一个文件）。

##### 示例 4-4\. xlsx_parsing.py

```py
# an example of reading data from an .xlsx file with Python, using the "openpyxl"
# library. First, you'll need to pip install the openpyxl library:
# https://pypi.org/project/openpyxl/
# the source data can be composed and downloaded from:
# https://fred.stlouisfed.org/series/U6RATE

# specify the "chapter" you want to import from the "openpyxl" library
# in this case, "load_workbook"
from openpyxl import load_workbook

# import the `csv` library, to create our output file
import csv

# pass our filename as an ingredient to the `openpyxl` library's
# `load_workbook()` "recipe"
# store the result in a variable called `source_workbook`
source_workbook = load_workbook(filename = 'fredgraph.xlsx')

# an .xlsx workbook can have multiple sheets
# print their names here for reference
print(source_workbook.sheetnames) ![1](assets/1.png)

# loop through the worksheets in `source_workbook`
for sheet_num, sheet_name in enumerate(source_workbook.sheetnames): ![2](assets/2.png)

    # create a variable that points to the current worksheet by
    # passing the current value of `sheet_name` to `source_workbook`
    current_sheet = source_workbook[sheet_name]

    # print `sheet_name`, just to see what it is
    print(sheet_name)

    # create an output file called "xlsx_"+sheet_name
    output_file = open("xlsx_"+sheet_name+".csv","w") ![3](assets/3.png)

    # use this csv library's "writer" recipe to easily write rows of data
    # to `output_file`, instead of reading data *from* it
    output_writer = csv.writer(output_file)

    # loop through every row in our sheet
    for row in current_sheet.iter_rows(): ![4](assets/4.png)

        # we'll create an empty list where we'll put the actual
        # values of the cells in each row
        row_cells = [] ![5](assets/5.png)

        # for every cell (or column) in each row....
        for cell in row:

            # let's print what's in here, just to see how the code sees it
            print(cell, cell.value)

            # add the values to the end of our list with the `append()` method
            row_cells.append(cell.value)

        # write our newly (re)constructed data row to the output file
        output_writer.writerow(row_cells) ![6](assets/6.png)

    # officially close the `.csv` file we just wrote all that data to
    output_file.close()
```

[![1](assets/1.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO4-1)

类似于*csv*库的`DictReader()`函数，`openpyxl`的`load_workbook()`函数会向我们的源数据添加属性，例如显示工作簿中所有数据表名称的属性。

[![2](assets/2.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO4-2)

即使我们的示例工作簿仅包含一个工作表，未来可能会有更多。我们将使用`enumerate()`函数，这样我们可以访问迭代器*和*工作表名称。这将帮助我们为每个工作表创建一个*.csv*文件。

[![3](assets/3.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO4-3)

每个`source_workbook`中的工作表都需要其独特命名的输出*.csv*文件。为了生成这些文件，我们将使用名称为`"xlsx_"+sheet_name+".csv"`的新文件进行“打开”，并通过将`w`作为“mode”参数来使其*可写*（直到现在，我们一直使用`r`模式从*.csv*文件中*读取*数据）。

[![4](assets/4.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO4-4)

函数`iter_rows()`专用于*openpyxl*库。在这里，它将`source_workbook`的行转换为可以*迭代*或循环的列表。

[![5](assets/5.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO4-5)

*openpyxl*库将每个数据单元格视为Python [`tuple`数据类型](https://docs.python.org/3/library/stdtypes.html#tuple)。如果我们尝试直接打印`current_sheet`的行，则会得到不太有用的单元格位置，而不是它们包含的数据值。为了解决这个问题，我们将在此循环内再做*另一个*循环，逐个遍历每行中的每个单元格，并将实际的数据值添加到`row_cells`中。

[![6](assets/6.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO4-6)

注意，此代码与示例中`for cell in row`代码左对齐。这意味着它在该循环之外，因此仅在给定行中的所有单元格都已附加到我们的列表之后才会运行。

此脚本还开始展示了，就像两位厨师可能有不同的准备同一道菜的方式一样，库的创建者们可能会在如何（重新）结构化每种源文件类型上做出不同选择，这对我们的代码也有相应的影响。例如，*openpyxl* 库的创建者选择将每个数据单元格的位置标签（例如 `A6`）和其包含的值存储在一个 Python `tuple` 中。这个设计决策导致我们需要第二个 `for` 循环来逐个访问每行数据，因为我们实际上必须逐个访问数据单元格，以构建成为输出 *.csv* 文件中的单行数据的 Python 列表。同样，如果您使用电子表格程序打开由 [示例 4-4](#xlsx_parsing) 中的脚本创建的 *xlsx_FRED Graph.csv* 输出文件，您会看到原始的 *.xls* 文件在 `observation_date` 列中显示的值是 YYYY-MM-DD 格式，但我们的输出文件将这些值显示为 YYYY-MM-DD HH:MM:SS 格式。这是因为 *openpyxl* 的创建者们决定自动将任何类似日期的数据字符串转换为 Python 的 `datetime` 类型。显然，这些选择没有对错之分；我们只需要在编写代码时考虑到它们，以确保不会扭曲或误解源数据。

现在，我们已经处理了数据文件的 *.xlsx* 版本，让我们看看当我们将其解析为 *.ods* 格式时会发生什么，如 [示例 4-5](#ods_parsing) 所示。

##### 示例 4-5\. ods_parsing.py

```py
# an example of reading data from an .ods file with Python, using the
# "pyexcel_ods" library. First, you'll need to pip install the library:
# https://pypi.org/project/pyexcel-ods/

# specify the "chapter" of the "pyexcel_ods" library you want to import,
# in this case, `get_data`
from pyexcel_ods import get_data

# import the `csv` library, to create our output file
import csv

# pass our filename as an ingredient to the `pyexcel_ods` library's
# `get_data()` "recipe"
# store the result in a variable called `source_workbook`
source_workbook = get_data("fredgraph.ods")

# an `.ods` workbook can have multiple sheets
for sheet_name, sheet_data in source_workbook.items(): ![1](assets/1.png)

    # print `sheet_name`, just to see what it is
    print(sheet_name)

    # create "ods_"+sheet_name+".csv" as an output file for the current sheet
    output_file = open("ods_"+sheet_name+".csv","w")

    # use this csv library's "writer" recipe to easily write rows of data
    # to `output_file`, instead of reading data *from* it
    output_writer = csv.writer(output_file)

    # now, we need to loop through every row in our sheet
    for row in sheet_data: ![2](assets/2.png)

        # use the `writerow` recipe to write each `row`
        # directly to our output file
        output_writer.writerow(row) ![3](assets/3.png)

    # officially close the `.csv` file we just wrote all that data to
    output_file.close()
```

[![1](assets/1.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO5-1)

*pyexcel_ods* 库将我们的源数据转换为 Python 的 `OrderedDict` 数据类型。然后，相关的 `items()` 方法允许我们访问每个工作表的名称和数据，作为一个可以循环遍历的键/值对。在这种情况下，`sheet_name` 是“键”，整个工作表的数据是“值”。

[![2](assets/2.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO5-2)

在这里，`sheet_data` 已经是一个列表，因此我们可以使用基本的 `for` 循环来遍历该列表。

[![3](assets/3.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO5-3)

此库将工作表中的每一行转换为一个列表，这就是为什么我们可以直接将它们传递给 `writerow()` 方法的原因。

对于 *pyexcel_ods* 库而言，我们输出的 *.csv* 文件的内容更接近我们通过诸如 Google Sheets 这样的电子表格程序打开原始 *fredgraph.xls* 文件时所看到的内容 —— 例如，`observation_date` 字段以简单的 YYYY-MM-DD 格式呈现。此外，库的创建者们决定将每行的值视为列表，这使得我们可以直接将每条记录写入输出文件，而无需创建任何额外的循环或列表。

最后，让我们看看当我们使用 *xlrd* 库直接解析原始的 *.xls* 文件时会发生什么，如 [示例 4-6](#xls_parsing) 所示。

##### 示例 4-6\. xls_parsing.py

```py
# a simple example of reading data from a .xls file with Python
# using the "xrld" library. First, pip install the xlrd library:
# https://pypi.org/project/xlrd/2.0.1/

# import the "xlrd" library
import xlrd

# import the `csv` library, to create our output file
import csv

# pass our filename as an ingredient to the `xlrd` library's
# `open_workbook()` "recipe"
# store the result in a variable called `source_workbook`
source_workbook = xlrd.open_workbook("fredgraph.xls") ![1](assets/1.png)

# an `.xls` workbook can have multiple sheets
for sheet_name in source_workbook.sheet_names():

    # create a variable that points to the current worksheet by
    # passing the current value of `sheet_name` to the `sheet_by_name` recipe
    current_sheet = source_workbook.sheet_by_name(sheet_name)

    # print `sheet_name`, just to see what it is
    print(sheet_name)

    # create "xls_"+sheet_name+".csv" as an output file for the current sheet
    output_file = open("xls_"+sheet_name+".csv","w")

    # use the `csv` library's "writer" recipe to easily write rows of data
    # to `output_file`, instead of reading data *from* it
    output_writer = csv.writer(output_file)

    # now, we need to loop through every row in our sheet
    for row_num, row in enumerate(current_sheet.get_rows()): ![2](assets/2.png)

        # each row is already a list, but we need to use the `row_value()`
        # method to access them
        # then we can use the `writerow` recipe to write them
        # directly to our output file
        output_writer.writerow(current_sheet.row_values(row_num)) ![3](assets/3.png)

    # officially close the `.csv` file we just wrote all that data to
    output_file.close()
```

[![1](assets/1.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO6-1)

注意，这个结构与我们在使用*csv*库时使用的结构类似。

[![2](assets/2.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO6-2)

函数`get_rows()`是特定于*xlrd*库的；它将我们当前工作表的行转换为一个可以循环遍历的列表。

[![3](assets/3.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO6-3)

在我们的输出文件中将会有一些关于“日期”的怪异之处。我们将看看如何在[“解密Excel日期”](ch07.html#decrypting_excel_dates)中修复这些日期。

这个输出文件中我们将看到一些*严重*的奇怪数值记录在`observation_date`字段中，这反映了正如*xlrd*库的创建者所说的那样：

> Excel电子表格中的日期：实际上，这样的东西是不存在的。你所拥有的是浮点数和虔诚的希望。

因此，从*.xls*文件中获得有用的、人类可读的日期需要一些重要的清理工作，我们将在[“解密Excel日期”](ch07.html#decrypting_excel_dates)中解决这个问题。

正如这些练习所希望展示的那样，通过一些聪明的库和对基本代码配置的一些调整，使用Python快速轻松地处理来自各种表格类型数据格式的数据是可能的。同时，我希望这些示例也说明了为什么几乎总是更倾向于使用基于文本和/或开放源代码的格式，因为它们通常需要更少的“清理”和转换以使它们进入清晰、可用的状态。

## 最后，固定宽度

尽管我在本节的开头没有提到它，但非常古老版本的表格类型数据之一是所谓的“固定宽度”。顾名思义，固定宽度表中的每个数据列包含特定的、预定义的字符数，而且*总是*那么多的字符。这意味着固定宽度文件中的有意义数据通常会被额外的字符填充，例如空格或零。

尽管在当代数据系统中非常罕见，但如果你在处理可能存在几十年历史的政府数据源，仍然可能会遇到固定宽度格式。[^16](ch04.html#idm45143422210096)例如，美国国家海洋和大气管理局（[National Oceanic and Atmospheric Administration](https://noaa.gov/our-history) ，NOAA）的起源可以追溯到19世纪初，通过其[全球历史气候网络](https://ncdc.noaa.gov/data-access/land-based-station-data/land-based-datasets/global-historical-climatology-network-ghcn)在网上免费提供了大量详细的最新天气信息，其中很多是以固定宽度格式发布的。例如，关于气象站的唯一标识符、位置以及它们属于哪个（些）网络的信息存储在[*ghcnd-stations.txt* 文件](https://www1.ncdc.noaa.gov/pub/data/ghcn/daily)中。要解释任何实际的天气数据读数（其中许多也是以固定宽度文件发布的），您需要将气象站数据与天气数据进行交叉引用。

与其他表格类型的数据文件相比，如果没有访问描述文件及其字段组织方式的元数据，使用固定宽度数据可能特别棘手。对于分隔文件，通常可以在文本编辑器中查看文件并以合理的置信水平识别所使用的分隔符。在最坏的情况下，您可以尝试使用不同的分隔符解析文件，看看哪个产生了最佳结果。对于固定宽度文件，特别是对于大文件，如果在您检查的数据样本中某个字段没有数据，很容易意外地将多个数据字段合并在一起。

幸运的是，我们正在使用作为数据源的*ghcnd-stations.txt*文件的元数据*也包含在*NOAA网站的同一文件夹中的*readme.txt*文件中。

在浏览*readme.txt*文件时，我们发现了标题为`IV. FORMAT OF "ghcnd-stations.txt"`的部分，其中包含以下表格：

```py
------------------------------
Variable   Columns   Type
------------------------------
ID            1-11   Character
LATITUDE     13-20   Real
LONGITUDE    22-30   Real
ELEVATION    32-37   Real
STATE        39-40   Character
NAME         42-71   Character
GSN FLAG     73-75   Character
HCN/CRN FLAG 77-79   Character
WMO ID       81-85   Character
------------------------------
```

随后详细描述了每个字段包含或表示的内容，包括单位等信息。由于这个强大的*数据字典*，我们现在不仅知道*ghcnd-stations.txt*文件的组织方式，还知道如何解释它包含的信息。正如我们将在[第6章](ch06.html#chapter6)中看到的，找到（或构建）数据字典是评估或改善数据质量的重要部分。然而，目前，我们可以专注于将这个固定宽度文件转换为*.csv*，如[示例4-7](#fixed_width_parsing)中所详述的那样。

##### 示例 4-7\. fixed_width_parsing.py

```py
# an example of reading data from a fixed-width file with Python.
# the source file for this example comes from NOAA and can be accessed here:
# https://www1.ncdc.noaa.gov/pub/data/ghcn/daily/ghcnd-stations.txt
# the metadata for the file can be found here:
# https://www1.ncdc.noaa.gov/pub/data/ghcn/daily/readme.txt

# import the `csv` library, to create our output file
import csv

filename = "ghcnd-stations"

# reading from a basic text file doesn't require any special libraries
# so we'll just open the file in read format ("r") as usual
source_file = open(filename+".txt", "r")

# the built-in "readlines()" method does just what you'd think:
# it reads in a text file and converts it to a list of lines
stations_list = source_file.readlines()

# create an output file for our transformed data
output_file = open(filename+".csv","w")

# use the `csv` library's "writer" recipe to easily write rows of data
# to `output_file`, instead of reading data *from* it
output_writer = csv.writer(output_file)

# create the header list
headers = ["ID","LATITUDE","LONGITUDE","ELEVATION","STATE","NAME","GSN_FLAG",
           "HCNCRN_FLAG","WMO_ID"] ![1](assets/1.png)

# write our headers to the output file
output_writer.writerow(headers)

# loop through each line of our file (multiple "sheets" are not possible)
for line in stations_list:

    # create an empty list, to which we'll append each set of characters that
    # makes up a given "column" of data
    new_row = []

    # ID: positions 1-11
    new_row.append(line[0:11]) ![2](assets/2.png)

    # LATITUDE: positions 13-20
    new_row.append(line[12:20])

    # LONGITUDE: positions 22-30
    new_row.append(line[21:30])

    # ELEVATION: positions 32-37
    new_row.append(line[31:37])

    # STATE: positions 39-40
    new_row.append(line[38:40])

    # NAME: positions 42-71
    new_row.append(line[41:71])

    # GSN_FLAG: positions 73-75
    new_row.append(line[72:75])

    # HCNCRN_FLAG: positions 77-79
    new_row.append(line[76:79])

    # WMO_ID: positions 81-85
    new_row.append(line[80:85])

    # now all that's left is to use the
    # `writerow` function to write new_row to our output file
    output_writer.writerow(new_row)

# officially close the `.csv` file we just wrote all that data to
output_file.close()
```

[![1](assets/1.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO7-1)

由于文件内部没有我们可以用作列标题的内容，我们必须根据 [*readme.txt* 文件中的信息](https://www1.ncdc.noaa.gov/pub/data/ghcn/daily/readme.txt) “硬编码”它们。请注意，我已经删除了特殊字符，并在空格位置使用下划线，以便在稍后清理和分析数据时最大限度地减少麻烦。

[![2](assets/2.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO7-2)

Python 实际上将文本行视为字符列表，因此我们只需告诉它给出两个编号位置之间的字符。就像 `range()` 函数一样，包括第一个位置的字符，但第二个数字不包括在内。还要记住，Python 从零开始计算列表项（通常称为 *零索引*）。这意味着对于每个条目，第一个数字将比元数据所示的少一个，但右手边的数字将相同。

如果你运行 [示例 4-7](#fixed_width_parsing) 中的脚本，并在电子表格程序中打开你的输出 *.csv* 文件，你会注意到某些列中的值格式不一致。例如，在 `ELEVATION` 列中，带小数点的数字左对齐，而没有小数点的数字右对齐。到底是怎么回事？

再次打开文本编辑器查看文件是有启发性的。尽管我们创建的文件 *技术上* 是逗号分隔的，但我们放入每个新“分隔”列的值仍然包含原始文件中存在的额外空格。因此，我们的新文件仍然看起来相当“固定宽度”。

换句话说——就像我们在 Excel “日期”案例中看到的那样——将我们的文件转换为 *.csv* 文件并不会“自动”在输出文件中生成合理的数据类型。确定每个字段应该具有的数据类型，并清理它们以使其表现得合适，是数据清理过程的一部分，我们将在 [第7章](ch07.html#chapter7) 中讨论。

## 基于 Web 的数据源驱动实时更新

表格类型数据格式的结构非常适合一个已经被过滤、修订并转化为相对整理良好的数字、日期和短字符串集合的世界。然而，随着互联网的兴起，传输大量“自由”文本数据的需求也随之而来，比如新闻故事和社交媒体动态。由于这类数据内容通常包括逗号、句点和引号等影响其语义含义的字符，将其放入传统的分隔格式中将会出现问题。此外，分隔格式的水平偏向（涉及大量左右滚动）与 Web 的垂直滚动约定相矛盾。Feed-based 数据格式已经设计用来解决这些限制。

在高层次上，基于*feed*的数据格式主要有两种：XML和JSON。它们都是文本格式，允许数据提供者定义自己独特的数据结构，使其极其灵活，因此非常适用于互联网连接的网站和平台上发现的各种内容。无论它们位于在线位置还是您在本地保存了一份副本，您都可以通过它们的*.xml*和*.json*文件扩展名一部分来识别它们：

*.xml*

可扩展标记语言涵盖了广泛的文件格式，包括*.rss*、*.atom*，甚至*.html*。作为最通用的标记语言类型，XML非常灵活，也许是最早用于基于网络的数据feed的数据格式。

*.json*

JSON文件比XML文件稍微新一些，但目的类似。总体而言，JSON文件比XML文件描述性较少（因此更短、更简洁）。这意味着它们可以编码几乎与XML文件相同数量的数据，同时占用更少的空间，这对移动网络的速度尤为重要。同样重要的是，JSON文件本质上是JavaScript编程语言中的大型`object`数据类型——这是许多，如果不是大多数，网站和移动应用的基础语言。这意味着解析JSON格式的数据对于使用JavaScript的任何网站或程序来说都非常容易，尤其是与XML相比。幸运的是，JavaScript的`object`数据类型与Python的`dict`数据类型非常相似，这也使得在Python中处理JSON非常简单。

在我们深入讨论如何在Python中处理这些文件类型之前，让我们回顾一下，当我们需要*feed*类型的数据时以及在何处找到它。

### 何时使用*feed*类型数据

从某种意义上说，*feed*类型的数据对于21世纪而言就像20世纪的表格类型数据一样：每天在网络上生成、存储和交换的*feed*类型数据的体积可能比全球所有表格类型数据的总和还要大数百万倍——这主要是因为*feed*类型数据是社交媒体网站、新闻应用程序等一切的动力来源。

从数据处理的角度来看，当你探索的现象是时间敏感的并且经常或者说不可预测地更新时，通常你会想要*feed*类型的数据。典型地，这种类型的数据是响应于人类或自然过程生成的，比如（再次）在社交媒体上发布、发布新闻故事或记录地震。

基于文件的表格类型数据和基于Web的提要类型数据都可以包含历史信息，但正如我们在本章开头讨论的那样，前者通常反映了某一固定时间点的数据。相比之下，后者通常以“逆时间顺序”（最新的首先）组织，首个条目是您访问数据时最近创建的数据记录，而不是预定的发布日期。

### 如何找到提要类型的数据

提要类型的数据几乎完全可以在网上找到，通常位于称为应用程序编程接口（API）*端点*的特殊网址上。我们将在[第5章](ch05.html#chapter5)详细讨论使用API的细节，但现在你只需要知道API端点实际上只是数据页面：你可以使用常规的Web浏览器查看许多页面，但你所看到的只是数据本身。一些API端点甚至会根据您发送给它们的信息返回不同的数据，这正是处理提要类型数据如此灵活的部分原因：通过仅更改代码中的几个单词或值，您可以访问完全不同的数据集！

找到提供提要类型数据的API并不需要太多特殊的搜索策略，因为通常具有API的网站和服务*希望*您找到它们。为什么呢？简单来说，当有人编写使用API的代码时，它（通常）会为提供API的公司带来一些好处，即使这种好处只是更多的公众曝光。例如，在Twitter早期，许多Web开发人员使用Twitter API编写程序，这不仅使平台更加有用，*还*节省了公司理解用户需求并构建的费用和工作量。通过最初免费提供其平台数据的API，Twitter促使了几家公司的诞生，这些公司最终被Twitter收购，尽管还有更多公司在API或其服务条款发生变化时被迫停业。^([17](ch04.html#idm45143421937168)) 这突显了处理任何类型数据时可能出现的特定问题之一，尤其是由盈利公司提供的提要类型数据：数据本身及您访问它的权利随时都可能在没有警告的情况下发生变化。因此，尽管提要类型数据源确实很有价值，但它们在更多方面上也是短暂的。

## 使用Python整理提要类型数据

与表格类型的数据类似，使用Python整理提要类型的数据是可能的，这得益于一些有用的库以及像JSON这样的格式已经与Python编程语言中的现有数据类型相似。此外，在接下来的章节中，我们将看到XML和JSON对于我们的目的通常可以互换使用（尽管许多API只会提供其中一种格式的数据）。

### XML：一种标记语言来统一它们

标记语言是计算机中最古老的标准化文档格式之一，旨在创建既能够轻松被人类阅读又能够被机器解析的基于文本的文档。XML在20世纪90年代成为互联网基础设施的越来越重要的一部分，因为各种设备访问和显示基于Web的信息，使得内容（如文本和图像）与格式（如页面布局）的分离变得更加必要。与HTML文档不同——其中内容和格式完全混合——XML文档几乎不指定其信息应如何显示。相反，它的标签和属性充当关于它包含的信息类型以及数据本身的*元数据*。

要了解XML的外观，可以查看[示例 4-8](https://wiki.example.org/sample_xml_document)。

##### 示例 4-8\. 一个样本XML文档

```py
 <?xml version="1.0" encoding="UTF-8"?>
 <mainDoc>
    <!--This is a comment-->
    <elements>
        <element1>This is some text in the document.</element1>
        <element2>This is some other data in the document.</element2>
        <element3 someAttribute="aValue" />
    </elements>
    <someElement anAttribute="anotherValue">More content</someElement>
</mainDoc>
```

这里有几个要点。第一行称为*文档类型*（或`doc-type`）声明；它告诉我们，文档的其余部分应解释为XML（而不是其他任何网络或标记语言，本章稍后将介绍其中一些）。

从以下行开始：

```py
<mainDoc>
```

我们进入文档本身的内容。XML如此灵活的部分原因在于它只包含两种真正的语法结构，这两种都包含在[示例 4-8](https://wiki.example.org/sample_xml_document)中：

标签

标签可以是成对的（如`element1`、`element2`、`someElement`或甚至`mainDoc`），也可以是自闭合的（如`element3`）。标签的名称始终用*尖括号*（`<>`）括起来。对于闭合标签，在开放的尖括号后面紧跟着斜杠（`/`）。成对的标签或自闭合标签也被称为XML的*元素*。

属性

属性只能存在于标签内部（如`anAttribute`）。属性是一种*键/值对*，其中属性名（或*键*）紧跟着等号（`=`），后面是用双引号（`""`）括起来的*值*。

XML元素是包含在开放标签及其匹配闭合标签之间的任何内容（例如，`<elements>`和`</elements>`）。因此，给定的XML元素可能包含许多标签，每个标签也可以包含其他标签。任何标签也可以具有任意数量的属性（包括没有）。自闭合标签也被视为元素之一。

当标签出现在其他标签内部时，*最近打开的标签必须首先关闭*。换句话说，虽然以下是一个合法的XML结构：

```py
 <outerElement>
    <!-- Notice that that the `innerElement1` is closed
 before the `innerElement2` tag is opened -->
    <innerElement1>Some content</innerElement1>
    <innerElement2>More content</innerElement2>
 </outerElement>
```

但这不是：

```py
 <outerElement>
    <!-- NOPE! The `innerElement2` tag was opened
 before the `innerElement1` tag was closed -->
    <innerElement1>Some content<innerElement2>More content</innerElement1>
    </innerElement2>
 </outerElement>
```

*先开后闭*原则也被称为*嵌套*，类似于 [图 2-3](ch02.html#loop_nesting_diagram) 中的“嵌套” `for...in` 循环。^([18](ch04.html#idm45143413262208)) 嵌套在 XML 文档中尤为重要，因为它管理了我们用代码读取或*解析* XML（和其他标记语言）文档的主要机制之一。在 XML 文档中，`doc-type` 声明之后的第一个元素称为*根*元素。如果 XML 文档已经格式化，根元素将始终左对齐，而直接*在*该元素内嵌套的任何元素将向右缩进一级，并称为*子*元素。因此，在 [示例 4-8](#sample_xml_document) 中，`<mainDoc>` 将被视为*根*元素，`<elements>` 将被视为其子元素。同样，`<mainDoc>` 是`<elements>` 的*父*元素（[示例 4-9](#sample_xml_annotated)）。

##### Example 4-9\. 一个带注释的 XML 文档

```py
 <?xml version="1.0" encoding="UTF-8"?>
 <mainDoc>
    <!--`mainDoc` is the *root* element, and `elements` is its *child*-->
    <elements>
        <!-- `elements` is the *parent* of `element1`, `element2`, and
 `element3`, which are *siblings* of one another -->
        <element1>This is text data in the document.</element1>
        <element2>This is some other data in the document.</element2>
        <element3 someAttribute="aValue" />
    </elements>
    <!-- `someElement` is also a *child* of `mainDoc`,
 and a *sibling* of `elements` -->
    <someElement anAttribute="anotherValue">More content</someElement>
</mainDoc>
```

鉴于这种谱系术语的趋势，您可能会想知道：如果`<elements>`是`<element3>`的父级，`<mainDoc>`是`<elements>`的父级，那么`<mainDoc>`是否是`<element3>`的*祖父*？答案是：是的，但也不是。虽然`<mainDoc>`确实是`<element3>`的“父级”的“父级”，但在描述 XML 结构时从不使用术语“祖父”—这可能会很快变得复杂！相反，我们简单地描述这种关系正是：`<mainDoc>`是`<element3>`的*父级*的*父级*。

幸运的是，与 XML 属性相关的复杂性不存在：它们只是键/值对，并且它们*只能*存在于 XML 标签内部，如下所示：

```py
 <element3 someAttribute="aValue" />
```

请注意，等号两侧没有空格，就像元素标签的尖括号和斜杠之间没有空格一样。

就像用英语（或 Python）写作一样，何时使用标签而不是属性来表示特定信息，主要取决于个人偏好和风格。例如，[示例 4-10](#sample_book_XML_1) 和 [示例 4-11](#sample_book_XML_2) 中都包含了关于这本书相同的信息，但每个结构略有不同。

##### Example 4-10\. 示例 XML 书籍数据—更多属性

```py
<aBook>
    <bookURL url="https://www.oreilly.com/library/view/practical-python-data/
 9781492091493"/>
    <bookAbstract>
    There are awesome discoveries to be made and valuable stories to be
    told in datasets--and this book will help you uncover them.
    </bookAbstract>
    <pubDate date="2022-02-01" />
</aBook>
```

##### Example 4-11\. 示例 XML 书籍数据—更多元素

```py
<aBook>
    <bookURL>
        https://www.oreilly.com/library/view/practical-python-data/9781492091493
    </bookURL>
    <bookAbstract>
        There are awesome discoveries to be made and valuable stories to be
        told in datasets--and this book will help you uncover them.
    </bookAbstract>
    <pubDate>2022-02-01</pubDate>
</aBook>
```

这种灵活性意味着 XML 非常适应各种数据源和格式化偏好。同时，它可能很容易创建这样的情况：*每一个*新数据源都需要编写定制代码。显然，这将是一个相当低效的系统，特别是如果许多人和组织发布的数据类型非常相似的情况下。

因此，不足为奇，有许多XML *规范* 定义了格式化特定类型数据的XML文档的附加规则。我在这里突出了一些值得注意的例子，因为这些是您在数据整理工作中可能会遇到的格式。尽管它们具有各种格式名称和文件扩展名，但我们可以使用稍后将在[示例 4-12](#xml_parsing)中详细讨论的相同方法来解析它们：

RSS

简易信息聚合（Really Simple Syndication）是一个XML规范，最早在1990年代末用于新闻信息。*.atom* XML格式也广泛用于这些目的。

KML

Keyhole标记语言（Keyhole Markup Language）是国际上公认的用于编码二维和三维地理数据的标准，并且与Google Earth等工具兼容。

SVG

可缩放矢量图形（Scalable Vector Graphics）是网络图形常用的格式，因其能够在不损失质量的情况下进行缩放。许多常见的图形程序可以输出*.svg*文件，这些文件可以用于网页和其他文档中，能够在各种屏幕尺寸和设备上显示良好。

EPUB

电子出版格式（*.epub*）是广泛接受的数字图书出版开放标准。

正如您从上述列表中可以看到的那样，一些常见的XML格式清楚地表明它们与XML的关系；而其他许多则没有。^([19](ch04.html#idm45143410254992))

现在我们对XML文件的工作原理有了高层次的理解，让我们看看如何使用Python解析一个XML文件。尽管Python有一些用于解析XML的内置工具，但我们将使用一个名为*lxml*的库，这个库特别擅长[快速解析大型XML文件](https://nickjanetakis.com/blog/how-i-used-the-lxml-library-to-parse-xml-20x-faster-in-python#xmltodict-vs-python-s-standard-library-vs-lxml)。尽管我们接下来的示例文件相当小，但请知道，即使我们的数据文件变得更大，我们基本上可以使用相同的代码。

首先，我们将使用从FRED网站下载的“U6”失业数据的XML版本，使用其API。^([20](ch04.html#idm45143410249984)) 在从[Google Drive](https://drive.google.com/file/d/1gPGaDTT9Nn6BtlTtVp7gQLSuocMyIaLU)下载此文件的副本后，您可以使用[示例 4-12](#xml_parsing)中的脚本将源XML转换为*.csv*。首先安装`pip install`：

```py
pip install lxml
```

##### 示例 4-12\. xml_parsing.py

```py
# an example of reading data from an .xml file with Python, using the "lxml"
# library.
# first, you'll need to pip install the lxml library:
# https://pypi.org/project/lxml/
# a helpful tutorial can be found here: https://lxml.de/tutorial.html
# the data used here is an instance of
# https://api.stlouisfed.org/fred/series/observations?series_id=U6RATE& \
# api_key=YOUR_API_KEY_HERE

# specify the "chapter" of the `lxml` library you want to import,
# in this case, `etree`, which stands for "ElementTree"
from lxml import etree

# import the `csv` library, to create our output file
import csv

# choose a filename
filename = "U6_FRED_data" ![1](assets/1.png)

# open our data file in read format, using "rb" as the "mode"
xml_source_file = open(filename+".xml","rb") ![2](assets/2.png)

# pass our xml_source_file as an ingredient to the `lxml` library's
# `etree.parse()` method and store the result in a variable called `xml_doc`
xml_doc = etree.parse(xml_source_file)

# start by getting the current xml document's "root" element
document_root = xml_doc.getroot() ![3](assets/3.png)

# let's print it out to see what it looks like
print(etree.tostring(document_root)) ![4](assets/4.png)

# confirm that `document_root` is a well-formed XML element
if etree.iselement(document_root):

    # create our output file, naming it "xml_"+filename+".csv
    output_file = open("xml_"+filename+".csv","w")

    # use the `csv` library's "writer" recipe to easily write rows of data
    # to `output_file`, instead of reading data *from* it
    output_writer = csv.writer(output_file)

    # grab the first element of our xml document (using `document_root[0]`)
    # and write its attribute keys as column headers to our output file
    output_writer.writerow(document_root[0].attrib.keys()) ![5](assets/5.png)

    # now, we need to loop through every element in our XML file
      for child in document_root: ![6](assets/6.png)

        # now we'll use the `.values()` method to get each element's values
        # as a list and then use that directly with the `writerow` recipe
        output_writer.writerow(child.attrib.values())

    # officially close the `.csv` file we just wrote all that data to
    output_file.close()
```

[![1](assets/1.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO8-1)

在本例中，数据文件中没有任何内容（如工作表名称），我们可以将其作为文件名使用，因此我们将自己创建一个，并用它来加载源数据并标记输出文件。

[![2](assets/2.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO8-2)

我撒了谎！我们一直在使用`open()`函数的“模式”值假定我们希望将源文件解释为*文本*。但因为*lxml*库期望字节数据而不是文本，我们将使用`rb`（“读取字节”）作为“模式”。

[![3](assets/3.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO8-3)

有很多格式错误的 XML！为了确保看起来像好的 XML 实际上确实是好的，我们将检索当前 XML 文档的“根”元素，并确保它有效。

[![4](assets/4.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO8-4)

因为我们的 XML 当前存储为字节数据，所以我们需要使用`etree.tostring()`方法来将其视为文本。

[![5](assets/5.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO8-5)

多亏了*lxml*，我们文档中的每个 XML 元素（或“节点”）都有一个名为`attrib`的属性，其数据类型是 Python 字典（`dict`）。使用[`.keys()`方法](https://docs.python.org/3/library/stdtypes.html#typesmapping)会返回我们 XML 元素所有属性键的列表。由于源文件中的所有元素都是相同的，我们可以使用第一个元素的键来创建输出文件的“标题行”。

[![6](assets/6.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO8-6)

*lxml*库会将 XML 元素[转换为列表](https://lxml.de/tutorial.html#elements-are-lists)，因此我们可以使用简单的`for...in`循环遍历文档中的元素。

事实上，我们的失业数据的 XML 版本结构非常简单：它只是一个元素列表，*所有*我们想要访问的值都存储为属性。因此，我们能够从每个元素中提取属性值作为列表，并用一行代码直接写入到我们的*.csv*文件中。

当然，有许多时候我们会想从更复杂的 XML 格式中提取数据，特别是像 RSS 或 Atom 这样的格式。为了看看如何处理稍微复杂一点的东西，在[示例 4-13](#bbc_example)中，我们将解析 BBC 的科学与环境故事的 RSS 源，你可以从我的[Google Drive](https://drive.google.com/file/d/1zOaksshLfmXxLTipoOjTTnuO6PsVQgg2)下载副本。

##### 示例 4-13\. rss_parsing.py

```py
# an example of reading data from an .xml file with Python, using the "lxml"
# library.
# first, you'll need to pip install the lxml library:
# https://pypi.org/project/lxml/
# the data used here is an instance of
# http://feeds.bbci.co.uk/news/science_and_environment/rss.xml

# specify the "chapter" of the `lxml` library you want to import,
# in this case, `etree`, which stands for "ElementTree"
from lxml import etree

# import the `csv` library, to create our output file
import csv

# choose a filename, for simplicity
filename = "BBC News - Science & Environment XML Feed"

# open our data file in read format, using "rb" as the "mode"
xml_source_file = open(filename+".xml","rb")

# pass our xml_source_file as an ingredient to the `lxml` library's
# `etree.parse()` method and store the result in a variable called `xml_doc`
xml_doc = etree.parse(xml_source_file)

# start by getting the current xml document's "root" element
document_root = xml_doc.getroot()

# if the document_root is a well-formed XML element
if etree.iselement(document_root):

    # create our output file, naming it "rss_"+filename+".csv"
    output_file = open("rss_"+filename+".csv","w")

    # use the `csv` library's "writer" recipe to easily write rows of data
    # to `output_file`, instead of reading data *from* it
    output_writer = csv.writer(output_file)

    # document_root[0] is the "channel" element
    main_channel = document_root[0]

    # the `find()` method returns *only* the first instance of the element name
    article_example = main_channel.find('item') ![1](assets/1.png)

    # create an empty list in which to store our future column headers
    tag_list = []

    for child in article_example.iterdescendants(): ![2](assets/2.png)

        # add each tag to our would-be header list
        tag_list.append(child.tag) ![3](assets/3.png)

        # if the current tag has any attributes
        if child.attrib: ![4](assets/4.png)

            # loop through the attribute keys in the tag
            for attribute_name in child.attrib.keys(): ![5](assets/5.png)

                # append the attribute name to our `tag_list` column headers
                tag_list.append(attribute_name)

    # write the contents of `tag_list` to our output file as column headers
    output_writer.writerow(tag_list) ![6](assets/6.png)

    # now we want to grab *every* <item> element in our file
    # so we use the `findall()` method instead of `find()`
    for item in main_channel.findall('item'):

        # empty list for holding our new row's content
        new_row = []

        # now we'll use our list of tags to get the contents of each element
        for tag in tag_list:

            # if there is anything in the element with a given tag name
            if item.findtext(tag):

                # append it to our new row
                new_row.append(item.findtext(tag))

            # otherwise, make sure it's the "isPermaLink" attribute
            elif tag == "isPermaLink":

                # grab its value from the <guid> element
                # and append it to our row
                new_row.append(item.find('guid').get("isPermaLink"))

        # write the new row to our output file!
        output_writer.writerow(new_row)

    # officially close the `.csv` file we just wrote all that data to
    output_file.close()
```

[![1](assets/1.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO9-1)

和往常一样，我们需要在程序处理和视觉审核之间取得平衡。通过查看我们的数据，可以清楚地看到每篇文章的信息都存储在单独的`item`元素中。然而，复制单个标签和属性名称将是耗时且容易出错的，因此我们将遍历*一个*`item`元素，并列出其中所有标签（和属性），然后将其用作输出*.csv*文件的列标题。

[![2](assets/2.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO9-2)

`iterdescendants()` 方法是 *lxml* 库特有的。它仅返回 XML 元素的 *后代*，而 [更常见的 `iter()` 方法则会返回 *元素本身* 及其子元素或“后代”。](https://lxml.de/api.html#iteration)

[![3](assets/3.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO9-3)

使用 `child.tag` 将检索子元素标签名的文本。例如，对于 `` <pubDate>` `` 元素，它将返回 `pubDate`。

[![4](assets/4.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO9-4)

我们的 `<item>` 元素中只有一个标签具有属性，但我们仍然希望将其包含在输出中。

[![5](assets/5.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO9-5)

`keys()` 方法将为我们提供属于标签的属性列表中所有键的列表。确保将其名称作为字符串获取（而不是一个单项列表）。

[![6](assets/6.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO9-6)

整个 `article_example` `for` 循环只是为了构建 `tag_list` —— 但这是值得的！

如您从 [示例 4-13](#bbc_example) 中所见，借助 *lxml* 库，即使在 Python 中解析稍微复杂的 XML 也仍然相对简单。

尽管 XML 仍然是新闻源等一小部分文件类型的流行数据格式，但有许多功能使其不太适合处理现代网络的高容量数据源。

首先，存在尺寸的简单问题。尽管 XML 文件可以非常描述性地描述，减少了对独立数据字典的需求，但是大多数元素同时包含开标签和相应的闭标签（例如，`<item>` 和 `</item>`），这也使得 XML 有些 *冗长*：XML 文档中有很多文本 *不是* 内容。当您的文档有几十甚至几千个元素时，这并不是什么大问题，但是当您尝试处理社交网络上的数百万或数十亿篇帖子时，所有这些冗余文本确实会减慢速度。

第二，虽然将 XML 转换为其他数据格式并不 *难*，但这个过程也不是完全无缝的。*lxml* 库（以及其他一些）使得用 Python 解析 XML 变得非常简单，但是使用像 JavaScript 这样的面向网络的语言执行相同任务则是冗长而繁琐的。考虑到 JavaScript 在网络上的普及程度，一种能够与 JavaScript 无缝配合的数据格式在某个时候的开发是不奇怪的。正如我们将在下一节中看到的那样，作为一种数据格式，XML 的许多局限性都被 *.json* 格式的 *对象* 特性所解决，这在当前是互联网上最流行的 feed 类型数据格式。

### JSON：Web 数据，下一代

从原理上讲，JSON 类似于 XML，因为它使用嵌套将相关信息聚合到记录和字段中。JSON 也相当易读，尽管它不支持注释，这意味着 JSON 数据源可能需要比 XML 文档更健壮的数据字典。

要开始，请看看[示例 4-14](#sample_JSON_document)中的小 JSON 文档。

##### 示例 4-14\. 示例 JSON 文档

```py
{
"author": "Susan E. McGregor",
"book": {
    "bookURL": "https://www.oreilly.com/library/view/practical-python-data/
 9781492091493/",
    "bookAbstract": "There are awesome discoveries to be made and valuable
 stories to be told in datasets--and this book will help you uncover
 them.",
    "pubDate": "2022-02-01"
},
"papers": [{
    "paperURL": "https://www.usenix.org/conference/usenixsecurity15/
 technical-sessions/presentation/mcgregor",
    "paperTitle": "Investigating the computer security practices and needs
 of journalists",
    "pubDate": "2015-08-12"
},
    {
    "paperURL": "https://www.aclweb.org/anthology/W18-5104.pdf",
    "paperTitle": "Predictive embeddings for hate speech detection on
 twitter",
    "pubDate": "2018-10-31"
}
    ]
}
```

像 XML 一样，JSON 文档的语法“规则”非常简单：在 JSON 文档中只有三种不同的数据结构，所有这些都出现在[示例 4-14](#sample_JSON_document)中：

键/值对

从技术上讲，JSON 文档中的每个内容都是键/值对，*键*用引号括起来位于冒号 (`:`) 的左侧，*值*是出现在冒号右侧的内容。请注意，虽然键必须始终是字符串，*值*可以是字符串（如 `author`）、对象（如 `book`）或列表（如 `papers`）。

对象

这些使用一对大括号 (`{}`) 打开和关闭。在[示例 4-14](#sample_JSON_document)中，总共有四个对象：文档本身（由左对齐的大括号指示）、`book` 对象以及 `papers` 列表中的两个未命名对象。

列表

这些由方括号 (`[]`) 包围，只能包含逗号分隔的对象。

虽然 XML 和 JSON 可用于编码相同的数据，但它们在允许的内容方面存在一些显著差异。例如，JSON 文件不包含 `doc-type` 规范，也不能包含注释。此外，尽管 XML 列表有些隐式（任何重复元素都像列表功能），但在 JSON 中，列表必须由方括号 (`[]`) 指定。

最后，尽管 JSON 设计时考虑了 JavaScript，但您可能已经注意到它的结构与 Python 的 `dict` 和 `list` 类型非常相似。这也是使用 Python 和 JavaScript（以及其他一系列语言）解析 JSON 非常简单的原因之一。

要看到这一点有多简单，我们将在[示例 4-15](#json_parsing)中解析与我们在[示例 4-12](#xml_parsing)中相同的数据，但使用 FRED API 提供的 *.json* 格式。您可以从这个 Google Drive 链接下载文件：[*https://drive.google.com/file/d/1Mpb2f5qYgHnKcU1sTxTmhOPHfzIdeBsq/view?usp=sharing*](https://drive.google.com/file/d/1Mpb2f5qYgHnKcU1sTxTmhOPHfzIdeBsq/view?usp=sharing)。

##### 示例 4-15\. json_parsing.py

```py
# a simple example of reading data from a .json file with Python,
# using the built-in "json" library. The data used here is an instance of
# https://api.stlouisfed.org/fred/series/observations?series_id=U6RATE& \
# file_type=json&api_key=YOUR_API_KEY_HERE

# import the `json` library, since that's our source file format
import json

# import the `csv` library, to create our output file
import csv

# choose a filename
filename = "U6_FRED_data"

# open the file in read format ("r") as usual
json_source_file = open(filename+".json","r")

# pass the `json_source_file` as an ingredient to the json library's `load()`
# method and store the result in a variable called `json_data`
json_data = json.load(json_source_file)

# create our output file, naming it "json_"+filename
output_file = open("json_"+filename+".csv","w")

# use the `csv` library's "writer" recipe to easily write rows of data
# to `output_file`, instead of reading data *from* it
output_writer = csv.writer(output_file)

# grab the first element (at position "0"), and use its keys as the column headers
output_writer.writerow(list(json_data["observations"][0].keys())) ![1](assets/1.png)

for obj in json_data["observations"]: ![2](assets/2.png)

    # we'll create an empty list where we'll put the actual values of each object
    obj_values = []

    # for every `key` (which will become a column), in each object
    for key, value in obj.items(): ![3](assets/3.png)

        # let's print what's in here, just to see how the code sees it
        print(key,value)

        # add the values to our list
        obj_values.append(value)

    # now we've got the whole row, write the data to our output file
    output_writer.writerow(obj_values)

# officially close the `.csv` file we just wrote all that data to
output_file.close()
```

[![1](assets/1.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO10-1)

因为 *json* 库将每个对象解释为[字典视图对象](https://docs.python.org/3/library/stdtypes.html#dict-views)，我们需要告诉 Python 使用 `list()` 函数将其转换为常规列表。

[![2](assets/2.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO10-2)

在大多数情况下，找到文档中主 JSON 对象的名称（或“键”）的最简单方法就是查看它。然而，由于 JSON 数据通常在一行上呈现，我们可以通过将其粘贴到 [JSONLint](https://jsonlint.com) 中来更好地理解其结构。这让我们看到我们的目标数据是一个键为 `observations` 的列表。

[![3](assets/3.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO10-3)

由于 *json* 库的工作方式，如果我们试图直接写入行，我们将得到以 `dict` 标记的值，而不是数据值本身。因此，我们需要再做一个循环，逐个遍历每个 `json` 对象中的每个值，并将该值附加到我们的 `obj_values` 列表中。

虽然 JSON 不如 XML *那样*易于阅读，但它具有其他我们已经提到的优点，比如更小的文件大小和更广泛的代码兼容性。同样，虽然 JSON 不如 XML 描述性强，JSON 数据源（通常是 API）通常有相当好的文档记录；这减少了仅仅推断给定键/值对描述的需求。然而，与所有数据处理一样，处理 JSON 格式数据的第一步是尽可能地理解其上下文。

# 处理非结构化数据

正如我们在 [“结构化与非结构化数据”](#structured_vs_unstructured) 中讨论的那样，创建数据的过程取决于向信息引入一些结构；否则，我们无法系统地分析或从中得出意义。尽管后者通常包括大段人工编写的“自由”文本，但表格类型和 feed 类型数据都相对结构化，并且最重要的是，可以被机器读取。

当我们处理非结构化数据时，相反地，我们的工作总是涉及近似值：我们无法确定我们的程序化处理努力是否会返回底层信息的准确解释。这是因为大多数非结构化数据是设计成人类感知和解释的内容的表示。正如我们在 [第 2 章](ch02.html#chapter2) 中讨论的，虽然计算机可以比人类更快更少出错地处理大量数据，但它们仍然可能被无结构数据欺骗，这种数据永远不会愚弄人类，比如将一个 [稍作修改的停止标志误认为是限速标志](https://spectrum.ieee.org/cars-that-think/transportation/sensors/slight-street-sign-modifications-can-fool-machine-learning-algorithms)。自然地，这意味着当处理*不能*被机器读取的数据时，我们总是需要额外的验证和验证——但 Python 仍然可以帮助我们将这些数据整理成更可用的格式。

## 基于图像的文本：访问 PDF 中的数据

便携式文档格式（PDF）是在1990年代初创建的一种机制，用于保持电子文档的视觉完整性——无论是在文本编辑程序中创建还是从印刷材料中捕获。保持文档的视觉外观也意味着，与可机器读取格式（如文字处理文档）不同，难以更改或提取其内容——这是创建从合同的数字版本到正式信函的重要特性。

换句话说，最初设计时，在PDF中处理数据确实有些困难。然而，因为访问印刷文档中的数据是一个共同的问题，光学字符识别（OCR）的工作实际上早在19世纪末就已经开始。即使数字化OCR工具几十年来已经在软件包和在线上广泛可用，因此虽然它们远非完美，但这种文件类型中包含的数据也并非完全无法获取。

### 处理PDF中的文本的时间

通常来说，处理PDF文件是最后的选择（正如我们将在[第五章](ch05.html#chapter5)中看到的那样，网页抓取也应如此）。一般来说，如果可以避免依赖PDF信息，那么就应该这样做。正如前面所述，从PDF中提取信息的过程通常会产生文档内容的*近似*值，因此准确性的校对是基于任何基于*.pdf*的数据处理工作流的不可推卸部分。话虽如此，有大量仅以图像或扫描文档PDF形式可用的信息，而Python是从此类文档中提取相对准确的第一版本的有效方法。

### 如何找到PDF文件

如果您确信所需数据只能在PDF格式中找到，那么您可以（而且应该）使用[“智能搜索特定数据类型”](#smart_searching)中的提示，在线搜索中定位此文件类型。很可能，您会向个人或组织请求信息，他们将以PDF形式提供信息，让您自行处理如何提取所需信息的问题。因此，大多数情况下您不需要寻找PDF文件——很不幸，它们通常会找到您。

## 使用Python处理PDF文件

因为PDF文件可以从可机器阅读的文本（如文字处理文档）和扫描图像生成，有时可以通过程序相对少的错误提取文档的“活”文本。然而，尽管这种方法看似简单，但由于*.pdf*文件可以采用各种难以准确检测的编码生成，因此该方法仍然可能不可靠。因此，虽然这可能是从*.pdf*中提取文本的高准确性方法，但对于任何给定文件而言，它的可行性较低。

由于这个原因，我将在这里专注于使用 OCR 来识别和提取 *.pdf* 文件中的文本。这将需要两个步骤：

1.  将文档页面转换为单独的图像。

1.  对页面图像运行 OCR，提取文本，并将其写入单独的文本文件。

毫不奇怪，为了使所有这些成为可能，我们需要安装相当多的 Python 库。首先，我们将安装一些用于将我们的 *.pdf* 页面转换为图像的库。第一个是一个通用库，叫做 `poppler`，它是使我们的 Python 特定库 `pdf2image` 工作所必需的。我们将使用 `pdf2image` 来（你猜对了！）将我们的 *.pdf* 文件转换为一系列图像：

```py
sudo apt install poppler-utils
```

然后：

```py
pip install pdf2image
```

接下来，我们需要安装执行 OCR 过程的工具。第一个是一个通用库，叫做 [*tesseract-ocr*](https://github.com/tesseract-ocr/tesseract)，它使用机器学习来识别图像中的文本；第二个是依赖于 *tesseract-ocr* 的一个 Python 库，叫做 *pytesseract*：

```py
sudo apt-get install tesseract-ocr
```

然后：

```py
pip install pytesseract
```

最后，我们需要一个 Python 的辅助库，可以执行计算机视觉，以弥合我们的页面图像和我们的 OCR 库之间的差距：

```py
pip install opencv-python
```

哇！如果这看起来像是很多额外的库，要记住，我们这里实际上使用的是*机器学习*，这是一种让很多“人工智能”技术走向前沿的数据科学技术之一。幸运的是，特别是 Tesseract 相对健壮和包容：虽然它最初是由惠普公司在 1980 年代早期开发的专有系统，但在 2005 年开源，目前支持超过 100 种语言——所以也可以放心尝试将 [示例 4-16](#pdf_parsing) 中的解决方案用于非英文文本！

##### 示例 4-16\. pdf_parsing.py

```py
# a basic example of reading data from a .pdf file with Python,
# using `pdf2image` to convert it to images, and then using the
# openCV and `tesseract` libraries to extract the text

# the source data was downloaded from:
# https://files.stlouisfed.org/files/htdocs/publications/page1-econ/2020/12/01/ \
# unemployment-insurance-a-tried-and-true-safety-net_SE.pdf

# the built-in `operating system` or `os` Python library will let us create
# a new folder in which to store our converted images and output text
import os

# we'll import the `convert_from_path` "chapter" of the `pdf2image` library
from pdf2image import convert_from_path

# the built-in `glob`library offers a handy way to loop through all the files
# in a folder that have a certain file extension, for example
import glob

# `cv2` is the actual library name for `openCV`
import cv2

# and of course, we need our Python library for interfacing
# with the tesseract OCR process
import pytesseract

# we'll use the pdf name to name both our generated images and text files
pdf_name = "SafetyNet"

# our source pdf is in the same folder as our Python script
pdf_source_file = pdf_name+".pdf"

# as long as a folder with the same name as the pdf does not already exist
if os.path.isdir(pdf_name) == False:

    # create a new folder with that name
    target_folder = os.mkdir(pdf_name)

# store all the pages of the PDF in a variable
pages = convert_from_path(pdf_source_file, 300) ![1](assets/1.png)

# loop through all the converted pages, enumerating them so that the page
# number can be used to label the resulting images
for page_num, page in enumerate(pages):

    # create unique filenames for each page image, combining the
    # folder name and the page number
    filename = os.path.join(pdf_name,"p"+str(page_num)+".png") ![2](assets/2.png)

    # save the image of the page in system
    page.save(filename, 'PNG')

# next, go through all the files in the folder that end in `.png`
for img_file in glob.glob(os.path.join(pdf_name, '*.png')): ![3](assets/3.png)

    # replace the slash in the image's filename with a dot
    temp_name = img_file.replace("/",".")

    # pull the unique page name (e.g. `p2`) from the `temp_name`
    text_filename = temp_name.split(".")[1] ![4](assets/4.png)

    # now! create a new, writable file, also in our target folder, that
    # has the same name as the image, but is a `.txt` file
    output_file = open(os.path.join(pdf_name,text_filename+".txt"), "w")

    # use the `cv2` library to interpret our image
    img = cv2.imread(img_file)

    # create a new variable to hold the results of using pytesseract's
    # `image_to_string()` function, which will do just that
    converted_text = pytesseract.image_to_string(img)

    # write our extracted text to our output file
    output_file.write(converted_text)

    # close the output file
    output_file.close()
```

[![1](assets/1.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO11-1)

在这里，我们将源文件的路径（在这种情况下，它只是文件名，因为它与我们的脚本位于同一文件夹中）和输出图像的期望每英寸点数（DPI）分辨率传递给 `convert_from_path()` 函数。虽然设置较低的 DPI 会快得多，但质量较差的图像可能会导致明显不准确的 OCR 结果。300 DPI 是标准的“打印”质量分辨率。

[![2](assets/2.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO11-2)

在这里，我们使用 *os* 库的 `.join` 函数将新文件保存到我们的目标文件夹中。我们还必须使用 `str()` 函数将页面号转换为文件名中使用的字符串。

[![3](assets/3.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO11-3)

请注意 `*.png` 可以翻译为“以 .png 结尾的任何文件”。`glob()` 函数创建了一个包含我们存储图像的文件夹中所有文件名的列表（在这种情况下，它的值为 `pdf_name`）。

[![4](assets/4.png)](#co_working_with_file_based_and_feed_based_data_in_python_CO11-4)

字符串操作非常棘手！为了为我们的 OCR 文本文件生成唯一（但匹配的！）文件名，我们需要从 `img_file` 的值中提取一个以 `SafetyNet/` 开头且以 `.png` 结尾的唯一页面名称。因此，我们将斜杠替换为句点，得到类似 `SafetyNet.p1.png` 的内容，然后如果在句点上使用 `split()` *这个*，我们将得到一个列表：`["SafetyNet", "p1", "png"]`。最后，我们可以访问位置为1的“页面名称”。我们需要做所有这些，因为我们不能确定 `glob()` 会首先从图像文件夹中提取 `p1.png`，或者它是否会按顺序提取所有图像。

在大多数情况下，运行此脚本可以满足我们的需求：只需几十行代码，它将一个多页的 PDF 文件首先转换为图像，然后将（大部分）内容写入一系列新的文本文件中。

然而，这种全能方法也有其局限性。将 PDF 转换为图像——或者将图像转换为文本——是我们可能经常需要做的任务之一，但不总是同时进行。换句话说，长远来看，为解决这个问题编写两个*独立的*脚本可能更为实用，并且可以依次运行它们。事实上，通过稍加调整，我们可能可以分解前述脚本，使得我们可以将*任何* PDF 转换为图像或*任何*图像转换为文本，而无需编写*任何*新代码。听起来相当巧妙，对吧？

这个重新思考和重组工作中的代码的过程被称为*代码重构*。在英语写作中，我们会描述这个过程为修订或编辑，而目标都是相同的：使你的工作更简单、更清晰和更有效。与文档编写一样，重构实际上是扩展数据处理工作的另一种重要方式，因为它使得*代码重用*变得更加直接。我们将在[第8章](ch08.html#chapter8)中探讨代码重构和脚本重用的各种策略。

## 使用 Tabula 访问 PDF 表格

如果你查看了前面部分生成的文本文件，你可能已经注意到这些文件中有很多“额外内容”：页码和页眉、换行符以及其他[“废物”](https://en.wikipedia.org/wiki/Cruft)。同时，也有一些关键元素缺失，比如图片和表格。

虽然我们的数据工作不会涉及到分析图像（这是一个更加专业化的领域），但在 PDF 文件中找到包含我们希望处理的数据的表格并不罕见。事实上，在我的家乡领域——新闻业，这个问题非常常见，以至于一群调查记者设计并构建了一个名为[Tabula](https://tabula.technology)的工具，专门用来解决这个问题。

Tabula 并不是一个 Python 库——它实际上是一个独立的软件。要试用它，请[下载安装程序](https://tabula.technology/#download-install)适合您系统的版本；如果您使用的是 Chromebook 或 Linux 机器，您需要下载 *.zip* 文件，并按 *README.txt* 中的说明操作。无论您使用哪种系统，您可能需要先安装 Java 编程库，您可以通过在终端窗口中运行以下命令来完成：

```py
sudo apt install default-jre
```

像我们将在后面章节讨论的一些其他开源工具（如 OpenRefine，在[第二章](ch02.html#chapter2)中用于准备一些示例数据，并在[第十一章](ch11.html#chapter11)中简要介绍），Tabula 在幕后完成其工作（尽管部分工作可在终端窗口中看到），您可以通过 web 浏览器与其交互。这是一种获取更传统图形界面访问权限的方式，同时仍然让您的计算机大部分资源空闲以进行大量数据工作。

# 结论

希望本章中的编程示例已经开始让您了解，凭借精心选择的库和在[第二章](ch02.html#chapter2)中介绍的几个基本的 Python 脚本概念，您可以用相对较少的 Python 代码解决各种各样的数据处理问题。

你可能也注意到，除了我们的 PDF 文本之外，本章中所有练习的输出本质上都是一个 *.csv* 文件。这并非偶然。*.csv* 文件不仅高效且多用途，而且我们几乎需要表格类型的数据来进行几乎任何基本的统计分析或可视化。这并不是说分析非表格数据不可能；事实上，这正是许多当代计算机科学研究（如机器学习）所关注的内容。然而，由于这些系统通常复杂且不透明，它们并不适合我们在这里专注于的数据处理工作。因此，我们将把精力放在能帮助我们理解、解释和传达关于世界的新见解的分析类型上。

最后，在本章中，我们的工作集中在基于文件的数据和预先保存的基于 feed 的数据版本上，而在[第五章](ch05.html#chapter5)中，我们将探讨如何使用 Python 结合 API 和网页抓取来从在线系统中处理数据，必要时甚至直接从网页本身获取数据！

^([1](ch04.html#idm45143426969488-marker)) 与您可能知道的其他一些格式相比，如 *mp4* 或 *png*，它们通常分别与音乐和图像相关联。

^([2](ch04.html#idm45143426940336-marker)) 尽管您很快就会知道如何处理它！

^([3](ch04.html#idm45143423524592-marker)) 实际上，您不需要那么多运气——我们将在“使用 Python 处理 PDF 文档”一节中讨论如何做到这一点。

^([4](ch04.html#idm45143423518064-marker)) 在计算机科学中，“数据”和“信息”这两个术语的使用方式恰好相反： “数据”是收集的关于世界的原始事实，“信息”是组织和结构化这些数据后的有意义的最终产品。然而，近年来，随着“大数据”讨论主导了许多领域，我在此使用的解释方式变得更加普遍，因此为了清晰起见，在本书中我将坚持使用这种解释方式。

^([5](ch04.html#idm45143422908112-marker)) 来自[美国劳工统计局](https://bls.gov/news.release/empsit.t15.htm)。

^([6](ch04.html#idm45143422903792-marker)) “多职工”作者是Stéphane Auray，David L. Fuller和Guillaume Vandenbroucke，发布于2018年12月21日，[*https://research.stlouisfed.org/publications/economic-synopses/2018/12/21/multiple-jobholders*](https://research.stlouisfed.org/publications/economic-synopses/2018/12/21/multiple-jobholders)。

^([7](ch04.html#idm45143422901552-marker)) 参见“改进临时和替代工作安排数据的新建议”，[*https://blogs.bls.gov/blog/tag/contingent-workers*](https://blogs.bls.gov/blog/tag/contingent-workers); “灵活工作的价值：Uber司机的证据”作者是M. Keith Chen等，*Nber Working Paper Series* No. 23296，[*https://nber.org/system/files/working_papers/w23296/w23296.pdf*](https://nber.org/system/files/working_papers/w23296/w23296.pdf)。

^([8](ch04.html#idm45143422892896-marker)) 你也可以在[FRED 网站](https://fredhelp.stlouisfed.org/fred/graphs/customize-a-fred-graph/data-transformation-add-series-to-existing-line)找到相关的操作说明。

^([9](ch04.html#idm45143422886480-marker)) 你也可以直接从[我的Google Drive](https://drive.google.com/drive/u/0/folders/1cU5Tdg_fvrCcwvAAyhMOhpbEcI2fF7sb)下载这些文件的副本。

^([10](ch04.html#idm45143422842640-marker)) 截至本文撰写时，LibreOffice可以处理与Microsoft Excel相同数量的行数（2^(20)），但列数远远不及。虽然Google Sheets可以处理比Excel更多的列，但只能处理约40,000行。

^([11](ch04.html#idm45143422837216-marker)) 截至本文撰写时，所有这些库已经在Google Colab中可用并准备就绪。

^([12](ch04.html#idm45143422318528-marker)) 关于`get_rows()`更多信息，请参阅[*xlrd*文档](https://xlrd.readthedocs.io/en/latest/api.html#xlrd-sheet)。

^([13](ch04.html#idm45143422098736-marker)) 关于这个问题，请参阅[*xlrd*文档](https://xlrd.readthedocs.io/en/latest/dates.html)。

^([14](ch04.html#idm45143422094400-marker)) 作者是Stephen John Machin和Chris Withers，《Excel电子表格中的日期》[*https://xlrd.readthedocs.io/en/latest/dates.html*](https://xlrd.readthedocs.io/en/latest/dates.html)。

^([15](ch04.html#idm45143422084224-marker)) 如果你在文本编辑器中打开前面三个代码示例的输出文件，你会发现开源的*.ods*格式是最简单和最干净的。

^([16](ch04.html#idm45143422210096-marker)) 例如，像[Pennsylvania](https://spotlightpa.org/news/2021/05/pa-unemployment-claims-overhaul-ibm-gsi-benefits-labor-industry)或[Colorado](https://denverpost.com/2021/01/10/colorado-unemployment-benefits-new-claims-system)这样的地方。

^([17](ch04.html#idm45143421937168-marker)) 请参阅Vassili van der Mersch的文章，“Twitter与开发者关系的10年斗争”来自Nordic APIs，详见[Nordic APIs](https://nordicapis.com/twitter-10-year-struggle-with-developer-relations)。

^([18](ch04.html#idm45143413262208-marker)) 与Python代码不同，XML文档在工作时*不*需要正确缩进，但这当然会使它们更易读！

^([19](ch04.html#idm45143410254992-marker)) 有趣的事实：*.xlsx*格式中的第二个*x*实际上指的是XML！

^([20](ch04.html#idm45143410249984-marker)) 同样地，我们将逐步介绍像这样的API的使用，在[第5章](ch05.html#chapter5)中，但是使用这个文档让我们看到不同的数据格式如何影响我们与数据的交互。

^([21](ch04.html#idm45143409767664-marker)) 如果已应用样式表，例如我们在[示例 4-13](#bbc_example)中使用的BBC feed，您可以右键点击页面并选择“查看源代码”以查看“原始”XML。

^([22](ch04.html#idm45143409744752-marker)) 查看更多信息，请参阅[Adobe关于PDF页面](https://acrobat.adobe.com/us/en/acrobat/about-adobe-pdf.html)。

^([23](ch04.html#idm45143409740288-marker)) George Nagy, “文档识别中的颠覆性发展,” *Pattern Recognition Letters* 79 (2016): 106–112, [*https://doi.org/10.1016/j.patrec.2015.11.024*](https://doi.org/10.1016/j.patrec.2015.11.024). 可查看[*https://ecse.rpi.edu/~nagy/PDF_chrono/2016_PRL_Disruptive_asPublished.pdf*](https://ecse.rpi.edu/~nagy/PDF_chrono/2016_PRL_Disruptive_asPublished.pdf)。
