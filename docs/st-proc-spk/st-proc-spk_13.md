# 第九章：结构化流实战

现在我们更好地理解了结构化流 API 和编程模型，在本章中，我们创建了一个小而完整的受物联网（IoT）启发的流程程序。

# 在线资源

作为示例，我们将使用书籍在线资源中的`Structured-Streaming-in-action`笔记本，位于[*https://github.com/stream-processing-with-spark*](https://github.com/stream-processing-with-spark)。

我们的用例将是作为流源从 Apache Kafka 消耗传感器读数流。

我们将会将传入的 IoT 传感器数据与包含所有已知传感器及其配置的静态参考文件进行关联。通过这种方式，我们会为每个传入的记录添加特定的传感器参数，以便处理报告的数据。然后，我们将所有处理正确的记录保存到 Parquet 格式的文件中。

# Apache Kafka

Apache Kafka 是最流行的可扩展消息代理之一，用于在事件驱动系统中解耦生产者和消费者。它是基于分布式提交日志抽象的高度可伸缩的分布式流平台。它提供类似于消息队列或企业消息系统的功能，但在以下三个重要领域与其前身有所不同：

+   运行在商品集群上，使其具有高度可伸缩性。

+   容错数据存储确保数据接收和传递的一致性。

+   拉取型消费者允许在不同的时间和速率下消费数据，从实时到微批到批处理，从而为各种应用程序提供数据提供的可能性。

您可以在[*http://kafka.apache.org*](http://kafka.apache.org)找到 Kafka。

# 消费流源

我们程序的第一部分涉及创建流`Dataset`：

```
val rawData = sparkSession.readStream
      .format("kafka")
      .option("kafka.bootstrap.servers", kafkaBootstrapServer)
      .option("subscribe", topic)
      .option("startingOffsets", "earliest")
      .load()

> rawData: org.apache.spark.sql.DataFrame
```

结构化流的入口是现有的 Spark Session（`sparkSession`）。正如您在第一行所看到的那样，创建流`Dataset`几乎与创建使用`read`操作的静态`Dataset`相同。`sparkSession.readStream`返回一个`DataStreamReader`，这是一个实现构建器模式以收集构建流源所需信息的类，使用*流畅*API。在该 API 中，我们找到了`format`选项，让我们指定我们的源提供程序，而在我们的情况下，这是`kafka`。随后的选项特定于源：

`kafka.bootstrap.servers`

指示要联系的引导服务器集的集合，作为逗号分隔的`host:port`地址列表

`subscribe`

指定要订阅的主题或主题集

`startingOffsets`

当此应用程序刚开始运行时应用的偏移复位策略

我们将在第十章中详细介绍 Kafka 流处理提供程序。

`load()` 方法评估 `DataStreamReader` 构建器，并创建一个 `DataFrame` 作为结果，正如我们在返回的值中看到的那样：

```
> rawData: org.apache.spark.sql.DataFrame
```

一个 `DataFrame` 是 `Dataset[Row]` 的别名，并且有已知的模式。创建后，您可以像常规的 `Dataset` 一样使用流 `Dataset`。这使得在结构化流中可以使用完整的 `Dataset` API，尽管某些例外情况存在，因为并非所有操作（例如 `show()` 或 `count()`）在流上下文中都有意义。

要以编程方式区分流 `Dataset` 和静态 `Dataset`，我们可以询问 `Dataset` 是否属于流类型：

```
rawData.isStreaming
res7: Boolean = true
```

我们还可以使用现有的 `Dataset` API 探索附加到其上的模式，正如在 示例 9-1 中所示。

##### 示例 9-1\. Kafka 架构

```
rawData.printSchema()

root
 |-- key: binary (nullable = true)
 |-- value: binary (nullable = true)
 |-- topic: string (nullable = true)
 |-- partition: integer (nullable = true)
 |-- offset: long (nullable = true)
 |-- timestamp: timestamp (nullable = true)
 |-- timestampType: integer (nullable = true)
```

一般而言，结构化流需要对消费流的模式进行明确声明。在 `kafka` 的特定情况下，结果 `Dataset` 的模式是固定的，与流的内容无关。它由一组字段组成，特定于 Kakfa `source`：`key`、`value`、`topic`、`partition`、`offset`、`timestamp` 和 `timestampType`，正如我们在 示例 9-1 中看到的那样。在大多数情况下，应用程序将主要关注流的实际负载所在的 `value` 字段的内容。

# 应用逻辑

回想一下，我们的工作意图是将传入的物联网传感器数据与包含所有已知传感器及其配置的参考文件相关联。这样，我们将丰富每个传入记录的特定传感器参数，以便解释报告的数据。然后，我们将所有处理正确的记录保存到一个 Parquet 文件中。来自未知传感器的数据将保存到一个单独的文件中，以供后续分析。

使用结构化流，我们的工作可以通过 `Dataset` 操作来实现：

```
val iotData = rawData.select($"value").as[String].flatMap{record =>
  val fields = record.split(",")
  Try {
    SensorData(fields(0).toInt, fields(1).toLong, fields(2).toDouble)
  }.toOption
}

val sensorRef = sparkSession.read.parquet(s"$workDir/$referenceFile")
sensorRef.cache()

val sensorWithInfo = sensorRef.join(iotData, Seq("sensorId"), "inner")

val knownSensors = sensorWithInfo
  .withColumn("dnvalue", $"value"*($"maxRange"-$"minRange")+$"minRange")
  .drop("value", "maxRange", "minRange")
```

在第一步中，我们将 CSV 格式的记录转换回 `SensorData` 条目。我们在从 `value` 字段提取的 `String` 类型的 `Dataset[String]` 上应用 Scala 函数操作。

然后，我们使用流 `Dataset` 对静态 `Dataset` 进行 `inner join`，以将传感器数据与相应的参考数据关联，使用 `sensorId` 作为键。

为了完成我们的应用程序，我们使用参考数据中的最小-最大范围来计算传感器读数的实际值。

# 写入流式接收器

我们流应用的最后一步是将丰富的物联网数据写入 Parquet 格式的文件。在结构化流中，`write` 操作至关重要：它标志着对流上声明的转换的完成，定义了一个 *写模式*，并在调用 start() 后，连续查询的处理将开始。

在结构化流处理中，所有操作都是对我们希望对流数据进行的操作的延迟声明。只有当我们调用`start()`时，流的实际消费才会开始，并且对数据的查询操作会实现为实际结果：

```
val knownSensorsQuery = knownSensors.writeStream
  .outputMode("append")
  .format("parquet")
  .option("path", targetPath)
  .option("checkpointLocation", "/tmp/checkpoint")
  .start()
```

让我们详细分解这个操作：

+   `writeStream` 创建一个构建器对象，在这里我们可以使用*流畅*接口配置所需写入操作的选项。

+   通过`format`，我们指定了将结果材料化到下游的接收器。在我们的情况下，我们使用内置的`FileStreamSink`和 Parquet 格式。

+   `mode` 是结构化流处理中的一个新概念：理论上，我们可以访问到迄今为止在流中看到的所有数据，因此我们也有选项来生成该数据的不同视图。

+   这里使用的`append`模式意味着我们的流处理计算产生的新记录会被输出。

调用`start`的结果是一个`StreamingQuery`实例。该对象提供了控制查询执行和请求有关我们正在运行的流查询状态信息的方法，正如示例 9-2 所示。

##### 示例 9-2\. 查询进度

```
knownSensorsQuery.recentProgress

res37: Array[org.apache.spark.sql.streaming.StreamingQueryProgress] =
Array({
  "id" : "6b9fe3eb-7749-4294-b3e7-2561f1e840b6",
  "runId" : "0d8d5605-bf78-4169-8cfe-98311fc8365c",
  "name" : null,
  "timestamp" : "2017-08-10T16:20:00.065Z",
  "numInputRows" : 4348,
  "inputRowsPerSecond" : 395272.7272727273,
  "processedRowsPerSecond" : 28986.666666666668,
  "durationMs" : {
    "addBatch" : 127,
    "getBatch" : 3,
    "getOffset" : 1,
    "queryPlanning" : 7,
    "triggerExecution" : 150,
    "walCommit" : 11
  },
  "stateOperators" : [ ],
  "sources" : [ {
    "description" : "KafkaSource[Subscribe[iot-data]]",
    "startOffset" : {
      "iot-data" : {
        "0" : 19048348
      }
    },
    "endOffset" : {
      "iot-data" : {
        "0" : 19052696
      }
    },
    "numInputRow...
```

在 示例 9-2 中，通过调用`knownSensorsQuery.recentProgress`，我们可以看到`StreamingQueryProgress`的结果。如果`numInputRows`有非零值，我们可以确定我们的作业正在消费数据。我们现在有一个正常运行的结构化流处理作业。

# 总结

希望通过这个实践章节，能够向你展示如何使用结构化流处理创建你的第一个非平凡应用程序。

阅读完本章后，你应该更好地理解了结构化流处理程序的结构以及如何处理流应用程序，从消费数据、使用`Dataset`和`DataFrames` API 进行处理，到将数据输出到外部。此时，你几乎准备好迎接创建自己的流处理作业的挑战了。

在接下来的章节中，你将深入学习结构化流处理的不同方面。
