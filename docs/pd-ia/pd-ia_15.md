# 13 配置 pandas

本章涵盖

+   配置 pandas 显示设置以同时适用于笔记本和单个单元格

+   限制打印的 `DataFrame` 行和列的数量

+   修改小数点数字的精度

+   截断单元格的文本内容

+   当数值低于某个下限时进行数值舍入

在我们处理本书的数据集的过程中，我们看到了 pandas 如何通过在数据展示上做出合理的决策来改善我们的用户体验。例如，当我们输出一个 1,000 行的 `DataFrame` 时，该库假设我们更愿意看到开始和结束部分的 30 行，而不是整个数据集，这可能会使屏幕变得杂乱。有时，我们可能想要打破 pandas 的假设并调整其设置以适应我们的自定义显示需求。幸运的是，该库公开了许多其内部设置供我们修改。在本章中，我们将学习如何配置选项，例如行和列限制、浮点精度和值舍入。让我们动手看看我们如何可以改变这些设置。

## 13.1 获取和设置 pandas 选项

我们将首先导入 pandas 库并将其分配一个别名 `pd`：

```
In  [1] import pandas as pd
```

本章的数据集，happiness.csv，是根据幸福感对世界各国进行排名。民意调查公司盖洛普在联合国的支持下收集了这些数据。每一行都包含一个国家的总幸福感得分以及人均国内生产总值（GDP）、社会支持、预期寿命和慷慨程度的个人得分。该数据集包含 6 列和 156 行：

```
In  [2] happiness = pd.read_csv("happiness.csv")
        happiness.head()

Out [2]

 ** Country  Score  GDP per cap...  Social sup...  Life expect...   Generosity**
0      Finland  7.769         1.340          1.587           0.986          0.153
1      Denmark  7.600         1.383          1.573           0.996          0.252
2       Norway  7.554         1.488          1.582           1.028          0.271
3      Iceland  7.494         1.380          1.624           1.026          0.354
4  Netherlands  7.488         1.396          1.522           0.999          0.322
```

Pandas 将其设置存储在库顶层的单个 `options` 对象中。每个选项都属于一个父类别。让我们从 `display` 类别开始，它包含 pandas 数据结构的打印表示的设置。

最高级别的 `describe_option` 函数返回给定设置的文档。我们可以传递一个包含设置名称的字符串。让我们看看 `max_rows` 选项，它嵌套在 `display` 父类别中。`max_rows` 设置配置了 pandas 在截断 `DataFrame` 之前打印的最大行数：

```
In  [3] pd.describe_option("display.max_rows")

Out [3]

        display.max_rows : int
            If max_rows is exceeded, switch to truncate view. Depending on
            `large_repr`, objects are either centrally truncated or printed
            as a summary view. 'None' value means unlimited.

            In case python/IPython is running in a terminal and
            `large_repr` equals 'truncate' this can be set to 0 and pandas
            will auto-detect the height of the terminal and print a
            truncated object which fits the screen height. The IPython
            notebook, IPython qtconsole, or IDLE do not run in a terminal
            and hence it is not possible to do correct auto-detection.
           [default: 60] [currently: 60]
```

注意，文档的末尾包括设置的默认值和当前值。

Pandas 将打印出所有与字符串参数匹配的库选项。该库使用正则表达式来比较 `describe_option` 的参数与其可用设置。提醒一下，*正则表达式* 是一种用于文本的搜索模式；请参阅附录 E 以获取详细概述。下一个示例传递了一个参数 `"max_col"`。Pandas 打印出与该术语匹配的两个设置的文档：

```
In  [4] pd.describe_option("max_col")

Out [4]

display.max_columns : int
    If max_cols is exceeded, switch to truncate view. Depending on
    `large_repr`, objects are either centrally truncated or printed as
    a summary view. 'None' value means unlimited.

    In case python/IPython is running in a terminal and `large_repr`
    equals 'truncate' this can be set to 0 and pandas will auto-detect
    the width of the terminal and print a truncated object which fits
    the screen width. The IPython notebook, IPython qtconsole, or IDLE
    do not run in a terminal and hence it is not possible to do
    correct auto-detection.
    [default: 20] [currently: 5]
display.max_colwidth : int or None
    The maximum width in characters of a column in the repr of
    a pandas data structure. When the column overflows, a "..."
    placeholder is embedded in the output. A 'None' value means unlimited.
    [default: 50] [currently: 9]
```

虽然正则表达式很有吸引力，但我建议写出设置的完整名称，包括其父类别。明确的代码往往会导致错误更少。

有两种方式来获取设置的当前值。第一种方式是 pandas 顶级层的 `get_option` 函数；像 `describe_option` 一样，它接受一个字符串参数，包含设置的名称。第二种方法是访问父类别和特定设置，作为顶级 `pd.options` 对象上的属性。

以下示例显示了两种策略的语法。这两行代码都返回 `max_rows` 设置的 60，这意味着 pandas 将截断任何长度超过 60 行的 `DataFrame` 输出：

```
In  [5] # The two lines below are equivalent
        pd.get_option("display.max_rows")
        pd.options.display.max_rows

Out [5] 60
```

同样，有两种方式可以*设置*配置设置的新的值。pandas 顶级层的 `set_option` 函数接受设置作为其第一个参数，其新值作为第二个参数。或者，我们可以通过 `pd.options` 对象上的属性访问选项，并用等号分配新值：

```
In  [6] # The two lines below are equivalent
        pd.set_option("display.max_rows", 6)
        pd.options.display.max_rows = 6
```

我们已指示 pandas 如果 `DataFrame` 输出超过六行，则截断输出：

```
In  [7] pd.options.display.max_rows

Out [7] 6
```

让我们看看实际的变化。下一个示例要求 pandas 打印幸福数据的头六行。六行最大行的阈值没有被越过，所以 pandas 没有截断地输出了 `DataFrame`：

```
In  [8] happiness.head(6)

Out [8]

 ** Country  Score  GDP per cap...  Social sup...  Life expect...   Generosity**
0      Finland  7.769         1.340          1.587           0.986          0.153
1      Denmark  7.600         1.383          1.573           0.996          0.252
2       Norway  7.554         1.488          1.582           1.028          0.271
3      Iceland  7.494         1.380          1.624           1.026          0.354
4  Netherlands  7.488         1.396          1.522           0.999          0.322
5  Switzerland  7.480         1.452          1.526           1.052          0.263
```

现在让我们越过阈值，让 pandas 打印幸福数据的头七行。库总是旨在在截断前后打印相同数量的行。在下一个示例中，它从输出开始打印三行，从输出末尾打印三行，截断中间的行（索引 3）：

```
In  [9] happiness.head(7)

Out [9]

 ** Country  Score  GDP per cap...  Social sup...  Life expect...   Generosity**
0      Finland  7.769         1.340            1.587         0.986          0.153
1      Denmark  7.600         1.383            1.573         0.996          0.252
2       Norway  7.554         1.488            1.582         1.028          0.271
...        ...    ...           ...              ...           ...            ...
4  Netherlands  7.488         1.396            1.522         0.999          0.322
5  Switzerland  7.480         1.452            1.526         1.052          0.263
6       Sweden  7.343         1.387            1.487         1.009          0.267

7 rows × 6 columns
```

`max_rows` 设置声明了打印的行数。互补的 `display.max_columns` 选项设置了打印的最大列数。默认值是 `20`：

```
In  [10] # The two lines below are equivalent
         pd.get_option("display.max_columns")
         pd.options.display.max_columns

Out [10] 20
```

再次，要分配新值，我们可以使用 `set_option` 函数或直接访问嵌套的 `max_columns` 属性：

```
In  [11] # The two lines below are equivalent
         pd.set_option("display.max_columns", 2)
         pd.options.display.max_columns = 2
```

如果我们设置偶数个最大列数，pandas 将从其最大列数中排除截断列。幸福 `DataFrame` 有六列，但以下输出只显示了其中两列。Pandas 包含第一列和最后一列，即国家和国民慷慨度，并在两者之间放置一个截断列：

```
In  [12] happiness.head(7)

Out [12]

 ** Country  ...  Generosity**
0      Finland            0.153
1      Denmark  ...       0.252
2       Norway  ...       0.271
...        ...  ...         ...
4  Netherlands  ...       0.322
5  Switzerland  ...       0.263
6       Sweden  ...       0.267

7 rows × 6 columns
```

如果我们设置奇数个最大列数，pandas 将包括截断列在其列计数中。奇数确保 pandas 可以在截断的两侧放置相等数量的列。下一个示例将 `max_columns` 值设置为 `5`。幸福输出显示了最左边的两列（国家得分），截断列，以及最右边的两列（预期寿命和国民慷慨度）。Pandas 打印了原始六列中的四列：

```
In  [13] # The two lines below are equivalent
         pd.set_option("display.max_columns", 5)
         pd.options.display.max_columns = 5

In  [14] happiness.head(7)

Out [14]

 ** Country  Score  ...  Life expectancy   Generosity**
0      Finland  7.769  ...            0.986        0.153
1      Denmark  7.600  ...            0.996        0.252
2       Norway  7.554  ...            1.028        0.271
...        ...    ...  ...              ...          ...
4  Netherlands  7.488  ...            0.999        0.322
5  Switzerland  7.480  ...            1.052        0.263
6       Sweden  7.343  ...            1.009        0.267

5 rows × 6 columns
```

要将设置恢复到其原始值，请将名称传递给 pandas 顶级层的 `reset_option` 函数。下一个示例重置了 `max_rows` 设置：

```
In  [15] pd.reset_option("display.max_rows")
```

我们可以通过再次调用 `get_option` 函数来确认更改：

```
In  [16] pd.get_option("display.max_rows")

Out [16] 60
```

Pandas 已将 `max_rows` 设置重置为其默认值 `60`。

## 13.2 精度

现在我们已经熟悉了 pandas 更改设置的 API，让我们来了解一下几个流行的配置选项。

`display.precision` 选项设置浮点数后的数字位数。默认值是 `6`：

```
In  [17] pd.describe_option("display.precision")

Out [17]

         display.precision : int
             Floating point output precision (number of significant
             digits). This is only a suggestion
             [default: 6] [currently: 6]
```

下一个示例将精度设置为 `2`。该设置影响幸福感中所有四个浮点列的值：

```
In  [18] # The two lines below are equivalent
         pd.set_option("display.precision", 2)
         pd.options.display.precision = 2

In  [19] happiness.head()

Out [19]

 ** Country  Score  ...  Life expectancy  Generosity**
0      Finland   7.77  ...             1.34        0.15
1      Denmark   7.60  ...             1.38        0.25
2       Norway   7.55  ...             1.49        0.27
3      Iceland   7.49  ...             1.38        0.35
4  Netherlands   7.49  ...             1.40        0.32

5 rows × 6 columns
```

`precision` 设置仅改变浮点数的表示形式。Pandas 在 `DataFrame` 中保留原始值，我们可以通过使用 `loc` 访问器从浮点列（如分数）中提取样本值来证明这一点：

```
In  [20] happiness.loc[0, "Score"]

Out [20] 7.769
```

分数列的原始值，`7.769`，仍然存在。当 pandas 打印 `DataFrame` 时，它会将值的表示形式更改为 `7.77`。

## 13.3 最大列宽

`display.max_colwidth` 设置设置 pandas 在截断单元格文本之前打印的最大字符数：

```
In  [21] pd.describe_option("display.max_colwidth")

Out [21]

         display.max_colwidth : int or None
             The maximum width in characters of a column in the repr of
             a pandas data structure. When the column overflows, a "..."
             placeholder is embedded in the output. A 'None' value means
             unlimited.
            [default: 50] [currently: 50]
```

下一个示例要求 pandas 在文本长度超过九个字符时截断文本：

```
In  [22] # The two lines below are equivalent
         pd.set_option("display.max_colwidth", 9)
         pd.options.display.max_colwidth = 9
```

让我们看看当我们输出幸福感时会发生什么：

```
In  [23] happiness.tail()

Out [23]

 **Country    Score  ...  Life expectancy  Generosity**
151          Rwanda     3.33  ...             0.61        0.22
152        Tanzania     3.23  ...             0.50        0.28
153          Afgha...   3.20  ...             0.36        0.16
154    Central Afr...   3.08  ...             0.10        0.23
155          South...   2.85  ...             0.29        0.20

5 rows × 6 columns
```

Pandas 缩短了最后三个国家值（阿富汗、中非共和国和南苏丹）。输出中的前两个值（卢旺达六个字符和坦桑尼亚八个字符）不受影响。

## 13.4 截断阈值

在某些分析中，如果值合理接近 `0`，我们可能会认为它们是无意义的。例如，您的业务领域可能认为值 `0.10` 是“与 `0` 一样好”或“实际上是 `0`”。`display.chop_threshold` 选项设置浮点值必须跨越的底限才能打印。Pandas 将显示低于阈值的任何值作为 `0`：

```
In  [24] pd.describe_option("display.chop_threshold")

Out [24]

         display.chop_threshold : float or None
             if set to a float value, all float values smaller then the
             given threshold will be displayed as exactly 0 by repr and
             friends.
            [default: None] [currently: None]
```

此示例将 `0.25` 设置为截断阈值：

```
In  [25] pd.set_option("display.chop_threshold", 0.25)
```

在下一个输出中，请注意，pandas 将寿命和慷慨列的索引 154（分别为 `0.105` 和 `0.235`）打印为输出中的 `0.00`：

```
In  [26] happiness.tail()

Out [26]

 **Country  Score  ...  Life expectancy  Generosity**
151          Rwanda   3.33  ...             0.61        0.00
152        Tanzania   3.23  ...             0.50        0.28
153     Afghanistan   3.20  ...             0.36        0.00
154  Central Afr...   3.08  ...             0.00        0.00
155     South Sudan   2.85  ...             0.29        0.00

5 rows × 6 columns
```

与 `precision` 设置类似，`chop_threshold` 不会更改 `DataFrame` 中的底层值——只会更改它们的打印表示形式。

## 13.5 选项上下文

我们迄今为止更改的设置都是全局的。当我们更改它们时，我们会改变之后执行的 Jupyter Notebook 单元格的输出。全局设置持续到我们为其分配新值为止。例如，如果我们将 `display.max_columns` 设置为 `6`，那么 Jupyter 将为所有未来的单元格执行输出最多六列的 `DataFrame`。

有时，我们可能希望为单个单元格自定义表示选项。我们可以使用 pandas 的顶级 `option_context` 函数来完成此任务。我们将该函数与 Python 的内置 `with` 关键字配对以创建上下文块。将 *上下文块* 想象为一个临时的执行环境。`option_context` 函数在代码块执行时设置 pandas 选项的临时值；全局 pandas 设置不受影响。

我们将设置作为顺序参数传递给 `option_context` 函数。下一个示例打印了幸福感 `DataFrame`：

+   `display.max_columns` 设置为 `5`

+   `display.max_rows` 设置为 `10`

+   `display.precision` 设置为 `3`

Jupyter 不识别 `with` 块的内容作为笔记本单元的最终语句。因此，我们需要使用名为 `display` 的笔记本函数来手动输出 `DataFrame`：

```
In  [27] with pd.option_context(
             "display.max_columns", 5,
             "display.max_rows", 10,
             "display.precision", 3
         ):
            display(happiness)

Out [27]

 **Country  Score  ...  Life expectancy  Generosity**
0           Finland  7.769  ...            0.986       0.153
1           Denmark  7.600  ...            0.996       0.252
2            Norway  7.554  ...            1.028       0.271
3           Iceland  7.494  ...            1.026       0.354
4       Netherlands  7.488  ...            0.999       0.322
...             ...    ...  ...              ...         ...
151          Rwanda  3.334  ...            0.614       0.217
152        Tanzania  3.231  ...            0.499       0.276
153     Afghanistan  3.203  ...            0.361       0.158
154  Central Afr...  3.083  ...            0.105       0.235
155     South Sudan  2.853  ...            0.295       0.202

156 rows × 6 columns
```

由于我们使用了 `with` 关键字，我们没有更改这三个选项的全局笔记本设置；它们保留了它们的原始值。

`option_context` 函数对于为不同的单元执行分配不同的选项很有帮助。如果您希望所有输出都有一致的表现，我建议在 Jupyter Notebook 的顶部单元中一次性设置选项。

## 摘要

+   `describe_option` 函数返回 pandas 设置的文档。

+   `set_option` 函数设置了一个设置的新的值。

+   我们也可以通过访问和覆盖 `pd.options` 对象上的属性来更改设置。

+   `reset_option` 函数将 pandas 设置改回其默认值。

+   `display.max_rows` 和 `display.max_columns` 选项设置 pandas 在输出中显示的最大行/列数。

+   `display.precision` 设置改变小数点后的数字位数。

+   `display.max_colwidth` 选项设置了一个数值阈值，当 pandas 打印字符时，超过此阈值的字符将被截断。

+   `display.chop_threshold` 选项设置了一个数值下限。如果值没有超过阈值，pandas 将它们打印为零。

+   将 `option_context` 函数和 `with` 关键字配对，以创建一个完全的临时执行上下文块。
