# 第四十八章：机器学习预测传达的伦理问题

# 拉多·科托洛夫

![](img/Rado_Kotorov.png)

CEO，Trendalyze 公司

今天，人们对我们拥有的惊人计算能力充满着着迷。计算机能够比人类更快地找到信息，比许多人更精确地从数据中提取洞察，比专家更快地回答问题，比大师更擅长下棋，等等。人们对机器已经建立起了如此多的尊重和信任，以至于他们经常将由机器生成的洞察作为事实来传达。

在他的文章[“中位数不是信息的全部”](https://oreil.ly/FhJ5Z)，最初发表于 1985 年的*Discover*杂志上，著名的进化人类学家斯蒂芬·杰伊·古尔德首次警示我们，不具备数学或科学背景的普通人面对统计和机器学习预测时所面临的危险和道德后果。在文章中，他描述了自己被诊断出患有致命癌症，医生拒绝告诉他预期寿命的个人经历。他在哈佛医学图书馆自行研究后得知，中位数预期寿命仅为八个月：“这就是为什么他们没有给我任何东西看的原因，”他想，“然后我的思维再次开始运转，谢天谢地。”

古尔德接着解释为什么使用中位数、平均数或任何其他统计预测来传达不治之症的预期寿命是错误的。终末期患者的积极态度在增加治疗效果中起着至关重要的作用。但统计预测通常会杀死积极态度，因为不懂统计学的人们不可避免地会误解信息。正如他所指出的：

> “八个月的中位数死亡率”在我们的口头语中意味着什么？我怀疑大多数没有统计学培训的人会把这样的声明理解为“我可能会在八个月内死去” —— 这个结论必须避免，因为这种说法不准确，而态度的重要性如此之大。

许多统计趋势指标（如中位数和平均数）的问题在于，人们将它们视为硬性事实，而忽略了它们周围的变异。但事实应该恰恰相反。变异是生活的事实，而中位数和平均数只是提供对更为复杂现实的不精确表示的人工产物。在他的诊断后，古尔德还活了 20 年，并出版了许多书籍。

2020 年 1 月 31 日，在*《每日秀》与特雷弗·诺亚*节目中，该节目主持人问：如果美国的预期寿命首次增长到 74 岁，我们会如何处理这些信息？我们会祝贺那些达到这个年龄的人吗？我们会设定个人目标以达到这个年龄吗？我们会认为未能达到这个年龄的人是失败者吗？所有这些都指向了我们如何传达从机器学习中得出的信息的重要性。

想象一个自动决策系统，其中一个患者通过算法进行诊断，并且预期寿命显示为一个大大闪烁的关键绩效指标（KPI）。这不仅毫无意义，还可能令人泄气。

随着我们部署更多的机器学习应用程序，我们可能会看到更多这样的关键绩效指标（KPI）。我们还没有开发出能够有意义地向医生和患者传达变化重要性及其解释的可视化工具。数据科学家指出，解释的负担落在医生身上。但医生不是数据科学家，与许多其他人一样，他们更倾向于接受这些预测作为事实。趋势及其周围变化的含义越难解释，人们就越有可能把单一数字作为生活的事实。

这个问题不仅限于医疗行业。想象一下，如果任何行业的管理者把中心趋势视为必须达成的硬性事实，来定义他们必须实现的目标。会有许多不准确的计划，甚至会错失更多的机会。因此，分析行业必须集中解决机器生成的见解和预测的沟通问题。我们不能期望普通人和专业人士理解复杂建模过程中的所有复杂性。