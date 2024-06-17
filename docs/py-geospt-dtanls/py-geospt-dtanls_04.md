# 第四章。云中的地理空间分析：谷歌地球引擎及其他工具

如何访问地理空间数据？尽管拥有企业账户的数据专业人士可能不会考虑个人计算机的限制和依赖于开源数据，但我们其他人通常会在限制条件下工作。云中的地理空间分析已经缩小了这一差距，因为这意味着我们不再需要在本地存储大量数据。一般公众从未如此全球化地获得过开源地理空间数据。本章将向您展示如何找到用于探索和学习的数据。

多年来，美国及全球的空间计划一直从卫星和传感器中收集数据，但直到最近我们才有能力实时操作这些数据进行分析。美国地质调查局托管的[EarthExplorer](https://oreil.ly/OnxdN)（Landsat），以及[Copernicus开放获取中心](https://oreil.ly/gnY7c)，提供来自欧洲空间局（ESA）哨兵卫星的数据。Landsat高分辨率卫星图像使我们能够评估和测量环境变化，理解气候科学和农业实践的影响，并在时间和空间上应对自然灾害等等。免费卫星图像的出现使得全球经济挑战区域的决策者能够获得见解并专注于解决方案。

*空间分析*包括应用于位置数据的方法和工具，其结果因位置或分析对象的框架而异。它本质上是“位置特定”的分析。这可以简单到查找最近的地铁站，或询问社区中有多少绿地或公园，也可以复杂到揭示交通可达性或健康结果中的模式。*空间算法*是通过列出和执行与地理属性集成的顺序指令来解决问题的方法，用于分析、建模和预测。

GIS解决依赖于位置信息（如纬度、经度和投影）的空间问题。空间信息回答“在地球表面的哪里发生了某事？”

例如，想象一下，您走出曼哈顿的第41街和麦迪逊大道的酒店。由于天气比您预期的寒冷得多，您在地图应用中搜索可能购买外套的地方。瞬间，服装店的位置出现在您的屏幕上。

或者从市场营销的角度来说，比如您在一家户外用品公司工作，为高品质的户外服装制造商。您可以使用地理空间信息来回答诸如：您潜在客户居住在哪里，访问哪里或旅行到哪里？附近的潜在零售位置是否是盈利的营销决策？潜在客户愿意走多远？在您正在考虑的每个位置内的平均收入是多少？这些*何处*组成部分存在于零售和商业环境中，军事、气候科学和医疗保健等领域，仅举几例。

属性是空间参考数据的另一个重要组成部分。*空间属性*在空间上有界限；这些可以包括社区边界或基础设施，如道路或地铁站，通常用多边形表示。空间参考数据还可以具有非空间属性，例如特定位置居民的收入，并为位置智能提供背景。

GIS中的*I*越来越多地存储在云端。今天，您的笔记本可以访问由云端地理空间分析处理服务提供的宠字节信息。本章将探讨其中一种服务：[谷歌地球引擎（GEE）](https://oreil.ly/ukyb0)。

2007年，微软计算机科学家Jim Gray，在同年失踪前曾发表过相当有远见的言论：“对于数据分析，一种可能是将数据移动到您的位置，但另一种可能性是将您的查询移动到数据。您可以移动您的问题，也可以移动数据。通常情况下，移动问题比移动数据更有效。”这是在云端进行地理空间分析的基本原则。

在本章中，您将使用GEE来执行与空间环境中地理属性相关的各种任务。我们还将简要介绍另一个与Python集成的工具：Leafmap。通过本章末尾，您将对这些接口有足够的熟悉度，以便跟随后续章节，并最终启动自己的独立项目。

# 谷歌地球引擎设置

但首先，您需要创建您的工作环境。本章的Jupyter笔记本可以在[GitHub](https://oreil.ly/SbS0R)上找到。您可以打开它们并跟随操作，或者在有空时分别进行代码实验和探索。安装必要的软件包和资源的说明也将涵盖在内。

GEE 数据库包含超过 60 *拍字节* 的卫星图像和遥感以及地理空间数据，所有这些数据都是免费提供的，经过预处理，易于访问。想象一下试图将所有这些数据下载到您的笔记本电脑上！GEE 的算法允许公众在云端创建交互式应用程序或数据产品。您只需申请一个免费的 [Google Earth Engine 账户](https://oreil.ly/xVOCN)（配备 250 千兆字节的存储空间），并在获得访问权限后在终端或笔记本中进行身份验证。按照以下步骤进行：

```py
To authorize access needed by Earth Engine, open the following URL 
in a web browser and follow the instructions:
https://accounts.google.com/o/oauth2/auth?client_id=xxx
The authorization workflow will generate a code, which you should paste 
in the box below
Enter verification code: code will be here

Successfully saved authorization token.
```

GEE 将向您发送一个唯一的链接和验证代码。将代码粘贴到框中并按 Enter 键。

# 使用 GEE 控制台和 geemap

GEE 控制台是快速定位图像并运行代码的资源。但是 Python 并非 GEE 的本机语言：GEE 代码编辑器专为在 JavaScript 中编写和执行脚本而设计。其 JavaScript API 具有强大的 IDE、广泛的文档和交互式可视化功能，而这些在 Python 环境中都不是本机可用的。要在 Python 环境中使用完整的交互性，您需要使用 [geemap](https://geemap.org)，这是由 [吴秋生博士](https://oreil.ly/bGQq1) 创建的用于与 GEE 交互的 Python 包。

幸运的是，您可以使用广泛的 GEE 目录，通过单击一个按钮快速定位和可视化数据，即使没有或只有有限的 JavaScript 专业知识。您可以通过浏览“脚本”选项卡轻松找到界面并生成地图。每个代码脚本都允许您运行 JavaScript 代码并生成地图。但是，如果您希望自主构建自己的地图并进行交互操作，您将需要使用 geemap。GEE 目录（如 [图 4-1](#the_gee_console) 所示）包含您在决定如何与 geemap 中的数据交互时所需的有用信息。

![GEE 控制台](assets/pgda_0401.png)

###### 图 4-1\. GEE 目录

浏览 Earth Engine 数据目录，找到数据集 [集合](https://oreil.ly/yAow0)，并向下滚动页面。在页面底部，您会注意到提供了 JavaScript 代码。只需将其复制并粘贴到控制台中，如 [图 4-2](#google_earth_engine_ide) 所示。

[图 4-2](#google_earth_engine_ide) 展示了当您将代码粘贴到控制台并从中心面板的选项列表中选择“运行”时生成的内容。例如，来自 [图 4-1](#the_gee_console) 的数据生成了 USGS Landsat 8 Level 2, Collection 2，被标识为 `ee.ImageCollection("LANDSAT/LC08/C02/T1_L2")`。

![Google Earth Engine IDE](assets/pgda_0402.png)

###### 图 4-2\. Google Earth Engine IDE

让我们学习如何在 Jupyter Notebook 中使用 Python 脚本生成 GEE 图像。geemap 甚至有一个工具，可以直接在您的 Jupyter Notebook 中将 JavaScript 代码转换为 Python。

Jupyter Notebook 是独立于您的 Python 环境的实体。最初它是以其与三种不同编程语言 Julia、Python 和 R 交互的能力而命名的，但它已经远远超出了最初的能力范围。您必须告诉系统您想要使用的 Python 版本。*内核* 是 Notebook 和 Python 之间通信的方式。

安装 geemap 将在 Notebook 环境中创建一个类似于 GEE 控制台的控制台，但使用的是 Python API 而不是 JavaScript。一旦您设置了 Conda 环境，您就可以在 Jupyter Notebook 中与 GEE 进行交互。首先，您需要下载所需的软件包和库。

## 创建 Conda 环境

[Anaconda](https://www.anaconda.com) 是一个流行的跨平台 Python 和其他编程语言的分发管理器，用于安装和管理 Conda 软件包。您可以将 Anaconda 视为所有数据科学工具的存储库。Conda 管理您的软件包和工具，允许您根据需要上传新工具并自定义您的工作环境。

Conda 软件包存储在 Anaconda 仓库或云中，因此您无需额外的工具进行安装。Conda 允许您根据需要使用您喜欢的 Python 版本创建任意数量的环境。您也可以选择下载一个更精简的 Anaconda 版本，称为[Miniconda](https://oreil.ly/nh0LE)，无论您使用的是哪种操作系统，我都推荐它。两者都是简单的安装。我建议查看[Ted Petrou 的这篇教程](https://oreil.ly/nPTlh)中的 Miniconda 安装说明。

## 打开 Jupyter Notebook

Jupyter Notebook 是开源的互动式 Web 工具。它们在您的浏览器中运行，不需要额外的下载。您可以在[GitHub](https://oreil.ly/SbS0R)上访问本章的 Jupyter Notebook：文件名为*4 Geospatial Analytics in the Cloud*。您可以在 Notebook 的文件菜单中找到并配置已安装的*nbextensions*。这些是方便的插件，为您的 Jupyter Notebook 环境增加更多功能。

## 安装 geemap 和其他软件包

安装完成 Conda 环境后，您可以打开终端或命令提示符安装 geemap。逐行执行以下代码以激活您的工作环境。在这里，我将我的地理空间环境命名为`gee`：

```py
conda create -n gee python=3.9
conda activate gee
conda install geemap -c conda-forge
conda install cartopy -c conda-forge
conda install jupyter_contrib_nbextensions -c conda-forge
jupyter contrib nbextension install --user
```

请注意，我指定了要包含在环境中的 Python 版本。我这样做是因为仍有一些依赖项尚未准备好支持最新版本的 Python。这是环境很有用的一个重要原因：如果存在兼容性冲突，您将收到警告，并且可以使用避免这些冲突的版本创建环境。

这次安装还包括了 Cartopy，一个用于地理空间数据处理的 Python 包；`jupyter_contrib_nbextensions`，一个用于扩展功能的包；以及 `contrib_nbextensions`，它将向 Jupyter 配置中添加样式。

现在你已经将包安装到了你的环境中，每当你打开一个新的会话时，你只需要在一个代码单元中运行 `import geemap`。环境现在在你激活时可见，如下所示 `(gee)`：

```py
(gee) MacBook-Pro-8:~ bonnymcclain$ conda list
```

此环境将包含所有相关的包以及它们的依赖关系。你可以创建不同的环境（简称为 `env`），其中包含每个项目特有的依赖关系和包。

`conda list` 命令会显示当前环境中安装了哪些包。下面是我执行该命令时加载的部分内容：

```py
# packages in environment at /Users/bonnymcclain/opt/miniconda3/envs/geo:
#
# Name                         Version                 Build    Channel
aiohttp                        3.7.4          py38h96a0964_0    conda-forge
anyio                          3.1.0          py38h50d1736_0    conda-forge
appnope                        0.1.2          py38h50d1736_1    conda-forge
argon2-cffi                    20.1.0         py38h5406a74_2    conda-forge
async-timeout                  3.0.1                 py_1000    conda-forge
async_generator                1.10                     py_0    conda-forge
attrs                          21.2.0           pyhd8ed1ab_0    conda-forge
backcall                       0.2.0            pyh9f0ad1d_0    conda-forge
backports                      1.0                      py_2    conda-forge
backports.functools_lru_cache  1.6.4            pyhd8ed1ab_0    conda-forge
beautifulsoup4                 4.9.3            pyhb0f4dca_0    conda-forge
bleach                         3.3.0            pyh44b312d_0    conda-forge
bqplot                         0.12.27          pyhd8ed1ab_0    conda-forge
branca                         0.4.2            pyhd8ed1ab_0    conda-forge
brotlipy                       0.7.0       py38h5406a74_1001    conda-forge
bzip2                          1.0.8              h0d85af4_4    conda-forge
c-ares                         1.17.1             h0d85af4_1    conda-forge
ca-certificates                2020.12.5          h033912b_0    conda-forge
cachetools                     4.2.2            pyhd8ed1ab_0    conda-forge
cartopy                        0.19.0.post1   py38h4be4431_0    conda-forge
certifi                        2020.12.5      py38h50d1736_1    conda-forge
cffi                           1.14.5         py38ha97d567_0    conda-forge
chardet                        4.0.0          py38h50d1736_1    conda-forge
click                          8.0.1          py38h50d1736_0    conda-forge
colour                         0.1.5                    py_0    conda-forge
geemap                         0.8.16           pyhd8ed1ab_0    conda-forge
...
```

这对于你的代码由于缺少依赖而抛出错误很有帮助。运行 `conda list`；你应该也会看到列出的版本。运行 `conda env list` 将显示你已经安装的任何环境。

我为每个激活的环境安装一个内核（运行在你的环境中的操作系统的一部分）：

```py
conda install ipykernel
```

现在你可以将内核添加到你的环境中——在这种情况下，`*<你的环境名称>*` 是 `gee`：

```py
python -m ipykernel install --user --name myenv --display-name 
"<your environment name>"
```

你的本地计算机必须访问文件。`import` 语句将包添加为 Python *对象*（即，一组数据和方法）到当前正在运行的程序实例中。

打开你的终端并在控制台中写入 `**jupyter notebook**`。一个 Notebook 应该会在你的浏览器中打开。你需要将所需的库导入到 Notebook 中。你可以在代码 shell 中看到它们列出。请注意，`os` 允许你访问运行 Python 的操作系统，`ee` 是 Earth Engine 库，而 `geemap` 允许你通过 Python 进行接口操作。你将使用 `import` 函数导入这些库：

```py
import os
import ee
import geemap
#geemap.update_package()
```

计算机操作系统的中心组件是*内核*。内核针对每种编程语言具体化，并且默认内核取决于你在 Notebook 中运行的 Python 版本。

你需要重新启动内核以使更新生效。从菜单中选择 Kernel，滚动到重新运行选项。你现在可以开始在 Notebook 中工作了。

请注意上一个代码块的最后一行中的井号（#）。在 Python 代码中，井号表示*注释*，或者一行代码，不会运行。当你想要运行该行代码时，删除井号。为了确保你使用的是更新的 geemap 包，*取消注释*最后一行（即，删除最后一行的井号）然后再运行代码。更新 geemap 后，你可以再次插入井号，因为你不需要每次运行代码时都更新该包。你还可以添加注释文本以包含任何澄清细节。你将在本书的许多代码块中看到这种做法。

# 导航 geemap

我之前提到了对象。Python专注于对象，而不是您在其他编程语言中熟悉的*函数*，因此被称为*面向对象*的编程语言。

您可能还记得早期章节中提到的*类*就像建筑物的蓝图一样。建筑物是对象，但是可以根据一套蓝图建造许多建筑物，对吧？代码中的对象是类的*实例*，就像建筑物是那些蓝图的*实例*一样。创建对象称为*实例化*。

下一个代码块是声明一个对象实例，我称之为`m``ap`，并在`geemap.Map()`中定义属性和方法。您可以将您的变量设置为任何您喜欢的内容，但要保持一致。我建议保持简单但信息丰富和实用。您将使用对象名称`map`访问对象的属性。截至本文撰写时，geemap默认显示世界地图。如果您希望将地图中心定位到特定位置，可以使用纬度/经度（lat/lon）坐标和缩放级别来指定位置。以下将使您的地图以美国为中心：

```py
map = geemap.Map(center=(40, -100), zoom=4)
map
```

## 图层与工具

[图 4-3](#the_layers_and_tools_menu) 显示了地图右侧的图层和工具菜单。地图的每一层实际上都是自己的数据库，其中包含地理数据的集合。这些可能包括道路、建筑物、河流或湖泊，都以点、线和/或多边形的集合形式表示，可以是矢量数据或来自栅格数据的影像。图层图标将显示地图中的不同图层。您可以更改它们的透明度，切换图层的显示/隐藏，并检查其他属性。输入以下代码以在您的笔记本中渲染地图：

```py
map = geemap.Map(center =(40, -100), zoom=4)
map.basemap()
map
```

![图层与工具菜单](assets/pgda_0403.png)

###### 图 4-3\. 图层与工具菜单

单击不同的选项以查看它们如何自定义地图。

球形图标（在[图 4-4](#searching_location_data_using_gee_asset)中的最左侧）是搜索位置/数据功能。在这里，您可以通过输入名称和地址或一组纬度/经度坐标或搜索和导入数据来找到要加载到地图上的数据。随着我们构建几个地图图层，我们将探索更多这些选项，并向您展示一些快捷方式，帮助您浏览地图画布。

![在导入窗口中使用GEE资产ID（Landsat/LC08/C01）搜索位置数据](assets/pgda_0404.png)

###### 图 4-4\. 使用GEE资产ID（Landsat/LC08/C01）在导入窗口中搜索位置数据

您可以通过在geemap中输入搜索参数来访问USGS Landsat地图。选择“数据”并在搜索栏中输入几个词条。再次探索GEE可能有助于识别感兴趣的地图。您可以按名称将它们调用到您自己的项目中。一旦选择了一个Landsat集合并选择了资产ID，点击导入按钮（在[图 4-4](#searching_location_data_using_gee_asset)中显示为蓝色）。

要查看可用参数和其他自定义选项，请将光标置于`geemap.Map()`括号内，然后按Shift + Tab。现在可以在[Figue 4-5](#a_basemap_in_gee_with_docstring)下的地图下方看到文本。在这里，您可以阅读有关可用参数和进一步自定义地图的信息。

![在GEE中带有文档字符串的底图](assets/pgda_0405.png)

###### 图4-5\. 在GEE中带有文档字符串的底图

## 底图

请回顾[Figue 4-3](#the_layers_and_tools_menu)一刻，看看最右边的下拉菜单。此菜单包含可用*底图*的字典：这些基础地图是您数据探索的基础。Jupyter Notebook允许您在不编写代码的情况下滚动查看可用的底图。

根据您的数据问题或数据的性质，您可能希望显示不同的地理空间信息。例如，如果您有兴趣显示*水文*（水体的物理特性和可航性），您可能不会选择展示主要道路和高速公路的底图。为了迅速和高效，底图存储为栅格或矢量瓦片。底图字典与*TileLayer*交互，允许与地图服务如NASA的[全球影像浏览服务（GIBS）](https://oreil.ly/AOwFm)和[OpenStreetMap](https://oreil.ly/1Ti4J)进行连接。

geemap软件包将GEE的所有分析功能带入*ipyleaflet*，这是一个交互式库，可以将地图引入到您的笔记本中，允许您在更新位置和缩放级别时看到地图的动态更新。geemap的默认地图是Google Maps的全球视图。接下来，您将使用OSM作为您的底图，请运行：

```py
add_google_map = False
```

高级用户可以选择创建自己的TileLayer，但[ipyleaflet地图和底图](https://oreil.ly/HJJbT)资源中也提供多种默认底图。

现在您知道如何将地图加载到您的笔记本中，请大胆开始尝试吧。您的目标是让自己变得好奇，并且感到舒适地在Jupyter Notebook中导航和选择不同的工具。

# 探索Landsat 9图像收集

我们一直在使用Landsat数据，所以让我们看看Landsat 9数据，它于2022年初首次发布，并且在本文撰写时仍在发布中。要查看数据集的可用部分，请运行以下代码：

```py
collection = ee.ImageCollection('LANDSAT/LC09/C02/T1_L2')
print(collection.size().getInfo())
```

这输出：`106919`。

这一收藏包括106,919张图像，并且仍在增加中！

作为比较，Landsat/LC08/C02/T1_L2收集包含1,351,632张图像。到本书出版时，Landsat 9的图像数量将大大增加。您可以计算所有匹配波段的中位值来减少图像收集的大小：

```py
median = collection.median()
```

# 使用光谱带进行工作

正如您在[第1章](ch01.xhtml#introduction_to_geospatial_analytics)学到的，*光谱波段* 就像不同类型光的容器。反射光被捕捉为一系列不同波长或颜色的光能波段。想象电磁光谱。光谱的每个部分实际上是一个波段。本节中关于波段的信息旨在突出数据的位置，以便输入到代码中以访问正确的信息。Landsat 8 收集的[波段](https://oreil.ly/NN6Eq)适用于 Landsat 9。您将需要这些数据来应用*缩放因子*，或线性距离的比较，以调整基于地图投影的面积和角度的扭曲（也在[第1章](ch01.xhtml#introduction_to_geospatial_analytics)中涵盖）。请记住，地球是椭圆形的，而不是完美的球体！我们从比例和偏移中推导出缩放因子，如[图4-6](#band_characteristics_of_landsat_eightso)所示。

![Landsat 8/9 数据的波段特征](assets/pgda_0406.png)

###### 图4-6。Landsat 8/9 数据的波段特征

USGS 提供了关于哪些光谱波段最适合不同类型研究的指导。您可以了解[更多关于科学](https://oreil.ly/vTrFF)，并探索[常见的 Landsat 波段组合](https://oreil.ly/HdXC0)。

您将在 Jupyter Notebook 中导入*ee.ImageCollection*并将其添加为数据层到您的地图中。然后，您将从所有图像创建一个复合图像。这将产生光谱波段的中位数值。

在 Python 中，我们使用关键字`def`定义函数。在下面的代码中，函数名为`apply_scale_factors`，参数为`(image)`：

```py
def apply_scale_factors(image):
   opticalBands = image.select('SR_B.*').multiply(0.0000275).add(-0.2)
   thermalBands = image.select('ST_B.*').multiply(0.00341802).add(149.0)
   return image.addBands(opticalBands, None, True).addBands(thermalBands, None, 
   True)
```

星号（*）告诉函数您要选择满足定义的搜索要求的多个波段。Landsat 的传感器是操作陆地成像仪（OLI）和热红外传感器（TIRS）。OLI 产生光谱波段 1 到 9，而 TIRS 包括两个热波段：SR_B 和 ST_B。

冒号（:）标志着函数体的开始。在缩进的函数体内，`return` 语句确定要返回的值。函数定义完成后，使用参数调用函数将返回一个值：

```py
dataset = apply_scale_factors(median)
```

要理解为什么您会希望选择某些波段，请将它们视为具有光谱特征签名。自然彩色波段使用 SR_B4 代表红色，SR_B3 代表绿色，SR_B2 代表蓝色。绿色表示健康的植被，棕色表示不那么健康，白灰色通常表示城市特征，水域将显示为深蓝或黑色。

近红外（NIR）复合物使用 NIR（SR_B5）、红色（SR_B4）和绿色（SR_B3）。红色区域具有更好的植被健康状况，暗区域是水域，城市区域是白色。因此，请包括这些作为您的可视化参数：

```py
vis_natural = {
   'bands': ['SR_B4', 'SR_B3', 'SR_B2'],
   'min': 0.0,
   'max': 0.3,
}

vis_nir = {
   'bands': ['SR_B5', 'SR_B4', 'SR_B3'],
   'min': 0.0,
   'max': 0.3,
}
```

现在将它们作为数据层添加到您的地图中：

```py
Map.addLayer(dataset, vis_natural, 'True color (432)')
Map.addLayer(dataset, vis_nir, 'Color infrared (543)')
Map
```

在 [图 4-7](#different_band_combinations_of_landsat) 中，我将红外层关闭，这样您可以更清楚地看到其他波段。还似乎有云层存在。Landsat 9 每 16 天重采样一次，因此在查看时会有所不同。

![Landsat 8/9 不同波段组合](assets/pgda_0407.png)

###### 图 4-7\. Landsat 8/9 不同波段组合

如果将鼠标悬停在工具栏图标上，将会看到图层菜单出现。您可以更改任何地图的不透明度，并取消选择您不想在图层菜单中查看的图层。您还可以单击齿轮图标以探索属性。您还可以指定要显示的最小和最大值。拉伸数据会展开像素值，您可以尝试不同的值。您的数据将显示波段的范围，您可以决定要显示哪些值：

```py
vis_params = [
   {'bands': ['SR_B4', 'SR_B3', 'SR_B2'], 'min': 0, 'max': 0.3},
   {'bands': ['SR_B5', 'SR_B4', 'SR_B3'], 'min': 0, 'max': 0.3},
   {'bands': ['SR_B7', 'SR_B6', 'SR_B4'], 'min': 0, 'max': 0.3},
   {'bands': ['SR_B6', 'SR_B5', 'SR_B2'], 'min': 0, 'max': 0.3},
]
```

要为这些图层添加标签，请创建标签列表：

```py
labels = [
   'Natural Color (4, 3, 2)',
   'Color Infrared (5, 4, 3)',
   'Short-Wave Infrared (7, 6 4)',
   'Agriculture (6, 5, 2)',
]
```

然后为每一层分配一个标签：

```py
geemap.linked_maps(
   rows=2,
   cols=2,
   height="400px",
   center=[-3.4653, -62.2159],
   zoom=4,
   ee_objects=[dataset],
   vis_params=vis_params,
   labels=labels,
   label_position="topright",
)
```

在 [图 4-8](#landsat_band_combinations) 中检查另外两个参数，您还可以看到短波红外线。这里，深绿色表示密集植被，蓝色显示城市区域，健康植被为绿色，裸地为品红色。

![Landsat 波段组合](assets/pgda_0408.png)

###### 图 4-8\. Landsat 波段组合

让我们将您对 GEE 和 geemap 的介绍应用到开始探索。

# 国家土地覆盖数据库底图

[National Land Cover Database (NLCD)](https://oreil.ly/xNJul) 跟踪美国的土地覆盖情况。它在 [Earth Engine 数据目录](https://oreil.ly/1IuYF) 中免费提供，并每五年更新一次。*土地覆盖* 数据包括空间参考和地表特征，例如树冠覆盖（我们在上一章节中探索过的），不透水表面以及生物多样性和气候变化的额外模式。*不透水土地覆盖* 意味着非自然表面，例如沥青、混凝土或其他人造层，限制雨水渗入土壤的自然能力。这些信息可以帮助预测在暴雨期间哪些区域可能更容易发生洪水。

在本节中，您将使用 NLCD 数据对几个 Landsat 基础上的不透水数据层（用于城市类）和决策树分类（用于其他类）进行检查。我们不会在这里进行完整的活动，只是快速导向，但我鼓励您进一步探索。

## 访问数据

转到 [Earth Engine 数据目录](https://oreil.ly/oWZcr) 并滚动到 NLCD_Releases/2019_REL/NLCD 或国家土地覆盖数据库，如 [图 4-9](#the_earth_engine_data_catalog) 所示。早些时候我们注意到，您只需将此数据添加到地图上，但这里还有几个选项我想向您展示。复制 JavaScript 代码并将其放在剪贴板上。

![Earth Engine 数据目录](assets/pgda_0409.png)

###### 图 4-9\. Earth Engine 数据目录

NLCD 目录提供了丰富的信息，包括采集的日期范围、数据源、图像片段、数据描述、多光谱波段信息和图像属性。

在 geemap 中，生成世界地图的默认设置：

```py
map = geemap.Map()  
map
```

接下来，选择转换 JavaScript 图标。将显示 [图 4-9](#the_earth_engine_data_catalog) 中的框。将 JavaScript 代码从目录粘贴到框中。按照弹出窗口中的代码注释中显示的说明，继续执行 [图 4-10](#using_geemap_to_convert_a_script_from_j) 中显示的操作：

```py
// Import the NLCD collection.
var dataset = ee.ImageCollection('USGS/NLCD_RELEASES/2019_REL/NLCD');

// The collection contains images for multiple years and regions in the USA.
print('Products:', dataset.aggregate_array('system:index'));

// Filter the collection to the 2016 product.
var nlcd2016 = dataset.filter(ee.Filter.eq('system:index', '2016')).first();

// Each product has multiple bands for describing aspects of land cover.
print('Bands:', nlcd2016.bandNames());

// Select the land cover band.
var landcover = nlcd2016.select('landcover');

// Display land cover on the map.
Map.setCenter(-95, 38, 5);
Map.addLayer(landcover, null, 'Landcover');
```

点击“转换”后，您将看到代码从 JavaScript 更新为 Python，如 [图 4-10](#using_geemap_to_convert_a_script_from_j) 所示。

![使用 geemap 将脚本从 JavaScript 转换为 Python](assets/pgda_0410.png)

###### 图 4-10\. 使用 geemap 将脚本从 JavaScript 转换为 Python

如果代码没有更新到 Jupyter Notebook 的新单元格中，您可以剪切并粘贴到新单元格中运行该单元格。地图现在将显示为您的地图。

现在让我们包括默认的 NLCD 图例。选择地表覆盖层。要发现可用作默认选项的图例，请运行 `builtin_legend` 函数：

```py
legends = geemap.builtin_legends
for legend in legends:
    print(legend)
```

NLCD 的图例将作为一个选项列出。选择它将其添加到您的地图中。

## 创建自定义图例

虽然 NLCD 提供了内置的图例选项，但许多数据集没有——即使有，这些图例也不总是完全符合您的需求。因此，能够创建自己的地图图例是很有帮助的。现在让我们看看如何做到这一点。

数据集中的类通常对应于您在图例中希望显示的类别。幸运的是，您可以将类表转换为图例。

如果您的数据来自 GEE 数据目录，您可以在那里找到一个类表。然后使用以下代码（或在 Jupyter Notebook 中找到此代码单元格）并将类表中的文本复制到其中：

```py
map = geemap.Map()

legend_dict = {
    '11 Open Water': '466b9f',
    '12 Perennial Ice/Snow': 'd1def8',
    '21 Developed, Open Space': 'dec5c5',
    '22 Developed, Low Intensity': 'd99282',
    '23 Developed, Medium Intensity': 'eb0000',
    '24 Developed High Intensity': 'ab0000',
    '31 Barren Land (Rock/Sand/Clay)': 'b3ac9f',
    '41 Deciduous Forest': '68ab5f',
    '42 Evergreen Forest': '1c5f2c',
    '43 Mixed Forest': 'b5c58f',
    '51 Dwarf Scrub': 'af963c',
    '52 Shrub/Scrub': 'ccb879',
    '71 Grassland/Herbaceous': 'dfdfc2',
    '72 Sedge/Herbaceous': 'd1d182',
    '73 Lichens': 'a3cc51',
    '74 Moss': '82ba9e',
    '81 Pasture/Hay': 'dcd939',
    '82 Cultivated Crops': 'ab6c28',
    '90 Woody Wetlands': 'b8d9eb',
    '95 Emergent Herbaceous Wetlands': '6c9fb8' }

landcover = ee.Image('USGS/NLCD/NLCD2019').select('landcover')
Map.addLayer(landcover, {}, 'NLCD Land Cover')

Map.add_legend(legend_title="NLCD Land Cover Classification", 
   legend_dict=legend_dict)
Map
```

您可以在 [geemap 文档](https://geemap.org) 中找到更多关于手动构建和自定义图例的信息。

现在您可以探索地图并深入研究您感兴趣的区域。您想要从这张地图中提出什么问题？花些时间来探索。有许多不同的工具可以用来自定义您的地图！

[GEE 目录](https://oreil.ly/3Q25Z) 很广泛。当您在此学习到的技能探索不同的数据库和数据集时，您将能够处理栅格和矢量数据，并上传自己的数据源。有关在 geemap 上的有用附加功能列表，请参见 [geemap GitHub 页面](https://geemap.org/usage)。不过，我也想向您介绍 GEE 的一个替代方案。

# Leafmap: Google Earth Engine 的替代方案

在不依赖于 GEE 的情况下可视化地理空间数据并不受限制！如果您没有访问 GEE 帐户或不感兴趣使用 GEE，可以考虑使用 Leafmap。Leafmap 是一个 Python 包，允许您在 Jupyter Notebook 环境中可视化交互式地理空间数据。它基于您已经使用过的 geemap 包，但正如您在本节中将看到的，Leafmap 提供了在 GEE 平台之外访问地理空间数据的功能。其图形用户界面减少了您需要编写的代码量。它的核心是各种开源包。

Leafmap 适用于许多不同的绘图后端，包括 ipyleaflet（在此上下文中，*后端* 是在服务器上运行并接收客户端请求的内部代码）。用户看不到后端，但它们仍在运行。

您可以通过[GitHub 链接](https://oreil.ly/9ADWy)访问 Jupyter Notebook Leafmap。根据您的 Python 版本，请参阅[Leafmap 文档](https://oreil.ly/f2Ztx)获取特定的安装说明。（如果您不确定您安装了哪个版本，请在终端输入`**python**`，它将输出您已安装的版本号。这对于遇到包安装问题时非常重要。）

可以设置一个新的环境来使用 Leafmap。我最初使用 Python 3.8 创建了下面代码中显示的 Conda 环境，但后续版本也可能适用。我将这个环境命名为`geo`，因为它在不同版本的 Python 中运行：

```py
conda create -n geo python=3.8
conda activate geo
conda install geopandas
conda install leafmap -c conda-forge
conda install mamba -c conda-forge
mamba install leafmap xarray_leaflet -c conda-forge
conda install jupyter_contrib_nbextensions -c conda-forge
pip install keplergl
```

与之前一样，要打开 Notebook，请输入`**jupyter notebook**`并按 Enter 键。现在将以下代码输入到 Notebook 中，类似于显示在[图 4-11](#installing_basemaps_in_leafmap)中的内容：

```py
from ipyleaflet import *
m = Map(center=[48.8566, 2.3522], zoom=10, height=600, widescreen=False,
basemaps=basemaps.Stamen.Terrain)
m
```

![Leafmap 中安装底图](assets/pgda_0411.png)

###### 图 4-11\. Leafmap 中安装底图

更改底图就像将光标放在底图括号内并在键盘上选择 Tab 键一样简单。[图 4-12](#changing_basemaps_in_leafmap)展示了可用的选项。这里选择了 Esri 作为底图，但您可以向上或向下滚动直到找到合适的选项。一定要探索。一旦输入`**Esri**`，选项将显示。

![Leafmap 中更改底图](assets/pgda_0412.png)

###### 图 4-12\. Leafmap 中更改底图

另一个有用的工具是能够预设您的缩放级别。当您在 Notebook 中运行单元格时，您将可以在不同的缩放级别之间滑动选择：

```py
m.interact(zoom=(5,10,1))
```

[图 4-13](#zoom_levels_in_leafmap)显示了输出。

![Leafmap 中的缩放级别](assets/pgda_0413.png)

###### 图 4-13\. Leafmap 中的缩放级别

您还可以通过将小地图插入到较大地图中提供参考，如[图 4-14](#a_map_within_a_map_the_minimap_function)所示。要执行此操作，请输入以下代码：

```py
minimap = Map(
    zoom_control=False, attribution_control=False, 
    zoom=5, center=m.center, basemap=basemaps.Stamen.Terrain
)
minimap.layout.width = '200px'
minimap.layout.height = '200px'
link((minimap, 'center'), (m, 'center'))
minimap_control = WidgetControl(widget=minimap, position='bottomleft')
m.add_control(minimap_control)
```

显示在[图 4-14](#a_map_within_a_map_the_minimap_function)中的小地图将帮助用户在更大的背景下保持方向感。

![地图中的地图：Leafmap 中的小地图功能](assets/pgda_0414.png)

###### 图 4-14\. 地图中的地图：Leafmap 中的小地图功能

# 总结

本章探讨了Google Earth Engine及其相关工具、库和包，这些工具可用于回答地理空间问题，并介绍了另一种工具Leafmap。这一章及其相关笔记本将成为你在下一章项目中的便捷参考。你已在画布上呈现了可视化效果并创建了地图。接下来，你将开始分析这些关系并探索工具，以进行地理空间数据的高级分析。
