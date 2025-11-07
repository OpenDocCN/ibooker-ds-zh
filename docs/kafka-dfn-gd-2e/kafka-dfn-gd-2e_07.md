# 第五章：以编程方式管理 Apache Kafka

有许多用于管理 Kafka 的 CLI 和 GUI 工具（我们将在第九章中讨论它们），但有时您也希望从客户端应用程序中执行一些管理命令。根据用户输入或数据按需创建新主题是一个特别常见的用例：物联网（IoT）应用程序经常从用户设备接收事件，并根据设备类型将事件写入主题。如果制造商生产了一种新类型的设备，您要么必须通过某种流程记住也创建一个主题，要么应用程序可以在收到未识别设备类型的事件时动态创建一个新主题。第二种选择有缺点，但在适当的场景中避免依赖其他流程生成主题是一个吸引人的特性。

Apache Kafka 在 0.11 版本中添加了 AdminClient，以提供用于管理功能的编程 API，以前是在命令行中完成的：列出、创建和删除主题；描述集群；管理 ACL；以及修改配置。

这里有一个例子。您的应用程序将向特定主题生成事件。这意味着在生成第一个事件之前，主题必须存在。在 Apache Kafka 添加 AdminClient 之前，几乎没有什么选择，而且没有一个特别用户友好的选择：您可以从“producer.send()”方法捕获“UNKNOWN_TOPIC_OR_PARTITION”异常，并让用户知道他们需要创建主题，或者您可以希望您写入的 Kafka 集群启用了自动主题创建，或者您可以尝试依赖内部 API 并处理没有兼容性保证的后果。现在 Apache Kafka 提供了 AdminClient，有一个更好的解决方案：使用 AdminClient 检查主题是否存在，如果不存在，立即创建它。

在本章中，我们将概述 AdminClient，然后深入探讨如何在应用程序中使用它的细节。我们将重点介绍最常用的功能：主题、消费者组和实体配置的管理。

# AdminClient 概述

当您开始使用 Kafka AdminClient 时，了解其核心设计原则将有所帮助。当您了解了 AdminClient 的设计方式以及应该如何使用时，每种方法的具体内容将更加直观。

## 异步和最终一致的 API

也许关于 Kafka 的 AdminClient 最重要的一点是它是异步的。每个方法在将请求传递给集群控制器后立即返回，并且每个方法返回一个或多个“Future”对象。“Future”对象是异步操作的结果，并且具有用于检查异步操作状态、取消异步操作、等待其完成以及在其完成后执行函数的方法。Kafka 的 AdminClient 将“Future”对象封装到“Result”对象中，提供了等待操作完成的方法和用于常见后续操作的辅助方法。例如，“Kafka​AdminClient.createTopics”返回“CreateTopicsResult”对象，该对象允许您等待所有主题创建完成，单独检查每个主题的状态，并在创建后检索特定主题的配置。

由于 Kafka 从控制器到代理的元数据传播是异步的，AdminClient API 返回的“Futures”在控制器状态完全更新后被视为完成。在那时，不是每个代理都可能意识到新状态，因此“listTopics”请求可能会由一个不是最新的代理处理，并且不会包含最近创建的主题。这种属性也被称为*最终一致性*：最终每个代理都将了解每个主题，但我们无法保证这将在何时发生。

## 选项

AdminClient 中的每个方法都接受一个特定于该方法的`Options`对象作为参数。例如，`listTopics`方法将`ListTopicsOptions`对象作为参数，`describeCluster`将`DescribeClusterOptions`作为参数。这些对象包含了请求将由代理如何处理的不同设置。所有 AdminClient 方法都具有的一个设置是`timeoutMs`：这控制客户端在抛出`TimeoutException`之前等待来自集群的响应的时间。这限制了您的应用程序可能被 AdminClient 操作阻塞的时间。其他选项包括`listTopics`是否还应返回内部主题，以及`describeCluster`是否还应返回客户端被授权在集群上执行哪些操作。

## 扁平层次结构

Apache Kafka 协议支持的所有管理操作都直接在`KafkaAdminClient`中实现。没有对象层次结构或命名空间。这有点有争议，因为接口可能相当庞大，也许有点令人不知所措，但主要好处是，如果您想知道如何在 Kafka 上以编程方式执行任何管理操作，您只需搜索一个 JavaDoc，并且您的 IDE 自动完成将非常方便。您不必纠结于是否只是错过了查找的正确位置。如果不在 AdminClient 中，那么它还没有被实现（但欢迎贡献！）。

###### 提示

如果您有兴趣为 Apache Kafka 做出贡献，请查看我们的[“如何贡献”指南](https://oreil.ly/8zFsj)。在着手进行对架构或协议的更重大更改之前，先从较小的、不具争议的错误修复和改进开始。也鼓励非代码贡献，如错误报告、文档改进、回答问题和博客文章。

## 附加说明

修改集群状态的所有操作——创建、删除和更改——都由控制器处理。读取集群状态的操作——列出和描述——可以由任何代理处理，并且会被定向到最不负载的代理（基于客户端所知）。这不应影响您作为 API 用户，但如果您发现意外行为，注意到某些操作成功而其他操作失败，或者如果您试图弄清楚为什么某个操作花费太长时间，这可能是有好处的。

在我们撰写本章时（Apache Kafka 2.5 即将发布），大多数管理操作可以通过 AdminClient 或直接通过修改 ZooKeeper 中的集群元数据来执行。我们强烈建议您永远不要直接使用 ZooKeeper，如果您绝对必须这样做，请将其报告为 Apache Kafka 的错误。原因是在不久的将来，Apache Kafka 社区将删除对 ZooKeeper 的依赖，每个使用 ZooKeeper 直接进行管理操作的应用程序都必须进行修改。另一方面，AdminClient API 将保持完全相同，只是在 Kafka 集群内部有不同的实现。

# AdminClient 生命周期：创建、配置和关闭

要使用 Kafka 的 AdminClient，您首先必须构建 AdminClient 类的实例。这非常简单：

```java
Properties props = new Properties();
props.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
AdminClient admin = AdminClient.create(props);
// TODO: Do something useful with AdminClient
admin.close(Duration.ofSeconds(30));
```

静态的`create`方法接受一个配置了`Properties`对象的参数。唯一必需的配置是集群的 URI：一个逗号分隔的要连接的代理列表。通常在生产环境中，您希望至少指定三个代理，以防其中一个当前不可用。我们将在第十一章中讨论如何单独配置安全和经过身份验证的连接。

如果您启动了 AdminClient，最终您会想要关闭它。重要的是要记住，当您调用`close`时，可能仍然有一些 AdminClient 操作正在进行中。因此，`close`方法接受一个超时参数。一旦您调用`close`，就不能调用任何其他方法或发送任何其他请求，但客户端将等待响应直到超时到期。超时到期后，客户端将中止所有正在进行的操作，并释放所有资源。在没有超时的情况下调用`close`意味着客户端将等待所有正在进行的操作完成。

您可能还记得第三章和第四章中提到的`KafkaProducer`和`Kafka​Con⁠sumer`有许多重要的配置参数。好消息是 AdminClient 要简单得多，没有太多需要配置的地方。您可以在[Kafka 文档](https://oreil.ly/0kjKE)中阅读所有配置参数。在我们看来，重要的配置参数在以下部分中有描述。

## client.dns.lookup

这个配置是在 Apache Kafka 2.1.0 版本中引入的。

默认情况下，Kafka 根据引导服务器配置中提供的主机名（以及稍后在`advertised.listeners`配置中由代理返回的名称）验证、解析和创建连接。这种简单的模型在大多数情况下都有效，但未能涵盖两个重要的用例：使用 DNS 别名，特别是在引导配置中，以及使用映射到多个 IP 地址的单个 DNS。这些听起来相似，但略有不同。让我们更详细地看看这两种互斥的情况。

### 使用 DNS 别名

假设您有多个代理，命名规则如下：`broker1.hostname.com`，`broker2.hostname.com`等。您可能希望创建一个单个的 DNS 别名，将所有这些代理映射到一个别名上，而不是在引导服务器配置中指定所有这些代理，这可能很难维护。您将使用`all-brokers.hostname.com`进行引导，因为您实际上并不关心哪个代理从客户端获得初始连接。这一切都非常方便，除非您使用 SASL 进行身份验证。如果使用 SASL，客户端将尝试对`all-brokers.hostname.com`进行身份验证，但服务器主体将是`broker2.hostname.com`。如果名称不匹配，SASL 将拒绝进行身份验证（代理证书可能是中间人攻击），连接将失败。

在这种情况下，您将希望使用`client.dns.lookup=resolve_canonical_bootstrap_servers_only`。通过这种配置，客户端将“展开”DNS 别名，结果将与在原始引导列表中将 DNS 别名连接到的所有代理作为代理一样。

### 具有多个 IP 地址的 DNS 名称

在现代网络架构中，通常会将所有代理放在代理或负载均衡器后面。如果您使用 Kubernetes，这种情况尤其常见，因为负载均衡器是必要的，以允许来自 Kubernetes 集群外部的连接。在这些情况下，您不希望负载均衡器成为单点故障。因此，非常常见的是`broker1.hostname.com`指向一组 IP，所有这些 IP 都解析为负载均衡器，并且所有这些 IP 都将流量路由到同一个代理。这些 IP 也可能随时间而变化。默认情况下，Kafka 客户端将尝试连接主机名解析的第一个 IP。这意味着如果该 IP 不可用，客户端将无法连接，即使代理完全可用。因此，强烈建议使用`client.dns.lookup=use_all_dns_ips`，以确保客户端不会错过高可用负载均衡层的好处。

## request.timeout.ms

这个配置限制了应用程序等待 AdminClient 响应的时间。这包括在客户端收到可重试错误时重试的时间。

默认值是 120 秒，这相当长，但某些 AdminClient 操作，特别是消费者组管理命令，可能需要一段时间才能响应。正如我们在“AdminClient Overview”中提到的，每个 AdminClient 方法都接受一个`Options`对象，其中可以包含一个特定于该调用的超时值。如果 AdminClient 操作对于您的应用程序的关键路径，您可能希望使用较低的超时值，并以不同的方式处理来自 Kafka 的及时响应的缺乏。一个常见的例子是，服务在首次启动时尝试验证特定主题的存在，但如果 Kafka 花费超过 30 秒来响应，您可能希望继续启动服务器，并稍后验证主题的存在（或完全跳过此验证）。

# 基本主题管理

现在我们已经创建并配置了一个 AdminClient，是时候看看我们可以用它做什么了。Kafka 的 AdminClient 最常见的用例是主题管理。这包括列出主题、描述主题、创建主题和删除主题。

让我们首先列出集群中的所有主题：

```java
ListTopicsResult topics = admin.listTopics();
topics.names().get().forEach(System.out::println);
```

请注意，`admin.listTopics()`返回`ListTopicsResult`对象，它是对`Futures`集合的薄包装。还要注意，`topics.name()`返回`name`的`Future`集。当我们在这个`Future`上调用`get()`时，执行线程将等待服务器响应一组主题名称，或者我们收到超时异常。一旦我们得到列表，我们遍历它以打印所有主题名称。

现在让我们尝试一些更有雄心的事情：检查主题是否存在，如果不存在则创建。检查特定主题是否存在的一种方法是获取所有主题的列表，并检查您需要的主题是否在列表中。在大型集群上，这可能效率低下。此外，有时您希望检查的不仅仅是主题是否存在 - 您希望确保主题具有正确数量的分区和副本。例如，Kafka Connect 和 Confluent Schema Registry 使用 Kafka 主题存储配置。当它们启动时，它们会检查配置主题是否存在，它只有一个分区以确保配置更改按严格顺序到达，它有三个副本以确保可用性，并且主题是压缩的，因此旧配置将被无限期保留：

```java
DescribeTopicsResult demoTopic = admin.describeTopics(TOPIC_LIST); // ①

try {
    topicDescription = demoTopic.values().get(TOPIC_NAME).get(); // ②
    System.out.println("Description of demo topic:" + topicDescription);

    if (topicDescription.partitions().size() != NUM_PARTITIONS) { // ③
      System.out.println("Topic has wrong number of partitions. Exiting.");
      System.exit(-1);
    }
} catch (ExecutionException e) { // ④
    // exit early for almost all exceptions
    if (! (e.getCause() instanceof UnknownTopicOrPartitionException)) {
        e.printStackTrace();
        throw e;
    }

    // if we are here, topic doesn't exist
    System.out.println("Topic " + TOPIC_NAME +
        " does not exist. Going to create it now");
    // Note that number of partitions and replicas is optional. If they are
    // not specified, the defaults configured on the Kafka brokers will be used
    CreateTopicsResult newTopic = admin.createTopics(Collections.singletonList(
            new NewTopic(TOPIC_NAME, NUM_PARTITIONS, REP_FACTOR))); // ⑤

    // Check that the topic was created correctly:
    if (newTopic.numPartitions(TOPIC_NAME).get() != NUM_PARTITIONS) { // ⑥
        System.out.println("Topic has wrong number of partitions.");
        System.exit(-1);
    }
}
```

①

验证主题是否以正确的配置存在，我们使用要验证的主题名称列表调用`describe​Top⁠ics()`。这将返回`Descri⁠be​TopicResult`对象，其中包装了主题名称到`Future`描述的映射。

②

我们已经看到，如果我们等待`Future`完成，使用`get()`我们可以得到我们想要的结果，在这种情况下是`TopicDescription`。但也有可能服务器无法正确完成请求 - 如果主题不存在，服务器无法响应其描述。在这种情况下，服务器将返回错误，并且`Future`将通过抛出`Execution​Exception`完成。服务器发送的实际错误将是异常的`cause`。由于我们想要处理主题不存在的情况，我们处理这些异常。

③

如果主题存在，`Future`将通过返回`TopicDescription`来完成，其中包含主题所有分区的列表，以及每个分区中作为领导者的经纪人的副本列表和同步副本列表。请注意，这不包括主题的配置。我们将在本章后面讨论配置。

④

请注意，当 Kafka 响应错误时，所有 AdminClient 结果对象都会抛出`ExecutionException`。这是因为 AdminClient 结果被包装在`Future`对象中，而这些对象包装了异常。您总是需要检查`ExecutionException`的原因以获取 Kafka 返回的错误。

⑤

如果主题不存在，我们将创建一个新主题。在创建主题时，您可以仅指定名称并对所有细节使用默认值。您还可以指定分区数、副本数和配置。

⑥

最后，您希望等待主题创建完成，并可能验证结果。在此示例中，我们正在检查分区数。由于我们在创建主题时指定了分区数，我们相当确定它是正确的。如果您在创建主题时依赖经纪人默认值，则更常见地检查结果。请注意，由于我们再次调用`get()`来检查`CreateTopic`的结果，此方法可能会抛出异常。在这种情况下，`TopicExists​Exception`很常见，您需要处理它（也许通过描述主题来检查正确的配置）。

现在我们有了一个主题，让我们删除它：

```java
admin.deleteTopics(TOPIC_LIST).all().get();

// Check that it is gone. Note that due to the async nature of deletes,
// it is possible that at this point the topic still exists
try {
    topicDescription = demoTopic.values().get(TOPIC_NAME).get();
    System.out.println("Topic " + TOPIC_NAME + " is still around");
} catch (ExecutionException e) {
    System.out.println("Topic " + TOPIC_NAME + " is gone");
}
```

此时代码应该相当熟悉。我们使用`deleteTopics`方法删除一个主题名称列表，并使用`get()`等待完成。

###### 警告

尽管代码很简单，请记住，在 Kafka 中，删除主题是最终的——没有回收站或垃圾桶可以帮助您恢复已删除的主题，也没有检查来验证主题是否为空，以及您是否真的想要删除它。删除错误的主题可能意味着无法恢复的数据丢失，因此请特别小心处理此方法。

到目前为止，所有示例都使用了不同`AdminClient`方法返回的`Future`上的阻塞`get()`调用。大多数情况下，这就是您所需要的——管理操作很少，等待操作成功或超时通常是可以接受的。有一个例外：如果您要写入一个预期处理大量管理请求的服务器。在这种情况下，您不希望在等待 Kafka 响应时阻塞服务器线程。您希望继续接受用户的请求并将其发送到 Kafka，当 Kafka 响应时，将响应发送给客户端。在这些情况下，`KafkaFuture`的多功能性就变得非常有用。这是一个简单的例子。

```java
vertx.createHttpServer().requestHandler(request -> { // ①
    String topic = request.getParam("topic"); // ②
    String timeout = request.getParam("timeout");
    int timeoutMs = NumberUtils.toInt(timeout, 1000);

    DescribeTopicsResult demoTopic = admin.describeTopics( // ③
            Collections.singletonList(topic),
            new DescribeTopicsOptions().timeoutMs(timeoutMs));

    demoTopic.values().get(topic).whenComplete( // ④
            new KafkaFuture.BiConsumer<TopicDescription, Throwable>() {
                @Override
                public void accept(final TopicDescription topicDescription,
                                   final Throwable throwable) {
                    if (throwable != null) {
                      request.response().end("Error trying to describe topic "
                              + topic + " due to " + throwable.getMessage()); // ⑤
                    } else {
                        request.response().end(topicDescription.toString()); // ⑥
                    }
                }
            });
}).listen(8080);
```

①

我们使用 Vert.x 创建一个简单的 HTTP 服务器。每当此服务器收到请求时，它将调用我们在这里定义的`requestHandler`。

②

请求包括一个主题名称作为参数，我们将用这个主题的描述作为响应。

③

我们像往常一样调用`AdminClient.describeTopics`并获得包装的`Future`作为响应。

④

我们不使用阻塞的`get()`调用，而是构造一个在`Future`完成时将被调用的函数。

⑤

如果`Future`完成时出现异常，我们会将错误发送给 HTTP 客户端。

⑥

如果`Future`成功完成，我们将使用主题描述回复客户端。

关键在于我们不会等待 Kafka 的响应。当来自 Kafka 的响应到达时，`DescribeTopic​Result`将向 HTTP 客户端发送响应。与此同时，HTTP 服务器可以继续处理其他请求。您可以通过使用`SIGSTOP`来暂停 Kafka（不要在生产环境中尝试！）并向 Vert.x 发送两个 HTTP 请求来检查此行为：一个具有较长的超时值，一个具有较短的超时值。即使您在第一个请求之后发送了第二个请求，由于较低的超时值，它将更早地响应，并且不会在第一个请求之后阻塞。

# 配置管理

配置管理是通过描述和更新`ConfigResource`集合来完成的。配置资源可以是代理、代理记录器和主题。通常使用`kafka-config.sh`或其他 Kafka 管理工具来检查和修改代理和代理记录器配置，但从使用它们的应用程序中检查和更新主题配置是非常常见的。

例如，许多应用程序依赖于正确操作的压缩主题。有意义的是，定期（比默认保留期更频繁，以确保安全）这些应用程序将检查主题是否确实被压缩，并采取行动来纠正主题配置（如果未被压缩）。

以下是一个示例：

```java
ConfigResource configResource =
        new ConfigResource(ConfigResource.Type.TOPIC, TOPIC_NAME); // ①
DescribeConfigsResult configsResult =
        admin.describeConfigs(Collections.singleton(configResource));
Config configs = configsResult.all().get().get(configResource);

// print nondefault configs
configs.entries().stream().filter(
        entry -> !entry.isDefault()).forEach(System.out::println); // ②

// Check if topic is compacted
ConfigEntry compaction = new ConfigEntry(TopicConfig.CLEANUP_POLICY_CONFIG,
        TopicConfig.CLEANUP_POLICY_COMPACT);
if (!configs.entries().contains(compaction)) {
    // if topic is not compacted, compact it
    Collection<AlterConfigOp> configOp = new ArrayList<AlterConfigOp>();
    configOp.add(new AlterConfigOp(compaction, AlterConfigOp.OpType.SET)); // ③
    Map<ConfigResource, Collection<AlterConfigOp>> alterConf = new HashMap<>();
    alterConf.put(configResource, configOp);
    admin.incrementalAlterConfigs(alterConf).all().get();
} else {
    System.out.println("Topic " + TOPIC_NAME + " is compacted topic");
}
```

①

如上所述，有几种类型的`ConfigResource`；在这里，我们正在检查特定主题的配置。您可以在同一请求中指定来自不同类型的多个不同资源。

②

`describeConfigs`的结果是从每个`ConfigResource`到一组配置的映射。每个配置条目都有一个`isDefault()`方法，让我们知道哪些配置已被修改。如果用户配置了主题以具有非默认值，或者如果修改了代理级别配置并且创建的主题从代理继承了此非默认值，则认为主题配置是非默认的。

③

要修改配置，指定要修改的`ConfigResource`的映射和一组操作。每个配置修改操作由配置条目（配置的名称和值；在本例中，`cleanup.policy`是配置名称，`compacted`是值）和操作类型组成。在 Kafka 中，有四种类型的操作可以修改配置：`SET`，用于设置配置值；`DELETE`，用于删除值并重置为默认值；`APPEND`；和`SUBSTRACT`。最后两种仅适用于具有`List`类型的配置，并允许添加和删除值，而无需每次都将整个列表发送到 Kafka。

在紧急情况下，描述配置可能会非常方便。我们记得有一次在升级过程中，代理的配置文件被意外替换为损坏的副本。在重新启动第一个代理后发现它无法启动。团队没有办法恢复原始配置，因此我们准备进行大量的试错，试图重建正确的配置并使代理恢复正常。一位站点可靠性工程师（SRE）通过连接到剩余的代理之一并使用 AdminClient 转储其配置来挽救了当天。

# 消费者组管理

我们之前提到过，与大多数消息队列不同，Kafka 允许你以先前消费和处理数据的确切顺序重新处理数据。在第四章中，我们讨论了消费者组时，解释了如何使用消费者 API 从主题中返回并重新读取旧消息。但是使用这些 API 意味着你需要提前将重新处理数据的能力编程到你的应用程序中。你的应用程序本身必须暴露“重新处理”功能。

有几种情况下，你会想要导致应用程序重新处理消息，即使这种能力事先没有内置到应用程序中。在事故期间排除应用程序故障是其中一种情况。另一种情况是在灾难恢复故障转移场景中准备应用程序在新集群上运行（我们将在第九章中更详细地讨论这一点，当我们讨论灾难恢复技术时）。

在本节中，我们将看看如何使用 AdminClient 来以编程方式探索和修改消费者组以及这些组提交的偏移量。在第十章中，我们将看看可用于执行相同操作的外部工具。

## 探索消费者组

如果你想要探索和修改消费者组，第一步是列出它们：

```java
admin.listConsumerGroups().valid().get().forEach(System.out::println);
```

通过使用`valid()`方法，`get()`将返回的集合只包含集群返回的没有错误的消费者组，如果有的话。任何错误将被完全忽略，而不是作为异常抛出。`errors()`方法可用于获取所有异常。如果像我们在其他示例中所做的那样使用`all()`，集群返回的第一个错误将作为异常抛出。这种错误的可能原因是授权，即你没有权限查看该组，或者某些消费者组的协调者不可用。

如果我们想要更多关于某些组的信息，我们可以描述它们：

```java
ConsumerGroupDescription groupDescription = admin
        .describeConsumerGroups(CONSUMER_GRP_LIST)
        .describedGroups().get(CONSUMER_GROUP).get();
        System.out.println("Description of group " + CONSUMER_GROUP
                + ":" + groupDescription);
```

描述包含了关于该组的大量信息。这包括了组成员、它们的标识符和主机、分配给它们的分区、用于分配的算法，以及组协调者的主机。在故障排除消费者组时，这个描述非常有用。关于消费者组最重要的信息之一在这个描述中缺失了——不可避免地，我们会想知道该组对于它正在消费的每个分区最后提交的偏移量是多少，以及它落后于日志中最新消息的数量。

过去，获取这些信息的唯一方法是解析消费者组写入内部 Kafka 主题的提交消息。虽然这种方法达到了其目的，但 Kafka 不保证内部消息格式的兼容性，因此不推荐使用旧方法。我们将看看 Kafka 的 AdminClient 如何允许我们检索这些信息：

```java
Map<TopicPartition, OffsetAndMetadata> offsets =
        admin.listConsumerGroupOffsets(CONSUMER_GROUP)
                .partitionsToOffsetAndMetadata().get(); // ①

Map<TopicPartition, OffsetSpec> requestLatestOffsets = new HashMap<>();

for(TopicPartition tp: offsets.keySet()) {
    requestLatestOffsets.put(tp, OffsetSpec.latest()); // ②
}

Map<TopicPartition, ListOffsetsResult.ListOffsetsResultInfo> latestOffsets =
        admin.listOffsets(requestLatestOffsets).all().get();

for (Map.Entry<TopicPartition, OffsetAndMetadata> e: offsets.entrySet()) { // ③
    String topic = e.getKey().topic();
    int partition =  e.getKey().partition();
    long committedOffset = e.getValue().offset();
    long latestOffset = latestOffsets.get(e.getKey()).offset();

    System.out.println("Consumer group " + CONSUMER_GROUP
            + " has committed offset " + committedOffset
            + " to topic " + topic + " partition " + partition
            + ". The latest offset in the partition is "
            +  latestOffset + " so consumer group is "
            + (latestOffset - committedOffset) + " records behind");
}
```

①

我们获取消费者组处理的所有主题和分区的映射，以及每个分区的最新提交的偏移量。请注意，与`describe​ConsumerGroups`不同，`listConsumerGroupOffsets`只接受单个消费者组，而不是集合。

②

对于结果中的每个主题和分区，我们想要获取分区中最后一条消息的偏移量。`OffsetSpec`有三种非常方便的实现：`earliest()`、`latest()`和`forTimestamp()`，它们允许我们获取分区中的最早和最新的偏移量，以及在指定时间之后或立即之后写入的记录的偏移量。

③

最后，我们遍历所有分区，并对于每个分区打印最后提交的偏移量、分区中的最新偏移量以及它们之间的滞后。

## 修改消费者组

到目前为止，我们只是探索了可用的信息。AdminClient 还具有修改消费者组的方法：删除组、删除成员、删除提交的偏移量和修改偏移量。这些通常由 SRE 用于构建临时工具，以从紧急情况中恢复。

在所有这些情况中，修改偏移量是最有用的。删除偏移量可能看起来像是让消费者“从头开始”的简单方法，但这实际上取决于消费者的配置 - 如果消费者启动并且找不到偏移量，它会从头开始吗？还是跳到最新的消息？除非我们有`auto.offset.reset`的值，否则我们无法知道。显式地修改提交的偏移量为最早可用的偏移量将强制消费者从主题的开头开始处理，并且基本上会导致消费者“重置”。

请记住，消费者组在偏移量在偏移量主题中发生变化时不会收到更新。它们只在分配新分区给消费者或启动时读取偏移量。为了防止您对消费者不会知道的偏移量进行更改（因此将覆盖），Kafka 将阻止您在消费者组活动时修改偏移量。

还要记住，如果消费者应用程序维护状态（大多数流处理应用程序都会维护状态），重置偏移量并导致消费者组从主题的开头开始处理可能会对存储的状态产生奇怪的影响。例如，假设您有一个流应用程序，不断计算商店销售的鞋子数量，并且假设在早上 8:00 发现输入错误，并且您想要完全重新计算自 3:00 a.m.以来的数量。如果您将偏移量重置为 3:00 a.m.，而没有适当修改存储的聚合，您将计算今天卖出的每双鞋子两次（您还将处理 3:00 a.m.和 8:00 a.m.之间的所有数据，但让我们假设这是必要的来纠正错误）。您需要小心相应地更新存储的状态。在开发环境中，我们通常在重置偏移量到输入主题的开头之前完全删除状态存储。

在脑海中牢记所有这些警告，让我们来看一个例子：

```java
Map<TopicPartition, ListOffsetsResult.ListOffsetsResultInfo> earliestOffsets =
    admin.listOffsets(requestEarliestOffsets).all().get(); // ①

Map<TopicPartition, OffsetAndMetadata> resetOffsets = new HashMap<>();
for (Map.Entry<TopicPartition, ListOffsetsResult.ListOffsetsResultInfo> e:
        earliestOffsets.entrySet()) {
  resetOffsets.put(e.getKey(), new OffsetAndMetadata(e.getValue().offset())); // ②
}

try {
  admin.alterConsumerGroupOffsets(CONSUMER_GROUP, resetOffsets).all().get(); // ③
} catch (ExecutionException e) {
  System.out.println("Failed to update the offsets committed by group "
            + CONSUMER_GROUP + " with error " + e.getMessage());
  if (e.getCause() instanceof UnknownMemberIdException)
      System.out.println("Check if consumer group is still active."); // ④
}
```

①

要重置消费者组，以便它将从最早的偏移量开始处理，我们需要首先获取最早的偏移量。获取最早的偏移量类似于获取上一个示例中显示的最新的偏移量。

②

在这个循环中，我们将`listOffsets`返回的带有`ListOffsetsResultInfo`值的映射转换为`alterConsumerGroupOffsets`所需的带有`OffsetAndMetadata`值的映射。

③

调用`alterConsumerGroupOffsets`之后，我们正在等待`Future`完成，以便我们可以看到它是否成功完成。

④

`alterConsumerGroupOffsets`失败的最常见原因之一是我们没有首先停止消费者组（这必须通过直接关闭消费应用程序来完成；没有用于关闭消费者组的管理命令）。如果组仍处于活动状态，我们尝试修改偏移量将被消费者协调器视为不是该组成员的客户端提交了该组的偏移量。在这种情况下，我们将收到`Unknown​Mem⁠berIdException`。

# 集群元数据

应用程序很少需要显式发现连接的集群的任何信息。您可以生产和消费消息，而无需了解存在多少个代理和哪一个是控制器。Kafka 客户端会将这些信息抽象化，客户端只需要关注主题和分区。

但是，以防您好奇，这段小片段将满足您的好奇心：

```java
DescribeClusterResult cluster = admin.describeCluster();

System.out.println("Connected to cluster " + cluster.clusterId().get()); // ①
System.out.println("The brokers in the cluster are:");
cluster.nodes().get().forEach(node -> System.out.println("    * " + node));
System.out.println("The controller is: " + cluster.controller().get());
```

①

集群标识符是 GUID，因此不适合人类阅读。检查您的客户端是否连接到正确的集群仍然是有用的。

# 高级管理操作

在本节中，我们将讨论一些很少使用但在需要时非常有用的方法。这些对于 SRE 在事故期间非常重要，但不要等到发生事故才学会如何使用它们。在为时已晚之前阅读和练习。请注意，这里的方法除了它们都属于这个类别之外，几乎没有任何关联。

## 向主题添加分区

通常在创建主题时会设置主题的分区数。由于每个分区的吞吐量可能非常高，因此很少会遇到主题容量限制的情况。此外，如果主题中的消息具有键，则消费者可以假定具有相同键的所有消息将始终进入同一分区，并且将由同一消费者按相同顺序处理。

出于这些原因，很少需要向主题添加分区，并且可能会有风险。您需要检查该操作是否不会破坏从主题中消费的任何应用程序。然而，有时您确实会达到现有分区可以处理的吞吐量上限，并且别无选择，只能添加一些分区。

您可以使用`createPartitions`方法向一组主题添加分区。请注意，如果尝试一次扩展多个主题，则可能会成功扩展其中一些主题，而其他主题将失败。

```java
Map<String, NewPartitions> newPartitions = new HashMap<>();
newPartitions.put(TOPIC_NAME, NewPartitions.increaseTo(NUM_PARTITIONS+2)); // ①
admin.createPartitions(newPartitions).all().get();
```

①

在扩展主题时，您需要指定主题在添加分区后将拥有的总分区数，而不是新分区的数量。

###### 提示

由于`createPartition`方法将主题中新分区添加后的总分区数作为参数，因此您可能需要描述主题并找出在扩展之前存在多少分区。

## 从主题中删除记录

当前的隐私法律规定了数据的特定保留政策。不幸的是，虽然 Kafka 有主题的保留政策，但它们并没有以确保合法合规的方式实施。如果一个主题的保留政策是 30 天，如果所有数据都适合每个分区中的单个段，那么它可以存储旧数据。

`deleteRecords`方法将标记所有偏移量早于调用该方法时指定的偏移量的记录为已删除，并使它们对 Kafka 消费者不可访问。该方法返回最高的已删除偏移量，因此我们可以检查删除是否确实按预期发生。磁盘上的完全清理将异步进行。请记住，`listOffsets`方法可用于获取在特定时间之后或立即之后编写的记录的偏移量。这些方法可以一起用于删除早于任何特定时间点的记录：

```java
Map<TopicPartition, ListOffsetsResult.ListOffsetsResultInfo> olderOffsets =
        admin.listOffsets(requestOlderOffsets).all().get();
Map<TopicPartition, RecordsToDelete> recordsToDelete = new HashMap<>();
for (Map.Entry<TopicPartition, ListOffsetsResult.ListOffsetsResultInfo>  e:
        olderOffsets.entrySet())
    recordsToDelete.put(e.getKey(),
            RecordsToDelete.beforeOffset(e.getValue().offset()));
 admin.deleteRecords(recordsToDelete).all().get();
```

## 领导者选举

这种方法允许您触发两种不同类型的领导者选举：

首选领导者选举

每个分区都有一个被指定为*首选领导者*的副本。它是首选的，因为如果所有分区都使用其首选领导者副本作为领导者，每个代理上的领导者数量应该是平衡的。默认情况下，Kafka 每五分钟会检查首选领导者副本是否确实是领导者，如果不是但有资格成为领导者，它将选举首选领导者副本为领导者。如果`auto.leader.rebalance.enable`为`false`，或者如果您希望此过程更快地发生，`electLeader()`方法可以触发此过程。

非干净领导者选举

如果分区的领导者副本变得不可用，并且其他副本没有资格成为领导者（通常是因为它们缺少数据），那么该分区将没有领导者，因此不可用。解决此问题的一种方法是触发*非干净领导者*选举，这意味着将一个否则没有资格成为领导者的副本选举为领导者。这将导致数据丢失——所有写入旧领导者但尚未复制到新领导者的事件将丢失。`electLeader()`方法也可以用于触发非干净领导者选举。

该方法是异步的，这意味着即使在成功返回后，直到所有代理都意识到新状态并调用`describeTopics()`后，调用可能会返回不一致的结果。如果触发多个分区的领导者选举，可能会对一些分区成功，对另一些分区失败：

```java
Set<TopicPartition> electableTopics = new HashSet<>();
electableTopics.add(new TopicPartition(TOPIC_NAME, 0));
try {
    admin.electLeaders(ElectionType.PREFERRED, electableTopics).all().get(); // ①
} catch (ExecutionException e) {
    if (e.getCause() instanceof ElectionNotNeededException) {
        System.out.println("All leaders are preferred already"); // ②
    }
}
```

①

我们正在为特定主题的单个分区选举首选领导者。我们可以指定任意数量的分区和主题。如果您使用`null`而不是分区集合调用该命令，它将触发您选择的所有分区的选举类型。

②

如果集群处于健康状态，该命令将不起作用。只有当首选领导者以外的副本是当前领导者时，首选领导者选举和非干净领导者选举才会生效。

## 重新分配副本

有时，您可能不喜欢某些副本的当前位置。也许一个代理已经过载，您想要移动一些副本。也许您想要添加更多的副本。也许您想要从一个代理中移动所有副本，以便您可以移除该机器。或者也许一些主题太吵了，您需要将它们与其余工作负载隔离开来。在所有这些情况下，`alterPartitionReassignments`可以让您对每个分区的每个副本的放置进行精细控制。请记住，将副本从一个代理重新分配到另一个代理可能涉及从一个代理复制大量数据到另一个代理。请注意可用的网络带宽，并根据需要使用配额限制复制；配额是代理配置，因此您可以使用`AdminClient`来描述它们并更新它们。

在本例中，假设我们有一个 ID 为 0 的单个代理。我们的主题有几个分区，每个分区都有一个副本在这个代理上。添加新代理后，我们希望使用它来存储主题的一些副本。我们将以稍微不同的方式为主题中的每个分区分配副本：

```java
Map<TopicPartition, Optional<NewPartitionReassignment>> reassignment = new HashMap<>();
reassignment.put(new TopicPartition(TOPIC_NAME, 0),
        Optional.of(new NewPartitionReassignment(Arrays.asList(0,1)))); // ①
reassignment.put(new TopicPartition(TOPIC_NAME, 1),
        Optional.of(new NewPartitionReassignment(Arrays.asList(1)))); // ②
reassignment.put(new TopicPartition(TOPIC_NAME, 2),
        Optional.of(new NewPartitionReassignment(Arrays.asList(1,0)))); // ③
reassignment.put(new TopicPartition(TOPIC_NAME, 3), Optional.empty()); // ④

admin.alterPartitionReassignments(reassignment).all().get();

System.out.println("currently reassigning: " +
        admin.listPartitionReassignments().reassignments().get()); // ⑤
demoTopic = admin.describeTopics(TOPIC_LIST);
topicDescription = demoTopic.values().get(TOPIC_NAME).get();
System.out.println("Description of demo topic:" + topicDescription); // ⑥
```

①

我们已经向分区 0 添加了另一个副本，将新副本放在了 ID 为 1 的新代理上，但保持领导者不变。

②

我们没有向分区 1 添加任何副本；我们只是将现有的一个副本移动到新代理上。由于我们只有一个副本，它也是领导者。

③

我们已经向分区 2 添加了另一个副本，并将其设置为首选领导者。下一个首选领导者选举将把领导权转移到新经纪人上的新副本。现有副本将成为跟随者。

④

分区 3 没有正在进行的重新分配，但如果有的话，这将取消它并将状态返回到重新分配操作开始之前的状态。

⑤

我们可以列出正在进行的重新分配。

⑥

我们也可以打印新状态，但请记住，直到显示一致的结果可能需要一段时间。

# 测试

Apache Kafka 提供了一个测试类`MockAdminClient`，您可以用任意数量的经纪人初始化它，并用它来测试您的应用程序是否正确运行，而无需运行实际的 Kafka 集群并真正执行管理操作。虽然`MockAdminClient`不是 Kafka API 的一部分，因此可能会在没有警告的情况下发生变化，但它模拟了公共方法，因此方法签名将保持兼容。在这个类的便利性是否值得冒这个风险的问题上存在一些权衡，所以请记住这一点。 

这个测试类特别引人注目的地方在于一些常见方法有非常全面的模拟：您可以使用`MockAdminClient`创建主题，然后调用`listTopics()`将列出您“创建”的主题。

但并非所有方法都被模拟。如果您使用版本为 2.5 或更早的`AdminClient`并调用`MockAdminClient`的`incrementalAlterConfigs()`，您将收到一个`UnsupportedOperationException`，但您可以通过注入自己的实现来处理这个问题。

为了演示如何使用`MockAdminClient`进行测试，让我们从实现一个类开始，该类实例化为一个管理客户端，并使用它来创建主题：

```java
public TopicCreator(AdminClient admin) {
    this.admin = admin;
}

// Example of a method that will create a topic if its name starts with "test"
public void maybeCreateTopic(String topicName)
        throws ExecutionException, InterruptedException {
    Collection<NewTopic> topics = new ArrayList<>();
    topics.add(new NewTopic(topicName, 1, (short) 1));
    if (topicName.toLowerCase().startsWith("test")) {
        admin.createTopics(topics);

        // alter configs just to demonstrate a point
        ConfigResource configResource =
                  new ConfigResource(ConfigResource.Type.TOPIC, topicName);
        ConfigEntry compaction =
                  new ConfigEntry(TopicConfig.CLEANUP_POLICY_CONFIG,
                          TopicConfig.CLEANUP_POLICY_COMPACT);
        Collection<AlterConfigOp> configOp = new ArrayList<AlterConfigOp>();
        configOp.add(new AlterConfigOp(compaction, AlterConfigOp.OpType.SET));
        Map<ConfigResource, Collection<AlterConfigOp>> alterConf =
            new HashMap<>();
        alterConf.put(configResource, configOp);
        admin.incrementalAlterConfigs(alterConf).all().get();
    }
}
```

这里的逻辑并不复杂：如果主题名称以“test”开头，`maybeCreateTopic`将创建主题。我们还修改了主题配置，以便演示我们如何处理我们使用的方法在模拟客户端中未实现的情况。

###### 注意

我们使用[Mockito](https://site.mockito.org)测试框架来验证`MockAdminClient`方法是否按预期调用，并填充未实现的方法。Mockito 是一个相当简单的模拟框架，具有良好的 API，非常适合用于单元测试的小例子。

我们将通过实例化我们的模拟客户端来开始测试：

```java
@Before
public void setUp() {
    Node broker = new Node(0,"localhost",9092);
    this.admin = spy(new MockAdminClient(Collections.singletonList(broker),
        broker)); // ①

    // without this, the tests will throw
    // `java.lang.UnsupportedOperationException: Not implemented yet`
    AlterConfigsResult emptyResult = mock(AlterConfigsResult.class);
    doReturn(KafkaFuture.completedFuture(null)).when(emptyResult).all();
    doReturn(emptyResult).when(admin).incrementalAlterConfigs(any()); // ②
}
```

①

`MockAdminClient`是用经纪人列表（这里我们只使用一个）和一个将成为我们控制器的经纪人实例化的。经纪人只是经纪人 ID，主机名和端口 - 当然都是假的。在执行这些测试时，不会运行任何经纪人。我们将使用 Mockito 的`spy`注入，这样我们以后可以检查`TopicCreator`是否执行正确。

②

在这里，我们使用 Mockito 的`doReturn`方法来确保模拟管理客户端不会抛出异常。我们正在测试的方法期望具有`AlterConfig​Result`对象，该对象具有返回`KafkaFuture`的`all()`方法。我们确保虚假的`incrementalAlterConfigs`确实返回了这一点。

现在我们有了一个适当的虚假 AdminClient，我们可以使用它来测试`maybeCreateTopic()`方法是否正常工作：

```java
@Test
public void testCreateTestTopic()
        throws ExecutionException, InterruptedException {
    TopicCreator tc = new TopicCreator(admin);
    tc.maybeCreateTopic("test.is.a.test.topic");
    verify(admin, times(1)).createTopics(any()); // ①
}

@Test
public void testNotTopic() throws ExecutionException, InterruptedException {
    TopicCreator tc = new TopicCreator(admin);
    tc.maybeCreateTopic("not.a.test");
    verify(admin, never()).createTopics(any()); // ②
}
```

①

主题名称以“test”开头，因此我们期望`maybeCreateTopic()`创建一个主题。我们检查`createTopics()`是否被调用了一次。

②

当主题名称不以“test”开头时，我们验证`createTopics()`根本没有被调用。

最后一点说明：Apache Kafka 发布了`MockAdminClient`在一个测试 jar 中，所以确保你的*pom.xml*包含一个测试依赖：

```java
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.5.0</version>
    <classifier>test</classifier>
    <scope>test</scope>
</dependency>
```

# 总结

AdminClient 是 Kafka 开发工具包中很有用的工具。对于想要动态创建主题并验证其配置是否正确的应用程序开发人员来说，它非常有用。对于运维人员和 SREs 来说，他们想要围绕 Kafka 创建工具和自动化，或者需要从事故中恢复时，AdminClient 也非常有用。AdminClient 有很多有用的方法，SREs 可以把它看作是 Kafka 操作的瑞士军刀。

在本章中，我们涵盖了使用 Kafka 的 AdminClient 的所有基础知识：主题管理、配置管理和消费者组管理，以及一些其他有用的方法，这些方法在需要时都很有用。
