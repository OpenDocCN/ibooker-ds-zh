# 第二十五章：Chapter 23\. 推荐系统

> O nature, nature, why art thou so dishonest, as ever to send men with these false recommendations into the world!
> 
> Henry Fielding

Another common data problem is producing *recommendations* of some sort. Netflix recommends movies you might want to watch. Amazon recommends products you might want to buy. Twitter recommends users you might want to follow. In this chapter, we’ll look at several ways to use data to make recommendations.

In particular, we’ll look at the dataset of `users_interests` that we’ve used before:

```py
users_interests = [
    ["Hadoop", "Big Data", "HBase", "Java", "Spark", "Storm", "Cassandra"],
    ["NoSQL", "MongoDB", "Cassandra", "HBase", "Postgres"],
    ["Python", "scikit-learn", "scipy", "numpy", "statsmodels", "pandas"],
    ["R", "Python", "statistics", "regression", "probability"],
    ["machine learning", "regression", "decision trees", "libsvm"],
    ["Python", "R", "Java", "C++", "Haskell", "programming languages"],
    ["statistics", "probability", "mathematics", "theory"],
    ["machine learning", "scikit-learn", "Mahout", "neural networks"],
    ["neural networks", "deep learning", "Big Data", "artificial intelligence"],
    ["Hadoop", "Java", "MapReduce", "Big Data"],
    ["statistics", "R", "statsmodels"],
    ["C++", "deep learning", "artificial intelligence", "probability"],
    ["pandas", "R", "Python"],
    ["databases", "HBase", "Postgres", "MySQL", "MongoDB"],
    ["libsvm", "regression", "support vector machines"]
]
```

And we’ll think about the problem of recommending new interests to a user based on her currently specified interests.

# Manual Curation

Before the internet, when you needed book recommendations you would go to the library, where a librarian was available to suggest books that were relevant to your interests or similar to books you liked.

Given DataSciencester’s limited number of users and interests, it would be easy for you to spend an afternoon manually recommending interests for each user. But this method doesn’t scale particularly well, and it’s limited by your personal knowledge and imagination. (Not that I’m suggesting that your personal knowledge and imagination are limited.) So let’s think about what we can do with *data*.

# Recommending What’s Popular

One easy approach is to simply recommend what’s popular:

```py
from collections import Counter

popular_interests = Counter(interest
                            for user_interests in users_interests
                            for interest in user_interests)
```

which looks like:

```py
[('Python', 4),
 ('R', 4),
 ('Java', 3),
 ('regression', 3),
 ('statistics', 3),
 ('probability', 3),
 # ...
]
```

Having computed this, we can just suggest to a user the most popular interests that he’s not already interested in:

```py
from typing import List, Tuple

def most_popular_new_interests(
        user_interests: List[str],
        max_results: int = 5) -> List[Tuple[str, int]]:
    suggestions = [(interest, frequency)
                   for interest, frequency in popular_interests.most_common()
                   if interest not in user_interests]
    return suggestions[:max_results]
```

So, if you are user 1, with interests:

```py
["NoSQL", "MongoDB", "Cassandra", "HBase", "Postgres"]
```

then we’d recommend you:

```py
[('Python', 4), ('R', 4), ('Java', 3), ('regression', 3), ('statistics', 3)]
```

If you are user 3, who’s already interested in many of those things, you’d instead get:

```py
[('Java', 3), ('HBase', 3), ('Big Data', 3),
 ('neural networks', 2), ('Hadoop', 2)]
```

Of course, “lots of people are interested in Python, so maybe you should be too” is not the most compelling sales pitch. If someone is brand new to our site and we don’t know anything about them, that’s possibly the best we can do. Let’s see how we can do better by basing each user’s recommendations on her existing interests.

# 基于用户的协同过滤

One way of taking a user’s interests into account is to look for users who are somehow *similar* to her, and then suggest the things that those users are interested in.

In order to do that, we’ll need a way to measure how similar two users are. Here we’ll use cosine similarity, which we used in 第二十一章 to measure how similar two word vectors were.

We’ll apply this to vectors of 0s and 1s, each vector `v` representing one user’s interests. `v[i]` will be 1 if the user specified the *i*th interest, and 0 otherwise. Accordingly, “similar users” will mean “users whose interest vectors most nearly point in the same direction.” Users with identical interests will have similarity 1\. Users with no identical interests will have similarity 0\. Otherwise, the similarity will fall in between, with numbers closer to 1 indicating “very similar” and numbers closer to 0 indicating “not very similar.”

一个很好的开始是收集已知的兴趣，并（隐式地）为它们分配索引。我们可以通过使用集合推导来找到唯一的兴趣，并将它们排序成一个列表。结果列表中的第一个兴趣将是兴趣 0，依此类推：

```py
unique_interests = sorted({interest
                           for user_interests in users_interests
                           for interest in user_interests})
```

这给我们一个以这样开始的列表：

```py
assert unique_interests[:6] == [
    'Big Data',
    'C++',
    'Cassandra',
    'HBase',
    'Hadoop',
    'Haskell',
    # ...
]
```

接下来，我们想为每个用户生成一个“兴趣”向量，其中包含 0 和 1。我们只需遍历`unique_interests`列表，如果用户具有每个兴趣，则替换为 1，否则为 0：

```py
def make_user_interest_vector(user_interests: List[str]) -> List[int]:
    """
 Given a list of interests, produce a vector whose ith element is 1
 if unique_interests[i] is in the list, 0 otherwise
 """
    return [1 if interest in user_interests else 0
            for interest in unique_interests]
```

现在我们可以制作一个用户兴趣向量的列表：

```py
user_interest_vectors = [make_user_interest_vector(user_interests)
                         for user_interests in users_interests]
```

现在，如果用户`i`指定了兴趣`j`，那么`user_interest_vectors[i][j]`等于 1，否则为 0。

因为我们有一个小数据集，计算所有用户之间的成对相似性是没有问题的：

```py
from scratch.nlp import cosine_similarity

user_similarities = [[cosine_similarity(interest_vector_i, interest_vector_j)
                      for interest_vector_j in user_interest_vectors]
                     for interest_vector_i in user_interest_vectors]
```

之后，`user_similarities[i][j]`给出了用户`i`和`j`之间的相似性：

```py
# Users 0 and 9 share interests in Hadoop, Java, and Big Data
assert 0.56 < user_similarities[0][9] < 0.58, "several shared interests"

# Users 0 and 8 share only one interest: Big Data
assert 0.18 < user_similarities[0][8] < 0.20, "only one shared interest"
```

特别地，`user_similarities[i]`是用户`i`与每个其他用户的相似性向量。我们可以使用这个来编写一个函数，找出与给定用户最相似的用户。我们会确保不包括用户本身，也不包括任何相似性为零的用户。并且我们会按照相似性从高到低对结果进行排序：

```py
def most_similar_users_to(user_id: int) -> List[Tuple[int, float]]:
    pairs = [(other_user_id, similarity)                      # Find other
             for other_user_id, similarity in                 # users with
                enumerate(user_similarities[user_id])         # nonzero
             if user_id != other_user_id and similarity > 0]  # similarity.

    return sorted(pairs,                                      # Sort them
                  key=lambda pair: pair[-1],                  # most similar
                  reverse=True)                               # first.
```

例如，如果我们调用`most_similar_users_to(0)`，我们会得到：

```py
[(9, 0.5669467095138409),
 (1, 0.3380617018914066),
 (8, 0.1889822365046136),
 (13, 0.1690308509457033),
 (5, 0.1543033499620919)]
```

我们如何利用这个来向用户建议新的兴趣？对于每个兴趣，我们可以简单地加上对它感兴趣的其他用户的用户相似性：

```py
from collections import defaultdict

def user_based_suggestions(user_id: int,
                           include_current_interests: bool = False):
    # Sum up the similarities
    suggestions: Dict[str, float] = defaultdict(float)
    for other_user_id, similarity in most_similar_users_to(user_id):
        for interest in users_interests[other_user_id]:
            suggestions[interest] += similarity

    # Convert them to a sorted list
    suggestions = sorted(suggestions.items(),
                         key=lambda pair: pair[-1],  # weight
                         reverse=True)

    # And (maybe) exclude already interests
    if include_current_interests:
        return suggestions
    else:
        return [(suggestion, weight)
                for suggestion, weight in suggestions
                if suggestion not in users_interests[user_id]]
```

如果我们调用`user_based_suggestions(0)`，那么前几个建议的兴趣是：

```py
[('MapReduce', 0.5669467095138409),
 ('MongoDB', 0.50709255283711),
 ('Postgres', 0.50709255283711),
 ('NoSQL', 0.3380617018914066),
 ('neural networks', 0.1889822365046136),
 ('deep learning', 0.1889822365046136),
 ('artificial intelligence', 0.1889822365046136),
 #...
]
```

对于那些声称兴趣是“大数据”和数据库相关的人来说，这些看起来是相当不错的建议。（权重本质上没有意义；我们只是用它们来排序。）

当项目数量变得非常大时，这种方法效果不佳。回想一下第十二章中的维度诅咒 —— 在高维向量空间中，大多数向量相距甚远（并且指向非常不同的方向）。也就是说，当兴趣的数量很多时，对于给定用户，“最相似的用户”可能完全不相似。

想象一个像 Amazon.com 这样的网站，我在过去几十年里购买了成千上万件物品。你可以基于购买模式尝试识别与我类似的用户，但在全世界范围内，几乎没有人的购买历史看起来像我的。无论我的“最相似”的购物者是谁，他可能与我完全不相似，他的购买几乎肯定不会提供好的推荐。

# 基于物品的协同过滤

另一种方法是直接计算兴趣之间的相似性。然后我们可以通过聚合与她当前兴趣相似的兴趣来为每个用户生成建议。

要开始，我们将希望*转置*我们的用户-兴趣矩阵，以便行对应于兴趣，列对应于用户：

```py
interest_user_matrix = [[user_interest_vector[j]
                         for user_interest_vector in user_interest_vectors]
                        for j, _ in enumerate(unique_interests)]
```

这是什么样子？`interest_user_matrix` 的第 `j` 行就是 `user_interest_matrix` 的第 `j` 列。也就是说，对于每个具有该兴趣的用户，它的值为 1，对于每个没有该兴趣的用户，它的值为 0。

例如，`unique_interests[0]` 是大数据，所以 `interest_user_matrix[0]` 是：

```py
[1, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0]
```

因为用户 0、8 和 9 表示对大数据感兴趣。

现在我们可以再次使用余弦相似度。如果完全相同的用户对两个主题感兴趣，它们的相似度将为 1。如果没有两个用户对两个主题感兴趣，它们的相似度将为 0：

```py
interest_similarities = [[cosine_similarity(user_vector_i, user_vector_j)
                          for user_vector_j in interest_user_matrix]
                         for user_vector_i in interest_user_matrix]
```

例如，我们可以使用以下方法找到与大数据（兴趣 0）最相似的兴趣：

```py
def most_similar_interests_to(interest_id: int):
    similarities = interest_similarities[interest_id]
    pairs = [(unique_interests[other_interest_id], similarity)
             for other_interest_id, similarity in enumerate(similarities)
             if interest_id != other_interest_id and similarity > 0]
    return sorted(pairs,
                  key=lambda pair: pair[-1],
                  reverse=True)
```

这表明以下类似的兴趣：

```py
[('Hadoop', 0.8164965809277261),
 ('Java', 0.6666666666666666),
 ('MapReduce', 0.5773502691896258),
 ('Spark', 0.5773502691896258),
 ('Storm', 0.5773502691896258),
 ('Cassandra', 0.4082482904638631),
 ('artificial intelligence', 0.4082482904638631),
 ('deep learning', 0.4082482904638631),
 ('neural networks', 0.4082482904638631),
 ('HBase', 0.3333333333333333)]
```

现在我们可以通过累加与其相似的兴趣的相似性来为用户创建推荐：

```py
def item_based_suggestions(user_id: int,
                           include_current_interests: bool = False):
    # Add up the similar interests
    suggestions = defaultdict(float)
    user_interest_vector = user_interest_vectors[user_id]
    for interest_id, is_interested in enumerate(user_interest_vector):
        if is_interested == 1:
            similar_interests = most_similar_interests_to(interest_id)
            for interest, similarity in similar_interests:
                suggestions[interest] += similarity

    # Sort them by weight
    suggestions = sorted(suggestions.items(),
                         key=lambda pair: pair[-1],
                         reverse=True)

    if include_current_interests:
        return suggestions
    else:
        return [(suggestion, weight)
                for suggestion, weight in suggestions
                if suggestion not in users_interests[user_id]]
```

对于用户 0，这将生成以下（看起来合理的）推荐：

```py
[('MapReduce', 1.861807319565799),
 ('Postgres', 1.3164965809277263),
 ('MongoDB', 1.3164965809277263),
 ('NoSQL', 1.2844570503761732),
 ('programming languages', 0.5773502691896258),
 ('MySQL', 0.5773502691896258),
 ('Haskell', 0.5773502691896258),
 ('databases', 0.5773502691896258),
 ('neural networks', 0.4082482904638631),
 ('deep learning', 0.4082482904638631),
 ('C++', 0.4082482904638631),
 ('artificial intelligence', 0.4082482904638631),
 ('Python', 0.2886751345948129),
 ('R', 0.2886751345948129)]
```

# 矩阵分解

正如我们所看到的，我们可以将用户的偏好表示为一个 `[num_users, num_items]` 的矩阵，其中 1 表示喜欢的项目，0 表示不喜欢的项目。

有时您实际上可能有数值型的 *评分*；例如，当您写亚马逊评论时，您为物品分配了从 1 到 5 星的评分。您仍然可以通过数字在一个 `[num_users, num_items]` 的矩阵中表示这些（暂时忽略未评分项目的问题）。

在本节中，我们假设已经有了这样的评分数据，并尝试学习一个能够预测给定用户和项目评分的模型。

解决这个问题的一种方法是假设每个用户都有一些潜在的“类型”，可以表示为一组数字向量，而每个项目同样也有一些潜在的“类型”。

如果将用户类型表示为 `[num_users, dim]` 矩阵，将项目类型的转置表示为 `[dim, num_items]` 矩阵，则它们的乘积是一个 `[num_users, num_items]` 矩阵。因此，构建这样一个模型的一种方式是将偏好矩阵“因子化”为用户矩阵和项目矩阵的乘积。

（也许这种潜在类型的想法会让你想起我们在 第二十一章 中开发的词嵌入。记住这个想法。）

而不是使用我们虚构的 10 用户数据集，我们将使用 MovieLens 100k 数据集，其中包含许多用户对许多电影的评分，评分从 0 到 5 不等。每个用户只对少数电影进行了评分。我们将尝试构建一个系统，可以预测任意给定的（用户，电影）对的评分。我们将训练它以在每个用户评分的电影上表现良好；希望它能推广到用户未评分的电影。

首先，让我们获取数据集。您可以从 [*http://files.grouplens.org/datasets/movielens/ml-100k.zip*](http://files.grouplens.org/datasets/movielens/ml-100k.zip) 下载它。

解压并提取文件；我们仅使用其中两个：

```py
# This points to the current directory, modify if your files are elsewhere.
MOVIES = "u.item"   # pipe-delimited: movie_id|title|...
RATINGS = "u.data"  # tab-delimited: user_id, movie_id, rating, timestamp
```

常见情况下，我们会引入 `NamedTuple` 来使工作更加简便：

```py
from typing import NamedTuple

class Rating(NamedTuple):
    user_id: str
    movie_id: str
    rating: float
```

###### 注意

电影 ID 和用户 ID 实际上是整数，但它们不是连续的，这意味着如果我们将它们作为整数处理，将会有很多浪费的维度（除非我们重新编号所有内容）。因此，为了简化起见，我们将它们视为字符串处理。

现在让我们读取数据并探索它。电影文件是管道分隔的，并且有许多列。我们只关心前两列，即 ID 和标题：

```py
import csv
# We specify this encoding to avoid a UnicodeDecodeError.
# See: https://stackoverflow.com/a/53136168/1076346.
with open(MOVIES, encoding="iso-8859-1") as f:
    reader = csv.reader(f, delimiter="|")
    movies = {movie_id: title for movie_id, title, *_ in reader}
```

评分文件是制表符分隔的，包含四列：`user_id`、`movie_id`、评分（1 到 5），以及`timestamp`。我们将忽略时间戳，因为我们不需要它：

```py
# Create a list of [Rating]
with open(RATINGS, encoding="iso-8859-1") as f:
    reader = csv.reader(f, delimiter="\t")
    ratings = [Rating(user_id, movie_id, float(rating))
               for user_id, movie_id, rating, _ in reader]

# 1682 movies rated by 943 users
assert len(movies) == 1682
assert len(list({rating.user_id for rating in ratings})) == 943
```

有很多有趣的探索性分析可以在这些数据上进行；例如，您可能对*星球大战*电影的平均评分感兴趣（该数据集来自 1998 年，比*星球大战：幽灵的威胁*晚一年）：

```py
import re

# Data structure for accumulating ratings by movie_id
star_wars_ratings = {movie_id: []
                     for movie_id, title in movies.items()
                     if re.search("Star Wars|Empire Strikes|Jedi", title)}

# Iterate over ratings, accumulating the Star Wars ones
for rating in ratings:
    if rating.movie_id in star_wars_ratings:
        star_wars_ratings[rating.movie_id].append(rating.rating)

# Compute the average rating for each movie
avg_ratings = [(sum(title_ratings) / len(title_ratings), movie_id)
               for movie_id, title_ratings in star_wars_ratings.items()]

# And then print them in order
for avg_rating, movie_id in sorted(avg_ratings, reverse=True):
    print(f"{avg_rating:.2f} {movies[movie_id]}")
```

它们都评分很高：

```py
4.36 Star Wars (1977)
4.20 Empire Strikes Back, The (1980)
4.01 Return of the Jedi (1983)
```

所以让我们尝试设计一个模型来预测这些评分。作为第一步，让我们将评分数据分成训练集、验证集和测试集：

```py
import random
random.seed(0)
random.shuffle(ratings)

split1 = int(len(ratings) * 0.7)
split2 = int(len(ratings) * 0.85)

train = ratings[:split1]              # 70% of the data
validation = ratings[split1:split2]   # 15% of the data
test = ratings[split2:]               # 15% of the data
```

拥有一个简单的基线模型总是好的，并确保我们的模型比它表现更好。这里一个简单的基线模型可能是“预测平均评分”。我们将使用均方误差作为我们的指标，所以让我们看看基线在我们的测试集上表现如何：

```py
avg_rating = sum(rating.rating for rating in train) / len(train)
baseline_error = sum((rating.rating - avg_rating) ** 2
                     for rating in test) / len(test)

# This is what we hope to do better than
assert 1.26 < baseline_error < 1.27
```

给定我们的嵌入，预测的评分由用户嵌入和电影嵌入的矩阵乘积给出。对于给定的用户和电影，该值只是对应嵌入的点积。

所以让我们从创建嵌入开始。我们将它们表示为`dict`，其中键是 ID，值是向量，这样可以轻松地检索给定 ID 的嵌入：

```py
from scratch.deep_learning import random_tensor

EMBEDDING_DIM = 2

# Find unique ids
user_ids = {rating.user_id for rating in ratings}
movie_ids = {rating.movie_id for rating in ratings}

# Then create a random vector per id
user_vectors = {user_id: random_tensor(EMBEDDING_DIM)
                for user_id in user_ids}
movie_vectors = {movie_id: random_tensor(EMBEDDING_DIM)
                 for movie_id in movie_ids}
```

到目前为止，我们应该相当擅长编写训练循环：

```py
from typing import List
import tqdm
from scratch.linear_algebra import dot

def loop(dataset: List[Rating],
         learning_rate: float = None) -> None:
    with tqdm.tqdm(dataset) as t:
        loss = 0.0
        for i, rating in enumerate(t):
            movie_vector = movie_vectors[rating.movie_id]
            user_vector = user_vectors[rating.user_id]
            predicted = dot(user_vector, movie_vector)
            error = predicted - rating.rating
            loss += error ** 2

            if learning_rate is not None:
                #     predicted = m_0 * u_0 + ... + m_k * u_k
                # So each u_j enters output with coefficent m_j
                # and each m_j enters output with coefficient u_j
                user_gradient = [error * m_j for m_j in movie_vector]
                movie_gradient = [error * u_j for u_j in user_vector]

                # Take gradient steps
                for j in range(EMBEDDING_DIM):
                    user_vector[j] -= learning_rate * user_gradient[j]
                    movie_vector[j] -= learning_rate * movie_gradient[j]

            t.set_description(f"avg loss: {loss / (i + 1)}")
```

现在我们可以训练我们的模型（即找到最优的嵌入）。对我来说，如果我每个时期都稍微降低学习率，效果最好：

```py
learning_rate = 0.05
for epoch in range(20):
    learning_rate *= 0.9
    print(epoch, learning_rate)
    loop(train, learning_rate=learning_rate)
    loop(validation)
loop(test)
```

这个模型很容易过拟合训练集。我在测试集上的平均损失是大约 0.89，这时`EMBEDDING_DIM=2`的情况下取得最佳结果。

###### 注意

如果您想要更高维度的嵌入，您可以尝试像我们在“正则化”中使用的正则化。特别是，在每次梯度更新时，您可以将权重收缩至 0 附近。但我没能通过这种方式获得更好的结果。

现在，检查学习到的向量。没有理由期望这两个组件特别有意义，因此我们将使用主成分分析：

```py
from scratch.working_with_data import pca, transform

original_vectors = [vector for vector in movie_vectors.values()]
components = pca(original_vectors, 2)
```

让我们将我们的向量转换为表示主成分，并加入电影 ID 和平均评分：

```py
ratings_by_movie = defaultdict(list)
for rating in ratings:
    ratings_by_movie[rating.movie_id].append(rating.rating)

vectors = [
    (movie_id,
     sum(ratings_by_movie[movie_id]) / len(ratings_by_movie[movie_id]),
     movies[movie_id],
     vector)
    for movie_id, vector in zip(movie_vectors.keys(),
                                transform(original_vectors, components))
]

# Print top 25 and bottom 25 by first principal component
print(sorted(vectors, key=lambda v: v[-1][0])[:25])
print(sorted(vectors, key=lambda v: v[-1][0])[-25:])
```

前 25 个电影评分都很高，而后 25 个大部分是低评分的（或在训练数据中未评级），这表明第一个主成分主要捕捉了“这部电影有多好？”

对于我来说，很难理解第二个组件的意义；而且二维嵌入的表现只比一维嵌入略好，这表明第二个组件捕捉到的可能是非常微妙的内容。（可以推测，在较大的 MovieLens 数据集中可能有更有趣的事情发生。）

# 进一步探索

+   [惊喜](http://surpriselib.com/)是一个用于“构建和分析推荐系统”的 Python 库，似乎相当受欢迎且更新及时。

+   [Netflix Prize](http://www.netflixprize.com) 是一个相当有名的比赛，旨在构建更好的系统，向 Netflix 用户推荐电影。
