# 4）新教徒是否比天主教徒更喜欢休闲？

> 原文：[`causal-methods.github.io/Book/4%29_Do_Protestants_Prefer_Less_Leisure_than_Catholics.html`](https://causal-methods.github.io/Book/4%29_Do_Protestants_Prefer_Less_Leisure_than_Catholics.html)

[Vitor Kamada](https://www.linkedin.com/in/vitor-kamada-1b73a078)

电子邮件：econometrics.methods@gmail.com

最后更新时间：10-4-2020

马克斯·韦伯（1930）认为，新教伦理，尤其是加尔文主义的变体，比天主教更符合资本主义。韦伯（1930）观察到，北欧的新教地区比南欧的天主教地区更发达。他假设新教徒工作更努力，储蓄更多，更依赖自己，对政府的期望更少。所有这些特征都将导致更大的经济繁荣。

也许，宗教并不是经济表现出色的原因。教育是一个混杂因素。从历史上看，新教徒有更高的识字水平，因为他们被激励阅读圣经。

因果效应也可能是相反的。也许，一个工业人更有可能成为新教徒。宗教是一个选择变量。人们自我选择符合他们自己世界观的意识形态。

让我们打开 Basten & Betz（2013）的数据。每一行代表瑞士西部的一个市镇，沃州和弗里堡州。

```py
# Load data from Basten & Betz (2013)
import numpy as np
import pandas as pd
path = "https://github.com/causal-methods/Data/raw/master/" 
df = pd.read_stata(path + "finaldata.dta")
df.head(4) 
```

|  | id | name | district | district_name | foreignpop2000 | prot1980s | cath1980s | noreligion1980s | canton | totalpop2000 | ... | pfl | pfi | pfr | reineink_pc_mean | meanpart | popden2000 | foreignpopshare2000 | sub_hs | super_hs | murten |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 2135.0 | Gruyères | 1003.0 | District de la Gruyère | 159 | 0.062251 | 0.907336 | 1.621622 | 10 | 1546.0 | ... | 42.722500 | 52.234196 | 40.143444 | 48.099865 | 40.359428 | 54.455795 | 10.284605 | 85.910339 | 6.770357 | 0.0 |
| 1 | 2128.0 | Châtel-sur-Montsalvens | 1003.0 | District de la Gruyère | 23 | 0.053191 | 0.917526 | 2.061856 | 10 | 205.0 | ... | 49.223751 | 56.793213 | 44.365696 | 42.465569 | 45.434593 | 102.499985 | 11.219512 | 83.832336 | 8.383233 | 0.0 |
| 2 | 2127.0 | Charmey | 1003.0 | District de la Gruyère | 166 | 0.028424 | 0.960818 | 0.255537 | 10 | 1574.0 | ... | 41.087502 | 53.120682 | 39.674942 | 44.451229 | 42.641624 | 20.066286 | 10.546379 | 87.331535 | 6.019766 | 0.0 |
| 3 | 2125.0 | Bulle | 1003.0 | District de la Gruyère | 2863 | 0.053967 | 0.923239 | 1.013825 | 10 | 11149.0 | ... | 47.326248 | 55.033939 | 43.350178 | 50.217991 | 40.885822 | 897.664978 | 25.679434 | 82.203598 | 8.904452 | 0.0 |

4 行×157 列

瑞士在地理和制度方面非常多样化。将生活在阿尔卑斯山的农村天主教徒与苏黎世的城市受过高等教育的新教徒进行比较是不公平的。

从历史上看，城市有不同的激励措施来采纳新教或保持天主教。拥有更强大商人行会的城市更有可能采纳新教；而由贵族统治的城市更有可能保持天主教。

如果我们使用整个国家，混杂因素太多。分析将局限于沃州（历史上是新教徒）和弗里堡（历史上是天主教徒）的州。请参见下面来自 Basten & Betz（2013）的地图。

这片占瑞士 4.5%的 4,883 平方公里地区在制度上和地理上是同质的。1536 年，沃州并不是自愿成为新教徒，而是因为一场战争而被迫。因此，这是一个准实验设置，因为治疗区和对照区在历史事件的影响下相互类似。

![alt text](img/03462ff98f0897d95eb32e15eae7e342.png)

**来源：** Basten & Betz (2013)

在下面的图表中，我们可以看到，市镇中新教徒的比例越高，休闲偏好就越低。蓝色点是历史上的天主教市镇（弗里堡），而红色点是历史上的新教市镇（沃州）。看起来像是不同的子群。我们如何找出是否有因果效应的证据，还是仅仅是相关性？

```py
# Create variable "Region" for the graphic
def category(var):
    if var == 1:
        return "Protestant (Vaud)"
    else:   
        return "Catholic (Fribourg)"
df['Region'] = df["vaud"].apply(category)

# Rename variables with auto-explanatory names
df = df.rename(columns={"prot1980s": "Share_of_Protestants",
                        "pfl": "Preference_for_Leisure"})

# Scatter plot
import plotly.express as px
leisure = px.scatter(df,
                     x="Share_of_Protestants",
                     y="Preference_for_Leisure",
                     color="Region")
leisure.show() 
```

让我们深入分析。记住，回归不连续是最接近实验的技术。

在下面的图表中，偏好休闲在边界距离=0 处存在不连续性。边界距离大于 0 的地区包括历史上的新教市镇（沃州）；而边界距离小于 0 的地区包括历史上的天主教市镇（弗里堡）。

运行变量“边界距离”决定了地区，但并不决定新教徒的比例。然而，新教徒的比例随着距离的增加而增加。

```py
df = df.rename(columns={"borderdis": "Border_Distance_in_Km"})

leisure = px.scatter(df,
                     x="Border_Distance_in_Km",
                     y="Preference_for_Leisure",
                     color="Share_of_Protestants",
                     title="Discontinuity at Distance = 0")
leisure.show() 
```

由于边界是任意的，即由历史事件决定的，靠近边界的市镇很可能彼此更相似，而远离边界的市镇。因此，让我们将分析限制在 5 公里范围内的市镇。

```py
df5 = df[df['Border_Distance_in_Km'] >= -5]
df5 = df5[df5['Border_Distance_in_Km'] <= 5] 
```

简单的均值比较显示，新教市镇的休闲偏好较低（39.5%）与天主教市镇（48.2%）相比。差异为-8.7%。

请注意，新教地区的瑞士法郎平均收入水平较高（47.2K vs 43.7K），而基尼系数捕捉到的不平等程度也较高（0.36 vs 0.30）。

```py
df5 = df5.rename(columns={"reineink_pc_mean": "Mean_Income_(CHF)",
                          "Ecoplan_gini"    : "Gini_1996"})

outcome = ['Preference_for_Leisure', 'Mean_Income_(CHF)', 'Gini_1996'] 
```

考虑到只使用了 5 公里范围内的市镇（49 个天主教市镇和 84 个新教市镇），上述比较并不“糟糕”。此外，两个地区在 1980 年的无宗教信仰比例（1.7% vs 2.9%）和海拔高度（642 vs 639 米）方面相似。

然而，一个更可信的方法是使用一个带有运行变量($r_i$)的回归不连续框架。

```py
control = ['noreligion1980s', 'altitude']
df5.loc[:, control].groupby(df5['Region']).agg([np.mean, np.std]).T 
```

|  | 地区 | 天主教（弗里堡） | 新教（沃州） |
| --- | --- | --- | --- |
| 无宗教 1980 年代 | 平均值 | 1.729423 | 2.949534 |
| std | 1.499346 | 2.726086 |
| 海拔 | 平均值 | 642.591858 | 639.607117 |
| std | 120.230320 | 113.563847 |

在图表“模糊回归不连续”中，我们可以清楚地看到运行变量“边界距离”与处理变量“新教徒比例”之间存在很强的相关性。变量“边界距离”并不决定处理状态，但增加了成为新教徒的概率。因此，这是一个模糊回归不连续的情况，而不是一个尖锐的回归不连续。

让$D_i$成为单位$i$的处理状态。$P(D_i=1|r_i)$是在截断$r_0$处处理概率的跳跃：

$$P(D_i=1|r_i)$$$$= f_1(r_i) \ if \ r_i\geq r_0$$$$= f_0(r_i) \ if \ r_i< r_0$$

其中$f_1(r_i)$和$f_0(r_i)$是可以假定任何值的函数。在尖锐的回归不连续中，$f_1(r_i)$为 1，$f_0(r_i)$为 0。

```py
fuzzy = px.scatter(df5,
                     x="Border_Distance_in_Km",
                     y="Share_of_Protestants",
                     color="Region",
                     title="Fuzzy Regression Discontinuity")
fuzzy.show() 
```

在下面的图表中，新教徒比例变量被模拟，以说明尖锐的回归不连续的情况。

```py
def dummy(var):
    if var >= 0:
        return 1
    else:   
        return 0

df5["Simulated_Share_Protestant"] = df5["Border_Distance_in_Km"].apply(dummy)

sharp = px.scatter(df5,
                     x="Border_Distance_in_Km",
                     y="Simulated_Share_Protestant",
                     color="Region",
                     title="Sharp Regression Discontinuity")
sharp.show() 
```

假设$Y$ = 休闲偏好，$D_r$ = 新教徒比例，$r$ = 边界距离。现在，我们有一个估计以下方程的问题：

$$Y = \beta_0+\rho D_r+ \beta_1r+\epsilon$$

兴趣变量$D_r$不再被$r$“净化”，也就是说，它不再完全由运行变量$r$决定。因此，$D_r$很可能与误差项$\epsilon$相关：

$$Cov(D_r, \epsilon)\neq0$$

我们可以使用一个与误差项$\epsilon$不相关且在控制其他因素后与$D_r$相关的工具变量$Z$来解决这个问题：

$$Cov(Z, \epsilon) = 0$$$$Cov(Z, D_r) \neq 0$$

$Z$的自然候选人是变量“vaud”：如果是瓦州的市政当局，则为 1；如果是弗里堡的市政当局，则为 0。没有理由相信这个变量与误差项$\epsilon$相关，因为分隔该地区的边界是在 1536 年确定的，当时伯尔尼共和国征服了瓦州。第二个假设也是有效的，因为瓦州地区的新教徒比弗里堡地区更多。

工具变量方法首先是使用$Z$“纯化”$D_r$：

$$D_r=\gamma_0+\gamma_1Z+\gamma_2r+\upsilon$$

然后，我们通过进行普通最小二乘（OLS）回归得到$\hat{D}_r$的拟合值，并将其代入方程：

$$Y = \beta_0+\rho \hat{D}_r+ \beta_1r+\epsilon$$

逻辑是“纯化”的$\hat{D}_r$与误差项$\epsilon$不相关。现在，我们可以运行普通最小二乘（OLS）来得到$\hat{D}_r$对$Y$的孤立影响，即$\rho$将是无偏的因果效应。

```py
# Install libray to run Instrumental Variable estimation
!pip install linearmodels 
```

```py
Requirement already satisfied: linearmodels in c:\anaconda\envs\textbook\lib\site-packages (4.17)
Requirement already satisfied: scipy>=1 in c:\anaconda\envs\textbook\lib\site-packages (from linearmodels) (1.5.3)
Requirement already satisfied: pandas>=0.23 in c:\anaconda\envs\textbook\lib\site-packages (from linearmodels) (1.1.3)
Requirement already satisfied: property-cached>=1.6.3 in c:\anaconda\envs\textbook\lib\site-packages (from linearmodels) (1.6.4)
Requirement already satisfied: patsy in c:\anaconda\envs\textbook\lib\site-packages (from linearmodels) (0.5.1)
Requirement already satisfied: Cython>=0.29.14 in c:\anaconda\envs\textbook\lib\site-packages (from linearmodels) (0.29.21)
Requirement already satisfied: statsmodels>=0.9 in c:\anaconda\envs\textbook\lib\site-packages (from linearmodels) (0.10.2)
Requirement already satisfied: numpy>=1.15 in c:\anaconda\envs\textbook\lib\site-packages (from linearmodels) (1.19.2) 
```

```py
WARNING: No metadata found in c:\anaconda\envs\textbook\lib\site-packages
ERROR: Could not install packages due to an EnvironmentError: [Errno 2] No such file or directory: 'c:\\anaconda\\envs\\textbook\\lib\\site-packages\\numpy-1.19.2.dist-info\\METADATA' 
```

计算机自动运行工具变量（IV）程序的两个阶段。我们指出内生变量$D_r$是“新教徒比例”，工具变量$Z$是“vaud”。我们还添加了变量“t_dist”，即变量“vaud”和“边境距离”的交互。

结果是，“新教徒比例”使休闲偏好减少了 13.4%。

```py
from linearmodels.iv import IV2SLS
iv = 'Preference_for_Leisure ~ 1 + Border_Distance_in_Km + t_dist + [Share_of_Protestants ~ vaud]'
iv_result = IV2SLS.from_formula(iv, df5).fit(cov_type='robust')

print(iv_result) 
```

```py
C:\Anaconda\envs\textbook\lib\site-packages\statsmodels\tools\_testing.py:19: FutureWarning:

pandas.util.testing is deprecated. Use the functions in the public API at pandas.testing instead. 
```

```py
 IV-2SLS Estimation Summary                            
==================================================================================
Dep. Variable:     Preference_for_Leisure   R-squared:                      0.4706
Estimator:                        IV-2SLS   Adj. R-squared:                 0.4583
No. Observations:                     133   F-statistic:                    99.425
Date:                    Fri, Nov 06 2020   P-value (F-stat)                0.0000
Time:                            21:12:05   Distribution:                  chi2(3)
Cov. Estimator:                    robust                                         

                                   Parameter Estimates                                   
=========================================================================================
                       Parameter  Std. Err.     T-stat    P-value    Lower CI    Upper CI
-----------------------------------------------------------------------------------------
Intercept                 50.528     1.8885     26.755     0.0000      46.826      54.229
Border_Distance_in_Km     0.4380     0.6291     0.6963     0.4862     -0.7949      1.6710
t_dist                   -0.3636     0.7989    -0.4552     0.6490     -1.9294      1.2022
Share_of_Protestants     -13.460     3.1132    -4.3235     0.0000     -19.562     -7.3582
=========================================================================================

Endogenous: Share_of_Protestants
Instruments: vaud
Robust Covariance (Heteroskedastic)
Debiased: False 
```

```py
C:\Anaconda\envs\textbook\lib\site-packages\linearmodels\iv\data.py:25: FutureWarning:

is_categorical is deprecated and will be removed in a future version.  Use is_categorical_dtype instead 
```

我们还可以检查第一阶段，看看工具变量“vaud”是否与“新教徒比例”相关联，控制其他因素如“边境距离”和“t_dist”后。瓦州使新教徒比例增加了 67%。“vaud”的 t 值为 20，即在没有任何疑问的情况下具有统计学意义。

因此，我们确信第二阶段的结果比简单的均值比较更可信。13.4%的工具变量影响比 8.7%的简单均值比较更可信。

```py
print(iv_result.first_stage) 
```

```py
 First Stage Estimation Results        
===============================================
                           Share_of_Protestants
-----------------------------------------------
R-squared                                0.9338
Partial R-squared                        0.7095
Shea's R-squared                         0.7095
Partial F-statistic                      403.04
P-value (Partial F-stat)                 0.0000
Partial F-stat Distn                    chi2(1)
==========================          ===========
Intercept                                0.1336
                                       (7.7957)
Border_Distance_in_Km                    0.0169
                                       (2.8397)
t_dist                                  -0.0057
                                      (-0.4691)
vaud                                     0.6709
                                       (20.076)
-----------------------------------------------

T-stats reported in parentheses
T-stats use same covariance type as original model 
```

8.7%的简单均值比较结果更接近于下面的天真锐度回归（SRD）的结果为 9%。瓦州地区对休闲的偏好比弗里堡少 9%。我们不能得出新教徒对休闲的偏好比天主教徒少 9%的结论。瓦州地区并不是 100%的新教徒。弗里堡地区也不是 100%的天主教徒。

Fuzz 回归离散度（FRD）使用工具变量（IV）估计，是对天真比较的修正。FRD 隔离了新教徒对休闲偏好的影响。因此，最可信的估计是，新教徒对休闲的偏好比天主教徒少 13.4%。

```py
naive_srd = 'Preference_for_Leisure ~ 1 + vaud + Border_Distance_in_Km + t_dist'
srd = IV2SLS.from_formula(naive_srd, df5).fit(cov_type='robust')
print(srd) 
```

```py
 OLS Estimation Summary                              
==================================================================================
Dep. Variable:     Preference_for_Leisure   R-squared:                      0.3830
Estimator:                            OLS   Adj. R-squared:                 0.3686
No. Observations:                     133   F-statistic:                    91.767
Date:                    Fri, Nov 06 2020   P-value (F-stat)                0.0000
Time:                            21:12:05   Distribution:                  chi2(3)
Cov. Estimator:                    robust                                         

                                   Parameter Estimates                                   
=========================================================================================
                       Parameter  Std. Err.     T-stat    P-value    Lower CI    Upper CI
-----------------------------------------------------------------------------------------
Intercept                 48.730     1.5731     30.976     0.0000      45.646      51.813
vaud                     -9.0303     2.1821    -4.1383     0.0000     -13.307     -4.7534
Border_Distance_in_Km     0.2107     0.6028     0.3496     0.7266     -0.9707      1.3922
t_dist                   -0.2874     0.8479    -0.3390     0.7346     -1.9493      1.3744
========================================================================================= 
```

## 练习

1）在估计瑞士（整个国家）的新教徒对天主教徒的经济繁荣的因果影响时，会有哪些混杂因素？详细解释每个混杂因素。

2）有人可能会认为在 1536 年伯尔尼共和国征服瓦州之前，西部瑞士是一个多样化的地区。在这种情况下，弗里堡（天主教徒）对瓦州（新教徒）来说不是一个合理的对照组。在 1536 年之前，可以使用哪些变量来测试该地区的同质性/多样性？指出数据是否存在，或者是否可以收集数据。

3）我复制了 Basten 和 Betz（2013）的主要结果，并注意到他们没有控制教育因素。Becker & Woessmann（2009）认为普鲁士在 19 世纪的新教徒对经济繁荣的影响是由于较高的识字率。Basten 和 Betz（2013）在[在线附录](https://github.com/causal-methods/Papers/raw/master/Beyond-Work-Ethic/2011-0231_app.pdf)中呈现了下表。表中的数字是加强还是削弱了 Basten 和 Betz（2013）的结果？请解释你的推理。

![alt text](img/cb3d12a1b61905e0fce56d1f245903ca.png)

**来源：** Basten & Betz（2013）

4）休闲偏好是一项调查中的自我报告变量。也许，新教徒比天主教徒更有动机宣称对休闲的偏好较低。在现实世界中，新教徒可能和天主教徒一样喜欢休闲。声明的偏好可能与真实行为不符。相关的研究问题是宗教（新教）是否导致人们实际上努力工作。要有创意地提出一种方法（方法和数据）来解决上述问题。

5）使用巴斯滕和贝茨（2013）的数据，调查新教徒是否导致瑞士西部的平均收入更高。采用您认为最可信的规范来恢复无偏因果效应。解释和证明推理的每一步。解释主要结果。

## 参考文献

巴斯滕，克里斯托夫和弗兰克·贝茨（2013）。[超越职业道德：宗教，个人和政治偏好](https://github.com/causal-methods/Papers/raw/master/Beyond-Work-Ethic/Beyond%20Work%20Ethic.pdf)。《美国经济学杂志：经济政策》5（3）：67-91。[在线附录](https://github.com/causal-methods/Papers/raw/master/Beyond-Work-Ethic/2011-0231_app.pdf)

贝克尔，萨沙 O.和鲁德格·沃斯曼。 （2009）。韦伯错了吗？新教经济史的人力资本理论。《季度经济学杂志》124（2）：531–96。

韦伯，马克思。 （1930）。[新教伦理与资本主义精神](https://www.marxists.org/reference/archive/weber/protestant-ethic/)。纽约：斯克里布纳，（原始出版于 1905 年）。
