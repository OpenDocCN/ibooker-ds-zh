# 第一章：介绍实体解析

在全球范围内，大量的数据正在被收集和存储，每天都在增加。这些数据记录了我们生活的世界，以及我们周围的人、地点和事物的变化属性和特征。

在这个全球数据处理的生态系统中，组织独立地收集关于同一现实世界实体的重叠信息集。每个组织都有自己的方法来组织和编目其持有的数据。

公司和机构希望从这些原始数据中获取有价值的洞见。已经开发了先进的分析技术来识别数据中的模式，提取含义，甚至尝试预测未来。这些算法的性能取决于输入的数据质量和丰富程度。通过结合来自多个组织的数据，通常可以创建更丰富、更完整的数据集，从中可以得出更有价值的结论。

本书将指导您如何合并这些异构数据集，创建关于我们生活世界的更丰富的数据集。这个合并数据集的过程被称为各种名字，包括名称匹配、模糊匹配、记录链接、实体协调和实体解析。在本书中，我们将使用术语*实体解析*来描述解析即连接数据的整体过程，这些数据涉及到现实世界中的实体。

# 什么是实体解析？

实体解析是一种关键的分析技术，用于识别指向同一现实世界实体的数据记录。这种匹配过程使得可以在单个来源内去重条目，并在没有公共唯一标识符的情况下连接不同的数据源。

实体解析使企业能够构建丰富和全面的数据资产，揭示关系，并为营销和风险管理目的构建网络。这往往是充分利用机器学习和人工智能潜力的关键前提。

例如，在医疗服务领域，经常需要从不同的实践或历史档案中联合记录。在金融服务中，需要调和客户数据库，以提供最相关的产品和服务或启用欺诈检测。为了增强抗灾能力或提供环境和社会问题的透明度，公司需要将供应链记录与风险情报来源进行联接。

# 为什么需要实体解析？

在日常生活中，作为个体，我们被分配了很多编号——根据我的医疗服务提供者，我通过一个编号进行识别，通过我的雇主又通过另一个编号，再通过我的国家政府，等等。当我注册服务时，通常我的银行、选择的零售商或在线服务提供商会为我分配一个（有时是多个）编号。为什么会有这么多编号？在更简单的时代，当服务是在本地社区提供时，客户是以个人身份认识的，互动是面对面进行的，很明显你知道你在处理谁。交流通常是离散的交易，没有必要参考任何先前的业务，也不需要保留与个别客户关联的记录。

随着越来越多的服务开始远程提供，并在更广泛的区域甚至国家范围内提供，有必要找到一种识别谁是谁的方法。名字显然不够唯一，因此通常将名字与位置结合起来创建一个复合标识符：琼斯夫人变成了来自布罗姆利的琼斯夫人，而不是来自哈罗的琼斯夫人。随着记录从纸质形式迁移到电子形式，分配一个唯一的机器可读编号开始了今天围绕我们的数字和字母数字混合标识符的时代。

在它们各自领域的限制内，这些标识符通常运作良好。我用我的唯一编号来识别自己，很明显我是同一个回头客。这种标识符允许快速建立两方之间的共同语境，并减少误解的可能性。这些标识符通常没有共同之处，在长度和格式上有所不同，并根据不同的方案分配。没有机制可以在它们之间进行转换，或者识别它们单独和集体地指代的是我，而不是另一个个体。

然而，当业务被去人化时，我不认识我正在交易的人，他们也不认识我，如果我多次注册相同的服务会发生什么？也许我忘记了用我的唯一编号进行标识，或者有人代表我提交了新的申请。将创建第二个也能标识我身份的编号。这种重复使得服务提供者更难以提供个性化服务，因为他们现在必须合并两个不同的记录才能充分了解我是谁以及我的需求是什么。

在较大的组织中，匹配客户记录的问题变得更加具有挑战性。不同的功能或业务线可能会维护适合其目的的记录，但这些记录是独立设计的。 一个常见的问题是如何构建客户的综合（或 360 度）视图。客户可能在多年来与组织的不同部分进行了交互。他们可能在不同的上下文中进行交互——作为个人，作为联合家庭的一部分，或者可能是与公司或其他法律实体相关联的官方能力。在这些不同的交互过程中，同一个人可能在各种系统中被分配了多个标识符。

这种情况通常是由于（经常是历史性的）合并和收购而引起的，其中要将重叠的客户集合合并并一致地对待为一个整体人口。我们如何将一个领域的客户与另一个领域的客户进行匹配？

当将由不同组织提供的数据集合并在一起时，也会出现记录合并的挑战。由于通常不存在广泛采用的标准或个体之间的公共键，特别是与个人相关的键，因此合并它们的数据通常被忽视并且不是一项微不足道的任务。

# 实体解析的主要挑战

如果我们分配的唯一标识符都不同且无法匹配，我们如何确定两个记录指的是同一个实体？我们最好的方法是比较这些实体的各个属性，例如他们的名称，如果它们有足够多的相似之处，就做出我们最好的判断，即它们是匹配的。这听起来足够简单，对吧？让我们深入了解一些为什么这并不像听起来那么简单的原因。

## 名称的缺乏唯一性

首先，存在着识别名称或标签之间的唯一性的挑战。将相同的名称重复分配给不同的现实实体明显存在一个难题，即区分谁是谁。也许你在互联网上搜索过自己的名字。除非你的名字特别不常见，否则你很可能会发现有很多与你完全相同的同名者。

## 命名规范不一致

名称以各种方式和数据结构记录。有时名称会完整描述，但通常会出现缩写或省略名称的不太重要的部分。例如，我的名字可能像表 1-1 中的任何一个变体一样完全正确地表达。

表 1-1. 名称变体

| 名称 |
| --- |
| 迈克尔·谢拉 |
| 迈克尔·威廉·谢拉 |
| 迈克尔·威廉·罗伯特·谢拉 |
| 迈克尔·W·R·谢拉 |
| M W R 谢拉 |
| M W 谢拉 |

这些名字互不完全匹配，但都指向同一个人，同一个现实世界的实体。头衔、昵称、缩写形式或重音字符都会使找到精确匹配的过程受挫。复姓或带连字符的姓氏会进一步增加变数。

在国际背景下，命名惯例在全球范围内差异巨大。个人姓名可能出现在名字的开头或结尾，而姓氏可能有也可能没有。姓氏也可能根据个体的性别和婚姻状况而异。姓名可能用各种字母表/字符集写成，或在不同语言之间翻译得不同。¹

## 数据捕获的不一致性

捕捉和记录名字或标签的过程通常反映了获取者的数据标准。在最基本的层次上，一些数据获取过程将仅使用大写字母，其他人则使用小写字母，而许多人则允许混合大小写，其中首字母大写。

名字可能仅在对话中听到，没有机会澄清正确的拼写，或者可能在匆忙中被错误地转录。在手动重新键入过程中，名字或标签经常会被误输入或者意外省略。有时，如果原始上下文丢失，可能会使用不同的约定，这些约定很容易被误解。例如，即使是一个简单的名字，也可能被记录为“名字，姓氏”，或者“姓氏，名字”，甚至完全错误地转置到错误的字段中。

国际数据捕获可能导致不同脚本之间的音译不一致，或在口头捕获时出现转录错误。

## 工作示例

让我们考虑一个简单的虚构例子，来说明这些挑战可能如何显现。首先，想象我们唯一拥有的信息是如 表 1-2 中所示的名字。

表 1-2\. 示例记录

| **名称  ** |
| --- |
| 迈克尔·谢拉 |
| 迈克尔·威廉·谢拉 |

“迈克尔·谢拉”和“迈克尔·威廉·谢拉”是否指的是同一个实体？在没有其他信息的情况下，两者很可能指的是同一个人。第二个名字增加了一个中间名，但除此之外它们几乎相同，比较两个姓氏将产生完全匹配。请注意，我偷偷加了一个常见的拼写错误。你发现了吗？

如果我们再增加一个属性，能帮助提高匹配准确性吗？如果你记不住会员号码，服务提供商通常会要求提供出生日期来帮助识别您（出于安全考虑）。出生日期是一个特别有用的属性，因为它不会改变，并且具有大量潜在值（称为*高基数*）。此外，日期的组合结构，包括日、月和年的个体值，可能会在确立精确等价关系时为我们提供线索。例如，参考表 1-3。

表 1-3\. 示例记录—2

| **姓名    ** | **出生日期 ** |
| --- | --- |
| 迈克尔·谢勒   | 1970 年 1 月 4 日    |
| 迈克尔·威廉·谢勒   | 1970 年 1 月 14 日   |

乍一看，两个记录的出生日期并不相等，因此我们可能会认为它们不匹配。如果这两个人的出生日期相差 10 天，他们不太可能是同一个人！然而，这两者之间只有个位数的差异，前者在日期子字段中缺少前导数字 1——这可能是打字错误吗？很难说。如果这些记录来自不同的来源，我们还需要考虑数据格式是否一致——是英国的 DD/MM/YYYY 格式还是美国的 MM/DD/YYYY 格式？

如果我们增加了出生地点呢？虽然这个属性不应改变，但可以用不同级别的细化或不同的标点符号来表达。表 1-4 展示了增强记录。

表 1-4\. 示例记录—3

| **姓名    ** | **出生日期 ** | **出生地点   ** |
| --- | --- | --- |
| 迈克尔·谢勒   | 1970 年 1 月 4 日    | 斯托·奥恩·瓦尔德 |
| 迈克尔·威廉·谢勒   | 1970 年 1 月 14 日   | 斯托·奥恩·瓦尔德 |

这里没有任何一个记录的出生地点完全匹配，尽管两者都可能属实。

因此，出生地点，可能以不同的精确级别记录，并不能像我们之前想象的那样帮助我们。那么像手机号码这样更私人化的信息呢？当然，我们中的许多人在一生中会更换电话号码，但是如果能够在换供应商时保留一部受喜爱和社交广泛的手机号码，这个号码就成为一个更具粘性的属性，我们可以使用它。然而，即使在这里，我们也面临挑战。个人可能拥有多个号码（例如工作和个人号码），或者标识符可能以各种格式记录，包括空格或连字符。它可能包含或不包含国际拨号前缀。

表 1-5 展示了我们的完整记录。

表 1-5\. 示例记录—4

| **姓名    ** | **出生日期 ** | **出生地点   ** | **手机号码** |
| --- | --- | --- | --- |
| 迈克尔·谢勒   | 1970 年 1 月 4 日    | 斯托·奥恩·瓦尔德 | 07700 900999 |
| 迈克尔·威廉·谢勒   | 1970 年 1 月 14 日   | 斯托·奥恩·瓦尔德 | 0770-090-0999 |

正如您所见，这个解析挑战很快就变得非常复杂。

## 故意模糊化

大多数导致匹配过程中数据不一致的情况，都是通过粗心但出于善意的数据捕获过程引起的。然而，对于某些用途，我们必须考虑数据被恶意混淆的情况，以掩盖实体的真实身份，并防止可能揭示犯罪意图或关联的关联。

## 匹配排列

如果我让你将你的名字与一个简单的表格，比如说 30 个名字的表格，进行匹配，你可能可以在几秒钟内完成。一个更长的列表可能需要几分钟，但这仍然是一个实际的任务。然而，如果我要求你将一个包含 100 个名字的列表与另一个包含 100 个名字的列表进行比较，这个任务就变得更加繁琐和容易出错了。

不仅潜在匹配数量增加到 10,000（100 × 100），而且如果您想在第二个表中一次通过这样做，您必须将第一个表中的所有 100 个名称都记在脑子里——这并不容易！

同样，如果我让你在一个列表中对 100 个名字进行去重，你实际上需要进行比较：

1.  第一个名字与剩余的 99 个名字，然后

1.  第二个名字与剩余的 98 个名字等等。

实际上，您需要进行 4,950 次比较。以每秒一次的速度计算，仅仅对两个短列表进行比较就需要大约 80 分钟的工作时间。对于更大的数据集，潜在的组合数量变得不切实际，即使对于性能最佳的硬件也是如此。

## 盲匹配？

到目前为止，我们假设我们寻求匹配的数据集对我们是完全透明的——即属性的值是 readily available 的，完整的，并且没有以任何方式被模糊或掩盖。在某些情况下，由于隐私约束或地缘政治因素阻止数据跨越国界移动，这种理想情况是不可能的。如何在看不到数据的情况下找到匹配项？这看起来像魔术，但正如我们将在第十章中看到的那样，有加密技术可以使匹配仍然发生，而不需要完全暴露要匹配的列表。

# 实体解析过程

为了克服上述挑战，基本的实体解析过程被分为四个连续步骤：

1.  数据标准化

1.  记录阻塞

1.  属性比较

1.  匹配分类

在匹配分类之后，可能需要进行额外的后处理步骤：

+   聚类

+   规范化

让我们依次简要描述每一个步骤。

## 数据标准化

在我们比较记录之前，我们需要确保我们有一致的数据结构，以便我们可以测试属性之间的等价性。我们还需要确保这些属性的格式一致。这个处理步骤通常涉及到字段的拆分，删除空值和多余字符。它通常是针对源数据集定制的。

## 记录阻塞

为了克服记录比较的数量不切实际高的挑战，通常会使用一种称为*阻塞*的过程。该过程不是将每个记录与每个其他记录进行比较，而是仅对根据某些属性之间的就绪等价性预先选择的记录对进行全面比较。这种过滤方法集中了解析过程在那些最有可能匹配的记录上。

## 属性比较

接下来是通过阻塞过程选择的记录对之间比较各个属性的过程。等价度可以根据属性之间的精确匹配或相似性函数来确定。该过程产生了两个记录对之间的等价度量集合。

## 匹配分类

基本实体解析过程的最后一步是确定个体属性之间的集体相似性是否足以声明两个记录匹配，即解析它们是否指向同一现实世界的实体。这种判断可以根据一组手动定义的规则进行，也可以基于机器学习的概率方法。

## 聚类

一旦我们的匹配分类完成，我们可以通过它们的匹配对将记录分组为连接的群集。将记录对包含在群集中可能是通过额外的匹配置信度阈值来确定的。未达到此阈值的记录将形成独立的群集。如果我们的匹配标准允许不同的等价标准，则我们的群集可能是不传递的；即记录 A 可能与记录 B 配对，记录 B 可能与记录 C 配对，但记录 C 可能无法与记录 A 配对。因此，群集可能高度相互关联或松散耦合。

## 规范化

解析后可能需要确定应使用哪些属性值来表示实体。如果使用近似匹配技术确定了等价性，或者如果一对或群集中存在但未在匹配过程中使用的附加可变属性，则可能需要决定哪个值最具代表性。然后，生成的规范属性值用于后续计算中描述解析的实体。

## 工作示例

回到我们简单的示例，让我们将这些步骤应用到我们的数据上。首先，让我们标准化我们的数据，分割名字属性，标准化出生日期，并删除出生地点和手机号码字段中的额外字符。表 1-6 显示了我们经过清理的记录。

表 1-6\. 第 1 步：数据标准化记录

| **名字  ** | **姓氏** | **出生日期 ** | **出生地点   ** | **手机号码** |
| --- | --- | --- | --- | --- |
| 迈克尔   | 谢拉 | 1970 年 1 月 4 日    | 斯托·翁·沃尔德 | 07700 900999 |
| 迈克尔   | 谢拉 | 1970 年 1 月 14 日   | 斯托·翁·沃尔德 | 07700 900999 |

在这个简单的例子中，我们只需要考虑一个配对，因此不需要应用阻塞技术。我们将在第五章中讨论这个问题。

接下来，我们将比较每个属性的精确匹配情况。表格 1-7 显示了每个属性的比较结果，可以是“匹配”或“无匹配”。

表格 1-7\. 第三步：属性比较

| **属性** | **值记录 1** | **值记录 2** | **比较结果** |
| --- | --- | --- | --- |
| 名字 | 迈克尔  | 米迦勒  | 无匹配 |
| 姓氏 | 谢勒 | 谢勒 | 匹配 |
| 出生日期 | 1970 年 1 月 4 日 | 1970 年 1 月 14 日 | 无匹配 |
| 出生地 | 斯托-瓦尔德 | 斯托-瓦尔德 | 匹配 |
| 手机号码 | 07700 900999 | 07700 900999 | 匹配 |

最后，我们应用第 4 步确定是否存在总体匹配。一个简单的规则可能是，如果大多数属性匹配，则我们得出总体记录匹配的结论，就像在这种情况下一样。

或者，我们可以考虑各种匹配属性的组合是否足以声明匹配。在我们的例子中，为了声明匹配，我们可以寻找以下任一条件：

+   姓名匹配和（出生日期或出生地匹配），或

+   姓名匹配和手机号匹配

我们可以进一步采取这种方法，并为我们的每个属性比较分配一个*相对权重*；例如，手机号码匹配可能比出生日期匹配的价值高出两倍，等等。结合这些加权分数产生一个总体匹配分数，可以根据给定的置信度阈值来考虑。

我们将更多地研究不同方法来确定这些相对权重，使用统计技术和机器学习，在第四章中。

正如我们所见，不同的属性在帮助我们确定是否存在匹配时可能具有不同的强度。之前，我们考虑了在找到一个相当常见的名字与找到一个较少见的名字之间找到匹配的可能性。例如，在英国的情况下，史密斯姓的匹配可能比谢勒姓的信息量要少—谢勒姓的人比史密斯姓的人少，因此匹配本身的可能性从一开始就较低（较低的先验概率）。

这种概率方法在某些分类属性的值（即有限值集合的属性）中特别有效，其中某些值比其他值更常见。如果我们考虑一个城市属性作为英国数据集中地址匹配的一部分，那么伦敦出现的频率可能远远高于巴斯，因此可能会受到较少的加权。

请注意，我们尚未能够确定哪个出生日期是确切正确的，因此我们面临一个规范化的挑战。

# 衡量性能

统计方法可能帮助我们决定如何评估和结合比较各个属性所提供的所有线索，但我们如何决定组合是否足够好？如何设置置信度阈值来声明匹配？这取决于我们重视什么以及我们打算如何使用我们新发现的匹配。

我们更关心确保发现每一个潜在的匹配，如果在这个过程中声明了一些后来被证明是错误的匹配，我们也能接受吗？这个度量称为*召回率*。或者，我们不想浪费时间在不正确的匹配上，但如果在此过程中错过了一些真实的匹配，我们可以接受。这称为*精确度*。

比较两条记录时，可能出现四种不同的情况。表 1-8 列出了匹配决策和实际情况的不同组合。

表 1-8. 匹配分类

| **你决定** | **实际情况** | **实例** |
| --- | --- | --- |
| 匹配 | 匹配 | 真正阳性 (TP) |
| 匹配 | 不匹配 | 假阳性 (FP) |
| 不匹配 | 匹配 | 假阴性 (FN) |
| 不匹配 | 不匹配 | 真负 (TN) |

如果我们的召回率测量很高，那么我们只宣布相对较少的假阴性，即当我们声明匹配时，我们很少会错过一个好的候选。如果我们的精确度很高，那么当我们声明匹配时，我们几乎总是做对的。

在一个极端情况下，假设我们声明每一个候选对都是匹配的；我们将没有任何假阴性，我们的召回率度量将是完美的（1.0）；我们永远不会漏掉一个匹配。当然，我们的精确度将非常低，因为我们会错误地声明大量不匹配为匹配。或者，想象在理想情况下，当每个属性完全等效时，我们才宣布匹配；那么我们将永远不会错误地宣布匹配，我们的精确度将是完美的（1.0），但代价是我们的召回率将非常低，因为很多好的匹配都会错过。

理想情况下，我们当然希望同时具备高召回率和精确度——我们的匹配既正确又全面——但这很难实现！第六章 更详细地描述了这个过程。

# 入门指南

那么，我们如何解决这些挑战呢？

希望本章为您提供了对实体解析是什么，为什么需要它以及过程中的主要步骤的良好理解。接下来的章节将通过一组基于公开数据的实际工作示例，手把手地指导您。

幸运的是，除了商业选项外，还有几个开源的 Python 库可以为我们做大部分的繁重工作。这些框架为我们构建适合我们数据和背景的定制匹配过程提供了支持。

在我们开始之前，我们将在下一章节中进行一个小的偏离，来设置我们的分析环境，并回顾我们将使用的一些基础 Python 数据科学库，然后我们将考虑我们实体解析过程的第一步——准备我们的数据以便匹配。

¹ 关于全局命名惯例的详细信息，请参阅[此指南](https://oreil.ly/Hzu6D)。
