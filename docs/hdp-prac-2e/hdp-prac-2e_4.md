## 附录. 安装 Hadoop 及相关工具

本附录包含有关如何安装 Hadoop 以及书中使用的其他工具的说明。


##### 快速开始使用 Hadoop

使用预安装的虚拟机是快速开始使用 Hadoop 的最快方式之一。以下是一些流行的虚拟机列表：

+   Cloudera Quickstart VM—[`www.cloudera.com/content/cloudera-content/cloudera-docs/DemoVMs/Cloudera-QuickStart-VM/cloudera_quickstart_vm.html`](http://www.cloudera.com/content/cloudera-content/cloudera-docs/DemoVMs/Cloudera-QuickStart-VM/cloudera_quickstart_vm.html)

+   Hortonworks Sandbox—[`hortonworks.com/products/hortonworkssandbox/`](http://hortonworks.com/products/hortonworkssandbox/)

+   MapR Sandbox for Hadoop—[`doc.mapr.com/display/MapR/MapR+Sandbox+for+Hadoop`](http://doc.mapr.com/display/MapR/MapR+Sandbox+for+Hadoop)


### A.1\. 书中的代码

在我们进入安装 Hadoop 的说明之前，让我们先为您设置好这本书所附带的代码。代码托管在 GitHub 上，网址为 [`github.com/alexholmes/hiped2`](https://github.com/alexholmes/hiped2)。为了让您快速启动，有一些预包装的 tarball，您无需构建代码——只需安装即可。

#### 下载

首先，您需要从 [`github.com/alexholmes/hiped2/releases`](https://github.com/alexholmes/hiped2/releases) 下载代码的最新版本。

#### 安装

第二步是将 tarball 解压到您选择的目录中。例如，以下操作将代码解压到 /usr/local 目录，这是您将安装 Hadoop 的同一目录：

```
$ cd /usr/local
$ sudo tar -xzvf <download directory>/hip-<version>-package.tar.gz
```

#### 将主目录添加到您的路径中

书中的所有示例都假设代码的主目录在您的路径中。完成此操作的方法因操作系统和 shell 而异。如果您使用的是 Linux Bash，则以下命令应该可以工作（第二个命令需要使用单引号以避免变量替换）：

```
$ echo "export HIP_HOME=/usr/local/hip-<version>" >> ~/.bash_profile
$ echo 'export PATH=${PATH}:${HIP_HOME}/bin' >> ~/.bash_profile
```

#### 运行示例作业

您可以使用以下命令来测试您的安装。这假设您有一个正在运行的 Hadoop 设置（如果您没有，请跳转到 章节 A.3）：

```
# create two input files in HDFS
$ hadoop fs -mkdir -p hip/input
$ echo "cat sat mat" | hadoop fs -put - hip/input/1.txt
$ echo "dog lay mat" | hadoop fs -put - hip/input/2.txt

# run the inverted index example
$ hip hip.ch1.InvertedIndexJob --input hip/input --output hip/output

# examine the results in HDFS
$ hadoop fs -cat hip/output/part*
```

#### 下载源代码并构建

有些技术（如 Avro 代码生成）需要访问完整源代码。首先，使用 git 检出源代码：

```
$ git clone git@github.com:alexholmes/hiped2.git
```

设置您的环境，以便某些技术知道源代码安装的位置：

```
$ echo "export HIP_SRC=<installation dir>/hiped2" >> ~/.bash_profile
```

您可以使用 Maven 构建项目：

```
$ cd hiped2
$ mvn clean validate package
```

这将生成一个 target/hip-<version>-package.tar.gz 文件，这是在发布时上传到 GitHub 的同一文件。

### A.2\. 推荐的 Java 版本

Hadoop 项目维护了一个推荐列表，列出了在生产环境中与 Hadoop 一起工作表现良好的 Java 版本。有关详细信息，请参阅 Hadoop Wiki 上的“Hadoop Java Versions”页面 [`wiki.apache.org/hadoop/HadoopJavaVersions`](http://wiki.apache.org/hadoop/HadoopJavaVersions)。

### A.3\. Hadoop

本节涵盖 Apache Hadoop 分发的安装、配置和运行。如果你使用的是其他 Hadoop 分发版，请参考特定分发的说明。

#### Apache tarball 安装

以下说明适用于想要安装 Apache Hadoop 分发版 tarball 版本的用户。这是一个伪分布式设置，不适用于多节点集群.^([1])

> ¹ 伪分布式模式是指所有 Hadoop 组件都在单个主机上运行。

首先，你需要从 Apache 下载页面 [`hadoop.apache.org/common/releases.html#Download`](http://hadoop.apache.org/common/releases.html#Download) 下载 tarball，并在 /usr/local 下解压：

```
$ cd /usr/local
$ sudo tar -xzf <path-to-apache-tarball>

$ sudo ln -s hadoop-<version> hadoop

$ sudo chown -R <user>:<group> /usr/local/hadoop*
$ mkdir /usr/local/hadoop/tmp
```

| |
| --- |

##### 没有 root 权限的用户安装目录

如果你没有主机上的 root 权限，你可以在不同的目录下安装 Hadoop，并在以下说明中将 /usr/local 替换为你的目录名。

| |
| --- |

#### Hadoop 1 及更早版本的伪分布式模式配置

以下说明适用于 Hadoop 1 及更早版本。如果你正在使用 Hadoop 2，请跳到下一节。

编辑文件 /usr/local/hadoop/conf/core-site.xml，并确保其看起来如下所示：

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>

  <property>
    <name>hadoop.tmp.dir</name>
    <value>/usr/local/hadoop/tmp</value>
  </property>

  <property>
    <name>fs.default.name</name>

    <value>hdfs://localhost:8020</value>
  </property>

</configuration>
```

然后编辑文件 /usr/local/hadoop/conf/hdfs-site.xml，并确保其看起来如下所示：

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
     <!-- specify this so that running 'hadoop namenode -format'
          formats the right dir -->
     <name>dfs.name.dir</name>
     <value>/usr/local/hadoop/cache/hadoop/dfs/name</value>
  </property>
</configuration>
```

最后，编辑文件 /usr/local/hadoop/conf/mapred-site.xml，并确保其看起来如下所示（你可能首先需要将 mapred-site.xml.template 复制到 mapred-site.xml）：

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
  <property>
    <name>mapred.job.tracker</name>
    <value>localhost:8021</value>
  </property>
</configuration>
```

#### Hadoop 2 的伪分布式模式配置

以下说明适用于 Hadoop 2。如果你正在使用 Hadoop 1 及更早版本，请参阅上一节。

编辑文件 /usr/local/hadoop/etc/hadoop/core-site.xml，并确保其看起来如下所示：

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>

  <property>

    <name>hadoop.tmp.dir</name>
    <value>/usr/local/hadoop/tmp</value>
  </property>

  <property>
    <name>fs.default.name</name>
    <value>hdfs://localhost:8020</value>
  </property>

</configuration>
```

然后编辑文件 /usr/local/hadoop/etc/hadoop/hdfs-site.xml，并确保其看起来如下所示：

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
</configuration>
```

接下来，编辑文件 /usr/local/hadoop/etc/hadoop/mapred-site.xml，并确保其看起来如下所示：

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
```

最后，编辑文件 /usr/local/hadoop/etc/hadoop/yarn-site.xml，并确保其看起来如下所示：

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
    <description>Shuffle service that needs to be set for
                       Map Reduce to run.</description>
  </property>
  <property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
  </property>
  <property>

    <name>yarn.log-aggregation.retain-seconds</name>
    <value>2592000</value>
  </property>
  <property>
    <name>yarn.log.server.url</name>
    <value>http://0.0.0.0:19888/jobhistory/logs/</value>
  </property>
  <property>
    <name>yarn.nodemanager.delete.debug-delay-sec</name>
    <value>-1</value>
    <description>Amount of time in seconds to wait before
                 deleting container resources.</description>
  </property>
</configuration>
```

#### 设置 SSH

Hadoop 使用 Secure Shell (SSH) 在伪分布式模式下远程启动进程，如 Data-Node 和 TaskTracker，即使所有内容都在单个节点上运行。如果你还没有 SSH 密钥对，可以使用以下命令创建一个：

```
$ ssh-keygen -b 2048 -t rsa
```

你需要将 .ssh/id_rsa 文件复制到 authorized_keys 文件中：

```
$ cp ~/.ssh/id_rsa.pub  ~/.ssh/authorized_keys
```

你还需要运行一个 SSH 代理，这样在启动和停止 Hadoop 时就不会被要求输入密码无数次。不同的操作系统有不同的运行 SSH 代理的方式，CentOS 和其他 Red Hat 衍生版^([2]) 以及 OS X 的详细信息可以在网上找到。如果你运行的是不同的系统，Google 是你的朋友。

> ² 请参阅 Red Hat 部署指南中关于“配置 ssh-agent”的[www.centos.org/docs/5/html/5.2/Deployment_Guide/s3-openssh-config-ssh-agent.html](http://www.centos.org/docs/5/html/5.2/Deployment_Guide/s3-openssh-config-ssh-agent.html)部分。
> 
> ³ 请参阅“在 Mac OS X Leopard 中使用 SSH Agent”的[www-uxsup.csx.cam.ac.uk/~aia21/osx/leopard-ssh.html](http://www-uxsup.csx.cam.ac.uk/~aia21/osx/leopard-ssh.html)。

要验证代理正在运行并且已加载您的密钥，请尝试打开到本地系统的 SSH 连接：

```
$ ssh 127.0.0.1
```

如果您被提示输入密码，则代理没有运行或没有加载您的密钥。

#### Java

您需要在您的系统上安装当前版本的 Java（1.6 或更高版本）。您需要确保系统路径包括您的 Java 安装的二进制目录。或者，您可以编辑 /usr/local/hadoop/conf/hadoop-env.sh，取消注释 JAVA_HOME 行，并使用您的 Java 安装位置更新值。

#### 环境设置

为了方便起见，建议您将 Hadoop 二进制目录添加到您的路径中。以下代码显示了您可以在 ~/.bash_profile（假设您正在运行 Bash）的底部添加的内容：

```
HADOOP_HOME=/usr/local/hadoop
PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export PATH
```

#### 格式化 HDFS

接下来您需要格式化 HDFS。本节中此后的命令假设 Hadoop 二进制目录已存在于您的路径中，如前所述。在 Hadoop 1 及更早版本上，键入

```
$ hadoop namenode -format
```

在 Hadoop 2 及更高版本上，键入

```
$ hdfs namenode -format
```

在 HDFS 格式化后，您就可以开始使用 Hadoop 了。

#### 启动 Hadoop 1 及更早版本

在版本 1 及更早版本上，可以使用单个命令启动 Hadoop：

```
$ start-all.sh
```

运行启动脚本后，使用 `jps` Java 工具检查所有进程是否正在运行。您应该看到以下输出（进程 ID 除外，将不同）：

```
$ jps
23836 JobTracker
23475 NameNode
23982 TaskTracker
23619 DataNode
24024 Jps
23756 SecondaryNameNode
```

如果这些进程中的任何一个没有运行，请检查日志目录 (/usr/local/hadoop/logs)，以查看进程为什么无法正确启动。前面提到的每个进程都有两个可以通过名称识别的输出文件，应该检查是否有错误。

最常见的错误是前面展示的 HDFS 格式化步骤被跳过了。

#### 启动 Hadoop 2

启动 Hadoop 版本 2 需要以下命令：

```
$ yarn-daemon.sh start resourcemanager
$ yarn-daemon.sh start nodemanager
$ hadoop-daemon.sh start namenode
$ hadoop-daemon.sh start datanode
$ mr-jobhistory-daemon.sh start historyserver
```

运行启动脚本后，使用 `jps` Java 工具检查所有进程是否正在运行。您应该看到以下输出，尽管顺序和进程 ID 将不同：

```
$ jps
32542 NameNode
1085  Jps
32131 ResourceManager
32613 DataNode
32358 NodeManager
1030  JobHistoryServer
```

如果这些进程中的任何一个没有运行，请检查日志目录 (/usr/local/hadoop/logs)，以查看进程为什么无法正确启动。前面提到的每个进程都有两个可以通过名称识别的输出文件，应该检查是否有错误。最常见错误是前面展示的 HDFS 格式化步骤被跳过了。

#### 在 HDFS 上为您的用户创建一个家目录

一旦 Hadoop 启动并运行，您首先想要做的是为您自己的用户创建一个主目录。如果您在 Hadoop 1 上运行，命令是

```
$ hadoop fs -mkdir /user/<your-linux-username>
```

在 Hadoop 2 上，您将运行

```
$ hdfs dfs -mkdir -p /user/<your-linux-username>
```

#### 验证安装

以下命令可以用来测试您的 Hadoop 安装。前两个命令在 HDFS 中创建一个目录并创建一个文件：

```
$ hadoop fs -mkdir /tmp
$ echo "the cat sat on the mat" | hadoop fs -put - /tmp/input.txt
```

接下来，您想运行一个单词计数 MapReduce 作业。在 Hadoop 1 及更早版本上，运行以下命令：

```
$ hadoop jar /usr/local/hadoop/*-examples*.jar wordcount \
  /tmp/input.txt /tmp/output
```

在 Hadoop 2 上，运行以下命令：

```
$ hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/*-examples*.jar \
  wordcount /tmp/input.txt /tmp/output
```

检查并验证 HDFS 上的 MapReduce 作业输出（输出将根据您用于作业输入的配置文件内容而有所不同）：

```
$ hadoop fs -cat /tmp/output/part*
at    1
mat    1
on    1
sat    1
the    2
```

#### 停止 Hadoop 1

要停止 Hadoop 1，请使用以下命令：

```
$ stop-all.sh
```

#### 停止 Hadoop 2

要停止 Hadoop 2，请使用以下命令：

```
$ mr-jobhistory-daemon.sh stop historyserver
$ hadoop-daemon.sh stop datanode
$ hadoop-daemon.sh stop namenode
$ yarn-daemon.sh stop nodemanager
$ yarn-daemon.sh stop resourcemanager
```

就像启动一样，`jps`命令可以用来验证所有 Hadoop 进程是否已停止。

#### Hadoop 1.x UI 端口

Hadoop 中有许多 Web 应用程序。表 A.1 列出了它们，包括它们运行的端口和 URL（假设它们运行在本地主机上，如果您有一个伪分布式安装运行，情况就是这样）。

##### 表 A.1. Hadoop 1.x Web 应用程序和端口

| 组件 | 默认端口 | 配置参数 | 本地 URL |
| --- | --- | --- | --- |
| MapReduce 作业跟踪器 | 50030 | mapred.job.tracker.http.address | http://127.0.0.1:50030/ |
| MapReduce 任务跟踪器 | 50060 | mapred.task.tracker.http.address | http://127.0.0.1:50060/ |
| HDFS NameNode | 50070 | dfs.http.address | http://127.0.0.1:50070/ |
| HDFS 数据节点 | 50075 | dfs.datanode.http.address | http://127.0.0.1:50075/ |
| HDFS 辅助-NameNode | 50090 | dfs.secondary.http.address | http://127.0.0.1:50090/ |
| HDFS 备份和检查点节点 | 50105 | dfs.backup.http.address | http://127.0.0.1:50105/ |

每个这些 URL 都支持以下常见路径：

+   ***/logs*** — 这显示了 hadoop.log.dir 下所有文件的列表。默认情况下，这位于每个 Hadoop 节点的$HADOOP_HOME/logs 下。

+   ***/logLevel*** — 这可以用来查看和设置 Java 包的日志级别。

+   ***/metrics*** — 这显示了 JVM 和组件级别的统计信息。它在 Hadoop 0.21 及更高版本中可用（不在 1.0、0.20.x 或更早版本中）。

+   ***/stacks*** — 这显示了守护进程中所有当前 Java 线程的堆栈转储。

#### Hadoop 2.x UI 端口

Hadoop 中有许多 Web 应用程序。表 A.2 列出了它们，包括它们运行的端口和 URL（假设它们运行在本地主机上，如果您有一个伪分布式安装运行，情况就是这样）。

##### 表 A.2. Hadoop 2.x Web 应用程序和端口

| 组件 | 默认端口 | 配置参数 | 本地 URL |
| --- | --- | --- | --- |
| YARN 资源管理器 | 8088 | yarn.resourcemanager.webapp.address | http://localhost:8088/cluster |
| YARN 节点管理器 | 8042 | yarn.nodemanager.webapp.address | http://localhost:8042/node |
| MapReduce 作业历史记录 | 19888 | mapreduce.jobhistory.webapp.address | http://localhost:19888/jobhistory |
| HDFS 名称节点 | 50070 | dfs.http.address | http://127.0.0.1:50070/ |
| HDFS 数据节点 | 50075 | dfs.datanode.http.address | http://127.0.0.1:50075/ |

### A.4\. Flume

Flume 是一个日志收集和分发系统，可以将数据传输到大量主机上的 HDFS。它是由 Cloudera 最初开发的 Apache 项目。

第五章 包含了关于 Flume 及其使用方法的章节。

#### 获取更多信息

表 A.3 列出了一些有用的资源，以帮助您更熟悉 Flume。

##### 表 A.3\. 有用资源

| 资源 | 网址 |
| --- | --- |
| Flume 主页 | [`flume.apache.org/`](http://flume.apache.org/) |
| Flume 用户指南 | [`flume.apache.org/FlumeUserGuide.html`](http://flume.apache.org/FlumeUserGuide.html) |
| Flume 入门指南 | [`cwiki.apache.org/confluence/display/FLUME/Getting+Started`](https://cwiki.apache.org/confluence/display/FLUME/Getting+Started) |

#### 在 Apache Hadoop 1.x 系统上的安装

按照资源中引用的入门指南进行操作。

#### 在 Apache Hadoop 2.x 系统上的安装

如果您试图让 Flume 1.4 与 Hadoop 2 一起工作，请按照入门指南安装 Flume。接下来，您需要从 Flume 的 lib 目录中删除 protobuf 和 guava JARs，因为它们与 Hadoop 2 捆绑的版本冲突：

```
$ mv ${flume_bin}/lib/{protobuf-java-2.4.1.jar,guava-10.0.1.jar} ~/
```

### A.5\. Oozie

Oozie 是一个起源于雅虎的 Apache 项目。它是一个 Hadoop 工作流引擎，用于管理数据处理活动。

#### 获取更多信息

表 A.4 列出了一些有用的资源，以帮助您更熟悉 Oozie。

##### 表 A.4\. 有用资源

| 资源 | 网址 |
| --- | --- |
| Oozie 项目页面 | [`oozie.apache.org/`](https://oozie.apache.org/) |
| Oozie 快速入门 | [`oozie.apache.org/docs/4.0.0/DG_QuickStart.html`](https://oozie.apache.org/docs/4.0.0/DG_QuickStart.html) |
| 其他 Oozie 资源 | [`oozie.apache.org/docs/4.0.0/index.html`](https://oozie.apache.org/docs/4.0.0/index.html) |

#### 在 Hadoop 1.x 系统上的安装

按照快速入门指南安装 Oozie。Oozie 文档中包含安装说明。

如果您正在使用 Oozie 4.4.0 并针对 Hadoop 2.2.0，您需要运行以下命令来修补您的 Maven 文件并执行构建：

```
cd oozie-4.0.0/
find . -name pom.xml | xargs sed -ri 's/(2.2.0\-SNAPSHOT)/2.2.0/'
mvn -DskipTests=true -P hadoop-2 clean package assembly:single
```

#### 在 Hadoop 2.x 系统上的安装

不幸的是，Oozie 4.0.0 与 Hadoop 2 不兼容。为了使 Oozie 与 Hadoop 一起工作，您首先需要从项目页面下载 4.0.0 的 tarball，然后解包它。接下来，运行以下命令以更改目标 Hadoop 版本：

```
$ cd oozie-4.0.0/
$ find . -name pom.xml | xargs sed -ri 's/(2.2.0\-SNAPSHOT)/2.2.0/'
```

现在您只需要在 Maven 中针对 hadoop-2 配置文件：

```
$ mvn -DskipTests=true -P hadoop-2 clean package assembly:single
```

### A.6\. Sqoop

Sqoop 是一种工具，用于将数据从关系型数据库导入到 Hadoop，反之亦然。它支持任何 JDBC 兼容的数据库，并且它还提供了用于高效数据传输到 MySQL 和 PostgreSQL 的本地连接器。

第五章 包含了使用 Sqoop 进行导入和导出的详细信息。

#### 获取更多信息

表 A.5 列出了一些有用的资源，可以帮助您更熟悉 Sqoop。

##### 表 A.5\. 有用资源

| 资源 | URL |
| --- | --- |
| Sqoop 项目页面 | [`sqoop.apache.org/`](http://sqoop.apache.org/) |
| Sqoop 用户指南 | [`sqoop.apache.org/docs/1.4.4/SqoopUserGuide.html`](http://sqoop.apache.org/docs/1.4.4/SqoopUserGuide.html) |

#### | 安装 |

从项目页面下载 Sqoop tarball。选择与您的 Hadoop 安装匹配的版本，并解压 tarball。以下说明假设您正在 /usr/local 下安装：

```
$ sudo tar -xzf \
    sqoop-<version>.bin.hadoop-<hadoop-version>.tar.gz \
    -C /usr/local/
$ ln -s /usr/local/sqoop-<version> /usr/local/sqoop
```


##### Sqoop 2

本书目前涵盖 Sqoop 版本 1。在选择要下载的 tarball 时，请注意，1.99.x 版本及更高版本是 Sqoop 2 版本，因此请确保选择一个较旧的版本。


如果您计划使用 Sqoop 与 MySQL 一起使用，您需要从 [`dev.mysql.com/downloads/connector/j/`](http://dev.mysql.com/downloads/connector/j/) 下载 MySQL JDBC 驱动程序的 tarball，将其解压到一个目录中，然后将 JAR 文件复制到 Sqoop lib 目录：

```
$ tar -xzf mysql-connector-java-<version>.tar.gz
$ cd mysql-connector-java-<version>
$ sudo cp mysql-connector-java-<version>-bin.jar \
  /usr/local/sqoop/lib
```

运行 Sqoop 时，您可能需要设置一些环境变量。它们列在 表 A.6 中。

##### 表 A.6\. Sqoop 环境变量

| 环境变量 | 描述 |
| --- | --- |
| JAVA_HOME | Java 安装所在的目录。如果您在 Red Hat 上安装了 Sun JDK，这将是指 /usr/java/latest。 |
| HADOOP_HOME | 您 Hadoop 安装所在的目录。 |
| HIVE_HOME | 仅在您计划使用 Hive 与 Sqoop 一起使用时才需要。指 Hive 安装所在的目录。 |
| HBASE_HOME | 仅在您计划使用 Sqoop 与 HBase 一起使用时才需要。指 HBase 安装所在的目录。 |

/usr/local/sqoop/bin 目录包含 Sqoop 的二进制文件。第五章 包含了展示如何使用二进制文件进行导入和导出的多种技术。

### A.7\. HBase

HBase 是一个基于 Google 的 BigTable 模式的实时、键/值、分布式、基于列的数据库。

#### 获取更多信息

表 A.7 列出了一些有用的资源，可以帮助您更熟悉 HBase。

##### 表 A.7\. 有用资源

| 资源 | URL |
| --- | --- |
| Apache HBase 项目页面 | [`hbase.apache.org/`](http://hbase.apache.org/) |
| Apache HBase 快速入门 | [`hbase.apache.org/book/quickstart.html`](http://hbase.apache.org/book/quickstart.html) |
| Apache HBase 参考指南 | [`hbase.apache.org/book/book.html`](http://hbase.apache.org/book/book.html) |
| Cloudera 关于 HBase 的“做与不做”博客文章 | [`blog.cloudera.com/blog/2011/04/hbase-dos-and-donts/`](http://blog.cloudera.com/blog/2011/04/hbase-dos-and-donts/) |

#### 安装

按照位于[`hbase.apache.org/book/quickstart.html`](https://hbase.apache.org/book/quickstart.html)的快速入门指南中的安装说明进行操作。

### A.8\. Kafka

Kafka 是由 LinkedIn 构建的一个发布/订阅消息系统。

#### 获取更多信息

表 A.8 列出了一些有用资源，以帮助您更熟悉 Kafka。

##### 表 A.8\. 有用资源

| 资源 | URL |
| --- | --- |
| Kafka 项目页面 | [`kafka.apache.org/`](http://kafka.apache.org/) |
| Kafka 文档 | [`kafka.apache.org/documentation.html`](http://kafka.apache.org/documentation.html) |
| Kafka 快速入门 | [`kafka.apache.org/08/quickstart.html`](http://kafka.apache.org/08/quickstart.html) |

#### 安装

按照快速入门指南中的安装说明进行操作。

### A.9\. Camus

Camus 是一个将 Kafka 中的数据导入 Hadoop 的工具。

#### 获取更多信息

表 A.9 列出了一些有用资源，以帮助您更熟悉 Camus。

##### 表 A.9\. 有用资源

| 资源 | URL |
| --- | --- |
| Camus 项目页面 | [`github.com/linkedin/camus`](https://github.com/linkedin/camus) |
| Camus 概述 | [`github.com/linkedin/camus/wiki/Camus-Overview`](https://github.com/linkedin/camus/wiki/Camus-Overview) |

#### 在 Hadoop 1 上安装

从 GitHub 的 0.8 分支下载代码，并运行以下命令来构建它：

```
$ mvn clean package
```

#### 在 Hadoop 2 上安装

在撰写本文时，Camus 的 0.8 版本不支持 Hadoop 2。您有几个选项可以使它工作——如果您只是对 Camus 进行实验，可以从我的 GitHub 项目中下载修补过的代码版本。或者，您可以修补 Maven 构建文件。

##### 使用我的修补过的 GitHub 项目

从 GitHub 下载我克隆并修补过的 Camus 版本，并像 Hadoop 1 版本一样构建它：

```
$ wget https://github.com/alexholmes/camus/archive/camus-kafka-0.8.zip
$ unzip camus-kafka-0.8.zip
$ cd camus-camus-kafka-0.8
$ mvn clean package
```

##### 修补 Maven 构建文件

如果您想修补原始的 Camus 文件，可以通过查看我自己的克隆中应用的补丁来完成：[`mng.bz/Q8GV`](https://mng.bz/Q8GV)。

### A.10\. Avro

Avro 是一个提供压缩、模式演进和代码生成等功能的数据序列化系统。它可以看作是 SequenceFile 的一个更复杂的版本，具有模式演进等附加功能。

第三章 包含了关于如何在 MapReduce 以及与基本输入/输出流中使用 Avro 的详细信息。

#### 获取更多信息

表 A.10 列出了一些有用资源，以帮助您更熟悉 Avro。

##### 表 A.10\. 有用资源

| 资源 | URL |
| --- | --- |
| Avro 项目页面 | [`avro.apache.org/`](http://avro.apache.org/) |
| Avro 问题跟踪页面 | [`issues.apache.org/jira/browse/AVRO`](https://issues.apache.org/jira/browse/AVRO) |
| Cloudera 关于 Avro 的博客 | [`blog.cloudera.com/blog/2011/12/apache-avro-at-richrelevance/`](http://blog.cloudera.com/blog/2011/12/apache-avro-at-richrelevance/) |
| CDH 使用 Avro 的页面 | [`www.cloudera.com/content/cloudera-content/cloudera-docs/CDH5/5.0/CDH5-Installation-Guide/cdh5ig_avro_usage.html`](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH5/5.0/CDH5-Installation-Guide/cdh5ig_avro_usage.html) |

#### 安装

Avro 是一个完整的 Apache 项目，因此您可以从 Apache 项目页面上的下载链接下载二进制文件。

### A.11\. Apache Thrift

Apache Thrift 实质上是 Facebook 的 Protocol Buffers 版本。它提供了非常相似的数据序列化和 RPC 功能。在这本书中，我使用它与 Elephant Bird 一起支持 MapReduce 中的 Thrift。Elephant Bird 目前支持 Thrift 版本 0.7。

#### 获取更多信息

Thrift 文档不足，这一点在项目页面上得到了证实。表 A.11 列出了一些有用的资源，以帮助您更熟悉 Thrift。

##### 表 A.11\. 有用资源

| 资源 | 网址 |
| --- | --- |
| Thrift 项目页面 | [`thrift.apache.org/`](http://thrift.apache.org/) |
| 包含 Thrift 教程的博客文章 | [`bit.ly/vXpZ0z`](http://bit.ly/vXpZ0z) |

#### 构建 Thrift 0.7

要构建 Thrift，请下载 0.7 版本的 tarball 并提取内容。您可能需要安装一些 Thrift 依赖项：

```
$ sudo yum  install automake libtool flex bison pkgconfig gcc-c++ \
  boost-devel libevent-devel zlib-devel python-devel \
  ruby-devel php53.x86_64 php53-devel.x86_64 openssl-devel
```

构建并安装原生和 Java/Python 库及二进制文件：

```
$ ./configure
$ make
$ make check
$ sudo make install
```

构建 Java 库。此步骤需要安装 Ant，相关说明可在 Apache Ant 手册中找到，网址为 [`ant.apache.org/manual/index.html`](http://ant.apache.org/manual/index.html)：

```
$ cd lib/java
$ ant
```

将 Java JAR 文件复制到 Hadoop 的 lib 目录。以下说明适用于 CDH：

```
# replace the following path with your actual
# Hadoop installation directory
#
# the following is the CDH Hadoop home dir
#
export HADOOP_HOME=/usr/lib/hadoop

$ cp lib/java/libthrift.jar $HADOOP_HOME/lib/
```

### A.12\. Protocol Buffers

Protocol Buffers 是 Google 的数据序列化和远程过程调用（RPC）库，在 Google 中被广泛使用。在这本书中，我们将与 Elephant Bird 和 Rhipe 一起使用它。Elephant Bird 需要 Protocol Buffers 的 2.3.0 版本（与其他版本不兼容），而 Rhipe 只与 Protocol Buffers 版本 2.4.0 及更高版本兼容。

#### 获取更多信息

表 A.12 列出了一些有用的资源，以帮助您更熟悉 Protocol Buffers。

##### 表 A.12\. 有用资源

| 资源 | 网址 |
| --- | --- |
| Protocol Buffers 项目页面 | [`code.google.com/p/protobuf/`](http://code.google.com/p/protobuf/) |
| Protocol Buffers 开发者指南 | [`developers.google.com/protocol-buffers/docs/overview?csw=1`](https://developers.google.com/protocol-buffers/docs/overview?csw=1) |
| 协议缓冲区下载页面，包含 2.3.0 版本的链接（与 Elephant Bird 一起使用所需） | [`code.google.com/p/protobuf/downloads/list`](http://code.google.com/p/protobuf/downloads/list) |

#### 构建 Protocol Buffers

要构建 Protocol Buffers，从 [`code.google.com/p/protobuf/downloads`](http://code.google.com/p/protobuf/downloads) 下载 2.3 或 2.4 版本的源 tarball（2.3 版本用于 Elephant Bird，2.4 版本用于 Rhipe）并提取内容。

您需要一个 C++ 编译器，可以在 64 位 RHEL 系统上使用以下命令安装：

```
sudo yum install gcc-c++.x86_64
```

构建和安装原生库和二进制文件：

```
$ cd protobuf-<version>/
$ ./configure
$ make
$ make check
$ sudo make install
```

构建 Java 库：

```
$ cd java
$ mvn package install
```

将 Java JAR 文件复制到 Hadoop 的 lib 目录中。以下说明适用于 CDH：

```
# replace the following path with your actual
# Hadoop installation directory
#
# the following is the CDH Hadoop home dir
#
export HADOOP_HOME=/usr/lib/hadoop

$ cp target/protobuf-java-2.3.0.jar $HADOOP_HOME/lib/
```

### A.13. Snappy

Snappy 是由 Google 开发的原生压缩编解码器，提供快速的压缩和解压缩时间。它不能分割（与 LZOP 压缩相反）。在本书的代码示例中，由于不需要可分割的压缩，我们将使用 Snappy，因为它的时间效率更高。

Snappy 自 1.0.2 和 2 版本以来已集成到 Apache Hadoop 发行版中。

#### 获取更多信息

表 A.13 列出了一些有用的资源，可以帮助您更熟悉 Snappy。

##### 表 A.13. 有用资源

| 资源 | URL |
| --- | --- |
| Google 的 Snappy 项目页面 | [`code.google.com/p/snappy/`](http://code.google.com/p/snappy/) |
| Snappy 与 Hadoop 的集成 | [`code.google.com/p/hadoop-snappy/`](http://code.google.com/p/hadoop-snappy/) |

### A.14. LZOP

LZOP 是一种压缩编解码器，可用于在 MapReduce 中支持可分割压缩。第四章有一个专门介绍如何使用 LZOP 的部分。在本节中，我们将介绍如何构建和设置您的集群以使用 LZOP。

#### 获取更多信息

表 A.14 展示了一个有用的资源，可以帮助您更熟悉 LZOP。

##### 表 A.14. 有用资源

| 资源 | URL |
| --- | --- |
| 由 Twitter 维护的 Hadoop LZO 项目 | [`github.com/twitter/hadoop-lzo`](https://github.com/twitter/hadoop-lzo) |

#### 构建 LZOP

以下步骤将指导您配置 LZOP 压缩。在您这样做之前，有一些事情需要考虑：

+   强烈建议您在部署生产环境中使用的相同硬件上构建库。

+   所有安装和配置步骤都需要在所有将使用 LZOP 的客户端主机以及您集群中的所有 DataNode 上执行。

+   这些步骤适用于 Apache Hadoop 发行版。如果您使用的是其他发行版，请参阅特定发行版的说明。

Twitter 的 LZO 项目页面提供了有关如何下载依赖项和构建项目的说明。请遵循项目主页上的“构建和配置”部分。

##### 配置 Hadoop

您需要配置 Hadoop 核心来使它了解您的新压缩编解码器。将以下行添加到您的 core-site.xml 中。确保删除换行符和空格，以便在逗号之间没有空白字符：

```
<property>
  <name>mapred.compress.map.output</name>
  <value>true</value>
</property>
<property>
  <name>mapred.map.output.compression.codec</name>
  <value>com.hadoop.compression.lzo.LzoCodec</value>
</property>
<property>
  <name>io.compression.codecs</name>
  <value>org.apache.hadoop.io.compress.GzipCodec,
  org.apache.hadoop.io.compress.DefaultCodec,
  org.apache.hadoop.io.compress.BZip2Codec,
  com.hadoop.compression.lzo.LzoCodec,
  com.hadoop.compression.lzo.LzopCodec,

  org.apache.hadoop.io.compress.SnappyCodec</value>
</property>
<property>
  <name>io.compression.codec.lzo.class</name>
  <value>com.hadoop.compression.lzo.LzoCodec</value>
</property>
```

`io.compression.codecs` 的值假设您已经安装了 Snappy 压缩编解码器。如果没有，请从值中删除 `org.apache.hadoop.io.compress.SnappyCodec`。

### A.15\. Elephant Bird

Elephant Bird 是一个提供用于处理 LZOP 压缩数据的实用程序的项目。它还提供了一个容器格式，支持在 MapReduce 中使用 Protocol Buffers 和 Thrift。

#### 获取更多信息

表 A.15 展示了一个有用的资源，以帮助您更熟悉 Elephant Bird。

##### 表 A.15\. 有用资源

| 资源 | URL |
| --- | --- |
| Elephant Bird 项目页面 | [`github.com/kevinweil/elephant-bird`](https://github.com/kevinweil/elephant-bird) |

在撰写本文时，由于使用了不兼容版本的 Protocol Buffers，Elephant Bird（4.4）的当前版本与 Hadoop 2 不兼容。为了使 Elephant Bird 在本书中工作，我不得不从 trunk 构建一个与 Hadoop 2 兼容的项目版本（4.5 版本发布时也将如此）。

### A.16\. Hive

Hive 是 Hadoop 之上的 SQL 接口。

#### 获取更多信息

表 A.16 列出了一些有用的资源，以帮助您更熟悉 Hive。

##### 表 A.16\. 有用资源

| 资源 | URL |
| --- | --- |
| Hive 项目页面 | [`hive.apache.org/`](http://hive.apache.org/) |
| 入门 | [`cwiki.apache.org/confluence/display/Hive/GettingStarted`](https://cwiki.apache.org/confluence/display/Hive/GettingStarted) |

#### 安装

按照 Hive 入门指南中的安装说明进行操作。

### A.17\. R

R 是一个用于统计编程和图形的开源工具。

#### 获取更多信息

表 A.17 列出了一些有用的资源，以帮助您更熟悉 R。

##### 表 A.17\. 有用资源

| 资源 | URL |
| --- | --- |
| R 项目页面 | [`www.r-project.org/`](http://www.r-project.org/) |
| R 函数搜索引擎 | [`rseek.org/`](http://rseek.org/) |

#### 在基于 Red Hat 的系统上安装

从 Yum 安装 R 使事情变得简单：它将确定 RPM 依赖关系并为您安装它们。

访问 [`www.r-project.org/`](http://www.r-project.org/)，点击 CRAN，选择一个靠近您的下载区域，选择 Red Hat，并选择适合您系统的版本和架构。替换以下代码中的 `baseurl` URL 并执行命令以将 R 镜像仓库添加到您的 Yum 配置中：

```
$ sudo -s
$ cat << EOF > /etc/yum.repos.d/r.repo
# R-Statistical Computing
[R]
name=R-Statistics
baseurl=http://cran.mirrors.hoobly.com/bin/linux/redhat/el5/x86_64/
enabled=1
gpgcheck=0
EOF
```

在 64 位系统上可以使用简单的 Yum 命令安装 R：

```
$ sudo yum install R.x86_64
```


##### Perl-File-Copy-Recursive RPM

在 CentOS 上，Yum 安装可能会失败，并抱怨缺少依赖项。在这种情况下，你可能需要手动安装 perl-File-Copy-Recursive RPM（对于 CentOS，你可以从 [`mng.bz/n4C2`](http://mng.bz/n4C2) 获取）。


#### 非 Red Hat 系统上的安装

访问 [`www.r-project.org/`](http://www.r-project.org/)，点击 CRAN，选择一个靠近你的下载区域，并选择适合你系统的相应二进制文件。

### A.18\. RHadoop

RHadoop 是由 Revolution Analytics 开发的一个开源工具，用于将 R 与 MapReduce 集成。

#### 获取更多信息

表 A.18 列出了一些有用的资源，帮助你更熟悉 RHadoop。

##### 表 A.18\. 有用资源

| 资源 | URL |
| --- | --- |
| RHadoop 项目页面 | [`github.com/RevolutionAnalytics/RHadoop/wiki`](https://github.com/RevolutionAnalytics/RHadoop/wiki) |
| RHadoop 下载和先决条件 | [`github.com/RevolutionAnalytics/RHadoop/wiki/Downloads`](https://github.com/RevolutionAnalytics/RHadoop/wiki/Downloads) |

#### rmr/rhdfs 安装

你的 Hadoop 集群中的每个节点都需要以下组件：

+   R（安装说明在 A.17 节 中）。

+   许多 RHadoop 和依赖包

RHadoop 需要你设置环境变量以指向 Hadoop 二进制文件和 streaming JAR 文件。最好将其存放在你的 .bash_profile（或等效文件）中。

```
$ export HADOOP_CMD=/usr/local/hadoop/bin/hadoop
$ export HADOOP_STREAMING=${HADOOP_HOME}/share/hadoop/tools/lib/
hadoop-streaming-<version>.jar
```

我们将重点关注 `rmr` 和 `rhdfs` RHadoop 包，它们提供了与 R 的 MapReduce 和 HDFS 集成。点击 [`github.com/RevolutionAnalytics/RHadoop/wiki/Downloads`](https://github.com/RevolutionAnalytics/RHadoop/wiki/Downloads) 上的 `rmr` 和 `rhdfs` 下载链接。然后执行以下命令：

```
$ sudo -s
yum install -y libcurl-devel java-1.7.0-openjdk-devel
$ export HADOOP_CMD=/usr/bin/hadoop
$ R CMD javareconf
$ R
> install.packages( c('rJava'),
    repos='http://cran.revolutionanalytics.com')
> install.packages( c('RJSONIO', 'itertools', 'digest', 'Rcpp','httr',
    'functional','devtools', 'reshape2', 'plyr', 'caTools'),
    repos='http://cran.revolutionanalytics.com')

$ R CMD INSTALL  /media/psf/Home/Downloads/rhdfs_1.0.8.tar.gz
$ R CMD INSTALL  /media/psf/Home/Downloads/rmr2_3.1.1.tar.gz
$ R CMD INSTALL rmr_<version>.tar.gz
$ R CMD INSTALL rhdfs_<version>.tar.gz
```

如果你安装 rJava 时遇到错误，你可能需要在运行 rJava 安装之前设置 `JAVA_HOME` 并重新配置 R：

```
$ sudo -s
$ export JAVA_HOME=/usr/java/latest
$ R CMD javareconf
$ R
> install.packages("rJava")
```

通过运行以下命令来测试 `rmr` 包是否正确安装——如果没有生成错误消息，这意味着你已经成功安装了 RHadoop 包。

```
$ R
> library(rmr2)
```

### A.19\. Mahout

Mahout 是一个预测分析项目，它为其一些算法提供了 JVM 内部和 MapReduce 实现。

#### 获取更多信息

表 A.19 列出了一些有用的资源，帮助你更熟悉 Mahout。

##### 表 A.19\. 有用资源

| 资源 | URL |
| --- | --- |
| Mahout 项目页面 | [`mahout.apache.org/`](http://mahout.apache.org/) |
| Mahout 下载 | [`cwiki.apache.org/confluence/display/MAHOUT/Downloads`](https://cwiki.apache.org/confluence/display/MAHOUT/Downloads) |

#### 安装

Mahout 应该安装在可以访问你的 Hadoop 集群的节点上。Mahout 是一个客户端库，不需要在 Hadoop 集群上安装。

##### 构建 Mahout 发行版

要使 Mahout 与 Hadoop 2 一起工作，我不得不检出代码，修改构建文件，然后构建一个发行版。第一步是检出代码：

```
$ git clone https://github.com/apache/mahout.git
$ cd mahout
```

接下来，你需要修改 pom.xml 文件并从文件中删除以下部分：

```
<plugin>
<inherited>true</inherited>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-gpg-plugin</artifactId>
<version>1.4</version>

<executions>
  <execution>
    <goals>
      <goal>sign</goal>
    </goals>
  </execution>
</executions>
</plugin>
```

最后，构建一个发行版：

```
$ mvn -Dhadoop2.version=2.2.0 -DskipTests -Prelease
```

这将在 distribution/target/mahout-distribution-1.0-SNAPSHOT.tar.gz 位置生成一个 tarball，你可以使用下一节中的说明进行安装。

##### 安装 Mahout

Mahout 以 tarball 的形式打包。以下说明适用于大多数 Linux 操作系统。

如果你正在安装官方的 Mahout 版本，请点击 Mahout 下载页面上的“官方版本”链接并选择当前版本。如果 Mahout 1 尚未发布，而你想要与 Hadoop 2 一起使用 Mahout，请按照上一节中的说明生成 tarball。

使用以下说明安装 Mahout：

```
$ cd /usr/local
$ sudo tar -xzf <path-to-mahout-tarball>

$ sudo ln -s mahout-distribution-<version> mahout

$ sudo chown -R <user>:<group> /usr/local/mahout*
```

为了方便，值得更新你的 ~/.bash_profile，将 `MAHOUT_HOME` 环境变量导出到你的安装目录。以下命令展示了如何在命令行上执行此操作（相同的命令可以复制到你的 bash 配置文件中）：

```
$ export MAHOUT_HOME=/usr/local/mahout
```
