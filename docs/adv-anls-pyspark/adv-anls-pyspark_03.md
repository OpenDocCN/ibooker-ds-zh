# 第三章。推荐音乐与Audioscrobbler数据集

推荐引擎是大规模机器学习的最受欢迎的示例之一；例如，大多数人都熟悉亚马逊的推荐引擎。它是一个共同的基础，因为推荐引擎无处不在，从社交网络到视频网站再到在线零售商。我们也可以直接观察它们的运作。我们知道计算机正在挑选在Spotify上播放的曲目，这与我们不一定注意到Gmail是否决定入站邮件是否为垃圾邮件的方式类似。

推荐引擎的输出比其他机器学习算法更直观易懂。这甚至是令人兴奋的。尽管我们认为音乐口味是个人的且难以解释，推荐系统却出奇地能够很好地识别我们未曾预料到会喜欢的曲目。对于音乐或电影等领域，推荐系统经常被部署，我们可以比较容易地推断为什么推荐的音乐与某人的听歌历史相符。并非所有的聚类或分类算法都符合这一描述。例如，支持向量机分类器是一组系数，即使是从业者也很难表达这些数字在进行预测时的含义。

以推荐引擎为中心的下三章探讨PySpark上的关键机器学习算法似乎很合适，特别是推荐音乐。这是一个引入PySpark和MLlib的实际应用的可访问方式，并介绍一些基本的机器学习思想，这些思想将在随后的章节中进一步发展。

在本章中，我们将在PySpark中实现一个推荐系统。具体而言，我们将使用一个音乐流媒体服务提供的开放数据集上的交替最小二乘（ALS）算法。我们将从理解数据集并在PySpark中导入开始。然后我们将讨论选择ALS算法的动机及其在PySpark中的实现。接下来是数据准备和使用PySpark构建模型。最后，我们将进行一些用户推荐，并讨论通过超参数选择改进我们的模型的方法。

# 数据设置

我们将使用由Audioscrobbler发布的数据集。Audioscrobbler是[Last.fm](http://www.last.fm)的第一个音乐推荐系统，是2002年成立的最早的互联网流媒体电台站点之一。Audioscrobbler提供了一个用于“scrobbling”（记录听众歌曲播放）的开放API。Last.fm利用这些信息构建了一个强大的音乐推荐引擎。该系统通过第三方应用程序和网站能够向推荐引擎提供听歌数据，达到了数百万用户。

那时，关于推荐引擎的研究大多局限于从类似评分的数据中学习。也就是说，推荐系统通常被视为在如“Bob 给 Prince 评了 3.5 星。”这样的输入上操作的工具。Audioscrobbler 数据集很有趣，因为它仅记录了播放信息：“Bob 播放了 Prince 的一首曲目。”一个播放包含的信息比一个评分少。仅仅因为 Bob 播放了这首曲目，并不意味着他实际上喜欢它。你或者我偶尔也可能播放一个我们不喜欢的歌手的歌曲，甚至播放一个专辑然后离开房间。

然而，听众评价音乐的频率远低于他们播放音乐的频率。因此，这样的数据集要大得多，涵盖的用户和艺术家更多，包含的总信息量也更多，即使每个个体数据点携带的信息较少。这种类型的数据通常被称为 *隐式反馈* 数据，因为用户和艺术家之间的连接是作为其他行为的副产品而暗示的，并不是作为显式评分或赞的形式给出。

2005 年 Last.fm 分发的数据集快照可以在 [在线压缩档案](https://oreil.ly/Z7sfL) 中找到。下载这个档案，并在其中找到几个文件。首先，需要使数据集的文件可用。如果你使用的是远程集群，请将所有三个数据文件复制到存储中。本章将假定这些文件在 *data/* 目录下可用。

启动 `pyspark-shell`。请注意，本章中的计算将比简单应用程序消耗更多内存。例如，如果你是在本地而不是在集群上运行，可能需要指定类似 `--driver-memory 4g` 这样的选项，以确保有足够的内存完成这些计算。主要数据集在 *user_artist_data.txt* 文件中。它包含约 141,000 位唯一用户和 1.6 百万位唯一艺术家。记录了大约 2420 万位用户对艺术家的播放次数，以及它们的计数。让我们将这个数据集读入一个 DataFrame 并查看它：

```py
raw_user_artist_path = "data/audioscrobbler_data/user_artist_data.txt"
raw_user_artist_data = spark.read.text(raw_user_artist_path)

raw_user_artist_data.show(5)

...
+-------------------+
|              value|
+-------------------+
|       1000002 1 55|
| 1000002 1000006 33|
|  1000002 1000007 8|
|1000002 1000009 144|
|1000002 1000010 314|
+-------------------+
```

像 ALS 这样的机器学习任务可能比简单的文本处理更需要计算资源。最好将数据分成更小的片段——更多的分区——进行处理。你可以在读取文本文件后链式调用 `.repartition(n)`，以指定一个不同且更大的分区数。例如，你可以将这个数设置得比集群中的核心数更高。

数据集还在 *artist_data.txt* 文件中按 ID 列出了每位艺术家的姓名。请注意，当播放时，客户端应用程序会提交正在播放的艺术家的名称。这个名称可能拼写错误或非标准，并且这可能只有在后期才能检测到。例如，“The Smiths”，“Smiths, The” 和 “the smiths” 可能会在数据集中出现为不同的艺术家 ID，尽管它们显然是同一位艺术家。因此，数据集还包括 *artist_alias.txt*，其中映射了已知拼写错误或变体的艺术家 ID 到该艺术家的规范 ID。让我们也将这两个数据集读入 PySpark：

```py
raw_artist_data = spark.read.text("data/audioscrobbler_data/artist_data.txt")

raw_artist_data.show(5)

...
+--------------------+
|               value|
+--------------------+
|1134999\t06Crazy ...|
|6821360\tPang Nak...|
|10113088\tTerfel,...|
|10151459\tThe Fla...|
|6826647\tBodensta...|
+--------------------+
only showing top 5 rows
...

raw_artist_alias = spark.read.text("data/audioscrobbler_data/artist_alias.txt")

raw_artist_alias.show(5)

...
+-----------------+
|            value|
+-----------------+
| 1092764\t1000311|
| 1095122\t1000557|
| 6708070\t1007267|
|10088054\t1042317|
| 1195917\t1042317|
+-----------------+
only showing top 5 rows
```

现在我们对数据集有了基本了解，我们可以讨论我们对推荐算法的需求，并且随后理解为什么交替最小二乘算法是一个不错的选择。

# 我们对推荐系统的要求

我们需要选择一个适合我们数据的推荐算法。以下是我们的考虑：

隐式反馈

数据完全由用户和艺术家歌曲之间的互动组成。除了它们的名字外，没有关于用户或艺术家的其他信息。我们需要一种可以在没有用户或艺术家属性访问的情况下学习的算法。这些通常被称为协同过滤算法。例如，决定两个用户可能分享相似的口味，因为他们年龄相同*不是*协同过滤的例子。决定两个用户可能都喜欢同一首歌，因为他们播放了许多其他相同的歌曲*是*一个例子。

稀疏性

我们的数据集看起来很大，因为它包含数千万的播放次数。但从另一个角度来看，它又很小且稀疏，因为它是稀疏的。平均而言，每个用户从大约171位艺术家那里播放了歌曲——这些艺术家中有160万。有些用户只听过一个艺术家的歌曲。我们需要一种算法，即使对这些用户也能提供合理的推荐。毕竟，每个单独的听众最初肯定只是从一个播放开始的！

可扩展性和实时预测

最后，我们需要一个在建立大模型和快速创建推荐方面都能扩展的算法。推荐通常需要几乎实时——不到一秒的时间，而不是明天。

一类可能适合的广泛算法是潜在因子模型。它们试图通过相对较少的*未观察到的潜在原因*来解释大量用户和项目之间的*观察到的互动*。例如，考虑一个顾客购买了金属乐队Megadeth和Pantera的专辑，但同时也购买了古典作曲家莫扎特的专辑。可能很难解释为什么会购买这些专辑而不是其他的。然而，这可能只是更大音乐口味集合中的一个小窗口。也许这位顾客喜欢从金属到前卫摇滚再到古典的一致音乐谱系。这种解释更为简单，并且额外提出了许多其他可能感兴趣的专辑。在这个例子中，“喜欢金属、前卫摇滚和古典”这三个潜在因子可以解释成千上万个单独的专辑偏好。

在我们的案例中，我们将特别使用一种矩阵因子化模型。在数学上，这些算法将用户和产品数据视为一个大矩阵 *A*，如果用户 *i* 播放了艺术家 *j*，则第 *i* 行和第 *j* 列的条目存在。*A* 是稀疏的：大多数 *A* 的条目都是 0，因为实际数据中只有少数所有可能的用户-艺术家组合。他们将 *A* 分解为两个较小矩阵的乘积，*X* 和 *Y*。它们非常瘦长——因为 *A* 有很多行和列，但它们都只有几列（*k*）。*k* 列对应于被用来解释互动数据的潜在因素。

由于 *k* 较小，因此因子化只能是近似的，如[图 3-1](#ALSFactorization)所示。

![aaps 0301](assets/aaps_0301.png)

###### 图 3-1\. 矩阵因子化

这些算法有时被称为矩阵完成算法，因为原始矩阵 *A* 可能非常稀疏，但乘积 *XY*^(*T*) 是密集的。很少有，如果有的话，条目是 0，因此模型仅是 *A* 的近似。它是一个模型，因为它为原始 *A* 中许多缺失的条目（即 0）产生了一个值。

这是一个令人高兴的例子，线性代数直接而优雅地映射到直觉中。这两个矩阵包含每个用户和每个艺术家的一行。这些行只有很少的值——*k*。每个值对应于模型中的一个潜在特征。因此，这些行表达了用户和艺术家与这些 *k* 个潜在特征的关联程度，这些特征可能对应于品味或流派。它只是将用户特征和特征艺术家矩阵的乘积，得到了整个密集用户-艺术家互动矩阵的完整估计。这个乘积可以被认为是将项目映射到它们的属性，然后根据用户属性加权。

坏消息是 *A* = *XY*^(*T*) 通常根本没有确切的解，因为 *X* 和 *Y* 不够大（严格来说，[秩](https://oreil.ly/OfVj4)太低），无法完美地表示 *A*。这实际上是件好事。*A* 只是所有可能发生的互动中的一个小样本。某种程度上，我们认为 *A* 是一个非常零散、因此难以解释的更简单的潜在现实的观点，它只能通过其中的一些少量因素，*k* 个因素，进行很好的解释。想象一幅描绘猫的拼图。最终拼图很容易描述：一只猫。然而，当你手上只有几片时，你看到的图像却很难描述。

*XY*^(*T*) 应该尽可能接近 *A*。毕竟，这是我们所依赖的唯一依据。它不会也不应该完全复制它。坏消息是，这不能直接为同时获得最佳的 *X* 和 *Y* 而解决。好消息是，如果已知 *Y*，那么解决最佳 *X* 就很简单，反之亦然。但事先都不知道！

幸运的是，有一些算法可以摆脱这种困境并找到一个合理的解决方案。 PySpark中可用的一个这样的算法是ALS算法。

## 交替最小二乘法算法

我们将使用交替最小二乘法算法从我们的数据集中计算潜在因子。 这种方法在Netflix Prize竞赛期间由《“Collaborative Filtering for Implicit Feedback Datasets”》和《“Large-Scale Parallel Collaborative Filtering for the Netflix Prize”》等论文流行起来。 PySpark MLlib的ALS实现汲取了这两篇论文的思想，并且是目前在Spark MLlib中唯一实现的推荐算法。

这里是一段代码片段（非功能性的），让你一窥后面章节的内容：

```py
from pyspark.ml.recommendation import ALS

als = ALS(maxIter=5, regParam=0.01, userCol="user",
          itemCol="artist", ratingCol="count")
model = als.fit(train)

predictions = model.transform(test)
```

通过ALS，我们将把我们的输入数据视为一个大的稀疏矩阵*A*，并找出*X*和*Y*，就像前面的部分所讨论的那样。 起初，*Y*是未知的，但可以初始化为一个由随机选择的行向量组成的矩阵。 然后，简单的线性代数给出了最佳的*X*解，给定*A*和*Y*。 实际上，可以分别计算*X*的每一行*i*作为*Y*和*A*的函数，并且可以并行进行。 对于大规模计算来说，这是一个很好的特性：

<math display="block"><mrow><msub><mi>A</mi><mi>i</mi></msub><mi>Y</mi><mo>(</mo><msup><mi>Y</mi><mi>T</mi></msup><mi>Y</mi><msup><mo>)</mo><mi>–1</mi></msup> <mo>=</mo> <msub><mi>X</mi><mi>i</mi></msub></mrow></math>

实际上无法完全实现相等，因此目标实际上是将|*A*[*i*]*Y*(*Y*^(*T*)*Y*)^(*–1*) – *X*[*i*]|最小化，或者说是两个矩阵条目之间的平方差的总和。 这就是名称中“最小二乘”的含义。 实际上，这从未通过计算逆矩阵来解决，而是通过QR分解等方法更快、更直接地解决。 这个方程只是详细说明了如何计算行向量的理论。

通过同样的方式可以计算每个*Y*[*j*]来自*X*。 同样地，来自*Y*的计算*X*，以此类推。 这就是“交替”部分的来源。 只有一个小问题：*Y*是编造出来的——而且是随机的！ *X*被最佳计算了，是的，但对*Y*给出了一个虚假的解决方案。 幸运的是，如果这个过程重复进行，*X*和*Y*最终会收敛到合理的解决方案。

当用于因子分解表示隐式数据的矩阵时，ALS因子分解会更加复杂一些。 它并不直接分解输入矩阵*A*，而是一个由0和1组成的矩阵*P*，其中包含*A*中包含正值的位置为1，其他位置为0。 *A*中的值稍后将作为权重加入。 这个细节超出了本书的范围，但并不影响理解如何使用该算法。

最后，ALS算法也可以利用输入数据的稀疏性。 这个特性以及它对简单、优化的线性代数和数据并行的依赖，使其在大规模情况下非常快速。

接下来，我们将预处理我们的数据集，并使其适合与ALS算法一起使用。

# 数据准备

构建模型的第一步是了解可用的数据，并将其解析或转换为在Spark中进行分析时有用的形式。

Spark MLlib的ALS实现在用户和项目的ID不是严格要求为数字时也可以工作，但当ID实际上可以表示为32位整数时效率更高。这是因为在底层使用JVM的数据类型来表示数据。这个数据集是否已经符合这个要求？

```py
raw_user_artist_data.show(10)

...
+-------------------+
|              value|
+-------------------+
|       1000002 1 55|
| 1000002 1000006 33|
|  1000002 1000007 8|
|1000002 1000009 144|
|1000002 1000010 314|
|  1000002 1000013 8|
| 1000002 1000014 42|
| 1000002 1000017 69|
|1000002 1000024 329|
|  1000002 1000025 1|
+-------------------+
```

文件的每一行包含用户ID、艺术家ID和播放计数，用空格分隔。为了对用户ID进行统计，我们通过空格字符分割行并解析值为整数。结果概念上有三个“列”：用户ID、艺术家ID和计数，都是`int`类型。将其转换为列名为“user”、“artist”和“count”的数据框是有意义的，因为这样可以简单地计算最大值和最小值：

```py
from pyspark.sql.functions import split, min, max
from pyspark.sql.types import IntegerType, StringType

user_artist_df = raw_user_artist_data.withColumn('user',
                                    split(raw_user_artist_data['value'], ' ').\
                                    getItem(0).\
                                    cast(IntegerType()))
user_artist_df = user_artist_df.withColumn('artist',
                                    split(raw_user_artist_data['value'], ' ').\
                                    getItem(1).\
                                    cast(IntegerType()))
user_artist_df = user_artist_df.withColumn('count',
                                    split(raw_user_artist_data['value'], ' ').\
                                    getItem(2).\
                                    cast(IntegerType())).drop('value')

user_artist_df.select([min("user"), max("user"), min("artist"),\
                                    max("artist")]).show()
...
+---------+---------+-----------+-----------+
|min(user)|max(user)|min(artist)|max(artist)|
+---------+---------+-----------+-----------+
|       90|  2443548|          1|   10794401|
+---------+---------+-----------+-----------+
```

最大的用户和艺术家ID分别为2443548和10794401（它们的最小值分别为90和1；没有负值）。这些值明显小于2147483647。不需要额外的转换即可使用这些ID。

在本示例中稍后知道与不透明数字ID对应的艺术家名称将会很有用。`raw_artist_data`包含由制表符分隔的艺术家ID和名称。PySpark的split函数接受正则表达式作为`pattern`参数的值。我们可以使用空白字符`\s`进行分割：

```py
from pyspark.sql.functions import col

artist_by_id = raw_artist_data.withColumn('id', split(col('value'), '\s+', 2).\
                                                getItem(0).\
                                                cast(IntegerType()))
artist_by_id = artist_by_id.withColumn('name', split(col('value'), '\s+', 2).\
                                               getItem(1).\
                                               cast(StringType())).drop('value')

artist_by_id.show(5)
...
+--------+--------------------+
|      id|                name|
+--------+--------------------+
| 1134999|        06Crazy Life|
| 6821360|        Pang Nakarin|
|10113088|Terfel, Bartoli- ...|
|10151459| The Flaming Sidebur|
| 6826647|   Bodenstandig 3000|
+--------+--------------------+
```

这将导致一个数据框，其列为`id`和`name`，代表艺术家ID和名称。

`raw_artist_alias`将可能拼写错误或非标准的艺术家ID映射到艺术家规范名称的ID。这个数据集相对较小，包含约200,000条目。每行包含两个ID，用制表符分隔。我们将以与`raw_artist_data`类似的方式解析它：

```py
artist_alias = raw_artist_alias.withColumn('artist',
                                          split(col('value'), '\s+').\
                                                getItem(0).\
                                                cast(IntegerType())).\
                                withColumn('alias',
                                            split(col('value'), '\s+').\
                                            getItem(1).\
                                            cast(StringType())).\
                                drop('value')

artist_alias.show(5)
...
+--------+-------+
|  artist|  alias|
+--------+-------+
| 1092764|1000311|
| 1095122|1000557|
| 6708070|1007267|
|10088054|1042317|
| 1195917|1042317|
+--------+-------+
```

第一个条目将ID 1092764映射到1000311。我们可以从`artist_by_id`数据框中查找这些信息：

```py
artist_by_id.filter(artist_by_id.id.isin(1092764, 1000311)).show()
...

+-------+--------------+
|     id|          name|
+-------+--------------+
|1000311| Steve Winwood|
|1092764|Winwood, Steve|
+-------+--------------+
```

此条目显然将“Winwood, Steve”映射为“Steve Winwood”，这实际上是艺术家的正确名称。

# 构建第一个模型

尽管该数据集几乎适合与Spark MLlib的ALS实现一起使用，但它需要进行一个小的额外转换。应用别名数据集以将所有艺术家ID转换为规范ID（如果存在不同的规范ID）将会很有用：

```py
from pyspark.sql.functions import broadcast, when

train_data = train_data = user_artist_df.join(broadcast(artist_alias),
                                              'artist', how='left').\ train_data = train_data.withColumn('artist',
                                    when(col('alias').isNull(), col('artist')).\
                                    otherwise(col('alias'))) ![1](assets/1.png)
train_data = train_data.withColumn('artist', col('artist').\
                                             cast(IntegerType())).\
                                             drop('alias')

train_data.cache()

train_data.count()
...
24296858
```

[![1](assets/1.png)](#co_recommending_music_and_the_audioscrobbler_dataset_CO1-1)

如果存在艺术家别名，则获取艺术家别名；否则，获取原始艺术家。

我们广播先前创建的`artist_alias`数据框。这使得Spark在集群中的每个执行器上仅发送和保存一个副本。当有成千上万个任务并且许多任务并行执行时，这可以节省大量的网络流量和内存。作为经验法则，在与一个非常大的数据集进行连接时，广播一个显著较小的数据集是很有帮助的。

调用`cache`告诉Spark，在计算后应该暂时存储这个DataFrame，并且在集群内存中保持。这很有帮助，因为ALS算法是迭代的，通常需要多次访问这些数据。如果没有这一步，每次访问时DataFrame都可能会从原始数据中重新计算！Spark UI中的存储选项卡将显示DataFrame的缓存量和内存使用量，如图[Figure 3-2](#Recommender_StorageTag)所示。这个DataFrame在整个集群中大约消耗了120 MB。

![aaps 0302](assets/aaps_0302.png)

###### 图3-2\. Spark UI中的存储选项卡，显示缓存的DataFrame内存使用情况

当您使用`cache`或`persist`时，DataFrame不会完全缓存，直到您触发一个需要遍历每条记录的动作（例如`count`）。如果您使用像`show(1)`这样的操作，只有一个分区会被缓存。这是因为PySpark的优化器会发现您只需计算一个分区即可检索一条记录。

注意，在UI中，“反序列化”标签在图[Figure 3-2](#Recommender_StorageTag)中实际上只与RDD相关，其中“序列化”表示数据存储在内存中，不是作为对象，而是作为序列化的字节。然而，像这种DataFrame实例会单独对常见数据类型在内存中进行“编码”。

实际上，120 MB的消耗量令人惊讶地小。考虑到这里存储了大约2400万次播放记录，一个快速的粗略计算表明，每个用户-艺术家-计数条目平均只消耗5字节。然而，仅三个32位整数本身应该消耗12字节。这是DataFrame的一个优点之一。因为存储的数据类型是基本的32位整数，它们在内存中的表示可以进行优化。

最后，我们可以构建一个模型：

```py
from pyspark.ml.recommendation import ALS

model = ALS(rank=10, seed=0, maxIter=5, regParam=0.1,
            implicitPrefs=True, alpha=1.0, userCol='user',
            itemCol='artist', ratingCol='count'). \
        fit(train_data)
```

这将使用一些默认配置构建一个`ALSModel`作为`model`。根据您的集群不同，此操作可能需要几分钟甚至更长时间。与某些机器学习模型最终形式可能仅包含几个参数或系数不同，这种模型非常庞大。它包含模型中每个用户和产品的10个值的特征向量，在这种情况下超过170万个。该模型包含这些大型用户特征和产品特征矩阵作为它们自己的DataFrame。

您的结果中的值可能会有所不同。最终模型取决于随机选择的初始特征向量集。然而，MLlib中此类组件的默认行为是使用相同的随机选择集合，默认情况下使用固定种子。这与其他库不同，其他库通常不会默认固定随机元素的行为。因此，在这里和其他地方，随机种子设置为`(… seed=0,…)。

要查看一些特征向量，请尝试以下操作，它仅显示一行并且不截断特征向量的宽显示：

```py
model.userFactors.show(1, truncate = False)

...
+---+----------------------------------------------- ...
|id |features                                        ...
+---+----------------------------------------------- ...
|90 |[0.16020626, 0.20717518, -0.1719469, 0.06038466 ...
+---+----------------------------------------------- ...
```

在`ALS`上调用的其他方法，如`setAlpha`，设置的是可以影响模型推荐质量的*超参数*值。这些将在后面解释。更重要的第一个问题是：这个模型好吗？它能产生好的推荐吗？这是我们将在下一节试图回答的问题。

# 抽查推荐

我们首先应该看看艺术家的推荐是否有直觉上的意义，通过检查一个用户、他或她的播放以及对该用户的推荐来判断。比如说，用户2093760。首先，让我们看看他或她的播放，以了解这个人的口味。提取此用户听过的艺术家ID并打印他们的名称。这意味着通过搜索该用户播放的艺术家ID来过滤艺术家集，并按顺序打印名称：

```py
user_id = 2093760

existing_artist_ids = train_data.filter(train_data.user == user_id) \ ![1](assets/1.png)
  .select("artist").collect() ![2](assets/2.png)

existing_artist_ids = [i[0] for i in existing_artist_ids]

artist_by_id.filter(col('id').isin(existing_artist_ids)).show() ![3](assets/3.png)
...
+-------+---------------+
|     id|           name|
+-------+---------------+
|   1180|     David Gray|
|    378|  Blackalicious|
|    813|     Jurassic 5|
|1255340|The Saw Doctors|
|    942|         Xzibit|
+-------+---------------+
```

[![1](assets/1.png)](#co_recommending_music_and_the_audioscrobbler_dataset_CO2-1)

查找其用户为2093760的行。

[![2](assets/2.png)](#co_recommending_music_and_the_audioscrobbler_dataset_CO2-2)

收集艺术家ID数据集。

[![3](assets/3.png)](#co_recommending_music_and_the_audioscrobbler_dataset_CO2-3)

过滤这些艺术家。

这些艺术家看起来像是主流流行音乐和嘻哈的混合。是不是恐龙5的粉丝？记住，那是2005年。如果你好奇的话，锯医生是一个非常受爱尔兰欢迎的爱尔兰摇滚乐队。

现在，简单地为用户做出推荐虽然用这种方式计算需要一些时间。它适用于批量评分，但不适合实时使用情况：

```py
user_subset = train_data.select('user').where(col('user') == user_id).distinct()
top_predictions = model.recommendForUserSubset(user_subset, 5)

top_predictions.show()
...
+-------+--------------------+
|   user|     recommendations|
+-------+--------------------+
|2093760|[{2814, 0.0294106...|
+-------+--------------------+
```

结果推荐包含由艺术家ID组成的列表，当然还有“预测”。对于这种ALS算法，预测是一个通常在0到1之间的不透明值，较高的值意味着更好的推荐。它不是概率，但可以看作是一个估计的0/1值，指示用户是否会与艺术家互动。

在提取了推荐的艺术家ID之后，我们可以类似地查找艺术家名称：

```py
top_predictions_pandas = top_predictions.toPandas()
print(top_prediction_pandas)
...
      user                                    recommendations
0  2093760  [(2814, 0.029410675168037415), (1300642, 0.028...
...

recommended_artist_ids = [i[0] for i in top_predictions_pandas.\
                                        recommendations[0]]

artist_by_id.filter(col('id').isin(recommended_artist_ids)).show()
...
+-------+----------+
|     id|      name|
+-------+----------+
|   2814|   50 Cent|
|   4605|Snoop Dogg|
|1007614|     Jay-Z|
|1001819|      2Pac|
|1300642|  The Game|
+-------+----------+
```

结果全是嘻哈。乍一看，这看起来不是一个很好的推荐集。虽然这些通常是受欢迎的艺术家，但似乎并没有根据这位用户的听歌习惯来个性化推荐。

# 评估推荐质量

当然，这只是对一个用户结果的主观判断。除了该用户之外，其他人很难量化推荐的好坏。此外，人工对甚至是小样本输出进行评分以评估结果是不可行的。

假设用户倾向于播放有吸引力的艺术家的歌曲，而不播放无吸引力的艺术家的歌曲，这是合理的。因此，用户的播放记录只能部分反映出“好”的和“坏”的艺术家推荐。这是一个有问题的假设，但在没有其他数据的情况下，这可能是最好的做法。例如，假设用户2093760喜欢的艺术家远不止之前列出的5个，并且在那170万名未播放的艺术家中，有些是感兴趣的，而不是所有的都是“坏”推荐。

如果一个推荐系统的评估是根据其在推荐列表中高排名好艺术家的能力，那将是一个通用的度量标准之一，适用于像推荐系统这样排名物品的系统。问题在于，“好”被定义为“用户曾经听过的艺术家”，而推荐系统已经把所有这些信息作为输入接收了。它可以轻松地返回用户以前听过的艺术家作为顶级推荐，并得到完美的分数。但这并不实用，特别是因为推荐系统的角色是推荐用户以前从未听过的艺术家。

为了使其具有意义，一些艺术家的播放数据可以被保留，并且隐藏在ALS模型构建过程之外。然后，这些保留的数据可以被解释为每个用户的一组好推荐，但是这些推荐并不是推荐系统已经给出的。推荐系统被要求对模型中的所有项目进行排名，并检查保留艺术家的排名。理想情况下，推荐系统会将它们全部或几乎全部排在列表的顶部。

然后，我们可以通过比较所有保留艺术家的排名与其余艺术家的排名来计算推荐系统的分数。（在实践中，我们仅检查所有这些对中的一部分样本，因为可能存在大量这样的对。）保留艺术家排名较高的比例就是其分数。分数为1.0表示完美，0.0是最差的分数，0.5是从随机排名艺术家中达到的期望值。

这个度量标准直接与信息检索概念“接收者操作特征曲线（ROC曲线）”有关。上述段落中的度量标准等于这个ROC曲线下的面积，确实被称为AUC，即曲线下面积。AUC可以被视为随机选择的一个好推荐高于随机选择的一个坏推荐的概率。

AUC指标也用于分类器的评估。它与相关方法一起实现在MLlib类`BinaryClassificationMetrics`中。对于推荐系统，我们将计算每个用户的AUC，并对结果取平均值。得到的度量标准略有不同，可能称为“平均AUC”。我们将实现这一点，因为它在PySpark中尚未（完全）实现。

其他与排名相关的评估指标实现在`RankingMetrics`中。这些包括精度、召回率和[平均精度均值（MAP）](https://oreil.ly/obbTT)等指标。MAP也经常被使用，更专注于顶部推荐的质量。然而，在这里AUC将作为衡量整个模型输出质量的常见和广泛的指标使用。

实际上，在所有机器学习中，保留一些数据以选择模型并评估其准确性的过程都是常见的实践。通常，数据被分为三个子集：训练集、交叉验证（CV）集和测试集。在这个初始示例中为简单起见，只使用两个集合：训练集和CV集。这足以选择一个模型。在[第四章](ch04.xhtml#making_predictions_with_decision_trees_and_decision_forests)，这个想法将扩展到包括测试集。

# 计算AUC

在本书附带的源代码中提供了平均AUC的实现。这里不会重复，但在源代码的注释中有详细解释。它接受CV集作为每个用户的“正向”或“好”的艺术家，并接受一个预测函数。这个函数将包含每个用户-艺术家对的数据框转换为另一个数据框，其中也包含其估计的互动强度作为“预测”，一个数字，较高的值意味着在推荐中排名更高。

要使用输入数据，我们必须将其分为训练集和CV集。ALS模型仅在训练数据集上进行训练，CV集用于评估模型。这里，90%的数据用于训练，剩余的10%用于交叉验证：

```py
def area_under_curve(
    positive_data,
    b_all_artist_IDs,
    predict_function):
...

all_data = user_artist_df.join(broadcast(artist_alias), 'artist', how='left') \
    .withColumn('artist', when(col('alias').isNull(), col('artist'))\
    .otherwise(col('alias'))) \
    .withColumn('artist', col('artist').cast(IntegerType())).drop('alias')

train_data, cv_data = all_data.randomSplit([0.9, 0.1], seed=54321)
train_data.cache()
cv_data.cache()

all_artist_ids = all_data.select("artist").distinct().count()
b_all_artist_ids = broadcast(all_artist_ids)

model = ALS(rank=10, seed=0, maxIter=5, regParam=0.1,
            implicitPrefs=True, alpha=1.0, userCol='user',
            itemCol='artist', ratingCol='count') \
        .fit(train_data)
area_under_curve(cv_data, b_all_artist_ids, model.transform)
```

注意，`areaUnderCurve`接受一个函数作为其第三个参数。在这里，从`ALSModel`中传入的`transform`方法被传递进去，但很快会被替换为另一种方法。

结果约为0.879。这好吗？它显然高于从随机推荐中预期的0.5，接近1.0，这是可能的最高分数。通常，AUC超过0.9会被认为是高的。

但这是一个准确的评估吗？这个评估可以用不同的90%作为训练集重复进行。得到的AUC值的平均值可能更好地估计了算法在数据集上的性能。实际上，一个常见的做法是将数据分成*k*个大小相似的子集，使用*k* - 1个子集一起进行训练，并在剩余的子集上进行评估。我们可以重复这个过程*k*次，每次使用不同的子集。这被称为*k*折交叉验证，这里为简单起见不会在示例中实现，但MLlib中的`CrossValidator`API支持这种技术。验证API将在[“随机森林”](ch04.xhtml#RandomDecisionForests)中重新讨论。

将其与简单方法进行基准测试很有帮助。例如，考虑向每个用户推荐全球播放量最高的艺术家。这不是个性化的，但简单且可能有效。定义这个简单的预测函数并评估其AUC分数：

```py
from pyspark.sql.functions import sum as _sum

def predict_most_listened(train):
    listen_counts = train.groupBy("artist")\
                    .agg(_sum("count").alias("prediction"))\
                    .select("artist", "prediction")

    return all_data.join(listen_counts, "artist", "left_outer").\
                    select("user", "artist", "prediction")

area_under_curve(cv_data, b_all_artist_ids, predict_most_listened(train_data))
```

结果也约为0.880。这表明根据这个度量标准，非个性化的推荐已经相当有效。然而，我们预期“个性化”的推荐在比较中会得分更高。显然，模型需要一些调整。它可以做得更好吗？

# 超参数选择

到目前为止，用于构建 `ALSModel` 的超参数值仅仅是给出而没有注释。它们不是由算法学习的，必须由调用者选择。配置的超参数是：

`setRank(10)`

模型中的潜在因子数量，或者说用户特征和产品特征矩阵中的列数*k*。在非平凡情况下，这也是它们的秩。

`setMaxIter(5)`

因子分解运行的迭代次数。更多的迭代需要更多时间，但可能产生更好的因子分解。

`setRegParam(0.01)`

一个标准的过拟合参数，通常也称为*lambda*。较高的值抵抗过拟合，但是值过高会损害因子分解的准确性。

`setAlpha(1.0)`

控制观察到的用户产品交互与未观察到的交互在因子分解中的相对权重。

`rank`、`regParam` 和 `alpha` 可以被视为模型的*超参数*。(`maxIter` 更多是对因子分解中资源使用的约束。) 这些不是最终出现在 `ALSModel` 内部矩阵中的数值—那些只是其*参数*，由算法选择。这些超参数反而是构建过程的参数。

前述列表中使用的值未必是最优的。选择好的超参数值是机器学习中的常见问题。选择值的最基本方法是简单地尝试不同的组合并评估每个组合的度量标准，选择产生最佳值的组合。

在下面的示例中，尝试了八种可能的组合：`rank` = 5 或 30，`regParam` = 4.0 或 0.0001，以及 `alpha` = 1.0 或 40.0。这些值仍然有些猜测性质，但被选择来覆盖广泛的参数值范围。结果按照最高AUC分数的顺序打印：

```py
from pprint import pprint
from itertools import product

ranks = [5, 30]
reg_params = [4.0, 0.0001]
alphas = [1.0, 40.0]
hyperparam_combinations = list(product(*[ranks, reg_params, alphas]))

evaluations = []

for c in hyperparam_combinations:
    rank = c[0]
    reg_param = c[1]
    alpha = c[2]
    model = ALS().setSeed(0).setImplicitPrefs(true).setRank(rank).\
                  setRegParam(reg_param).setAlpha(alpha).setMaxIter(20).\
                  setUserCol("user").setItemCol("artist").\
                  setRatingCol("count").setPredictionCol("prediction").\
        fit(trainData)

    auc = area_under_curve(cv_aata, b_all_artist_ids, model.transform)

    model.userFactors.unpersist() ![1](assets/1.png)
    model.itemFactors.unpersist()

    evaluations.append((auc, (rank, regParam, alpha)))

evaluations.sort(key=lambda x:x[0], reverse=True) ![2](assets/2.png)
pprint(evaluations)

...
(0.8928367485129145,(30,4.0,40.0))
(0.891835487024326,(30,1.0E-4,40.0))
(0.8912376926662007,(30,4.0,1.0))
(0.889240668173946,(5,4.0,40.0))
(0.8886268430389741,(5,4.0,1.0))
(0.8883278461068959,(5,1.0E-4,40.0))
(0.8825350012228627,(5,1.0E-4,1.0))
(0.8770527940660278,(30,1.0E-4,1.0))
```

[![1](assets/1.png)](#co_recommending_music_and_the_audioscrobbler_dataset_CO3-1)

立即释放模型资源。

[![2](assets/2.png)](#co_recommending_music_and_the_audioscrobbler_dataset_CO3-2)

按第一个值（AUC）降序排序，并打印。

绝对值上的差异很小，但对于AUC值来说仍然有一定显著性。有趣的是，参数`alpha`在40时似乎比1要好得多。（对于感兴趣的人，40是在前面提到的原始ALS论文中作为默认值提出的一个值。）这可以解释为模型更好地集中于用户实际听过的内容，而不是未听过的内容。

更高的`regParam`看起来也更好。这表明模型在某种程度上容易过拟合，因此需要更高的`regParam`来抵制试图精确拟合每个用户给出的稀疏输入。过拟合将在[“随机森林”](ch04.xhtml#RandomDecisionForests)中更详细地讨论。

预期地，对于这种规模的模型来说，5个特征相当少，并且在解释品味方面表现不佳，相比使用30个特征的模型而言。可能最佳特征数量实际上比30更高，这些值在过小方面是相似的。

当然，这个过程可以针对不同的数值范围或更多的值重复进行。这是一种选择超参数的蛮力方法。然而，在拥有数TB内存和数百核心的集群不罕见的世界中，并且像Spark这样的框架可以利用并行性和内存来提高速度，这变得非常可行。

严格来说，不需要理解超参数的含义，尽管了解正常值范围有助于开始搜索既不太大也不太小的参数空间。

这是一个相当手动的超参数循环、模型构建和评估方式。在[第4章](ch04.xhtml#making_predictions_with_decision_trees_and_decision_forests)中，了解更多关于Spark ML API之后，我们会发现有一种更自动化的方式可以使用`Pipeline`和`TrainValidationSplit`来计算这些。

# 做推荐

暂时使用最佳超参数集，新模型为用户2093760推荐什么？

```py
+-----------+
|       name|
+-----------+
|  [unknown]|
|The Beatles|
|     Eminem|
|         U2|
|  Green Day|
+-----------+
```

据传闻，对于这个用户来说这更有意义，主要是流行摇滚而不是所有的嘻哈。`[unknown]`显然不是一个艺术家。查询原始数据集发现它出现了429,447次，几乎进入前100名！这是一个在没有艺术家的情况下播放的默认值，可能是由某个特定的scrobbling客户端提供的。这不是有用的信息，我们应该在重新开始前从输入中丢弃它。这是数据科学实践通常是迭代的一个例子，在每个阶段都会发现关于数据的发现。

这个模型可以用来为所有用户做推荐。这在一个批处理过程中可能会很有用，每小时甚至更频繁地重新计算模型和用户的推荐，具体取决于数据的规模和集群的速度。

然而，目前为止，Spark MLlib的ALS实现不支持一种向所有用户推荐的方法。可以一次向一个用户推荐，如上所示，尽管每次推荐将启动一个持续几秒钟的短暂分布式作业。这可能适用于快速为小组用户重新计算推荐。这里，从数据中取出的100个用户将收到推荐并打印出来：

```py
some_users = all_data.select("user").distinct().limit(100) ![1](assets/1.png)
val someRecommendations =
  someUsers.map(userID => (userID, makeRecommendations(model, userID, 5)))
someRecommendations.foreach { case (userID, recsDF) =>
  val recommendedArtists = recsDF.select("artist").as[Int].collect()
  println(s"$userID -> ${recommendedArtists.mkString(", ")}")
}

...
1000190 -> 6694932, 435, 1005820, 58, 1244362
1001043 -> 1854, 4267, 1006016, 4468, 1274
1001129 -> 234, 1411, 1307, 189, 121
...
```

[![1](assets/1.png)](#co_recommending_music_and_the_audioscrobbler_dataset_CO4-1)

100个不同用户的子集

这里仅仅打印了推荐内容。它们也可以轻松写入到像[HBase](https://oreil.ly/SQImy)这样的外部存储中，在运行时提供快速查找。

# 接下来该做什么

当然，可以花更多时间调整模型参数，并查找和修复输入中的异常，比如`[unknown]`艺术家。例如，对播放计数的快速分析显示，用户2064012惊人地播放了艺术家4468达到了439,771次！艺术家4468是不太可能成功的另类金属乐队System of a Down，早些时候出现在推荐中。假设平均歌曲长度为4分钟，这相当于播放“Chop Suey！”和“B.Y.O.B.”等多达33年的时间！因为该乐队1998年开始制作唱片，这意味着要同时播放四到五首歌曲七年时间。这必须是垃圾信息或数据错误，是生产系统必须解决的真实世界数据问题的又一个例子。

ALS并不是唯一可能的推荐算法，但目前它是Spark MLlib支持的唯一算法。然而，MLlib也支持一种适用于非隐式数据的ALS变体。其使用方式相同，只是将`ALS`配置为`setImplicitPrefs(false)`。例如，当数据类似于评分而不是计数时，这种配置是合适的。从`ALSModel.transform`推荐方法返回的`prediction`列实际上是一个估计的评分。在这种情况下，简单的RMSE（均方根误差）指标适合用于评估推荐系统。

后来，Spark MLlib或其他库可能提供其他的推荐算法。

在生产环境中，推荐引擎通常需要实时进行推荐，因为它们被用于像电子商务网站这样的场景，客户在浏览产品页面时经常请求推荐。预先计算和存储推荐结果是一种在规模上提供推荐的合理方式。这种方法的一个缺点是它需要为可能很快需要推荐的所有用户预先计算推荐结果，而这些用户可能是任何用户中的一部分。例如，如果100万用户中只有10,000个用户在一天内访问网站，那么每天预先计算所有100万用户的推荐结果将浪费99%的努力。

我们最好根据需要即时计算推荐。虽然我们可以使用`ALSModel`为一个用户计算推荐，但这必然是一个分布式操作，需要几秒钟的时间，因为`ALSModel`实际上是一个非常庞大的分布式数据集。而其他模型则提供了更快的评分。
