# 附录 A. D3 的进入/退出模式

如 [“使用数据更新 DOM”](ch17.xhtml#d3bar_update) 所示，D3 现在有了一个更用户友好的 `join` 方法，来替代基于 `enter`、`exit` 和 `remove` 方法的旧数据连接实现模式。`join` 方法是 D3 的一个很好的补充，但在线上有成千上万的使用旧数据连接模式的示例。为了使用/转换这些示例，了解 D3 连接数据时底层发生的事情会有所帮助。

为了演示 D3 的数据连接，让我们深入了解当 D3 连接数据时的底层原理。让我们从我们没有条形图的图表开始，SVG 画布和图表组都已准备好：

```py
...
    <div id="nobel-bar">
      <svg width="600" height="400">
        <g class="chart" transform="translate(40, 20)"></g>
      </svg>
    </div>
...
```

为了使用 D3 连接数据，我们首先需要以正确的形式准备一些数据。通常这将是一个对象数组，就像我们条形图的 `nobelData`：

```py
var nobelData = [
    {key:'United States', value:336},
    {key:'United Kingdom', value:98},
    {key:'Germany', value:79},
    ...
]
```

D3 数据连接分为两个阶段。首先，我们使用 `data` 方法添加要连接的数据，然后使用 `join` 方法执行连接。

要将我们的诺贝尔数据添加到一组条形图中，我们需要做以下操作。首先，选择一个容器来放置我们的条形图，这里是我们的 `chart` 类 SVG 组。

然后我们定义容器，这里是一个类为 `bar` 的 CSS 选择器：

```py
var svg = d3.select('#nobel-bar .chart');

var bars =  svg.selectAll('.bar')
              .data(nobelData);
```

现在我们来看一下 D3 的 `data` 方法稍微反直觉的一面。我们的第一个 `select` 返回了我们 `nobel-bar` SVG 画布中的 `chart` 组，但第二个 `selectAll` 返回了所有类为 `bars` 的元素，但实际上并没有。如果没有条形图，我们到底将数据绑定到什么？答案是，在幕后，D3 会记录哪些 DOM 元素已绑定到 `nobelData`，以及哪些没有。接下来，我们将看看如何利用这一点，使用基本的 `enter` 方法。

# 进入方法

D3 的 [`enter` 方法](https://oreil.ly/veBF9)（以及其姊妹 `exit`）既是 D3 强大功能和表现力的基础，也是许多混淆的根源。尽管如前所述，新的 `join` 方法简化了事务，但如果您真的希望提高 D3 技能，了解 `enter` 方法也是值得的。现在让我们通过一个非常简单和缓慢的演示来介绍它。

我们将从一个经典简单的小示例开始，为我们的诺贝尔奖数据的每个成员添加一个条形矩形。我们将使用前六个诺贝尔奖获奖国家作为我们的绑定数据：

```py
var nobelData = [
    {key:'United States', value:200},
    {key:'United Kingdom', value:80},
    {key:'France', value:47},
    {key:'Switzerland', value:23},
    {key:'Japan', value:21},
    {key:'Austria', value:12}
];
```

手头有了我们的数据集，让我们先使用 D3 抓取图表组，并将其保存到一个 `svg` 变量中。我们将用它来选择目前不存在的 `bar` 类元素：

```py
var svg = d3.select('#nobel-bar .chart');

var bars = svg.selectAll('.bar')
    .data(nobelData);
```

虽然`bars`选择为空，但在幕后，D3已经记录了我们刚刚绑定到它的数据。此时，我们可以利用这一事实和`enter`方法来使用我们的数据创建一些条形图。在我们的`bars`选择上调用`enter`返回了所有未绑定到条形图的数据的子选择（在本例中为`nobelData`），因为原始选择中没有条形图（我们的图表为空），所有数据都是未绑定的，因此`enter`返回一个大小为六的输入选择（基本上是所有未绑定数据的占位符节点）：

```py
bars = bars.enter(); # returns six placeholder nodes
```

我们可以使用`bars`中的占位符节点创建一些DOM元素——在我们的情况下，一些条形图。我们不会费心试图将它们放置正确（根据惯例，y轴从屏幕顶部向下），但我们会使用数据值和索引来设置条形图的位置和高度：

```py
bars.append('rect')
    .classed('bar', true)
    .attr('width', 10)
    .attr('height', function(d){return d.value;}) ![1](assets/1.png)
    .attr('x', function(d, i) { return i * 12; });
```

[![1](assets/1.png)](#co_d3__8217_s_enter_exit_pattern_CO1-1)

如果您向D3的设置器方法（`attr`、`style`等）提供回调函数，则提供的第一个和第二个参数是个体数据对象的值（例如，`d == {key: 'United States', value: 200}`）和它的索引（`i`）。

使用回调函数设置条形图的高度和x位置（允许2像素的填充），并在我们的六个节点选择上调用`append`将产生[图 A-1](#d3bar_enter_full)。

![dpj2 aa01](assets/dpj2_aa01.png)

###### 图 A-1\. 使用D3的`enter`方法生成一些条形图

我鼓励您通常使用Chrome（或等效）的元素选项卡来调查您的D3生成的HTML。使用元素选项卡查看我们的小型条形图显示[图 A-2](#d3bar_enter_elements)。

![dpj2 aa02](assets/dpj2_aa02.png)

###### 图 A-2\. 使用元素选项卡查看`enter`和`append`生成的HTML

因此，我们已经看到在空选择上调用`enter`时会发生什么。但是，在具有用户驱动的变化数据集的交互式图表中，当我们已经有一些条形图时会发生什么？

让我们在我们的起始HTML中添加一些`bar`类矩形：

```py
<div id="nobel-bar">
  <svg width="600" height="400">
    <g class="chart" transform="translate(40, 20)">
      <rect class='bar'></rect>
      <rect class='bar'></rect>
    </g>
  </svg>
</div>
```

如果我们现在执行与之前相同的数据绑定和输入，在我们的选择上调用`data`，两个占位矩形绑定到我们`nobelData`数组的前两个成员（即`[{key: 'United States', value: 200}, {key: 'United Kingdom', value:80}]`）。这意味着`enter`现在仅返回四个占位符，与`nobelData`数组的最后四个元素关联：

```py
var svg = d3.select('#nobel-bar .chart');

var bars = svg.selectAll('.bar')
    .data(nobelData);

bars = bars.enter(); # return four placeholder nodes
```

如果现在在输入的`bars`上调用`append`，我们将得到[图 A-3](#d3bar_enter_part)中显示的结果，显示最后四个条形图（请注意它们保留了它们的索引`i`，用于设置它们的x位置）。

![dpj2 aa03](assets/dpj2_aa03.png)

###### 图 A-3\. 在现有条形图上调用`enter`和`append`

[图 A-4](#d3bar_enter_elements_part) 显示了最后四个条形图生成的 HTML。正如我们将看到的，前两个元素的数据现在绑定到我们添加到初始条形图组的两个虚拟节点上。我们只是还没有使用它来调整那些矩形的属性。使用新数据更新旧条形图是我们即将看到的更新模式的关键元素之一。

![dpj2 aa04](assets/dpj2_aa04.png)

###### 图 A-4\. 使用元素标签查看由`enter`和`append`在部分选择上生成的 HTML

强调一下，理解`enter`和`exit`（以及`remove`）对于与 D3 健康进展至关重要。多玩一下，检查你生成的 HTML，输入一些数据，并且通常变得有些混乱，学习其方方面面。在进入 D3 的核心——更新模式——之前，让我们先看一看访问绑定数据。

# 访问绑定的数据

查看 DOM 变化的好方法是使用浏览器的 HTML 检查器和控制台跟踪 D3 的变化。在 [图 A-1](#d3bar_enter_full) 中，我们使用 Chrome 的控制台查看代表第一个条形图的`rect`元素，在数据绑定之前和使用`data`方法将`nobelData`绑定到条形图之后。正如你所看到的，D3 已经向`rect`元素添加了一个`__data__`对象，用于存储其绑定的数据——在本例中是我们`nobelData`列表的第一个成员。`__data__`对象由 D3 的内部管理使用，其数据基本上是供给更新方法如`attr`的函数使用的。

让我们看一个使用元素的`__data__`对象中数据的小例子，设置其`name`属性。`name`属性对于制作特定的 D3 选择非常有用。例如，如果用户选择了特定的国家，现在我们可以使用 D3 获取其所有命名组件，并根据需要调整它们的样式。我们将使用在 [图 A-5](#d3bar_console_db) 中绑定数据的条形图，并使用其绑定数据的`key`属性设置名称：

```py
let bar = d3.select('#nobel-bar .bar');

bar.attr('name', function(d, i){ ![1](assets/1.png)

    let sane_key = d.key.replace(/ /g, '_'); ![2](assets/2.png)

    console.log('__data__ is: ' + JSON.stringify(d)
    + ', index is ' + i)

    return 'bar__' + sane_key; ![3](assets/3.png)
    });
// console out: // __data__ is: {"key":"United States","value":336}, index is 0
```

[![1](assets/1.png)](#co_d3__8217_s_enter_exit_pattern_CO2-1)

所有 D3 的设置方法都可以将函数作为它们的第二个参数。这个函数接收绑定到选定元素的数据（`d`）及其在数据数组中的位置（`i`）。

[![2](assets/2.png)](#co_d3__8217_s_enter_exit_pattern_CO2-2)

我们使用正则表达式（regex）将键中的所有空格替换为下划线（例如，美国 → 美国_）。

[![3](assets/3.png)](#co_d3__8217_s_enter_exit_pattern_CO2-3)

这将把条形图的`name`属性设置为`'bar__美国'`。

[图 17-3](ch17.xhtml#d3bar_selects) 中列出的所有设置方法（`attr`、`style`、`text` 等）都可以接受一个函数作为第二个参数，该函数将接收绑定到元素的数据以及元素的数组索引。该函数的返回值用于设置属性的值。正如我们将看到的那样，在绑定新数据并使用这些函数式设置器来适应属性、样式和属性时，交互式可视化将反映出对可视化数据集的更改。

![dpj2 aa05](assets/dpj2_aa05.png)

###### 图 A-5\. 使用 Chrome 控制台展示使用 D3 的 `data` 方法绑定数据后添加 `__data__` 对象
