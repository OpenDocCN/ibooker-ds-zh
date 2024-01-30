# 3) 在伊斯兰或世俗政权下，女性更有可能完成高中吗？

> 原文：[`causal-methods.github.io/Book/3%29_Are_Females_More_Likely_to_Complete_High_School_Under_Islamic_or_Secular_Regime.html`](https://causal-methods.github.io/Book/3%29_Are_Females_More_Likely_to_Complete_High_School_Under_Islamic_or_Secular_Regime.html)

[Vitor Kamada](https://www.linkedin.com/in/vitor-kamada-1b73a078)

电子邮件：econometrics.methods@gmail.com

最后更新：10-3-2020

让我们打开 Meyersson（2014）的数据。每一行代表土耳其的一个市镇。

```py
# Load data from Meyersson (2014)
import numpy as np
import pandas as pd
path = "https://github.com/causal-methods/Data/raw/master/" 
df = pd.read_stata(path + "regdata0.dta")
df.head() 
```

|  | province_pre | ilce_pre | belediye_pre | province_post | ilce_post | belediye_post | vshr_islam2004 | educpop_1520f | hischshr614m | hischshr614f | ... | jhischshr1520m | jhivshr1520f | jhivshr1520m | rpopshr1520 | rpopshr2125 | rpopshr2630 | rpopshr3164 | nonagshr1530f | nonagshr1530m | anyc |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | Adana | Aladag | Aladag | Adana | Aladag | Aladag | 0.367583 | 540.0 | 0.0 | 0.0 | ... | 0.478448 | 0.012963 | 0.025862 | 1.116244 | 1.113730 | 0.955681 | 0.954823 | 0.046778 | 0.273176 | 1.0 |
| 1 | Adana | Aladag | Akoren | Adana | Aladag | Akoren | 0.518204 | 342.0 | 0.0 | 0.0 | ... | 0.513089 | 0.023392 | 0.020942 | 1.002742 | 0.993227 | 1.093731 | 1.018202 | 0.020325 | 0.146221 | 0.0 |
| 2 | Adana | Buyuksehir | Buyuksehir | Adana |  | Buyuksehir | 0.397450 | 76944.0 | 0.0 | 0.0 | ... | 0.348721 | 0.036871 | 0.060343 | 1.006071 | 1.094471 | 1.039968 | 0.990001 | 0.148594 | 0.505949 | 1.0 |
| 3 | Adana | Ceyhan | Sarimazi | Adana | Ceyhan | Sarimazi | 0.559827 | 318.0 | 0.0 | 0.0 | ... | 0.331343 | 0.022013 | 0.074627 | 1.124591 | 0.891861 | 0.816490 | 0.916518 | 0.040111 | 0.347439 | 0.0 |
| 4 | Adana | Ceyhan | Sagkaya | Adana | Ceyhan | Sagkaya | 0.568675 | 149.0 | 0.0 | 0.0 | ... | 0.503650 | 0.053691 | 0.043796 | 1.079437 | 1.208691 | 1.114033 | 0.979060 | 0.070681 | 0.208333 | 0.0 |

5 行×236 列

变量‘hischshr1520f’是根据 2000 年人口普查数据，15-20 岁女性完成高中的比例。不幸的是，年龄是被聚合的。15 和 16 岁的青少年不太可能在土耳其完成高中。最好是有按年龄分开的数据。由于 15 和 16 岁的人无法从分析中移除，15-20 岁女性完成高中的比例非常低：16.3%。

变量‘i94’是如果一个伊斯兰市长在 1994 年赢得了市镇选举则为 1，如果一个世俗市长赢得了则为 0。伊斯兰党在土耳其执政了 12%的市镇。

```py
# Drop missing values
df = df.dropna(subset=['hischshr1520f', 'i94'])

# Round 2 decimals
pd.set_option('precision', 4)

# Summary Statistics
df.loc[:, ('hischshr1520f', 'i94')].describe()[0:3] 
```

|  | hischshr1520f | i94 |
| --- | --- | --- |
| count | 2632.0000 | 2632.0000 |
| mean | 0.1631 | 0.1197 |
| std | 0.0958 | 0.3246 |

在由伊斯兰市长执政的市镇中，15-20 岁女性的平均高中完成率为 14%，而在由世俗市长执政的市镇中为 16.6%。

这是一个天真的比较，因为数据不是来自实验。市长类型并没有被随机化，实际上也不能被随机化。例如，贫困可能导致更高水平的宗教信仰和更低的教育成就。导致高中完成率较低的可能是贫困而不是宗教。

```py
df.loc[:, ('hischshr1520f')].groupby(df['i94']).agg([np.size, np.mean]) 
```

|  | size | mean |
| --- | --- | --- |
| i94 |  |  |
| --- | --- | --- |
| 0.0 | 2317.0 | 0.1662 |
| 1.0 | 315.0 | 0.1404 |

图表“天真比较”显示控制组和实验组是基于变量‘iwm94’：伊斯兰赢得的边际。该变量被居中为 0。因此，如果赢得的边际高于 0，伊斯兰市长赢得了选举。另一方面，如果赢得的边际低于 0，伊斯兰市长输掉了选举。

就平均高中学历而言，治疗组（14%）和对照组（16.6%）之间的差异为-2.6%。使用观察数据比较市政结果的问题在于，治疗组与对照组并不相似。因此，混杂因素可能会使结果产生偏差。

```py
import matplotlib.pyplot as plt

# Scatter plot with vertical line
plt.scatter(df['iwm94'], df['hischshr1520f'], alpha=0.2)
plt.vlines(0, 0, 0.8, colors='red', linestyles='dashed')

# Labels
plt.title('Naive Comparison')
plt.xlabel('Islamic win margin')
plt.ylabel('Female aged 15-20 with high school')

# Control vs Treatment
plt.text(-1, 0.7, r'$\bar{y}_{control}=16.6\%$', fontsize=16,
         bbox={'facecolor':'yellow', 'alpha':0.2})
plt.text(0.2, 0.7, r'$\bar{y}_{treatment}=14\%$', fontsize=16,
         bbox={'facecolor':'yellow', 'alpha':0.2})
plt.show() 
```

![_images/3)_Are_Females_More_Likely_to_Complete_High_School_Under_Islamic_or_Secular_Regime_9_0.png](img/393fff974058173a23fbecd049fa95eb.png)

由伊斯兰主导的高中学历与世俗主导的高中学历之间的 2.6%差异在 1%的显著水平上是统计学显著的。考虑到高中毕业率的平均值为 16.3%，这个幅度也是相关的。但是，请注意，这是一个天真的比较，可能存在偏见。

```py
# Naive Comparison
df['Intercept'] = 1
import statsmodels.api as sm
naive = sm.OLS(df['hischshr1520f'], df[['Intercept', 'i94']],
                    missing='drop').fit()
print(naive.summary().tables[1]) 
```

```py
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
Intercept      0.1662      0.002     83.813      0.000       0.162       0.170
i94           -0.0258      0.006     -4.505      0.000      -0.037      -0.015
============================================================================== 
```

判断天真比较是否可能存在偏见的一种方法是检查由伊斯兰主导的市政是否与由世俗主导的市政不同。

伊斯兰主要赢得的市政当中，1994 年伊斯兰选票比例更高（41% vs 10%），获得选票的政党数量更多（5.9 vs 5.5），对数人口更多（8.3 vs 7.7），19 岁以下人口比例更高（44% vs 40%），家庭规模更大（6.4 vs 5.75），区中心比例更高（39% vs 33%），省中心比例更高（6.6% vs 1.6%）。

```py
df = df.rename(columns={"shhs"   : "household",
                        "merkezi": "district",
                        "merkezp": "province"})

control = ['vshr_islam1994', 'partycount', 'lpop1994',
           'ageshr19', 'household', 'district', 'province']
full = df.loc[:, control].groupby(df['i94']).agg([np.mean]).T
full.index = full.index.get_level_values(0)
full 
```

| i94 | 0.0 | 1.0 |
| --- | --- | --- |
| vshr_islam1994 | 0.1012 | 0.4145 |
| 党派数量 | 5.4907 | 5.8889 |
| 人口 1994 | 7.7745 | 8.3154 |
| ageshr19 | 0.3996 | 0.4453 |
| 家庭 | 5.7515 | 6.4449 |
| 区 | 0.3375 | 0.3937 |
| 省 | 0.0168 | 0.0667 |

使对照组和治疗组相似的一种方法是使用多元回归。对治疗变量'i94'的系数进行解释是*ceteris paribus*，也就是说，伊斯兰主要对高中学历的影响，考虑其他一切不变。这里的诀窍是“其他一切不变”，这意味着只有在回归中受控的因素。这是一个不完美的解决方案，因为在实际情况下，不可能控制影响结果变量的所有因素。然而，与简单回归相比，多元回归可能更少受到遗漏变量偏差的影响。

多元回归挑战了天真比较的结果。伊斯兰政权对高中毕业率的积极影响比世俗政权高出 1.4%。结果在 5%的显著水平上是统计学显著的。

```py
multiple = sm.OLS(df['hischshr1520f'],
                      df[['Intercept', 'i94'] + control],
                      missing='drop').fit()
print(multiple.summary().tables[1]) 
```

```py
==================================================================================
                     coef    std err          t      P>|t|      [0.025      0.975]
----------------------------------------------------------------------------------
Intercept          0.2626      0.015     17.634      0.000       0.233       0.292
i94                0.0139      0.006      2.355      0.019       0.002       0.026
vshr_islam1994    -0.0894      0.013     -6.886      0.000      -0.115      -0.064
partycount        -0.0038      0.001     -3.560      0.000      -0.006      -0.002
lpop1994           0.0159      0.002      7.514      0.000       0.012       0.020
ageshr19          -0.6125      0.021    -29.675      0.000      -0.653      -0.572
household          0.0057      0.001      8.223      0.000       0.004       0.007
district           0.0605      0.004     16.140      0.000       0.053       0.068
province           0.0357      0.010      3.499      0.000       0.016       0.056
================================================================================== 
```

多元回归的结果看起来有些反直觉。治疗变量的符号是如何改变的？

让我们从另一个角度来看数据。图表“天真比较”是所有市政的散点图。每个点代表一个市政。很难看出任何模式或趋势。

让我们绘制相同的图形，但根据高中毕业率的相似性将市政聚合在 29 个箱子中。这些箱子在下图中是蓝色的球。球的大小与用于计算高中毕业率均值的市政数量成比例。

如果你仔细观察截断点附近（垂直红线），当变量伊斯兰胜利边际= 0 时，你会看到高中毕业率水平的不连续或跳跃。

```py
# Library for Regression Discontinuity
!pip install rdd 
```

```py
Collecting rdd
  Using cached rdd-0.0.3.tar.gz (4.4 kB)
Requirement already satisfied: pandas in c:\anaconda\envs\textbook\lib\site-packages (from rdd) (1.1.3)
Requirement already satisfied: numpy in c:\anaconda\envs\textbook\lib\site-packages (from rdd) (1.19.2)
Requirement already satisfied: statsmodels in c:\anaconda\envs\textbook\lib\site-packages (from rdd) (0.12.0)
Requirement already satisfied: pytz>=2017.2 in c:\anaconda\envs\textbook\lib\site-packages (from pandas->rdd) (2020.1)
Requirement already satisfied: python-dateutil>=2.7.3 in c:\anaconda\envs\textbook\lib\site-packages (from pandas->rdd) (2.8.1)
Requirement already satisfied: patsy>=0.5 in c:\anaconda\envs\textbook\lib\site-packages (from statsmodels->rdd) (0.5.1)
Requirement already satisfied: scipy>=1.1 in c:\anaconda\envs\textbook\lib\site-packages (from statsmodels->rdd) (1.5.3)
Requirement already satisfied: six>=1.5 in c:\anaconda\envs\textbook\lib\site-packages (from python-dateutil>=2.7.3->pandas->rdd) (1.15.0)
Building wheels for collected packages: rdd
  Building wheel for rdd (setup.py): started
  Building wheel for rdd (setup.py): finished with status 'done'
  Created wheel for rdd: filename=rdd-0.0.3-py3-none-any.whl size=4723 sha256=e114b2f6b2da5c594f5856bd364a95d11da3dc2327c28b419668f2230f7c18c1
  Stored in directory: c:\users\vitor kamada\appdata\local\pip\cache\wheels\f0\b8\ed\f7a5bcaa0a1b5d89d33d70db90992fd816ac6cff666020255d
Successfully built rdd
Installing collected packages: rdd
Successfully installed rdd-0.0.3 
```

```py
from rdd import rdd

# Aggregate the data in 29 bins
threshold = 0
data_rdd = rdd.truncated_data(df, 'iwm94', 0.99, cut=threshold)
data_binned = rdd.bin_data(data_rdd, 'hischshr1520f', 'iwm94', 29)

# Labels
plt.title('Comparison using aggregate data (Bins)')
plt.xlabel('Islamic win margin')
plt.ylabel('Female aged 15-20 with high school')

# Scatterplot 
plt.scatter(data_binned['iwm94'], data_binned['hischshr1520f'],
    s = data_binned['n_obs'], facecolors='none', edgecolors='blue')

# Red Vertical Line
plt.axvline(x=0, color='red')

plt.show() 
```

![_images/3)_Are_Females_More_Likely_to_Complete_High_School_Under_Islamic_or_Secular_Regime_18_0.png](img/1a46724ba6019230e5e824d2cb16a0c0.png)

也许你并不相信存在不连续或跳跃的截断点。让我们用 10 个箱子绘制相同的图形，并限制变量伊斯兰胜利边际的带宽（范围）。与选择任意带宽（h）不同，让我们使用 Imbens＆Kalyanaraman（2012）开发的方法来获得最小化均方误差的最佳带宽。

最佳带宽($\hat{h}$)为 0.23，也就是说，让我们在截断上下方创建一个 0.23 的窗口来创建 10 个箱子。

```py
#  Optimal Bandwidth based on Imbens & Kalyanaraman (2012)
#  This bandwidth minimizes the mean squared error.
bandwidth_opt = rdd.optimal_bandwidth(df['hischshr1520f'],
                              df['iwm94'], cut=threshold)
bandwidth_opt 
```

```py
0.2398161605552802 
```

以下是 10 个箱子。控制组中有 5 个箱子，伊斯兰赢得的优势<0，处理组中有 5 个箱子，伊斯兰赢得的优势>0。请注意，高中毕业率在索引 4 和 5 之间跳跃，分别为 13.8%和 15.5%，即箱子 5 和 6。13.8%和 15.5%的值是基于分别 141 和 106 个市政府('n_obs')计算的。

```py
#  Aggregate the data in 10 bins using Optimal Bandwidth
data_rdd = rdd.truncated_data(df, 'iwm94', bandwidth_opt, cut=threshold)
data_binned = rdd.bin_data(data_rdd, 'hischshr1520f', 'iwm94', 10)
data_binned 
```

|  | 0 | hischshr1520f | iwm94 | n_obs |
| --- | --- | --- | --- | --- |
| 0 | 0.0 | 0.1769 | -0.2159 | 136.0 |
| 1 | 0.0 | 0.1602 | -0.1685 | 142.0 |
| 2 | 0.0 | 0.1696 | -0.1211 | 162.0 |
| 3 | 0.0 | 0.1288 | -0.0737 | 139.0 |
| 4 | 0.0 | 0.1381 | -0.0263 | 141.0 |
| 5 | 0.0 | 0.1554 | 0.0211 | 106.0 |
| 6 | 0.0 | 0.1395 | 0.0685 | 81.0 |
| 7 | 0.0 | 0.1437 | 0.1159 | 58.0 |
| 8 | 0.0 | 0.1408 | 0.1633 | 36.0 |
| 9 | 0.0 | 0.0997 | 0.2107 | 19.0 |

在图表“使用最佳带宽(h = 0.27)进行比较”中，蓝线适用于控制组(5 个箱子)，橙线适用于处理组(5 个箱子)。现在，不连续或跳跃是明显的。我们称这种方法为回归离散度(RD)。红色垂直线($\hat{\tau}_{rd}=3.5$% )是高中毕业率的增加。请注意，这种方法模拟了一个实验。伊斯兰党几乎赢得和几乎输掉的市政府很可能相互类似。直觉是“几乎赢得”和“几乎输掉”是像抛硬币一样的随机过程。选举的相反结果可能是随机发生的。另一方面，很难想象伊斯兰市长会在以 30%的更大优势赢得的市政府中输掉。

```py
# Scatterplot
plt.scatter(data_binned['iwm94'], data_binned['hischshr1520f'],
    s = data_binned['n_obs'], facecolors='none', edgecolors='blue')

# Labels
plt.title('Comparison using Optimum Bandwidth (h = 0.27)')
plt.xlabel('Islamic win margin')
plt.ylabel('Female aged 15-20 with high school')

# Regression
x = data_binned['iwm94']
y = data_binned['hischshr1520f']

c_slope , c_intercept = np.polyfit(x[0:5], y[0:5], 1)
plt.plot(x[0:6], c_slope*x[0:6] + c_intercept)

t_slope , t_intercept  = np.polyfit(x[5:10], y[5:10], 1)
plt.plot(x[4:10], t_slope*x[4:10] + t_intercept)

# Vertical Line
plt.vlines(0, 0, 0.2, colors='green', alpha =0.5)
plt.vlines(0, c_intercept, t_intercept, colors='red', linestyles='-')

# Plot Black Arrow
plt.axes().arrow(0, (t_intercept + c_intercept)/2, 
         dx = 0.15, dy =-0.06, head_width=0.02,
         head_length=0.01, fc='k', ec='k')

# RD effect
plt.text(0.1, 0.06, r'$\hat{\tau}_{rd}=3.5\%$', fontsize=16,
         bbox={'facecolor':'yellow', 'alpha':0.2})

plt.show() 
```

```py
C:\Anaconda\envs\textbook\lib\site-packages\ipykernel_launcher.py:25: MatplotlibDeprecationWarning: Adding an axes using the same arguments as a previous axes currently reuses the earlier instance.  In a future version, a new instance will always be created and returned.  Meanwhile, this warning can be suppressed, and the future behavior ensured, by passing a unique label to each axes instance. 
```

![_images/3)_Are_Females_More_Likely_to_Complete_High_School_Under_Islamic_or_Secular_Regime_24_1.png](img/84964dd3fee733d6f3ae7b3b75417e71.png)

```py
# RD effect given by the vertical red line
t_intercept - c_intercept 
```

```py
0.03584571077550233 
```

让我们将样本限制在伊斯兰市长以 5%的优势赢得或输掉的市政府。我们可以看到控制组和处理组之间的相似性比本章开头使用全样本进行比较更大。

然而，这种相似性并不接近于“完美实验”。部分原因是控制组和处理组的样本量较小。因此，当我们运行回归离散度时，建议添加控制变量。

```py
# bandwidth (h) = 5%
df5 = df[df['iwm94'] >= -0.05]
df5 = df5[df5['iwm94'] <= 0.05]

sample5 = df5.loc[:, control].groupby(df5['i94']).agg([np.mean]).T

sample5.index = full.index.get_level_values(0)
sample5 
```

| i94 | 0.0 | 1.0 |
| --- | --- | --- |
| vshr_islam1994 | 0.3026 | 0.3558 |
| partycount | 5.9730 | 5.8807 |
| lpop1994 | 8.2408 | 8.2791 |
| ageshr19 | 0.4415 | 0.4422 |
| household | 6.2888 | 6.4254 |
| district | 0.4595 | 0.4037 |
| province | 0.0338 | 0.0826 |

让我们正式阐述回归离散度的理论。

让$D_r$成为一个虚拟变量：如果分析单位接受了处理，则为 1，否则为 0。下标$r$表示处理($D_r$)是运行变量$r$的函数。

$$D_r = 1 \ 或 \ 0$$

在 Sharp 回归离散度中，处理($D_r$)由运行变量($r$)决定。

$$D_r = 1, \ 如果 \ r \geq r_0$$

$$D_r = 0, \ 如果 \ r < r_0$$

其中，$r_0$是一个任意的截断或阈值。

回归离散度的最基本规范是：

$$Y = \beta_0+\tau D_r+ \beta_1r+\epsilon$$

其中$Y$是结果变量，$\beta_0$是截距，$\tau$是处理变量($D_r$)的影响，$\beta_1$是运行变量($r$)的系数，$\epsilon$是误差项。

请注意，在实验中，处理是随机的，但在回归离散度中，处理完全由运行变量决定。随机过程的反面是确定性过程。这是反直觉的，但是当决定处理分配的规则(截断)是任意的时，确定性分配具有与随机化相同的效果。

一般来说，观察性研究的可信度非常低，因为遗漏变量偏差（OVB）的基本问题。误差项中的许多未观察因素可能与处理变量相关。因此，在回归框架中的一个大错误是将运行变量留在误差项中。

在所有估计器中，回归离散度可能是最接近黄金标准，随机实验的方法。主要缺点是回归离散度只捕捉局部平均处理效应（LATE）。将结果推广到带宽之外的实体是不合理的。

伊斯兰市长的影响是女性学校完成率高出 4％，使用带宽为 5％的回归离散度。这个结果在 5％的显著水平上是统计上显著的。

```py
#  Real RD specification
#  Meyersson (2014) doesn't use the interaction term, because 
# the results are unstable. In general the coefficient,
# of the interaction term is not statistically significant.
# df['i94_iwm94'] = df['i94']*df['iwm94']
# RD = ['Intercept', 'i94', 'iwm94', 'i94_iwm94']

RD = ['Intercept', 'i94', 'iwm94']

# bandwidth of 5%
df5 = df[df['iwm94'] >= -0.05]
df5 = df5[df5['iwm94'] <= 0.05]
rd5 = sm.OLS(df5['hischshr1520f'],
                      df5[RD + control],
                      missing='drop').fit()
print(rd5.summary()) 
```

```py
 OLS Regression Results                            
==============================================================================
Dep. Variable:          hischshr1520f   R-squared:                       0.570
Model:                            OLS   Adj. R-squared:                  0.554
Method:                 Least Squares   F-statistic:                     36.32
Date:                Wed, 28 Oct 2020   Prob (F-statistic):           1.67e-40
Time:                        17:41:01   Log-Likelihood:                 353.09
No. Observations:                 257   AIC:                            -686.2
Df Residuals:                     247   BIC:                            -650.7
Df Model:                           9                                         
Covariance Type:            nonrobust                                         
==================================================================================
                     coef    std err          t      P>|t|      [0.025      0.975]
----------------------------------------------------------------------------------
Intercept          0.3179      0.043      7.314      0.000       0.232       0.403
i94                0.0399      0.016      2.540      0.012       0.009       0.071
iwm94             -0.4059      0.284     -1.427      0.155      -0.966       0.154
vshr_islam1994    -0.0502      0.060     -0.842      0.401      -0.168       0.067
partycount        -0.0003      0.003     -0.074      0.941      -0.007       0.007
lpop1994           0.0091      0.005      1.718      0.087      -0.001       0.020
ageshr19          -0.7383      0.065    -11.416      0.000      -0.866      -0.611
household          0.0075      0.002      3.716      0.000       0.004       0.011
district           0.0642      0.010      6.164      0.000       0.044       0.085
province           0.0191      0.019      1.004      0.316      -0.018       0.057
==============================================================================
Omnibus:                       17.670   Durbin-Watson:                   1.658
Prob(Omnibus):                  0.000   Jarque-Bera (JB):               19.403
Skew:                           0.615   Prob(JB):                     6.12e-05
Kurtosis:                       3.546   Cond. No.                         898.
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified. 
```

伊斯兰市长的影响是女性学校完成率高出 2.1％，使用回归离散度，最佳带宽为 0.27，根据 Imbens＆Kalyanaraman（2012）计算。这个结果在 1％的显著水平上是统计上显著的。

因此，回归离散度估计器表明，天真的比较在错误的方向上存在偏见。

```py
# bandwidth_opt is 0.2715
df27 = df[df['iwm94'] >= -bandwidth_opt]
df27 = df27[df27['iwm94'] <= bandwidth_opt]
rd27 = sm.OLS(df27['hischshr1520f'],
                      df27[RD + control],
                      missing='drop').fit()
print(rd27.summary()) 
```

```py
 OLS Regression Results                            
==============================================================================
Dep. Variable:          hischshr1520f   R-squared:                       0.534
Model:                            OLS   Adj. R-squared:                  0.530
Method:                 Least Squares   F-statistic:                     128.8
Date:                Wed, 28 Oct 2020   Prob (F-statistic):          5.95e-161
Time:                        17:41:01   Log-Likelihood:                 1349.9
No. Observations:                1020   AIC:                            -2680.
Df Residuals:                    1010   BIC:                            -2630.
Df Model:                           9                                         
Covariance Type:            nonrobust                                         
==================================================================================
                     coef    std err          t      P>|t|      [0.025      0.975]
----------------------------------------------------------------------------------
Intercept          0.2943      0.023     12.910      0.000       0.250       0.339
i94                0.0214      0.008      2.775      0.006       0.006       0.036
iwm94             -0.0343      0.038     -0.899      0.369      -0.109       0.041
vshr_islam1994    -0.0961      0.030     -3.219      0.001      -0.155      -0.038
partycount        -0.0026      0.002     -1.543      0.123      -0.006       0.001
lpop1994           0.0135      0.003      4.719      0.000       0.008       0.019
ageshr19          -0.6761      0.032    -20.949      0.000      -0.739      -0.613
household          0.0072      0.001      6.132      0.000       0.005       0.010
district           0.0575      0.006     10.364      0.000       0.047       0.068
province           0.0390      0.010      3.788      0.000       0.019       0.059
==============================================================================
Omnibus:                      179.124   Durbin-Watson:                   1.610
Prob(Omnibus):                  0.000   Jarque-Bera (JB):              373.491
Skew:                           1.001   Prob(JB):                     7.90e-82
Kurtosis:                       5.186   Cond. No.                         270.
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified. 
```

## 练习

1）使用 Meyersson（2014）的数据运行回归离散度：a）使用完整样本，b）另一个带宽为 0.1（两侧均为 10％）。使用本章两个示例的相同规范。解释处理变量的系数。结果“a”或“b”更可信？请解释。

2）下面是变量伊斯兰赢得的边际的直方图。您是否看到截断或异常模式，其中截断= 0？在调查奔跑变量的截断周围是否发生了奇怪的事情的合理性是什么？

```py
import plotly.express as px
fig = px.histogram(df, x="iwm94")
fig.update_layout(shapes=[
    dict(
      type= 'line',
      yref= 'paper', y0 = 0, y1 = 1,
      xref= 'x', x0 = 0, x1 = 0)])
fig.show() 
```

3）我修改了“伊斯兰赢得的边际”变量以教育为目的。假设这是 Meyersson（2014）的真实运行变量。请参见下面的直方图。在这种假设情况下，您能从土耳其的选举中推断出什么？在这种情况下运行回归离散度存在问题吗？如果有，您可以采取什么措施来解决问题？

```py
def corrupt(variable):
    if variable <= 0 and variable >= -.025:
        return 0.025
    else:   
        return variable

df['running'] = df["iwm94"].apply(corrupt)

fig = px.histogram(df, x="running")
fig.update_layout(shapes=[
    dict(
      type= 'line',
      yref= 'paper', y0 = 0, y1 = 1,
      xref= 'x', x0 = 0, x1 = 0)])
fig.show() 
```

4）为不熟悉因果推断的机器学习专家解释下面的图形？变量“伊斯兰选票份额”可以用作运行变量吗？推测。

```py
def category(var):
    if var <= 0.05 and var >= -.05:
        return "5%"
    else:   
        return "rest"

df['margin'] = df["iwm94"].apply(category)

fig = px.scatter(df, x="vshr_islam1994", y="iwm94", color ="margin",
                 labels={"iwm94": "Islamic win margin",
                         "vshr_islam1994": "Islamic vote share"})
fig.update_layout(shapes=[
    dict(
      type= 'line',
      yref= 'paper', y0 = 1/2, y1 = 1/2,
      xref= 'x', x0 = 0, x1 = 1)])
fig.show() 
```

5）在伊斯兰或世俗政权下，男性更有可能完成高中吗？根据数据和严格的分析来证明您的答案。变量“hischshr1520m”是 15-20 岁男性中高中教育的比例。

## 参考

Imbens，G.，＆Kalyanaraman，K.（2012）。回归离散度估计器的最佳带宽选择。经济研究评论，79（3），933-959。

Meyersson，Erik。 （2014）。[伊斯兰统治与穷人和虔诚者的赋权](https://github.com/causal-methods/Papers/raw/master/Islamic%20Rule%20and%20the%20Empowerment%20of%20the%20Poor%20and%20Pious.pdf)。经济计量学，82（1），229-269。
