# 第八十九章：为了对抗预测性执法中的偏见，正义不应该视而不见

# Eric Siegel

![](img/Eric_Siegel.png)

创始人，预测分析世界

犯罪预测模型陷入了一种争议不断的泥潭，因为它们本身无法实现种族公平。这是一个本质上无法解决的问题。事实证明，尽管这些模型在同等精度下成功地标记（即分配更高的概率给）黑人和白人被告，但由于这样做，它们也更经常*错误地*标记黑人被告而不是白人[¹]。

然而，尽管存在这种看似矛盾的困境，我们正目睹一个前所未有的机会，通过改变预测性执法的方式来积极促进社会公正，而不是 passively reinforcing 今天的不公平现象。

预测性执法引入了数量化元素，用来权衡由人类做出的重要执法决策，比如是否进行调查或拘留、设置多长的刑期以及是否假释。在做出这些决定时，法官和执法人员会考虑到嫌疑人或被告未来可能被定罪的计算概率。计算这些预测概率的工作由*预测建模*（又称*机器学习*）软件来完成。它会自动地[通过整理历史定罪记录来建立模式](https://oreil.ly/-5_sE)，而这些模式组合起来就形成了一个*预测模型*，用来计算那些未来尚不明朗的个体的概率。

尽管“色盲”，犯罪预测模型却对各种族有所区别对待。通常情况下，这些模型在计算时并没有明确地考虑种族或任何受保护的类别（我在本书的第五部分中详细介绍了这一政策的明显例外，即“*明显歧视性算法*“）。尽管如此，黑人被告比白人更频繁地被标记为高风险。

这种不平等现象直接源于我们所生活的种族失衡的世界。例如，被告的前科数量是预测模型的标准输入之一，因为有前科的被告比没有前科的被告更有可能在释放后再次犯罪。由于更多黑人被告有前科记录，这意味着预测模型更频繁地标记黑人被告而不是白人。一个黑人被告并没有因为种族而被标记，但却更有可能被标记。

然而，今天的激烈争论并不是关于这种更高的标记率，而是关于更高的*错误*标记率。预测模型错误地标记那些不会再犯的黑人被告比他们错误地标记白人被告更频繁。在关于预测性执法偏见最广泛引用的文章中，[ProPublica 报告](https://oreil.ly/5JKDE)称，全国使用的 COMPAS 模型错误地标记白人被告的比例为 23.5%，黑人被告的比例为 44.9%。换句话说，*不应该被标记的黑人被告几乎是白人的两倍被错误标记*。

与此相反，支持 COMPAS 的人反驳说，每个标记对两个种族都是同样合理的。回应 ProPublica，COMPAS 的创造者指出，在被标记为高风险的人群中，[被错误标记的比例在黑人和白人被告中是相似的](https://oreil.ly/AfNuK)：分别为 37%和 41%。换句话说，*在被标记的被告中，COMPAS 对白人和黑人被告的错误率是相同的*。[其他数据科学家也同意](https://oreil.ly/E-I60)，这符合无偏见模型的标准。

每个个体的标记似乎在种族上是公平的，但虚假标记的总体率并非如此。尽管这两种说法可能似乎相互矛盾，但它们都是正确的：

+   如果你被标记，不管种族如何，应该是理所当然的。

+   如果你不应该被标记，如果你是黑人，你更有可能被错误标记。

谁是对的？一方面，所有的标记似乎都同样值得。对于被指定更高概率的被告，无论是白人还是黑人被告，随后的起诉率都是一样的。另一方面，在那些不会再犯的被告中，黑人面临更高的虚假标记风险。更细腻的立场声称，要解决这个问题，[我们必须就公平的定义达成一致](https://oreil.ly/gkWQc)。

但与其就模型是否“有偏见”争论不休，更明智的解决方法是就对抗种族不平等的措施达成一致。对“有偏见”一词的争论让人分心，而不是仅仅评估模型是否加剧了种族不公正，让我们增强预测性执法，积极减少不公正。这种看似矛盾的现象揭示了当今种族不平等的一个通常隐藏的症状：如果预测标记被校准为对两个群体同样精确，那么鉴于黑人被告总体重新犯率更高，该群体就会遭受更多虚假标记的影响。

这真是一个惊人的不平等现象。对于任何种族的被告人来说，被标记意味着承受着标记可能是虚假的重大风险。这可能导致额外的监禁年限，而被监禁的被告人失去了证明未来不会犯罪的自由（因为他们无法证明这是有根据的）。对于黑人群体而言，比白人更频繁地承受这种风险更是雪上加霜：不仅黑人更有可能成为被告人，而且黑人被告因虚假预测未来犯罪更有可能被判额外监禁年限。

为了解决这个问题，让我们教育和指导执法决策者了解这种观察到的不平等。培训法官、假释委员会和警官了解在他们获得计算出的黑人嫌疑人、被告或罪犯将会重新犯罪的概率时的相关警告。通过这样做，赋予这些决策者将这些考量纳入他们的决策过程中的能力。

在处理重新犯罪概率时需要反思的三个关键考虑因素是：

*你所关注的概率受到被告人种族的影响，通过替代者。*

虽然种族并非公式的直接输入因素，但该模型可能会包含非自愿的因素，这些因素近似种族，如家庭背景、社区（“你所在社区的犯罪率如何？”）、教育水平（仅部分选择），以及[家人和朋友的行为](https://oreil.ly/iQ8Nt)。

由于有偏见的基本事实，这些概率对黑人被告人不利。

由于黑人更常被调查、逮捕，因此更频繁地被定罪，尽管犯下同样罪行的白人，模型性能的测量不揭示黑人被告更频繁地受到不公平标记的程度。

*黑人群体深受虚假标记之苦。*

将这一系统问题纳入考虑有助于更大的利益。承认这一问题提供了一个机会，以帮助补偿过去和现在的种族不公正和随之而来的剥夺周期。这正是预测性执法可以缓解这些循环模式而不是无意中放大它们的地方。

预测犯罪模型本身必须从设计上保持无色彩的，但我们如何在上下文中解释和应用它们则不能保持如此。以这种方式重新引入种族是从简单地检测预测模型的种族偏见转向积极设计预测性执法，以促进种族正义的唯一手段。

¹ 本文中许多具体细节的参考资料可以在埃里克·西格尔的文章“如何用预测性执法对抗偏见”中找到，*Voices*（博客），*科学美国人*，2018 年 2 月 19 日，[*https://oreil.ly/OT6py*](https://oreil.ly/OT6py)。
