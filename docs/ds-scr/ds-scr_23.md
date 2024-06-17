# 第22章 网络分析

> 你与你周围所有事物的连接实际上定义了你是谁。
> 
> Aaron O’Connell

许多有趣的数据问题可以通过*网络*的方式进行有益的思考，网络由某种类型的*节点*和连接它们的*边*组成。

例如，你的Facebook朋友形成一个网络的节点，其边是友谊关系。一个不那么明显的例子是互联网本身，其中每个网页是一个节点，每个从一个页面到另一个页面的超链接是一条边。

Facebook友谊是相互的——如果我在Facebook上是你的朋友，那么必然你也是我的朋友。在这种情况下，我们称这些边是*无向*的。而超链接则不是——我的网站链接到*whitehouse.gov*，但（出于我无法理解的原因）*whitehouse.gov*拒绝链接到我的网站。我们称这些类型的边为*有向*的。我们将研究这两种类型的网络。

# 中介中心性

在[第1章](ch01.html#introduction)中，我们通过计算每个用户拥有的朋友数量来计算DataSciencester网络中的关键连接者。现在我们有足够的机制来看看其他方法。我们将使用相同的网络，但现在我们将使用`NamedTuple`来处理数据。

回想一下，网络（[图22-1](#datasciencester_network_ch21)）包括用户：

```py
from typing import NamedTuple

class User(NamedTuple):
    id: int
    name: str

users = [User(0, "Hero"), User(1, "Dunn"), User(2, "Sue"), User(3, "Chi"),
         User(4, "Thor"), User(5, "Clive"), User(6, "Hicks"),
         User(7, "Devin"), User(8, "Kate"), User(9, "Klein")]
```

以及友谊：

```py
friend_pairs = [(0, 1), (0, 2), (1, 2), (1, 3), (2, 3), (3, 4),
                (4, 5), (5, 6), (5, 7), (6, 8), (7, 8), (8, 9)]
```

![DataSciencester网络。](assets/dsf2_0101.png)

###### 图22-1\. DataSciencester网络

友谊将更容易作为一个`dict`来处理：

```py
from typing import Dict, List

# type alias for keeping track of Friendships
Friendships = Dict[int, List[int]]

friendships: Friendships = {user.id: [] for user in users}

for i, j in friend_pairs:
    friendships[i].append(j)
    friendships[j].append(i)

assert friendships[4] == [3, 5]
assert friendships[8] == [6, 7, 9]
```

当我们离开时，我们对我们关于*度中心性*的概念并不满意，这与我们对网络中关键连接者的直觉并不完全一致。

另一种度量标准是*中介中心性*，它识别频繁出现在其他人对之间最短路径上的人。特别地，节点*i*的中介中心性通过为每对节点*j*和*k*添加上通过*i*的最短路径的比例来计算。

也就是说，要弄清楚Thor的中介中心性，我们需要计算所有不是Thor的人之间所有最短路径。然后我们需要计算有多少条这些最短路径通过Thor。例如，Chi（`id` 3）和Clive（`id` 5）之间唯一的最短路径通过Thor，而Hero（`id` 0）和Chi（`id` 3）之间的两条最短路径都不通过Thor。

因此，作为第一步，我们需要找出所有人之间的最短路径。有一些非常复杂的算法可以高效地完成这个任务，但是（几乎总是如此），我们将使用一种效率较低但更易于理解的算法。

这个算法（广度优先搜索的实现）是本书中比较复杂的算法之一，所以让我们仔细讨论一下它：

1.  我们的目标是一个函数，它接受一个`from_user`，并找到到每个其他用户的*所有*最短路径。

1.  我们将路径表示为用户ID的`list`。因为每条路径都从`from_user`开始，我们不会在列表中包含她的ID。这意味着表示路径的列表长度将是路径本身的长度。

1.  我们将维护一个名为`shortest_paths_to`的字典，其中键是用户ID，值是以指定ID结尾的路径列表。如果有唯一的最短路径，列表将只包含该路径。如果有多条最短路径，则列表将包含所有这些路径。

1.  我们还会维护一个称为`frontier`的队列，按照我们希望探索它们的顺序包含我们想要探索的用户。我们将它们存储为对`(prev_user, user)`，这样我们就知道如何到达每一个用户。我们将队列初始化为`from_user`的所有邻居。（我们还没有讨论过队列，它们是优化了“添加到末尾”和“从前面删除”的数据结构。在Python中，它们被实现为`collections.deque`，实际上是一个双端队列。）

1.  在探索图形时，每当我们发现新的邻居，而我们还不知道到达它们的最短路径时，我们将它们添加到队列的末尾以供稍后探索，当前用户为`prev_user`。

1.  当我们从队列中取出一个用户，并且我们以前从未遇到过该用户时，我们肯定找到了一条或多条最短路径——每条最短路径到`prev_user`再添加一步。

1.  当我们从队列中取出一个用户，并且我们之前*遇到过*该用户时，那么要么我们找到了另一条最短路径（在这种情况下，我们应该添加它），要么我们找到了一条更长的路径（在这种情况下，我们不应该添加）。

1.  当队列中没有更多的用户时，我们已经探索了整个图形（或者至少是从起始用户可达的部分），我们完成了。

我们可以把所有这些组合成一个（很大的）函数：

```py
from collections import deque

Path = List[int]

def shortest_paths_from(from_user_id: int,
                        friendships: Friendships) -> Dict[int, List[Path]]:
    # A dictionary from user_id to *all* shortest paths to that user.
    shortest_paths_to: Dict[int, List[Path]] = {from_user_id: [[]]}

    # A queue of (previous user, next user) that we need to check.
    # Starts out with all pairs (from_user, friend_of_from_user).
    frontier = deque((from_user_id, friend_id)
                     for friend_id in friendships[from_user_id])

    # Keep going until we empty the queue.
    while frontier:
        # Remove the pair that's next in the queue.
        prev_user_id, user_id = frontier.popleft()

        # Because of the way we're adding to the queue,
        # necessarily we already know some shortest paths to prev_user.
        paths_to_prev_user = shortest_paths_to[prev_user_id]
        new_paths_to_user = [path + [user_id] for path in paths_to_prev_user]

        # It's possible we already know a shortest path to user_id.
        old_paths_to_user = shortest_paths_to.get(user_id, [])

        # What's the shortest path to here that we've seen so far?
        if old_paths_to_user:
            min_path_length = len(old_paths_to_user[0])
        else:
            min_path_length = float('inf')

        # Only keep paths that aren't too long and are actually new.
        new_paths_to_user = [path
                             for path in new_paths_to_user
                             if len(path) <= min_path_length
                             and path not in old_paths_to_user]

        shortest_paths_to[user_id] = old_paths_to_user + new_paths_to_user

        # Add never-seen neighbors to the frontier.
        frontier.extend((user_id, friend_id)
                        for friend_id in friendships[user_id]
                        if friend_id not in shortest_paths_to)

    return shortest_paths_to
```

现在让我们计算所有的最短路径：

```py
# For each from_user, for each to_user, a list of shortest paths.
shortest_paths = {user.id: shortest_paths_from(user.id, friendships)
                  for user in users}
```

现在我们终于可以计算介数中心性了。对于每对节点*i*和*j*，我们知道从*i*到*j*的*n*条最短路径。然后，对于每条路径，我们只需将1/n添加到该路径上每个节点的中心性：

```py
betweenness_centrality = {user.id: 0.0 for user in users}

for source in users:
    for target_id, paths in shortest_paths[source.id].items():
        if source.id < target_id:      # don't double count
            num_paths = len(paths)     # how many shortest paths?
            contrib = 1 / num_paths    # contribution to centrality
            for path in paths:
                for between_id in path:
                    if between_id not in [source.id, target_id]:
                        betweenness_centrality[between_id] += contrib
```

如[图22-2](#network_sized_by_betweenness)所示，用户0和9的中心性为0（因为它们都不在任何其他用户之间的最短路径上），而3、4和5都具有很高的中心性（因为它们都位于许多最短路径上）。

![DataSciencester网络按介数中心性大小排序。](assets/dsf2_2202.png)

###### 图22-2\. DataSciencester网络按介数中心性大小排序

###### 注意

通常中心性数值本身并不那么有意义。我们关心的是每个节点的数值与其他节点的数值相比如何。

另一个我们可以看的度量标准是*closeness centrality*。首先，对于每个用户，我们计算她的*farness*，即她到每个其他用户的最短路径长度之和。由于我们已经计算了每对节点之间的最短路径，所以将它们的长度相加很容易。（如果有多条最短路径，它们的长度都相同，所以我们只需看第一条。）

```py
def farness(user_id: int) -> float:
    """the sum of the lengths of the shortest paths to each other user"""
    return sum(len(paths[0])
               for paths in shortest_paths[user_id].values())
```

之后计算接近中心度（[图22-3](#network_sized_by_closeness)）的工作量就很小：

```py
closeness_centrality = {user.id: 1 / farness(user.id) for user in users}
```

![根据接近中心度调整大小的DataSciencester网络。](assets/dsf2_2203.png)

###### 图22-3\. 根据接近中心度调整大小的DataSciencester网络

这里变化很少——即使非常中心的节点距离外围节点也相当远。

正如我们所见，计算最短路径有点麻烦。因此，介数和接近中心度在大型网络上并不经常使用。不太直观（但通常更容易计算）的*eigenvector centrality*更常用。

# 特征向量中心度

要谈论特征向量中心度，我们必须谈论特征向量，而要谈论特征向量，我们必须谈论矩阵乘法。

## 矩阵乘法

如果*A*是一个<math><mrow><mi>n</mi> <mo>×</mo> <mi>m</mi></mrow></math>矩阵，*B*是一个<math><mrow><mi>m</mi> <mo>×</mo> <mi>k</mi></mrow></math>矩阵（注意*A*的第二维度与*B*的第一维度相同），它们的乘积*AB*是一个<math><mrow><mi>n</mi> <mo>×</mo> <mi>k</mi></mrow></math>矩阵，其(*i*,*j*)项为：

<math alttext="upper A Subscript i Baseline 1 Baseline upper B Subscript 1 j plus upper A Subscript i Baseline 2 Baseline upper B Subscript 2 j plus ellipsis plus upper A Subscript i m Baseline upper B Subscript m j" display="block"><mrow><msub><mi>A</mi> <mrow><mi>i</mi><mn>1</mn></mrow></msub> <msub><mi>B</mi> <mrow><mn>1</mn><mi>j</mi></mrow></msub> <mo>+</mo> <msub><mi>A</mi> <mrow><mi>i</mi><mn>2</mn></mrow></msub> <msub><mi>B</mi> <mrow><mn>2</mn><mi>j</mi></mrow></msub> <mo>+</mo> <mo>⋯</mo> <mo>+</mo> <msub><mi>A</mi> <mrow><mi>i</mi><mi>m</mi></mrow></msub> <msub><mi>B</mi> <mrow><mi>m</mi><mi>j</mi></mrow></msub></mrow></math>

这只是*A*的第*i*行（看作向量）与*B*的第*j*列（也看作向量）的点积。

我们可以使用[第四章](ch04.html#linear_algebra)中的`make_matrix`函数来实现这一点：

```py
from scratch.linear_algebra import Matrix, make_matrix, shape

def matrix_times_matrix(m1: Matrix, m2: Matrix) -> Matrix:
    nr1, nc1 = shape(m1)
    nr2, nc2 = shape(m2)
    assert nc1 == nr2, "must have (# of columns in m1) == (# of rows in m2)"

    def entry_fn(i: int, j: int) -> float:
        """dot product of i-th row of m1 with j-th column of m2"""
        return sum(m1[i][k] * m2[k][j] for k in range(nc1))

    return make_matrix(nr1, nc2, entry_fn)
```

如果我们将一个*m*维向量视为`(m, 1)`矩阵，我们可以将其乘以一个`(n, m)`矩阵得到一个`(n, 1)`矩阵，然后我们可以将其视为一个*n*维向量。

这意味着另一种思考一个`(n, m)`矩阵的方式是将其视为将*m*维向量转换为*n*维向量的线性映射：

```py
from scratch.linear_algebra import Vector, dot

def matrix_times_vector(m: Matrix, v: Vector) -> Vector:
    nr, nc = shape(m)
    n = len(v)
    assert nc == n, "must have (# of cols in m) == (# of elements in v)"

    return [dot(row, v) for row in m]  # output has length nr
```

当*A*是一个*方阵*时，这个操作将*n*维向量映射到其他*n*维向量。对于某些矩阵*A*和向量*v*，当*A*作用于*v*时，可能得到*v*的一个标量倍数—也就是说，结果是一个指向*v*相同方向的向量。当这种情况发生时（并且此外*v*不是全零向量），我们称*v*是*A*的一个*特征向量*。我们称这个乘数为*特征值*。

找到*A*的一个特征向量的一个可能方法是选择一个起始向量*v*，应用`matrix_times_vector`，重新缩放结果使其大小为1，并重复，直到过程收敛：

```py
from typing import Tuple
import random
from scratch.linear_algebra import magnitude, distance

def find_eigenvector(m: Matrix,
                     tolerance: float = 0.00001) -> Tuple[Vector, float]:
    guess = [random.random() for _ in m]

    while True:
        result = matrix_times_vector(m, guess)    # transform guess
        norm = magnitude(result)                  # compute norm
        next_guess = [x / norm for x in result]   # rescale

        if distance(guess, next_guess) < tolerance:
            # convergence so return (eigenvector, eigenvalue)
            return next_guess, norm

        guess = next_guess
```

通过构造，返回的 `guess` 是一个向量，使得当你将 `matrix_times_vector` 应用于它并将其重新缩放为长度为 1 时，你会得到一个非常接近其自身的向量——这意味着它是一个特征向量。

并非所有的实数矩阵都有特征向量和特征值。例如，矩阵：

```py
rotate = [[ 0, 1],
          [-1, 0]]
```

将向量顺时针旋转 90 度，这意味着它唯一将其映射到自身的标量倍数的向量是一个零向量。如果你尝试 `find_eigenvector(rotate)`，它会无限运行。甚至具有特征向量的矩阵有时也会陷入循环中。考虑以下矩阵：

```py
flip = [[0, 1],
        [1, 0]]
```

此矩阵将任何向量 `[x, y]` 映射到 `[y, x]`。这意味着，例如，`[1, 1]` 是一个特征向量，其特征值为 1。然而，如果你从具有不同坐标的随机向量开始，`find_eigenvector` 将永远只是无限交换坐标。（像 NumPy 这样的非从头开始的库使用不同的方法，可以在这种情况下工作。）尽管如此，当 `find_eigenvector` 确实返回一个结果时，那个结果确实是一个特征向量。

## 中心性

这如何帮助我们理解 DataSciencester 网络？首先，我们需要将网络中的连接表示为一个 `adjacency_matrix`，其（*i*,*j*）th 元素为 1（如果用户 *i* 和用户 *j* 是朋友）或 0（如果不是）：

```py
def entry_fn(i: int, j: int):
    return 1 if (i, j) in friend_pairs or (j, i) in friend_pairs else 0

n = len(users)
adjacency_matrix = make_matrix(n, n, entry_fn)
```

然后，每个用户的特征向量中心性就是在 `find_eigenvector` 返回的特征向量中对应于该用户的条目 ([图 22-4](#network_sized_by_eigenvector))。

![DataSciencester 网络的特征向量中心性大小。](assets/dsf2_2204.png)

###### 图 22-4\. DataSciencester 网络的特征向量中心性大小

###### 注意

基于远远超出本书范围的技术原因，任何非零的邻接矩阵必然具有一个特征向量，其所有值都是非负的。幸运的是，对于这个 `adjacency_matrix`，我们的 `find_eigenvector` 函数找到了它。

```py
eigenvector_centralities, _ = find_eigenvector(adjacency_matrix)
```

具有高特征向量中心性的用户应该是那些拥有许多连接并且连接到自身中心性高的人的用户。

在这里，用户 1 和 2 是最中心的，因为他们都有三个连接到自身中心性高的人。随着我们远离他们，人们的中心性逐渐下降。

在这样一个小网络上，特征向量中心性表现得有些不稳定。如果你尝试添加或删除链接，你会发现网络中的微小变化可能会极大地改变中心性数值。在一个规模大得多的网络中，情况可能不会特别如此。

我们仍然没有解释为什么特征向量可能导致一个合理的中心性概念。成为特征向量意味着如果你计算：

```py
matrix_times_vector(adjacency_matrix, eigenvector_centralities)
```

结果是 `eigenvector_centralities` 的标量倍数。

如果你看矩阵乘法的工作方式，`matrix_times_vector` 生成一个向量，其 *i*th 元素为：

```py
dot(adjacency_matrix[i], eigenvector_centralities)
```

这正是连接到用户 *i* 的用户的特征向量中心性的总和。

换句话说，特征向量中心度是一个数字，每个用户一个，其值是他的邻居值的常数倍。在这种情况下，中心度意味着与自身中心度很高的人连接。你直接连接的中心度越高，你自己就越中心。这当然是一个循环定义——特征向量是打破循环性的方法。

另一种理解方式是通过思考`find_eigenvector`在这里的作用来理解这个问题。它首先为每个节点分配一个随机的中心度，然后重复以下两个步骤，直到过程收敛：

1.  给每个节点一个新的中心度分数，该分数等于其邻居（旧的）中心度分数的总和。

1.  重新调整中心度向量，使其大小为1。

尽管其背后的数学可能一开始看起来有些难懂，但计算本身相对简单（不像介数中心度那样），甚至可以在非常大的图上执行。 （至少，如果你使用真正的线性代数库，那么在大型图上执行起来是很容易的。如果你使用我们的矩阵作为列表的实现，你会有些困难。）

# 有向图和 PageRank

DataSciencester 并没有得到很多关注，因此收入副总裁考虑从友谊模式转向背书模式。结果表明，没有人特别在意哪些数据科学家*彼此是朋友*，但技术招聘人员非常在意其他数据科学家*受到*其他数据科学家的*尊重*。

在这个新模型中，我们将跟踪不再代表互惠关系的背书`(source, target)`，而是`source`背书`target`作为一个优秀的数据科学家（参见[图 22-5](#datasciencester_network_endorsements)）。

![数据科学家背书网络。](assets/dsf2_2205.png)

###### 图 22-5\. DataSciencester 的背书网络

我们需要考虑这种不对称性：

```py
endorsements = [(0, 1), (1, 0), (0, 2), (2, 0), (1, 2),
                (2, 1), (1, 3), (2, 3), (3, 4), (5, 4),
                (5, 6), (7, 5), (6, 8), (8, 7), (8, 9)]
```

之后，我们可以轻松地找到`most_endorsed`数据科学家，并将这些信息卖给招聘人员：

```py
from collections import Counter

endorsement_counts = Counter(target for source, target in endorsements)
```

然而，“背书数量”是一个容易操控的度量标准。你所需要做的就是创建假账户，并让它们为你背书。或者与你的朋友安排互相为对方背书。（就像用户0、1和2似乎已经做过的那样。）

更好的度量标准应考虑*谁*为你背书。来自背书颇多的人的背书应该比来自背书较少的人的背书更有价值。这就是 PageRank 算法的本质，它被谷歌用来根据其他网站链接到它们的网站来排名网站，以及那些链接到这些网站的网站，依此类推。

（如果这让你想起特征向量中心度的想法，那是正常的。）

一个简化版本看起来像这样：

1.  网络中总共有1.0（或100%）的 PageRank。

1.  最初，这个 PageRank 在节点之间均匀分布。

1.  在每一步中，每个节点的大部分 PageRank 均匀分布在其出链之间。

1.  在每个步骤中，每个节点的PageRank剩余部分均匀分布在所有节点之间。

```py
import tqdm

def page_rank(users: List[User],
              endorsements: List[Tuple[int, int]],
              damping: float = 0.85,
              num_iters: int = 100) -> Dict[int, float]:
    # Compute how many people each person endorses
    outgoing_counts = Counter(target for source, target in endorsements)

    # Initially distribute PageRank evenly
    num_users = len(users)
    pr = {user.id : 1 / num_users for user in users}

    # Small fraction of PageRank that each node gets each iteration
    base_pr = (1 - damping) / num_users

    for iter in tqdm.trange(num_iters):
        next_pr = {user.id : base_pr for user in users}  # start with base_pr

        for source, target in endorsements:
            # Add damped fraction of source pr to target
            next_pr[target] += damping * pr[source] / outgoing_counts[source]

        pr = next_pr

    return pr
```

如果我们计算页面排名：

```py
pr = page_rank(users, endorsements)

# Thor (user_id 4) has higher page rank than anyone else
assert pr[4] > max(page_rank
                   for user_id, page_rank in pr.items()
                   if user_id != 4)
```

PageRank（[图 22-6](#network_sized_by_pagerank)）将用户 4（Thor）标识为排名最高的数据科学家。

![按PageRank排序的数据科学家网络。](assets/dsf2_2206.png)

###### 图 22-6\. 数据科学家网络按PageRank排序的大小

尽管Thor获得的认可少于用户 0、1 和 2（各有两个），但他的认可带来的排名来自他们的认可。此外，他的两个认可者仅认可了他一个人，这意味着他不必与任何其他人分享他们的排名。

# 进一步探索

+   除了我们使用的这些之外，还有[许多其他的中心度概念](http://en.wikipedia.org/wiki/Centrality)（尽管我们使用的这些基本上是最受欢迎的）。

+   [NetworkX](http://networkx.github.io/) 是用于网络分析的 Python 库。它有计算中心度和可视化图形的函数。

+   [Gephi](https://gephi.org/) 是一款爱它或者恨它的基于GUI的网络可视化工具。
