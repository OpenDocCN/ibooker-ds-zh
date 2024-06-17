# 第二十七章：简单散点图

另一种常用的图表类型是简单的散点图，它与线图非常相似。点不是通过线段连接，而是分别用点、圆或其他形状表示。我们将从设置绘图笔记本和导入我们将使用的包开始：

```py
In [1]: %matplotlib inline
        import matplotlib.pyplot as plt
        plt.style.use('seaborn-whitegrid')
        import numpy as np
```

# 使用 plt.plot 创建散点图

在前一章中，我们使用 `plt.plot`/`ax.plot` 来生成线图。事实证明，这个函数也可以生成散点图（参见 图 27-1）。

```py
In [2]: x = np.linspace(0, 10, 30)
        y = np.sin(x)

        plt.plot(x, y, 'o', color='black');
```

![output 4 0](img/output_4_0.png)

###### 图 27-1\. 散点图示例

函数调用中的第三个参数是一个字符，代表用于绘图的符号类型。正如你可以指定 `'-'` 或 `'--'` 控制线条样式一样，标记样式也有自己一套简短的字符串代码。可用符号的完整列表可以在 `plt.plot` 的文档中或 Matplotlib 的[在线文档](https://oreil.ly/tmYIL)中找到。大多数可能性都相当直观，并且其中一些更常见的示例在此处演示（参见 图 27-2）。

```py
In [3]: rng = np.random.default_rng(0)
        for marker in ['o', '.', ',', 'x', '+', 'v', '^', '<', '>', 's', 'd']:
            plt.plot(rng.random(2), rng.random(2), marker, color='black',
                     label="marker='{0}'".format(marker))
        plt.legend(numpoints=1, fontsize=13)
        plt.xlim(0, 1.8);
```

![output 6 0](img/output_6_0.png)

###### 图 27-2\. 点数示例

进一步地，这些字符代码可以与线条和颜色代码一起使用，以绘制带有连接线的点（参见 图 27-3）。

```py
In [4]: plt.plot(x, y, '-ok');
```

![output 8 0](img/output_8_0.png)

###### 图 27-3\. 结合线条和点标记的示例

`plt.plot` 的额外关键字参数可以指定线条和标记的多种属性，正如你可以在 图 27-4 中看到的。

```py
In [5]: plt.plot(x, y, '-p', color='gray',
                 markersize=15, linewidth=4,
                 markerfacecolor='white',
                 markeredgecolor='gray',
                 markeredgewidth=2)
        plt.ylim(-1.2, 1.2);
```

![output 10 0](img/output_10_0.png)

###### 图 27-4\. 自定义线条和点标记

这些选项使得 `plt.plot` 成为 Matplotlib 中二维图的主要工具。要了解所有可用选项的详细描述，请参考 [`plt.plot` 文档](https://oreil.ly/ON1xj)。

# 使用 plt.scatter 创建散点图

创建散点图的第二种更强大的方法是 `plt.scatter` 函数，其用法与 `plt.plot` 函数非常相似（参见 图 27-5）。

```py
In [6]: plt.scatter(x, y, marker='o');
```

![output 13 0](img/output_13_0.png)

###### 图 27-5\. 一个简单的散点图

`plt.scatter` 与 `plt.plot` 的主要区别在于，它可以用于创建散点图，其中可以单独控制或映射到数据的每个点的属性（大小、填充颜色、边缘颜色等）。

为了更好地观察重叠的结果，我们创建一个随机散点图，点具有多种颜色和大小。为了调整透明度，我们还会使用 `alpha` 关键字（参见 图 27-6）。

```py
In [7]: rng = np.random.default_rng(0)
        x = rng.normal(size=100)
        y = rng.normal(size=100)
        colors = rng.random(100)
        sizes = 1000 * rng.random(100)

        plt.scatter(x, y, c=colors, s=sizes, alpha=0.3)
        plt.colorbar();  # show color scale
```

![output 15 0](img/output_15_0.png)

###### 图 27-6\. 在散点图中更改点的大小和颜色

注意，颜色参数自动映射到颜色比例（这里通过`colorbar`命令显示），点的大小以像素表示。通过这种方式，可以利用点的颜色和大小来传达可视化信息，以便可视化多维数据。

例如，我们可以使用来自 Scikit-Learn 的鸢尾花数据集，其中每个样本是三种类型的花之一，其花瓣和萼片的大小已经被仔细测量（见图 27-7）。

```py
In [8]: from sklearn.datasets import load_iris
        iris = load_iris()
        features = iris.data.T

        plt.scatter(features[0], features[1], alpha=0.4,
                    s=100*features[3], c=iris.target, cmap='viridis')
        plt.xlabel(iris.feature_names[0])
        plt.ylabel(iris.feature_names[1]);
```

![output 17 0](img/output_17_0.png)

###### 图 27-7\. 使用点属性来编码鸢尾花数据的特征¹

我们可以看到，这个散点图使我们能够同时探索数据的四个不同维度：每个点的(*x*, *y*)位置对应于萼片的长度和宽度，点的大小与花瓣的宽度相关，颜色与特定种类的花相关。像这样的多颜色和多特征散点图既可以用于数据探索，也可以用于数据展示。

# 绘图与散点图：关于效率的一点说明

除了`plt.plot`和`plt.scatter`中提供的不同特性外，为什么你可能选择使用一个而不是另一个？虽然对于少量数据来说这并不重要，但是随着数据集超过几千个点，`plt.plot`比`plt.scatter`效率显著更高。原因在于，`plt.scatter`可以为每个点渲染不同的大小和/或颜色，因此渲染器必须额外工作来构建每个点。而对于`plt.plot`，每个点的标记是相同的，因此确定点的外观的工作仅需一次处理整个数据集。对于大数据集，这种差异可能导致性能大不相同，因此在处理大数据集时，应优先选择`plt.plot`而不是`plt.scatter`。

# 可视化不确定性

对于任何科学测量，准确地考虑不确定性几乎与准确报告数字本身同样重要，甚至更重要。例如，想象我正在使用一些天体物理观测来估计哈勃常数，即宇宙膨胀速率的本地测量。我知道当前文献建议的值约为 70 (km/s)/Mpc，而我的方法测量的值为 74 (km/s)/Mpc。这些值是否一致？基于这些信息，唯一正确的答案是：没有办法知道。

假设我将这些信息与报告的不确定性一起增加：当前文献建议的值为 70 ± 2.5 (km/s)/Mpc，而我的方法测量的值为 74 ± 5 (km/s)/Mpc。现在这些值是否一致？这是一个可以定量回答的问题。

在数据和结果的可视化中，有效地显示这些误差可以使绘图传达更完整的信息。

## 基本误差条

一种标准的可视化不确定性的方法是使用误差条。可以通过单个 Matplotlib 函数调用创建基本的误差条，如图 27-8 所示。

```py
In [1]: %matplotlib inline
        import matplotlib.pyplot as plt
        plt.style.use('seaborn-whitegrid')
        import numpy as np
```

```py
In [2]: x = np.linspace(0, 10, 50)
        dy = 0.8
        y = np.sin(x) + dy * np.random.randn(50)

        plt.errorbar(x, y, yerr=dy, fmt='.k');
```

这里的 `fmt` 是一个控制线条和点的外观的格式代码，其语法与前一章和本章早些时候概述的 `plt.plot` 的简写相同。

![output 4 0](img/output_4_0.png)

###### 图 27-8\. 一个误差条示例

除了这些基本选项外，`errorbar` 函数还有许多选项可以微调输出结果。使用这些附加选项，您可以轻松定制误差条绘图的美学效果。特别是在拥挤的图中，我经常发现将误差条的颜色设为比点本身更浅是有帮助的（见图 27-9）。

```py
In [3]: plt.errorbar(x, y, yerr=dy, fmt='o', color='black',
                     ecolor='lightgray', elinewidth=3, capsize=0);
```

![output 6 0](img/output_6_0.png)

###### 图 27-9\. 自定义误差条

除了这些选项外，您还可以指定水平误差条、单侧误差条和许多其他变体。有关可用选项的更多信息，请参阅 `plt.errorbar` 的文档字符串。

## 连续误差

在某些情况下，希望在连续量上显示误差条。虽然 Matplotlib 没有针对这种类型应用的内置便捷例程，但可以相对轻松地结合 `plt.plot` 和 `plt.fill_between` 这样的基本图形元素来得到有用的结果。

在这里，我们将执行简单的*高斯过程回归*，使用 Scikit-Learn API（详见第三十八章）。这是一种将非常灵活的非参数函数拟合到具有连续不确定度测量的数据的方法。我们目前不会深入讨论高斯过程回归的细节，而是专注于如何可视化这种连续误差测量：

```py
In [4]: from sklearn.gaussian_process import GaussianProcessRegressor

        # define the model and draw some data
        model = lambda x: x * np.sin(x)
        xdata = np.array([1, 3, 5, 6, 8])
        ydata = model(xdata)

        # Compute the Gaussian process fit
        gp = GaussianProcessRegressor()
        gp.fit(xdata[:, np.newaxis], ydata)

        xfit = np.linspace(0, 10, 1000)
        yfit, dyfit = gp.predict(xfit[:, np.newaxis], return_std=True)
```

现在我们有 `xfit`、`yfit` 和 `dyfit`，它们对我们数据的连续拟合进行了采样。我们可以像前面的部分一样将它们传递给 `plt.errorbar` 函数，但我们实际上不想绘制 1,000 个点和 1,000 个误差条。相反，我们可以使用 `plt.fill_between` 函数并使用浅色来可视化这个连续误差（见图 27-10）。

```py
In [5]: # Visualize the result
        plt.plot(xdata, ydata, 'or')
        plt.plot(xfit, yfit, '-', color='gray')
        plt.fill_between(xfit, yfit - dyfit, yfit + dyfit,
                         color='gray', alpha=0.2)
        plt.xlim(0, 10);
```

![output 11 0](img/output_11_0.png)

###### 图 27-10\. 用填充区域表示连续不确定性

查看 `fill_between` 的调用签名：我们传递一个 x 值，然后是下限 *y* 值和上限 *y* 值，结果是这些区域之间的区域被填充。

得到的图形直观地展示了高斯过程回归算法的运行情况：在接近测量数据点的区域，模型受到强约束，这反映在较小的模型不确定性中。在远离测量数据点的区域，模型约束不强，模型不确定性增加。

欲了解更多关于`plt.fill_between`（及其紧密相关的`plt.fill`函数）可用选项的信息，请参阅函数文档字符串或 Matplotlib 文档。

最后，如果这对你来说有点太低级了，请参考第三十六章，在那里我们讨论了 Seaborn 包，它具有更简化的 API 来可视化这种连续误差条类型。

¹ 这幅图的全彩版可在[GitHub](https://oreil.ly/PDSH_GitHub)上找到。
