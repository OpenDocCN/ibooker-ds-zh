# 9 案例研究 2 的解决方案

本节涵盖

+   测量统计显著性

+   派生测试

+   使用 Pandas 操作表格

我们被要求分析我们的朋友弗雷德收集的在线广告点击数据。他的广告数据表监控了 30 种不同颜色的广告点击。我们的目标是发现一种比蓝色产生更多点击的广告颜色。我们将通过以下步骤来实现：

1.  使用 Pandas 加载并清理我们的广告数据。

1.  在蓝色和其他记录的颜色之间运行派生测试。

1.  使用适当确定的显著性水平检查计算出的 p 值的统计显著性。

警告：剧透警告！案例研究 2 的解决方案即将揭晓。我强烈建议你在阅读解决方案之前尝试解决问题。原始问题陈述可在案例研究开始处参考。

## 9.1 在 Pandas 中处理广告点击表

让我们先加载我们的广告点击表到 Pandas。然后我们检查表中的行数和列数。

列表 9.1 将广告点击表加载到 Pandas

```
df = pd.read_csv('colored_ad_click_table.csv')
num_rows, num_cols = df.shape
print(f"Table contains {num_rows} rows and {num_cols} columns")

Table contains 30 rows and 41 columns
```

我们的表包含 30 行和 41 列。行应与每天的颜色点击次数和观看次数相对应。让我们通过检查列名称来确认。

列表 9.2 检查列名称

```
print(df.columns)

Index(['Color', 'Click Count: Day 1', 'View Count: Day 1',
       'Click Count: Day 2', 'View Count: Day 2', 'Click Count: Day 3',
       'View Count: Day 3', 'Click Count: Day 4', 'View Count: Day 4',
       'Click Count: Day 5', 'View Count: Day 5', 'Click Count: Day 6',
       'View Count: Day 6', 'Click Count: Day 7', 'View Count: Day 7',
       'Click Count: Day 8', 'View Count: Day 8', 'Click Count: Day 9',
       'View Count: Day 9', 'Click Count: Day 10', 'View Count: Day 10',
       'Click Count: Day 11', 'View Count: Day 11', 'Click Count: Day 12',
       'View Count: Day 12', 'Click Count: Day 13', 'View Count: Day 13',
       'Click Count: Day 14', 'View Count: Day 14', 'Click Count: Day 15',
       'View Count: Day 15', 'Click Count: Day 16', 'View Count: Day 16',
       'Click Count: Day 17', 'View Count: Day 17', 'Click Count: Day 18',
       'View Count: Day 18', 'Click Count: Day 19', 'View Count: Day 19',
       'Click Count: Day 20', 'View Count: Day 20'],
      dtype='object')
```

列与我们的预期一致：第一列包含所有分析的颜色，其余 40 列包含实验每一天的点击次数和观看次数。作为一个理智的检查，让我们检查我们表中存储的数据质量。我们首先输出分析的颜色名称。

列表 9.3 检查颜色名称

```
print(df.Color.values)

['Pink' 'Gray' 'Sapphire' 'Purple' 'Coral' 'Olive' 'Navy' 'Maroon' 'Teal'
 'Cyan' 'Orange' 'Black' 'Tan' 'Red' 'Blue' 'Brown' 'Turquoise' 'Indigo'
 'Gold' 'Jade' 'Ultramarine' 'Yellow' 'Viridian' 'Violet' 'Green'
 'Aquamarine' 'Magenta' 'Silver' 'Bronze' 'Lime']
```

在 *颜色* 列中存在 30 种常见的颜色。每种颜色名称的首字母都大写。因此，我们可以通过执行 `assert 'Blue' in df.Color` 来确认颜色蓝色是否存在。

列表 9.4 检查蓝色

```
assert 'Blue' in df.Color.values
```

基于字符串的 *颜色* 列看起来很好。让我们将注意力转向剩余的 40 个数值列。输出所有 40 个列会导致大量数据。相反，我们将检查实验第一天的列：`Click Count: Day 1` 和 `View Count: Day 1`。我们选择这两个列，并使用 `describe()` 来总结其内容。

列表 9.5 总结实验第一天

```
selected_columns = ['Color', 'Click Count: Day 1', 'View Count: Day 1']
print(df[selected_columns].describe())

       Click Count: Day 1  View Count: Day 1
count           30.000000               30.0
mean            23.533333              100.0
std              7.454382                0.0
min             12.000000              100.0
25%             19.250000              100.0
50%             24.000000              100.0
75%             26.750000              100.0
max             49.000000              100.0
```

`Click Count: Day 1` 列的值从 12 到 49 点击不等。同时，`View Count: Day 1` 的最小值和最大值都是 100 次观看。因此，该列的所有值都是 100 次观看。这种行为是预期的。我们特别被告知每种颜色每天都会收到 100 次观看。让我们确认所有日观看次数都等于 100。

列表 9.6 确认等效日观看次数

```
view_columns = [column for column in df.columns if 'View' in column]
assert np.all(df[view_columns].values == 100)                          ❶
```

❶ 确保 NumPy 数组中值等于 100 的高效代码

所有观看次数都等于 100。因此，所有的 20 个 `View Count` 列都是冗余的。我们可以从表中删除它们。

列表 9.7 从表中删除观看次数

```
df.drop(columns=view_columns, inplace=True)

print(df.columns)
Index(['Color', 'Click Count: Day 1', 'Click Count: Day 2',
       'Click Count: Day 3', 'Click Count: Day 4', 'Click Count: Day 5',
       'Click Count: Day 6', 'Click Count: Day 7', 'Click Count: Day 8',
       'Click Count: Day 9', 'Click Count: Day 10', 'Click Count: Day 11',
       'Click Count: Day 12', 'Click Count: Day 13', 'Click Count: Day 14',
       'Click Count: Day 15', 'Click Count: Day 16', 'Click Count: Day 17',
       'Click Count: Day 18', 'Click Count: Day 19', 'Click Count: Day 20'],
      dtype='object')
```

已删除冗余列。仅保留颜色和点击次数数据。我们的 20 个`点击次数`列对应于每天每 100 次查看的点击次数，因此我们可以将这些计数视为百分比。实际上，每行的颜色被映射到每天广告点击的百分比。让我们总结蓝色广告的每日广告点击百分比。要生成该摘要，我们按颜色索引每一行，然后调用`df.T.Blue.describe()`。

列表 9.8 总结每日蓝色点击统计数据

```
df.set_index('Color', inplace=True)
print(df.T.Blue.describe())

count    20.000000
mean     28.350000
std       5.499043
min      18.000000
25%      25.750000
50%      27.500000
75%      30.250000
max      42.000000
Name: Blue, dtype: float64
```

蓝色广告的每日点击百分比范围从 18%到 42%。平均点击百分比为 28.35%：平均而言，28.35%的蓝色广告在每次查看时都会被点击。这个平均点击率相当不错。它与其他 29 种颜色相比如何？我们准备找出答案。

## 9.2 从均值差异计算 p 值

让我们先过滤数据。我们删除蓝色，留下其他 29 种颜色。然后我们转置我们的表格，通过列名访问颜色。

列表 9.9 创建无蓝色表格

```
df_not_blue = df.T.drop(columns='Blue')
print(df_not_blue.head(2))

Color               Pink  Gray  Sapphire  Purple  Coral  Olive  Navy  Maroon  \
Click Count: Day 1    21    27        30      26     26     26    38      21
Click Count: Day 2    20    27        32      21     24     19    29      29

Color               Teal  Cyan  ...  Ultramarine  Yellow  Viridian  Violet  \
Click Count: Day 1    25    24  ...           49      14       27      15
Click Count: Day 2    25    22  ...           41      24       23      22

Color               Green  Aquamarine  Magenta  Silver  Bronze  Lime
Click Count: Day 1     14          24       18      26      19    20
Click Count: Day 2     25          28       21      24      19    19

[2 rows x 29 columns]
```

我们的`df_not_blue`表包含 29 种颜色的点击百分比。我们想将这些百分比与我们的蓝色百分比进行比较。更确切地说，我们想知道是否存在一种颜色的平均点击率与蓝色的平均点击率在统计上不同。我们如何比较这些均值？每个颜色的样本均值很容易获得，但我们没有总体均值。因此，我们最好的选择是运行一个排列测试。要运行测试，我们需要定义一个可重用的排列测试函数。该函数将接受两个 NumPy 数组作为输入，并返回一个 p 值作为输出。

列表 9.10 定义排列测试函数

```
def permutation_test(data_array_a, data_array_b):
    data_mean_a = data_array_a.mean()
    data_mean_b = data_array_b.mean()
    extreme_mean_diff = abs(data_mean_a - data_mean_b)                   ❶
    total_data = np.hstack([data_array_a, data_array_b])
    number_extreme_values = 0.0
    for _ in range(30000):
        np.random.shuffle(total_data)
        sample_a = total_data[:data_array_a.size]
        sample_b = total_data[data_array_a.size:]
        if abs(sample_a.mean() - sample_b.mean()) >= extreme_mean_diff:  ❷
            number_extreme_values += 1

    p_value = number_extreme_values / 30000
    return p_value
```

❶ 样本均值之间的观察差异

❷ 重采样均值之间的差异非常大。

我们将在蓝色和其他 29 种颜色之间运行一个排列测试。然后，我们将根据它们的 p 值结果对这些颜色进行排序。我们的输出以热图（图 9.1）的形式可视化，以更好地强调 p 值之间的差异。

列表 9.11 在颜色上运行排列测试

```
np.random.seed(0)
blue_clicks = df.T.Blue.values
color_to_p_value = {}
for color, color_clicks in df_not_blue.items():
    p_value = permutation_test(blue_clicks, color_clicks)
    color_to_p_value[color] = p_value
sorted_colors, sorted_p_values = zip(*sorted(color_to_p_value.items(),   ❶
                                             key=lambda x: x[1]))
plt.figure(figsize=(3, 10))                                              ❷
sns.heatmap([[p_value] for p_value in sorted_p_values],                  ❸
            cmap='YlGnBu', annot=True, xticklabels=['p-value'],
            yticklabels=sorted_colors)
plt.show()
```

❶ 高效的 Python 代码用于排序字典并返回两个列表：一个排序后的值列表和一个关联的键列表。每个排序后的 p 值在位置 i 与 sorted_colors[i]中的颜色对齐。

❷ 调整绘制的热图的宽度和高度分别为 3 英寸和 10 英寸。这些调整提高了热图可视化的质量。

❸ sns.heatmap 方法接受一个二维表作为输入。因此，我们将我们的包含 29 行 1 列的 p 值一维列表转换为二维表。

![图片](img/09-01.png)

图 9.1 排列测试返回的 p 值/颜色对的热图：21 种颜色映射到小于 0.05 的 p 值。

大多数颜色生成的 p 值明显低于 0.05。黑色具有最低的 p 值：其广告点击百分比必须与蓝色有显著差异。但从设计角度来看，黑色不是一个非常易于点击的颜色。文本链接通常不是黑色，因为黑色链接很难与普通文本区分开来。这里有些可疑的事情正在发生：黑色和蓝色记录的点击数到底有什么区别？我们可以通过打印`df_not_blue.Black.mean()`来检查。

列表 9.12 查找黑色的平均点击率

```
mean_black = df_not_blue.Black.mean()
print(f"Mean click-rate of black is {mean_black}")

Mean click-rate of black is 21.6
```

黑色的平均点击率为 21.6。这个值明显低于蓝色的平均点击率 28.35。因此，颜色之间的统计差异是由点击黑色的人数较少造成的。也许其他低 p 值也是由于低点击率造成的。让我们过滤掉那些平均值低于蓝色平均值的颜色，然后打印剩余的颜色。

列表 9.13 使用低点击率过滤颜色

```
remaining_colors = df[df.T.mean().values > blue_clicks.mean()].index    ❶
size = remaining_colors.size
print(f"{size} colors have on average more clicks than Blue.")
print("These colors are:")
print(remaining_colors.values)

5 colors have on average more clicks than Blue.
These colors are:
['Sapphire' 'Navy' 'Teal' 'Ultramarine' 'Aquamarine']
```

❶ 高效的一行代码用于过滤颜色。首先，代码创建一个布尔数组。该数组指定哪些颜色的平均值大于蓝色。布尔数组被输入到 df 中进行过滤。过滤结果的索引指定了剩余的颜色名称。

剩余的颜色只有五种。这些颜色都是不同色调的蓝色。让我们打印这五种剩余颜色的排序后的 p 值；我们也打印平均点击次数以方便分析。

列表 9.14 打印剩余的五种颜色

```
for color, p_value in sorted(color_to_p_value.items(), key=lambda x: x[1]):
    if color in remaining_colors:
        mean = df_not_blue[color].mean()
        print(f"{color} has a p-value of {p_value} and a mean of {mean}")

Ultramarine has a p-value of 0.0034 and a mean of 34.2
Navy has a p-value of 0.5911666666666666 and a mean of 29.3
Aquamarine has a p-value of 0.6654666666666667 and a mean of 29.2
Sapphire has a p-value of 0.7457666666666667 and a mean of 28.9
Teal has a p-value of 0.9745 and a mean of 28.45
```

## 9.3 确定统计显著性

四种颜色的 p 值较大。只有一种颜色的 p 值较小。这种颜色是群青色：一种特殊的蓝色色调。其平均值为 34.2，大于蓝色的平均值为 28.35。群青色的 p 值为 0.0034。这个 p 值在统计上显著吗？嗯，它比标准显著性水平 0.05 低 10 倍以上。然而，这个显著性水平并没有考虑到我们与蓝色和其他 29 种颜色的比较。每一次比较都是一个实验，测试颜色是否与蓝色不同。如果我们进行足够的实验，那么我们迟早会遇到一个低 p 值。纠正这种情况的最佳方法是执行 Bonferroni 校正——否则，我们将成为 p 值黑客的受害者。要执行 Bonferroni 校正，我们将显著性水平降低到 0.05 / 29。

列表 9.15 应用 Bonferroni 校正

```
significance_level = 0.05 / 29
print(f"Adjusted significance level is {significance_level}")
if color_to_p_value['Ultramarine'] <= significance_level:
    print("Our p-value is statistically significant")
else:
    print("Our p-value is not statistically significant")

Adjusted significance level is 0.001724137931034483
Our p-value is not statistically significant
```

我们的 p 值不具有统计学意义——弗雷德为我们进行了太多的实验，以至于我们无法得出有意义的结论。没有有效的理由预期黑色、棕色或灰色会优于蓝色。也许如果弗雷德忽略了一些这些颜色，我们的分析可能会更有成效。可能的话，如果弗雷德仅仅比较蓝色与其他五种蓝色变体，我们可能会得到一个具有统计学意义的结论。让我们探索这样一个假设情况：弗雷德发起五次实验，而群青的 p 值保持不变。

列表 9.16 探索一个假设的显著性水平

```
hypothetical_sig_level = 0.05 / 5
print(f"Hypothetical significance level is {hypothetical_sig_level}")
if color_to_p_value['Ultramarine'] <= hypothetical_sig_level:
    print("Our hypothetical p-value would have been statistically significant")
else:
    print("Our hypothetical p-value would not have been statistically
           significant")

Hypothetical significance level is 0.01
Our hypothetical p-value would have been statistically significant
```

在这些假设条件下，我们的结果将是具有统计学意义的。遗憾的是，我们无法使用假设条件来降低我们的显著性水平。我们无法保证重新进行实验会重现 0.0034 的 p 值。p 值会波动，过多的实验增加了不可信波动的可能性。鉴于弗雷德的高实验数量，我们根本无法得出具有统计学意义的结论。

然而，并非一切尽失。群青仍然代表了一种有希望的蓝色替代品。弗雷德是否应该执行这种替代？也许吧。让我们考虑我们的两种替代场景。在第一种场景中，零假设是正确的。如果是这样，那么蓝色和群青共享相同的总体均值。在这种情况下，用群青替换蓝色不会影响广告点击率。在第二种场景中，较高的群青点击率实际上是具有统计学意义的。如果是这样，那么用群青替换蓝色将产生更多的广告点击。因此，弗雷德通过将所有广告设置为群青，有得无失。

从逻辑角度来看，弗雷德确实应该用群青替换蓝色。但如果他进行替换，仍会存在一些不确定性；弗雷德永远不知道群青是否真的比蓝色带来更多的点击。如果弗雷德的好奇心占了上风怎么办？如果他真的想要一个答案，他唯一的选择是再进行一次实验。在那个实验中，一半显示的广告将是蓝色，另一半显示的广告将是群青。弗雷德的软件将展示广告，同时记录所有的点击和观看次数。然后我们可以重新计算 p 值，并将其与适当的显著性水平进行比较，该水平将保持在 0.05。由于只进行一次实验，因此不需要 Bonferroni 校正。在 p 值比较之后，弗雷德最终将知道群青是否优于蓝色。

## 9.4 41 种蓝色的阴影：一个现实生活中的警示故事

弗雷德假设分析每一种颜色都会产生更有影响力的结果，但他错了。更多的数据并不一定是更好的：有时更多的数据会导致更多的不确定性。

弗雷德不是统计学家。他未能理解过度分析的后果可以原谅。然而，今天某些在商业领域运作的量化专家却不能这么说。以一个著名公司发生的著名事件为例。该公司需要为其网站上的链接选择一种颜色。首席设计师选择了一种视觉上吸引人的蓝色色调，但一位高级高管不信任这个决定。设计师为什么选择这种蓝色而不是其他颜色呢？

该高管来自量化背景，并坚持认为应该通过一次大规模的分析测试来科学地选择链接颜色，以确定完美的蓝色色调。公司网站链接完全随机分配了 41 种蓝色，并记录了数百万次点击。最终，基于每次查看的最大点击量，选定了“最佳”的蓝色色调。

该高管继续将这种方法公之于众。全球统计学家都感到不安。高管的决策揭示了其对基本统计学的无知，这种无知既让高管也使公司感到尴尬。

## 摘要

+   数据量并不总是越多越好。进行无意义的过量分析测试增加了出现异常结果的可能性。

+   在进行分析之前花时间思考问题是有价值的。如果弗雷德仔细考虑了 31 种颜色，他就会发现测试它们所有颜色是毫无意义的。许多颜色会使链接看起来很丑。像黑色这样的颜色不太可能比蓝色带来更多的点击量。过滤颜色集将导致更有信息量的测试。

+   尽管弗雷德的实验存在缺陷，我们仍然设法提取了一个有用的见解。群青可能证明是蓝色的合理替代品，尽管还需要更多的测试。偶尔，数据科学家会遇到有缺陷的数据，但仍然可能得出有价值的见解。
