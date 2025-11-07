# 附录 A Julia 的第一步

本附录涵盖

+   安装和设置 Julia

+   在 Julia 中获取帮助

+   寻找关于 Julia 的帮助信息的地方

+   在 Julia 中管理包

+   关于使用 Julia 的标准方式的概述

## A.1 安装和设置 Julia

在本节中，我将解释如何获取、安装和配置您的 Julia 环境。首先，访问“下载 Julia”网页 ([`julialang.org/downloads/`](https://julialang.org/downloads/))，并下载适合您所使用操作系统的 Julia 版本。

本书是用 Julia 1.7 编写和测试的。当您阅读本书时，可能在新版本的“下载”部分中可以获得更新的 Julia 版本。这不应该成为问题。我们使用的代码应该在 Julia 1.*x* 的任何新版本下都能运行。然而，例如，Julia 显示输出的方式可能会有细微的差异。如果您想使用与我编写本书时相同的 Julia 版本，您应该能够在“旧版未维护发布”页面 ([`julialang.org/downloads/oldreleases/`](https://julialang.org/downloads/oldreleases/)) 上找到 Julia 1.7。

下载适合您操作系统的相应 Julia 版本后，转到“官方二进制文件的平台特定说明”页面 ([`julialang.org/downloads/platform/`](https://julialang.org/downloads/platform/))，并遵循您操作系统的设置说明。特别是，请确保将 Julia 添加到您的 PATH 环境变量中，以便 Julia 可执行文件可以在您的系统上轻松运行（说明是针对特定操作系统的，并在网站上提供）。在此过程之后，您应该能够通过打开终端并输入 julia 命令来启动 Julia。

以下是从终端运行的最小 Julia 会话。在运行示例之前，我在电脑上打开了一个终端。$ 符号是我机器上的操作系统提示符。

您首先通过输入 julia 命令来启动 Julia。然后显示 Julia 标签和 julia> 提示符，表示您现在可以执行 Julia 命令。要终止 Julia，请输入 exit() 命令并返回操作系统，这由 $ 提示符表示：

```
$ julia                           ❶
               _
   _       _ _(_)_     |  Documentation: https://docs.julialang.org
  (_)     | (_) (_)    |
   _ _   _| |_  __ _   |  Type "?" for help, "]?" for Pkg help.
  | | | | | | |/ _` |  |
  | | |_| | | | (_| |  |  Version 1.7.2 (2022-02-06)
 _/ |\__'_|_|_|\__'_|  |  Official https://julialang.org/ release
|__/                   |

julia> exit()                     ❷

$                                 ❸
```

❶ 在操作系统提示符下，julia 命令启动 Julia。

❷ julia> 提示符表示我们处于一个 Julia 会话中。执行 exit() 函数将终止 Julia。

❸ 退出 Julia 后，我们回到了操作系统提示符。

在本书中，我始终使用这种展示输出的风格。您应该发送给 Julia 的所有命令都在 julia> 提示符之后给出。显示的其余文本是自动打印在终端上的内容。

## A.2 在 Julia 中以及关于 Julia 获取帮助

在本节中，我将解释如何在 Julia 中获取帮助以及如何查找标准资源，您可以从这些资源中学习关于 Julia 的知识。

Julia 有一个内置的帮助模式。当您处于此模式时，Julia 将尝试打印关于您输入文本的文档。

这里有一个关于如何获取 && 运算符帮助的示例。从 Julia 提示符开始跟随示例。首先，按问号键输入“?”。提示符将从 julia> 变为 help?>，表示你已进入帮助模式。现在输入 && 并按 Enter 键以获取有关此运算符的信息：

```
julia> ?      ❶

help?> &&
search: &&

  x && y

  Short-circuiting boolean AND.

  See also &, the ternary operator ? :, and the manual section on control flow.

  Examples
  ≡≡≡≡≡≡≡≡≡≡

  julia> x = 3;

  julia> x > 1 && x < 10 && x isa Int
  true

  julia> x < 0 && error("expected positive x")
  false
```

❶ 在键入“?”后，提示符（就地）变为帮助?>。

通常，在 Julia 文档中，你会得到关于给定命令的解释，以及有关其他相关命令的信息和如何使用它的示例。

除了内置的帮助之外，Julia 在网络上还有广泛的文档可用。最重要的资源是 Julia 文档（[`docs.julialang.org/en/v1/`](https://docs.julialang.org/en/v1/)），它分为三个部分。第一部分是语言的完整手册，第二部分涵盖了标准 Julia 安装中所有函数的文档，第三部分讨论了 Julia 的内部机制。

大多数包都有文档站点。例如，DataFrames.jl 包的文档位于 [`dataframes.juliadata.org/stable/`](https://dataframes.juliadata.org/stable/)。Julia 文档和为包创建的文档具有类似的设计。这是因为 Documenter.jl 包被用作从 docstrings 和 Markdown 文件构建文档的默认方法。

在 Julia 网站上，你可以在“学习”部分找到额外的教学材料链接（[`julialang.org/learning/`](https://julialang.org/learning/)）。它们包括 YouTube 视频、交互式教程以及一系列可以帮助你学习 Julia 工作各个方面的书籍。

Julia 的在线社区对于任何 Julia 用户来说都是重要的资源。如果你对 Julia 有任何疑问，我建议你从 Discourse 论坛开始（[`discourse.julialang.org`](https://discourse.julialang.org/)）。对于更随意的对话，你可以使用 Slack ([`julialang.org/slack/`](https://julialang.org/slack/)) 或 Zulip ([`julialang.zulipchat.com/register/`](https://julialang.zulipchat.com/register/))）。Julia 语言及其大多数包托管在 GitHub 上。因此，如果你想报告一个错误或提交一个功能请求，请在适当的 GitHub 仓库中打开一个问题。例如，对于 DataFrames.jl 包，你可以通过以下链接进行操作 [`github.com/JuliaData/DataFrames.jl/issues`](https://github.com/JuliaData/DataFrames.jl/issues)。

最后，Julia 在 Stack Overflow 上有 [julia] 标签的存在（[`stackoverflow.com/tags/julia`](https://stackoverflow.com/tags/julia)）。

## A.3 在 Julia 中管理包

Julia 的集成部分是其包管理器。它允许您安装和管理您可能在项目中想要使用的包。在本节中，我介绍了关于在 Julia 中管理包的最重要信息。关于此主题的深入讨论可在 Pkg.jl 文档（[`pkgdocs.julialang.org/v1/`](https://pkgdocs.julialang.org/v1/)）中找到。

### A.3.1 项目环境

讨论 Julia 中的包时的关键概念是*环境*：可以本地于单个项目或按名称共享和选择的独立包集合。环境中的包和版本的确切集合由 Project.toml 和 Manifest.toml 文件捕获。例如，在本书配套的 GitHub 存储库（[`github.com/bkamins/JuliaForDataAnalysis`](https://github.com/bkamins/JuliaForDataAnalysis)）中，您可以在根目录中找到这两个文件。您不需要手动编辑这些文件或详细了解其结构，但了解其内容是有用的。

Project.toml 文件指定了在给定项目中可以直接加载哪些包。以下是此文件的一部分，摘自本书的代码：

```
[deps]
Arrow = "69666777-d1a9-59fb-9406-91d4454c9d45"
BenchmarkTools = "6e4b80f9-dd63-53aa-95a3-0cdb28fa8baf"
CSV = "336ed68f-0bac-5ca0-87d4-7b16caf5d00b"
```

Manifest.toml 文件包含更多信息。它包括项目所需的所有包——即 Project.toml 文件中列出的包（称为*直接依赖*）以及为正确设置项目环境所需的所有其他包（Project.toml 中列出的包所需的包，称为*间接依赖*）。对于每个包，都给出了您项目中使用的确切版本信息。以下是此文件的一部分，摘自本书的代码：

```
# This file is machine-generated - editing it directly is not advised

julia_version = "1.7.2"
manifest_format = "2.0"

[[deps.AbstractFFTs]]
deps = ["ChainRulesCore", "LinearAlgebra"]
git-tree-sha1 = "69f7020bd72f069c219b5e8c236c1fa90d2cb409"
uuid = "621f4979-c628-5d54-868e-fcf4e3e8185c"
version = "1.2.1"

[[deps.Adapt]]
deps = ["LinearAlgebra"]
git-tree-sha1 = "af92965fb30777147966f58acb05da51c5616b5f"
uuid = "79e6a3ab-5dfb-504d-930d-738a2a938a0e"
version = "3.3.3"
```

总结来说，如果某个文件夹包含 Project.toml 和 Manifest.toml 文件，则它们定义了一个项目环境。

### A.3.2 激活项目环境

启动您的 Julia 会话后，您可以通过按键盘上的方括号键（]）进入包管理模式。当您这样做时，提示符将从 julia>变为 pkg>，表示您已进入包管理模式。默认情况下，此提示符将看起来像这样：

```
(@v1.7) pkg>
```

注意，在 pkg>提示符之前，您有(@v1.7)前缀。这表明 Julia 正在使用默认（全局）项目环境。默认环境由 Julia 为用户便利性提供，但建议不要在项目中依赖它，而是使用特定于项目的 Project.toml 和 Manifest.toml 文件。我将解释如何通过使用本书配套的 GitHub 存储库（[`github.com/bkamins/JuliaForDataAnalysis`](https://github.com/bkamins/JuliaForDataAnalysis)）来激活此环境。

在继续之前，请将此存储库下载到您的计算机上的一个文件夹中。在下面的示例中，我假设您已将此存储库存储在 D:\JuliaForDataAnalysis 文件夹中（这是一个 Windows 上的示例路径；在 Linux 或 macOS 上，路径将有所不同）。

要激活特定于项目的环境，你需要执行以下操作（这是最简单的情况）：

1.  使用 cd 函数将 Julia 的工作目录更改为 D:\JuliaForDataAnalysis 文件夹。

1.  使用 isfile 函数检查工作目录中是否存在 Project.toml 和 Manifest.toml 文件。（这并非严格必要，但我包括这一步是为了确保你在工作目录中有这些文件。）

1.  通过按下 ] 键切换到包管理器模式。

1.  使用 activate . 命令激活项目环境。

1.  可选地，通过使用 instantiate 命令来实例化环境。（这一步确保 Julia 从网络下载所有必需的包，如果你是第一次使用项目环境，则此步骤是必需的。）

1.  通过按下退格键离开包管理器模式。

这里是这些步骤的代码：

```
julia> cd("D:/JuliaForDataAnalysis")       ❶

julia> isfile("Project.toml")
true

julia> isfile("Manifest.toml")
true

(@v1.7) pkg> activate .                    ❷
  Activating project at `D:\JuliaForDataAnalysis`

(JuliaForDataAnalysis) pkg> instantiate

julia>                                     ❸
```

❶ 在 Windows 中，你可以使用斜杠 (/) 而不是反斜杠 (\) 作为路径分隔符。

❷ 按下 ] 键切换到包管理器模式。

❸ 按下退格键返回到 Julia 模式。

注意，在 Windows 中，你可以使用斜杠 (/) 而不是标准反斜杠 (\) 作为路径分隔符。

在 activate 命令中，我们传递一个点 (.)，这表示 Julia 的当前工作目录。我们可以通过 cd 函数避免更改工作目录，而是在 activate 命令中传递环境的路径，如下所示：

```
(@v1.7) pkg> activate D:/JuliaForDataAnalysis
  Activating project at `D:\JuliaForDataAnalysis`

(JuliaForDataAnalysis) pkg>
```

我更喜欢将 Julia 的工作目录切换到存储 Project.toml 和 Manifest.toml 文件的地方，因为通常它们存储在其他项目文件（如 Julia 代码或源数据）相同的目录中。

观察到更改项目环境后，其名称将作为 pkg> 提示符的前缀显示。在我们的例子中，这个前缀是 (JuliaForDataAnalysis)。

在激活环境后，你将执行的所有操作（例如，使用包或添加或删除包）都将在这个激活的环境中完成。

在以下典型场景中，项目环境的激活被简化了：

+   如果你在一个包含 Project.toml 和 Manifest.toml 文件的文件夹中的终端操作系统提示符下，那么当你使用 julia --project 调用启动 Julia 时，由这些文件定义的项目环境将自动激活。

+   如果你正在使用 Visual Studio Code（在第 A.4 节中讨论），并且已经打开包含 Project.toml 和 Manifest.toml 文件的文件夹，然后启动 Julia 服务器，Visual Studio Code 将自动激活由这些文件定义的项目环境。

+   如果你正在使用 Jupyter 交互式环境（在第 A.4 节中讨论），那么，类似于前面的场景，如果包含 Jupyter 笔记本的文件夹也包含 Project.toml 和 Manifest.toml 文件，那么由它们定义的环境将自动激活。

运行本书中的代码示例

伴随这本书的 GitHub 仓库 ([`github.com/bkamins/Julia ForDataAnalysis`](https://github.com/bkamins/JuliaForDataAnalysis)) 包含 Project.toml 和 Manifest.toml 文件，这些文件指定了我展示的所有代码示例中使用的项目环境。因此，我建议当您测试这本书中的任何代码示例时，请确保在激活此项目环境的情况下运行它们。这将确保您不需要手动安装任何包，并且您使用的包的版本与我创建书籍时使用的版本相匹配。

### A.3.3 安装包可能遇到的问题

一些 Julia 包在使用之前需要外部依赖。如果您在 Linux 上工作，这个问题主要会遇到。如果是这种情况，相应 Julia 包的文档通常会提供所有必需的安装说明。

例如，如果我们考虑这本书中使用的包，如果您想使用 Plots.jl 进行绘图，可能需要一些配置。默认情况下，此包使用 GR 框架 ([`gr-framework.org`](https://gr-framework.org/)) 来显示创建的图表。在 Linux 上，要使用此框架，您需要安装几个依赖项，如 [`gr-framework.org/julia.html`](https://gr-framework.org/julia.html) 中所述。例如，如果您使用 Ubuntu，请使用以下命令确保所有依赖项都可用：

```
apt install libxt6 libxrender1 libxext6 libgl1-mesa-glx libqt5widgets5
```

使用依赖于外部二进制依赖项的包时可能遇到的另一个潜在问题是，您可能需要手动调用它们的构建脚本。例如，当依赖项的二进制文件发生变化时，有时需要这样做。在这种情况下，在包管理器模式下调用构建命令（提示符应该是 pkg>）。这将调用所有具有构建脚本的包的构建脚本。

### A.3.4 管理包

在您激活并实例化项目环境后，您可以开始编写使用给定环境提供的包的 Julia 程序。然而，您有时会想要管理可用的包。最常见的包管理操作是列出可用的包、添加包、删除包和更新包。我将向您展示如何执行这些操作。在以下示例中，我将在一个空文件夹 D:\Example 中工作，以确保我们不会意外修改您已有的项目环境。

首先，创建 D:\Example 文件夹（或任何空文件夹），并在该文件夹中启动您的终端。接下来，使用 julia 命令启动 Julia，并使用 pwd 函数确保您在适当的文件夹中：

```
$ julia
               _
   _       _ _(_)_     |  Documentation: https://docs.julialang.org
  (_)     | (_) (_)    |
   _ _   _| |_  __ _   |  Type "?" for help, "]?" for Pkg help.
  | | | | | | |/ _` |  |
  | | |_| | | | (_| |  |  Version 1.7.2 (2022-02-06)
 _/ |\__'_|_|_|\__'_|  |  Official https://julialang.org/ release
|__/                   |

julia> pwd()
"D:\\Example"
```

现在通过按下 ] 键切换到包管理器模式，并激活当前工作目录中的环境：

```
(@v1.7) pkg> activate .
  Activating new project at `D:\Example`

(Example) pkg>
```

这是一个新的空环境。我们可以通过运行状态命令来检查它：

```
(Example) pkg> status
      Status `D:\Example\Project.toml` (empty project)
```

现在，我们通过使用 add BenchmarkTools 命令将 BenchmarkTools.jl 包添加到这个环境中：

```
(Example) pkg> add BenchmarkTools
    Updating registry at `D:\.julia\registries\General`
    Updating git-repo `https://github.com/JuliaRegistries/General.git`
   Resolving package versions...
    Updating `D:\Example\Project.toml`
  [6e4b80f9] + BenchmarkTools v1.3.1
    Updating `D:\Example\Manifest.toml`
  [6e4b80f9] + BenchmarkTools v1.3.1
  [682c06a0] + JSON v0.21.3
  [69de0a69] + Parsers v2.2.3
  [56f22d72] + Artifacts
  [ade2ca70] + Dates
  [8f399da3] + Libdl
  [37e2e46d] + LinearAlgebra
  [56ddb016] + Logging
  [a63ad114] + Mmap
  [de0858da] + Printf
  [9abbd945] + Profile
  [9a3f8284] + Random
  [ea8e919c] + SHA
  [9e88b42a] + Serialization
  [2f01184e] + SparseArrays
  [10745b16] + Statistics
  [cf7118a7] + UUIDs
  [4ec0a83e] + Unicode
  [e66e0078] + CompilerSupportLibraries_jll
  [4536629a] + OpenBLAS_jll
  [8e850b90] + libblastrampoline_jll
```

在此过程中，我们得到的信息表明 BenchmarkTools 条目被添加到 Project.toml 文件中，并且一个包列表被添加到 Manifest.toml 文件中。加号 (+) 字符表示添加了一个包。回想一下，Manifest.toml 包含了我们项目的直接依赖项以及其他需要正确设置项目环境的包。

让我们再次检查我们的项目环境状态：

```
(Example) pkg> status
      Status `D:\Example\Project.toml`
  [6e4b80f9] BenchmarkTools v1.3.1
```

我们看到现在我们已经安装了 BenchmarkTools.jl 包，版本为 1.3.1。

经过一段时间后，BenchmarkTools.jl 可能会有新的版本发布。你可以通过使用更新命令来更新已安装包的版本到最新发布版。在我们的例子中，因为我们刚刚安装了 BenchmarkTools.jl 包，所以该命令不会做出任何更改：

```
(Example) pkg> update
    Updating registry at `D: \.julia\registries\General`
    Updating git-repo `https://github.com/JuliaRegistries/General.git`
  No Changes to `D:\Example\Project.toml`
  No Changes to `D:\Example\Manifest.toml`
```

最后，如果你想从你的项目环境中移除一个包，请使用移除命令：

```
(Example) pkg> remove BenchmarkTools
    Updating `D:\Example\Project.toml`
  [6e4b80f9] - BenchmarkTools v1.3.1
    Updating `D:\Example\Manifest.toml`
  [6e4b80f9] - BenchmarkTools v1.3.1
  [682c06a0] - JSON v0.21.3
  [69de0a69] - Parsers v2.2.3
  [56f22d72] - Artifacts
  [ade2ca70] - Dates
  [8f399da3] - Libdl
  [37e2e46d] - LinearAlgebra
  [56ddb016] - Logging
  [a63ad114] - Mmap
  [de0858da] - Printf
  [9abbd945] - Profile
  [9a3f8284] - Random
  [ea8e919c] - SHA
  [9e88b42a] - Serialization
  [2f01184e] - SparseArrays
  [10745b16] - Statistics
  [cf7118a7] - UUIDs
  [4ec0a83e] - Unicode
  [e66e0078] - CompilerSupportLibraries_jll
  [4536629a] - OpenBLAS_jll
  [8e850b90] - libblastrampoline_jll

(Example) pkg> status
      Status `D:\Example\Project.toml` (empty project)
```

包管理器不仅从 Project.toml 中移除了 BenchmarkTools 包，还从 Manifest.toml 中移除了所有不必要的包。减号 (-) 字符表示移除一个包。

### A.3.5 设置与 Python 的集成

PyCall.jl 包提供了 Julia 与 Python 的集成。我们在第五章中讨论了该包的使用。在这里，我讨论了在 Windows 和 Mac 系统上安装该包的过程。

在我们刚刚创建的 Example 项目环境中，添加 PyCall.jl 包（我裁剪了添加到 Manifest.toml 中的包列表，因为它很长）：

```
(Example) pkg> add PyCall
   Resolving package versions...
    Updating `D:\Example\Project.toml`
  [438e738f] + PyCall v1.93.1
    Updating `D:\Example\Manifest.toml`
  [8f4d0f93] + Conda v1.7.0
  [682c06a0] + JSON v0.21.3
  [1914dd2f] + MacroTools v0.5.9

...

  [83775a58] + Zlib_jll
  [8e850b90] + libblastrampoline_jll
  [8e850ede] + nghttp2_jll
```

通常，整个配置过程应该自动完成，此时你应该能够开始使用 Python。让我们检查 PyCall.jl 包默认使用的 Python 可执行文件的路径：

```
julia> using PyCall

julia> PyCall.python
"C:\\Users\\user\\.julia\\conda\\3\\python.exe"
```

如你所见，Python 安装在 conda 目录中的 Julia 安装内部。这是因为默认情况下，在 Windows 和 Mac 系统上安装 PyCall.jl 包时，会安装一个专属于 Julia 的最小 Python 发行版（通过 Miniconda）（不在你的 PATH 中）。或者，你可以指定应该使用的另一个 Python 安装，如文档中所述（[`mng.bz/lRVM`](http://mng.bz/lRVM)）。

在 GNU/Linux 系统下，情况不同，PyCall.jl 将默认使用你的 PATH 中的 python3 程序（如果有——否则使用 python）。

### A.3.6 设置与 R 的集成

RCall.jl 包提供了 Julia 与 R 语言的集成。我们在第十章中讨论了这个包。我将向你展示如何安装它。

作为第一步，我建议你下载并安装 R 到你的机器上。你必须在你开始 Julia 会话之前这样做，否则在尝试安装 RCall.jl 包时你会得到错误。

Windows 用户可以在 [`cran.r-project.org/bin/windows/base/`](https://cran.r-project.org/bin/windows/base/) 找到安装说明，macOS 用户在 [`cran.r-project.org/bin/macosx/`](https://cran.r-project.org/bin/macosx/)。

如果你使用 Linux，安装将取决于你使用的发行版。如果你使用 Ubuntu，你可以在 [`mng.bz/BZAg`](http://mng.bz/BZAg) 找到可以遵循的说明。

完成此安装后，当 RCall.jl 包被添加时，操作系统应该能够自动检测它。

在 Example 项目环境中，我们添加了 RCall.jl 包（我剪裁了添加到 Manifest.toml 中的包列表，因为它很长）：

```
(Example) pkg> add RCall
   Resolving package versions...
   Installed DualNumbers ______ v0.6.7
   Installed NaNMath __________ v1.0.0
   Installed InverseFunctions _ v0.1.3
   Installed Compat ___________ v3.42.0
   Installed LogExpFunctions __ v0.3.7
    Updating `D:\Example\Project.toml`
  [6f49c342] + RCall v0.13.13
    Updating `D:\Example\Manifest.toml`
  [49dc2e85] + Calculus v0.5.1
  [324d7699] + CategoricalArrays v0.10.3
  [d360d2e6] + ChainRulesCore v1.13.0

...

  [cf7118a7] + UUIDs
  [05823500] + OpenLibm_jll
  [3f19e933] + p7zip_jll
```

通常，RCall.jl 包应该能够自动检测你的 R 安装。你可以通过使用该包并检查 R 可执行文件的位置来验证它是否正常工作：

```
julia> using RCall

julia> RCall.Rhome
"C:\\Program Files\\R\\R-4.1.2"
```

如果你在机器上自动检测 R 安装时遇到问题，请参考文档（[`mng.bz/dedX`](http://mng.bz/dedX)）以获取更详细的说明，因为这些说明取决于你使用的操作系统。

## A.4 检查与 Julia 交互的标准方式

在本节中，我将讨论用户与 Julia 交互的四种最常见方式：

+   使用终端和 Julia 可执行文件

+   使用 Visual Studio Code

+   使用 Jupyter Notebook

+   使用 Pluto Notebook

### A.4.1 使用终端

终端是与 Julia 交互的最基本方式。你有两种运行 Julia 的选项。

第一种选项是通过运行 julia 可执行文件来启动一个交互会话。然后，如 A.1 节所述，你将看到 julia> 提示符，并能够交互式地执行 Julia 命令。

第二种选项是运行 Julia 脚本。如果你的 Julia 代码存储在文件中（例如，命名为 code.jl），那么通过运行 julia code.jl，你将要求 Julia 运行存储在 code.jl 中的代码，然后终止。

此外，Julia 可执行文件可以接受多个命令行选项和开关。你可以在 Julia 手册的“命令行选项”部分找到完整的列表（[`mng.bz/rnjZ`](http://mng.bz/rnjZ)）。

### A.4.2 使用 Visual Studio Code

与 Julia 一起工作的流行选项之一是使用 Visual Studio Code。你可以在 [`code.visualstudio.com/`](https://code.visualstudio.com/) 下载此集成开发环境。

接下来，你需要安装 Julia 扩展。你可以在 [`mng.bz/VyRO`](http://mng.bz/VyRO) 找到说明。该扩展提供了内置动态自动完成、内联结果、绘图面板、集成 REPL、变量视图、代码导航、调试器等功能。查看扩展的文档以了解如何使用和配置所有选项。

### A.4.3 使用 Jupyter Notebook

Julia 代码可以在 Jupyter Notebook 中运行（[`jupyter.org`](https://jupyter.org/)）。这种组合允许你通过使用图形笔记本与 Julia 语言进行交互，该笔记本将代码、格式化文本、数学和多媒体结合在一个文档中。

要在 Jupyter Notebook 中使用 Julia，首先安装 IJulia.jl 包。接下来，在浏览器中运行 IJulia Notebook 的最简单方法是运行以下代码：

```
using IJulia
notebook()
```

对于更高级的安装选项，例如指定要使用的特定 Jupyter 安装，请参阅 IJulia.jl 包的文档 ([`julialang.github.io/IJulia.jl/stable/`](https://julialang.github.io/IJulia.jl/stable/))。

### A.4.4 使用 Pluto 笔记本

Pluto 笔记本允许您将代码和文本结合起来，就像 Jupyter 笔记本一样。区别在于 Pluto 笔记本是响应式的：如果您更改一个变量，Pluto 会自动重新运行引用该变量的单元格。单元格可以在笔记本中任意顺序放置，因为它会自动识别它们之间的依赖关系。此外，Pluto 笔记本了解笔记本中正在使用哪些包。您无需自己安装包，因为 Pluto 笔记本会自动为您管理项目环境。

您可以在包网站上了解更多关于 Pluto 笔记本功能以及如何使用它们的信息 ([`github.com/fonsp/Pluto.jl`](https://github.com/fonsp/Pluto.jl))。
