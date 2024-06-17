# 第 10 章\. 展示您的数据

在我们已经投入访问、评估、清理、转换、增强和分析数据的所有努力之后，我们终于到达了一个阶段，即准备开始考虑向他人传达我们所学到的东西。无论是为了向同事进行正式演示，还是为了向朋友和追随者发布社交媒体帖子，通过我们的数据整理工作生成的见解分享是一个机会，使我们的工作能够超越个人产生影响。

就像我们数据整理过程的每个部分一样，有效和准确地传达我们的见解涉及应用少数一成不变的规则，但更多的是判断力。这当然适用于书面沟通，但当涉及到数据沟通的方面时，可能更是如此，因为这部分通常引起最多关注：*可视化*。

正如我们在[“数据分析可视化”](ch09.html#vizforanalysis)中提到的，为了有效地与他人分享我们的数据见解，创建可视化图表需要与生成这些见解时的焦点和方法不同的关注点。例如，除非您试图接触*非常*专业的受众（比如通过学术出版物），否则直方图在您分享研究发现时不太可能进入您的可视化词汇表中。同时，*极有可能*您会使用柱状图或柱形图的某种形式^([1](ch10.html#idm45143395782464))来与非专业人士分享您的见解，因为这些广泛使用且高度易读的图形形式对大多数观众来说相对容易准确解释。

换句话说，我们选择用来视觉呈现我们数据发现的方式，不仅应受我们拥有的（或用来得出结论的）*数据*的启发，还应受我们试图接触的*受众*的影响。许多软件包（包括我们已经使用过的一些软件包）如果您只需指向一个结构化数据集并传入几个可选参数，它们将乐意生成图表和图形。虽然这在最低限度上可以产生可视化效果，但其输出更像是机器翻译而不是诗歌。是的，在高层次上它可能符合语言的“规则”，但其意义的清晰度（更不用说其有效性或*雄辩性*）是值得怀疑的。因此，利用可视化使我们的数据见解真正可访问他人，需要仔细考虑*您想要传达的内容*，*哪种视觉形式最适合*以及*如何*能够根据您的特定需求进行定制。

在本章的过程中，我们将依次涵盖每一个任务，从一些旨在帮助您识别数据发现中关键点的策略开始。之后，我们将回顾数据最常见（并且最有用！）的可视形式。在每种情况下，我们将讨论使用它们的规则和最佳实践，以及使用*seaborn*和*matplotlib*库在Python中渲染它们的基本代码。最后，我们将以真实数据的基本可视化为例，逐步介绍如何定制和完善其各个元素，以将（通常）可用的默认呈现转变为准确*且*吸引人的东西。在此过程中，我希望您能接触到至少几种新工具和方法，这些工具和方法将帮助您更加批判性地思考数据可视化——无论它是您自己数据处理努力的结果与否。

# 视觉修辞的基础

正如我已经多次提到的，编写优秀代码的过程在大多数其他情况下与写作一样。例如，在[第8章](ch08.html#chapter8)中，我们花了时间修订和重组代码，虽然它已经*运行*，但最终演变成了更清晰、更干净和更可重复使用的脚本和函数。从高层次来看，这个过程与您修改文章或论文的方式并没有太大不同：一旦您把所有关键想法收集到一起，稍后可以回到这篇文章，看看如何重新措辞和重新组织，使文章更为简洁，概念更为逻辑。

尽管相同的写-编辑-润色循环适用于数据可视化，但我们正在处理的单元与文章非常不同，更像一个*段落*——因为一般来说，一个可视化应该用来传达一个*单一的*思想。无论您的可视化是印刷版还是数字版，静态还是交互式，无论它是长篇演讲的一部分还是将成为独立的社交媒体帖子。一个可视化 = 一个关键思想。

我现在强调这一点，因为如果你来到这一章希望看到如何构建[Gapminder风格](https://www.google.com/publicdata/directory)的互动图表或详细的[流图](https://flowingdata.com/2008/02/25/ebb-and-flow-of-box-office-receipts-over-past-20-years)，我想要立即让你失望：在这一章中，我的重点将放在最常用的可视化类型上，如柱状图、折线图和条形图。部分原因是因为它们仍然是表示只有一个自变量的数据最简单和最可解释的方式，而这正是你大多数时候应该尝试呈现的。是的，更复杂的可视化*可以*用于绘制多个自变量——最初的Gapminder可视化是一个很好的例子——但没有[一位可爱的瑞典男子实时指导观众](https://youtube.com/watch?v=jbkSRLYSojo)，它们更像是漂亮的玩具而不是信息工具。这就是为什么我们这里的重点将放在将可访问的视觉形式精炼为我喜欢称之为*流畅*图形的东西上——像最好的文本一样，清晰、简单和易于访问地传达信息。虽然制作流畅的图形并不排除视觉复杂性甚至交互性，但它*确实*要求视觉的每个方面都有助于图形的清晰度和意义。

实现这种视觉流畅性意味着在三个主要阶段考虑数据可视化：

1\. 优化你的关注点

你究竟想传达什么？在某些方面，这类似于选择你的原始数据处理问题的过程：无论你的处理和分析揭示了什么，你都需要在每个可视化中传达*一个*想法。你如何知道自己是否有效地完成了这个任务？大多数情况下，这意味着你能够用*一个句子*表达它。与你早期的数据处理问题一样，你制定的数据陈述将作为制定可视化选择的“基本事实”。任何有助于你更清晰地传达你的想法的东西都会被保留；*其他所有东西*都会被删除。

2\. 寻找适合的视觉形式

你的数据最适合以柱状图还是折线图显示？是地图吗？饼状图？散点图或气泡图？确定数据的最佳视觉形式总是需要一些试验。与此同时，选择一种视觉形式来表达你的数据陈述不仅仅是偏好或口味的问题；关于某些类型的数据和数据关系如何视觉编码有一些不容置疑的规则。是的，美学确实在可视化的效果中起着作用，但它们不能取代准确性的需要。

3\. 提升清晰度和意义

即使确定了主要的视觉形式，还有许多方法可以改善或降低您的可视化的清晰度、可访问性、视觉吸引力和雄辩性。至少，您需要在颜色、图案、比例尺和图例以及标签、标题和注释之间做出决策。如果您的数据陈述特别复杂，您需要仔细地增加更多的视觉结构来捕捉这些细微差别，例如误差条或不确定性范围，或者可能是预测和/或缺失的数据。

在接下来的几节中，我们不仅会概念性地讨论每个阶段，还会利用实际数据来看看它们在实践中如何使用Python。

# 作出你的数据陈述

许多年前，我有幸邀请到《纽约时报》的[Amanda Cox](https://en.wikipedia.org/wiki/Amanda_Cox)作为我的数据可视化课程的特邀讲师，她分享了一个评估特定数据陈述是否适合可视化的绝佳提示：“如果你的标题中没有动词，那就有问题。”

浅尝辄止的话，当然，这个要求很容易满足。^([2](ch10.html#idm45143395497264)) 然而，她的陈述精神暗示了一种更为严格的要求：你的图形标题应清晰地表达某种重要的关系或主张，并且支持证据应该在图形本身可见。为什么这么重要？首先，将你的主张直接放在标题中会鼓励读者首先*看*你的图形；清晰地命名帮助确保观众确实会——字面上——知道*在哪里*寻找这些主张的支持证据。当然，作为信息设计师，我们的工作是确保我们所有图形的视觉线索也这样做，但往往[正是标题吸引了人们的注意](https://psychologicalscience.org/news/how-headlines-change-the-way-we-think.html)。

如果你简单地不能在你的图形标题中找到一个动作动词，这是一个提示，即可视化可能*不*是传达你的见解的最佳方式。诚然，在适当的情况下，人类可以非常迅速地处理可视化，但只有当可视化有所“表达”时才能实现这种优势。换句话说，虽然你可能能够生成一个标题为“一年的每日国库长期利率”的视觉上准确的图表，但现实是，即使是最大的政策狂热者也会想知道他们为什么要费心去看它。如果不是正确的工具，不要坚持可视化！记住，我们的目标是尽可能有效地传达我们的数据整理见解，而不是不惜一切代价地以视觉方式表达它们。通过首先专注于完善你的数据陈述，并确认它具有你需要的力量，你将避免花费大量时间设计和构建一个实际上并不做你想要的事情或你的观众需要的可视化。当然，基本的数据可视化可以使用数据集和一个强大的可视化库（如*seaborn*）快速生成。但是，制作真正*雄辩的*可视化需要仔细考虑以及对甚至最好的库的默认图表进行详细定制。因此，在你投入所有的时间和精力之前，确保你的复杂可视化不是从你的分析中提取的一个突出的统计数字会更好。

一旦你在那里确立了一个强有力的数据陈述“标题”，那么，现在是时候确定哪种图形形式将帮助你最有效地呈现数据证据来支持你的主张了。

# 图表、图形和地图：哦，我的天！

即使我们限制在更为直接的图形形式上，仍然有足够的选项可以使我们在找到最适合我们数据的最佳图形的过程中感到有点不知所措。你应该选择折线图还是条形图？如果条形图是最佳选择，它应该是水平的还是垂直的？饼图*有时*可以吗？不幸的是，这是一个情况，我们的 Python 库在这方面基本上是*无法*帮助我们的，因为通常它们只会英勇地尝试根据您提供的数据生成您要求的任何类型的图表。我们需要一个更好的方法。

这就是一个有着明确的数据陈述的重要性。您的陈述是否涉及*绝对*值，比如我们PPP贷款数据中的`CurrentApprovalAmount`，还是是否侧重于值之间的*关系*，就像[“大流行使年度外国直接投资流量减少了三分之一”](https://economist.com/graphic-detail/2021/06/21/the-pandemic-cut-annual-fdi-flows-by-one-third)一样？虽然关于*绝对*值的声明通常最好通过条形图来表达，但关于关系的数据陈述可以通过更广泛的视觉形式来很好地支持。例如，如果您的数据陈述涉及随时间变化，那么折线图或散点图就是一个很好的开始。同时，一些视觉形式，如饼图和地图，很难适应除了单个时间点的数据之外的任何东西。

事实上，对于数据可视化，几乎没有硬性规则[^3]，但我已经在接下来的部分中概述了存在的规则——以及一些关于设计图形的一般提示。虽然这些准则将帮助您选择一种不会逆向您的数据的可视化形式，但这只是下一步。真正提升您的图形的选择是我们将在[“优美视觉元素”](#elementsofeloquentvisuals)中讨论的选择。在那一部分中，我们将超越（仍然非常出色的）*seaborn*的默认设置，开始更深入地研究*matplotlib*，以控制标签、颜色和注释等可以真正使您的可视化脱颖而出的内容。

## 饼图

饼图在可视化中是一个令人意外的极具争议性的话题。尽管饼图对[教授儿童关于分数的知识](https://pbs.org/parents/recipes/pegs-pizza-fractions)很有帮助，但有很多人认为它们在有效的可视化词汇中几乎没有[任何位置](https://storytellingwithdata.com/blog/2011/07/death-to-pie-charts)。

我个人认为，有特定的情况——虽然有限——在这些情况下，饼图是支持您的数据陈述的最佳可视化方式。例如，如果您试图阐明您的数据中有多少*比例*或*份额*具有特定值，并且其余值可以明智地分为四个或更少的类别，那么饼图可能是您想要的。特别是如果生成的图表突出显示与整体的一些[“可识别”](https://store.moma.org/for-the-home/kitchen-dining/cookware-kitchen-tools/visual-measuring-cups/8711-802262.html)部分对应的值（例如 1/4、1/3、1/2、2/3 或 3/4），由于人眼能够在[不费力](https://csc2.ncsu.edu/faculty/healey/PP)的情况下检测到这些差异，因此这一点尤为真实。

例如，查看 [2021 年 6 月纽约市民主党市长初选的结果](https://washingtonpost.com/elections/election-results/new-york/nyc-primary)，我们可以想象编写一个数据声明，如“尽管候选人众多，前三名候选人几乎占据了四分之三的第一选择票。”由于只有四名候选人获得了超过10%的第一选择票，因此将所有其余候选人归为单一的“其他”类别也是合理的。在这种情况下，饼图是准确呈现结果并支持我们主张的完全合理的方式之一，部分原因是因为它使得我们可以轻松地看到领先者显著超越其他候选人。

鉴于饼图的争议性质，一般而言相当灵活的*seaborn*库中并没有提供饼图选项并不完全令人意外。然而，我们可以直接使用*matplotlib*来获得一个非常实用的饼图。^([4](ch10.html#idm45143395804112)) 不过，在*matplotlib*的饼图功能中还存在一些特殊之处需要克服。例如，最佳实践要求饼图从最大部分开始，即“12点钟”位置，其余部分按顺时针方向以降序添加。然而，在*matplotlib*中，第一部分从“3点钟”位置开始，并以逆时针方向添加其他部分。因此，我们需要指定 `startangle=90` 并反转片段的从大到小的顺序。^([5](ch10.html#idm45143395798752)) 同样，在*matplotlib*中，默认情况下，饼图的每个“片段”被分配了不同的颜色色调（例如紫色、红色、绿色、橙色和蓝色），这在某些类型的色盲人士中可能无法访问。由于我们的数据声明在概念上将前三名候选人分组，因此我将它们都设置为相同的绿色阴影；并且由于*所有*的候选人都来自同一政党，我将所有的片段保持在绿色系列中。要查看如何编码这种类型的图表（包括这些小的自定义），请参阅[例子 10-1](#a_humble_pie) 和在[图 10-1](#nyc_primary_pie) 中的结果可视化。

##### 例子 10-1\. a_humble_pie.py

```py
import matplotlib.pyplot as plt

# matplotlib works counterclockwise, so we need to essentially reverse
# the order of our pie-value "slices"
candidate_names = ['Adams', 'Wiley', 'Garcia', 'Yang', 'Others']
candidate_names.reverse()
vote_pct = [30.8, 21.3, 19.6, 12.2, 16.1]
vote_pct.reverse()

colors = ['#006d2c','#006d2c', '#006d2c', '#31a354','#74c476']
colors.reverse()

fig1, ax1 = plt.subplots()
# by default, the starting axis is the x-axis; making this value 90 ensures
# that it is a vertical line instead
ax1.pie(vote_pct, labels=candidate_names, autopct='%.1f%%', startangle=90,
        colors=colors) ![1](assets/1.png)
ax1.axis('equal')  # equal aspect ratio ensures that pie is drawn as a circle.

# show the plot!
plt.show()
```

[![1](assets/1.png)](#co_presenting_your_data_CO1-1)

我们传递给`autopct`的参数应该是“格式化字符串字面量”，也称为[*f-string*](https://docs.python.org/3/tutorial/inputoutput.html#formatted-string-literals)。这个例子指定将分数表达为*浮点数*（小数点）到一位精度。双百分号符(`%%`)在这里用于打印输出中的一个单个百分号符号（通过用另一个百分号符号转义保留的百分号符号）。

![纽约市初选饼图。](assets/ppdw_1001.png)

###### 图 10-1\. 纽约市初选饼图

总结一下，如果您考虑使用饼图：

规则

你的数据类别在概念上（字面上也是）必须总结为一个“整体”。

指南

类别数量应压缩到五个或更少。

指南

想要突出显示的数据比例应为总数的1/4、1/3、1/2、2/3或3/4。

然而，如果你的数据不符合其中一个或多个要求，条形图可能是探索的下一个图形形式。

## 条形图和柱状图

条形图通常是突出显示离散、名义（与比例相对的）数据值之间关系的最有效方式。与饼状图不同，条形图可以准确表示数据值不总结为单一“整体”的数据集。它们还可以有效地表示正负值（如果需要同时表示），并且可以显示不同类别数据和数据值随时间的变化。这些条可以垂直（有时这些图形被描述为*柱状*图）或水平排列，以使标签和数据关系更加清晰可读。

换句话说，条形图*极其*灵活，提供了多种选项来有效展示数据声明的证据。然而，在使用条形图时有*一个切实可行的规则*：*数据值必须从零开始*！尽管有些人试图这样做，但实际上并没有例外情况，详细规定见[这里](https://datajournalism.com/read/longreads/the-unspoken-rules-of-visualisation)。为什么这条规则如此重要？因为将图表的条形起点设置为一个*非零*数字意味着它们的长度视觉上的差异将不再与它们*实际*数值上的差异成比例。

例如，假设你在工作中为加薪辩护，目前的时薪为美国联邦最低工资标准每小时$7.25。你请求老板将你的小时工资提高到[$9.19](https://data.bls.gov/cgi-bin/cpicalc.pl?cost1=7.25&year1=200907&year2=202107)，以反映自2009年以来通货膨胀对最低工资的影响。

“好吧”，你的老板说，“让我们看看加薪会是什么样子”，然后向你展示了类似于[图 10-2](#inaccurate_comparison)所示的图表。

![一张不准确的工资比较图](assets/ppdw_1002.png)

###### 图 10-2\. 一张不准确的工资比较图

看出问题了吗？通过将条形图的起点设置在5而不是零，[图 10-2](#inaccurate_comparison)使得看起来$9.19每小时几乎*翻倍*了你当前的工资。但是简单的数学计算（9.19 - 7.25 / 7.25 = ~.27）表明，它只比你目前的收入多出25%多一点。正如你所见，将条形图的起点设置为零不是品味、美学或语义问题——这是[一种视觉谎言](https://flowingdata.com/2017/02/09/how-to-spot-visualization-lies)。

即使是专业的图形团队有时也会出错。来看看凯撒·丰格在他的博客《*垃圾图表*》中突出显示的这个例子，《“工作文化”》并在[图 10-3](#inaccurate_retirement)中复制。

![一个不准确的退休年龄比较](assets/ppdw_1003.png)

###### 图 10-3\. 一个不准确的退休年龄比较

在[图 10-3](#inaccurate_retirement)中，一个“破碎”的条形图声称显示了不同国家的男性停止工作的时间，与官方退休年龄相比。就像[图 10-2](#inaccurate_comparison)中一样，未从零开始的条形图严重地错误地表示了它们的真实价值差异：根据数据标签，法国男性的退休年龄比日本男性提前约10年 —— 工作年限相差15%。但是日本的实际条形图长度*超过法国的两倍*。当然，如果所有条形图都从零开始，它们的值之间的差异就不会显得如此引人注目。

是怎么回事？这个可视化的设计者真的想要欺骗我们认为日本男性工作的时间是法国男性的两倍吗？几乎可以肯定不是。像[图 10-3](#inaccurate_retirement)中的这样的图形是表达*一个*想法每次可视化是如此重要的原因：试图叠加多于一个想法的内容是最有可能导致问题并最终得到不准确和误导性可视化的地方。在[图 10-3](#inaccurate_retirement)中，设计者试图展示两种不同的测量方法（官方退休年龄与男性停止工作的年龄之间的差异*以及*该年龄是什么）。它们的刻度是不兼容的。将条形图从零开始，读者将无法区分点放置的年龄；改变刻度以使点的放置清晰可读，而条形图变得不准确。无论哪种方式，你都让读者做了很多工作 —— 尤其是因为图形的标题没有告诉他们他们应该寻找什么 ;-)

让我们看看当我们从一个清晰的动作动词标题开始，并使用它来重新设计这个图形时会发生什么：“日本的男性在退休年龄之后工作多年，而其他国家则远早于此。” 这里，我们的标题/数据陈述是关于突出官方退休和实际退休之间的*差异*。 现在我们可以设计一个水平条形图，既支持这一说法，*又*准确表示潜在数据，如[示例 10-2](#retirement_age)中所示和随后的[图 10-4](#retirement_gap_graphic)。

##### 示例 10-2\. retirement_age.py

```py
import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns
import numpy as np

# (abbreviated) list of countries
countries = ['Japan', 'Iceland', 'Switzerland', 'France', 'Ireland', 'Germany',
             'Italy', 'Belgium']

# difference in years between official and actual retirement age
retirement_gap = [9, 2, 2, -1, -2, -2, -7, -8]

# zip the two lists together, and specify the column names as we make the DataFrame
retirement_data = pd.DataFrame(list(zip(countries, retirement_gap)),
               columns =['country', 'retirement_gap'])

# in practice, we might prefer to write a function that generates this list,
# based on our data values
bar_colors = ['#d01c8b', '#d01c8b', '#d01c8b', '#4dac26','#4dac26','#4dac26',
              '#4dac26','#4dac26']

# pass our data and palette to the `seaborn` `barplot()` function
ax = sns.barplot(x="retirement_gap", y="country",
                 data=retirement_data, palette=bar_colors) ![1](assets/1.png)

# show the plot!
plt.show()
```

[![1](assets/1.png)](#co_presenting_your_data_CO2-1)

通过将我们的数值分配给 x 轴，并将分类值分配给 y 轴，*seaborn* 将呈现这个图表为一个水平而不是垂直的条形图。

![退休差距水平条形图。](assets/ppdw_1004.png)

###### 图 10-4\. 退休差距水平条形图

因为我的数据说明/标题现在明确涉及官方退休年龄与实际退休年龄之间的*差异*，我选择直接绘制这种差异，并重新排列数据：法国男性仅比官方退休年龄早退休一年，而比利时男性大约提前退休八年。为了进一步突出官方退休年龄前后的差异，我还根据条形的正负值对其进行了颜色编码。

此数据集中正负值的混合——以及更长的国家名称标签——使这个图表作为水平条形图比垂直条形图更易读。然而，为了比较起见，如果我们想将其作为垂直图表测试，我们需要交换我们传递给`barplot()`函数的数据列作为`x`和`y`参数。例如，通过更改如下内容：

```py
ax = sns.barplot(x="retirement_gap", y="country", data=retirement_data,
                 palette=bar_colors)
```

到此：

```py
ax = sns.barplot(x="country", y="retirement_gap", data=retirement_data,
                 palette=bar_colors)
```

尽管这种变化很容易实现，但在呈现数据集时，垂直或水平渲染的可读性可能存在真正的差异。具体来说，垂直条形图通常更适合具有较短标签、变化较小和/或几乎没有负值的数据，而水平条形图通常更适合差异较大的数据（特别是如果有大量负值）和/或具有较长标签的数据。

虽然[图 10-4](#retirement_gap_graphic)中显示的可视化仍然相当简单，但如果你运行代码，你会自己看到例如在调色板上下功夫会给可视化带来的质量差异。虽然我在这里选择了二进制品红/绿色编码，但我也可以指定`seaborn`的[170种调色板](https://medium.com/@morganjonesartist/color-guide-to-seaborn-palettes-da849406d44f)（例如，`palette='BuGn'`），这将（大部分地）使每个条形的颜色强度与其值对齐。

总结一下，在处理条形图时：

规则

条形图必须从零开始！

指导方针

垂直条形图适用于数据更密集、变化较小的情况。

指导方针

水平条形图更适合更多的变化和/或更长的标签。

## 折线图

当你的数据陈述关于*变化率*而不是*值差异*时，是时候探索折线图了。与条形图类似，折线图可以有效显示多个类别的数值数据，但只能显示它*随时间变化*的情况。然而，因为它们不直观地编码绝对数据值，折线图刻度*不*需要从零开始。

起初，这可能看起来像是一种操纵的邀请——事实上，折线图一直处于一些重大政治争议的中心。^([6](ch10.html#idm45143395440176)) 然而，对于条形图和折线图，真正驱动y轴刻度的是*数据*：正如我们不能决定将条形图从零开始，将y轴刻度扩展到最大数据测量的多倍是荒谬的，如[图 10-5](#bad_wage_math_flat)所示。

![另一个糟糕的工资比较图形](assets/ppdw_1005.png)

###### 图 10-5\. 另一个糟糕的工资比较图形

尽管在技术上是准确的，但[图 10-5](#bad_wage_math_flat) 中超长的y轴刻度已经将数据值压缩到我们的眼睛无法准确或有效区分它们的程度。因此，对于条形图，y轴的最高值通常应在下一个“整数”标记的增量处（更多关于此内容，请参见[“选择刻度”](#selecting_scales)）。对于线图，可视化专家如唐娜·王建议数据值的范围应占据y轴空间的大约三分之二。^([7](ch10.html#idm45143395430576))

当然，这种方法突出了线图中数据点的选择对整体信息传达的影响。例如，请考虑来自[*经济学人*](https://economist.com/graphic-detail/2021/06/21/the-pandemic-cut-annual-fdi-flows-by-one-third)的这张图，它在[图 10-6](#fdi_flow_original) 中重新制作。

![2007-2020 年外国直接投资流动](assets/ppdw_1006.png)

###### 图 10-6\. 2007–2020 年外国直接投资流动（FDI）

在这种情况下，原始标题“疫情使年度外国直接投资流动减少三分之一”实际上相当有效；它既积极又具体。但是，虽然这个标题描述的数据包含在随附的图表中，但并未得到强调——尽管所述变化发生在2019年至2020年之间。如果我们修改图形，仅侧重于这两年发生的情况，如[图 10-7](#fdi_slopegraph) 所示，我们既可以更清楚地支持数据陈述，*还*能揭示数据的另一个维度：尽管“发达”国家的外国直接投资大幅下降，“发展中”地区却基本保持稳定。正如文章本身所述，“向富裕国家的流入速度下降得比向发展中国家的速度要快得多——分别下降了58%和仅8%。”

这种两点线图，也称为*斜率图*，不仅使读者能够轻松看到标题声明背后的证据，还使他们能够推断出疫情对“发达”与“发展中”国家外国直接投资的不均影响——从而为文章后来的声明提供证据。正如您在[示例 10-3](#covid_fdi_impact) 中所看到的，生成这种基本线图只需几行代码。

![疫情使年度外国直接投资流动减少三分之一。](assets/ppdw_1007.png)

###### 图 10-7\. 疫情使年度外国直接投资流动减少三分之一

##### 示例 10-3\. covid_FDI_impact.py

```py
import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns
import numpy as np

# each individual array is a row of data
FDI = np.array([[0.8, 0.7], [0.3, 0.6]])

fdi_data = pd.DataFrame(data=FDI,
              columns=['Developed', 'Developing'])

ax = sns.lineplot(data=fdi_data)

# show the plot!
plt.show()
```

此时，你可能会想知道，仅包括两年数据是否在某种程度上是*错误*的。毕竟，我们手头上有（至少）十年的数据。

当然，真正的问题不在于我们*是否*有更多数据，而是我们所做的数据声明是否以某种方式误读了更广泛的趋势。查看 [图 10-6](#fdi_flow_original) 中的原始图表，显然FDI在过去15年左右只有两次如此*迅速*下降：分别是从2007年到2008年以及从2016年到2017年。为什么？我们不确定——无论是原始图表还是全文（我查过了！）都没有明确说明。我们*确实*知道的是价值的绝对变化（大约5000亿美元）和价值的比例变化足够*且*独特，以至于仅关注一年的变化并不具有误导性。如果我们想向读者保证这一点，最好在表格中提供额外的细节，他们可以在其中详细查看精确的数字，而不会被主要观点分散注意力。

当*变化率*（体现在每条线的斜率中）是数据声明信息的核心时，线图是必不可少的视觉形式。虽然这种类型的图表不需要从零开始，但它*只能*用于表示随时间变化的数据。回顾一下，在使用线图时，适用以下几点：

规则

数据点必须表示随时间的值。

指南

数据线应占垂直图表区域的大约2/3。

指南

四条或更少的线应当有明显的颜色/标签区分。

虽然起初可能看起来违反直觉，但在线图上有大量数据线其实是可以的——只要它们的样式不与我们数据声明的证据竞争。正如我们将在下一节看到的那样，这种“背景”数据实际上可以成为为读者提供额外背景的一种有用方式，从而更有效地支持您的主要声明。

## 散点图

虽然散点图在一般数据传播中并不常用，但它们作为线图的时间点对应物可以是不可替代的，尤其是当您有大量数据点来展示明显趋势或与该趋势偏离时。

例如，考虑一下[图 10-8](#nyt_temp_scatter) 中的图形，该图形重现了来自这篇[*纽约时报*故事](https://nytimes.com/interactive/2021/06/29/upshot/portland-seattle-vancouver-weather.html)中的一个图表，并说明了在40多年的时间里，从俄勒冈州波特兰市到加拿大温哥华市的城市中，2021年6月的三个连续日的最高温度远高于预期范围。虽然标题可以更加生动，但可视化本身传达了一个明确的信息：波特兰市2021年6月下旬的三天最高温度超过了过去40年里的每一天。

![1979-2021年波特兰市每日最高气温。](assets/ppdw_1008.png)

###### 图10-8\. 波特兰市1979年至2021年的每日最高气温

大多数情况下，散点图用于显示随时间（如[图 10-8](#nyt_temp_scatter)）或跨许多个体成员（例如“学校”、“大城市”、“河流供水的湖泊”或“漫威电影”）捕获的数据值。有时，散点图可能包括计算的趋势线，用作将个别数据点与“预期”平均值进行比较的基准；其他时候，基准可能是由专业、法律或社会规范确定的值。

例如，从一篇*Pioneer Press*关于某些学校的学生表现优于典型指标的报道中获得灵感，^([8](ch10.html#idm45143395284112))我们可以使用*seaborn*绘制加利福尼亚学校系统的历史数据，生成一个散点图并突出显示一个异常数据点。此示例的代码可以在[示例 10-4](#schools_that_work)中找到。

##### 示例 10-4\. schools_that_work.py

```py
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd

# import the school test data
school_data = pd.read_csv("apib12tx.csv")

# plot test scores against the percentage of students receiving meal support
sns.scatterplot(data=school_data, x="MEALS", y="API12B", alpha=0.6, linewidth=0) ![1](assets/1.png)

# highlight a high-performing school
highlight_school = school_data[school_data['SNAME'] == \
                               "Chin (John Yehall) Elementary"]
plt.scatter(highlight_school['MEALS'], highlight_school['API12B'],
            color='orange', alpha=1.0) ![2](assets/2.png)

# show the plot!
plt.show()
```

[![1](assets/1.png)](#co_presenting_your_data_CO3-1)

`alpha` 参数控制点的不透明度；60% 的不透明度（作为十进制 `0.6`）在这里被证明是视觉上清晰以及点重叠的热图效果的正确平衡。`linewidth=0` 参数消除了每个点周围的轮廓，这会影响调整透明度时的热图效果。

[![2](assets/2.png)](#co_presenting_your_data_CO3-2)

要“突出显示”学校，我们基本上只是在所选数据点的 x 和 y 坐标处创建一个单点散点图。

使用散点图的一个关键挑战是*遮挡*问题，即数据点可能重叠在一起，从而遮挡了数据的真实密度。对此的一种处理方法是添加*抖动*——对个别点的放置添加少量随机性，旨在最小化视觉上的重叠。然而，在*seaborn* 0.11.2 中，抖动被列为[可选参数](https://seaborn.pydata.org/generated/seaborn.scatterplot.html)，但被列为“不支持”。幸运的是，通过调整数据点的透明度或*alpha*，我们可以保留数据的精度而不失其可解释性。通过使可视化中的所有点都有些透明，重叠的数据点就形成了一种基于透明度的*热图*，这样就能清晰地展示趋势，而不失特异性，正如[图 10-9](#schools_that_work_scatter)所示。

![在某些学校，历史并非命运](assets/ppdw_1009.png)

###### 图 10-9\. 在某些学校，历史并非命运

那么，何时使用散点图才有意义？

指南

数据必须足够大量，以便能看到趋势和异常值。

指南

应可视化相关基准，无论是来自数据还是外部规则。

指南

大多数数据应该是“背景”颜色，高亮显示的点不应超过少数几个。

指南

调整透明度或应用抖动以最小化数据遮挡。

## 地图

对于我们许多人来说，地图是最熟悉的可视化类型之一；根据你的情况，你可能使用地图规划上班或上学的路线，找到新的商店或餐馆，或者找到新的公园或自行车道。地图也是大众传播中常见的视觉形式，它们出现为天气地图、选举地图，甚至是提供陌生地点参照的“定位器”地图。如果我们的数据具有地理组成部分，那么考虑将其制成地图是很自然的。

然而，在现实中，*除非你的数据是*关于*地理，否则你真的不应该将其映射*。为什么？因为地图代表*土地面积*。不是流行度，甚至不是人口—而通常我们的数据就是关于人口的。例如，让我们回想一下来自[第 6 章](ch06.html#chapter6)的 PPP 贷款数据。如果你使用 `value_counts('ProjectState')` 对州的批准贷款数量进行聚类，你将得到以下输出（重新格式化为列以节省空间）：

```py
CA    99478      VA    18682      CT    10197      NH     4197      AK     2076
TX    60245      NC    18022      AL     9025      ID     3697      PR     2032
NY    54199      MN    16473      OK     8598      NM     3524      VT     1918
FL    46787      CO    15662      SC     8522      ME     3490      WY     1791
IL    33614      MD    15170      UT     7729      HI     3414      GU      305
PA    30768      WI    14729      KY     7623      DC     3175      VI      184
OH    26379      IN    13820      IA     7003      RI     3012      MP       55
NJ    24907      MO    13511      KS     6869      WV     2669      AS       18
MI    24208      TN    12994      NV     6466      MT     2648
MA    21734      AZ    12602      NE     4965      ND     2625
GA    20069      OR    10899      AR     4841      DE     2384
WA    18869      LA    10828      MS     4540      SD     2247
```

毫不费力地，你可能已经猜到这个表格中州的排列顺序与[这个表格](https://en.wikipedia.org/wiki/List_of_U.S._states_and_territories_by_population)中州的排列顺序相似，后者按人口排名州。换句话说，如果我们要对 PPP 贷款批准的数据进行“映射”，那么它基本上会成为一个人口地图。但是让我们假设我们解决了这个问题，并且我们通过人口对贷款数量进行了标准化，因此生成了一个新列，称为“人均批准贷款”或类似的内容。即使现在我们已经将数据从人口转换为流行度，但我们实际上只是用地理形式给自己提出了条形图问题：特定州实际占据的*视觉区域*与我们显示的数据*不*成比例。无论我们选择什么颜色搭配或数据范围，特拉华州的可视空间都将只占怀俄明州的1/50，尽管它批准的贷款数量多出25%。通过对此进行映射，我们只是在确保我们的视觉表现与实际数据相悖。

显然，有*很多*地图可视化，其中相当多的地图可能存在我迄今为止强调的一个或两个错误。地图提供了一个*看似*简单直接的视觉组织原则，许多人无法抵制它，但将其用于非地理数据实际上对数据和读者都是一种伤害。

当然，真正地理现象的优秀地图是存在的：例如，这些*纽约时报*的人口普查地图，提供了一种思慮周到的方法来展示[census data](https://nytimes.com/interactive/2015/07/08/us/census-race-map.html)，而*ProPublica*关于[休斯顿洪水区](https://projects.propublica.org/graphics/harvey-maps)的工作则说明了地理（自然和人造）在极端天气事件中的重要性。至于关于风数据的美丽和独特呈现，请查看[hint.fm上的这个风图](http://hint.fm/wind)。

考虑到地图很少是支持数据主张的*正确*可视化方式（以及在Python中构建它们可能有多复杂），我们在这里不会展示映射的代码示例。如果您正在处理您认为必须映射的数据，我建议看看[*geopandas*库](https://geopandas.org/index.html)，该库旨在轻松地将与*pandas*数据框相关的映射相关形状信息结合起来生成可视化。

# 优雅视觉的要素

尽管我大部分职业生涯都是信息设计师，但我并不认为自己是真正的图形设计师。我不能为你的业务做一个好的标志，就像你学会了一些Python编程后不能解决打印机问题一样。幸运的是，通过学习、大量阅读、少数课程以及许多才华横溢的设计师的慷慨帮助，我对有效的视觉设计有了一定了解，特别是信息设计。我在工作中学到的一些要点在这里概述。

## “挑剔”的细节确实有所不同

一百万年前，当我第一次作为网络初创公司的前端程序员工作时，我的首要任务是制作*运行良好*的东西——让我告诉你，当它们真正运行时，我是*非常*高兴的。随着时间的推移，我甚至在一些与我们在[第8章](ch08.html#chapter8)中进行的相同方式中对我的代码进行了改进和重构，并且我对这些编程工件相当满意。从程序员的角度来看，我的工作是相当干净的。

但我在一个*设计*团队工作，我与之合作的设计师总是推动我微调看起来对我而言更像“很好有”，而不是必要的小细节。如果幻灯片中的照片稍微减慢或者弹回一点，这真的有关键吗？当时，编写和调整这样的效果意味着我需要写入和调整（非常粗略的）物理方程，这不是我喜欢的。而且，所有这些定制都在我代码中制造了混乱。

但我最喜欢编程的原因之一是解决问题，所以最终我埋头进行了他们要求的更改。一旦我看到它运行起来，我意识到最终结果变得更加精致和令人满意。我开始赞赏设计的*质量*在那些“小”细节中，并且那些“挑剔”的东西——比如点击或轻敲时的颜色变化——实际上使得我的界面和图形更加清晰和可用。换句话说，不要忽视设计中的“细节”——它们不仅仅是为了让事物“看起来漂亮”。*设计的本质在于它的功能*。

## 相信你的眼睛（和专家们）

在数字化的视觉元素中，必然以定量术语表达：[十六进制颜色代码](https://computerhope.com/htmcolor.htm)的数学起源和x/y坐标定位可能使人误以为为图表找到“正确”的颜色或标注标签的正确位置只是数学问题。但事实并非如此。例如，颜色感知既复杂又高度个体化——我们不能确保别人看到的是我们“相同”的颜色——当然，有许多类型的颜色“盲”可能会阻止某些人根本无法感知某些对比色对（例如红/绿，蓝/橙，黄/紫）。没有方程可以解释这一点。

实际上，我们可能使用的每一种“颜色”实际上都由三个不同的属性来表征：色调（例如红色与蓝色），亮度（或光度）和饱和度。彩色“搭配”通过这些特征以非常特定的方式对齐或对比。当然，当涉及到可视化时，我们需要颜色做的远不止是看起来好看；它们必须*有意义地编码*信息。但是一个颜色“比另一个更蓝20%”是什么意思？不管怎样，这不仅仅是在你的十六进制颜色值中翻转一些数字的问题。^([9](ch10.html#idm45143394216688))

幸运的是，我们得到了专家的帮助。二十多年来，寻找颜色建议的人（主要是地图，虽然它也是其他类型图表的一个很好的起点）一直可以求助于宾州州立大学地理学教授兼制图师[Cynthia Brewer](http://personal.psu.edu/cab38)的工作，她的[ColorBrewer](https://colorbrewer2.org)工具提供了出色的免费视觉设计颜色分布。同样，Dona Wong的优秀著作*《华尔街日报信息图表指南》*（Norton）包括了我个人喜欢的一些图形颜色组合。

如果您非常坚决地想选择自己的颜色调色板，那么下一个最佳方法就是求助于我们拥有的最伟大的色彩权威：自然。找到自然界的照片（花朵的照片通常效果特别好），并使用颜色捕获工具选择对比色，如果需要的话，或者选择单一颜色的几种色调。使用这些更新几乎任何可视化软件包的默认值，您将欣赏到您的图形变得更加吸引人和专业。

当涉及到诸如字体大小和标签位置之类的事情时，也没有什么方程式可供遵循——您主要只需*看*一下您的图形，然后在它们看起来不太对劲时微调一下。例如，我清楚地记得在*华尔街日报*编码时，我编写了一个同事设计的列表的代码布局，并且我自然而然地编写了一个`for`循环来精确定位它们。问题是，当我运行代码并渲染它时，某些地方看起来不对劲。我确信我只是在估计间距时估计不准确，于是问他每个元素之间应该有多少像素的白色空间。“我不知道，”他说，“我只是看了看。”

虽然我认识到在您刚开始进行视觉设计时，这种建议可能令人沮丧，但我也可以承诺，如果您给自己一些时间来尝试，最终您将学会相信*自己的*眼睛。有了这个，一些实践，以及对以下各节中列出的（明确定义的）细节的关注，您很快就会产生既准确又*优美*的可视化效果。

## 选择刻度

在[“图表、图形和地图：哦，我的天！”](#charts_graphs_maps)中，我们讨论了与我们的可视化*准确性*相关的规模问题；而在这里，我们的重点是清晰度和可读性。像*seaborn*和*matplotlib*这样的软件包将根据您的数据自动选择刻度和轴限制，但出于各种原因，这些默认值可能需要调整。一旦您确认了数据的适当*数值*范围，您将希望审查您的图形实际渲染的方式，并确保它也遵循这些一般规则：

+   轴限和标记的值应为整数和/或5或10的倍数。

+   值标签*不*应该以科学计数法表示。

+   单位应该只出现在每个轴的最后一个元素上。

+   虽然不是理想的情况，标签可能需要编辑或者（不太推荐的方式）倾斜以保持可读性。

## 选择颜色

除了寻求专家意见来选择特定的颜色之外，还要考虑您的数据元素应该有多少颜色。颜色可以是突出特定数据点或区分测量值和预测值的宝贵方式。在为您的图表或图形选择颜色时，请记住：

每个数据类别一个颜色

例如，如果您显示了有关PPP贷款计划的几个月的数据，则所有的柱子应该是*同*一种颜色。同样，同一变量的不同值应该是单一颜色的不同阴影。

避免连续的颜色分布。

虽然根据其值调整每个可视元素的颜色可能*看起来*更精确，就像我们在[Figure 10-5](#bad_wage_math_flat)中看到的数据视觉压缩一样，连续的色彩调色板（或*渐变*）生成的颜色差异小到人眼实际上无法感知。这就是你的分布计算（你*确实*做了那些吧？）会派上用场的地方：创建一个最多五种颜色的色标（或*渐变*），然后将每一种颜色分配给您数据的一个*五分位数*。

使用分歧色标要谨慎。

只有当数据围绕真正的“中性”值变化时，分歧色标才真正适用。在某些情况下，这个值可能是零。在其他情况下，它可能是领域中的一个约定值（例如，美联储认为通胀率约为[2%](https://federalreserve.gov/faqs/5D58E72F066A4DBDA80BBA659C55F774.htm)是理想的）。

永远不要给超过四个不同类别的数据上色。

不过，把背景中的上下文数据用灰色是可以的。

测试颜色可访问性。

ColorBrewer等工具包含生成只有色盲安全组合的选项。如果您使用自己的颜色，请通过将图形转换为灰度来测试您的选择。如果您在可视化中仍然可以区分所有颜色，您的读者应该也能够。

## 最重要的是，注释！

我们可视化过程的目标是分享我们的数据见解并支持我们的主张。虽然我们的行动动词标题应该概括我们图形的主要思想，但通常有必要突出或添加图形本身特定数据点的上下文。这*不*是星号和脚注的地方。准确或有效地理解图形所需的信息必须成为图形主要视觉范围的一部分。你可以使可视化中的数据更易理解的一些关键方法包括以下几点：

直接标记分类差异。

而不是创建一个单独的图例，直接将数据标签放在可视化中。这样读者就不必在您的图形和一个单独的键之间来回查看，以理解被呈现的信息。

用颜色突出相关数据点。

如果一个关键数据点对你的整体主张至关重要，用对比色突出它。

添加上下文注释。

这些与相关数据元素连接的少量文本（如果必要，使用细线*引线*连接）可以是标签、解释或重要的背景信息。无论是什么，确保它尽可能靠近数据，并始终在图形本身的视觉范围内。

# 从基础到美观：使用 seaborn 和 matplotlib 自定义可视化

关于设计（无论是视觉还是其他方面）的最后一点说明。虽然我在[“雄辩视觉元素”](#elementsofeloquentvisuals)中努力将有效可视化的元素分解为组成部分，但一幅真正雄辩的图形并不是一堆可互换的部分。改变其中的一个部分——移动一个标签，改变一个颜色——意味着许多，如果不是所有的其余部分都需要调整，以使整体恢复平衡。这正是为什么我之前没有为每个单独的设计方面提供代码示例的原因：孤立地看，很难理解为什么任何给定的元素如此重要。但将其视为一个完整的整体，每个元素如何有助于图形的影响将会变得清晰（希望如此）。

要在 Python 中实现这些自定义视觉效果，我们仍然要依赖 *seaborn* 和 *matplotlib* 库。但是，虽然在以前的情况下我们让 *seaborn* 处理大部分繁重的工作，在这个例子中，真正发挥作用的是 *matplotlib* 提供的细粒度控制。是的，我们仍然让 *seaborn* 处理高级任务，比如实际按比例绘制数据值。但 *matplotlib* 将给我们提供我们需要的杠杆，以指定从标签放置和方向到单位、标签、注释和突出显示值等一切，这些都是我们真正需要的，以使我们的可视化真正适应我们的主张。

对于这个例子，我们将暂时离开 PPP 数据，转而使用一些 COVID-19 数据，这些数据是由一组研究人员根据约翰斯·霍普金斯大学的数据整理而成，并在[我们世界的数据](https://ourworldindata.org/coronavirus-source-data)上公开。我们的目标是突出显示 2020 年 7 月美国确诊 COVID-19 病例激增的情况，当时这一情况被归因于许多州在之前春季较早时加快了重新开放^([11](ch10.html#idm45143394142944))，以及在 7 月 4 日假期期间的聚会^([12](ch10.html#idm45143394140400))。要了解依赖默认值和自定义轴范围、标签和颜色之间的区别，请比较[图 10-10](#seaborn_defaults)和[图 10-11](#customized_visuals)。生成[图 10-11](#customized_visuals)的代码显示在[示例 10-5](#refined_covid_barchart)中。

##### 示例 10-5。refined_covid_barchart.py

```py
# `pandas` for data loading; `seaborn` and `matplotlib` for visuals
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# `FuncFormatter` to format axis labels
from matplotlib.ticker import FuncFormatter

# `datetime` to interpret and customize dates
from datetime import datetime

# load the data
vaccine_data = pd.read_csv('owid-covid-data.csv')

# convert the `date` column to a "real" date
vaccine_data['date']= pd.to_datetime(vaccine_data['date'])

# group the data by country and month
country_and_month = vaccine_data.groupby('iso_code').resample('M',
                                                              on='date').sum()

# use `reset_index()` to "flatten" the DataFrame headers
country_and_month_update = country_and_month.reset_index()

# select just the United States' data
just_USA = country_and_month_update[country_and_month_update['iso_code']=='USA']

# make the foundational barplot with `seaborn`
ax = sns.barplot(x="date", y="new_cases", palette=['#bababa'], data=just_USA)

# loop through the bars rectangles and set the color for the July 2020
# bar to red
for i, bar in enumerate(ax.patches):
    if i == 6:
        bar.set_color('#ca0020')

# set the maximum y-axis value to 7M
ax.set_ylim(0,7000000)

# setting the axis labels
plt.xlabel('Month')
plt.ylabel('New cases (M)')

# modify the color, placement and orientation of the "tick labels"
ax.tick_params(direction='out', length=5, width=1, color='#404040',
               colors='#404040',pad=4, grid_color='#404040', grid_alpha=1,
               rotation=45) ![1](assets/1.png)

# functions for formatting the axis "tick labels"
# `millions()` will convert the scientific notation to millions of cases
def millions(val, pos): ![2](assets/2.png)
    modified_val = val*1e-6
    formatted_val = str(modified_val)
    if val == ax.get_ylim()[1]:
        formatted_val = formatted_val+'M'
    if val == 0:
        formatted_val = "0"
    return formatted_val

# `custom_dates()` will abbreviate the dates to be more readable
def custom_dates(val, pos): ![2](assets/2.png)
    dates_list = just_USA.date.tolist()
    date_label = ""
    if pos is not None: ![3](assets/3.png)
        current_value = dates_list[pos]
        current_month = datetime.strftime(current_value, '%b')
        date_label = current_month
        if date_label == 'Jan':
            date_label = date_label + " '"+ datetime.strftime(current_value,
                                                              '%y')
    return date_label

# assign formatter functions
y_formatter = FuncFormatter(millions)
x_formatter = FuncFormatter(custom_dates)

# apply the formatter functions to the appropriate axis
ax.yaxis.set_major_formatter(y_formatter)
ax.xaxis.set_major_formatter(x_formatter)

# create and position the annotation text
ax.text(4, 3000000, "Confirmed cases\noften lag infection\nby several weeks.") ![4](assets/4.png)

# get the value of all bars as a list
bar_value = just_USA.new_cases.tolist()

# create the leader line
ax.vlines( x = 6, color='#404040', linewidth=1, alpha=.7,
                         ymin = bar_value[6]+100000, ymax = 3000000-100000)

# set the title of the chart
plt.title("COVID-19 cases spike following relaxed restrictions\n" + \
          "in the spring of 2020", fontweight="bold")

# show the chart!
plt.show()
```

![1](assets/1.png)](#co_presenting_your_data_CO4-1)

自定义图表上每个轴上指示值的“刻度标签”的方向、颜色和其他属性可以通过 *matplotlib* 的 [`tick_params()` 方法](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.tick_params.html)来完成。

![2](assets/2.png)](#co_presenting_your_data_CO4-2)

提供给任何分配给任何`FuncFormatter`函数的自定义函数的参数将是“值”和“刻度”位置。

[![3](assets/3.png)](#co_presenting_your_data_CO4-4)

在“交互模式”下，如果`pos`为`None`，该函数将抛出错误。

[![4](assets/4.png)](#co_presenting_your_data_CO4-5)

默认情况下，覆盖在图表上的[文本元素的定位](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.text.html)是“数据坐标”，例如，值为1将使文本框的起始左对齐于第一列的中心点。提供的“y”值锚定文本框的*底部*。

![基本的 seaborn 图表。](assets/ppdw_1010.png)

###### 图 10-10\. 由可视化库默认生成的条形图

![定制可视化](assets/ppdw_1011.png)

###### 图 10-11\. 定制化的可视化

正如您从[示例 10-5](#refined_covid_barchart)中可以看到的那样，在Python中定制我们的可视化过程是相当复杂的；调整默认输出的外观和感觉几乎使所需的代码行数增加了近三倍。

与此同时，此实例中的默认输出基本上是难以辨认的。通过淡化颜色，突出显著数据点，以及（可能最重要的是）精炼我们的数据标签，我们设法创建了一张图表，在大多数出版环境中都能自成一格。显然，这里我们的大部分代码可以被精炼和重新利用，以使这种程度的定制成为未来图形制作的一种非常少见的努力。

# 超越基础

尽管我在本章中致力于涵盖有效和准确的数据可视化基础，事实上，有价值的可视化理念可以来自任何地方，而不仅仅是专门讨论此主题的书籍或博客。在我在《华尔街日报》期间，我最喜欢的图形之一是[失业率可视化](https://jennifervalentinodevries.com/2009/09/16/grid-graphic-u-s-unemployment-rate)，显示在[图 10-12](#unemployment_grid)，灵感来自我在一个关于气候变化的博物馆展览中遇到的类似形式；几何热图格式允许读者一目了然地比较数十年来每月失业率。如果你对设计感兴趣，你可以通过简单地批判性地观察你每天看到的媒体（无论是在线广告还是餐厅菜单），学到很多关于什么做得好（或不好）的东西。如果某件事感觉不够精致或吸引人，仔细看看它的组成部分。颜色是否不协调？字体是否难以阅读？或者只是有太多东西挤在太小的空间里？

当然，批评他人的作品很容易。如果你真的想改进自己的可视化，挑战自己尝试构建更好的解决方案，你将在此过程中学到大量知识。如果你发现自己在寻找正确的词汇来描述问题，你可能想看一些[附录 D](app04.html#appendix_d) 中的资源，那里有一些我自己扩展和改进可视化工作的最喜欢的资源。

![随时间变化的美国失业率。](assets/ppdw_1012.png)

###### 图 10-12\. 美国失业率随时间变化（最初设计用于*《华尔街日报》*）

# 结论

像编程一样，可视化是一门应用艺术：提高它的唯一方法是*去做*。如果你手头没有项目可做，从一个你认为在某种程度上不太好的“发现”可视化开始，然后重新设计它。你可以使用 Python 或其他计算工具，或者只是一支铅笔和纸——关键是要尝试解决你在原始可视化中看到的问题。在这个过程中，你将亲身体验到每个设计决策中固有的权衡，并开始建立减少这些权衡以服务于你的可视化目标所需的技能。然后，当你面临自己的数据可视化挑战时，你将有一个可以参考的先前工作作品集，以思考如何最好地呈现你拥有的数据并阐明你需要的观点。

到目前为止，我们已经用 Python 进行了几乎所有我们想做的数据整理，但是*Python 之外*还有一整个世界的数据和可视化工具，这些工具在支持你的数据整理和数据质量工作时可以非常有用。我们将在[第 11 章](ch11.html#chapter11)中介绍这些工具。

^([1](ch10.html#idm45143395782464-marker)) 其中直方图是一种特殊类型。

^([2](ch10.html#idm45143395497264-marker)) 特别是如果你给自己信心[*连接动词*](https://merriam-webster.com/dictionary/linking%20verb)。

^([3](ch10.html#idm45143395160848-marker)) 尽管这些几乎总是公开和频繁地被违反，例如，Junk Chart 的[“从零开始改善这个图表，但只是稍微改善了一点”](https://junkcharts.typepad.com/junk_charts/2021/06/start-at-zero-improves-this-chart-but-only-slightly.html)。

^([4](ch10.html#idm45143395804112-marker)) 由于 *pandas* 和 *seaborn* 库都大量依赖于 *matplotlib*，在许多情况下，需要直接使用 *matplotlib* 的特性进行重要的定制，我们将在[“优雅视觉元素”](#elementsofeloquentvisuals)中更详细地看到。

^([5](ch10.html#idm45143395798752-marker)) 显然，我们可以简单地反转我们最初指定数据的顺序，但我更喜欢数据顺序和最终可视化顺序匹配。

^([6](ch10.html#idm45143395440176-marker)) “曲棍球桨图：科学史上最具争议的图表解释” by Chris Mooney，[*https://theatlantic.com/technology/archive/2013/05/the-hockey-stick-the-most-controversial-chart-in-science-explained/275753*](https://theatlantic.com/technology/archive/2013/05/the-hockey-stick-the-most-controversial-chart-in-science-explained/275753)；也就是说，[组织结构图](http://voices.washingtonpost.com/ezra-klein/2009/07/when_health-care_reform_stops.html)也曾经（谩骂警告）[盛行一时](https://flickr.com/photos/robertpalmer/3743826461)。

^([7](ch10.html#idm45143395430576-marker)) 欲知更多详情，请参阅Wong的《华尔街日报信息图指南》（诺顿出版社）。

^([8](ch10.html#idm45143395284112-marker)) Megan Boldt等人，“学校那工作：尽管表面看起来不佳，但那些表现优于预期的学校有共同特点”，2010年7月9日，[*https://twincities.com/2010/07/09/schools-that-work-despite-appearances-schools-doing-better-than-expected-have-traits-in-common*](https://twincities.com/2010/07/09/schools-that-work-despite-appearances-schools-doing-better-than-expected-have-traits-in-common)。

^([9](ch10.html#idm45143394216688-marker)) 相信我，我已经尝试过。

^([10](ch10.html#idm45143394184624-marker)) 你可以通过使用`pandas`的`quantile()`函数，并传入`0.2`、`0.4`、`0.6`等值来快速完成这一操作。有关如何计算这些值及其代表什么的更一般性提醒，请参见[示例 9-3](ch09.html#ppp_loan_central_and_dist)在[第 9 章](ch09.html#chapter9)中。

^([11](ch10.html#idm45143394142944-marker)) Lazaro Gamio，“自州开始重新开放以来，冠状病毒病例如何上升”， *纽约时报*，2020年7月9日，[*https://nytimes.com/interactive/2020/07/09/us/coronavirus-cases-reopening-trends.html*](https://nytimes.com/interactive/2020/07/09/us/coronavirus-cases-reopening-trends.html)。

^([12](ch10.html#idm45143394140400-marker)) Anne Gearan、Derek Hawkins和Siobhán O’Grady，“上个月冠状病毒病例增长近50%，由首批重新开放的州领导”， *华盛顿邮报*，2020年7月1日，[*https://washingtonpost.com/politics/coronavirus-cases-rose-by-nearly-50-percent-last-month-led-by-states-that-reopened-first/2020/07/01/3337f1ec-bb96-11ea-80b9-40ece9a701dc_story.html*](https://washingtonpost.com/politics/coronavirus-cases-rose-by-nearly-50-percent-last-month-led-by-states-that-reopened-first/2020/07/01/3337f1ec-bb96-11ea-80b9-40ece9a701dc_story.html)。

^([13](ch10.html#idm45143394137600-marker)) Mark Olalde和Nicole Hayden，“加利福尼亚州7月4日后COVID-19病例激增。专家表示家庭聚会助长了传播。” *USA Today*，2020年8月2日，[*https://usatoday.com/story/news/nation/2020/08/02/covid-19-spike-california-after-july-4-linked-family-gatherings/5569158002*](https://usatoday.com/story/news/nation/2020/08/02/covid-19-spike-california-after-july-4-linked-family-gatherings/5569158002)。
