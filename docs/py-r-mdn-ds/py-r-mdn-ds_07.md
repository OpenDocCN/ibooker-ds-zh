# 第4章。数据格式背景

Boyan Angelov

在这一章中，我们将审查Python和R中用于导入和处理多种格式数据的工具。我们将涵盖一系列包，进行比较和对比，并突出它们的有效性。在这次旅程结束时，您将能够自信地选择包。每个部分通过一个特定的小案例研究展示工具的能力，这些案例基于数据科学家每天遇到的任务。如果您正在将工作从一种语言转换到另一种，或者只是想快速了解如何使用完整、维护良好和特定背景的包，本章将为您指引。

在我们深入讨论之前，请记住开源生态系统是不断变化的。新的发展，例如[转换器模型](https://www.tensorflow.org/tutorials/text/transformer)和[xAI](https://robotics.sciencemag.org/content/4/37/eaay7120)，似乎每隔一周就会出现。这些通常旨在降低学习曲线并增加开发者的生产力。这种多样性的爆炸也适用于相关的包，导致几乎不断涌现新的（希望是）更好的工具。如果您有非常具体的问题，可能已经有适合您的包，因此您不必重新发明轮子。工具选择可能会令人不知所措，但同时这种多样性的选择可以提高数据科学工作的质量和速度。

本章中的包选择在视野上可能显得有限，因此澄清我们的选择标准是至关重要的。那么，一个好的工具应该具备哪些特质？

+   它应该**开源**：有大量有价值的商业工具可用，但我们坚信开源工具具有巨大的优势。它们往往更容易扩展和理解其内部工作原理，并且更受欢迎。

+   它应该**功能完备**：该包应包括一套全面的函数，帮助用户在不依赖其他工具的情况下完成他们的基础工作。

+   它应该**维护良好**：使用开源软件（OSS）的一个缺点是，有时包的生命周期很短，它们的维护被放弃（所谓的“废弃软件”）。我们希望使用那些正在积极开发的包，这样我们就可以确信它们是最新的。

让我们从一个定义开始。什么是“数据格式”？[有几种答案](https://en.wikipedia.org/wiki/Data_format)可供选择。可能的候选者包括*数据类型*、*录音格式*和*文件格式*。*数据类型*与存储在数据库中的数据或编程语言中的类型有关（例如整数、浮点数或字符串）。*录音格式*是数据如何存储在物理介质上，例如CD或DVD。最后，我们关注的是*文件格式*，即信息如何为计算目的*准备*。

有了这个定义，人们可能会想，为什么我们要专门用一整章节来讨论文件格式呢？也许你在其他情境下已经接触过它们，比如用`.ppt`或`.pptx`扩展名保存 PowerPoint 幻灯片（并且想知道哪个更好）。然而，问题远不止于基本工具的兼容性。信息存储的方式会影响整个下游数据科学流程。例如，如果我们的最终目标是进行高级分析，而信息以文本格式存储，我们必须关注诸如字符编码等因素（这是一个臭名昭著的问题，尤其是对于 Python^([1](ch04.xhtml#idm45127452048648))）。为了有效处理这些数据，还需要经历几个步骤^([2](ch04.xhtml#idm45127452046808))，比如[tokenization](https://en.wikipedia.org/wiki/Tokenization)和[stop word](https://en.wikipedia.org/wiki/Stop_word)移除。然而，这些步骤对于图像数据则不适用，即使我们的最终目标可能是相同的，比如分类。在这种情况下，更适合的是其他处理技术，比如调整大小和缩放。这些数据处理管道的差异展示在[pipelines_diff](#pipelines_diff)上。总结一下：数据格式是影响你能够做什么、不能做什么的最重要因素。

###### 注意

现在我们在这个上下文中首次使用“管道”这个词，让我们利用这个机会来定义它。你可能听说过“数据是新的石油”的表达。这个表达不仅仅是一种简单的营销策略，而是一种有用的思考数据的方式。在数据和石油处理之间存在惊人的许多类似之处。你可以想象，企业收集的初始数据是最原始的形式，最初可能有限的用途。然后，在它被用于某些应用程序之前，它经历了一系列称为数据处理的步骤（例如用于训练 ML 模型或供给仪表盘）。在石油处理中，这被称为精炼和丰富化 - 使数据对业务目的可用。管道将不同类型的石油（原油、精炼油）通过系统传输到最终状态。在数据科学和工程中，同样的术语可以用来描述处理和交付数据所需的基础设施和技术。

1.  常见数据格式管道之间的差异。绿色表示工作流程之间的共享步骤。image::img/pipelines_diff.jpg[""]

在处理特定数据格式时，还需要考虑基础设施和性能。例如，对于图像数据，您需要更多的存储空间。对于时间序列数据，您可能需要使用特定的数据库，例如[Influx DB](https://www.influxdata.com/products/influxdb-overview/)。最后，在性能方面，图像分类通常使用基于卷积神经网络（CNN）的深度学习方法来解决，这可能需要图形处理单元（GPU）。如果没有，模型训练可能会非常缓慢，并成为开发工作和潜在生产部署的瓶颈。

现在我们已经介绍了仔细考虑使用哪些包的原因，让我们来看看可能的数据格式。这个概述在[表 4-1](#data-format-zoo-table)中展示（请注意，这些工具主要设计用于小到中等规模的数据集）。诚然，我们只是浅尝辄止，还有一些显著的遗漏（比如音频和视频）。在这里，我们将专注于最常用的格式。

表 4-1\. 数据格式及用于处理其中存储数据的流行 Python 和 R 包概览。

| 数据类型 | Python 包 | R 包 |
| --- | --- | --- |
| 表格 | `pandas` | `readr`，`rio` |
| 图像 | `open-cv`，`scikit-image`，`PIL` | `magickr`，`imager`，`EBImage` |
| 文本 | `nltk`，`spaCy` | `tidytext`，`stringr` |
| 时间序列 | `prophet`，`sktime` | `prophet`，`ts`，`zoo` |
| 空间 | `gdal`，`geopandas`，`pysal` | `rgdal`，`sp`，`sf`，`raster` |

这张表远非详尽无遗，我们确信新工具将很快出现，但这些工具是实现我们选择标准的工作马，让我们在接下来的章节中看看它们的表现，看看哪些是最适合这项工作的！

# 外部与基础包

在[第二章](ch02.xhtml#ch03)和[第三章](ch03.xhtml#ch04)中，我们在学习过程的早期就介绍了包。在 Python 中，我们在一开始就使用了`pandas`，而在 R 中，我们也相对迅速地过渡到了`tidyverse`。这使我们比如果深入探讨那些初学者不太可能需要的过时语言特性，更快地提高了生产力^([3](ch04.xhtml#idm45127452007560))。一个编程语言的实用性取决于其第三方包的可用性和质量，而不是语言本身的核心特性。

###### 注

这并不意味着仅仅使用基础 R 就可以完成很多事情（正如您将在即将看到的一些示例中所见），但利用开源生态系统是提高您生产力的基本技能，避免重复造轮子。

# 回头学习基础知识

过度使用第三方包存在风险，你必须知道何时是回归基础的正确时机。否则，你可能会陷入虚假的安全感，并依赖于像`pandas`这样的工具提供的支持。这可能导致在处理更具体的现实世界挑战时遇到困难。

现在让我们通过详细讨论一个我们已经熟悉的主题，来看看这个包与基础语言概念在实践中是如何发挥作用的：表格数据^([4](ch04.xhtml#idm45127452002248))。在Python中至少有两种方法可以做到这一点。首先是使用`pandas`：

```py
import pandas as pd

data = pd.read_csv(“dataset.csv”)
```

其次是使用内置的`csv`模块：

```py
import csv

with open(“dataset.csv”, “r”) as file: ![1](Images/1.png)
	reader = csv.reader(file, delimiter=“,”)
for row in reader: ![2](Images/2.png)
	print(row)
```

[![1](Images/1.png)](#co_data_format_context_CO1-1)

注意你需要指定[file mode](https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files)，在本例中是`"r"`（表示“读取”）。这是为了确保文件不会意外被覆盖，这暗示了一种更面向通用目的的语言。

[![2](Images/2.png)](#co_data_format_context_CO1-2)

对于初学者来说，使用循环读取文件可能看起来很奇怪，但这使过程更加明确。

这个例子告诉我们，`pandas`中的`pd.read_csv()`提供了一种更简洁、方便和直观的导入数据方式。它比纯粹的Python少了明确，也不是必需的。`pd.read_csv()`本质上是现有功能的*便利封装* —— 对我们很有帮助！

在这里，我们看到包有两个功能。首先，正如我们所期望的那样，它们提供*新*的功能。其次，它们还是现有标准功能的便利封装，使我们的生活更轻松。

这在R的`rio`包中得到了很好的展示^([5](ch04.xhtml#idm45127451851368))。`rio`代表“R input/output”，它确实如其名称所示^([6](ch04.xhtml#idm45127451849368))。在这里，单个函数`import()`使用文件的文件名扩展名来选择在一系列包中导入的最佳函数。这适用于Excel、SPSS、stata、SAS或许多其他常见格式。

另一个R tidyverse包`vroom`允许快速导入表格数据，并可以在一个命令中读取整个目录的文件，使用`map()`函数或`for loops`。

最后，经常被忽视以推广tidyverse为代价的`data.table`包提供了出色的`fread()`，它可以以远低于基础R或`readr`提供的速度导入非常大的文件。

学习如何使用第三方包的实用性在我们尝试执行更复杂的任务时变得更加明显，正如我们下面在处理其他数据格式时将会看到。

现在我们可以欣赏包的优势，我们将展示它们的一些功能。为此，我们将处理几个不同的真实用例，列在 [表 4-2](#case-study-table) 中。我们不会专注于细节实现，而是覆盖那些暴露它们在任务中优点（和缺点）的元素。由于本章重点是数据格式，而 [第五章](ch05.xhtml#ch06) 则全面介绍工作流程，所有这些案例研究都涉及数据处理（如前面在 [???](#pipelines_diff) 中所示）。 

###### 注意

为了教学目的，我们省略了部分代码。如果您想跟着做，请查看 [书籍存储库](https://github.com/moderndatadesign/PyR4MDS) 中的可执行代码。

表 4-2\. 不同用例的概述

| 数据格式 | 使用案例 |
| --- | --- |
| 图像 | [游泳池和汽车检测](https://www.kaggle.com/kbhartiya83/swimming-pool-and-car-detection) |
| 文本 | [Amazon 音乐评论处理](http://jmcauley.ucsd.edu/data/amazon/) |
| 时间序列 | [澳大利亚日温度](https://raw.githubusercontent.com/jbrownlee/Datasets/master/daily-min-temperatures.csv) |
| 空间 | [*Loxodonta africana* 物种分布数据](https://www.gbif.org/) |

更多关于如何下载和处理这些数据的信息可以在官方 [存储库](https://github.com/moderndatadesign/PyR4MDS) 中找到。

# 图像数据

对数据科学家来说，图像提出了一系列独特的挑战。我们将通过处理航空图像处理的挑战来演示最佳方法，这是农业、生物多样性保护、城市规划和气候变化研究日益重要的领域。我们的迷你用例使用了来自 Kaggle 的数据，旨在帮助检测游泳池和汽车。有关数据集的更多信息，请使用 [表 4-2](#case-study-table) 中的网址。

## OpenCV 和 scikit-image

正如我们在本章开头提到的，下游目的极大地影响数据处理。由于航空数据经常用于训练机器学习算法，我们的重点将放在准备性任务上。

[OpenCV](https://opencv.org/) 包是在 Python 中处理图像数据的最常见方法之一。它包含了加载、操作和存储图像所需的所有工具。名称中的 “CV” 代表计算机视觉 - 这是专注于图像数据的机器学习领域。我们还将使用的另一个便捷工具是 `scikit-image`。正如其名称所示，它与 [scikit-learn](https://scikit-learn.org/stable/)^([7](ch04.xhtml#idm45127451817912)) 密切相关。

这是我们任务的步骤（参见 [表 4-2](#case-study-table)）：

1.  调整图像大小到特定尺寸

1.  将图像转换为黑白

1.  通过旋转图像增强数据

要使ML算法成功地从数据中学习，必须对输入进行清洗（数据整理），标准化（即缩放）和过滤（特征工程）^([8](ch04.xhtml#idm45127451810536))。你可以想象收集一组图像数据集（例如从Google图像中爬取^([9](ch04.xhtml#idm45127451809800))数据）。它们可能在大小和/或颜色等方面有所不同。我们任务列表中的步骤1和2帮助我们处理这些差异。第3步对ML应用非常有用。ML算法的性能（例如分类准确度或曲线下面积（AUC））主要取决于训练数据的数量，通常供应量有限。为了解决这个问题，而不是获取更多数据^([10](ch04.xhtml#idm45127451808600))，数据科学家发现，对已有数据进行操作，如旋转和裁剪，可以引入新的数据点。然后可以再次用这些数据点来训练模型并提高性能。这个过程正式称为数据增强^([11](ch04.xhtml#idm45127451807656))。

足够的讨论 - 让我们从导入数据开始吧！如果你想跟着做，请检查本书的[代码仓库](https://github.com/moderndatadesign/PyR4MDS)。

```py
import cv2 ![1](Images/1.png)
single_image = cv2.imread("img_01.jpg")

plt.imshow(single_image)
plt.show()
```

[![1](Images/1.png)](#co_data_format_context_CO2-1)

使用`cv2`可能看起来令人困惑，因为包名为`OpenCV`。`cv2`被用作缩写名称。与`scikit-image`相同的命名模式，在导入语句中缩短为`skimage`。

![](Images/prds_0401.png)

###### 图4-1\. Python中使用`matplotlib`绘制的原始图像。

那么`cv2`将数据存储在哪种对象类型中？我们可以用`type`来检查：

```py
print(type(single_image))
numpy.ndarray
```

在这里，我们可以观察到一个重要的特性，它已经为使用Python进行CV任务相对于R提供了优势。图像直接存储为`numpy`多维数组（`nd`代表n维），使其可以访问Python生态系统中的各种其他工具。因为这是建立在`pyData`堆栈上的，所以得到了良好的支持。这对于R也适用吗？让我们看看`magick`包：

```py
library(magick)
single_image <- image_read('img_01.jpg')
class(single_image)
[1] "magick-image"
```

`magick-image`类仅限于来自`magick`包或其他密切相关工具的函数使用，而不是强大的基础R方法（例如第2章中展示的方法，特别是`plot()`除外）。各种开源包之间如何支持彼此的不同方法在本章的示例中有所体现，如图4-2所示，是一个共同的主题。

###### 注意

至少有一个例外情况符合这一规则 - `EBImage`包，它是[Bioconductor](https://bioconductor.org/)的一部分。通过使用它，您可以以原始数组形式访问图像，然后在此基础上使用其他工具。这里的缺点是它是一个特定领域的包的一部分，可能不容易在标准CV流水线中看到它是如何工作的。

![](Images/prds_0402.png)

###### 图4-2\. 两种包设计层次结构，在数据生命周期中使用（从底部到顶部）。左侧模式显示了一个次优结构，用户被迫在第一级别使用特定目的的工具，这限制了他们的灵活性和生产力。右侧模式显示了一个更好的结构，在数据传递的初始阶段有标准工具，从而在下游能够使用多种工具。

注意，在前一步骤中（加载Python中的原始图像时），我们还使用了最流行的绘图工具之一 - `matplotlib`（数据可视化在[第5章](ch05.xhtml#ch06)中有所涉及），因此我们再次利用了这种更好的设计模式。

现在我们知道图像数据存储为`numpy`的`ndarray`，我们可以使用`numpy`的方法。图像的尺寸是多少？我们可以尝试`ndarray`的`.shape`方法来获取：

```py
print(single_image.shape)
224 224 3
```

确实有效！前两个输出值分别对应于图像的`height`和`width`，第三个则是图像中的通道数 - 在这种情况下是三个 ((r)ed, (g)reen 和 (b)lue)。现在让我们继续并完成第一步标准化 - 图像调整大小。在这里我们将第一次使用`cv2`：

```py
single_image = cv2.resize(single_image,(150, 150))
print(single_image.shape)
(150, 150, 3)
```

###### 注意

如果你在两种语言中都有使用这些基础工具的经验，你将能够快速测试你的想法，即使不知道这些方法是否存在。如果你使用的工具设计良好（如[图4-2](#package_design)中的更好设计），通常它们会按预期工作！

完美，它的运行效果如同魔术一般！接下来的步骤是将图像转换为黑白。为此，我们同样会使用`cv2`：

```py
gray_image = cv2.cvtColor(single_image, cv2.COLOR_RGB2GRAY)
print(gray_image.shape)
(150, 150)
```

颜色呈绿色调，而非灰色。默认选项选择了一种颜色方案，使得对人眼的对比更容易辨别，比黑白色更加明显。当你观察`numpy`的`ndarray`形状时，你会发现通道数已经消失了 - 现在只剩下一个。现在让我们完成我们的任务，并进行简单的数据增强步骤，将图像水平翻转。在这里，我们再次利用数据存储为`numpy`数组的优势。我们将直接使用`numpy`中的一个函数，而不依赖于其他CV库（如`OpenCV`或`scikit-image`）：

```py
flipped_image = np.fliplr(gray_image)
```

结果显示在[图4-3](#flipped_image)中。

![](Images/prds_0403.png)

###### 图4-3\. 使用`numpy`函数翻转图像的绘图。

我们可以使用`scikit-image`进行更多的图像操作任务，比如旋转，即使使用这种不同的包也能按预期在我们的数据格式上工作：

```py
from skimage import transform
rotated_image = transform.rotate(single_image, angle=45)
```

我们经历的数据标准化和增强步骤说明了较简单的包设计（[图4-2](#package_design)）如何提高我们的生产力。我们可以通过展示第三步的负面例子来进一步强调这一点，这次是在R语言中。为此，我们将依赖于`adimpro`包：

```py
library(adimpro)
rotate.image(single_image, angle = 90, compress=NULL)
```

每当我们加载另一个包时，都会降低我们代码的质量、可读性和可重用性。这个问题主要是由于可能存在未知的bug、更陡的学习曲线或第三方包缺乏一致和详尽的文档。通过快速检查[CRAN上的`adimpro`](https://cran.r-project.org/web/packages/adimpro/index.html)，可以发现它最后一次更新是在2019年11月^([12](ch04.xhtml#idm45127451457144))。这就是为什么更倾向于使用像`OpenCV`这样的工具，它利用`PyData`堆栈^([13](ch04.xhtml#idm45127451455800))处理图像数据，如`numpy`。

一个较少复杂、模块化且足够抽象的包设计可以大大提高数据科学家在使用工具时的生产力和满意度。他们可以自由专注于实际工作，而不是处理复杂的文档或大量弃用的包。这些考虑使得Python在导入和处理图像数据方面成为明显的优选，但对其他格式是否也是如此呢？

# 文本数据

文本数据分析通常与自然语言处理（NLP）术语交替使用。这又是ML的一个子领域。因此，Python工具的主导地位并不奇怪。处理文本数据本质上需要大量计算资源，这也是为什么Python工具主导的一个很好的原因。另一个原因是，在R中处理更大的数据集可能比在Python中更具挑战性^([14](ch04.xhtml#idm45127451424392))（关于此主题的更多内容，请参阅[第5章](ch05.xhtml#ch06)）。这是一个大数据问题。随着互联网服务和Twitter、Facebook等社交媒体巨头的兴起，文本数据量近年来激增。这些组织也大力投资于相关技术和开源，因为他们可获取的大部分数据都是文本格式。

与图像数据类似，我们将设计一个标准的NLP任务。它应包含NLP流水线的最基本元素。作为数据集，我们选择了来自亚马逊产品评论数据集的文本（见[表4-2](#case-study-table)），并需准备用于文本分类、情感分析或主题建模等高级分析用例。完成所需的步骤如下：

1.  对数据进行分词

1.  去除停用词

1.  标记词性（PoS）

我们还将通过`spaCy`介绍更高级的方法（如词嵌入），以展示Python包的功能，并同时提供几个R示例以进行比较。

## NLTK和spaCy

那么 Python 中最常用的工具是什么？最流行的工具之一通常被称为 NLP 的瑞士军刀 - 自然语言工具包（NLTK）^([15](ch04.xhtml#idm45127451414392))。它包含了涵盖整个流程的丰富工具集。它还具有出色的文档和相对较低的 API 学习曲线。

# NLTK 书籍

NLTK 的作者们还写了一本关于文本数据处理最易于理解的书籍 - NLTK 书籍，当前版本为3\. 它可以免费在线阅读，网址在[官方网站](https://www.nltk.org/book/)。这可以作为一个很好的参考手册，如果你想深入了解我们在本节中涵盖的一些主题，请随时查阅！

作为一名数据科学家，在项目中的第一步之一是查看原始数据。这里是一个示例评论及其数据类型：

```py
example_review = reviews["reviewText"].sample()
print(example_review)
print(type(example_review))
```

```py
I just recently purchased her ''Paint The Sky With Stars''
 CD and was so impressed that I bought 3 of her previously
 released CD's and plan to buy all her music.  She is
 truely talented and her music is very unique with a
 combination of modern classical and pop with a hint of
 an Angelic tone. I still recommend you buy this CD. Anybody
  who has an appreciation for music will certainly enjoy her music.

str
```

这里很重要 - 数据存储在 Python 中的一个基本数据类型中 - `str`（字符串）。类似于将图像数据存储为多维`numpy`数组，许多其他工具也可以访问它。例如，假设我们要使用一种能够高效搜索和替换字符串部分的工具，比如[flashtext](https://github.com/vi3k6i5/flashtext)。在这种情况下，我们可以在这里使用它而不会出现格式问题，也无需强制转换^([16](ch04.xhtml#idm45127451365400))数据类型。

现在我们可以在我们的迷你案例研究中迈出第一步 - *分词*。它将评论分割成词或句子等组件：

```py
sentences = nltk.sent_tokenize(example_review)
print(sentences)
```

```py
["I just recently purchased her ''Paint The Sky With Stars''
CD and was so impressed that I bought 3 of her
previously released CD's and plan to buy all her music.",
 'She is truely talented and her music is very unique with
  a combination of modern classical and pop with a hint of an Angelic tone.',
 'I still recommend you buy this CD.',
 'Anybody who has an appreciation for music will certainly enjoy her music.']
```

足够简单！为了举例说明，在 R 中使用一些来自`tidytext`的函数尝试这个相对简单的任务会有多困难？

```py
tidy_reviews <- amazon_reviews %>%
  unnest_tokens(word, reviewText) %>%
  mutate(word = lemmatize_strings(word, dictionary = lexicon::hash_lemmas))
```

这是使用的最详细记录方法之一。其中一个问题是它过于依赖“整洁数据”概念，以及`dplyr`中的管道连接概念（我们在[第2章](ch02.xhtml#ch03)中涵盖了这两个概念）。这些概念特定于 R 语言，如果要成功使用`tidytext`，首先必须学习它们，而不能直接开始处理数据。第二个问题是此过程的输出 - 包含处理列中数据的新`data.frame`。虽然这可能是最终所需的内容，但它跳过了一些中间步骤，比我们使用`nltk`时的抽象层级要高几层。降低这种抽象并以更模块化的方式工作（例如首先处理单个文本字段）符合软件开发的最佳实践，如 DRY（“不要重复自己”）和关注点分离。

我们小型 NLP 数据处理流水线的第二步是移除停用词^([17](ch04.xhtml#idm45127451276088))。

```py
tidy_reviews <- tidy_reviews %>%
  anti_join(stop_words)
```

这段代码存在与之前相同的问题，还有一个新的令人困惑的函数 - `anti_join`。让我们来比较一下简单的列表推导（更多信息见[第3章](ch03.xhtml#ch04)中的`nltk`步骤）：

```py
english_stop_words = set(stopwords.words("english"))
cleaned_words = [word for word in words if word not in english_stop_words]
```

`english_stop_words`只是一个`list`，然后我们只需循环遍历另一个`list`（`words`）中的每个单词，并删除*如果*它同时存在于两者中。这更易于理解。没有依赖于与直接相关的高级概念或函数。这段代码也处于正确的抽象级别。小代码块可以更灵活地作为更大文本处理管道函数的一部分使用。在R中，类似的“元”处理函数可能会变得臃肿 - 执行缓慢且难以阅读。

虽然`nltk`可以执行这些基本任务，但现在我们将看一看更高级的包 - `spaCy`。在我们的案例研究的第三和最后一步中，我们将使用它进行词性标注（Part of Speech，PoS）^([18](ch04.xhtml#idm45127451178120))。

```py
import spacy

nlp = spacy.load("en_core_web_sm") ![1](Images/1.png)

doc = nlp(example_review) ![2](Images/2.png)
print(type(doc))
```

```py
spacy.tokens.doc.Doc
```

[![1](Images/1.png)](#co_data_format_context_CO3-1)

在这里，我们通过一个函数加载所有我们需要的高级功能。

[![2](Images/2.png)](#co_data_format_context_CO3-2)

我们取一条示例评论并将其提供给`spaCy`模型，结果是`spacy.tokens.doc.Doc`类型，而不是`str`。然后可以对该对象进行各种其他操作：

```py
for token in doc:
    print(token.text, token.pos_)
```

数据在加载时已经进行了标记。不仅如此 - 所有的词性标注也已经完成了！

我们所涉及的数据处理步骤相对基础。那么，再来看看一些更新、更高级的自然语言处理方法呢？例如，我们可以考虑词嵌入。这是更高级的文本向量化方法之一，其中每个向量表示一个词在其上下文中的含义^([19](ch04.xhtml#idm45127451081640))。为此，我们可以直接使用来自`spaCy`代码的同一个`nlp`对象：

```py
for token in doc:
    print(token.text, token.has_vector, token.vector_norm, token.is_oov)
```

```py
for token in doc:...
I True 21.885008 True
just True 22.404057 True
recently True 23.668447 True
purchased True 23.86188 True
her True 21.763712 True
' True 18.825636 True
```

看到这些功能已经内置在其中一款最受欢迎的Python自然语言处理包中真是个惊喜。在这个自然语言处理方法的水平上，我们几乎看不到R（或其他语言）的任何替代方案。许多在R中类似的解决方案依赖于围绕Python后端的包装代码（这可能违背使用R语言的初衷）^([20](ch04.xhtml#idm45127451027384))。这种模式在书中经常见到，特别是在[第5章](ch05.xhtml#ch06)。对于一些其他高级方法，如变换器模型，也是如此^([21](ch04.xhtml#idm45127451025304))。

对于第二轮来说，Python 再次胜出。`nltk`、`spaCy`以及其他相关包的能力使其成为进行自然语言处理工作的优秀选择！

# 时间序列数据

时间序列格式用于存储具有时间维度的任何数据。它可以是简单的本地杂货店洗发水销售数据，带有时间戳，也可以是从农业领域湿度传感器网络中测量的数百万数据点。

###### 注意

R在时间序列数据分析中的统治地位也有一些例外情况。特别是，最近深度学习方法的发展，尤其是长短期记忆网络（LSTM），已被证明在时间序列预测方面非常成功。就像其他深度学习方法一样（详见[第五章](ch05.xhtml#ch06)），这是一个更好地由Python工具支持的领域。

## Base R

R用户可以使用相当多的不同包来分析时间序列数据，包括`xts`和`zoo`，但我们将以base R函数为起点。在此之后，我们将看一看一个更现代的包，以说明更高级的功能 - [Facebook的Prophet](https://facebook.github.io/prophet/)。

天气数据既易获取又相对容易解释，因此在我们的案例研究中，我们将分析澳大利亚的每日最低温度（详见[表4-2](#case-study-table)）。要进行时间序列分析，我们需要执行以下步骤：

1.  将数据加载到适当的格式中

1.  绘制数据

1.  去除噪声和季节性影响并提取趋势

然后我们就能进行更高级的分析了。假设我们已经将数据从`.csv`文件加载到R中的`data.frame`对象中。这里没有什么特别之处。然而，与大多数Python包不同，R需要将数据强制转换为特定的对象类型。在这种情况下，我们需要将`data.frame`转换为`ts`（代表时间序列）。

```py
df_ts <- ts(ts_data_raw$Temp, start=c(1981, 01, 01),
            end=c(1990, 12, 31), frequency=365)
class(df_ts)
```

那么，为什么我们要选择它而不是`pandas`呢？嗯，即使你成功地将原始数据转换为时间序列`pd.DataFrame`，你还会遇到一个新概念 - `DataFrame`索引（见[图4-4](#ts_index)）。要高效地进行数据操作，你需要先了解这是如何工作的！

![](Images/prds_0404.png)

###### 图4-4\. `pandas`中的时间序列索引。

这个索引概念可能会让人感到困惑，所以现在让我们看看R中的替代方法以及是否更好。使用`df_ts`时间序列对象，我们已经可以做一些有用的事情。当你在R中使用更高级的时间序列包时，这也是一个很好的起点，因为将`ts`对象转换为`xts`或`zoo`应该不会出错（这再次是我们在[图4-2](#package_design)中讨论的良好对象设计的一个例子）。你可以尝试做的第一件事是对对象进行`plot`，这在R中通常会产生良好的结果：

```py
plot(df_ts)
```

###### 注意

调用`plot`函数并不仅仅是使用一个标准函数，可以在R中绘制各种不同的对象（这是你所期望的）。它调用与数据对象关联的特定方法（关于函数和方法之间的区别的更多信息，请参阅[第二章](ch02.xhtml#ch03)）。这个简单函数调用背后隐藏了很多复杂性！

![](Images/prds_0405.png)

###### 图4-5\. 在base R中绘制时间序列（`ts`）对象。

在 [图 4-5](#ts_plot) 上的 `plot(df_ts)` 结果已经很有用。x 轴上的日期被识别，并选择了 `line` 绘图，而不是默认的 `points` 绘图。分析时间序列数据（以及大多数机器学习数据）中最普遍的问题是处理噪声。与其他数据格式的不同之处在于存在几种不同的噪声来源，可以清除不同的模式。这是通过一种称为分解的技术实现的，我们有内置且命名良好的函数 `decompose` 来完成这一点：

```py
decomposed_ts <- decompose(df_ts)
plot(decomposed_ts)
```

结果可以在 [图 4-6](#ts_decompose) 上看到。

![](Images/prds_0406.png)

###### 图 4-6\. 在基本 R 中绘制的分解时间序列图。

我们可以看出随机噪声和季节性以及总体模式是什么。我们只需在基本 R 中进行一次函数调用就实现了所有这些！在 Python 中，我们需要使用 `statsmodels` 包来达到同样的效果。

## prophet

对于分析时间序列数据，我们还有另一个令人兴奋的包例子。它同时为 R 和 Python 开发（类似于 `lime` 可解释的 ML 工具） - [Facebook Prophet](https://facebook.github.io/prophet/)。这个例子可以帮助我们比较 API 设计上的差异。`prophet` 是一个主要优势在于领域用户能够根据其特定需求进行调整，API 使用便捷且专注于生产就绪的包。这些因素使其成为原型化时间序列工作和在数据产品中使用的不错选择。让我们来看看；我们的数据存储为 `pandas` 的 `DataFrame` 在 `df` 中：

```py
from fbprophet import Prophet

m = Prophet()
m.fit(df) ![1](Images/1.png)

future = m.make_future_dataframe(periods=365) ![2](Images/2.png)
future.tail()
```

[![1](Images/1.png)](#co_data_format_context_CO4-1)

在这里，我们再次看到相同的 `fit` API 模式，借鉴自 `scikit-learn`。

[![2](Images/2.png)](#co_data_format_context_CO4-2)

这一步创建了一个新的空的 `Data.Frame`，以便稍后存储我们的预测结果。

```py
library(prophet)

m <- prophet(df)

future <- make_future_dataframe(m, periods = 365)
tail(future)
```

两者都足够简单，并包含相同数量的步骤 - 这是一致的 API 设计的一个很好的例子（在 [第 5 章](ch05.xhtml#ch06) 中详细讨论）。

###### 注意

提供跨语言一致的用户体验是一个有趣且有帮助的想法，但我们预计它不会被广泛实现。很少有组织拥有进行这种工作的资源，这可能限制选择软件设计的折衷。

此时，您可以感受到同时掌握两种语言将为您的日常工作带来重大优势。如果您只接触过 Python 包生态系统，您可能会尝试寻找类似的工具来分析时间序列，从而错失基本 R 和相关 R 包提供的令人难以置信的机会。

# 空间数据

空间数据分析是现代机器学习中最有前景的领域之一，并且具有丰富的历史。近年来已开发出新工具，但长期以来 R 一直占据优势，尽管 Python 近年来也有了一些进展。与之前的部分一样，我们将通过一个实际例子来看看这些包是如何运作的。

###### 注意

空间数据有几种格式可用。在本小节中，我们专注于对*raster*数据的分析。对于其他格式，在Python中有一些有趣的工具可用，比如[GeoPandas](https://geopandas.org/)，但这超出了本章的范围。

我们的任务是处理*Loxodonta africana*（非洲象）的出现数据^([22](ch04.xhtml#idm45127450735128))和环境数据，使其适用于空间预测。这种数据处理在物种分布建模（SDM）中很典型，其中预测用于构建用于保护的栖息地适宜性地图。这个案例研究比之前的更复杂，并且许多步骤中隐藏了一些复杂性，包装程序正在进行大量的操作。步骤如下：

1.  获取环境栅格数据

1.  裁剪栅格以适应感兴趣的区域

1.  使用抽样方法处理空间自相关

## 栅格

为了解决这个问题的第一步，我们需要处理栅格数据^([23](ch04.xhtml#idm45127450728616))。从某种意义上说，这与标准图像数据非常相似，但在处理步骤上仍然有所不同。对此，R语言有出色的`raster`包可用（另一种选择是Python的`gdal`和R的`rgdal`，但在我们看来更难使用）。

```py
library(raster)
climate_variables <- getData(name = "worldclim", var = "bio", res = 10)
```

`raster`允许我们下载大多数常见的有用的空间环境数据集，包括生物气候数据^([24](ch04.xhtml#idm45127450683720))。

```py
e <- extent(xmin, xmax, ymin, ymax)
coords_absence <- dismo::randomPoints(climate_variables, 10000, ext = e)
points_absence <- sp::SpatialPoints(coords_absence,
                                    proj4string = climate_variables@crs)
env_absence <- raster::extract(climate_variables, points_absence)
```

这里我们使用便捷的`extent`函数来裁剪（切割）栅格数据 - 我们只对围绕出现数据的所有环境层的一个子集感兴趣。在这里，我们使用经度和纬度坐标来绘制这个矩形。作为下一步，为了形成一个分类问题，我们从栅格数据中随机抽样数据点（这些被称为“伪缺失”）。你可以想象这些是我们分类任务中的`0`，而出现（观测）则是`1` - 目标变量。然后我们将伪缺失转换为`空间点`，最后也提取它们的气候数据。在`SpatialPoints`函数中，你还可以看到我们如何指定地理投影系统，这是分析空间数据时的基本概念之一。

在进行ML工作时，最常见的问题之一是数据内的相关性。正确数据集的基本假设是数据中的个体观察结果彼此*独立*，以获得准确的统计结果。由于其本质的原因，空间数据始终存在这个问题。这个问题被称为*空间自相关*。有几个可用于从数据中抽样以减轻这一风险的包。其中一个这样的包是`ENMeval`：

```py
library(ENMeval)
check1 <- get.checkerboard1(occs, envs, bg, aggregation.factor=5)
```

`get.checkerboard1`函数以均匀分布的方式对数据进行采样，类似于从黑白棋盘的每个方块中取等量的点。然后，我们可以使用这些重新采样的数据成功训练 ML 模型，而不必担心空间自相关。最后一步，我们可以取得这些预测并创建生境适宜性地图，显示在（[???](#sdm_map)）。

```py
raster_prediction <- predict(predictors, model)
plot(raster_prediction)
```

1.  在 R 中绘制`raster`对象预测，生成生境适宜性地图。image::img/sdm_map.png[""]

当您处理空间栅格数据时，R 提供了更好的包设计。像`raster`这样的基础工具为更高级的特定应用程序（如`ENMeval`和`dismo`）提供了一致的基础，而无需担心复杂的转换或易出错的类型强制转换。

# 最终思考

在本章中，我们详细介绍了不同的常见数据格式，以及处理它们准备进行高级任务的最佳包。在每个案例研究中，我们展示了一个良好的包设计如何使数据科学家更加高效。我们已经看到，对于更多以 ML 为重点的任务，如 CV 和 NLP，Python 提供了更好的用户体验和更低的学习曲线。相比之下，对于时间序列预测和空间分析，R 则占据了上风。这些选择在[图 4-7](#decision_tree)中显示。

![](Images/prds_0407.png)

###### 图 4-7\. 包选择的决策树。

最佳工具的共同之处在于更好的包设计（[图 4-2](#package_design)）。您应该始终使用最适合工作的工具，并注意您使用的工具的复杂性、文档和性能！

^([1](ch04.xhtml#idm45127452048648-marker)) 欲了解更详细的解释，请查看此处：[*https://realpython.com/python-encodings-guide/*](https://realpython.com/python-encodings-guide/)。

^([2](ch04.xhtml#idm45127452046808-marker)) 这通常被称为“数据血统”。

^([3](ch04.xhtml#idm45127452007560-marker)) 还有谁没有学会在 Python 中`if __name__ == "__main__"`是做什么的？

^([4](ch04.xhtml#idm45127452002248-marker)) 数据中的一个表，存储在一个文件中。

^([5](ch04.xhtml#idm45127451851368-marker)) 别忘了`tidyr`，它在[第二章](ch02.xhtml#ch03)中已经讨论过了。

^([6](ch04.xhtml#idm45127451849368-marker)) 我们确实提到统计学家非常字面吧？

^([7](ch04.xhtml#idm45127451817912-marker)) 这种一致性是第三部分的各章节中的共同主题，并在[第五章](ch05.xhtml#ch06)中另外讨论。

^([8](ch04.xhtml#idm45127451810536-marker)) 记住 - 输入垃圾，输出垃圾。

^([9](ch04.xhtml#idm45127451809800-marker)) 使用代码浏览网页内容，下载并以机器可读格式存储。

^([10](ch04.xhtml#idm45127451808600-marker)) 这在某些情况下可能是昂贵的，甚至是不可能的。

^([11](ch04.xhtml#idm45127451807656-marker)) 如果你想了解更多关于图像数据增强的信息，请查看[此教程](https://www.tensorflow.org/tutorials/images/data_augmentation)。

^([12](ch04.xhtml#idm45127451457144-marker)) 在撰写时。

^([13](ch04.xhtml#idm45127451455800-marker)) 不要与同名会议混淆，PyData堆栈指的是`NumPy`、`SciPy`、`Pandas`、`IPython`和`matplotlib`。

^([14](ch04.xhtml#idm45127451424392-marker)) R社区也响应了这一号召，并在最近改进了工具链，但与其Python对应部分相比仍有一定差距。

^([15](ch04.xhtml#idm45127451414392-marker)) 欲了解更多关于NLTK的信息，请查阅官方书籍，链接在[这里](https://www.nltk.org/book/)。

^([16](ch04.xhtml#idm45127451365400-marker)) 数据类型强制转换是将一个数据类型转换为另一个的过程。

^([17](ch04.xhtml#idm45127451276088-marker)) 这是自然语言处理中的常见步骤。一些停用词的例子包括“the”、“a”和“this”。这些词需要移除，因为它们很少提供有用的信息给机器学习算法。

^([18](ch04.xhtml#idm45127451178120-marker)) 将词语标记为它们所属的词性的过程。

^([19](ch04.xhtml#idm45127451081640-marker)) 将文本转换为数字，以便机器学习算法处理。

^([20](ch04.xhtml#idm45127451027384-marker)) 例如尝试创建自定义嵌入。更多信息请查阅RStudio博客，链接在[这里](https://blogs.rstudio.com/ai/posts/2017-12-22-word-embeddings-with-keras/)。

^([21](ch04.xhtml#idm45127451025304-marker)) 您可以在[这里](https://blogs.rstudio.com/ai/posts/2020-07-30-state-of-the-art-nlp-models-from-r/)阅读更多相关信息。

^([22](ch04.xhtml#idm45127450735128-marker)) 在野外记录的动物位置标记观测。

^([23](ch04.xhtml#idm45127450728616-marker)) 表示单元格的数据，其中单元格值代表某些信息。

^([24](ch04.xhtml#idm45127450683720-marker)) 生态学家确定的高度预测物种分布的环境特征，例如湿度和温度。
