# 5）联邦储备是否能够阻止大萧条？

> 原文：[`causal-methods.github.io/Book/5%29_Could_the_Federal_Reserve_Prevent_the_Great_Depression.html`](https://causal-methods.github.io/Book/5%29_Could_the_Federal_Reserve_Prevent_the_Great_Depression.html)

[Vitor Kamada](https://www.linkedin.com/in/vitor-kamada-1b73a078)

电子邮件：econometrics.methods@gmail.com

最后更新日期：2020 年 10 月 12 日

关于大萧条，新古典经济学家认为经济活动的下降“导致”了银行倒闭。凯恩斯（1936）则相反地认为，银行破产导致了企业倒闭。

Richardson & Troost（2009）注意到，在大萧条期间，密西西比州被联邦储备（Fed）的不同分支控制，分为圣路易斯和亚特兰大两个地区。

与圣路易斯 Fed 不同，亚特兰大 Fed 采取了凯恩斯主义的贴现贷款和向流动性不足的银行提供紧急流动性的政策，使得借款更加困难。

让我们打开 Ziebarth（2013）的数据。每一行都是 1929 年、1931 年、1933 年和 1935 年的人口普查制造业（CoM）中的一家公司。

```py
# Load data from Ziebarth (2013)
import numpy as np
import pandas as pd
path = "https://github.com/causal-methods/Data/raw/master/" 
data = pd.read_stata(path + "MS_data_all_years_regs.dta")
data.head() 
```

|  | 县 | 平均工资收入人数 | 平均工资 a | 平均工资 b | 平均工资 d | 开始 | 电力容量编号 | 汽油容量编号 | 马车容量编号 | 人口普查年份 | ... | 1933 年未进入 | 1931 年未退出 | 1935 年未进入 | 1933 年未退出 | 1931 年平衡 | 1933 年平衡 | 1935 年平衡 | 产品数量 | rrtsap | delta_indic |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 |  | NaN | NaN | NaN | NaN |  |  |  |  | 1933 | ... | NaN | NaN | NaN | NaN | 0.0 | 0.0 | 0.0 | 0.0 | NaN | 0.0 |
| 1 | Greene | 12.000000 | 0.706404 | 0.254306 | NaN | 1933 年 1 月 1 日 |  |  |  | 1933 | ... | NaN | NaN | NaN | NaN | 0.0 | 0.0 | 0.0 | 2.0 | 100.797699 | 0.0 |
| 2 | Hinds | 4.000000 | 0.242670 | 0.215411 | NaN | 1933 年 4 月 1 日 |  |  |  | 1933 | ... | NaN | NaN | NaN | NaN | 0.0 | 0.0 | 0.0 | 1.0 | 404.526093 | 0.0 |
| 3 | Wayne | 36.000000 | 0.064300 | 0.099206 | NaN | 1933 年 1 月 1 日 |  |  |  | 1933 | ... | NaN | NaN | NaN | NaN | 0.0 | 0.0 | 0.0 | 3.0 | 104.497620 | 0.0 |
| 4 | Attala | 6.333333 | NaN | 0.437281 | NaN | 1933 年 1 月 1 日 |  |  |  | 1933 | ... | NaN | NaN | NaN | NaN | 0.0 | 0.0 | 0.0 | 1.0 | 148.165237 | 0.0 |

5 行×538 列

首先，我们必须检查 1929 年圣路易斯和亚特兰大两个地区有多相似。所有变量都以对数形式报告。圣路易斯地区公司的平均收入为 10.88；而亚特兰大地区公司的平均收入为 10.78。圣路易斯和亚特兰大的工资收入（4.54 比 4.69）和每个工人的工作时间（4.07 比 4）也相似。

```py
# Round 2 decimals
pd.set_option('precision', 2)

# Restrict the sample to the year: 1929
df1929 = data[data.censusyear.isin([1929])]

vars= ['log_total_output_value', 'log_wage_earners_total',
       'log_hours_per_wage_earner']

df1929.loc[:, vars].groupby(df1929["st_louis_fed"]).agg([np.size, np.mean]) 
```

|  | log_total_output_value | log_wage_earners_total | log_hours_per_wage_earner |
| --- | --- | --- | --- |
|  | 大小 | 平均 | 大小 | 平均 | 大小 | 平均 |
| --- | --- | --- | --- | --- | --- | --- |
| 圣路易斯 Fed |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- |
| 0.0 | 424.0 | 10.78 | 424.0 | 4.69 | 424.0 | 4.00 |
| 1.0 | 367.0 | 10.88 | 367.0 | 4.54 | 367.0 | 4.07 |

此外，圣路易斯和亚特兰大的平均价格（1.72 比 1.55）和平均数量（8.63 比 8.83）也相似，如果样本限制在只有 1 种产品的公司。因此，亚特兰大地区是圣路易斯地区的一个合理对照组。

```py
# Restrict sample to firms with 1 product
df1929_1 = df1929[df1929.num_products.isin([1])]

per_unit = ['log_output_price_1', 'log_output_quantity_1']

df1929_1.loc[:, per_unit].groupby(df1929_1["st_louis_fed"]).agg([np.size, np.mean]) 
```

|  | log_output_price_1 | log_output_quantity_1 |
| --- | --- | --- |
|  | 大小 | 平均 | 大小 | 平均 |
| --- | --- | --- | --- | --- |
| 圣路易斯 Fed |  |  |  |  |
| --- | --- | --- | --- | --- |
| 0.0 | 221.0 | 1.55 | 221.0 | 8.83 |
| 1.0 | 225.0 | 1.72 | 225.0 | 8.63 |

我们想要看看圣路易斯 Fed 的信贷约束政策是否减少了公司的收入。换句话说，亚特兰大 Fed 是否拯救了濒临破产的公司。

为此，我们必须探索时间维度：比较 1929 年之前和 1931 年之后的公司收入。

让我们将样本限制在 1929 年和 1931 年。然后，让我们删除缺失值。

```py
# Restrict the sample to the years: 1929 and 1931
df = data[data.censusyear.isin([1929, 1931])]

vars = ['firmid', 'censusyear', 'log_total_output_value',
        'st_louis_fed', 'industrycode', 'year_1931']

# Drop missing values 
df = df.dropna(subset=vars) 
```

现在，我们可以声明一个面板数据结构，即设置分析单位和时间维度。注意表中的变量“firmid”和“censusyear”成为了索引。顺序很重要。第一个变量必须是分析单位，第二个变量必须是时间单位。在表中看到，公司（id = 12）在 1929 年和 1931 年观察到。

注意，在清理数据集后声明了面板数据结构。例如，如果在声明面板数据后删除了缺失值，进行回归的命令可能会返回错误。

```py
df = df.set_index(['firmid', 'censusyear'])
df.head() 
```

|  |  | 县 | 平均就业人数 | 平均工资 a | 平均工资 b | 平均工资 d | 开始 | 电力容量编号 | 汽油容量编号 | 马车容量编号 | 城镇或村庄 | ... | 1933 年不进入 | 1931 年不退出 | 1935 年不进入 | 1933 年不退出 | 1931 年平衡 | 1933 年平衡 | 1935 年平衡 | 产品数量 | rrtsap | delta_indic |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| firmid | censusyear |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 12 | 1929 | Oktibbeha | 1.75 | 0.38 | 0.50 | NaN | January 1, 1929 |  |  |  | A & M College | ... | NaN | NaN | NaN | NaN | 1.0 | 0.0 | 1.0 | 1.0 | 328.31 | 0.0 |
| 1931 | Oktibbeha | 1.75 | 0.25 | 0.30 | NaN | January 1, 1931 |  |  |  | A and M College | ... | NaN | 1.0 | NaN | NaN | 1.0 | 0.0 | 0.0 | 1.0 | NaN | 0.0 |
| 13 | 1929 | Warren | 7.25 | 0.49 | 0.58 | NaN | January 1, 1929 |  |  |  | Clarksburg | ... | NaN | NaN | NaN | NaN | 1.0 | 0.0 | 1.0 | 2.0 | 670.07 | 1.0 |
| 1931 | Warren | 3.50 | NaN | 0.71 | NaN | January 1, 1931 |  |  |  | Vicksburg, | ... | NaN | 1.0 | NaN | NaN | 1.0 | 0.0 | 0.0 | 2.0 | NaN | 1.0 |
| 14 | 1929 | Monroe | 12.92 | 0.17 | 0.23 | NaN | January 1, 1929 |  |  |  | Aberdeen | ... | NaN | NaN | NaN | NaN | 1.0 | 0.0 | 1.0 | 2.0 | 314.90 | 0.0 |

5 行×536 列

让我们解释面板数据相对于横截面数据的优势。后者是某一时间点或时期的快照；而在面板数据中，同一分析单位随时间观察。

让\(Y_{it}\)表示单位\(i\)在时间\(t\)的结果变量。虚拟变量\(d2_{it}\)在第二个时期为 1，在第一个时期为 0。注意解释变量\(X_{it}\)在单位\(i\)和时间\(t\)上变化，但未观察到的因素\(\alpha_i\)在时间上不变。未观察到的因素是一个不可用的变量（数据），可能与感兴趣的变量相关，导致结果偏差。

\[Y_{it}=\beta_0+\delta_0d2_{it}+\beta_1X_{it}+\alpha_i+\epsilon_{it}\]

探索时间变化的优势在于，未观察到的因素\(\alpha_i\)可以通过一阶差分（FD）方法消除。

在第二个时期（\(t=2\)），时间虚拟\(d2=1\)：

\[Y_{i2}=\beta_0+\delta_0+\beta_1X_{i2}+\alpha_i+\epsilon_{i2}\]

在第一个时期（\(t=1\)），时间虚拟\(d2=0\)：

\[Y_{i1}=\beta_0+\beta_1X_{i1}+\alpha_i+\epsilon_{i1}\]

然后：

\[Y_{i2}-Y_{i1}\]\[=\delta_0+\beta_1(X_{i2}-X_{i1})+\epsilon_{i2}-\epsilon_{i1}\]\[\Delta Y_i=\delta_0+\beta_1\Delta X_i+\Delta \epsilon_i\]

因此，如果观察到相同的单位随时间变化（面板数据），就不需要担心一个可以被认为在分析的时间内保持不变的因素。我们可以假设公司文化和制度实践在短时间内变化不大。这些因素可能解释了公司之间收入的差异，但如果上述假设是正确的，它们不会对结果产生偏见。

让我们安装可以运行面板数据回归的库。

```py
!pip install linearmodels 
```

```py
Requirement already satisfied: linearmodels in c:\anaconda\envs\textbook\lib\site-packages (4.17)
Requirement already satisfied: numpy>=1.15 in c:\anaconda\envs\textbook\lib\site-packages (from linearmodels) (1.19.2) 
```

```py
WARNING: No metadata found in c:\anaconda\envs\textbook\lib\site-packages
ERROR: Could not install packages due to an EnvironmentError: [Errno 2] No such file or directory: 'c:\\anaconda\\envs\\textbook\\lib\\site-packages\\numpy-1.19.2.dist-info\\METADATA' 
```

让我们使用双重差分（DID）方法来估计圣路易斯联邦储备银行政策对公司收入的影响。除了探索时间差异外，还必须使用治疗-对照差异来估计政策的因果影响。

让\(Y\)成为结果变量'log_total_output_value'，\(d2\)成为时间虚拟变量'year_1931'，\(dT\)成为治疗虚拟变量'st_louis_fed'，\(d2 \cdot dT\)成为前两个虚拟变量之间的交互项：

\[Y = \beta_0+\delta_0d2+\beta_1dT+\delta_1 (d2\cdot dT)+ \epsilon\]

DID 估计量由\(\delta_1\)给出，而不是由\(\beta_1\)或\(\delta_0\)给出。首先，我们计算“治疗组（圣路易斯）”和“对照组（亚特兰大）”之间的差异，然后计算“之后（1931 年）”和“之前（1921 年）”之间的差异。

\[\hat{\delta}_1 = (\bar{y}_{2,T}-\bar{y}_{2,C})-(\bar{y}_{1,T}-\bar{y}_{1,C})\]

顺序无关紧要。如果我们首先计算“之后（1931 年）”和“之前（1921 年）”之间的差异，然后计算“治疗组（圣路易斯）”和“对照组（亚特兰大）”之间的差异，结果将是相同的\(\delta_1\)。

\[\hat{\delta}_1 = (\bar{y}_{2,T}-\bar{y}_{1,T})-(\bar{y}_{2,C}-\bar{y}_{1,C})\]

让我们正式地展示，我们必须在 DID 估计量\(\delta_0\)中两次取差异：

如果\(d2=0\)且\(dT=0\)，那么\(Y_{0,0}=\beta_0\)。

如果\(d2=1\)且\(dT=0\)，那么\(Y_{1,0}=\beta_0+\delta_0\)。

对于对照组，“之后-之前”的差异是：

\[Y_{1,0}-Y_{0,0}=\delta_0\]

让我们对治疗组应用相同的推理：

如果\(d2=0\)且\(dT=1\)，那么\(Y_{0,1}=\beta_0 + \beta_1\)。

如果\(d2=1\)且\(dT=1\)，那么\(Y_{1,1}=\beta_0+\delta_0+ \beta_1+\delta_1\)。

对于治疗组，“之后-之前”的差异是：

\[Y_{1,1}-Y_{0,1}=\delta_0+\delta_1\]

然后，如果我们计算“治疗组-对照组”的差异，我们得到：

\[\delta_1\]

![alt text](img/caa3e0ce33ed3354e73d31f5c81c7997.png)

让我们手动计算“大萧条”期间公司收入图表中的\(\hat{\delta}_1\)。请注意，在双重差分（DID）方法中，根据对照组（亚特兰大）构建了一个反事实。这只是亚特兰大线的平行移位。反事实是治疗组（圣路易斯）的假设结果，如果圣路易斯联邦储备银行遵循了亚特兰大联邦储备银行相同的政策。

\[\hat{\delta}_1 = (\bar{y}_{2,T}-\bar{y}_{2,C})-(\bar{y}_{1,T}-\bar{y}_{1,C})\]\[=(10.32-10.42)-(10.87-10.78)\]\[=(-0.1)-(-0.1)\]\[=-0.2\]

圣路易斯联邦储备银行的严格信贷政策使公司的收入减少了约 20%。1931 年底的简单均值比较结果仅为-10%。因此，如果不使用反事实推理，圣路易斯联邦储备银行政策的负面影响将被大大低估。

```py
# Mean Revenue for the Graphic
table = pd.crosstab(df['year_1931'], df['st_louis_fed'], 
        values=df['log_total_output_value'], aggfunc='mean')

# Build Graphic
import plotly.graph_objects as go
fig = go.Figure()

# x axis
year = [1929, 1931]

# Atlanta Line
fig.add_trace(go.Scatter(x=year, y=table[0],
                         name='Atlanta (Control)'))
# St. Louis Line
fig.add_trace(go.Scatter(x=year, y=table[1],
                         name='St. Louis (Treatment)'))
# Counterfactual
end_point = (table[1][0] - table[0][0]) + table[0][1]
counter = [table[1][0], end_point]
fig.add_trace(go.Scatter(x=year, y= counter,
                         name='Counterfactual',
                         line=dict(dash='dot') ))

# Difference-in-Differences (DID) estimation
fig.add_trace(go.Scatter(x=[1931, 1931],
                         y=[table[1][1], end_point],
                         name='$\delta_1=0.2$',
                         line=dict(dash='dashdot') ))

# Labels
fig.update_layout(title="Firm's Revenue during the Great Depression",
                  xaxis_type='category',
                  xaxis_title='Year',
                  yaxis_title='Log(Revenue)')

fig.show() 
```

通过回归实施的双重差分（DID）的结果是：

\[\hat{Y} = 10.8-0.35d2+0.095dT-0.20(d2\cdot dT)\]

```py
from linearmodels import PanelOLS

Y = df['log_total_output_value']
df['const'] = 1
df['louis_1931'] = df['st_louis_fed']*df['year_1931']

## Difference-in-Differences (DID) specification
dd = ['const', 'st_louis_fed', 'year_1931', 'louis_1931']

dif_in_dif = PanelOLS(Y, df[dd]).fit(cov_type='clustered',
                                     cluster_entity=True)
print(dif_in_dif) 
```

```py
C:\Anaconda\envs\textbook\lib\site-packages\statsmodels\tools\_testing.py:19: FutureWarning:

pandas.util.testing is deprecated. Use the functions in the public API at pandas.testing instead. 
```

```py
 PanelOLS Estimation Summary                             
====================================================================================
Dep. Variable:     log_total_output_value   R-squared:                        0.0257
Estimator:                       PanelOLS   R-squared (Between):              0.0145
No. Observations:                    1227   R-squared (Within):               0.2381
Date:                    Fri, Nov 06 2020   R-squared (Overall):              0.0257
Time:                            21:12:12   Log-likelihood                   -2135.5
Cov. Estimator:                 Clustered                                           
                                            F-statistic:                      10.761
Entities:                             938   P-value                           0.0000
Avg Obs:                           1.3081   Distribution:                  F(3,1223)
Min Obs:                           1.0000                                           
Max Obs:                           2.0000   F-statistic (robust):             18.780
                                            P-value                           0.0000
Time periods:                           2   Distribution:                  F(3,1223)
Avg Obs:                           613.50                                           
Min Obs:                           575.00                                           
Max Obs:                           652.00                                           

                              Parameter Estimates                               
================================================================================
              Parameter  Std. Err.     T-stat    P-value    Lower CI    Upper CI
--------------------------------------------------------------------------------
const            10.781     0.0723     149.13     0.0000      10.639      10.923
st_louis_fed     0.0945     0.1043     0.9057     0.3653     -0.1102      0.2991
year_1931       -0.3521     0.0853    -4.1285     0.0000     -0.5194     -0.1848
louis_1931      -0.1994     0.1237    -1.6112     0.1074     -0.4422      0.0434
================================================================================ 
```

```py
C:\Anaconda\envs\textbook\lib\site-packages\linearmodels\panel\data.py:98: FutureWarning:

is_categorical is deprecated and will be removed in a future version.  Use is_categorical_dtype instead 
```

圣路易斯联邦储备银行的政策使公司收入减少了 18%（\(1-e^{-0.1994}\)）。然而，p 值为 0.1074。结果在 10%的显著性水平下并不具有统计学意义。

```py
from math import exp
1 - exp(dif_in_dif.params.louis_1931 ) 
```

```py
0.18076309464004925 
```

有人可能会认为公司之间的差异是一个混杂因素。一个或另一个大公司可能会使结果产生偏差。

这个问题可以通过使用固定效应（FE）或 Within Estimator 来解决。该技术类似于 First-Difference（FD），但数据转换不同。时间去均值过程用于消除未观察到的因素\(\alpha_i\)。

\[Y_{it}=\beta X_{it}+\alpha_i+\epsilon_{it}\]

然后，我们对每个\(i\)在时间\(t\)上的变量进行平均：

\[\bar{Y}_{i}=\beta \bar{X}_{i}+\alpha_i+\bar{\epsilon}_{i}\]

然后，我们取差异，未观察到的因素\(\alpha_i\)消失：

\[Y_{it}-\bar{Y}_{i}=\beta (X_{it}-\bar{X}_{i})+\epsilon_{it}-\bar{\epsilon}_{i}\]

我们可以用更简洁的方式写出上面的方程：

\[\ddot{Y}_{it}=\beta \ddot{X}_{it}+\ddot{\epsilon}_{it}\]

正如我们之前宣布的，公司是这个面板数据集中的分析单位，计算机会自动使用命令“entity_effects=True”实现公司固定效应（FE）。

我们在双重差分（DID）规范中添加了公司固定效应（FE），结果并没有改变太多。直觉是双重差分（DID）技术已经减轻了内生性问题。

圣路易斯联邦政策使公司收入减少了 17%（\(1-e^{-0.1862}\)）。结果在 10%的显著水平上是统计上显著的。

```py
firmFE = PanelOLS(Y, df[dd], entity_effects=True)
print(firmFE.fit(cov_type='clustered', cluster_entity=True)) 
```

```py
 PanelOLS Estimation Summary                             
====================================================================================
Dep. Variable:     log_total_output_value   R-squared:                        0.2649
Estimator:                       PanelOLS   R-squared (Between):             -0.4554
No. Observations:                    1227   R-squared (Within):               0.2649
Date:                    Fri, Nov 06 2020   R-squared (Overall):             -0.4266
Time:                            21:12:12   Log-likelihood                   -202.61
Cov. Estimator:                 Clustered                                           
                                            F-statistic:                      34.361
Entities:                             938   P-value                           0.0000
Avg Obs:                           1.3081   Distribution:                   F(3,286)
Min Obs:                           1.0000                                           
Max Obs:                           2.0000   F-statistic (robust):             31.245
                                            P-value                           0.0000
Time periods:                           2   Distribution:                   F(3,286)
Avg Obs:                           613.50                                           
Min Obs:                           575.00                                           
Max Obs:                           652.00                                           

                              Parameter Estimates                               
================================================================================
              Parameter  Std. Err.     T-stat    P-value    Lower CI    Upper CI
--------------------------------------------------------------------------------
const            9.9656     0.4511     22.091     0.0000      9.0777      10.854
st_louis_fed     1.9842     1.0365     1.9144     0.0566     -0.0559      4.0243
year_1931       -0.3666     0.0657    -5.5843     0.0000     -0.4959     -0.2374
louis_1931      -0.1862     0.0982    -1.8965     0.0589     -0.3794      0.0071
================================================================================

F-test for Poolability: 6.8220
P-value: 0.0000
Distribution: F(937,286)

Included effects: Entity 
```

固定效应（FE）可以通过添加虚拟变量手动实现。有不同的固定效应。让我们在双重差分（DID）规范中添加行业固定效应，以排除结果可能受行业特定冲击驱动的可能性。

圣路易斯联邦政策使公司收入减少了 14.2%（\(1-e^{-0.1533}\)）。结果在 10%的显著水平上是统计上显著的。

为什么不同时添加公司和行业固定效应？这是可能的，也是可取的，但由于多重共线性问题，计算机不会返回任何结果。我们每家公司只有两个观察值（2 年）。如果我们为每家公司添加一个虚拟变量，就像运行一个变量比观察值更多的回归。

在他的论文中，Ziebarth（2013）使用了公司和行业固定效应，这是如何可能的？Ziebarth（2013）使用了 Stata 软件。在多重共线性问题的情况下，Stata 会自动删除一些变量并输出结果。尽管这种做法在经济学顶级期刊中很常见，但这并不是“真正”的固定效应。

```py
industryFE = PanelOLS(Y, df[dd + ['industrycode']])
print(industryFE.fit(cov_type='clustered', cluster_entity=True)) 
```

```py
 PanelOLS Estimation Summary                             
====================================================================================
Dep. Variable:     log_total_output_value   R-squared:                        0.5498
Estimator:                       PanelOLS   R-squared (Between):              0.5462
No. Observations:                    1227   R-squared (Within):               0.3913
Date:                    Fri, Nov 06 2020   R-squared (Overall):              0.5498
Time:                            21:12:12   Log-likelihood                   -1661.9
Cov. Estimator:                 Clustered                                           
                                            F-statistic:                      29.971
Entities:                             938   P-value                           0.0000
Avg Obs:                           1.3081   Distribution:                 F(48,1178)
Min Obs:                           1.0000                                           
Max Obs:                           2.0000   F-statistic (robust):          4.791e+15
                                            P-value                           0.0000
Time periods:                           2   Distribution:                 F(48,1178)
Avg Obs:                           613.50                                           
Min Obs:                           575.00                                           
Max Obs:                           652.00                                           

                                 Parameter Estimates                                 
=====================================================================================
                   Parameter  Std. Err.     T-stat    P-value    Lower CI    Upper CI
-------------------------------------------------------------------------------------
const                 10.391     0.2249     46.201     0.0000      9.9497      10.832
st_louis_fed         -0.1740     0.0805    -2.1610     0.0309     -0.3319     -0.0160
year_1931            -0.3163     0.0660    -4.7922     0.0000     -0.4458     -0.1868
louis_1931           -0.1533     0.0916    -1.6741     0.0944     -0.3331      0.0264
industrycode.1005    -0.1966     0.3637    -0.5407     0.5888     -0.9101      0.5169
industrycode.101      0.2349     0.2403     0.9775     0.3285     -0.2366      0.7064
industrycode.1014    -0.1667     1.0812    -0.1542     0.8775     -2.2880      1.9546
industrycode.103      1.5898     0.2938     5.4104     0.0000      1.0133      2.1663
industrycode.104      1.1349     0.2818     4.0280     0.0001      0.5821      1.6877
industrycode.105      0.9660     0.4191     2.3049     0.0213      0.1437      1.7882
industrycode.107      0.9374     0.3073     3.0502     0.0023      0.3344      1.5404
industrycode.110      0.2299     0.3447     0.6671     0.5049     -0.4463      0.9062
industrycode.111      2.8271     0.4097     6.9010     0.0000      2.0233      3.6308
industrycode.112      0.2676     0.4185     0.6394     0.5227     -0.5535      1.0888
industrycode.114      2.7559     0.4623     5.9612     0.0000      1.8489      3.6630
industrycode.116      0.4498     0.4351     1.0337     0.3015     -0.4039      1.3035
industrycode.117     -0.1337     0.5436    -0.2461     0.8057     -1.2002      0.9327
industrycode.118      0.1025     0.2483     0.4127     0.6799     -0.3847      0.5896
industrycode.119     -0.3121     0.2336    -1.3360     0.1818     -0.7705      0.1462
industrycode.1204    -0.0102     0.3031    -0.0338     0.9731     -0.6048      0.5844
industrycode.123      0.2379     0.7093     0.3354     0.7374     -1.1537      1.6295
industrycode.126      1.8847     0.4828     3.9039     0.0001      0.9375      2.8319
industrycode.128     -0.0025     0.5277    -0.0047     0.9963     -1.0377      1.0328
industrycode.1303     2.6410     0.2293     11.519     0.0000      2.1912      3.0908
industrycode.136      0.2537     0.2493     1.0178     0.3090     -0.2354      0.7428
industrycode.1410    -1.4091     0.2967    -4.7496     0.0000     -1.9912     -0.8270
industrycode.1502     1.4523     0.3858     3.7644     0.0002      0.6954      2.2093
industrycode.1604    -0.5652     0.4195    -1.3473     0.1781     -1.3882      0.2579
industrycode.1624    -1.0476     1.0108    -1.0365     0.3002     -3.0307      0.9355
industrycode.1640    -0.8320     0.3613    -2.3030     0.0215     -1.5409     -0.1232
industrycode.1676     0.5830     0.2512     2.3205     0.0205      0.0901      1.0759
industrycode.1677    -0.7025     0.2346    -2.9947     0.0028     -1.1627     -0.2422
industrycode.203     -0.6435     0.2241    -2.8712     0.0042     -1.0832     -0.2038
industrycode.210B     1.7535     0.2293     7.6482     0.0000      1.3037      2.2033
industrycode.216      2.7641     0.3549     7.7888     0.0000      2.0679      3.4604
industrycode.234a     1.6871     0.5410     3.1187     0.0019      0.6258      2.7485
industrycode.265      1.2065     0.4193     2.8774     0.0041      0.3838      2.0292
industrycode.266     -0.3244     0.2736    -1.1857     0.2360     -0.8612      0.2124
industrycode.304      1.2156     0.4982     2.4400     0.0148      0.2382      2.1930
industrycode.314      0.8008     0.2411     3.3211     0.0009      0.3277      1.2740
industrycode.317     -0.1470     0.2750    -0.5347     0.5930     -0.6866      0.3925
industrycode.515     -0.4623     0.2862    -1.6153     0.1065     -1.0238      0.0992
industrycode.518     -0.5243     0.2887    -1.8157     0.0697     -1.0908      0.0422
industrycode.520     -0.2234     0.2731    -0.8179     0.4136     -0.7592      0.3124
industrycode.614      1.8029     0.3945     4.5707     0.0000      1.0290      2.5768
industrycode.622      2.9239     0.2430     12.035     0.0000      2.4473      3.4006
industrycode.633      0.0122     1.5690     0.0078     0.9938     -3.0661      3.0906
industrycode.651      1.9170     0.8690     2.2060     0.0276      0.2121      3.6220
industrycode.652      2.5495     0.2211     11.531     0.0000      2.1158      2.9833
===================================================================================== 
```

仅仅出于练习目的，假设未观察到的因素\(\alpha_i\)被忽略。这个假设被称为随机效应（RE）。在这种情况下，\(\alpha_i\)将包含在误差项\(v_{it}\)中，并可能使结果产生偏见。

\[Y_{it}=\beta X_{it}+v_{it}\]\[v_{it}= \alpha_i+\epsilon_{it}\]

在一个实验中，处理变量与未观察到的因素\(\alpha_i\)不相关。在这种情况下，随机效应（RE）模型具有产生比固定效应模型更低标准误差的优势。

请注意，如果我们运行简单的随机效应（RE）回归，我们可能错误地得出结论，即圣路易斯联邦政策使公司收入增加了 7%。

```py
from linearmodels import RandomEffects
re = RandomEffects(Y, df[['const', 'st_louis_fed']])
print(re.fit(cov_type='clustered', cluster_entity=True)) 
```

```py
 RandomEffects Estimation Summary                          
====================================================================================
Dep. Variable:     log_total_output_value   R-squared:                        0.3689
Estimator:                  RandomEffects   R-squared (Between):             -0.0001
No. Observations:                    1227   R-squared (Within):               0.0025
Date:                    Fri, Nov 06 2020   R-squared (Overall):             -0.0027
Time:                            21:12:13   Log-likelihood                   -1260.9
Cov. Estimator:                 Clustered                                           
                                            F-statistic:                      716.04
Entities:                             938   P-value                           0.0000
Avg Obs:                           1.3081   Distribution:                  F(1,1225)
Min Obs:                           1.0000                                           
Max Obs:                           2.0000   F-statistic (robust):             0.5811
                                            P-value                           0.4460
Time periods:                           2   Distribution:                  F(1,1225)
Avg Obs:                           613.50                                           
Min Obs:                           575.00                                           
Max Obs:                           652.00                                           

                              Parameter Estimates                               
================================================================================
              Parameter  Std. Err.     T-stat    P-value    Lower CI    Upper CI
--------------------------------------------------------------------------------
const            10.518     0.0603     174.36     0.0000      10.399      10.636
st_louis_fed     0.0721     0.0946     0.7623     0.4460     -0.1135      0.2577
================================================================================ 
```

## 练习

假设在非实验设置中，控制组与处理组不同。请解释是否合理使用双重差分（DID）来估计因果效应？在 DID 框架中，你应该修改或添加什么？

假设一项研究声称基于双重差分（DID）方法，联邦储备通过 2008 年的银行纾困避免了大规模的企业倒闭。假设另一项基于回归不连续（RD）的研究声称相反或否认了联邦储备对企业倒闭的影响。你认为哪种是更可信的经验策略，DID 还是 RD 来估计联邦政策的因果影响？请解释你的答案。

在面板数据中，分析单位可以是公司或县，公司级别的结果更可信还是县级别的结果更可信？请解释。

使用 Ziebarth（2013）的数据来估计圣路易斯联邦政策对公司收入的影响。具体来说，运行带有随机效应（RE）的双重差分（DID）。解释结果。关于未观察到的因素\(\alpha_i\)可以推断出什么？

使用 Ziebarth（2013）的数据来估计圣路易斯联邦政策对公司收入的影响。具体来说，运行带有公司固定效应（FE）的双重差分（DID），而不使用命令“entity_effects=True”。提示：你必须为每家公司使用虚拟变量。

## 参考

凯恩斯，约翰梅纳德。 （1936）。《就业、利息和货币的一般理论》。由美国 Harcourt，Brace 和 Company 出版，并由美国 Polygraphic Company 在纽约印刷。

理查森，加里和威廉特鲁斯特（2009 年）。《大萧条期间货币干预减轻了银行恐慌：来自联邦储备区域边界的准实验证据，1929-1933 年》。《政治经济学杂志》117（6）：1031-73。

齐巴斯，尼古拉斯 L.（2013 年）。《通过大萧条期间密西西比州的自然实验来识别银行倒闭的影响》。《美国经济杂志：宏观经济学》5（1）：81-101。
