# 第六章：用 LDA 和 Spark NLP 理解维基百科

随着近年来非结构化文本数据量的增加，获取相关和所需信息变得困难。语言技术提供了强大的方法，可以用来挖掘文本数据，并获取我们正在寻找的信息。在本章中，我们将使用 PySpark 和 Spark NLP（自然语言处理）库来使用一种这样的技术——主题建模。具体而言，我们将使用潜在狄利克雷算法（LDA）来理解维基百科文档数据集。

*主题建模*是自然语言处理中最常见的任务之一，是一种用于数据建模的统计方法，帮助发现文档集合中存在的潜在主题。从数百万篇文档中提取主题分布可以在许多方面有用，例如识别特定产品或所有产品投诉的原因，或在新闻文章中识别主题。主题建模的最流行算法是 LDA。它是一种生成模型，假设文档由主题分布表示。而主题本身则由词汇分布表示。PySpark MLlib 提供了一种优化的 LDA 版本，专门设计用于分布式环境中运行。我们将使用 Spark NLP 预处理数据，并使用 Spark MLlib 的 LDA 从数据中提取主题，构建一个简单的主题建模流水线。

在本章中，我们将开始一个谦虚的任务，即基于潜在的主题和关系来提炼人类知识。我们将应用 LDA 到由维基百科文章组成的语料库中。我们将从理解 LDA 的基础知识开始，并在 PySpark 中实现它。然后，我们将下载数据集，并通过安装 Spark NLP 来设置我们的编程环境。接下来是数据预处理。您将见证 Spark NLP 库的开箱即用方法的强大，这使得自然语言处理任务变得更加容易。

然后，我们将使用 TF-IDF（词频-逆文档频率）技术对我们的文档中的术语进行评分，并将结果输出到我们的 LDA 模型中。最后，我们将通过我们的模型分配给输入文档的主题来完成。我们应该能够理解一个条目属于哪个桶，而无需阅读它。让我们从回顾 LDA 的基础知识开始。

# 潜在狄利克雷分配

潜在狄利克雷分配背后的想法是，文档是基于一组主题生成的。在这个过程中，我们假设每个文档在主题上分布，并且每个主题在一组词上分布。每个文档和每个单词都是从这些分布中采样生成的。LDA 学习者反向工作，并尝试识别观察到的最可能的分布。

它试图将语料库提炼成一组相关的*主题*。每个主题捕获数据中的一个变化线索，通常对应于语料库讨论的概念之一。一个文档可以属于多个主题。您可以将 LDA 视为提供一种*软聚类*文档的方法。不深入数学细节，LDA 主题模型描述两个主要属性：在对特定文档进行采样时选择主题的机会，以及在选择主题时选择特定术语的机会。例如，LDA 可能会发现一个与术语“Asimov”和“robot”强相关的主题，并与“基础系列”和“科幻小说”等文档相关联。通过仅选择最重要的概念，LDA 可以丢弃一些无关的噪音并合并共现的线索，从而得出数据的更简单表示。

我们可以在各种任务中使用这种技术。例如，它可以帮助我们在提供输入条目时推荐类似的维基百科条目。通过封装语料库中的变化模式，它可以基于比仅仅计数单词出现和共现更深入的理解进行评分。接下来，让我们来看看 PySpark 的 LDA 实现。

## PySpark 中的 LDA

PySpark MLlib 提供了 LDA 实现作为其聚类算法之一。以下是一些示例代码：

```py
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

![1 我们将在我们的数据框上应用 LDA，主题数（*k*）设为 2。![2](img/#co_understanding_wikipedia___span_class__keep_together__with_lda_and_spark_nlp__span__CO1-2)

数据框描述了我们主题中每个术语关联的概率权重。

我们将探讨 PySpark 的 LDA 实现以及应用到维基百科数据集时相关的参数。不过，首先我们需要下载相关的数据集。这是我们接下来要做的事情。

# 获取数据

维基百科提供其所有文章的数据备份。完整的备份以单个大型 XML 文件形式提供。这些可以通过[下载](https://oreil.ly/DhGlJ)然后放置在云存储解决方案（如 AWS S3 或 GCS，Google Cloud Storage）或 HDFS 上。例如：

```py
$ curl -s -L https://dumps.wikimedia.org/enwiki/latest/\
$ enwiki-latest-pages-articles-multistream.xml.bz2 \
$   | bzip2 -cd \
$   | hadoop fs -put - wikidump.xml
```

这可能需要一些时间。

处理这么大量的数据最好使用一个包含几个节点的集群来完成。要在本地机器上运行本章的代码，一个更好的选择是使用[Wikipedia 的导出页面](https://oreil.ly/Rrpmr)生成一个较小的数据集。尝试获取来自多个具有许多页面和少量子类别的类别（例如生物学、健康和几何学）的所有页面。为了使下面的代码运行起来，请下载这个数据集到*ch06-LDA/*目录，并将其重命名为*wikidump.xml*。

我们需要将维基百科的 XML 转储转换为我们可以在 PySpark 中轻松使用的格式。在本地机器上工作时，我们可以使用方便的[WikiExtractor 工具](https://oreil.ly/pfwrE)来提取和清理文本。它从维基百科数据库转储中提取和清理文本，正如我们所拥有的那样。

使用 pip 安装它：

```py
$ pip3 install wikiextractor
```

然后，在包含下载文件的目录中运行以下命令就很简单了：

```py
$ wikiextractor wikidump.xml
```

输出存储在名为`text`的给定目录中的一个或多个大小相似的文件中。每个文件将以以下格式包含多个文档：

```py
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

重命名文本目录为 wikidump

接下来，在开始处理数据之前，让我们先熟悉一下 Spark NLP 库。

# Spark NLP

Spark NLP 库最初由[John Snow Labs](https://oreil.ly/E9KVt)于 2017 年初设计，作为 Spark 原生的注释库，以充分利用 Spark SQL 和 MLlib 模块的优势。灵感来自于尝试使用 Spark 来分发其他通常不考虑并发或分布式计算的 NLP 库。

Spark NLP 与任何其他注释库具有相同的概念，但在存储注释的方式上有所不同。大多数注释库将注释存储在文档对象中，但 Spark NLP 为不同类型的注释创建列。注释器实现为转换器、估计器和模型。在下一节中，当我们将它们应用于预处理我们的数据集时，我们将查看这些内容。在此之前，让我们在系统上下载和设置 Spark NLP。

## 设置您的环境

通过 pip 安装 Spark NLP：

```py
pip3 install spark-nlp==3.2.3
```

启动 PySpark shell：

```py
pyspark --packages com.johnsnowlabs.nlp:spark-nlp_2.12:3.4.4
```

让我们在 PySpark shell 中导入 Spark NLP：

```py
import sparknlp

spark = sparknlp.start()
```

现在，您可以导入我们将使用的相关 Spark NLP 模块：

```py
from sparknlp.base import DocumentAssembler, Finisher
from sparknlp.annotator import (Lemmatizer, Stemmer,
                                Tokenizer, Normalizer,
                                StopWordsCleaner)
from sparknlp.pretrained import PretrainedPipeline
```

现在我们已经设置好了编程环境，让我们开始处理我们的数据集。我们将从将数据解析为 PySpark DataFrame 开始。

# 解析数据

WikiExtractor 的输出可以根据输入转储的大小创建多个目录。我们希望将所有数据作为单个 DataFrame 导入。让我们首先指定输入目录：

```py
data_source = 'wikidump/*/*'
```

我们使用`sparkContext`访问的`wholeTextFiles`方法导入数据。此方法将数据读取为 RDD。我们将其转换为 DataFrame，因为这正是我们想要的：

```py
raw_data = spark.sparkContext.wholeTextFiles(data_source).toDF()
raw_data.show(1, vertical=True)
...

-RECORD 0-------------------
 _1  | file:/home/analyt...
 _2  | <doc id="18831" u...
```

结果 DataFrame 将包含两列。记录的数量将对应于读取的文件数量。第一列包含文件路径，第二列包含相应的文本内容。文本包含多个条目，但我们希望每行对应一个条目。根据我们早期看到的条目结构，我们可以使用几个 PySpark 实用程序：`split`和`explode`来分隔条目。

```py
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

`split`函数用于根据提供的模式匹配将 DataFrame 字符串`Column`拆分为数组。在前面的代码中，我们根据*</doc>*字符串将组合的文档 XML 字符串拆分为数组。这有效地给我们提供了一个包含多个文档的数组。然后，我们使用`explode`为`split`函数返回的数组中的每个元素创建新行。这导致相应的每个文档创建了行。

通过我们先前操作获取的`content`列中的结构进行遍历：

```py
df.show(1, truncate=False, vertical=True)
...
-RECORD 0

-------------------------------------------------------------------
 content | <doc id="18831" url="?curid=18831" title="Mathematics">
Mathematics

Mathematics (from Greek: ) includes the study of such topics as numbers...
```

我们可以通过提取条目的标题进一步拆分我们的`content`列：

```py
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

现在我们有了解析后的数据集，让我们继续使用 Spark NLP 进行预处理。

# 使用 Spark NLP 准备数据

我们之前提到过，基于文档注释模型的库如 Spark NLP 具有“文档”的概念。在 PySpark 中本地不存在这样的概念。因此，Spark NLP 的核心设计原则之一是与 MLlib 的强大互操作性。这是通过提供 DataFrame 兼容的转换器来将文本列转换为文档，并将注释转换为 PySpark 数据类型来实现的。

我们首先通过`DocumentAssembler`创建我们的`document`列：

```py
document_assembler = DocumentAssembler() \
    .setInputCol("content") \
    .setOutputCol("document") \
    .setCleanupMode("shrink")

document_assembler.transform(df).select('document').limit(1).collect()
...

Row(document=[Row(annotatorType='document', begin=0, end=289, result='...',
    metadata={'sentence': '0'}, embeddings=[])])
```

在解析部分，我们本可以利用 Spark NLP 的[`DocumentNormalizer`](https://oreil.ly/UL1vp)注释器。

我们可以像在前面的代码中那样直接转换输入的数据框架。但是，我们将使用`DocumentAssembler`和其他所需的注释器作为 ML 流水线的一部分。

作为我们预处理流水线的一部分，我们将使用以下注释器：`Tokenizer`，`Normalizer`，`StopWordsCleaner`和`Stemmer`。

让我们从`Tokenizer`开始：

```py
# Split sentence to tokens(array)
tokenizer = Tokenizer() \
  .setInputCols(["document"]) \
  .setOutputCol("token")
```

`Tokenizer`是一个基础注释器。几乎所有基于文本的数据处理都始于某种形式的分词，即将原始文本分解为小块的过程。标记可以是单词、字符或子词（n-gram）。大多数经典的 NLP 算法期望标记作为基本输入。许多正在开发的深度学习算法以字符作为基本输入。大多数 NLP 应用仍然使用分词。

接下来是`Normalizer`：

```py
# clean unwanted characters and garbage
normalizer = Normalizer() \
    .setInputCols(["token"]) \
    .setOutputCol("normalized") \
    .setLowercase(True)
```

`Normalizer`从前面步骤的标记中清除标记，并从文本中删除所有不需要的字符。

接下来是`StopWordsCleaner`：

```py
# remove stopwords
stopwords_cleaner = StopWordsCleaner()\
      .setInputCols("normalized")\
      .setOutputCol("cleanTokens")\
      .setCaseSensitive(False)
```

此注释器从文本中删除*停用词*。像“the,” “is,” 和 “at”这样的停用词是如此常见，它们可以被移除而不会显著改变文本的含义。移除停用词在希望仅处理文本中最语义重要的单词并忽略那些很少语义相关的单词（如冠词和介词）时非常有用。

最后是`Stemmer`：

```py
# stem the words to bring them to the root form.
stemmer = Stemmer() \
    .setInputCols(["cleanTokens"]) \
    .setOutputCol("stem")
```

`Stemmer`返回单词的硬干部，目的是检索单词的有意义部分。*词干提取*是将单词减少到其根单词词干的过程，目的是检索有意义的部分。例如，“picking”，“picked”和“picks”都有“pick”作为根。

我们快要完成了。在完成 NLP 流水线之前，我们需要添加`Finisher`。当我们将数据框中的每一行转换为文档时，Spark NLP 会添加自己的结构。`Finisher`至关重要，因为它帮助我们恢复预期的结构，即一个标记数组：

```py
finisher = Finisher() \
    .setInputCols(["stem"]) \
    .setOutputCols(["tokens"]) \
    .setOutputAsArray(True) \
    .setCleanAnnotations(False)
```

现在我们已经准备就绪。让我们构建我们的流水线，以便每个阶段可以按顺序执行：

```py
from pyspark.ml import Pipeline
nlp_pipeline = Pipeline(
    stages=[document_assembler,
            tokenizer,
            normalizer,
            stopwords_cleaner,
            stemmer,
            finisher])
```

执行流水线并转换数据框：

```py
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

训练流水线。

![2](img/#co_understanding_wikipedia___span_class__keep_together__with_lda_and_spark_nlp__span__CO3-2)

应用流水线来转换数据框。

NLP 流水线创建了中间列，我们不需要这些列。让我们移除冗余列：

```py
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

接下来，我们将理解 TF-IDF 的基础并在预处理的数据集`token_df`上实现它，之后再建立 LDA 模型。

# TF-IDF

在应用 LDA 之前，我们需要将我们的数据转换为数值表示。我们将使用词频-逆文档频率方法来获取这样的表示。大致上，TF-IDF 用于确定与给定文档对应的术语的重要性。以下是 Python 代码中该公式的表示。实际上我们不会使用这段代码，因为 PySpark 提供了自己的实现。

```py
import math

def term_doc_weight(term_frequency_in_doc, total_terms_in_doc,
                    term_freq_in_corpus, total_docs):
  tf = term_frequency_in_doc / total_terms_in_doc
  doc_freq = total_docs / term_freq_in_corpus
  idf = math.log(doc_freq)
  tf * idf
}
```

TF-IDF 捕捉了关于术语对文档相关性的两种直觉。首先，我们期望术语在文档中出现得越频繁，它对该文档的重要性就越高。其次，全局意义上，并非所有术语都是平等的。在整个语料库中遇到很少出现的单词比在大多数文档中出现的单词更有意义；因此，该度量使用了单词在全语料库中出现的*逆数*。

语料库中单词的频率通常呈指数分布。一个常见单词的出现次数可能是一个较不常见单词的十倍，而后者又可能比一个稀有单词出现的十到一百倍更常见。如果基于原始逆文档频率来衡量指标，稀有单词将会占据主导地位，而几乎忽略其他所有单词的影响。为了捕捉这种分布，该方案使用了逆文档频率的*对数*。这种方式通过将它们之间的乘法差距转换为加法差距，从而减轻了文档频率之间的差异。

该模型基于几个假设。它将每个文档视为“词袋”，即不关注词语的顺序、句子结构或否定形式。通过一次性地表示每个术语，模型在处理多义性（同一词语具有多种含义）时遇到困难。例如，模型无法区分“Radiohead is the best band ever” 和 “I broke a rubber band” 中“band” 的用法。如果这两个句子在语料库中频繁出现，模型可能将“Radiohead” 关联到 “rubber”。

现在让我们使用 PySpark 实现 TF-IDF。

# 计算 TF-IDF

首先，我们将使用 `CountVectorizer` 计算 TF（词项频率；即文档中每个词项的频率），该过程保留了正在创建的词汇表，以便我们可以将主题映射回相应的单词。TF 创建一个矩阵，计算词汇表中每个词在每个文本主体中出现的次数。然后，根据其频率给每个词赋予权重。在拟合过程中获得我们数据的词汇表，并在转换步骤中获取计数：

```py
from pyspark.ml.feature import CountVectorizer
cv = CountVectorizer(inputCol="tokens", outputCol="raw_features")

# train the model
cv_model = cv.fit(tokens_df)

# transform the data. Output column name will be raw_features.
vectorized_tokens = cv_model.transform(tokens_df)
```

接下来，我们进行 IDF 计算（文档中术语出现的逆频率），以减少常见术语的权重：

```py
from pyspark.ml.feature import IDF
idf = IDF(inputCol="raw_features", outputCol="features")

idf_model = idf.fit(vectorized_tokens)

vectorized_df = idf_model.transform(vectorized_tokens)
```

结果如下所示：

```py
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

经过所有的预处理和特征工程处理，我们现在可以创建我们的 LDA 模型。这是我们将在下一节中完成的工作。

# 创建我们的 LDA 模型

我们之前提到过，LDA 将语料库提炼为一组相关主题。在本节的后面部分，我们将看到这些主题的示例。在此之前，我们需要确定我们的 LDA 模型需要的两个超参数。它们是主题数（称为 *k*）和迭代次数。

选择 *k* 的多种方法。两种流行的用于此目的的度量是困惑度和主题一致性。前者可以通过 PySpark 实现获得。基本思想是尝试找出在这些指标改善开始变得不显著的 *k* 值。如果你熟悉 K-means 中用于找到聚类数的“拐点方法”，那么这类似。根据语料库的大小，这可能是一个资源密集型和耗时的过程，因为您需要为多个 *k* 值构建模型。另一种方法可能是尝试创建数据集的代表性样本，并使用它来确定 *k*。这留给你作为一个练习去阅读和尝试。

由于您可能正在本地工作，我们现在将为其分配合理的值（*k* 设置为 5，`max_iter` 设置为 50）。

让我们创建我们的 LDA 模型：

```py
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

困惑度是衡量模型预测样本能力的度量。低困惑度表明概率分布在预测样本时表现良好。在比较不同模型时，选择困惑度值较低的那个模型。

现在我们已经创建了我们的模型，我们希望将主题输出为人类可读的形式。我们将获取从我们的预处理步骤生成的词汇表，从 LDA 模型获取主题，并将它们映射起来。

```py
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

创建一个对我们的词汇表的引用。

![2](img/#co_understanding_wikipedia___span_class__keep_together__with_lda_and_spark_nlp__span__CO4-2)

使用`describeTopics`方法从 LDA 模型获取生成的主题，并将它们加载到 Python 列表中。

![3](img/#co_understanding_wikipedia___span_class__keep_together__with_lda_and_spark_nlp__span__CO4-3)

获取来自我们主题的词汇术语的索引。

现在让我们生成从主题索引到词汇表的映射：

```py
for i, topic in enumerate(topics, start=1):
    print(f"topic {i}: {topic}")
...

topic 1: 'islam', 'health', 'drug', 'empir', 'medicin', 'polici',...
topic 2: ['formula', 'group', 'algebra', 'gener', 'transform',    ...
topic 3: ['triangl', 'plane', 'line', 'point', 'two', 'tangent',  ...
topic 4: ['face', 'therapeut', 'framework', 'particl', 'interf',  ...
topic 5: ['comput', 'polynomi', 'pattern', 'internet', 'network', ...
```

上述结果并不完美，但在主题中可以注意到一些模式。主题 1 主要与健康相关。它还包含对伊斯兰教和帝国的引用。这可能是因为它们在医学史上被引用，或者其他原因？主题 2 和 3 与数学有关，后者倾向于几何学。主题 5 是计算机和数学的混合。即使您没有阅读任何文档，您也可以合理准确地猜测它们的类别。这很令人兴奋！

我们现在还可以检查我们的输入文档与哪些主题最相关。单个文档可以有多个显著的主题关联。现在，我们只会查看最强关联的主题。

让我们在输入数据框上运行 LDA 模型的转换操作：

```py
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

正如您可以看到的那样，每个文档都有与其关联的主题概率分布。为了获取每个文档的相关主题，我们想要找出具有最高概率分数的主题索引。然后我们可以将其映射到之前获取的主题。

我们将编写一个 PySpark UDF 来找到每条记录的最高主题概率分数：

```py
from pyspark.sql.types import IntegerType
max_index = fun.udf(lambda x: x.tolist().index(max(x)) + 1, IntegerType())
lda_df = lda_df.withColumn('topic_index',
                        max_index(fun.col('topicDistribution')))
```

```py
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

如果你还记得，主题 2 与数学相关联。输出符合我们的期望。您可以扫描数据集的更多部分，以查看其执行情况。您可以使用`where`或`filter`命令选择特定主题，并与之前生成的主题列表进行比较，以更好地了解已创建的聚类。正如本章开头所承诺的，我们能够在不阅读文章的情况下将其聚类到不同的主题中！

# 接下来的步骤

在本章中，我们对维基百科语料库进行了 LDA 分析。在此过程中，我们还学习了使用令人惊叹的 Spark NLP 库和 TF-IDF 技术进行文本预处理。您可以通过改进模型的预处理和超参数调整进一步完善它。此外，甚至可以尝试基于文档相似性推荐类似条目，当用户提供输入时。这样的相似度度量可以通过从 LDA 获得的概率分布向量来获得。

此外，还存在多种理解大型文本语料库的方法。例如，一种称为潜在语义分析（LSA）的技术在类似的应用中很有用，并且在本书的前一版中使用了相同的数据集。深度学习，这在[第十章中有探讨，也提供了进行主题建模的途径。您可以自行探索这些方法。
