# 第9章。顶点：数据分析的R语言

在本章中，我们将应用在R中学到的关于数据分析和可视化的知识来探索和测试*mpg*数据集中的关系。在这里，您将学习到几种新的R技术，包括如何进行t检验和线性回归。我们将从调用必要的包开始，从书籍存储库的*datasets*文件夹中读取*mpg.csv*，并选择感兴趣的列。迄今为止，我们还没有在本书中使用`tidymodels`，因此您可能需要安装它。

```py
library(tidyverse)
library(psych)
library(tidymodels)

# Read in the data, select only the columns we need
mpg <- read_csv('datasets/mpg/mpg.csv') %>%
  select(mpg, weight, horsepower, origin, cylinders)

#> -- Column specification -----------------------------------------------------
#> cols(
#>  mpg = col_double(),
#>  cylinders = col_double(),
#>  displacement = col_double(),
#>  horsepower = col_double(),
#>  weight = col_double(),
#>  acceleration = col_double(),
#>  model.year = col_double(),
#>  origin = col_character(),
#>  car.name = col_character()
#> )

head(mpg)
#> # A tibble: 6 x 5
#>     mpg weight horsepower origin cylinders
#>   <dbl>  <dbl>      <dbl> <chr>      <dbl>
#> 1    18   3504        130 USA            8
#> 2    15   3693        165 USA            8
#> 3    18   3436        150 USA            8
#> 4    16   3433        150 USA            8
#> 5    17   3449        140 USA            8
#> 6    15   4341        198 USA            8
```

# 探索性数据分析

探索性数据分析是探索数据时的良好起点。我们将使用`psych`的`describe()`函数来做这件事：

```py
describe(mpg)
#>            vars   n    mean     sd  median trimmed    mad  min
#> mpg           1 392   23.45   7.81   22.75   22.99   8.60    9
#> weight        2 392 2977.58 849.40 2803.50 2916.94 948.12 1613
#> horsepower    3 392  104.47  38.49   93.50   99.82  28.91   46
#> origin*       4 392    2.42   0.81    3.00    2.53   0.00    1
#> cylinders     5 392    5.47   1.71    4.00    5.35   0.00    3
#>               max  range  skew kurtosis    se
#> mpg          46.6   37.6  0.45    -0.54  0.39
#> weight     5140.0 3527.0  0.52    -0.83 42.90
#> horsepower  230.0  184.0  1.08     0.65  1.94
#> origin*       3.0    2.0 -0.91    -0.86  0.04
#> cylinders     8.0    5.0  0.50    -1.40  0.09
```

因为*origin*是一个分类变量，我们应该谨慎解释其描述性统计。 （实际上，`psych`使用`*`来表示此警告。）然而，我们可以安全地分析其单向频率表，我们将使用一个新的`dplyr`函数`count()`来完成：

```py
mpg %>%
  count(origin)
#> # A tibble: 3 x 2
#>   origin     n
#>   <chr>  <int>
#> 1 Asia      79
#> 2 Europe    68
#> 3 USA      245
```

我们从结果中的计数列*n*中得知，虽然大多数观察值都是美国汽车，但亚洲和欧洲汽车的观察值仍可能代表它们的子群体。

让我们进一步通过`cylinders`来细分这些计数，以得出一个二维频率表。我将结合`count()`和`pivot_wider()`来将`cylinders`显示在列中：

```py
mpg %>%
  count(origin, cylinders) %>%
  pivot_wider(values_from = n, names_from = cylinders)
#> # A tibble: 3 x 6
#>   origin   `3`   `4`   `6`   `5`   `8`
#>   <chr>  <int> <int> <int> <int> <int>
#> 1 Asia       4    69     6    NA    NA
#> 2 Europe    NA    61     4     3    NA
#> 3 USA       NA    69    73    NA   103
```

请记住，在R中，`NA`表示缺失值，这种情况是因为某些交叉部分没有观察到。

并非许多汽车拥有三缸或五缸引擎，*只有*美国汽车拥有八缸引擎。在分析数据时，数据集*不平衡*是常见的，其中某些水平的观察数量不成比例。通常需要特殊技术来对这类数据进行建模。要了解更多关于处理不平衡数据的信息，请查看彼得·布鲁斯等人的[*《数据科学实用统计》*](https://oreil.ly/jv8RS)，第2版（O’Reilly）。

我们还可以找出每个*origin*水平的描述性统计。首先，我们将使用`select()`选择感兴趣的变量，然后可以使用`psych`的`describeBy()`函数，将`groupBy`设置为`origin`：

```py
mpg %>%
  select(mpg, origin) %>%
  describeBy(group = 'origin')

#>  Descriptive statistics by group
#> origin: Asia
        vars  n  mean   sd median trimmed  mad min  max range
#> mpg        1 79 30.45 6.09   31.6   30.47 6.52  18 46.6  28.6
#> origin*    2 79  1.00 0.00    1.0    1.00 0.00   1  1.0   0.0
        skew kurtosis   se
#> mpg     0.01    -0.39 0.69
#> origin*  NaN      NaN 0.00

#> origin: Europe
        vars  n mean   sd median trimmed  mad  min  max range
#> mpg        1 68 27.6 6.58     26    27.1 5.78 16.2 44.3  28.1
#> origin*    2 68  1.0 0.00      1     1.0 0.00  1.0  1.0   0.0
        skew kurtosis  se
#> mpg     0.73     0.31 0.8
#> origin*  NaN      NaN 0.0

#> origin: USA
        vars   n  mean   sd median trimmed  mad min max range
#> mpg        1 245 20.03 6.44   18.5   19.37 6.67   9  39    30
#> origin*    2 245  1.00 0.00    1.0    1.00 0.00   1   1     0
        skew kurtosis   se
#> mpg     0.83     0.03 0.41
#> origin*  NaN      NaN 0.00
```

让我们进一步了解*origin*和*mpg*之间的潜在关系。我们将从可视化*mpg*分布的直方图开始，如[图 9-1](#mpg-hist)所示：

```py
ggplot(data = mpg, aes(x = mpg)) +
  geom_histogram()
#> `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![Histogram](assets/aina_0901.png)

###### 图 9-1。*mpg*的分布

现在，我们可以集中于通过*origin*来可视化*mpg*的分布。将所有三个*origin*水平叠加在一个直方图上可能会显得杂乱，因此类似于[图 9-2](#mpg-box)所示的箱线图可能更合适：

```py
ggplot(data = mpg, aes(x = origin, y = mpg)) +
  geom_boxplot()
```

![Boxplot](assets/aina_0902.png)

###### 图 9-2。*mpg*按*origin*分布

如果我们更喜欢将它们可视化为直方图，而不是搞乱，我们可以在 R 中使用 *facet* 图来实现。使用 `facet_wrap()` 将 `ggplot2` 图分成子图或 *facet*。我们将从 `~` 或波浪号运算符开始，后面是变量名。当您在 R 中看到波浪号时，请将其视为“按照”的意思。例如，我们在这里按 `origin` 进行分面直方图，结果显示在 [图 9-3](#mpg-facet) 中：

```py
# Histogram of mpg, facted by origin
ggplot(data = mpg, aes(x = mpg)) +
  geom_histogram() +
  facet_grid(~ origin)
#> `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![分面直方图](assets/aina_0903.png)

###### 图 9-3\. *mpg* 按 *origin* 的分布

# 假设检验

您可以继续使用这些方法探索数据，但让我们进入假设检验。特别是，我想知道美国车和欧洲车之间的里程是否有显著差异。让我们创建一个包含这些观测值的新数据框架；我们将使用它进行 t 检验。

```py
mpg_filtered <- filter(mpg, origin=='USA' | origin=='Europe')
```

## 独立样本 t 检验

R 包含了一个 `t.test()` 函数：我们需要用 `data` 参数指定数据的来源，并且还需要指定要测试的 *公式*。为此，我们将使用 `~` 运算符来设置独立和因变量之间的关系。因变量位于 `~` 的前面，后面是独立变量。同样，您可以将此符号解释为分析 `mpg` 通过 `origin` 的效果。

```py
# Dependent variable ~ ("by") independent variable
t.test(mpg ~ origin, data = mpg_filtered)
#> 	Welch Two Sample t-test
#>
#>     data:  mpg by origin
#>     t = 8.4311, df = 105.32, p-value = 1.93e-13
#>     alternative hypothesis: true difference in means is not equal to 0
#>     95 percent confidence interval:
#>     5.789361 9.349583
#>     sample estimates:
#>     mean in group Europe    mean in group USA
#>                 27.60294             20.03347
```

R 程序甚至明确说明了我们的备择假设是什么，并且在 p 值的同时还包括了置信区间（你可以看出这个程序是为统计分析而建立的）。根据 p 值，我们将拒绝原假设；似乎有证据表明均值存在差异。

现在让我们将注意力转向连续变量之间的关系。首先，我们将使用基本 R 中的 `cor()` 函数打印相关矩阵。我们仅针对 *mpg* 中的连续变量执行此操作：

```py
select(mpg, mpg:horsepower) %>%
  cor()
#>                   mpg     weight horsepower
#> mpg         1.0000000 -0.8322442 -0.7784268
#> weight     -0.8322442  1.0000000  0.8645377
#> horsepower -0.7784268  0.8645377  1.0000000
```

我们可以使用 `ggplot2` 来可视化，例如体重和里程之间的关系，如 [图 9-4](#mpg-scatter) 所示：

```py
ggplot(data = mpg, aes(x = weight,y = mpg)) +
  geom_point() + xlab('weight (pounds)') +
  ylab('mileage (mpg)') + ggtitle('Relationship between weight and mileage')
```

![散点图](assets/aina_0904.png)

###### 图 9-4\. *weight* 与 *mpg* 的散点图

或者，我们可以使用基本 R 中的 `pairs()` 函数生成所有变量组合的成对图，类似于相关矩阵的布局。[图 9-5](#pairplot) 是 *mpg* 中选定变量的成对图：

```py
select(mpg, mpg:horsepower) %>%
  pairs()
```

![成对图](assets/aina_0905.png)

###### 图 9-5\. 成对图

## 线性回归

现在我们准备进行线性回归，使用基本的 R `lm()` 函数（这是线性模型的缩写）。与 `t.test()` 类似，我们会指定数据集和一个公式。线性回归返回的输出比 t 检验要多一些，因此通常先将结果分配给 R 中的一个新对象，然后分别探索其各个元素。特别是 `summary()` 函数提供了回归模型的有用概览：

```py
mpg_regression <- lm(mpg ~ weight, data = mpg)
summary(mpg_regression)

#>     Call:
#>     lm(formula = mpg ~ weight, data = mpg)
#>
#>     Residuals:
#>         Min       1Q   Median       3Q      Max
#>     -11.9736  -2.7556  -0.3358   2.1379  16.5194
#>
#>     Coefficients:
#>                 Estimate Std. Error t value Pr(>|t|)
#>     (Intercept) 46.216524   0.798673   57.87   <2e-16 ***
#>     weight      -0.007647   0.000258  -29.64   <2e-16 ***
#>     ---
#>     Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
#>
#>     Residual standard error: 4.333 on 390 degrees of freedom
#>     Multiple R-squared:  0.6926,	Adjusted R-squared:  0.6918
#>     F-statistic: 878.8 on 1 and 390 DF,  p-value: < 2.2e-16
```

此输出应该看起来很熟悉。在这里，您将看到系数、p值和R平方等指标。再次强调，车重对里程的影响似乎是显著的。

最后但同样重要的是，我们可以在散点图上拟合这条回归线，方法是在我们的`ggplot()`函数中包含`geom_smooth()`，并将`method`设置为`lm`。这将生成[图9-6](#mpg-fit-scatter)：

```py
ggplot(data = mpg, aes(x = weight, y = mpg)) +
  geom_point() + xlab('weight (pounds)') +
  ylab('mileage (mpg)') + ggtitle('Relationship between weight and mileage') +
  geom_smooth(method = lm)
#> `geom_smooth()` using formula 'y ~ x'
```

![适合的散点图](assets/aina_0906.png)

###### 图9-6\. *mpg*数据集中*weight*与*mpg*的散点图及拟合回归线

## 训练/测试拆分和验证

[第5章](ch05.html#data-analytics-stack)简要回顾了机器学习如何与更广泛的数据处理相关联。一种由机器学习推广的技术是您可能在数据分析工作中遇到的*训练/测试拆分*。其核心思想是在数据的子集上*训练*模型，然后在另一个子集上*测试*模型。这确保了模型不仅适用于特定抽样观测，而是能够推广到更广泛的人群。数据科学家通常特别关注模型在测试数据上进行预测的效果。

让我们在R中将我们的*mpg*数据集拆分，对部分数据进行线性回归模型训练，然后在剩余数据上进行测试。为此，我们将使用`tidymodels`包。虽然不是`tidyverse`的一部分，但该包是基于相同原则构建的，因此与之兼容。

在[第2章](ch02.html#foundations-of-probability)中，您可能还记得，由于我们使用的是随机数，您在工作簿中看到的结果与书中记录的不同。因为我们将再次随机拆分我们的数据集，我们可能会遇到同样的问题。为了避免这种情况，我们可以设置R的随机数发生器的*种子*，这样每次生成的随机数序列都是相同的。这可以通过`set.seed()`函数来实现。您可以将其设置为任何数字；通常使用`1234`：

```py
set.seed(1234)
```

要开始拆分，我们可以使用名为`initial_split()`的适当命名的函数；然后，我们将使用`training()`和`testing()`函数将数据子集分为训练和测试数据集。

```py
mpg_split <- initial_split(mpg)
mpg_train <- training(mpg_split)
mpg_test <- testing(mpg_split)
```

默认情况下，`tidymodels`将数据的观测分成两组：75%的观测分配给训练组，其余分配给测试组。我们可以使用基本R中的`dim()`函数来确认每个数据集的行数和列数：

```py
dim(mpg_train)
#> [1] 294   5
dim(mpg_test)
#> [1] 98  5
```

在294个和98个观测值中，我们的训练和测试样本大小应足够大，以进行反映性统计推断。虽然这在机器学习中使用的大数据集中很少考虑，但当拆分数据时，充足的样本量可能是一个限制因素。

可以将数据拆分为比例为75/25之外的其他比例，使用拆分数据的特殊技术等等。有关更多信息，请查阅`tidymodels`文档；在您对回归分析更加熟悉之前，这些默认值都是可以的。

要构建我们的训练模型，我们将首先使用`linear_reg()`函数*指定*模型的类型，然后*拟合*它。`fit()`函数的输入对您应该很熟悉，除了这一次我们仅使用了*mpg*的训练子集。

```py
# Specify what kind of model this is
lm_spec <- linear_reg()

# Fit the model to the data
lm_fit <- lm_spec %>%
  fit(mpg ~ weight, data = mpg_train)
#> Warning message:
#> Engine set to `lm`.
```

您将从控制台输出中看到，来自基本R的`lm()`函数，您以前使用过的函数，被用作拟合模型的*引擎*。

我们可以使用`tidy()`函数获取训练模型的系数和p值，并使用`glance()`获取其性能指标（如R平方）。

```py
tidy(lm_fit)
#> # A tibble: 2 x 5
#>   term        estimate std.error statistic   p.value
#>   <chr>          <dbl>     <dbl>     <dbl>     <dbl>
#> 1 (Intercept) 47.3      0.894         52.9 1.37e-151
#> 2 weight      -0.00795  0.000290     -27.5 6.84e- 83
#>
glance(lm_fit)
#> # A tibble: 1 x 12
#>   r.squared adj.r.squared sigma statistic  p.value    df logLik   AIC
#>       <dbl>         <dbl> <dbl>     <dbl>    <dbl> <dbl>  <dbl> <dbl>
#> 1     0.721         0.720  4.23      754\. 6.84e-83     1  -840\. 1687.
#> # ... with 4 more variables: BIC <dbl>, deviance <dbl>,
#> #   df.residual <int>, nobs <int>
```

这很好，但是我们*真正*想知道的是当我们将该模型应用于新数据集时，该模型的表现如何；这就是测试拆分的作用。要在`mpg_test`上进行预测，我们将使用`predict()`函数。我还将使用`bind_cols()`将预测的Y值列添加到数据框中。此列默认将被称为`.pred`。

```py
mpg_results <- predict(lm_fit, new_data = mpg_test) %>%
  bind_cols(mpg_test)

mpg_results
#> # A tibble: 98 x 6
#>    .pred   mpg weight horsepower origin cylinders
#>    <dbl> <dbl>  <dbl>      <dbl> <chr>      <dbl>
#>  1  20.0    16   3433        150 USA            8
#>  2  16.7    15   3850        190 USA            8
#>  3  25.2    18   2774         97 USA            6
#>  4  30.3    27   2130         88 Asia           4
#>  5  28.0    24   2430         90 Europe         4
#>  6  21.0    19   3302         88 USA            6
#>  7  14.2    14   4154        153 USA            8
#>  8  14.7    14   4096        150 USA            8
#>  9  29.6    23   2220         86 USA            4
#> 10  29.2    24   2278         95 Asia           4
#> # ... with 88 more rows
```

现在我们已经将模型应用于这些新数据，让我们评估一下它的性能。例如，我们可以使用`rsq()`函数找到它的R平方。从我们的`mpg_results`数据框中，我们需要使用`truth`参数指定包含实际Y值的列，以及使用`estimate`列指定预测值。

```py
rsq(data = mpg_results, truth = mpg, estimate = .pred)
#> # A tibble: 1 x 3
#>   .metric .estimator .estimate
#>   <chr>   <chr>          <dbl>
#> 1 rsq     standard       0.606
```

在R平方为60.6%时，从训练数据集得出的模型解释了测试数据中相当数量的变异。

另一个常见的评估指标是均方根误差（RMSE）。您在[第4章](ch04.html#foundations-of-data-analytics)中了解到了*残差*的概念，即实际值与预测值之间的差异；RMSE是残差的标准偏差，因此是误差传播方式的估计值。`rmse()`函数返回RMSE。

```py
rmse(data = mpg_results, truth = mpg, estimate = .pred)
#> # A tibble: 1 x 3
#>   .metric .estimator .estimate
#>   <chr>   <chr>          <dbl>
#> 1 rmse    standard        4.65
```

因为它相对于因变量的比例，没有一种适用于所有情况的方式来评估RMSE，但在使用相同数据的两个竞争模型之间，更小的RMSE是首选。

`tidymodels`在R中提供了许多用于拟合和评估模型的技术。我们已经看过一个回归模型，它使用连续因变量，但也可以构建*分类*模型，其中因变量是分类的。这个包相对较新，所以可用的文献相对较少，但随着这个包的流行，预计会有更多的文献出现。

# 结论

当然，您可以进行更多的探索和测试来探索这些数据集和其他数据集之间的关系，但我们在这里所采取的步骤是一个坚实的开端。之前，您可以在Excel中进行这项工作并进行解释，现在您已经开始在R中进行了。

# 练习

请花点时间尝试使用R分析一个熟悉的数据集，采用熟悉的步骤。在[第四章](ch04.html#foundations-of-data-analytics)结束时，您曾练习在[书库](https://oreil.ly/egOx1)中的`ais`数据集的数据分析。该数据在R包`DAAG`中可用；尝试从那里安装和加载它（它作为对象`ais`可用）。请执行以下操作：

1.  通过性别（*sex*）可视化红细胞计数（*rcc*）的分布。

1.  两组性别之间的红细胞计数是否有显著差异？

1.  在这个数据集中产生相关变量的相关矩阵。

1.  可视化身高（*ht*）和体重（*wt*）之间的关系。

1.  将*ht*回归到*wt*。找到拟合回归线的方程。是否存在显著关系？体重（*wt*）解释的身高（*ht*）变异百分比是多少？

1.  将回归模型分割成训练集和测试集。测试模型的R平方和RMSE是多少？
