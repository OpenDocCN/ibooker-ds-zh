# 第七章。如何解释文本分类器

在前几章中，我们已经学习了许多关于针对非结构化文本数据的高级分析方法。从统计学开始，使用自然语言处理，我们从文本中找到了有趣的见解。

使用监督方法进行分类，我们通过训练算法将文本文档分配到已知类别。虽然我们已经检查了分类过程的质量，但我们忽略了一个重要的方面：我们不知道模型为什么决定将一个类别分配给一个文本。

如果类别是正确的，这可能听起来不重要。然而，在日常生活中，您经常必须*解释*您自己的决定，并使它们对他人*透明*。对于机器学习算法也是如此。

在现实项目中，您很可能经常听到“为什么算法分配了这个类别/情绪？”的问题。甚至在此之前，了解算法是如何学习的将帮助您通过使用不同的算法、添加特征、更改权重等来改进分类。与结构化数据相比，对于文本来说，这个问题更为重要，因为人类可以解释文本本身。此外，文本有许多人为因素，比如电子邮件中的签名，最好避免这些因素，并确保它们不是分类中的主要特征。

除了技术视角之外，还有一些法律方面需要注意。您可能需要证明您的算法没有偏见或不歧视。欧盟的GDPR甚至要求对公共网站上做出决策（比如只允许某种支付方式）的算法进行证明。

最后但同样重要的是，信任需要信息。如果您尽可能地公开您的结果，您将大大增加某人对您的方法的信心和信任。

# 你将学到什么，我们将构建什么

在本章中，我们将介绍几种解释监督机器学习模型结果的方法。在可能的情况下，我们将建立在先前章节中的分类示例之上。

我们将从重新审视[第六章](ch06.xhtml#ch-classification)中的错误报告的分类开始。一些报告被正确分类，一些没有。我们将退后一步，分析分类是否总是二进制决策。对于某些模型来说，它不是，我们将计算错误报告属于某个类别的概率，并与正确值（所谓的*地* *实*）进行核对。

在接下来的部分中，我们将分析哪些特征决定了模型的决策。我们可以使用支持向量机来计算这一点。我们将尝试解释结果，并看看我们是否可以利用这些知识来改进方法。

之后，我们将采取更一般的方法，并介绍*本地可解释的模型无关解释*（LIME）。LIME（几乎）不依赖于特定的机器学习模型，可以解释许多算法的结果。

近年来人们在研究可解释人工智能方面投入了大量工作，并提出了一种更复杂的模型称为*Anchor*，我们将在本章的最后部分介绍它。

在学习了本章之后，您将了解到解释监督学习模型结果的不同方法。您将能够将这些方法应用于您自己的项目，并决定哪种方法最适合您的特定需求。您将能够解释结果并创建直观的可视化，以便非专家也能轻松理解。

# 蓝图：使用预测概率确定分类置信度

您可能还记得[第6章](ch06.xhtml#ch-classification)中的例子，我们尝试根据其组件对缺陷报告进行分类。现在我们将使用在该章节中找到的最佳参数来训练支持向量机。其余的符号表示保持不变：

```py
svc = SVC(kernel="linear", C=1, probability=True, random_state=42)
svc.fit(X_train_tf, Y_train)

```

如果您还记得分类报告，我们的平均精确度和召回率为75%，因此分类效果相当不错。但也有一些情况下预测与实际值不同。现在我们将更详细地查看这些预测结果，以了解是否有可以用来区分“好”和“坏”预测的模式，而不查看实际结果，因为在真实的分类场景中这些将是未知的。

为此，我们将使用支持向量机模型的`predict_proba`函数，该函数告诉我们SVM的内部情况，即它对各个类别计算的概率（显然，预测本身的概率最高）。^([1](ch07.xhtml#idm45634192517144)) 作为参数，它期望一个由文档向量组成的矩阵。结果是不同类别的概率。作为第一步，我们将从预测结果构建一个`DataFrame`：

```py
X_test_tf = tfidf.transform(X_test)
Y_pred = svc.predict(X_test_tf)
result = pd.DataFrame({ 'text': X_test.values, 'actual': Y_test.values,
                        'predicted': Y_pred })

```

让我们尝试使用测试数据集的一个文档，并假设我们想优化我们的分类，主要关注预测错误的情况：

```py
result[result["actual"] != result["predicted"]].head()

```

`Out:`

|   | 文本 | 实际 | 预测 |
| --- | --- | --- | --- |
| 2 | 在执行JDT/UI时Delta处理器中的NPE... | Core | UI |
| 15 | 在编辑器中插入文本块时排版不佳... | UI | 文本 |
| 16 | 在调试相同对象时的差异... | Debug | Core |
| 20 | 模板中对类成员使用Foreach不起作用... | Core | UI |
| 21 | 交换比较运算符的左右操作数... | UI | Core |

文档21看起来是一个很好的候选。预测的类“Core”是错误的，但“left”和“right”听起来也像是UI（这将是正确的）。让我们深入研究一下：

```py
text = result.iloc[21]["text"]
print(text)

```

`Out:`

```py
exchange left and right operands for comparison operators changes semantics
Fix for Bug 149803 was not good.; ; The right fix should do the following;
if --> if --> if ; if ; if
```

这看起来是更详细分析的一个不错的候选项，因为它包含了既可能是核心也可能是 UI 的词汇。也许如果我们查看概率，我们就能更详细地理解这一点。计算这个是相当容易的：

```py
 svc.predict_proba(X_test_tf[21])

```

`输出：`

```py
array([[0.002669, 0.46736578, 0.07725225, 0.00319434, 0.06874877,
        0.38076986]])

```

记住类别的顺序是 APT、核心、调试、文档、文本和 UI，该算法对核心的确信度比对 UI 高一些，而 UI 则是其次选择。

这种情况总是这样吗？我们将尝试找出答案，并计算测试数据集中所有文档的决策概率，并将其添加到一个 `DataFrame` 中：

```py
class_names = ["APT", "Core", "Debug", "Doc", "Text", "UI"]
prob = svc.predict_proba(X_test_tf)
# new dataframe for explainable results
er = result.copy().reset_index()
for c in enumerate(class_names):
    er[c] = prob[:, i]

```

让我们看看数据帧的一些样本，并找出是否在算法相当确信其决策（即，所选类别的概率远高于其他类别）的情况下预测更准确：

```py
er[["actual", "predicted"] + class_names].sample(5, random_state=99)

```

`输出：`

|   | 实际 | 预测 | APT | 核心 | 调试 | 文档 | 文本 | UI |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 266 | UI | UI | 0.000598 | 0.000929 | 0.000476 | 0.001377 | 0.224473 | 0.772148 |
| 835 | 文本 | 文本 | 0.002083 | 0.032109 | 0.001481 | 0.002085 | 0.696666 | 0.265577 |
| 998 | 文本 | 文本 | 0.000356 | 0.026525 | 0.003425 | 0.000673 | 0.942136 | 0.026884 |
| 754 | 核心 | 文本 | 0.003862 | 0.334308 | 0.011312 | 0.015478 | 0.492112 | 0.142927 |
| 686 | UI | UI | 0.019319 | 0.099088 | 0.143744 | 0.082969 | 0.053174 | 0.601705 |

查看表格，只有一个错误的预测（754）。在这种情况下，算法相当“不确定”，并且以低于 50% 的概率决定了类别。我们能找到这种情况的模式吗？

让我们尝试构建两个 `DataFrame`，一个包含正确的预测，另一个包含错误的预测。然后，我们将分析最高概率的分布，看看是否能找到任何差异：

```py
er['max_probability'] = er[class_names].max(axis=1)
correct = (er[er['actual'] == er['predicted']])
wrong   = (er[er['actual'] != er['predicted']])

```

我们现在将其绘制成直方图：

```py
correct["max_probability"].plot.hist(title="Correct")
wrong["max_probability"].plot.hist(title="Wrong")

```

`输出：`

![](Images/btap_07in01.jpg)![](Images/btap_07in02.jpg)

我们可以看到，在正确预测的情况下，模型经常以高概率决定，而当决策错误时，概率明显较低。正如我们稍后将看到的那样，错误类别中的高概率小峰值是由于短文本或缺失单词造成的。

最后，我们将看看是否可以改进结果，如果我们只考虑已经做出概率超过 80% 决策的情况：

```py
high = er[er["max_probability"] > 0.8]
print(classification_report(high["actual"], high["predicted"]))

```

`输出：`

```py
                precision    recall  f1-score   support

         APT       0.90      0.75      0.82        12
        Core       0.94      0.89      0.92       264
       Debug       0.94      0.99      0.96       202
         Doc       1.00      0.67      0.80         3
        Text       0.78      0.75      0.77        72
          UI       0.90      0.92      0.91       342

    accuracy                           0.91       895
   macro avg       0.91      0.83      0.86       895
weighted avg       0.91      0.91      0.91       895

```

将其与原始结果进行比较，如下所示：

```py
print(classification_report(er["actual"], er["predicted"]))

```

`输出：`

```py
              precision    recall  f1-score   support

         APT       0.90      0.56      0.69        16
        Core       0.76      0.77      0.76       546
       Debug       0.90      0.78      0.84       302
         Doc       1.00      0.25      0.40        12
        Text       0.64      0.51      0.57       236
          UI       0.72      0.82      0.77       699

    accuracy                           0.75      1811
   macro avg       0.82      0.62      0.67      1811
weighted avg       0.75      0.75      0.75      1811

```

我们可以看到，在预测核心、调试、文本和用户界面组件的精度方面，我们有了显著的改进，同时也提高了召回率。这很棒，因为支持向量机（SVM）的解释使我们进入了一个数据子集，分类器在这里工作得更好。然而，在样本较少的组件（Apt、Doc）中，实际上只改善了召回率。看来这些类别中的样本太少了，算法基于文本的信息也太少，难以作出决策。在 Doc 的情况下，我们刚刚移除了大部分属于此类的文档，从而提高了召回率。

然而，改进是有代价的。我们排除了超过 900 份文件，大约是数据集的一半。因此，总体而言，我们在较小的数据集中实际上找到了更少的文档！在某些项目中，让模型仅在“确定”情况下做决策，并且丢弃模棱两可的情况（或手动分类），可能是有用的。这通常取决于业务需求。

在这一部分中，我们发现了预测概率与结果质量之间的相关性。但我们尚未理解模型如何预测（即使用哪些单词）。我们将在下一节中进行分析。

# 蓝图：测量预测模型的特征重要性

在本节中，我们希望找出哪些特征对模型找到正确类别是相关的。幸运的是，我们的 SVM 类可以告诉我们必要的参数（称为*系数*）：

```py
svc.coef_

```

`输出：`

```py
<15x6403 sparse matrix of type '<class 'numpy.float64'>'
       with 64451 stored elements in Compressed Sparse Row format>

```

6403 是词汇表的大小（检查`len(tfidf.get_feature_names()`），但是 15 是从哪里来的呢？这有点复杂。从技术上讲，系数组织成一个矩阵，每个类与其他类以一对一的方式竞争。由于我们有六个类别，并且类别不必与自身竞争，因此有 15 个组合（组合数 6 选 2）。这 15 个系数如表 7-1 所述组织。

表 7-1\. 多类支持向量分类器的系数布局

|   | APT | 核心 | 调试 | Doc | 文本 | 用户界面 |
| --- | --- | --- | --- | --- | --- | --- |
| APT |   | 0 | 1 | 2 | 3 | 4 |
| 核心 |   |   | 5 | 6 | 7 | 8 |
| 调试 |   |   |   | 9 | 10 | 11 |
| Doc |   |   |   |   | 12 | 13 |
| 文本 |   |   |   |   |   | 14 |
| 用户界面 |   |   |   |   |   |   |

# 系数结构取决于机器学习模型

如果您使用其他分类器，则系数可能具有完全不同的组织结构。即使对于 SVM，使用 SGDClassifier 创建的非线性模型也每类只有一个系数集。当我们讨论 ELI5 时，我们将看到一些示例。

应首先阅读行，因此如果我们想要了解模型如何区分 APT 和核心组件，我们应该查看系数的索引 0。然而，我们更感兴趣的是核心和 UI 的差异，因此我们取索引 8\. 在第一步中，我们按照它们的值对系数进行排序，并保留词汇位置的索引：

```py
# coef_[8] yields a matrix, A[0] converts to array and takes first row
coef = svc.coef_[8].A[0]
vocabulary_positions = coef.argsort()
vocabulary = tfidf.get_feature_names()

```

接下来，我们现在获取顶部的正负贡献：

```py
top_words = 10
top_positive_coef = vocabulary_positions[-top_words:].tolist()
top_negative_coef = vocabulary_positions[:top_words].tolist()

```

然后，我们将这些聚合到一个 `DataFrame` 中，以便更容易地显示结果：

```py
core_ui = pd.DataFrame([[vocabulary[c],
                  coef[c]] for c in top_positive_coef + top_negative_coef],
                  columns=["feature", "coefficient"]).sort_values("coefficient")

```

我们希望可视化系数的贡献，以便易于理解。正值偏好核心组件，负值偏好 UI，如图 [7-1](#fig-coefficients-core-ui) 所示。为此，我们使用以下方法：

```py
core_ui.set_index("feature").plot.barh()

```

这些结果非常容易解释。SVM 模型很好地学习到，*compiler* 和 *ast* 这些词语是特定于核心组件的，而 *wizard*、*ui* 和 *dialog* 则用于识别 UI 组件中的错误。似乎在 UI 中更倾向于快速修复，这强调了核心的长期稳定性。

我们刚刚找到了整个 SVM 模型选择核心和 UI 之间的重要特征。但这并不表明哪些特征对于识别可以归类为核心的 bug 很重要。如果我们想要获取这些核心组件的特征，并考虑到先前的矩阵，我们需要索引 5、6、7 和 8\. 采用这种策略，我们忽略了 APT 和核心之间的差异。要考虑到这一点，我们需要减去索引 0：

```py
c = svc.coef_
coef = (c[5] + c[6] + c[7] + c[8] - c[0]).A[0]
vocabulary_positions = coef.argsort()

```

![](Images/btap_07in03.jpg)

###### 图 7-1\. UI 的词语贡献（负）和核心的贡献（正）。

其余代码几乎与之前的代码相同。我们现在将图表扩展到 20 个词语（图 [7-2](#fig-coefficients-core)）：

```py
top_words = 20
top_positive_coef = vocabulary_positions[-top_words:].tolist()
top_negative_coef = vocabulary_positions[:top_words].tolist()
core = pd.DataFrame([[vocabulary[c], coef[c]]
                      for c in top_positive_coef + top_negative_coef], 
                    columns=["feature", "coefficient"]).\
          sort_values("coefficient")
core.set_index("feature").plot.barh(figsize=(6, 10),
              color=[['red']*top_words + ['green']*top_words])

```

在图表中，您可以看到模型用于识别核心组件的许多词语，以及主要用于识别其他组件的词语。

您可以使用本蓝图中描述的方法，使 SVM 模型的结果透明和可解释。在许多项目中，这已被证明非常有价值，因为它消除了机器学习的“魔力”和主观性。

这方法效果相当好，但我们还不知道模型对某些词语的变化有多敏感。这是一个更复杂的问题，我们将在下一节中尝试回答。

![](Images/btap_07in04.jpg)

###### 图 7-2\. 偏好或反对核心组件的系数。

# 蓝图：使用 LIME 解释分类结果

LIME 是 [“局部可解释模型无关解释”](https://oreil.ly/D8cIN) 的首字母缩写，是一个用于可解释机器学习的流行框架。它是在 [华盛顿大学](https://oreil.ly/Q8zly) 构想的，并且在 [GitHub 上](https://oreil.ly/bErrv) 公开可用。

让我们看看LIME的定义特征。它通过单独查看每个预测*局部地*工作。通过修改输入向量以找到预测敏感的局部组件来实现这一点。

# 可解释性需要计算时间

运行解释器代码可能需要相当长的时间。我们尝试通过调整示例的方式，让您在普通计算机上等待时间不超过10分钟。但是，通过增加样本大小，这可能很容易需要几个小时。

从向量周围的行为来看，它将得出哪些组件更重要或不重要的结论。LIME将可视化贡献，并解释算法*对个别文档*的决策机制。

LIME不依赖于特定的机器学习模型，可以应用于多种问题。并非所有模型都符合条件；模型需要预测类别的概率。并非所有支持向量机模型都能做到这一点。此外，在像文本分析中常见的高维特征空间中使用复杂模型进行预测时间较长并不是很实际。由于LIME试图局部修改特征向量，因此需要执行大量预测，在这种情况下需要很长时间才能完成。

最后，LIME将根据每个样本生成模型解释，并帮助您理解模型。您可以用它来改进模型，也可以用来解释分类的工作原理。虽然模型仍然是黑箱，但您将获得一些可能发生在箱子里的知识。

让我们回到前一节的分类问题，并尝试为几个样本找到LIME解释。由于LIME需要文本作为输入和分类概率作为输出，我们将向量化器和分类器安排在*管道*中：

```py
from sklearn.pipeline import make_pipeline
pipeline = make_pipeline(tfidf, best_model)

```

如果我们给它一些文本，流水线应该能够进行预测，就像这里做的那样：

```py
pipeline.predict_proba(["compiler not working"])

```

`输出:`

```py
array([[0.00240522, 0.95605684, 0.00440957, 0.00100242, 0.00971824,
        0.02640771]])

```

分类器建议将此置于类别2中的概率非常高，即核心。因此，我们的流水线正按照我们希望的方式运行：我们可以将文本文档作为参数传递给它，并返回文档属于每个类别的概率。现在是打开LIME的时候了，首先导入该包（您可能需要使用`pip`或`conda`先安装该包）。之后，我们将创建一个解释器，这是LIME的核心元素之一，负责解释单个预测：

```py
from lime.lime_text import LimeTextExplainer
explainer = LimeTextExplainer(class_names=class_names)

```

我们检查`DataFrame`中错误预测的类别如下：

```py
er[er["predicted"] != er["actual"]].head(5)

```

`输出:`

|   | 索引 | 文本 | 实际 | 预测 | APT | 核心 | 调试 | 文档 | 文本 | UI |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | 2 | Delta处理器中的NPE执行JDT/UI ... | 核心 | UI | 0.003357 | 0.309548 | 0.046491 | 0.002031 | 0.012309 | 0.626265 |
| 15 | 15 | 在编辑器中插入文本块严重对齐不良... | UI | 文本 | 0.001576 | 0.063076 | 0.034610 | 0.003907 | 0.614473 | 0.282356 |
| 16 | 16 | 调试相同对象时的差异 W... | 调试 | 核心 | 0.002677 | 0.430862 | 0.313465 | 0.004193 | 0.055838 | 0.192965 |
| 20 | 20 | 模板的 foreach 对类成员不起作用... | 核心 | UI | 0.000880 | 0.044018 | 0.001019 | 0.000783 | 0.130766 | 0.822535 |
| 21 | 21 | 交换比较中左右操作数... | UI | 核心 | 0.002669 | 0.467366 | 0.077252 | 0.003194 | 0.068749 | 0.380770 |

看看相应记录（我们的情况下是第 21 行）：

```py
id = 21
print('Document id: %d' % id)
print('Predicted class =', er.iloc[id]["predicted"])
print('True class: %s' % er.iloc[id]["actual"])

```

`Out:`

```py
Document id: 21
Predicted class = Core
True class: UI

```

现在是 LIME 向我们解释的时候了！

```py
exp = explainer.explain_instance(result.iloc[id]["text"],
      pipeline.predict_proba, num_features=10, labels=[1, 5])
print('Explanation for class %s' % class_names[1])
print('\n'.join(map(str, exp.as_list(label=1))))
print()
print('Explanation for class %s' % class_names[5])
print('\n'.join(map(str, exp.as_list(label=5))))

```

`Out:`

```py
Explanation for class Core
('fix', -0.14306948642919184)
('Bug', 0.14077384623641856)
('following', 0.11150012169630388)
('comparison', 0.10122423126000728)
('Fix', -0.0884162779420967)
('right', 0.08315255286108318)
('semantics', 0.08143857054730141)
('changes', -0.079427782008582)
('left', 0.03188240169394561)
('good', -0.0027133756042246504)

Explanation for class UI
('fix', 0.15069083664026453)
('Bug', -0.14853911521141774)
('right', 0.11283930406785869)
('comparison', -0.10654654371478504)
('left', -0.10391669738035045)
('following', -0.1003931859632352)
('semantics', -0.056644426928774076)
('Fix', 0.05365037666619837)
('changes', 0.040806391076561165)
('good', 0.0401761761717476)

```

LIME 展示了哪些词语它认为对某个类别有利（正面）或不利（负面）。这与我们在 SVM 示例中实现的情况非常相似。更好的是，现在它独立于模型本身；它只需要支持`predict_proba`（这也适用于随机森林等）。

使用 LIME，您可以将分析扩展到更多类别，并创建它们特定词语的图形表示：

```py
exp = explainer.explain_instance(result.iloc[id]["text"],
            pipeline.predict_proba, num_features=6, top_labels=3)
exp.show_in_notebook(text=False)

```

`Out:`

![](Images/btap_07_svgcombo_1.jpg)![](Images/btap_07_svgcombo_2.jpg)

这看起来很直观，更适合解释甚至包含在演示中。我们可以清楚地看到*fix* 和 *right* 对于分配 UI 类别至关重要，同时反对核心。然而，*Bug* 表示核心，正如*comparison* 和 *semantics* 一样。不幸的是，这不是人类接受作为分类规则的样子；它们似乎过于具体，没有抽象化。换句话说，我们的模型看起来*过拟合*。

# 改进模型

有了这些知识和熟悉票务的专家的经验，您可以改进模型。例如，我们可以询问*Bug* 是否真的特定于核心，或者我们最好将其作为停用词。把所有内容转换为小写可能也会证明有用。

LIME 甚至可以帮助您找到有助于全面解释模型性能的代表性样本。这个功能被称为*子模块挑选*，工作原理如下：

```py
from lime import submodular_pick
import numpy as np
np.random.seed(42)
lsm = submodular_pick.SubmodularPick(explainer, er["text"].values,
                                        pipeline.predict_proba,
                                        sample_size=100,
                                        num_features=20,
                                        num_exps_desired=5)

```

个别“挑选”可以像之前笔记本中显示的那样进行可视化，并且现在更加完整，带有高亮显示。我们在这里只展示了第一个挑选：

```py
lsm.explanations[0].show_in_notebook()

```

`Out:`

![](Images/btap_07_svgcombo_3.jpg)

在以下情况下，我们可以解释结果，但它似乎并没有学习抽象化，这又是*过拟合*的迹象。

![](Images/btap_07_highlightwords.jpg)

LIME 软件模块适用于 scikit-learn 中的线性支持向量机，但不适用于具有更复杂内核的支持向量机。图形化展示很好，但不直接适合演示。因此，我们将看看 ELI5，这是一种替代实现，试图克服这些问题。

# 蓝图：使用 ELI5 解释分类结果

ELI5（“Explain it to me like I’m 5”）是另一个流行的机器学习解释软件库，也使用LIME算法。由于它可用于非线性SVM，并且具有不同的API，我们将简要介绍它，并展示如何在我们的案例中使用它。

ELI5需要一个使用`libsvm`训练过的模型，而我们之前的SVC模型不幸不是这样的。幸运的是，训练SVM非常快速，因此我们可以用相同的数据创建一个新的分类器，但使用基于`libsvm`的模型，并检查其性能。你可能还记得[第6章](ch06.xhtml#ch-classification)中的分类报告，它提供了模型质量的良好总结：

```py
from sklearn.linear_model import SGDClassifier
svm = SGDClassifier(loss='hinge', max_iter=1000, tol=1e-3, random_state=42)
svm.fit(X_train_tf, Y_train)
Y_pred_svm = svm.predict(X_test_tf)
print(classification_report(Y_test, Y_pred_svm))

```

`Out:`

```py
              precision    recall  f1-score   support

         APT       0.89      0.50      0.64        16
        Core       0.77      0.78      0.77       546
       Debug       0.85      0.84      0.85       302
         Doc       0.75      0.25      0.38        12
        Text       0.62      0.59      0.60       236
          UI       0.76      0.79      0.78       699

    accuracy                           0.76      1811
   macro avg       0.77      0.62      0.67      1811
weighted avg       0.76      0.76      0.76      1811

```

看看最后一行，这大致与我们使用SVC取得的效果一样好。因此，解释它是有意义的！使用ELI5，找到这个模型的解释是很容易的：

```py
import eli5
eli5.show_weights(svm, top=10, vec=tfidf, target_names=class_names)

```

![](Images/btap_07_eli5_1.jpg)

正面特征（即词汇）显示为绿色。更浓烈的绿色意味着该词对应类别的贡献更大。红色则完全相反：出现在红色中的词汇会“排斥”类别（例如，第二行下部的“refactoring”强烈排斥`Core`类）。`<BIAS>`则是一个特例，包含所谓的*截距*，即模型的系统性失败。

如您所见，我们现在为各个类别获得了权重。这是由于非线性SVM模型在多类场景下与SVC不同的工作方式。每个类别都“打分”，没有竞争。乍一看，这些词看起来非常合理。

ELI5还可以解释单个观察结果：

```py
eli5.show_prediction(svm, X_test.iloc[21],  vec=tfidf, target_names=class_names)

```

![](Images/btap_07_eli5_2.jpg)![](Images/btap_07_eli5_3.jpg)![](Images/btap_07_eli5_4.jpg)![](Images/btap_07_eli5_5.jpg)![](Images/btap_07_eli5_6.jpg)![](Images/btap_07_eli5_7.jpg)

这是一个很好的可视化工具，用于理解哪些词汇对算法决定类别具有贡献。与原始的LIME软件包相比，使用ELI5需要的代码要少得多，你可以将ELI5用于非线性SVM模型。根据你的分类器和使用情况，你可能会选择LIME或ELI5。由于使用了相同的方法，结果应该是可比较的（如果不是相同的）。

# 工作正在进行中

ELI5仍在积极开发中，您可能会在新版本的scikit-learn中遇到困难。在本章中，我们使用了ELI5版本0.10.1。

ELI5是一个易于使用的软件库，用于理解和可视化分类器的决策逻辑，但它也受到底层LIME算法的缺点的影响，例如只能通过示例来解释的可解释性。为了使黑盒分类更透明，获得模型使用的“规则”将是有见地的。这是华盛顿大学团队创建后续项目Anchor的动机。

# 蓝图：使用Anchor解释分类结果

类似于LIME，[Anchor](https://oreil.ly/qSDMl)与任何黑盒模型都兼容。作为解释工具，它创建了规则，即所谓的*锚点*，用于解释模型的行为。阅读这些规则，你不仅能够解释模型的预测，还能以与模型学习相同的方式进行预测。

相较于LIME，Anchor在通过规则更好地解释模型方面具有显著优势。然而，软件本身还是比较新的，仍在不断完善中。并非所有示例对我们都适用，因此我们选择了一些有助于解释分类模型的方法。

## 使用带有屏蔽词的分布

Anchor有多种使用方式。我们从所谓的*未知*分布开始。Anchor将通过用词汇*unknown*替换预测中被认为不重要的现有标记，解释模型的决策方式。

再次，我们将使用ID为21的文档。在这种情况下，分类器需要在两个概率大致相同的类别之间进行选择，这对研究是一个有趣的示例。

为了在文本中创建（语义）差异，Anchor使用spaCy的词向量，并需要包含这些向量的spaCy模型，例如`en_core_web_lg`。

因此，作为先决条件，您应该安装`anchor-exp`和`spacy`（使用`conda`或`pip`），并加载以下模型：

```py
python -m spacy download en_core_web_lg
```

在第一步中，我们可以实例化我们的解释器。解释器具有一些概率元素，因此最好同时重新启动随机数生成器：

```py
np.random.seed(42)
explainer_unk = anchor_text.AnchorText(nlp, class_names, \
                use_unk_distribution=True)

```

让我们检查预测结果及其替代方案，并将其与真实情况进行比较。`predicted_class_ids`包含预测类的索引，按概率降序排列，因此元素0是预测值，元素1是其最接近的竞争者：

```py
text = er.iloc[21]["text"]
actual = er.iloc[21]["actual"]
# we want the class with the highest probability and must invert the order
predicted_class_ids = np.argsort(pipeline.predict_proba([text])[0])[::-1]
pred = explainer_unk.class_names[predicted_class_ids[0]]
alternative = explainer_unk.class_names[predicted_class_ids[1]]
print(f'predicted {pred}, alternative {alternative}, actual {actual}')

```

`Out:`

```py
predicted Core, alternative UI, actual UI

```

在下一步中，我们将让算法找出预测的规则。参数与之前的LIME相同：

```py
exp_unk = explainer_unk.explain_instance(text, pipeline.predict, threshold=0.95)

```

计算时间取决于CPU的速度，可能需要高达60分钟。

现在一切都包含在解释器中，因此我们可以查询解释器，了解模型的内部工作情况：

```py
print(f'Rule: {" AND ".join(exp_unk.names())}')
print(f'Precision: {exp_unk.precision()}')

```

`Out:`

```py
Rule: following AND comparison AND Bug AND semantics AND for
Precision: 0.9865771812080537

```

因此，规则告诉我们，单词*following*和*comparison*与*Bug*和*semantic*的组合会导致“Core”预测，精度超过98%，但不幸的是这是错误的。现在，我们还可以找到模型将其分类为Core的典型示例：

```py
print(f'Made-up examples where anchor rule matches and model predicts {pred}\n')
print('\n'.join([x[0] for x in exp_unk.examples(only_same_prediction=True)]))

```

下面显示的UNK标记代表“未知”，意味着对应位置的词汇不重要：

```py
Made-up examples where anchor rule matches and model predicts Core

UNK left UNK UNK UNK UNK comparison operators UNK semantics Fix for Bug UNK UNK
exchange left UNK UNK operands UNK comparison operators changes semantics Fix fo
exchange UNK and UNK operands UNK comparison UNK UNK semantics UNK for Bug UNK U
exchange UNK and right UNK for comparison UNK UNK semantics UNK for Bug 149803 U
UNK left UNK UNK operands UNK comparison UNK changes semantics UNK for Bug 14980
exchange left UNK right UNK UNK comparison UNK changes semantics Fix for Bug UNK
UNK UNK and right operands for comparison operators UNK semantics Fix for Bug 14
UNK left and right operands UNK comparison operators changes semantics UNK for B
exchange left UNK UNK operands UNK comparison operators UNK semantics UNK for Bu
UNK UNK UNK UNK operands for comparison operators changes semantics Fix for Bug

```

我们还可以要求提供符合规则但模型预测错误类的示例：

```py
print(f'Made-up examples where anchor rule matches and model predicts \
 {alternative}\n')
print('\n'.join([x[0] for x in exp_unk.examples(partial_index=0, \
      only_different_prediction=True)]))

```

`Out:`

```py
Made-up examples where anchor rule matches and model predicts UI

exchange left and right UNK for UNK UNK UNK UNK Fix for UNK 149803 was not UNK .
exchange left UNK UNK UNK for UNK UNK UNK semantics Fix for Bug 149803 UNK not U
exchange left UNK UNK operands for comparison operators UNK UNK Fix UNK Bug 1498
exchange left UNK right operands UNK comparison UNK UNK UNK Fix for UNK UNK UNK
exchange left and right operands UNK UNK operators UNK UNK Fix UNK UNK UNK UNK U
UNK UNK and UNK UNK UNK comparison UNK UNK UNK Fix for UNK UNK was not good UNK
exchange left and UNK UNK UNK UNK operators UNK UNK Fix UNK Bug 149803 was not U
exchange left and right UNK UNK UNK operators UNK UNK UNK for Bug 149803 UNK UNK
exchange left UNK right UNK for UNK operators changes UNK Fix UNK UNK UNK was no
UNK left UNK UNK operands UNK UNK operators changes UNK UNK for UNK 149803 was n

```

老实说，这对模型来说并不是一个好结果。我们本来期望模型学习的底层规则会对不同组件特定的单词比较敏感。然而，并没有明显的理由可以解释为什么*following*和*Bug*会特定于核心。这些都是一些通用词汇，不太具有任何类别的特征。

UNK令牌有点误导。即使它们在这个样本中并不重要，它们可能被其他真实的单词替换，这些单词会影响算法的决策。Anchor也可以帮助我们说明这一点。

## 使用真实词语进行工作

通过在解释器的原始构造函数中替换`use_unk_distribution=False`，我们可以告诉Anchor使用真实词语（类似于使用spaCy的词向量替换）并观察模型的行为：

```py
np.random.seed(42)
explainer_no_unk = anchor_text.AnchorText(nlp, class_names,
                   use_unk_distribution=False, use_bert=False)
exp_no_unk = explainer_no_unk.explain_instance(text, pipeline.predict,
             threshold=0.95)
print(f'Rule: {" AND ".join(exp_no_unk.names())}')
print(f'Precision: {exp_no_unk.precision()}')

```

`输出：`

```py
Rule: following AND Bug AND comparison AND semantics AND left AND right
Precision: 0.9601990049751243

```

这些规则与之前未知的分布有些不同。似乎有些单词变得更加特定于核心，如*left*和*right*，而其他词语如*for*则消失了。

让我们还让Anchor生成一些替代文本，这些文本也会（错误地）被分类为核心，因为前面的规则也适用：

```py
Examples where anchor applies and model predicts Core:

exchange left and right suffixes for comparison operators affects semantics NEED
exchange left and right operands for comparison operators depends semantics UPDA
exchange left and right operands for comparison operators indicates semantics so
exchange left and right operands for comparison operators changes semantics Firm
exchange left and right operands into comparison dispatchers changes semantics F
exchange left and right operands with comparison operators changes semantics Fix
exchange left and right operands beyond comparison operators changes semantics M
exchange left and right operands though comparison representatives changes seman
exchange left and right operands before comparison operators depends semantics M
exchange left and right operands as comparison operators changes semantics THING

```

一些单词已经改变，并且并没有影响分类结果。在某些情况下，只是介词，通常情况下这不会影响结果。然而，*operators*也可以被*dispatchers*替换而不会影响结果。Anchor向您展示它对这些修改是稳定的。

将先前的结果与模型正确预测“UI”的结果进行比较。同样，这种差异影响单词如*changes*、*metaphors*等，这些单词显然比前一个例子中的较小修改更具意义，但很难想象你作为一个人类会将这些词解释为不同类别的信号：

```py
Examples where anchor applies and model predicts UI:

exchange left and good operands for comparison operators changes metaphors Fix i
exchange landed and right operands for comparison supervisors changes derivation
exchange left and happy operands for correlation operators changes equivalences
exchange left and right operands for scenario operators changes paradigms Fix be
exchange left and right operands for trade customers occurs semantics Fix as BoT
exchange did and right operands than consumer operators changes analogies Instal
exchange left and few operands for reason operators depends semantics Fix for Bu
exchange left and right operands for percentage operators changes semantics MESS
exchange left and right pathnames after comparison operators depends fallacies F
exchange left and right operands of selection operators changes descriptors Fix

```

Anchor还有一种直观的方式在笔记本中显示结果，重要的单词会被突出显示，同时还包括它计算出的规则：^([2](ch07.xhtml#idm45634189444808))

```py
exp_unk.show_in_notebook()

```

`输出：`

![](Images/btap_07_highlightwords2.jpg)

由于你很可能也熟悉软件开发，单靠规则很难确定正确的类别。换句话说，这意味着当模型使用语料库训练时，模型似乎是相当脆弱的。只有那些具有大量背景知识的项目贡献者才能可能真正确定“正确”的类别（我们稍后将在[第11章](ch11.xhtml#ch-sentiment)回顾）。因此，发现分类器有效并不一定意味着它真正学习的方式对我们是透明的。

总结本节，Anchor 很有趣。Anchor 的作者选择版本号 0.0.1 并非偶然；该程序仍处于起步阶段。在我们的实验中，我们看到了一些小问题，要使其在生产环境中运行，还需要改进许多事情。但从概念上来说，它已经非常令人信服，可以用于解释单个预测并使模型透明化。特别是计算出的规则几乎是独一无二的，任何其他解决方案都无法创建。

# 结语

使用本章介绍的技术将有助于使您的模型预测更加透明。

从技术角度来看，这种透明性可以极大地帮助您在选择竞争模型或改进特征模型时提供支持。本章介绍的技术能够深入了解模型的“内部运作”，有助于检测和改进不可信的模型。

从商业角度来看，可解释性对项目来说是一个很好的销售主张。如果不只是追求黑盒模型，而是使模型透明化，那么在谈论模型并展示它们时会更容易。最近的文章在[福布斯](https://oreil.ly/Xcfjx)和[VentureBeat](https://oreil.ly/SIa-R)上已经专注于这一有趣的发展。当您想要构建可信的机器学习解决方案时，“信任”模型将变得越来越重要。

可解释人工智能是一个年轻的领域。我们可以预期未来会看到巨大的进步，更好的算法和改进的工具。

大多数情况下，机器学习方法都很好地作为黑盒模型运行。只要结果一致，我们就不需要为模型辩护，这样也挺好。但如果其中任何一个受到质疑（这种情况变得越来越普遍），那么可解释人工智能的时代就已经到来了。

^([1](ch07.xhtml#idm45634192517144-marker)) 从图形上来看，您可以将这些概率视为样本到由 SVM 定义的超平面的距离。

^([2](ch07.xhtml#idm45634189444808-marker)) 我们在让此工作起来时遇到了一些困难，因为它只适用于数值类别。我们计划提交一些拉取请求，以使上游也适用于文本类别。
