# 七、卖淫合法化对犯罪的影响

> 原文：[`causal-methods.github.io/Book/7%29_The_Impact_of_Legalizing_Prostitution_on_Crime.html`](https://causal-methods.github.io/Book/7%29_The_Impact_of_Legalizing_Prostitution_on_Crime.html)
>
> 译者：[飞龙](https://github.com/wizardforcel)
>
> 协议：[CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/)


[Vitor Kamada](https://www.linkedin.com/in/vitor-kamada-1b73a078)

电子邮件：econometrics.methods@gmail.com

最近更新：2020 年 11 月 2 日

在荷兰，有法定的卖淫区，荷兰称之为 tippelzones。Bisschop 等人（2017）报告称，tippelzone 的开放可以减少大约 30-40%的性虐待和强奸案件。

让我们打开 Bisschop 等人的数据集。每一行是荷兰的一个城市。同一个城市在 1994 年至 2011 年之间被观察到。

```py
import numpy as np
import pandas as pd
pd.set_option('precision', 3)

# Data from Bisschop et al. (2017)
path = "https://github.com/causal-methods/Data/raw/master/" 
df = pd.read_stata(path + "CBSregist2015.dta")
df.head(5) 
```

|  | 城市 | 年份 | 开放 | 关闭 | 城市 1 | logpopdens | openingReg | mayorCDA | mayorCU | mayorD66 | ... | 相似盗窃率 | 聚合盗窃率 | 相似盗窃率的自然对数 | 聚合盗窃率的自然对数 | 盗窃率 | 盗窃率的自然对数 | 职务侵犯率 | 职务暴力率 | 职务侵犯率的自然对数 | 职务暴力率的自然对数 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 阿姆斯特丹 | 1994-01-01 | 0.0 | 0.0 | 1.0 | 8.381 | 0.0 | 0.0 | 0.0 | 0.0 | ... | 59.246 | 57.934 | 8.364 | 8.342 | 117.181 | 9.046 | 7.596 | 5.110 | 6.310 | 5.914 |
| 1 | 阿姆斯特丹 | 1995-01-01 | 0.0 | 0.0 | 1.0 | 8.379 | 0.0 | 0.0 | 0.0 | 0.0 | ... | 50.815 | 43.823 | 8.208 | 8.060 | 94.637 | 8.830 | 7.061 | 4.361 | 6.234 | 5.753 |
| 2 | 阿姆斯特丹 | 1996-01-01 | 1.0 | 0.0 | 1.0 | 8.373 | 0.0 | 0.0 | 0.0 | 0.0 | ... | 42.333 | 37.111 | 8.020 | 7.888 | 79.444 | 8.649 | 7.520 | 5.431 | 6.292 | 5.966 |
| 3 | 阿姆斯特丹 | 1997-01-01 | 1.0 | 0.0 | 1.0 | 8.369 | 0.0 | 0.0 | 0.0 | 0.0 | ... | 46.843 | 32.860 | 8.117 | 7.762 | 79.704 | 8.648 | 6.852 | 4.195 | 6.194 | 5.704 |
| 4 | 阿姆斯特丹 | 1998-01-01 | 1.0 | 0.0 | 1.0 | 8.373 | 0.0 | 0.0 | 0.0 | 0.0 | ... | 45.255 | 33.907 | 8.086 | 7.798 | 79.162 | 8.646 | 6.127 | 4.595 | 6.087 | 5.799 |

5 行×65 列

让我们将城市分成 3 组。大城市和中等城市至少在某一年有 tippelzone，而样本中的其他城市没有 tippelzone。

```py
big_cities = ["Amsterdam", "Rotterdam", "Den Haag"]
medium_cities = ["Utrecht", "Nijmegen", "Groningen",
                 "Heerlen", "Eindhoven", "Arnhem"]

# Classify cities
def classify(var):
    if var in big_cities:
        return "Big Cities"
    elif var in medium_cities:
        return "Medium Cities"    
    else:   
        return "No tippelzone"

df['group'] = df["city"].apply(classify) 
```

以下是每 10,000 名居民的年度犯罪报告。总体而言，大城市的犯罪率更高。唯一的例外是与毒品有关的犯罪。

```py
outcome = ['sexassaultpcN', 'rapepcN', 
	'drugspcN', 'maltreatpcN', 'weaponspcN']

df.groupby('group')[outcome].mean().T 
```

| 组 | 大城市 | 中等城市 | 无 tippelzone |
| --- | --- | --- | --- |
| 性侵犯率 | 0.775 | 0.626 | 0.664 |
| 强奸率 | 1.032 | 0.846 | 0.691 |
| 毒品犯罪率 | 14.921 | 15.599 | 12.779 |
| maltreatpcN | 21.259 | 18.665 | 17.864 |
| weaponspcN | 5.635 | 4.385 | 4.207 |

tippelzones 城市的人口和人口密度比没有 tippelzones 的城市更多。平均家庭收入（以 1,000 欧元计）在 3 个组中相似。tippelzones 城市也有受过较高教育的个体。移民比例在大城市中更高（11.4%）。社会保险福利（“insurWWAO_pc”）的份额与 3 个组相似。

```py
demographics = ['popul_100', 'pop_dens_100', 'popmale1565_100',
            'inkhh', 'educhpc', 'nondutchpc', 'insurWWAO_pc']

df.groupby('group')[demographics].mean().T 
```

| 组 | 大城市 | 中等城市 | 无 tippelzone |
| --- | --- | --- | --- |
| popul_100 | 5974.886 | 1724.191 | 1131.138 |
| pop_dens_100 | 43.258 | 22.977 | 19.560 |
| popmale1565_100 | 2101.446 | 617.019 | 392.255 |
| inkhh | 29.052 | 28.989 | 30.502 |
| educhpc | 0.300 | 0.317 | 0.245 |
| 非荷兰人比例 | 0.114 | 0.059 | 0.052 |
| insurWWAO_pc | 0.074 | 0.081 | 0.078 |

基督教联盟在没有 tippelzone 的城市中拥有更多的市长（31%）。值得一提的是，该党反对开放 tippelzone。

```py
political_party = ['mayorSoc', 'mayorLib', 'mayorChr']
df.groupby('group')[political_party].mean().T 
```

| 组 | 大城市 | 中等城市 | 无 tippelzone |
| --- | --- | --- | --- |
| mayorSoc | 0.481 | 0.556 | 0.410 |
| mayorLib | 0.259 | 0.324 | 0.278 |
| mayorChr | 0.259 | 0.120 | 0.312 |

数据集是平衡的面板数据。有必要按顺序声明指数：分析单位和时间单位。

```py
df['year'] = pd.DatetimeIndex(df['year']).year
df['Dyear'] = pd.Categorical(df.year)

# Set Panel Data
# Set city as the unit of analysis
df25 = df.set_index(['city1', 'year']) 
```

设$Y_{ct}$为城市$c$在年份$t$的犯罪率。设$D_{ct}$ = 1，如果城市$c$在年份$t$有开放的 tippelzone；否则为 0。让我们估计以下模型：

$$ln(Y_{ct})=\alpha_c+\rho D_{ct}+\beta X_{ct}+\gamma_t + \epsilon_{ct}$$

其中$\alpha_c$是城市固定效应，$X_{ct}$是控制变量向量，$\gamma_t$是年固定效应，$\epsilon_{ct}$是通常的误差项。

```py
import statsmodels.formula.api as smf

Ys = ["lnsexassaultN", "lnrapeN", "lndrugsN",
      "lnweaponsN", "lnmaltreatN"]

base = "~ 1 + opening"
fe = "+ C(city) + C(Dyear)"

controls = ['logpopmale1565', 'logpopdens', 'inkhh', 
	'educhpc', 'nondutchpc', 'insurWWAO', 'mayorCDA',
  'mayorCU', 'mayorD66', 'mayorVVD']

Xs = ""
for var in controls:
    Xs = Xs + '+' + var

columns = []
for Y in Ys:
  result = smf.ols(Y + base + fe + Xs, df25).fit(cov_type='cluster',
                cov_kwds={'groups': df25['city']})
  columns.append(result) 
```

```py
C:\Anaconda\envs\textbook\lib\site-packages\statsmodels\tools\_testing.py:19: FutureWarning: pandas.util.testing is deprecated. Use the functions in the public API at pandas.testing instead.
  import pandas.util.testing as tm 
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

第 1 列表明开放 tippelzone 将性虐待减少 26%（$e^{-0.302}-1$）。在其他列中，tippelzone 的系数在统计上不显著。看起来合法化卖淫会减少性虐待，但不会减少其他犯罪，如强奸、攻击、非法武器和毒品相关犯罪。

```py
# Settings for a nice table
from stargazer.stargazer import Stargazer
stargazer = Stargazer(columns)

stargazer.title('The Impact of Tippelzone on Crime')

names = ['Sex Abuse', 'Rape', 'Drugs', 'Weapons', 'Assault']
stargazer.custom_columns(names, [1, 1, 1, 1, 1])

stargazer.covariate_order(['opening'])

stargazer.add_line('Covariates', ['Yes', 'Yes', 'Yes', 'Yes', 'Yes'])

stargazer.add_line('City Fixed Effects', ['Yes', 'Yes', 'Yes', 'Yes', 'Yes'])
stargazer.add_line('Year Fixed Effects', ['Yes', 'Yes', 'Yes', 'Yes', 'Yes'])

stargazer 
```

```py
C:\Anaconda\envs\textbook\lib\site-packages\statsmodels\base\model.py:1752: ValueWarning: covariance of constraints does not have full rank. The number of constraints is 52, but rank is 24
  'rank is %d' % (J, J_), ValueWarning)
C:\Anaconda\envs\textbook\lib\site-packages\statsmodels\base\model.py:1752: ValueWarning: covariance of constraints does not have full rank. The number of constraints is 52, but rank is 24
  'rank is %d' % (J, J_), ValueWarning)
C:\Anaconda\envs\textbook\lib\site-packages\statsmodels\base\model.py:1752: ValueWarning: covariance of constraints does not have full rank. The number of constraints is 52, but rank is 24
  'rank is %d' % (J, J_), ValueWarning)
C:\Anaconda\envs\textbook\lib\site-packages\statsmodels\base\model.py:1752: ValueWarning: covariance of constraints does not have full rank. The number of constraints is 52, but rank is 24
  'rank is %d' % (J, J_), ValueWarning)
C:\Anaconda\envs\textbook\lib\site-packages\statsmodels\base\model.py:1752: ValueWarning: covariance of constraints does not have full rank. The number of constraints is 52, but rank is 24
  'rank is %d' % (J, J_), ValueWarning) 
```

Tippelzone 对犯罪的影响

|  |
| --- |
|  |
|  | 性虐待 | 强奸 | 毒品 | 武器 | 攻击 |
|  | (1) | (2) | (3) | (4) | (5) |
|  |
| 开放 | -0.302^(***) | -0.042 | -0.057 | -0.074 | -0.017 |
|  | (0.091) | (0.060) | (0.076) | (0.093) | (0.060) |
| 协变量 | 是 | 是 | 是 | 是 | 是 |
| 城市固定效应 | 是 | 是 | 是 | 是 | 是 |
| 年固定效应 | 是 | 是 | 是 | 是 | 是 |
|  |
| 观察 | 450 | 450 | 450 | 450 | 450 |
| R² | 0.727 | 0.794 | 0.899 | 0.907 | 0.957 |
| 调整 R² | 0.692 | 0.767 | 0.886 | 0.895 | 0.952 |
| 残差标准误差 | 0.485 (df=397) | 0.449 (df=397) | 0.330 (df=397) | 0.289 (df=397) | 0.165 (df=397) |
| F 统计量 | 26.328^(***) (df=52; 397) | 65.173^(***) (df=52; 397) | 45.273^(***) (df=52; 397) | 10.007^(***) (df=52; 397) | 252.545^(***) (df=52; 397) |
|  |
| 注意： | ^*p<0.1; ^(**)p<0.05; ^(***)p<0.01 |

```py
import math
math.exp(-0.302) - 1 
```

```py
-0.2606619351104681 
```

## 练习

1）在论文的引言部分，Bisschop 等人（2017: 29）表示：“我们的研究是第一个提供卖淫监管与犯罪之间关联的因果证据之一。”在讨论部分，Bisschop 等人（2017:44）表示：“开放 tippelzone，无论是否有许可制度，都与性虐待和强奸的短期减少 30-40%相关，并且结果在不同规范下都是稳健的。”为什么 Bisschop 等人（2017）在引言部分使用“因果”一词，在讨论部分使用“相关”一词？您是否认为 Bisschop 等人（2017）的主要结果是“因果”还是“相关”？请解释。

2）Bisschop 等人（2017: 29）表示：“我们进行了几项实证测试，以评估 tippelzone 开放周围的内生犯罪趋势。”他们为什么这样做？这其中的逻辑是什么？是否存在内生犯罪趋势？请解释并具体说明您的答案。

3）Bisschop 等人（2017: 36）表示：“...时间趋势$\mu_t$是使用年固定效应来建模的。”模拟时间趋势的其他方法是什么？编写不同假设下创建时间趋势的代码片段。提示：记住这是面板数据。在横截面数据中有效的代码将在面板数据结构中创建错误的变量。

4）Bisschop 等人（2017: 36）表示：“我们使用差异中的差异规范来研究 tippelzone 存在对各种犯罪的影响。”部署差异中的差异估计器的关键假设是什么？

5）复制表格“Tippelzone 对犯罪的影响”，不包括阿姆斯特丹、鹿特丹和海牙。此外，用以下四个变量替换变量“opening”：

i) “everopenNoReg”: 如果城市$c$在年份$t$之前曾开放过没有许可证的 tippelzone，则为 1，否则为 0。

ii) “openingRegP”: 如果城市$c$在年份$t$之前开放了 tippelzone 并引入了事后许可证，则为 1，否则为 0。

iii) “openingRegA”: 如果城市$c$在年份$t$之前曾开放过带有许可证的 tippelzone，则为 1，否则为 0。

iv) “closing”: 如果城市$c$在年份$t$之前关闭 tippelzone，则为 1，否则为 0。

解释结果。

## 参考

Bisschop, Paul, Stephen Kastoryano, and Bas van der Klaauw. (2017). [街头卖淫区和犯罪](https://github.com/causal-methods/Papers/raw/master/Street%20Prostitution%20Zones%20and%20Crime.pdf). 美国经济学杂志：经济政策，9 (4): 28-63.
