# 第4章。线性代数

稍微转换一下思路，让我们远离概率和统计，进入线性代数领域。有时人们会混淆线性代数和基本代数，可能认为它与使用代数函数*y* = *mx* + *b*绘制线条有关。这就是为什么线性代数可能应该被称为“向量代数”或“矩阵代数”，因为它更加抽象。线性系统发挥作用，但以一种更加形而上的方式。

那么，线性代数到底是什么？嗯，*线性代数*关注线性系统，但通过向量空间和矩阵来表示它们。如果你不知道什么是向量或矩阵，不用担心！我们将深入定义和探索它们。线性代数对于许多应用领域的数学、统计学、运筹学、数据科学和机器学习都至关重要。当你在这些领域中处理数据时，你正在使用线性代数，也许你甚至不知道。

你可以暂时不学习线性代数，使用机器学习和统计库为你完成所有工作。但是，如果你想理解这些黑匣子背后的直觉，并更有效地处理数据，理解线性代数的基础是不可避免的。线性代数是一个庞大的主题，可以填满厚厚的教科书，所以当然我们不能在这本书的一章中完全掌握它。然而，我们可以学到足够多，以便更加熟练地应用它并有效地在数据科学领域中导航。在本书的剩余章节中，包括第[5](ch05.xhtml#ch05)章和第[7](ch07.xhtml#ch07)章，也将有机会应用它。

# 什么是向量？

简单来说，*向量*是空间中具有特定方向和长度的箭头，通常代表一段数据。它是线性代数的中心构建模块，包括矩阵和线性变换。在其基本形式中，它没有位置的概念，所以始终想象它的尾部从笛卡尔平面的原点（0,0）开始。

[图 4-1](#IcpsmEJwfp)展示了一个向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>，它在水平方向移动三步，在垂直方向移动两步。

![emds 0401](Images/emds_0401.png)

###### 图4-1。一个简单的向量

再次强调，向量的目的是直观地表示一段数据。如果你有一条房屋面积为18,000平方英尺，估值为26万美元的数据记录，我们可以将其表示为一个向量[18000, 2600000]，在水平方向移动18000步，在垂直方向移动260000步。

我们数学上声明一个向量如下：

<math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mover accent="true"><mi>v</mi> <mo>→</mo></mover> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mi>x</mi></mtd></mtr> <mtr><mtd><mi>y</mi></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mover accent="true"><mi>v</mi> <mo>→</mo></mover> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>3</mn></mtd></mtr> <mtr><mtd><mn>2</mn></m></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math>

我们可以使用简单的Python集合，如Python列表，在[示例 4-1](#BlpnGwdoVc)中所示声明一个向量。

##### 示例 4-1\. 使用列表在Python中声明向量

```py
v = [3, 2]
print(v)
```

然而，当我们开始对向量进行数学计算，特别是在执行诸如机器学习之类的任务时，我们应该使用NumPy库，因为它比纯Python更高效。您还可以使用SymPy执行线性代数运算，在本章中，当小数变得不方便时，我们偶尔会使用它。然而，在实践中，您可能主要使用NumPy，因此我们主要会坚持使用它。

要声明一个向量，您可以使用NumPy的`array()`函数，然后可以像[示例 4-2](#kAgRAsEUac)中所示传递一组数字给它。

##### 示例 4-2\. 使用NumPy在Python中声明向量

```py
import numpy as np
v = np.array([3, 2])
print(v)
```

# Python速度慢，但其数值库不慢

Python是一个计算速度较慢的语言平台，因为它不像Java、C#、C等编译为较低级别的机器代码和字节码。它在运行时动态解释。然而，Python的数值和科学库并不慢。像NumPy这样的库通常是用低级语言如C和C++编写的，因此它们在计算上是高效的。Python实际上充当“胶水代码”，为您的任务集成这些库。

向量有无数的实际应用。在物理学中，向量通常被认为是一个方向和大小。在数学中，它是XY平面上的一个方向和比例，有点像运动。在计算机科学中，它是存储数据的一组数字。作为数据科学专业人员，我们将最熟悉计算机科学的上下文。然而，重要的是我们永远不要忘记视觉方面，这样我们就不会把向量看作是神秘的数字网格。没有视觉理解，几乎不可能掌握许多基本的线性代数概念，如线性相关性和行列式。

这里有一些更多的向量示例。在[图 4-2](#pHSCTHSSvm)中，请注意一些向量在X和Y轴上具有负方向。具有负方向的向量在我们后面合并时会产生影响，基本上是相减而不是相加。

![emds 0402](Images/emds_0402.png)

###### 图 4-2\. 不同向量的抽样

还要注意，向量可以存在于超过两个维度。接下来我们声明一个沿着 x、y 和 z 轴的三维向量：

<math alttext="ModifyingAbove v With right-arrow equals Start 3 By 1 Matrix 1st Row  x 2nd Row  y 3rd Row  z EndMatrix equals Start 3 By 1 Matrix 1st Row  4 2nd Row  1 3rd Row  2 EndMatrix" display="block"><mrow><mover accent="true"><mi>v</mi> <mo>→</mo></mover> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mi>x</mi></mtd></mtr> <mtr><mtd><mi>y</mi></mtd></mtr> <mtr><mtd><mi>z</mi></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>4</mn></mtd></mtr> <mtr><mtd><mn>1</mn></mtd></mtr> <mtr><mtd><mn>2</mn></mtd></mtr></mtable></mfenced></mrow></math>

要创建这个向量，我们在 x 方向走了四步，在 y 方向走了一步，在 z 方向走了两步。这在[图 4-3](#JQBuLSITaf)中有可视化展示。请注意，我们不再在二维网格上显示向量，而是在一个三维空间中，有三个轴：x、y 和 z。

![emds 0403](Images/emds_0403.png)

###### 图 4-3\. 一个三维向量

当然，我们可以使用三个数值在 Python 中表示这个三维向量，就像在[示例 4-3](#eQMUWBhCIB)中声明的那样。

##### 示例 4-3\. 在 Python 中使用 NumPy 声明一个三维向量

```py
import numpy as np
v = np.array([4, 1, 2])
print(v)
```

像许多数学模型一样，可视化超过三维是具有挑战性的，这是我们在本书中不会花费精力去做的事情。但从数字上来看，仍然很简单。[示例 4-4](#onWRdWVGdW)展示了我们如何在 Python 中数学上声明一个五维向量。

<math alttext="ModifyingAbove v With right-arrow equals Start 5 By 1 Matrix 1st Row  6 2nd Row  1 3rd Row  5 4th Row  8 5th Row  3 EndMatrix" display="block"><mrow><mover accent="true"><mi>v</mi> <mo>→</mo></mover> <mo>=</mo> <mfenced separators="" open="[" close="]"><mtable><mtr><mtd><mn>6</mn></mtd></mtr> <mtr><mtd><mn>1</mn></mtd></mtr> <mtr><mtd><mn>5</mn></mtd></mtr> <mtr><mtd><mn>8</mn></mtd></mtr> <mtr><mtd><mn>3</mn></mtd></mtr></mtable></mfenced></mrow></math>

##### 示例 4-4\. 在 Python 中使用 NumPy 声明一个五维向量

```py
import numpy as np
v = np.array([6, 1, 5, 8, 3])
print(v)
```

## 添加和组合向量

单独看，向量并不是非常有趣。它表示一个方向和大小，有点像在空间中的移动。但当你开始组合向量，也就是*向量相加*时，事情就变得有趣起来。我们实际上将两个向量的运动合并成一个单一的向量。

假设我们有两个向量 <math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math> 和 <math alttext="ModifyingAbove w With right-arrow"><mover accent="true"><mi>w</mi> <mo>→</mo></mover></math> 如[图 4-4](#NDAOFFKCBg)所示。我们如何将这两个向量相加呢？

![emds 0404](Images/emds_0404.png)

###### 图 4-4\. 将两个向量相加

我们将在稍后讨论为什么添加向量是有用的。但如果我们想要结合这两个向量，包括它们的方向和大小，那会是什么样子呢？从数字上来看，这很简单。你只需将各自的 x 值相加，然后将 y 值相加，得到一个新的向量，如[示例 4-5](#bvfoUpUGnc)所示。

<math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mover accent="true"><mi>v</mi> <mo>→</mo></mover> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>3</mn></mtd></mtr> <mtr><mtd><mn>2</mn></m></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mover accent="true"><mi>w</mi> <mo>→</mo></mover> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>2</mn></mtd></mtr> <mtr><mtd><mrow><mo>-</mo> <mn>1</mn></mrow></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mover accent="true"><mi>v</mi> <mo>→</mo></mover> <mo>+</mo> <mover accent="true"><mi>w</mi> <mo>→</mo></mover> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mn>3</mn> <mo>+</mo> <mn>2</mn></mrow></mtd></mtr> <mtr><mtd><mrow><mn>2</mn> <mo>+</mo> <mo>-</mo> <mn>1</mn></mrow></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>5</mn></mtd></mtr> <mtr><mtd><mn>1</mn></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math>

##### 示例 4-5\. 使用 NumPy 在 Python 中将两个向量相加

```py
from numpy import array

v = array([3,2])
w = array([2,-1])

# sum the vectors
v_plus_w = v + w

# display summed vector
print(v_plus_w) # [5, 1]
```

但这在视觉上意味着什么呢？要将这两个向量视觉上相加，将一个向量连接到另一个向量的末端，然后走到最后一个向量的顶端（[图 4-5](#jHAeQufeOr)）。你最终停留的位置就是一个新向量，是这两个向量相加的结果。

![emds 0405](Images/emds_0405.png)

###### 图 4-5\. 将两个向量相加得到一个新向量

如图 [4-5](#jHAeQufeOr) 所示，当我们走到最后一个向量 <math alttext="ModifyingAbove w With right-arrow"><mover accent="true"><mi>w</mi> <mo>→</mo></mover></math> 的末端时，我们得到一个新向量 [5, 1]。这个新向量是将 <math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math> 和 <math alttext="ModifyingAbove w With right-arrow"><mover accent="true"><mi>w</mi> <mo>→</mo></mover></math> 相加的结果。在实践中，这可以简单地将数据相加。如果我们在一个区域中总结房屋价值和其平方英尺，我们将以这种方式将多个向量相加成一个单一向量。

请注意，无论我们是先将 <math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math> 加上 <math alttext="ModifyingAbove w With right-arrow"><mover accent="true"><mi>w</mi> <mo>→</mo></mover></math> 还是反之，都不影响结果，这意味着它是*可交换*的，操作顺序不重要。如果我们先走 <math alttext="ModifyingAbove w With right-arrow"><mover accent="true"><mi>w</mi> <mo>→</mo></mover></math> 再走 <math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>，我们最终得到的向量 [5, 1] 与 [图 4-6](#CvgeFbUEet) 中可视化的相同。

![emds 0406](Images/emds_0406.png)

###### 图4-6\. 向量的加法是可交换的

## 缩放向量

*缩放*是增加或减小向量的长度。你可以通过乘以一个称为*标量*的单个值来增加/减小向量。[图4-7](#TDiKegoNgG)是向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>被放大2倍的示例。

![emds 0407](Images/emds_0407.png)

###### 图4-7\. 向量的缩放

从数学上讲，你将向量的每个元素乘以标量值：

<math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mover accent="true"><mi>v</mi> <mo>→</mo></mover> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>3</mn></mtd></mtr> <mtr><mtd><mn>1</mn></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mn>2</mn> <mover accent="true"><mi>v</mi> <mo>→</mo></mover> <mo>=</mo> <mn>2</mn> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>3</mn></mtd></mtr> <mtr><mtd><mn>1</mn></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mn>3</mn> <mo>×</mo> <mn>2</mn></mrow></mtd></mtr> <mtr><mtd><mrow><mn>1</mn> <mo>×</mo> <mn>2</mn></mrow></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>6</mn></mtd></mtr> <mtr><mtd><mn>2</mn></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math>

在Python中执行这个缩放操作就像将向量乘以标量一样简单，就像在[示例4-6](#bFNICSfEJu)中编写的那样。

##### 示例4-6\. 在Python中使用NumPy缩放数字

```py
from numpy import array

v = array([3,1])

# scale the vector
scaled_v = 2.0 * v

# display scaled vector
print(scaled_v) # [6 2]
```

在这里，[图4-8](#wfUpJAIJuR)中的<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>被缩小了一半。

![emds 0408](Images/emds_0408.png)

###### 图4-8\. 将向量缩小一半

这里需要注意的一个重要细节是，缩放向量不会改变其方向，只会改变其大小。但是有一个轻微的例外，如[图4-9](#FhcgmMWkaq)中所示。当你用一个负数乘以一个向量时，它会翻转向量的方向。

![emds 0409](Images/emds_0409.png)

###### 图4-9\. 负标量会翻转向量方向

仔细思考一下，通过负数进行缩放实际上并没有改变方向，因为它仍然存在于同一条线上。这引出了一个称为线性相关的关键概念。

## 跨度和线性相关性

这两个操作，相加两个向量并对它们进行缩放，带来了一个简单但强大的想法。通过这两个操作，我们可以组合两个向量并对它们进行缩放，以创建我们想要的任何结果向量。[图 4-10](#RkeRvAFrnW)展示了取两个向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>和<math alttext="ModifyingAbove w With right-arrow"><mover accent="true"><mi>w</mi> <mo>→</mo></mover></math>，进行缩放和组合的六个示例。这些向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>和<math alttext="ModifyingAbove w With right-arrow"><mover accent="true"><mi>w</mi> <mo>→</mo></mover></math>，固定在两个不同方向上，可以被缩放和相加以创建*任何*新向量<math alttext="ModifyingAbove v plus w With right-arrow"><mover accent="true"><mrow><mi>v</mi><mo>+</mo><mi>w</mi></mrow> <mo>→</mo></mover></math>。

![emds 0410](Images/emds_0410.png)

###### 图4-10。缩放两个相加的向量允许我们创建任何新向量

再次，<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>和<math alttext="ModifyingAbove w With right-arrow"><mover accent="true"><mi>w</mi> <mo>→</mo></mover></math>在方向上固定，除了使用负标量进行翻转，但我们可以使用缩放自由地创建由<math alttext="ModifyingAbove v plus w With right-arrow"><mover accent="true"><mrow><mi>v</mi><mo>+</mo><mi>w</mi></mrow> <mo>→</mo></mover></math>组成的任何向量。

整个可能向量空间称为*span*，在大多数情况下，我们的span可以通过缩放和求和这两个向量来创建无限数量的向量。当我们有两个不同方向的向量时，它们是*线性独立*的，并且具有这种无限的span。

但在哪种情况下我们受限于可以创建的向量？思考一下并继续阅读。

当两个向量存在于相同方向，或存在于同一条线上时会发生什么？这些向量的组合也被限制在同一条线上，将我们的span限制在那条线上。无论如何缩放，结果的和向量也被困在同一条线上。这使它们成为*线性相关*，如[图 4-11](#WgWduALDRL)所示。

![emds 0411](Images/emds_0411.png)

###### 图4-11。线性相关向量

这里的span被困在与由两个向量组成的同一条线上。因为这两个向量存在于相同的基础线上，我们无法通过缩放灵活地创建任何新向量。

在三维或更高维空间中，当我们有一组线性相关的向量时，我们经常会被困在较低维度的平面上。这里有一个例子，即使我们有三维向量如[图 4-12](#NqKDEfAlvb)所述，我们也被困在二维平面上。

![emds 0412](Images/emds_0412.png)

###### 图4-12。三维空间中的线性相关性；请注意我们的张成空间被限制在一个平面上

后面我们将学习一个简单的工具叫做行列式来检查线性相关性，但我们为什么关心两个向量是线性相关还是线性无关呢？当它们线性相关时，很多问题会变得困难或无法解决。例如，当我们在本章后面学习方程组时，一个线性相关的方程组会导致变量消失，使问题无法解决。但如果你有线性无关性，那么从两个或更多向量中创建任何你需要的向量的灵活性将变得无价，以便解决问题！

# 线性变换

将具有固定方向的两个向量相加，但按比例缩放它们以获得不同的组合向量的概念非常重要。这个组合向量，除了线性相关的情况外，可以指向任何方向并具有我们选择的任何长度。这为线性变换建立了一种直观，我们可以使用一个向量以类似函数的方式来转换另一个向量。

## 基向量

想象我们有两个简单的向量<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>和<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>（“i-hat”和“j-hat”）。这些被称为*基向量*，用于描述对其他向量的变换。它们通常长度为1，并且指向垂直正方向，如[图4-13](#sEddcHheJh)中可视化的。

![emds 0413](Images/emds_0413.png)

###### 图4-13。基向量<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>和<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>

将基向量视为构建或转换任何向量的构建块。我们的基向量用一个2×2矩阵表示，其中第一列是<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>，第二列是<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>：

<math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mover accent="true"><mi>i</mi> <mo>^</mo></mover> <mo>=</mo> <mfenced separators="" open="[" close="]"><mtable><mtr><mtd><mn>1</mn></td></mtr> <mtr><mtd><mn>0</mn></td></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mover accent="true"><mi>j</mi> <mo>^</mo></mover> <mo>=</mo> <mfenced separators="" open="[" close="]"><mtable><mtr><mtd><mn>0</mn></td></mtr> <mtr><mtd><mn>1</mn></td></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mtext>基向量</mtext> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>1</mn></td> <mtd><mn>0</mn></td></mtr> <mtr><mtd><mn>0</mn></td> <mtd><mn>1</mn></td></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math>

*矩阵*是一组向量（如<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>，<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>），可以有多行多列，是打包数据的便捷方式。我们可以使用<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>和<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>通过缩放和相加来创建任何我们想要的向量。让我们从长度为1开始，并在[图4-14](#MvKpOBjCEA)中展示结果向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>。

![emds 0414](Images/emds_0414.png)

###### 图4-14\. 从基向量创建向量

我希望向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>落在[3, 2]处。如果我们将<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>拉伸3倍，<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>拉伸2倍，那么<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>会发生什么？首先，我们分别按照下面的方式对它们进行缩放：

<math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mn>3</mn> <mover accent="true"><mi>i</mi> <mo>^</mo></mover> <mo>=</mo> <mn>3</mn> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>1</mn></td></mtr> <mtr><mtd><mn>0</mn></td></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>3</mn></td></mtr> <mtr><mtd><mn>0</mn></td></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mn>2</mn> <mover accent="true"><mi>j</mi> <mo>^</mo></mover> <mo>=</mo> <mn>2</mn> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>0</mn></td></mtr> <mtr><mtd><mn>1</mn></td></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>0</mn></td></tr> <mtr><mtd><mn>2</mn></td></tr></mtable></mfenced></mrow></mtd></tr></mtable></math>

如果我们在这两个方向上拉伸空间，那么这对 <math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math> 会有什么影响呢？嗯，它将会随着 <math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math> 和 <math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math> 一起拉伸。这被称为*线性变换*，我们通过跟踪基向量的移动来进行向量的拉伸、挤压、剪切或旋转。在这种情况下（[图 4-15](#ioUOiIisaH)），缩放 <math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math> 和 <math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math> 已经沿着我们的向量 <math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math> 拉伸了空间。

![emds 0415](Images/emds_0415.png)

###### 图 4-15\. 一个线性变换

但是 <math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math> 会落在哪里呢？很容易看出它会落在这里，即 [3, 2]。记住向量 <math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math> 是由 <math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math> 和 <math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math> 相加而成的。因此，我们只需将拉伸的 <math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math> 和 <math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math> 相加，就能看出向量 <math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math> 落在哪里：

<math alttext="ModifyingAbove v With right-arrow Subscript n e w Baseline equals StartBinomialOrMatrix 3 Choose 0 EndBinomialOrMatrix plus StartBinomialOrMatrix 0 Choose 2 EndBinomialOrMatrix equals StartBinomialOrMatrix 3 Choose 2 EndBinomialOrMatrix" display="block"><mrow><msub><mover accent="true"><mi>v</mi> <mo>→</mo></mover> <mrow><mi>n</mi><mi>e</mi><mi>w</mi></mrow></msub> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>3</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd></mtr></mtable></mfenced> <mo>+</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>0</mn></mtd></mtr> <mtr><mtd><mn>2</mn></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>3</mn></mtd></mtr> <mtr><mtd><mn>2</mn></mtd></mtr></mtable></mfenced></mrow></math>

一般来说，通过线性变换，你可以实现四种运动，如[图4-16](#FFQRVdcspb)所示。

![emds 0416](Images/emds_0416.png)

###### 图4-16。线性变换可以实现四种运动

这四种线性变换是线性代数的核心部分。缩放向量将拉伸或挤压它。旋转将转动向量空间，而反转将翻转向量空间，使<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>和<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>交换位置。

值得注意的是，你不能有非线性的变换，导致曲线或波浪状的变换不再遵守直线。这就是为什么我们称其为线性代数，而不是非线性代数！

## 矩阵向量乘法

这将引出线性代数中的下一个重要概念。在变换后跟踪<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>和<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>落在哪里的概念很重要，因为这不仅允许我们创建向量，还可以变换现有向量。如果你想要真正的线性代数启示，想想为什么创建向量和变换向量实际上是相同的事情。这完全取决于相对性，考虑到你的基向量在变换前后都是起点。

给定作为矩阵打包的基向量<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>和<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>，变换向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>的公式是：

<math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mfenced open="[" close="]"><mtable><mtr><mtd><msub><mi>x</mi> <mrow><mi>n</mi><mi>e</mi><mi>w</mi></mrow></msub></mtd></mtr> <mtr><mtd><msub><mi>y</mi> <mrow><mi>n</mi><mi>e</mi><mi>w</mi></mrow></msub></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mi>a</mi></mtd> <mtd><mi>b</mi></mtd></mtr> <mtr><mtd><mi>c</mi></mtd> <mtd><mi>d</mi></mtd></mtr></mtable></mfenced> <mfenced open="[" close="]"><mtable><mtr><mtd><mi>x</mi></mtd></mtr> <mtr><mtd><mi>y</mi></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mfenced open="[" close="]"><mtable><mtr><mtd><msub><mi>x</mi> <mrow><mi>n</mi><mi>e</mi><mi>w</mi></mrow></msub></mtd></mtr> <mtr><mtd><msub><mi>y</mi> <mrow><mi>n</mi><mi>e</mi><mi>w</mi></mrow></msub></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mi>a</mi> <mi>x</mi> <mo>+</mo> <mi>b</mi> <mi>y</mi></mrow></mtd></mtr> <mtr><mtd><mrow><mi>c</mi> <mi>x</mi> <mo>+</mo> <mi>d</mi> <mi>y</mi></mrow></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math>

<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math> 是第一列[*a, c*]，<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math> 是第二列[*b, d*]。我们将这两个基向量打包成一个矩阵，再次表示为一个在两个或更多维度中以数字网格形式表达的向量集合。通过应用基向量对向量进行转换被称为*矩阵向量乘法*。这一开始可能看起来有些牵强，但这个公式是对缩放和添加<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>和<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>的快捷方式，就像我们之前对两个向量进行加法，并将该转换应用于任何向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>。

因此，实际上，矩阵确实是表示为基向量的转换。

要在Python中使用NumPy执行这种转换，我们需要将基向量声明为矩阵，然后使用`dot()`运算符将其应用于向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>（参见[示例 4-7](#CHJGoFdTjq)）。`dot()`运算符将执行我们刚刚描述的矩阵和向量之间的缩放和加法。这被称为*点积*，我们将在本章中探讨它。

##### 示例 4-7\. NumPy中的矩阵向量乘法

```py
from numpy import array

# compose basis matrix with i-hat and j-hat
basis = array(
    [[3, 0],
     [0, 2]]
 )

# declare vector v
v = array([1,1])

# create new vector
# by transforming v with dot product
new_v = basis.dot(v)

print(new_v) # [3, 2]
```

当考虑基向量时，我更喜欢将基向量分解然后将它们组合成一个矩阵。只需注意你需要*转置*，或者交换列和行。这是因为NumPy的`array()`函数会执行我们不希望的相反方向，将每个向量填充为一行而不是一列。在NumPy中的转置示例可在[示例 4-8](#RCsLehHkev)中看到。

##### 示例 4-8\. 分离基向量并将它们应用为一个转换

```py
from numpy import array

# Declare i-hat and j-hat
i_hat = array([2, 0])
j_hat = array([0, 3])

# compose basis matrix using i-hat and j-hat
# also need to transpose rows into columns
basis = array([i_hat, j_hat]).transpose()

# declare vector v
v = array([1,1])

# create new vector
# by transforming v with dot product
new_v = basis.dot(v)

print(new_v) # [2, 3]
```

这里有另一个例子。让我们从向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>为[2, 1]开始，<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>和<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>分别从[1, 0]和[0, 1]开始。然后我们将<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>和<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>转换为[2, 0]和[0, 3]。向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>会发生什么？通过手工使用我们的公式进行数学计算，我们得到如下结果：

<math alttext="开始二项式或矩阵 x 下标 n e w 选择 y 下标 n e w 等于开始 2 乘 2 矩阵 第一行第一列 a 第二列 b 第二行第一列 c 第二列 d 结束矩阵 开始二项式或矩阵 x 选择 y 结束二项式或矩阵 等于开始二项式或矩阵 a x 加 b y 选择 c x 加 d y 结束二项式或矩阵" display="block"><mrow><mfenced open="[" close="]"><mtable><mtr><mtd><msub><mi>x</mi> <mrow><mi>n</mi><mi>e</mi><mi>w</mi></mrow></msub></mtd></mtr> <mtr><mtd><msub><mi>y</mi> <mrow><mi>n</mi><mi>e</mi><mi>w</mi></mrow></msub></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mi>a</mi></mtd> <mtd><mi>b</mi></mtd></mtr> <mtr><mtd><mi>c</mi></mtd> <mtd><mi>d</mi></mtd></mtr></mtable></mfenced> <mfenced open="[" close="]"><mtable><mtr><mtd><mi>x</mi></mtd></mtr> <mtr><mtd><mi>y</mi></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mi>a</mi> <mi>x</mi> <mo>+</mo> <mi>b</mi> <mi>y</mi></mrow></mtd></mtr> <mtr><mtd><mrow><mi>c</mi> <mi>x</mi> <mo>+</mo> <mi>d</mi> <mi>y</mi></mrow></mtd></mtr></mtable></mfenced></mrow></math><math alttext="开始二项式或矩阵 x 下标 n e w 选择 y 下标 n e w 等于开始 2 乘 2 矩阵 第一行第一列 2 第二列 0 第二行第一列 0 第二列 3 结束矩阵 开始二项式或矩阵 2 选择 1 结束二项式或矩阵 等于开始二项式或矩阵 左括号 2 右括号 左括号 2 右括号 加 左括号 0 右括号 左括号 1 右括号 选择 左括号 2 右括号 左括号 0 右括号 加 左括号 3 右括号 左括号 1 右括号 结束二项式或矩阵 等于开始二项式或矩阵 4 选择 3 结束二项式或矩阵" display="block"><mrow><mfenced open="[" close="]"><mtable><mtr><mtd><msub><mi>x</mi> <mrow><mi>n</mi><mi>e</mi><mi>w</mi></mrow></msub></mtd></mtr> <mtr><mtd><msub><mi>y</mi> <mrow><mi>n</mi><mi>e</mi><mi>w</mi></mrow></msub></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>2</mn></mtd> <mtd><mn>0</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd> <mtd><mn>3</mn></mtd></mtr></mtable></mfenced> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>2</mn></mtd></mtr> <mtr><mtd><mn>1</mn></m></tr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mo>(</mo> <mn>2</mn> <mo>)</mo> <mo>(</mo> <mn>2</mn> <mo>)</mo> <mo>+</mo> <mo>(</mo> <mn>0</mn> <mo>)</mo> <mo>(</mo> <mn>1</mn> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd><mrow><mo>(</mo> <mn>2</mn> <mo>)</mo> <mo>(</mo> <mn>0</mn> <mo>)</mo> <mo>+</mo> <mo>(</mo> <mn>3</mn> <mo>)</mo> <mo>(</mo> <mn>1</mn> <mo>)</mo></mrow></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>4</mn></mtd></mtr> <mtr><mtd><mn>3</mn></mtd></mtr></mtable></mfenced></mrow></math>

[示例4-9](#glJhkqdFVq)展示了这个解决方案在Python中的应用。

##### 示例4-9。使用NumPy转换向量

```py
from numpy import array

# Declare i-hat and j-hat
i_hat = array([2, 0])
j_hat = array([0, 3])

# compose basis matrix using i-hat and j-hat
# also need to transpose rows into columns
basis = array([i_hat, j_hat]).transpose()

# declare vector v 0
v = array([2,1])

# create new vector
# by transforming v with dot product
new_v = basis.dot(v)

print(new_v) # [4, 3]
```

向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>现在落在[4, 3]处。[图4-17](#wOOmFvfSph)展示了这个变换的样子。

![emds 0417](Images/emds_0417.png)

###### 图4-17。一个拉伸的线性变换

这是一个将事情提升到一个新水平的例子。让我们取值为[2, 1]的向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>。<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>和<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>起始于[1, 0]和[0, 1]，但随后被变换并落在[2, 3]和[2, -1]。<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>会发生什么？让我们在[图4-18](#BFAJsJbkfa)和[示例4-10](#JcsfGNwpLU)中看看。

![emds 0418](Images/emds_0418.png)

###### 图4-18。一个进行旋转、剪切和空间翻转的线性变换

##### 示例4-10。一个更复杂的变换

```py
from numpy import array

# Declare i-hat and j-hat
i_hat = array([2, 3])
j_hat = array([2, -1])

# compose basis matrix using i-hat and j-hat
# also need to transpose rows into columns
basis = array([i_hat, j_hat]).transpose()

# declare vector v 0
v = array([2,1])

# create new vector
# by transforming v with dot product
new_v = basis.dot(v)

print(new_v) # [6, 5]
```

这里发生了很多事情。我们不仅缩放了<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>和<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>，还拉长了向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>。我们实际上还剪切、旋转和翻转了空间。当<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>和<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>在顺时针方向上交换位置时，你会知道空间被翻转了，我们将在本章后面学习如何通过行列式检测这一点。

# 矩阵乘法

我们学会了如何将一个向量和一个矩阵相乘，但是将两个矩阵相乘到底实现了什么？将*矩阵乘法*看作是对向量空间应用多个变换。每个变换就像一个函数，我们首先应用最内层的变换，然后依次向外应用每个后续变换。

这里是我们如何对任意向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>（值为[*x, y*]）应用旋转和剪切：

<math alttext="Start 2 By 2 Matrix 1st Row 1st Column 1 2nd Column 1 2nd Row 1st Column 0 2nd Column 1 EndMatrix Start 2 By 2 Matrix 1st Row 1st Column 0 2nd Column negative 1 2nd Row 1st Column 1 2nd Column 0 EndMatrix StartBinomialOrMatrix x Choose y EndBinomialOrMatrix" display="block"><mrow><mfenced open="[" close="]"><mtable><mtr><mtd><mn>1</mn></mtd> <mtd><mn>1</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd> <mtd><mn>1</mn></mtd></mtr></mtable></mfenced> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>0</mn></mtd> <mtd><mrow><mo>-</mo> <mn>1</mn></mrow></mtd></mtr> <mtr><mtd><mn>1</mn></mtd> <mtd><mn>0</mn></mtd></mtr></mtable></mfenced> <mfenced open="[" close="]"><mtable><mtr><mtd><mi>x</mi></mtd></mtr> <mtr><mtd><mi>y</mi></mtd></mtr></mtable></mfenced></mrow></math>

我们实际上可以通过使用这个公式将这两个变换合并，将一个变换应用到最后一个上。你需要将第一个矩阵的每一行与第二个矩阵的每一列相乘并相加，按照“上下！上下！”的模式进行：

<math alttext="Start 2 By 2 Matrix 1st Row 1st Column a 2nd Column b 2nd Row 1st Column c 2nd Column d EndMatrix Start 2 By 2 Matrix 1st Row 1st Column e 2nd Column f 2nd Row 1st Column g 2nd Column h EndMatrix equals Start 2 By 2 Matrix 1st Row 1st Column a e plus b g 2nd Column a f plus b h 2nd Row 1st Column c e plus d y 2nd Column c f plus d h EndMatrix" display="block"><mrow><mfenced open="[" close="]"><mtable><mtr><mtd><mi>a</mi></mtd> <mtd><mi>b</mi></mtd></mtr> <mtr><mtd><mi>c</mi></mtd> <mtd><mi>d</mi></mtd></mtr></mtable></mfenced> <mfenced open="[" close="]"><mtable><mtr><mtd><mi>e</mi></mtd> <mtd><mi>f</mi></mtd></mtr> <mtr><mtd><mi>g</mi></mtd> <mtd><mi>h</mi></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mi>a</mi> <mi>e</mi> <mo>+</mo> <mi>b</mi> <mi>g</mi></mrow></mtd> <mtd><mrow><mi>a</mi> <mi>f</mi> <mo>+</mo> <mi>b</mi> <mi>h</mi></mrow></mtd></mtr> <mtr><mtd><mrow><mi>c</mi> <mi>e</mi> <mo>+</mo> <mi>d</mi> <mi>y</mi></mrow></mtd> <mtd><mrow><mi>c</mi> <mi>f</mi> <mo>+</mo> <mi>d</mi> <mi>h</mi></mrow></mtd></mtr></mtable></mfenced></mrow></math>

因此，我们实际上可以通过使用这个公式将这两个单独的变换（旋转和剪切）合并为一个单一的变换：

<math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mfenced open="[" close="]"><mtable><mtr><mtd><mn>1</mn></mtd> <mtd><mn>1</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd> <mtd><mn>1</mn></mtd></mtr></mtable></mfenced> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>0</mn></mtd> <mtd><mrow><mo>-</mo> <mn>1</mn></mrow></mtd></mtr> <mtr><mtd><mn>1</mn></mtd> <mtd><mn>0</mn></mtd></mtr></mtable></mfenced> <mfenced open="[" close="]"><mtable><mtr><mtd><mi>x</mi></mtd></mtr> <mtr><mtd><mi>y</mi></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mo>(</mo> <mn>1</mn> <mo>)</mo> <mo>(</mo> <mn>0</mn> <mo>)</mo> <mo>+</mo> <mo>(</mo> <mn>1</mn> <mo>)</mo> <mo>(</mo> <mn>1</mn> <mo>)</mo></mrow></mtd> <mtd><mrow><mo>(</mo> <mo>-</mo> <mn>1</mn> <mo>)</mo> <mo>(</mo> <mn>1</mn> <mo>)</mo> <mo>+</mo> <mo>(</mo> <mn>1</mn> <mo>)</mo> <mo>(</mo> <mn>0</mn> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd><mrow><mo>(</mo> <mn>0</mn> <mo>)</mo> <mo>(</mo> <mn>0</mn> <mo>)</mo> <mo>+</mo> <mo>(</mo> <mn>1</mn> <mo>)</mo> <mo>(</mo> <mn>1</mn> <mo>)</mo></mrow></mtd> <mtd><mrow><mo>(</mo> <mn>0</mn> <mo>)</mo> <mo>(</mo> <mo>-</mo> <mn>1</mn> <mo>)</mo> <mo>+</mo> <mo>(</mo> <mn>1</mn> <mo>)</mo> <mo>(</mo> <mn>0</mn> <mo>)</mo></mrow></mtd></mtr></mtable></mfenced> <mfenced open="[" close="]"><mtable><mtr><mtd><mi>x</mi></mtd></mtr> <mtr><mtd><mi>y</mi></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>1</mn></mtd> <mtd><mrow><mo>-</mo> <mn>1</mn></mrow></mtd></mtr> <mtr><mtd><mn>1</mn></mtd> <mtd><mn>0</mn></mtd></mtr></mtable></mfenced> <mfenced open="[" close="]"><mtable><mtr><mtd><mi>x</mi></mtd></mtr> <mtr><mtd><mi>y</mi></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math>

要在 Python 中使用 NumPy 执行这个操作，你可以简单地使用 `matmul()` 或 `@` 运算符来组合这两个矩阵（[示例 4-11](#kobdVPLRAh)）。然后我们将转身并将这个合并的变换应用于一个向量 [1, 2]。

##### 示例 4-11\. 组合两个变换

```py
from numpy import array

# Transformation 1
i_hat1 = array([0, 1])
j_hat1 = array([-1, 0])
transform1 = array([i_hat1, j_hat1]).transpose()

# Transformation 2
i_hat2 = array([1, 0])
j_hat2 = array([1, 1])
transform2 = array([i_hat2, j_hat2]).transpose()

# Combine Transformations
combined = transform2 @ transform1

# Test
print("COMBINED MATRIX:\n {}".format(combined))

v = array([1, 2])
print(combined.dot(v))  # [-1, 1]
```

# 使用 `dot()` 与 `matmul()` 以及 `@`

一般来说，你应该更倾向于使用 `matmul()` 和它的简写 `@` 来组合矩阵，而不是 NumPy 中的 `dot()` 运算符。前者通常对于高维矩阵和元素广播有更好的策略。

如果你喜欢深入研究这些实现细节，[这个 StackOverflow 问题是一个很好的起点](https://oreil.ly/YX83Q)。

请注意，我们也可以将每个变换分别应用于向量 <math alttext="在 v 上方带箭头"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>，并且仍然会得到相同的结果。如果你用这三行替换最后一行，分别应用每个变换，你仍然会得到那个新向量 [-1, 1]：

```py
rotated = transform1.dot(v)
sheered = transform2.dot(rotated)
print(sheered) # [-1, 1]
```

请注意，应用每个变换的顺序很重要！如果我们在 `变换2` 上应用 `变换1`，我们会得到一个不同的结果 [-2, 3]，如 [示例 4-12](#IOcCtlSitO) 中计算的那样。因此矩阵点积不是可交换的，这意味着你不能改变顺序并期望得到相同的结果！

##### 示例 4-12\. 反向应用变换

```py
from numpy import array

# Transformation 1
i_hat1 = array([0, 1])
j_hat1 = array([-1, 0])
transform1 = array([i_hat1, j_hat1]).transpose()

# Transformation 2
i_hat2 = array([1, 0])
j_hat2 = array([1, 1])
transform2 = array([i_hat2, j_hat2]).transpose()

# Combine Transformations, apply sheer first and then rotation
combined = transform1 @ transform2

# Test
print("COMBINED MATRIX:\n {}".format(combined))

v = array([1, 2])
print(combined.dot(v)) # [-2, 3]
```

将每个变换看作一个函数，并且我们从最内层到最外层应用它们，就像嵌套函数调用一样。

# 行列式

当我们进行线性变换时，有时会“扩展”或“挤压”空间，这种情况发生的程度可能会有所帮助。从向量空间中的采样区域中取出一个样本区域，看看在我们对 <math alttext="在 i 上方修改"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math> 和 <math alttext="在 j 上方修改"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math> 进行缩放后会发生什么？

![emds 0420](Images/emds_0420.png)

###### 图 4-20\. 一个行列式测量线性变换如何缩放一个区域

请注意，面积增加了6.0倍，这个因子被称为*行列式*。行列式描述了在向量空间中采样区域随线性变换的尺度变化，这可以提供有关变换的有用信息。

[示例 4-13](#PAtNLPvfAr) 展示了如何在 Python 中计算这个行列式。

##### 示例 4-13\. 计算行列式

```py
from numpy.linalg import det
from numpy import array

i_hat = array([3, 0])
j_hat = array([0, 2])

basis = array([i_hat, j_hat]).transpose()

determinant = det(basis)

print(determinant) # prints 6.0
```

简单的剪切和旋转不会影响行列式，因为面积不会改变。[图 4-21](#NKgNjQhHDF) 和 [示例 4-14](#jglHQIPrRe) 展示了一个简单的剪切，行列式仍然保持为1.0，表明它没有改变。

![emds 0421](Images/emds_0421.png)

###### 图 4-21\. 简单的剪切不会改变行列式

##### 示例 4-14\. 一个剪切的行列式

```py
from numpy.linalg import det
from numpy import array

i_hat = array([1, 0])
j_hat = array([1, 1])

basis = array([i_hat, j_hat]).transpose()

determinant = det(basis)

print(determinant) # prints 1.0
```

但是缩放会增加或减少行列式，因为这会增加/减少采样区域。当方向翻转时（ <math alttext="在 i 上方修改"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math> ， <math alttext="在 j 上方修改"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math> 顺时针交换位置），那么行列式将为负。[图 4-22](#LgGdleomel) 和 [示例 4-15](#DUDwHbbBQL) 说明了一个行列式展示了一个不仅缩放而且翻转了向量空间方向的变换。

![emds 0422](Images/emds_0422.png)

###### 图 4-22\. 在翻转空间上的行列式是负的

##### 示例 4-15\. 一个负的行列式

```py
from numpy.linalg import det
from numpy import array

i_hat = array([-2, 1])
j_hat = array([1, 2])

basis = array([i_hat, j_hat]).transpose()

determinant = det(basis)

print(determinant) # prints -5.0
```

因为这个行列式是负的，我们很快就看到方向已经翻转。但行列式告诉你的最关键的信息是变换是否线性相关。如果行列式为0，那意味着所有的空间都被压缩到一个较低的维度。

在[图4-23](#VBGcgBScmp)中，我们看到两个线性相关的变换，其中一个二维空间被压缩成一维，一个三维空间被压缩成两维。在这两种情况下，面积和体积分别为0！

![emds 0423](Images/emds_0423.png)

###### 图4-23。二维和三维中的线性依赖

[示例4-16](#gvdhUhSoVO)展示了前述2D示例的代码，将整个二维空间压缩成一个一维数轴。

##### 示例4-16。行列式为零

```py
from numpy.linalg import det
from numpy import array

i_hat = array([-2, 1])
j_hat = array([3, -1.5])

basis = array([i_hat, j_hat]).transpose()

determinant = det(basis)

print(determinant) # prints 0.0
```

因此，测试行列式为0对于确定一个变换是否具有线性依赖性非常有帮助。当你遇到这种情况时，你可能会发现手头上有一个困难或无法解决的问题。

# 特殊类型的矩阵

我们应该涵盖一些值得注意的矩阵情况。

## 方阵

*方阵*是行数和列数相等的矩阵：

<math alttext="Start 3 By 3 Matrix 1st Row 1st Column 4 2nd Column 2 3rd Column 7 2nd Row 1st Column 5 2nd Column 1 3rd Column 9 3rd Row 1st Column 4 2nd Column 0 3rd Column 1 EndMatrix" display="block"><mfenced separators="" open="[" close="]"><mtable><mtr><mtd><mn>4</mn></mtd> <mtd><mn>2</mn></mtd> <mtd><mn>7</mn></mtd></mtr> <mtr><mtd><mn>5</mn></mtd> <mtd><mn>1</mn></mtd> <mtd><mn>9</mn></mtd></mtr> <mtr><mtd><mn>4</mn></mtd> <mtd><mn>0</mn></mtd> <mtd><mn>1</mn></mtd></mtr></mtable></mfenced></math>

它们主要用于表示线性变换，并且是许多操作的要求，如特征分解。

## 身份矩阵

*身份矩阵*是一个对角线为1的方阵，而其他值为0：

<math alttext="Start 3 By 3 Matrix 1st Row 1st Column 1 2nd Column 0 3rd Column 0 2nd Row 1st Column 0 2nd Column 1 3rd Column 0 3rd Row 1st Column 0 2nd Column 0 3rd Column 1 EndMatrix" display="block"><mfenced separators="" open="[" close="]"><mtable><mtr><mtd><mn>1</mn></mtd> <mtd><mn>0</mn></mtd> <mtd><mn>0</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd> <mtd><mn>1</mn></mtd> <mtd><mn>0</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd> <mtd><mn>0</mn></mtd> <mtd><mn>1</mn></mtd></mtr></mtable></mfenced></math>

身份矩阵有什么了不起的地方？当你有一个身份矩阵时，实际上是撤销了一个变换并找到了起始基向量。这在下一节解方程组中将起到重要作用。

## 逆矩阵

*逆矩阵*是一个可以撤销另一个矩阵变换的矩阵。假设我有矩阵*A*：

<math alttext="upper A equals Start 3 By 3 Matrix 1st Row 1st Column 4 2nd Column 2 3rd Column 4 2nd Row 1st Column 5 2nd Column 3 3rd Column 7 3rd Row 1st Column 9 2nd Column 3 3rd Column 6 EndMatrix" display="block"><mrow><mi>A</mi> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>4</mn></mtd> <mtd><mn>2</mn></mtd> <mtd><mn>4</mn></mtd></mtr> <mtr><mtd><mn>5</mn></mtd> <mtd><mn>3</mn></mtd> <mtd><mn>7</mn></mtd></mtr> <mtr><mtd><mn>9</mn></mtd> <mtd><mn>3</mn></mtd> <mtd><mn>6</mn></mtd></mtr></mtable></mfenced></mrow></math>

矩阵*A*的逆被称为<math alttext="upper A Superscript negative 1"><msup><mi>A</mi> <mrow><mo>-</mo><mn>1</mn></mrow></msup></math>。我们将在下一节学习如何使用Sympy或NumPy计算逆，但这就是矩阵*A*的逆：

<math alttext="upper A Superscript negative 1 Baseline equals Start 3 By 3 Matrix 1st Row 1st Column negative one-half 2nd Column 0 3rd Column one-third 2nd Row 1st Column 5.5 2nd Column negative 2 3rd Column four-thirds 3rd Row 1st Column negative 2 2nd Column 1 3rd Column one-third EndMatrix" display="block"><mrow><msup><mi>A</mi> <mrow><mo>-</mo><mn>1</mn></mrow></msup> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mo>-</mo> <mfrac><mn>1</mn> <mn>2</mn></mfrac></mrow></mtd> <mtd><mn>0</mn></mtd> <mtd><mfrac><mn>1</mn> <mn>3</mn></mfrac></mtd></mtr> <mtr><mtd><mrow><mn>5</mn> <mo>.</mo> <mn>5</mn></mrow></mtd> <mtd><mrow><mo>-</mo> <mn>2</mn></mrow></mtd> <mtd><mfrac><mn>4</mn> <mn>3</mn></mfrac></mtd></mtr> <mtr><mtd><mrow><mo>-</mo> <mn>2</mn></mrow></mtd> <mtd><mn>1</mn></mtd> <mtd><mfrac><mn>1</mn> <mn>3</mn></mfrac></mtd></mtr></mtable></mfenced></mrow></math>

当我们在<math alttext="upper A Superscript negative 1"><msup><mi>A</mi> <mrow><mo>-</mo><mn>1</mn></mrow></msup></math>和*A*之间进行矩阵乘法时，我们得到一个身份矩阵。我们将在下一节关于方程组的NumPy和Sympy中看到这一点。

<math alttext="Start 3 By 3 Matrix 1st Row 1st Column negative one-half 2nd Column 0 3rd Column one-third 2nd Row 1st Column 5.5 2nd Column negative 2 3rd Column four-thirds 3rd Row 1st Column negative 2 2nd Column 1 3rd Column one-third EndMatrix Start 3 By 3 Matrix 1st Row 1st Column 4 2nd Column 2 3rd Column 4 2nd Row 1st Column 5 2nd Column 3 3rd Column 7 3rd Row 1st Column 9 2nd Column 3 3rd Column 6 EndMatrix equals Start 3 By 3 Matrix 1st Row 1st Column 1 2nd Column 0 3rd Column 0 2nd Row 1st Column 0 2nd Column 1 3rd Column 0 3rd Row 1st Column 0 2nd Column 0 3rd Column 1 EndMatrix" display="block"><mrow><mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mo>-</mo> <mfrac><mn>1</mn> <mn>2</mn></mfrac></mrow></mtd> <mtd><mn>0</mn></mtd> <mtd><mfrac><mn>1</mn> <mn>3</mn></mfrac></mtd></mtr> <mtr><mtd><mrow><mn>5</mn> <mo>.</mo> <mn>5</mn></mrow></mtd> <mtd><mrow><mo>-</mo> <mn>2</mn></mrow></mtd> <mtd><mfrac><mn>4</mn> <mn>3</mn></mfrac></mtd></mtr> <mtr><mtd><mrow><mo>-</mo> <mn>2</mn></mrow></mtd> <mtd><mn>1</mn></mtd> <mtd><mfrac><mn>1</mn> <mn>3</mn></mfrac></mtd></mtr></mtable></mfenced> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>4</mn></mtd> <mtd><mn>2</mn></mtd> <mtd><mn>4</mn></mtd></mtr> <mtr><mtd><mn>5</mn></mtd> <mtd><mn>3</mn></mtd> <mtd><mn>7</mn></mtd></mtr> <mtr><mtd><mn>9</mn></mtd> <mtd><mn>3</mn></mtd> <mtd><mn>6</mn></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced separators="" open="[" close="]"><mtable><mtr><mtd><mn>1</mn></mtd> <mtd><mn>0</mn></mtd> <mtd><mn>0</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd> <mtd><mn>1</mn></mtd> <mtd><mn>0</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd> <mtd><mn>0</mn></mtd> <mtd><mn>1</mn></mtd></mtr></mtable></mfenced></mrow></math>

## 对角矩阵

与身份矩阵类似的是*对角矩阵*，它在对角线上有非零值，而其余值为0。对角矩阵在某些计算中是可取的，因为它们代表应用于向量空间的简单标量。它出现在一些线性代数操作中。

<math alttext="Start 3 By 3 Matrix 1st Row 1st Column 4 2nd Column 0 3rd Column 0 2nd Row 1st Column 0 2nd Column 2 3rd Column 0 3rd Row 1st Column 0 2nd Column 0 3rd Column 5 EndMatrix" display="block"><mfenced separators="" open="[" close="]"><mtable><mtr><mtd><mn>4</mn></mtd> <mtd><mn>0</mn></mtd> <mtd><mn>0</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd> <mtd><mn>2</mn></mtd> <mtd><mn>0</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd> <mtd><mn>0</mn></mtd> <mtd><mn>5</mn></mtd></mtr></mtable></mfenced></math>

## 三角矩阵

与对角矩阵类似的是*三角矩阵*，它在一个三角形值的对角线前有非零值，而其余值为0。

<math alttext="Start 3 By 3 Matrix 1st Row 1st Column 4 2nd Column 2 3rd Column 9 2nd Row 1st Column 0 2nd Column 1 3rd Column 6 3rd Row 1st Column 0 2nd Column 0 3rd Column 5 EndMatrix" display="block"><mfenced separators="" open="[" close="]"><mtable><mtr><mtd><mn>4</mn></mtd> <mtd><mn>2</mn></mtd> <mtd><mn>9</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd> <mtd><mn>1</mn></mtd> <mtd><mn>6</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd> <mtd><mn>0</mn></mtd> <mtd><mn>5</mn></mtd></mtr></mtable></mfenced></math>

三角矩阵在许多数值分析任务中是理想的，因为它们通常更容易在方程组中解决。它们也出现在某些分解任务中，比如[LU分解](https://oreil.ly/vYK8t)。

## 稀疏矩阵

有时，你会遇到大部分元素为零且只有很少非零元素的矩阵。这些被称为*稀疏矩阵*。从纯数学的角度来看，它们并不是特别有趣。但从计算的角度来看，它们提供了创造效率的机会。如果一个矩阵大部分为0，稀疏矩阵的实现将不会浪费空间存储大量的0，而是只跟踪非零的单元格。

<math alttext="sparse colon Start 4 By 3 Matrix 1st Row 1st Column 0 2nd Column 0 3rd Column 0 2nd Row 1st Column 0 2nd Column 0 3rd Column 2 3rd Row 1st Column 0 2nd Column 0 3rd Column 0 4th Row 1st Column 0 2nd Column 0 3rd Column 0 EndMatrix" display="block"><mrow><mtext>sparse:</mtext> <mfenced separators="" open="[" close="]"><mtable><mtr><mtd><mn>0</mn></mtd> <mtd><mn>0</mn></mtd> <mtd><mn>0</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd> <mtd><mn>0</mn></mtd> <mtd><mn>2</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd> <mtd><mn>0</mn></mtd> <mtd><mn>0</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd> <mtd><mn>0</mn></mtd> <mtd><mn>0</mn></mtd></mtr></mtable></mfenced></mrow></math>

当你有大型稀疏矩阵时，你可能会明确使用稀疏函数来创建你的矩阵。

# 方程组和逆矩阵

线性代数的基本用例之一是解方程组。学习逆矩阵也是一个很好的应用。假设你被提供以下方程，需要解出*x*、*y*和*z*：

<math display="block"><mrow><mn>4</mn> <mi>x</mi> <mo>+</mo> <mn>2</mn> <mi>y</mi> <mo>+</mo> <mn>4</mn> <mi>z</mi> <mo>=</mo> <mn>44</mn></mrow></math> <math display="block"><mrow><mn>5</mn> <mi>x</mi> <mo>+</mo> <mn>3</mn> <mi>y</mi> <mo>+</mo> <mn>7</mn> <mi>z</mi> <mo>=</mo> <mn>56</mn></mrow></math> <math display="block"><mrow><mn>9</mn> <mi>x</mi> <mo>+</mo> <mn>3</mn> <mi>y</mi> <mo>+</mo> <mn>6</mn> <mi>z</mi> <mo>=</mo> <mn>72</mn></mrow></math>

你可以尝试手动尝试不同的代数运算来分离三个变量，但如果你想让计算机解决它，你需要将这个问题用矩阵表示，如下所示。将系数提取到矩阵*A*中，方程右侧的值提取到矩阵*B*中，未知变量提取到矩阵*X*中：

<math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi>A</mi> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>4</mn></mtd> <mtd><mn>2</mn></mtd> <mtd><mn>4</mn></mtd></mtr> <mtr><mtd><mn>5</mn></mtd> <mtd><mn>3</mn></mtd> <mtd><mn>7</mn></mtd></mtr> <mtr><mtd><mn>9</mn></mtd> <mtd><mn>3</mn></mtd> <mtd><mn>6</mn></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi>B</mi> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>44</mn></mtd></mtr> <mtr><mtd><mn>56</mn></mtd></mtr> <mtr><mtd><mn>72</mn></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi>X</mi> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mi>x</mi></mtd></mtr> <mtr><mtd><mi>y</mi></mtd></mtr> <mtr><mtd><mi>z</mi></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math>

线性方程组的函数是*AX* = *B*。我们需要用另一个矩阵*X*转换矩阵*A*，结果得到矩阵*B*：

<math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi>A</mi> <mi>X</mi> <mo>=</mo> <mi>B</mi></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mfenced open="[" close="]"><mtable><mtr><mtd><mn>4</mn></mtd> <mtd><mn>2</mn></mtd> <mtd><mn>4</mn></mtd></mtr> <mtr><mtd><mn>5</mn></mtd> <mtd><mn>3</mn></mtd> <mtd><mn>7</mn></mtd></mtr> <mtr><mtd><mn>9</mn></mtd> <mtd><mn>3</mn></mtd> <mtd><mn>6</mn></mtd></mtr></mtable></mfenced> <mo>·</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mi>x</mi></mtd></mtr> <mtr><mtd><mi>y</mi></mtd></tr> <mtr><mtd><mi>z</mi></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>44</mn></mtd></mtr> <mtr><mtd><mn>56</mn></mtd></tr> <mtr><mtd><mn>72</mn></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math>

我们需要“撤销”*A*，这样我们就可以隔离*X*并获得*x*、*y*和*z*的值。撤销*A*的方法是取*A*的逆，用<math alttext="upper A Superscript negative 1"><msup><mi>A</mi> <mrow><mo>-</mo><mn>1</mn></mrow></msup></math>表示，并通过矩阵乘法应用于*A*。我们可以用代数方式表达这一点：

<math display="block"><mrow><mi>A</mi> <mi>X</mi> <mo>=</mo> <mi>B</mi></mrow></math> <math display="block"><mrow><msup><mi>A</mi> <mrow><mo>-</mo><mn>1</mn></mrow></msup> <mi>A</mi> <mi>X</mi> <mo>=</mo> <msup><mi>A</mi> <mrow><mo>-</mo><mn>1</mn></mrow></msup> <mi>B</mi></mrow></math> <math display="block"><mrow><mi>X</mi> <mo>=</mo> <msup><mi>A</mi> <mrow><mo>-</mo><mn>1</mn></mrow></msup> <mi>B</mi></mrow></math>

要计算矩阵*A*的逆，我们可能会使用计算机，而不是手动使用高斯消元法寻找解决方案，这在本书中我们不会涉及。这是矩阵*A*的逆：

<math alttext="upper A Superscript negative 1 Baseline equals Start 3 By 3 Matrix 1st Row 1st Column negative one-half 2nd Column 0 3rd Column one-third 2nd Row 1st Column 5.5 2nd Column negative 2 3rd Column four-thirds 3rd Row 1st Column negative 2 2nd Column 1 3rd Column one-third EndMatrix" display="block"><mrow><msup><mi>A</mi> <mrow><mo>-</mo><mn>1</mn></mrow></msup> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mo>-</mo> <mfrac><mn>1</mn> <mn>2</mn></mfrac></mrow></mtd> <mtd><mn>0</mn></mtd> <mtd><mfrac><mn>1</mn> <mn>3</mn></mfrac></mtd></mtr> <mtr><mtd><mrow><mn>5</mn> <mo>.</mo> <mn>5</mn></mrow></mtd> <mtd><mrow><mo>-</mo> <mn>2</mn></mrow></mtd> <mtd><mfrac><mn>4</mn> <mn>3</mn></mfrac></mtd></mtr> <mtr><mtd><mrow><mo>-</mo> <mn>2</mn></mrow></mtd> <mtd><mn>1</mn></mtd> <mtd><mfrac><mn>1</mn> <mn>3</mn></mfrac></mtd></mtr></mtable></mfenced></mrow></math>

当我们将<math alttext="upper A Superscript negative 1"><msup><mi>A</mi> <mrow><mo>-</mo><mn>1</mn></mrow></msup></math>与*A*相乘时，将创建一个单位矩阵，一个除了对角线上为1之外全为零的矩阵。单位矩阵是线性代数中相当于乘以1的概念，意味着它基本上没有影响，将有效地隔离*x*、*y*和*z*的值：

<math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><msup><mi>A</mi> <mrow><mo>-</mo><mn>1</mn></mrow></msup> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mo>-</mo> <mfrac><mn>1</mn> <mn>2</mn></mfrac></mrow></mtd> <mtd><mn>0</mn></mtd> <mtd><mfrac><mn>1</mn> <mn>3</mn></mfrac></mtd></mtr> <mtr><mtd><mrow><mn>5</mn> <mo>.</mo> <mn>5</mn></mrow></mtd> <mtd><mrow><mo>-</mo> <mn>2</mn></mrow></mtd> <mtd><mfrac><mn>4</mn> <mn>3</mn></mfrac></mtd></mtr> <mtr><mtd><mrow><mo>-</mo> <mn>2</mn></mrow></mtd> <mtd><mn>1</mn></mtd> <mtd><mfrac><mn>1</mn> <mn>3</mn></mfrac></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi>A</mi> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>4</mn></mtd> <mtd><mn>2</mn></mtd> <mtd><mn>4</mn></mtd></mtr> <mtr><mtd><mn>5</mn></mtd> <mtd><mn>3</mn></mtd> <mtd><mn>7</mn></mtd></mtr> <mtr><mtd><mn>9</mn></mtd> <mtd><mn>3</mn></mtd> <mtd><mn>6</mn></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><msup><mi>A</mi> <mrow><mo>-</mo><mn>1</mn></mrow></msup> <mi>A</mi> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>1</mn></mtd> <mtd><mn>0</mn></mtd> <mtd><mn>0</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd> <mtd><mn>1</mn></mtd> <mtd><mn>0</mn></mtd></mtr> <mtr><mtd><mn>0</mn></mtd> <mtd><mn>0</mn></mtd> <mtd><mn>1</mn></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math>

要在 Python 中看到这个单位矩阵的运作，您将需要使用 SymPy 而不是 NumPy。 NumPy 中的浮点小数不会使单位矩阵显得那么明显，但在[示例 4-17](#chOOTGrRCm)中以符号方式进行，我们将看到一个清晰的符号输出。请注意，在 SymPy 中进行矩阵乘法时，我们使用星号 *** 而不是 *@*。

##### 示例 4-17\. 使用 SymPy 研究逆矩阵和单位矩阵

```py
from sympy import *

# 4x + 2y + 4z = 44
# 5x + 3y + 7z = 56
# 9x + 3y + 6z = 72

A = Matrix([
    [4, 2, 4],
    [5, 3, 7],
    [9, 3, 6]
])

# dot product between A and its inverse
# will produce identity function
inverse = A.inv()
identity = inverse * A

# prints Matrix([[-1/2, 0, 1/3], [11/2, -2, -4/3], [-2, 1, 1/3]])
print("INVERSE: {}".format(inverse))

# prints Matrix([[1, 0, 0], [0, 1, 0], [0, 0, 1]])
print("IDENTITY: {}".format(identity))
```

在实践中，浮点精度的缺失不会对我们的答案产生太大影响，因此使用 NumPy 来解决 *x* 应该是可以的。[示例 4-18](#wqpBKiRBAT) 展示了使用 NumPy 的解决方案。

##### 示例 4-18\. 使用 NumPy 解决一组方程

```py
from numpy import array
from numpy.linalg import inv

# 4x + 2y + 4z = 44
# 5x + 3y + 7z = 56
# 9x + 3y + 6z = 72

A = array([
    [4, 2, 4],
    [5, 3, 7],
    [9, 3, 6]
])

B = array([
    44,
    56,
    72
])

X = inv(A).dot(B)

print(X) # [ 2\. 34\. -8.]
```

因此 *x* = 2，*y* = 34，*z* = –8\. [示例 4-19](#dpclnceGpV) 展示了 SymPy 中的完整解决方案，作为 NumPy 的替代方案。

##### 示例 4-19\. 使用 SymPy 解决一组方程

```py
from sympy import *

# 4x + 2y + 4z = 44
# 5x + 3y + 7z = 56
# 9x + 3y + 6z = 72

A = Matrix([
    [4, 2, 4],
    [5, 3, 7],
    [9, 3, 6]
])

B = Matrix([
    44,
    56,
    72
])

X = A.inv() * B

print(X) # Matrix([[2], [34], [-8]])
```

这里是数学符号表示的解决方案：

<math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><msup><mi>A</mi> <mrow><mo>-</mo><mn>1</mn></mrow></msup> <mi>B</mi> <mo>=</mo> <mi>X</mi></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mfenced open="[" close="]"><mtable><mtr><mtd><mrow><mo>-</mo> <mfrac><mn>1</mn> <mn>2</mn></mfrac></mrow></mtd> <mtd><mn>0</mn></td> <mtd><mfrac><mn>1</mn> <mn>3</mn></mfrac></mtd></mtr> <mtr><mtd><mrow><mn>5</mn> <mo>.</mo> <mn>5</mn></mrow></mtd> <mtd><mrow><mo>-</mo> <mn>2</mn></mrow></mtd> <mtd><mfrac><mn>4</mn> <mn>3</mn></mfrac></mtd></mtr> <mtr><mtd><mrow><mo>-</mo> <mn>2</mn></mrow></mtd> <mtd><mn>1</mn></mtd> <mtd><mfrac><mn>1</mn> <mn>3</mn></mfrac></mtd></mtr></mtable></mfenced> <mfenced open="[" close="]"><mtable><mtr><mtd><mn>44</mn></mtd></mtr> <mtr><mtd><mn>56</mn></mtd></mtr> <mtr><mtd><mn>72</mn></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mi>x</mi></mtd></mtr> <mtr><mtd><mi>y</mi></mtd></mtr> <mtr><mtd><mi>z</mi></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mfenced open="[" close="]"><mtable><mtr><mtd><mn>2</mn></mtd></mtr> <mtr><mtd><mn>34</mn></mtd></mtr> <mtr><mtd><mrow><mo>-</mo> <mn>8</mn></mrow></mtd></mtr></mtable></mfenced> <mo>=</mo> <mfenced open="[" close="]"><mtable><mtr><mtd><mi>x</mi></mtd></mtr> <mtr><mtd><mi>y</mi></mtd></mtr> <mtr><mtd><mi>z</mi></mtd></mtr></mtable></mfenced></mrow></mtd></mtr></mtable></math>

希望这给了你逆矩阵的直觉以及它们如何用于解方程组的用途。

# 线性规划中的方程组

这种解方程组的方法也用于线性规划，其中不等式定义约束条件，目标是最小化/最大化。

[PatrickJMT 在线性规划方面有很多优质视频](https://bit.ly/3aVyrD6)。我们也在[附录 A](app01.xhtml#appendix)中简要介绍了它。

在实际操作中，你很少需要手动计算逆矩阵，可以让计算机为你完成。但如果你有需求或好奇，你会想了解高斯消元法。[PatrickJMT 在 YouTube 上的视频](https://oreil.ly/RfXAv)展示了许多关于高斯消元法的视频。

# 特征向量和特征值

*矩阵分解*是将矩阵分解为其基本组件，类似于分解数字（例如，10 可以分解为 2 × 5）。

矩阵分解对于诸如找到逆矩阵、计算行列式以及线性回归等任务非常有帮助。根据你的任务，有许多分解矩阵的方法。在[第 5 章](ch05.xhtml#ch05)中，我们将使用一种矩阵分解技术，QR 分解，来执行线性回归。

但在本章中，让我们专注于一种常见方法，称为特征分解，这经常用于机器学习和主成分分析。在这个层面上，我们没有精力深入研究每个应用。现在，只需知道特征分解有助于将矩阵分解为在不同机器学习任务中更易处理的组件。还要注意它只适用于方阵。

在特征分解中，有两个组成部分：用 lambda 表示的特征值 <math alttext="lamda"><mi>λ</mi></math> 和用 *v* 表示的特征向量，如 [图 4-24](#mFUgimEtDa) 所示。

![emds 0424](Images/emds_0424.png)

###### 图 4-24\. 特征向量和特征值

如果我们有一个方阵 *A*，它有以下特征值方程：

<math alttext="dollar-sign upper A v equals lamda v dollar-sign"><mrow><mi>A</mi> <mi>v</mi> <mo>=</mo> <mi>λ</mi> <mi>v</mi></mrow></math>

如果 *A* 是原始矩阵，它由特征向量 <math alttext="v"><mi>v</mi></math> 和特征值 <math alttext="lamda"><mi>λ</mi></math> 组成。对于父矩阵的每个维度，都有一个特征向量和特征值，并非所有矩阵都可以分解为特征向量和特征值。有时甚至会出现复数（虚数）。

[示例 4-20](#QVjawvbekH) 是我们如何在 NumPy 中为给定矩阵 <math alttext="upper A"><mi>A</mi></math> 计算特征向量和特征值的方法。

##### 示例 4-20\. 在 NumPy 中执行特征分解

```py
from numpy import array, diag
from numpy.linalg import eig, inv

A = array([
    [1, 2],
    [4, 5]
])

eigenvals, eigenvecs = eig(A)

print("EIGENVALUES")
print(eigenvals)
print("\nEIGENVECTORS")
print(eigenvecs)

"""
EIGENVALUES
[-0.46410162  6.46410162]

EIGENVECTORS
[[-0.80689822 -0.34372377]
 [ 0.59069049 -0.9390708 ]]
"""
```

那么我们如何从特征向量和特征值重建矩阵 *A* 呢？回想一下这个公式：

<math alttext="dollar-sign upper A v equals lamda v dollar-sign"><mrow><mi>A</mi> <mi>v</mi> <mo>=</mo> <mi>λ</mi> <mi>v</mi></mrow></math>

我们需要对重构 *A* 的公式进行一些调整：

<math alttext="dollar-sign upper A equals upper Q normal upper Lamda upper Q Superscript negative 1 dollar-sign"><mrow><mi>A</mi> <mo>=</mo> <mi>Q</mi> <mi>Λ</mi> <msup><mi>Q</mi> <mrow><mo>-</mo><mn>1</mn></mrow></msup></mrow></math>

在这个新公式中，<math alttext="upper Q"><mi>Q</mi></math> 是特征向量，<math alttext="normal upper Lamda"><mi>Λ</mi></math> 是对角形式的特征值，<math alttext="upper Q Superscript negative 1"><msup><mi>Q</mi> <mrow><mo>-</mo><mn>1</mn></mrow></msup></math> 是 <math alttext="upper Q"><mi>Q</mi></math> 的逆矩阵。对角形式意味着向量被填充成一个零矩阵，并且占据对角线，类似于单位矩阵的模式。

[示例 4-21](#dJbWIrPcEW) 在 Python 中将示例完整地展示，从分解矩阵开始，然后重新组合它。

##### 示例 4-21\. 在 NumPy 中分解和重组矩阵

```py
from numpy import array, diag
from numpy.linalg import eig, inv

A = array([
    [1, 2],
    [4, 5]
])

eigenvals, eigenvecs = eig(A)

print("EIGENVALUES")
print(eigenvals)
print("\nEIGENVECTORS")
print(eigenvecs)

print("\nREBUILD MATRIX")
Q = eigenvecs
R = inv(Q)

L = diag(eigenvals)
B = Q @ L @ R

print(B)

"""
EIGENVALUES
[-0.46410162  6.46410162]

EIGENVECTORS
[[-0.80689822 -0.34372377]
 [ 0.59069049 -0.9390708 ]]

REBUILD MATRIX
[[1\. 2.]
 [4\. 5.]]
"""
```

正如你所看到的，我们重建的矩阵就是我们开始的那个矩阵。

# 结论

线性代数可能令人困惑，充满了神秘和值得思考的想法。你可能会发现整个主题就像一个大的兔子洞，你是对的！然而，如果你想要有一个长期成功的数据科学职业，继续对它感到好奇是个好主意。它是统计计算、机器学习和其他应用数据科学领域的基础。最终，它是计算机科学的基础。你当然可以一段时间不了解它，但在某个时候你会遇到理解上的限制。

你可能会想知道这些理念如何实际应用，因为它们可能感觉很理论。不用担心；我们将在本书中看到一些实际应用。但是理论和几何解释对于在处理数据时具有直觉是很重要的，通过直观地理解线性变换，你就能为以后可能遇到的更高级概念做好准备。

如果你想更多地了解线性规划，没有比[3Blue1Brown的YouTube播放列表“线性代数的本质”](https://oreil.ly/FSCNz)更好的地方了。[PatrickJMT的线性代数视频](https://oreil.ly/Hx9GP)也很有帮助。

如果你想更加熟悉NumPy，推荐阅读Wes McKinney的O'Reilly图书《Python数据分析》（第二版）。它并不太关注线性代数，但提供了关于在数据集上使用NumPy、Pandas和Python的实用指导。

# 练习

1.  向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>的值为[1, 2]，然后发生了一个变换。<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>落在[2, 0]，<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>落在[0, 1.5]。<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>会落在哪里？

1.  向量<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>的值为[1, 2]，然后发生了一个变换。<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>落在[-2, 1]，<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>落在[1, -2]。<math alttext="ModifyingAbove v With right-arrow"><mover accent="true"><mi>v</mi> <mo>→</mo></mover></math>会落在哪里？

1.  一个变换<math alttext="ModifyingAbove i With caret"><mover accent="true"><mi>i</mi> <mo>^</mo></mover></math>落在[1, 0]，<math alttext="ModifyingAbove j With caret"><mover accent="true"><mi>j</mi> <mo>^</mo></mover></math>落在[2, 2]。这个变换的行列式是多少？

1.  一个线性变换中是否可以进行两个或更多个线性变换？为什么？

1.  解方程组以求解*x*、*y*和*z*：

    <math display="block"><mrow><mn>3</mn> <mi>x</mi> <mo>+</mo> <mn>1</mn> <mi>y</mi> <mo>+</mo> <mn>0</mn> <mi>z</mi> <mo>=</mo> <mo>=</mo> <mn>54</mn></mrow></math> <math display="block"><mrow><mn>2</mn> <mi>x</mi> <mo>+</mo> <mn>4</mn> <mi>y</mi> <mo>+</mo> <mn>1</mn> <mi>z</mi> <mo>=</mo> <mn>12</mn></mrow></math> <math display="block"><mrow><mn>3</mn> <mi>x</mi> <mo>+</mo> <mn>1</mn> <mi>y</mi> <mo>+</mo> <mn>8</mn> <mi>z</mi> <mo>=</mo> <mn>6</mn></mrow></math>

1.  以下矩阵是否线性相关？为什么？

    <math alttext="Start 2 By 2 Matrix 1st Row 1st Column 2 2nd Column 1 2nd Row 1st Column 6 2nd Column 3 EndMatrix" display="block"><mfenced open="[" close="]"><mtable><mtr><mtd><mn>2</mn></mtd> <mtd><mn>1</mn></mtd></mtr> <mtr><mtd><mn>6</mn></mtd> <mtd><mn>3</mn></mtd></mtr></mtable></mfenced></math>

答案在[附录 B](app02.xhtml#exercise_answers)中。
