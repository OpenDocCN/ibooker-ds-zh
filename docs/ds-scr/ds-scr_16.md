# 第15章。多元回归

> 我不会看着问题并在里面加入不影响它的变量。
> 
> 比尔·帕塞尔

虽然副总统对你的预测模型印象深刻，但她认为你可以做得更好。因此，你收集了额外的数据：你知道每个用户每天工作的小时数，以及他们是否拥有博士学位。你希望利用这些额外数据来改进你的模型。

因此，你假设一个包含更多独立变量的线性模型：

<math alttext="minutes equals alpha plus beta 1 friends plus beta 2 work hours plus beta 3 phd plus epsilon" display="block"><mrow><mtext>minutes</mtext> <mo>=</mo> <mi>α</mi> <mo>+</mo> <msub><mi>β</mi> <mn>1</mn></msub> <mtext>friends</mtext> <mo>+</mo> <msub><mi>β</mi> <mn>2</mn></msub> <mtext>work</mtext> <mtext>hours</mtext> <mo>+</mo> <msub><mi>β</mi> <mn>3</mn></msub> <mtext>phd</mtext> <mo>+</mo> <mi>ε</mi></mrow></math>

显然，用户是否拥有博士学位不是一个数字——但是，正如我们在[第11章](ch11.html#machine_learning)中提到的，我们可以引入一个*虚拟变量*，对于拥有博士学位的用户设为1，没有的设为0，之后它与其他变量一样是数值化的。

# 模型

回想一下，在[第14章](ch14.html#simple_linear_regression)中，我们拟合了一个形式为：

<math alttext="y Subscript i Baseline equals alpha plus beta x Subscript i Baseline plus epsilon Subscript i" display="block"><mrow><msub><mi>y</mi> <mi>i</mi></msub> <mo>=</mo> <mi>α</mi> <mo>+</mo> <mi>β</mi> <msub><mi>x</mi> <mi>i</mi></msub> <mo>+</mo> <msub><mi>ε</mi> <mi>i</mi></msub></mrow></math>

现在想象每个输入<math><msub><mi>x</mi> <mi>i</mi></msub></math>不是单个数字，而是一个包含*k*个数字的向量，<math><mrow><msub><mi>x</mi> <mrow><mi>i</mi><mn>1</mn></mrow></msub> <mo>,</mo> <mo>...</mo> <mo>,</mo> <msub><mi>x</mi> <mrow><mi>i</mi><mi>k</mi></mrow></msub></mrow></math> 。多元回归模型假设：

<math alttext="y Subscript i Baseline equals alpha plus beta 1 x Subscript i Baseline 1 Baseline plus period period period plus beta Subscript k Baseline x Subscript i k Baseline plus epsilon Subscript i Baseline" display="block"><mrow><msub><mi>y</mi> <mi>i</mi></msub> <mo>=</mo> <mi>α</mi> <mo>+</mo> <msub><mi>β</mi> <mn>1</mn></msub> <msub><mi>x</mi> <mrow><mi>i</mi><mn>1</mn></mrow></msub> <mo>+</mo> <mo>.</mo> <mo>.</mo> <mo>.</mo> <mo>+</mo> <msub><mi>β</mi> <mi>k</mi></msub> <msub><mi>x</mi> <mrow><mi>i</mi><mi>k</mi></mrow></msub> <mo>+</mo> <msub><mi>ε</mi> <mi>i</mi></msub></mrow></math>

在多元回归中，参数向量通常称为*β*。我们希望这个向量包括常数项，可以通过在数据中添加一列1来实现：

```py
beta = [alpha, beta_1, ..., beta_k]
```

以及：

```py
x_i = [1, x_i1, ..., x_ik]
```

那么我们的模型就是：

```py
from scratch.linear_algebra import dot, Vector

def predict(x: Vector, beta: Vector) -> float:
    """assumes that the first element of x is 1"""
    return dot(x, beta)
```

在这种特殊情况下，我们的自变量`x`将是一个向量列表，每个向量如下所示：

```py
[1,    # constant term
 49,   # number of friends
 4,    # work hours per day
 0]    # doesn't have PhD
```

# 最小二乘模型的进一步假设

为了使这个模型（以及我们的解决方案）有意义，还需要一些进一步的假设。

第一个假设是*x*的列是*线性独立*的——没有办法将任何一个写成其他一些的加权和。如果这个假设失败，估计`beta`是不可能的。在一个极端情况下，想象我们在数据中有一个额外的字段`num_acquaintances`，对于每个用户都恰好等于`num_friends`。

然后，从任意`beta`开始，如果我们将`num_friends`系数增加*任意*量，并将相同量从`num_acquaintances`系数减去，模型的预测将保持不变。这意味着没有办法找到`num_friends`的*系数*。（通常这种假设的违反不会那么明显。）

第二个重要假设是*x*的列与误差<math><mi>ε</mi></math>不相关。如果这一点不成立，我们对`beta`的估计将会系统错误。

例如，在[第14章](ch14.html#simple_linear_regression)中，我们建立了一个模型，预测每增加一个朋友与额外0.90分钟的网站使用时间相关。

想象也是这种情况：

+   工作时间更长的人在网站上花费的时间较少。

+   拥有更多朋友的人 tend to work more hours.

换句话说，假设“实际”模型如下：

<math alttext="minutes equals alpha plus beta 1 friends plus beta 2 work hours plus epsilon" display="block"><mrow><mtext>minutes</mtext> <mo>=</mo> <mi>α</mi> <mo>+</mo> <msub><mi>β</mi> <mn>1</mn></msub> <mtext>friends</mtext> <mo>+</mo> <msub><mi>β</mi> <mn>2</mn></msub> <mtext>work</mtext> <mtext>hours</mtext> <mo>+</mo> <mi>ε</mi></mrow></math>

其中<math><msub><mi>β</mi> <mn>2</mn></msub></math>是负数，而且工作时间和朋友数量是正相关的。在这种情况下，当我们最小化单变量模型的误差时：

<math alttext="minutes equals alpha plus beta 1 friends plus epsilon" display="block"><mrow><mtext>minutes</mtext> <mo>=</mo> <mi>α</mi> <mo>+</mo> <msub><mi>β</mi> <mn>1</mn></msub> <mtext>friends</mtext> <mo>+</mo> <mi>ε</mi></mrow></math>

我们会低估<math><msub><mi>β</mi> <mn>1</mn></msub></math> 。

想象一下，如果我们使用单变量模型并使用“实际”值<math><msub><mi>β</mi> <mn>1</mn></msub></math> 进行预测会发生什么。（也就是说，这个值是通过最小化我们称之为“实际”模型的误差得到的。）预测值会倾向于对工作时间较长的用户过大，并且对工作时间较少的用户也稍微偏大，因为<math><mrow><msub><mi>β</mi> <mn>2</mn></msub> <mo><</mo> <mn>0</mn></mrow></math> 而我们“忘记”将其包含在内。由于工作时间与朋友数量呈正相关，这意味着对于朋友较多的用户，预测值往往过大，而对于朋友较少的用户，则稍微过大。

这样做的结果是，我们可以通过减少对<math><msub><mi>β</mi> <mn>1</mn></msub></math>的估计来减少（单变量模型中的）误差，这意味着误差最小化的<math><msub><mi>β</mi> <mn>1</mn></msub></math>小于“实际”值。也就是说，在这种情况下，单变量最小二乘解法会倾向于低估<math><msub><mi>β</mi> <mn>1</mn></msub></math>。而且，通常情况下，每当自变量与这些误差相关联时，我们的最小二乘解法都会给我们一个偏倚的<math><msub><mi>β</mi> <mn>1</mn></msub></math>估计。

# 拟合模型

就像我们在简单线性模型中所做的那样，我们会选择`beta`来最小化平方误差的和。手动找到一个确切的解决方案并不容易，这意味着我们需要使用梯度下降法。同样，我们希望最小化平方误差的和。误差函数与我们在[第14章](ch14.html#simple_linear_regression)中使用的几乎完全相同，只是不再期望参数`[alpha, beta]`，而是会接受任意长度的向量：

```py
from typing import List

def error(x: Vector, y: float, beta: Vector) -> float:
    return predict(x, beta) - y

def squared_error(x: Vector, y: float, beta: Vector) -> float:
    return error(x, y, beta) ** 2

x = [1, 2, 3]
y = 30
beta = [4, 4, 4]  # so prediction = 4 + 8 + 12 = 24

assert error(x, y, beta) == -6
assert squared_error(x, y, beta) == 36
```

如果你懂得微积分，计算梯度就很容易：

```py
def sqerror_gradient(x: Vector, y: float, beta: Vector) -> Vector:
    err = error(x, y, beta)
    return [2 * err * x_i for x_i in x]

assert sqerror_gradient(x, y, beta) == [-12, -24, -36]
```

否则，你需要相信我的话。

此时，我们准备使用梯度下降法找到最优的`beta`。让我们首先编写一个`least_squares_fit`函数，可以处理任何数据集：

```py
import random
import tqdm
from scratch.linear_algebra import vector_mean
from scratch.gradient_descent import gradient_step

def least_squares_fit(xs: List[Vector],
                      ys: List[float],
                      learning_rate: float = 0.001,
                      num_steps: int = 1000,
                      batch_size: int = 1) -> Vector:
    """
 Find the beta that minimizes the sum of squared errors
 assuming the model y = dot(x, beta).
 """
    # Start with a random guess
    guess = [random.random() for _ in xs[0]]

    for _ in tqdm.trange(num_steps, desc="least squares fit"):
        for start in range(0, len(xs), batch_size):
            batch_xs = xs[start:start+batch_size]
            batch_ys = ys[start:start+batch_size]

            gradient = vector_mean([sqerror_gradient(x, y, guess)
                                    for x, y in zip(batch_xs, batch_ys)])
            guess = gradient_step(guess, gradient, -learning_rate)

    return guess
```

然后我们可以将其应用到我们的数据中：

```py
from scratch.statistics import daily_minutes_good
from scratch.gradient_descent import gradient_step

random.seed(0)
# I used trial and error to choose num_iters and step_size.
# This will run for a while.
learning_rate = 0.001

beta = least_squares_fit(inputs, daily_minutes_good, learning_rate, 5000, 25)
assert 30.50 < beta[0] < 30.70  # constant
assert  0.96 < beta[1] <  1.00  # num friends
assert -1.89 < beta[2] < -1.85  # work hours per day
assert  0.91 < beta[3] <  0.94  # has PhD
```

在实践中，你不会使用梯度下降法来估计线性回归；你会使用超出本书范围的线性代数技术来得到精确的系数。如果你这样做，你会得到如下方程：

<math display="block"><mrow><mtext>minutes</mtext> <mo>=</mo> <mn>30</mn> <mo>.</mo> <mn>58</mn> <mo>+</mo> <mn>0</mn> <mo>.</mo> <mn>972</mn> <mtext>friends</mtext> <mo>-</mo> <mn>1</mn> <mo>.</mo> <mn>87</mn> <mtext>work</mtext> <mtext>hours</mtext> <mo>+</mo> <mn>0</mn> <mo>.</mo> <mn>923</mn> <mtext>phd</mtext></mrow></math>

这与我们找到的结果非常接近。

# 解释模型

模型中的系数代表每个因素的其他条件相等估计影响的总和。其他条件相等时，每增加一个朋友，每天会多花一分钟在网站上。其他条件相等时，用户工作日每增加一个小时，每天会少花约两分钟在网站上。其他条件相等时，拥有博士学位与每天在网站上多花一分钟相关联。

这并没有（直接）告诉我们关于变量之间互动的任何信息。有可能工作时间对于朋友多的人和朋友少的人有不同的影响。这个模型没有捕捉到这一点。处理这种情况的一种方法是引入一个新变量，即“朋友”和“工作时间”的*乘积*。这实际上允许“工作时间”系数随着朋友数量的增加而增加（或减少）。

或者可能是，你有更多的朋友，你在网站上花的时间就越多*直到一个点*，之后进一步的朋友导致你在网站上花费的时间减少。（也许有太多的朋友经验就太压倒性了？）我们可以尝试通过添加另一个变量，即朋友数量的*平方*，来捕捉这一点在我们的模型中。

一旦我们开始添加变量，就需要担心它们的系数是否“重要”。我们可以无限制地添加乘积、对数、平方和更高次方。

# 拟合优度

再次我们可以看一下R平方：

```py
from scratch.simple_linear_regression import total_sum_of_squares

def multiple_r_squared(xs: List[Vector], ys: Vector, beta: Vector) -> float:
    sum_of_squared_errors = sum(error(x, y, beta) ** 2
                                for x, y in zip(xs, ys))
    return 1.0 - sum_of_squared_errors / total_sum_of_squares(ys)
```

现在已经增加到0.68：

```py
assert 0.67 < multiple_r_squared(inputs, daily_minutes_good, beta) < 0.68
```

请记住，然而，向回归中添加新变量*必然*会增加R平方。毕竟，简单回归模型只是多重回归模型的特殊情况，其中“工作时间”和“博士学位”的系数都等于0。最佳的多重回归模型将至少有一个与该模型一样小的误差。

因此，在多重回归中，我们还需要看看系数的*标准误差*，这些标准误差衡量我们对每个<math><msub><mi>β</mi> <mi>i</mi></msub></math>的估计有多确定。整体回归可能非常适合我们的数据，但如果一些自变量相关（或不相关），它们的系数可能意义不大。

测量这些误差的典型方法始于另一个假设——误差<math><msub><mi>ε</mi> <mi>i</mi></msub></math>是独立的正态随机变量，均值为0，有一些共享的（未知）标准偏差<math><mi>σ</mi></math>。在这种情况下，我们（或者更可能是我们的统计软件）可以使用一些线性代数来找出每个系数的标准误差。它越大，我们对该系数的模型越不确定。不幸的是，我们没有设置好可以从头开始执行这种线性代数的工具。

# 偏离：自助法

想象我们有一个由某个（对我们来说未知的）分布生成的包含*n*个数据点的样本：

```py
data = get_sample(num_points=n)
```

在[第5章](ch05.html#statistics)中，我们编写了一个可以计算样本`中位数`的函数，我们可以将其用作对分布本身中位数的估计。

但我们对我们的估计有多自信呢？如果样本中所有数据点都非常接近100，那么实际中位数似乎也接近100。如果样本中大约一半的数据点接近0，而另一半接近200，则我们对中位数的估计就不太确定。

如果我们能够重复获得新样本，我们可以计算许多样本的中位数，并查看这些中位数的分布。通常我们做不到这一点。在这种情况下，我们可以通过从我们的数据中*有放回地*选择*n*个数据点来*bootstrap*新数据集。然后我们可以计算这些合成数据集的中位数：

```py
from typing import TypeVar, Callable

X = TypeVar('X')        # Generic type for data
Stat = TypeVar('Stat')  # Generic type for "statistic"

def bootstrap_sample(data: List[X]) -> List[X]:
    """randomly samples len(data) elements with replacement"""
    return [random.choice(data) for _ in data]

def bootstrap_statistic(data: List[X],
                        stats_fn: Callable[[List[X]], Stat],
                        num_samples: int) -> List[Stat]:
    """evaluates stats_fn on num_samples bootstrap samples from data"""
    return [stats_fn(bootstrap_sample(data)) for _ in range(num_samples)]
```

例如，考虑以下两个数据集：

```py
# 101 points all very close to 100
close_to_100 = [99.5 + random.random() for _ in range(101)]

# 101 points, 50 of them near 0, 50 of them near 200
far_from_100 = ([99.5 + random.random()] +
                [random.random() for _ in range(50)] +
                [200 + random.random() for _ in range(50)])
```

如果计算这两个数据集的`median`，两者都将非常接近100。然而，如果你看一下：

```py
from scratch.statistics import median, standard_deviation

medians_close = bootstrap_statistic(close_to_100, median, 100)
```

你将主要看到数字非常接近100。但如果你看一下：

```py
medians_far = bootstrap_statistic(far_from_100, median, 100)
```

你会看到很多接近0和很多接近200的数字。

第一组中位数的`标准偏差`接近0，而第二组中位数的则接近100：

```py
assert standard_deviation(medians_close) < 1
assert standard_deviation(medians_far) > 90
```

（这种极端情况下，通过手动检查数据很容易找到答案，但通常情况下这是不成立的。）

# 回归系数的标准误差

我们可以采取同样的方法来估计回归系数的标准误差。我们重复从我们的数据中取出一个`bootstrap_sample`，并基于该样本估计`beta`。如果与一个独立变量（比如`num_friends`）对应的系数在样本中变化不大，那么我们可以相信我们的估计相对较为精确。如果系数在样本中变化很大，那么我们就不能对我们的估计感到有信心。

唯一的微妙之处在于，在抽样之前，我们需要`zip`我们的`x`数据和`y`数据，以确保独立变量和因变量的相应值一起被抽样。这意味着`bootstrap_sample`将返回一个成对的列表`(x_i, y_i)`，我们需要重新组装成一个`x_sample`和一个`y_sample`：

```py
from typing import Tuple

import datetime

def estimate_sample_beta(pairs: List[Tuple[Vector, float]]):
    x_sample = [x for x, _ in pairs]
    y_sample = [y for _, y in pairs]
    beta = least_squares_fit(x_sample, y_sample, learning_rate, 5000, 25)
    print("bootstrap sample", beta)
    return beta

random.seed(0) # so that you get the same results as me

# This will take a couple of minutes!
bootstrap_betas = bootstrap_statistic(list(zip(inputs, daily_minutes_good)),
                                      estimate_sample_beta,
                                      100)
```

在此之后，我们可以估计每个系数的标准偏差。

```py
bootstrap_standard_errors = [
    standard_deviation([beta[i] for beta in bootstrap_betas])
    for i in range(4)]

print(bootstrap_standard_errors)

# [1.272,    # constant term, actual error = 1.19
#  0.103,    # num_friends,   actual error = 0.080
#  0.155,    # work_hours,    actual error = 0.127
#  1.249]    # phd,           actual error = 0.998
```

（如果我们收集了超过100个样本并使用了超过5,000次迭代来估计每个`beta`，我们可能会得到更好的估计，但我们没有那么多时间。）

我们可以使用这些来测试假设，比如“<math><msub><mi>β</mi> <mi>i</mi></msub></math>是否等于0？”在零假设<math><mrow><msub><mi>β</mi> <mi>i</mi></msub> <mo>=</mo> <mn>0</mn></mrow></math>（以及我们对<math><msub><mi>ε</mi> <mi>i</mi></msub></math>分布的其他假设）下，统计量：

<math alttext="t Subscript j Baseline equals ModifyingAbove beta Subscript j Baseline With caret slash ModifyingAbove sigma Subscript j Baseline With caret" display="block"><mrow><msub><mi>t</mi> <mi>j</mi></msub> <mo>=</mo> <mover accent="true"><msub><mi>β</mi> <mi>j</mi></msub> <mo>^</mo></mover> <mo>/</mo> <mover accent="true"><msub><mi>σ</mi> <mi>j</mi></msub> <mo>^</mo></mover></mrow></math>

其中，我们对<math><msub><mi>β</mi> <mi>j</mi></msub></math>的估计除以其标准误差的估计值，遵循“ <math><mrow><mi>n</mi> <mo>-</mo> <mi>k</mi></mrow></math>自由度”的*学生t分布*。

如果我们有一个`students_t_cdf`函数，我们可以为每个最小二乘系数计算*p*-值，以指示如果实际系数为0，则观察到这样的值的可能性有多大。不幸的是，我们没有这样的函数。（尽管如果我们不是从头开始工作，我们会有这样的函数。）

然而，随着自由度的增大，*t*-分布越来越接近于标准正态分布。在像这样的情况下，其中*n*远大于*k*，我们可以使用`normal_cdf`而仍然感觉良好：

```py
from scratch.probability import normal_cdf

def p_value(beta_hat_j: float, sigma_hat_j: float) -> float:
    if beta_hat_j > 0:
        # if the coefficient is positive, we need to compute twice the
        # probability of seeing an even *larger* value
        return 2 * (1 - normal_cdf(beta_hat_j / sigma_hat_j))
    else:
        # otherwise twice the probability of seeing a *smaller* value
        return 2 * normal_cdf(beta_hat_j / sigma_hat_j)

assert p_value(30.58, 1.27)   < 0.001  # constant term
assert p_value(0.972, 0.103)  < 0.001  # num_friends
assert p_value(-1.865, 0.155) < 0.001  # work_hours
assert p_value(0.923, 1.249)  > 0.4    # phd
```

（在不像这样的情况下，我们可能会使用知道如何计算*t*-分布以及如何计算确切标准误差的统计软件。）

虽然大多数系数的*p*-值非常小（表明它们确实是非零的），但“PhD”的系数与0的差异不“显著”，这使得“PhD”的系数很可能是随机的，而不是有意义的。

在更复杂的回归场景中，有时您可能希望对数据进行更复杂的假设检验，例如“至少一个<math><msub><mi>β</mi> <mi>j</mi></msub></math>非零”或“<math><msub><mi>β</mi> <mn>1</mn></msub></math>等于<math><msub><mi>β</mi> <mn>2</mn></msub></math> *和* <math><msub><mi>β</mi> <mn>3</mn></msub></math>等于<math><msub><mi>β</mi> <mn>4</mn></msub></math>。” 您可以使用*F-检验*来执行此操作，但遗憾的是，这超出了本书的范围。

# 正则化

在实践中，您经常希望将线性回归应用于具有大量变量的数据集。这会产生一些额外的复杂性。首先，您使用的变量越多，就越有可能将模型过度拟合到训练集。其次，非零系数越多，就越难以理解它们。如果目标是*解释*某种现象，那么具有三个因素的稀疏模型可能比稍好的具有数百个因素的模型更有用。

*正则化*是一种方法，其中我们将惩罚项添加到误差项中，随着`beta`的增大而增加。然后，我们最小化组合误差和惩罚。我们越重视惩罚项，就越能够阻止大的系数。

例如，在*岭回归*中，我们添加的惩罚与`beta_i`的平方和成比例（通常不惩罚`beta_0`，即常数项）：

```py
# alpha is a *hyperparameter* controlling how harsh the penalty is.
# Sometimes it's called "lambda" but that already means something in Python.
def ridge_penalty(beta: Vector, alpha: float) -> float:
    return alpha * dot(beta[1:], beta[1:])

def squared_error_ridge(x: Vector,
                        y: float,
                        beta: Vector,
                        alpha: float) -> float:
    """estimate error plus ridge penalty on beta"""
    return error(x, y, beta) ** 2 + ridge_penalty(beta, alpha)
```

然后我们可以按照通常的方式将其插入梯度下降：

```py
from scratch.linear_algebra import add

def ridge_penalty_gradient(beta: Vector, alpha: float) -> Vector:
    """gradient of just the ridge penalty"""
    return [0.] + [2 * alpha * beta_j for beta_j in beta[1:]]

def sqerror_ridge_gradient(x: Vector,
                           y: float,
                           beta: Vector,
                           alpha: float) -> Vector:
    """
 the gradient corresponding to the ith squared error term
 including the ridge penalty
 """
    return add(sqerror_gradient(x, y, beta),
               ridge_penalty_gradient(beta, alpha))
```

然后我们只需修改`least_squares_fit`函数，以使用`sqerror_ridge_gradient`而不是`sqerror_gradient`。（我不会在这里重复代码。）

将`alpha`设置为0后，就没有惩罚了，我们获得了与以前相同的结果：

```py
random.seed(0)
beta_0 = least_squares_fit_ridge(inputs, daily_minutes_good, 0.0,  # alpha
                                 learning_rate, 5000, 25)
# [30.51, 0.97, -1.85, 0.91]
assert 5 < dot(beta_0[1:], beta_0[1:]) < 6
assert 0.67 < multiple_r_squared(inputs, daily_minutes_good, beta_0) < 0.69
```

随着`alpha`的增加，拟合的好坏变得更差，但`beta`的大小变小：

```py
beta_0_1 = least_squares_fit_ridge(inputs, daily_minutes_good, 0.1,  # alpha
                                   learning_rate, 5000, 25)
# [30.8, 0.95, -1.83, 0.54]
assert 4 < dot(beta_0_1[1:], beta_0_1[1:]) < 5
assert 0.67 < multiple_r_squared(inputs, daily_minutes_good, beta_0_1) < 0.69

beta_1 = least_squares_fit_ridge(inputs, daily_minutes_good, 1,  # alpha
                                 learning_rate, 5000, 25)
# [30.6, 0.90, -1.68, 0.10]
assert 3 < dot(beta_1[1:], beta_1[1:]) < 4
assert 0.67 < multiple_r_squared(inputs, daily_minutes_good, beta_1) < 0.69

beta_10 = least_squares_fit_ridge(inputs, daily_minutes_good,10,  # alpha
                                  learning_rate, 5000, 25)
# [28.3, 0.67, -0.90, -0.01]
assert 1 < dot(beta_10[1:], beta_10[1:]) < 2
assert 0.5 < multiple_r_squared(inputs, daily_minutes_good, beta_10) < 0.6
```

特别是，“PhD”的系数在增加惩罚时消失，这与我们先前的结果相符，即其与0没有显著不同。

###### 注意

通常在使用这种方法之前，你应该`重新缩放`你的数据。毕竟，如果你将工作经验从年转换为世纪，其最小二乘系数将增加100倍，并且突然受到更严重的惩罚，尽管模型是相同的。

另一种方法是*套索回归*，它使用惩罚项：

```py
def lasso_penalty(beta, alpha):
    return alpha * sum(abs(beta_i) for beta_i in beta[1:])
```

虽然岭回归的惩罚在整体上缩小了系数，但套索惩罚倾向于强制系数为0，这使其非常适合学习稀疏模型。不幸的是，它不适合梯度下降，这意味着我们无法从头开始解决它。

# 进一步探索

+   回归背后有丰富而广泛的理论支持。这是另一个你应该考虑阅读教科书或至少大量维基百科文章的地方。

+   scikit-learn有一个[`linear_model`模块](https://scikit-learn.org/stable/modules/linear_model.html)，提供类似于我们的`LinearRegression`模型，以及岭回归、套索回归和其他类型的正则化。

+   [Statsmodels](https://www.statsmodels.org)是另一个Python模块，其中包含（除其他内容外）线性回归模型。
