# 第五章：构建业务案例

学习为模型或实验撰写业务案例是数据科学家必须发展的关键技能。它不仅可以帮助您快速了解一个新项目是否值得您的时间和精力，还可以帮助您获得利益相关者的支持。此外，它与将使您脱颖而出的极端所有权类型保持一致。

业务案例可以复杂到任何您想要的程度，但很多时候您可以得出足够好的估计。在本章中，我将介绍业务案例创建的基本原理。

# 一些构建业务案例的原则

虽然每个业务案例都不同，但大多数可以使用相同的基本原则来构建：比较做出决策或不做出决策，计算所有选项的成本和收益，只考虑增量变化，许多时候您只能考虑单位经济。

决策

业务案例通常用于评估正在考虑的新决策，无论是新的广告活动、杠杆变动还是其他决策。

成本、收益和盈亏平衡

最有趣的决策往往涉及权衡。一个关键的起点是列举决策带来的主要成本和收益。业务案例将围绕着作为收益和成本之间货币差异计算出来的*净收益*构建。*盈亏平衡*意味着具有零净收益，并作为您决策的极限案例或最坏情况。

增量性

一个良好的业务案例应该只考虑那些来源于决策的成本和收益。例如，如果您在进行一个实验，您的工资可以被视为一种成本，但这并非*增量*，因为如果您在做其他事情，公司也必须支付您。只有增量成本和收益应该被纳入考虑范围。

单位经济学

大多数情况下，只有您的*平均*客户会发生什么事情是重要的，所以您可以只专注于这个孤立单元的增量成本和收益。业务案例依赖于您为这个单元计算出来的净收益的符号；通常，扩展到整个客户基础会以相同比例影响成本和收益，不影响总体净收益的符号。

# 例子：积极的保留策略

让我们评估公司是否应该推出积极的保留策略。在成本方面，您需要给客户提供留下的动机。有很多方法可以做到这一点，但大多数可以轻松地转化为一个货币数额 *c*。在收益方面，一个额外留下来的客户可以产生每用户平均收入 *r*，本来可能会丢失的。

假设您的客户基数大小为 *B*。其中，*A* 接受了激励措施。此外，仅有 *TP* 位真正可能会流失的目标客户（真正的阳性）。通过平衡成本和收益来获得盈亏平衡条件：

<math alttext="upper B times StartFraction upper A Over upper B EndFraction times c equals upper B times StartFraction upper T upper P Over upper B EndFraction times r" display="block"><mrow><mi>B</mi> <mo>×</mo> <mfrac><mi>A</mi> <mi>B</mi></mfrac> <mo>×</mo> <mi>c</mi> <mo>=</mo> <mi>B</mi> <mo>×</mo> <mfrac><mrow><mi>T</mi><mi>P</mi></mrow> <mi>B</mi></mfrac> <mo>×</mo> <mi>r</mi></mrow></math>

您可以看到在第二章中呈现的一种技术的应用。请注意，在这种情况下，您可以仅专注于平均单元：

<math alttext="StartFraction upper A Over upper B EndFraction times c equals StartFraction upper T upper P Over upper B EndFraction times r" display="block"><mrow><mfrac><mi>A</mi> <mi>B</mi></mfrac> <mo>×</mo> <mi>c</mi> <mo>=</mo> <mfrac><mrow><mi>T</mi><mi>P</mi></mrow> <mi>B</mi></mfrac> <mo>×</mo> <mi>r</mi></mrow></math>

当净收益为非负时，运行活动是有意义的：

<math alttext="StartFraction upper T upper P Over upper B EndFraction times r minus StartFraction upper A Over upper B EndFraction times c greater-than-or-equal-to 0" display="block"><mrow><mfrac><mrow><mi>T</mi><mi>P</mi></mrow> <mi>B</mi></mfrac> <mo>×</mo> <mi>r</mi> <mo>-</mo> <mfrac><mi>A</mi> <mi>B</mi></mfrac> <mo>×</mo> <mi>c</mi> <mo>≥</mo> <mn>0</mn></mrow></math>

第一个分数只是活动基础或样本中的真正正例率；第二个分数是接受率。或者，更方便的是，您还可以将这些视为预期利益和成本的样本估计，以便您的决策问题在不确定性下映射得更加清晰：在活动之前，您不知道谁会接受奖励或在其缺席时实际上会流失。

您现在可以插入一些数字来模拟不同情景下的商业案例。此外，您还可以分析可操作的杠杆。这里有三个杠杆可以促使业务案例起作用：

提高真正正例率。

您可以通过用机器学习（ML）模型进行更准确的预测来帮助业务案例，从真正的正例角度来看。

控制成本。

您可以降低奖励的价值（*c*）。有时可以安全地假设接受率会随之增加，因此这两个术语朝着同一个方向发展。

只针对高 ARPU 客户。

直观上讲，奖励应优先给高价值客户。在不平等中，这对应于更高的*r*。

注意增量性的启动：在利益方面，您应该只包括真正会流失的客户（真正的正例）带来的*已保存* ARPU。那些无论如何都会留下的客户，如果接受了但没有带来增量收益，会增加成本。

关于*假阴性*呢？记住这些是未被针对的客户，但却流失了。您可以将丢失的收入包括为成本，以便在 ML 实施中权衡精度和召回率。

# 防欺诈

银行经常为防止欺诈（以及反洗钱）目的设立交易限额。让我们为超出限额的交易决定制止的商业案例做出构建。

直观地说，存在两种成本：欺诈成本（*c[f]*）和如果客户流失则为丢失或放弃的收入（*c[ch]*）。为简单起见，我将假设一个被阻止交易的客户肯定会流失，但在实际应用中可以放宽这一假设。在收入方面，如果允许交易通过，公司将获得票面金额（*t*）。

一旦交易进入，您可以选择接受或阻止它。无论采取什么行动，它都可能合法也可能不合法。表 5-1 显示了所有四种行动和结果组合的成本和收益。

表 5-1\. 防欺诈的成本和收益

| 行动 | 结果 | 收益 | 成本 |
| --- | --- | --- | --- |
| 接受 | 欺诈 | *t* | <math alttext="c Subscript f"><msub><mi>c</mi> <mi>f</mi></msub></math> |
| 接受 | 合法 | *t* | 0 |
| 阻止 | 欺诈 | 0 | 0 |
| 阻止 | 合法 | 0 | <math alttext="c Subscript c h"><msub><mi>c</mi> <mrow><mi>c</mi><mi>h</mi></mrow></msub></math> |

用*p*表示每次交易是欺诈的概率。计算每种可能行动的预期净收益，你得到：

<math alttext="上 E 左括号净收益竖线接受右括号等于 p 左括号 t-c 下标 f Baseline 右括号加左括号 1 减 p 右括号 t 等于 t 减 pc 下标 f" display="block"><mrow><mi>E</mi><mrow><mo>(</mo><mtext>净收益</mtext><mtext>竖线</mtext><mtext>接受</mtext><mo>)</mo></mrow><mo>=</mo><mi>p</mi><mrow><mo>(</mo><mi>t</mi><mo>-</mo><msub><mi>c</mi><mi>f</mi></msub><mo>)</mo></mrow><mo>+</mo><mrow><mo>(</mo><mn>1</mn><mo>-</mo><mi>p</mi><mo>)</mo></mrow><mi>t</mi><mo>=</mo><mi>t</mi><mo>-</mo><mi>p</mi><msub><mi>c</mi><mi>f</mi></msub></mrow></math><math alttext="上 E 左括号净收益竖线块右括号等于减左括号 1 减 p 右括号 c 下标 ch" display="block"><mrow><mi>E</mi><mrow><mo>(</mo><mtext>净收益</mtext><mtext>竖线</mtext><mtext>块</mtext><mo>)</mo></mrow><mo>=</mo><mo>-</mo><mrow><mo>(</mo><mn>1</mn><mo>-</mo><mi>p</mi><mo>)</mo></mrow><msub><mi>c</mi><mrow><mi>c</mi><mi>h</mi></mrow></msub></mrow></math>

阻止一个票面为*t*的交易是最优的，只要阻止的净收益超过接受交易的净收益：

<math alttext="upper E left-parenthesis net benefits vertical-bar block right-parenthesis minus upper E left-parenthesis net benefits vertical-bar accept right-parenthesis equals p c Subscript f Baseline minus left-parenthesis t plus left-parenthesis 1 minus p right-parenthesis c Subscript c h Baseline right-parenthesis greater-than-or-equal-to 0" display="block"><mrow><mi>E</mi> <mrow><mo>(</mo> <mtext>net</mtext> <mtext>benefits</mtext> <mo>|</mo> <mtext>block</mtext> <mo>)</mo></mrow> <mo>-</mo> <mi>E</mi> <mrow><mo>(</mo> <mtext>net</mtext> <mtext>benefits</mtext> <mo>|</mo> <mtext>accept</mtext> <mo>)</mo></mrow> <mo>=</mo> <mi>p</mi> <msub><mi>c</mi> <mi>f</mi></msub> <mo>-</mo> <mrow><mo>(</mo> <mi>t</mi> <mo>+</mo> <mrow><mo>(</mo> <mn>1</mn> <mo>-</mo> <mi>p</mi> <mo>)</mo></mrow> <msub><mi>c</mi> <mrow><mi>c</mi><mi>h</mi></mrow></msub> <mo>)</mo></mrow> <mo>≥</mo> <mn>0</mn></mrow></math>

这最后一个不等式是业务案例的核心。在收益方面，如果交易是欺诈的，通过阻止可以节省欺诈成本（*c[f]*）。在成本方面，通过阻止交易，你有效地忽略了收入*t*，并且如果交易不是欺诈的话，会产生流失成本（*c[ch]*）的潜在成本。

和之前一样，让我们把注意力转向杠杆。除了阻止或接受，你总是可以选择一个限制（*L*），使得高额的票会被阻止，其他任何事情都会被接受。但是这个不等式中的限制在哪里？

欺诈的概率通常是这个限制的函数：*p(t|L)*。在许多应用中，当欺诈者寻求短期和快速的相对较大回报时，这个函数通常是增加的。通过设置一个足够大的限制，你可以集中精力处理高概率的交易。欺诈的成本通常是票本身，因此这也对收益有直接影响。然而存在一种权衡：如果交易不是欺诈的，你可能面临高价值客户的流失。

# 购买外部数据集

这种逻辑适用于你想分析的任何决策。在不深入细节的情况下，我会简要讨论购买外部数据集的情况，这是大多数数据科学团队在某个时候会评估的决策。

成本是您的数据提供者决定收费的任何内容。好处是您的公司可以通过数据创造的增量收入。在某些情况下，这是直接的，因为数据本身改善了决策过程。我想到的是 KYC（了解您的客户）或身份管理等用例。在这些情况下，您几乎可以将数据映射到收入上。

在大多数从数据科学角度来看有趣的其他案例中，增量收入依赖于关键假设。例如，如果您已经在生产中使用了一个决策过程中的机器学习模型，您可以量化使业务案例呈正增长的最小增量性能，考虑到这个成本。或者，您可以尝试在考虑这种增量性能的情况下谈判更好的条款。

这个想法可以通过这样的方式概括：

<math alttext="KPI left-parenthesis augmented dataset right-parenthesis minus KPI left-parenthesis original dataset right-parenthesis greater-than-or-equal-to c" display="block"><mrow><mtext>KPI</mtext> <mo>(</mo> <mtext>augmented</mtext> <mtext>dataset</mtext> <mo>)</mo> <mo>-</mo> <mtext>KPI</mtext> <mo>(</mo> <mtext>original</mtext> <mtext>dataset</mtext> <mo>)</mo> <mo>≥</mo> <mi>c</mi></mrow></math>

KPI 是您的 ML 模型性能度量的*函数*。我强调函数部分，因为您需要能够将性能指标转换为货币价值，如收入，以便与成本进行比较。请注意，通过使用原始数据集作为基准，您仅考虑增量效应。

# 在数据科学项目上工作

正如在第一章中建议的那样，数据科学家应该参与对公司具有增量作用的项目。假设您有两个备选项目，*A*和*B*。您应该从哪一个开始？使用相同的逻辑，如果：

<math alttext="revenue left-parenthesis upper A right-parenthesis minus cost left-parenthesis upper A right-parenthesis greater-than-or-equal-to revenue left-parenthesis upper B right-parenthesis minus cost left-parenthesis upper B right-parenthesis" display="block"><mrow><mtext>revenue</mtext> <mo>(</mo> <mi>A</mi> <mo>)</mo> <mo>-</mo> <mtext>cost</mtext> <mo>(</mo> <mi>A</mi> <mo>)</mo> <mo>≥</mo> <mtext>revenue</mtext> <mo>(</mo> <mi>B</mi> <mo>)</mo> <mo>-</mo> <mtext>cost</mtext> <mo>(</mo> <mi>B</mi> <mo>)</mo></mrow></math>

要做出决策，您需要填入一些数字，计算本身就是一个项目。重要的是从不平等中获得的直觉：优先考虑那些有实质性增量净收入的项目，考虑*您*的实施成本。

在第四章中，我展示了一个简单的 2×2 框架如何帮助您通过价值和努力轴对每个项目进行排名来优化您的工作流程。尽管这个图形设备非常有用，但您可能会在排名在一个维度上占主导地位并在另一个维度上被主导的项目中遇到麻烦（例如，图 4-5 中的项目*x*和*z*）。通过使用一个公共规模（货币）来评估努力（成本）和收入，前面的不平等解决了这个问题。

# 关键收获

这些是本章的关键收获：

相关性

学习撰写业务案例对于利益相关者管理和极端所有权目的至关重要，以及在各种备选项目之间分配数据科学资源。

撰写业务案例的原则

通常，您需要了解成本和收益，以及盈亏平衡。只关注增量变化。很多时候，您只需要关心影响您平均客户的单位经济学。

# 进一步阅读

在我的书籍*Analytical Skills for AI and Data Science*中，我描述了一些技术，可以帮助您简化业务案例，专注于一阶效应。它还将帮助您理解不确定情况下的决策制定。

这种成本效益分析在经济分析中很常见。我在这里称为*增量性*的东西通常被称为*边际分析*。我推荐给非经济学家的三本书是：由 Steven E. Landsburg（Free Press）撰写的*The Armchair Economist: Economics and Everyday Life*，由 Kate Raworth（Chelsea Green Publishing）撰写的*Doughnut Economics: Seven Ways to Think Like a 21st-Century Economist*，以及由 Charles Wheelan（W. W. Norton）撰写的*Naked Economics: Undressing the Dismal Science*。
