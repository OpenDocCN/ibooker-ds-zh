# 第四章： NumPy 基础

正如你可能从第一章中记得的那样，NumPy 是 Python 中科学计算的核心包，提供对基于数组的计算和线性代数的支持。由于 NumPy 是 pandas 的基础，我将在本章介绍其基础知识：解释了 NumPy 数组是什么之后，我们将研究向量化和广播，这两个重要概念允许你编写简洁的数学代码，并在 pandas 中再次遇到。之后，我们将看到为什么 NumPy 提供了称为通用函数的特殊函数，然后通过学习如何获取和设置数组的值以及解释 NumPy 数组的视图和副本之间的区别来结束本章。即使在本书中我们几乎不会直接使用 NumPy，了解其基础知识将使我们更容易学习下一章的 pandas。

使用 NumPy 入门

在这一节中，我们将学习一维和二维 NumPy 数组及背后的技术术语向量化、广播和通用函数。

NumPy 数组

要执行基于数组的计算，如上一章节中遇到的嵌套列表，你需要编写某种形式的循环。例如，要将一个数字添加到嵌套列表中的每个元素，可以使用以下嵌套列表推导式：

> `In``[``1``]:``matrix``=``[[``1``,``2``,``3``],``[``4``,``5``,``6``],``[``7``,``8``,``9``]]`
> 
> `In``[``2``]:``[[``i``+``1``for``i``in``row``]``for``row``in``matrix``]`
> 
> `Out[2]: [[2, 3, 4], [5, 6, 7], [8, 9, 10]]`

这种方法不太易读，更重要的是，如果你使用大数组，逐个元素进行循环会变得非常缓慢。根据你的用例和数组的大小，使用 NumPy 数组而不是 Python 列表可以使计算速度提高几倍甚至提高到一百倍。NumPy 通过利用用 C 或 Fortran 编写的代码来实现这种性能提升——这些是编译性语言，比 Python 快得多。NumPy 数组是一种用于同类型数据的 N 维数组。同类型意味着数组中的所有元素都需要是相同的数据类型。最常见的情况是处理浮点数的一维和二维数组，如图 4-1 所示。

![](img/00042.jpg)

图 4-1 一维和二维 NumPy 数组

让我们创建一个一维和二维数组，在本章节中使用：

> `In``[``3``]:``# 首先，让我们导入 NumPy``import``numpy``as``np`
> 
> `In``[``4``]:``# 使用一个简单列表构建一个一维数组``array1``=``np``.``array``([``10``,``100``,``1000.``])`
> 
> `In``[``5``]:``# 使用嵌套列表构建一个二维数组``array2``=``np``.``array``([[``1.``,``2.``,``3.``],``[``4.``,``5.``,``6.``]])`
> 
> 数组维度
> 
> 重要的是要注意一维和二维数组之间的区别：一维数组只有一个轴，因此没有明确的列或行方向。虽然这类似于 VBA 中的数组，但如果你来自像 MATLAB 这样的语言，你可能需要习惯它，因为一维数组在那里始终具有列或行方向。

即使`array1`由整数组成，除了最后一个元素（它是一个浮点数），NumPy 数组的同质性强制数组的数据类型为`float64`，它能够容纳所有元素。要了解数组的数据类型，请访问其`dtype`属性：

> `In``[``6``]:``array1``.``dtype`
> 
> `Out[6]: dtype('float64')`

由于`dtype`返回的是`float64`而不是我们在上一章中遇到的`float`，你可能已经猜到 NumPy 使用了比 Python 数据类型更精细的自己的数值数据类型。不过，通常这不是问题，因为大多数情况下，Python 和 NumPy 之间的不同数据类型转换是自动的。如果你需要将 NumPy 数据类型显式转换为 Python 基本数据类型，只需使用相应的构造函数（我稍后会详细说明如何从数组中访问元素）：

> `In``[``7``]:``float``(``array1``[``0``])`
> 
> `Out[7]: 10.0`

要查看 NumPy 数据类型的完整列表，请参阅[NumPy 文档](https://oreil.ly/irDyH)。使用 NumPy 数组，你可以编写简单的代码来执行基于数组的计算，我们接下来会看到。

矢量化和广播

如果你构建一个标量和一个 NumPy 数组的和，NumPy 将执行逐元素操作，这意味着你不必自己循环遍历元素。NumPy 社区将这称为矢量化。它允许你编写简洁的代码，几乎与数学符号一样：

> `In``[``8``]:``array2``+``1`
> 
> `Out[8]: array([[2., 3., 4.],                [5., 6., 7.]])`
> 
> SCALAR
> 
> 标量指的是像浮点数或字符串这样的基本 Python 数据类型。这是为了将它们与具有多个元素的数据结构如列表和字典或一维和二维 NumPy 数组区分开来。

当你处理两个数组时，相同的原则也适用：NumPy 逐元素执行操作：

> `In``[``9``]:``array2``*``array2`
> 
> `Out[9]: array([[ 1.,  4.,  9.],                [16., 25., 36.]])`

如果你在算术运算中使用两个形状不同的数组，NumPy 会自动将较小的数组扩展到较大的数组，以使它们的形状兼容。这称为广播：

> `In``[``10``]:``array2``*``array1`
> 
> `Out[10]: array([[  10.,  200., 3000.],                 [  40.,  500., 6000.]])`

要执行矩阵乘法或点积，请使用`@`运算符：1

> `In``[``11``]:``array2``@``array2``.``T``# array2.T 是 array2.transpose()的快捷方式`
> 
> `Out[11]: array([[14., 32.],                 [32., 77.]])`

不要被本节介绍的术语（如标量、向量化或广播）吓到！如果你曾在 Excel 中使用过数组，这些内容应该会感到非常自然，如图 4-2 所示。该截图来自于 array_calculations.xlsx，在配套库的 xl 目录下可以找到。

![](img/00052.jpg)

图 4-2\. Excel 中基于数组的计算

你现在知道数组按元素进行算术运算，但如何在数组的每个元素上应用函数呢？这就是通用函数的用处所在。

通用函数（ufunc）

通用函数（ufunc）作用于 NumPy 数组中的每个元素。例如，若在 NumPy 数组上使用 Python 标准的平方根函数`math`模块，则会产生错误：

> `In``[``12``]:``import``math`
> 
> `In``[``13``]:``math``.``sqrt``(``array2``)``# This will raise en Error`
> 
> `--------------------------------------------------------------------------- TypeError                                 Traceback (most recent call last) <ipython-input-13-5c37e8f41094> in <module> ----> 1 math.sqrt(array2)  # This will raise en Error  TypeError: only size-1 arrays can be converted to Python scalars`

当然，你也可以编写嵌套循环来获取每个元素的平方根，然后从结果中再次构建一个 NumPy 数组：

> `In``[``14``]:``np``.``array``([[``math``.``sqrt``(``i``)``for``i``in``row``]``for``row``in``array2``])`
> 
> `Out[14]: array([[1.        , 1.41421356, 1.73205081],                 [2.        , 2.23606798, 2.44948974]])`

在 NumPy 没有提供 ufunc 且数组足够小的情况下，这种方式可以奏效。然而，若 NumPy 有对应的 ufunc，请使用它，因为它在处理大数组时速度更快——而且更易于输入和阅读：

> `In``[``15``]:``np``.``sqrt``(``array2``)`
> 
> `Out[15]: array([[1.        , 1.41421356, 1.73205081],                 [2.        , 2.23606798, 2.44948974]])`

NumPy 的一些 ufunc，例如`sum`，也可以作为数组方法使用：如果要对每列求和，则执行以下操作：

> `In``[``16``]:``array2``.``sum``(``axis``=``0``)``# 返回一个 1 维数组`
> 
> `Out[16]: array([5., 7., 9.])`

参数`axis=0`表示沿着行的轴，而`axis=1`表示沿着列的轴，如图 4-1 所示。若省略`axis`参数，则对整个数组求和：

> `In``[``17``]:``array2``.``sum``()``
> 
> `Out[17]: 21.0`

本书后续部分还会介绍更多 NumPy 的 ufunc，因为它们可与 pandas 的 DataFrame 一起使用。

到目前为止，我们一直在处理整个数组。下一节将向你展示如何操作数组的部分，并介绍一些有用的数组构造函数。

创建和操作数组

在介绍一些有用的数组构造函数之前，我将从获取和设置数组元素的角度开始这一节，包括一种用于生成伪随机数的构造函数，您可以将其用于蒙特卡洛模拟。最后，我将解释数组的视图和副本之间的区别。

获取和设置数组元素

在上一章中，我向您展示了如何索引和切片列表以访问特定元素。当您处理像本章第一个示例中的 `matrix` 这样的嵌套列表时，您可以使用链式索引：`matrix[0][0]` 将获得第一行的第一个元素。然而，使用 NumPy 数组时，您可以在单个方括号对中为两个维度提供索引和切片参数：

> `numpy_array``[``row_selection``,``column_selection``]`

对于一维数组，这简化为 `numpy_array[selection]`。当您选择单个元素时，将返回一个标量；否则，将返回一个一维或二维数组。请记住，切片表示法使用起始索引（包括）和结束索引（不包括），中间使用冒号，如 `start:end`。通过省略开始和结束索引，留下一个冒号，因此该冒号表示二维数组中的所有行或所有列。我在 Figure 4-3 中可视化了一些例子，但您可能也想再看看 Figure 4-1，因为那里标记了索引和轴。请记住，通过对二维数组的列或行进行切片，您最终得到的是一个一维数组，而不是一个二维列或行向量！

![](img/00061.jpg)

图 4-3\. 选择 NumPy 数组的元素

运行以下代码来尝试 Figure 4-3 中展示的示例：

> `In``[``18``]:``array1``[``2``]``# 返回一个标量`
> 
> `Out[18]: 1000.0`
> 
> `In``[``19``]:``array2``[``0``,``0``]``# 返回一个标量`
> 
> `Out[19]: 1.0`
> 
> `In``[``20``]:``array2``[:,``1``:]``# 返回一个 2 维数组`
> 
> `Out[20]: array([[2., 3.],                 [5., 6.]])`
> 
> `In``[``21``]:``array2``[:,``1``]``# 返回一个 1 维数组`
> 
> `Out[21]: array([2., 5.])`
> 
> `In``[``22``]:``array2``[``1``,``:``2``]``# 返回一个 1 维数组`
> 
> `Out[22]: array([4., 5.])`

到目前为止，我通过手动构建示例数组，即通过在列表中提供数字来完成。但是 NumPy 还提供了几个有用的函数来构建数组。

有用的数组构造函数

NumPy 提供了几种构建数组的方法，这些方法对于创建 pandas DataFrames 也很有帮助，正如我们将在 第五章 中看到的那样。一个简单创建数组的方式是使用 `arange` 函数。这代表数组范围，类似于我们在前一章中遇到的内置 `range` 函数，不同之处在于 `arange` 返回一个 NumPy 数组。结合 `reshape` 可以快速生成具有所需维度的数组：

> `In``[``23``]:``np``.``arange``(``2``*``5``)``.``reshape``(``2``,``5``)``# 2 行，5 列`
> 
> `Out[23]: array([[0, 1, 2, 3, 4],                 [5, 6, 7, 8, 9]])`

另一个常见的需求，例如蒙特卡洛模拟，是生成正态分布的伪随机数数组。NumPy 可以轻松实现这一点：

> `In``[``24``]:``np``.``random``.``randn``(``2``,``3``)``# 2 行，3 列`
> 
> `Out[24]: array([[-0.30047275, -1.19614685, -0.13652283],                 [ 1.05769357,  0.03347978, -1.2153504 ]])`

其他有用的构造函数包括 `np.ones` 和 `np.zeros`，分别用于创建全为 1 和全为 0 的数组，以及 `np.eye` 用于创建单位矩阵。我们将在下一章再次遇到其中一些构造函数，但现在让我们学习一下 NumPy 数组视图和副本之间的区别。

视图 vs. 复制

当你切片 NumPy 数组时，它们返回视图。这意味着你在处理原始数组的子集而不复制数据。在视图上设置值也将更改原始数组：

> `In``[``25``]:``array2`
> 
> `Out[25]: array([[1., 2., 3.],                 [4., 5., 6.]])`
> 
> `In``[``26``]:``subset``=``array2``[:,``:``2``]``subset`
> 
> `Out[26]: array([[1., 2.],                 [4., 5.]])`
> 
> `In``[``27``]:``subset``[``0``,``0``]``=``1000`
> 
> `In``[``28``]:``subset`
> 
> `Out[28]: array([[1000.,    2.],                 [   4.,    5.]])`
> 
> `In``[``29``]:``array2`
> 
> `Out[29]: array([[1000.,    2.,    3.],                 [   4.,    5.,    6.]])`

如果这不是你想要的，你需要按照以下方式更改 `In [26]`：

> `subset``=``array2``[:,``:``2``]``.``copy``()`

在副本上操作将不会改变原始数组。

结论

在本章中，我向你展示了如何使用 NumPy 数组以及背后的向量化和广播表达式。抛开这些技术术语，使用数组应该感觉非常直观，因为它们紧密遵循数学符号。虽然 NumPy 是一个非常强大的库，但在用于数据分析时有两个主要问题：

+   > > > > 整个 NumPy 数组都需要是相同的数据类型。例如，这意味着当数组包含文本和数字混合时，你不能执行本章中的任何算术运算。一旦涉及文本，数组将具有数据类型`object`，这将不允许数学运算。
+   > > > > 
+   > > > > 使用 NumPy 数组进行数据分析使得难以知道每列或每行指的是什么，因为通常通过它们的位置来选择列，比如在 `array2[:, 1]` 中。

pandas 通过在 NumPy 数组之上提供更智能的数据结构来解决了这些问题。它们是什么以及如何工作将是下一章的主题。

> 1   如果你已经很久没有上线性代数课了，你可以跳过这个例子——矩阵乘法不是本书的基础。
