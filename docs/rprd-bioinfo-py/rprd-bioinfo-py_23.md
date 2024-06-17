# 附录 B. 理解 $PATH 和安装命令行程序

`PATH` 是一个环境变量，定义了在给定命令搜索时将要查找的目录。也就是说，如果我输入 `foo`，并且在我的 `PATH` 中既没有内置命令、shell 函数、命令别名，也没有可以作为 **`foo`** 执行的程序，那么我将会收到找不到该命令的提示：

```py
$ foo
-bash: foo: command not found
```

在 Windows PowerShell 中，我可以使用 **`echo $env:Path`** 检查 `PATH`，而在 Unix 平台上，我使用 **`echo $PATH`** 命令。两个路径都作为一个长字符串打印出来，没有空格，列出所有目录名称，Windows 上以分号分隔，Unix 上以冒号分隔。如果操作系统没有路径的概念，它将不得不在机器上的 *每个目录* 中搜索给定的命令。这可能需要几分钟到几小时，因此将搜索限制在几个目录中是有道理的。

接下来是我在我的 Macintosh 上的路径。注意，我必须在名称前面加上一个美元符号 (`$`)，告诉我的 shell (`bash`) 这是一个变量，而不是字面字符串 `PATH`。为了使这更易读，我将使用 Perl 将冒号替换为换行符。请注意，此命令仅在安装了 Perl 的 Unix 命令行上才能正常工作：

```py
$ echo $PATH | perl -pe 's/:/\n/g' ![1](assets/1.png)
/Users/kyclark/.local/bin ![2](assets/2.png)
/Library/Frameworks/Python.framework/Versions/3.9/bin ![3](assets/3.png)
/usr/local/bin ![4](assets/4.png)
/usr/bin ![5](assets/5.png)
/bin
/usr/sbin
/sbin
```

[![1](assets/1.png)](#co_understanding__path_and_installing_command_line_programs_CO1-1)

Perl 替换 (`s//`) 命令将第一个模式 (`:`) 全局替换为第二个 (`\n`)。

[![2](assets/2.png)](#co_understanding__path_and_installing_command_line_programs_CO1-2)

这是我通常为安装自己的程序创建的自定义目录。

[![3](assets/3.png)](#co_understanding__path_and_installing_command_line_programs_CO1-3)

这是 Python 安装自身的位置。

[![4](assets/4.png)](#co_understanding__path_and_installing_command_line_programs_CO1-4)

这是一个标准的用户安装软件目录。

[![5](assets/5.png)](#co_understanding__path_and_installing_command_line_programs_CO1-5)

其余的目录都是用来查找程序的更标准的目录。

目录将按照它们定义的顺序进行搜索，因此顺序可能非常重要。例如，Python 路径在系统路径之前列出，以便当我输入 **`python3`** 时，它会使用我本地 Python 目录中找到的版本，而不是可能预先安装在系统上的版本。

注意，我 `PATH` 中的所有目录名称都以 *bin* 结尾。这是 *二进制* 的缩写，来自于许多程序以二进制形式存在的事实。例如，C 程序的源代码以一种伪英语语言编写，编译成机器可读的可执行文件。该文件的内容是二进制编码的指令，操作系统可以执行。

相比之下，Python程序通常安装为其源代码文件，在运行时由Python执行。如果你想全局安装你的Python程序之一，我建议你将其复制到已列出在你的`PATH`中的目录之一。例如，*/usr/local/bin*是用户通过*本地*安装软件的典型目录。这是一个如此常见的目录，以至于通常出现在`PATH`中。如果你在个人设备上工作，比如笔记本电脑，在这里你拥有管理员权限，你应该能够将新文件写入此位置。

例如，如果我想要能够在[第一章](ch01.html#ch01)中运行`dna.py`程序而不提供源代码的完整路径，我可以将其复制到我的`PATH`中的某个位置：

```py
$ cp 01_dna/dna.py /usr/local/bin
```

然而，你可能没有足够的权限来执行此操作。Unix系统从一开始就设计成*多租户*操作系统，这意味着它们支持许多不同的人同时使用系统。重要的是防止用户写入和删除他们不应该的文件，因此操作系统可能会阻止你将`dna.py`写入你不拥有的目录。例如，如果你在大学的共享高性能计算（HPC）系统上工作，你肯定没有这样的特权。

当无法安装到系统目录时，最简单的方法是在你的`HOME`目录中创建一个位置用于这些文件。在我的笔记本电脑上，这就是我的`HOME`目录：

```py
$ echo $HOME
/Users/kyclark
```

在我的几乎所有系统上，我创建了一个`$HOME/.local`目录用于安装程序。大多数shell将波浪号（`~`）解释为`HOME`：

```py
$ mkdir ~/.local
```

按照惯例，以点开头的文件和目录通常被`ls`命令隐藏。你可以使用**`ls -a`**列出目录的*所有*内容。你可能会注意到许多其他*点文件*，它们被各种程序用来持久化选项和程序状态。我喜欢称其为`.local`，这样在我的目录列表中通常看不到它。

为软件安装在`HOME`中创建一个目录在生物信息学中编译程序从源代码是特别有用的。这种安装大多数情况下开始使用`configure`程序收集关于你的系统的信息，比如你的C编译器的位置等等。这个程序几乎总是有一个`--prefix`选项，我将设置为这个目录：

```py
$ ./configure --prefix=$HOME/.local
```

结果安装将把二进制编译文件放入`$HOME/.local/bin`。它还可能安装头文件和手册页以及其他支持数据到`$HOME/.local`中的其他目录。

无论你决定在何处安装本地程序，你都需要确保你的`PATH`已更新以在该目录中搜索，除了其他目录外。我倾向于使用`bash` shell，我`HOME`中的一个点文件是一个名为`.bashrc`（有时是`.bash_profile`或`.profile`）的文件。我可以添加这行来将我的自定义目录放在`PATH`的最前面：

```py
export PATH=$HOME/.local/bin:$PATH
```

如果你使用不同的shell，可能需要稍作调整。最近，macOS开始使用`zsh`（Z shell）作为默认shell，或者你的HPC系统可能使用另一种shell。它们都有`PATH`的概念，并且都允许你以某种方式进行自定义。在Windows上，你可以使用以下命令将目录追加到你的路径中：

```py
> $env:Path += ";~/.local/bin"
```

这里是我可以创建目录并复制程序的方法：

```py
$ mkdir -p ~/.local/bin
$ cp 01_dna/dna.py ~/.local/bin
```

现在我应该能够在Unix机器的任何位置执行**`dna.py`**：

```py
$ dna.py
usage: dna.py [-h] DNA
dna.py: error: the following arguments are required: DNA
```

Windows的shell如`cmd.exe`和PowerShell不会像Unix shell那样读取和执行shebang，因此在程序名之前，你需要包括命令**`python.exe`**或**`python3.exe`**：

```py
> python.exe C:\Users\kyclark\.local\bin\dna.py
usage: dna.py [-h] DNA
dna.py: error: the following arguments are required: DNA
```

确保`python.exe --version`显示你正在使用的是版本3而不是版本2。你可能需要安装最新版本的Python。我仅展示了使用`python.exe`的Windows命令，假设这意味着Python 3，但根据你的系统，你可能需要使用`python3.exe`。
