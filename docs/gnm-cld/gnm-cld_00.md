# 第八章：高级主题

在前一章中，您已经概述了使用 Argo CD 配方实施 GitOps 工作流的概述。Argo CD 是一个著名且具有影响力的开源项目，既适用于简单的用例，也适用于更复杂的用例。在本章中，我们将讨论在您在 GitOps 旅程中前进时所需的主题，以及在多集群场景下管理安全性、自动化和高级部署模型的内容。

安全性是自动化和 DevOps 的关键方面。 DevSecOps 是一个新的定义，其中安全性是整个 IT 生命周期中共享的责任。此外，《DevSecOps 宣言》（https://www.devsecops.org）将安全性视为操作和减少摩擦的价值贡献代码的一部分。这与 GitOps 原则相一致，其中一切都是声明性的。

另一方面，这也引出了一个问题，即如何避免在 Git 中存储未加密的明文凭证。正如克里斯蒂安·埃尔南德斯的书《通往 GitOps 的路径》中所述，Argo CD 目前幸运地提供了两种模式来管理 GitOps 工作流中的安全性：

+   在 Git 中存储加密的秘密，例如使用 Sealed Secret（见食谱 8.1）

+   将秘密存储在外部服务或保险库中，然后只在 Git 中存储对这些秘密的引用（参见食谱 8.2）

本章随后介绍了高级部署技术，展示了如何使用 Argo CD 管理 Webhooks（参见食谱 8.3）和 ApplicationSets（参见食谱 8.4）。ApplicationSets 是 Argo CD 的一个组件，允许从单个 Kubernetes 资源管理多个应用程序、存储库或集群的部署。从本质上讲，这是一个模板化系统，可以在多个 Kubernetes 集群中部署和同步 GitOps 应用程序（参见食谱 8.5）。

最后但同样重要的是，本书以使用 Argo Rollouts 实现 Kubernetes 的渐进式交付结束（食谱 8.6），这对于使用蓝绿或金丝雀等先进部署技术部署应用程序非常有用。

# 8.1 加密敏感数据（密封密钥）

## 问题

您希望在 Git 中管理 Kubernetes Secrets 和加密对象。

## 解决方案

[密封密钥](https://oreil.ly/MWTNB) 是 Bitnami 的一个开源项目，用于将 Kubernetes Secrets 加密为 `SealedSecret` Kubernetes 自定义资源，表示一个安全存储在 Git 中的加密对象。

密封密钥使用公钥加密技术，由两个主要组件组成：

+   一个 Kubernetes 控制器，了解用于解密和加密加密秘密的私钥和公钥，并负责协调。该控制器还支持私钥的自动轮换和密钥过期管理，以强制重新加密秘密。

+   `kubeseal` 是开发人员用来在提交到 Git 存储库之前加密其秘密的命令行界面。

`SealedSecret` 对象仅由运行在目标 Kubernetes 集群中的 `SealedSecret` 控制器加密和解密。这个操作仅由这个组件独占，因此没有其他人可以解密该对象。`kubeseal` CLI 允许开发者将普通的 Kubernetes Secret 资源转换为 `SealedSecret` 资源定义，如 Figure 8-1 所示。

在您的 Kubernetes 集群中使用 Argo CD，您可以从 [GitHub 项目的发布版本](https://oreil.ly/zmEh3) 中为您的操作系统安装 `kubeseal` CLI。在撰写本书时，我们使用的是版本 0.18.2。

###### 小贴士

在 macOS 上，`kubeseal` 可以通过 [Homebrew](https://brew.sh) 安装如下：

[PRE0]

![Sealed Secrets with GitOps](img/gocb_0801.png)

###### 图 8-1\. Sealed Secrets with GitOps

安装 CLI 后，您可以按如下方式安装控制器：

[PRE1]

您应该获得类似以下的输出：

[PRE2]

例如，让我们为部署在 第五章 中的 Pac-Man 游戏创建一个 Secret：

[PRE3]

您应该有以下输出：

[PRE4]

这里您可以看到 YAML 表示：

[PRE5]

[PRE6]

现在，您可以通过以下方式将 Secret 转换为 `SealedSecret`：

[PRE7]

[PRE8]

![1](img/#co_advanced_topics_CO1-1)

这里找到由 Sealed Secrets 控制器加密的数据。

现在，您可以安全地将您的 `SealedSecret` 推送到 Kubernetes 部署清单存储库并创建 Argo CD 应用程序。这是来自 [本书仓库的示例](https://oreil.ly/TXHRa)：

[PRE9]

检查应用程序是否正在运行并且健康：

[PRE10]

您应该获得类似以下的输出：

[PRE11]

# 8.2 使用 ArgoCD 加密 Secrets（ArgoCD + HashiCorp Vault + External Secret）

## 问题

您希望避免在 Git 中存储凭据，并希望在外部服务或保险库中管理它们。

## 解决方案

在 Recipe 8.1 中，您看到了如何按照 GitOps 声明式方式管理 Git 中的加密数据，但如何避免即使加密凭据也与 GitOps 一起存储？

其中一种解决方案是 [External Secrets](https://oreil.ly/ytBeU)，这是一个由 GoDaddy 最初创建的开源项目，旨在将密钥存储在不同供应商的外部服务或保险库中，然后仅在 Git 中存储对这些密钥的引用。

如今，External Secrets 支持诸如 AWS Secrets Manager、HashiCorp Vault、Google Secrets Manager、Azure Key Vault 等系统。其思想是为存储和管理密钥生命周期的外部 API 提供用户友好的抽象。

ExternalSecrets 深度上是一个 Kubernetes 控制器，它从包含指向外部密钥管理系统中密钥的参考的自定义资源中调解 Secrets 到集群。自定义资源 `SecretStore` 指定了包含机密数据的后端，并定义了模板，通过它可以将其转换为 Secret，如您在 Figure 8-2 中看到的那样。SecretStore 具有配置以连接到外部密钥管理系统。

因此，`ExternalSecrets` 对象可以安全地存储在 Git 中，因为它们不包含任何机密信息，只包含管理凭据的外部服务的引用。

![使用 Argo CD 的外部 Secrets](img/gocb_0802.png)

###### 图 8-2\. 使用 Argo CD 的外部 Secrets

您可以按照以下步骤使用 Helm Chart 安装 External Secrets。在撰写本书时，我们使用的是版本 0.5.9：

[PRE12]

您应该会得到类似以下的输出：

[PRE13]

要开始使用 ExternalSecrets，您需要设置一个 SecretStore 或 ClusterSecretStore 资源（例如，通过创建一个 *vault* SecretStore）。

更多关于不同类型的 SecretStores 及如何配置它们的信息可以在我们的 [GitHub 页面](https://oreil.ly/LQzEh) 找到。

###### 提示

您还可以通过 [OperatorHub.io](https://oreil.ly/w3x71) 从 OLM 安装 External Secrets Operator。

例如，您可以使用支持的提供商之一，如 [HashiCorp Vault](https://oreil.ly/sg7yS)，执行以下操作。

首先下载并安装适用于您操作系统的 [HashiCorp Vault](https://oreil.ly/vjGSq)，并获取您的 [Vault Token](https://oreil.ly/6Y5cS)。然后按照以下步骤创建一个 Kubernetes Secret：

[PRE14]

然后创建一个 `SecretStore` 作为对这个外部系统的引用：

[PRE15]

![1](img/#co_advanced_topics_CO2-1)

运行您的 Vault 的主机名

![2](img/#co_advanced_topics_CO2-2)

包含 vault 令牌的 Kubernetes Secret 的名称

![3](img/#co_advanced_topics_CO2-3)

用于访问包含 vault 令牌内容的 Kubernetes Secret 中的值的关键：

[PRE16]

现在，您可以按照以下步骤在您的 Vault 中创建一个 Secret：

[PRE17]

然后从 `ExternalSecret` 中引用它，如下所示：

[PRE18]

[PRE19]

现在，您可以按照以下步骤使用 External Secrets 使用 Argo CD 部署 Pac-Man 游戏：

[PRE20]

# 8.3 自动触发应用程序部署（Argo CD Webhooks）

## 问题

您不想等待 Argo CD 同步，而是希望在 Git 中发生更改时立即部署应用程序。

## 解决方案

虽然 Argo CD 每三分钟轮询 Git 仓库以检测受监视的 Kubernetes manifests 的更改，但它也支持来自流行 Git 服务器（如 GitHub、GitLab 或 Bitbucket）的 webhook 通知的事件驱动方法。

在您的 Argo CD 安装中启用了 [Argo CD Webhooks](https://oreil.ly/3Ab46)，并且可以在端点 `/api/webhooks` 处使用。

要使用 Minikube 测试 Argo CD 的 webhook，您可以使用 Helm 安装一个本地 Git 服务器，如 [Gitea](https://docs.gitea.io)，一个用 Go 编写的开源轻量级服务器，如下所示：

[PRE21]

您应该会得到类似以下的输出：

[PRE22]

###### 提示

使用默认凭据登录到 Gitea 服务器，您可以在 Helm Chart 的 *values.yaml* 文件中找到它们，或者通过覆盖定义新的凭据。您可以在这里找到 Helm Chart 的链接：[here](https://oreil.ly/Nkaeu)。

将 [Pac-Man](https://oreil.ly/LwTaC) manifests 仓库导入 Gitea。

配置 Argo 应用：

[PRE23]

要向 Gitea 添加 Webhook，请导航到右上角，然后单击“设置”。选择“Webhooks”选项卡，并按照图 8-3 中所示进行配置：

+   负载 URL：`http://localhost:9090/api/webhooks`

+   内容类型：`application/json`

![Gitea Webhooks](img/gocb_0803.png)

###### 图 8-3\. Gitea Webhooks

###### 提示

在这个示例中，您可以忽略密钥；但是，最佳实践是为您的 Webhooks 配置密钥。从[文档](https://oreil.ly/udDkS)中了解更多信息。

保存并推送更改到 Gitea 上的仓库。在您推送后，您将立即看到来自 Argo CD 的新同步。

# 8.4 部署到多个集群

## 问题

您想要将应用程序部署到不同的集群。

## 解决方案

Argo CD 支持 `ApplicationSet` 资源来“模板化” Argo CD `Application` 资源。它涵盖了不同的用例，但最重要的是：

+   使用 Kubernetes 清单来针对多个 Kubernetes 集群。

+   从一个或多个 Git 仓库部署多个应用程序。

由于 `ApplicationSet` 是一个包含运行时占位符的模板文件，我们需要用一些值来填充这些占位符。为此，`ApplicationSet` 具有 *生成器* 的概念。

生成器负责生成参数，这些参数最终将替换模板占位符，以生成有效的 Argo CD `Application`。

创建以下 `ApplicationSet`：

[PRE24]

![1](img/#co_advanced_topics_CO3-1)

定义一个生成器

![2](img/#co_advanced_topics_CO3-2)

设置参数的值

![3](img/#co_advanced_topics_CO3-3)

将 `Application` 资源定义为模板

![4](img/#co_advanced_topics_CO3-4)

`cluster` 占位符

![5](img/#co_advanced_topics_CO3-5)

`url` 占位符

通过运行以下命令应用先前的文件：

[PRE25]

当将此 `ApplicationSet` 应用于集群时，Argo CD 会生成并自动注册两个 `Application` 资源。第一个是：

[PRE26]

第二个：

[PRE27]

通过运行以下命令检查两个 `Application` 资源的创建：

[PRE28]

输出应类似于（已截断）：

[PRE29]

通过删除 `ApplicationSet` 文件来删除这两个应用程序：

[PRE30]

## 讨论

我们已经看到了最简单的生成器，但截至本书编写时总共有八个生成器：

列表

通过一个固定的集群列表生成 `Application` 定义。（这是我们之前看到的那个）。

集群

类似于 *List*，但基于 Argo CD 中定义的集群列表。

Git

根据 Git 仓库中的 JSON/YAML 属性文件或仓库的目录布局生成 `Application` 定义。

SCM 提供者

通过组织内的仓库生成 `Application` 定义。

拉取请求

从开放式拉取请求生成 `Application` 定义。

集群决策资源

使用 [鸭子类型](https://oreil.ly/kpRkV) 生成 `Application` 定义。

矩阵

结合两个独立生成器的值。

合并

合并两个或更多生成器的值。

在前面的示例中，我们从一个固定的元素列表中创建了`Application`对象。当可配置环境的数量较少时，这样做是合适的；例如，在示例中，两个集群分别指代两个 Git 文件夹（`ch08/bgd-gen/staging`和`ch08/bgd-gen/prod`）。对于多个环境的情况（即多个文件夹），我们可以动态地使用*Git*生成器来为每个目录生成一个`Application`。

让我们将前面的示例迁移到使用 Git 生成器。作为提醒，使用的 Git 目录布局是：

[PRE31]

创建类型为`ApplicationSet`的新文件，为配置的 Git 存储库中的每个目录生成一个`Application`：

[PRE32]

![1](img/#co_advanced_topics_CO4-1)

配置 Git 存储库以读取布局

![2](img/#co_advanced_topics_CO4-2)

开始扫描目录的初始路径

![3](img/#co_advanced_topics_CO4-3)

`Application`定义

![4](img/#co_advanced_topics_CO4-4)

Git 存储库中与路径通配符（`staging`或`prod`）匹配的目录路径

![5](img/#co_advanced_topics_CO4-5)

目录路径（完整路径）

![6](img/#co_advanced_topics_CO4-6)

最右边的路径名

应用资源：

[PRE33]

由于有两个目录，Argo CD 创建了两个应用程序：

[PRE34]

此外，当您的应用程序由不同组件（服务、数据库、分布式缓存、电子邮件服务器等）组成，并且每个元素的部署文件放置在其他目录中时，此生成器也非常方便。或者，例如，一个包含所有要安装在集群中的操作器的存储库：

[PRE35]

Git 生成器可以创建具有在 JSON/YAML 文件中指定的参数的`Application`对象，而不是响应目录。

以下片段显示了一个示例 JSON 文件：

[PRE36]

这是用于响应这些文件的`ApplicationSet`的节选：

[PRE37]

![1](img/#co_advanced_topics_CO5-1)

查找放置在`app`所有子目录中的所有*config.json*文件

![2](img/#co_advanced_topics_CO5-2)

注入在*config.json*中设置的值

此`ApplicationSet`将为匹配`path`表达式的文件夹中的每个*config.json*文件生成一个`Application`。

## 参见

+   [Argo CD 生成器](https://oreil.ly/EnOfl)

+   [鸭子类型](https://oreil.ly/tEFQW)

# 8.5 将拉取请求部署到集群

## 问题

部署应用程序预览时，希望在创建拉取请求时进行。

## 解决方案

使用*pull request*生成器自动发现存储库中的所有打开拉取请求，并创建一个`Application`对象。

让我们创建一个`ApplicationSet`来响应在配置的存储库上创建带有`preview`标签的 GitHub 拉取请求。

创建名为*bgd-pr-application-set.yaml*的新文件，其内容如下：

[PRE38]

![1](img/#co_advanced_topics_CO6-1)

GitHub 拉取请求生成器

![2](img/#co_advanced_topics_CO6-2)

组织/用户

![3](img/#co_advanced_topics_CO6-3)

存储库

![4](img/#co_advanced_topics_CO6-4)

选择目标 PR

![5](img/#co_advanced_topics_CO6-5)

轮询时间以检查是否有新的 PR（60 秒）

![6](img/#co_advanced_topics_CO6-6)

使用分支名称和编号设置名称

![7](img/#co_advanced_topics_CO6-7)

设置 Git SHA 号码

运行以下命令应用先前的文件：

[PRE39]

现在，如果您列出 Argo CD 应用程序，您会发现没有注册任何应用程序。原因是仓库中尚未带有`preview`标签的拉取请求：

[PRE40]

对仓库创建拉取请求并标记为`preview`。

在 GitHub 中，拉取请求窗口应类似于图 8-4。

![GitHub 中的拉取请求](img/gocb_0804.png)

###### 图 8-4\. GitHub 中的拉取请求

等待一分钟，直到`ApplicationSet`检测到更改并创建`Application`对象。

运行以下命令来检查是否已检测和注册更改：

[PRE41]

检查`Application`对拉取请求的注册：

[PRE42]

当拉取请求关闭时，`Application`对象会自动删除。

## 讨论

撰写本书时，支持以下拉取请求提供者：

+   GitHub

+   Bitbucket

+   Gitea

+   GitLab

ApplicationSet 控制器每隔`requeueAfterSeconds`时间间隔轮询以检测变更，同时还支持使用 Webhook 事件。

要进行配置，请参考 Recipe 8.3，但同时在 Git 提供商中也启用发送拉取请求事件。

# 8.6 使用高级部署技术

## 问题

您希望使用蓝绿部署或金丝雀发布等先进的部署技术来部署应用程序。

## 解决方案

使用 [Argo Rollouts](https://oreil.ly/g4mlf) 项目来更新应用程序的部署。

Argo Rollouts 是一个 Kubernetes 控制器，提供先进的部署技术，如蓝绿部署、金丝雀发布、镜像、暗金丝雀、流量分析等，以供 Kubernetes 使用。它与许多 Kubernetes 项目集成，如 Ambassador、Istio、AWS 负载均衡器控制器、NGNI、SMI 或 Traefik 用于流量管理，以及 Prometheus、Datadog 和 New Relic 等项目用于执行分析以推动渐进式交付。

要将 Argo Rollouts 安装到集群中，请在终端窗口中运行以下命令：

[PRE43]

虽然不是强制性的，但我们建议您安装 Argo Rollouts Kubectl 插件以可视化部署过程。请按照[说明](https://oreil.ly/1GWsz)安装它。一切就绪后，让我们部署 BGD 应用程序的初始版本。

Argo Rollouts 不使用标准 Kubernetes `Deployment` 文件，而是一个名为 `Rollout` 的特定新 Kubernetes 资源。它类似于`Deployment`对象，因此支持其所有选项，但它添加了一些字段以配置滚动更新。

让我们部署应用程序的第一个版本。当 Kubernetes 执行滚动更新时，我们将定义金丝雀发布过程，本例中的步骤如下：

1.  将 20%的流量转发到新版本。

1.  等待人类决定是否继续进行。

1.  自动将 40%、60%、80%的流量转发到新版本，每次增加后等待 30 秒。

创建一个名为*bgd-rollout.yaml*的新文件，内容如下：

[PRE44]

![1](img/#co_advanced_topics_CO7-1)

金丝雀发布

![2](img/#co_advanced_topics_CO7-2)

要执行的步骤列表

![3](img/#co_advanced_topics_CO7-3)

设置金丝雀比例

![4](img/#co_advanced_topics_CO7-4)

发布被暂停

![5](img/#co_advanced_topics_CO7-5)

暂停 30 秒后继续发布

![6](img/#co_advanced_topics_CO7-6)

`template`部署定义

应用资源以部署应用程序。由于没有先前的部署，金丝雀部分被忽略：

[PRE45]

目前，根据`replicas`字段指定了五个 Pod：

[PRE46]

并使用 Argo Rollout Kubectl 插件：

[PRE47]

让我们部署一个新版本以触发金丝雀滚动更新。创建一个名为*bgd-rollout-v2.yaml*的新文件，内容与上一个文件完全相同，但将环境变量`COLOR`的值更改为`green`：

[PRE48]

应用先前的资源并检查 Argo Rollouts 如何执行滚动更新。再次列出 Pod 以确保 20%的 Pod 是新的，而另外 80%是旧版本：

[PRE49]

![1](img/#co_advanced_topics_CO8-1)

新版本 Pod

并使用 Argo Rollout Kubectl 插件执行相同操作：

[PRE50]

请记住，滚动更新过程暂停，直到操作员执行手动步骤才能让进程继续。在终端窗口中运行以下命令：

[PRE51]

该发布被提升并继续以下步骤，即每 30 秒替换旧版本 Pod 为新版本：

[PRE52]

新版本逐步部署到集群完成滚动更新。

## 讨论

Kubernetes 不原生实现高级部署技术。因此，Argo Rollouts 使用已部署的 Pod 数量来实现金丝雀发布。

正如前面提到的，Argo Rollouts 与像[Istio](https://istio.io)这样提供高级流量管理功能的 Kubernetes 产品集成。

使用 Istio，在基础设施级别正确执行流量分流，而不像第一个示例中玩弄副本数量。Argo Rollouts 与 Istio 集成以执行金丝雀发布，自动更新 Istio 的`VirtualService`对象。

假设您已了解 Istio 并且具有安装了 Istio 的 Kubernetes 集群，您可以通过将`Rollout`资源的`trafficRouting`设置为`Istio`来执行 Argo Rollouts 与 Istio 的集成。

首先，创建一个带有 Istio 配置的`Rollout`文件：

[PRE53]

![1](img/#co_advanced_topics_CO9-1)

金丝雀部分

![2](img/#co_advanced_topics_CO9-2)

引用指向新服务版本的 Kubernetes Service

![3](img/#co_advanced_topics_CO9-3)

指向旧服务版本的 Kubernetes 服务的引用

![4](img/#co_advanced_topics_CO9-4)

配置 Istio

![5](img/#co_advanced_topics_CO9-5)

更新权重的 `VirtualService` 的参考

![6](img/#co_advanced_topics_CO9-6)

`VirtualService` 的名称

![7](img/#co_advanced_topics_CO9-7)

`VirtualService` 中的路由名称

![8](img/#co_advanced_topics_CO9-8)

部署 Istio 边车容器

然后，我们创建两个 Kubernetes 服务，指向相同的部署，用于将流量重定向到旧版本或新版本。

下面的 Kubernetes 服务用于 `stableService` 字段：

[PRE54]

金丝雀版本与之相同，但名称不同。它是在 `canaryService` 字段中使用的服务：

[PRE55]

最后，创建 Istio 虚拟服务，由 Argo Rollouts 更新每个服务的金丝雀流量：

[PRE56]

![1](img/#co_advanced_topics_CO10-1)

稳定的 Kubernetes 服务

![2](img/#co_advanced_topics_CO10-2)

金丝雀 Kubernetes 服务

![3](img/#co_advanced_topics_CO10-3)

路由名称

应用这些资源后，我们将启动应用程序的第一个版本：

[PRE57]

当 `Rollout` 对象上发生任何更新时，将按照解决方案中描述的方式启动金丝雀发布。现在，Argo Rollouts 自动更新 *bgd 虚拟服务* 的权重，而不是调整 Pod 数量。

## 参见

+   [Argo Rollouts - Kubernetes 渐进交付控制器](https://oreil.ly/XQ64b)

+   [Istio - Argo Rollouts](https://oreil.ly/lKDYH)

+   [Istio](https://istio.io)

+   [来自 Red Hat 的 Istio 教程](https://oreil.ly/Vzk9G)
