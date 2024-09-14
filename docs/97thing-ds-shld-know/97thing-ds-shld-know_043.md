# 第三十九章：算法的使用方式与人类决策者不同

# Rachel Thomas

![](img/Rachel_Thomas.png)

fast.ai 的联合创始人；USF 应用数据伦理中心主任

人们经常讨论算法，好像它们是即插即用的，可以与人类决策者互换——例如，仅仅比较错误率来决定是否用算法结果替换人类决策者。然而，在实践中，算法和人类决策者的使用方式是不同的，如果不解决这些差异可能会导致许多伦理风险和危害。

这里有一些算法和人类决策者在实际运用中不同的常见方式：

+   算法更有可能在实施时没有*追索过程*。

+   算法经常*规模化应用*。

+   算法系统*成本低廉*。

+   人们更容易认为算法是*客观*或*无误的*。

这些因素之间存在很大的重叠。如果实施算法的主要动机是削减成本，那么增加上诉流程（甚至是认真检查错误）可能被视为“不必要”的开支。

考虑一个案例研究：阿肯色州实施软件来确定人们的医疗保健福利后，许多人看到他们接收护理的数量急剧减少，但没有得到解释和上诉的途径。患有脑瘫的女士 Tammy Dobbs 需要助理帮助她起床、上厕所等，但她的帮助时间突然减少了 20 小时每周，生活变得更糟。最终，一场长时间的法庭审理揭示了软件实施中的错误，Tammy 的工作时间恢复了（以及其他受错误影响的人们）。

另一个现实案例来自于一个用于解雇公立学校教师的算法。对于第五年级教师莎拉·维索奇的课堂观察得到了积极的评价。她的[副校长写道](https://oreil.ly/ALVe-)，“访问一个教室是一种愉快的体验，在这个教室中，合理教学的要素、积极学习环境和正面的学习氛围被如此有效地结合在一起。” 两个月后，她被一种不透明的算法与其他 200 多名教师一起解雇。PTA 主席和维索奇学生的家长称她为“我曾经接触过的最好的老师之一。每次我看到她，她都在关心孩子们，帮助他们复习功课；她花时间和他们在一起。” 那些因为没有追索机制而失去必要的医疗保健或被解雇的人们真是一个真正的反乌托邦！

数学家凯西·奥尼尔在她 2016 年的著作[*数学毁灭之武器*](https://weaponsofmathdestructionbook.com)（Crown）中写道，许多算法系统

> 倾向于惩罚贫困者。它们专门进行大宗交易，而且价格便宜。这就是它们吸引人的部分原因。相比之下，富人通常受益于个性化的建议。一家白手套律师事务所或独家预备学校将更多地依赖推荐和面对面的面试，而不像快餐连锁店或财政困难的城市学区那样。我们会一再看到，特权阶层更多地通过人工处理，而大众更多地通过机器处理。

这种伤害可能会因为许多人错误地认为计算机是客观且无误的事实而加剧。在加利福尼亚州兰开斯特市，一位[市官员](https://oreil.ly/mY_If)称，“通过机器学习、自动化，成功率达到 99%，因此那个机器人将会——将会——在预测下一步发生的事情时有 99%的准确率，这真的很有趣。”这种说法完全不正确。这是一个危险而常见的误解，可能导致人们忽视计算机输出中的有害错误。

作为机器人学家[彼得·哈斯](https://oreil.ly/dIvsh)在一次 TEDx 演讲中所说，“在 AI 领域，我们拥有米尔格拉姆的终极权威人物”，指的是斯坦利·米尔格拉姆的[著名实验](https://oreil.ly/yWvhA)，显示大多数人会服从权威人物的命令，甚至可能伤害或杀害其他人类。那么，人们更有可能信任被认为是客观和正确的算法？

由于算法通常用于更大规模的应用，大规模生产相同的偏见，并且被假定为无误或客观，我们无法以苹果和苹果的方式将它们与人类决策者进行比较。此外，在实施决策算法时，解决这些差异非常重要。必须实施系统来识别错误，并在任何算法实施之外实施追索机制。同样重要的是，确保使用算法输出的人员明白计算机并非无误，他们有能力提出任何他们发现的问题。