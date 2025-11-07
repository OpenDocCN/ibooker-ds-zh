# 第二章：开始使用 Amazon Redshift

Amazon Redshift 使您能够运行业务分析，无需预置服务器或基础设施，使得开始变得轻松。它包括 AWS 控制台中的基于 Web 的查询编辑器，可以开始加载和分析数据，无需安装软件。Amazon Redshift 还兼容您喜爱的查询编辑器，如 DBeaver、SQL WorkbenchJ 和 Toad，使用提供的*Java 数据库连接*（JDBC）或*开放数据库连接*（ODBC）驱动程序。

在本章中，我们将提供 Amazon Redshift 架构概述，描述平台的关键组件。接下来，我们将提供有关如何“开始使用 Amazon Redshift 无服务器”并开始查询“样本数据”的步骤，只需点击几下按钮。我们还将描述“何时使用预置集群？”以及如何为无服务器和预置数据仓库“估算您的 Amazon Redshift 成本”。然后，我们将讨论“AWS 账户管理”，描述如何在您的组织中创建单个账户或管理多个账户。最后，我们将介绍“连接到您的 Amazon Redshift 数据仓库”的一些选项以及如何管理用户访问。

# Amazon Redshift 架构概述

> 名字有什么关系呢？我们可以用任何其他名字来称呼一朵玫瑰，它依然会一样芳香。
> 
> *罗密欧与朱丽叶*

威廉·莎士比亚在他的剧作*罗密欧与朱丽叶*中使用这句话来表达，事物的命名是无关紧要的。

关于名为*Redshift*的原因和含义有许多理论。我们发现这个术语是基于天文学现象红移而创造的，红移是天文学中的一个概念，指的是光的波长在扩展时被拉长，因此光线向红色移动，这是可见光谱中最长波长端的颜色。

因为数据不断扩展，Amazon Redshift 提供了一个平台来存储和分析数据，并实现无缝扩展。作为第一个完全托管的 PB 级云数据仓库，Amazon Redshift 是从本地数据仓库解决方案转型而来的架构，后者需要前期资本投入、扩展工作和全职资源来操作。

Amazon Redshift 数据仓库服务兼容 ANSI SQL，专为 OLAP 工作负载设计，以压缩的列式存储格式存储数据。此服务可作为预置或无服务器数据仓库使用。

图 2-1 展示了一个*预置*集群。使用最新一代节点类型[RA3](https://oreil.ly/m-WgQ)，您可以根据工作负载独立扩展计算和存储。该节点类型的存储为*Amazon Redshift Managed Storage*（RMS），由 Amazon S3 支持。

![Amazon Redshift 架构](img/ardg_0201.png)

###### 图 2-1。Amazon Redshift 架构

*无服务器*架构（见 图 2-2）自动化了许多监视和扩展 Amazon Redshift 的操作活动。Amazon Redshift 无服务器利用基于机器学习的工作负载监控系统，自动扩展计算资源以满足工作负载的需求。随着需求随着更多并发用户和新工作负载的进展而变化，您的数据仓库会自动扩展，以提供一致的查询执行时间。最后，使用 Amazon Redshift 无服务器开始的体验也更简单；仅在使用时收费。结合 RMS 的数据共享能力，一个部门可以使用无服务器开始分析隔离的数据仓库中的数据，而不影响共享数据。由于不必支付闲置费用，您的团队无需联系管理员来暂停或恢复数据仓库。

![Amazon Redshift 无服务器架构](img/ardg_0202.png)

###### 图 2-2\. Amazon Redshift 无服务器架构

Amazon Redshift 预置和无服务器都基于包括领导节点和一个或多个计算节点的 MPP 框架。Amazon Redshift 还包括用于代码编译、查询计划缓存、结果缓存、数据湖访问、并发扩展（CS）、机器学习、联合查询和数据共享的额外组件。我们将在随后的章节详细介绍这些内容。

使用 Amazon Redshift，您可以在几分钟内开始分析数据，使用无服务器或预置数据仓库。我们将从如何创建一个无服务器数据仓库并使用样例数据集运行查询开始。

# 开始使用 Amazon Redshift 无服务器

Amazon Redshift 无服务器和预置两者在功能上有很多相同的能力，比如加载和查询结构化和半结构化数据，向 PostgreSQL 和 MySQL 中的运行数据库联邦化，以及在数据湖中查询数据。此外，无服务器和 RA3 预置数据仓库都构建在 RMS 之上，使两者都能访问其他 Amazon Redshift 数据仓库中生成的数据，无论是预置还是无服务器。尽管在选择工作负载时可能需要考虑某一种方式，但很明显，无服务器是开始使用 Amazon Redshift 的最简单方式。

## 创建 Amazon Redshift 无服务器数据仓库

亚马逊 Redshift 无服务器分为工作组和命名空间，以单独管理存储和计算资源。命名空间是包含数据库对象的集合，包括数据库、模式、表、用户、用户权限以及用于加密数据的 AWS 密钥管理服务密钥。命名空间下分组的其他资源包括数据共享、恢复点和使用限制。工作组是包含计算资源的集合，包括亚马逊 RPU 基础容量、虚拟私有云（VPC）子网组、安全组和限制。

要开始使用亚马逊 Redshift 无服务器，您可以使用 AWS 控制台、AWS CLI 或亚马逊 Redshift 无服务器 API 配置存储（命名空间）和计算（工作组）属性。

要使用 AWS 控制台部署亚马逊 Redshift 无服务器数据仓库，请导航到 [创建工作组](https://oreil.ly/fDs-g) 页面，并选择您的配置选项。在第一部分，您将选择工作组名称。接下来，您将通过选择基础 RPU 容量来确定初始计算能力。这是亚马逊 Redshift 无服务器在开始处理您的工作负载时使用的计算能力，但它可以根据您的工作负载需求自动扩展。您可以选择至少 8 个 RPUs 或多达 512 个 RPUs，默认为 128 个。有关确定您用例的最佳 RPU 大小的详细信息，请参阅 “亚马逊 Redshift 无服务器计算成本”。

最后，您将选择安全配置（参见 Figure 2-3）。这将确定数据仓库将在哪个网络配置上部署。默认情况下，无服务器数据仓库将部署在默认 VPC、子网和安全组中。您可以使用默认设置或自定义每个设置。有关网络配置考虑事项的详细讨论，请参阅 “私有/公共 VPC 和安全访问”。

![工作组名称和安全配置](img/ardg_0203.png)

###### Figure 2-3\. 工作组名称和安全配置

接下来，创建一个新的命名空间（参见 Figure 2-4）或将您的工作组附加到现有的命名空间。命名空间将包含您的数据库对象。

![命名空间](img/ardg_0204.png)

###### Figure 2-4\. 命名空间

如果创建新的命名空间，系统将提示您指定权限和管理员凭据（参见 Figure 2-5）。与预配置的集群类似，权限是通过工作组可以假定的身份和访问管理（IAM）角色定义的，以访问 AWS 环境中的其他资源。在下一个示例中，我们创建了一个名为 `RedshiftRole` 的 IAM 角色，为其分配了 `AmazonRedshiftAllCommandsFullAccess` 策略，并将其设置为默认角色。

IAM 角色中定义的权限将影响 Amazon Redshift 数据仓库可以访问的 AWS 服务，例如读取和写入 Amazon S3。这不应与分配给数据库用户、数据库组或数据库角色的权限混淆。

![权限和管理员凭据](img/ardg_0205.png)

###### 图 2-5\. 权限和管理员凭据

最后，在您的新命名空间中，指定是否要自定义加密和日志记录（图 2-6）。

![加密和日志记录](img/ardg_0206.png)

###### 图 2-6\. 加密和日志记录

完成后，您可以通过导航到 [Redshift 工作组](https://oreil.ly/G8Mdn) 页面来检索您的工作组列表（图 2-7），在该页面上您可以检索工作组端点、JDBC 和 ODBC URL 等信息。有关连接到您的工作组的选项，请参见 “连接到您的 Amazon Redshift 数据仓库”。

![工作组列表](img/ardg_0207.png)

###### 图 2-7\. 工作组列表

配置为 8 或 16 RPU 支持最多 128 TB 的 Amazon RMS 容量。如果您使用的托管存储超过 128 TB，则无法降级到少于 32 RPU。

# 示例数据

一旦您启动了 Amazon Redshift，无论是使用无服务器还是预配，您可以开始创建所需的数据模型，以构建数据仓库解决方案。

当您创建 Amazon Redshift 无服务器数据仓库时，您可以开始与三个示例数据集互动。第一个数据集 `tickit` 包含七张表：两个事实表和五个维度，如 图 2-8 所示。这个示例数据库应用帮助分析人员跟踪虚构的 Tickit 网站上的销售活动，用户在此购买和出售体育赛事、演出和音乐会的门票。特别是，分析人员可以识别随时间变化的票务流动情况，卖家的成功率，以及最畅销的事件、场馆和季节。分析人员可以利用这些信息来激励经常访问该网站的买家和卖家，吸引新用户，并推动广告和促销活动。

![Tickit 示例数据模型](img/ardg_0208.png)

###### 图 2-8\. Tickit 示例数据模型

## 激活示例数据模型并使用查询编辑器进行查询

您可以使用亚马逊 Redshift 查询编辑器 V2 来激活示例数据集并开始查询。有关如何开始使用查询编辑器 V2 的详细信息，请参阅“使用查询编辑器 V2 查询数据库”。认证后，展开 `sample_data_dev` 数据库。然后点击每个模式名称旁边的文件夹图标以打开示例笔记本（参见 Figure 2-9）。这将创建示例数据模型表、数据和与示例数据模型相关的查询。查询编辑器 V2 支持笔记本和编辑器界面。激活数据集后，示例查询也将在笔记本界面中打开，您可以查询数据。

![开始查询 tickit 数据集](img/ardg_0209.png)

###### 图 2-9\. 查询 `tickit` 示例数据集

一旦激活示例数据集，您可以开始查询数据。从笔记本提供的查询中，尝试使用 `tickit` 数据的以下示例查询。

第一个查询 (Example 2-1) 查找每个事件的总销售收入。

##### 示例 2-1\. 每个事件的总销售收入

```
SELECT e.eventname, total_price
FROM (
  SELECT eventid, sum(pricepaid) total_price
  FROM   tickit.sales
  GROUP BY eventid) Q, tickit.event E
WHERE Q.eventid = E.eventid
QUALIFY ntile(1000) over(order by total_price desc) = 1
ORDER BY total_price desc;
```

第二个查询 (Example 2-2) 检索特定日期的总销售数量。

##### 示例 2-2\. 特定日期的总销售数量

```
SELECT sum(qtysold)
FROM   tickit.sales, tickit.date
WHERE  sales.dateid = date.dateid
AND    caldate = '2008-01-05';
```

您也可以通过点击 + 按钮并选择“编辑器”菜单选项来打开查询编辑器界面（参见 Figure 2-10）。

![在编辑器模式下查询 tickit 示例数据集](img/ardg_0210.png)

###### 图 2-10\. 在编辑器模式下查询 `tickit` 示例数据集

尝试在编辑器中输入以下查询并运行以查看结果。这个查询 (Example 2-3) 检索前十个买家的总销售数量。

##### 示例 2-3\. 前十大买家的总销售数量

```
SELECT firstname, lastname, total_quantity
FROM   (SELECT buyerid, sum(qtysold) total_quantity
        FROM  tickit.sales
        GROUP BY buyerid
        ORDER BY total_quantity desc limit 10) Q, tickit.users
WHERE Q.buyerid = userid
ORDER BY Q.total_quantity desc;
```

这个查询 (Example 2-4) 查找在所有时间总销售中位于 99.9 百分位数的事件。

##### 示例 2-4\. 销售额的 99.9%事件

```
SELECT eventname, total_price
FROM  (SELECT eventid,
        total_price,
        ntile(1000) over(order by total_price desc) as percentile
       FROM (SELECT eventid, sum(pricepaid) total_price
             FROM   tickit.sales
             GROUP BY eventid)) Q, tickit.event E
       WHERE Q.eventid = E.eventid
       AND percentile = 1
ORDER BY total_price desc;
```

另外两个数据集是 `tpch` 和 `tpcds`，这些都是来自 [tpc.org](https://www.tpc.org) 的标准基准数据集。[TPC-H](https://www.tpc.org/tpch) 和 [TPC-DS](https://www.tpc.org/tpcds) 数据集是决策支持基准测试。这些包括一套面向业务的即席查询和并发数据修改。所选的查询和数据库中的数据具有广泛的行业相关性。这些基准测试提供了作为通用决策支持系统性能的代表性评估。

这些数据模型位于 `sample_data_dev` 数据库中，具有各自的 `tpch` 和 `tpcds` 模式。您可以激活这些数据模型，并通过点击模式名称旁边的文件夹图标来访问每个模式的相关对象，就像您之前为 `tickit` 模式所做的那样。这将在一个笔记本界面中打开所有查询（参见 Figure 2-11）。现在您可以尝试运行示例查询。第一个查询返回已开票、已发货和已退货的业务金额。

![在 tpch 数据集上尝试示例查询](img/ardg_0211.png)

###### 图 2-11\. 查询 `tpch` 样本数据集

# 何时使用预置集群？

亚马逊 Redshift 提供了第二种部署选项，您可以对数据仓库具有更多控制。设置亚马逊 Redshift 数据仓库时的关键考虑因素是选择最适合您工作负载的配置和架构，以获得最佳的开箱即用性能。

根据来自亚马逊创始人杰夫·贝索斯于 2020 年 11 月 23 日的一篇文章：

> 有两种类型的决策。有些决策是不可逆转且影响深远的；我们称之为单向门 […​] 它们需要慢慢和仔细地作出。在亚马逊，我经常扮演首席减速官的角色：“哇，我想看到这个决策再分析十七种方式，因为它具有高度重大且不可逆的影响。” 问题在于大多数决策不是这样的。大多数决策是双向门。

正如贝索斯所述，选择无服务器和预置之间是一个双向门决策，即使您没有选择最佳配置，因为亚马逊 Redshift 是按使用付费的云解决方案，可以在几分钟内增加或删除容量。您的分析架构设计应基于快速实验和验证。

预置集群提供了调整数据仓库性能的能力。使用预置集群，您可以控制何时以及如何调整集群大小，有能力手动配置工作负载管理，并确定何时暂停/恢复数据仓库。虽然预置集群可能需要更多的管理，但它提供了更多调整可预见工作负载和优化成本的选项。如果您有非常一致和恒定的工作负载，利用预置集群和购买预留实例可能是更为成本优化的选择。

在设计生产数据仓库的架构时，您有许多选择。您需要决定是否有意义在一个亚马逊 Redshift 数据仓库中运行工作负载，还是将工作负载拆分/隔离到多个亚马逊 Redshift 数据仓库中。您还需要决定在一个或多个数据仓库中使用预置还是无服务器是否有意义。在下面的示例中，展示了支持相同工作负载的两种不同策略（图 2-12）。

![Redshift 部署选项](img/ardg_0212.png)

###### 图 2-12\. 亚马逊 Redshift 部署选项

在第一章“用于数据的 AWS”中，您学习了现代数据架构如何鼓励分布式团队独立拥有和设计其面向领域的解决方案。在以下的亚马逊 Redshift 数据网格架构（图 2-13）中，您可以看到一种既利用了配置的数据仓库，又利用了多个无服务器数据仓库的架构。在这种环境中，您使用配置的数据仓库（1）从多个来源进行数据摄取，因为工作负载配置是一致的，并且在大部分时间内运行。您可以购买预留实例，并完全控制可预测的成本。此外，您设置了一个无服务器数据仓库（2），它可以使用数据共享从配置的数据仓库中读取数据。此服务器无服务器数据仓库的用户也从共享数据中读取数据，并通过加载自己的数据、连接到共享数据并根据需要聚合数据来整理新的数据集。这些无服务器数据仓库的用户成为其领域或部门特定数据的管理者。他们可以访问其领域之外的任何数据，但不依赖于该组织的处理需求。最后，您设置了另一个无服务器数据仓库（3），它也使用数据共享从配置的数据仓库中读取来自另一个无服务器数据仓库（2）的整理数据集。这些工作负载在计算需求、运行时间以及活动时间长度等方面各不相同。

![Redshift 数据网格架构](img/ardg_0213.png)

###### 图 2-13\. Redshift 数据网格架构

无论您选择哪种部署选项，都可以使用快照/恢复过程切换。有关如何将快照从配置的集群恢复到无服务器数据仓库的详细信息，请参阅[在线文档](https://oreil.ly/a66BT)。

## 创建 Amazon Redshift 配置集群

使用配置选项部署 Amazon Redshift 意味着您将部署特定节点类型的一定数量的计算节点，形成一个集群。要部署 Amazon Redshift 配置集群，请导航至[创建集群](https://oreil.ly/2xS0U)页面并选择您的配置选项。您还可以按照 AWS 文档中描述的步骤来[创建样例 Amazon Redshift 集群](https://oreil.ly/ptbqk)。

在第一部分中，您将选择集群名称和大小（图 2-14）。有关确定适用于您用例的最佳集群大小的详细信息，请参阅“Amazon Redshift 配置计算成本”。

![集群名称和大小](img/ardg_0214.png)

###### 图 2-14\. 集群名称和大小

接下来，您将决定是否加载样例数据（可选）并设置管理员凭据（图 2-15）。

![集群载入示例数据并设置管理员凭证](img/ardg_0215.png)

###### 图 2-15\. 载入示例数据并设置管理员凭证

接下来，指定集群权限（图 2-16）通过为您的亚马逊 Redshift 集群分配 IAM 角色，以访问 AWS 环境中的其他资源。如果您打算执行诸如从 Amazon S3 批量加载数据到 Amazon Redshift 的操作，则需要这些权限。提供了一个名为 `AmazonRedshiftAllCommandsFullAccess` 的托管策略，可用于关联到包含您可能使用的常见服务的角色。在下一个示例中，我们创建了一个 IAM 角色 `RedshiftRole`，分配了 `AmazonRedshiftAllCommandsFullAccess` 策略，并将其设为默认角色。有关授权亚马逊 Redshift 代表您访问其他亚马逊服务的更多详细信息，请参阅[在线文档](https://oreil.ly/zjIDn)。

![集群权限](img/ardg_0216.png)

###### 图 2-16\. 集群权限

最后，设置集群的附加配置（图 2-17）。这些配置决定了您的集群将在您网络配置的上下文中部署的位置。默认情况下，集群将部署在默认的 VPC、子网和安全组中。您可以使用默认设置或自定义每个设置。有关网络配置考虑事项的详细讨论，请参阅“私有/公共 VPC 和安全访问”。

![集群附加配置](img/ardg_0217.png)

###### 图 2-17\. 集群附加配置

您可以从[Redshift 集群](https://oreil.ly/Sgl51)页面查看您的集群列表（图 2-18）。单击集群，您可以获取一般信息，如集群终端节点以及 JDBC 和 ODBC URL。有关连接到集群的选项，请参阅“连接到您的亚马逊 Redshift 数据仓库”。

![集群列表](img/ardg_0218.png)

###### 图 2-18\. 集群列表

# 估算您的亚马逊 Redshift 成本

当估算亚马逊 Redshift 的成本时，需要考虑的两个主要因素是存储和计算。对于 RA3 节点类型和无服务器的情况，您必须单独考虑托管存储成本，而不是无服务器计算成本或预置计算成本。然而，如果您使用 DC2 节点类型，则只需考虑预置计算成本，因为所有存储都是局部存储在计算节点上。

## 亚马逊 Redshift 托管存储

Amazon RMS 是一种压缩且列格式化的数据结构，专为分析工作负载的最佳性能而设计。存储与计算分离，并且具有弹性。您可以将数据从 TB 扩展到 PB。无论 Amazon Redshift 数据仓库是 RA3 预配还是无服务器，它都可以共享。即使 Amazon Redshift 数据仓库处于非活动状态，共享数据也对消费者可用。例如，如果数据的主要所有者是预配集群，则消费者可以访问该数据，即使预配集群暂停。此功能使您可以选择适合工作负载的计算而无需移动或转换数据。它还确保数据不会重复，从而减少维护和存储成本。由于您可以轻松地在不同的计算选项之间移动，并且即使计算未运行，存储也可用，因此存储的定价是独立的。要估算使用 RA3 集群或无服务器数据仓库时的存储成本，您可以估算压缩存储需求乘以每月的存储价格。例如，假设您的存储使用量为 10 TB。基于每 GB 每月$0.024 的定价，您的存储费用将为$240：

```
$0.024*10*1000 = $240
```

获取关于亚马逊 Redshift 托管存储定价的最新详细信息，请参阅[在线文档](https://oreil.ly/QsZE6)。

## 亚马逊 Redshift 无服务器计算成本

对于亚马逊 Redshift 无服务器，除了 RMS 成本外，还有一个关键成本需要考虑：计算成本。为了便于亚马逊 Redshift 计费，创建了一种称为 RPU 的计量单位（图 2-19），表示在给定时间段内使用的计算能力。设置工作组时，会配置 128 个 RPUs 的基础容量。当亚马逊 Redshift 无服务器从空闲状态唤醒并需要更多计算时，将会扩展此基础。您可以通过编辑亚马逊 Redshift 无服务器配置并修改`Limits`来修改工作组的基础容量。

![亚马逊 Redshift 无服务器基础容量](img/ardg_0219.png)

###### 图 2-19\. 亚马逊 Redshift 无服务器基础容量

每个查询都会在系统中记录，只有在服务器无服务器工作组运行查询的时间段内才会收费（交易的最低计费时间为 60 秒）。例如，假设您使用了默认的 128 RPUs，并且在一个小时内只提交了 1 个运行 90 秒的查询。根据 us-west-2 地区的定价$0.45/RPU 小时，您的查询费用将为$1.44：

```
rate/sec = $0.45*128/60/60 = $0.016
secs = max(90,60) = 90
rate*secs = $0.016*90 = $1.44
```

假设这实际上是一个按计划触发执行 3 个同时查询（15 秒、30 秒和 90 秒）的仪表板，其中最长的查询在 90 秒内完成。还假设这是该小时工作组的唯一工作负载。您仍将仅收取$1.44，因为工作组仅在这 90 秒内运行，而无服务器工作组能够使用 128 RPUs 的基本容量完成作业：

```
rate/sec = $0.45*128/60/60 = $0.016
secs = max(15,30,90,60) = 90
rate*secs = $0.016*90 = $1.44
```

### 设置不同的基本容量值

假设您配置工作组的基本容量为 8 RPUs 或 1/16 的计算能力而不是 128 RPUs。很可能，90 秒完成的工作量将需要长达 16 倍的时间（1440 秒），但价格仍将为$1.44：

```
rate/sec = $0.45*8/60/60 = $0.001
secs = max(1440,60) = 1440
rate*secs = $0.001*1440 = $1.44
```

如果像 8 RPUs 这样的小基本容量配置，如果查询使用了大量内存，工作负载可能会比 1440 秒更长，因为可用内存更少，您将进行磁盘分页。但是，在某些情况下，工作负载可能需要的时间较短，因为原始 90 秒可能包含了不需要推算的开销。

在选择基本容量时的另一个考虑因素是最低收费 60 秒。如果您有很多少于 60 秒的查询，并且在查询之间有空闲时间，则较小的基本容量配置上的每个查询费用将较低。

在以下示例中，假设您有一个查询，在 128 RPU 基本容量配置上运行 1 秒钟，在 8 RPU 基本容量配置上运行 16 秒钟。如果这是该分钟内唯一运行的查询，则它们各自将按其各自的最低收费标准收费，导致价格相差 16 倍：

```
8 RPU rate/sec = $0.001
secs = max(1,60) = 60
rate*secs = $0.060

128 RPU rate/sec = $0.016
secs = max(16,60) = 60
rate*secs = $0.960
```

### 高/频繁使用

在另一个例子中，假设您有一个流媒体应用程序，每分钟加载到您的数据仓库，但每个加载只需 5 秒钟（图 2-20）。因为无服务器工作组上的每个事务都按照最低 60 秒计费，即使每个查询只需 5 秒完成，也将基于 60 秒的最低收费计费。

![流媒体使用时间轴](img/ardg_0220.png)

###### 图 2-20\. 流媒体使用时间轴

在这个例子中，虽然总的查询执行时间是`5*60=300`秒，但应用了 60 秒的最低收费时，计费使用时间为`60*60=3600`秒。基于 us-west-2 地区的价格为$0.45/RPU 小时，您的这个工作负载的费用将为$57.60：

```
rate/sec = $0.45*128/60/60 = $0.016
secs = max(5,60)*60 = 60*60 = 3600
rate*secs = $0.016*3600 = $57.60
```

在这些情况中，建议在满足查询服务水平协议（SLAs）和预算要求的基本容量配置上进行实验，记住您只需支付您使用的计算费用。

关于 Amazon Redshift 定价的最新详细信息，包括使用预留容量可以节省多少费用，请查看[在线文档](https://oreil.ly/9kwzy)。

## Amazon Redshift 预留计算成本

对于 RA3 预留集群，除了存储成本外，主要额外成本来自集群大小。规划预留数据仓库的第一步是确定节点类型和您所需的节点数。在确定 Amazon Redshift 预留集群大小时，请考虑能够满足您分析处理需求性能 SLA 的稳态计算。每种节点类型由一个计算配置文件定义，并且随着节点类型的选择越来越大和更昂贵，您可以选择最适合满足您需求的节点类型和节点数量。下表（表 2-1）总结了每种节点类型的分配资源，截至 2023 年。您可以获取最新的计算配置文件，并在[在线文档](https://oreil.ly/hDesT)中了解更多有关不同节点类型的信息。

表 2-1\. RA3 节点类型

| 节点类型 | RMS | 内存 | vCPU |
| --- | --- | --- | --- |
| ra3.xlplus | 32 TB | 32 GB | 4 |
| ra3.4xlarge | 128 TB | 96 GB | 12 |
| ra3.16xlarge | 128 TB | 384 GB | 48 |

在确定集群中需要的节点大小和节点数时，请考虑您的处理需求。从最大的节点大小开始，当您超出性能 SLA 时考虑更小的节点大小。这种策略有助于减少节点之间数据洗牌时的网络流量。例如，ra3.16xl 的 2 个节点在 vCPU 和内存方面等效于 ra3.4xlarge 的 8 个节点和 ra3.xlplus 节点类型的 24 个节点。如果您从 ra3.16xlarge 节点类型开始，并发现您远远超过了性能 SLA，并且集群 CPU 利用率低，您可以利用调整选项将集群调整为较小的节点类型。有关调整选项的更多信息，请参阅[在线文档](https://oreil.ly/sRQdC)。

如果您尚未运行工作负载并需要初始集群大小，则 Amazon Redshift 控制台提供了一个工具（图 2-21），帮助您选择集群大小，考虑参数如所需存储量及工作负载。

![帮助我选择](img/ardg_0221.png)

###### 图 2-21\. 帮助我选择

Amazon Redshift 提供的预留集群的关键方面是，可以按需计费，也可以购买预留实例，这样您就可以获得固定成本和深度折扣。在“帮助我选择”工具中，提供了计算成本摘要（图 2-22）。

承诺 1 年预订可获得 33%的折扣，承诺 3 年预订可获得 61%的折扣。

![样本成本估算](img/ardg_0222.png)

###### 图 2-22\. 样本成本估算

对于 Amazon Redshift 提供的集群，有一些功能会导致额外的变动成本，相比之下，在 Amazon Redshift 无服务器中作为 RPUs 打包。首先是名为*并发扩展*的功能，在高峰活动期间，会自动为您的数据仓库提供临时集群。其次是名为*Amazon Redshift Spectrum*的功能，利用一个独立的计算机群来查询 Amazon S3 数据湖中的数据。在第五章，“扩展和性能优化”中，我们将详细讨论并发扩展，在第三章，“设置数据模型和数据摄入”和第四章，“数据转换策略”中，我们将详细讨论如何查询外部数据。现在我们提到它们是为了强调您如何能够控制这些功能并拥有更可预测的成本结构。

# AWS 账户管理

您需要一个 AWS 账户来启动任何 AWS 服务，包括 Amazon Redshift。AWS 账户由创建该账户的根用户拥有。创建 AWS 账户后，您可以使用*身份和访问管理*（IAM）服务创建额外的用户并分配权限给 IAM 用户。我们提到 AWS 账户，因为与您启动的任何其他 AWS 服务一样，对于 Amazon Redshift，您可以通过在不同的账户中启动数据仓库来设立边界，例如开发与生产（见图 2-23）。

![AWS 控制塔，AWS 组织和 AWS 账户](img/ardg_0223.png)

###### 图 2-23\. AWS 控制塔，AWS 组织和 AWS 账户

AWS 组织（[AWS Organizations](https://oreil.ly/tvrqD)）帮助您集体管理多个 AWS 账户，分配资源，将账户分组为组织单位（OUs），在这些单位上应用治理和安全策略，并通过指定一个管理账户简化您的组织计费，以便利用单一账单的数量折扣。其他 AWS 账户是成员账户。

通过设置一个指定的安全 OU，并为审计单独设置一个隔离账户，将使您的安全团队能够快速审核整个组织中的所有访问。单独的日志记录账户将允许您集中所有日志文件，当您能够将所有数据关联在一起时，可以轻松地排除端到端应用程序问题。请参阅[“使用 AWS 组织的最佳实践设置多账户 AWS 环境”](https://oreil.ly/tvrqD)观看短视频。

不必手动设计 AWS 组织和 AWS 账户，您可以利用 AWS 控制塔服务。AWS 控制塔是一个独立的服务，提供预设体验，根据最佳实践蓝图自动设置您的 AWS 组织和 AWS 账户。它还使用预打包的服务控制策略（SCP）和治理规则，用于安全、运营和合规性。

无论您是在 AWS 上全新启动还是已有现有的多账户 AWS 环境，都可以使用 AWS Control Tower。AWS Control Tower 自动设置着陆区，以便将外部文件引入到您的分析环境中，为持续治理应用保护栏杆，自动化供应工作流程，并为 OU、账户和保护栏杆提供预打包仪表板以实现可见性。有关更多详情，请参阅短视频[“什么是 AWS Control Tower？”](https://oreil.ly/7zyKO)。

# 连接到您的亚马逊 Redshift 数据仓库

一旦确定了部署架构，接下来您应该问自己的问题是如何控制访问以满足安全需求。今天 IT 组织面临的固有挑战是确保数据平台的访问既安全又灵活且易于管理。管理员需要能够为所有需要访问的人提供访问和工具，但必须符合公司政策并通过批准的网络途径进行访问。根据您需要管理的用户数量以及是否已有用于用户管理、身份验证和授权的现有基础设施，您可以选择一种策略而非另一种。一旦连接，用户的身份将存储在 Amazon Redshift 元数据中。该身份可以分配给一个或多个组，并且将为该用户的授权和活动记录日志以进行追踪。Amazon Redshift 包含不同的功能，可以实现安全和灵活的访问，有许多考虑因素可供选择哪些选项进行利用。此外，AWS 提供了多种工具来访问您的 Amazon Redshift 数据仓库，以及如何利用这些安全功能的多种选项。

## 私有/公有 VPC 和安全访问

无论您选择无服务器或预置，Amazon Redshift 始终部署在 VPC 和子网中。根据这一选择，数据仓库只能由内部资源访问，或者可以由公共互联网上的资源访问。有关为您的组织确定最佳 VPC 策略的更多详细信息，请阅读[在线文档](https://oreil.ly/TKM1N)中关于 VPC 场景的内容。此外，当您使用基于 JDBC 或 ODBC 驱动程序的工具连接到 Amazon Redshift 数据仓库时，您是通过 TCP/IP 连接到特定的主机名和端口号。与部署到 VPC 的任何 AWS 服务一样，您可以通过自定义入站规则来控制到 Amazon Redshift 的网络流量，以适配与您的数据仓库关联的安全组。有关理解安全组和设置入站规则的更多详细信息，请参阅[在线文档](https://oreil.ly/EArzG)。最后，您可能会遇到 Amazon Redshift 需要连接到运行在您的 VPC 外部或另一个 AWS 帐户中的 AWS 服务的情况。对于这些场景，VPC 端点可能是合适的选择。有关 VPC 端点的更多信息，请参阅[在线文档](https://oreil.ly/Rw5gK)。

包含带有 AWS Direct Connect 的私有子网的以下示例架构（图 2-24）通常由企业用户使用，因为它包含限制只允许企业基础设施内部人员或通过 VPN 服务外部访问的控制措施。为确保与 AWS 云的高带宽和安全连接，该架构利用 AWS Direct Connect。虚拟私有网关确保传输的数据安全且加密。正确设置后，AWS 云内的资源（如 Amazon Redshift 数据仓库和分析工具）可通过私有 IP 地址访问，无论用户是否在内部或外部客户端机器上。

在 AWS 中，Amazon Redshift 部署在私有子网中，这意味着它没有直接的互联网连接，对环境的任何访问都必须起源于 VPC 内部或通过虚拟私有网关。然而，有一些 AWS 服务并不在您的 AWS VPC 内运行。Amazon Redshift 中用于 `COPY` 和 `UNLOAD` 命令的关键服务是 Amazon S3 服务。为确保从 Amazon S3 传输到 Amazon Redshift 的数据通过您的 VPC，已启用增强型 VPC 路由功能，并配置了私有子网，该子网引用了指向 Amazon S3 的 VPC 端点的路由表。有关如何配置增强型 VPC 路由的详细信息，请参阅[在线文档](https://oreil.ly/yRMs-)。

![Private Subnet DirectConnect](img/ardg_0224.png)

###### 图 2-24\. 带有 AWS Direct Connect 的私有子网

## 存储密码

在 Amazon Redshift 数据仓库中维护用户的最简单机制是创建一个本地用户并在 Amazon Redshift 元数据中存储密码。这种策略几乎适用于任何数据库平台，并且可以简单地使用[`CREATE USER`](https://oreil.ly/Qmq49)命令来完成。例如：

```
CREATE USER name PASSWORD 'password';
```

当您要维护的用户非常少时，通常会使用此策略，但是随着用户群体的增加，可能会产生管理开销。

## 临时凭证

要考虑的下一个概念，仅适用于 Amazon Redshift 预置集群的是临时凭证。这种策略是基于 API 函数[`GetClusterCredentials`](https://oreil.ly/ABhoZ)的。API 调用将以`DbUser`和`DbGroups`参数作为输入参数，允许您加入一个或多个数据库组进行授权。API 调用还接受一个`AutoCreate`参数，当设置时，如果数据库中不存在用户，则会创建一个用户。API 将返回一个用户提供的临时密码。一旦检索到临时密码，您可以使用用户名和临时密码组合登录。

要为预置数据仓库使用临时凭证策略，用户需要首先进行身份验证并关联到 IAM 身份，可以是 IAM 用户或作为 IAM 角色登录。该身份需要具有允许用户使用`redshift:GetClusterCredentials`操作的 IAM 策略。要启用创建新用户和动态加入组等功能，可以添加`redshift:CreateUser`和`redshift:JoinGroup`权限。为了确保用户不能为任何用户获取临时密码或加入他们不应该加入的数据库组，建议缩小策略范围并添加一个条件，确保他们获取临时凭证的用户名与其身份匹配。以下是授予用户访问`dev`数据库和加入`marketing`组的示例策略。还有一个条件，确保`aws:userid`与传递给`GetClusterCredentials` API 命令的`DbUser`匹配：

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "redshift:GetClusterCredentials",
        "redshift:CreateUser",
        "redshift:JoinGroup"
        ],
      "Resource": [
        "arn:aws:redshift:*:123456789012:cluster:rs-cluster",
        "arn:aws:redshift:*:123456789012:dbuser:rs-cluster/${redshift:DbUser}",
        "arn:aws:redshift:*:123456789012:dbname:rs-cluster/dev",
        "arn:aws:redshift:*:123456789012:dbgroup:rs-cluster/marketing"
        ],
      "Condition": {
        "StringLike": {
          "aws:userid": "*:${redshift:DbUser}"
        }
      }
    }
  ]
}
```

除了允许已通过 IAM 身份登录到 AWS 的用户查询 Amazon Redshift 外，该策略还内置于 JDBC 和 ODBC 驱动程序中。您只需向驱动程序指示将使用 IAM 进行身份验证并将其 IAM 身份传递给它即可。更多关于如何配置 JDBC 或 ODBC 连接以使用 IAM 凭证的信息，请参阅[在线文档](https://oreil.ly/2nkuO)。

通过传递`AccessKeyID`和`SecretAccessKey`来传递 IAM 身份的简单方法。请参阅以下示例 JDBC URL：

```
jdbc:redshift:iam://examplecluster:us-west-2/dev?
    AccessKeyID=AKIAIOSFODNN7EXAMPLE&
    SecretAccessKey=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

## 联合用户

下一个需要考虑的概念是*联合用户*。此策略基于 API 函数[`GetClusterCredentialsWithIAM`](https://oreil.ly/Ygkg1)（针对预配）和[`GetCredentials`](https://oreil.ly/dlVqQ)（针对无服务器）实现。类似于 API 调用`GetClusterCredentials`，这些 API 将获取用户的临时密码；不过，与传递这些 API 的授权参数不同，API 将根据登录的身份读取值。在无服务器情况下，它将从`aws:userid`检索用户名，并将从主体标签`RedshiftDbRoles`中检索数据库角色以授权会话。在预配情况下，它仅检索存储在`aws:userid`中的用户名。在两种情况下，默认情况下，如果用户不存在，将创建用户。一旦获取临时密码，用户可以使用用户名和临时密码组合登录。

类似于`GetClusterCredentials`，要使用联合用户，用户首先需要进行身份验证，并关联到 IAM 身份，可以是 IAM 用户或作为 IAM 角色登录。该身份需要具有 IAM 策略，允许用户对预配执行`redshift:GetClusterCredentialsWithIAM`操作和对无服务器执行`redshift-serverless:GetCredentials`操作。使用`联合用户`选项时，您无需任何额外权限。在无服务器情况下，如果要启用基于角色的授权，则需要确保使用的 IAM 身份具有设置`RedshiftDbRoles`主体标签。以下是授予用户访问`dev`数据库的示例策略：

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action":
        "redshift-serverless:GetCredentials",
      "Resource":
        "arn:aws:redshift-serverless:*:123456789012:workgroup/rs-workgroup",
    }
  ]
}
```

类似于`GetClusterCredentials` API，联合用户功能已经内置于 JDBC 和 ODBC 驱动程序中，并且在连接到无服务器数据仓库时默认使用，只要在驱动程序中指定将使用 IAM 进行身份验证。了解如何配置 JDBC 或 ODBC 连接以使用 IAM 凭据，请参阅[在线文档](https://oreil.ly/2nkuO)。

## 基于身份提供者的基于 SAML 的身份验证

通常情况下，对于企业和大规模的 Amazon Redshift 部署，您的用户身份不会存储在 IAM 中，而是存储在身份提供者（IdP）如 Okta、Azure Ad 或 PingFederate 中。如果是这种情况，将 IAM 或 Amazon Redshift 元数据中的单独目录保持一致并非可扩展的解决方案。此外，用户可能属于多个组，并为每个组合维护单独的 IAM 角色将变得难以管理。AWS 平台本地支持使用[`AssumeRoleWithSAML`](https://oreil.ly/WUg3v) API 命令从 IdP 建立 IAM 身份。通过利用这一策略，可以假定包含执行`GetClusterCredentials`权限的 IAM 角色。此功能已内置于 Amazon Redshift JDBC 和 ODBC 驱动程序中。在图 2-25 中，您可以看到 SQL 客户端如何与 IdP 和 AWS 服务（如 AWS 安全令牌服务（AWS STS）和 IAM）交互，然后才能连接到 Amazon Redshift。

![SAML 认证](img/ardg_0225.png)

###### 图 2-25\. 安全断言标记语言（SAML）认证

查看[在线文档](https://oreil.ly/RqO_P)以详细了解不同的 IdP 和支持的功能，如多因素身份验证。

想要设置 Okta IdP 的逐步指南，请参阅使用`OktaCredentialsProvider`插件的[博客文章](https://oreil.ly/mTqC3)。对于需要多因素身份验证的场景，请参阅同样使用 Okta IdP 但利用`BrowserSamlCredentialsProvider`插件的[博客文章](https://oreil.ly/pQPST)。

无论您使用 Okta 还是其他 IdP，所需的步骤包括：

1.  在您的 IdP 中创建一个供用户访问的 SAML 应用程序。

1.  配置应用程序引用 IAM 角色并传递用户和组信息。

1.  从您的 IdP 下载应用程序元数据。

1.  在 AWS 控制台内使用刚刚下载的元数据建立一个 IdP。

## 本地 IdP 集成

当应用程序通过 OAuth 令牌建立信任关系时，Amazon Redshift 具有额外的插件，例如`BrowserAzureOAuth2CredentialsProvider`。与利用 API 命令或 IAM 建立信任关系不同，Amazon Redshift 驱动程序将发起初始请求到身份提供者以验证凭据，并弹出多因素认证提示（如果需要），并接收 OAuth 令牌。接下来，Amazon Redshift 服务将使用 OAuth 令牌向 IdP 发起进一步调用以获取组授权信息。本地 IdP 架构（图 2-26）概述了在使用 Azure Active Directory 作为身份提供者时，Microsoft Power BI 如何集成以及本地集成的工作原理。

![本地 IdP 架构](img/ardg_0226.png)

###### 图 2-26\. 本地 IdP 架构

若要详细了解如何设置与 Power BI 和活动目录的本地 IdP 集成，请参阅此 [博客文章](https://oreil.ly/fJFvP)。

## 亚马逊 Redshift 数据 API

连接到亚马逊 Redshift 并查询数据的另一种方法是使用亚马逊 Redshift 数据 API（图 2-27）。使用此策略，用户不直接连接到亚马逊 Redshift，而是连接到安全的 HTTP 端点。您可以使用端点运行 SQL 语句，而无需管理连接。Data API 的调用是异步的。使用此策略，应用程序需要访问互联网，或者如果您正在从私有 VPC 运行应用程序，则可以设置 VPC 端点。有关设置 VPC 端点的详细信息，请参阅此 [在线文档](https://oreil.ly/jaqge)。

![亚马逊 Redshift 数据 API](img/ardg_0227.png)

###### 图 2-27\. 亚马逊 Redshift 数据 API

要使用亚马逊 Redshift 数据 API，用户需要首先通过 AWS 进行身份验证，并与 IAM 身份相关联，可以是 IAM 用户或以 IAM 角色身份登录。该角色将需要权限来使用亚马逊 Redshift 数据 API。权限以 `redshift-data:` 为前缀，包括元数据查询如 `ListTables` 和 `ListSchemas`，执行命令如 `ExecuteStatement` 和 `GetStatementResults`，以及监控查询如 `DescribeStatement`。可在 [在线文档](https://oreil.ly/PnVAL) 中查看详细的命令列表。

在使用亚马逊 Redshift 数据 API 时，您有两种身份验证选项。首先是使用临时凭证，在调用 API 时传递 `DbUser` 参数，但不传递 `SecretArn` 参数。要了解更多关于使用临时凭证的信息，请参见 “临时凭证”。相反，您可以通过传递 `SecretArn` 参数而不传递 `DbUser` 参数来使用存储的密码进行身份验证。要了解更多关于使用存储密码的信息，请参见 “存储的密码”。

在以下示例中，您可以看到如何利用亚马逊 Redshift 数据 API 执行一个简单的 `CREATE SCHEMA` 命令，使用存储在密钥中的凭证：

```
aws redshift-data execute-statement \
     --database <your-db-name>  \
     --cluster-identifier <your-cluster-id> \
     --secret-arn <your-secret-arn> \
     --sql "CREATE SCHEMA demo;" \
     --region <your-region>
```

要详细了解如何使用亚马逊 Redshift 数据 API，请参阅此 [博客文章](https://oreil.ly/qsFf6)。

## 使用查询编辑器 V2 查询数据库

在确定了身份验证策略之后，下一步是查询您的数据。亚马逊 Redshift 提供两个版本的查询编辑器——传统查询编辑器和查询编辑器 V2（图 2-28）。您可以使用它们来编写和运行对亚马逊 Redshift 数据仓库的查询。虽然传统查询编辑器仍然可用，但我们建议使用查询编辑器 V2，因为它具有额外的功能，允许您管理数据库对象、可视化结果，并与团队共享工作，除了编辑和运行查询。它显示数据库、模式和树形视图面板中的所有对象，便于访问数据库对象。

![查询编辑器 V2](img/ardg_0228.png)

###### Figure 2-28\. 查询编辑器 V2

查询编辑器 V2 有两种标签类型（Figure 2-29）：编辑器标签和笔记本标签。编辑器标签允许您在一个页面中整理所有查询，并同时触发执行。编辑器将按顺序执行所有查询，并在不同的结果标签中生成结果。SQL 笔记本标签包含 SQL 和 Markdown 单元格，您可以在单个文档中组织、注释和共享多个 SQL 命令。编辑器脚本和笔记本均可保存并与团队共享，以便进行协作。在本书中，我们将使用查询编辑器 V2，因为这是在 Amazon Redshift 中查询数据的未来方向。

![查询编辑器 V2 标签类型](img/ardg_0229.png)

###### Figure 2-29\. 查询编辑器 V2 标签类型

要在 AWS 控制台内访问查询编辑器 V2，请单击查询数据按钮下方的链接（Figure 2-30）。

![查询数据](img/ardg_0230.png)

###### Figure 2-30\. 查询数据

要允许用户访问查询编辑器 V2，您可以将 AWS 管理的查询编辑器 V2 策略之一（Table 2-2）附加到 IAM 用户或角色上。这些托管策略还提供对其他所需服务的访问。如果您希望为最终用户自定义权限，也可以创建用户管理的策略。

Table 2-2\. 查询编辑器 V2 策略

| 策略 | 描述 |
| --- | --- |
| AmazonRedshiftQueryEditorV2FullAccess | 授予对查询编辑器 V2 操作和资源的完全访问权限。这主要用于管理员。 |
| AmazonRedshiftQueryEditorV2NoSharing | 允许在不共享资源的情况下使用查询编辑器 V2。用户不能与团队成员共享他们的查询。 |
| AmazonRedshiftQueryEditorV2ReadSharing | 允许在有限共享资源的情况下使用查询编辑器 V2。授权的主体可以读取与团队共享的保存查询，但不能更新它们。 |
| AmazonRedshiftQueryEditorV2ReadWriteSharing | 允许在共享资源的情况下使用查询编辑器 V2。授权的主体可以读取和更新团队共享的资源。 |

要实现团队成员之间的查询协作，您可以对 IAM 主体进行标记。例如，如果您有一组用户作为`marketing_group`的一部分，并且希望他们通过共享他们的查询进行协作，您需确保他们的 IAM 角色`marketing_role`被分配了`AmazonRedshiftQueryEditorV2ReadSharing`策略。您还可以使用标签`sqlworkbench-team`为角色打标签，其值为`marketing_group`。现在，使用`marketing_role`登录的最终用户可以访问查询编辑器 V2，并具有共享其查询的能力。

当使用查询编辑器连接到您的 Amazon Redshift 数据仓库时，您将看到几个选项。您可以作为“联合用户”连接，使用“临时凭证”，通过“数据库用户名和密码”，或通过“AWS Secrets Manager”连接。

查询编辑器 V2 中的临时凭证选项仅在连接到已配置的集群时可用。

### 联合用户

当使用查询编辑器 V2 并选择联合用户选项（图 2-31）时，对于已配置与无服务器数据仓库不同。对于已配置，查询编辑器 V2 将依赖于临时凭证认证方法中使用的 `GetClusterCredentials`。然而，与其要求用户和组信息不同，它将从两个主体标签 `RedshiftDbUser` 和 `RedshiftDbGroups` 中查找这些值，并将这些标签中的值传递给 API 调用。这些主体标签可以直接在 IAM 中设置，也可以从 IdP 传递。要使用此策略，从 IdP 传递的 IAM 角色还需要具有使用临时凭证的权限，如前所述。使用此策略既具有可扩展性又易于使用，因为终端用户唯一需要输入的是数据库名称。有关如何设置 Okta 联合用户登录的详细步骤，请参阅此[博客文章](https://oreil.ly/70qK6)。

![查询编辑器 V2 联合用户](img/ardg_0231.png)

###### 图 2-31\. 查询编辑器 V2 联合用户

相反，当选择联合用户（图 2-31）用于无服务器数据仓库时，查询编辑器 V2 将利用联合用户认证方法中使用的 `GetCredentials` API 调用。类似地，数据库用户名将从 `aws:userid` 中检索，数据库角色将从 `RedshiftDbRoles` 主体标签中检索。

### 临时凭证

临时凭证选项（图 2-32）与联合用户选项类似，即 IAM 身份需要具有使用临时凭证的权限。一个显著的区别是它不依赖于主体标签，因此除了 `Database` 参数外，用户还必须输入用户名，并且不会自动添加到组中。

![查询编辑器 V2 临时凭证](img/ardg_0232.png)

###### 图 2-32\. 查询编辑器 V2 临时凭证

### 数据库用户名和密码

使用密码选项（图 2-33）时，除了 `Database` 和 `User name` 参数外，用户还必须提供密码。为了在后续会话中更容易使用，密码将保存在 Secrets Manager 服务中，并且使用查询编辑器的 IAM 身份需要具有读写 Secrets Manager 的权限。

![查询编辑器 V2 存储密码](img/ardg_0233.png)

###### 图 2-33\. 查询编辑器 V2 存储密码

### AWS Secrets Manager

使用 AWS Secrets Manager 选项（图 2-34），用户只需指定预定义的 AWS 秘密。在这种情况下，该秘密将由管理员预先创建，因此使用查询编辑器的 IAM 身份*不*需要权限写入 Secrets Manager。

![Query Editor V2 AWS Secrets Manager](img/ardg_0234.png)

###### 图 2-34\. 查询编辑器 V2 AWS Secrets Manager

若要了解更多有关查询编辑器 V2 的示例，请参阅以下博客文章：

+   [“使用 Amazon Redshift Query Editor V2 简化数据分析”](https://oreil.ly/wBMv-)

+   [“介绍 Amazon Redshift Query Editor V2，一款供数据分析师使用的免费 Web 查询创作工具”](https://oreil.ly/spxKw)

+   [“使用 Active Directory 联合服务（AD FS）将 Amazon Redshift Query Editor V2 联合访问”：第三部分](https://oreil.ly/ihsKQ)

## 使用 Amazon QuickSight 进行商业智能分析

您可以使用 BI 平台通过 JDBC 和 ODBC 驱动程序连接到 Amazon Redshift，这些驱动程序可以作为这些应用程序的一部分打包或下载。与 Amazon Redshift 集成的流行 BI 平台包括[MicroStrategy](https://www.microstrategy.com)，[Power BI](https://powerbi.microsoft.com)，[Tableau](https://www.tableau.com)和[Looker](https://www.looker.com)。有关使用这些驱动程序连接到 Amazon Redshift 的更多信息，请参阅“使用 JDBC/ODBC 连接到 Amazon Redshift”。

AWS 还提供一种原生云服务器无服务 BI 服务，[Amazon QuickSight](https://aws.amazon.com/quicksight)，与 Amazon Redshift 紧密集成，无需设置驱动程序。 Amazon QuickSight 采用按用户付费的定价模型，自动扩展，无需维护服务器。您可以在[Amazon QuickSight Gallery](https://oreil.ly/PX_nz)上探索许多实时示例仪表板，演示该工具中提供的各种功能。

在以下零售分析示例仪表板中（图 2-35），您可以看到 QuickSight 的许多关键功能。其中一些功能包括内置的机器学习算法用于预测和检测异常，定制的叙述内容以及丰富的可视化效果。

![QuickSight 示例](img/ardg_0235.png)

###### 图 2-35\. QuickSight 示例仪表板

在众多 QuickSight 数据源（图 2-36）中，有两种选项可连接到 Amazon Redshift：Redshift 自动发现和 Redshift 手动连接。一旦建立连接，用户可以通过几次点击开始构建报告和仪表板。

![QuickSight 数据源](img/ardg_0236.png)

###### 图 2-36\. QuickSight 数据源

当您利用 Redshift 自动发现选项时，用户选择要连接的 Redshift 实例 ID，并输入数据库、用户名和密码（图 2-37）。

![QuickSight Redshift 自动发现连接](img/ardg_0237.png)

###### 图 2-37\. QuickSight Redshift 自动发现连接

当您使用 Redshift 手动连接选项时，用户除了输入数据库、用户名和密码外，还需输入数据库服务器和端口（图 2-38）。

![QuickSight Redshift 手动连接](img/ardg_0238.png)

###### 图 2-38\. QuickSight Redshift 手动连接

## 使用 JDBC/ODBC 连接 Amazon Redshift

许多第三方 BI 和 ETL 工具通过 JDBC 和 ODBC 驱动访问 DB 平台。Amazon Redshift 提供开源驱动程序供您下载，并可能已经打包在您的第三方工具中。这些驱动程序由 AWS 经常更新，以配合产品发布的新功能。虽然 Amazon Redshift 也兼容 Postgres 驱动程序，但这些驱动程序不支持 Amazon Redshift 驱动程序中提供的所有功能，例如 IAM 认证。要配置您的 Amazon Redshift 驱动程序，您可以通过访问 Amazon Redshift 控制台获取连接 URL。

对于预置集群，您可以通过检查集群摘要页面找到 JDBC/ODBC URL（图 2-39）。

![Amazon Redshift 预置 JDBC/ODBC URL](img/ardg_0239.png)

###### 图 2-39\. Amazon Redshift 预置 JDBC/ODBC URL

对于服务器无服务器工作组，您可以通过检查工作组摘要页面找到 JDBC/ODBC URL（图 2-40）。

![Amazon Redshift 无服务器 JDBC/ODBC URL](img/ardg_0240.png)

###### 图 2-40\. Amazon Redshift 无服务器 JDBC/ODBC URL

在以下示例中，使用开源 Java 工具 [SQL Workbench/J](https://www.sql-workbench.eu)，我们已经使用收集的信息设置了客户端配置（图 2-41）。使用此工具，您可以利用扩展属性对话框设置用于功能（如基于 SAML 的身份验证）所需的可选参数。

![JDBC 客户端](img/ardg_0241.png)

###### 图 2-41\. JDBC 客户端配置

还请注意 Autocommit 复选框：这是一个经常被忽视的设置，在大多数情况下应启用。因为 Amazon Redshift 遵循 ACID（原子性、一致性、隔离性、持久性）规范的原子性特性，需要在多个用户事务中维护数据状态。当禁用时，Amazon Redshift 将假定每个连接都是新事务的开始，并且除非执行显式的 `commit` 语句，否则不会提交事务。如果忽视了此设置，系统可能会不必要地使用资源来跟踪多个事务状态。如果发现自己处于这种情况，请使用 [`PG_TERMINATE_BACKEND`](https://oreil.ly/WvQzN) 命令终止空闲连接。

要获取有关配置 JDBC 驱动程序的更多详细信息和选项，请查阅此[JDBC 驱动程序文档](https://oreil.ly/C7mcs)，要获取有关配置 ODBC 驱动程序的更多详细信息和选项，请查阅此[ODBC 在线文档](https://oreil.ly/3Oto5)。

# 总结

在本章中，我们向您展示了 Amazon Redshift 的架构及其入门方式。我们比较了无服务器和预配置的部署选项，展示了如何快速查询示例数据，并展示了不同的认证和连接选项。

在下一章中，我们将向您展示如何从数据湖、操作性源和实时流数据源最佳建模和加载数据。
