# 第 9 章\. 分层随机化

在上一章中，我们看到了随机化的最简单形式：顾客来了，我们像掷一枚象征性的硬币或骰子一样随机决定。正面，他们看到版本 A；反面，他们看到版本 B。概率可能不是 50/50，但是是恒定的，并且独立于顾客特征。没有“我的对照组比我的处理组稍微年长一些，让我们确保下一个千禧一代的顾客进入对照组。”因此，你的对照组和处理组是“概率上等效”的，这是统计学上的说法，即如果你永远运行你的实验，你的两组将与你的总体人群具有完全相同的比例。然而，在实践中，你的实验组可能会有很大的差异。将解释变量添加到最终分析中可以在一定程度上弥补这些不平衡，但正如我们现在将看到的那样，如果我们事先知道谁将参与我们的实验，我们可以做得更好。

在本章中，我将向您介绍分层随机化，这将确保我们的实验组尽可能相似。这显著增强了实验的解释力，特别是在无法获得大样本量时尤为有用。

分层随机化可以应用于任何我们有预先确定的顾客/员工等名单来建立我们的实验组的情况。考虑到 A/B 测试通常涉及对电子邮件或网站进行轻微更改，我本可以选择电子邮件营销活动作为例子。但我想演示更大的业务举措，这些举措通常由公司高管基于他们的“战略感”做出，并且可以进行测试和验证。

这里的业务背景是，默认情况下，AirCnC为业主提供至少 24 小时的时间来清洁他们的物业，以便安排两次预订之间的间隔。在需求旺盛的市场上，物业一经可订即订，这成为一个重要的限制因素。业务领导人急于减少这一时间并增加每个物业的月度利润。与往常一样，公司内存在两种不同的看法：

+   财务部门主张为业主提供每次预订至少两晚的最短时长选择权。

+   客户体验部门认为最短时长会对客户满意度产生不利影响；相反，它主张为业主提供免费的专业清洁公司服务，以换取将清洁窗口从 24 小时缩减到 8 小时。

在业务中经常出现这样的情况。双方都有一些有说服力的论点，涉及问题的不同方面或强调不同的指标（这里是预订利润与客户体验），并且/或者提供支持其立场的案例性证据（“这家其他公司做了X，所以我们也应该这样做”）。常见的结果是，谁拥有最大的影响力，“赢了”，他们的解决方案被实施，即组织政治起作用。

到了这一步，你可能期望我说一些类似于“但实验可以让你绕过所有政治问题，达到最佳解决方案而不费吹灰之力”的话。我真希望事情能那么简单！事实是，实验在这种情况下可以帮助很大，但它并不是灵丹妙药，原因有两个。

第一个原因是，除非一个解决方案在所有方面都优于另一个，否则在竞争目标之间可能需要进行一些真正的权衡：公司愿意为了利润增加而承受多少客户满意度的降低？这个问题本质上是政治性的，因为公司内不同的利益相关者在这个问题上有不同的偏好。如果你希望你的实验成功，你将不得不尽可能清楚地提出这些权衡，并尽可能提前解决它们。

第二个原因是，你的实验最多会让一方感到满意（如果对照组取得了最佳结果，它也可能使两方都不满意！）。不满意的业务领导人，就像其他不满意的人类一样，在事后很擅长找到合理化的理由：“旧金山湾区与其他高需求市场不同”，“测量客户满意度的调查下降了，但净推荐分数上升了，而NPS是更好的衡量‘真实’客户满意度的指标”等等。

这两个原因使得适当地规划和运行你的实验变得更加重要，不仅从实验设计的角度，也从业务角度来看。

# 实验规划

正如我们在上一章看到的那样，成功规划实验要求我们清楚地阐述其变革理论：

+   业务目标和目标指标是什么？

+   我们的干预措施的定义是什么？

+   这些如何通过我们的行为逻辑相连接？

你现在应该对这个过程很熟悉了，所以我不会详细阐述如何做，我只会快速地介绍一下步骤，让你更深入地理解实验。特别是，我将利用这个机会从行为学的角度指出实验的一些特殊之处。

## 业务目标和目标指标

这个实验的业务目标，或者说我们试图解决的业务问题，是通过减少高需求市场的停机时间来增加盈利能力。因为向业主提供免费清洁服务的成本很高（财务部门估计每天为$10），我们需要将其纳入我们的分析中。

我们将通过相应地修改我们的目标指标来实现这一点。我们的基本指标是每日平均预订利润；相反，我们将使用每日平均预订利润*减去额外成本*。这只是意味着我们需要从基本指标中减去$10，这是免费清洁服务的成本。

然而，也有一些担忧，即最短持续时间干预可能会对客户满意度产生负面影响。我们如何考虑这一点？

我看到其他作者提倡的一个解决方案是使用指标的加权平均值（有时称为总体评估标准[OEC]）。在我们目前的例子中，这意味着为我们的两个变量分配权重，例如各50%，然后使用该新指标作为我们的目标。如果您或您的业务伙伴希望，您当然可以自由使用这种方法，但是我建议选择一个独特的目标指标，如果必要，将几个其他“防护”指标列入监视列表，原因有几个。

第一个问题是，业务目标之间的权衡最终是战略性和政治性决策。关于增加利润值得多少CSAT点，没有客观上的最佳答案；这取决于组织、其背景和当前的优先事项。在某一时间确定的加权平均值使过程看起来具有技术客观性的外观，但实际上只是固化的主观性。

第二个问题是，一个OEC使这些权衡线性化。如果一个CSAT点等于$10百万的利润，那么五个CSAT点等于$50百万的利润。但是，CSAT点数减少可能意味着更少的满意顾客，而减少五个点可能意味着社交媒体风暴。一美元永远等于另一美元，但对于几乎任何其他事情，一系列小变化通常比单个大变化更可取。盲目依赖OEC可能导致更冒险的决策。OEC方法的支持者可能会反对，显然你不应该盲目依赖它；但如果你要看各种组成部分并且进行公开讨论，我不太确定OEC如何帮助。

此外，OEC将业务干预视为固定。继续使用1点CSAT = $10百万的等价，以下两个选项将具有相等的评分为零：

1.  第一个干预措施使利润增加了$1百万，并使CSAT减少了0.1点。

1.  第二个干预措施使利润增加了$50百万，并使CSAT减少了5点。

但从行为角度来看存在着很大的差异。第一个选项基本上是一匹死马，几乎没有希望复活，而第二个选项更像是一匹善变的纯种马。通过将其针对特定的客户细分，改变其条款或呈现方式，可能会找到一种在不承担成本的情况下获得其一些好处的方式。在这种意义上，一个具有显著正负效果的干预措施需要进行探索和设计迭代，而不是OEC鼓励的二元决策。

最后，我认为在某些情况下，OEC被用作捷径。假设某个干预措施增加了短期利润，但也增加了流失率的概率。这并不是一个真正的战略折衷：我们应该衡量流失率对生命周期价值的影响，然后确定对利润的净影响。说你的OEC将是90%的短期利润和10%的流失率效应是一种猜测而不是真正的测量交换率。有了本书的因果行为框架，我们可以比猜测做得更好。

因此，在本章的其余部分，我们将以每日平均预订利润作为我们的单一目标指标，并假设CSAT正在背景中监控任何令人担忧的变化。

###### 注意

说得很好，但当我们谈论跟踪客户满意度时，我们指的是哪些客户？如果客户只想预订一个晚上的地点，并被提供一个有最短时长要求的地点，他们可能会决定预订其他地点（无论是在对照组还是免费清洁组），或者完全放弃通过AirCnC预订而选择预订酒店。因此，我们不能简单地衡量明确定义的客户群体的客户体验。

不幸的是，这个问题并不罕见：每当您进行一个实验，实验单位的随机分配不是客户时，您都必须问自己这将如何在客户端发挥作用。我们可以利用的是最短时长只会影响那些寻找一晚预订的客户；我们还知道客户在展示可用物业之前会输入他们的期望时长。因此，我们可以跟踪我们实验中所有寻找一晚期望时长的客户，并检查他们最终是在同一物业预订了几晚还是在其他物业预订了一晚，或者最终根本没有预订。每当他们预订时，我们还会跟踪他们如何评价他们的住宿。这显然远非完美，因为我们将为不同客户子群体跟踪不同的指标，但这是我们能做到的最好，也是我认为更多地依靠监测保护变量而不是在OEC中聚合它们的另一个例子。

## 干预定义

在确定成功标准之后，我们需要确保对我们正在测试的内容有清晰的理解。当组织的利益很高时，例如，业务领导人就此问题发生争执时，这尤为重要。在这里，需要指出的是，我们为业主提供设置最短租期的机会，这与强制规定最短租期并非同一概念。同样，业主可能会选择或不选择免费清洁服务。此外，处理方式本身仍可能存在不同解释的空间。免费清洁服务对公司来说究竟是多么彻底且成本高昂？我们强制客户遵守的最短租期是多少？

因为我们的两种干预措施都有些复杂，并且依赖于业主理解提议并选择接受，因此可能通过UX研究创建几种不同设计并进行质量测试是一个好主意。此外，在运行实验后，考虑略微不同版本的实施方式也是一个好主意。

最终，您将希望确保所有利益相关者都对实验设计感到满意，并愿意签署批准。这将减少（但不会完全消除！）当结果出来时，他们可能会辩称所测试的内容未能充分代表他们提出的解决方案的风险。

## 行为逻辑

对于两种处理方式，行为逻辑是不同的：最短租期方法可能会增加每次预订的持续时间和金额，但可能会减少总预订数量；另一方面，免费清洁方法可能会增加预订数量，但由于额外成本而降低每次预订的利润。此外，我们需要考虑到我们的实验处理/干预是*提议*，业主可以选择接受或不接受（[图9-1](#the_cd_of_the_two_treatments_under_cons)）。

![考虑中的两种处理方式的CD](Images/BEDA_0901.png)

###### 图9-1\. 考虑中的两种处理方式的CD

## 数据和包

这一章的[GitHub文件夹](https://oreil.ly/BehavioralDataAnalysisCh9)包含两个CSV文件，其中列出了[表9-1](#variables_in_our_data-id00081)中的变量。勾号（✓）表示该文件中存在的变量，而叉号（☓）表示该文件中不存在的变量。

表9-1\. 我们数据中的变量

|  | 变量描述 | chap9-historical_data.csv | chap9-experimental_data.csv |
| --- | --- | --- | --- |
| *ID* | 物业/业主ID，1-5000 | ✓ | ✓ |
| *sq_ft* | 物业面积，460-1120 | ✓ | ✓ |
| *tier* | 物业等级，分类，从1到3，等级递减 | ✓ | ✓ |
| *avg_review* | 物业平均评分，0-10 | ✓ | ✓ |
| *BPday* | 每日预订利润，目标变量，0-126 | ✓ | ✓ |
| *Period* | 历史数据中的月份索引，1-35，实验数据中隐含的36 | ✓ |  |
| *Month* | 年份中的月份，1-12 | ✓ | ✓ |
| *组* | 实验分配，“ctrl”，“treat1”（免费清洁），“treat2”（最小预订时长） | ☓ | ✓ |
| *符合* | 指示所有者是否按其分配的组进行处理的二元变量 | ☓ | ✓ |

在本章中，除了常用的包外，我们还将使用以下软件包：

```py
## R
library(blockTools) # For function block()
library(caret) # For one-hot encoding function dummyVars()
library(scales) # For function rescale()
```

```py
## Python
import random # For functions sample() and shuffle()
# To rescale numeric variables
from sklearn.preprocessing import MinMaxScaler
# To one-hot encode cat. variables
from sklearn.preprocessing import OneHotEncoder
```

# 确定随机分配和样本大小/功效

对于这个实验，我们将一次性根据某一时点所有者列表分配实验组。这使我们有机会通过称为分层的方法显著改善纯随机分配，从一开始就确保我们的两组平衡良好。这将使我们能够从任何给定样本大小中获得更多的统计功效。

因此，我将首先解释随机分配的方法，以便我们可以在功效分析的模拟中使用它。最后，我们将将这些模拟结果与传统的统计功效分析进行比较。

## 随机分配

在进入分层之前，让我们看看标准随机化会是什么样子。

### 随机分配级别

我们随机分配的首要考虑是我们将实施和测量实验结果的级别。在前一章中，我讨论了随机分配应该在客户级别还是预订级别进行的问题。在当前情况下，实验治疗的物流，特别是免费清洁的问题，不允许在预订级别进行实施。因此，我们将在物业所有者的级别上进行随机分配，这在AirCnC的数据中此时为5,000号。

### 标准随机化

这个过程类似于我们在上一个实验中使用的过程，但更简单，因为它可以离线完成而不是实时进行：首先，我们为实验人口中的每个个体分配一个介于0和1之间的随机数。然后，我们根据该随机数分配组别：如果K是我们想要的组的数量（包括一个控制组），那么所有随机数小于1/K的个体属于第一组，所有随机数介于1/K和2/K之间的个体属于第二组，依此类推。下面的代码展示了使用三组和样本大小（仅用于说明目的）为5,000的方法：

```py
## R
no_strat_assgnt_fun <- function(dat, Nexp){
  K <- 3  
  dat <- dat %>%
    distinct(ID) %>%
    slice_sample(n=Nexp) %>%
    mutate(assgnt = runif(Nexp,0,1)) %>%
    mutate(group = case_when(
      assgnt < = 1/K ~ "ctrl",
      assgnt > 1/K & assgnt < = 2/K ~ "treat1",
      assgnt > 2/K ~ "treat2")) %>%
    mutate(group = as.factor(group)) %>%
    select(-assgnt)
  return(dat)
}
no_strat_assgnt <- no_strat_assgnt_fun(hist_data, Nexp = 5000)
```

```py
## Python
def no_strat_assgnt_fun(dat_df, Nexp, K):
    dat_df = pd.DataFrame({'ID': dat_df.ID.unique()})
    dat_df = dat_df.sample(Nexp)
    dat_df['assgnt'] = np.random.uniform(0,1,Nexp)
    dat_df['group'] = 'ctrl'
    dat_df.loc[dat_df['assgnt'].between(0, 1/K, inclusive=True), 
               'group'] = 'treat1'
    dat_df.loc[dat_df['assgnt'].between(1/K, 2/K, inclusive=False), 
               'group'] = 'treat2'
    del(dat_df['assgnt'])
    return dat_df
no_strat_assgnt = no_strat_assgnt_fun(hist_data_df, Nexp = 5000, K = 3)
```

这种方法的一个优点是，通过创建一个简单的循环，可以轻松地将其推广到任意数量的组，控制组标记为`0`，第一个治疗组标记为`1`，依此类推：

```py
## R
no_strat_assgnt_fun <- function(dat, Nexp, K){
  dat <- dat %>%
    distinct(ID) %>%
    slice_sample(n=Nexp) %>%
    mutate(assgnt = runif(Nexp,0,1)) %>%
    mutate(group = -1) # initializing the “group” variable
  for(i in seq(1,K)){
    dat$group = ifelse(dat$assgnt >= (i-1)/K & dat$assgnt < i/K,i-1,dat$group)} 
  dat <- dat %>%
    mutate(group = as.factor(group)) %>%
    select(-assgnt)
  return(dat)
}
no_strat_assgnt <- no_strat_assgnt_fun(hist_data, Nexp = 5000, K = 4)
```

```py
## Python
def no_strat_assgnt_fun(dat_df, Nexp, K):
    dat_df = pd.DataFrame({'ID': dat_df.ID.unique()})
    dat_df = dat_df.sample(Nexp)
    dat_df['assgnt'] = np.random.uniform(0,1,Nexp)
    dat_df['group'] = -1 # initializing the “group” variable
    for i in range(K):
        dat_df.loc[dat_df['assgnt'].between(i/K, (i+1)/K, inclusive=True), 
               'group'] = i
    del(dat_df['assgnt'])
    return dat_df   
no_strat_assgnt = no_strat_assgnt_fun(hist_data_df, Nexp = 5000, K = 4)
```

然而，前一方法的一个问题是，实验组在客户特征上可能不会完全平衡。为了创建平衡的实验组，我们将使用一种称为分层的技术。

### 分层随机化

为什么纯随机分配不是我们最佳的选择？让我们想象一下，我们正在对20名客户进行实验，其中10名男性和10名女性。如果我们以50%的概率将每个客户随机分配到对照组或治疗组，我们期望平均每组将有5名男性和5名女性。“平均”在这里意味着如果我们重复这个分配很多次，平均每组对照组中的男性数量将为5。但在任何给定的实验中，我们只有34.4%的机会在每组中确切地得到5名男性和5名女性，并且基于超几何分布，我们有8.9%的机会得到7名或更多男性在一组中。很明显，随着样本量的增加，这个问题会变得不那么明显。对于100名男性和100名女性，得到70名或更多男性在一组中的概率变得微不足道。但我们不仅关心性别：理想情况下，我们还希望年龄、居住状态、使用模式等方面保持良好的平衡。这将确保我们的结果尽可能适用于我们整个客户群，而不仅仅适用于其中的特定子群。

幸运的是，当我们有幸同时将实验组分配给所有个体时，例如，我们可以比交叉手指并希望最好的做得更好。我们可以对数据进行分层：我们创建了类似客户的“层”，称为strata，^([2](ch09.xhtml#ch01fn19))然后将它们分配到我们的实验组之间。对于我们的10名男性和10名女性客户，我们将创建一层男性，其中5名男性将进入对照组，5名男性将进入治疗组，女性也是类似。这意味着每个个体仍然有50%的机会进入任何一组，但我们的对照组和治疗组现在在性别上完全平衡了。

分层可以应用于任意数量的变量。以性别和居住状态为例，我们将创建一个所有来自堪萨斯州的女性的层，并将其平均分配到我们的对照组和治疗组，依此类推。对于大量变量或连续变量，找到完全匹配变得不可能；在我们的数据中，我们可能没有两个年龄相同、属性完全相同的堪萨斯州女性。解决方案是创建“尽可能相似”的个体对，例如，一个58岁的女性和一个900平方英尺的属性，以及一个56岁的女性和一个930平方英尺的属性，然后随机分配其中一个到对照组，另一个到治疗组。这样，他们仍然有相同的概率单独进入任何实验组。当每个strata只有两个个体时，这也被称为“匹配”，因为我们正在创建匹配的客户对。

像往常一样，直觉是足够清楚的，但实施的细节却是关键。这里有两个步骤：

1.  给“尽可能相似”的短语赋予数学上的意义。

1.  高效地浏览我们的数据，将每个客户分配给一对。

我们将用来表达“尽可能相似”的数学概念是距离。距离可以很容易地应用于单个数值变量。如果一个业主年龄为56岁，另一个业主年龄为58岁，则它们之间的距离为58 − 56 = 2年。类似地，我们可以说一个900平方英尺的房产和一个930平方英尺的房产之间的距离是30平方英尺。

第一个复杂之处在于聚合多个数值变量。我们可以简单地将两个数字相加（或者等效地取平均值），并说我们的两个业主之间的“距离”为2 + 30 = 32个距离单位。这种方法的问题在于，面积数字比年龄数字要大得多，就像我们在例子中看到的那样。两个业主之间30年的差异在行为上可能比他们属性之间30平方英尺的差异更重要。可以通过重新缩放所有数值变量，使它们的最小值重置为0，最大值为1来解决这个问题。这意味着最年轻和最年长业主之间的“距离”为1，最小和最大房产之间的距离也为1。这并不是一个完美的解决方案，特别是在存在异常值时，但对于大多数目的而言，它是快速且足够好的。

第二个复杂之处来自分类变量。一个城市房和一个公寓之间的“距离”是多少？或者拥有游泳池与否的“距离”是多少？一个常见的解决方案是说，如果它们属于同一类别，则两个属性之间的距离为0，否则为1。例如，一个联排别墅和一个独立屋在属性类型变量上的距离为1。数学上，这通过独热编码来实现：即，我们为每个类别创建尽可能多的二进制0/1变量。例如，我们将属性类型=("独立屋"，"联排别墅"，"公寓")转换为三个变量，*type.house*，*type.townhouse*和*type.apartment*。一个公寓会对变量*type.apartment*有值1，对其他两个变量有值0。这还具有将分类“距离”与数值距离可比性的额外优势。实际上，我们在说联排别墅和公寓之间的差异与最小和最大房产之间的差异一样重要。从行为学的角度来看，这又是一个值得讨论的问题，但这是一个很好的起点，通常也是一个很好的结束点。

我编写了R和Python函数，通过重新缩放数值变量和对分类变量进行独热编码来准备我们的数据。这只是样板代码，如果你不关心实现细节，可以跳过这部分代码。

```py
## Python code (output not shown)
def strat_prep_fun(dat_df):
    # Extracting property-level variables
    dat_df = dat_df.groupby(['ID']).agg(
        tier = ('tier', 'mean'),
        avg_review = ('avg_review', 'mean'),
        sq_ft = ('sq_ft', 'mean'),
        BPday = ('BPday', 'mean')).reset_index()
    dat_df['tier'] = pd.Categorical(dat_df.tier, categories=[3,2,1], 
                                    ordered = True)
    dat_df['ID'] = dat_df.ID.astype(str)
    num_df = dat_df.copy().loc[:,dat_df.dtypes=='float64'] #Numeric vars 
    cat_df = dat_df.copy().loc[:,dat_df.dtypes=='category'] #Categorical vars

    # Normalizing all numeric variables to [0,1]
    scaler = MinMaxScaler()
    scaler.fit(num_df)
    num_np = scaler.transform(num_df)

    # One-hot encoding all categorical variables
    enc = OneHotEncoder(handle_unknown='ignore')
    enc.fit(cat_df)
    cat_np = enc.transform(cat_df).toarray()

    #Binding arrays
    data_np = np.concatenate((num_np, cat_np), axis=1)
    del num_df, num_np, cat_df, cat_np, enc, scaler
    return data_np
prepped_data_np = strat_prep_fun(hist_data_df)
```

```py
## R
> strat_prep_fun <- function(dat){
    # Extracting property-level variables
    dat <- dat %>%
      group_by(ID, tier) %>%
      summarise(sq_ft = mean(sq_ft),
                avg_review = mean(avg_review),
                BPday = mean(BPday)) %>%
      ungroup()

    # Isolating the different components of our data
    ID <- dat$ID  # Owner identifier
    dat <- dat %>% select(-ID)
    cat_vars <- dat %>%
      #Selecting categorical variables
      select_if(is.factor) 
    num_vars <- dat %>%
      #Selecting numeric variables
      select_if(function(x) is.numeric(x)|is.integer(x)) 

    #One-hot encoding categorical variables
    cat_vars_out <- data.frame(predict(dummyVars(" ~.", data=cat_vars), 
                                       newdata = cat_vars))

    # Normalizing numeric variables
    num_vars_out <- num_vars %>%
      mutate_all(rescale)

    # Putting the variables back together
    dat_out <- cbind(ID, num_vars_out, cat_vars_out)  %>%
      mutate(ID = as.character(ID)) %>%
      mutate_if(is.numeric, function(x) round(x, 4)) #Rounding for readability

    return(dat_out)}
> prepped_data <- strat_prep_fun(hist_data)
`summarise()` regrouping output by 'ID' (override with `.groups` argument)
> head(prepped_data, 5)
    ID  sq_ft avg_review  BPday tier.3 tier.2 tier.1
1    1 0.3321     0.3514 0.2365      1      0      0
2   10 0.3802     0.7191 0.5231      1      0      0
3  100 0.8370     0.6105 0.6603      0      0      1
4 1000 0.4476     0.4882 0.3843      1      0      0
5 1001 0.3323     0.7276 0.4316      0      1      0
```

一旦我们准备好数据，第二步就是创建配对。这个计算密集型问题在处理更大的数据时很快变得难以解决（至少是如果你想要最优解）。幸运的是，已经创建了算法可以为您处理这个问题。在 R 中，我们可以使用 `blockTools` 包中的 `block()` 函数：

```py
## R
stratified_data <- block(prepped_data, id.vars = c("ID"), n.tr = 3, 
                         algorithm = "naiveGreedy", distance = "euclidean")
```

该函数的参数是：

`id.vars`

这是用于标识数据中个体的变量。

`n.tr`

这是实验组数量，包括对照组。

`algorithm`

表示要使用的算法的名称。可预见的是，`"optimal"` 将在整体上产生最佳配对，但在大数据和有限计算能力的情况下可能很快变得不可行；`"naiveGreedy"` 是计算需求最少且良好的起点。`"optGreedy"` 通常是在准备进行最终分配时的一个很好的折衷选择。

`distance`

表示个体之间距离如何计算。`"euclidean"` 是从高中学来的距离函数，适合我们准备的数据。

函数 `block()` 返回的层次分配格式冗长，因此我创建了一个便捷的包装器，将其输出转换为可用格式。欢迎在 [GitHub](https://oreil.ly/BehavioralDataAnalysisCh9) 查看其代码：这里我只展示它的输出：

```py
## R
> Nexp <- 4998 #Restricting our data to a multiple of 3
> stratified_data <- block_wrapper_fun(prepped_data, Nexp)
Warning message:
attributes are not identical across measure variables;
they will be dropped 
> head(stratified_data,3)
    ID  sq_ft avg_review  BPday tier.3 tier.2 tier.1  group
1  224 0.6932     0.8167 0.4964      1      0      0 treat1
2 3627 0.4143     0.9290 0.6084      1      0      0 treat1
3 4190 0.6686     0.5976 0.2820      1      0      0 treat1
```

请注意，5000 不能被 3 整除，因此我们需要随机丢弃两行，最接近且小于 5000 且能被 3 整除的数字是 4,998。比较两种随机分配方法将表明，通过分层随机化获得的实验组要比通过标准随机化获得的组更加相似。

除了帮助减少实验中的噪音外，分层还有助于如果您打算进行亚组或调节分析（我们稍后会在书中讨论）。正如一句话所说，“在你能控制的地方进行[分层]，在你无法控制的地方进行随机化”（Gerber 和 Green，2012）。

分层随机化是一种有效且健壮的实验分配方法。其健壮性主要来自其透明性：您始终可以事后检查您的实验组在数值变量的均值和分类变量比例方面是否平衡良好。

另外，因为每对中的个体在任何实验组中的结束概率相同，即使使用不良或错误定义的距离函数，也不会比纯随机分配更差。主要风险在于包含太多无关紧要的变量，这些变量会淹没相关变量。然而，这可以通过仅包含作为因果图或主要人口统计变量的一部分的变量来轻松解决。不要仅仅因为可以而添加大量其他变量。

具有大量类别的分类变量有时也会由于其粗糙性而给您的分层增加噪音。以就业为例，数据科学家与统计学家不同，但直觉上，这种差异小于它们与消防员之间的联合差异。非常精细的变量忽略了这种细微差别，最好用更广泛的类别来替代。

免得这些警告使您丧失信心：分层是有效且稳健的；不要害怕使用它。即使仅基于一些关键人口统计变量进行分层也会显著改善，并应该是您的默认方法。

现在我们已经确定了随机分配的方法，让我们进行功效分析，确定样本大小。

## 使用Bootstrap模拟的功效分析

在与业务伙伴的讨论后，我们确定我们希望在净预订利润（BP）每日增加$2时达到90%的功率，因为这是他们感兴趣的最小可观察效果。这意味着对于免费清洁干预（治疗1），“原始”每日BP将增加$12，而对于最短持续时间干预（治疗2），将增加$2。这不会在任何实质性方面影响我们的分析，因为我们可以通过从免费清洁组物业的BP/day中减去$10的成本来简单地转移结果变量，但我们需要记住并执行。为简单起见，我将只讨论最短持续时间干预在我们的功率分析中的情况。

在这种情况下，模拟方法真正发挥作用，因为通常没有专门的公式来计算功效或样本大小，或者现有的公式变得非常复杂。另一种选择是使用标准公式，这些公式忽略了实际情况的细节（例如，我们实验数据的分层），并且祈祷一切顺利（墨菲定律：可能不会）。

我们的过程与[第8章](ch08.xhtml#experimental_design_the_basics)中的相同：

1.  首先，我们将定义我们的度量函数和决策函数。

1.  然后我们将创建一个函数，模拟给定样本大小和效果大小的单个实验。

1.  最后，我们将创建一个函数，模拟大量实验，并计算其中多少结果为真阳性（即，我们的决策函数充分捕捉到效果）；真阳性的百分比即为该样本大小的功效。

### 单次模拟

我们最小持续时间治疗的度量函数如下：

```py
## R
treat2_metric_fun <- function(dat){
  lin_model <- lm(BPday~sq_ft+tier+avg_review+group, data = dat)
  summ <- summary(lin_model)
  coeff <- summ$coefficients['grouptreat2', 'Estimate']
  return(coeff)}
```

```py
## Python
 def treat2_metric_fun(dat_df):
    model = ols("BPday~sq_ft+tier+avg_review+group", data=dat_df)
    res = model.fit(disp=0)
    coeff = res.params['group[T.treat2]']
    return coeff
```

治疗1的度量函数将类似定义。

我们将从[第8章](ch08.xhtml#experimental_design_the_basics)中重用`boot_CI_fun()`和`decision_fun()`函数。换句话说，我们的决策规则是，如果其90%置信区间严格高于零，则实施治疗。我将在下文中重复它们的代码，仅作参考：

```py
## R
> boot_CI_fun <- function(dat, metric_fun, B = 100, conf.level = 0.9){
    #Setting the number of bootstrap samples
    boot_metric_fun <- function(dat, J){
      boot_dat <- dat[J,]
      return(metric_fun(boot_dat))}
    boot.out <- boot(data=dat, statistic=boot_metric_fun, R=B)
    confint <- boot.ci(boot.out, conf = conf.level, type = c('perc'))
    CI <- confint$percent[c(4,5)]

    return(CI)}
> decision_fun <- function(dat, metric_fun){
    boot_CI <- boot_CI_fun(dat, metric_fun)
    decision <- ifelse(boot_CI[1]>0,1,0)
    return(decision)}
```

```py
## Python
def boot_CI_fun(dat_df, metric_fun, B = 100, conf_level = 0.9):
  #Setting sample size
  N = len(dat_df)
  coeffs = []

  for i in range(B):
      sim_data_df = dat_df.sample(n=N, replace = True)
      coeff = metric_fun(sim_data_df)
      coeffs.append(coeff)

  coeffs.sort()
  start_idx = round(B * (1 - conf_level) / 2)
  end_idx = - round(B * (1 - conf_level) / 2)
  confint = [coeffs[start_idx], coeffs[end_idx]]  
  return(confint)

def decision_fun(dat_df, metric_fun, B = 100, conf_level = 0.9):
    boot_CI = boot_CI_fun(dat_df, metric_fun, B = B, conf_level = conf_level)
    decision = 1 if boot_CI[0] > 0  else 0
    return decision
```

然后，我们可以编写运行单个模拟的函数，其中嵌入了迄今为止我们所看到的逻辑：

```py
## R
single_sim_fun <- function(dat, Nexp, eff_size){

  #Filter the data down to a random month ![1](Images/1.png)          
  per <- sample(1:35, size=1)
  dat <- dat %>%
    filter(period == per)

  #Prepare the stratified assignment for a random sample of desired size ![2](Images/2.png) 
  stratified_assgnt <- dat %>%
    slice_sample(n=Nexp) %>%
    #Stratified assignment
    block_wrapper_fun() %>%
    #extract the ID and group assignment
    select(ID, group)

  sim_data <- dat %>%
    #Apply assignment to full data ![3](Images/3.png)                           
    inner_join(stratified_assgnt) %>%
    #Add target effect size
    mutate(BPday = ifelse(group == 'treat2', BPday + eff_size, BPday))

  #Calculate the decision (we want it to be 1) ![4](Images/4.png)    
  decision <- decision_fun(sim_data, treat2_metric_fun)
  return(decision)}
```

```py
## Python
def single_sim_fun(dat_df, metric_fun, Nexp, eff_size, B = 100, 
                   conf_level = 0.9):

    #Filter the data down to a random month ![1](Images/1.png) 
    per = random.sample(range(35), 1)[0] + 1
    dat_df = dat_df.loc[dat_df.period == per]
    dat_df = dat_df.sample(n=Nexp)

    #Prepare the stratified assignment for a random sample of desired size ![2](Images/2.png)
    assgnt = strat_assgnt_fun(dat_df, Nexp = Nexp)
    sim_data_df = dat_df.merge(assgnt, on='ID', how='inner')

    #Add target effect size ![3](Images/3.png)
    sim_data_df.BPday = np.where(sim_data_df.group == 'treat2', 
                                 sim_data_df.BPday + eff_size, sim_data_df.BPday)

    #Calculate the decision (we want it to be 1) ![4](Images/4.png)  
    decision = decision_fun(sim_data_df, metric_fun, B = B, 
                            conf_level = conf_level)
    return decision
```

[![1](Images/1.png)](#comarker91b)

随机选择一个月份，模仿我们将如何运行实际实验的方式（我们不希望在功效分析中使用相隔 10 年的数据）。

[![2](Images/2.png)](#comarker92b)

为所需大小的样本生成分层随机分配。

[![3](Images/3.png)](#comarker93b)

将分配应用于数据，并将目标效应大小应用于治疗组 2。

[![4](Images/4.png)](#comarker94b)

应用决策函数并返回其输出。

### 大规模模拟

从那里，我们可以像[第 8 章](ch08.xhtml#experimental_design_the_basics)中一样，对功效模拟应用相同的总体函数（在下面重复引用）：

```py
## R
power_sim_fun <- function(dat, Nexp, eff_size, Nsim){
  power_list <- vector(mode = "list", length = Nsim)
  for(i in 1:Nsim){
    power_list[[i]] <- single_sim_fun(dat, Nexp, eff_size)}
  power <- mean(unlist(power_list))
  return(power)}
```

```py
## Python
def power_sim_fun(dat_df, metric_fun, Nexp, eff_size, Nsim, B = 100, 
                  conf_level = 0.9):
    power_lst = []
    for i in range(Nsim):
        power_lst.append(single_sim_fun(dat_df, metric_fun = metric_fun, 
                                        Nexp = Nexp, eff_size = eff_size, 
                                        B = B, conf_level = conf_level))
    power = np.mean(power_lst)
    return(power)
```

我们的最大样本量将是 5,000，因为这是 AirCnC 拥有的所有房主的总数。这对于模拟目的来说是一个可以管理的数量，所以让我们首先用这个样本量运行 100 次模拟。如果最终发现我们需要使用整个人口进行实验，那么逐步提高样本量就没有意义了。我们找到了一个功效为 1，这让人感到安慰：所需的样本量小于我们的总人口。从那里开始，我们尝试不同的样本量，随着我们逼近具有 0.90 功效的样本量，逐步增加模拟次数（[图 9-2](#iterative_power_simulations_with_increa)）。

![随着模拟次数增加进行迭代的能力模拟，标签指示运行顺序](Images/BEDA_0902.png)

###### 图 9-2\. 迭代式功效模拟，随着模拟次数增加，标签指示运行顺序

看起来样本量为 1,500 就足够了。就像[第 8 章](ch08.xhtml#experimental_design_the_basics)中那样，现在让我们确定在该样本量下各种效应大小的功效曲线（[图 9-3](#power_to_detect_various_effect_sizes_an)）。

![检测各种效应大小和显著性的能力（样本量 = 1,500）](Images/BEDA_0903.png)

###### 图 9-3\. 检测各种效应大小和显著性的能力（样本量 = 1,500）

正如您在[图 9-3](#power_to_detect_various_effect_sizes_an)中所看到的，我们的功效曲线急剧下降，从效应大小为 $2 下降到 $1，效应大小小于 1 时我们的功效几乎为零；也就是说，如果我们假设治疗将 *BPday* 增加 $1，我们很可能得到一个包含零的置信区间，然后得出结论认为没有效应。在曲线的左端，我们的模拟显著性等于零，而不是预期的 5%。让我们讨论导致这种情况的原因以及我们是否应该担心。

### 理解功效和显著性的权衡

正如我在上一章中提到的，如果我们的数据“行为良好”（即，正态分布，纯粹随机分配到实验组等），并且没有真实效应，我们期望 90%-CI 90% 的时间包含零，严格高于它的时间 5%，严格低于它的时间 5%。在这里，由于分层随机化，我们的假阳性率似乎低于 5%：我在 500 次模拟中没有观察到任何假阳性。通过减少数据中的噪声，分层随机化还降低了假阳性的风险。

这很好，但在这种情况下可能有点过头了，因为它还将小正效应的功率曲线下降到 1，正如我们在[图 9-3](#power_to_detect_various_effect_sizes_an)中看到的那样。让我们换个说法：如果我们有 5% 的机会认为存在效应，而实际上没有，那么当存在一个小效应时，我们至少有 5% 的机会认为存在效应。从这个意义上说，显著性为我们提供了一些对于低效应尺寸的“免费”功率。

让我们将先前的功率曲线与较低置信水平的功率曲线进行比较。根据定义，这将给我们更窄的置信区间，意味着更高的显著性和更高的功效，特别是对于小效应尺寸（[图 9-4](#comparison_of_power_curves_for_confiden)）。

![置信水平为 0.90（实线）、0.80（长虚线）、0.60（短虚线）和 0.40（点线）的功率曲线比较](Images/BEDA_0904.png)

###### 图 9-4。置信水平为 0.90（实线）、0.80（长虚线）、0.60（短虚线）和 0.40（点线）的功率曲线比较

如您所见，使用 40%-CI，我们只会在显著性上获得小幅增长，但在功效上却会大幅增加，功效约为 50%，可以检测到效应尺寸为 0.5。

这是否意味着我们应该使用 40%-CI 而不是 90%-CI？这取决于情况。让我们回到业务问题上。我们的业务合作伙伴要求对于效应尺寸为 2 的功效达到 90%，因为如果效益低于这个值，他们不愿意费心实施任何一种治疗方法。因此，用不包含零的 CI 捕获真实的效应尺寸为 0.5，或者拥有包含零的 CI，从业务角度来看本质上是一样的。无论哪种情况，都不会实施任何治疗。因此，90%-CI 的功率曲线更能反映我们的业务目标。

另一方面，像[第 8 章](ch08.xhtml#experimental_design_the_basics)中的“1-点击按钮”这样的实验，实施的成本和风险是有限的。90% 功率阈值只是一个基准，几乎任何严格正效应都将得到实施。在这种情况下，为小效应尺寸增加功效可能值得略微增加显著性。

更广泛地说，当您的数据或实验设计偏离标准框架时，功效分析不再是将常规数字插入公式的简单事务，而需要理解正在发生的事情并就正确的决策进行判断。幸运的是，功效曲线提供了一个很好的工具，可以在不同情景和不同决策规则下可视化实验的可能结果。

# 分析和解释实验结果

一旦我们进行了实验，我们可以分析其结果。我们的目标指标——每天平均预订利润是连续的，而不是二元的；因此，两种适当的方法是平均数T检验和线性回归。如果您想了解更多关于T检验的内容，我会推荐您参考Gerber和Green（2012），而线性回归我会进行介绍。

在进行定量分析之前，请记住，我们无法强制业主将最短停留期设置为两晚或同意减少清洁窗口的持续时间以换取免费清洁服务。我们只能提供他们选择的机会，有些人选择了，而其他人没有。在技术术语上，这种方法被称为*鼓励设计*，因为我们鼓励受试者接受我们的提议。

鼓励设计非常常见，但它引入了一些额外的考虑，因为现在我们在治疗组中有两类人：那些选择参与的人和那些没有选择的人。出于实际目的，这意味着我们可以尝试回答两个不同的问题：

+   如果我们为整个所有者群体提供选择治疗的可能性会发生什么情况？

+   如果我们强制整个业主群体接受治疗，而不给他们选择退出的选项，会发生什么？

第一个问题的答案被称为*i**ntention-to-treat*（ITT）估计，因为我们打算让人们接受治疗，但我们并不强制。第二个问题更为复杂，仅基于鼓励设计我们无法完全回答（或者至少需要额外的假设），但我们可以通过*complier average causal effect*（CACE）估计得到比ITT估计更接近的近似值。

让我们依次计算这两个估计值。

## 鼓励干预的意图治疗估计

让我们首先计算ITT估计，这将非常简单：它只是实验分配效应的系数，正如我们在前一章中计算的那样。我们是否应该考虑到治疗组中大多数业主并未选择参与的事实？不需要。ITT系数被选择退出的人所稀释是一个特性，而不是错误：同样的稀释会在更大规模上发生。

让我们为选择参与清洁组的业主减少$10，以考虑额外成本，然后运行线性回归。我们可以分别应用我们的度量函数，但我更喜欢一次运行整个回归，以便能够看到其他系数：

```py
## Python (output not shown)
exp_data_reg_df = exp_data_df.copy()
exp_data_reg_df.BPday = np.where((exp_data_reg_df.compliant == 1) & \
                                 (exp_data_reg_df.group == 'treat2'), 
                                 exp_data_reg_df.BPday -10, 
                                 exp_data_reg_df.BPday)
print(ols("BPday~sq_ft+tier+avg_review+group", 
          data=exp_data_reg_df).fit(disp=0).summary())
```

```py
## R
> exp_data_reg <- exp_data %>%
    mutate(BPday = BPday - ifelse(group=="treat2" & compliant, 10,0))
> lin_model <- lm(BPday~sq_ft+tier+avg_review+group, data = exp_data_reg)
> summary(lin_model)

...
Coefficients:
             Estimate Std. Error t value        Pr(>|t|)    
(Intercept) 19.232831   3.573522   5.382 0.0000000854103 ***
sq_ft        0.006846   0.003726   1.838          0.0663 .  
tier2        1.059599   0.840598   1.261          0.2077    
tier1        5.170473   1.036066   4.990 0.0000006728868 ***
avg_review   1.692557   0.253566   6.675 0.0000000000347 ***
`grouptreat1`  `0.966938`   `0.888683`   `1.088`          `0.2767`
`grouptreat2` `-0.172594`   `0.888391`  `-0.194`          `0.8460`
...
```

我们对我们感兴趣的变量，即每天预订利润，进行回归，回归变量包括房产面积、城市等级、顾客评价的平均值以及实验组。*Grouptreat1*的系数指的是最低持续时间治疗，而*Grouptreat2*则指免费清洁治疗。

第一项处理平均使*BPday*增加了约$0.97，但p值相对较高，约为0.27。这表明该系数可能与零没有显著差异，事实上，相应的Bootstrap 90%-CI约为[0.002; 2.66]。

###### 注

如果您运行T检验比较第一处理组和对照组，您会发现检验统计量的绝对值为0.96，接近我们刚刚进行的回归中的系数。同样，对照组和第一处理组之间*BPday*平均值的原始差异约为0.85。这会让我们惊讶吗？不会。由于分层，我们的实验组非常平衡，因此其他独立变量在各组之间具有相同的平均效果。这意味着即使不考虑协变量的指标也是无偏的（但是它们的p值可能会偏离，因为它们不考虑分层）。

第二项处理在成本后使*BPday*减少了约$0.17，价值不高。相应的置信区间为[-2.23; 1.61]。

请记住，我们的商业伙伴只有在干预能够在成本之上每天产生额外$2的*BP*时才感兴趣。从表面上看，这将排除实施最低持续时间干预，不仅因为统计显著性边缘，而且主要因为缺乏经济意义。即使置信区间的下限恰好高于零，这也不会改变我们商业伙伴的决定。

如果这必须是结局，我们的商业伙伴将不会实施任何鼓励措施。然而，考虑到如果我们在全面强制实施最低持续时间治疗而不让人们选择退出会发生什么可能是值得的。

## 强制干预的顺从者平均因果估计

当我们设计一种鼓励方案时，我们能否估计强制实施治疗的效果？试图回答这个问题的一种诱人但不正确的方法是，一方面比较选择参与类别（即“接受治疗者”）的业务指标的值，另一方面比较选择退出类别和对照组的值，将后两者合并为“未接受治疗者”。人们可能会假设这种比较反映了在全面实施治疗措施（例如在热门市场设定两晚住宿的最低要求，或单方面缩短清洁时间并提供免费清洁，而不考虑业主的偏好）后的预期结果。然而，事实并非如此，因为选择接受治疗并不是随机的，很可能会产生混杂因素。在接受治疗的群体内部，可能会有选择参与者在某些方面与选择不参与者不同，例如，他们可能有财务需求或其他特征，使他们对自己的财产更加关注和努力（[Figure 9-5](#the_experimental_allocation_is_randomi)）。

![实验分配是随机的，但接受免费清洁治疗并非如此](Images/BEDA_0905.png)

###### 图 9-5\. 实验分配是随机的，但接受免费清洁治疗并非如此

如果这种相关性是正确的话，那么接受免费清洁服务的人群可能会表现出增加每日预订利润的行为，并且会偏向于高估我们的系数。换句话说，如果我们将选择参与者与选择不参与者进行比较，我们可能会错误地将某些行为的效果归因于此优惠。随机分配确保实验组之间的比较是无偏的，但对于后续的亚组，并不能保证任何东西。

###### 注意

在电子邮件A/B测试的情况下，随机化效果受限意味着各种指标（例如开启率、点击率等）的分子应都是实验组的人数，而不是前一阶段的人数。如果有50%的人打开了您的电子邮件，并且这50%中有50%的人点击了电子邮件，则点击率应表示为25%，而不是50%。

在鼓励设计中，我们希望治疗组的人员选择加入并接受治疗，但我们也希望对控制组的人员不进行治疗。然而，在某些情况下，我们无法阻止他们接受治疗。在我们的例子中，免费清洁治疗具有完全在我们控制之下的特点：业主可以使用专业的清洁服务，但他们必须支付费用，并且软件中内置了预订之间的时间窗口。因此，没有治疗组之外的人员能够获得这种精确的治疗。然而，对于两晚最低住宿治疗，情况就不那么清晰了：控制组之外的物业业主可能会通过拒绝单晚预订请求来非正式地执行两晚最低住宿（[图 9-6](#the_experimental_allocation_is_randomiz)）。

![实验分配是随机的，但接受最低预订治疗并非如此，并且可以发生在治疗组之外](Images/BEDA_0906.png)

###### 图 9-6\. 实验分配是随机的，但接受最低预订治疗并非如此，并且可以发生在治疗组之外

在最低预订治疗中，我们可以观察到业主存在四种可能的情况：

1.  处于对照组且无最低住宿要求的情况下

1.  处于对照组且无最低住宿要求的情况下

1.  处于治疗组且有两晚最低住宿要求的情况下

1.  处于治疗组且无最低住宿要求的情况下

这种分类尚未回答我们的问题，但它为我们区分治疗本身的效果与有利于设定两晚最低住宿的未观察因素的效果提供了一些重要的基础。假设我们可以观察这些因素，并按照这些因素的减少顺序对所有业主进行排名。由于随机分配，我们可以假设在未观察因素的分布方面，控制组和治疗组是相对一致的（[图 9-7](#distribution_of_unobserved_factors_and)）。

对于控制组中所有价值不明因素足够高的业主，将实施两晚最低住宿（B组），而控制组其他业主则不会（A组）。对于治疗组，我们可以合理假设，具有高因素值的业主仍在实施两晚最低住宿，而我们的鼓励干预只是降低了门槛，使一些原本不会实施的业主参与其中（他们一起形成C组）。最后，尽管我们尽了最大努力，但那些因素过低的业主仍然没有实施（D组）。

![未观察因素的分布及两组中的观察行为](Images/BEDA_0907.png)

###### 图 9-7\. 未观察因素的分布及两组中的观察行为

在计量经济学术语中，无论其实验分组如何（控制组中的B组和治疗组中的对应部分C组），总是会接受治疗的受试者被称为*始终接受者*。永远不会接受治疗的受试者（治疗组中的D组和控制组中的对应部分A组）可预见地被称为*从不接受者*。只有在治疗组时才接受治疗的受试者（A组和C组的重叠部分）被称为*顺从者*。

理论上，你可以有第四类别，即只有在控制组中才接受治疗的受试者。他们被称为*反抗者*，因为他们总是做我们希望他们做的完全相反的事情。心理学中对此的技术术语是*反应性*。虽然在现实生活中可能会发生（咳嗽，青少年，咳嗽），但在商业环境中很少成为问题，除非你试图强迫人们做他们不想做的事情，我不会帮助你。

根据定义，我们无法观察到实验中未观察到的因素，这意味着我们只能确定两个群体：分配到控制组的始终接受者（B组）和分配到治疗组的从不接受者（D组）。我们不知道治疗组中实施两晚最低限制的业主是始终接受者还是顺从者，也不知道不实施的控制组业主是顺从者还是从不接受者。然而，这就是其中的诀窍所在，我们可以在实验组之间抵消始终接受者和从不接受者，以衡量对顺从者的治疗效果，这被称为*顺从者平均因果效应*（CACE）。CACE的公式非常简单：^([4](ch09.xhtml#ch01fn21))

<math><mrow><mrow><mi>C</mi><mi>A</mi><mi>C</mi><mi>E</mi><mo>=</mo><mrow><mfrac><mstyle scriptlevel="0" displaystyle="true"><mrow><mi>I</mi><mi>T</mi><mi>T</mi></mrow></mstyle><mstyle scriptlevel="0"><mrow><mi>P</mi><mrow><mo>(</mo><mi>t</mi><mi>r</mi><mi>e</mi><mi>a</mi><mi>t</mi><mi>e</mi><mi>d</mi><mrow><mo>|</mo><mi>T</mi><mi>G</mi></mrow><mo>)</mo><mo>−</mo><mi>P</mi><mrow><mo>(</mo><mi>t</mi><mi>r</mi><mi>e</mi><mi>a</mi><mi>t</mi><mi>e</mi><mi>d</mi><mrow><mo>|</mo><mi>C</mi><mi>G</mi></mrow><mo>)</mo></mrow></mrow></mrow></mstyle></mfrac></mrow></mrow></mrow></math>

换句话说，要确定治疗对顺从者的影响，我们只需按照我们先前的 ITT 估计加权，方法是通过我们实验中的非顺从度来衡量：如果我们在两组中都有全面顺从，即在对照组中没有人接受治疗（P(treated|CG) = 0），在治疗组中每个人都接受治疗（P(treated|TG) = 1），这简化为 ITT 估计。在鼓励设计中，我们经常可以阻止控制组的人们接受治疗，但只有一小部分治疗组的人实际接受了治疗。在这种情况下，CACE 是 ITT 的倍数：如果治疗组只有 10% 的人接受治疗，那么我们的效果会被大大削弱，我们的 CACE 等于 10 倍的 ITT。

CACE 在两个方面非常有用：首先，它为我们提供了在全面实施治疗的情况下的效果估计，没有选择退出的可能性。其次，观察 ITT 和 CACE 之间的关系使我们能够区分两种可能的情况：

+   ITT 低，但 *P*(*treated*|*TG*) - *P*(*treated*|*CG*) 高，这意味着干预对顺从者的影响较低，但顺从度较高。

+   相反，ITT 高，但 *P*(*treated*|*TG*) - *P*(*treated*|*CG*) 低，这意味着干预对顺从者有很大影响，但顺从度低。

在第一种情况下，我们将集中精力提高干预措施的效果，而在第二种情况下，我们将专注于提高接受率，可能通过强制性干预。这些见解也可以帮助我们探索替代设计：也许 8 小时太短了，但 12 小时会更可接受？也许我们不需要提供*免费*清洁，仅建议业主选择信誉良好的服务提供商就足够了？

在我们目前的实验中，治疗组的接受率平均相当低，大约为 20%：

```py
## R (output not shown)
> exp_data_reg %>%
    group_by(group) %>%
    summarise(compliance_rate = mean(compliant))
```

```py
## Python
exp_data_reg_df.groupby('group').agg(compliance_rate = ('compliant', 'mean'))
Out[15]: 
        compliance_rate
group                  
ctrl              1.000
treat1            0.238
treat2            0.166
```

这意味着我们的 CACE 估计比最小持续治疗的 ITT 估计高得多：

*CACE*[1] = *ITT*[1]/*ComplianceRate*[1] = 0.97/0.24 ≈ 4.06

现在，这是一个更有趣的数值。低接受率和高 CACE 表明，我们的干预在实施时基本上是有效的，并且确实产生了价值。我们可以尝试通过改变设计来提高接受率，或者将干预变为强制性。

CACE 的解释非常简洁但狭窄：因为我们（隐式地）比较了相同的人，也就是符合者，跨对照组和治疗组，我们对治疗效果的估计是无偏的。 我们没有无意中捕捉到其他因素的影响。 但是，我们仅为我们人口中的那个狭窄切片进行测量，因此一般性并不是一定的。 符合者可能具有与我们的治疗相互作用的特征。 也就是说，他们可能具有影响（或不仅仅是）是否接受治疗的特征，以及治疗对他们产生的影响有多大。 这就是我们需要从因果关系转向行为视角的地方：我们的治疗是一种提升所有人的潮流，还是参与的人很重要？ 例如，在下一章中，我们将看到呼叫中心中的谈话路径的例子。 在这种情况下，符合不仅意味着应用治疗，还意味着努力说服并不仅仅是敷衍了事。

我们的干预是通过 AirCnC 的网站实施的。 无论您从谁那里租用，两晚最少就是两晚最少。 这意味着，如果在全面推广的情况下，我们可以确信我们的治疗将按计划实施，我们可以向业务伙伴发出放行信号。

# 结论

在前一章中，我们必须在客户连接到网站时“即兴”随机分配实验。 在本章中，我们能够一次性完成随机分配，因此通过创建一对相似的受试者，我们能够对样本进行*分层*（又称*阻断*），其中一个受试者被分配到对照组，另一个受试者被分配到治疗组。 尽管这增加了一层复杂性，但它也显著提高了实验的效果（在统计学上称为功效）。 一旦您熟悉了分层，您将会欣赏到即使从小样本中也能提取见解的能力。

我们还引入了第二个复杂性：我们的实验干预是一种*鼓励*性治疗。 我们为业主提供了可能性，但我们不能强迫他们接受，并且接受并不是随机的。 在这种情况下，我们可以轻松测量鼓励干预本身的效果，但是衡量接受提议的效果（也就是参与治疗）则更加棘手。 幸运的是，我们在实验人口中符合者的 CACE 为该效果提供了一个无偏的估计。 当我们可以假设个人特征与治疗之间没有相互作用时，CACE 可以推广到我们的整个实验人口。 即使我们无法推广到那么远，它也提供了比简单地比较对照组和治疗组（即意图治疗估计）更无偏的估计。

最后，我们进行了多种处理。这并没有从根本上改变任何事情，但也增加了一些复杂性。我建议您在实验旅程中只使用一种处理方式，但我相信长期来看，您会欣赏到运行实验的组织“固定成本”：需要得到所有利益相关者（业务合作伙伴、法律部门等）的批准，并建立技术和数据管道几乎与使用一种处理方式相比几乎没有更多时间。因此，一次性运行多种处理的实验是增加年度测试处理数的关键步骤。

^([1](ch09.xhtml#ch01fn18-marker)) 感谢Andreas Kaltenbrunner指出这是一个超几何分布，而不是二项分布。

^([2](ch09.xhtml#ch01fn19-marker)) 这是拉丁词层的复数形式，*stratum*，因此有层化一词。

^([3](ch09.xhtml#ch01fn20-marker)) 在技术上，有用于分层抽样的函数，如`sklearn.utils.resample()`，但这些函数不允许像我们这里做的基于距离的匹配。

^([4](ch09.xhtml#ch01fn21-marker)) 如果您对其来源感兴趣，可以在[书籍的GitHub仓库](https://github.com/FlorentBuissonOReilly/BehavioralDataAnalysis)中找到推导过程。
