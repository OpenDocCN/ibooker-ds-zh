# 7 基本统计

本章涵盖了

+   描述性统计

+   频率和列联表

+   相关性和协方差

+   t 检验

+   非参数统计

在前面的章节中，你学习了如何将数据导入 R，并使用各种函数来组织和转换数据，使其成为有用的格式。然后我们回顾了基本的数据可视化方法。

一旦你的数据得到适当的组织，并且你已经开始从视觉上探索它，下一步通常是数值上描述每个变量的分布，然后是探索所选变量之间的关系，一次两个。目标是回答如下问题：

+   这些天汽车的平均油耗是多少？具体来说，在汽车品牌和型号的调查中，每加仑英里数（均值、标准差、中位数、范围等）的分布是什么？

+   在一项新药试验之后，药物组与安慰剂组的结果（无改善、有些改善、显著改善）是什么？参与者的性别对结果有影响吗？

+   收入和预期寿命之间的相关性是多少？它与零的差异是否显著？

+   你在美国的不同地区犯罪后更有可能被判入狱吗？地区间的差异在统计上是否显著？

在本章中，我们将回顾 R 函数，用于生成基本的描述性和推断性统计。首先，我们将查看定量变量的位置和尺度度量。然后，你将学习如何为分类变量生成频率和列联表（以及相关的卡方检验）。接下来，我们将检查可用于连续和有序变量的各种相关系数形式。最后，我们将通过参数方法（t 检验）和非参数方法（曼-惠特尼 U 检验、克鲁斯卡尔-沃利斯检验）来研究组间差异。尽管我们的重点是数值结果，但我们将参考图形方法来可视化这些结果。

本章中涵盖的统计方法通常在第一年的本科生统计学课程中教授。如果你对这些方法不熟悉，McCall（2000）和 Kirk（2008）是两个很好的参考文献。或者，对于涵盖的每个主题，许多在线资源（如维基百科）都提供了丰富的信息。

## 7.1 描述性统计

在本节中，我们将探讨连续变量的集中趋势、变异性和分布形状的度量。为了说明目的，我们将使用你在第一章中首次看到的“汽车道路测试”（`mtcars`）数据集中的几个变量。我们的重点将放在每加仑英里数（`mpg`）、马力（`hp`）和重量（`wt`）上：

```
> myvars <- c("mpg", "hp", "wt")
> head(mtcars[myvars])
                   mpg   hp   wt
Mazda RX4         21.0  110  2.62
Mazda RX4 Wag     21.0  110  2.88
Datsun 710        22.8   93  2.32
Hornet 4 Drive    21.4  110  3.21
Hornet Sportabout 18.7  175  3.44
Valiant           18.1  105  3.46
```

首先，我们将查看所有 32 辆车的描述性统计量。然后，我们将按变速器类型（`am`）和发动机气缸配置（`vs`）进行考察。前者编码为`0=自动，` `1=手动`，后者编码为`0=V 形`和`1=直列`。

### 7.1.1 方法大全

当涉及到计算描述性统计量时，R 拥有丰富的资源。让我们从基本安装版中包含的函数开始。然后我们将查看通过使用用户贡献的包可用的扩展。

在基本安装中，你可以使用`summary()`函数来获取描述性统计量。以下列表提供了一个示例。

列表 7.1 使用`summary()`进行描述性统计

```
> myvars <- c("mpg", "hp", "wt")
> summary(mtcars[myvars])
      mpg             hp              wt      
 Min.   :10.4   Min.   : 52.0   Min.   :1.51  
 1st Qu.:15.4   1st Qu.: 96.5   1st Qu.:2.58  
 Median :19.2   Median :123.0   Median :3.33  
 Mean   :20.1   Mean   :146.7   Mean   :3.22  
 3rd Qu.:22.8   3rd Qu.:180.0   3rd Qu.:3.61  
 Max.   :33.9   Max.   :335.0   Max.   :5.42  
```

`summary()`函数提供了数值变量的最小值、最大值、四分位数和平均值，以及因子和逻辑向量的频率。你可以使用第五章中的`apply()`和`sapply()`函数来提供你选择的任何描述性统计量。`apply()`函数与矩阵一起使用，而`sapply()`函数与数据框一起使用。`sapply()`函数的格式如下

```
sapply(x, *FUN*, *options*)
```

其中`x`是数据框，`FUN`是任意函数。如果存在`options`，它们会被传递给`FUN`。你可以插入这里的典型函数包括`mean()`、`sd()`、`var()`、`min()`、`max()`、`median()`、`length()`、`range()`和`quantile()`。函数`fivenum()`返回 Tukey 的五数摘要（最小值、下四分位数、中位数、上四分位数和最大值）。

令人惊讶的是，基本安装版不提供偏度和峰度的函数，但你可以添加自己的。下一列表中的示例提供了几个描述性统计量，包括偏度和峰度。

列表 7.2 使用`sapply()`进行描述性统计

```
> mystats <- function(x, na.omit=FALSE){
                if (na.omit)
                    x <- x[!is.na(x)]
                m <- mean(x)
                n <- length(x)
                s <- sd(x)
                skew <- sum((x-m)³/s³)/n
                kurt <- sum((x-m)⁴/s⁴)/n - 3
                return(c(n=n, mean=m, stdev=s, 
                       skew=skew, kurtosis=kurt))
              }

> myvars <- c("mpg", "hp", "wt")
> sapply(mtcars[myvars], mystats)
            mpg      hp       wt
n         32.000   32.000  32.0000
mean      20.091  146.688   3.2172
stdev      6.027   68.563   0.9785
skew       0.611    0.726   0.4231
kurtosis  -0.373   -0.136  -0.0227               
```

在这个样本中，汽车的均值 mpg 为 20.1，标准差为 6.0。分布向右偏斜（+0.61），并且相对于正态分布来说略平坦（–0.37）。如果你绘制数据图，这一点最为明显。注意，如果你想要省略缺失值，可以使用`sapply(mtcars[myvars],` `mystats,` `na.omit =TRUE)`。

### 7.1.2 更多的方法

几个用户贡献的包提供了描述性统计的函数，包括`Hmisc`、`pastecs`、`psych`、`skimr`和`summytools`。由于空间限制，我们只演示前三个，但你可以用这五个中的任何一个生成有用的摘要。因为这些包不包括在基本分布中，所以你需要在首次使用时安装它们（见第 1.4 节）。

`Hmisc`包中的`describe()`函数返回变量的数量和观测值的数量，缺失值和唯一值的数量，平均值、四分位数以及最高和最低的五个值。以下列表提供了一个示例。

列表 7.3 使用`Hmisc`包中的`describe()`进行描述性统计

```
> library(Hmisc)
> myvars <- c("mpg", "hp", "wt")
> describe(mtcars[myvars])

 3  Variables      32  Observations
---------------------------------------------------------------------------
mpg 
n missing  unique  Mean    .05   .10     .25   .50    .75    .90    .95
32      0    25   20.09 12.00  14.34  15.43  19.20  22.80  30.09  31.30

lowest : 10.4 13.3 14.3 14.7 15.0, highest: 26.0 27.3 30.4 32.4 33.9 
---------------------------------------------------------------------------
hp 
n missing  unique    Mean    .05     .10   .2     .50   .75   .90     .95
32       0     22   146.7  63.65  66.00 96.50 123.00 180.00 243.50 253.55 

lowest :  52  62  65  66  91, highest: 215 230 245 264 335 
---------------------------------------------------------------------------
wt 
n missing  unique    Mean    .05    .10    .25    .50    .75    .90   .95
32      0      29   3.217  1.736  1.956  2.581  3.325  3.610  4.048 5.293

lowest : 1.513 1.615 1.835 1.935 2.140, highest: 3.845 4.070 5.250 5.345 5.424 
---------------------------------------------------------------------------
```

`pastecs` 包包含一个名为 `stat.desc()` 的函数，它提供了一系列广泛的描述性统计。格式如下

```
stat.desc(*x*, basic=TRUE, desc=TRUE, norm=FALSE, p=0.95)
```

其中 *`x`* 是一个数据框或时间序列。如果 `basic=TRUE`（默认值），则提供值数、空值、缺失值、最小值、最大值、范围和总和。如果 `desc=TRUE`（也是默认值），则还会提供中位数、均值、均值的标准误差、均值的 95% 置信区间、方差、标准差和变异系数。最后，如果 `norm=TRUE`（非默认值），则返回正态分布统计信息，包括偏度和峰度（及其统计显著性）以及 Shapiro–Wilk 正态性检验。使用 p-value 选项来计算均值的置信区间（默认为 .95）。下一个列表提供了一个示例。

列表 7.4 在 `pastecs` 包中使用 `stat.desc()` 进行描述性统计

```
> library(pastecs)
> myvars <- c("mpg", "hp", "wt")
> stat.desc(mtcars[myvars])
                mpg       hp      wt
nbr.val       32.00   32.000  32.000
nbr.null       0.00    0.000   0.000
nbr.na         0.00    0.000   0.000
min           10.40   52.000   1.513
max           33.90  335.000   5.424
range         23.50  283.000   3.911
sum          642.90 4694.000 102.952
median        19.20  123.000   3.325
mean          20.09  146.688   3.217
SE.mean        1.07   12.120   0.173
CI.mean.0.95   2.17   24.720   0.353
var           36.32 4700.867   0.957
std.dev        6.03   68.563   0.978
coef.var       0.30    0.467   0.304
```

似乎这还不够，`psych` 包还有一个名为 `describe()` 的函数，它提供了非缺失观测值的数量、均值、标准差、中位数、截断均值、中位数绝对偏差、最小值、最大值、范围、偏度、峰度和均值的标准误差。你可以在下面的列表中看到一个示例。

列表 7.5 在 `psych` 包中使用 `describe()` 进行描述性统计

```
> library(psych)
Attaching package: 'psych'
        The following object(s) are masked from package:Hmisc :
         describe 
> myvars <- c("mpg", "hp", "wt")
> describe(mtcars[myvars])
    var  n   mean    sd median trimmed   mad   min    max
mpg   1 32  20.09  6.03  19.20   19.70  5.41 10.40  33.90
hp    2 32 146.69 68.56 123.00  141.19 77.10 52.00 335.00
wt    3 32   3.22  0.98   3.33    3.15  0.77  1.51   5.42
     range skew kurtosis    se
mpg  23.50 0.61    -0.37  1.07
hp  283.00 0.73    -0.14 12.12
wt    3.91 0.42    -0.02  0.17
```

我告诉你，这是富得流油的尴尬！

注意：在先前的例子中，`psych` 和 `Hmisc` 包都提供了一个名为 `describe()` 的函数。R 是如何知道使用哪一个的？简单来说，最后加载的包具有优先权，如列表 7.5 所示。在这里，`psych` 包是在 `Hmisc` 包之后加载的，并且会打印出一个消息，表明 `Hmisc` 中的 `describe()` 函数被 `psych` 中的函数覆盖。当你输入 `describe()` 函数时，R 会首先搜索 `psych` 包并执行它。如果你想使用 `Hmisc` 版本，你可以输入 `Hmisc::describe(mt)`。函数仍然存在。你必须给 R 提供更多信息才能找到它。

现在你已经知道了如何生成整个数据的描述性统计，让我们回顾一下如何获取数据的子组的统计信息。

### 7.1.3 按组进行描述性统计

当比较个体或观测值的组时，通常关注的是每个组的描述性统计，而不是整个样本。可以使用基础 R 的 `by()` 函数生成组统计信息。格式如下

```
by(*data, INDICES, FUN*)
```

其中 *`data`* 是一个数据框或矩阵，*`INDICES`* 是一个因子或因子列表，它定义了组，而 *`FUN`* 是一个作用于数据框所有列的任意函数。下一个列表提供了一个示例。

列表 7.6 使用 `by()` 按组进行描述性统计

```
> dstats <- function(x)sapply(x, mystats)
> myvars <- c("mpg", "hp", "wt")
> by(mtcars[myvars], mtcars$am, dstats)

mtcars$am: 0
             mpg        hp        wt
n          19.000    19.0000   19.000
mean       17.147   160.2632    3.769
stdev       3.834    53.9082    0.777
skew        0.014    -0.0142    0.976
kurtosis   -0.803    -1.2097    0.142
---------------------------------------- 
mtcars$am: 1
             mpg        hp        wt
n          13.0000    13.000   13.000
mean       24.3923   126.846    2.411
stdev       6.1665    84.062    0.617
skew        0.0526     1.360    0.210
kurtosis   -1.4554     0.563   -1.174
```

在这种情况下，`dstats()` 将列表 7.2 中的 `mystats()` 函数应用于数据框的每一列。将其放在 `by()` 函数中，你可以得到 `am` 的每个级别的汇总统计信息。

在下一个例子（列表 7.7）中，为两个 `by` 变量（`am` 和 `vs`）生成了汇总统计量，并且每个组的统计结果都使用自定义标签打印出来。此外，在计算统计量之前，省略了缺失值。

列表 7.7：由多个变量定义的组的描述性统计

```
> dstats <- function(x)sapply(x, mystats, na.omit=TRUE)
> myvars <- c("mpg", "hp", "wt")
> by(mtcars[myvars], 
     list(Transmission=mtcars$am,
          Engine=mtcars$vs), 
     FUN=dstats)

Transmission: 0
Engine: 0
                mpg          hp         wt
n        12.0000000  12.0000000 12.0000000
mean     15.0500000 194.1666667  4.1040833
stdev     2.7743959  33.3598379  0.7683069
skew     -0.2843325   0.2785849  0.8542070
kurtosis -0.9635443  -1.4385375 -1.1433587
----------------------------------------------------------------- 
Transmission: 1
Engine: 0
                mpg          hp          wt
n         5.0000000   6.0000000  6.00000000
mean     19.5000000 180.8333333  2.85750000
stdev     4.4294469  98.8158219  0.48672117
skew      0.3135121   0.4842372  0.01270294
kurtosis -1.7595065  -1.7270981 -1.40961807
----------------------------------------------------------------- 
Transmission: 0
Engine: 1
                mpg          hp         wt
n         7.0000000   7.0000000  7.0000000
mean     20.7428571 102.1428571  3.1942857
stdev     2.4710707  20.9318622  0.3477598
skew      0.1014749  -0.7248459 -1.1532766
kurtosis -1.7480372  -0.7805708 -0.1170979
----------------------------------------------------------------- 
Transmission: 1
Engine: 1
                mpg         hp         wt
n         7.0000000  7.0000000  7.0000000
mean     28.3714286 80.5714286  2.0282857
stdev     4.7577005 24.1444068  0.4400840
skew     -0.3474537  0.2609545  0.4009511
kurtosis -1.7290639 -1.9077611 -1.3677833
```

虽然前面的例子使用了 `mystats()` 函数，但你也可以使用 `Hmisc` 和 `psych` 包中的 `describe()` 函数，或者 `pastecs` 包中的 `stat.desc()` 函数。实际上，`by()` 函数提供了一个通用的机制，可以重复对任何子组进行任何分析。

### 7.1.4 使用 dplyr 交互式汇总数据

到目前为止，我们一直关注生成给定数据框的全面描述性统计方法。然而，在交互式、探索性数据分析中，我们的目标是回答有针对性的问题。在这种情况下，我们希望从特定的观察组中获得有限数量的统计信息。

在第 3.11 节中介绍的 `dplyr` 包为我们提供了快速灵活地完成此任务的工具。`summarize()` 和 `summarize_all()` 函数可以用来计算任何统计量，而 `group_by()` 函数可以用来指定计算这些统计量的组。

作为演示，让我们使用 `carData` 包中的 `Salaries` 数据框提出并回答一系列问题。该数据集包含美国一所大学 2008-2009 年九个月的美元薪资（`salary`），涉及 397 名教职员工。这些数据是作为持续监测男性和女性教职员工薪资差异的持续努力的一部分而收集的。

在继续之前，请确保已安装 `carData` 和 `dplyr` 包（`install.packages(c("carData", "dplyr"))`）。然后加载这些包

```
library(dplyr)
library(carData)
```

我们现在准备好对数据进行查询。

397 名教授的薪资中位数和薪资范围是多少？

```
> Salaries %>%
    summarize(med = median(salary), 
              min = min(salary), 
              max = max(salary))
     med   min    max
1 107300 57800 231545
```

`Salaries` 数据集被传递给 `summarize()` 函数，该函数计算薪资的中位数、最小值和最大值，并将结果作为一行 tibble（数据框）返回。九个月薪资的中位数是 $107,300，至少有一个人赚得超过 $230,000。显然，我需要要求加薪。

按性别和职称，教职员工的数量、中位薪资和薪资范围是多少？

```
> Salaries %>%
    group_by(rank, sex) %>%
    summarize(n = length(salary),
              med = median(salary), 
              min = min(salary), 
              max = max(salary))

  rank      sex        n     med   min    max
  <fct>     <fct>  <int>   <dbl> <int>  <int>
1 AsstProf  Female    11  77000  63100  97032
2 AsstProf  Male      56  80182  63900  95079
3 AssocProf Female    10  90556\. 62884 109650
4 AssocProf Male      54  95626\. 70000 126431
5 Prof      Female    18 120258\. 90450 161101
6 Prof      Male     248 123996  57800 231545
```

当在 `by_group()` 语句中指定分类变量时，`summarize()` 函数为它们级别的每个组合生成一行统计信息。在每一所学院的职称中，女性的中位薪资低于男性。此外，这所大学有大量的男性正教授。

按性别和职称，教职员工的平均服务年限和自获得博士学位以来的年限是多少？

```
> Salaries %>%
    group_by(rank, sex) %>%
    select(yrs.service, yrs.since.phd) %>%
    summarize_all(mean)

  rank      sex    yrs.service yrs.since.phd
  <fct>     <fct>        <dbl>         <dbl>
1 AsstProf  Female        2.55          5.64
2 AsstProf  Male          2.34          5   
3 AssocProf Female       11.5          15.5 
4 AssocProf Male         12.0          15.4 
5 Prof      Female       17.1          23.7 
6 Prof      Male         23.2          28.6
```

`summarize_all()` 函数为每个非分组变量（`yrs.service` 和 `yrs.since.phd`）计算汇总统计量。如果你想要每个变量的多个统计量，请以列表形式提供。例如，`summarize_all(list(mean=mean, std=sd))` 将为每个变量计算均值和标准差。男性和女性在助理教授和副教授级别上的经验历史相当。然而，女性正教授的经验年数少于她们的男性同行。

`dplyr` 方法的一个优点是结果以 tibbles（数据框）的形式返回。这允许你进一步分析这些汇总结果，绘制它们，并将它们重新格式化以供打印。它还提供了一个简单的机制来聚合数据。

通常情况下，数据分析师都有自己的偏好，选择哪些描述性统计量来展示，以及他们喜欢如何格式化这些统计量。这可能是为什么有这么多变体可供选择。选择最适合你的一个，或者创建你自己的！

### 7.1.5 可视化结果

分布特性的数值汇总很重要，但它们不能替代视觉表示。对于定量变量，你有直方图（第 6.4 节）、密度图（第 6.5 节）、箱线图（第 6.6 节）和点图（第 6.7 节）。它们可以提供通过依赖少量描述性统计量容易错过的见解。

到目前为止考虑的函数提供了定量变量的汇总。下一节中的函数允许你检查分类变量的分布。

## 7.2 频率和列联表

在本节中，我们将查看来自分类变量的频率和列联表，以及独立性检验、关联度测量和图形显示结果的方法。我们将使用基本安装中的函数，以及 `vcd` 和 `gmodels` 包中的函数。在以下示例中，假设 `A`、`B` 和 `C` 代表分类变量。

本节的数据来自 `vcd` 包中包含的 `Arthritis` 数据集。数据来自 Koch 和 Edward（1988 年）的研究，代表了一种针对类风湿性关节炎的新治疗方法的双盲临床试验。以下是前几个观测值：

```
> library(vcd)
> head(Arthritis)
    ID      Treatment       Sex     Age     Improved
1   57      Treated         Male    27      Some
2   46      Treated         Male    29      None
3   77      Treated         Male    30      None
4   17      Treated         Male    32      Marked
5   36      Treated         Male    46      Marked
6   23      Treated         Male    58      Marked
```

处理（安慰剂，治疗），性别（男性，女性），以及改善（无，一些，显著）都是分类因素。在下一节中，你将根据数据创建频率和列联表（交叉分类）。

### 7.2.1 生成频率表

R 提供了多种创建频率和列联表的方法。最重要的函数列于表 7.1 中。

表 7.1 创建和操作列联表的函数

| 函数 | 描述 |
| --- | --- |
| `table(*var1*, *var2*, ..., *varN*)` | 从 `N` 个分类变量（因素）创建一个 `N` 方列联表 |
| `xtabs(*formula, data*)` | 根据公式和矩阵或数据框创建一个`N`维列联表 |
| `prop.table(*table, margins*)` | 将表条目表示为由`margins`定义的边际表的分数 |
| `margin.table(*table, margins*)` | 计算由`margins`定义的边际表的条目总和 |
| `addmargins(*table, margins*)` | 在表上添加摘要`margins`（默认为总和） |
| `ftable(*table*)` | 创建一个紧凑的、“扁平”的列联表 |

在接下来的几节中，我们将使用这些函数中的每一个来探索分类变量。我们将从简单的频率开始，然后是双向列联表，最后是多维列联表。第一步是使用`table()`或`xtabs()`函数创建一个表，然后使用其他函数对其进行操作。

一维表

你可以使用`table()`函数生成简单的频率计数。以下是一个示例：

```
> mytable <- with(Arthritis, table(Improved))
> mytable
Improved
  None   Some  Marked 
   42     14     28
```

你可以使用`prop.table()`将这些频率转换为比例

```
> prop.table(mytable)
Improved
  None   Some  Marked 
 0.500  0.167  0.333
```

或者使用`prop.table()*100`转换为百分比：

```
> prop.table(mytable)*100
Improved
  None   Some  Marked 
  50.0   16.7   33.3
```

在这里，你可以看到 50%的研究参与者有一些或显著的改善（16.7 + 33.3）。

二维表

对于双向表，`table()`函数的格式是

```
mytable <- table(*A*, *B*)
```

其中`A`是行变量，`B`是列变量。或者，`xtabs()`函数允许你使用公式样式输入创建列联表。格式是

```
*mytable* <- xtabs(~ *A* + *B*, data=*mydata*)
```

其中`mydata`是一个矩阵或数据框。一般来说，要交叉分类的变量出现在公式的右边（即`~`的右边），由加号分隔。如果一个变量出现在公式的左边，它被假定为频率向量（如果数据已经过分类很有用）。

对于`Arthritis`数据，你有

```
> mytable <- xtabs(~ Treatment + Improved, data=Arthritis)
> mytable
          Improved
Treatment  None  Some  Marked
  Placebo   29    7      7
  Treated   13    7     21
```

你可以使用`margin.table()`和`prop.table()`函数分别生成边际频率和比例。对于行总和和行比例，你有

```
> margin.table(mytable, 1)
Treatment
Placebo Treated 
   43      41 
> prop.table(mytable, 1)
           Improved
Treatment   None   Some   Marked
  Placebo  0.674  0.163   0.163
  Treated  0.317  0.171   0.512
```

索引（`1`）指的是`xtabs()`语句中的第一个变量——行变量。每行的比例加起来等于 1。查看表格，你可以看到 51%的接受治疗的人有显著改善，而接受安慰剂的人只有 16%。

对于列总和和列比例，你有

```
> margin.table(mytable, 2)
Improved
   None   Some  Marked 
    42     14     28 
> prop.table(mytable, 2)
            Improved
Treatment   None   Some   Marked
  Placebo  0.690  0.500   0.250
  Treated  0.310  0.500   0.750
```

这里，索引（`2`）指的是`xtabs()`语句中的第二个变量——即列。每列的比例加起来等于 1。

使用此语句可以获得单元格比例：

```
> prop.table(mytable)
            Improved
Treatment   None    Some   Marked
  Placebo  0.3452  0.0833  0.0833
  Treated  0.1548  0.0833  0.2500
```

所有单元格比例的总和加起来等于 1。

你可以使用`addmargins()`函数将这些表的边际总和添加到其中。例如，以下代码添加了一个`Sum`行和列：

```
> addmargins(mytable)
                Improved
Treatment   None    Some   Marked    Sum
  Placebo    29       7       7       43
  Treated    13       7      21       41
  Sum        42      14      28       84
> addmargins(prop.table(mytable))
                Improved
Treatment   None    Some   Marked    Sum
  Placebo  0.3452  0.0833  0.0833  0.5119
  Treated  0.1548  0.0833  0.2500  0.4881
  Sum      0.5000  0.1667  0.3333  1.0000
```

当使用`addmargins()`时，默认是为表中的所有变量创建总和边际。相比之下，以下代码仅添加一个`Sum`列：

```
> addmargins(prop.table(mytable, 1), 2)
                Improved
Treatment   None    Some   Marked    Sum
  Placebo   0.674   0.163   0.163    1.000
  Treated   0.317   0.171   0.512    1.000
```

同样，此代码添加了一个`Sum`行：

```
> addmargins(prop.table(mytable, 2), 1)
            Improved
Treatment   None    Some   Marked
  Placebo   0.690   0.500   0.250
  Treated   0.310   0.500   0.750
  Sum       1.000   1.000   1.000
```

在表中，你可以看到 25%的显著改善的患者接受了安慰剂。

注意：`table()`函数默认忽略缺失值（`NAs`）。要包括`NA`作为频率计数中的有效类别，请包含表格选项`useNA="ifany"`。

创建双向表的第三种方法是`gmodels`包中的`CrossTable()`函数。`CrossTable()`函数生成类似于 SAS 中的`PROC FREQ`或 SPSS 中的`CROSSTABS`的双向表。以下列表提供了一个示例。

列表 7.8 使用`CrossTable`的二维表

```
> library(gmodels)
> CrossTable(Arthritis$Treatment, Arthritis$Improved)

   Cell Contents
|-------------------------|
|                       N |
| Chi-square contribution |
|           N / Row Total |
|           N / Col Total |
|         N / Table Total |
|-------------------------|

Total Observations in Table:  84 

                    | Arthritis$Improved 
Arthritis$Treatment |      None |      Some |    Marked | Row Total | 
--------------------|-----------|-----------|-----------|-----------|
            Placebo |        29 |         7 |         7 |        43 | 
                    |     2.616 |     0.004 |     3.752 |           | 
                    |     0.674 |     0.163 |     0.163 |     0.512 | 
                    |     0.690 |     0.500 |     0.250 |           | 
                    |     0.345 |     0.083 |     0.083 |           | 
--------------------|-----------|-----------|-----------|-----------|
            Treated |        13 |         7 |        21 |        41 | 
                    |     2.744 |     0.004 |     3.935 |           | 
                    |     0.317 |     0.171 |     0.512 |     0.488 | 
                    |     0.310 |     0.500 |     0.750 |           | 
                    |     0.155 |     0.083 |     0.250 |           | 
--------------------|-----------|-----------|-----------|-----------|
       Column Total |        42 |        14 |        28 |        84 | 
                    |     0.500 |     0.167 |     0.333 |           | 
--------------------|-----------|-----------|-----------|-----------|
```

`CrossTable()`函数有选项可以报告百分比（行、列和单元格）；指定小数位数；生成卡方、Fisher 和 McNemar 独立性检验；报告期望值和残差值（皮尔逊、标准化和调整标准化）；将缺失值视为有效；用行和列标题注释；并以 SAS 或 SPSS 风格输出格式化。有关详细信息，请参阅`help(CrossTable)`。

如果你有两个以上的分类变量，你将处理多维表。我们将在下一部分考虑这些。

多维表

`table()`和`xtabs()`都可以用于根据三个或更多分类变量生成多维表。`margin.table()`、`prop.table()`和`addmargins()`函数自然扩展到超过两个维度。此外，`ftable()`函数可以用于以紧凑和吸引人的方式打印多维表。以下列表提供了一个示例。

列表 7.9 三维列联表

```
> mytable <- xtabs(~ Treatment+Sex+Improved, data=Arthritis)   ❶
> mytable          
, , Improved = None   

           Sex
Treatment  Female  Male
  Placebo      19    10
  Treated       6     7

, , Improved = Some

           Sex
Treatment  Female  Male
  Placebo       7     0
  Treated       5     2

, , Improved = Marked

           Sex
Treatment  Female  Male
  Placebo       6     1
  Treated      16     5

> ftable(mytable)                
                   Sex Female Male
Treatment Improved                
Placebo   None             19   10
          Some              7    0
          Marked            6    1
Treated   None              6    7
          Some              5    2
          Marked           16    5

> margin.table(mytable, 1)                                    ❷

Treatment 
Placebo Treated                                   
     43      41 
> margin.table(mytable, 2)        
Sex
Female   Male 
    59     25 
> margin.table(mytable, 3)
Improved
  None   Some Marked 
    42     14     28 
> margin.table(mytable, c(1, 3))                              ❸
         Improved
Treatment None Some Marked                          
  Placebo   29    7      7
  Treated   13    7     21
 > ftable(prop.table(mytable, c(1, 2)))                       ❹
                 Improved  None  Some Marked
Treatment Sex                                           
Placebo   Female          0.594 0.219  0.188
          Male            0.909 0.000  0.091
Treated   Female          0.222 0.185  0.593
          Male            0.500 0.143  0.357

> ftable(addmargins(prop.table(mytable, c(1, 2)), 3))     
                 Improved  None  Some Marked   Sum
Treatment Sex                                     
Placebo   Female          0.594 0.219  0.188 1.000
          Male            0.909 0.000  0.091 1.000
Treated   Female          0.222 0.185  0.593 1.000
          Male            0.500 0.143  0.357 1.000
```

❶ 单元频率

❷ 边际频率

❸ 处理 × 改进边际频率

❹ 处理 × 性别改进比例

❶处的代码生成了三维分类的单元频率。该代码还演示了如何使用`ftable()`函数打印出更紧凑和吸引人的表格版本。

❷处的代码生成了`Treatment`、`Sex`和`Improved`的边际频率。因为你使用公式`~Treatment+Sex + Improved`创建了表格，所以`Treatment`通过索引`1`引用，`Sex`通过索引`2`引用，而`Improved`通过索引`3`引用。

❸处的代码生成了`Treatment` `x Improved`分类的边际频率，按`Sex`求和。❹提供了每个`Treatment` `×` `Sex`组合中`None`、`Some`和`Marked`改进的病人比例。在这里，你可以看到 36%的接受治疗的男性有显著的改善，而接受治疗的女性中有 59%。一般来说，比例将在`prop.table()`调用中未包含的索引上求和为 1（在这个例子中是第三个索引，即`Improved`）。你可以在最后一个示例中看到这一点，其中你在第三个索引上添加了一个总和边缘。

如果你想要百分比而不是比例，可以将结果表乘以 100。例如，此语句

```
ftable(addmargins(prop.table(mytable, c(1, 2)), 3)) * 100
```

生成此表：

```
                   Sex Female  Male   Sum
Treatment Improved                       
Placebo   None           65.5  34.5 100.0
          Some          100.0   0.0 100.0
          Marked         85.7  14.3 100.0
Treated   None           46.2  53.8 100.0
          Some           71.4  28.6 100.0
          Marked         76.2  23.8 100.0
```

列联表告诉你表中每个变量组合的案例频率或比例，但你可能也对表中的变量是否相关或独立感兴趣。独立性检验将在下一节中介绍。

### 7.2.2 独立性检验

R 提供了多种测试分类变量独立性的方法。本节中描述的三种测试分别是卡方独立性检验、Fisher 精确检验和 Cochran-Mantel-Haenszel 检验。

独立性卡方检验

你可以将 `chisq.test()` 函数应用于双向表，以产生行和列变量的独立性卡方检验。请参见下一列表中的示例。

列表 7.10 独立性卡方检验

```
> library(vcd)
> mytable <- xtabs(~Treatment+Improved, data=Arthritis)          
> chisq.test(mytable)                                          
        Pearson’s Chi-squared test
data:  mytable                                                 
 X-squared = 13.1, df = 2, p-value = 0.001463               ❶

> mytable <- xtabs(~Improved+Sex, data=Arthritis)              
> chisq.test(mytable)                                           
        Pearson's Chi-squared test                                
data:  mytable  
 X-squared = 4.84, df = 2, p-value = 0.0889                ❷

Warning message:    
In chisq.test(mytable) : Chi-squared approximation may be incorrect
```

❶ 治疗和改善不是独立的。

❷ 性别和改善是独立的。

从结果来看，似乎治疗接受情况与改善程度之间存在关联（p < .01）。但似乎没有发现患者性别与改善程度之间的关联（p > .05）。p 值是在假设总体中行和列变量的独立性成立的情况下，获得样本结果的概率。因为概率很小❶，所以你拒绝治疗类型和结果独立的假设。因为概率❷并不小，所以假设结果和性别独立是合理的。列表 7.10 中的警告信息产生是因为表格中的六个单元格（男性，有些改善）的期望值小于 `5`，这可能会使卡方近似无效。

Fisher 精确检验

你可以通过 `fisher.test()` 函数进行 Fisher 精确检验。Fisher 精确检验评估的是列联表中行和列的独立性零假设，其中边际是固定的。格式是 `fisher.test(mytable)`，其中 `mytable` 是一个双向表。以下是一个示例：

```
> mytable <- xtabs(~Treatment+Improved, data=Arthritis)
> fisher.test(mytable)
        Fisher's Exact Test for Count Data
data:  mytable 
p-value = 0.001393
alternative hypothesis: two.sided
```

与许多统计软件包不同，`fisher.test()` 函数可以应用于任何具有两个或更多行和列的双向表，而不仅仅是 2 × 2 独立性表。

Cochran-Mantel-Haenszel 检验

`mantelhaen.test()` 函数提供了对第三变量每个层中两个名义变量条件独立性的零假设的 Cochran-Mantel-Haenszel 卡方检验。以下代码测试了 `Treatment` 和 `Improved` 变量在 `Sex` 的每个水平上是否独立的假设。该测试假设不存在三重交互作用（`Treatment` × `Improved` × `Sex`）：

```
> mytable <- xtabs(~Treatment+Improved+Sex, data=Arthritis)
> mantelhaen.test(mytable)
        Cochran-Mantel-Haenszel test
data:  mytable 
Cochran-Mantel-Haenszel M² = 14.6, df = 2, p-value = 0.0006647
```

结果表明，在 `Sex` 的每个水平上，接受的治疗和报告的改善情况并不是独立的（即在控制性别的情况下，接受治疗的人比接受安慰剂的人改善更多）。

### 7.2.3 关联度量

上一节中的显著性检验评估是否存在足够的证据来拒绝变量之间独立性的零假设。如果你可以拒绝零假设，你的兴趣自然会转向关联性度量，以衡量现有关系的强度。`vcd` 包中的 `assocstats()` 函数可以用来计算双向表的 phi 系数、列联系数和 Cramér 的 V 值。以下是一个示例。

列表 7.11 双向表的关联性度量

```
> library(vcd)
> mytable <- xtabs(~Treatment+Improved, data=Arthritis)
> assocstats(mytable)
                    X² df  P(> X²)
Likelihood Ratio 13.530  2 0.0011536
Pearson          13.055  2 0.0014626

Phi-Coefficient   : 0.394 
Contingency Coeff.: 0.367 
Cramer's V        : 0.394
```

通常情况下，较大的数值表示更强的关联性。`vcd` 包还提供了一个 `kappa()` 函数，可以计算混淆矩阵的 Cohen 的 kappa 和加权 kappa（例如，两个将一组对象分类到类别中的评委之间的一致程度）。

### 7.2.4 结果的可视化

R 具有探索分类变量之间关系的机制，这些机制远远超出了大多数其他统计平台所发现的。你通常使用条形图来可视化一维中的频率（参见第 6.1 节）。`vcd` 包提供了用于使用镶嵌图和关联图可视化多维数据集中分类变量之间关系的优秀函数（参见第 11.4 节）。最后，`ca` 包中的对应分析函数允许你使用各种几何表示来可视化列联表中行与列之间的关系（Nenadic´和 Greenacre，2007）。

这结束了关于列联表的讨论，直到我们在第十一章和第十九章中探讨更高级的主题。接下来，让我们看看各种类型的相关系数。

## 7.3 相关系数

相关系数用于描述定量变量之间的关系。符号（正号或负号）表示关系的方向（正相关或负相关），而数值表示关系的强度（从 0 表示没有关系到 1 表示完全可预测的关系）。

在本节中，我们将探讨各种相关系数以及显著性检验。我们将使用 R 基础安装中可用的 `state.x77` 数据集。它提供了 1977 年 50 个美国州的人口、收入、文盲率、预期寿命、谋杀率和高中毕业率的数据。还有温度和土地面积指标，但我们将删除它们以节省空间。使用 `help(state .x77)` 可以了解更多关于该文件的信息。除了基础安装外，我们还将使用 `psych` 和 `ggm` 包。

### 7.3.1 相关类型的种类

R 可以产生各种相关系数，包括皮尔逊、斯皮尔曼、肯德尔、偏相关、多项相关和多项序列相关。让我们逐一来看。

皮尔逊、斯皮尔曼和肯德尔相关系数

皮尔逊积矩相关系数评估两个定量变量之间线性关系的程度。斯皮尔曼秩相关系数评估两个秩次变量之间的关系程度。肯德尔 tau 也是一种非参数的秩相关度量。

`cor()`函数产生所有三个相关系数，而`cov()`函数提供协方差。有许多选项，但生成相关性的简化格式如下：

```
cor(x, use= , method= ) 
```

选项在表 7.2 中描述。

表 7.2 `cor/cov`选项

| 选项 | 描述 |
| --- | --- |
| `x` | 矩阵或数据框。 |
| `use` | 指定缺失数据的处理方式。选项有`all.obs`（假设没有缺失数据——缺失数据将产生错误）、`everything`（任何涉及缺失值的案例的相关性都将设置为`missing`）、`complete.obs`（逐行删除）和`pairwise.complete.obs`（成对删除）。 |
| `method` | 指定相关类型。选项有`pearson`、`spearman`和`kendall`。 |

默认选项是`use="everything"`和`method="pearson"`。以下列表提供了一个示例。

列表 7.12 协方差和相关系数

```
> states<- state.x77[,1:6]
> cov(states)
           Population Income Illiteracy Life Exp  Murder  HS Grad
Population   19931684 571230    292.868 -407.842 5663.52 -3551.51
Income         571230 377573   -163.702  280.663 -521.89  3076.77
Illiteracy        293   -164      0.372   -0.482    1.58    -3.24
Life Exp         -408    281     -0.482    1.802   -3.87     6.31
Murder           5664   -522      1.582   -3.869   13.63   -14.55
HS Grad         -3552   3077     -3.235    6.313  -14.55    65.24

> cor(states)
           Population Income Illiteracy Life Exp Murder HS Grad
Population     1.0000  0.208      0.108   -0.068  0.344 -0.0985
Income         0.2082  1.000     -0.437    0.340 -0.230  0.6199
Illiteracy     0.1076 -0.437      1.000   -0.588  0.703 -0.6572
Life Exp      -0.0681  0.340     -0.588    1.000 -0.781  0.5822
Murder         0.3436 -0.230      0.703   -0.781  1.000 -0.4880
HS Grad       -0.0985  0.620     -0.657    0.582 -0.488  1.0000
> cor(states, method="spearman")
           Population Income Illiteracy Life Exp Murder HS Grad
Population      1.000  0.125      0.313   -0.104  0.346  -0.383
Income          0.125  1.000     -0.315    0.324 -0.217   0.510
Illiteracy      0.313 -0.315      1.000   -0.555  0.672  -0.655
Life Exp       -0.104  0.324     -0.555    1.000 -0.780   0.524
Murder          0.346 -0.217      0.672   -0.780  1.000  -0.437
HS Grad        -0.383  0.510     -0.655    0.524 -0.437   1.000
```

第一次调用产生方差和协方差，第二次提供皮尔逊积矩相关系数，第三次产生斯皮尔曼秩相关系数。例如，您可以看到收入与高中毕业率之间存在强烈的正相关，而文盲率与预期寿命之间存在强烈的负相关。

注意，默认情况下您会得到方阵（所有变量与所有其他变量交叉）。您也可以生成非方阵，如下面的示例所示：

```
> x <- states[,c("Population", "Income", "Illiteracy", "HS Grad")]
> y <- states[,c("Life Exp", "Murder")]
> cor(x,y)
           Life Exp Murder
Population   -0.068  0.344
Income        0.340 -0.230
Illiteracy   -0.588  0.703
HS Grad       0.582 -0.488
```

当您对一组变量与另一组变量之间的关系感兴趣时，这个函数版本特别有用。请注意，结果不会告诉您相关性是否显著不同于 0（即，是否有足够的证据基于样本数据来得出结论，认为总体相关性不同于 0）。为此，您需要进行显著性测试（在 7.3.2 节中描述）。

部分相关

**部分相关**是指两个定量变量之间的相关关系，同时控制一个或多个其他定量变量。您可以使用`ggm`包中的`pcor()`函数来提供部分相关系数。`ggm`包默认未安装，因此请确保在首次使用时安装它。格式如下：

```
pcor(*u*, *S*)
```

其中`u`是一个数字向量，前两个数字是要相关变量的索引，其余数字是条件变量的索引（即部分化的变量）。`S`是变量之间的协方差矩阵。以下示例将有助于澄清这一点：

```
> library(ggm)
> colnames(states)
[1] "Population" "Income" "Illiteracy" "Life Exp" "Murder" "HS Grad"  
> pcor(c(1,5,2,3,6), cov(states))
[1] 0.346             
```

在这种情况下，0.346 是在控制收入、文盲率和高中毕业率（分别对应变量 `2`、`3` 和 `6`）的影响下，人口（变量 `1`）与谋杀率（变量 `5`）之间的相关系数。在社会科学中，使用偏相关系数是很常见的。

其他类型的相关性

`polycor` 包中的 `hetcor()` 函数可以计算包含数值变量之间的皮尔逊积矩相关系数、数值和有序变量之间的多项式相关系数、有序变量之间的多项式相关系数以及二元变量的四分位相关系数的异质相关矩阵。多项式、多项式和四分位相关系数假设有序或二元变量是从潜在的正态分布中派生出来的。有关更多信息，请参阅此包的文档。

### 7.3.2 测试相关性的显著性

一旦生成了相关系数，你如何测试它们的统计显著性？典型的零假设是没有关系（即总体中的相关系数为 0）。你可以使用 `cor.test()` 函数来测试单个皮尔逊、斯皮尔曼和肯德尔相关系数。简化格式如下

```
cor.test(*x*, *y*, alternative = , method = )
```

其中 *`x`* 和 *`y`* 是要相关联的变量，`alternative` 指定是双尾还是单尾测试（`"two.side"`、`"less"` 或 `"greater"`），而 `method` 指定要计算的相关类型（`"pearson"`、`"kendall"` 或 `"spearman"`）。当研究假设是总体相关系数小于 0 时，使用 `alternative="less"`。当研究假设是总体相关系数大于 0 时，使用 `alternative="greater"`。默认情况下，假设 `alternative="two.side"`（总体相关系数不等于 0）。以下列表提供了一个示例。

列表 7.13 测试相关系数的显著性

```
> cor.test(states[,3], states[,5])

        Pearson's product-moment correlation

data:  states[, 3] and states[, 5] 
t = 6.85, df = 48, p-value = 1.258e-08
alternative hypothesis: true correlation is not equal to 0 
95 percent confidence interval:
 0.528 0.821 
sample estimates:
  cor 
0.703 
```

此代码测试了零假设，即预期寿命与谋杀率之间的皮尔逊相关系数为 0。假设总体相关系数为 0，你预计在 1000 万次中只会看到 0.703 这样大的样本相关系数不到一次（即 `p=1.258e-08`）。鉴于这种情况不太可能发生，你将拒绝零假设，支持研究假设，即预期寿命与谋杀率之间的总体相关系数**不是** 0。

很遗憾，使用 `cor.test()` 你一次只能测试一个相关性。幸运的是，`psych` 包中提供的 `corr.test()` 函数允许你更进一步。`corr.test()` 函数可以生成皮尔逊、斯皮尔曼和肯德尔相关系数矩阵的相关性和显著性水平。以下列表提供了一个示例。

列表 7.14 通过 `corr.test()` 计算相关矩阵和显著性测试

```
> library(psych)
> corr.test(states, use="complete")

Call:corr.test(x = states, use = "complete")
Correlation matrix 
           Population Income Illiteracy Life Exp Murder HS Grad
Population       1.00   0.21       0.11    -0.07   0.34   -0.10    
Income           0.21   1.00      -0.44     0.34  -0.23    0.62
Illiteracy       0.11  -0.44       1.00    -0.59   0.70   -0.66
Life Exp        -0.07   0.34      -0.59     1.00  -0.78    0.58
Murder           0.34  -0.23       0.70    -0.78   1.00   -0.49
HS Grad         -0.10   0.62      -0.66     0.58  -0.49    1.00

Sample Size 
[1] 50

Probability value 
           Population Income Illiteracy Life Exp Murder HS Grad
Population       0.00   0.15       0.46     0.64   0.01     0.5     
Income           0.15   0.00       0.00     0.02   0.11     0.0
Illiteracy       0.46   0.00       0.00     0.00   0.00     0.0
Life Exp         0.64   0.02       0.00     0.00   0.00     0.0
Murder           0.01   0.11       0.00     0.00   0.00     0.0
HS Grad          0.50   0.00       0.00     0.00   0.00     0.0        
```

`use=`选项可以是`"pairwise"`或`"complete"`（分别表示成对或列表删除缺失值）。`method=`选项是`"pearson"`（默认值）、`"spearman"`或`"kendall"`。在这里，你可以看到文盲率和预期寿命之间的相关性（-0.59）与零显著不同（p=0.00），这表明随着文盲率的上升，预期寿命往往会下降。然而，人口规模和高中毕业率之间的相关性（-0.10）与 0 没有显著差异（p=0.5）。

其他显著性测试

在 7.4.1 节中，我们研究了部分相关。`psych`包中的`pcor.test()`函数可以用来测试在控制一个或多个额外变量的情况下，两个变量的条件独立性，假设多元正态性。格式是

```
pcor.test(*r*, *q*, *n*)
```

其中*`r`*是由`pcor()`函数产生的部分相关，*`q`*是正在控制的变量数量，*`n`*是样本大小。

在离开这个主题之前，我应该提到，`psych`包中的`r.test()`函数也提供了一些有用的显著性测试。该函数可以用来测试以下内容：

+   相关系数的显著性

+   两个独立相关系数之间的差异

+   共享单个变量的两个相关系数之间的差异

+   基于完全不同变量的两个相关系数之间的差异

有关详细信息，请参阅`help(r.test)`。

### 7.3.3 可视化相关性

通过散点图和散点图矩阵可以可视化相关性背后的双变量关系，而相关图提供了一种独特且强大的方法，以有意义的方式比较大量相关系数。第十一章涵盖了这两者。

## 7.4 t 检验

研究中最常见的活动是比较两组。接受新药的患者是否比使用现有药物的患者显示出更大的改善？一个制造过程是否比另一个制造过程产生的缺陷更少？两种教学方法中哪一种最具有成本效益？如果你的结果变量是分类的，你可以使用第 7.3 节中描述的方法。在这里，我们将重点关注结果变量是连续且假设为正态分布的组间比较。

对于这个示例，我们将使用与`MASS`包一起分发的`UScrime`数据集。它包含了关于 1960 年 47 个美国州惩罚制度对犯罪率影响的信息。感兴趣的结果变量将是`Prob`（监禁的概率）、`U1`（14-24 岁城市男性的失业率）和`U2`（35-39 岁城市男性的失业率）。分类变量`So`（南方州的指示变量）将作为分组变量。数据已被原始作者进行了缩放。（我考虑将这一节命名为“旧南方的犯罪与惩罚”，但更冷静的头脑占了上风。）

### 7.4.1 独立 t 检验

在南方犯罪你是否更有可能被监禁？感兴趣的比较是南方各州与非南方各州，因变量是被监禁的概率。可以使用双组独立 t 检验来检验两个总体均值相等的假设。在这里，你假设两组是独立的，并且数据是从正态总体中抽取的。格式可以是

```
t.test(*y ~ x, data*) 
```

其中 *`y`* 是数值型，*`x`* 是二元变量，或者

```
t.test(*y1, y2*)
```

其中 *`y1`* 和 *`y2`* 是数值向量（每个组的因变量）。可选的 `data` 参数指的是包含变量的矩阵或数据框。与大多数统计软件包不同，默认测试假设方差不等，并应用威尔士自由度修正。你可以添加 `var.equal=TRUE` 选项来指定方差相等和合并方差估计。默认情况下，假设双尾备择假设（即，均值不同，但方向未指定）。你可以添加选项 `alternative="less"` 或 `alternative="greater"` 来指定方向性测试。

以下代码使用双尾测试且不假设方差相等，比较南方（组 1）和非南方（组 0）各州被监禁的概率：

```
> library(MASS)
> t.test(Prob ~ So, data=UScrime)

        Welch Two Sample t-test

data:  Prob by So 
t = -3.8954, df = 24.925, p-value = 0.0006506                           
alternative hypothesis: true difference in means is not equal to 0 
95 percent confidence interval:
 -0.03852569 -0.01187439 
sample estimates:
mean in group 0 mean in group 1 
     0.03851265      0.06371269
```

你可以拒绝南方各州和非南方各州被监禁概率相等的假设（p < .001）。

注意：因为因变量是比例，你可能尝试在执行 t 检验之前将其转换为正态分布。在当前情况下，所有合理的因变量转换（`Y/1-Y`，`log(Y/1-Y)`，`arcsin(Y)`，和 `arcsin(sqrt(Y))`）都会得出相同的结论。第八章详细介绍了转换。

### 7.4.2 相关 t 检验

作为第二个例子，你可能想知道年轻男性（14–24 岁）的失业率是否高于老年男性（35–39 岁）。在这种情况下，两组并不独立。你不会期望阿拉巴马州年轻和老年男性的失业率无关。当两组的观测值相关时，你有一个相关组设计。前后比较或重复测量设计也会产生相关组。

相关 t 检验假设组间差异呈正态分布。在这种情况下，格式为

```
t.test(*y1*, *y2*, paired=TRUE) 
```

其中 `y1` 和 `y2` 是两个相关组的数值向量。结果如下：

```
> library(MASS)
> sapply(UScrime[c("U1","U2")], function(x)(c(mean=mean(x),sd=sd(x))))
       U1    U2
mean 95.5 33.98                
sd   18.0  8.45

> with(UScrime, t.test(U1, U2, paired=TRUE))

        Paired t-test

data:  U1 and U2 
t = 32.4066, df = 46, p-value < 2.2e-16
alternative hypothesis: true difference in means is not equal to 0 
95 percent confidence interval:
 57.67003 65.30870 
sample estimates:
mean of the differences 
               61.48936
```

均值差异（61.5）足够大，足以拒绝老年和年轻男性失业率均值相同的假设。年轻男性的失业率更高。实际上，如果总体均值相等，获得如此大的样本差异的概率小于 0.00000000000000022（即，2.2e–16）。

### 7.4.3 当有超过两个组时

如果你想比较超过两组，你可以假设数据是从正态总体中独立采样的，可以使用方差分析（ANOVA）。ANOVA 是一种涵盖许多实验和准实验设计的综合方法，因此它拥有自己的章节。你可以随时放弃这一节，跳转到第九章。

## 7.5 组间差异的非参数检验

如果你无法满足 t 检验或方差分析（ANOVA）的参数假设，你可以转向非参数方法。例如，如果结果变量严重偏斜或具有序数性质，你可能希望使用本节中的技术。

### 7.5.1 比较两组

如果两组是独立的，你可以使用 Wilcoxon 秩和检验（更通俗地称为 Mann-Whitney U 检验）来评估观察值是否来自相同的概率分布（即，一个群体中获得更高分数的概率是否大于另一个群体）。格式可以是

```
wilcox.test(*Y* ~ *X*, *data*) 
```

其中`y`是数值型，`x`是二元变量，或者

```
wilcox.test(*y1*, *y2*) 
```

其中`y1`和`y2`是每个组的输出变量。可选的`data`参数指的是包含变量的矩阵或数据框。默认是双尾检验。你可以添加`exact`选项以产生精确检验，以及`-alternative="less"`或`alternative="greater"`来指定方向性检验。

如果你将 Mann-Whitney U 检验应用于上一节中的监禁率问题，你会得到以下结果：

```
> with(UScrime, by(Prob, So, median))

So: 0
[1] 0.0382
-------------------- 
So: 1
[1] 0.0556

> wilcox.test(Prob ~ So, data=UScrime)

        Wilcoxon rank sum test

data:  Prob by So 
W = 81, p-value = 8.488e-05 
alternative hypothesis: true location shift is not equal to 0
```

再次，你可以拒绝南部和非南部各州监禁率相同的假设（p < .001）。

Wilcoxon 符号秩检验为相关样本 t 检验提供了一个非参数替代方案。在组别配对且正态性假设不成立的情况下适用。其格式与 Mann-Whitney U 检验相同，但需要添加`paired=TRUE`选项。让我们将其应用于上一节中的失业问题：

```
> sapply(UScrime[c("U1","U2")], median)
U1 U2 
92 34 

> with(UScrime, wilcox.test(U1, U2, paired=TRUE))

        Wilcoxon signed rank test with continuity correction

data:  U1 and U2 
V = 1128, p-value = 2.464e-09                                       
alternative hypothesis: true location shift is not equal to 0 
```

再次，你得出与配对 t 检验相同的结论。

在这种情况下，参数 t 检验及其非参数等价检验得出相同的结论。当 t 检验的假设合理时，参数检验更有效（如果存在差异，更有可能发现）。当假设明显不合理时（例如，等级排序数据），非参数检验更合适。

### 7.5.2 比较超过两组

当你比较超过两个组时，你必须转向其他方法。考虑 7.3 节中的 `state.x77` 数据集。它包含美国各州的人口、收入、文盲率、预期寿命、谋杀率和高中毕业率数据。如果你想比较该国四个地区（东北、南、北中、西）的文盲率怎么办？这被称为 *单因素设计*，对于解决这个问题，既有参数方法也有非参数方法。

如果无法满足 ANOVA 设计的假设，可以使用非参数方法来评估组间差异。如果组是独立的，Kruskal-Wallis 检验是一种有用的方法。如果组是相关的（例如，重复测量或随机区组设计），则 Friedman 检验更为合适。

Kruskal-Wallis 检验的格式为

```
kruskal.test(*y ~ A*, *data*)
```

其中 `y` 是数值结果变量，`A` 是具有两个或更多级别的分组变量（如果有两个级别，则等同于 Mann-Whitney U 检验）。对于 Friedman 检验，格式为

```
friedman.test(*y ~ A* | *B, data*)
```

其中 `y` 是数值结果变量，`A` 是分组变量，`B` 是识别匹配观察值的分组变量。在这两种情况下，`data` 是一个可选参数，指定包含变量的矩阵或数据框。

让我们应用 Kruskal-Wallis 检验来解决文盲问题。首先，你必须将地区标识符添加到数据集中。这些标识符包含在 R 基础安装中提供的 `state.region` 数据集中：

```
states <- data.frame(state.region, state.x77)
```

现在你可以应用这个测试：

```
> kruskal.test(Illiteracy ~ state.region, data=states)
        Kruskal-Wallis rank sum test
data:  states$Illiteracy by states$state.region 
Kruskal-Wallis chi-squared = 22.7, df = 3, p-value = 4.726e-05    
```

显著性检验表明，该国的四个地区（东北、南、北中、西）的文盲率并不相同（p <.001）。

虽然你可以拒绝无差异的零假设，但这个检验并不能告诉你 *哪些* 地区之间存在显著差异。为了回答这个问题，你可以使用 Wilcoxon 检验一次比较两组。一种更优雅的方法是应用多重比较程序，该程序在控制 I 类错误率（发现不存在差异的概率）的同时计算所有成对比较。我已经创建了一个名为 `wmc()` 的函数，可用于此目的。它使用 Wilcoxon 检验一次比较两组，并使用 `p.adj()` 函数调整概率值。

说实话，我在章节标题中相当夸张地使用了 *基本* 的定义，但因为这个函数非常适合这里，我希望你能理解。你可以从 [`rkabacoff.com/RiA/wmc.R`](https://rkabacoff.com/RiA/wmc.R) 下载包含 `wmc()` 的文本文件。以下列表使用此函数比较了四个美国地区的文盲率。

列表 7.15 非参数多重比较

```
> source("https://rkabacoff.com/RiA/wmc.R")            ❶
> states <- data.frame(state.region, state.x77)
> wmc(Illiteracy ~ state.region, data=states, method="holm")

Descriptive Statistics                                 ❷

        West North Central Northeast South
n      13.00         12.00       9.0 16.00
median  0.60          0.70       1.1  1.75
mad     0.15          0.15       0.3  0.59

Multiple Comparisons (Wilcoxon Rank Sum Tests)         ❸
Probability Adjustment = holm

        Group.1       Group.2  W       p    
1          West North Central 88 8.7e-01    
2          West     Northeast 46 8.7e-01    
3          West         South 39 1.8e-02   *
4 North Central     Northeast 20 5.4e-02   .
5 North Central         South  2 8.1e-05 ***
6     Northeast         South 18 1.2e-02   *
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

❶ 访问函数

❷ 基本统计

❸ 成对比较

`source()`函数下载并执行定义`wmc()`函数的 R 脚本❶。该函数的格式为`wmc(``y` `~` `A``,` `data``,` `method``)`，其中`y`是数值结果变量，`A`是分组变量，`data`是包含这些变量的数据框，而`method`是用于限制 I 类错误的途径。列表 7.15 使用 Holm（1979）开发的一种调整方法，该方法提供了对家族错误率（在一系列比较中犯一个或多个 I 类错误的概率）的强控制。有关其他方法的描述，请参阅`help(p.adjust)`。

`wmc()`函数首先为每个组提供样本大小、中位数和中位数绝对偏差❷。西方地区的文盲率最低，而南方地区最高。然后该函数生成六个统计比较（西部与中部北部、西部与东北部、西部与南部、中部北部与东北部、中部北部与南部，以及东北部与南部）❸。你可以从双尾 p 值（`p`）中看出，南部与其他三个地区存在显著差异，而其他三个地区在 p < .05 水平上彼此之间没有差异。

## 7.6 可视化组间差异

在第 7.4 节和第 7.5 节中，我们探讨了比较组之间的统计方法。检查组间差异的视觉表现也是全面数据分析策略的一个关键部分。它允许你评估差异的大小，识别任何影响结果分布特征的因素（例如偏斜、双峰或异常值），并评估测试假设的适当性。R 提供了广泛的图形方法来比较组，包括在第 6.6 节中介绍的箱线图（简单、带缺口和小提琴图），在第 6.5 节中介绍的重叠核密度图，以及在第九章中讨论的用于在方差分析框架中可视化结果的图形方法。

## 摘要

+   描述性统计可以以数值方式描述定量变量的分布。R 中的许多包都提供了数据框的描述性统计。包的选择主要是一个个人偏好的问题。

+   频率表和交叉表总结了分类变量的分布。

+   t 检验和 Mann-Whitney U 检验可以用来比较两个组在定量结果上的差异。

+   可以使用卡方检验来评估两个分类变量之间的关联。相关系数用于评估两个定量变量之间的关联。

+   数据可视化通常应伴随数值摘要和统计测试。否则，你可能会错过数据的重要特征。
