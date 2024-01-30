# 9）股票市场如何缓解以巴冲突？

> 原文：[`causal-methods.github.io/Book/9%29_How_Can_Stock_Market_Mitigate_the_Israeli_Palestinian_Conflict.html`](https://causal-methods.github.io/Book/9%29_How_Can_Stock_Market_Mitigate_the_Israeli_Palestinian_Conflict.html)

[Vitor Kamada](https://www.linkedin.com/in/vitor-kamada-1b73a078)

电子邮件：econometrics.methods@gmail.com

最近更新：11-5-2020

“商业是最具破坏性偏见的良药。”（孟德斯鸠，1748 年：第 II 卷，第一章）

Jha & Shayo（2019）随机将 1345 名犹太以色列选民分为金融资产治疗组和对照组。他们报告称，接触股票市场会增加 4-6%的可能性，投票支持主张和平解决冲突的政党。让我们打开 Jha & Shayo（2019）的数据集。每一行都是以色列公民。

```py
import numpy as np
import pandas as pd
pd.set_option('precision', 3)

# Data from Jha & Shayo (2019)
path = "https://github.com/causal-methods/Data/raw/master/" 
df = pd.read_stata(path + "replicationdata.dta")
df.head(5) 
```

```py
C:\Anaconda\envs\textbook\lib\site-packages\pandas\io\stata.py:1433: UnicodeWarning: 
One or more strings in the dta file could not be decoded using utf-8, and
so the fallback encoding of latin-1 is being used.  This can happen when a file
has been incorrectly encoded by Stata or some other software. You should verify
the string values returned are correct.
  warnings.warn(msg, UnicodeWarning) 
```

|  | 统计 | tradestock6 | 愿意承担 1 到 10 的风险 | 用户 ID | 男性 | 关系 | 关系 ID | nafa | 教育 | ses | ... | tradetot_m4 | pricechange_m4 | bought_bm4 | sold_bm4 | active_bm4 | 资产类型 | 上周 | 过去 3 年 | 下周 | facts_0_m4 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 完成 | NaN | 3 | 60814.0 | 0.0 | 犹太教 | 世俗 | 耶路撒冷 | 硕士 | 平均以下 | ... | 3.0 | -0.673 | 0.0 | 0.0 | 0.0 | 1.0 | 0.0 | 0.0 | 1.0 | 2.0 |
| 1 | 完成 | 1.0 | 2 | 60824.0 | 0.0 | 犹太教 | 世俗 | 特拉维夫 | 文学士 | 平均以上 | ... | 3.0 | 5.323 | 2.0 | 0.0 | 2.0 | 1.0 | 1.0 | 1.0 | 0.0 | 3.0 |
| 2 | 完成 | 0.0 | 3 | 61067.0 | 1.0 | 犹太教 | 世俗 | 中心 | 博士 | 平均以上 | ... | 3.0 | 0.000 | 2.0 | 1.0 | 3.0 | 0.0 | 0.0 | 0.0 | 0.0 | 0.0 |
| 3 | 完成 | NaN | 4 | 61095.0 | 1.0 | 犹太教 | 世俗 | 海法 | 文学士学生 | 平均以下 | ... | 3.0 | 5.323 | 3.0 | 2.0 | 3.0 | 0.0 | 0.0 | 1.0 | 0.0 | 1.0 |
| 4 | 完成 | 0.0 | 4 | 61198.0 | 1.0 | 犹太教 | 世俗 | 北部 | 硕士 | 平均以下 | ... | 3.0 | 0.000 | 0.0 | 0.0 | 0.0 | 1.0 | 0.0 | 0.0 | 0.0 | 1.0 |

5 行×526 列

```py
# Drop missing values
df = df.dropna(subset=['left_s3']) 
```

图 1 显示，2013 年选举中，对照组和治疗组的投票情况相似。但在 2015 年，左翼党派在对照组中获得了 24.8%的选票，而在治疗组中获得了 30.9%。相反，右翼党派在对照组中获得了 35.8%的选票，而在治疗组中获得了 31.2%。根据 Jha & Shayo（2019）的说法，右翼和左翼党派在经济政策方面是相似的。主要区别在于左翼党派支持和平进程，而右翼党派认为任何为和平做出的让步都会对以色列国家构成风险。

```py
# Data: Vote Share by year
v2013 = df.groupby('assettreat')['left_2013', 'right_2013'].mean().T
v2015 = df.groupby('assettreat')['left_s3', 'right_s3'].mean().T
prop = v2013.append(v2015)

# Plot Bar Chart
import plotly.graph_objects as go
node = ['Left 2013', 'Right 2013', 'Left 2015', 'Right 2015']

fig = go.Figure(data=[             
    go.Bar(name='Control', x=node, y = prop[0]),
    go.Bar(name='Treatment', x=node, y = prop[1]) ])

fig.update_layout(barmode='group',
  title_text = 'Graphic 1 - Elections: 2013 vs 2015 ',
  font=dict(size=18) )

fig.update_yaxes(title_text = "Vote Share")

fig.show() 
```

```py
C:\Anaconda\envs\textbook\lib\site-packages\ipykernel_launcher.py:2: FutureWarning: Indexing with multiple keys (implicitly converted to a tuple of keys) will be deprecated, use a list instead.

C:\Anaconda\envs\textbook\lib\site-packages\ipykernel_launcher.py:3: FutureWarning: Indexing with multiple keys (implicitly converted to a tuple of keys) will be deprecated, use a list instead.
  This is separate from the ipykernel package so we can avoid doing imports until 
```

下表显示，治疗组与对照组相似。例外情况是年龄和愿意承担风险。治疗组的以色列人比对照组年轻（39.3 岁对 41.5 岁）。治疗组在愿意承担风险方面也更偏好，评估指数从 1 到 10 变化（4.7 对 4.3）。

我们根据 OLS 回归和分层固定效应计算 p 值。

```py
control = ['right_2013', 'left_2013', 'p_index_s1', 'e_index_init',
    'tradestock6all', 'male', 'age', 'postsecondary', 'BA_student',
	  'college_grad', 'married', 'r_sec',  'r_trad', 'r_relig', 'r_ultra', 
		'g_jerusalem', 'g_north', 'g_haifa', 'g_center', 'g_telaviv', 'g_south',
    'g_wb', 'faminc', 'willingrisk1to10', 'patient', 'plitscore'] 
```

```py
import statsmodels.formula.api as smf

result = []
for var in control:
    # OLS with 104 randomization strata fixed effects
    reg = smf.ols(var + "~ 1 + assettreat + C(block13)", df)
    # 104 is the last variable: the coefficient of assettreat
    pvalue = reg.fit().pvalues[104]
    result.append(pvalue) 
```

```py
C:\Anaconda\envs\textbook\lib\site-packages\statsmodels\tools\_testing.py:19: FutureWarning:

pandas.util.testing is deprecated. Use the functions in the public API at pandas.testing instead. 
```

```py
table = df.groupby('assettreat')[control].mean().T  
table['p-value'] = result
table 
```

| 资产治疗 | 0.0 | 1.0 | p 值 |
| --- | --- | --- | --- |
| right_2013 | 0.245 | 0.241 | 0.964 |
| left_2013 | 0.126 | 0.137 | 0.213 |
| p_index_s1 | 0.004 | 0.051 | 0.399 |
| e_index_init | -0.005 | 0.007 | 0.752 |
| tradestock6all | 0.368 | 0.355 | 0.290 |
| 男性 | 0.513 | 0.521 | 0.470 |
| 年龄 | 41.530 | 39.289 | 0.011 |
| 高中后教育 | 0.232 | 0.230 | 0.953 |
| 文学士学生 | 0.152 | 0.148 | 0.834 |
| 大学毕业 | 0.427 | 0.426 | 0.860 |
| 已婚 | 0.629 | 0.598 | 0.295 |
| r_sec | 0.636 | 0.627 | 0.582 |
| r_trad | 0.172 | 0.164 | 0.823 |
| r_relig | 0.119 | 0.124 | 0.780 |
| r_ultra | 0.073 | 0.085 | 0.222 |
| g_jerusalem | 0.096 | 0.091 | 0.800 |
| g_north | 0.089 | 0.097 | 0.595 |
| g_haifa | 0.123 | 0.142 | 0.290 |
| g_center | 0.298 | 0.290 | 0.766 |
| g_telaviv | 0.212 | 0.194 | 0.276 |
| g_south | 0.116 | 0.104 | 0.596 |
| g_wb | 0.066 | 0.081 | 0.341 |
| 家庭收入 | 11162.162 | 10996.970 | 0.511 |
| 愿意承担风险 1 到 10 | 4.344 | 4.716 | 0.009 |
| 患者 | 0.642 | 0.657 | 0.645 |
| plitscore | 69.726 | 70.664 | 0.550 |

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

表 1 的第 1 列显示了意向治疗（ITT）的估计值为 6.1%。值得一提的是，这种类型的实验很少有完全的遵从。通常，在需要随时间跟踪个体时，控制组和治疗组都会有一定的流失率。

第 2 列显示 ITT 效应对控制变量和分层固定效应的鲁棒性。

第 3 列使用变量“vote_wgt”作为权重呈现了加权最小二乘法（WLS）。Jha & Shayo（2019）提取了一个随机样本，其中非正统中心选民被过度抽样。逻辑是增加对最有趣的群体：摇摆选民的精度。使用权重可以在不过度抽样的情况下重现结果。

第 4 列显示结果是由接受以色列股票（“isrstock”）和投资券（“cash”）的治疗组个体驱动的。要小心得出巴勒斯坦股票没有影响的结论。在实验期间，以色列股票价格上涨，但巴勒斯坦股票下跌。

```py
ITT1 = smf.ols("left_s3 ~ 1 + assettreat",
                 df).fit()

Xs = ['right_2013', 'left_2013', 'male', 'age', 'age2',
      'postsecondary', 'BA_student', 'college_grad',
      'married', 'tradestock6all', 'r_trad', 'r_relig',
      'r_ultra', 'g_jerusalem', 'g_north', 'g_haifa',
      'g_telaviv', 'g_south', 'g_wb', 'C(newses)',
      'willingrisk1to10', 'patient', 'plitscore']

controls = ""
for X in Xs:
    controls = controls + '+' + X      

ITT2 = smf.ols("left_s3 ~ 1 + assettreat" + controls +
                 "+C(block13)", df).fit()

WLS = smf.wls("left_s3 ~ 1 + assettreat" + controls +
       "+C(block13)", df, weights=df['vote_wgt']).fit()

treatments = "+ isrstock + palstock + cash"    
WLS2 = smf.wls("left_s3 ~ 1" + treatments + controls +
       "+C(block13)", df, weights=df['vote_wgt']).fit() 
```

```py
# Settings for a nice table
from stargazer.stargazer import Stargazer
stargazer = Stargazer([ITT1, ITT2, WLS, WLS2])

stargazer.title('Table 1 - Intent to Treat Estimates of Stock'
   + ' Exposure on Voting for a Left Party')

names = ['ITT', 'ITT', 'WLS', 'WLS']
stargazer.custom_columns(names, [1, 1, 1, 1])

stargazer.covariate_order(['assettreat', 'isrstock',
                           'palstock', 'cash'])

stargazer.add_line('Strata Fixed Effects', ['No', 'Yes',
                                            'Yes', 'Yes'])
stargazer.add_line('Covariates', ['No', 'Yes', 'Yes', 'Yes'])

stargazer 
```

表 1 - 对左翼党投票的意向治疗估计股票暴露

|  |
| --- |
|  | *因变量：left_s3* |
|  |
|  | ITT | ITT | WLS | WLS |
|  | (1) | (2) | (3) | (4) |
|  |
| 资产处理 | 0.061^(**) | 0.059^(**) | 0.043^(**) |  |
|  | (0.030) | (0.024) | (0.020) |  |
| isrstock |  |  |  | 0.053^(**) |
|  |  |  |  | (0.024) |
| palstock |  |  |  | 0.024 |
|  |  |  |  | (0.024) |
| 现金 |  |  |  | 0.065^(**) |
|  |  |  |  | (0.029) |
| 分层固定效应 | 否 | 是 | 是 | 是 |
| 协变量 | 否 | 是 | 是 | 是 |
|  |
| 观察 | 1,311 | 1,311 | 1,311 | 1,311 |
| R² | 0.003 | 0.447 | 0.570 | 0.571 |
| 调整后的 R² | 0.002 | 0.385 | 0.522 | 0.522 |
| 残差标准误差 | 0.456 (df=1309) | 0.358 (df=1178) | 0.308 (df=1178) | 0.308 (df=1176) |
| F 统计量 | 4.146^(**) (df=1; 1309) | 7.225^(***) (df=132; 1178) | 11.834^(***) (df=132; 1178) | 11.689^(***) (df=134; 1176) |
|  |
| 注： | ^*p<0.1; ^(**)p<0.05; ^(***)p<0.01 |

表 2 的第 1 列呈现了第一阶段回归。ITT（“资产处理”）是符合分配（“资产 _comp”）的工具变量（IV）。变量“资产 _comp”表示谁实际完成了实验。在控制了几个协变量和分层固定效应之后，ITT 的系数在统计上是显著的。变量“资产处理”是一个完美的 IV。这个变量是随机的。因此，它与误差项不相关。

第 2 列显示了控制函数方法（CF）的结果。对待处理的治疗效应（TOT）估计为 7.3%。接受治疗的个体投票给左翼党的概率增加了 7.3%。在这个框架中，CF 等同于 2SLS。在 CF 中，我们使用第一阶段的残差（$\hat{u}$）来控制第二阶段的内生性。注意残差在统计上是显著的。因此，需要进行修正。我们可以得出结论，表 1 低估了金融资产暴露对左翼党投票的影响。

```py
# Fist Stage
FS = smf.ols("asset_comp ~ 1 + assettreat" + controls +
                 "+C(block13)", df).fit()

# Control Function Approach
df['resid'] = FS.resid
CF = smf.ols("left_s3 ~ 1 + asset_comp + resid" + controls +
                 "+C(block13)", df).fit() 
```

```py
# Settings for a nice table
stargazer = Stargazer([FS, CF])

stargazer.title('Table 2 - Impact of Stock Exposure'
 ' on Voting for a Left Party')

names = ['First Stage', 'Control Function']
stargazer.custom_columns(names, [1, 1])

stargazer.covariate_order(['assettreat', 'asset_comp', 'resid'])

stargazer.add_line('Strata Fixed Effects', ['Yes', 'Yes'])
stargazer.add_line('Covariates', ['Yes', 'Yes'])

stargazer 
```

表 2 - 股票暴露对左翼党投票的影响

|  |
| --- |
|  |
|  | 第一阶段 | 控制函数 |
|  | (1) | (2) |
|  |
| 资产处理 | 0.809^(***) |  |
|  | (0.022) |  |
| 资产 _comp |  | 0.073^(**) |
|  |  | (0.029) |
| 残差 |  | -0.094^(**) |
|  |  | (0.043) |
| 分层固定效应 | 是 | 是 |
| 协变量 | 是 | 是 |
|  |
| 观察 | 1,311 | 1,311 |
| R² | 0.586 | 0.448 |
| 调整后的 R² | 0.540 | 0.385 |
| 残差标准误差 | 0.327 (df=1178) | 0.358 (df=1177) |
| F 统计量 | 12.654^(***) (df=132; 1178) | 7.171^(***) (df=133; 1177) |
|  |
| 注： | ^*p<0.1; ^(**)p<0.05; ^(***)p<0.01 |

## 练习

1）Jha＆Shayo（2019）的样本由以色列选民组成。推测在巴勒斯坦选民的情况下，结果是否会在质量上相同。证明你的理由。

2）从 Jha＆Shayo（2019）出发，有什么有前途的研究问题？

3）什么是社会期望偏差？描述 Jha＆Shayo（2019）为减轻社会期望偏差所做的工作。

4）使用 2SLS 而不是控制函数方法复制表 2 的结果。不要使用库“linearmodels”。使用库“statsmodels”手动进行 2SLS。

5）暴露于巴勒斯坦股票是否会降低投票右翼政党的概率？运行一些回归来证明你的立场。

## 参考

Jha，S.和 Shayo，M.（2019）。[估价和平：金融市场暴露对选票和政治态度的影响](https://github.com/causal-methods/Papers/raw/master/Valuing%20Peace.pdf)。计量经济学，87：1561-1588。

蒙特斯基耶，C.（1748）。[法律精神](https://oll.libertyfund.org/titles/montesquieu-complete-works-vol-1-the-spirit-of-laws)。伦敦：T. Evans，1777 年，4 卷。第 2 卷。在线自由图书馆。
