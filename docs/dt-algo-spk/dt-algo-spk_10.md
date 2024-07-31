# 第七章：与外部数据源交互

在 Spark 中，为了运行任何算法，您需要从数据源读取输入数据，然后将您的算法应用为一组 PySpark 转换和操作（表达为 DAG），最后将所需的输出写入目标数据源。因此，为了编写性能良好的算法，了解从外部数据源读取和写入至关重要。

在前几章中，我们已经探讨了如何与 Spark 中的内置数据源（RDD 和 DataFrame）进行交互。在本章中，我们将专注于 Spark 如何与外部数据源进行接口交互。

如 图 7-1 所示，Spark 可以从广泛的外部存储系统（如 Linux 文件系统、Amazon S3、HDFS、Hive 表以及关系数据库（如 Oracle、MySQL 或 PostgreSQL））中读取数据，通过其数据源接口。本章将向您展示如何读取数据，并将其转换为 RDD 或 DataFrame 以进行进一步处理。我还将向您展示如何将 Spark 的数据写回到文件、Amazon S3 和 JDBC 兼容的数据库中。

![daws 0701](img/daws_0701.png)

###### 图 7-1\. Spark 外部数据源

# 关系数据库

让我们从关系数据库开始。关系数据库是一组数据项，以一组形式描述的表格组织起来（使用 SQL `CREATE` `TABLE` 语句创建），可以在不需要重新组织表格本身的情况下，以许多不同的方式访问或重新组装数据。开源关系数据库（如 MySQL 和 PostgreSQL）目前是存储社交媒体网络记录、财务记录、医疗记录、个人信息和制造数据的主要选择。还有许多著名且广泛使用的许可专有关系数据库，如 MS SQL Server 和 Oracle。

一个关系数据库表格非正式地具有一组行和命名列，如 图 7-2 所示。表格中的每一行可以有其自己的唯一键（称为主键）。表格中的行可以通过添加指向其他表格行的唯一键的列（这些列称为外键）来链接到其他表格的行。

![daws 0702](img/daws_0702.png)

###### 图 7-2\. 关系数据库表格示例

PySpark 提供了两个类，用于从关系数据库读取数据和将数据写入关系数据库，以及其他外部数据源。这两个类定义如下：

`class` `pyspark.sql.DataFrameReader(spark)`

这是从外部存储系统（文件系统、键值存储等）读取数据到 DataFrame 的接口。使用 `spark.read()` 来访问此功能。

`class` `pyspark.sql.DataFrameWriter(df)`

这是将 DataFrame 写入外部存储系统的接口。使用 `DataFrame.write()` 来访问此功能。

## 从数据库中读取数据

PySpark 使我们能够从关系数据库表中读取数据，并创建一个新的 DataFrame。您可以使用`pyspark.sql.DataFrameReader.load()` Python 方法从任何符合 JDBC 的数据库中读取表。`load()`方法定义如下：

```
load(path=None, format=None, schema=None, **options)
t :class`DataFrame`.

Parameters:
     path – optional string or a list of string
            for file-system backed data sources.
   format – optional string for format of the data
            source. Default to 'parquet'.
   schema – optional pyspark.sql.types.StructType for
            the input schema or a DDL-formatted string
            (for example `col-1 INT, col-2 DOUBLE`).
  options – all other string options
```

要从符合 JDBC 的数据库表中读取数据，需要指定`format("jdbc")`。然后可以将表属性和连接参数（例如 JDBC URL 和数据库凭据）作为`options(*<key>*, *<value>*)`对传入。

要读取并写入符合 JDBC 的关系数据库中的数据，您需要访问数据库服务器并具有足够的权限。

### 步骤 1\. 创建数据库表

在此步骤中，我们将连接到 MySQL 数据库服务器，并创建一个名为`dept`的表，包含七行数据。我们执行`mysql`客户端程序来进入 MySQL 客户端 shell（例如，如果你在 MacBook 上安装了 MySQL 数据库，则 MySQL 客户端将位于*/usr/local/bin/mysql*）：

```
$ mysql -uroot -p ![1](img/1.png)
Enter password: *<your-root-password>* ![2](img/2.png)
Welcome to the MySQL monitor.  Commands end with ; or \g.
Server version: 5.7.18 MySQL Community Server (GPL)
```

```
mysql> show databases; ![3](img/3.png)
+--------------------+ | Database           | ![4](img/4.png)
+--------------------+ | information_schema |
| mysql              | ![5](img/5.png)
| performance_schema |
+--------------------+ 3 rows in set (0.00 sec)
```

![1](img/#co_interacting_with_external_data_sources_CO1-1)

调用 MySQL shell 客户端。

![2](img/#co_interacting_with_external_data_sources_CO1-2)

输入`root`用户的有效密码。

![3](img/#co_interacting_with_external_data_sources_CO1-3)

列出 MySQL 数据库服务器中可用的数据库。

![4](img/#co_interacting_with_external_data_sources_CO1-4)

这三个数据库是由 MySQL 数据库服务器创建的。

![5](img/#co_interacting_with_external_data_sources_CO1-5)

`mysql`数据库管理用户、组和权限。

接下来，我们将创建并选择一个数据库：

```
mysql> create database metadb; ![1](img/1.png)
mysql> use metadb; ![2](img/2.png)
Database changed
mysql>
mysql> show tables; ![3](img/3.png)
Empty set (0.00 sec)
```

![1](img/#co_interacting_with_external_data_sources_CO2-1)

创建一个名为`metadb`的新数据库。

![2](img/#co_interacting_with_external_data_sources_CO2-2)

将`metadb`设置为当前默认数据库。

![3](img/#co_interacting_with_external_data_sources_CO2-3)

显示`metadb`数据库中的表（因为它是一个新数据库，所以其中将没有任何表）。

然后我们将在`metadb`数据库内创建一个名为`dept`的新表：

```
mysql> create table dept ( ![1](img/1.png)
    ->   dept_number int,
    ->   dept_name  varchar(128),
    ->   dept_location varchar(128),
    ->   manager varchar(128)
    -> );

mysql> show tables; ![2](img/2.png)
+------------------+ | Tables_in_metadb |
+------------------+ | dept             |
+------------------+ 
mysql> desc dept; ![3](img/3.png)
+---------------+--------------+------+-----+---------+-------+ | Field         | Type         | Null | Key | Default | Extra |
+---------------+--------------+------+-----+---------+-------+ | dept_number   | int(11)      | YES  |     | NULL    |       |
| dept_name     | varchar(128) | YES  |     | NULL    |       |
| dept_location | varchar(128) | YES  |     | NULL    |       |
| manager       | varchar(128) | YES  |     | NULL    |       |
+---------------+--------------+------+-----+---------+-------+
```

![1](img/#co_interacting_with_external_data_sources_CO3-1)

这是`dept`表的表定义，它包含四列。

![2](img/#co_interacting_with_external_data_sources_CO3-2)

列出`metadb`数据库中的表。

![3](img/#co_interacting_with_external_data_sources_CO3-3)

描述`dept`表的模式。

最后，我们使用`INSERT`语句将以下七行数据插入到`dept`表中：

```
mysql> INSERT INTO dept
  -> (dept_number,  dept_name,  dept_location, manager)
  -> VALUES
  ->  (10, 'ACCOUNTING', 'NEW YORK, NY', 'alex'),
  ->  (20, 'RESEARCH',   'DALLAS, TX', 'alex'),
  ->  (30, 'SALES',      'CHICAGO, IL', 'jane'),
  ->  (40, 'OPERATIONS', 'BOSTON, MA', 'jane'),
  ->  (50, 'MARKETING', 'Sunnyvale, CA', 'jane'),
  ->  (60, 'SOFTWARE', 'Stanford, CA', 'jane'),
  ->  (70, 'HARDWARE', 'BOSTON, MA', 'sophia');
```

我们可以检查`dept`表的内容，确保它包含这七行数据：

```
mysql> select * from dept;
+-------------+------------+---------------+---------+
| dept_number | dept_name  | dept_location | manager |
+-------------+------------+---------------+---------+
|          10 | ACCOUNTING | NEW YORK, NY  | alex    |
|          20 | RESEARCH   | DALLAS, TX    | alex    |
|          30 | SALES      | CHICAGO, IL   | jane    |
|          40 | OPERATIONS | BOSTON, MA    | jane    |
|          50 | MARKETING  | Sunnyvale, CA | jane    |
|          60 | SOFTWARE   | Stanford, CA  | jane    |
|          70 | HARDWARE   | BOSTON, MA    | sophia  |
+-------------+------------+---------------+---------+
7 rows in set (0.00 sec)
```

目前，我们确信数据库服务器上存在一个名为`metadb`的数据库，其中包含一个具有七条记录的`dept`表。

### 步骤 2：将数据库表读入 DataFrame 中

一旦您有一个 JDBC 兼容的表（如 `dept`），那么您可以使用 `pyspark.sql.DataFrameReader` 类的方法（`option()` 和 `load()` 的结合）来读取表的内容并创建一个新的 DataFrame。要执行此读取操作，您需要一个 JAR 文件，这是一个 MySQL JDBC 驱动程序（您可以从 [MySQL 网站](https://dev.mysql.com/downloads) 下载此 JAR 文件）。您可以将包含 MySQL 驱动程序类的 JAR 文件放在任何您喜欢的位置；我会把它放在：

```
 .../code/jars/mysql-connector-java-5.1.42.jar
```

###### 注意

MySQL 提供标准的数据库驱动程序连接（参见 [Connector/J](https://oreil.ly/WwK6k) 了解详情），用于与符合行业标准 ODBC 和 JDBC 的应用程序和工具一起使用 MySQL。任何支持 ODBC 或 JDBC 的系统都可以使用 MySQL。

接下来，我们通过将 JAR 文件传递给 `$SPARK_HOME/bin/pyspark` 程序来进入 PySpark shell：

```
export JAR=/book/code/jars/mysql-connector-java-5.1.42.jar ![1](img/1.png)
$SPARK_HOME/bin/pyspark --jars $JAR ![2](img/2.png)
```

```
SparkSession available as `'spark'`.
>>> spark  ![3](img/3.png)
<pyspark.sql.session.SparkSession object at 0x10a5f2a50>
>>>
```

![1](img/#co_interacting_with_external_data_sources_CO4-1)

这是 MySQL 的驱动程序类 JAR。

![2](img/#co_interacting_with_external_data_sources_CO4-2)

启动 PySpark shell，加载 MySQL 驱动程序类 JAR。

![3](img/#co_interacting_with_external_data_sources_CO4-3)

确保 `SparkSession` 可用。

现在我们可以使用 `SparkSession` 读取一个关系表并创建一个新的 DataFrame：

```
dataframe_mysql = spark \ ![1](img/1.png)
   .read \ ![2](img/2.png)
   .format("jdbc") \ ![3](img/3.png)
   .option("url", "jdbc:mysql://localhost/metadb") \ ![4](img/4.png)
   .option("driver", "com.mysql.jdbc.Driver") \ ![5](img/5.png)
   .option("dbtable", "dept") \ ![6](img/6.png)
   .option("user", "root") \ ![7](img/7.png)
   .option("password", "mp22_pass") \ ![8](img/8.png)
   .load() ![9](img/9.png)
```

![1](img/#co_interacting_with_external_data_sources_CO5-1)

`spark` 是 `SparkSession` 的一个实例。

![2](img/#co_interacting_with_external_data_sources_CO5-2)

返回一个可以用来读取数据作为 DataFrame 的 `DataFrameReader`

![3](img/#co_interacting_with_external_data_sources_CO5-3)

表明您正在读取 JDBC 兼容数据

![4](img/#co_interacting_with_external_data_sources_CO5-4)

数据库 URL

![5](img/#co_interacting_with_external_data_sources_CO5-5)

JDBC 驱动程序（从 JAR 文件加载）

![6](img/#co_interacting_with_external_data_sources_CO5-6)

数据库表名

![7](img/#co_interacting_with_external_data_sources_CO5-7)

数据库用户名

![8](img/#co_interacting_with_external_data_sources_CO5-8)

数据库密码

![9](img/#co_interacting_with_external_data_sources_CO5-9)

从 JDBC 数据源加载数据并将其作为 DataFrame 返回

让我们来看看新创建的 DataFrame：

```
>>> dataframe_mysql.count() ![1](img/1.png)
7
>>> dataframe_mysql.show() ![2](img/2.png)
+-----------+----------+-------------+-------+
|dept_number| dept_name|dept_location|manager|
+-----------+----------+-------------+-------+
|         10|ACCOUNTING| NEW YORK, NY|   alex|
|         20|  RESEARCH|   DALLAS, TX|   alex|
|         30|     SALES|  CHICAGO, IL|   jane|
|         40|OPERATIONS|   BOSTON, MA|   jane|
|         50| MARKETING|Sunnyvale, CA|   jane|
|         60|  SOFTWARE| Stanford, CA|   jane|
|         70|  HARDWARE|   BOSTON, MA| sophia|
+-----------+----------+-------------+-------+
```

![1](img/#co_interacting_with_external_data_sources_CO6-1)

计算 DataFrame 中行数。

![2](img/#co_interacting_with_external_data_sources_CO6-2)

打印前 20 行到控制台。

我们还可以查看其模式：

```
>>> dataframe_mysql.printSchema ![1](img/1.png)
<bound method DataFrame.printSchema of
DataFrame[
           dept_number: int,
           dept_name: string,
           dept_location: string,
           manager: string
          ]
```

![1](img/#co_interacting_with_external_data_sources_CO7-1)

以树状格式打印模式。

### 步骤 3：查询 DataFrame

PySpark 提供了许多访问 DataFrame 的方式。除了各种类似 SQL 的方法（如 `select(*<columns>*)`、`groupBy(*<columns>*)`、`min()`、`max()` 等），它还允许您通过首先将其注册为“表”，然后针对该注册表执行完全成熟的 SQL 查询来执行对 DataFrame 的查询。我们将很快讨论 DataFrame 表注册。首先，我们将使用 DataFrame 方法执行一些类似 SQL 的查询。

在这里，我们选择所有行的两列，`dept_number` 和 `manager`：

```
>>> dataframe_mysql.select("dept_number", "manager") ![1](img/1.png)
               .show() ![2](img/2.png)
+-----------+-------+
|dept_number|manager|
+-----------+-------+
|         10|   alex|
|         20|   alex|
|         30|   jane|
|         40|   jane|
|         50|   jane|
|         60|   jane|
|         70| sophia|
+-----------+-------+
```

![1](img/#co_interacting_with_external_data_sources_CO8-1)

从 DataFrame 中选择 `dept_number` 和 `manager` 列。

![2](img/#co_interacting_with_external_data_sources_CO8-2)

显示选择结果。

接下来，我们按 `manager` 列对所有行进行分组，然后找到最小的 `dept_number`：

```
>>> dataframe_mysql.select("dept_number", "manager")
                   .groupBy("manager")
                   .min("dept_number")
                   .collect()
[
 Row(manager=u'jane', min(dept_number)=30),
 Row(manager=u'sophia', min(dept_number)=70),
 Row(manager=u'alex', min(dept_number)=10)
]
```

在这里，我们按 `manager` 列对所有行进行分组，然后找到分组数据的频率：

```
>>> dataframe_mysql.select("dept_number", "manager")
                   .groupBy("manager")
                   .count()
                   .show()
+--------+-------+
|manager | count |
+--------+-------+
| jane   |  4    |
| sophia |  1    |
| alex   |  2    |
+--------+-------+
```

在这里我们做同样的事情，但额外按 `manager` 列排序输出：

```
>>> dataframe_mysql.select("dept_number", "manager")
                   .groupBy("manager")
                   .count()
                   .orderBy("manager")
                   .show()
+--------+-------+
|manager | count |
+--------+-------+
| alex   |  2    |
| jane   |  4    |
| sophia |  1    |
+--------+-------+
```

要对 DataFrame 执行完全成熟的 SQL 查询，首先必须将 DataFrame 注册为表：

```
DataFrame.registerTempTable(*<your-desired-table-name>*)

```

您随后可以对其执行常规的 SQL 查询，就像它是一个关系数据库表一样：

```
>>> dataframe_mysql.registerTempTable("mydept") ![1](img/1.png)
>>> spark.sql("select * from mydept where dept_number > 30") ![2](img/2.png)
         .show() ![3](img/3.png)
+-----------+----------+-------------+-------+
|dept_number| dept_name|dept_location|manager|
+-----------+----------+-------------+-------+
|         40|OPERATIONS|   BOSTON, MA|   jane|
|         50| MARKETING|Sunnyvale, CA|   jane|
|         60|  SOFTWARE| Stanford, CA|   jane|
|         70|  HARDWARE|   BOSTON, MA| sophia|
+-----------+----------+-------------+-------+
```

![1](img/#co_interacting_with_external_data_sources_CO9-1)

使用给定的名称将此 DataFrame 注册为临时表。

![2](img/#co_interacting_with_external_data_sources_CO9-2)

现在您可以针对您注册的表执行 SQL 查询了。

![3](img/#co_interacting_with_external_data_sources_CO9-3)

这会将前 20 行打印到控制台。

此查询使用 `dept_location` 列的“like”模式匹配：

```
>>> spark.sql("select * from mydept where dept_location like '%CA'")
         .show()
+-----------+----------+-------------+-------+
|dept_number| dept_name|dept_location|manager|
+-----------+----------+-------------+-------+
|         50| MARKETING|Sunnyvale, CA|   jane|
|         60|  SOFTWARE| Stanford, CA|   jane|
+-----------+----------+-------------+-------+
>>>
```

在这里，我们使用 `GROUP` `BY`：

```
>>> spark.sql("select manager, count(*) as count from mydept group by manager")
  .show()
+-------+-----+
|manager|count|
+-------+-----+
|   jane|    4|
| sophia|    1|
|   alex|    2|
+-------+-----+
```

## 将 DataFrame 写入数据库

我们可以使用 `DataFrameWriter.save()` 方法将 Spark DataFrame 写入或保存到外部数据源，比如关系数据库表。让我们通过一个示例来详细说明。

首先，我们将创建一个三元组列表 `(<name>, <age>, <salary>)` 作为本地 Python 集合：

```
>>> triplets = [ ("alex", 60, 18000),
...              ("adel", 40, 45000),
...              ("adel", 50, 77000),
...              ("jane", 40, 52000),
...              ("jane", 60, 81000),
...              ("alex", 50, 62000),
...              ("mary", 50, 92000),
...              ("mary", 60, 63000),
...              ("mary", 40, 55000),
...              ("mary", 40, 55000)
...            ]
```

然后，我们将使用 `SparkSession.createDataFrame()` 方法将其转换为 Spark DataFrame：

```
>>> tripletsDF = spark.createDataFrame( ![1](img/1.png)
...                     triplets, ![2](img/2.png)
...                     ['name', 'age', 'salary'] ![3](img/3.png)
...              )
>>> tripletsDF.show() ![4](img/4.png)
+----+---+------+
|name|age|salary|
+----+---+------+
|alex| 60| 18000|
|adel| 40| 45000|
|adel| 50| 77000|
|jane| 40| 52000|
|jane| 60| 81000|
|alex| 50| 62000|
|mary| 50| 92000|
|mary| 60| 63000|
|mary| 40| 55000|
|mary| 40| 55000|
+----+---+------+
```

![1](img/#co_interacting_with_external_data_sources_CO10-1)

创建一个新的 DataFrame。

![2](img/#co_interacting_with_external_data_sources_CO10-2)

将三元组转换为 DataFrame。

![3](img/#co_interacting_with_external_data_sources_CO10-3)

对创建的 DataFrame 强制执行模式。

![4](img/#co_interacting_with_external_data_sources_CO10-4)

显示新创建的 DataFrame 的内容。

现在，我们可以将 DataFrame 转换为名为 `triplets` 的关系表：

```
tripletsDF
  .write ![1](img/1.png)
  .format("jdbc") ![2](img/2.png)
  .option("driver", "com.mysql.jdbc.Driver") ![3](img/3.png)
  .mode("overwrite") ![4](img/4.png)
  .option("url", "jdbc:mysql://localhost/metadb") ![5](img/5.png)
  .option("dbtable", "triplets") ![6](img/6.png)
  .option("user", "root") ![7](img/7.png)
  .option("password", "mp22_pass") ![8](img/8.png)
  .save() ![9](img/9.png)
```

![1](img/#co_interacting_with_external_data_sources_CO11-1)

返回一个可以用于向外部设备写入的 `DataFrameWriter`

![2](img/#co_interacting_with_external_data_sources_CO11-2)

表明您正在向兼容 JDBC 的数据库写入

![3](img/#co_interacting_with_external_data_sources_CO11-3)

JDBC 驱动程序（从 JAR 文件加载）

![4](img/#co_interacting_with_external_data_sources_CO11-4)

如果表已经存在，则覆盖该表

![5](img/#co_interacting_with_external_data_sources_CO11-5)

数据库 URL

![6](img/#co_interacting_with_external_data_sources_CO11-6)

目标数据库表名称

![7](img/#co_interacting_with_external_data_sources_CO11-7)

数据库用户名

![8](img/#co_interacting_with_external_data_sources_CO11-8)

数据库密码

![9](img/#co_interacting_with_external_data_sources_CO11-9)

将 DataFrame 数据保存为数据库表

当将 DataFrame 的内容写入外部设备时，可以选择所需的模式。Spark JDBC 写入器支持以下模式：

`append`

将 DataFrame 的内容附加到任何现有数据。

`overwrite`

覆盖任何现有数据。

`ignore`

如果数据已经存在，则静默地忽略此操作。

`error`（默认情况）

如果数据已经存在，则抛出异常。

在这里，我们验证 `triplets` 表已在 MySQL 数据库服务器上的 `'metadb'` 数据库中创建：

```
$ mysql -uroot -p ![1](img/1.png)
Enter password:  *<password>* ![2](img/2.png)
Welcome to the MySQL Server version: 5.7.18
```

```
mysql> use metadb; ![3](img/3.png)
Database changed

mysql> desc triplets; ![4](img/4.png)
+--------+------------+------+-----+---------+-------+ | Field  | Type       | Null | Key | Default | Extra |
+--------+------------+------+-----+---------+-------+ | name   | text       | YES  |     | NULL    |       |
| age    | bigint(20) | YES  |     | NULL    |       |
| salary | bigint(20) | YES  |     | NULL    |       |
+--------+------------+------+-----+---------+-------+ 
mysql> select * from triplets; ![5](img/5.png)
+------+------+--------+ | name | age  | salary |
+------+------+--------+ | jane |   40 |  52000 |
| adel |   50 |  77000 |
| jane |   60 |  81000 |
| alex |   50 |  62000 |
| mary |   40 |  55000 |
| mary |   40 |  55000 |
| adel |   40 |  45000 |
| mary |   60 |  63000 |
| alex |   60 |  18000 |
| mary |   50 |  92000 |
+------+------+--------+ 10 rows in set (0.00 sec)
```

![1](img/ch02.xhtml#comarker1)

启动 MySQL 客户端 shell。

![2](img/ch02.xhtml#comarker2)

为 `root` 用户输入密码。

![3](img/ch02.xhtml#comarker3)

选择所需的数据库。

![4](img/ch02.xhtml#comarker4)

确保 `triplets` 表已创建。

![5](img/ch02.xhtml#comarker5)

显示 `triplets` 表的内容。

接下来，我们从 MySQL 关系数据库中重新读取 `triplets` 表，以确保该表可读：

```
>>> tripletsDF_mysql =
      spark ![1](img/1.png)
       .read ![2](img/2.png)
       .format("jdbc") ![3](img/3.png)
       .option("url", "jdbc:mysql://localhost/metadb") ![4](img/4.png)
       .option("driver", "com.mysql.jdbc.Driver") ![5](img/5.png)
       .option("dbtable", "triplets") ![6](img/6.png)
       .option("user", "root") ![7](img/7.png)
       .option("password", "mp22_pass") ![8](img/8.png)
       .load() ![9](img/9.png)

>>> tripletsDF_mysql.show() ![10](img/10.png)
+----+---+------+
|name|age|salary|
+----+---+------+
|jane| 40| 52000|
|adel| 50| 77000|
|jane| 60| 81000|
|alex| 50| 62000|
|mary| 40| 55000|
|mary| 40| 55000|
|adel| 40| 45000|
|mary| 60| 63000|
|alex| 60| 18000|
|mary| 50| 92000|
+----+---+------+
```

![1](img/#co_interacting_with_external_data_sources_CO12-1)

`SparkSession` 的一个实例

![2](img/#co_interacting_with_external_data_sources_CO12-2)

返回一个 `DataFrameReader`，可用于将数据读取为 DataFrame

![3](img/#co_interacting_with_external_data_sources_CO12-3)

表示正在读取符合 JDBC 的数据

![4](img/#co_interacting_with_external_data_sources_CO12-4)

数据库 URL

![5](img/#co_interacting_with_external_data_sources_CO12-5)

JDBC 驱动程序（从 JAR 文件加载）

![6](img/#co_interacting_with_external_data_sources_CO12-6)

要读取的数据库表

![7](img/#co_interacting_with_external_data_sources_CO12-7)

数据库用户名

![8](img/#co_interacting_with_external_data_sources_CO12-8)

数据库密码

![9](img/#co_interacting_with_external_data_sources_CO12-9)

从 JDBC 数据源加载数据并将其返回为 DataFrame

![10](img/#co_interacting_with_external_data_sources_CO12-10)

显示新创建 DataFrame 的内容

最后，我们将在新创建的 DataFrame 上执行一些 SQL 查询。

下面的查询找到 `salary` 列的最小值和最大值：

```
>>> tripletsDF_mysql.registerTempTable("mytriplets") ![1](img/1.png)
>>> spark.sql("select min(salary), max(salary) from mytriplets") ![2](img/2.png)
         .show() ![3](img/3.png)
+-----------+-----------+
|min(salary)|max(salary)|
+-----------+-----------+
|      18000|      92000|
+-----------+-----------+
```

![1](img/#co_interacting_with_external_data_sources_CO13-1)

使用名为 `mytriplets` 的临时表名注册此 DataFrame。

![2](img/#co_interacting_with_external_data_sources_CO13-2)

执行 SQL 语句并创建新的 DataFrame。

![3](img/#co_interacting_with_external_data_sources_CO13-3)

显示 SQL 语句的结果。

在这里，我们通过使用 SQL 的 `GROUP BY` 对 `age` 列进行聚合：

```
>>> spark.sql("select age, count(*) from mytriplets group by age").show()
+---+--------+
|age|count(1)|
+---+--------+
| 50|       3|
| 60|       3|
| 40|       4|
+---+--------+
```

接下来，我们对前面 SQL 查询的结果进行排序：

```
>>> spark.sql("select age, count(*) from mytriplets group by age order by age")
  .show()
+---+--------+
|age|count(1)|
+---+--------+
| 40|       4|
| 50|       3|
| 60|       3|
+---+--------+
```

# 读取文本文件

Spark 允许我们读取文本文件并从中创建 DataFrame。考虑以下文本文件：

```
$ cat people.txt
Alex,30,Tennis
Betty,40,Swimming
Dave,20,Walking
Jeff,77,Baseball
```

让我们首先创建一个 `RDD[Row]`（其中每个元素是一个 `Row` 对象）：

```
>>> from pyspark.sql import Row

>>> def create_row(rec):
        p = rec.split(",")
        return Row(name=p[0], age=int(p[1]), hobby=p[2])
>>> #end-def
>>> input_path = "people.txt"
>>> # Load a text file and convert each line to a Row
>>> records = spark.sparkContext.textFile(input_path) ![1](img/1.png)
>>> records.collect()
[
 u'Alex,30,Tennis',
 u'Betty,40,Swimming',
 u'Dave,20,Walking',
 u'Jeff,77,Baseball'
]
>>> people = records.map(create_row) ![2](img/2.png)
>>> people.collect()
[
 Row(age=30, hobby=u'Tennis', name=u'Alex'),
 Row(age=40, hobby=u'Swimming', name=u'Betty'),
 Row(age=20, hobby=u'Walking', name=u'Dave'),
 Row(age=77, hobby=u'Baseball', name=u'Jeff')
]
```

![1](img/#co_interacting_with_external_data_sources_CO14-1)

`records` 是一个 `RDD[String]`。

![2](img/#co_interacting_with_external_data_sources_CO14-2)

`people` 是一个 `RDD[Row]`。

现在我们有了 `people` 作为 `RDD[Row]`，可以轻松地创建一个 DataFrame：

```
>>> people_df = spark.createDataFrame(people) ![1](img/1.png)
>>> people_df.show()
+---+--------+-----+
|age|   hobby| name|
+---+--------+-----+
| 30|  Tennis| Alex|
| 40|Swimming|Betty|
| 20| Walking| Dave|
| 77|Baseball| Jeff|
+---+--------+-----+
>>> people_df.printSchema() ![2](img/2.png)
root
 |-- age: long (nullable = true)
 |-- hobby: string (nullable = true)
 |-- name: string (nullable = true)
```

![1](img/#co_interacting_with_external_data_sources_CO15-1)

`people_df` 是一个 `DataFrame[Row]`。

![2](img/#co_interacting_with_external_data_sources_CO15-2)

显示创建的 DataFrame 的模式。

接下来，我们将使用 SQL 查询来操作创建的 DataFrame：

```
>>> people_df.registerTempTable("people_table") ![1](img/1.png)
>>> spark.sql("select * from people_table").show() ![2](img/2.png)
+---+--------+-----+
|age|   hobby| name|
+---+--------+-----+
| 30|  Tennis| Alex|
| 40|Swimming|Betty|
| 20| Walking| Dave|
| 77|Baseball| Jeff|
+---+--------+-----+

>>> spark.sql("select * from people_table where age > 35").show() ![3](img/3.png)
+---+--------+-----+
|age|   hobby| name|
+---+--------+-----+
| 40|Swimming|Betty|
| 77|Baseball| Jeff|
+---+--------+-----+
```

![1](img/#co_interacting_with_external_data_sources_CO16-1)

将 `people_df` DataFrame 注册为临时表，表名为 `people_table`。

![2](img/#co_interacting_with_external_data_sources_CO16-2)

`spark.sql(*sql-query*)` 创建一个新的 DataFrame。

![3](img/#co_interacting_with_external_data_sources_CO16-3)

`spark.sql(*sql-query*)` 创建一个新的 DataFrame。

我们可以使用 `DataFrame.write()` 将 DataFrame 保存为文本文件。

# 读取和写入 CSV 文件

逗号分隔值文件是一种文本文件，允许以表格结构的格式保存数据。以下是一个带有标题行（包含列名的元数据，用逗号分隔）的简单 CSV 文件示例，名为 *cats.with.header.csv*：

```
$ cat cats.with.header.csv
#name,age,gender,weight ![1](img/1.png)
cuttie,2,female,6 ![2](img/2.png)
mono,3,male,9 ![3](img/3.png)
fuzzy,1,female,4 ![4](img/4.png)
```

![1](img/#co_interacting_with_external_data_sources_CO17-1)

头记录以 `#` 开头，描述列

![2](img/#co_interacting_with_external_data_sources_CO17-2)

第一条记录

![3](img/#co_interacting_with_external_data_sources_CO17-3)

第二条记录

![4](img/#co_interacting_with_external_data_sources_CO17-4)

第三和最后一条记录

以下是一个不带标题的简单 CSV 文件示例，名为 *cats.no.header.csv*：

```
$ cat cats.no.header.csv
cuttie,2,female,6
mono,3,male,9
fuzzy,1,female,4
```

在接下来的部分，我们将使用这两个文件演示 Spark 如何读取 CSV 文件。

## 读取 CSV 文件

Spark 提供了许多方法将 CSV 文件加载到 DataFrame 中。这里我会展示其中一些方法。

在这个例子中，使用 PySpark shell，我们读取了一个带有标题的 CSV 文件，并将其加载为 DataFrame：

```
# spark : pyspark.sql.session.SparkSession object
input_path = '/pyspark_book/code/chap08/cats.with.header.csv'
cats = spark ![1](img/1.png)
        .read ![2](img/2.png)
        .format("csv") ![3](img/3.png)
        .option("header", "true") ![4](img/4.png)
        .option("inferSchema", "true") ![5](img/5.png)
        .load(input_path) ![6](img/6.png)
```

![1](img/#co_interacting_with_external_data_sources_CO18-1)

使用 `SparkSession` 创建一个名为 `cats` 的新 `DataFrame`。

![2](img/#co_interacting_with_external_data_sources_CO18-2)

返回一个可以用于将数据读取为 DataFrame 的`DataFrameReader`。

![3](img/#co_interacting_with_external_data_sources_CO18-3)

指定输入数据源格式为 CSV。

![4](img/#co_interacting_with_external_data_sources_CO18-4)

指示输入的 CSV 文件包含标题（注意标题不是实际数据的一部分）。

![5](img/#co_interacting_with_external_data_sources_CO18-5)

从输入文件推断 DataFrame 的模式。

![6](img/#co_interacting_with_external_data_sources_CO18-6)

提供 CSV 文件的路径。

现在我们可以显示新创建的 DataFrame 的内容和推断出的模式：

```
>>> cats.show() ![1](img/1.png)
+------+---+------+------+
|  name|age|gender|weight|
+------+---+------+------+
|cuttie|  2|female|     6|
|  mono|  3|  male|     9|
| fuzzy|  1|female|     4|
+------+---+------+------+

>>> cats.printSchema ![2](img/2.png)
<bound method DataFrame.printSchema of DataFrame
[
 name: string,
 age: int,
 gender: string,
 weight: int
]>
>>> cats.count() ![3](img/3.png)
3
```

![1](img/#co_interacting_with_external_data_sources_CO19-1)

显示 DataFrame 的内容。

![2](img/#co_interacting_with_external_data_sources_CO19-2)

显示 DataFrame 的模式。

![3](img/#co_interacting_with_external_data_sources_CO19-3)

显示 DataFrame 的大小。

现在我将向您展示如何读取一个没有标题的 CSV 文件，并从中创建一个新的 DataFrame：

```
input_path = '/pyspark_book/code/chap08/cats.no.header.csv'
cats2 = spark ![1](img/1.png)
            .read ![2](img/2.png)
            .format("csv") ![3](img/3.png)
            .option("header","false") ![4](img/4.png)
            .option("inferSchema", "true") ![5](img/5.png)
            .load(input_path) ![6](img/6.png)
```

![1](img/#co_interacting_with_external_data_sources_CO20-1)

使用`SparkSession`创建一个名为`cats`的新 DataFrame。

![2](img/#co_interacting_with_external_data_sources_CO20-2)

返回一个可以用于将数据读取为 DataFrame 的`DataFrameReader`。

![3](img/#co_interacting_with_external_data_sources_CO20-3)

指定输入数据源格式为 CSV。

![4](img/#co_interacting_with_external_data_sources_CO20-4)

指示输入的 CSV 文件没有标题。

![5](img/#co_interacting_with_external_data_sources_CO20-5)

从输入文件推断 DataFrame 的模式。

![6](img/#co_interacting_with_external_data_sources_CO20-6)

提供 CSV 文件的路径。

让我们检查新创建的 DataFrame 及其推断出的模式的内容：

```
>>> cats2.show()
+------+---+------+---+
|   _c0|_c1|   _c2|_c3| ![1](img/1.png)
+------+---+------+---+
|cuttie|  2|female|  6|
|  mono|  3|  male|  9|
| fuzzy|  1|female|  4|
+------+---+------+---+
```

![1](img/#co_interacting_with_external_data_sources_CO21-1)

默认列名

接下来，我们将定义一个包含四列的模式：

```
>>> from pyspark.sql.types import StructType
>>> from pyspark.sql.types import StructField
>>> from pyspark.sql.types import StringType
>>> from pyspark.sql.types import IntegerType
>>>
>>> catsSchema = StructType([
... StructField("name", StringType(), True),
... StructField("age", IntegerType(), True),
... StructField("gender", StringType(), True),
... StructField("weight", IntegerType(), True)
... ])
```

如果我们使用定义的模式读取相同的 CSV 文件并从中创建一个 DataFrame，会发生以下情况：

```
>>> input_path = '/book/code/chap07/cats.no.header.csv'
>>> cats3 = spark
   .read
   .format("csv")
   .option("header","false")
   .option("inferSchema", "true")
   .load(input_path, schema = catsSchema)
>>> cats3.show()
+------+---+------+------+
|  name|age|gender|weight| ![1](img/1.png)
+------+---+------+------+
|cuttie|  2|female|     6|
|  mono|  3|  male|     9|
| fuzzy|  1|female|     4|
+------+---+------+------+

>>> cats.count()
3
```

![1](img/#co_interacting_with_external_data_sources_CO22-1)

使用显式列名

我们可以将这个预定义的模式应用于任何没有标题的 CSV 文件：

```
>>> cats4 = spark
   .read ![1](img/1.png)
    .csv("file:///tmp/cats.no.header.csv", ![2](img/2.png)
         schema = catsSchema, ![3](img/3.png)
         header = "false") ![4](img/4.png)
>>> cats4.show()
+------+---+------+------+
|  name|age|gender|weight| ![5](img/5.png)
+------+---+------+------+
|cuttie|  2|female|     6|
|  mono|  3|  male|     9|
| fuzzy|  1|female|     4|
+------+---+------+------+

>>> cats4.printSchema
<bound method DataFrame.printSchema of DataFrame
[
 name: string,
 age: int,
 gender: string,
 weight: int
]>
>>> cats4.count()
3
```

![1](img/#co_interacting_with_external_data_sources_CO23-1)

返回一个可以用于将数据读取为 DataFrame 的`DataFrameReader`。

![2](img/#co_interacting_with_external_data_sources_CO23-2)

读取一个 CSV 文件。

![3](img/#co_interacting_with_external_data_sources_CO23-3)

使用给定的模式来读取 CSV 文件。

![4](img/#co_interacting_with_external_data_sources_CO23-4)

指示 CSV 文件没有标题。

![5](img/#co_interacting_with_external_data_sources_CO23-5)

使用显式列名。

## 写入 CSV 文件

有几种方法可以在 Spark 中从 DataFrames 创建 CSV 文件。最简单的选项是使用`DataFrameWriter`类的`.csv()`方法，通过`DataFrame.write()`访问。该方法定义如下（请注意，这只是可用参数的一个小子集）：

```
csv(path, mode=None, compression=None, sep=None, ...)
Saves the content of the DataFrame in CSV format
at the specified path.
```

```
Parameters:
  path – the path in any Hadoop supported file system.
  mode – specifies the behavior of the save operation when data already exists.
         "append":  Append contents of this DataFrame to existing data.
         "overwrite": Overwrite existing data.
         "ignore": Silently ignore this operation if data already exists.
         "error": Throw an exception if data already exists.
  compression – compression codec to use when saving to file.
  sep – sets a single character as a separator for each field and value.
         If None is set, it uses the default value.
```

让我们使用这种方法将我们的`cats4` DataFrame 保存为 CSV 文件：

```
>>> cats4.show()
+------+---+------+------+
|  name|age|gender|weight|
+------+---+------+------+
|cuttie|  2|female|     6|
|  mono|  3|  male|     9|
| fuzzy|  1|female|     4|
+------+---+------+------+

>>> cats4.write.csv("file:///tmp/cats4", sep = ';')
```

然后检查保存的文件：

```
$ ls -l /tmp/cats4
total 8
-rw-r--r--  ...   0 Apr 12 16:46 _SUCCESS
-rw-r--r--  ...  49 Apr 12 16:46 part-00000-...-c000.csv

$ cat /tmp/cats4/part*
cuttie;2;female;6
mono;3;male;9
fuzzy;1;female;4
```

注意，在`ls`（列出）命令的输出中，我们看到两种类型的文件：

+   大小为零的*SUCCESS*文件，表示`write`操作成功。

+   一个或多个以*part-*开头的文件，这些文件代表单个分区的输出。

还要注意保存的文件中没有标题数据。

让我们再试一次，这次指定我们想要一个标题行：

```
>>> cats4.write.csv("file:///tmp/cats48",
                    sep = ';',
                    header = 'true')
```

```
$ ls -l /tmp/cats48
total 8
-rw-r--r--  ...   0 Apr 12 16:49 _SUCCESS
-rw-r--r--  ...  72 Apr 12 16:49 part-00000-...-c000.csv

$ cat /tmp/cats48/part*
name;age;gender;weight ![1](img/1.png)
cuttie;2;female;6
mono;3;male;9
fuzzy;1;female;4
```

![1](img/#co_interacting_with_external_data_sources_CO24-1)

我们 DataFrame 的标题

# 读取和写入 JSON 文件

JavaScript 对象表示法是一种轻量级的基于文本的数据交换格式，易于人类阅读和编写。JSON 对象由一组（键，值）对组成，用花括号括起来，像这样：

```
{
  "first_name" : "John", ![1](img/1.png)
  "last_name" : "Smith",
  "age" : 23,
  "gender" : "Male",
  "cars": [ "Ford", "BMW", "Fiat" ] ![2](img/2.png)
}
```

![1](img/#co_interacting_with_external_data_sources_CO25-1)

一个简单的（键，值）对

![2](img/#co_interacting_with_external_data_sources_CO25-2)

一个数组值

## 读取 JSON 文件

JSON 数据可以使用`DataFrameReader.json()`方法读取，该方法可以接受一组参数，如路径和模式。考虑以下 JSON 文件：

```
$ cat $SPARK_HOME/examples/src/main/resources/employees.json
{"name":"Michael", "salary":3000}
{"name":"Andy", "salary":4500}
{"name":"Justin", "salary":3500}
{"name":"Berta", "salary":4000}
```

我们可以按以下方式读取此文件并将其转换为 DataFrame：

```
>>> data_path = 'examples/src/main/resources/employees.json'
>>> df = spark.read.json(data_path)
>>> df.show()
+-------+------+
|   name|salary|
+-------+------+
|Michael|  3000|
|   Andy|  4500|
| Justin|  3500|
|  Berta|  4000|
+-------+------+

>>> df.printSchema
<bound method DataFrame.printSchema of DataFrame
[
 name: string,
 salary: bigint
]>
>>> df.count()
4
```

您还可以使用`load()`方法，并将其传递给一个或多个 JSON 文件：

```
>>> data_path = 'examples/src/main/resources/employees.json'
>>> df2 = spark.read.format('json')
               .load([data_path,  data_path]) ![1](img/1.png)
>>> df2.show()
+-------+------+
|   name|salary|
+-------+------+
|Michael|  3000|
|   Andy|  4500|
| Justin|  3500|
|  Berta|  4000|
|Michael|  3000| ![2](img/2.png)
|   Andy|  4500|
| Justin|  3500|
|  Berta|  4000|
+-------+------+

>>> df2.printSchema
<bound method DataFrame.printSchema of DataFrame
[
 name: string,
 salary: bigint
]>
>>> df2.count()
8
```

![1](img/#co_interacting_with_external_data_sources_CO26-1)

注意，`data_path`加载了两次。

![2](img/#co_interacting_with_external_data_sources_CO26-2)

结果 DataFrame 中因此包含文件内容两次。您还可以使用此方法从多个输入文件创建 DataFrame。

## 写 JSON 文件

要将 DataFrame 写入`json`对象，我们可以使用`DataFrameWriter.json()`方法。该方法接受一组参数，并将 DataFrame 的内容保存为 JSON 格式：

```
json(
     path,
     mode=None,
     compression=None,
     dateFormat=None,
     timestampFormat=None
)

Parameters:
  path – the path in any Hadoop supported file system
  mode – specifies the behavior of the save operation when data already exists.
         Possible values are: "append", "overwrite", "ignore", "error"
  compression – compression codec to use when saving to file.
  dateFormat – sets the string that indicates a date format.
  timestampFormat – sets the string that indicates a timestamp format.
```

让我们首先创建一个 DataFrame：

```
>>> data = [("name", "alex"), ("gender", "male"), ("state", "CA")]
>>> df = spark.createDataFrame(data, ['key', 'value'])
>>> df.show()
+------+-----+
|   key|value|
+------+-----+
|  name| alex|
|gender| male|
| state|   CA|
+------+-----+
```

接下来，将其作为 JSON 写入输出路径：

```
>>> df.write.json('/tmp/data')
```

```
$ ls -l /tmp/data
total 24
-rw-r--r-- ...   0 Apr  2 01:15 _SUCCESS
-rw-r--r-- ...   0 Apr  2 01:15 part-00000-...-c000.json
-rw-r--r-- ...   0 Apr  2 01:15 part-00001-...-c000.json
...
-rw-r--r-- ...  29 Apr  2 01:15 part-00007-...-c000.json
```

注意，我们有八个以*part-*开头的文件名：这意味着我们的 DataFrame 由八个分区表示。

让我们来看看这些文件：

```
$ cat /tmp/data/part*
{"key":"name","value":"alex"}
{"key":"gender","value":"male"}
{"key":"state","value":"CA"}
```

如果要创建单个文件输出，则可以在将其写出之前将 DataFrame 放入单个分区：

```
>>> df.repartition(1).write.json('/tmp/data') ![1](img/1.png)
```

![1](img/#co_interacting_with_external_data_sources_CO27-1)

`repartition(numPartitions)`返回一个由给定分区表达式分区的新 DataFrame；有关详细信息，请参见第五章。

# 从 Amazon S3 读取和写入

Amazon Simple Storage Service（S3）是由 Amazon Web Services（AWS）提供的一项服务，通过 Web 服务接口提供对象存储。S3 对象被视为 Web 对象，即通过 Internet 协议使用 URL 标识符访问。每个 S3 对象都有一个唯一的 URL，格式如下：

```
http://s3.*<region>*.amazonaws.com/*<bucket>*/*<key>*
```

例如：

```
http://s3.us-east-1.amazonaws.com/project-dev/dna/sample123.vcf
```

其中：`project-dev`是存储桶名称，`dna/sample123.vcf`是一个键。

S3 对象也可以通过以下 URI 模式访问：

`s3n`

使用 S3 本地文件系统，这是用于在 S3 上读写常规文件的本地文件系统。

`s3a`

使用 S3A 文件系统，这是原生文件系统的继任者。设计为`s3n`的替代品，此文件系统绑定支持更大的文件并承诺更高的性能。

`s3`

使用 S3 块文件系统，这是一个基于块的文件系统，由 S3 支持。文件以块的形式存储，就像在 HDFS 中一样。

`s3`和`s3n`/`s3a`的区别在于，`s3`是 Amazon S3 上的块级覆盖层，而`s3n`/`s3a`不是（它们是基于对象的）。`s3n`和`s3a`的区别在于，`s3n`支持最大 5GB 的对象大小，而`s3a`支持最大 5TB 的对象大小并且具有更好的性能（这两个特性是因为它使用了多部分上传）。

例如，使用`s3` URI 模式，我们可以访问*sample72.vcf*文件如下：

```
s3://project-dev/dna/sample72.vcf
```

一般来说，要访问 AWS 的任何服务，您必须经过身份验证。有很多方法可以做到这一点。一种方法是从命令行导出您的访问密钥和秘密密钥：

```
export AWS_ACCESS_KEY_ID="AKIAI74O5KPLUQGVOJWQ"
export AWS_SECRET_ACCESS_KEY="LmuKE7afdasdfxK2vj1nfA0Bp"
```

另一个选项是使用`SparkContext`对象来设置您的凭据：

```
# spark: SparkSession
sc = spark.sparkContext
# set access key
sc._jsc.hadoopConfiguration()
   .set("fs.s3.awsAccessKeyId", "AKIAI74O5KPLUQGVOJWQ")
# set secret key
sc._jsc.hadoopConfiguration()
  .set("fs.s3.awsSecretAccessKey", "LmuKE7afdasdfxK2vj1nfA0Bp")
```

## 从 Amazon S3 读取

您需要使用`s3`、`s3n`或`s3a`（用于更大的 S3 对象）URI 模式来从 S3 中读取对象。

如果`spark`是`SparkSession`的实例，则可以使用以下内容加载文本文件（Amazon S3 对象），并返回一个带有名为`value`的单个字符串列的 DataFrame（变量名为`df`）：

```
s3_object_path = "s3n://bucket-name/object-path"
df = spark.read.text(s3_object_path)
```

以下示例展示了如何读取 S3 对象。首先我们使用`boto3`库（`boto3`是 AWS 的 Python SDK，允许 Python 开发者编写利用 Amazon 服务如 S3 和 EC2 的软件）来验证对象是否存在，然后使用 PySpark 来读取它。

在以下代码中，我们检查是否存在`s3://caselog-dev/tmp/csv_file_10_rows.csv`：

```
>>> import boto3
>>> s3 = boto3.resource('s3')
>>> bucket = 'caselog-dev'
>>> key = 'tmp/csv_file_10_rows.csv'
>>> obj = s3.Object(bucket, key)
>>> obj
s3.Object(bucket_name='caselog-dev', key='tmp/csv_file_10_rows.csv')
>>> obj.get()['Body'].read().decode('utf-8')
u'0,a,0.0\n1,b,1.1\n2,c,2.2\n3,d,\n4,,4.4\n,f,5.5\n,,\n7,h,7.7\n8,i,8.8\n9,j,9.9'
```

然后，我们加载该对象并创建一个新的`DataFrame[String]`：

```
>>> s3_object_path = "s3n://caselog-dev/tmp/csv_file_10_rows.csv" ![1](img/1.png)
>>> df = spark.read.text(s3_object_path) ![2](img/2.png)
>>> df.show() ![3](img/3.png)
+-------+
|  value|
+-------+
|0,a,0.0|
|1,b,1.1|
|2,c,2.2|
|   3,d,|
| 4,,4.4|
| ,f,5.5|
|     ,,|
|7,h,7.7|
|8,i,8.8|
|9,j,9.9|
+-------+

>>> df.printSchema  ![4](img/4.png)
<bound method DataFrame.printSchema of DataFrame[value: string]>
```

![1](img/#co_interacting_with_external_data_sources_CO28-1)

定义您的 S3 对象路径。

![2](img/#co_interacting_with_external_data_sources_CO28-2)

使用`SparkSession`（作为`spark`）加载 S3 对象并创建一个 DataFrame。

![3](img/#co_interacting_with_external_data_sources_CO28-3)

展示新创建的 DataFrame 的内容。

![4](img/#co_interacting_with_external_data_sources_CO28-4)

显示新创建的 DataFrame 的模式。

## 写入到 Amazon S3

一旦您创建了 DataFrame：

```
>>># spark: SparkSession
>>> pairs_data = [("alex", 4), ("alex", 8),
                  ("rafa", 3), ("rafa", 6)]
>>> df = spark.createDataFrame(pairs_data, ['name', 'number'])
```

您可以检查其内容及其关联的架构：

```
>>> df.show()
+----+------+
|name|number|
+----+------+
|alex|     4|
|alex|     8|
|rafa|     3|
|rafa|     6|
+----+------+

>>> df.printSchema
<bound method DataFrame.printSchema of DataFrame
[
 name: string,
 number: bigint
]>
```

接下来，将数据保存到 Amazon S3 文件系统：

```
>>> df
     .write
     .format("csv")
     .mode("overwrite")
     .save("s3n://caselog-dev/output/pairs")
```

您将看到以下文件被创建：

```
https://s3.amazonaws.com/caselog-dev/output/pairs/_SUCCESS
https://s3.amazonaws.com/caselog-dev/output/pairs/part-00000-...-c000.csv
https://s3.amazonaws.com/caselog-dev/output/pairs/part-00001-...-c000.csv
```

现在让我们将其读取回来：

```
>>># Read S3 object as text
>>> s3_object_path = "s3n://caselog-dev/output/pairs"
>>> df = spark.read.text(s3_object_path)
>>> df.show()
+------+
| value|
+------+
|alex,4|
|alex,8|
|rafa,3|
|rafa,6|
+------+

>>> df.printSchema
<bound method DataFrame.printSchema of DataFrame[value: string]>
```

我们也可以将 S3 对象读取为 CSV 格式：

```
>>> df2 = spark.read.format("csv").load(s3_object_path)
>>> df2.show()
+----+---+
| _c0|_c1| ![1](img/1.png)
+----+---+
|alex|  4|
|alex|  8|
|rafa|  3|
|rafa|  6|
+----+---+
```

![1](img/#co_interacting_with_external_data_sources_CO29-1)

默认列名

# 读取和写入 Hadoop 文件

[Hadoop](http://hadoop.apache.org) 是一个开源的 MapReduce 编程框架，支持在分布式计算环境中处理和存储极大数据集。它设计用于从单个服务器扩展到数千台机器。由 [Apache Software Foundation](http://apache.org)（ASF）赞助的 Hadoop 项目包括以下模块：

Hadoop 公共模块

支持其他 Hadoop 模块的常见实用工具。

Hadoop 分布式文件系统（HDFS）

一个分布式文件系统，提供高吞吐量访问应用程序数据。HDFS 允许在使用 MapReduce 编程模型的计算机集群上进行大数据集的分布式处理。

Hadoop YARN

用于作业调度和集群资源管理的框架。

Hadoop MapReduce

基于 YARN 的系统，用于大数据集的并行处理。

在本节中，我将展示如何从 HDFS 中读取文件并创建 RDD 和 DataFrame，以及如何将 RDD 和 DataFrame 写入 HDFS。要跟随示例，您需要访问一个 Hadoop 集群。

## 读取 Hadoop 文本文件

为了演示从 HDFS 中读取文件的完整过程，首先我们将在 HDFS 中创建一个文本文件，然后使用 PySpark 将其读取为 DataFrame 和 RDD。

让 *name_age_salary.csv* 是 Linux 文件系统中的一个文本文件（可以使用任何文本编辑器创建此文件——注意 `$` 是 Linux 操作系统提示符）：

```
$ export input_path = "/book/code/chap07/name_age_salary.csv"
$ cat $input_path
alex,60,18000
adel,40,45000
adel,50,77000
jane,40,52000
jane,60,81000
alex,50,62000
mary,50,92000
mary,60,63000
mary,40,55000
mary,40,55000
```

使用 **`$HADOOP_HOME/bin/hdfs`** 命令在 HDFS 中创建一个 */test* 目录：

```
$ hdfs dfs -mkdir /test
```

然后我将 *name_age_salary.csv* 复制到 *hdfs:///test/* 目录：

```
$ hdfs dfs -put $input_path /test/
$ hdfs dfs -ls /test/
-rw-r--r--   1 ...  140 ... /test/name_age_salary.csv
```

并且我检查文件的内容：

```
$ hdfs dfs -cat /test/name_age_salary.csv
alex,60,18000
adel,40,45000
adel,50,77000
jane,40,52000
jane,60,81000
alex,50,62000
mary,50,92000
mary,60,63000
mary,40,55000
mary,40,55000
```

现在我们已经在 HDFS 中创建了一个文件，我们将读取它，并从其内容创建一个 DataFrame 和一个 RDD。

首先，我们读取 HDFS 文件，并创建一个带有默认列名 (`_c0, _c1, _c2`) 的 DataFrame。HDFS URI 的一般格式如下：

```
  hdfs://*<server>*:*<port>*/*<directories>*/*<filename>*

```

其中 *`<server>`* 是 NameNode 的主机名，*`<port>`* 是 NameNode 的端口号。

在本例中，我使用了安装在我的 MacBook 上的 Hadoop 实例；NameNode 是 `localhost`，端口号是 `9000`：

```
>>> uri = 'hdfs://localhost:9000/test/name_age_salary.csv'
>>> df = spark.read.csv(uri)
>>> df.show()
+----+---+-----+
| _c0|_c1|  _c2| ![1](img/1.png)
+----+---+-----+
|alex| 60|18000|
|adel| 40|45000|
|adel| 50|77000|
|jane| 40|52000|
|jane| 60|81000|
|alex| 50|62000|
|mary| 50|92000|
|mary| 60|63000|
|mary| 40|55000|
|mary| 40|55000|
+----+---+-----+
```

![1](img/#co_interacting_with_external_data_sources_CO30-1)

默认列名

让我们检查新创建的 DataFrame 的架构：

```
>>> df.printSchema
<bound method DataFrame.printSchema of DataFrame
[
 _c0: string,
 _c1: string,
 _c2: string
]>
```

如果您想在 DataFrame 上强制使用自己的显式模式（列名和数据类型），则可以这样做：

```
>>> from pyspark.sql.types import StructType
>>> from pyspark.sql.types import StructField
>>> from pyspark.sql.types import StringType
>>> from pyspark.sql.types import IntegerType
>>>
>>> empSchema = StructType( ![1
    StructField("name", StringType(), True),
    StructField("age", IntegerType(), True),
    StructField("salary", StringType(), True)
    ])
>>>
>>> uri = 'hdfs://localhost:9000/test/name_age_salary.csv'
>>> df2 = spark.read.csv(uri, schema = empSchema) ![2](img/2.png)

>>> df2.show()
+----+---+------+
|name|age|salary| ![3](img/3.png)
+----+---+------+
|alex| 60| 18000|
|adel| 40| 45000|
|adel| 50| 77000|
|jane| 40| 52000|
|jane| 60| 81000|
|alex| 50| 62000|
|mary| 50| 92000|
|mary| 60| 63000|
|mary| 40| 55000|
|mary| 40| 55000|
+----+---+------+

>>> df2.printSchema
<bound method DataFrame.printSchema of DataFrame
[
 name: string,
 age: int,
 salary: string
]>
```

![1](img/#co_interacting_with_external_data_sources_CO31-1)

显式模式定义

![2](img/#co_interacting_with_external_data_sources_CO31-2)

强制使用显式模式

![3](img/#co_interacting_with_external_data_sources_CO31-3)

显式列名

你还可以从 HDFS 文件中读取，并创建一个`RDD[String]`：

```
>>> rdd = spark.sparkContext.textFile(uri)

>>> rdd.collect()
[
 u'alex,60,18000',
 u'adel,40,45000',
 u'adel,50,77000',
 u'jane,40,52000',
 u'jane,60,81000',
 u'alex,50,62000',
 u'mary,50,92000',
 u'mary,60,63000',
 u'mary,40,55000',
 u'mary,40,55000'
]
```

## 写入 Hadoop 文本文件

PySpark 的 API 允许我们将 RDD 和 DataFrame 保存为 HDFS 文件。首先让我们看看如何将 RDD 保存到 HDFS 文件中：

```
>>> pairs = [('alex', 2), ('alex', 3),
             ('jane', 5), ('jane', 6)]
>>> rdd = spark.sparkContext.parallelize(pairs)
>>> rdd.collect()
[('alex', 2), ('alex', 3), ('jane', 5), ('jane', 6)]
>>> rdd.count()
4
>>> rdd.saveAsTextFile("hdfs://localhost:9000/test/pairs")
```

`RDD.saveAsTextFile(path)`方法将数据集的元素写入文本文件（或一组文本文件）到指定的目录中，可以是本地文件系统、HDFS 或其他任何 Hadoop 支持的文件系统。Spark 会调用每个元素的`toString()`方法将其转换为文件中的文本行。

接下来，让我们查看在 HDFS 中创建了什么（这里的输出格式已经适合页面）：

```
$ hdfs dfs -ls hdfs://localhost:9000/test/pairs
Found 9 items
-rw-r--r-- ...  0 ... hdfs://localhost:9000/test/pairs/_SUCCESS
-rw-r--r-- ...  0 ... hdfs://localhost:9000/test/pairs/part-00000
-rw-r--r-- ... 12 ... hdfs://localhost:9000/test/pairs/part-00001
...
-rw-r--r-- ... 12 ... hdfs://localhost:9000/test/pairs/part-00007

$ hdfs dfs -cat hdfs://localhost:9000/test/pairs/part*
(*alex*, 2)
(*alex*, 3)
(*jane*, 5)
(*jane*, 6)
```

我们得到了八个*part-*文件的原因是因为源 RDD 有八个分区：

```
>>> rdd.getNumPartitions()
8
```

如果你想创建一个单一的*part-*文件，那么你应该创建一个单一的 RDD 分区：

```
>>> rdd_single = spark.sparkContext.parallelize(pairs, 1)
>>> rdd_single.collect()
[('alex', 2), ('alex', 3), ('jane', 5), ('jane', 6)]
>>> rdd_single.getNumPartitions()
1
>>> rdd_single.saveAsTextFile("hdfs://localhost:9000/test/pairs_single")
```

让我们查看在 HDFS 行创建了什么：

```
$ hdfs dfs -ls hdfs://localhost:9000/test/pairs_single
Found 2 items
-rw-r--r--  0  hdfs://localhost:9000/test/pairs_single/_SUCCESS
-rw-r--r-- 48  hdfs://localhost:9000/test/pairs_single/part-00000

$ hdfs dfs -cat hdfs://localhost:9000/test/pairs_single/part-00000
(*alex*, 2)
(*alex*, 3)
(*jane*, 5)
(*jane*, 6)
```

我们可以通过使用`DataFrameWriter`将 DataFrame 保存到 HDFS 中：

```
>>> pairs = [('alex', 2), ('alex', 3),
             ('jane', 5), ('jane', 6)]
>>>
>>> pairsDF = spark.createDataFrame(pairs)
>>> pairsDF.show()
+----+---+
|  _1| _2|
+----+---+
|alex|  2|
|alex|  3|
|jane|  5|
|jane|  6|
+----+---+

>>> pairsDF.write.csv("hdfs://localhost:9000/test/pairs_df")
```

下面是在 HDFS 中创建的内容：

```
$ hdfs dfs -ls hdfs://localhost:9000/test/pairs_df
Found 9 items
-rw-... 0 hdfs://localhost:9000/test/pairs_df/_SUCCESS
-rw-... 0 hdfs://localhost:9000/test/pairs_df/part-00000-...-c000.csv
...
-rw-... 7 hdfs://localhost:9000/test/pairs_df/part-00007-...-c000.csv

$ hdfs dfs -cat hdfs://localhost:9000/test/pairs_df/part*
alex,2
alex,3
jane,5
jane,6
```

您可以将 DataFrame 以不同的数据格式保存到 HDFS 中。例如，要以 Parquet 格式保存 DataFrame，可以使用以下模板：

```
# df is an existing DataFrame object.
# format options are 'csv', 'parquet', 'json'
df.write.save(
   '/target/path/',
   format='parquet',
   mode='append'
)
```

## 读取和写入 HDFS SequenceFiles

Hadoop 支持持久化任何文件类型，包括在 HDFS 中的`SequenceFile`。`SequenceFile` 是由二进制（键，值）对组成的平面文件。Hadoop 将`SequenceFile`类定义为`org.apache.hadoop.io.SequenceFile`。`SequenceFile` 提供了用于写入、读取和排序的`SequenceFile.Writer`、`SequenceFile.Reader`和`SequenceFile.Sorter`类。`SequenceFile` 是 Hadoop 的标准二进制序列化格式。它存储`Writable`（键，值）对的记录，并支持分割和压缩。由于它们比文本文件更高效，因此`SequenceFile` 在 MapReduce 管道中常用于中间数据存储。

### 读取 HDFS SequenceFiles

Spark 支持使用`SparkContext.sequenceFile()`方法读取`SequenceFile`。例如，在 Python 中读取具有`Text`键和`DoubleWritable`值的`SequenceFile`，我们将执行以下操作：

```
# spark: an instance of SparkSession
rdd = spark.sparkContext.sequenceFile(path)
```

请注意，与 Java 或 Scala 不同，我们不会将（键，值）对的数据类型传递给 Spark API；Spark 会自动将 Hadoop 的`Text`转换为`String`，将`DoubleWritable`转换为`Double`。

### 写入 HDFS SequenceFiles

PySpark 的`RDD.saveAsSequenceFile()`方法允许用户将（键，值）对的 RDD 保存为`SequenceFile`。例如，我们可以从 Python 集合创建一个 RDD，并将其保存为`SequenceFile`，如下所示：

```
# spark: an instance of SparkSession
pairs = [('key1', 10.0), ('key2', 20.0),
         ('key3', 30.0), ('key4', 40.0)]
rdd = spark.sparkContext.parallelize(pairs)
rdd.saveAsSequenceFile('/tmp/sequencefile/')
```

然后我们可以读取新创建的`SequenceFile`，并将其转换为（键，值）对的 RDD：

```
# spark: an instance of SparkSession
rdd2 = spark.sparkContext.sequenceFile('/tmp/sequencefile/')
rdd2.collect()
[(u'key1', 10.0),
 (u'key2', 20.0),
 (u'key3', 30.0),
 (u'key4', 40.0)
]
```

# 读写 Parquet 文件

[Parquet](https://parquet.apache.org) 是许多数据处理系统支持的列式数据格式。它是自描述的（包含元数据）、与语言无关的，并且非常适合快速分析。

Spark SQL 在读取和写入 Parquet 文件时提供支持，同时自动保留原始数据的模式。在写入 Parquet 文件时，所有列都会自动转换为可为空，以确保兼容性。

图 7-3 描述了一个逻辑表及其相关的行和列布局。

![daws 0703](img/daws_0703.png)

###### 图 7-3\. 带有行布局和列布局的逻辑表

## 写入 Parquet 文件

在本节中，我将向您展示如何使用 PySpark API 将 JSON 文件读取为 DataFrame，然后将其保存为 Parquet 文件。假设我们有以下 JSON 文件：

```
$ cat examples/src/main/resources/employees.json
{"name":"Michael", "salary":3000}
{"name":"Andy", "salary":4500}
{"name":"Justin", "salary":3500}
{"name":"Berta", "salary":4000}
```

使用 `DataFrameReader`，我们将 JSON 文件读取为 DataFrame 对象，命名为 `peopleDF`：

```
>>> input_path = "examples/src/main/resources/employees.json"
>>> peopleDF = spark.read.json(input_path)
>>> peopleDF.show()
+-------+------+
|   name|salary|
+-------+------+
|Michael|  3000|
|   Andy|  4500|
| Justin|  3500|
|  Berta|  4000|
+-------+------+

>>> peopleDF.printSchema()
root
 |-- name: string (nullable = true)
 |-- salary: long (nullable = true)
```

然后我们可以将其保存为 Parquet 文件，保留模式信息：

```
>>> peopleDF.write.parquet("file:///tmp/people.parquet")
```

您可以检查目录内容以查看生成的 Parquet 文件：

```
$ ls -l /tmp/people.parquet/
-rw-r--r-- ...   0 Apr 30 15:06 _SUCCESS
-rw-r--r-- ... 634 Apr 30 15:06 part-00000-...-c000.snappy.parquet
```

为了测试和调试目的，您可以从 Python 集合创建 Parquet 文件：

```
>>> tuples =  [("alex", "Math", 97),
               ("jane", "Econ", 82),
               ("jane", "Math", 99)]
>>> column_names = ["name", "subject", "grade"]
>>> df = spark.createDataFrame(tuples, column_names) ![1](img/1.png)
>>> df.show()
+----+-------+-----+
|name|subject|grade|
+----+-------+-----+
|alex|   Math|   97|
|jane|   Econ|   82|
|jane|   Math|   99|
+----+-------+-----+

>>> df.write.parquet("file:///tmp/parquet") ![2](img/2.png)
```

![1](img/#co_interacting_with_external_data_sources_CO32-1)

将您的 Python 集合转换为 DataFrame。

![2](img/#co_interacting_with_external_data_sources_CO32-2)

将您的 DataFrame 保存为一组 Parquet 文件。

同样，您可以检查目录以查看创建的 Parquet 文件：

```
$ ls -1 /tmp/parquet
_SUCCESS
part-00000-...-c000.snappy.parquet
part-00002-...-c000.snappy.parquet
part-00005-...-c000.snappy.parquet
part-00007-...-c000.snappy.parquet
```

## 读取 Parquet 文件

在这一节中，使用 PySpark，我们将读取刚刚创建的 Parquet 文件。注意，Parquet 文件是自描述的，因此模式得以保留。加载 Parquet 文件的结果是一个 DataFrame：

```
>>> input_path = "file:///tmp/people.parquet"
>>> parquetFile = spark.read.parquet(input_path)
>>> parquetFile.show()
+-------+------+
|   name|salary|
+-------+------+
|Michael|  3000|
|   Andy|  4500|
| Justin|  3500|
|  Berta|  4000|
+-------+------+

>>> parquetFile.printSchema()
root
 |-- name: string (nullable = true)
 |-- salary: long (nullable = true)
```

Parquet 文件也可以用来创建临时视图，然后在 SQL 语句中使用：

```
>>> parquetFile.createOrReplaceTempView("parquet_table") ![1](img/1.png)
>>> query = "SELECT name, salary FROM parquet_table WHERE salary > 3800"
>>> filtered = spark.sql(query)
>>> filtered.show()
+-----+------+
| name|salary|
+-----+------+
| Andy|  4500|
|Berta|  4000|
+-----+------+
```

![1](img/#co_interacting_with_external_data_sources_CO33-1)

`parquet_table` 充当关系表。

Parquet 支持集合数据类型，包括数组类型。以下示例读取使用数组的 Parquet 文件：

```
>>> parquet_file = "examples/src/main/resources/users.parquet"
>>> usersDF = spark.read.parquet(parquet_file)
>>> users.show()
+------+--------------+----------------+
|  name|favorite_color|favorite_numbers|
+------+--------------+----------------+
|Alyssa|          null|  [3, 9, 15, 20]|
|   Ben|           red|              []|
+------+--------------+----------------+

>>> usersDF.printSchema()
root
 |-- name: string (nullable = true)
 |-- favorite_color: string (nullable = true)
 |-- favorite_numbers: array (nullable = true)
 |    |-- element: integer (containsNull = true)
```

# 读取和写入 Avro 文件

[Apache Avro](https://avro.apache.org) 是一种语言中立的数据序列化系统。它将数据定义存储为 JSON 格式，易于读取和解释，而数据本身则以紧凑高效的二进制格式存储。Avro 文件包含标记，可用于将大型数据集分割成适合 MapReduce 处理的子集。Avro 是一种非常快速的序列化格式。

## 读取 Avro 文件

使用 PySpark，我们可以读取 Avro 文件并按如下方式创建一个关联的 DataFrame：

```
$ pyspark --packages org.apache.spark:spark-avro_2.11:2.4.0 ![1](img/1.png)
SparkSession available as *spark*.
```

```
>>> path = "/book/code/chap08/twitter.avro"
>>> df = spark.read.format("avro").load(path) ![2](img/2.png)
>>> df.show(truncate=False)
+----------+-----------------------------------+----------+
|username  |tweet                              |timestamp |
+----------+-----------------------------------+----------+
|miguno    |Rock: Nerf paper, scissors is fine.|1366150681|
|BlizzardCS|Works as intended.  Terran is IMBA.|1366154481|
+----------+-----------------------------------+----------+
```

![1](img/#co_interacting_with_external_data_sources_CO34-1)

要读取/写入 Avro 文件，您必须导入所需的 Avro 库；`spark-avro` 模块是外部的，不包含在 `spark-submit` 或 `pyspark` 的默认设置中。

![2](img/#co_interacting_with_external_data_sources_CO34-2)

读取 Avro 文件并创建一个 DataFrame。

## 写入 Avro 文件

从 DataFrame 创建 Avro 文件同样简单。这里，我们将使用上一节创建的 DataFrame，将其保存为 Avro 文件，然后再将其读回作为 DataFrame：

```
$ pyspark --packages org.apache.spark:spark-avro_2.11:2.4.0 ![1](img/1.png)
SparkSession available as *spark*.
```

```
>>># df : DataFrame (created in previous section)
>>> output_path = "/tmp/avro/mytweets.avro"
>>> df.select("username", "tweet")
      .write.format("avro")
      .save(output_path) ![2](img/2.png)
>>> df2 = spark.read.format("avro").load(outputPath) ![3](img/3.png)
>>> df2.show(truncate=False)
+----------+-----------------------------------+
|username  |tweet                              |
+----------+-----------------------------------+
|miguno    |Rock: Nerf paper, scissors is fine.|
|BlizzardCS|Works as intended.  Terran is IMBA.|
+----------+-----------------------------------+
```

![1](img/#co_interacting_with_external_data_sources_CO35-1)

导入所需的 Avro 库。

![2](img/#co_interacting_with_external_data_sources_CO35-2)

创建一个 Avro 文件。

![3](img/#co_interacting_with_external_data_sources_CO35-3)

从新的 Avro 文件创建一个 DataFrame。

# 从 MS SQL Server 读取和写入

[微软 SQL Server](https://oreil.ly/2JckS) 是微软的关系型数据库管理系统，设计和构建用于将信息以记录形式存储在关系表中。

## 写入 MS SQL Server

以下示例展示了如何将 DataFrame (`df`) 写入新的 SQL Server 表中：

```
# define database URL
server_name = "jdbc:sqlserver://{SERVER_ADDRESS}"
database_name = "my_database_name"
url = server_name + ";" + "databaseName=" + database_name + ";"
# define table name and username/password
table_name = "my_table_name"
username = "my_username"
password = "my_password"

try:
  df.write \
    .format("com.microsoft.sqlserver.jdbc.spark") \ ![1](img/1.png)
    .mode("overwrite") \ ![2](img/2.png)
    .option("url", url) \
    .option("dbtable", table_name) \
    .option("user", username) \
    .option("password", password) \
    .save()
except ValueError as error :
    print("Connector write failed", error)
```

![1](img/#co_interacting_with_external_data_sources_CO36-1)

包含此类的 JAR 文件必须在你的 `CLASSPATH` 中。

![2](img/#co_interacting_with_external_data_sources_CO36-2)

默认情况下，`overwrite` 模式会先删除数据库中已存在的表。

要将 DataFrame 的行追加到现有表中，只需用 `mode("append")` 替换 `mode("overwrite")`。

注意，Spark 的 MS SQL 连接器在执行数据库的批量插入时，默认使用 `READ_COMMITTED` 隔离级别。如果希望覆盖隔离级别，请使用此 `mssqlIsolationLevel` 选项，如下所示：

```
    .option("mssqlIsolationLevel", "READ_UNCOMMITTED")
```

## 从 MS SQL Server 读取

要从现有的 SQL Server 表中读取，可以使用以下代码片段作为模板：

```
jdbc_df = spark.read \
        .format("com.microsoft.sqlserver.jdbc.spark") \
        .option("url", url) \
        .option("dbtable", table_name) \
        .option("user", username) \
        .option("password", password)\
        .load()
```

# 读取图像文件

Spark 2.4.0.+ 能够读取二进制数据，在许多机器学习应用中非常有用（如人脸识别和逻辑回归）。Spark 可以从目录中加载图像文件到 DataFrame，通过 Java 库中的 `ImageIO` 将压缩图像（*.jpg*, *.png* 等）转换为原始图像表示。加载的 *DataFrame* 有一个 `StructType` 列 `"image"`，包含存储为图像模式的图像数据。

## 从图像创建一个 DataFrame。

假设我们在一个目录中有以下图像：

```
$ ls -l chap07/images
-rw-r--r--@ ...  27295 Feb  3 10:55 cat1.jpg
-rw-r--r--@ ...  35914 Feb  3 10:55 cat2.jpg
-rw-r--r--@ ...  26354 Feb  3 10:55 cat3.jpg
-rw-r--r--@ ...  30432 Feb  3 10:55 cat4.jpg
-rw-r--r--@ ...   6641 Feb  3 10:53 duck1.jpg
-rw-r--r--@ ...  11621 Feb  3 10:54 duck2.jpg
-rw-r--r--@ ...     13 Feb  3 10:55 not-image.txt ![1](img/1.png)
```

![1](img/#co_interacting_with_external_data_sources_CO37-1)

不是图像

我们可以将所有图像加载到 DataFrame 中，并忽略任何不是图像的文件，如下所示：

```
>>> images_path = '/book/code/chap07/images'
>>> df = spark.read
               .format("image") ![1](img/1.png)
               .option("dropInvalid", "true") ![2](img/2.png)
               .load(images_path) ![3](img/3.png)
>>> df.count()
6
```

![1](img/#co_interacting_with_external_data_sources_CO38-1)

`format` 必须是 `"image"`。

![2](img/#co_interacting_with_external_data_sources_CO38-2)

删除/忽略非图像文件。

![3](img/#co_interacting_with_external_data_sources_CO38-3)

加载图像并创建 DataFrame。

让我们检查 DataFrame 的模式：

```
>>> df.printSchema()
root
 |-- image: struct (nullable = true)
 |    |-- origin: string (nullable = true) ![1](img/1.png)
 |    |-- height: integer (nullable = true) ![2](img/2.png)
 |    |-- width: integer (nullable = true) ![3](img/3.png)
 |    |-- nChannels: integer (nullable = true) ![4](img/4.png)
 |    |-- mode: integer (nullable = true) ![5](img/5.png)
 |    |-- data: binary (nullable = true) ![6](img/6.png)
```

![1](img/#co_interacting_with_external_data_sources_CO39-1)

图像的文件路径

![2](img/#co_interacting_with_external_data_sources_CO39-2)

图像的高度

![3](img/#co_interacting_with_external_data_sources_CO39-3)

图像的宽度

![4](img/#co_interacting_with_external_data_sources_CO39-4)

图像通道数

![5](img/#co_interacting_with_external_data_sources_CO39-5)

兼容 OpenCV 的类型

![6](img/#co_interacting_with_external_data_sources_CO39-6)

图像字节

现在，让我们来看看创建的图像 DataFrame 中的一些列：

```
>>> df.select("image.origin", "image.width", "image.height")
      .show(truncate=False)
+--------------------------+-----+------+
|origin                    |width|height|
+--------------------------+-----+------+
|file:///book/.../cat2.jpg |300  |311   |
|file:///book/.../cat4.jpg |199  |313   |
|file:///book/.../cat1.jpg |300  |200   |
|file:///book/.../cat3.jpg |300  |296   |
|file:///book/.../duck2.jpg|275  |183   |
|file:///book/.../duck1.jpg|227  |222   |
+--------------------------+-----+------+
```

# 摘要

总结一下：

+   读写数据是数据算法的一个重要部分。根据项目和数据需求，应谨慎选择数据源。

+   Spark DataSource API 提供了一个可插拔的机制，用于通过 Spark SQL 访问结构化数据。数据源不仅可以是简单的管道，用于转换数据并将其拉入 Spark。

+   Spark SQL 支持从现有的关系型数据库表、Apache Hive 表、列式存储格式如 Parquet 和 ORC，以及行式存储格式如 Avro 中读取数据。Spark 提供了一个简单的 API 用于与所有 JDBC 兼容的关系型数据库、Amazon S3、HDFS 等集成。您还可以轻松地从文本、CSV 和 JSON 文件等外部数据源中读取和保存数据。
