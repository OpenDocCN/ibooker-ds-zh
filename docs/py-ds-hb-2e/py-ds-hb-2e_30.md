# 第二十六章：简单线图

可能所有绘图中最简单的是单个函数 <math alttext="y equals f left-parenthesis x right-parenthesis"><mrow><mi>y</mi> <mo>=</mo> <mi>f</mi> <mo>(</mo> <mi>x</mi> <mo>)</mo></mrow></math> 的可视化。在这里，我们将首次创建这种类型的简单绘图。如同接下来的所有章节一样，我们将从设置用于绘图的笔记本开始，并导入我们将使用的包：

```py
In [1]: %matplotlib inline
        import matplotlib.pyplot as plt
        plt.style.use('seaborn-whitegrid')
        import numpy as np
```

对于所有的 Matplotlib 图，我们首先创建一个图形和坐标轴。在它们最简单的形式下，可以像下面这样做（见图 26-1）。

```py
In [2]: fig = plt.figure()
        ax = plt.axes()
```

在 Matplotlib 中，*figure*（一个 `plt.Figure` 类的实例）可以被视为一个包含所有代表坐标轴、图形、文本和标签的对象的单个容器。*axes*（一个 `plt.Axes` 类的实例）就是我们看到的上述内容：一个带有刻度、网格和标签的边界框，最终将包含构成我们可视化的绘图元素。在本书的这一部分，我通常使用变量名 `fig` 表示一个图形实例，使用 `ax` 表示一个坐标轴实例或一组坐标轴实例。

![output 4 0](img/output_4_0.png)

###### 图 26-1\. 一个空的网格坐标轴

一旦我们创建了一个坐标轴，就可以使用 `ax.plot` 方法绘制一些数据。让我们从一个简单的正弦波开始，如 图 26-2 所示。

```py
In [3]: fig = plt.figure()
        ax = plt.axes()

        x = np.linspace(0, 10, 1000)
        ax.plot(x, np.sin(x));
```

![output 6 0](img/output_6_0.png)

###### 图 26-2\. 一个简单的正弦波

注意最后一行末尾的分号是有意为之：它抑制了从输出中显示绘图的文本表示。

或者，我们可以使用 PyLab 接口，让图形和坐标轴在后台自动创建（参见 第 IV 部分 讨论这两种接口）；如 图 26-3 所示，结果是相同的。

```py
In [4]: plt.plot(x, np.sin(x));
```

![output 8 0](img/output_8_0.png)

###### 图 26-3\. 通过面向对象接口的简单正弦波

如果我们想要创建一个包含多条线的单个图形（参见 图 26-4），我们可以简单地多次调用 `plot` 函数：

```py
In [5]: plt.plot(x, np.sin(x))
        plt.plot(x, np.cos(x));
```

这就是在 Matplotlib 中绘制简单函数的全部内容！现在我们将深入了解如何控制坐标轴和线条的外观的更多细节。

![output 10 0](img/output_10_0.png)

###### 图 26-4\. 多条线的重叠绘图

# 调整绘图：线条颜色和样式

你可能希望对图表进行的第一个调整是控制线条的颜色和样式。`plt.plot` 函数接受额外的参数来指定这些内容。要调整颜色，可以使用 `color` 关键字，接受一个表示几乎任何想象的颜色的字符串参数。颜色可以以多种方式指定；参见 图 26-5 来查看以下示例的输出：

```py
In [6]: plt.plot(x, np.sin(x - 0), color='blue')        # specify color by name
        plt.plot(x, np.sin(x - 1), color='g')           # short color code (rgbcmyk)
        plt.plot(x, np.sin(x - 2), color='0.75')        # grayscale between 0 and 1
        plt.plot(x, np.sin(x - 3), color='#FFDD44')     # hex code (RRGGBB, 00 to FF)
        plt.plot(x, np.sin(x - 4), color=(1.0,0.2,0.3)) # RGB tuple, values 0 to 1
        plt.plot(x, np.sin(x - 5), color='chartreuse'); # HTML color names supported
```

![output 14 0](img/output_14_0.png)

###### 图 26-5\. 控制绘图元素的颜色

如果未指定颜色，则 Matplotlib 将自动循环使用一组默认颜色来绘制多条线。

同样地，可以使用 `linestyle` 关键字来调整线条样式（参见 图 26-6）。

```py
In [7]: plt.plot(x, x + 0, linestyle='solid')
        plt.plot(x, x + 1, linestyle='dashed')
        plt.plot(x, x + 2, linestyle='dashdot')
        plt.plot(x, x + 3, linestyle='dotted');

        # For short, you can use the following codes:
        plt.plot(x, x + 4, linestyle='-')  # solid
        plt.plot(x, x + 5, linestyle='--') # dashed
        plt.plot(x, x + 6, linestyle='-.') # dashdot
        plt.plot(x, x + 7, linestyle=':'); # dotted
```

![output 16 0](img/output_16_0.png)

###### 图 26-6\. 各种线条样式的示例

虽然对于阅读你的代码的人来说可能不太清晰，但你可以通过将 `linestyle` 和 `color` 代码合并为单个非关键字参数传递给 `plt.plot` 函数来节省一些按键。 图 26-7 显示了结果。

```py
In [8]: plt.plot(x, x + 0, '-g')   # solid green
        plt.plot(x, x + 1, '--c')  # dashed cyan
        plt.plot(x, x + 2, '-.k')  # dashdot black
        plt.plot(x, x + 3, ':r');  # dotted red
```

![output 18 0](img/output_18_0.png)

###### 图 26-7\. 使用简写语法控制颜色和样式

这些单字符颜色代码反映了 RGB（红/绿/蓝）和 CMYK（青/洋红/黄/黑）颜色系统中的标准缩写，通常用于数字彩色图形。

还有许多其他关键字参数可用于微调图表的外观；有关详细信息，请通过 IPython 的帮助工具阅读 `plt.plot` 函数的文档字符串（参见 第 1 章）。

# 调整图表：坐标轴限制

Matplotlib 在为你的图表选择默认的轴限制方面做得相当不错，但有时更精细的控制会更好。调整限制的最基本方法是使用 `plt.xlim` 和 `plt.ylim` 函数（参见 图 26-8）。

```py
In [9]: plt.plot(x, np.sin(x))

        plt.xlim(-1, 11)
        plt.ylim(-1.5, 1.5);
```

![output 21 0](img/output_21_0.png)

###### 图 26-8\. 设置坐标轴限制的示例

如果因某种原因你希望任一轴显示反向，只需反转参数的顺序（参见 图 26-9）。

```py
In [10]: plt.plot(x, np.sin(x))

         plt.xlim(10, 0)
         plt.ylim(1.2, -1.2);
```

![output 23 0](img/output_23_0.png)

###### 图 26-9\. 反转 y 轴的示例

一个有用的相关方法是 `plt.axis`（请注意这里可能会导致 *axes*（带有 *e*）和 *axis*（带有 *i*）之间的潜在混淆），它允许更质量化地指定轴限制。例如，你可以自动收紧当前内容周围的边界，如 图 26-10 所示。

```py
In [11]: plt.plot(x, np.sin(x))
         plt.axis('tight');
```

![output 25 0](img/output_25_0.png)

###### 图 26-10\. “紧凑”布局的示例

或者，您可以指定希望有一个相等的轴比率，这样 `x` 中的一个单位在视觉上等同于 `y` 中的一个单位，如 Figure 26-11 所示。

```py
In [12]: plt.plot(x, np.sin(x))
         plt.axis('equal');
```

![output 27 0](img/output_27_0.png)

###### Figure 26-11\. “equal” 布局示例，单位与输出分辨率匹配

其他轴选项包括 `'on'`、`'off'`、`'square'`、`'image'` 等。有关更多信息，请参阅 `plt.axis` 文档字符串。

# 绘图标签

作为本章的最后一部分，我们将简要讨论绘图的标签：标题、坐标轴标签和简单图例。标题和坐标轴标签是最简单的标签——有方法可以快速设置它们（见 Figure 26-12）。

```py
In [13]: plt.plot(x, np.sin(x))
         plt.title("A Sine Curve")
         plt.xlabel("x")
         plt.ylabel("sin(x)");
```

![output 30 0](img/output_30_0.png)

###### Figure 26-12\. 坐标轴标签和标题示例

可以使用函数的可选参数调整这些标签的位置、大小和样式，这些参数在文档字符串中有描述。

当在单个坐标轴中显示多行时，创建一个标签每种线型的图例是非常有用的。再次强调，Matplotlib 提供了一种内置的快速创建这种图例的方法；通过（你猜对了）`plt.legend` 方法来实现。虽然有几种有效的使用方法，但我发现最简单的方法是使用 `plot` 函数的 `label` 关键字来指定每条线的标签（见 Figure 26-13）。

```py
In [14]: plt.plot(x, np.sin(x), '-g', label='sin(x)')
         plt.plot(x, np.cos(x), ':b', label='cos(x)')
         plt.axis('equal')

         plt.legend();
```

![output 33 0](img/output_33_0.png)

###### Figure 26-13\. 绘图图例示例

如您所见，`plt.legend` 函数跟踪线型和颜色，并将其与正确的标签匹配。有关指定和格式化绘图图例的更多信息，请参阅 `plt.legend` 文档字符串；此外，我们将在 第二十九章 中涵盖一些更高级的图例选项。

# Matplotlib 的一些注意事项

虽然大多数 `plt` 函数可以直接转换为 `ax` 方法（`plt.plot` → `ax.plot`，`plt.legend` → `ax.legend` 等），但并非所有命令都是如此。特别是用于设置限制、标签和标题的功能略有修改。为了在 MATLAB 风格函数和面向对象方法之间进行过渡，请进行以下更改：

+   `plt.xlabel` → `ax.set_xlabel`

+   `plt.ylabel` → `ax.set_ylabel`

+   `plt.xlim` → `ax.set_xlim`

+   `plt.ylim` → `ax.set_ylim`

+   `plt.title` → `ax.set_title`

在面向对象的绘图接口中，与单独调用这些函数不同，通常更方便使用 `ax.set` 方法一次性设置所有这些属性（见 Figure 26-14）。

```py
In [15]: ax = plt.axes()
         ax.plot(x, np.sin(x))
         ax.set(xlim=(0, 10), ylim=(-2, 2),
                xlabel='x', ylabel='sin(x)',
                title='A Simple Plot');
```

![output 36 0](img/output_36_0.png)

###### Figure 26-14\. 使用 `ax.set` 一次性设置多个属性的示例
