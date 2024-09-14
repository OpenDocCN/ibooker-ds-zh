# 第三十二章：文本和注释

创建良好的可视化图表涉及引导读者，使图表讲述一个故事。在某些情况下，可以完全通过视觉方式讲述这个故事，无需添加文本，但在其他情况下，小的文本提示和标签是必要的。也许你会使用的最基本的注释类型是坐标轴标签和标题，但选项远不止于此。让我们看看一些数据及其如何可视化和注释，以传达有趣的信息。我们将开始设置绘图笔记本并导入将要使用的函数：

```py
In [1]: %matplotlib inline
        import matplotlib.pyplot as plt
        import matplotlib as mpl
        plt.style.use('seaborn-whitegrid')
        import numpy as np
        import pandas as pd
```

# 示例：节假日对美国出生的影响

让我们回到之前处理的一些数据，在“例子：出生率数据”中，我们生成了一个绘制整个日历年平均出生的图表。我们将从那里使用相同的清理过程开始，并绘制结果（参见图 32-1）。

```py
In [2]: # shell command to download the data:
        # !cd data && curl -O \
        #   https://raw.githubusercontent.com/jakevdp/data-CDCbirths/master/
        #   births.csv
```

```py
In [3]: from datetime import datetime

        births = pd.read_csv('data/births.csv')

        quartiles = np.percentile(births['births'], [25, 50, 75])
        mu, sig = quartiles[1], 0.74 * (quartiles[2] - quartiles[0])
        births = births.query('(births > @mu - 5 * @sig) &
                               (births < @mu + 5 * @sig)')

        births['day'] = births['day'].astype(int)

        births.index = pd.to_datetime(10000 * births.year +
                                      100 * births.month +
                                      births.day, format='%Y%m%d')
        births_by_date = births.pivot_table('births',
                                            [births.index.month, births.index.day])
        births_by_date.index = [datetime(2012, month, day)
                                for (month, day) in births_by_date.index]
```

```py
In [4]: fig, ax = plt.subplots(figsize=(12, 4))
        births_by_date.plot(ax=ax);
```

![output 6 0](img/output_6_0.png)

###### 图 32-1\. 每日平均出生数按日期统计¹

当我们可视化这样的数据时，注释图表的特定特征通常很有用，以吸引读者的注意。可以使用 `plt.text`/`ax.text` 函数手动完成此操作，该函数将文本放置在特定的 *x*/*y* 值处（参见图 32-2）。

```py
In [5]: fig, ax = plt.subplots(figsize=(12, 4))
        births_by_date.plot(ax=ax)

        # Add labels to the plot
        style = dict(size=10, color='gray')

        ax.text('2012-1-1', 3950, "New Year's Day", **style)
        ax.text('2012-7-4', 4250, "Independence Day", ha='center', **style)
        ax.text('2012-9-4', 4850, "Labor Day", ha='center', **style)
        ax.text('2012-10-31', 4600, "Halloween", ha='right', **style)
        ax.text('2012-11-25', 4450, "Thanksgiving", ha='center', **style)
        ax.text('2012-12-25', 3850, "Christmas ", ha='right', **style)

        # Label the axes
        ax.set(title='USA births by day of year (1969-1988)',
               ylabel='average daily births')

        # Format the x-axis with centered month labels
        ax.xaxis.set_major_locator(mpl.dates.MonthLocator())
        ax.xaxis.set_minor_locator(mpl.dates.MonthLocator(bymonthday=15))
        ax.xaxis.set_major_formatter(plt.NullFormatter())
        ax.xaxis.set_minor_formatter(mpl.dates.DateFormatter('%h'));
```

![output 8 0](img/output_8_0.png)

###### 图 32-2\. 按日期注释的每日平均出生数²

`ax.text` 方法需要一个 *x* 位置、一个 *y* 位置和一个字符串，然后是可选的关键字，指定文本的颜色、大小、样式、对齐方式和其他属性。这里我们使用了 `ha='right'` 和 `ha='center'`，其中 `ha` 是 *水平对齐* 的缩写。有关可用选项的更多信息，请参阅 `plt.text` 和 `mpl.text.Text` 的文档字符串。

# 转换和文本位置

在前面的示例中，我们将文本注释锚定在数据位置上。有时候，将文本锚定在轴或图的固定位置上，而与数据无关，更为可取。在 Matplotlib 中，通过修改 *transform* 来实现这一点。

Matplotlib 使用几种不同的坐标系统：数学上，位于 <math alttext="left-parenthesis x comma y right-parenthesis equals left-parenthesis 1 comma 1 right-parenthesis"><mrow><mo>(</mo> <mi>x</mi> <mo>,</mo> <mi>y</mi> <mo>)</mo> <mo>=</mo> <mo>(</mo> <mn>1</mn> <mo>,</mo> <mn>1</mn> <mo>)</mo></mrow></math> 处的数据点对应于轴或图的特定位置，进而对应于屏幕上的特定像素。在数学上，这些坐标系统之间的转换相对简单，Matplotlib 在内部使用一组良好开发的工具来执行这些转换（这些工具可以在 `matplotlib.transforms` 子模块中探索）。

典型用户很少需要担心变换的细节，但在考虑在图上放置文本时，这些知识是有帮助的。在这种情况下，有三种预定义的变换可能会有所帮助：

`ax.transData`

与数据坐标相关联的变换

`ax.transAxes`

与轴相关联的变换（以轴尺寸为单位）

`fig.transFigure`

与图形相关联的变换（以图形尺寸为单位）

让我们看一个示例，使用这些变换在不同位置绘制文本（参见 图 32-3）。

```py
In [6]: fig, ax = plt.subplots(facecolor='lightgray')
        ax.axis([0, 10, 0, 10])

        # transform=ax.transData is the default, but we'll specify it anyway
        ax.text(1, 5, ". Data: (1, 5)", transform=ax.transData)
        ax.text(0.5, 0.1, ". Axes: (0.5, 0.1)", transform=ax.transAxes)
        ax.text(0.2, 0.2, ". Figure: (0.2, 0.2)", transform=fig.transFigure);
```

![output 11 0](img/output_11_0.png)

###### 图 32-3\. 比较 Matplotlib 的坐标系

Matplotlib 的默认文本对齐方式是使每个字符串开头的“.”大致标记指定的坐标位置。

`transData` 坐标提供与 x 和 y 轴标签关联的通常数据坐标。`transAxes` 坐标给出了从轴的左下角（白色框）开始的位置，作为总轴尺寸的一部分的分数。`transFigure` 坐标类似，但指定了从图的左下角（灰色框）开始的位置，作为总图尺寸的一部分的分数。

注意，现在如果我们更改坐标轴限制，只有 `transData` 坐标会受到影响，而其他坐标保持不变（参见 图 32-4）。

```py
In [7]: ax.set_xlim(0, 2)
        ax.set_ylim(-6, 6)
        fig
```

![output 13 0](img/output_13_0.png)

###### 图 32-4\. 比较 Matplotlib 的坐标系

通过交互式更改坐标轴限制，可以更清楚地看到这种行为：如果您在笔记本中执行此代码，可以通过将 `%matplotlib inline` 更改为 `%matplotlib notebook` 并使用每个图的菜单与图进行交互来实现这一点。

# 箭头和注释

除了刻度和文本，另一个有用的注释标记是简单的箭头。

虽然有 `plt.arrow` 函数可用，但我不建议使用它：它创建的箭头是 SVG 对象，会受到绘图的不同纵横比的影响，使得难以正确使用。相反，我建议使用 `plt.annotate` 函数，它创建一些文本和箭头，并允许非常灵活地指定箭头。

这里演示了使用几种选项的 `annotate` 的示例（参见 图 32-5）。

```py
In [8]: fig, ax = plt.subplots()

        x = np.linspace(0, 20, 1000)
        ax.plot(x, np.cos(x))
        ax.axis('equal')

        ax.annotate('local maximum', xy=(6.28, 1), xytext=(10, 4),
                    arrowprops=dict(facecolor='black', shrink=0.05))

        ax.annotate('local minimum', xy=(5 * np.pi, -1), xytext=(2, -6),
                    arrowprops=dict(arrowstyle="->",
                                    connectionstyle="angle3,angleA=0,angleB=-90"));
```

![output 16 0](img/output_16_0.png)

###### 图 32-5\. 注释示例

箭头样式由 `arrowprops` 字典控制，其中有许多可用选项。这些选项在 Matplotlib 的在线文档中有很好的记录，因此不重复在此介绍，更有用的是展示一些示例。让我们使用之前的出生率图来演示几种可能的选项（参见 图 32-6）。

```py
In [9]: fig, ax = plt.subplots(figsize=(12, 4))
        births_by_date.plot(ax=ax)

        # Add labels to the plot
        ax.annotate("New Year's Day", xy=('2012-1-1', 4100),  xycoords='data',
                    xytext=(50, -30), textcoords='offset points',
                    arrowprops=dict(arrowstyle="->",
                                    connectionstyle="arc3,rad=-0.2"))

        ax.annotate("Independence Day", xy=('2012-7-4', 4250),  xycoords='data',
                    bbox=dict(boxstyle="round", fc="none", ec="gray"),
                    xytext=(10, -40), textcoords='offset points', ha='center',
                    arrowprops=dict(arrowstyle="->"))

        ax.annotate('Labor Day Weekend', xy=('2012-9-4', 4850), xycoords='data',
                    ha='center', xytext=(0, -20), textcoords='offset points')
        ax.annotate('', xy=('2012-9-1', 4850), xytext=('2012-9-7', 4850),
                    xycoords='data', textcoords='data',
                    arrowprops={'arrowstyle': '|-|,widthA=0.2,widthB=0.2', })

        ax.annotate('Halloween', xy=('2012-10-31', 4600),  xycoords='data',
                    xytext=(-80, -40), textcoords='offset points',
                    arrowprops=dict(arrowstyle="fancy",
                                    fc="0.6", ec="none",
                                    connectionstyle="angle3,angleA=0,angleB=-90"))

        ax.annotate('Thanksgiving', xy=('2012-11-25', 4500),  xycoords='data',
                    xytext=(-120, -60), textcoords='offset points',
                    bbox=dict(boxstyle="round4,pad=.5", fc="0.9"),
                    arrowprops=dict(
                        arrowstyle="->",
                        connectionstyle="angle,angleA=0,angleB=80,rad=20"))

        ax.annotate('Christmas', xy=('2012-12-25', 3850),  xycoords='data',
                     xytext=(-30, 0), textcoords='offset points',
                     size=13, ha='right', va="center",
                     bbox=dict(boxstyle="round", alpha=0.1),
                     arrowprops=dict(arrowstyle="wedge,tail_width=0.5", alpha=0.1));

        # Label the axes
        ax.set(title='USA births by day of year (1969-1988)',
               ylabel='average daily births')

        # Format the x-axis with centered month labels
        ax.xaxis.set_major_locator(mpl.dates.MonthLocator())
        ax.xaxis.set_minor_locator(mpl.dates.MonthLocator(bymonthday=15))
        ax.xaxis.set_major_formatter(plt.NullFormatter())
        ax.xaxis.set_minor_formatter(mpl.dates.DateFormatter('%h'));

        ax.set_ylim(3600, 5400);
```

各种选项使`annotate`功能强大且灵活：您可以创建几乎任何箭头样式。不幸的是，这也意味着这些功能通常必须手动调整，这在生成出版质量的图形时可能非常耗时！最后，我要指出，前述样式混合绝不是展示数据的最佳实践，而是作为展示某些可用选项的示例。

更多关于可用箭头和注释样式的讨论和示例可以在 Matplotlib 的[注释教程](https://oreil.ly/abuPw)中找到。

![output 18 0](img/output_18_0.png)

###### 图 32-6\. 按天平均出生率的注释³

¹ 该图的完整版本可以在[GitHub](https://oreil.ly/PDSH_GitHub)找到。

² 该图的完整版本可以在[GitHub](https://oreil.ly/PDSH_GitHub)找到。

³ 该图的完整版本可以在[GitHub](https://oreil.ly/PDSH_GitHub)找到。
