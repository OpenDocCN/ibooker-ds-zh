# 第8章 无监督方法：主题建模和聚类

当处理大量文档时，您想要在不阅读所有文档的情况下首先问的问题之一是“它们在谈论什么？”您对文档的主题感兴趣，即文档中经常一起使用的（理想情况下是语义的）单词。

主题建模试图通过使用统计技术从文档语料库中找出主题来解决这个挑战。根据您的向量化（见第5章），您可能会发现不同类型的主题。主题由特征（单词、n-gram等）的概率分布组成。

主题通常彼此重叠；它们并不明确分开。文档也是如此：不可能将文档唯一地分配给一个主题；文档始终包含不同主题的混合体。主题建模的目的不是将主题分配给任意文档，而是找出语料库的全局结构。

通常，一组文档具有由类别、关键词等确定的显式结构。如果我们想要查看语料库的有机构成，那么主题建模将对揭示潜在结构有很大帮助。

主题建模已经被人们熟知很长一段时间，并在过去的15年中获得了巨大的流行，这主要归因于LDA（一种用于发现主题的随机方法）的发明。LDA灵活多变，允许进行许多修改。然而，它并不是主题建模的唯一方法（尽管通过文献，您可能会认为它是唯一的，因为很多文献都倾向于LDA）。概念上更简单的方法包括非负矩阵分解、奇异值分解（有时称为LSI）等。

# 您将学到什么以及我们将构建什么

在这一章中，我们将深入研究各种主题建模方法，试图找到这些方法之间的差异和相似之处，并在同一个用例上运行它们。根据您的需求，尝试单一方法可能是个不错的主意，但比较几种方法的结果也是一个好选择。

在学习了本章后，您将了解到不同的主题建模方法及其特定的优缺点。您将了解到主题建模不仅可以用于发现主题，还可以用于快速创建文档语料库的摘要。您将学会选择正确的实体粒度来计算主题模型的重要性。您已经通过许多参数实验找到了最佳的主题模型。您可以通过数量方法和数据来评判生成的主题模型的质量。

# 我们的数据集：联合国大会辩论

我们的用例是语义分析联合国大会辩论语料库。您可能从早期关于文本统计的章节了解过这个数据集。

这一次，我们更感兴趣的是演讲的含义和语义内容，以及我们如何将它们按主题排列。我们想知道演讲者在谈论什么，并回答这样的问题：文档语料库中是否有结构？有哪些主题？哪一个最突出？这种情况随时间而变化吗？

## 检查语料库的统计数据

在开始主题建模之前，检查底层文本语料库的统计数据总是一个好主意。根据此分析的结果，您通常会选择分析不同的实体，例如文档、部分文本或段落。

我们对作者和其他信息不是很感兴趣，因此只需处理提供的一个 *CSV* 文件即可：

```py
import pandas as pd
debates = pd.read_csv("un-general-debates.csv")
debates.info()

```

`输出：`

```py
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 7507 entries, 0 to 7506
Data columns (total 4 columns):
session    7507 non-null int64
year       7507 non-null int64
country    7507 non-null object
text       7507 non-null object
dtypes: int64(2), object(2)
memory usage: 234.7+ KB

```

结果看起来不错。文本列中没有空值；我们可能稍后会使用年份和国家，它们也只有非空值。

演讲非常长，涵盖了许多主题，因为每个国家每年只能发表一次演讲。演讲的不同部分几乎总是由段落分隔。不幸的是，数据集存在一些格式问题。比较两篇选定演讲的文本：

```py
print(repr(df.iloc[2666]["text"][0:200]))
print(repr(df.iloc[4729]["text"][0:200]))

```

`输出：`

```py
'\ufeffIt is indeed a pleasure for me and the members of my delegation to
extend to Ambassador Garba our sincere congratulations on his election to the
presidency of the forty-fourth session of the General '
'\ufeffOn behalf of the State of Kuwait, it\ngives me pleasure to congratulate
Mr. Han Seung-soo,\nand his friendly country, the Republic of Korea, on
his\nelection as President of the fifty-sixth session of t'

```

正如您所见，在一些演讲中，换行符用于分隔段落。在其他演讲的转录中，换行符用于分隔行。因此，为了恢复段落，我们不能只是在换行符处拆分。事实证明，在行尾出现的句号、感叹号或问号处拆分效果也很好。我们忽略停止后的空格：

```py
import re
df["paragraphs"] = df["text"].map(lambda text: re.split('[.?!]\s*\n', text))
df["number_of_paragraphs"] = df["paragraphs"].map(len)

```

根据 [第2章](ch02.xhtml#ch-api) 中的分析，我们已经知道每年的演讲数量变化不大。段落数量也是这样吗？

```py
%matplotlib inline
debates.groupby('year').agg({'number_of_paragraphs': 'mean'}).plot.bar()

```

`输出：`

![](Images/btap_08in01.jpg)

段落平均数量随时间显著减少。我们本应该预期，随着每年演讲者人数的增加和演讲总时间的限制。

除此之外，统计分析显示数据集没有系统性问题。语料库仍然很新；任何一年都没有缺失数据。我们现在可以安全地开始揭示潜在结构并检测主题。

## 准备工作

主题建模是一种机器学习方法，需要矢量化数据。所有主题建模方法都从文档-术语矩阵开始。回顾这个矩阵的含义（它在 [第4章](ch04.xhtml#ch-preparation) 中介绍过），其元素是对应文档（行）中单词（列）的词频（或经常作为 TF-IDF 权重进行缩放）。该矩阵是稀疏的，因为大多数文档只包含词汇的一小部分。

让我们计算演讲和演讲段落的 TF-IDF 矩阵。首先，我们需要从 scikit-learn 中导入必要的包。我们从一个简单的方法开始，使用标准的 spaCy 停用词：

```py
from sklearn.feature_extraction.text import TfidfVectorizer
from spacy.lang.en.stop_words import STOP_WORDS as stopwords

```

计算演讲的文档-术语矩阵很容易；我们还包括二元组：

```py
tfidf_text = TfidfVectorizer(stop_words=stopwords, min_df=5, max_df=0.7)
vectors_text = tfidf_text.fit_transform(debates['text'])
vectors_text.shape

```

`输出：`

```py
(7507, 24611)

```

对于段落来说，稍微复杂一些，因为我们首先必须展平列表。在同一步骤中，我们省略空段落：

```py
# flatten the paragraphs keeping the years
paragraph_df = pd.DataFrame([{ "text": paragraph, "year": year } 
                               for paragraphs, year in \
                               zip(df["paragraphs"], df["year"]) 
                                    for paragraph in paragraphs if paragraph])

tfidf_para_vectorizer = TfidfVectorizer(stop_words=stopwords, min_df=5,
                                        max_df=0.7)
tfidf_para_vectors = tfidf_para_vectorizer.fit_transform(paragraph_df["text"])
tfidf_para_vectors.shape

```

`输出：`

```py
(282210, 25165)

```

当然，段落矩阵的行数要多得多。列数（单词数）也不同，因为 `min_df` 和 `max_df` 在选择特征时有影响，文档的数量也已经改变。

# 非负矩阵分解（NMF）

在文档语料库中找到潜在结构的概念上最简单的方法是对文档-术语矩阵进行因子分解。幸运的是，文档-术语矩阵只有正值元素；因此，我们可以使用线性代数中允许我们表示[矩阵为两个其他非负矩阵的乘积的方法](https://oreil.ly/JVpFA)。按照惯例，原始矩阵称为 *V*，而因子是 *W* 和 *H*：

<math alttext="normal upper V almost-equals normal upper W dot normal upper H"><mrow><mi mathvariant="normal">V</mi> <mo>≈</mo> <mi mathvariant="normal">W</mi> <mo>·</mo> <mi mathvariant="normal">H</mi></mrow></math>

或者我们可以以图形方式表示它（可视化进行矩阵乘法所需的维度），如[图 8-1](#nmf-decomposition)所示。

根据维度的不同，可以执行精确的因子分解。但由于这样做计算成本更高，近似因子分解已经足够。

![](Images/btap_0801.jpg)

###### 图 8-1\. 概要的非负矩阵分解；原始矩阵 V 被分解为 W 和 H。

在文本分析的背景下，*W* 和 *H* 都有一个解释。矩阵 *W* 的行数与文档-术语矩阵相同，因此将文档映射到主题（文档-主题矩阵）。*H* 的列数与特征数相同，因此显示了主题由特征构成的方式（主题-特征矩阵）。主题的数量（*W* 的列数和 *H* 的行数）可以任意选择。这个数字越小，因子分解的精确度就越低。

## 蓝图：使用 NMF 创建文档的主题模型

在 scikit-learn 中为演讲执行此分解真的很容易。由于（几乎）所有主题模型都需要主题数量作为参数，我们任意选择了 10 个主题（后来证明这是一个很好的选择）：

```py
from sklearn.decomposition import NMF

nmf_text_model = NMF(n_components=10, random_state=42)
W_text_matrix = nmf_text_model.fit_transform(tfidf_text_vectors)
H_text_matrix = nmf_text_model.components_

```

与 `TfidfVectorizer` 类似，NMF 也有一个 `fit_transform` 方法，返回其中一个正因子矩阵。可以通过 NMF 类的 `components_` 成员变量访问另一个因子。

主题是单词分布。我们现在将分析这个分布，看看我们是否可以找到主题的解释。看看[图 8-1](#nmf-decomposition)，我们需要考虑 *H* 矩阵，并找到每行（主题）中最大值的索引，然后将其用作词汇表中的查找索引。因为这对所有主题模型都有帮助，我们定义一个输出摘要的函数：

```py
def display_topics(model, features, no_top_words=5):
    for topic, word_vector in enumerate(model.components_):
        total = word_vector.sum()
        largest = word_vector.argsort()[::-1] # invert sort order
        print("\nTopic %02d" % topic)
        for i in range(0, no_top_words):
            print(" %s (%2.2f)" % (features[largest[i]],
                  word_vector[largest[i]]*100.0/total))

```

调用此函数，我们可以得到 NMF 在演讲中检测到的主题的良好总结（数字是单词对各自主题的百分比贡献）：

```py
display_topics(nmf_text_model, tfidf_text_vectorizer.get_feature_names())
```

`输出：`

| **主题 00** co (0.79)

操作 (0.65)

裁军 (0.36)

核 (0.34)

关系 (0.25) | **主题 01** 恐怖主义 (0.38)

挑战 (0.32)

可持续 (0.30)

千年 (0.29)

改革 (0.28) | **主题 02** 非洲 (1.15)

非洲 (0.82)

南 (0.63)

纳米比亚 (0.36)

代表团 (0.30) | **主题 03** 阿拉伯 (1.02)

以色列 (0.89)

巴勒斯坦的 (0.60)

黎巴嫩 (0.54)

以色列的 (0.54) | **主题 04** 美国的 (0.33)

美国 (0.31)

拉丁 (0.31)

巴拿马 (0.21)

玻利维亚 (0.21) |

| **主题 05** 太平洋 (1.55)

岛屿 (1.23)

索罗门 (0.86)

岛屿 (0.82)

斐济 (0.71) | **主题 06** 苏联 (0.81)

共和国 (0.78)

核 (0.68)

越南 (0.64)

社会主义 (0.63) | **主题 07** 几内亚 (4.26)

赤道 (1.75)

比绍 (1.53)

巴布亚 (1.47)

共和国 (0.57) | **主题 08** 欧洲 (0.61)

欧洲 (0.44)

合作 (0.39)

波斯尼亚 (0.34)

赫尔采哥维纳 (0.30) | **主题 09** 加勒比 (0.98)

小 (0.66)

巴哈马 (0.63)

圣 (0.63)

巴巴多斯 (0.61) |

主题 00 和 主题 01 看起来非常有前景，因为人们正在讨论核裁军和恐怖主义。这些确实是联合国大会辩论中的真实主题。

然而，后续主题或多或少集中在世界不同地区。这是因为演讲者主要提到自己的国家和邻国。这在主题 03 中特别明显，反映了中东的冲突。

查看单词在主题中的贡献百分比也很有趣。由于单词数量众多，单个贡献相当小，除了主题 07 中的*几内亚*。正如我们后面将看到的，单词在主题内的百分比是主题模型质量的一个很好的指标。如果主题内的百分比迅速下降，则表明该主题定义良好，而缓慢下降的单词概率表明主题不太明显。直觉上找出主题分离得有多好要困难得多；我们稍后将进行审视。

发现“大”主题有多大将会很有趣，即每个主题主要可以分配给多少篇文档。可以通过文档-主题矩阵计算，并对所有文档中的各个主题贡献求和来计算这一点。将它们与总和归一化，并乘以100给出一个百分比值：

```py
W_text_matrix.sum(axis=0)/W_text_matrix.sum()*100.0
```

`输出:`

```py
array([11.13926287, 17.07197914, 13.64509781, 10.18184685, 11.43081404,
        5.94072639,  7.89602474,  4.17282682, 11.83871081,  6.68271054])

```

我们可以清楚地看到，有较小和较大的主题，但基本上没有离群值。具有均匀分布是质量指标。例如，如果你的主题模型中有一两个大主题与其他所有主题相比，你可能需要调整主题数量。

在接下来的部分，我们将使用演讲段落作为主题建模的实体，并尝试找出是否改进了主题。

## 蓝图：使用NMF创建段落的主题模型

在联合国的一般辩论中，以及许多其他文本中，不同的主题通常会混合，这使得主题建模算法难以找到个别演讲的共同主题。特别是在较长的文本中，文档往往涵盖多个而不仅仅是一个主题。我们如何处理这种情况？一种想法是在文档中找到更具主题一致性的较小实体。

在我们的语料库中，段落是演讲的自然分割，我们可以假设演讲者在一个段落内试图坚持一个主题。在许多文档中，段落是一个很好的候选对象（如果可以识别），我们已经准备好了相应的TF-IDF向量。让我们尝试计算它们的主题模型：

```py
nmf_para_model = NMF(n_components=10, random_state=42)
W_para_matrix = nmf_para_model.fit_transform(tfidf_para_vectors)
H_para_matrix = nmf_para_model.components_

```

我们之前开发的`display_topics`函数可以用来找到主题的内容：

```py
display_topics(nmf_para_model, tfidf_para_vectorizer.get_feature_names())
```

`Out:`

| **主题 00** 国家 (5.63)

united (5.52)

组织 (1.27)

州 (1.03)

宪章 (0.93) | **主题 01** 总的 (2.87)

会议 (2.83)

大会 (2.81)

先生 (1.98)

主席 (1.81) | **主题 02** 国家 (4.44)

发展 (2.49)

经济 (1.49)

发展 (1.35)

贸易 (0.92) | **主题 03** 人民 (1.36)

和平 (1.34)

东 (1.28)

中 (1.17)

巴勒斯坦 (1.14) | **主题 04** 核 (4.93)

武器 (3.27)

裁军 (2.01)

条约 (1.70)

扩散 (1.46) |

| **主题 05** 权利 (6.49)

人类 (6.18)

尊重 (1.15)

基础 (0.86)

全球 (0.82) | **主题 06** 非洲 (3.83)

南 (3.32)

非洲 (1.70)

纳米比亚 (1.38)

种族隔离 (1.19) | **主题 07** 安全 (6.13)

理事会 (5.88)

永久 (1.50)

改革 (1.48)

和平 (1.30) | **主题 08** 国际 (2.05)

世界 (1.50)

共同体 (0.92)

新 (0.77)

和平 (0.67) | **主题 09** 发展 (4.47)

可持续 (1.18)

经济 (1.07)

社会 (1.00)

目标 (0.93) |

与以前用于演讲主题建模的结果相比，我们几乎失去了所有国家或地区，除了南非和中东地区。这些都是由于引发了世界其他地区兴趣的地区冲突。段落中的主题如“人权”，“国际关系”，“发展中国家”，“核武器”，“安理会”，“世界和平”和“可持续发展”（最后一个可能只是最近才出现）与演讲的主题相比显得更加合理。观察单词的百分比值，我们可以看到它们下降得更快，主题更加显著。

# 潜在语义分析/索引

另一种执行主题建模的算法是基于所谓的奇异值分解（SVD），这是线性代数中的另一种方法。

从图形上看，我们可以将奇异值分解（SVD）视为以一种方式重新排列文档和单词，以揭示文档-词矩阵中的块结构。在[topicmodels.info](https://oreil.ly/yJnWL)有一个这个过程的良好可视化。[图8-2](#svd-animation)显示了文档-词矩阵的开始和最终的块对角形式。

利用主轴定理，正交 *n* × *n* 矩阵有一个特征值分解。不幸的是，我们没有正交的方形文档-词矩阵（除了少数情况）。因此，我们需要一种称为*奇异值分解*的泛化。在其最一般的形式中，该定理表明任何 *m* × *n* 矩阵**V **都可以分解如下：

<math alttext="normal upper V equals normal upper U dot normal upper Sigma dot normal upper V Superscript asterisk"><mrow><mi mathvariant="normal">V</mi> <mo>=</mo> <mi mathvariant="normal">U</mi> <mo>·</mo> <mi>Σ</mi> <mo>·</mo> <msup><mi mathvariant="normal">V</mi> <mo>*</mo></msup></mrow></math>

![](Images/btap_0802.jpg)

###### 图 8-2\. 使用 SVD 进行主题建模的可视化。

*U* 是一个单位 *m* × *m* 矩阵，*V** 是一个 *n* × *n* 矩阵，*Σ* 是一个 *m* × *n* 对角矩阵，其中包含奇异值。对于这个方程，有确切的解，但是由于它们需要大量的时间和计算工作来找到，所以我们正在寻找可以快速找到的近似解。这个近似方法仅考虑最大的奇异值。这导致*Σ*成为一个 *t* × *t* 矩阵；相应地，*U*有 *m* × *t* 和 *V** 有 *t* × *n* 的维度。从图形上看，这类似于非负矩阵分解，如[图 8-3](#svd-decomposition)所示。

![](Images/btap_0803.jpg)

###### 图 8-3\. 示意奇异值分解。

奇异值是*Σ*的对角元素。文档-主题关系包含在*U*中，而词-主题映射由*V**表示。注意，*U*的元素和*V**的元素都不能保证是正的。贡献的相对大小仍然是可解释的，但概率解释不再有效。

## 蓝图：使用 SVD 为段落创建主题模型

在 scikit-learn 中，SVD 的接口与 NMF 的接口相同。这次我们直接从段落开始：

```py
from sklearn.decomposition import TruncatedSVD

svd_para_model = TruncatedSVD(n_components = 10, random_state=42)
W_svd_para_matrix = svd_para_model.fit_transform(tfidf_para_vectors)
H_svd_para_matrix = svd_para_model.components_

```

我们之前定义的用于评估主题模型的函数也可以使用：

```py
display_topics(svd_para_model, tfidf_para_vectorizer.get_feature_names())
```

`输出：`

| **主题 00** 国家（0.67）

联合（0.65）

国际（0.58）

和平（0.46）

世界（0.46） | **主题 01** 一般（14.04）

装配（13.09）

会话（12.94）

先生（10.02）

总统（8.59） | **主题 02** 国家（19.15）

发展（14.61）

经济（13.91）

发展中（13.00）

会议（10.29） | **主题 03** 国家（4.41）

联合（4.06）

发展（0.95）

组织（0.84）

宪章（0.80） | **主题 04** 核（21.13）

武器（14.01）

裁军（9.02）

条约（7.23）

扩散（6.31） |

| **主题 05** 权利（29.50）

人类（28.81）

核（9.20）

武器（6.42）

尊重（4.98） | **主题 06** 非洲（8.73）

南方（8.24）

联合（3.91）

非洲（3.71）

国家（3.41） | **主题 07** 理事会（14.96）

安全（13.38）

非洲（8.50）

南方（6.11）

非洲（3.94） | **主题 08** 世界（48.49）

国际（41.03）

和平（32.98）

社区（23.27）

非洲（22.00） | **主题 09** 发展（63.98）

可持续（20.78）

和平（20.74）

目标（15.92）

非洲（15.61） |

大多数生成的主题与非负矩阵分解的主题非常相似。然而，中东冲突这一主题这次没有单独出现。由于主题-词映射也可能具有负值，因此归一化从主题到主题有所不同。只有构成主题的单词的相对大小才是相关的。

不用担心负百分比。这是因为SVD不保证W中的值为正，因此个别单词的贡献可能为负。这意味着出现在文档中的单词“排斥”相应的主题。

如果我们想确定主题的大小，现在就要查看分解的奇异值：

```py
svd_para.singular_values_

```

`Out:`

```py
array([68.21400653, 39.20120165, 36.36831431, 33.44682727, 31.76183677,
       30.59557993, 29.14061799, 27.40264054, 26.85684195, 25.90408013])

```

主题的大小与NMF方法的段落相当相符。

NMF和SVF都使用了文档-词矩阵（应用了TF-IDF转换）作为主题分解的基础。此外，*U*矩阵的维度与*W*的维度相同；*V*和*H*也是如此。因此，这两种方法产生类似且可比较的结果并不奇怪。由于这些方法计算速度很快，因此我们建议在实际项目中首先使用线性代数方法。

现在我们将摆脱这些基于线性代数的方法，专注于概率主题模型，在过去20年中已经变得极为流行。

# 潜在狄利克雷分配

LDA可以说是当今使用最广泛的主题建模方法。它在过去15年间变得流行，并且可以灵活地适应不同的使用场景。

它是如何工作的？

LDA将每个文档视为包含不同主题。换句话说，每个文档是不同主题的混合。同样，主题是从词中混合而来。为了保持每个文档中主题数量的少而且只包含一些重要词语，LDA最初使用[狄利克雷分布](https://oreil.ly/Kkd9k)，即所谓的*狄利克雷先验*。这一分布用于为文档分配主题和为主题找到单词。狄利克雷分布确保文档只有少量主题，并且主题主要由少量单词定义。假设LDA生成了像之前那样的主题分布，一个主题可能由诸如*核*、*条约*和*裁军*等词汇构成，而另一个主题则由*可持续*、*发展*等词汇组成。

在初始分配之后，生成过程开始。它使用主题和单词的狄利克雷分布，并尝试用随机抽样重新创建原始文档中的单词。这个过程必须多次迭代，因此计算量很大。^([2](ch08.xhtml#idm45634187759704)) 另一方面，结果可以用来为任何确定的主题生成文档。

## 蓝图：使用LDA为段落创建主题模型

Scikit-learn隐藏了所有这些差异，并使用与其他主题建模方法相同的API：

```py
from sklearn.feature_extraction.text import CountVectorizer

count_para_vectorizer = CountVectorizer(stop_words=stopwords, min_df=5,
                        max_df=0.7)
count_para_vectors = count_para_vectorizer.fit_transform(paragraph_df["text"])

```

```py
from sklearn.decomposition import LatentDirichletAllocation

lda_para_model = LatentDirichletAllocation(n_components = 10, random_state=42)
W_lda_para_matrix = lda_para_model.fit_transform(count_para_vectors)
H_lda_para_matrix = lda_para_model.components_

```

# 等待时间

由于概率抽样的原因，该过程比NMF和SVD需要更长时间。期望至少分钟，甚至小时的运行时间。

我们的效用函数可以再次用于可视化段落语料库的潜在主题：

```py
display_topics(lda_para_model, tfidf_para.get_feature_names())

```

`Out:`

| **主题 00** 非洲（2.38）

人们（1.86）

南方（1.57）

纳米比亚（0.88）

政权（0.75）| **主题 01** 共和国（1.52）

政府（1.39）

联合（1.21）

和平（1.16）

人民（1.02）| **主题 02** 普通（4.22）

大会（3.63）

会议（3.38）

总统（2.33）

先生（2.32）| **主题 03** 人类（3.62）

权利（3.48）

国际（1.83）

法律（1.01）

恐怖主义（0.99）| **主题 04** 世界（2.22）

人们（1.14）

国家（0.94）

年（0.88）

今天（0.66）|

| **主题 05** 和平（1.76）

安全（1.63）

东方（1.34）

中间（1.34）

以色列（1.24）| **主题 06** 国家（3.19）

发展（2.70）

经济（2.22）

发展（1.61）

国际（1.45）| **主题 07** 核（3.14）

武器（2.32）

裁军（1.82）

国家（1.47）

军备（1.46）| **主题 08** 国家（5.50）

联合（5.11）

国际（1.46）

安全（1.45）

组织（1.44）| **主题 09** 国际（1.96）

世界（1.91）

和平（1.60）

经济（1.00）

关系（0.99）|

有趣的是观察到，与前述的线性代数方法相比，LDA生成了完全不同的主题结构。*人们*是三个完全不同主题中最突出的词。在主题 04 中，南非与以色列和巴勒斯坦有关联，而在主题 00 中，塞浦路斯、阿富汗和伊拉克有关联。这不容易解释。这也反映在主题的逐渐减少的单词权重中。

其他主题更容易理解，比如气候变化、核武器、选举、发展中国家和组织问题。

在这个例子中，LDA 的结果并不比 NMF 或 SVD 好多少。然而，由于抽样过程，LDA 并不仅限于样本主题仅仅由单词组成。还有几种变体，比如作者-主题模型，也可以抽样分类特征。此外，由于在LDA领域有很多研究，其他想法也经常被发表，这些想法大大超出了文本分析的焦点（例如，见Minghui Qiu等人的 [“不仅仅是我们说了什么，而是我们如何说它们：基于LDA的行为-主题模型”](https://oreil.ly/dnqq5) 或Rahji Abdurehman的 [“关键词辅助LDA：探索监督主题建模的新方法”](https://oreil.ly/DDClf)）。

## 蓝图：可视化LDA结果

由于LDA非常流行，Python中有一个很好的包来可视化LDA结果，称为pyLDAvis。^([3](ch08.xhtml#idm45634187547192)) 幸运的是，它可以直接使用sciki-learn的结果进行可视化。

注意，这需要一些时间：

```py
import pyLDAvis.sklearn

lda_display = pyLDAvis.sklearn.prepare(lda_para_model, count_para_vectors,
                            count_para_vectorizer, sort_topics=False)
pyLDAvis.display(lda_display)

```

`Out:`

![](Images/btap_08in02.jpg)

可视化中提供了大量信息。让我们从“气泡”话题开始，并点击它。现在看一下红色条，它们象征着当前选定话题中的单词分布。由于条的长度没有迅速减少，说明话题 2 并不十分显著。这与我们在 [“Blueprint: Creating a Topic Model for Paragraphs with LDA”](#ch08-topic-model-para) 表格中看到的效果相同（看看话题 1，在那里我们使用了数组索引，而 pyLDAvis 从 1 开始枚举话题）。

为了可视化结果，话题从原始维度（单词数）通过主成分分析（PCA）映射到二维空间，这是一种标准的降维方法。这导致了一个点；圆圈被添加以查看话题的相对大小。可以通过在准备阶段传递 `mds="tsne"` 参数来使用 T-SNE 替代 PCA。这改变了话题之间的距离映射，并显示了较少重叠的话题气泡。然而，这只是在可视化时将许多单词维度投影到仅两个维度的一个副作用。因此，查看话题的单词分布并不完全依赖于低维度的可视化是一个好主意。

看到话题 4、6 和 10（“国际”）之间的强重叠是很有趣的，而话题 3（“大会”）似乎远离其他所有话题。通过悬停在其他话题气泡上或点击它们，您可以查看右侧的单词分布。尽管不是所有话题都完全分离，但有些话题（如话题 1 和话题 7）远离其他话题。尝试悬停在它们上面，您会发现它们的单词内容也不同。对于这样的话题，提取最具代表性的文档并将它们用作监督学习的训练集可能是有用的。

pyLDAvis 是一个很好的工具，适合在演示文稿中使用截图。尽管看起来探索性十足，但真正的探索在于修改算法的特征和超参数。

使用 pyLDAvis 能让我们很好地了解话题是如何相互排列的，以及哪些单词是重要的。然而，如果我们需要更质量的话题理解，可以使用额外的可视化工具。

# Blueprint: 使用词云来显示和比较话题模型

到目前为止，我们已经使用列表显示了话题模型。这样，我们可以很好地识别不同话题的显著程度。然而，在许多情况下，话题模型用于给出关于语料库有效性和更好可视化的第一印象。正如我们在 [第 1 章](ch01.xhtml#ch-exploration) 中看到的，词云是展示这一点的定性和直观工具。

我们可以直接使用词云来展示我们的主题模型。代码可以很容易地从之前定义的`display_topics`函数中推导出来：

```py
import matplotlib.pyplot as plt
from wordcloud import WordCloud

def wordcloud_topics(model, features, no_top_words=40):
    for topic, words in enumerate(model.components_):
        size = {}
        largest = words.argsort()[::-1] # invert sort order
        for i in range(0, no_top_words):
            size[features[largest[i]]] = abs(words[largest[i]])
        wc = WordCloud(background_color="white", max_words=100,
                       width=960, height=540)
        wc.generate_from_frequencies(size)
        plt.figure(figsize=(12,12))
        plt.imshow(wc, interpolation='bilinear')
        plt.axis("off")
        # if you don't want to save the topic model, comment the next line
        plt.savefig(f'topic{topic}.png')

```

通过使用此代码，我们可以定性地比较NMF模型（[图 8-4](#fig-wordcloud-nmf)）的结果与LDA模型（[图 8-5](#fig-wordcloud-lda)）。较大的单词在各自的主题中更为重要。如果许多单词的大小大致相同，则该主题没有明显表现：

```py
wordcloud_topics(nmf_para_model, tfidf_para_vectorizer.get_feature_names())
wordcloud_topics(lda_para_model, count_para_vectorizer.get_feature_names())

```

# 使用单独的缩放制作词云

词云中的字体大小在每个主题内部使用缩放，因此在绘制任何最终结论之前，验证实际数字非常重要。

现在的展示更加引人入胜。很容易在两种方法之间匹配主题，比如0-NMF与8-LDA。对于大多数主题来说，这是显而易见的，但也存在差异。1-LDA（“人民共和国”）在NMF中没有相对应项，而9-NMF（“可持续发展”）在LDA中找不到。

由于我们找到了主题的良好定性可视化，我们现在对主题分布随时间的变化感兴趣。

![](Images/btap_0804.jpg)

###### 图 8-4。展示NMF主题模型的词云。

![](Images/btap_0805.jpg)

###### 图 8-5。展示LDA主题模型的词云。

# 蓝图：计算文档主题分布和时间演变

正如您在本章开头的分析中所看到的，演讲的元数据随时间变化。这引发了一个有趣的问题，即主题的分布随时间如何变化。结果表明，这很容易计算并且具有洞察力。

像scikit-learn的向量化器一样，主题模型也有一个`transform`方法，用于计算现有文档的主题分布，保持已拟合的主题模型不变。让我们首先使用这个方法将1990年之前和之后的演讲分开。为此，我们为1990年之前和之后的文档创建NumPy数组：

```py
import numpy as np
before_1990 = np.array(paragraph_df["year"] < 1990)
after_1990 = ~ before_1990

```

然后，我们可以计算相应的*W*矩阵：

```py
W_para_matrix_early = nmf_para_model.transform(tfidf_para_vectors[before_1990])
W_para_matrix_late  = nmf_para_model.transform(tfidf_para_vectors[after_1990])
print(W_para_matrix_early.sum(axis=0)/W_para_matrix_early.sum()*100.0)
print(W_para_matrix_late.sum(axis=0)/W_para_matrix_late.sum()*100.0)
```

`Out:`

```py
['9.34', '10.43', '12.18', '12.18', '7.82', '6.05', '12.10', '5.85', '17.36',
 '6.69']
['7.48', '8.34', '9.75', '9.75', '6.26', '4.84', '9.68', '4.68', '13.90',
 '5.36']

```

结果非常有趣，某些百分比发生了显著变化；特别是后期年份中倒数第二个主题的大小要小得多。现在，我们将尝试更深入地研究主题及其随时间的变化。

让我们尝试计算各个年份的分布，看看是否能找到可视化方法来揭示可能的模式：

```py
year_data = []
years = np.unique(paragraph_years)
for year in tqdm(years):
    W_year = nmf_para_model.transform(tfidf_para_vectors[paragraph_years \
                                      == year])
    year_data.append([year] + list(W_year.sum(axis=0)/W_year.sum()*100.0))

```

为了使图表更直观，我们首先创建一个包含两个最重要单词的主题列表：

```py
topic_names = []
voc = tfidf_para_vectorizer.get_feature_names()
for topic in nmf_para_model.components_:
    important = topic.argsort()
    top_word = voc[important[-1]] + " " + voc[important[-2]]
    topic_names.append("Topic " + top_word)

```

然后，我们将结果与以前的主题作为列名合并到一个`DataFrame`中，这样我们可以轻松地进行可视化，如下所示：

```py
df_year = pd.DataFrame(year_data,
               columns=["year"] + topic_names).set_index("year")
df_year.plot.area()

```

`Out:`

![](Images/btap_08in03.jpg)

在生成的图表中，您可以看到主题分布随着年份的变化而变化。我们可以看到，“可持续发展”主题在持续增加，而“南非”在种族隔离制度结束后失去了流行度。

相比于展示单个（猜测的）单词的时间发展，主题似乎更自然，因为它们源于文本语料库本身。请注意，此图表是通过一种纯无监督的方法生成的，因此其中没有偏见。一切都已经在辩论数据中；我们只是揭示了它。

到目前为止，我们在主题建模中仅使用了 scikit-learn。在 Python 生态系统中，有一个专门用于主题模型的库称为 Gensim，我们现在将对其进行调查。

# 使用 Gensim 进行主题建模

除了 scikit-learn，[*Gensim*](https://oreil.ly/Ybn63) 是另一个在 Python 中执行主题建模的流行工具。与 scikit-learn 相比，它提供了更多用于计算主题模型的算法，并且还可以给出关于模型质量的估计。

## 蓝图：为 Gensim 准备数据

在我们开始计算 Gensim 模型之前，我们必须准备数据。不幸的是，API 和术语与 scikit-learn 不同。在第一步中，我们必须准备词汇表。Gensim 没有集成的分词器，而是期望每篇文档语料库的每一行已经被分词了：

```py
# create tokenized documents
gensim_paragraphs = [[w for w in re.findall(r'\b\w\w+\b' , paragraph.lower())
                          if w not in stopwords]
                             for paragraph in paragraph_df["text"]]
```

分词后，我们可以用这些分词后的文档初始化 Gensim 字典。将字典视为从单词到列的映射（就像我们在 [第二章](ch02.xhtml#ch-api) 中使用的特征）：

```py
from gensim.corpora import Dictionary
dict_gensim_para = Dictionary(gensim_paragraphs)

```

与 scikit-learn 的 `TfidfVectorizer` 类似，我们可以通过过滤出现频率不够高或者太高的单词来减少词汇量。为了保持低维度，我们选择单词至少出现在五篇文档中，但不能超过文档的 70%。正如我们在 [第二章](ch02.xhtml#ch-api) 中看到的，这些参数可以进行优化，并需要一些实验。

在 Gensim 中，这通过参数 `no_below` 和 `no_above` 过滤器实现（在 scikit-learn 中，类似的是 `min_df` 和 `max_df`）：

```py
dict_gensim_para.filter_extremes(no_below=5, no_above=0.7)

```

读取了字典后，我们现在可以使用 Gensim 计算词袋矩阵（在 Gensim 中称为 *语料库*，但我们将坚持我们当前的术语）：

```py
bow_gensim_para = [dict_gensim_para.doc2bow(paragraph) \
                    for paragraph in gensim_paragraphs]

```

最后，我们可以执行 TF-IDF 转换。第一行适配词袋模型，而第二行转换权重：

```py
from gensim.models import TfidfModel
tfidf_gensim_para = TfidfModel(bow_gensim_para)
vectors_gensim_para = tfidf_gensim_para[bow_gensim_para]

```

`vectors_gensim_para` 矩阵是我们将在 Gensim 中进行所有即将进行的主题建模任务的矩阵。

## 蓝图：使用 Gensim 进行非负矩阵分解

让我们首先检查 NMF 的结果，看看我们是否可以复现 scikit-learn 的结果：

```py
from gensim.models.nmf import Nmf
nmf_gensim_para = Nmf(vectors_gensim_para, num_topics=10,
                      id2word=dict_gensim_para, kappa=0.1, eval_every=5)

```

评估可能需要一些时间。虽然 Gensim 提供了一个 `show_topics` 方法来直接显示主题，但我们有一个不同的实现，使其看起来像 scikit-learn 的结果，这样更容易进行比较：

```py
display_topics_gensim(nmf_gensim_para)

```

`输出：`

| **主题 00** 国家 (0.03)

联合 (0.02)

人类 (0.02)

权利 (0.02)

角色 (0.01) | **主题 01** 非洲 (0.02)

南部 (0.02)

人们 (0.02)

政府 (0.01)

共和国 (0.01) | **主题 02** 经济 (0.01)

发展 (0.01)

国家 (0.01)

社会 (0.01)

国际（0.01）| **主题 03** 国家（0.02）

发展中（0.02）

资源（0.01）

海（0.01）

发达（0.01）| **主题 04** 以色列（0.02）

阿拉伯（0.02）

巴勒斯坦（0.02）

理事会（0.01）

安全（0.01）|

| **主题 05** 组织（0.02）

宪章（0.02）

原则（0.02）

成员（0.01）

尊重（0.01）| **主题 06** 问题（0.01）

解决方案（0.01）

东部（0.01）

情况（0.01）

问题（0.01）| **主题 07** 核（0.02）

公司（0.02）

操作（0.02）

裁军（0.02）

武器（0.02）| **主题 08** 会议（0.02）

将军（0.02）

大会（0.02）

先生（0.02）

总统（0.02）| **主题 09** 世界（0.02）

和平（0.02）

人民（0.02）

安全（0.01）

国家（0.01）|

NMF也是一种统计方法，因此结果不应与我们用scikit-learn计算的结果完全相同，但它们非常相似。Gensim有用于计算主题模型的一致性评分的代码，作为质量指标。让我们试试这个：

```py
from gensim.models.coherencemodel import CoherenceModel

nmf_gensim_para_coherence = CoherenceModel(model=nmf_gensim_para,
                                           texts=gensim_paragraphs,
                                           dictionary=dict_gensim_para,
                                           coherence='c_v')
nmf_gensim_para_coherence_score = nmf_gensim_para_coherence.get_coherence()
print(nmf_gensim_para_coherence_score)

```

`Out:`

```py
0.6500661701098243

```

分数随主题数量变化。如果想找到最佳主题数，常见的方法是运行多个不同值的NMF，计算一致性评分，然后选择最大化评分的主题数。

让我们尝试用LDA做同样的事情并比较质量指标。

## 蓝图：使用Gensim的LDA

使用Gensim运行LDA与准备好的数据一样简单如使用NMF。 `LdaModel` 类有许多用于调整模型的参数；我们在这里使用推荐的数值：

```py
from gensim.models import LdaModel
lda_gensim_para = LdaModel(corpus=bow_gensim_para, id2word=dict_gensim_para,
    chunksize=2000, alpha='auto', eta='auto', iterations=400, num_topics=10, 
    passes=20, eval_every=None, random_state=42)
```

我们对主题的词分布很感兴趣：

```py
display_topics_gensim(lda_gensim_para)

```

`Out:`

| **主题 00** 气候（0.12）

公约（0.03）

太平洋（0.02）

环境（0.02）

海（0.02）| **主题 01** 国家（0.05）

人民（0.05）

政府（0.03）

国家（0.02）

支持（0.02）| **主题 02** 国家（0.10）

联合（0.10）

人类（0.04）

安全（0.03）

权利（0.03）| **主题 03** 国际（0.03）

社区（0.01）

努力（0.01）

新（0.01）

全球（0.01）| **主题 04** 非洲（0.06）

非洲人（0.06）

大陆（0.02）

恐怖主义者（0.02）

罪行（0.02）|

| **主题 05** 世界（0.05）

年份（0.02）

今天（0.02）

和平（0.01）

时间（0.01）| **主题 06** 和平（0.03）

冲突（0.02）

区域（0.02）

人民（0.02）

国家（0.02）| **主题 07** 南（0.10）

苏丹（0.05）

中国（0.04）

亚洲（0.04）

索马里（0.04）| **主题 08** 将军（0.10）

大会（0.09）

会议（0.05）

总统（0.04）

秘书（0.04）| **主题 09** 发展（0.07）

国家（0.05）

经济（0.03）

可持续（0.02）

2015年（0.02）|

主题的解释并不像NMF生成的解释那样容易。如前所示检查一致性评分，我们发现较低的评分为0.45270703180962374。Gensim还允许我们计算LDA模型的困惑度评分。困惑度衡量概率模型预测样本的能力。当我们执行 `lda_gensim_para.log_perplexity(vectors_gensim_para)` 时，我们得到一个困惑度评分为 -9.70558947109483。

## 蓝图：计算一致性评分

Gensim还可以计算主题一致性。方法本身是一个包含分割、概率估计、确认度量计算和聚合的四阶段过程。幸运的是，Gensim有一个`CoherenceModel`类，封装了所有这些单一任务，我们可以直接使用它：

```py
from gensim.models.coherencemodel import CoherenceModel

lda_gensim_para_coherence = CoherenceModel(model=lda_gensim_para,
    texts=gensim_paragraphs, dictionary=dict_gensim_para, coherence='c_v')
lda_gensim_para_coherence_score = lda_gensim_para_coherence.get_coherence()
print(lda_gensim_para_coherence_score)

```

`Out:`

```py
0.5444930496493174

```

用`nmf`替换`lda`，我们可以为我们的NMF模型计算相同的得分：

```py
nmf_gensim_para_coherence = CoherenceModel(model=nmf_gensim_para,
    texts=gensim_paragraphs, dictionary=dict_gensim_para, coherence='c_v')
nmf_gensim_para_coherence_score = nmf_gensim_para_coherence.get_coherence()
print(nmf_gensim_para_coherence_score)

```

`Out:`

```py
0.6505110480127619

```

分数要高得多，这意味着与LDA相比，NMF模型更接近真实主题。

计算LDA模型各个主题的一致性得分更加简单，因为它直接由LDA模型支持。让我们首先看一下平均值：

```py
top_topics = lda_gensim_para.top_topics(vectors_gensim_para, topn=5)
avg_topic_coherence = sum([t[1] for t in top_topics]) / len(top_topics)
print('Average topic coherence: %.4f.' % avg_topic_coherence)

```

`Out:`

```py
Average topic coherence: -2.4709.

```

我们还对各个主题的一致性得分感兴趣，这些得分包含在`top_topics`中。但是，输出内容太冗长（检查一下！），因此我们试图通过仅将一致性得分与主题中最重要的单词一起打印来压缩它：

```py
[(t[1], " ".join([w[1] for w in t[0]])) for t in top_topics]

```

`Out:`

```py
[(-1.5361194241843663, 'general assembly session president secretary'),
 (-1.7014902754187737, 'nations united human security rights'),
 (-1.8485895463251694, 'country people government national support'),
 (-1.9729985026779555, 'peace conflict region people state'),
 (-1.9743434414778658, 'world years today peace time'),
 (-2.0202823396586433, 'international community efforts new global'),
 (-2.7269347656599225, 'development countries economic sustainable 2015'),
 (-2.9089975883502706, 'climate convention pacific environmental sea'),
 (-3.8680684770508753, 'africa african continent terrorist crimes'),
 (-4.1515707817343195, 'south sudan china asia somalia')]

```

使用Gensim可以轻松计算主题模型的一致性得分。绝对值很难解释，但是通过变化方法（NMF与LDA）或主题数可以让您了解您希望在主题模型中前进的方向。一致性得分和一致性模型是Gensim的一大优势，因为它们（尚）未包含在scikit-learn中。

由于很难估计“正确”的主题数量，我们现在看一种创建层次模型的方法，不需要固定的主题数作为参数。

## 蓝图：找到最佳主题数

在前面的章节中，我们始终使用了10个主题。到目前为止，我们还没有将此主题模型的质量与具有较少或更多主题数的不同模型进行比较。我们希望找到一种结构化的方式来找到最佳主题数量，而无需深入解释每个主题模型。

原来有一种方法可以实现这一点。主题模型的“质量”可以通过先前引入的一致性得分来衡量。为了找到最佳一致性得分，我们现在将为不同数量的主题使用LDA模型来计算它。我们将尝试找到最高得分，这应该给我们提供最佳的主题数量：

```py
from gensim.models.ldamulticore import LdaMulticore
lda_para_model_n = []
for n in tqdm(range(5, 21)):
    lda_model = LdaMulticore(corpus=bow_gensim_para, id2word=dict_gensim_para,
                             chunksize=2000, eta='auto', iterations=400,
                             num_topics=n, passes=20, eval_every=None,
                             random_state=42)
    lda_coherence = CoherenceModel(model=lda_model, texts=gensim_paragraphs,
                                   dictionary=dict_gensim_para, coherence='c_v')
    lda_para_model_n.append((n, lda_model, lda_coherence.get_coherence()))

```

# 一致性计算需要时间

计算LDA模型（及其一致性）在计算上是昂贵的，因此在现实生活中，最好优化算法，仅计算少量模型和困惑度。有时，如果只计算少量主题的一致性得分，这可能是有意义的。

现在我们可以选择哪个主题数产生良好的一致性得分。注意，通常随着主题数量的增加，得分会增加。选择太多的主题会使解释变得困难：

```py
pd.DataFrame(lda_para_model_n, columns=["n", "model", \
    "coherence"]).set_index("n")[["coherence"]].plot(figsize=(16,9))
```

`Out:`

![](Images/btap_08in04.jpg)

总体而言，图表随主题数量增加而增长，这几乎总是情况。但是，我们可以看到在13和17个主题时出现了“峰值”，因此这些数字看起来是不错的选择。我们将为17个主题的结果进行可视化：

```py
display_topics_gensim(lda_para_model_n[12][1])
```

`Out:`

| **主题 00** 和平 (0.02)

国际 (0.02)

合作 (0.01)

国家 (0.01)

地区 (0.01) | **主题 01** 将军 (0.05)

装配 (0.04)

会议 (0.02)

总统 (0.03)

先生 (0.03) | **主题 02** 联合 (0.04)

国家 (0.04)

国家 (0.03)

欧洲 (0.02)

联盟 (0.02) | **主题 03** 国家 (0.07)

联合 (0.07)

安全 (0.03)

理事会 (0.02)

国际 (0.02) | **主题 04** 发展 (0.03)

将军 (0.02)

会议 (0.02)

装配 (0.02)

可持续 (0.01) | **主题 05** 国际 (0.03)

恐怖主义 (0.03)

国家 (0.01)

伊拉克 (0.01)

行为 (0.01) |

| **主题 06** 和平 (0.03)

东部 (0.02)

中东 (0.02)

以色列 (0.02)

解决方案 (0.01) | **主题 07** 非洲 (0.08)

南方 (0.05)

非洲 (0.05)

纳米比亚 (0.02)

共和国 (0.01) | **主题 08** 国家 (0.04)

小 (0.04)

岛屿 (0.03)

海洋 (0.02)

太平洋 (0.02) | **主题 09** 世界 (0.03)

国际 (0.02)

问题 (0.01)

战争 (0.01)

和平 (0.01) | **主题 10** 人类 (0.07)

权利 (0.06)

法律 (0.02)

尊重 (0.02)

国际 (0.01) | **主题 11** 气候 (0.03)

变革 (0.03)

全球 (0.02)

环境 (0.01)

能源 (0.01) |

| **主题 12** 世界 (0.03)

人们 (0.02)

未来 (0.01)

年度 (0.01)

今天 (0.01) | **主题 13** 人民 (0.03)

独立 (0.02)

人民 (0.02)

斗争 (0.01)

国家 (0.01) | **主题 14** 人民 (0.02)

国家 (0.02)

政府 (0.02)

人道主义 (0.01)

难民 (0.01) | **主题 15** 国家 (0.05)

发展 (0.03)

经济 (0.03)

发展中 (0.02)

贸易 (0.01) | **主题 16** 核 (0.06)

武器 (0.04)

裁军 (0.03)

武器 (0.03)

条约 (0.02) |

大多数主题都很容易解释，但有些主题很难（如0、3、8），因为它们包含许多单词，大小相近，但不完全相同。17个主题的主题模型是否更容易解释？实际上并非如此。连贯性得分更高，但这并不一定意味着更明显的解释。换句话说，如果主题数量过多，仅依赖连贯性得分可能是危险的。尽管理论上，较高的连贯性应有助于更好的可解释性，但通常存在权衡，选择较少的主题可以使生活更轻松。回顾连贯性图表，10似乎是一个不错的选择，因为它是连贯性得分的*局部最大值*。

由于明显很难找到“正确”的主题数量，我们现在将看看一种创建层次模型并且不需要固定主题数量作为参数的方法。

## 蓝图：使用Gensim创建层次狄利克雷过程

退一步，回想一下在 [“Blueprint: Using LDA with Gensim”](#ch08usingldawithgensim) 中关于主题的可视化。主题的大小差异很大，有些主题有较大的重叠。如果结果能先给我们更广泛的主题，然后在其下方列出一些子主题，那将是非常好的。这正是层次狄利克雷过程（HDP）的确切想法。层次主题模型应该先给我们几个广泛的主题，这些主题有良好的分离性，然后通过添加更多词汇和更详细的主题定义来进一步详细说明。

HDP 目前仍然比较新，尚未进行广泛的分析。Gensim 在研究中也经常被使用，并且已经集成了 HDP 的实验性实现。由于我们可以直接使用已有的向量化，尝试起来并不复杂。请注意，我们再次使用词袋向量化，因为狄利克雷过程本身可以正确处理频繁出现的词：

```py
from gensim.models import HdpModel
hdp_gensim_para = HdpModel(corpus=bow_gensim_para, id2word=dict_gensim_para)

```

HDP 能够估计主题的数量，并能展示其识别出的所有内容：

```py
hdp_gensim_para.print_topics(num_words=10)

```

`输出：`

![](Images/btap_08in05.jpg)

结果有时很难理解。可以先执行一个只包含少数主题的“粗略”主题建模。如果发现某个主题确实很大或者怀疑可能有子主题，可以创建原始语料库的子集，其中仅包含那些与该主题具有显著混合的文档。这需要一些手动交互，但通常比仅使用 HDP 得到更好的结果。在这个开发阶段，我们不建议仅使用 HDP。

主题模型专注于揭示大量文档语料库的主题结构。由于所有文档被建模为不同主题的混合物，它们不适合于将文档分配到确切的一个主题中。这可以通过聚类来实现。

# Blueprint: 使用聚类揭示文本数据的结构

除了主题建模，还有许多其他无监督方法。并非所有方法都适用于文本数据，但许多聚类算法可以使用。与主题建模相比，对我们来说重要的是每个文档（或段落）都被分配到一个簇中。

# 对于单一类型的文本，聚类效果良好

在我们的情况下，合理假设每个文档属于一个簇，因为一个段落中可能包含的不同内容并不多。对于更大的文本片段，我们更倾向于使用主题建模来考虑可能的混合情况。

大多数聚类方法需要簇的数量作为参数，虽然有少数方法（如均值漂移）可以猜测正确的簇数量。后者大多数不适用于稀疏数据，因此不适合文本分析。在我们的情况下，我们决定使用 k-means 聚类，但 birch 或谱聚类应该以类似的方式工作。有几种解释说明了 k-means 算法的工作原理。^([4](ch08.xhtml#idm45634185352648))

# 聚类比主题建模慢得多

对于大多数算法，聚类需要相当长的时间，甚至比 LDA 还要长。因此，在执行下一个代码片段中的聚类时，请做好大约等待一小时的准备。

scikit-learn 的聚类 API 与我们在主题模型中看到的类似：

```py
from sklearn.cluster import KMeans
k_means_text = KMeans(n_clusters=10, random_state=42)
k_means_text.fit(tfidf_para_vectors)

```

```py
KMeans(n_clusters=10, random_state=42)

```

但是现在要找出有多少段落属于哪个聚类变得更容易了。所有必要的东西都在 `k_means_para` 对象的 `labels_` 字段中。对于每个文档，它包含了聚类算法分配的标签：

```py
np.unique(k_means_para.labels_, return_counts=True)
```

`输出：`

```py
(array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9], dtype=int32),
array([133370,  41705,  12396,   9142,  12674,  21080,  19727,  10563,
         10437,  11116]))
```

在许多情况下，你可能已经发现了一些概念上的问题。如果数据太异构，大多数聚类往往很小（包含相对较小的词汇），并伴随着一个吸收所有剩余的大聚类。幸运的是（由于段落很短），这在这里并不是问题；聚类 0 比其他聚类要大得多，但它并不是数量级。让我们用 y 轴显示聚类的大小来可视化分布（参见 [图 8-6](#fig-cluster-size)）：

```py
sizes = []
for i in range(10):
    sizes.append({"cluster": i, "size": np.sum(k_means_para.labels_==i)})
pd.DataFrame(sizes).set_index("cluster").plot.bar(figsize=(16,9))

```

可视化聚类的工作方式与主题模型类似。但是，我们必须手动计算各个特征的贡献。为此，我们将集群中所有文档的 TF-IDF 向量相加，并仅保留最大值。

![](Images/btap_08in06.jpg)

###### 图 8-6\. 聚类大小的可视化。

这些是它们相应单词的权重。实际上，这与前面的代码唯一的区别就是：

```py
def wordcloud_clusters(model, vectors, features, no_top_words=40):
    for cluster in np.unique(model.labels_):
        size = {}
        words = vectors[model.labels_ == cluster].sum(axis=0).A[0]
        largest = words.argsort()[::-1] # invert sort order
        for i in range(0, no_top_words):
            size[features[largest[i]]] = abs(words[largest[i]])
        wc = WordCloud(background_color="white", max_words=100,
                       width=960, height=540)
        wc.generate_from_frequencies(size)
        plt.figure(figsize=(12,12))
        plt.imshow(wc, interpolation='bilinear')
        plt.axis("off")
        # if you don't want to save the topic model, comment the next line
        plt.savefig(f'cluster{cluster}.png')

wordcloud_clusters(k_means_para, tfidf_para_vectors,
                   tfidf_para_vectorizer.get_feature_names())
```

`输出：`

![](Images/btap_08in07.jpg)

正如你所看到的，结果与各种主题建模方法（幸运地）并没有太大不同；你可能会认出核武器、南非、大会等主题。然而，请注意，聚类更加明显。换句话说，它们有更具体的单词。不幸的是，这并不适用于最大的聚类 1，它没有明确的方向，但有许多具有相似较小尺寸的单词。这是与主题建模相比聚类算法的典型现象。

聚类计算可能需要相当长的时间，尤其是与 NMF 主题模型相比。积极的一面是，我们现在可以自由选择某个聚类中的文档（与主题模型相反，这是明确定义的）并执行其他更复杂的操作，如层次聚类等。

聚类的质量可以通过使用一致性或 Calinski-Harabasz 分数来计算。这些指标并不针对稀疏数据进行优化，计算时间较长，因此我们在这里跳过它们。

# 进一步的想法

在本章中，我们展示了执行主题建模的不同方法。但是，我们只是触及了可能性的表面：

+   可以在向量化过程中添加n-gram。在scikit-learn中，通过使用`ngram_range`参数可以轻松实现这一点。Gensim有一个特殊的`Phrases`类。由于n-gram具有更高的TF-IDF权重，它们可以在话题的特征中起到重要作用，并添加大量的上下文信息。

+   由于我们已经使用多年来依赖时间相关的话题模型，您也可以使用国家或大洲，并找出其大使在演讲中最相关的话题。

+   使用整个演讲而不是段落来计算LDA话题模型的一致性分数，并进行比较。

# 总结和建议

在日常工作中，无监督方法（如话题建模或聚类）通常被用作了解未知文本语料库内容的首选方法。进一步检查是否选择了正确的特征或是否仍可优化，这也是非常有用的。

计算话题时，最重要的决定之一是你将用来计算话题的实体。正如我们蓝图示例所示，文件并不总是最佳选择，特别是当它们非常长，并且由算法确定的子实体组成时。

找到正确的话题数量始终是一个挑战。通常，这必须通过计算质量指标来迭代解决。一个经常使用的更为实用的方法是尝试合理数量的话题，并找出结果是否可解释。

使用（大量）更多的话题（如几百个），话题模型经常被用作文本文档的降维技术。通过生成的向量化，可以在潜在空间中计算相似度分数，并且通常与TF-IDF空间中的朴素距离相比，产生更好的结果。

# 结论

话题模型是一种强大的技术，并且计算成本不高。因此，它们可以广泛用于文本分析。使用它们的首要原因是揭示文档语料库的潜在结构。

话题模型对于获取大型未知文本的总结和结构的概念也是有用的。因此，它们通常在分析的开始阶段被常规使用。

由于存在大量不同的算法和实现方法，因此尝试不同的方法并查看哪种方法在给定的文本语料库中产生最佳结果是有意义的。基于线性代数的方法速度很快，并且通过计算相应的质量指标，可以进行分析。

在执行主题建模之前以不同方式聚合数据可以导致有趣的变化。正如我们在联合国大会辩论数据集中看到的那样，段落更适合，因为发言者一个接一个地讨论了一个话题。如果您有来自许多作者的语料库，将每位作者的所有文本串联起来将为您提供不同类型作者的人物模型。

^([1](ch08.xhtml#idm45634189170488-marker)) Blei, David M., et al. “潜在狄利克雷分配。” *机器学习研究杂志* 3 (4–5): 993–1022\. doi:10.1162/jmlr.2003.3.4-5.993.

^([2](ch08.xhtml#idm45634187759704-marker)) 要了解更详细的描述，请参阅[Wikipedia页面](https://oreil.ly/yr5yA)。

^([3](ch08.xhtml#idm45634187547192-marker)) pyLDAvis必须单独安装，使用**`pip install pyldavis`**或**`conda install pyldavis`**。

^([4](ch08.xhtml#idm45634185352648-marker)) 参见，例如，安德烈·A·沙巴林的[k-means聚类页面](https://oreil.ly/OTGWX)或纳夫塔利·哈里斯的[“可视化K-Means聚类”](https://oreil.ly/Po3bL)。
