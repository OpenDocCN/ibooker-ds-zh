# 第六章：使用 LDA 和 Spark NLP 理解维基百科

近年来，随着非结构化文本数据的增长，获取相关和期望的信息变得困难。语言技术提供了强大的方法，可以用来挖掘文本数据并获取我们所寻找的信息。在本章中，我们将使用 PySpark 和 Spark NLP（自然语言处理）库使用一种这样的技术——主题建模。具体来说，我们将使用潜在狄利克雷算法（LDA）来理解维基百科文档的数据集。

*主题建模*，是自然语言处理中最常见的任务之一，是一种用于数据建模的统计方法，有助于发现文档集合中存在的潜在主题。从数百万个文档中提取主题分布在许多方面都很有用，例如，识别某个产品或所有产品投诉的原因，或在新闻文章中识别主题。主题建模的最流行算法是 LDA。它是一个生成模型，假设文档由主题分布表示。主题本身由词语分布表示。PySpark MLlib 提供了一个专为分布式环境设计的优化版本的 LDA。我们将使用 Spark NLP 对数据进行预处理，并使用 Spark MLlib 的 LDA 构建一个简单的主题建模管道来从数据中提取主题。

在本章中，我们将着手于根据潜在（隐藏）主题和关系来提炼人类知识的谦虚任务。我们将应用 LDA 到包含在维基百科中的文章语料库中。我们将从理解 LDA 的基础知识开始，并在 PySpark 中实施它。然后，我们将下载数据集，并通过安装 Spark NLP 来设置我们的编程环境。这将紧随数据预处理。你将见证到 Spark NLP 库提供的开箱即用方法的强大功能，这使得 NLP 任务变得更加容易。

然后，我们将使用 TF-IDF（词频-逆文档频率）技术对我们文档中的术语进行评分，并将结果输出到我们的 LDA 模型中。最后，我们将浏览模型分配给输入文档的主题。我们应该能够理解一个条目属于哪个桶，而无需阅读它。让我们首先复习一下 LDA 的基础知识。

# 潜在狄利克雷分配

潜在狄利克雷分配背后的理念是，文档是基于一组主题生成的。在这个过程中，我们假设每个文档在主题上分布，每个主题在一组术语上分布。每个文档和每个词都是从这些分布中抽样生成的。LDA 学习者向后工作，并试图识别观察到的最有可能的分布。

它试图将语料库提炼为一组相关的*主题*。每个主题捕捉数据中的一个变化线索，通常对应语料库讨论的一个想法。一个文档可以是多个主题的一部分。你可以把 LDA 看作是一种*软聚类*文档的方式。不深入数学细节，LDA 主题模型描述了两个主要属性：在抽样特定文档时选择主题的机会，以及在选择主题时选择特定术语的机会。例如，LDA 可能会发现一个与术语“阿西莫夫”和“机器人”强相关的主题，并与文档“基地系列”和“科幻小说”相关联。通过仅选择最重要的概念，LDA 可以丢弃一些不相关的噪音并合并共现的线索，以得到数据的更简单的表现形式。

我们可以在各种任务中应用这种技术。例如，当提供输入条目时，它可以帮助我们推荐类似的维基百科条目。通过封装语料库中的变异模式，它可以基于比简单计算单词出现和共现更深的理解来打分。接下来，让我们看一下 PySpark 的 LDA 实现。

## PySpark 中的 LDA

PySpark MLlib 提供了 LDA 实现作为其聚类算法之一。以下是一些示例代码：

```
from pyspark.ml.linalg import Vectors
from pyspark.ml.clustering import LDA

df = spark.createDataFrame([[1, Vectors.dense([0.0, 1.0])],
      [2, Vectors.dense([2.0, 3.0])],],
      ["id", "features"])

lda = LDA(k=2, seed=1) ![1](img/1.png)
lda.setMaxIter(10)

model = lda.fit(df)

model.vocabSize()
2

model.describeTopics().show() ![2](img/2.png)
+-----+-----------+--------------------+
|topic|termIndices|         termWeights|
+-----+-----------+--------------------+
|    0|     [0, 1]|[0.53331100994293...|
|    1|     [1, 0]|0.50230220117597...|
+-----+-----------+--------------------+
```

![1 我们将 LDA 应用于数据帧，主题数(*k*)设定为 2。![2](img/#co_understanding_wikipedia___span_class__keep_together__with_lda_and_spark_nlp__span__CO1-2)

描述我们主题中每个术语关联的概率权重的数据帧。

我们将探索 PySpark 的 LDA 实现及其应用于维基百科数据集时相关的参数。不过首先，我们需要下载相关的数据集。接下来我们将做这件事。

# 获取数据

Wikipedia 提供其所有文章的 dump。完整的 dump 以单个大 XML 文件的形式出现。这些可以被[下载](https://oreil.ly/DhGlJ)，然后放置在云存储解决方案（如 AWS S3 或 GCS，Google Cloud Storage）或 HDFS 上。例如：

```
$ curl -s -L https://dumps.wikimedia.org/enwiki/latest/\
$ enwiki-latest-pages-articles-multistream.xml.bz2 \
$   | bzip2 -cd \
$   | hadoop fs -put - wikidump.xml
```

这可能需要一些时间。

处理这些数据量最合理的方式是使用几个节点的集群进行。要在本地机器上运行本章的代码，更好的选择是使用[Wikipedia 导出页面](https://oreil.ly/Rrpmr)生成一个较小的 dump。尝试获取多个页面数多且子类别少的多个类别，例如生物学、健康和几何学。为了使下面的代码起作用，请将 dump 下载到*ch06-LDA/*目录并将其重命名为*wikidump.xml*。

我们需要将 Wikipedia XML 转储文件转换为 PySpark 轻松处理的格式。在本地机器上工作时，我们可以使用便捷的 [WikiExtractor 工具](https://oreil.ly/pfwrE)。它从 Wikipedia 数据库转储中提取和清理文本，例如我们所拥有的。

使用 pip 安装它：

```
$ pip3 install wikiextractor
```

然后，只需在包含下载文件的目录中运行以下命令：

```
$ wikiextractor wikidump.xml
```

输出存储在名为`text`的给定目录中的一个或多个文件中。每个文件将以以下格式包含多个文档：

```
$ mv text wikidump ![1](img/1.png)
$ tree wikidump
...
wikidump
└── AA
    └── wiki_00

...
$ head -n 5 wikidump/AA/wiki_00
...

<doc id="18831" url="?curid=18831" title="Mathematics">
Mathematics

Mathematics (from Greek: ) includes the study of such topics as numbers ...
...
```

![1](img/#co_understanding_wikipedia___span_class__keep_together__with_lda_and_spark_nlp__span__CO2-1)

将 text 目录重命名为 wikidump

接下来，让我们在开始处理数据之前熟悉 Spark NLP 库。

# Spark NLP

Spark NLP 库最初由 [John Snow Labs](https://oreil.ly/E9KVt) 于 2017 年初设计，作为一种本地于 Spark 的注释库，以充分利用 Spark SQL 和 MLlib 模块的功能。灵感来自于尝试使用 Spark 分发其他 NLP 库，这些库通常没有考虑并发性或分布式计算。

Spark NLP 具有与任何其他注释库相同的概念，但在注释存储方式上有所不同。大多数注释库将注释存储在文档对象中，但 Spark NLP 为不同类型的注释创建列。注释器实现为转换器、估计器和模型。在下一节中，当我们将它们应用于我们的数据集进行预处理时，我们将查看它们。在此之前，让我们在系统上下载并设置 Spark NLP。

## 设置您的环境：

通过 pip 安装 Spark NLP：

```
pip3 install spark-nlp==3.2.3
```

启动 PySpark shell：

```
pyspark --packages com.johnsnowlabs.nlp:spark-nlp_2.12:3.4.4
```

让我们在 PySpark shell 中导入 Spark NLP：

```
import sparknlp

spark = sparknlp.start()
```

现在，您可以导入我们将使用的相关 Spark NLP 模块：

```
from sparknlp.base import DocumentAssembler, Finisher
from sparknlp.annotator import (Lemmatizer, Stemmer,
                                Tokenizer, Normalizer,
                                StopWordsCleaner)
from sparknlp.pretrained import PretrainedPipeline
```

现在我们已经设置好了编程环境，让我们开始处理我们的数据集。我们将从将数据解析为 PySpark DataFrame 开始。

# 解析数据

WikiExtractor 的输出可能会根据输入转储的大小创建多个目录。我们希望将所有数据导入为单个 DataFrame。让我们从指定的输入目录开始：

```
data_source = 'wikidump/*/*'
```

我们使用 `sparkContext` 可访问的 `wholeTextFiles` 方法导入数据。该方法将数据读取为 RDD。我们将其转换为 DataFrame，因为这是我们想要的：

```
raw_data = spark.sparkContext.wholeTextFiles(data_source).toDF()
raw_data.show(1, vertical=True)
...

-RECORD 0-------------------
 _1  | file:/home/analyt...
 _2  | <doc id="18831" u...
```

结果 DataFrame 将由两列组成。记录的数量将对应于已读取的文件数量。第一列包含文件路径，第二列包含相应的文本内容。文本包含多个条目，但我们希望每行对应一个单独的条目。基于我们之前看到的条目结构，我们可以使用几个 PySpark 工具：`split` 和 `explode` 来分隔条目。

```
from pyspark.sql import functions as fun
df = raw_data.withColumn('content', fun.explode(fun.split(fun.col("_2"),
  "</doc>")))
df = df.drop(fun.col('_2')).drop(fun.col('_1'))

df.show(4, vertical=True)
...
-RECORD 0-----------------------
 content | <doc id="18831" u...
-RECORD 1-----------------------
 content |
<doc id="5627588...
-RECORD 2-----------------------
 content |
<doc id="3354393...
-RECORD 3-----------------------
 content |
<doc id="5999808...
only showing top 4 rows
```

`split` 函数用于根据提供的模式将 DataFrame 字符串 `Column` 拆分为数组。在之前的代码中，我们根据 *</doc>* 字符串将组合的文档 XML 字符串拆分为数组。这实际上为我们提供了一个包含多个文档的数组。然后，我们使用 `explode` 将根据 `split` 函数返回的数组中的每个元素创建新行。这导致为每个文档创建相应的行。

通过我们之前操作的 `content` 列获取的结构进行遍历：

```
df.show(1, truncate=False, vertical=True)
...
-RECORD 0

-------------------------------------------------------------------
 content | <doc id="18831" url="?curid=18831" title="Mathematics">
Mathematics

Mathematics (from Greek: ) includes the study of such topics as numbers...
```

我们可以通过提取条目的标题来进一步拆分我们的 `content` 列：

```
df = df.withColumn('title', fun.split(fun.col('content'), '\n').getItem(2)) \
        .withColumn('content', fun.split(fun.col('content'), '\n').getItem(4))
df.show(4, vertical=True)
...
-RECORD 0-----------------------
 content | In mathematics, a...
 title   | Tertiary ideal
-RECORD 1-----------------------
 content | In algebra, a bin...
 title   | Binomial (polynom...
-RECORD 2-----------------------
 content | Algebra (from ) i...
 title   | Algebra
-RECORD 3-----------------------
 content | In set theory, th...
 title   | Kernel (set theory)
only showing top 4 rows
...
```

现在我们有了解析后的数据集，让我们使用 Spark NLP 进行预处理。

# 使用 Spark NLP 准备数据

我们之前提到，基于文档注释器模型的库（如 Spark NLP）具有“文档”的概念。在 PySpark 中本地不存在这样的概念。因此，Spark NLP 的一个核心设计原则之一是与 MLlib 强大的互操作性。通过提供与 DataFrame 兼容的转换器，将文本列转换为文档，并将注释转换为 PySpark 数据类型。

我们首先通过 `DocumentAssembler` 创建我们的 `document` 列：

```
document_assembler = DocumentAssembler() \
    .setInputCol("content") \
    .setOutputCol("document") \
    .setCleanupMode("shrink")

document_assembler.transform(df).select('document').limit(1).collect()
...

Row(document=[Row(annotatorType='document', begin=0, end=289, result='...',
    metadata={'sentence': '0'}, embeddings=[])])
```

我们可以在解析部分利用 Spark NLP 的 [`DocumentNormalizer`](https://oreil.ly/UL1vp) 注释器。

我们可以像之前的代码中所做的那样直接转换输入的 DataFrame。但是，我们将使用 `DocumentAssembler` 和其他必需的注释器作为 ML 流水线的一部分。

我们将使用以下注释器作为我们的预处理流水线的一部分：`Tokenizer`、`Normalizer`、`StopWordsCleaner` 和 `Stemmer`。

让我们从 `Tokenizer` 开始：

```
# Split sentence to tokens(array)
tokenizer = Tokenizer() \
  .setInputCols(["document"]) \
  .setOutputCol("token")
```

`Tokenizer` 是一个基础注释器。几乎所有基于文本的数据处理都以某种形式的分词开始，这是将原始文本分解成小块的过程。标记可以是单词、字符或子词（n-gram）。大多数经典的自然语言处理算法都期望标记作为基本输入。正在开发许多将字符作为基本输入的深度学习算法。大多数自然语言处理应用程序仍然使用分词。

接下来是 `Normalizer`：

```
# clean unwanted characters and garbage
normalizer = Normalizer() \
    .setInputCols(["token"]) \
    .setOutputCol("normalized") \
    .setLowercase(True)
```

`Normalizer` 清理前一步骤中的标记，并从文本中删除所有不需要的字符。

接下来是 `StopWordsCleaner`：

```
# remove stopwords
stopwords_cleaner = StopWordsCleaner()\
      .setInputCols("normalized")\
      .setOutputCol("cleanTokens")\
      .setCaseSensitive(False)
```

这个注释器从文本中删除*停用词*。停用词如“the”、“is”和“at”非常常见，可以在不显著改变文本含义的情况下删除。从文本中删除停用词在希望处理只有最重要的语义单词而忽略文章和介词等很少有语义相关性的词时非常有用。

最后是 `Stemmer`：

```
# stem the words to bring them to the root form.
stemmer = Stemmer() \
    .setInputCols(["cleanTokens"]) \
    .setOutputCol("stem")
```

`Stemmer` 返回单词的硬干出来，目的是检索单词的有意义部分。*Stemming* 是将单词减少到其根词干的过程，目的是检索有意义的部分。例如，“picking”，“picked” 和 “picks” 都以 “pick” 作为根。

我们几乎完成了。在完成我们的 NLP 管道之前，我们需要添加`Finisher`。当我们将数据帧中的每一行转换为文档时，Spark NLP 添加了自己的结构。`Finisher` 非常关键，因为它帮助我们恢复预期的结构，即一个令牌数组：

```
finisher = Finisher() \
    .setInputCols(["stem"]) \
    .setOutputCols(["tokens"]) \
    .setOutputAsArray(True) \
    .setCleanAnnotations(False)
```

现在我们已经准备好所有必需的组件。让我们构建我们的管道，以便每个阶段可以按顺序执行：

```
from pyspark.ml import Pipeline
nlp_pipeline = Pipeline(
    stages=[document_assembler,
            tokenizer,
            normalizer,
            stopwords_cleaner,
            stemmer,
            finisher])
```

执行管道并转换数据帧：

```
nlp_model = nlp_pipeline.fit(df) ![1](img/1.png)

processed_df  = nlp_model.transform(df) ![2](img/2.png)

processed_df.printSchema()
...

root
 |-- content: string (nullable = true)
 |-- title: string (nullable = true)
 |-- document: array (nullable = true)
 |    |-- element: struct (containsNull = true)
 |    |    |-- annotatorType: string (nullable = true)
 |    |    |-- begin: integer (nullable = false)
 |    |    |-- end: integer (nullable = false)
 |    |    |-- result: string (nullable = true)
 |    |    |-- metadata: map (nullable = true)
 |    |    |    |-- key: string
 |    |    |    |-- value: string (valueContainsNull = true)
 |    |    |-- embeddings: array (nullable = true)
 |    |    |    |-- element: float (containsNull = false)
 |-- token: array (nullable = true)
 |    |-- element: struct (containsNull = true)
 |    |    |-- annotatorType: string (nullable = true)
 |    |    |-- begin: integer (nullable = false)
 |    |    |-- end: integer (nullable = false)
 |    |    |-- result: string (nullable = true)
 |    |    |-- metadata: map (nullable = true)
 |    |    |    |-- key: string
 |    |    |    |-- value: string (valueContainsNull = true)
 |    |    |-- embeddings: array (nullable = true)
 |    |    |    |-- element: float (containsNull = false)
 |-- normalized: array (nullable = true)
 |    |-- element: struct (containsNull = true)
 |    |    |-- annotatorType: string (nullable = true)
 |    |    |-- begin: integer (nullable = false)
 |    |    |-- end: integer (nullable = false)
 |    |    |-- result: string (nullable = true)
 |    |    |-- metadata: map (nullable = true)
 |    |    |    |-- key: string
 |    |    |    |-- value: string (valueContainsNull = true)
 |    |    |-- embeddings: array (nullable = true)
 |    |    |    |-- element: float (containsNull = false)
 |-- cleanTokens: array (nullable = true)
 |    |-- element: struct (containsNull = true)
 |    |    |-- annotatorType: string (nullable = true)
 |    |    |-- begin: integer (nullable = false)
 |    |    |-- end: integer (nullable = false)
 |    |    |-- result: string (nullable = true)
 |    |    |-- metadata: map (nullable = true)
 |    |    |    |-- key: string
 |    |    |    |-- value: string (valueContainsNull = true)
 |    |    |-- embeddings: array (nullable = true)
 |    |    |    |-- element: float (containsNull = false)
 |-- stem: array (nullable = true)
 |    |-- element: struct (containsNull = true)
 |    |    |-- annotatorType: string (nullable = true)
 |    |    |-- begin: integer (nullable = false)
 |    |    |-- end: integer (nullable = false)
 |    |    |-- result: string (nullable = true)
 |    |    |-- metadata: map (nullable = true)
 |    |    |    |-- key: string
 |    |    |    |-- value: string (valueContainsNull = true)
 |    |    |-- embeddings: array (nullable = true)
 |    |    |    |-- element: float (containsNull = false)
 |-- tokens: array (nullable = true)
 |    |-- element: string (containsNull = true)
```

![1](img/#co_understanding_wikipedia___span_class__keep_together__with_lda_and_spark_nlp__span__CO3-1)

训练管道。

![2](img/#co_understanding_wikipedia___span_class__keep_together__with_lda_and_spark_nlp__span__CO3-2)

将管道应用于转换数据帧。

NLP 管道创建了我们不需要的中间列。让我们删除这些冗余列：

```
tokens_df = processed_df.select('title','tokens')
tokens_df.show(2, vertical=True)
...

-RECORD 0----------------------
 title  | Tertiary ideal
 tokens | mathemat, tertia...
-RECORD 1----------------------
 title  | Binomial (polynom...
 tokens | algebra, binomi,...
only showing top 2 rows
```

接下来，我们将了解 TF-IDF 的基础知识，并在预处理数据集 `token_df` 上实施它，该数据集在构建 LDA 模型之前已经获得。

# TF-IDF

在应用 LDA 之前，我们需要将数据转换为数字表示。我们将使用术语频率-逆文档频率方法来获得这样的表示。大致来说，TF-IDF 用于确定与给定文档对应的术语的重要性。这里是 Python 代码中该公式的表示。我们实际上不会使用这段代码，因为 PySpark 提供了自己的实现。

```
import math

def term_doc_weight(term_frequency_in_doc, total_terms_in_doc,
                    term_freq_in_corpus, total_docs):
  tf = term_frequency_in_doc / total_terms_in_doc
  doc_freq = total_docs / term_freq_in_corpus
  idf = math.log(doc_freq)
  tf * idf
}
```

TF-IDF 捕捉了有关术语与文档相关性的两种直觉。首先，我们预期一个术语在文档中出现的次数越多，它对该文档的重要性就越大。其次，在全局意义上，并非所有术语都是相等的。在整个语料库中很少出现的词比大多数文档中出现的词更有意义；因此，该度量使用术语在整个语料库中出现的逆数。 

语料库中单词的频率往往呈指数分布。一个常见的词可能出现的次数是一个稍微常见的词的十倍，而后者可能出现的次数是一个稀有词的十到一百倍。基于原始逆文档频率的度量会赋予稀有词巨大的权重，并几乎忽略所有其他词的影响。为了捕捉这种分布，该方案使用逆文档频率的*对数*。这通过将它们之间的乘法差距转换为加法差距，使文档频率之间的差异变得温和。

该模型依赖于一些假设。它将每个文档视为“词袋”，即不关注单词的顺序、句子结构或否定。通过表示每个术语一次，该模型难以处理 *一词多义*，即同一个词具有多个含义的情况。例如，模型无法区分“Radiohead is the best band ever” 和 “I broke a rubber band” 中的“band” 的使用。如果这两个句子在语料库中频繁出现，它可能会将“Radiohead” 与“rubber” 关联起来。

现在我们继续实现使用 PySpark 的 TF-IDF。

# 计算 TF-IDFs

首先，我们将使用 `CountVectorizer` 计算 TF（术语频率；即文档中每个术语的频率），它会跟踪正在创建的词汇表，以便我们可以将我们的主题映射回相应的单词。TF 创建一个矩阵，统计词汇表中每个词在每个文本主体中出现的次数。然后，根据其频率给每个词赋予一个权重。我们在拟合时获取数据的词汇表并在转换步骤中获取计数：

```
from pyspark.ml.feature import CountVectorizer
cv = CountVectorizer(inputCol="tokens", outputCol="raw_features")

# train the model
cv_model = cv.fit(tokens_df)

# transform the data. Output column name will be raw_features.
vectorized_tokens = cv_model.transform(tokens_df)
```

然后，我们进行 IDF（文档中包含某个术语的反向频率），它会减少常见术语的权重：

```
from pyspark.ml.feature import IDF
idf = IDF(inputCol="raw_features", outputCol="features")

idf_model = idf.fit(vectorized_tokens)

vectorized_df = idf_model.transform(vectorized_tokens)
```

结果将如下所示：

```
vectorized_df = vectorized_df.drop(fun.col('raw_features'))

vectorized_df.show(6)
...

+--------------------+--------------------+--------------------+
|               title|              tokens|            features|
+--------------------+--------------------+--------------------+
|      Tertiary ideal|[mathemat, tertia...|(2451,[1,6,43,56,...|
|Binomial (polynom...|[algebra, binomi,...|(2451,[0,10,14,34...|
|             Algebra|[algebra, on, bro...|(2451,[0,1,5,6,15...|
| Kernel (set theory)|[set, theori, ker...|(2451,[2,3,13,19,...|
|Generalized arith...|[mathemat, gener,...|(2451,[1,2,6,45,4...|
+--------------------+--------------------+--------------------+
```

在所有预处理和特征工程完成后，我们现在可以创建我们的 LDA 模型。这将在下一节中进行。

# 创建我们的 LDA 模型

我们之前提到过，LDA 将语料库提炼为一组相关主题。我们将在本节后面进一步查看此类主题的示例。在此之前，我们需要决定我们的 LDA 模型所需的两个超参数。它们是主题数量（称为 *k*）和迭代次数。

有多种方法可以选择 *k*。用于此目的的两种流行指标是困惑度和主题一致性。前者由 PySpark 的实现提供。基本思想是试图找出这些指标改善开始变得不显著的 *k*。如果您熟悉 K-means 的寻找簇数的“拐点法”，这类似。根据语料库的大小，这可能是一个资源密集和耗时的过程，因为您需要为多个 *k* 值构建模型。另一个选择可能是尝试创建数据集的代表性样本，并使用它来确定 *k*。您可以阅读相关资料并尝试这一点。

由于您现在可能正在本地工作，我们将暂时设定合理的值（*k* 为 5，`max_iter` 为 50）。

让我们创建我们的 LDA 模型：

```
from pyspark.ml.clustering import LDA

num_topics = 5
max_iter = 50

lda = LDA(k=num_topics, maxIter=max_iter)
model = lda.fit(vectorized_df)

lp = model.logPerplexity(vectorized_df)

print("The upper bound on perplexity: " + str(lp))
...

The upper bound on perplexity: 6.768323190833805
```

困惑度是衡量模型对样本预测能力的指标。低困惑度表明概率分布在预测样本方面表现良好。在比较不同模型时，选择困惑度值较低的模型。

现在我们已经创建了我们的模型，我们想将主题输出为人类可读的形式。我们将从我们的预处理步骤中生成的词汇表中获取词汇表，从 LDA 模型中获取主题，并将它们映射起来。

```
vocab = cv_model.vocabulary ![1raw_topics = model.describeTopics().collect() ![2

topic_inds = [ind.termIndices for ind in raw_topics] ![3](img/3.png)

topics = []
for topic in topic_inds:
    _topic = []
    for ind in topic:
        _topic.append(vocab[ind])
    topics.append(_topic)
```

![1](img/#co_understanding_wikipedia___span_class__keep_together__with_lda_and_spark_nlp__span__CO4-1)

创建对我们词汇表的引用。

![2](img/#co_understanding_wikipedia___span_class__keep_together__with_lda_and_spark_nlp__span__CO4-2)

使用`describeTopics`从 LDA 模型获取生成的主题，并将它们加载到 Python 列表中。

![3](img/#co_understanding_wikipedia___span_class__keep_together__with_lda_and_spark_nlp__span__CO4-3)

从我们的主题中获取词汇表术语的索引。

现在让我们生成从我们的主题索引到我们的词汇表的映射：

```
for i, topic in enumerate(topics, start=1):
    print(f"topic {i}: {topic}")
...

topic 1: 'islam', 'health', 'drug', 'empir', 'medicin', 'polici',...
topic 2: ['formula', 'group', 'algebra', 'gener', 'transform',    ...
topic 3: ['triangl', 'plane', 'line', 'point', 'two', 'tangent',  ...
topic 4: ['face', 'therapeut', 'framework', 'particl', 'interf',  ...
topic 5: ['comput', 'polynomi', 'pattern', 'internet', 'network', ...
```

上述结果并不完美，但在主题中可以注意到一些模式。主题 1 主要与健康有关。它还包含对伊斯兰教和帝国的引用。这可能是因为它们在医学历史中被引用，反之亦然，或者其他原因？主题 2 和 3 与数学有关，后者倾向于几何学。主题 5 是计算和数学的混合体。即使您没有阅读任何文档，您也可以以合理的准确度猜测它们的类别。这很令人兴奋！

现在我们还可以检查我们的输入文档与哪些主题最相关。单个文档可以具有多个显著的主题关联。目前，我们只会查看与之最相关的主题。

让我们在我们的输入数据帧上运行 LDA 模型的转换操作：

```
lda_df = model.transform(vectorized_df)
lda_df.select(fun.col('title'), fun.col('topicDistribution')).\
                show(2, vertical=True, truncate=False)
...
-RECORD 0-------------------------------------
 title             | Tertiary ideal
 topicDistribution | [5.673953573608612E-4,...
-RECORD 1----------------------------------...
 title             | Binomial (polynomial) ...
 topicDistribution | [0.0019374384060205127...
only showing top 2 rows
```

正如您所看到的，每个文档都与其相关的主题概率分布相关联。为了获取每个文档的相关主题，我们想找出具有最高概率得分的主题索引。然后，我们可以将其映射到之前获取的主题。

我们将编写一个 PySpark UDF，以查找每条记录的最高主题概率分数：

```
from pyspark.sql.types import IntegerType
max_index = fun.udf(lambda x: x.tolist().index(max(x)) + 1, IntegerType())
lda_df = lda_df.withColumn('topic_index',
                        max_index(fun.col('topicDistribution')))
```

```
lda_df.select('title', 'topic_index').show(10, truncate=False)
...

+----------------------------------+-----------+
|title                             |topic_index|
+----------------------------------+-----------+
|Tertiary ideal                    |2          |
|Binomial (polynomial)             |2          |
|Algebra                           |2          |
|Kernel (set theory)               |2          |
|Generalized arithmetic progression|2          |
|Schur algebra                     |2          |
|Outline of algebra                |2          |
|Recurrence relation               |5          |
|Rational difference equation      |5          |
|Polynomial arithmetic             |2          |
+----------------------------------+-----------+
only showing top 11 rows
```

Topic 2，如果你还记得，与数学有关。输出结果符合我们的预期。您可以扫描更多数据集，查看其表现。您可以使用`where`或`filter`命令选择特定主题，并与之前生成的主题列表进行比较，以更好地理解已创建的聚类。正如本章开头所承诺的那样，我们能够将文章分成不同的主题，而无需阅读它们！

# 接下来的步骤

在本章中，我们对维基百科语料库执行了 LDA。在这个过程中，我们还学习了使用令人惊叹的 Spark NLP 库和 TF-IDF 技术进行文本预处理。您可以通过改进模型的预处理和超参数调整来进一步构建这一过程。此外，当用户提供输入时，您甚至可以尝试基于文档相似性推荐类似条目。这样的相似性度量可以通过使用从 LDA 获得的概率分布向量来获得。

此外，还存在多种其他方法来理解大型文本语料库。例如，被称为潜在语义分析（LSA）的技术在类似的应用中非常有用，并且在本书的上一版本中也使用了相同数据集。深度学习，在[第十章中进行了探讨，也提供了进行主题建模的途径。您可以自行探索这些方法。
