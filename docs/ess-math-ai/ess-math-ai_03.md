# 第三章。将函数拟合到数据

> *今天适合。明天呢？*

在本章中，我们介绍了许多人工智能应用的核心数学思想，包括神经网络的数学引擎。我们的目标是内化人工智能问题的机器学习部分的以下结构：

+   根据具体用例识别问题：分类图像，分类文档，预测房价，检测欺诈或异常，推荐下一个产品，预测犯罪分子再次犯罪的可能性，根据外部图像预测建筑的内部结构，将语音转换为文本，生成音频，生成图像，生成视频，*等等*。

+   获取适当的数据，以便*训练*我们的模型做正确的事情。我们说我们的模型从数据中*学习*。确保这些数据是干净的、完整的，并且如果必要的话，根据我们正在实施的具体模型进行转换（归一化、标准化、一些特征聚合，*等等*）。这一步通常比实施和训练机器学习模型耗费更多时间。

+   创建一个*假设函数*。我们可以互换使用假设函数、*学习函数*、*预测函数*、*训练函数*和*模型*这些术语。我们的主要假设是，这个输入/输出的数学函数解释了观察到的数据，并且可以在以后用于对新数据进行预测。我们给我们的模型特征，比如一个人的日常习惯，它返回一个预测，比如这个人偿还贷款的可能性。在本章中，我们将给我们的模型鱼的长度测量，并且它将返回它的重量。

+   我们将遇到许多模型（包括神经网络），其中我们的训练函数具有称为*权重*的未知参数。目标是使用数据找到这些权重的数值。在找到这些权重值之后，我们可以使用*训练*函数进行预测，将新数据点的特征插入到训练函数的公式中。

+   为了找到未知权重的值，我们创建*另一个函数*，称为*误差函数*、*成本函数*、*目标函数*或*损失函数*（人工智能领域的一切都有三个或更多的名称）。这个函数必须衡量地面真相和我们的预测之间的某种距离。自然地，我们希望我们的预测尽可能接近地面真相，因此我们寻找最小化我们损失函数的权重值。从数学上讲，我们解决了一个最小化问题。*数学优化*领域对人工智能至关重要。

+   在整个过程中，我们是工程师，所以我们决定训练函数、损失函数、优化方法和计算机实现的数学公式。不同的工程师决定不同的流程，产生不同的性能结果，这是可以接受的。最终的评判是部署模型的性能，与普遍看法相反，数学模型是灵活的，可以在需要时进行调整和修改。部署后监控性能至关重要。

+   由于我们的目标是找到最小化预测和实际值之间误差的权重数值，我们需要找到一种高效的数学方法来搜索这些*最小化者*：那些产生最小误差的特殊权重数值。*梯度下降*方法在这里起着关键作用。这种强大而简单的方法涉及计算我们误差函数的*一个导数*。这就是我们花了一半的微积分课程计算导数的原因之一（以及梯度：这是高维度中的一个导数）。还有其他需要计算两个导数的方法。我们将遇到它们，并评论使用高阶方法的利弊。

+   当数据集非常庞大，而我们的模型恰好是一个分层神经网络时，我们需要一种高效的方法来计算这个导数。*反向传播算法*就是在这一点上发挥作用。我们将在下一章中讨论梯度下降和反向传播。

+   如果我们的学习函数对给定数据拟合得太好，那么它在新数据上表现不佳。原因是对数据拟合得太好的函数意味着它既捕捉到了数据中的噪音，也捕捉到了信号（例如，[图3-1](#Fig_noise_fit_regular_fit)左侧的函数）。我们不希望捕捉到噪音。这就是*正则化*发挥作用的地方。有多种数学方法可以使函数正则化，这意味着使其更加平滑和不那么振荡和不规则。一般来说，跟随数据中的噪音的函数振荡太多。我们希望更规则的函数。我们将在下一章中介绍正则化技术。

![280](assets/emai_0301.png)

###### 图3-1。左：拟合函数完美地拟合了数据，但它不是一个很好的预测函数，因为它拟合了数据中的噪音而不是主要信号。右：一个更规则的函数拟合了相同的数据集。使用这个函数将比左侧子图中的函数给出更好的预测，即使左侧子图中的函数更好地匹配了数据点。

在接下来的章节中，我们将用真实但简单的数据集探索AI问题的上述结构。我们将在接下来的章节中看到相同的概念如何推广到更复杂的任务。

# 传统且非常有用的机器学习模型

本章中使用的所有数据都带有地面真相标签，我们模型的目标是*预测*新的（未见过的）和未标记的数据的标签。这是*监督*学习。

在接下来的几节中，我们将使用以下流行的机器学习模型将训练函数拟合到我们的标记数据中。虽然您可能会听到关于AI最新和最伟大发展的许多消息，但在典型的商业环境中，您可能最好从这些更传统的模型开始：

1.  **线性回归**：预测数值。

1.  **逻辑回归**：分类到两个类别（二元分类）。

1.  **Softmax回归**：分类到多个类别。

1.  **支持向量机**：分类到两个类别，或回归（预测数值）。

1.  **决策树**：分类到任意数量的类别，或回归（预测数值）。

1.  **随机森林**：分类到任意数量的类别，或回归（预测数值）。

1.  **模型集成**：通过平均预测值、投票最受欢迎的类别或其他捆绑机制来捆绑许多模型的结果。

我们在相同的数据集上尝试多个模型以进行性能比较。在现实世界中，很少有任何模型在没有与许多其他模型进行比较的情况下部署。这是计算密集型AI行业的特性，这也是为什么我们需要并行计算的原因，它使我们能够同时训练多个模型（除了那些构建和改进其他模型结果的模型，比如*堆叠*的情况，我们不能使用并行计算）。

在我们深入研究任何机器学习模型之前，非常重要的一点是，一再有报道称，数据科学家和/或人工智能研究人员只有大约百分之五的时间用于训练机器学习模型。大部分时间都用于获取数据、清理数据、组织数据、为数据创建适当的管道，*等等*，*在*将数据输入机器学习模型之前。因此，机器学习只是生产过程中的一步，一旦数据准备好训练模型就变得很容易。我们将发现这些机器学习模型是如何工作的：我们需要的大部分数学知识都存在于这些模型中。人工智能研究人员一直在努力改进机器学习模型，并将它们自动适应到生产管道中。因此，对我们来说，最终学习整个流程，从原始数据（包括其存储、硬件、查询协议，*等等*）到部署和监控，是非常重要的。学习机器学习只是更大更有趣故事的一部分。

我们必须从*回归*开始，因为回归的思想对于接下来的大多数人工智能模型和应用都是如此基础。只有对于*线性回归*，我们才会使用*解析*方法找到最小化权重，直接给出所需权重的显式公式，以训练数据集及其目标标签为参数。正是线性回归模型的简单性使得这种显式解析解成为可能。大多数其他模型没有这样的显式解，我们必须使用数值方法找到它们的最小值，其中梯度下降方法非常受欢迎。

在回归和许多其他即将出现的模型中，包括接下来几章的神经网络，要注意建模过程中的以下进展：

1.  训练函数

1.  损失函数

1.  优化

# 数值解*vs.*解析解

了解数学问题的数值解和解析解之间的区别非常重要。数学问题可以是任何东西，比如：

+   找到某个函数的最小值。

+   找到从目的地*A*到目的地*B*的最佳方式，预算受限。

+   找到设计和查询数据仓库的最佳方式。

+   寻找数学方程的解（其中左手边有数学内容*等于*右手边有数学内容）。这些方程可以是代数方程、常微分方程、偏微分方程、积分微分方程、方程组，或者任何类型的数学方程。它们的解可以是静态的，也可以随时间演变。它们可以模拟物理、生物、社会经济或自然世界的任何事物。

以下是词汇表：

+   *数值*：与数字有关。

+   *解析*：与分析有关。

一般来说，数值解比分析解容易获得得多，也更容易获得，只要我们有足够的计算能力来模拟和计算这些解。我们通常只需要离散化一些连续空间和/或函数，尽管有时需要非常巧妙的方法，并在这些离散量上评估函数。数值解的唯一问题是它们只是近似解。除非它们有估计值表明它们与真实分析解的差距有多大，以及它们收敛到这些真实解的速度有多快，而这又需要数学背景和分析，数值解并不是精确的。然而，它们确实提供了关于真实解的非常有用的见解。在许多情况下，数值解是唯一可用的解，许多科学和工程领域如果不依赖于复杂问题的数值解，就不会有任何进展。如果这些领域等待分析解和证明发生，或者换句话说，等待数学理论“赶上”，它们的进展将会非常缓慢。

另一方面，分析解是精确的、稳健的，并且有整个数学理论支持它们。它们伴随着定理和证明。当分析解可用时，它们非常强大。然而，它们并不容易获取，有时甚至是不可能的，并且确实需要在微积分、数学分析、代数、微分方程理论等领域具有深厚的知识和专业知识。然而，分析方法对于描述解的重要性质（即使明确的解不可用）、指导数值技术，并提供基本事实以比较近似数值方法（在这些分析解可用的幸运情况下）是极其有价值的。

一些研究人员纯粹是分析和理论的，另一些则是纯粹数值和计算的，而最好的存在位置是在接近交集的地方，我们对数学问题的分析和数值方面有一个良好的理解。

# 回归：预测一个数值

在[Kaggle网站](https://www.kaggle.com)上快速搜索回归数据集会返回许多优秀的数据集和相关笔记本。我随机选择了一个简单的[Fish Market](https://www.kaggle.com/aungpyaeap/fish-market)数据集，我们将用它来解释我们即将介绍的数学。我们的目标是构建一个模型，根据鱼的五种不同长度测量或特征来预测鱼的重量，这些特征在数据集中标记为：Length1、Length2、Length3、Height和Width（见[图3-2](#Fig_fish_data)）。为简单起见，我们选择不将分类特征Species纳入此模型，尽管我们可以（这样会给我们更好的预测，因为鱼的类型是其重量的良好预测因子）。如果我们选择包括Species特征，那么我们将不得不将其值转换为数值值，使用*one hot coding*，这意味着确切地说：根据其类别（类型）为每条鱼分配由一和零组成的代码。我们的Species特征有七个类别：鲈鱼、鲷鱼、鲫鱼、梭子鱼、胖鱼、Parkki和白鱼。因此，如果我们的鱼是梭子鱼，那么我们将把它的种类编码为（0,0,0,1,0,0,0），如果它是鲷鱼，我们将把它的种类编码为（0,1,0,0,0,0,0）。当然，这会给我们的特征空间增加七个维度，并增加七个权重来训练。

![280](assets/emai_0302.png)

###### 图3-2. 从Kaggle的[Fish Market](https://www.kaggle.com/aungpyaeap/fish-market)下载的鱼数据集的前五行。重量列是目标特征，我们的目标是构建一个模型，根据鱼的长度测量来预测新鱼的重量。

让我们节省墨水空间，并将我们的五个特征重新标记为：<math alttext="x 1"><msub><mi>x</mi> <mn>1</mn></msub></math> , <math alttext="x 2"><msub><mi>x</mi> <mn>2</mn></msub></math> , <math alttext="x 3"><msub><mi>x</mi> <mn>3</mn></msub></math> , <math alttext="x 4"><msub><mi>x</mi> <mn>4</mn></msub></math> , 和 <math alttext="x 5"><msub><mi>x</mi> <mn>5</mn></msub></math> , 然后将鱼的重量写成这五个特征的函数 <math alttext="y equals f left-parenthesis x 1 comma x 2 comma x 3 comma x 4 comma x 5 right-parenthesis"><mrow><mi>y</mi> <mo>=</mo> <mi>f</mi> <mo>(</mo> <msub><mi>x</mi> <mn>1</mn></msub> <mo>,</mo> <msub><mi>x</mi> <mn>2</mn></msub> <mo>,</mo> <msub><mi>x</mi> <mn>3</mn></msub> <mo>,</mo> <msub><mi>x</mi> <mn>4</mn></msub> <mo>,</mo> <msub><mi>x</mi> <mn>5</mn></msub> <mo>)</mo></mrow></math> . 这样，一旦我们确定了这个函数的一个可接受的公式，我们只需要输入某条鱼的特征值，我们的函数就会输出该鱼的预测重量。

本节为即将到来的一切构建了基础，因此首先看看它是如何组织的是很重要的：

**训练函数**

+   参数模型 *vs.* 非参数模型

**损失函数**

+   预测值 *vs.* 真实值

+   绝对值距离 *vs.* 平方距离

+   具有奇点（尖点）的函数

+   对于线性回归，损失函数是均方误差

+   本书中的向量始终是列向量

+   训练、验证和测试子集

+   当训练数据具有高度相关的特征时

**优化**

+   凸景观 *vs.* 非凸景观

+   我们如何找到函数的最小值点？

+   微积分简介

+   一维优化示例

+   我们一直在使用的线性代数表达式的导数

+   最小化均方误差损失函数

+   警告：将大矩阵相乘是非常昂贵的。应该将矩阵乘以向量。

+   警告：我们永远不希望训练数据拟合得太好

## 训练函数

对数据的快速探索，例如绘制重量与各种长度特征的关系，使我们可以假设一个线性模型（即使在这种情况下非线性模型可能更好）。也就是说，我们假设重量线性地依赖于长度特征（参见[图3-3](#Fig_weight_lengths_scatterplots)）。

![280](assets/emai_0303.png)

###### 图3-3\. 鱼市场数值特征的散点图。有关更多详细信息，请查看附加的Jupyter笔记本，或者与此数据集相关的一些公开的[Kaggle笔记本](https://www.kaggle.com/aungpyaeap/fish-market/code)。

这意味着鱼的重量y可以使用其五个不同长度测量的*线性组合*来计算，再加上一个偏置项 <math alttext="omega 0"><msub><mi>ω</mi> <mn>0</mn></msub></math> ，得到以下*训练函数*：

<math alttext="dollar-sign y equals omega 0 plus omega 1 x 1 plus omega 2 x 2 plus omega 3 x 3 plus omega 4 x 4 plus omega 5 x 5 dollar-sign"><mrow><mi>y</mi> <mo>=</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msub><mi>x</mi> <mn>1</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>2</mn></msub> <msub><mi>x</mi> <mn>2</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>3</mn></msub> <msub><mi>x</mi> <mn>3</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>4</mn></msub> <msub><mi>x</mi> <mn>4</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>5</mn></msub> <msub><mi>x</mi> <mn>5</mn></msub></mrow></math>

在我们在建模过程中做出的主要决定是使用线性训练函数f(x1, x2, x3, x4, x5)，我们所要做的就是找到参数ω0，ω1，ω2，ω3，ω4和ω5的适当值。我们将从数据中*学习*出ω的最佳值。利用数据找到适当的ω的过程称为*训练*模型。*训练*好的模型是指ω的值已经确定的模型。

一般来说，无论是线性还是非线性的训练函数，包括表示神经网络的函数，都有我们需要从给定数据中学习的未知参数ω。对于线性模型，每个参数在预测过程中给每个特征赋予一定的权重。因此，如果ω2的值大于ω5的值，那么第二个特征在我们的预测中起到比第五个特征更重要的作用，假设第二个和第五个特征具有可比较的规模。这是在训练模型之前对数据进行缩放或归一化的好处之一。另一方面，如果与第三个特征相关联的ω3值消失，即变为零或可以忽略，那么第三个特征可以从数据集中省略，因为它在我们的预测中没有作用。因此，从数据中学习我们的ω允许我们在数学上计算每个特征对我们的预测的贡献（或者在数据准备阶段合并一些特征时，特征组合的重要性）。换句话说，模型学习数据特征如何相互作用以及这些相互作用的强度。结论是，通过训练学习函数，我们可以量化特征如何相互作用以产生已观察到的和尚未观察到的结果。

# 注意：参数模型与非参数模型

一个具有预先内置参数（我们称之为权重）的模型，比如我们当前的线性回归模型中的ω：

<math alttext="dollar-sign y equals omega 0 plus omega 1 x 1 plus omega 2 x 2 plus omega 3 x 3 plus omega 4 x 4 plus omega 5 x 5 dollar-sign"><mrow><mi>y</mi> <mo>=</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msub><mi>x</mi> <mn>1</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>2</mn></msub> <msub><mi>x</mi> <mn>2</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>3</mn></msub> <msub><mi>x</mi> <mn>3</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>4</mn></msub> <msub><mi>x</mi> <mn>4</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>5</mn></msub> <msub><mi>x</mi> <mn>5</mn></msub></mrow></math>

（以及神经网络的<math alttext="omega"><mi>ω</mi></math>）被称为*参数模型*。这意味着我们在实际训练之前固定训练函数的公式，所有训练所做的就是解决公式中涉及的参数。提前固定公式类似于指定训练函数所属的*族*，找到参数值指定了最能解释数据的确切成员。

*非参数模型*，例如我们将在本章后面讨论的决策树和随机森林，不会提前指定训练函数及其参数的公式。因此，当我们训练非参数模型时，我们不知道训练模型最终会有多少参数。模型会根据数据*自适应*并确定所需的参数数量。在这里要小心，过度拟合的警钟正在响！请记住，我们不希望我们的模型过度适应数据，因为它们可能无法很好地推广到未见过的数据。这些模型通常配有帮助它们避免过拟合的技术。

参数化和非参数化模型都有*其他参数*称为*超参数*，在训练过程中也需要进行调整。然而，这些参数并没有内置到训练函数的公式中（非参数化模型的公式中也没有）。我们将在本书中遇到许多超参数。

## 损失函数

我们已经确信，下一个逻辑步骤是找到适合训练函数（我们的线性参数模型）中出现的<math alttext="omega"><mi>ω</mi></math>的合适值，使用我们拥有的数据。为了做到这一点，我们需要*优化适当的损失函数*。

### 预测值与真实值

假设我们为我们每个未知的<math alttext="omega 0"><msub><mi>ω</mi> <mn>0</mn></msub></math>，<math alttext="omega 1"><msub><mi>ω</mi> <mn>1</mn></msub></math>，<math alttext="omega 2"><msub><mi>ω</mi> <mn>2</mn></msub></math>，<math alttext="omega 3"><msub><mi>ω</mi> <mn>3</mn></msub></math>，<math alttext="omega 4"><msub><mi>ω</mi> <mn>4</mn></msub></math>和<math alttext="omega 5"><msub><mi>ω</mi> <mn>5</mn></msub></math>分配一些随机数值，例如<math alttext="omega 0等于负3"><mrow><msub><mi>ω</mi> <mn>0</mn></msub> <mo>=</mo> <mo>-</mo> <mn>3</mn></mrow></math>，<math alttext="omega 1等于4"><mrow><msub><mi>ω</mi> <mn>1</mn></msub> <mo>=</mo> <mn>4</mn></mrow></math>，<math alttext="omega 2等于0.2"><mrow><msub><mi>ω</mi> <mn>2</mn></msub> <mo>=</mo> <mn>0</mn> <mo>.</mo> <mn>2</mn></mrow></math>，<math alttext="omega 3等于0.03"><mrow><msub><mi>ω</mi> <mn>3</mn></msub> <mo>=</mo> <mn>0</mn> <mo>.</mo> <mn>03</mn></mrow></math>，<math alttext="omega 4等于0.4"><mrow><msub><mi>ω</mi> <mn>4</mn></msub> <mo>=</mo> <mn>0</mn> <mo>.</mo> <mn>4</mn></mrow></math>，和<math alttext="omega 5等于0.5"><mrow><msub><mi>ω</mi> <mn>5</mn></msub> <mo>=</mo> <mn>0</mn> <mo>.</mo> <mn>5</mn></mrow></math>。然后线性训练函数的公式<math alttext="y等于omega 0加omega 1 x 1加omega 2 x 2加omega 3 x 3加omega 4 x 4加omega 5 x 5"><mrow><mi>y</mi> <mo>=</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msub><mi>x</mi> <mn>1</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>2</mn></msub> <msub><mi>x</mi> <mn>2</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>3</mn></msub> <msub><mi>x</mi> <mn>3</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>4</mn></msub> <msub><mi>x</mi> <mn>4</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>5</mn></msub> <msub><mi>x</mi> <mn>5</mn></msub></mrow></math>变为：

<math alttext="dollar-sign y equals negative 3 plus 4 x 1 plus 0.2 x 2 plus 0.03 x 3 plus 0.4 x 4 plus 0.5 x 5 dollar-sign"><mrow><mi>y</mi> <mo>=</mo> <mo>-</mo> <mn>3</mn> <mo>+</mo> <mn>4</mn> <msub><mi>x</mi> <mn>1</mn></msub> <mo>+</mo> <mn>0</mn> <mo>.</mo> <mn>2</mn> <msub><mi>x</mi> <mn>2</mn></msub> <mo>+</mo> <mn>0</mn> <mo>.</mo> <mn>03</mn> <msub><mi>x</mi> <mn>3</mn></msub> <mo>+</mo> <mn>0</mn> <mo>.</mo> <mn>4</mn> <msub><mi>x</mi> <mn>4</mn></msub> <mo>+</mo> <mn>0</mn> <mo>.</mo> <mn>5</mn> <msub><mi>x</mi> <mn>5</mn></msub></mrow></math>

并准备好进行预测：为<math alttext="i Superscript t h"><msup><mi>i</mi> <mrow><mi>t</mi><mi>h</mi></mrow></msup></math>条鱼的长度特征插入数值，然后获得这条鱼的重量预测值。例如，我们数据集中的第一条鱼是鲱鱼，长度测量值为<math alttext="x 1 Superscript 1 Baseline equals 23.2"><mrow><msubsup><mi>x</mi> <mn>1</mn> <mn>1</mn></msubsup> <mo>=</mo> <mn>23</mn> <mo>.</mo> <mn>2</mn></mrow></math>，<math alttext="x 2 Superscript 1 Baseline equals 25.4"><mrow><msubsup><mi>x</mi> <mn>2</mn> <mn>1</mn></msubsup> <mo>=</mo> <mn>25</mn> <mo>.</mo> <mn>4</mn></mrow></math>，<math alttext="x 3 Superscript 1 Baseline equals 30"><mrow><msubsup><mi>x</mi> <mn>3</mn> <mn>1</mn></msubsup> <mo>=</mo> <mn>30</mn></mrow></math>，<math alttext="x 4 Superscript 1 Baseline equals 11.52"><mrow><msubsup><mi>x</mi> <mn>4</mn> <mn>1</mn></msubsup> <mo>=</mo> <mn>11</mn> <mo>.</mo> <mn>52</mn></mrow></math>，以及<math alttext="x 5 Superscript 1 Baseline equals 4.02"><mrow><msubsup><mi>x</mi> <mn>5</mn> <mn>1</mn></msubsup> <mo>=</mo> <mn>4</mn> <mo>.</mo> <mn>02</mn></mrow></math>。将这些值代入训练函数，我们得到了这条鱼的重量预测值：

<math alttext="dollar-sign StartLayout 1st Row 1st Column y Subscript p r e d i c t Superscript 1 2nd Column equals omega 0 plus omega 1 x 1 Superscript 1 Baseline plus omega 2 x 2 Superscript 1 Baseline plus omega 3 x 3 Superscript 1 Baseline plus omega 4 x 4 Superscript 1 Baseline plus omega 5 x 5 Superscript 1 Baseline 2nd Row 1st Column Blank 2nd Column equals negative 3 plus 4 left-parenthesis 23.2 right-parenthesis plus 0.2 left-parenthesis 25.4 right-parenthesis plus 0.03 left-parenthesis 30 right-parenthesis plus 0.4 left-parenthesis 11.52 right-parenthesis plus 0.5 left-parenthesis 4.02 right-parenthesis 3rd Row 1st Column Blank 2nd Column equals 102.398 grams period EndLayout dollar-sign"><mtable displaystyle="true"><mtr><mtd columnalign="right"><msubsup><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow> <mn>1</mn></msubsup></mtd> <mtd columnalign="left"><mrow><mo>=</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msubsup><mi>x</mi> <mn>1</mn> <mn>1</mn></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>2</mn></msub> <msubsup><mi>x</mi> <mn>2</mn> <mn>1</mn></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>3</mn></msub> <msubsup><mi>x</mi> <mn>3</mn> <mn>1</mn></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>4</mn></msub> <msubsup><mi>x</mi> <mn>4</mn> <mn>1</mn></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>5</mn></msub> <msubsup><mi>x</mi> <mn>5</mn> <mn>1</mn></msubsup></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>=</mo> <mo>-</mo> <mn>3</mn> <mo>+</mo> <mn>4</mn> <mo>(</mo> <mn>23</mn> <mo>.</mo> <mn>2</mn> <mo>)</mo> <mo>+</mo> <mn>0</mn> <mo>.</mo> <mn>2</mn> <mo>(</mo> <mn>25</mn> <mo>.</mo> <mn>4</mn> <mo>)</mo> <mo>+</mo> <mn>0</mn> <mo>.</mo> <mn>03</mn> <mo>(</mo> <mn>30</mn> <mo>)</mo> <mo>+</mo> <mn>0</mn> <mo>.</mo> <mn>4</mn> <mo>(</mo> <mn>11</mn> <mo>.</mo> <mn>52</mn> <mo>)</mo> <mo>+</mo> <mn>0</mn> <mo>.</mo> <mn>5</mn> <mo>(</mo> <mn>4</mn> <mo>.</mo> <mn>02</mn> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>=</mo> <mn>102</mn> <mo>.</mo> <mn>398</mn> <mtext>grams.</mtext></mrow></mtd></mtr></mtable></math>

通常，对于第i条鱼，我们有：

<math alttext="dollar-sign y Subscript p r e d i c t Superscript i Baseline equals omega 0 plus omega 1 x 1 Superscript i Baseline plus omega 2 x 2 Superscript i Baseline plus omega 3 x 3 Superscript i Baseline plus omega 4 x 4 Superscript i Baseline plus omega 5 x 5 Superscript i dollar-sign"><mrow><msubsup><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow> <mi>i</mi></msubsup> <mo>=</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msubsup><mi>x</mi> <mn>1</mn> <mi>i</mi></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>2</mn></msub> <msubsup><mi>x</mi> <mn>2</mn> <mi>i</mi></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>3</mn></msub> <msubsup><mi>x</mi> <mn>3</mn> <mi>i</mi></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>4</mn></msub> <msubsup><mi>x</mi> <mn>4</mn> <mi>i</mi></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>5</mn></msub> <msubsup><mi>x</mi> <mn>5</mn> <mi>i</mi></msubsup></mrow></math>

然而，考虑的鱼有一个*真实*重量，<math alttext="y Subscript t r u e Superscript i"><msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mi>i</mi></msubsup></math>，如果它属于标记数据集，则为其标签。对于我们数据集中的第一条鱼，真实重量是<math alttext="y Subscript t r u e Superscript 1 Baseline equals 242"><mrow><msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mn>1</mn></msubsup> <mo>=</mo> <mn>242</mn></mrow></math> *克*。我们随机选择的线性模型预测了102.398 *克*。这当然相差甚远，因为我们根本没有校准<math alttext="omega"><mi>ω</mi></math>值。无论如何，我们可以测量我们的模型预测的重量与真实重量之间的*误差*，然后找到更好的方法来选择<math alttext="omega"><mi>ω</mi></math>。

### 绝对值距离*vs*平方距离

数学的一大优点是它有多种方法来衡量事物之间的差距，使用不同的距离度量。例如，我们可以天真地将两个量之间的距离测量为如果它们不同则为1，如果它们相同则为0，编码为：不同-1，相似-0。当然，使用这样一个天真的度量，我们会失去大量信息，因为两和十之间的距离将等于两和一百万之间的距离，即1。

在机器学习中有一些流行的距离度量。我们首先介绍两种最常用的：

+   绝对值距离：<math alttext="StartAbsoluteValue y Subscript p r e d i c t Baseline minus y Subscript t r u e Baseline EndAbsoluteValue"><mrow><mrow><mo>|</mo></mrow> <msub><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>-</mo> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mrow><mo>|</mo></mrow></mrow></math>，源自微积分函数<math alttext="StartAbsoluteValue x EndAbsoluteValue"><mrow><mo>|</mo> <mi>x</mi> <mo>|</mo></mrow></math>。

+   平方距离：<math alttext="StartAbsoluteValue y Subscript p r e d i c t Baseline minus y Subscript t r u e Baseline EndAbsoluteValue squared"><mrow><mrow><mo>|</mo></mrow> <msub><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>-</mo> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <msup><mrow><mo>|</mo></mrow> <mn>2</mn></msup></mrow></math>，源自微积分函数<math alttext="StartAbsoluteValue x EndAbsoluteValue squared"><msup><mrow><mo>|</mo><mi>x</mi><mo>|</mo></mrow> <mn>2</mn></msup></math>（对于标量量来说，这与<math alttext="x squared"><msup><mi>x</mi> <mn>2</mn></msup></math>是相同的）。当然，这也会平方单位。

检查函数图像<math alttext="StartAbsoluteValue x EndAbsoluteValue"><mrow><mo>|</mo> <mi>x</mi> <mo>|</mo></mrow></math>和<math alttext="x squared"><msup><mi>x</mi> <mn>2</mn></msup></math>在[图3-4](#Fig_abs_x_square_x)中，我们注意到在点(0,0)处函数的平滑度有很大的差异。函数<math alttext="StartAbsoluteValue x EndAbsoluteValue"><mrow><mo>|</mo> <mi>x</mi> <mo>|</mo></mrow></math>在该点有一个拐角，使得它在x=0处不可微。这种在x=0处的<math alttext="StartAbsoluteValue x EndAbsoluteValue"><mrow><mo>|</mo> <mi>x</mi> <mo>|</mo></mrow></math>的*奇点*使得许多从业者（包括数学家！）不愿将这个函数或具有类似奇点的函数纳入他们的模型中。然而，让我们铭记以下事实：

*数学模型是灵活的*。当我们遇到障碍时，我们会深入挖掘，了解发生了什么，然后我们会克服障碍。

![275](assets/emai_0304.png)

###### 图3-4。左：图形<math alttext="StartAbsoluteValue x EndAbsoluteValue"><mrow><mo>|</mo> <mi>x</mi> <mo>|</mo></mrow></math>在<math alttext="x等于0"><mrow><mi>x</mi> <mo>=</mo> <mn>0</mn></mrow></math>处有一个转角，使得其在该点的导数未定义。右：图形<math alttext="StartAbsoluteValue x EndAbsoluteValue squared"><msup><mrow><mo>|</mo><mi>x</mi><mo>|</mo></mrow> <mn>2</mn></msup></math>在<math alttext="x等于0"><mrow><mi>x</mi> <mo>=</mo> <mn>0</mn></mrow></math>处平滑，因此其导数在那里没有问题。

除了函数<math alttext="StartAbsoluteValue x EndAbsoluteValue"><mrow><mo>|</mo> <mi>x</mi> <mo>|</mo></mrow></math>和<math alttext="StartAbsoluteValue x EndAbsoluteValue squared"><msup><mrow><mo>|</mo><mi>x</mi><mo>|</mo></mrow> <mn>2</mn></msup></math>的*规则性*之外（即它们在所有点是否都有导数），在决定是否将任一函数纳入我们的误差公式之前，我们还需要注意另一个问题：*如果一个数很大，那么它的平方就更大*。这个简单的观察意味着，如果我们决定使用真实值和预测值之间的平方距离来衡量误差，那么我们的方法将对数据中的异常值*更敏感*。一个混乱的异常值可能会使我们整个预测函数偏向它，因此远离数据中更普遍的模式。理想情况下，我们应该在数据准备步骤中处理异常值，并决定是否应该在将数据输入任何机器学习模型之前保留它们。

<math alttext="StartAbsoluteValue x EndAbsoluteValue"><mrow><mo>|</mo> <mi>x</mi> <mo>|</mo></mrow></math>（以及类似的分段线性函数）和<math alttext="x squared"><msup><mi>x</mi> <mn>2</mn></msup></math>（以及类似的非线性但可微函数）之间的最后一个区别是，<math alttext="StartAbsoluteValue x EndAbsoluteValue"><mrow><mo>|</mo> <mi>x</mi> <mo>|</mo></mrow></math>的导数非常简单：

1 如果x>0，-1 如果x<0（如果x=0，则未定义）。

在涉及数十亿次计算步骤的模型中，当使用<math alttext="StartAbsoluteValue x EndAbsoluteValue"><mrow><mo>|</mo> <mi>x</mi> <mo>|</mo></mrow></math>的导数时，*无需评估任何内容*，这一特性被证明非常有价值。通常情况下，既不是线性的也不是分段线性的函数的导数需要进行评估（因为它们的公式中不仅有常数，还有*x*），这在大数据环境中可能会很昂贵。

### 具有奇点的函数

一般来说，*可微*函数的图形没有尖点、转角、角或任何尖锐的地方。如果它们有这样的*奇点*，那么这些点处的函数就没有导数。原因是在尖锐的点上，你可以画出两条不同的切线，取决于你决定是在点的左边还是右边画切线（见[图3-5](#Fig_singularity_tangents)）。回想一下，函数在某一点的导数是函数图形在该点的切线的斜率。如果有两个*不同*的斜率，那么我们就无法定义该点的导数。

![275](assets/emai_0305.png)

###### 图3-5。在奇点处，导数不存在。在这些点上，切线可能有多个可能的斜率。

切线斜率的*不连续*在依赖于评估函数的导数的方法（如梯度下降法）中造成了问题。问题在于：

1.  如果你碰巧落在一个古怪的尖点上，那么方法就不知道该怎么办，因为那里没有定义的导数。有些人会为那一点分配一个导数值（称为*次梯度*或*次微分*）然后继续前进。实际上，我们会不幸地正好落在那一个可怕的点上的几率有多大呢？除非函数的景观看起来像阿尔卑斯山的崎岖地形（实际上很多函数确实如此），数值方法可能会设法避开它们。

1.  另一个问题是不稳定性。由于导数的值在函数的景观中跳跃得如此突然，使用这个导数的方法也会突然改变数值，如果你试图收敛到某个地方，就会产生不稳定性。想象一下你正在瑞士阿尔卑斯山徒步旅行[图3-6](#Fig_Swiss_Alps)（损失函数的景观），你的目的地是山谷下面那个漂亮的小镇（误差值最低的地方）。然后*突然*你被某个外星人带到了山的*另一边*，你再也看不到你的目的地了。事实上，现在你在山谷下看到的只有一些丑陋的灌木丛，还有一个非常狭窄的峡谷，如果你的方法把你带到那里，你就会被困住。你原来的目的地的收敛现在是不稳定的，甚至完全丢失了。

![275](assets/emai_0306.png)

###### 图3-6\. 瑞士阿尔卑斯山：优化类似于徒步穿越函数的景观。

尽管如此，具有这种奇点的函数在机器学习中经常被使用。我们将在一些神经网络训练函数的公式（修正线性单元函数-谁起这些名字？）、一些损失函数（绝对值距离）和一些正则化项（Lasso回归-这些也是谁起的名字？）中遇到它们。

### 对于线性回归，损失函数是均方误差

回到本节的主要目标：构建一个误差函数，也称为*损失函数*，它编码了我们的模型在进行预测时产生了多少误差，并且必须尽量小。

对于线性回归，我们使用*均方误差函数*。该函数对*m*个数据点的预测与真实值之间的平方距离误差进行平均（我们很快会提到包括哪些数据点）：

均方误差等于1/m（|y^predict^1 - y^true^1|^2 + |y^predict^2 - y^true^2|^2 + ... + |y^predict^m - y^true^m|^2）

让我们使用求和符号更紧凑地写出上面的表达式：

均方误差等于1/m的sigma-求和（从i=1到m）|y^predict^i - y^true^i|^2

现在我们养成了使用更紧凑的线性代数符号的好习惯。这种习惯在这个领域非常方便，因为我们不想在试图跟踪索引的同时淹没。索引可能潜入我们对理解一切的美好梦想中，并迅速将它们转变成非常可怕的噩梦。使用紧凑的线性代数符号的另一个非常重要的原因是，为机器学习模型构建的软件和硬件都针对矩阵和*张量*（想象一个由分层矩阵组成的对象，就像一个三维盒子而不是一个平面正方形）计算进行了优化。此外，美丽的数值线性代数领域已经解决了许多潜在问题，并为我们提供了快速执行各种矩阵计算的方法。

使用线性代数符号，我们可以将均方误差写成：

均方误差是预测基准减去真实值的修改后的y箭头预测基准减去修改后的y箭头真实基准的t次方左括号修改后的y箭头预测基准减去修改后的y箭头真实基准右括号等于1除以m平行于修改后的y箭头预测基准减去修改后的y箭头真实基准平行于句号

最后一个等式引入了向量的l平方范数，根据定义，这只是其分量的平方和的平方根。

主要观点：*我们构建的损失函数编码了训练过程中涉及的数据点的预测和地面真相之间的差异，用某种范数来衡量：作为距离的数学实体*。我们可以使用许多其他范数，但l平方范数非常受欢迎。

### 符号：本书中的向量始终是列向量

为了在整本书中符号一致，*所有*向量都是列向量。因此，如果一个向量修改后的v箭头有四个分量，符号修改后的v箭头代表的是修改后的v箭头代表的是4乘1矩阵第一行v1第二行v2第三行v3第四行v4。

向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>的转置始终是一个行向量。具有四个分量的上述向量的转置是<math alttext="ModifyingAbove v With right-arrow Superscript t Baseline equals Start 1 By 4 Matrix 1st Row 1st Column v 1 2nd Column v 2 3rd Column v 3 4th Column v 4 EndMatrix"><mrow><msup><mover accent="true"><mi>v</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mo>=</mo> <mfenced close=")" open="("><mtable><mtr><mtd><msub><mi>v</mi> <mn>1</mn></msub></mtd> <mtd><msub><mi>v</mi> <mn>2</mn></msub></mtd> <mtd><msub><mi>v</mi> <mn>3</mn></msub></mtd> <mtd><msub><mi>v</mi> <mn>4</mn></msub></mtd></mtr></mtable></mfenced></mrow></math>。

我们永远不会使用点积符号（也称为数量积，因为我们*乘以*两个向量，但我们的答案是一个标量）。而不是写两个向量的点积<math alttext="ModifyingAbove a With right-arrow period ModifyingAbove b With right-arrow"><mrow><mover accent="true"><mi>a</mi> <mo>→</mo></mover> <mo>.</mo> <mover accent="true"><mi>b</mi> <mo>→</mo></mover></mrow></math>，我们将写成<math alttext="ModifyingAbove a With right-arrow Superscript t Baseline ModifyingAbove b With right-arrow"><mrow><msup><mover accent="true"><mi>a</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>b</mi> <mo>→</mo></mover></mrow></math>，这是一样的，但本质上将列向量视为形状为：*向量的长度乘以1*的矩阵，其转置为形状为：*1乘以向量的长度*的矩阵。

假设现在<math alttext="ModifyingAbove a With right-arrow"><mover accent="true"><mi>a</mi> <mo>→</mo></mover></math>和<math alttext="ModifyingAbove b With right-arrow"><mover accent="true"><mi>b</mi> <mo>→</mo></mover></math>有四个分量，那么

<math alttext="dollar-sign ModifyingAbove a With right-arrow Superscript t Baseline ModifyingAbove b With right-arrow equals Start 1 By 4 Matrix 1st Row 1st Column a 1 2nd Column a 2 3rd Column a 3 4th Column a 4 EndMatrix Start 4 By 1 Matrix 1st Row  b 1 2nd Row  b 2 3rd Row  b 3 4th Row  b 4 EndMatrix equals a 1 b 1 plus a 2 b 2 plus a 3 b 3 plus a 4 b 4 equals sigma-summation Underscript i equals 1 Overscript 4 Endscripts a Subscript i Baseline b Subscript i Baseline period dollar-sign"><mrow><msup><mover accent="true"><mi>a</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>b</mi> <mo>→</mo></mover> <mo>=</mo> <mfenced close=")" open="("><mtable><mtr><mtd><msub><mi>a</mi> <mn>1</mn></msub></mtd> <mtd><msub><mi>a</mi> <mn>2</mn></msub></mtd> <mtd><msub><mi>a</mi> <mn>3</mn></msub></mtd> <mtd><msub><mi>a</mi> <mn>4</mn></msub></mtd></mtr></mtable></mfenced> <mfenced close=")" open="("><mtable><mtr><mtd><msub><mi>b</mi> <mn>1</mn></msub></mtd></mtr> <mtr><mtd><msub><mi>b</mi> <mn>2</mn></msub></mtd></mtr> <mtr><mtd><msub><mi>b</mi> <mn>3</mn></msub></mtd></mtr> <mtr><mtd><msub><mi>b</mi> <mn>4</mn></msub></mtd></mtr></mtable></mfenced> <mo>=</mo> <msub><mi>a</mi> <mn>1</mn></msub> <msub><mi>b</mi> <mn>1</mn></msub> <mo>+</mo> <msub><mi>a</mi> <mn>2</mn></msub> <msub><mi>b</mi> <mn>2</mn></msub> <mo>+</mo> <msub><mi>a</mi> <mn>3</mn></msub> <msub><mi>b</mi> <mn>3</mn></msub> <mo>+</mo> <msub><mi>a</mi> <mn>4</mn></msub> <msub><mi>b</mi> <mn>4</mn></msub> <mo>=</mo> <msubsup><mo>∑</mo> <mrow><mi>i</mi><mo>=</mo><mn>1</mn></mrow> <mn>4</mn></msubsup> <msub><mi>a</mi> <mi>i</mi></msub> <msub><mi>b</mi> <mi>i</mi></msub> <mo>.</mo></mrow></math>

此外，

<math alttext="dollar-sign parallel-to ModifyingAbove a With right-arrow parallel-to equals ModifyingAbove a With right-arrow Superscript t Baseline ModifyingAbove a With right-arrow equals a 1 squared plus a 2 squared plus a 3 squared plus a 4 squared period dollar-sign"><mrow><mrow><mo>∥</mo></mrow> <mover accent="true"><mi>a</mi> <mo>→</mo></mover> <msubsup><mrow><mo>∥</mo></mrow> <msup><mi>l</mi> <mn>2</mn></msup> <mn>2</mn></msubsup> <mo>=</mo> <msup><mover accent="true"><mi>a</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>a</mi> <mo>→</mo></mover> <mo>=</mo> <msubsup><mi>a</mi> <mn>1</mn> <mn>2</mn></msubsup> <mo>+</mo> <msubsup><mi>a</mi> <mn>2</mn> <mn>2</mn></msubsup> <mo>+</mo> <msubsup><mi>a</mi> <mn>3</mn> <mn>2</mn></msubsup> <mo>+</mo> <msubsup><mi>a</mi> <mn>4</mn> <mn>2</mn></msubsup> <mo>.</mo></mrow></math>

同样地，

<math alttext="dollar-sign parallel-to ModifyingAbove b With right-arrow parallel-to equals ModifyingAbove b With right-arrow Superscript t Baseline ModifyingAbove b With right-arrow equals b 1 squared plus b 2 squared plus b 3 squared plus b 4 squared period dollar-sign"><mrow><mrow><mo>∥</mo></mrow> <mover accent="true"><mi>b</mi> <mo>→</mo></mover> <msubsup><mrow><mo>∥</mo></mrow> <msup><mi>l</mi> <mn>2</mn></msup> <mn>2</mn></msubsup> <mo>=</mo> <msup><mover accent="true"><mi>b</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>b</mi> <mo>→</mo></mover> <mo>=</mo> <msubsup><mi>b</mi> <mn>1</mn> <mn>2</mn></msubsup> <mo>+</mo> <msubsup><mi>b</mi> <mn>2</mn> <mn>2</mn></msubsup> <mo>+</mo> <msubsup><mi>b</mi> <mn>3</mn> <mn>2</mn></msubsup> <mo>+</mo> <msubsup><mi>b</mi> <mn>4</mn> <mn>2</mn></msubsup> <mo>.</mo></mrow></math>

这样，我们在整个过程中都使用矩阵表示，并且只在字母上方加上箭头，以表明我们正在处理一个列向量。

### 训练、验证和测试子集

我们在损失函数中包含哪些数据点？我们包括整个数据集，其中的一些小批次，甚至只有一个点吗？我们是针对*训练子集*、*验证子集*还是*测试子集*来测量均方误差？这些子集到底是什么？

实际上，我们将数据集分成三个子集：

1.  训练子集：这是我们用来拟合训练函数的数据子集。这意味着这个子集中的数据点被纳入我们的损失函数中（通过将它们的特征值和标签插入到损失函数的<math alttext="y Subscript p r e d i c t"><msub><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub></math>和<math alttext="y Subscript t r u e"><msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></math>中）。

1.  验证子集：这个子集中的数据点被多种方式使用：

    +   常见的描述是，我们使用这个子集来*调整机器学习模型的超参数*。 超参数是机器学习模型中*不是*我们试图解决的训练函数的<math alttext="omega"><mi>ω</mi></math>的任何参数。 在机器学习中，有许多这样的参数，它们的值会影响模型的结果和性能。 超参数的示例包括（您现在不必知道这些是什么）：出现在梯度下降方法中的学习率，决定支持向量机方法中边缘宽度的超参数，原始数据分成训练、验证和测试子集的百分比，随机批量梯度下降时的批量大小，权重衰减超参数，例如Ridge、LASSO和Elastic Net回归中使用的超参数，带有动量方法的超参数，例如带有动量的梯度下降和ADAM（这些方法加速了方法向最小值的收敛，这些项乘以需要在测试和部署之前进行调整的超参数），神经网络的架构，例如层数、每层的宽度，*等*，以及优化过程中的*epochs*的数量（优化器已经看到整个训练子集的传递次数）。

    +   验证子集还帮助我们知道何时*停止*优化*之前*过度拟合我们的训练子集。

    +   它还作为一个测试集，用来比较*不同*机器学习模型在同一数据集上的性能，例如，比较线性回归模型、随机森林和神经网络的性能。

1.  *测试*子集：在决定使用最佳模型（或对多个模型的结果进行平均或聚合）并训练模型之后，我们使用这个未触及的数据子集作为模型在部署到真实世界之前的最后阶段测试。由于模型之前没有见过这个子集中的任何数据点（这意味着它没有在优化过程中包含其中任何数据点），因此它可以被视为最接近真实世界情况的类比。这使我们能够在开始将其应用于全新的真实世界数据之前评估我们模型的性能。

### 总结

在继续之前，让我们简要回顾一下。

+   我们当前的机器学习模型称为*线性回归*。

+   我们的训练函数是线性的，公式为：

<math alttext="dollar-sign y equals omega 0 plus omega 1 x 1 plus omega 2 x 2 plus omega 3 x 3 plus omega 4 x 4 plus omega 5 x 5 period dollar-sign"><mrow><mi>y</mi> <mo>=</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msub><mi>x</mi> <mn>1</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>2</mn></msub> <msub><mi>x</mi> <mn>2</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>3</mn></msub> <msub><mi>x</mi> <mn>3</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>4</mn></msub> <msub><mi>x</mi> <mn>4</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>5</mn></msub> <msub><mi>x</mi> <mn>5</mn></msub> <mo>.</mo></mrow></math>

<math alttext="x"><mi>x</mi></math>是特征，<math alttext="omega"><mi>ω</mi></math>是未知的权重或参数。

+   如果我们将特定数据点的特征值（例如第十个数据点）代入训练函数的公式中，我们就可以得到我们模型对该点的预测：

<math alttext="dollar-sign y Subscript p r e d i c t Superscript 10 Baseline equals omega 0 plus omega 1 x 1 Superscript 10 Baseline plus omega 2 x 2 Superscript 10 Baseline plus omega 3 x 3 Superscript 10 Baseline plus omega 4 x 4 Superscript 10 Baseline plus omega 5 x 5 Superscript 10 Baseline period dollar-sign"><mrow><msubsup><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow> <mn>10</mn></msubsup> <mo>=</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msubsup><mi>x</mi> <mn>1</mn> <mn>10</mn></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>2</mn></msub> <msubsup><mi>x</mi> <mn>2</mn> <mn>10</mn></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>3</mn></msub> <msubsup><mi>x</mi> <mn>3</mn> <mn>10</mn></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>4</mn></msub> <msubsup><mi>x</mi> <mn>4</mn> <mn>10</mn></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>5</mn></msub> <msubsup><mi>x</mi> <mn>5</mn> <mn>10</mn></msubsup> <mo>.</mo></mrow></math>

上标10表示这些值对应于第十个数据点。

+   我们的损失函数是带有以下公式的均方误差函数：

<math alttext="dollar-sign Mean Squared Error StartFraction 1 Over m EndFraction left-parenthesis ModifyingAbove y With right-arrow Subscript p r e d i c t Baseline minus ModifyingAbove y With right-arrow Subscript t r u e Baseline right-parenthesis Superscript t Baseline left-parenthesis ModifyingAbove y With right-arrow Subscript p r e d i c t Baseline minus ModifyingAbove y With right-arrow Subscript t r u e Baseline right-parenthesis equals StartFraction 1 Over m EndFraction parallel-to ModifyingAbove y With right-arrow Subscript p r e d i c t Baseline minus ModifyingAbove y With right-arrow Subscript t r u e Baseline parallel-to period dollar-sign"><mrow><mtext>Mean</mtext> <mtext>Squared</mtext> <mtext>Error</mtext> <mo>=</mo> <mfrac><mn>1</mn> <mi>m</mi></mfrac> <msup><mrow><mo>(</mo><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>-</mo><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>)</mo></mrow> <mi>t</mi></msup> <mrow><mo>(</mo> <msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>-</mo> <msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>)</mo></mrow> <mo>=</mo> <mfrac><mn>1</mn> <mi>m</mi></mfrac> <msubsup><mrow><mo>∥</mo><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>-</mo><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>∥</mo></mrow> <mrow><msup><mi>l</mi> <mn>2</mn></msup></mrow> <mn>2</mn></msubsup> <mo>.</mo></mrow></math>

+   我们想要找到最小化这个损失函数的<math alttext="omega"><mi>ω</mi></math>的值。因此，下一步必须是解决一个最小化（优化）问题。

为了让我们的优化工作更加轻松，我们将再次使用线性代数（向量和矩阵）的便捷符号。这使我们能够将整个训练数据子集作为损失函数公式中的矩阵，并立即在训练子集上进行计算，而不是在每个数据点上分别计算。这种小小的符号操作可以避免我们在非常大的数据集上出现许多难以跟踪的组件的错误、痛苦和繁琐计算。

首先，写出我们模型对训练子集的每个数据点的预测：

<math alttext="dollar-sign StartLayout 1st Row 1st Column y Subscript p r e d i c t Superscript 1 2nd Column equals 1 omega 0 plus omega 1 x 1 Superscript 1 Baseline plus omega 2 x 2 Superscript 1 Baseline plus omega 3 x 3 Superscript 1 Baseline plus omega 4 x 4 Superscript 1 Baseline plus omega 5 x 5 Superscript 1 Baseline 2nd Row 1st Column y Subscript p r e d i c t Superscript 2 2nd Column equals 1 omega 0 plus omega 1 x 1 squared plus omega 2 x 2 squared plus omega 3 x 3 squared plus omega 4 x 4 squared plus omega 5 x 5 squared 3rd Row 1st Column  ellipsis 4th Row 1st Column y Subscript p r e d i c t Superscript m 2nd Column equals 1 omega 0 plus omega 1 x 1 Superscript m Baseline plus omega 2 x 2 Superscript m Baseline plus omega 3 x 3 Superscript m Baseline plus omega 4 x 4 Superscript m Baseline plus omega 5 x 5 Superscript m EndLayout dollar-sign"><mtable displaystyle="true"><mtr><mtd columnalign="right"><msubsup><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow> <mn>1</mn></msubsup></mtd> <mtd columnalign="left"><mrow><mo>=</mo> <mn>1</mn> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msubsup><mi>x</mi> <mn>1</mn> <mn>1</mn></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>2</mn></msub> <msubsup><mi>x</mi> <mn>2</mn> <mn>1</mn></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>3</mn></sub> <msubsup><mi>x</mi> <mn>3</mn> <mn>1</mn></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>4</mn></msub> <msubsup><mi>x</mi> <mn>4</mn> <mn>1</mn></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>5</mn></msub> <msubsup><mi>x</mi> <mn>5</mn> <mn>1</mn></msubsup></mrow></mtd></mtr> <mtr><mtd columnalign="right"><msubsup><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow> <mn>2</mn></msubsup></mtd> <mtd columnalign="left"><mrow><mo>=</mo> <mn>1</mn> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msubsup><mi>x</mi> <mn>1</mn> <mn>2</mn></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>2</mn></msub> <msubsup><mi>x</mi> <mn>2</mn> <mn>2</mn></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>3</mn></msub> <msubsup><mi>x</mi> <mn>3</mn> <mn>2</mn></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>4</mn></msub> <msubsup><mi>x</mi> <mn>4</mn> <mn>2</mn></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>5</mn></msub> <msubsup><mi>x</mi> <mn>5</mn> <mn>2</mn></msubsup></mrow></mtd></mtr> <mtr><mtd columnalign="right"><mo>⋮</mo></mtd></mtr> <mtr><mtd columnalign="right"><msubsup><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow> <mi>m</mi></msubsup></mtd> <mtd columnalign="left"><mrow><mo>=</mo> <mn>1</mn> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msubsup><mi>x</mi> <mn>1</mn> <mi>m</mi></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>2</mn></msub> <msubsup><mi>x</mi> <mn>2</mn> <mi>m</mi></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>3</mn></msub> <msubsup><mi>x</mi> <mn>3</mn> <mi>m</mi></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>4</mn></msub> <msubsup><mi>x</mi> <mn>4</mn> <mi>m</mi></msubsup> <mo>+</mo> <msub><mi>ω</mi> <mn>5</mn></msub> <msubsup><mi>x</mi> <mn>5</mn> <mi>m</mi></msubsup></mrow></mtd></mtr></mtable></math>

我们可以轻松地将上述系统安排如下：

<math alttext="美元符号 开始 4x1 矩阵 第一行  y 下标 p r e d i c t 上标 1 2nd Row  y 下标 p r e d i c t 上标 2 3rd Row   省略 4th Row  y 下标 p r e d i c t 上标 m 等于 开始 4x1 矩阵 第一行  1 2nd Row  1 3rd Row   省略 4th Row  1 EndMatrix omega 0 加 开始 4x1 矩阵 第一行  x 1 上标 1 2nd Row  x 1 平方 3rd Row   省略 4th Row  x 1 上标 m EndMatrix omega 1 加 开始 4x1 矩阵 第一行  x 1 上标 1 2nd Row  x 2 平方 3rd Row   省略 4th Row  x 2 上标 m EndMatrix omega 2 加 开始 4x1 矩阵 第一行  x 3 上标 1 2nd Row  x 3 平方 3rd Row   省略 4th Row  x 3 上标 m EndMatrix omega 3 加 开始 4x1 矩阵 第一行  x 4 上标 1 2nd Row  x 4 平方 3rd Row   省略 4th Row  x 4 上标 m EndMatrix omega 4 加 开始 4x1 矩阵 第一行  x 5 上标 1 2nd Row  x 5 平方 3rd Row   省略 4th Row  x 5 上标 m EndMatrix omega 5 逗号 美元符号"><mrow><mfenced close=")" open="("><mtable><mtr><mtd><msubsup><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow> <mn>1</mn></msubsup></mtd></mtr> <mtr><mtd><msubsup><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow> <mn>2</mn></msubsup></mtd></mtr> <mtr><mtd><mo>⋮</mo></mtd></mtr> <mtr><mtd><msubsup><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow> <mi>m</mi></msubsup></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced close=")" open="("><mtable><mtr><mtd><mn>1</mn></mtd></mtr> <mtr><mtd><mn>1</mn></mtd></mtr> <mtr><mtd><mo>⋮</mo></mtd></mtr> <mtr><mtd><mn>1</mn></mtd></mtr></mtable></mfenced> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <mfenced close=")" open="("><mtable><mtr><mtd><msubsup><mi>x</mi> <mn>1</mn> <mn>1</mn></msubsup></mtd></mtr> <mtr><mtd><msubsup><mi>x</mi> <mn>1</mn> <mn>2</mn></msubsup></mtd></mtr> <mtr><mtd><mo>⋮</mo></mtd></mtr> <mtr><mtd><msubsup><mi>x</mi> <mn>1</mn> <mi>m</mi></msubsup></mtd></mtr></mtable></mfenced> <msub><mi>ω</mi> <mn>1</mn></msub> <mo>+</mo> <mfenced close=")" open="("><mtable><mtr><mtd><msubsup><mi>x</mi> <mn>1</mn> <mn>1</mn></msubsup></mtd></mtr> <mtr><mtd><msubsup><mi>x</mi> <mn>2</mn> <mn>2</mn></msubsup></mtd></mtr> <mtr><mtd><mo>⋮</mo></mtd></mtr> <mtr><mtd><msubsup><mi>x</mi> <mn>2</mn> <mi>m</mi></msubsup></mtd></mtr></mtable></mfenced> <msub><mi>ω</mi> <mn>2</mn></msub> <mo>+</mo> <mfenced close=")" open="("><mtable><mtr><mtd><msubsup><mi>x</mi> <mn>3</mn> <mn>1</mn></msubsup></mtd></mtr> <mtr><mtd><msubsup><mi>x</mi> <mn>3</mn> <mn>2</mn></msubsup></mtd></mtr> <mtr><mtd><mo>⋮</mo></mtd></mtr> <mtr><mtd><msubsup><mi>x</mi> <mn>3</mn> <mi>m</mi></msubsup></mtd></mtr></mtable></mfenced> <msub><mi>ω</mi> <mn>3</mn></msub> <mo>+</mo> <mfenced close=")" open="("><mtable><mtr><mtd><msubsup><mi>x</mi> <mn>4</mn> <mn>1</mn></msubsup></mtd></mtr> <mtr><mtd><msubsup><mi>x</mi> <mn>4</mn> <mn>2</mn></msubsup></mtd></mtr> <mtr><mtd><mo>⋮</mo></mtd></mtr> <mtr><mtd><msubsup><mi>x</mi> <mn>4</mn> <mi>m</mi></msubsup></mtd></mtr></mtable></mfenced> <msub><mi>ω</mi> <mn>4</mn></msub> <mo>+</mo> <mfenced close=")" open="("><mtable><mtr><mtd><msubsup><mi>x</mi> <mn>5</mn> <mn>1</mn></msubsup></mtd></mtr> <mtr><mtd><msubsup><mi>x</mi> <mn>5</mn> <mn>2</mn></msubsup></mtd></mtr> <mtr><mtd><mo>⋮</mo></mtd></mtr> <mtr><mtd><msubsup><mi>x</mi> <mn>5</mn> <mi>m</mi></msubsup></mtd></mtr></mtable></mfenced> <msub><mi>ω</mi> <mn>5</mn></msub> <mo>,</mo></mrow></math>

或者更好：

美元符号开始4乘1矩阵第一行y下标predict上标1第二行y下标predict上标2第三行省略号第四行y下标predict上标m等于4乘6矩阵第一行第一列1第二列x1上标1第三列x2上标1第四列x3上标1第五列x4上标1第六列x5上标1第二行第一列1第二列x1平方第三列x2平方第四列x3平方第五列x4平方第六列x5平方第三行第一列省略号第四行第一列1第二列x1上标m第三列x2上标m第四列x3上标m第五列x4上标m第六列x5上标m结束矩阵开始6乘1矩阵第一行omega0第二行omega1第三行omega2第四行omega3第五行omega4第六行omega5结束矩阵。

上述方程左侧的向量是<math alttext="ModifyingAbove y With right-arrow Subscript p r e d i c t"><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub></math>，右侧的矩阵是训练子集*X*与包含1的向量增广，右侧的最后一个向量包含了所有未知的权重。将这个向量称为<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>，然后用训练子集和<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>简洁地写成<math alttext="ModifyingAbove y With right-arrow Subscript p r e d i c t"><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub></math>：

<math alttext="dollar-sign ModifyingAbove y With right-arrow Subscript p r e d i c t Baseline equals upper X ModifyingAbove omega With right-arrow period dollar-sign"><mrow><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>=</mo> <mi>X</mi> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>.</mo></mrow></math>

现在我们之前写过的均方误差损失函数的公式如下：

<math alttext="dollar-sign Mean Squared Error StartFraction 1 Over m EndFraction left-parenthesis ModifyingAbove y With right-arrow Subscript p r e d i c t Baseline minus ModifyingAbove y With right-arrow Subscript t r u e Baseline right-parenthesis Superscript t Baseline left-parenthesis ModifyingAbove y With right-arrow Subscript p r e d i c t Baseline minus ModifyingAbove y With right-arrow Subscript t r u e Baseline right-parenthesis equals StartFraction 1 Over m EndFraction parallel-to ModifyingAbove y With right-arrow Subscript p r e d i c t Baseline minus ModifyingAbove y With right-arrow Subscript t r u e Baseline parallel-to dollar-sign"><mrow><mtext>Mean</mtext> <mtext>Squared</mtext> <mtext>Error</mtext> <mo>=</mo> <mfrac><mn>1</mn> <mi>m</mi></mfrac> <msup><mrow><mo>(</mo><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>-</mo><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>)</mo></mrow> <mi>t</mi></msup> <mrow><mo>(</mo> <msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>-</mo> <msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>)</mo></mrow> <mo>=</mo> <mfrac><mn>1</mn> <mi>m</mi></mfrac> <msubsup><mrow><mo>∥</mo><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>-</mo><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>∥</mo></mrow> <mrow><msup><mi>l</mi> <mn>2</mn></msup></mrow> <mn>2</mn></msubsup></mrow></math>

变成：

<math alttext="dollar-sign Mean Squared Error StartFraction 1 Over m EndFraction left-parenthesis upper X ModifyingAbove omega With right-arrow minus ModifyingAbove y With right-arrow Subscript t r u e Baseline right-parenthesis Superscript t Baseline left-parenthesis upper X ModifyingAbove omega With right-arrow minus ModifyingAbove y With right-arrow Subscript t r u e Baseline right-parenthesis equals StartFraction 1 Over m EndFraction parallel-to upper X ModifyingAbove omega With right-arrow minus ModifyingAbove y With right-arrow Subscript t r u e Baseline parallel-to period dollar-sign"><mrow><mtext>均方误差</mtext> <mo>=</mo> <mfrac><mn>1</mn> <mi>m</mi></mfrac> <msup><mfenced close=")" open="(" separators=""><mi>X</mi><mover accent="true"><mi>ω</mi> <mo>→</mo></mover><mo>-</mo><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></mfenced> <mi>t</mi></msup> <mfenced close=")" open="(" separators=""><mi>X</mi> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>-</mo> <msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></mfenced> <mo>=</mo> <mfrac><mn>1</mn> <mi>m</mi></mfrac> <msubsup><mrow><mo>∥</mo><mi>X</mi><mover accent="true"><mi>ω</mi> <mo>→</mo></mover><mo>-</mo><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>∥</mo></mrow> <mrow><msup><mi>l</mi> <mn>2</mn></msup></mrow> <mn>2</mn></msubsup> <mo>.</mo></mrow></math>

我们现在准备找到最小化精心编写的损失函数的<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>。为此，我们必须访问优化的丰富而美丽的数学领域。

### 当训练数据具有高度相关的特征时

检查训练矩阵（附加了一个包含1的向量）

<math alttext="dollar-sign upper X equals Start 4 By 6 Matrix 1st Row 1st Column 1 2nd Column x 1 Superscript 1 Baseline 3rd Column x 2 Superscript 1 Baseline 4th Column x 3 Superscript 1 Baseline 5th Column x 4 Superscript 1 Baseline 6th Column x 5 Superscript 1 Baseline 2nd Row 1st Column 1 2nd Column x 1 squared 3rd Column x 2 squared 4th Column x 3 squared 5th Column x 4 squared 6th Column x 5 squared 3rd Row 1st Column  ellipsis 4th Row 1st Column 1 2nd Column x 1 Superscript m Baseline 3rd Column x 2 Superscript m Baseline 4th Column x 3 Superscript m Baseline 5th Column x 4 Superscript m Baseline 6th Column x 5 Superscript m EndMatrix dollar-sign"><mrow><mi>X</mi> <mo>=</mo> <mfenced close=")" open="("><mtable><mtr><mtd><mn>1</mn></mtd> <mtd><msubsup><mi>x</mi> <mn>1</mn> <mn>1</mn></msubsup></mtd> <mtd><msubsup><mi>x</mi> <mn>2</mn> <mn>1</mn></msubsup></mtd> <mtd><msubsup><mi>x</mi> <mn>3</mn> <mn>1</mn></msubsup></mtd> <mtd><msubsup><mi>x</mi> <mn>4</mn> <mn>1</mn></msubsup></mtd> <mtd><msubsup><mi>x</mi> <mn>5</mn> <mn>1</mn></msubsup></mtd></mtr> <mtr><mtd><mn>1</mn></mtd> <mtd><msubsup><mi>x</mi> <mn>1</mn> <mn>2</mn></msubsup></mtd> <mtd><msubsup><mi>x</mi> <mn>2</mn> <mn>2</mn></msubsup></mtd> <mtd><msubsup><mi>x</mi> <mn>3</mn> <mn>2</mn></msubsup></mtd> <mtd><msubsup><mi>x</mi> <mn>4</mn> <mn>2</mn></msubsup></mtd> <mtd><msubsup><mi>x</mi> <mn>5</mn> <mn>2</mn></msubsup></mtd></mtr> <mtr><mtd><mo>⋮</mo></mtd></mtr> <mtr><mtd><mn>1</mn></mtd> <mtd><msubsup><mi>x</mi> <mn>1</mn> <mi>m</mi></msubsup></mtd> <mtd><msubsup><mi>x</mi> <mn>2</mn> <mi>m</mi></msubsup></mtd> <mtd><msubsup><mi>x</mi> <mn>3</mn> <mi>m</mi></msubsup></mtd> <mtd><msubsup><mi>x</mi> <mn>4</mn> <mi>m</mi></msubsup></mtd> <mtd><msubsup><mi>x</mi> <mn>5</mn> <mi>m</mi></msubsup></mtd></mtr></mtable></mfenced></mrow></math>

出现在向量<math alttext="ModifyingAbove y With right-arrow Subscript p r e d i c t Baseline equals upper X ModifyingAbove omega With right-arrow"><mrow><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>=</mo> <mi>X</mi> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover></mrow></math>中，均方误差损失函数的公式，以及后来确定未知<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>（也称为*正规方程*）的公式。

<math alttext="dollar-sign ModifyingAbove omega With right-arrow equals left-parenthesis upper X Superscript t Baseline upper X right-parenthesis Superscript negative 1 Baseline upper X Superscript t Baseline ModifyingAbove y With right-arrow Subscript t r u e Baseline comma dollar-sign"><mrow><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>=</mo> <msup><mrow><mo>(</mo><msup><mi>X</mi> <mi>t</mi></msup> <mi>X</mi><mo>)</mo></mrow> <mrow><mo>-</mo><mn>1</mn></row></msup> <msup><mi>X</mi> <mi>t</mi></msup> <msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>,</mo></mrow></math>

我们可以看到，如果数据的两个或更多特征（*x*列）高度相关，我们的模型可能会出现问题：这意味着特征之间存在强烈的线性关系，因此其中一个特征可以通过其他特征的线性组合来确定（或几乎确定）。因此，相应的特征列是*不线性独立*的（或接近不是线性独立的）。对于矩阵来说，这是一个问题，因为它表明矩阵要么不能被反转，要么*病态*。病态的矩阵在计算中产生大的不稳定性，因为训练数据的轻微变化（必须假设）会导致模型参数的大变化，从而使其预测不可靠。

我们在计算中希望得到条件良好的矩阵，因此必须消除病态条件的来源。当我们有高度相关的特征时，一个可能的途径是只在我们的模型中包含其中一个，因为其他特征并不添加太多信息。另一个解决方案是应用主成分分析等降维技术，我们将在[第11章](ch11.xhtml#ch11)中遇到。鱼市数据集具有高度相关的特征，附带的Jupyter Notebook解决了这些问题。

也就是说，重要的是要注意，一些机器学习模型，如决策树和随机森林（下文讨论），不受相关特征的影响，而其他一些模型，如当前的线性回归模型，以及接下来的逻辑回归和支持向量机模型受到了负面影响。至于神经网络模型，即使它们可以在训练过程中*学习*数据特征中的相关性，但当这些冗余在时间之前得到处理时，它们的表现更好，除了节省计算成本和时间。

## 优化

优化意味着寻找最佳、最大、最小或极端解决方案。

我们编写了一个线性训练函数

<math alttext="dollar-sign y equals omega 0 plus omega 1 x 1 plus omega 2 x 2 plus omega 3 x 3 plus omega 4 x 4 plus omega 5 x 5 period dollar-sign"><mrow><mi>y</mi> <mo>=</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msub><mi>x</mi> <mn>1</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>2</mn></msub> <msub><mi>x</mi> <mn>2</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>3</mn></msub> <msub><mi>x</mi> <mn>3</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>4</mn></msub> <msub><mi>x</mi> <mn>4</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>5</mn></msub> <msub><mi>x</mi> <mn>5</mn></msub> <mo>.</mo></mrow></math>

我们留下了它的六个参数<math alttext="omega 0"><msub><mi>ω</mi> <mn>0</mn></msub></math>，<math alttext="omega 1"><msub><mi>ω</mi> <mn>1</mn></msub></math>，<math alttext="omega 2"><msub><mi>ω</mi> <mn>2</mn></msub></math>，<math alttext="omega 3"><msub><mi>ω</mi> <mn>3</mn></msub></math>，<math alttext="omega 4"><msub><mi>ω</mi> <mn>4</mn></msub></math>和<math alttext="omega 5"><msub><mi>ω</mi> <mn>5</mn></msub></math>的值未知。目标是找到使我们的训练函数*最适合训练数据子集*的值，其中*最适合*一词是使用损失函数量化的。该函数提供了模型训练函数所做预测与真实情况的偏差程度的度量。我们希望这个损失函数很小，因此我们解决了一个最小化问题。

我们不会坐在那里尝试每个可能的<math alttext="omega"><mi>ω</mi></math>值，直到找到使损失最小的组合。即使我们这样做了，我们也不会知道何时停止，因为我们不知道是否还有其他*更好*的值。我们必须对损失函数的地形有先验知识，并利用其数学特性。类比是盲目徒步穿越瑞士阿尔卑斯山*vs.*带着详细地图徒步穿越（[图3-7](#Fig_Swiss_Alps2)显示了瑞士阿尔卑斯山的崎岖地形）。我们不是盲目地搜索损失函数的地形以寻找最小值，而是利用*优化*领域。优化是数学的一个美丽分支，提供了各种方法来高效地搜索函数的最优解及其对应的最优值。

![275](assets/emai_0307.png)

###### 图3-7. 瑞士阿尔卑斯山：优化类似于徒步穿越函数的地形。目的地是最低谷的底部（最小化）或最高峰的顶部（最大化）。我们需要两样东西：最小化或最大化点的坐标，以及这些点的地形高度。

本章和接下来几章的优化问题如下：

<math alttext="dollar-sign min Underscript ModifyingAbove omega With right-arrow Endscripts Loss Function period dollar-sign"><mrow><msub><mo form="prefix" movablelimits="true">min</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover></msub> <mtext>Loss</mtext> <mtext>Function</mtext> <mo>.</mo></mrow></math>

对于当前的线性回归模型，这是：

<math alttext="dollar-sign min Underscript ModifyingAbove omega With right-arrow Endscripts StartFraction 1 Over m EndFraction left-parenthesis upper X ModifyingAbove omega With right-arrow minus ModifyingAbove y With right-arrow Subscript t r u e Baseline right-parenthesis Superscript t Baseline left-parenthesis upper X ModifyingAbove omega With right-arrow minus ModifyingAbove y With right-arrow Subscript t r u e Baseline right-parenthesis equals min Underscript ModifyingAbove omega With right-arrow Endscripts StartFraction 1 Over m EndFraction parallel-to upper X ModifyingAbove omega With right-arrow minus ModifyingAbove y With right-arrow Subscript t r u e Baseline parallel-to period dollar-sign"><mrow><msub><mo form="prefix" movablelimits="true">min</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover></msub> <mfrac><mn>1</mn> <mi>m</mi></mfrac> <msup><mfenced close=")" open="(" separators=""><mi>X</mi><mover accent="true"><mi>ω</mi> <mo>→</mo></mover><mo>-</mo><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></mfenced> <mi>t</mi></msup> <mfenced close=")" open="(" separators=""><mi>X</mi> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>-</mo> <msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></mfenced> <mo>=</mo> <msub><mo form="prefix" movablelimits="true">min</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover></msub> <mfrac><mn>1</mn> <mi>m</mi></mfrac> <msubsup><mrow><mo>∥</mo><mi>X</mi><mover accent="true"><mi>ω</mi> <mo>→</mo></mover><mo>-</mo><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>∥</mo></mrow> <mrow><msup><mi>l</mi> <mn>2</mn></msup></mrow> <mn>2</mn></msubsup> <mo>.</mo></mrow></math>

当我们进行数学运算时，我们绝不能忘记我们知道什么，我们正在寻找什么。否则我们可能会陷入循环逻辑的陷阱。在上述公式中，我们知道：

+   *m*（训练子集中的实例数），

+   *X*（训练子集增加了一个全为1的向量），

+   在训练子集对应的标签向量中的最小损失函数值。

我们正在寻找：

+   最小化的<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>：我们必须找到它。

+   在最小化的损失函数值处的<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>。

### 凸景观 *vs.* 非凸景观

*最容易处理*和最容易解决的方程是线性的。不幸的是，我们处理的大多数函数（和方程）都是非线性的。与此同时，这并不太不幸，因为线性的生活是平淡、无聊、乏味和无趣的。当我们手头的函数是完全非线性的时候，有时我们会在我们关心的某些点附近*线性化*它。这里的想法是，即使函数的整体图像可能是非线性的，我们可能能够在我们关注的局部用线性函数来近似它。换句话说，在一个非常小的邻域内，非线性函数可能看起来和行为上是线性的，尽管该邻域可能是无限小的。打个比方，想想地球从我们自己的地方看起来是平的，从高处我们只能看到它的非线性形状。当我们想要在某一点附近线性化一个函数时，我们通过计算函数对所有变量的一个导数来近似它，因为这给我们提供了近似平坦空间的斜率（它测量了近似平坦空间的倾斜度）。

令人沮丧的消息是，在一个点附近进行线性化可能不够，我们可能希望在多个位置使用线性近似。幸运的是，这是可行的，因为我们在计算上所要做的就是在几个点上评估一个导数。这将我们带到了（在线性函数之后）*最容易处理的函数*：分段线性函数，它们在结构上是线性的，或者在孤立点或位置上是线性的。*线性规划*领域处理这样的函数，其中要优化的函数是线性的，而优化发生的域的边界是分段线性的（它们是半空间的交集）。

当我们的目标是优化时，最好处理的函数要么是线性的（线性规划领域帮助我们），要么是凸的（我们不用担心陷入局部最小值，而且我们有很好的不等式帮助我们进行分析）。

在机器学习中要记住的一个重要类型的函数是两个或多个凸函数的最大值。这些函数总是凸的。线性函数是平的，因此它们同时是凸的和凹的。这很有用，因为有些函数被定义为线性函数的最大值：这些函数不一定是线性的（它们是分段线性的），但是保证是凸的。也就是说，即使我们在取线性函数的最大值时失去了线性性，但我们得到了凸性的补偿。

在神经网络中作为非线性激活函数使用的修正线性单元函数（ReLU）是一个被定义为两个线性函数的最大值的例子：<math alttext="upper R e upper L upper U left-parenthesis x right-parenthesis equals m a x left-parenthesis 0 comma x right-parenthesis"><mrow><mi>R</mi> <mi>e</mi> <mi>L</mi> <mi>U</mi> <mo>(</mo> <mi>x</mi> <mo>)</mo> <mo>=</mo> <mi>m</mi> <mi>a</mi> <mi>x</mi> <mo>(</mo> <mn>0</mn> <mo>,</mo> <mi>x</mi> <mo>)</mo></mrow></math>。另一个例子是支持向量机中使用的铰链损失函数：<math alttext="upper H left-parenthesis x right-parenthesis equals m a x left-parenthesis 0 comma 1 minus t x right-parenthesis"><mrow><mi>H</mi> <mo>(</mo> <mi>x</mi> <mo>)</mo> <mo>=</mo> <mi>m</mi> <mi>a</mi> <mi>x</mi> <mo>(</mo> <mn>0</mn> <mo>,</mo> <mn>1</mn> <mo>-</mo> <mi>t</mi> <mi>x</mi> <mo>)</mo></mrow></math>，其中*t*是1或-1。

请注意，一组凸函数的最小值不能保证是凸的，它可能有双井。然而，它们的最大值肯定是凸的。

线性和凸性之间还有另一个关系：如果我们有一个凸函数（非线性，因为线性将是平凡的），那么所有保持在函数下方的线性函数的最大值恰好等于它。换句话说，凸性取代了线性性，意思是当线性性不可用时，但凸性可用时，我们可以用所有图形位于函数图形下方的线性函数的最大值来替换我们的凸函数（见[图3-8](#Fig_convex_above_tangents)）。请记住，凸函数的图形位于任意点的切线图形上方，而切线是线性的。这为我们提供了一条直接利用线性函数简单性的路径，当我们有凸函数时。当我们考虑*所有*切线的最大值时，我们有相等，当我们考虑少数点的切线的最大值时，我们只有近似。

![300](assets/emai_0308.png)

###### 图3-8。凸函数等于其所有切线的最大值。

[图3-9](#Fig_convex_landscape)和[图3-10](#Fig_nonconvex_landscape)分别展示了非线性凸函数和非凸函数的一般景观。总的来说，凸函数的景观对于最小化问题是有利的。我们不必担心被困在局部最小值，因为对于凸函数来说，任何局部最小值也是全局最小值。非凸函数的景观有峰值、谷底和鞍点。在这样的景观上进行最小化问题会有被困在局部最小值而无法找到全局最小值的风险。

![280](assets/emai_0309.png)

###### 图3-9。凸函数的景观对于最小化问题是有利的。我们不必担心被困在局部最小值，因为对于凸函数来说，任何局部最小值也是全局最小值。

![280](assets/emai_0310.png)

###### 图3-10。非凸函数的景观有峰值、谷底和鞍点。在这样的景观上进行最小化问题会有被困在局部最小值而无法找到全局最小值的风险。

最后，请确保你知道凸函数、凸集和凸优化问题之间的区别，凸优化问题是在凸集上优化凸函数。

### 我们如何找到函数的最小值？

一般来说，有两种方法来找到函数的最小值（和/或最大值）。通常的权衡是：

1.  只计算一个导数并缓慢收敛到最小值（尽管有加速方法可以加快收敛速度）。这些被称为*梯度*方法。梯度是多个变量的函数的一个导数。例如，我们的损失函数是多个<math alttext="omega"><mi>ω</mi></math>的函数（或一个向量<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>）。

1.  计算两个导数（在计算上更昂贵，尤其是当我们有成千上万个参数时，这是一个大的缺点），并更快地收敛到最小值。可以通过近似第二导数来节省一些计算成本，而不是精确计算它。第二导数方法被称为*牛顿*方法。*Hessian*（二阶导数的矩阵）或Hessian的近似出现在这些方法中。

我们从不需要计算超过两个导数。

但是为什么函数的一阶和二阶导数对于找到其最优解如此重要呢？简洁的答案是，一阶导数包含了函数在某一点上增加或减少的速度信息（因此如果你按照它的方向，你可能会上升到最大值或下降到最小值），而二阶导数包含了函数的“碗”的形状信息，它是向上弯曲还是向下弯曲。

微积分中的一个关键思想仍然是基本的：极小值（和/或极大值）发生在*临界点*（定义为函数的一个导数为零或不存在的点）或边界点。因此，为了找到这些最优解，我们必须搜索*边界点*（如果我们的搜索空间有边界）*和*内部临界点。

我们如何找到搜索空间内部的临界点？

方法1

我们按照这些步骤进行。

+   找到我们函数的一个导数（不太糟糕，我们在微积分中都做过），

+   然后将其设置为零（我们都可以写出等于和零的符号），

+   并解出使我们的导数为零的<math alttext="omega"><mi>ω</mi></math>（这是一个糟糕的步骤！）。

对于其导数是线性的函数，比如我们的均方误差损失函数，解这些<math alttext="omega"><mi>ω</mi></math>是相对容易的。线性代数领域特别是为了帮助解线性方程组而建立的。数值线性代数领域是为了帮助解决现实和大型的线性方程组而建立的，其中病态条件很普遍。当我们的系统是线性的时，我们有很多可用的工具（和软件包）。

另一方面，当我们的方程是非线性的时，找到解是完全不同的故事。这成为了一个碰运气的游戏，大多数情况下都是运气不佳！以下是一个简短的例子，说明了解线性和非线性方程之间的差异：

解线性方程

找到<math alttext="omega"><mi>ω</mi></math>，使得<math alttext="0.002 omega minus 5 equals 0"><mrow><mn>0</mn> <mo>.</mo> <mn>002</mn> <mi>ω</mi> <mo>-</mo> <mn>5</mn> <mo>=</mo> <mn>0</mn></mrow></math>。

*解决方案*：将5移到另一边，然后除以0.002，我们得到<math alttext="omega equals 5 slash 0.002 equals 2500"><mrow><mi>ω</mi> <mo>=</mo> <mn>5</mn> <mo>/</mo> <mn>0</mn> <mo>.</mo> <mn>002</mn> <mo>=</mo> <mn>2500</mn></mrow></math>。完成。

解非线性方程

找到<math alttext="omega"><mi>ω</mi></math>，使得<math alttext="0.002 sine left-parenthesis omega right-parenthesis minus 5 omega squared plus e Superscript omega Baseline equals 0"><mrow><mn>0</mn> <mo>.</mo> <mn>002</mn> <mo form="prefix">sin</mo> <mrow><mo>(</mo> <mi>ω</mi> <mo>)</mo></mrow> <mo>-</mo> <mn>5</mn> <msup><mi>ω</mi> <mn>2</mn></msup> <mo>+</mo> <msup><mi>e</mi> <mi>ω</mi></msup> <mo>=</mo> <mn>0</mn></mrow></math>。

*解决方案*：是的，我要离开了。我们需要一个数值方法！（见[图3-11](#Fig_roots_f_nonlinear)以图形逼近解这个非线性方程的解）。

![275](assets/emai_0311.png)

###### 图3-11。解非线性方程很困难。在这里，我们绘制<math alttext="f left-parenthesis omega right-parenthesis equals 0.002 sine left-parenthesis omega right-parenthesis minus 5 omega squared plus e Superscript omega"><mrow><mi>f</mi> <mrow><mo>(</mo> <mi>ω</mi> <mo>)</mo></mrow> <mo>=</mo> <mn>0</mn> <mo>.</mo> <mn>002</mn> <mo form="prefix">sin</mo> <mrow><mo>(</mo> <mi>ω</mi> <mo>)</mo></mrow> <mo>-</mo> <mn>5</mn> <msup><mi>ω</mi> <mn>2</mn></msup> <mo>+</mo> <msup><mi>e</mi> <mi>ω</mi></msup></mrow></math>并在图上近似其三个根（使<math alttext="f left-parenthesis omega right-parenthesis equals 0"><mrow><mi>f</mi> <mo>(</mo> <mi>ω</mi> <mo>)</mo> <mo>=</mo> <mn>0</mn></mrow></math>的点）。

有许多数值技术专门用于求解非线性方程的解（以及专门用于数值求解非线性常微分方程和偏微分方程的整个领域）。这些方法找到近似解，然后提供数值解与精确解相差多远的界限。它们通常构造一个在某些条件下收敛到解析解的序列。有些方法收敛速度比其他方法快，并且更适合某些问题而不适合其他问题。

方法2

另一个选择是沿着梯度方向前进，以便向最小值下降或向最大值上升。

要理解这些梯度类型的方法，可以想象下山徒步旅行（或者如果方法加速或具有动量，则滑雪下山）。我们从搜索空间中的一个随机点开始，这将使我们处于函数景观的初始高度水平。现在，该方法将我们移动到搜索空间中的一个新点，希望在这个新位置，我们最终到达的高度水平比我们来自的高度水平*更低*。因此，我们将*下降*。我们重复这个过程，理想情况下，如果函数的地形配合，这些点的序列将收敛到我们正在寻找的函数的最小化器。

当然，对于函数的景观具有许多峰和谷的情况，我们从哪里开始，或者换句话说，如何初始化，很重要，因为我们可能下降到与我们想要到达的完全不同的山谷。我们可能最终会陷入局部最小值而不是全局最小值。

凸函数和下界有界的函数的形状像沙拉碗，因此我们不必担心被困在局部最小值处，远离全局最小值。凸函数可能存在另一个令人担忧的问题：当函数的碗形太窄时，我们的方法可能变得非常缓慢。我们将在下一章中详细讨论这一点。

方法1和方法2都很有用且受欢迎。有时，我们别无选择，只能使用其中一种，这取决于每种方法在特定设置下收敛的速度有多快，我们试图优化的函数有多“规则”（它有多少良好的导数），等等。有时，这只是品味的问题。对于线性回归的均方误差损失函数，两种方法都适用，所以我们将使用方法1，只是因为我们将在本书中对*所有*其他损失函数使用梯度下降方法。

我们必须提到，对于下降方法的登山下山类比是很好的，但有点误导。当我们人类下山时，我们在物理上属于与我们的山脉景观存在的相同三维空间，这意味着我们处于某个海拔高度，我们能够下降到更低海拔的位置，即使我们被蒙住眼睛，即使天气雾蒙蒙，我们只能一步一步地下降。我们感知海拔然后向下移动。另一方面，数值下降方法并不在与函数景观嵌入的相同空间维度中搜索最小值。相反，它们在地面上搜索，即函数景观的一维*下方*（参见[图3-12](#Fig_ground_level_search)）。这使得朝向最小值的下降变得更加困难，因为在地面上，我们可以从任意一点移动到任何其他点，而不知道我们上方存在什么高度水平，直到我们在该点评估函数本身并找到高度。因此，我们的方法可能会意外地将我们从一个具有某个海拔高度的地面点移动到另一个具有*更高*海拔高度的地面点，因此离最小值更远。这就是为什么在地面上定位一个快速*减小*函数高度的方向以及我们在地面上可以移动多远（步长）而*仍然减小*我们上方函数高度的重要性。步长也称为*学习率超参数*，每当我们使用下降方法时都会遇到。

![250](assets/emai_0312.png)

###### 图3-12。搜索最小值发生在地面上，而不是直接在函数的景观上。

回到我们的主要目标：我们想要找到最佳的<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>，以便为我们的训练函数最小化均方误差损失函数，方法1：对损失函数进行一阶导数，并将其设置为零，然后解出向量<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>。为此，我们需要掌握*对线性代数表达式进行微积分*。让我们首先回顾一下我们的微积分课程。

### 微积分简介

在微积分的第一门课程中，我们学习关于单变量函数（<math alttext="f left-parenthesis omega right-parenthesis"><mrow><mi>f</mi> <mo>(</mo> <mi>ω</mi> <mo>)</mo></mrow></math>）及其图形，并在特定点进行评估。然后我们学习数学分析中最重要的操作：极限。从极限概念中，我们定义函数的连续性和不连续性，点的导数<math alttext="f prime left-parenthesis omega right-parenthesis"><mrow><msup><mi>f</mi> <mo>'</mo></msup> <mrow><mo>(</mo> <mi>ω</mi> <mo>)</mo></mrow></mrow></math>（通过点的切线斜率的极限）和域上的积分（由函数在域上确定的小区域的和的极限）。我们以微积分基本定理结束课程，将积分和微分作为反向操作进行关联。导数的关键特性之一是它确定函数在某一点的增长或减少速度，因此，在其定义域的内部定位函数的最小值和/或最大值中起着至关重要的作用（边界点是分开的）。

在多变量微积分课程中，通常是微积分的第三门课程，许多概念从单变量微积分中转移，包括导数的重要性，现在称为*梯度*，因为我们有几个变量，用于定位任何内部最小值和/或最大值。函数<math alttext="f left-parenthesis ModifyingAbove omega With right-arrow right-parenthesis"><mrow><mi>f</mi> <mo>(</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>)</mo></mrow></math>的梯度<math alttext="normal nabla left-parenthesis f left-parenthesis ModifyingAbove omega With right-arrow right-parenthesis right-parenthesis"><mrow><mi>∇</mi> <mo>(</mo> <mi>f</mi> <mrow><mo>(</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>)</mo></mrow> <mo>)</mo></mrow></math>是函数对于变量向量<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>的导数。

在深度学习中，未知的权重是以矩阵而不是向量的形式组织的，因此我们需要对变量矩阵*W*的函数<math alttext="f left-parenthesis upper W right-parenthesis"><mrow><mi>f</mi> <mo>(</mo> <mi>W</mi> <mo>)</mo></mrow></math>进行导数。

对于我们在AI中的目的，我们需要计算导数的函数是损失函数，其中包含了训练函数。根据导数的链式法则，我们还需要计算对于<math alttext="omega"><mi>ω</mi></math>的训练函数的导数。

让我们使用单变量微积分的一个简单例子来演示，然后立即过渡到对线性代数表达式进行导数运算。

### 一维优化示例

*找到函数<math alttext="f left-parenthesis omega right-parenthesis equals 3 plus left-parenthesis 0.5 omega minus 2 right-parenthesis squared"><mrow><mi>f</mi> <mrow><mo>(</mo> <mi>ω</mi> <mo>)</mo></mrow> <mo>=</mo> <mn>3</mn> <mo>+</mo> <msup><mrow><mo>(</mo><mn>0</mn><mo>.</mo><mn>5</mn><mi>ω</mi><mo>-</mo><mn>2</mn><mo>)</mo></mrow> <mn>2</mn></msup></mrow></math>在区间[-1,6]上的最小值（如果有的话）和最小值。*

一个不可能的长方法是尝试在-1和6之间尝试*无限多*个<math alttext="omega"><mi>ω</mi></math>的值，并选择给出最低*f*值的<math alttext="omega"><mi>ω</mi></math>。另一种方法是使用我们的微积分知识，即优化器（最小化器和/或最大化器）发生在临界点（导数不存在或为零）或边界点。有关参考，请参阅[图3-13](#Fig_minimize_f)。

![275](assets/emai_0313.png)

###### 图3-13。函数的最小值<math alttext="f left-parenthesis omega right-parenthesis equals 3 plus left-parenthesis 0.5 omega minus 2 right-parenthesis squared"><mrow><mi>f</mi> <mrow><mo>(</mo> <mi>ω</mi> <mo>)</mo></mrow> <mo>=</mo> <mn>3</mn> <mo>+</mo> <msup><mrow><mo>(</mo><mn>0</mn><mo>.</mo><mn>5</mn><mi>ω</mi><mo>-</mo><mn>2</mn><mo>)</mo></mrow> <mn>2</mn></msup></mrow></math>在区间[-1,6]上是3，并且发生在临界点<math alttext="omega equals 4"><mrow><mi>ω</mi> <mo>=</mo> <mn>4</mn></mrow></math>。在这个临界点，函数的导数为零，这意味着如果我们画一条切线，它将是水平的。

我们的边界点是*-1*和*6*，所以我们首先在这些点上评估我们的函数：<math alttext="f left-parenthesis negative 1 right-parenthesis equals 3 plus left-parenthesis 0.5 left-parenthesis negative 1 right-parenthesis minus 2 right-parenthesis squared equals 9.25"><mrow><mi>f</mi> <mrow><mo>(</mo> <mo>-</mo> <mn>1</mn> <mo>)</mo></mrow> <mo>=</mo> <mn>3</mn> <mo>+</mo> <msup><mrow><mo>(</mo><mn>0</mn><mo>.</mo><mn>5</mn><mrow><mo>(</mo><mo>-</mo><mn>1</mn><mo>)</mo></mrow><mo>-</mo><mn>2</mn><mo>)</mo></mrow> <mn>2</mn></msup> <mo>=</mo> <mn>9</mn> <mo>.</mo> <mn>25</mn></mrow></math> 和 <math alttext="f left-parenthesis 6 right-parenthesis equals 3 plus left-parenthesis 0.5 left-parenthesis 6 right-parenthesis minus 2 right-parenthesis squared equals 4"><mrow><mi>f</mi> <mrow><mo>(</mo> <mn>6</mn> <mo>)</mo></mrow> <mo>=</mo> <mn>3</mn> <mo>+</mo> <msup><mrow><mo>(</mo><mn>0</mn><mo>.</mo><mn>5</mn><mrow><mo>(</mo><mn>6</mn><mo>)</mo></mrow><mo>-</mo><mn>2</mn><mo>)</mo></mrow> <mn>2</mn></msup> <mo>=</mo> <mn>4</mn></mrow></math> 。显然，*-1*不是一个最小化器，因为*f(6)<f(-1)*，所以这个边界点退出了竞争，现在只有边界点*6*与内部临界点竞争。为了找到我们的临界点，我们检查区间*[-1,6]*内函数的导数：<math alttext="f prime left-parenthesis omega right-parenthesis equals 0 plus 2 left-parenthesis 0.5 omega minus 2 right-parenthesis asterisk 0.5 equals 0.25 left-parenthesis 0.5 omega minus 2 right-parenthesis"><mrow><msup><mi>f</mi> <mo>'</mo></msup> <mrow><mo>(</mo> <mi>ω</mi> <mo>)</mo></mrow> <mo>=</mo> <mn>0</mn> <mo>+</mo> <mn>2</mn> <mrow><mo>(</mo> <mn>0</mn> <mo>.</mo> <mn>5</mn> <mi>ω</mi> <mo>-</mo> <mn>2</mn> <mo>)</mo></mrow> <mo>*</mo> <mn>0</mn> <mo>.</mo> <mn>5</mn> <mo>=</mo> <mn>0</mn> <mo>.</mo> <mn>25</mn> <mrow><mo>(</mo> <mn>0</mn> <mo>.</mo> <mn>5</mn> <mi>ω</mi> <mo>-</mo> <mn>2</mn> <mo>)</mo></mrow></mrow></math> 。将这个导数设为零，我们有<math alttext="0.25 left-parenthesis 0.5 omega minus 2 right-parenthesis equals 0"><mrow><mn>0</mn> <mo>.</mo> <mn>25</mn> <mo>(</mo> <mn>0</mn> <mo>.</mo> <mn>5</mn> <mi>ω</mi> <mo>-</mo> <mn>2</mn> <mo>)</mo> <mo>=</mo> <mn>0</mn></mrow></math> 意味着<math alttext="omega equals 4"><mrow><mi>ω</mi> <mo>=</mo> <mn>4</mn></mrow></math> 。因此，我们只在区间*[-1,6]*的内部找到了一个临界点<math alttext="omega equals 4"><mrow><mi>ω</mi> <mo>=</mo> <mn>4</mn></mrow></math>。在这个特殊点，函数的值是<math alttext="f left-parenthesis 4 right-parenthesis equals 3 plus left-parenthesis 0.5 left-parenthesis 4 right-parenthesis minus 2 right-parenthesis squared equals 3"><mrow><mi>f</mi> <mrow><mo>(</mo> <mn>4</mn> <mo>)</mo></mrow> <mo>=</mo> <mn>3</mn> <mo>+</mo> <msup><mrow><mo>(</mo><mn>0</mn><mo>.</mo><mn>5</mn><mrow><mo>(</mo><mn>4</mn><mo>)</mo></mrow><mo>-</mo><mn>2</mn><mo>)</mo></mrow> <mn>2</mn></msup> <mo>=</mo> <mn>3</mn></mrow></math> 。由于*f*的值在这里是最低的，显然我们已经找到了我们最小化竞赛的赢家，即<math alttext="omega equals 4"><mrow><mi>ω</mi> <mo>=</mo> <mn>4</mn></mrow></math>，最小的*f*值等于*3*。

### 我们经常使用的线性代数表达式的导数

在涉及向量和矩阵的表达式上直接计算导数是高效的，而不必将它们分解成它们的分量。以下两种方法很受欢迎：

1.  当*a*和<math alttext="omega"><mi>ω</mi></math>是标量且*a*是常数时，<math alttext="f left-parenthesis omega right-parenthesis equals a omega"><mrow><mi>f</mi> <mo>(</mo> <mi>ω</mi> <mo>)</mo> <mo>=</mo> <mi>a</mi> <mi>ω</mi></mrow></math>的导数是<math alttext="f prime left-parenthesis omega right-parenthesis equals a"><mrow><msup><mi>f</mi> <mo>'</mo></msup> <mrow><mo>(</mo> <mi>ω</mi> <mo>)</mo></mrow> <mo>=</mo> <mi>a</mi></mrow></math>。当<math alttext="ModifyingAbove a With right-arrow"><mover accent="true"><mi>a</mi> <mo>→</mo></mover></math>和<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>是向量（长度相同）且<math alttext="ModifyingAbove a With right-arrow"><mover accent="true"><mi>a</mi> <mo>→</mo></mover></math>的条目是常数时，<math alttext="f left-parenthesis ModifyingAbove omega With right-arrow right-parenthesis equals ModifyingAbove a With right-arrow Superscript t Baseline ModifyingAbove omega With right-arrow"><mrow><mi>f</mi> <mrow><mo>(</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>)</mo></mrow> <mo>=</mo> <msup><mover accent="true"><mi>a</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover></mrow></math>的梯度是<math alttext="normal nabla f left-parenthesis ModifyingAbove omega With right-arrow right-parenthesis equals ModifyingAbove a With right-arrow"><mrow><mi>∇</mi> <mi>f</mi> <mrow><mo>(</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>)</mo></mrow> <mo>=</mo> <mover accent="true"><mi>a</mi> <mo>→</mo></mover></mrow></math>。同样，<math alttext="f left-parenthesis ModifyingAbove omega With right-arrow right-parenthesis equals ModifyingAbove w With right-arrow Superscript t Baseline ModifyingAbove a With right-arrow"><mrow><mi>f</mi> <mrow><mo>(</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>)</mo></mrow> <mo>=</mo> <msup><mover accent="true"><mi>w</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>a</mi> <mo>→</mo></mover></mrow></math>的梯度是<math alttext="normal nabla f left-parenthesis ModifyingAbove omega With right-arrow right-parenthesis equals ModifyingAbove a With right-arrow"><mrow><mi>∇</mi> <mi>f</mi> <mrow><mo>(</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>)</mo></mrow> <mo>=</mo> <mover accent="true"><mi>a</mi> <mo>→</mo></mover></mrow></math>。

1.  当*s*是标量且常数，<math alttext="omega"><mi>ω</mi></math>是标量时，二次函数<math alttext="f left-parenthesis omega right-parenthesis equals s omega squared"><mrow><mi>f</mi> <mrow><mo>(</mo> <mi>ω</mi> <mo>)</mo></mrow> <mo>=</mo> <mi>s</mi> <msup><mi>ω</mi> <mn>2</mn></msup></mrow></math>的导数是<math alttext="f prime left-parenthesis omega right-parenthesis equals 2 s omega"><mrow><msup><mi>f</mi> <mo>'</mo></msup> <mrow><mo>(</mo> <mi>ω</mi> <mo>)</mo></mrow> <mo>=</mo> <mn>2</mn> <mi>s</mi> <mi>ω</mi></mrow></math>。类似的高维情况是当*S*是具有常数条目的对称矩阵时，函数<math alttext="f left-parenthesis ModifyingAbove omega With right-arrow right-parenthesis equals ModifyingAbove omega With right-arrow Superscript t Baseline upper S ModifyingAbove omega With right-arrow"><mrow><mi>f</mi> <mrow><mo>(</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>)</mo></mrow> <mo>=</mo> <msup><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mi>S</mi> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover></mrow></math>是二次的，其梯度是<math alttext="normal nabla f left-parenthesis ModifyingAbove omega With right-arrow right-parenthesis equals 2 upper S ModifyingAbove omega With right-arrow"><mrow><mi>∇</mi> <mi>f</mi> <mrow><mo>(</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>)</mo></mrow> <mo>=</mo> <mn>2</mn> <mi>S</mi> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover></mrow></math>。

### 最小化均方误差损失函数

我们终于准备好最小化均方误差损失函数了。

<math alttext="dollar-sign upper L left-parenthesis ModifyingAbove omega With right-arrow right-parenthesis equals StartFraction 1 Over m EndFraction left-parenthesis upper X ModifyingAbove omega With right-arrow minus ModifyingAbove y With right-arrow Subscript t r u e Baseline right-parenthesis Superscript t Baseline left-parenthesis upper X ModifyingAbove omega With right-arrow minus ModifyingAbove y With right-arrow Subscript t r u e Baseline right-parenthesis period dollar-sign"><mrow><mi>L</mi> <mrow><mo>(</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>)</mo></mrow> <mo>=</mo> <mfrac><mn>1</mn> <mi>m</mi></mfrac> <msup><mfenced close=")" open="(" separators=""><mi>X</mi><mover accent="true"><mi>ω</mi> <mo>→</mo></mover><mo>-</mo><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></mfenced> <mi>t</mi></msup> <mfenced close=")" open="(" separators=""><mi>X</mi> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>-</mo> <msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></mfenced> <mo>.</mo></mrow></math>

让我们在将其梯度展开并将其设置为零之前打开上述表达式：

数学符号开始布局第一行第一列上方L左括号修改上方omega箭头右括号第二列等于分数1/m左括号左括号上方X修改上方omega箭头右括号上标t减去上方y箭头下标true上标t右括号左括号上方X修改上方omega箭头减去上方y箭头下标true右括号第二行第一列空第二列等于分数1/m左括号上方omega箭头上标t上方X上标t减去上方y箭头下标true上标t右括号左括号上方X修改上方omega箭头减去上方y箭头下标true右括号第三行第一列空第二列等于分数1/m左括号上方omega箭头上标t上方X上标t上方X修改上方omega箭头减去上方omega箭头上标t上方X上标t上方y箭头下标true减去上方y箭头下标true上标t上方X修改上方omega箭头加上上方y箭头下标true上标t上方y箭头下标true右括号第四行第一列空第二列等于分数1/m左括号上方omega箭头上标S上方omega箭头减去上方omega箭头上标t上方a箭头减去上方a箭头上标t上方omega箭头加上上方y箭头下标true上标t上方y箭头下标true右括号，结束布局。

在最后一步中，我们设置了<math alttext="upper X Superscript t Baseline upper X equals upper S"><mrow><msup><mi>X</mi> <mi>t</mi></msup> <mi>X</mi> <mo>=</mo> <mi>S</mi></mrow></math>和<math alttext="upper X Superscript t Baseline ModifyingAbove y With right-arrow Subscript t r u e Baseline equals ModifyingAbove a With right-arrow"><mrow><msup><mi>X</mi> <mi>t</mi></msup> <msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>=</mo> <mover accent="true"><mi>a</mi> <mo>→</mo></mover></mrow></math>。接下来，对上述表达式关于<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>的梯度进行计算，并将其设置为零。在计算梯度时，我们使用了上面小节中学到的关于线性代数表达式的微分知识：

<math alttext="dollar-sign StartLayout 1st Row 1st Column normal nabla upper L left-parenthesis ModifyingAbove omega With right-arrow right-parenthesis 2nd Column equals StartFraction 1 Over m EndFraction left-parenthesis 2 upper S ModifyingAbove omega With right-arrow minus ModifyingAbove a With right-arrow minus ModifyingAbove a With right-arrow plus 0 right-parenthesis 2nd Row 1st Column Blank 2nd Column equals ModifyingAbove 0 With right-arrow EndLayout dollar-sign"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi>∇</mi> <mi>L</mi> <mo>(</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>)</mo></mrow></mtd> <mtd columnalign="left"><mrow><mo>=</mo> <mfrac><mn>1</mn> <mi>m</mi></mfrac> <mfenced close=")" open="(" separators=""><mn>2</mn> <mi>S</mi> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>-</mo> <mover accent="true"><mi>a</mi> <mo>→</mo></mover> <mo>-</mo> <mover accent="true"><mi>a</mi> <mo>→</mo></mover> <mo>+</mo> <mn>0</mn></mfenced></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>=</mo> <mover accent="true"><mn>0</mn> <mo>→</mo></mover></mrow></mtd></mtr></mtable></math>

现在很容易解出<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>：

<math alttext="dollar-sign StartFraction 1 Over m EndFraction left-parenthesis 2 upper S ModifyingAbove omega With right-arrow minus 2 ModifyingAbove a With right-arrow right-parenthesis equals ModifyingAbove 0 With right-arrow dollar-sign"><mrow><mfrac><mn>1</mn> <mi>m</mi></mfrac> <mfenced close=")" open="(" separators=""><mn>2</mn> <mi>S</mi> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>-</mo> <mn>2</mn> <mover accent="true"><mi>a</mi> <mo>→</mo></mover></mfenced> <mo>=</mo> <mover accent="true"><mn>0</mn> <mo>→</mo></mover></mrow></math>

所以

<math alttext="dollar-sign 2 upper S ModifyingAbove omega With right-arrow equals 2 ModifyingAbove a With right-arrow dollar-sign"><mrow><mn>2</mn> <mi>S</mi> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>=</mo> <mn>2</mn> <mover accent="true"><mi>a</mi> <mo>→</mo></mover></mrow></math>

这给出了

<math alttext="dollar-sign ModifyingAbove omega With right-arrow equals upper S Superscript negative 1 Baseline ModifyingAbove a With right-arrow dollar-sign"><mrow><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>=</mo> <msup><mi>S</mi> <mrow><mo>-</mo><mn>1</mn></mrow></msup> <mover accent="true"><mi>a</mi> <mo>→</mo></mover></mrow></math>

现在回想一下，我们设置了<math alttext="upper S equals upper X Superscript t Baseline upper X"><mrow><mi>S</mi> <mo>=</mo> <msup><mi>X</mi> <mi>t</mi></msup> <mi>X</mi></mrow></math>和<math alttext="ModifyingAbove a With right-arrow equals upper X Superscript t Baseline y Subscript t r u e"><mrow><mover accent="true"><mi>a</mi> <mo>→</mo></mover> <mo>=</mo> <msup><mi>X</mi> <mi>t</mi></msup> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></mrow></math>，所以让我们用训练集*X*（增加了1）和相应的标签向量<math alttext="ModifyingAbove y With right-arrow Subscript t r u e"><msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></math>来重新写我们的最小化<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>：

<math alttext="dollar-sign ModifyingAbove omega With right-arrow equals left-parenthesis upper X Superscript t Baseline upper X right-parenthesis Superscript negative 1 Baseline upper X Superscript t Baseline ModifyingAbove y With right-arrow Subscript t r u e Baseline period dollar-sign"><mrow><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>=</mo> <msup><mrow><mo>(</mo><msup><mi>X</mi> <mi>t</mi></msup> <mi>X</mi><mo>)</mo></mrow> <mrow><mo>-</mo><mn>1</mn></mrow></msup> <msup><mi>X</mi> <mi>t</mi></msup> <msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>.</mo></mrow></math>

对于Fish Market数据集，这将是（请参阅附带的Jupyter笔记本）：

<math alttext="dollar-sign ModifyingAbove omega With right-arrow equals Start 6 By 1 Matrix 1st Row  omega 0 2nd Row  omega 1 3rd Row  omega 2 4th Row  omega 3 5th Row  omega 4 6th Row  omega 5 EndMatrix equals Start 6 By 1 Matrix 1st Row  negative 475.19929130109716 2nd Row  82.84970118 3rd Row  negative 28.85952426 4th Row  negative 28.50769512 5th Row  29.82981435 6th Row  30.97250278 EndMatrix dollar-sign"><mrow><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>=</mo> <mfenced close=")" open="("><mtable><mtr><mtd><msub><mi>ω</mi> <mn>0</mn></msub></mtd></mtr> <mtr><mtd><msub><mi>ω</mi> <mn>1</mn></msub></mtd></mtr> <mtr><mtd><msub><mi>ω</mi> <mn>2</mn></msub></mtd></mtr> <mtr><mtd><msub><mi>ω</mi> <mn>3</mn></msub></mtd></mtr> <mtr><mtd><msub><mi>ω</mi> <mn>4</mn></msub></mtd></mtr> <mtr><mtd><msub><mi>ω</mi> <mn>5</mn></msub></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced close=")" open="("><mtable><mtr><mtd><mrow><mo>-</mo> <mn>475</mn> <mo>.</mo> <mn>19929130109716</mn></mrow></mtd></mtr> <mtr><mtd><mrow><mn>82</mn> <mo>.</mo> <mn>84970118</mn></mrow></mtd></mtr> <mtr><mtd><mrow><mo>-</mo> <mn>28</mn> <mo>.</mo> <mn>85952426</mn></mrow></mtd></mtr> <mtr><mtd><mrow><mo>-</mo> <mn>28</mn> <mo>.</mo> <mn>50769512</mn></mrow></mtd></mtr> <mtr><mtd><mrow><mn>29</mn> <mo>.</mo> <mn>82981435</mn></mrow></mtd></mtr> <mtr><mtd><mrow><mn>30</mn> <mo>.</mo> <mn>97250278</mn></mrow></mtd></mtr></mtable></mfenced></mrow></math>

# 注意：将大矩阵相乘非常昂贵。请改为将矩阵乘以向量。

尽量避免相乘矩阵，而是用*向量*相乘。例如，在正规方程<math alttext="ModifyingAbove omega With right-arrow equals left-parenthesis upper X Superscript t Baseline upper X right-parenthesis Superscript negative 1 Baseline upper X Superscript t Baseline ModifyingAbove y With right-arrow Subscript t r u e"><mrow><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>=</mo> <msup><mrow><mo>(</mo><msup><mi>X</mi> <mi>t</mi></msup> <mi>X</mi><mo>)</mo></mrow> <mrow><mo>-</mo><mn>1</mn></mrow></msup> <msup><mi>X</mi> <mi>t</mi></msup> <msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></mrow></math>，首先计算<math alttext="upper X Superscript t Baseline ModifyingAbove y With right-arrow Subscript t r u e"><mrow><msup><mi>X</mi> <mi>t</mi></msup> <msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></mrow></math>，避免计算<math alttext="left-parenthesis upper X Superscript t Baseline upper X right-parenthesis Superscript negative 1"><msup><mrow><mo>(</mo><msup><mi>X</mi> <mi>t</mi></msup> <mi>X</mi><mo>)</mo></mrow> <mrow><mo>-</mo><mn>1</mn></mrow></msup></math>。解决这个问题的方法是使用*X*的*伪逆*来解决线性系统<math alttext="upper X ModifyingAbove omega With right-arrow equals ModifyingAbove y With right-arrow Subscript t r u e"><mrow><mi>X</mi> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>=</mo> <msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></mrow></math>（查看附带的Jupyter笔记本）。我们将在[第11章](ch11.xhtml#ch11)中讨论*伪逆*，但现在，它允许我们求解（相当于除以）没有逆的矩阵。

我们只是找到了权重向量<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>，它能够在我们的训练数据和线性回归训练函数之间提供最佳拟合：

<math alttext="dollar-sign f left-parenthesis ModifyingAbove omega With right-arrow semicolon ModifyingAbove x With right-arrow right-parenthesis equals omega 0 plus omega 1 x 1 plus omega 2 x 2 plus omega 3 x 3 plus omega 4 x 4 plus omega 5 x 5 period dollar-sign"><mrow><mi>f</mi> <mrow><mo>(</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>;</mo> <mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mo>)</mo></mrow> <mo>=</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msub><mi>x</mi> <mn>1</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>2</mn></msub> <msub><mi>x</mi> <mn>2</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>3</mn></msub> <msub><mi>x</mi> <mn>3</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>4</mn></msub> <msub><mi>x</mi> <mn>4</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>5</mn></msub> <msub><mi>x</mi> <mn>5</mn></msub> <mo>.</mo></mrow></math>

我们使用了一种分析方法（计算损失函数的梯度并将其置为零）来推导正规方程给出的解决方案。这是我们能够推导出解析解的非常罕见的情况之一。所有其他找到最小化<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>的方法都是数值方法。

# 注意：我们永远不希望训练数据拟合得太好。

我们计算的<math alttext="ModifyingAbove omega With right-arrow equals left-parenthesis upper X Superscript t Baseline upper X right-parenthesis Superscript negative 1 Baseline upper X Superscript t Baseline ModifyingAbove y With right-arrow Subscript t r u e"><mrow><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>=</mo> <msup><mrow><mo>(</mo><msup><mi>X</mi> <mi>t</mi></msup> <mi>X</mi><mo>)</mo></mrow> <mrow><mo>-</mo><mn>1</mn></mrow></msup> <msup><mi>X</mi> <mi>t</mi></msup> <msub><mover accent="true"><mi>y</mi> <mo>→</mo></mover> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></mrow></math>给出了使训练函数*最佳拟合*训练数据的<math alttext="omega"><mi>ω</mi></math>值，但是过于拟合意味着训练函数可能也会捕捉到数据中的噪音而不仅仅是信号。因此，上述解决方案，甚至是最小化问题本身，需要进行修改，以*不要获得过于完美的拟合*。*正则化*或*提前停止*在这里是有帮助的。我们将在下一章节中花一些时间来讨论这些问题。

这是回归的长路。因为我们刚刚开始，所以我们必须通过微积分和线性代数。介绍即将到来的机器学习模型：逻辑回归、支持向量机、决策树和随机森林，将会更快，因为我们所做的一切都是将完全相同的思想应用于不同的函数。

# 逻辑回归：分类为两类

逻辑回归主要用于分类任务。我们首先解释如何将这个模型用于二元分类任务（将数据分类为两类，例如癌症/非癌症；适合儿童/不适合儿童；可能偿还贷款/不太可能等）。然后我们将模型推广到将数据分类为多个类别（例如，将手写数字图像分类为0、1、2、3、4、5、6、7、8或9）。同样，我们有相同的数学设置：

1.  训练函数

1.  损失函数

1.  优化

## 训练函数

与线性回归类似，逻辑回归的训练函数计算特征的线性组合并添加一个常数偏差项，但是不是直接输出结果，而是通过*逻辑函数*，其图在[图3-14](#Fig_logistic)中绘制，其公式为：

<math alttext="dollar-sign sigma left-parenthesis s right-parenthesis equals StartFraction 1 Over 1 plus e Superscript negative s Baseline EndFraction dollar-sign"><mrow><mi>σ</mi> <mrow><mo>(</mo> <mi>s</mi> <mo>)</mo></mrow> <mo>=</mo> <mfrac><mn>1</mn> <mrow><mn>1</mn><mo>+</mo><msup><mi>e</mi> <mrow><mo>-</mo><mi>s</mi></mrow></msup></mrow></mfrac></mrow></math>![275](assets/emai_0314.png)

###### 图3-14. 逻辑函数的图形 <math alttext="sigma left-parenthesis s right-parenthesis equals StartFraction 1 Over 1 plus e Superscript negative s Baseline EndFraction"><mrow><mi>σ</mi> <mrow><mo>(</mo> <mi>s</mi> <mo>)</mo></mrow> <mo>=</mo> <mfrac><mn>1</mn> <mrow><mn>1</mn><mo>+</mo><msup><mi>e</mi> <mrow><mo>-</mo><mi>s</mi></mrow></msup></mrow></mfrac></mrow></math>。注意，这个函数可以在任何*s*处进行评估，并始终输出介于*0*和*1*之间的数字，因此其输出可以被解释为概率。

这是一个只接受值在*0*和*1*之间的函数，因此它的输出可以被解释为数据点属于某一类的概率：如果输出小于*0.5*，则将数据点分类为属于第一类，如果输出大于*0.5*，则将数据点分类为另一类。数字*0.5*是做出分类数据点决定的*阈值*。

因此，这里的训练函数最终是特征的线性组合，加上偏差，首先与逻辑函数组合，最后与阈值函数组合：

<math alttext="dollar-sign y equals upper T h r e s h left-parenthesis sigma left-parenthesis omega 0 plus omega 1 x 1 plus ellipsis plus omega Subscript n Baseline x Subscript n Baseline right-parenthesis right-parenthesis dollar-sign"><mrow><mi>y</mi> <mo>=</mo> <mi>T</mi> <mi>h</mi> <mi>r</mi> <mi>e</mi> <mi>s</mi> <mi>h</mi> <mo>(</mo> <mi>σ</mi> <mrow><mo>(</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msub><mi>x</mi> <mn>1</mn></msub> <mo>+</mo> <mo>⋯</mo> <mo>+</mo> <msub><mi>ω</mi> <mi>n</mi></msub> <msub><mi>x</mi> <mi>n</mi></msub> <mo>)</mo></mrow> <mo>)</mo></mrow></math>

与线性回归情况类似，<math alttext="omega"><mi>ω</mi></math> 是我们需要优化损失函数的未知数。就像线性回归一样，这些未知数的数量等于数据特征的数量，再加上一个偏差项。对于像分类图像这样的任务，每个像素都是一个特征，所以我们可能有成千上万个。

## 损失函数

让我们为分类设计一个良好的*损失函数*。我们是工程师，我们希望惩罚错误分类的训练数据点。在我们的标记数据集中，如果一个实例属于一个类，那么它的<math alttext="y Subscript t r u e Baseline equals 1"><mrow><msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>=</mo> <mn>1</mn></mrow></math>，如果不属于，则<math alttext="y Subscript t r u e Baseline equals 0"><mrow><msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>=</mo> <mn>0</mn></mrow></math>。

我们希望我们的训练函数输出<math alttext="y Subscript p r e d i c t Baseline equals 1"><mrow><msub><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>=</mo> <mn>1</mn></mrow></math>，对于属于正类的训练实例（其<math alttext="y Subscript t r u e"><msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></math>也为1）。成功的<math alttext="omega"><mi>ω</mi></math>值应该给出一个较高的*t*值（线性组合步骤的结果），以进入逻辑函数，从而为正实例分配高概率，并通过0.5阈值获得<math alttext="y Subscript p r e d i c t Baseline equals 1"><mrow><msub><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>=</mo> <mn>1</mn></mrow></math>。因此，如果线性组合加偏差步骤给出一个较低的*t*值，而<math alttext="y Subscript t r u e Baseline equals 1"><mrow><msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>=</mo> <mn>1</mn></mrow></math>，则对其进行惩罚。

类似地，成功的权重值应该给出一个较低的*t*值，以进入逻辑函数，用于不属于该类的训练实例（它们真实的<math alttext="y Subscript t r u e Baseline equals 0"><mrow><msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>=</mo> <mn>0</mn></mrow></math>）。因此，如果线性组合加偏差步骤给出一个较高的*t*值，而<math alttext="y Subscript t r u e Baseline equals 0"><mrow><msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>=</mo> <mn>0</mn></mrow></math>，则对其进行惩罚。

那么我们如何找到一个惩罚错误分类的训练数据点的损失函数呢？假阳性和假阴性都应该受到惩罚。回想一下，这个分类模型的输出要么是*1*，要么是*0*：

+   想象一下奖励*1*并惩罚*0*的微积分函数：<math alttext="minus log left-parenthesis s right-parenthesis"><mrow><mo>-</mo> <mo form="prefix">log</mo> <mo>(</mo> <mi>s</mi> <mo>)</mo></mrow></math>（见[图3-15](#Fig_log_s_log_1_s)）。

+   考虑一个对*1*进行惩罚并对*0*进行奖励的微积分函数：<math alttext="减去对数左括号1减s右括号"><mrow><mo>-</mo> <mo form="prefix">log</mo> <mo>(</mo> <mn>1</mn> <mo>-</mo> <mi>s</mi> <mo>)</mo></mrow></math>（见[图3-15](#Fig_log_s_log_1_s)）。

![275](assets/emai_0315.png)

###### 图3-15\. 左：函数<math alttext="f左括号s右括号等于减l o g左括号s右括号"><mrow><mi>f</mi> <mo>(</mo> <mi>s</mi> <mo>)</mo> <mo>=</mo> <mo>-</mo> <mi>l</mi> <mi>o</mi> <mi>g</mi> <mo>(</mo> <mi>s</mi> <mo>)</mo></mrow></math>的图。该函数为接近*0*的数字分配高值，并为接近*1*的数字分配低值。右：函数<math alttext="f左括号s右括号等于减l o g左括号1减s右括号"><mrow><mi>f</mi> <mo>(</mo> <mi>s</mi> <mo>)</mo> <mo>=</mo> <mo>-</mo> <mi>l</mi> <mi>o</mi> <mi>g</mi> <mo>(</mo> <mn>1</mn> <mo>-</mo> <mi>s</mi> <mo>)</mo></mrow></math>的图。该函数为接近*1*的数字分配高值，并为接近*0*的数字分配低值。

现在关注当前选择的<math alttext="omega"><mi>ω</mi></math>的逻辑函数<math alttext="sigma左括号s右括号"><mrow><mi>σ</mi> <mo>(</mo> <mi>s</mi> <mo>)</mo></mrow></math>的输出：

+   如果<math alttext="sigma左括号s右括号"><mrow><mi>σ</mi> <mo>(</mo> <mi>s</mi> <mo>)</mo></mrow></math>小于*0.5*（模型预测为<math alttext="y下标predictBaseline等于0"><mrow><msub><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>=</mo> <mn>0</mn></mrow></math>），但真实值<math alttext="y下标trueBaseline等于1"><mrow><msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>=</mo> <mn>1</mn></mrow></math>（假阴性），则通过惩罚<math alttext="减去对数左括号sigma左括号s右括号右括号"><mrow><mo>-</mo> <mo form="prefix">log</mo> <mo>(</mo> <mi>σ</mi> <mo>(</mo> <mi>s</mi> <mo>)</mo> <mo>)</mo></mrow></math>让模型付出代价。如果相反地<math alttext="sigma左括号s右括号大于0.5"><mrow><mi>σ</mi> <mo>(</mo> <mi>s</mi> <mo>)</mo> <mo>></mo> <mn>0</mn> <mo>.</mo> <mn>5</mn></mrow></math>，即模型预测为<math alttext="y下标predictBaseline等于1"><mrow><msub><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>=</mo> <mn>1</mn></mrow></math>（真阳性），<math alttext="减去对数左括号sigma左括号s右括号右括号"><mrow><mo>-</mo> <mo form="prefix">log</mo> <mo>(</mo> <mi>σ</mi> <mo>(</mo> <mi>s</mi> <mo>)</mo> <mo>)</mo></mrow></math>很小，因此不需要付出高惩罚。

+   同样，如果<math alttext="sigma左括号s右括号"><mrow><mi>σ</mi> <mo>(</mo> <mi>s</mi> <mo>)</mo></mrow></math>大于*0.5*，但真实值<math alttext="y下标trueBaseline等于0"><mrow><msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>=</mo> <mn>0</mn></mrow></math>（假阳性），则通过惩罚<math alttext="减去对数左括号1减sigma左括号s右括号右括号"><mrow><mo>-</mo> <mo form="prefix">log</mo> <mo>(</mo> <mn>1</mn> <mo>-</mo> <mi>σ</mi> <mo>(</mo> <mi>s</mi> <mo>)</mo> <mo>)</mo></mrow></math>让模型付出代价。同样，对于真阴性也不需要付出高惩罚。

因此，我们可以将误分类一个训练实例的成本写成：

<math alttext="dollar-sign c o s t equals Start 2 By 2 Matrix 1st Row 1st Column Blank 2nd Column minus log left-parenthesis sigma left-parenthesis s right-parenthesis right-parenthesis if y Subscript t r u e Baseline equals 1 2nd Row 1st Column Blank 2nd Column minus log left-parenthesis 1 minus sigma left-parenthesis s right-parenthesis right-parenthesis if y Subscript t r u e Baseline equals 0 EndMatrix equals minus y Subscript t r u e Baseline log left-parenthesis sigma left-parenthesis s right-parenthesis right-parenthesis minus left-parenthesis 1 minus y Subscript t r u e Baseline right-parenthesis log left-parenthesis 1 minus sigma left-parenthesis s right-parenthesis right-parenthesis dollar-sign"><mrow><mi>c</mi> <mi>o</mi> <mi>s</mi> <mi>t</mi> <mo>=</mo> <mfenced close="}" open="{" separators=""><mtable displaystyle="true"><mtr><mtd columnalign="left"><mrow><mo>-</mo> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>σ</mi> <mrow><mo>(</mo> <mi>s</mi> <mo>)</mo></mrow> <mo>)</mo></mrow> <mtext>if</mtext> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>=</mo> <mn>1</mn></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>-</mo> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mn>1</mn> <mo>-</mo> <mi>σ</mi> <mrow><mo>(</mo> <mi>s</mi> <mo>)</mo></mrow> <mo>)</mo></mrow> <mtext>if</mtext> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>=</mo> <mn>0</mn></mrow></mtd></mtr></mtable></mfenced> <mo>=</mo> <mo>-</mo> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>σ</mi> <mrow><mo>(</mo> <mi>s</mi> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>-</mo> <mrow><mo>(</mo> <mn>1</mn> <mo>-</mo> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>)</mo></mrow> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mn>1</mn> <mo>-</mo> <mi>σ</mi> <mrow><mo>(</mo> <mi>s</mi> <mo>)</mo></mrow> <mo>)</mo></mrow></mrow></math>

最后，损失函数是*m*个训练实例的平均成本，给出了流行的*交叉熵损失函数*的公式。

<math alttext="dollar-sign StartLayout 1st Row 1st Column upper L left-parenthesis ModifyingAbove omega With right-arrow right-parenthesis equals 2nd Column minus StartFraction 1 Over m EndFraction sigma-summation Underscript i equals 1 Overscript m Endscripts y Subscript t r u e Superscript i Baseline log left-parenthesis sigma left-parenthesis omega 0 plus omega 1 x 1 Superscript i Baseline plus ellipsis plus omega Subscript n Baseline x Subscript n Superscript i Baseline right-parenthesis right-parenthesis plus 2nd Row 1st Column Blank 2nd Column left-parenthesis 1 minus y Subscript t r u e Superscript i Baseline right-parenthesis log left-parenthesis 1 minus sigma left-parenthesis omega 0 plus omega 1 x 1 Superscript i Baseline plus ellipsis plus omega Subscript n Baseline x Subscript n Superscript i Baseline right-parenthesis right-parenthesis EndLayout dollar-sign"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi>L</mi> <mo>(</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>)</mo> <mo>=</mo></mrow></mtd> <mtd columnalign="left"><mrow><mo>-</mo> <mfrac><mn>1</mn> <mi>m</mi></mfrac> <munderover><mo>∑</mo> <mrow><mi>i</mi><mo>=</mo><mn>1</mn></mrow> <mi>m</mi></munderover> <msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mi>i</mi></msubsup> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>σ</mi> <mrow><mo>(</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msubsup><mi>x</mi> <mn>1</mn> <mi>i</mi></msubsup> <mo>+</mo> <mo>⋯</mo> <mo>+</mo> <msub><mi>ω</mi> <mi>n</mi></msub> <msubsup><mi>x</mi> <mi>n</mi> <mi>i</mi></msubsup> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>+</mo></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mrow><mo>(</mo> <mn>1</mn> <mo>-</mo> <msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mi>i</mi></msubsup> <mo>)</mo></mrow> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mn>1</mn> <mo>-</mo> <mi>σ</mi> <mrow><mo>(</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msubsup><mi>x</mi> <mn>1</mn> <mi>i</mi></msubsup> <mo>+</mo> <mo>⋯</mo> <mo>+</mo> <msub><mi>ω</mi> <mi>n</mi></msub> <msubsup><mi>x</mi> <mi>n</mi> <mi>i</mi></msubsup> <mo>)</mo></mrow> <mo>)</mo></mrow></mrow></mtd></mtr></mtable></math>

## 优化

与线性回归情况不同，如果我们决定通过设置<math alttext="normal nabla upper L left-parenthesis omega right-parenthesis equals 0"><mrow><mi>∇</mi> <mi>L</mi> <mo>(</mo> <mi>ω</mi> <mo>)</mo> <mo>=</mo> <mn>0</mn></mrow></math>来最小化损失函数，那么<math alttext="omega"><mi>ω</mi></math>没有封闭形式的解析解。好消息是这个函数是凸函数，所以下一章的梯度下降（或随机或小批量梯度下降）保证能找到最小值（如果*学习率*不是太大，并且等待足够长的时间）。

# Softmax回归：多类分类

我们可以很容易地将逻辑回归的思想推广到多类分类。一个著名的非二进制分类任务的例子是使用[MNIST数据集](http://yann.lecun.com/exdb/mnist/)对手写数字0、1、2、3、4、5、6、7、8和9进行分类。这个数据集包含了70,000张手写数字的图像（见[图3-16](#Fig_MNIST)中这些图像的样本），分为60,000张训练子集和10,000张测试子集。每个图像都标有它所属的类别，即这十个数字中的一个。

![275](assets/emai_0316.png)

###### 图3-16。MNIST数据集的样本图像。([图片来源：维基百科](https://en.wikipedia.org/wiki/MNIST_database))

[此数据集的链接](http://yann.lecun.com/exdb/mnist/)还包含许多分类模型的结果，包括线性分类器、*k最近邻*、*决策树*、带有各种*核*的*支持向量机*，以及具有各种架构的神经网络，以及相应论文的参考文献和发表年份。看到随着年份的推移和方法的演变，性能的进展是很有趣的。

# 注意：不要将多类别分类与多输出模型混淆

Softmax回归一次预测一个类别，所以我们不能用它来分类，例如，在同一张图像中的五个人。相反，我们可以用它来检查给定的Facebook图像是否是我的照片，我的妹妹的照片，我的哥哥的照片，我的丈夫的照片，或者我的女儿的照片。传入softmax回归模型的图像只能有我们五个人中的一个，否则模型的分类就不太明显。这意味着我们的类别必须是相互排斥的。所以当Facebook自动在同一张图像中标记五个人时，它们使用的是一个多输出模型，而不是softmax回归模型。

假设我们有数据点的特征，并且我们想要利用这些信息来将数据点分类到*k*个可能的类别中。以下的训练函数、损失函数和优化过程现在应该是清楚的。

# 关于图像数据的特征

对于灰度图像，每个像素强度都是一个特征，所以图像通常有成千上万个特征。灰度图像通常表示为数字的二维矩阵，像素强度作为矩阵的条目。彩色图像有三个通道，红色、绿色和蓝色，每个通道再次表示为数字的二维矩阵，并且通道叠加在彼此之上，形成三层二维矩阵。这种结构称为张量。查看这个[链接的Jupyter笔记本]，它说明了我们如何在Python中处理灰度和彩色图像。

## 训练函数

第一步总是相同的：线性组合特征并添加一个常数偏差项。在逻辑回归中，当我们只有两个类别时，我们将结果传递到逻辑函数中，公式如下：

<math alttext="dollar-sign sigma left-parenthesis s right-parenthesis equals StartFraction 1 Over 1 plus e Superscript negative s Baseline EndFraction equals StartStartFraction 1 OverOver 1 plus StartFraction 1 Over e Superscript s Baseline EndFraction EndEndFraction equals StartFraction e Superscript s Baseline Over 1 plus e Superscript s Baseline EndFraction equals StartFraction e Superscript s Baseline Over e Superscript 0 Baseline plus e Superscript s Baseline EndFraction comma dollar-sign"><mrow><mi>σ</mi> <mrow><mo>(</mo> <mi>s</mi> <mo>)</mo></mrow> <mo>=</mo> <mfrac><mn>1</mn> <mrow><mn>1</mn><mo>+</mo><msup><mi>e</mi> <mrow><mo>-</mo><mi>s</mi></mrow></msup></mrow></mfrac> <mo>=</mo> <mfrac><mn>1</mn> <mrow><mn>1</mn><mo>+</mo><mfrac><mn>1</mn> <msup><mi>e</mi> <mi>s</mi></msup></mfrac></mrow></mfrac> <mo>=</mo> <mfrac><msup><mi>e</mi> <mi>s</mi></msup> <mrow><mn>1</mn><mo>+</mo><msup><mi>e</mi> <mi>s</mi></msup></mrow></mfrac> <mo>=</mo> <mfrac><msup><mi>e</mi> <mi>s</mi></msup> <mrow><msup><mi>e</mi> <mn>0</mn></msup> <mo>+</mo><msup><mi>e</mi> <mi>s</mi></msup></mrow></mfrac> <mo>,</mo></mrow></math>

我们将其解释为数据点属于感兴趣类别的概率或不属于的概率。请注意，我们将逻辑函数的公式重写为<math alttext="sigma left-parenthesis s right-parenthesis equals StartFraction e Superscript s Baseline Over e Superscript 0 Baseline plus e Superscript s Baseline EndFraction">，以突出它捕捉了两个概率，每个类别一个。换句话说，<math alttext="sigma left-parenthesis s right-parenthesis">给出了数据点属于感兴趣类别的概率，而<math alttext="1 minus sigma left-parenthesis s right-parenthesis equals StartFraction e Superscript 0 Baseline Over e Superscript 0 Baseline plus e Superscript s Baseline EndFraction">给出了数据点不属于该类别的概率。

当我们有多个类别而不仅仅是两个时，对于同一个数据点，我们重复相同的过程多次：每个类别一次。每个类别都有自己的偏差和一组权重，线性组合特征，因此，给定具有特征值<math alttext="x 1">，<math alttext="x 2">，...和<math alttext="x Subscript n">的数据点，我们计算*k*不同的线性组合加上偏差：

<math alttext="dollar-sign StartLayout 1st Row 1st Column s Superscript 1 2nd Column equals omega 0 Superscript 1 Baseline plus omega 1 Superscript 1 Baseline x 1 plus omega 2 Superscript 1 Baseline x 2 plus ellipsis plus omega Subscript n Superscript 1 Baseline x Subscript n Baseline 2nd Row 1st Column s squared 2nd Column equals omega 0 squared plus omega 1 squared x 1 plus omega 2 squared x 2 plus ellipsis plus omega Subscript n Superscript 2 Baseline x Subscript n Baseline 3rd Row 1st Column Blank 4th Row 1st Column  ellipsis 5th Row 1st Column s Superscript k 2nd Column equals omega 0 Superscript k Baseline plus omega 1 Superscript k Baseline x 1 plus omega 2 Superscript k Baseline x 2 plus ellipsis plus omega Subscript n Superscript k Baseline x Subscript n Baseline period EndLayout dollar-sign"><mtable displaystyle="true"><mtr><mtd columnalign="right"><msup><mi>s</mi> <mn>1</mn></msup></mtd> <mtd columnalign="left"><mrow><mo>=</mo> <msubsup><mi>ω</mi> <mn>0</mn> <mn>1</mn></msubsup> <mo>+</mo> <msubsup><mi>ω</mi> <mn>1</mn> <mn>1</mn></msubsup> <msub><mi>x</mi> <mn>1</mn></msub> <mo>+</mo> <msubsup><mi>ω</mi> <mn>2</mn> <mn>1</mn></msubsup> <msub><mi>x</mi> <mn>2</mn></msub> <mo>+</mo> <mo>⋯</mo> <mo>+</mo> <msubsup><mi>ω</mi> <mi>n</mi> <mn>1</mn></msubsup> <msub><mi>x</mi> <mi>n</mi></msub></mrow></mtd></mtr> <mtr><mtd columnalign="right"><msup><mi>s</mi> <mn>2</mn></msup></mtd> <mtd columnalign="left"><mrow><mo>=</mo> <msubsup><mi>ω</mi> <mn>0</mn> <mn>2</mn></msubsup> <mo>+</mo> <msubsup><mi>ω</mi> <mn>1</mn> <mn>2</mn></msubsup> <msub><mi>x</mi> <mn>1</mn></msub> <mo>+</mo> <msubsup><mi>ω</mi> <mn>2</mn> <mn>2</mn></msubsup> <msub><mi>x</mi> <mn>2</mn></msub> <mo>+</mo> <mo>⋯</mo> <mo>+</mo> <msubsup><mi>ω</mi> <mi>n</mi> <mn>2</mn></msubsup> <msub><mi>x</mi> <mi>n</mi></msub></mrow></mtd></mtr> <mtr><mtd columnalign="right"><mo>⋮</mo></mtd></mtr> <mtr><mtd columnalign="right"><msup><mi>s</mi> <mi>k</mi></msup></mtd> <mtd columnalign="left"><mrow><mo>=</mo> <msubsup><mi>ω</mi> <mn>0</mn> <mi>k</mi></msubsup> <mo>+</mo> <msubsup><mi>ω</mi> <mn>1</mn> <mi>k</mi></msubsup> <msub><mi>x</mi> <mn>1</mn></msub> <mo>+</mo> <msubsup><mi>ω</mi> <mn>2</mn> <mi>k</mi></msubsup> <msub><mi>x</mi> <mn>2</mn></msub> <mo>+</mo> <mo>⋯</mo> <mo>+</mo> <msubsup><mi>ω</mi> <mi>n</mi> <mi>k</mi></msubsup> <msub><mi>x</mi> <mi>n</mi></msub> <mo>.</mo></mrow></mtd></mtr></mtable></math>

# 养成良好习惯

你想养成一个良好的习惯，即跟踪你的训练函数中有多少未知的ω出现在公式中。记住，这些是我们通过最小化损失函数找到的ω。另一个良好的习惯是以高效和一致的方式组织它们在模型中（在向量、矩阵等中）。在softmax情况下，当我们有k个类别和每个数据点的n个特征时，我们最终得到k×n个ω用于线性组合，然后k个偏差，总共有k×n加k个未知的ω。例如，如果我们使用softmax回归模型来对手写数字的[MNIST数据集](https://en.wikipedia.org/wiki/MNIST_database)中的图像进行分类，每个图像有28×28个像素，即784个特征，我们想将它们分类为10个类别，因此我们最终需要优化7850个ω。对于线性和逻辑回归模型，我们只需要优化n加1个未知的ω。

接下来，我们将这*k*个结果传递到一个称为*softmax函数*的函数中，该函数将逻辑函数从两个类推广到多个类，并且我们也将其解释为概率。Softmax函数的公式如下：

<math alttext="dollar-sign sigma left-parenthesis s Superscript j Baseline right-parenthesis equals StartFraction e Superscript s Super Superscript j Superscript Baseline Over e Superscript s Super Superscript 1 Superscript Baseline plus e Superscript s squared Baseline plus ellipsis plus e Superscript s Super Superscript k Superscript Baseline EndFraction dollar-sign"><mrow><mi>σ</mi> <mrow><mo>(</mo> <msup><mi>s</mi> <mi>j</mi></msup> <mo>)</mo></mrow> <mo>=</mo> <mfrac><msup><mi>e</mi> <msup><mi>s</mi> <mi>j</mi></msup></msup> <mrow><msup><mi>e</mi> <msup><mi>s</mi> <mn>1</mn></msup></msup> <mo>+</mo><msup><mi>e</mi> <msup><mi>s</mi> <mn>2</mn></sup></msup> <mo>+</mo><mo>⋯</mo><mo>+</mo><msup><mi>e</mi> <msup><mi>s</mi> <mi>k</mi></msup></msup></mrow></mfrac></mrow></math>

这样，同一个数据点将得到*k*个概率分数，每个分数对应一个类别。最后，我们将数据点分类为获得最大概率分数的类别。

汇总以上所有内容，我们得到了训练函数的最终公式，现在我们可以用于分类（也就是在通过最小化适当的损失函数找到最优的<math alttext="omega"><mi>ω</mi></math>值之后）：

<math alttext="dollar-sign y equals j such that sigma left-parenthesis omega 0 Superscript j Baseline plus omega 1 Superscript j Baseline x 1 plus ellipsis plus omega Subscript n Superscript j Baseline x Subscript n Baseline right-parenthesis is maximal period dollar-sign"><mrow><mi>y</mi> <mo>=</mo> <mi>j</mi> <mtext>such</mtext> <mtext>that</mtext> <mi>σ</mi> <mo>(</mo> <msubsup><mi>ω</mi> <mn>0</mn> <mi>j</mi></msubsup> <mo>+</mo> <msubsup><mi>ω</mi> <mn>1</mn> <mi>j</mi></msubsup> <msub><mi>x</mi> <mn>1</mn></msub> <mo>+</mo> <mo>⋯</mo> <mo>+</mo> <msubsup><mi>ω</mi> <mi>n</mi> <mi>j</mi></msubsup> <msub><mi>x</mi> <mi>n</mi></msub> <mo>)</mo> <mtext>is</mtext> <mtext>maximal.</mtext></mrow></math>

请注意，对于上述训练函数，我们所需做的就是输入数据特征（*x*值），它将返回一个类别编号：*j*。

## 损失函数

我们推导了逻辑回归的交叉熵损失函数：

<math alttext="dollar-sign upper L left-parenthesis ModifyingAbove omega With right-arrow right-parenthesis equals minus StartFraction 1 Over m EndFraction sigma-summation Underscript i equals 1 Overscript m Endscripts y Subscript t r u e Superscript i Baseline log left-parenthesis sigma left-parenthesis omega 0 plus omega 1 x 1 Superscript i Baseline plus ellipsis plus omega Subscript n Baseline x Subscript n Superscript i Baseline right-parenthesis right-parenthesis plus left-parenthesis 1 minus y Subscript t r u e Superscript i Baseline right-parenthesis log left-parenthesis 1 minus sigma left-parenthesis omega 0 plus omega 1 x 1 Superscript i Baseline plus ellipsis plus omega Subscript n Baseline x Subscript n Superscript i Baseline right-parenthesis right-parenthesis comma dollar-sign"><mrow><mi>L</mi> <mrow><mo>(</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>)</mo></mrow> <mo>=</mo> <mo>-</mo> <mfrac><mn>1</mn> <mi>m</mi></mfrac> <msubsup><mo>∑</mo> <mrow><mi>i</mi><mo>=</mo><mn>1</mn></mrow> <mi>m</mi></msubsup> <msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mi>i</mi></msubsup> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>σ</mi> <mrow><mo>(</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msubsup><mi>x</mi> <mn>1</mn> <mi>i</mi></msubsup> <mo>+</mo> <mo>⋯</mo> <mo>+</mo> <msub><mi>ω</mi> <mi>n</mi></msub> <msubsup><mi>x</mi> <mi>n</mi> <mi>i</mi></msubsup> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>+</mo> <mrow><mo>(</mo> <mn>1</mn> <mo>-</mo> <msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mi>i</mi></msubsup> <mo>)</mo></mrow> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mn>1</mn> <mo>-</mo> <mi>σ</mi> <mrow><mo>(</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>1</mn></msub> <msubsup><mi>x</mi> <mn>1</mn> <mi>i</mi></msubsup> <mo>+</mo> <mo>⋯</mo> <mo>+</mo> <msub><mi>ω</mi> <mi>n</mi></msub> <msubsup><mi>x</mi> <mi>n</mi> <mi>i</mi></msubsup> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>,</mo></mrow></math>

使用

<math alttext="dollar-sign c o s t equals Start 2 By 2 Matrix 1st Row 1st Column Blank 2nd Column minus log left-parenthesis sigma left-parenthesis s right-parenthesis right-parenthesis if y Subscript t r u e Baseline equals 1 2nd Row 1st Column Blank 2nd Column minus log left-parenthesis 1 minus sigma left-parenthesis s right-parenthesis right-parenthesis if y Subscript t r u e Baseline equals 0 EndMatrix equals minus y Subscript t r u e Baseline log left-parenthesis sigma left-parenthesis s right-parenthesis right-parenthesis minus left-parenthesis 1 minus y Subscript t r u e Baseline right-parenthesis log left-parenthesis 1 minus sigma left-parenthesis s right-parenthesis right-parenthesis dollar-sign"><mrow><mi>c</mi> <mi>o</mi> <mi>s</mi> <mi>t</mi> <mo>=</mo> <mfenced close="}" open="{" separators=""><mtable displaystyle="true"><mtr><mtd columnalign="left"><mrow><mo>-</mo> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>σ</mi> <mrow><mo>(</mo> <mi>s</mi> <mo>)</mo></mrow> <mo>)</mo></mrow> <mtext>if</mtext> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>=</mo> <mn>1</mn></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>-</mo> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mn>1</mn> <mo>-</mo> <mi>σ</mi> <mrow><mo>(</mo> <mi>s</mi> <mo>)</mo></mrow> <mo>)</mo></mrow> <mtext>if</mtext> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>=</mo> <mn>0</mn></mrow></mtd></mtr></mtable></mfenced> <mo>=</mo> <mo>-</mo> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>σ</mi> <mrow><mo>(</mo> <mi>s</mi> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>-</mo> <mrow><mo>(</mo> <mn>1</mn> <mo>-</mo> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mo>)</mo></mrow> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mn>1</mn> <mo>-</mo> <mi>σ</mi> <mrow><mo>(</mo> <mi>s</mi> <mo>)</mo></mrow> <mo>)</mo></mrow></mrow></math>

现在我们将相同的逻辑推广到多个类别。让我们使用符号 <math alttext="y Subscript t r u e comma i Baseline equals 1"><mrow><msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi><mo>,</mo><mi>i</mi></mrow></msub> <mo>=</mo> <mn>1</mn></mrow></math> 来表示，如果某个数据点属于第i类，则为1，否则为零。然后我们有与将某个数据点误分类相关的成本：

<math alttext="dollar-sign c o s t equals Start 5 By 2 Matrix 1st Row 1st Column Blank 2nd Column minus log left-parenthesis sigma left-parenthesis s Superscript 1 Baseline right-parenthesis right-parenthesis if y Subscript t r u e comma 1 Baseline equals 1 2nd Row 1st Column Blank 2nd Column minus log left-parenthesis sigma left-parenthesis s squared right-parenthesis right-parenthesis if y Subscript t r u e comma 2 Baseline equals 1 3rd Row 1st Column Blank 2nd Column minus log left-parenthesis sigma left-parenthesis s cubed right-parenthesis right-parenthesis if y Subscript t r u e comma 3 Baseline equals 1 4th Row 1st Column  ellipsis 5th Row 1st Column Blank 2nd Column minus log left-parenthesis sigma left-parenthesis s Superscript k Baseline right-parenthesis right-parenthesis if y Subscript t r u e comma k Baseline equals 1 EndMatrix equals minus y Subscript t r u e comma 1 Baseline log left-parenthesis sigma left-parenthesis s Superscript 1 Baseline right-parenthesis right-parenthesis minus ellipsis minus y Subscript t r u e comma k Baseline log left-parenthesis sigma left-parenthesis s Superscript k Baseline right-parenthesis right-parenthesis period dollar-sign"><mrow><mi>c</mi> <mi>o</mi> <mi>s</mi> <mi>t</mi> <mo>=</mo> <mfenced close="}" open="{" separators=""><mtable displaystyle="true"><mtr><mtd columnalign="left"><mrow><mo>-</mo> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>σ</mi> <mrow><mo>(</mo> <msup><mi>s</mi> <mn>1</mn></msup> <mo>)</mo></mrow> <mo>)</mo></mrow> <mtext>if</mtext> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi><mo>,</mo><mn>1</mn></mrow></msub> <mo>=</mo> <mn>1</mn></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>-</mo> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>σ</mi> <mrow><mo>(</mo> <msup><mi>s</mi> <mn>2</mn></msup> <mo>)</mo></mrow> <mo>)</mo></mrow> <mtext>if</mtext> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi><mo>,</mo><mn>2</mn></mrow></msub> <mo>=</mo> <mn>1</mn></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>-</mo> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>σ</mi> <mrow><mo>(</mo> <msup><mi>s</mi> <mn>3</mn></msup> <mo>)</mo></mrow> <mo>)</mo></mrow> <mtext>if</mtext> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi><mo>,</mo><mn>3</mn></mrow></msub> <mo>=</mo> <mn>1</mn></mrow></mtd></mtr> <mtr><mtd columnalign="right"><mo>⋮</mo></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>-</mo> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>σ</mi> <mrow><mo>(</mo> <msup><mi>s</mi> <mi>k</mi></msup> <mo>)</mo></mrow> <mo>)</mo></mrow> <mtext>if</mtext> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi><mo>,</mo><mi>k</mi></mrow></msub> <mo>=</mo> <mn>1</mn></mrow></mtd></mtr></mtable></mfenced> <mo>=</mo> <mo>-</mo> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi><mo>,</mo><mn>1</mn></mrow></msub> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>σ</mi> <mrow><mo>(</mo> <msup><mi>s</mi> <mn>1</mn></msup> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>-</mo> <mo>⋯</mo> <mo>-</mo> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi><mo>,</mo><mi>k</mi></mrow></msub> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>σ</mi> <mrow><mo>(</mo> <msup><mi>s</mi> <mi>k</mi></msup> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>.</mo></mrow></math>

在训练集中对所有*m*个数据点进行平均，我们得到*广义交叉熵损失函数*，将交叉熵损失函数从只有两类的情况推广到多类的情况：

<math alttext="dollar-sign upper L left-parenthesis ModifyingAbove omega With right-arrow right-parenthesis equals minus StartFraction 1 Over m EndFraction sigma-summation Underscript i equals 1 Overscript m Endscripts y Subscript t r u e comma 1 Superscript i Baseline log left-parenthesis sigma left-parenthesis omega 0 Superscript 1 Baseline plus omega 1 Superscript 1 Baseline x 1 Superscript i Baseline plus ellipsis plus omega Subscript n Superscript 1 Baseline x Subscript n Superscript i Baseline right-parenthesis right-parenthesis plus y Subscript t r u e comma 2 Superscript i Baseline log left-parenthesis sigma left-parenthesis omega 0 squared plus omega 1 squared x 1 Superscript i Baseline plus ellipsis plus omega Subscript n Superscript 2 Baseline x Subscript n Superscript i Baseline right-parenthesis right-parenthesis plus ellipsis plus y Subscript t r u e comma k Superscript i Baseline log left-parenthesis sigma left-parenthesis omega 0 Superscript k Baseline plus omega 1 Superscript k Baseline x 1 Superscript i Baseline plus ellipsis plus omega Subscript n Superscript k Baseline x Subscript n Superscript i Baseline right-parenthesis right-parenthesis dollar-sign"><mrow><mi>L</mi> <mrow><mo>(</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>)</mo></mrow> <mo>=</mo> <mo>-</mo> <mfrac><mn>1</mn> <mi>m</mi></mfrac> <msubsup><mo>∑</mo> <mrow><mi>i</mi><mo>=</mo><mn>1</mn></mrow> <mi>m</mi></msubsup> <msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi><mo>,</mo><mn>1</mn></mrow> <mi>i</mi></msubsup> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>σ</mi> <mrow><mo>(</mo> <msubsup><mi>ω</mi> <mn>0</mn> <mn>1</mn></msubsup> <mo>+</mo> <msubsup><mi>ω</mi> <mn>1</mn> <mn>1</mn></msubsup> <msubsup><mi>x</mi> <mn>1</mn> <mi>i</mi></msubsup> <mo>+</mo> <mo>⋯</mo> <mo>+</mo> <msubsup><mi>ω</mi> <mi>n</mi> <mn>1</mn></msubsup> <msubsup><mi>x</mi> <mi>n</mi> <mi>i</mi></msubsup> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>+</mo> <msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi><mo>,</mo><mn>2</mn></mrow> <mi>i</mi></msubsup> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>σ</mi> <mrow><mo>(</mo> <msubsup><mi>ω</mi> <mn>0</mn> <mn>2</mn></msubsup> <mo>+</mo> <msubsup><mi>ω</mi> <mn>1</mn> <mn>2</mn></msubsup> <msubsup><mi>x</mi> <mn>1</mn> <mi>i</mi></msubsup> <mo>+</mo> <mo>⋯</mo> <mo>+</mo> <msubsup><mi>ω</mi> <mi>n</mi> <mn>2</mn></msubsup> <msubsup><mi>x</mi> <mi>n</mi> <mi>i</mi></msubsup> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>+</mo> <mo>⋯</mo> <mo>+</mo> <msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi><mo>,</mo><mi>k</mi></mrow> <mi>i</mi></msubsup> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>σ</mi> <mrow><mo>(</mo> <msubsup><mi>ω</mi> <mn>0</mn> <mi>k</mi></msubsup> <mo>+</mo> <msubsup><mi>ω</mi> <mn>1</mn> <mi>k</mi></msubsup> <msubsup><mi>x</mi> <mn>1</mn> <mi>i</mi></msubsup> <mo>+</mo> <mo>⋯</mo> <mo>+</mo> <msubsup><mi>ω</mi> <mi>n</mi> <mi>k</mi></msubsup> <msubsup><mi>x</mi> <mi>n</mi> <mi>i</mi></msubsup> <mo>)</mo></mrow> <mo>)</mo></mrow></mrow></math>

## 优化

现在我们有了损失函数的公式，我们可以搜索它的最小化<math alttext="omega"><mi>ω</mi></math>。由于我们将遇到的大多数损失函数都没有关于训练集和它们的目标标签的最小化的显式公式，因此我们将使用数值方法来寻找最小化值，特别是：下一章的梯度下降、随机梯度下降或小批量梯度下降。同样，广义交叉熵损失函数的凸性有助于我们在最小化过程中，因此我们保证能找到我们寻求的<math alttext="omega"><mi>ω</mi></math>。

# 关于交叉熵和信息论的说明

交叉熵概念源自信息论。我们将在本章后面讨论决策树时详细说明。现在，请记住以下数量，其中*p*是事件发生的概率：

<math alttext="dollar-sign log left-parenthesis StartFraction 1 Over p EndFraction right-parenthesis equals minus log left-parenthesis p right-parenthesis dollar-sign"><mrow><mo form="prefix">log</mo> <mrow><mo>(</mo> <mfrac><mn>1</mn> <mi>p</mi></mfrac> <mo>)</mo></mrow> <mo>=</mo> <mo>-</mo> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>p</mi> <mo>)</mo></mrow></mrow></math>

当*p*较小时，上述数量较大，因此，它量化了*不太可能*事件的更大*惊喜*。

# 关于逻辑函数和softmax函数以及统计力学的说明

如果您熟悉统计力学，您可能已经注意到逻辑函数和softmax函数以与统计力学领域的*配分函数*相同的方式计算概率，计算系统处于某种状态的概率。

# 将上述模型纳入神经网络的最后一层

线性回归模型通过适当地线性组合数据特征进行预测，然后加入偏差。逻辑回归和softmax回归模型通过适当地线性组合数据特征，加入偏差，然后将结果传递到概率评分函数中进行分类。在这些简单模型中，数据的特征仅被线性组合，因此，这些模型在捕捉数据特征之间潜在重要的非线性交互方面较弱。神经网络模型将非线性的*激活函数*纳入其训练函数中，并在多个层次上进行，因此更适合检测非线性和更复杂的关系。神经网络的最后一层是其输出层。倒数第二层将一些高阶特征输出并输入到最后一层。如果我们希望网络将数据分类为多个类别，那么我们可以将最后一层设为softmax层；如果我们希望将其分类为两个类别，那么我们的最后一层可以是逻辑回归层；如果我们希望网络预测数值，那么我们可以将其最后一层设为回归层。我们将在[第5章](ch05.xhtml#ch05)中看到这些示例。

# 其他流行的机器学习技术和技术集成

在回归和逻辑回归之后，重要的是要涉足机器学习社区，并学习一些最流行的分类和回归任务技术背后的思想。*支持向量机*、*决策树*和*随机森林*非常强大和流行，能够执行分类和回归任务。自然的问题是，我们何时使用特定的机器学习方法，包括线性和逻辑回归，以及后来的神经网络？我们如何知道使用哪种方法并基于我们的结论和预测？这些是数学分析机器学习模型的类型的问题。

由于对每种方法的数学分析，包括它通常最适合的数据集类型，现在才开始受到严肃关注，这是在最近增加了对人工智能、机器学习和数据科学研究的资源分配之后。目前的做法是在同一数据集上尝试每种方法，并使用结果最好的那种。也就是说，假设我们有必要的计算和时间资源来尝试不同的机器学习技术。更好的是，如果你有时间和资源来训练各种机器学习模型（并行计算在这里非常完美），那么*集成方法*会将它们的结果组合起来，无论是通过平均还是投票，讽刺的是，数学上是合理的，会比最好的个体表现者取得更好的结果，*甚至当最好的表现者是弱表现者时也是如此！*

一个集成的例子是随机森林：它是一组决策树的集成。

当我们基于集成进行预测时，行业术语如*bagging*（或*bootstrap aggregating*）、*pasting*、*boosting*（比如*ADA boost*和*Gradient boosting*）、*stacking*和*random patches*会出现。Bagging和pasting在训练集的不同随机子集上训练*相同*的机器学习模型。Bagging使用替换从训练集中抽取实例，而pasting则不使用替换。*Random patches*也从特征空间中抽样，每次在随机特征子集上训练机器学习模型。当数据集具有许多特征时，比如图像（其中每个像素都是一个特征），这是非常有帮助的。*Stacking*学习集成的预测机制，而不是简单的投票或平均值。

## 支持向量机

支持向量机是一种极其流行的机器学习方法，能够在具有线性（平面）和非线性（曲线）决策边界的情况下执行分类和回归任务。

对于分类，该方法寻求使用尽可能宽的间隔来分离标记数据，从而产生最佳的分隔*高速公路*，而不是薄薄的分隔线。让我们解释一下支持向量机如何在本章的训练函数、损失函数和优化的结构上对标记数据实例进行分类。

训练函数

我们再次使用未知权重<math alttext="omega"><mi>ω</mi></math>的数据点的特征进行线性组合，并添加偏差<math alttext="omega 0"><msub><mi>ω</mi> <mn>0</mn></msub></math>。然后通过*sign*函数得到答案：如果特征的线性组合加上偏差是一个正数，返回1（或分类为第一类），如果是负数，返回-1（或分类为另一类）。因此，训练函数的公式变为：

<math alttext="dollar-sign f left-parenthesis ModifyingAbove omega With right-arrow semicolon ModifyingAbove x With right-arrow right-parenthesis equals s i g n left-parenthesis ModifyingAbove omega With right-arrow Superscript t Baseline ModifyingAbove x With right-arrow plus omega 0 right-parenthesis dollar-sign"><mrow><mi>f</mi> <mrow><mo>(</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>;</mo> <mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mo>)</mo></mrow> <mo>=</mo> <mi>s</mi> <mi>i</mi> <mi>g</mi> <mi>n</mi> <mrow><mo>(</mo> <msup><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mo>+</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>)</mo></mrow></mrow></math>

损失函数

我们必须设计一个损失函数，惩罚错误分类的点。对于逻辑回归，我们使用交叉熵损失函数。对于支持向量机，我们的损失函数基于一个称为*hinge loss function*的函数：

<math alttext="dollar-sign max left-parenthesis 0 comma 1 minus y Subscript t r u e Baseline left-parenthesis ModifyingAbove omega With right-arrow Superscript t Baseline ModifyingAbove x With right-arrow plus omega 0 right-parenthesis right-parenthesis period dollar-sign"><mrow><mo form="prefix" movablelimits="true">max</mo> <mo>(</mo> <mn>0</mn> <mo>,</mo> <mn>1</mn> <mo>-</mo> <msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub> <mrow><mo>(</mo> <msup><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mo>+</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>)</mo></mrow> <mo>)</mo> <mo>.</mo></mrow></math>

让我们看看铰链损失函数如何惩罚分类错误。首先，回想一下<math alttext="y Subscript t r u e"><msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></math>是1或-1，取决于数据点是属于正类还是负类。

+   如果对于某个数据点<math alttext="y Subscript t r u e"><msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></math>为1，但<math alttext="ModifyingAbove omega With right-arrow Superscript t Baseline ModifyingAbove x With right-arrow plus omega 0 less-than 0"><mrow><msup><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mo>+</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo><</mo> <mn>0</mn></mrow></math>，训练函数将错误分类，并给出<math alttext="y Subscript p r e d i c t Baseline equals negative 1"><mrow><msub><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>=</mo> <mo>-</mo> <mn>1</mn></mrow></math>，而铰链损失函数的值将为<math alttext="1 minus left-parenthesis 1 right-parenthesis left-parenthesis ModifyingAbove omega With right-arrow Superscript t Baseline ModifyingAbove x With right-arrow plus omega 0 right-parenthesis greater-than 1"><mrow><mn>1</mn> <mo>-</mo> <mrow><mo>(</mo> <mn>1</mn> <mo>)</mo></mrow> <mrow><mo>(</mo> <msup><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mo>+</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>)</mo></mrow> <mo>></mo> <mn>1</mn></mrow></math>，这是一个高惩罚，当你的目标是最小化时。

+   另一方面，如果<math alttext="y Subscript t r u e"><msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></math>为1且<math alttext="ModifyingAbove omega With right-arrow Superscript t Baseline ModifyingAbove x With right-arrow plus omega 0 less-than 0"><mrow><msup><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mo>+</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo><</mo> <mn>0</mn></mrow></math>，训练函数将正确分类它并给出<math alttext="y Subscript p r e d i c t Baseline equals 1"><mrow><msub><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>=</mo> <mn>1</mn></mrow></math>。然而，铰链损失函数设计成这样，即使<math alttext="ModifyingAbove omega With right-arrow Superscript t Baseline ModifyingAbove x With right-arrow plus omega 0 less-than 1"><mrow><msup><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mo>+</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo><</mo> <mn>1</mn></mrow></math>，它仍会对我们进行惩罚，其值将为<math alttext="1 minus left-parenthesis 1 right-parenthesis left-parenthesis ModifyingAbove omega With right-arrow Superscript t Baseline ModifyingAbove x With right-arrow plus omega 0 right-parenthesis"><mrow><mn>1</mn> <mo>-</mo> <mrow><mo>(</mo> <mn>1</mn> <mo>)</mo></mrow> <mrow><mo>(</mo> <msup><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mo>+</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>)</mo></mrow></mrow></math>，现在小于1但仍大于零。

+   只有当<math alttext="y Subscript t r u e"><msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></math>为1且<math alttext="ModifyingAbove omega With right-arrow Superscript t Baseline ModifyingAbove x With right-arrow plus omega 0 less-than 1"><mrow><msup><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mo>+</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo><</mo> <mn>1</mn></mrow></math>（训练函数仍然会正确分类这一点，并给出<math alttext="y Subscript p r e d i c t Baseline equals 1"><mrow><msub><mi>y</mi> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi><mi>i</mi><mi>c</mi><mi>t</mi></mrow></msub> <mo>=</mo> <mn>1</mn></mrow></math>），则铰链损失函数值将为零，因为它将是零和负量之间的最大值。

+   当<math alttext="y Subscript t r u e"><msub><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow></msub></math>为-1时，相同的逻辑适用：铰链损失函数会对错误的预测进行严厉惩罚，对正确的预测进行轻微惩罚，但如果它与“零除数”（大于1的边距）的距离不够远，它将进行惩罚，并且仅当预测正确且该点距离“零除数”大于1时才返回零。

+   请注意，零分隔器的方程式为<math alttext="ModifyingAbove omega With right-arrow Superscript t Baseline ModifyingAbove x With right-arrow plus omega 0 equals 0"><mrow><msup><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mo>+</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>=</mo> <mn>0</mn></mrow></math>，边缘边缘的方程式为<math alttext="ModifyingAbove omega With right-arrow Superscript t Baseline ModifyingAbove x With right-arrow plus omega 0 equals negative 1"><mrow><msup><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mo>+</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>=</mo> <mo>-</mo> <mn>1</mn></mrow></math>和<math alttext="ModifyingAbove omega With right-arrow Superscript t Baseline ModifyingAbove x With right-arrow plus omega 0 equals 1"><mrow><msup><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mo>+</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>=</mo> <mn>1</mn></mrow></math>。边缘边缘之间的距离很容易计算为<math alttext="StartFraction 2 Over parallel-to omega parallel-to EndFraction"><mfrac><mn>2</mn> <msub><mrow><mo>∥</mo><mi>ω</mi><mo>∥</mo></mrow> <mn>2</mn></msub></mfrac></math>。因此，如果我们想要增加这个边缘宽度，我们必须减少<math alttext="parallel-to omega parallel-to"><msub><mrow><mo>∥</mo><mi>ω</mi><mo>∥</mo></mrow> <mn>2</mn></msub></math>，因此，这个术语必须进入损失函数，以及hingle损失函数，惩罚误分类的点和边缘边界内的点。

现在，如果我们将所有m个数据点在训练集中的hinge损失平均，并添加<math alttext="parallel-to omega parallel-to"><msubsup><mrow><mo>∥</mo><mi>ω</mi><mo>∥</mo></mrow> <mn>2</mn> <mn>2</mn></msubsup></math>，我们就得到了支持向量机常用的损失函数公式：

<math alttext="dollar-sign upper L left-parenthesis ModifyingAbove omega With right-arrow right-parenthesis equals StartFraction 1 Over m EndFraction sigma-summation Underscript i equals 1 Overscript m Endscripts max left-parenthesis 0 comma 1 minus y Subscript t r u e Superscript i Baseline left-parenthesis ModifyingAbove omega With right-arrow Superscript t Baseline ModifyingAbove x With right-arrow Superscript i Baseline plus omega 0 right-parenthesis right-parenthesis plus lamda parallel-to omega parallel-to dollar-sign"><mrow><mi>L</mi> <mrow><mo>(</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>)</mo></mrow> <mo>=</mo> <mfrac><mn>1</mn> <mi>m</mi></mfrac> <msubsup><mo>∑</mo> <mrow><mi>i</mi><mo>=</mo><mn>1</mn></mrow> <mi>m</mi></msubsup> <mo form="prefix" movablelimits="true">max</mo> <mrow><mo>(</mo> <mn>0</mn> <mo>,</mo> <mn>1</mn> <mo>-</mo> <msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mi>i</mi></msubsup> <mrow><mo>(</mo> <msup><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mi>t</mi></msup> <msup><mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mi>i</mi></msup> <mo>+</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>+</mo> <mi>λ</mi> <msubsup><mrow><mo>∥</mo><mi>ω</mi><mo>∥</mo></mrow> <mn>2</mn> <mn>2</mn></msubsup></mrow></math>

优化

我们现在的目标是寻找最小化损失函数的<math alttext="ModifyingAbove w With right-arrow"><mover accent="true"><mi>w</mi> <mo>→</mo></mover></math>。让我们观察这个损失函数一分钟：

+   它有两个项：<math alttext="StartFraction 1 Over m EndFraction sigma-summation Underscript i equals 1 Overscript m Endscripts max left-parenthesis 0 comma 1 minus y Subscript t r u e Superscript i Baseline left-parenthesis ModifyingAbove omega With right-arrow Superscript t Baseline ModifyingAbove x With right-arrow Superscript i Baseline plus omega 0 right-parenthesis right-parenthesis"><mrow><mfrac><mn>1</mn> <mi>m</mi></mfrac> <msubsup><mo>∑</mo> <mrow><mi>i</mi><mo>=</mo><mn>1</mn></mrow> <mi>m</mi></msubsup> <mo form="prefix" movablelimits="true">max</mo> <mrow><mo>(</mo> <mn>0</mn> <mo>,</mo> <mn>1</mn> <mo>-</mo> <msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mi>i</mi></msubsup> <mrow><mo>(</mo> <msup><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mi>t</mi></msup> <msup><mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mi>i</mi></msup> <mo>+</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>)</mo></mrow> <mo>)</mo></mrow></math>和<math alttext="lamda parallel-to ModifyingAbove omega With right-arrow parallel-to"><mrow><mrow><mi>λ</mi> <mo>∥</mo></mrow> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <msubsup><mrow><mo>∥</mo></mrow> <mn>2</mn> <mn>2</mn></msubsup></mrow></math>。每当我们在优化问题中有多个项时，很可能它们是竞争项，也就是说，使第一项小且快乐的相同<math alttext="omega"><mi>ω</mi></math>值可能会使第二项大且悲伤。因此，在寻找优化它们的和的<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>时，它是两个项之间的推拉游戏。

+   与<math alttext="lamda parallel-to ModifyingAbove omega With right-arrow parallel-to"><mrow><mrow><mi>λ</mi> <mo>∥</mo></mrow> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <msubsup><mrow><mo>∥</mo></mrow> <mn>2</mn> <mn>2</mn></msubsup></mrow></math> 项一起出现的<math alttext="lamda"><mi>λ</mi></math>是我们可以在训练过程的验证阶段调整的模型超参数的一个例子。请注意，控制<math alttext="lamda"><mi>λ</mi></math>的值有助于我们控制边缘的宽度：如果我们选择一个较大的<math alttext="lamda"><mi>λ</mi></math>值，优化器将忙于选择具有非常低<math alttext="parallel-to omega parallel-to"><msubsup><mrow><mo>∥</mo><mi>ω</mi><mo>∥</mo></mrow> <mn>2</mn> <mn>2</mn></msubsup></math>的<math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>，以补偿较大的<math alttext="lamda"><mi>λ</mi></math>，并且损失函数的第一项将受到较少的关注。但请记住，较小的<math alttext="parallel-to omega parallel-to"><msub><mrow><mo>∥</mo><mi>ω</mi><mo>∥</mo></mrow> <mn>2</mn></msub></math>意味着更大的边缘！

+   <math alttext="lamda parallel-to ModifyingAbove omega With right-arrow parallel-to"><mrow><mrow><mi>λ</mi> <mo>∥</mo></mrow> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <msubsup><mrow><mo>∥</mo></mrow> <mn>2</mn> <mn>2</mn></msubsup></mrow></math> 项也可以被视为*正则化项*，我们将在下一章中讨论。

+   这个损失函数是凸的，并且下界为零，所以它的最小化问题并不太糟糕：我们不必担心陷入局部最小值。第一项具有奇点，但正如我们之前提到的，我们可以在奇异点定义其次梯度，然后应用下降方法。

一些优化问题可以被重新表述，而不是解决原始的*原始*问题，我们最终解决它的*对偶*问题！通常，一个问题比另一个更容易解决。我们可以将对偶问题看作是原始问题的平行宇宙中的另一个优化问题。这些宇宙在优化器处相遇。因此，解决一个问题会自动给出另一个问题的解。我们在研究优化时研究对偶性。特别感兴趣和应用广泛的领域是线性和二次优化，也称为线性和二次规划。我们目前有的最小化问题：

<math alttext="dollar-sign min Underscript ModifyingAbove omega With right-arrow Endscripts StartFraction 1 Over m EndFraction sigma-summation Underscript i equals 1 Overscript m Endscripts max left-parenthesis 0 comma 1 minus y Subscript t r u e Superscript i Baseline left-parenthesis ModifyingAbove omega With right-arrow Superscript t Baseline ModifyingAbove x With right-arrow Superscript i Baseline plus omega 0 right-parenthesis right-parenthesis plus lamda parallel-to omega parallel-to dollar-sign"><mrow><msub><mo form="prefix" movablelimits="true">min</mo> <mover accent="true"><mi>ω</mi> <mo>→</mo></mover></msub> <mfrac><mn>1</mn> <mi>m</mi></mfrac> <msubsup><mo>∑</mo> <mrow><mi>i</mi><mo>=</mo><mn>1</mn></mrow> <mi>m</mi></msubsup> <mo form="prefix" movablelimits="true">max</mo> <mrow><mo>(</mo> <mn>0</mn> <mo>,</mo> <mn>1</mn> <mo>-</mo> <msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mi>i</mi></msubsup> <mrow><mo>(</mo> <msup><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mi>t</mi></msup> <msup><mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mi>i</mi></msup> <mo>+</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>+</mo> <mi>λ</mi> <msubsup><mrow><mo>∥</mo><mi>ω</mi><mo>∥</mo></mrow> <mn>2</mn> <mn>2</mn></msubsup></mrow></math>

这是二次规划的一个例子，它有一个对偶问题的公式，结果比原问题更容易优化（特别是当特征数量很高时）：

<math alttext="dollar-sign max Underscript ModifyingAbove alpha With right-arrow Endscripts sigma-summation Underscript j equals 1 Overscript m Endscripts alpha Subscript j minus one-half sigma-summation Underscript j equals 1 Overscript m Endscripts sigma-summation Underscript k equals 1 Overscript m Endscripts alpha Subscript j Baseline alpha Subscript k Baseline y Subscript t r u e Superscript j Baseline y Subscript t r u e Superscript k Baseline left-parenthesis left-parenthesis ModifyingAbove x With right-arrow Superscript j Baseline right-parenthesis Superscript t Baseline ModifyingAbove x With right-arrow Superscript k Baseline right-parenthesis dollar-sign"><mrow><msub><mo form="prefix" movablelimits="true">max</mo> <mover accent="true"><mi>α</mi> <mo>→</mo></mover></msub> <msubsup><mo>∑</mo> <mrow><mi>j</mi><mo>=</mo><mn>1</mn></mrow> <mi>m</mi></msubsup> <msub><mi>α</mi> <mi>j</mi></msub> <mo>-</mo> <mfrac><mn>1</mn> <mn>2</mn></mfrac> <msubsup><mo>∑</mo> <mrow><mi>j</mi><mo>=</mo><mn>1</mn></mrow> <mi>m</mi></msubsup> <msubsup><mo>∑</mo> <mrow><mi>k</mi><mo>=</mo><mn>1</mn></mrow> <mi>m</mi></msubsup> <msub><mi>α</mi> <mi>j</mi></msub> <msub><mi>α</mi> <mi>k</mi></msub> <msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mi>j</mi></msubsup> <msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mi>k</mi></msubsup> <mrow><mo>(</mo> <msup><mrow><mo>(</mo><msup><mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mi>j</mi></msup> <mo>)</mo></mrow> <mi>t</mi></msup> <msup><mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mi>k</mi></msup> <mo>)</mo></mrow></mrow></math>

受到约束条件 <math alttext="alpha Subscript j Baseline greater-than-or-equal-to 0"><mrow><msub><mi>α</mi> <mi>j</mi></msub> <mo>≥</mo> <mn>0</mn></mrow></math> 和 <math alttext="sigma-summation Underscript j equals 1 Overscript m Endscripts alpha Subscript j Baseline y Subscript t r u e Superscript j Baseline equals 0"><mrow><msubsup><mo>∑</mo> <mrow><mi>j</mi><mo>=</mo><mn>1</mn></mrow> <mi>m</mi></msubsup> <msub><mi>α</mi> <mi>j</mi></msub> <msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mi>j</mi></msubsup> <mo>=</mo> <mn>0</mn></mrow></math> 。当我们学习原始问题和对偶问题时，编写上述公式通常是直接的，因此我们跳过推导，以免打断我们的流程。

二次规划是一个非常成熟的领域，有许多软件包可以解决这个问题。一旦我们找到最大化的 <math alttext="ModifyingAbove alpha With right-arrow"><mover accent="true"><mi>α</mi> <mo>→</mo></mover></math>，我们可以找到向量 <math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>，使用 <math alttext="ModifyingAbove omega With right-arrow equals sigma-summation Underscript j equals 1 Overscript m Endscripts alpha Subscript j Baseline y Subscript t r u e Superscript i Baseline ModifyingAbove x With right-arrow Superscript j"><mrow><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mo>=</mo> <msubsup><mo>∑</mo> <mrow><mi>j</mi><mo>=</mo><mn>1</mn></mrow> <mi>m</mi></msubsup> <msub><mi>α</mi> <mi>j</mi></msub> <msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mi>i</mi></msubsup> <msup><mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mi>j</mi></msup></mrow></math> 最小化原始问题。一旦我们有了 <math alttext="ModifyingAbove omega With right-arrow"><mover accent="true"><mi>ω</mi> <mo>→</mo></mover></math>，我们可以使用我们现在训练过的函数对新数据点进行分类：

<math alttext="dollar-sign StartLayout 1st Row 1st Column f left-parenthesis ModifyingAbove x With right-arrow Subscript n e w Baseline right-parenthesis 2nd Column equals s i g n left-parenthesis ModifyingAbove omega With right-arrow Superscript t Baseline ModifyingAbove x With right-arrow Subscript n e w Baseline plus omega 0 right-parenthesis 2nd Row 1st Column Blank 2nd Column equals s i g n left-parenthesis sigma-summation Underscript j Endscripts alpha Subscript j Baseline y Superscript i Baseline left-parenthesis ModifyingAbove x With right-arrow Superscript j Baseline right-parenthesis Superscript t Baseline ModifyingAbove x With right-arrow Subscript n e w Baseline plus omega 0 right-parenthesis period EndLayout dollar-sign"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi>f</mi> <mo>(</mo> <msub><mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mrow><mi>n</mi><mi>e</mi><mi>w</mi></mrow></msub> <mo>)</mo></mrow></mtd> <mtd columnalign="left"><mrow><mo>=</mo> <mi>s</mi> <mi>i</mi> <mi>g</mi> <mi>n</mi> <mo>(</mo> <msup><mover accent="true"><mi>ω</mi> <mo>→</mo></mover> <mi>t</mi></msup> <msub><mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mrow><mi>n</mi><mi>e</mi><mi>w</mi></mrow></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>=</mo> <mi>s</mi> <mi>i</mi> <mi>g</mi> <mi>n</mi> <mo>(</mo> <munder><mo>∑</mo> <mi>j</mi></munder> <msub><mi>α</mi> <mi>j</mi></msub> <msup><mi>y</mi> <mi>i</mi></msup> <msup><mrow><mo>(</mo><msup><mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mi>j</mi></msup> <mo>)</mo></mrow> <mi>t</mi></msup> <msub><mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mrow><mi>n</mi><mi>e</mi><mi>w</mi></mrow></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>0</mn></msub> <mo>)</mo> <mo>.</mo></mrow></mtd></mtr></mtable></math>

如果你想避免二次规划，还有另一种非常快速的方法叫做*坐标下降*，它解决了对偶问题，并且在具有大量特征的大数据集上表现非常好。

# 核技巧：我们可以将相同的思想应用于非线性分类

关于对偶问题的一个非常重要的注意事项：数据点只出现在一对中，更具体地说，只出现在一个标量积中，即<math alttext="left-parenthesis ModifyingAbove x With right-arrow Superscript j Baseline right-parenthesis Superscript t Baseline ModifyingAbove x With right-arrow Superscript k">。同样，它们只在训练函数中作为标量积出现。这个简单的观察可以产生魔法：

+   如果我们找到一个函数<math alttext="upper K left-parenthesis ModifyingAbove x With right-arrow Superscript j Baseline comma ModifyingAbove x With right-arrow Superscript j Baseline right-parenthesis">，可以应用于数据点对，并且恰好给出我们转换后的数据点对的标量积到某个更高维度的空间（而不知道实际的转换是什么），那么我们可以通过用<math alttext="upper K left-parenthesis ModifyingAbove x With right-arrow Superscript j Baseline comma ModifyingAbove x With right-arrow Superscript j Baseline right-parenthesis">替换对偶问题公式中的标量积，在更高维度空间中解决完全相同的对偶问题。

+   这里的直觉是，在低维度中非线性可分的数据几乎总是在高维度中线性可分。因此，将所有数据点转换到更高的维度，然后进行分离。核技巧解决了在更高维度中的线性分类问题，*而不需要*转换每个点。核本身评估了转换数据的点积，而不需要转换数据。非常酷的东西。

核函数的示例包括：

+   <math alttext="upper K left-parenthesis ModifyingAbove x With right-arrow Superscript j Baseline comma ModifyingAbove x With right-arrow Superscript j Baseline right-parenthesis equals left-parenthesis left-parenthesis ModifyingAbove x With right-arrow Superscript j Baseline right-parenthesis Superscript t Baseline ModifyingAbove x With right-arrow Superscript j Baseline right-parenthesis squared">。

+   多项式核：<math alttext="upper K left-parenthesis ModifyingAbove x With right-arrow Superscript j Baseline comma ModifyingAbove x With right-arrow Superscript j Baseline right-parenthesis equals left-parenthesis 1 plus left-parenthesis ModifyingAbove x With right-arrow Superscript j Baseline right-parenthesis Superscript t Baseline ModifyingAbove x With right-arrow Superscript j Baseline right-parenthesis Superscript d"><mrow><mi>K</mi> <mrow><mo>(</mo> <msup><mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mi>j</mi></msup> <mo>,</mo> <msup><mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mi>j</mi></msup> <mo>)</mo></mrow> <mo>=</mo> <msup><mrow><mo>(</mo><mn>1</mn><mo>+</mo><msup><mrow><mo>(</mo><msup><mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mi>j</mi></msup> <mo>)</mo></mrow> <mi>t</mi></msup> <msup><mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mi>j</mi></msup> <mo>)</mo></mrow> <mi>d</mi></msup></mrow></math> .

+   高斯核：<math alttext="upper K left-parenthesis ModifyingAbove x With right-arrow Superscript j Baseline comma ModifyingAbove x With right-arrow Superscript j Baseline right-parenthesis equals e Superscript minus gamma StartAbsoluteValue x Super Subscript j Superscript minus x Super Subscript k Superscript EndAbsoluteValue squared"><mrow><mi>K</mi> <mrow><mo>(</mo> <msup><mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mi>j</mi></msup> <mo>,</mo> <msup><mover accent="true"><mi>x</mi> <mo>→</mo></mover> <mi>j</mi></msup> <mo>)</mo></mrow> <mo>=</mo> <msup><mi>e</mi> <mrow><mrow><mo>-</mo><mi>γ</mi><mo>|</mo></mrow><msub><mi>x</mi> <mi>j</mi></msub> <mo>-</mo><msub><mi>x</mi> <mi>k</mi></msub> <msup><mrow><mo>|</mo></mrow> <mn>2</mn></msup></mrow></msup></mrow></math> .

## 决策树

本章的驱动主题是一切都是一个函数，决策树本质上是一个以布尔变量作为输入的函数（这些变量只能假定为*true*（或1）或*false*（或0）值），例如：特征>5，特征=晴天，特征=男性，*等*，并输出*决策*，例如：批准贷款，分类为covid19，返回25，*等*。我们使用逻辑*或*、*和*和*非*运算符，而不是添加或乘以布尔变量。

但是，如果我们的特征在原始数据集中不是布尔变量怎么办？那么我们必须在将它们馈送到模型进行预测之前将它们转换为布尔变量。例如，[图3-17](#Fig_regression_tree)中的决策树是在Fish Market数据集上训练的。它是一个回归树。该树采用原始数据，但表示树的函数实际上是在新变量上操作的，这些新变量是原始数据特征转换为布尔变量：

1.  a1=(Width <math alttext="小于或等于"><mo>≤</mo></math> 5.117)

1.  a2=(Length3 <math alttext="小于或等于"><mo>≤</mo></math> 59.55)

1.  a3=(Length3 <math alttext="小于或等于"><mo>≤</mo></math> 41.1)

1.  a4=(Length3 <math alttext="小于或等于"><mo>≤</mo></math> 34.9)

1.  a5=(Length3 <math alttext="小于或等于"><mo>≤</mo></math> 27.95)

1.  a6=(Length3 <math alttext="小于或等于"><mo>≤</mo></math> 21.25)

![400](assets/emai_0317.png)

###### 图3-17\. 基于Fish Market数据集构建的回归决策树。有关详细信息，请参阅附带的Jupyter笔记本。

现在，表示[图3-17](#Fig_regression_tree)中决策树的函数是：

$f(a Baseline 1, a Baseline 2, a Baseline 3, a Baseline 4, a Baseline 5, a Baseline 6) = (a Baseline 1 and a Baseline 5 and a Baseline 6) times 39.584 plus (a Baseline 1 and a Baseline 5 and not a Baseline 6) times 139.968 plus (a Baseline 1 and not a Baseline 5 and a Baseline 4) times 287.278 plus (a Baseline 1 and not a Baseline 5 and not a Baseline 4) times 422.769 plus (not a Baseline 1 and a Baseline 2 and a Baseline 3) times 639.737 plus (not a Baseline 1 and a Baseline 2 and not a Baseline 3) times 824.211 plus (not a Baseline 1 and not a Baseline 2) times 1600 dollar-sign$

请注意，与我们迄今在本章中遇到的训练函数不同，上述函数没有我们需要解决的参数ω。这被称为*非参数模型*，它不会提前固定函数的*形状*。这使得它具有与数据*一起增长*或者说适应数据的灵活性。当然，这种对数据的高适应性也带来了过拟合数据的高风险。幸运的是，有办法解决这个问题，我们在这里列出了一些方法，但没有详细说明：在生长后修剪树，限制层数，设置每个节点的最小数据实例数，或者使用一组树而不是一棵树，称为*随机森林*，下面会讨论。

一个非常重要的观察：决策树决定只在原始数据集的两个特征上进行分割，即宽度和长度3特征。决策树的设计方式使得更重要的特征（那些对我们的预测提供最多信息的特征）更接近根部。因此，决策树可以帮助进行特征选择，我们选择最重要的特征来对我们最终模型的预测做出贡献。

难怪宽度和长度3特征最终成为预测鱼重量最重要的特征。[图3-18](#Fig_fish_corr_matrix)中的相关矩阵和[图3-3](#Fig_weight_lengths_scatterplots)中的散点图显示所有长度特征之间存在极强的相关性。这意味着它们提供的信息是冗余的，并且在我们的预测模型中包含所有这些特征将增加计算成本并降低性能。

![280](assets/emai_0318.png)

###### 图3-18. 鱼市场数据集的相关矩阵。所有长度特征之间存在极强的相关性。

# 注：特征选择

我们刚刚介绍了非常重要的特征选择主题。现实世界的数据集包含许多特征，其中一些可能提供冗余信息，其他一些对于预测我们的目标标签来说根本不重要。在机器学习模型中包含无关或冗余的特征会增加计算成本并降低性能。我们刚刚看到决策树是帮助选择重要特征的一种方法。另一种方法是一种称为Lasso回归的正则化技术，我们将在[第4章](ch04.xhtml#ch04)中介绍。还有一些统计测试可以测试特征之间的依赖关系。*F-测试*测试线性依赖关系（这会为相关特征给出更高的分数，但仅仅依靠相关性是具有误导性的），*互信息*测试非线性依赖关系。这些提供了一个*度量*，衡量特征对确定目标标签的贡献程度，并因此有助于特征选择，保留最有前途的特征。我们还可以测试特征之间的依赖关系，以及它们的相关性和散点图。*方差*阈值移除方差很小或没有方差的特征，因为如果一个特征在自身内部变化不大，它的预测能力就很小。

我们如何在数据集上训练决策树？我们优化哪个函数？通常在*生成*决策树时优化两个函数：*熵*和*基尼不纯度*。使用其中一个与另一个并没有太大区别。我们接下来会详细介绍这两个函数。

### 熵和基尼不纯度

在这里，我们决定根据被评估为*最重要*的特征来分割树的节点。*熵*和*基尼不纯度*是衡量特征重要性的两种流行方法。它们在数学上并不等价，但它们都能工作并提供合理的决策树。基尼不纯度通常计算成本较低，因此它是软件包中的默认设置，但您可以选择从默认设置更改为熵。当某些类别的频率远高于其他类别时，使用基尼不纯度往往会产生较不平衡的树。这些类别最终会被孤立在它们自己的分支中。然而，在许多情况下，使用熵或基尼不纯度在生成的决策树中并没有太大的差异。

1.  使用熵方法，我们寻找提供*最大信息增益*的特征分割（我们很快会给出它的公式）。信息增益源自信息理论，它与*熵*的概念有关。而熵又源自热力学和统计物理学，它量化了某个系统中的无序程度。

1.  使用基尼不纯度方法，我们寻找提供子节点最低平均基尼不纯度的特征分割（我们也很快会给出它的公式）。

为了最大化信息增益（或最小化基尼不纯度），生长决策树的算法必须遍历训练数据子集的每个特征，并计算使用该特定特征作为节点进行拆分时所实现的信息增益（或基尼不纯度），然后选择提供最高信息增益（或子节点具有最低平均基尼不纯度）的特征。此外，如果特征具有实际数值，算法必须决定*在节点上提出什么问题*，也就是说，在哪个特征值上进行拆分，例如，<math alttext="x 5 less-than 0.1"><mrow><msub><mi>x</mi> <mn>5</mn></msub> <mo><</mo> <mn>0</mn> <mo>.</mo> <mn>1</mn></mrow></math>？算法必须在树的每一层依次执行此操作，计算每个节点中数据实例的特征的信息增益（或基尼不纯度），有时还要计算每个拆分值的可能性。通过示例更容易理解。但首先，我们写出熵、信息增益和基尼不纯度的公式。

#### 熵和信息增益

理解熵公式最简单的方法是依靠直觉，即如果事件高概率发生，那么与之相关的惊喜就很少。因此，当<math alttext="p left-parenthesis e v e n t right-parenthesis"><mrow><mi>p</mi> <mo>(</mo> <mi>e</mi> <mi>v</mi> <mi>e</mi> <mi>n</mi> <mi>t</mi> <mo>)</mo></mrow></math>很大时，它的惊喜就很低。我们可以用一个随着概率增加而减少的函数来数学上表示这一点。微积分函数<math alttext="log StartFraction 1 Over x EndFraction"><mrow><mo form="prefix">log</mo> <mfrac><mn>1</mn> <mi>x</mi></mfrac></mrow></math>适用，并且具有独立事件的惊喜相加的附加属性。因此，我们可以定义：

<math alttext="dollar-sign upper S u r p r i s e left-parenthesis e v e n t right-parenthesis equals log StartFraction 1 Over p left-parenthesis e v e n t right-parenthesis EndFraction equals minus log left-parenthesis p left-parenthesis e v e n t right-parenthesis right-parenthesis dollar-sign"><mrow><mi>S</mi> <mi>u</mi> <mi>r</mi> <mi>p</mi> <mi>r</mi> <mi>i</mi> <mi>s</mi> <mi>e</mi> <mrow><mo>(</mo> <mi>e</mi> <mi>v</mi> <mi>e</mi> <mi>n</mi> <mi>t</mi> <mo>)</mo></mrow> <mo>=</mo> <mo form="prefix">log</mo> <mfrac><mn>1</mn> <mrow><mi>p</mi><mo>(</mo><mi>e</mi><mi>v</mi><mi>e</mi><mi>n</mi><mi>t</mi><mo>)</mo></mrow></mfrac> <mo>=</mo> <mo>-</mo> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>e</mi> <mi>v</mi> <mi>e</mi> <mi>n</mi> <mi>t</mi> <mo>)</mo></mrow> <mo>)</mo></mrow></mrow></math>

现在，随机变量的熵（在我们的情况下是训练数据集中的特定特征）被定义为与随机变量相关的*预期惊喜*，因此我们必须将随机变量的每种可能结果（所涉及特征的每个值的惊喜）与它们各自的概率相乘，得到：

<math alttext="dollar-sign upper E n t r o p y left-parenthesis upper X right-parenthesis equals minus p left-parenthesis o u t c o m e 1 right-parenthesis log left-parenthesis p left-parenthesis o u t c o m e 1 right-parenthesis right-parenthesis minus p left-parenthesis o u t c o m e 2 right-parenthesis log left-parenthesis p left-parenthesis o u t c o m e 2 right-parenthesis right-parenthesis minus ellipsis minus p left-parenthesis o u t c o m e Subscript n Baseline right-parenthesis log left-parenthesis p left-parenthesis o u t c o m e Subscript n Baseline right-parenthesis right-parenthesis period dollar-sign"><mrow><mi>E</mi> <mi>n</mi> <mi>t</mi> <mi>r</mi> <mi>o</mi> <mi>p</mi> <mi>y</mi> <mrow><mo>(</mo> <mi>X</mi> <mo>)</mo></mrow> <mo>=</mo> <mo>-</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>o</mi> <mi>u</mi> <mi>t</mi> <mi>c</mi> <mi>o</mi> <mi>m</mi> <msub><mi>e</mi> <mn>1</mn></msub> <mo>)</mo></mrow> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>o</mi> <mi>u</mi> <mi>t</mi> <mi>c</mi> <mi>o</mi> <mi>m</mi> <msub><mi>e</mi> <mn>1</mn></msub> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>-</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>o</mi> <mi>u</mi> <mi>t</mi> <mi>c</mi> <mi>o</mi> <mi>m</mi> <msub><mi>e</mi> <mn>2</mn></msub> <mo>)</mo></mrow> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>o</mi> <mi>u</mi> <mi>t</mi> <mi>c</mi> <mi>o</mi> <mi>m</mi> <msub><mi>e</mi> <mn>2</mn></msub> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>-</mo> <mo>⋯</mo> <mo>-</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>o</mi> <mi>u</mi> <mi>t</mi> <mi>c</mi> <mi>o</mi> <mi>m</mi> <msub><mi>e</mi> <mi>n</mi></msub> <mo>)</mo></mrow> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>o</mi> <mi>u</mi> <mi>t</mi> <mi>c</mi> <mi>o</mi> <mi>m</mi> <msub><mi>e</mi> <mi>n</mi></msub> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>.</mo></mrow></math>

我们训练数据的一个特征的熵假设有一堆值是：

<math alttext="dollar-sign upper E n t r o p y left-parenthesis upper F e a t u r e right-parenthesis equals minus p left-parenthesis v a l u e 1 right-parenthesis log left-parenthesis p left-parenthesis v a l u e 1 right-parenthesis right-parenthesis minus p left-parenthesis v a l u e 2 right-parenthesis log left-parenthesis p left-parenthesis v a l u e 2 right-parenthesis right-parenthesis minus ellipsis minus p left-parenthesis v a l u e Subscript n Baseline right-parenthesis log left-parenthesis p left-parenthesis v a l u e Subscript n Baseline right-parenthesis right-parenthesis dollar-sign"><mrow><mi>E</mi> <mi>n</mi> <mi>t</mi> <mi>r</mi> <mi>o</mi> <mi>p</mi> <mi>y</mi> <mrow><mo>(</mo> <mi>F</mi> <mi>e</mi> <mi>a</mi> <mi>t</mi> <mi>u</mi> <mi>r</mi> <mi>e</mi> <mo>)</mo></mrow> <mo>=</mo> <mo>-</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>1</mn></msub> <mo>)</mo></mrow> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>1</mn></msub> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>-</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>2</mn></msub> <mo>)</mo></mrow> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>2</mn></msub> <mo>)</mo></mrow> <mo>)</mo></mrow> <mo>-</mo> <mo>⋯</mo> <mo>-</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mi>n</mi></msub> <mo>)</mo></mrow> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mi>n</mi></msub> <mo>)</mo></mrow> <mo>)</mo></mrow></mrow></math>

由于我们的目标是选择在提供关于结果（标签或目标特征）的大信息增益的特征上进行分割，让我们首先计算结果特征的熵。为简单起见，假设这是一个二元分类问题，因此结果特征只有两个值：正（在类中）和负（不在类中）。如果让*p*表示目标特征中的正实例数，*n*表示负实例数，则*p+n=m*将是训练数据子集中的实例数。现在从该目标列中选择正实例的概率将是<math alttext="StartFraction p Over m EndFraction equals StartFraction p Over p plus n EndFraction"><mrow><mfrac><mi>p</mi> <mi>m</mi></mfrac> <mo>=</mo> <mfrac><mi>p</mi> <mrow><mi>p</mi><mo>+</mo><mi>n</mi></mrow></mfrac></mrow></math>，选择负实例的概率类似地是<math alttext="StartFraction n Over m EndFraction equals StartFraction n Over p plus n EndFraction"><mrow><mfrac><mi>n</mi> <mi>m</mi></mfrac> <mo>=</mo> <mfrac><mi>n</mi> <mrow><mi>p</mi><mo>+</mo><mi>n</mi></mrow></mfrac></mrow></math>。因此，结果特征的熵（不利用其他特征的任何信息）是：

<math alttext="dollar-sign StartLayout 1st Row 1st Column Blank 2nd Column upper E n t r o p y left-parenthesis Outcome Feature right-parenthesis 2nd Row 1st Column Blank 2nd Column equals minus p left-parenthesis p o s i t i v e right-parenthesis log left-parenthesis p left-parenthesis p o s i t i v e right-parenthesis right-parenthesis minus p left-parenthesis n e g a t i v e right-parenthesis log left-parenthesis p left-parenthesis n e g a t i v e right-parenthesis right-parenthesis 3rd Row 1st Column Blank 2nd Column equals minus StartFraction p Over p plus n EndFraction log left-parenthesis StartFraction p Over p plus n EndFraction right-parenthesis minus StartFraction n Over p plus n EndFraction log left-parenthesis StartFraction n Over p plus n EndFraction right-parenthesis EndLayout dollar-sign"><mtable displaystyle="true"><mtr><mtd columnalign="left"><mrow><mi>E</mi> <mi>n</mi> <mi>t</mi> <mi>r</mi> <mi>o</mi> <mi>p</mi> <mi>y</mi> <mo>(</mo> <mtext>Outcome</mtext> <mtext>Feature</mtext> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>=</mo> <mo>-</mo> <mi>p</mi> <mo>(</mo> <mi>p</mi> <mi>o</mi> <mi>s</mi> <mi>i</mi> <mi>t</mi> <mi>i</mi> <mi>v</mi> <mi>e</mi> <mo>)</mo> <mo form="prefix">log</mo> <mo>(</mo> <mi>p</mi> <mo>(</mo> <mi>p</mi> <mi>o</mi> <mi>s</mi> <mi>i</mi> <mi>t</mi> <mi>i</mi> <mi>v</mi> <mi>e</mi> <mo>)</mo> <mo>)</mo> <mo>-</mo> <mi>p</mi> <mo>(</mo> <mi>n</mi> <mi>e</mi> <mi>g</mi> <mi>a</mi> <mi>t</mi> <mi>i</mi> <mi>v</mi> <mi>e</mi> <mo>)</mo> <mo form="prefix">log</mo> <mo>(</mo> <mi>p</mi> <mo>(</mo> <mi>n</mi> <mi>e</mi> <mi>g</mi> <mi>a</mi> <mi>t</mi> <mi>i</mi> <mi>v</mi> <mi>e</mi> <mo>)</mo> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>=</mo> <mo>-</mo> <mfrac><mi>p</mi> <mrow><mi>p</mi><mo>+</mo><mi>n</mi></mrow></mfrac> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mfrac><mi>p</mi> <mrow><mi>p</mi><mo>+</mo><mi>n</mi></mrow></mfrac> <mo>)</mo></mrow> <mo>-</mo> <mfrac><mi>n</mi> <mrow><mi>p</mi><mo>+</mo><mi>n</mi></mrow></mfrac> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mfrac><mi>n</mi> <mrow><mi>p</mi><mo>+</mo><mi>n</mi></mrow></mfrac> <mo>)</mo></mrow></mrow></mtd></mtr></mtable></math>

现在我们利用另一个特征的信息，并计算结果特征熵的差异，我们期望随着获得更多信息（更多信息通常会导致更少的惊喜），结果特征的熵将减少。

假设我们选择特征*A*来分割我们决策树上的一个节点。假设特征*A*有四个值，并且有<math alttext="k 1"><msub><mi>k</mi> <mn>1</mn></msub></math>个实例具有<math alttext="v a l u e 1"><mrow><mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>1</mn></msub></mrow></math>，其中<math alttext="p 1"><msub><mi>p</mi> <mn>1</mn></msub></math>个标记为正，<math alttext="n 1"><msub><mi>n</mi> <mn>1</mn></msub></math>个标记为负，所以<math alttext="p 1 plus n 1 equals k 1"><mrow><msub><mi>p</mi> <mn>1</mn></msub> <mo>+</mo> <msub><mi>n</mi> <mn>1</mn></msub> <mo>=</mo> <msub><mi>k</mi> <mn>1</mn></msub></mrow></math>。同样，特征*A*有<math alttext="k 2"><msub><mi>k</mi> <mn>2</mn></msub></math>个实例具有<math alttext="v a l u e 2"><mrow><mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>2</mn></msub></mrow></math>，其中<math alttext="p 2"><msub><mi>p</mi> <mn>2</mn></msub></math>个标记为正，<math alttext="n 2"><msub><mi>n</mi> <mn>2</mn></msub></math>个标记为负，所以<math alttext="p 2 plus n 2 equals k 2"><mrow><msub><mi>p</mi> <mn>2</mn></msub> <mo>+</mo> <msub><mi>n</mi> <mn>2</mn></msub> <mo>=</mo> <msub><mi>k</mi> <mn>2</mn></msub></mrow></math>。对于特征*A*的<math alttext="v a l u e 3"><mrow><mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>3</mn></msub></mrow></math>和<math alttext="v a l u e 4"><mrow><mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>4</mn></msub></mrow></math>也是一样的。注意<math alttext="k 1 plus k 2 plus k 3 plus k 4 equals m"><mrow><msub><mi>k</mi> <mn>1</mn></msub> <mo>+</mo> <msub><mi>k</mi> <mn>2</mn></msub> <mo>+</mo> <msub><mi>k</mi> <mn>3</mn></msub> <mo>+</mo> <msub><mi>k</mi> <mn>4</mn></msub> <mo>=</mo> <mi>m</mi></mrow></math>，即数据集训练子集中的实例总数。现在，特征*A*的每个值<math alttext="v a l u e Subscript k"><mrow><mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mi>k</mi></msub></mrow></math>可以被视为一个随机变量，具有<math alttext="p Subscript k"><msub><mi>p</mi> <mi>k</mi></msub></math>个正结果和<math alttext="n Subscript k"><msub><mi>n</mi> <mi>k</mi></msub></math>个负结果，因此我们可以计算它的熵（期望的惊喜）。

数学符号开始布局第一行第一列熵左括号v a l u e 1右括号第二列等于负开始分数p 1除以p 1加n 1结束分数对数左括号开始分数p 1除以p 1加n 1结束分数右括号减开始分数n 1除以p 1加n 1结束分数对数左括号开始分数n 1除以p 1加n 1结束分数右括号第二行第一列熵左括号v a l u e 2右括号第二列等于负开始分数p 2除以p 2加n 2结束分数对数左括号开始分数p 2除以p 2加n 2结束分数右括号减开始分数n 2除以p 2加n 2结束分数对数左括号开始分数n 2除以p 2加n 2结束分数右括号第三行第一列熵左括号v a l u e 3右括号第二列等于负开始分数p 3除以p 3加n 3结束分数对数左括号开始分数p 3除以p 3加n 3结束分数右括号减开始分数n 3除以p 3加n 3结束分数对数左括号开始分数n 3除以p 3加n 3结束分数右括号第四行第一列熵左括号v a l u e 4右括号第二列等于负开始分数p 4除以p 4加n 4结束分数对数左括号开始分数p 4除以p 4加n 4结束分数右括号减开始分数n 4除以p 4加n 4结束分数对数左括号开始分数n 4除以p 4加n 4结束分数右括号结束布局。

既然我们有了这些信息，我们可以计算在特征*A*上分割后的*预期熵*，因此我们将上述四个熵分别乘以其相应的概率相加。请注意，<math alttext="p left-parenthesis v a l u e 1 right-parenthesis equals StartFraction k 1 Over m EndFraction"><mrow><mi>p</mi> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>1</mn></msub> <mo>)</mo></mrow> <mo>=</mo> <mfrac><msub><mi>k</mi> <mn>1</mn></msub> <mi>m</mi></mfrac></mrow></math>，<math alttext="p left-parenthesis v a l u e 2 right-parenthesis equals StartFraction k 2 Over m EndFraction"><mrow><mi>p</mi> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>2</mn></msub> <mo>)</mo></mrow> <mo>=</mo> <mfrac><msub><mi>k</mi> <mn>2</mn></msub> <mi>m</mi></mfrac></mrow></math>，<math alttext="p left-parenthesis v a l u e 3 right-parenthesis equals StartFraction k 3 Over m EndFraction"><mrow><mi>p</mi> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>3</mn></msub> <mo>)</mo></mrow> <mo>=</mo> <mfrac><msub><mi>k</mi> <mn>3</mn></msub> <mi>m</mi></mfrac></mrow></math>，以及<math alttext="p left-parenthesis v a l u e 4 right-parenthesis equals StartFraction k 4 Over m EndFraction"><mrow><mi>p</mi> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>4</mn></msub> <mo>)</mo></mrow> <mo>=</mo> <mfrac><msub><mi>k</mi> <mn>4</mn></msub> <mi>m</mi></mfrac></mrow></math>。因此，在特征*A*上分割后的预期熵将是：

<math alttext="dollar-sign StartLayout 1st Row 1st Column Blank 2nd Column Expected Entropy left-parenthesis Feature upper A right-parenthesis 2nd Row 1st Column Blank 2nd Column equals p left-parenthesis v a l u e 1 right-parenthesis Entropy left-parenthesis v a l u e 1 right-parenthesis plus p left-parenthesis v a l u e 2 right-parenthesis Entropy left-parenthesis v a l u e 2 right-parenthesis 3rd Row 1st Column Blank 2nd Column plus p left-parenthesis v a l u e 3 right-parenthesis Entropy left-parenthesis v a l u e 3 right-parenthesis plus p left-parenthesis v a l u e 4 right-parenthesis Entropy left-parenthesis v a l u e 4 right-parenthesis 4th Row 1st Column Blank 2nd Column equals StartFraction k 1 Over m EndFraction Entropy left-parenthesis v a l u e 1 right-parenthesis plus StartFraction k 2 Over m EndFraction Entropy left-parenthesis v a l u e 2 right-parenthesis plus StartFraction k 3 Over m EndFraction Entropy left-parenthesis v a l u e 3 right-parenthesis plus StartFraction k 4 Over m EndFraction Entropy left-parenthesis v a l u e 4 right-parenthesis EndLayout dollar-sign"><mtable displaystyle="true"><mtr><mtd columnalign="left"><mrow><mtext>预期</mtext> <mtext>熵</mtext> <mo>(</mo> <mtext>特征</mtext> <mtext>A</mtext> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>=</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>1</mn></msub> <mo>)</mo></mrow> <mtext>熵</mtext> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>1</mn></msub> <mo>)</mo></mrow> <mo>+</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>2</mn></msub> <mo>)</mo></mrow> <mtext>熵</mtext> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>2</mn></msub> <mo>)</mo></mrow></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>+</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>3</mn></msub> <mo>)</mo></mrow> <mtext>熵</mtext> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>3</mn></msub> <mo>)</mo></mrow> <mo>+</mo> <mi>p</mi> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>4</mn></msub> <mo>)</mo></mrow> <mtext>熵</mtext> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>4</mn></msub> <mo>)</mo></mrow></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>=</mo> <mfrac><msub><mi>k</mi> <mn>1</mn></msub> <mi>m</mi></mfrac> <mtext>熵</mtext> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>1</mn></msub> <mo>)</mo></mrow> <mo>+</mo> <mfrac><msub><mi>k</mi> <mn>2</mn></msub> <mi>m</mi></mfrac> <mtext>熵</mtext> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>2</mn></msub> <mo>)</mo></mrow> <mo>+</mo> <mfrac><msub><mi>k</mi> <mn>3</mn></msub> <mi>m</mi></mfrac> <mtext>熵</mtext> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>3</mn></msub> <mo>)</mo></mrow> <mo>+</mo> <mfrac><msub><mi>k</mi> <mn>4</mn></msub> <mi>m</mi></mfrac> <mtext>熵</mtext> <mrow><mo>(</mo> <mi>v</mi> <mi>a</mi> <mi>l</mi> <mi>u</mi> <msub><mi>e</mi> <mn>4</mn></msub> <mo>)</mo></mrow></mrow></mtd></mtr></mtable></math>

其中，特征*A*的每个值的熵在前一段中给出。

那么，使用特征*A*进行分割所获得的信息是什么？这将是没有来自特征*A*的任何信息时结果特征的熵与特征*A*的预期熵之间的差异。因此，我们有一个关于*信息增益*的公式，假设我们决定在特征*A*上进行分割：

<math alttext="dollar-sign StartLayout 1st Row 1st Column Blank 2nd Column Information Gain equals 2nd Row 1st Column Blank 2nd Column upper E n t r o p y left-parenthesis Outcome Feature right-parenthesis minus Expected Entropy left-parenthesis Feature upper A right-parenthesis equals 3rd Row 1st Column Blank 2nd Column minus StartFraction p Over p plus n EndFraction log left-parenthesis StartFraction p Over p plus n EndFraction right-parenthesis minus StartFraction n Over p plus n EndFraction log left-parenthesis StartFraction n Over p plus n EndFraction right-parenthesis minus Expected Entropy left-parenthesis Feature upper A right-parenthesis period EndLayout dollar-sign"><mtable displaystyle="true"><mtr><mtd columnalign="left"><mrow><mtext>Information</mtext> <mtext>Gain</mtext> <mo>=</mo></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mi>E</mi> <mi>n</mi> <mi>t</mi> <mi>r</mi> <mi>o</mi> <mi>p</mi> <mi>y</mi> <mo>(</mo> <mtext>Outcome</mtext> <mtext>Feature</mtext> <mo>)</mo> <mo>-</mo> <mtext>Expected</mtext> <mtext>Entropy</mtext> <mo>(</mo> <mtext>Feature</mtext> <mtext>A</mtext> <mo>)</mo> <mo>=</mo></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>-</mo> <mfrac><mi>p</mi> <mrow><mi>p</mi><mo>+</mo><mi>n</mi></mrow></mfrac> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mfrac><mi>p</mi> <mrow><mi>p</mi><mo>+</mo><mi>n</mi></mrow></mfrac> <mo>)</mo></mrow> <mo>-</mo> <mfrac><mi>n</mi> <mrow><mi>p</mi><mo>+</mo><mi>n</mi></mrow></mfrac> <mo form="prefix">log</mo> <mrow><mo>(</mo> <mfrac><mi>n</mi> <mrow><mi>p</mi><mo>+</mo><mi>n</mi></mrow></mfrac> <mo>)</mo></mrow> <mo>-</mo> <mtext>Expected</mtext> <mtext>Entropy</mtext> <mrow><mo>(</mo> <mtext>Feature</mtext> <mtext>A</mtext> <mo>)</mo></mrow> <mo>.</mo></mrow></mtd></mtr></mtable></math>

现在很容易浏览训练数据子集的每个特征，并计算使用该特征进行拆分后产生的信息增益。最终，决策树算法决定在具有最高信息增益的特征上进行拆分。该算法对每个节点和树的每一层递归执行此操作，直到没有要拆分的特征或数据实例。因此，我们得到了基于熵的决策树。

将上述逻辑推广到具有多类输出的情况并不太困难，例如，具有三个或更多目标标签的分类问题。经典的[Iris数据集](https://archive.ics.uci.edu/ml/datasets/iris)来自[UCI机器学习库](https://archive.ics.uci.edu/ml/index.php)是一个具有三个目标标签的绝佳示例。该数据集为给定的鸢尾花具有四个特征：其萼片长度和宽度，以及其花瓣长度和宽度。请注意，每个特征都是连续随机变量，而不是离散的。因此，我们必须设计一个测试来拆分每个特征的值，*在*应用上述逻辑之前。这是数据科学项目的特征工程阶段的一部分。这里的工程步骤是：将连续值特征转换为布尔特征，例如，*花瓣长度>2.45*？我们不会详细介绍如何选择数字2.45，但是现在你可能可以猜到这里也应该进行优化过程。

#### 基尼不纯度

每个决策树都由其节点，分支和叶子特征。如果一个节点只包含来自训练数据子集的具有相同目标标签的数据实例（这意味着它们属于同一类），则认为该节点是*纯*的。请注意，纯节点是期望的节点，因为我们知道它的类别。因此，算法希望以最小化节点的*不纯度*的方式来生长树：如果节点中的数据实例不都属于同一类，则该节点是*不纯*的。*基尼不纯度*以以下方式量化这种不纯度：

假设我们的分类问题有三个类别，就像[Iris数据集](https://archive.ics.uci.edu/ml/datasets/iris)一样。假设决策树中的某个节点是为了适应这个数据集而生长的，有*n*个训练实例，其中<math alttext="n 1"><msub><mi>n</mi> <mn>1</mn></msub></math>个属于第一类，<math alttext="n 2"><msub><mi>n</mi> <mn>2</mn></msub></math>个属于第二类，<math alttext="n 3"><msub><mi>n</mi> <mn>3</mn></msub></math>个属于第三类（所以<math alttext="n 1 plus n 2 plus n 3 equals n"><mrow><msub><mi>n</mi> <mn>1</mn></msub> <mo>+</mo> <msub><mi>n</mi> <mn>2</mn></msub> <mo>+</mo> <msub><mi>n</mi> <mn>3</mn></msub> <mo>=</mo> <mi>n</mi></mrow></math>）。那么这个节点的基尼不纯度由以下公式给出：

<math alttext="dollar-sign Gini impurity equals 1 minus left-parenthesis StartFraction n 1 Over n EndFraction right-parenthesis squared minus left-parenthesis StartFraction n 2 Over n EndFraction right-parenthesis squared minus left-parenthesis StartFraction n 3 Over n EndFraction right-parenthesis squared dollar-sign"><mrow><mtext>Gini</mtext> <mtext>impurity</mtext> <mo>=</mo> <mn>1</mn> <mo>-</mo> <msup><mrow><mo>(</mo><mfrac><msub><mi>n</mi> <mn>1</mn></msub> <mi>n</mi></mfrac><mo>)</mo></mrow> <mn>2</mn></msup> <mo>-</mo> <msup><mrow><mo>(</mo><mfrac><msub><mi>n</mi> <mn>2</mn></msub> <mi>n</mi></mfrac><mo>)</mo></mrow> <mn>2</mn></msup> <mo>-</mo> <msup><mrow><mo>(</mo><mfrac><msub><mi>n</mi> <mn>3</mn></msub> <mi>n</mi></mfrac><mo>)</mo></mrow> <mn>2</mn></msup></mrow></math>

因此，对于每个节点，计算属于每个类别的数据实例的比例，然后求平方，然后从1中减去这些的总和。请注意，如果节点的所有数据实例都属于同一类，则上述公式给出的基尼不纯度等于零。

决策树生长算法现在寻找每个特征和特征中的分割点，以产生平均基尼不纯度最低的子节点。这意味着节点的子节点平均上必须比父节点更纯。因此，该算法试图最小化两个子节点（二叉树）的基尼不纯度的加权平均值。每个子节点的基尼不纯度由其相对大小加权，其相对大小是其实例数与该树层中的总实例数（与其父节点的实例数相同）之比。因此，我们最终需要搜索解决以下最小化问题的特征和分割点（对于每个特征）组合：

<math alttext="dollar-sign min Underscript upper F e a t u r e comma upper F e a t u r e upper S p l i t upper V a l u e Endscripts StartFraction n Subscript l e f t Baseline Over n EndFraction upper G i n i left-parenthesis Left Node right-parenthesis plus StartFraction n Subscript r i g h t Baseline Over n EndFraction upper G i n i left-parenthesis Right Node right-parenthesis dollar-sign"><mrow><msub><mo form="prefix" movablelimits="true">min</mo> <mrow><mi>F</mi><mi>e</mi><mi>a</mi><mi>t</mi><mi>u</mi><mi>r</mi><mi>e</mi><mo>,</mo><mi>F</mi><mi>e</mi><mi>a</mi><mi>t</mi><mi>u</mi><mi>r</mi><mi>e</mi><mi>S</mi><mi>p</mi><mi>l</mi><mi>i</mi><mi>t</mi><mi>V</mi><mi>a</mi><mi>l</mi><mi>u</mi><mi>e</mi></mrow></msub> <mfrac><msub><mi>n</mi> <mrow><mi>l</mi><mi>e</mi><mi>f</mi><mi>t</mi></mrow></msub> <mi>n</mi></mfrac> <mi>G</mi> <mi>i</mi> <mi>n</mi> <mi>i</mi> <mrow><mo>(</mo> <mtext>Left</mtext> <mtext>Node</mtext> <mo>)</mo></mrow> <mo>+</mo> <mfrac><msub><mi>n</mi> <mrow><mi>r</mi><mi>i</mi><mi>g</mi><mi>h</mi><mi>t</mi></mrow></msub> <mi>n</mi></mfrac> <mi>G</mi> <mi>i</mi> <mi>n</mi> <mi>i</mi> <mrow><mo>(</mo> <mtext>Right</mtext> <mtext>Node</mtext> <mo>)</mo></mrow></mrow></math>

左右子节点中最终存在的数据实例的数量分别为<math alttext="n Subscript l e f t"><msub><mi>n</mi> <mrow><mi>l</mi><mi>e</mi><mi>f</mi><mi>t</mi></mrow></msub></math>和<math alttext="n Subscript r i g h t"><msub><mi>n</mi> <mrow><mi>r</mi><mi>i</mi><mi>g</mi><mi>h</mi><mi>t</mi></mrow></msub></math>，*n*是父节点中存在的数据实例的数量（注意<math alttext="n Subscript l e f t"><msub><mi>n</mi> <mrow><mi>l</mi><mi>e</mi><mi>f</mi><mi>t</mi></mrow></msub></math>和<math alttext="n Subscript r i g h t"><msub><mi>n</mi> <mrow><mi>r</mi><mi>i</mi><mi>g</mi><mi>h</mi><mi>t</mi></mrow></msub></math>必须加起来等于*n*)。

#### 回归决策树

需要指出的是，决策树既可以用于回归也可以用于分类。回归决策树返回的是预测值而不是类别，但与分类树类似的过程适用。

与选择最大化信息增益或最小化基尼不纯度的特征和特征值（例如，身高>3英尺吗？）来分割节点不同，我们选择最小化真实标签与左右子节点中所有实例标签的平均值之间的均方距离的特征和特征值。也就是说，算法选择要分割的特征和特征值，然后查看由该分割产生的左右子节点，并计算：

+   左节点中所有训练数据实例标签的平均值。这个平均值将成为左节点值<math alttext="y Subscript l e f t"><msub><mi>y</mi> <mrow><mi>l</mi><mi>e</mi><mi>f</mi><mi>t</mi></mrow></msub></math>，如果该节点最终成为叶节点，这就是决策树预测的值。

+   右节点中所有训练数据实例标签的平均值。这个平均值将成为右节点值<math alttext="y Subscript r i g h t"><msub><mi>y</mi> <mrow><mi>r</mi><mi>i</mi><mi>g</mi><mi>h</mi><mi>t</mi></mrow></msub></math>。同样，如果该节点最终成为叶节点，这就是决策树预测的值。

+   左节点值与左节点中每个实例的真实标签之间的平方距离之和<math alttext="sigma-summation Underscript upper L e f t upper N o d e upper I n s t a n c e s Endscripts StartAbsoluteValue y Subscript t r u e Superscript i Baseline minus y Subscript l e f t Baseline EndAbsoluteValue squared"><mrow><msub><mo>∑</mo> <mrow><mi>L</mi><mi>e</mi><mi>f</mi><mi>t</mi><mi>N</mi><mi>o</mi><mi>d</mi><mi>e</mi><mi>I</mi><mi>n</mi><mi>s</mi><mi>t</mi><mi>a</mi><mi>n</mi><mi>c</mi><mi>e</mi><mi>s</mi></mrow></msub> <msup><mrow><mo>|</mo><msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mi>i</mi></msubsup> <mo>-</mo><msub><mi>y</mi> <mrow><mi>l</mi><mi>e</mi><mi>f</mi><mi>t</mi></mrow></msub> <mo>|</mo></mrow> <mn>2</mn></msup></mrow></math>。

+   右节点值与右节点中每个实例的真实标签之间的平方距离之和<math alttext="sigma-summation Underscript upper R i g h t upper N o d e upper I n s t a n c e s Endscripts StartAbsoluteValue y Subscript t r u e Superscript i Baseline minus y Subscript r i g h t Baseline EndAbsoluteValue squared"><mrow><msub><mo>∑</mo> <mrow><mi>R</mi><mi>i</mi><mi>g</mi><mi>h</mi><mi>t</mi><mi>N</mi><mi>o</mi><mi>d</mi><mi>e</mi><mi>I</mi><mi>n</mi><mi>s</mi><mi>t</mi><mi>a</mi><mi>n</mi><mi>c</mi><mi>e</mi><mi>s</mi></mrow></msub> <msup><mrow><mo>|</mo><msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mi>i</mi></msubsup> <mo>-</mo><msub><mi>y</mi> <mrow><mi>r</mi><mi>i</mi><mi>g</mi><mi>h</mi><mi>t</mi></mrow></msub> <mo>|</mo></mrow> <mn>2</mn></msup></mrow></math>。

+   上述两个总和的加权平均值，其中每个节点的权重由其相对于父节点的大小加权，就像我们对基尼不纯度所做的那样：

<math alttext="dollar-sign StartFraction n Subscript l e f t Baseline Over n EndFraction sigma-summation Underscript upper L e f t upper N o d e upper I n s t a n c e s Endscripts StartAbsoluteValue y Subscript t r u e Superscript i Baseline minus y Subscript l e f t Baseline EndAbsoluteValue squared plus StartFraction n Subscript r i g h t Baseline Over n EndFraction sigma-summation Underscript upper R i g h t upper N o d e upper I n s t a n c e s Endscripts StartAbsoluteValue y Subscript t r u e Superscript i Baseline minus y Subscript r i g h t Baseline EndAbsoluteValue squared right-parenthesis period dollar-sign"><mrow><mfrac><msub><mi>n</mi> <mrow><mi>l</mi><mi>e</mi><mi>f</mi><mi>t</mi></mrow></msub> <mi>n</mi></mfrac> <msub><mo>∑</mo> <mrow><mi>L</mi><mi>e</mi><mi>f</mi><mi>t</mi><mi>N</mi><mi>o</mi><mi>d</mi><mi>e</mi><mi>I</mi><mi>n</mi><mi>s</mi><mi>t</mi><mi>a</mi><mi>n</mi><mi>c</mi><mi>e</mi><mi>s</mi></mrow></msub> <mrow><mo>|</mo></mrow> <msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mi>i</mi></msubsup> <mo>-</mo> <msub><mi>y</mi> <mrow><mi>l</mi><mi>e</mi><mi>f</mi><mi>t</mi></mrow></msub> <msup><mrow><mo>|</mo></mrow> <mn>2</mn></msup> <mo>+</mo> <mfrac><msub><mi>n</mi> <mrow><mi>r</mi><mi>i</mi><mi>g</mi><mi>h</mi><mi>t</mi></mrow></msub> <mi>n</mi></mfrac> <msub><mo>∑</mo> <mrow><mi>R</mi><mi>i</mi><mi>g</mi><mi>h</mi><mi>t</mi><mi>N</mi><mi>o</mi><mi>d</mi><mi>e</mi><mi>I</mi><mi>n</mi><mi>s</mi><mi>t</mi><mi>a</mi><mi>n</mi><mi>c</mi><mi>e</mi><mi>s</mi></mrow></msub> <mrow><mo>|</mo></mrow> <msubsup><mi>y</mi> <mrow><mi>t</mi><mi>r</mi><mi>u</mi><mi>e</mi></mrow> <mi>i</mi></msubsup> <mo>-</mo> <msub><mi>y</mi> <mrow><mi>r</mi><mi>i</mi><mi>g</mi><mi>h</mi><mi>t</mi></mrow></msub> <mrow><msup><mo>|</mo> <mn>2</mn></msup> <mo>)</mo></mrow> <mo>.</mo></mrow></math>

上述算法是*贪婪*和计算密集的，因为它必须对*每个特征和每个可能的特征拆分值*进行操作，然后选择提供左右子节点之间加权平方误差平均值最小的特征和特征拆分。

CART（分类和回归树）算法是一种著名的算法，被软件包使用，包括Python的Scikit Learn，我们在Jupyter笔记本中使用这本书的补充。该算法生成的树的节点只有两个子节点（二叉树），每个节点上的测试只有是或否的答案。其他算法，如ID3，可以生成具有两个或多个子节点的树。

### 决策树的缺点

决策树非常容易解释，因为有很多很好的原因而受欢迎：它们适应大数据集，不同的数据类型（离散和连续特征，不需要数据缩放），并且可以执行回归和分类任务。然而，它们可能不稳定，因为向数据集添加一个实例就可以改变树的根，从而导致非常不同的决策树。它们也对数据中的旋转敏感，因为它们的决策边界通常是水平和垂直的（不像支持向量机那样倾斜）。这是因为拆分通常发生在特定的特征值，因此决策边界最终与特征轴平行。解决这个问题的一个方法是转换数据集以匹配其*主轴*，使用稍后在[第11章](ch11.xhtml#ch11)中介绍的*奇异值分解方法*。决策树往往会过度拟合数据，因此需要修剪。这通常使用统计测试来完成。构建树涉及的贪婪算法，其中搜索发生在所有特征及其值上，使它们在计算上昂贵且不太准确。接下来讨论的随机森林解决了其中一些缺点。

## 随机森林

当我第一次了解决策树时，最令我困惑的是：

+   我们如何开始构建树，也就是如何决定哪个数据特征是根特征？

+   在特定的特征值上，我们决定如何分割一个节点？

+   我们什么时候停下来？

+   实质上，我们如何构建一棵树？

（请注意，我们在上一小节中回答了一些上述问题。）我在网上寻找答案时并没有使事情变得更容易，只是遇到了声明说决策树很容易构建和理解，所以感觉自己是唯一一个对决策树深感困惑的人。

当我了解了*随机森林*后，我的困惑立刻消失了。随机森林的惊人之处在于，我们可以获得非常好的回归或分类结果，*而不需要*回答我困惑的问题。通过随机化整个过程，也就是构建许多决策树并用两个词回答我的问题：“随机选择”，然后将它们的预测聚合在一起，产生非常好的结果，甚至比一个精心设计的决策树还要好。有人说*随机化经常产生可靠性*！

随机森林的另一个非常有用的特性是它们提供了*特征重要性*的度量，帮助我们找出哪些特征对我们的预测有显著影响，并且也有助于特征选择。

## k均值聚类

数据分析师的一个常见目标是将数据分成*簇*，每个簇突出显示某些共同特征。*k均值聚类*是一种常见的机器学习方法，它将*n*个数据点（向量）分成*k*个簇，其中每个数据点被分配到与其最近均值的簇。每个簇的均值，或者它的质心，作为簇的原型。总的来说，k均值聚类最小化了每个簇内的方差（到均值的平方欧氏距离）。

k均值聚类最常见的算法是迭代的：

+   从初始的一组*k*均值开始。这意味着我们提前指定了簇的数量，这就引出了一个问题：如何初始化？如何选择第一个*k*质心的位置？有相关的文献。

+   将每个数据点分配到与其平方欧氏距离最近的簇中。

+   重新计算每个簇的均值。

当数据点分配到每个簇不再改变时，算法收敛。

# 分类模型的性能度量

开发计算事物并产生输出的数学模型相对容易。但是，开发能够很好地执行我们期望的任务的模型则完全不同。此外，根据某些指标表现良好的模型在其他一些指标下表现不佳。我们需要额外小心地开发性能指标，并根据我们特定的用例决定依赖于哪些指标。

衡量预测数值的模型的性能，例如回归模型，比分类模型更容易，因为我们有许多方法来计算数字之间的距离（好的预测和坏的预测）。另一方面，当我们的任务是分类（我们可以使用逻辑回归、softmax回归、支持向量机、决策树、随机森林或神经网络等模型），我们必须对评估性能进行一些额外的思考。此外，通常存在权衡。例如，如果我们的任务是将YouTube视频分类为适合儿童观看（正面）或不适合儿童观看（负面），我们是否调整我们的模型以减少假阳性或假阴性的数量？显然，如果一个视频被分类为安全，而实际上是不安全的（假阳性），那么问题就更加棘手，因此我们的性能指标需要反映这一点。

以下是常用于分类模型的性能测量。不要担心记住它们的名称，因为它们的命名方式并不合乎逻辑。相反，花时间理解它们的含义。

+   **准确度**：预测模型正确分类的百分比：

<math alttext="美元符号上限 A c c u r a c y 等于 StartFraction 真正例加真负例 除以 所有预测为正例加所有预测为负例 美元符号"><mrow><mi>A</mi> <mi>c</mi> <mi>c</mi> <mi>u</mi> <mi>r</mi> <mi>a</mi> <mi>c</mi> <mi>y</mi> <mo>=</mo> <mfrac><mrow><mtext>真</mtext><mtext>正例</mtext><mtext>+</mtext><mtext>真</mtext><mtext>负例</mtext></mrow> <mrow><mtext>所有</mtext><mtext>预测为正例+</mtext><mtext>所有</mtext><mtext>预测为负例</mtext></mrow></mfrac></mrow></math>

+   **混淆矩阵**：计算所有真正例、假正例、真负例和假负例。

| 真负例 | 假正例 |
| --- | --- |
| 假负例 | 真正例 |

+   **精确度分数**：正预测的准确性：

<math alttext="美元符号上限 P r e c i s i o n 等于 StartFraction 真正例 除以 所有预测为正例 等于 StartFraction 真正例 除以 真正例加假正例 EndFraction 美元符号"><mrow><mi>P</mi> <mi>r</mi> <mi>e</mi> <mi>c</mi> <mi>i</mi> <mi>s</mi> <mi>i</mi> <mi>o</mi> <mi>n</mi> <mo>=</mo> <mfrac><mrow><mtext>真</mtext><mtext>正例</mtext></mrow> <mrow><mtext>所有</mtext><mtext>预测为正例</mtext></mrow></mfrac> <mo>=</mo> <mfrac><mrow><mtext>真</mtext><mtext>正例</mtext></mrow> <mrow><mtext>真</mtext><mtext>正例</mtext><mo>+</mo><mtext>假</mtext><mtext>正例</mtext></mrow></mfrac></mrow></math>

+   **召回率分数**：被正确分类的正实例的比率：

<math alttext="美元符号上限 R e c a l l 等于 StartFraction 真正例 除以 所有正标签 等于 StartFraction 真正例 除以 真正例加假负例 EndFraction 美元符号"><mrow><mi>R</mi> <mi>e</mi> <mi>c</mi> <mi>a</mi> <mi>l</mi> <mi>l</mi> <mo>=</mo> <mfrac><mrow><mtext>真</mtext><mtext>正例</mtext></mrow> <mrow><mtext>所有</mtext><mtext>正标签</mtext></mrow></mfrac> <mo>=</mo> <mfrac><mrow><mtext>真</mtext><mtext>正例</mtext></mrow> <mrow><mtext>真</mtext><mtext>正例</mtext><mo>+</mo><mtext>假</mtext><mtext>负例</mtext></mrow></mfrac></mrow></math>

+   **特异度**：被正确分类的负实例的比率：

<math alttext="美元符号上限 S p e c i f i c i t y 等于 StartFraction 真负例 除以 所有负标签 等于 StartFraction 真负例 除以 真负例加假正例 EndFraction 美元符号"><mrow><mi>S</mi> <mi>p</mi> <mi>e</mi> <mi>c</mi> <mi>i</mi> <mi>f</mi> <mi>i</mi> <mi>c</mi> <mi>i</mi> <mi>t</mi> <mi>y</mi> <mo>=</mo> <mfrac><mrow><mtext>真</mtext><mtext>负例</mtext></mrow> <mrow><mtext>所有</mtext><mtext>负标签</mtext></mrow></mfrac> <mo>=</mo> <mfrac><mrow><mtext>真</mtext><mtext>负例</mtext></mrow> <mrow><mtext>真</mtext><mtext>负例</mtext><mo>+</mo><mtext>假</mtext><mtext>正例</mtext></mrow></mfrac></mrow></math>

+   **<math alttext="上限 F 1"><msub><mi>F</mi> <mn>1</mn></msub></math> 分数**：只有在精确度和召回率得分都很高时才会很高：

<math alttext="dollar-sign upper F 1 equals StartStartFraction 2 OverOver StartFraction 1 Over p r e c i s i o n EndFraction plus StartFraction 1 Over r e c a l l EndFraction EndEndFraction dollar-sign"><mrow><msub><mi>F</mi> <mn>1</mn></msub> <mo>=</mo> <mfrac><mn>2</mn> <mrow><mfrac><mn>1</mn> <mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>c</mi><mi>i</mi><mi>s</mi><mi>i</mi><mi>o</mi><mi>n</mi></mrow></mfrac><mo>+</mo><mfrac><mn>1</mn> <mrow><mi>r</mi><mi>e</mi><mi>c</mi><mi>a</mi><mi>l</mi><mi>l</mi></mrow></mfrac></mrow></mfrac></mrow></math>

+   AUC（曲线下面积）和ROC（接收器操作特性）曲线：这些曲线提供了分类模型在各种阈值下的性能度量。我们可以使用这些曲线来衡量某个变量预测某个结果的能力，例如，GRE学科考试成绩如何预测在第一年通过研究生院的资格考试？

Andrew Ng的百页书《机器学习渴望》提供了性能指标最佳实践的出色指南。在真正的人工智能应用之前，请仔细阅读，因为该书的方法基于许多试验、成功和失败。

# 总结和展望

在本章中，我们调查了一些最流行的机器学习模型，强调了本书中出现的特定数学结构：训练函数、损失函数和优化。我们讨论了线性、逻辑和softmax回归，然后迅速地浏览了支持向量机、决策树、集成和随机森林。

此外，我们提出了研究以下数学主题的理由：

微积分

最小值和最大值发生在边界或导数为零或不存在的点。

线性代数

+   线性组合特征：<math alttext="omega 1 x 1 plus omega 2 x 2 plus ellipsis plus omega Subscript n Baseline x Subscript n"><mrow><msub><mi>ω</mi> <mn>1</mn></msub> <msub><mi>x</mi> <mn>1</mn></msub> <mo>+</mo> <msub><mi>ω</mi> <mn>2</mn></msub> <msub><mi>x</mi> <mn>2</mn></msub> <mo>+</mo> <mo>⋯</mo> <mo>+</mo> <msub><mi>ω</mi> <mi>n</mi></msub> <msub><mi>x</mi> <mi>n</mi></msub></mrow></math>。

+   使用矩阵和向量表示各种数学表达式。

+   两个向量的数量积<math alttext="ModifyingAbove a With right-arrow Superscript t Baseline ModifyingAbove b With right-arrow"><mrow><msup><mover accent="true"><mi>a</mi> <mo>→</mo></mover> <mi>t</mi></msup> <mover accent="true"><mi>b</mi> <mo>→</mo></mover></mrow></math>。

+   向量的<math alttext="l squared"><msup><mi>l</mi> <mn>2</mn></msup></math>范数。

+   避免使用病态矩阵。消除线性相关的特征。这也与特征选择有关。

+   避免矩阵相乘，这太昂贵了。改为将矩阵乘以向量。

优化

+   对于凸函数，我们不必担心陷入局部最小值，因为局部最小值也是全局最小值。我们担心狭窄的山谷（下一章）。

+   梯度下降方法，它们只需要一个导数（下一章）。

+   牛顿法，它们需要两个导数或两个导数的近似（对于大数据不方便）。

+   二次规划，对偶问题和坐标下降（都出现在支持向量机中）。

统计学

+   相关矩阵和散点图。

+   特征选择的F检验和互信息。

+   标准化数据特征（减去平均值并除以标准差）。

我们没有也不会再讨论的更多步骤（但尚未）：

+   验证我们的模型-调整权重值和超参数，以避免过拟合。

+   在测试数据的测试子集上测试训练模型，这是我们的模型在训练和验证步骤中没有使用（或看到）的（我们在附带的Jupyter笔记本中进行此操作）。

+   部署和监控最终模型。

+   永远不要停止思考如何改进我们的模型，以及如何更好地将它们整合到整个生产流程中。

在下一章中，我们将迈入神经网络的新而令人兴奋的时代。
