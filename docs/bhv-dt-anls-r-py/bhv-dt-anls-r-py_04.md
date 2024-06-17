# 第二部分：因果图与消除混杂

在[第一部分](part01.xhtml#understanding_behaviors)中，我们看到混杂可能会危害甚至最简单的数据分析。在[第二部分](#causal_diagrams_and_deconfounding)中，我们将学习构建因果图（CD）来表示、理解和消除变量之间的关系。

首先，[第三章](ch03.xhtml#introduction_to_causal_diagrams)介绍了因果图及其构建基础。

在[第四章](ch04.xhtml#building_causal_diagrams_from_scratch)中，我们将看到如何从头开始为新分析构建因果图。我们在冰淇淋示例中看到的因果图设计非常简单。但在现实生活中，确定应在我们的因果图中包含哪些变量，超出了我们感兴趣的因果关系，以及如何确定它们之间的关系，通常会变得复杂。

同样地，从我们的冰淇淋示例中消除混杂很简单：我们只需要在回归中包含我们感兴趣变量的联合原因。在更复杂的因果图中，确定应包含在回归中的变量可能变得困难。在[第五章](ch05.xhtml#using_causal_diagrams_to_deconfound_da)中，我们将看到可以应用于即使是最复杂因果图的规则。
