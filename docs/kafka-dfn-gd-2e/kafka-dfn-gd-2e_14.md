# 第十二章：管理 Kafka

管理 Kafka 集群需要额外的工具来执行对主题、配置等的管理更改。Kafka 提供了几个命令行接口（CLI）实用程序，用于对集群进行管理更改。这些工具是以 Java 类实现的，并且提供了一组本地脚本来正确调用这些类。虽然这些工具提供了基本功能，但您可能会发现它们在更复杂的操作或在更大规模上使用时存在不足。本章将仅描述作为 Apache Kafka 开源项目一部分提供的基本工具。有关社区中开发的高级工具的更多信息，可以在[Apache Kafka 网站](https://kafka.apache.org)上找到。

# 授权管理操作

虽然 Apache Kafka 实现了身份验证和授权来控制主题操作，但默认配置不限制这些工具的使用。这意味着这些 CLI 工具可以在不需要任何身份验证的情况下使用，这将允许执行诸如主题更改之类的操作，而无需进行安全检查或审计。始终确保对部署中的此工具的访问受到限制，仅限管理员才能防止未经授权的更改。

# 主题操作

`kafka-topics.sh`工具提供了对大多数主题操作的简单访问。它允许您在集群中创建、修改、删除和列出有关主题的信息。虽然通过此命令可能可以进行一些主题配置，但这些配置已被弃用，建议使用更健壮的方法使用`kafka-config.sh`工具进行配置更改。要使用`kafka-topics.sh`命令，必须通过`--bootstrap-server`选项提供集群连接字符串和端口。在接下来的示例中，集群连接字符串在 Kafka 集群中的一个主机上本地运行，并且我们将使用`localhost:9092`。

在本章中，所有工具都将位于目录*/usr/local/kafka/bin/*中。本节中的示例命令将假定您在此目录中，或者已将该目录添加到您的`$PATH`中。

# 检查版本

Kafka 的许多命令行工具对 Kafka 运行的版本有依赖，以正确运行。这包括一些命令可能会将数据存储在 ZooKeeper 中，而不是直接连接到经纪人本身。因此，重要的是确保您使用的工具版本与集群中经纪人的版本匹配。最安全的方法是在 Kafka 经纪人上运行工具，使用部署的版本。

## 创建新主题

通过`--create`命令创建新主题时，在集群中创建新主题需要几个必需的参数。使用此命令时必须提供这些参数，即使其中一些可能已经配置了经纪人级别的默认值。此时还可以使用`--config`选项进行其他参数和配置覆盖，但这将在本章后面进行介绍。以下是三个必需参数的列表：

`--topic`

您希望创建的主题名称。

`--replication-factor`

主题在集群中维护的副本数量。

`--partitions`

为主题创建的分区数。

# 良好的主题命名实践

主题名称可以包含字母数字字符、下划线、破折号和句点；但不建议在主题名称中使用句点。Kafka 内部度量标准将句点字符转换为下划线字符（例如，“topic.1”在度量计算中变为“topic_1”），这可能导致主题名称冲突。

另一个建议是避免使用双下划线来开始你的主题名称。按照惯例，Kafka 操作内部的主题使用双下划线命名约定创建（比如`__consumer_offsets`主题，用于跟踪消费者组偏移存储）。因此，不建议使用以双下划线命名约定开头的主题名称，以防混淆。

创建一个新主题很简单。运行`kafka-topics.sh`如下：

```java
# kafka-topics.sh --bootstrap-server <connection-string>:<port> --create --topic <string>
--replication-factor <integer> --partitions <integer>
#
```

该命令将导致集群创建一个具有指定名称和分区数的主题。对于每个分区，集群将适当地选择指定数量的副本。这意味着如果集群设置为机架感知副本分配，每个分区的副本将位于不同的机架上。如果不希望使用机架感知分配，指定`--disable-rack-aware`命令行参数。

例如，创建一个名为“my-topic”的主题，其中每个有两个副本的八个分区：

```java
# kafka-topics.sh --bootstrap-server localhost:9092 --create
--topic my-topic --replication-factor 2 --partitions 8
Created topic "my-topic".
#
```

# 正确使用 if-exists 和 if-not-exists 参数

在自动化中使用`kafka-topics.sh`时，创建新主题时可能希望使用`--if-not-exists`参数，如果主题已经存在，则不返回错误。

虽然`--alter`命令提供了一个`--if-exists`参数，但不建议使用它。使用这个参数会导致命令在被更改的主题不存在时不返回错误。这可能掩盖了应该创建但不存在的主题的问题。

## 列出集群中的所有主题

`--list`命令列出集群中的所有主题。列表格式化为每行一个主题，没有特定顺序，这对于生成完整的主题列表很有用。

以下是使用`--list`选项列出集群中所有主题的示例：

```java
# kafka-topics.sh --bootstrap-server localhost:9092 --list
__consumer_offsets
my-topic
other-topic
```

您会注意到内部的`__consumer_offsets`主题在这里列出。使用`--exclude-internal`运行命令将从列表中删除所有以前提到的双下划线开头的主题，这可能是有益的。

## 描述主题详细信息

还可以获取集群中一个或多个主题的详细信息。输出包括分区计数、主题配置覆盖，以及每个分区及其副本分配的列表。通过向命令提供`--topic`参数，可以将其限制为单个主题。

例如，在集群中描述我们最近创建的“my-topic”：

```java
# kafka-topics.sh --boostrap-server localhost:9092 --describe --topic my-topic
Topic: my-topic	PartitionCount: 8	ReplicationFactor: 2	Configs: segment.bytes=1073741824
 Topic: my-topic	Partition: 0	Leader: 1	Replicas: 1,0	Isr: 1,0
 Topic: my-topic	Partition: 1	Leader: 0	Replicas: 0,1	Isr: 0,1
 Topic: my-topic	Partition: 2	Leader: 1	Replicas: 1,0	Isr: 1,0
 Topic: my-topic	Partition: 3	Leader: 0	Replicas: 0,1	Isr: 0,1
 Topic: my-topic	Partition: 4	Leader: 1	Replicas: 1,0	Isr: 1,0
 Topic: my-topic	Partition: 5	Leader: 0	Replicas: 0,1	Isr: 0,1
 Topic: my-topic	Partition: 6	Leader: 1	Replicas: 1,0	Isr: 1,0
 Topic: my-topic	Partition: 7	Leader: 0	Replicas: 0,1	Isr: 0,1
#
```

`--describe`命令还有几个有用的选项用于过滤输出。这些对于更容易诊断集群问题很有帮助。对于这些命令，我们通常不指定`--topic`参数，因为意图是找到所有符合条件的集群中的主题或分区。这些选项不适用于`list`命令。以下是一些有用的配对列表：

`--topics-with-overrides`

这将仅描述与集群默认配置不同的主题。

`--exclude-internal`

前面提到的命令将从列表中删除所有以双下划线命名约定开头的主题。

以下命令用于帮助查找可能存在问题的主题分区：

`--under-replicated-partitions`

这显示了所有副本中有一个或多个与领导者不同步的所有分区。这不一定是坏事，因为集群维护、部署和重新平衡会导致副本不足的分区（或 URP），但需要注意。

`--at-min-isr-partitions`

这显示了所有副本数（包括领导者）与最小同步副本（ISRs）设置完全匹配的所有分区。这些主题仍然可供生产者或消费者客户端使用，但所有冗余已经丢失，它们有可能变得不可用。

`--under-min-isr-partitions`

这显示了所有 ISR 数量低于成功生产操作所需的最小配置的所有分区。这些分区实际上处于只读模式，无法进行生产操作。

`--unavailable-partitions`

这显示了所有没有领导者的主题分区。这是一个严重的情况，表明该分区已脱机，对生产者或消费者客户端不可用。

以下是一个查找处于最小 ISR 设置的主题的示例。在此示例中，主题配置为最小 ISR 为 1，并且副本因子（RF）为 2。主机 0 在线，主机 1 已停机进行维护：

```java
# kafka-topics.sh --bootstrap-server localhost:9092 --describe --at-min-isr-partitions
 Topic: my-topic Partition: 0    Leader: 0       Replicas: 0,1   Isr: 0
 Topic: my-topic Partition: 1    Leader: 0       Replicas: 0,1   Isr: 0
 Topic: my-topic Partition: 2    Leader: 0       Replicas: 0,1   Isr: 0
 Topic: my-topic Partition: 3    Leader: 0       Replicas: 0,1   Isr: 0
 Topic: my-topic Partition: 4    Leader: 0       Replicas: 0,1   Isr: 0
 Topic: my-topic Partition: 5    Leader: 0       Replicas: 0,1   Isr: 0
 Topic: my-topic Partition: 6    Leader: 0       Replicas: 0,1   Isr: 0
 Topic: my-topic Partition: 7    Leader: 0       Replicas: 0,1   Isr: 0
#
```

## 添加分区

有时需要增加主题的分区数。分区是主题在集群中扩展和复制的方式。增加分区计数的最常见原因是通过减少单个分区的吞吐量来横向扩展主题跨多个经纪人。如果消费者需要扩展以在单个消费者组中运行更多副本，则还可以增加主题。因为一个分区只能被消费者组中的一个成员消费。

以下是一个示例，使用`--alter`命令将名为“my-topic”的主题的分区数增加到 16，然后验证它是否起作用：

```java
# kafka-topics.sh --bootstrap-server localhost:9092
--alter --topic my-topic --partitions 16

# kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic my-topic
Topic: my-topic	PartitionCount: 16	ReplicationFactor: 2	Configs: segment.bytes=1073741824
 Topic: my-topic	Partition: 0	Leader: 1	Replicas: 1,0	Isr: 1,0
 Topic: my-topic	Partition: 1	Leader: 0	Replicas: 0,1	Isr: 0,1
 Topic: my-topic	Partition: 2	Leader: 1	Replicas: 1,0	Isr: 1,0
 Topic: my-topic	Partition: 3	Leader: 0	Replicas: 0,1	Isr: 0,1
 Topic: my-topic	Partition: 4	Leader: 1	Replicas: 1,0	Isr: 1,0
 Topic: my-topic	Partition: 5	Leader: 0	Replicas: 0,1	Isr: 0,1
 Topic: my-topic	Partition: 6	Leader: 1	Replicas: 1,0	Isr: 1,0
 Topic: my-topic	Partition: 7	Leader: 0	Replicas: 0,1	Isr: 0,1
 Topic: my-topic	Partition: 8	Leader: 1	Replicas: 1,0	Isr: 1,0
 Topic: my-topic	Partition: 9	Leader: 0	Replicas: 0,1	Isr: 0,1
 Topic: my-topic	Partition: 10	Leader: 1	Replicas: 1,0	Isr: 1,0
 Topic: my-topic	Partition: 11	Leader: 0	Replicas: 0,1	Isr: 0,1
 Topic: my-topic	Partition: 12	Leader: 1	Replicas: 1,0	Isr: 1,0
 Topic: my-topic	Partition: 13	Leader: 0	Replicas: 0,1	Isr: 0,1
 Topic: my-topic	Partition: 14	Leader: 1	Replicas: 1,0	Isr: 1,0
 Topic: my-topic	Partition: 15	Leader: 0	Replicas: 0,1	Isr: 0,1
#
```

# 调整带键主题

使用带有键消息的主题可能非常难以从消费者的角度添加分区。这是因为当分区数更改时，键到分区的映射将发生变化。因此，建议在创建主题时为包含键消息的主题设置分区数一次，并避免调整主题的大小。

## 减少分区

不可能减少主题的分区数。从主题中删除一个分区也会导致该主题中的部分数据被删除，这在客户端的角度来看是不一致的。此外，尝试将数据重新分配到剩余的分区将会很困难，并导致消息的顺序混乱。如果需要减少分区数，建议删除主题并重新创建它，或者（如果无法删除）创建现有主题的新版本，并将所有生产流量转移到新主题（例如“my-topic-v2”）。

## 删除主题

即使没有消息的主题也会使用磁盘空间、打开文件句柄和内存等集群资源。控制器还必须保留其必须了解的垃圾元数据，这可能会在大规模时影响性能。如果不再需要主题，则可以删除以释放这些资源。要执行此操作，集群中的经纪人必须配置`delete.topic.enable`选项设置为`true`。如果设置为`false`，则将忽略删除主题的请求，并且不会成功。

主题删除是一个异步操作。这意味着运行此命令将标记一个主题以进行删除，但删除可能不会立即发生，这取决于所需的数据量和清理。控制器将尽快通知经纪人有关即将删除的信息（在现有控制器任务完成后），然后经纪人将使主题的元数据无效并从磁盘中删除文件。强烈建议操作员不要一次删除一个或两个以上的主题，并且在删除其他主题之前给予充分的时间来完成，因为控制器执行这些操作的方式存在限制。在本书示例中显示的小集群中，主题删除几乎会立即发生，但在较大的集群中可能需要更长的时间。

# 数据丢失

删除主题也将删除其所有消息。这是一个不可逆的操作。请确保谨慎执行。

以下是使用`--delete`参数删除名为“my-topic”的主题的示例。根据 Kafka 的版本，将会有一条说明，让您知道如果没有设置其他配置，则该参数将不起作用：

```java
# kafka-topics.sh --bootstrap-server localhost:9092
--delete --topic my-topic

Note: This will have no impact if delete.topic.enable is not set
to true.
#
```

您会注意到没有明显的反馈表明主题删除是否成功完成。通过运行`--list`或`--describe`选项来验证删除是否成功，以查看主题是否不再存在于集群中。

# 消费者组

消费者组是协调的 Kafka 消费者组，从主题或单个主题的多个分区中消费。`kafka-consumer-groups.sh`工具有助于管理和了解从集群中的主题中消费的消费者组。它可用于列出消费者组，描述特定组，删除消费者组或特定组信息，或重置消费者组偏移信息。

# 基于 ZooKeeper 的消费者组

在较旧版本的 Kafka 中，可以在 ZooKeeper 中管理和维护消费者组。此行为在 0.11.0.*版本及更高版本中已弃用，不再使用旧的消费者组。提供的某些脚本的某些版本可能仍然显示已弃用的`--zookeeper`连接字符串命令，但不建议使用它们，除非您的旧环境中有一些消费者组尚未升级到 Kafka 的较新版本。

## 列出和描述组

要列出消费者组，请使用`--bootstrap-server`和`--list`参数。使用`kafka-consumer-groups.sh`脚本的特定消费者将显示为消费者列表中的`console-consumer-*<generated_id>*`：

```java
# kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
console-consumer-95554
console-consumer-9581
my-consumer
#
```

对于列出的任何组，可以通过将`--list`参数更改为`--describe`并添加`--group`参数来获取更多详细信息。这将列出该组正在从中消费的所有主题和分区，以及其他信息，例如每个主题分区的偏移量。表 12-1 对输出中提供的所有字段进行了全面描述。

例如，获取名为“my-consumer”的特定组的消费者组详细信息：

```java
# kafka-consumer-groups.sh --bootstrap-server localhost:9092
--describe --group my-consumer
GROUP          TOPIC          PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG        CONSUMER-ID                                       HOST                           CLIENT-ID
my-consumer     my-topic       0          2               4               2          consumer-1-029af89c-873c-4751-a720-cefd41a669d6   /127.0.0.1                     consumer-1
my-consumer     my-topic       1          2               3               1          consumer-1-029af89c-873c-4751-a720-cefd41a669d6   /127.0.0.1                     consumer-1
my-consumer     my-topic       2          2               3               1          consumer-2-42c1abd4-e3b2-425d-a8bb-e1ea49b29bb2   /127.0.0.1                     consumer-2
#
```

表 12-1。为名为“my-consumer”的组提供的字段

| 字段 | 描述 |
| --- | --- |
| GROUP | 消费者组的名称。

| TOPIC | 正在消费的主题的名称。

| PARTITION | 正在消费的分区的 ID 号。

| CURRENT-OFFSET | 消费者组为此主题分区要消费的下一个偏移量。这是消费者在分区内的位置。

| LOG-END-OFFSET | 主题分区的经纪人的当前高水位偏移。这是下一条消息要被生产到这个分区的偏移量。

| LAG | 消费者当前偏移和经纪人日志结束偏移之间的差异，用于此主题分区。

| CONSUMER-ID | 基于提供的客户端 ID 生成的唯一消费者 ID。

| HOST | 消费者组正在读取的主机的地址。

| CLIENT-ID | 客户端提供的标识客户端的字符串。

## 删除组

可以使用`--delete`参数执行消费者组的删除。这将删除整个组，包括该组正在消费的所有主题的所有存储偏移量。要执行此操作，组中的所有消费者都应该关闭，因为消费者组不应该有任何活跃成员。如果尝试删除一个不为空的组，将抛出一个错误，指出“该组不为空”，并且不会发生任何事情。还可以使用相同的命令通过添加`--topic`参数并指定要删除的主题偏移量来删除组正在消费的单个主题的偏移量，而不删除整个组。

以下是删除名为“my-consumer”的整个消费者组的示例：

```java
# kafka-consumer-groups.sh --bootstrap-server localhost:9092 --delete --group my-consumer
Deletion of requested consumer groups ('my-consumer') was successful.
#
```

## 偏移管理

除了显示和删除消费者组的偏移量之外，还可以批量检索偏移量并存储新的偏移量。当消费者出现需要重新读取消息的问题时，或者需要推进偏移量并跳过消费者无法处理的消息时（例如，如果有一条格式错误的消息），这对于重置消费者的偏移量非常有用。

### 导出偏移量

要将消费者组的偏移量导出到 CSV 文件中，请使用`--reset-offsets`参数和`--dry-run`选项。这将允许我们创建当前偏移量的导出文件格式，以便以后可以重用导入或回滚偏移量。CSV 格式的导出将采用以下配置：

*<topic-name>,<partition-number>,<offset>*

在不使用`--dry-run`选项运行相同命令将完全重置偏移量，因此要小心。

以下是导出主题“my-topic”被消费者组“my-consumer”消费的偏移量的示例，导出到名为*offsets.csv*的文件中：

```java
# kafka-consumer-groups.sh --bootstrap-server localhost:9092
--export --group my-consumer --topic my-topic
--reset-offsets --to-current --dry-run > offsets.csv

# cat offsets.csv
my-topic,0,8905
my-topic,1,8915
my-topic,2,9845
my-topic,3,8072
my-topic,4,8008
my-topic,5,8319
my-topic,6,8102
my-topic,7,12739
#
```

### 导入偏移量

导入偏移量工具是导出的相反。它接受导出上一节中偏移量生成的文件，并使用它来设置消费者组的当前偏移量。一种常见做法是导出消费者组的当前偏移量，复制文件（以便保留备份），并编辑副本以替换偏移量为所需值。

# 首先停止消费者

在执行此步骤之前，重要的是停止组中的所有消费者。如果在消费者组处于活动状态时编写，它们将不会读取新的偏移量。消费者将只覆盖导入的偏移量。

在以下示例中，我们从上一个示例中创建的名为*offsets.csv*的文件中导入名为“my-consumer”的消费者组的偏移量：

```java
# kafka-consumer-groups.sh --bootstrap-server localhost:9092
--reset-offsets --group my-consumer
--from-file offsets.csv --execute
 TOPIC                          PARTITION  NEW-OFFSET
 my-topic                        0          8905
 my-topic                        1          8915
 my-topic                        2          9845
 my-topic                        3          8072
 my-topic                        4          8008
 my-topic                        5          8319
 my-topic                        6          8102
 my-topic                        7          12739
#
```

# 动态配置更改

有大量的配置适用于主题、客户端、代理等，可以在运行时动态更新，而无需关闭或重新部署集群。`kafka-configs.sh`是修改这些配置的主要工具。目前有四种主要的动态配置更改的实体类型，即*entity-types*：*topics*、*brokers*、*users*和*clients*。对于每种实体类型，都有可以覆盖的特定配置。随着每个 Kafka 版本的发布，不断添加新的动态配置，因此最好确保您使用与运行的 Kafka 版本相匹配的工具版本。为了通过自动化方便地设置这些配置，可以使用`--add-config-file`参数，并使用预先格式化的文件来管理和更新所有要管理和更新的配置。

## 覆盖主题配置默认值

有许多配置是为主题默认设置的，这些配置在静态代理配置文件中定义（例如，保留时间策略）。通过动态配置，我们可以覆盖集群级别的默认值，以适应单个集群中不同用例的不同主题。表 12-2 显示了可以动态更改的主题的有效配置键。

更改主题配置的命令格式如下：

```java
kafka-configs.sh --bootstrap-server localhost:9092
--alter --entity-type topics --entity-name <topic-name>
--add-config <key>=<value>[,<key>=<value>...]
```

以下是将名为“my-topic”的主题保留设置为 1 小时（3,600,000 毫秒）的示例：

```java
# kafka-configs.sh --bootstrap-server localhost:9092
--alter --entity-type topics --entity-name my-topic
--add-config retention.ms=3600000
Updated config for topic: "my-topic".
#
```

表 12-2\. 主题的有效键

| 配置键 | 描述 |
| --- | --- |
| `cleanup.policy` | 如果设置为`compact`，则此主题中的消息将被丢弃，只保留具有给定键的最新消息（日志压缩）。 |
| `compression.type` | 代理在将此主题的消息批次写入磁盘时使用的压缩类型。 |
| `delete.retention.ms` | 以毫秒为单位，删除的墓碑将保留在此主题中的时间。仅适用于日志压缩主题。 |
| `file.delete.delay.ms` | 从磁盘中删除此主题的日志段和索引之前等待的时间，以毫秒为单位。 |
| `flush.messages` | 在强制将此主题的消息刷新到磁盘之前接收多少消息。 |
| `flush.ms` | 强制将此主题的消息刷新到磁盘之前的时间，以毫秒为单位。 |
| `follower.replication.​throt⁠tled.replicas` | 应该由追随者限制日志复制的副本列表。 |
| `index.interval.bytes` | 日志段索引中可以在消息之间产生多少字节。 |
| `leader.replication.​throt⁠tled.replica` | 领导者应该限制日志复制的副本列表。 |
| `max.compaction.lag.ms` | 消息在日志中不符合压缩的最长时间限制。 |
| `max.message.bytes` | 此主题单个消息的最大大小，以字节为单位。 |
| `message.downconversion.enable` | 如果启用，允许将消息格式版本降级为上一个版本，但会带来一些开销。 |
| `message.format.version` | 经纪人在将消息写入磁盘时将使用的消息格式版本。必须是有效的 API 版本号。 |
| `message.timestamp.​dif⁠fer⁠ence.max.ms` | 消息时间戳和经纪人时间戳之间的最大允许差异，以毫秒为单位。仅当`message.timestamp.type`设置为`CreateTime`时有效。 |
| `message.timestamp.type` | 写入磁盘时要使用的时间戳。当前值为`CreateTime`表示客户端指定的时间戳，`LogAppendTime`表示经纪人将消息写入分区的时间。 |
| `min.clean⁠able.​dirty.ratio` | 日志压缩器尝试压缩此主题分区的频率，表示为未压缩日志段数与总日志段数的比率。仅适用于日志压缩主题。 |
| `min.compaction.lag.ms` | 消息在日志中保持未压缩的最短时间。 |
| `min.insync.replicas` | 必须同步的最小副本数，才能认为主题的分区可用。 |
| `preallocate` | 如果设置为`true`，则在滚动新段时应预先分配此主题的日志段。 |
| `retention.bytes` | 保留此主题的消息字节数。 |
| `retention.ms` | 此主题的消息应保留的时间，以毫秒为单位。 |
| `segment.bytes` | 应该写入分区中单个日志段的消息字节数。 |
| `segment.index.bytes` | 单个日志段索引的最大大小，以字节为单位。 |
| `segment.jitter.ms` | 在滚动日志段时，随机添加到`segment.ms`的最大毫秒数。 |
| `segment.ms` | 每个分区的日志段应该旋转的频率，以毫秒为单位。 |
| `unclean.leader.​elec⁠tion.enable` | 如果设置为`false`，则不允许为此主题进行不干净的领导者选举。 |

## 覆盖客户端和用户配置默认值

对于 Kafka 客户端和用户，只有少数配置可以被覆盖，它们基本上都是配额的类型。更常见的两个要更改的配置是允许每个经纪人的特定客户端 ID 的生产者和消费者的字节/秒速率。可以为*用户*和*客户端*修改的共享配置的完整列表显示在表 12-3 中。

# 在负载不平衡的集群中不均匀的限流行为

因为限流是基于每个代理的基础，所以跨集群的分区领导权的平衡变得尤为重要，以便正确执行这一点。如果您在集群中有 5 个代理，并且为客户端指定了 10 MBps 的生产者配额，那么该客户端将被允许同时在每个代理上生产 10 MBps，总共 50 MBps，假设所有 5 个主机上的领导权是平衡的。但是，如果每个分区的领导权都在代理 1 上，那么同一个生产者只能生产最大 10 MBps。

表 12-3。客户端的配置（键）

| 配置键 | 描述 |
| --- | --- |
| `consumer_bytes_rate` | 允许单个客户端 ID 在一秒内从单个代理消费的字节数。 |
| `producer_bytes_rate` | 允许单个客户端 ID 在一秒内向单个代理生产的字节数。 |
| `controller_mutations_rate` | 接受创建主题请求、创建分区请求和删除主题请求的变异速率。速率是由创建或删除的分区数量累积而成。 |
| `request_percentage` | 用户或客户端请求在配额窗口内的百分比（总数为（num.io.threads + num.network.threads）× 100%）。 |

# 客户端 ID 与消费者组

客户端 ID 不一定与消费者组名称相同。消费者可以设置自己的客户端 ID，您可能有许多消费者在不同的组中指定相同的客户端 ID。为每个消费者组设置唯一标识该组的客户端 ID 被认为是最佳实践。这允许单个消费者组共享配额，并且更容易在日志中识别负责请求的组。

可以一次指定兼容的用户和客户端配置更改，以适用于两者的兼容配置。以下是一种在一个配置步骤中更改用户和客户端的控制器变异速率的命令示例：

```java
# kafka-configs.sh --bootstrap-server localhost:9092
--alter --add-config "controller_mutations_rate=10"
--entity-type clients --entity-name <client ID>
--entity-type users --entity-name <user ID>
#
```

## 覆盖代理配置默认值

主题和集群级别的配置主要在集群配置文件中静态配置，但是有大量的配置可以在运行时进行覆盖，而无需重新部署 Kafka。超过 80 个覆盖可以使用*kafka-configs.sh*进行更改。因此，我们不会在本书中列出所有这些配置，但可以通过`--help`命令进行引用，或者在[开源文档](https://oreil.ly/R8hhb)中找到。特别值得指出的一些重要配置是：

`min.insync.replicas`

调整需要确认写入的最小副本数，以使生产请求在生产者将 acks 设置为`all`（或`–1`）时成功。

`unclean.leader.election.enable`

即使导致数据丢失，也允许副本被选举为领导者。当允许有一些有损数据或者在短时间内无法避免不可恢复的数据丢失时，这是有用的。

`max.connections`

任何时候允许连接到代理的最大连接数。我们还可以使用`max.connections.per.ip`和`max.connections.per.ip.overrides`进行更精细的限制。

## 描述配置覆盖

所有配置覆盖都可以使用`kafka-config.sh`工具列出。这将允许您检查主题、代理或客户端的具体配置。与其他工具类似，这是使用`--describe`命令完成的。

在以下示例中，我们可以获取名为“my-topic”的主题的所有配置覆盖，我们观察到只有保留时间：

```java
# kafka-configs.sh --bootstrap-server localhost:9092
--describe --entity-type topics --entity-name my-topic
Configs for topics:my-topic are
retention.ms=3600000
#
```

# 仅主题覆盖

配置描述仅显示覆盖项-不包括集群默认配置。没有办法动态发现经纪人自己的配置。这意味着在使用此工具自动发现主题或客户端设置时，用户必须单独了解集群默认配置。

## 删除配置覆盖

动态配置可以完全删除，这将导致实体恢复到集群默认设置。要删除配置覆盖，使用`--alter`命令以及`--delete-config`参数。

例如，删除名为“my-topic”的主题的`retention.ms`的配置覆盖：

```java
# kafka-configs.sh --bootstrap-server localhost:9092
--alter --entity-type topics --entity-name my-topic
--delete-config retention.ms
Updated config for topic: "my-topic".
#
```

# 生产和消费

在使用 Kafka 时，通常需要手动生产或消费一些样本消息，以验证应用程序的运行情况。提供了两个实用程序来帮助处理这个问题，`kafka-console-consumer.sh`和`kafka-console-producer.sh`，这在第二章中简要提到过，用于验证我们的安装。这些工具是主要 Java 客户端库的包装器，允许您与 Kafka 主题进行交互，而无需编写整个应用程序来完成。

# 将输出导入另一个应用程序

虽然可以编写包装控制台消费者或生产者的应用程序（例如，消费消息并将其导入另一个应用程序进行处理），但这种类型的应用程序非常脆弱，应该避免使用。很难与控制台消费者进行交互而不丢失消息。同样，控制台生产者不允许使用所有功能，并且正确发送字节很棘手。最好直接使用 Java 客户端库或使用 Kafka 协议的其他语言的第三方客户端库。

## 控制台生产者

`kakfa-console-producer.sh`工具可用于将消息写入集群中的 Kafka 主题。默认情况下，消息每行读取一条，键和值之间用制表符分隔（如果没有制表符，则键为 null）。与控制台消费者一样，生产者使用默认序列化器读取和生成原始字节（即`DefaultEncoder`）。

控制台生产者要求提供至少两个参数，以知道要连接到哪个 Kafka 集群以及在该集群中要生产到哪个主题。第一个是我们习惯使用的`--bootstrap-server`连接字符串。在完成生产后，发送文件结束（EOF）字符以关闭客户端。在大多数常见的终端中，可以使用 Control-D 来完成这个操作。

在这里，我们可以看到一个将四条消息发送到名为“my-topic”的主题的示例：

```java
# kafka-console-producer.sh --bootstrap-server localhost:9092 --topic my-topic
>Message 1
>Test Message 2
>Test Message 3
>Message 4
>^D
#
```

### 使用生产者配置选项

可以将普通生产者配置选项传递给控制台生产者。有两种方法可以做到这一点，取决于您需要传递多少选项以及您喜欢如何传递。第一种方法是通过指定`--producer.config` `*<config-file>*`来提供生产者配置文件，其中`*<config-file>*`是包含配置选项的文件的完整路径。另一种方法是在命令行上指定选项，格式为`--producer-property` `*<key>*`=`*<value>*`，其中`*<key>*`是配置选项名称，`*<value>*`是要设置的值。这对于生产者选项（如`linger.ms`或`batch.size`）可能很有用。

# 混淆的命令行选项

`--property`命令行选项适用于控制台生产者和控制台消费者，但这不应与`--producer-property`或`--consumer-property`选项混淆。`--property`选项仅用于将配置传递给消息格式化程序，而不是客户端本身。

控制台生产者有许多命令行参数可用于与`--producer-property`选项一起使用，以调整其行为。一些更有用的选项包括：

`--batch-size`

指定如果不同步发送，则在单个批次中发送的消息数。

`--timeout`

如果生产者以异步模式运行，则在生产以避免在低产出主题上长时间等待之前等待批处理大小的最大时间。

`--compression-codec <string>`

指定在生成消息时要使用的压缩类型。有效类型可以是以下之一：`none`，`gzip`，`snappy`，`zstd`或`lz4`。默认值为`gzip`。

`--sync`

同步生成消息，等待每条消息在发送下一条消息之前得到确认。

### 行读取器选项

`kafka.tools.ConsoleProducer$LineMessageReader`类负责读取标准输入并创建生产者记录，还有一些有用的选项可以通过`--property`命令行选项传递给控制台生产者：

`ignore.error`

将`parse.key`设置为`true`时，设置为`false`以在不存在键分隔符时抛出异常。默认为`true`。

`parse.key`

将键始终设置为 null 时，设置为`false`。默认为`true`。

`key.separator`

指定在读取时在消息键和消息值之间使用的分隔符字符。默认为制表符。

# 更改行读取行为

您可以为 Kafka 提供自己的类，用于自定义读取行的方法。您创建的类必须扩展`kafka.​com⁠mon.MessageReader`，并将负责创建`ProducerRecord`。在命令行上使用`--line-reader`选项指定您的类，并确保包含您的类的 JAR 在类路径中。默认值为`kafka.tools.Console​Pro⁠ducer$LineMessageReader`。

在生成消息时，`LineMessageReader`将在第一个`key.separator`的实例上拆分输入。如果在此之后没有剩余字符，则消息的值将为空。如果行上不存在键分隔符字符，或者`parse.key`为 false，则键将为 null。

## 控制台消费者

`kafka-console-consumer.sh`工具提供了一种从 Kafka 集群中的一个或多个主题中消费消息的方法。消息以标准输出形式打印，以新行分隔。默认情况下，它输出消息中的原始字节，不包括键，没有格式（使用`DefaultFormatter`）。与生产者类似，需要一些基本选项才能开始：连接到集群的连接字符串，要从中消费的主题以及要消费的时间范围。

# 检查工具版本

使用与 Kafka 集群相同版本的消费者非常重要。较旧的控制台消费者可能通过与集群或 ZooKeeper 的不正确交互来损坏集群。

与其他命令一样，连接到集群的连接字符串将是`--bootstrap-server`选项；但是，您可以从两个选项中选择要消费的主题：

`--topic`

指定要从中消费的单个主题。

`--whitelist`

匹配要从中消费的所有主题的正则表达式（记得正确转义正则表达式，以免被 shell 错误处理）。

先前的选项中只能选择并使用一个。一旦控制台消费者启动，工具将继续尝试消费，直到给出 shell 转义命令（在这种情况下为 Ctrl-C）。以下是一个示例，消费与前缀*my*匹配的集群中的所有主题（在此示例中只有一个，“my-topic”）：

```java
# kafka-console-consumer.sh --bootstrap-server localhost:9092
--whitelist 'my.*' --from-beginning
Message 1
Test Message 2
Test Message 3
Message 4
^C
#
```

### 使用消费者配置选项

除了这些基本的命令行选项之外，还可以将普通的消费者配置选项传递给控制台消费者。与 `kafka-console-producer.sh` 工具类似，可以通过两种方式来实现，具体取决于需要传递多少选项以及您更喜欢的方式。第一种是通过指定 `--consumer.config` `*<config-file>*` 来提供一个消费者配置文件，其中 `*<config-file>*` 是包含配置选项的文件的完整路径。另一种方式是在命令行上指定选项，格式为 `--consumer-property` `*<key>*`=`*<value>*`，其中 `*<key>*` 是配置选项名称，`*<value>*` 是要设置的值。

还有一些常用的控制台消费者选项，对于了解和熟悉它们很有帮助：

`--formatter` `*<classname>*`

指定要用于解码消息的消息格式化类。默认为 `kafka.tools.DefaultMessageFormatter`。

`--from-beginning`

从最旧的偏移量开始消费指定主题中的消息。否则，消费将从最新的偏移量开始。

`--max-messages <int>`

在退出之前要消费的最大消息数。

`--partition <int>`

仅从具有给定 ID 的分区中消费。

`--offset`

要从中消费的偏移量 ID（如果提供）（`<int>`）。其他有效选项是 `earliest`，它将从开头消费，以及 `latest`，它将从最新的偏移量开始消费。

`--skip-message-on-error`

如果在处理时出现错误，则跳过消息而不是停止。用于调试。

### 消息格式化选项

除了默认值之外，还有三种可用的消息格式化器：

`kafka.tools.LoggingMessageFormatter`

使用记录器输出消息，而不是标准输出。消息以 INFO 级别打印，并包括时间戳、键和值。

`kafka.tools.ChecksumMessageFormatter`

仅打印消息校验和。

`kafka.tools.NoOpMessageFormatter`

消费消息但不输出它们。

以下是一个示例，消费与之前相同的消息，但使用 `kafka.tools.ChecksumMessageFormatter` 而不是默认值：

```java
# kafka-console-consumer.sh --bootstrap-server localhost:9092
--whitelist 'my.*' --from-beginning
--formatter kafka.tools.ChecksumMessageFormatter
checksum:0
checksum:0
checksum:0
checksum:0
#
```

`kafka.tools.DefaultMessageFormatter` 还有一些有用的选项，可以使用 `--property` 命令行选项传递，如 表 12-4 中所示。

表 12-4。消息格式化属性

| 属性 | 描述 |
| --- | --- |
| `print.timestamp` | 设置为 `true` 以显示每条消息的时间戳（如果可用）。 |
| `print.key` | 设置为 `true` 以显示消息键以及值。 |
| `print.offset` | 设置为 `true` 以显示消息偏移量以及值。 |
| `print.partition` | 设置为 `true` 以显示消息所消费的主题分区。 |
| `key.separator` | 指定在打印时在消息键和消息值之间使用的分隔符字符。 |
| `line.separator` | 指定在消息之间使用的分隔符字符。 |
| `key.deserializer` | 提供一个类名，用于在打印之前对消息键进行反序列化。 |
| `value.deserializer` | 提供一个类名，用于在打印之前对消息值进行反序列化。 |

反序列化类必须实现 `org.apache.kafka.common.​ser⁠ial⁠iza⁠tion.Deserializer`，控制台消费者将调用它们的 `toString` 方法来获取要显示的输出。通常，您会将这些反序列化器实现为一个 Java 类，然后通过在执行 `kafka_​con⁠sole_consumer.sh` 之前设置 `CLASSPATH` 环境变量来将其插入到控制台消费者的类路径中。

### 消费偏移量主题

有时候查看集群的消费者组提交了哪些偏移量是有用的。您可能想要查看特定组是否根本没有提交偏移量，或者偏移量提交的频率如何。这可以通过使用控制台消费者来消费名为`__consumer_offsets`的特殊内部主题来完成。所有消费者偏移量都被写入此主题作为消息。为了解码此主题中的消息，必须使用格式化类`kafka.coordinator.group.Group​Met⁠ada⁠taManager$OffsetsMessageFormatter`。

将我们学到的所有知识整合在一起，以下是从`__consumer_offsets`主题中消费最早消息的示例：

```java
# kafka-console-consumer.sh --bootstrap-server localhost:9092
--topic __consumer_offsets --from-beginning --max-messages 1
--formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter"
--consumer-property exclude.internal.topics=false
[my-group-name,my-topic,0]::[OffsetMetadata[1,NO_METADATA]
CommitTime 1623034799990 ExpirationTime 1623639599990]
Processed a total of 1 messages
#
```

# 分区管理

默认的 Kafka 安装还包含一些用于管理分区的脚本。其中一个工具允许重新选举领导副本；另一个是一个用于将分区分配给经纪人的低级实用程序。这些工具共同可以帮助在需要更多手动干预来平衡 Kafka 经纪人集群中的消息流量的情况下进行操作。

## 首选副本选举

如第七章所述，为了可靠性，分区可以具有多个副本。重要的是要理解，在任何给定时间点，这些副本中只有一个可以成为分区的领导者，并且所有的生产和消费操作都发生在该经纪人上。保持分区的副本在哪个经纪人上拥有领导权的平衡对于确保负载通过整个 Kafka 集群的分布是必要的。

在 Kafka 中，领导者被定义为副本列表中的第一个同步副本。但是，当经纪人停止或失去与集群其余部分的连接时，领导权将转移到另一个同步副本，并且原始副本不会自动恢复任何分区的领导权。如果未启用自动领导平衡，这可能会导致在整个集群上部署后出现极其低效的平衡。因此，建议确保启用此设置，或者使用其他开源工具（如 Cruise Control）来确保始终保持良好的平衡。

如果发现 Kafka 集群的平衡不佳，可以执行一个轻量级、通常不会产生影响的过程，称为*首选领导者选举*。这告诉集群控制器选择分区的理想领导者。客户端可以自动跟踪领导权的变化，因此它们将能够移动到领导权被转移的集群中的新经纪人。可以使用`kafka-leader-election.sh`实用程序手动触发此操作。这个工具的旧版本称为`kafka-preferred-replica-election.sh`也可用，但已被弃用，而新工具允许更多的自定义，比如指定我们是否需要“首选”或“不洁选举”类型。

作为一个例子，在集群中为所有主题启动首选领导者选举可以使用以下命令执行：

```java
# kafka-leader-election.sh --bootstrap-server localhost:9092
--election-type PREFERRED --all-topic-partitions
#
```

也可以在特定分区或主题上启动选举。这可以通过使用`--topic`选项和`--partition`选项直接传入主题名称和分区来完成。还可以传入要选举的多个分区的列表。这可以通过配置一个我们称之为*partitions.json*的 JSON 文件来完成：

```java
{
    "partitions": [
        {
            "partition": 1,
            "topic": "my-topic"
        },
        {
            "partition": 2,
            "topic": "foo"
        }
    ]
}
```

在这个例子中，我们将使用名为*partitions.json*的文件启动首选副本选举，指定了一个分区列表：

```java
# kafka-leader-election.sh --bootstrap-server localhost:9092
--election-type PREFERRED --path-to-json-file partitions.json
#
```

## 更改分区的副本

偶尔可能需要手动更改分区的副本分配。可能需要这样做的一些例子是：

+   经纪人的负载不均匀，自动领导者分配处理不正确。

+   如果经纪人被下线并且分区处于副本不足状态。

+   如果添加了新的经纪人，并且我们希望更快地平衡新的分区。

+   你想要调整主题的复制因子。

`kafka-reassign-partitions.sh`可用于执行此操作。这是一个多步过程，用于生成移动集并执行提供的移动集提议。首先，我们要使用经纪人列表和主题列表生成一组移动的提议。这将需要生成一个 JSON 文件，其中包含要提供的主题列表。下一步执行先前提议生成的移动。最后，该工具可以与生成的列表一起使用，以跟踪和验证分区重新分配的进度或完成情况。

让我们假设一个场景，你有一个由四个经纪人组成的 Kafka 集群。最近，你添加了两个新的经纪人，总数增加到了六个，你想把两个主题移动到第五和第六个经纪人上。

要生成一组分区移动，首先必须创建一个包含列出主题的 JSON 对象的文件。JSON 对象的格式如下（版本号目前始终为 1）：

```java
{
    "topics": [
        {
            "topic": "foo1"
        },
        {
            "topic": "foo2"
        }
    ],
    "version": 1
}
```

一旦我们定义了 JSON 文件，我们可以使用它来生成一组分区移动，将文件*topics.json*中列出的主题移动到 ID 为 5 和 6 的经纪人：

```java
# kafka-reassign-partitions.sh --bootstrap-server localhost:9092
--topics-to-move-json-file topics.json
--broker-list 5,6 --generate
 {"version":1,
 "partitions":[{"topic":"foo1","partition":2,"replicas":[1,2]},
 {"topic":"foo1","partition":0,"replicas":[3,4]},
 {"topic":"foo2","partition":2,"replicas":[1,2]},
 {"topic":"foo2","partition":0,"replicas":[3,4]},
 {"topic":"foo1","partition":1,"replicas":[2,3]},
 {"topic":"foo2","partition":1,"replicas":[2,3]}]
 }

 Proposed partition reassignment configuration

 {"version":1,
 "partitions":[{"topic":"foo1","partition":2,"replicas":[5,6]},
 {"topic":"foo1","partition":0,"replicas":[5,6]},
 {"topic":"foo2","partition":2,"replicas":[5,6]},
 {"topic":"foo2","partition":0,"replicas":[5,6]},
 {"topic":"foo1","partition":1,"replicas":[5,6]},
 {"topic":"foo2","partition":1,"replicas":[5,6]}]
 }
#
```

这里提出的输出格式正确，我们可以保存两个新的 JSON 文件，我们将它们称为*revert-reassignment.json*和*expand-cluster-reassignment.json*。第一个文件可用于将分区移动回原始位置，如果有必要进行回滚。第二个文件可用于下一步，因为这只是一个提议，尚未执行任何操作。你会注意到输出中领导权的平衡不够好，因为提议将导致所有领导权移动到经纪人 5。我们现在将忽略这一点，并假设集群自动领导权平衡已启用，这将有助于稍后进行分发。值得注意的是，如果你确切地知道要将分区移动到哪里，并且手动编写 JSON 来移动分区，第一步可以跳过。

要执行文件*expand-cluster-reassignment.json*中提出的分区重新分配，运行以下命令：

```java
# kafka-reassign-partitions.sh --bootstrap-server localhost:9092
--reassignment-json-file expand-cluster-reassignment.json
--execute
 Current partition replica assignment

 {"version":1,
 "partitions":[{"topic":"foo1","partition":2,"replicas":[1,2]},
 {"topic":"foo1","partition":0,"replicas":[3,4]},
 {"topic":"foo2","partition":2,"replicas":[1,2]},
 {"topic":"foo2","partition":0,"replicas":[3,4]},
 {"topic":"foo1","partition":1,"replicas":[2,3]},
 {"topic":"foo2","partition":1,"replicas":[2,3]}]
 }

 Save this to use as the --reassignment-json-file option during rollback
 Successfully started reassignment of partitions
 {"version":1,
 "partitions":[{"topic":"foo1","partition":2,"replicas":[5,6]},
 {"topic":"foo1","partition":0,"replicas":[5,6]},
 {"topic":"foo2","partition":2,"replicas":[5,6]},
 {"topic":"foo2","partition":0,"replicas":[5,6]},
 {"topic":"foo1","partition":1,"replicas":[5,6]},
 {"topic":"foo2","partition":1,"replicas":[5,6]}]
 }
#
```

这将开始将指定分区副本重新分配到新的经纪人。输出与生成的提议验证相同。集群控制器通过将新副本添加到每个分区的副本列表来执行此重新分配操作，这将暂时增加这些主题的复制因子。然后，新副本将从当前领导者复制每个分区的所有现有消息。根据磁盘上分区的大小，这可能需要大量时间，因为数据通过网络复制到新副本。复制完成后，控制器通过将旧副本从副本列表中删除来将复制因子减少到原始大小，并删除旧副本。

以下是命令的其他一些有用功能，你可以利用：

`--additional`

这个选项允许你添加到现有的重新分配中，这样它们可以继续执行而不中断，并且无需等待原始移动完成才能开始新的批处理。

`--disable-rack-aware`

有时，由于机架感知设置，提案的最终状态可能是不可能的。如果有必要，可以使用此命令覆盖。

`--throttle`

这个值以字节/秒为单位。重新分配分区对集群的性能有很大影响，因为它们会导致内存页缓存的一致性发生变化，并使用网络和磁盘 I/O。限制分区移动可以有助于防止这个问题。这可以与`--additional`标记结合使用，以限制可能导致问题的已启动重新分配过程。

# 在重新分配副本时改善网络利用率

当从单个代理中删除许多分区时，例如如果该代理正在从集群中移除，首先从代理中删除所有领导可能是有用的。这可以通过手动将领导权移出代理来完成；然而，使用前面的工具来做这件事是很困难的。其他开源工具，如 Cruise Control，包括代理“降级”等功能，可以安全地将领导权从代理转移出去，这可能是最简单的方法。

但是，如果您没有访问这样的工具，简单地重新启动代理就足够了。当代理准备关闭时，该特定代理上的所有分区的领导权将移动到集群中的其他代理。这可以显著提高重新分配的性能，并减少对集群的影响，因为复制流量将分布到许多代理中。但是，如果在代理弹出后启用了自动领导重新分配，领导权可能会返回到该代理，因此暂时禁用此功能可能是有益的。

要检查分区移动的进度，可以使用该工具来验证重新分配的状态。这将显示当前正在进行的重新分配，已完成的重新分配以及（如果有错误）失败的重新分配。为此，您必须拥有在执行步骤中使用的 JSON 对象的文件。

以下是在运行前面的分区重新分配时使用`--verify`选项时的潜在结果的示例，文件名为*expand-cluster-reassignment.json*：

```java
# kafka-reassign-partitions.sh --bootstrap-server localhost:9092
--reassignment-json-file expand-cluster-reassignment.json
--verify
Status of partition reassignment:
 Status of partition reassignment:
 Reassignment of partition [foo1,0] completed successfully
 Reassignment of partition [foo1,1] is in progress
 Reassignment of partition [foo1,2] is in progress
 Reassignment of partition [foo2,0] completed successfully
 Reassignment of partition [foo2,1] completed successfully
 Reassignment of partition [foo2,2] completed successfully
#
```

### 更改复制因子

`kafka-reassign-partitions.sh`工具也可以用于增加或减少分区的复制因子（RF）。在分区使用错误的 RF 创建时，扩展集群时需要增加冗余性，或者为了节省成本而减少冗余性的情况下，可能需要这样做。一个明显的例子是，如果调整了集群 RF 默认设置，现有主题将不会自动增加。该工具可用于增加现有分区的 RF。

例如，如果我们想要将上一个示例中的主题“foo1”从 RF = 2 增加到 RF = 3，那么我们可以制作一个类似于之前使用的执行提案的 JSON，只是我们会在副本集中添加一个额外的代理 ID。例如，我们可以构造一个名为*increase-foo1-RF.json*的 JSON，在其中我们将代理 4 添加到我们已经拥有的 5,6 的现有集合中：

```java
{
  {"version":1,
  "partitions":[{"topic":"foo1","partition":1,"replicas":[5,6,4]},
                {"topic":"foo1","partition":2,"replicas":[5,6,4]},
                {"topic":"foo1","partition":3,"replicas":[5,6,4]},
  }
}
```

然后，我们将使用之前显示的命令来执行此提案。当完成时，我们可以使用`--verify`标志或使用`kafka-topics.sh`脚本来描述主题来验证 RF 是否已经增加：

```java
# kafka-topics.sh --bootstrap-server localhost:9092 --topic foo1 --describe
 Topic:foo1	PartitionCount:3	ReplicationFactor:3	Configs:
 Topic: foo1	Partition: 0	Leader: 5	Replicas: 5,6,4	Isr: 5,6,4
 Topic: foo1	Partition: 1	Leader: 5	Replicas: 5,6,4	Isr: 5,6,4
 Topic: foo1	Partition: 2	Leader: 5	Replicas: 5,6,4	Isr: 5,6,4
#
```

### 取消副本重新分配

过去取消副本重新分配是一个危险的过程，需要通过删除`/admin/reassign_partitions` znode 来不安全地手动操作 ZooKeeper 节点（或 znode）。幸运的是，现在不再是这种情况。`kafka-reassign-partitions.sh`脚本（以及它作为包装器的 AdminClient）现在支持`--cancel`选项，该选项将取消正在进行的集群中的活动重新分配。在停止正在进行的分区移动时，`--cancel`命令旨在将副本集恢复到重新分配启动之前的状态。因此，如果从死掉的代理或负载过重的代理中删除副本，可能会使集群处于不良状态。还不能保证恢复的副本集将与以前的顺序相同。

## 转储日志段

偶尔您可能需要读取消息的特定内容，也许是因为您的主题中出现了一个损坏的“毒丸”消息，您的消费者无法处理它。提供了`kafka-dump-log.sh`工具来解码分区的日志段。这将允许您查看单个消息，而无需消费和解码它们。该工具将作为参数接受一个逗号分隔的日志段文件列表，并可以打印出消息摘要信息或详细消息数据。

在此示例中，我们将从一个名为“my-topic”的示例主题中转储日志，其中只有四条消息。首先，我们将简单地解码名为*00000000000000000000.log*的日志段文件，并检索有关每条消息的基本元数据信息，而不实际打印消息内容。在我们的示例 Kafka 安装中，Kafka 数据目录设置为*/tmp/kafka-logs*。因此，我们用于查找日志段的目录将是*/tmp/kafka-logs/<topic-name>-<partition>*，在这种情况下是*/tmp/kafka-logs/my-topic-0/*：

```java
# kafka-dump-log.sh --files /tmp/kafka-logs/my-topic-0/00000000000000000000.log
Dumping /tmp/kafka-logs/my-topic-0/00000000000000000000.log
Starting offset: 0
baseOffset: 0 lastOffset: 0 count: 1 baseSequence: -1 lastSequence: -1
 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0
 isTransactional: false isControl: false position: 0
 CreateTime: 1623034799990 size: 77 magic: 2
 compresscodec: NONE crc: 1773642166 isvalid: true
baseOffset: 1 lastOffset: 1 count: 1 baseSequence: -1 lastSequence: -1
 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0
 isTransactional: false isControl: false position: 77
 CreateTime: 1623034803631 size: 82 magic: 2
 compresscodec: NONE crc: 1638234280 isvalid: true
baseOffset: 2 lastOffset: 2 count: 1 baseSequence: -1 lastSequence: -1
 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0
 isTransactional: false isControl: false position: 159
 CreateTime: 1623034808233 size: 82 magic: 2
 compresscodec: NONE crc: 4143814684 isvalid: true
baseOffset: 3 lastOffset: 3 count: 1 baseSequence: -1 lastSequence: -1
 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0
 isTransactional: false isControl: false position: 241
 CreateTime: 1623034811837 size: 77 magic: 2
 compresscodec: NONE crc: 3096928182 isvalid: true
#
```

在下一个示例中，我们添加了`--print-data-log`选项，这将为我们提供实际的有效载荷信息和更多内容：

```java
# kafka-dump-log.sh --files /tmp/kafka-logs/my-topic-0/00000000000000000000.log --print-data-log
Dumping /tmp/kafka-logs/my-topic-0/00000000000000000000.log
Starting offset: 0
baseOffset: 0 lastOffset: 0 count: 1 baseSequence: -1 lastSequence: -1
 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0
 isTransactional: false  isControl: false position: 0
 CreateTime: 1623034799990 size: 77 magic: 2
 compresscodec: NONE crc: 1773642166 isvalid: true
| offset: 0 CreateTime: 1623034799990 keysize: -1 valuesize: 9
 sequence: -1 headerKeys: [] payload: Message 1
baseOffset: 1 lastOffset: 1 count: 1 baseSequence: -1 lastSequence: -1
 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0
 isTransactional: false isControl: false position: 77
 CreateTime: 1623034803631 size: 82 magic: 2
 compresscodec: NONE crc: 1638234280 isvalid: true
| offset: 1 CreateTime: 1623034803631 keysize: -1 valuesize: 14
 sequence: -1 headerKeys: [] payload: Test Message 2
baseOffset: 2 lastOffset: 2 count: 1 baseSequence: -1 lastSequence: -1
 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0
 isTransactional: false isControl: false position: 159
 CreateTime: 1623034808233 size: 82 magic: 2
 compresscodec: NONE crc: 4143814684 isvalid: true
| offset: 2 CreateTime: 1623034808233 keysize: -1 valuesize: 14
 sequence: -1 headerKeys: [] payload: Test Message 3
baseOffset: 3 lastOffset: 3 count: 1 baseSequence: -1 lastSequence: -1
 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0
 isTransactional: false isControl: false position: 241
 CreateTime: 1623034811837 size: 77 magic: 2
 compresscodec: NONE crc: 3096928182 isvalid: true
| offset: 3 CreateTime: 1623034811837 keysize: -1 valuesize: 9
 sequence: -1 headerKeys: [] payload: Message 4
#
```

该工具还包含一些其他有用的选项，例如验证与日志段一起使用的索引文件。索引用于在日志段中查找消息，如果损坏，将导致消费中的错误。验证是在代理以不洁净状态启动时执行的（即，它没有正常停止），但也可以手动执行。有两个选项用于检查索引，取决于您想要进行多少检查。选项“--index-sanity-check”将仅检查索引是否处于可用状态，而“--verify-index-only”将检查索引中的不匹配项，而不打印出所有索引条目。另一个有用的选项“--value-decoder-class”允许通过传递解码器对序列化消息进行反序列化。

## 副本验证

分区复制类似于常规 Kafka 消费者客户端：跟随者代理从最旧的偏移开始复制，并定期将当前偏移检查点到磁盘。当复制停止并重新启动时，它将从上次检查点继续。以前复制的日志段可能会从代理中删除，在这种情况下，跟随者不会填补这些间隙。

要验证主题分区的副本在集群中是否相同，可以使用`kafka-replica-verification.sh`工具进行验证。此工具将从给定一组主题分区的所有副本中获取消息，检查所有消息是否存在于所有副本中，并打印出给定分区的最大滞后。此过程将在循环中持续运行，直到取消。为此，您必须提供一个显式的逗号分隔的代理列表以连接到。默认情况下，将验证所有主题；但是，您还可以提供一个正则表达式，以匹配您希望验证的主题。

# 注意：集群影响

副本验证工具将对您的集群产生与重新分配分区类似的影响，因为它必须从最旧的偏移读取所有消息以验证副本。此外，它会并行从分区的所有副本中读取，因此应谨慎使用。

例如，在 kafka 代理 1 和 2 上验证以*my*开头的主题的副本，其中包含“my-topic”的分区 0：

```java
# kafka-replica-verification.sh --broker-list kafka.host1.domain.com:9092,kafka.host2.domain.com:9092
--topic-white-list 'my.*'

2021-06-07 03:28:21,829: verification process is started.
2021-06-07 03:28:51,949: max lag is 0 for partition my-topic-0 at offset 4 among 1 partitions
2021-06-07 03:29:22,039: max lag is 0 for partition my-topic-0 at offset 4 among 1 partitions
...
#
```

# 其他工具

Kafka 发行版中还包含了几个未在本书中深入介绍的工具，这些工具可以用于管理 Kafka 集群以满足特定用例。有关它们的更多信息可以在[Apache Kafka 网站](https://kafka.apache.org)上找到：

客户端 ACL

提供了一个命令行工具`kafka-acls.sh`，用于与 Kafka 客户端的访问控制进行交互。这包括了完整的授权者属性功能，设置拒绝或允许原则，集群或主题级别的限制，ZooKeeper TLS 文件配置等等。

轻量级 MirrorMaker

提供了一个轻量级的`kafka-mirror-maker.sh`脚本用于数据镜像。关于复制的更深入的内容可以在第十章中找到。

测试工具

还有一些其他用于测试 Kafka 或帮助执行功能升级的脚本。`kafka-broker-api-versions.sh`有助于在从一个 Kafka 版本升级到另一个版本时轻松识别可用 API 元素的不同版本，并检查兼容性问题。还有生产者和消费者性能测试脚本。还有一些脚本用于帮助管理 ZooKeeper。还有`trogdor.sh`，这是一个旨在运行基准测试和其他工作负载以尝试对系统进行压力测试的测试框架。

# 不安全的操作

有一些管理任务在技术上是可能的，但除非在极端情况下，不应尝试执行。通常情况下，这是在诊断问题并且已经没有其他选择时，或者发现了需要临时解决的特定错误时。这些任务通常是未记录的，不受支持的，并对应用程序造成一定风险。

这里记录了一些常见的任务，以便在紧急情况下有可能进行恢复。在正常的集群操作下，不建议使用它们，并且在执行之前应仔细考虑。

# 危险：这里有危险

本节中的操作通常涉及直接使用存储在 ZooKeeper 中的集群元数据。这可能是非常危险的操作，因此除非另有说明，否则必须非常小心，不要直接修改 ZooKeeper 中的信息。

## 移动集群控制器

每个 Kafka 集群都有一个被指定为控制器的单个代理。控制器有一个特殊的线程负责监督集群操作以及正常的代理工作。通常情况下，控制器选举是通过短暂的 ZooKeeper znode 监视自动完成的。当控制器关闭或不可用时，其他代理会尽快自我提名，因为一旦控制器关闭，znode 就会被删除。

在偶尔的情况下，当排除集群或代理的故障时，可能有必要强制将控制器移动到另一个代理，而不关闭主机。一个这样的例子是当控制器遇到异常或其他问题导致其运行但不起作用时。在这些情况下移动控制器通常不会有很高的风险，但由于这不是正常的任务，不应经常执行。

要强制移动控制器，需要手动删除*/admin/controller*下的 ZooKeeper znode，这将导致当前控制器辞职，并且集群将随机选择一个新的控制器。目前在 Apache Kafka 中没有办法指定特定的代理作为控制器。

## 删除待删除的主题

在尝试删除 Kafka 中的主题时，ZooKeeper 节点会请求创建删除。一旦每个副本完成主题的删除并确认删除完成，znode 将被删除。在正常情况下，集群会非常快地执行此操作。然而，有时这个过程可能出现问题。以下是一些删除请求可能被卡住的情况：

1.  请求者无法知道集群中是否启用了主题删除，并且可以请求从禁用删除的集群中删除主题。

1.  有一个非常大的主题要求被删除，但在处理请求之前，一个或多个副本集由于硬件故障而下线，删除无法完成，因为控制器无法确认删除是否成功完成。

要“解除”主题删除，首先删除*/admin/delete_topic/<topic>* znode。删除主题 ZooKeeper 节点（但不删除父节点*/admin/delete_topic*）将删除待处理的请求。如果删除被控制器中的缓存请求重新排队，可能还需要在删除主题 znode 后立即强制移动控制器，以确保控制器中没有挂起的缓存请求。

## 手动删除主题

如果您运行的是禁用删除主题的集群，或者发现自己需要在正常操作流程之外删除一些主题，那么可以手动从集群中删除它们。但是，这需要完全关闭集群中的所有经纪人，并且不能在集群中的任何经纪人运行时执行。

# 首先关闭经纪人

在集群在线时修改 ZooKeeper 中的集群元数据是非常危险的操作，可能会使集群处于不稳定状态。永远不要在集群在线时尝试删除或修改 ZooKeeper 中的主题元数据。

从集群中删除主题：

1.  关闭集群中的所有经纪人。

1.  从 Kafka 集群路径中删除 ZooKeeper 路径*/brokers/topics/<topic>*。请注意，此节点有必须首先删除的子节点。

1.  从每个经纪人的日志目录中删除分区目录。这些目录将被命名为`<topic>-<int>`，其中`<int>`是分区 ID。

1.  重新启动所有经纪人。

# 总结

运行 Kafka 集群可能是一项艰巨的任务，需要进行大量配置和维护任务，以确保系统以最佳性能运行。在本章中，我们讨论了许多常规任务，比如管理主题和客户端配置，这些是您经常需要处理的。我们还涵盖了一些更神秘的任务，这些任务是您需要用来调试问题的，比如检查日志段。最后，我们涵盖了一些操作，虽然不安全或常规，但可以帮助您摆脱困境。总的来说，这些工具将帮助您管理 Kafka 集群。随着您开始扩展 Kafka 集群的规模，即使使用这些工具也可能变得艰难和难以管理。强烈建议与开源 Kafka 社区合作，并利用生态系统中的许多其他开源项目，以帮助自动化本章中概述的许多任务。

现在我们对管理和管理我们的集群所需的工具有信心，但是如果没有适当的监控，这仍然是不可能的。第十三章将讨论监控经纪人和集群健康和运行状况的方法，以便您可以确保 Kafka 运行良好（并知道何时不是）。我们还将提供监控客户端的最佳实践，包括生产者和消费者。
