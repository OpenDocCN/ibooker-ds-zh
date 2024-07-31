# 第六章：转换数据

在定义于第三章中的 ELT 模式中，一旦数据被摄入到数据湖或数据仓库中（见第四章），管道中的下一步是数据转换。数据转换可以包括对数据的非上下文操纵以及考虑业务上下文和逻辑建模的数据。 

如果管道的目的是生成业务洞察或分析，则除了任何非上下文转换之外，数据还会进一步转换为数据模型。 从第二章回想起，数据模型以数据仓库中的一种或多种表形式来表示并定义数据，以便于数据分析。

尽管数据工程师有时在管道中构建非上下文转换，但现在数据分析师和分析工程师处理绝大部分数据转换已成为典型做法。 由于 ELT 模式的出现（他们在仓库中就有所需的数据！）以及支持以 SQL 为主要语言设计的工具和框架，这些角色比以往任何时候都更有权力。

本章既探讨了几乎每个数据管道都常见的非上下文转换，也讨论了用于仪表板、报告和业务问题一次性分析的数据模型。 由于 SQL 是数据分析师和分析工程师的语言，因此大多数转换代码示例都是用 SQL 编写的。 我包括了一些用 Python 编写的示例，以说明何时将非上下文转换紧密耦合到使用强大的 Python 库的数据摄入中是合理的。

就像第四章和五章中的数据摄取一样，代码样例极为简化，并作为管道中更复杂转换的起点。 要了解如何运行和管理转换与管道中其他步骤之间的依赖关系，请参见第八章。

# 非上下文转换

在第三章中，我简要提到了 EtLT 子模式的存在，其中小写字母*t*代表某些非上下文数据转换，例如以下内容：

+   在表中去重记录

+   将 URL 参数解析为单独的组件

虽然有无数例子，但通过提供这些转换的代码示例，我希望涵盖一些非上下文转换的常见模式。 下一节讨论何时以数据摄入（EtLT）和后摄入（ELT）的一部分执行这些转换是合理的。 

## 在表中去重记录

尽管不是理想的，但在数据被摄入到数据仓库的表中存在重复记录是可能的。 出现这种情况有很多原因：

+   增量数据摄取误以前摄取时间窗口重叠，并提取了在先前运行中已摄取的一些记录。

+   在源系统中无意中创建了重复记录。

+   在数据回填时，与后续加载到表中的数据重叠。

不管原因是什么，检查和删除重复记录最好使用 SQL 查询来执行。以下每个 SQL 查询都涉及到数据库中的`Orders`表，如表 6-1 所示。该表包含五条记录，其中两条是重复的。虽然对于`OrderId`为 1 的记录有三条，但第二行和第四行完全相同。本示例的目标是识别这种重复并解决它。虽然本示例有两条完全相同的记录，但以下代码示例的逻辑在表中有三条、四条甚至更多相同记录时也是有效的。

表 6-1\. 带有重复记录的订单表

| OrderId | OrderStatus | LastUpdated |
| --- | --- | --- |
| 1 | 缺货 | 2020-06-01 |
| 1 | 已发货 | 2020-06-09 |
| 2 | 已发货 | 2020-07-11 |
| 1 | 已发货 | 2020-06-09 |
| 3 | 已发货 | 2020-07-12 |

如果您想要创建一个用于示例 6-1 和 6-2 的`Orders`表，以下是可以使用的 SQL：

```
CREATE TABLE Orders (
  OrderId int,
  OrderStatus varchar(30),
  LastUpdated timestamp
);

INSERT INTO Orders
  VALUES(1,'Backordered', '2020-06-01');
INSERT INTO Orders
  VALUES(1,'Shipped', '2020-06-09');
INSERT INTO Orders
  VALUES(2,'Shipped', '2020-07-11');
INSERT INTO Orders
  VALUES(1,'Shipped', '2020-06-09');
INSERT INTO Orders
  VALUES(3,'Shipped', '2020-07-12');
```

简单地识别表中的重复记录。您可以使用 SQL 中的`GROUP BY`和`HAVING`语句。以下查询返回任何重复记录以及它们的数量：

```
SELECT OrderId,
  OrderStatus,
  LastUpdated,
  COUNT(*) AS dup_count
FROM Orders
GROUP BY OrderId, OrderStatus, LastUpdated
HAVING COUNT(*) > 1;
```

当执行时，查询返回以下结果：

```
OrderId | OrderStatus | LastUpdated | dup_count
1       | Shipped     | 2020-06-09  | 2
```

现在您知道至少存在一个重复记录，可以删除这些重复记录。我将介绍两种方法来实现。您选择的方法取决于与数据库优化相关的许多因素以及您对 SQL 语法的偏好。我建议尝试两种方法并比较运行时间。

第一种方法是使用一系列查询。第一个查询使用`DISTINCT`语句从原始表创建表的副本。第一个查询的结果集只有四行，因为两个重复的行被`DISTINCT`转换为一个行。接下来，原始表被截断。最后，去重后的数据集被插入到原始表中，如示例 6-1 所示。

##### 示例 6-1\. distinct_orders_1.sql

```
CREATE TABLE distinct_orders AS
SELECT DISTINCT OrderId,
  OrderStatus,
  LastUpdated
FROM ORDERS;

TRUNCATE TABLE Orders;

INSERT INTO Orders
SELECT * FROM distinct_orders;

DROP TABLE distinct_orders;
```

###### 警告

在对`Orders`表执行`TRUNCATE`操作后，直到完成以下`INSERT`操作之前，该表将为空。在此期间，`Orders`表为空，基本上不可由任何查询它的用户或进程访问。虽然`INSERT`操作可能不会花费太多时间，但对于非常大的表，您可能考虑删除`Orders`表，然后将`distinct_orders`重命名为`Orders`。

另一种方法是使用*窗口函数*对重复行进行分组，并分配行号以标识哪些行应删除，哪些行应保留。我将使用`ROW_NUMBER`函数对记录进行排名，并使用`PARTITION BY`语句按每列分组记录。通过这样做，任何具有多个匹配项（我们的重复项）的记录将被分配一个大于 1 的`ROW_NUMBER`。

如果您执行了示例 6-1，请确保使用本节早期的`INSERT`语句刷新`Orders`表，使其再次包含表 6-1 中显示的内容。您将希望有一个重复行用于以下示例！

当在`Orders`表上运行这样的查询时，会发生以下情况：

```
SELECT OrderId,
  OrderStatus,
  LastUpdated,
  ROW_NUMBER() OVER(PARTITION BY OrderId,
                    OrderStatus,
                    LastUpdated)
    AS dup_count
FROM Orders;
```

查询的结果如下：

```
orderid | orderstatus | lastupdated  |  dup_count
---------+-------------+-------------------+-----
      1 | Backordered | 2020-06-01   |     1
      1 | Shipped     | 2020-06-09   |     1
      1 | Shipped     | 2020-06-09   |     2
      2 | Shipped     | 2020-07-11   |     1
      3 | Shipped     | 2020-07-12   |     1
```

如您所见，结果集中的第三行具有`dup_count`值为`2`，因为它与其上方的记录是重复的。现在，就像第一种方法一样，您可以创建一个包含去重记录的表，截断`Orders`表，最后将清理后的数据集插入`Orders`中。示例 6-2 显示了完整的源代码。

##### 示例 6-2\. distinct_orders_2.sql

```
CREATE TABLE all_orders AS
SELECT
  OrderId,
  OrderStatus,
  LastUpdated,
  ROW_NUMBER() OVER(PARTITION BY OrderId,
                    OrderStatus,
                    LastUpdated)
    AS dup_count
FROM Orders;

TRUNCATE TABLE Orders;

-- only insert non-duplicated records
INSERT INTO Orders
  (OrderId, OrderStatus, LastUpdated)
SELECT
  OrderId,
  OrderStatus,
  LastUpdated
FROM all_orders
WHERE
  dup_count = 1;

DROP TABLE all_orders;
```

无论采用哪种方法，结果都是`Orders`表的去重版本，如表 6-2 所示。

表 6-2\. Orders 表（无重复项）

| OrderId | OrderStatus | LastUpdated |
| --- | --- | --- |
| 1 | 后订购 | 2020-06-01 |
| 1 | 已发货 | 2020-06-09 |
| 2 | 已发货 | 2020-07-11 |
| 3 | 已发货 | 2020-07-12 |

## 解析 URL

解析 URL 片段是一个与业务背景几乎无关的任务。有许多 URL 组件可以在转换步骤中解析，并存储在数据库表的各个列中。

例如，请考虑以下 URL：

> *https://www.mydomain.com/page-name?utm_content=textlink&utm_medium=social&utm_source=twitter&utm_campaign=fallsale*

有六个有价值且可以解析并存储为单独列的组件：

+   域名：*www.domain.com*

+   URL 路径：*/page-name*

+   utm_content 参数值：*textlink*

+   utm_medium 参数值：*social*

+   utm_source 参数值：*twitter*

+   utm_campaign 参数值：*fallsale*

解析 URL 可以在 SQL 和 Python 中进行。在运行转换时和 URL 存储的位置将指导您决定使用哪种语言。例如，如果您遵循 EtLT 模式并且可以在从源提取后但加载到数据仓库表之前解析 URL，则 Python 是一个非常好的选择。我将首先提供 Python 示例，然后是 SQL。

首先，使用`pip`安装`urllib3` Python 库。（有关 Python 配置说明，请参见“设置 Python 环境”）：

```
(env) $ pip install urllib3
```

接下来，使用`urlsplit`和`parse_qs`函数解析 URL 的相关组件。在下面的代码示例中，我这样做并打印出结果：

```
from urllib.parse import urlsplit, parse_qs

url = """https://www.mydomain.com/page-name?utm_content=textlink&utm_medium=social&utm_source=twitter&utm_campaign=fallsale"""

split_url = urlsplit(url)
params = parse_qs(split_url.query)

# domain
print(split_url.netloc)

# url path
print(split_url.path)

# utm parameters
print(params['utm_content'][0])
print(params['utm_medium'][0])
print(params['utm_source'][0])
print(params['utm_campaign'][0])
```

当执行时，代码示例会产生以下结果：

```
www.mydomain.com
/page-name
textlink
social
twitter
fallsale
```

就像第四章和第五章中的数据摄入代码示例一样，你也可以解析并将每个参数写入 CSV 文件，以加载到数据仓库中完成摄入。示例 6-3 包含了完成此操作的代码示例，但你可能需要迭代处理多个 URL！

##### 示例 6-3\. url_parse.sql

```
from urllib.parse import urlsplit, parse_qs
import csv

url = """https://www.mydomain.com/page-name?utm_content=textlink&utm_medium=social&utm_source=twitter&utm_campaign=fallsale"""

split_url = urlsplit(url)
params = parse_qs(split_url.query)
parsed_url = []
all_urls = []

# domain
parsed_url.append(split_url.netloc)

# url path
parsed_url.append(split_url.path)

parsed_url.append(params['utm_content'][0])
parsed_url.append(params['utm_medium'][0])
parsed_url.append(params['utm_source'][0])
parsed_url.append(params['utm_campaign'][0])

all_urls.append(parsed_url)

export_file = "export_file.csv"

with open(export_file, 'w') as fp:
	csvw = csv.writer(fp, delimiter='|')
	csvw.writerows(all_urls)

fp.close()
```

如果你需要解析已加载到数据仓库中的 URL，使用 SQL 可能会更具挑战性。虽然一些数据仓库供应商提供解析 URL 的函数，但其他则没有。例如，Snowflake 提供了一个名为`PARSE_URL`的函数，将 URL 解析为其组件，并将结果作为 JSON 对象返回。例如，如果你想解析前面示例中的 URL，结果将如下所示：

```
SELECT parse_url('https://www.mydomain.com/page-name?utm_content=textlink&utm_medium=social&utm_source=twitter&utm_campaign=fallsale');
```

```
+-----------------------------------------------------------------+
| PARSE_URL('https://www.mydomain.com/page-name?utm_content=textlink&utm_medium=social&utm_source=twitter&utm_campaign=fallsale') |
|-----------------------------------------------------------------|
| {                               |
|   "fragment": null,             |
|   "host": "www.mydomain.com",   |
|   "parameters": {               |
|     "utm_content": "textlink",  |
|     "utm_medium": "social",   |
|     "utm_source": "twitter",    |
|     "utm_campaign": "fallsale"  |
|   },                            |
|   "path": "/page-name",         |
|   "query": "utm_content=textlink&utm_medium=social&utm_source=twitter&utm_campaign=fallsale",                                            |
|   "scheme": "HTTPS"              |
| }                                |
+-----------------------------------------------------------------+
```

如果你正在使用 Redshift 或其他没有内置 URL 解析功能的数据仓库平台，你需要使用自定义字符串解析或正则表达式。例如，Redshift 有一个名为`REGEXP_SUBSTR`的函数。考虑到大多数数据仓库中解析 URL 的难度，我建议在数据摄入时使用 Python 或其他语言进行解析，并加载结构化的 URL 组件。

# 何时进行转换？在摄入期间还是之后？

从技术角度来看，不像前面部分中没有业务背景的数据转换可以在数据摄入期间或之后运行。然而，有些原因你应该考虑将它们作为摄入过程的一部分运行（EtLT 模式）：

1.  *使用除 SQL 以外的语言进行转换最简单*：就像在前面的示例中解析 URL 一样，如果你发现使用 Python 库处理转换要容易得多，那么作为数据摄入的一部分就这样做吧。在 ELT 模式中，摄入后的转换仅限于数据建模，由通常在 SQL 中最为熟悉的数据分析师执行。

1.  *该转换正在解决数据质量问题*：最好尽早在管道中解决数据质量问题（第九章更详细地讨论了这个主题）。例如，在前面的部分中，我提供了一个识别和删除已摄入数据中重复记录的示例。如果可以在摄入点捕捉并修复重复数据，就没有理由让数据分析师因为重复数据而遭受困扰。尽管该转换是用 SQL 编写的，但可以在摄入的最后阶段运行，而不必等待分析师转换数据。

当涉及到包含业务逻辑的转换时，最好将其与数据摄入分开。正如你将在下一节看到的，这种类型的转换被称为*数据建模*。

# 数据建模基础

为了分析、仪表板和报告使用而建模的数据是一个值得专门撰写一本书的主题。但是，在 ELT 模式中建模数据的一些原则我在本节中进行了讨论。

不同于前一节，数据建模是 ELT 模式管道中的转换步骤考虑业务上下文的地方。数据模型理清了从各种来源在提取和加载步骤（数据摄入）中加载到仓库中的所有数据。

## 关键数据建模术语

在本节中，当我使用术语*数据模型*时，我指的是数据仓库中的单个 SQL 表。在样本数据模型中，我将专注于模型的两个属性：

度量

这些是你想要衡量的事物！例如客户数量和收入的金额。

属性

这些是你想在报告或仪表板中进行过滤或分组的内容。例如日期、客户名称和国家。

此外，我将谈到数据模型的粒度。*粒度*是存储在数据模型中的详细级别。例如，一个必须提供每天下订单数量的模型需要日粒度。如果它必须回答每小时下订单数量的问题，那么它需要小时粒度。

*源表*是通过数据摄入（如第四章和第五章描述）加载到数据仓库或数据湖中的表。在数据建模中，模型是从源表以及其他数据模型构建的。

## 完全刷新数据建模

当建模已完全重新加载的数据时，比如在“从 MySQL 数据库中提取数据”中描述的情况下，你会遇到包含源数据存储的最新状态的表（或多个表）。例如，表 6-3 展示了类似于表 6-2 中的`Orders`表的记录，但只包含最新的记录，而不是完整的历史记录。请注意，`OrderId`为 1 的`Backordered`记录在此版本中不存在。这就是如果从源数据库完整加载到数据仓库中，表看起来像源系统中的`Orders`表当前状态的原因。

与表 6-2 的其他差异是第四列名为`CustomerId`，存储下订单客户的标识符，以及第五列`OrderTotal`，即订单的金额。

表 6-3\. 完全刷新的 Orders 表

| OrderId | OrderStatus | OrderDate | CustomerId | OrderTotal |
| --- | --- | --- | --- | --- |
| 1 | 已发货 | 2020-06-09 | 100 | 50.05 |
| 2 | 已发货 | 2020-07-11 | 101 | 57.45 |
| 3 | 已发货 | 2020-07-12 | 102 | 135.99 |
| 4 | 已发货 | 2020-07-12 | 100 | 43.00 |

除了`Orders`表之外，还要考虑已在仓库中完全加载的`Customers`表，显示在表 6-4 中（即包含每个客户记录的当前状态）。

表 6-4\. 完全刷新的 Customers 表

| CustomerId | CustomerName | CustomerCountry |
| --- | --- | --- |
| 100 | Jane | 美国 |
| 101 | Bob | 英国 |
| 102 | Miles | 英国 |

如果您希望在接下来的几节中在数据库中创建这些表格，您可以使用以下 SQL 语句来执行。请注意，如果您创建了“在表中去重记录”版本的`Orders`表，您需要首先进行`DROP`操作。

```
CREATE TABLE Orders (
  OrderId int,
  OrderStatus varchar(30),
  OrderDate timestamp,
  CustomerId int,
  OrderTotal numeric
);

INSERT INTO Orders
  VALUES(1,'Shipped','2020-06-09',100,50.05);
INSERT INTO Orders
  VALUES(2,'Shipped','2020-07-11',101,57.45);
INSERT INTO Orders
  VALUES(3,'Shipped','2020-07-12',102,135.99);
INSERT INTO Orders
  VALUES(4,'Shipped','2020-07-12',100,43.00);

CREATE TABLE Customers
(
  CustomerId int,
  CustomerName varchar(20),
  CustomerCountry varchar(10)
);

INSERT INTO Customers VALUES(100,'Jane','USA');
INSERT INTO Customers VALUES(101,'Bob','UK');
INSERT INTO Customers VALUES(102,'Miles','UK');
```

考虑需要创建一个数据模型，以便查询来回答以下问题：

+   某个国家在某个月份下的订单生成了多少收入？

+   在给定的一天中有多少订单？

虽然示例表仅包含少量记录，但请想象一下，如果两个表包含数百万条记录的情况。虽然使用 SQL 查询回答这些问题非常直接，但当数据量较大时，通过在某种程度上对数据模型进行聚合可以减少查询执行时间和模型中的数据量。

如果这些问题是数据模型的唯二要求，它必须提供两个措施：

+   总收入

+   订单计数

此外，模型必须允许根据以下两个属性对数据进行过滤或分组查询：

+   订单国家

+   订单日期

最后，模型的粒度是按日，因为需求中的最小时间单位是按日。

在这个高度简化的数据模型中，我将首先定义模型的结构（一个 SQL 表），然后插入从两个表连接源的数据：

```
CREATE TABLE IF NOT EXISTS order_summary_daily (
order_date date,
order_country varchar(10),
total_revenue numeric,
order_count int
);

INSERT INTO order_summary_daily
  (order_date, order_country,
  total_revenue, order_count)
SELECT
  o.OrderDate AS order_date,
  c.CustomerCountry AS order_country,
  SUM(o.OrderTotal) as total_revenue,
  COUNT(o.OrderId) AS order_count
FROM Orders o
INNER JOIN Customers c on
  c.CustomerId = o.CustomerId
GROUP BY o.OrderDate, c.CustomerCountry;
```

现在，您可以查询模型来回答需求中列出的问题：

```
-- How much revenue was generated from orders placed from a given country in a given month?

SELECT
  DATE_PART('month', order_date) as order_month,
  order_country,
  SUM(total_revenue) as order_revenue
FROM order_summary_daily
GROUP BY
  DATE_PART('month', order_date),
  order_country
ORDER BY
  DATE_PART('month', order_date),
  order_country;
```

使用表 6-3 和 6-4 中的示例数据，查询返回以下结果：

```
order_month | order_country | order_revenue
-------------+---------------+---------------
          6 | USA           |         50.05
          7 | UK            |        193.44
          7 | USA           |         43.00
(3 rows)
```

```
-- How many orders were placed on a given day?
SELECT
  order_date,
  SUM(order_count) as total_orders
FROM order_summary_daily
GROUP BY order_date
ORDER BY order_date;
```

返回以下结果：

```
order_date | total_orders
------------+--------------
2020-06-09 |            1
2020-07-11 |            1
2020-07-12 |            2
(3 rows)
```

## 完全刷新数据的慢变化维度

因为完全刷新的数据（如`Customers`中的记录）会覆盖现有数据的更改，通常会实现更高级的数据建模概念以跟踪历史更改。

例如，在下一节中，您将使用已增量加载的`Customers`表格，并包含对客户号为 100 的更新。正如您将在 表 6-6 中看到的那样，该客户有第二条记录表明她的`CustomerCountry`值于 2020-06-20 从“美国”更改为“英国”。这意味着她在 2020-07-12 下订单 4 时不再居住在美国。

在分析订单历史时，分析师可能希望将客户的订单分配到订单时所居住的地方。使用增量刷新数据，这样做稍微容易一些，如下一节所示。对于完全刷新的数据，需要在每次摄入之间保留`Customers`表格的完整历史记录，并自行跟踪这些更改。

这种方法在 Kimball（维度）建模中定义，并称为*慢变化维度*或*SCD*。在处理完全刷新的数据时，我经常使用 II 型 SCD，为实体的每次更改向表格中添加新记录，包括记录有效的日期范围。

简单来说，简的客户记录的 II 型 SCD 如 表 6-5 所示。请注意，最新记录的过期日期非常遥远。一些 II 型 SCD 使用 NULL 表示未过期记录，但远未来的日期使得查询表格时出错的可能性稍低，稍后您将看到。

表 6-5\. 具有客户数据的 II 型 SCD

| CustomerId | CustomerName | CustomerCountry | ValidFrom | Expired |
| --- | --- | --- | --- | --- |
| 100 | Jane | 美国 | 2019-05-01 7:01:10 | 2020-06-20 8:15:34 |
| 100 | Jane | 英国 | 2020-06-20 8:15:34 | 2199-12-31 00:00:00 |

你可以使用以下 SQL 语句在数据库中创建和填充此表：

```
CREATE TABLE Customers_scd
(
  CustomerId int,
  CustomerName varchar(20),
  CustomerCountry varchar(10),
  ValidFrom timestamp,
  Expired timestamp
);

INSERT INTO Customers_scd
  VALUES(100,'Jane','USA','2019-05-01 7:01:10',
    '2020-06-20 8:15:34');
INSERT INTO Customers_scd
  VALUES(100,'Jane','UK','2020-06-20 8:15:34',
    '2199-12-31 00:00:00');
```

您可以将此 SCD 与您之前创建的`Orders`表格连接起来，以获取订单时的客户记录属性。为此，除了使用`CustomerId`进行连接外，您还需要使用订单放置时 SCD 中的日期范围进行连接。例如，此查询将返回简的`Customers_scd`记录指示她当时住在的国家：

```
SELECT
  o.OrderId,
  o.OrderDate,
  c.CustomerName,
  c.CustomerCountry
FROM Orders o
INNER JOIN Customers_scd c
  ON o.CustomerId = c.CustomerId
    AND o.OrderDate BETWEEN c.ValidFrom AND c.Expired
ORDER BY o.OrderDate;
```

```
orderid |     orderdate     | customer | customer
                                name      country
---------+--------------------+--------+---------
      1 | 2020-06-09 00:00:00 | Jane   | USA
      4 | 2020-07-12 00:00:00 | Jane   | UK
(2 rows)
```

尽管此逻辑足以在数据建模中使用 SCD，但保持 SCD 的最新状态可能是一个挑战。对于`Customers`表格，您需要在每次摄入后对其进行快照，并查找任何已更改的`CustomerId`记录。如何处理这些取决于您使用的数据仓库和数据编排工具。如果您有兴趣实施 SCD，请学习 Kimball 建模的基础知识，这超出了本书的范围。如需更深入地了解此主题，请参阅 Ralph Kimball 和 Margy Ross 的书籍*数据仓库工具包*（Wiley, 2013）。

## 逐步建模增量摄入的数据

请回想来自第四章的信息，增量摄取的数据不仅包含源数据的当前状态，还包括自摄入开始以来的历史记录。例如，考虑与先前部分相同的`Orders`表，但是有一个名为`Customers_staging`的新客户表，它是增量摄入的。正如您在表 6-6 中所看到的，记录的`UpdatedDate`值有所更新，并且对于`CustomerId` 100 的新记录表明，简的`CustomerCountry`（她居住的地方）于 2020 年 06 月 20 日从美国变更为英国。

表 6-6\. 增量加载的 Customers_staging 表

| CustomerId | CustomerName | CustomerCountry | LastUpdated |
| --- | --- | --- | --- |
| 100 | Jane | 美国 | 2019-05-01 7:01:10 |
| 101 | Bob | 英国 | 2020-01-15 13:05:31 |
| 102 | Miles | 英国 | 2020-01-29 9:12:00 |
| 100 | Jane | 英国 | 2020-06-20 8:15:34 |

可以使用以下 SQL 语句在数据库中创建和填充`Customers_staging`表，以便在接下来的示例中使用：

```
CREATE TABLE Customers_staging (
  CustomerId int,
  CustomerName varchar(20),
  CustomerCountry varchar(10),
  LastUpdated timestamp
);

INSERT INTO Customers_staging
  VALUES(100,'Jane','USA','2019-05-01 7:01:10');
INSERT INTO Customers_staging
  VALUES(101,'Bob','UK','2020-01-15 13:05:31');
INSERT INTO Customers_staging
  VALUES(102,'Miles','UK','2020-01-29 9:12:00');
INSERT INTO Customers_staging
  VALUES(100,'Jane','UK','2020-06-20 8:15:34');
```

请回想前一节模型需要回答的问题，我将在本节中也应用到模型上：

+   在给定月份内来自特定国家下的订单生成了多少收入？

+   在特定日子下有多少订单？

在这种情况下，在构建数据模型之前，您需要决定如何处理`Customer`表中记录的更改。例如，在简的例子中，她在`Orders`表中的两个订单应分配给哪个国家？它们应该都分配给她当前的国家（英国）还是每个订单应该分配到下单时她所在的国家（分别是美国和英国）？

您所做的选择基于业务案例所需的逻辑，但每个实现都有些不同。我将以分配给她当前国家的示例开始。我将通过构建与前一节中相似的数据模型来完成这一点，但仅使用`Customers_staging`表中每个`CustomerId`的最新记录。请注意，由于模型要求的第二个问题需要每日粒度，我将在日期级别上构建模型：

```
CREATE TABLE order_summary_daily_current
(
  order_date date,
  order_country varchar(10),
  total_revenue numeric,
  order_count int
);

INSERT INTO order_summary_daily_current
  (order_date, order_country,
  total_revenue, order_count)
WITH customers_current AS
(
  SELECT CustomerId,
    MAX(LastUpdated) AS latest_update
  FROM Customers_staging
  GROUP BY CustomerId
)
SELECT
  o.OrderDate AS order_date,
  cs.CustomerCountry AS order_country,
  SUM(o.OrderTotal) AS total_revenue,
  COUNT(o.OrderId) AS order_count
FROM Orders o
INNER JOIN customers_current cc
  ON cc.CustomerId = o.CustomerId
INNER JOIN Customers_staging cs
  ON cs.CustomerId = cc.CustomerId
    AND cs.LastUpdated = cc.latest_update
GROUP BY o.OrderDate, cs.CustomerCountry;
```

当回答来自特定国家在给定月份内的订单生成了多少收入时，简的两个订单都分配给了英国，尽管您可能期望从她在美国生活时，将 6 月份的订单分配给美国，从而得到 50.05：

```
SELECT
  DATE_PART('month', order_date) AS order_month,
  order_country,
  SUM(total_revenue) AS order_revenue
FROM order_summary_daily_current
GROUP BY
  DATE_PART('month', order_date),
  order_country
ORDER BY
  DATE_PART('month', order_date),
  order_country;
```

```
order_month | order_country | order_revenue
-------------+---------------+---------------
          6 | UK            |         50.05
          7 | UK            |        236.44
(2 rows)
```

如果您希望根据客户下订单时所在国家来分配订单，那么构建模型就需要改变逻辑。不再查找每个`Customers_staging`中每个`CustomerId`的最近记录，而是查找每个客户下订单时更新时间在订单放置时间之前或同时的最近记录。换句话说，我希望获得客户下订单时的有效信息。该信息存储在他们的`Customer_staging`记录版本中，在该特定订单放置之后，直到后续更新为止。

以下示例中的`customer_pit`（*pit*是“时间点”）CTE 包含每个`CustomerId`/`OrderId`对的`MAX(cs.LastUpdated)`。我在最终的`SELECT`语句中使用这些信息来填充数据模型。请注意，在此查询中，必须根据`OrderId`和`CustomerId`进行连接。以下是`order_summary_daily_pit`模型的最终 SQL：

```
CREATE TABLE order_summary_daily_pit
(
  order_date date,
  order_country varchar(10),
  total_revenue numeric,
  order_count int
);

INSERT INTO order_summary_daily_pit
  (order_date, order_country, total_revenue, order_count)
WITH customer_pit AS
(
  SELECT
    cs.CustomerId,
    o.OrderId,
    MAX(cs.LastUpdated) AS max_update_date
  FROM Orders o
  INNER JOIN Customers_staging cs
    ON cs.CustomerId = o.CustomerId
      AND cs.LastUpdated <= o.OrderDate
  GROUP BY cs.CustomerId, o.OrderId
)
SELECT
  o.OrderDate AS order_date,
  cs.CustomerCountry AS order_country,
  SUM(o.OrderTotal) AS total_revenue,
  COUNT(o.OrderId) AS order_count
FROM Orders o
INNER JOIN customer_pit cp
  ON cp.CustomerId = o.CustomerId
    AND cp.OrderId = o.OrderId
INNER JOIN Customers_staging cs
  ON cs.CustomerId = cp.CustomerId
    AND cs.LastUpdated = cp.max_update_date
GROUP BY o.OrderDate, cs.CustomerCountry;
```

运行与之前相同的查询后，您将看到 Jane 的第一笔订单的收入在 2020 年 6 月分配给了美国，而第二笔订单在 2020 年 7 月分配给了英国，正如预期的那样：

```
SELECT
  DATE_PART('month', order_date) AS order_month,
  order_country,
  SUM(total_revenue) AS order_revenue
FROM order_summary_daily_pit
GROUP BY
  DATE_PART('month', order_date),
  order_country
ORDER BY
  DATE_PART('month', order_date),
  order_country;
```

```
order_month | order_country | order_revenue
-------------+---------------+---------------
          6 | USA           |         50.05
          7 | UK            |        236.44
(2 rows)
```

## 建模追加数据

*仅追加数据*（或*仅插入数据*）是指不可变数据，用于导入数据仓库。这种表中的每条记录都是一些永不改变的事件。例如，网站上所有页面浏览的表格。每次数据导入运行时，都会将新的页面浏览追加到表中，但不会更新或删除之前的事件。过去发生的事情已经发生，无法改变。

建模追加数据类似于建模完全刷新数据。然而，您可以通过利用一旦记录插入，它们永远不会更改的事实来优化基于这种数据构建的数据模型的创建和刷新。

表 6-7 是一个名为`PageViews`的追加数据表的示例，其中记录了网站上的页面浏览。表中的每条记录代表客户在公司网站上浏览页面的情况。每次数据导入作业运行时，都会将新的记录追加到表中，代表上次导入以来记录的页面浏览。

表 6-7\. 页面浏览表

| CustomerId | ViewTime | UrlPath | utm_medium |
| --- | --- | --- | --- |
| 100 | 2020-06-01 12:00:00 | /home | social |
| 100 | 2020-06-01 12:00:13 | /product/2554 | NULL |
| 101 | 2020-06-01 12:01:30 | /product/6754 | search |
| 102 | 2020-06-02 7:05:00 | /home | NULL |
| 101 | 2020-06-02 12:00:00 | /product/2554 | social |

您可以使用以下 SQL 查询在数据库中创建和填充`PageViews`表，以便在接下来的示例中使用。

```
CREATE TABLE PageViews (
  CustomerId int,
  ViewTime timestamp,
  UrlPath varchar(250),
  utm_medium varchar(50)
);

INSERT INTO PageViews
  VALUES(100,'2020-06-01 12:00:00',
    '/home','social');
INSERT INTO PageViews
  VALUES(100,'2020-06-01 12:00:13',
    '/product/2554',NULL);
INSERT INTO PageViews
  VALUES(101,'2020-06-01 12:01:30',
    '/product/6754','search');
INSERT INTO PageViews
  VALUES(102,'2020-06-02 7:05:00',
    '/home','NULL');
INSERT INTO PageViews
  VALUES(101,'2020-06-02 12:00:00',
    '/product/2554','social');
```

请注意，实际包含页面浏览数据的表可能包含几十个或更多列，存储有关浏览页面属性、引荐 URL、用户浏览器版本等的属性。

现在，我将定义一个数据模型，旨在回答以下问题。我将使用本章前文中定义的`Customers`表（表 6-4）来确定每位客户所居住的国家：

+   每天站点上每个`UrlPath`的页面浏览量是多少？

+   每天每个国家的客户生成多少页面浏览量？

数据模型的粒度是每天一次。有三个必需的属性。

+   页面查看的日期（无需时间戳）

+   页面查看的`UrlPath`

+   查看页面的客户所在的国家

只有一个必需的度量标准：

+   页面浏览量的计数

模型的结构如下：

```
CREATE TABLE pageviews_daily (
  view_date date,
  url_path varchar(250),
  customer_country varchar(50),
  view_count int
);
```

要首次填充模型，逻辑与本章“完全刷新数据建模”部分相同。从`PageViews`表中的所有记录都包括在`pageviews_daily`的填充中。示例 6-4 显示了 SQL。

##### 示例 6-4\. pageviews_daily.sql

```
INSERT INTO pageviews_daily
  (view_date, url_path, customer_country, view_count)
SELECT
  CAST(p.ViewTime as Date) AS view_date,
  p.UrlPath AS url_path,
  c.CustomerCountry AS customer_country,
  COUNT(*) AS view_count
FROM PageViews p
LEFT JOIN Customers c ON c.CustomerId = p.CustomerId
GROUP BY
  CAST(p.ViewTime as Date),
  p.UrlPath,
  c.CustomerCountry;
```

要回答模型要求的一个问题（每天每个国家的客户生成多少页面浏览量？），以下 SQL 将解决问题：

```
SELECT
  view_date,
  customer_country,
  SUM(view_count)
FROM pageviews_daily
GROUP BY view_date, customer_country
ORDER BY view_date, customer_country;
```

```
view_date  | customer_country | sum
------------+------------------+-----
2020-06-01 | UK               |   1
2020-06-01 | USA              |   2
2020-06-02 | UK               |   2
(3 rows)
```

现在考虑下次将数据摄入`PageViews`表时该做什么。会添加新记录，但所有现有记录保持不变。要更新`pageviews_daily`模型，有两个选项：

+   截断`pageviews_daily`表，并运行与首次填充它时使用的相同`INSERT`语句。在这种情况下，您正在*完全刷新*模型。

+   仅将新记录从`PageViews`加载到`pageviews_daily`中。在这种情况下，您正在*增量刷新*模型。

第一种选项最简单，而且在建模的分析师运行模型时不太可能导致逻辑错误。如果`INSERT`操作在您的使用情况下运行得足够快，请选择这条路线。但是要注意！虽然模型的完全刷新在初次开发时可能运行得足够快，但随着`PageViews`和`Customers`数据集的增长，刷新的运行时间也会增加。

第二个选项稍微复杂一些，但在处理较大数据集时可能会导致更短的运行时间。在这种情况下，增量刷新的棘手之处在于`pageviews_daily`表粒度为天（无时间戳的日期），而摄入到`PageViews`表中的新记录粒度为完整时间戳。

为什么这是个问题？很可能您在记录一个完整的日期结束时没有刷新`pageviews_daily`。换句话说，虽然`pageviews_daily`有 2020-06-02 的数据，但可能会在下一次摄入运行中加载该日的新记录到`PageViews`中。

[Table 6-8](https://example.org/pageviews_table_2) 就展示了这种情况。两条新记录已添加到之前版本的`PageViews`表格（来自[Table 6-7](https://example.org/pageviews_table)）。第一次新页面浏览发生在 2020-06-02，第二次是第二天。

Table 6-8\. PageViews 表格与额外记录

| CustomerId | ViewTime | UrlPath | utm_medium |
| --- | --- | --- | --- |
| 100 | 2020-06-01 12:00:00 | /home | social |
| 100 | 2020-06-01 12:00:13 | /product/2554 | NULL |
| 101 | 2020-06-01 12:01:30 | /product/6754 | search |
| 102 | 2020-06-02 7:05:00 | /home | NULL |
| 101 | 2020-06-02 12:00:00 | /product/2554 | social |
| 102 | 2020-06-02 12:03:42 | /home | NULL |
| 101 | 2020-06-03 12:25:01 | /product/567 | social |

在我尝试增量刷新`pageviews_daily`模型之前，先来看一下当前的快照：

```
SELECT *
FROM pageviews_daily
ORDER BY view_date, url_path, customer_country;
```

```
view_date  |   url_path   | customer | view_count
                            _country
------------+---------------+----------+---------
2020-06-01 | /home         | USA        |     1
2020-06-01 | /product/2554 | USA        |     1
2020-06-01 | /product/6754 | UK         |     1
2020-06-02 | /home         | UK         |     1
2020-06-02 | /product/2554 | UK         |     1
(5 rows)
```

现在，你可以使用以下 SQL 语句将[Table 6-8](https://example.org/pageviews_table_2)中显示的两条新记录插入到你的数据库中：

```
INSERT INTO PageViews
  VALUES(102,'2020-06-02 12:03:42',
    '/home',NULL);
INSERT INTO PageViews
  VALUES(101,'2020-06-03 12:25:01',
    '/product/567','social');
```

作为第一次增量刷新的尝试，你可以简单地将`PageViews`中时间戳大于当前`MAX(view_date)`（2020-06-02）的记录加入到`pageviews_daily`中。我会尝试这样做，但不是直接插入到`pageviews_daily`，而是创建另一个名为`pageviews_daily_2`的副本并用于本示例。为什么呢？因为，正如你马上会看到的，这并不是正确的方法！对应的 SQL 代码如下所示：

```
CREATE TABLE pageviews_daily_2 AS
SELECT * FROM pageviews_daily;

INSERT INTO pageviews_daily_2
  (view_date, url_path,
  customer_country, view_count)
SELECT
  CAST(p.ViewTime as Date) AS view_date,
  p.UrlPath AS url_path,
  c.CustomerCountry AS customer_country,
  COUNT(*) AS view_count
FROM PageViews p
LEFT JOIN Customers c
  ON c.CustomerId = p.CustomerId
WHERE
  p.ViewTime >
  (SELECT MAX(view_date) FROM pageviews_daily_2)
GROUP BY
  CAST(p.ViewTime as Date),
  p.UrlPath,
  c.CustomerCountry;
```

然而，正如你在下面的代码中看到的，你最终会得到几个重复的记录，因为所有 2020-06-02 午夜及以后的事件都包含在刷新中。换句话说，之前在模型中已经计算过的 2020-06-02 的页面浏览又被计算了一次。这是因为我们在每日粒度的`pageviews_daily`（及其副本`pageviews_daily_2`）中没有存储完整的时间戳。如果这个模型版本用于报告或分析，页面浏览次数将被高估！

```
SELECT *
FROM pageviews_daily_2
ORDER BY view_date, url_path, customer_country;
```

```
view_date  |   url_path   | customer | view_count
                            _country
------------+--------------+---------+-----------
2020-06-01 | /home         | USA     |     1
2020-06-01 | /product/2554 | USA     |     1
2020-06-01 | /product/6754 | UK      |     1
2020-06-02 | /home         | UK      |     2
2020-06-02 | /home         | UK      |     1
2020-06-02 | /product/2554 | UK      |     1
2020-06-02 | /product/2554 | UK      |     1
2020-06-03 | /product/567  | UK      |     1
(8 rows)
```

如果按日期汇总`view_count`，你会看到 2020-06-02 有五次页面浏览，而不是来自[Table 6-8](https://example.org/pageviews_table_2)的实际计数为三次。这是因为之前添加到`pageviews_daily_2`的两次页面浏览再次被添加进来：

```
SELECT
  view_date,
  SUM(view_count) AS daily_views
FROM pageviews_daily_2
GROUP BY view_date
ORDER BY view_date;
```

```
view_date  | daily_views
------------+-------------
2020-06-01 |           3
2020-06-02 |           5
2020-06-03 |           1
(3 rows)
```

许多分析师采取的另一种方法是存储`PageViews`表格中最后记录的完整时间戳，并将其用作增量刷新的下一个起始点。像上次一样，我会创建一个新表格（这次称为`pageviews_daily_3`），这并不是正确的解决方案：

```
CREATE TABLE pageviews_daily_3 AS
SELECT * FROM pageviews_daily;

INSERT INTO pageviews_daily_3
  (view_date, url_path,
  customer_country, view_count)
SELECT
  CAST(p.ViewTime as Date) AS view_date,
  p.UrlPath AS url_path,
  c.CustomerCountry AS customer_country,
  COUNT(*) AS view_count
FROM PageViews p
LEFT JOIN Customers c
  ON c.CustomerId = p.CustomerId
WHERE p.ViewTime > '2020-06-02 12:00:00'
GROUP BY
  CAST(p.ViewTime AS Date),
  p.UrlPath,
  c.CustomerCountry;
```

再次，如果你查看新版本的`pageviews_daily_3`，你会注意到一些非理想的地方。虽然 2020-06-02 的总页面浏览次数现在是正确的（`3`），但有两行是相同的（`view_date`为`2020-06-02`，`url_path`为`/home`，`customer_country`为`UK`）：

```
SELECT *
FROM pageviews_daily_3
ORDER BY view_date, url_path, customer_country;
```

```
view_date  |   url_path   | customer | view_count
                            _country
------------+--------------+---------+------------
2020-06-01 | /home         | USA     |     1
2020-06-01 | /product/2554 | USA     |     1
2020-06-01 | /product/6754 | UK      |     1
2020-06-02 | /home         | UK      |     1
2020-06-02 | /home         | UK      |     1
2020-06-02 | /product/2554 | UK      |     1
2020-06-03 | /product/567  | UK      |     1
(7 rows)
```

幸运的是，在这种情况下，按日和国家计算的页面浏览量是正确的。然而，存储不需要的数据是浪费的。这两条记录本可以合并成一条，视图计数值为 2。尽管这种情况下示例表格很小，但在实际情况下，这样的表格通常会有数十亿条记录。不必要的重复记录数量会增加存储和未来查询时间的浪费。

更好的方法是假设在最近一天（或一周、一个月等，根据表格的粒度）中加载了更多数据。我将采取的方法如下：

1.  创建名为`tmp_pageviews_daily`的`pageviews_daily`副本，其中包含截止到当前包含的倒数第二天的所有记录。在本例中，这意味着所有数据都截至到 2020-06-01。

1.  将所有源表（`PageViews`）的记录插入到从第二天（2020-06-02）开始的副本中。

1.  截断`pageviews_daily`并将数据从`tmp_pageviews_daily`加载到其中。

1.  删除`tmp_pageviews_daily`。

最终，模型增量刷新的正确 SQL 如下：

```
CREATE TABLE tmp_pageviews_daily AS
SELECT *
FROM pageviews_daily
WHERE view_date
  < (SELECT MAX(view_date) FROM pageviews_daily);

INSERT INTO tmp_pageviews_daily
  (view_date, url_path,
  customer_country, view_count)
SELECT
  CAST(p.ViewTime as Date) AS view_date,
  p.UrlPath AS url_path,
  c.CustomerCountry AS customer_country,
  COUNT(*) AS view_count
FROM PageViews p
LEFT JOIN Customers c
  ON c.CustomerId = p.CustomerId
WHERE p.ViewTime
  > (SELECT MAX(view_date) FROM pageviews_daily)
GROUP BY
  CAST(p.ViewTime as Date),
  p.UrlPath,
  c.CustomerCountry;

TRUNCATE TABLE pageviews_daily;

INSERT INTO pageviews_daily
SELECT * FROM tmp_pageviews_daily;

DROP TABLE tmp_pageviews_daily;
```

最后，以下是适当的增量刷新结果。页面浏览总数正确，且数据存储尽可能高效，符合模型要求：

```
SELECT *
FROM pageviews_daily
ORDER BY view_date, url_path, customer_country;
```

```
view_date  |   url_path   | customer | view_count
                            _country
------------+--------------+---------+------------
2020-06-01 | /home         | USA     |     1
2020-06-01 | /product/2554 | USA     |     1
2020-06-01 | /product/6754 | UK      |     1
2020-06-02 | /home         | UK      |     2
2020-06-02 | /product/2554 | UK      |     1
2020-06-03 | /product/567  | UK      |     1
(6 rows)
```

## 模型变更捕获数据

请回顾第四章，通过 CDC 摄取的数据在摄取后以特定方式存储在数据仓库中。例如，表 6-9 显示了通过 CDC 摄取的名为`Orders_cdc`的表的内容。它包含源系统中三个订单的历史记录。

表 6-9\. Orders_cdc 表

| EventType | OrderId | OrderStatus | LastUpdated |
| --- | --- | --- | --- |
| 插入 | 1 | Backordered | 2020-06-01 12:00:00 |
| 更新 | 1 | Shipped | 2020-06-09 12:00:25 |
| 删除 | 1 | Shipped | 2020-06-10 9:05:12 |
| 插入 | 2 | Backordered | 2020-07-01 11:00:00 |
| 更新 | 2 | Shipped | 2020-07-09 12:15:12 |
| 插入 | 3 | Backordered | 2020-07-11 13:10:12 |

您可以使用以下 SQL 语句创建并填充`Orders_cdc`表：

```
CREATE TABLE Orders_cdc
(
  EventType varchar(20),
  OrderId int,
  OrderStatus varchar(20),
  LastUpdated timestamp
);

INSERT INTO Orders_cdc
  VALUES('insert',1,'Backordered',
    '2020-06-01 12:00:00');
INSERT INTO Orders_cdc
  VALUES('update',1,'Shipped',
    '2020-06-09 12:00:25');
INSERT INTO Orders_cdc
  VALUES('delete',1,'Shipped',
    '2020-06-10 9:05:12');
INSERT INTO Orders_cdc
  VALUES('insert',2,'Backordered',
    '2020-07-01 11:00:00');
INSERT INTO Orders_cdc
  VALUES('update',2,'Shipped',
    '2020-07-09 12:15:12');
INSERT INTO Orders_cdc
  VALUES('insert',3,'Backordered',
    '2020-07-11 13:10:12');
```

订单 1 的记录首次创建于下单时，但状态为`Backordered`。八天后，该记录在发货时在源系统中更新。一天后，由于某种原因，该记录在源系统中被删除。订单 2 经历了类似的旅程，但从未被删除。订单 3 在下单时首次插入，从未更新过。多亏了 CDC，我们不仅知道所有订单的当前状态，还知道它们的完整历史。

如何建模以这种方式存储的数据取决于数据模型旨在回答什么问题。例如，您可能希望报告操作仪表板上所有订单的当前状态。也许仪表板需要显示每个状态中当前订单的数量。一个简单的模型可能看起来像这样：

```
CREATE TABLE orders_current (
  order_status varchar(20),
  order_count int
);

INSERT INTO orders_current
  (order_status, order_count)
  WITH o_latest AS
  (
    SELECT
       OrderId,
       MAX(LastUpdated) AS max_updated
    FROM Orders_cdc
    GROUP BY orderid
  )
  SELECT o.OrderStatus,
    Count(*) as order_count
  FROM Orders_cdc o
  INNER JOIN o_latest
    ON o_latest.OrderId = o_latest.OrderId
      AND o_latest.max_updated = o.LastUpdated
  GROUP BY o.OrderStatus;
```

在此示例中，我使用了 CTE 而不是子查询来查找每个`OrderId`的`MAX(LastUpdated)`时间戳。然后，我将结果 CTE 与`Orders_cdc`表连接，以获取每个订单最新记录的`OrderStatus`。

要回答最初的问题，您可以看到两个订单的`OrderStatus`为`Shipped`，还有一个订单仍然是`Backordered`：

```
SELECT * FROM orders_current;
```

```
order_status | order_count
--------------+-------------
Shipped      |           2
Backordered  |           1
(2 rows)
```

然而，这是否是正确的答案呢？请记住，虽然`OrderId` 1 的最新状态目前是`Shipped`，但`Order`记录已从源数据库中删除。尽管这可能看起来像是一个糟糕的系统设计，但现在假设当客户取消订单时，订单将从源系统中删除。为了考虑到删除的情况，我将对模型刷新进行小的修改以忽略删除操作：

```
TRUNCATE TABLE orders_current;

INSERT INTO orders_current
  (order_status, order_count)
  WITH o_latest AS
  (
    SELECT
       OrderId,
       MAX(LastUpdated) AS max_updated
    FROM Orders_cdc
    GROUP BY orderid
  )
  SELECT o.OrderStatus,
    Count(*) AS order_count
  FROM Orders_cdc o
  INNER JOIN o_latest
    ON o_latest.OrderId = o_latest.OrderId
      AND o_latest.max_updated = o.LastUpdated
  WHERE o.EventType <> 'delete'
  GROUP BY o.OrderStatus;
```

如您所见，删除的订单不再被考虑：

```
SELECT * FROM orders_current;
```

```
order_status | order_count
--------------+-------------
Shipped      |           1
Backordered  |           1
(2 rows)
```

CDC 摄入数据的另一个常见用途是理解变更本身。例如，也许分析师想要知道订单从`Backordered`到`Shipped`状态平均需要多长时间。我将再次使用 CTE（这次是两个！）来查找每个订单首次`Backordered`和`Shipped`的日期。然后我将这两个日期相减，以获取每个既是 backordered 又已经 shipped 的订单处于`Backordered`状态的天数。请注意，此逻辑有意忽略了`OrderId` 3，该订单当前是 backordered，但尚未发货：

```
CREATE TABLE orders_time_to_ship (
  OrderId int,
  backordered_days interval
);

INSERT INTO orders_time_to_ship
  (OrderId, backordered_days)
WITH o_backordered AS
(
  SELECT
     OrderId,
     MIN(LastUpdated) AS first_backordered
  FROM Orders_cdc
  WHERE OrderStatus = 'Backordered'
  GROUP BY OrderId
),
o_shipped AS
(
  SELECT
     OrderId,
     MIN(LastUpdated) AS first_shipped
  FROM Orders_cdc
  WHERE OrderStatus = 'Shipped'
  GROUP BY OrderId
)
SELECT b.OrderId,
  first_shipped - first_backordered
    AS backordered_days
FROM o_backordered b
INNER JOIN o_shipped s on s.OrderId = b.OrderId;
```

您可以看到每个订单的 backorder 时间，以及使用`AVG()`函数来回答最初的问题：

```
SELECT * FROM orders_time_to_ship;
```

```
orderid | backordered_days
---------+------------------
      1 | 8 days 00:00:25
      2 | 8 days 01:15:12
(2 rows)
```

```
SELECT AVG(backordered_days)
FROM orders_time_to_ship;
```

```
avg
-------------------
8 days 00:37:48.5
(1 row)
```

对于具有完整更改历史记录的数据，还有许多其他用例，但就像对已完全加载或仅追加的数据进行建模一样，有一些常见的最佳实践和考虑因素。

像前一节一样，通过利用 CDC 摄入的数据是增量加载而不是完全刷新，可以实现潜在的性能提升。但是，正如在该部分中指出的那样，有时性能增益并不值得采用增量模型刷新而不是全刷新的复杂性。在处理 CDC 数据时，我发现这种情况大多数情况下是正确的。处理更新和删除的额外复杂性通常足以使全刷新成为首选路径。
