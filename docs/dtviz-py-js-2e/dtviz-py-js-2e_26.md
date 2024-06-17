# 第 20 章\. 可视化个别获奖者

我们希望我们的诺贝尔奖可视化（Nobel-viz）包括一个当前选定获奖者列表和一个生物框（也称为 bio-box），用于显示个别获奖者的详细信息（参见[图 20-1](#d3bio_target)）。通过点击列表中的获奖者，用户可以在生物框中查看其详细信息。在本章中，我们将看到如何构建列表和生物框，如何在用户选择新数据时重新填充列表（通过菜单栏过滤器），以及如何使列表可点击。

![dpj2 2001](assets/dpj2_2001.png)

###### 图 20-1\. 章节目标元素

正如本章所示，D3 不仅用于构建 SVG 可视化。您可以将数据绑定到任何 DOM 元素，并使用它来更改其属性和属性或其事件处理回调函数。D3 的数据连接和事件处理（通过`on`方法实现）非常适用于本章的可点击列表和选择框等常见用户界面。^([1](ch20.xhtml#idm45607738336336))

让我们首先处理获奖者列表及其如何在当前选定获奖者数据集中构建。

# 创建列表

我们使用 HTML 表格构建获奖者列表（参见[图 20-1](#d3bio_target)），其中包含年份、类别和姓名列。此列表的基本骨架在 Nobel-viz 的 *index.xhtml* 文件中提供：

```py
<!DOCTYPE html>
<meta charset="utf-8">
<body>
...
    <div id="nobel-list">
      <h2>Selected winners</h2>
      <table>
        <thead>
          <tr>
            <th id='year'>Year</th>
            <th id='category'>Category</th>
            <th id='name'>Name</th>
          </tr>
        </thead>
        <tbody>
        </tbody>
      </table>
    </div>
...
</body>
```

我们将在 *style.css* 中使用少量 CSS 来样式化这个表格，调整列的宽度和字体大小：

```py
/* WINNERS LIST */
#nobel-list { overflow: scroll; overflow-x: hidden; } ![1](assets/1.png)

#nobel-list table{ font-size: 10px; }
#nobel-list table th#year { width: 30px }
#nobel-list table th#category { width: 120px }
#nobel-list table th#name { width: 120px }

#nobel-list h2 { font-size: 14px; margin: 4px;
text-align: center }
```

[![1](assets/1.png)](#co_visualizing_individual_winners_CO1-1)

`overflow: scroll` 将列表内容裁剪（保持在`nobel-list`容器内），并添加滚动条，以便访问所有获奖者。 `overflow-x: hidden` 抑制了水平滚动条的添加。

为了创建列表，我们将为当前数据集中的每个获奖者向表格的`<tbody>`元素添加`<tr>`行元素（每列包含一个`<td>`数据标签），生成如下内容：

```py
        ...
        <tbody>
          <tr>
            <td>2014</td>
            <td>Chemistry</td>
            <td>Eric Betzig</td>
          </tr>
          ...
        </tbody>
        ...
```

要创建这些行，我们的中心`onDataChange`在应用初始化时将调用`updateList`方法，并在用户应用数据过滤器并更改获奖者列表时随后调用（参见[“基本数据流”](ch16.xhtml#d3build_sect_components)）。`updateList`接收到的数据将具有以下结构：

```py
// data =
[{
  name:"C\u00e9sar Milstein",
  category:"Physiology or Medicine",
  gender:"male",
  country:"Argentina",
  year: 1984
  _id: "5693be6c26a7113f2cc0b3f4"
 },
 ...
]
```

[示例 20-1](#d3bio_list_code) 展示了`updateList`方法。首先按年份对接收到的数据进行排序，然后在移除任何现有行后，用于构建表格行。

##### 示例 20-1\. 创建选定获奖者列表

```py
let updateList = function (data) {
  let tableBody, rows, cells
  // Sort the winners' data by year
  data = data.sort(function (a, b) {
    return +b.year - +a.year
  })
  // select table-body from index.xhtml
  tableBody = d3.select('#nobel-list tbody')
  // create place-holder rows bound to winners' data
  rows = tableBody.selectAll('tr').data(data)

  rows.join( ![1](assets/1.png)
    (enter) => {
      // create any new rows required
      return enter.append('tr').on('click', function (event, d) { ![2](assets/2.png)
        console.log('You clicked a row ' + JSON.stringify(d))
        displayWinner(d)
      })
    },
    (update) => update,
    (exit) => {
      return exit ![3](assets/3.png)
        .transition()
        .duration(nbviz.TRANS_DURATION)
        .style('opacity', 0)
        .remove()
    }
  )

  cells = tableBody ![4](assets/4.png)
    .selectAll('tr')
    .selectAll('td')
    .data(function (d) {
      return [d.year, d.category, d.name]
    })
  // Append data cells, then set their text
  cells.join('td').text(d => d) ![5](assets/5.png)
  // Display a random winner if data is available
  if (data.length) { ![6](assets/6.png)
    displayWinner(data[Math.floor(Math.random() * data.length)])
  }
}
```

[![1](assets/1.png)](#co_visualizing_individual_winners_CO2-1)

一个现在熟悉的连接模式，使用绑定的获奖者数据创建和更新列表项。

[![2](assets/2.png)](#co_visualizing_individual_winners_CO2-2)

当用户点击一行时，此点击处理函数将将绑定到该行的获奖者数据传递给`displayWinner`方法，后者将相应地更新生物框。

[![3](assets/3.png)](#co_visualizing_individual_winners_CO2-3)

这个自定义的`exit`函数在两秒的过渡期内淡出任何多余的行，将它们的不透明度降低到零，然后移除它们。

[![4](assets/4.png)](#co_visualizing_individual_winners_CO2-4)

首先，我们使用获奖者的数据创建一个包含年份、类别和姓名的数据数组，这些数据将用于创建行的`<td>`数据单元格…​

[![5](assets/5.png)](#co_visualizing_individual_winners_CO2-5)

…​然后我们将这个数组与行的数据单元格（`td`）连接起来，并用它来设置它们的文本。

[![6](assets/6.png)](#co_visualizing_individual_winners_CO2-6)

每次数据更改时，我们从新数据集中随机选择一个获奖者，并在生物框中显示他或她。

当用户将光标移动到我们的获奖者表格中的一行上时，我们希望突出显示该行，并且还要更改指针的样式为`cursor`，以指示该行可以点击。以下CSS代码解决了这些细节问题，并添加到了我们的*style.css*文件中：

```py
#nobel-list tr:hover{
    cursor: pointer;
    background: lightblue;
}
```

我们的`updateList`方法在点击行或数据更改时（随机选择）调用`displayWinner`方法以构建获奖者的传记框。现在让我们看看如何构建生物框。

# 构建生物框

生物框使用获奖者对象填充一个小型传记的细节。生物框的HTML骨架提供在*index.xhtml*文件中，包括用于生物元素的内容块和一个`readmore`页脚，提供了一个指向获奖者更多信息的维基百科链接：

```py
<!DOCTYPE html>
<meta charset="utf-8">
<body>
...
    <div id="nobel-winner">
      <div id="picbox"></div>
      <div id='winner-title'></div>
      <div id='infobox'>
        <div class='property'>
          <div class='label'>Category</div>
          <span name='category'></span>
        </div>
        <div class='property'>
          <div class='label'>Year</div>
          <span name='year'></span>
        </div>
        <div class='property'>
          <div class='label'>Country</div>
          <span name='country'></span>
        </div>
      </div>
      <div id='biobox'></div>
      <div id='readmore'>
        <a href='#'>Read more at Wikipedia</a></div>
    </div>
...
</body>
```

在*style.css*中的一点点CSS设置了列表和生物框元素的位置，调整了它们的内容块大小，并提供了边框和字体的具体设置：

```py
/* WINNER INFOBOX */

#nobel-winner {
    font-size: 11px;
    overflow: auto;
    overflow-x: hidden;
    border-top: 4px solid;
}

#nobel-winner #winner-title {
    font-size: 12px;
    text-align: center;
    padding: 2px;
    font-weight: bold;
}

#nobel-winner #infobox .label {
    display: inline-block;
    width: 60px;
    font-weight: bold;
}

#nobel-winner #biobox { font-size: 11px; }
#nobel-winner #biobox p { text-align: justify; }

#nobel-winner #picbox {
    float: right;
    margin-left: 5px;
}
#nobel-winner #picbox img { width:100px; }

#nobel-winner #readmore {
    font-weight: bold;
    text-align: center;
}
```

确定我们的内容块就位后，我们需要回调我们的数据API以获取填充它们所需的数据。[示例 20-2](#d3bio_winner_code)展示了用于构建生物框的`displayWinner`方法。

##### 示例 20-2\. 更新所选获奖者的传记框

```py
let displayWinner = function (wData) {
  // store the winner's bio-box element
  let nw = d3.select('#nobel-winner')

  nw.select('#winner-title').text(wData.name)
  nw.style('border-color', nbviz.categoryFill(wData.category)) ![1](assets/1.png)

  nw.selectAll('.property span').text(function (d) { ![2](assets/2.png)
    var property = d3.select(this).attr('name')
    return wData[property]
  })

  nw.select('#biobox').html(wData.mini_bio)
  // Add an image if available, otherwise remove the old one
  if (wData.bio_image) { ![3](assets/3.png)
    nw.select('#picbox img')
      .attr('src', 'static/images/winners/' + wData.bio_image)
      .style('display', 'inline')
  } else {
    nw.select('#picbox img').style('display', 'none')
  }

  nw.select('#readmore a').attr( ![4](assets/4.png)
    'href',
    'http://en.wikipedia.org/wiki/' + wData.name
  )
}
```

[![1](assets/1.png)](#co_visualizing_individual_winners_CO3-1)

我们的`nobel-winner`元素有一个顶部边框（CSS: `border-top: 4px solid`），我们将根据获奖者的类别使用*nbviz_core.js*中定义的`categoryFill`方法来着色。

[![2](assets/2.png)](#co_visualizing_individual_winners_CO3-2)

我们选择所有具有类名`property`的`div`的`<span>`标签。这些标签的形式是`<span name=*category*></span>`。我们使用span的`name`属性从我们的诺贝尔获奖者数据中检索正确的属性，并将其用于设置标签的文本。

[![3](assets/3.png)](#co_visualizing_individual_winners_CO3-3)

在这里，如果有获奖者的图片可用，我们会设置其`src`（源）属性。如果没有可用的图片，我们使用图像标签的`display`属性将其隐藏（设置为`none`），或者显示它（默认为`inline`）。

[![4](assets/4.png)](#co_visualizing_individual_winners_CO3-4)

我们的获奖者姓名是从维基百科上获取的，可以用于检索他们的维基百科页面。

## 更新获奖者名单

当详细信息（获奖者名单和简介）模块被导入时，它会将回调函数附加到核心模块的回调数组中。当响应用户交互更新数据时，这个回调函数被调用，并且使用Crossfilter的国家维度更新列表，显示新的国家数据：

```py
nbviz.callbacks.push(() => { ![1](assets/1.png)
  let data = nbviz.countryDim.top(Infinity)
  updateList(data)
})
```

[![1](assets/1.png)](#co_visualizing_individual_winners_CO4-1)

当数据更新时，这个匿名函数在核心模块中被调用。

现在我们已经看到如何通过允许用户显示获奖者传记来为我们的Nobel-viz增添一些个性化内容，让我们在进入菜单栏构建的细节之前总结本章内容。

# 总结

在本章中，我们看到了D3如何用于构建传统的HTML结构，而不仅仅是SVG图形。D3不仅能够构建列表、表格等内容，就像显示圆圈或改变线条旋转一样自如。只要有需要通过网页元素反映的变化数据，D3通常能够优雅而高效地解决问题。

在我们完成获奖者名单和传记框的介绍后，我们已经看到了如何构建Nobel-viz中的所有视觉元素。现在只剩下看看如何构建可视化菜单栏以及它如何通过这些视觉元素反映数据集和奖项度量的变化。

^([1](ch20.xhtml#idm45607738336336-marker)) 我们将在[第21章](ch21.xhtml#chapter_d3_ui)中介绍选择框（作为数据过滤器）。
