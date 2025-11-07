# 第十一章：结构化流处理 Sinks

在前一章中，您学习了 sources，这是结构化流处理用于获取处理数据的抽象。在处理完这些数据后，我们可能希望对其进行某些操作。我们可能希望将其写入数据库以供后续查询，写入文件以进行进一步的（批处理）处理，或者将其写入另一个流式后端以保持数据的流动。

在结构化流处理中，*sinks* 是表示如何将数据传输到外部系统的抽象概念。结构化流处理内置了几个数据源，并定义了一个 API，使我们能够创建自定义的 sinks 来传输数据到不原生支持的其他系统。

本章中，我们将研究 sinks 的工作原理，审查结构化流处理提供的 sinks 的详细信息，并探讨如何创建自定义 sinks 来将数据写入不受默认实现支持的系统。

# 理解 Sinks

*Sinks* 作为结构化流处理中内部数据表示与外部系统之间的输出适配器。它们为流处理生成的数据提供写入路径。此外，它们还必须关闭可靠数据传递的循环。

要参与端到端可靠的数据传递，sinks 必须提供幂等写入操作。幂等意味着多次执行操作的结果等同于仅执行一次操作。在从故障中恢复时，Spark 可能会重新处理在故障发生时部分处理的数据。在源的一侧，这是通过使用重放功能来完成的。回想一下“理解 Sources”，可靠的 sources 必须提供重新播放未提交数据的方法，基于给定的偏移量。同样，sinks 必须提供在将记录写入外部源之前删除重复记录的方法。

重播源和幂等 sink 的组合赋予了结构化流处理其 *有效的仅一次* 数据传递语义。无法实现幂等性要求的 sinks 将导致至多“至少一次”语义的端到端传递保证。无法从流处理过程的故障中恢复的 sinks 被视为“不可靠”，因为它们可能会丢失数据。

在下一节中，我们将详细介绍结构化流处理中可用的 sinks。

###### 注意

第三方供应商可能会为其产品提供定制的结构化流处理 sinks。在将其中一个外部 sinks 集成到您的项目中时，请参考其文档以确定它们支持的数据传递保证。

# 可用的 Sinks

结构化流处理提供了几种输出，与支持的源相匹配，以及允许我们将数据输出到临时存储或控制台的输出。大致来说，我们可以将提供的输出分为可靠输出和学习/实验支持输出两类。此外，它还提供了一个可编程接口，允许我们与任意外部系统进行交互。

## 可靠输出

被视为可靠或适合生产的输出提供了明确定义的数据传输语义，并且对流处理过程的完全故障具有弹性。

提供的可靠输出如下：

文件输出

这将数据写入文件系统中的目录中的文件。它支持与文件源相同的文件格式：JSON、Parquet、逗号分隔值（CSV）和文本。

Kafka 输出

这将数据写入 Kafka，有效地保持数据“在移动”中。这是一个有趣的选择，可以将我们的处理结果与依赖 Kafka 作为数据主干的其他流处理框架集成。

## 用于实验的输出

提供以下输出以支持与结构化流处理的交互和实验。它们不提供故障恢复，因此在生产环境中使用这些输出是不鼓励的，因为可能导致数据丢失。

以下是非可靠输出：

内存输出

这将创建一个临时表，其中包含流查询的结果。生成的表可以在同一个 Java 虚拟机（JVM）进程内进行查询，从而允许集群内的查询访问流处理过程的结果。

控制台输出

这将查询的结果打印到控制台。在开发阶段，这对于直观地检查流处理结果非常有用。

## 输出 API

除了内置的输出，我们还可以选择以编程方式创建输出。这可以通过`foreach`操作实现，如其名称所示，它可以访问输出流的每个单独的结果记录。最后，可以直接使用`sink` API 开发自定义输出。

## 详细探讨输出

在本章的其余部分，我们探讨了每个输出的配置和可用选项。我们深入介绍了可靠输出，这应该提供了一个全面的应用视图，并且在你开始开发自己的应用程序时可以作为参考。

实验性输出在范围上有所限制，这也反映在后续覆盖的程度上。

在本章末尾，我们将查看自定义`sink` API 选项，并审查开发自己输出时需要考虑的事项。

###### 提示

如果您处于结构化流处理的初探阶段，您可能希望跳过本节，并在忙于开发自己的结构化流处理作业时稍后再回来查看。

# 文件输出

文件是常见的系统边界。当作为流处理的接收端使用时，它们允许数据在流导向处理后变得*静止*。这些文件可以成为*数据湖*的一部分，也可以被其他（批处理）过程消耗，作为结合了流式和批处理模式的更大处理流水线的一部分。

可扩展、可靠和分布式的文件系统——如 HDFS 或像亚马逊简单存储服务（Amazon S3）这样的对象存储——使得可以将大型数据集以任意格式存储为文件。在本地模式运行时，可以在探索或开发时使用本地文件系统作为这个接收端。

文件接收端支持与文件源相同的格式：

+   CSV

+   JSON

+   Parquet

+   ORC

+   Text

###### 注意

结构化流共享与批处理模式中使用的相同文件*数据源*实现。`DataFrameWriter`为每种文件格式提供的写入选项在流处理模式下也同样适用。在本节中，我们重点介绍最常用的选项。要获取最新的列表，请始终参考特定 Spark 版本的在线文档。

在深入探讨每种格式的细节之前，让我们先看一个通用的文件接收端示例，即示例 11-1 中的示例。

##### 示例 11-1\. 文件接收端示例

```
// assume an existing streaming dataframe df
val query = stream.writeStream
  .format("csv")
  .option("sep", "\t")
  .outputMode("append")
  .trigger(Trigger.ProcessingTime("30 seconds"))
  .option("path","<dest/path>")
  .option("checkpointLocation", "<checkpoint/path>")
  .start()
```

在这个例子中，我们使用`csv`格式将流结果写入到`<dest/path>`目标目录，使用`TAB`作为自定义分隔符。我们还指定了一个`checkpointLocation`，用于定期存储检查点元数据。

文件接收端仅支持`append`作为`outputMode`，并且在`writeStream`声明中可以安全地省略。尝试使用其他模式将导致查询启动时出现以下异常：`org.apache.spark.sql.AnalysisException: Data source ${format} does not support ${output_mode} output mode;`。

## 使用文件接收端的触发器

我们在示例 11-1 中看到的另一个参数是使用`trigger`。当没有指定触发器时，结构化流会在上一个批次完成后立即启动新批次的处理。对于文件接收端，根据输入流的吞吐量，这可能会导致生成许多小文件。这可能对文件系统的存储能力和性能有害。

考虑示例 11-2。

##### 示例 11-2\. 使用文件接收端的速率源

```
val stream = spark.readStream.format("rate").load()
val query = stream.writeStream
  .format("json")
  .option("path","/tmp/data/rate")
  .option("checkpointLocation", "/tmp/data/rate/checkpoint")
  .start()
```

如果让此查询运行一段时间，然后检查目标目录，应该会观察到大量小文件：

```
$ ls -1
part-00000-03a1ed33-3203-4c54-b3b5-dc52646311b2-c000.json
part-00000-03be34a1-f69a-4789-ad65-da351c9b2d49-c000.json
part-00000-03d296dd-c5f2-4945-98a8-993e3c67c1ad-c000.json
part-00000-0645a678-b2e5-4514-a782-b8364fb150a6-c000.json
...

# Count the files in the directory
$ ls -1 | wc -l
562

# A moment later
$ ls -1 | wc -l
608

# What's the content of a file?
$ cat part-00007-e74a5f4c-5e04-47e2-86f7-c9194c5e85fa-c000.json
{"timestamp":"2018-05-13T19:34:45.170+02:00","value":104}
```

正如我们在第十章中学到的，速率源默认每秒生成一条记录。当我们查看一个文件中包含的数据时，确实可以看到单个记录。事实上，查询每次新数据可用时生成一个文件。尽管该文件的内容不多，文件系统在跟踪文件数方面会有一些开销。在 Hadoop 分布式文件系统（HDFS）中，每个文件无论内容如何都会占用一个块，并复制`n`次。考虑到典型的 HDFS 块大小为 128 MB，我们可以看到我们使用文件汇集器的简单查询可能会迅速耗尽存储空间。

`trigger`配置是为了帮助我们避免这种情况。通过为文件生成提供时间触发器，我们可以确保每个文件中有足够的数据量。

我们可以通过修改前面的示例来观察时间`trigger`的效果如下：

```
import org.apache.spark.sql.streaming.Trigger

val stream = spark.readStream.format("rate").load()
val query = stream.writeStream
  .format("json")
  .trigger(Trigger.ProcessingTime("1 minute")) // <-- Add Trigger configuration
  .option("path","/tmp/data/rate")
  .option("checkpointLocation", "/tmp/data/rate/checkpoint")
  .start()
```

让我们发出查询，并等待几分钟。当我们检查目标目录时，应该比以前少很多文件，并且每个文件应该包含更多的记录。每个文件中的记录数取决于`DataFrame`的分区：

```
$ ls -1
part-00000-2ffc26f9-bd43-42f3-93a7-29db2ffb93f3-c000.json
part-00000-3cc02262-801b-42ed-b47e-1bb48c78185e-c000.json
part-00000-a7e8b037-6f21-4645-9338-fc8cf1580eff-c000.json
part-00000-ca984a73-5387-49fe-a864-bd85e502fd0d-c000.json
...

# Count the files in the directory
$ ls -1 | wc -l
34

# Some seconds later
$ ls -1 | wc -l
42

# What's the content of a file?

$ cat part-00000-ca984a73-5387-49fe-a864-bd85e502fd0d-c000.json
{"timestamp":"2018-05-13T22:02:59.275+02:00","value":94}
{"timestamp":"2018-05-13T22:03:00.275+02:00","value":95}
{"timestamp":"2018-05-13T22:03:01.275+02:00","value":96}
{"timestamp":"2018-05-13T22:03:02.275+02:00","value":97}
{"timestamp":"2018-05-13T22:03:03.275+02:00","value":98}
{"timestamp":"2018-05-13T22:03:04.275+02:00","value":99}
{"timestamp":"2018-05-13T22:03:05.275+02:00","value":100}
```

如果您在个人计算机上尝试此示例，则分区数默认为当前核心数。在我们的情况下，我们有八个核心，并且我们观察到每个分区有七到八条记录。虽然这仍然是非常少的记录，但它显示了可以推广到实际场景的原理。

尽管在这种情况下基于记录数或数据大小的`trigger`可能更有趣，但目前仅支持基于时间的触发器。随着结构化流的发展，这可能会发生变化。

## 所有支持的文件格式的常见配置选项

在之前的示例中，我们已经看到了使用方法`option`来设置汇合器中的配置选项的用法。

所有支持的文件格式共享以下配置选项：

`path`

流查询将数据文件写入目标文件系统中的目录。

`checkpointLocation`

可靠文件系统中存储检查点元数据的目录。每次查询执行间隔都会写入新的检查点信息。

`compression`（默认值：无）

所有支持的文件格式都可以压缩数据，尽管可用的压缩`codecs`可能因格式而异。每种格式的具体压缩算法在其相应部分中显示。

###### 注意

在配置文件汇总选项时，通常有必要记住，文件汇总写入的任何文件都可以使用相应的文件源读取。例如，当我们讨论 “JSON 文件源格式” 时，我们看到它通常期望文件的每一行是一个有效的 JSON 文档。同样，JSON 汇总格式将生成每行一个记录的文件。

## 常见的时间和日期格式化（CSV、JSON）

文本文件格式，如 CSV 和 JSON，接受日期和时间戳数据类型的自定义格式化：

`dateFormat`（默认：`yyyy-MM-dd`）

配置用于格式化`date`字段的模式。自定义模式遵循 [java.text.SimpleDateFormat](http://bit.ly/2VwrLku) 中定义的格式。

`timestampFormat`（默认：“yyyy-MM-dd’T’HH:mm:ss.SSSXXX”）

配置用于格式化`timestamp`字段的模式。自定义模式遵循 [java.text.SimpleDateFormat](http://bit.ly/2VwrLku) 中定义的格式。

`timeZone`（默认：本地时区）

配置用于格式化时间戳的时区。

## 文件汇总的 CSV 格式

使用 CSV 文件格式，我们可以以广泛支持的表格格式编写数据，可以被许多程序读取，从电子表格应用程序到各种企业软件。

### 选项

在 Spark 中，CSV 支持多种选项来控制字段分隔符、引用行为和包含头部信息。此外，通用的文件汇总选项和日期格式化选项也适用于 CSV 汇总。

###### 注意

在本节中，我们列出了最常用的选项。详细列表，请查看 [在线文档](http://bit.ly/2EdjVqe)。

以下是 CSV 汇总的常用选项：

`header`（默认：`false`）

一个标志，用于指示是否应将头部包含在生成的文件中。头部包含此流式`DataFrame`中字段的名称。

`quote`（默认：`"` [双引号]）

设置用于引用记录的字符。引用是必要的，当记录可能包含分隔符字符时，如果没有引用，将导致记录损坏。

`quoteAll`（默认：`false`）

用于指示是否应引用所有值或仅包含分隔符字符的标志。一些外部系统要求引用所有值。当使用 CSV 格式将生成的文件导入外部系统时，请检查该系统的导入要求以正确配置此选项。

`sep`（默认：`,` [逗号]）

配置用于字段之间的分隔符字符。分隔符必须是单个字符。否则，查询在运行时启动时会抛出`IllegalArgumentException`。

## JSON 文件汇总格式

JSON 文件汇允许我们使用 JSON Lines 格式将输出数据写入文件。该格式将输出数据集中的每个记录转换为一个有效的 JSON 文档，并写入一行文本中。JSON 文件汇对称地与 JSON 源相对应。正如我们预期的那样，使用此格式编写的文件可以通过 JSON 源再次读取。

###### 注意

当使用第三方 JSON 库读取生成的文件时，我们应该先将文件读取为文本行，然后将每行解析为表示一个记录的 JSON 文档。

### 选项

除了通用文件和文本格式选项之外，JSON 汇还支持这些特定的配置：

`encoding`（默认：`UTF-8`）

配置用于编写 JSON 文件的字符集编码。

`lineSep`（默认：`\n`）

设置要在 JSON 记录之间使用的行分隔符。

支持的`compression`选项（默认：`none`）：`none`、`bzip2`、`deflate`、`gzip`、`lz4`和`snappy`。

## Parquet 文件汇格式

Parquet 文件汇支持常见的文件汇配置，不具有特定于格式的选项。

支持的`compression`选项（默认：`snappy`）：`none`、`gzip`、`lzo`和`snappy`。

## 文本文件汇格式

文本文件汇写入纯文本文件。尽管其他文件格式会将流`DataFrame`或`Dataset`模式转换为特定的文件格式结构，文本汇期望的是一个展平的流`Dataset[String]`或带有单个`value`字段的流`DataFrame`，其类型为`StringType`。

文本文件格式的典型用法是编写在 Structured Streaming 中原生不支持的自定义基于文本的格式。为了实现这一目标，我们首先通过编程方式将数据转换为所需的文本表示形式。然后，我们使用文本格式将数据写入文件。试图将任何复杂模式写入文本汇将导致错误。

### 选项

除了汇和基于文本的格式的通用选项之外，文本汇支持以下配置选项：

`lineSep`（默认：`\n`）

配置用于终止每个文本行的行分隔符。

# Kafka 汇

正如我们在“Kafka Source”中讨论的那样，Kafka 是一个发布/订阅（pub/sub）系统。虽然 Kafka 源充当订阅者，但 Kafka 汇是发布者的对应物。Kafka 汇允许我们将数据写入 Kafka，然后其他订阅者可以消费这些数据，从而继续一系列流处理器的链条。

下游消费者可能是其他流处理器，使用 Structured Streaming 或任何其他可用的流处理框架实现，或者是（微）服务，用于消费企业生态系统中的流数据来支持应用程序。

## 理解 Kafka 发布模型

在 Kafka 中，数据表示为通过主题交换的键-值记录。主题由分布式分区组成。每个分区按接收顺序维护消息。此顺序由偏移量索引，消费者根据偏移量指示要读取的记录。当将记录发布到主题时，它被放置在主题的一个分区中。分区的选择取决于键。支配原则是具有相同键的记录将落在同一个分区中。因此，Kafka 中的排序是部分的。来自单个分区的记录序列将按到达时间顺序排序，但在分区之间没有排序保证。

这个模型具有高度的可伸缩性，Kafka 的实现确保低延迟的读写，使其成为流数据的优秀载体。

## 使用 Kafka Sink

现在你已经了解了 Kafka 发布模型，我们可以看看如何实际地将数据生产到 Kafka。我们刚刚看到 Kafka 记录结构化为键-值对。我们需要以相同的形式结构化我们的数据。

在最小实现中，我们必须确保我们的流`DataFrame`或`Dataset`具有`BinaryType`或`StringType`的`value`字段。这个要求的含义是，通常我们需要将数据编码成传输表示形式，然后再发送到 Kafka。

当未指定键时，结构化流将用`null`替换`key`。这使得 Kafka sink 使用循环分配来分配对应主题的分区。

如果我们想要保留对键分配的控制权，我们必须有一个`key`字段，也是`BinaryType`或`StringType`。这个`key`用于分区分配，从而确保具有相等键的记录之间的有序性。

可选地，我们可以通过添加一个`topic`字段来控制记录级别的目标主题。如果存在，`topic`的值必须对应于 Kafka 主题。在`writeStream`选项上设置`topic`会覆盖`topic`字段中的值。

相关记录将被发布到该主题。这个选项在实现分流模式时非常有用，其中传入的记录被分类到不同的专用主题，以供后续处理消费。例如，将传入的支持票据分类到专门的销售、技术和故障排除主题中，这些主题由相应的（微）服务在下游消费。

在我们把数据整理成正确的形状之后，我们还需要目标引导服务器的地址，以便连接到代理。

在实际操作中，这通常涉及两个步骤：

1.  将每个记录转换为名为`value`的单个字段，并可选择为每个记录分配一个键和一个主题。

1.  使用`writeStream`构建器声明我们的流目标。

示例 11-3 展示了这些步骤的使用。

##### 示例 11-3\. Kafka sink 示例

```
// Assume an existing streaming dataframe 'sensorData'
// with schema: id: String, timestamp: Long, sensorType: String, value: Double

// Create a key and a value from each record:

val kafkaFormattedStream = sensorData.select(
  $"id" as "key",
  to_json(
    struct($"id", $"timestamp", $"sensorType", $"value")
  ) as "value"
)

// In step two, we declare our streaming query:

val kafkaWriterQuery = kafkaFormat.writeStream
  .queryName("kafkaWriter")
  .outputMode("append")
  .format("kafka") // determines that the kafka sink is used
  .option("kafka.bootstrap.servers", kafkaBootstrapServer)
  .option("topic", targetTopic)
  .option("checkpointLocation", "/path/checkpoint")
  .option("failOnDataLoss", "false") // use this option when testing
  .start()
```

当我们在记录级别添加`topic`信息时，必须省略`topic`配置选项。

在 Example 11-4 中，我们修改了先前的代码，将每个记录写入与`sensorType`匹配的专用主题。即所有`humidity`记录进入 humidity 主题，所有`radiation`记录进入 radiation 主题，依此类推。

##### Example 11-4\. 将 Kafka 接收端写入不同主题

```
// assume an existing streaming dataframe 'sensorData'
// with schema: id: String, timestamp: Long, sensorType: String, value: Double

// Create a key, value and topic from each record:

val kafkaFormattedStream = sensorData.select(
  $"id" as "key",
  $"sensorType" as "topic",
  to_json(struct($"id", $"timestamp", $"value")) as "value"
)

// In step two, we declare our streaming query:

val kafkaWriterQuery = kafkaFormat.writeStream
  .queryName("kafkaWriter")
  .outputMode("append")
  .format("kafka") // determines that the kafka sink is used
  .option("kafka.bootstrap.servers", kafkaBootstrapServer)
  .option("checkpointLocation", "/path/checkpoint")
  .option("failOnDataLoss", "false") // use this option when testing
  .start()
```

请注意，我们已经移除了设置`option("topic", targetTopic)`，并且为每个记录添加了一个`topic`字段。这导致每个记录被路由到与其`sensorType`对应的主题。如果我们保留设置`option("topic", targetTopic)`，那么`topic`字段的值将不会起作用。`option("topic", targetTopic)`设置优先级更高。

### 选择编码方式

当我们仔细查看 Example 11-3 中的代码时，我们会看到我们通过将现有数据转换为其 JSON 表示来创建一个单一的`value`字段。在 Kafka 中，每个记录包含一个键和一个值。`value`字段包含记录的有效负载。为了向 Kafka 发送或接收任意复杂的记录，我们需要将该记录转换为一个单字段表示，以便将其放入`value`字段中。在结构化流中，必须通过用户代码完成从此传输值表示到实际记录的转换。理想情况下，我们选择的编码可以轻松转换为结构化记录，以利用 Spark 处理数据的能力。

常见的编码格式是 JSON。JSON 在 Spark 的结构化 API 中有原生支持，这一支持也扩展到了结构化流。正如我们在 Example 11-4 中看到的，我们通过使用 SQL 函数`to_json`来编写 JSON：`to_json(struct($"id", $"timestamp", $"value")) as "value")`。

二进制表示例如 AVRO 和 ProtoBuffers 也是可能的。在这种情况下，我们将`value`字段视为`BinaryType`，并使用第三方库进行编码/解码。

我们撰写本文时，尚未内置对二进制编码的支持，但已宣布将在即将推出的版本中支持 AVRO。

###### 警告

在选择编码格式时要考虑的一个重要因素是模式支持。在使用 Kafka 作为通信骨干的多服务模型中，通常会发现产生数据的服务使用与流处理器或其他消费者不同的编程模型、语言和/或框架。

为了确保互操作性，面向模式的编码是首选。具有模式定义允许在不同语言中创建工件，并确保生产的数据可以在后续被消费。

# 存储器接收端

Memory sink 是一个不可靠的输出端，将流处理的结果保存在内存中的临时表中。之所以被认为是不可靠的，是因为在流处理结束时将丢失所有数据，但在需要对流处理结果进行低延迟访问的场景中，它肯定是非常有用的。

由此输出端创建的临时表以查询名称命名。该表由流式查询支持，并将根据所选`outputMode`语义在每个触发器后更新。

结果表包含查询结果的最新视图，并可以使用经典的 Spark SQL 操作进行查询。查询必须在启动结构化流查询的同一进程（JVM）中执行。

由 Memory Sink 维护的表可以通过交互方式访问。这使得它成为与 Spark REPL 或笔记本等交互式数据探索工具理想的接口。

另一个常见用途是在流数据的顶部提供查询服务。这是通过将服务器模块（如 HTTP 服务器）与 Spark 驱动程序组合来完成的。然后，可以通过特定的 HTTP 端点调用来提供来自此内存表的数据。

示例 11-5 假设一个 `sensorData` 流数据集。流处理的结果被实例化到这个内存表中，该表在 SQL 上下文中可用作 `sample_memory_query`。

##### 示例 11-5\. Memory sink example

```
val sampleMemoryQuery = sensorData.writeStream
  .queryName("sample_memory_query")    // this query name will be the SQL table name
  .outputMode("append")
  .format("memory")
  .start()

// After the query starts we can access the data in the temp table
val memData = session.sql("select * from sample_memory_query")
memData.count() // show how many elements we have in our table
```

## 输出模式

Memory sink 支持所有输出模式：`Append`、`Update` 和 `Complete`。因此，我们可以将其与所有查询一起使用，包括聚合查询。Memory sink 与 `Complete` 模式的结合特别有趣，因为它提供了一个快速的、内存中可查询的存储，用于最新计算的完整状态。请注意，要支持 `Complete` 状态的查询，必须对有界基数键进行聚合，以确保处理状态的内存需求也在系统资源范围内受限。

# 控制台输出端（Console Sink）

对于所有喜欢在屏幕上输出“Hello, world!”的人，我们有控制台输出端。确实，控制台输出端允许我们将查询结果的一个小样本打印到标准输出。

它的使用仅限于交互式基于 shell 的环境中的调试和数据探索，比如`spark-shell`。正如我们预期的那样，这个输出端在没有将任何数据提交到另一个系统的情况下是不可靠的。

在生产环境中应避免使用控制台输出（Console sink），就像`println`在操作代码库中不被看好一样。

## 选项

下面是控制台输出端（Console sink）的可配置选项：

`numRows`（默认值：`20`）

每个查询触发器显示的最大行数。

`truncate`（默认值：`true`）

每一行单元格的输出是否应该被截断，有一个标志用来指示。

## 输出模式

从 Spark 2.3 开始，控制台输出端支持所有输出模式：`Append`、`Update` 和 `Complete`。

# Foreach Sink

有时我们需要将流处理应用程序与企业中的遗留系统集成。此外，作为一个年轻的项目，结构化流中可用接收器的范围相当有限。

`foreach`接收器包括一个 API 和接收器定义，提供对查询执行结果的访问。它将结构化流的写入能力扩展到任何提供 Java 虚拟机（JVM）客户端库的外部系统。

## ForeachWriter 接口

要使用`Foreach`接收器，我们必须提供`ForeachWriter`接口的实现。`ForeachWriter`控制写入操作的生命周期。其执行在执行器上分布，并且方法将针对流`DataFrame`或`Dataset`的每个分区调用，如示例 11-6 所示。

##### 示例 11-6\. ForeachWriter 的 API 定义

```
abstract class ForeachWriter[T] extends Serializable {

  def open(partitionId: Long, version: Long): Boolean

  def process(value: T): Unit

  def close(errorOrNull: Throwable): Unit

}
```

如我们在示例 11-6 中所见，`ForeachWriter`与流`Dataset`的类型`[T]`或在流`DataFrame`的情况下对应于`spark.sql.Row`绑定。其 API 包括三个方法：`open`，`process`和`close`：

`open`

每个触发间隔都会调用此方法，带有`partitionId`和唯一的`version`号。使用这两个参数，`ForeachWriter`必须决定是否处理所提供的分区。返回`true`将导致使用`process`方法中的逻辑处理每个元素。如果方法返回`false`，则跳过该分区的处理。

`process`

这提供对数据的访问，每次一个元素。应用于数据的函数必须产生副作用，比如将记录插入数据库，调用 REST API，或者使用网络库将数据通信到另一个系统。

`close`

该方法用于通知写入分区的结束。当此分区的输出操作成功终止时，`error`对象将为 null；否则，将包含一个`Throwable`。即使`open`返回`false`（表示不应处理该分区），也会在每个分区写入操作结束时调用`close`。

此合同是数据传递语义的一部分，因为它允许我们移除可能已经被发送到接收器但由于结构化流的恢复场景重新处理的重复分区。为了使该机制正常工作，接收器必须实现某种持久化方式来记住已经看到的`partition/version`组合。

在实现了我们的`ForeachWriter`之后，我们使用惯用的`writeStream`方法声明一个接收器，并调用带有`ForeachWriter`实例的专用`foreach`方法。

`ForeachWriter`的实现必须是`Serializable`。这是强制性的，因为`ForeachWriter`在处理流式`Dataset`或`DataFrame`的每个分区的每个节点上分布执行。在运行时，将为`Dataset`或`DataFrame`的每个分区创建一个新的反序列化的`ForeachWriter`实例的副本。因此，我们可能不会在`ForeachWriter`的初始构造函数中传递任何状态。

让我们将所有这些放在一个小示例中，展示 Foreach 接收器的工作方式，并说明处理状态处理和序列化要求的微妙复杂性。

## TCP Writer Sink: A Practical ForeachWriter Example

在这个示例中，我们将开发一个基于文本的 TCP 接收器，将查询结果传输到外部 TCP 套接字接收服务器。在这个示例中，我们将使用与 Spark 安装一起提供的`spark-shell`实用工具。

在示例 11-7 中，我们创建了一个简单的 TCP 客户端，可以连接并向服务器套接字写入文本，只要提供其`host`和`port`。请注意，这个类不是`Serializable`。`Socket`本质上是不可序列化的，因为它们依赖于底层系统的 I/O 端口。

##### 示例 11-7\. TCP 套接字客户端

```
class TCPWriter(host:String, port: Int) {
  import java.io.PrintWriter
  import java.net.Socket
  val socket = new Socket(host, port)
  val printer = new PrintWriter(socket.getOutputStream, true)
  def println(str: String) = printer.println(str)
  def close() = {
    printer.flush()
    printer.close()
    socket.close()
  }
}
```

接下来，在示例 11-8 中，我们将在`ForeachWriter`实现中使用这个`TCPWriter`。

##### 示例 11-8\. TCPForeachWriter 实现

```
import org.apache.spark.sql.ForeachWriter
class TCPForeachWriter(host: String, port: Int)
    extends ForeachWriter[RateTick] {

  @transient var writer: TCPWriter = _
  var localPartition: Long = 0
  var localVersion: Long = 0

  override def open(
      partitionId: Long,
      version: Long
    ): Boolean = {
    writer = new TCPWriter(host, port)
    localPartition = partitionId
    localVersion = version
    println(
      s"Writing partition [$partitionId] and version[$version]"
    )
    true // we always accept to write
  }

  override def process(value: RateTick): Unit = {
    val tickString = s"${v.timestamp}, ${v.value}"
    writer.println(
      s"$localPartition, $localVersion, $tickString"
    )
  }

  override def close(errorOrNull: Throwable): Unit = {
    if (errorOrNull == null) {
      println(
        s"Closing partition [$localPartition] and version[$localVersion]"
      )
      writer.close()
    } else {
      print("Query failed with: " + errorOrNull)
    }
  }
}
```

注意我们如何声明`TCPWriter`变量：`@transient var writer:TCPWriter = _`。`@transient`表示这个引用不应该被序列化。初始值为`null`（使用空变量初始化语法`_`）。只有在调用`open`时，我们才创建`TCPWriter`的实例，并将其分配给我们的变量以供稍后使用。

还要注意`process`方法如何接受`RateTick`类型的对象。当我们有一个类型化的`Dataset`时，实现`ForeachWriter`会更容易，因为我们处理特定的对象结构，而不是*流式 DataFrame*的通用数据容器`spark.sql.Row`。在这种情况下，我们将初始流式`DataFrame`转换为类型化的`Dataset[RateTick]`，然后继续到接收器阶段。

现在，为了完成我们的示例，我们创建一个简单的`Rate`数据源，并将产生的流直接写入我们新开发的`TCPForeachWriter`：

```
case class RateTick(timestamp: Long, value: Long)

val stream = spark.readStream.format("rate")
                  .option("rowsPerSecond", 100)
                  .load()
                  .as[RateTick]

val writerInstance = new TCPForeachWriter("localhost", 9876)

val query = stream
      .writeStream
      .foreach(writerInstance)
      .outputMode("append")
```

在开始我们的查询之前，我们运行一个简单的 TCP 服务器来观察结果。为此，我们使用`nc`，这是一个在命令行中创建 TCP/UDP 客户端和服务器的有用*nix 命令。在这种情况下，我们使用监听端口`9876`的 TCP 服务器：

```
# Tip: the syntax of netcat is system-specific.
# The following command should work on *nix and on OSX nc managed by homebrew.
# Check your system documentation for the proper syntax.
nc -lk 9876
```

最后，我们开始我们的查询：

```
val queryExecution = query.start()
```

在运行`nc`命令的 shell 中，我们应该看到类似以下的输出：

```
5, 1, 1528043018, 72
5, 1, 1528043018, 73
5, 1, 1528043018, 74
0, 1, 1528043018, 0
0, 1, 1528043018, 1
0, 1, 1528043018, 2
0, 1, 1528043018, 3
0, 1, 1528043018, 4
0, 1, 1528043018, 5
0, 1, 1528043018, 6
0, 1, 1528043018, 7
0, 1, 1528043018, 8
0, 1, 1528043018, 9
0, 1, 1528043018, 10
0, 1, 1528043018, 11
7, 1, 1528043019, 87
7, 1, 1528043019, 88
7, 1, 1528043019, 89
7, 1, 1528043019, 90
7, 1, 1528043019, 91
7, 1, 1528043019, 92
```

在输出中，第一列是`partition`，第二列是`version`，后面是`Rate`源产生的数据。有趣的是，数据在分区内是有序的，比如我们的示例中的`partition 0`，但在不同分区之间没有排序保证。分区在集群的不同机器上并行处理。没有保证哪一个先到达。

最后，为了结束查询执行，我们调用`stop`方法：

```
queryExecution.stop()
```

## 这个例子的教训

在这个例子中，您已经看到了如何正确使用一个简约的`socket`客户端来输出流查询的数据，使用 Foreach sink。Socket 通信是大多数数据库驱动程序和许多其他应用程序客户端的底层交互机制。我们在这里展示的方法是一个常见的模式，您可以有效地应用它来写入各种提供基于 JVM 的客户端库的外部系统。简而言之，我们可以总结这个模式如下：

1.  在`ForeachWriter`的主体中，创建一个`@transient`可变引用到我们的驱动类。

1.  在`open`方法中，初始化到外部系统的连接。将此连接分配给可变引用。保证此引用将被单个线程使用。

1.  在`process`中，将提供的数据元素发布到外部系统。

1.  最后，在`close`中，我们终止所有连接并清理任何状态。

## 故障排除`ForeachWriter`序列化问题

在示例 11-8 中，我们看到我们需要一个未初始化的可变引用到`TCPWriter`：`@transient var writer:TCPWriter = _`。这种看似复杂的结构是必要的，以确保我们只在`ForeachWriter`已经反序列化并在远程执行器上运行时实例化非可序列化类。

如果我们想探索在`ForeachWriter`实现中尝试包含非可序列化引用时发生的情况，我们可以像这样声明我们的`TCPWriter`实例：

```
import org.apache.spark.sql.ForeachWriter
class TCPForeachWriter(host: String, port: Int) extends ForeachWriter[RateTick] {

  val nonSerializableWriter:TCPWriter = new TCPWriter(host,port)
  // ... same code as before ...
}
```

尽管这看起来更简单、更熟悉，但当我们尝试使用此`ForeachWriter`实现运行查询时，我们会得到`org.apache.spark.SparkException: Task not serializable`。这会产生一个非常长的*堆栈跟踪*，其中包含对冒犯类的最佳努力指出。我们必须跟随堆栈跟踪，直到找到`Caused by`语句，如以下跟踪中所示：

```
Caused by: java.io.NotSerializableException: $line17.$read$$iw$$iw$TCPWriter
Serialization stack:
  - object not serializable (class: $line17.$read$$iw$$iw$TCPWriter,
      value: $line17.$read$$iw$$iw$TCPWriter@4f44d3e0)
  - field (class: $line20.$read$$iw$$iw$TCPForeachWriter,
      name: nonSerializableWriter, type: class $line17.$read$$iw$$iw$TCPWriter)
  - object (class $line20.$read$$iw$$iw$TCPForeachWriter,
      $line20.$read$$iw$$iw$TCPForeachWriter@54832ad9)
  - field (class: org.apache.spark.sql.execution.streaming.ForeachSink, name:
      org$apache$spark$sql$execution$streaming$ForeachSink$$writer,
      type: class org.apache.spark.sql.ForeachWriter)
```

正如本例在`spark-shell`中运行，我们发现一些奇怪的`$$`表示法，但当我们移除这些噪音时，我们可以看到非可序列化对象是`object not serializable (class: TCPWriter)`，其引用是字段`field name: nonSerializableWriter, type: class TCPWriter`。

`ForeachWriter` 实现中常见的序列化问题。希望通过本节的技巧，您能够在自己的实现中避免任何麻烦。但是，如果出现这种情况，Spark 将尽最大努力确定问题的源头。提供在堆栈跟踪中的这些信息对于调试和解决这些序列化问题非常有价值。
