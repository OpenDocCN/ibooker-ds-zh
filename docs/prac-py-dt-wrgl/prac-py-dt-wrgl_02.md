# 第二章。Python简介

您能读懂这段话吗？如果可以，我有个好消息告诉您：学习编程不会有任何困难。为什么这么说？因为总体而言，计算机编程语言—特别是Python语言—比起自然人类语言要简单得多。编程语言是由人类设计的，主要是为了让计算机阅读，因此它们的语法更简单，所涉及的“语法成分”要比自然语言少得多。所以，如果您对阅读英语感到相对舒适—这是以其庞大词汇量、不规则拼写和发音而闻名的语言—那么学习Python基础是完全可以做到的。

到本章末尾时，您将掌握所有必要的Python技能，可以开始使用我们将在[第4章](ch04.html#chapter4)中介绍的常见数据格式进行基本数据处理。为了达到这一目标，我们将从一些基本的编码练习开始，涵盖以下内容：

+   Python基础语法及其基本语法/句法要点

+   计算机如何读取和解释您的Python代码

+   如何使用他人（以及您自己！）编写的代码“配方”快速扩展您自己代码的功能

本章节中，您会找到演示每个概念的代码片段，这些代码片段也在附带的Jupyter笔记本和独立的Python文件中收集在GitHub上；这些可以被导入Google Colab中，或者下载到计算机上运行。虽然这些文件能让您看到本章的代码实际运行，但是我*强烈建议*您创建一个新的Colab/Jupyter笔记本或者独立的Python文件，自己动手练习编写、运行和评论这些代码（如果需要回顾如何操作，请参见[“Hello World！”](ch01.html#hello_world)）。虽然您可能觉得这有些愚蠢，但实际上对于提升数据处理技能和自信心来说，没有比从零开始设置Python文件，然后看到自己编写的代码让计算机按您的意愿执行更有用的了，即使只是“简单地”从另一个文件中重新输入。是的，这样做可能会遇到更多问题，但这正是关键所在：良好的数据处理不仅仅是学会如何“正确”地做事情，*更是学会在事情出错时如何恢复*。在早期给自己留些小错误的空间（以及学会如何识别和修正它们），这才是您作为数据处理员和程序员真正进步的方式。如果您只是运行已经运行的代码，您将永远无法获得那种经验。

因为这些小错误如此重要，我在本章中包含了一些“快速前进”部分，这些部分将为您提供将我提供的代码示例推向更高级的方法，这通常涉及有意“破坏”代码。通过本章结束时，您将准备好将我们涵盖的基础知识结合到依赖于真实世界数据的完整数据整理程序中。对于每个示例，我还将包括一些明确的提醒，让您了解您希望在数据日记中包含的内容，何时需要将您的代码备份到GitHub等等，这样您就可以开始真正熟悉这些过程，并开始发展一种感觉，了解何时需要在自己的数据整理项目中采取这些步骤。

现在您知道我们的目标在哪里了——让我们开始吧！

# 编程语言的“词类”

不同的人类语言使用不同的词汇和语法结构，但它们通常共享许多基本概念。例如，让我们看看以下两个句子：

```py
My name is Susan.          // English
```

```py
Je m'appelle Susan.        // French
```

这两个句子基本表达了同样的意思：它们陈述了我的名字是什么。尽管每种语言使用不同的词汇和略有不同的语法结构，但它们都包括像主语、宾语、动词和修饰语等词类。它们还都遵循类似的语法和*语法*规则，以组织单词和思想为句子、段落等结构。

许多编程语言也分享与自然语言类似的关键结构和组织元素。为了更好地理解这一点，让我们从学习新的人类语言时常用的方式开始：探索编程语言的“词类”。

## 名词 ≈ 变量

在英语中，名词通常被描述为任何指代“人、地方或事物”的词。尽管这不是一个非常精确的定义，但这是一个便于说明名词可以是不同类型实体的一种方便方法。在编程语言中，*变量*用于保存和引用我们编写程序时使用的不同*数据类型*。然而，与“人、地方和事物”不同的是，在Python编程语言中，变量可以是五种主要的数据类型之一：

+   数字

+   字符串

+   列表

+   字典（dict）

+   布尔

与人类语言类似，不同编程语言支持的变量类型存在许多重叠：例如，在Python中我们称之为*列表*的东西在JavaScript或C中称为*数组*；另一方面，JavaScript中的*对象*在Python中正式称为*映射*（或*字典*）^([1](ch02.html#idm45143423781840))

在阅读数据类型列表后，你可能已经能猜到其中至少一些数据类型的样子。好消息是，与现实世界中的名词不同，Python 中的每种数据类型都可以通过其格式和标点符号可靠地识别——因此，只要你仔细看待围绕数据的符号，就不用担心会将*dicts*和*lists*搞混。

要感受每种数据类型独特的标点结构，请看一下[示例 2-1](#parts_of_speech)。特别是，请确保在你的代码编辑器或笔记本中打开或复制此代码，这样你就可以看到*语法高亮*的效果。例如，数字应该是一个颜色，而字符串，方括号、花括号以及注释（以`#`开头的行）应该是另一种颜色。

##### 示例 2-1\. parts_of_speech.py

```py
 # a number is just digits
 25

 # a string is anything surrounded by matching quotation marks
 "Hello World"

 # a list is surrounded by square brackets, with commas between items
 # note that in Python, the first item in a list is considered to be
 # in position `0`, the next in position `1`, and so on
 ["this","is",1,"list"]

 # a dict is a set of key:value pairs, separated by commas and surrounded
 # by curly braces
 {"title":"Practical Python for Data Wrangling and Data Quality",
  "format": "book",
  "author": "Susan E. McGregor"
 }

 # a boolean is a data type that has only two values, true and false.
 True
```

当然，这个列表远非详尽无遗；就像人类语言支持“复杂名词”（比如“理发”和“卧室”）一样，编程语言也能构建更复杂的数据类型。然而，正如你很快会看到的那样，即使只有这几种基本类型，我们也能做很多事情。

在现实世界中，我们通常也会给我们生活中许多独特的“人、地、物”取名，以便更容易地引用和交流。在编程中，我们也是如此，原因正是一样的：给变量命名让我们能以计算机能理解的方式引用和修改特定的数据。为了看看这是如何运作的，让我们尝试将一个简单的英文句子翻译成 Python 代码：

```py
The author is Susan E. McGregor
```

在阅读这句话后，你将会将“Susan E. McGregor”与标签“作者”关联起来。如果有人问你谁写了这本书，你（希望的话）会记住这个并说“Susan E. McGregor”。在 Python 代码中，相应的“句子”如[示例 2-2](#naming_variable)所示。

##### 示例 2-2\. 命名 Python 变量

```py
author = "Susan E. McGregor"
```

这段代码告诉计算机在内存中留出一个盒子，标记为`author`，然后将字符串`"Susan E. McGregor"`放入该盒子。在程序的后续部分，如果我们询问计算机关于`author`变量，它会告诉我们它包含字符串`"Susan E. McGregor"`，如[示例 2-3](#printing_variable)所示。

##### 示例 2-3\. 打印 Python 变量的内容

```py
# create a variable named author, set its contents to "Susan E. McGregor"
author = "Susan E. McGregor"

# confirm that the computer "remembers" what's in the `author` variable
print(author)
```

### 名字中有什么含义？

在[示例 2-3](#printing_variable)中，我选择将我的变量命名为`author`，但这个选择并没有什么神奇之处。原则上，你几乎可以按照任何你想要的方式命名变量——唯一的“硬性规则”是变量名不能：

+   以数字开头

+   包含除了下划线（`_`）之外的标点符号

+   是“保留”字或“关键字”（比如 Number 或 Boolean，例如）

例如，在 [示例 2-3](#printing_variable) 中，我也可以称变量为 `nyc_resident` 或者甚至 `fuzzy_pink_bunny`。最重要的是，作为程序员的你，遵循前面列出的一些限制，*并且*在稍后访问其内容时使用*完全相同*的变量名（大小写敏感！）。例如，创建一个包含 [示例 2-4](#variable_names) 代码的新Python文件，然后运行它查看结果。

##### 示例 2-4\. noun_examples.py

```py
# create a variable named nyc_resident, set its contents to "Susan E. McGregor"
nyc_resident = "Susan E. McGregor"

# confirm that the computer "remembers" what's in the `nyc_resident` variable
print(nyc_resident)

# create a variable named fuzzyPinkBunny, set its contents to "Susan E. McGregor"
fuzzyPinkBunny = "Susan E. McGregor"

# confirm that the computer "remembers" what's in the `fuzzyPinkBunny` variable
print(fuzzyPinkBunny)

# but correct capitalization matters!
# the following line will produce an error
print(fuzzypinkbunny)
```

### 变量命名的最佳实践

虽然在 [示例 2-4](#variable_names) 中使用的所有示例都是*合法*的变量名，但并非所有都是特别*好*的变量名。正如我们在本书中将看到的那样，写好代码——就像任何其他写作一样——不仅仅是编写“可运行”的代码；它还关乎代码对计算机和人类的有用性和可理解性。因此，我认为良好命名变量是良好编程的重要组成部分。实际上，*好*的变量名包括：

+   描述性的

+   在给定文件或程序中唯一的

+   可读性强的

因为实现前两个属性通常需要使用多个单词，程序员通常使用两种风格约定来确保其变量名也保持*可读性*，这两种风格都显示在 [示例 2-4](#variable_names) 中。一种方法是在单词之间添加下划线 (`_`)（例如，`nyc_residents`），或者使用“驼峰命名法”，其中每个单词的首字母（除了第一个）大写（例如，`fuzzyPinkBunny`）。通常情况下，你应该选择一种风格并坚持使用它，尽管即使混合使用它们，你的代码也将正常工作（并且大多数情况下会满足可读性标准）。在本书中，我们将主要使用下划线，这也被认为更符合“Python风格”。

## 动词 ≈ 函数

在英语中，动词通常被描述为“动作”或“存在状态”。我们已经看到了后者的编程语言等价物：等号 (`=`) 和在前面示例中使用的 `print()` 函数。在英语中，我们使用“to be”动词的形式来描述某物*是*什么；在Python（以及许多其他编程语言中），变量的值*是*等号右侧出现的内容。这也是为什么等号有时被描述为*赋值操作符*的原因。

在编程中，“动作动词”的等价物是*函数*。在 Python 和许多其他编程语言中，有*内置函数*，它们表示语言“知道如何执行”的任务，例如通过`print()`函数输出结果。虽然类似，*方法*是专门设计用于特定数据类型并且需要在该数据类型的变量上“调用”的特殊函数以便工作。给定数据类型的可用方法往往反映了您可能希望执行的常见任务。因此，正如大多数人能够行走、说话、吃东西、喝水和抓取物体一样，大多数编程语言都有*字符串方法*，可以执行像将两个字符串粘合在一起（称为*连接*）或将两个字符串分开等任务。但由于在数字 5 上“拆分”没有意义，数字数据类型*没有*`split()`方法。

在实践中，内置函数和方法之间有什么区别？实际上并没有太大区别，除了我们在 Python “句子”或*语句*中包含这些“动词”的方式不同。对于内置函数，我们可以简单地写出函数名，并通过将它们放置在圆括号之间传递它所需的任何“参数”。例如，如果你回想一下我们的[示例 1-1](ch01.html#hello_world_code)，我们只需将字符串`Hello World!`传递给`print()`函数，就像这样：

```py
 print("Hello World!")
```

然而，在`split()`方法的情况下，*我们必须将方法附加到特定的字符串*上。该字符串可以是一个*文字*（即一系列被引号包围的字符），或者它可以是一个其值为字符串的变量。尝试在[示例 2-5](#split_string)中的代码文件或笔记本中执行，并查看您会得到什么样的输出！

##### 示例 2-5\. method_madness.py

```py
# splitting a string "literal" and then printing the result
split_world = "Hello World!".split()
print(split_world)

# assigning a string to a variable
# then printing the result of calling the `split()` method on it
world_msg = "Hello World!"
print(world_msg.split())
```

请注意，如果您尝试在不合适的数据类型或者没有意义的数据类型上运行`split()`方法，您将会收到一个错误。连续尝试这些操作（或者如果您使用笔记本，则在两个不同的单元格中），看看会发生什么：

```py
# the following will produce an error because
# the `split()` method must be called on a string in order to work!
split("Hello World!")
```

```py
# the following will produce an error because
# there is no `split()` method for numbers!
print(5.split())
```

与数据类型一样，方法和函数因其排版和标点而容易识别。内置函数（如`print()`）在代码编辑器或笔记本中会显示特定的颜色。例如，在 Atom 的默认 One Dark 主题中，变量名为浅灰色，操作符如`=`为紫色，内置函数如`print()`则为青色。您还可以通过它们的相关标点来识别函数：任何您看到的文本*紧接*在圆括号后面（例如`print()`），您都在看一个函数。方法也是如此，只是它们总是在适当的数据类型或变量名称之前，并通过一个句点(`.`)与之分隔。

在像 Python 这样的编程语言中，仅使用运算符、方法和内置函数就可以完成相当多的工作，特别是在执行基本数学等任务时。然而，在数据处理方面，我们需要更高级的技术。正如我们可以将复杂任务（如弹奏钢琴或踢足球）视为许多较简单动作的精心组合，例如移动手指或脚一样，通过深思熟虑地组合相对简单的运算符、方法和内置函数，我们可以构建非常复杂的编程功能。这些*用户定义函数*是我们可以通过创建代码“配方”，从而真正增强我们代码能力的地方，这些“配方”可以一次又一次地使用。

例如，假设我们想向两个不同的人打印相同的问候语。我们可以简单地像我们一直以来所做的那样使用`print()`函数，如[示例 2-6](#two_prints)所示。

##### 示例 2-6\. basic_greeting.py

```py
# create a variable named author
author = "Susan E. McGregor"

# create another variable named editor
editor  = "Jeff Bleiel"

# use the built-in print function to output "Hello" messages to each person
print("Hello "+author)
print("Hello "+editor)
```

关于[示例 2-6](#two_prints)中的代码，有几件事需要注意。首先，使用`print()`函数完全可以胜任；这段代码完全完成了任务。这很棒！第一次为某事编写代码（包括特定的数据处理任务），这基本上是我们的主要目标：确保其正确工作。

然而，一旦我们完成了这一点，我们可以开始考虑一些简单的方法来使我们的代码“更干净”和更有用。在先前的示例中，两个打印语句完全相同，只是使用的变量不同。每当我们在代码中看到这种重复时，这表明我们可能希望制作自己的用户定义函数，就像在[示例 2-7](#greet_me)中一样。

##### 示例 2-7\. greet_me.py

```py
# create a function that prints out a greeting
# to any name passed to the function

def greet_me(a_name):
    print("Hello "+a_name)

# create a variable named author
author = "Susan E. McGregor"

# create another variable named editor
editor  = "Jeff Bleiel"

# use my custom function, `greet_me` to output "Hello" messages to each person
greet_me(author)
greet_me(editor)
```

很神奇，对吧？在某些方面，我们几乎没有改变什么，但实际上在[示例 2-7](#greet_me)中，有很多东西。现在我们将花几分钟时间来突出一些在这里使用的新概念，但如果一开始不完全理解，不要担心——我们将在本书中继续讨论这些想法。

我们在[示例 2-7](#greet_me)中的主要工作是编写我们的第一个自定义函数`greet_me()`。我们通过使用几种不同的语法结构和排版指示符告诉计算机，我们希望它创建并记住这个函数以供将来使用。其中一些约定与我们已经见过的用于创建自定义变量的约定相匹配（例如使用描述性名称`greet_me()`），以及内置函数和方法的约定，例如紧接在函数名后面立即使用圆括号`()`。

在[图 2-1](#function_structure_diagram)中，我已经绘制了我们的`greet_me()`函数的代码图，以突出显示每一行的内容。

![自定义函数的组成部分](assets/ppdw_0201.png)

###### 图 2-1\. 自定义函数的组成部分

如你所见，从[图 2-1](#function_structure_diagram)可以看出，创建自定义函数意味着向计算机包括多个指标：

+   `def`关键字（缩写*def*ine）告诉计算机接下来是一个函数名。

+   立即跟在函数名后面的圆括号强调这是一个函数，并用于包围函数的*参数*（如果有的话）。

+   冒号（`:`）表示接下来缩进的代码行是函数的一部分。

+   如果我们想访问作为*参数*传递给我们函数的变量，我们使用函数定义中圆括号之间出现的“局部”*参数*名称。

+   我们可以在我们的自定义函数中使用任何类型的函数（包括内置和自定义）或方法*内部*。这是构建高效、灵活代码的关键策略。

当涉及到使用或“调用”我们的函数时，我们只需写函数名（`greet_me()`），确保在括号中放入与函数定义中出现的“配料”数量相同即可。由于我们定义了`greet_me()`函数仅接受*一个*参数“配料”（在这种情况下是`a_name`参数），当我们想要使用它时，必须提供*恰好*一个参数，否则将会出错。

## 使用自定义函数进行烹饪

正如你可能注意到的，我喜欢将用户定义的或“自定义”函数看作编程的“食谱”。像食谱一样，它们为计算机提供了将一个或多个原始数据“配料”转换为其他有用资源的可重复使用指令。有时只有一个参数或“配料”，就像我们的`greet_me()`食谱中一样；有时有多个参数，类型各异。写自定义函数没有严格的“对”或“错”—就像写烹饪食谱一样；每个人都有自己的风格。同时，当决定应该将什么放入给定函数时的策略，可以考虑我们通常如何使用（甚至可能编写！）烹饪食谱。

例如，显然可以*可能*编写一个称为“感恩节”的单一烹饪食谱，描述如何从头到尾制作整个假日大餐。根据你的假日风格，可能需要2到72小时“运行”，每年一次非常有用—其他时候几乎从不使用。如果你*确实*只想制作那个庞大食谱的一部分—也许你想在新年时享用感恩节的土豆泥—你首先必须查找感恩节的说明，识别并组合*只有*制作土豆泥的配料和步骤。这意味着在你开始烹饪之前要投入大量工作！

因此，虽然我们希望我们的自定义函数比计算机已经能够做的*稍微*复杂一些，但我们通常不希望它们真正*复杂*。就像“感恩节”食谱一样，制作巨大的函数（甚至程序）会限制它们在多大程度上可以被重复使用。实际上，创建简单而专注的函数会使我们的代码在长远来看更加有用和灵活——这是我们将在[第8章](ch08.html#chapter8)中详细探讨的一个过程。

## 库：从其他编程者那里借用自定义函数

如果自定义函数是编程配方，那么*库*就是编程食谱书：大量*其他人*编写的自定义函数的集合，*我们*可以用来转换我们的原始数据成分，而不必从零开始想出并编写我们自己的配方。正如我在[第1章](ch01.html#chapter1)中提到的那样，编写有用的Python库的大社区是我们首先选择使用Python的原因之一——正如你将在本章末尾的[“使用Citi Bike数据踏上旅途”](#hitting_the_road_intro)中看到的那样，使用它们既有用又强大。

然而，在我们真正利用库之前，我们需要掌握Python的另外两个基本语法结构：*循环*和*条件语句*。

# 掌控：循环和条件语句

正如我们所讨论的，编写Python代码在许多方面与用英语写作相似。除了依赖一些基本的“词类”之外，Python代码是从左到右编写并且基本上是从上到下阅读的。但计算机在数据整理程序中的路径更像是[“选择你自己的冒险”书](https://zh.wikipedia.org/wiki/选择你自己的冒险)而不是传统的论文或文章：根据程序员——也就是你提供的命令，一些代码片段可能会根据数据或其他因素被跳过或重复执行。

## 在循环中

当我们用Python进行数据整理时，我们最常见的目标之一是对数据集中的每条记录做*某些*处理。例如，假设我们想要对一组数字进行求和以找到它们的总和：

```py
# create a list that contains the number of pages in each chapter
# of a fictional print version of this book

page_counts = [28, 32, 44, 23, 56, 32, 12, 34, 30]
```

如果你需要在没有编程的情况下对一组数字进行求和，你有几个选择：你可以使用计算机上的计算器程序，实体计算器，或者甚至（哇！）一支铅笔和纸。如果你知道如何使用电子表格程序，你可以在那里输入每个数据项并使用`SUM()`函数。对于较短的列表，这些解决方案可能都还好，但它们不适合*扩展*：当然，手工（或计算器）加起来10个数字可能不会花费太长时间，但加起来100个数字就不同了。在时间上电子表格解决方案要好一些，但它仍然需要许多外部步骤——比如将数据复制粘贴到电子表格中，并基本手动选择应该加总的行或列。通过编程解决方案，我们几乎可以避免所有这些缺点——无论我们需要加总10行还是10百万行，我们的工作量几乎没有增加，计算机实际计算的时间也只会略微延长。

当然，因为编程仍然是在编写，我们可以表达给计算机的指令有多种方式。一种方式是让计算机查看列表中的每个数字并保持运行总数，就像在[示例 2-8](#pagecount_loop)中所示的那样。

##### 示例 2-8\. page_count_loop.py

```py
# fictional list of chapter page counts
page_counts = [28, 32, 44, 23, 56, 32, 12, 34, 30]

# variable for tracking total page count; starting value is 0
total_pages = 0

# for every item in the list, perform some action
for a_number in page_counts:

    # in this case, add the number to our "total_pages" variable
    total_pages = total_pages + a_number

print(total_pages)
```

在我们探讨其他可能告诉计算机执行此任务的方式之前，让我们先分解[示例 2-8](#pagecount_loop)。显然，我们从数字列表开始。接下来，我们创建一个变量来跟踪`total_pages`，我们必须显式地为其分配一个起始值`0`（大多数计算器程序会更或多少隐式地执行此操作）。最后，我们开始遍历我们的列表：

```py
for a_number in page_counts:
```

要理解这行代码最简单的方法是像读英文句子一样大声说出来：“对于列表中的每个`a_number`，`page_counts`执行以下操作。”实际上，事情就是这样的。对于`page_counts`列表中的每个项目，计算机都会按照缩进在`for...in...:`语句下的代码指令执行。在这种情况下，这意味着将`total_pages`的当前值与`a_number`的值相加，并将结果再次存储回`total_pages`中。

在某些方面，这是很直接的：我们已经非常明确地告诉计算机了`page_counts`（这是我们的数字列表）和`total_pages`的值。但`a_number`呢？它从哪里来的，计算机如何知道在哪里找到它？

像`print()`语句或`def...function_name():`结构一样，`for...in...:`配置内置于Python语言中，这就是为什么在编码时我们不必给它那么多指示。在这种情况下，`for...in...:`语句要提供的两个东西是：一个类似列表的变量（在本例中是`page_counts`）和计算机用来引用列表中当前项目的名称（在本例中是`a_number`），如[图 2-2](#for_loop_diagram)所示。

![`for`循环的结构。](assets/ppdw_0202.png)

###### 图 2-2\. `for`循环的结构

与所有变量名称一样，变量名称`a_number`没有任何“魔力”——我认为这是一个很好的选择，因为它具有描述性和可读性。重要的是，当我希望计算机对每个列表项执行某些操作时，我在缩进代码中使用的变量名称必须与我在顶部`for...in...:`语句中编写的名称相匹配。

在编程术语中，这种`for...in...:`结构被称为*for循环*，并且每种常用的编程语言都有它。之所以称为“循环”，是因为对于提供的列表中的每个项目，计算机都会运行每一行相关的代码——在Python中的情况下，是`for...in...:`语句下的每一行缩进的代码——然后移动到下一个项目并再次“循环”回第一行缩进的代码。当我们的“循环”只有一行代码时，这有点难以看到，因此让我们添加几行代码来更好地说明正在发生的情况，如[示例 2-9](#page_loop)所示。

##### 示例 2-9\. page_count_printout.py

```py
# fictional list of chapter page counts
page_counts = [28, 32, 44, 23, 56, 32, 12, 34, 30]

# variable for tracking total page count; starting value is 0
total_pages = 0

# for every item in the list, perform some action
for a_number in page_counts:
    print("Top of loop!")
    print("The current item is:")
    print(a_number)
    total_pages = total_pages + a_number
    print("The running total is:")
    print(total_pages)
    print("Bottom of loop!")

print(total_pages)
```

到目前为止，你可能会想：“这似乎是为了对一组数字求和而做很多工作。”当然，有一种更有效的方法来完成这个*特定*任务：Python有一个内置的`sum()`函数，它将接受我们的数字列表作为参数（见[示例 2-10](#pagecounts_sum)）。

##### 示例 2-10\. 使用`sum()`函数

```py
# fictional list of chapter page counts
page_counts = [28, 32, 44, 23, 56, 32, 12, 34, 30]

# `print()` the result of using the `sum()` function on the list ![1](assets/1.png)
print(sum(page_counts))
```

[![1](assets/1.png)](#co_introduction_to_python_CO1-1)

尝试自己将其添加到现有程序中！

即使我们从一开始就可以使用`sum()`函数，我也利用这个机会介绍了`for`循环，有几个原因。首先，因为这是一个很好的提醒，即使是对于简单的编程任务，也总会有不止一种方法。其次，因为`for`循环是数据整理（实际上是所有编程）的重要部分，`for...in...:`循环是我们将用于过滤、评估和重新格式化数据的关键工具之一。

## 一个条件…

`for`循环为我们提供了查看数据集中每个项目的简单方法，但数据整理还需要对数据做出*决策*。通常情况下，这意味着评估数据的某些方面，并根据数据的特定值执行某些操作（或什么也不做）。例如，如果我们想知道这本书有多少章节超过30页，有多少少于30页，我们需要一种方法来：

1.  检查我们`page_counts`列表中特定数字是否超过30。

1.  如果超过30，则将`over_30`计数器加1。

1.  否则，将1添加到我们的`under_30`计数器中。

幸运的是，Python具有一种内置的语法结构，可以做到这种类型的评估和决策：`if...else`语句。让我们通过修改[示例 2-9](#page_loop)中的`for`循环来看看它是如何工作的，以跟踪超过和少于30页的章节数量。

##### 示例 2-11\. page_count_conditional.py

```py
# fictional list of chapter page counts
page_counts = [28, 32, 44, 23, 56, 32, 12, 34, 30]

# create variables to keep track of:
# the total pages in the book
total_pages = 0

# the number of chapters with more than 30 pages,
under_30 = 0

# the number of chapters with fewer than 30 pages
over_30 = 0

# for every item in the page_counts list:
for a_number in page_counts:

    # add the current number of pages to our total_pages count
    total_pages = total_pages + a_number

    # check if the current number of pages is more than 30
    if a_number > 30:

        # if so, add 1 to our over_30 counter
        over_30 = over_30 + 1

    # otherwise...
    else:
        # add 1 to our under_30 counter
        under_30 = under_30 + 1

# print our various results
print(total_pages)
print("Number of chapters over 30 pages:")
print(over_30)
print("Number of chapters under 30 pages:")
print(under_30)
```

与`for`循环一样，理解`if...else`条件语句中正在发生的事情最简单的方法是大声说出来（并且同样重要的是，在代码的注释中写下这个句子）：“`if`当前页数大于30，则将`over_30`计数器加一。否则（`else`），将`under_30`计数器加一。”

虽然这个直觉上是有点意义的，但我想再次放慢脚步，更详细地讲解一下正在发生的事情，因为`if...else`语句是我们将反复使用的另一种编程结构。

首先，让我们看一下前面示例的缩进结构：所有属于`for`循环的代码从左边缘缩进一个制表符；这是计算机知道该代码“在”该循环内部的方式。类似地，属于`if...else`语句的每个部分的代码再次缩进一个制表符。在Python中，这种逐步缩进的过程实际上是*必需的*，否则代码将无法正常工作—如果缩进不正确，Python会抱怨。^([2](ch02.html#idm45143426063952)) 这种机制通常称为*嵌套*。更直观地理解正在发生的事情的一种方法显示在[图 2-3](#loop_nesting_diagram)中。

![代码嵌套的文氏图](assets/ppdw_0203.png)

###### 图2-3\. 代码“嵌套”

嵌套的几个含义我们将在本书的后面进行探讨，但现在的主要要点是为了使一行代码“属于”一个函数、循环或条件语句，*它必须缩进到比您想要它属于的结构右边缩进一个制表符的位置*。为了看到这一点的实际效果，让我们把到目前为止我们所做的一切集成起来，创建一个使用循环、条件语句和自定义定义的函数的Python程序，如[示例 2-12](#page_counts_complete)所示。

##### 示例 2-12\. page_count_custom_function.py

```py
# fictional list of chapter page counts
page_counts = [28, 32, 44, 23, 56, 32, 12, 34, 30]

# define a new `count_pages()` function that takes one ingredient/argument:
# a list of numbers
def count_pages(page_count_list): ![1](assets/1.png)

    # create variables to keep track of:
    # the total pages in the book
    total_pages = 0

    # the number of chapters with more than 30 pages,
    under_30 = 0

    # the number of chapters with fewer than 30 pages
    over_30 = 0

    # for every item in the page_count_list:
    for a_number in page_count_list: ![2](assets/2.png)

        # add the current number of pages to our total_pages count
        total_pages = total_pages + a_number

        # check if the current number of pages is more than 30
        if a_number > 30:

            # if so, add 1 to our over_30 counter
            over_30 = over_30 + 1

        # otherwise...
        else:
            # add 1 to our under_30 counter
            under_30 = under_30 + 1

    # print our various results
    print(total_pages)
    print("Number of chapters over 30 pages:")
    print(over_30)
    print("Number of chapters under 30 pages:")
    print(under_30)

# call/execute this "recipe", being sure to pass in our
# actual list as an argument/ingredient
count_pages(page_counts) ![3](assets/3.png)
```

[![1](assets/1.png)](#co_introduction_to_python_CO2-1)

我们可以通过缩进一个制表符并添加函数定义行将我们现有的代码“包装”到一个新函数中（这里称为`count_pages()`）。

[![2](assets/2.png)](#co_introduction_to_python_CO2-2)

我们必须将`for`循环引用的列表变量名称与函数定义中圆括号之间提供的参数名称匹配，这一点在![1](assets/1.png)中已经提到。

[![3](assets/3.png)](#co_introduction_to_python_CO2-3)

函数在实际*调用*或*执行*之前并不执行任何操作。在这一点上，我们需要为其提供我们希望其处理的具体参数/参数。

如果你将[示例 2-12](#page_counts_complete)与[示例 2-11](#page_counts)进行比较，你会发现我实际上只是从[示例 2-11](#page_counts)复制了代码，并做了三件事：

1.  我添加了函数定义语句`def count_pages(page_count_list):`。

1.  我将所有现有代码都缩进了一个额外的制表符，以便计算机将其视为“属于”我们新的`count_pages()`函数。在Atom中，你可以通过高亮显示要移动的所有代码行并按Tab键来一次性完成这个操作。我还更新了`for`循环顶部引用的变量，使其与函数`def`行中圆括号内提供的参数名匹配。

1.  我确保在最后“调用”了函数，将`page_counts`变量作为“成分”或*参数*传递给它。请注意，`count_pages(page_counts)`语句根本没有缩进。

希望你开始逐渐理解所有这些是如何结合在一起的。不过，在我们开始使用这些工具进行一些真实世界的数据处理之前，我们需要花些时间讨论当代码出现问题时会发生什么。

# 理解错误

正如我们在[第1章](ch01.html#chapter1)中提到的，计算机非常擅长快速执行重复的任务（通常也很准确）。这使得我们能够编写可以很好*扩展*的程序：能够对包含10项的列表（如我们的`page_counts`示例）进行求和或排序的相同代码也可以相当有效地用于包含10,000项的列表。

然而，与此同时，计算机确实、重要且不可挽回地愚蠢。计算机无法真正推断或创新——它们只能根据人类提供给它们的指令和数据选择其代码路径。因此，写代码与给幼儿下指令非常相似：你必须非常字面和非常明确，如果发生意外，你应该期待一场发脾气。^([3](ch02.html#idm45143428265504))

例如，当人类在书面句子中遇到拼写或语法错误时，我们通常甚至不会注意到它：根据周围句子或段落的上下文，大部分时间我们会在不费力气的情况下推断出适当的意思。事实上，即使一个句子中几乎*所有*单词中的*所有*字母都被重新排列，我们通常也能够毫不费力地阅读出来。^([4](ch02.html#idm45143428263280)) 相比之下，计算机只会大声抱怨并停止读取，如果你的代码中有一个逗号错位的话。

因此，编程中的错误不仅是不可避免的，而且是*预期的*。无论你编程多少，你写的任何代码块，只要超过几行，都会有各种类型的错误。与其担心如何*避免*错误，学习如何*解释*和*纠正*错误会更有用。在本书中，我有时会故意制造错误（或者鼓励你这样做，如在[“快进”](#greet_me_ff)和[“快进”](#loop_errors)中），这样你就可以熟悉它们，并开始制定自己的处理过程。作为起点，我们将讨论编程中发生的三种主要*类型*错误：语法错误、运行时错误和逻辑错误。

## 语法问题

在编程中的语法错误或*语法*错误可能同时是你遇到的最简单和最令人沮丧的错误类型之一——部分原因是它们非常频繁发生，而计算机对此却表现得非常吵闹。我之前提到的逗号错位的例子就是语法错误的一个例子：在某种程度上，你的代码违反了编程语言的语法规则。

我将这些错误描述为“简单”的原因是它们几乎总是如此。在我的经验中，大多数语法错误——甚至可以推广到编程错误——基本上都是笔误：逗号或引号被遗忘，或者一行代码的缩进过多或不足。不幸的是，许多我合作过的新手程序员似乎特别对这些错误感到沮丧，正是因为它们简单——因为他们觉得自己犯了这些错误很愚蠢。

实际上，经验丰富的程序员一直在经历语法错误。如果你刚开始学习，你可以从语法错误中学到的主要一点是如何*不*让它们让你步步后退。事实上，我在前面的“快进”部分中故意介绍如何故意“破坏”代码的原因之一，就是帮助说明错误是如何容易发生和被修复的。在编程时，你将学到的最重要的技能之一就是如何频繁地犯错，但不要因此而灰心。

如果你运行的代码出现语法错误，你会知道，因为（通常是多行）错误消息的最后一行会显示`SyntaxError`。但比起这个，更实用的是错误消息中告诉你错误出现在*哪个文件*和*哪一行*的部分，在实际*修复*错误时更为重要。随着时间的推移，只需转到那行代码并查找问题（通常是缺少标点：逗号、括号、冒号和引号）就足以发现问题所在。尽管错误消息还会包括（可能）有问题的代码行，并在Python认为缺少字符的地方下方标注一个插入符（`^`），但这并非万无一失。例如，[示例 2-13](#object_error) 展示了一个Python `dict` 缺少一行上的逗号的情况。

##### 示例 2-13\. 引入错误

```py
1 # although the actual error is on line 4 (missing comma)
2 # the error message points to line 5
3 book = {"title":"Practical Python for Data Wrangling and Data Quality",
4  "format": "book"
5  "author": "Susan E. McGregor"
6 }
```

这里是错误输出：

```py
  File "ObjectError.py", line 5
    "author": "Susan E. McGregor"
            ^
SyntaxError: invalid syntax
```

如你所见，尽管[示例 2-13](#object_error) 中的代码在第4行值`"book"`后缺少逗号，计算机却报告错误发生在第5行（因为计算机在那时意识到有问题）。总的来说，通常可以在计算机报告错误的行（或前一行）找到语法错误。

## 运行时困扰

编程中的*运行时*错误用于描述在“运行”或*执行*代码过程中出现的任何问题。与语法错误一样，运行时错误的很大一部分实质上也是拼写错误，例如错误复制的变量名称。例如，每当你看到一个包含短语*`some_variable`* `is not defined`的错误时，几乎可以肯定你的代码中有不匹配的变量名称（记住：大小写敏感！）。由于阅读整个“回溯”错误可能会有点复杂（它们倾向于引用Python编程语言的内部工作方式，而我个人认为这并不十分有用），建议直接从错误中复制变量名，然后在代码中进行大小写*不敏感*搜索（这是Atom的默认行为）。这种方法将突出显示变量名的类似（但*不完全*相同）拼写，加快查找不匹配的速度。

例如，在[示例 2-14](#param_mismatch) 中，函数定义中提供给`greet_me(a_name)`的参数名称与后续代码体中使用的名称*并不完全*匹配。

##### 示例 2-14\. 稍微不匹配的变量名称会生成运行时错误

```py
# create a function that prints out a greeting to any name passed to the function

def greet_me(a_name):
    print("Hello "+A_name)

# create a variable named author
author = "Susan E. McGregor"

# pass my `author` variable as the "ingredient" to the `greet_me` function
greet_me(author)
```

因为函数定义圆括号内部出现的参数名称始终优先，运行[示例 2-14](#param_mismatch) 中的代码会生成以下错误：

```py
  File "greet_me_parameter_mismatch.py", line 10, in <module>
    greet_me(author)
  File "greet_me_parameter_mismatch.py", line 4, in greet_me
    print("Hello "+A_name)
NameError: global name 'A_name' is not defined
```

注意，通常情况下，错误消息的最后几行提供了最有用的信息。最后一行告诉我们，我们尝试使用变量名`A_name`，但在此之前并未先定义它，而上一行包含了它出现的实际代码。有了这两条信息（再加上我们的搜索策略），在我们找到错误源之前可能不会花费太长时间。

另一种非常常见的运行时错误类型是在尝试对特定数据类型进行不适当操作时发生。在[“快进”](#greet_me_ff)中，你可能尝试运行代码`greet_me(14)`。在这种情况下，错误输出的最后一行将包含`TypeError`，这意味着我们的某部分代码接收到了与预期不同的数据类型。在这个例子中，问题在于该函数期望一个字符串（可以使用`+`号“添加”或*连接*到另一个字符串），但我们提供了一个数字，即`14`。

修复此类错误的挑战在于准确定位问题所在有些许棘手，因为涉及到变量值的*赋值*和实际*使用*位置之间的不匹配。特别是当你的程序变得更加复杂时，这两个过程可能在代码中完全不相关。例如，查看来自[示例 2-14](#param_mismatch)的错误输出，你可以看到它报告了两个位置。第一个是将变量传递到函数的行，因此可用值被*赋予*的位置：

```py
File "greet_me_parameter_mismatch.py", line 10, in <module>
```

第二个是传递给它的值被*使用*的行：

```py
File "greet_me_parameter_mismatch.py", line 4, in greet_me
```

正如我已经提到的，从一个问题变量或值的*使用*位置开始进行调试工作，然后逆向查找到分配值的代码行通常是有帮助的；只是要知道，分配值的代码行可能实际上并没有出现在错误输出中。这是使这类运行时错误比普通语法错误更难跟踪的部分原因之一。

运行时错误之所以难以诊断是我推荐频繁保存和测试你的代码的一个关键原因。如果自上次测试以来你只写了几行新代码，那么识别新运行时错误的来源将更加容易，因为问题*必然*出现在这段新代码中。采用这种`编写、运行、重复`的方法，当你需要查找任何新错误的来源时，需要覆盖的范围将会少得多，并且你可能能够相对快速地修复它们。

## 逻辑损失

到目前为止，最棘手的编程问题绝对是*逻辑*错误，这个术语广泛用来描述当你的程序*运行*时，只是*不是你打算的方式*发生的情况。这些错误特别阴险，因为从计算机的角度来看，一切都很正常：你的代码没有试图执行任何它认为混乱或“错误”的操作。但是请记住，计算机很笨，所以它们会很高兴地让你的程序执行产生毫无意义、混乱甚至误导性的结果。事实上，我们已经遇到了这种情况！

如果您回顾一下[示例 2-12](#page_counts_complete)，您会注意到我们的注释表达的意图与我们的实际代码所做的事情之间存在轻微的不一致。该示例的输出如下：

```py
291
Number of chapters over 30 pages:
5
Number of chapters under 30 pages:
4
```

但我们的数据看起来是这样的：

```py
 page_counts = [28, 32, 44, 23, 56, 32, 12, 34, 30]
```

看出问题了吗？虽然我们确实有5章的页数超过30页，但实际上只有3章的页数*不到*30页——有一章正好是30页。

现在，这可能看起来并不是一个特别重要的错误——毕竟，如果一个30页的章节被归类到少于30页的章节中，这有多重要呢？但是想象一下，如果这段代码不是用来计算章节页数，而是用来确定投票资格呢？如果这段代码只计算“超过18岁”的人数，将会有成千上万的人失去选举权。

要修复这个错误，在代码调整方面并不困难或复杂。我们只需要改变这一点：

```py
 # check if the current number of pages is more than 30
 if a_number > 30:
```

改为：

```py
 # check if the current number of pages is greater than or equal to 30
 if a_number >= 30:
```

这种类型错误的挑战在于，它完全依赖于我们作为程序员的勤奋，以确保在我们*设计*程序时，我们不会忽视可能导致不正确测量或结果的某些数据值或差异。因为计算机无法警告我们逻辑错误，避免它们的唯一可靠方法是一开始就仔细计划我们的程序，并仔细“审查”结果。在[“使用Citi Bike数据上路”](#hitting_the_road_intro)中，我们将看到当我们编写处理真实数据集的程序时，这个过程是什么样子的。

哎呀！现在我们已经涵盖了Python编程的所有基础知识（真的！），我们准备从我们关于本书的“玩具”示例中继续，使用（某种程度上）关于这本书的数据。为了让我们的脚步湿润，使用一个真实数据集进行数据整理任务，我们将转向来自纽约市自行车共享系统Citi Bike的信息。

# 使用Citi Bike数据上路

每个月，数百万人使用自行车共享系统在世界各地的城市和城镇中行驶。纽约市的Citi Bike计划于2013年推出了6000辆自行车，在2020年，该系统见证了其[第1亿次骑行](https://citibikenyc.com/about)。

Citi Bike提供了关于其系统运营的免费可访问数据，这些数据既是实时的又是历史的。为了查看使用Citi Bike数据回答一个简单问题所需的步骤，我们将使用Citi Bike数据来回答一个简单问题：Citi Bike不同类型的骑行者每天骑行多少次？

我们需要我们的Python技能来回答这个问题，因为Citi Bike骑行者每天骑行数以十万计——这意味着这些数据的一天对于Microsoft Excel或Google Sheets来说有太多的行了。但即使在Chromebook上，Python也不会在处理这些数据量时遇到问题。

要思考如何处理这个问题，让我们重新审视数据整理的步骤，详见[“什么是“数据整理”？”](ch01.html#describing_data_wrangling)：

1.  定位或收集数据

1.  评估数据的质量

1.  “清洁”、标准化和/或转换数据

1.  分析数据

1.  可视化数据

1.  传达数据

对于这个练习，我们将专注于步骤1至4，尽管你会看到，我已经做了一些准备工作，这将减少我们花在某些步骤上的时间。例如，我已经找到了[Citi Bike系统的数据](https://citibikenyc.com/system-data)，并下载了[2020年9月的出行历史数据](https://s3.amazonaws.com/tripdata/index.html)，确认了`User Type`列中出现的唯一值是`Customer`和`Subscriber`，并将整个九月的数据集削减为*仅仅*在2020年9月1日开始的骑行。虽然我们将在后续章节中详细介绍如何执行所有这些过程，但现在我想专注于如何将本章的教训应用于我们没有为自己准备的数据。^([7](ch02.html#idm45143427916368))

## 从伪代码开始

无论项目的数据整理大小如何，最好的方法之一是事先计划您的方法，并通过伪代码将该程序大纲包含在您的Python文件中。伪代码基本上意味着逐步编写您的程序将要做的事情，用普通英语（尽管您当然可以用其他自然语言进行伪代码，如果您更喜欢！）。除了为您提供思考程序需要完成的任务的空间而无需担心如何编写代码外，伪代码还将为您提供一个宝贵的参考，以便在需要休息一下项目并稍后回来时，确定下一步工作。尽管您无疑会遇到许多不经常完成此过程部分的专业程序员，但我可以保证这将帮助您更快地完成整理项目，并且这种习惯在任何专业的编程或数据科学环境中都受欢迎。

我更喜欢将程序大纲和伪代码放在我的Python文件顶部，作为一大块注释。起初，我将做三件事：

1.  陈述我的问题。

1.  描述如何“回答”我的问题。

1.  用简单的语言概述我程序将采取的步骤。

这意味着我要做的第一件事就是在我的文件中写很多注释，正如你在 [示例 2-15](#citibike_comment_block) 中看到的那样。

##### 示例 2-15\. hitting_the_road_with_citibike.py

```py
# question: How many Citi Bike rides each day are taken by
# "subscribers" versus "customers"?

# answer: Choose a single day of rides to examine.
# the dataset used for this exercise was generated from the original
# Citi Bike system data found here: https://s3.amazonaws.com/tripdata/index.html
# filename: 202009-citibike-tripdata.csv.zip

# program Outline:
# 1\. read in the data file: 202009CitibikeTripdataExample.csv
# 2\. create variables to count: subscribers, customers, and other
# 3\. for each row in the file:
#       a. If the "User Type" is "Subscriber," add 1 to "subscriber_count"
#       b. If the "User Type" is "Customer," add 1 to "customer_count"
#       c. Otherwise, add 1 to the "other" variable
# 4\. print out my results
```

现在程序大纲已经搞定，是时候开始我们程序的第一部分了：读取我们的数据。

###### 提示

每次像这样开始一个新文件时，请记住，即使它保存在本地 Git 存储库文件夹内，你也需要运行 `git add` 来备份你所做的任何更改（记住，你不能在 `add` 文件之前 `commit` 文件）。这些步骤在 [“别忘了 Git！”](#dont_forget_git) 中有详细说明，以防你需要恢复。虽然你可以自行决定多久 `commit` 一次，但对于这个练习，我建议在每个代码块之后 `commit` 你的代码（当然，附带描述性的提交消息！）。当你对编码和 Git `commit` 过程更加熟悉时，你会找到适合你的最佳代码备份频率和节奏。

加载不同的数据格式，正如我们在 [第四章](ch04.html#chapter4) 中将深入了解的那样，实际上可能是数据处理中更棘手的方面之一。幸运的是，有许多 Python 库可用来帮助，我们现在将使用其中之一！我在 [“库：从其他编程者那里借用自定义函数”](#library_love) 中简要提到了库；它们本质上是代码的食谱。对于这个项目，我们将使用 *csv* 库的“食谱”，它主要设计用于处理——你猜对了！——*.csv* 文件。文件扩展名 *.csv* 代表 *c*omma-*s*eparated *v*alue，如果你以前没有见过它，不要担心。我们将在 [第四章](ch04.html#chapter4) 中详细讨论文件类型（详细）。现在，有了 *csv* 库就意味着我们实际上不需要太多关于这种文件类型的知识来处理它，因为库的代码食谱将为我们完成大部分工作！

如果你在自己的文件中跟着做，你会想要将 [示例 2-16](#hitting_the_road_1) 中的代码添加到程序大纲中。

##### 示例 2-16\. hitting_the_road_with_citibike.py（续）

```py
# import the `csv` library ![1](assets/1.png)
import csv

# open the `202009CitibikeTripdataExample.csv` file in read ("r") mode
# this file should be in the same folder as our Python script or notebook
source_file = open("202009CitibikeTripdataExample.csv","r") ![2](assets/2.png)

# pass our `source_file` as an ingredient to the `csv` library's
# DictReader "recipe".
# store the result in a variable called `citibike_reader`
citibike_reader = csv.DictReader(source_file)

# the DictReader method has added some useful information to our data,
# like a `fieldnames` property that lets us access all the values
# in the first or "header" row
print(citibike_reader.fieldnames) ![3](assets/3.png)
```

[![1](assets/1.png)](#co_introduction_to_python_CO3-1)

*csv* 库包含一系列方便的代码食谱，用于处理我们的数据文件。

[![2](assets/2.png)](#co_introduction_to_python_CO3-2)

`open()` 是一个内置函数，它以文件名和“模式”作为参数。在这个例子中，目标文件（*202009CitibikeTripdataExample.csv*）应该与我们的 Python 脚本或笔记本在同一个文件夹中。对于“模式”，可以使用 `"r"` 表示“读取”或 `"w"` 表示“写入”。

[![3](assets/3.png)](#co_introduction_to_python_CO3-3)

通过打印出 `citibike_reader.fieldnames` 的值，我们可以看到“User Type”列的确切标签是 `usertype`。

此时，我们实际上可以运行此脚本，我们应该看到类似以下的输出：

```py
['tripduration', 'starttime', 'stoptime', 'start station id', 'start station
 name', 'start station latitude', 'start station longitude', 'end station id',
 'end station name', 'end station latitude', 'end station longitude', 'bikeid',
 'usertype', 'birth year', 'gender']
```

现在我们已经成功完成了大纲的第一步：我们已经读入了我们的数据。但我们还使用了 *csv* 库来转换数据，甚至生成了一些关于它的 *元数据*。重要的是，我们现在知道包含我们 “User Type” 信息的列的确切名称实际上是 `usertype`。这将有助于我们在编写 `if...else` 语句时的操作。为了确认一切按您的预期工作，请确保保存并运行您的代码。如果按预期工作（即，它打印出列标题的列表），现在是执行 `git commit` 循环的好时机：

```py
git status
git commit -m "*Commit message here*" *filename_with_extension*
git push

```

请记住，如果您正在使用 Google Colab，您可以通过选择文件 → “在 GitHub 中保存副本” 并在出现的覆盖窗口中输入提交消息，直接将您的代码提交到 GitHub。

现在我们已成功完成第一步，让我们继续进行第二步，如 [Example 2-17](#hitting_the_road_2) 所示。^([8](ch02.html#idm45143427725168))

##### 示例 2-17\. hitting_the_road_with_citibike.py（继续）

```py
# create a variable to hold the count of each type of Citi Bike user
# assign or "initialize" each with a value of zero (0)
subscriber_count = 0
customer_count = 0
other_user_count = 0
```

相当简单，对吧？继续进行第三步！因为我们需要检查文件中的每一行数据，所以我们需要编写一个 `for...in` 循环，其中将需要在 `usertype` 列中测试特定值的 `if...else`。为了帮助跟踪代码的每一行正在做什么，我将写很多以英文解释代码的注释，如 [Example 2-18](#hitting_the_road_3) 所示。

##### 示例 2-18\. hitting_the_road_with_citibike.py（继续进行）

```py
# step 3: loop through every row of our data
for a_row in citibike_reader: ![1](assets/1.png)

    # step 3a: if the value in the `usertype` column
    # of the current row is "Subscriber"
    if a_row["usertype"] == "Subscriber": ![2](assets/2.png)

        # add 1 to `subscriber_count`
        subscriber_count = subscriber_count +1

    # step 3b: otherwise (else), if the value in the `usertype` column
    # of the current row is "Customer"
    elif a_row["usertype"] == "Customer": ![3](assets/3.png)

        # add 1 to `subscriber_count`
        customer_count = customer_count + 1

    # step 3c: the `usertype` value is _neither_"Subscriber" nor "Customer",
    # so we'll add 1 to our catch-all `other_user_count` variable
    else: ![4](assets/4.png)
        other_user_count = other_user_count + 1
```

[![1](assets/1.png)](#co_introduction_to_python_CO4-1)

我们要确保我们的 `for` 循环正在处理已由我们的 DictReader 转换的数据，所以我们要确保在这里引用我们的 `citibike_reader` 变量。

[![2](assets/2.png)](#co_introduction_to_python_CO4-2)

为了让我的 `if` 语句“内嵌”到我的循环中，它们必须向右缩进一个 `tab`。

[![3](assets/3.png)](#co_introduction_to_python_CO4-3)

因为我们需要在这里使用 “else” —— 但也需要另一个 “if” 语句 —— 所以我们使用了复合关键字 `elif`，它是 “else if” 的简写。

[![4](assets/4.png)](#co_introduction_to_python_CO4-4)

这个最终的 `else` 将“捕获”任何数据行，其中 `usertype` 的值不是我们明确检查过的值之一（在本例中是“Subscriber”或“Customer”）。这充当了一个（非常）基本的数据质量检查：如果此数据列包含任何意外的值，`other_user_count` 将大于零 (`0`)，我们需要仔细查看原始数据。

好了，在[示例 2-18](#hitting_the_road_3)中有*很多*内容—或者看起来像是有很多！实际上，我们只是检查了每行的`usertype`列的值是否为`"Subscriber"`或`"Customer"`，如果是的话，我们将对应的`count`变量加一（或者*增加*）。如果`usertype`的值既不是这两者之一，我们将`other_user_count`变量加一。

虽然我们添加了比代码更多的行注释可能看起来有些奇怪，但实际上这是非常正常的—甚至是好的！毕竟，虽然计算机永远不会“忘记”如何读取Python代码，*你*如果不在注释中解释这段代码在做什么和为什么要这样做，*绝对*会忘记。而这并不是一件坏事！毕竟，如果要记住所有的代码，编程效率会大大降低。通过编写详细的注释，你确保将来能轻松理解你的代码，而不必再次将Python翻译成英语！

在我们继续之前，请确保运行你的代码。对于我们最常见的错误类型，“无消息就是好消息”，如果你没有收到任何错误，那就进行一次`git commit`循环。否则，现在是暂停并排除你遇到的任何问题的好时机。一旦解决了这些问题，你会发现只剩下一个非常简单的步骤：打印！让我们采用最直接的方式并使用内置的`print`语句，就像[示例 2-19](#hitting_the_road_4)中显示的那样。

##### 示例 2-19\. 使用 citibike.py 出发（我们到达了！）

```py
# step 4: print out our results, being sure to include "labels" in the process:
print("Number of subscribers:") ![1](assets/1.png)
print(subscriber_count)
print("Number of customers:")
print(customer_count)
print("Number of 'other' users:")
print(other_user_count)
```

[![1](assets/1.png)](#co_introduction_to_python_CO5-1)

请注意，这些`print()`语句是左对齐的，因为我们只想在`for`循环完成整个数据集的遍历后打印值。

当你把代码添加到[示例 2-19](#hitting_the_road_4)中后，保存并运行它。你的输出应该类似于这样：

```py
['tripduration', 'starttime', 'stoptime', 'start station id', 'start station
 name', 'start station latitude', 'start station longitude', 'end station id',
 'end station name', 'end station latitude', 'end station longitude', 'bikeid',
 'usertype', 'birth year', 'gender']
Number of subscribers:
58961
Number of customers:
17713
Number of 'other' users:
0
```

如果你看到这些内容—恭喜！你成功地编写了你的第一个真实世界数据整理程序。确保进行一次`git commit`来备份你的优秀工作！

## 寻求规模

在编写这个脚本时，我们完成了许多事情：

1.  我们成功并精确地统计了2020年9月某一天使用Citi Bikes的“订阅者”和“顾客”的数量。

1.  我们确认`usertype`列中没有其他值（因为我们的`other_user_count`变量的值为0）。

如果您之前使用电子表格或数据库程序进行数据整理，但这是您第一次在Python中工作，综合考虑所有事情，这个过程可能比您以前的方法花费更长时间。但正如我已经多次提到的，编码相比许多其他方法的一个关键优势是能够*无缝扩展*。这种扩展有两种方式。首先，它几乎可以在较大的数据集上以几乎与较小数据集相同的速度完成相同的工作。例如，在我的Chromebook上，*hitting_the_road_with_citibike.py*脚本大约在半秒内完成。如果我在*整个九月份*的数据上运行相同的脚本，大约需要12秒钟。大多数软件程序即使能够处理整个月份的数据，可能只需打开文件就需要这么长时间，更不用说对其进行任何工作了。因此，使用Python帮助我们扩展，因为我们可以更快速和有效地处理更大的数据集。您可以通过更改以下行来自行测试：

```py
source_file = open("202009CitibikeTripdataExample.csv","r")
```

至：

```py
source_file = open("202009-citibike-tripdata.csv","r")
```

如果您在独立的Python文件中工作，甚至可以通过在您的`python`命令之前添加`time`关键字来测量运行新文件上的脚本所需的时间：

```py
time python _your_filename_.py
```

这说明了我们通过Python实现的第二种类型的扩展，这与在不同数据集上（具有相同结构）计算机执行相同任务所需的*边际努力*有关。一旦我们编写了我们的程序，Python（以及一般的编程）真正闪耀的地方就在于此。为了在2020年9月的整个月份数据上运行我的脚本，*我所需要做的只是加载一个不同的数据文件*。通过更改我用`source_file =`语句打开的目标文件名，我能够处理整个九月份的数据，而不仅仅是一天的数据。换句话说，处理成千上万个额外数据行的额外（或“边际”）努力正好等于我复制和粘贴文件名所花费的时间。基于这个例子，我可以在几分钟内（或更短时间，正如我们将在[第8章](ch08.html#chapter8)中看到的那样）处理一整年的数据。这几乎是任何非编码数据整理方法都难以实现的事情。

我们在本节中构建的完整脚本是一个示例，展示了即使仅使用我们在本章中介绍的基本结构，您也可以使用Python进行一些非常有用和高效的数据整理。尽管在剩余的章节中有许多新的数据格式和挑战需要探索，但我希望这能让您感受到即使使用这些“基本”的Python工具和稍加努力和注意细节，您也能取得多大的成就。想象一下，如果您继续努力，还能实现什么！

# 结论

信不信由你，在本章中，我们涵盖了*所有*您进行数据整理所需的基本Python工具，以及几乎任何其他类型的Python编程！为了总结我们所学到的内容，我们学到了：

数据类型

这些是编程的“名词”：数字、字符串、列表、字典和布尔值。

函数

这些是编程的“动词”：运算符、内置函数和用户定义函数。

使用`for...in...`循环

这些让我们在列表中的每个项目上运行特定的代码块。

使用`if...else`条件语句

这些让我们根据数据的属性“决定”应该运行什么代码。

错误

我们探讨了编程时可能遇到的不同类型的错误，以及如何最好地处理和预防它们。

我们还练习了将这些概念结合和组合，以创建一个基本程序来处理来自纽约市自行车系统的一些样本数据。虽然我们将在未来的章节中扩展这个示例并探索其他示例，但我们下一个任务是更深入地了解如何作为数据整理工作的一部分评估数据本身。

^([1](ch02.html#idm45143423781840-marker)) 还有许多特定于语言的数据类型（如Python的*元组*数据类型），这些对我们的数据整理工作不是很相关，因此我们不会详细讨论这些。

^([2](ch02.html#idm45143426063952-marker)) 在许多其他编程语言中，花括号用于指示代码的哪些部分属于循环或条件语句。然而，正如我们在[“可读性”](ch01.html#readability)中提到的，Python是*依赖于空白*的。

^([3](ch02.html#idm45143428265504-marker)) 当然，与计算机不同，幼儿具有真正的独创性和学习能力。

^([4](ch02.html#idm45143428263280-marker)) 欲了解更多信息，请参阅Scott Rosenberg的Wordyard [博客文章](http://wordyard.com/2003/09/15/if-u-cn-rd-ths-msg-u-r-jst-lke-vryne-lse)。

^([5](ch02.html#idm45143428086384-marker)) 这实际上是一件好事，正如我们将在[第3章](ch03.html#chapter3)中看到的那样。

^([6](ch02.html#idm45143428081024-marker)) 关于代码覆盖率最佳实践的信息，请参阅[Google测试博客](https://testing.googleblog.com/2020/08/code-coverage-best-practices.html)上的“代码覆盖率最佳实践”。

^([7](ch02.html#idm45143427916368-marker)) 此练习的所有代码都可以在*hitting_the_road_with_citibike.py*和*hitting_the_road_with_citibike.ipynb*文件中找到。然而，我强烈建议您创建自己的新文件，并手动输入提供的代码。

^([8](ch02.html#idm45143427725168-marker)) 再次强调——从现在开始——将此代码添加到您的文件中，放在上一个代码块的最后一行下面。
