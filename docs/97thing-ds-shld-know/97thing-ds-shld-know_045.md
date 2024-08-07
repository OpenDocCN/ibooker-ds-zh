# 第四十一章：AI 伦理

# Cassie Kozyrkov

![](img/cassie_kozyrkov.png)

Google Cloud 首席决策科学家

为什么我们不讨论 AI 相对于其他技术独特更危险的地方呢？

与 AI 伦理相关的话题至关重要、及时且必不可少。我只是希望我们不要在……甚至与 AI 无关的情况下使用“AI 伦理”这个术语。许多所谓的 AI 伦理讨论焦点实际上是关于技术总体，而且它们并不新颖。将它们剔除，你会剩下一些集中在人格和奇点问题上的话题。不幸的是，这些会让你分心，而不是真正应该担心的事情。

将 AI 包装成镀铬的人形机器人，这是对公众无知的利用。我们这个物种看到任何事物都会投射人类的特征，从面包上看到面孔到云朵上看到身体。如果我在袜子上缝两个钮扣，我可能会和它说话。那只袜子娃娃不是人，AI 也不是；机器人只是另一种宠物石头。如今所说的“AI”不是指开发人形实体的替代品，而是一套写软件工具，让你可以通过示例（数据）而不是明确指令来编程。*这*才是 AI 的承诺和真正的危险所在。

## 分散注意力的层次

想象一下，你想自动化一个需要 10,000 步骤的任务。在传统编程中，*人类* 必须为每一个小指令努力工作。

将其视为需要人类手动组装的 10,000 个乐高积木。由于开发者们幸运地不耐烦，他们会将一些部分打包起来，这样就不必重复劳动。你可以下载这些包，只需将 50 个预先构建的每个有 200 个小积木的乐高构造件组合起来，而不是用 10,000 个零散的积木工作，如果你相信他人的工作，你可以将屋顶件连接到房屋件，而不是考虑瓦片和砖块的级别。

但有一点需要明白：即使*你*不必亲自完成所有这些（谢天谢地），这 10,000 步骤中的每一条指令都是由人类大脑苦苦思索出来的……而*那*部分将随着 ML/AI 而消失。

## AI 自动化无形之物

利用 AI，你可以说，“试着在这份数据上取得一个好成绩”，而不是编写“这样做，然后这样，然后这样……”的代码。

换句话说，AI 允许人类跳过手工制作 10,000 个明确解决步骤，而是通过开发者给出的示例中的模式自动产生这 10,000 行（或类似的内容）。这意味着即使没有人能够思考如何执行这些明确指令的任务，你也可以自动化这些任务。

如果你从未思考过*ML/AI 究竟自动化了谁的工作*，你准备好被震撼了：

+   开发者自动化/加速了其他人的工作。

+   ML/AI 自动化/加速了开发者的工作。

当今在 ML/AI 工程领域，有很多忙忙碌碌，但大多数是在启动和操控不友好的工具。在你的项目中可能会写下 10,000 行代码，但大多数是为了说服笨拙的工具接受你的数据。随着工具变得越来越好，你最终会发现在 ML/AI 中只有 *两个* 真正的指令：

1.  优化这个目标...

1.  ...在这个数据集上

就是这样。现在你可以用两行人类思维来自动化你的任务，而不是 10,000 行。这既美丽又令人恐惧！

## AI 实现了无思考

这里最直接的 ML/AI 特定问题是：*无思考的启用***。**

有些任务并不是很重要，能够毫不费力地完成它们是很棒的。但是当重要时，负责项目的人真的会在这两个 ML/AI 指令中的每一个上投入 5,000 指令的思考吗？

AI 是用例表达自己，但你不幸的是可以选择将系统指向一个数据集，而从未验证它是否包含相关、无偏见、高质量的例子。如果你允许自己在关键任务中无思考地选择数据，你可能会手忙脚乱地应对灾难。

AI 也不会阻止你选择一个听起来在你脑海中不错，但结果却是个糟糕主意的目标。比如领导可能对一个人类开发者说：“尽可能捕获尽可能多的垃圾邮件”，期望得到一个稳健而明智的过滤器。以同样的方式对 AI 算法表达这一目标，很快你就会开始纳闷为什么没有新邮件进来了。（答案：将 *所有* 东西标记为垃圾邮件可以得到你 *表述的* 目标的完美得分。）

AI 可怕的不是机器人。是人。

每当你将一个无思考的启用因素与速度和规模结合起来，你就会得到快速增强疏忽的配方。

## 我害怕 AI 吗？

不。

如果你问我是否害怕 AI，我听到的是在问我是否害怕人类的疏忽。这是问题对我有意义的唯一方式，因为我不相信机器人童话或与宠物石头交谈。

我对人类在 AI 未来持乐观态度，但我也在尽我所能不让其冒险。我坚信在 AI 时代负责任领导的技能是可以教授的，人们可以安全地构建有效的系统，推动进步，使周围人的生活更好。这就是为什么我和像我一样的人选择站出来，分享我们通过经验或通过挖掘以前孤立的学术学科所学到的东西。

技术改善我们的世界，解放我们免受疾病之苦，拓展我们的视野，将我们与亲人联系在一起，并赋予我们更长的生命。它也可以带来惊喜，不稳定和重新分配。它扩展得越多，可能带来的干扰就越大。始终应该将你的工具，包括人工智能，看作是在扩展你自己，而不是独立运行的。当它们扩展你的同时，确保你有能力避免踩到周围人的脚趾。

因此，由你来重新注入消失的思考回到你选择建立的人工智能项目中。通过将你的新发现的力量指向负责任的方向，你将释放出技术的最好一面。如果我们愿意，技术可以是美好的……而我相信我们会愿意。
