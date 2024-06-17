# 第一章\. 开发设置

本章涵盖了下载和安装本书所需的软件，并概述了推荐的开发环境。正如你将看到的，这并不像过去那样繁琐。我将分别介绍Python和JavaScript的依赖关系，并简要概述跨语言IDE。

# 附带的代码

本书涵盖的大部分代码都可以在GitHub仓库中找到，包括完整的诺贝尔奖可视化。要获取它，只需将其[git clone](https://git-scm.com/docs/git-clone)到合适的本地目录：

```py
$ git clone https://github.com/Kyrand/dataviz-with-python-and-js-ed-2.git
```

这将创建一个名为*dataviz-with-python-and-js-v2*的本地目录，并包含书中涵盖的关键源代码。

# Python

本书涵盖的大部分库都是基于Python的，但是Anaconda的存在使得为各种操作系统及其怪癖提供全面安装说明的尝试变得更加容易，Anaconda是一个Python平台，它将大多数流行的分析库捆绑在一个方便的软件包中。本书假设您正在使用自2008年发布以来已经稳固确立的Python 3。

## Anaconda

曾经安装一些较大的Python库本身就是一个挑战，特别是那些依赖于复杂低级C和Fortran包的库，如NumPy。现在这变得简单得多，大多数可以使用Python的`easy_install`和`pip`命令轻松安装：

```py
$ pip install NumPy
```

但是某些大数据处理库仍然难以安装。依赖管理和版本控制（您可能需要在同一台机器上使用不同版本的Python）可能会使事情变得更加复杂，这正是Anaconda发挥作用的地方。它执行所有的依赖检查和二进制安装，使您免于烦恼。对于像这样的书籍资源来说，这也非常方便。

要获取免费的Anaconda安装，请只需将浏览器导航到[Anaconda网站](https://oreil.ly/4FkCT)，选择适合您操作系统版本（理想情况下至少是Python 3.5），然后按照说明操作。Windows和OS X有图形安装程序（只需下载并双击），而Linux需要您运行一个小的bash脚本：

```py
$ bash Anaconda3-2021.11-Linux-x86_64.sh
```

这是最新的安装说明：

+   [对于Windows](https://oreil.ly/KErTO)

+   [对于macOS](https://oreil.ly/5cVfC)

+   [对于Linux](https://oreil.ly/tIQT5)

我建议在安装Anaconda时保持默认设置。

可以在[Anaconda网站](https://oreil.ly/tL7c9)找到官方检查指南。Windows和macOS用户可以使用Anaconda的Navigator GUI，或者与Linux用户一起使用Conda命令行界面。

## 安装额外的库

Anaconda包含了本书涵盖的几乎所有Python库（详见[Anaconda文档](https://oreil.ly/c2vRS)中的完整Anaconda库包列表）。在我们需要非Anaconda库时，可以使用[`pip`](https://oreil.ly/b0Eni)（Pip Installs Python的简称），这是安装Python库的事实标准。使用`pip`安装非常简单。只需在命令行中调用`pip install`，然后跟上包的名称，它就会被安装，或者出现一个合理的错误信息：

```py
$ pip install dataset
```

## 虚拟环境

[虚拟环境](https://oreil.ly/x7Uq5)提供了一种使用特定的Python版本和/或第三方库集创建沙盒式开发环境的方式。使用这些虚拟环境可以避免在全局Python环境中安装这些软件包，并且给你更多的灵活性（你可以尝试不同的包版本或者改变Python版本）。在Python开发中，使用虚拟环境已经成为一种最佳实践，我强烈建议你遵循这个做法。

Anaconda附带一个`conda`系统命令，使得创建和使用虚拟环境变得简单。让我们为本书创建一个特别的虚拟环境，基于完整的Anaconda包：

```py
$ conda create --name pyjsviz anaconda
...
#
# To activate this environment, use:
# $ source activate pyjsviz
#
# To deactivate this environment, use:
# $ source deactivate
#
```

正如最后的消息所说，要使用这个虚拟环境，你只需`source activate`它（对于Windows机器，可以省略`source`）：

```py
$ source activate pyjsviz
discarding /home/kyran/anaconda/bin from PATH
prepending /home/kyran/.conda/envs/pyjsviz/bin to PATH
(pyjsviz) $
```

注意，在命令行中会得到一个有用的提示，让你知道正在使用哪个虚拟环境。

`conda`命令不仅仅能够简化虚拟环境的使用，还结合了Python的`pip`安装器和`virtualenv`命令的功能等。你可以在[Anaconda文档](https://oreil.ly/KN0ZL)中获得完整的介绍。

如果你对标准的Python虚拟环境感到自信，它们已经通过将其整合到Python标准库中而变得更加易于使用。要从命令行创建一个虚拟环境：

```py
$ python -m venv python-js-viz
```

这将创建一个`python-js-viz`目录，其中包含虚拟环境的各种元素。这包括一些激活脚本。要在macOS或Linux上激活虚拟环境，运行激活脚本：

```py
$ source python-js-viz/bin/activate
```

在Windows机器上，运行*.bat*文件：

```py
$ python-js-viz/Scripts/activate.bat
```

然后你可以使用`pip`来安装Python库到虚拟环境，避免污染全局Python分布：

```py
$ (python-js-viz) pip install NumPy
```

要安装本书所需的所有库，可以使用书的[GitHub存储库](https://github.com/Kyrand/dataviz-with-python-and-js-ed-2)中的*requirements.txt*文件：

```py
$ (python-js-viz) pip install -r requirements.txt
```

你可以在Python文档的[虚拟环境](https://oreil.ly/dhCvZ)部分找到相关信息。

# JavaScript

好消息是，你几乎不需要任何JavaScript软件。唯一必须的是在本书中使用的Chrome/Chromium Web浏览器。它提供了任何当前浏览器中最强大的一套开发工具，并且跨平台。

要下载Chrome，只需访问[主页](https://oreil.ly/jNTUl)，然后下载适合您操作系统版本的程序。这应该会自动检测到。

本书中使用的所有JavaScript库都可以在附带的[GitHub仓库](https://github.com/Kyrand/dataviz-with-python-and-js-ed-2)中找到，但通常有两种方法将它们传递到浏览器。您可以使用内容交付网络（CDN），它有效地缓存从交付网络检索到的库的副本。或者，您可以使用本地库的副本传递到浏览器。这两种方法都使用HTML文档中的`script`标签。

## 内容交付网络

使用CDN，而不是在本地机器上安装库，浏览器从最近的可用服务器上获取JavaScript，从而使事情变得非常快速——比您自己提供内容更快。

要通过CDN包含库，您使用通常放置在HTML页面底部的常规`<script>`标签。例如，以下调用会添加当前版本的D3：

```py
<script
 src="https://cdnjs.cloudflare.com/ajax/libs/d3/7.1.1/d3.min.js"
 charset="utf-8">
</script>
```

## 在本地安装库

如果需要在本地安装JavaScript库，例如预期进行离线开发工作或无法保证网络连接，则有许多非常简单的方法可以实现。

您只需下载单独的库并将其放入本地服务器的静态文件夹中即可。这是典型的文件夹结构。第三方库放在根目录下的*static/libs*目录中，如下所示：

```py
nobel_viz/
└── static
    ├── css
    ├── data
    ├── libs
    │     └── d3.min.js
    └── js
```

如果您以这种方式组织事务，现在使用D3在您的脚本中需要使用`<script>`标签引用本地文件：

```py
<script src="/static/libs/d3.min.js"></script>
```

# 数据库

推荐用于中小型数据可视化项目的建议数据库是非常出色的、无服务器、基于文件的SQL数据库[SQLite](https://www.sqlite.org)。该数据库在书中展示的数据可视化工具链中被广泛使用，是您真正需要的唯一数据库。

这本书还涵盖了基本的Python与最受欢迎的非关系型数据库[MongoDB](https://www.mongodb.org)的交互。

SQLite

SQLite应该作为macOS和Linux机器的标准配备。对于Windows，请参阅[此指南](https://oreil.ly/Ck6qR)。

MongoDB

您可以在MongoDB文档中的各种操作系统的安装说明中找到[安装说明](https://oreil.ly/JIt8R)。

请注意，我们将直接或通过依赖它的库（例如[SQLAlchemy](https://www.sqlalchemy.org) SQL库）使用Python。这意味着我们可以通过更改一两行配置来将任何SQLite示例转换为其他SQL后端（例如[MySQL](https://www.mysql.com)或[PostgreSQL](https://www.postgresql.org)）。

## 启动和运行MongoDB

MongoDB 的安装可能比某些数据库复杂一些。正如提到的那样，你可以完全不安装基于服务器的 MongoDB 而完美地使用本书，但如果你想尝试或者在工作中需要使用它，这里有一些安装注意事项：

对于 OS X 用户，请查阅[官方文档](https://oreil.ly/zTEH5)获取 MongoDB 安装指南。

这篇来自官方文档的[针对 Windows 的特定指南](https://oreil.ly/OI5gB)应该能帮助你启动 MongoDB 服务器。你可能需要使用管理员权限来创建必要的数据目录等。

如今，你更可能将 MongoDB 安装到基于 Linux 的服务器上，最常见的是 Ubuntu 变种，它使用[deb](https://oreil.ly/rRQrG)文件格式来提供其软件包。[官方 MongoDB 文档](https://oreil.ly/SrRzJ)在涵盖 Ubuntu 安装方面做得很好。

MongoDB 使用一个*数据*目录来存储数据，并且，根据你的安装方式，你可能需要自己创建这个目录。在 OS X 和 Linux 系统上，默认是在根目录下的一个*数据*目录，你可以使用`mkdir`命令作为超级用户（`sudo`）创建它：

```py
$ sudo mkdir /data
$ sudo mkdir /data/db
```

然后你需要设置所有权为自己：

```py
$ sudo chown 'whoami' /data/db
```

在 Windows 上，安装[MongoDB 社区版](https://oreil.ly/3Vtft)后，你可以使用以下命令创建必要的*数据*目录：

```py
$ cd C:\
$ md "\data\db"
```

MongoDB 服务器通常会在 Linux 系统上默认启动；否则，在 Linux 和 OS X 上，可以使用以下命令启动服务器实例：

```py
$ mongod
```

在 Windows 社区版上，从命令提示符运行以下命令将启动一个服务器实例：

```py
C:\mongodb\bin\mongod.exe
```

## 使用 Docker 轻松安装 MongoDB

MongoDB 的安装可能有些棘手。例如，当前的 Ubuntu 变种（> 版本 22.04）存在[不兼容的 SSL 库](https://oreil.ly/ShOjF)。如果你已经安装了[Docker](https://oreil.ly/ZF5Uf)，一个工作的开发数据库在默认端口 27017 上只需一个命令即可：

```py
$ sudo docker run -dp 27017:27017 -v local-mongo:/data/db
              --name local-mongo --restart=always mongo
```

这很好地避开了本地库不兼容性等问题。

# 集成开发环境

正如我在[《IDE、框架和工具的神话》](ch04.xhtml#myth_ides)中所解释的，你并不需要一个 IDE 来编写 Python 或 JavaScript。现代浏览器（尤其是 Chrome）提供的开发工具意味着你实际上只需要一个好的代码编辑器就能拥有几乎最佳的设置。

这里的一个注意是，如今，中级到高级的 JavaScript 开发通常涉及到像 React、Vue 和 Svelte 这样的框架，这些框架受益于一个体面的 IDE 提供的各种功能，特别是处理多格式文件（其中 HTML、CSS 和 JS 都混合在一起）。好消息是，免费的[Visual Studio Code](https://code.visualstudio.com)（VSCode）已成为现代 Web 开发的事实标准。它有几乎所有插件，并且拥有庞大且活跃的社区，因此问题通常能够迅速得到解答和缺陷被解决。

对于Python，我尝试过几个专用的IDE，但它们从未让我满意。我试图解决的主要问题是找到一个像样的调试系统。在Python中使用文本编辑器设置断点并不是特别优雅，使用命令行调试器`pdb`有时感觉有点老派。尽管如此，Python确实包含了一个相当不错的日志系统，可以稍微减轻其默认调试器的笨拙。VSCode对于Python编程来说相当不错，但还有一些专门的Python IDE可能更为流畅。

以下是我尝试过的几个，并且不是很讨厌的：

[PyCharm](https://www.jetbrains.com/pycharm)

这个选项提供了强大的代码辅助和良好的调试功能，可能会成为经验丰富的Python程序员最喜欢的IDE之一。

[PyDev](https://pydev.org)

如果你喜欢Eclipse并且可以容忍其相当大的占用空间，这可能非常适合你。

[Wing Python IDE](https://www.wingware.com)

这是一个可靠的选择，具有出色的调试器，并在十五年开发中逐步改进。

# 摘要

通过免费的打包Python发行版，如Anaconda，以及在免费可用的Web浏览器中包含的复杂JavaScript开发工具，您所需的Python和JavaScript元素就只是几个点击之遥。再加上一个喜欢的编辑器和一个选择的数据库^([1](ch01.xhtml#idm45607814876176-marker))，您基本上就可以开始了。还有一些附加库，比如*node.js*，可能会很有用，但不算必要。既然我们已经建立了我们的编程环境，接下来的章节将教授启动我们的数据转换之旅所需的基础知识，从Python和JavaScript之间的语言桥梁开始，沿着工具链逐步推进。

^([1](ch01.xhtml#idm45607814876176-marker)) SQLite非常适合开发目的，而且不需要在您的机器上运行服务器。
