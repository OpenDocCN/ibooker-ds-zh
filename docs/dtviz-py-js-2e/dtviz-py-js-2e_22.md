# 第 16 章\. 构建可视化

在[第 15 章](ch15.xhtml#chapter_imagining)中，我们利用了我们对诺贝尔奖数据集的pandas探索的结果（参见[第 11 章](ch11.xhtml#chapter_pandas_exploring)），来想象一个可视化效果。[图 16-1](#d3build_target)展示了我们想象的可视化效果，在本章中，我们将看到如何构建它，利用JavaScript和D3的强大功能。

![dpj2 1601](assets/dpj2_1601.png)

###### 图 16-1\. 我们的目标，诺贝尔奖可视化

我将展示我们构想的视觉元素如何结合，将我们新鲜清理和处理的诺贝尔数据集转换为交互式网络可视化，可以轻松部署到数十亿设备上。但在深入细节之前，让我们先看看现代网络可视化的核心组件。

# 准备工作

在开始构建诺贝尔奖可视化之前，让我们考虑将使用的核心组件以及如何组织我们的文件。

## 核心组件

正如我们在[“一个基本页面带有占位符”](ch04.xhtml#webdev101_basic_page)中所看到的，构建现代网络可视化需要四个关键组件：

+   一个HTML框架，用于支撑我们的JavaScript创建

+   一个或多个CSS文件来控制数据可视化的外观和感觉

+   JavaScript 文件本身，包括可能需要的任何第三方库（D3 是我们最大的依赖项）

+   最后但同样重要的是，将要转换的数据，理想情况下是JSON或CSV格式（如果是完全静态数据）

在我们开始查看数据可视化组件之前，让我们先为我们的诺贝尔奖可视化（Nobel-viz）项目准备好文件结构，并确定如何向可视化提供数据。

## 组织您的文件

[示例 16-1](#nviz_files)展示了我们项目目录的结构。按照惯例，我们在根目录下有一个*index.xhtml*文件，其中包含所有用于可视化的库和资产（图像和数据）的*static*目录。

##### 示例 16-1\. 我们的诺贝尔奖可视化项目的文件结构

```py
nobel_viz
├── index.xhtml
└── static
    ├── css
    │   └── style.css
    ├── data          ![1](assets/1.png)
    │   ├── nobel_winners_biopic.json
    │   ├── winning_country_data.json
    │   ├── world-110m.json
    │   └── world-country-names-nobel.csv
    ├── images
    │   └── winners  ![2](assets/2.png)
    │       └── full
    │           ├── 002b4f05aa3758e2d6acadde4ed80aa991ed6357.jpg
    │           ├── 00d7ed381db8b5d18edc84694b7f9ce14ee57c5b.jpg
    │           ├── ...
    └── js                  ![3](assets/3.png)
        ├── nbviz_bar.mjs
        ├── nbviz_core.mjs
        ├── nbviz_details.mjs
        ├── nbviz_main.mjs
        ├── nbviz_map.mjs
        ├── nbviz_menu.mjs
        └── nbviz_time.mjs
    └── libs
        ├── crossfilter.min.js
        ├── d3.min.js
        └── topojson.min.js
```

[![1](assets/1.png)](#co_building_a_visualization_CO1-1)

我们将使用的静态数据文件，包括TopoJSON世界地图（详见[第 19 章](ch19.xhtml#chapter_d3_maps)）和从网络上抓取的国家数据（详见[“获取诺贝尔数据可视化的国家数据”](ch05.xhtml#country_data)）。

[![2](assets/2.png)](#co_building_a_visualization_CO1-2)

我们使用Scrapy爬取的诺贝尔奖获奖者的照片，详见[“使用管道进行文本和图像爬取”](ch06.xhtml#scraping_bio)。

[![3](assets/3.png)](#co_building_a_visualization_CO1-3)

*js*子目录包含我们的Nobel-viz JavaScript模块文件（*.mjs*），分为核心元素，并以*nbviz_*开头。

## 提供数据

我们在[第 6 章](ch06.xhtml#chapter_heavy_scraping)中抓取的包含小传的完整诺贝尔数据集大约有三兆字节的数据，在网络传输时经过压缩后则显著减少。按照现代网页的标准，这并不算是一大量的数据。实际上，平均网页大小大约在 2MB 到 3MB 之间^([1](ch16.xhtml#idm45607751186112))。尽管如此，它接近一个我们可能需要考虑将其分成更小的块以便根据需要加载的点。我们也可以使用像 SQLite 这样的数据库从 web 服务器动态提供数据（参见[第 13 章](ch13.xhtml#chapter_delivery_restful)）。正如现在这样，初始等待时间的小不便被浏览器缓存所有数据后的高速性能所补偿。而且只需最初获取一次数据使事情变得简单多了。

对于我们的诺贝尔可视化，我们将从数据目录中提供所有数据（见[示例 16-1](#nviz_files)，＃2），在应用程序初始化时获取。

# HTML 骨架

尽管我们的诺贝尔可视化有许多动态组件，但所需的 HTML 骨架却令人惊讶地简单。这展示了本书的一个核心主题——为了编程数据可视化，你并不需要非常传统的 web 开发技能。

*index.xhtml* 文件，在加载时创建可视化，在 [示例 16-2](#d3build_index) 中展示。三个组成部分是：

1.  导入了 CSS 样式表 *style.css*，设置字体、内容块位置等。

1.  HTML 占位符用 ID 形式为 `nobel-[foo]` 的视觉元素。

1.  JavaScript; 首先是第三方库，然后是我们的原始脚本。

我们将在接下来的章节详细介绍各个 HTML 部分，但我希望你能看到这个诺贝尔奖可视化的整体非编程元素。有了这个基本框架，你可以转向创意编程的工作，这正是 D3 鼓励和擅长的。当你习惯于在 HTML 中定义内容块，并使用 CSS 固定尺寸和定位时，你会发现你越来越多地花时间做你最喜欢的事情：用代码操纵数据。

###### 小贴士

我发现将识别的占位符，如地图容器 `<div id="nobel-map"></div>`，视为其各自元素的*所有权* 是很有帮助的。我们在主 CSS 或 JS^([2](ch16.xhtml#idm45607751164464)) 文件中设置这些框架的维度和相对定位，而动态地图等元素则根据其框架的大小自适应。这允许非编程设计师通过 CSS 样式更改可视化的外观和感觉。

##### 示例 16-2\. 我们的单页可视化访问文件 `index.xhtml`。

```py
<!DOCTYPE html>
<meta charset="utf-8">
<title>Visualizing the Nobel Prize</title>
<!-- 1\. IMPORT THE visualization'S CSS STYLING -->
<link rel="stylesheet" href="static/css/style.css"
media="screen" />
<body>
  <div id='chart'>
    <!-- 2\. A HEADER WITH TITLE AND SOME EXPLANATORY INFO -->
    <div id='title'>Visualizing the Nobel Prize</div>
    <div id="info">
      This is a companion piece to the book <a href='http://'>
      Data visualization with Python and JavaScript</a>, in which
      its construction is detailed. The data used was scraped
      Wikipedia using the <a href=
      'https://en.wikipedia.org/wiki
 /List_of_Nobel_laureates_by_country'>
      list of winners by country</a> as a starting point. The
      accompanying GitHub repo is <a href=
      'http://github.com/Kyrand/dataviz-with-python-and-js-ed-2'>
      here</a>.
    </div>
    <!-- 3\. THE PLACEHOLDERS FOR OUR VISUAL COMPONENTS  -->
    <div id="nbviz">
      <!-- BEGIN MENU BAR -->
      <div id="nobel-menu">
        <div id="cat-select">
          Category
          <select></select>
        </div>
        <div id="gender-select">
          Gender
          <select>
            <option value="All">All</option>
            <option value="female">Female</option>
            <option value="male">Male</option>
          </select>
        </div>
        <div id="country-select">
          Country
          <select></select>
        </div>
        <div id='metric-radio'>
          Number of Winners:&nbsp;
          <form>
            <label>absolute
              <input type="radio" name="mode" value="0" checked>
            </label>
            <label>per-capita
              <input type="radio" name="mode" value="1">
            </label>
          </form>
        </div>
      </div>
      <!-- END MENU BAR  -->
      <!-- BEGIN NOBEL-VIZ COMPONENTS -->
      <div id='chart-holder' class='_dev'>
        <!-- TIME LINE OF PRIZES -->
        <div id="nobel-time"></div>
        <!-- MAP AND TOOLTIP -->
        <div id="nobel-map">
          <div id="map-tooltip">
            <h2></h2>
            <p></p>
          </div>
        </div>
        <!-- LIST OF WINNERS -->
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
        <!-- BIOGRAPHY BOX -->
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
            <a href='#'>Read more at Wikipedia</a>
          </div>
        </div>
        <!-- NOBEL BAR CHART -->
        <div id="nobel-bar"></div>
      </div>
      <!-- END NOBEL-VIZ COMPONENTS -->
    </div>
  </div>
  <!-- 4\. THE JAVASCRIPT FILES -->
  <!-- THIRD-PARTY JAVASCRIPT LIBRARIES, MAINLY D3  -->
  <script src="libs/d3.min.js"></script>
  <!-- ... -->
  <!-- THE MAIN JAVASCRIPT MODULE FOR OUR NOBEL ELEMENTS -->
  <script src="static/js/nbviz_main.mjs" ></script>
</body>
```

HTML 框架（[示例 16-2](#d3build_index)）定义了我们 Nobel-viz 组件的层次结构，但它们的视觉大小和定位是在 *style.css* 文件中设置的。在接下来的部分中，我们将看到这是如何完成的，并查看我们可视化的一般样式。

# CSS 样式

我们将在各自的章节中处理我们图表中各个图表组件的样式（[图 16-1](#d3build_target)）。本节将涵盖其余的非特定 CSS，最重要的是我们元素内容块（*面板*）的大小和定位。

可视化的大小是一个棘手的选择。现在有许多不同的设备格式，如智能手机、平板电脑、移动设备等，具有多种不同的分辨率，如“视网膜”^([3](ch16.xhtml#idm45607750526128)) 和全高清（1,920×1,080）。因此，像素尺寸比以前更加多样化，像素密度变得更具有意义。大多数设备执行像素缩放来进行补偿，这就是为什么您可以在智能手机上仍然阅读文字，即使它具有与大型桌面显示器相同数量的像素。此外，大多数手持设备具有捏缩放和平移功能，允许用户轻松关注较大数据可视化的区域。对于我们的 Nobel 数据可视化，我们将选择一个折衷的分辨率 1,280×800 像素，这在大多数桌面监视器上看起来应该还行，并且在移动设备的横向模式下可用，包括我们 50 像素高的顶部用户控件。

首先，我们使用 `body` 选择器设置了一些通用的样式，应用于整个文档；一个无衬线字体，浅白色背景和一些链接细节被指定。我们还设置了可视化的宽度和其边距：

```py
body {
    font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
    background: #fefefe; ![1](assets/1.png)
    width: 1000px;
    margin: 0 auto; /* top and bottom 0, left and right auto */
}

a:link {
    color: royalblue;
    text-decoration: none; ![2](assets/2.png)
}

a:hover {
    text-decoration: underline;
}
```

[![1](assets/1.png)](#co_building_a_visualization_CO2-1)

这种颜色略偏白色（`#ffffff`），应有助于使页面稍微不那么亮，更易于眼睛。

[![2](assets/2.png)](#co_building_a_visualization_CO2-2)

我认为默认的下划线超链接看起来有点繁琐，因此我们去掉了装饰。

我们 Nobel-viz 有三个主要的 div 内容块，它们绝对定位在 `#chart` div 内（它们的相对父元素）。这些是主标题（`#title`）、可视化信息（`#info`）和主容器（`#nbviz`）。标题和信息是凭眼观察放置的，而主容器距页面顶部90像素，以便为它们留出空间，并且宽度设置为100%以便扩展到可用空间。以下 CSS 实现了这一点：

```py
#nbviz {
    position: absolute;
    top: 90px;
    width: 100%;
}

#title {
    position: absolute;
    font-size: 30px;
    font-weight: 100;
    top: 20px;
}

#info {
    position: absolute;
    font-size: 11px;
    top: 18px;
    width: 300px;
    right: 0px;
    line-height: 1.2;
}
```

`chart-holder` 的高度设置为 750 像素，宽度设置为其父元素的 100%，并具有 `relative` 的 `position` 属性，这意味着其子面板的绝对定位将相对于其左上角。我们的图表底部填充了 20 像素：

```py
#chart-holder {
    width: 100%;
    height: 750px;
    position: relative;
    padding: 0 0 20px 0; /* top right bottom left */
}

#chart-holder svg { ![1](assets/1.png)
    width: 100%;
    height: 100%;
}
```

[![1](assets/1.png)](#co_building_a_visualization_CO3-1)

我们希望我们组件的 SVG 上下文能够扩展以适应它们的容器。

考虑到 Nobel-viz 的高度约束为 750 像素，我们的等经纬度地图的宽高比为二，^([4](ch16.xhtml#idm45607750213296)) 并且需要将 100 年以上的诺贝尔奖圆形指示器适应到我们的时间图表中，根据尺寸进行调整建议将 [图 16-2](#d3build_dimensions) 作为我们可视化元素大小的一个良好折衷。

![dpj2 1602](assets/dpj2_1602.png)

###### 图 16-2\. Nobel-viz 的尺寸

此 CSS 样式按照 [图 16-2](#d3build_dimensions) 所示的位置和大小组件。

```py
#nobel-map, #nobel-winner, #nobel-bar, #nobel-time, #nobel-list{
    position:absolute; ![1](assets/1.png)
}

#nobel-time {
    top: 0;
    height: 150px;
    width: 100%; ![2](assets/2.png)
}

#nobel-map {
    background: azure;
    top: 160px;
    width: 700px;
    height: 350px;
}

#nobel-winner {
    top: 510px;
    left: 700px;
    height: 240px;
    width: 300px;
}

#nobel-bar {
    top: 510px;
    height: 240px;
    width: 700px;
}

#nobel-list {
    top: 160px;
    height: 340px;
    width: 290px;
    left: 700px;
    padding-left: 10px; ![3](assets/3.png)
}
```

[![1](assets/1.png)](#co_building_a_visualization_CO4-1)

我们希望绝对的手动调整位置，相对于 `chart-holder` 父容器。

[![2](assets/2.png)](#co_building_a_visualization_CO4-2)

时间轴占据了可视化的整个宽度。

[![3](assets/3.png)](#co_building_a_visualization_CO4-3)

您可以使用填充使组件有“呼吸空间”。

其他 CSS 样式特定于各个组件，并将在各自的章节中进行介绍。通过前面的 CSS，我们在 HTML 骨架上使用 JavaScript 来丰富我们的可视化。

# JavaScript 引擎

在任何尺寸的可视化中，早期实施一些模块化是很好的。网上许多 D3 的例子^([5](ch16.xhtml#idm45607749985552)) 都是单页面解决方案，将 HTML、CSS、JS 甚至数据都结合在一个页面上。虽然这对通过示例教学很好，但随着代码库的增长，情况将迅速恶化，使得修改变得困难，并增加了命名空间冲突等问题的可能性。

## 导入脚本

我们使用 `<script>` 标签将 JavaScript 文件包含在我们入口的 *index.xhtml* 文件的 `<body>` 标签底部，如 [示例 16-2](#d3build_index) 所示：

```py
<!DOCTYPE html>
<meta charset="utf-8"> ... <body> ... <!-- THIRD-PARTY JAVASCRIPT LIBRARIES, MAINLY D3 BASED  --> ![1](assets/1.png)
  <script src="static/libs/d3.min.js"></script>
  <script src="static/libs/topojson.min.js"></script>
  <script src="static/libs/crossfilter.min.js"></script>
  <!-- THE JAVASCRIPT FOR OUR NOBEL ELEMENTS -->
  <script src="static/js/nbviz_main.mjs"></script> ![2](assets/2.png)
</body>
```

[![1](assets/1.png)](#co_building_a_visualization_CO5-1)

我们使用第三方库的本地副本。

[![2](assets/2.png)](#co_building_a_visualization_CO5-2)

我们诺贝尔应用的主入口点，它请求其第一个数据集并启动显示。该模块导入了可视化中使用的所有其他模块。

## 模块化的 JS 与导入

在本书的第一版中，为建立 `nbviz` 命名空间，以便在诺贝尔数据可视化的各个组件中放置函数、变量、常量等，采用了一种常见但相当巧妙的模式。这里是一个示例，因为您可能会在现实中遇到类似的模式：

```py
/* js/nbviz_core.js
/* global $, _, crossfilter, d3  */ ![1](assets/1.png)
(function(nbviz) {
     //... MODULES PRIVATE VARS ETC..
     nbviz.foo = function(){ //... ![2](assets/2.png)
     };
}(window.nbviz = window.nbviz || {})); ![3](assets/3.png)
```

[![1](assets/1.png)](#co_building_a_visualization_CO6-1)

将变量定义为全局变量将防止它们触发[JSLint 错误](https://www.jslint.com)。

[![2](assets/2.png)](#co_building_a_visualization_CO6-2)

将此函数作为共享 `nbviz` 命名空间的一部分暴露给其他脚本。

[![3](assets/3.png)](#co_building_a_visualization_CO6-3)

如果可用，则使用 `nbviz` 对象，并在没有时创建它。

每个JS脚本都用这种模式封装，所有必需的脚本都包含在主 *index.xhtml* 的 `<script>` 标签中。随着跨浏览器支持JS模块的到来，我们有了一种更清洁、现代的方式来包含我们的JavaScript，这对于任何Pythonista都是熟悉的（参见 [“JavaScript Modules”](ch02.xhtml#sect_js_modules)）。

现在我们只需在 *index.xhtml* 中包含我们的主JS模块，这将导入所有其他所需的模块：

```py
// static/js/nbviz_main.mjs import nbviz from './nbviz_core.mjs'
import { initMenu } from './nbviz_menu.mjs'
import { initMap } from './nbviz_map.mjs'
import './nbviz_bar.mjs' ![1](assets/1.png)
import './nbviz_details.mjs'
import './nbviz_time.mjs'
```

[![1](assets/1.png)](#co_building_a_visualization_CO7-1)

这些被导入以初始化它们的更新回调。我们将在本章后面看到这是如何工作的。

在接下来的章节中，将详细解释用于生成可视化元素的JavaScript/D3。首先，我们将处理数据从（数据）服务器流向客户端浏览器，并在客户端内部由用户交互驱动的Nobel-viz数据流。

## 基本数据流

处理任何复杂项目中的数据有许多方法。对于交互式应用程序，特别是数据可视化，我发现最健壮的模式是拥有一个中央数据对象来缓存当前数据。除了缓存的数据外，我们还有一些主数据对象中存储的活动反映或子集。例如，在我们的Nobel-viz中，用户可以选择数据的多个子集（例如，只有物理类别的获奖者）。

如果用户触发了不同的数据反映，比如选择每人均奖金指标，一个标志^([6](ch16.xhtml#idm45607749683584))就会被设置（在这种情况下，`valuePerCapita` 被设置为 `0` 或 `1`）。然后我们更新所有的视觉组件，依赖于 `valuePerCapita` 的那些组件会相应地适应。地图指示器的大小会改变，柱状图会重新组织。

关键思想是确保视觉元素与用户驱动的数据变化同步。做到这一点的一种可靠方法是拥有一个单一的更新方法（这里称为 `onDataChange`），每当用户执行某些操作以更改数据时就调用此方法。该方法通知所有活动的视觉元素数据已更改，它们会相应地做出响应。

现在让我们看看应用程序的代码如何配合，从共享的核心工具开始。

## 核心代码

第一个加载的JavaScript文件是 *nbviz_core.js*。该脚本包含了我们可能希望在其他脚本中共享的任何代码。例如，我们有一个 `categoryFill` 方法，为每个类别返回特定的颜色。这被时间线组件使用，并作为传记框中的边框。这个核心代码包括我们可能想要隔离以进行测试的函数，或者只是为了使其他模块更清晰。

###### 提示

在编程中经常使用字符串常量作为字典键、比较项和生成的标签。在需要时输入这些字符串很容易养成坏习惯，但更好的方法是定义一个常量变量。例如，不使用`'if option === "All Categories"'`，而是使用`'if option === nbviz.ALL_CATS'`。在前者中，误输`'All Categories'`不会引发错误，这是一场意外。拥有`const`还意味着只需编辑一次即可更改所有字符串的出现。JavaScript有一个新的`const`关键字，使得强制常量变得更容易，尽管它仅阻止变量被重新赋值。参见[Mozilla文档](https://oreil.ly/AlEbm)获取一些示例和`const`限制的详细说明。

[示例 16-3](#build_core)展示了在其他模块之间共享的代码。任何打算供其他模块使用的内容都附加在共享的`nbviz`命名空间上。

##### 示例 16-3\. 在 nbviz_core.js 中的共享代码库

```py
let nbviz = {}
nbviz.ALL_CATS = 'All Categories'
nbviz.TRANS_DURATION = 2000 // time in ms for our visual transitions nbviz.MAX_CENTROID_RADIUS = 30
nbviz.MIN_CENTROID_RADIUS = 2
nbviz.COLORS = { palegold: '#E6BE8A' } // any named colors used 
nbviz.data = {} // our main data store nbviz.valuePerCapita = 0 // metric flag nbviz.activeCountry = null
nbviz.activeCategory = nbviz.ALL_CATS

nbviz.CATEGORIES = [
    "Chemistry", "Economics", "Literature", "Peace",
    "Physics", "Physiology or Medicine"
];
// takes a category like Physics and returns a color nbviz.categoryFill = function(category){
    var i = nbviz.CATEGORIES.indexOf(category);
    return d3.schemeCategory10[i]; ![1](assets/1.png)
};

let nestDataByYear = function(entries) {          ![2](assets/2.png)
//... };

nbviz.makeFilterAndDimensions = function(winnersData){
//... };

nbviz.filterByCountries = function(countryNames) {
//... };

nbviz.filterByCategory = function(cat) {
//... };

nbviz.getCountryData = function() {
// ... };

nbviz.callbacks = []
nbviz.onDataChange = function () { ![3](assets/3.png)
  nbviz.callbacks.forEach((cb) => cb())
}

export default nbviz ![4](assets/4.png)
```

[![1](assets/1.png)](#co_building_a_visualization_CO8-1)

我们使用 D3 的内置颜色方案来提供奖项类别颜色。`schemeCategory10`是一个包含 10 个颜色十六进制码的数组（`['#1f77b4', '#ff7f0e',...]`），我们通过类别索引来访问。

[![2](assets/2.png)](#co_building_a_visualization_CO8-2)

此处和以下的空方法将在接下来的章节中根据使用情境进行详细解释。

[![3](assets/3.png)](#co_building_a_visualization_CO8-3)

当数据集更改时（在应用程序初始化后，这是用户驱动的），调用此函数更新诺贝尔可视化元素。按顺序调用由组件模块设置并存储在`callbacks`数组中的更新回调，触发任何必要的视觉变化。详见[“基本数据流”](#d3build_sect_components)了解详细信息。

[![4](assets/4.png)](#co_building_a_visualization_CO8-4)

`nbviz`对象具有实用函数、常量和变量，是该模块的默认导出项，由其他模块导入，因此使用`import nbviz from ​./⁠nbviz_core`。

有了核心代码，让我们看看如何使用 D3 的实用方法初始化我们的应用程序来获取静态资源。

## 初始化诺贝尔奖可视化

为了启动应用程序，我们需要一些数据。我们使用 D3 的`json`和`csv`辅助函数加载数据并将其转换为 JavaScript 对象和数组。使用`Promise.all`^([7](ch16.xhtml#idm45607749511824))方法同时发起这些数据获取请求，等待所有四个请求都完成，然后将数据传递给指定的处理函数，在本例中为`ready`：

```py
// static/js/nbviz_main.mjs //...
  Promise.all([ ![1](assets/1.png)
    d3.json('static/data/world-110m.json'),
    d3.csv('static/data/world-country-names-nobel.csv'),
    d3.json('static/data/winning_country_data.json'),
    d3.json('static/data/nobel_winners_biopic.json'),
  ]).then(ready)

  function ready([worldMap, countryNames, countryData, winnersData]) { ![2](assets/2.png)
    // STORE OUR COUNTRY-DATA DATASET
    nbviz.data.countryData = countryData
    nbviz.data.winnersData = winnersData
    //... }
```

[![1](assets/1.png)](#co_building_a_visualization_CO9-1)

同时发起对四个数据文件的请求。这些静态文件包括一个世界地图（110m 分辨率）和一些我们将在可视化中使用的国家数据。

[![2](assets/2.png)](#co_building_a_visualization_CO9-2)

返回给`ready`的数组使用[JavaScript解构](https://oreil.ly/RZzXm)将顺序数据分配给相应的变量。

如果我们的数据请求成功，`ready`函数将接收所请求的数据，并准备好向可视元素发送数据。

## 准备就绪

在由`Promise.all`方法发起的延迟数据请求解决后，它调用指定的`ready`函数，并按添加顺序将数据集作为参数传递。

`ready`函数在[示例 16-4](#build_ready)中展示。如果数据下载没有错误，我们将使用获奖者数据创建一个活动过滤器（由Crossfilter库提供），用于允许用户根据类别、性别和国家选择诺贝尔获奖者的子集。然后调用一些初始化方法，最后使用`onDataChange`方法触发数据可视化元素的绘制，更新条形图、地图、时间线等。[图 16-3](#d3build_work_flow)中的示意图展示了数据变化传播的方式。

##### 示例 16-4\. 当初始数据请求已解决时调用`ready`函数

```py
//...
  function ready([worldMap, countryNames, countryData, winnersData]) {
    // STORE OUR COUNTRY-DATA DATASET
    nbviz.data.countryData = countryData
    nbviz.data.winnersData = winnersData
    // MAKE OUR FILTER AND ITS DIMENSIONS
    nbviz.makeFilterAndDimensions(winnersData) ![1](assets/1.png)
    // INITIALIZE MENU AND MAP
    initMenu()
    initMap(worldMap, countryNames)
    // TRIGGER UPDATE WITH FULL WINNERS' DATASET
    nbviz.onDataChange()
  }
```

[![1](assets/1.png)](#co_building_a_visualization_CO10-1)

该方法利用新加载的诺贝尔奖数据集创建我们将用于允许用户选择要可视化的数据子集的过滤器。详见“Filtering Data with Crossfilter”章节。

我们将在介绍Crossfilter库中的“Filtering Data with Crossfilter”章节时看到`makeFilterAndDimensions`方法（[示例 16-4](#build_ready)，![1](assets/1.png)）的工作原理。暂时假设我们有一种方法通过一些菜单选择器（例如选择所有女性获奖者）获取用户当前选择的数据。

![dpj2 1603](assets/dpj2_1603.png)

###### 图 16-3\. 应用程序的主要数据流

## 数据驱动更新

在`ready`函数中初始化菜单和地图（我们将在各自的章节中详细讨论其工作原理：[第19章](ch19.xhtml#chapter_d3_maps)讨论地图，[第21章](ch21.xhtml#chapter_d3_ui)讨论菜单）后，我们使用*nbviz_core.js*中定义的`onDataChange`方法触发可视元素的更新。`onDataChange`（参见[示例 16-5](#src_ondatachange)）是一个共享函数，当显示的数据集因用户交互而改变，或者用户选择不同的国家奖励度量（例如按人均测量而不是绝对数字）时调用。

##### 示例 16-5\. 当选定数据更改时调用的函数以更新可视元素

```py
// nbviz_core.js nbviz.callbacks = [] ![1](assets/1.png)

nbviz.onDataChange = function () {
  nbviz.callbacks.forEach((cb) => cb()) ![2](assets/2.png)
}
```

[![1](assets/1.png)](#co_building_a_visualization_CO11-1)

需要更新的每个组件模块都将其回调追加到此数组中。

[![2](assets/2.png)](#co_building_a_visualization_CO11-2)

数据变化时，依次调用组件回调函数，触发任何必要的视觉变化以反映新数据。

当模块首次导入时，它们将它们的回调添加到核心模块中的`callbacks`数组中。例如，这是条形图：

```py
// nbviz_bar.mjs import nbviz from './nbviz_core.mjs'
// ... nbviz.callbacks.push(() => {
  let data = nbviz.getCountryData()
  updateBarChart(data) ![1](assets/1.png)
})
```

[![1](assets/1.png)](#co_building_a_visualization_CO12-1)

当主要的核心更新函数调用此回调函数时，国家数据由本地更新函数使用以更改条形图。

主要数据集由`getCountryData`方法生成，该方法通过国家将获奖者分组，并添加一些国家信息，即人口大小和国际字母代码。[示例 16-6](#build_getCountryData)详细介绍了此方法。

##### 示例 16-6。创建主要的国家数据集

```py
nbviz.getCountryData = function() {
    var countryGroups = nbviz.countryDim.group().all(); ![1](assets/1.png)

    // make main data-ball
    var data = countryGroups.map( function(c) { ![2](assets/2.png)
        var cData = nbviz.data.countryData[c.key]; ![3](assets/3.png)
        var value = c.value;
        // if per capita value then divide by pop. size
        if(nbviz.valuePerCapita){
            value = value / cData.population; ![4](assets/4.png)
        }
        return {
            key: c.key, // e.g., Japan
            value: value, // e.g., 19 (prizes)
            code: cData.alpha3Code, // e.g., JPN
        };
    })
        .sort(function(a, b) { ![5](assets/5.png)
            return b.value - a.value; // descending
        });

    return data;
};
```

[![1](assets/1.png)](#co_building_a_visualization_CO13-1)

`countryDim`是我们Crossfilter维度之一（见[“使用Crossfilter进行数据过滤”](#crossfilter)），在这里提供组键、值计数（例如，`{key:Argentina, value:5}`）。

[![2](assets/2.png)](#co_building_a_visualization_CO13-2)

我们使用数组的`map`方法来创建一个新数组，并从我们的国家数据集中添加组件。

[![3](assets/3.png)](#co_building_a_visualization_CO13-3)

使用我们的组键（例如，澳大利亚）获取国家数据。

[![4](assets/4.png)](#co_building_a_visualization_CO13-4)

如果`valuePerCapita`单选开关打开，则我们将奖项数量除以该国家的人口数量，从而得到一个*更公平*的相对奖项计数。

[![5](assets/5.png)](#co_building_a_visualization_CO13-5)

使用`Array`的`sort`方法使数组按值降序排列。

我们的Nobel-viz元素的更新方法都利用Crossfilter库过滤的数据。现在让我们看看如何做到这一点。

## 使用Crossfilter进行数据过滤

Crossfilter是由D3的创作者Mike Bostock和Jason Davies开发的一个高度优化的库，用于使用JavaScript探索大型、多变量数据集。它非常快速，并且可以轻松处理远比我们的诺贝尔奖数据集大得多的数据集。我们将使用它来根据类别、性别和国家的维度来过滤我们的获奖者数据集。

选择Crossfilter有些雄心勃勃，但我想展示它的实际效果，因为我个人发现它非常有用。它也是[*dc.js*](https://dc-js.github.io/dc.js)，这个非常流行的D3图表库的基础，这证明了它的实用性。虽然Crossfilter在开始交叉维度过滤时可能有些难以理解，但大多数用例遵循一个基本模式，很快就能掌握。如果你发现自己试图切割和分析大型数据集，Crossfilter的优化将会是一大帮助。

### 创建过滤器

在初始化 Nobel-viz 时，*nbviz_core.js* 中定义的 `makeFilterAndDimensions` 方法被从 *nbviz_main.js* 中的 `ready` 方法中调用（参见[“Ready to Go”](#sect_ready)）。`makeFilterAndDimensions` 使用刚刚加载的诺贝尔奖数据集创建一个 Crossfilter 过滤器和一些基于它的维度（例如，奖项类别）。

我们首先使用初始化时获取的诺贝尔奖获得者数据集创建我们的过滤器。让我们再次看看那是什么样子：

```py
[{
  name:"C\u00e9sar Milstein",
  category:"Physiology or Medicine",
  gender:"male",
  country:"Argentina",
  year: 1984
 },
 {
  name:"Auguste Beernaert",
  category:"Peace",
  gender:"male",
  country:"Belgium",
  year: 1909
 },
 ...
}];
```

要创建我们的过滤器，请使用获奖者对象的数组调用 `crossfilter` 函数：

```py
nbviz.makeFilterAndDimensions = function(winnersData){
    // ADD OUR FILTER AND CREATE CATEGORY DIMENSIONS
    nbviz.filter = crossfilter(winnersData);
    //...
};
```

Crossfilter 的工作原理是允许您在数据上创建维度过滤器。您可以通过将函数应用于对象来这样做。在最简单的情况下，这将创建一个基于单一类别的维度，例如按性别划分。在这里，我们创建了将用于过滤诺贝尔奖的性别维度：

```py
nbviz.makeFilterAndDimensions = function(winnersData){
//...
    nbviz.genderDim = nbviz.filter.dimension(function(o) {
        return o.gender;
    });
//...
}
```

这个维度现在通过性别字段对我们的数据集进行了高效的排序。我们可以像这样使用它，来返回所有性别为女性的对象：

```py
nbviz.genderDim.filter('female'); ![1](assets/1.png)
var femaleWinners = nbviz.genderDim.top(Infinity); ![2](assets/2.png)
femaleWinners.length // 47
```

[![1](assets/1.png)](#co_building_a_visualization_CO14-1)

`filter` 接受一个单一值或者在适当情况下，一个范围（例如，[5, 21]—所有在5和21之间的值）。它也可以接受值的布尔函数。

[![2](assets/2.png)](#co_building_a_visualization_CO14-2)

一旦应用了过滤器，`top` 就会返回指定数量的排序对象。指定 `Infinity`^([9](ch16.xhtml#idm45607748487088)) 将返回所有过滤后的数据对象。

当我们开始应用多维度过滤器时，Crossfilter 真正发挥了作用，使我们能够将数据切片并分割成我们需要的任何子集，所有这些都以令人印象深刻的速度实现。^([10](ch16.xhtml#idm45607748484816))

让我们清除性别维度并添加一个新的维度，通过获奖类别进行过滤。要重置维度，^([11](ch16.xhtml#idm45607748454160)) 不带参数地应用 `filter` 方法：

```py
nbviz.genderDim.filter();
nbviz.genderDim.top(Infinity) //  the full Array[858] of objects
```

我们现在将创建一个新的奖项类别维度：

```py
nbviz.categoryDim = nbviz.filter.dimension(function(o) {
    return o.category;
});
```

现在我们可以按顺序过滤性别和类别维度，从而找到例如所有女性物理学奖获得者：

```py
nbviz.genderDim.filter('female');
nbviz.categoryDim.filter('Physics');
nbviz.genderDim.top(Infinity);
// Out:
// [
//  {name:"Marie Sklodowska-Curie", category:"Physics",...
//  {name:"Maria Goeppert-Mayer", category:"Physics",...
// ]
```

请注意，我们可以有选择地打开和关闭过滤器。因此，例如，我们可以移除物理类别过滤器，这意味着性别维度现在包含所有女性诺贝尔奖获得者：

```py
nbviz.categoryDim.filter();
nbviz.genderDim.top(Infinity); // Array[47] of objects
```

在我们的诺贝尔可视化中，这些过滤操作将由用户从最顶部的菜单栏进行选择驱动。

除了返回过滤后的子集，Crossfilter 还可以对数据执行分组操作。我们使用这个来获取柱状图和地图指示器的国家奖聚合数据：

```py
nbviz.genderDim.filter(); // reset gender dimension var countryGroup = nbviz.countryDim.group(); ![1](assets/1.png)
countryGroup.all(); ![2](assets/2.png)

// Out: // [ //  {key:"Argentina", value:5}, ![3](assets/3.png)
//  {key:"Australia", value:9}, //  {key:"Austria", value:14}, // ...]
```

[![1](assets/1.png)](#co_building_a_visualization_CO15-1)

Group 接受一个可选的函数作为参数，但默认通常是您想要的。

[![2](assets/2.png)](#co_building_a_visualization_CO15-2)

返回所有按键和值分组的组。不要修改返回的数组。^([12](ch16.xhtml#idm45607748238000))

[![3](assets/3.png)](#co_building_a_visualization_CO15-3)

`value` 是阿根廷诺贝尔奖获得者的总数。

要创建我们的 Crossfilter 过滤器和维度，我们使用在 *nbviz_core.js* 中定义的 `makeFilterAndDimensions` 方法。[Example 16-7](#makeFilterAndDims) 显示了整个方法。请注意，创建过滤器的顺序并不重要——它们的交集仍然相同。

##### Example 16-7\. 制作我们的 Crossfilter 过滤器和维度

```py
    nbviz.makeFilterAndDimensions = function(winnersData){
        // ADD OUR FILTER AND CREATE CATEGORY DIMENSIONS
        nbviz.filter = crossfilter(winnersData);
        nbviz.countryDim = nbviz.filter.dimension(function(o){ ![1](assets/1.png)
            return o.country;
        });

        nbviz.categoryDim = nbviz.filter.dimension(function(o) {
            return o.category;
        });

        nbviz.genderDim = nbviz.filter.dimension(function(o) {
            return o.gender;
        });
    };
```

[![1](assets/1.png)](#co_building_a_visualization_CO16-1)

我们使用完整的 JavaScript 函数来进行教学，但现在可能会使用缩短的形式：`o => o.country`。

# 运行诺贝尔奖可视化应用程序

要运行诺贝尔奖可视化，我们需要一个能够访问根 *index.xhtml* 文件的 Web 服务器。为了开发目的，我们可以利用 Python 内置的 `http` 模块来启动所需的服务器。在包含我们的索引文件的根目录中，运行：

```py
$ python -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 ...
```

现在打开浏览器窗口，转到 *http:localhost:8080*，您应该会看到[图 16-4](#d3build_nobelviz)。

![dpj2 1604](assets/dpj2_1604.png)

###### 图 16-4\. 完成的 Nobel-viz 应用程序

# 总结

在本章中，我们概述了如何在[第 15 章](ch15.xhtml#chapter_imagining)中想象的可视化中实现。主干由 HTML、CSS 和 JavaScript 构建块组装而成，并描述了应用程序中的数据馈送和数据流。在接下来的章节中，我们将看到我们的 Nobel-viz 的各个组件如何使用发送到它们的数据来创建我们的交互式可视化。我们将从一个大的章节开始，介绍 D3 的基础知识，同时展示如何构建我们应用程序的条形图组件。这应该为你后续的 D3 焦点章节做好准备。

^([1](ch16.xhtml#idm45607751186112-marker)) 请查看这些 [SpeedCurve](https://oreil.ly/ngdOJ) 和 [Web Almanac](https://oreil.ly/qIvox) 的帖子，了解平均网页大小的一些分析。

^([2](ch16.xhtml#idm45607751164464-marker)) 我建议将 JavaScript 样式保存给特殊场合，尽可能地使用纯 CSS。

^([3](ch16.xhtml#idm45607750526128-marker)) 目前约为 2,560×1,600 像素。

^([4](ch16.xhtml#idm45607750213296-marker)) 请参阅[“投影”](ch19.xhtml#sect_projections)以比较不同的几何投影。考虑到展示所有获得诺贝尔奖的国家的约束条件，等经纬投影效果最好。

^([5](ch16.xhtml#idm45607749985552-marker)) 请查看[D3 的 GitHub 上的集合](https://oreil.ly/khvac)。

^([6](ch16.xhtml#idm45607749683584-marker)) 在我们的应用程序中，我尽可能地保持简单；随着 UI 选项数量的增加，将标志、范围等存储在专用对象中是明智的选择。

^([7](ch16.xhtml#idm45607749511824-marker)) 您可以在[Mozilla 文档](https://oreil.ly/67Odo)中了解更多关于 `Promise.all` 的信息。

^([8](ch16.xhtml#idm45607748719440-marker)) 请参阅这个[Square 页面](https://square.github.io/crossfilter)，这是一个令人印象深刻的示例。

^([9](ch16.xhtml#idm45607748487088-marker)) JavaScript 的 [`Infinity`](https://oreil.ly/Ll5xV) 是表示无穷大的数值。

^([10](ch16.xhtml#idm45607748484816-marker)) Crossfilter 被设计用于实时更新数百万条记录，以响应用户输入。

^([11](ch16.xhtml#idm45607748454160-marker)) 这将清除此维度上的所有过滤器。

^([12](ch16.xhtml#idm45607748238000-marker)) 请参阅 [Crossfilter GitHub 页面](https://oreil.ly/saEpG)。
