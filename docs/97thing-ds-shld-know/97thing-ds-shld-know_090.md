# 第八十四章：伦理 CRISP-DM：伦理数据科学开发框架

# 柯林·坎宁安

![](img/Collin_Cunningham.png)

亚马逊网络服务数据科学家

良好的数据科学创造了一种超越冷漠、无色过程的人类感觉的幻觉。然而，模型的目标是单一的：做出以前最小化损失函数的决策（或者类似机械的东西）。因此，我们必须系统地强化在没有同情心的地方实施同情心和伦理。

*跨行业标准数据挖掘过程*，通常称为 CRISP-DM，在分析开发中广泛使用。CRISP-DM 的步骤包括：

+   业务理解

+   数据理解

+   数据准备

+   建模

+   评估

+   部署

尽管 CRISP-DM 是为数据挖掘开发的，但成功的数据科学项目无论是否有意无意地在某种程度上遵循这些程序。为了在处理数据时做出更加符合伦理的决策，我们可以通过在每个步骤考虑一个问题来增强这一过程。通过这样做，我们为进行数据科学创造了一个具体的伦理框架。

## 业务理解

*这种解决方案可能的外部性是什么？* 每个成功的数据科学项目必须从理解问题以及其存在环境开始。这是定位项目成功的基础步骤，无论是在有效建模还是伦理方面，因为模型并不孤立存在。它可能有用户，但其结果也会影响其他人。花时间考虑解决方案的后果不仅可以节省时间，还可以防止灾难。积极与相关利益相关者进行明确讨论这些潜在后果是至关重要的。

## 数据理解

*我的数据是否反映了不道德的偏见？* 潜藏在人类数据中的是样本人群的意识和潜意识偏见。这些明确和隐含的偏见值得写一篇独立的论文，但下面是每种偏见类型的例子：

+   微软的 Twitter 聊天机器人 Tay 在吸收有意针对它的侮辱性言论后开始发布反犹太主义言论。

+   招聘模型是在先前某一特定人口统计数据集中占据职位的招聘模式上进行训练的。

作为数据科学家，我们理解深入了解数据内容和模式的价值，但评估数据如何可能腐化模型同样至关重要。

## 数据准备

*如何清洗具有偏见的数据？* 数据的完整性不可侵犯。然而，可以（而且有必要）清洗问题内容的数据，而不损害其完整性。在统计学家们暴动之前，请允许我解释。假设开发人员正在创建一个应用程序来预测支票欺诈。欺诈和真实支票之间的自然不平衡可能促使需要平衡数据集。下一个道德步骤可能是跨不同人群统计组平衡数据，以避免系统执行中可能出现的不平衡。否则，这种隐含的偏见可能在特定人群中生成更多的支票欺诈案例，再次被模型吸收，从而加剧偏见循环。这并非总是易事，就像词嵌入中的性别偏见示例。¹ 显式偏见应直接过滤。

## 建模

*我的模型容易受外部影响吗？* 网络设计模式越来越受欢迎。让模型自由灵活地适应带来很大价值，但这样做也会重新引入前一步骤中消除的危险因素。在高风险情况下，监控和清理数据之前的数据准入至关重要。以微软为例，开发人员未能预见数据集中潜在偏见——在损害发生后才意识到 Tay 吸收到的冒犯内容。

## 评估与部署

*我如何量化不道德的后果？* 负责任地部署模型需要监控和评估其在实际应用中的表现的指标。我们可以添加跟踪不道德外部影响的度量。例如，执法犯罪预测系统应跟踪其是否对某个社区过度执法，确保警务人员部署在不同人口统计区域时的平衡。模型的全部影响可能难以预测，因此定期重新评估模型非常重要，其中包括从与之互动的人群收集反馈。道德度量应当与效能度量一同突出展示。

情感共鸣无法量化；它缺乏严谨性和刚性。我们必须设法在我们提供的解决方案中印刻我们自己的道德指南。最终，我们要对我们交付的产品以及其后果负责。因此，在整个开发生命周期中，通过严格的反思制度，我们可以确保交付最大限度减少有害影响的道德模型。

¹ 出自 arXiv 的新兴技术，“如何通过向量空间数学揭示语言中的隐藏性别歧视”，*麻省理工科技评论*，2016 年 7 月 27 日。