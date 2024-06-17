# 附录 A. 使用 make 记录命令和创建工作流

`make` 程序是在 1976 年创建的，用于从源代码文件构建可执行程序。尽管最初是为了帮助使用 C 语言编程而开发的，但它不限于该语言，甚至不限于编译代码的任务。根据手册，人们可以使用它来描述任何需要在其他文件更改时自动更新一些文件的任务。`make` 程序已经远远超出了其作为构建工具的角色，成为一个工作流系统。

# Makefiles 就像食谱一样

当你运行 `make` 命令时，它会在当前工作目录中寻找名为 *Makefile*（或 *makefile*）的文件。该文件包含描述组合在一起创建某些输出的离散操作的食谱。想象一下柠檬蛋白饼的配方有需要按特定顺序和组合完成的步骤。例如，我需要分别制作馅饼皮、填料和蛋白霜，然后将它们组合在一起烘烤，才能享受美味。我可以用一种称为 *串图* 的东西来可视化这一过程，如 [图 A-1](#fig_a1.1) 所示。

![mpfb aa01](assets/mpfb_aa01.png)

###### 图 A-1\. 描述如何制作馅饼的串图，改编自 Brendan Fong 和 David Spivak 的《应用范畴论邀请（组合性的七个素描）》，剑桥大学出版社，2019 年。

如果你提前一天做好馅饼皮并冷藏，以及填料同样也适用，但是确实需要先把皮放入盘中，然后是填料，最后是蛋白霜。实际的食谱可能会引用其他地方的通用配方来制作馅饼皮和蛋白霜，并且只列出制作柠檬馅料和烘烤说明的步骤。

我可以编写一个 *Makefile* 来模拟这些想法。我将使用 shell 脚本来假装我正在将各种成分组合到像 *crust.txt* 和 *filling.txt* 这样的输出文件中。在 *app01_makefiles/pie* 目录中，我编写了一个 *combine.sh* 脚本，它期望一个文件名和一个要放入文件中的“成分”列表：

```py
$ cd app01_makefiles/pie/
$ ./combine.sh
usage: combine.sh FILE ingredients
```

我可以像这样假装制作馅饼皮：

```py
$ ./combine.sh crust.txt flour butter water
```

现在有一个名为 *crust.txt* 的文件，内容如下：

```py
$ cat crust.txt
Will combine flour butter water
```

在 *Makefile* 中，一个食谱通常但不必要创建一个输出文件。请注意，在这个例子中，`clean` 目标移除文件：

```py
all: crust.txt filling.txt meringue.txt ![1](assets/1.png)
	./combine.sh pie.txt crust.txt filling.txt meringue.txt ![2](assets/2.png)
	./cook.sh pie.txt 375 45

filling.txt:                            ![3](assets/3.png)
	./combine.sh filling.txt lemon butter sugar

meringue.txt:                           ![4](assets/4.png)
	./combine.sh meringue.txt eggwhites sugar

crust.txt:                              ![5](assets/5.png)
	./combine.sh crust.txt flour butter water

clean:                                  ![6](assets/6.png)
	rm -f crust.txt meringue.txt filling.txt pie.txt
```

[![1](assets/1.png)](#co_documenting_commands_and_creating_workflows_with_make_CO1-1)

定义了一个名为 `all` 的目标。当未指定目标时，将运行第一个目标。惯例是 `all` 目标将运行*所有*必要的目标以完成某个默认目标，比如构建软件。这里我想从组件文件中创建 *pie.txt* 文件并“烘烤”它。`all` 的名字并不重要，重要的是它被首先定义。目标名称后跟一个冒号，然后是运行此目标之前必须满足的任何依赖项。

[![2](assets/2.png)](#co_documenting_commands_and_creating_workflows_with_make_CO1-2)

`all`目标有两个要运行的命令。每个命令都用制表符缩进。

[![3](assets/3.png)](#co_documenting_commands_and_creating_workflows_with_make_CO1-3)

这是 `filling.txt` 目标。此目标的目标是创建名为 *filling.txt* 的文件。通常但不必要的是使用输出文件名作为目标名。此目标只有一个命令，即将填料的成分组合在一起。

[![4](assets/4.png)](#co_documenting_commands_and_creating_workflows_with_make_CO1-4)

这是 `meringue.txt` 目标，它结合了蛋清和糖。

[![5](assets/5.png)](#co_documenting_commands_and_creating_workflows_with_make_CO1-5)

这是 `crust.txt` 目标，它结合了面粉、黄油和水。

[![6](assets/6.png)](#co_documenting_commands_and_creating_workflows_with_make_CO1-6)

通常会有一个 `clean` 目标，用于删除在正常构建过程中创建的任何文件。

如前面的例子所示，目标有一个名称，后面跟着冒号。任何依赖动作都可以按你希望的顺序在冒号后列出。目标的动作必须用制表符缩进，如图 A-2 所示，你可以定义任意多个命令。

![mpfb aa02](assets/mpfb_aa02.png)

###### 图 A-2。一个 Makefile 目标以冒号结尾，可选择跟随依赖项；所有目标的动作必须用单个制表符缩进。

# 运行特定目标

*Makefile* 中的每个动作称为 *目标*、*规则* 或 *配方*。目标的顺序除了第一个目标作为默认目标外并不重要。目标可以像 Python 程序中的函数一样引用早先或晚于它定义的其他目标。

要运行特定目标，我运行**`make target`**来让 `make` 运行给定配方的命令：

```py
$ make filling.txt
./combine.sh filling.txt lemon butter sugar
```

现在有一个名为 *filling.txt* 的文件：

```py
$ cat filling.txt
Will combine lemon butter sugar
```

如果我尝试再次运行此目标，将告知无需执行任何操作，因为文件已存在：

```py
$ make filling.txt
make: 'filling.txt' is up to date.
```

`make` 存在的原因之一正是为了不必要地创建文件，除非某些底层源代码已更改。在构建软件或运行流水线的过程中，可能不必生成某些输出，除非输入已更改，例如修改了源代码。要强制 `make` 运行*filling.txt*目标，我可以删除该文件，或者运行**`make clean`**以删除已创建的任何文件：

```py
$ make clean
rm -f crust.txt meringue.txt filling.txt pie.txt
```

# 运行没有目标

如果你不带任何参数运行 `make` 命令，它会自动运行第一个目标。这是将 `all` 目标（或类似的目标）放在第一位的主要原因。但要小心，不要将像 `clean` 这样有破坏性的目标放在第一位，否则可能会意外运行它并删除有价值的数据。

当我运行带有前述 *Makefile* 的 `make` 时，输出如下：

```py
$ make                                                  ![1](assets/1.png)
./combine.sh crust.txt flour butter water               ![2](assets/2.png)
./combine.sh filling.txt lemon butter sugar             ![3](assets/3.png)
./combine.sh meringue.txt eggwhites sugar               ![4](assets/4.png)
./combine.sh pie.txt crust.txt filling.txt meringue.txt ![5](assets/5.png)
./cook.sh pie.txt 375 45                                ![6](assets/6.png)
Will cook "pie.txt" at 375 degrees for 45 minutes.
```

[![1](assets/1.png)](#co_documenting_commands_and_creating_workflows_with_make_CO2-1)

我不带任何参数运行 `make`。它会在当前工作目录中的名为 *Makefile* 的文件中查找第一个目标。

[![2](assets/2.png)](#co_documenting_commands_and_creating_workflows_with_make_CO2-2)

*crust.txt* 配方首先运行。因为我没有指定目标，`make` 运行了第一个定义的 `all` 目标，而这个目标列出了*crust.txt*作为第一个依赖项。

[![3](assets/3.png)](#co_documenting_commands_and_creating_workflows_with_make_CO2-3)

接下来运行 *filling.txt* 目标。

[![4](assets/4.png)](#co_documenting_commands_and_creating_workflows_with_make_CO2-4)

然后是*meringue.txt*。

[![5](assets/5.png)](#co_documenting_commands_and_creating_workflows_with_make_CO2-5)

接下来我组装*pie.txt*。

[![6](assets/6.png)](#co_documenting_commands_and_creating_workflows_with_make_CO2-6)

然后我在 375 度下烤 pie 45 分钟。

如果我再次运行 `make`，我会看到跳过产生 *crust.txt*、*filling.txt* 和 *meringue.txt* 文件的中间步骤，因为它们已经存在：

```py
$ make
./combine.sh pie.txt crust.txt filling.txt meringue.txt
./cook.sh pie.txt 375 45
Will cook "pie.txt" at 375 degrees for 45 minutes.
```

如果我想强制重新创建它们，可以运行`make clean && make`，其中 `&&` 是逻辑*与*，仅在第一个命令成功运行后才运行第二个命令：

```py
$ make clean && make
rm -f crust.txt meringue.txt filling.txt pie.txt
./combine.sh crust.txt flour butter water
./combine.sh filling.txt lemon butter sugar
./combine.sh meringue.txt eggwhites sugar
./combine.sh pie.txt crust.txt filling.txt meringue.txt
./cook.sh pie.txt 375 45
Will cook "pie.txt" at 375 degrees for 45 minutes.
```

# Makefile 创建 DAGs

每个目标都可以指定其他目标作为必须先完成的前提条件或依赖关系。这些操作创建了一个图结构，有一个起点和穿过目标的路径，最终创建一些输出文件。任何目标描述的路径应该是一个*有向*（从开始到结束）*无环*（没有循环或无限循环）*图*，或称为 DAG，如图 A-3 所示。

![mpfb aa03](assets/mpfb_aa03.png)

###### 图 A-3\. 这些目标可以组合在一起，描述一个有向无环图的操作，以产生某个结果

许多分析管道就是这样——一个输入的图，如 FASTA 序列文件，以及一些转换（修剪，过滤，比较），最终输出 BLAST 命中，基因预测或功能注释等结果。你会惊讶地发现 `make` 可以被滥用来记录你的工作甚至创建完全功能的分析管道。

# 使用 make 编译一个 C 程序

我相信至少在生活中使用一次 `make` 来理解它存在的原因是有帮助的。我会花点时间编写并编译一个用 C 语言编写的“Hello, World”示例。在 *app01_makefiles/c-hello* 目录中，你会找到一个简单的 C 程序，它将打印“Hello, World！”下面是 *hello.c* 的源代码：

```py
#include <stdio.h>            ![1](assets/1.png)
int main() {                  ![2](assets/2.png)
   printf("Hello, World!\n"); ![3](assets/3.png)
   return 0;                  ![4](assets/4.png)
}                             ![5](assets/5.png)
```

[![1](assets/1.png)](#co_documenting_commands_and_creating_workflows_with_make_CO3-1)

就像在 `bash` 中一样，`#` 字符引入了 C 语言中的注释，但这是一个特殊的注释，允许使用外部代码模块。在这里，我想使用 `printf`（打印格式）函数，所以我需要 `include` 标准输入/输出（input/output）模块，称为 `stdio`。我只需要包含“头”文件 `stdio.h`，以获取该模块中的函数定义。这是一个标准模块，C 编译器将在各种位置查找任何包含的文件以找到它。有时由于找不到某个头文件，你可能无法从源代码编译 C（或 C++）程序。例如，gzip 库通常用于解压/压缩数据，但它并不总是以其他程序可以 `include` 的库形式安装。因此，你需要下载并安装 `libgz` 程序，确保将头文件安装到正确的 `include` 目录中。请注意，像 `apt-get` 和 `yum` 这样的软件包管理器通常有 `-dev` 或 `-devel` 包，你必须安装这些包来获取这些头文件；也就是说，你需要安装 `libgz` 和 `libgz-dev` 或类似的包。

[![2](assets/2.png)](#co_documenting_commands_and_creating_workflows_with_make_CO3-2)

这是在 C 中函数声明的开始。函数名（`main`）前面是它的返回类型（`int`）。函数的参数列在名称后的括号内。在这种情况下没有参数，所以括号为空。开头的大括号（`{`）表示函数代码的开始。请注意，C 将自动执行 `main()` 函数，并且每个 C 程序必须有一个 `main()` 函数，这是程序的起点。

[![3](assets/3.png)](#co_documenting_commands_and_creating_workflows_with_make_CO3-3)

`printf()` 函数将给定的字符串打印到命令行。此函数定义在 `stdio` 库中，这就是为什么我需要 `#include` 上面的头文件。

[![4](assets/4.png)](#co_documenting_commands_and_creating_workflows_with_make_CO3-4)

`return`将退出函数并返回值`0`。因为这是`main()`函数的返回值，这将是整个程序的退出值。值`0`表示程序正常运行—即“零错误”。任何非零值都表示失败。

[![5](assets/5.png)](#co_documenting_commands_and_creating_workflows_with_make_CO3-5)

结束花括号（`}`）是第2行的配对符号，并标记了`main()`函数的结束。

要将其转换为可执行程序，您需要在计算机上安装C编译器。例如，我可以使用GNU C编译器`gcc`：

```py
$ gcc hello.c
```

这将创建一个名为*a.out*的文件，这是一个可执行文件。在我的Macintosh上，`file`会报告如下：

```py
$ file a.out
a.out: Mach-O 64-bit executable arm64
```

然后我可以执行它：

```py
$ ./a.out
Hello, World!
```

我不喜欢*a.out*这个名称，所以我可以使用`-o`选项来命名输出文件为*hello*：

```py
$ gcc -o hello hello.c
```

运行生成的*hello*可执行文件。您应该看到相同的输出。

而不是每次修改*hello.c*时都要输入`gcc -o hello hello.c`，我可以将其放入*Makefile*中：

```py
hello:
	gcc -o hello hello.c
```

现在我可以运行**`make hello`**或者如果这是第一个目标，只需运行**`make`**：

```py
$ make
gcc -o hello hello.c
```

如果我再次运行**`make`**，因为*hello.c*文件没有改变，所以什么都不会发生：

```py
$ make
make: 'hello' is up to date.
```

如果我将*hello.c*代码更改为打印“Hola”而不是“Hello”，然后再次运行`make`会发生什么？

```py
$ make
make: 'hello' is up to date.
```

我可以通过使用`-B`选项强制`make`运行目标：

```py
$ make -B
gcc -o hello hello.c
```

现在新程序已经编译完成：

```py
$ ./hello
Hola, World!
```

这只是一个简单的示例，您可能想知道这如何节省时间。在C或任何语言的实际项目中，可能会有多个*.c*文件，以及描述它们函数的头文件（*.h*文件），以便其他*.c*文件可以使用它们。C编译器需要将每个*.c*文件转换为*.o*（*.out*）文件，然后将它们链接在一起成为单个可执行文件。想象一下，您有数十个*.c*文件，并且更改一个文件中的一行代码。您想要手动输入数十个命令重新编译和链接所有代码吗？当然不。您会构建一个工具来自动化这些操作。

我可以向*Makefile*添加不生成新文件的目标。通常会有一个`clean`目标，用于清理不再需要的文件和目录。这里我可以创建一个`clean`目标来删除*hello*可执行文件：

```py
clean:
	rm -f hello
```

如果我希望在运行`hello`目标之前始终删除可执行文件，我可以将其作为依赖项添加：

```py
hello: clean
	gcc -o hello hello.c
```

对于`make`来说，很好地记录这是一个*phony*目标，因为目标的结果不是一个新创建的文件。我使用`.PHONY:`目标并列出所有虚假目标。现在完整的*Makefile*如下：

```py
$ cat Makefile
.PHONY: clean

hello: clean
	gcc -o hello hello.c

clean:
	rm -f hello
```

如果您在*c-hello*目录中运行**`make`**与前述的*Makefile*，您应该看到这样的输出：

```py
$ make
rm -f hello
gcc -o hello hello.c
```

现在您的目录中应该有一个*hello*可执行文件可以运行：

```py
$ ./hello
Hello, World!
```

注意，`clean`目标可以在甚至在目标本身被提到之前就列为`hello`目标的依赖项。`make`会读取整个文件，然后使用依赖关系来解析图形。如果你将`foo`作为`hello`的附加依赖项并再次运行**`make`**，你会看到这样的情况：

```py
$ make
make: *** No rule to make target 'foo', needed by 'hello'.  Stop.
```

*Makefile*允许我编写独立的动作组，这些动作按它们的依赖顺序排列。它们就像高级语言中的*函数*。我基本上写了一个输出为另一个程序的程序。

我建议你运行**`cat hello`**来查看*hello*文件的内容。它主要是一些看起来像乱码的二进制信息，但你可能也能看到一些明文。你也可以使用**`strings hello`**来提取文本字符串。

# 使用make作为快捷方式

让我们看看如何滥用*Makefile*来为命令创建快捷方式。在*app01_makefiles/hello*目录中，你会找到以下*Makefile*：

```py
$ cat Makefile
.PHONY: hello            ![1](assets/1.png)

hello:                   ![2](assets/2.png)
	echo "Hello, World!" ![3](assets/3.png)
```

[![1](assets/1.png)](#co_documenting_commands_and_creating_workflows_with_make_CO4-1)

由于`hello`目标不生成文件，我将其列为伪目标。

[![2](assets/2.png)](#co_documenting_commands_and_creating_workflows_with_make_CO4-2)

这是`hello`目标。目标的名称应该只由字母和数字组成，在其前面不应有空格，并在冒号（`:`）之后。

[![3](assets/3.png)](#co_documenting_commands_and_creating_workflows_with_make_CO4-3)

在以制表符缩进的行上列出了`hello`目标要运行的命令（们）。

我可以用**`make`**来执行这个：

```py
$ make
echo "Hello, World!"
Hello, World!
```

我经常使用*Makefile*来记住如何使用各种参数调用命令。也就是说，我可能会编写一个分析流水线，然后记录如何在各种数据集上运行程序及其所有参数。通过这种方式，我可以立即通过运行目标来复现我的工作。

# 定义变量

这里是我写的一个*Makefile*的示例，用来记录我如何使用Centrifuge程序对短读取进行分类分配：

```py
INDEX_DIR = /data/centrifuge-indexes         ![1](assets/1.png)

clean_paired:
    rm -rf $(HOME)/work/data/centrifuge/paired-out

paired: clean_paired                         ![2](assets/2.png)
    ./run_centrifuge.py \                    ![3](assets/3.png)
    -q $(HOME)/work/data/centrifuge/paired \ ![4](assets/4.png)
    -I $(INDEX_DIR) \                        ![5](assets/5.png)
    -i 'p_compressed+h+v' \
    -x "9606, 32630" \
    -o $(HOME)/work/data/centrifuge/paired-out \
    -T "C/Fe Cycling"
```

[![1](assets/1.png)](#co_documenting_commands_and_creating_workflows_with_make_CO5-1)

这里我定义了变量`INDEX_DIR`并赋了一个值。注意`=`两边必须有空格。我个人偏好使用全大写来命名我的变量，但这是我的个人喜好。

[![2](assets/2.png)](#co_documenting_commands_and_creating_workflows_with_make_CO5-2)

在运行此目标之前，请先运行`clean_paired`目标。这确保没有上一次运行留下的输出。

[![3](assets/3.png)](#co_documenting_commands_and_creating_workflows_with_make_CO5-3)

由于这个操作很长，所以我使用反斜杠（`\`）像在命令行上一样表示命令延续到下一行。

[![4](assets/4.png)](#co_documenting_commands_and_creating_workflows_with_make_CO5-4)

要使`make`*解除引用*或使用`$HOME`环境变量的值，请使用语法`$(HOME)`。

[![5](assets/5.png)](#co_documenting_commands_and_creating_workflows_with_make_CO5-5)

`$(INDEX_DIR)`是指顶部定义的变量。

# 编写工作流程

在*app01_makefiles/yeast*目录中有一个示例，展示了如何将工作流程写成`make`目标。目标是下载酵母基因组并将各种基因类型（如“可疑”，“未分类”，“已验证”等）进行特征化。这通过一系列命令行工具（如`wget`，`grep`和`awk`）以及名为*download.sh*的自定义Shell脚本组合在一起，并按顺序由`make`运行：

```py
.PHONY: all fasta features test clean

FEATURES = http://downloads.yeastgenome.org/curation/$\
    chromosomal_feature/
    SGD_features.tab

all: fasta genome chr-count chr-size features gene-count verified-genes \
     uncharacterized-genes gene-types terminated-genes test

clean:
	find . \( -name \*gene\* -o -name chr-\* \) -exec rm {} \;
	rm -rf fasta SGD_features.tab

fasta:
	./download.sh

genome: fasta
	(cd fasta && cat *.fsa > genome.fa)

chr-count: genome
	grep -e '^>' "fasta/genome.fa" | grep 'chromosome' | wc -l > chr-count

chr-size: genome
	grep -ve '^>' "fasta/genome.fa" | wc -c > chr-size

features:
	wget -nc $(FEATURES)

gene-count: features
	cut -f 2 SGD_features.tab | grep ORF | wc -l > gene-count

verified-genes: features
	awk -F"\t" '$$3 == "Verified" {print}' SGD_features.tab | \
		wc -l > verified-genes

uncharacterized-genes: features
	awk -F"\t" '$$2 == "ORF" && $$3 == "Uncharacterized" {print $$2}' \
		SGD_features.tab | wc -l > uncharacterized-genes

gene-types: features
	awk -F"\t" '{print $$3}' SGD_features.tab | sort | uniq -c > gene-types

terminated-genes:
	grep -o '/G=[^ ]*' palinsreg.txt | cut -d = -f 2 | \
		sort -u > terminated-genes

test:
	pytest -xv ./test.py
```

我不打算评论所有命令。我主要想演示我可以滥用*Makefile*创建工作流程的程度。我不仅记录了所有步骤，而且使用**`make`**命令就可以*运行*它们。如果不使用`make`，我将不得不编写一个Shell脚本来完成这个任务，或者更可能切换到像Python这样的更强大的语言。用任一语言编写的结果程序可能会更长，更多错误，更难理解。有时候，你只需要一个*Makefile*和一些Shell命令。

# 其他工作流程管理器

当您遇到`make`的限制时，您可以选择切换到工作流程管理器。有许多选择。例如：

+   Snakemake通过Python扩展了`make`的基本概念。

+   通用工作流程语言（CWL）在配置文件（YAML格式）中定义工作流程和参数，您可以使用`cwltool`或`cwl-runner`（均采用Python实现）等工具执行另一个配置文件描述的工作流程。

+   工作流描述语言（WDL）采用类似的方法来描述工作流和参数，并且可以使用Cromwell引擎运行。

+   Pegasus允许您使用Python代码来描述一个工作流，然后将其写入XML文件，这是引擎的输入，用于运行您的代码。

+   Nextflow类似，您可以使用名为Groovy（Java的子集）的完整编程语言编写可以由Nextflow引擎运行的工作流。

所有这些系统都遵循与`make`相同的基本思想，因此理解`make`的工作原理以及如何编写工作流程的各个部分以及它们之间的交互是您可能创建的任何更大分析工作流程的基础。

# 进一步阅读

这里有一些其他资源可以帮助您学习`make`：

+   [GNU Make手册](https://oreil.ly/D9daZ)

+   *GNU Make手册* 由John Graham-Cumming（No Starch Press，2015）

+   [*使用GNU Make管理项目*](https://oreil.ly/D8Oyk) 由Robert Mecklenburg（O'Reilly，2004）
