# 第十一章：连接设计模式

在本章中，我们将探讨连接数据集的实用设计模式。与前几章一样，我将专注于在实际环境中有用的模式。PySpark 支持 RDD 和 DataFrames 的基本连接操作（`pyspark.RDD.join()`和`pyspark.sql.DataFrame.join()`），这将适用于大多数用例。但是，在某些情况下，这种连接可能会很昂贵，因此我还将展示一些特殊的连接算法，这些算法可能会很有用。

本章介绍了连接两个数据集的基本概念，并提供了一些有用和实用的连接设计模式示例。我将展示如何在 MapReduce 范式中实现连接操作以及如何使用 Spark 的转换来执行连接。您将看到如何使用 RDD 和 DataFrames 执行映射端连接，以及如何使用布隆过滤器执行高效连接。

# 连接操作介绍

在关系数据库世界中，连接两个具有共同键的表（也称为“关系”）——即一个或多个列中的属性或一组属性，这些属性允许唯一标识表中每个记录（元组或行）——是一个频繁的操作。

考虑以下两个表，`T1` 和 `T2`：

```
T1 = {(k1, v1)}
T2 = {(k2, v2)}
```

其中：

+   k1 是 T1 的键，v1 是关联的属性。

+   k2 是 T2 的键，v2 是关联的属性。

简单内连接会通过合并两个或多个表中具有匹配键的行来创建新表，定义如下：

```
T1.join(T2) = {(k, (v1, v2))}
T2.join(T1) = {(k, (v2, v1))}
```

其中：

+   k = k1 = k2。

+   (k, v1) 存在于 T1 中。

+   (k, v2) 存在于 T2 中。

为了说明其工作原理，让我们创建两个表，填充一些示例数据，然后进行连接。首先我们将创建我们的表，`T1` 和 `T2`：

```
>>> d1 = [('a', 10), ('a', 11), ('a', 12), ('b', 100), ('b', 200), ('c', 80)]
>>> T1 = spark.createDataFrame(d1, ['id', 'v1'])
>>> T1.show()
+---+---+
| id| v1|
+---+---+
|  a| 10|
|  a| 11|
|  a| 12|
|  b|100|
|  b|200|
|  c| 80|
+---+---+

>>> d2 = [('a', 40), ('a', 50), ('b', 300), ('b', 400), ('d', 90)]
>>> T2 = spark.createDataFrame(d2, ['id', 'v2'])
>>> T2.show()
+---+---+
| id| v2|
+---+---+
|  a| 40|
|  a| 50|
|  b|300|
|  b|400|
|  d| 90|
+---+---+
```

然后我们将使用内连接将它们连接起来（Spark 中的默认连接类型）。请注意，由于在另一张表中找不到匹配行，具有`id`为`c`（来自`T1`）和`d`（来自`T2`）的行被丢弃：

```
>>> joined = T1.join(T2, (T1.id == T2.id))
>>> joined.show(100, truncate=False)
+---+---+---+---+
|id |v1 |id |v2 |
+---+---+---+---+
|a  |10 |a  |50 |
|a  |10 |a  |40 |
|a  |11 |a  |50 |
|a  |11 |a  |40 |
|a  |12 |a  |50 |
|a  |12 |a  |40 |
|b  |100|b  |400|
|b  |100|b  |300|
|b  |200|b  |400|
|b  |200|b  |300|
+---+---+---+---+
```

可以对具有共同键的两个表执行许多类型的连接操作，但在实践中，三种连接类型最为常见：

`INNER` `JOIN(T1, T2)`

在两个表 T1 和 T2 中结合记录，只要这些记录在两个表中具有匹配值的键。

`LEFT` `JOIN(T1, T2)`

返回左表（T1）的所有记录以及右表（T2）中匹配的记录。如果某个特定记录没有匹配项，则右表相应列中将会有`NULL`值。

`RIGHT` `JOIN(T1, T2)`

返回连接右侧表（T2）的所有行，并且左侧表（T1）中有匹配行的行。对于左侧没有匹配行的行，结果集将包含空值。

所有这些连接类型都受到 PySpark 的支持，以及一些其他不常用的类型。有关 PySpark 支持的不同连接类型的介绍，请参阅 Spark by {Examples} 网站上的教程[“PySpark Join Types”](https://oreil.ly/JoIUD)。

连接两个表可能是一个昂贵的操作，因为它可能需要找到笛卡尔积（对于两个集合 A 和 B，所有有序对 (x, y) 其中 x 在 A 中且 y 在 B 中）。在刚刚展示的示例中，这不会成为问题，但考虑一个大数据示例：如果表 `T1` 有三十亿行，表 `T2` 有一百万行，那么这两个表的笛卡尔积将有三千兆（3 后跟 15 个零）个数据点。在本章中，我介绍了一些基本的设计模式，可以帮助简化连接操作，以降低这种成本。通常情况下，在选择和使用连接设计模式时，并没有银弹：务必使用真实数据测试您提出的解决方案的性能和可扩展性。

# 在 MapReduce 中进行连接

这一节是为了教学目的而呈现的，展示了在分布式计算环境中如何实现 `join()` 函数。假设我们有两个关系，`R(k, b)` 和 `S(k, c)`，其中 `k` 是一个公共键，`b` 和 `c` 分别表示 `R` 和 `S` 的属性。我们如何找到 `R` 和 `S` 的连接？连接操作的目标是找到在它们的键 `k` 上一致的元组。`R` 和 `S` 的自然连接的 MapReduce 实现可以如下实现。首先，在映射阶段：

+   对于在 `R` 中的元组   对于元组 `(k, b)` 在 `R` 中，以 `(k, ("R", b))` 的形式发出一个 (键, 值) 对。

+   对于元组 `(k, c)` 在 `S` 中，以 `(k, ("S", c))` 的形式发出一个 (键, 值) 对。

然后，在减少阶段：

+   如果一个 reducer 键 `k` 有值列表 `[("R", v),("S", w)]`，那么以 `(k, (v, w))` 的形式发出一个 (键, 值) 对。请注意，`join(R, S)` 将产生 `(k, (v, w))`，而 `join(S, R)` 将产生 `(k, (w, v))而 `join(S, R)` 将产生 `(k, (w, v))`。

因此，如果一个 reducer 键 `k` 有值列表 `[("R", v1), ("R", v2), ("S", w1), ("S", w2)]`，那么我们将发出四个 (键, 值) 对：

```
(k, (v1, w1))
(k, (v1, w2))
(k, (v2, w1))
(k, (v2, w2))
```

因此，要在两个关系 `R` 和 `S` 之间执行自然连接，我们需要两个映射函数和一个 reducer 函数。

## 映射阶段

映射阶段有两个步骤：

1.  映射关系 `R`：

    ```
    # key: relation R
    # value: (k, b) tuple in R
    map(key, value) {
      emit(k, ("R", b))
    }
    ```

1.  映射关系 `S`：

```
# key: relation S
# value: (k, c) tuple in S
map(key, value) {
  emit(k, ("S", c))
}
```

映射器的输出（作为排序和洗牌阶段的输入）将是：

```
(k1, "R", r1)
(k1, "R", r2)
...
(k1, "S", s1)
(k1, "S", s2)
...
(k2, "R", r3)
(k2, "R", r4)
...
(k2, "S", s3)
(k2, "S", s4)
...
```

## 减少器阶段

在编写一个 reducer 函数之前，我们需要理解 MapReduce 的神奇之处，这发生在排序和洗牌阶段。这类似于 SQL 的 `GROUP BY` 函数；一旦所有的映射器完成，它们的输出会被排序、洗牌，并作为输入发送给 reducer(s)。

在我们的示例中，排序和洗牌阶段的输出将是：

```
(k1, [("R", r1), ("R", r2), ..., ("S", s1), ("S", s2), ...]
(k2, [("R", r3), ("R", r4), ..., ("S", s3), ("S", s4), ...]
...
```

接下来是 reducer 函数。对于每个键 `k`，我们构建两个列表：`list_R`（将保存来自关系 `R` 的值/属性）和 `list_S`（将保存来自关系 `S` 的值/属性）。然后我们确定 `list_R` 和 `list_S` 的笛卡尔积，以找到连接元组（伪代码）：

```
# key: a unique key
# values: [(relation, attrs)] where relation in {"R", "S"}
# and  attrs are the relation attributes
reduce(key, values) {
  list_R = []
  list_S = []
  for (tuple in values) {
    relation = tuple[0]
    attributes = tuple[1]
    if (relation == "R") {
       list_R.append(attributes)
    }
    else {
       list_S.append(attributes)
    }
  }

  if (len(list_R) == 0) OR (len(list_S) == 0) {
     # no common key
     return
  }

  # len(list_R) > 0 AND len(list_S) > 0
  # perform Cartesian product of list_R and list_S
  for (r in list_R) {
    for (s in list_S) {
       emit(key, (r, s))
    }
  }

}
```

## 在 PySpark 中的实现

本节展示了如何在 PySpark 中实现两个数据集的自然连接（带有一些共同的键），而不使用`join()`函数。我提出这个解决方案是为了展示 Spark 的强大之处，以及如何在需要时执行自定义连接。

假设我们有以下数据集，`T1`和`T2`：

```
d1 = [('a', 10), ('a', 11), ('a', 12), ('b', 100), ('b', 200), ('c', 80)]
d2 = [('a', 40), ('a', 50), ('b', 300), ('b', 400), ('d', 90)]
T1 = spark.sparkContext.parallelize(d1)
T2 = spark.sparkContext.parallelize(d2)
```

首先，我们将这些 RDDs 映射到包含关系名称的形式：

```
t1_mapped = T1.map(lambda x: (x[0], ("T1", x[1])))
t2_mapped = T2.map(lambda x: (x[0], ("T2", x[1])))
```

接下来，为了对 mapper 生成的（键，值）对执行缩减操作，我们将这两个数据集组合成单个数据集：

```
combined  = t1_mapped.union(t2_mapped)
```

然后我们在一个单一的组合数据集上执行`groupByKey()`转换：

```
grouped  = combined.groupByKey()
```

最后，我们找到每个`grouped`条目的值的笛卡尔积：

```
# entry[0]: key
# entry[1]: values as:
# [("T1", t11), ("T1", t12), ..., ("T2", t21), ("T2", t22), ...]
import itertools
def cartesian_product(entry):
  T1 = []
  T2 = []
  key = entry[0]
  values = entry[1]
  for tuple in values:
    relation = tuple[0]
    attributes = tuple[1]
    if (relation == "T1"): T1.append(attributes)
    else: T2.append(attributes)
  #end-for

  if (len(T1) == 0) or (len(T2) == 0):
     # no common key
     return []

  # len(T1) > 0 AND len(T2) > 0
  joined_elements = []
  # perform Cartesian product of T1 and T2
  for element in itertools.product(T1, T2):
    joined_elements.append((key, element))
  #end-for
  return joined_elements
#end-def

joined = grouped.flatMap(cartesian_product)
```

# 使用 RDD 进行 Map-Side Join

如我们所见，连接是一种可能昂贵的操作，用于基于它们之间的共同键组合来自两个（或更多）数据集的记录。在关系数据库中，索引可以帮助减少连接操作的成本；然而，像 Hadoop 和 Spark 这样的大数据引擎不支持数据索引。那么，我们可以做些什么来最小化两个分布式数据集之间连接的成本？在这里，我将介绍一种设计模式，它可以完全消除 MapReduce 范式中的洗牌和排序阶段：*map-side join*。

Map-side join 是一个过程，其中两个数据集由 mapper 而不是实际的连接函数（由 mapper 和 reducer 的组合执行）连接。除了减少洗牌和减少阶段中的排序和合并成本外，这还可以加快任务的执行速度，提高性能。

要帮助你理解这是如何运作的，我们从一个 SQL 示例开始。假设我们在 MySQL 数据库中有两个表，`EMP`和`DEPT`，我们想要对它们进行连接操作。这两个表的定义如下：

```
mysql> use testdb;
Database changed

mysql> select * from emp;
+--------+----------+---------+
| emp_id | emp_name | dept_id |
+--------+----------+---------+
|   1000 | alex     | 10      |
|   2000 | ted      | 10      |
|   3000 | mat      | 20      |
|   4000 | max      | 20      |
|   5000 | joe      | 10      |
+--------+----------+---------+
5 rows in set (0.00 sec)

mysql> select * from dept;
+---------+------------+---------------+
| dept_id | dept_name  | dept_location |
+---------+------------+---------------+
|      10 | ACCOUNTING | NEW YORK, NY  |
|      20 | RESEARCH   | DALLAS, TX    |
|      30 | SALES      | CHICAGO, IL   |
|      40 | OPERATIONS | BOSTON, MA    |
|      50 | MARKETING  | Sunnyvale, CA |
|      60 | SOFTWARE   | Stanford, CA  |
+---------+------------+---------------+
6 rows in set (0.00 sec)
```

然后，我们使用`INNER JOIN`在`dept_id`键上连接两个表：

```
mysql> select e.emp_id, e.emp_name, e.dept_id, d.dept_name, d.dept_location
         from emp e, dept d
             where e.dept_id = d.dept_id;
+--------+----------+---------+------------+---------------+
| emp_id | emp_name | dept_id | dept_name  | dept_location |
+--------+----------+---------+------------+---------------+
|   1000 | alex     | 10      | ACCOUNTING | NEW YORK, NY  |
|   2000 | ted      | 10      | ACCOUNTING | NEW YORK, NY  |
|   5000 | joe      | 10      | ACCOUNTING | NEW YORK, NY  |
|   3000 | mat      | 20      | RESEARCH   | DALLAS, TX    |
|   4000 | max      | 20      | RESEARCH   | DALLAS, TX    |
+--------+----------+---------+------------+---------------+
5 rows in set (0.00 sec)
```

Map-side join 类似于 SQL 中的内连接，但任务仅由 mapper 执行（请注意，内连接和 map-side join 的结果必须相同）。

一般来说，在大数据集上进行连接是昂贵的，但很少希望将一个大表`A`的全部内容与另一个大表`B`的全部内容连接起来。给定两个表`A`和`B`，当表`A`（称为*事实表*）很大而表`B`（*维度表*）是小到中等规模时，map-side join 将是最合适的。为了执行这种类型的连接，我们首先从`B`创建一个哈希表，并将其广播到所有节点。接下来，我们通过 mapper 迭代表`A`的所有元素，然后通过广播的哈希表访问表`B`中的相关信息。

为了演示，我们将从我们的`EMP`和`DEPT`表创建两个 RDDs。首先，我们将`EMP`创建为`RDD[(dept_id, (emp_id, emp_name))]`：

```
EMP = spark.sparkContext.parallelize(
[
  (10, (1000, 'alex')),
  (10, (2000, 'ted')),
  (20, (3000, 'mat')),
  (20, (4000, 'max')),
  (10, (5000, 'joe'))
])
```

接下来，我们将`DEPT`创建为`RDD[(dept_id, (dept_name, dept_location))]`：

```
DEPT= spark.sparkContext.parallelize(
[ (10, ('ACCOUNTING', 'NEW YORK, NY')),
  (20, ('RESEARCH', 'DALLAS, TX')),
  (30, ('SALES', 'CHICAGO, IL')),
  (40, ('OPERATIONS', 'BOSTON, MA')),
  (50, ('MARKETING', 'Sunnyvale, CA')),
  (60, ('SOFTWARE', 'Stanford, CA'))
])
```

`EMP`和`DEPT`具有共同的键`dept_id`，所以我们可以如下连接这两个 RDDs：

```
>>> sorted(EMP.join(DEPT).collect())
[
 (10, ((1000, 'alex'), ('ACCOUNTING', 'NEW YORK, NY'))),
 (10, ((2000, 'ted'), ('ACCOUNTING', 'NEW YORK, NY'))),
 (10, ((5000, 'joe'), ('ACCOUNTING', 'NEW YORK, NY'))),
 (20, ((3000, 'mat'), ('RESEARCH', 'DALLAS, TX'))),
 (20, ((4000, 'max'), ('RESEARCH', 'DALLAS, TX')))
]
```

地图端连接如何优化此任务？假设`EMP`是一个大数据集，而`DEPT`是一个相对较小的数据集。使用地图端连接在`dept_id`上将`EMP`与`DEPT`连接时，我们将从小表创建广播变量（使用自定义函数`to_hash_table()`）：

```
# build a dictionary of (key, value),
# where key = dept_id
#       value = (dept_name , dept_location)

def to_hash_table(dept_as_list):
  hast_table = {}
  for d in dept_as_list:
    dept_id = d[0]
    dept_name_location = d[1]
    hash_table[dept_id] = dept_name_location
  return hash_table
#end-def

dept_hash_table = to_hash_table(DEPT.collect())
```

或者，您可以使用 Spark 操作`collectAsMap()`构建哈希表，该操作将此 RDD（`DEPT`）中的（键，值）对作为字典返回到主节点：

```
dept_hash_table = DEPT.collectAsMap()
```

现在，使用`pyspark.SparkContext.broadcast()`，我们可以将只读变量`dept_hash_table`广播到 Spark 集群，使其在各种转换（包括 mapper 和 reducer）中可用：

```
sc = spark.sparkContext
hash_table_broadcasted = sc.broadcast(dept_hash_table)
```

为了执行地图端连接，在 mapper 中我们可以通过以下方式访问此变量：

```
dept_hash_table = hash_table_broadcasted.value
```

使用如下定义的`map_side_join()`函数：

```
# e as an element of EMP RDD
def map_side_join(e):
  dept_id = e[0]
  # get hash_table from broadcasted object
  hash_table = hash_table_broadcasted.value
  dept_name_location = hash_table[dept_id]
  return (e, dept_name_location)
#end-def
```

然后，我们可以使用`map()`转换执行连接：

```
joined = EMP.map(map_side_join)
```

这使我们能够不洗牌维度表（即`DEPT`），并获得相当良好的连接性能。

通过地图端连接，我们只需使用`map()`函数迭代`EMP`表的每一行，并从广播的哈希表中检索维度值（如`dept_name`和`dept_location`）。`map()`函数将并行执行每个分区，每个分区将拥有自己的哈希表副本。

总结一下，地图端连接方法具有以下重要优势：

+   通过将较小的 RDD/表作为广播变量，从而避免洗牌，减少连接操作的成本，最小化需要在洗牌和减少阶段进行排序和合并的数据量。

+   通过避免大量网络 I/O 来提高连接操作的性能。其主要缺点是，地图端连接设计模式仅在希望执行连接操作的 RDD/表之一足够小，可以放入内存时才适合使用。如果两个表都很大，则不适合选择此方法。

# 使用 DataFrame 进行地图端连接

正如我在前面的部分中讨论的那样，当其中一个表（事实表）很大而另一个（维度表）足够小以广播时，地图端连接是有意义的。

在以下示例中（受到 Dmitry Tolpeko 文章[“Spark 中的地图端连接”](https://oreil.ly/2sHBy)的启发），我将展示如何使用 DataFrame 与广播变量实现地图端连接。假设我们有表 11-1 所示的事实表，以及表 11-2 和表 11-3 所示的两个维度表。

表 11-1\. `航班`（事实表）

| from | to | airline | flight_number | departure |
| --- | --- | --- | --- | --- |
| DTW | ORD | SW | 225 | 17:10 |
| DTW | JFK | SW | 355 | 8:20 |
| SEA | JFK | DL | 418 | 7:00 |
| SFO | LAX | AA | 1250 | 7:05 |
| SFO | JFK | VX | 12 | 7:05 |
| JFK | LAX | DL | 424 | 7:10 |
| LAX | SEA | DL | 5737 | 7:10 |

表 11-2\. `机场`（维度表）

| code | name | city | state |
| --- | --- | --- | --- |
| DTW | 底特律机场 | 底特律 | 密歇根州 |
| ORD | 芝加哥奥黑尔 | 芝加哥 | 伊利诺伊州 |
| JFK | 约翰·肯尼迪机场 | 纽约 | 纽约州 |
| LAX | 洛杉矶机场 | 洛杉矶 | 加利福尼亚州 |
| SEA | 西雅图-塔科马机场 | 西雅图 | 华盛顿州 |
| SFO | 旧金山机场 | 旧金山 | 加利福尼亚州 |

Table 11-3\. `Airlines`（维度表）

| 代码 | 航空公司名称 |
| --- | --- |
| SW | 西南航空 |
| AA | 美国航空 |
| DL | 三角洲航空 |
| VX | 维珍美国航空 |

我们的目标是扩展`Flights`表，用实际的航空公司名称替换航空公司代码，并用实际的机场名称替换机场代码。这一操作需要将事实表`Flights`与两个维度表（`Airports`和`Airlines`）进行连接。由于维度表足够小以适应内存，我们可以将其广播到所有工作节点的所有映射器上。Table 11-4 显示了所需的连接输出。

Table 11-4\. 连接后的表

| 出发城市 | 到达城市 | 航空公司 | 航班号 | 出发时间 |
| --- | --- | --- | --- | --- |
| 底特律 | 芝加哥 | 西南航空 | 225 | 17:10 |
| 底特律 | 纽约 | 西南航空 | 355 | 8:20 |
| 西雅图 | 纽约 | 三角洲航空 | 418 | 7:00 |
| 旧金山 | 洛杉矶 | 美国航空 | 1250 | 7:05 |
| 旧金山 | 纽约 | 维珍美国航空 | 12 | 7:05 |
| 纽约 | 洛杉矶 | 三角洲航空 | 424 | 7:10 |
| 洛杉矶 | 西雅图 | 三角洲航空 | 5737 | 7:10 |

要实现这个结果，我们需要执行以下步骤：

1.  为`Airports`创建广播变量。首先，我们从`Airports`表创建一个 RDD，并将其保存为`dict[(key, value)]`，其中 key 是机场代码，value 是机场名称。

1.  为`Airlines`创建广播变量。接下来，我们从`Airlines`表创建一个 RDD，并将其保存为`dict[(key, value)]`，其中 key 是航空公司代码，value 是航空公司名称。

1.  从`Flights`表创建一个 DataFrame，以与步骤 1 和 2 中创建的缓存广播变量进行连接。

1.  映射`Flights` DataFrame 的每条记录，并通过在步骤 1 和 2 中创建的缓存字典中查找值进行简单连接。

接下来，我将讨论另一种设计模式，使用布隆过滤器进行连接，可用于高效地连接两个表。

## -   创建机场缓存的步骤

这一步将`Airports`表（作为字典）创建为广播变量，并缓存在所有工作节点上：

```
>>> airports_data = [
...   ("DTW", "Detroit Airport", "Detroit", "MI"),
...   ("ORD", "Chicago O'Hare", "Chicago",  "IL"),
...   ("JFK", "John F. Kennedy Int. Airport", "New York", "NY"),
...   ("LAX", "Los Angeles Int. Airport", "Los Angeles", "CA"),
...   ("SEA", "Seattle-Tacoma Int. Airport", "Seattle", "WA"),
...   ("SFO", "San Francisco Int. Airport", "San Francisco", "CA")
... ]
>>>
>>> airports_rdd = spark.sparkContext.parallelize(airports_data)\
...   .map(lambda tuple4: (tuple4[0], (tuple4[1],tuple4[2],tuple4[3])))

>>> airports_dict = airports_rdd.collectAsMap()
>>>
>>> airports_cache = spark.sparkContext.broadcast(airports_dict)
>>> airports_cache.value
{'DTW': ('Detroit Airport', 'Detroit', 'MI'),
 'ORD': ("Chicago O'Hare", 'Chicago', 'IL'),
 'JFK': ('John F. Kennedy Int. Airport', 'New York', 'NY'),
 'LAX': ('Los Angeles Int. Airport', 'Los Angeles', 'CA'),
 'SEA': ('Seattle-Tacoma Int. Airport', 'Seattle', 'WA'),
 'SFO': ('San Francisco Int. Airport', 'San Francisco', 'CA')}
```

## Step 2: 为航空公司创建缓存

这一步将`Airlines`表创建为广播变量，并缓存在所有工作节点上：

```
>>> airlines_data = [
...   ("SW", "Southwest Airlines"),
...   ("AA", "American Airlines"),
...   ("DL", "Delta Airlines"),
...   ("VX", "Virgin America")
... ]

>>> airlines_rdd = spark.sparkContext.parallelize(airlines_data)\
...   .map(lambda tuple2: (tuple2[0], tuple2[1]))

>>> airlines_dict = airlines_rdd.collectAsMap()
>>> airlines_cache = spark.sparkContext.broadcast(airlines_dict)
>>> airlines_cache
>>> airlines_cache.value
{'SW': 'Southwest Airlines',
 'AA': 'American Airlines',
 'DL': 'Delta Airlines',
 'VX': 'Virgin America'}
```

## Step 3: 创建事实表

这一步将`Flights`表创建为 DataFrame，并将其用作事实表，与步骤 1 和 2 中创建的缓存字典进行连接：

```
>>> flights_data = [
...   ("DTW", "ORD", "SW", "225",  "17:10"),
...   ("DTW", "JFK", "SW", "355",  "8:20"),
...   ("SEA", "JFK", "DL", "418",  "7:00"),
...   ("SFO", "LAX", "AA", "1250", "7:05"),
...   ("SFO", "JFK", "VX", "12",   "7:05"),
...   ("JFK", "LAX", "DL", "424",  "7:10"),
...   ("LAX", "SEA", "DL", "5737", "7:10")
... ]
>>> flight_columns = ["from", "to", "airline", "flight_number", "departure"]
>>> flights = spark.createDataFrame(flights_data, flight_columns)
>>> flights.show(truncate=False)

+----+---+-------+-------------+---------+
|from|to |airline|flight_number|departure|
+----+---+-------+-------------+---------+
|DTW |ORD|SW     |225          |17:10    |
|DTW |JFK|SW     |355          |8:20     |
|SEA |JFK|DL     |418          |7:00     |
|SFO |LAX|AA     |1250         |7:05     |
|SFO |JFK|VX     |12           |7:05     |
|JFK |LAX|DL     |424          |7:10     |
|LAX |SEA|DL     |5737         |7:10     |
+----+---+-------+-------------+---------+
```

## Step 4: 应用映射端连接

最后，我们迭代事实表并执行映射端连接：

```
>>> from pyspark.sql.functions import udf
>>> from pyspark.sql.types import StringType
>>>
>>> def get_airport(code):
...   return airports_cache.value[code][1]
...
>>> def get_airline(code):
...   return airlines_cache.value[code]

>>> airport_udf = udf(get_airport, StringType())
...
>>> airport_udf = udf(get_airport, StringType())
>>>
>>> flights.select(
        airport_udf("from").alias("from_city"), ![1](img/1.png)
        airport_udf("to").alias("to_city"), ![2](img/2.png)
        airline_udf("airline").alias("airline_name"), ![3](img/3.png)
        "flight_number", "departure").show(truncate=False)
+-------------+-----------+------------------+-------------+---------+
|from_city    |to_city    |airline_name      |flight_number|departure|
+-------------+-----------+------------------+-------------+---------+
|Detroit      |Chicago    |Southwest Airlines|225          |17:10    |
|Detroit      |New York   |Southwest Airlines|355          |8:20     |
|Seattle      |New York   |Delta Airlines    |418          |7:00     |
|San Francisco|Los Angeles|American Airlines |1250         |7:05     |
|San Francisco|New York   |Virgin America    |12           |7:05     |
|New York     |Los Angeles|Delta Airlines    |424          |7:10     |
|Los Angeles  |Seattle    |Delta Airlines    |5737         |7:10     |
+-------------+-----------+------------------+-------------+---------+
```

![1](img/#co_join_design_patterns_CO1-1)

机场的映射端连接

![2](img/#co_join_design_patterns_CO1-2)

机场的地图端连接

![3](img/#co_join_design_patterns_CO1-3)

航空公司的地图端连接

# 使用布隆过滤器进行高效连接

给定两个 RDD，一个较大的`RDD[(K, V)]`和一个较小的`RDD[(K, W)]`，Spark 允许我们在键`K`上执行连接操作。在使用 Spark 时，连接两个 RDD 是一种常见操作。在某些情况下，连接被用作过滤的一种形式：例如，如果您想对`RDD[(K, V)]`中的记录子集执行操作，这些记录由另一个`RDD[(K, W)]`中的实体表示，您可以使用内连接来实现这种效果。然而，您可能更喜欢避免连接操作引入的洗牌，特别是如果您想要用于过滤的`RDD[(K, W)]`显著小于您将对其进行进一步计算的主要`RDD[(K, V)]`。

您可以使用广播连接（使用作为布隆过滤器的集合）来执行过滤，但这需要将希望按其收集的较小 RDD 整体收集到驱动程序内存中，即使它相对较小（几千或几百万条记录），这仍可能导致一些不希望的内存压力。如果要避免连接操作引入的洗牌，则可以使用布隆过滤器。这将连接`RDD[(K, V)]`与从较小的`RDD[(K, W)]`构建的布隆过滤器之间的问题简化为一个简单的`map()`转换，其中我们检查键`K`是否存在于布隆过滤器中。

## 布隆过滤器介绍

[布隆过滤器](https://oreil.ly/sFnRT)是一种空间高效的概率数据结构，可用于测试元素是否为集合的成员。它可能对不实际为集合成员的元素返回 true（即可能存在误报），但对于确实在集合中的元素，它不会返回 false；查询将返回“可能在集合中”或“绝对不在集合中”。可以向集合添加元素，但不能删除。随着向集合添加的元素越来越多，误报的概率就越大。

简而言之，我们可以总结布隆过滤器的属性如下：

+   给定一个大集合`S = {x[1], x[2], …, x[n]}`，布隆过滤器是一种概率、快速且空间高效的缓存生成器。它不会存储集合中的项本身，并且使用的空间比理论上正确存储数据所需的空间少；这是其潜在不准确性的根源。

+   它基本上近似于集合成员操作，并尝试回答“项目`x`是否存在于集合`S`中？”的问题。

+   它允许误报。这意味着对于某些不在集合中的`x`，布隆过滤器可能会指示`x`在集合中。

+   它不允许误报。这意味着如果`x`在集合中，布隆过滤器永远不会指示`x`不在集合中。

为了更清晰地表达，让我们看一个简单的关系或表之间的连接示例。假设我们想要在共同键 `K` 上连接 `R=RDD(K, V)` 和 `S=RDD(K, W)`。进一步假设以下内容为真：

```
count(R) = 1000,000,000  (larger dataset)
count(S) =   10,000,000  (smaller dataset)
```

要进行基本连接，我们需要检查 10 万亿（10¹²）条记录，这是一个庞大且耗时的过程。减少连接操作所需的时间和复杂性的一种方法是在关系 `S` 上使用布隆过滤器（较小的数据集），然后在关系 `R` 上使用构建好的布隆过滤器数据结构。这可以消除 `R` 中不必要的记录（可能将其大小减少到 20,000,000 条记录），使连接更快速和高效。

现在，让我们半正式地定义布隆过滤器数据结构。我们如何构建一个？假阳性错误的概率是多少，以及我们如何降低其概率？这就是布隆过滤器的工作原理。给定集合 `S = {x[1], x[2], …, x[n]}`：

+   让 `B` 成为一个 `m` 位数组（`m` > 1），初始化为 0。`B` 的元素是 `B[0]`, `B[1]`, `B[2]`, …, `B[m-1]`。存储数组 `B` 所需的内存量仅为存储整个集合 `S` 所需内存量的一小部分。假阳性的概率与位向量（数组 `B`）的大小成反比。

+   让 `{H[1], H[2], …, H[k]}` 成为一组 `k` 个哈希函数。如果 `Hi = a`，则设置 `B[a] = 1`。您可以使用 SHA1、MD5 和 Murmer 作为哈希函数。例如：

    +   `Hi = MD5(x+i)`

    +   `Hi = MD5(x || i)`

+   要检查 `x` 是否 <math alttext="element-of"><mo>∈</mo></math> `S`，检查 `Hi` 的 `B`。所有 `k` 值必须为 `1`。

+   可能会出现假阳性，其中所有 `k` 值均为 `1`，但 `x` 不在 `S` 中。假阳性的概率为：

    <math alttext="dollar-sign left-parenthesis 1 minus left-bracket 1 minus StartFraction 1 Over m EndFraction right-bracket Superscript k n Baseline right-parenthesis Superscript k Baseline almost-equals left-parenthesis 1 minus e Superscript minus k n slash m Baseline right-parenthesis Superscript k dollar-sign"><mrow><msup><mfenced separators="" open="(" close=")"><mn>1</mn> <mo>-</mo> <msup><mfenced separators="" open="[" close="]"><mn>1</mn> <mo>-</mo> <mfrac><mn>1</mn> <mi>m</mi></mfrac></mfenced> <mrow><mi>k</mi><mi>n</mi></mrow></msup></mfenced> <mi>k</mi></msup> <mo>≈</mo> <msup><mfenced separators="" open="(" close=")"><mn>1</mn> <mo>-</mo> <msup><mi>e</mi> <mrow><mo>-</mo><mi>k</mi><mi>n</mi><mo>/</mo><mi>m</mi></mrow></msup></mfenced> <mi>k</mi></msup></mrow></math>

+   什么是最优的哈希函数数量？对于给定的 *m*（选择用于布隆过滤器的位数）和 *n*（数据集的大小），使假阳性概率最小的 *k* 值（哈希函数的数量）是（*ln* 代表“自然对数”）：

    <math alttext="dollar-sign k equals StartFraction m Over n EndFraction l n left-parenthesis 2 right-parenthesis dollar-sign"><mrow><mi>k</mi> <mo>=</mo> <mfrac><mi>m</mi> <mi>n</mi></mfrac> <mi>l</mi> <mi>n</mi> <mrow><mo>(</mo> <mn>2</mn> <mo>)</mo></mrow></mrow></math><math alttext="dollar-sign m equals minus StartFraction n l n left-parenthesis p right-parenthesis Over left-parenthesis l n left-parenthesis 2 right-parenthesis right-parenthesis squared EndFraction dollar-sign"><mrow><mi>m</mi> <mo>=</mo> <mo>-</mo> <mfrac><mrow><mi>n</mi><mi>l</mi><mi>n</mi><mo>(</mo><mi>p</mi><mo>)</mo></mrow> <msup><mrow><mo>(</mo><mi>l</mi><mi>n</mi><mrow><mo>(</mo><mn>2</mn><mo>)</mo></mrow><mo>)</mo></mrow> <mn>2</mn></msup></mfrac></mrow></math>

+   因此，特定位被翻转为 1 的概率是：

    <math alttext="dollar-sign 1 minus left-parenthesis 1 minus StartFraction 1 Over m EndFraction right-parenthesis Superscript k n Baseline almost-equals 1 minus e Superscript minus StartFraction k n Over m EndFraction dollar-sign"><mrow><mn>1</mn> <mo>-</mo> <msup><mfenced separators="" open="(" close=")"><mn>1</mn> <mo>-</mo> <mfrac><mn>1</mn> <mi>m</mi></mfrac></mfenced> <mrow><mi>k</mi><mi>n</mi></mrow></msup> <mo>≈</mo> <mn>1</mn> <mo>-</mo> <msup><mi>e</mi> <mrow><mo>-</mo><mfrac><mrow><mi>k</mi><mi>n</mi></mrow> <mi>m</mi></mfrac></mrow></msup></mrow></math>

接下来，让我们看一个布隆过滤器的例子。

## 一个简单的布隆过滤器示例

这个示例展示了如何在大小为 10 的布隆过滤器（`m = 10`）上插入元素并执行查询，使用三个哈希函数 `H = {H[1], H[2], H[3]}`，其中 `H(x)` 表示这三个哈希函数的结果。我们从一个初始化为 `0` 的长度为 10 位的数组 `B` 开始：

```
Array B:
   initialized:
         index  0  1  2  3  4  5  6  7  8  9
         value  0  0  0  0  0  0  0  0  0  0

   insert element a,  H(a) = (2, 5, 6)
         index  0  1  2  3  4  5  6  7  8  9
         value  0  0  1  0  0  1  1  0  0  0

   insert element b,  H(b) = (1, 5, 8)
         index  0  1  2  3  4  5  6  7  8  9
         value  0  1  1  0  0  1  1  0  1  0

   query element c
   H(c) = (5, 8, 9) => c is not a member (since B[9]=0)

   query element d
   H(d) = (2, 5, 8) => d is a member (False Positive)

   query element e
   H(e) = (1, 2, 6) => e is a member (False Positive)

   query element f
   H(f) = (2, 5, 6) => f is a member (Positive)
```

## Python 中的布隆过滤器

下面的代码段展示了如何在 Python 中创建和使用布隆过滤器（您可以自己编写布隆过滤器库，但通常情况下，如果已经存在库，则应该使用它）：

```
# instantiate BloomFilter with custom settings
>>> from bloom_filter import BloomFilter
>>> bloom = BloomFilter(max_elements=100000, error_rate=0.01)

# Test whether the Bloom-filter has seen a key
>>> "test-key" in bloom
False

# Mark the key as seen
>>> bloom.add("test-key")

# Now check again
>>> "test-key" in bloom
True
```

## 在 PySpark 中使用布隆过滤器

布隆过滤器是一种小型、紧凑且快速的用于集合成员测试的数据结构。它可以用于促进两个 RDD/关系/表的连接，如 `R(K, V)` 和 `S(K, W)`，其中一个关系具有大量记录，而另一个关系具有较少的记录（例如，`R` 可能有 10 亿条记录，而 `S` 可能有 1000 万条记录）。

在键字段 `K` 上执行传统的 `R` 和 `S` 的连接操作将花费很长时间且效率低下。我们可以通过将关系 `S(K, W)` 构建为布隆过滤器，并使用构建的数据结构（使用 Spark 的广播机制）来测试 `R(K, V)` 中的值是否属于其中来加快速度。请注意，为了减少 PySpark 作业的 I/O 成本，我们在映射任务中使用布隆过滤器来进行减少端连接优化。如何实现呢？以下步骤展示了如何在映射器中使用布隆过滤器（表示 `S` 的数据结构），来替代 `R` 和 `S` 之间的连接操作：

1.  构建布隆过滤器，使用两个关系/表中较小的一个。初始化布隆过滤器（创建 `BloomFilter` 的实例），然后使用 `BloomFilter.add()` 来构建数据结构。我们将构建好的布隆过滤器称为 `the_bloom_filter`。

1.  广播构建好的布隆过滤器。使用 `SparkContext.broadcast()` 将 `the_bloom_filter` 广播到所有工作节点，这样它就可以在所有 Spark 转换（包括映射器）中使用：

    ```
    # to broadcast it to all worker nodes for read-only purposes
    sc = spark.sparkContext
    broadcasted_bloom_filter = sc.broadcast(the_bloom_filter)
    ```

1.  在映射器中使用广播对象。现在，我们可以使用布隆过滤器来消除 `R` 中不需要的元素：

    ```
    # e is an element of R(k, b)
    def bloom_filter_function(e):
      # get a copy of the Bloom filter
      the_bloom_filter = broadcasted_bloom_filter.value()
      # use the_bloom_filter for element e
      key = e[0]
      if key in the_bloom_filter:
        return True
      else:
        return False
    #end-def
    ```

我们使用 `bloom_filter_function()` 对 `R=RDD[(K, V)]` 进行处理，只保留键在 `S=RDD[(K, W)]` 中的元素：

```
# R=RDD[(K, V)]
# joined = RDD[(K, V)] where K is in S=RDD[(K, W)]
joined = R.filter(bloom_filter_function)
```

# 总结

本章介绍了在优化连接操作成本至关重要的情况下可以使用的一些设计模式。我向您展示了如何在 MapReduce 范式中实现连接，并呈现了映射端连接，它将连接操作简化为一个简单的映射器和一个对构建字典的查找操作（避免了实际的 `join()` 函数）。这种设计模式完全消除了将任何数据洗牌到减少阶段的需要。然后我向您展示了使用布隆过滤器作为过滤操作的更有效替代方法。正如您所见，通过使用布隆过滤器，可以避免连接操作所导致的洗牌操作。接下来，我们将通过查看特征工程的设计模式来结束本书。
