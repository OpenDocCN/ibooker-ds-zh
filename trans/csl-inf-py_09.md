# 八、Airbnb 的主人是否歧视黑人客人？

> 原文：[`causal-methods.github.io/Book/8%29_Do_Hosts_Discriminate_against_Black_Guests_in_Airbnb.html`](https://causal-methods.github.io/Book/8%29_Do_Hosts_Discriminate_against_Black_Guests_in_Airbnb.html)

[Vitor Kamada](https://www.linkedin.com/in/vitor-kamada-1b73a078)

电子邮件：econometrics.methods@gmail.com

最后更新时间：11-1-2020

Edelman et al.（2017）发现，黑人名字听起来比白人名字听起来更不可能被 Airbnb 接受为客人，减少了 16%。这个结果不仅仅是相关性。种族变量是随机的。黑人和白人之间唯一的区别是名字。除此之外，黑人和白人客人是一样的。

让我们打开 Edelman 等人（2017）的数据集。每一行是 2015 年 7 月 Airbnb 的一处物业。样本由巴尔的摩、达拉斯、洛杉矶、圣路易斯和华盛顿特区的所有物业组成。

```py
import numpy as np
import pandas as pd
pd.set_option('precision', 3)

# Data from Edelman et al. (2017)
path = "https://github.com/causal-methods/Data/raw/master/" 
df = pd.read_csv(path + "Airbnb.csv")
df.head(5) 
```

|  | 主人回应 | 回应日期 | 消息数量 | 自动编码 | 纬度 | 经度 | 床类型 | 物业类型 | 取消政策 | 客人数量 | ... | 洛杉矶 | 圣路易斯 | 华盛顿特区 | 总客人 | 原始黑人 | 物业黑人 | 任何黑人 | 过去客人合并 | 九月填充 | pr 填充 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 是 | 2015-07-19 08:26:17 | 2.0 | 1.0 | 34.081 | -118.270 | 真正的床 | 房子 | 灵活 | 3.0 | ... | 1 | 0 | 0 | 11.0 | 0.0 | 0.0 | 0.0 | 匹配（3） | 1 | 0.412 |
| 1 | 否或不可用 | 2015-07-14 14:13:39 | NaN | 1.0 | 38.911 | -77.020 | NaN | 房子 | 中等 | 2.0 | ... | 0 | 0 | 1 | 167.0 | 0.0 | 0.0 | 0.0 | 匹配（3） | 1 | 0.686 |
| 2 | 请求更多信息（你能验证吗？有多少... | 2015-07-20 16:24:08 | 2.0 | 0.0 | 34.005 | -118.481 | 拉出沙发 | 公寓 | 严格 | 1.0 | ... | 1 | 0 | 0 | 19.0 | 0.0 | 0.0 | 0.0 | 匹配（3） | 0 | 0.331 |
| 3 | 我会回复你 | 2015-07-20 06:47:38 | NaN | 0.0 | 34.092 | -118.282 | NaN | 房子 | 严格 | 8.0 | ... | 1 | 0 | 0 | 41.0 | 0.0 | 0.0 | 0.0 | 匹配（3） | 0 | 0.536 |
| 4 | 未发送消息 | . | NaN | 1.0 | 38.830 | -76.897 | 真正的床 | 房子 | 严格 | 2.0 | ... | 0 | 0 | 1 | 28.0 | 0.0 | 0.0 | 0.0 | 匹配（3） | 1 | 0.555 |

5 行×104 列

下面的图表显示，黑人客人收到的“是”的回应比白人客人少。有人可能会争辩说 Edelman 等人（2017）的结果是由主人回应的差异驱动的，比如有条件的或非回应。例如，你可以争辩说黑人更有可能有被归类为垃圾邮件的假账户。然而，请注意，歧视结果是由“是”和“否”驱动的，而不是由中间回应驱动的。

```py
# Data for bar chart
count = pd.crosstab(df["graph_bins"], df["guest_black"])

import plotly.graph_objects as go

node = ['Conditional No', 'Conditional Yes', 'No',
        'No Response', 'Yes']
fig = go.Figure(data=[
    go.Bar(name='Guest is white', x=node, y=count[0]),
    go.Bar(name='Guest is African American', x=node, y=count[1]) ])

fig.update_layout(barmode='group',
  title_text = 'Host Responses by Race',
  font=dict(size=18) )

fig.show() 
```

让我们复制 Edelman 等人（2017）的主要结果。

```py
import statsmodels.api as sm

df['const'] = 1 

# Column 1
#  The default missing ='drop' of statsmodels doesn't apply
# to the cluster variable. Therefore, it is necessary to drop
# the missing values like below to get the clustered standard 
# errors.
df1 = df.dropna(subset=['yes', 'guest_black', 'name_by_city'])
reg1 = sm.OLS(df1['yes'], df1[['const', 'guest_black']])
res1 = reg1.fit(cov_type='cluster',
                cov_kwds={'groups': df1['name_by_city']})

# Column 2
vars2 = ['yes', 'guest_black', 'name_by_city', 
        'host_race_black', 'host_gender_M']
df2 = df.dropna(subset = vars2)
reg2 = sm.OLS(df2['yes'], df2[['const', 'guest_black',
                    'host_race_black', 'host_gender_M']])
res2 = reg2.fit(cov_type='cluster',
                cov_kwds={'groups': df2['name_by_city']})

# Column 3
vars3 = ['yes', 'guest_black', 'name_by_city', 
         'host_race_black', 'host_gender_M',
         'multiple_listings', 'shared_property',
         'ten_reviews', 'log_price']
df3 = df.dropna(subset = vars3)
reg3 = sm.OLS(df3['yes'], df3[['const', 'guest_black',
                    'host_race_black', 'host_gender_M',
                    'multiple_listings', 'shared_property',
                    'ten_reviews', 'log_price']])
res3 = reg3.fit(cov_type='cluster',
                cov_kwds={'groups': df3['name_by_city']})

columns =[res1, res2, res3] 
```

```py
C:\Anaconda\envs\textbook\lib\site-packages\statsmodels\tools\_testing.py:19: FutureWarning:

pandas.util.testing is deprecated. Use the functions in the public API at pandas.testing instead. 
```

```py
#  Library to print professional publication
# tables in Latex, HTML, etc.
!pip install stargazer 
```

```py
Requirement already satisfied: stargazer in c:\anaconda\envs\textbook\lib\site-packages (0.0.5) 
```

```py
WARNING: Error parsing requirements for numpy: [Errno 2] No such file or directory: 'c:\\anaconda\\envs\\textbook\\lib\\site-packages\\numpy-1.19.2.dist-info\\METADATA' 
```

在第一列中，听起来像白人的名字被接受的概率为 49%；而听起来像黑人的名字被接受的概率大约为 41%。因此，黑人名字带来了 8%的惩罚。这个结果在第 2 列和第 3 列的一组控制变量中非常稳健。

```py
# Settings for a nice table
from stargazer.stargazer import Stargazer
stargazer = Stargazer(columns)
stargazer.title('The Impact of Race on Likelihood of Acceptance')
stargazer 
```

种族对接受可能性的影响

|  |
| --- |
|  | *因变量：是* |
|  |
|  | (1) | (2) | (3) |
|  |
| 常数 | 0.488^(***) | 0.497^(***) | 0.755^(***) |
|  | (0.012) | (0.013) | (0.067) |
| 黑人客人 | -0.080^(***) | -0.080^(***) | -0.087^(***) |
|  | (0.017) | (0.017) | (0.017) |
| 主人性别 M |  | -0.050^(***) | -0.048^(***) |
|  |  | (0.014) | (0.014) |
| 主人种族黑人 |  | 0.069^(***) | 0.093^(***) |
|  |  | (0.023) | (0.023) |
| 对数价格 |  |  | -0.062^(***) |
|  |  |  | (0.013) |
| 多个列表 |  |  | 0.062^(***) |
|  |  | (0.015) |
| 共享物业 |  |  | -0.068^(***) |
|  |  |  | (0.017) |
| 十次评论 |  |  | 0.120^(***) |
|  |  |  | (0.013) |
|  |
| 观察 | 6,235 | 6,235 | 6,168 |
| R² | 0.006 | 0.010 | 0.040 |
| 调整后的 R² | 0.006 | 0.009 | 0.039 |
| 残差标准误差 | 0.496（df=6233） | 0.495（df=6231） | 0.488（df=6160） |
| F 统计量 | 21.879^(***) (df=1; 6233) | 15.899^(***) (df=3; 6231) | 35.523^(***) (df=7; 6160) |
|  |
| 注： | ^*p<0.1; ^(**)p<0.05; ^(***)p<0.01 |

下表显示了有关主人和房产的摘要统计信息。在实验中，控制变量的平均值与分组实验和对照组的平均值相同。

```py
control = ['host_race_white', 'host_race_black', 'host_gender_F', 
	'host_gender_M', 'price', 'bedrooms', 'bathrooms', 'number_of_reviews', 
	'multiple_listings', 'any_black', 'tract_listings', 'black_proportion']

df.describe()[control].T 
```

|  | 计数 | 平均值 | 标准差 | 最小值 | 25% | 50% | 75% | 最大值 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| host_race_white | 6392.0 | 0.634 | 0.482 | 0.0 | 0.00 | 1.00 | 1.000 | 1.000 |
| host_race_black | 6392.0 | 0.078 | 0.269 | 0.0 | 0.00 | 0.00 | 0.000 | 1.000 |
| host_gender_F | 6392.0 | 0.376 | 0.485 | 0.0 | 0.00 | 0.00 | 1.000 | 1.000 |
| host_gender_M | 6392.0 | 0.298 | 0.457 | 0.0 | 0.00 | 0.00 | 1.000 | 1.000 |
| 价格 | 6302.0 | 181.108 | 1280.228 | 10.0 | 75.00 | 109.00 | 175.000 | 100000.000 |
| 卧室 | 6242.0 | 3.177 | 2.265 | 1.0 | 2.00 | 2.00 | 4.000 | 16.000 |
| 浴室 | 6285.0 | 3.169 | 2.264 | 1.0 | 2.00 | 2.00 | 4.000 | 16.000 |
| 评论数量 | 6390.0 | 30.869 | 72.505 | 0.0 | 2.00 | 9.00 | 29.000 | 1208.000 |
| 多个列表 | 6392.0 | 0.326 | 0.469 | 0.0 | 0.00 | 0.00 | 1.000 | 1.000 |
| 任何黑人 | 6390.0 | 0.282 | 0.450 | 0.0 | 0.00 | 0.00 | 1.000 | 1.000 |
| 地域列表 | 6392.0 | 9.514 | 9.277 | 1.0 | 2.00 | 6.00 | 14.000 | 53.000 |
| 黑人比例 | 6378.0 | 0.140 | 0.203 | 0.0 | 0.03 | 0.05 | 0.142 | 0.984 |

平衡处理测试(t 检验)显示黑人和白人客人是相同的。

```py
result = []

for var in control:
    # Do the T-test and save the p-value
    pvalue = sm.OLS(df[var], df[['const', 'guest_black']],
               missing = 'drop').fit().pvalues[1]
    result.append(pvalue) 
```

```py
ttest = df.groupby('guest_black').agg([np.mean])[control].T
ttest['p-value'] = result
ttest 
```

|  | 黑人客人 | 0.0 | 1.0 | p 值 |
| --- | --- | --- | --- | --- |
| host_race_white | 平均值 | 0.643 | 0.626 | 0.154 |
| host_race_black | 平均值 | 0.078 | 0.078 | 0.972 |
| host_gender_F | 平均值 | 0.381 | 0.372 | 0.439 |
| host_gender_M | 平均值 | 0.298 | 0.299 | 0.896 |
| 价格 | 平均值 | 166.429 | 195.815 | 0.362 |
| 卧室 | 平均值 | 3.178 | 3.176 | 0.962 |
| 浴室 | 平均值 | 3.172 | 3.167 | 0.927 |
| 评论数量 | 平均值 | 30.709 | 31.030 | 0.860 |
| 多个列表 | 平均值 | 0.321 | 0.330 | 0.451 |
| 任何黑人 | 平均值 | 0.287 | 0.277 | 0.382 |
| 地域列表 | 平均值 | 9.494 | 9.538 | 0.848 |
| 黑人比例 | 平均值 | 0.141 | 0.140 | 0.919 |

## 练习

1）据我所知，关于种族歧视的文献中最重要的三篇实证论文是 Bertrand & Mullainathan (2004), Oreopoulos (2011), 和 Edelman et al. (2017)。这三篇论文都使用了实地实验来捕捉因果关系并排除混杂因素。在互联网上搜索并返回一份关于种族歧视的实验论文的参考列表。

2）告诉我一个你热衷的话题。返回一个关于你的话题的实验论文的参考列表。

3）有人认为特定的名字驱动了 Edelman 等人(2017)的结果。在下面的表格中，你可以看到代表黑人和白人的名字并不多。如何反驳这个批评？你可以做什么来证明结果不是由特定的名字驱动的？

```py
female = df['guest_gender']=='female'
df[female].groupby(['guest_race', 'guest_first_name'])['yes'].mean() 
```

```py
guest_race  guest_first_name
black       Lakisha             0.433
            Latonya             0.370
            Latoya              0.442
            Tamika              0.482
            Tanisha             0.413
white       Allison             0.500
            Anne                0.567
            Kristen             0.486
            Laurie              0.508
            Meredith            0.498
Name: yes, dtype: float64 
```

```py
male = df['guest_gender']=='male'
df[male].groupby(['guest_race', 'guest_first_name'])['yes'].mean() 
```

```py
guest_race  guest_first_name
black       Darnell             0.412
            Jamal               0.354
            Jermaine            0.379
            Kareem              0.436
            Leroy               0.371
            Rasheed             0.409
            Tyrone              0.377
white       Brad                0.419
            Brent               0.494
            Brett               0.466
            Greg                0.467
            Jay                 0.581
            Todd                0.448
Name: yes, dtype: float64 
```

4）根据下表，是否有任何潜在的研究问题可以探讨？请证明。

```py
pd.crosstab(index= [df['host_gender_F'], df['host_race']],
            columns=[df['guest_gender'], df['guest_race']], 
            values=df['yes'], aggfunc='mean') 
```

|  | 客人性别 | 女性 | 男性 |
| --- | --- | --- | --- |
|  | 客人种族 | 黑人 | 白人 | 黑人 | 白人 |
| --- | --- | --- | --- | --- | --- |
| host_gender_F | host_race |  |  |  |  |
| --- | --- | --- | --- | --- | --- |
| 0 | UU | 0.400 | 0.542 | 0.158 | 0.381 |
| 亚裔 | 0.319 | 0.378 | 0.474 | 0.511 |
| 黑人 | 0.444 | 0.643 | 0.419 | 0.569 |
| 西班牙裔 | 0.464 | 0.571 | 0.375 | 0.478 |
| 多 | 0.568 | 0.727 | 0.408 | 0.357 |
| 不明确 | 0.444 | 0.500 | 0.444 | 0.333 |
| 不明确的三票 | 0.476 | 0.392 | 0.368 | 0.367 |
| 白人 | 0.383 | 0.514 | 0.386 | 0.449 |
| 1 | UU | 0.444 | 0.250 | 0.333 | 0.750 |
| 亚裔 | 0.429 | 0.607 | 0.436 | 0.460 |
| 黑人 | 0.603 | 0.537 | 0.397 | 0.446 |
| 西班牙裔 | 0.391 | 0.667 | 0.292 | 0.389 |
| 不明确 | 0.600 | 0.556 | 0.125 | 0.400 |
| 不明确的三票 | 0.387 | 0.583 | 0.312 | 0.657 |
| 白人 | 0.450 | 0.494 | 0.370 | 0.476 |

5）在 Edelman 等人（2017 年）中，变量“name_by_city”被用来对标准误差进行聚类。变量“name_by_city”是如何基于其他变量创建的？展示代码。

6）使用 Edelman 等人（2017 年）的数据来测试同族偏好假设，即主人可能更喜欢相同种族的客人。使用 Stargazer 库生成一个漂亮的表格。解释结果。

7）总的来说，人们知道社会经济地位与种族有关。Fryer＆Levitt（2004 年）表明，独特的非洲裔美国人名字与较低的社会经济地位相关。Edelman 等人（2017 年：17）明确表示：“我们的发现无法确定歧视是基于种族，社会经济地位，还是这两者的结合。”提出一个实验设计来分离种族和社会经济地位的影响。解释您的假设并详细描述程序。

## 参考

Bertrand，Marianne 和 Sendhil Mullainathan。 （2004）。[艾米丽和格雷格比拉基莎和贾迈尔更受雇用吗？劳动市场歧视的实地实验](https://github.com/causal-methods/Papers/raw/master/Are%20Emily%20and%20Greg%20More%20Employable%20than%20Lakisha%20and%20Jamal.pdf)。《美国经济评论》，94（4）：991-1013。

Edelman，Benjamin，Michael Luca 和 Dan Svirsky。 （2017）。[共享经济中的种族歧视：来自实地实验的证据](https://github.com/causal-methods/Papers/raw/master/Racial%20Discrimination%20in%20the%20Sharing%20Economy.pdf)。《美国经济学杂志：应用经济学》，9（2）：1-22。

Fryer，Roland G. Jr.和 Steven D. Levitt。 （2004）。Distinctively Black Names 的原因和后果。《经济学季刊》，119（3）：767–805。

Oreopoulos，Philip。 （2011）。[为什么技术移民在劳动市场上挣扎？一项涉及一万三千份简历的实地实验](https://github.com/causal-methods/Papers/raw/master/Oreopoulos/Why%20Do%20Skilled%20Immigrants%20Struggle%20in%20the%20Labor%20Market.pdf)。《美国经济学杂志：经济政策》，3（4）：148-71。
