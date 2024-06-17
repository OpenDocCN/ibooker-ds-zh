# 前言

> 统计学是一个用途惊人但实际有效的从业者很少的学科。
> 
> Bradley Efron和R. J. Tibshirani，《自举法入门》（1993年）

欢迎来到*用R和Python进行行为数据分析*！我们生活在数据时代已成陈词滥调。工程师现在常常利用机器和涡轮机上的传感器数据来预测它们的故障时间并进行预防性维护。同样地，营销人员利用大量数据，从你的人口统计信息到过去的购买记录，来确定何时向你展示哪些广告。正如一句俗语所言，“数据是新石油”，而算法是推动我们经济前进的新型燃烧引擎。

大多数关于分析、机器学习和数据科学的书籍都暗示工程师和营销人员试图解决的问题可以用相同的方法和工具处理。当然，变量有不同的名称，并且需要一些特定领域的知识，但是k均值聚类就是k均值聚类，无论是在涡轮机数据还是社交媒体帖子数据上。通过这种方式全盘采纳机器学习工具，企业通常能够准确预测行为，但却失去了对实际情况更深入和丰富的理解。这导致了数据科学模型被批评为“黑匣子”的现象。

本书不是追求准确但不透明的预测，而是努力回答这样一个问题：“是什么驱动了行为？”如果我们决定给潜在客户发送电子邮件，他们会因为电子邮件而订阅我们的服务吗？哪些客户群体应该收到电子邮件？年长客户是否因为年龄较大而倾向于购买不同的产品？客户体验对忠诚度和保留率的影响是什么？通过改变我们的视角，从预测行为转向解释行为并测量其原因，我们将能够打破“相关不等于因果”的诅咒，这一诅咒阻碍了几代分析师对他们模型结果的自信。

这种转变不会因引入新的分析工具而来：我们将只使用两种数据分析工具：传统的线性回归及其逻辑回归衍生物。这两种模型本质上更易读取，尽管通常以较低的预测准确度为代价（即在预测中产生更多且更大的误差），但对于我们在这里测量变量之间关系的目的来说并不重要。

相反，我们将花费大量时间学习如何理解数据。在我的数据科学面试官角色中，我见过许多候选人可以使用复杂的机器学习算法，但对数据的实际理解却不深：除了算法告诉他们的内容，他们对数据的运作几乎没有直观的感觉。

我相信你可以培养出这种直觉，并在这个过程中通过采用以下方法极大地增加你的分析项目的价值和成果：

+   一种行为科学的思维方式，将数据视为了解人类心理和行为的一种视角，而非终点。

+   一套因果分析工具包，让我们能够自信地说一件事情导致另一件事情，并确定这种关系有多强。

虽然这些方法各自都能带来巨大的收益，但我认为它们是天然的互补，最好一起使用。鉴于“使用因果分析工具包的行为科学思维方式”有点啰嗦，我将其称为因果行为方法或框架。这个框架有一个额外的好处：它同样适用于实验数据和历史数据，同时利用它们之间的差异。这与传统的分析方法形成对比，传统方法使用完全不同的工具处理它们（例如，实验数据使用ANOVA和T检验）；数据科学则不会区别对待实验数据和历史数据。

# 本书适合人群

如果你在使用R或Python分析业务数据，那么这本书适合你。我用“业务”这个词比较宽泛，指的是任何以正确的洞察力和可操作的结论推动行动为重点的盈利、非盈利或政府组织。

在数学和统计背景方面，无论你是业务分析师制定月度预测，还是UX研究员研究点击行为，或者是数据科学家构建机器学习模型，都无关紧要。这本书有一个基本要求：你需要至少对线性回归和逻辑回归有些许了解。如果你理解回归，你就能跟上本书的论点，并从中获得巨大的收益。另一方面，我相信，即使是具有统计学或计算机科学博士学位的专家数据科学家，如果他们不是行为或因果分析的专家，也会发现这些材料是新的和有用的。

在编程背景方面，你需要能够阅读和编写R或Python代码，最好两者都会。我不会教你如何定义函数或如何操作数据结构，比如数据框或pandas。已经有很多优秀的书在做这方面的工作，比我做得更好（例如，[*Python数据分析*](https://www.oreilly.com/library/view/python-for-data/9781491957653/) 作者Wes McKinney（O'Reilly）和 [*R数据科学*](https://www.oreilly.com/library/view/r-for-data/9781491910382/) 作者Garrett Grolemund和Hadley Wickham（O'Reilly））。如果你读过这些书，参加过入门课程，或者在工作中至少使用过其中一种语言，那么你就有能力学习这里的内容。同样地，我通常不会展示和讨论书中用于创建众多图表的代码，尽管它将出现在书的GitHub中。

# 本书不适合人群

如果你在学术界或需要遵循学术规范（例如，制药试验）的领域，这本书可能仍然对你有兴趣，但我描述的方法可能会让你与你的导师/编辑/经理产生冲突。

本书*不*概述传统行为数据分析方法，比如T检验或方差分析（ANOVA）。我还没有遇到过回归比这些方法在回答业务问题上更有效的情况，这就是为什么我故意将本书限制在线性和逻辑回归上的原因。如果你想学习其他方法，你需要去别处寻找（例如，[*使用Scikit-Learn、Keras和TensorFlow进行机器学习实践*](https://www.oreilly.com/library/view/hands-on-machine-learning/9781492032632/)（O'Reilly）由Aurélien Géron撰写的机器学习算法）。

在应用设置中理解和改变行为需要数据分析和定性技能两者。本书主要集中在前者，主要出于空间考虑。此外，已经有许多优秀的书籍涵盖了后者，比如理查德·塞勒斯坦（Richard Thaler）和卡斯·桑斯坦（Cass Sunstein）的《*推动：改善有关健康、财富和幸福的决策*》（Penguin）以及斯蒂芬·温德尔（Stephen Wendel）的[*为行为变革设计：应用心理学和行为经济学*](https://www.oreilly.com/library/view/designing-for-behavior/9781492056027/)（O'Reilly）。尽管如此，我将介绍行为科学的概念，以便你即使对这个领域还不熟悉，也能应用本书中的工具。

最后，如果你完全是新手对R或Python的数据分析，这本书不适合你。我建议你从一些优秀的介绍书籍开始，比如本节中提到的一些书籍。

# R和Python代码

为什么要用R *和* Python？为什么不用两者中更优秀的那个？“R与Python”的辩论在互联网上仍然存在，并且如我所知，大多数时候这种争论是无关紧要的。事实是，你将不得不使用你的组织中使用的任何一种语言。我曾在一家医疗公司工作过，由于历史和法规的原因，SAS是主导语言。我经常使用R和Python进行自己的分析，但由于无法避免处理遗留的SAS代码，我在那里的第一个月就学会了我需要的SAS的基础知识。除非你的整个职业生涯都在一家不使用R或Python的公司，否则你最终可能会掌握这两者的基础知识，所以你最好接受双语能力。我还没有遇到过任何人说“学习阅读[另一种语言]的代码是浪费时间”。

假设你很幸运地在一个同时使用这两种语言的组织中工作，你应该选择哪种语言呢？我认为这实际取决于你的环境和你需要做的任务。例如，我个人更喜欢用 R 进行探索性数据分析（EDA），但我发现 Python 在网页抓取方面更容易使用。我建议根据你工作的具体情况选择，并依赖最新的信息：两种语言都在不断改进，过去某个版本的 R 或 Python 可能不适用于当前版本。例如，Python 正在成为比以前更友好的EDA环境。你的精力最好花在学习这两种语言上，而不是在论坛上寻找哪种更好。

## 代码环境

在每章的开头，我将指出需要专门加载的 R 和 Python 包。此外，整本书我还会使用一些标准包；为避免重复，它们只在这里提及（它们已经包含在 GitHub 上所有脚本中）。你应该始终从它们开始编写你的代码，以及一些参数设置：

```py
## R
library(tidyverse)
library(boot) #Required for Bootstrap simulations
library(rstudioapi) #To load data from local folder
library(ggpubr) #To generate multi-plots

# Setting the random seed will ensure reproducibility of random numbers
set.seed(1234)
# I personally find the default scientific number notation (i.e. with 
# exponents) less readable in results, so I cancel it 
options(scipen=10)

## Python
import pandas as pd
import numpy as np
import statsmodels.formula.api as smf
from statsmodels.formula.api import ols
import matplotlib.pyplot as plt # For graphics
import seaborn as sns # For graphics
```

## 代码约定

我在 RStudio 中使用 R。R 4.0 在我写这本书的时候发布了，我已经采用了它，以尽可能保持书籍的更新。

R 代码是用代码字体编写的，并带有指示使用的语言的注释，就像这样：

```py
## R
> x <- 3
> x
[1] 3
```

我在 Anaconda 的 Spyder 中使用 Python。关于 “Python 2.0 vs. 3.0” 的讨论希望现在已经结束了（至少对于新代码来说；旧代码可能是另一回事），我将使用 Python 3.7。Python 代码的约定与 R 类似：

```py
## Python
In [1]: x = 3
In [2]: x
Out[2]: 3
```

我们经常会查看回归的输出。这些输出可能非常冗长，并包含了很多本书论点无关的诊断信息。在现实生活中，你不应忽略它们，但这是其他书更好涵盖的问题。因此，我会像这样缩写输出：

```py
## R
> model1 <- lm(icecream_sales ~ temps, data=stand_dat)
> summary(model1)

...
Coefficients:
             Estimate Std. Error t value Pr(>|t|)    
(Intercept) -4519.055    454.566  -9.941   <2e-16 ***
temps        1145.320      7.826 146.348   <2e-16 ***
...

```

```py
## Python 
model1 = ols("icecream_sales ~ temps", data=stand_data_df)
print(model1.fit().summary())

...     
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
Intercept  -4519.0554    454.566     -9.941      0.000   -5410.439   -3627.672
temps       1145.3197      7.826    146.348      0.000    1129.973    1160.666
...
```

## 函数式编程入门

作为程序员从初学者到中级水平的其中一步，就是停止编写代码的长串指令脚本，而是将代码结构化为函数。在本书中，我们将跨章节编写和重复使用函数，例如以下内容来构建 Bootstrap 置信区间：

```py
## R
boot_CI_fun <- function(dat, metric_fun, B=20, conf.level=0.9){

  boot_vec <- sapply(1:B, function(x){
    cat("bootstrap iteration ", x, "\n")
    metric_fun(slice_sample(dat, n = nrow(dat), replace = TRUE))})
  boot_vec <- sort(boot_vec, decreasing = FALSE)
  offset = round(B * (1 - conf.level) / 2)
  CI <- c(boot_vec[offset], boot_vec[B+1-offset])
  return(CI)
}

```

```py
## Python 
def boot_CI_fun(dat_df, metric_fun, B = 20, conf_level = 9/10):

  coeff_boot = []

  # Calculate coeff of interest for each simulation
  for b in range(B):
      print("beginning iteration number " + str(b) + "\n")
      boot_df = dat_df.groupby("rep_ID").sample(n=1200, replace=True)
      coeff = metric_fun(boot_df)
      coeff_boot.append(coeff)

  # Extract confidence interval
  coeff_boot.sort()
  offset = round(B * (1 - conf_level) / 2)
  CI = [coeff_boot[offset], coeff_boot[-(offset+1)]]

  return CI
```

函数还有一个额外的优势，即限制理解上的溢出：即使你不理解前面的函数如何工作，你仍然可以认为它们返回置信区间并遵循其余推理的逻辑，推迟到以后深入了解它们的代码。

## 使用代码示例

附加材料（代码示例等）可在 [*https://oreil.ly/BehavioralDataAnalysis*](https://oreil.ly/BehavioralDataAnalysis) 下载。

如果您有技术问题或在使用代码示例时遇到问题，请发送电子邮件至 [*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com)。

本书旨在帮助您完成工作。一般来说，如果本书提供了示例代码，您可以在自己的程序和文档中使用它。除非您复制了代码的大部分，否则无需联系我们寻求许可。例如，编写一个使用本书多个代码片段的程序并不需要许可。销售或分发O'Reilly书籍中的示例需要许可。引用本书并引用示例代码来回答问题不需要许可。将本书中大量的示例代码整合到您产品的文档中需要许可。

我们感谢但不需要署名。署名通常包括标题、作者、出版商和ISBN。例如：“*R和Python的行为数据分析*，作者Florent Buisson（O’Reilly）。版权2021 Florent Buisson，978-1-492-06137-3。”

如果您觉得您使用的代码示例超出了合理使用范围或上述许可，请随时通过[*permissions@oreilly.com*](mailto:permissions@oreilly.com)联系我们。

# 导航本书

本书的核心思想是，有效的数据分析依赖于数据、推理和模型之间的不断交流。

+   真实世界中的实际行为及相关的心理现象，如意图、思想和情绪

+   因果分析，特别是因果图

+   数据

本书分为五个部分：

[第一部分，*理解行为*](part01.xhtml#understanding_behaviors)

本部分通过因果行为框架和行为、因果推理和数据之间的联系来铺设舞台。

[第二部分，*因果图与去混淆*](part02.xhtml#causal_diagrams_and_deconfounding)

这部分介绍了混淆概念，并解释了因果图如何帮助我们解决数据分析中的混淆问题。

[第三部分，*鲁棒数据分析*](part03.xhtml#robust_data_analysis)

在这里，我们探讨了处理缺失数据的工具，并介绍了Bootstrap模拟，因为在本书的其余部分中我们将广泛依赖Bootstrap置信区间。小型、不完整或形状不规则的数据（例如具有多个高峰或异常值的数据）并非新问题，但在行为数据中可能尤为突出。

[第四部分，*设计和分析实验*](part04.xhtml#designing_and_analyzing_experiments)

在这一部分，我们将讨论如何设计和分析实验。

[第五部分，*行为数据分析的高级工具*](part05.xhtml#advanced_tools_in_behavioral_data_analy)

最后，我们综合一切来探讨中介效应、调节效应和工具变量。

本书的各个部分在一定程度上相互依赖，因此我建议至少在第一遍阅读时按顺序阅读它们。

# 本书使用的惯例

本书中使用以下排版惯例：

*斜体*

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`等宽字体`

用于程序清单，以及在段落中引用程序元素，如变量或函数名称，数据库，数据类型，环境变量，语句和关键字。

**`等宽粗体`**

显示用户应该按原样键入的命令或其他文本。

*`等宽斜体`*

显示应由用户提供值或由上下文确定值的文本。

###### 提示

这个元素表示提示或建议。

###### 注意

这个元素表示一般性说明。

###### 警告

这个元素指示警告或注意事项。

# 奥莱利在线学习

###### 注意

40多年来，[*奥莱利媒体*](http://oreilly.com)一直提供技术和商业培训，知识和见解，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专长。奥莱利的在线学习平台让您随需应变地访问直播培训课程、深度学习路径、交互式编码环境，以及奥莱利和其他200多家出版商的大量文本和视频内容。更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送至出版商：

+   奥莱利媒体，公司

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

您可以访问本书的网页，其中列出了勘误表，示例和附加信息，网址为[*https://oreil.ly/Behavioral_Data_Analysis_with_R_and_Python*](https://oreil.ly/Behavioral_Data_Analysis_with_R_and_Python)。

发送电子邮件至[*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com)以评论或询问关于本书的技术问题。

有关我们的书籍和课程的新闻和信息，请访问[*http://oreilly.com*](http://oreilly.com)。

在Facebook上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在Twitter上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

在YouTube上观看我们：[*http://youtube.com/oreillymedia*](http://youtube.com/oreillymedia)

# 致谢

作者们经常感谢他们的配偶对他们的耐心，并特别感谢那些有深刻见解的审阅者。我很幸运，这两者都集中在同一个人身上。我想没有其他人敢于或能够如此多次地把我送回绘图板，而这本书因此而大为改进。因此，我首先感谢我的生活和思想伴侣。

我的几位同事和行为科学家朋友慷慨地抽出时间阅读和评论了早期草稿。这本书因此变得更加出色。谢谢（按字母顺序逆序排列）Jean Utke、Jessica Jakubowski、Chinmaya Gupta和Phaedra Daipha！

特别感谢Bethany Winkel在写作中的帮助。

现在，我对最初的草稿有多么粗糙和令人困惑感到后悔。我的开发编辑和技术审阅人员耐心地推动我一路走到现在这本书的成就，分享他们丰富的视角和专业知识。感谢Gary O’Brien，感谢Xuan Yin，Shannon White，Jason Stanley，Matt LeMay和Andreas Kaltenbrunner。
