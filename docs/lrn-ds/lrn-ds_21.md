# 第16章 模型选择

到目前为止，当我们拟合模型时，我们已经采用了几种策略来决定包含哪些特征：

+   使用残差图评估模型拟合度。

+   将统计模型与物理模型连接起来。

+   保持模型简单。

+   比较在日益复杂的模型之间残差标准差和MSE的改进。

例如，在我们检查单变量模型中的上升流动性时，我们发现残差图中有曲率。增加第二个变量大大改善了平均损失（MSE以及相关的多个<math><msup><mi>R</mi> <mn>2</mn></msup></math>），但残差中仍然存在一些曲率。一个七变量模型在降低MSE方面几乎没有比两变量模型更多的改进，因此尽管两变量模型仍然显示出残差中的一些模式，我们选择了这个更简单的模型。

另一个例子是，在我们建立一只驴的体重模型时，在[第18章](ch18.html#ch-donkey)中，我们将从物理模型中获取指导。我们将忽略驴的附肢，并借鉴桶和驴身体的相似性开始拟合一个模型，通过其长度和围度（类似于桶的高度和周长）来解释体重。然后，我们将继续通过添加与驴的物理状况和年龄相关的分类特征来调整该模型，合并类别，并排除其他可能的特征，以保持模型简单。

我们在构建这些模型时所做的决策基于判断，而在本章中，我们将这些决策与更正式的标准结合起来。首先，我们提供一个示例，说明为什么在模型中包含太多特征通常不是一个好主意。这种现象称为*过度拟合*，经常导致模型过于贴近数据并捕捉到数据中的一些噪音。然后，当新的观察到来时，预测结果比较简单模型更糟糕。本章的其余部分提供了一些技术，如训练集-测试集分割、交叉验证和正则化，来限制过度拟合的影响。当模型可能包含大量潜在特征时，这些技术尤为有用。我们还提供一个合成示例，我们在其中了解到真实模型，以解释模型方差和偏差的概念及其与过拟合和欠拟合的关系。

# 过度拟合

当我们有许多可用于模型的特征时，选择包括或排除哪些特征会迅速变得复杂。在上升移动性的例子中，在[第15章](ch15.html#ch-linear)中，我们选择了七个变量中的两个来拟合模型，但是对于一个双变量模型，我们可以检查并拟合21对特征。如果考虑所有一、二、...、七个变量模型，还有超过一百种模型可供选择。检查数百个残差图以决定何时简单到足够，以及确定一个模型是相当困难的。不幸的是，最小化均方误差的概念并非完全有用。当我们向模型添加一个变量时，均方误差通常会变小。回顾模型拟合的几何视角（[第15章](ch15.html#ch-linear)），添加一个特征到模型中会添加一个 <math><mi>n</mi></math> -维向量到特征空间中，并且结果向量与其在由解释变量张成的空间内的投影之间的误差会减小。我们可能认为这是一件好事，因为我们的模型更接近数据，但过度拟合存在危险。

当模型过于紧密地跟随数据并捕捉到结果中的随机噪声变化时，就会发生过度拟合。当这种情况发生时，新的观察结果就无法很好地预测。举个例子可以帮助澄清这个概念。

## 示例：能源消耗

在这个例子中，我们研究一个可以下载的[数据集](https://oreil.ly/ngD4G)，其中包含明尼苏达州一个私人住宅的公用事业账单信息。我们记录了一个家庭每月的气体使用量（立方英尺）和该月的平均温度（华氏度）。^([1](ch16.html#id1643)) 我们首先读取数据：

```py
`heat_df` `=` `pd``.``read_csv``(``"``data/utilities.csv``"``,` `usecols``=``[``"``temp``"``,` `"``ccf``"``]``)`
`heat_df`

```

|   | 温度 | 立方英尺 |
| --- | --- | --- |
| **0** | 29 | 166 |
| **1** | 31 | 179 |
| **2** | 15 | 224 |
| **...** | ... | ... |
| **96** | 76 | 11 |
| **97** | 55 | 32 |
| **98** | 39 | 91 |

```py
99 rows × 2 columns
```

我们将从查看气体消耗作为温度函数的散点图开始：

![](assets/leds_16in01.png)

这个关系显示了曲率（左图），但当我们尝试通过对数变换将其变成直线（右图）时，在低温区域出现了不同的曲率。此外，还有两个异常点。当我们查阅文档时，发现这些点代表记录错误，因此我们将它们移除。

看看二次曲线能否捕捉气体使用量和温度之间的关系。多项式仍然被认为是线性模型。它们在其多项式特征中是线性的。例如，我们可以将二次模型表示为：

<math display="block"><msub><mi>θ</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>θ</mi> <mn>1</mn></msub> <mi>x</mi> <mo>+</mo> <msub><mi>θ</mi> <mn>2</mn></msub> <msup><mi>x</mi> <mn>2</mn></msup></math>

这个模型对特征 <math><mi>x</mi></math> 和 <math><msup><mi>x</mi> <mn>2</mn></msup></math> 是线性的，在矩阵表示中，我们可以将这个模型写成 <math><mrow><mtext mathvariant="bold">X</mtext></mrow> <mrow><mi mathvariant="bold-italic">θ</mi></mrow></math> ，其中 <math><mtext mathvariant="bold">X</mtext></math> 是设计矩阵：

<math display="block"><mtable columnalign="right" columnspacing="0em" displaystyle="true" rowspacing="3pt"><mtr><mtd><mrow><mo>⌈</mo> <mtable columnalign="center" columnspacing="1em" rowspacing="4pt"><mtr><mtd><mn>1</mn></mtd> <mtd><msub><mi>x</mi> <mn>1</mn></msub></mtd> <mtd><msubsup><mi>x</mi> <mn>1</mn> <mn>2</mn></msubsup></mtd></mtr> <mtr><mtd><mn>1</mn></mtd> <mtd><msub><mi>x</mi> <mn>2</mn></msub></mtd> <mtd><msubsup><mi>x</mi> <mn>2</mn> <mn>2</mn></msubsup></mtd></mtr> <mtr><mtd><mrow><mo>⋮</mo></mrow></mtd> <mtd><mrow><mo>⋮</mo></mrow></mtd> <mtd><mrow><mo>⋮</mo></mrow></mtd></mtr> <mtr><mtd><mn>1</mn></mtd> <mtd><msub><mi>x</mi> <mi>n</mi></msub></mtd> <mtd><msubsup><mi>x</mi> <mi>n</mi> <mn>2</mn></msubsup></mtd></mtr></mtable> <mo>⌉</mo></mrow></mtd></mtr></mtable></math>

我们可以使用`scikit-learn`中的`Polynomial​Fea⁠tures`工具创建设计矩阵的多项式特征：

```py
`y` `=` `heat_df``[``'``ccf``'``]`
`X` `=` `heat_df``[``[``'``temp``'``]``]`

`from` `sklearn``.``preprocessing` `import` `PolynomialFeatures`

`poly` `=` `PolynomialFeatures``(``degree``=``2``,` `include_bias``=``False``)`
`poly_features` `=` `poly``.``fit_transform``(``X``)`
`poly_features`

```

```py
array([[  29.,  841.],
       [  31.,  961.],
       [  15.,  225.],
       ...,
       [  76., 5776.],
       [  55., 3025.],
       [  39., 1521.]])

```

我们将参数`include_bias`设置为`False`，因为我们计划在`scikit-learn`中使用`LinearRegression`方法拟合多项式，默认情况下会在模型中包括常数项。我们用以下方法拟合多项式：

```py
`from` `sklearn``.``linear_model` `import` `LinearRegression`

`model_deg2` `=` `LinearRegression``(``)``.``fit``(``poly_features``,` `y``)`

```

为了快速了解拟合的质量，让我们在散点图上叠加拟合的二次曲线，并查看残差：

![](assets/leds_16in02.png)

二次多项式很好地捕捉了数据中的曲线，但残差显示出在70°F到80°F温度范围内略微上升的趋势，这表明有些拟合不足。此外，残差中也有些漏斗形状，在较冷的月份中，燃气消耗的变化性更大。我们可能会预期这种行为，因为我们只有月均温度。

为了比较，我们使用更高阶的多项式拟合了几个模型，并集体检查拟合曲线：

```py
`poly12` `=` `PolynomialFeatures``(``degree``=``12``,` `include_bias``=``False``)`
`poly_features12` `=` `poly12``.``fit_transform``(``X``)`

`degrees` `=` `[``1``,` `2``,` `3``,` `6``,` `8``,` `12``]`

`mods` `=` `[``LinearRegression``(``)``.``fit``(``poly_features12``[``:``,` `:``deg``]``,` `y``)`
        `for` `deg` `in` `degrees``]`

```

###### 警告

我们在本节中使用多项式特征来演示过拟合，但直接拟合<math><mi>x</mi> <mo>,</mo> <msup><mi>x</mi> <mn>2</mn></msup> <mo>,</mo> <msup><mi>x</mi> <mn>3</mn></msup> <mo>,</mo> <mo>…</mo></math>等多项式在实践中是不可取的。不幸的是，这些多项式特征往往高度相关。例如，能源数据中<math><mi>x</mi></math>和<math><msup><mi>x</mi> <mn>2</mn></msup></math>之间的相关性为0.98。高度相关的特征会导致不稳定的系数，即x值的微小变化可能会导致多项式系数的大幅变化。此外，当x值较大时，正规方程的条件很差，系数的解释和比较可能会很困难。

更好的做法是使用构造成彼此正交的多项式。这些多项式填充与原始多项式相同的空间，但它们彼此不相关，并提供更稳定的拟合。

让我们将所有的多项式拟合放在同一张图上，这样我们可以看到高阶多项式的弯曲越来越奇怪：

![](assets/leds_16in03.png)

我们还可以在单独的面板中可视化不同的多项式拟合：

![](assets/leds_16in04.png)

左上方面的一次曲线（直线）未能捕捉数据中的曲线模式。二次曲线开始捕捉，而三次曲线看起来有所改进，但请注意图表右侧的上升弯曲。六次、八次和十二次的多项式越来越贴近数据，因为它们变得越来越曲折。这些多项式似乎适应数据中的虚假波动。总体来看，这六条曲线说明了欠拟合和过拟合。左上角的拟合线欠拟合，完全错过了曲线。而右下角的十二次多项式明显过拟合，呈现了我们认为在这种情况下无意义的蜿蜒模式。

一般来说，随着特征的增加，模型变得更加复杂，均方误差（MSE）下降，但与此同时，拟合的模型变得越来越不稳定和对数据敏感。当我们过度拟合时，模型过于紧密地跟随数据，对新观测的预测效果很差。评估拟合模型的一种简单技术是在新数据上计算MSE，这些数据未用于建模。由于通常情况下我们无法获取更多数据，因此我们会保留一些原始数据来评估拟合的模型。这个技术是下一节的主题。

# 训练-测试分离

虽然我们希望在建立模型时使用所有数据，但我们也想了解模型在新数据上的表现。通常情况下，我们无法收集额外的数据来评估模型，因此我们会将部分数据保留下来，称为*测试集*，以代表新数据。其余的数据称为*训练集*，我们使用这部分数据来建立模型。然后，在选择了模型之后，我们提取测试集，并查看模型（在训练集上拟合）在测试集中预测结果的表现。[图 16-1](#train-test-diagram) 演示了这个概念。

![](assets/leds_1601.png)

###### 图 16-1\. 训练-测试分离将数据分为两部分：训练集用于建立模型，测试集评估该模型

通常，测试集包含数据的10%到25%。从图表中可能不清楚的是，这种分割通常是随机进行的，因此训练集和测试集彼此相似。

我们可以使用[第 15 章](ch15.html#ch-linear) 中介绍的概念来描述这个过程。设计矩阵，<math><mtext mathvariant="bold">X</mtext></math> ，和结果，<math><mrow><mi mathvariant="bold">y</mi></mrow></math> ，各自被分成两部分；标记为<math><msub><mtext mathvariant="bold">X</mtext> <mi>T</mi></msub></math> 的设计矩阵和相应的结果，标记为<math><msub><mrow><mi mathvariant="bold">y</mi></mrow> <mi>T</mi></msub></math> ，形成训练集。我们通过这些数据最小化<math><mi mathvariant="bold-italic">θ</mi></math> 的平均平方损失：

<math display="block"><munder><mo movablelimits="true">min</mo> <mrow><mi mathvariant="bold-italic">θ</mi></mrow></munder> <mo fence="false" stretchy="false">‖</mo> <msub><mrow><mi mathvariant="bold">y</mi></mrow> <mi>T</mi></msub> <mo>−</mo> <mrow><msub><mtext mathvariant="bold">X</mtext> <mi>T</mi></msub></mrow> <mrow><mrow><mi mathvariant="bold-italic">θ</mi></mrow></mrow> <msup><mo fence="false" stretchy="false">‖</mo> <mn>2</mn></msup></math>

最小化训练误差的系数<math><msub><mrow><mover><mi mathvariant="bold-italic">θ</mi> <mo mathvariant="bold" stretchy="false">^</mo></mover></mrow> <mi>T</mi></msub></math> 用于预测测试集的结果，其中标记为<math><msub><mtext mathvariant="bold">X</mtext> <mi>S</mi></msub></math> 和 <math><msub><mrow><mi mathvariant="bold">y</mi></mrow> <mi>S</mi></msub></math>：

<math display="block"><mo fence="false" stretchy="false">‖</mo> <msub><mrow><mi mathvariant="bold">y</mi></mrow> <mi>S</mi></msub> <mo>−</mo> <mrow><msub><mtext mathvariant="bold">X</mtext> <mi>S</mi></msub></mrow> <mrow><msub><mrow><mover><mi mathvariant="bold-italic">θ</mi> <mo mathvariant="bold" stretchy="false">^</mo></mover></mrow> <mi>T</mi></msub></mrow> <msup><mo fence="false" stretchy="false">‖</mo> <mn>2</mn></msup></math>

由于<math><msub><mtext mathvariant="bold">X</mtext> <mi>S</mi></msub></math>和<math><msub><mrow><mi mathvariant="bold">y</mi></mrow> <mi>S</mi></msub></math>没有用于构建模型，它们可以合理估计我们可能对新观测到的损失。

我们使用上一节中的气耗多项式模型来演示训练-测试分离。为此，我们执行以下步骤：

1.  将数据随机分为两部分，训练集和测试集。

1.  对训练集拟合几个多项式模型并选择一个。

1.  计算在所选多项式（其系数由训练集拟合）上的测试集的MSE。

对于第一步，我们使用`scikit-learn`中的`train_test_split`方法将数据随机分为两部分，并为模型评估设置了22个观测值：

```py
`from` `sklearn``.``model_selection` `import` `train_test_split`

`test_size` `=` `22`

`X_train``,` `X_test``,` `y_train``,` `y_test` `=` `train_test_split``(`
    `X``,` `y``,` `test_size``=``test_size``,` `random_state``=``42``)`

`print``(``f``'``Training set size:` `{``len``(``X_train``)``}``'``)`
`print``(``f``'``Test set size:` `{``len``(``X_test``)``}``'``)`

```

```py
Training set size: 75
Test set size: 22

```

与前一节类似，我们将气温与燃气消耗的模型拟合到各种多项式中。但这次，我们只使用训练数据：

```py
`poly` `=` `PolynomialFeatures``(``degree``=``12``,` `include_bias``=``False``)`
`poly_train` `=` `poly``.``fit_transform``(``X_train``)`

`degree` `=` `np``.``arange``(``1``,``13``)`

`mods` `=` `[``LinearRegression``(``)``.``fit``(``poly_train``[``:``,` `:``j``]``,` `y_train``)`
        `for` `j` `in` `degree``]`

```

我们找出了每个模型的MSE：

```py
`from` `sklearn``.``metrics` `import` `mean_squared_error`

`error_train` `=` `[`
    `mean_squared_error``(``y_train``,` `mods``[``j``]``.``predict``(``poly_train``[``:``,` `:` `(``j` `+` `1``)``]``)``)`
    `for` `j` `in` `range``(``12``)`
`]`

```

为了可视化MSE的变化，我们将每个拟合的多项式的MSE绘制成其次数的图：

```py
`px``.``line``(``x``=``degree``,` `y``=``error_train``,` `markers``=``True``,`
        `labels``=``dict``(``x``=``'``Degree of polynomial``'``,` `y``=``'``Train set MSE``'``)``,`
        `width``=``350``,` `height``=``250``)`

```

![](assets/leds_16in05.png)

注意到随着模型复杂度的增加，训练误差逐渐减小。我们之前看到高阶多项式显示出了我们认为不反映数据中潜在结构的起伏行为。考虑到这一点，我们可能会选择一个更简单但MSE显著减小的模型。这可能是3、4或5次方。让我们选择3次方，因为这三个模型在MSE方面的差异非常小，而且它是最简单的。

现在我们已经选择了我们的模型，我们使用测试集提供了对其MSE的独立评估。我们为测试集准备设计矩阵，并使用在训练集上拟合的3次多项式来预测测试集中每一行的结果。最后，我们计算了测试集的MSE：

```py
`poly_test` `=` `poly``.``fit_transform``(``X_test``)`
`y_hat` `=` `mods``[``2``]``.``predict``(``poly_test``[``:``,` `:``3``]``)`

`mean_squared_error``(``y_test``,` `y_hat``)`

```

```py
307.44460133992294

```

该模型的均方误差（MSE）比在训练数据上计算的MSE要大得多。这说明了在使用相同数据来拟合和评估模型时所存在的问题：MSE并不充分反映出对新观测的MSE。为了进一步说明过拟合问题，我们计算了这些模型的测试误差：

```py
`error_test` `=` `[`
    `mean_squared_error``(``y_test``,` `mods``[``j``]``.``predict``(``poly_test``[``:``,` `:` `(``j` `+` `1``)``]``)``)`
    `for` `j` `in` `range``(``12``)`
`]`

```

在实践中，我们不会在承诺模型之前查看测试集。在训练集上拟合模型并在测试集上评估它之间交替可以导致过拟合。但出于演示目的，我们绘制了我们拟合的所有多项式模型在测试集上的MSE：

![](assets/leds_16in06.png)

注意，对于所有模型而言，测试集的均方误差（MSE）大于训练集的均方误差，而不仅仅是我们选择的模型。更重要的是，注意当模型从欠拟合到更好地拟合数据曲线时，测试集的均方误差最初是下降的。然后，随着模型复杂度的增加，测试集的均方误差增加。这些更复杂的模型对训练数据过拟合，导致预测测试集时出现较大的误差。这种现象的一个理想化示意图如[图 16-2](#train-test-overfit)所示。

![](assets/leds_1602.png)

###### 图 16-2\. 随着模型复杂度的增加，训练集的误差减少，而测试集的误差增加

测试数据提供新观察的预测误差评估。仅在我们已经选择了模型之后才使用测试集是至关重要的。否则，我们会陷入使用相同数据选择和评估模型的陷阱中。在选择模型时，我们回归到了简单性的论点，因为我们意识到越来越复杂的模型往往会过拟合。然而，我们也可以扩展训练-测试方法来帮助选择模型。这是下一节的主题。

# 交叉验证

我们可以使用训练-测试范式来帮助选择模型。其思想是进一步将训练集分成单独的部分，在其中一个部分上拟合模型，然后在另一个部分上评估模型。这种方法称为*交叉验证*。我们描述的是一种版本，称为<math><mi>k</mi></math> *-折叠交叉验证*。[图 16-3](#cvdiagram)展示了这种数据划分背后的思想。

![](assets/leds_1603.png)

###### 图 16-3\. 一个例子展示了五折交叉验证，其中训练集被分为五部分，轮流用于验证在其余数据上构建的模型

交叉验证可以帮助选择模型的一般形式。这包括多项式的阶数，模型中的特征数量，或者正则化惩罚的截止（在下一节中介绍）。<math><mi>k</mi></math> -折叠交叉验证的基本步骤如下：

1.  将训练集分成<math><mi>k</mi></math>个大致相同大小的部分；每部分称为一个*折叠*。使用与创建训练集和测试集相同的技术来创建这些折叠。通常情况下，我们随机划分数据。

1.  将一个折叠保留作为测试集：

    +   在剩余的训练数据上拟合所有模型（训练数据减去特定折叠的数据）。

    +   使用您保留的折叠来评估所有这些模型。

1.  重复此过程共<math><mi>k</mi></math>次，每次将一个折叠保留出来，使用剩余的训练集来拟合模型，并在保留的折叠上评估模型。

1.  合并每个模型在折叠中的拟合误差，并选择具有最小误差的模型。

这些拟合模型在不同的折叠中不会具有相同的系数。例如，当我们拟合一个三次多项式时，我们对<math><mi>k</mi></math>个折叠中的MSE取平均值，得到三次拟合多项式的平均MSE。然后我们比较这些MSE，并选择具有最低MSE的多项式次数。在三次多项式中，<math><mi>x</mi></math>，<math><msup><mi>x</mi><mn>2</mn></msup></math>和<math><msup><mi>x</mi><mn>3</mn></msup></math>项的实际系数在每个<math><mi>k</mi></math>个拟合中是不同的。一旦选择了多项式次数，我们使用所有训练数据重新拟合模型，并在测试集上评估它。（在选择模型的任何早期步骤中，我们没有使用测试集。）

通常，我们使用5或10个折叠。另一种流行的选择是将一个观察结果放入每个折叠中。这种特殊情况称为*留一法交叉验证*。其流行之处在于调整最小二乘拟合以减少一个观察结果的简单性。

通常，<math><mi>k</mi></math>折交叉验证需要一些计算时间，因为我们通常必须为每个折叠从头开始重新拟合每个模型。`scikit-learn`库提供了一个方便的[`sklearn.model_selection.KFold`](https://oreil.ly/tnHTv)类来实现<math><mi>k</mi></math>折交叉验证。

为了让你了解k折交叉验证的工作原理，我们将在燃气消耗示例上演示这种技术。但是，这次我们将拟合不同类型的模型。在数据的原始散点图中，看起来点落在两条连接的线段上。在寒冷的温度下，燃气消耗与温度之间的关系看起来大致是负斜率，约为<math><mo>−</mo><mn>4</mn></math> 立方英尺/度，而在温暖的月份，关系则似乎几乎是平坦的。因此，我们可以拟合一条弯曲的线条而不是拟合多项式。

让我们从65度处拟合一条有弯曲的线。为此，我们创建一个特征，使得温度高于65°F的点具有不同的斜率。该模型是：

<math display="block"><mi>y</mi> <mo>=</mo> <msub><mi>θ</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>θ</mi> <mn>1</mn></msub> <mi>x</mi> <mo>+</mo> <msub><mi>θ</mi> <mn>2</mn></msub> <mo stretchy="false">(</mo> <mi>x</mi> <mo>−</mo> <mn>65</mn> <msup><mo stretchy="false">)</mo> <mo>+</mo></msup></math>

在这里，<math><mo stretchy="false">(</mo><msup><mo stretchy="false">)</mo><mo>+</mo></msup></math>代表“正部分”，所以当<math><mi>x</mi></math>小于65时，它评估为0，当<math><mi>x</mi></math>大于或等于65时，它就是<math><mi>x</mi><mo>−</mo><mn>65</mn></math>。我们创建这个新特征并将其添加到设计矩阵中：

```py
`y` `=` `heat_df``[``"``ccf``"``]`
`X` `=` `heat_df``[``[``"``temp``"``]``]`
`X``[``"``temp65p``"``]` `=` `(``X``[``"``temp``"``]` `-` `65``)` `*` `(``X``[``"``temp``"``]` `>``=` `65``)`

```

然后，我们使用这两个特征拟合模型：

```py
`bend_index` `=` `LinearRegression``(``)``.``fit``(``X``,` `y``)`

```

让我们将这条拟合的“曲线”叠加在散点图上，看看它如何捕捉数据的形状：

![](assets/leds_16in07.png)

这个模型似乎比多项式更好地拟合了数据。但是可能有许多种弯线模型。线条可能在 55 度或 60 度处弯曲等。我们可以使用 <math><mi>k</mi></math> 折交叉验证来选择线条弯曲的温度值。让我们考虑在 <math><mn>40</mn> <mo>,</mo> <mn>41</mn> <mo>,</mo> <mn>42</mn> <mo>,</mo> <mo>…</mo> <mo>,</mo> <mn>68</mn> <mo>,</mo> <mn>69</mn></math> 度处弯曲的模型。对于这些模型的每一个，我们需要创建额外的特征来使线条在那里弯曲：

```py
`bends` `=` `np``.``arange``(``40``,` `70``,` `1``)`

`for` `i` `in` `bends``:`
    `col` `=` `"``temp``"` `+` `i``.``astype``(``"``str``"``)` `+` `"``p``"`
    `heat_df``[``col``]` `=` `(``heat_df``[``"``temp``"``]` `-` `i``)` `*` `(``heat_df``[``"``temp``"``]` `>``=` `i``)`
`heat_df`

```

|   | temp | ccf | temp40p | temp41p | ... | temp66p | temp67p | temp68p | temp69p |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **0** | 29 | 166 | 0 | 0 | ... | 0 | 0 | 0 | 0 |
| **1** | 31 | 179 | 0 | 0 | ... | 0 | 0 | 0 | 0 |
| **2** | 15 | 224 | 0 | 0 | ... | 0 | 0 | 0 | 0 |
| **...** | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| **96** | 76 | 11 | 36 | 35 | ... | 10 | 9 | 8 | 7 |
| **97** | 55 | 32 | 15 | 14 | ... | 0 | 0 | 0 | 0 |
| **98** | 39 | 91 | 0 | 0 | ... | 0 | 0 | 0 | 0 |

```py
97 rows × 32 columns
```

交叉验证的第一步是创建我们的训练集和测试集。与之前一样，我们随机选择 22 个观测值放入测试集。这样剩下 75 个观测值作为训练集：

```py
`y` `=` `heat_df``[``'``ccf``'``]`
`X` `=` `heat_df``.``drop``(``columns``=``[``'``ccf``'``]``)`

`test_size` `=` `22`

`X_train``,` `X_test``,` `y_train``,` `y_test` `=` `train_test_split``(`
    `X``,` `y``,` `test_size``=``test_size``,` `random_state``=``0``)`

```

现在我们可以将训练集分成折叠。我们使用三个折叠，以便每个折叠中有 25 个观测值。对于每个折叠，我们拟合 30 个模型，每个模型对应线条中的一个弯曲点。对于这一步骤，我们使用 `scikit-learn` 中的 `KFold` 方法来划分数据：

```py
`from` `sklearn``.``model_selection` `import` `KFold`

`kf` `=` `KFold``(``n_splits``=``3``,` `shuffle``=``True``,` `random_state``=``42``)`

`validation_errors` `=` `np``.``zeros``(``(``3``,` `30``)``)`

`def` `validate_bend_model``(``X``,` `y``,` `X_valid``,` `y_valid``,` `bend_index``)``:`
    `model` `=` `LinearRegression``(``)``.``fit``(``X``.``iloc``[``:``,` `[``0``,` `bend_index``]``]``,` `y``)`
    `predictions` `=` `model``.``predict``(``X_valid``.``iloc``[``:``,` `[``0``,` `bend_index``]``]``)`
    `return` `mean_squared_error``(``y_valid``,` `predictions``)`

`for` `fold``,` `(``train_idx``,` `valid_idx``)` `in` `enumerate``(``kf``.``split``(``X_train``)``)``:`
    `cv_X_train``,` `cv_X_valid` `=` `(``X_train``.``iloc``[``train_idx``,` `:``]``,`
                              `X_train``.``iloc``[``valid_idx``,` `:``]``)`
    `cv_Y_train``,` `cv_Y_valid` `=` `(``y_train``.``iloc``[``train_idx``]``,`
                              `y_train``.``iloc``[``valid_idx``]``)`

    `error_bend` `=` `[`
        `validate_bend_model``(`
            `cv_X_train``,` `cv_Y_train``,` `cv_X_valid``,` `cv_Y_valid``,` `bend_index`
        `)`
        `for` `bend_index` `in` `range``(``1``,` `31``)`
    `]`

    `validation_errors``[``fold``]``[``:``]` `=` `error_bend`

```

然后我们找到三个折叠中的平均验证误差，并将它们绘制在弯曲位置的图表上：

```py
`totals` `=` `validation_errors``.``mean``(``axis``=``0``)`

```

![](assets/leds_16in08.png)

MSE 在 57 到 60 度之间看起来相当平缓。最小值出现在 58 度，因此我们选择那个模型。为了评估这个模型在测试集上的表现，我们首先在整个训练集上以 58 度拟合弯线模型：

```py
`bent_final` `=` `LinearRegression``(``)``.``fit``(`
    `X_train``.``loc``[``:``,` `[``"``temp``"``,` `"``temp58p``"``]``]``,` `y_train`
`)`

```

然后我们使用拟合的模型来预测测试集的气体消耗：

```py
`y_pred_test` `=` `bent_final``.``predict``(``X_test``.``loc``[``:``,` `[``"``temp``"``,` `"``temp58p``"``]``]``)`

`mean_squared_error``(``y_test``,` `y_pred_test``)`

```

```py
71.40781435952441

```

让我们将弯线拟合叠加到散点图上，并检查残差，以了解拟合质量：

![](assets/leds_16in09.png)

拟合曲线看起来合理，并且残差比多项式拟合的要小得多。

###### 注意

在本节的教学目的上，我们使用 `KFold` 手动将训练数据分为三个折叠，然后使用循环找到模型验证误差。在实践中，我们建议使用 `sklearn.model_selection.GridSearchCV` 与 `sklearn.pipeline.Pipeline` 对象，它可以自动将数据分成训练集和验证集，并找到在折叠中平均验证误差最低的模型。

使用交叉验证来管理模型复杂度有几个关键的限制：通常需要使复杂度离散变化，并且可能没有自然的方式来对模型进行排序。与其改变一系列模型的维度，我们可以拟合一个大模型并对系数的大小施加约束。这个概念被称为正则化，将在下一节讨论。

# 正则化

我们刚刚看到交叉验证如何帮助找到适合的模型维度，从而平衡欠拟合和过拟合。与其选择模型的维度，我们可以构建一个包含所有特征的模型，但限制系数的大小。通过在均方误差上添加一个系数大小的惩罚项来防止过拟合。这个惩罚项称为*正则化项*，表达式为<math><mi>λ</mi> <munderover><mo>∑</mo> <mrow><mi>j</mi> <mo>=</mo> <mn>1</mn></mrow> <mrow><mi>p</mi></mrow></munderover> <msubsup><mi>θ</mi> <mi>j</mi> <mn>2</mn></msubsup></math> 。我们通过最小化均方误差和这个惩罚项的组合来拟合模型：

<math display="block"><mfrac><mn>1</mn> <mi>n</mi></mfrac> <munderover><mo>∑</mo> <mrow><mi>i</mi> <mo>=</mo> <mn>1</mn></mrow> <mrow><mi>n</mi></mrow></munderover> <mo stretchy="false">(</mo> <msub><mi>y</mi> <mi>i</mi></msub> <mo>−</mo> <msub><mrow><mi mathvariant="bold">x</mi></mrow> <mi>i</mi></msub> <mi mathvariant="bold-italic">θ</mi> <msup><mo stretchy="false">)</mo> <mn>2</mn></msup>  <mo>+</mo>  <mi>λ</mi> <munderover><mo>∑</mo> <mrow><mi>j</mi> <mo>=</mo> <mn>1</mn></mrow> <mrow><mi>p</mi></mrow></munderover> <msubsup><mi>θ</mi> <mi>j</mi> <mn>2</mn></msubsup></math>

当*正则化参数*<math><mi>λ</mi></math>很大时，会惩罚大的系数。（通常通过交叉验证来选择。）

对系数的平方进行惩罚称为<math><msub><mi>L</mi> <mn>2</mn></msub></math>正则化，或称为*岭回归*。另一种流行的正则化方法惩罚系数的绝对大小：

<math display="block"><mtable columnalign="right" displaystyle="true" rowspacing="3pt"><mtr><mtd><mfrac><mn>1</mn> <mi>n</mi></mfrac> <munderover><mo>∑</mo> <mrow><mi>i</mi> <mo>=</mo> <mn>1</mn></mrow> <mrow><mi>n</mi></mrow></munderover> <mo stretchy="false">(</mo> <msub><mi>y</mi> <mi>i</mi></msub> <mo>−</mo> <msub><mrow><mi mathvariant="bold">x</mi></mrow> <mi>i</mi></msub> <mi mathvariant="bold-italic">θ</mi> <msup><mo stretchy="false">)</mo> <mn>2</mn></msup>  <mo>+</mo>  <mi>λ</mi> <munderover><mo>∑</mo> <mrow><mi>j</mi> <mo>=</mo> <mn>1</mn></mrow> <mrow><mi>p</mi></mrow></munderover> <mrow><mo stretchy="false">|</mo></mrow> <msub><mi>θ</mi> <mi>j</mi></msub> <mo stretchy="false">|</mo></mtd></mtr></mtable></math>

这个<math><msub><mi>L</mi> <mn>1</mn></msub></math>正则化的线性模型也被称为*套索回归*（lasso代表最小绝对值收缩和选择运算符）。

为了了解正则化的工作原理，让我们考虑极端情况：当<math><mi>λ</mi></math>非常大或接近0时（<math><mi>λ</mi></math>从不为负）。当正则化参数很大时，系数会受到严重的惩罚，因此它们会收缩。另一方面，当<math><mi>λ</mi></math>很小时，系数不受限制。实际上，当<math><mi>λ</mi></math>为0时，我们回到了普通最小二乘法的世界。当我们考虑通过正则化控制系数大小时，会遇到几个问题：

+   我们不希望对截距项进行正则化。这样一来，一个大的惩罚就会拟合一个常数模型。

+   当特征具有非常不同的尺度时，惩罚可能会对它们产生不同的影响，具有较大值的特征会比其他特征受到更多的惩罚。为了避免这种情况，我们在拟合模型之前将所有特征标准化，使它们的均值为0，方差为1。

让我们看一个包含35个特征的例子。

# 模型的偏差和方差

在本节中，我们提供了一种不同的思考过拟合和欠拟合问题的方式。我们进行了模拟研究，从我们设计的模型中生成了合成数据。这样，我们知道真实模型，并可以看到在拟合数据时我们离真实情况有多近。

我们可以构建一个数据的通用模型如下：

<math display="block"><mi>y</mi> <mo>=</mo> <mi>g</mi> <mo stretchy="false">(</mo> <mrow><mi mathvariant="bold">x</mi></mrow> <mo stretchy="false">)</mo> <mo>+</mo> <mrow><mi>ϵ</mi></mrow></math>

这个表达式使得模型的两个组成部分很容易看出来：信号 <math><mi>g</mi> <mo stretchy="false">(</mo> <mi>x</mi> <mo stretchy="false">)</mo></math> 和噪声 <math><mi>ϵ</mi></math> 。在我们的模型中，我们假设噪声没有趋势或模式，方差恒定，并且每个观测值的噪声是独立的。

例如，让我们取 <math><mi>g</mi> <mo stretchy="false">(</mo> <mi>x</mi> <mo stretchy="false">)</mo> <mo>=</mo> <mi>sin</mi> <mo>⁡</mo> <mo stretchy="false">(</mo> <mi>x</mi> <mo stretchy="false">)</mo> <mo>+</mo> <mn>0.3</mn> <mi>x</mi></math> ，噪声来自均值为 0，标准差为 0.2 的正态曲线。我们可以从这个模型生成数据，使用以下函数：

```py
`def` `g``(``x``)``:`
    `return` `np``.``sin``(``x``)` `+` `0.3` `*` `x`

`def` `gen_noise``(``n``)``:`
    `return` `np``.``random``.``normal``(``scale``=``0.2``,` `size``=``n``)`

`def` `draw``(``n``)``:`
    `points` `=` `np``.``random``.``choice``(``np``.``arange``(``0``,` `10``,` `0.05``)``,` `size``=``n``)`
    `return` `points``,` `g``(``points``)` `+` `gen_noise``(``n``)`

```

让我们生成 50 个数据点 <math><mo stretchy="false">(</mo> <msub><mi>x</mi> <mi>i</mi></msub> <mo>,</mo> <msub><mi>y</mi> <mi>i</mi></msub> <mo stretchy="false">)</mo></math> ，<math><mi>i</mi> <mo>=</mo> <mn>1</mn> <mo>,</mo> <mo>…</mo> <mo>,</mo> <mn>50</mn></math> ，从这个模型中：

```py
`np``.``random``.``seed``(``42``)`

`xs``,` `ys` `=` `draw``(``50``)`
`noise` `=` `ys` `-` `g``(``xs``)`

```

我们可以绘制我们的数据，因为我们知道真实信号，我们可以找到错误并将它们绘制出来：

![](assets/leds_16in10.png)

左边的图显示 <math><mi>g</mi></math> 作为虚线曲线。我们还可以看到 <math><mo stretchy="false">(</mo> <mi>x</mi> <mo>,</mo> <mi>y</mi> <mo stretchy="false">)</mo></math> 对形成了这条曲线的散点分布。右边的图显示了 50 个点的误差，<math><mi>y</mi> <mo>−</mo> <mi>g</mi> <mo stretchy="false">(</mo> <mi>x</mi> <mo stretchy="false">)</mo></math> 。请注意，它们没有形成模式。

当我们对数据进行模型拟合时，我们最小化均方误差。让我们用一般性写出这个最小化：

<math display="block"><munder><mo movablelimits="true">min</mo> <mrow><mi>f</mi> <mo>∈</mo> <mrow><mi mathvariant="script">F</mi></mrow></mrow></munder> <mfrac><mn>1</mn> <mi>n</mi></mfrac> <munderover><mo>∑</mo> <mrow><mi>i</mi> <mo>=</mo> <mn>1</mn></mrow> <mrow><mi>n</mi></mrow></munderover> <mo stretchy="false">[</mo> <msub><mi>y</mi> <mi>i</mi></msub> <mo>−</mo> <mi>f</mi> <mo stretchy="false">(</mo> <msub><mrow><mi mathvariant="bold">x</mi></mrow> <mi>i</mi></msub> <mo stretchy="false">)</mo> <msup><mo stretchy="false">]</mo> <mn>2</mn></msup></math>

最小化是对函数集合 <math><mrow><mi mathvariant="script">F</mi></mrow></math> 进行的。我们在本章中已经看到，这个函数集合可能是 12 阶多项式，或者简单的弯曲线。一个重要的点是真实模型 <math><mi>g</mi></math> 不必是集合中的一个函数。

让我们把 <math><mrow><mi mathvariant="script">F</mi></mrow></math> 定为二次多项式的集合；换句话说，可以表示为 <math><msub><mi>θ</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>θ</mi> <mn>1</mn></msub> <mi>x</mi> <mo>+</mo> <msub><mi>θ</mi> <mn>2</mn></msub> <msup><mi>x</mi> <mn>2</mn></msup></math> 的函数。由于 <math><mi>g</mi> <mo stretchy="false">(</mo> <mi>x</mi> <mo stretchy="false">)</mo> <mo>=</mo> <mi>sin</mi> <mo>⁡</mo> <mo stretchy="false">(</mo> <mi>x</mi> <mo stretchy="false">)</mo> <mo>+</mo> <mn>0.3</mn> <mi>x</mi></math> ，它不属于我们正在优化的函数集合。

让我们对我们的 50 个数据点进行多项式拟合：

```py
`poly` `=` `PolynomialFeatures``(``degree``=``2``,` `include_bias``=``False``)`
`poly_features` `=` `poly``.``fit_transform``(``xs``.``reshape``(``-``1``,` `1``)``)`

`model_deg2` `=` `LinearRegression``(``)``.``fit``(``poly_features``,` `ys``)`

```

```py
Fitted Model: 0.98 + -0.19x + 0.05x^2

```

再次，我们知道真实模型不是二次的（因为我们建立了它）。让我们绘制数据和拟合曲线：

![](assets/leds_16in11.png)

二次函数不太适合数据，并且也不能很好地表示底层曲线，因为我们选择的模型集（二阶多项式）无法捕捉<math><mi>g</mi></math>中的曲率。

如果我们重复这个过程，并从真实模型中生成另外50个点，并将二次多项式拟合到这些数据上，那么二次多项式的拟合系数会因为依赖于新数据集而改变。我们可以多次重复这个过程，并对拟合曲线取平均。这个平均曲线将类似于从我们真实模型中取50个数据点拟合的典型二次多项式最佳拟合曲线。为了演示这个概念，让我们生成25组50个数据点，并对每个数据集拟合一个二次多项式：

```py
`def` `fit``(``n``)``:`
    `xs_new` `=` `np``.``random``.``choice``(``np``.``arange``(``0``,` `10``,` `0.05``)``,` `size``=``n``)`
    `ys_new` `=` `g``(``xs_new``)` `+` `gen_noise``(``n``)`
    `X_new` `=` `xs_new``.``reshape``(``-``1``,` `1``)`
    `mod_new` `=` `LinearRegression``(``)``.``fit``(``poly``.``fit_transform``(``X_new``)``,` `ys_new``)`
    `return` `mod_new``.``predict``(``poly_features_x_full``)``.``flatten``(``)`

```

```py
`fits` `=` `[``fit``(``50``)` `for` `j` `in` `range``(``25``)``]`

```

我们可以在图中显示所有25个拟合模型以及真实函数<math><mi>g</mi></math>和拟合曲线的平均值<math><mrow><mover><mi>f</mi> <mo stretchy="false">¯</mo></mover></mrow></math> 。为此，我们使用透明度来区分重叠的曲线：

![](assets/leds_16in12.png)

我们可以看到25个拟合的二次多项式与数据有所不同。这个概念被称为*模型变异*。25个二次多项式的平均值由实线表示。平均二次多项式与真实曲线之间的差异称为*模型偏差*。

当信号<math><mi>g</mi></math>不属于模型空间<math><mrow><mi mathvariant="script">F</mi></mrow></math>时，我们有模型偏差。如果模型空间能很好地逼近<math><mi>g</mi></math>，那么偏差就很小。例如，一个10次多项式可以很接近我们示例中使用的<math><mi>g</mi></math>。另一方面，正如我们在本章前面看到的，高阶多项式可能会过度拟合数据并且在尝试接近数据时变化很大。模型空间越复杂，拟合模型的变化就越大。使用过于简单的模型会导致高模型偏差（<math><mi>g</mi></math>和<math><mrow><mover><mi>f</mi> <mo stretchy="false">¯</mo></mover></mrow></math>之间的差异），而使用过于复杂的模型可能会导致高模型方差（<math><mrow><mover><mi>f</mi> <mo stretchy="false">^</mo></mover></mrow></math>在<math><mrow><mover><mi>f</mi> <mo stretchy="false">¯</mo></mover></mrow></math>周围的波动）。这个概念被称为*偏差-方差权衡*。模型选择旨在平衡这些竞争性的拟合不足来源。

# 总结

在本章中，我们看到当我们最小化均方误差来拟合模型并评估时会出现问题。训练集-测试集分割帮助我们避开这个问题，其中我们用训练集拟合模型，并在设置好的测试数据上评估我们的拟合模型。

“过度使用”测试集非常重要，因此我们保持其与模型分离直到我们决定了一个模型。为了帮助我们做出决定，我们可能使用交叉验证，模拟将数据分为测试和训练集。同样重要的是，只使用训练集进行交叉验证，并将原始测试集远离任何模型选择过程。

正则化采取了不同的方法，通过惩罚均方误差来防止模型过度拟合数据。在正则化中，我们利用所有可用数据来拟合模型，但是缩小系数的大小。

偏差-方差折衷使我们能够更准确地描述本章中看到的建模现象：拟合不足与模型偏差有关；过度拟合导致模型方差增大。在[图16-4](#model-bias-variance-diagram)中，x轴表示模型复杂度，y轴表示模型不适配的这两个组成部分：模型偏差和模型方差。请注意，随着拟合模型的复杂性增加，模型偏差减少，模型方差增加。从测试误差的角度来看，我们看到这种误差首先减少，然后由于模型方差超过模型偏差的减少而增加。为了选择一个有用的模型，我们必须在模型偏差和模型方差之间取得平衡。

![](assets/leds_1604.png)

###### 图16-4\. 偏差-方差折衷

如果模型能够完全拟合人口过程，收集更多观察数据会减少偏差。如果模型本质上无法建模人口（如我们的合成示例），即使是无限数据也无法消除模型偏差。就方差而言，收集更多数据也会减少方差。数据科学的一个最新趋势是选择具有低偏差和高内在方差（例如神经网络）的模型，但收集许多数据点以使模型方差足够低以进行准确预测。虽然在实践中有效，但为这些模型收集足够的数据通常需要大量的时间和金钱。

创建更多的特征，无论其是否有用，通常会增加模型的方差。具有许多参数的模型具有许多可能的参数组合，因此比具有少量参数的模型具有更高的方差。另一方面，在模型中添加一个有用的特征（例如，当基础过程是二次的时添加二次特征），可以减少偏差。但即使添加一个无用的特征，很少会增加偏差。

熟悉偏差-方差折衷可以帮助您更好地拟合模型。并且使用诸如训练-测试分割、交叉验证和正则化等技术可以改善这个问题。

建模的另一个部分考虑了拟合系数和曲线的变化。我们可能希望为系数提供置信区间或未来观测的预测带。这些区间和带子给出了拟合模型准确性的感觉。接下来我们将讨论这个概念。

^([1](ch16.html#id1643-marker)) 这些数据来自于丹尼尔·T·卡普兰（Daniel T. Kaplan）（CreateSpace Independent Publishing Platform, 2009）。
