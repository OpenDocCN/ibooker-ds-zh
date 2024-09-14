# 第二十二章：向量化字符串操作

Python 的一个优点是相对容易处理和操作字符串数据。Pandas 在此基础上构建，并提供了一套全面的*向量化字符串操作*，这是处理（即：清理）现实世界数据时必不可少的部分。在本章中，我们将逐步介绍一些 Pandas 字符串操作，然后看看如何使用它们部分清理从互联网收集的非常混乱的食谱数据集。

# 引入 Pandas 字符串操作

在之前的章节中，我们看到工具如 NumPy 和 Pandas 如何将算术操作泛化，以便我们可以轻松快速地在许多数组元素上执行相同的操作。例如：

```py
In [1]: import numpy as np
        x = np.array([2, 3, 5, 7, 11, 13])
        x * 2
Out[1]: array([ 4,  6, 10, 14, 22, 26])
```

这种操作的*向量化*简化了操作数组数据的语法：我们不再需要担心数组的大小或形状，而只需关注我们想要进行的操作。对于字符串数组，NumPy 没有提供如此简单的访问方式，因此你只能使用更冗长的循环语法：

```py
In [2]: data = ['peter', 'Paul', 'MARY', 'gUIDO']
        [s.capitalize() for s in data]
Out[2]: ['Peter', 'Paul', 'Mary', 'Guido']
```

这可能足以处理一些数据，但如果有任何缺失值，它将会出错，因此这种方法需要额外的检查：

```py
In [3]: data = ['peter', 'Paul', None, 'MARY', 'gUIDO']
        [s if s is None else s.capitalize() for s in data]
Out[3]: ['Peter', 'Paul', None, 'Mary', 'Guido']
```

这种手动方法不仅冗长且不方便，还容易出错。

Pandas 包括功能来同时解决对向量化字符串操作的需求以及通过 Pandas `Series`和`Index`对象的`str`属性正确处理缺失数据的需求。因此，例如，如果我们创建一个包含这些数据的 Pandas `Series`，我们可以直接调用`str.capitalize`方法，其中内置了缺失值处理：

```py
In [4]: import pandas as pd
        names = pd.Series(data)
        names.str.capitalize()
Out[4]: 0    Peter
        1     Paul
        2     None
        3     Mary
        4    Guido
        dtype: object
```

# Pandas 字符串方法表

如果你对 Python 中的字符串操作有很好的理解，大部分 Pandas 字符串语法都足够直观，可能只需列出可用的方法就足够了。我们先从这里开始，然后深入探讨一些细微之处。本节的示例使用以下`Series`对象：

```py
In [5]: monte = pd.Series(['Graham Chapman', 'John Cleese', 'Terry Gilliam',
                           'Eric Idle', 'Terry Jones', 'Michael Palin'])
```

## 类似于 Python 字符串方法的方法

几乎所有 Python 内置的字符串方法都有与之对应的 Pandas 向量化字符串方法。以下 Pandas `str`方法与 Python 字符串方法相对应：

| `len` | `lower` | `translate` | `islower` | `ljust` |
| --- | --- | --- | --- | --- |
| `upper` | `startswith` | `isupper` | `rjust` | `find` |
| `endswith` | `isnumeric` | `center` | `rfind` | `isalnum` |
| `isdecimal` | `zfill` | `index` | `isalpha` | `split` |
| `strip` | `rindex` | `isdigit` | `rsplit` | `rstrip` |
| `capitalize` | `isspace` | `partition` | `lstrip` | `swapcase` |

注意这些具有不同的返回值。一些像`lower`这样的方法返回一系列字符串：

```py
In [6]: monte.str.lower()
Out[6]: 0    graham chapman
        1       john cleese
        2     terry gilliam
        3         eric idle
        4       terry jones
        5     michael palin
        dtype: object
```

但有些返回数字：

```py
In [7]: monte.str.len()
Out[7]: 0    14
        1    11
        2    13
        3     9
        4    11
        5    13
        dtype: int64
```

或者布尔值：

```py
In [8]: monte.str.startswith('T')
Out[8]: 0    False
        1    False
        2     True
        3    False
        4     True
        5    False
        dtype: bool
```

还有一些方法返回每个元素的列表或其他复合值：

```py
In [9]: monte.str.split()
Out[9]: 0    [Graham, Chapman]
        1       [John, Cleese]
        2     [Terry, Gilliam]
        3         [Eric, Idle]
        4       [Terry, Jones]
        5     [Michael, Palin]
        dtype: object
```

当我们继续讨论时，我们将看到这种系列列表对象的进一步操作。

## 使用正则表达式的方法

此外，还有几种方法接受正则表达式（regexps）来检查每个字符串元素的内容，并遵循 Python 内置 `re` 模块的一些 API 约定（参见 Table 22-1）。

Table 22-1\. Pandas 方法与 Python `re` 模块函数的映射关系

| 方法 | 描述 |
| --- | --- |
| `match` | 对每个元素调用 `re.match`，返回布尔值。 |
| `extract` | 对每个元素调用 `re.match`，返回匹配的字符串组。 |
| `findall` | 对每个元素调用 `re.findall` |
| `replace` | 用其他字符串替换模式的出现 |
| `contains` | 对每个元素调用 `re.search`，返回布尔值 |
| `count` | 计算模式的出现次数 |
| `split` | 等同于 `str.split`，但接受正则表达式 |
| `rsplit` | 等同于 `str.rsplit`，但接受正则表达式 |

使用这些方法，我们可以进行各种操作。例如，通过请求每个元素开头的一组连续字符，我们可以从中提取每个元素的名字：

```py
In [10]: monte.str.extract('([A-Za-z]+)', expand=False)
Out[10]: 0     Graham
         1       John
         2      Terry
         3       Eric
         4      Terry
         5    Michael
         dtype: object
```

或者我们可以做一些更复杂的事情，比如找出所有以辅音开头和结尾的名字，利用正则表达式的开头（`^`）和结尾（`$`）字符：

```py
In [11]: monte.str.findall(r'^[^AEIOU].*[^aeiou]$')
Out[11]: 0    [Graham Chapman]
         1                  []
         2     [Terry Gilliam]
         3                  []
         4       [Terry Jones]
         5     [Michael Palin]
         dtype: object
```

能够简洁地应用正则表达式于 `Series` 或 `DataFrame` 条目之上，为数据的分析和清理开辟了许多可能性。

## 杂项方法

最后，Table 22-2 列出了使其他便捷操作得以实现的杂项方法。

Table 22-2\. 其他 Pandas 字符串方法

| 方法 | 描述 |
| --- | --- |
| `get` | 对每个元素进行索引 |
| `slice` | 对每个元素进行切片 |
| `slice_replace` | 用传递的值替换每个元素中的片段 |
| `cat` | 连接字符串 |
| `repeat` | 重复值 |
| `normalize` | 返回字符串的 Unicode 形式 |
| `pad` | 在字符串的左侧、右侧或两侧添加空格 |
| `wrap` | 将长字符串分割成长度小于给定宽度的行 |
| `join` | 将 `Series` 中每个元素的字符串用指定分隔符连接起来 |
| `get_dummies` | 提取作为 `DataFrame` 的虚拟变量 |

### 向量化项访问和切片

特别是 `get` 和 `slice` 操作，使得可以从每个数组中进行向量化元素访问。例如，我们可以使用 `str.slice(0, 3)` 获取每个数组的前三个字符的片段。这种行为也可以通过 Python 的正常索引语法实现；例如，`df.str.slice(0, 3)` 相当于 `df.str[0:3]`：

```py
In [12]: monte.str[0:3]
Out[12]: 0    Gra
         1    Joh
         2    Ter
         3    Eri
         4    Ter
         5    Mic
         dtype: object
```

通过 `df.str.get(i)` 和 `df.str[i]` 进行的索引与之类似。

这些索引方法还允许您访问由 `split` 返回的数组的元素。例如，结合 `split` 和 `str` 索引，可以提取每个条目的姓氏：

```py
In [13]: monte.str.split().str[-1]
Out[13]: 0    Chapman
         1     Cleese
         2    Gilliam
         3       Idle
         4      Jones
         5      Palin
         dtype: object
```

### 指标变量

另一种需要额外解释的方法是`get_dummies`方法。当你的数据包含某种编码指示器时，这将非常有用。例如，我们可能有一个数据集，其中包含以代码形式的信息，比如 A = “出生在美国”，B = “出生在英国”，C = “喜欢奶酪”，D = “喜欢午餐肉”：

```py
In [14]: full_monte = pd.DataFrame({'name': monte,
                                    'info': ['B|C|D', 'B|D', 'A|C',
                                             'B|D', 'B|C', 'B|C|D']})
         full_monte
Out[14]:              name   info
         0  Graham Chapman  B|C|D
         1     John Cleese    B|D
         2   Terry Gilliam    A|C
         3       Eric Idle    B|D
         4     Terry Jones    B|C
         5   Michael Palin  B|C|D
```

`get_dummies`例程允许我们将这些指示变量拆分成一个`DataFrame`：

```py
In [15]: full_monte['info'].str.get_dummies('|')
Out[15]:    A  B  C  D
         0  0  1  1  1
         1  0  1  0  1
         2  1  0  1  0
         3  0  1  0  1
         4  0  1  1  0
         5  0  1  1  1
```

借助这些操作作为构建块，您可以在清理数据时构建各种无穷无尽的字符串处理过程。

我们在这里不会进一步深入这些方法，但我鼓励您阅读 [“处理文本数据”](https://oreil.ly/oYgWA) 在 Pandas 在线文档中，或参考 “进一步资源” 中列出的资源。

# 示例：食谱数据库

在清理混乱的现实世界数据时，这些向量化的字符串操作变得非常有用。这里我将通过一个例子详细介绍这一点，使用从网上各种来源编译的开放食谱数据库。我们的目标是将食谱数据解析成成分列表，以便我们可以根据手头上的一些成分快速找到一个食谱。用于编译这些脚本的代码可以在 [GitHub](https://oreil.ly/3S0Rg) 找到，并且数据库的最新版本链接也可以在那里找到。

这个数据库大小约为 30 MB，可以使用以下命令下载并解压：

```py
In [16]: # repo = "https://raw.githubusercontent.com/jakevdp/open-recipe-data/master"
         # !cd data && curl -O {repo}/recipeitems.json.gz
         # !gunzip data/recipeitems.json.gz
```

数据库以 JSON 格式存在，因此我们将使用`pd.read_json`来读取它（对于这个数据集，需要使用`lines=True`，因为文件的每一行都是一个 JSON 条目）：

```py
In [17]: recipes = pd.read_json('data/recipeitems.json', lines=True)
         recipes.shape
Out[17]: (173278, 17)
```

我们看到有将近 175,000 个食谱和 17 列。让我们看看一行，看看我们有什么：

```py
In [18]: recipes.iloc[0]
Out[18]: _id                                {'$oid': '5160756b96cc62079cc2db15'}
         name                                    Drop Biscuits and Sausage Gravy
         ingredients           Biscuits\n3 cups All-purpose Flour\n2 Tablespo...
         url                   http://thepioneerwoman.com/cooking/2013/03/dro...
         image                 http://static.thepioneerwoman.com/cooking/file...
         ts                                             {'$date': 1365276011104}
         cookTime                                                          PT30M
         source                                                  thepioneerwoman
         recipeYield                                                          12
         datePublished                                                2013-03-11
         prepTime                                                          PT10M
         description           Late Saturday afternoon, after Marlboro Man ha...
         totalTime                                                           NaN
         creator                                                             NaN
         recipeCategory                                                      NaN
         dateModified                                                        NaN
         recipeInstructions                                                  NaN
         Name: 0, dtype: object
```

那里有很多信息，但其中大部分都是以非常混乱的形式存在的，这是从网上爬取数据的典型情况。特别是成分列表以字符串格式存在；我们需要仔细提取我们感兴趣的信息。让我们先仔细查看一下这些成分：

```py
In [19]: recipes.ingredients.str.len().describe()
Out[19]: count    173278.000000
         mean        244.617926
         std         146.705285
         min           0.000000
         25%         147.000000
         50%         221.000000
         75%         314.000000
         max        9067.000000
         Name: ingredients, dtype: float64
```

成分列表平均长度为 250 个字符，最小为 0，最大接近 10,000 个字符！

出于好奇，让我们看看哪个食谱的成分列表最长：

```py
In [20]: recipes.name[np.argmax(recipes.ingredients.str.len())]
Out[20]: 'Carrot Pineapple Spice &amp; Brownie Layer Cake with Whipped Cream &amp;
          > Cream Cheese Frosting and Marzipan Carrots'
```

我们可以进行其他聚合探索；例如，我们可以查看有多少食谱是早餐食品（使用正则表达式语法匹配小写和大写字母）：

```py
In [21]: recipes.description.str.contains('[Bb]reakfast').sum()
Out[21]: 3524
```

或者有多少食谱将肉桂列为成分：

```py
In [22]: recipes.ingredients.str.contains('[Cc]innamon').sum()
Out[22]: 10526
```

我们甚至可以查看是否有任何食谱将成分拼错为“cinamon”：

```py
In [23]: recipes.ingredients.str.contains('[Cc]inamon').sum()
Out[23]: 11
```

这是 Pandas 字符串工具可以实现的数据探索类型。Python 在这类数据整理方面表现得非常出色。

## 一个简单的食谱推荐器

让我们再进一步，开始制作一个简单的菜谱推荐系统：给定一系列食材，我们希望找到使用所有这些食材的任何菜谱。虽然概念上很简单，但由于数据的异构性而变得复杂：例如，从每行提取一个干净的食材列表并不容易。因此，我们会稍微作弊一点：我们将从常见食材列表开始，然后简单地搜索是否在每个菜谱的食材列表中。为简单起见，我们暂时只使用香草和香料：

```py
In [24]: spice_list = ['salt', 'pepper', 'oregano', 'sage', 'parsley',
                       'rosemary', 'tarragon', 'thyme', 'paprika', 'cumin']
```

然后，我们可以构建一个由`True`和`False`值组成的布尔`DataFrame`，指示每种食材是否出现在列表中：

```py
In [25]: import re
         spice_df = pd.DataFrame({
             spice: recipes.ingredients.str.contains(spice, re.IGNORECASE)
             for spice in spice_list})
         spice_df.head()
Out[25]:     salt  pepper  oregano   sage  parsley  rosemary  tarragon  thyme   \
         0  False   False    False   True    False     False     False  False
         1  False   False    False  False    False     False     False  False
         2   True    True    False  False    False     False     False  False
         3  False   False    False  False    False     False     False  False
         4  False   False    False  False    False     False     False  False

            paprika   cumin
         0    False   False
         1    False   False
         2    False    True
         3    False   False
         4    False   False
```

现在，举个例子，假设我们想找到使用欧芹、辣椒粉和龙蒿的菜谱。我们可以使用`DataFrame`的`query`方法快速计算这一点，有关详细信息，请参阅 第二十四章：

```py
In [26]: selection = spice_df.query('parsley & paprika & tarragon')
         len(selection)
Out[26]: 10
```

我们只找到了这种组合的 10 个菜谱。让我们使用此选择返回的索引来发现这些菜谱的名称：

```py
In [27]: recipes.name[selection.index]
Out[27]: 2069      All cremat with a Little Gem, dandelion and wa...
         74964                         Lobster with Thermidor butter
         93768      Burton's Southern Fried Chicken with White Gravy
         113926                     Mijo's Slow Cooker Shredded Beef
         137686                     Asparagus Soup with Poached Eggs
         140530                                 Fried Oyster Po’boys
         158475                Lamb shank tagine with herb tabbouleh
         158486                 Southern fried chicken in buttermilk
         163175            Fried Chicken Sliders with Pickles + Slaw
         165243                        Bar Tartine Cauliflower Salad
         Name: name, dtype: object
```

现在我们已经将菜谱选择从 175,000 缩减到了 10，我们可以更加明智地决定晚餐要做什么了。

## 进一步探索菜谱

希望这个例子给了你一点关于 Pandas 字符串方法能够高效实现的数据清理操作类型的味道（嘿）。当然，构建一个健壮的菜谱推荐系统需要 *很多* 工作！从每个菜谱中提取完整的食材列表将是任务的重要部分；不幸的是，使用的各种格式的广泛变化使得这成为一个相对耗时的过程。这表明在数据科学中，清理和整理真实世界数据通常占据了大部分工作量—而 Pandas 提供了可以帮助您高效完成这项工作的工具。
