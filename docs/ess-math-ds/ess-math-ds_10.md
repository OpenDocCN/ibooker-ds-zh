# 附录B. 练习答案

# 第1章

1.  62.6738是有理数，因为它的小数位数有限，可以表示为分数626738 / 10000。

1.  <math alttext="10 Superscript 7 Baseline 10 Superscript negative 5 Baseline equals 10 Superscript 7 plus negative 5 Baseline equals 10 squared equals 100"><mrow><msup><mn>10</mn> <mn>7</mn></msup> <msup><mn>10</mn> <mrow><mo>-</mo><mn>5</mn></mrow></msup> <mo>=</mo> <msup><mn>10</mn> <mrow><mn>7</mn><mo>+</mo><mo>-</mo><mn>5</mn></mrow></msup> <mo>=</mo> <msup><mn>10</mn> <mn>2</mn></msup> <mo>=</mo> <mn>100</mn></mrow></math>

1.  <math alttext="81 Superscript one-half"><msup><mn>81</mn> <mfrac><mn>1</mn> <mn>2</mn></mfrac></msup></math> = <math alttext="StartRoot left-parenthesis EndRoot 81 right-parenthesis equals 9"><mrow><msqrt><mo>(</mo></msqrt> <mrow><mn>81</mn> <mo>)</mo> <mo>=</mo> <mn>9</mn></mrow></mrow></math>

1.  <math alttext="25 Superscript three-halves Baseline equals left-parenthesis 25 Superscript 1 slash 2 Baseline right-parenthesis cubed equals 5 cubed equals 125"><mrow><msup><mn>25</mn> <mfrac><mn>3</mn> <mn>2</mn></mfrac></msup> <mo>=</mo> <msup><mrow><mo>(</mo><msup><mn>25</mn> <mrow><mn>1</mn><mo>/</mo><mn>2</mn></mrow></msup> <mo>)</mo></mrow> <mn>3</mn></msup> <mo>=</mo> <msup><mn>5</mn> <mn>3</mn></msup> <mo>=</mo> <mn>125</mn></mrow></math>

1.  结果金额为$1,161.47。Python脚本如下：

    ```py
    from math import exp

    p = 1000
    r = .05
    t = 3
    n = 12

    a = p * (1 + (r/n))**(n * t)

    print(a) # prints 1161.4722313334678
    ```

1.  结果金额为$1161.83。Python脚本如下：

    ```py
    from math import exp

    p = 1000 # principal, starting amount
    r = .05 # interest rate, by year
    t = 3.0 # time, number of years

    a = p * exp(r*t)

    print(a) # prints 1161.834242728283
    ```

1.  导数计算为6*x*，这使得*x*=3时的斜率为18。SymPy代码如下：

    ```py
    from sympy import *

    # Declare 'x' to SymPy
    x = symbols('x')

    # Now just use Python syntax to declare function
    f = 3*x**2 + 1

    # Calculate the derivative of the function
    dx_f = diff(f)
    print(dx_f) # prints 6*x
    print(dx_f.subs(x,3)) # 18
    ```

1.  曲线在0到2之间的面积为10。SymPy代码如下：

    ```py
    from sympy import *

    # Declare 'x' to SymPy
    x = symbols('x')

    # Now just use Python syntax to declare function
    f = 3*x**2 + 1

    # Calculate the integral of the function with respect to x
    # for the area between x = 0 and 2
    area = integrate(f, (x, 0, 2))

    print(area) # prints 10
    ```

# 第2章

1.  0.3 × 0.4 = 0.12；参见[“条件概率和贝叶斯定理”](ch02.xhtml#condprobsect)。

1.  (1 - 0.3) + 0.4 - (.03 × 0.4) = 0.98；参见[“联合概率”](ch02.xhtml#unionprobsect)，请记住我们在寻找*没有雨*，因此从1.0中减去该概率。

1.  0.3 × 0.2 = 0.06；参见[“条件概率和贝叶斯定理”](ch02.xhtml#condprobsect)。

1.  以下Python代码计算出0.822的答案，将50名以上未出现乘客的概率相加：

    ```py
    from scipy.stats import binom

    n = 137
    p = .40

    p_50_or_more_noshows = 0.0

    for x in range(50,138):
        p_50_or_more_noshows += binom.pmf(x, n, p)

    print(p_50_or_more_noshows) # 0.822095588147425
    ```

1.  使用SciPy中显示的Beta分布，获取到0.5的面积并从1.0中减去。结果约为0.98，因此这枚硬币极不可能是公平的。

    ```py
    from scipy.stats import beta

    heads = 8
    tails = 2

    p = 1.0 - beta.cdf(.5, heads, tails)

    print(p) # 0.98046875
    ```

# 第3章

1.  平均值为1.752，标准差约为0.02135。Python代码如下：

    ```py
    from math import sqrt

    sample = [1.78, 1.75, 1.72, 1.74, 1.77]

    def mean(values):
        return sum(values) /len(values)

    def variance_sample(values):
        mean = sum(values) / len(values)
        var = sum((v - mean) ** 2 for v in values) / len(values)
        return var

    def std_dev_sample(values):
        return sqrt(variance_sample(values))

    mean = mean(sample)
    std_dev = std_dev_sample(sample)

    print("MEAN: ", mean) # 1.752
    print("STD DEV: ", std_dev) # 0.02135415650406264
    ```

1.  使用CDF获取30到20个月之间的值，大约是0.06的面积。Python代码如下：

    ```py
    from scipy.stats import norm

    mean = 42
    std_dev = 8

    x = norm.cdf(30, mean, std_dev) - norm.cdf(20, mean, std_dev)

    print(x) # 0.06382743803380352
    ```

1.  有99%的概率，卷筒的平均丝径在1.7026到1.7285之间。Python代码如下：

    ```py
    from math import sqrt
    from scipy.stats import norm

    def critical_z_value(p, mean=0.0, std=1.0):
        norm_dist = norm(loc=mean, scale=std)
        left_area = (1.0 - p) / 2.0
        right_area = 1.0 - ((1.0 - p) / 2.0)
        return norm_dist.ppf(left_area), norm_dist.ppf(right_area)

    def ci_large_sample(p, sample_mean, sample_std, n):
        # Sample size must be greater than 30

        lower, upper = critical_z_value(p)
        lower_ci = lower * (sample_std / sqrt(n))
        upper_ci = upper * (sample_std / sqrt(n))

        return sample_mean + lower_ci, sample_mean + upper_ci

    print(ci_large_sample(p=.99, sample_mean=1.715588,
        sample_std=0.029252, n=34))
    # (1.7026658973748656, 1.7285101026251342)
    ```

1.  营销活动的p值为0.01888。Python代码如下：

    ```py
    from scipy.stats import norm

    mean = 10345
    std_dev = 552

    p1 = 1.0 - norm.cdf(11641, mean, std_dev)

    # Take advantage of symmetry
    p2 = p1

    # P-value of both tails
    # I could have also just multiplied by 2
    p_value = p1 + p2

    print("Two-tailed P-value", p_value)
    if p_value <= .05:
        print("Passes two-tailed test")
    else:
        print("Fails two-tailed test")

    # Two-tailed P-value 0.01888333596496139
    # Passes two-tailed test
    ```

# 第4章

1.  向量落在[2, 3]。Python代码如下：

    ```py
    from numpy import array

    v = array([1,2])

    i_hat = array([2, 0])
    j_hat = array([0, 1.5])

    # fix this line
    basis = array([i_hat, j_hat])

    # transform vector v into w
    w = basis.dot(v)

    print(w) # [2\. 3.]
    ```

1.  向量落在[0, -3]。Python代码如下：

    ```py
    from numpy import array

    v = array([1,2])

    i_hat = array([-2, 1])
    j_hat = array([1, -2])

    # fix this line
    basis = array([i_hat, j_hat])

    # transform vector v into w
    w = basis.dot(v)

    print(w) # [ 0, -3]
    ```

1.  行列式为2.0。Python代码如下：

    ```py
    import numpy as np
    from numpy.linalg import det

    i_hat = np.array([1, 0])
    j_hat = np.array([2, 2])

    basis = np.array([i_hat,j_hat]).transpose()

    determinant = det(basis)

    print(determinant) # 2.0
    ```

1.  是的，因为矩阵乘法允许我们将多个矩阵合并为一个表示单一转换的矩阵。

1.  *x* = 19.8，*y* = –5.4，*z* = –6。代码如下：

    ```py
    from numpy import array
    from numpy.linalg import inv

    A = array([
        [3, 1, 0],
        [2, 4, 1],
        [3, 1, 8]
    ])

    B = array([
        54,
        12,
        6
    ])

    X = inv(A).dot(B)

    print(X) # [19.8 -5.4 -6\. ]
    ```

1.  是的，它是线性相关的。尽管在NumPy中存在一些浮点不精确性，行列式有效地为0。

    ```py
    from numpy.linalg import det
    from numpy import array

    i_hat = array([2, 6])
    j_hat = array([1, 3])

    basis = array([i_hat, j_hat]).transpose()
    print(basis)

    determinant = det(basis)

    print(determinant) # -3.330669073875464e-16
    ```

    为了解决浮点问题，使用SymPy，你将得到0：

    ```py
    from sympy import *

    basis = Matrix([
        [2,1],
        [6,3]
    ])

    determinant = det(basis)

    print(determinant) # 0
    ```

# 第5章

1.  有很多工具和方法可以执行线性回归，就像我们在[第5章](ch05.xhtml#ch05)中学到的一样，但是这里使用scikit-learn进行解决。斜率是1.75919315，截距是4.69359655。

    ```py
    import pandas as pd
    import matplotlib.pyplot as plt
    from sklearn.linear_model import LinearRegression

    # Import points
    df = pd.read_csv('https://bit.ly/3C8JzrM', delimiter=",")

    # Extract input variables (all rows, all columns but last column)
    X = df.values[:, :-1]

    # Extract output column (all rows, last column)
    Y = df.values[:, -1]

    # Fit a line to the points
    fit = LinearRegression().fit(X, Y)

    # m = 1.75919315, b = 4.69359655
    m = fit.coef_.flatten()
    b = fit.intercept_.flatten()
    print("m = {0}".format(m))
    print("b = {0}".format(b))

    # show in chart
    plt.plot(X, Y, 'o') # scatterplot
    plt.plot(X, m*X+b) # line
    plt.show()
    ```

1.  我们得到高达0.92421的相关性和测试值23.8355，显著统计范围为±1.9844。这种相关性绝对有用且统计上显著。代码如下：

    ```py
    import pandas as pd

    # Read data into Pandas dataframe
    df = pd.read_csv('https://bit.ly/3C8JzrM', delimiter=",")

    # Print correlations between variables
    correlations = df.corr(method='pearson')
    print(correlations)

    # OUTPUT:
    #          x        y
    # x  1.00000  0.92421
    # y  0.92421  1.00000

    # Test for statistical significance
    from scipy.stats import t
    from math import sqrt

    # sample size
    n = df.shape[0]
    print(n)
    lower_cv = t(n - 1).ppf(.025)
    upper_cv = t(n - 1).ppf(.975)

    # retrieve correlation coefficient
    r = correlations["y"]["x"]

    # Perform the test
    test_value = r / sqrt((1 - r ** 2) / (n - 2))

    print("TEST VALUE: {}".format(test_value))
    print("CRITICAL RANGE: {}, {}".format(lower_cv, upper_cv))

    if test_value < lower_cv or test_value > upper_cv:
        print("CORRELATION PROVEN, REJECT H0")
    else:
        print("CORRELATION NOT PROVEN, FAILED TO REJECT H0 ")

    # Calculate p-value
    if test_value > 0:
        p_value = 1.0 - t(n - 1).cdf(test_value)
    else:
        p_value = t(n - 1).cdf(test_value)

    # Two-tailed, so multiply by 2
    p_value = p_value * 2
    print("P-VALUE: {}".format(p_value))

    """
    TEST VALUE: 23.835515323677328
    CRITICAL RANGE: -1.9844674544266925, 1.984467454426692
    CORRELATION PROVEN, REJECT H0
    P-VALUE: 0.0 (extremely small)
    """
    ```

1.  在<math alttext="x equals 50"><mrow><mi>x</mi> <mo>=</mo> <mn>50</mn></mrow></math>时，预测区间在50.79到134.51之间。代码如下：

    ```py
    import pandas as pd
    from scipy.stats import t
    from math import sqrt

    # Load the data
    points = list(pd.read_csv('https://bit.ly/3C8JzrM', delimiter=",") \
        .itertuples())

    n = len(points)

    # Linear Regression Line
    m = 1.75919315
    b = 4.69359655

    # Calculate Prediction Interval for x = 50
    x_0 = 50
    x_mean = sum(p.x for p in points) / len(points)

    t_value = t(n - 2).ppf(.975)

    standard_error = sqrt(sum((p.y - (m * p.x + b)) ** 2 for p in points) / \
        (n - 2))

    margin_of_error = t_value * standard_error * \
                      sqrt(1 + (1 / n) + (n * (x_0 - x_mean) ** 2) / \
                           (n * sum(p.x ** 2 for p in points) - \
    	sum(p.x for p in points) ** 2))

    predicted_y = m*x_0 + b

    # Calculate prediction interval
    print(predicted_y - margin_of_error, predicted_y + margin_of_error)
    # 50.792086501055955 134.51442159894404
    ```

1.  将测试数据集分成三分，并使用k-fold（其中*k* = 3）评估时表现还不错。在这三个数据集中，均方误差大约为0.83，标准偏差为0.03。

    ```py
    import pandas as pd
    from sklearn.linear_model import LinearRegression
    from sklearn.model_selection import KFold, cross_val_score

    df = pd.read_csv('https://bit.ly/3C8JzrM', delimiter=",")

    # Extract input variables (all rows, all columns but last column)
    X = df.values[:, :-1]

    # Extract output column (all rows, last column)\
    Y = df.values[:, -1]

    # Perform a simple linear regression
    kfold = KFold(n_splits=3, random_state=7, shuffle=True)
    model = LinearRegression()
    results = cross_val_score(model, X, Y, cv=kfold)
    print(results)
    print("MSE: mean=%.3f (stdev-%.3f)" % (results.mean(), results.std()))
    """
    [0.86119665 0.78237719 0.85733887]
    MSE: mean=0.834 (stdev-0.036)
    """
    ```

# 第6章

1.  当你在scikit-learn中运行这个算法时，准确率非常高。我运行时，平均在测试折叠上至少获得99.9%的准确率。

    ```py
    import pandas as pd
    from sklearn.linear_model import LogisticRegression
    from sklearn.metrics import confusion_matrix
    from sklearn.model_selection import KFold, cross_val_score

    # Load the data
    df = pd.read_csv("https://bit.ly/3imidqa", delimiter=",")

    X = df.values[:, :-1]
    Y = df.values[:, -1]

    kfold = KFold(n_splits=3, shuffle=True)
    model = LogisticRegression(penalty='none')
    results = cross_val_score(model, X, Y, cv=kfold)

    print("Accuracy Mean: %.3f (stdev=%.3f)" % (results.mean(),
    results.std()))
    ```

1.  混淆矩阵将产生大量的真阳性和真阴性，以及很少的假阳性和假阴性。运行这段代码，你会看到：

    ```py
    import pandas as pd
    from sklearn.linear_model import LogisticRegression
    from sklearn.metrics import confusion_matrix
    from sklearn.model_selection import train_test_split

    # Load the data
    df = pd.read_csv("https://bit.ly/3imidqa", delimiter=",")

    # Extract input variables (all rows, all columns but last column)
    X = df.values[:, :-1]

    # Extract output column (all rows, last column)\
    Y = df.values[:, -1]

    model = LogisticRegression(solver='liblinear')

    X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=.33)
    model.fit(X_train, Y_train)
    prediction = model.predict(X_test)

    """
    The confusion matrix evaluates accuracy within each category.
    [[truepositives falsenegatives]
     [falsepositives truenegatives]]

    The diagonal represents correct predictions,
    so we want those to be higher
    """
    matrix = confusion_matrix(y_true=Y_test, y_pred=prediction)
    print(matrix)
    ```

1.  下面展示了一个用于测试用户输入颜色的交互式shell。考虑测试黑色（0,0,0）和白色（255,255,255），看看是否能正确预测暗色和浅色字体。

    ```py
    import pandas as pd
    from sklearn.linear_model import LogisticRegression
    import numpy as np
    from sklearn.model_selection import train_test_split

    # Load the data
    df = pd.read_csv("https://bit.ly/3imidqa", delimiter=",")

    # Extract input variables (all rows, all columns but last column)
    X = df.values[:, :-1]

    # Extract output column (all rows, last column)
    Y = df.values[:, -1]

    model = LogisticRegression(solver='liblinear')

    X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=.33)
    model.fit(X_train, Y_train)
    prediction = model.predict(X_test)

    # Test a prediction
    while True:
        n = input("Input a color {red},{green},{blue}: ")
        (r, g, b) = n.split(",")
        x = model.predict(np.array([[int(r), int(g), int(b)]]))
        if model.predict(np.array([[int(r), int(g), int(b)]]))[0] == 0.0:
            print("LIGHT")
        else:
            print("DARK")
    ```

1.  是的，逻辑回归在预测给定背景颜色的浅色或暗色字体方面非常有效。准确率不仅极高，而且混淆矩阵在主对角线的右上到左下有很高的数字，其他单元格的数字则较低。

# 第7章

显然，你可以尝试不同的隐藏层、激活函数、不同大小的测试数据集等进行大量的实验和尝试。我尝试使用一个有三个节点的隐藏层，使用ReLU激活函数，但在我的测试数据集上难以得到良好的预测。混淆矩阵和准确率一直很差，我运行的任何配置更是表现不佳。

神经网络可能失败的原因有两个：1）测试数据集对于神经网络来说太小（神经网络对数据需求极大），2）对于这类问题，像逻辑回归这样更简单和更有效的模型存在。这并不是说你不能找到一个适用的配置，但你必须小心，避免通过p-hack的方式使得结果过拟合到你所拥有的少量训练和测试数据。

这是我使用的scikit-learn代码：

```py
import pandas as pd
# load data
from sklearn.metrics import confusion_matrix
from sklearn.model_selection import train_test_split
from sklearn.neural_network import MLPClassifier

df = pd.read_csv('https://tinyurl.com/y6r7qjrp', delimiter=",")

# Extract input variables (all rows, all columns but last column)
X = df.values[:, :-1]

# Extract output column (all rows, last column)
Y = df.values[:, -1]

# Separate training and testing data
X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=1/3)

nn = MLPClassifier(solver='sgd',
                   hidden_layer_sizes=(3, ),
                   activation='relu',
                   max_iter=100_000,
                   learning_rate_init=.05)

nn.fit(X_train, Y_train)

print("Training set score: %f" % nn.score(X_train, Y_train))
print("Test set score: %f" % nn.score(X_test, Y_test))

print("Confusion matrix:")
matrix = confusion_matrix(y_true=Y_test, y_pred=nn.predict(X_test))
print(matrix)
```
