# 第 7 章\. 双语数据科学的案例研究

Rick J. Scavetta

Boyan Angelov

在本章的最后一部分，我们的目标是展示一个案例研究，演示本书中展示的所有概念和工具的样本。尽管数据科学提供了几乎令人难以置信的多样的方法和应用，但我们通常在日常工作中依赖于核心工具包。因此，您不太可能使用本书（或本案例研究，同样如此）中提出的所有工具。但这没关系！我们希望您专注于案例研究中与您工作最相关的部分，并且受到启发，成为一名现代的双语数据科学家。

# 24 年间发生了 1.88 百万次野火。

我们的案例研究将集中在*美国野火数据集* ^([1](ch07.xhtml#idm45127447555192))。这个数据集由美国农业部（USDA）发布，包含了 1.88 百万个地理参考的野火记录。这些火灾总共导致了 24 年间 1.4 亿英亩森林的损失。如果您想在本章中执行代码，请直接从[USDA网站](https://doi.org/10.2737/RDS-2013-0009.4)或[Kaggle](https://www.kaggle.com/rtatman/188-million-us-wildfires)下载SQLite数据集。一些预处理已经完成，例如去重。

有 39 个特征，再加上原始格式中的另一个形状变量。其中许多是唯一标识符或冗余的分类和连续表示。因此，为了简化我们的案例研究，我们将专注于[表 7-1](#csFeatures)中列出的少数几个特征。

表 7-1\. `fires`表包含描述 1992-2015 年间美国 1.88 百万次野火的 39 个特征

| 变量 | 描述 |
| --- | --- |
| STAT_CAUSE_DESCR | 火灾原因（目标变量） |
| OWNER_CODE | 土地主要所有者的代码 |
| DISCOVERY_DOY | 火灾发现或确认的年度日期 |
| FIRE_SIZE | 最终火灾规模的估计（英亩） |
| LATITUDE | 火灾的纬度（NAD83） |
| LONGITUDE | 火灾的经度（NAD83） |

我们将开发一个分类模型来预测火灾的原因（`STAT_CAUSE_CODE`），使用其他五个特征作为特征。目标和模型是次要的；这不是一个机器学习案例研究。因此，我们不会关注诸如交叉验证或超参数调整等细节^([2](ch07.xhtml#idm45127447535416))。我们还将限制自己仅使用 2015 年的观察数据，并排除夏威夷和阿拉斯加，以减少数据集的规模。我们案例研究的最终产品将是生成一个交互式文档，允许我们输入新的预测者数值，如[图 7-1](#case_arch)^([3](ch07.xhtml#idm45127447533464))所示。

在我们深入研究之前，值得花一点时间考虑数据谱系 - 从原始到产品。回答以下问题将有助于我们定位方向。

1.  最终产品是什么？

1.  如何使用，由谁使用？

1.  我们是否可以将项目分解为组件？

1.  每个组件将如何构建？即 Python 还是 R？可能需要哪些额外的包？

1.  这些组件如何协同工作？

回答这些问题使我们能够从原始数据到最终产品中找到一条路径，希望能避免瓶颈。对于第一个问题，我们已经说明我们想构建一个交互式文档。对于第二个问题，为了保持简单，让我们假设它是为了轻松输入新的特征值并查看模型的预测。

问题 3-5 是我们在本书中考虑的内容。在问题 3 中，我们将部分想象为我们整体工作流程的一系列步骤。问题 4 已在[第 4 章](ch04.xhtml#ch05)和[第 5 章](ch05.xhtml#ch06)中讨论过。我们在[表 7-2](#csOverview)中总结了这些步骤。

表 7-2\. 我们案例研究的步骤及其对应的语言。

| 组件/步骤 | 语言 | 额外的包？ |
| --- | --- | --- |
| 1\. 数据导入 | R | `RSQLite`, `DBI` |
| 2\. EDA & Data Visualization | R | `ggplot2`, `GGally`, `visdat`, `nanair` |
| 4\. 特征工程 | Python | `scikit-learn` |
| 5\. 机器学习 | Python | `scikit-learn` |
| 6\. 地图 | R | `leaflet` |
| 7\. 交互式网络界面 | R | 在 RMarkdown 中运行的`shiny` |

最后，第 5 个问题要求我们考虑项目的架构。[图 7-1](#case_arch)中呈现的图表显示了[表 7-2](#csOverview)中的每个步骤将如何链接在一起。

![](Images/prds_0701.png)

###### 图 7-1\. 我们案例研究项目的架构。

好了，现在我们知道我们要去哪里了，让我们精心选择我们的工具，并将所有组件组装成一个统一的整体。

###### 注

我们专门使用 RStudio IDE 准备了这个案例研究。正如我们在[第 6 章](ch06.xhtml#ch07)中讨论的那样，如果我们在 R 中访问 Python 函数，这将是正确的方式。原因在于 RMarkdown 中执行 Python 代码块的内置功能，环境和绘图窗格的特性，以及围绕`shiny`的工具。

# 设置和数据导入

从我们的图表中可以看出，我们的最终产品将是一个交互式的 RMarkdown 文档。所以让我们像在[第 5 章](ch05.xhtml#ch06)中所做的那样开始。我们的 YAML^([4](ch07.xhtml#idm45127447463752)) 头部至少包括：

```py
---
title: "R & Python Case Study"
author: "Python & R for the modern data scientist"
runtime: shiny
---
```

###### 注

为了具有更好的格式，我们将在以下示例中排除指定 RMarkdown 代码块的字符。自然地，如果您在跟随，您需要添加它们。

由于数据存储在 SQLite 数据库中，除了我们已经看到的包外，我们需要使用一些额外的包。我们的第一个代码块是：

```py
library(tidyverse)
library(RSQLite) # SQLite
library(DBI) # R Database Interface
```

在我们的第二个代码块中，我们将连接到数据库并列出所有的 33 个可用表格。

```py
# Connect to an in-memory RSQLite database
con <- dbConnect(SQLite(), "ch07/data/FPA_FOD_20170508.sqlite")

# Show all tables
dbListTables(con)
```

创建连接（`con`）对象是建立对数据库进行程序化访问的标准做法。与 R 不同，Python 内置支持使用 `sqlite3` 包打开此类文件。这比 R 更可取，因为我们不需要安装和加载两个额外的包。尽管如此，R 是初始步骤的核心语言，因此我们最好从一开始就在 R 中导入数据。

我们的数据存储在 `Fires` 表中。由于我们知道要访问的列，因此可以在导入时指定。

在使用远程或共享数据库时，记得关闭连接非常重要，因为这可能会阻止其他用户访问数据库并引发问题^([5](ch07.xhtml#idm45127447406008))。

```py
fires <- dbGetQuery(con, "
 SELECT
 STAT_CAUSE_DESCR, OWNER_CODE, DISCOVERY_DOY,
 FIRE_SIZE, LATITUDE, LONGITUDE
 FROM Fires
 WHERE (FIRE_YEAR=2015 AND STATE != 'AK' AND STATE != 'HI');")
dbDisconnect(con)

dim(fires)
```

我们在第一次导入时已经限制了数据集的大小。抛弃这么多数据是一件令人遗憾的事情。但我们这样做是因为旧数据，尤其是在气候应用中，往往不太代表当前或近期的情况。基于旧数据的预测可能存在固有偏见。通过限制数据集的大小，我们还减少了内存使用量，提高了性能。

# 性能提示

在处理庞大数据集的情况下（几乎或完全不适合您机器内存的数据集），您可以使用此类摄取命令仅选择样本，例如 `LIMIT 1000`。

我们可以使用 `tidyverse` 函数 `dplyr::glimpse()` 快速预览数据：

```py
glimpse(fires)
```

```py
Rows: 73,688
Columns: 6
$ STAT_CAUSE_DESCR <chr> "Lightning", "Lightning", "Lightning", "Lightning", "Misc…
$ OWNER_CODE       <dbl> 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 8, 5, 8, 5, 5, 5, …
$ DISCOVERY_DOY    <int> 226, 232, 195, 226, 272, 181, 146, 219, 191, 192, 191, 19…
$ FIRE_SIZE        <dbl> 0.10, 6313.00, 0.25, 0.10, 0.10, 0.25, 0.10, 0.10, 0.50, …
$ LATITUDE         <dbl> 45.93417, 45.51528, 45.72722, 45.45556, 44.41667, 46.0522…
$ LONGITUDE        <dbl> -113.0208, -113.2453, -112.9439, -113.7497, -112.8433, -1…
```

# 探索性数据分析与数据可视化

由于数据集仍然相对较大，我们应该仔细考虑最佳的数据可视化策略。我们的第一反应可能是绘制地图，因为我们有纬度和经度坐标。这可以直接作为 x 和 y 轴坐标输入到 `ggplot2` 中：

```py
g <- ggplot(fires, aes(x = LONGITUDE,
                  y = LATITUDE,
                  size = FIRE_SIZE,
                  color = factor(OWNER_CODE))) +
  geom_point(alpha = 0.15, shape = 16) +
  scale_size(range = c(0.5, 10)) +
  theme_classic() +
  theme(legend.position = "bottom",
        panel.background = element_rect(fill = "grey10"))

g
```

![](Images/prds_0702.png)

###### 图 7-2。绘制个别火灾的大小。

通过将 `OWNER_CODE` 映射到颜色美学，我们可以看到一些州之间存在强烈的相关性。我们可以预测这将对我们模型的性能产生重大影响。在上述代码片段中，我们将绘图分配给对象 `g`。这并不是绝对必要的，但在这种情况下我们这样做是为了展示 `ggplot2` 层叠方法的强大之处。我们可以在此图中添加一个 `facet_wrap()` 层，并将其分为 13 个面板，或 *小多面体*，每个 `STAT_CAUSE_DESCR` 类型一个。  

```py
g +
  facet_wrap(facets = vars(STAT_CAUSE_DESCR), nrow = 4)
```

![](Images/prds_0703.png)

###### 图 7-3。根据火灾原因分组的火灾图。

这使我们能够欣赏到一些原因很常见，而另一些很少见的情况，这是我们很快会以另一种方式再次看到的观察结果。我们还可以开始评估，例如，地区、所有者代码和火灾原因之间是否存在强烈的关联。

回到整个数据集，一个全面了解的简便方法是使用配对图，有时称为 splom（如果它完全由数值数据组成则称为“散点图矩阵”）。`GGally` 包^([6](ch07.xhtml#idm45127447143208)) 提供了一个出色的函数 `ggpairs()`，它生成一个图表矩阵。对角线上显示每对双变量图的单变量密度图或直方图。在上三角区域，连续特征之间的相关性也可见。

```py
library(GGally)
fires %>%
  ggpairs()
```

![](Images/prds_0704.png)

###### 图 7-4\. 一个配对图。

这个信息丰富的可视化需要一些时间来处理。它作为*探索性*绘图在探索性数据分析中非常方便，但不一定适用于结果报告中的*解释性*。你能发现任何异常模式吗？首先，`STAT_CAISE_DESCR` 看起来是不平衡的^([7](ch07.xhtml#idm45127447128744))，意味着每个类别的观察数量之间存在显著差异。另外，`OWNER_CODE` 似乎是双峰的（有两个峰值）。这些属性可能会根据我们选择的模型对我们的分析产生负面影响。其次，所有的相关性似乎都相对较低，这使得我们的工作更加轻松（因为相关的数据对机器学习不利）。尽管如此，我们已经知道位置（`LATITUDE` 和 `LONGITUDE`）与 `OWNER CODE` 之间存在强关联关系，这是从我们之前的图中得出的。因此，我们应该对这些相关性持谨慎态度。我们期望在特征工程中能检测到这个问题。第三，`FIRE_SIZE` 的分布非常不寻常。它看起来像是空的图表，只有 x 轴和 y 轴。我们看到一个密度图，在非常低的范围内有一个非常高且狭窄的峰值，以及一个极长的正偏态。我们可以快速生成一个`log10`转换后的密度图：

```py
ggplot(fires, aes(FIRE_SIZE)) +
  geom_density() +
  scale_x_log10()
```

![](Images/prds_0705.png)

###### 图 7-5\. 对数转换后的 `FIRE_SIZE` 特征的密度图。

# 其他可视化内容

对于案例研究，我们将任务保持在最低限度，但可能有一些其他有趣的可视化内容可以帮助最终用户讲述故事。例如，请注意数据集具有时间维度。了解随时间变化的森林火灾数量（和质量）将会很有趣。我们将这些内容留给有兴趣的用户使用优秀的 `gganimate` 包进行探索。

交互式数据可视化通常被滥用，没有特定的目的。即使对于最流行的包，文档也只展示了基本的用法。在我们的情况下，由于我们在空间设置中有如此多的数据点，并且我们希望得到一个易于访问的最终交付成果，创建一个交互地图是一个明显的选择。如同 [第5章](ch05.xhtml#ch06) 中我们使用 `leaflet`：

```py
library(leaflet)

leaflet() %>%
  addTiles() %>%
  addMarkers(lng = df$LONGITUDE, lat = df$LATITUDE,
  clusterOptions = markerClusterOptions()
)
```

![](Images/prds_0706.png)

###### 图 7-6\. 显示森林火灾位置的交互地图。

注意如何使用`clusterOptions`允许我们同时呈现所有数据，而不会使用户感到不知所措或降低可见性。对于我们的目的，这满足了我们在探索性数据分析中使用一些出色可视化的好奇心。我们可以应用许多其他统计方法，但让我们转向Python中的机器学习。

# 机器学习

到目前为止，我们已经对可能影响火灾原因的因素有了一些了解。让我们深入使用Python中的`scikit-learn`来构建机器学习模型^([8](ch07.xhtml#idm45127446950648))。

我们认为在Python中进行机器学习是最好的选择，正如我们在[第5章](ch05.xhtml#ch06)中所见。我们将使用随机森林算法。选择这种算法有几个原因：

1.  这是一个成熟的算法

1.  这相对容易理解

1.  在训练之前不需要进行特征缩放

还有其他一些原因使其优秀，比如良好的缺失数据处理能力和开箱即用的可解释性。

## 设置我们的Python环境

正如在[第6章](ch06.xhtml#ch07)中讨论的那样，使用`reticulate`包有几种访问Python的方式。选择取决于情况，我们在项目架构中已经说明。在这里，我们将我们的R `data.frame`传递给Python虚拟环境。如果你按照[第6章](ch06.xhtml#ch07)中的步骤进行操作，你已经设置好了`modern_data`虚拟环境。我们已经在此环境中安装了一些包。简而言之，我们执行了以下命令：

```py
library(reticulate)

# Create a new virtualenv
virtualenv_create("modern_data")

# Install Python packages into this virtualenv
library(tidyverse)
c("scikit-learn", "pandas", "seaborn") %>%
  purrr::map(~ virtualenv_install("modern_data", .))
```

如果你没有安装`modern_data`虚拟环境，或者你使用的是Windows，请参考文件`0 - setup.R`和`1 - activate.R`中的步骤，并在[第6章](ch06.xhtml#ch07)中讨论。在此时，你可能需要重新启动R，以确保可以使用以下命令激活你的虚拟环境：

```py
# Activate virtual environment
use_virtualenv("modern_data", required = TRUE)

# If using miniconda (windows)
# use_condaenv("modern_data")
```

我们将所有Python步骤都包含在一个单独的脚本中；你可以在书的[仓库](https://github.com/moderndatadesign/PyR4MDS)的`ml.py`下找到此脚本。首先，我们将导入必要的模块。

```py
from sklearn.ensemble import RandomForestClassifier
from sklearn. preprocessing import LabelEncoder
from sklearn.model_selection import train_test_split
from sklearn import metrics
```

## 特征工程

数据集中有些特征对数据分析师可能有信息意义，但对于训练模型来说最好是无用的，甚至会降低其准确性。这被称为向数据集中“添加噪音”，我们要尽量避免这种情况。这就是特征工程的目的。让我们只选择我们需要的特征，如[表7-1](#csFeatures)中所指定的那样。我们还使用了标准的机器学习约定，将它们存储在`X`中，我们的目标在‘y’中。

```py
features = ["OWNER_CODE", "DISCOVERY_DOY", "FIRE_SIZE", "LATITUDE", "LONGITUDE"]
X = df[features]
y = df["STAT_CAUSE_DESCR"]
```

在这里，我们创建了一个`LaberEncoder`的实例。我们用它来将分类特征编码为数值。在我们的案例中，我们将其应用于我们的目标：

```py
le = LabelEncoder()
y = le.fit_transform(y)
```

在这里，我们将数据集分为训练集和测试集（请注意，我们还使用了便捷的`stratify`参数，以确保分割函数公平地对我们的不平衡类别进行采样）：

```py
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.33,
                                                    random_state=42, stratify=y)
```

## 模型训练

要应用随机森林分类器，我们将创建一个 `RandomForestClassifier` 的实例。如同 [第 5 章](ch05.xhtml#ch06) 中使用 `fit/predict` 范式并将预测值存储在 `preds` 中一样：

```py
clf = RandomForestClassifier()

clf.fit(X_train, y_train)

preds = clf.predict(X_test)
```

在最后一步，我们将为对象分配混淆矩阵和准确率得分。

```py
conmat = metrics.confusion_matrix(y_test, preds)
acc = metrics.accuracy_score(y_test, preds)
```

在完成我们的脚本之后，我们可以将其导入 R：

```py
source_python("ml.py")
```

运行此命令后，我们将直接在我们的环境中访问所有 Python 对象。准确率为 `0.58`，这并不是非凡的，但肯定比随机要好得多！

# 利用 Python 脚本的优势

当我们使用 `reticulate` 中的 `source_python` 函数时，我们可以显著提高我们的生产力，特别是在与双语团队合作时。想象一下，你的同事在 Python 中构建了 ML 部分，而你需要将他们的工作包含在你的工作中。这只需像源代码一样简单，而不用担心重新编码一切。当加入一个新公司或项目并继承需要立即使用的 Python 代码时，这种情况也是可能的。

如果我们想利用 `ggplot` 来检查混淆矩阵，我们首先需要转换为 R 的 `data.frame`。`value` 然后是每种情况的观察次数，我们将其映射到 `size`，并将 `shape` 更改为 1（圆形）。结果显示在 [图 7-7](#conf_mat_plot) 上。

```py
library(ggplot2)
py$conmat %>%
  as.data.frame.table(responseName = "value") %>%
  ggplot(aes(Var1, Var2, size = value)) +
  geom_point(shape = 1)
```

![](Images/prds_0707.png)

###### 图 7-7\. 分类器混淆矩阵的绘图。

并不奇怪，我们有一些组具有非常高的匹配度，因为我们已经知道我们的数据一开始就不平衡。现在，我们如何处理这段漂亮的 Python 代码和输出呢？在 [第 6 章](ch06.xhtml#ch07) 的结尾，我们看到了创建交互式文档的简单有效方法（记住你在 [第 5 章](ch05.xhtml#ch06) 学到的东西），使用带有 `shiny` 运行时的 RMarkdown。让我们在这里实现相同的概念。

# 预测和用户界面

一旦我们建立了 Python 模型，通常的做法是使用模拟输入进行测试。这使我们能够确保我们的模型能处理正确的输入数据，在 ML 工程中连接实际用户输入之前都是标准做法。为此，我们将为我们模型的五个特征创建五个 `sliderInputs`。在这里，我们为简单起见硬编码了最小和最大值，但这些当然可以是动态的。

```py
sliderInput("OWNER_CODE", "Owner code:",
            min = 1, max = 15, value = 1)
sliderInput("DISCOVERY_DOY", "Day of the year:",
            min = 1, max = 365, value = 36)
sliderInput("FIRE_SIZE", "Number of bins (log10):",
            min = -4, max = 6, value = 1)
sliderInput("LATITUDE", "latitude:",
            min = 17.965571, max = 48.9992, value = 30)
sliderInput("LONGITUDE", "Longitude:",
            min = -124.6615, max = -65.321389, value = 30)
```

类似于我们在 [第 6 章](ch06.xhtml#ch07) 结尾所做的，我们将访问内部 `input` 列表中的这些值，并使用 `shiny` 包函数来呈现适当的输出。

```py
prediction <- renderText({
  input_df <- data.frame(OWNER_CODE = input$OWNER_CODE,
                         DISCOVERY_DOY = input$DISCOVERY_DOY,
                         FIRE_SIZE = input$FIRE_SIZE,
                         LATITUDE = input$LATITUDE,
                         LONGITUDE = input$LONGITUDE)

  clf$predict(r_to_py(input_df))
})
```

![](Images/prds_0708.png)

###### 图 7-8\. 我们案例研究的结果。

这些元素将根据用户输入的更改动态响应。这正是我们工作所需的，因为这是一个互动产品，而不是静态产品。您可以看到我们在准备这个项目时使用的所有不同代码块。它们应该需要很少的更改，最显著的一个是在推断部分捕捉用户输入的能力。这可以通过访问`input`对象来完成。

# 总结思路

在这个案例研究中，我们展示了如何将现代数据科学家所拥有的优秀工具结合起来，以创建令人惊叹的用户体验，这些体验在视觉上令人愉悦并且促进决策。这只是这样一个优雅系统的基本例子，我们相信通过展示可能性，我们的读者——您——将创建未来的数据科学产品！

^([1](ch07.xhtml#idm45127447555192-marker)) 短，卡伦C. 2017年。美国1992-2015年空间野火发生数据，FPA_FOD_20170508。第4版。科林斯堡，科罗拉多州：森林服务研究数据档案馆。[*https://doi.org/10.2737/RDS-2013-0009.4*](https://doi.org/10.2737/RDS-2013-0009.4)

^([2](ch07.xhtml#idm45127447535416-marker)) 我们将详细开发一个强大的分类模型留给我们有动力的读者。事实上，您可能还对预测英亩最终火灾大小的回归感兴趣。有好奇心的读者可以注意到，在Kaggle上有一些有趣的笔记本可以帮助您入门。

^([3](ch07.xhtml#idm45127447533464-marker)) 这与开发、托管和部署强大的ML模型大相径庭，而且，在任何情况下，这也不是本书的重点。

^([4](ch07.xhtml#idm45127447463752-marker)) 有些读者可能对这种语言不太熟悉。它通常用于像这种情况下的代码配置选项。

^([5](ch07.xhtml#idm45127447406008-marker)) 这部分也可以通过在R中使用`dbplyr`包或在RStudio中使用连接面板来很好地完成。

^([6](ch07.xhtml#idm45127447143208-marker)) 这个包用于扩展`ggplot2`在转换数据集方面的功能。

^([7](ch07.xhtml#idm45127447128744-marker)) 作为Python ML生态系统模块化的另一个例子，请查看`imbalanced-learn`包[这里](https://imbalanced-learn.org/)，如果您正在寻找解决方案。

^([8](ch07.xhtml#idm45127446950648-marker)) 这不是所有可能方法或优化的全面阐述，因为我们的重点是建立双语工作流程，而不是详细探讨机器学习技术。读者可以选择参考官方的`scikit-learn`文档以获取进一步的指导，位于适当命名的[“选择正确的估计器”](https://scikit-learn.org/stable/tutorial/machine_learning_map/index.html)页面。
