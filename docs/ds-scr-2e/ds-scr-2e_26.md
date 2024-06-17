# 第二十五章：MapReduce

> 未来已经到来，只是尚未均匀分布。
> 
> 威廉·吉布森

MapReduce 是一种在大型数据集上执行并行处理的编程模型。虽然它是一种强大的技术，但其基础相对简单。

想象我们有一系列希望以某种方式处理的项目。例如，这些项目可能是网站日志、各种书籍的文本、图像文件或其他任何内容。MapReduce 算法的基本版本包括以下步骤：

1.  使用`mapper`函数将每个项转换为零个或多个键/值对。（通常称为`map`函数，但 Python 已经有一个名为`map`的函数，我们不需要混淆这两者。）

1.  收集所有具有相同键的对。

1.  对每个分组值集合使用`reducer`函数，以生成相应键的输出值。

###### 注意

MapReduce 已经有些过时了，以至于我考虑从第二版中删除这一章节。但我决定它仍然是一个有趣的主题，所以最终我还是留了下来（显然）。

这些都有点抽象，让我们看一个具体的例子。数据科学中几乎没有绝对规则，但其中一个规则是，您的第一个 MapReduce 示例必须涉及单词计数。

# 示例：单词计数

DataSciencester 已经发展到数百万用户！这对于您的工作安全来说是好事，但也使得例行分析略微更加困难。

例如，您的内容副总裁想知道人们在其状态更新中谈论的内容。作为第一次尝试，您决定计算出现的单词，以便可以准备一份最频繁出现单词的报告。

当您只有几百个用户时，这样做很简单：

```py
from typing import List
from collections import Counter

def tokenize(document: str) -> List[str]:
    """Just split on whitespace"""
    return document.split()

def word_count_old(documents: List[str]):
    """Word count not using MapReduce"""
    return Counter(word
        for document in documents
        for word in tokenize(document))
```

当您有数百万用户时，`documents`（状态更新）的集合突然变得太大，无法放入您的计算机中。如果您能将其适应 MapReduce 模型，您可以使用工程师们实施的一些“大数据”基础设施。

首先，我们需要一个将文档转换为键/值对序列的函数。我们希望输出按单词分组，这意味着键应该是单词。对于每个单词，我们只需发出值`1`来表示该对应单词的出现次数为一次：

```py
from typing import Iterator, Tuple

def wc_mapper(document: str) -> Iterator[Tuple[str, int]]:
    """For each word in the document, emit (word, 1)"""
    for word in tokenize(document):
        yield (word, 1)
```

暂时跳过“管道”步骤 2，想象一下对于某个单词，我们已经收集到了我们发出的相应计数列表。为了生成该单词的总计数，我们只需：

```py
from typing import Iterable

def wc_reducer(word: str,
               counts: Iterable[int]) -> Iterator[Tuple[str, int]]:
    """Sum up the counts for a word"""
    yield (word, sum(counts))
```

回到步骤 2，现在我们需要收集`wc_mapper`的结果并将其提供给`wc_reducer`。让我们考虑如何在一台计算机上完成这项工作：

```py
from collections import defaultdict

def word_count(documents: List[str]) -> List[Tuple[str, int]]:
    """Count the words in the input documents using MapReduce"""

    collector = defaultdict(list)  # To store grouped values

    for document in documents:
        for word, count in wc_mapper(document):
            collector[word].append(count)

    return [output
            for word, counts in collector.items()
            for output in wc_reducer(word, counts)]
```

假设我们有三个文档`["data science", "big data", "science fiction"]`。

然后，将`wc_mapper`应用于第一个文档，产生两对`("data", 1)`和`("science", 1)`。在我们处理完所有三个文档之后，`collector`包含：

```py
{"data" : [1, 1],
 "science" : [1, 1],
 "big" : [1],
 "fiction" : [1]}
```

然后，`wc_reducer`生成每个单词的计数：

```py
[("data", 2), ("science", 2), ("big", 1), ("fiction", 1)]
```

# 为什么要使用 MapReduce？

正如前面提到的，MapReduce 的主要优势在于它允许我们通过将处理移到数据上来分布计算。假设我们想要跨数十亿文档进行单词计数。

我们最初的（非 MapReduce）方法要求处理文档的机器能够访问每个文档。这意味着所有文档都需要在该机器上存储，或者在处理过程中将其传输到该机器上。更重要的是，这意味着机器一次只能处理一个文档。

###### 注意

如果具有多个核心并且代码已重写以利用它们，可能可以同时处理几个。但即便如此，所有文档仍然必须*到达*该机器。

现在假设我们的数十亿文档分散在 100 台机器上。有了正确的基础设施（并且忽略某些细节），我们可以执行以下操作：

+   让每台机器在其文档上运行映射器，生成大量的键/值对。

+   将这些键/值对分发到多个“减少”机器，确保与任何给定键对应的所有对最终都在同一台机器上。

+   让每个减少机器按键分组然后对每组值运行减少器。

+   返回每个（键，输出）对。

这其中令人惊奇的是它的水平扩展能力。如果我们将机器数量翻倍，那么（忽略运行 MapReduce 系统的某些固定成本），我们的计算速度应该大约加快一倍。每台映射器机器只需完成一半的工作量，并且（假设有足够多的不同键来进一步分发减少器的工作）减少器机器也是如此。

# 更普遍的 MapReduce

如果你仔细想一想，你会发现前面示例中所有与计数特定单词有关的代码都包含在`wc_mapper`和`wc_reducer`函数中。这意味着只需做出一些更改，我们就可以得到一个更通用的框架（仍然在单台机器上运行）。

我们可以使用通用类型完全类型注释我们的`map_reduce`函数，但这在教学上可能会有些混乱，因此在本章中，我们对类型注释要更加随意：

```py
from typing import Callable, Iterable, Any, Tuple

# A key/value pair is just a 2-tuple
KV = Tuple[Any, Any]

# A Mapper is a function that returns an Iterable of key/value pairs
Mapper = Callable[..., Iterable[KV]]

# A Reducer is a function that takes a key and an iterable of values
# and returns a key/value pair
Reducer = Callable[[Any, Iterable], KV]
```

现在我们可以编写一个通用的`map_reduce`函数：

```py
def map_reduce(inputs: Iterable,
               mapper: Mapper,
               reducer: Reducer) -> List[KV]:
    """Run MapReduce on the inputs using mapper and reducer"""
    collector = defaultdict(list)

    for input in inputs:
        for key, value in mapper(input):
            collector[key].append(value)

    return [output
            for key, values in collector.items()
            for output in reducer(key, values)]
```

然后，我们可以简单地通过以下方式计算单词数：

```py
word_counts = map_reduce(documents, wc_mapper, wc_reducer)
```

这使我们能够灵活地解决各种问题。

在继续之前，请注意`wc_reducer`仅仅是对每个键对应的值求和。这种聚合是如此普遍，以至于值得将其抽象出来：

```py
def values_reducer(values_fn: Callable) -> Reducer:
    """Return a reducer that just applies values_fn to its values"""
    def reduce(key, values: Iterable) -> KV:
        return (key, values_fn(values))

    return reduce
```

然后我们可以轻松创建：

```py
sum_reducer = values_reducer(sum)
max_reducer = values_reducer(max)
min_reducer = values_reducer(min)
count_distinct_reducer = values_reducer(lambda values: len(set(values)))

assert sum_reducer("key", [1, 2, 3, 3]) == ("key", 9)
assert min_reducer("key", [1, 2, 3, 3]) == ("key", 1)
assert max_reducer("key", [1, 2, 3, 3]) == ("key", 3)
assert count_distinct_reducer("key", [1, 2, 3, 3]) == ("key", 3)
```

等等。

# 示例：分析状态更新

内容 VP 对单词计数印象深刻，并询问您可以从人们的状态更新中学到什么其他内容。您设法提取出一个看起来像这样的状态更新数据集：

```py
status_updates = [
    {"id": 2,
     "username" : "joelgrus",
     "text" : "Should I write a second edition of my data science book?",
     "created_at" : datetime.datetime(2018, 2, 21, 11, 47, 0),
     "liked_by" : ["data_guy", "data_gal", "mike"] },
     # ...
]
```

假设我们需要弄清楚人们在一周中哪一天最常谈论数据科学。为了找到这一点，我们只需计算每周的数据科学更新次数。这意味着我们需要按周几分组，这就是我们的关键。如果我们对每个包含“数据科学”的更新发出值 `1`，我们可以简单地通过 `sum` 得到总数：

```py
def data_science_day_mapper(status_update: dict) -> Iterable:
    """Yields (day_of_week, 1) if status_update contains "data science" """
    if "data science" in status_update["text"].lower():
        day_of_week = status_update["created_at"].weekday()
        yield (day_of_week, 1)

data_science_days = map_reduce(status_updates,
                               data_science_day_mapper,
                               sum_reducer)
```

作为一个稍微复杂的例子，想象一下我们需要找出每个用户在其状态更新中最常用的单词。对于 `mapper`，有三种可能的方法会脱颖而出：

+   将用户名放入键中；将单词和计数放入值中。

+   将单词放入键中；将用户名和计数放入值中。

+   将用户名和单词放入键中；将计数放入值中。

如果你再仔细考虑一下，我们肯定想要按 `username` 进行分组，因为我们希望单独考虑每个人的话语。而且我们不想按 `word` 进行分组，因为我们的减少器需要查看每个人的所有单词以找出哪个最受欢迎。这意味着第一个选项是正确的选择：

```py
def words_per_user_mapper(status_update: dict):
    user = status_update["username"]
    for word in tokenize(status_update["text"]):
        yield (user, (word, 1))

def most_popular_word_reducer(user: str,
                              words_and_counts: Iterable[KV]):
    """
 Given a sequence of (word, count) pairs,
 return the word with the highest total count
 """
    word_counts = Counter()
    for word, count in words_and_counts:
        word_counts[word] += count

    word, count = word_counts.most_common(1)[0]

    yield (user, (word, count))

user_words = map_reduce(status_updates,
                        words_per_user_mapper,
                        most_popular_word_reducer)
```

或者我们可以找出每个用户的不同状态点赞者数量：

```py
def liker_mapper(status_update: dict):
    user = status_update["username"]
    for liker in status_update["liked_by"]:
        yield (user, liker)

distinct_likers_per_user = map_reduce(status_updates,
                                      liker_mapper,
                                      count_distinct_reducer)
```

# 示例：矩阵乘法

从 “矩阵乘法” 回想一下，给定一个 `[n, m]` 矩阵 `A` 和一个 `[m, k]` 矩阵 `B`，我们可以将它们相乘得到一个 `[n, k]` 矩阵 `C`，其中 `C` 中第 `i` 行第 `j` 列的元素由以下给出：

```py
C[i][j] = sum(A[i][x] * B[x][j] for x in range(m))
```

只要我们像我们一直在做的那样，将我们的矩阵表示为列表的列表，这就起作用了。

但是大矩阵有时是 *sparse* 的，这意味着它们的大多数元素等于 0。对于大稀疏矩阵，列表列表可能是非常浪费的表示方式。更紧凑的表示法仅存储具有非零值的位置：

```py
from typing import NamedTuple

class Entry(NamedTuple):
    name: str
    i: int
    j: int
    value: float
```

例如，一个 10 亿 × 10 亿的矩阵有 1 *quintillion* 个条目，这在计算机上存储起来并不容易。但如果每行只有几个非零条目，这种替代表示法则小得多。

在这种表示法给定的情况下，我们发现可以使用 MapReduce 以分布方式执行矩阵乘法。

为了激励我们的算法，请注意每个元素 `A[i][j]` 仅用于计算 `C` 的第 `i` 行的元素，每个元素 `B[i][j]` 仅用于计算 `C` 的第 `j` 列的元素。我们的目标是使我们的 `reducer` 的每个输出成为 `C` 的一个单一条目，这意味着我们的 `mapper` 需要发出标识 `C` 的单个条目的键。这建议以下操作：

```py
def matrix_multiply_mapper(num_rows_a: int, num_cols_b: int) -> Mapper:
    # C[x][y] = A[x][0] * B[0][y] + ... + A[x][m] * B[m][y]
    #
    # so an element A[i][j] goes into every C[i][y] with coef B[j][y]
    # and an element B[i][j] goes into every C[x][j] with coef A[x][i]
    def mapper(entry: Entry):
        if entry.name == "A":
            for y in range(num_cols_b):
                key = (entry.i, y)              # which element of C
                value = (entry.j, entry.value)  # which entry in the sum
                yield (key, value)
        else:
            for x in range(num_rows_a):
                key = (x, entry.j)              # which element of C
                value = (entry.i, entry.value)  # which entry in the sum
                yield (key, value)

    return mapper
```

然后：

```py
def matrix_multiply_reducer(key: Tuple[int, int],
                            indexed_values: Iterable[Tuple[int, int]]):
    results_by_index = defaultdict(list)

    for index, value in indexed_values:
        results_by_index[index].append(value)

    # Multiply the values for positions with two values
    # (one from A, and one from B) and sum them up.
    sumproduct = sum(values[0] * values[1]
                     for values in results_by_index.values()
                     if len(values) == 2)

    if sumproduct != 0.0:
        yield (key, sumproduct)
```

例如，如果你有这两个矩阵：

```py
A = [[3, 2, 0],
     [0, 0, 0]]

B = [[4, -1, 0],
     [10, 0, 0],
     [0, 0, 0]]
```

你可以将它们重写为元组：

```py
entries = [Entry("A", 0, 0, 3), Entry("A", 0, 1,  2), Entry("B", 0, 0, 4),
           Entry("B", 0, 1, -1), Entry("B", 1, 0, 10)]

mapper = matrix_multiply_mapper(num_rows_a=2, num_cols_b=3)
reducer = matrix_multiply_reducer

# Product should be [[32, -3, 0], [0, 0, 0]].
# So it should have two entries.
assert (set(map_reduce(entries, mapper, reducer)) ==
        {((0, 1), -3), ((0, 0), 32)})
```

在如此小的矩阵上这并不是非常有趣，但是如果你有数百万行和数百万列，它可能会帮助你很多。

# 一则：组合器

你可能已经注意到，我们的许多 mapper 看起来包含了大量额外的信息。例如，在计算单词数量时，我们可以发射 `(word, None)` 并且只计算长度，而不是发射 `(word, 1)` 并对值求和。

我们没有这样做的一个原因是，在分布式设置中，有时我们想要使用*组合器*来减少必须从一台机器传输到另一台机器的数据量。如果我们的某个 mapper 机器看到单词 *data* 出现了 500 次，我们可以告诉它将 `("data", 1)` 的 500 个实例合并成一个 `("data", 500)`，然后再交给 reducer 处理。这样可以减少传输的数据量，从而使我们的算法速度显著提高。

由于我们写的 reducer 的方式，它将正确处理这些合并的数据。（如果我们使用 `len` 写的话，就不会。）

# 深入探讨

+   正如我所说的，与我写第一版时相比，MapReduce 现在似乎不那么流行了。也许不值得投入大量时间。

+   尽管如此，最广泛使用的 MapReduce 系统是[Hadoop](http://hadoop.apache.org)。有各种商业和非商业的发行版，以及一个庞大的与 Hadoop 相关的工具生态系统。

+   Amazon.com 提供了一个[Elastic MapReduce](http://aws.amazon.com/elasticmapreduce/)服务，可能比自己搭建集群更容易。

+   Hadoop 作业通常具有较高的延迟，这使得它们在“实时”分析方面表现不佳。这些工作负载的流行选择是[Spark](http://spark.apache.org/)，它可以像 MapReduce 一样运行。
