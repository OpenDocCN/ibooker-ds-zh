# 第 8 章\. 在 Google Cloud 上扩展

在本章中，我们将介绍如何扩展我们的实体解析过程，以便在合理的时间内匹配大型数据集。我们将使用在 Google Cloud Platform (GCP) 上并行运行的虚拟机群集来分担工作量，减少解析实体所需的时间。

我们将逐步介绍如何在 Cloud Platform 上注册新账户以及如何配置我们将需要的存储和计算服务。一旦我们的基础设施准备好，我们将重新运行[第 6 章](ch06.html#chapter_6)中的公司匹配示例，将模型训练和实体解析步骤分割到一个托管的计算资源群集中。

最后，我们将检查我们的性能是否一致，并确保我们完全清理，删除集群并返回我们借用的虚拟机，以确保我们不会继续产生额外的费用。

# Google Cloud 设置

要构建我们的云基础设施，我们首先需要在 GCP 上注册一个账户。为此，请在浏览器中访问*cloud.google.com*。从这里，您可以点击“开始”以开始注册过程。您需要使用 Google 邮箱地址注册，或者创建一个新账户。如[图 8-1](#fig-8-1)所示。

![](assets/hoer_0801.png)

###### 图 8-1\. GCP 登录

您需要选择您的国家，阅读并接受 Google 的服务条款，然后点击“继续”。见[图 8-2](#fig-8-2)。

![](assets/hoer_0802.png)

###### 图 8-2\. 注册 GCP，账户信息第一步

在下一页上，您将被要求验证您的地址和支付信息，然后才能点击“开始我的免费试用”。

# Google Cloud Platform 费用

请注意，了解使用 Google Cloud Platform 上任何产品所带来的持续费用是您的责任。根据个人经验，我可以说，很容易忽略持续运行的虚拟机或忽略您仍需付费的持久性磁盘。

在撰写本文时，Google Cloud 提供首次使用平台的 300 美元免费信用额，可在前 90 天内使用。此外，他们还声明在免费试用结束后不会自动收费，因此如果您使用信用卡或借记卡，除非您手动升级到付费账户，否则不会收取费用。

*当然，这些条款可能会更改，因此请在注册时仔细阅读条款。*

一旦您注册成功，您将被带到 Google Cloud 控制台。

## 设置项目存储

您的第一个任务是创建一个项目。在 GCP 上，项目是您管理的资源和数据的逻辑组。为了本书的目的，我们所有的工作将被组织在一个项目中。

首先，选择您喜欢的项目名称，Google 将为您建议一个相应的项目 ID。您可能希望编辑他们的建议，以简化或缩短这个项目 ID，因为您可能需要多次输入这个项目 ID。

作为个人用户，您不需要指定项目的组织所有者，如[图 8-3](#fig-8-3)所示。

![](assets/hoer_0803.png)

###### 图 8-3\. “创建项目”对话框

创建项目后，您将进入项目仪表板。

我们首先需要在 GCP 上存储我们的数据的地方。标准数据存储产品称为 Cloud Storage，其中具体的数据容器称为存储桶。存储桶具有全局唯一的名称和存储桶及其数据内容存储的地理位置。如果您愿意，存储桶的名称可以与您的项目 ID 相同。

要创建一个存储桶，您可以单击导航菜单主页（屏幕左上角的三个水平线内的圆圈）选择云存储，然后从下拉导航菜单中选择桶。[图 8-4](#fig-8-4)显示了菜单选项。

![](assets/hoer_0804.png)

###### 图 8-4\. 导航菜单—云存储

从这里，从顶部菜单中单击创建存储桶，选择您喜欢的名称，然后单击继续。参见[图 8-5](#fig-8-5)。

![](assets/hoer_0805.png)

###### 图 8-5\. 创建存储桶—命名

接下来，您需要选择首选存储位置，如[图 8-6](#fig-8-6)所示。对于本项目，您可以接受默认设置，或者如果您愿意，可以选择不同的区域。

您可以按“继续”查看剩余的高级配置选项，或者直接跳转到“创建”。现在我们已经定义了一些存储空间，下一步是保留一些计算资源来运行我们的实体解析过程。

![](assets/hoer_0806.png)

###### 图 8-6\. 创建存储桶—数据存储位置

# 创建 Dataproc 集群

与前几章类似，我们将使用 Splink 框架执行匹配。为了将我们的过程扩展到多台计算机上运行，我们需要将后端数据库从 DuckDB 切换到 Spark。

在 GCP 上运行 Spark 的一个方便的方法是使用 *Dataproc 集群*，它负责创建一些虚拟机并配置它们来执行 Spark 作业。

要创建一个集群，我们首先需要启用 Cloud Dataproc API。返回导航菜单，然后选择 Dataproc，然后像[图 8-7](#fig-8-7)那样选择 Clusters。

![](assets/hoer_0807.png)

###### 图 8-7\. 导航菜单—Dataproc 集群

然后，您将看到 API 屏幕。确保您阅读并接受条款和相关费用，然后单击启用。参见[图 8-8](#fig-8-8)。

![](assets/hoer_0808.png)

###### 图 8-8\. 启用 Cloud Dataproc API

启用 API 后，您可以单击创建集群以配置您的 Dataproc 实例。Dataproc 集群可以直接构建在 Compute Engine 虚拟机上，也可以通过 GKE（Google Kubernetes Engine）构建。对于本示例，两者之间的区别并不重要，因此建议您选择 Compute Engine，因为它是两者中较简单的一个。

接下来，您将看到[图 8-9](#fig-8-9)中的屏幕。

![](assets/hoer_0809.png)

###### 图 8-9\. 在 Compute Engine 上创建集群

在这里，您可以为您的集群命名，选择其所在的位置，并选择集群的类型。接下来，滚动到“组件”部分，并选择“组件网关”和“Jupyter 笔记本”，如 [图 8-10](#fig-8-10) 所示。这一点很重要，因为它允许我们配置集群并使用 Jupyter 执行我们的实体解析笔记本。

![](assets/hoer_0810.png)

###### 图 8-10\. Dataproc 组件

当您配置完组件后，您可以接受本页其余部分的默认设置—参见 [图 8-11](#fig-8-11)—然后选择“配置节点”选项。

![](assets/hoer_0811.png)

###### 图 8-11\. 配置工作节点

下一步是配置我们集群中的管理节点和工作节点。同样，您可以接受默认设置，但在移动到“自定义集群”之前，请检查工作节点数量是否设置为 2。

最后一步，但同样重要的是，考虑安排删除集群以避免在您完成使用后忘记手动删除集群而产生的任何持续费用。我还建议配置 Cloud Storage 临时存储桶以使用您之前创建的存储桶；否则，Dataproc 进程将为您创建一个可能在清理操作中被遗漏的存储桶。参见 [图 8-12](#fig-8-12)。

![](assets/hoer_0812.png)

###### 图 8-12\. 自定义集群—删除和临时存储桶

最后，点击“创建”指示 GCP 为您创建集群。这将需要一些时间。

# 配置一个 Dataproc 集群

基本集群运行后，我们可以通过点击集群名称，然后在显示的“Web 界面”部分选择 Jupyter 来连接到集群，如 [图 8-13](#fig-8-13) 所示。

![](assets/hoer_0813.png)

###### 图 8-13\. 集群 Web 界面—Jupyter

这将在新的浏览器窗口中启动一个熟悉的 Jupyter 环境。

我们的下一个任务是下载并配置我们需要的软件和数据。从“新建”菜单中，选择“终端”以在第二个浏览器窗口中打开命令提示符。切换到主目录：

```py
>>>cd /home
```

然后从 GitHub 仓库克隆存储库，并切换到新创建的目录中：

```py
>>>git clone https://github.com/mshearer0/handsonentityresolution

>>>cd handsonentityresolution
```

接下来，返回到 Jupyter 环境，并打开 *Chapter6.ipynb* 笔记本。运行笔记本中的数据获取和标准化部分，以重新创建干净的 Mari 和 Basic 数据集。

编辑“保存到本地存储”部分，将文件保存到以下位置：

```py
df_c.to_csv('/home/handsonentityresolution/basic_clean.csv')

df_m.to_csv('/home/handsonentityresolution/mari_clean.csv',
   index=False)
```

现在我们已经重建了我们的数据集，我们需要将它们复制到我们之前创建的 Cloud Storage 存储桶中，以便所有节点都可以访问。我们在终端中执行以下命令：

```py
>>>gsutil cp /home/handsonentityresolution/* gs://<*your
   bucket*>/handsonentityresolution/
```

###### 注意

注意：记得替换您的存储桶名称！

这将在您的存储桶中创建目录 *handsonentityresolution* 并复制 GitHub 仓库文件。这些文件将在本章和下一章中使用。

接下来我们需要安装 Splink：

```py
>>>pip install splink
```

以前，我们依赖于内置于 DuckDB 中的近似字符串匹配函数，如 Jaro-Winkler。这些例程在 Spark 中默认情况下不可用，因此我们需要下载并安装一个包含这些用户定义函数（UDF）的 Java ARchive（JAR）文件，Splink 将调用它们：

```py
​>>>wget https://github.com/moj-analytical-services/
   splink_scalaudfs/raw/spark3_x/jars/scala-udf-similarity-
   0.1.1_spark3.x.jar
```

再次，我们将此文件复制到我们的存储桶中，以便这些函数可以供集群工作节点使用：

```py
>>>gsutil cp /home/handsonentityresolution/*.jar
   gs://<*your bucket*>/handsonentityresolution/
```

要告知我们的集群在启动时从哪里获取此文件，我们需要在 Jupyter 中的路径 */Local Disk/etc/spark/conf.dist/* 中浏览到 *spark-defaults.conf* 文件，并添加以下行，记得替换您的存储桶名称：

```py
spark.jars=gs://<*your_bucket*>/handsonentityresolution/
   scala-udf-similarity-0.1.1_spark3.x.jar
```

要激活此文件，您需要关闭您的 Jupyter 窗口，返回到集群菜单，然后停止并重新启动您的集群。

# Spark 上的实体解析

最后，我们准备开始我们的匹配过程。在 Jupyter Notebook 中打开 *Chapter8.ipynb*。

首先，我们加载之前保存到我们存储桶中的数据文件到 pandas DataFrames 中：

```py
df_m = pd.read_csv('gs://<*your bucket*>/
   handsonentityresolution/mari_clean.csv')
df_c = pd.read_csv('gs://<*your bucket*>/
   handsonentityresolution/basic_clean.csv')
```

接下来我们配置 Splink 设置。这些与我们在 DuckDB 后端使用的设置略有不同：

```py
from pyspark import SparkContext, SparkConf
from pyspark.sql import SparkSession
from pyspark.sql import types

conf = SparkConf()
conf.set("spark.default.parallelism", "240")
conf.set("spark.sql.shuffle.partitions", "240")

sc = SparkContext.getOrCreate(conf=conf)
spark = SparkSession(sc)
spark.sparkContext.setCheckpointDir("gs://<*your bucket*>/
    handsonentityresolution/")

spark.udfspark.udf.registerJavaFunction(
   "jaro_winkler_similarity",
   "uk.gov.moj.dash.linkage.JaroWinklerSimilarity",
   types.DoubleType())
```

首先，我们导入 `pyspark` 函数，允许我们从 Python 创建一个新的 Spark 会话。接下来，我们设置配置参数来定义我们想要的并行处理量。然后我们创建 `SparkSession` 并设置一个 `Checkpoint` 目录，Spark 将其用作临时存储。

最后，我们注册一个新的 Java 函数，以便 Splink 可以从之前设置的 JAR 文件中获取 Jaro-Winkler 相似度算法。

接下来，我们需要设置一个 Spark 模式，我们可以将我们的数据映射到其中：

```py
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

schema = StructType(
   [StructField("Postcode", StringType()),
    StructField("CompanyName", StringType()),
    StructField("unique_id", IntegerType())]
)
```

然后我们可以从 pandas DataFrames (`df`) 和我们刚刚定义的模式创建 Spark DataFrames (`dfs`)。由于两个数据集具有相同的结构，我们可以使用相同的模式：

```py
dfs_m = spark.createDataFrame(df_m, schema)
dfs_c = spark.createDataFrame(df_c, schema)
```

我们的下一步是配置 Splink。这些设置与我们在[第6章](ch06.html#chapter_6)中使用的设置相同：

```py
import splink.spark.comparison_library as cl

settings = {
   "link_type": "link_only",
   "blocking_rules_to_generate_predictions": [ "l.Postcode = r.Postcode",
   "l.CompanyName = r.CompanyName", ],
   "comparisons": [ cl.jaro_winkler_at_thresholds("CompanyName",[0.9,0.8]), ],
   "retain_intermediate_calculation_columns" : True,
   "retain_matching_columns" : True
}
```

然后我们使用我们创建的 Spark DataFrames 和设置来设置一个 `SparkLinker`：

```py
from splink.spark.linker import SparkLinker
linker = SparkLinker([dfs_m, dfs_c], settings, input_table_aliases=
["dfs_m", "dfs_c"])
```

正如在[第6章](ch06.html#chapter_6)中一样，我们使用随机抽样和期望最大化算法分别训练 *u* 和 *m* 值：

```py
linker.estimate_u_using_random_sampling(max_pairs=5e7)
linker.estimate_parameters_using_expectation_maximisation
   ("l.Postcode = r.Postcode")
```

###### 注意

这是我们开始看到切换到 Spark 的好处的地方。以前的模型训练需要超过一个小时，现在仅需几分钟即可完成。

或者，您可以从存储库加载预训练模型 *Chapter8_Splink_Settings.json*。

```py
linker.load_model("<*your_path*>/Chapter8_Splink_Settings.json")
```

然后我们可以运行我们的预测并获得我们的结果：

```py
df_pred = linker.predict(threshold_match_probability=0.1)
   .as_pandas_dataframe()
len(df_pred)
```

# 性能测量

正如预期的那样，切换到 Spark 并没有实质性地改变我们的结果。在 0.1 的匹配阈值下，我们有 192 个匹配项。我们的结果显示在[表 8-1](#table-8-1)中。

表 8-1\. MCA 匹配结果（Spark）—低阈值

| **匹配阈值 = 0.1** | **匹配数量** | **唯一匹配实体** |
| --- | --- | --- |
| 名称和邮政编码匹配 | 47 | 45 |
| 仅名称匹配 | 37 | 31 |
| 仅邮政编码匹配 | 108 | 27 |
| **总匹配数** | **192** | **85（去重后）** |
| 未匹配 |   | 11（其中2个溶解） |
| **总组织数** |   | **96** |

这会因模型参数计算中的轻微变化而略微提高精确度和准确性。

# 整理一下！

为了确保您不会继续支付虚拟机及其磁盘的费用，请确保您从集群菜单中**删除您的集群（不仅仅是停止，这将继续积累磁盘费用）**。

如果您希望在下一章中继续使用Cloud Storage存储桶中的文件，请确保删除任何已创建的暂存或临时存储桶，如[图 8-14](#fig-8-14)所示。

![](assets/hoer_0814.png)

###### 图 8-14\. 删除暂存和临时存储桶

# 摘要

在本章中，我们学习了如何将我们的实体解析过程扩展到多台机器上运行。这使我们能够匹配比单台机器或合理的执行时间范围更大的数据集。

在这个过程中，我们已经看到如何使用Google Cloud Platform来提供计算和存储资源，我们可以按需使用，并且只支付我们所需的带宽费用。

我们还看到，即使是一个相对简单的示例，我们在运行实体解析过程之前需要大量的配置工作。在下一章中，我们将看到云服务提供商提供的API可以抽象出大部分这些复杂性。
