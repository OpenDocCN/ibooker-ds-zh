# 第十二章 数学逻辑

> *人类会打破规则……*

在人工智能领域的历史上，基于逻辑的代理人出现在基于机器学习和神经网络的代理人之前。我们之所以在逻辑之前讨论了机器学习、神经网络、概率推理、图形表示和运算研究，是因为我们希望将所有这些内容融入到一个关于代理人推理的叙事中，而不是将逻辑看作是古老的，神经网络看作是现代的。我们希望将最近的进展视为增强逻辑人工智能代理人表示和推理世界的方式。一个好的思考方式类似于启蒙：一个 AI 代理人过去使用手工编码的知识库和规则来进行推理和决策，然后突然之间它得到启蒙，变得更具推理工具、网络和神经元，使其能够扩展其知识库和推理方法。这样，它就具有了更多的表达能力，可以应对更复杂和不确定的情况。此外，将所有工具结合在一起将使代理人有时可以打破更严格逻辑框架的规则，根据情况采用更灵活的规则，就像人类一样。打破、弯曲甚至改变规则是人类独有的特征。

单词*逻辑*的字典含义为本章设定了基调，并证明了其进展。

# 逻辑

组织用于合理思考和推理的规则和过程的框架。这是一个奠定了进行推理和推断的有效性原则的框架。

上述定义中需要注意的最重要的词是*框架*和*推理原则*。逻辑系统在一个代理人内部编码了指导可靠推理和正确证明的原则。设计能够收集知识、逻辑推理的代理人，使用一个灵活的逻辑系统，适应他们存在的环境中的不确定性，并基于这种逻辑推理进行推断和决策，是人工智能的核心。

我们讨论了我们可以编程到代理人中的各种数学逻辑系统。目标是赋予 AI 代理人能力，使其能够进行适当的推理。这些逻辑框架需要知识库来配合不同大小的推理规则。它们还具有不同程度的表达和演绎能力。

# 各种逻辑框架

在本章中我们将重点介绍的各种逻辑框架（*命题、一阶、时间、概率和模糊*），我们将回答两个关于它们在代理人中如何运作的问题：

1.  代理人的世界中存在哪些对象？也就是说，代理人如何感知其世界的构成？

1.  代理人如何感知对象的状态？也就是说，在特定的逻辑框架下，代理人可以为其世界中的每个对象分配什么值？

如果我们将我们的代理人比作一只[蚂蚁及其体验世界的方式](http://astronomy.nmsu.edu/geas/lectures/lecture28/slide03.xhtml)，就很容易理解：由于蚂蚁预先确定的感知框架和允许的移动方式，蚂蚁将世界，以及其曲率，体验为二维。如果蚂蚁得到增强并赋予更具表现力的感知框架和允许的移动方式（例如翅膀），那么它将体验到三维世界。

# 命题逻辑

+   代理人的世界中存在哪些对象？

*简单或复杂的陈述，称为命题，因此称为命题逻辑。*

+   代理人如何感知对象的状态？

*真（1），假（0），或未知*。命题逻辑也被称为布尔逻辑，因为其中的对象只能有两种状态。命题逻辑中的悖论是指无法分类为真或假的陈述，根据逻辑框架的*真值表*。

以下是陈述及其状态的示例：

+   正在下雨（可以是真也可以是假）

+   埃菲尔铁塔在巴黎（始终为真）

+   公园里有可疑活动（可以是真也可以是假）

+   这句话是假的（悖论）

+   我快乐 *且* 我悲伤（始终为假，除非你问我的丈夫）

+   我快乐 *或* 我悲伤（始终为真）

+   如果分数是 13，那么学生就不及格（真假取决于不及格的阈值，因此我们需要在知识库中添加一个陈述，说所有分数低于 16 的学生都不及格，并将其值设为真）。

+   1+2 等于 2+1（在具有算术规则的代理中始终为真）。

+   巴黎浪漫（在命题逻辑中，这必须是真或假，但在模糊逻辑中，它可以在零到一的范围内取值，例如 0.8，这更符合我们感知世界的方式，以一个范围而不是绝对值。当然，如果我在编程一个代理并且受限于命题逻辑，我会为这个陈述分配真值，但是一个讨厌巴黎的人会分配假值。哦，好吧）。

命题逻辑世界中的对象是简单陈述和复杂陈述。我们可以使用五个允许的运算符从简单陈述中形成复杂陈述：非（否定）；和；或；蕴含（与*如果那样*相同）；等同于（与*如果且仅如果*相同）。

我们还有五条规则来确定一个陈述是真还是假：

1.  如果一个陈述的否定为真，那么该陈述为假。

1.  <math alttext="s t a t e m e n t 1"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>1</mn></msub></mrow></math> *且* <math alttext="s t a t e m e n t 2"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>2</mn></msub></mrow></math> 为真，只有当<math alttext="s t a t e m e n t 1"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>1</mn></msub></mrow></math> 和<math alttext="s t a t e m e n t 2"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>2</mn></msub></mrow></math> 都为真时才为真。

1.  如果<math alttext="s t a t e m e n t 1"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>1</mn></msub></mrow></math> *或* <math alttext="s t a t e m e n t 2"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>2</mn></msub></mrow></math> 为真，那么只有当<math alttext="s t a t e m e n t 1"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>1</mn></msub></mrow></math> 或<math alttext="s t a t e m e n t 2"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>2</mn></msub></mrow></math> 为真时，该语句为真（或者两者都为真）。

1.  <math alttext="s t a t e m e n t 1"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>1</mn></msub></mrow></math> *implies* <math alttext="s t a t e m e n t 2"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>2</mn></msub></mrow></math> 是真的，除非当 <math alttext="s t a t e m e n t 1"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>1</mn></msub></mrow></math> 为真且 <math alttext="s t a t e m e n t 2"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>2</mn></msub></mrow></math> 为假时。

1.  <math alttext="s t a t e m e n t 1"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>1</mn></msub></mrow></math> *equivalent to* <math alttext="s t a t e m e n t 2"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>2</mn></msub></mrow></math> 当且仅当 <math alttext="s t a t e m e n t 1"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>1</mn></msub></mrow></math> 和 <math alttext="s t a t e m e n t 2"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>2</mn></msub></mrow></math> 都为真或都为假。

我们可以通过一个*真值表*总结上述规则，考虑到 <math alttext="s t a t e m e n t 1"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>1</mn></msub></mrow></math> 和 <math alttext="s t a t e m e n t 2"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>2</mn></msub></mrow></math> 的状态以及它们的组合使用五个允许的运算符。在下面的真值表中，我们使用 <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> 代表 <math alttext="s t a t e m e n t 1"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>1</mn></msub></mrow></math>，使用 <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> 代表 <math alttext="s t a t e m e n t 2"><mrow><mi>s</mi> <mi>t</mi> <mi>a</mi> <mi>t</mi> <mi>e</mi> <mi>m</mi> <mi>e</mi> <mi>n</mi> <msub><mi>t</mi> <mn>2</mn></msub></mrow></math> 以节省空间：

| <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> | <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> | not <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> | <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> and <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> | <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> or <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> | <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> implies <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> | <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> equivalent to <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> |
| --- | --- | --- | --- | --- | --- | --- |
| F | F | T | F | F | T | T |
| F | T | T | F | T | T | F |
| T | F | F | F | T | F | F |
| T | T | F | T | T | T | T |

我们可以通过简单的递归评估使用上述表格计算任何复杂语句的真值。例如，如果我们处于一个 <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> 为真，<math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> 为假，且 <math alttext="upper S 3"><msub><mi>S</mi> <mn>3</mn></msub></math> 为真的世界中，那么该语句：

*非* <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *且* ( <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> *或* <math alttext="upper S 3"><msub><mi>S</mi> <mn>3</mn></msub></math> ) <math alttext="long left right double arrow"><mo>⟺</mo></math> F *且*(F *或* T) = F *且* T = F。

为了能够使用命题逻辑推理和证明定理，建立逻辑等价是有帮助的，意味着具有完全相同真值表的陈述，因此它们可以在推理过程中相互替换。以下是一些逻辑等价的示例：

+   与的交换律： <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *与* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> <math alttext="long left right double arrow"><mo>⟺</mo></math> <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> *与* <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> 。

+   或的交换律： <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *或* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> <math alttext="long left right double arrow"><mo>⟺</mo></math> <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> *或* <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> 。

+   双重否定消除：非(非 <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> ) <math alttext="long left right double arrow"><mo>⟺</mo></math> <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> 。

+   逆否命题： <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *蕴含* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> <math alttext="long left right double arrow"><mo>⟺</mo></math> 非( <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> ) *蕴含* 非( <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> ) 。

+   蕴含消除： <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *蕴含* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> <math alttext="long left right double arrow"><mo>⟺</mo></math> 非( <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> ) *或* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> 。

+   德摩根定律：非( <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *且* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> ) <math alttext="long left right double arrow"><mo>⟺</mo></math> 非( <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> ) *或* 非( <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> )。

+   德摩根定律：非( <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *或* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> ) <math alttext="long left right double arrow"><mo>⟺</mo></math> 非( <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> ) *且* 非( <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> )。

让我们通过展示它们具有相同的真值表来证明 <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *蕴含* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> <math alttext="long left right double arrow"><mo>⟺</mo></math> 非( <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> ) *或* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math>，因为这种等价对于一些人来说并不那么直观：

| <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> | not ( <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> ) | <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> | not( <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> ) *or* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> | <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *implies* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> |
| --- | --- | --- | --- | --- |
| F | T | F | T | T |
| F | T | T | T | T |
| T | F | T | T | T |
| T | F | F | F | F |

展示逻辑等价性有用的一个例子是*反证法*推理方式：为了证明陈述 <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> 意味着陈述 <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> ，我们可以假设我们有 <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> ，但同时我们没有 <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> ，然后我们得出一个错误或荒谬的结论，这证明我们不能假设 <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> 而不得出 <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> 。我们可以使用命题逻辑等价性验证这种证明方式的有效性，即 <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> 意味着 <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> ：

<math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *意味着* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> = 真 <math alttext="long left right double arrow"><mo>⟺</mo></math>

非（ <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> ）*或* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> = 真（蕴含消除） <math alttext="long left right double arrow"><mo>⟺</mo></math>

非（非（ <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> ）*或* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> ）= 非（真） <math alttext="long left right double arrow"><mo>⟺</mo></math>

<math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *和* 非（ <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> ）= 假（德摩根定律和双重否定）。

我们赋予命题逻辑框架*推理规则*，这样我们就能够从一个陈述（简单或复杂）顺序推理到下一个，并达到所需的目标或正确证明一个陈述。以下是一些伴随命题逻辑的推理规则：

+   如果 <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *意味着* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> 是真的，并且我们已知 <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> ，那么我们可以推断 <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> 。

+   如果 <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *和* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> 是真的，那么我们可以推断 <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> 。同样，我们也可以推断 <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> 。 

+   如果 <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *等价于* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> ，那么我们可以推断（ <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *意味着* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> ）*和*（ <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> *意味着* <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> ）

+   相反，如果（<math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *蕴含* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> ）*和*（<math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> *蕴含* <math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> ）那么我们可以推断出（<math alttext="upper S 1"><msub><mi>S</mi> <mn>1</mn></msub></math> *等价于* <math alttext="upper S 2"><msub><mi>S</mi> <mn>2</mn></msub></math> ）。

最后强调命题逻辑不适用于大型环境，并且不能有效地捕捉普遍关系模式。然而，命题逻辑为一阶逻辑和高阶逻辑提供了基础，因为这些逻辑建立在命题逻辑的机制之上。

## 从少量公理到整个理论

上述推理规则是*正确*的：它们只允许我们证明真实陈述，即给定一个真实陈述和一个与之相符的推理规则，我们得到一个真实陈述。因此，正确推理规则提供的保证是它们不允许从真实陈述中推导出错误陈述。我们需要比这个保证稍微多一点。

当我们能够仅使用系统的知识库（公理）和推理规则推断*所有*可能的真实陈述时，逻辑框架是*完备*的。系统的*完备性*的概念非常重要：在所有数学系统中，如数论、概率论、集合论或欧几里得几何中，我们从一组公理开始（数论和数学分析的皮亚诺公理，概率论的概率公理），然后使用逻辑推理规则从这些公理中推导定理。在任何数学理论中的一个主要问题是公理和推理规则是否确保其完备性和一致性。

*然而，没有一阶理论有足够的力量来唯一描述具有无限域的结构，比如自然数或实数线。可以在更强的逻辑中获得完全描述这两个结构的公理系统（即，范畴公理系统），比如二阶逻辑。*

## 在代理中编码逻辑

在继续进行一阶逻辑之前，让我们回顾一下在具有命题逻辑的 AI 代理环境中学到的内容。以下过程很重要，对于更具表现力的逻辑来说也是一样的：

+   我们在形式为真实陈述的初始知识库（公理）中编写。

+   我们编写推理规则。

+   代理感知其世界当前状态的某些陈述。

+   代理可能有或没有目标陈述。

+   代理使用推理规则推断新的陈述，并决定要做什么（移动到下一个房间，打开门，设置闹钟，*等等*）。

+   代理系统的完备性（知识库和推理规则的结合）在这里很重要，因为它允许代理在足够的推理步骤中推导出*任何*可满足的目标陈述。

## 确定性和概率机器学习如何适应其中？

机器学习（包括）神经网络的前提是我们不将初始知识库编程到代理中，也不编写推理规则。相反，我们编写的是一种表示输入数据、期望输出和将输入映射到输出的假设函数的方法。然后代理通过优化目标函数（损失函数）来学习函数的参数。最后，代理使用学到的函数对新的输入数据进行推理。因此，在这种情况下，知识库和规则可以通过*学习期间*或*推理期间*进行分离。在学习期间，知识库是数据和假设函数，目标是最小化损失，规则是优化过程。学习后，代理使用学到的函数进行推理。

如果我们将确定性假设函数替换为数据特征的联合概率分布，我们可以以完全相同的方式思考概率机器学习模型。一旦代理学会了它，就可以用于推理。例如，贝叶斯网络对于不确定知识的作用类似于命题逻辑对于确定知识的作用。

# 一阶逻辑

+   代理的世界中存在哪些对象？

“陈述、对象和它们之间的关系。”

+   代理如何感知对象的状态？

“真（1）、假（0）或未知。”

命题逻辑非常适合说明基于知识的代理如何工作，并解释某种逻辑的“语言”和推理规则的基本规则。然而，命题逻辑在能够表示的知识和推理方式方面存在局限。例如，在命题逻辑中，这样的陈述：

所有年龄超过十八岁的用户都可以看到这则广告。

可以表达为一个“蕴含”陈述（与“如果那么”相同），因为这种语言存在于命题逻辑框架中。这就是我们如何在命题逻辑中将上述陈述表达为推理的方式：

（年满十八岁的用户意味着看广告）*并且*（年满十八岁 = T）然后我们可以推断（看广告 = T）。

现在让我们考虑一个略有不同的陈述：

一些年满十八岁的用户点击广告。

突然之间，命题逻辑的语言不足以表达上述陈述中的“一些”数量！仅依赖于命题逻辑的代理将不得不将整个陈述存储在其知识库中，然后不知道如何从中推断出任何有用信息。也就是说，假设代理获得了用户确实年满十八岁的信息，它无法预测用户是否会点击广告。

我们需要一种语言（或逻辑框架），其中的词汇包括“存在”和“对于所有”，这样我们就可以写出类似以下的内容：

对于所有年满十八岁的用户，存在一部分点击广告的用户。

这两个额外的量词正是“一阶逻辑”框架提供的。这种词汇量的增加使我们能够更经济地存储知识库中的内容，因为我们能够将知识分解为对象和它们之间的关系。例如，与其存储：

所有年满十八岁的用户都看广告；

年满十八岁的一些用户点击广告；

一些年满十八岁的用户购买产品；

点击广告的一些用户购买了产品；

作为一个只有命题逻辑框架的代理的知识库中的三个独立陈述（我们仍然不知道如何从中推断出任何有用信息），我们可以在一阶逻辑中存储两个陈述：

对于所有年满十八岁的用户，看广告 = T；

对于所有看广告 = T 的用户，存在一部分点击广告的用户；

对于所有点击广告的用户，存在一部分购买产品的用户。

请注意，在命题逻辑和一阶逻辑中，仅凭上述陈述，我们将无法推断特定年满十八岁的用户是否会点击广告或购买产品，甚至无法推断这样做的用户的百分比，但至少在一阶逻辑中，我们有语言来更简洁地表达相同的知识，并且以一种能够进行一些有用推断的方式。一阶逻辑与命题逻辑最显著的特征是，在命题逻辑已有的基础语言上增加了诸如“存在”和“对于所有”之类的量词，这些量词已经存在于命题逻辑中的“非”、“和”、“或”、“蕴含”和“等价于”之上。这一小的增加打开了一个表达对象与其描述以及它们之间关系的大门：

命题逻辑和一阶逻辑的强大之处在于它们的推理规则独立于领域及其知识库或公理集。现在，为了为特定领域，如数学领域或电路工程，开发一个知识库，我们必须仔细研究该领域，选择词汇，然后制定支持所需推理的公理集。

### *对于所有*和*存在*之间的关系

*对于所有*和*存在*通过否定相互连接。以下两个陈述是等价的：

+   所有年满十八岁以上的用户看到广告。

+   不存在一个年满十八岁以上的人不看广告。

在命题逻辑语言中，上述两个陈述翻译为：

+   对于所有用户，使得用户>18 为真，看到广告为真。

+   不存在这样的用户，使得用户>18 且看到广告是假的

这些是关系：

+   not(存在一个 x 使得 P 为真) <math alttext="long left right double arrow"><mo>⟺</mo></math> 对于所有 x，P 为假。

+   not(对于所有 x，P 为真) <math alttext="long left right double arrow"><mo>⟺</mo></math> 存在一个 x 使得 P 为假。

+   存在一个 x 使得 P 为真 <math alttext="long left right double arrow"><mo>⟺</mo></math> 不是对所有 x，P 为假。

+   对于所有 x，P 为真 <math alttext="long left right double arrow"><mo>⟺</mo></math> 不存在一个 x 使得 P 为假。

我们不能在不欣赏转向一阶逻辑所获得的表达能力的情况下离开这一部分。这种逻辑框架现在足以支持这样的断言和推理：

神经网络的普适逼近定理

粗略地说，普适逼近定理断言：对于所有连续函数，存在一个神经网络可以将函数近似到我们希望的程度。请注意，这并不告诉我们如何构建这样的网络，它只是断言其存在。尽管如此，这个定理足够强大，使我们对神经网络在各种应用中近似所有种类的输入到输出函数的成功感到不惊讶。

推断关系

父母和孩子之间具有相反的关系：如果 Sary 是 Hala 的孩子，则 Hala 是 Sary 的母亲。此外，关系是单向的：Sary 不能是 Hala 的母亲。在一阶逻辑中，我们可以分配两个指示关系的函数：*母亲*和*孩子*，可以由*Hala*和*Sary*或任何其他母亲和孩子填充的变量，以及这些*函数*之间对所有输入变量成立的关系：

对于所有的 x，y，如果 mother(x,y)=T，则 mother(y,x)=F；

对于所有的 x，y，mother(x,y) <math alttext="long left right double arrow"><mo>⟺</mo></math> child(y,x)。

现在，如果我们给一个代理提供这些知识，并告诉它 Hala 是 Sary 的母亲，或者 mother(Hala, Sary)=T，那么它将能够回答类似以下的查询：

+   Hala 是 Sary 的母亲吗？T

+   Sary 是 Hala 的母亲吗？F

+   Sary 是 Hala 的孩子吗？T

+   Hala 是 Sary 的孩子吗？F

+   Laura 是 Joseph 的母亲吗？未知

请注意，在命题逻辑世界中，我们将不得不将每个陈述单独存储，这是极其低效的。

# 概率逻辑

+   代理的世界中存在哪些对象？

*陈述*

+   代理如何感知对象的状态？

*一个介于 0 和 1 之间的概率值，表示一个陈述为真的可能性。*

概率是一阶逻辑的扩展，允许我们量化对陈述真实性的不确定性。我们不是断言一个陈述是真还是假，而是为我们对该陈述真实性的信念程度分配一个介于零和一之间的分数。命题和一阶逻辑提供了一组推理规则，允许我们确定一些陈述的真实性，假设其他一些陈述为真。概率理论提供了一组推理规则，允许我们确定一个陈述在其他陈述的真实性可能性的基础上有多大可能是真的。

处理不确定性的这种扩展比一阶逻辑具有更具表现力的框架。概率公理允许我们扩展传统逻辑真值表和推理规则：例如，P(A)+P(not (A))=1：如果 A 为真，则 P(A)=1 且 P(not A)=0，这与关于陈述及其否定的一阶逻辑一致。

将概率理论视为一阶逻辑的自然扩展对于需要将事物连接在一起而不是将它们视为不同事物的思维是令人满意的。这样看待也自然地导致了关于数据的贝叶斯推理，因为我们在收集更多知识并做出更好推断时更新代理人的先验分布。这以最*逻辑*的方式将我们所有的主题联系在一起。

# 模糊逻辑

在代理人的世界中存在哪些对象？具有[0,1]之间真实程度的陈述。代理人如何感知对象的状态？已知的区间值。

命题和一阶逻辑的世界是黑白分明的，真或假。它们允许我们从真实陈述开始推断其他真实陈述。这种设置非常适合数学，其中一切都可以是对或错（真或假），或者适合具有非常明确边界的 SIMS 的视频游戏。在现实世界中，许多陈述可能是模糊的，无论它们是完全真（1）还是完全假（0），这意味着它们存在于*真实度*的尺度上，而不是在边缘上：*巴黎浪漫*；*她很快乐*；*黑暗骑士电影很好*。*模糊逻辑*允许这样做，并为陈述分配 0 到 1 之间的值，而不是严格的 0 或严格的 1：

巴黎浪漫（0.8）；她很快乐（0.6）；黑暗骑士电影很好（0.9）。

在一个真假程度呈滑动尺度的模糊世界中如何进行推理？这绝对不像在真假世界中进行推理那样直截了当。例如，给定上述真值，陈述巴黎浪漫*且*她很快乐有多真实？我们需要新的规则来分配这些值，并且我们需要了解上下文或领域知识。另一个选择是单词向量，我们在第七章中讨论过。这些向量在不同维度中携带单词的含义，因此我们可以计算代表单词巴黎的向量与代表浪漫点的向量之间的余弦相似度，并将其分配为巴黎浪漫的陈述的真值。

请注意，概率理论中的信念程度与模糊逻辑中的真实度尺度不同。在概率逻辑中，陈述本身是明确的。我们想要推断的是明确陈述为真的概率。概率理论不推理不完全真实或假的陈述：我们不计算巴黎浪漫的概率，而是计算随机询问巴黎是否浪漫的人会回答真或假的概率。

关于模糊逻辑的一个有趣之处在于，它将其他逻辑中存在的两个原则抛在一边：如果一个陈述是真的，那么它的否定就是假的原则，以及两个矛盾的陈述不能同时为真的原则。这实际上打开了不一致性和*开放宇宙*的大门。在某种程度上，模糊逻辑并不试图纠正模糊性，而是接受它并利用它来允许在边界不清晰的世界中运作。

# 时间逻辑

还有其他类型的特殊用途逻辑，其中某些对象，比如本节中的“时间”，受到特别关注，有自己的公理和推理规则，因为它们对需要表示的知识和对其进行推理至关重要。*时间逻辑*将时间依赖性以及关于时间依赖性的公理和推理规则置于其结构的前沿，而不是将包含时间信息的陈述添加到知识库中。在时间逻辑中，陈述或事实在某些时间点或时间间隔上是真实的，这些时间是有序的。

代理的世界中存在哪些对象？陈述、对象、关系、时间。代理如何感知对象的状态？真（1）、假（0）或未知。

在时间逻辑中，我们可以表示如下陈述：

+   当时间是早上 7 点时，闹钟会响。

+   每当向服务器发出请求时，最终会授予访问权限，但永远不会同时授予两个请求。

# 与人类自然语言的比较

我们在整章中讨论了能够表达人类自然语言似乎轻松做到的知识的逻辑系统。我刚刚用英语写了一本关于数学的整本书，而不是使用任何其他技术语言。我们是如何做到的？人类如何表示和扩展他们的知识库，自然语言使用什么规则来表示和推理，使其如此富有表现力？此外，使用的特定自然语言并不重要：任何多语言的说话者都知道思想，但不一定知道他们用来表达这种思想的特定语言。人们知道或想要表达的东西有一种内在的非语言表示。它是如何工作的，我们如何解锁它的秘密并将其传授给我们的机器？

与人类语言类似，如果我们用两种不同的形式逻辑来表示相同的知识，那么我们可以推断相同的事实（假设逻辑具有完整的推理规则）。唯一的区别在于哪种逻辑框架提供了更容易的推理路径。

也就是说，人类自然语言在许多情况下允许模糊性，并且不能在没有数学形式和所采用的形式逻辑的情况下做出绝对的数学断言。我不能要求一个没有 GPS 系统的人预测从 DC 到 NYC 的驾驶时间。

# 总结和展望

一个具有各种类型逻辑的 AI 代理可以表达关于世界的知识，对其进行推理，回答查询，并进行推断，这些都在这些逻辑的边界内是允许的。

我们讨论了各种逻辑框架，包括命题逻辑、一阶逻辑、概率逻辑、模糊逻辑和时间逻辑。

下一个自然的问题将是：代理的知识库应该包含什么内容？如何表示关于世界的事实？知识应该在什么框架中表示和推理？

+   命题逻辑？

+   一阶逻辑？

+   用于推理计划的分层任务网络？

+   用于处理不确定性推理的贝叶斯网络？

+   因果图和因果推理，代理可以有选择地违反逻辑规则？

+   用于随时间推理的马尔可夫模型？

+   用于推理图像、声音或其他数据的深度神经网络？

另一个可能的下一步是深入研究我们讨论过的任何逻辑框架，学习它们的推理规则和现有的推理算法，以及它们的优势、劣势以及适用于哪种知识库。这些研究中的一个重要主题是调查提供完整证明系统的推理规则，意味着一个系统，其中公理或知识库以及规则允许证明*所有*可能的真陈述。这些规则包括命题逻辑的*分辨推理规则*和一阶逻辑的*广义分辨推理规则*，适用于特定类型的知识库。这些对于理论（证明数学定理）和技术（验证和合成）软件和硬件都很重要。最后，一些逻辑比其他逻辑*更具表现力*，意思是我们可以用更具表现力的逻辑表示的一些陈述，不能用较不具表现力的逻辑语言中的任何有限数量的陈述来表达。例如，*高阶逻辑*（我们在本章中没有讨论）比一阶逻辑（我们在本章中讨论过，足够强大以支持整个数学理论）更具表现力。
