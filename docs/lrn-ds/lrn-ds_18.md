# 第14章 数据交换

数据可以以许多不同的格式存储和交换。到目前为止，我们专注于纯文本分隔和固定宽度格式（[第8章](ch08.html#ch-files)）。在本章中，我们稍微扩展了视野，并介绍了几种其他流行的格式。虽然CSV、TSV和FWF文件有助于将数据组织成数据框架，但其他文件格式可以节省空间或表示更复杂的数据结构。*二进制*文件（*binary*是指不是纯文本格式的格式）可能比纯文本数据源更经济。例如，在本章中，我们介绍了NetCDF，这是一种用于交换大量科学数据的流行二进制格式。其他像JSON和XML这样的纯文本格式可以以更通用和有用于复杂数据结构的方式组织数据。甚至HTML网页，作为XML的近亲，通常包含我们可以抓取并整理以进行分析的有用信息。

在本章中，我们介绍了这些流行格式，描述了它们组织的心智模型，并提供了示例。除了介绍这些格式外，我们还涵盖了在线获取数据的程序化方法。在互联网出现之前，数据科学家必须亲自搬移硬盘驱动器才能与他人共享数据。现在，我们可以自由地从世界各地的计算机中检索数据集。我们介绍了HTTP，这是Web的主要通信协议，以及REST，一种数据传输的架构。通过了解一些关于这些Web技术的知识，我们可以更好地利用Web作为数据来源。

本书始终为数据整理、探索和建模提供了可重复的代码示例。在本章中，我们将讨论如何以可重复的方式获取在线数据。

我们首先介绍NetCDF，然后是JSON。接着，在概述用于数据交换的Web协议之后，我们通过介绍XML、HTML和XPath，一个从这些类型文件中提取内容的工具，来结束本章。

# NetCDF数据

[网络通用数据格式（NetCDF）](https://oreil.ly/_qZGj)是存储面向数组的科学数据的方便高效的格式。该格式的心智模型通过多维值网格表示变量。[图 14-1](#netcdf-diagram)展示了这个概念。例如，降雨量每天在全球各地记录。我们可以想象这些降雨量值排列成一个立方体，其中经度沿着立方体的一边，纬度沿着另一边，日期沿着第三个维度。立方体中的每个单元格存储了特定位置每天记录的降雨量。NetCDF 文件还包含我们称之为*元数据*的有关立方体尺寸的信息。在数据框中，同样的信息会以完全不同的方式组织，对于每次降雨测量，我们需要经度、纬度和日期三个特征。这将意味着重复大量数据。使用 NetCDF 文件，我们不需要为每天重复经度和纬度值，也不需要为每个位置重复日期。

![](assets/leds_1401.png)

###### 图 14-1\. 该图表代表了 NetCDF 数据的模型。数据组织成一个三维数组，其中包含了时间和位置（纬度、经度和时间）上的降雨记录。“X”标记了特定位置在特定日期的一个降雨测量。

NetCDF 除了更紧凑之外，还有其他几个优点：

可扩展的

它提供了对数据子集的高效访问。

可附加的

您可以轻松地添加新数据而无需重新定义结构。

可共享

它是一种独立于编程语言和操作系统的常见格式。

自描述的

源文件包含了数据组织的描述和数据本身。

社区

这些工具是由用户社区提供的。

###### 注意

NetCDF 格式是*二进制*数据的一个例子 —— 这类数据不能像 CSV 这样的文本格式一样直接在文本编辑器如 `vim` 或 Visual Studio Code 中读取。还有许多其他二进制数据格式，包括 SQLite 数据库（来自[第 7 章](ch07.html#ch-sql)）、Feather 和 Apache Arrow。二进制数据格式提供了存储数据集的灵活性，但通常需要特殊工具来打开和读取。

NetCDF变量不仅限于三个维度。例如，我们可以添加海拔到我们的地球科学应用程序中，以便我们在时间、纬度、经度和海拔上记录温度等数据。维度不一定要对应物理维度。气候科学家经常运行几个模型，并将模型号存储在维度中以及模型输出。虽然NetCDF最初是为大气科学家设计的，由大气研究公司(UCAR)开发，但这种格式已经广受欢迎，并且现在在全球数千个教育、研究和政府网站上使用。应用程序已扩展到其他领域，如天文学和物理学，通过[史密森尼/ NASA天体物理数据系统(ADS)](https://oreil.ly/kg9kV)，以及医学成像通过[医学图像NetCDF(MINC)](https://oreil.ly/6t3gJ)。

NetCDF文件有三个基本组件：维度、变量和各种元数据。*变量*包含我们认为的数据，例如降水记录。每个变量都有名称、存储类型和形状，即维度的数量。*维度*组件给出每个维度的名称和网格点的数量。*坐标*提供了其他信息，特别是测量点的位置，例如经度，在这里这些可能是<math><mn>0.0</mn> <mo>,</mo> <mn>0.25</mn> <mo>,</mo> <mn>0.50</mn> <mo>,</mo> <mo>…</mo> <mo>,</mo> <mn>359.75</mn></math>。其他元数据包括*属性*。变量的属性可以包含有关变量的辅助信息，其他属性包含关于文件的全局信息，例如发布数据集的人员、其联系信息以及使用数据的权限。这些全局信息对确保可重复结果至关重要。

以下示例检查了特定NetCDF文件的组件，并演示了如何从变量中提取数据的部分。

[气候数据存储](https://oreil.ly/NAhRW)提供了来自各种气候部门和服务的数据集合。我们访问了他们的网站，并请求了2022年12月两周的温度和总降水量的测量数据。让我们简要检查这些数据的组织结构，如何提取子集，并进行可视化。

数据位于NetCDF文件*CDS_ERA5_22-12.nc*中。让我们首先弄清楚文件有多大：

```py
`from` `pathlib` `import` `Path`
`import` `os`

`file_path` `=` `Path``(``)` `/` `'``data``'` `/` `'``CDS_ERA5_22-12.nc``'`

`kib` `=` `1024`
`size` `=` `os``.``path``.``getsize``(``file_path``)`
`np``.``round``(``size` `/` `kib``*``*``3``)`

```

```py
2.0

```

尽管只有三个变量（总降水量、降雨率、温度）的两周数据，但文件的大小达到了2 GiB！这些气候数据通常会相当庞大。

`xarray`包对处理类似数组的数据非常有用，尤其是NetCDF格式的数据。我们使用它的功能来探索我们气候文件的组件。首先我们打开文件：

```py
`import` `xarray` `as` `xr`

`ds` `=` `xr``.``open_dataset``(``file_path``)`

```

现在让我们检查文件的维度组件：

```py
`ds``.``dims`

```

```py
Frozen(SortedKeysDict({'longitude': 1440, 'latitude': 721, 'time': 408}))

```

就像在 [图 14-1](#netcdf-diagram) 中一样，我们的文件有三个维度：经度、纬度和时间。 每个维度的大小告诉我们有超过 400,000 个数据值单元格（1440 × 721 × 408）。 如果这些数据在数据框中，则数据框将具有 400,000 行，其中包含大量重复的纬度、经度和时间列！ 相反，我们只需要它们的值一次，坐标组件就会给我们提供它们：

```py
`ds``.``coords`

```

```py
Coordinates:
  * longitude  (longitude) float32 0.0 0.25 0.5 0.75 ... 359.0 359.2 359.5 359.8
  * latitude   (latitude) float32 90.0 89.75 89.5 89.25 ... -89.5 -89.75 -90.0
  * time       (time) datetime64[ns] 2022-12-15 ... 2022-12-31T23:00:00

```

我们文件中的每个变量都是三维的。 实际上，一个变量不一定要有所有三个维度，但在我们的示例中确实有：

```py
`ds``.``data_vars`

```

```py
Data variables:
    t2m      (time, latitude, longitude) float32 ...
    lsrr     (time, latitude, longitude) float32 ...
    tp       (time, latitude, longitude) float32 ...

```

变量的元数据提供单位和较长的描述，而源的元数据则为我们提供诸如检索数据的时间等信息：

```py
`ds``.``tp``.``attrs`

```

```py
{'units': 'm', 'long_name': 'Total precipitation'}

```

```py
`ds``.``attrs`

```

```py
{'Conventions': 'CF-1.6',
 'history': '2023-01-19 19:54:37 GMT by grib_to_netcdf-2.25.1: /opt/ecmwf/mars-client/bin/grib_to_netcdf.bin -S param -o /cache/data6/adaptor.mars.internal-1674158060.3800251-17201-13-c46a8ac2-f1b6-4b57-a14e-801c001f7b2b.nc /cache/tmp/c46a8ac2-f1b6-4b57-a14e-801c001f7b2b-adaptor.mars.internal-1674158033.856014-17201-20-tmp.grib'}

```

通过将所有这些信息保存在源文件中，我们不会冒丢失信息或使描述与数据不同步的风险。

就像使用 `pandas` 一样，`xarray` 提供了许多不同的方法来选择要处理的数据部分。 我们展示了两个例子。 首先，我们专注于一个特定的位置，并使用线性图来查看时间内的总降水量：

```py
`plt``.``figure``(``)`
`(``ds``.``sel``(``latitude``=``37.75``,` `longitude``=``237.5``)``.``tp` `*` `100``)``.``plot``(``figsize``=``(``8``,``3``)``)`
`plt``.``xlabel``(``'``'``)`
`plt``.``ylabel``(``'``Total precipitation (cm)``'``)`
`plt``.``show``(``)``;`

```

```py
<Figure size 288x216 with 0 Axes>

```

![](assets/leds_14in01.png)

接下来，我们选择一个日期，2022年12月31日，下午1点，并将纬度和经度缩小到美国大陆范围内，以制作温度地图：

```py
`import` `datetime`
`one_day` `=` `datetime``.``datetime``(``2022``,` `12``,` `31``,` `13``,` `0``,` `0``)`

```

```py
`min_lon``,` `min_lat``,` `max_lon``,` `max_lat` `=` `232``,` `21``,` `300``,` `50`

`mask_lon` `=` `(``ds``.``longitude` `>` `min_lon``)` `&` `(``ds``.``longitude` `<` `max_lon``)`
`mask_lat` `=` `(``ds``.``latitude` `>` `min_lat``)` `&` `(``ds``.``latitude` `<` `max_lat``)`

`ds_oneday_us` `=` `ds``.``sel``(``time``=``one_day``)``.``t2m``.``where``(``mask_lon` `&` `mask_lat``,` `drop``=``True``)`

```

就像对于数据框的 `loc` 一样，`sel` 返回一个新的 `DataArray`，其数据由沿指定维度的索引标签确定，对于本例来说，即日期。 而且就像 `np.where` 一样，`xr.where` 根据提供的逻辑条件返回元素。 我们使用 `drop=True` 来减少数据集的大小。

让我们制作一个温度色彩地图，其中颜色代表温度：

```py
`ds_oneday_us``.``plot``(``figsize``=``(``8``,``4``)``)`

```

![](assets/leds_14in02.png)

从这张地图中，我们可以看出美国的形状、温暖的加勒比海和更冷的山脉。

我们通过关闭文件来结束：

```py
`ds``.``close``(``)`

```

这个对 NetCDF 的简要介绍旨在介绍基本概念。 我们的主要目标是展示其他类型的数据格式存在，并且可以比纯文本读入数据框更具优势。 对于感兴趣的读者，NetCDF 拥有丰富的软件包和功能。 例如，除了 `xarray` 模块之外，NetCDF 文件还可以使用其他 Python 模块（如 [`netCDF4`](https://oreil.ly/UlX_k) 和 [`gdal`](https://oreil.ly/fKeQh)）进行读取。 NetCDF 社区还提供了与 NetCDF 数据交互的命令行工具。 制作可视化和地图的选项包括 `matplotlib`、[`iris`](https://oreil.ly/ozNrI)（建立在 `netCDF4` 之上）和 [`cartopy`](https://oreil.ly/9N7y7)。

接下来我们考虑 JSON 格式，它比 CSV 和 FWF 格式更灵活，可以表示分层数据。

# JSON 数据

JavaScript 对象表示法（JSON）是在 web 上交换数据的流行格式。 这种纯文本格式具有简单灵活的语法，与 Python 字典非常匹配，易于机器解析和人类阅读。

简而言之，JSON有两种主要结构，对象和数组：

对象

像Python的`dict`一样，JSON对象是一个无序的名称-值对集合。这些对包含在大括号中；每个对都格式为`"name":value`，并用逗号分隔。

数组

像Python的`list`一样，JSON数组是一个有序的值集合，包含在方括号中，其中值没有名称，并用逗号分隔。

对象和数组中的值可以是不同类型的，并且可以嵌套。也就是说，数组可以包含对象，反之亦然。原始类型仅限于双引号中的字符串，文本表示中的数字，true或false作为逻辑，以及null。

以下简短的JSON文件演示了所有这些语法特性：

```py
{"lender_id":"matt",  
  "loan_count":23,
  "status":[2,  1,  3],  
  "sponsored":  false,  
  "sponsor_name":  null,
  "lender_dem":{"sex":"m","age":77  }  
}

```

在这里，我们有一个包含六个名称-值对的对象。值是异构的；其中四个是原始值：字符串，数字，逻辑和null。`status`值由三个（有序的）数字数组组成，而`lender_dem`是包含人口统计信息的对象。

内置的`json`包可用于在Python中处理JSON文件。例如，我们可以将这个小文件加载到Python字典中：

```py
`import` `json`
`from` `pathlib` `import` `Path`

`file_path` `=` `Path``(``)` `/` `'``data``'` `/` `'``js_ex``'` `/` `'``ex.json``'`

```

```py
`ex_dict` `=` `json``.``load``(``open``(``file_path``)``)`
`ex_dict`

```

```py
{'lender_id': 'matt',
 'loan_count': 23,
 'status': [2, 1, 3],
 'sponsored': False,
 'sponsor_name': None,
 'lender_dem': {'sex': 'm', 'age': 77}}

```

字典与Kiva文件的格式相匹配。这种格式并不自然地转换为数据框。`json_normalize`方法可以将这种半结构化的JSON数据组织成一个平面表：

```py
`ex_df` `=` `pd``.``json_normalize``(``ex_dict``)`
`ex_df`

```

|   | lender_id | loan_count | status | sponsored | sponsor_name | lender_dem.sex | lender_dem.age |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **0** | matt | 23 | [2, 1, 3] | False | None | m | 77 |

注意，在这个单行数据框中，第三个元素是一个列表，而嵌套对象被转换为两列。

JSON中数据结构的灵活性非常大，这意味着如果我们想要从JSON内容创建数据框，我们需要了解JSON文件中数据的组织方式。我们提供了三种结构，这些结构可以轻松地转换为数据框。

在[第12章](ch12.html#ch-pa)中使用的PurpleAir站点列表是JSON格式的。在那一章中，我们没有注意到格式，只是使用`json`库的`load`方法将文件内容读入字典，然后转换为数据框。在这里，我们简化了该文件，同时保持了一般结构，以便更容易进行检查。

我们首先检查原始文件，然后将其重新组织成另外两种可能用于表示数据框的JSON结构。通过这些示例，我们旨在展示JSON的灵活性。 [图14-2](#json-diagram) 中的图表显示了这三种可能性的表示。

![](assets/leds_1402.png)

###### 图14-2\. JSON格式文件存储数据框的三种不同方法。

图表中最左侧的数据框按行组织。每一行都是具有命名值的对象，其中名称对应于数据框的列名。然后将行收集到数组中。这种结构与原始文件的结构相吻合。在下面的代码中，我们显示文件内容：

```py
{"Header": [
    {"status": "Success",
     "request_time": "2022-12-29T01:48:30-05:00",
     "url": "https://aqs.epa.gov/data/api/dailyData/...",
     "rows": 4
    }
  ],
  "Data": [
    {"site": "0014", "date": "02-27", "aqi": 30},
    {"site": "0014", "date": "02-24", "aqi": 17},
    {"site": "0014", "date": "02-21", "aqi": 60},
    {"site": "0014", "date": "01-15", "aqi": null}
  ]
}

```

我们看到文件包含一个对象，有两个元素，名为`Header`和`Data`。`Data`元素是一个数组，每一行数据框中都有一个元素，正如前面描述的，每个元素都是一个对象。让我们将文件加载到字典中并检查其内容（详见[第八章](ch08.html#ch-files)有关查找文件路径和打印内容的更多信息）：

```py
`from` `pathlib` `import` `Path`
`import` `os`

`epa_file_path` `=` `Path``(``'``data/js_ex/epa_row.json``'``)`

```

```py
`data_row` `=` `json``.``loads``(``epa_file_path``.``read_text``(``)``)`
`data_row`

```

```py
{'Header': [{'status': 'Success',
   'request_time': '2022-12-29T01:48:30-05:00',
   'url': 'https://aqs.epa.gov/data/api/dailyData/...',
   'rows': 4}],
 'Data': [{'site': '0014', 'date': '02-27', 'aqi': 30},
  {'site': '0014', 'date': '02-24', 'aqi': 17},
  {'site': '0014', 'date': '02-21', 'aqi': 60},
  {'site': '0014', 'date': '01-15', 'aqi': None}]}

```

我们可以快速将对象数组转换为数据框，只需进行以下调用：

```py
`pd``.``DataFrame``(``data_row``[``"``Data``"``]``)`

```

|   | 网站 | 日期 | 空气质量指数 |
| --- | --- | --- | --- |
| **0** | 0014 | 02-27 | 30.0 |
| **1** | 0014 | 02-24 | 17.0 |
| **2** | 0014 | 02-21 | 60.0 |
| **3** | 0014 | 01-15 | NaN |

图中的中间图表在[图 14-2](#json-diagram)中采用了列的方法来组织数据。这里列被提供为数组，并收集到一个对象中，名称与数据框的列名相匹配。以下文件展示了该概念：

```py
`epa_col_path` `=` `Path``(``'``data/js_ex/epa_col.json``'``)`
`print``(``epa_col_path``.``read_text``(``)``)`

```

```py
{"site":[ "0014", "0014", "0014", "0014"],
"date":["02-27", "02-24", "02-21", "01-15"],
"aqi":[30,17,60,null]}

```

由于`pd.read_json()`期望这种格式，我们可以直接将文件读入数据框，而不需要先加载到字典中：

```py
`pd``.``read_json``(``epa_col_path``)`

```

|   | 网站 | 日期 | 空气质量指数 |
| --- | --- | --- | --- |
| **0** | 14 | 02-27 | 30.0 |
| **1** | 14 | 02-24 | 17.0 |
| **2** | 14 | 02-21 | 60.0 |
| **3** | 14 | 01-15 | NaN |

最后，我们将数据组织成类似矩阵的结构（图中右侧的图表），并分别为特征提供列名。数据矩阵被组织为一个数组的数组：

```py
{'vars': ['site', 'date', 'aqi'],
 'data': [['0014', '02-27', 30],
  ['0014', '02-24', 17],
  ['0014', '02-21', 60],
  ['0014', '01-15', None]]}

```

我们可以提供`vars`和`data`来创建数据框：

```py
`pd``.``DataFrame``(``data_mat``[``"``data``"``]``,` `columns``=``data_mat``[``"``vars``"``]``)`

```

|   | 网站 | 日期 | 空气质量指数 |
| --- | --- | --- | --- |
| **0** | 0014 | 02-27 | 30.0 |
| **1** | 0014 | 02-24 | 17.0 |
| **2** | 0014 | 02-21 | 60.0 |
| **3** | 0014 | 01-15 | NaN |

我们包含这些示例是为了展示JSON的多功能性。主要的收获是JSON文件可以以不同的方式排列数据，因此我们通常需要在成功将数据读入数据框之前检查文件。JSON文件在存储在网络上的数据中非常常见：本节中的示例是从PurpleAir和Kiva网站下载的文件。尽管在本节中我们手动下载了数据，但我们经常希望一次下载多个数据文件，或者我们希望有一个可靠且可重现的下载记录。在下一节中，我们将介绍HTTP，这是一个协议，让我们能够编写程序自动从网络上下载数据。

# HTTP

HTTP（超文本传输协议）是访问网络资源的通用基础设施。互联网上提供了大量的数据集，通过HTTP我们可以获取这些数据集。

互联网允许计算机彼此通信，而HTTP则对通信进行结构化。 HTTP是一种简单的*请求-响应*协议，其中客户端向服务器提交一个特殊格式的文本*请求*，服务器则返回一个特殊格式的文本*响应*。 客户端可以是Web浏览器或我们的Python会话。

HTTP请求由两部分组成：头部和可选的正文。 头部必须遵循特定的语法。 请求获取在[图14-3](#fig-wiki-1500)中显示的维基百科页面的示例如下所示：

```py
GET /wiki/1500_metres_world_record_progression HTTP/1.1
Host: en.wikipedia.org
User-Agent: curl/7.65.2
Accept: */* 
{blank_line}

```

第一行包含三个信息部分：以请求的方法开头，这是`GET`；其后是我们想要的网页的URL；最后是协议和版本。 接下来的三行每行提供服务器的辅助信息。 这些信息的格式为`名称: 值`。 最后，空行标志着头部的结束。 请注意，在前面的片段中，我们用`{blank_line}`标记了空行；实际消息中，这是一个空行。

![](assets/leds_1403.png)

###### 图14-3\. 维基百科页面截图，显示1500米赛跑的世界纪录数据

客户端的计算机通过互联网将此消息发送给维基百科服务器。 服务器处理请求并发送响应，响应也包括头部和正文。 响应的头部如下所示：

```py
< HTTP/1.1 200 OK
< date: Fri, 24 Feb 2023 00:11:49 GMT
< server: mw1369.eqiad.wmnet
< x-content-type-options: nosniff
< content-language: en
< vary: Accept-Encoding,Cookie,Authorization
< last-modified: Tue, 21 Feb 2023 15:00:46 GMT
< content-type: text/html; charset=UTF-8
...
< content-length: 153912
{blank_line}

```

第一行声明请求成功完成；状态代码为200。 接下来的行提供了客户端的额外信息。 我们大大缩短了这个头部，仅关注告诉我们正文内容为HTML，使用UTF-8编码，并且内容长度为153,912个字符的几个信息。 最后，头部末尾的空行告诉客户端，服务器已经完成发送头部信息，响应正文随后而来。

几乎每个与互联网交互的应用程序都使用HTTP。 例如，如果您在Web浏览器中访问相同的维基百科页面，浏览器会执行与刚刚显示的基本HTTP请求相同的操作。 当它接收到响应时，它会在浏览器窗口中显示正文，该正文看起来像[图14-3](#fig-wiki-1500)中的屏幕截图。

在实践中，我们不会手动编写完整的HTTP请求。 相反，我们使用诸如`requests` Python库之类的工具来为我们构建请求。 以下代码为我们构造了获取[图14-3](#fig-wiki-1500)页面的HTTP请求。 我们只需将URL传递给`requests.get`。 名称中的“get”表示正在使用`GET`方法：

```py
`import` `requests`

`url_1500` `=` `'``https://en.wikipedia.org/wiki/1500_metres_world_record_progression``'`

```

```py
`resp_1500` `=` `requests``.``get``(``url_1500``)`

```

我们可以检查我们的请求状态，以确保服务器成功完成它：

```py
`resp_1500``.``status_code`

```

```py
200

```

我们可以通过对象的属性彻底检查请求和响应。 例如，让我们看一看我们请求的头部中的键值对：

```py
`for` `key` `in` `resp_1500``.``request``.``headers``:`
    `print``(``f``'``{``key``}``:` `{``resp_1500``.``request``.``headers``[``key``]``}``'``)`

```

```py
User-Agent: python-requests/2.25.1
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive

```

虽然我们在函数调用中没有指定任何头信息，但`request.get`为我们提供了一些基本信息。如果需要发送特殊的头信息，我们可以在调用中指定它们。

现在让我们来查看从服务器收到的响应头：

```py
`len``(``resp_1500``.``headers``)`

```

```py
20

```

正如我们之前看到的，响应中有大量的头信息。我们仅显示`date`、`content-type`和`content-length`：

```py
`keys` `=` `[``'``date``'``,` `'``content-type``'``,` `'``content-length``'` `]`
`for` `key` `in` `keys``:`
    `print``(``f``'``{``key``}``:` `{``resp_1500``.``headers``[``key``]``}``'``)`

```

```py
date: Fri, 10 Mar 2023 01:54:13 GMT
content-type: text/html; charset=UTF-8
content-length: 23064

```

最后，我们显示响应体的前几百个字符（整个内容过长，无法在此完整显示）：

```py
`resp_1500``.``text``[``:``600``]`

```

```py
'<!DOCTYPE html>\n<html class="client-nojs vector-feature-language-in-header-enabled vector-feature-language-in-main-page-header-disabled vector-feature-language-alert-in-sidebar-enabled vector-feature-sticky-header-disabled vector-feature-page-tools-disabled vector-feature-page-tools-pinned-disabled vector-feature-toc-pinned-enabled vector-feature-main-menu-pinned-disabled vector-feature-limited-width-enabled vector-feature-limited-width-content-enabled" lang="en" dir="ltr">\n<head>\n<meta charset="UTF-8"/>\n<title>1500 metres world record progression - Wikipedia</title>\n<script>document.documentE'

```

我们确认响应是一个HTML文档，并且包含标题`1500 metres world record progression - Wikipedia`。我们已成功获取了[图14-3](#fig-wiki-1500)中显示的网页。

我们的HTTP请求已成功，服务器返回了状态码`200`。还有数百种其他HTTP状态码。幸运的是，它们被分组到不同的类别中，以便记忆（见[表14-1](#response-codes)）。

表14-1。响应状态码

| Code | Type | Description |
| --- | --- | --- |
| 100s | 信息性 | 需要客户端或服务器进一步输入（100 Continue、102 Processing等）。 |
| 200s | 成功 | 客户端请求成功（200 OK、202 Accepted等）。 |
| 300s | 重定向 | 请求的URL位于其他位置，可能需要用户进一步操作（300 Multiple Choices、301 Moved Permanently等）。 |
| 400s | 客户端错误 | 发生了客户端错误（400 Bad Request、403 Forbidden、404 Not Found等）。 |
| 500s | 服务器错误 | 发生了服务器端错误或服务器无法执行请求（500 Internal Server Error、503 Service Unavailable等）。 |

一个常见的错误代码可能看起来很熟悉，即404，表示我们请求的资源不存在。我们在这里发送这样的请求：

```py
`url` `=` `"``https://www.youtube.com/404errorwow``"`
`bad_loc` `=` `requests``.``get``(``url``)`
`bad_loc``.``status_code`

```

```py
404

```

我们发出的请求是用`GET` HTTP请求获取网页。有四种主要的HTTP请求类型：`GET`、`POST`、`PUT`和`DELETE`。最常用的两种方法是`GET`和`POST`。我们刚刚使用`GET`来获取网页：

```py
`resp_1500``.``request``.``method`

```

```py
'GET'

```

`POST`请求用于将特定信息从客户端发送到服务器。在下一节中，我们将使用`POST`来从Spotify获取数据。

# REST

网络服务越来越多地采用REST（表述性状态转移）架构，供开发人员访问其数据。这些包括像Twitter和Instagram这样的社交媒体平台，像Spotify这样的音乐应用，像Zillow这样的房地产应用，像气候数据存储这样的科学数据源，以及世界银行的政府数据等等。REST背后的基本思想是，每个URL标识一个资源（数据）。

REST 是*无状态*的，意味着服务器不会在连续的请求中记住客户端的状态。REST 的这一方面具有一些优势：服务器和客户端可以理解任何收到的消息，不必查看先前的消息；可以在客户端或服务器端更改代码而不影响服务的操作；访问是可伸缩的、快速的、模块化的和独立的。

在本节中，我们将通过一个示例来从 Spotify 获取数据。

我们的示例遵循[Steven Morse 的博客文章](https://oreil.ly/zI-5z)，我们在一系列请求中使用`POST`和`GET`方法来检索[The Clash](https://www.theclash.com)的歌曲数据。

###### 注意

在实践中，我们不会自己为 Spotify 编写`GET`和`POST`请求。相反，我们会使用[`spotipy`](https://oreil.ly/fPQX0)库，该库具有与[Spotify web API](https://oreil.ly/NH4ZO)交互的功能。尽管如此，数据科学家通常会发现自己想要访问的数据只能通过 REST 获得，而没有 Python 库可用。因此，本节展示了如何从类似 Spotify 的 RESTful 网站获取数据。

通常，REST 应用程序会提供带有如何请求其数据的示例的文档。Spotify 提供了针对想要构建应用程序的开发者的广泛文档，但我们也可以仅仅用来探索数据访问服务。为此，我们需要注册开发者帐号并获取客户端 ID 和密钥，然后在我们的 HTTP 请求中使用它们来识别自己给 Spotify。

注册后，我们可以开始请求数据。此过程分两步：认证和请求资源。

要进行身份验证，我们发出一个 POST 请求，将我们的客户端 ID 和密钥提供给 Web 服务。我们在请求的标头中提供这些信息。作为回报，我们从服务器接收到一个授权我们进行请求的令牌。

我们开始流程并进行身份验证：

```py
`AUTH_URL` `=` `'``https://accounts.spotify.com/api/token``'`

```

```py
`import` `requests`
`auth_response` `=` `requests``.``post``(``AUTH_URL``,` `{`
    `'``grant_type``'``:` `'``client_credentials``'``,`
    `'``client_id``'``:` `CLIENT_ID``,`
    `'``client_secret``'``:` `CLIENT_SECRET``,`
`}``)`

```

我们在 POST 请求的标头中以键值对的形式提供了我们的 ID 和密钥。我们可以检查请求的状态以查看是否成功：

```py
`auth_response``.``status_code`

```

```py
200

```

现在让我们检查响应体中的内容类型：

```py
`auth_response``.``headers``[``'``content-type``'``]`

```

```py
'application/json'

```

响应体包含我们需要在下一步获取数据时使用的令牌。由于此信息格式为 JSON，我们可以检查键并检索令牌：

```py
`auth_response_data` `=` `auth_response``.``json``(``)`
`auth_response_data``.``keys``(``)`

```

```py
dict_keys(['access_token', 'token_type', 'expires_in'])

```

```py
`access_token` `=` `auth_response_data``[``'``access_token``'``]`
`token_type` `=` `auth_response_data``[``'``token_type``'``]`

```

请注意，我们隐藏了我们的 ID 和密钥，以防其他人模仿我们。没有有效的 ID 和密钥，此请求将无法成功。例如，在这里，我们编造了一个 ID 和密钥并尝试进行身份验证：

```py
`bad_ID` `=` `'``0123456789``'`
`bad_SECRET` `=` `'``a1b2c3d4e5``'`

`auth_bad` `=` `requests``.``post``(``AUTH_URL``,` `{`
    `'``grant_type``'``:` `'``client_credentials``'``,`
    `'``client_id``'``:` `bad_ID``,` `'``client_secret``'``:` `bad_SECRET``,`
`}``)`

```

我们检查此“坏”请求的状态：

```py
`auth_bad``.``status_code`

```

```py
400

```

根据[表 14-1](#response-codes)，400 码表示我们发出了一个错误请求。作为一个例子，如果我们花费太多时间进行请求，Spotify 会关闭我们。在撰写本节时，我们遇到了这个问题几次，并收到了以下代码，告诉我们我们的令牌已过期：

```py
res_clash.status_code

401

```

现在进行第二步，让我们获取一些数据。

对 Spotify 的资源可以通过 `GET` 进行请求。其他服务可能需要 POST。请求必须包括我们从 web 服务认证时收到的令牌，我们可以一次又一次地使用。我们将访问令牌传递到我们的 `GET` 请求的头部。我们将名称-值对构造为字典：

```py
`headers` `=` `{``"``Authorization``"``:` `f``"``{``token_type``}`  `{``access_token``}``"``}`

```

开发者 API 告诉我们，艺术家的专辑可在类似于 *https://api.spotify.com/v1/artists/3RGLhK1IP9jnYFH4BRFJBS/albums* 的 URL 上找到，其中 *artists/* 和 */albums* 之间的代码是艺术家的 ID。这个特定的代码是 The Clash 的。有关专辑上音轨的信息可在类似于 *https://api.spotify.com/v1/albums/49kzgMsxHU5CTeb2XmFHjo/tracks* 的 URL 上找到，这里的标识符是专辑的。

如果我们知道艺术家的 ID，我们可以检索其专辑的 ID，进而可以获取关于专辑上音轨的数据。我们的第一步是从 Spotify 的网站获取 The Clash 的 ID：

```py
`artist_id` `=` `'``3RGLhK1IP9jnYFH4BRFJBS``'`

```

我们的第一个数据请求检索了组的专辑。我们使用 `artist_id` 构建 URL，并在头部传递我们的访问令牌：

```py
`BASE_URL` `=` `"``https://api.spotify.com/v1/``"`

`res_clash` `=` `requests``.``get``(`
    `BASE_URL` `+` `"``artists/``"` `+` `artist_id` `+` `"``/albums``"``,`
    `headers``=``headers``,`
    `params``=``{``"``include_groups``"``:` `"``album``"``}``,`
`)`

```

```py
`res_clash``.``status_code`

```

```py
200

```

我们的请求成功了。现在让我们检查响应主体的`content-type`：

```py
`res_clash``.``headers``[``'``content-type``'``]`

```

```py
'application/json; charset=utf-8'

```

返回的资源是 JSON，因此我们可以将其加载到 Python 字典中：

```py
`clash_albums` `=` `res_clash``.``json``(``)`

```

经过一番搜索，我们可以发现专辑信息在 `items` 元素中。第一个专辑的键是：

```py
`clash_albums``[``'``items``'``]``[``0``]``.``keys``(``)`

```

```py
dict_keys(['album_group', 'album_type', 'artists', 'available_markets', 'external_urls', 'href', 'id', 'images', 'name', 'release_date', 'release_date_precision', 'total_tracks', 'type', 'uri'])

```

让我们打印几个专辑的专辑 ID、名称和发行日期：

```py
`for` `album` `in` `clash_albums``[``'``items``'``]``[``:``4``]``:`
    `print``(``'``ID:` `'``,` `album``[``'``id``'``]``,` `'` `'``,` `album``[``'``name``'``]``,` `'``----``'``,` `album``[``'``release_date``'``]``)`

```

```py
ID:  7nL9UERtRQCB5eWEQCINsh   Combat Rock + The People's Hall ---- 2022-05-20
ID:  3un5bLdxz0zKhiZXlmnxWE   Live At Shea Stadium ---- 2008-08-26
ID:  4dMWTj1OkiCKFN5yBMP1vS   Live at Shea Stadium (Remastered) ---- 2008
ID:  1Au9637RH9pXjBv5uS3JpQ   From Here To Eternity Live ---- 1999-10-04

```

我们看到一些专辑是重新混音的，而另一些是现场演出。接下来，我们循环遍历专辑，获取它们的 ID，并为每个专辑请求有关音轨的信息：

```py
`tracks` `=` `[``]`

`for` `album` `in` `clash_albums``[``'``items``'``]``:` 
    `tracks_url` `=` `f``"``{``BASE_URL``}``albums/``{``album``[``'``id``'``]``}``/tracks``"`
    `res_tracks` `=` `requests``.``get``(``tracks_url``,` `headers``=``headers``)`
    `album_tracks` `=` `res_tracks``.``json``(``)``[``'``items``'``]`

    `for` `track` `in` `album_tracks``:`
        `features_url` `=` `f``"``{``BASE_URL``}``audio-features/``{``track``[``'``id``'``]``}``"`
        `res_feat` `=` `requests``.``get``(``features_url``,` `headers``=``headers``)`
        `features` `=` `res_feat``.``json``(``)`

        `features``.``update``(``{`
            `'``track_name``'``:` `track``.``get``(``'``name``'``)``,`
            `'``album_name``'``:` `album``[``'``name``'``]``,`
            `'``release_date``'``:` `album``[``'``release_date``'``]``,`
            `'``album_id``'``:` `album``[``'``id``'``]`
        `}``)`

        `tracks``.``append``(``features``)` 

```

在这些音轨上有超过十几个功能可供探索。让我们以绘制 The Clash 歌曲的舞蹈性和响度为例结束本示例：

![](assets/leds_14in03.png)

本节介绍了 REST API，它提供了程序下载数据的标准化方法。这里展示的示例下载了 JSON 数据。其他时候，来自 REST 请求的数据可能是 XML 格式的。有时我们想要的数据没有 REST API 可用，我们必须从 HTML 中提取数据，这是一种与 XML 类似的格式。接下来我们将描述如何处理这些格式。

# XML、HTML 和 XPath

可扩展标记语言（XML）可以表示各种类型的信息，例如发送到和从 Web 服务传送的数据，包括网页、电子表格、SVG 等可视显示、社交网络结构、像微软的 docx 这样的文字处理文档、数据库等等。对于数据科学家来说，了解 XML 会有所帮助。

尽管它的名称是 XML，但它不是一种语言。相反，它是一个非常通用的结构，我们可以用它来定义表示和组织数据的格式。XML 提供了这些“方言”或词汇表的基本结构和语法。如果你读过或撰写过 HTML，你会认出 XML 的格式。

XML的基本单位是*元素*，也被称为*节点*。一个元素有一个名称，可以有属性、子元素和文本。

下面标注的XML植物目录片段提供了这些部分的示例（此内容改编自[W3Schools](https://oreil.ly/qPa6s)）：

```py
<catalog>                                The topmost node, aka root node.
    <plant>                              The first child of the root node.
        <common>Bloodroot</common>       common is the first child of plant.
        <botanical>Sanguinaria canadensis</botanical>
        <zone>4</zone>                   This zone node has text content: 4.
        <light>Mostly Shady</light>
        <price curr="USD">$2.44</price>  This node has an attribute.
        <availability date="0399"/>      Empty nodes can be collapsed.
    </plant>                             Nodes must be closed. 
    <plant>                              The two plant nodes are siblings.
        <common>Columbine</common>
        <botanical>Aquilegia canadensis</botanical>
        <zone>3</zone>
        <light>Mostly Shady</light>
        <price curr="CAD">$9.37</price>
        <availability date="0199"/>
    </plant>
</catalog>

```

我们为此XML片段添加了缩进以便更容易看到结构。实际文件中不需要缩进。

XML文档是纯文本文件，具有以下语法规则：

+   每个元素都以开始标签开始，例如`<plant>`，并以相同名称的结束标签关闭，例如`</plant>`。

+   XML元素可以包含其他XML元素。

+   XML元素可以是纯文本，例如`<common>Columbine</common>`中的“Columbine”。

+   XML元素可以具有可选的属性。元素`<price curr=`"CAD"`>`具有属性`curr`，其值为`"CAD"`。

+   特殊情况下，当节点没有子节点时，结束标签可以折叠到开始标签中。例如`<availability date="0199"/>`。

当它遵循特定规则时，我们称XML文档为良好格式的文档。其中最重要的规则是：

+   一个根节点包含文档中的所有其他元素。

+   元素正确嵌套；开放节点在其所有子节点周围关闭，不再多余。

+   标签名称区分大小写。

+   属性值采用`name=“value”`格式，可以使用单引号或双引号。

有关文档为良好格式的其他规则。这些与空白、特殊字符、命名约定和重复属性有关。

**XML的分层结构**使其可以表示为树形结构。[图 14-4](#fig-xml-tree)展示了植物目录XML的树形表示。

![](assets/leds_1404.png)

###### 图 14-4\. XML文档的层次结构；浅灰色框表示文本元素，按设计，这些元素不能有子节点。

与JSON类似，XML文档是纯文本。我们可以用纯文本查看器来读取它，对于机器来说读取和创建XML内容也很容易。XML的可扩展性允许内容轻松合并到更高级别的容器文档中，并且可以轻松地与其他应用程序交换。XML还支持二进制数据和任意字符集。

如前所述，HTML看起来很像XML。这不是偶然的，事实上，XHTML是HTML的子集，遵循良好格式XML的规则。让我们回到之前从互联网上检索的维基百科页面的例子，并展示如何使用XML工具从其表格内容创建数据框架。

## 示例：从维基百科抓取赛时

在本章的早些时候，我们使用了一个 HTTP 请求从维基百科检索了 HTML 页面，如[图 14-3](#fig-wiki-1500)所示。这个页面的内容是 HTML 格式的，本质上是 XML 词汇。我们可以利用页面的分层结构和 XML 工具来访问其中一个表格中的数据，并将其整理成数据框。特别是，我们对页面中的第二个表格感兴趣，其中的一部分显示在[图 14-5](#fig-html-table)的截图中。

![](assets/leds_1405.png)

###### 图 14-5\. 网页中包含我们想要提取的数据的第二个表格的截图

在我们处理这个表格之前，我们先快速总结一下基本 HTML 表格的格式。这是一个带有表头和两行三列的表格的 HTML 格式：

```py
<table>
 <tbody>
  <tr>
   <th>A</th><th>B</th><th>C</th> 
  </tr>
  <tr>
   <td>1</td><td>2</td><td>3</td>
  </tr>
  <tr>
   <td>5</td><td>6</td><td>7</td>
  </tr>
 </tbody>
</table>

```

注意表格是如何以`<tr>`元素为行布局的，每行中的每个单元格是包含在`<td>`元素中的文本，用于在表格中显示。

我们的第一个任务是从网页内容中创建一个树结构。为此，我们使用`lxml`库，它提供了访问 C 库`libxml2`来处理 XML 内容的功能。回想一下，`resp_1500`包含了我们请求的响应，页面位于响应体中。我们可以使用`lxml.html`模块中的`fromstring`方法将网页解析为一个分层结构：

```py
`from` `lxml` `import` `html`

`tree_1500` `=` `html``.``fromstring``(``resp_1500``.``content``)`

```

```py
`type``(``tree_1500``)`

```

```py
lxml.html.HtmlElement

```

现在我们可以使用文档的树结构来处理文档。我们可以通过以下搜索找到 HTML 文档中的所有表格：

```py
`tables` `=` `tree_1500``.``xpath``(``'``//table``'``)`
`type``(``tables``)`

```

```py
list

```

```py
`len``(``tables``)`

```

```py
7

```

这个搜索使用了 XPath `//table` 表达式，在文档中的任何位置搜索所有表格节点。

我们在文档中找到了六个表格。如果我们检查网页，包括通过浏览器查看其 HTML 源代码，我们可以发现文档中的第二个表格包含 IAF 时代的时间。这是我们想要的表格。[图 14-5](#fig-html-table)中的截图显示，第一列包含比赛时间，第三列包含名称，第四列包含比赛日期。我们可以依次提取这些信息。我们使用以下 XPath 表达式完成这些操作：

```py
`times` `=` `tree_1500``.``xpath``(``'``//table[3]/tbody/tr/td[1]/b/text()``'``)`
`names` `=` `tree_1500``.``xpath``(``'``//table[3]/tbody/tr/td[3]/a/text()``'``)`
`dates` `=` `tree_1500``.``xpath``(``'``//table[3]/tbody/tr/td[4]/text()``'``)`

```

```py
`type``(``times``[``0``]``)`

```

```py
lxml.etree._ElementUnicodeResult

```

这些返回值的行为类似于列表，但每个值都是树的元素。我们可以将它们转换为字符串：

```py
`date_str` `=` `[``str``(``s``)` `for` `s` `in` `dates``]`
`name_str` `=` `[``str``(``s``)` `for` `s` `in` `names``]`

```

对于时间，我们希望将其转换为秒。函数`get_sec`可以完成这个转换。而我们希望从日期字符串中提取比赛年份：

```py
`def` `get_sec``(``time``)``:`
    `"""convert time into seconds."""`
    `time` `=` `str``(``time``)`
    `time` `=` `time``.``replace``(``"``+``"``,``"``"``)`
    `m``,` `s` `=` `time``.``split``(``'``:``'``)`
    `return` `float``(``m``)` `*` `60` `+` `float``(``s``)`

```

```py
`time_sec` `=` `[``get_sec``(``rt``)` `for` `rt` `in` `times``]`
`race_year` `=` `pd``.``to_datetime``(``date_str``,` `format``=``'``%Y``-``%m``-``%d``\n``'``)``.``year`

```

我们可以创建一个数据框并绘制图表，以展示比赛时间随年份的变化情况：

![](assets/leds_14in04.png)

正如你可能已经注意到的那样，从 HTML 页面中提取数据需要仔细检查源代码，找到我们需要的数字在文档中的位置。我们大量使用 XPath 工具进行提取。它的语言优雅而强大。我们接下来介绍它。

## XPath

当我们处理 XML 文档时，通常希望从中提取数据并将其带入数据框中。XPath 可以在这方面提供帮助。XPath 可以递归地遍历 XML 树以查找元素。例如，在前面的示例中，我们使用表达式 `//table` 定位网页中所有表格节点。

XPath 表达式作用于良构 XML 的层次结构。它们简洁并且格式类似于计算机文件系统中目录层次结构中定位文件的方式。但它们更加强大。XPath 与正则表达式类似，我们指定要匹配内容的模式。与正则表达式一样，撰写正确的 XPath 表达式需要经验。

XPath 表达式形成逻辑步骤，用于识别和过滤树中的节点。结果是一个*节点集*，其中每个节点最多出现一次。节点集也具有与源中节点出现顺序匹配的顺序；这一点非常方便。

每个 XPath 表达式由一个或多个*位置步骤*组成，用“/”分隔。每个位置步骤有三个部分——*轴*、*节点测试*和可选的*谓词*：

+   轴指定查找的方向，例如向下、向上或横向。我们专门使用轴的快捷方式。默认是向下一步查找树中的子节点。`//` 表示尽可能向下查找整个树，`..` 表示向上一步到父节点。

+   节点测试标识要查找的节点的名称或类型。通常只是标签名或者对于文本元素是 `text()`。

+   谓词像过滤器一样作用于进一步限制节点集。这些谓词以方括号表示，例如 `[2]`，保留节点集中的第二个节点，以及 `[ @date ]`，保留具有日期属性的所有节点。

我们可以将位置步骤连接在一起，以创建强大的搜索指令。[表 14-2](#xpath-examples) 提供了一些涵盖最常见表达式的示例。请参考 [图 14-4](#fig-xml-tree) 中的树进行跟踪。

表 14-2\. XPath 示例

| 表达式 | 结果 | 描述 |
| --- | --- | --- |
| ‘//common’ | Two nodes | 在树中向下查找任何共同节点。 |
| ‘/catalog/plant/common’ | Two nodes | 从根节点 *catalog* 沿特定路径向所有植物节点遍历，并在植物节点中的所有共同节点中查找。 |
| ‘//common/text()’ | Bloodroot, Columbine | 定位所有共同节点的文本内容。 |
| ‘//plant[2]/price/text()’ | $9.37 | 在树的任何位置定位植物节点，然后过滤并仅获取第二个节点。从此植物节点进入其价格子节点并定位其文本。 |
| ‘//@date’ | 0399, 0199 | 定位树中任何名为“date”的属性值。 |
| ‘//price[@curr=“CAD”]/text()’ | $9.37 | 具有货币属性值“CAD”的任何价格节点的文本内容。 |

您可以在目录文件中的表中尝试XPath表达式。我们使用`etree`模块将文件加载到Python中。`parse`方法读取文件到一个元素树中。

```py
`from` `lxml` `import` `etree`

`catalog` `=` `etree``.``parse``(``'``data/catalog.xml``'``)`

```

`lxml`库让我们能够访问XPath。让我们试试吧。

这个简单的XPath表达式定位树中任何`<light>`节点的所有文本内容：

```py
`catalog``.``xpath``(``'``//light/text()``'``)`

```

```py
['Mostly Shady', 'Mostly Shady']

```

注意返回了两个元素。虽然文本内容相同，但我们的树中有两个`<light>`节点，因此返回了每个节点的文本内容。以下表达式稍微有些复杂：

```py
`catalog``.``xpath``(``'``//price[@curr=``"``CAD``"``]/../common/text()``'``)`

```

```py
['Columbine']

```

该表达式定位树中所有`<price>`节点，然后根据它们的`curr`属性是否为`CAD`进行过滤。然后，对剩余节点（在本例中只有一个）在树中向上移动一步至父节点，然后返回到任何子“common”节点，并获取其文本内容。非常复杂的过程！

接下来，我们提供一个示例，使用HTTP请求检索XML格式的数据，并使用XPath将内容整理成数据框。

## 示例：访问ECB的汇率

欧洲央行（ECB）提供了在线XML格式的汇率信息。让我们通过HTTP请求获取ECB的最新汇率：

```py
`url_base` `=` `'``https://www.ecb.europa.eu/stats/eurofxref/``'`
`url2` `=` `'``eurofxref-hist-90d.xml?d574942462c9e687c3235ce020466aae``'`
`resECB` `=` `requests``.``get``(``url_base``+``url2``)`

```

```py
`resECB``.``status_code`

```

```py
200

```

同样地，我们可以使用`lxml`库解析从ECB接收到的文本文档，但这次内容是从ECB返回的字符串，而不是文件：

```py
`ecb_tree` `=` `etree``.``fromstring``(``resECB``.``content``)`

```

为了提取我们想要的数据，我们需要了解它的组织方式。这是内容的一部分片段：

```py
<gesmes:Envelope xmlns:gesmes="http://www.gesmes.org/xml/2002-08-01"
        xmlns="http://www.ecb.int/vocabulary/2002-08-01/eurofxref">
<gesmes:subject>Reference rates</gesmes:subject>
<gesmes:Sender>
<gesmes:name>European Central Bank</gesmes:name>
</gesmes:Sender>
<Cube>
<Cube time="2023-02-24">
<Cube currency="USD" rate="1.057"/>
<Cube currency="JPY" rate="143.55"/>
<Cube currency="BGN" rate="1.9558"/>
</Cube>
<Cube time="2023-02-23">
<Cube currency="USD" rate="1.0616"/>
<Cube currency="JPY" rate="143.32"/>
<Cube currency="BGN" rate="1.9558"/>
</Cube>
</Cube>
</gesmes:Envelope>

```

这份文档在结构上与植物目录有很大不同。代码片段展示了三个层次的标签，它们都有相同的名称，且没有文本内容。所有相关信息都包含在属性值中。根`<Envelope>`节点中有`xmlns`和`gesmes:Enve⁠lope`等奇怪的标签名，这些与命名空间有关。

XML允许内容创建者使用自己的词汇，称为*命名空间*。命名空间为词汇提供规则，例如允许的标签名和属性名，以及节点嵌套的限制。XML文档可以合并来自不同应用程序的词汇。为了保持一致，文档中提供了有关命名空间的信息。

ECB文件的根节点是`<Envelope>`。标签名中的额外`gesmes:`表示这些标签属于gesmes词汇，这是一个用于时间序列信息交换的国际标准。`<Envelope>`中还有另一个命名空间。它是文件的默认命名空间，因为它没有像“`gesmes:`”那样的前缀。如果在标签名中未提供命名空间，则默认为此命名空间。

这意味着我们需要在搜索节点时考虑这些命名空间。让我们看看在提取日期时的运作方式。从片段中，我们看到日期存储在“time”属性中。这些 `<Cube>` 是顶层 `<Cube>` 的子节点。我们可以给出一个非常具体的 XPath 表达式，从根节点步进到其 `<Cube>` 子节点，然后进入下一级的 `<Cube>` 节点：

```py
`namespaceURI` `=` `'``http://www.ecb.int/vocabulary/2002-08-01/eurofxref``'`

`date` `=` `ecb_tree``.``xpath``(``'``./x:Cube/x:Cube/@time``'``,` `namespaces` `=` `{``'``x``'``:``namespaceURI``}``)`
`date``[``:``5``]`

```

```py
['2023-07-18', '2023-07-17', '2023-07-14', '2023-07-13', '2023-07-12']

```

表达式中的 `.` 是一个快捷方式，表示“从这里”，因为我们位于树的顶部，它相当于“从根节点”。我们在表达式中指定了命名空间为“`x:`”。尽管 `<Cube>` 节点使用了默认命名空间，但我们必须在 XPath 表达式中指定它。幸运的是，我们可以简单地将命名空间作为参数传递，并用我们自己的标签（在这种情况下是“x”）来保持标记名称的简短性。

与 HTML 表格类似，我们可以将日期值转换为字符串，再从字符串转换为时间戳：

```py
`date_str` `=` `[``str``(``s``)` `for` `s` `in` `date``]`
`timestamps` `=` `pd``.``to_datetime``(``date_str``)`
`xrates` `=` `pd``.``DataFrame``(``{``"``date``"``:``timestamps``}``)`

```

至于汇率，它们也出现在 `<Cube>` 节点中，但这些节点有一个“rate”属性。例如，我们可以使用以下 XPath 表达式访问所有英镑的汇率（目前我们忽略命名空间）：

`//Cube[@currency = "GBP"]/@rate`

这个表达式表示在文档中的任何位置查找所有 `<Cube>` 节点，根据节点是否具有货币属性值“GBP”进行过滤，并返回它们的汇率属性值。

由于我们想要提取多种货币的汇率，我们对这个 XPath 表达式进行了泛化。我们还想将汇率转换为数字存储类型，并使它们相对于第一天的汇率，以便不同的货币处于相同的比例尺上，这样更适合绘图：

```py
`currs` `=` `[``'``GBP``'``,` `'``USD``'``,` `'``CAD``'``]`

`for` `ctry` `in` `currs``:`
    `expr` `=` `'``.//x:Cube[@currency =` `"``'` `+` `ctry` `+` `'``"``]/@rate``'`
    `rates` `=` `ecb_tree``.``xpath``(``expr``,` `namespaces` `=` `{``'``x``'``:``namespaceURI``}``)`
    `rates_num` `=` `[``float``(``rate``)` `for` `rate` `in` `rates``]`
    `first` `=` `rates_num``[``len``(``rates_num``)``-``1``]`
    `xrates``[``ctry``]` `=` `[``rate` `/` `first` `for` `rate` `in` `rates_num``]`  

```

我们以汇率的折线图作为这个示例的结束。

![](assets/leds_14in05.png)

结合对 JSON、HTTP、REST 和 HTML 的知识，我们可以访问网上可用的大量数据。例如，在本节中，我们编写了从维基百科页面抓取数据的代码。这种方法的一个关键优势是我们可以在几个月后重新运行此代码，自动更新数据和图表。一个关键缺点是我们的方法与网页结构紧密耦合——如果有人更新了维基百科页面，而表格不再是页面上的第二个表格，我们的代码也需要一些修改才能工作。尽管如此，掌握从网页抓取数据的技能打开了广泛数据的大门，使各种有用的分析成为可能。

# 摘要

互联网上存储和交换的数据种类繁多。在本章中，我们的目标是让您领略到可用格式的多样性，并基本理解如何从在线来源和服务获取数据。我们还解决了以可重复的方式获取数据的重要目标。与其从网页复制粘贴或手工填写表单，我们演示了如何编写代码来获取数据。这些代码为您的工作流程和数据来源提供了记录。

每种介绍的格式，我们都描述了其结构模型。对数据集组织的基本理解有助于您发现质量问题，读取源文件中的错误，以及最佳处理和分析数据的方法。从长远来看，随着您继续发展数据科学技能，您将接触到其他形式的数据交换，我们期待这种考虑组织模型并通过一些简单案例动手实践的方法能为您服务良好。

我们仅仅触及了网络服务的表面。还有许多其他有用的主题，比如在发出多个请求或批量检索数据时保持与服务器的连接活动，使用Cookie和进行多个连接。但是理解此处介绍的基础知识可以让您走得更远。例如，如果您使用一个库从API获取数据但遇到错误，可以查看HTTP请求来调试代码。当新的网络服务上线时，您也会知道可能性。

网络礼仪是我们必须提及的一个话题。如果您计划从网站抓取数据，最好检查您是否有权限这样做。当我们注册成为Web应用的客户时，通常会勾选同意服务条款的框。

如果您使用网络服务或抓取网页，请注意不要过度请求网站。如果网站提供了类似CSV、JSON或XML格式的数据版本，最好下载并使用这些数据，而不是从网页抓取。同样，如果有一个Python库提供对Web应用的结构化访问，请使用它而不是编写自己的代码。在发送请求时，先从小处开始测试您的代码，并考虑保存结果，以免不必要地重复请求。

本章的目标不是使您成为这些特定数据格式的专家。相反，我们希望为您提供学习更多关于数据格式所需的信心，评估不同格式的优缺点，并参与可能使用您之前未见过的格式的项目。

现在您已经有了使用不同数据格式的经验，我们将回到我们在[第4章](ch04.html#ch-modeling)中引入的建模主题，认真地继续讨论。
