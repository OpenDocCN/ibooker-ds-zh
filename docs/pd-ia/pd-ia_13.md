# 11 处理日期和时间

本章涵盖

+   将字符串 `Series` 转换为日期时间

+   从日期时间对象检索日期和时间信息

+   将日期四舍五入到周、月和季度末

+   在日期时间之间添加和减去

一个 *datetime* 是用于存储日期和时间的数据类型。它可以表示一个特定的日期（例如 2021 年 10 月 4 日），一个特定的时间（例如上午 11:50），或者两者（例如 2021 年 10 月 4 日上午 11:50）。日期时间非常有价值，因为它们允许我们跟踪时间趋势。一个金融分析师可能会使用日期时间来确定股票表现最佳的星期几。一个餐馆老板可能会使用它们来发现顾客光顾业务的高峰时段。一个运营经理可能会使用它们来识别生产中造成瓶颈的过程部分。数据集中的 *何时* 常常可以引导到 *为什么*。

在本章中，我们将回顾 Python 的内置 datetime 对象，并了解 pandas 如何通过其 `Timestamp` 和 `Timedelta` 对象来改进它们。我们还将学习如何使用该库将字符串转换为日期，添加和减去时间偏移量，计算持续时间，等等。没有时间可以浪费（这里有个双关语），让我们开始吧。

## 11.1 介绍 Timestamp 对象

一个 *模块* 是一个包含 Python 代码的文件。Python 的标准库是语言中内置的超过 250 个模块的集合，它们为常见问题提供经过实战检验的解决方案，例如数据库连接、数学和测试。标准库的存在是为了让开发者能够编写使用核心语言特性的软件，而不是安装额外的依赖项。常有人说 Python “自带电池”；就像玩具一样，语言可以直接使用。

### 11.1.1 Python 如何处理日期时间

为了减少内存消耗，Python 默认不会自动加载其标准库模块。相反，我们必须明确地将任何所需的模块导入到我们的项目中。与外部包（如 pandas）一样，我们可以使用 `import` 关键字导入一个模块，并使用 `as` 关键字为其分配别名。标准库的 `datetime` 模块是我们的目标；它存储用于处理日期和时间的类。`dt` 是 `datetime` 模块的流行别名。让我们启动一个新的 Jupyter Notebook 并导入 `datetime` 以及 `pandas` 库：

```
In  [1] import datetime as dt
        import pandas as pd
```

让我们回顾模块中的四个类：`date`、`time`、`datetime` 和 `timedelta`。（有关类和对象的更多详细信息，请参阅附录 B。）

`date` 模型历史中的一个单独的一天。该对象不存储任何时间。`date` 类构造函数接受顺序的 `year`、`month` 和 `day` 参数。所有参数都期望是整数。下一个示例为我的生日，1991 年 4 月 12 日，实例化一个 `date` 对象：

```
In  [2] # The two lines below are equivalent
        birthday = dt.date(1991, 4, 12)
        birthday = dt.date(year = 1991, month = 4, day = 12)
        birthday

Out [2] datetime.date(1991, 4, 12)
```

`date` 对象将构造函数的参数保存为对象属性。我们可以通过 `year`、`month` 和 `day` 属性来访问它们的值：

```
In  [3] birthday.year

Out [3] 1991

In  [4] birthday.month

Out [4] 4

In  [5] birthday.day

Out [5] 12
```

`date`对象是*不可变的*——我们创建后不能更改其内部状态。如果我们尝试覆盖任何`date`属性，Python 将引发`AttributeError`异常：

```
In  [6] birthday.month = 10

---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-15-2690a31d7b19> in <module>
----> 1 birthday.month = 10

AttributeError: attribute 'month' of 'datetime.date' objects is not writable
```

补充的`time`类表示一天中的特定时间。日期无关紧要。`time`构造函数的前三个参数接受整数参数，用于`hour`、`minute`和`second`。与`date`对象一样，`time`对象是不可变的。下一个示例实例化一个`time`对象，表示上午 6:43:25：

```
In  [7] # The two lines below are equivalent
        alarm_clock = dt.time(6, 43, 25)
        alarm_clock = dt.time(hour = 6, minute = 43, second = 25)
        alarm_clock

Out [7] datetime.time(6, 43, 25)
```

所有三个参数的默认值都是 0。如果我们不带参数实例化一个`time`对象，它将代表午夜（凌晨 12:00:00）。午夜是一天的 0 小时、0 分钟和 0 秒：

```
In  [8] dt.time()

Out [8] datetime.time(0, 0)
```

下一个示例将 9 传递给`hour`参数，42 传递给`second`参数，没有为`minute`参数传递值。`time`对象将`minutes`值替换为 0。得到的时间是上午 9:00:42：

```
In  [9] dt.time(hour = 9, second = 42)

Out [9] datetime.time(9, 0, 42)
```

`time`构造函数使用 24 小时制时钟；我们可以传递大于或等于 12 的`hour`值来表示下午或晚上的时间。下一个示例表示 19:43:22 或等价于晚上 7:43:22：

```
In  [10] dt.time(hour = 19, minute = 43, second = 22)

Out [10] datetime.time(19, 43, 22)
```

`time`对象将构造函数参数保存为对象属性。我们可以使用`hour`、`minute`和`second`属性访问它们的值：

```
In  [11] alarm_clock.hour

Out [11] 6

In  [12] alarm_clock.minute

Out [12] 43

In  [13] alarm_clock.second

Out [13] 25
```

接下来是`datetime`对象，它包含日期和时间。它的前六个参数是`year`、`month`、`day`、`hour`、`minute`和`second`：

```
In  [14] # The two lines below are equivalent
         moon_landing = dt.datetime(1969, 7, 20, 22, 56, 20)
         moon_landing = dt.datetime(
             year = 1969,
             month = 7,
             day = 20,
             hour = 22,
             minute = 56,
             second = 20
         )
         moon_landing

Out [14] datetime.datetime(1969, 7, 20, 22, 56, 20)
```

`year`、`month`和`day`参数是必需的。与时间相关的属性是可选的，默认值为`0`。下一个示例表示 2020 年 1 月 1 日凌晨（12:00:00 a.m.）。我们明确传递了`year`、`month`和`day`参数；`hour`、`minute`和`second`参数隐式地回退到`0`：

```
In  [15] dt.datetime(2020, 1, 1)

Out [15] datetime.datetime(2020, 1, 1, 0, 0)
```

我们从`datetime`模块中的最后一个值得注意的对象是`timedelta`，它表示一个持续时间——时间的长度。其构造函数的参数包括`weeks`、`days`和`hours`。所有参数都是可选的，默认值为`0`。构造函数将时间长度相加以计算总持续时间。在下一个示例中，我们添加了 8 周和 6 天，总共 62 天（8 周 * 7 天 + 6 天）。Python 还添加了 3 小时、58 分钟和 12 秒，总共有 14,292 秒（238 分钟 * 60 秒 + 12 秒）：

```
In  [16] dt.timedelta(
             weeks = 8,
             days = 6,
             hours = 3,
             minutes = 58,
             seconds = 12
         )

Out [16] datetime.timedelta(days=62, seconds=14292)
```

现在我们已经熟悉了 Python 如何表示日期、时间和持续时间，让我们来探索 pandas 如何在此基础上构建这些概念。

### 11.1.2 Pandas 如何处理日期时间

Python 的`datetime`模块受到了一些批评。一些常见的投诉包括

+   需要跟踪大量的模块。我们只在本章中介绍了`datetime`，但还有其他模块可用于日历、时间转换、实用函数等。

+   需要记住大量的课程。

+   复杂、困难的对象 API 用于时区逻辑。

Pandas 引入了 `Timestamp` 对象作为 Python 的 `datetime` 对象的替代品。我们可以将 `Timestamp` 和 `datetime` 对象视为兄弟；在 pandas 生态系统中，它们经常可以互换，例如作为方法参数传递。就像 `Series` 扩展了 Python 列表一样，`Timestamp` 为更原始的 `datetime` 对象添加了功能。随着我们进入本章，我们将看到一些这些特性：

`Timestamp` 构造函数在 pandas 的顶层可用；它接受与 `datetime` 构造函数相同的参数。三个与日期相关的参数（`year`、`month` 和 `day`）是必需的。与时间相关的参数是可选的，默认为 `0.` 这里，我们再次模拟 1991 年 4 月 12 日，一个辉煌的日子：

```
In  [17] # The two lines below are equivalent
         pd.Timestamp(1991, 4, 12)
         pd.Timestamp(year = 1991, month = 4, day = 12)

Out [17] Timestamp('1991-04-12 00:00:00')
```

Pandas 认为如果两个对象存储相同的信息，则 `Timestamp` 等于 `date`/`datetime`。我们可以使用 `==` 符号来比较对象相等性：

```
In  [18] (pd.Timestamp(year = 1991, month = 4, day = 12)
            == dt.date(year = 1991, month = 4, day = 12))

Out [18] True

In  [19] (pd.Timestamp(year = 1991, month = 4, day = 12, minute = 2)
            == dt.datetime(year = 1991, month = 4, day = 12, minute = 2))

Out [19] True
```

如果日期或时间有任何差异，两个对象将不相等。下一个示例使用 `minute` 值为 `2` 的 `Timestamp` 和 `minute` 值为 `1` 的 `datetime` 实例化。相等比较的结果为 `False`：

```
In  [20] (pd.Timestamp(year = 1991, month = 4, day = 12, minute = 2)
            == dt.datetime(year = 1991, month = 4, day = 12, minute = 1))

Out [20] False
```

`Timestamp` 构造函数非常灵活，接受各种输入。下一个示例将字符串传递给构造函数而不是整数序列。文本存储了一个日期，格式为 YYYY-MM-DD（四位年份，两位月份，两位日期）。Pandas 正确地解析了输入中的月份、日期和年份：

```
In  [21] pd.Timestamp("2015-03-31")

Out [21] Timestamp('2015-03-31 00:00:00')
```

Pandas 识别许多标准的 datetime 字符串格式。下一个示例将日期字符串中的破折号替换为斜杠：

```
In  [22] pd.Timestamp("2015/03/31")

Out [22] Timestamp('2015-03-31 00:00:00')
```

下一个示例传递一个 MM/DD/YYYY 格式的字符串，这对 pandas 来说没问题：

```
In  [23] pd.Timestamp("03/31/2015")

Out [23] Timestamp('2015-03-31 00:00:00')
```

我们还可以以各种书面格式包含时间：

```
In  [24] pd.Timestamp("2021-03-08 08:35:15")

Out [24] Timestamp('2021-03-08 08:35:15')

In  [25] pd.Timestamp("2021-03-08 6:13:29 PM")

Out [25] Timestamp('2021-03-08 18:13:29')
```

最后，`Timestamp` 构造函数接受 Python 的原生 `date`、`time` 和 `datetime` 对象。下一个示例从 `datetime` 对象解析数据：

```
In  [26] pd.Timestamp(dt.datetime(2000, 2, 3, 21, 35, 22))

Out [26] Timestamp('2000-02-03 21:35:22')
```

`Timestamp` 对象实现了所有 `datetime` 属性，如 `hour`、`minute` 和 `second`。下一个示例将之前的 `Timestamp` 保存到变量中，然后输出几个属性：

```
In  [27] my_time = pd.Timestamp(dt.datetime(2000, 2, 3, 21, 35, 22))
         print(my_time.year)
         print(my_time.month)
         print(my_time.day)
         print(my_time.hour)
         print(my_time.minute)
         print(my_time.second)
Out [27] 2000
         2
         3
         21
         35
         22
```

Pandas 尽力确保其 datetime 对象与 Python 内置的类似。我们可以认为这些对象在 pandas 操作中是有效可互换的。

## 11.2 在 DatetimeIndex 中存储多个时间戳

*索引* 是附加到 pandas 数据结构上的标识标签集合。我们迄今为止遇到的最常见的索引是 `RangeIndex`，它是一系列升序或降序的数值。我们可以通过 `index` 属性访问 `Series` 或 `DataFrame` 的索引：

```
In  [28] pd.Series([1, 2, 3]).index

Out [28] RangeIndex(start=0, stop=3, step=1)
```

Pandas 使用一个 `Index` 对象来存储一系列字符串标签。在下一个示例中，请注意 pandas 附加到 `Series` 的索引对象会根据其内容而变化：

```
In  [29] pd.Series([1, 2, 3], index = ["A", "B", "C"]).index

Out [29] Index(['A', 'B', 'C'], dtype='object')
```

`DatetimeIndex` 是用于存储 `Timestamp` 对象的索引。如果我们向 `Series` 构造函数的 `index` 参数传递一个 `Timestamp`s 的列表，pandas 将将 `DatetimeIndex` 附加到 `Series`：

```
In  [30] timestamps = [
             pd.Timestamp("2020-01-01"),
             pd.Timestamp("2020-02-01"),
             pd.Timestamp("2020-03-01"),
         ]

         pd.Series([1, 2, 3], index = timestamps).index

Out [30] DatetimeIndex(['2020-01-01', '2020-02-01', '2020-03-01'],
         dtype='datetime64[ns]', freq=None)
```

如果我们传递一个 Python `datetime` 对象的列表，Pandas 也会使用 `DatetimeIndex`：

```
In  [31] datetimes = [
             dt.datetime(2020, 1, 1),
             dt.datetime(2020, 2, 1),
             dt.datetime(2020, 3, 1),
         ]

         pd.Series([1, 2, 3], index = datetimes).index

Out [31] DatetimeIndex(['2020-01-01', '2020-02-01', '2020-03-01'],
         dtype='datetime64[ns]', freq=None)
```

我们也可以从头创建一个 `DatetimeIndex`。其构造函数位于 pandas 的顶层。构造函数的 `data` 参数接受任何日期的可迭代集合。我们可以将日期作为字符串、datetimes、`Timestamp`s 或甚至数据类型的混合传递。Pandas 将将所有值转换为等效的 `Timestamp`s 并存储在索引中：

```
In  [32] string_dates = ["2018/01/02", "2016/04/12", "2009/09/07"]
         pd.DatetimeIndex(data = string_dates)

Out [32] DatetimeIndex(['2018-01-02', '2016-04-12', '2009-09-07'],
         dtype='datetime64[ns]', freq=None)

In  [33] mixed_dates = [
             dt.date(2018, 1, 2),
             "2016/04/12",
             pd.Timestamp(2009, 9, 7)
         ]

         dt_index = pd.DatetimeIndex(mixed_dates)
         dt_index

Out [33] DatetimeIndex(['2018-01-02', '2016-04-12', '2009-09-07'],
         dtype='datetime64[ns]', freq=None)
```

现在我们已经将 `DatetimeIndex` 分配给 `dt_index` 变量，让我们将其附加到 pandas 数据结构中。下一个示例将索引连接到一个样本 `Series`：

```
In  [34] s = pd.Series(data = [100, 200, 300], index = dt_index)
         s

Out [34] 2018-01-02    100
         2016-04-12    200
         2009-09-07    300
         dtype: int64
```

只有当我们将值存储为 `Timestamp`s 而不是字符串时，pandas 才能执行日期和时间相关的操作。Pandas 无法从像 `"2018-01-02"` 这样的字符串中推断出星期几，因为它将其视为数字和短划线的集合，而不是实际的日期。这就是为什么在第一次导入数据集时，将所有相关字符串列转换为日期时间至关重要：

我们可以使用 `sort_index` 方法按升序或降序排序 `DatetimeIndex`。下一个示例按升序（从最早到最新）排序索引日期：

```
In  [35] s.sort_index()

Out [35] 2009-09-07    300
         2016-04-12    200
         2018-01-02    100
         dtype: int64
```

Pandas 在排序或比较日期时间时考虑日期和时间。如果两个 `Timestamp`s 使用相同的日期，pandas 将比较它们的时、分、秒等：

对于 `Timestamp`s，有各种排序和比较操作可用。例如，小于符号（`<`）检查一个 `Timestamp` 是否早于另一个：

```
In  [36] morning = pd.Timestamp("2020-01-01 11:23:22 AM")
         evening = pd.Timestamp("2020-01-01 11:23:22 PM")

         morning < evening

Out [36] True
```

在 11.7 节中，我们将学习如何将这些类型的比较应用于 `Series` 中的所有值。

## 11.3 将列或索引值转换为日期时间

我们本章的第一个数据集，disney.csv，包含了华特迪士尼公司近 60 年的股价，这是世界上最知名娱乐品牌之一。每一行包括一个日期，该日股票的最高价和最低价，以及开盘价和收盘价：

```
In  [37] disney = pd.read_csv("disney.csv")
         disney.head()

Out [37]

 ** Date      High       Low      Open     Close**
0  1962-01-02  0.096026  0.092908  0.092908  0.092908
1  1962-01-03  0.094467  0.092908  0.092908  0.094155
2  1962-01-04  0.094467  0.093532  0.094155  0.094155
3  1962-01-05  0.094779  0.093844  0.094155  0.094467
4  1962-01-08  0.095714  0.092285  0.094467  0.094155
```

`read_csv` 函数默认将非数字列的所有值导入为字符串。我们可以通过 `DataFrame` 的 `dtypes` 属性来查看列的数据类型。注意，日期列的数据类型为 `"object"`，这是 pandas 对字符串的指定：

```
In  [38] disney.dtypes

Out [38] Date      object
         High     float64
         Low      float64
         Open     float64
         Close    float64
         dtype: object
```

我们必须明确告诉 pandas 哪些列的值要转换为日期时间。我们之前看到的一个选项是 `read_csv` 函数的 `parse_dates` 参数，它在第三章中引入。我们可以将参数传递给一个列表，其中包含 pandas 应将其值转换为日期时间的列：

```
In  [39] disney = pd.read_csv("disney.csv", parse_dates = ["Date"])
```

另一个解决方案是 pandas 顶层中的`to_datetime`转换函数。该函数接受一个可迭代对象（例如 Python 列表、元组、`Series`或索引），将其值转换为日期时间，并返回新的值在一个`DatetimeIndex`中。以下是一个小例子：

```
In  [40] string_dates = ["2015-01-01", "2016-02-02", "2017-03-03"]
         dt_index = pd.to_datetime(string_dates)
         dt_index

Out [40] DatetimeIndex(['2015-01-01', '2016-02-02', '2017-03-03'],
         dtype='datetime64[ns]', freq=None)
```

让我们把来自 disney `DataFrame`的日期`Series`传递给`to_datetime`函数：

```
In  [41] pd.to_datetime(disney["Date"]).head()

Out [41] 0   1962-01-02
         1   1962-01-03
         2   1962-01-04
         3   1962-01-05
         4   1962-01-08
         Name: Date, dtype: datetime64[ns]
```

我们有一个日期时间的`Series`，所以让我们覆盖原始`DataFrame`。接下来的代码示例将原始日期列替换为新的日期时间`Series`。记住，Python 首先评估等号右侧的表达式：

```
In  [42] disney["Date"] = pd.to_datetime(disney["Date"])
```

让我们再次通过`dtypes`属性检查日期列：

```
In  [43] disney.dtypes

Out [43] Date     datetime64[ns]
         High            float64
         Low             float64
         Open            float64
         Close           float64
         dtype: object
```

太好了；我们有一个日期时间列！我们的日期值存储正确后，我们可以探索 pandas 提供的强大的内置日期时间功能。

## 11.4 使用 DatetimeProperties 对象

一个日期时间`Series`包含一个特殊的`dt`属性，它暴露了一个`DatetimeProperties`对象：

```
In  [44] disney["Date"].dt

Out [44] <pandas.core.indexes.accessors.DatetimeProperties object at
         0x116247950>
```

我们可以在`DatetimeProperties`对象上访问属性并调用方法来从列的日期时间值中提取信息。`dt`属性对于日期时间就像`str`属性对于字符串一样。（参见第六章对`str`的回顾。）这两个属性都专门用于特定类型数据的操作。

让我们从`DatetimeProperties`对象的`day`属性开始探索，该属性从每个日期中提取出天。Pandas 返回值在一个新的`Series`中：

```
In  [45] disney["Date"].head(3)

Out [45] 0   1962-01-02
         1   1962-01-03
         2   1962-01-04
         Name: Date, dtype: datetime64[ns]

In  [46] disney["Date"].dt.day.head(3)

Out [46] 0    2
         1    3
         2    4
         Name: Date, dtype: int64
```

`month`属性返回一个包含月份数字的`Series`。1 月有`month`值为`1`，2 月有`month`值为`2`，依此类推。需要注意的是，这与我们在 Python/pandas 中通常的计数方式不同，在那里我们给第一个元素分配值为`0`：

```
In  [47] disney["Date"].dt.month.head(3)

Out [47] 0    1
         1    1
         2    1
         Name: Date, dtype: int64
```

`year`属性返回一个新的包含年份的`Series`：

```
In  [48] disney["Date"].dt.year.head(3)

Out [48] 0    1962
         1    1962
         2    1962
         Name: Date, dtype: int64
```

之前的属性相当简单。我们可以要求 pandas 提取更有趣的信息。一个例子是`dayofweek`属性，它返回每个日期星期数的`Series`。`0`表示星期一，`1`表示星期二，以此类推，直到`6`表示星期日。在以下输出中，索引位置 0 处的`1`值表示 1962 年 1 月 2 日是星期二：

```
In  [49] disney["Date"].dt.dayofweek.head()

Out [49] 0    1
         1    2
         2    3
         3    4
         4    0
         Name: Date, dtype: int64
```

如果我们想要的是星期的名称而不是数字，那么`day_name`方法就能派上用场。注意语法。我们是在`dt`对象上调用这个方法，而不是在`Series`本身上：

```
In  [50] disney["Date"].dt.day_name().head()

Out [50] 0      Tuesday
         1    Wednesday
         2     Thursday
         3       Friday
         4       Monday
         Name: Date, dtype: object
```

我们可以将这些`dt`属性和方法与其他 pandas 功能结合使用进行高级分析。以下是一个例子。让我们计算迪士尼股票按星期的平均表现。我们将首先将`dt.day_name`方法返回的`Series`附加到 disney `DataFrame`上：

```
In  [51] disney["Day of Week"] = disney["Date"].dt.day_name()
```

我们可以根据新星期几列的值对行进行分组（这是一种在第七章中介绍的技术）：

```
In  [52] group = disney.groupby("Day of Week")
```

我们可以调用`GroupBy`对象的`mean`方法来计算每个分组的值的平均值：

```
In  [53] group.mean()

Out [53]

                  High        Low       Open      Close
**Day of Week** 
Friday       23.767304  23.318898  23.552872  23.554498
Monday       23.377271  22.930606  23.161392  23.162543
Thursday     23.770234  23.288687  23.534561  23.540359
Tuesday      23.791234  23.335267  23.571755  23.562907
Wednesday    23.842743  23.355419  23.605618  23.609873
```

在三行代码中，我们计算了按周计算的平均股票表现。

让我们回到`dt`对象方法。补充的`month_name`方法返回包含日期月份名称的`Series`：

```
In  [54] disney["Date"].dt.month_name().head()

Out [54] 0    January
         1    January
         2    January
         3    January
         4    January
         Name: Date, dtype: object
```

`dt`对象上的一些属性返回布尔值。假设我们想探索迪士尼在其历史中每个季度的股票表现。商业年的四个季度分别从 1 月 1 日、4 月 1 日、7 月 1 日和 10 月 1 日开始。`is_quarter_start`属性返回一个布尔`Series`，其中`True`表示该行的日期落在季度开始日：

```
In  [55] disney["Date"].dt.is_quarter_start.tail()

Out [55] 14722    False
         14723    False
         14724    False
         14725     True
         14726    False
         Name: Date, dtype: bool
```

我们可以使用布尔`Series`来提取在季度开始时掉落的迪士尼行。下一个示例使用熟悉的方括号语法来提取行：

```
In  [56] disney[disney["Date"].dt.is_quarter_start].head()

Out [56]

 **Date      High       Low      Open     Close Day of Week**
189 1962-10-01  0.064849  0.062355  0.063913  0.062355      Monday
314 1963-04-01  0.087989  0.086704  0.087025  0.086704      Monday
377 1963-07-01  0.096338  0.095053  0.096338  0.095696      Monday
441 1963-10-01  0.110467  0.107898  0.107898  0.110467     Tuesday
565 1964-04-01  0.116248  0.112394  0.112394  0.116248   Wednesday
```

我们可以使用`is_quarter_end`属性来提取在季度结束时掉落的日期：

```
In  [57] disney[disney["Date"].dt.is_quarter_end].head()

Out [57]

 **Date      High       Low      Open     Close Day of Week**
251 1962-12-31  0.074501  0.071290  0.074501  0.072253      Monday
440 1963-09-30  0.109825  0.105972  0.108541  0.107577      Monday
502 1963-12-31  0.101476  0.096980  0.097622  0.101476     Tuesday
564 1964-03-31  0.115605  0.112394  0.114963  0.112394     Tuesday
628 1964-06-30  0.101476  0.100191  0.101476  0.100834     Tuesday
```

补充的`is_month_start`和`is_month_end`属性确认日期是在月份的开始或结束：

```
In  [58] disney[disney["Date"].dt.is_month_start].head()

Out [58]

 **Date      High       Low      Open     Close Day of Week**
22  1962-02-01  0.096338  0.093532  0.093532  0.094779    Thursday
41  1962-03-01  0.095714  0.093532  0.093532  0.095714    Thursday
83  1962-05-01  0.087296  0.085426  0.085738  0.086673     Tuesday
105 1962-06-01  0.079814  0.077943  0.079814  0.079814      Friday
147 1962-08-01  0.068590  0.068278  0.068590  0.068590   Wednesday
In  [59] disney[disney["Date"].dt.is_month_end].head()

Out [59]

 **Date      High       Low      Open     Close Day of Week**
21  1962-01-31  0.093844  0.092908  0.093532  0.093532   Wednesday
40  1962-02-28  0.094779  0.093220  0.094155  0.093220   Wednesday
82  1962-04-30  0.087608  0.085738  0.087608  0.085738      Monday
104 1962-05-31  0.082308  0.079814  0.079814  0.079814    Thursday
146 1962-07-31  0.069214  0.068278  0.068278  0.068590     Tuesday
```

`is_year_start`属性如果日期在年初，则返回`True`。下一个示例返回一个空的`DataFrame`；由于新年那天股市关闭，数据集中的日期都不符合标准：

```
In  [60] disney[disney["Date"].dt.is_year_start].head()

Out [60]

 **Date      High       Low      Open     Close Day of Week**
```

补充的`is_year_end`属性如果日期在年底，则返回`True`：

```
In  [61] disney[disney["Date"].dt.is_year_end].head()

Out [61]

 **Date      High       Low      Open     Close Day of Week**
251  1962-12-31  0.074501  0.071290  0.074501  0.072253      Monday
502  1963-12-31  0.101476  0.096980  0.097622  0.101476     Tuesday
755  1964-12-31  0.117853  0.116890  0.116890  0.116890    Thursday
1007 1965-12-31  0.154141  0.150929  0.153498  0.152214      Friday
1736 1968-12-31  0.439301  0.431594  0.434163  0.436732     Tuesday
```

无论属性如何，过滤过程都保持不变：创建一个布尔`Series`，然后将其传递到`DataFrame`后面的方括号内。

## 11.5 添加和减去时间持续时间

我们可以使用`DateOffset`对象添加或减去一致的时间持续时间。其构造函数在 pandas 的顶层可用。构造函数接受`years`、`months`、`days`等参数。下一个示例模拟了三年、四个月和三天的时间：

```
In  [62] pd.DateOffset(years = 3, months = 4, days = 5)

Out [62] <DateOffset: days=5, months=4, years=3>
```

这里是迪士尼`DataFrame`前五行的提醒：

```
In  [63] disney["Date"].head()

Out [63] 0   1962-01-02
         1   1962-01-03
         2   1962-01-04
         3   1962-01-05
         4   1962-01-08
         Name: Date, dtype: datetime64[ns]
```

为了举例，让我们假设我们的记录系统出现故障，日期列中的日期偏差了五天。我们可以使用加号（`+`）和`DateOffset`对象向日期时间`Series`中的每个日期添加一个一致的时间量。加号表示“向前移动”或“进入未来。”下一个示例将日期列中的每个日期增加五天：

```
In  [64] (disney["Date"] + pd.DateOffset(days = 5)).head()

Out [64] 0   1962-01-07
         1   1962-01-08
         2   1962-01-09
         3   1962-01-10
         4   1962-01-13
         Name: Date, dtype: datetime64[ns]
```

当与`DateOffset`一起使用时，减号（`-`）从日期`Series`中的每个日期减去一个持续时间。减号表示“向后移动”或“进入过去。”下一个示例将每个日期向后移动三天：

```
In  [65] (disney["Date"] - pd.DateOffset(days = 3)).head()

Out [65] 0   1961-12-30
         1   1961-12-31
         2   1962-01-01
         3   1962-01-02
         4   1962-01-05
         Name: Date, dtype: datetime64[ns]
```

尽管前面的输出没有显示，但`Timestamp`对象确实内部存储时间。当我们将日期列的值转换为日期时间时，pandas 假设每个日期的时间为午夜。下一个示例向`DateOffset`构造函数添加一个`hours`参数，以向日期中的每个日期时间添加一个一致的时间。结果`Series`显示日期和时间：

```
In  [66] (disney["Date"] + pd.DateOffset(days = 10, hours = 6)).head()

Out [66] 0   1962-01-12 06:00:00
         1   1962-01-13 06:00:00
         2   1962-01-14 06:00:00
         3   1962-01-15 06:00:00
         4   1962-01-18 06:00:00
         Name: Date, dtype: datetime64[ns]
```

Pandas 在减去持续时间时应用相同的逻辑。下一个示例从每个日期减去一年、三个月、十天、六小时和三分钟：

```
In  [67] (
             disney["Date"]
             - pd.DateOffset(
                 years = 1, months = 3, days = 10, hours = 6, minutes = 3
             )
         ).head()

Out [67] 0   1960-09-21 17:57:00
         1   1960-09-22 17:57:00
         2   1960-09-23 17:57:00
         3   1960-09-24 17:57:00
         4   1960-09-27 17:57:00
         Name: Date, dtype: datetime64[ns]
```

`DateOffset`构造函数支持额外的秒、微秒和纳秒关键字参数。有关更多信息，请参阅 pandas 文档。

## 11.6 日期偏移

`DateOffset`对象对于向每个日期添加或减去固定的时间量是最优的。现实世界的分析通常需要更动态的计算。假设我们想要将每个日期四舍五入到当前月的月底。每个日期距离其月底的天数不同，因此一致的`DateOffset`添加是不够的。

Pandas 提供了预构建的偏移对象，用于动态的时间计算。这些对象定义在库中的`offsets.py`模块中。在我们的代码中，我们必须使用它们的完整路径作为前缀：`pd.offsets`。

一个示例偏移量是`MonthEnd`，它将每个日期四舍五入到下个月的月底。这里是对日期列最后五行的复习：

```
In  [68] disney["Date"].tail()

Out [68] 14722   2020-06-26
         14723   2020-06-29
         14724   2020-06-30
         14725   2020-07-01
         14726   2020-07-02
         Name: Date, dtype: datetime64[ns]
```

我们可以将第 11.5 节中的加法和减法语法应用于 pandas 的偏移对象。下一个示例返回一个新的`Series`，将每个日期时间四舍五入到月底。加号表示时间向前移动，因此我们移动到下个月的月底：

```
In  [69] (disney["Date"] + pd.offsets.MonthEnd()).tail()

Out [69] 14722   2020-06-30
         14723   2020-06-30
         14724   2020-07-31
         14725   2020-07-31
         14726   2020-07-31
         Name: Date, dtype: datetime64[ns]
```

必须在预期的方向上有所移动。Pandas 不能将日期四舍五入到相同的日期。因此，如果一个日期位于月底，库会将其四舍五入到下个月的月底。Pandas 将索引位置 14724 处的 2020-06-30 四舍五入到 2020-07-31，即下一个可用的月底。

减号将每个日期向后移动。下一个示例使用`MonthEnd`偏移量将日期四舍五入到上个月的月底。Pandas 将前三个日期（2020-06-26、2020-06-29 和 2020-06-30）四舍五入到 5 月的最后一天，即 2020-05-31。它将最后两个日期（2020-07-01 和 2020-07-02）四舍五入到 6 月的最后一天，即 2020-06-30：

```
In  [70] (disney["Date"] - pd.offsets.MonthEnd()).tail()

Out [70] 14722   2020-05-31
         14723   2020-05-31
         14724   2020-05-31
         14725   2020-06-30
         14726   2020-06-30
         Name: Date, dtype: datetime64[ns]
```

相补的`MonthBegin`偏移量将四舍五入到一个月的第一天。下一个示例使用加号`+`将每个日期四舍五入到下个月的第一天。Pandas 将前三个日期（2020-06-26、2020-06-29 和 2020-06-30）四舍五入到 7 月的开始，即 2020-07-01。Pandas 将剩下的两个 7 月日期（2020-07-01 和 2020-07-02）四舍五入到 8 月的第一天，即 2020-08-01：

```
In  [71] (disney["Date"] + pd.offsets.MonthBegin()).tail()

Out [71] 14722   2020-07-01
         14723   2020-07-01
         14724   2020-07-01
         14725   2020-08-01
         14726   2020-08-01
         Name: Date, dtype: datetime64[ns]
```

我们可以将`MonthBegin`偏移量与减号结合使用，将日期回滚到月份的开始。在下一个示例中，pandas 将前三个日期（2020-06-26、2020-06-29 和 2020-06-30）回滚到 2020 年 6 月 1 日，即 6 月的开始。它将最后一个日期，2020-07-02，回滚到 7 月的开始，即 2020-07-01。一个有趣的情况是索引位置 14725 处的 2020-07-01。正如我们之前提到的，pandas 不能将日期回滚到相同的日期。因此，必须有一些回滚的动作，所以 pandas 将其回滚到上个月的开头，即 2020-06-01：

```
In  [72] (disney["Date"] - pd.offsets.MonthBegin()).tail()

Out [72] 14722   2020-06-01
         14723   2020-06-01
         14724   2020-06-01
         14725   2020-06-01
         14726   2020-07-01
         Name: Date, dtype: datetime64[ns]
```

对于商业时间计算，有一组特殊的偏移量可用；它们的名称以大写 `"B"` 开头。例如，商业月末 (`BMonthEnd`) 偏移量将四舍五入到该月的最后工作日。这五个工作日是星期一、星期二、星期三、星期四和星期五。

考虑以下三个日期时间的 `Series`。这三个日期分别落在星期四、星期五和星期六：

```
In  [73] may_dates = ["2020-05-28", "2020-05-29", "2020-05-30"]
         end_of_may = pd.Series(pd.to_datetime(may_dates))
         end_of_may

Out [73] 0   2020-05-28
         1   2020-05-29
         2   2020-05-30
         dtype: datetime64[ns]
```

让我们比较 `MonthEnd` 和 `BMonthEnd` 偏移量。当我们用加号搭配 `MonthEnd` 偏移量时，pandas 将所有三个日期四舍五入到 2020 年 5 月的最后一天，即 2020-05-31。无论这个日期是否是工作日或周末都无关紧要：

```
In  [74] end_of_may + pd.offsets.MonthEnd()

Out [74] 0   2020-05-31
         1   2020-05-31
         2   2020-05-31
         dtype: datetime64[ns]
```

`BMonthEnd` 偏移量返回不同的结果集。2020 年 5 月的最后工作日是星期五，5 月 29 日。Pandas 将 `Series` 中的第一个日期，2020-05-28，四舍五入到 29 日。下一个日期，2020-05-29，正好是该月的最后工作日。Pandas 不能将日期四舍五入到相同的日期，所以它将 2020-05-29 四舍五入到 6 月的最后工作日，即 2020-06-30，星期二。`Series` 中的最后一个日期，2020-05-30，是星期六。5 月没有剩余的工作日，所以 pandas 同样将日期四舍五入到 6 月的最后工作日，即 2020-06-30：

```
In  [75] end_of_may + pd.offsets.BMonthEnd()

Out [75] 0   2020-05-29
         1   2020-06-30
         2   2020-06-30
         dtype: datetime64[ns]
```

`pd.offsets` 模块包括额外的偏移量，用于四舍五入到季度、商业季度、年份、商业年份等的开始和结束。请在你的空闲时间自由探索它们。

## 11.7 `Timedelta` 对象

你可能还记得本章前面提到的 Python 的原生 `timedelta` 对象。`timedelta` 模型了持续时间——两个时间点之间的距离。像一小时这样的持续时间表示时间的长度；它没有特定的日期或时间。Pandas 使用自己的 `Timedelta` 对象来模型持续时间。

注意：这两个对象很容易混淆。`timedelta` 是 Python 内置的，而 `Timedelta` 是 pandas 内置的。当与 pandas 操作一起使用时，这两个对象是可以互换的。

`Timedelta` 构造函数在 pandas 顶层可用。它接受时间单位的键控参数，如 `days`、`hours`、`minutes` 和 `seconds`。下一个示例实例化了一个 `Timedelta`，它模型了八天、七小时、六分钟和五秒：

```
In  [76] duration = pd.Timedelta(
            days = 8,
            hours = 7,
            minutes = 6,
            seconds = 5
        )

         duration

Out [76] Timedelta('8 days 07:06:05')
```

pandas 顶层中的 `to_timedelta` 函数将它的参数转换为 `Timedelta` 对象。我们可以传递一个字符串，如下一个示例所示：

```
In  [77] duration = pd.to_timedelta("3 hours, 5 minutes, 12 seconds")

Out [77] Timedelta('0 days 03:05:12')
```

我们还可以将一个整数和一个 `unit` 参数一起传递给 `to_timedelta` 函数。`unit` 参数声明了数字所代表的时间单位。接受的参数包括 `"hour"`、`"day"` 和 `"minute"`。下一个示例中的 `Timedelta` 模型了一个五小时的持续时间：

```
In  [78] pd.to_timedelta(5, unit = "hour")

Out [78] Timedelta('0 days 05:00:00')
```

我们可以将一个可迭代对象，如列表，传递给 `to_timedelta` 函数，将其值转换为 `Timedeltas`。Pandas 将 `Timedeltas` 存储在 `TimedeltaIndex` 中，这是一个用于存储持续时间的 pandas 索引：

```
In  [79] pd.to_timedelta([5, 10, 15], unit = "day")

Out [79] TimedeltaIndex(['5 days', '10 days', '15 days'],
         dtype='timedelta64[ns]', freq=None)
```

通常，`Timedelta` 对象是从头创建的，而不是从头创建的。例如，从一个 `Timestamp` 减去另一个 `Timestamp` 会自动返回一个 `Timedelta`：

```
In  [80] pd.Timestamp("1999-02-05") - pd.Timestamp("1998-05-24")

Out [80] Timedelta('257 days 00:00:00')
```

现在我们已经熟悉了 `Timedelta`，让我们导入本章的第二个数据集：deliveries.csv。CSV 跟踪一家虚构公司的产品运输。每一行包括订单日期和交货日期：

```
In  [81] deliveries = pd.read_csv("deliveries.csv")
         deliveries.head()

Out [81]

 **order_date delivery_date**
0    5/24/98        2/5/99
1    4/22/92        3/6/98
2    2/10/91       8/26/92
3    7/21/92      11/20/97
4     9/2/93       6/10/98
```

让我们练习将两个列中的值转换为日期时间。是的，我们可以使用 `parse_dates` 参数，但让我们尝试另一种方法。一个选项是两次调用 `to_datetime` 函数，一次用于订单日期列，一次用于交货日期列，并覆盖现有的 `DataFrame` 列：

```
In  [82] deliveries["order_date"] = pd.to_datetime(
             deliveries["order_date"]
         )

         deliveries["delivery_date"] = pd.to_datetime(
             deliveries["delivery_date"]
         )
```

一个更可扩展的解决方案是使用 `for` 循环遍历列名。我们可以动态地引用一个交货列，使用 `to_datetime` 从它创建一个 `Timestamps` 的 `DatetimeIndex`，然后覆盖原始列：

```
In  [83] for column in ["order_date", "delivery_date"]:
             deliveries[column] = pd.to_datetime(deliveries[column])
```

让我们看看交货情况。新的列格式确认我们已经将字符串转换为日期时间：

```
In  [84] deliveries.head()

Out [84]

 **order_date delivery_date**
0 1998-05-24    1999-02-05
1 1992-04-22    1998-03-06
2 1991-02-10    1992-08-26
3 1992-07-21    1997-11-20
4 1993-09-02    1998-06-10
```

让我们来计算每批货物的持续时间。使用 pandas，这个计算就像从订单日期列减去交货日期列一样简单：

```
In  [85] (deliveries["delivery_date"] - deliveries["order_date"]).head()

Out [85] 0    257 days
         1   2144 days
         2    563 days
         3   1948 days
         4   1742 days
         dtype: timedelta64[ns]
```

Pandas 返回一个 `timedelta` 的 `Series`。让我们将这个新的 `Series`附加到交货 `DataFrame` 的末尾：

```
In  [86] deliveries["duration"] = (
             deliveries["delivery_date"] - deliveries["order_date"]
         )
         deliveries.head()

Out [86]

 **order_date delivery_date  duration**
0 1998-05-24    1999-02-05  257 days
1 1992-04-22    1998-03-06 2144 days
2 1991-02-10    1992-08-26  563 days
3 1992-07-21    1997-11-20 1948 days
4 1993-09-02    1998-06-10 1742 days
```

现在我们有两个 `Timestamp` 列和一个 `Timedelta` 列：

```
In  [87] deliveries.dtypes

Out [87] order_date        datetime64[ns]
         delivery_date     datetime64[ns]
         duration         timedelta64[ns]
         dtype: object
```

我们可以从 `Timestamp` 对象中添加或减去 `Timedelta`。下一个示例从每行的持续时间中减去交货日期列。可预测的是，新 `Series` 中的值与订单日期列中的值相同：

```
In  [88] (deliveries["delivery_date"] - deliveries["duration"]).head()

Out [88] 0   1998-05-24
         1   1992-04-22
         2   1991-02-10
         3   1992-07-21
         4   1993-09-02
         dtype: datetime64[ns]
```

加号符号将 `Timedelta` 添加到 `Timestamp`。假设我们想找到每个包裹需要两倍时间到达的交货日期。我们可以将持续时间列中的 `Timedelta` 值添加到交货日期列中的 `Timestamp` 值：

```
In  [89] (deliveries["delivery_date"] + deliveries["duration"]).head()

Out [89] 0   1999-10-20
         1   2004-01-18
         2   1994-03-12
         3   2003-03-22
         4   2003-03-18
         dtype: datetime64[ns]
```

`sort_values` 方法与 `Timedelta` `Series` 一起工作。下一个示例按升序对持续时间列进行排序，从最短的交货到最长的交货：

```
In  [90] deliveries.sort_values("duration")

Out [90]

 **order_date delivery_date  duration**
454 1990-05-24    1990-06-01    8 days
294 1994-08-11    1994-08-20    9 days
10  1998-05-10    1998-05-19    9 days
499 1993-06-03    1993-06-13   10 days
143 1997-09-20    1997-10-06   16 days
...        ...           ...       ...
152 1990-09-18    1999-12-19 3379 days
62  1990-04-02    1999-08-16 3423 days
458 1990-02-13    1999-11-15 3562 days
145 1990-03-07    1999-12-25 3580 days
448 1990-01-20    1999-11-12 3583 days

501 rows × 3 columns
```

数学方法也适用于 `Timedelta` `Series`。接下来的几个示例突出了我们在本书中使用的三种方法：`max` 用于最大值，`min` 用于最小值，`mean` 用于平均值：

```
In  [91] deliveries["duration"].max()

Out [91] Timedelta('3583 days 00:00:00')

In  [92] deliveries["duration"].min()

Out [92] Timedelta('8 days 00:00:00')

In  [93] deliveries["duration"].mean()

Out [93] Timedelta('1217 days 22:53:53.532934')
```

这是下一个挑战。让我们过滤 `DataFrame`，以找到交货时间超过一年的包裹。我们可以使用大于符号 (`>`) 来比较每个持续时间列的值与固定持续时间。我们可以将时间长度指定为 `Timedelta` 或字符串。下一个示例使用 `"365 days"`：

```
In  [94] # The two lines below are equivalent
         (deliveries["duration"] > pd.Timedelta(days = 365)).head()
         (deliveries["duration"] > "365 days").head()

Out [94] 0      False
         1       True
         2       True
         3       True
         4       True
         Name: Delivery Time, dtype: bool
```

让我们使用布尔 `Series` 过滤出交货时间超过 365 天的 `deliveries` 行：

```
In  [95] deliveries[deliveries["duration"] > "365 days"].head()

Out [95]

 **order_date delivery_date  duration**
1 1992-04-22    1998-03-06 2144 days
2 1991-02-10    1992-08-26  563 days
3 1992-07-21    1997-11-20 1948 days
4 1993-09-02    1998-06-10 1742 days
6 1990-01-25    1994-10-02 1711 days
```

我们可以根据需要将比较持续时间细化。下一个示例包括字符串中的天数、小时和分钟，用逗号分隔时间单位：

```
In  [96] long_time = (
             deliveries["duration"] > "2000 days, 8 hours, 4 minutes"
         )

         deliveries[long_time].head()

Out [96]

 ** order_date delivery_date  duration**
1  1992-04-22    1998-03-06 2144 days
7  1992-02-23    1998-12-30 2502 days
11 1992-10-17    1998-10-06 2180 days
12 1992-05-30    1999-08-15 2633 days
15 1990-01-20    1998-07-24 3107 days
```

作为提醒，Pandas 可以对 `Timedelta` 列进行排序。要发现最长或最短时长，我们可以在时长 `Series` 上调用 `sort_values` 方法。

## 11.8 编程挑战

这是练习本章引入的概念的机会。

### 11.8.1 问题

Citi Bike NYC 是纽约市的官方自行车共享计划。居民和游客可以在城市数百个地点取车和还车。骑行数据是公开的，由市政府每月发布，网址为 [`www.citibikenyc.com/system-data`](https://www.citibikenyc.com/system-data)。citibike.csv 是 2020 年 6 月骑行的约 190 万次骑行数据的集合。为了简化，数据集已从原始版本修改，仅包括两个列：每次骑行的开始时间和结束时间。让我们导入数据集并将其分配给 `citi_bike` 变量：

```
In  [97] citi_bike = pd.read_csv("citibike.csv")
         citi_bike.head()

Out [97]
 ** start_time                 stop_time**
0  2020-06-01 00:00:03.3720  2020-06-01 00:17:46.2080
1  2020-06-01 00:00:03.5530  2020-06-01 01:03:33.9360
2  2020-06-01 00:00:09.6140  2020-06-01 00:17:06.8330
3  2020-06-01 00:00:12.1780  2020-06-01 00:03:58.8640
4  2020-06-01 00:00:21.2550  2020-06-01 00:24:18.9650
```

start_time 和 stop_time 列中的 datetime 条目包括年、月、日、时、分、秒和微秒。（*微秒*是等于一百万分之一秒的时间单位。）

我们可以使用 `info` 方法打印一个摘要，包括 `DataFrame` 的长度、列的数据类型和内存使用情况。注意，pandas 已经将两个列的值作为字符串导入：

```
In  [98] citi_bike.info()

Out [98]

<class 'pandas.core.frame.DataFrame'>
RangeIndex: 1882273 entries, 0 to 1882272
Data columns (total 2 columns):
 #   Column      Dtype
---  ------      -----
 0   start_time  object
 1   stop_time   object
dtypes: object(2)
memory usage: 28.7+ MB
```

这是练习本章引入的概念的机会。

1.  将 start_time 和 stop_time 列转换为存储 datetime (`Timestamp`) 值而不是字符串。

1.  计算每周每天（周一、周二等）发生的骑行次数。哪一天是骑行最受欢迎的工作日？以 start_time 列作为起点。

1.  计算每月每周内每周的骑行次数。为此，将 start_time 列中的每个日期四舍五入到其前一个或当前的周一。假设每周从周一开始，周日结束。因此，六月的第一个星期是 6 月 1 日星期一到 6 月 7 日星期日。

1.  计算每次骑行的时长，并将结果保存到一个新的时长列中。

1.  查找一次骑行的平均时长。

1.  从数据集中提取按时长排序的前五次最长骑行。

### 11.8.2 解决方案

让我们逐个解决这些问题：

1.  pandas 顶层中的 `to_datetime` 转换函数可以很好地将 start_time 和 end_time 列的值转换为 `Timestamps`。下面的代码示例使用 `for` 循环遍历列名列表，将每个列传递给 `to_datetime` 函数，并用新的 datetime `Series` 覆盖现有的字符串列：

    ```
    In  [99] for column in ["start_time", "stop_time"]:
                 citi_bike[column] = pd.to_datetime(citi_bike[column])
    ```

    让我们再次调用 `info` 方法来确认这两个列存储的是 datetime 值：

    ```
    In  [100] citi_bike.info()

    Out [100]

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1882273 entries, 0 to 1882272
    Data columns (total 2 columns):
     #   Column      Dtype
    ---  ------      -----
     0   start_time  datetime64[ns]
     1   stop_time   datetime64[ns]
    dtypes: datetime64ns
    memory usage: 28.7 MB
    ```

1.  我们需要分两步来计算每周的骑行次数。首先，我们从 start_time 列中的每个 datetime 提取星期几；然后计算星期几的出现次数。`dt.day_name` 方法返回一个包含每个日期星期几名称的 `Series`：

    ```
    In  [101] citi_bike["start_time"].dt.day_name().head()

    Out [101] 0    Monday
              1    Monday
              2    Monday
              3    Monday
              4    Monday
              Name: start_time, dtype: object
    ```

    然后，我们可以在返回的 `Series` 上调用可靠的 `value_counts` 方法来计算工作日。在 2020 年 6 月，星期二是最受欢迎的骑行日：

    ```
    In  [102] citi_bike["start_time"].dt.day_name().value_counts()

    Out [102] Tuesday      305833
              Sunday       301482
              Monday       292690
              Saturday     285966
              Friday       258479
              Wednesday    222647
              Thursday     215176
              Name: start_time, dtype: int64
    ```

1.  下一个挑战要求我们将每个日期分组到其对应的周桶中。我们可以通过将日期四舍五入到其前一个或当前星期一来实现。这里有一个巧妙的解决方案：我们可以使用 `dayofweek` 属性来返回一个数字 `Series`。`0` 表示星期一，`1` 表示星期二，`6` 表示星期日，依此类推：

    ```
    In  [103] citi_bike["start_time"].dt.dayofweek.head()

    Out [103] 0    0
              1    0
              2    0
              3    0
              4    0
                 Name: start_time, dtype: int64
    ```

    星期几的数字也代表从最近的星期一到当前日期的天数距离。例如，6 月 1 日的星期一有 `dayofweek` 值为 `0`。该日期距离最近的星期一有 0 天。同样，6 月 2 日的星期二有 `dayofweek` 值为 1。该日期距离最近的星期一（6 月 1 日）有 1 天。让我们将这个 `Series` 保存到 `days_away_from_monday` 变量中：

    ```
    In  [104] days_away_from_monday = citi_bike["start_time"].dt.dayofweek
    ```

    如果我们从日期中减去 `dayofweek` 值，我们将有效地将每个日期四舍五入到其前一个星期一。我们可以将 `dayofweek` `Series` 传递给 `to_timedelta` 函数以将其转换为时长 `Series`。我们将传递一个设置为 `"day"` 的单位参数，告诉 pandas 将数值视为天数：

    ```
    In  [105] citi_bike["start_time"] - pd.to_timedelta(
                  days_away_from_monday, unit = "day"
              )

    Out [105] 0         2020-06-01 00:00:03.372
              1         2020-06-01 00:00:03.553
              2         2020-06-01 00:00:09.614
              3         2020-06-01 00:00:12.178
              4         2020-06-01 00:00:21.255
                                  ...
              1882268   2020-06-29 23:59:41.116
              1882269   2020-06-29 23:59:46.426
              1882270   2020-06-29 23:59:47.477
              1882271   2020-06-29 23:59:53.395
              1882272   2020-06-29 23:59:53.901
              Name: start_time, Length: 1882273, dtype: datetime64[ns]
    ```

    让我们将新的 `Series` 保存到 `dates_rounded_to_monday` 变量中：

    ```
    In  [106] dates_rounded_to_monday = citi_bike[
                  "start_time"
              ] - pd.to_timedelta(days_away_from_monday, unit = "day")
    ```

    我们已经完成了一半。我们已经将日期四舍五入到正确的星期一，但 `value_counts` 方法还不能使用。日期之间的时间差异会导致 pandas 认为它们不相等：

    ```
    In  [107] dates_rounded_to_monday.value_counts().head()

    Out [107] 2020-06-22 20:13:36.208    3
              2020-06-08 17:17:26.335    3
              2020-06-08 16:50:44.596    3
              2020-06-15 19:24:26.737    3
              2020-06-08 19:49:21.686    3
              Name: start_time, dtype: int64
    ```

    让我们使用 `dt.date` 属性来返回一个包含每个 datetime 的 `Series`：

    ```
    In  [108] dates_rounded_to_monday.dt.date.head()

    Out [108] 0    2020-06-01
              1    2020-06-01
              2    2020-06-01
              3    2020-06-01
              4    2020-06-01
              Name: start_time, dtype: object
    ```

    现在我们已经隔离了日期，我们可以调用 `value_counts` 方法来计算每个值的出现次数。从 6 月 15 日星期一到 6 月 21 日星期日的这一周，是整个月自行车骑行次数最多的一周：

    ```
    In  [109] dates_rounded_to_monday.dt.date.value_counts()

    Out [109] 2020-06-15    481211
              2020-06-08    471384
              2020-06-22    465412
              2020-06-01    337590
              2020-06-29    126676
              Name: start_time, dtype: int64
    ```

1.  要计算每段骑行的时长，我们可以从停止时间列减去开始时间列。Pandas 将返回一个 `Timedelta`s 的 `Series`。我们需要将这个 `Series` 保存到下一个示例中，所以让我们将其附加到 `DataFrame` 作为一个名为 duration 的新列：

    ```
    In  [110] citi_bike["duration"] = (
                  citi_bike["stop_time"] - citi_bike["start_time"]
              )

              citi_bike.head()

    Out [110]

     ** start_time               stop_time               duration**
    0 2020-06-01 00:00:03.372 2020-06-01 00:17:46.208 0 days 00:17:42.836000
    1 2020-06-01 00:00:03.553 2020-06-01 01:03:33.936 0 days 01:03:30.383000
    2 2020-06-01 00:00:09.614 2020-06-01 00:17:06.833 0 days 00:16:57.219000
    3 2020-06-01 00:00:12.178 2020-06-01 00:03:58.864 0 days 00:03:46.686000
    4 2020-06-01 00:00:21.255 2020-06-01 00:24:18.965 0 days 00:23:57.710000
    ```

    注意，如果列存储的是字符串，之前的减法操作将引发错误；这就是为什么在转换它们为日期时间之前，这是强制性的。

1.  接下来，我们必须找出所有自行车骑行平均时长。这个过程很简单：我们可以在新的时长列上调用 `mean` 方法进行计算。平均骑行时长为 27 分钟和 19 秒：

    ```
    In  [111] citi_bike["duration"].mean()

    Out [111] Timedelta('0 days 00:27:19.590506853')
    ```

1.  最后一个问题要求识别数据集中最长的五段自行车骑行。一个解决方案是使用 `sort_values` 方法按降序排序时长列的值，然后使用 `head` 方法查看前五行。这些会话可能属于那些在完成骑行后忘记检查自行车的人：

    ```
    In  [112] citi_bike["duration"].sort_values(ascending = False).head()

    Out [112] 50593    32 days 15:01:54.940000
              98339    31 days 01:47:20.632000
              52306    30 days 19:32:20.696000
              15171    30 days 04:26:48.424000
              149761   28 days 09:24:50.696000
              Name: duration, dtype: timedelta64[ns]
    ```

    另一个选项是 `nlargest` 方法。我们可以在时长 `Series` 或整个 `DataFrame` 上调用此方法。让我们选择后者：

    ```
    In  [113] citi_bike.nlargest(n = 5, columns = "duration")

    Out [113]

     ** start_time              stop_time               duration**
    50593  2020-06-01 21:30:17... 2020-07-04 12:32:12... 32 days 15:01:54.94...
    98339  2020-06-02 19:41:39... 2020-07-03 21:29:00... 31 days 01:47:20.63...
    52306  2020-06-01 22:17:10... 2020-07-02 17:49:31... 30 days 19:32:20.69...
    15171  2020-06-01 13:01:41... 2020-07-01 17:28:30... 30 days 04:26:48.42...
    149761 2020-06-04 14:36:53... 2020-07-03 00:01:44... 28 days 09:24:50.69...
    ```

这就是数据集中最长的五次骑行记录。恭喜您完成编码挑战！

## 摘要

+   Pandas 的 `Timestamp` 对象是一个灵活、强大的替代品，用于 Python 的原生 `datetime` 对象。

+   在 datetime `Series` 上的 `dt` 访问器揭示了一个具有属性和方法以提取日期、月份、星期名称等属性的 `DatetimeProperties` 对象。

+   `Timedelta` 对象表示一个持续时间。

+   当我们从两个 `Timestamp` 对象中减去时，Pandas 创建一个 `Timedelta` 对象。

+   `pd.offsets` 包中的偏移量动态地将日期四舍五入到最近的周、月、季度等。我们可以用加号向前四舍五入，用减号向后四舍五入。

+   `DatetimeIndex` 是 `Timestamp` 值的容器。我们可以将其作为索引或列添加到 Pandas 数据结构中。

+   `TimedeltaIndex` 是 `Timedelta` 对象的容器。

+   最高级的 `to_datetime` 函数将值的可迭代序列转换为 `Timestamp` 的 `DatetimeIndex`。
