# 第四部分：设计与分析实验

在行为科学家和商业因果数据科学家中，运行实验是他们的基础工作。确实，随机分配实验组与控制组的受试者可以使我们消除任何潜在的混杂因素，而无需甚至识别它。

关于 A/B 测试的书籍随处可见。这本书的演示有何不同之处？我认为其方法的几个方面使其既更简单又更强大。

首先，将实验重新构建在因果行为框架内，将帮助您创建更好和更有效的实验，并更好地理解从观察到实验数据分析的光谱，而不是将它们视为相互分离的。

其次，大多数关于 A/B 测试的书籍依赖于统计测试，如均值的 T 检验或比例的检验。相反，我将依赖于我们已知的工具，线性和逻辑回归，这将使我们的实验更简单更强大。

最后，传统的实验方法根据其 p 值决定是否实施测试干预，这并不能带来最佳的业务决策。相反，我将依赖于**Bootstrap**及其置信区间，逐步确立其作为最佳实践。

因此，第八章将展示在使用回归和 Bootstrap 时，“简单”的 A/B 测试是什么样子。也就是说，对于每个顾客，我们抛掷一个比喻性的硬币。正面他们看到版本 A，反面他们看到版本 B。这通常是网站 A/B 测试的唯一解决方案。

然而，如果您事先知道将从中抽取实验对象的人群，您可以通过分层法创建更平衡的实验组。这可以显著增加实验的能力，正如我将在第九章中展示的那样。

最后，经常发生这样的情况，您无法按所需水平随机分组。例如，您对变更对客户的影响感兴趣，但必须在呼叫中心代表的级别上随机分组。这需要集群随机化和层次建模，我们将在第十章中看到。