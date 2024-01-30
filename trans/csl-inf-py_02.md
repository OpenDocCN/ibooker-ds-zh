# 1）国际象棋选手比其他人更理性吗？

> 原文：[`causal-methods.github.io/Book/1%29_Are_Chess_Players_More_Rational_than_the_Rest_of_the_Population.html`](https://causal-methods.github.io/Book/1%29_Are_Chess_Players_More_Rational_than_the_Rest_of_the_Population.html)

[Vitor Kamada](https://www.linkedin.com/in/vitor-kamada-1b73a078)

电子邮件：econometrics.methods@gmail.com

最后更新日期：2020 年 10 月 28 日

看一下下面来自 Palacios-Huerta & Volij（2009）的蜈蚣游戏图。每个玩家只有两种策略：“停止”或“继续”。如果玩家 1 在第一轮停止，他会得到 4 美元，玩家 2 会得到 1 美元。如果玩家 1 继续，玩家 2 可以选择停止或继续。如果玩家 2 停止，他将获得 8 美元，玩家 1 将获得 2 美元。如果玩家 2 继续，玩家 1 再次可以决定“停止”或“继续”。从社会角度来看，最好是两名玩家在六轮中都选择“继续”，然后玩家 1 可以获得 256 美元，玩家 2 可以获得 64 美元。然而，如果玩家 2 是理性的，他永远不会在第 6 轮继续玩，“因为如果他停止，他可以获得 128 美元。知道通过反向归纳，玩家 1 在第 1 轮继续是不理性的。

![alt text](img/019e7e72f34636900ac8a048d456e3ae.png)

**来源**：Palacios-Huerta & Volij（2009）

几项实验研究表明，几乎没有人在第一次机会停下来。根据 Palacios-Huerta & Volij（2009）的下表，只有 7.5%的学生在第一轮停下来。大多数学生（35%）在第 3 轮停下来。在 McKelvey & Palfrey（1992）中可以找到大样本量的类似结果。

![alt text](img/739d9d4869c7f8ee2c4949f2eaad01aa.png)

**来源**：Palacios-Huerta & Volij（2009）

让我们打开 Palacios-Huerta & Volij（2009）的数据集，其中包含有关国际象棋选手如何玩蜈蚣游戏的信息。

```py
import pandas as pd
path = "https://github.com/causal-methods/Data/raw/master/" 
sheet_name = "Data Chess Tournaments"
chess = pd.read_excel(path + "Chess.xls", sheet_name)
chess 
```

|  | 比赛 | 标题 1 | ELORating1 | 标题 2 | ELORating2 | 终节点 |
| --- | --- | --- | --- | --- | --- | --- |
| 0 | 贝纳斯克 | GM | 2634 | 0 | 2079 | 1 |
| 1 | 贝纳斯克 | GM | 2621 | 0 | 2169 | 1 |
| 2 | 贝纳斯克 | GM | 2562 | FM | 2307 | 1 |
| 3 | 贝纳斯克 | GM | 2558 | 0 | 2029 | 1 |
| 4 | 贝纳斯克 | GM | 2521 | 0 | 2090 | 1 |
| ... | ... | ... | ... | ... | ... | ... |
| 206 | 塞斯塔奥 | GM | 2501 | 0 | 2019 | 1 |
| 207 | 塞斯塔奥 | 0 | 2050 | 0 | 2240 | 2 |
| 208 | 塞斯塔奥 | 0 | 2020 | 0 | 2004 | 1 |
| 209 | 塞斯塔奥 | 0 | 2133 | GM | 2596 | 1 |
| 210 | 塞斯塔奥 | 0 | 2070 | FM | 2175 | 1 |

211 行×6 列

让我们统计每个类别中的国际象棋选手人数。在样本中，我们可以看到 26 名国际大师（GM），29 名国际大师（IM）和 15 名联邦大师（FM）。

```py
chess['Title1'].value_counts() 
```

```py
0      130
IM      29
GM      26
FM      15
WGM      6
WIM      3
WFM      1
CM       1
Name: Title1, dtype: int64 
```

让我们将所有其他国际象棋选手标记为“其他”。

```py
dictionary = {    'Title1':
               {    0: "Other",
                'WGM': "Other", 
                'WIM': "Other", 
                'WFM': "Other",
                 'CM': "Other" }}

chess.replace(dictionary, inplace=True) 
```

我们在“其他”类别中有 141 名国际象棋选手。

```py
chess['Title1'].value_counts() 
```

```py
Other    141
IM        29
GM        26
FM        15
Name: Title1, dtype: int64 
```

让我们关注标有 1 的国际象棋选手的头衔和评分。让我们忽略他们对手的头衔和评分，也就是标有 2 的国际象棋选手。

```py
title_rating1 = chess.filter(["Title1", "ELORating1"])
title_rating1 
```

|  | 标题 1 | ELORating1 |
| --- | --- | --- |
| 0 | GM | 2634 |
| 1 | GM | 2621 |
| 2 | GM | 2562 |
| 3 | GM | 2558 |
| 4 | GM | 2521 |
| ... | ... | ... |
| 206 | GM | 2501 |
| 207 | 其他 | 2050 |
| 208 | 其他 | 2020 |
| 209 | 其他 | 2133 |
| 210 | 其他 | 2070 |

211 行×2 列

我们可以看到国际大师的平均评分为 2513，而国际大师的平均评分为 2411。

请注意，评分是预测谁将赢得国际象棋比赛的很好的指标。

极不可能在“其他”类别中的国际象棋选手击败国际大师。

评分可以被理解为理性和反向归纳的代理。

```py
# Round 2 decimals
pd.set_option('precision', 2)

import numpy as np
title_rating1.groupby('Title1').agg(np.mean) 
```

|  | ELORating1 |
| --- | --- |
| 标题 1 |  |
| --- | --- |
| FM | 2324.13 |
| GM | 2512.96 |
| IM | 2411.69 |
| 其他 | 2144.60 |

让我们将分析限制在只有国际大师。

```py
grandmasters = chess[chess['Title1'] == "GM"]
grandmasters 
```

|  | 比赛 | 标题 1 | ELORating1 | 标题 2 | ELORating2 | 终节点 |
| --- | --- | --- | --- | --- | --- | --- |
| 0 | 贝纳斯克 | GM | 2634 | 0 | 2079 | 1 |
| 1 | 贝纳斯克 | GM | 2621 | 0 | 2169 | 1 |
| 2 | Benasque | GM | 2562 | FM | 2307 | 1 |
| 3 | Benasque | GM | 2558 | 0 | 2029 | 1 |
| 4 | Benasque | GM | 2521 | 0 | 2090 | 1 |
| 5 | Benasque | GM | 2521 | FM | 2342 | 1 |
| 7 | Benasque | GM | 2510 | 0 | 2018 | 1 |
| 8 | Benasque | GM | 2507 | 0 | 2043 | 1 |
| 9 | Benasque | GM | 2500 | 0 | 2097 | 1 |
| 10 | Benasque | GM | 2495 | 0 | 2043 | 1 |
| 12 | Benasque | GM | 2488 | 0 | 2045 | 1 |
| 13 | Benasque | GM | 2488 | GM | 2514 | 1 |
| 15 | Benasque | GM | 2485 | 0 | 2092 | 1 |
| 16 | Benasque | GM | 2482 | 0 | 2040 | 1 |
| 18 | Benasque | GM | 2475 | 0 | 2229 | 1 |
| 19 | Benasque | GM | 2473 | 0 | 2045 | 1 |
| 35 | Benasque | GM | 2378 | 0 | 2248 | 1 |
| 156 | Leon | GM | 2527 | 0 | 2200 | 1 |
| 168 | Sestao | GM | 2671 | 0 | 2086 | 1 |
| 170 | Sestao | GM | 2495 | 0 | 2094 | 1 |
| 172 | Sestao | GM | 2532 | 0 | 2075 | 1 |
| 174 | Sestao | GM | 2501 | 0 | 2005 | 1 |
| 176 | Sestao | GM | 2516 | GM | 2501 | 1 |
| 177 | Sestao | GM | 2444 | 0 | 2136 | 1 |
| 193 | Sestao | GM | 2452 | 0 | 2027 | 1 |
| 206 | Sestao | GM | 2501 | 0 | 2019 | 1 |

所有 26 位特级大师都在第一轮停下了蜈蚣游戏。他们就是标准经济学教科书中描述的理性经济人。

不幸的是，在 78 亿人口的人口中，根据国际象棋联合会(FIDE)的说法，我们只有 1721 位特级大师。

[2020 年 7 月 13 日访问的特级大师名单](https://ratings.fide.com/advaction.phtml?idcode=&name=&title=g&other_title=&country=&sex=&srating=0&erating=3000&birthday=&radio=rating&ex_rated=&line=desc&inactiv=&offset=1700)

```py
grandmasters.groupby('EndNode').size() 
```

```py
EndNode
1    26
dtype: int64 
```

让我们来检查一下国际大师(IM)。

```py
international_master = chess[chess['Title1'] == "IM"]

# Return only 4 observations
international_master.head(4) 
```

|  | Tournament | Title1 | ELORating1 | Title2 | ELORating2 | EndNode |
| --- | --- | --- | --- | --- | --- | --- |
| 6 | Benasque | IM | 2521 | 0 | 2179 | 1 |
| 11 | Benasque | IM | 2492 | 0 | 2093 | 1 |
| 14 | Benasque | IM | 2487 | IM | 2474 | 2 |
| 17 | Benasque | IM | 2479 | 0 | 2085 | 1 |

并非所有国际大师都在第一轮停下。

5 位国际大师在第二轮停下，还有 2 位在第三轮停下。

```py
node_IM = international_master.groupby('EndNode').size()
node_IM 
```

```py
EndNode
1    22
2     5
3     2
dtype: int64 
```

这 5 位在第二轮停下的国际大师代表了总国际大师人数的 17%。

只有 76%的国际大师按照新古典经济理论的预测行事。

```py
length_IM = len(international_master)
prop_IM = node_IM / length_IM
prop_IM 
```

```py
EndNode
1    0.76
2    0.17
3    0.07
dtype: float64 
```

让我们对联邦大师应用相同的程序。

在每个节点停下的联邦大师的比例与国际大师相似。

```py
federation_master = chess[chess['Title1'] == "FM"]

node_FM = federation_master.groupby('EndNode').size()
length_FM = len(federation_master)

prop_FM = node_FM / length_FM
prop_FM 
```

```py
EndNode
1    0.73
2    0.20
3    0.07
dtype: float64 
```

让我们将先前的描述性统计数据放在一个条形图中。

更容易地可视化每个结束蜈蚣游戏的节点上国际象棋选手的比例。

条形图表明，国际大师和联邦大师在玩蜈蚣游戏时的方式不同于国际大师和联邦大师。然而，国际大师和联邦大师在玩蜈蚣游戏时的方式看起来是相似的。

```py
import plotly.graph_objects as go
node = ['Node 1', 'Node 2', 'Node 3']

fig = go.Figure(data=[
    go.Bar(name='Grandmasters', x=node, y=[1,0,0]),
    go.Bar(name='International Masters', x=node, y=prop_IM),
    go.Bar(name='Federation Masters', x=node, y=prop_FM) ])

fig.update_layout(barmode='group',
  title_text = 'Share of Chess Players that Ended Centipede Game at Each Node',
  font=dict(size=21) )

fig.show() 
```

让我们正式地测试一下特级大师(\(p_{g}\))的比例是否等于国际大师(\(p_{i}\))的比例。零假设(\(H_0\))是：

\[H_0: p_{g} = p_{i}\]

z 统计量为：

\[z=\frac{\hat{p}_{g}-\hat{p}_{i}}{se(\hat{p}_{g}-\hat{p}_{i})}\]

其中\(se(\hat{p}_{g}-\hat{p}_{i})\)是样本比例之间的标准误差：

\[se(\hat{p}_{g}-\hat{p}_{i})=\sqrt{\frac{\hat{p}_{g}(1-\hat{p}_{g})}{n_g}+\frac{\hat{p}_{i}(1-\hat{p}_{i})}{n_i}}\]

其中\(n_g\)是特级大师的样本量，\(n_i\)是国际大师的样本量。

对于节点 1，我们知道：

\[\hat{p}_{g}=\frac{26}{26}=1\]\[\hat{p}_{i}=\frac{22}{29}=0.73\]

然后：

\[z=2.68\]

z 统计量的 p 值为 0.007。因此，在显著性水平\(\alpha=1\%\)上拒绝零假设(\(H_0\))。

```py
from statsmodels.stats.proportion import proportions_ztest

#  I inserted manually the data from Grandmasters to 
# ilustrate the input format
count = np.array([ 26, node_IM[1] ]) # number of stops
nobs = np.array([ 26, length_IM ])   # sample size

proportions_ztest(count, nobs) 
```

```py
C:\Anaconda\envs\textbook\lib\site-packages\statsmodels\tools\_testing.py:19: FutureWarning:

pandas.util.testing is deprecated. Use the functions in the public API at pandas.testing instead. 
```

```py
(2.68162114289528, 0.00732663816543149) 
```

让我们还在节点 1 测试一下，国际大师(\(p_{i}\))的比例是否等于联邦大师(\(p_{f}\))的比例。零假设(\(H_0\))是：

\[H_0: p_{i} = p_{f}\]

z 统计量为 0.18，相应的 p 值为 0.85。因此，我们无法拒绝国际大师比例等于联邦大师比例的零假设。

```py
count = np.array([ node_IM[1], node_FM[1] ])
nobs = np.array([ length_IM, length_FM ])

proportions_ztest(count, nobs) 
```

```py
(0.1836204648065827, 0.8543112075010346) 
```

## 练习

1）使用 Palacios-Huerta & Volij（2009）的电子表格“Data Chess Tournaments”来计算停留在节点 1、节点 2、节点 3、节点 4 和节点 5 的其他国际象棋选手的比例。比例将总和为 100%。其他国际象棋选手是一个排除所有国际象棋选手（包括特级大师、国际大师和联邦大师）的类别。

2）这个问题涉及 Palacios-Huerta & Volij（2009）的电子表格“Data UPV Students-One shot”。

a) 在 Google Colab 中打开电子表格“Data UPV Students-One shot”。

b) 巴斯克大学（UPV）的学生玩了多少对蜈蚣游戏？

c) 计算停留在节点 1、节点 2、节点 3、节点 4、节点 5 和节点 6 的学生比例。比例将总和为 100%。

3）比较练习 1（国际象棋选手）和练习 2（c）（学生）的结果。为什么这两个子群体玩蜈蚣游戏的方式不同？推测并证明你的推理。

4）使用 Palacios-Huerta & Volij（2009）的电子表格“Data Chess Tournaments”来测试在蜈蚣游戏的节点 3 停止的国际大师比例是否等于在节点 3 停止的其他国际象棋选手的比例。

5）对于这个问题，使用 Palacios-Huerta & Volij（2009）的电子表格“Data Chess Tournaments”。创建一个条形图，比较国际大师和其他国际象棋选手在每个节点停止蜈蚣游戏的比例。

6）假设你是一位新古典经济学家。你如何证明在更强的理性和自利的假设下建立的标准经济理论与 Palacios-Huerta & Volij（2009）的论文中呈现的实证证据相矛盾？详细说明你的理由。

## 参考文献

Fey, Mark, Richard D. McKelvey, and Thomas R. Palfrey. (1996). An Experimental Study of Constant-Sum Centipede Games. International Journal of Game Theory, 25(3): 269–87.

Palacios-Huerta, Ignacio, and Oscar Volij. (2009). [Field Centipedes](https://github.com/causal-methods/Papers/raw/master/Centipedes/Field%20Centipedes.pdf). American Economic Review, 99 (4): 1619-35.
