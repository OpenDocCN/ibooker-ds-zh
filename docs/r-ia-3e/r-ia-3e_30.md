# 附录 B. 自定义启动环境

程序员喜欢做的第一件事之一就是自定义他们的启动环境，以符合他们偏好的工作方式。自定义启动环境允许您设置 R 选项、指定工作目录、加载常用包、加载用户编写的函数、设置默认 CRAN 下载站点，并执行任何数量的维护任务。

您可以通过站点初始化文件 (Rprofile.site) 或目录初始化文件 (.Rprofile) 来自定义 R 环境。这些是包含在启动时执行的 R 代码的文本文件。

启动时，R 将从 R_HOME/etc 目录中源文件 Rprofile.site，其中 `R_HOME` 是一个环境变量。然后它将在当前工作目录中寻找要源文件的 .Rprofile 文件。如果 R 找不到此文件，它将在用户的主目录中寻找。您可以使用 `Sys.getenv("R`*_*`HOME")`、`Sys.getenv ("HOME")` 和 `getwd()` 分别识别 R_HOME、HOME 和当前工作目录的位置。

您可以在这些文件中放置两个特殊函数。`.First()` 函数在每个 R 会话开始时执行，而 `.Last()` 函数在每个会话结束时执行。Rprofile.site 文件的示例在列表 B.1 中展示。

列表 B.1 样本 .Rprofile

```
options(digits=4)                                                    ❶
options(show.signif.stars=FALSE)                                     ❶
options(scipen=999)                                                  ❶
options

options(prompt="> ")                                                 ❷
options(continue="  ")                                               ❷

options(repos = c(CRAN = "https://cran.rstudio.com/"))               ❸

.libPaths("C:/my_R_library")                                         ❹

.env <- new.env()                                                    ❺
.env$h <- utils::head                                                ❺
.env$t <- utils::tail                                                ❺
.env$s <- base::summary                                              ❺
.env$ht <- function(x){                                              ❺
  base::rbind(utils::head(x), utils::tail(x))                        ❺
}                                                                    ❺
.env$phelp <- function(pckg){                                        ❺
  utils::help(package = deparse(substitute(pckg)))                   ❺
}                                                                    ❺
attach(.env)                                                         ❺

.First <- function(){                                                ❻
  v <- R.Version()                                                   ❻
  msg1 <- paste0(v$version.string, ' -- ', ' "', v$nickname, '"')    ❻
  msg2 <- paste0("Platform: ", v$platform)                           ❻
  cat("\f")                                                          ❻
  cat(msg1, "\n", msg2, "\n\n", sep="")                              ❻
                                                                     ❻
  if(interactive()){                                                 ❻
    suppressMessages(require(tidyverse))                             ❻
  }                                                                  ❻
                                                                     ❻
}                                                                    ❻

.Last <- function(){                                                 ❼
  cat("\nGoodbye at ", date(), "\n")                                 ❼
}                                                                    ❼
```

❶ 设置常用选项

❷ 设置 R 交互式提示符

❸ 设置 CRAN 镜像默认值

❹ 设置本地库路径

❺ 创建快捷键

❻ 启动函数

❼ 会话结束函数

让我们看看这个 .Rprofile 实现了什么：

+   打印的输出被定制。有效数字的位数设置为 4（默认为 7），系数摘要表上不打印星号，并且抑制了科学记数法。

+   提示符的第一行设置为 ">", 而续行使用空格（而不是 "+" 符号）。

+   `install.packages()` 命令的默认 CRAN 镜像站点设置为 cran.rstudio.com。

+   为已安装的包定义了一个个人目录。设置 `.libPaths` 值允许您在 R 目录树之外为包创建本地库。这在升级期间保留包可能很有用。

+   为 `tail()`、`head()` 和 `summary()` 函数定义了一键快捷键。定义了新的函数 `ht()` 和 `phelp()`。`ht()` 结合了 head 和 tail 的输出。`phelp()` 列出包中的函数（带有链接的帮助）。

+   `.First()` 函数

    +   用较短的定制欢迎信息替换标准欢迎信息。在我的机器上，新的欢迎信息只是这样：

        R 版本 4.1.0 (2021-05-18)—"Camp Pontanezen"

        平台：x86_64-w64-mingw32/x64 (64 位)

    +   将 `tidyverse` 包加载到交互会话中。`tidyverse` 的启动信息被抑制。

+   `.Last()` 函数打印一个定制的告别信息。这也是执行任何清理活动的绝佳位置，包括存档命令历史、程序输出和数据文件。

警告！如果你在 .Rprofile 文件中定义函数或加载包，而不是在脚本中，它们将变得不太便携。你的代码现在依赖于启动文件才能正确运行。在其他缺少此文件的机器上，它们将无法运行。

如果你想给你的 .Rprofile 文件加速，可以看看 Jumping Rivers 的 `rprofile` 包（[`www.jumpingrivers.com/blog/customising-your-rprofile/`](https://www.jumpingrivers.com/blog/customising-your-rprofile/)）。还有其他方法可以自定义启动环境，包括使用命令行选项和环境变量。更多详情请参阅“R 简介”手册中的 `help(Startup)` 和附录 B（[`cran.r-project.org/doc/manuals/R-intro.pdf`](http://cran.r-project.org/doc/manuals/R-intro.pdf)）。
