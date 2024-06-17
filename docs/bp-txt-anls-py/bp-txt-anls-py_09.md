# 第9章。文本摘要

互联网上有大量关于每个主题的信息。Google 搜索返回数百万条搜索结果，其中包含文本、图像、视频等内容。即使我们只考虑文本内容，也不可能全部阅读。文本摘要方法能够将文本信息压缩成几行或一个段落的简短摘要，并使大多数用户能够理解。文本摘要的应用不仅限于互联网，还包括类似法律助理案例摘要、书籍梗概等领域。

# 您将学到什么以及我们将要构建的内容

在本章中，我们将从文本摘要的介绍开始，并概述所使用的方法。我们将分析不同类型的文本数据及其特定特征，这些特征对于确定摘要方法的选择非常有用。我们将提供适用于不同用例的蓝图，并分析它们的性能。在本章末尾，您将对不同的文本摘要方法有很好的理解，并能够为任何应用选择合适的方法。

# 文本摘要

很可能在生活中的某个时刻，您有意或无意地进行了摘要任务。例如，告诉朋友您昨晚观看的电影，或尝试向家人解释您的工作。我们都喜欢向世界其他地方提供我们经历的简要总结，以分享我们的感受并激励他人。*文本摘要*被定义为在保留有用信息的同时生成更简洁的长文本摘要的方法，而不会失去整体背景。这是一种我们非常熟悉的方法：在阅读课程教材、讲义笔记甚至本书时，许多学生会尝试突出重要的句子或做简短的笔记来捕捉重要概念。自动文本摘要方法允许我们使用计算机来完成这项任务。

摘要方法可以大致分为*抽取*和*生成*方法。在抽取式摘要中，会从给定的文本中识别重要短语或句子，并将它们组合成整个文本的摘要。这些方法通过正确分配权重来识别文本的重要部分，删除可能传达冗余信息的句子，对文本的不同部分进行排名，并将最重要的部分组合成摘要。这些方法选择原始文本的一部分作为摘要，因此每个句子在语法上都是准确的，但可能不会形成连贯的段落。

另一方面，抽象总结方法尝试像人类一样转述并生成摘要。这通常涉及使用能够生成提供文本语法正确摘要的短语和句子的深度神经网络。然而，训练深度神经网络的过程需要大量的训练数据，并且涉及NLP的多个子领域，如自然语言生成、语义分割等。

抽象总结方法是一个活跃研究领域，有几种[方法](https://oreil.ly/DxXd1)致力于改进现有技术。Hugging Face的[`Transformers`库](https://oreil.ly/JS-x8)提供了一个使用预训练模型执行总结任务的实现。我们将在[第11章](ch11.xhtml#ch-sentiment)详细探讨预训练模型和Transformers库的概念。在许多用例中，萃取式总结更受青睐，因为这些方法实现简单且运行速度快。在本章中，我们将专注于使用萃取式总结的蓝图。

假设您在一家法律公司工作，希望查看历史案例以帮助准备当前案例。由于案件程序和判决非常长，他们希望生成摘要，并仅在相关时查看整个案例。这样的摘要帮助他们快速查看多个案例，并有效分配时间。我们可以将此视为应用于长篇文本的文本总结示例。另一个用例可能是媒体公司每天早晨向订阅者发送新闻简报，重点突出前一天的重要事件。客户不喜欢长邮件，因此创建每篇文章的简短摘要对保持他们的参与至关重要。在这种用例中，您需要总结较短的文本。在处理这些项目时，也许您需要在使用Slack或Microsoft Teams等聊天沟通工具的团队中工作。有共享的聊天组（或频道），所有团队成员可以彼此交流。如果您在会议中离开几个小时，聊天信息可能会迅速积累，导致大量未读消息和讨论。作为用户，浏览100多条未读消息很困难，并且无法确定是否错过了重要内容。在这种情况下，通过自动化机器人总结这些错过的讨论可能会有所帮助。

在每个用例中，我们看到不同类型的文本需要总结。让我们简要再次呈现它们：

+   结构化撰写的长篇文本，包含段落并分布在多页之间。例如案件程序、研究论文、教科书等。

+   短文本，如新闻文章和博客，其中可能包含图像、数据和其他图形元素。

+   多个短文本片段采用对话形式，可以包含表情符号等特殊字符，结构不是很严谨。例如Twitter的线索、在线讨论论坛和群组消息应用程序。

这些类型的文本数据每种呈现信息方式不同，因此用于一个类型的摘要方法可能不适用于另一种。在我们的蓝图中，我们提出适用于这些文本类型的方法，并提供指导以确定适当的方法。

## 抽取方法

所有抽取方法都遵循这三个基本步骤：

1.  创建文本的中间表示。

1.  基于选择的表示对句子/短语进行评分。

1.  对句子进行排名和选择，以创建文本摘要。

虽然大多数蓝图会按照这些步骤进行，但它们用来创建中间表示或分数的具体方法会有所不同。

## 数据预处理

在继续实际蓝图之前，我们将重复使用[第3章](ch03.xhtml#ch-scraping)中的蓝图来读取我们想要总结的给定URL。在这份蓝图中，我们将专注于使用文本生成摘要，但您可以研究[第3章](ch03.xhtml#ch-scraping)以获取从URL提取数据的详细概述。为了简洁起见，文章的输出已经缩短；要查看整篇文章，您可以访问以下URL：

```py
import reprlib
r = reprlib.Repr()
r.maxstring = 800

url1 = "https://www.reuters.com/article/us-qualcomm-m-a-broadcom-5g/\
 what-is-5g-and-who-are-the-major-players-idUSKCN1GR1IN"
article_name1 = download_article(url1)
article1 = parse_article(article_name1)
print ('Article Published on', r.repr(article1['time']))
print (r.repr(article1['text']))

```

`输出：`

```py
Article Published on '2018-03-15T11:36:28+0000'
'LONDON/SAN FRANCISCO (Reuters) - U.S. President Donald Trump has blocked
microchip maker Broadcom Ltd’s (AVGO.O) $117 billion takeover of rival Qualcomm
(QCOM.O) amid concerns that it would give China the upper hand in the next
generation of mobile communications, or 5G. A 5G sign is seen at the Mobile
World Congress in Barcelona, Spain February 28, 2018\. REUTERS/Yves HermanBelow
are some facts... 4G wireless and looks set to top the list of patent holders
heading into the 5G cycle. Huawei, Nokia, Ericsson and others are also vying to
amass 5G patents, which has helped spur complex cross-licensing agreements like
the deal struck late last year Nokia and Huawei around handsets. Editing by Kim
Miyoung in Singapore and Jason Neely in LondonOur Standards:The Thomson Reuters
Trust Principles.'

```

###### 注意

我们使用`reprlib`包，该包允许我们自定义打印语句的输出。在这种情况下，打印完整文章的内容是没有意义的。我们限制输出的大小为800个字符，`reprlib`包重新格式化输出，显示文章开头和结尾的选定序列词语。

# 蓝图：使用主题表示进行文本摘要

让我们首先尝试自己总结一下例子Reuters文章。阅读完之后，我们可以提供以下手动生成的摘要：

> 5G是下一代无线技术，将依赖更密集的小天线阵列，提供比当前4G网络快50到100倍的数据速度。这些新网络预计不仅将数据传输速度提高到手机和电脑，还将扩展到汽车、货物、农作物设备等各种传感器。高通是今天智能手机通信芯片市场的主导者，人们担心新加坡的博通公司收购高通可能会导致高通削减研发支出或将公司战略重要部分出售给其他买家，包括在中国的买家。这可能会削弱高通，在5G竞赛中促进中国超越美国的风险。

作为人类，我们理解文章传达的内容，然后生成我们理解的摘要。然而，算法没有这种理解，因此必须依赖于识别重要主题来确定是否应将句子包括在摘要中。在示例文章中，主题可能是技术、电信和5G等广泛主题，但对于算法来说，这只是一组重要单词的集合。我们的第一种方法试图区分重要和不那么重要的单词，从而使我们能够将包含重要单词的句子排名较高。

## 使用TF-IDF值识别重要词语

最简单的方法是基于句子中单词的TF-IDF值的总和来识别重要句子。详细解释TF-IDF在[第5章](ch05.xhtml#ch-vectorization)中提供，但对于这个蓝图，我们应用TF-IDF向量化，然后将值聚合到句子级别。我们可以为每个句子生成一个分数，作为该句子中每个单词的TF-IDF值的总和。这意味着得分高的句子包含的重要单词比文章中的其他句子多：

```py
from sklearn.feature_extraction.text import TfidfVectorizer
from nltk import tokenize

sentences = tokenize.sent_tokenize(article1['text'])
tfidfVectorizer = TfidfVectorizer()
words_tfidf = tfidfVectorizer.fit_transform(sentences)

```

在这种情况下，文章中大约有20句话，我们选择创建一个只有原始文章大小的10%的简要总结（大约两到三句话）。我们对每个句子的TF-IDF值进行求和，并使用`np.argsort`对它们进行排序。这种方法按升序对每个句子的索引进行排序，我们使用`[::-1]`来逆转返回的索引。为了确保与文章中呈现的思路相同，我们按照它们出现的顺序打印所选的摘要句子。我们可以看到我们生成的摘要结果，如下所示：

```py
# Parameter to specify number of summary sentences required
num_summary_sentence = 3

# Sort the sentences in descending order by the sum of TF-IDF values
sent_sum = words_tfidf.sum(axis=1)
important_sent = np.argsort(sent_sum, axis=0)[::-1]

# Print three most important sentences in the order they appear in the article
for i in range(0, len(sentences)):
    if i in important_sent[:num_summary_sentence]:
        print (sentences[i])

```

`输出：`

```py
LONDON/SAN FRANCISCO (Reuters) - U.S. President Donald Trump has blocked
microchip maker Broadcom Ltd’s (AVGO.O) $117 billion takeover of rival Qualcomm
(QCOM.O) amid concerns that it would give China the upper hand in the next
generation of mobile communications, or 5G.
5G networks, now in the final testing stage, will rely on denser arrays of
small antennas and the cloud to offer data speeds up to 50 or 100 times faster
than current 4G networks and serve as critical infrastructure for a range of
industries.
The concern is that a takeover by Singapore-based Broadcom could see the firm
cut research and development spending by Qualcomm or hive off strategically
important parts of the company to other buyers, including in China, U.S.
officials and analysts have said.

```

在这种方法中，我们使用TF-IDF值创建文本的中间表示，根据这些值对句子进行评分，并选择三个得分最高的句子。使用这种方法选择的句子与我们之前写的手动摘要一致，并捕捉了文章涵盖的主要要点。一些细微差别，比如Qualcomm在行业中的重要性和5G技术的具体应用，被忽略了。但这种方法作为快速识别重要句子并自动生成新闻文章摘要的良好蓝图。我们将这个蓝图封装成一个名为`tfidf_summary`的函数，该函数在附带的笔记本中定义并在本章后面再次使用。

## LSA算法

在基于抽取的摘要方法中，使用的一种现代方法是*潜在语义分析*（LSA）。LSA是一种通用方法，用于主题建模、文档相似性和其他任务。LSA假设意思相近的词会出现在同一篇文档中。在LSA算法中，我们首先将整篇文章表示为一个句子-词矩阵。文档-词矩阵的概念已在[第8章](ch08.xhtml#ch-topicmodels)中介绍过，我们可以将该概念调整为适合句子-词矩阵。每行代表一个句子，每列代表一个词。该矩阵中每个单元格的值是词频通常按TF-IDF权重进行缩放。该方法的目标是通过创建句子-词矩阵的修改表示来将所有单词减少到几个主题中。为了创建修改后的表示，我们应用非负矩阵分解的方法，将该矩阵表示为具有较少行/列的两个新分解矩阵的乘积。您可以参考[第8章](ch08.xhtml#ch-topicmodels)更详细地了解这一方法。在矩阵分解步骤之后，我们可以通过选择前N个重要主题生成摘要，然后选择每个主题中最重要的句子来形成我们的摘要。

我们不再从头开始应用LSA，而是利用`sumy`包，可以使用命令`**pip install sumy**`进行安装。该库提供了同一库内的多种摘要方法。此库使用一个集成的停用词列表，并结合来自NLTK的分词器和词干处理功能，但可以进行配置。此外，它还能够从纯文本、HTML和文件中读取输入。这使我们能够快速测试不同的摘要方法，并更改默认配置以适应特定的使用案例。目前，我们将使用默认选项，包括识别前三个句子：

```py
from sumy.parsers.plaintext import PlaintextParser
from sumy.nlp.tokenizers import Tokenizer
from sumy.nlp.stemmers import Stemmer
from sumy.utils import get_stop_words

from sumy.summarizers.lsa import LsaSummarizer

LANGUAGE = "english"
stemmer = Stemmer(LANGUAGE)

parser = PlaintextParser.from_string(article1['text'], Tokenizer(LANGUAGE))
summarizer = LsaSummarizer(stemmer)
summarizer.stop_words = get_stop_words(LANGUAGE)

for sentence in summarizer(parser.document, num_summary_sentence):
    print (str(sentence))

```

`输出：`

```py
LONDON/SAN FRANCISCO (Reuters) - U.S. President Donald Trump has blocked
microchip maker Broadcom Ltd’s (AVGO.O) $117 billion takeover of rival Qualcomm
(QCOM.O) amid concerns that it would give China the upper hand in the next
generation of mobile communications, or 5G.
Moving to new networks promises to enable new mobile services and even whole
new business models, but could pose challenges for countries and industries
unprepared to invest in the transition.
The concern is that a takeover by Singapore-based Broadcom could see the firm
cut research and development spending by Qualcomm or hive off strategically
important parts of the company to other buyers, including in China, U.S.
officials and analysts have said.

```

通过分析结果，我们看到TF-IDF的结果仅有一句与LSA的结果有所不同，即第2句。虽然LSA方法选择突出显示一个关于挑战主题的句子，但TF-IDF方法选择了一个更多关于5G信息的句子。在这种情况下，两种方法生成的摘要并没有非常不同，但让我们分析一下这种方法在更长文章上的工作效果。

我们将这个蓝图封装成一个函数`lsa_summary`，该函数在附带的笔记本中定义，并可重复使用：

```py
r.maxstring = 800
url2 = "https://www.reuters.com/article/us-usa-economy-watchlist-graphic/\
 predicting-the-next-u-s-recession-idUSKCN1V31JE"
article_name2 = download_article(url2)
article2 = parse_article(article_name2)
print ('Article Published', r.repr(article1['time']))
print (r.repr(article2['text']))

```

`输出：`

```py
Article Published '2018-03-15T11:36:28+0000'
'NEW YORK A protracted trade war between China and the United States, the
world’s largest economies, and a deteriorating global growth outlook has left
investors apprehensive about the end to the longest expansion in American
history. FILE PHOTO: Ships and shipping containers are pictured at the port of
Long Beach in Long Beach, California, U.S., January 30, 2019\.   REUTERS/Mike
BlakeThe recent ...hton wrote in the June Cass Freight Index report.  12.
MISERY INDEX The so-called Misery Index adds together the unemployment rate and
the inflation rate. It typically rises during recessions and sometimes prior to
downturns. It has slipped lower in 2019 and does not look very miserable.
Reporting by Saqib Iqbal Ahmed; Editing by Chizu NomiyamaOur Standards:The
Thomson Reuters Trust Principles.'

```

然后：

```py
summary_sentence = tfidf_summary(article2['text'], num_summary_sentence)
for sentence in summary_sentence:
    print (sentence)

```

`输出：`

```py
REUTERS/Mike BlakeThe recent rise in U.S.-China trade war tensions has brought
forward the next U.S. recession, according to a majority of economists polled
by Reuters who now expect the Federal Reserve to cut rates again in September
and once more next year.
On Tuesday, U.S. stocks jumped sharply higher and safe-havens like the Japanese
yen and Gold retreated after the U.S. Trade Representative said additional
tariffs on some Chinese goods, including cell phones and laptops, will be
delayed to Dec. 15.
ISM said its index of national factory activity slipped to 51.2 last month, the
lowest reading since August 2016, as U.S. manufacturing activity slowed to a
near three-year low in July and hiring at factories shifted into lower gear,
suggesting a further loss of momentum in economic growth early in the third
quarter.

```

最后：

```py
summary_sentence = lsa_summary(article2['text'], num_summary_sentence)
for sentence in summary_sentence:
    print (sentence)

```

`输出：`

```py
NEW YORK A protracted trade war between China and the United States, the
world’s largest economies, and a deteriorating global growth outlook has left
investors apprehensive about the end to the longest expansion in American
history.
REUTERS/Mike BlakeThe recent rise in U.S.-China trade war tensions has brought
forward the next U.S. recession, according to a majority of economists polled
by Reuters who now expect the Federal Reserve to cut rates again in September
and once more next year.
Trade tensions have pulled corporate confidence and global growth to multi-year
lows and U.S. President Donald Trump’s announcement of more tariffs have raised
downside risks significantly, Morgan Stanley analysts said in a recent note.

```

在这里，选择的摘要句子的差异变得更加明显。贸易战紧张局势的主要话题被两种方法捕捉到，但LSA摘要器还突出了投资者的担忧和企业信心等重要话题。虽然TF-IDF试图在其选择的句子中表达相同的观点，但它没有选择正确的句子，因此未能传达这一观点。还有其他基于主题的摘要方法，但我们选择突出LSA作为一个简单且广泛使用的方法。

###### 注意

有趣的是，`sumy`库还提供了自动文本摘要的一个最古老的方法（`LuhnSummarizer`）的实现，该方法由[Hans Peter Luhn于1958年创造](https://oreil.ly/j6cQI)。这种方法也是基于通过识别重要词汇的计数和设置阈值来表示主题。您可以将其用作文本摘要实验的基准方法，并比较其他方法提供的改进。

# 蓝图：使用指示器表示对文本进行摘要

指示器表示方法旨在通过使用句子的特征及其与文档中其他句子的关系来创建句子的中间表示，而不仅仅是使用句子中的单词。[TextRank](https://oreil.ly/yY29h)是指示器方法中最流行的例子之一。TextRank受PageRank启发，是一种“基于图的排名算法，最初由Google用于排名搜索结果。根据TextRank论文的作者，基于图的算法依赖于网页结构的集体知识，而不是单个网页内容的分析”，这导致了改进的性能。在我们的背景下应用，我们将依赖句子的特征和它们之间的链接，而不是依赖每个句子所包含的主题。

首先我们将尝试理解PageRank算法的工作原理，然后将方法应用于文本摘要问题。让我们考虑一个网页列表（A、B、C、D、E和F）及其彼此之间的链接。在[图9-1](#fig-pagerank-graph)中，页面A包含指向页面D的链接。页面B包含指向A和D的链接，依此类推。我们还可以用一个矩阵表示，行表示每个页面，列表示来自其他页面的入链。图中显示的矩阵表示我们的图，行表示每个节点，列表示来自其他节点的入链，单元格的值表示它们之间边的权重。我们从一个简单的表示开始（1表示有入链，0表示没有）。然后我们可以通过将每个网页的出链总数进行除法来归一化这些值。例如，页面C有两个出链（到页面E和F），因此每个出链的值为0.5。

![](Images/btap_0901.jpg)

###### 图9-1\. 网页链接和相应的PageRank矩阵。

对于给定页面的PageRank是所有具有链接的其他页面的PageRank的加权和。这也意味着计算PageRank是一个迭代函数，我们必须从每个页面的一些假设的PageRank初始值开始。如果我们假设所有初始值为1，并按照[图9-2](#fig-pagerank-results)所示的方式进行矩阵乘法，我们可以在一次迭代后得到每个页面的PageRank（不考虑此示例的阻尼因子）。

[Brin和Page的研究论文](https://oreil.ly/WjjFv)表明，重复进行多次迭代计算后，数值稳定，因此我们得到每个页面的PageRank或重要性。TextRank通过将文本中的每个句子视为一个页面和因此图中的一个节点来调整先前的方法。节点之间边的权重由句子之间的相似性决定，TextRank的作者建议通过计算共享词汇标记的数量（归一化为两个句子的大小）来实现简单的方法。还有其他相似度度量，如余弦距离和最长公共子串也可以使用。

![](Images/btap_0902.jpg)

###### 图9-2\. PageRank算法的一次迭代应用。

由于sumy包还提供了TextRank实现，我们将使用它为我们之前看到的关于美国经济衰退的文章生成总结的句子：

```py
from sumy.summarizers.text_rank import TextRankSummarizer

parser = PlaintextParser.from_string(article2['text'], Tokenizer(LANGUAGE))
summarizer = TextRankSummarizer(stemmer)
summarizer.stop_words = get_stop_words(LANGUAGE)

for sentence in summarizer(parser.document, num_summary_sentence):
    print (str(sentence))

```

```py
REUTERS/Mike BlakeThe recent rise in U.S.-China trade war tensions has brought
forward the next U.S. recession, according to a majority of economists polled
by Reuters who now expect the Federal Reserve to cut rates again in September
and once more next year.
As recession signals go, this so-called inversion in the yield curve has a
solid track record as a predictor of recessions.
Markets turned down before the 2001 recession and tumbled at the start of the
2008 recession.

```

当总结句之一保持不变时，这种方法选择返回其他两个可能与本文主要结论相关联的句子。虽然这些句子本身可能并不重要，但使用基于图的方法选择了支持文章主题的高度关联句子。我们将这个蓝图封装成一个函数`textrank_summary`，允许我们进行重复使用。

我们还想看看这种方法在我们之前查看过的关于5G技术的较短文章上的运作：

```py
parser = PlaintextParser.from_string(article1['text'], Tokenizer(LANGUAGE))
summarizer = TextRankSummarizer(stemmer)
summarizer.stop_words = get_stop_words(LANGUAGE)

for sentence in summarizer(parser.document, num_summary_sentence):
    print (str(sentence))

```

`Out:`

```py
Acquiring Qualcomm would represent the jewel in the crown of Broadcom’s
portfolio of communications chips, which supply wi-fi, power management, video
and other features in smartphones alongside Qualcomm’s core baseband chips -
radio modems that wirelessly connect phones to networks.
Qualcomm (QCOM.O) is the dominant player in smartphone communications chips,
making half of all core baseband radio chips in smartphones.
Slideshow (2 Images)The standards are set by a global body to ensure all phones
work across different mobile networks, and whoever’s essential patents end up
making it into the standard stands to reap huge royalty licensing revenue
streams.

```

我们看到，结果捕捉到了高通收购的中心思想，但没有提及LSA方法选择的5G技术。TextRank通常在长文本内容的情况下表现更好，因为它能够使用图链接识别最重要的句子。在较短的文本内容中，图不是很大，因此网络智慧发挥的作用较小。让我们使用来自维基百科的更长内容的例子来进一步说明这一点。我们将重复使用来自[第2章](ch02.xhtml#ch-api)的蓝图，下载维基百科文章的文本内容。在这种情况下，我们选择描述历史事件或事件系列的文章：蒙古入侵欧洲。由于这是更长的文本，我们选择总结大约10句话，以提供更好的总结：

```py
p_wiki = wiki_wiki.page('Mongol_invasion_of_Europe')
print (r.repr(p_wiki.text))

```

`Out:`

```py
'The Mongol invasion of Europe in the 13th century occurred from the 1220s into
the 1240s. In Eastern Europe, the Mongols destroyed Volga Bulgaria, Cumania,
Alania, and the Kievan Rus\' federation. In Central Europe, the Mongol armies
launched a tw...tnotes\nReferences\nSverdrup, Carl (2010). "Numbers in Mongol
Warfare". Journal of Medieval Military History. Boydell Press. 8: 109–17 [p.
115]. ISBN 978-1-84383-596-7.\n\nFurther reading\nExternal links\nThe Islamic
World to 1600: The Golden Horde'

```

然后：

```py
r.maxstring = 200

num_summary_sentence = 10

summary_sentence = textrank_summary(p_wiki.text, num_summary_sentence)

for sentence in summary_sentence:
    print (sentence)

```

我们将结果展示为原始维基百科页面中的突出显示句子（[Figure 9-3](#fig-wikipedia-summary)），以展示使用TextRank算法通过从文章的每个部分中选择最重要的句子，几乎准确地对文章进行了总结。我们可以比较这与LSA方法的工作，但我们将这留给读者使用先前的蓝图作为练习。根据我们的经验，当我们想要总结大量的文本内容时，例如科学研究论文、作品集以及世界领导人的演讲或多个网页时，我们会选择像TextRank这样基于图的方法。

![](Images/btap_0903.jpg)

###### Figure 9-3\. 维基百科页面，突出显示了选定的摘要句子。

# 衡量文本摘要方法的性能

到目前为止，我们在蓝图中已经看到了许多方法来生成某段文本的摘要。每个摘要在细微之处都有所不同，我们必须依靠我们的主观评估。在选择最适合特定使用案例的方法方面，这无疑是一个挑战。在本节中，我们将介绍常用的准确度度量标准，并展示它们如何被用来经验性地选择最佳的摘要方法。

我们必须理解，要自动评估某段给定文本的摘要，必须有一个可以进行比较的参考摘要。通常，这是由人类编写的摘要，称为*黄金标准*。每个自动生成的摘要都可以与黄金标准进行比较，以获得准确度的度量。这也为我们提供了比较多种方法并选择最佳方法的机会。然而，我们经常会遇到一个问题，即并非每个使用案例都有人类生成的摘要存在。在这种情况下，我们可以选择一个代理度量来视为黄金标准。在新闻文章的案例中，一个例子就是标题。虽然它是由人类编写的，但作为一个代理度量它并不准确，因为它可能非常简短，更像是一个引导性陈述来吸引用户。虽然这可能不会给我们带来最佳结果，但比较不同摘要方法的性能仍然是有用的。

*用于Gisting评估的召回导向的Understudy*（ROUGE）是最常用的测量摘要准确性的方法之一。有几种类型的ROUGE度量标准，但基本思想很简单。它通过比较自动生成的摘要与黄金标准之间的共享术语数量来得出准确度的度量。ROUGE-N是一种度量标准，用于衡量常见的n-gram（ROUGE-1比较单个词，ROUGE-2比较二元组，依此类推）。

原始的 [ROUGE 论文](https://oreil.ly/Tsncq) 比较了在自动生成的摘要中出现的单词中有多少也出现在金标准中。这就是我们在 [第 6 章](ch06.xhtml#ch-classification) 中介绍的 *召回率*。因此，如果金标准中大多数单词也出现在生成的摘要中，我们将获得高分。然而，单靠这一指标并不能讲述整个故事。考虑到我们生成了一个冗长但包含金标准中大多数单词的摘要。这个摘要将获得高分，但它不是一个好的摘要，因为它没有提供简洁的表示。这就是为什么 ROUGE 测量已经扩展到将共享单词的数量与生成的摘要中的总单词数进行比较。这表明了精度：生成摘要中实际有用的单词数。我们可以结合这些措施生成 F 分数。

让我们看一个我们生成摘要的 ROUGE 示例。由于我们没有金标准的人工生成摘要，我们使用文章标题作为金标准的代理。虽然这样计算独立简单，但我们利用名为 `rouge_scorer` 的 Python 包来使我们的生活更轻松。这个包实现了我们后来将使用的所有 ROUGE 测量，并且可以通过执行命令 `**pip install rouge_scorer**` 进行安装。我们利用一个打印实用函数 `print_rouge_score` 来展示得分的简洁视图：

```py
num_summary_sentence = 3
gold_standard = article2['headline']
summary = ""

summary = ''.join(textrank_summary(article2['text'], num_summary_sentence))
scorer = rouge_scorer.RougeScorer(['rouge1'], use_stemmer=True)
scores = scorer.score(gold_standard, summary)
print_rouge_score(scores)

```

`输出：`

```py
rouge1 Precision: 0.06 Recall: 0.83 fmeasure: 0.11

```

先前的结果显示，TextRank 生成的摘要具有高召回率但低精度。这是我们金标准是一个极短标题的结果，本身并不是最佳选择，但在这里用于说明。我们度量标准的最重要用途是与另一种总结方法进行比较，在这种情况下，让我们与 LSA 生成的摘要进行比较：

```py
summary = ''.join(lsa_summary(article2['text'], num_summary_sentence))
scores = scorer.score(gold_standard, summary)
print_rouge_score(scores)

```

`输出：`

```py
rouge1 Precision: 0.04 Recall: 0.83 fmeasure: 0.08

```

上述结果表明，在这种情况下，TextRank 是优越的方法，因为它具有更高的精度，而两种方法的召回率相同。我们可以轻松地扩展 ROUGE-1 到 ROUGE-2，这将比较两个词（二元组）的公共序列的数量。另一个重要的指标是 ROUGE-L，它通过识别参考摘要与生成摘要之间的最长公共子序列来衡量。句子的子序列是一个新句子，可以从原始句子中删除一些单词而不改变剩余单词的相对顺序。这个指标的优势在于它不专注于精确的序列匹配，而是反映句子级词序的顺序匹配。让我们分析维基百科页面的 ROUGE-2 和 ROUGE-L 指标。再次强调，我们没有一个金标准，因此我们将使用简介段落作为我们金标准的代理：

```py
num_summary_sentence = 10
gold_standard = p_wiki.summary

summary = ''.join(textrank_summary(p_wiki.text, num_summary_sentence))

scorer = rouge_scorer.RougeScorer(['rouge2','rougeL'], use_stemmer=True)
scores = scorer.score(gold_standard, summary)
print_rouge_score(scores)

```

`输出：`

```py
rouge2 Precision: 0.18 Recall: 0.46 fmeasure: 0.26
rougeL Precision: 0.16 Recall: 0.40 fmeasure: 0.23

```

然后：

```py
summary = ''.join(lsa_summary(p_wiki.text, num_summary_sentence))

scorer = rouge_scorer.RougeScorer(['rouge2','rougeL'], use_stemmer=True)
scores = scorer.score(gold_standard, summary)
print_rouge_score(scores)

```

`输出：`

```py
rouge2 Precision: 0.04 Recall: 0.08 fmeasure: 0.05
rougeL Precision: 0.12 Recall: 0.25 fmeasure: 0.16

```

根据结果，我们看到TextRank比LSA更准确。我们可以使用与前面展示的相同方法来查看哪种方法对较短的维基百科条目效果最好，这将留给读者作为练习。当应用到您的用例时，重要的是选择正确的摘要进行比较。例如，在处理新闻文章时，您可以查找文章内包含的摘要部分，而不是使用标题，或者为少数文章生成自己的摘要。这样可以在不同方法之间进行公平比较。

# 蓝图：使用机器学习进行文本总结

许多人可能参与了关于旅行规划、编程等主题的在线讨论论坛。在这些平台上，用户以线程的形式进行交流。任何人都可以开始一个线程，其他成员则在该线程上提供他们的回应。线程可能会变得很长，关键信息可能会丢失。在这个蓝图中，我们将使用从研究论文中提取的数据，^([2](ch09.xhtml#idm45634183488696)) 这些数据包含了一个线程中所有帖子的文本以及该线程的摘要，如[图9-4](#fig-thread-illustration)所示。

在这个蓝图中，我们将使用机器学习来帮助我们自动识别整个线程中最重要的帖子，这些帖子准确地总结了整个线程。我们首先使用注释者的摘要为我们的数据集创建目标标签。然后生成能够确定特定帖子是否应该出现在摘要中的特征，并最终训练一个模型并评估其准确性。手头的任务类似于文本分类，但是在帖子级别上执行。

虽然论坛线程用于说明这个蓝图，但它也可以轻松地用于其他用例。例如，考虑[CNN和每日邮报新闻摘要任务](https://oreil.ly/T_RNc)，[DUC](https://oreil.ly/0Hlov)，或[SUMMAC](https://oreil.ly/Wg322)数据集。在这些数据集中，你会找到每篇文章的文本和突出显示的摘要句子。这些与本蓝图中呈现的每个线程的文本和摘要类似。

![图片](Images/btap_0904.jpg)

###### 图9-4。一个线程中的帖子及其来自旅行论坛的对应摘要。

## 第1步：创建目标标签

第一步是加载数据集，了解其结构，并使用提供的摘要创建目标标签。我们已经执行了初始的数据准备步骤，创建了一个格式良好的`DataFrame`，如下所示。请参阅书籍的GitHub仓库中的`Data_Preparation`笔记本，详细了解这些步骤：

```py
import pandas as pd
import numpy as np

df = pd.read_csv('travel_threads.csv', sep='|', dtype={'ThreadID': 'object'})
df[df['ThreadID']=='60763_5_3122150'].head(1).T

```

|   | 170 |
| --- | --- |
| 日期 | 2009年9月29日，1:41 |
| 文件名 | thread41_system20 |
| 线程ID | 60763_5_3122150 |
| 标题 | 需要预订哪些景点？ |
| 帖子编号 | 1 |
| text | Hi I am coming to NY in Oct! So excited&quot; Have wanted to visit for years. We are planning on doing all the usual stuff so wont list it all but wondered which attractions should be pre booked and which can you just turn up at> I am plannin on booking ESB but what else? thanks x |
| 用户ID | musicqueenLon... |
| summary | A woman was planning to travel NYC in October and needed some suggestions about attractions in the NYC. She was planning on booking ESB.Someone suggested that the TOTR was much better compared to ESB. The other suggestion was to prebook the show to avoid wasting time in line.Someone also suggested her New York Party Shuttle tours. |

这个数据集中的每一行都指的是主题中的一个帖子。每个主题由一个唯一的 `ThreadID` 标识，`DataFrame` 中可能有多行具有相同的 `ThreadID`。`Title` 列指的是用户开始主题时使用的名称。每个帖子的内容都在 `text` 列中，还包括其他细节，比如创建帖子的用户的姓名（`userID`）、帖子创建时间（`Date`）以及在主题中的位置（`postNum`）。对于这个数据集，每个主题都提供了人工生成的摘要，位于 `summary` 列中。

我们将重用[第四章](ch04.xhtml#ch-preparation)中的正则表达式清理和 spaCy 流水线蓝图，以删除帖子中的特殊格式、URL 和其他标点符号。我们还将生成文本的词形还原表示，用于预测。你可以在本章的附带笔记本中找到函数定义。由于我们正在使用 spaCy 的词形还原功能，执行可能需要几分钟才能完成：

```py
# Applying regex based cleaning function
df['text'] = df['text'].apply(regex_clean)
# Extracting lemmas using spacy pipeline
df['lemmas'] = df['text'].apply(clean)

```

我们数据集中的每个观测都包含一个帖子，该帖子是主题的一部分。如果我们在这个层面应用训练-测试分割，那么可能会导致两个属于同一主题的帖子分别进入训练集和测试集，这将导致训练不准确。因此，我们使用 `GroupShuffleSplit` 将所有帖子分组到它们各自的主题中，然后随机选择 80% 的主题来创建训练数据集，其余的主题组成测试数据集。这个函数确保属于同一主题的帖子属于同一数据集。`GroupShuffleSplit` 函数实际上并不分割数据，而是提供了一组索引，这些索引标识了由 `train_split` 和 `test_split` 确定的数据的分割。我们使用这些索引来创建这两个数据集：

```py
from sklearn.model_selection import GroupShuffleSplit

gss = GroupShuffleSplit(n_splits=1, test_size=0.2)
train_split, test_split = next(gss.split(df, groups=df['ThreadID']))

```

```py
train_df = df.iloc[train_split]
test_df = df.iloc[test_split]

print ('Number of threads for Training ', train_df['ThreadID'].nunique())
print ('Number of threads for Testing ', test_df['ThreadID'].nunique())

```

`输出：`

```py
Number of threads for Training  559
Number of threads for Testing  140

```

我们的下一步是确定每篇文章的目标标签。目标标签定义了是否应将特定文章包含在摘要中。我们通过将每篇文章与注释员摘要进行比较，并选择最相似的文章来确定这一点。有几种度量标准可用于确定两个句子的相似性，但在我们的用例中，我们处理短文本，因此选择了[Jaro-Winkler距离](https://oreil.ly/b5q0B)。我们使用`textdistance`包，该包还提供其他距离度量的实现。您可以使用命令`**pip install textdistance**`轻松安装它。您还可以轻松修改蓝图，并根据您的用例选择度量标准。

在接下来的步骤中，我们根据所选择的度量标准确定相似性并对主题中的所有帖子进行排序。然后，我们创建名为`summaryPost`的目标标签，其中包含一个True或False值，指示此帖子是否属于摘要。这是基于帖子的排名和压缩因子。我们选择了30%的压缩因子，这意味着我们选择按相似性排序的所有帖子中的前30%来包含在摘要中：

```py
import textdistance

compression_factor = 0.3

train_df['similarity'] = train_df.apply(
    lambda x: textdistance.jaro_winkler(x.text, x.summary), axis=1)
train_df["rank"] = train_df.groupby("ThreadID")["similarity"].rank(
    "max", ascending=False)

topN = lambda x: x <= np.ceil(compression_factor * x.max())
train_df['summaryPost'] = train_df.groupby('ThreadID')['rank'].apply(topN)

```

```py
train_df[['text','summaryPost']][train_df['ThreadID']=='60763_5_3122150'].head(3)

```

`输出:`

|   | text | summaryPost |
| --- | --- | --- |
| 170 | 嗨，我十月份要去纽约！好兴奋！多年来一直想去参观。我们计划做所有传统的事情，所以不会列出所有的事情，但想知道哪些景点应该提前预订，哪些可以直接到场？我打算预订帝国大厦，还有什么？谢谢 x | True |
| 171 | 如果我是你，我不会去帝国大厦，TOPR要好得多。你还有哪些景点考虑？ | False |
| 172 | 自由女神像，如果您计划去雕像本身或埃利斯岛（而不是乘船经过）：http://www.statuecruises.com/ 另外，我们更喜欢提前预订演出和戏剧，而不是尝试购买当天票，因为这样可以避免排队浪费时间。如果这听起来对您有吸引力，请看看http://www.broadwaybox.com/ | True |

正如您在前面的结果中看到的，对于给定的主题，第一和第三篇文章被标记为`summaryPost`，但第二篇文章不重要，不会被包含在摘要中。由于我们定义了目标标签的方式，很少情况下可能会将非常短的帖子包含在摘要中。当一个短帖子包含与主题标题相同的词时，可能会发生这种情况。这对摘要没有用，我们通过将所有包含20个词或更少的帖子设置为不包含在摘要中来进行修正：

```py
train_df.loc[train_df['text'].str.len() <= 20, 'summaryPost'] = False

```

## 步骤2：添加帮助模型预测的特征

由于我们在这个蓝图中处理的是论坛主题，我们可以生成一些额外的特征来帮助我们的模型进行预测。主题的标题简洁地传达了主题，并且在识别应该在摘要中实际选择的帖子时可能会有所帮助。我们不能直接将标题包含为一个特征，因为对于主题中的每个帖子来说它都是相同的，但是我们可以计算帖子与标题之间的相似度作为其中一个特征：

```py
train_df['titleSimilarity'] = train_df.apply(
    lambda x: textdistance.jaro_winkler(x.text, x.Title), axis=1)

```

另一个有用的特征可能是帖子的长度。短帖子可能是在询问澄清问题，不会捕捉到主题的最有用的知识。长帖子可能表明正在分享大量有用信息。帖子在主题中的位置也可能是一个有用的指标，用于确定是否应该将其包含在摘要中。这可能会根据论坛主题的组织方式而有所不同。在旅行论坛的情况下，帖子是按时间顺序排序的，帖子的发生是通过列`postNum`给出的，我们可以直接将其用作一个特征：

```py
# Adding post length as a feature
train_df['textLength'] = train_df['text'].str.len()

```

作为最后一步，让我们使用*TfidfVectorizer*创建我们之前提取的词元的向量化表示。然后，我们创建一个新的`DataFrame`，`train_df_tf`，其中包含向量化的词元和我们之前创建的附加特征：

```py
feature_cols = ['titleSimilarity','textLength','postNum']

```

```py
train_df['combined'] = [
    ' '.join(map(str, l)) for l in train_df['lemmas'] if l is not '']
tfidf = TfidfVectorizer(min_df=10, ngram_range=(1, 2), stop_words="english")
tfidf_result = tfidf.fit_transform(train_df['combined']).toarray()

tfidf_df = pd.DataFrame(tfidf_result, columns=tfidf.get_feature_names())
tfidf_df.columns = ["word_" + str(x) for x in tfidf_df.columns]
tfidf_df.index = train_df.index
train_df_tf = pd.concat([train_df[feature_cols], tfidf_df], axis=1)

```

添加特征的这一步骤可以根据使用情况进行扩展或定制。例如，如果我们想要总结更长的文本，那么一个句子所属的段落将是重要的。通常，每个段落或部分都试图捕捉一个思想，并且在该水平上使用的句子相似性度量将是相关的。如果我们试图生成科学论文的摘要，那么引用次数和用于这些引用的句子已被证明是有用的。我们还必须在测试数据集上重复相同的特征工程步骤，我们在附带的笔记本中展示了这一点，但在这里排除了。

## 步骤3：构建机器学习模型

现在我们已经生成了特征，我们将重用[第6章](ch06.xhtml#ch-classification)中的文本分类蓝图，但是使用`RandomForestClassifier`模型代替SVM模型。在构建用于摘要的机器学习模型时，我们可能有除了向量化的文本表示之外的其他特征。特别是在存在数字和分类特征的组合的情况下，基于树的分类器可能会表现得更好：

```py
from sklearn.ensemble import RandomForestClassifier

model1 = RandomForestClassifier()
model1.fit(train_df_tf, train_df['summaryPost'])

```

`输出:`

```py
RandomForestClassifier(bootstrap=True, ccp_alpha=0.0, class_weight=None,
                       criterion='gini', max_depth=None, max_features='auto',
                       max_leaf_nodes=None, max_samples=None,
                       min_impurity_decrease=0.0, min_impurity_split=None,
                       min_samples_leaf=1, min_samples_split=2,
                       min_weight_fraction_leaf=0.0, n_estimators=100,
                       n_jobs=None, oob_score=False, random_state=20, verbose=0,
                       warm_start=False)

```

让我们在测试主题上应用这个模型，并预测摘要帖子。为了确定准确性，我们连接所有识别的摘要帖子，并通过与注释摘要进行比较生成ROUGE-1分数：

```py
# Function to calculate rouge_score for each thread
def calculate_rouge_score(x, column_name):
    # Get the original summary - only first value since they are repeated
    ref_summary = x['summary'].values[0]

    # Join all posts that have been predicted as summary
    predicted_summary = ''.join(x['text'][x[column_name]])

    # Return the rouge score for each ThreadID
    scorer = rouge_scorer.RougeScorer(['rouge1'], use_stemmer=True)
    scores = scorer.score(ref_summary, predicted_summary)
    return scores['rouge1'].fmeasure

```

```py
test_df['predictedSummaryPost'] = model1.predict(test_df_tf)
print('Mean ROUGE-1 Score for test threads',
      test_df.groupby('ThreadID')[['summary','text','predictedSummaryPost']] \
      .apply(calculate_rouge_score, column_name='predictedSummaryPost').mean())

```

`输出:`

```py
Mean ROUGE-1 Score for test threads 0.3439714323225145

```

我们看到，测试集中所有主题的平均 ROUGE-1 分数为 0.34，与其他[公共摘要任务](https://oreil.ly/SaCk2)上的抽取式摘要分数相当。您还会注意到排行榜上使用预训练模型如 BERT 改善了分数，我们在[第 11 章](ch11.xhtml#ch-sentiment)中详细探讨了这一技术。

```py
random.seed(2)
random.sample(test_df['ThreadID'].unique().tolist(), 1)

```

`Out:`

```py
['60974_588_2180141']

```

让我们也来看看由这个模型生成的一个摘要结果，以了解它可能有多有用：

```py
example_df = test_df[test_df['ThreadID'] == '60974_588_2180141']
print('Total number of posts', example_df['postNum'].max())
print('Number of summary posts',
      example_df[example_df['predictedSummaryPost']].count().values[0])
print('Title: ', example_df['Title'].values[0])
example_df[['postNum', 'text']][example_df['predictedSummaryPost']]

```

`Out:`

```py
Total number of posts 9
Number of summary posts 2
Title:  What's fun for kids?

```

|   | postNum | text |
| --- | --- | --- |
| 551 | 4 | 看来你真的很幸运，因为有很多事情可以做，包括艾尔姆伍德艺术节（http://www.elmwoodartfest.org），为年轻人准备的特别活动，表演（包括我最喜欢的本地歌手之一尼基·希克斯的表演），以及各种美食。艾尔姆伍德大道是该地区最丰富多彩且充满活力的社区之一，非常适合步行。布法罗爱尔兰文化节也将在汉堡的周末举行，正好在展览会场地：www.buf... |
| 552 | 5 | 根据您的时间安排，快速到尼亚加拉大瀑布旅行是一个很好的选择。从汉堡开车45分钟，非常值得投资时间。否则，您可以去安哥拉的一些海滩享受时光。如果女孩们喜欢购物，您可以去加勒利亚购物中心，这是一个非常大的商场。如果您喜欢一个更有特色的下午，可以在艾尔布赖特诺艺术画廊午餐，漫步艾尔姆伍德大道，然后逛逛一些时尚店铺，这将是一个很酷的下午。达里恩湖主题公园距离... |

在前面的例子中，原始主题包括九个帖子，其中两个被选出来总结主题，如前所示。阅读总结帖子显示，主题是关于年轻人的活动，已经有一些具体建议，比如艾尔姆伍德大道，达里恩湖主题公园等。想象一下，在浏览论坛搜索结果时，鼠标悬停时提供这些信息。这为用户提供了足够准确的摘要，以决定是否有趣并单击获取更多详细信息或继续查看其他搜索结果。您还可以轻松地将此蓝图与其他数据集重新使用，如开头所述，并自定义距离函数，引入附加功能，然后训练模型。

# 结尾语

在本章中，我们介绍了文本摘要的概念，并提供了可用于为不同用例生成摘要的蓝图。如果您希望从诸如网页、博客和新闻文章等短文本生成摘要，则基于LSA摘要器的主题表示的第一个蓝图将是一个不错的选择。如果您处理的文本更大，例如演讲稿、书籍章节或科学文章，则使用TextRank的蓝图将是一个更好的选择。这些蓝图作为您迈向自动文本摘要的第一步非常棒，因为它们简单又快速。然而，使用机器学习的第三个蓝图为您的特定用例提供了更定制的解决方案。只要您拥有必要的标注数据，就可以通过添加特征和优化机器学习模型来定制此方法以提高性能。例如，您的公司或产品可能有多个管理用户数据、条款和条件以及其他流程的政策文件，您希望为新用户或员工总结这些文件的重要内容。您可以从第三个蓝图开始，并通过添加特征（例如从句数量、使用块字母、是否存在粗体或下划线文本等）来定制第二步，以帮助模型总结政策文件中的重要要点。

# 进一步阅读

+   Allahyari, Mehdi等人提出了一份关于文本摘要技术的简要调查。[*https://arxiv.org/pdf/1707.02268.pdf*](https://arxiv.org/pdf/1707.02268.pdf)。

+   Bhatia, Sumit等人提出了一种总结在线论坛讨论的方法——个体消息的对话行为是否有助于？[*http://sumitbhatia.net/papers/emnlp14.pdf*](http://sumitbhatia.net/papers/emnlp14.pdf)。

+   Collins, Ed等人提出了一种对科学论文进行提取式摘要的监督方法。[*https://arxiv.org/pdf/1706.03946.pdf*](https://arxiv.org/pdf/1706.03946.pdf)。

+   Tarnpradab, Sansiri等人提出了一种通过层次注意力网络实现在线论坛讨论摘要的方法。[*https://aaai.org/ocs/index.php/FLAIRS/FLAIRS17/paper/viewFile/15500/14945*](https://aaai.org/ocs/index.php/FLAIRS/FLAIRS17/paper/viewFile/15500/14945)。

^([1](ch09.xhtml#idm45634184456856-marker)) 您可以在[GitHub](https://oreil.ly/I0FMA)上找到有关该软件包的更多信息，包括我们在设计此蓝图时使用的使用指南。

^([2](ch09.xhtml#idm45634183488696-marker)) Sansiri Tarnpradab等人提出了一种通过层次注意力网络实现在线论坛讨论摘要的方法。[*https://arxiv.org/abs/1805.10390*](https://arxiv.org/abs/1805.10390)。也可以查看[数据集（*.zip*）](https://oreil.ly/cqU_O)。
