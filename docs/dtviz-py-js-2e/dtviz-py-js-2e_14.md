# 第10章\. 使用Matplotlib可视化数据

作为数据可视化者，熟悉数据的最佳方式之一是通过交互式可视化来理解它，使用已演变的各种图表和绘图来总结和优化数据集。传统上，这个探索阶段的成果被呈现为静态图像，但越来越多地被用来构建更具吸引力的基于Web的交互式图表，比如您可能见过的酷炫的D3可视化之一（我们将在[第V部分](part05.xhtml#part_viz)中构建其中之一）。

Python的Matplotlib及其扩展系列（例如统计焦点的seaborn）形成了一个成熟且非常可自定义的绘图生态系统。Matplotlib绘图可以通过IPython（Qt和笔记本版本）进行交互使用，为您在数据中找到有趣信息提供了非常强大和直观的方式。在本章中，我们将介绍Matplotlib及其伟大的扩展之一，seaborn。

# pyplot和面向对象的Matplotlib

Matplotlib可能会让人感到相当困惑，特别是如果你随机在网上找例子。主要的复杂因素是有两种主要的绘图方式，它们足够相似以至于容易混淆，但又足够不同以至于会导致许多令人沮丧的错误。第一种方式使用全局状态机直接与Matplotlib的`pyplot`模块交互。第二种面向对象的方法使用更熟悉的图和轴类的概念，提供了一种编程的选择。我将在接下来的章节中澄清它们的差异，但是作为一个粗略的经验法则，如果你正在交互式地处理单个绘图，`pyplot`的全局状态是一个方便的快捷方式。对于其他所有场合，使用面向对象的方法显式声明你的图和轴是有意义的。

# 启动交互式会话

我们将使用[Jupyter notebook](https://jupyter.org)进行交互式可视化。使用以下命令启动会话：

```py
$ jupyter notebook
```

您可以在IPython会话中使用[Matplotlib魔术命令](https://oreil.ly/KhWbX)之一来启用交互式Matplotlib。单独使用`%matplotlib`将使用默认的GUI后端创建绘图窗口，但您可以直接指定后端。以下命令应该适用于标准和Qt控制台IPython：^([1](ch10.xhtml#idm45607769764064))

```py
%matplotlib [qt | osx | wx ...]
```

要在笔记本或Qt控制台中获取内联图形，可以使用`inline`指令。请注意，使用内联绘图时，无法在创建后进行修改，不同于独立的Matplotlib窗口：

```py
%matplotlib inline
```

无论您是在交互式环境还是在Python程序中使用Matplotlib，您都会使用类似的导入方式：

```py
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

###### 注意

你将发现许多使用`pylab`的Matplotlib示例。`pylab`是一个便捷模块，它将`matplotlib.pyplot`（用于绘图）和NumPy批量导入到单一命名空间中。现在`pylab`基本上已经过时，但即使它没有过时，我仍然建议避免使用这个命名空间，并显式地合并和导入`pyplot`和`numpy`。

虽然NumPy和pandas不是强制的，但Matplotlib设计时考虑了它们，能处理NumPy数组，通过关联处理pandas Series。

在IPython中创建内联图的能力对于与Matplotlib的愉快交互至关重要，我们使用以下“魔术”^([2](ch10.xhtml#idm45607769728560)) 指令来实现这一点：

```py
In [0]: %matplotlib inline
```

现在你的Matplotlib图表将插入到你的IPython工作流中。这适用于Qt和notebook版本。在notebooks中，图表将被合并到活动单元格中。

# 修改绘图

在内联模式下，当Jupyter notebook单元格或（多行）输入运行后，绘图上下文被刷新。这意味着你不能使用`gcf`（获取当前图形）方法从先前单元格或输入更改图表，而必须在新的输入/单元格中重复所有绘图命令并进行任何添加或修改。

# 使用pyplot的全局状态进行交互绘图

`pyplot`模块提供了一个全局状态，你可以进行交互式操作。^([3](ch10.xhtml#idm45607769721600)) 这是用于交互式数据探索的，当你创建简单图表时非常方便。你会看到很多示例使用`pyplot`，但对于更复杂的绘图，Matplotlib的面向对象API（我们马上会看到）更适合。在演示全局绘图使用之前，让我们创建一些随机数据来显示，借助pandas有用的`period_range`方法：

```py
from datetime import datetime

x = pd.period_range(datetime.now(), periods=200, freq='d')![1](assets/1.png)
x = x.to_timestamp().to_pydatetime() ![2](assets/2.png)
y = np.random.randn(200, 3).cumsum(0) ![3](assets/3.png)
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO1-1)

使用当前时间（`datetime.now()`）从现在开始创建包含200天（`d`）元素的pandas `datetime`索引。

[![2](assets/2.png)](#co_visualizing_data_with_matplotlib_CO1-2)

将`datetime`索引转换为Python的`datetimes`。

[![3](assets/3.png)](#co_visualizing_data_with_matplotlib_CO1-3)

创建三个包含200个元素的随机数组，沿着0轴求和。

现在我们有一个包含200个时间槽的y轴和三个随机数组作为补充的x值。这些作为独立的参数提供给`(line)plot`方法：

```py
plt.plot(x, y)
```

这给我们展示了不是特别激动人心的图表，如[图 10-1](#mpl_3lines_default)所示。注意Matplotlib如何自然地处理多维NumPy线性数组。

![dpj2 1001](assets/dpj2_1001.png)

###### 图10-1\. 默认线性图

尽管 Matplotlib 的默认设置普遍被认为不够理想，但它的一个优点是你可以进行大量的自定义。这也是为什么有一个丰富的图表库生态系统，用更好的默认设置、更吸引人的配色方案等封装了 Matplotlib。让我们通过使用原始 Matplotlib 来定制我们的默认图表，看看这种自定义的效果。

## 配置 Matplotlib

Matplotlib 提供了广泛的[配置选项](https://oreil.ly/IbgVA)，可以在[`matplotlibrc` 文件](https://oreil.ly/knyiZ)中指定，也可以通过类似字典的 `rcParams` 变量进行动态配置。这里我们改变了图表线条的宽度和默认颜色：

```py
import matplotlib as mpl
mpl.rcParams['lines.linewidth'] = 2
mpl.rcParams['lines.color'] = 'r' # red
```

你可以在[主站点](https://oreil.ly/LBqxb)找到一个样例 `matplotlibrc` 文件。

除了使用 `rcParams` 变量外，你还可以使用 `gcf` （获取当前图形）方法直接获取当前活动的图形并对其进行操作。

让我们看一个小例子，配置当前图形的大小。

## 设置图形大小

如果你的图表默认的可读性较差，或者宽高比不理想，你可能需要改变它的大小。默认情况下，Matplotlib 使用英寸作为其绘图大小的单位。考虑到 Matplotlib 可以保存到许多后端（通常是基于矢量图形的），这是合理的。这里我们展示了两种使用 `pyplot` 设置图形大小为八乘四英寸的方法，分别是使用 `rcParams` 和 `gcf`：

```py
# Two ways to set the figure size to 8 by 4 inches
plt.rcParams['figure.figsize'] = (8,4)
plt.gcf().set_size_inches(8, 4)
```

## 点，而不是像素

Matplotlib 使用点而不是像素来测量其图形的大小。这是用于印刷质量出版物的标准度量单位，而 Matplotlib 则用于生成出版质量的图像。

默认情况下，一个点大约是 1/72 英寸宽，但是 Matplotlib 允许你通过更改生成的任何图形的每英寸点数（dpi）来调整此值。数值越高，图像质量越好。为了在 IPython 会话期间交互地显示内联图形，分辨率通常是由用于生成图表的后端引擎（如 Qt、WxAgg、tkinter 等）决定的。查看[Matplotlib 文档](https://oreil.ly/4ENnG)了解有关后端的解释。

## 标签和图例

[图 10-1](#mpl_3lines_default) 需要告诉我们线的含义，此外还有其他内容。Matplotlib 提供了一个方便的图例框用于标记线条，像大多数 Matplotlib 的功能一样，这也是可以进行大量配置的。标记我们的三条线涉及到一点间接性，因为 `plot` 方法只接受一个标签，它会应用到生成的所有线条上。幸运的是，`plot` 命令返回创建的所有 `Line2D` 对象，这些对象可以被 `legend` 方法用于设置单独的标签。

因为这张图将会以黑白形式出现（如果您阅读本书的打印版本），我们需要一种方法来区分线条，而不是使用默认的颜色。在 Matplotlib 中实现这一点的最简单方法是顺序创建线条，指定 *x* 和 *y* 值以及线型。我们将使用实线 (-)、虚线 (--) 和点划线 (-.) 来创建线条。请注意 NumPy 的列索引用法（见 [Figure 7-1](ch07.xhtml#numpy_indexing)）：

```py
#plots = plt.plot(x,y)
plots = plt.plot(x, y[:,0], '-', x, y[:,1], '--', x, y[:,2], '-.')
plots
Out:
[<matplotlib.lines.Line2D at 0x9b31a90>,
 <matplotlib.lines.Line2D at 0x9b4da90>,
 <matplotlib.lines.Line2D at 0x9b4dcd0>]
```

[`legend`方法](https://oreil.ly/2hEMc) 可以设置标签，建议图例框的位置，并配置许多其他内容：

```py
plt.legend(plots, ('foo', 'bar', 'baz'), ![1](assets/1.png)
           loc='best', ![2](assets/2.png)
           framealpha=0.5, ![3](assets/3.png)
           prop={'size':'small', 'family':'monospace'}) ![4](assets/4.png)
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO2-1)

设置我们三个图的标签。

[![2](assets/2.png)](#co_visualizing_data_with_matplotlib_CO2-2)

使用`best`位置应避免遮挡线条。

[![3](assets/3.png)](#co_visualizing_data_with_matplotlib_CO2-3)

设置图例的透明度。

[![4](assets/4.png)](#co_visualizing_data_with_matplotlib_CO2-4)

这里我们调整图例的字体属性：^([5](ch10.xhtml#idm45607769227712))

## 标题和轴标签

添加标题和轴标签非常简单：

```py
plt.title('Random trends')
plt.xlabel('Date')
plt.ylabel('Cum. sum')
```

使用`figtext`方法可以添加一些文本：^([6](ch10.xhtml#idm45607769200272))

```py
plt.figtext(0.995, 0.01, ![1](assets/1.png)
            '© Acme designs 2022',
            ha='right', va='bottom') ![2](assets/2.png)
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO3-1)

文本相对于图大小的位置。

[![2](assets/2.png)](#co_visualizing_data_with_matplotlib_CO3-2)

水平(`ha`)和垂直(`va`)对齐。

完整的代码显示在 [Example 10-1](#mpl_3lines_custom_code) 中，生成的图表在 [Figure 10-2](#mpl_3lines_custom) 中。

##### Example 10-1\. 自定义折线图

```py
plots = plt.plot(x, y[:,0], '-', x, y[:,1], '--', x, y[:,2], '-.')
plt.legend(plots, ('foo', 'bar', 'baz'), loc='best,
                    framealpha=0.25,
                    prop={'size':'small', 'family':'monospace'})
plt.gcf().set_size_inches(8, 4)
plt.title('Random trends')
plt.xlabel('Date')
plt.ylabel('Cum. sum')
plt.grid(True) ![1](assets/1.png)
plt.figtext(0.995, 0.01, '© Acme Designs 2021',
ha='right', va='bottom')
plt.tight_layout() ![2](assets/2.png)
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO4-1)

这将在图中添加点状网格，标记轴刻度。

[![2](assets/2.png)](#co_visualizing_data_with_matplotlib_CO4-2)

[`tight_layout`方法](https://oreil.ly/roH2Z)应保证所有的绘图元素都在图框内。否则，可能会发现刻度标签或图例被截断。

![dpj2 1002](assets/dpj2_1002.png)

###### 图 10-2\. 自定义折线图

我们在 [Example 10-1](#mpl_3lines_custom_code) 中使用了`tight_layout`方法，以防止绘图元素被遮挡或截断。在某些系统（特别是 macOS）中，`tight_layout`已知可能会引起问题。如果您遇到任何问题，可以参考这个 [问题线程](https://oreil.ly/qGONZ)。

```py
plt.gcf().set_tight_layout(True)
```

## 保存您的图表

Matplotlib 在保存绘图方面表现出色，提供多种输出格式。^([7](ch10.xhtml#idm45607768868400)) 可用的格式取决于可用的后端，但通常支持 PNG、PDF、PS、EPS 和 SVG。PNG 代表便携式网络图形，是分发网络图像的最流行格式。其他格式都是基于矢量的，可以在不产生像素化伪影的情况下平滑缩放。对于高质量的印刷工作，这可能是您想要的格式。

保存操作就像这样简单：

```py
plt.tight_layout() # force plot into figure dimensions
plt.savefig('mpl_3lines_custom.svg')
```

您可以使用 `format="svg"` 明确设置格式，但 Matplotlib 也能理解 *.svg* 后缀。为避免标签截断，使用 `tight_layout` 方法。^([8](ch10.xhtml#idm45607768834368))

# 图形和面向对象的 Matplotlib

正如刚刚展示的，交互式地操作 `pyplot` 的全局状态对于快速数据草图和单一绘图工作效果良好。然而，如果您希望更多地控制图表，Matplotlib 的图形和坐标轴面向对象（OO）方法是更好的选择。您看到的大多数高级绘图演示都是用这种方式完成的。

从本质上讲，使用面向对象的 Matplotlib，我们处理的是一个图形（figure），可以将其视为一个带有一个或多个坐标轴（或绘图）的绘图画布。图形（figure）和坐标轴（axes）都有可以独立指定的属性。在这个意义上，之前讨论的交互式 `pyplot` 路线是将绘图绘制到全局图形的单个坐标轴上。

我们可以使用 `pyplot` 的 `figure` 方法创建一个图形：

```py
fig = plt.figure(
          figsize=(8, 4), # figure size in inches
          dpi=200, # dots per inch
          tight_layout=True, # fit axes, labels, etc. to canvas
          linewidth=1, edgecolor='r' # 1 pixel wide, red border
          )
```

正如您所见，图形与全局 `pyplot` 模块共享一部分属性。这些属性可以在创建图形时设置，也可以通过类似的方法设置（例如 `fig.text()` 而不是 `plt.fig_text()`）。每个图形可以有多个轴，每个轴类似于单个全局绘图状态，但具有多个独立属性的优势。

## 坐标轴和子图

`figure.add_axes` 方法允许精确控制图形中轴的位置（例如，使您能够在主轴中嵌入一个较小的绘图）。绘图元素的定位使用的是 0 → 1 的坐标系统，其中 1 是图形的宽度或高度。您可以使用四元素列表或元组指定位置，以设置底部左侧和顶部右侧边界：

```py
# h = height, w = width
fig.add_axes([0.2, 0.2, #[bottom(h*0.2), left(w*0.2),
              0.8, 0.8])# top(h*0.8), right(w*0.8)]
```

[示例 10-2](#mpl_add_axes_code) 展示了插入较大坐标轴中较小坐标轴所需的代码，使用我们的随机测试数据。结果显示在 [图 10-3](#mpl_add_axes) 中。

##### 示例 10-2\. 使用 `figure.add_axes` 插入图表

```py
fig = plt.figure(figsize=(8,4))
# --- Main Axes
ax = fig.add_axes([0.1, 0.1, 0.8, 0.8])
ax.set_title('Main Axes with Insert Child Axes')
ax.plot(x, y[:,0]) ![1](assets/1.png)
ax.set_xlabel('Date')
ax.set_ylabel('Cum. sum')
# --- Inserted Axes
ax = fig.add_axes([0.15, 0.15, 0.3, 0.3])
ax.plot(x, y[:,1], color='g') # 'g' for green
ax.set_xticks([]); ![2](assets/2.png)
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO5-1)

这选择了我们随机生成的 NumPy y 数据的第一列。

[![2](assets/2.png)](#co_visualizing_data_with_matplotlib_CO5-2)

从我们嵌入的绘图中删除 x 轴刻度和标签。

虽然`add_axes`给了我们很多调整图表外观的空间，但大多数情况下，Matplotlib内置的网格布局系统使生活变得更加轻松。^([9](ch10.xhtml#idm45607768572320)) 最简单的选项是使用`figure.subplots`，它允许您指定等大小的行列布局的网格。如果您想要具有不同大小的网格，`gridspec`模块是您的首选。

![dpj2 1003](assets/dpj2_1003.png)

###### 图例 10-3\. 使用`figure.add_axes`插入绘图

调用不带参数的`subplots`返回一个带有单个轴的图。这在使用`pyplot`全局状态机时最接近，如[“使用pyplot的交互绘图”](#sect_pyplot_global)中所示。[示例 10-3](#mpl_3lines_custom_axes_code)展示了等效于[示例 10-1](#mpl_3lines_custom_code)中`pyplot`演示的图和轴，生成了[图例 10-2](#mpl_3lines_custom)中的图表。请注意使用“setter”方法设置图和轴。

##### 示例 10-3\. 使用单个图和轴绘图

```py
figure, ax = plt.subplots()
plots = ax.plot(x, y, label='')
figure.set_size_inches(8, 4)
ax.legend(plots, ('foo', 'bar', 'baz'), loc='best', framealpha=0.25,
          prop={'size':'small', 'family':'monospace'})
ax.set_title('Random trends')
ax.set_xlabel('Date')
ax.set_ylabel('Cum. sum')
ax.grid(True)
figure.text(0.995, 0.01, '©  Acme Designs 2022',
            ha='right', va='bottom')
figure.tight_layout()
```

调用带有行数（`nrows`）和列数（`ncols`）参数的`subplots`（如[示例 10-4](#mpl_subplots_code)所示）允许将多个绘图放置在网格布局上（见[图例 10-4](#mpl_subplots)的结果）。`subplots`的调用返回图和轴的数组，按行列顺序排列。在本例中，我们指定了一列，因此`axes`是一个包含三个堆叠轴的单个数组。

##### 示例 10-4\. 使用子图

```py
fig, axes = plt.subplots(
                    nrows=3, ncols=1, ![1](assets/1.png)
                    sharex=True, sharey=True, ![2](assets/2.png)
                    figsize=(8, 8))
labelled_data = zip(y.transpose(), ![3](assets/3.png)
                    ('foo', 'bar', 'baz'), ('b', 'g', 'r'))
fig.suptitle('Three Random Trends', fontsize=16)
for i, ld in enumerate(labelled_data):
    ax = axes[i]
    ax.plot(x, ld[0], label=ld[1], color=ld[2])
    ax.set_ylabel('Cum. sum')
    ax.legend(loc='upper left', framealpha=0.5,
              prop={'size':'small'})
axes[-1].set_xlabel('Date') ![4](assets/4.png)
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO6-1)

指定了一个三行一列的子图网格。

[![2](assets/2.png)](#co_visualizing_data_with_matplotlib_CO6-2)

我们希望共享x和y轴，自动调整限制以便进行简单比较。

[![3](assets/3.png)](#co_visualizing_data_with_matplotlib_CO6-3)

将y切换为行列，并将线数据、标签和线颜色一起压缩。

[![4](assets/4.png)](#co_visualizing_data_with_matplotlib_CO6-4)

给最后一个共享的x轴加上标签。

![dpj2 1004](assets/dpj2_1004.png)

###### 图例 10-4\. 三个子图

我们利用Python便利的`zip`方法生成了包含线数据的三个字典。[`zip`](https://oreil.ly/G8YGh)接受长度为*n*的列表或元组，并返回*n*个列表，按顺序匹配元素：

```py
letters = ['a', 'b']
numbers = [1, 2]
zip(letters, numbers)
Out:
[('a', 1), ('b', 2)]
```

在`for`循环中，我们使用`enumerate`为索引`i`提供了一个轴，使用我们的`labelled_data`来提供绘图属性。

注意在`subplots`调用中指定的共享x和y轴，如[示例 10-4](#mpl_subplots_code)（2）所示。这样可以在现在标准化的y轴上轻松比较三个图表，为了避免冗余的x标签，我们仅在最后一行调用`set_xlabel`，使用Python方便的负索引。

现在我们已经讨论了IPython和Matplotlib交互式使用的两种方式，即使用全局状态（通过`plt`访问）和面向对象的API，让我们看看您将用来探索数据集的几种常见绘图类型。

# 绘图类型

除了刚才演示的折线图外，Matplotlib 还有许多其他类型的图表可用。接下来我将演示几种在探索性数据可视化中常用的类型。

## 条形图

朴素的条形图是许多视觉数据探索中的重要工具。与大多数 Matplotlib 图表一样，可以进行大量自定义。我们将介绍几个变体，帮助您理解其要领。

[示例 10-5](#mpl_barchart_code) 中的代码生成了 [图 10-5](#mpl_barchart) 的条形图。请注意，您需要指定自己的条和标签位置。这种灵活性深受 Matplotlib 爱好者的喜爱，并且相当容易上手。尽管如此，有时这样的工作可能会显得乏味。编写一些辅助方法非常简单，此外还有许多封装了 Matplotlib 的库，使得操作更加用户友好。正如我们将在 [第 11 章](ch11.xhtml#chapter_pandas_exploring) 中看到的那样，基于 pandas 的 Matplotlib 绘图功能要简单得多。

##### 示例 10-5\. 简单的条形图

```py
labels = ["Physics", "Chemistry", "Literature", "Peace"]
foo_data =   [3, 6, 10, 4]

bar_width = 0.5
xlocations = np.array(range(len(foo_data))) + bar_width ![1](assets/1.png)
plt.bar(xlocations, foo_data, width=bar_width)
plt.yticks(range(0, 12)) ![2](assets/2.png)
plt.xticks(xlocations, labels) ![3](assets/3.png)
plt.title("Prizes won by Fooland")
plt.gca().get_xaxis().tick_bottom()
plt.gca().get_yaxis().tick_left()
plt.gcf().set_size_inches((8, 4))
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO7-1)

在这里，我们创建了中间条的位置，两个 `bar_width` 之间相隔。

[![2](assets/2.png)](#co_visualizing_data_with_matplotlib_CO7-2)

为了演示目的，我们在此硬编码了 x 值，通常您会希望动态计算范围。

[![3](assets/3.png)](#co_visualizing_data_with_matplotlib_CO7-3)

这会将刻度标签放置在条的中间。

![dpj2 1005](assets/dpj2_1005.png)

###### 图 10-5\. 简单的条形图

多组条形图尤其有用。在 [示例 10-6](#mpl_barchart_multi_code) 中，我们添加了更多的国家数据（来自一个虚构的 Barland），并使用 `subplots` 方法生成了分组条形图（见 [图 10-6](#mpl_barchart_multi)）。再次手动指定条的位置，使用 `ax.bar` 添加了两组条形图。请注意，我们的轴的 x 范围会以合理的方式自动重新缩放，以增量为 0.5：

```py
ax.get_xlim()
# Out: (-0.5, 3.5)
```

如果自动缩放不能达到预期效果，请使用相应的设置方法（例如此处的 `set_xlim`）。

##### 示例 10-6\. 创建分组条形图

```py
labels = ["Physics", "Chemistry", "Literature", "Peace"]
foo_data = [3, 6, 10, 4]
bar_data = [8, 3, 6, 1]

fig, ax = plt.subplots(figsize=(8, 4))
bar_width = 0.4 ![1](assets/1.png)
xlocs = np.arange(len(foo_data))
ax.bar(xlocs-bar_width, foo_data, bar_width,
       color='#fde0bc', label='Fooland') ![2](assets/2.png)
ax.bar(xlocs, bar_data, bar_width, color='peru', label='Barland')
#--- ticks, labels, grids, and title
ax.set_yticks(range(12))
ax.set_xticks(ticks=range(len(foo_data)))
ax.set_xticklabels(labels)
ax.yaxis.grid(True)
ax.legend(loc='best')
ax.set_ylabel('Number of prizes')
fig.suptitle('Prizes by country')
fig.tight_layout(pad=2) ![3](assets/3.png)
fig.savefig('mpl_barchart_multi.png', dpi=200) ![4](assets/4.png)
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO8-1)

对于我们的双条组，使用 `1` 的宽度，这个条宽度提供了 0.1 的条填充。

[![2](assets/2.png)](#co_visualizing_data_with_matplotlib_CO8-2)

Matplotlib 支持标准的 HTML 颜色，可以使用十六进制值或名称。

[![3](assets/3.png)](#co_visualizing_data_with_matplotlib_CO8-3)

我们使用 `pad` 参数来指定围绕图像的填充，其值为字体大小的一部分。

[![4](assets/4.png)](#co_visualizing_data_with_matplotlib_CO8-4)

这将以每英寸 200 点的高分辨率保存图像。

![dpj2 1006](assets/dpj2_1006.png)

###### 图 10-6\. 分组条形图

如果条形图很多并且使用刻度标签，横向放置它们通常更有用，因为标签可能会相互重叠在同一行。将 [图 10-6](#mpl_barchart_multi) 转为水平方向很容易，只需将 `bar` 方法替换为其水平对应方法 `barh`，并交换轴标签和限制（参见 [示例 10-7](#mpl_barchart_multi_h_code) 和生成的图表 [图 10-7](#mpl_barchart_multi_h)）。

##### 示例 10-7\. 将 [示例 10-6](#mpl_barchart_multi_code) 转换为水平条形图

```py
# ...
ylocs = np.arange(len(foo_data))
ax.barh(ylocs-bar_width, foo_data, bar_width, color='#fde0bc',
        label='Fooland') ![1](assets/1.png)
ax.barh(ylocs, bar_data, bar_width, color='peru', label='Barland')
# --- labels, grids and title, then save
ax.set_xticks(range(12)) ![2](assets/2.png)
ax.set_yticks(ticks=ylocs-bar_width/2)
ax.set_yticklabels(labels)
ax.xaxis.grid(True)
ax.legend(loc='best')
ax.set_xlabel('Number of prizes')
# ...
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO9-1)

要创建水平条形图，我们使用 `barh` 替代 `bar`。

[![2](assets/2.png)](#co_visualizing_data_with_matplotlib_CO9-2)

水平图表需要交换水平和垂直轴。

![dpj2 1007](assets/dpj2_1007.png)

###### 图 10-7\. 将条形图横置

在 Matplotlib 中实现堆叠条形图很容易。（参见 [10](ch10.xhtml#idm45607767371344)）[示例 10-8](#mpl_barchart_multi_stack_code) 将 [图 10-6](#mpl_barchart_multi) 转为堆叠形式；[图 10-8](#mpl_barchart_multi_stack) 展示了结果。使用 `bar` 方法的 bottom 参数将提升条的底部设置为前一组的顶部。

##### 示例 10-8\. 将 [示例 10-6](#mpl_barchart_multi_code) 转换为堆叠条形图

```py
# ...
bar_width = 0.8
xlocs = np.arange(len(foo_data))
ax.bar(xlocs, foo_data, bar_width, color='#fde0bc', ![1](assets/1.png)
       label='Fooland')
ax.bar(xlocs, bar_data, bar_width, color='peru',    ![2](assets/2.png)
       label='Barland', bottom=foo_data)
# --- labels, grids and title, then save
ax.set_yticks(range(18))
ax.set_xticks(ticks=xlocs)
ax.set_xticklabels(labels)
# ...
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO10-1)

`foo_data` 和 `bar_data` 条形图组共享相同的 x 轴位置。

[![2](assets/2.png)](#co_visualizing_data_with_matplotlib_CO10-2)

`bar_data` 组的底部是 `foo_data` 组的顶部，形成了堆叠条形图。

![dpj2 1008](assets/dpj2_1008.png)

###### 图 10-8\. 堆叠条形图

## 散点图

另一个有用的图表是散点图，它接受点大小、颜色等选项的 2D 数组。

[示例 10-9](#mpl_scatter_code) 显示了一个快速散点图的代码，使用 Matplotlib 自动调整 x 和 y 的限制。我们通过添加正态分布的随机数（标准差为 10）创建了一条嘈杂的线。[图 10-9](#mpl_scatterplot) 展示了生成的图表。

##### 示例 10-9\. 简单的散点图

```py
num_points = 100
gradient = 0.5
x = np.array(range(num_points))
y = np.random.randn(num_points) * 10 + x*gradient ![1](assets/1.png)
fig, ax = plt.subplots(figsize=(8, 4))
ax.scatter(x, y) ![2](assets/2.png)

fig.suptitle('A Simple Scatterplot')
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO11-1)

`randn` 函数提供正态分布的随机数，我们将其缩放到 0 到 10 的范围内，并且添加一个依赖 x 轴的值。

[![2](assets/2.png)](#co_visualizing_data_with_matplotlib_CO11-2)

x 和 y 数组的大小相等，提供了点的坐标。

![dpj2 1009](assets/dpj2_1009.png)

###### 图 10-9\. 简单的散点图

通过将标记大小和颜色索引数组传递给当前默认的颜色映射，我们可以调整单个点的大小和颜色。需要注意的一点（可能会令人困惑的一点）是，我们指定的是标记边界框的面积，而不是圆的直径。这意味着，如果我们希望点的直径是圆的两倍，我们必须将大小增加四倍。^([11](ch10.xhtml#idm45607767082480)) 在 [示例 10-10](#mpl_scatter_sc_code) 中，我们向简单的散点图添加了大小和颜色信息，生成了 [图 10-10](#mpl_scatter_sc)。

##### 示例 10-10\. 调整点的大小和颜色

```py
num_points = 100
gradient = 0.5
x = np.array(range(num_points))
y = np.random.randn(num_points) * 10 + x*gradient
fig, ax = plt.subplots(figsize=(8, 4))
colors = np.random.rand(num_points) ![1](assets/1.png)
size = np.pi * (2 + np.random.rand(num_points) * 8) ** 2 ![2](assets/2.png)
ax.scatter(x, y, s=size, c=colors, alpha=0.5) ![3](assets/3.png)
fig.suptitle('Scatterplot with Color and Size Specified')
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO12-1)

这会生成默认颜色映射的 100 个随机颜色值，取值范围在 0 到 1 之间。

[![2](assets/2.png)](#co_visualizing_data_with_matplotlib_CO12-2)

我们使用幂符号 `**` 对介于 2 到 10 之间的值进行平方，这是我们标记宽度范围的方法。

[![3](assets/3.png)](#co_visualizing_data_with_matplotlib_CO12-3)

我们使用 `alpha` 参数使我们的标记半透明。

![dpj2 1010](assets/dpj2_1010.png)

###### 图 10-10\. 调整点的大小和颜色

# Matplotlib 颜色映射

Matplotlib 提供了大量的颜色映射可供选择，选择适当的颜色映射可以显著提高可视化质量。请参阅 [颜色映射文档](https://oreil.ly/g8Q9b) 了解详情。

### 添加回归线

回归线是两个变量之间相关性的简单预测模型，本例中是散点图的 x 和 y 坐标。该线基本上是通过图的点拟合而成，并且将其添加到散点图中是一种有用的数据可视化技术，也是演示 Matplotlib 和 NumPy 交互的良好方式。

在 [示例 10-11](#mpl_scatter_regression_code) 中，NumPy 的非常有用的 `polyfit` 函数用于生成由 x 和 y 数组定义的点的最佳拟合直线的梯度和常数。然后，我们在相同的坐标轴上绘制这条直线和散点图（参见 [图 10-11](#mpl_scatter_regression)）。

##### 示例 10-11\. 带有回归线的散点图

```py
num_points = 100
gradient = 0.5
x = np.array(range(num_points))
y = np.random.randn(num_points) * 10 + x*gradient
fig, ax = plt.subplots(figsize=(8, 4))
ax.scatter(x, y)
m, c = np.polyfit(x, y ,1) ![1](assets/1.png)
ax.plot(x, m*x + c) ![2](assets/2.png)
fig.suptitle('Scatterplot With Regression-line')
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO13-1)

我们使用 NumPy 的 `polyfit` 在 1D 中获取一条最佳拟合直线通过我们随机点的线性梯度 (`m`) 和常数 (`c`)。

[![2](assets/2.png)](#co_visualizing_data_with_matplotlib_CO13-2)

使用梯度和常数在散点图的坐标轴上绘制一条直线 (`y` = `mx` + `c`)。

![dpj2 1011](assets/dpj2_1011.png)

###### 图 10-11\. 带有回归线的散点图

在进行线性回归时，通常建议绘制置信区间。这可以根据点的数量和分布给出线性拟合的可靠性概念。可以使用Matplotlib和NumPy实现置信区间，但操作起来有点麻烦。幸运的是，有一个基于Matplotlib构建的库，它具有额外的专业函数用于统计分析和数据可视化，并且在许多人看来比Matplotlib的默认视觉效果更好。这个库就是seaborn，现在我们来简单看一下。

# seaborn

有许多库将Matplotlib强大的绘图能力封装成更用户友好的形式^([12](ch10.xhtml#idm45607766687440))，对于我们这些数据可视化者来说，这些库与pandas的兼容性非常好。

[Bokeh](https://bokeh.pydata.org/en/latest)是一个专为网络设计的交互式可视化库，产生浏览器渲染的输出，因此非常适合IPython笔记本。它是一个伟大的成就，设计哲学与D3类似。^([13](ch10.xhtml#idm45607766685872))

但是，要进行交互式、探索性数据可视化，以便对数据有所感觉并建议可视化方法，我推荐使用[seaborn](https://oreil.ly/b2RpH)。seaborn通过一些强大的统计图扩展了Matplotlib，并且与PyData堆栈非常好地集成，与NumPy、pandas以及在SciPy和[statsmodels](https://oreil.ly/peqqT)中找到的统计例程良好地配合。

seaborn的一个好处是不隐藏Matplotlib的API，允许您使用Matplotlib丰富的工具调整图表。从这个意义上说，它并不取代Matplotlib及其相关技能，而是一个非常令人印象深刻的扩展。

要使用seaborn，只需扩展您的标准Matplotlib导入：

```py
import numpy as np
import pandas as pd
import seaborn as sns # relies on matplotlib
import matplotlib as mpl
import matplotlib.pyplot as plt
```

Matplotlib提供了许多绘图样式，可以通过调用`use`方法设置当前样式为seaborn的默认样式，这将为图表提供一个微妙的灰色网格：

```py
matplotlib.style.use('seaborn')
```

你可以在[Matplotlib文档](https://oreil.ly/9RTub)中查看所有可用的样式及其视觉效果。

seaborn的许多函数都设计成接受pandas DataFrame，你可以指定描述二维散点的列值，例如来自[示例 10-9](#mpl_scatter_code)的现有x和y数组。让我们使用它们来生成一些虚拟数据：

```py
data = pd.DataFrame({'dummy x':x, 'dummy y':y})
```

现在我们有一些带有x（'dummy x'）和y（'dummy y'）值的`data`。[示例 10-12](#mpl_scatter_seaborn_code)演示了使用seaborn专用的线性回归绘图`lmplot`，生成了[图 10-12](#mpl_scatter_seaborn)中的图表。请注意，对于某些seaborn绘图，我们可以通过传递以英寸为单位的大小（高度）和宽高比来调整图形大小。还请注意，seaborn共享`pyplot`的全局上下文。

##### 示例 10-12\. 使用seaborn进行线性回归绘图

```py
data = pd.DataFrame({'dummy x':x, 'dummy y':y})
sns.lmplot(data=data, x='dummy x', y='dummy y', ![1](assets/1.png)
           height=4, aspect=2) ![2](assets/2.png)
plt.tight_layout() ![3](assets/3.png)
plt.savefig('mpl_scatter_seaborn.png') ![3](assets/3.png)
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO14-1)

`x` 和 `y` 参数指定了定义图形点坐标的 DataFrame 数据的列名。

[![2](assets/2.png)](#co_visualizing_data_with_matplotlib_CO14-2)

为了设置图形大小，我们提供以英寸为单位的高度和宽高比。在这里，我们将使用 2 的比率以更好地适应本书的页面格式。

[![3](assets/3.png)](#co_visualizing_data_with_matplotlib_CO14-3)

seaborn 共享 `pyplot` 的全局上下文，允许您保存其绘制的图像，就像使用 Matplotlib 一样。

![dpj2 1012](assets/dpj2_1012.png)

###### Figure 10-12\. 使用 seaborn 的线性回归图

正如你所期待的那样，seaborn 这个强调画出吸引人的图形的库，允许大量的视觉定制。让我们对 [Figure 10-12](#mpl_scatter_seaborn) 的外观进行一些改变，并将置信区间调整为 68% 的标准误估计（查看 [Figure 10-13](#mpl_scatter_seaborn_custom) 的结果）：

```py
sns.lmplot(data=data, x='dummy x', y='dummy y', height=4, aspect=2,
           scatter_kws={"color": "slategray"}, ![1](assets/1.png)
           line_kws={"linewidth": 2, "linestyle":'--', ![2](assets/2.png)
                     "color": "seagreen"},
           markers='D', ![3](assets/3.png)
           ci=68) ![4](assets/4.png)
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO15-1)

提供散点图组件的关键字参数，将点的颜色设置为石板灰。

[![2](assets/2.png)](#co_visualizing_data_with_matplotlib_CO15-2)

提供线图组件的关键字参数，设置线宽和样式。

[![3](assets/3.png)](#co_visualizing_data_with_matplotlib_CO15-3)

将 Matplotlib 的标记设置为钻石，使用 Matplotlib 标记代码 *D*。

[![4](assets/4.png)](#co_visualizing_data_with_matplotlib_CO15-4)

我们设置了 68% 的置信区间，即标准误估计。

![dpj2 1013](assets/dpj2_1013.png)

###### Figure 10-13\. 自定义 seaborn 散点图

seaborn 提供了一些比 Matplotlib 基本设置更有用的图形。让我们来看看其中最有趣的之一，使用 seaborn 的 FacetGrid 绘制多维数据的反射。

## FacetGrids

常被称为“格栅”或“镶嵌”绘图，能够在数据集的不同子集上绘制多个相同图形实例的能力，是一种鸟瞰数据的好方法。大量信息可以在一个图中呈现，并且可以快速理解不同维度之间的关系。这种技术与 Edward Tufte 推广的“小多面板图”有关。

FacetGrids 要求数据以 pandas DataFrame 的形式存在（参见 [“DataFrame”](ch08.xhtml#pandas_objects)），并且按照 Hadley Wickham 的说法，应该是“整洁”的形式，意味着 DataFrame 的每一列应该是一个变量，每一行是一个观察结果。

让我们使用Tips，seaborn的一个测试数据集，^([14](ch10.xhtml#idm45607766287520))展示FacetGrid的工作原理。Tips是一个小数据集，显示了小费的分布情况，根据不同维度如周几或顾客是否吸烟。^([15](ch10.xhtml#idm45607766286048))首先，让我们使用`load_dataset`方法将Tips数据加载到pandas DataFrame中：

```py
In [0]: tips = sns.load_dataset('tips')
Out[0]:
     total_bill   tip     sex smoker   day    time  size
0         16.99  1.01  Female     No   Sun  Dinner     2
1         10.34  1.66    Male     No   Sun  Dinner     3
2         21.01  3.50    Male     No   Sun  Dinner     3
3         23.68  3.31    Male     No   Sun  Dinner     2
...
```

要创建一个FacetGrid，我们指定`tips` DataFrame和一个感兴趣的列，如顾客的吸烟状态。该列将用于创建我们的绘图组。吸烟列中有两个类别（'smoker=Yes'和'smoker=No'），这意味着我们的facet-grid中将有两个图表。然后，我们使用grid的`map`方法创建小费与总账单的多个散点图：

```py
g = sns.FacetGrid(tips, col="smoker", height=4, aspect=1)
g.map(plt.scatter, "total_bill", "tip") ![1](assets/1.png)
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO16-1)

`map`接受一个绘图类，本例中为`scatter`，以及此散点图所需的两个（`tips`）维度。

这将产生两个散点图，如[Figure 10-14](#mpl_facetgrid_1)所示，一个用于每种吸烟状态，显示了小费与总账单的相关性。

![dpj2 1014](assets/dpj2_1014.png)

###### Figure 10-14\. 使用散点图的seaborn FacetGrid

我们可以通过指定要在散点图中使用的标记来包含`tips`数据的另一个维度。让我们将其设置为红色菱形表示女性，蓝色方形表示男性：

```py
pal = dict(Female='red', Male='blue')
g = sns.FacetGrid(tips, col="smoker",
                  hue="sex", hue_kws={"marker": ["D", "s"]}, ![1](assets/1.png)
                  palette=pal, height=4, aspect=1, )
g.map(plt.scatter, "total_bill", "tip", alpha=.4)
g.add_legend();
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO17-1)

为`sex`维度添加了标记颜色（`hue`），使用了菱形（`D`）和方形（`s`）形状，并使用我们的调色板（`pal`）使它们呈现红色和蓝色。

您可以在[Figure 10-15](#mpl_facetgrid_2)中看到生成的FacetGrid。

![dpj2 1015](assets/dpj2_1015.png)

###### Figure 10-15\. Scatter plot with diamond and square markers for sex

我们可以使用行和列来创建数据维度的子集。结合一个`regplot`，^([16](ch10.xhtml#idm45607766007680))可以探索五个维度：

```py
pal = dict(Female='red', Male='blue')
g = sns.FacetGrid(tips, col="smoker", row="time", ![1](assets/1.png)
                  hue="sex", hue_kws={"marker": ["D", "s"]},
                  palette=pal, height=4, aspect=1, )
g.map(sns.regplot, "total_bill", "tip", alpha=.4)
g.add_legend();
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO18-1)

添加一个时间行，将小费按午餐和晚餐分开。

[Figure 10-16](#mpl_facetgrid_tips3)展示了四个`regplot`，为女性和男性hue组生成带有置信区间的线性回归模型拟合。图表标题显示正在使用的数据子集，每行具有相同的时间和吸烟者状态。

![dpj2 1016](assets/dpj2_1016.png)

###### Figure 10-16\. Visualizing five dimensions

我们可以通过`lmplot`来实现与我们在[Example 10-12](#mpl_scatter_seaborn_code)中看到的相同效果，它封装了FacetGrid和`regplot`以便于使用。以下代码生成[Figure 10-16](#mpl_facetgrid_tips3)：

```py
pal = dict(Female='red', Male='blue')
sns.lmplot(x="total_bill", y="tip", hue="sex",
           markers=["D", "s"], ![1](assets/1.png)
           col="smoker", row="time", data=tips, palette=pal,
           height=4, aspect=1
           );
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO19-1)

注意使用`markers`关键字，而不是我们在FacetGrid图中使用的`kws_hue`字典。

`lmplot` 提供了一个很好的快捷方式来生成 FacetGrid 的 `regplot`，但是 FacetGrid 的 `map` 允许您使用 seaborn 和 Matplotlib 图表的全套来在维度子集上创建图表。这是一种非常强大的技术，也是深入了解数据的一个很好的方式。

## PairGrid

PairGrid 是另一种相当酷的 seaborn 绘图类型，提供了一种快速评估多维数据的方式。与 FacetGrid 不同，您不会将数据集分成子集，然后按指定的维度进行比较。使用 PairGrid，数据集的各个维度将按成对方式在一个方形网格中进行比较。默认情况下，将比较所有维度，但是您可以通过在声明 PairGrid 时向 `vars` 参数提供列表来指定要绘制的维度。^([17](ch10.xhtml#idm45607765677072))

我们通过使用经典的 Iris 数据集来演示这种成对比较的效用，展示包含三种 Iris 种类成员的一组数据的一些重要统计信息。首先，我们加载示例数据集：

```py
In [0]: iris = sns.load_dataset('iris')
In [1]: iris.head()
Out[1]:
   sepal_length sepal_width  petal_length  petal_width  species
0           5.1         3.5           1.4          0.2   setosa
1           4.9         3.0           1.4          0.2   setosa
2           4.7         3.2           1.3          0.2   setosa
...
```

为了捕捉按物种分组的花瓣和萼片尺寸之间的关系，我们首先创建一个 `PairGrid` 对象，将其色调设置为 `species`，然后使用其映射方法在成对网格的对角线上和非对角线上创建图表，生成图表见 [Figure 10-17](#mpl_pairgrid_iris)：

```py
sns.set_theme(font_scale=1.5) ![1](assets/1.png)
g = sns.PairGrid(iris, hue="species") ![2](assets/2.png)
g.map_diag(plt.hist) ![3](assets/3.png)
g.map_offdiag(plt.scatter) ![4](assets/4.png)
g.add_legend();
```

[![1](assets/1.png)](#co_visualizing_data_with_matplotlib_CO20-1)

使用 seaborn 的 `set_theme` 方法调整字体大小（参见[文档](https://oreil.ly/rSmrH)获取可用调整项的完整列表）。

[![2](assets/2.png)](#co_visualizing_data_with_matplotlib_CO20-2)

将标记和子条设置为按物种着色。

[![3](assets/3.png)](#co_visualizing_data_with_matplotlib_CO20-3)

在网格的对角线上放置物种尺寸的直方图。

[![4](assets/4.png)](#co_visualizing_data_with_matplotlib_CO20-4)

使用标准散点图比较对角线尺寸。

![dpj2 1017](assets/dpj2_1017.png)

###### 图 10-17\. Iris 测量的 PairGrid 汇总

正如您在 [Figure 10-17](#mpl_pairgrid_iris) 中看到的，seaborn 的几行代码就能创建一组丰富信息的图表，相关联不同 Iris 测量指标。这种图表称为 [散点矩阵](https://oreil.ly/UAJ8T)，是在多变量集中查找变量对线性相关性的一个很好的方式。目前，网格中存在冗余：例如，`sepal_width-petal_length` 和 `petal_length-sepal_width` 的图表。`PairGrid` 提供了使用主对角线之上或之下的冗余图表来提供数据的不同反映的机会。查看一些 seaborn 文档中的示例以获取更多信息。^([18](ch10.xhtml#idm45607765484192))

在本节中，我介绍了一些 seaborn 的图表，下一章我们在探索新贵数据集时将看到更多。但 seaborn 还有许多其他非常方便和强大的统计工具。进一步调查时，我建议从 [seaborn 主要文档](https://stanford.io/28L8ezk) 开始。那里有一些很好的示例、完整的 API 文档和一些好的教程，可以补充您在本章学到的内容。

# 总结

本章介绍了 Python 绘图的强大工具 Matplotlib。它是一个成熟的大型库，拥有丰富的文档和活跃的社区。如果您有特定的定制需求，很可能在某处能找到示例。我建议启动一个 [Jupyter 笔记本](https://jupyter.org) 并尝试使用数据集进行操作。

我们看到 seaborn 通过一些有用的统计方法扩展了 Matplotlib，并且许多人认为它具有更优美的美学。它还允许访问 Matplotlib 图形和坐标轴的内部，如果需要的话可以进行完全自定义。

在下一章中，我们将结合 pandas 使用 Matplotlib 探索我们新获取并清理的诺贝尔数据集。我们将使用本章中演示的一些图表类型，看到一些有用的新功能。

^([1](ch10.xhtml#idm45607769764064-marker)) 如果启动 GUI 会话时出现错误，请尝试更改后端设置（例如，如果在 macOS 上使用 `%matplotlib qt` 无效，请尝试 `%matplotlib osx`）。

^([2](ch10.xhtml#idm45607769728560-marker)) IPython 拥有大量这样的函数，可以为普通的 Python 解释器提供一整套有用的额外功能。请查看 [IPython 网站](https://oreil.ly/0gUSc)。

^([3](ch10.xhtml#idm45607769721600-marker)) 这受到 [MATLAB](https://oreil.ly/sw9KZ) 的启发。

^([4](ch10.xhtml#idm45607769467872-marker)) 您可以在 [Matplotlib 的文档](https://oreil.ly/iqlBE) 中找到有关线型的详细信息。

^([5](ch10.xhtml#idm45607769227712-marker)) 更多细节请参阅 [文档](https://oreil.ly/upz5A)。

^([6](ch10.xhtml#idm45607769200272-marker)) 有关详情，请参阅 [Matplotlib 网站](https://oreil.ly/oD0lN)。

^([7](ch10.xhtml#idm45607768868400-marker)) 除了提供多种格式外，它还理解 [LaTeX 数学模式](https://www.latex-project.org)，这是一种语言，允许您在标题、图例等处使用数学符号。这是 Matplotlib 受到学术界喜爱的原因之一，因为它能够生成期刊质量的图像。

^([8](ch10.xhtml#idm45607768834368-marker)) 更多详细信息，请访问 [Matplotlib 网站](https://oreil.ly/GacYP)。

^([9](ch10.xhtml#idm45607768572320-marker)) 便捷的 `tight_layout` 选项假定网格布局子图。

^([10](ch10.xhtml#idm45607767371344-marker)) 叠加条形图是否特别适合理解数据组？请参阅[Solomon Messing的博客](https://oreil.ly/nClO0)，进行精彩的讨论，并提供一个“好”使用的例子。

^([11](ch10.xhtml#idm45607767082480-marker)) 设置标记大小而不是宽度或半径，实际上是一个很好的默认选择，使其与我们试图反映的任何值成比例。

^([12](ch10.xhtml#idm45607766687440-marker)) 普遍认为Matplotlib的默认设置并不那么好，通过改进可以轻松提升任何包的表现。

^([13](ch10.xhtml#idm45607766685872-marker)) D3和Bokeh都向经典的可视化著作《图形语法》（Springer，Leland Wilkinson著）致敬。

^([14](ch10.xhtml#idm45607766287520-marker)) seaborn有一些方便的数据集，你可以在[GitHub](https://oreil.ly/clELR)上找到。

^([15](ch10.xhtml#idm45607766286048-marker)) Tips数据集使用性别作为一个类别，而本书的数据集使用性别。过去这些词汇通常可以互换使用，但现在情况已经不同。请参阅[Yale School of Medicine的文章](https://oreil.ly/P0zWt)以获取解释。

^([16](ch10.xhtml#idm45607766007680-marker)) `regplot`，即回归图，相当于`lmplot`，在[示例 10-12](#mpl_scatter_seaborn_code)中有使用。后者结合了`regplot`和FacetGrid以提供便利。

^([17](ch10.xhtml#idm45607765677072-marker)) 还有`x_vars`和`y_vars`参数，使您能够指定非方形网格。

^([18](ch10.xhtml#idm45607765484192-marker)) 如果你感兴趣，有一个D3的例子在[*bl.ocks.org*站点](https://oreil.ly/ox8VW)上构建了一个散点矩阵。
