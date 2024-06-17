# 第二十四章：数据库和 SQL

> 记忆是人类最好的朋友，也是最坏的敌人。
> 
> Gilbert Parker

您需要的数据通常存储在*数据库*中，这些系统专门设计用于高效存储和查询数据。这些大部分是*关系型*数据库，如 PostgreSQL、MySQL 和 SQL Server，它们将数据存储在*表*中，并通常使用结构化查询语言（SQL）进行查询，这是一种用于操作数据的声明性语言。

SQL 是数据科学家工具包中非常重要的一部分。在本章中，我们将创建 NotQuiteABase，这是 Python 实现的一个几乎不是数据库的东西。我们还将介绍 SQL 的基础知识，并展示它们在我们的几乎不是数据库中的工作方式，这是我能想到的最“从头开始”的方式，帮助您理解它们在做什么。我希望在 NotQuiteABase 中解决问题将使您对如何使用 SQL 解决相同问题有一个良好的感觉。

# 创建表和插入

关系数据库是表的集合，以及它们之间的关系。表只是行的集合，与我们一直在处理的一些矩阵类似。然而，表还有一个固定的*模式*，包括列名和列类型。

例如，想象一个包含每个用户的`user_id`、`name`和`num_friends`的`users`数据集：

```py
users = [[0, "Hero", 0],
         [1, "Dunn", 2],
         [2, "Sue", 3],
         [3, "Chi", 3]]
```

在 SQL 中，我们可以这样创建这个表：

```py
CREATE TABLE users (
    user_id INT NOT NULL,
    name VARCHAR(200),
    num_friends INT);
```

注意我们指定了`user_id`和`num_friends`必须是整数（并且`user_id`不允许为`NULL`，表示缺少值，类似于我们的`None`），而`name`应该是长度不超过 200 的字符串。我们将类似地使用 Python 类型。

###### 注意

SQL 几乎完全不区分大小写和缩进。这里的大写和缩进风格是我喜欢的风格。如果您开始学习 SQL，您肯定会遇到其他样式不同的例子。

您可以使用`INSERT`语句插入行：

```py
INSERT INTO users (user_id, name, num_friends) VALUES (0, 'Hero', 0);
```

还要注意 SQL 语句需要以分号结尾，并且 SQL 中字符串需要用单引号括起来。

在 NotQuiteABase 中，您将通过指定类似的模式来创建一个`Table`。然后，要插入一行，您将使用表的`insert`方法，该方法接受一个与表列名顺序相同的`list`行值。

在幕后，我们将每一行都存储为一个从列名到值的`dict`。一个真正的数据库永远不会使用这样浪费空间的表示方法，但这样做将使得 NotQuiteABase 更容易处理。

我们将 NotQuiteABase `Table`实现为一个巨大的类，我们将一次实现一个方法。让我们先把导入和类型别名处理掉：

```py
from typing import Tuple, Sequence, List, Any, Callable, Dict, Iterator
from collections import defaultdict

# A few type aliases we'll use later
Row = Dict[str, Any]                        # A database row
WhereClause = Callable[[Row], bool]         # Predicate for a single row
HavingClause = Callable[[List[Row]], bool]  # Predicate over multiple rows
```

让我们从构造函数开始。要创建一个 NotQuiteABase 表，我们需要传入列名列表和列类型列表，就像您在创建 SQL 数据库中的表时所做的一样：

```py
class Table:
    def __init__(self, columns: List[str], types: List[type]) -> None:
        assert len(columns) == len(types), "# of columns must == # of types"

        self.columns = columns         # Names of columns
        self.types = types             # Data types of columns
        self.rows: List[Row] = []      # (no data yet)
```

我们将添加一个帮助方法来获取列的类型：

```py
    def col2type(self, col: str) -> type:
        idx = self.columns.index(col)      # Find the index of the column,
        return self.types[idx]             # and return its type.
```

我们将添加一个 `insert` 方法来检查您要插入的值是否有效。特别是，您必须提供正确数量的值，并且每个值必须是正确的类型（或 `None`）：

```py
    def insert(self, values: list) -> None:
        # Check for right # of values
        if len(values) != len(self.types):
            raise ValueError(f"You need to provide {len(self.types)} values")

        # Check for right types of values
        for value, typ3 in zip(values, self.types):
            if not isinstance(value, typ3) and value is not None:
                raise TypeError(f"Expected type {typ3} but got {value}")

        # Add the corresponding dict as a "row"
        self.rows.append(dict(zip(self.columns, values)))
```

在实际的 SQL 数据库中，你需要明确指定任何给定列是否允许包含空值 (`None`)；为了简化我们的生活，我们只会说任何列都可以。

我们还将引入一些 dunder 方法，允许我们将表视为一个 `List[Row]`，我们主要用于测试我们的代码：

```py
    def __getitem__(self, idx: int) -> Row:
        return self.rows[idx]

    def __iter__(self) -> Iterator[Row]:
        return iter(self.rows)

    def __len__(self) -> int:
        return len(self.rows)
```

我们将添加一个方法来漂亮地打印我们的表：

```py
    def __repr__(self):
        """Pretty representation of the table: columns then rows"""
        rows = "\n".join(str(row) for row in self.rows)

        return f"{self.columns}\n{rows}"
```

现在我们可以创建我们的 `Users` 表：

```py
# Constructor requires column names and types
users = Table(['user_id', 'name', 'num_friends'], [int, str, int])
users.insert([0, "Hero", 0])
users.insert([1, "Dunn", 2])
users.insert([2, "Sue", 3])
users.insert([3, "Chi", 3])
users.insert([4, "Thor", 3])
users.insert([5, "Clive", 2])
users.insert([6, "Hicks", 3])
users.insert([7, "Devin", 2])
users.insert([8, "Kate", 2])
users.insert([9, "Klein", 3])
users.insert([10, "Jen", 1])
```

如果您现在 `print(users)`，您将看到：

```py
['user_id', 'name', 'num_friends']
{'user_id': 0, 'name': 'Hero', 'num_friends': 0}
{'user_id': 1, 'name': 'Dunn', 'num_friends': 2}
{'user_id': 2, 'name': 'Sue', 'num_friends': 3}
...
```

列表样的 API 使得编写测试变得容易：

```py
assert len(users) == 11
assert users[1]['name'] == 'Dunn'
```

我们还有更多功能要添加。

# 更新

有时您需要更新已经在数据库中的数据。例如，如果 Dunn 又交了一个朋友，您可能需要这样做：

```py
UPDATE users
SET num_friends = 3
WHERE user_id = 1;
```

关键特性包括：

+   要更新哪个表

+   要更新哪些行

+   要更新哪些字段

+   它们的新值应该是什么

我们将在 NotQuiteABase 中添加一个类似的 `update` 方法。它的第一个参数将是一个 `dict`，其键是要更新的列，其值是这些字段的新值。其第二个（可选）参数应该是一个 `predicate`，对于应该更新的行返回 `True`，否则返回 `False`：

```py
    def update(self,
               updates: Dict[str, Any],
               predicate: WhereClause = lambda row: True):
        # First make sure the updates have valid names and types
        for column, new_value in updates.items():
            if column not in self.columns:
                raise ValueError(f"invalid column: {column}")

            typ3 = self.col2type(column)
            if not isinstance(new_value, typ3) and new_value is not None:
                raise TypeError(f"expected type {typ3}, but got {new_value}")

        # Now update
        for row in self.rows:
            if predicate(row):
                for column, new_value in updates.items():
                    row[column] = new_value
```

之后我们可以简单地这样做：

```py
assert users[1]['num_friends'] == 2             # Original value

users.update({'num_friends' : 3},               # Set num_friends = 3
             lambda row: row['user_id'] == 1)   # in rows where user_id == 1

assert users[1]['num_friends'] == 3             # Updated value
```

# 删除

在 SQL 中从表中删除行有两种方法。危险的方式会删除表中的每一行：

```py
DELETE FROM users;
```

较不危险的方式添加了一个 `WHERE` 子句，并且仅删除满足特定条件的行：

```py
DELETE FROM users WHERE user_id = 1;
```

将此功能添加到我们的 `Table` 中很容易：

```py
    def delete(self, predicate: WhereClause = lambda row: True) -> None:
        """Delete all rows matching predicate"""
        self.rows = [row for row in self.rows if not predicate(row)]
```

如果您提供一个 `predicate` 函数（即 `WHERE` 子句），这将仅删除满足它的行。如果您不提供一个，那么默认的 `predicate` 总是返回 `True`，并且您将删除每一行。

例如：

```py
# We're not actually going to run these
users.delete(lambda row: row["user_id"] == 1)  # Deletes rows with user_id == 1
users.delete()                                 # Deletes every row
```

# 选择

通常你不直接检查 SQL 表。相反，您使用 `SELECT` 语句查询它们：

```py
SELECT * FROM users;                            -- get the entire contents
SELECT * FROM users LIMIT 2;                    -- get the first two rows
SELECT user_id FROM users;                      -- only get specific columns
SELECT user_id FROM users WHERE name = 'Dunn';  -- only get specific rows
```

您还可以使用 `SELECT` 语句计算字段：

```py
SELECT LENGTH(name) AS name_length FROM users;
```

我们将给我们的 `Table` 类添加一个 `select` 方法，该方法返回一个新的 `Table`。该方法接受两个可选参数：

+   `keep_columns` 指定结果中要保留的列名。如果您没有提供它，结果将包含所有列。

+   `additional_columns` 是一个字典，其键是新列名，值是指定如何计算新列值的函数。我们将查看这些函数的类型注解来确定新列的类型，因此这些函数需要有注解的返回类型。

如果你没有提供它们中的任何一个，你将简单地得到表的一个副本：

```py
    def select(self,
               keep_columns: List[str] = None,
               additional_columns: Dict[str, Callable] = None) -> 'Table':

        if keep_columns is None:         # If no columns specified,
            keep_columns = self.columns  # return all columns

        if additional_columns is None:
            additional_columns = {}

        # New column names and types
        new_columns = keep_columns + list(additional_columns.keys())
        keep_types = [self.col2type(col) for col in keep_columns]

        # This is how to get the return type from a type annotation.
        # It will crash if `calculation` doesn't have a return type.
        add_types = [calculation.__annotations__['return']
                     for calculation in additional_columns.values()]

        # Create a new table for results
        new_table = Table(new_columns, keep_types + add_types)

        for row in self.rows:
            new_row = [row[column] for column in keep_columns]
            for column_name, calculation in additional_columns.items():
                new_row.append(calculation(row))
            new_table.insert(new_row)

        return new_table
```

###### 注意

还记得在第二章中我们说过类型注解实际上什么也不做吗？好吧，这里是反例。但是看看我们必须经历多么复杂的过程才能得到它们。

我们的`select`返回一个新的`Table`，而典型的 SQL `SELECT`仅产生某种临时结果集（除非您将结果明确插入到表中）。

我们还需要`where`和`limit`方法。这两者都很简单：

```py
    def where(self, predicate: WhereClause = lambda row: True) -> 'Table':
        """Return only the rows that satisfy the supplied predicate"""
        where_table = Table(self.columns, self.types)
        for row in self.rows:
            if predicate(row):
                values = [row[column] for column in self.columns]
                where_table.insert(values)
        return where_table

    def limit(self, num_rows: int) -> 'Table':
        """Return only the first `num_rows` rows"""
        limit_table = Table(self.columns, self.types)
        for i, row in enumerate(self.rows):
            if i >= num_rows:
                break
            values = [row[column] for column in self.columns]
            limit_table.insert(values)
        return limit_table
```

然后我们可以轻松地构造与前面的 SQL 语句相等的 NotQuiteABase 等效语句：

```py
# SELECT * FROM users;
all_users = users.select()
assert len(all_users) == 11

# SELECT * FROM users LIMIT 2;
two_users = users.limit(2)
assert len(two_users) == 2

# SELECT user_id FROM users;
just_ids = users.select(keep_columns=["user_id"])
assert just_ids.columns == ['user_id']

# SELECT user_id FROM users WHERE name = 'Dunn';
dunn_ids = (
    users
    .where(lambda row: row["name"] == "Dunn")
    .select(keep_columns=["user_id"])
)
assert len(dunn_ids) == 1
assert dunn_ids[0] == {"user_id": 1}

# SELECT LENGTH(name) AS name_length FROM users;
def name_length(row) -> int: return len(row["name"])

name_lengths = users.select(keep_columns=[],
                            additional_columns = {"name_length": name_length})
assert name_lengths[0]['name_length'] == len("Hero")
```

注意，对于多行“流畅”查询，我们必须将整个查询包装在括号中。

# GROUP BY

另一个常见的 SQL 操作是`GROUP BY`，它将具有指定列中相同值的行分组在一起，并生成诸如`MIN`、`MAX`、`COUNT`和`SUM`之类的聚合值。

例如，您可能希望找到每个可能的名称长度的用户数和最小的`user_id`：

```py
SELECT LENGTH(name) as name_length,
 MIN(user_id) AS min_user_id,
 COUNT(*) AS num_users
FROM users
GROUP BY LENGTH(name);
```

我们选择的每个字段都需要在`GROUP BY`子句（其中`name_length`是）或聚合计算（`min_user_id`和`num_users`是）中。

SQL 还支持一个`HAVING`子句，其行为类似于`WHERE`子句，只是其过滤器应用于聚合（而`WHERE`将在聚合之前过滤行）。

您可能想知道以特定字母开头的用户名的平均朋友数量，但仅查看其对应平均值大于 1 的字母的结果。（是的，这些示例中有些是人为构造的。）

```py
SELECT SUBSTR(name, 1, 1) AS first_letter,
 AVG(num_friends) AS avg_num_friends
FROM users
GROUP BY SUBSTR(name, 1, 1)
HAVING AVG(num_friends) > 1;
```

###### 注意

不同数据库中用于处理字符串的函数各不相同；一些数据库可能会使用`SUBSTRING`或其他东西。

您还可以计算整体聚合值。在这种情况下，您可以省略`GROUP BY`：

```py
SELECT SUM(user_id) as user_id_sum
FROM users
WHERE user_id > 1;
```

要将此功能添加到 NotQuiteABase 的`Table`中，我们将添加一个`group_by`方法。它接受您要按组分组的列的名称，您要在每个组上运行的聚合函数的字典，以及一个可选的名为`having`的谓词，该谓词对多行进行操作。

然后执行以下步骤：

1.  创建一个`defaultdict`来将`tuple`（按分组值）映射到行（包含分组值的行）。请记住，您不能使用列表作为`dict`的键；您必须使用元组。

1.  遍历表的行，填充`defaultdict`。

1.  创建一个具有正确输出列的新表。

1.  遍历`defaultdict`并填充输出表，应用`having`过滤器（如果有）。

```py
    def group_by(self,
                 group_by_columns: List[str],
                 aggregates: Dict[str, Callable],
                 having: HavingClause = lambda group: True) -> 'Table':

        grouped_rows = defaultdict(list)

        # Populate groups
        for row in self.rows:
            key = tuple(row[column] for column in group_by_columns)
            grouped_rows[key].append(row)

        # Result table consists of group_by columns and aggregates
        new_columns = group_by_columns + list(aggregates.keys())
        group_by_types = [self.col2type(col) for col in group_by_columns]
        aggregate_types = [agg.__annotations__['return']
                           for agg in aggregates.values()]
        result_table = Table(new_columns, group_by_types + aggregate_types)

        for key, rows in grouped_rows.items():
            if having(rows):
                new_row = list(key)
                for aggregate_name, aggregate_fn in aggregates.items():
                    new_row.append(aggregate_fn(rows))
                result_table.insert(new_row)

        return result_table
```

###### 注意

实际的数据库几乎肯定会以更有效的方式执行此操作。

同样，让我们看看如何执行与前面的 SQL 语句等效的操作。`name_length`指标是：

```py
def min_user_id(rows) -> int:
    return min(row["user_id"] for row in rows)

def length(rows) -> int:
    return len(rows)

stats_by_length = (
    users
    .select(additional_columns={"name_length" : name_length})
    .group_by(group_by_columns=["name_length"],
              aggregates={"min_user_id" : min_user_id,
                          "num_users" : length})
)
```

`first_letter`指标是：

```py
def first_letter_of_name(row: Row) -> str:
    return row["name"][0] if row["name"] else ""

def average_num_friends(rows: List[Row]) -> float:
    return sum(row["num_friends"] for row in rows) / len(rows)

def enough_friends(rows: List[Row]) -> bool:
    return average_num_friends(rows) > 1

avg_friends_by_letter = (
    users
    .select(additional_columns={'first_letter' : first_letter_of_name})
    .group_by(group_by_columns=['first_letter'],
              aggregates={"avg_num_friends" : average_num_friends},
              having=enough_friends)
)
```

`user_id_sum`是：

```py
def sum_user_ids(rows: List[Row]) -> int:
    return sum(row["user_id"] for row in rows)

user_id_sum = (
    users
    .where(lambda row: row["user_id"] > 1)
    .group_by(group_by_columns=[],
              aggregates={ "user_id_sum" : sum_user_ids })
)
```

# ORDER BY

经常，您可能希望对结果进行排序。例如，您可能希望知道用户的（按字母顺序）前两个名称：

```py
SELECT * FROM users
ORDER BY name
LIMIT 2;
```

这很容易通过给我们的`Table`添加一个`order_by`方法来实现，该方法接受一个`order`函数来实现：

```py
    def order_by(self, order: Callable[[Row], Any]) -> 'Table':
        new_table = self.select()       # make a copy
        new_table.rows.sort(key=order)
        return new_table
```

然后我们可以像这样使用它们：

```py
friendliest_letters = (
    avg_friends_by_letter
    .order_by(lambda row: -row["avg_num_friends"])
    .limit(4)
)
```

SQL 的`ORDER BY`允许您为每个排序字段指定`ASC`（升序）或`DESC`（降序）；在这里，我们必须将其嵌入到我们的`order`函数中。

# JOIN

关系型数据库表通常是*规范化*的，这意味着它们被组织成最小化冗余。例如，当我们在 Python 中处理用户的兴趣时，我们可以为每个用户分配一个包含其兴趣的`list`。

SQL 表通常不能包含列表，所以典型的解决方案是创建第二个表，称为`user_interests`，包含`user_id`和`interest`之间的一对多关系。在 SQL 中，你可以这样做：

```py
CREATE TABLE user_interests (
    user_id INT NOT NULL,
    interest VARCHAR(100) NOT NULL
);
```

而在 NotQuiteABase 中，你需要创建这样一个表：

```py
user_interests = Table(['user_id', 'interest'], [int, str])
user_interests.insert([0, "SQL"])
user_interests.insert([0, "NoSQL"])
user_interests.insert([2, "SQL"])
user_interests.insert([2, "MySQL"])
```

###### 注意

仍然存在大量冗余 —— 兴趣“SQL”存储在两个不同的地方。在实际数据库中，您可能会将`user_id`和`interest_id`存储在`user_interests`表中，然后创建第三个表`interests`，将`interest_id`映射到`interest`，这样您只需存储兴趣名称一次。但这会使我们的示例变得比必要的复杂。

当我们的数据分布在不同的表中时，我们如何分析它？通过将表进行`JOIN`。`JOIN`将左表中的行与右表中相应的行组合在一起，其中“相应”的含义基于我们如何指定连接的方式。

例如，要查找对 SQL 感兴趣的用户，你会这样查询：

```py
SELECT users.name
FROM users
JOIN user_interests
ON users.user_id = user_interests.user_id
WHERE user_interests.interest = 'SQL'
```

`JOIN`指示，对于`users`中的每一行，我们应该查看`user_id`并将该行与包含相同`user_id`的`user_interests`中的每一行关联起来。

注意，我们必须指定要`JOIN`的表和要`ON`连接的列。这是一个`INNER JOIN`，它根据指定的连接条件返回匹配的行组合（仅限匹配的行组合）。

还有一种`LEFT JOIN`，除了匹配行的组合外，还返回每个左表行的未匹配行（在这种情况下，右表应该出现的字段都是`NULL`）。

使用`LEFT JOIN`，很容易统计每个用户的兴趣数量：

```py
SELECT users.id, COUNT(user_interests.interest) AS num_interests
FROM users
LEFT JOIN user_interests
ON users.user_id = user_interests.user_id
```

`LEFT JOIN`确保没有兴趣的用户仍然在连接数据集中具有行（`user_interests`字段的值为`NULL`），而`COUNT`仅计算非`NULL`值。

NotQuiteABase 的`join`实现将更为严格 —— 它仅仅在两个表中存在共同列时进行连接。即便如此，编写起来也不是件简单的事情：

```py
    def join(self, other_table: 'Table', left_join: bool = False) -> 'Table':

        join_on_columns = [c for c in self.columns           # columns in
                           if c in other_table.columns]      # both tables

        additional_columns = [c for c in other_table.columns # columns only
                              if c not in join_on_columns]   # in right table

        # all columns from left table + additional_columns from right table
        new_columns = self.columns + additional_columns
        new_types = self.types + [other_table.col2type(col)
                                  for col in additional_columns]

        join_table = Table(new_columns, new_types)

        for row in self.rows:
            def is_join(other_row):
                return all(other_row[c] == row[c] for c in join_on_columns)

            other_rows = other_table.where(is_join).rows

            # Each other row that matches this one produces a result row.
            for other_row in other_rows:
                join_table.insert([row[c] for c in self.columns] +
                                  [other_row[c] for c in additional_columns])

            # If no rows match and it's a left join, output with Nones.
            if left_join and not other_rows:
                join_table.insert([row[c] for c in self.columns] +
                                  [None for c in additional_columns])

        return join_table
```

因此，我们可以找到对 SQL 感兴趣的用户：

```py
sql_users = (
    users
    .join(user_interests)
    .where(lambda row: row["interest"] == "SQL")
    .select(keep_columns=["name"])
)
```

我们可以通过以下方式获得兴趣计数：

```py
def count_interests(rows: List[Row]) -> int:
    """counts how many rows have non-None interests"""
    return len([row for row in rows if row["interest"] is not None])

user_interest_counts = (
    users
    .join(user_interests, left_join=True)
    .group_by(group_by_columns=["user_id"],
              aggregates={"num_interests" : count_interests })
)
```

在 SQL 中，还有一种`RIGHT JOIN`，它保留来自右表且没有匹配的行，还有一种`FULL OUTER JOIN`，它保留来自两个表且没有匹配的行。我们不会实现其中任何一种。

# 子查询

在 SQL 中，您可以从（和`JOIN`）查询的结果中`SELECT`，就像它们是表一样。因此，如果您想找到任何对 SQL 感兴趣的人中最小的`user_id`，您可以使用子查询。（当然，您也可以使用`JOIN`执行相同的计算，但这不会说明子查询。）

```py
SELECT MIN(user_id) AS min_user_id FROM
(SELECT user_id FROM user_interests WHERE interest = 'SQL') sql_interests;
```

鉴于我们设计的 NotQuiteABase 的方式，我们可以免费获得这些功能。（我们的查询结果是实际的表。）

```py
likes_sql_user_ids = (
    user_interests
    .where(lambda row: row["interest"] == "SQL")
    .select(keep_columns=['user_id'])
)

likes_sql_user_ids.group_by(group_by_columns=[],
                            aggregates={ "min_user_id" : min_user_id })
```

# 索引

要查找包含特定值（比如`name`为“Hero”的行），NotQuiteABase 必须检查表中的每一行。如果表中有很多行，这可能需要很长时间。

类似地，我们的`join`算法非常低效。对于左表中的每一行，它都要检查右表中的每一行是否匹配。对于两个大表来说，这可能永远都需要很长时间。

此外，您经常希望对某些列应用约束。例如，在您的`users`表中，您可能不希望允许两个不同的用户具有相同的`user_id`。

索引解决了所有这些问题。如果`user_interests`表上有一个关于`user_id`的索引，智能的`join`算法可以直接找到匹配项，而不必扫描整个表。如果`users`表上有一个关于`user_id`的“唯一”索引，如果尝试插入重复项，则会收到错误提示。

数据库中的每个表可以有一个或多个索引，这些索引允许您通过关键列快速查找行，在表之间有效地进行连接，并在列或列组合上强制唯一约束。

良好设计和使用索引有点像黑魔法（这取决于具体的数据库有所不同），但是如果您经常进行数据库工作，学习这些知识是值得的。

# 查询优化

回顾查询以查找所有对 SQL 感兴趣的用户：

```py
SELECT users.name
FROM users
JOIN user_interests
ON users.user_id = user_interests.user_id
WHERE user_interests.interest = 'SQL'
```

在 NotQuiteABase 中有（至少）两种不同的方法来编写此查询。您可以在执行连接之前过滤`user_interests`表：

```py
(
    user_interests
    .where(lambda row: row["interest"] == "SQL")
    .join(users)
    .select(["name"])
)
```

或者您可以过滤连接的结果：

```py
(
    user_interests
    .join(users)
    .where(lambda row: row["interest"] == "SQL")
    .select(["name"])
)
```

无论哪种方式，最终的结果都是相同的，但是在连接之前过滤几乎肯定更有效，因为在这种情况下，`join`操作的行数要少得多。

在 SQL 中，您通常不必担心这个问题。您可以“声明”您想要的结果，然后由查询引擎来执行它们（并有效地使用索引）。

# NoSQL

数据库的一个最新趋势是向非关系型的“NoSQL”数据库发展，它们不以表格形式表示数据。例如，MongoDB 是一种流行的无模式数据库，其元素是任意复杂的 JSON 文档，而不是行。

有列数据库，它们将数据存储在列中而不是行中（当数据具有许多列但查询只需少数列时很好），键/值存储优化了通过键检索单个（复杂）值的数据库，用于存储和遍历图形的数据库，优化用于跨多个数据中心运行的数据库，专为内存运行而设计的数据库，用于存储时间序列数据的数据库等等。

明天的热门可能甚至现在都不存在，所以我不能做更多的事情，只能告诉您 NoSQL 是一种事物。所以现在您知道了。它是一种事物。

# 进一步探索

+   如果你想要下载一个关系型数据库来玩玩，[SQLite](http://www.sqlite.org) 快速且小巧，而 [MySQL](http://www.mysql.com) 和 [PostgreSQL](http://www.postgresql.org) 则更大且功能丰富。所有这些都是免费的，并且有大量文档支持。

+   如果你想探索 NoSQL，[MongoDB](http://www.mongodb.org) 非常简单入门，这既是一种福音也有点儿“诅咒”。它的文档也相当不错。

+   [NoSQL 的维基百科文章](http://en.wikipedia.org/wiki/NoSQL)几乎可以肯定地包含了在这本书写作时甚至都不存在的数据库链接。
