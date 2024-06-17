# 第9章。分析基因组数据与BDG项目

下一代DNA测序（NGS）技术的出现迅速将生命科学转变为一个数据驱动的领域。然而，充分利用这些数据却遭遇到了一个传统的计算生态系统，其构建在难以使用的低级原语上，用于分布式计算，以及一大堆半结构化的基于文本的文件格式。

本章将有两个主要目的。首先，我们介绍一套流行的序列化和文件格式（Avro和Parquet），它们简化了数据管理中的许多问题。这些序列化技术使我们能够将数据转换为紧凑的、机器友好的二进制表示形式。这有助于在网络间移动数据，并帮助不同编程语言之间的跨兼容性。虽然我们将在基因组数据中使用数据序列化技术，但这些概念在处理大量数据时也是有用的。

其次，我们展示如何在PySpark生态系统中执行典型的基因组学任务。具体来说，我们将使用PySpark和开源的ADAM库来操作大量的基因组学数据，并处理来自多个来源的数据，创建一个用于预测转录因子（TF）结合位点的数据集。为此，我们将从[ENCODE数据集](https://oreil.ly/h0yOq)中获取基因组注释。本章将作为ADAM项目的教程，该项目包括一组面向基因组学的Avro模式、基于PySpark的API以及用于大规模基因组学分析的命令行工具。除了其他应用外，ADAM使用PySpark提供了[基因组分析工具包（GATK）](https://oreil.ly/k2YZH)的本地分布式实现。

我们将首先讨论生物信息学领域使用的各种数据格式，相关的挑战，以及序列化格式如何帮助解决问题。接着，我们将安装ADAM项目，并使用一个样本数据集探索其API。然后，我们将使用多个基因组学数据集准备一个数据集，用于预测特定类型蛋白质（CTCF转录因子）DNA序列中的结合位点。这些数据集将从公开可用的ENCODE数据集中获取。由于基因组暗示了一个一维坐标系统，许多基因组学操作具有空间性质。ADAM项目提供了一个针对基因组学的API，用于执行分布式空间连接。

对于那些感兴趣的人，Eric Lander的EdX课程《[生物学简介](https://oreil.ly/WIky1)》是一个很好的入门。而对于生物信息学的介绍，可以参考亚瑟·莱斯克的《[生物信息学简介](https://www.example.org/introduction_to_bioinformatics)》（牛津大学出版社）。

# 解耦存储与建模

生物信息学家花费了大量时间担心文件格式 —— *.fasta* ， *.fastq* ， *.sam* ， *.bam* ， *.vcf* ， *.gvcf* ， *.bcf* ， *.bed* ， *.gff* ， *.gtf* ， *.narrowPeak* ， *.wig* ， *.bigWig* ， *.bigBed* ， *.ped* 和 *.tped* 等等。一些科学家还觉得有必要为他们的自定义工具指定自己的自定义格式。此外，许多格式规范是不完整或含糊不清的（这使得很难确保实现是一致的或符合规范），并指定了 ASCII 编码的数据。ASCII 数据在生物信息学中非常常见，但效率低下且相对难以压缩。此外，数据必须始终进行解析，需要额外的计算周期。

这是特别令人担忧的，因为所有这些文件格式本质上只存储了几种常见的对象类型：对齐的序列读取，调用的基因型，序列特征和表型。（*序列特征* 这个术语在基因组学中有些多重含义，但在本章中我们指的是 UCSC 基因组浏览器的轨迹元素。）像 [`biopython`](http://biopython.org) 这样的库非常流行，因为它们充斥着解析器（例如， `Bio.SeqIO` ），这些解析器试图将所有文件格式读取到少量常见的内存模型中（例如， `Bio.Seq` ， `Bio.SeqRecord` ， `Bio.SeqFeature` ）。

我们可以使用像 Apache Avro 这样的序列化框架一举解决所有这些问题。关键在于 Avro 将数据模型（即显式模式）与底层存储文件格式以及语言的内存表示分离开来。Avro 指定了如何在进程之间传递特定类型的数据，无论是在互联网上运行的进程之间，还是尝试将数据写入特定文件格式的进程。例如，使用 Avro 的 Java 程序可以将数据写入多种与 Avro 数据模型兼容的底层文件格式。这使得每个进程都不再需要担心与多种文件格式的兼容性：进程只需要知道如何读取 Avro 数据，文件系统只需要知道如何提供 Avro 数据。

让我们以序列特征为例。我们首先使用 Avro 接口定义语言（IDL）为对象指定所需的模式：

```py
enum Strand {
  Forward,
  Reverse,
  Independent
}

record SequenceFeature {
  string featureId;
  string featureType; ![1](assets/1.png)
  string chromosome;
  long startCoord;
  long endCoord;
  Strand strand;
  double value;
  map<string> attributes;
}
```

[![1](assets/1.png)](#co_analyzing_genomics_data___span_class__keep_together__and_the_bdg_project__span__CO1-1)

例如，“保守性”，“蜈蚣”，“基因”

这种数据类型可以用来编码，例如，基因组中特定位置的保守水平、启动子或核糖体结合位点的存在、TF结合位点等。可以将其视为JSON的二进制版本，但更受限制且具有更高的性能。根据特定的数据模式，Avro规范确定了对象的精确二进制编码方式，以便可以在不同编程语言的进程之间轻松传输（甚至写入到不同编程语言的进程之间），通过网络或者存储到磁盘上。Avro项目包括用于处理多种语言（包括Java、C/C++、Python和Perl）中的Avro编码数据的模块；在此之后，语言可以自由地将对象存储在内存中，以最有利的方式。数据建模与存储格式的分离提供了另一种灵活性/抽象层次；Avro数据可以存储为Avro序列化的二进制对象（Avro容器文件）、用于快速查询的列式文件格式（Parquet文件）或者作为文本JSON数据以实现最大的灵活性（最小的效率）。最后，Avro支持模式演化，允许用户在需要时添加新字段，同时软件会优雅地处理新/旧版本的模式。

总体上，Avro是一种高效的二进制编码，允许您指定可适应变化的数据模式，从多种编程语言处理相同的数据，并使用多种格式存储数据。决定使用Avro模式存储数据将使您摆脱永远使用越来越多的自定义数据格式的困境，同时提高计算性能。

在前面的例子中使用的特定`SequenceFeature`模型对于真实数据来说有些简单，但是Big Data Genomics（BDG）项目已经定义了Avro模式来表示以下对象以及许多其他对象：

+   `AlignmentRecord` 用于读取。

+   `Variant`用于已知的基因组变异和元数据。

+   `Genotype`用于特定位点的已调用基因型。

+   `Feature`用于序列特征（基因组片段上的注释）。

实际的模式可以在[bdg-formats GitHub存储库](https://oreil.ly/gCf1f)中找到。BDG格式可以用作广泛使用的“传统”格式（如BAM和VCF）的替代，但更常用作高性能的“中间”格式。（这些BDG格式的最初目标是取代BAM和VCF的使用，但它们的固执普及性使得实现这一目标变得困难。）Avro相对于自定义ASCII格式提供了许多性能和数据建模的优势。

在本章的其余部分，我们将使用一些BDG模式来完成一些典型的基因组学任务。在我们能够做到这一点之前，我们需要安装ADAM项目。这将是我们接下来将要做的事情。

# ADAM的设置

BDG的核心基因组工具称为ADAM。从一组映射的读取开始，该核心包括可以执行标记重复、基质质量分数重新校准、插入/缺失重对齐和变体调用等任务的工具。ADAM还包含一个命令行界面，用于简化使用。与传统的HPC工具不同，ADAM可以自动跨集群并行处理，无需手动分割文件或手动调度作业。

我们可以通过pip安装ADAM：

```py
pip3 install bdgenomics.adam
```

可以在[GitHub页面](https://oreil.ly/4eFnX)找到其他安装方法。

ADAM还配备了一个提交脚本，可方便与Spark的`spark-submit`脚本进行交互：

```py
adam-submit
...

Using ADAM_MAIN=org.bdgenomics.adam.cli.ADAMMain
Using spark-submit=/home/analytical-monk/miniconda3/envs/pyspark/bin/spark-submit

       e        888~-_         e            e    e
      d8b       888   \       d8b          d8b  d8b
     /Y88b      888    |     /Y88b        d888bdY88b
    /  Y88b     888    |    /  Y88b      / Y88Y Y888b
   /____Y88b    888   /    /____Y88b    /   YY   Y888b
  /      Y88b   888_-~    /      Y88b  /          Y888b

Usage: adam-submit [<spark-args> --] <adam-args>

Choose one of the following commands:

ADAM ACTIONS
          countKmers : Counts the k-mers/q-mers from a read dataset...
     countSliceKmers : Counts the k-mers/q-mers from a slice dataset...
 transformAlignments : Convert SAM/BAM to ADAM format and optionally...
   transformFeatures : Convert a file with sequence features into...
  transformGenotypes : Convert a file with genotypes into correspondi...
  transformSequences : Convert a FASTA file as sequences into corresp...
     transformSlices : Convert a FASTA file as slices into correspond...
   transformVariants : Convert a file with variants into correspondin...
         mergeShards : Merges the shards of a fil...
            coverage : Calculate the coverage from a given ADAM fil...
CONVERSION OPERATION
          adam2fastq : Convert BAM to FASTQ file
  transformFragments : Convert alignments into fragment records
PRIN
               print : Print an ADAM formatted fil
            flagstat : Print statistics on reads in an ADAM file...
                view : View certain reads from an alignment-record file.
```

现在，您应该能够从命令行运行ADAM并获得使用消息。正如使用消息中所指出的，Spark参数在ADAM特定参数之前给出。

安装好ADAM后，我们可以开始处理基因组数据。接下来，我们将通过处理一个示例数据集来探索ADAM的API。

# 使用ADAM处理基因组数据入门

我们将从包含一些映射的NGS读取的*.bam*文件开始，将其转换为相应的BDG格式（在本例中为`AlignedRecord`），并保存到HDFS中。首先，我们找到一个合适的*.bam*文件：

```py
# Note: this file is 16 GB
curl -O ftp://ftp.ncbi.nlm.nih.gov/1000genomes/ftp/phase3/data\
/HG00103/alignment/HG00103.mapped.ILLUMINA.bwa.GBR\
.low_coverage.20120522.bam

# or using Aspera instead (which is *much* faster)
ascp -i path/to/asperaweb_id_dsa.openssh -QTr -l 10G \
anonftp@ftp.ncbi.nlm.nih.gov:/1000genomes/ftp/phase3/data\
/HG00103/alignment/HG00103.mapped.ILLUMINA.bwa.GBR\
.low_coverage.20120522.bam .
```

将下载的文件移动到一个目录中，我们将在这一章节中存储所有数据：

```py
mv HG00103.mapped.ILLUMINA.bwa.GBR\
.low_coverage.20120522.bam data/genomics
```

接下来，我们将使用ADAM CLI。

## 使用ADAM CLI进行文件格式转换

然后，我们可以使用ADAM的`transform`命令将*.bam*文件转换为Parquet格式（在[“Parquet格式和列式存储”](#parquet-format)中有描述）。这在集群和`local`模式下都可以使用：

```py
adam-submit \
  --master yarn \ ![1](assets/1.png)
  --deploy-mode client \
  --driver-memory 8G \
  --num-executors 6 \
  --executor-cores 4 \
  --executor-memory 12G \
  -- \
  transform \ ![2](assets/2.png)
  data/genomics/HG00103.mapped.ILLUMINA.bwa.GBR\ .low_coverage.20120522.bam \
  data/genomics/HG00103
```

[![1](assets/1.png)](#co_analyzing_genomics_data___span_class__keep_together__and_the_bdg_project__span__CO2-1)

在YARN上运行的示例Spark参数

[![2](assets/2.png)](#co_analyzing_genomics_data___span_class__keep_together__and_the_bdg_project__span__CO2-2)

ADAM子命令本身

这应该会启动相当大量的输出到控制台，包括跟踪作业进度的URL。

结果数据集是*data/genomics/reads/HG00103/*目录中所有文件的串联，其中每个*part-*.parquet*文件是一个PySpark任务的输出。您还会注意到，由于列式存储，数据比初始的*.bam*文件（底层为gzipped）压缩得更有效（请参见[“Parquet格式和列式存储”](#parquet-format)）。

```py
$ du -sh data/genomics/HG00103*bam
16G  data/genomics/HG00103\. [...] .bam

$ du -sh data/genomics/HG00103/
13G  data/genomics/HG00103
```

让我们看看交互会话中的一个对象是什么样子。

## 使用PySpark和ADAM摄取基因组数据

首先，我们使用ADAM助手命令启动PySpark shell。它加载所有必需的JAR文件。

```py
pyadam

...

[...]
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _  / __/   _/
   /__ / .__/\_,_/_/ /_/\_\   version 3.2.1
      /_/

Using Python version 3.6.12 (default, Sep  8 2020 23:10:56)
Spark context Web UI available at http://192.168.29.60:4040
Spark context available as 'sc'.
SparkSession available as 'spark'.

>>>
```

在某些情况下，当尝试使用ADAM与PySpark时，您可能会遇到TypeError错误，并提到JavaPackage对象未被加载的问题。这是一个已知问题，并在[此处](https://oreil.ly/67uBd)有文档记录。

在这种情况下，请尝试线程中建议的解决方案。可以通过运行以下命令来启动带有ADAM的PySpark shell：

```py
!pyspark --conf spark.serializer=org.apache.spark.
serializer.KryoSerializer --conf spark.kryo.registrator=
org.bdgenomics.adam.serialization.ADAMKryoRegistrator
--jars `find-adam-assembly.sh` --driver-class-path
`find-adam-assembly.sh`
```

现在我们将加载对齐的读取数据作为`AlignmentDataset`：

```py
from bdgenomics.adam.adamContext import ADAMContext

ac = ADAMContext(spark)

readsData = ac.loadAlignments("data/HG00103")

readsDataDF = readsData.toDF()
readsDataDF.show(1, vertical=True)

...

-RECORD 0-----------------------------------------
 referenceName             | hs37d5
 start                     | 21810734
 originalStart             | null
 end                       | 21810826
 mappingQuality            | 0
 readName                  | SRR062640.14600566
 sequence                  | TCCATTCCACTCAGTTT...
 qualityScores             | /MOONNCRQPIQIKRGL...
 cigar                     | 92M8S
 originalCigar             | null
 basesTrimmedFromStart     | 0
 basesTrimmedFromEnd       | 0
 readPaired                | true
 properPair                | false
 readMapped                | false
 mateMapped                | true
 failedVendorQualityChecks | false
 duplicateRead             | false
 readNegativeStrand        | false
 mateNegativeStrand        | false
 primaryAlignment          | true
 secondaryAlignment        | false
 supplementaryAlignment    | false
 mismatchingPositions      | null
 originalQualityScores     | null
 readGroupId               | SRR062640
 readGroupSampleId         | HG00103
 mateAlignmentStart        | 21810734
 mateReferenceName         | hs37d5
 insertSize                | null
 readInFragment            | 1
 attributes                | RG:Z:SRR062640\tX...
only showing top 1 row
```

您可能会得到不同的读取，因为数据的分区在您的系统上可能不同，所以无法保证哪个读取会先返回。

现在我们可以与数据集互动地提出问题，同时在后台集群中执行计算。这个数据集有多少个读取数？

```py
readsData.toDF().count()
...
160397565
```

这个数据集的读取是否来自所有人类染色体？

```py
unique_chr = readsDataDF.select('referenceName').distinct().collect()
unique_chr = [u.referenceName for u in unique_chr]

unique_chr.sort()
...
1
10
11
12
[...]
GL000249.1
MT
NC_007605
X
Y
hs37d5
```

是的，我们观察到来自染色体1到22、X和Y的读取，以及一些其他不属于“主”染色体的染色体片段或位置未知的染色体块。让我们更仔细地分析一下代码：

```py
readsData = ac.loadAlignments("data/HG00103") ![1](assets/1.png)

readsDataDF = readsData.toDF() ![2](assets/2.png)

unique_chr = readsDataDF.select('referenceName').distinct(). \ ![3](assets/3.png)
              collect() ![4](assets/4.png)
```

[![1](assets/1.png)](#co_analyzing_genomics_data___span_class__keep_together__and_the_bdg_project__span__CO3-1)

`AlignmentDataset`：包含所有数据的ADAM类型。

[![2](assets/2.png)](#co_analyzing_genomics_data___span_class__keep_together__and_the_bdg_project__span__CO3-2)

`DataFrame`：底层的Spark DataFrame。

[![3](assets/3.png)](#co_analyzing_genomics_data___span_class__keep_together__and_the_bdg_project__span__CO3-3)

这将聚合所有不同的contig名称；这将很小。

[![4](assets/4.png)](#co_analyzing_genomics_data___span_class__keep_together__and_the_bdg_project__span__CO3-4)

这将触发计算，并将DataFrame中的数据返回到客户端应用程序（Shell）。

举个更临床的例子，假设我们正在测试一个个体的基因组，以检查他们是否携带任何可能导致其子女患囊性纤维化（CF）的基因变异。我们的基因检测使用下一代DNA测序从多个相关基因生成读取数，比如CFTR基因（其突变可以导致CF）。在通过我们的基因分型流程运行数据后，我们确定CFTR基因似乎有一个导致其功能受损的早期终止密码子。然而，这种突变在[Human Gene Mutation Database](https://oreil.ly/wULRR)中从未报告过，也不在[Sickkids CFTR database](https://oreil.ly/u1L0j)中（这是聚合CF基因变异的数据库）。我们想回到原始测序数据，以查看可能有害的基因型调用是否为假阳性。为此，我们需要手动分析所有映射到该变异位点的读数，比如染色体7上的117149189位置（参见[图 9-1](#IGV_HG00103_CFTR)）：

```py
from pyspark.sql import functions as fun
cftr_reads = readsDataDF.where("referenceName == 7").\
              where(fun.col("start") <= 117149189).\
              where(fun.col("end") > 117149189)

cftr_reads.count()
...

9
```

现在可以手动检查这九个读取，或者例如通过自定义对齐器处理它们，并检查报告的致病变异是否为假阳性。

![aaps 0901](assets/aaps_0901.png)

###### 图 9-1\. 在CFTR基因中chr7:117149189处的HG00103的综合基因组查看器可视化

假设我们正在运营一个临床实验室，为临床医生提供携带者筛查服务。使用像AWS S3这样的云存储系统存档原始数据可以确保数据保持相对温暖（相对于磁带存档来说）。除了有一个可靠的数据处理系统外，我们还可以轻松访问所有过去的数据进行质量控制，或者在需要手动干预的情况下，例如之前介绍的CFTR案例。除了快速访问所有数据的便利性外，中心化还使得执行大规模分析研究（如人群遗传学、大规模质量控制分析等）变得更加容易。

现在我们已经熟悉了ADAM API，让我们开始创建我们的转录因子预测数据集。

# 从ENCODE数据预测转录因子结合位点

在这个例子中，我们将使用公开可用的序列特征数据来构建一个简单的转录因子结合模型。TFs是结合基因组中特定DNA序列的蛋白质，并帮助控制不同基因的表达。因此，它们在确定特定细胞的表型方面至关重要，并参与许多生理和疾病过程。ChIP-seq是一种基于NGS的分析方法，允许对特定细胞/组织类型中特定TF结合位点进行全基因组特征化。然而，除了ChIP-seq的成本和技术难度之外，它还需要为每种组织/TF组合进行单独的实验。相比之下，DNase-seq是一种在全基因组范围内查找开放染色质区域的分析方法，并且只需每种组织类型执行一次。与为每种组织/TF组合执行ChIP-seq实验以测定TF结合位点不同，我们希望在仅有DNase-seq数据的情况下预测新组织类型中的TF结合位点。

特别是，我们将使用来自[HT-SELEX](https://oreil.ly/t5OEkL)的已知序列模体数据和其他来自[公开可用的ENCODE数据集](https://oreil.ly/eFJ9n)的DNase-seq数据来预测CTCF TF的结合位点。我们选择了六种不同的细胞类型，这些类型具有可用的DNase-seq和CTCF ChIP-seq数据用于训练。训练示例将是一个DNase敏感性（HS）峰（基因组的一个片段），TF是否结合/未结合的二进制标签将从ChIP-seq数据中导出。

总结整体数据流程：主要的训练/测试样本将从DNase-seq数据中派生。每个开放染色质区域（基因组上的一个区间）将用于生成是否会在那里结合特定组织类型中的特定转录因子的预测。为此，我们将ChIP-seq数据空间连接到DNase-seq数据；每个重叠都是DNase序列对象的正标签。最后，为了提高预测准确性，我们在DNase-seq数据的每个区间中生成一个额外的特征——到转录起始位点的距离（使用GENCODE数据集）。通过执行空间连接（可能进行聚合），将该特征添加到训练样本中。

我们将使用以下细胞系的数据：

GM12878

常研究的淋巴母细胞系

K562

女性慢性髓系白血病

BJ

皮肤成纤维细胞

HEK293

胚胎肾

H54

胶质母细胞瘤

HepG2

肝细胞癌

首先，我们下载每个细胞系的DNase数据，格式为*.narrowPeak*：

```py
mkdir data/genomics/dnase

curl -O -L "https://www.encodeproject.org/ \
              files/ENCFF001UVC/@@download/ENCFF001UVC.bed.gz" | \
              gunzip > data/genomics/dnase/GM12878.DNase.narrowPeak ![1](assets/1.png)
curl -O -L "https://www.encodeproject.org/ \
              files/ENCFF001UWQ/@@download/ENCFF001UWQ.bed.gz" | \
              gunzip > data/genomics/dnase/K562.DNase.narrowPeak
curl -O -L "https://www.encodeproject.org/ \
              files/ENCFF001WEI/@@download/ENCFF001WEI.bed.gz" | \
              gunzip > data/genomics/dnase/BJ.DNase.narrowPeak
curl -O -L "https://www.encodeproject.org/ \
              files/ENCFF001UVQ/@@download/ENCFF001UVQ.bed.gz" | \
              gunzip > data/genomics/dnase/HEK293.DNase.narrowPeak
curl -O -L "https://www.encodeproject.org/ \
            files/ENCFF001SOM/@@download/ENCFF001SOM.bed.gz" | \
            gunzip > data/genomics/dnase/H54.DNase.narrowPeak
curl -O -L "https://www.encodeproject.org/ \
            files/ENCFF001UVU/@@download/ENCFF001UVU.bed.gz" | \
            gunzip > data/genomics/dnase/HepG2.DNase.narrowPeak

[...]
```

[![1](assets/1.png)](#co_analyzing_genomics_data___span_class__keep_together__and_the_bdg_project__span__CO4-1)

流式解压缩

接下来，我们下载CTCF TF的ChIP-seq数据，也是*.narrowPeak*格式，以及GTF格式的GENCODE数据：

```py
mkdir data/genomics/chip-seq

curl -O -L "https://www.encodeproject.org/ \
 files/ENCFF001VED/@@download/ENCFF001VED.bed.gz" | \
            gunzip > data/genomics/chip-seq/GM12878.ChIP-seq.CTCF.narrowPeak
curl -O -L "https://www.encodeproject.org/ \
 files/ENCFF001VMZ/@@download/ENCFF001VMZ.bed.gz" | \
            gunzip > data/genomics/chip-seq/K562.ChIP-seq.CTCF.narrowPeak
curl -O -L "https://www.encodeproject.org/ \
 files/ENCFF001XMU/@@download/ENCFF001XMU.bed.gz" | \
            gunzip > data/genomics/chip-seq/BJ.ChIP-seq.CTCF.narrowPeak
curl -O -L "https://www.encodeproject.org/ \
 files/ENCFF001XQU/@@download/ENCFF001XQU.bed.gz" | \
            gunzip > data/genomics/chip-seq/HEK293.ChIP-seq.CTCF.narrowPeak
curl -O -L "https://www.encodeproject.org/ \
 files/ENCFF001USC/@@download/ENCFF001USC.bed.gz" | \
            gunzip> data/genomics/chip-seq/H54.ChIP-seq.CTCF.narrowPeak
curl -O -L "https://www.encodeproject.org/ \
 files/ENCFF001XRC/@@download/ENCFF001XRC.bed.gz" | \
            gunzip> data/genomics/chip-seq/HepG2.ChIP-seq.CTCF.narrowPeak

curl -s -L "http://ftp.ebi.ac.uk/pub/databases/gencode/\
 Gencode_human/release_18/gencode.v18.annotation.gtf.gz" | \
            gunzip > data/genomics/gencode.v18.annotation.gtf
[...]
```

请注意，我们如何在将数据存入文件系统的过程中使用`gunzip`解压缩数据流。

从所有这些原始数据中，我们希望生成一个具有以下模式的训练集：

1.  染色体

1.  起始位点

1.  结束

1.  距离最近的转录起始位点（TSS）

1.  TF标识（在本例中始终为“CTCF”）

1.  细胞系

1.  TF结合状态（布尔值；目标变量）

该数据集可以轻松转换为DataFrame，用于输入到机器学习库中。由于我们需要为多个细胞系生成数据，我们将为每个细胞系单独定义一个DataFrame，并在最后进行连接：

```py
cell_lines = ["GM12878", "K562", "BJ", "HEK293", "H54", "HepG2"]
for cell in cell_lines:
## For each cell line…
  ## …generate a suitable DataFrame
## Concatenate the DataFrames and carry through into MLlib, for example
```

我们定义一个实用函数和一个广播变量，用于生成特征：

```py
local_prefix = "data/genomics"
import pyspark.sql.functions as fun

## UDF for finding closest transcription start site
## naive; exercise for reader: make this faster
def distance_to_closest(loci, query):
  return min([abs(x - query) for x in loci])
distance_to_closest_udf = fun.udf(distance_to_closest)

## build in-memory structure for computing distance to TSS
## we are essentially implementing a broadcast join here
tss_data = ac.loadFeatures("data/genomics/gencode.v18.annotation.gtf")
tss_df = tss_data.toDF().filter(fun.col("featureType") == 'transcript')
b_tss_df = spark.sparkContext.broadcast(tss_df.groupBy('referenceName').\
                agg(fun.collect_list("start").alias("start_sites")))
```

现在，我们已经加载了定义我们训练样本所需的数据，我们定义了计算每个细胞系数据的“循环”的主体。请注意我们如何读取ChIP-seq和DNase数据的文本表示，因为这些数据集并不大，不会影响性能。

为此，我们加载DNase和ChIP-seq数据：

```py
current_cell_line = cell_lines[0]

dnase_path = f'data/genomics/dnase/{current_cell_line}.DNase.narrowPeak'
dnase_data = ac.loadFeatures(dnase_path) ![1](assets/1.png)
dnase_data.toDF().columns ![2](assets/2.png)
...
['featureId', 'sampleId', 'name', 'source', 'featureType', 'referenceName',
'start', 'end', 'strand', 'phase', 'frame', 'score', 'geneId', 'transcriptId',
'exonId', 'proteinId', 'aliases', 'parentIds', 'target', 'gap', 'derivesFrom',
'notes', 'dbxrefs', 'ontologyTerms', 'circular', 'attributes']

...

chip_seq_path = f'data/genomics/chip-seq/ \
                  {current_cell_line}.ChIP-seq.CTCF.narrowPeak'
chipseq_data = ac.loadFeatures(chipseq_path) ![1](assets/1.png)
```

[![1](assets/1.png)](#co_analyzing_genomics_data___span_class__keep_together__and_the_bdg_project__span__CO5-1)

`FeatureDataset`

[![2](assets/2.png)](#co_analyzing_genomics_data___span_class__keep_together__and_the_bdg_project__span__CO5-2)

Dnase DataFrame中的列

与 `chipseq_data` 中的 `ReferenceRegion` 重叠的位点具有 TF 结合位点，因此标记为 `true`，而其余的位点标记为 `false`。这是通过 ADAM API 中提供的 1D 空间连接原语来实现的。连接功能需要一个按 `ReferenceRegion` 键入的 RDD，并将生成根据通常连接语义（例如内连接与外连接）重叠区域的元组。

```py
dnase_with_label = dnase_data.leftOuterShuffleRegionJoin(chipseq_data)
dnase_with_label_df = dnase_with_label.toDF()
...

-RECORD 0----------------------------------------------------------------------..
 _1  | {null, null, chr1.1, null, null, chr1, 713841, 714424, INDEPENDENT, null..
 _2  | {null, null, null, null, null, chr1, 713945, 714492, INDEPENDENT, null, ..
-RECORD 1----------------------------------------------------------------------..
 _1  | {null, null, chr1.2, null, null, chr1, 740179, 740374, INDEPENDENT, null..
 _2  | {null, null, null, null, null, chr1, 740127, 740310, INDEPENDENT, null, ..
-RECORD 2----------------------------------------------------------------------..
 _1  | {null, null, chr1.3, null, null, chr1, 762054, 763213, INDEPENDENT, null..
 _2  | null...
only showing top 3 rows
...

dnase_with_label_df = dnase_with_label_df.\
                        withColumn("label", \
                                    ~fun.col("_2").isNull())
dnase_with_label_df.show(5)
```

现在我们在每个 DNase 峰上计算最终的特征集：

```py
## build final training DF
training_df = dnase_with_label_df.withColumn(
    "contig", fun.col("_1").referenceName).withColumn(
    "start", fun.col("_1").start).withColumn(
    "end", fun.col("_1").end).withColumn(
    "tf", fun.lit("CTCF")).withColumn(
    "cell_line", fun.lit(current_cell_line)).drop("_1", "_2")

training_df = training_df.join(b_tss_df,
                               training_df.contig == b_tss_df.referenceName,
                               "inner") ![1](assets/1.png)

training_df.withColumn("closest_tss",
                      fun.least(distance_to_closest_udf(fun.col("start_sites"),
                                                        fun.col("start")),
                          distance_to_closest_udf(fun.col("start_sites"),
                                                  fun.col("end")))) ![2](assets/2.png)
```

[![1](assets/1.png)](#co_analyzing_genomics_data___span_class__keep_together__and_the_bdg_project__span__CO6-1)

与之前创建的 `tss_df` 进行左连接。

[![2](assets/2.png)](#co_analyzing_genomics_data___span_class__keep_together__and_the_bdg_project__span__CO6-2)

获取最接近的 TSS 距离。

在循环遍历各个细胞系时，每次通过后计算这个最终的 DF。最后，我们将每个细胞系的每个 DF 合并并将这些数据缓存在内存中，以便为训练模型做准备。

```py
preTrainingData = data_by_cellLine.union(...)
preTrainingData.cache()

preTrainingData.count()
preTrainingData.filter(fun.col("label") == true).count()
```

此时，`preTrainingData` 中的数据可以被归一化并转换为一个 DataFrame，用于训练分类器，如 [“Random Forests”](ch04.xhtml#RandomDecisionForests) 中所述。请注意，应执行交叉验证，在每个折叠中，您应保留一个细胞系的数据。

# 接下来该去哪里

许多基因组学中的计算都很适合于 PySpark 的计算范式。当您进行即席分析时，像 ADAM 这样的项目最有价值的贡献是提供了表示底层分析对象的 Avro 模式集合（以及转换工具）。我们看到一旦数据转换为相应的 Avro 模式，许多大规模计算变得相对容易表达和分布。

尽管在 PySpark 上进行科学研究的工具可能仍然相对匮乏，但确实存在一些项目，可以帮助避免重复造轮子。我们探索了 ADAM 中实现的核心功能，但该项目已经为整个 GATK 最佳实践管道（包括插入缺失重整和去重复）提供了实现。除了 ADAM 外，Broad Institute 现在还在使用 Spark 开发主要软件项目，包括 [GATK4](https://oreil.ly/hGR87) 的最新版本和一个名为 [Hail](https://oreil.ly/V6Wpl) 的用于大规模人群遗传学计算的项目。所有这些工具都是开源的，因此如果您开始在自己的工作中使用它们，请考虑贡献改进！
