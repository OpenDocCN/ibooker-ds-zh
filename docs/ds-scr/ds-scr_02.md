# 第1章 介绍

> “数据！数据！数据！”他不耐烦地喊道。“没有粘土我无法制造砖块。”
> 
> 亚瑟·柯南·道尔

# 数据的崛起

我们生活在一个数据泛滥的世界。网站跟踪每个用户的每一次点击。你的智能手机每天每秒都在记录你的位置和速度。“量化自我”的人们穿着像计步器一样的设备，始终记录着他们的心率、运动习惯、饮食和睡眠模式。智能汽车收集驾驶习惯，智能家居收集生活习惯，智能营销人员收集购买习惯。互联网本身就是一个包含巨大知识图谱的网络，其中包括（但不限于）一个庞大的交叉参考百科全书；关于电影、音乐、体育结果、弹珠机、网络文化和鸡尾酒的专业数据库；以及来自太多政府的太多（有些几乎是真实的！）统计数据，以至于你无法完全理解。

在这些数据中埋藏着无数问题的答案，这些问题甚至没有人曾想过去问。在本书中，我们将学习如何找到它们。

# 什么是数据科学？

有一个笑话说，数据科学家是那些比计算机科学家懂更多统计学、比统计学家懂更多计算机科学的人。（我并不是说这是个好笑话。）事实上，一些数据科学家在实际上更像是统计学家，而另一些则几乎无法与软件工程师区分开来。有些是机器学习专家，而另一些甚至无法从幼儿园的机器学习出来。有些是拥有令人印象深刻出版记录的博士，而另一些从未读过学术论文（虽然他们真是太可耻了）。简而言之，几乎无论你如何定义数据科学，你都会找到那些对于定义完全错误的从业者。

尽管如此，我们不会因此而放弃尝试。我们会说，数据科学家是那些从杂乱数据中提取见解的人。今天的世界充满了试图将数据转化为见解的人们。

例如，约会网站OkCupid要求其会员回答成千上万个问题，以便找到最合适的匹配对象。但它也分析这些结果，以找出你可以问某人的听起来无伤大雅的问题，来了解[在第一次约会时某人愿意与你发生关系的可能性有多大](https://theblog.okcupid.com/the-best-questions-for-a-first-date-dba6adaa9df2)。

Facebook要求你列出你的家乡和当前位置，表面上是为了让你的朋友更容易找到并联系你。但它也分析这些位置，以[识别全球迁移模式](https://www.facebook.com/notes/facebook-data-science/coordinated-migration/10151930946453859)和[不同足球队球迷居住地的分布](https://www.facebook.com/notes/facebook-data-science/nfl-fans-on-facebook/10151298370823859)。

作为一家大型零售商，Target跟踪你的在线和门店购买及互动行为。它使用[数据来预测模型](https://www.nytimes.com/2012/02/19/magazine/shopping-habits.html)，以更好地向客户市场化婴儿相关购买。

在2012年，奥巴马竞选团队雇用了数十名数据科学家，通过数据挖掘和实验，找到需要额外关注的选民，选择最佳的特定捐款呼吁和方案，并将选民动员工作集中在最有可能有用的地方。而在2016年，特朗普竞选团队[测试了多种在线广告](https://www.wired.com/2016/11/facebook-won-trump-election-not-just-fake-news/)，并分析数据找出有效和无效的广告。

现在，在你开始感到太厌倦之前：一些数据科学家偶尔也会运用他们的技能来做些善事——[使用数据使政府更有效](https://www.marketplace.org/2014/08/22/tech/beyond-ad-clicks-using-big-data-social-good)，[帮助无家可归者](https://dssg.uchicago.edu/2014/08/20/tracking-the-paths-of-homelessness/)，以及[改善公共健康](https://plus.google.com/communities/109572103057302114737)。但如果你喜欢研究如何最好地让人们点击广告，那对你的职业生涯肯定也是有好处的。

# 激励假设：DataSciencester

祝贺！你刚刚被聘为DataSciencester的数据科学主管，*数据科学家*的社交网络。

###### 注意

当我写这本书的第一版时，我认为“为数据科学家建立社交网络”是一个有趣、愚蠢的假设。自那时以来，人们实际上创建了为数据科学家建立的社交网络，并从风险投资家那里筹集到比我从我的书中赚到的钱多得多的资金。很可能这里有一个关于愚蠢的数据科学假设和/或图书出版的宝贵教训。

尽管*数据科学家*为核心，DataSciencester实际上从未投资于建立自己的数据科学实践。（公平地说，DataSciencester从未真正投资于建立自己的产品。）这将是你的工作！在本书中，我们将通过解决你在工作中遇到的问题来学习数据科学概念。有时我们会查看用户明确提供的数据，有时我们会查看通过他们与网站的互动生成的数据，有时甚至会查看我们设计的实验数据。

而且因为DataSciencester有着强烈的“非自主创新”精神，我们将从头开始建立自己的工具。最后，你将对数据科学的基础有相当扎实的理解。你将准备好在一个基础更稳固的公司应用你的技能，或者解决任何其他你感兴趣的问题。

欢迎加入，并祝你好运！（星期五可以穿牛仔裤，洗手间在右边的走廊尽头。）

## 寻找关键联络人

你在DataSciencester的第一天上班，网络副总裁对你的用户充满了疑问。直到现在，他没有人可以问，所以他对你的加入非常兴奋。

他特别想让你识别出数据科学家中的“关键连接者”。为此，他给了你整个DataSciencester网络的数据转储。（在现实生活中，人们通常不会把你需要的数据直接交给你。[第9章](ch09.html#getting_data)专门讨论获取数据的问题。）

这个数据转储看起来是什么样子？它包含了一个用户列表，每个用户由一个`dict`表示，其中包含该用户的`id`（一个数字）和`name`（在一个伟大的宇宙巧合中，与用户的`id`押韵）：

```py
users = [
    { "id": 0, "name": "Hero" },
    { "id": 1, "name": "Dunn" },
    { "id": 2, "name": "Sue" },
    { "id": 3, "name": "Chi" },
    { "id": 4, "name": "Thor" },
    { "id": 5, "name": "Clive" },
    { "id": 6, "name": "Hicks" },
    { "id": 7, "name": "Devin" },
    { "id": 8, "name": "Kate" },
    { "id": 9, "name": "Klein" }
]
```

他还给了你“友谊”数据，表示为一组ID对的列表：

```py
friendship_pairs = [(0, 1), (0, 2), (1, 2), (1, 3), (2, 3), (3, 4),
                    (4, 5), (5, 6), (5, 7), (6, 8), (7, 8), (8, 9)]
```

例如，元组`(0, 1)`表示`id`为0（Hero）的数据科学家和`id`为1（Dunn）的数据科学家是朋友。网络在[图1-1](#datasciencester_network_ch01)中有所展示。

![DataSciencester网络。](assets/dsf2_0101.png)

###### 图1-1\. 数据科学家网络

将友谊表示为一组对并不是最容易处理它们的方式。要找出用户1的所有友谊，你必须遍历每一对，查找包含1的对。如果有很多对，这将需要很长时间。

相反，让我们创建一个`dict`，其中键是用户的`id`，值是朋友的`id`列表。（在`dict`中查找东西非常快。）

###### 注意

现在先别太过于纠结代码的细节。在[第2章](ch02.html#python)，我将带你快速入门Python。现在只需试着把握我们正在做的大致意思。

我们仍然必须查看每一对来创建`dict`，但我们只需这样做一次，之后查找将会很快：

```py
# Initialize the dict with an empty list for each user id:
friendships = {user["id"]: [] for user in users}

# And loop over the friendship pairs to populate it:
for i, j in friendship_pairs:
    friendships[i].append(j)  # Add j as a friend of user i
    friendships[j].append(i)  # Add i as a friend of user j
```

现在我们已经将友谊关系存入`dict`中，我们可以轻松地询问我们的图形问题，比如“平均连接数是多少？”

首先，我们通过总结所有`friends`列表的长度来找到*总*连接数：

```py
def number_of_friends(user):
    """How many friends does _user_ have?"""
    user_id = user["id"]
    friend_ids = friendships[user_id]
    return len(friend_ids)

total_connections = sum(number_of_friends(user)
                        for user in users)        # 24
```

然后我们只需通过用户数量来除以：

```py
num_users = len(users)                            # length of the users list
avg_connections = total_connections / num_users   # 24 / 10 == 2.4
```

要找到最连接的人也很容易 —— 他们是朋友最多的人。

由于用户数量不多，我们可以简单地按“最多朋友”到“最少朋友”的顺序排序：

```py
# Create a list (user_id, number_of_friends).
num_friends_by_id = [(user["id"], number_of_friends(user))
                     for user in users]

num_friends_by_id.sort(                                # Sort the list
       key=lambda id_and_friends: id_and_friends[1],   # by num_friends
       reverse=True)                                   # largest to smallest

# Each pair is (user_id, num_friends):
# [(1, 3), (2, 3), (3, 3), (5, 3), (8, 3),
#  (0, 2), (4, 2), (6, 2), (7, 2), (9, 1)]
```

另一种理解我们所做的是作为识别网络中某些关键人物的一种方式。事实上，我们刚刚计算的是网络度量指标*度中心性*（[图1-2](#network_sized_by_degree)）。

![DataSciencester网络按度数排列大小。](assets/dsf2_0102.png)

###### 图1-2\. 数据科学家网络按度数排列大小

这个方法非常容易计算，但不总是给出你希望或预期的结果。例如，在 DataSciencester 网络中，Thor（`id` 4）只有两个连接，而 Dunn（`id` 1）有三个。然而，当我们查看网络时，直觉上 Thor 应该更为核心。在[第22章](ch22.html#network_analysis)中，我们将更详细地研究网络，并且会探讨可能与我们直觉更符合的更复杂的中心性概念。

## 可能认识的数据科学家

当你还在填写新员工文件时，友谊副总裁来到你的办公桌前。她希望在你的成员之间建立更多连接，她要求你设计一个“数据科学家你可能认识”的建议者。

你的第一反应是建议用户可能认识他们朋友的朋友。因此，你编写了一些代码来迭代他们的朋友并收集朋友的朋友：

```py
def foaf_ids_bad(user):
    """foaf is short for "friend of a friend" """
    return [foaf_id
            for friend_id in friendships[user["id"]]
            for foaf_id in friendships[friend_id]]
```

当我们在`users[0]`（Hero）上调用它时，它产生：

```py
[0, 2, 3, 0, 1, 3]
```

它包括用户0两次，因为Hero确实与他的两个朋友都是朋友。它包括用户1和2，尽管他们已经是Hero的朋友。它包括用户3两次，因为通过两个不同的朋友可以到达Chi：

```py
print(friendships[0])  # [1, 2]
print(friendships[1])  # [0, 2, 3]
print(friendships[2])  # [0, 1, 3]
```

知道人们通过多种方式是朋友的信息似乎是有趣的信息，因此也许我们应该产生共同朋友的计数。而且我们可能应该排除用户已知的人：

```py
from collections import Counter                   # not loaded by default

def friends_of_friends(user):
    user_id = user["id"]
    return Counter(
        foaf_id
        for friend_id in friendships[user_id]     # For each of my friends,
        for foaf_id in friendships[friend_id]     # find their friends
        if foaf_id != user_id                     # who aren't me
        and foaf_id not in friendships[user_id]   # and aren't my friends.
    )

print(friends_of_friends(users[3]))               # Counter({0: 2, 5: 1})
```

这正确地告诉了 Chi（`id` 3），她与 Hero（`id` 0）有两个共同的朋友，但只有一个与 Clive（`id` 5）有共同朋友。

作为一名数据科学家，您知道您也可能喜欢与兴趣相似的用户会面。（这是数据科学“实质性专业知识”方面的一个很好的例子。）在询问之后，您成功获取了这些数据，作为一对（用户ID，兴趣）的列表：

```py
interests = [
    (0, "Hadoop"), (0, "Big Data"), (0, "HBase"), (0, "Java"),
    (0, "Spark"), (0, "Storm"), (0, "Cassandra"),
    (1, "NoSQL"), (1, "MongoDB"), (1, "Cassandra"), (1, "HBase"),
    (1, "Postgres"), (2, "Python"), (2, "scikit-learn"), (2, "scipy"),
    (2, "numpy"), (2, "statsmodels"), (2, "pandas"), (3, "R"), (3, "Python"),
    (3, "statistics"), (3, "regression"), (3, "probability"),
    (4, "machine learning"), (4, "regression"), (4, "decision trees"),
    (4, "libsvm"), (5, "Python"), (5, "R"), (5, "Java"), (5, "C++"),
    (5, "Haskell"), (5, "programming languages"), (6, "statistics"),
    (6, "probability"), (6, "mathematics"), (6, "theory"),
    (7, "machine learning"), (7, "scikit-learn"), (7, "Mahout"),
    (7, "neural networks"), (8, "neural networks"), (8, "deep learning"),
    (8, "Big Data"), (8, "artificial intelligence"), (9, "Hadoop"),
    (9, "Java"), (9, "MapReduce"), (9, "Big Data")
]
```

例如，Hero（`id` 0）与 Klein（`id` 9）没有共同朋友，但他们分享 Java 和大数据的兴趣。

建立一个找出具有特定兴趣的用户的函数很容易：

```py
def data_scientists_who_like(target_interest):
    """Find the ids of all users who like the target interest."""
    return [user_id
            for user_id, user_interest in interests
            if user_interest == target_interest]
```

这样做虽然有效，但每次搜索都需要检查整个兴趣列表。如果我们有很多用户和兴趣（或者我们想进行大量搜索），我们可能最好建立一个从兴趣到用户的索引：

```py
from collections import defaultdict

# Keys are interests, values are lists of user_ids with that interest
user_ids_by_interest = defaultdict(list)

for user_id, interest in interests:
    user_ids_by_interest[interest].append(user_id)
```

另一个从用户到兴趣的：

```py
# Keys are user_ids, values are lists of interests for that user_id.
interests_by_user_id = defaultdict(list)

for user_id, interest in interests:
    interests_by_user_id[user_id].append(interest)
```

现在很容易找出与给定用户共同兴趣最多的人：

+   迭代用户的兴趣。

+   对于每个兴趣，迭代具有该兴趣的其他用户。

+   记录我们看到每个其他用户的次数。

在代码中：

```py
def most_common_interests_with(user):
    return Counter(
        interested_user_id
        for interest in interests_by_user_id[user["id"]]
        for interested_user_id in user_ids_by_interest[interest]
        if interested_user_id != user["id"]
    )
```

我们可以利用这一点来构建一个更丰富的“数据科学家你可能认识”的功能，基于共同朋友和共同兴趣的组合。我们将在[第23章](ch23.html#recommender_systems)中探讨这些应用的类型。

## 薪水和经验

正在准备去午餐时，公共关系副总裁问您是否可以提供一些关于数据科学家赚多少钱的有趣事实。薪资数据当然是敏感的，但他设法为您提供了一个匿名数据集，其中包含每个用户的`薪资`（以美元计）和作为数据科学家的`任期`（以年计）：

```py
salaries_and_tenures = [(83000, 8.7), (88000, 8.1),
                        (48000, 0.7), (76000, 6),
                        (69000, 6.5), (76000, 7.5),
                        (60000, 2.5), (83000, 10),
                        (48000, 1.9), (63000, 4.2)]
```

自然的第一步是绘制数据（我们将在[第3章](ch03.html#visualizing_data)中看到如何做到这一点）。您可以在[图1-3](#salaries)中看到结果。

![按年经验计算的工资。](assets/dsf2_0103.png)

###### 图1-3\. 按经验年限计算的工资

看起来明显，有更多经验的人往往赚更多。您如何将其转化为有趣的事实？您的第一个想法是查看每个任期的平均工资：

```py
# Keys are years, values are lists of the salaries for each tenure.
salary_by_tenure = defaultdict(list)

for salary, tenure in salaries_and_tenures:
    salary_by_tenure[tenure].append(salary)

# Keys are years, each value is average salary for that tenure.
average_salary_by_tenure = {
    tenure: sum(salaries) / len(salaries)
    for tenure, salaries in salary_by_tenure.items()
}
```

结果证明这并不特别有用，因为没有一个用户拥有相同的任期，这意味着我们只是报告个别用户的薪水：

```py
{0.7: 48000.0,
 1.9: 48000.0,
 2.5: 60000.0,
 4.2: 63000.0,
 6: 76000.0,
 6.5: 69000.0,
 7.5: 76000.0,
 8.1: 88000.0,
 8.7: 83000.0,
 10: 83000.0}
```

对职位进行分桶可能更有帮助：

```py
def tenure_bucket(tenure):
    if tenure < 2:
        return "less than two"
    elif tenure < 5:
        return "between two and five"
    else:
        return "more than five"
```

然后我们可以将对应于每个桶的工资分组在一起：

```py
# Keys are tenure buckets, values are lists of salaries for that bucket.
salary_by_tenure_bucket = defaultdict(list)

for salary, tenure in salaries_and_tenures:
    bucket = tenure_bucket(tenure)
    salary_by_tenure_bucket[bucket].append(salary)
```

最后为每个组计算平均工资：

```py
# Keys are tenure buckets, values are average salary for that bucket.
average_salary_by_bucket = {
  tenure_bucket: sum(salaries) / len(salaries)
  for tenure_bucket, salaries in salary_by_tenure_bucket.items()
}
```

哪个更有趣：

```py
{'between two and five': 61500.0,
 'less than two': 48000.0,
 'more than five': 79166.66666666667}
```

而你的声音片段是：“有五年以上经验的数据科学家比没有经验或经验较少的数据科学家赚65%的工资！”

但我们选择桶的方式相当随意。我们真正想要的是对增加一年经验的平均薪水效应做一些说明。除了制作更生动的趣味事实外，这还允许我们对我们不知道的工资做出*预测*。我们将在[第14章](ch14.html#simple_linear_regression)中探讨这个想法。

## 付费帐户

当您回到桌前时，收入副总裁正在等待您。她想更好地了解哪些用户为帐户付费，哪些不付费。（她知道他们的名字，但那不是特别可行的信息。）

您注意到经验年限与付费帐户之间似乎存在对应关系：

```py
0.7  paid
1.9  unpaid
2.5  paid
4.2  unpaid
6.0  unpaid
6.5  unpaid
7.5  unpaid
8.1  unpaid
8.7  paid
10.0 paid
```

经验非常少或非常多的用户往往会支付；经验平均的用户则不会。因此，如果你想创建一个模型——尽管这绝对不是足够的数据来建立模型——你可以尝试预测经验非常少或非常多的用户的“有偿”情况，以及经验适中的用户的“无偿”情况：

```py
def predict_paid_or_unpaid(years_experience):
  if years_experience < 3.0:
    return "paid"
  elif years_experience < 8.5:
    return "unpaid"
  else:
    return "paid"
```

当然，我们完全是靠眼睛估计的分界线。

有了更多数据（和更多数学），我们可以构建一个模型，根据用户的经验年限来预测他是否会付费。我们将在[第16章](ch16.html#logistic_regression)中研究这类问题。

## 兴趣主题

当您完成第一天工作时，内容战略副总裁要求您提供关于用户最感兴趣的主题的数据，以便她能够相应地规划博客日历。你已经有了来自朋友推荐项目的原始数据：

```py
interests = [
    (0, "Hadoop"), (0, "Big Data"), (0, "HBase"), (0, "Java"),
    (0, "Spark"), (0, "Storm"), (0, "Cassandra"),
    (1, "NoSQL"), (1, "MongoDB"), (1, "Cassandra"), (1, "HBase"),
    (1, "Postgres"), (2, "Python"), (2, "scikit-learn"), (2, "scipy"),
    (2, "numpy"), (2, "statsmodels"), (2, "pandas"), (3, "R"), (3, "Python"),
    (3, "statistics"), (3, "regression"), (3, "probability"),
    (4, "machine learning"), (4, "regression"), (4, "decision trees"),
    (4, "libsvm"), (5, "Python"), (5, "R"), (5, "Java"), (5, "C++"),
    (5, "Haskell"), (5, "programming languages"), (6, "statistics"),
    (6, "probability"), (6, "mathematics"), (6, "theory"),
    (7, "machine learning"), (7, "scikit-learn"), (7, "Mahout"),
    (7, "neural networks"), (8, "neural networks"), (8, "deep learning"),
    (8, "Big Data"), (8, "artificial intelligence"), (9, "Hadoop"),
    (9, "Java"), (9, "MapReduce"), (9, "Big Data")
]
```

找到最受欢迎的兴趣的一种简单（如果不是特别令人兴奋）的方法是计算单词数：

1.  将每个兴趣都转换为小写（因为不同用户可能会或不会将其兴趣大写）。

1.  将其分割成单词。

1.  计算结果。

在代码中：

```py
words_and_counts = Counter(word
                           for user, interest in interests
                           for word in interest.lower().split())
```

这使得可以轻松列出出现超过一次的单词：

```py
for word, count in words_and_counts.most_common():
    if count > 1:
        print(word, count)
```

这会得到您预期的结果（除非您期望“scikit-learn”被分成两个单词，在这种情况下它不会得到您期望的结果）：

```py
learning 3
java 3
python 3
big 3
data 3
hbase 2
regression 2
cassandra 2
statistics 2
probability 2
hadoop 2
networks 2
machine 2
neural 2
scikit-learn 2
r 2
```

我们将在[第21章](ch21.html#natural_language_processing)中探讨从数据中提取主题的更复杂方法。

## 继续

这是一个成功的第一天！精疲力尽地，你溜出大楼，趁没人找你要事情之前。好好休息，因为明天是新员工入职培训。（是的，你在新员工入职培训*之前*已经度过了一整天的工作。这个问题可以找HR解决。）
