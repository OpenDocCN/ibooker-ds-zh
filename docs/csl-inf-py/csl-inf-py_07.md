# 六、2007-2009 年大衰退期间加拿大就业市场上白人女性名字的溢价

> 原文：[`causal-methods.github.io/Book/6%29_The_Premium_of_Having_a_White_Female_Name_in_the_Canadian_Job_Market_During_the_Great_Recession_2007_2009.html`](https://causal-methods.github.io/Book/6%29_The_Premium_of_Having_a_White_Female_Name_in_the_Canadian_Job_Market_During_the_Great_Recession_2007_2009.html)
>
> 译者：[飞龙](https://github.com/wizardforcel)
>
> 协议：[CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/)


[Vitor Kamada](https://www.linkedin.com/in/vitor-kamada-1b73a078)

电子邮件：econometrics.methods@gmail.com

最近更新：2020 年 9 月 15 日

我重新分析了 Oreopoulos（2011）的实验数据，并发现在 2007-2009 年大萧条期间，在加拿大的就业市场中，拥有白人女性名字是有优势的。白人女性在 2009 年 2 月至 9 月之间的回电率比白人男性高出 8%。考虑到白人男性在不同的回归规范下的回电率约为 10%，这一效应的幅度是相当高的。

Oreopoulos（2011）发现，英文名字的回电率为 15.7%，而印度、巴基斯坦、中国和希腊名字的回电率为 6%。我认为他的主要发现在很大程度上是由白人女性驱动的。我发现，在 2009 年 2 月至 9 月的大萧条最严重时期，拥有白人男性名字与印度、中国和希腊男性名字相比，并没有太多优势。 

我使用了 Oreopoulos（2011）的数据集。每一行是发送给多个多伦多和蒙特利尔地区职业的简历。

```py
import numpy as np
import pandas as pd
pd.set_option('precision', 3)

# Data from Oreopoulos (2011)
path = "https://github.com/causal-methods/Data/raw/master/" 
df = pd.read_stata(path + "oreopoulos.dta")
df.head(5) 
```

|  | 公司 ID | 职业类型 | name_ethnicity | 额外证书 | 名字 | 语言技能 | 资格认证 | 参考 | 法律 | 列出的资格认证 | ... | 说话技能 | 社交能力 | 写作能力 | 秋季数据 | 中国人 | 印度人 | 英国人 | 巴基斯坦人 | 中加人 | 相同经验 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | -3 | 行政 | 加拿大 | 0.0 | JillWilson | 0.0 | 0.0 | 0.0 | 0.0 | 0.0 | ... | 70.0 | 50.0 | 67.0 | 2.0 | NaN | NaN | NaN | NaN | NaN | NaN |
| 1 | -3 | 行政 | 印度 | 0.0 | PanavSingh | 0.0 | 0.0 | 0.0 | 0.0 | 0.0 | ... | 70.0 | 50.0 | 67.0 | 2.0 | NaN | NaN | NaN | NaN | NaN | NaN |
| 2 | -3 | 行政 | 印度 | 0.0 | RahulKaur | 0.0 | 0.0 | 0.0 | 1.0 | 1.0 | ... | 70.0 | 50.0 | 67.0 | 2.0 | NaN | NaN | NaN | NaN | NaN | NaN |
| 3 | -3 | 行政 | 中国 | 0.0 | 雷丽 | 0.0 | 1.0 | 1.0 | 0.0 | 1.0 | ... | 70.0 | 50.0 | 67.0 | 2.0 | NaN | NaN | NaN | NaN | NaN | NaN |
| 4 | -4 | 行政 | 印度 | 0.0 | MayaKumar | 1.0 | 0.0 | 0.0 | 0.0 | 0.0 | ... | 80.0 | 70.0 | 65.0 | 2.0 | NaN | NaN | NaN | NaN | NaN | NaN |

5 行×31 列

```py
# Transform the variable of interest in % 
df["callback"] = 100*df["callback"] 
```

Oreopoulos（2011）收集的第一批实验数据是在 2008 年 4 月至 8 月之间。这是大萧条的“起始期”。

```py
# Restrict data to April and August 2008
df0 = df[(df.fall_data == 0)] 
```

回电率对工作面试的比例对于加拿大名字（15.84%）要比中国和印度名字（9.23%和 9.53%）高得多。名族是随机的。不可能争辩说加拿大人有更多的教育或经验来证明大约 5%的差异。所有的简历在质量上都是一样的，除了申请人的名字。因此，我们可以得出结论，对移民的歧视是一个真实的现象。

```py
mean = df0.groupby('name_ethnicity').agg([np.mean, np.size])
mean["callback"] 
```

|  | 平均值 | 大小 |
| --- | --- | --- |
| name_ethnicity |  |  |
| --- | --- | --- |
| 加拿大 | 15.845 | 953.0 |
| 中国 | 9.231 | 1430.0 |
| 印度 | 9.534 | 1416.0 |

在 2008 年 4 月至 8 月的样本中，女性名字似乎比男性名字稍微有一些优势，但可以忽略不计，以获得工作面试。这一结果支持了 Oreopoulos（2011）的发现。在他的论文中，女性的系数在大部分回归中都不具有统计学显著性。

```py
prop = pd.crosstab(index= df0['name_ethnicity'], columns=df0['female'], 
            values=df0['callback'], aggfunc='mean')
prop 
```

| 女性 | 0.0 | 1.0 |
| --- | --- | --- |
| name_ethnicity |  |  |
| --- | --- | --- |
| 加拿大 | 15.551 | 16.122 |
| 中国 | 9.065 | 9.392 |
| 印度 | 9.490 | 9.577 |

Oreopoulos（2011）收集的第三波实验数据是在 2009 年 2 月至 9 月之间进行的。这是大萧条的最糟糕时期。

```py
# Restrict data to February and September 2009
df2 = df[(df.fall_data == 2)] 
```

加拿大名字的回电率为 14％，而中国名字为 8.96％，英文名和中国姓氏为 7.13％，希腊为 10.11％，印度为 7.9％。

请注意，总体上，这第三波样本中的回电率略低于第一波样本，对于两个样本中的常见种族来说。

```py
mean = df2.groupby('name_ethnicity').agg([np.mean, np.size])
mean["callback"] 
```

|  | 平均 | 大小 |
| --- | --- | --- |
| 名字种族 |  |  |
| --- | --- | --- |
| 加拿大 | 14.080 | 1044.0 |
| 中国 | 8.956 | 1418.0 |
| 中国-加拿大 | 7.128 | 491.0 |
| 希腊 | 10.109 | 366.0 |
| 印度 | 7.899 | 1937.0 |

```py
import plotly.express as px

y = mean["callback"].values[:, 0]
x = mean["callback"].index

fig = px.bar(df2, x, y, color = x,
             title="Callback Rate for Interview by Name Ethnicity",
             labels={ "y": "Callback Rate (%)",
                      "x": "Name Ethnicity",                 
                      "color": ""} )

fig.update_layout(font_size = 17)
fig.show() 
```

在 2009 年 2 月至 9 月的样本中，白人女性的名字回电率为 18.3％，而白人男性的名字回电率为 10.17％。我们没有看到其他种族有这么大的差异。事实上，对于希腊名字来说，效果是相反的。希腊男性的名字回电率为 10.71％，而希腊女性的名字回电率为 9.6％。白人男性的名字比中国和印度男性的名字有优势，但幅度不像白人男性与白人女性之间的差异那么大。

```py
prop = pd.crosstab(index= df2['name_ethnicity'], columns=df2['female'], 
            values=df2['callback'], aggfunc='mean')
prop 
```

| 女性 | 0.0 | 1.0 |
| --- | --- | --- |
| 名字种族 |  |  |
| --- | --- | --- |
| 加拿大 | 10.169 | 18.129 |
| 中国 | 8.715 | 9.177 |
| 中国-加拿大 | 6.410 | 7.782 |
| 希腊 | 10.714 | 9.596 |
| 印度 | 7.252 | 8.559 |

```py
import plotly.graph_objects as go

ethnicity = prop.index
male = prop.values[:,0]
female = prop.values[:,1]

fig = go.Figure(data=[
         go.Bar(name='Male', x = ethnicity, y = male),
         go.Bar(name='Female', x = ethnicity, y = female) ])

fig.update_layout(barmode='group', font_size = 17,
      title = "Callback Rate for Interview by Gender",
      yaxis = dict(title='Callback Rate (%)'),
      xaxis = dict(title='Name Ethnicity') )

fig.show() 
```

有人可能会争辩说，有一些混杂因素导致了观察到的差异。例如，有人可能会说，在现实世界中，女性比男性更受教育和更合格。请记住，这是实验数据，所有简历都是人为构造的，所有相关维度都是由 Oreopoulos（2011）随机化的。控制变量表明，女性和男性彼此相似。

```py
control = ['additional_credential', 'ba_quality',
           'extracurricular_skills', 'language_skills',
           'certificate', 'ma', 'same_exp', 'exp_highquality',
           'skillspeaking', 'skillsocialper', 'skillwriting']

df2.groupby('female').agg([np.mean])[control] 
```

|  | 附加证书 | 学士质量 | 课外技能 | 语言技能 | 证书 | 硕士 | 相同经验 | 经验高质量 | 说话技能 | 社交能力 | 写作技能 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  | 平均 | 平均 | 平均 | 平均 | 平均 | 平均 | 平均 | 平均 | 平均 | 平均 | 平均 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 女性 |  |  |  |  |  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0.0 | 0.059 | 0.640 | 0.598 | 0.306 | 0.007 | 0.170 | NaN | 0.197 | 70.757 | 59.654 | 64.286 |
| 1.0 | 0.054 | 0.655 | 0.595 | 0.322 | 0.008 | 0.185 | NaN | 0.169 | 70.524 | 59.777 | 64.216 |

敏锐的读者可能会争辩说，仅仅表明男性和女性相互之间相似是不够的，以支持我关于白人女性溢价的论点。我必须表明样本中的平均白人女性与平均白人男性相似。对于某些变量，白人女性看起来略微更合格，但对于其他变量，略微不太合格。许多数据维度都是随机的，观察到的差异看起来是抽样变异的产物。总的来说，白人男性和女性看起来很相似。我们可以在回归框架中严格控制所有这些因素。我看到种族之间的变化比性别之间的变化更大。种族之间的变化看起来对实验来说过多。因此，我将按种族分解回归分析，并控制几个因素。

```py
df2.groupby(['female', 'name_ethnicity']).agg([np.mean])[control] 
```

|  |  | 附加证书 | 学士质量 | 课外技能 | 语言技能 | 证书 | 硕士 | 相同经验 | 经验高质量 | 说话技能 | 社交能力 | 写作技能 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  | 平均 | 平均 | 平均 | 平均 | 平均 | 平均 | 平均 | 平均 | 平均 | 平均 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 女性 | 名字种族 |  |  |  |  |  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0.0 | 加拿大 | 0.056 | 0.746 | 0.623 | 0.343 | 0.004 | 0.209 | NaN | 0.190 | 70.422 | 59.070 | 63.546 |
| 中国人 | 0.062 | 0.607 | 0.592 | 0.326 | 0.007 | 0.165 | NaN | 0.183 | 70.702 | 59.850 | 64.312 |
| 中-加 | 0.068 | 0.628 | 0.538 | 0.282 | 0.013 | 0.154 | NaN | 0.214 | 71.141 | 59.513 | 63.979 |
| 希腊人 | 0.065 | 0.774 | 0.631 | 0.321 | 0.012 | 0.214 | NaN | 0.232 | 70.976 | 59.696 | 65.220 |
| 印度人 | 0.055 | 0.586 | 0.596 | 0.276 | 0.006 | 0.147 | NaN | 0.200 | 70.846 | 59.862 | 64.582 |
| 1.0 | 加拿大 | 0.055 | 0.789 | 0.577 | 0.327 | 0.004 | 0.228 | NaN | 0.177 | 70.146 | 60.041 | 64.179 |
| 中国人 | 0.055 | 0.590 | 0.596 | 0.300 | 0.011 | 0.189 | NaN | 0.179 | 70.799 | 59.607 | 64.349 |
| 中-加 | 0.047 | 0.638 | 0.638 | 0.346 | 0.004 | 0.113 | NaN | 0.171 | 70.000 | 59.506 | 63.233 |
| 希腊人 | 0.045 | 0.808 | 0.566 | 0.308 | 0.010 | 0.217 | NaN | 0.136 | 71.258 | 60.662 | 64.894 |
| 印度人 | 0.055 | 0.608 | 0.599 | 0.334 | 0.008 | 0.172 | NaN | 0.163 | 70.503 | 59.657 | 64.257 |

让$y_{rjt}$是一个虚拟变量，如果简历$r$发送到工作$j$在时间$t$收到回电，则为 1；否则为 0。感兴趣的变量是“女性”虚拟变量和与“简历类型”的交互。

有五种“简历类型”：0) 英文名，加拿大教育和经验；1) 外国名字，加拿大教育和经验；2) 外国名字和教育，加拿大经验；3) 外国名字和教育，混合经验；和 4) 外国名字，教育和经验。

以下的线性概率模型是首选的规范：

$$y_{rjt}= \beta Female_{rjt}+\gamma Resume\ Types_{rjt}+ \delta Female_{rjt} \cdot Resume\ Types_{rjt} + \alpha X + \epsilon_{rjt}$$

其中$X$是控制变量的向量，$\epsilon_{rjt}$是通常的误差项。所有回归都呈现了对异方差性的稳健标准误差。

对于表 1、2 和 3，我们呈现了 4 个回归，以比较“加拿大人”与特定种族。逻辑是保持一个同质样本，避免可能混淆结果的种族变化。

表 1 呈现了没有交互作用和控制变量的结果。作为女性的优势范围从增加 3.64%到 5.97%的回电率，相对于男性。白人男性的回电率，基准（类型 0），范围从 11.14%到 12.29%。Oreopoulos（2011）提出的类型 0 的估计值从 15.4%到 16%不等，但他的估计捕捉了英文名字的影响，而没有孤立地考虑性别影响。

我们看到一个模式，类型 1、2、3 和 4 的系数都是负数，并且随着“外国”的程度绝对值增加。一个人在名字、教育和经验方面越“外国”，回电率就越低。但仅仅一个外国名字就足以使回电率比英文名字低 3.38%到 5.11%。总体而言，结果在 1%的显著水平上是统计学显著的。一个例外是类型 1 的系数，用于英文名和中国姓氏的回归（3）。这里描述的模式与 Oreopoulos（2011）报告的主要发现相匹配。

```py
import statsmodels.formula.api as smf

# Sample Restriction based on name ethnicity
Canada = df2.name_ethnicity == "Canada"
Indian = df2[(Canada) | (df2.name_ethnicity == "Indian")]
Chinese = df2[(Canada) | (df2.name_ethnicity == "Chinese")]
Chn_Cdn = df2[(Canada) | (df2.name_ethnicity == "Chn-Cdn")]
Greek = df2[(Canada) | (df2.name_ethnicity == "Greek")]

sample = [Indian, Chinese, Chn_Cdn, Greek]

#  Run the simple model for each ethnicity
# and save the results
model1 = "callback ~ female + C(type)"

result1 = []
for data in sample:
   ols = smf.ols(model1, data).fit(cov_type='HC1')
   result1.append(ols) 
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

```py
# Settings for a nice table
from stargazer.stargazer import Stargazer
stargazer = Stargazer(result1)

stargazer.title('Table 1 - Callback Rates by Resume Type')

names = ['Indian', 'Chinese', 'Chn_Cdn', 'Greek']
stargazer.custom_columns(names, [1, 1, 1, 1])

order = ['female', 'Intercept', 'C(type)[T.1.0]',
         'C(type)[T.2.0]', 'C(type)[T.3.0]',
         'C(type)[T.4.0]']     
stargazer.covariate_order(order)

dict1 = {'C(type)[T.1.0]': '1) Foreign Name, Cdn Educ and Exp',
         'C(type)[T.2.0]': '2) Foreign Name and Educ, Cdn exp',
         'C(type)[T.3.0]': '3) Foreign Name and Educ, Mixed Exp',
         'C(type)[T.4.0]': '4) All Foreign (Name, Educ, and Exp)',
              'Intercept': '0) English Name, Cdn Educ and Exp',
                 'female': 'Female'}
stargazer.rename_covariates(dict1)

stargazer 
```

表 1 - 简历类型的回电率

|  |
| --- |
|  | *因变量：回电* |
|  |
|  | 印度人 | 中国人 | 中-加 | 希腊人 |
|  | (1) | (2) | (3) | (4) |
|  |
| 女性 | 3.648^(***) | 3.639^(***) | 5.974^(***) | 5.614^(***) |
|  | (1.096) | (1.266) | (1.641) | (1.797) |
| 0) 英文名，加拿大教育和经验 | 12.288^(***) | 12.292^(***) | 11.145^(***) | 11.322^(***) |
|  | (1.119) | (1.144) | (1.196) | (1.236) |
| 1) 外国名字，加拿大教育和经验 | -5.112^(***) | -3.382^* | -3.068 | -4.250^(**) |
|  | (1.523) | (1.748) | (2.531) | (1.929) |
| 2) 外国名字和教育，加拿大经验 | -4.024^(**) | -6.713^(***) | -9.047^(***) |  |
|  | (1.712) | (1.809) | (2.291) |  |
| 3) 外国名字和教育，混合经验 | -8.603^(***) | -5.960^(***) | -7.571^(***) |  |
|  | (1.574) | (1.893) | (2.865) |  |
| 4) 所有外国人（姓名，教育和经验） | -8.916^(***) | -6.184^(***) | -12.592^(***) |  |
|  | (1.626) | (2.004) | (1.863) |  |
|  |
| 观察 | 2,981 | 2,462 | 1,535 | 1,410 |
| R² | 0.016 | 0.011 | 0.022 | 0.010 |
| 调整后的 R² | 0.015 | 0.009 | 0.019 | 0.008 |
| 残差标准误差 | 29.870 (df=2975) | 31.314 (df=2456) | 32.030 (df=1529) | 33.558 (df=1407) |
| F 统计量 | 9.668^(***) (df=5; 2975) | 5.181^(***) (df=5; 2456) | 11.715^(***) (df=5; 1529) | 6.261^(***) (df=2; 1407) |
|  |
| 注： | ^*p<0.1; ^(**)p<0.05; ^(***)p<0.01 |

表 2 添加了女性和简历类型的交互项。女性的系数仅捕捉了白人女性的影响，因为外国女性是由女性和类型之间的交互项捕捉的。与白人男性（10.17%的基线）相比，白人女性的回访率增加了 7.96%。

交互项的系数在绝对值上为负，但并非全部统计上显着。这种模式表明，外国女性的回访率与白人女性相比非常低。 

有趣的是，类型 1、2、3 和 4 的系数在幅度上较低，并且与表 1 相比在统计上不太显着。这种模式表明，白人男性比印度人和中国姓氏的人有优势，但不包括希腊人或中国人（名字和姓氏）。这两个最后一组的系数在统计上不显着。

```py
model2 = "callback ~ female*C(type)"

result2 = []
for data in sample:
   ols = smf.ols(model2, data).fit(cov_type='HC1')
   result2.append(ols) 
```

```py
stargazer = Stargazer(result2)

stargazer.title('Table 2 - Callback Rates by Resume Type and Gender')

stargazer.custom_columns(names, [1, 1, 1, 1])

dict2 = {'female:C(type)[T.1.0]':'[Female]x[1]',
         'female:C(type)[T.2.0]':'[Female]x[2]',
         'female:C(type)[T.3.0]':'[Female]x[3]',
         'female:C(type)[T.4.0]':'[Female]x[4]'}

list2 = list(dict2.keys())

dict2.update(dict1)
stargazer.rename_covariates(dict2)

list2 = order + list2
stargazer.covariate_order(list2)

stargazer 
```

表 2 - 简历类型和性别的回访率

|  |
| --- |
|  | *因变量：回访* |
|  |
|  | 印度 | 中国 | 中国加拿大 | 希腊 |
|  | (1) | (2) | (3) | (4) |
|  |
| 女性 | 7.959^(***) | 7.959^(***) | 7.959^(***) | 7.959^(***) |
|  | (2.152) | (2.152) | (2.155) | (2.151) |
| 0) 英文名，加拿大教育和经验 | 10.169^(***) | 10.169^(***) | 10.169^(***) | 10.169^(***) |
|  | (1.314) | (1.314) | (1.316) | (1.314) |
| 1) 外国名字，加拿大教育和经验 | -3.829^(**) | 1.306 | -0.492 | 0.545 |
|  | (1.856) | (2.431) | (3.345) | (2.727) |
| 2) 外国名字和教育，加拿大经验 | 0.437 | -3.676 | -6.780^(**) |  |
|  | (2.309) | (2.385) | (2.705) |  |
| 3) 外国名字和教育，混合经验 | -4.139^* | -2.526 | -1.281 |  |
|  | (2.141) | (2.498) | (4.455) |  |
| 4) 所有外国人（姓名，教育和经验） | -4.844^(**) | -2.792 | -10.169^(***) |  |
|  | (2.172) | (2.711) | (1.316) |  |
| [女性]x[1] | -2.651 | -9.178^(***) | -5.137 | -9.077^(**) |
|  | (3.046) | (3.493) | (5.057) | (3.838) |
| [女性]x[2] | -9.177^(***) | -6.026^* | -4.569 |  |
|  | (3.423) | (3.598) | (4.584) |  |
| [女性]x[3] | -8.989^(***) | -6.993^* | -12.500^(**) |  |
|  | (3.142) | (3.792) | (5.644) |  |
| [女性]x[4] | -8.316^(**) | -6.703^* | -4.388 |  |
|  | (3.250) | (3.994) | (3.291) |  |
|  |
| 观察 | 2,981 | 2,462 | 1,535 | 1,410 |
| R² | 0.021 | 0.015 | 0.025 | 0.013 |
| 调整后的 R² | 0.018 | 0.011 | 0.019 | 0.011 |
| 残差标准误差 | 29.823 (df=2971) | 31.280 (df=2452) | 32.027 (df=1525) | 33.512 (df=1406) |
| F 统计量 | 6.217^(***) (df=9; 2971) | 3.334^(***) (df=9; 2452) | 23.435^(***) (df=9; 1525) | 5.478^(***) (df=3; 1406) |
|  |
| 注： | ^*p<0.1; ^(**)p<0.05; ^(***)p<0.01 |

表 3 添加了控制变量作为鲁棒性检查。总体结果与表 2 相似。与表 2 相比，白人女性的影响甚至略有增加。白人女性的巨大溢价仍然超过所有其他类别。白人女性的溢价优于来自世界排名前 200 的大学的学士学位，大型公司的经验，课外活动，流利的法语和其他语言以及加拿大硕士学位的累积影响。

请注意，对于类型 1，只有印度回归的系数在统计上显着。白人男性的名字与中国，中国加拿大和希腊名字没有优势。

```py
control1 = "+ ba_quality + extracurricular_skills + language_skills"
control2 = "+ ma + exp_highquality"
model3 = "callback ~ female*C(type)" + control1 + control2

result3 = []
for data in sample:
   ols = smf.ols(model3, data).fit(cov_type='HC1')
   result3.append(ols) 
```

```py
stargazer = Stargazer(result3)

stargazer.title('Table 3 - Callback Rates and Robustness Checks')

stargazer.custom_columns(names, [1, 1, 1, 1])

dict3 = {'ba_quality':'Top 200 world ranking university',
         'exp_highquality':'High quality work experience',
         'extracurricular_skills'	:'List extra-curricular activities',
         'language_skills':'Fluent in French and other languages',
         'ma':'Canadian master’s degree'}

list3 = list(dict3.keys())

dict3.update(dict2)
stargazer.rename_covariates(dict3)

list3 = list2 + list3
stargazer.covariate_order(list3)

stargazer 
```

表 3 - 回访率和鲁棒性检查

|  |
| --- |
|  | *因变量：回访* |
|  |
|  | 印度 | 中国 | 中国加拿大 | 希腊 |
|  | (1) | (2) | (3) | (4) |
|  |
| 女性 | 8.204^(***) | 8.255^(***) | 8.004^(***) | 8.295^(***) |
|  | (2.146) | (2.148) | (2.157) | (2.156) |
| 0) 英文名字，加拿大教育和经验 | 9.150^(***) | 10.445^(***) | 11.024^(***) | 11.480^(***) |
|  | (1.819) | (2.038) | (2.444) | (2.654) |
| 1) 外国名字，加拿大教育和经验 | -3.392^* | 1.859 | 0.673 | 0.685 |
|  | (1.856) | (2.459) | (3.368) | (2.721) |
| 2) 外国名字和教育，加拿大经验 | 0.245 | -4.620^* | -7.554^(***) |  |
|  | (2.354) | (2.376) | (2.626) |  |
| 3) 外国名字和教育，混合经验 | -4.272^(**) | -3.355 | -1.926 |  |
|  | (2.175) | (2.504) | (4.434) |  |
| 4) 所有外国人（姓名，教育和经验） | -5.147^(**) | -3.768 | -10.405^(***) |  |
|  | (2.175) | (2.784) | (1.607) |  |
| [女性]x[1] | -3.348 | -9.773^(***) | -6.237 | -8.966^(**) |
|  | (3.033) | (3.505) | (5.037) | (3.837) |
| [女性]x[2] | -9.607^(***) | -5.873 | -4.512 |  |
|  | (3.405) | (3.588) | (4.552) |  |
| [女性]x[3] | -9.608^(***) | -7.426^* | -12.058^(**) |  |
|  | (3.140) | (3.793) | (5.571) |  |
| [女性]x[4] | -8.414^(***) | -6.724^* | -3.811 |  |
|  | (3.232) | (3.989) | (3.382) |  |
| 世界排名前 200 的大学 | -2.391^* | -3.752^(***) | -2.622 | -4.440^* |
|  | (1.228) | (1.450) | (1.970) | (2.330) |
| 高质量的工作经验 | -0.330 | -1.026 | -2.208 | 2.119 |
|  | (1.399) | (1.620) | (1.954) | (2.424) |
| 列出课外活动 | 1.807^* | 1.869 | -1.739 | 0.400 |
|  | (1.093) | (1.263) | (1.721) | (1.825) |
| 流利的法语和其他语言 | 4.718^(***) | 4.308^(***) | 5.939^(***) | 4.712^(**) |
|  | (1.255) | (1.434) | (1.881) | (2.021) |
| 加拿大硕士学位 | 0.583 | 0.364 | 2.720 | -1.275 |
|  | (1.507) | (1.663) | (2.285) | (2.101) |
|  |
| 观察 | 2,981 | 2,462 | 1,535 | 1,410 |
| R² | 0.029 | 0.023 | 0.037 | 0.023 |
| 调整后的 R² | 0.024 | 0.018 | 0.028 | 0.017 |
| 残差标准误差 | 29.726（df=2966） | 31.174（df=2447） | 31.875（df=1520） | 33.403（df=1401） |
| F 统计量 | 5.297^(***)（df=14; 2966） | 3.563^(***)（df=14; 2447） | 11.819^(***)（df=14; 1520） | 3.413^(***)（df=8; 1401） |
|  |
| 注： | ^*p<0.1; ^(**)p<0.05; ^(***)p<0.01 |

## 练习

1）为什么白人女性的溢价出现在 2009 年 2 月至 9 月的大衰退期间，而在 2008 年 4 月和 8 月之前没有出现？推测。

2）招聘人员可能更愿意与白人女性交谈，但不一定会雇佣她们。我如何能确定更高的回电率是否反映在更多的工作提供中。例如，我如何获取数据来检查这种关系？

3）你能从下表推断出什么？你有什么见解要分享吗？

```py
pd.crosstab(index= [df2['name_ethnicity'], df2['female'],
                           df2['name']], columns=df2['type'], 
                         values=df2['callback'], aggfunc='mean') 
```

|  |  | 类型 | 0.0 | 1.0 | 2.0 | 3.0 | 4.0 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 名字 _ 种族 | 女性 | 名字 |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 加拿大 | 0.0 | 格雷格·约翰逊 | 11.561 | NaN | NaN | NaN | NaN |
| 约翰·马丁 | 8.235 | NaN | NaN | NaN | NaN |
| 马修·威尔逊 | 10.638 | NaN | NaN | NaN | NaN |
| 1.0 | 艾莉森·约翰逊 | 18.675 | NaN | NaN | NaN | NaN |
| 凯丽·马丁 | 20.455 | NaN | NaN | NaN | NaN |
| 吉尔·威尔逊 | 15.205 | NaN | NaN | NaN | NaN |
| 中国人 | 0.0 | 刘东 | NaN | 10.870 | 3.390 | 13.158 | 2.381 |
| 李蕾 | NaN | 14.062 | 9.756 | 8.065 | 11.364 |
| 张勇 | NaN | 10.227 | 7.407 | 3.509 | 8.333 |
| 1.0 | 刘敏 | NaN | 8.235 | 5.357 | 11.321 | 15.556 |
| 李娜 | NaN | 10.127 | 12.698 | 5.556 | 2.703 |
| 张秀英 | NaN | 11.927 | 6.780 | 9.091 | 7.018 |
| 中国-加拿大 | 0.0 | 王艾瑞克 | NaN | 9.677 | 3.390 | 8.889 | 0.000 |
| 1.0 | 王美琪 | NaN | 12.500 | 6.780 | 4.348 | 3.571 |
| 希腊 | 0.0 | 鲁卡斯·米诺普洛斯 | NaN | 10.714 | NaN | NaN | NaN |
| 1.0 | NicoleMinsopoulos | NaN | 9.596 | NaN | NaN | NaN |
| 印度人 | 0.0 | 阿尔琼·库马尔 | NaN | 8.642 | 8.333 | 3.125 | 4.762 |
| 帕纳夫·辛格 | NaN | 1.333 | 15.942 | 7.317 | 2.500 |
| 拉胡尔·考尔 | NaN | 8.571 | 10.769 | 5.455 | 7.317 |
| 萨米尔·夏尔马 | NaN | 5.814 | 6.897 | 10.256 | 6.522 |
| 1.0 | MayaKumar | NaN | 14.286 | 6.944 | 3.448 | 5.263 |
| PriyankaKaur | NaN | 6.481 | 14.815 | 3.636 | 5.128 |
| ShreyaSharma | NaN | 13.158 | 8.772 | 4.651 | 5.128 |
| TaraSingh | NaN | 14.286 | 8.065 | 9.091 | 4.444 |

4）你能从下表推断出什么？你有什么见解要分享吗？

```py
pd.crosstab(index= df2['occupation_type'],
                   columns=[df2['name_ethnicity'], df2['female']], 
                   values=df2['callback'], aggfunc='mean') 
```

| 姓名种族 | 加拿大 | 中国 | 中国-加拿大 | 希腊 | 印度 |
| --- | --- | --- | --- | --- | --- |
| 女性 | 0.0 | 1.0 | 0.0 | 1.0 | 0.0 | 1.0 | 0.0 | 1.0 | 0.0 | 1.0 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 职业类型 |  |  |  |  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 会计 | 2.703 | 8.929 | 5.769 | 6.780 | 0.000 | 4.545 | 7.692 | 14.286 | 1.429 | 4.348 |
| 行政 | 7.895 | 23.288 | 10.112 | 8.036 | 6.897 | 8.333 | 13.793 | 10.714 | 7.383 | 6.015 |
| 土木工程师 | 5.556 | 50.000 | 6.250 | 6.250 | 0.000 | 11.111 | 0.000 | 0.000 | 4.762 | 0.000 |
| 文书工作 | 4.444 | 8.140 | 5.172 | 6.000 | 0.000 | 7.692 | 9.091 | 3.704 | 5.000 | 4.790 |
| 电子商务 | 0.000 | 0.000 | 9.091 | 0.000 | 0.000 | 0.000 | 0.000 | 0.000 | 0.000 | 0.000 |
| 电气工程师 | 6.250 | 28.571 | 7.143 | 8.333 | 14.286 | 0.000 | NaN | 33.333 | 16.667 | 0.000 |
| 行政助理 | 23.077 | 17.647 | 5.263 | 16.000 | 16.667 | 16.667 | 14.286 | 0.000 | 6.250 | 7.407 |
| 金融 | 16.667 | 26.316 | 5.000 | 12.195 | 0.000 | 9.091 | 14.286 | 0.000 | 12.766 | 2.439 |
| 餐饮服务经理 | 16.667 | 16.667 | 0.000 | 8.333 | 20.000 | 0.000 | 0.000 | 0.000 | 0.000 | 7.143 |
| 人力资源工资 | 20.000 | 18.182 | 0.000 | 0.000 | 0.000 | 20.000 | 0.000 | 0.000 | 6.250 | 0.000 |
| 保险 | 53.846 | 40.000 | 14.286 | 13.636 | 28.571 | 16.667 | 0.000 | 0.000 | 25.926 | 40.000 |
| 市场营销和销售 | 12.791 | 22.222 | 12.409 | 11.966 | 9.091 | 4.545 | 16.129 | 18.605 | 7.143 | 10.638 |
| 生产 | 0.000 | 0.000 | 0.000 | 4.762 | 0.000 | 0.000 | 0.000 | 0.000 | 3.571 | 2.857 |
| 程序员 | 10.256 | 17.391 | 13.462 | 10.526 | 0.000 | 16.000 | 16.667 | 5.882 | 11.688 | 17.333 |
| 零售 | 19.048 | 21.622 | 14.545 | 17.647 | 22.727 | 5.556 | 6.667 | 15.000 | 6.757 | 19.403 |
| 技术 | 0.000 | 16.667 | 3.226 | 4.000 | 0.000 | 9.091 | 33.333 | 0.000 | 5.556 | 7.407 |

5）解释表 4 的结果。重点关注固定效应（职业，姓名和城市）的添加。

```py
FE = "+ C(occupation_type) + C(city) + C(name)"
model4 = "callback ~ female*C(type) " + control1 + control2 + FE

result4 = []
for data in sample:
   ols = smf.ols(model4, data).fit(cov_type='HC1')
   result4.append(ols) 
```

```py
stargazer = Stargazer(result4)

stargazer.title('Table 4 - Callback Rates and Fixed Effects')
stargazer.custom_columns(names, [1, 1, 1, 1])
stargazer.rename_covariates(dict3)
stargazer.covariate_order(list3)

stargazer.add_line('Fixed Effects', ['', '', '', ''])
stargazer.add_line('Occupation', ['Yes', 'Yes', 'Yes', 'Yes'])
stargazer.add_line('Name', ['Yes', 'Yes', 'Yes', 'Yes'])
stargazer.add_line('City', ['Yes', 'Yes', 'Yes', 'Yes'])

stargazer 
```

```py
C:\Anaconda\envs\textbook\lib\site-packages\statsmodels\base\model.py:1752: ValueWarning:

covariance of constraints does not have full rank. The number of constraints is 43, but rank is 1

C:\Anaconda\envs\textbook\lib\site-packages\statsmodels\base\model.py:1752: ValueWarning:

covariance of constraints does not have full rank. The number of constraints is 41, but rank is 39

C:\Anaconda\envs\textbook\lib\site-packages\statsmodels\base\model.py:1752: ValueWarning:

covariance of constraints does not have full rank. The number of constraints is 37, but rank is 35

C:\Anaconda\envs\textbook\lib\site-packages\statsmodels\base\model.py:1752: ValueWarning:

covariance of constraints does not have full rank. The number of constraints is 31, but rank is 29 
```

表 4 - 回拨率和固定效应

|  |
| --- |
|  | *因变量：回拨* |
|  |
| 印度 | 中国 | 中国-加拿大 | 希腊 |
|  | (1) | (2) | (3) | (4) |
|  |
| 女性 | 2168155124746.490 | 8.231^(***) | 7.999^(***) | 7.792^(***) |
|  | (3022361939094.652) | (2.376) | (2.485) | (2.480) |
| 0) 英文姓名，加拿大教育和经验 | -2168155124731.589 | 9.538^(***) | 7.654^(***) | 8.468^(***) |
|  | (3022361939094.750) | (2.105) | (2.473) | (2.807) |
| 1) 外国姓名，加拿大教育和经验 | 4759752128597.301 | 1.385 | 1.978 | -1.574 |
|  | (6634992814337.623) | (2.063) | (2.687) | (1.396) |
| 2) 外国姓名和教育，加拿大经验 | 4759752128601.816 | -4.993^(**) | -5.685^(**) |  |
|  | (6634992814337.526) | (1.969) | (2.236) |  |
| 3) 外国姓名和教育，混合经验 | 4759752128596.260 | -2.756 | 2.146 |  |
|  | (6634992814337.408) | (2.129) | (3.472) |  |
| 4) 所有外国（姓名，教育和经验） | 4759752128595.535 | -4.003^* | -6.330^(***) |  |
|  | (6634992814336.644) | (2.225) | (1.687) |  |
| [女性]x[1] | -2358382463705.539 | -5.826^(**) | -0.835 | -3.019^(**) |
|  | (3287534786974.611) | (2.624) | (3.952) | (1.413) |
| [女性]x[2] | -2358382463711.714 | -0.692 | -0.194 |  |
|  | (3287534786974.415) | (2.679) | (3.520) |  |
| [女性]x[3] | -2358382463711.806 | -2.998 | -6.336 |  |
|  | (3287534786974.468) | (2.985) | (4.299) |  |
| [女性]x[4] | -2358382463710.742 | -2.505 | 0.693 |  |
|  | (3287534786973.757) | (2.988) | (3.001) |  |
| 世界排名前 200 的大学 | -0.292 | -0.891 | 0.834 | 2.322 |
|  | (1.274) | (1.503) | (2.172) | (3.111) |
| 高质量工作经验 | -0.723 | -0.925 | -1.886 | 2.698 |
|  | (1.363) | (1.596) | (1.899) | (2.364) |
| 列出课外活动 | 1.752 | 1.823 | -1.637 | 0.243 |
|  | (1.069) | (1.251) | (1.677) | (1.794) |
| 流利的法语和其他语言 | 2.268^* | 1.634 | 3.095^* | 1.924 |
|  | (1.272) | (1.484) | (1.870) | (2.081) |
| 加拿大硕士学位 | 0.717 | 0.567 | 3.083 | -0.794 |
|  | (1.452) | (1.628) | (2.239) | (2.053) |
| 固定效应 |  |  |  |  |
| 职业 | 是 | 是 | 是 | 是 |
| 名字 | 是 | 是 | 是 | 是 |
| 城市 | 是 | 是 | 是 | 是 |
|  |
| 观察 | 2,981 | 2,462 | 1,535 | 1,410 |
| R² | 0.082 | 0.066 | 0.097 | 0.076 |
| 调整后的 R² | 0.070 | 0.051 | 0.076 | 0.057 |
| 残差标准误差 | 29.024 (df=2940) | 30.638 (df=2423) | 31.083 (df=1500) | 32.726 (df=1381) |
| F 统计量 | 0.515 ^((df=40; 2940)) | 8.453^(***) (df=38; 2423) | 6.549^(***) (df=34; 1500) | 7.902^(***) (df=28; 1381) |
|  |
| 注： | ^*p<0.1; ^(**)p<0.05; ^(***)p<0.01 |

6）Oreopoulos (2011)收集的第二波实验数据是在 2008 年 9 月至 11 月之间。使用这些数据来调查在加拿大就业市场中是否拥有白人女性姓名会有额外的优势。只生成一张专业出版表，并解释主要结果。

7）对于这个问题，要像 Bertrand & Mullainathan (2004)和 Oreopoulos (2011)一样打破常规思维。一些研究认为身高较高的人赚更多钱并不是因为身高的直接影响，而是通过自尊心的间接影响。提出一个可行的研究设计来测试以下因果关系：

a) 身高和薪水。

b) 身高和自尊心。

c) 自尊心和薪水。

## 参考

Bertrand, Marianne, and Sendhil Mullainathan. (2004). [Are Emily and Greg More Employable Than Lakisha and Jamal? A Field Experiment on Labor Market Discrimination](https://github.com/causal-methods/Papers/raw/master/Are%20Emily%20and%20Greg%20More%20Employable%20than%20Lakisha%20and%20Jamal.pdf). American Economic Review, 94 (4): 991-1013.

Oreopoulos, Philip. (2011). [Why Do Skilled Immigrants Struggle in the Labor Market? A Field Experiment with Thirteen Thousand Resumes](https://github.com/causal-methods/Papers/raw/master/Oreopoulos/Why%20Do%20Skilled%20Immigrants%20Struggle%20in%20the%20Labor%20Market.pdf). American Economic Journal: Economic Policy, 3 (4): 148-71.
