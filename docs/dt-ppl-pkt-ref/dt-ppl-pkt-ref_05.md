# 第五章：数据摄取：加载数据

在第四章中，您从所需的源系统提取了数据。现在是通过将数据加载到 Redshift 数据仓库来完成数据摄取的时候了。如何加载取决于数据提取的输出。在本节中，我将描述如何将提取出的数据加载到 CSV 文件中，其中值对应于表中的每列，以及包含 CDC 格式数据的提取输出。

# 配置 Amazon Redshift 数据仓库作为目的地

如果您正在使用 Amazon Redshift 作为数据仓库，那么在提取数据后使用 S3 加载数据的集成就非常简单。第一步是为加载数据创建 IAM 角色（如果还没有）。

###### 注意

要了解如何设置 Amazon Redshift 集群，请查看最新的[文档和定价信息，包括免费试用](https://oreil.ly/YSaxa)。

要创建角色，请按照以下说明操作，或查看[AWS 文档](https://oreil.ly/QEJzH)获取最新详情：

1.  在 AWS 控制台的服务菜单（或顶部导航栏）下，导航到 IAM。

1.  在左侧导航菜单中，选择“角色”，然后点击“创建角色”按钮。

1.  将显示一个 AWS 服务列表供您选择。找到并选择 Redshift。

1.  在“选择您的用例”下，选择 Redshift – Customizable。

1.  在下一页（附加权限策略）上，搜索并选择 AmazonS3ReadOnlyAccess，然后点击“下一步”。

1.  给您的角色命名（例如，“RedshiftLoadRole”），然后点击“创建角色”。

1.  点击新角色的名称，并复制*角色 Amazon 资源名称*（ARN），以便您可以在本章后面使用。您还可以在 IAM 控制台的角色属性下找到它。ARN 的格式如下：`arn:aws:iam::*<aws-account-id>*:role/*<role-name>*`。

现在，您可以将刚创建的 IAM 角色与您的 Redshift 集群关联起来。要执行此操作，请按照以下步骤操作，或查看[Redshift 文档](https://oreil.ly/uHLEk)获取更多详细信息。

###### 注意

您的集群将花费一到两分钟来应用这些更改，但在此期间仍然可以访问它。

1.  返回 AWS 服务菜单，然后转到 Amazon Redshift。

1.  在导航菜单中，选择“集群”，然后选择要加载数据的集群。

1.  在操作下，点击“管理 IAM 角色”。

1.  加载后会显示“管理 IAM 角色”页面，您可以在“可用角色”下拉菜单中选择您的角色。然后点击“添加 IAM 角色”。

1.  点击完成。

最后，在您创建的 pipeline.conf 文件中添加另一部分，包括您的 Redshift 凭据和刚创建的 IAM 角色名称。您可以在 AWS Redshift 控制台页面上找到 Redshift 集群连接信息：

```
[aws_creds]
database = my_warehouse
username = pipeline_user
password = weifj4tji4j
host = my_example.4754875843.us-east-1.redshift.amazonaws.com
port = 5439
iam_role = RedshiftLoadRole
```

# 将数据加载到 Redshift 数据仓库

将从 S3 存储为 CSV 文件中每列对应于 Redshift 表中每列的值提取和存储的数据加载到 Redshift 相对简单。 这种格式的数据最常见，是从诸如 MySQL 或 MongoDB 数据库之类的源中提取数据的结果。 要加载到目标 Redshift 表中的每个 CSV 文件中的行对应于要加载到目标表中的记录，CSV 中的每列对应于目标表中的列。 如果您从 MySQL binlog 或其他 CDC 日志中提取了事件，请参阅下一节有关加载说明。

将数据从 S3 加载到 Redshift 的最有效方法是使用`COPY`命令。 `COPY`可以作为 SQL 语句在查询 Redshift 集群的 SQL 客户端中执行，或者在使用 Boto3 库的 Python 脚本中执行。 `COPY`将加载的数据追加到目标表的现有行中。

`COPY`命令的语法如下。 所有方括号([])项都是可选的：

```
COPY table_name
[ column_list ]
FROM source_file
authorization
[ [ FORMAT ] [ AS ] data_format ]
[ parameter [ argument ] [, .. ] ]
```

###### 注意

您可以在[AWS 文档](https://oreil.ly/0uWo8)中了解更多有关附加选项和`COPY`命令的一般信息。

在最简单的形式中，使用 IAM 角色授权如第四章中指定的，并且从 SQL 客户端运行时，S3 存储桶中的文件看起来像这样：

```
COPY my_schema.my_table
FROM 's3://bucket-name/file.csv’
iam_role ‘<my-arn>’;
```

正如您从“配置 Amazon Redshift Warehouse 作为目标”中回想起的那样，ARN 的格式如下：

```
arn:aws:iam::<aws-account-id>:role/<role-name>
```

如果您将角色命名为`RedshiftLoadRole`，则`COPY`命令语法如下所示。 请注意，ARN 中的数字值特定于您的 AWS 帐户：

```
COPY my_schema.my_table
FROM 's3://bucket-name/file.csv’
iam_role 'arn:aws:iam::222:role/RedshiftLoadRole’;
```

当执行时，*file.csv*的内容将追加到 Redshift 集群中`my_schema`模式下名为`my_table`的表中。

默认情况下，`COPY`命令将数据插入到目标表的列中，顺序与输入文件中字段的顺序相同。 换句话说，除非另有指定，否则您在此示例中加载的 CSV 中的字段顺序应与 Redshift 目标表中列的顺序匹配。 如果您想指定列顺序，可以通过按与输入文件匹配的顺序添加目标列的名称来执行此操作，如下所示：

```
COPY my_schema.my_table (column_1, column_2, ....)
FROM 's3://bucket-name/file.csv'
iam_role 'arn:aws:iam::222:role/RedshiftLoadRole';
```

也可以使用 Boto3 库在 Python 脚本中实现`COPY`命令。 实际上，按照第四章中数据提取示例的模板，通过 Python 加载数据可以创建更标准化的数据流水线。

要与本章早期配置的 Redshift 集群进行交互，需要安装`psycopg2`库：

```
(env) $ pip install psycopg2
```

现在可以开始编写 Python 脚本。 创建一个名为*copy_to_redshift.py*的新文件，并添加以下三个代码块。

第一步是导入`boto3`以与 S3 存储桶交互，`psycopg2`以在 Redshift 集群上运行`COPY`命令，以及`configparser`库以读取*pipeline.conf*文件：

```
import boto3
import configparser
import psycopg2
```

接下来，使用`psycopg2.connect`函数和存储在*pipeline.conf*文件中的凭据连接到 Redshift 集群：

```
parser = configparser.ConfigParser()
parser.read("pipeline.conf")
dbname = parser.get("aws_creds", "database")
user = parser.get("aws_creds", "username")
password = parser.get("aws_creds", "password")
host = parser.get("aws_creds", "host")
port = parser.get("aws_creds", "port")

# connect to the redshift cluster
rs_conn = psycopg2.connect(
    "dbname=" + dbname
    + " user=" + user
    + " password=" + password
    + " host=" + host
    + " port=" + port)
```

现在，你可以使用`psycopg2`的`Cursor`对象执行`COPY`命令。运行与本节中手动运行的`COPY`命令相同的`COPY`命令，但是不要直接编码 AWS 账户 ID 和 IAM 角色名称，而是从*pipeline.conf*文件中加载这些值：

```
# load the account_id and iam_role from the
# conf files
parser = configparser.ConfigParser()
parser.read("pipeline.conf")
account_id = parser.get("aws_boto_credentials",
              "account_id")
iam_role = parser.get("aws_creds", "iam_role")
bucket_name = parser.get("aws_boto_credentials",
              "bucket_name")

# run the COPY command to load the file into Redshift
file_path = ("s3://"
    + bucket_name
    + "/order_extract.csv")
role_string = ("arn:aws:iam::"
    + account_id
    + ":role/" + iam_role)

sql = "COPY public.Orders"
sql = sql + " from %s "
sql = sql + " iam_role %s;"

# create a cursor object and execute the COPY
cur = rs_conn.cursor()
cur.execute(sql,(file_path, role_string))

# close the cursor and commit the transaction
cur.close()
rs_conn.commit()

# close the connection
rs_conn.close()
```

在运行脚本之前，如果目标表还不存在，你需要先创建它。在本例中，我正在加载从“完整或增量 MySQL 表抽取”中提取出的数据，该数据保存在*order_extract.csv*文件中。当然，你可以加载任何你想要的数据。只需确保目标表的结构匹配即可。要在你的集群上创建目标表，请通过 Redshift 查询编辑器或其他连接到你的集群的应用程序运行以下 SQL：

```
CREATE TABLE public.Orders (
  OrderId int,
  OrderStatus varchar(30),
  LastUpdated timestamp
);
```

最后，按以下步骤运行脚本：

```
(env) $ python copy_to_redshift.py
```

## 增量加载与完整加载对比

在前面的代码示例中，`COPY`命令从提取的 CSV 文件直接加载数据到 Redshift 集群中的表中。如果 CSV 文件中的数据来自于不可变源的增量抽取（例如不可变事件数据或其他“仅插入”数据集），那么无需进行其他操作。然而，如果 CSV 文件中的数据包含更新的记录以及插入或源表的全部内容，则需要做更多工作，或者至少需要考虑一些因素。

以“完整或增量 MySQL 表抽取”中的`Orders`表为例。这意味着你从 CSV 文件中加载的数据可能是从源 MySQL 表中完整或增量提取出来的。

如果数据是完整提取的，那么在运行`COPY`操作之前，需要对 Redshift 中的目标表进行截断（使用 TRUNCATE）。更新后的代码片段如下所示：

```
import boto3
import configparser
import psycopg2

parser = configparser.ConfigParser()
parser.read("pipeline.conf")
dbname = parser.get("aws_creds", "database")
user = parser.get("aws_creds", "username")
password = parser.get("aws_creds", "password")
host = parser.get("aws_creds", "host")
port = parser.get("aws_creds", "port")

# connect to the redshift cluster
rs_conn = psycopg2.connect(
    "dbname=" + dbname
    + " user=" + user
    + " password=" + password
    + " host=" + host
    + " port=" + port)

parser = configparser.ConfigParser()
parser.read("pipeline.conf")
account_id = parser.get("aws_boto_credentials",
                  "account_id")
iam_role = parser.get("aws_creds", "iam_role")
bucket_name = parser.get("aws_boto_credentials",
                  "bucket_name")

# truncate the destination table
sql = "TRUNCATE public.Orders;"
cur = rs_conn.cursor()
cur.execute(sql)

cur.close()
rs_conn.commit()

# run the COPY command to load the file into Redshift
file_path = ("s3://"
    + bucket_name
    + "/order_extract.csv")
role_string = ("arn:aws:iam::"
    + account_id
    + ":role/" + iam_role)

sql = "COPY public.Orders"
sql = sql + " from %s "
sql = sql + " iam_role %s;"

# create a cursor object and execute the COPY command
cur = rs_conn.cursor()
cur.execute(sql,(file_path, role_string))

# close the cursor and commit the transaction
cur.close()
rs_conn.commit()

# close the connection
rs_conn.close()
```

如果数据是增量提取的，则不应该对目标表进行截断。如果截断了，那么最后一次运行抽取作业后剩下的只是更新的记录。有几种方法可以处理以这种方式提取的数据，但最好的方法是保持简单。

在这种情况下，你可以简单地使用`COPY`命令加载数据（不使用`TRUNCATE`！），并依靠时间戳来确定记录的最新状态或查看历史记录。例如，假设源表中的记录已修改并因此存在于正在加载的 CSV 文件中。加载完成后，你将在 Redshift 目标表中看到类似于表 5-1 的内容。

表 5-1\. Redshift 中的订单表

| OrderId | 订单状态 | 最后更新时间 |
| --- | --- | --- |
| 1 | 已备货 | 2020-06-01 12:00:00 |
| 1 | 已发货 | 2020-06-09 12:00:25 |

正如你在表 5-1 中所看到的，ID 值为 1 的订单在表中出现了两次。第一条记录存在于最新加载之前，并且第二条刚从 CSV 文件中加载。第一条记录是由于在 2020-06-01 更新了记录时订单处于`Backordered`状态时创建的。它在 2020-06-09 再次更新时`Shipped`，并包含在你最后加载的 CSV 文件中。

从历史记录保存的角度来看，在目标表中拥有这两条记录是理想的。在流水线的转换阶段后，分析师可以根据特定分析的需求选择使用其中一条或两条记录。也许他们想知道订单在缺货状态下的持续时间。他们需要这两条记录。如果他们想知道订单的当前状态，他们也可以得到这些信息。

虽然在目标表中为相同的`OrderId`拥有多条记录可能会让人感到不舒服，但在这种情况下，这样做是正确的选择！数据摄取的目标是专注于提取和加载数据。如何处理数据是流水线转换阶段的任务，在第六章 中有所探讨。

## 从 CDC 日志中提取加载数据

如果你的数据是通过 CDC 方法提取的，那么还有另一个考虑因素。尽管这与增量加载的数据类似，你不仅可以访问插入和更新的记录，还可以访问删除的记录。

以从第四章 中提取的 MySQL 二进制日志为例。回想一下代码示例的输出是一个名为*orders_extract.csv*的 CSV 文件，上传到了 S3 存储桶。其内容如下所示：

```
insert|1|Backordered|2020-06-01 12:00:00
update|1|Shipped|2020-06-09 12:00:25
```

就像本节前面的增量加载示例一样，`OrderId` 1 有两条记录。当加载到数据仓库时，数据看起来就像表 5-1 中的样子。然而，与之前的示例不同，*orders_extract.csv* 包含了文件中记录事件的列。在这个例子中，可以是`insert`或`update`。如果这些是唯一的两种事件类型，你可以忽略事件字段，并最终得到在 Redshift 中看起来像表 5-1 的表格。从那里开始，分析师在后续流水线中构建数据模型时将可以访问这两条记录。然而，请考虑*orders_extract.csv* 的另一个版本，其中包含了一行额外的内容：

```
insert|1|Backordered|2020-06-01 12:00:00
update|1|Shipped|2020-06-09 12:00:25
delete|1|Shipped|2020-06-10 9:05:12
```

第三行显示订单记录在更新后的第二天被删除了。在完全提取中，该记录将完全消失，并且增量提取不会捕获到删除操作（请参阅“从 MySQL 数据库提取数据”以获取更详细的解释）。然而，使用 CDC，删除事件被捕获并包含在 CSV 文件中。

为了容纳已删除的记录，需要在 Redshift 仓库的目标表中添加一个列来存储事件类型。表 5-2 展示了 `Orders` 的扩展版本的外观。

表 5-2\. Redshift 中具有 EventType 的 Orders 表

| EventType | OrderId | OrderStatus | LastUpdated |
| --- | --- | --- | --- |
| insert | 1 | 已备货 | 2020-06-01 12:00:00 |
| update | 1 | 已发货 | 2020-06-09 12:00:25 |
| delete | 1 | 已发货 | 2020-06-10 9:05:12 |

数据管道中数据摄取的目标是有效地从源提取数据并加载到目标中。管道中的转换步骤是针对特定用例对数据建模的逻辑所在。第六章讨论了如何对通过 CDC 摄取加载的数据进行建模，例如本例。

# 将 Snowflake 仓库配置为目标

如果您将 Snowflake 作为数据仓库，有三种选项可配置从 Snowflake 实例访问 S3 存储桶的访问权限：

+   配置 Snowflake 存储集成

+   配置 AWS IAM 角色

+   配置 AWS IAM 用户

其中，推荐第一种，因为在稍后从 Snowflake 中与 S3 存储桶交互时，使用 Snowflake 存储集成是多么无缝。由于配置的具体步骤包括多个步骤，建议参考有关该主题的[最新 Snowflake 文档](https://oreil.ly/RCoMT)。

在配置的最后一步中，您将创建一个 *外部 stage*。外部 stage 是指向外部存储位置的对象，以便 Snowflake 可以访问它。您之前创建的 S3 存储桶将作为该位置。

在创建 stage 之前，最好在 Snowflake 中定义一个 `FILE FORMAT`，您既可以为 stage 引用，也可以稍后用于类似的文件格式。由于本章的示例创建了管道分隔的 CSV 文件，因此创建以下 `FILE FORMAT`：

```
CREATE or REPLACE FILE FORMAT pipe_csv_format
TYPE = 'csv'
FIELD_DELIMITER = '|';
```

当根据 Snowflake 文档的最后一步创建桶的 stage 时，语法将类似于：

```
USE SCHEMA my_db.my_schema;

CREATE STAGE my_s3_stage
  storage_integration = s3_int
  url = 's3://pipeline-bucket/'
  file_format = pipe_csv_format;
```

在 “将数据加载到 Snowflake 数据仓库” 中，您将使用 stage 将从 S3 存储桶中提取并存储的数据加载到 Snowflake 中。

最后，您需要向*pipeline.conf*文件中添加一个部分，其中包含 Snowflake 登录凭据。请注意，您指定的用户必须在您刚创建的阶段上具有`USAGE`权限。此外，`account_name`值必须根据您的云提供商和帐户所在地区进行格式化。例如，如果您的帐户名为`snowflake_acct1`，托管在 AWS 的美国东部（俄亥俄州）区域，则`account_name`值将是`snowflake_acct1.us-east-2.aws`。因为此值将用于使用 Python 通过`snowflake-connector-python`库连接到 Snowflake，您可以参考[库文档](https://oreil.ly/ijcHw)以获取有关确定`account_name`正确值的帮助。

以下是添加到*pipeline.conf*的部分：

```
[snowflake_creds]
username = snowflake_user
password = snowflake_password
account_name = snowflake_acct1.us-east-2.aws
```

# 将数据加载到 Snowflake 数据仓库

将数据加载到 Snowflake 与之前加载数据到 Redshift 的模式几乎相同。因此，我不会讨论处理全量、增量或 CDC 数据提取的具体细节。相反，我将描述从已提取文件加载数据的语法。

将数据加载到 Snowflake 的机制是`COPY INTO`命令。`COPY INTO`将一个或多个文件的内容加载到 Snowflake 仓库中的表中。您可以在[Snowflake 文档](https://oreil.ly/E3KG5)中了解有关该命令的高级用法和选项。

###### 注意

Snowflake 还有一个名为*Snowpipe*的数据集成服务，允许从文件加载数据，这些文件一旦在 Snowflake 阶段（如本节示例中使用的阶段）中可用即可。您可以使用 Snowpipe 连续加载数据，而不是通过`COPY INTO`命令安排批量加载。

在第四章中的每个提取示例都将 CSV 文件写入了一个 S3 存储桶。在“将 Snowflake 仓库配置为目的地”中，您创建了一个名为`my_s3_stage`的 Snowflake 阶段，该阶段链接到该存储桶。现在，使用`COPY INTO`命令，您可以将文件加载到 Snowflake 表中，如下所示：

```
COPY INTO destination_table
  FROM @my_s3_stage/extract_file.csv;
```

还可以一次加载多个文件到表中。在某些情况下，由于数据量或自上次加载以来的多个提取作业运行的结果，数据被提取为多个文件。如果文件具有一致的命名模式（而且应该有！），您可以使用`pattern`参数加载它们所有：

```
COPY INTO destination_table
  FROM @my_s3_stage
  pattern='.*extract.*.csv';
```

###### 注意

文件的格式在创建 Snowflake 阶段时已设置（管道分隔的 CSV 文件），因此不需要在`COPY INTO`命令语法中指定它。

现在您已经了解了`COPY INTO`命令的工作原理，现在是时候编写一个简短的 Python 脚本，以便安排并执行它，从而自动化管道中的加载。详情请参阅第七章，了解有关此及其他管道编排技术的更多详细信息。

首先，您需要安装一个 Python 库来连接到您的 Snowflake 实例。您可以使用`pip`完成此操作：

```
(env) $ pip install snowflake-connector-python
```

现在，您可以编写一个简单的 Python 脚本连接到您的 Snowflake 实例，并使用`COPY INTO`将 CSV 文件的内容加载到目标表中：

```
import snowflake.connector
import configparser

parser = configparser.ConfigParser()
parser.read("pipeline.conf")
username = parser.get("snowflake_creds",
            "username")
password =  parser.get("snowflake_creds",
            "password")
account_name = parser.get("snowflake_creds",
            "account_name")

snow_conn = snowflake.connector.connect(
    user = username,
    password = password,
    account = account_name
    )

sql = """COPY INTO destination_table
 FROM @my_s3_stage
 pattern='.*extract.*.csv';"""

cur = snow_conn.cursor()
cur.execute(sql)
cur.close()
```

# 使用文件存储作为数据湖

有时，从 S3 存储桶（或其他云存储）中提取数据而不加载到数据仓库中是有意义的。以这种方式以结构化或半结构化形式存储的数据通常称为*数据湖*。

与数据仓库不同，数据湖以原始且有时非结构化的形式存储数据，可以存储多种格式的数据。它的存储成本较低，但并不像数据仓库中的结构化数据那样优化用于查询。

然而，近年来出现了一些工具，使得对数据湖中的数据进行查询变得更加可访问，通常对于熟悉 SQL 的用户来说也更加透明。例如，Amazon Athena 是一个 AWS 服务，允许用户使用 SQL 查询存储在 S3 中的数据。Amazon Redshift Spectrum 是一种服务，允许 Redshift 访问 S3 中的数据作为*外部表*，并在与 Redshift 仓库中的表一起查询时引用它。其他云提供商和产品也具有类似的功能。

在何时应考虑使用这种方法而不是结构化和加载数据到您的仓库中？有几种情况显得特别突出。

在基于云存储的数据湖中存储大量数据比在仓库中存储便宜（对于使用与 Snowflake 数据仓库相同存储的 Snowflake 数据湖不适用）。此外，由于它是非结构化或半结构化数据（没有预定义的模式），因此更改存储的数据类型或属性要比修改仓库模式容易得多。JSON 文档是您可能在数据湖中遇到的半结构化数据类型的示例。如果数据结构经常变化，您可能会考虑将其暂时存储在数据湖中。

在数据科学或机器学习项目的探索阶段，数据科学家或机器学习工程师可能尚不清楚他们需要数据呈现的确切“形状”。通过以原始形式访问湖中的数据，他们可以探索数据，并确定需要利用数据的哪些属性。一旦确定，您可以确定是否有意义将数据加载到仓库中的表中，并获得随之而来的查询优化。

实际上，许多组织在其数据基础设施中既有数据湖又有数据仓库。随着时间的推移，这两者已经成为互补而非竞争的解决方案。

# 开源框架

正如您现在已经注意到的，每个数据摄取中（提取和加载步骤都有）都存在重复的步骤。因此，近年来出现了许多框架，提供核心功能和与常见数据源和目标的连接。正如本节所讨论的，其中一些是开源的，而下一节则概述了一些流行的商业数据摄取产品。

一个流行的开源框架称为[Singer](https://www.singer.io)。Singer 用 Python 编写，使用*taps*从源提取数据，并以 JSON 流方式传输到*target*。例如，如果您想从 MySQL 数据库提取数据并将其加载到 Google BigQuery 数据仓库中，您将使用 MySQL tap 和 BigQuery target。

就像本章中的代码示例一样，使用 Singer 仍然需要使用单独的编排框架来调度和协调数据摄取（有关更多信息，请参见第七章）。然而，无论您使用 Singer 还是其他框架，通过一个良好构建的基础，可以快速启动和运行。

作为一个开源项目，有大量可用的 taps 和 targets（请参阅表 5-3 中一些最受欢迎的）。您还可以向项目贡献自己的内容。Singer 有着良好的文档和活跃的[Slack](https://oreil.ly/tBQs0)和[GitHub](https://oreil.ly/nLJgF)社区。

表 5-3\. 流行的 singer taps 和 targets

| Taps | Targets |
| --- | --- |
| Google Analytics | CSV |
| Jira | Google BigQuery |
| MySQL | PostgreSQL |
| PostgreSQL | Amazon Redshift |
| Salesforce | Snowflake |

# 商业替代方案

有几种商业云托管产品可以实现许多常见的数据摄取，而无需编写一行代码。它们还具有内置的调度和作业编排功能。当然，这一切都是有代价的。

两个最受欢迎的商业数据摄取工具是[Stitch](https://www.stitchdata.com)和[Fivetran](https://fivetran.com)。两者都是完全基于 Web 的，数据工程师以及数据团队中的其他数据专业人员都可以访问。它们提供数百个预构建的“连接器”，用于常见数据源，如 Salesforce、HubSpot、Google Analytics、GitHub 等。您还可以从 MySQL、Postgres 和其他数据库中获取数据。还内置了对 Amazon Redshift、Snowflake 等数据仓库的支持。

如果您从支持的源中提取数据，将节省大量构建新数据摄取的时间。此外，正如第七章中详细概述的那样，调度和编排数据摄取并不是微不足道的任务。使用 Stitch 和 Fivetran，您可以在浏览器中构建、调度和对中断的摄取管道进行警报。

两个平台上选择的连接器还支持作业执行超时、重复数据处理、源系统架构更改等功能。如果你自行构建摄取，你需要自己考虑所有这些因素。

当然，这其中也存在一些权衡：

成本

Stitch 和 Fivetran 都采用基于数据量的定价模型。虽然它们在量化数据和每个定价层中包含的其他功能方面有所不同，但归根结底，你支付的费用取决于你摄取的数据量。如果你有多个高容量数据源需要摄取，费用将会增加。

锁定供应商

一旦你投资了某个供应商，未来要迁移到另一个工具或产品将需要大量工作。

定制化需要编码

如果你想从的源系统没有预构建的连接器，你就得自己写一些代码。对于 Stitch 来说，这意味着编写自定义的 Singer tap（参见前一节），而对于 Fivetran，则需要使用 AWS Lambda、Azure Function 或 Google Cloud Functions 编写云函数。如果你有许多自定义数据源，比如自定义构建的 REST API，你最终会不得不编写自定义代码，然后支付 Stitch 或 Fivetran 运行它。

安全和隐私

虽然这两款产品作为数据的透传器使用，并不会长时间存储数据，但它们仍然技术上可以访问你的源系统以及目标地点（通常是数据仓库或数据湖）。虽然 Fivetran 和 Stitch 都符合高安全标准，但一些组织由于风险容忍度、监管要求、潜在责任以及审查和批准新数据处理器的额外开销而不愿使用它们。

选择构建或购买对每个组织和使用案例来说都是复杂且独特的。值得注意的是，一些组织同时使用自定义代码和像 Fivetran 或 Stitch 这样的产品进行数据摄取。例如，编写自定义代码来处理一些在商业平台上运行成本高昂的高容量摄取可能更具成本效益，但使用 Stitch 或 Fivetran 进行具有预构建、供应商支持连接器的摄取也是值得的。

如果你选择了自定义和商业工具的混合使用，记住你需要考虑如何标准化日志记录、警报和依赖管理等事项。本书的后续章节将讨论这些主题，并触及跨多个平台管理流水线的挑战。
