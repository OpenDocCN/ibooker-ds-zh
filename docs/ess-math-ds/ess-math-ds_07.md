# 第7章 神经网络

在过去的10年里，一种经历复兴的回归和分类技术是神经网络。在最简单的定义中，*神经网络*是一个包含权重、偏差和非线性函数层的多层回归，位于输入变量和输出变量之间。*深度学习*是神经网络的一种流行变体，利用包含权重和偏差的多个“隐藏”（或中间）层节点。每个节点在传递给非线性函数（称为激活函数）之前类似于一个线性函数。就像我们在[第5章](ch05.xhtml#ch05)中学到的线性回归一样，优化技术如随机梯度下降被用来找到最优的权重和偏差值以最小化残差。

神经网络为计算机以前难以解决的问题提供了令人兴奋的解决方案。从识别图像中的物体到处理音频中的单词，神经网络已经创造了影响我们日常生活的工具。这包括虚拟助手和搜索引擎，以及我们iPhone中的照片工具。

鉴于媒体的炒作和大胆宣称主导着关于神经网络的新闻头条，也许令人惊讶的是，它们自上世纪50年代以来就存在了。它们在2010年后突然变得流行的原因是由于数据和计算能力的不断增长。2011年至2015年之间的ImageNet挑战赛可能是复兴的最大推动力，将对140万张图像进行一千个类别的分类准确率提高到了96.4%。

然而，就像任何机器学习技术一样，它只适用于狭义定义的问题。即使是创建“自动驾驶”汽车的项目也不使用端到端的深度学习，主要使用手工编码的规则系统，卷积神经网络充当“标签制造机”来识别道路上的物体。我们将在本章后面讨论这一点，以了解神经网络实际上是如何使用的。但首先我们将在NumPy中构建一个简单的神经网络，然后使用scikit-learn作为库实现。

# 何时使用神经网络和深度学习

神经网络和深度学习可用于分类和回归，那么它们与线性回归、逻辑回归和其他类型的机器学习相比如何？你可能听说过“当你手中只有一把锤子时，所有事情看起来都像钉子”。每种算法都有其特定情况下的优势和劣势。线性回归、逻辑回归以及梯度提升树（本书未涵盖）在结构化数据上做出了相当出色的预测。将结构化数据视为可以轻松表示为表格的数据，具有行和列。但感知问题如图像分类则不太结构化，因为我们试图找到像素组之间的模糊相关性以识别形状和模式，而不是表格中的数据行。尝试预测正在输入的句子中的下四五个单词，或者解密音频剪辑中的单词，也是感知问题，是神经网络用于自然语言处理的例子。

在本章中，我们将主要关注只有一个隐藏层的简单神经网络。

# 使用神经网络是否有些大材小用？

对于即将介绍的例子来说，使用神经网络可能有些大材小用，因为逻辑回归可能更实用。甚至可以使用[公式方法](https://oreil.ly/M4W8i)。然而，我一直是一个喜欢通过将复杂技术应用于简单问题来理解的人。你可以了解技术的优势和局限性，而不会被大型数据集所分散注意力。因此，请尽量不要在更实用的情况下使用神经网络。为了理解技术，我们将在本章中打破这个规则。

# 一个简单的神经网络

这里有一个简单的例子，让你对神经网络有所了解。我想要预测给定颜色背景下的字体应该是浅色（1）还是深色（0）。以下是不同背景颜色的几个示例，见[图 7-1](#bQLVHMTtrn)。顶部一行最适合浅色字体，底部一行最适合深色字体。

![emds 0701](Images/emds_0701.png)

###### 图 7-1\. 浅色背景颜色最适合深色字体，而深色背景颜色最适合浅色字体

在计算机科学中，表示颜色的一种方式是使用RGB值，即红色、绿色和蓝色值。每个值都介于0和255之间，表示这三种颜色如何混合以创建所需的颜色。例如，如果我们将RGB表示为（红色，绿色，蓝色），那么深橙色的RGB值为（255,140,0），粉色为（255,192,203）。黑色为（0,0,0），白色为（255,255,255）。

从机器学习和回归的角度来看，我们有三个数值输入变量`red`、`green`和`blue`来捕捉给定背景颜色。我们需要对这些输入变量拟合一个函数，并输出是否应该为该背景颜色使用浅色（1）或深色（0）字体。

# 通过RGB表示颜色

在线有数百种颜色选择器调色板可供尝试RGB值。W3 Schools有一个[这里](https://oreil.ly/T57gu)。

请注意，这个例子与神经网络识别图像的工作原理并不相去甚远，因为每个像素通常被建模为三个数值RGB值。在这种情况下，我们只关注一个“像素”作为背景颜色。

让我们从高层次开始，把所有的实现细节放在一边。我们将以洋葱的方式来处理这个主题，从更高的理解开始，然后慢慢剥离细节。目前，这就是为什么我们简单地将一个接受输入并产生输出的过程标记为“神秘数学”。我们有三个数值输入变量R、G和B，这些变量被这个神秘的数学处理。然后它输出一个介于0和1之间的预测，如[图7-2](#VtUKGwbfou)所示。

![emds 0702](Images/emds_0702.png)

###### 图7-2。我们有三个数值RGB值用于预测使用浅色或深色字体

这个预测输出表示一个概率。输出概率是使用神经网络进行分类的最常见模型。一旦我们用它们的数值替换RGB，我们会发现小于0.5会建议使用深色字体，而大于0.5会建议使用浅色字体，如[图7-3](#jnOWiEgusW)所示。

![emds 0703](Images/emds_0703.png)

###### 图7-3。如果我们输入一个粉色的背景色（255,192,203），那么神秘的数学会推荐使用浅色字体，因为输出概率0.89大于0.5

那个神秘的数学黑匣子里到底发生了什么？让我们在[图7-4](#sGQdjdjUMw)中看一看。

我们还缺少神经网络的另一个部分，即激活函数，但我们很快会讨论到。让我们先了解这里发生了什么。左侧的第一层只是三个变量的输入，这些变量在这种情况下是红色、绿色和蓝色值。在隐藏（中间）层中，请注意我们在输入和输出之间产生了三个*节点*，或者说是权重和偏置的函数。每个节点本质上是一个线性函数，斜率为<math alttext="upper W Subscript i"><msub><mi>W</mi> <mi>i</mi></msub></math>，截距为<math alttext="upper B Subscript i"><msub><mi>B</mi> <mi>i</mi></msub></math>，与输入变量<math alttext="upper X Subscript i"><msub><mi>X</mi> <mi>i</mi></msub></math>相乘并求和。每个输入节点和隐藏节点之间有一个权重<math alttext="upper W Subscript i"><msub><mi>W</mi> <mi>i</mi></msub></math>，每个隐藏节点和输出节点之间有另一组权重。每个隐藏和输出节点都会额外添加一个偏置<math alttext="upper B Subscript i"><msub><mi>B</mi> <mi>i</mi></msub></math>。

![emds 0704](Images/emds_0704.png)

###### 图7-4。神经网络的隐藏层对每个输入变量应用权重和偏置值，输出层对该输出应用另一组权重和偏置

注意，输出节点重复执行相同的操作，将隐藏层的加权和求和输出作为输入传递到最终层，其中另一组权重和偏置将被应用。

简而言之，这是一个回归问题，就像线性回归或逻辑回归一样，但需要解决更多参数。权重和偏置值类似于*m*和*b*，或者线性回归中的<math alttext="beta 1"><msub><mi>β</mi> <mn>1</mn></msub></math>和<math alttext="beta 0"><msub><mi>β</mi> <mn>0</mn></msub></math>参数。我们使用随机梯度下降和最小化损失，就像线性回归一样，但我们需要一种称为反向传播的额外工具来解开权重<math alttext="upper W Subscript i"><msub><mi>W</mi> <mi>i</mi></msub></math>和偏置<math alttext="upper B Subscript i"><msub><mi>B</mi> <mi>i</mi></msub></math>值，并使用链式法则计算它们的偏导数。我们将在本章后面详细讨论这一点，但现在让我们假设我们已经优化了权重和偏置值。我们需要先讨论激活函数。

## 激活函数

接下来让我们介绍激活函数。*激活函数*是一个非线性函数，它转换或压缩节点中的加权和值，帮助神经网络有效地分离数据，以便进行分类。让我们看一下[图7-5](#PvLebFIsiT)。如果没有激活函数，你的隐藏层将毫无生产力，表现不会比线性回归好。

![emds 0705](Images/emds_0705.png)

###### 图 7-5\. 应用激活函数

*ReLU 激活函数*将隐藏节点的任何负输出归零。如果权重、偏置和输入相乘并求和得到负数，它将被转换为 0。否则输出保持不变。这是使用 SymPy ([示例 7-1](#WIqRPnWuNe)) 绘制的 ReLU 图（[图 7-6](#tKkerIrVkt))。

##### 示例 7-1\. 绘制 ReLU 函数

```py
from sympy import *

# plot relu
x = symbols('x')
relu = Max(0, x)
plot(relu)
```

![emds 0706](Images/emds_0706.png)

###### 图 7-6\. ReLU 函数图

ReLU 是“修正线性单元”的缩写，但这只是一种将负值转换为 0 的花哨方式。ReLU 在神经网络和深度学习中的隐藏层中变得流行，因为它速度快，并且缓解了[梯度消失问题](https://oreil.ly/QGlM7)。梯度消失发生在偏导数斜率变得非常小，导致过早接近 0 并使训练停滞。

输出层有一个重要的任务：它接收来自神经网络隐藏层的大量数学，并将其转换为可解释的结果，例如呈现分类预测。对于这个特定的神经网络，输出层使用*逻辑激活函数*，这是一个简单的 S 形曲线。如果您阅读[第 6 章](ch06.xhtml#ch06)，逻辑（或 S 形）函数应该很熟悉，它表明逻辑回归在我们的神经网络中充当一层。输出节点的权重、偏置和从隐藏层传入的每个值求和。之后，它通过逻辑函数传递结果值，以便输出介于 0 和 1 之间的数字。就像[第 6 章](ch06.xhtml#ch06)中的逻辑回归一样，这代表了输入到神经网络的给定颜色建议使用浅色字体的概率。如果大于或等于 0.5，则神经网络建议使用浅色字体，否则建议使用深色字体。

这是使用 SymPy ([示例 7-2](#EFehQtrsWp)) 绘制的逻辑函数图（[图 7-7](#kIPouDqVgq)）。

##### 示例 7-2\. SymPy 中的逻辑激活函数

```py
from sympy import *

# plot logistic
x = symbols('x')
logistic = 1 / (1 + exp(-x))
plot(logistic)
```

![emds 0707](Images/emds_0707.png)

###### 图 7-7\. 逻辑激活函数

当我们通过激活函数传递节点的加权、偏置和求和值时，我们现在称之为*激活输出*，意味着它已通过激活函数进行了过滤。当激活输出离开隐藏层时，信号准备好被馈送到下一层。激活函数可能会增强、减弱或保持信号不变。这就是神经网络中大脑和突触的比喻的来源。

鉴于复杂性的潜在可能性，您可能想知道是否还有其他激活函数。一些常见的激活函数显示在[表 7-1](#GhMbfHKelT)中。

表 7-1\. 常见激活函数

| 名称 | 典型使用层 | 描述 | 注释 |
| --- | --- | --- | --- |
| 线性 | 输出 | 保持值不变 | 不常用 |
| Logistic | 输出层 | S 形 sigmoid 曲线 | 将值压缩在 0 和 1 之间，通常用于二元分类 |
| 双曲正切 | 隐藏层 | tanh，在 -1 和 1 之间的 S 形 sigmoid 曲线 | 通过将均值接近 0 来“居中”数据 |
| ReLU | 隐藏层 | 将负值转换为 0 | 比 sigmoid 和 tanh 更快的流行激活函数，缓解消失梯度问题，计算成本低廉 |
| Leaky ReLU | 隐藏层 | 将负值乘以 0.01 | ReLU 的有争议变体，边缘化而不是消除负值 |
| Softmax | 输出层 | 确保所有输出节点加起来为 1.0 | 适用于多类别分类，重新缩放输出使其加起来为 1.0 |

这不是激活函数的全面列表，理论上神经网络中任何函数都可以是激活函数。

尽管这个神经网络表面上支持两类（浅色或深色字体），但实际上它被建模为一类：字体是否应该是浅色（1）或不是（0）。如果您想支持多个类别，可以为每个类别添加更多输出节点。例如，如果您试图识别手写数字 0-9，将有 10 个输出节点，代表给定图像是这些数字的概率。当有多个类别时，您可能还考虑在输出时使用 softmax 激活。[图 7-8](#uPTTSNePsO) 展示了一个以像素化图像为输入的示例，其中像素被分解为单独的神经网络输入，然后通过两个中间层，最后一个输出层，有 10 个节点代表 10 个类别的概率（数字 0-9）。

![emds 0708](Images/emds_0708.png)

###### 图 7-8\. 一个神经网络，将每个像素作为输入，并预测图像包含的数字

在神经网络上使用 MNIST 数据集的示例可以在 [附录 A](app01.xhtml#appendix) 中找到。

# 我不知道要使用什么激活函数！

如果不确定要使用哪些激活函数，当前最佳实践倾向于在中间层使用 ReLU，在输出层使用 logistic（sigmoid）。如果输出中有多个分类，可以在输出层使用 softmax。

## 前向传播

让我们使用 NumPy 捕获到目前为止学到的知识。请注意，我尚未优化参数（我们的权重和偏置值）。我们将用随机值初始化它们。

[示例 7-3](#OMrNeTihfU) 是创建一个简单的前馈神经网络的 Python 代码，尚未进行优化。*前馈* 意味着我们只是将一种颜色输入到神经网络中，看看它输出什么。权重和偏置是随机初始化的，并将在本章后面进行优化，因此暂时不要期望有用的输出。

##### 示例 7-3\. 一个具有随机权重和偏置值的简单前向传播网络

```py
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split

all_data = pd.read_csv("https://tinyurl.com/y2qmhfsr")

# Extract the input columns, scale down by 255
all_inputs = (all_data.iloc[:, 0:3].values / 255.0)
all_outputs = all_data.iloc[:, -1].values

# Split train and test data sets
X_train, X_test, Y_train, Y_test = train_test_split(all_inputs, all_outputs,
    test_size=1/3)
n = X_train.shape[0] # number of training records

# Build neural network with weights and biases
# with random initialization
w_hidden = np.random.rand(3, 3)
w_output = np.random.rand(1, 3)

b_hidden = np.random.rand(3, 1)
b_output = np.random.rand(1, 1)

# Activation functions
relu = lambda x: np.maximum(x, 0)
logistic = lambda x: 1 / (1 + np.exp(-x))

# Runs inputs through the neural network to get predicted outputs
def forward_prop(X):
    Z1 = w_hidden @ X + b_hidden
    A1 = relu(Z1)
    Z2 = w_output @ A1 + b_output
    A2 = logistic(Z2)
    return Z1, A1, Z2, A2

# Calculate accuracy
test_predictions = forward_prop(X_test.transpose())[3] # grab only output layer, A2
test_comparisons = np.equal((test_predictions >= .5).flatten().astype(int), Y_test)
accuracy = sum(test_comparisons.astype(int) / X_test.shape[0])
print("ACCURACY: ", accuracy)
```

这里有几点需要注意。 包含 RGB 输入值以及输出值（1 代表亮，0 代表暗）的数据集包含在[此 CSV 文件](https://oreil.ly/1TZIK)中。 我将输入列 R、G 和 B 的值缩小了 1/255 的因子，使它们介于 0 和 1 之间。 这将有助于后续的训练，以便压缩数字空间。

请注意，我还使用 scikit-learn 将数据的 2/3 用于训练，1/3 用于测试，我们在[第 5 章](ch05.xhtml#ch05)中学习了如何做到这一点。 `n` 简单地是训练数据记录的数量。

现在请注意[示例 7-4](#omJOMqoOMh)中显示的代码行。

##### 示例 7-4。 NumPy 中的权重矩阵和偏置向量

```py
# Build neural network with weights and biases
# with random initialization
w_hidden = np.random.rand(3, 3)
w_output = np.random.rand(1, 3)

b_hidden = np.random.rand(3, 1)
b_output = np.random.rand(1, 1)
```

这些声明了我们神经网络隐藏层和输出层的权重和偏置。 这可能还不明显，但矩阵乘法将使我们的代码变得强大简单，使用线性代数和 NumPy。

权重和偏置将被初始化为介于 0 和 1 之间的随机值。 让我们首先看一下权重矩阵。 当我运行代码时，我得到了这些矩阵：

<math display="block"><mrow><msub><mi>W</mi> <mrow><mi>h</mi><mi>i</mi><mi>d</mi><mi>d</mi><mi>e</mi><mi>n</mi></mrow></msub> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mn>0.034535</mn></mrow></mtd> <mtd><mrow><mn>0.5185636</mn></mrow></mtd> <mtd><mrow><mn>0.81485028</mn></mrow></mtd></mtr> <mtr><mtd><mrow><mn>0.3329199</mn></mrow></mtd> <mtd><mrow><mn>0.53873853</mn></mrow></mtd> <mtd><mrow><mn>0.96359003</mn></mrow></mtd></mtr> <mtr><mtd><mrow><mn>0.19808306</mn></mrow></mtd> <mtd><mrow><mn>0.45422182</mn></mrow></mtd> <mtd><mrow><mn>0.36618893</mn></mrow></mtd></mtr></mtable></mfenced></mrow></math><math display="block"><mrow><msub><mi>W</mi> <mrow><mi>o</mi><mi>u</mi><mi>t</mi><mi>p</mi><mi>u</mi><mi>t</mi></mrow></msub> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mn>0.82652072</mn></mrow></mtd> <mtd><mrow><mn>0.30781539</mn></mrow></mtd> <mtd><mrow><mn>0.93095565</mn></mrow></mtd></mtr></mtable></mfenced></mrow></math>

请注意，<math alttext="upper W Subscript h i d d e n"><msub><mi>W</mi> <mrow><mi>h</mi><mi>i</mi><mi>d</mi><mi>d</mi><mi>e</mi><mi>n</mi></mrow></msub></math> 是隐藏层中的权重。第一行代表第一个节点的权重 <math alttext="upper W 1"><msub><mi>W</mi> <mn>1</mn></msub></math>，<math alttext="upper W 2"><msub><mi>W</mi> <mn>2</mn></msub></math> 和 <math alttext="upper W 3"><msub><mi>W</mi> <mn>3</mn></msub></math>。第二行是第二个节点，带有权重 <math alttext="upper W 4"><msub><mi>W</mi> <mn>4</mn></msub></math>，<math alttext="upper W 5"><msub><mi>W</mi> <mn>5</mn></msub></math> 和 <math alttext="upper W 6"><msub><mi>W</mi> <mn>6</mn></msub></math>。第三行是第三个节点，带有权重 <math alttext="upper W 7"><msub><mi>W</mi> <mn>7</mn></msub></math>，<math alttext="upper W 8"><msub><mi>W</mi> <mn>8</mn></msub></math> 和 <math alttext="upper W 9"><msub><mi>W</mi> <mn>9</mn></msub></math>。

输出层只有一个节点，意味着其矩阵只有一行，带有权重 <math alttext="upper W 10"><msub><mi>W</mi> <mn>10</mn></msub></math>，<math alttext="upper W 11"><msub><mi>W</mi> <mn>11</mn></msub></math> 和 <math alttext="upper W 12"><msub><mi>W</mi> <mn>12</mn></msub></math>。

看到规律了吗？每个节点在矩阵中表示为一行。如果有三个节点，就有三行。如果只有一个节点，就只有一行。每一列保存着该节点的权重值。

让我们也看看偏差。由于每个节点有一个偏差，隐藏层将有三行偏差，输出层将有一行偏差。每个节点只有一个偏差，所以只有一列：

<math display="block"><mrow><msub><mi>B</mi> <mrow><mi>h</mi><mi>i</mi><mi>d</mi><mi>d</mi><mi>e</mi><mi>n</mi></mrow></msub> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mn>0.41379442</mn></mrow></mtd></mtr> <mtr><mtd><mrow><mn>0.81666079</mn></mrow></mtd></mtr> <mtr><mtd><mrow><mn>0.07511252</mn></mrow></mtd></mtr></mtable></mfenced></mrow></math><math display="block"><mrow><msub><mi>B</mi> <mrow><mi>o</mi><mi>u</mi><mi>t</mi><mi>p</mi><mi>u</mi><mi>t</mi></mrow></msub> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mn>0.58018555</mn></mrow></mtd></mtr></mtable></mfenced></mrow></math>

现在让我们将这些矩阵值与我们在 [图 7-9](#vSdgvskUoQ) 中展示的神经网络进行比较。

![emds 0709](Images/emds_0709.png)

###### 图 7-9。将我们的神经网络与权重和偏差矩阵值进行可视化

那么除了紧凑难懂之外，这种矩阵形式中的权重和偏差有什么好处呢？让我们把注意力转向 [示例 7-5](#cKHHrQlImr) 中的这些代码行。

##### 示例 7-5。我们神经网络的激活函数和前向传播函数

```py
# Activation functions
relu = lambda x: np.maximum(x, 0)
logistic = lambda x: 1 / (1 + np.exp(-x))

# Runs inputs through the neural network to get predicted outputs
def forward_prop(X):
    Z1 = w_hidden @ X + b_hidden
    A1 = relu(Z1)
    Z2 = w_output @ A1 + b_output
    A2 = logistic(Z2)
    return Z1, A1, Z2, A2
```

这段代码很重要，因为它使用矩阵乘法和矩阵-向量乘法简洁地执行我们整个神经网络。我们在[第4章](ch04.xhtml#ch04)中学习了这些操作。它仅用几行代码将三个RGB输入的颜色通过权重、偏置和激活函数运行。

首先，我声明`relu()`和`logistic()`激活函数，它们分别接受给定的输入值并返回曲线的输出值。`forward_prop()`函数为包含R、G和B值的给定颜色输入`X`执行整个神经网络。它返回四个阶段的矩阵输出：`Z1`、`A1`、`Z2`和`A2`。数字“1”和“2”表示操作属于第1层和第2层。“Z”表示来自该层的未激活输出，“A”是来自该层的激活输出。

隐藏层由`Z1`和`A1`表示。`Z1`是应用于`X`的权重和偏置。然后`A1`获取来自`Z1`的输出，并通过激活ReLU函数。`Z2`获取来自`A1`的输出，并应用输出层的权重和偏置。该输出依次通过激活函数，即逻辑曲线，变为`A2`。最终阶段，`A2`，是输出层的预测概率，返回一个介于0和1之间的值。我们称之为`A2`，因为它是来自第2层的“激活”输出。

让我们更详细地分解这个，从`Z1`开始：

<math alttext="upper Z 1 equals upper W Subscript h i d d e n Baseline upper X plus upper B Subscript h i d d e n" display="block"><mrow><msub><mi>Z</mi> <mn>1</mn></msub> <mo>=</mo> <msub><mi>W</mi> <mrow><mi>h</mi><mi>i</mi><mi>d</mi><mi>d</mi><mi>e</mi><mi>n</mi></mrow></msub> <mi>X</mi> <mo>+</mo> <msub><mi>B</mi> <mrow><mi>h</mi><mi>i</mi><mi>d</mi><mi>d</mi><mi>e</mi><mi>n</mi></mrow></msub></mrow></math>

首先，我们在<math alttext="upper W Subscript h i d d e n"><msub><mi>W</mi> <mrow><mi>h</mi><mi>i</mi><mi>d</mi><mi>d</mi><mi>e</mi><mi>n</mi></mrow></msub></math>和输入颜色`X`之间执行矩阵-向量乘法。我们将<math alttext="upper W Subscript h i d d e n"><msub><mi>W</mi> <mrow><mi>h</mi><mi>i</mi><mi>d</mi><mi>d</mi><mi>e</mi><mi>n</mi></mrow></msub></math>的每一行（每一行都是一个节点的权重集）与向量`X`（RGB颜色输入值）相乘。然后将偏置添加到该结果中，如[图 7-10](#jPQPcShSLf)所示。

![emds 0710](Images/emds_0710.png)

###### 图7-10。将隐藏层权重和偏置应用于输入`X`，使用矩阵-向量乘法以及向量加法

那个<math alttext="upper Z 1"><msub><mi>Z</mi> <mn>1</mn></msub></math>向量是隐藏层的原始输出，但我们仍然需要通过激活函数将<math alttext="upper Z 1"><msub><mi>Z</mi> <mn>1</mn></msub></math>转换为<math alttext="upper A 1"><msub><mi>A</mi> <mn>1</mn></msub></math>。很简单。只需将该向量中的每个值通过ReLU函数传递，就会给我们<math alttext="upper A 1"><msub><mi>A</mi> <mn>1</mn></msub></math>。因为所有值都是正值，所以不应该有影响。

<math alttext="upper A 1 equals upper R e upper L upper U left-parenthesis upper Z 1 right-parenthesis" display="block"><mrow><msub><mi>A</mi> <mn>1</mn></msub> <mo>=</mo> <mi>R</mi> <mi>e</mi> <mi>L</mi> <mi>U</mi> <mrow><mo>(</mo> <msub><mi>Z</mi> <mn>1</mn></msub> <mo>)</mo></mrow></mrow></math><math display="block"><mrow><msub><mi>A</mi> <mn>1</mn></msub> <mo>=</mo> <mfenced separators="" open="[" close="]"><mtable><mtr><mtd><mrow><mi>R</mi> <mi>e</mi> <mi>L</mi> <mi>U</mi> <mo>(</mo> <mn>1.36054964190909</mn> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd><mrow><mi>R</mi> <mi>e</mi> <mi>L</mi> <mi>U</mi> <mo>(</mo> <mn>2.15471757888247</mn> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd><mrow><mi>R</mi> <mi>e</mi> <mi>L</mi> <mi>U</mi> <mo>(</mo> <mn>0.719554393391768</mn> <mo>)</mo></mrow></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced separators="" open="[" close="]"><mtable><mtr><mtd><mrow><mn>1.36054964190909</mn></mrow></mtd></mtr> <mtr><mtd><mrow><mn>2.15471757888247</mn></mrow></mtd></mtr> <mtr><mtd><mrow><mn>0.719554393391768</mn></mrow></mtd></mtr></mtable></mfenced></mrow></math>

现在让我们将隐藏层输出<math alttext="upper A 1"><msub><mi>A</mi> <mn>1</mn></msub></math>传递到最终层，得到<math alttext="upper Z 2"><msub><mi>Z</mi> <mn>2</mn></msub></math>，然后<math alttext="upper A 2"><msub><mi>A</mi> <mn>2</mn></msub></math>。 <math alttext="upper A 1"><msub><mi>A</mi> <mn>1</mn></msub></math> 成为输出层的输入。

<math alttext="upper Z 2 equals upper W Subscript o u t p u t Baseline upper A 1 plus upper B Subscript o u t p u t" display="block"><mrow><msub><mi>Z</mi> <mn>2</mn></msub> <mo>=</mo> <msub><mi>W</mi> <mrow><mi>o</mi><mi>u</mi><mi>t</mi><mi>p</mi><mi>u</mi><mi>t</mi></mrow></msub> <msub><mi>A</mi> <mn>1</mn></msub> <mo>+</mo> <msub><mi>B</mi> <mrow><mi>o</mi><mi>u</mi><mi>t</mi><mi>p</mi><mi>u</mi><mi>t</mi></mrow></msub></mrow></math><math display="block"><mrow><msub><mi>Z</mi> <mn>2</mn></msub> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mn>0.82652072</mn></mrow></mtd> <mtd><mrow><mn>0.3078159</mn></mrow></mtd> <mtd><mrow><mn>0.93095565</mn></mrow></mtd></mtr></mtable></mfenced> <mfenced separators="" open="[" close="]"><mtable><mtr><mtd><mrow><mn>1.36054964190909</mn></mrow></mtd></mtr> <mtr><mtd><mrow><mn>2.15471757888247</mn></mrow></mtd></mtr> <mtr><mtd><mrow><mn>0.719554393391768</mn></mrow></mtd></mtr></mtable></mfenced> <mo>+</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mn>0.58018555</mn></mrow></mtd></mtr></mtable></mfenced></mrow></math><math display="block"><mrow><msub><mi>Z</mi> <mn>2</mn></msub> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mn>2.45765202842636</mn></mrow></mtd></mtr></mtable></mfenced> <mo>+</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mn>0.58018555</mn></mrow></mtd></mtr></mtable></mfenced></mrow></math><math display="block"><mrow><msub><mi>Z</mi> <mn>2</mn></msub> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mn>3.03783757842636</mn></mrow></mtd></mtr></mtable></mfenced></mrow></math>

最后，将这个单个值<math alttext="upper Z 2"><msub><mi>Z</mi> <mn>2</mn></msub></math>通过激活函数传递，得到<math alttext="upper A 2"><msub><mi>A</mi> <mn>2</mn></msub></math>。这将产生一个约为0.95425的预测：

<math display="block" class="mathml_bottom_space"><mrow><msub><mi>A</mi> <mn>2</mn></msub> <mo>=</mo> <mi>l</mi> <mi>o</mi> <mi>g</mi> <mi>i</mi> <mi>s</mi> <mi>t</mi> <mi>i</mi> <mi>c</mi> <mrow><mo>(</mo> <msub><mi>Z</mi> <mn>2</mn></msub> <mo>)</mo></mrow></mrow></math> <math display="block" class="mathml_bottom_space"><mrow><msub><mi>A</mi> <mn>2</mn></msub> <mo>=</mo> <mi>l</mi> <mi>o</mi> <mi>g</mi> <mi>i</mi> <mi>s</mi> <mi>t</mi> <mi>i</mi> <mi>c</mi> <mrow><mo>(</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mn>3.0378364795204</mn></mrow></mtd></mtr></mtable></mfenced> <mo>)</mo></mrow></mrow></math> <math display="block"><mrow><msub><mi>A</mi> <mn>2</mn></msub> <mo>=</mo> <mn>0.954254478103241</mn></mrow></math>

执行我们整个神经网络，尽管我们尚未对其进行训练。但请花点时间欣赏，我们已经将所有这些输入值、权重、偏差和非线性函数转化为一个将提供预测的单个值。

再次强调，`A2`是最终输出，用于预测背景颜色是否需要浅色（1）或深色（1）字体。尽管我们的权重和偏差尚未优化，让我们按照[示例 7-6](#mLlirJipiN)计算准确率。取测试数据集`X_test`，转置它，并通过`forward_prop()`函数传递，但只获取`A2`向量，其中包含每个测试颜色的预测。然后将预测与实际值进行比较，并计算正确预测的百分比。

##### 示例 7-6\. 计算准确率

```py
# Calculate accuracy
test_predictions = forward_prop(X_test.transpose())[3]  # grab only A2
test_comparisons = np.equal((test_predictions >= .5).flatten().astype(int), Y_test)
accuracy = sum(test_comparisons.astype(int) / X_test.shape[0])
print("ACCURACY: ", accuracy)
```

当我运行[示例 7-3](#OMrNeTihfU)中的整个代码时，我大致获得55%到67%的准确率。请记住，权重和偏差是随机生成的，因此答案会有所不同。虽然这个准确率似乎很高，考虑到参数是随机生成的，但请记住，输出预测是二元的：浅色或深色。因此，对于每个预测，随机抛硬币也可能产生这种结果，所以这个数字不应令人惊讶。

# 不要忘记检查是否存在数据不平衡！

正如在[第6章](ch06.xhtml#ch06)中讨论的那样，不要忘记分析数据以检查是否存在不平衡的类别。整个背景颜色数据集有点不平衡：512种颜色的输出为0，833种颜色的输出为1。这可能会使准确率产生偏差，也可能是我们的随机权重和偏差倾向于高于50%准确率的原因。如果数据极度不平衡（例如99%的数据属于一类），请记住使用混淆矩阵来跟踪假阳性和假阴性。

到目前为止，结构上一切都合理吗？在继续之前，随时可以回顾一切。我们只剩下最后一个部分要讲解：优化权重和偏差。去冲杯浓缩咖啡或氮气咖啡吧，因为这是本书中我们将要做的最复杂的数学！

# 反向传播

在我们开始使用随机梯度下降来优化我们的神经网络之前，我们面临的挑战是如何相应地改变每个权重和偏差值，尽管它们都纠缠在一起以创建输出变量，然后用于计算残差。我们如何找到每个权重<math alttext="upper W Subscript i"><msub><mi>W</mi> <mi>i</mi></msub></math> 和偏差<math alttext="upper B Subscript i"><msub><mi>B</mi> <mi>i</mi></msub></math> 变量的导数？我们需要使用我们在[第1章](ch01.xhtml#ch01)中讨论过的链式法则。

## 计算权重和偏差的导数

我们还没有准备好应用随机梯度下降来训练我们的神经网络。我们需要求出相对于权重<math alttext="upper W Subscript i"><msub><mi>W</mi> <mi>i</mi></msub></math> 和偏差<math alttext="upper B Subscript i"><msub><mi>B</mi> <mi>i</mi></msub></math> 的偏导数，而且我们有链式法则来帮助我们。

虽然过程基本相同，但在神经网络上使用随机梯度下降存在一个复杂性。一层中的节点将它们的权重和偏置传递到下一层，然后应用另一组权重和偏置。这创建了一个类似洋葱的嵌套结构，我们需要从输出层开始解开。

在梯度下降过程中，我们需要找出应该调整哪些权重和偏置，以及调整多少，以减少整体成本函数。单个预测的成本将是神经网络的平方输出<math alttext="upper A 2"><msub><mi>A</mi> <mn>2</mn></msub></math>减去实际值<math alttext="upper Y"><mi>Y</mi></math>：

<math alttext="upper C equals left-parenthesis upper A 2 minus upper Y right-parenthesis squared" display="block"><mrow><mi>C</mi> <mo>=</mo> <msup><mrow><mo>(</mo><msub><mi>A</mi> <mn>2</mn></msub> <mo>-</mo><mi>Y</mi><mo>)</mo></mrow> <mn>2</mn></msup></mrow></math>

但让我们再往下一层。那个激活输出<math alttext="upper A 2"><msub><mi>A</mi> <mn>2</mn></msub></math>只是带有激活函数的<math alttext="upper Z 2"><msub><mi>Z</mi> <mn>2</mn></msub></math>：

<math alttext="upper A 2 equals s i g m o i d left-parenthesis upper Z 2 right-parenthesis" display="block"><mrow><msub><mi>A</mi> <mn>2</mn></msub> <mo>=</mo> <mi>s</mi> <mi>i</mi> <mi>g</mi> <mi>m</mi> <mi>o</mi> <mi>i</mi> <mi>d</mi> <mrow><mo>(</mo> <msub><mi>Z</mi> <mn>2</mn></msub> <mo>)</mo></mrow></mrow></math>

转而，<math alttext="upper Z 2"><msub><mi>Z</mi> <mn>2</mn></msub></math>是应用于激活输出<math alttext="upper A 1"><msub><mi>A</mi> <mn>1</mn></msub></math>的输出权重和偏置，这些输出来自隐藏层：

<math alttext="upper Z 2 equals upper W 2 upper A 1 plus upper B 2" display="block"><mrow><msub><mi>Z</mi> <mn>2</mn></msub> <mo>=</mo> <msub><mi>W</mi> <mn>2</mn></msub> <msub><mi>A</mi> <mn>1</mn></msub> <mo>+</mo> <msub><mi>B</mi> <mn>2</mn></msub></mrow></math>

<math alttext="upper A 1"><msub><mi>A</mi> <mn>1</mn></msub></math>是由通过ReLU激活函数传递的<math alttext="upper Z 1"><msub><mi>Z</mi> <mn>1</mn></msub></math>构建而成：

<math alttext="upper A 1 equals upper R e upper L upper U left-parenthesis upper Z 1 right-parenthesis" display="block"><mrow><msub><mi>A</mi> <mn>1</mn></msub> <mo>=</mo> <mi>R</mi> <mi>e</mi> <mi>L</mi> <mi>U</mi> <mrow><mo>(</mo> <msub><mi>Z</mi> <mn>1</mn></msub> <mo>)</mo></mrow></mrow></math>

最后，<math alttext="upper Z 1"><msub><mi>Z</mi> <mn>1</mn></msub></math>是由隐藏层加权和偏置的输入x值：

<math alttext="upper Z 1 equals upper W 1 upper X plus upper B 1" display="block"><mrow><msub><mi>Z</mi> <mn>1</mn></msub> <mo>=</mo> <msub><mi>W</mi> <mn>1</mn></msub> <mi>X</mi> <mo>+</mo> <msub><mi>B</mi> <mn>1</mn></msub></mrow></math>

我们需要找到包含在矩阵和向量<math alttext="upper W 1"><msub><mi>W</mi> <mn>1</mn></msub></math>，<math alttext="upper B 1"><msub><mi>B</mi> <mn>1</mn></msub></math>，<math alttext="upper W 2"><msub><mi>W</mi> <mn>2</mn></msub></math>和<math alttext="upper B 2"><msub><mi>B</mi> <mn>2</mn></msub></math>中的权重和偏置，以最小化我们的损失。通过微调它们的斜率，我们可以改变对最小化损失影响最大的权重和偏置。然而，对权重或偏置进行微小调整会一直传播到外层的损失函数。这就是链式法则可以帮助我们找出这种影响的地方。

让我们专注于找到输出层权重<math alttext="upper W 2"><msub><mi>W</mi> <mn>2</mn></msub></math>和成本函数<math alttext="upper C"><mi>C</mi></math>之间的关系。权重<math alttext="upper W 2"><msub><mi>W</mi> <mn>2</mn></msub></math>的变化导致未激活的输出<math alttext="upper Z 2"><msub><mi>Z</mi> <mn>2</mn></msub></math>的变化。然后改变激活输出<math alttext="upper A 2"><msub><mi>A</mi> <mn>2</mn></msub></math>，进而改变成本函数<math alttext="upper C"><mi>C</mi></math>。利用链式法则，我们可以定义关于<math alttext="upper W 2"><msub><mi>W</mi> <mn>2</mn></msub></math>的导数如下：

<math alttext="StartFraction d upper C Over d upper W 2 EndFraction equals StartFraction d upper Z 2 Over d upper W 2 EndFraction StartFraction d upper A 2 Over d upper Z 2 EndFraction StartFraction d upper C Over d upper A 2 EndFraction" display="block"><mrow><mfrac><mrow><mi>d</mi><mi>C</mi></mrow> <mrow><mi>d</mi><msub><mi>W</mi> <mn>2</mn></msub></mrow></mfrac> <mo>=</mo> <mfrac><mrow><mi>d</mi><msub><mi>Z</mi> <mn>2</mn></msub></mrow> <mrow><mi>d</mi><msub><mi>W</mi> <mn>2</mn></msub></mrow></mfrac> <mfrac><mrow><mi>d</mi><msub><mi>A</mi> <mn>2</mn></msub></mrow> <mrow><mi>d</mi><msub><mi>Z</mi> <mn>2</mn></msub></mrow></mfrac> <mfrac><mrow><mi>d</mi><mi>C</mi></mrow> <mrow><mi>d</mi><msub><mi>A</mi> <mn>2</mn></msub></mrow></mfrac></mrow></math>

当我们将这三个梯度相乘在一起时，我们得到了改变<math alttext="upper W 2"><msub><mi>W</mi> <mn>2</mn></msub></math>将如何改变成本函数<math alttext="upper C"><mi>C</mi></math>的度量。

现在我们将计算这三个导数。让我们使用SymPy计算在[示例 7-7](#bJeDOpjeQi)中关于<math alttext="upper A 2"><msub><mi>A</mi> <mn>2</mn></msub></math>的成本函数的导数。

<math alttext="StartFraction d upper C Over d upper A 2 EndFraction equals 2 upper A 2 minus 2 y" display="block"><mrow><mfrac><mrow><mi>d</mi><mi>C</mi></mrow> <mrow><mi>d</mi><msub><mi>A</mi> <mn>2</mn></msub></mrow></mfrac> <mo>=</mo> <mn>2</mn> <msub><mi>A</mi> <mn>2</mn></msub> <mo>-</mo> <mn>2</mn> <mi>y</mi></mrow></math>

##### 示例 7-7. 计算成本函数关于<math alttext="upper A 2"><msub><mi>A</mi> <mn>2</mn></msub></math>的导数

```py
from sympy import *

A2, y = symbols('A2 Y')
C = (A2 - Y)**2
dC_dA2 = diff(C, A2)
print(dC_dA2) # 2*A2 - 2*Y
```

接下来，让我们找到关于<math alttext="upper Z 2"><msub><mi>Z</mi> <mn>2</mn></msub></math>的导数与<math alttext="upper A 2"><msub><mi>A</mi> <mn>2</mn></msub></math>的导数（[示例 7-8](#FiirvKJUSW)）。记住<math alttext="upper A 2"><msub><mi>A</mi> <mn>2</mn></msub></math>是激活函数的输出，在这种情况下是逻辑函数。因此，我们实际上只是在求取S形曲线的导数。

<math alttext="StartFraction d upper A 2 Over d upper Z 2 EndFraction equals StartFraction e Superscript minus upper Z 2 Baseline Over left-parenthesis 1 plus e Superscript minus upper Z 2 Baseline right-parenthesis squared EndFraction" display="block"><mrow><mfrac><mrow><mi>d</mi><msub><mi>A</mi> <mn>2</mn></msub></mrow> <mrow><mi>d</mi><msub><mi>Z</mi> <mn>2</mn></msub></mrow></mfrac> <mo>=</mo> <mfrac><msup><mi>e</mi> <mrow><mo>-</mo><msub><mi>Z</mi> <mn>2</mn></msub></mrow></msup> <msup><mfenced separators="" open="(" close=")"><mn>1</mn><mo>+</mo><msup><mi>e</mi> <mrow><mo>-</mo><msub><mi>Z</mi> <mn>2</mn></msub></mrow></msup></mfenced> <mn>2</mn></msup></mfrac></mrow></math>

##### 示例 7-8. 找到关于<math alttext="upper Z 2"><msub><mi>Z</mi> <mn>2</mn></msub></math>的导数与<math alttext="upper A 2"><msub><mi>A</mi> <mn>2</mn></msub></math>

```py
from sympy import *

Z2 = symbols('Z2')

logistic = lambda x: 1 / (1 + exp(-x))

A2 = logistic(Z2)
dA2_dZ2 = diff(A2, Z2)
print(dA2_dZ2) # exp(-Z2)/(1 + exp(-Z2))**2
```

<math alttext="upper Z 2"><msub><mi>Z</mi> <mn>2</mn></msub></math>关于<math alttext="upper W 2"><msub><mi>W</mi> <mn>2</mn></msub></math>的导数将会得到<math alttext="upper A 1"><msub><mi>A</mi> <mn>1</mn></msub></math>，因为它只是一个线性函数，将返回斜率（[示例 7-9](#kPubQpquMU)）。

<math alttext="StartFraction d upper Z 2 Over d upper W 1 EndFraction equals upper A 1" display="block"><mrow><mfrac><mrow><mi>d</mi><msub><mi>Z</mi> <mn>2</mn></msub></mrow> <mrow><mi>d</mi><msub><mi>W</mi> <mn>1</mn></msub></mrow></mfrac> <mo>=</mo> <msub><mi>A</mi> <mn>1</mn></msub></mrow></math>

##### 示例 7-9. 关于<math alttext="upper W 2"><msub><mi>W</mi> <mn>2</mn></msub></math>的导数<math alttext="upper Z 2"><msub><mi>Z</mi> <mn>2</mn></msub></math>

```py
from sympy import *

A1, W2, B2 = symbols('A1, W2, B2')

Z2 = A1*W2 + B2
dZ2_dW2 = diff(Z2, W2)
print(dZ2_dW2) # A1
```

将所有内容整合在一起，这里是找到改变<math alttext="upper W 2"><msub><mi>W</mi> <mn>2</mn></msub></math>中的权重会如何影响成本函数<math alttext="upper C"><mi>C</mi></math>的导数：

<math alttext="StartFraction d upper C Over d w 2 EndFraction equals StartFraction d upper Z 2 Over d w 2 EndFraction StartFraction d upper A 2 Over d upper Z 2 EndFraction StartFraction d upper C Over d upper A 2 EndFraction equals left-parenthesis upper A 1 right-parenthesis left-parenthesis StartFraction e Superscript minus upper Z 2 Baseline Over left-parenthesis 1 plus e Superscript minus upper Z 2 Baseline right-parenthesis squared EndFraction right-parenthesis left-parenthesis 2 upper A 2 minus 2 y right-parenthesis" display="block"><mrow><mfrac><mrow><mi>d</mi><mi>C</mi></mrow> <mrow><mi>d</mi><msub><mi>w</mi> <mn>2</mn></msub></mrow></mfrac> <mo>=</mo> <mfrac><mrow><mi>d</mi><msub><mi>Z</mi> <mn>2</mn></msub></mrow> <mrow><mi>d</mi><msub><mi>w</mi> <mn>2</mn></msub></mrow></mfrac> <mfrac><mrow><mi>d</mi><msub><mi>A</mi> <mn>2</mn></msub></mrow> <mrow><mi>d</mi><msub><mi>Z</mi> <mn>2</mn></msub></mrow></mfrac> <mfrac><mrow><mi>d</mi><mi>C</mi></mrow> <mrow><mi>d</mi><msub><mi>A</mi> <mn>2</mn></msub></mrow></mfrac> <mo>=</mo> <mrow><mo>(</mo> <msub><mi>A</mi> <mn>1</mn></msub> <mo>)</mo></mrow> <mrow><mo>(</mo> <mfrac><msup><mi>e</mi> <mrow><mo>-</mo><msub><mi>Z</mi> <mn>2</mn></msub></mrow></msup> <msup><mfenced separators="" open="(" close=")"><mn>1</mn><mo>+</mo><msup><mi>e</mi> <mrow><mo>-</mo><msub><mi>Z</mi> <mn>2</mn></msub></mrow></msup></mfenced> <mn>2</mn></msup></mfrac> <mo>)</mo></mrow> <mrow><mo>(</mo> <mn>2</mn> <msub><mi>A</mi> <mn>2</mn></msub> <mo>-</mo> <mn>2</mn> <mi>y</mi> <mo>)</mo></mrow></mrow></math>

当我们运行一个输入`X`与三个输入R、G和B值时，我们将得到<math alttext="upper A 1"><msub><mi>A</mi> <mn>1</mn></msub></math>、<math alttext="upper A 2"><msub><mi>A</mi> <mn>2</mn></msub></math>、<math alttext="upper Z 2"><msub><mi>Z</mi> <mn>2</mn></msub></math>和<math alttext="y"><mi>y</mi></math>的值。

# 不要在数学中迷失！

在这一点很容易在数学中迷失并忘记你最初想要实现的目标，即找到成本函数关于输出层中的权重（<math alttext="upper W 2"><msub><mi>W</mi> <mn>2</mn></msub></math>）的导数。当你发现自己陷入困境并忘记你要做什么时，那就退一步，出去走走，喝杯咖啡，提醒自己你要达成的目标。如果你做不到，你应该从头开始，一步步地找回迷失的地方。

然而，这只是神经网络的一个组成部分，对于<math alttext="upper W 2"><msub><mi>W</mi> <mn>2</mn></msub></math>的导数。以下是我们在[示例 7-10](#hqtBpgBaGT)中使用SymPy计算的其余部分偏导数，这些是我们在链式求导中需要的。

##### 示例 7-10。计算我们神经网络所需的所有偏导数

```py
from sympy import *

W1, W2, B1, B2, A1, A2, Z1, Z2, X, Y = \
    symbols('W1 W2 B1 B2 A1 A2 Z1 Z2 X Y')

# Calculate derivative of cost function with respect to A2
C = (A2 - Y)**2
dC_dA2 = diff(C, A2)
print("dC_dA2 = ", dC_dA2) # 2*A2 - 2*Y

# Calculate derivative of A2 with respect to Z2
logistic = lambda x: 1 / (1 + exp(-x))
_A2 = logistic(Z2)
dA2_dZ2 = diff(_A2, Z2)
print("dA2_dZ2 = ", dA2_dZ2) # exp(-Z2)/(1 + exp(-Z2))**2

# Calculate derivative of Z2 with respect to A1
_Z2 = A1*W2 + B2
dZ2_dA1 = diff(_Z2, A1)
print("dZ2_dA1 = ", dZ2_dA1) # W2

# Calculate derivative of Z2 with respect to W2
dZ2_dW2 = diff(_Z2, W2)
print("dZ2_dW2 = ", dZ2_dW2) # A1

# Calculate derivative of Z2 with respect to B2
dZ2_dB2 = diff(_Z2, B2)
print("dZ2_dB2 = ", dZ2_dB2) # 1

# Calculate derivative of A1 with respect to Z1
relu = lambda x: Max(x, 0)
_A1 = relu(Z1)

d_relu = lambda x: x > 0 # Slope is 1 if positive, 0 if negative
dA1_dZ1 = d_relu(Z1)
print("dA1_dZ1 = ", dA1_dZ1) # Z1 > 0

# Calculate derivative of Z1 with respect to W1
_Z1 = X*W1 + B1
dZ1_dW1 = diff(_Z1, W1)
print("dZ1_dW1 = ", dZ1_dW1) # X

# Calculate derivative of Z1 with respect to B1
dZ1_dB1 = diff(_Z1, B1)
print("dZ1_dB1 = ", dZ1_dB1) # 1
```

注意，ReLU是手动计算的，而不是使用SymPy的`diff()`函数。这是因为导数适用于平滑曲线，而不是ReLU上存在的锯齿角。但通过简单地声明斜率为正数为1，负数为0，可以轻松解决这个问题。这是有道理的，因为负数具有斜率为0的平坦线。但正数保持不变，具有1:1的斜率。

这些偏导数可以链接在一起，以创建相对于权重和偏置的新偏导数。让我们为<math alttext="upper W 1"><msub><mi>W</mi> <mn>1</mn></msub></math>、<math alttext="upper W 2"><msub><mi>W</mi> <mn>2</mn></msub></math>、<math alttext="upper B 1"><msub><mi>B</mi> <mn>1</mn></msub></math>和<math alttext="upper B 2"><msub><mi>B</mi> <mn>2</mn></msub></math>相对于成本函数的四个偏导数。我们已经讨论了<math alttext="StartFraction d upper C Over d w 2 EndFraction"><mfrac><mrow><mi>d</mi><mi>C</mi></mrow> <mrow><mi>d</mi><msub><mi>w</mi> <mn>2</mn></msub></mrow></mfrac></math>。让我们将其与其他三个链式求导一起展示：

<math alttext="StartFraction d upper C Over d upper W 2 EndFraction equals StartFraction d upper Z 2 Over d upper W 2 EndFraction StartFraction d upper A 2 Over d upper Z 2 EndFraction StartFraction d upper C Over d upper A 2 EndFraction equals left-parenthesis upper A 1 right-parenthesis left-parenthesis StartFraction e Superscript minus upper Z 2 Baseline Over left-parenthesis 1 plus e Superscript minus upper Z 2 Baseline right-parenthesis squared EndFraction right-parenthesis left-parenthesis 2 upper A 2 minus 2 y right-parenthesis" display="block"><mrow><mfrac><mrow><mi>d</mi><mi>C</mi></mrow> <mrow><mi>d</mi><msub><mi>W</mi> <mn>2</mn></msub></mrow></mfrac> <mo>=</mo> <mfrac><mrow><mi>d</mi><msub><mi>Z</mi> <mn>2</mn></msub></mrow> <mrow><mi>d</mi><msub><mi>W</mi> <mn>2</mn></msub></mrow></mfrac> <mfrac><mrow><mi>d</mi><msub><mi>A</mi> <mn>2</mn></msub></mrow> <mrow><mi>d</mi><msub><mi>Z</mi> <mn>2</mn></msub></mrow></mfrac> <mfrac><mrow><mi>d</mi><mi>C</mi></mrow> <mrow><mi>d</mi><msub><mi>A</mi> <mn>2</mn></msub></mrow></mfrac> <mo>=</mo> <mrow><mo>(</mo> <msub><mi>A</mi> <mn>1</mn></msub> <mo>)</mo></mrow> <mrow><mo>(</mo> <mfrac><msup><mi>e</mi> <mrow><mo>-</mo><msub><mi>Z</mi> <mn>2</mn></msub></mrow></msup> <msup><mfenced separators="" open="(" close=")"><mn>1</mn><mo>+</mo><msup><mi>e</mi> <mrow><mo>-</mo><msub><mi>Z</mi> <mn>2</mn></msub></mrow></msup></mfenced> <mn>2</mn></msup></mfrac> <mo>)</mo></mrow> <mrow><mo>(</mo> <mn>2</mn> <msub><mi>A</mi> <mn>2</mn></msub> <mo>-</mo> <mn>2</mn> <mi>y</mi> <mo>)</mo></mrow></mrow></math><math alttext="StartFraction d upper C Over d upper B 2 EndFraction equals StartFraction d upper Z 2 Over d upper B 2 EndFraction StartFraction d upper A 2 Over d upper Z 2 EndFraction StartFraction d upper C Over d upper A 2 EndFraction equals left-parenthesis 1 right-parenthesis left-parenthesis StartFraction e Superscript minus upper Z 2 Baseline Over left-parenthesis 1 plus e Superscript minus upper Z 2 Baseline right-parenthesis squared EndFraction right-parenthesis left-parenthesis 2 upper A 2 minus 2 y right-parenthesis" display="block"><mrow><mfrac><mrow><mi>d</mi><mi>C</mi></mrow> <mrow><mi>d</mi><msub><mi>B</mi> <mn>2</mn></msub></mrow></mfrac> <mo>=</mo> <mfrac><mrow><mi>d</mi><msub><mi>Z</mi> <mn>2</mn></msub></mrow> <mrow><mi>d</mi><msub><mi>B</mi> <mn>2</mn></msub></mrow></mfrac> <mfrac><mrow><mi>d</mi><msub><mi>A</mi> <mn>2</mn></msub></mrow> <mrow><mi>d</mi><msub><mi>Z</mi> <mn>2</mn></msub></mrow></mfrac> <mfrac><mrow><mi>d</mi><mi>C</mi></mrow> <mrow><mi>d</mi><msub><mi>A</mi> <mn>2</mn></msub></mrow></mfrac> <mo>=</mo> <mrow><mo>(</mo> <mn>1</mn> <mo>)</mo></mrow> <mrow><mo>(</mo> <mfrac><msup><mi>e</mi> <mrow><mo>-</mo><msub><mi>Z</mi> <mn>2</mn></msub></mrow></msup> <msup><mfenced separators="" open="(" close=")"><mn>1</mn><mo>+</mo><msup><mi>e</mi> <mrow><mo>-</mo><msub><mi>Z</mi> <mn>2</mn></msub></mrow></msup></mfenced> <mn>2</mn></msup></mfrac> <mo>)</mo></mrow> <mrow><mo>(</mo> <mn>2</mn> <msub><mi>A</mi> <mn>2</mn></msub> <mo>-</mo> <mn>2</mn> <mi>y</mi> <mo>)</mo></mrow></mrow></math><math alttext="StartFraction d upper C Over d upper W 1 EndFraction equals StartFraction d upper C Over upper D upper A 2 EndFraction StartFraction upper D upper A 2 Over d upper Z 2 EndFraction StartFraction d upper Z 2 Over d upper A 1 EndFraction StartFraction d upper A 1 Over d upper Z 1 EndFraction StartFraction d upper Z 1 Over d upper W 1 EndFraction equals left-parenthesis 2 upper A 2 minus 2 y right-parenthesis left-parenthesis StartFraction e Superscript minus upper Z 2 Baseline Over left-parenthesis 1 plus e Superscript minus upper Z 2 Baseline right-parenthesis squared EndFraction right-parenthesis left-parenthesis upper W 2 right-parenthesis left-parenthesis upper Z 1 greater-than 0 right-parenthesis left-parenthesis upper X right-parenthesis" display="block"><mrow><mfrac><mrow><mi>d</mi><mi>C</mi></mrow> <mrow><mi>d</mi><msub><mi>W</mi> <mn>1</mn></msub></mrow></mfrac> <mo>=</mo> <mfrac><mrow><mi>d</mi><mi>C</mi></mrow> <mrow><mi>D</mi><msub><mi>A</mi> <mn>2</mn></msub></mrow></mfrac> <mfrac><mrow><mi>D</mi><msub><mi>A</mi> <mn>2</mn></msub></mrow> <mrow><mi>d</mi><msub><mi>Z</mi> <mn>2</mn></msub></mrow></mfrac> <mfrac><mrow><mi>d</mi><msub><mi>Z</mi> <mn>2</mn></msub></mrow> <mrow><mi>d</mi><msub><mi>A</mi> <mn>1</mn></msub></mrow></mfrac> <mfrac><mrow><mi>d</mi><msub><mi>A</mi> <mn>1</mn></msub></mrow> <mrow><mi>d</mi><msub><mi>Z</mi> <mn>1</mn></msub></mrow></mfrac> <mfrac><mrow><mi>d</mi><msub><mi>Z</mi> <mn>1</mn></msub></mrow> <mrow><mi>d</mi><msub><mi>W</mi> <mn>1</mn></msub></mrow></mfrac> <mo>=</mo> <mrow><mo>(</mo> <mn>2</mn> <msub><mi>A</mi> <mn>2</mn></msub> <mo>-</mo> <mn>2</mn> <mi>y</mi> <mo>)</mo></mrow> <mrow><mo>(</mo> <mfrac><msup><mi>e</mi> <mrow><mo>-</mo><msub><mi>Z</mi> <mn>2</mn></msub></mrow></msup> <msup><mfenced separators="" open="(" close=")"><mn>1</mn><mo>+</mo><msup><mi>e</mi> <mrow><mo>-</mo><msub><mi>Z</mi> <mn>2</mn

我们将使用这些链式梯度来计算成本函数*C*相对于<math alttext="upper W 1"><msub><mi>W</mi> <mn>1</mn></msub></math>，<math alttext="upper B 1"><msub><mi>B</mi> <mn>1</mn></msub></math>，<math alttext="upper W 2"><msub><mi>W</mi> <mn>2</mn></msub></math>和<math alttext="upper B 2"><msub><mi>B</mi> <mn>2</mn></msub></math>的斜率。

## 随机梯度下降

现在我们准备整合链式法则来执行随机梯度下降。为了保持简单，我们每次迭代只对一个训练记录进行采样。在神经网络和深度学习中通常使用批量梯度下降和小批量梯度下降，但是在每次迭代中只处理一个样本就足够了，因为其中涉及足够的线性代数和微积分。

让我们来看看我们的神经网络的完整实现，使用反向传播的随机梯度下降，在[示例 7-11](#MfDTsSwbfB)中。

##### 示例 7-11\. 使用随机梯度下降实现神经网络

```py
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split

all_data = pd.read_csv("https://tinyurl.com/y2qmhfsr")

# Learning rate controls how slowly we approach a solution
# Make it too small, it will take too long to run.
# Make it too big, it will likely overshoot and miss the solution.
L = 0.05

# Extract the input columns, scale down by 255
all_inputs = (all_data.iloc[:, 0:3].values / 255.0)
all_outputs = all_data.iloc[:, -1].values

# Split train and test data sets
X_train, X_test, Y_train, Y_test = train_test_split(all_inputs, all_outputs,
    test_size=1 / 3)
n = X_train.shape[0]

# Build neural network with weights and biases
# with random initialization
w_hidden = np.random.rand(3, 3)
w_output = np.random.rand(1, 3)

b_hidden = np.random.rand(3, 1)
b_output = np.random.rand(1, 1)

# Activation functions
relu = lambda x: np.maximum(x, 0)
logistic = lambda x: 1 / (1 + np.exp(-x))

# Runs inputs through the neural network to get predicted outputs
def forward_prop(X):
    Z1 = w_hidden @ X + b_hidden
    A1 = relu(Z1)
    Z2 = w_output @ A1 + b_output
    A2 = logistic(Z2)
    return Z1, A1, Z2, A2

# Derivatives of Activation functions
d_relu = lambda x: x > 0
d_logistic = lambda x: np.exp(-x) / (1 + np.exp(-x)) ** 2

# returns slopes for weights and biases
# using chain rule
def backward_prop(Z1, A1, Z2, A2, X, Y):
    dC_dA2 = 2 * A2 - 2 * Y
    dA2_dZ2 = d_logistic(Z2)
    dZ2_dA1 = w_output
    dZ2_dW2 = A1
    dZ2_dB2 = 1
    dA1_dZ1 = d_relu(Z1)
    dZ1_dW1 = X
    dZ1_dB1 = 1

    dC_dW2 = dC_dA2 @ dA2_dZ2 @ dZ2_dW2.T

    dC_dB2 = dC_dA2 @ dA2_dZ2 * dZ2_dB2

    dC_dA1 = dC_dA2 @ dA2_dZ2 @ dZ2_dA1

    dC_dW1 = dC_dA1 @ dA1_dZ1 @ dZ1_dW1.T

    dC_dB1 = dC_dA1 @ dA1_dZ1 * dZ1_dB1

    return dC_dW1, dC_dB1, dC_dW2, dC_dB2

# Execute gradient descent
for i in range(100_000):
    # randomly select one of the training data
    idx = np.random.choice(n, 1, replace=False)
    X_sample = X_train[idx].transpose()
    Y_sample = Y_train[idx]

    # run randomly selected training data through neural network
    Z1, A1, Z2, A2 = forward_prop(X_sample)

    # distribute error through backpropagation
    # and return slopes for weights and biases
    dW1, dB1, dW2, dB2 = backward_prop(Z1, A1, Z2, A2, X_sample, Y_sample)

    # update weights and biases
    w_hidden -= L * dW1
    b_hidden -= L * dB1
    w_output -= L * dW2
    b_output -= L * dB2

# Calculate accuracy
test_predictions = forward_prop(X_test.transpose())[3]  # grab only A2
test_comparisons = np.equal((test_predictions >= .5).flatten().astype(int), Y_test)
accuracy = sum(test_comparisons.astype(int) / X_test.shape[0])
print("ACCURACY: ", accuracy)
```

这里涉及很多内容，但是建立在我们在本章学到的一切基础之上。我们进行了10万次随机梯度下降迭代。将训练和测试数据分别按2/3和1/3划分，根据随机性的不同，我在测试数据集中获得了大约97-99%的准确率。这意味着训练后，我的神经网络能够正确识别97-99%的测试数据，并做出正确的浅色/深色字体预测。

`backward_prop()`函数在这里起着关键作用，实现链式法则，将输出节点的误差（平方残差）分配并向后传播到输出和隐藏权重/偏差，以获得相对于每个权重/偏差的斜率。然后我们在`for`循环中使用这些斜率，分别通过乘以学习率`L`来微调权重/偏差，就像我们在第[5](ch05.xhtml#ch05)章和第[6](ch06.xhtml#ch06)章中所做的那样。我们进行一些矩阵-向量乘法，根据斜率向后传播误差，并在需要时转置矩阵和向量，以使行和列之间的维度匹配起来。

如果你想让神经网络更具交互性，这里有一段代码片段在[示例 7-12](#wjmhutDoNG)中，我们可以输入不同的背景颜色（通过R、G和B值），看看它是否预测为浅色或深色字体。将其附加到之前的代码[示例 7-11](#MfDTsSwbfB)的底部，然后试一试！

##### 示例 7-12\. 为我们的神经网络添加一个交互式shell

```py
# Interact and test with new colors
def predict_probability(r, g, b):
    X = np.array([[r, g, b]]).transpose() / 255
    Z1, A1, Z2, A2 = forward_prop(X)
    return A2

def predict_font_shade(r, g, b):
    output_values = predict_probability(r, g, b)
    if output_values > .5:
        return "DARK"
    else:
        return "LIGHT"

while True:
    col_input = input("Predict light or dark font. Input values R,G,B: ")
    (r, g, b) = col_input.split(",")
    print(predict_font_shade(int(r), int(g), int(b)))
```

从头开始构建自己的神经网络需要大量的工作和数学知识，但它让你深入了解它们的真实本质。通过逐层工作、微积分和线性代数，我们对像PyTorch和TensorFlow这样的深度学习库在幕后做了什么有了更深刻的理解。

从阅读本整章节中你已经了解到，要使神经网络正常运转有很多要素。在代码的不同部分设置断点可以帮助你了解每个矩阵操作在做什么。你也可以将代码移植到 Jupyter Notebook 中，以获得更多对每个步骤的视觉洞察。

# 3Blue1Brown 关于反向传播的视频

3Blue1Brown有一些经典视频讨论[反向传播](https://youtu.be/Ilg3gGewQ5U)和神经网络背后的[微积分](https://youtu.be/tIeHLnjs5U8)。

# 使用 scikit-learn

scikit-learn 中有一些有限的神经网络功能。如果你对深度学习感兴趣，你可能会想学习 PyTorch 或 TensorFlow，并购买一台配备强大 GPU 的计算机（这是购买你一直想要的游戏电脑的绝佳借口！）。我被告知现在所有酷炫的孩子都在使用 PyTorch。然而，scikit-learn 中确实有一些方便的模型可用，包括`MLPClassifier`，它代表“多层感知器分类器”。这是一个用于分类的神经网络，默认使用逻辑输出激活。

[示例 7-13](#rchKuwCjUt)是我们开发的背景颜色分类应用的 scikit-learn 版本。`activation`参数指定了隐藏层。

##### 示例 7-13\. 使用 scikit-learn 神经网络分类器

```py
import pandas as pd
# load data
from sklearn.model_selection import train_test_split
from sklearn.neural_network import MLPClassifier

df = pd.read_csv('https://bit.ly/3GsNzGt', delimiter=",")

# Extract input variables (all rows, all columns but last column)
# Note we should do some linear scaling here
X = (df.values[:, :-1] / 255.0)

# Extract output column (all rows, last column)
Y = df.values[:, -1]

# Separate training and testing data
X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=1/3)

nn = MLPClassifier(solver='sgd',
                   hidden_layer_sizes=(3, ),
                   activation='relu',
                   max_iter=100_000,
                   learning_rate_init=.05)

nn.fit(X_train, Y_train)

# Print weights and biases
print(nn.coefs_ )
print(nn.intercepts_)

print("Training set score: %f" % nn.score(X_train, Y_train))
print("Test set score: %f" % nn.score(X_test, Y_test))
```

运行这段代码，我在测试数据上获得了约 99.3% 的准确率。

# 使用 scikit-learn 的 MNIST 示例

要查看一个使用 MNIST 数据集预测手写数字的 scikit-learn 示例，请参阅[附录 A](app01.xhtml#appendix)。

# 神经网络和深度学习的局限性

尽管神经网络具有许多优势，但在某些类型的任务上仍然存在困难。这种对层、节点和激活函数的灵活性使其能够以非线性方式拟合数据...可能太灵活了。为什么？它可能对数据过拟合。深度学习教育的先驱、谷歌大脑前负责人安德鲁·吴在 2021 年的一次新闻发布会上提到这是一个问题。在被问及为什么机器学习尚未取代放射科医生时，这是他在[*IEEE Spectrum*文章](https://oreil.ly/ljXsz)中的回答：

> 结果表明，当我们从斯坦福医院收集数据，然后在同一家医院的数据上进行训练和测试时，确实可以发表论文，显示[算法]在发现某些病况方面与人类放射科医生相媲美。
> 
> 结果表明，当你将同样的模型，同样的人工智能系统，带到街对面的一家老医院，使用一台老机器，技术人员使用稍有不同的成像协议时，数据漂移会导致人工智能系统的性能显著下降。相比之下，任何一名人类放射科医生都可以走到街对面的老医院并做得很好。
> 
> 因此，即使在某个特定数据集上的某个时间点上，我们可以展示这是有效的，临床现实是这些模型仍然需要大量工作才能投入生产。

换句话说，机器学习过度拟合了斯坦福医院的训练和测试数据集。当应用到其他设备不同的医院时，由于过度拟合，性能显著下降。

自动驾驶汽车和无人驾驶汽车也面临相同的挑战。仅仅在一个停车标志上训练神经网络是不够的！它必须在围绕该停车标志的无数条件下进行训练：晴天、雨天、夜晚和白天、有涂鸦、被树挡住、在不同的地点等等。在交通场景中，想象所有不同类型的车辆、行人、穿着服装的行人以及将遇到的无限数量的边缘情况！简单地说，通过在神经网络中增加更多的权重和偏差来捕捉在道路上遇到的每种事件是没有效果的。

这就是为什么自动驾驶汽车本身不会以端到端的方式使用神经网络。相反，不同的软件和传感器模块被分解，其中一个模块可能使用神经网络在物体周围画一个框。然后另一个模块将使用不同的神经网络对该框中的物体进行分类，比如行人。从那里，传统的基于规则的逻辑将尝试预测行人的路径，硬编码逻辑将从不同的条件中选择如何反应。机器学习仅限于标记制作活动，而不涉及车辆的战术和机动。此外，像雷达这样的基本传感器在车辆前方检测到未知物体时将会停止，这只是技术堆栈中另一个不使用机器学习或深度学习的部分。

尽管媒体头条报道神经网络和深度学习在诸如国际象棋和围棋等游戏中**击败人类**，甚至胜过[战斗飞行模拟中的飞行员](https://oreil.ly/hbdYI)，这可能令人惊讶。在这样的强化学习环境中，需要记住模拟是封闭的世界，在这里可以生成无限量的标记数据，并通过虚拟有限世界进行学习。然而，现实世界并非我们可以生成无限量数据的模拟环境。此外，这不是一本哲学书，所以我们将不讨论我们是否生活在模拟中。抱歉，埃隆！在现实世界中收集数据是昂贵且困难的。除此之外，现实世界充满了无限的不可预测性和罕见事件。所有这些因素驱使机器学习从业者[转向数据录入劳动来标记交通物体的图片](https://oreil.ly/mhjvz)和其他数据。自动驾驶汽车初创公司通常必须将这种数据录入工作与模拟数据配对，因为需要生成训练数据的里程数和边缘情况场景太过庞大，无法简单地通过驾驶数百万英里的车队来收集。

这些都是人工智能研究喜欢使用棋盘游戏和视频游戏的原因，因为可以轻松干净地生成无限标记数据。谷歌的著名工程师弗朗西斯·乔勒特（Francis Chollet）为 TensorFlow 开发了 Keras（还写了一本很棒的书，*Python 深度学习*），在一篇[*Verge*文章](https://oreil.ly/4PDLf)中分享了一些见解：

> 问题在于，一旦你选择了一个度量标准，你就会采取任何可用的捷径来操纵它。例如，如果你将下棋作为智能的度量标准（我们从上世纪70年代开始一直持续到90年代），你最终会得到一个只会下棋的系统，仅此而已。没有理由认为它对其他任何事情都有好处。你最终会得到树搜索和极小化，这并不能教会你任何关于人类智能的东西。如今，将追求在像 Dota 或 StarCraft 这样的视频游戏中的技能作为一种普遍智能的替代品，陷入了完全相同的智力陷阱...
> 
> 如果我着手使用深度学习以超人水平“解决”《魔兽争霸III》，只要我有足够的工程人才和计算能力（这类任务需要数千万美元的资金），你可以确信我会成功。但一旦我做到了，我会学到关于智能或泛化的什么？嗯，什么也没有。充其量，我会开发关于扩展深度学习的工程知识。所以我并不认为这是科学研究，因为它并没有教会我们任何我们不已经知道的东西。它没有回答任何未解之谜。如果问题是，“我们能以超人水平玩 X 吗？”，答案肯定是，“是的，只要你能生成足够密集的训练情境样本，并将它们输入到一个足够表达力的深度学习模型中。” 这一点我们已经知道有一段时间了。

换句话说，我们必须小心，不要混淆算法在游戏中的表现与尚未解决的更广泛能力。机器学习、神经网络和深度学习都是狭义地解决定义明确的问题。它们不能广泛推理或选择自己的任务，也不能思考之前未见过的对象。就像任何编码应用程序一样，它们只会执行它们被编程要执行的任务。

无论使用什么工具，解决问题是最重要的。不应该偏袒神经网络或其他任何可用工具。在这一点上，使用神经网络可能不是你面临的任务的最佳选择。重要的是要始终考虑你正在努力解决的问题，而不是把特定工具作为主要目标。深度学习的使用必须是策略性的和有充分理由的。当然有使用案例，但在你的日常工作中，你可能会更成功地使用简单和更偏向的模型，如线性回归、逻辑回归或传统的基于规则的系统。但如果你发现自己需要对图像中的对象进行分类，并且有预算和人力来构建数据集，那么深度学习将是你最好的选择。

# 结论

神经网络和深度学习提供了一些令人兴奋的应用，我们在这一章中只是触及了表面。从识别图像到处理自然语言，继续应用神经网络及其不同类型的深度学习仍然有用武之地。

从零开始，我们学习了如何构建一个具有一个隐藏层的简单神经网络，以预测在背景颜色下是否应该使用浅色或深色字体。我们还应用了一些高级微积分概念来计算嵌套函数的偏导数，并将其应用于随机梯度下降来训练我们的神经网络。我们还涉及到了像scikit-learn这样的库。虽然在本书中我们没有足够的篇幅来讨论TensorFlow、PyTorch和更高级的应用，但有很多优秀的资源可以扩展你的知识。

[3Blue1Brown有一个关于神经网络和反向传播的精彩播放列表](https://oreil.ly/VjwBr)，值得多次观看。[Josh Starmer的StatQuest播放列表关于神经网络](https://oreil.ly/YWnF2)也很有帮助，特别是在将神经网络可视化为流形操作方面。关于流形理论和神经网络的另一个优秀视频可以在[Art of the Problem这里找到](https://youtu.be/e5xKayCBOeU)。最后，当你准备深入研究时，可以查看Aurélien Géron的*使用Scikit-Learn、Keras和TensorFlow进行实践机器学习*（O’Reilly）和Francois Chollet的*Python深度学习*（Manning）。

如果你读到了本章的结尾，并且觉得你合理地吸收了一切，恭喜！你不仅有效地学习了概率、统计学、微积分和线性代数，还将其应用于线性回归、逻辑回归和神经网络等实际应用。我们将在下一章讨论你如何继续前进，并开始你职业成长的新阶段。

# 练习

将神经网络应用于我们在[第6章](ch06.xhtml#ch06)中处理的员工留存数据。您可以在[这里](https://tinyurl.com/y6r7qjrp)导入数据。尝试构建这个神经网络，使其对这个数据集进行预测，并使用准确率和混淆矩阵来评估性能。这对于这个问题是一个好模型吗？为什么？

虽然欢迎您从头开始构建神经网络，但考虑使用scikit-learn、PyTorch或其他深度学习库以节省时间。

答案在[附录B](app02.xhtml#exercise_answers)中。
