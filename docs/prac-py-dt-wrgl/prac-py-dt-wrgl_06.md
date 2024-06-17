# 第六章。评估数据质量

在过去的两章中，我们集中精力识别和访问不同位置（从电子表格到网站）的不同格式的数据。但实际上，获取我们手头（可能）有趣的数据只是个开始。下一步是进行彻底的质量评估，以了解我们所拥有的数据是否有用、可挽救或只是垃圾。

正如你可能从阅读[第三章](ch03.html#chapter3)中获得的，打造高质量数据是一项复杂且耗时的工作。这个过程大致上包括研究、实验和顽强的毅力。最重要的是，致力于数据质量意味着你必须愿意投入大量的时间和精力，并且*仍然愿意放弃一切重新开始*，如果尽管你的最大努力，你所拥有的数据仍无法达到要求。

实际上，归根结底，这最后一个标准可能是使得与数据一起进行真正高质量、有意义的工作变得真正困难的原因。技术技能，正如我希望你已经发现的那样，需要一些努力才能掌握，但只要有足够的练习，它们仍然是可以实现的。研究技能稍微难以记录和传达，但通过本书中的例子，你将有助于发展许多与信息发现和整理相关的技能，这些技能对评估和改善数据质量至关重要。

当谈到接受这样一个事实时，即经过数十个小时的工作后，你可能需要“放弃”一个数据集，因为它的缺陷太深或太广泛，我唯一能提供的建议就是你要试着记住：*学习关于世界有意义的事情是一个“长期游戏”*。这就是为什么我总是建议那些有兴趣学习如何进行数据整理和分析的人，首先要找到一个对他们来说真正有趣和/或重要的关于世界的问题。要做好这项工作，你必须比“完成”更关心*做对*。但如果你真的在数据整理的*过程*中真正重视你所学到的东西，不论是因为学习了新的Python或数据整理策略，还是因为与新的专家接触或发现了新的信息资源，这也会有所帮助。事实上，如果你真的关心你正在探索的主题，做好数据工作的努力绝不会被浪费。只是它可能会引导你朝着一个你没有预料到的方向发展。

经常情况下，您会发现虽然一个数据集不能回答您最初的问题，但它仍然可以揭示其中某些关键方面。其他时候，您可能会发现关于您探索的主题**没有**数据，并且您可以利用这个事实来寻求帮助——发现**不存在**的数据可能会促使您和其他人彻底改变焦点。而且，由于“完美”的数据集永远不会存在，有时候您可能会选择*非常小心地*分享您知道有缺陷的数据，因为即使它提供的部分见解也对公众利益有重要作用。在每种情况下重要的是，您愿意对自己所做的选择负责——因为无论“原始”数据是什么或来自何处，经过清洗、增强、转换和/或分析的数据集仍然是*您的*。

如果您感到有些不知所措——嗯，这并不完全是偶然。我们生活在一个时刻，使用强大的数字工具很容易而不考虑它们的现实世界后果，以及建造“先进”技术的人大多数情况下是在按照他们自己的利益编写算法规则的时刻。^([1](ch06.html#idm45143406437120))最终，数据既可以是信息的机制*也*可以是操纵的机制，既可以解释*也*可以利用。确保您的工作落在这些界限的哪一侧，最终取决于您自己。

但实际上，实现数据质量意味着什么呢？为了了解这一点，我们将在本章的剩余部分中评估现实世界数据，从数据*完整性*和数据*适配性*的角度来看，这些概念在[第 3 章](ch03.html#chapter3)中已经介绍过。我们将使用的数据集是来自美国薪资保护计划（PPP）的贷款数据的单个实例，该计划包含有关在 COVID-19 大流行期间向小企业发放的数百万笔贷款的信息。正如我们在本章剩余部分中将首次亲身经历的那样，PPP 数据体现了许多“发现”数据中常见的挑战：不明确的术语和从一个数据版本到下一个数据版本的不明确变化留下了数据*适配性*问题的空间，这些问题需要额外的研究来解决。数据*完整性*问题更为直接（尽管解决起来可能不一定更快）——比如确认数据集中某个特定银行的拼写是否始终一致，或者我们的数据文件是否包含应有的值和时间范围。尽管我们将会应对大多数这些挑战，但最终我们的见解将是高度自信的，*而非*无可辩驳的。与所有数据工作一样，它们将是知情决策、逻辑推理和大量数据处理的累积结果。为了完成这些处理工作，我们最依赖的 Python 库是*pandas*，它提供了一套流行且强大的工具，用于处理表格类型的数据。让我们开始吧！

# **大流行和 PPP**

在2020年春季，美国政府宣布了一个贷款计划，旨在帮助稳定因COVID-19大流行导致的美国经济下滑。该计划拥有近1万亿美元的指定资金，^([2](ch06.html#idm45143406492528)) PPP的目标显然是帮助小企业支付租金，并继续支付员工的工资，尽管有时会面临强制关闭和其他限制。尽管第一笔资金发放后失业率出现下降，但联邦政府的一些部分似乎决心不透明资金去向的要求。^([3](ch06.html#idm45143406490368))

PPP贷款是否帮助挽救了美国的小企业？随着时间的推移，我们可能会认为这是一个足够直接的问题来回答，但我们是来亲自找出答案的。为此，当然我们会从数据开始进行系统评估，首先是对其整体质量的系统评估，在这一过程中，我们依次审查我们每个特征的PPP贷款数据的数据*完整性*和数据*适配性*。同样重要的是，我们将*仔细记录*这个过程的每一部分，以便我们有记录我们所做的事情、我们做出的选择以及背后的推理的记录。这个数据日记将是我们未来的重要资源，特别是在我们需要解释或重现我们的任何工作时。虽然您可以使用任何您喜欢的形式和格式，我喜欢保持我的格式简单，并让我的数据日记与我的代码一起，因此我将记录我的工作在一个Markdown文件中，我可以轻松地备份和在GitHub上阅读。^([4](ch06.html#idm45143406483424)) 由于这个文件实际上是我所做的一切的一个正在进行的清单，我打算将其称为*ppp_process_log.md*。

# 评估数据完整性

您是否应该通过评估数据完整性或数据适配性开始数据整理过程？毫不奇怪：两者兼顾。正如在[“数据整理是什么？”](ch01.html#describing_data_wrangling)中所讨论的，除非您有某种问题要解决，并且有一定的感觉某个特定数据集可以帮助您回答它——换句话说，直到您对数据整理过程的方向有了一些想法，并且您的数据“适合”这个过程。同时，要完全评估数据集的适配性通常是困难的，直到其完整性被探索过。例如，如果数据中存在间隙，是否可以通过某种方式填补？是否可以找到缺失的元数据？如果可以，那么我们可能能够解决这些完整性问题，并以更完整的信息回到适配性的评估中。如果我们然后发现可以通过增强数据来改善其适配性（我们将在[第7章](ch07.html#chapter7)中详细探讨），当然，这将启动*另一轮*数据完整性评估。然后循环重新开始。

如果你担心这会导致数据整理的无限循环，你并非完全错误；所有好的问题都会引发其他问题，因此永远有更多东西可以学习。尽管如此，我们可以投入到数据整理工作中的时间、精力和其他资源*并非*无限，这就是为什么我们必须做出知情的决策并加以记录。彻底评估我们数据的质量将有助于我们做出这些决策。通过系统地检查数据的完整性和适应性，确保不仅最终获得高质量的数据，而且制定和记录决策，可以用来描述（甚至在必要时捍卫）所产生的任何见解。这并不意味着每个人都会同意你的结论。然而，它确实有助于确保你可以就此进行有意义的讨论——这才是真正推动知识进步的。

因此，让我们毫不拖延地开始我们的数据完整性评估吧！

###### 注意

本章使用的许多数据已从互联网上删除和/或替换为其他文件——这并不罕见。尽管如此，我故意保留了本章内容的大部分原貌，因为它们反映了在尝试进行数据整理工作时面临的真实和典型挑战，尽管这些数据已发生变化。

它还说明了数据在数字世界中如何迅速且几乎无法追踪地演变和变化。这是需要记住的一点。同时，你可以在这个[Google Drive 文件夹](https://drive.google.com/file/d/1EtUB0nK9aQeWWWGUOiayO9Oe-avsKvXH/view?usp=sharing)中找到本章引用的所有数据集。

## 是否具有已知的出处？

在撰写本文时，“最新PPP贷款数据”短语的第一个搜索结果是美国财政部网站上的一个页面，链接到2020年8月的数据^([5](ch06.html#idm45143406687968))。第二个结果链接到更近期的数据，来自负责实际管理资金的小型企业管理局（SBA）^([6](ch06.html#idm45143406684480))。

尽管这两个网站都是与PPP相关的政府机构的合法网站，但对我们来说，与SBA发布的数据主要合作更有意义。尽管如此，在我首次寻找这些数据时，它*并不*是我首先找到的东西。

我非常想强调这一点，因为财政部显然是我们正在寻找的数据的一个合理和声誉良好的来源，但它仍然不是*最佳*的选择。这就是为什么评估你的数据不仅来自*何处*，还要考虑你*如何*找到它的重要性。

## 是否及时？

如果我们想要利用数据来了解当前世界的状态，首先需要确定我们的数据来自*何时*，并确认它是目前可获得的最新数据。

如果这看起来应该很简单，请再想一想——许多网站不会自动为每篇文章注明日期，因此甚至确定某事何时上线通常是一项挑战。或者，一个标明其内容日期的网站可能会在进行任何变更（甚至是微小的变更）时更改日期；一些网站可能会定期更新“发布”日期，以试图操控搜索引擎算法，这些算法偏爱更新的内容。

换句话说，确定对于你特定的数据整理问题来说，*实际上*最新、最相关的数据可能需要一些挖掘。唯一确定的方法将是尝试几个不同的搜索词组，点击几组结果，并可能向专家寻求建议。在这个过程中，你很可能会找到足够的参考资料来确认最近可用的数据。

## 它是完整的吗？

到目前为止，我们知道PPP已经进行了多次数据发布。虽然我们可以相当有信心地说我们已经成功地找到了最*近*的数据，但接下来的问题是：这是*所有*的数据吗？

由于我们主要关注那些获得较大贷款的企业，我们只需要担心检查一个文件：*public_150k_plus.csv*。^([7](ch06.html#idm45143406648768))但是我们如何知道这是否包括到目前为止的所有程序阶段，或者只是自2020年8月首次数据发布以来的贷款？由于我们可以访问这两组数据，^([8](ch06.html#idm45143406646448))我们可以采用几种策略：

1.  查找我们“最近”数据文件中的最早日期，并确认它们是*在*2020年8月8日之前。

1.  比较两个数据集的文件大小和/或行数，以确认更新的文件比旧文件更大。

1.  比较它们包含的数据以确认较早文件中的所有记录是否已经存在于较新的文件中。

此时，你可能会想，“等等，确认最早的日期不就足够了吗？为什么我们要做其他两个？这两者似乎都更加困难。”当然，在*理论*上，只确认我们从早期阶段获得了数据应该就足够了。但事实上，这只是一个相当粗略的检查。显然，我们不能通过尝试自己收集更全面的数据来确认联邦政府发布的数据是否完整。另一方面（正如我们马上会看到的），政府和大型组织（包括——也许尤其是——银行）的数据收集过程远非完美。^([9](ch06.html#idm45143406638368))由于我们将使用这些数据来得出结论，如果我们拥有这些数据，我认为彻底地检查是值得的。

令人高兴的是，进行我们的第一个“完整性”检查非常简单：只需在文本编辑器中打开数据，我们可以看到第一个条目包含一个“DateApproved”值为*05/01/2020*，这表明最近的数据集*确实*包含从2020年春季发放的PPP贷款第一轮数据，如[图 6-1](#ppp_loan_data_recent)所示。

![最近PPP贷款数据的快速文本编辑器视图。](assets/ppdw_0601.png)

###### 图 6-1\. 最近PPP贷款数据的快速文本编辑器视图

太容易了！如果我们想再进一步，我们可以随时编写一个快速脚本来查找最近数据中最早和最近的`LoanStatus`日期。为了避免混淆，我已将更新的文件重命名为*public_150k_plus_recent.csv*。目前，^([10](ch06.html#idm45143406630688)) 根据从[*https://home.treasury.gov/policy-issues/cares-act/assistance-for-small-businesses/sba-paycheck-protection-program-loan-level-data*](https://home.treasury.gov/policy-issues/cares-act/assistance-for-small-businesses/sba-paycheck-protection-program-loan-level-data)链接的数据，会导致SBA Box账户上的*不同*文件夹，显示的上传日期为[2020年8月14日](https://sba.app.box.com/s/ox4mwmvli4ndbp14401xr411m8sefx3i)。我们需要下载整个文件夹，*150k plus 0808.zip*，但我们可以提取CSV并将其重命名为*public_150k_plus_080820.csv*。

为了帮助我们进行这个过程，通常情况下，我们会寻找一个库来简化操作 —— 这次我们选择Pandas，这是一个用于操作表格类型数据的广为人知的Python库。虽然我们不会在这里详细讨论*pandas*库（已经有一本由Pandas的创作者Wes McKinney所写的优秀的O'Reilly书籍，*Python for Data Analysis*！），但我们肯定会在今后进行数据质量检查时充分利用它的许多有用功能。

首先，我们需要通过以下命令行安装这个库：

```py
pip install pandas
```

现在，让我们使用它来提取最近的PPP贷款数据，并查看该文件中的最早和最近日期，如[示例 6-1](#ppp_date_range)所示。

##### 示例 6-1\. ppp_date_range.py

```py
# quick script for finding the earliest and latest loan dates in the PPP loan
# data

# importing the `pandas` library
import pandas as pd ![1](assets/1.png)

# read the recent data into a pandas DataFrame using its `read_csv()` method
ppp_data = pd.read_csv('public_150k_plus_recent.csv')

# convert the values in the `DateApproved` column to *actual* dates
ppp_data['DateApproved'] = pd.to_datetime(ppp_data['DateApproved'],
                                          format='%m/%d/%Y') ![2](assets/2.png)

# print out the `min()` and `max()` values in the `DateApproved` column
print(ppp_data['DateApproved'].min())
print(ppp_data['DateApproved'].max())
```

[![1](assets/1.png)](#co_assessing_data_quality_CO1-1)

在这里使用`as`关键字可以为库创建一个昵称，以便我们稍后在代码中引用时使用更少的字符。

[![2](assets/2.png)](#co_assessing_data_quality_CO1-2)

要找到最早和最近的日期，我们首先需要将数据集中的（字符串）数值转换为*实际*的`Date`数据类型。在这里，我们将使用Pandas的`to_datetime()`函数，并提供（1）[要转换的列](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.to_datetime.html)，以及（2）[日期的格式](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior)，即它们*当前*在我们的数据集中的外观。

正如我们从以下输出中看到的那样，这个数据集中最早的贷款是在2020年4月3日进行的，而最近的贷款是在2021年1月31日进行的：

```py
2020-04-03 00:00:00
2021-01-31 00:00:00
```

请注意，虽然我们不*绝对*需要将`format`参数传递给 Pandas 的`to_datetime()`函数，但这样做总是一个好主意；如果我们不提供此信息，那么 Pandas 必须尝试“猜测”它所查看的日期格式，这可能会使实际处理需要很长时间。 在这里，它只能为我们节省约一秒的处理时间 - 但对于较大的数据集（或多个数据集），这些时间可能会迅速累积！

现在让我们比较最近数据的文件大小和2020年8月发布的文件的大小。 只需在一个finder窗口中查看*public_150k_plus_recent.csv*和*public_150k_plus_080820.csv*文件，我们就可以看到最新数据的文件大小比先前的文件要*大得多*：8月份的数据约为124 MB，而最近的数据则为几百兆字节。 到目前为止还不错。

借鉴我们在[示例 4-7](ch04.html#fixed_width_parsing)和[示例 5-10](ch05.html#downloading_the_data)中使用的技术，让我们编写一个快速脚本来确定每个文件中有多少行数据，如[示例 6-2](#ppp_numrows)所示。

##### 示例 6-2\. ppp_numrows.py

```py
# quick script to print out the number of rows in each of our PPP loan data files
# this is a pretty basic task, so no need to import extra libraries!

# open the August PPP data in "read" mode
august_data = open("public_150k_plus_080820.csv","r")

# use `readlines()` to convert the lines in the data file into a list
print("August file has "+str(len(august_data.readlines()))+" rows.") ![1](assets/1.png)

# ditto for the recent PPP data
recent_data = open("public_150k_plus_recent.csv","r")

# once again, print the number of lines
print("Recent file has "+str(len(recent_data.readlines()))+" rows.")
```

[![1](assets/1.png)](#co_assessing_data_quality_CO2-1)

使用`readlines()`方法将数据文件的行放入列表后，我们可以使用内置的`len()`方法来确定有多少行。 要打印结果，我们必须首先使用内置的`str()`函数将该数字转换为字符串，否则 Python 会对我们大喊大叫。

运行此脚本确认，2020年8月的文件包含662,516行，而最近版本（2021年2月1日）包含766,500行。

最后，让我们比较两个文件的内容，以确认较早文件中的所有内容是否出现在较新文件中。 这无疑将是一个多步骤的过程，但让我们从在文本编辑器中打开8月份数据文件开始，如[图 6-2](#ppp_loan_data_august)所示。

![2020年8月PPP贷款数据的快速文本编辑器视图。](assets/ppdw_0602.png)

###### 图 6-2\. 2020年8月PPP贷款数据的快速文本编辑器视图

立即就能看出存在一些……差异，这将使得这个特定的检查比我们预期的更加复杂。 首先，很明显，最近的文件中有*更多*的数据列，这意味着我们需要制定一种策略来将较早的数据集中的记录与最近的数据集进行匹配。

首先，我们需要了解两个文件之间似乎重叠的数据列。 我们将通过创建和比较两个小的*样本*CSV文件来做到这一点，其中包含每个数据集的前几行数据。

接下来，我们将编写一个快速脚本，将我们的源文件中的每个文件转换为 `DataFrame`——这是一种特殊的 Pandas 数据类型，用于表格类型数据——然后将前几行写入一个单独的 CSV 文件，如[示例 6-3](#ppp_data_samples)所示。

##### 示例 6-3\. ppp_data_samples.py

```py
# quick script for creating new CSVs that each contain the first few rows of
# our larger data files

# importing the `pandas` library
import pandas as pd

# read the august data into a pandas DataFrame using its `read_csv()` method
august_ppp_data = pd.read_csv('public_150k_plus_080820.csv')

# the `head()` method returns the DataFrame's column headers
# along with the first 5 rows of data
august_sample = august_ppp_data.head()

# write those first few rows to a CSV called `august_sample.csv`
# using the pandas `to_csv()` method
august_sample.to_csv('august_sample.csv', index=False) ![1](assets/1.png)

# read the recent data into a pandas DataFrame using its `read_csv()` method
recent_ppp_data = pd.read_csv('public_150k_plus_recent.csv')

# the `head()` method returns the DataFrame's column headers
# along with the first 5 rows of data
recent_sample = recent_ppp_data.head()

# write those first few rows to a CSV called `recent_sample.csv`
recent_sample.to_csv('recent_sample.csv', index=False)
```

[![1](assets/1.png)](#co_assessing_data_quality_CO3-1)

*pandas* 的 `to_csv()` 方法封装了我们之前在普通的 Python 中所做的几个步骤：打开一个新的可写文件并将数据保存到其中。由于 Pandas 在创建每个 DataFrame 时都会添加一个索引列（实质上包含行号），因此我们需要包含第二个“成分” `index=False`，因为我们*不*希望这些行号出现在我们的输出 CSV 文件中。

现在我们有了两个较小的文件样本，让我们打开它们并仅仅通过视觉比较它们的列标题及其内容。您可以在[图 6-3](#august_data_sample)中看到八月数据的屏幕截图，而最近的数据则显示在[图 6-4](#recent_data_sample)中。

![八月 PPP 贷款数据文件的前几行。](assets/ppdw_0603.png)

###### 图 6-3\. 八月 PPP 贷款数据文件的前几行

![最近 PPP 贷款数据文件的前几行。](assets/ppdw_0604.png)

###### 图 6-4\. 最近 PPP 贷款数据文件的前几行

幸运的是，看起来两个文件的前几行至少包含*一些*相同的条目。这将使我们更容易比较它们的内容，并（希望能够）确定我们可以使用哪些列来匹配行。

让我们首先选择一个单独的条目来处理，最好是尽可能多填写的条目。例如，某个名为 `SUMTER COATINGS, INC.` 的东西出现在八月数据样本的第 6 行（在电子表格界面中标记）的 `BusinessName` 列下。在最近数据样本的第 2 行，同样的值出现在 `BorrowerName` 列下。在八月样本文件中，术语 `Synovus Bank` 出现在称为 `Lender` 的列中，而该术语在最近的样本文件中出现在称为 `ServicingLenderName` 的列中。到目前为止还算顺利——或者说是吗？

尽管这些行之间的许多细节在两个数据文件中似乎匹配，但一些看似重要的内容*不*匹配。例如，在八月数据样本中，`DateApproved` 列中的值为 `05/03/2020`；而在最近的数据样本中，它是 `05/01/2020`。如果我们再看另一个看似共享的条目，我们会发现对于名为 `PLEASANT PLACES, INC.` 的企业/借款人（八月数据中的第 5 行和最近数据中的第 3 行），两个文件中的 `CD` 列（最近文件中的电子表格列 AF，八月文件中的列 P）有不同的值，显示在八月数据中为 `SC-01`，在最近的数据中为 `SC-06`。到底发生了什么？

此时，我们必须对*哪些*列值需要匹配以便我们将特定贷款行视为两者相同作出一些判断。要求业务名称和放贷人名称匹配似乎是一个很好的起点。因为我们知道现在已经进行了多轮贷款，所以我们可能还想要求`DateApproved`列中的值匹配（尽管Synovus Bank在两天内向Sumter Coatings，Inc.放款似乎不太可能）。那么不匹配的国会选区呢？如果我们查看南卡罗来纳州的国会选区地图，可以清楚地看到自2013年以来它们的边界没有变化，^([11](ch06.html#idm45143405919056))尽管第1和第6选区似乎有一个共同的边界。考虑到这一点，我们可能会得出结论这里的差异只是一个错误。

正如你所看到的，我们已经发现了一些显著的数据质量问题——而我们只看了五行数据！因为显然我们不能一次解决所有这些差异，所以我们需要从汇集（我们希望）确实匹配的行开始，同时确保跟踪任何不匹配的行。为了做到这一点，我们需要*连接*或*合并*这两个数据集。但因为我们已经知道会有差异，所以我们需要一种方式来跟踪这些差异。换句话说，我们希望我们连接过程创建的文件包括*每一个*来自*两个*数据集的行，无论它们是否匹配。为了做到这一点，我们需要使用所谓的[外连接](https://pandas.pydata.org/pandas-docs/stable/user_guide/merging.html#brief-primer-on-merge-methods-relational-algebra)，我们将在[示例 6-4](#ppp_data_join)中使用它。

注意，因为外连接保留*所有*数据行，所以我们的结果数据集可能有多达个体数据集合并后的行数——在这种情况下大约有*140万行*。不过别担心！Python可以处理。但你的设备可以吗？

如果你有一台Macintosh或Windows机器，在这些示例中处理大约500 MB的数据应该没有问题。如果像我一样，你在使用Chromebook或类似设备，现在就是迁移到云端的时刻。

##### 示例6-4\. ppp_data_join.py

```py
# quick script for creating new CSVs that each contain the first few rows of
# our larger data files

# importing the `pandas` library
import pandas as pd

# read the august data into a pandas DataFrame using its `read_csv()` method
august_ppp_data = pd.read_csv('public_150k_plus_080820.csv')

# read the recent data into a pandas DataFrame using its `read_csv()` method
recent_ppp_data = pd.read_csv('public_150k_plus_recent.csv')

# now that we have both files in memory, let's merge them!
merged_data = pd.merge(august_ppp_data,recent_ppp_data,how='outer',
    left_on=['BusinessName','Lender','DateApproved'],right_on=['BorrowerName',
    'ServicingLenderName','DateApproved'],indicator=True) ![1](assets/1.png)

# `print()` the values in the "indicator" column,
# which has a default label of `_merge`
print(merged_data.value_counts('_merge')) ![2](assets/2.png)
```

[![1](assets/1.png)](#co_assessing_data_quality_CO4-1)

我们在这里传递`indicator=True`参数，因为它将创建一个新的列，让我们知道哪些行出现在一个或两个文件中。

[![2](assets/2.png)](#co_assessing_data_quality_CO4-2)

使用`indicator=True`会生成一个名为`merge`的列，每行的值显示该行在哪个数据集中匹配。正如我们将从输出中看到的那样，该值将是`both`、`left_only`或`right_only`之一。

如果一切顺利，你会得到这样的输出：

```py
_merge
both          595866
right_only    171334
left_only      67333
dtype: int64
```

那么这意味着什么呢？看起来我们在企业名称、服务贷款人名称和贷款日期上努力匹配八月数据与最近数据成功匹配了 595,866 笔贷款。`right_only` 贷款是*最近*的贷款，未匹配成功的（`recent_ppp_data` 是我们 `pd.merge()` 函数的第二个参数，因此被认为是“右侧”）；我们找到了其中的 171,334 笔。这似乎完全合理，因为我们可以想象自八月以来可能发放了许多新贷款。

这里令人担忧的数字是 `left_only`: 67,333 笔贷款在八月的数据中出现，但未与我们最近的数据集中的任何贷款行匹配。这表明我们最近的数据可能不完整，或者存在严重的质量问题。

从我们对样本数据的非常粗略的检查中，我们已经知道 `DateApproved` 列可能存在一些问题，所以让我们看看如果我们消除匹配日期的需求会发生什么。为此，我们将只需将 [示例 6-5](#ppp_data_join_cont) 中的片段添加到 [示例 6-4](#ppp_data_join) 中，但不指定日期需要匹配。让我们看看结果如何。

##### 示例 6-5\. ppp_data_join.py（续）

```py
# merge the data again, removing the match on `DateApproved`
merged_data_no_date = pd.merge(august_ppp_data,recent_ppp_data,how='outer',
    left_on=['BusinessName','Lender'],right_on=['BorrowerName',
    'ServicingLenderName'],indicator=True)

# `print()` the values in the "indicator" column,
# which has a default label of `_merge`
print(merged_data_no_date.value_counts('_merge'))
```

现在我们的输出看起来像这样：

```py
_merge
both          671942
right_only     96656
left_only      22634
dtype: int64
```

换句话说，如果我们只要求企业名称和贷款人匹配，那么我们在最近的数据中“发现”了另外约 45,000 笔八月数据中的贷款。当然，我们*不知道*这些新“匹配”中有多少是由于数据输入错误（比如我们 `05/03/2020` 与 `05/01/2020` 的问题）导致的，有多少代表了多笔贷款。^([12](ch06.html#idm45143405666608)) 我们只知道，从八月数据中我们还找不到 22,634 笔贷款。

那么，如果我们简单地检查一个给定的企业是否同时出现在两个数据集中呢？这似乎是最基本的比较方式：理论上，负责服务 PPP 贷款的银行或贷款人可能会在多个月内变更，或者可能因为（可能的）微小数据输入差异而导致额外的不匹配。请记住：我们目前的目标仅仅是评估我们能够多么信任*最近*的数据是否包含所有的八月数据。

那么让我们再添加一个最后的、*非常*宽松的合并，看看会发生什么。添加到 [示例 6-6](#ppp_data_join_cont2) 中的片段，我们将仅仅根据企业名称匹配，看看结果如何。

##### 示例 6-6\. ppp_data_join.py（续）

```py
# merge the data again, matching only on `BusinessName`/`BorrowerName`
merged_data_biz_only = pd.merge(august_ppp_data,recent_ppp_data,how='outer',
    left_on=['BusinessName'],right_on=['BorrowerName'],indicator=True)

# `print()` the values in the "indicator" column,
# which has a default label of `_merge`
print(merged_data_biz_only.value_counts('_merge'))
```

现在我们的输出是这样的：

```py
_merge
both          706349
right_only     77064
left_only       7207
dtype: int64
```

事情看起来好了一点：在总共的 790,620（706,349 + 77,064 + 7,207）个可能贷款中，“找到”了除了 7,207 之外的所有贷款，这不到 0.1%。这*相当不错*；我们可能会倾向于把这些缺失数据量称为“四舍五入误差”然后继续前进。但在我们对已经占据了 99.9% 所有 PPP 贷款感到满足之前，让我们停下来考虑一下那些“少量”的缺失数据到底代表了什么。即使我们假设每一个这些“丢失”的贷款都是最小可能金额（请记住我们只看的贷款金额在 15 万美元或以上），这仍意味着我们最近的数据集中*至少*有 10,810,500,000 美元——超过 10 亿美元！——可能贷款未解决。考虑到我每年如何努力来计算（和支付）我的税款，我当然希望联邦政府不会简单地“丢失” 10 亿美元的纳税人的钱而不担心。但*我们*能为此做些什么呢？这就是我们开始对数据进行处理的部分，既令人望而生畏又充满活力：是时候与人交流了！

尽管联系主题专家始终是开始数据质量调查的好方法，在这种情况下，我们有更好的东西：关于放贷人和贷款接收者本身的信息。在文件中包含的企业名称和位置之间，我们可能会找到至少一些来自八月数据集中“丢失”的 7,207 笔贷款的联系信息，并尝试了解到底发生了什么。

在您拿起电话开始拨打人们的时候（是的，大多数情况下您应该*打电话*），请先想清楚您要问什么。虽然想象可能发生一些恶意行为（放贷人在隐藏资金吗？企业在误导自己吗？）很诱人，但有一句老话（而且非常灵活）可能会让你稍作思考：“绝不要归因于恶意，那些可以用无能/愚蠢/疏忽来解释的事情。”^([13](ch06.html#idm45143405581136)) 换句话说，这些贷款在最近的数据中可能没有出现是因为某些数据输入错误，甚至是因为*贷款从未到账*。

事实上，在打电话给全国各地几家汽车维修和理发店之后，这正是我经常听到的故事。我与多位受访者交谈后得出了类似的结论：他们申请了贷款，被告知贷款已获批，然后——资金就从未到账。虽然试图确认每一笔“丢失”的贷款是否如此并不现实，但从距离遥远的多家企业那里听到基本相同的故事，让我*相当有信心*这些贷款没有出现在最终数据集中，因为资金实际上从未发出。^([14](ch06.html#idm45143405576560))

到目前为止，我们可以相当肯定我们最近的PPP贷款数据实际上是“完整的”符合我们的目的。虽然对我们的数据集进行这样一个几乎十多个数据完整性措施中的其中一个测试可能感觉很多工作，但请记住，在此过程中，我们已经学到了宝贵的信息，这将使我们后续的“测试”工作更加迅速——甚至可以说是微不足道。所以，让我们转向下一个标准。

## 它是否有良好的注释？

经过确认，我们的数据在时间上是*及时的*和*完整的*后，我们需要详细了解我们最近PPP贷款数据中的数据列实际包含什么信息。^[15](ch06.html#idm45143405548048) 与我们的完整性评估一样，我们可以从数据本身开始。通过查看列名和一些包含在该列中的值，我们可以开始对我们需要更多信息的内容有所了解。如果列名看起来描述性并且我们在该列中找到的数据值与我们对列名的解释相符，那么这是一个非常好的起点。

尽管我们有几个选项可以查看我们的列名及其对应的值，让我们继续利用我们之前创建的样本文件。由于屏幕宽度通常会阻止我们轻松打印大量数据列（而我们可以轻松向下滚动查看更多行），我们将首先*转置*我们的样本数据（即将列转换为行，将行转换为列），以便更轻松地查看列标题。^[16](ch06.html#idm45143405541024) 我们还将应用一些额外的数据类型过滤，以便更容易看到缺失的数据。我们对此的第一次尝试可以在[示例 6-7](#ppp_columns_review)中看到。

##### 示例 6-7\. ppp_columns_review.py

```py
# quick script for reviewing all the column names in the PPP data
# to see what we can infer about them from the data itself

# importing the `pandas` library
import pandas as pd

# read the recent data into a pandas DataFrame using its `read_csv()` method
ppp_data_sample = pd.read_csv('recent_sample.csv')

# convert all missing data entries to '<NA>' using the `convertdtypes()` method
converted_data_sample = ppp_data_sample.convert_dtypes() ![1](assets/1.png)

# transpose the whole sample
transposed_ppp_data_sample = converted_data_sample.transpose()

# print out the results!
print(transposed_ppp_data_sample)
```

[![1](assets/1.png)](#co_assessing_data_quality_CO5-1)

为了加快速度，Pandas 的`read_csv()`方法会将所有缺失的条目转换为`NaN`（不是一个数字），详见[*https://pandas.pydata.org/pandas-docs/stable/user_guide/missing_data.html*](https://pandas.pydata.org/pandas-docs/stable/user_guide/missing_data.html)。为了将这些值转换为更通用（和直观的）*<NA>*标签，我们将`convertdtypes()`方法应用于整个DataFrame，详见[*https://pandas.pydata.org/pandas-docs/stable/user_guide/missing_data.html#missing-data-na-conversion*](https://pandas.pydata.org/pandas-docs/stable/user_guide/missing_data.html#missing-data-na-conversion)。

如您在[图 6-5](#transposed_sample_recent)中所见，这使我们可以将所有原始列名称显示为行，并将几个原始行作为数据列。透过这些信息，我们可以开始对我们已知和未知的内容有所了解。

多亏了它们相对描述性的名称，我们可以开始猜测在许多这些列中可能找到的内容。例如，像 `DateApproved`、`ProcessingMethod`、`BorrowerName`、`BorrowerAddress`、`BorrowerCity`、`BorrowerCity`、`BorrowerState` 和 `BorrowerZip` 这样的列相对直观。在某些情况下，名称是描述性的，但没有给出我们所需的所有信息。例如，虽然 `SBAGuarantyPercentage` 告诉我们关于列内容及其单位（假设是 SBA 担保的贷款金额百分比）的信息，但 `Term` 列并没有告诉我们该值应解释为 24 天、周、月还是年。同样，虽然 `BusinessAgeDescription` 中的值本身是描述性的（例如，`Existing or more than 2 years old`），`LoanStatus` 值为 `Exemption 4` 并没有真正帮助我们理解贷款的具体情况。最后，像 `LMIIndicator` 这样的列名对专家来说可能很容易解释，但对于我们这些不熟悉贷款行话的人来说可能很难识别。

此时我们真正需要的是一个“数据字典”——有时用这个术语来指代描述（特别是）表格类型数据内容的文档。数据字典非常重要，因为虽然表格类型数据非常方便进行分析，但它本身并没有提供包括我们需要回答的“应该使用什么单位？”或“编码类别意味着什么？”等关于数据*元数据*——即关于数据的数据——的方法，这些问题在处理复杂数据集时经常遇到。

找到数据字典最可能的地方应该是我们最初获取数据的地方（请记住，在[第四章](ch04.html#chapter4)中，我们发现 *ghcnd-stations.txt* 文件的描述链接在与数据位于同一文件夹的 *readme.txt* 文件中）。在这种情况下，这意味着回到 [SBA 网站](https://sba.gov/funding-programs/loans/coronavirus-relief-options/paycheck-protection-program/ppp-data)，看看我们能找到什么。

![ppdw 0605](assets/ppdw_0605.png)

###### 图 6-5\. 最近的样本数据转置

起初，情况看起来相当有希望。在该页面的“所有数据”部分下，我们看到一个链接，承诺提供“关键数据方面”的摘要，如 [图 6-6](#ppp_data_landing_page) 所示。^([17](ch06.html#idm45143405428640))

![SBA 网站上 PPP 贷款数据的登陆页面](assets/ppdw_0606.png)

###### 图 6-6\. SBA 网站上 PPP 贷款数据的登陆页面

[点击链接](https://sba.gov/document/report-paycheck-protection-program-ppp-loan-data-key-aspects) 带我们到一个页面（截至目前为止），列出了 2020 年夏季的两份 PDF 文档。不幸的是，它们都似乎不包含我们需要的内容 —— 它们大部分都是有关 PPP 贷款如何处理的免责声明类型的文字，尽管它们确实确认了我们的发现，“取消”的贷款不会出现在数据库中，如 [图 6-7](#ppp_data_aspects) 所示。

![‘回顾工资保护计划 (PPP) 数据时需要记住的信息’ 摘录](assets/ppdw_0607.png)

###### 图 6-7\. *回顾 PPP 数据时需要记住的信息* 摘录

现在呢？我们有几个选项。我们可以转向更一般的研究策略，以便了解更多关于 PPP 的信息，并希望自己填补一些空白。例如，阅读足够的针对潜在 PPP 申请者的网站文章将清楚地表明，`Term` 列的适当单位几乎肯定是周。同样，如果我们对 `LMIIndicator`、`LMI Indicator` 和 `LMI Indicator loans` 进行足够的网络搜索，最终我们将找到一个 Wikipedia 页面，该页面表明这个术语 *可能* 是 “Loan Mortgage Insurance Indicator” 的简称 —— 但我们几乎肯定不会确定。

换句话说，现在是再次寻找一些人类帮助的时候了。但是我们可以联系谁呢？已经查看过 PPP 数据的学者可能是一个开始的地方，但如果数据发布比较新，可能会很难找到他们。因此，就像我们在尝试确认所有那些缺失贷款从八月数据集中发生了什么时一样，我们将直接去找信息源：SBA。幸运的是，如果我们回到下载实际数据文件的网站，如 [图 6-8](#ppp_box_site) 所示，我们会发现上传这些文件的人是 Stephen Morris。^([18](ch06.html#idm45143405416416))

![PPP 数据下载门户](assets/ppdw_0608.png)

###### 图 6-8\. PPP 数据下载门户

在打电话和发送电子邮件后，我得知，在我询问的时候，SBA 还没有为 PPP 贷款数据创建数据字典，尽管莫里斯确实向我推荐了原始 PDF 和 SBA 的 7a 贷款计划的数据字典，而 PPP 贷款结构正是基于这个计划。尽管后者的文件（位于 [*https://data.sba.gov/dataset/7-a-504-foia*](https://data.sba.gov/dataset/7-a-504-foia)）与 PPP 贷款数据列仍有显著差异，但它确实提供了一些关键元素的见解。例如，该文件包括一个名为 `LoanStatus` 的列描述，其可能的值似乎至少部分与我们迄今在 PPP 贷款数据中找到的内容相似：

```py
LoanStatus        Current status of loan:
               • NOT FUNDED = Undisbursed
               • PIF = Paid In Full
               • CHGOFF = Charged Off
               • CANCLD = Canceled
               • EXEMPT = The status of loans that have been disbursed but
                   have not been canceled, paid in full, or charged off are
                   exempt from disclosure under FOIA Exemption 4
```

这些数据是否“充分标注”？在某种程度上是的。它的标注程度不如我们在[“最后，固定宽度”](ch04.html#fixed_width)中使用的NOAA数据那么好，但通过一些努力，我们很可能能够建立起对PPP贷款数据的自己的数据字典，并且对其相当有信心。

## 它是否高容量？

由于我们已经完成了“完整性”审查，我们基本上已经回答了我们使用的数据是否*一般*是高容量的问题。在超过750,000行数据的情况下，几乎没有情况是不足以至少进行*一些*有用形式的分析的。

与此同时，我们也不能确定我们将能够进行哪些类型的分析，因为我们拥有的数据行数并不重要，如果其中大多数是空的话。那么我们该如何检查呢？对于我们只期望找到少量可能值的列，比如`LoanStatus`，我们可以使用Pandas的`value_counts()`方法来总结其内容。对于具有非常多样化值的列（如`BorrowerName`或`BorrowerAddress`），我们将需要专门检查*缺失*值，然后将其与总行数进行比较，以了解可能缺失的数据量。首先，我们将总结我们预计只包含少数不同值的列，如[示例 6-8](#ppp_columns_summary)所示。我们还将统计更多种类列中的`NA`条目——在本例中是`BorrowerAddress`。

##### 示例 6-8\. ppp_columns_summary.py

```py
# quick script for reviewing all the column names in the PPP data
# to see what we can infer about them from the data itself

# importing the `pandas` library
import pandas as pd

# read the recent data sample into a pandas DataFrame
ppp_data = pd.read_csv('public_150k_plus_recent.csv')

# print the summary of values that appear in the `LoanStatus` column
print(ppp_data.value_counts('LoanStatus'))

# print the total number of entries in the `LoanStatus` column
print(sum(ppp_data.value_counts('LoanStatus')))

# print the summary of values that appear in the `Gender` column
print(ppp_data.value_counts('Gender'))

# print the total number of entries in the `Gender` column
print(sum(ppp_data.value_counts('Gender')))

# print how many rows do not list a value for `BorrowerAddress`
print(ppp_data['BorrowerAddress'].isna().sum())
```

[示例 6-8](#ppp_columns_summary)的输出，如[示例 6-9](#recent_col_summaries)所示，开始描绘了我们数据集的约750,000行中实际包含了哪些数据。例如，大多数贷款的状态是“第四项豁免”，我们从注释调查中知道这意味着“豁免免于FOIA第四项规定下的披露”。^([19](ch06.html#idm45143405275328)) 类似地，我们可以看到超过三分之二的贷款申请者在申请时未指明其性别，还有17笔贷款甚至没有列出借款人的地址！

##### 示例 6-9\. 最近的数据列摘要

```py
LoanStatus
Exemption 4            549011
Paid in Full           110120
Active Un-Disbursed    107368
dtype: int64
766499

Gender
Unanswered      563074
Male Owned      168969
Female Owned     34456
dtype: int64
766499

17
```

这些数据是否“高容量”？是的——虽然整体上我们丢失了相当多的数据，但在我们所拥有的行和列中似乎有相对高比例的有用信息。请记住，在某些情况下，仅有几十行数据就足以生成有关某事物的有用见解——只要它们包含真正有意义的信息。这就是为什么仅仅查看数据文件的大小或者它包含的行数来确认它是否真正“高容量”是不够的——我们实际上必须详细检查数据才能确定。

## 它是否一致？

另一件事，我们从先前的完整性检查中了解到的是，PPP贷款数据的格式在2020年8月和之后的发布版本之间绝对*不*一致：2020年12月开始发布的数据比之前发布的数据[更加详细](https://washingtonpost.com/business/2020/06/11/trump-administration-wont-say-who-got-511-billion-taxpayer-backed-coronavirus-loans)，甚至大多数列名在两个文件之间也有所不同。

由于我们现在仅依赖于最近的数据文件，我们需要考虑的一种不同类型的一致性是数据集内部值的一致性。例如，我们可能希望像`InitialApprovalAmount`和`CurrentApprovalAmount`这样的金额列中使用的单位是相同的。不过，最好还是检查一下，以防万一。在[示例 6-10](#ppp_min_max_loan)中，我们将再次进行快速的最小/最大确认，以确保这些数字落在我们预期的范围内。

##### 示例 6-10\. ppp_min_max_loan.py

```py
# quick script for finding the minimum and maximum loans currently approved
# in our PPP loan dataset

# importing the `pandas` library
import pandas as pd

# read the recent data into a pandas DataFrame
ppp_data = pd.read_csv('public_150k_plus_recent.csv')

# use the pandas `min()` and `max()` methods to retrieve the
# largest and smallest values, respectively
print(ppp_data['CurrentApprovalAmount'].min())
print(ppp_data['CurrentApprovalAmount'].max())
```

根据我们数据文件的标题—*150k_plus*—我们预计我们将在批准的*最低*贷款金额中找到$150,000。快速搜索“PPP最大贷款额”将我们导向SBA网站上的一份文件，该文件表明对于大多数类型的企业，^([20](ch06.html#idm45143405217296))最大贷款金额为$10 million。事实上，运行我们的脚本似乎确认了这一最小和最大值在我们的数据中得到了反映：

```py
150000.0
10000000.0
```

此时，我们还希望检查最常见（也是最隐蔽的）不一致形式之一：拼写差异。*每当*你处理由人类输入的数据时，都会遇到拼写问题：额外的空格、打字错误和标点差异，至少是这些问题。这是一个问题，因为如果我们想要回答一个看似简单的问题，比如“X银行的贷款来源于多少？”，我们需要在整个数据中保持该银行名称的拼写一致。但几乎可以肯定的是，起初是不会一致的。

幸运的是，因为这是一个如此常见的数据问题，已经有许多成熟的方法来处理它。在这里，我们将使用一种称为*指纹技术*的方法开始检查数据集中银行名称拼写的一致性。虽然在寻找拼写不同的重复项时可以按照音标等方式对这些公司名称进行分组，但我们选择指纹技术，因为它遵循一个简单、严格但有效的*算法*（实际上，只是一组集合），可以尽量减少我们将两个本不应该相同的名称匹配起来的风险。

具体来说，我们将使用的指纹算法执行以下操作：^([21](ch06.html#idm45143405172768))

1.  删除前导和尾随空格

1.  将所有字符更改为它们的小写表示

1.  删除所有标点符号和控制字符

1.  将扩展的西方字符标准化为它们的ASCII表示（例如“gödel” → “godel”）

1.  将字符串分割为以空格分隔的*标记*

1.  对标记进行排序并删除重复项

1.  将标记重新连接起来

和往常一样，虽然我们可以自己编写此代码，但我们很幸运地发现Python社区中已有人已经完成了这项工作，并创建了*pip*可安装的库*fingerprints*：

```py
pip install fingerprints
```

目前，我们的主要关注点是确认是否存在拼写差异；实际上，我们将在[第7章](ch07.html#chapter7)中研究如何处理这些差异。因此，我们现在只需计算数据集中所有唯一银行名称的数量，然后查看该列表中有多少个唯一的指纹。如果文件中的所有银行名称确实都是不同的，那么从*理论上*讲，这两个列表的长度应该相同。另一方面，如果数据集中的某些银行名称仅因为轻微的标点符号和空白差异而“唯一”，那么我们的指纹列表将会比“唯一”银行名称列表*短*。这表明，为了确保例如我们确实能够检索与单个银行相关的*所有*贷款，我们需要进行一些数据转换。 ([示例 6-11](#ppp_lender_names))。

##### 示例 6-11\. ppp_lender_names.py

```py
# quick script for determining whether there are typos &c. in any of the PPP
# loan data's bank names

# importing the `pandas` library
import pandas as pd

# importing the `fingerprints` library, which will help us generate normalized
# labels for each of the bank names in our dataset
import fingerprints

# read the recent data into a pandas DataFrame
ppp_data = pd.read_csv('public_150k_plus_recent.csv')

# use the pandas DataFrame `unique()` method to create a list of unique
# bank names in our data's `OriginatingLender` column
unique_names = ppp_data['OriginatingLender'].unique()

# confirm how many unique names there are
print(len(unique_names))

# create an empty list to hold the fingerprint of each of the unique names
fingerprint_list = []

# iterate through each name in the list of unique names
for name in unique_names:

    # for each name, generate its fingerprint
    # and append it to the end of the list
    fingerprint_list.append(fingerprints.generate(name))

# use the `set()` function to remove duplicates and sort `fingerprint_list`
fingerprint_set = set(fingerprint_list)

# check the length of `fingerprint_set`
print(len(fingerprint_set))
```

运行此脚本会产生输出：^([22](ch06.html#idm45143405153824))

```py
4337
4242
```

两个列表之间的长度差异确实表明我们的数据集中存在一些拼写差异：原始数据中“唯一”名称的数量为4,337个，但这些名称的不同指纹数量仅为4,242个。尽管这里的差异“仅”约为100项，但这些差异可能会影响数千行数据，因为每家银行平均发放了数百笔贷款（750,000/4337 = 约173）。因此，我们无法确定我们的原始数据集中有多少行包含给定的“拼写错误”名称（也无法确定这是否是详尽无遗的）。在[“校正拼写不一致”](ch07.html#correcting_inconsistencies)一节中，我们将通过使用这些指纹来转换我们的数据，以更好地识别特定的出借人和借款人。

## 它是多变量吗？

就像数据集如果包含许多行，更有可能是高容量一样，如果数据集有许多列，则更有可能是多变量的。然而，与我们对容量的质量检查类似，确定我们所拥有的列是否使我们的数据真正成为多变量，还需要对它们所包含的数据进行质量检查。

例如，虽然我们的PPP贷款数据包含50个数据列，其中约有十几列基本上是扩展的地址，因为借款人的位置、发放贷款者和服务贷款者各自被分解为一个独立的列，包括街道地址、城市、州和邮政编码。虽然许多其余的列可能包含唯一的数据，我们需要了解它们包含的内容，以了解我们实际上有多少数据特征或*特征*可供使用。

例如，我们的数据集中有多少笔贷款报告称申请资金不仅用于工资支出？尽管数据结构（以及贷款计划）允许借款人将贷款用于其他用途（如[医疗费用和租金](https://sba.gov/funding-programs/loans/covid-19-relief-options/paycheck-protection-program/first-draw-ppp-loan)），但这在数据中显示出了多大程度上？

就像我们评估我们的数据真正是多么高的体积一样，我们将查看更多列的内容，以确定它们似乎提供的详细信息是否确实反映在它们包含的数据量中。在这种情况下（[例子 6-12](#ppp_loan_uses)），我们将查看每个`PROCEED`列中多少行不包含值。

##### 例子 6-12\. ppp_loan_uses.py

```py
# quick script for determining what borrowers did (or really, did not) state
# they would use PPP loan funds for

# importing the `pandas` library
import pandas as pd

# read the recent data sample into a pandas DataFrame
ppp_data = pd.read_csv('public_150k_plus_recent.csv')

# print how many rows do not list a value for `UTILITIES_PROCEED`
print(ppp_data['UTILITIES_PROCEED'].isna().sum())

# print how many rows do not list a value for `PAYROLL_PROCEED`
print(ppp_data['PAYROLL_PROCEED'].isna().sum())

# print how many rows do not list a value for `MORTGAGE_INTEREST_PROCEED`
print(ppp_data['MORTGAGE_INTEREST_PROCEED'].isna().sum())

# print how many rows do not list a value for `RENT_PROCEED`
print(ppp_data['RENT_PROCEED'].isna().sum())

# print how many rows do not list a value for `REFINANCE_EIDL_PROCEED`
print(ppp_data['REFINANCE_EIDL_PROCEED'].isna().sum())

# print how many rows do not list a value for `HEALTH_CARE_PROCEED`
print(ppp_data['HEALTH_CARE_PROCEED'].isna().sum())

# print how many rows do not list a value for `DEBT_INTEREST_PROCEED`
print(ppp_data['DEBT_INTEREST_PROCEED'].isna().sum())

# create a new DataFrame that contains all rows reporting *only* payroll costs
# that is, where all _other_ costs are listed as "NA"
payroll_only = ppp_data[(ppp_data['UTILITIES_PROCEED'].isna()) & (ppp_data
    ['MORTGAGE_INTEREST_PROCEED'].isna()) & (ppp_data
    ['MORTGAGE_INTEREST_PROCEED'].isna()) & (ppp_data['RENT_PROCEED'].isna()) &
    (ppp_data['REFINANCE_EIDL_PROCEED'].isna()) &  (ppp_data
    ['HEALTH_CARE_PROCEED'].isna()) & (ppp_data['DEBT_INTEREST_PROCEED'].isna())
    ]

# print the length of our "payroll costs only" DataFrame
print(len(payroll_only.index))
```

正如我们从[例子 6-13](#reported_use_of_funds)的输出中所看到的，绝大多数企业（除了1,828家）在申请时表示他们打算将资金用于支付工资支出，不到三分之一的企业报告称他们可能也会用这笔钱支付公用事业费用。另一部分提供了有关用于租金的信息。与此同时，我们最后的测试显示，超过三分之二的所有企业仅列出了工资支出作为其PPP资金的预期使用。

##### 例子 6-13\. 报告的PPP贷款资金使用

```py
570995
1828
719946
666788
743125
708892
734456
538905
```

这对我们的数据“多元化”有什么意义？即使我们决定不考虑专门用于地址详细信息的额外列，或看似未充分利用的`PROCEED`列，这个数据集中仍然包含大量信息可供探索。看起来我们很可能能够至少开始得出有关谁获得了PPP贷款以及他们如何使用资金的结论。然而，我们不能假设我们的数据集的列或行具有数据内容，直到我们自己检查和确认。

## 是否原子？

这是另一个实例，我们先前在数据方面的工作让我们能够在数据完整性这个指标上相当快地说“是的！”。虽然我们的数据在八月版本只包含贷款金额范围，但我们知道这个数据集每一行都包含一个贷款，包括它们的确切金额。由于我们有具体的数字而不是摘要或汇总值，我们可以相当有信心地认为我们的数据足够细粒度或“原子性”，以支持后续可能的广泛数据分析。

## 是否清楚？

尽管这个数据集并*不*特别注释详细，但由于大部分列标签及其含义都相当清晰，我们仍然能够通过许多数据完整性检查。例如，如果我们不确定`CD`代表什么，看一下一些值（例如`SC-05`）使得推断这代表“国会选区”相当直接。随着我们在处理公共和政府数据集方面获得更多经验（或在特定学科领域工作），越来越多的代码和行话将更快地变得清晰。

对于那些标签不太清晰的列，与SBA的斯蒂芬·莫里斯交换了几封电子邮件是有益的。例如，他确认了`Term`列的适当单位是月（而不是最初可能的周），以及`PROCEED`列描述了贷款资金将用于的情况，“根据‘贷款人在借款人申请表上向SBA提交的内容’。”

我与莫里斯的通信也说明了，尽可能地向主要信息来源专家求助是进行数据完整性检查的重要步骤。如果你还记得，开始时某些列标题的含义*并不*清楚，其中之一是`LMIIndicator`。由于我的样本数据行没有包含此列的值，我开始进行一些网络搜索，结果包括“贷款人抵押保险”（如[图 6-9](#lmi_indicator_search_results)所示）；当时，这*看起来*像是对列标题的合理解释。

![LMI指标贷款的搜索结果](assets/ppdw_0609.png)

###### 图 6-9\. LMI指标贷款的搜索结果

唯一的问题？ 它是错误的。正如莫里斯在电子邮件中澄清的那样，“LMI指标告诉我们借款人是否地理上位于低中等收入地区。”

这里的教训是，如果你没有官方的数据词典，你总是要对你试图从列标题中推断出多少东西持有一些谨慎态度；即使*看起来*清楚的那些也可能不是你所想象的意思。如果有任何疑问，请务必联系一些专家（最好是编制数据的人）来确认你的推断。

## 它是否具有维度结构？

如果当我们在[第三章](ch03.html#chapter3)讨论它时，*维度结构化*数据的概念似乎有点抽象，希望现在我们面前有一个真实数据集后能更容易理解一些。维度结构化数据包括有关我们可以用来对数据进行分组的有用类别或类别的信息，以及帮助我们进行更细粒度分析的更原子特征。

就我们的PPP贷款数据而言，我认为一些有用的“维度”数据列包括`RuralUrbanIndicator`、`HubzoneIndicator`、新澄清的`LMIIndicator`、`NAICSCode`，甚至在某种程度上`SBAOfficeCode`。 像`RaceEthnicity`、`Gender`和`Veteran`这样的列在维度上也可能有用，但我们知道其中许多是“未回答”的，这限制了我们可以从中推断的内容。 另一方面，其他数据列可以帮助我们回答有关到目前为止从PPP中受益的企业的位置和类型的有用问题。

更甚者，像`NAICSCode`这样的列提供了通过了解受益企业属于哪些行业的可能性，从而有助于我们与美国劳工统计局和其他有关美国就业部门的数据集进行比较。 我们将在 [“增强数据”](ch07.html#augmenting_data) 中更深入地探讨这个过程。

到目前为止，我们已经能够对一些数据完整性问题作出肯定的回答，而其他问题则更多是有条件的，这表明在我们进入数据分析阶段之前，我们需要进行额外的转换和评估。 然而，在这之前，我们需要转向数据*适配性*的关键问题：我们的数据是否展示了我们需要的*有效性*、*可靠性*和*代表性*，以便对美国小企业如何受PPP影响进行有意义的结论。

# 评估数据适配性

现在，我们已经根据几乎十多个不同的度量评估了我们数据集的完整性，是时候评估它对我们目的的适配程度了；也就是说，这些数据是否真的能够为我们提供我们所询问的问题的答案。 为此，我们将依据我们的三个主要数据*适配性*标准：有效性、可靠性和代表性。 现在是我们需要考虑我们原始问题并思考的时候了：PPP是否帮助挽救了美国的小企业？

## 有效性

请记住，我们对有效性的工作定义是“某物品所测量的程度是否符合其预期”。 即使我们的PPP贷款数据是完美的，我们仍然需要以某种方式确定它是否能在某种程度上回答那个问题。 在这一点上，我们知道我们的数据集提供了这个问题至关重要的一个关键部分的答案，因为我们现在相当有信心，它准确地详细说明了哪些企业目前拥有*批准*的PPP贷款。 通过我们对其完整性的调查（特别是围绕完整性和我们对注释信息或*元数据*的搜索），我们也相当有信心，已经*取消*的贷款不会出现在数据集中——我们通过SBA关于PPP计划的公布信息以及直接联系企业已经确认了这一点。

我们还可以使用数据集的元素（具体来说是`LoanStatus`和`LoanStatusDate`）来感知那些已经被批准贷款的75万多家企业中，哪些实际上已经收到了资金。我们可以通过首先使用`value_counts()`方法对`LoanStatus`列进行总结来进行检查，就像我们之前展示的那样，见[示例 6-14](#ppp_loan_status)。

##### 示例 6-14\. ppp_loan_status.py

```py
# quick script for determining how many loans have been disbursed

# importing the `pandas` library
import pandas as pd

# read the recent data sample into a pandas DataFrame
ppp_data = pd.read_csv('public_150k_plus_recent.csv')

# print a summary of values in the `LoanStatus` column
print(ppp_data['LoanStatus'].value_counts())
print(sum(ppp_data['LoanStatus'].value_counts())) ![1](assets/1.png)
```

[![1](assets/1.png)](#co_assessing_data_quality_CO6-1)

请注意，由于`value_counts()`方法*不*包括`NA`值，我也在汇总条目以确保每一行都已被记录。

此脚本的输出，如[示例 6-15](#loanstatus_summary)所示，确认了我们数据集中目前有766,499笔贷款，其中超过100,000笔尚未实际发送给企业，而另外超过100,000家企业似乎已经偿还了贷款。

##### 示例 6-15\. `LoanStatus` 摘要

```py
Exemption 4            549011
Paid in Full           110120
Active Un-Disbursed    107368
Name: LoanStatus, dtype: int64
766499
```

如果我们希望评估收到PPP贷款的小企业的命运，那么我们需要确保只关注那些实际上已经*收到*资金的企业——这意味着我们应该将我们的查询限制在`LoanStatus`值为“Exemption 4”或“Paid in Full”的企业上。

理论上，当企业申请PPP贷款时，它们要求足够的资金来维持其业务的运营，因此我们可能会倾向于假设，如果一个企业得到了PPP资金，它应该做得不错。但是，就像可能过于宽松的标准可能已经允许许多企业[欺诈地获得PPP贷款](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3906395)一样，企业*收到*PPP资金并不保证它仍然做得好。这一现实由《华尔街日报》的这篇文章所体现，[讲述了一家尽管收到PPP贷款仍然破产的公司的故事](https://wsj.com/articles/hundreds-of-companies-that-got-stimulus-aid-have-failed-11605609180)。由于我们已经知道这家企业破产了，找到其记录在我们的数据集中可以帮助我们了解这些记录可能的情况，如[示例 6-16](#ppp_find_waterford)所示。

##### 示例 6-16\. *ppp_find_waterford.py*

```py
# quick script for finding a business within our dataset by (partial) name

# importing the `pandas` library
import pandas as pd

# read the recent data sample into a pandas DataFrame
ppp_data = pd.read_csv('public_150k_plus_recent.csv')

# create a DataFrame without any missing `BorrowerName` values
ppp_data_named_borrowers = ppp_data[ppp_data['BorrowerName'].notna()] ![1](assets/1.png)

# because precise matching can be tricky,
# we'll use the pandas `str.contains()` method
bankruptcy_example = ppp_data_named_borrowers[ \
                                ppp_data_named_borrowers['BorrowerName']
                                .str.contains('WATERFORD RECEPTIONS')] ![2](assets/2.png)

# transposing the result so it's easier to read
print(bankruptcy_example.transpose())
```

[![1](assets/1.png)](#co_assessing_data_quality_CO7-1)

Pandas无法搜索具有`NA`值的任何列中的字符串，因此我们需要创建一个不包含目标列中任何这些值的DataFrame，仅供审核目的（显然，我们可能希望调查没有命名借款人的贷款）。

[![2](assets/2.png)](#co_assessing_data_quality_CO7-2)

虽然`str.contains()`将成功地匹配字符串的一部分，但它*是*区分大小写的。这意味着借款人姓名全为大写的事实很重要！

从这个脚本的以下输出来看，情况显而易见：贷款以“Exemption 4”的状态出现，并且也许更有趣的是，具有`LoanStatusDate`为`NA`。但除此之外，并没有迹象表明这家企业，呃，已经不再营业。

```py
LoanNumber                                        7560217107
DateApproved                                      04/14/2020
SBAOfficeCode                                            353
ProcessingMethod                                         PPP
BorrowerName                       WATERFORD RECEPTIONS, LLC
BorrowerAddress                         6715 COMMERCE STREET
BorrowerCity                                     SPRINGFIELD
BorrowerState                                             VA
BorrowerZip                                            22150
LoanStatusDate                                           NaN
LoanStatus                                       Exemption 4
Term                                                      24
SBAGuarantyPercentage                                    100
InitialApprovalAmount                               413345.0
CurrentApprovalAmount                               413345.0
UndisbursedAmount                                        0.0
FranchiseName                                            NaN
ServicingLenderLocationID                             122873
ServicingLenderName                                EagleBank
ServicingLenderAddress                     7815 Woodmont Ave
ServicingLenderCity                                 BETHESDA
ServicingLenderState                                      MD
ServicingLenderZip                                     20814
RuralUrbanIndicator                                        U
HubzoneIndicator                                           N
LMIIndicator                                             NaN
BusinessAgeDescription       New Business or 2 years or less
ProjectCity                                      SPRINGFIELD
ProjectCountyName                                    FAIRFAX
ProjectState                                              VA
ProjectZip                                        22150-0001
CD                                                     VA-08
JobsReported                                            45.0
NAICSCode                                           722320.0
RaceEthnicity                                     Unanswered
UTILITIES_PROCEED                                        NaN
PAYROLL_PROCEED                                     413345.0
MORTGAGE_INTEREST_PROCEED                                NaN
RENT_PROCEED                                             NaN
REFINANCE_EIDL_PROCEED                                   NaN
HEALTH_CARE_PROCEED                                      NaN
DEBT_INTEREST_PROCEED                                    NaN
BusinessType                 Limited  Liability Company(LLC)
OriginatingLenderLocationID                           122873
OriginatingLender                                  EagleBank
OriginatingLenderCity                               BETHESDA
OriginatingLenderState                                    MD
Gender                                            Male Owned
Veteran                                          Non-Veteran
NonProfit                                                NaN
```

实际上，如果我们快速检查具有`LoanStatusDate`为`NA`的贷款数量，通过将以下行添加到我们脚本的末尾，我们可以看到它与`LoanStatus`为`Exemption 4`的贷款完全匹配：

```py
print(sum(ppp_data['LoanStatusDate'].isna()))
```

那么，这个PPP贷款数据是否*衡量了它应该衡量的内容*？我会说是的，但这并不是全部。正如我们从`LoanStatus`信息的总结中所看到的，出现在这个数据集中的并非所有企业*实际上都得到了贷款*；它们已经获得批准，并且仍然*可能*获得资金（我们知道它们的贷款没有被取消）——但其中有107,368家尚未拿到资金，我们无法确定它们是否最终会获得。

仅仅从这个数据集中，我们也不能说出哪些企业已经获得了资金。有些可能仍在运营，而其他一些可能已经破产。还有些可能在没有申请破产的情况下清算了。换句话说，虽然PPP数据在回答我们问题的某些部分上具有很强的有效性，但要回答整个问题将需要远远不止这一个数据集。

## 可靠性

谈到*可靠性*时，我们感兴趣的主要标准是*准确性*和*稳定性*。换句话说，PPP数据在多大程度上反映了谁获得了PPP贷款，以及随着时间的推移，获得这些贷款的人群是否可能发生变化？

多亏了我们先前的调查，我们现在知道这个数据集的*稳定性*远非完美。在8月份数据集中被批准贷款并出现的数千家企业，在当前数据集中*没有*出现（我们将在下一节讨论其对*代表性*的影响），这与SBA的文件^([23](ch06.html#idm45143404387856))相符，取消的贷款不包括在内^([24](ch06.html#idm45143404385984))。此外，目前还不清楚在更新时，先前版本的数据是否仍然可用，这使得除非我们开始下载并归档每个版本，否则很难确定发生了什么变化。

即使数据集本身的数字随时间可能也不是特别稳定。例如，我们知道有538,905家企业报告称他们只会将他们的PPP贷款用于工资支出。但正如SBA代表斯蒂芬·莫里斯通过电子邮件解释的那样：“这些数据在某种程度上是推测性的，因为并不要求借款人将资金用于他们在申请书上选择的用途。”换句话说，除非PPP贷款接收者的贷款原谅或偿还过程*要求*详细说明资金的使用情况（并且该信息随后在该数据集中更新），否则我们无法确定我们在各种`PROCEED`列中看到的数字是否准确或稳定。

## 代表性

PPP贷款数据是否代表了实际获得PPP贷款的所有人？很可能。在公众的强烈抗议之后，许多大型和/或上市公司退还了早期的PPP贷款，SBA表示他们将密切审查200万美元以上的贷款，并且到7月初，已经有近300亿美元的贷款被退还或取消了。在我们相对详尽的8月和2月数据集之间的比较之后，我们可以相当有信心地说，我们所拥有的数据集至少代表了迄今为止谁收到了PPP贷款。

与此同时，这并没有像我们可能认为的那样告诉我们太多信息。我们已经知道，在这些数据中出现的绝大多数PPP贷款接收者没有披露他们的性别、种族、族裔或退伍军人身份，这意味着我们实际上无法知道PPP贷款接收者的人口统计信息（如果有的话）是否反映了美国小企业主群体的情况。事实上，正如我们将在[第9章](ch09.html#chapter9)中看到的，我们几乎不可能对PPP贷款接收者的人口统计信息做出结论，因为提供此类信息的申请者数量非常少。

但是代表性的问题实际上远不止于此，还涉及到（以及超出）在8月数据发布和更近期的数据发布之间消失的7207笔贷款。这些缺失的贷款反映了申请贷款并获批的企业，但用一名员工的话来说：“这些钱从来没到账。”这意味着虽然我们知道有多少企业*获得了*PPP贷款，但我们无法知道有*多少企业申请了*。因为这些取消的贷款已经从数据中移除，我们现在面临的是我所称的“分母”问题的实例。

### 分母问题

几乎每个数据驱动探究领域都承认了分母问题，尽管称呼不同。有时被称为*基准测试*或*基准问题*^([25](ch06.html#idm45143404363328))，分母问题概括了在缺乏足够比较信息以便将数据置于上下文中时的困难。在大多数情况下，这是因为你真正需要的比较数据从未被收集。

在我们探索PPP数据的过程中，我们已经遇到了这个问题的一个版本：我们知道哪些企业获得了贷款，但我们不知道谁申请了却被拒绝了，或者为什么被拒绝了（至少在某些情况下，甚至是接受者也不知道）。对于评估PPP贷款流程来说，这是一个问题，因为我们不知道是否有合法的申请者被拒绝，即使一些企业获得了多轮贷款。如果我们想知道贷款的分配是否公平——甚至有效——了解谁*没有*被包括进来和谁*被*包括进来一样重要。

到目前为止，我们遇到的一些分母问题可能可以通过某种类型的补充数据来回答——这就是*《华尔街日报》*在将PPP贷款数据与破产申请进行比较时所做的。在其他情况下，解决方案将是我们自己建立我们自己的存档，如果——似乎在这里是这种情况——数据提供者未提供早期数据与更新版本一起提供。

# 结论

那么，薪资保护计划是否帮助挽救了小企业呢？*也许*。一方面，很难想象这么多钱可以花掉而没有*一些*积极的效果。另一方面，现实世界——以及我们对其的数据——是一个复杂、相互依存的混乱。如果薪资保护计划的贷款接受者调整了其业务模式并找到了新的收入来源，我们会把它归类为“由PPP拯救”类别吗？同样地，如果一家未获得贷款的企业失败了，是因为它没有获得贷款，还是它本来就会失败？我们提出越来越多这样的“如果”，就越有可能发生一些事情：

1.  我们会头痛欲裂，认为**真的**什么都不可能知道，并寻找一种安心的拖延方式。

1.  在玩游戏/读互联网/向困惑的朋友和亲戚抱怨我们如何花费了所有时间来处理一个我们不确定其价值的数据集之后，我们会遇到一些能给我们提出一个*稍微*不同、可能*更窄*的问题的东西，并兴奋地回到我们的数据集，渴望看看我们是否能比上一个问题更好地回答这个新问题。

这个过程是艰难而曲折的吗？是的。这也让我想起了我们在[“How? And for Whom?”](ch03.html#how_for_whom)中关于建构效度的讨论。最终，这实际上也是*新知识是如何形成的*。考虑新选项、深思熟虑、测试并（可能）以稍微更多的信息和理解重新开始整个过程的不确定、令人沮丧和易失的工作，正是真正*独创*见解形成的方式。这是世界上每个算法系统都在拼命尝试模拟或模仿的事情。但如果你愿意付出努力，*你可以*真正成功。

此时，我觉得我已经对这个数据集有了相当多的了解——足够让我知道，我可能*无法*用它来回答我的最初问题，但仍然可以想象它可能产生一些有趣的见解。例如，我确信最近的数据准确反映了当前批准的贷款状态，因为我确认了在更近期文件中缺失的贷款（出于某种原因）可能实际上从未实际发放。同时，尽管SBA宣布截至2021年1月初已经宽限[超过1000亿美元的PPP贷款已经被宽限](https://sba.gov/article/2021/jan/12/11-million-paycheck-protection-program-loans-forgiven-so-far-totaling-over-100-billion)，但似乎在6周多之后，`LoanStatus`列中并没有明确的值表示已宽限的贷款。虽然SBA的Stephen Morris在2021年3月初停止回复我的电子邮件，但截至2021年5月初，数据字典似乎是可用的，^([26](ch06.html#idm45143404333728))尽管它和更新的数据都没有包含这些信息。

当然，在这里还有很多要学习的：关于谁获得了贷款及他们所在地的信息，他们获得了多少批准的贷款，以及谁在发放这些贷款。虽然数据还远非完美，但我可以保留过去数据集的副本以安抚自己，以便将来如果有重大变化，至少可以手边资源来察觉它。鉴于此，是时候从我们工作的评估阶段转入实际清理和转换数据的任务，我们将在[第7章](ch07.html#chapter7)中处理。

^([1](ch06.html#idm45143406437120-marker))即使他们并不总是意识到这一点。

^([2](ch06.html#idm45143406492528-marker))详情请参阅[*https://en.wikipedia.org/wiki/Paycheck_Protection_Program*](https://en.wikipedia.org/wiki/Paycheck_Protection_Program)。

^([3](ch06.html#idm45143406490368-marker)) “谁得到了半万亿美元的COVID贷款？特朗普政府不会说”，[*https://marketplace.org/shows/make-me-smart-with-kai-and-molly/who-got-half-a-billion-in-covid-loans-the-trump-administration-wont-say*](https://marketplace.org/shows/make-me-smart-with-kai-and-molly/who-got-half-a-billion-in-covid-loans-the-trump-administration-wont-say)。

^([4](ch06.html#idm45143406483424-marker)) 你可以在[*https://github.com/adam-p/markdown-here/wiki/Markdown-Here-Cheatsheet*](https://github.com/adam-p/markdown-here/wiki/Markdown-Here-Cheatsheet)找到一个很好的 Markdown 速查表。

^([5](ch06.html#idm45143406687968-marker)) 这个内容的原始链接是[*https://home.treasury.gov/policy-issues/cares-act/assistance-for-small-businesses/sba-paycheck-protection-program-loan-level-data*](https://home.treasury.gov/policy-issues/cares-act/assistance-for-small-businesses/sba-paycheck-protection-program-loan-level-data)，然而现在可以在[*https://home.treasury.gov/policy-issues/coronavirus/assistance-for-small-businesses/paycheck-protection-program*](https://home.treasury.gov/policy-issues/coronavirus/assistance-for-small-businesses/paycheck-protection-program)找到。

^([6](ch06.html#idm45143406684480-marker)) 而这第二个链接是[*https://sba.gov/funding-programs/loans/coronavirus-relief-options/paycheck-protection-program/ppp-data*](https://sba.gov/funding-programs/loans/coronavirus-relief-options/paycheck-protection-program/ppp-data)。

^([7](ch06.html#idm45143406648768-marker)) 你可以在[*https://drive.google.com/file/d/1EtUB0nK9aQeWWWGUOiayO9Oe-avsKvXH/view?usp=sharing*](https://drive.google.com/file/d/1EtUB0nK9aQeWWWGUOiayO9Oe-avsKvXH/view?usp=sharing)下载这个文件。

^([8](ch06.html#idm45143406646448-marker)) 2020年8月的数据可以在[*https://drive.google.com/file/d/11wTOapbAzcfeCQVVB-YJFIpsQVaZxJAm/view?usp=sharing*](https://drive.google.com/file/d/11wTOapbAzcfeCQVVB-YJFIpsQVaZxJAm/view?usp=sharing)找到；2021年2月的数据可以在[*https://drive.google.com/file/d/1EtUB0nK9aQeWWWGUOiayO9Oe-avsKvXH/view?usp=sharing*](https://drive.google.com/file/d/1EtUB0nK9aQeWWWGUOiayO9Oe-avsKvXH/view?usp=sharing)找到。

^([9](ch06.html#idm45143406638368-marker)) 例如，2008年金融危机后，有许多银行明显没有妥善记录，因为他们重新打包和出售或“证券化”住房抵押贷款，导致一些法院[拒绝他们对房主的取房权尝试](https://nytimes.com/2009/10/25/business/economy/25gret.html)。

^([10](ch06.html#idm45143406630688-marker)) 以下链接被包含是为了保存记录，因为它们代表了本章提供的数据集的原始位置。

^([11](ch06.html#idm45143405919056-marker)) 有关南卡罗来纳州国会选区，请参阅维基百科的页面[*https://en.wikipedia.org/wiki/South_Carolina%27s_congressional_districts#Historical_and_present_district_boundaries*](https://en.wikipedia.org/wiki/South_Carolina%27s_congressional_districts#Historical_and_present_district_boundaries)。

^([12](ch06.html#idm45143405666608-marker)) SBA开始接受所谓“第二轮”PPP贷款的申请，日期为[2021年1月31日](https://uschamber.com/co/run/business-financing/second-draw-ppp-loans)。

^([13](ch06.html#idm45143405581136-marker)) 尝试归因公理总是问题重重，但我更喜欢将这个表达归因为“汉隆剃刀”，见于[“Jargon File”](https://jargon-file.org/archive/jargon-4.4.7.dos.txt)，主要因为该文件解释了许多计算机/编程行话的起源。

^([14](ch06.html#idm45143405576560-marker)) 当然，“为什么”它从未发送是一个有趣的问题。

^([15](ch06.html#idm45143405548048-marker)) 你可能已经注意到，在这章中，我没有按照[第三章](ch03.html#chapter3)中列出的顺序详细讨论我们的数据完整性标准。因为PPP是基于现有的7(a)贷款计划进行[建模](https://journalofaccountancy.com/news/2020/apr/paycheck-protection-program-ppp-loans-sba-details-coronavirus.html)，我做出了（可疑的）假设，即这些数据将被很好地注释。当然，这些也是关于PPP的*唯一*可用数据集，所以我的选择有限（就像在处理现实世界数据时经常发生的情况一样）。

^([16](ch06.html#idm45143405541024-marker)) 我有时发现将数据转置为“把它放到一边”更容易理解。

^([17](ch06.html#idm45143405428640-marker)) 自这章节写作起，此页面已经发生了相当大的变化。值得注意的是，主数据位置现在包括一个“数据字典”——但这仅在数据首次发布几个月后才发布。

^([18](ch06.html#idm45143405416416-marker)) 值得注意的是，在我联系他不久后，此归因被改为`SM`。

^([19](ch06.html#idm45143405275328-marker)) 有关FOIA请求和豁免的更多信息，请参阅[“FOIA/L Requests”](app03.html#foia_requests)，以及在[司法部网站](https://justice.gov/oip/exemption-4-after-supreme-courts-ruling-food-marketing-institute-v-argus-leader-media)上的豁免全文。

^([20](ch06.html#idm45143405217296-marker)) 具体来说，“薪资保护计划：如何计算第一次PPP贷款的最高金额及应提供的业务类型文档”，可以在[*https://sba.gov/sites/default/files/2021-01/PPP%20--%20How%20to%20Calculate%20Maximum%20Loan%20Amounts%20for%20First%20Draw%20PPP%20Loans%20%281.17.2021%29-508.pdf*](https://sba.gov/sites/default/files/2021-01/PPP%20--%20How%20to%20Calculate%20Maximum%20Loan%20Amounts%20for%20First%20Draw%20PPP%20Loans%20%281.17.2021%29-508.pdf)找到。

^([21](ch06.html#idm45143405172768-marker)) 查看[*https://github.com/OpenRefine/OpenRefine/wiki/Clustering-In-Depth*](https://github.com/OpenRefine/OpenRefine/wiki/Clustering-In-Depth)获取更多关于聚类的信息。

^([22](ch06.html#idm45143405153824-marker)) 当你运行这个脚本时，你可能会看到一个关于安装pyICU的警告。然而，安装这个库有点复杂，并且不会改变我们此次练习的结果。不过，如果你打算广泛使用这个指纹识别过程，你可能希望投入额外的时间来设置pyICU。你可以在这里找到关于这个过程的更多信息：[*https://pypi.org/project/PyICU*](https://pypi.org/project/PyICU)。

^([23](ch06.html#idm45143404387856-marker)) 可以从[*https://sba.gov/document/report-paycheck-protection-program-ppp-loan-data-key-aspects*](https://sba.gov/document/report-paycheck-protection-program-ppp-loan-data-key-aspects)下载。

^([24](ch06.html#idm45143404385984-marker)) 再次强调，了解为何以及如何取消这些贷款可能是有益的，但我们在这里找不到这些信息——只有一些关于从哪里开始查找的线索。

^([25](ch06.html#idm45143404363328-marker)) 在涉及美国物业法时，“分母问题”似乎有非常具体的含义，但不用说，这并不是我在这里使用它的方式。

^([26](ch06.html#idm45143404333728-marker)) 你可以在[*https://data.sba.gov/dataset/ppp-foia/resource/aab8e9f9-36d1-42e1-b3ba-e59c79f1d7f0*](https://data.sba.gov/dataset/ppp-foia/resource/aab8e9f9-36d1-42e1-b3ba-e59c79f1d7f0)找到它。
