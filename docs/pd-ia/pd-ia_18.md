# 附录 B. Python 入门教程

pandas 库建立在 Python 之上，Python 是一种流行的编程语言，由荷兰开发者 Guido van Rossum 于 1991 年首次发布。*库*（也称为*包*）是一组功能工具箱，它扩展了编程语言的核心功能。库通过提供解决方案来加速开发者的生产力，例如数据库连接、代码质量和测试。大多数 Python 项目都使用库。毕竟，如果有人已经解决了问题，为什么还要从头开始解决问题呢？Python 包索引（PyPi）是一个集中在线仓库，有超过 300,000 个库可供下载。Pandas 是这 300,000 个库之一；它实现了复杂的数据结构，擅长存储和操作多维数据。在我们探索 pandas 为 Python 添加了什么之前，了解基础语言中有什么是很重要的。

Python 是一种面向对象编程（OOP）语言。面向对象范式将软件程序视为一组相互通信的对象。*对象*是一个数字数据结构，用于存储信息并提供访问和操作该信息的方式。每个对象都有其存在的原因或目的。我们可以将每个对象想象成戏剧中的一个演员，而软件程序则是一场表演。

将对象视为数字构建块是一种有用的思考方式。以电子表格软件如 Excel 为例。作为用户，我们可以区分工作簿、工作表和单元格之间的差异。工作簿包含工作表，工作表包含单元格，单元格包含值。我们将这三个实体视为三个不同的业务逻辑容器，每个容器都有指定的职责，并且我们以不同的方式与之交互。当构建面向对象的计算机程序时，开发者会以同样的方式思考，识别并构建程序运行所需的“块”。

在 Python 社区中，你经常会听到“万物皆对象”的表达。这个说法意味着该语言实现了所有数据类型，即使是像数字和文本这样的简单类型，也作为对象实现。像 pandas 这样的库添加了新的对象集合——一组额外的构建块——到语言中。

作为一名从数据分析师转变为软件工程师的人，我见证了行业内许多角色的 Python 熟练度要求。我可以从经验中提出，你不需要成为一名高级程序员就能有效地使用 pandas。然而，对 Python 核心机制的基本理解将显著加快你掌握该库的速度。本附录强调了你需要了解的关键语言要素，以便取得成功。

## B.1 简单数据类型

数据有多种类型。例如，整数 5 与十进制数 8.46 的类型不同。5 和 8.46 都与文本值`"Bob"`不同。

让我们从探索 Python 中内置的核心数据类型开始。确保您已安装 Anaconda 发行版，并设置了一个包含 Jupyter Notebook 编码环境的 `conda` 环境。如果您需要帮助，请参阅附录 A 中的安装说明。激活为本书创建的 `conda` 环境，执行命令 `jupyter notebook`，并创建一个新的 Notebook。

在我们开始之前的一个快速提示：在 Python 中，井号符号（`#`）创建一个注释。一个*注释*是 Python 在处理代码时忽略的文本行。开发者使用注释为他们的代码提供内联文档。以下是一个示例：

```
# Adds two numbers together
1 + 1
```

我们也可以在一段代码后面添加注释。Python 会忽略井号符号之后的所有内容。该行的其余部分会正常执行：

```
1 + 1 # Adds two numbers together
```

虽然前面的例子计算结果为 2，但下一个例子不会产生任何输出。注释有效地禁用了该行，因此 Python 忽略了加法操作：

```
# 1 + 1
```

我在本书中的代码单元中使用了注释，以提供对当前操作补充说明。您不需要将注释复制到您的 Jupyter Notebook 中。

### B.1.1 数字

一个*整数*是一个整数；它没有分数或小数部分。20 是一个例子：

```
In  [1] 20

Out [1] 20
```

一个整数可以是任何正数、负数或零。负数前面有一个负号（`-`）：

```
In  [2] -13

Out [2] -13
```

一个*浮点数*（俗称*浮点*）是一个带有分数或小数部分的数字。我们使用点来声明小数点。7.349 是一个浮点数的例子：

```
In  [3] 7.349

Out [3] 7.349
```

整数和浮点数在 Python 中代表不同的数据类型，或者说代表不同的对象。通过查找小数点的存在来区分这两个。例如，`5.0` 是一个浮点对象，而 `5` 是一个整数对象。

### B.1.2 字符串

一个*字符串*是由零个或多个文本字符组成的集合。我们通过将一段文本包裹在单引号、双引号或三重引号中来声明一个字符串。这三个选项之间有区别，但对于初学者来说并不重要。本书中我们将坚持使用双引号。Jupyter Notebook 对三种语法选项的输出是相同的：

```
In  [4] 'Good morning'

Out [4] 'Good morning'

In  [5] "Good afternoon"

Out [5] 'Good afternoon'

In  [6] """Good night"""

Out [6] 'Good night'
```

字符串不仅限于字母字符；它们可以包括数字、空格和符号。考虑下一个例子，它包括七个字母字符、一个美元符号、两个数字、一个空格和一个感叹号：

```
In  [7] "$15 dollars!"

Out [7] '$15 dollars!'
```

通过引号的存在来识别字符串。许多初学者对像 `"5"` 这样的值感到困惑，这是一个包含单个数字字符的字符串。`"5"` 不是一个整数。

一个*空字符串*没有字符。我们通过在两个引号之间不放置任何内容来创建它：

```
In  [8] ""

Out [8] ''
```

字符串的长度指的是其字符的数量。例如，字符串 `"Monkey` `business"` 的长度为 15 个字符；其中 `Monkey` 有六个字符，`business` 有八个字符，两个单词之间有一个空格。

Python 根据每个字符串字符在行中的顺序为其分配一个数字。这个数字称为*索引*，它从 0 开始计数。在字符串 `"car"` 中，

+   `"c"` 位于索引位置 0。

+   `"a"` 位于索引位置 1。

+   `"r"` 位于索引位置 2。

字符串的最后一个索引位置总是比其长度少 1。字符串 `"car"` 的长度为 3，因此其最后一个索引位置是 2。基于 0 的索引可能会让新开发者感到困惑；这是一个难以进行的心智转变，因为我们从小学就被教导从 1 开始计数。

我们可以通过索引位置提取字符串中的任何字符。在字符串后，输入一对带有索引值的方括号。下一个示例提取了 `"Python"` 中的 `"h"` 字符。`"h"` 字符是序列中的第四个字符，因此它的索引是 3：

```
In  [9] "Python"[3]

Out [9] 'h'
```

要从字符串的末尾提取，在方括号内提供一个负值。`-1` 的值提取最后一个字符，`-2` 提取倒数第二个字符，依此类推。下一个示例针对 Python 中的第四个倒数字符，即 `"t"`：

```
In  [10] "Python"[-4]

Out [10] 't'
```

在前面的示例中，`"Python"[2]` 会产生相同的 `"t"` 输出。

我们可以使用特殊的语法从字符串中提取多个字符。这个过程称为*切片*。在方括号内放置两个数字，用冒号分隔。左侧的值设置起始索引。右侧的值设置最终索引。起始索引是包含的；Python 包含该索引处的字符。结束索引是不包含的；Python 不包含该索引处的字符。我知道这很棘手。

下一个示例从索引位置 2（包含）到索引位置 5（不包含）提取所有字符。切片包括索引位置 2 的 `"t"`、索引位置 3 的 `"h"` 和索引位置 4 的 `"o"`：

```
In  [11] "Python"[2:5]

Out [11] 'tho'
```

如果 0 是起始索引，我们可以从方括号中移除它并得到相同的结果。选择最适合你的语法选项：

```
In  [12] # The two lines below are equivalent
         "Python"[0:4]
         "Python"[:4]

Out [12] 'Pyth'
```

这里还有一个快捷方式：要从一个索引提取到字符串的末尾，移除结束索引。以下示例显示了两种从 `"h"`（索引 3）提取到 `"Python"` 字符串末尾字符的方法：

```
In  [13] # The two lines below are equivalent
         "Python"[3:6]
         "Python"[3:]

Out [13] 'hon'
```

我们还可以移除两个数字。单个冒号告诉 Python “从开头到结尾。”结果是字符串的副本：

```
In  [14] "Python"[:]

Out [14] 'Python'
```

我们可以在字符串切片中混合使用正索引和负索引位置。让我们从索引 1（`"y"`) 提取到最后一个字符（`"n"`）：

```
In  [15] "Python"[1:-1]

Out [15] 'ytho'
```

我们还可以传递一个可选的第三个数字来设置*步长间隔*——两个索引位置之间的间隔。下一个示例以 2 的间隔从索引位置 0（包含）到 6（不包含）提取字符。这个切片包括索引位置 0 的 `"P"`、索引位置 2 的 `"t"` 和索引位置 4 的 `"o"`：

```
In  [16] "Python"[0:6:2]

Out [16] 'Pto'
```

这里有一个酷技巧：我们可以将-1 作为第三个数字传递，以便从列表的末尾向前推进到开头。结果是反转的字符串：

```
In  [17] "Python"[::-1]

Out [17] 'nohtyP'
```

切片对于从较大的字符串中提取文本片段非常有用——这是我们在第六章中详细讨论的主题。

### B.1.3 布尔

*布尔* 数据类型代表逻辑上的真值概念。它只能有两个值之一：`True` 或 `False`。布尔数据类型是以英国数学家和哲学家乔治·布尔的名字命名的。它通常表示一种非此即彼的关系：是或否，开或关，有效或无效，活跃或非活跃，等等。

```
In  [18] True

Out [18] True

In  [19] False

Out [19] False
```

我们通常通过计算或比较得到布尔数据类型，我们将在 B.2.2 节中看到。

### B.1.4 `None` 对象

`None` 对象代表无或值的缺失。与布尔类型一样，这是一个难以理解的概念，因为它比整数等具体值更抽象。

假设我们决定测量我们城镇一周的每日气温，但忘记了在星期五进行测量。七天的气温中有六天是整数。我们如何记录缺失的那一天的气温？我们可能会输入“缺失”、“未知”或“null”。在 Python 中，`None` 对象模拟了同样的概念。语言需要某种东西来传达值的缺失。它需要一个对象来代表并宣布值是缺失的、不存在的或不需要的。当我们执行包含 `None` 的单元格时，Jupyter Notebook 不会输出任何内容：

```
In  [20] None
```

与布尔类型一样，我们通常会得到一个 `None` 值，而不是手动创建它。随着我们阅读本书，我们将更详细地探讨该对象。

## B.2 运算符

*运算符* 是执行操作的符号。一个经典的例子来自小学的加法运算符：加号 (+)。运算符作用的值称为 *操作数*。在表达式 3 + 5 中，

+   + 是运算符。

+   3 和 5 是操作数。

在本节中，我们将探讨 Python 中内置的各种数学和逻辑运算符。

### B.2.1 数学运算符

让我们写出介绍中的数学表达式。Jupyter 将直接在单元格下方输出计算结果：

```
In  [21] 3 + 5

Out [21] 8
```

在运算符两侧添加空格是一种惯例，可以使代码更容易阅读。接下来的两个示例说明了减法 (`-`) 和乘法 (`*`)：

```
In  [22] 3 - 5

Out [22] -2

In  [23] 3 * 5

Out [23] 15
```

`**` 是指数运算符。下一个示例将 3 提到 5 次幂（3 自身乘以 5 次）：

```
In  [24] 3 ** 5

Out [24] 243
```

`/` 符号执行除法。下一个示例将 3 除以 5：

```
In  [25] 3 / 5

Out [25] 0.6
```

在数学术语中，*商* 是一个数除以另一个数的结果。使用 `/` 运算符的除法始终返回一个浮点数商，即使除数可以整除被除数：

```
In  [26] 18 / 6

Out [26] 3.0
```

*地板除法* 是一种替代的除法类型，它会从商中移除小数余数。它需要两个正斜杠 (`//`)，并返回一个整数商。下一个示例演示了这两个运算符之间的区别：

```
In  [27] 8 / 3

Out [27] 2.6666666666666665

In  [28] 8 // 3

Out [28] 2
```

*取模* 运算符 (`%`) 返回除法的结果余数。当 5 除以 3 时，2 是余数：

```
In  [29] 5 % 3

Out [29] 2
```

我们还可以使用加法和乘法运算符与字符串。加号用于连接两个字符串。这个过程的技术术语是 *连接*。

```
In  [30] "race" + "car"

Out [30] 'racecar'
```

乘号会重复字符串给定次数：

```
In  [31] "Mahi" * 2

Out [31] 'MahiMahi'
```

对象的类型决定了它支持的运算符和操作。例如，我们可以除以整数，但不能除以字符串。面向对象编程的主要技能是识别你正在处理的对象以及它可以执行的操作。

我们可以将一个字符串连接到另一个字符串，我们也可以将一个数字加到另一个数字上。但当我们尝试将一个字符串和一个数字相加时会发生什么？

```
In  [32] 3 + "5"

---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-9-d4e36ca990f8> in <module>
----> 1 3 + "5"

TypeError: unsupported operand type(s) for +: 'int' and 'str'
```

哎呀。这是我们第一次接触到 Python 错误——语言中内置的几十种错误之一。错误的技术名称是 *异常*。像 Python 中的其他一切一样，异常也是一个对象。每当我们在语法或逻辑上犯错误时，Jupyter Notebook 会显示一个分析，其中包括错误的名称和触发它的行号。技术术语 *raise* 常用来表示 Python 遇到了异常。我们可以这样说：“我尝试将一个数字和一个字符串相加，Python 抛出了一个异常。”

当我们在操作中使用错误的数据类型时，Python 会抛出一个 `TypeError` 异常。在上面的示例中，Python 观察到一个数字和一个加号，并假设将跟随另一个数字。然而，它接收到一个字符串，它无法将字符串添加到整数中。我们将在 B.4.1 节中看到如何将整数转换为字符串（反之亦然）。

### B.2.2 等于和不等于运算符

Python 认为两个对象相等，如果它们持有相同的值。我们可以通过将它们放在等于运算符的两侧来比较两个对象的相等性（`==`）。如果两个对象相等，则运算符返回 `True`。提醒一下，`True` 是一个布尔值。

```
In  [33] 10 == 10

Out [33] True
```

小心：等于运算符有两个等号。Python 为一个完全不同的操作保留了单个等号，我们将在 B.3 节中介绍。

如果两个对象不相等，等于运算符返回 `False`。`True` 和 `False` 是布尔值的有效值：

```
In  [34] 10 == 20

Out [34] False
```

这里有一些等于运算符与字符串的示例：

```
In  [35] "Hello" == "Hello"

Out [35] True

In  [36] "Hello" == "Goodbye"

Out [36] False
```

在比较两个字符串时，大小写敏感很重要。在下一个示例中，一个字符串以大写 `"H"` 开头，而另一个以小写 `"h"` 开头，因此 Python 认为这两个字符串不相等：

```
In  [37] "Hello" == "hello"

Out [37] False
```

不等于运算符（`!=`）是等于运算符的逆运算；如果两个对象不相等，则返回 `True`。例如，10 不等于 20：

```
In  [38] 10 != 20

Out [38] True
```

同样，字符串 `"Hello"` 不等于字符串 `"Goodbye"`：

```
In  [39] "Hello" != "Goodbye"

Out [39] True
```

如果两个对象相等，不等于运算符返回 `False`：

```
In  [40] 10 != 10

Out [40] False

In  [41] "Hello" != "Hello"

Out [41] False
```

Python 支持数字之间的数学比较。`<` 运算符检查左侧的操作数是否小于右侧的操作数。以下示例检查 -5 是否小于 3：

```
In  [42] -5 < 3

Out [42] True
```

`>`运算符检查左侧的操作数是否大于右侧的操作数。下一个示例评估 5 是否大于 7；结果是`False`。

```
In  [43] 5 > 7

Out [43] False
```

`<=`操作符检查左侧操作数是否小于或等于右侧操作数。在这里，我们检查 11 是否小于或等于 11：

```
In  [44] 11 <= 11

Out [44] True
```

相补的`>=`操作符检查左侧操作数是否大于或等于右侧操作数。下一个示例检查 4 是否大于或等于 5：

```
In  [45] 4 >= 5

Out [45] False
```

Pandas 使我们能够将这些比较应用于整个数据列，这是我们在第五章中讨论的主题。

## B.3 变量

*变量*是我们分配给对象的名称；我们可以将其与房屋地址进行比较，因为它是标签、引用和标识符。变量名应该是清晰且描述性的，描述对象存储的数据以及它在我们的应用程序中扮演的用途。例如，`revenues_for_quarter4`比`r`或`r4`更好的变量名。

我们使用赋值运算符（单个等号`=`）将变量赋给对象。下一个示例将四个变量（`name`、`age`、`high_school_gpa`和`is_handsome`）赋给四种不同的数据类型（字符串、整数、浮点数和布尔值）：

```
In  [46] name = "Boris"
         age = 28
         high_school_gpa = 3.7
         is_handsome = True
```

在 Jupyter Notebook 中，对带有变量赋值的单元格执行不会产生任何输出，但之后我们可以在笔记本中的任何单元格中使用该变量。变量是它所持有值的替代品：

```
In  [47] name

Out [47] 'Boris'
```

变量名必须以字母或下划线开头。在第一个字母之后，它只能包含字母、数字或下划线。

正如它们的名称所暗示的，变量可以存储在程序执行过程中变化的值。让我们将`age`变量重新赋值为新的值`35`。在我们执行单元格之后，`age`变量对其先前值`28`的引用将丢失：

```
In  [48] age = 35
         age

Out [48] 35
```

我们可以在赋值运算符的两侧使用相同的变量。Python 始终首先评估等号右侧的值。在下一个示例中，Python 将单元格执行开始时`age`的值`35`加到`10`上。得到的总和`45`被保存到`age`变量中：

```
In  [49] age = age + 10
         age

Out [49] 45
```

Python 是一种*动态类型*语言，这意味着变量对数据类型一无所知。变量是程序中任何对象的占位符名称。只有对象知道它的数据类型。因此，我们可以将变量从一种类型重新赋值到另一种类型。下一个示例将`high_school_gpa`变量从其原始的浮点值`3.7`重新赋值为字符串`"A+"`：

```
In  [50] high_school_gpa = "A+"
```

当程序中不存在变量时，Python 会引发`NameError`异常：

```
In  [51] last_name

---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
<ipython-input-5-e1aeda7b4fde> in <module>
----> 1 last_name

NameError: name 'last_name' is not defined
```

当你误拼变量名时，通常会遇到`NameError`异常。这种异常不必害怕；更正拼写，然后再次执行单元格。

## B.4 函数

一个 *函数* 是由一个或多个步骤组成的程序。将函数想象成编程语言中的烹饪食谱——一系列产生一致结果的指令。函数使软件具有可重用性。因为函数从开始到结束捕获了一部分业务逻辑，所以当我们需要多次执行相同的操作时，我们可以重用它。

我们声明一个函数然后执行它。在声明中，我们写下函数应该采取的步骤。在执行中，我们运行函数。按照我们的烹饪类比，声明一个函数相当于写下食谱，执行一个函数相当于烹饪食谱。执行函数的技术术语是 *调用* 或 *调用*。

### B.4.1 参数和返回值

Python 随带提供了超过 65 个内置函数。我们也可以声明我们自己的自定义函数。让我们深入一个例子。内置的 `len` 函数返回给定对象的长度。长度的概念因数据类型而异；对于字符串，它是字符的计数。

我们通过输入函数名和一对开闭括号来调用一个函数。就像烹饪食谱可以接受配料一样，函数调用可以接受称为 *参数* 的输入。我们按顺序在括号内传递参数，参数之间用逗号分隔。

`len` 函数期望一个参数：它应该计算长度的对象。下一个示例将 `"Python` `is` `fun"` 字符串参数传递给函数：

```
In  [52] len("Python is fun")

Out [52] 13
```

烹饪食谱产生最终输出——一顿饭。同样，Python 函数产生一个称为 *返回值* 的最终输出。在上一个示例中，`len` 是被调用的函数，`"Python` `is` `fun"` 是它的单个参数，`13` 是返回值。

就这些了！函数是一个可以调用零个或多个参数并产生返回值的程序。

这里是 Python 中三个更受欢迎的内置函数：

+   `int`，它将它的参数转换为整数

+   `float`，它将它的参数转换为浮点数

+   `str`，它将它的参数转换为字符串

下三个示例展示了这些函数的实际应用。第一个示例使用 `"20"` 字符串参数调用 `int` 函数，并产生返回值 `20`。你能识别剩余两个函数的参数和返回值吗？

```
In  [53] int("20")

Out [53] 20

In  [54] float("14.3")

Out [54] 14.3

In  [55] str(5)

Out [55] '5'
```

这里还有一个常见的错误：当函数接收到正确数据类型但不适用的值时，Python 会抛出一个 `ValueError` 异常。在下一个示例中，`int` 函数接收了一个字符串（一个合适的类型），但这个字符串无法从中提取出整数：

```
In  [56] int("xyz")

---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-6-ed77017b9e49> in <module>
----> 1 int("xyz")

ValueError: invalid literal for int() with base 10: 'xyz'
```

另一个流行的内置函数是 `print`，它将文本输出到屏幕。它接受任意数量的参数。当我们在程序执行过程中想要观察变量的值时，该函数通常非常有用。下一个示例四次调用 `print` 函数，使用 `value` 变量，其值变化了几次：

```
In  [57] value = 10
         print(value)

         value = value - 3
         print(value)

         value = value * 4
         print(value)

         value = value / 2
         print(value)

Out [57] 10
         7
         28
         14.0
```

如果一个函数接受多个参数，我们必须用逗号分隔每两个后续参数。开发者经常在逗号后添加一个空格以提高可读性。

当我们向 `print` 函数传递多个参数时，它会按顺序输出所有参数。在下一个示例中，请注意 Python 使用空格分隔三个打印的元素：

```
In  [58] print("Cherry", "Strawberry", "Key Lime")

Out [58] Cherry Strawberry Key Lime
```

**参数**是为预期函数参数赋予的名称。调用中的每个参数都对应一个参数。在之前的示例中，我们向 `print` 函数传递了参数，但没有指定它们的参数。

对于某些参数，我们必须明确写出参数名称。例如，`print` 函数的 `sep`（分隔符）参数自定义 Python 在每个打印值之间插入的字符串。如果我们想传递一个自定义参数，我们必须明确写出 `sep` 参数。我们使用等号将参数分配给函数的关键字参数。下一个示例输出相同的三个字符串，但指示 `print` 函数用感叹号分隔它们：

```
In  [59] print("Cherry", "Strawberry", "Key Lime", sep = "!")

Out [59] Cherry!Strawberry!Key Lime
```

让我们回到上一个示例之前。为什么三个值之间用空格隔开？

默认参数是一个后备值，当函数调用没有明确提供一个值时，Python 会将它传递给参数。`print` 函数的 `sep` 参数有一个默认参数 `" "`。如果我们调用 `print` 函数而没有为 `sep` 参数提供参数，Python 将自动传递一个包含一个空格的字符串。以下两行代码产生相同的输出：

```
In  [60] # The two lines below are equivalent
         print("Cherry", "Strawberry", "Key Lime")
         print("Cherry", "Strawberry", "Key Lime", sep=" ")

Out [60] Cherry Strawberry Key Lime
         Cherry Strawberry Key Lime
```

我们称像 `sep` 这样的参数为**关键字参数**。在传递参数时，我们必须写出它们的特定参数名称。Python 要求我们在传递顺序参数之后传递关键字参数。以下是一个 `print` 函数调用的另一个示例，它向 `sep` 参数传递了不同的字符串参数：

```
In  [61] print("Cherry", "Strawberry", "Key Lime", sep="*!*")

Out [61] Cherry*!*Strawberry*!*Key Lime
```

`print` 函数的 `end` 参数自定义 Python 添加到所有输出末尾的字符串。该参数的默认参数是 `"\n"`，这是一个 Python 识别为换行符的特殊字符。在下一个示例中，我们明确地将相同的 `"\n"` 参数传递给 `end` 参数：

```
In  [62] print("Cherry", "Strawberry", "Key Lime", end="\n")
         print("Peach Cobbler")

Out [62] Cherry Strawberry Key Lime
         Peach Cobbler
```

我们可以在函数调用中传递多个关键字参数。技术规则仍然适用：用逗号分隔每两个参数。下一个示例两次调用了 `print` 函数。第一次调用用 `"!"` 分隔其三个参数，并以 `"***"` 结束输出。因为第一次调用没有强制换行，所以第二次调用的输出从第一个调用结束的地方继续：

```
In  [63] print("Cherry", "Strawberry", "Key Lime", sep="!", end="***")
         print("Peach Cobbler")

Out [63] Cherry!Strawberry!Key Lime***Peach Cobbler
```

请花点时间思考一下前面示例中的代码格式。长行代码可能难以阅读，尤其是当我们把多个参数放在一起时。Python 社区倾向于几种格式化解决方案。一个选项是将所有参数放在单独的一行上：

```
In  [64] print(
             "Cherry", "Strawberry", "Key Lime", sep="!", end="***"
         )

Out [64] Cherry!Strawberry!Key Lime***
```

另一个选项是在参数之间添加换行符：

```
In  [65] print(
             "Cherry",
             "Strawberry",
             "Key Lime",
             sep="!",
             end="***",
         )

Out [65] Cherry!Strawberry!Key Lime***
```

这三个代码示例在技术上都是有效的。Python 代码的格式化有多种方式。我在整本书中使用了多种格式化选项。我的最终目标是可读性。你不必遵循我使用的格式化约定。我会尽我所能说明哪些差异是技术性的，哪些是美学的。

### B.4.2 自定义函数

我们可以在程序中声明自定义函数。函数的目标是将一个独特的业务逻辑捕获在单一、可重用的过程中。软件工程领域的一个常见格言是 *DRY*，它是 *不要重复自己* 的缩写。这个缩写是一个警告，表明重复相同的逻辑或行为可能导致程序不稳定。你重复代码的地方越多，如果需求发生变化，你需要编辑的地方就越多。函数解决了 DRY 问题。

让我们来看一个示例。假设我们是气象学家，正在处理天气数据。我们的工作要求我们在程序中将温度从华氏度转换为摄氏度。转换有一个简单、一致的公式。编写一个函数来将 *一个* 温度从华氏度转换为摄氏度是个好主意，因为我们可以隔离转换逻辑并在需要时重复使用它。

我们用一个 `def` 关键字开始函数定义。我们在 `def` 后面跟函数的名称，一对开括号和闭括号，以及一个冒号。多单词的函数名和变量名遵循 `snake_case` 命名约定。这个约定将每两个单词用下划线分隔，使得名称看起来像蛇。让我们称我们的函数为 `convert_to_fahrenheit`：

```
def convert_to_fahrenheit():
```

为了复习，一个 *参数* 是一个预期函数参数的名称。我们希望 `convert_to_fahrenheit` 函数接受一个单一参数：摄氏温度。让我们称这个参数为 `celsius_temp`：

```
def convert_to_fahrenheit(celsius_temp):
```

如果我们在声明函数时定义了一个参数，那么在调用它时必须为该参数传递一个参数。因此，每次运行 `convert_to_fahrenheit` 时，我们都必须为 `celsius_temp` 提供一个值。

我们下一步是定义函数的功能。我们在函数体中声明函数的步骤，这是位于其名称下方的一个缩进代码部分。Python 使用缩进来建立程序中构造之间的关系。函数体是 *块* 的一个例子，它是嵌套在另一个代码部分中的代码段。根据 PEP-8¹，Python 社区的风格指南，我们应该使用四个空格来缩进块中的每一行：

```
def convert_to_fahrenheit(celsius_temp):
    # This indented line belongs to the function
    # So does this indented line

# This line is not indented, so it does not belong to convert_to_fahrenheit
```

我们可以在函数体中使用函数的参数。在我们的例子中，我们可以在 `convert_to_fahrenheit` 函数的任何地方使用 `celsius_temp` 参数。

我们可以在函数体中声明变量。这些变量被称为 *局部变量*，因为它们绑定到函数执行的范围内。Python 在函数运行完成后立即将局部变量从内存中移除。

让我们写出转换的逻辑！将摄氏温度转换为华氏温度的公式是将它乘以 9/5 并加 32：

```
def convert_to_fahrenheit(celsius_temp):
    first_step = celsius_temp * (9 / 5)
    fahrenheit_temperature = first_step + 32
```

在这个阶段，我们的函数正确地计算了华氏温度，但它并没有将评估结果发送回主程序。我们需要使用 `return` 关键字来标记华氏温度为函数的最终输出。我们将它返回到外部世界：

```
In  [66] def convert_to_fahrenheit(celsius_temp):
             first_step = celsius_temp * (9 / 5)
             fahrenheit_temperature = first_step + 32
             return fahrenheit_temperature
```

我们的功能已经完成，现在让我们来测试它！我们使用一对括号来调用自定义函数，这与我们用于 Python 内置函数的语法相同。下一个示例使用 `10` 作为样本参数调用了 `convert_to_fahrenheit` 函数。Python 将 `celsius_temp` 参数设置为 `10` 并运行函数体。该函数返回值为 `50.0`：

```
In  [67] convert_to_fahrenheit(10)

Out [67] 50.0
```

我们可以提供关键字参数而不是位置参数。下一个示例明确写出了 `celsius_temp` 参数的名称。以下代码与前面的代码等效：

```
In  [68] convert_to_fahrenheit(celsius_temp = 10)

Out [68] 50.0
```

虽然它们不是必需的，但关键字参数有助于使我们的程序更清晰。前面的示例更好地说明了 `convert_to_fahrenheit` 函数的输入代表什么。

## B.5 模块

一个 *模块* 是一个单独的 Python 文件。Python 的 *标准库* 是一个包含超过 250 个模块的语言集合，这些模块内置到语言中以加速生产力。这些模块帮助进行技术操作，如数学、音频分析和 URL 请求。为了减少程序的内存消耗，Python 默认不会加载这些模块。当我们的程序需要时，我们必须手动导入我们想要的特定模块。

导入内置模块和外部包的语法是相同的：输入 `import` 关键字，然后是模块或包的名称。让我们导入 Python 的 `datetime` 模块，它帮助我们处理日期和时间：

```
In  [69] import datetime
```

*别名* 是导入的替代名称——一个我们可以分配给模块的快捷方式，这样在引用它时就不必写出其完整名称。别名实际上取决于我们，但某些昵称已经在 Python 开发者中确立了自己作为最受欢迎的。例如，`datetime` 模块的流行别名是 `dt`。我们使用 `as` 关键字来分配别名：

```
In  [70] import datetime as dt
```

现在我们可以用 `dt` 而不是 `datetime` 来引用模块。

## B.6 类和对象

我们迄今为止探索的所有数据类型——整数、浮点数、布尔值、字符串、异常、函数，甚至模块——都是对象。*对象* 是一种数字数据结构，用于存储、访问和操作一种类型的数据。

*类* 是创建对象的蓝图。将其视为一个图表或模板，Python 从中构建对象。

我们称从类构建的对象为该类的 *实例*。从类创建对象的行为称为 *实例化*。

Python 的内置 `type` 函数返回我们作为参数传递给它的对象的类。下一个例子两次调用 `type` 函数，使用两个不同的字符串：`"peanut butter"` 和 `"jelly"`。尽管它们的内容不同，但这两个字符串是由相同的蓝图、相同的类、`str` 类构建的。它们都是字符串：

```
In  [71] type("peanut butter")

Out [71] str

In  [72] type("jelly")

Out [72] str
```

这些例子相当简单。当我们不确定正在处理什么类型的对象时，`type` 函数很有帮助。如果我们调用一个自定义函数并且不确定它返回什么类型的对象，我们可以将它的返回值传递给 `type` 来找出。

*字面量* 是创建从类创建对象的简写语法。我们迄今为止遇到的一个例子是双引号，它创建字符串（`"hello"`）。对于更复杂的对象，我们需要使用不同的创建过程。

在 B.5 节中导入的 `datetime` 模块有一个 `date` 类，它模拟时间中的日期。假设我们正在尝试将列奥纳多·达·芬奇的生日，1452 年 4 月 15 日，表示为一个 `date` 对象。

要从类创建一个实例，写上类名后跟一对括号。例如，`date()` 创建了一个来自 `date` 类的 `date` 对象。语法与调用函数相同。在实例化对象时，我们有时可以向构造函数传递参数，即创建对象的函数。`date` 构造函数的前三个参数代表 `date` 对象将包含的年、月和日。这三个参数是必需的：

```
In  [73] da_vinci_birthday = dt.date(1452, 4, 15)
         da_vinci_birthday

Out [73] datetime.date(1452, 4, 15)
```

现在我们有一个 `da_vinci_birthday` 变量，它包含一个代表 1452 年 4 月 15 日的 `date` 对象。

## B.7 属性和方法

*属性* 是属于对象、特征或细节的内部数据片段，它揭示了关于对象的信息。我们使用点符号来访问对象的属性。一个 `date` 对象上的三个示例属性是 `day`、`month` 和 `year`：

```
In  [74] da_vinci_birthday.day

Out [74] 15

In  [75] da_vinci_birthday.month

Out [75] 4

In  [76] da_vinci_birthday.year

Out [76] 1452
```

*方法*是我们可以向对象发出的动作或命令。将方法视为属于对象的函数。*属性*构成了对象的状态，而方法代表了对象的行为。像函数一样，方法可以接受参数并产生返回值。

我们在方法名称后面使用一对括号来调用方法。确保在对象和方法名称之间添加一个点。`date`对象的一个示例方法是一个`weekday`。`weekday`方法返回日期的星期几作为整数。`0`表示星期日，`6`表示星期六：

```
In  [77] da_vinci_birthday.weekday()

Out [77] 3
```

莱昂纳多出生于星期三！

`weekday`等方法的简便性和可重用性是`date`对象存在的原因。想象一下，如果用文本字符串来模拟日期逻辑会有多困难。想象一下，如果每个开发者都构建他们自己的定制解决方案。哎呀。Python 的开发者预计用户将需要处理日期，因此他们构建了一个可重用的`date`类来模拟这个现实世界的结构。

关键要点是 Python 标准库为开发者提供了许多实用类和函数来解决常见问题。然而，随着程序复杂性的增加，仅使用 Python 的核心对象来模拟现实世界思想变得困难。为了解决这个问题，开发者向语言中添加自定义对象。这些对象模拟特定领域的业务逻辑。开发者将这些对象打包成库。这就是 pandas 的全部：一组用于解决数据分析领域特定问题的额外类。

## B.8 字符串方法

字符串对象有一套自己的方法。这里有一些例子。

`upper`方法返回一个所有字符都为大写的新的字符串：

```
In  [78] "Hello".upper()

Out [78] "HELLO"
```

我们可以在变量上调用方法。回想一下，*变量*是对象的占位符名称。Python 会将变量替换为它引用的对象。下一个示例在`greeting`变量引用的字符串上调用`upper`方法。输出与前面的代码示例相同：

```
In  [79] greeting = "Hello"
         greeting.upper()

Out [79] "HELLO"
```

有两种类型的对象：可变和不可变。*可变*对象可以改变。*不可变*对象不能改变。字符串、数字和布尔值是不可变对象的例子；我们创建后不能修改它们。字符串`"Hello"`始终是字符串`"Hello"`。数字 5 始终是数字 5。

在前面的例子中，`upper`方法调用没有修改分配给`greeting`变量的原始`"Hello"`字符串。相反，方法调用返回了一个所有字母都为大写的新的字符串。我们可以输出`greeting`变量来确认字符保留了原始的大小写：

```
In  [80] greeting

Out [80] 'Hello'
```

字符串是不可变的，所以它的方法不会修改原始对象。我们将在 B.9 节开始探索一些可变对象。

相补的`lower`方法返回一个所有字符都转换为小写的新的字符串：

```
In  [81] "1611 BROADWAY".lower()

Out [81] '1611 broadway'
```

甚至还有一个 `swapcase` 方法，它返回一个新字符串，其中每个字符的大小写都被反转。大写字母变为小写，小写字母变为大写：

```
In  [82] "uPsIdE dOwN".swapcase()

Out [82] 'UpSiDe DoWn'
```

一个方法可以接受参数。让我们看看 `replace` 方法，它将所有子字符串的出现次数与指定的字符序列交换。该功能类似于文字处理程序中的查找和替换功能。`replace` 方法接受两个参数：

+   要查找的子字符串

+   要替换的值

下一个示例将所有 `"S"` 出现替换为 `"$"`：

```
In  [83] "Sally Sells Seashells by the Seashore".replace("S", "$")

Out [83] '$ally $ells $eashells by the $eashore'
```

在这个例子中，

+   `"Sally Sells Seashells by the Seashore"` 是原始的字符串 *对象*。

+   `replace` 是对字符串调用的 *方法*。

+   `"S"` 是传递给 `replace` 方法调用的 *第一个参数*。

+   `"$"` 是传递给 `replace` 方法调用的 *第二个参数*。

+   `"$ally $ells $eashells by the $eashore"` 是 `replace` 方法的 *返回值*。

一个方法返回值的数据类型可以与原始对象不同。例如，`isspace` 方法作用于字符串，但返回一个布尔值。如果字符串仅由空格组成，则方法返回 `True`；否则，返回 `False`。

```
In  [84] "  ".isspace()

Out [84] True

In  [85] "3 Amigos".isspace()

Out [85] False
```

字符串有一系列用于删除空白的方法。`rstrip`（右删除）方法从字符串的末尾删除空白：

```
In  [86] data = "    10/31/2019  "
         data.rstrip()

Out [86] '    10/31/2019'
```

`lstrip`（左删除）方法从字符串的开始删除空白：

```
In  [87] data.lstrip()

Out [87] '10/31/2019  '
```

`strip` 方法从字符串的两端删除空白：

```
In  [88] data.strip()

Out [88] '10/31/2019'
```

`capitalize` 方法将字符串的第一个字符大写。此方法通常在处理小写名称、地点或组织时非常有用：

```
In  [89] "robert".capitalize()

Out [89] 'Robert'
```

`title` 方法将字符串中每个单词的首字母大写，使用空格来标识每个单词的开始和结束位置：

```
In  [90] "once upon a time".title()

Out [90] 'Once Upon A Time'
```

我们可以在一行中连续调用多个方法。这种技术称为 *方法链*。在下一个示例中，`lower` 方法返回一个新的字符串对象，然后我们调用 `title` 方法。`title` 的返回值又是另一个新的字符串对象：

```
In  [91] "BENJAMIN FRANKLIN".lower().title()

Out [91] 'Benjamin Franklin'
```

`in` 关键字检查子字符串是否存在于另一个字符串中。在关键字之前输入要搜索的字符串，在关键字之后输入要搜索的字符串。操作返回一个布尔值：

```
In  [92] "tuna" in "fortunate"

Out [92] True

In  [93] "salmon" in "fortunate"

Out [93] False
```

`startswith` 方法检查子字符串是否存在于字符串的开头：

```
In  [94] "factory".startswith("fact")

Out [94] True
```

`endswith` 方法检查子字符串是否存在于字符串的末尾：

```
In  [95] "garage".endswith("rage")

Out [95] True
```

`count` 方法计算字符串中子字符串的出现次数。下一个示例计算 `"celebrate"` 中 `"e"` 字符的数量：

```
In  [96] "celebrate".count("e")

Out [96] 3
```

`find` 和 `index` 方法定位字符或子字符串的索引位置。这些方法返回参数首次出现的位置索引。回想一下，索引位置从 0 开始计数。下一个示例搜索 `"celebrate"` 中第一个 `"e"` 的索引。Python 在索引 1 处定位它：

```
In  [97] "celebrate".find("e")

Out [97] 1

In  [98] "celebrate".index("e")

Out [98] 1
```

`find` 和 `index` 方法有什么区别？如果字符串不包含参数，`find` 将返回 `-1`，而 `index` 将引发 `ValueError` 异常：

```
In  [99] "celebrate".find("z")

Out [99] -1

In  [100] "celebrate".index("z")

---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-5-bf78a69262aa> in <module>
----> 1 "celebrate".index("z")

ValueError: substring not found
```

每种方法都适用于特定的情况；两种选择没有哪种比另一种更好。如果你的程序依赖于一个子字符串存在于更大的字符串中，例如，你可以使用 `index` 方法并处理错误。相比之下，如果子字符串的缺失不会阻止你的程序执行，你可以使用 `find` 方法来避免崩溃。

## B.9 列表

*列表* 是一个按顺序存储对象的容器。列表的目的是双重的：提供一个“盒子”来存储值，并保持它们的顺序。我们称列表中的项目为 *元素*。在其他编程语言中，这种数据结构通常被称为 *数组*。

我们使用一对开方括号和闭方括号来声明一个列表。我们在方括号内写下我们的元素，每两个元素之间用逗号分隔。下一个示例创建了一个包含五个字符串的列表：

```
In  [101] backstreet_boys = ["Nick", "AJ", "Brian", "Howie", "Kevin"]
```

列表的长度等于其元素的数量。还记得那个可靠的 `len` 函数吗？它可以帮助我们确定史上最伟大的男孩乐队有多少成员：

```
In  [102] len(backstreet_boys)

Out [102] 5
```

*空列表* 是一个没有元素的列表。它的长度为 `0`：

```
In  [103] []

Out [103] []
```

列表可以存储任何数据类型的元素：字符串、数字、浮点数、布尔值等等。一个 *同质* 列表是指所有元素都具有相同类型的列表。以下三个列表是同质的。第一个包含整数，第二个包含浮点数，第三个包含布尔值：

```
In  [104] prime_numbers = [2, 3, 5, 7, 11]

In  [105] stock_prices_for_last_four_days = [99.93, 105.23, 102.18, 94.45]

In  [106] settings = [True, False, False, True, True, False]
```

列表也可以存储不同数据类型的元素。一个 *异质* 列表是指元素具有不同数据类型的列表。以下列表包含一个字符串、一个整数、一个布尔值和一个浮点数：

```
In  [107] motley_crew = ["rhinoceros", 42, False, 100.05]
```

就像 Python 为字符串中的每个字符分配索引位置一样，Python 为列表中的每个元素分配一个索引位置。索引表示元素在行中的位置，并从 0 开始计数。在以下三个元素的 `favorite_foods` 列表中，

+   `"Sushi"` 占据索引位置 0。

+   `"Steak"` 占据索引位置 1。

+   `"Barbeque"` 占据索引位置 2。

```
In  [108] favorite_foods = ["Sushi", "Steak", "Barbeque"]
```

关于列表格式的两个快速说明。首先，Python 允许我们在列表的最后一个元素后插入一个逗号。逗号根本不影响列表；它是一种替代语法：

```
In  [109] favorite_foods = ["Sushi", "Steak", "Barbeque",]
```

其次，一些 Python 风格指南建议将长列表拆分，以便每个元素占据一行。这种格式也不会以任何技术方式影响列表。语法看起来是这样的：

```
In  [110] favorite_foods = [
              "Sushi",
              "Steak",
              "Barbeque",
          ]
```

在本书的示例中，我使用了我认为最能增强可读性的格式化风格。你可以使用你觉得最舒服的格式。

我们可以通过索引位置访问列表元素。在列表（或引用它的变量）后面传递索引，放在一对方括号中：

```
In  [111] favorite_foods[1]

Out [111] 'Steak'
```

在 B.1.2 节中，我们介绍了一种切片语法来从字符串中提取字符。我们可以使用相同的语法从列表中提取元素。下一个示例从索引位置 1 到 3 提取元素。记住，在列表切片中，起始索引是包含的，而结束索引是不包含的：

```
In  [112] favorite_foods[1:3]

Out [112] ['Steak', 'Barbeque']
```

我们可以移除冒号前的数字来从列表的开头提取。下一个示例从列表的开头提取到索引 2（不包括）的元素：

```
In  [113] favorite_foods[:2]

Out [113] ['Sushi', 'Steak']
```

我们可以移除冒号后的数字来提取到列表的末尾。下一个示例从索引 2 提取到列表的末尾：

```
In  [114] favorite_foods[2:]

Out [114] ['Barbeque']
```

忽略两个数字以创建列表的副本：

```
In  [115] favorite_foods[:]

Out [115] ['Sushi', 'Steak', 'Barbeque']
```

最后，我们可以在方括号中提供一个可选的第三个数字来以间隔提取元素。下一个示例以 2 的增量从索引位置 0（包含）到索引位置 3（不包含）提取元素：

```
In  [116] favorite_foods[0:3:2]

Out [116] ['Sushi', 'Barbeque']
```

所有切片选项都返回一个新的列表。

让我们逐一了解一些列表方法。`append` 方法将新元素添加到列表的末尾：

```
In  [117] favorite_foods.append("Burrito")
          favorite_foods

Out [117] ['Sushi', 'Steak', 'Barbeque', 'Burrito']
```

你还记得我们关于可变性和不可变性的讨论吗？列表是一个可变对象的例子，是一个*能够*改变的对象。在创建列表后，我们可以添加、删除或替换列表中的元素。在前面的例子中，`append` 方法修改了由 `favorite_foods` 变量引用的列表。我们没有创建一个新的列表。

相比之下，字符串是一个不可变对象的例子。当我们调用像 `upper` 这样的方法时，Python 返回一个新的字符串；原始字符串保持不变。不可变对象不能改变。

列表包含多种变异方法。`extend` 方法将多个元素添加到列表的末尾。它接受一个参数，即要添加值的列表：

```
In  [118] favorite_foods.extend(["Tacos", "Pizza", "Cheeseburger"])
          favorite_foods

Out [118] ['Sushi', 'Steak', 'Barbeque', 'Burrito', 'Tacos', 'Pizza',
          'Cheeseburger']
```

`insert` 方法将元素添加到列表的特定索引位置。它的第一个参数是我们想要插入元素的位置，第二个参数是新的元素。Python 会将指定索引位置及其后的值推到下一个槽位。下一个示例将字符串 `"Pasta"` 插入到索引位置 2。列表将 `"Barbeque"` 和所有后续元素向上移动一个索引位置：

```
In  [119] favorite_foods.insert(2, "Pasta")
          favorite_foods

Out [119] ['Sushi',
           'Steak',
           'Pasta',
           'Barbeque',
           'Burrito',
           'Tacos',
           'Pizza',
           'Cheeseburger']
```

`in` 关键字可以检查列表是否包含一个元素。`"Pizza"` 在我们的 `favorite_foods` 列表中存在，而 `"Caviar"` 不存在：

```
In  [120] "Pizza" in favorite_foods

Out [120] True

In  [121] "Caviar" in favorite_foods

Out [121] False
```

`not in` 操作符确认列表中不存在一个元素。它返回 `in` 操作符的逆布尔值：

```
In  [122] "Pizza" not in favorite_foods

Out [122] False

In  [123] "Caviar" not in favorite_foods

Out [123] True
```

`count` 方法计算元素在列表中出现的次数：

```
In  [124] favorite_foods.append("Pasta")
          favorite_foods

Out [124] ['Sushi',
           'Steak',
           'Pasta',
           'Barbeque',
           'Burrito',
           'Tacos',
           'Pizza',
           'Cheeseburger',
           'Pasta']

In  [125] favorite_foods.count("Pasta")

Out [125] 2
```

`remove` 方法从列表中删除第一个出现的元素。注意，Python 不会删除该元素的后续出现：

```
In  [126] favorite_foods.remove("Pasta")
          favorite_foods

Out [126] ['Sushi',
           'Steak',
           'Barbeque',
           'Burrito',
           'Tacos',
           'Pizza',
           'Cheeseburger',
           'Pasta']
```

让我们去除列表末尾的其他 `"Pasta"` 字符串。`pop` 方法从列表中移除并返回最后一个元素：

```
In  [127] favorite_foods.pop()

Out [127] 'Pasta'

In  [128] favorite_foods

Out [128] ['Sushi', 'Steak', 'Barbeque', 'Burrito', 'Tacos', 'Pizza',
           'Cheeseburger']
```

`pop`方法也接受一个整数参数，表示 Python 应该删除的值的索引位置。在下一个示例中，我们删除了索引位置 2 的`"Barbeque"`值。`"Burrito"`字符串滑入索引位置 2，并且它后面的元素也向下移动一个索引：

```
In  [129] favorite_foods.pop(2)

Out [129] 'Barbeque'

In  [130] favorite_foods

Out [130] ['Sushi', 'Steak', 'Burrito', 'Tacos', 'Pizza', 'Cheeseburger']
```

列表可以存储任何对象，包括其他列表。在下一个示例中，我们声明了一个包含三个嵌套列表的列表。每个嵌套列表包含三个整数：

```
In  [131] spreadsheet = [
              [1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]
          ]
```

让我们花点时间反思一下前面的视觉表示。你能看到任何与电子表格的相似之处吗？嵌套列表是我们表示多维、表格化数据集合的一种方式。我们可以将最外层的列表视为工作表，每个内部列表视为数据行。

### B.9.1 列表迭代

列表是集合对象的一个例子。它能够存储多个值——一个*集合*的值。*迭代*意味着逐个移动集合对象的元素。

遍历列表项最常见的方式是使用`for`循环。其语法看起来像这样：

```
for variable_name in some_list:
    # Do something
```

`for`循环由几个组件组成：

+   `for`关键字。

+   一个变量名，在迭代过程中将逐个存储列表元素。

+   `in`关键字。

+   要迭代的列表。

+   Python 在每次迭代期间将运行的代码块。我们可以在代码块中使用变量名。

作为提醒，一个*块*是缩进代码的部分。Python 使用缩进来关联程序中的结构。位于函数名下方的块定义了函数的功能。同样，位于`for`循环下方的块定义了每次迭代期间发生的事情。

在下一个示例中，我们迭代一个包含四个字符串的列表，打印出每个字符串的长度：

```
In  [132] for season in ["Winter", "Spring", "Summer", "Fall"]:
              print(len(season))

Out [132] 6
          6
          6
          4
```

前面的迭代包含四个循环。`season`变量按顺序持有`"Winter"`、`"Spring"`、`"Summer"`和`"Fall"`的值。在每次迭代中，我们将当前字符串传递给`len`函数。`len`函数返回一个数字，我们将它打印出来。

假设我们想要将字符串的长度相加。我们必须将`for`循环与其他 Python 概念结合起来。在下一个示例中，我们首先初始化一个`letter_count`变量来保存累积总和。在`for`循环块内部，我们使用`len`函数计算当前字符串的长度，然后覆盖运行总和。最后，在循环完成后输出`letter_count`的值：

```
In  [133] letter_count = 0

          for season in ["Winter", "Spring", "Summer", "Fall"]:
              letter_count = letter_count + len(season)

          letter_count

Out [133] 22
```

`for`循环是迭代列表最传统的方法。Python 还支持另一种语法，我们将在 B.9.2 节中讨论。

### B.9.2 列表推导式

*列表* *推导式*是创建列表的简写语法，从集合对象中生成。假设我们有一个包含六个数字的列表：

```
In  [134] numbers = [4, 8, 15, 16, 23, 42]
```

假设我们想要创建一个包含那些数字平方的新列表。换句话说，我们想要对原始列表中的每个元素应用一个一致的运算。一个解决方案是遍历`numbers`中的每个整数，取其平方，并将结果添加到新列表中。提醒一下，`append`方法将元素添加到列表的末尾：

```
In  [135] squares = []

          for number in numbers:
              squares.append(number ** 2)

          squares

Out [135] [16, 64, 225, 256, 529, 1764]
```

列表推导可以单行生成相同的平方列表。其语法需要一个成对的开放和闭合方括号。在括号内，我们首先描述我们想要对迭代的每个元素做什么，然后是从中获取可迭代项的集合。

下一个示例仍然遍历`numbers`列表，并将每个列表元素分配给一个`number`变量。我们在`for`关键字之前声明我们想要对每个`number`做什么。我们将`number ** 2`计算移到开始，将`for in`逻辑移到末尾：

```
In  [136] squares = [number ** 2 for number in numbers]
          squares

Out [136] [16, 64, 225, 256, 529, 1764]
```

列表推导被认为是创建新列表的更 Pythonic 方式，从现有的数据结构中创建。*Pythonic 方式*描述了 Python 开发者随着时间的推移所采用的推荐实践集合。

### B.9.3 将字符串转换为列表及其相反操作

我们现在熟悉列表和字符串了，让我们看看我们如何可以将它们一起使用。假设我们的程序中有一个包含地址的字符串：

```
In  [137] empire_state_bldg = "20 West 34th Street, New York, NY, 10001"
```

如果我们想要将地址分解成更小的组成部分：街道、城市、州和邮政编码呢？请注意，该字符串使用逗号来分隔这四个部分。

字符串的`split`方法通过使用*分隔符*，一个或多个字符的序列来标记边界，将字符串分解。下一个示例要求`split`方法在`empire_state_building`的每个逗号出现处进行分割。该方法返回一个由较小的字符串组成的列表：

```
In  [138] empire_state_bldg.split(",")

Out [138] ['20 West 34th Street', ' New York', ' NY', ' 10001']
```

这段代码是朝着正确方向迈出的一步。但请注意，列表中的最后三个元素前面有一个前置空格。虽然我们可以遍历列表的元素并对每个元素调用`strip`方法来移除其空白字符，但一个更优的解决方案是将空格添加到`split`方法的分隔符参数中：

```
In  [139] empire_state_bldg.split(", ")

Out [139] ['20 West 34th Street', 'New York', 'NY', '10001']
```

我们已经成功将字符串分解成字符串列表。

该过程也可以反向进行。假设我们将地址存储在一个列表中，并希望将列表的元素连接成一个单独的字符串：

```
In  [140] chrysler_bldg = ["405 Lexington Ave", "New York", "NY", "10174"]
```

首先，我们必须声明我们希望 Python 在两个列表元素之间注入的字符串。然后我们可以对字符串调用`join`方法，并将列表作为参数传入。Python 会将列表的元素连接起来，每个元素之间用分隔符分隔。下一个示例使用逗号和空格作为分隔符：

```
In  [141] ", ".join(chrysler_bldg)

Out [141] '405 Lexington Ave, New York, NY, 10174'
```

`split`和`join`方法对于处理文本数据很有帮助，这些数据通常需要被分离和重新合并。

## B.10 元组

一个 *元组* 与 Python 列表类似的数据结构。元组也按顺序存储元素，但与列表不同，它是不可变的。一旦创建了元组，我们就不能添加、删除或替换其中的元素。

定义元组的唯一技术要求是声明多个元素，并且用逗号分隔每个后续的两个元素。以下示例声明了一个包含三个元素的元组：

```
In  [142] "Rock", "Pop", "Country"

Out [142] ('Rock', 'Pop', 'Country')
```

然而，通常我们使用一对括号来声明元组。这种语法使得从视觉上识别对象变得更容易：

```
In  [143] music_genres = ("Rock", "Pop", "Country")
          music_genres

Out [143] ('Rock', 'Pop', 'Country')
```

`len` 函数返回元组的长度：

```
In  [144] len(music_genres)

Out [144] 3
```

要声明一个只有一个元素的元组，我们必须在元素后面包含一个逗号。Python 需要逗号来识别元组。比较以下两个输出的差异。第一个示例没有使用逗号；Python 将值读取为字符串。

```
In  [145] one_hit_wonders = ("Never Gonna Give You Up")
          one_hit_wonders

Out [145] 'Never Gonna Give You Up'
```

相比之下，这里的语法返回一个元组。是的，一个符号在 Python 中可以产生巨大的差异：

```
In  [146] one_hit_wonders = ("Never Gonna Give You Up",)
          one_hit_wonders

Out [146] ('Never Gonna Give You Up',)
```

使用 `tuple` 函数创建一个 *空元组*，即一个没有元素的元组：

```
In  [147] empty_tuple = tuple()
          empty_tuple

Out [147] ()

In  [148] len(empty_tuple)

Out [148] 0
```

与列表一样，你可以通过索引位置访问元组元素。与列表一样，你可以使用 `for` 循环遍历元组元素。唯一不能做的是修改元组。由于其不可变性，元组不包含如 `append`、`pop` 和 `insert` 这样的突变方法。

如果你有一组有序的元素，并且知道它不会改变，你可以选择使用元组而不是列表来存储它。

## B.11 字典

列表和元组是存储有序对象的最佳数据结构。我们需要另一种数据结构来解决不同类型的问题：在对象之间建立关联。

考虑一家餐厅的菜单。每个菜单项都是一个唯一的标识符，我们用它来查找相应的价格。菜单项和其成本是关联的。项目的顺序并不重要；重要的是两份数据之间的 *联系*。

一个 *字典* 是一个可变、无序的键值对集合。一对由一个键和一个值组成。每个键都是一个值的标识符。键必须是唯一的。值可以包含重复项。

我们使用一对花括号 (`{}`) 声明字典。以下示例创建了一个空字典：

```
In  [149] {}

Out [149] {}
```

让我们在 Python 中模拟一个示例餐厅菜单。在花括号内，我们使用冒号 (`:`) 为其值分配一个键。以下示例声明了一个包含一个键值对的字典。字符串键 `"Cheeseburger"` 被分配了浮点值 `7.99`：

```
In  [150] { "Cheeseburger": 7.99 }
Out [150] {'Cheeseburger': 7.99}
```

当声明包含多个键值对的字典时，用逗号分隔每个两个键值对。让我们扩展我们的 `menu` 字典以包含三个键值对。注意 `"French Fries"` 和 `"Soda"` 键的值是相同的：

```
In  [151] menu = {"Cheeseburger": 7.99, "French Fries": 2.99, "Soda": 2.99}
          menu

Out [151] {'Cheeseburger': 7.99, 'French Fries': 2.99, 'Soda': 2.99}
```

我们可以通过将其传递给 Python 的内置 `len` 函数来计算字典中键值对的数量：

```
In  [152] len(menu)

Out [152] 3
```

我们使用键从字典中检索值。在字典后面立即放置一对方括号，并紧跟键。语法与通过索引位置访问列表元素相同。以下示例提取了 `"French Fries"` 键的值：

```
In  [153] menu["French Fries"]

Out [153] 2.99
```

在列表中，索引位置始终是数字。在字典中，键可以是任何不可变的数据类型：整数、浮点数、字符串、布尔值等等。

如果键在字典中不存在，Python 将引发一个 `KeyError` 异常。`KeyError` 是原生 Python 错误的另一个例子：

```
In  [154] menu["Steak"]

---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-19-0ad3e3ec4cd7> in <module>
----> 1 menu["Steak"]

KeyError: 'Steak'
```

总是注意大小写敏感。如果单个字符不匹配，Python 将无法找到键。在我们的字典中不存在 `"soda"` 这个键。只有 `"Soda"`：

```
In  [155] menu["soda"]

---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-20-47940ceca824> in <module>
----> 1 menu["soda"]

KeyError: 'soda'
```

`get` 方法也可以通过键提取字典值：

```
In  [156] menu.get("French Fries")

Out [156] 2.99
```

`get` 方法的优势是，如果键不存在，它返回 `None` 而不是引发错误。记住，`None` 是 Python 用来表示不存在或空值的概念的一个对象。`None` 值在 Jupyter Notebook 中不会产生任何视觉输出。但我们可以用 `print` 函数包裹调用，强制 Python 打印 `None` 的字符串表示：

```
In  [157] print(menu.get("Steak"))

Out [157] None
```

`get` 方法的第二个参数是在字典中键不存在时返回的自定义值。在下一个示例中，字符串 `"Steak"` 并不是 `menu` 字典中的键，所以 Python 返回 `99.99`：

```
In  [158] menu.get("Steak", 99.99)

Out [158] 99.99
```

字典是一个可变的数据结构。在创建字典后，我们可以向其中添加键值对或从字典中删除键值对。要添加一个新的键值对，提供键并用赋值运算符（`=`）为其赋值：

```
In  [159] menu["Taco"] = 0.99
          menu

Out [159] {'Cheeseburger': 7.99, 'French Fries': 2.99, 'Soda': 1.99,
          'Taco': 0.99}
```

如果键已经存在于字典中，Python 将覆盖其原始值。在下一个示例中，将 `"Cheeseburger"` 键的值从 `7.99` 更改为 `9.99`：

```
In  [160] print(menu["Cheeseburger"])
          menu["Cheeseburger"] = 9.99
          print(menu["Cheeseburger"])

Out [160] 7.99
          9.99
```

`pop` 方法从一个字典中移除一个键值对；它接受一个键作为参数并返回其值。如果键在字典中不存在，Python 将引发一个 `KeyError` 异常：

```
In  [161] menu.pop("French Fries")

Out [161] 2.99

In  [162] menu

Out [162] {'Cheeseburger': 9.99, 'Soda': 1.99, 'Taco': 0.99}
```

`in` 关键字检查一个元素是否存在于字典的键中：

```
In  [163] "Soda" in menu

Out [163] True

In  [164] "Spaghetti" in menu

Out [164] False
```

要检查字典值中的包含情况，在字典上调用 `values` 方法。该方法返回一个类似列表的对象，包含字典的值。我们可以将 `in` 操作符与 `values` 方法的返回值结合使用：

```
In  [165] 1.99 in menu.values()

Out [165] True

In  [166] 499.99 in menu.values()

Out [166] False
```

`values` 方法返回的对象类型与我们之前看到的列表、元组和字典不同。我们不一定需要知道这个对象是什么，然而。我们只关心我们如何与之交互。`in` 操作符检查一个值是否在对象中，而 `values` 方法返回的对象知道如何处理它。

### B.11.1 字典迭代

我们应该始终假设字典的键值对是无序的。如果您需要一个保持顺序的数据结构，请使用列表或元组。如果您需要创建对象之间的关联，请使用字典。

即使我们不能保证迭代顺序的确定性，我们仍然可以使用 `for` 循环一次迭代一个键值对来遍历字典。字典的 `items` 方法在每次迭代时返回一个包含两个元素的元组。该元组包含一个键及其相应的值。我们可以在 `for` 关键字之后声明多个变量来存储每个键和值。在下一个示例中，`state` 变量包含每个字典键，而 `capital` 变量包含每个值：

```
In  [167] capitals = {
              "New York": "Albany",
              "Florida": "Tallahassee",
              "California": "Sacramento"
          }

          for state, capital in capitals.items():
              print("The capital of " + state + " is " + capital + ".")

          The capital of New York is Albany.
          The capital of Florida is Tallahassee.
          The capital of California is Sacramento.
```

在第一次迭代中，Python 返回一个包含 `("New York", "Albany")` 的元组。在第二次迭代中，它返回一个包含 `("Florida", "Tallahassee")` 的元组，依此类推。

## B.12 集合

列表和字典对象有助于解决顺序和关联的问题。集合帮助解决另一个常见需求：唯一性。一个 *集合* 是一个无序、可变且元素唯一的集合。它禁止重复。

我们使用一对花括号声明一个集合。我们在花括号中填充元素，每两个元素之间用逗号分隔。下一个示例声明了一个包含六个数字的集合：

```
In  [168] favorite_numbers = { 4, 8, 15, 16, 23, 42 }
```

眼光敏锐的读者可能会注意到，声明集合的花括号语法与声明字典的语法相同。Python 可以根据键值对的存在与否来区分这两种类型的对象。

因为 Python 将空的一对花括号解释为空字典，所以创建空集合的唯一方法是使用内置的 `set` 函数：

```
In  [169] set()

Out [169] set()
```

这里有一些有用的集合方法。`add` 方法向集合中添加一个新元素：

```
In  [170] favorite_numbers.add(100)
          favorite_numbers

Out [170] {4, 8, 15, 16, 23, 42, 100}
```

Python 只会在集合中不存在该元素的情况下向集合中添加一个元素。下一个示例尝试将 15 添加到 `favorite_numbers` 中。Python 发现 15 已经存在于集合中，因此对象保持不变：

```
In  [171] favorite_numbers.add(15)
          favorite_numbers

Out [171] {4, 8, 15, 16, 23, 42, 100}
```

集合没有顺序的概念。如果我们尝试通过索引位置访问集合元素，Python 会引发 `TypeError` 异常：

```
In  [172] favorite_numbers[2]

---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-17-e392cd51c821> in <module>
----> 1 favorite_numbers[2]

TypeError: 'set' object is not subscriptable
```

当我们尝试对一个无效对象应用操作时，Python 会引发 `TypeError` 异常。集合元素是无序的，因此元素没有索引位置。

除了防止重复外，集合非常适合识别两个数据集合之间的相似性和差异性。让我们定义两个字符串集合：

```
In  [173] candy_bars = { "Milky Way", "Snickers", "100 Grand" }
          sweet_things = { "Sour Patch Kids", "Reeses Pieces", "Snickers" }
```

`intersection` 方法返回一个新集合，其中包含在两个原始集合中都找到的元素。`&` 符号执行相同的逻辑。在下一个示例中，`"Snickers"` 是 `candy_bars` 和 `sweet_things` 之间唯一的字符串：

```
In  [174] candy_bars.intersection(sweet_things)

Out [174] {'Snickers'}

In  [175] candy_bars & sweet_things

Out [175] {'Snickers'}
```

`union` 方法返回一个集合，该集合包含两个集合的所有元素。`|` 符号执行相同的逻辑。请注意，像 `"Snickers"` 这样的重复值只会出现一次：

```
In  [176] candy_bars.union(sweet_things)

Out [176] {'100 Grand', 'Milky Way', 'Reeses Pieces', 'Snickers', 'Sour
          Patch Kids'}

In  [177] candy_bars | sweet_things

Out [177] {'100 Grand', 'Milky Way', 'Reeses Pieces', 'Snickers', 'Sour
          Patch Kids'}
```

`difference`方法返回一个集合，其中包含在调用该方法时在集合中存在的元素，但不包含在作为参数传递的集合中。我们可以使用`-`符号作为快捷方式。在下一个例子中，`"100 Grand"`和`"Milky Way"`存在于`candy_bars`中，但不在`sweet_things`中：

```
In  [178] candy_bars.difference(sweet_things)

Out [178] {'100 Grand', 'Milky Way'}

In  [179] candy_bars - sweet_things

Out [179] {'100 Grand', 'Milky Way'}
```

`symmetric_difference`方法返回一个集合，其中包含在任一集合中找到的元素，但不包含在两个集合中。`^`语法可以达到相同的结果：

```
In  [180] candy_bars.symmetric_difference(sweet_things)

Out [180] {'100 Grand', 'Milky Way', 'Reeses Pieces', 'Sour Patch Kids'}

In  [181] candy_bars ^ sweet_things

Out [181] {'100 Grand', 'Milky Way', 'Reeses Pieces', 'Sour Patch Kids'}
```

就这些了！我们已经学到了很多 Python 知识：数据类型、函数、迭代等等。如果你记不住所有细节也没关系。相反，当你需要复习 Python 核心机制时，随时回到这个附录。在我们使用 pandas 库的过程中，我们会用到并回顾很多这些想法。

* * *

¹ 请参阅“PEP 8—Python 代码风格指南”，[`www.python.org/dev/peps/pep-0008`](https://www.python.org/dev/peps/pep-0008)。
