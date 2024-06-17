# 附录 A. 补充主题

# 使用SymPy进行LaTeX渲染

当你对数学符号更加熟悉时，将你的SymPy表达式显示为数学符号可能会很有帮助。

最快的方法是在SymPy中使用 `latex()` 函数对你的表达式，然后复制结果到LaTeX数学查看器中。

[示例 A-1](#pqebhBvrfC) 是一个将简单表达式转换为LaTeX字符串的示例。当然，我们也可以对导数、积分和其他SymPy操作的结果进行渲染成LaTeX。但让我们保持示例简单。

##### 示例 A-1\. 使用SymPy将表达式转换为LaTeX

```py
from sympy import *

x,y = symbols('x y')

z = x**2 / sqrt(2*y**3 - 1)

print(latex(z))

# prints
# \frac{x^{2}}{\sqrt{2 y^{3} - 1}}
```

这个 `\frac{x^{2}}{\sqrt{2 y^{3} - 1}}` 字符串是格式化的mathlatex，有多种工具和文档格式可以适应它。但为了简单地渲染mathlatex，可以去LaTeX方程编辑器。这里有两个我在线使用的不同的：

+   [Lagrida LaTeX 方程编辑器](https://latexeditor.lagrida.com)

+   [CodeCogs 方程编辑器](https://latex.codecogs.com)

在 [图 A-1](#daiTFCUMVR) 中，我使用Lagrida的LaTeX编辑器来渲染数学表达式。

![emds aa01](Images/emds_aa01.png)

###### 图 A-1\. 使用数学编辑器查看SymPy LaTeX输出

如果你想要省略复制/粘贴步骤，你可以将LaTeX直接附加为CodeCogs LaTeX编辑器URL的参数，就像[示例 A-2](#ECwErAvaVL)中展示的那样，它将在你的浏览器中显示渲染后的数学方程。

##### 示例 A-2\. 使用CodeCogs打开一个mathlatex渲染。

```py
import webbrowser
from sympy import *

x,y = symbols('x y')

z = x**2 / sqrt(2*y**3 - 1)

webbrowser.open("https://latex.codecogs.com/png.image?\dpi{200}" + latex(z))
```

如果你使用Jupyter，你也可以使用[插件来渲染mathlatex](https://oreil.ly/mWYf7)。

# 从头开始的二项分布

如果你想从头实现一个二项分布，这里是所有你需要的部分在 [示例 A-3](#HvgtibdvEk) 中。

##### 示例 A-3\. 从头构建一个二项分布

```py
# Factorials multiply consecutive descending integers down to 1
# EXAMPLE: 5! = 5 * 4 * 3 * 2 * 1
def factorial(n: int):
    f = 1
    for i in range(n):
        f *= (i + 1)
    return f

# Generates the coefficient needed for the binomial distribution
def binomial_coefficient(n: int, k: int):
    return factorial(n) / (factorial(k) * factorial(n - k))

# Binomial distribution calculates the probability of k events out of n trials
# given the p probability of k occurring
def binomial_distribution(k: int, n: int, p: float):
    return binomial_coefficient(n, k) * (p ** k) * (1.0 - p) ** (n - k)

# 10 trials where each has 90% success probability
n = 10
p = 0.9

for k in range(n + 1):
    probability = binomial_distribution(k, n, p)
    print("{0} - {1}".format(k, probability))
```

使用 `factorial()` 和 `binomial_coefficient()`，我们可以从头构建一个二项分布函数。阶乘函数将一系列整数从1到`n`相乘。例如，5! 的阶乘将是 <math alttext="1 asterisk 2 asterisk 3 asterisk 4 asterisk 5 equals 120"><mrow><mn>1</mn> <mo>*</mo> <mn>2</mn> <mo>*</mo> <mn>3</mn> <mo>*</mo> <mn>4</mn> <mo>*</mo> <mn>5</mn> <mo>=</mo> <mn>120</mn></mrow></math> 。

二项式系数函数允许我们从 *n* 个可能性中选择 *k* 个结果，而不考虑顺序。如果你有 *k* = 2 和 *n* = 3，那将产生集合 (1,2) 和 (1,2,3)。在这两组之间，可能的不同组合将是 (1,3)，(1,2) 和 (2,3)。因此，这将是一个二项式系数为3。当然，使用 `binomial_coefficient()` 函数，我们可以避免所有这些排列工作，而是使用阶乘和乘法来实现。

在实现`binomial_distribution()`时，请注意我们如何取二项式系数，并将其乘以成功概率`p`发生`k`次（因此指数）。然后我们乘以相反情况：失败概率`1.0 - p`在`n - k`次中发生。这使我们能够跨多次试验考虑事件发生与否的概率`p`。

# 从头开始构建贝塔分布

如果你想知道如何从头开始构建贝塔分布，你将需要重新使用我们用于二项分布的`factorial()`函数，以及我们在[第二章](ch02.xhtml#ch02)中构建的`approximate_integral()`函数。

就像我们在[第一章](ch01.xhtml#ch01)中所做的那样，我们在感兴趣的范围内根据曲线包装矩形，如图A-2所示。

![emds aa02](Images/emds_aa02.png)

###### 图A-2\. 包装矩形以找到面积/概率

这只是使用六个矩形；如果我们使用更多矩形，将会获得更高的准确性。让我们从头实现`beta_distribution()`并在0.9到1.0之间使用1,000个矩形进行积分，如示例A-4所示。

##### 示例A-4\. 从头开始的贝塔分布

```py
# Factorials multiply consecutive descending integers down to 1
# EXAMPLE: 5! = 5 * 4 * 3 * 2 * 1
def factorial(n: int):
    f = 1
    for i in range(n):
        f *= (i + 1)
    return f

def approximate_integral(a, b, n, f):
    delta_x = (b - a) / n
    total_sum = 0

    for i in range(1, n + 1):
        midpoint = 0.5 * (2 * a + delta_x * (2 * i - 1))
        total_sum += f(midpoint)

    return total_sum * delta_x

def beta_distribution(x: float, alpha: float, beta: float) -> float:
    if x < 0.0 or x > 1.0:
        raise ValueError("x must be between 0.0 and 1.0")

    numerator = x ** (alpha - 1.0) * (1.0 - x) ** (beta - 1.0)
    denominator = (1.0 * factorial(alpha - 1) * factorial(beta - 1)) / \
	    (1.0 * factorial(alpha + beta - 1))

    return numerator / denominator

greater_than_90 = approximate_integral(a=.90, b=1.0, n=1000,
    f=lambda x: beta_distribution(x, 8, 2))
less_than_90 = 1.0 - greater_than_90

print("GREATER THAN 90%: {}, LESS THAN 90%: {}".format(greater_than_90,
    less_than_90))
```

请注意，使用`beta_distribution()`函数时，我们提供一个给定的概率`x`，一个衡量成功的`alpha`值，以及一个衡量失败的`beta`值。该函数将返回观察到给定概率`x`的可能性。但是，要获得观察到概率`x`的概率，我们需要在`x`值范围内找到一个区域。

幸运的是，我们已经定义并准备好使用来自[第二章](ch02.xhtml#ch02)的`approximate_integral()`函数。我们可以计算成功率大于90%和小于90%的概率，如最后几行所示。

# 推导贝叶斯定理

如果你想理解为什么贝叶斯定理有效而不是听信我的话，让我们进行一个思想实验。假设我有一个10万人口的人群。将其与我们给定的概率相乘，以得到喝咖啡的人数和患癌症的人数：

<math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi>N</mi> <mo>=</mo> <mn>100,000</mn></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi>P</mi> <mo>(</mo> 喝咖啡的人 <mo>)</mo> <mo>=</mo> <mn>.65</mn></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi>P</mi> <mo>(</mo> 癌症 <mo>)</mo> <mo>=</mo> <mn>.005</mn></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mtext>喝咖啡的人数</mtext> <mo>=</mo> <mn>65,000</mn></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mtext>癌症患者数</mtext> <mo>=</mo> <mn>500</mn></mrow></mtd></mtr></mtable></math>

我们有65,000名咖啡饮用者和500名癌症患者。现在，这500名癌症患者中，有多少是咖啡饮用者？我们提供了条件概率<math alttext="upper P left-parenthesis Coffee vertical-bar Cancer right-parenthesis"><mrow><mi>P</mi> <mo>(</mo> 癌症|咖啡 <mo>)</mo></mrow></math>，我们可以将其乘以这500人，得到应有425名喝咖啡的癌症患者：

<math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi>P</mi> <mo>(</mo> 喝咖啡的人|癌症 <mo>)</mo> <mo>=</mo> <mn>.85</mn></mrow></mtd></mtr></mtable></math> <math display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mtext>喝咖啡的癌症患者数</mtext> <mo>=</mo> <mn>500</mn> <mo>×</mo> <mn>.85</mn> <mo>=</mo> <mn>425</mn></mrow></mtd></mtr></mtable></math>

现在，喝咖啡的人中得癌症的百分比是多少？我们需要除以哪两个数字？我们已经知道喝咖啡的人数 *和* 得癌症的人数。因此，我们将这个比例与总的喝咖啡的人数进行比较：

<math display="block"><mrow><mi>P</mi> <mrow><mo>(</mo> 癌症|喝咖啡的人 <mo>)</mo></mrow> <mo>=</mo> <mfrac><mrow><mtext>喝咖啡的癌症患者数</mtext></mrow> <mrow><mtext>喝咖啡的人数</mtext></mrow></mfrac></mrow></math><math display="block"><mrow><mi>P</mi> <mrow><mo>(</mo> 癌症|喝咖啡的人 <mo>)</mo></mrow> <mo>=</mo> <mfrac><mn>425</mn> <mrow><mn>65,000</mn></mrow></mfrac></mrow></math><math display="block"><mrow><mi>P</mi> <mo>(</mo> 癌症|喝咖啡的人 <mo>)</mo> <mo>=</mo> <mn>0.006538</mn></mrow></math>

稍等片刻，我们是不是刚刚颠倒了条件概率？是的！我们从<math alttext="upper P left-parenthesis Coffee Drinker vertical-bar Cancer right-parenthesis"><mrow><mi>P</mi> <mo>(</mo> <mtext>Coffee</mtext> <mtext>Drinker|Cancer</mtext> <mo>)</mo></mrow></math>开始，最终得到<math alttext="upper P left-parenthesis Cancer vertical-bar Coffee Drinker right-parenthesis"><mrow><mi>P</mi> <mo>(</mo> <mtext>Cancer|Coffee</mtext> <mtext>Drinker</mtext> <mo>)</mo></mrow></math>。通过取两个子集（65,000名咖啡饮用者和500名癌症患者），然后应用我们拥有的条件概率来应用联合概率，我们最终在我们的人口中得到了既饮咖啡又患癌症的425人。然后我们将这个数字除以咖啡饮用者的数量，得到患癌症的概率，假如一个人是咖啡饮用者。

但是贝叶斯定理在哪里呢？让我们专注于<math alttext="upper P left-parenthesis Cancer vertical-bar Coffee Drinker right-parenthesis"><mrow><mi>P</mi> <mo>(</mo> <mtext>Cancer|Coffee</mtext> <mtext>Drinker</mtext> <mo>)</mo></mrow></math>表达式，并用我们先前计算的所有表达式扩展它：

<math display="block"><mrow><mi>P</mi> <mrow><mo>(</mo> <mtext>Cancer|Coffee</mtext> <mtext>Drinker</mtext> <mo>)</mo></mrow> <mo>=</mo> <mfrac><mrow><mn>100,000</mn><mo>×</mo><mi>P</mi><mo>(</mo><mtext>Cancer</mtext><mo>)</mo><mo>×</mo><mi>P</mi><mo>(</mo><mtext>Coffee</mtext><mtext>Drinker|Cancer</mtext><mo>)</mo></mrow> <mrow><mn>100,000</mn><mo>×</mo><mi>P</mi><mo>(</mo><mtext>Coffee</mtext><mtext>Drinker</mtext><mo>)</mo></mrow></mfrac></mrow></math>

注意人口<math alttext="upper N"><mi>N</mi></math>为10万，在分子和分母中都存在，所以可以取消。这看起来现在熟悉吗？

<math alttext="upper P left-parenthesis Cancer vertical-bar Coffee Drinker right-parenthesis equals StartFraction upper P left-parenthesis Cancer right-parenthesis times upper P left-parenthesis Coffee Drinker vertical-bar Cancer right-parenthesis Over upper P left-parenthesis Coffee Drinker right-parenthesis EndFraction" display="block"><mrow><mi>P</mi> <mrow><mo>(</mo> <mtext>Cancer|Coffee</mtext> <mtext>Drinker</mtext> <mo>)</mo></mrow> <mo>=</mo> <mfrac><mrow><mi>P</mi><mo>(</mo><mtext>Cancer</mtext><mo>)</mo><mo>×</mo><mi>P</mi><mo>(</mo><mtext>Coffee</mtext><mtext>Drinker|Cancer</mtext><mo>)</mo></mrow> <mrow><mi>P</mi><mo>(</mo><mtext>Coffee</mtext><mtext>Drinker</mtext><mo>)</mo></mrow></mfrac></mrow></math>

毫无疑问，这应该与贝叶斯定理相符！

<math display="block"><mrow><mi>P</mi> <mrow><mo>(</mo> <mtext>A|B</mtext> <mo>)</mo></mrow> <mo>=</mo> <mfrac><mrow><mi>P</mi><mo>(</mo><mtext>B|A</mtext><mo>)</mo><mo>*</mo><mi>P</mi><mo>(</mo><mi>B</mi><mo>)</mo></mrow> <mrow><mi>P</mi><mo>(</mo><mi>A</mi><mo>)</mo></mrow></mfrac></mrow></math> <math display="block"><mrow><mi>P</mi> <mrow><mo>(</mo> <mtext>Cancer|Coffee</mtext> <mtext>Drinker</mtext> <mo>)</mo></mrow> <mo>=</mo> <mfrac><mrow><mi>P</mi><mo>(</mo><mtext>Cancer</mtext><mo>)</mo><mo>×</mo><mi>P</mi><mo>(</mo><mtext>Coffee</mtext><mtext>Drinker|Cancer</mtext><mo>)</mo></mrow> <mrow><mi>P</mi><mo>(</mo><mtext>Coffee</mtext><mtext>Drinker</mtext><mo>)</mo></mrow></mfrac></mrow></math>

如果你对贝叶斯定理感到困惑或者在其背后的直觉上挣扎，尝试基于提供的概率数据获取固定人口的子集。然后你可以追踪你的方式来颠倒一个条件概率。

# 从头开始的CDF和逆CDF

要计算正态分布的面积，我们当然可以使用我们在[第1章](ch01.xhtml#ch01)中学到的矩形填充法，之前在附录中应用于beta分布。它不需要累积密度函数（CDF），而只需在概率密度函数（PDF）下填充矩形。使用这种方法，我们可以找到金毛寻回犬体重在61至62磅之间的概率，如[示例A-5](#CjcMWPpGJu)所示，使用1,000个填充的矩形对正态PDF进行估算。

##### 示例 A-5\. Python中的正态分布函数

```py
import math

def normal_pdf(x: float, mean: float, std_dev: float) -> float:
    return (1.0 / (2.0 * math.pi * std_dev ** 2) ** 0.5) *
      math.exp(-1.0 * ((x - mean) ** 2 / (2.0 * std_dev ** 2)))

def approximate_integral(a, b, n, f):
    delta_x = (b - a) / n
    total_sum = 0

    for i in range(1, n + 1):
        midpoint = 0.5 * (2 * a + delta_x * (2 * i - 1))
        total_sum += f(midpoint)

    return total_sum * delta_x

p_between_61_and_62 = approximate_integral(a=61, b=62, n=7,
  f= lambda x: normal_pdf(x,64.43,2.99))

print(p_between_61_and_62) # 0.0825344984983386
```

这将为我们提供大约8.25%的概率，即金毛犬重量在61至62磅之间。如果我们想利用已经为我们集成且不需要任何矩形包装的CDF，我们可以像在[示例 A-6](#KuqFAIPtpi)中所示从头开始声明它。

##### 示例 A-6\. 在Python中使用称为`ppf()`的逆CDF

```py
import math

def normal_cdf(x: float, mean: float, std_dev: float) -> float:
    return (1 + math.erf((x - mean) / math.sqrt(2) / std_dev)) / 2

mean = 64.43
std_dev = 2.99

x = normal_cdf(66, mean, std_dev) - normal_cdf(62, mean, std_dev)

print(x)  # prints 0.49204501470628936
```

`math.erf()`被称为误差函数，通常用于计算累积分布。最后，要从头开始做逆CDF，您需要使用名为`erfinv()`的`erf()`函数的逆函数。[示例 A-7](#JORdwCMIRL) 使用从头编码的逆CDF计算了一千个随机生成的金毛犬重量。

##### 示例 A-7\. 生成随机金毛犬重量

```py
import random
from scipy.special import erfinv

def inv_normal_cdf(p: float, mean: float, std_dev: float):
    return mean + (std_dev * (2.0 ** 0.5) * erfinv((2.0 * p) - 1.0))

mean = 64.43
std_dev = 2.99

for i in range(0,1000):
    random_p = random.uniform(0.0, 1.0)
    print(inv_normal_cdf(random_p, mean, std_dev))
```

# 使用 e 预测随时间发生的事件概率

让我们看看您可能会发现有用的<math alttext="e"><mi>e</mi></math>的另一个用例。假设您是丙烷罐的制造商。显然，您不希望罐子泄漏，否则可能会在开放火焰和火花周围造成危险。测试新罐设计时，您的工程师报告说，在给定的一年中，有5%的机会它会泄漏。

你知道这已经是一个不可接受的高数字，但你想知道这种概率如何随时间复合。现在你问自己，“在2年内发生泄漏的概率是多少？5年？10年？”随着时间的推移，暴露的时间越长，看到罐子泄漏的概率不是越来越高吗？欧拉数再次派上用场！

<math display="block"><mrow><msub><mi>P</mi> <mrow><mi>l</mi><mi>e</mi><mi>a</mi><mi>k</mi></mrow></msub> <mo>=</mo> <mn>1.0</mn> <mo>-</mo> <msup><mi>e</mi> <mrow><mo>-</mo><mi>λ</mi><mi>T</mi></mrow></msup></mrow></math>

此函数建模随时间的事件发生概率，或者在本例中是罐子在*T*时间后泄漏的概率。 <math alttext="e"><mi>e</mi></math> 再次是欧拉数，lambda <math alttext="lamda"><mi>λ</mi></math> 是每个单位时间（每年）的失效率，*T* 是经过的时间量（年数）。

如果我们绘制此函数，其中*T*是我们的x轴，泄漏的概率是我们的y轴，λ = .05，[图 A-3](#HKCiJNGOam)显示了我们得到的结果。

![emds aa03](Images/emds_aa03.png)

###### 图 A-3\. 预测随时间泄漏概率

这是我们在Python中为<math alttext="lamda equals .05"><mrow><mi>λ</mi> <mo>=</mo> <mo>.</mo> <mn>05</mn></mrow></math>和<math alttext="upper T equals 5"><mrow><mi>T</mi> <mo>=</mo> <mn>5</mn></mrow></math>年建模此函数的方式，参见[示例 A-8](#RJmkBwKJPf)。

##### 示例 A-8\. 用于预测随时间泄漏概率的代码

```py
from math import exp

# Probability of leak in one year
p_leak = .05

# number of years
t = 5

# Probability of leak within five years
# 0.22119921692859512
p_leak_5_years = 1.0 - exp(-p_leak * t)

print("PROBABILITY OF LEAK WITHIN 5 YEARS: {}".format(p_leak_5_years))
```

2年后罐子失效的概率约为9.5%，5年约为22.1%，10年约为39.3%。随着时间的推移，罐子泄漏的可能性越来越大。我们可以将此公式推广为预测任何在给定期间内具有概率的事件，并查看该概率在不同时间段内的变化。

# 爬坡和线性回归

如果您发现从头开始构建机器学习中的微积分令人不知所措，您可以尝试一种更加蛮力的方法。让我们尝试一种 *爬山* 算法，其中我们通过添加一些随机值来随机调整 *m* 和 *b*，进行一定次数的迭代。这些随机值将是正或负的（这将使加法操作有效地成为减法），我们只保留能够改善平方和的调整。

但我们随机生成任何数字作为调整吗？我们更倾向于较小的移动，但偶尔也许会允许较大的移动。这样，我们主要是微小的调整，但偶尔如果需要的话会进行大幅跳跃。最好的工具来做到这一点是标准正态分布，均值为 0，标准差为 1。回想一下第 [3 章](ch03.xhtml#ch03) 中提到的标准正态分布在 0 附近有大量的值，而远离 0 的值（无论是负向还是正向）的概率较低，如 [图 A-4](#dvHeFXxfYw) 所示。

![emds aa04](Images/emds_aa04.png)

###### 图 A-4\. 标准正态分布中大多数值都很小且接近于 0，而较大的值在尾部出现的频率较低

回到线性回归，我们将从 0 或其他起始值开始设置 `m` 和 `b`。然后在一个 `for` 循环中进行 150,000 次迭代，我们将随机调整 `m` 和 `b`，通过添加从标准正态分布中采样的值。如果随机调整改善/减少了平方和，我们将保留它。但如果平方和增加了，我们将撤消该随机调整。换句话说，我们只保留能够改善平方和的调整。让我们在 [示例 A-9](#MLMseKbSoJ) 中看一看。

##### 示例 A-9\. 使用爬山算法进行线性回归

```py
from numpy.random import normal
import pandas as pd

# Import points from CSV
points = [p for p in pd.read_csv("https://bit.ly/2KF29Bd").itertuples()]

# Building the model
m = 0.0
b = 0.0

# The number of iterations to perform
iterations = 150000

# Number of points
n = float(len(points))

# Initialize with a really large loss
# that we know will get replaced
best_loss = 10000000000000.0

for i in range(iterations):

    # Randomly adjust "m" and "b"
    m_adjust = normal(0,1)
    b_adjust = normal(0,1)

    m += m_adjust
    b += b_adjust

    # Calculate loss, which is total sum squared error
    new_loss = 0.0
    for p in points:
        new_loss += (p.y - (m * p.x + b)) ** 2

    # If loss has improved, keep new values. Otherwise revert.
    if new_loss < best_loss:
        print("y = {0}x + {1}".format(m, b))
        best_loss = new_loss
    else:
        m -= m_adjust
        b -= b_adjust

print("y = {0}x + {1}".format(m, b))
```

您将看到算法的进展，但最终您应该得到一个大约为 `y = 1.9395722046562853x + 4.731834051245578` 的拟合函数。让我们验证这个答案。当我使用 Excel 或 Desmos 进行线性回归时，Desmos 给出了 `y = 1.93939x + 4.73333`。还不错！我几乎接近了！

为什么我们需要一百万次迭代？通过实验，我发现这足够多的迭代次数使得解决方案不再显著改善，并且收敛到了接近于最优的 `m` 和 `b` 值以最小化平方和。您将发现许多机器学习库和算法都有一个迭代次数的参数，它确切地做到这一点。您需要足够多的迭代次数使其大致收敛到正确的答案，但不要太多以至于在已经找到可接受的解决方案时浪费计算时间。

你可能会问为什么我将 `best_loss` 设置为一个极大的数字。我这样做是为了用我知道会被覆盖的值初始化最佳损失，并且它将与每次迭代的新损失进行比较，以查看是否有改进。我也可以使用正无穷 `float('inf')` 而不是一个非常大的数字。

# 爬山算法与逻辑回归

就像之前的线性回归示例一样，我们也可以将爬山算法应用于逻辑回归。如果你觉得微积分和偏导数一次性学习太过于复杂，可以再次使用这一技术。

爬山方法完全相同：用正态分布的随机值调整 `m` 和 `b`。然而，我们确实有一个不同的目标函数，即最大似然估计，在 [第 6 章](ch06.xhtml#ch06) 中讨论过。因此，我们只采用增加似然估计的随机调整，并在足够的迭代之后应该会收敛到一个拟合的逻辑回归模型。

所有这些都在 [示例 A-10](#WjhOVwMJCO) 中演示。

##### 示例 A-10\. 使用爬山算法进行简单的逻辑回归

```py
import math
import random

import numpy as np
import pandas as pd

# Desmos graph: https://www.desmos.com/calculator/6cb10atg3l

points = [p for p in pd.read_csv("https://tinyurl.com/y2cocoo7").itertuples()]

best_likelihood = -10_000_000
b0 = .01
b1 = .01

# calculate maximum likelihood

def predict_probability(x):
    p = 1.0 / (1.0001 + math.exp(-(b0 + b1 * x)))
    return p

for i in range(1_000_000):

    # Select b0 or b1 randomly, and adjust it randomly
    random_b = random.choice(range(2))

    random_adjust = np.random.normal()

    if random_b == 0:
        b0 += random_adjust
    elif random_b == 1:
        b1 += random_adjust

    # Calculate total likelihood
    true_estimates = sum(math.log(predict_probability(p.x)) \
       	for p in points if p.y == 1.0)
    false_estimates = sum(math.log(1.0 - predict_probability(p.x)) \
        for p in points if p.y == 0.0)

    total_likelihood = true_estimates + false_estimates

    # If likelihood improves, keep the random adjustment. Otherwise revert.
    if best_likelihood < total_likelihood:
        best_likelihood = total_likelihood
    elif random_b == 0:
        b0 -= random_adjust
    elif random_b == 1:
        b1 -= random_adjust

print("1.0 / (1 + exp(-({0} + {1}*x))".format(b0, b1))
print("BEST LIKELIHOOD: {0}".format(math.exp(best_likelihood)))
```

参见 [第 6 章](ch06.xhtml#ch06) 获取更多关于最大似然估计、逻辑函数以及我们为什么使用 `log()` 函数的详细信息。

# 线性规划简介

每个数据科学专业人士都应熟悉的技术是*线性规划*，它通过使用“松弛变量”来适应方程组来解决不等式系统。当你在线性规划系统中有离散整数或二进制变量（0 或 1）时，它被称为*整数规划*。当使用线性连续和整数变量时，它被称为*混合整数规划*。

尽管比数据驱动更多地依赖算法，线性规划及其变体可用于解决广泛的经典人工智能问题。如果将线性规划系统称为人工智能听起来有些靠不住，但许多供应商和公司都将其视为一种常见做法，因为这会增加其感知价值。

在实践中，最好使用众多现有的求解器库来为你执行线性规划，但是在本节末尾将提供如何从头开始执行的资源。在这些示例中，我们将使用 [PuLP](https://pypi.org/project/PuLP)，虽然 [Pyomo](https://www.pyomo.org) 也是一个选择。我们还将使用图形直觉，尽管超过三个维度的问题不能轻易地进行可视化。

这是我们的例子。你有两条产品线：iPac 和 iPac Ultra。iPac 每件产品盈利 $200，而 iPac Ultra 每件产品盈利 $300。

然而，装配线只能工作 20 小时，而制造 iPac 需要 1 小时，制造 iPac Ultra 需要 3 小时。

一天只能提供 45 套装备，而 iPac 需要 6 套，而 iPac Ultra 需要 2 套。

假设所有供应都将被销售，我们应该销售多少个 iPac 和 iPac Ultra 以实现利润最大化？

让我们首先注意第一个约束条件并将其拆分：

> …装配线只能工作 20 小时，生产一个 iPac 需要 1 小时，生产一个 iPac Ultra 需要 3 小时。

我们可以将其表示为一个不等式，其中*x*是 iPac 单元的数量，*y*是 iPac Ultra 单元的数量。两者必须为正，并且[图 A-5](#kCkjAwhqqP)显示我们可以相应地绘制图形。

<math alttext="x plus 3 y less-than-or-equal-to 20 left-parenthesis x greater-than-or-equal-to 0 comma y greater-than-or-equal-to 0 right-parenthesis" display="block"><mrow><mi>x</mi> <mo>+</mo> <mn>3</mn> <mi>y</mi> <mo>≤</mo> <mn>20</mn> <mo>(</mo> <mi>x</mi> <mo>≥</mo> <mn>0</mn> <mo>,</mo> <mi>y</mi> <mo>≥</mo> <mn>0</mn> <mo>)</mo></mrow></math>![emds aa05](Images/emds_aa05.png)

###### 图 A-5\. 绘制第一个约束条件

现在让我们看看第二个约束条件：

> 一天只能提供 45 个套件，其中 iPac 需要 6 个套件，而 iPac Ultra 需要 2 个套件。

我们还可以根据[图 A-6](#VnhgsinHGv)进行建模和绘图。

<math alttext="6 x plus 2 y less-than-or-equal-to 45 left-parenthesis x greater-than-or-equal-to 0 comma y greater-than-or-equal-to 0 right-parenthesis" display="block"><mrow><mn>6</mn> <mi>x</mi> <mo>+</mo> <mn>2</mn> <mi>y</mi> <mo>≤</mo> <mn>45</mn> <mo>(</mo> <mi>x</mi> <mo>≥</mo> <mn>0</mn> <mo>,</mo> <mi>y</mi> <mo>≥</mo> <mn>0</mn> <mo>)</mo></mrow></math>![emds aa06](Images/emds_aa06.png)

###### 图 A-6\. 绘制第二个约束条件

注意在[图 A-6](#VnhgsinHGv)中，现在这两个约束条件之间存在重叠。我们的解决方案位于该重叠区域内，我们称之为*可行区域*。最后，我们正在最大化我们的利润*Z*，下面给出了 iPac 和 iPac Ultra 的利润金额。

<math alttext="upper Z equals 200 x plus 300 y" display="block"><mrow><mi>Z</mi> <mo>=</mo> <mn>200</mn> <mi>x</mi> <mo>+</mo> <mn>300</mn> <mi>y</mi></mrow></math>

如果我们将这个函数表示为一条线，我们可以尽可能地增加*Z*，直到这条线不再位于可行区域内。然后我们注意在[图 A-7](#OPAqHuJwSt)中可视化的 x 和 y 值。

# Objective Function 的 Desmos 图

如果您需要以更交互式和动画的方式看到这个图形，请查看[Desmos 上的这张图](https://oreil.ly/RQMBT)。

![emds aa07](Images/emds_aa07.png)

###### 图 A-7\. 将我们的目标线增加到不再位于可行区域内

当利润*Z*尽可能地增加时，当那条线“刚好触及”可行区域时，您将落在可行区域的一个顶点或角上。该顶点提供了将最大化利润所需的 x 和 y 值，如[图 A-8](#bdonOArOSi)所示。

尽管我们可以使用NumPy和一堆矩阵运算来进行数值求解，但使用PuLP会更容易，如[示例 A-11](#MJjcOPJjvs)所示。请注意，`LpVariable`定义了要解决的变量。`LpProblem`是线性规划系统，使用Python操作符添加约束和目标函数。然后通过在`LpProblem`上调用`solve()`来求解变量。

![emds aa08](Images/emds_aa08.png)

###### A-8\. 线性规划系统的最大化目标

##### A-11\. 使用Python PuLP解决线性规划系统

```py
# GRAPH" https://www.desmos.com/calculator/iildqi2vt7

from pulp import *

# declare your variables
x = LpVariable("x", 0)   # 0<=x
y = LpVariable("y", 0) # 0<=y

# defines the problem
prob = LpProblem("factory_problem", LpMaximize)

# defines the constraints
prob += x + 3*y <= 20
prob += 6*x +2*y <= 45

# defines the objective function to maximize
prob += 200*x + 300*y

# solve the problem
status = prob.solve()
print(LpStatus[status])

# print the results x = 5.9375, y = 4.6875
print(value(x))
print(value(y))
```

也许你会想知道是否有意义构建5.9375和4.6875单位。如果您的变量能容忍连续值，线性规划系统会更高效，也许您可以在之后对它们进行四舍五入处理。但某些类型的问题绝对需要处理整数和二进制变量。

要将`x`和`y`变量强制为整数，可以使用`cat=LpInteger`参数，如[示例 A-12](#pqntuvKNRo)所示。

##### A-12\. 强制变量作为整数解决

```py
# declare your variables
x = LpVariable("x", 0, cat=LpInteger) # 0<=x
y = LpVariable("y", 0, cat=LpInteger) # 0<=y
```

从图形上来看，这意味着我们用离散点填充我们的可行区域，而不是连续的区域。我们的解决方案不一定会落在一个顶点上，而是接近顶点的点，如[图 A-9](#TWnQjpTKGu)所示。

线性规划中有一些特殊情况，如[图 A-10](#FnSsEsjOEf)所示。有时可能有多个解决方案。有时可能根本没有解决方案。

这只是线性规划的一个快速介绍示例，不幸的是，这本书里没有足够的空间来充分讨论这个话题。它可以用于一些出人意料的问题，包括调度受限资源（如工人、服务器任务或房间）、解决数独和优化金融投资组合。

![emds aa09](Images/emds_aa09.png)

###### A-9\. 离散线性规划系统

![emds aa10](Images/emds_aa10.png)

###### A-10\. 线性规划的特殊情况

如果您想了解更多，有一些很好的YouTube视频，包括[PatrickJMT](https://oreil.ly/lqeeR)和[Josh Emmanuel](https://oreil.ly/jAHWc)。如果您想深入研究离散优化，Pascal Van Hentenryck教授在Coursera上组织了一门极好的课程，[请点击这里](https://oreil.ly/aVGxY)。

# 使用scikit-learn进行MNIST分类器

[示例 A-13](#rqjdqTwRJL)展示了如何使用scikit-learn的神经网络进行手写数字分类。

##### A-13\. scikit-learn中的手写数字分类器神经网络示例

```py
import numpy as np
import pandas as pd
# load data
from sklearn.model_selection import train_test_split
from sklearn.neural_network import MLPClassifier

df = pd.read_csv('https://bit.ly/3ilJc2C', compression='zip', delimiter=",")

# Extract input variables (all rows, all columns but last column)
# Note we should do some linear scaling here
X = (df.values[:, :-1] / 255.0)

# Extract output column (all rows, last column)
Y = df.values[:, -1]

# Get a count of each group to ensure samples are equitably balanced
print(df.groupby(["class"]).agg({"class" : [np.size]}))

# Separate training and testing data
# Note that I use the 'stratify' parameter to ensure
# each class is proportionally represented in both sets
X_train, X_test, Y_train, Y_test = train_test_split(X, Y,
    test_size=.33, random_state=10, stratify=Y)

nn = MLPClassifier(solver='sgd',
                   hidden_layer_sizes=(100, ),
                   activation='logistic',
                   max_iter=480,
                   learning_rate_init=.1)

nn.fit(X_train, Y_train)

print("Training set score: %f" % nn.score(X_train, Y_train))
print("Test set score: %f" % nn.score(X_test, Y_test))

# Display heat map
import matplotlib.pyplot as plt
fig, axes = plt.subplots(4, 4)

# use global min / max to ensure all weights are shown on the same scale
vmin, vmax = nn.coefs_[0].min(), nn.coefs_[0].max()
for coef, ax in zip(nn.coefs_[0].T, axes.ravel()):
    ax.matshow(coef.reshape(28, 28), cmap=plt.cm.gray, vmin=.5 * vmin, vmax=.5 * vmax)
    ax.set_xticks(())
    ax.set_yticks(())

plt.show()
```
