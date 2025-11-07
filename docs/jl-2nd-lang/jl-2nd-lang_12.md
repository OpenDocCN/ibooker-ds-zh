# 10 表示未知值

本章涵盖

+   理解如何使用未定义的值

+   使用 Nothing 类型表示值的缺失

+   使用 Missing 类型处理存在但未知的值

在任何编程语言中处理的一个重要问题是表示值的缺失。长期以来，大多数主流编程语言，如 C/C++、Java、C#、Python 和 Ruby，都有一个名为 null 或 nil 的值，这是变量在没有值时可能包含的内容。更准确地说：null 或 nil 表示变量没有绑定到具体对象。

这在什么情况下会有用？让我们以 Julia 的 findfirst 函数为例。它定位子串的第一个出现：

```
julia> findfirst("hello", "hello world")    ❶
1:5

julia> findfirst("foo", "hello world")      ❷
```

❶ 找到子串 hello。

❷ 未找到子串 foo。

但如何表示找不到子串呢？像 Java、C#和 Python 这样的语言会使用 null 或 nil 关键字来表示这一点。然而，它的发明者，英国计算机科学家 Tony Hoare，称 null 指针是他的十亿美元错误，并非没有原因。

这使得编写安全代码变得困难，因为任何变量在任何给定时间都可能为 null。在支持 null 的语言编写的程序中，你需要大量的样板代码来执行 null 检查。这是因为对 null 对象执行操作是不安全的。

因此，现代语言倾向于避免有 null 对象或指针。Julia 没有通用的 null 对象或指针。相反，它有各种类型来表示未知或缺失的值。本章将向您介绍这些不同类型，如何使用它们以及何时使用它们。

## 10.1 nothing 对象

Julia 中最接近 null 的对象是 Nothing 类型的 nothing 对象。它是在 Julia 中定义的一个简单具体类型，如下所示。

列表 10.1 Julia 定义的 Nothing 类型和 nothing 常量

```
struct Nothing
    # look, no fields
end

const nothing = Nothing()
```

nothing 对象是类型 Nothing 的一个实例。然而，Nothing 的每个实例都是同一个对象。您可以在 REPL 中自行测试：

```
julia> none = Nothing()

julia> none == nothing
true

julia> Nothing() == Nothing()
true
```

然而，这里并没有什么神奇的事情发生。当你用零字段调用复合类型的构造函数时，你总是得到相同的对象返回。用更正式的话来说：对于没有字段的类型 T，类型 T 的每个实例 t 都是同一个对象。以下示例应该有助于澄清：

```
julia> struct Empty end

julia> empty = Empty()
Empty()

julia> none = Empty()
Empty()

julia> empty == none
true

julia> Empty() == Empty()
true
```

然而，不同空复合类型的实例被认为是不同的。因此，Empty()返回的对象与 Nothing()不同：

```
julia> Empty() == Nothing()
false

julia> empty = Empty()
Empty()

julia> empty == nothing
false
```

空复合类型使得在 Julia 中创建具有特殊意义的专用对象变得容易。按照惯例，Julia 使用 nothing 来表示某物找不到或不存在：

```
julia> findfirst("four", "one two three four")
15:18

julia> findfirst("four", "one two three")

julia> typeof(ans)
Nothing

julia> findfirst("four", "one two three") == nothing
true
```

## 10.2 在数据结构中使用 nothing

多级火箭类似于一个更通用的数据结构，称为 *链表*。就像火箭的例子一样，将多个对象链接起来通常很有用。例如，你可以使用它来表示由多个车厢组成的火车，这些车厢承载着一些货物。以下定义将不会工作。你能确定为什么吗？

列表 10.2 定义一个无限火车

```
struct Wagon
     cargo::Int                              ❶
     next::Wagon                             ❷
end

cargo(w::Wagon) = w.cargo + cargo(w.next)    ❸
```

❶ 火车车厢中有大量的货物

❷ 链接到这个车厢的下一节车厢

❸ 计算所有车厢中的总货物量。

使用我们给出的定义，无法构建由这些车厢组成的火车。我将通过一个例子来澄清：I’ll clarify with an example:

```
train = Wagon(3, Wagon(4, Wagon(1, Wagon(2, ....))))
```

无法结束这列车厢的链条。每个 Wagon 构造函数都需要一个车厢对象作为其第二个参数。为了说明无限车厢链条，我在代码示例中插入了 ....。下一个字段始终必须是一个 Wagon。但如果你将 Wagon 设为一个抽象类型呢？这是可能的解决方案之一，这已经在多级火箭的例子中得到了应用。

记住，并非每个 Rocket 子类型都有一个下一阶段字段。然而，在本章中，我将介绍一个更通用的解决方案来解决这个问题，利用 *参数化类型*。这只是为了介绍基础知识，因为第十八章完全致力于参数化类型。

重要参数化类型可能看起来只是对高级 Julia 程序员有吸引力的附加功能。然而，我故意在代码示例中尽量减少参数化类型的使用。现实世界的 Julia 代码广泛使用参数化类型。参数化类型对于类型正确性、性能和减少代码重复至关重要。

### 10.2.1 参数化类型是什么？

当我定义范围和配对时，你已经接触到了参数化类型。如果 P{T} 是一个参数化类型 P，那么 T 就是类型参数。我知道这听起来非常抽象，但通过一些例子，它将变得非常清晰：

```
julia> ['A', 'B', 'D']
3-element Vector{Char}:   ❶
 'A'
 'B'
 'D'

julia> typeof(3:4)
UnitRange{Int64}          ❷

julia> typeof(0x3//0x4)
Rational{UInt8}           ❸
```

❶ 带有 Char 类型参数的参数化类型 Vector

❷ 带有 Int64 类型参数的参数化类型 UnitRange

❸ 带有 UInt8 类型参数的参数化类型 Rational

你可以将 Vector 视为一个模板来创建一个实际类型。要创建一个具体的向量，你需要知道向量中元素的类型。在第一个例子中，类型参数是 Char，因为每个元素都是一个字符。对于 UnitRange，类型参数表示范围的起始和结束的类型。对于 Rational，类型参数指定分数中的分子和分母的类型。

类型参数对于参数化类型，就像值对于函数。你将一个值输入到函数中，得到一个值输出：

```
y = f(x)
```

你将 x 输入到函数 f 中，得到值 y。对于参数化类型，可以做出相同的类比：

```
S = P{T}
```

你将类型 T 输入到 P 中，得到类型 S。你可以通过一些实际的 Julia 类型来演示这一点：

```
julia> IntRange = UnitRange{Int}         ❶
UnitRange{Int64}

julia> FloatRange = UnitRange{Float64}   ❷
UnitRange{Float64}

julia> IntRange(3, 5)
3:5

julia> FloatRange(3, 5)                  ❸
3.0:5.0

julia> NumPair = Pair{Int, Float32}
Pair{Int64, Float32}

julia> NumPair(3, 5)
3 => 5.0f0

julia> 3 => 5
3 => 5
```

❶ 创建一个名为 IntRange 的范围类型。

❷ 基于浮点数创建一个范围类型。

❸ 使用自定义范围类型构建一个范围。

在这个例子中，你可以看到类型可以被处理得像对象一样。你创建新的类型对象，并将它们绑定到变量 IntRange、FloatRange 和 NumPair。然后使用这些自定义类型来实例化不同类型的对象。

### 10.2.2 使用联合类型结束车厢列车

联合是一种参数化类型。你可以提供多个类型参数来构建新的类型。联合类型是特殊的，因为它们可以作为任何列出的类型参数的占位符。或者，你可以将联合类型视为将两种或更多类型组合成一种类型的方法。

假设你有名为 T1、T2 和 T3 的类型。你可以通过编写 Union{T1, T2, T3}来创建这些类型的联合。这创建了一个新类型，它可以作为任何这些类型的占位符。这意味着如果你编写了一个具有签名的 f(x::Union{T1, T2, T3})的方法，那么每当 x 是 T1、T2 或 T3 类型时，这个特定的方法就会被调用。让我们看一个具体的例子：

```
julia> f(x::Union{Int, String}) = x³
f (generic function with 1 method)

julia> f(3)
27

julia> f(" hello ")
" hello  hello  hello "

julia> f(0.42)
ERROR: MethodError: no method matching f(::Float64)
Closest candidates are:
  f(!Matched::Union{Int64, String}) at none:1
```

最后一个例子失败是因为 x 是一个浮点数，而我们只定义了一个接受 Int 和 String 联合的函数 f 的方法。Float64 没有被包括在内。

在类型联合中包含的每个类型都将被计为该联合的子类型。你使用<:运算符来定义子类型或测试类型是否为子类型：

```
julia> String <: Union{Int64, String}
true

julia> Int64 <: Union{Int64, String}
true

julia> Float64 <: Union{Int64, String}
false

julia> Union{Int64, String} == Union{String, Int64}    ❶
 true
```

❶ 类型参数顺序无关紧要。

联合定义中类型参数的顺序无关紧要，如最后评估的表达式所示。有了联合类型，你可以用无限列车解决问题。

列表 10.3 定义有限列车

```
struct Wagon
     cargo::Int
     next::Union{Wagon, Nothing}             ❶
end

cargo(w::Wagon) = w.cargo + cargo(w.next)
cargo(w) = 0                                 ❷
```

❶ 下一个链接的车厢可以是空值。

❷ 非车厢的值没有货物。

重新加载你的 Julia REPL，并粘贴新的类型定义。此代码将允许你创建有限的车厢链。注意，已经定义了两种货物方法。你有两种不同的情况要处理，因为 next 可以是 Wagon 或无值：

```
julia> train = Wagon(3, Wagon(4, Wagon(1, nothing)))
Wagon(3, Wagon(4, Wagon(1, nothing)))

julia> cargo(train)
8

julia> train = Wagon(3, Wagon(4, Wagon(1, 42)))              ❶
ERROR: MethodError: Cannot `convert` an object of type Int64
to an object of type Wagon
```

❶ 尝试将 42 用作下一个车厢。

上一个例子包括是为了展示由于联合定义，next 只能是 Wagon 或 Nothing 对象。因此，将 next 设置为整数，如 42，是不合法的。这会导致 Julia 类型系统大声抗议，通过抛出异常。

## 10.3 缺失值

缺失值在 Julia 中用缺失对象表示，其类型为 Missing。这看起来与 nothing 非常相似，那么为什么你需要它呢？

这是因为 Julia 旨在成为科学计算、统计学、大数据等学术领域的良好语言。在统计学中，缺失数据是一个重要概念。它经常发生，因为在几乎任何用于统计的数据收集中都会有缺失数据。例如，你可能遇到参与者填写表格的情况，其中一些人未能填写所有字段。

一些参与者可能在研究完成前离开，留下那些进行实验的人拥有不完整的数据。缺失数据也可能由于数据录入错误而存在。所以，与“无”的概念不同，缺失数据实际上存在于现实世界中。我们只是不知道它是什么。

专门为统计学家设计的软件，如 R（见[`www.r-project.org`](https://www.r-project.org)）和 SAS（见[`www.sas.com`](https://www.sas.com)），长期以来一直认为缺失数据应该传播而不是抛出异常。这意味着如果在大计算中的任何部分引入了缺失值，整个计算将评估为缺失。Julia 也选择了遵循这一惯例。让我们看看这在实践中意味着什么。

列表 10.4 比较数学表达式中缺失值和“无”的行为

```
julia> missing < 10
missing

julia> nothing < 10
ERROR: MethodError: no method matching isless(::Nothing, ::Int64)

julia> 10 + missing
missing

julia> 10 + nothing
ERROR: MethodError: no method matching +(::Int64, ::Nothing)
```

你可以在列表 10.4 中看到，涉及缺失值的每个数学运算都会评估为缺失，与“无”不同，它会导致抛出异常。这样做的原因是，过去在统计工作中已经犯了许多严重的错误，这些错误源于没有注意到存在缺失值。由于缺失值在 Julia 中像病毒一样传播，未处理的缺失值会迅速被发现。

缺失值可以显式处理。例如，如果你想计算可能包含缺失值的数组的总和或平均值，你可以使用 skipmissing 函数来避免尝试将缺失值包含在结果中：

```
julia> using Statistics

julia> xs = [2, missing, 4, 8];

julia> sum(xs)                 ❶
missing

julia> sum(skipmissing(xs))    ❷
14

julia> median(skipmissing(xs))
4.0

julia> mean(skipmissing(xs))
4.666666666666667
```

❶ 缺失值的出现会污染整个计算，导致结果缺失。

❷ 跳过缺失值，因此你可以将非缺失值相加。

## 10.4 非数值

与缺失值多少有些相关的是浮点数 NaN（非数值）。当操作的结果未定义时，你会得到 NaN。这通常在除以零时成为一个问题：

```
julia> 0/0
NaN

julia> 1/0
Inf

julia> -1/0
-Inf
```

在这种情况下，Inf 代表*无穷大*，是除以非零数除以零时得到的结果。这有些道理。当除数接近零时，结果往往会变得更大。

很容易将 NaN 视为与缺失值相似，并且它们可以互换。毕竟，NaN 也会在所有计算中传播。

列表 10.5 数学运算中 NaN 的传播

```
julia> NaN + 10
NaN

julia> NaN/4
NaN

julia> NaN < 10
false

julia> NaN > 10
false
```

然而，NaN 的比较结果为假。以下是不应使用 NaN 作为缺失值的原因：如果你在算法中犯了一个错误导致 0/0 发生，你将得到 NaN。这将与输入缺失值无法区分。

列表 10.6 无法区分导致 NaN 的计算或输入为 NaN

```
julia> calc(x) = 3x/x;

julia> calc(0)
NaN

julia> calc(NaN)
NaN
```

你可能会错误地认为你的算法正在工作，因为它在计算中移除了缺失值，从而掩盖了一个缺陷。例如，在列表 10.6 中，你在除法之前没有检查输入 x 是否为零。因此，当 x 为 0 时，你得到一个 NaN 作为结果。如果你将 NaN 作为 calc 函数的输入以指示缺失值，那么你无法区分程序员错误和缺失值。

## 10.5 未定义数据

在 Julia 中，你很少会遇到未定义的数据，但了解这一点是值得的。未定义数据发生在变量或结构体的字段未设置时。通常，Julia 会尽量聪明地处理这种情况；如果你定义了一个具有数字字段的 struct，Julia 会自动将它们初始化为零，除非你做了其他操作。然而，如果你定义了一个没有告诉 Julia 字段类型的 struct，Julia 就没有方法猜测字段应该初始化为什么。

列表 10.7 定义一个复合类型，使用未定义的值实例化

```
julia> struct Person
           firstname
           lastname
           Person() = new()
       end

julia> friend = Person()
Person(#undef, #undef)

julia> friend.firstname
ERROR: UndefRefError: access to undefined reference
```

Julia 允许构建具有未初始化字段的复合对象。然而，如果你尝试访问一个未初始化的字段，它将抛出一个异常。未初始化的值没有好处，但它们有助于捕捉程序员的错误。

## 10.6 将所有内容组合在一起

区分这些 *nothing* 概念中的每一个可能有点令人畏惧，所以我将简要总结它们之间的区别：nothing 是程序员的 null 类型。当某物不存在时，程序员想要的就是这个。missing 是统计员的 null 类型。当他们的输入数据中缺少值时，他们想要的就是这个。NaN 表示在代码中某个地方发生了非法的数学操作。换句话说，这与计算有关，而不是与数据的统计收集有关。*未定义* 是指程序员忘记初始化所有使用的数据。这很可能是程序中的错误。

作为最后的提醒：Julia 在常规意义上没有 null，因为你需要使用类型联合显式允许 nothing 值。否则，函数参数不会意外地传递一个 nothing 值。

## 摘要

+   Julia 中的未知值由 nothing、missing、NaN 和 undefined 表示。

+   nothing 是类型 Nothing，表示不存在的东西。当查找函数失败或在数据结构中（例如，用于终止链表）时，将其用作返回值。

+   missing 是存在但缺失的数据，例如在调查中。它是类型 Missing。当实现从文件中读取统计数据的代码时，使用 missing 作为缺失数据的占位符。

+   NaN 是非法数学操作的结果。如果你的函数返回 NaN，你应该调查你是否犯了一个编程错误。例如，你确保你的代码中 0/0 永远不会发生吗？

+   未定义是指变量未初始化为已知值。

+   在该语言中，既没有“无”也没有“缺失”作为内置值，但它们被定义为没有任何字段的复合类型。

+   联合参数化类型与“无”和“缺失”类型一起使用非常实用。例如，如果一个字段可以是字符串或无，则将类型定义为 Union{Nothing, String}。
