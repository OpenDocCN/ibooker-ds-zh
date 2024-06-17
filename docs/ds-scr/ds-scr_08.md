# 第 7 章。假设与推断

> 被统计数据感动是真正聪明的人的标志。
> 
> 乔治·伯纳德·肖

我们会用所有这些统计数据和概率理论做什么呢？数据科学的*科学*部分经常涉及形成和测试关于数据及其生成过程的*假设*。

# 统计假设检验

作为数据科学家，我们经常想测试某个假设是否可能成立。对于我们来说，假设是一些断言，比如“这枚硬币是公平的”或“数据科学家更喜欢 Python 而不是 R”或“人们更有可能在看不见关闭按钮的烦人插页广告出现后，不阅读内容直接离开页面”。这些可以转化为关于数据的统计信息。在各种假设下，这些统计信息可以被视为来自已知分布的随机变量的观察结果，这使我们能够对这些假设的可能性进行陈述。

在经典设置中，我们有一个*零假设*，<math><msub><mi>H</mi> <mn>0</mn></msub></math> ，代表某些默认位置，以及我们想要与之比较的一些备选假设，<math><msub><mi>H</mi> <mn>1</mn></msub></math> 。我们使用统计数据来决定我们是否可以拒绝<math><msub><mi>H</mi> <mn>0</mn></msub></math> 作为虚假的。这可能通过一个例子更容易理解。

# 示例：抛硬币

想象我们有一枚硬币，我们想测试它是否公平。我们假设硬币有一定概率 *p* 出现正面，因此我们的零假设是硬币是公平的——也就是说，*p* = 0.5。我们将这个假设与备选假设 *p* ≠ 0.5 进行测试。

特别是，我们的测试将涉及抛硬币 *n* 次，并计算正面出现的次数 *X*。每次抛硬币都是伯努利试验，这意味着 *X* 是一个二项分布变量，我们可以用正态分布（如我们在 [第 6 章](ch06.html#probability) 中看到的）来近似它：

```py
from typing import Tuple
import math

def normal_approximation_to_binomial(n: int, p: float) -> Tuple[float, float]:
    """Returns mu and sigma corresponding to a Binomial(n, p)"""
    mu = p * n
    sigma = math.sqrt(p * (1 - p) * n)
    return mu, sigma
```

每当一个随机变量遊服从正态分布时，我们可以使用 `normal_cdf` 来确定其实现值在特定区间内或外的概率：

```py
from scratch.probability import normal_cdf

# The normal cdf _is_ the probability the variable is below a threshold
normal_probability_below = normal_cdf

# It's above the threshold if it's not below the threshold
def normal_probability_above(lo: float,
                             mu: float = 0,
                             sigma: float = 1) -> float:
    """The probability that an N(mu, sigma) is greater than lo."""
    return 1 - normal_cdf(lo, mu, sigma)

# It's between if it's less than hi, but not less than lo
def normal_probability_between(lo: float,
                               hi: float,
                               mu: float = 0,
                               sigma: float = 1) -> float:
    """The probability that an N(mu, sigma) is between lo and hi."""
    return normal_cdf(hi, mu, sigma) - normal_cdf(lo, mu, sigma)

# It's outside if it's not between
def normal_probability_outside(lo: float,
                               hi: float,
                               mu: float = 0,
                               sigma: float = 1) -> float:
    """The probability that an N(mu, sigma) is not between lo and hi."""
    return 1 - normal_probability_between(lo, hi, mu, sigma)
```

我们也可以反过来——找到非尾部区域或（对称的）包含一定概率水平的平均值的区间。例如，如果我们想找到一个以均值为中心且包含 60% 概率的区间，那么我们找到上下尾部各包含 20% 概率的截止点（留下 60%）：

```py
from scratch.probability import inverse_normal_cdf

def normal_upper_bound(probability: float,
                       mu: float = 0,
                       sigma: float = 1) -> float:
    """Returns the z for which P(Z <= z) = probability"""
    return inverse_normal_cdf(probability, mu, sigma)

def normal_lower_bound(probability: float,
                       mu: float = 0,
                       sigma: float = 1) -> float:
    """Returns the z for which P(Z >= z) = probability"""
    return inverse_normal_cdf(1 - probability, mu, sigma)

def normal_two_sided_bounds(probability: float,
                            mu: float = 0,
                            sigma: float = 1) -> Tuple[float, float]:
    """
 Returns the symmetric (about the mean) bounds
 that contain the specified probability
 """
    tail_probability = (1 - probability) / 2

    # upper bound should have tail_probability above it
    upper_bound = normal_lower_bound(tail_probability, mu, sigma)

    # lower bound should have tail_probability below it
    lower_bound = normal_upper_bound(tail_probability, mu, sigma)

    return lower_bound, upper_bound
```

特别是，假设我们选择抛硬币 *n* = 1,000 次。如果我们的公平假设成立，*X* 应该近似正态分布，均值为 500，标准差为 15.8：

```py
mu_0, sigma_0 = normal_approximation_to_binomial(1000, 0.5)
```

我们需要就*显著性*做出决策——我们愿意做*第一类错误*的程度，即拒绝<math><msub><mi>H</mi> <mn>0</mn></msub></math>即使它是真的。由于历史记载已经遗失，这种愿意通常设定为5%或1%。让我们选择5%。

考虑到如果*X*落在以下界限之外则拒绝<math><msub><mi>H</mi> <mn>0</mn></msub></math>的测试：

```py
# (469, 531)
lower_bound, upper_bound = normal_two_sided_bounds(0.95, mu_0, sigma_0)
```

假设*p*真的等于0.5（即<math><msub><mi>H</mi> <mn>0</mn></msub></math>为真），我们只有5%的机会观察到一个落在这个区间之外的*X*，这正是我们想要的显著性水平。换句话说，如果<math><msub><mi>H</mi> <mn>0</mn></msub></math>为真，那么大约有20次中有19次这个测试会给出正确的结果。

我们还经常关注测试的*功效*，即不犯*第二类错误*（“假阴性”）的概率，即我们未能拒绝<math><msub><mi>H</mi> <mn>0</mn></msub></math>，尽管它是错误的。为了衡量这一点，我们必须明确<math><msub><mi>H</mi> <mn>0</mn></msub></math>假设为假*的含义*。 （仅知道*p*不等于0.5并不能给我们关于*X*分布的大量信息。）特别是，让我们检查*p*真的是0.55的情况，这样硬币稍微偏向正面。

在这种情况下，我们可以计算测试的功效：

```py
# 95% bounds based on assumption p is 0.5
lo, hi = normal_two_sided_bounds(0.95, mu_0, sigma_0)

# actual mu and sigma based on p = 0.55
mu_1, sigma_1 = normal_approximation_to_binomial(1000, 0.55)

# a type 2 error means we fail to reject the null hypothesis,
# which will happen when X is still in our original interval
type_2_probability = normal_probability_between(lo, hi, mu_1, sigma_1)
power = 1 - type_2_probability      # 0.887
```

假设相反，我们的零假设是硬币不偏向正面，或者<math><mrow><mi>p</mi> <mo>≤</mo> <mn>0</mn> <mo>.</mo> <mn>5</mn></mrow></math>。在这种情况下，我们需要一个*单边测试*，当*X*远大于500时拒绝零假设，但当*X*小于500时不拒绝。因此，一个5%显著性测试涉及使用`normal_probability_below`找到概率分布中95%以下的截止值：

```py
hi = normal_upper_bound(0.95, mu_0, sigma_0)
# is 526 (< 531, since we need more probability in the upper tail)

type_2_probability = normal_probability_below(hi, mu_1, sigma_1)
power = 1 - type_2_probability      # 0.936
```

这是一个更强大的测试，因为当*X*低于469（如果<math><msub><mi>H</mi> <mn>1</mn></msub></math>为真的话，这几乎不太可能发生）时，它不再拒绝<math><msub><mi>H</mi> <mn>0</mn></msub></math>，而是在*X*在526到531之间时拒绝<math><msub><mi>H</mi> <mn>0</mn></msub></math>（如果<math><msub><mi>H</mi> <mn>1</mn></msub></math>为真的话，这有一定可能发生）。

# p-值

对前述测试的另一种思考方式涉及*p值*。我们不再基于某个概率截断选择边界，而是计算概率——假设<math><msub><mi>H</mi> <mn>0</mn></msub></math>为真——我们会看到一个至少与我们实际观察到的值一样极端的值的概率。

对于我们的双边测试，检验硬币是否公平，我们计算：

```py
def two_sided_p_value(x: float, mu: float = 0, sigma: float = 1) -> float:
    """
 How likely are we to see a value at least as extreme as x (in either
 direction) if our values are from an N(mu, sigma)?
 """
    if x >= mu:
        # x is greater than the mean, so the tail is everything greater than x
        return 2 * normal_probability_above(x, mu, sigma)
    else:
        # x is less than the mean, so the tail is everything less than x
        return 2 * normal_probability_below(x, mu, sigma)
```

如果我们看到530次正面朝上，我们会计算：

```py
two_sided_p_value(529.5, mu_0, sigma_0)   # 0.062
```

###### 注意

为什么我们使用`529.5`而不是`530`？这就是所谓的[*连续性校正*](http://en.wikipedia.org/wiki/Continuity_correction)。这反映了`normal_probability_between(529.5, 530.5, mu_0, sigma_0)`比`normal_probability_between(530, 531, mu_0, sigma_0)`更好地估计看到530枚硬币正面的概率。

相应地，`normal_probability_above(529.5, mu_0, sigma_0)`更好地估计了至少看到530枚硬币正面的概率。你可能已经注意到，我们在生成[图 6-4](ch06.html#make_hist_result)的代码中也使用了这个。

说服自己这是一个明智的估计的一种方法是通过模拟：

```py
import random

extreme_value_count = 0
for _ in range(1000):
    num_heads = sum(1 if random.random() < 0.5 else 0    # Count # of heads
                    for _ in range(1000))                # in 1000 flips,
    if num_heads >= 530 or num_heads <= 470:             # and count how often
        extreme_value_count += 1                         # the # is 'extreme'

# p-value was 0.062 => ~62 extreme values out of 1000
assert 59 < extreme_value_count < 65, f"{extreme_value_count}"
```

由于*p*-值大于我们的5%显著性水平，我们不拒绝零假设。如果我们看到532枚硬币正面，*p*-值将是：

```py
two_sided_p_value(531.5, mu_0, sigma_0)   # 0.0463
```

这比5%的显著性水平还要小，这意味着我们将拒绝零假设。这和之前完全一样的检验，只是统计学上的不同方法而已。

同样，我们会得到：

```py
upper_p_value = normal_probability_above
lower_p_value = normal_probability_below
```

对于我们的单边检验，如果我们看到525枚硬币正面，我们会计算：

```py
upper_p_value(524.5, mu_0, sigma_0) # 0.061
```

这意味着我们不会拒绝零假设。如果我们看到527枚硬币正面，计算将是：

```py
upper_p_value(526.5, mu_0, sigma_0) # 0.047
```

我们将拒绝零假设。

###### 警告

在使用`normal_probability_above`计算*p*-值之前，请确保你的数据大致服从正态分布。糟糕的数据科学充满了人们声称某些观察到的事件在随机发生的机会是百万分之一的例子，当他们真正的意思是“机会，假设数据是正态分布的”，如果数据不是的话，这是相当毫无意义的。

有各种检验正态性的统计方法，但即使是绘制数据也是一个很好的开始。

# 置信区间

我们一直在测试关于硬币正面概率*p*值的假设，这是未知“正面”分布的一个*参数*。在这种情况下，第三种方法是围绕参数的观察值构建一个*置信区间*。

例如，我们可以通过观察与每次翻转相对应的伯努利变量的平均值来估计不公平硬币的概率—如果是正面则为1，如果是反面则为0。如果我们在1,000次翻转中观察到525枚硬币正面，则我们估计*p*等于0.525。

我们对这个估计有多么*有信心*？嗯，如果我们知道*p*的确切值，那么中心极限定理（回顾[“中心极限定理”](ch06.html#central_limit_theorem)）告诉我们，这些伯努利变量的平均值应该近似正态分布，均值为*p*，标准差为：

```py
math.sqrt(p * (1 - p) / 1000)
```

在这里我们不知道*p*的值，所以我们使用我们的估计值：

```py
p_hat = 525 / 1000
mu = p_hat
sigma = math.sqrt(p_hat * (1 - p_hat) / 1000)   # 0.0158
```

这并不完全合理，但人们似乎仍然这样做。使用正态近似，我们得出“95%的置信度”下以下区间包含真实参数*p*的结论：

```py
normal_two_sided_bounds(0.95, mu, sigma)        # [0.4940, 0.5560]
```

###### 注意

这是关于*区间*而不是关于*p*的说法。你应该理解它是这样的断言：如果你重复进行实验很多次，那么95%的时间，“真实”的参数（每次都相同）会落在观察到的置信区间内（每次可能不同）。

特别是，我们不会得出硬币不公平的结论，因为0.5位于我们的置信区间内。

如果我们看到了540次正面，那么我们将会：

```py
p_hat = 540 / 1000
mu = p_hat
sigma = math.sqrt(p_hat * (1 - p_hat) / 1000) # 0.0158
normal_two_sided_bounds(0.95, mu, sigma) # [0.5091, 0.5709]
```

在这里，“公平硬币”的置信区间中不存在。（如果真的是公平硬币假设，它不会通过一个测试。）

# p-操纵

一个过程，只有5%的时间错误地拒绝原假设，按定义来说：

```py
from typing import List

def run_experiment() -> List[bool]:
    """Flips a fair coin 1000 times, True = heads, False = tails"""
    return [random.random() < 0.5 for _ in range(1000)]

def reject_fairness(experiment: List[bool]) -> bool:
    """Using the 5% significance levels"""
    num_heads = len([flip for flip in experiment if flip])
    return num_heads < 469 or num_heads > 531

random.seed(0)
experiments = [run_experiment() for _ in range(1000)]
num_rejections = len([experiment
                      for experiment in experiments
                      if reject_fairness(experiment)])

assert num_rejections == 46
```

这意味着，如果你试图找到“显著”结果，通常你可以找到。对数据集测试足够多的假设，几乎肯定会出现一个显著结果。移除正确的异常值，你可能会将*p*-值降低到0.05以下。（我们在[“相关性”](ch05.html#correlation)中做了类似的事情；你注意到了吗？）

这有时被称为[*p-操纵*](https://www.nature.com/news/scientific-method-statistical-errors-1.14700)，在某种程度上是“从*p*-值框架推断”的结果。一篇批评这种方法的好文章是[“地球是圆的”](http://www.iro.umontreal.ca/~dift3913/cours/papers/cohen1994_The_earth_is_round.pdf)，作者是雅各布·科恩。

如果你想进行良好的*科学*研究，你应该在查看数据之前确定你的假设，清理数据时不要考虑假设，并且要记住*p*-值并不能替代常识。（另一种方法在[“贝叶斯推断”](#bayesian_inference)中讨论。）

# 例如：运行A/B测试

你在DataSciencester的主要职责之一是体验优化，这是一个试图让人们点击广告的委婉说法。你的一个广告客户开发了一种新的面向数据科学家的能量饮料，广告部副总裁希望你帮助选择广告A（“味道棒！”）和广告B（“更少偏见！”）之间的区别。

作为*科学家*，你决定通过随机向站点访问者展示两个广告，并跟踪点击每个广告的人数来运行*实验*。

如果在1,000个A观众中有990个点击他们的广告，而在1,000个B观众中只有10个点击他们的广告，你可以相当有信心认为A是更好的广告。但如果差异不那么明显呢？这就是你会使用统计推断的地方。

假设<math><msub><mi>N</mi> <mn>A</mn></msub></math>人看到广告A，并且其中<math><msub><mi>n</mi> <mn>A</mn></msub></math>人点击了它。我们可以将每次广告浏览视为伯努利试验，其中<math><msub><mi>p</mi> <mn>A</mn></msub></math>是某人点击广告A的概率。那么（如果<math><msub><mi>N</mi> <mn>A</mn></msub></math>很大，这里是这样），我们知道<math><mrow><msub><mi>n</mi> <mi>A</mi></msub> <mo>/</mo> <msub><mi>N</mi> <mi>A</mi></msub></mrow></math>大致上是一个均值为<math><msub><mi>p</mi> <mi>A</mi></msub></math>，标准差为<math><mrow><msub><mi>σ</mi> <mi>A</mi></msub> <mo>=</mo> <msqrt><mrow><msub><mi>p</mi> <mi>A</mi></msub> <mrow><mo>(</mo> <mn>1</mn> <mo>-</mo> <msub><mi>p</mi> <mi>A</mi></msub> <mo>)</mo></mrow> <mo>/</mo> <msub><mi>N</mi> <mi>A</mi></msub></mrow></msqrt></mrow></math>的正态随机变量。

同样，<math><mrow><msub><mi>n</mi> <mi>B</mi></msub> <mo>/</mo> <msub><mi>N</mi> <mi>B</mi></msub></mrow></math>大致上是一个均值为<math><msub><mi>p</mi> <mi>B</mi></msub></math>，标准差为<math><mrow><msub><mi>σ</mi> <mi>B</mi></msub> <mo>=</mo> <msqrt><mrow><msub><mi>p</mi> <mi>B</mi></msub> <mrow><mo>(</mo> <mn>1</mn> <mo>-</mo> <msub><mi>p</mi> <mi>B</mi></msub> <mo>)</mo></mrow> <mo>/</mo> <msub><mi>N</mi> <mi>B</mi></msub></mrow></msqrt></mrow></math>。我们可以用代码表示这个过程：

```py
def estimated_parameters(N: int, n: int) -> Tuple[float, float]:
    p = n / N
    sigma = math.sqrt(p * (1 - p) / N)
    return p, sigma
```

如果我们假设这两个正态分布是独立的（这似乎是合理的，因为单个伯努利试验应该是独立的），那么它们的差异也应该是均值为<math><mrow><msub><mi>p</mi> <mi>B</mi></msub> <mo>-</mo> <msub><mi>p</mi> <mi>A</mi></msub></mrow></math>，标准差为<math><msqrt><mrow><msubsup><mi>σ</mi> <mi>A</mi> <mn>2</mn></msubsup> <mo>+</mo> <msubsup><mi>σ</mi> <mi>B</mi> <mn>2</mn></msubsup></mrow></msqrt></math>。

###### 注意

这有点作弊。只有当您*知道*标准偏差时，数学才能完全工作。在这里，我们是从数据中估计它们，这意味着我们确实应该使用*t*分布。但是对于足够大的数据集，它接近标准正态分布，所以差别不大。

这意味着我们可以测试*零假设*，即<math><msub><mi>p</mi> <mi>A</mi></msub></math>和<math><msub><mi>p</mi> <mi>B</mi></msub></math>相同（即<math><mrow><msub><mi>p</mi> <mi>A</mi></msub> <mo>-</mo> <msub><mi>p</mi> <mi>B</mi></msub></mrow></math>为0），使用的统计量是：

```py
def a_b_test_statistic(N_A: int, n_A: int, N_B: int, n_B: int) -> float:
    p_A, sigma_A = estimated_parameters(N_A, n_A)
    p_B, sigma_B = estimated_parameters(N_B, n_B)
    return (p_B - p_A) / math.sqrt(sigma_A ** 2 + sigma_B ** 2)
```

应该大约是一个标准正态分布。

例如，如果“味道好”的广告在1,000次浏览中获得了200次点击，“更少偏见”的广告在1,000次浏览中获得了180次点击，则统计量等于：

```py
z = a_b_test_statistic(1000, 200, 1000, 180)    # -1.14
```

如果实际上平均值相等，观察到这么大差异的概率将是：

```py
two_sided_p_value(z)                            # 0.254
```

这个概率足够大，我们无法得出有太大差异的结论。另一方面，如果“更少偏见”的点击仅为150次，则有：

```py
z = a_b_test_statistic(1000, 200, 1000, 150)    # -2.94
two_sided_p_value(z)                            # 0.003
```

这意味着如果广告效果相同，我们看到这么大的差异的概率只有0.003。

# 贝叶斯推断

我们所看到的程序涉及对我们的*测试*做出概率陈述：例如，“如果我们的零假设成立，你观察到如此极端的统计量的概率只有3%”。

推理的另一种方法涉及将未知参数本身视为随机变量。分析员（也就是你）从参数的*先验分布*开始，然后使用观察到的数据和贝叶斯定理来获得参数的更新*后验分布*。与对测试进行概率判断不同，你对参数进行概率判断。

例如，当未知参数是概率时（如我们抛硬币的示例），我们通常使用*Beta分布*中的先验，该分布将其所有概率都放在0和1之间：

```py
def B(alpha: float, beta: float) -> float:
    """A normalizing constant so that the total probability is 1"""
    return math.gamma(alpha) * math.gamma(beta) / math.gamma(alpha + beta)

def beta_pdf(x: float, alpha: float, beta: float) -> float:
    if x <= 0 or x >= 1:          # no weight outside of [0, 1]
        return 0
    return x ** (alpha - 1) * (1 - x) ** (beta - 1) / B(alpha, beta)
```

一般来说，这个分布将其权重集中在：

```py
alpha / (alpha + beta)
```

而且`alpha`和`beta`越大，分布就越“紧密”。

例如，如果`alpha`和`beta`都为1，那就是均匀分布（以0.5为中心，非常分散）。如果`alpha`远大于`beta`，大部分权重都集中在1附近。如果`alpha`远小于`beta`，大部分权重都集中在0附近。[图7-1](#beta_priors)展示了几种不同的Beta分布。

![示例Beta分布。](assets/dsf2_0701.png)

###### 图7-1。示例Beta分布

假设我们对*p*有一个先验分布。也许我们不想对硬币是否公平发表立场，我们选择`alpha`和`beta`都等于1。或者我们非常相信硬币55%的时间会正面朝上，我们选择`alpha`等于55，`beta`等于45。

然后我们多次抛硬币，看到*h*次正面和*t*次反面。贝叶斯定理（以及一些在这里过于繁琐的数学）告诉我们*p*的后验分布再次是Beta分布，但参数为`alpha + h`和`beta + t`。

###### 注意

后验分布再次是Beta分布并非巧合。头数由二项式分布给出，而Beta是与二项式分布[*共轭先验*](http://www.johndcook.com/blog/conjugate_prior_diagram/)。这意味着每当您使用相应的二项式观察更新Beta先验时，您将得到一个Beta后验。

假设你抛了10次硬币，只看到3次正面。如果你从均匀先验开始（在某种意义上拒绝对硬币的公平性发表立场），你的后验分布将是一个Beta(4, 8)，中心在0.33左右。由于你认为所有概率都是同等可能的，你的最佳猜测接近观察到的概率。

如果你最初使用一个Beta(20, 20)（表达了一种认为硬币大致公平的信念），你的后验分布将是一个Beta(23, 27)，中心在0.46左右，表明修正后的信念可能硬币稍微偏向反面。

如果你最初假设一个Beta(30, 10)（表达了一种认为硬币有75%概率翻转为正面的信念），你的后验分布将是一个Beta(33, 17)，中心在0.66左右。在这种情况下，你仍然相信硬币有正面的倾向，但不像最初那样强烈。这三种不同的后验分布在[图 7-2](#beta_posteriors)中绘制出来。

![来自不同先验的后验分布。](assets/dsf2_0702.png)

###### 图 7-2。来自不同先验的后验分布

如果你不断地翻转硬币，先验的影响将越来越小，最终你将拥有（几乎）相同的后验分布，无论你最初选择了哪个先验。

例如，无论你最初认为硬币有多么偏向正面，看到2000次翻转中有1000次正面后，很难维持那种信念（除非你选择像Beta(1000000,1)这样的先验，这是疯子的行为）。

有趣的是，这使我们能够对假设进行概率性陈述：“基于先验和观察数据，硬币的正面概率在49%到51%之间的可能性仅为5%。”这在哲学上与“如果硬币是公平的，我们预期观察到极端数据的概率也仅为5%”这样的陈述有很大的不同。

使用贝叶斯推断来测试假设在某种程度上被认为是有争议的——部分原因是因为数学可以变得相当复杂，部分原因是因为选择先验的主观性质。我们在本书中不会进一步使用它，但了解这一点是很好的。

# 进一步探索

+   我们只是浅尝辄止地探讨了统计推断中你应该了解的内容。第五章末推荐的书籍详细讨论了这些内容。

+   Coursera提供了一门[数据分析与统计推断](https://www.coursera.org/course/statistics)课程，涵盖了许多这些主题。
