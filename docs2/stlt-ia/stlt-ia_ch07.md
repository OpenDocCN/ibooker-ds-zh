# 第八章：8 使用 Streamlit 构建 CRUD 应用程序

### 本章涵盖

+   为持久存储设置关系型数据库

+   使用 SQL 执行 CRUD 操作

+   开发多页 Streamlit 应用程序

+   在 Streamlit 应用程序中创建共享数据库连接

+   用户身份验证

1957 年，一位科幻小说作家 Theodore Sturgeon 著名地说，“百分之九十的*一切*都是 CRUD”。虽然这最初是对科幻小说体裁的一种愤世嫉俗的辩护——其观点是，在这方面它与任何其他事物没有区别——但这个谚语后来获得了不同的含义，成为软件工程中最不为人知的秘密：百分之九十的一切都是 CRUD。

通过 CRUD，我指的是创建、读取、更新和删除，这四种平凡操作几乎出现在任何著名的软件中。

想想看。像 Facebook 这样的社交媒体平台围绕着创建帖子、阅读动态、更新个人资料和删除内容。电子商务网站通过类似操作管理产品、订单、客户账户和评论。即使是 Windows 上的简单记事本也围绕着创建、读取、更新和删除文本文件。

精通 CRUD 操作对于在软件设计方面打下坚实基础至关重要，因为它们的实现通常涉及解决非平凡挑战。在本章中，我们将使用 Streamlit 创建一个 CRUD 应用程序，从头开始实现这些操作，同时涵盖相关主题，如用户身份验证。

## 8.1 Haiku Haven：Streamlit 中的 CRUD 应用程序

对于我们的 CRUD 探险，我们将选择日本艺术中的*俳句*——具体来说，我们将创建一个网站，让用户能够编写和分享他们自己的俳句。

对于不太熟悉日本文学的人来说，俳句是一种遵循某些规则的短三行诗：第一行和第三行必须包含五个音节，而第二行必须包含七个音节。例如，这是我关于 Streamlit 写的一首俳句：

> *这么多网络应用！*
> 
> *用 Python 我能做吗？*
> 
> *然后我尝试了 Streamlit。*

我知道*，*对吧？我有时也怀疑自己是否错过了自己的使命。不管怎样，你会发现这首诗遵循了我上面提到的 5-7-5 音节规则，因此是一个有效的俳句。

Haiku Haven 将是一个初学者诗人可以创作、精炼和管理俳句的地方。它将使用户能够*从头开始创建*俳句，*阅读*他们所创作的俳句，*更新*他们已经创建的俳句，如果他们决定它不符合标准，则*删除*它。

### 8.1.1 陈述概念和需求

如往常一样，我们将首先简洁地陈述我们应用程序的概念：

##### 概念

Haiku Haven，一个允许用户创建、编辑和管理俳句的网站

这个概念以及我们之前关于 CRUD 的讨论应该给你一个基本的概念，但让我们明确具体的需求，以便我们对 Haiku Haven 的愿景达成共识。

#### 需求

Haiku Haven 的用户应该能够：

+   使用用户名和密码创建和登录账户

+   在他们的用户名下创建 haiku

+   查看他们创建的 haiku

+   更新 haiku

+   删除他们的 haiku

希望你能意识到你经常遇到这类应用。如果你在需求中将“haiku”一词替换为“image”，你将得到 Instagram 的一个基本版本。用“task”替换可以得到像 Asana 这样的生产力工具，用“post”替换可以得到 Twitter 或 WordPress。

重点在于，这些需求不仅仅是关于 haiku 的——它们代表了软件设计中的一个通用模式。几乎每个应用程序都围绕管理某种数据展开，这些数据基于 CRUD 操作。

通过构建 Haiku Haven，你不仅为诗歌爱好者创建了一个有趣的应用程序；你还在学习如何构建现代软件的基本工作流程。你将处理用户身份验证、数据存储和检索——这些技能适用于你未来可能创建的几乎任何应用程序或系统。

#### 超出范围的内容

我们可以将很多功能集成到 Haiku Haven 中（想想你可以在 Twitter 上做的一切），但我们只有一个章节，所以我们将专注于绝对核心的内容。这意味着我们 *不会* 关注：

+   使 haiku 对其他用户可见并可搜索——haiku 对作者来说是私有的

+   “社交”功能，如点赞、评论和分享

+   辅助功能，如分页

+   高级安全功能（尽管我们将确保基础正确）

### 8.1.2 可视化用户体验

Haiku Haven 将是我们的第一个多页面应用。我们需要制作多个体验或流程——账户创建流程、登录和登出，以及实际的 CRUD 部分（创建、读取、更新和删除 haiku）。

图 8.1 尝试勾勒出我们应用的不同部分可能的样子。

![image](img/ch08__image001.png)

##### 图 8.1 我们希望在 Haiku Haven 中出现的页面的粗略草图

由于这些流程在各种常见应用中的普遍性，我不会在这里占用太多空间来详细解释它们，但这里有一些亮点：

+   登录页面使用密码进行身份验证，你可以在注册页面设置

+   一个“我的 haiku”页面代表了应用内的登录体验，让用户可以创建、查看、编辑或删除他们的 haiku。

+   此外，还有一个 haiku 编辑页面，在这里进行 haiku 的创作。

### 8.1.3 实施构思

由于 Haiku Haven 代表 CRUD 网络应用——因此，根据斯特金定律，90% 的 *所有* 网络应用——它的实现应该涉及一些非常常见的模式。

事实上，我们将使用的设计由三个在大多数在线应用中都能看到的组件组成：前端、后端和 *数据库*。图 8.2 展示了这种方法。

![image](img/ch08__image002.png)

##### 图 8.2 显示前端、后端和数据库的应用设计

前端，正如我们在先前的应用程序中看到的，由用户与之交互的小部件组成。每个主要操作，如创建账户或更新俳句，都会在后台调用相应的函数。

我们将把后端可用的功能分为两组：一组包括与用户相关的操作，如创建账户或验证用户，另一组包括与俳句相关的操作——创建、读取、更新或删除俳句。

这里有趣的部分——对我们来说是新的——是数据库，它用于在**表格**中永久存储与用户和俳句相关的信息。正如我们很快就会看到的，数据库也使得检索我们存储的信息变得容易。

虽然我们的应用程序故意很简单，但它捕捉了大多数 Web 应用程序的核心要素：用于用户交互的前端、用于处理请求的后端以及用于存储和检索数据的数据库。这三个支柱无缝协作，形成了无数应用程序的基础，无论是简单还是复杂。

## 8.2 设置持久化存储

我们迄今为止创建的应用程序中存在的一个最重要的缺陷是缺乏持久化存储。本质上，在我们迄今为止的所有应用程序中，如果用户关闭浏览器窗口，他们就会丢失他们的数据和进度。这对俳句天堂来说完全不行；我们需要用户能够在将来某个时候注销并再次登录后保存和访问他们的俳句——我们需要将数据存储在**应用程序本身之外**。

解决数据存储问题有几种不同的方法，但我们将使用一种相当常见的技术：关系型数据库。

### 8.2.1 关系型数据库概念

**关系型数据库**是一种数据存储系统，它将数据组织成结构化的**表格**，其中每个表格由**行**（也称为**记录**）和**列**（也称为**字段**）组成。一行代表某种类型的条目或实体，一列是实体的属性。**模式**定义了这些表格的结构，包括每列的**数据类型**以及不同表格之间的**关系**。

关系型数据库依赖于一种称为**结构化查询语言**（SQL）的语言来创建、管理和**查询**表。

如果这些内容让你感到熟悉，那很可能是因为我们已经在处理这类事情一段时间了。在第六章和第七章中，我们与 Pandas 数据框一起工作，这些数据框也处理表格数据——除了数据框在程序运行时存储在内存中，而数据库用于**持久**存储，即即使在程序运行完成后仍然存在的存储。

我们在第七章中也简要介绍了 SQL，当时我们使用它来检索我们存储在 Google BigQuery 中的销售数据的行。确实，BigQuery 通常被认为是一种关系型数据库，尽管它与我们本章将使用的关系型数据库不同。

### 8.2.2 俳句天堂的数据模型

为了更好地理解这一切，让我们尝试弄清楚我们如何在一个关系型数据库中**建模**Haiku Haven 的数据。

广义而言，为应用建模数据包括以下步骤：

+   确定应用中涉及的**实体**

+   定义这些实体之间的**关系**

+   列出每个实体的**属性**

+   将实体、属性和关系转换为关系数据库模式

#### 确定实体

通常来说，识别应用中涉及的实体的一种好方法是列出所有代表应用核心概念的**名词**。例如，如果你考虑 Twitter，以下所有这些都可能被认为是实体：用户、推文、转发、直接消息、提及、关注者、标签等。

Haiku Haven 当然要简单得多。我们可以相当容易地确定我们的应用需要处理的两个关键实体：**俳句**和**用户**。

#### 定义实体之间的关系

任何两个实体之间的关系应该根据它们之间可能交互的**性质**和**基数**来定义。用英语来说，这意味着你应该说明一个实体是如何与另一个实体相关的，以及每种实体在这一关系中每一边可以有多少个。

例如，俳句和用户是相关的，因为一个用户**可以写**俳句（我们上面提到的“性质”）。同时，一个用户可以写很多俳句，而一个特定的俳句只能由一个用户写。因此，用户和俳句之间的关系是**一对一**或 1:n（“基数”）。

#### 列出每个实体的属性

实体的属性是描述它的字段。在我们的例子中：

+   一个用户有一个**用户名**和**密码**。在现实生活中，我们可能还想捕捉用户的姓名或用户账户创建的时间，但让我们保持简单。

+   俳句有其**文本**和**作者**（碰巧是用户）。我们还应该给每个俳句一个**数字 ID**以便于引用。俳句的**创建时间**在应用中显示可能很重要，所以我们也考虑这一点。

所有这些都可以用一个实体-关系图（或 ER 图）表示，如图 8.3 所示。

![image](img/ch08__image003.png)

##### 图 8.3 实体-关系（ER）图，展示了用户和俳句实体

当然，现实世界中的应用程序的 ER 图通常比这复杂得多，但我希望这有助于说明这个概念。

#### 将所有内容转换为数据库模式

绘制 ER 图来构思数据模型是有帮助的，但我们真正想要的最终结果是一个可以在我们的关系型数据库中使用的模式。

将实体、属性和关系转换为数据库中的表没有固定的规则，但*一般来说*：实体成为表，属性成为列，一对一关系成为*外键*（稍后将有更多介绍），多对多关系成为它们自己的表。

![image](img/ch08__image004.png)

##### 图 8.4 用户和俳句表之间的外键关系数据库模式

在我们的案例中，如图 8.4 所示，我们将有两个表，`users`和`haikus`，它们具有我们之前讨论的属性作为列。`users`表中的每一行代表一个单个用户，而`haikus`表中的每一行代表一个单个俳句。此外，每个表都有一个*主键*，这是一个可以用来唯一标识表中任何行的字段。对于`users`，主键是`username`字段（这很有道理，因为每个用户都有一个用户名，且没有两个用户可以有相同的用户名）。对于`haikus`，它是`haiku_id`。

用户和俳句之间的关系反映在俳句表中的`author`列中，该列包含必须出现在用户表中的用户名。这样的列（`author`）被称为*外键*，因为它指向另一个（“外部”）表的主键（`username`）。

### 8.2.3 PostgreSQL：一个真正的关系数据库

足够的理论了！现在让我们动手操作一个真实的数据库。在本章中，我们将使用 PostgreSQL（发音为“post-gress-cue-ell”），这是行业中最古老、最稳健、最受欢迎的数据库之一。

##### 注意

在第七章中我们遇到的 Google BigQuery 也可以被认为是一个关系数据库（尽管它更像是基于云的数据仓库）。虽然 BigQuery 优化用于分析用例（如查询大量数据以生成报告或发现趋势），但 PostgreSQL 更适合事务用例，例如处理对单个记录的频繁小更新，并在并发操作中维护数据一致性。

#### 安装 PostgreSQL

在本章的末尾，当我们部署俳句天堂到生产环境时，我们将使用免费云服务来设置 PostgreSQL。然而，对于本地开发，我们首先需要一个本地安装。

要安装 PostgreSQL，从`https://www.postgresql.org/download/`下载适用于您操作系统的安装程序，并按照屏幕上的说明运行。对于大多数选项，您可以接受默认设置。在安装过程中，您将被提示为数据库超级用户设置密码。请确保记住这个密码——您将在下一节需要它。

安装完成后，你应该可以访问 PostgreSQL 命令行工具，称为 `psql`，它位于安装文件夹的`bin/`目录中。如果你保留了默认选项，这通常会是：

+   macOS 上的`/Library/PostgreSQL/17/bin/psql`

+   Windows 上的`C:\Program Files\PostgreSQL\17\bin\psql`

为了简化访问，您应该配置系统，以便您可以直接从终端运行 `psql` 命令。这需要将 `bin/` 目录添加到系统环境变量中。您可能已经为 `streamlit` 命令做了类似的事情，详细步骤可以在附录 A（第 A.4 节）中找到。

如果由于任何原因无法将路径添加到环境变量中，您仍然可以通过输入其完整路径而不是仅命令来使用 `psql`。例如：

+   macOS: `/Library/PostgreSQL/17/bin/psql`

+   Windows: `"C:\Program Files\PostgreSQL\17\bin\psql"`（包括引号）

在本章的其余部分，如果需要，当您看到 `psql` 时，请替换为完整路径。

#### 为 Haiku Haven 创建数据库

一旦您设置了 `psql` 命令，通过运行以下命令登录到您的本地 PostgreSQL 实例：

```py
psql -U postgres -d postgres
```

当提示输入密码时，请输入您在安装过程中配置的密码。此命令以默认用户（`-U`）`postgres` 登录 PostgreSQL——这是一个可以随意做任何事的行政用户。`-d` 指定您想要连接到默认数据库，而这个默认数据库——如果按照我的理解来说——也被称为 `postgres`。现在您应该能看到类似以下的 `psql` 提示符：

```py
postgres=#
```

当您的应用程序与 PostgreSQL 通信时，您不希望它以广泛的行政权限运行——那将是一场安全噩梦。相反，让我们创建一个范围更窄的用户。在 `psql` 提示符中输入以下内容，并用您选择的密码替换引号内的字符串：

```py
CREATE USER haiku_lord WITH PASSWORD '<Pick a password you like>';
```

显然，您可以使用您喜欢的任何用户名，但我会假设我们使用 `haiku_lord`。如果这成功了（别忘了结尾的分号！），您应该会得到一条只说 `CREATE ROLE` 的输出行。

在您可以在 PostgreSQL 中创建表之前，您首先需要创建一个**数据库**（在这里您可以将其视为表的容器）。您刚刚创建的 `haiku_lord` 用户还不能做这件事，所以输入以下命令来允许它：

```py
ALTER USER haiku_lord CREATEDB;
```

现在我们有一个具有适当权限的 Haiku Haven 特定用户，我们就完成了默认的 `postgres` 用户，所以通过输入 `exit` 退出 `psql` 壳，然后按如下方式重新运行它：

```py
psql -U haiku_lord -d postgres
```

假设您在提示时输入了为 `haiku_db` 选定的密码，您再次连接到了 `postgres` 数据库，但现在您是以 `haiku_lord` 的身份操作的（如果您愿意，可以通过输入 `SELECT current_user;` 来验证这一点）。

要创建一个名为 `haikudb` 的数据库来存储 Haiku Haven 的表，请输入：

```py
CREATE DATABASE haikudb;
```

您可以通过输入 `\l` 来列出您本地 PostgreSQL 实例中可用的所有数据库。现在您应该能够看到其中包含 `haikudb`、`postgres` 以及几个其他数据库。

要开始使用我们的数据库，我们需要“连接”到它。为此，请输入：

```py
\c haikudb;
```

一个确认消息——“您现在已连接到数据库 "haikudb" 作为用户 "haiku_lord"”——应该让您知道这成功了。

保持这个终端窗口中的`psql` shell 打开。我们将在本章的整个过程中不断回到它。如果您最终关闭了它，您可以通过输入以下命令恢复到这个状态：

```py
psql -U haiku_lord -d haikudb
```

### 8.2.4 SQL 入门课程

PostgreSQL 中的 SQL 代表结构化查询语言。它被不同地发音为"sequel"和"ess-cue-ell"，SQL 是数据库的语言，用于创建和更新表，最重要的是，用于*查询*以获取我们确切需要的数据。如果您是开发者或在任何形式的数据领域工作，SQL 也是您应该知道的最受欢迎和最有用的语言之一。

如果您不知道 SQL，今天真是您的幸运日，因为我们将在这个部分快速浏览基础知识。

#### 创建表

在本章前面，我们提出了 Haiku Haven 的数据库模式，其中包含两个表：`users`和`haikus`。

我们现在将在 PostgreSQL 中实际创建这些表。回到您连接到`haikudb`数据库的`psql` shell（或如之前所示重新运行`psql`），并输入以下 SQL 命令：

```py
CREATE TABLE users (
  username VARCHAR(100) PRIMARY KEY,
  password_hash VARCHAR(128)
);
```

`CREATE TABLE`命令创建了一个具有特定模式的表。在上面的例子中，表名为`users`，它有两个列：`username`和`password_hash`——如果您还记得，这是我们之前确定的重要用户属性字段。

为什么它说`password_hash`而不是仅仅`password`？请稍等，我将在本章稍后解释这一点。现在，我们只需将其视为用户的密码。

将您的注意力转向我们定义列的行：

```py
username VARCHAR(100) PRIMARY KEY,
password_hash VARCHAR(128)
```

这些序列，由逗号分隔，是*列指定*，允许您配置每一列——包括指定数据类型。

`VARCHAR`是 PostgreSQL 中的一种数据类型；它是一种可以具有可变字符数的字符串类型。`VARCHAR(100)`表示该列可以包含最多 100 个字符。如果您尝试存储超过 100 个字符，PostgreSQL 将抛出一个错误。

您还会注意到`username`列旁边的`PRIMARY KEY`。这表示`username`列将被用来唯一标识用户表中的一行。这还意味着`users`表中的每一行都必须有一个`username`，并且只有一个用户可以有一个特定的`username`。

要检查是否成功，您可以使用`psql`命令`\dt`，它列出当前数据库中的所有表。现在这样做应该会给出：

```py
          List of relations
 Schema | Name  | Type  |   Owner
--------+-------+-------+------------
 public | users | table | haiku_lord
(1 row)
```

接下来，让我们使用另一个`CREATE TABLE`命令创建我们的第二个表`haikus`：

```py
CREATE TABLE haikus (
  haiku_id SERIAL PRIMARY KEY,
  created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
  author VARCHAR(100),
  text TEXT,
  FOREIGN KEY (author) REFERENCES users(username)
);
```

这里有一些新内容：

+   `haiku_id`，主键，数据类型为`SERIAL`。这意味着 PostgreSQL 将自动为该列提供一个自增整数值。第一行插入将具有`1`作为其`haiku_id`，第二行将具有`2`，依此类推。

+   `created_at` 的数据类型为 `TIMESTAMPTZ`，这是一个带有时区信息的日期时间戳。`DEFAULT` 关键字指定了在向表中插入行时如果没有明确提供值，则放入此列的值。在这种情况下，我们希望提供当前的时间戳 (`CURRENT_TIMESTAMP`) 作为此默认值。

+   `text` 的数据类型为 `TEXT`，类似于 `VARCHAR`，但没有最大字符长度限制。这对于俳句的实际内容来说是有意义的。

+   最后一行，`FOREIGN KEY (author) REFERENCES users(username)` 表示作者列必须有一个值，该值存在于用户表中的某些行的 `username` 列中。这被称为 *外键约束*。

#### 向表中插入行

接下来，让我们使用 SQL 的 `INSERT INTO` 语句向这些表中插入行。

我们首先通过向 `users` 表中添加一行来创建一个新用户：

```py
INSERT INTO users (username, password_hash) VALUES ('alice', 'Pass_word&34');
```

`INSERT INTO` 命令通过指定列值将行插入到表中。表名 `users` 后的 `(username, password_hash)` 表示我们即将指定的列值对应于 `username` 和 `password_hash` 列，顺序如下。

`VALUES` 后的部分给出了要输入的实际值。这里的效果是 `users` 现在有一行——`username` 为 `alice`，`password_hash` 设置为 `Pass_word&34`。

```py
INSERT INTO haikus (author, text) VALUES
('alice', E'Two foxes leap high\nOne lands safely on the earth\nWhere is the other?');
```

注意，尽管 `haikus` 有四个列，但我们只指定了其中两个的值——`author` 和 `text`。PostgreSQL 会自动提供 `haiku_id` 以及 `created_at` 的默认值。

还要注意俳句开头引号前的 `E`。这标志着值是一个 *转义字符串*，正确地将 `\n` 转义序列转换为换行符。

在我们查询这些表之前，为了以防万一，让我们在 `alice` 下再添加一首令人不安的俳句：

```py
INSERT INTO haikus (author, text) VALUES
('alice', E'Five frogs are jumping\nFour come down as expected\nBut one goes missing.');
```

#### 查询表

SQL 的真正优势在于我们可以在表被行填充后以多种灵活的方式 *查询* 表（从它们中读取数据）。

要检索数据，我们使用 `SELECT` 语句。此命令允许我们指定我们想要看到的列，并根据某些标准过滤行。让我们从一个简单的查询开始，以检索 `users` 表中的所有行和列：

```py
SELECT * FROM users;
```

星号 * 表示我们想要检索表中的所有列。因此，我们将看到所有用户及其相关的 `user_id` 和 `password_hash` 列表。我们目前只有一个用户，所以我们得到：

```py
 username | password_hash
----------+---------------
 alice    | Pass_word&34
(1 row)
```

##### 注意

如果你看到密码如此容易被查询而感到惊恐，请不要担心！正如我们很快就会看到的，我们实际上并不会以这种方式存储密码。

我们也可以选择只从表中检索某些列。例如，如果我们只想从 `haikus` 表中检索 `haiku_id`、`created_at` 和 `author` 字段，我们可以这样写：

```py
SELECT haiku_id, created_at, author FROM haikus;
```

这将给我们：

```py
 haiku_id |          created_at           | author
----------+-------------------------------+--------
        1 | 2024-12-10 16:12:11.71654-08  | alice
        2 | 2024-12-10 16:12:16.364669-08 | alice
(2 rows)
```

我们还可以使用 `WHERE` 子句根据条件（或条件集）过滤我们想要看到的行。例如，如果我们只想看到由 `alice` 编写的且包含 `"fox"` 的俳句的 `haiku_id` 和 `text`，我们可以使用以下内容：

```py
SELECT haiku_id, text FROM haikus WHERE author = 'alice' AND text LIKE '%fox%';
```

以获得：

```py
 haiku_id |             text
----------+-------------------------------
        1 | Two foxes leap high          +
          | One lands safely on the earth+
          | Where is the other?
(1 row)
```

我们在这里使用两个由关键字 `AND` 分隔的条件过滤了 `haikus` 表：`author = 'alice'` 和 `text LIKE '%fox%'`。

`LIKE` 关键字用于执行文本匹配。`%` 符号表示“零个或多个字符”，使 `%fox%` 成为一个匹配包含单词 'fox' 的任何文本的模式。

`SELECT` 语句可以做得比这更多，例如计算汇总统计信息、聚合行、通过连接从多个表中获取数据等等，但这些内容超出了本章的范围。

#### 更新表

在插入行后，您可以使用 `UPDATE` 语句更新表中的行。例如，假设您想将字符串 "[By Alice] " 添加到 `haiku_id` 为 1 的俳句的开头：

```py
UPDATE haikus SET text = '[By Alice] ' || text WHERE haiku_id = 1;
```

在这里，我们正在设置 `haikus` 表中 `haiku_id` 为 `1` 的行的 `text` 列。SQL 中使用 `||` 运算符来连接字符串，因此 `SET text = '[By Alice] ' || text` 将 "[By Alice] " 添加到开头。如果我们现在通过以下方式 `SELECT` 该俳句的文本：

```py
SELECT text FROM haikus WHERE haiku_id = 1;
```

我们将得到：

```py
              text
--------------------------------
 [By Alice] Two foxes leap high+
 One lands safely on the earth +
 Where is the other?
(1 row)
```

虽然我们在这里只更新了一行，但 `UPDATE` 将更新所有与 `WHERE` 子句匹配的行。如果我们省略了 `WHERE` 子句，则 `UPDATE` 将应用于表中的每一行。

#### 删除行（和表）

要从表中删除行，我们使用 `DELETE FROM` 命令。与 `UPDATE` 的情况一样，它使用 `WHERE` 条件来确定要删除的行。

因此，我们可以通过以下命令删除 ID 为 2 的俳句：

```py
DELETE FROM haikus WHERE haiku_id = 2;
```

小心使用此命令！如果您省略了 `WHERE` 子句，`DELETE FROM` 将删除 *每个* 单独的行！

最后，我们可以通过使用 `DROP TABLE` 命令本身删除一个表（而不是仅删除表中的行）。

例如，如果我们想删除 `haikus` 表（目前我们不需要这样做），我们可以输入以下内容：

```py
DROP TABLE haikus;
```

当然，SQL 的内容远不止我们所学的这些——实际上，整个职业生涯都可以通过对其有足够的了解来塑造——但这正是我们本章所需的所有内容。

### 8.2.5 将 PostgreSQL 连接到我们的应用程序

我们现在已经为“俳句天堂”数据库设置了数据库。我们还学会了手动插入行和查询数据库 *手动*。接下来要做的就是使我们的应用程序能够通过 Python *程序化*地查询和修改数据库。

为了做到这一点，我们将使用一个名为 `psycopg2` 的第三方 Python 模块，它为我们提供了一种在应用程序中与 PostgreSQL 通信的方式。

现在就安装它，通过在新终端窗口中运行 `pip install psycopg2` 来完成。

#### 数据库连接是如何工作的

在使用 `psycopg2` 之前，让我们了解 PostgreSQL 查询在 Python 中的工作方式。

在前面的章节中，我们了解到 Streamlit 应用程序由一个 Streamlit 服务器进程提供服务，该进程监听机器上的特定端口（通常是 8501 或附近某个端口）。同样，PostgreSQL 数据库在 PostgreSQL 服务器上运行，该服务器监听不同的端口——默认为 5432，除非你在安装时设置了不同的端口号。

为了执行 SQL 命令，应用程序必须与 PostgreSQL 服务器建立*连接*。单个连接一次只能处理一个命令。

但如果多个用户同时尝试创建或更新一个俳句，怎么办？我们可以在单个连接上排队 SQL 命令，但这可能会导致延迟。使用多个*连接*更好，但设置每个连接可能会消耗资源——尤其是在生产环境中，因为它涉及到网络初始化和身份验证。

`psycopg2`的解决方案是一个*连接池*，它高效地管理多个*可重用*的数据库连接，如图 8.5 所示。

![image](img/ch08__image005.png)

##### 图 8.5 具有 1 个最小连接和 5 个最大连接的数据库连接池

连接池维护一组可以被应用程序的不同部分或访问应用程序的不同用户快速重用的连接。当应用程序的一部分需要运行数据库查询时，它可以：

+   从池中请求一个连接

+   使用连接来执行查询

+   将连接返回到池中以供重用

#### 使用`psycopg2`连接到我们的数据库

考虑到所有这些，让我们尝试设置一个实时数据库连接并运行一个查询！在终端中输入`python`（或`python3`）以打开 Python shell。

一旦你连接上了，从模块中导入我们需要的内容：

```py
>>> from psycopg2.pool import ThreadedConnectionPool
```

`ThreadedConnectionPool`是我们将要使用的连接池类。为了实例化它，我们需要知道我们的 PostgreSQL 服务器运行的地址、端口号、我们创建的 PostgreSQL 用户的用户名和密码，以及我们数据库的名称。

我们可以将所有这些组合成一个*连接字符串*，其形式如下：

```py
postgresql://<PostgreSQL username>:<Password>@<Address of server>:<Port number>/<Database name>
```

在我们的情况下，我们将输入：

```py
>>> connection_string = 'postgresql://haiku_lord:<password>@localhost:5432/haikudb'
```

显然，你需要将`<password>`替换为你为`haiku_lord`创建的实际密码。由于我们现在在本地运行 PostgreSQL，服务器地址简单地是`localhost`，但在本章后面切换到托管 PostgreSQL 服务时，这将会改变。

我们现在可以创建我们的连接池：

```py
>>> connection_pool = ThreadedConnectionPool(1, 5, connection_string)
```

这里的 1 和 5 是池中的最小和最大连接数。这意味着即使在我们收到任何请求之前，也始终保留一个连接可用，并且可以维护最多 5 个并发连接。

让我们使用`getconn`方法从池中获取一个连接：

```py
>>> connection = connection_pool.getconn()
```

为了执行查询，我们需要一个*游标*，它是一个指针，允许我们以灵活的方式执行 SQL 命令并从数据库中检索结果。我们通过在`connection`对象上调用`cursor()`方法来创建一个：

```py
>>> cursor = connection.cursor()
Let's now write the query itself. We'll try to fetch the haiku_id and author fields for a given author:
>>> query = 'SELECT haiku_id, author FROM haikus WHERE author = %s'
```

注意这里的 `%s`。这使得作者的用户名成为查询的参数。通过插入不同的用户名，我们可以获取不同作者的俳句。现在，我们想要 `alice` 的俳句，所以我们将创建一个包含查询的元组来传递。

```py
>>> params = ('alice',)
```

`('alice',)` 是包含字符串 `'alice'` 的单元素元组的字面表示。

```py
>>> cursor.execute(query, params)
```

一旦执行完成，让我们获取所有结果。

```py
>>> cursor.fetchall()
[(2, 'alice'), (1, 'alice')]
```

结果以元组列表的形式出现，其中每个元组代表数据库中的一行。由于我们的 `SELECT` 子句说了 `haiku_id, author`，每个结果元组的第一个元素是 `haiku_id`，而第二个是 `author` 字段。

由于我们已经完成了查询，让我们将连接返回到连接池，以便应用程序的其他部分可以使用它：

```py
>>> connection_pool.putconn(connection)
```

这就结束了我们对 `psycopg2` 的说明。作为最后一步，让我们清理连接池：

```py
>>> connection_pool.closeall()
```

这将关闭所有连接，并且无法再从连接池中请求更多。

#### 创建数据库类

现在我们知道了如何在 Python 中运行 SQL 查询，我们准备开始编写应用程序的代码。

在本章中，我们将把我们的代码组织在两个文件夹中：`backend` 和 `frontend`，以及一个主入口脚本（我们将使用 `streamlit run`），它位于这两个文件夹之外。

我们刚才提到的数据库连接内容相当技术性，我们宁愿不在应用程序代码的其余部分处理它。如果我们可以有一个数据库对象，我们可以直接要求执行所需的查询，而不必担心连接池和游标的细节。每当我们要运行特定的查询时，我们应该能够编写 `database.execute_query(query, params)`，传递我们想要执行的查询和想要提供的参数。

为了设置这个环境，让我们创建一个 `Database` *类*。

在后端文件夹中创建一个新的 Python 文件，命名为 `database.py`，并将列表 8.1 中的代码复制进去。

##### 列表 8.1 后端/database.py

```py
from psycopg2.pool import ThreadedConnectionPool

MIN_CONNECTIONS = 1
MAX_CONNECTIONS = 10

class Database:
  def __init__(self, connection_string):
    self.connection_pool = ThreadedConnectionPool(
      MIN_CONNECTIONS, MAX_CONNECTIONS, connection_string
    )

  def connect(self):
    return self.connection_pool.getconn()

  def close(self, connection):
    self.connection_pool.putconn(connection)

  def close_all(self):
    print("Closing all connections...")
    self.connection_pool.closeall()

  def execute_query(self, query, params=()):
    connection = self.connect()
    try:
      cursor = connection.cursor()
      cursor.execute(query, params)
      results = cursor.fetchall()
      connection.commit()
      return results
    except Exception:
      connection.rollback()
      raise
    finally:
      self.close(connection)
```

到目前为止，你应该已经习惯了我们在前几章中使用过的简单数据类。数据类简化了在 Python 中创建类所使用的语法，但对于更复杂的使用场景，我们需要剥去这一层，并编写传统的类定义。

类本质上是一个可以被转换成具体的 Python 对象的蓝图，它可以拥有属性——这些是对象的属性或与对象关联的数据——以及方法——或定义对象行为并能与其属性交互的函数。

让我们研究 `Database` 类的定义，如列表 8.1 所示。我们将从 `__init__` 方法开始：

```py
def __init__(self, connection_string):
  self.connection_pool = ThreadedConnectionPool(
    MIN_CONNECTIONS, MAX_CONNECTIONS, connection_string
  )
```

`__init__`（发音为“dunder init”）是 Python 中的一个特殊方法。当一个对象首次从一个类创建时，`__init__` 方法会自动执行。我们的 `__init__` 方法接受的两个参数是 `self` 和 `connection_string`。

在 Python 类定义中，你会非常频繁地看到 `self`。类内部方法的第一参数是一个特殊的参数，在调用它时不会明确传递（我们稍后会看到）。它始终指向被调用方法的对象。按照惯例，这个参数被命名为 `self`——尽管技术上你可以称它为任何你喜欢的名字。

`connection_string` 的目的是存储我们之前形成的字符串类型，其中包含数据库配置。

这行 `self.connection_pool = ThreadedConnectionPool(MIN_CONNECTIONS, MAX_CONNECTIONS, connection_string)` 创建了一个 `psycopg2` 的 `ThreadedConnectionPool`，就像我们在前面的部分中所演示的那样，并将其分配给 `self.connection_pool`，使其成为从类创建的对象的属性。

`MIN_CONNECTIONS` 和 `MAX_CONNECTIONS` 在文件顶部定义，可以轻松配置或稍后更改。

我们实际上是在创建数据库对象时立即初始化连接池。

我们还定义了除 `__init__` 之外的方法，例如 `connect`，它从连接池中获取连接，`close`，它将给定的连接返回到连接池，以及 `close_all`，它关闭连接池本身（并在终端窗口中打印一条消息）。这些代码应该很熟悉，因为我们之前在尝试 `psycopg2` 时讨论过。

然而，我们通常不会直接调用这些方法。回想一下，我们真正想要的是一个简单的 `execute_query` 方法，它会处理所有底层的数据库连接逻辑。让我们现在将注意力转向该方法：

```py
def execute_query(self, query, params=()):
  connection = self.connect()
  try:
    cursor = connection.cursor()
    cursor.execute(query, params)
    results = cursor.fetchall()
    connection.commit()
    return results
  except Exception:
    connection.rollback()
    raise
  finally:
    self.close(connection)
```

`execute_query` 自然接受我们想要执行的 SQL 查询以及我们想要提供给查询的任何参数（默认设置为空元组 `()`）。

方法的主体首先从连接池中获取一个连接。然后我们看到一个 `try`-`finally` 块。

在 Python 中，`try`-`except`-`finally` 是一个用于错误处理的构造 `that's`。其思路是将你的常规代码写在 `try` 块中。如果在运行 `try` 代码时发生异常，Python 会停止执行并跳转到 `except` 块，该块会 "捕获" 异常，允许你记录一个合理的错误信息，例如，或者使用其他类型的处理逻辑。

无论是否有异常，`finally` 块中的代码总是会执行。

`try` 块中的代码只是获取一个游标，执行查询并返回结果，就像我们在探索 `psycopg2` 时所看到的那样。你可能注意到的唯一新事物是 `connection.commit()` 这一行。

这一行存在是因为我们不仅仅会使用 `execute_query` 来运行 `SELECT` 查询。我们还会运行 `INSERT INTO`、`UPDATE` 和 `DELETE FROM` 命令——所有这些都会 *修改* 表，而不仅仅是 *读取* 它们。

在 PostgreSQL 中，当你修改数据时，直到你 *提交* 它们之前，这些更改不会被永久保存。

如果在 Python 尝试执行此代码时发生某种错误（例如，给定的查询可能包含不正确的 SQL 语法），该怎么办？我们进入由`except Exception`标记的`except`块，其中包含`connection.rollback()`这一行。这将撤销所做的任何临时更改，以便数据库保持原始状态。

我们还打印出我们遇到的错误，然后重新抛出异常，让常规的异常流程接管——例如，将异常消息打印到屏幕上。我们通过`try`-`except`结构实现的是，在发生异常时将自己注入流程中，并确保任何部分更改都被回滚。

那么`finally`块呢？嗯，它调用文件中更高处的`close`方法，将连接返回到池中。无论`try`块中发生什么，连接总是会被释放。如果我们没有这段代码，连接在函数退出后仍然会被分配，最终池将耗尽新的连接。将此代码放在`finally`块中确保即使在发生错误的情况下，连接也会被返回。

#### 在我们的应用中使用数据库类

在设置了`Database`类之后，要使我们的应用实现持久存储和检索，剩下的就是实际上在我们的 Streamlit 应用中使用这个类。

创建数据库类的实例需要提供一个连接字符串。连接字符串是敏感的，因为它们包含 PostgreSQL 凭据，所以我们不希望将字符串放入我们的代码中。

你可能知道接下来会发生什么——我们需要像过去一样使用`st.secrets`！无需多言，立即在你的应用根目录（即创建`backend`目录的父文件夹）中创建一个`.streamlit`文件夹，并在其中创建一个`secrets.toml`文件，其内容类似于列表 8.2 中所示。

##### 列表 8.2 .streamlit/secrets.toml

```py
[config]
connection_string = "postgresql://haiku_lord:password@localhost:5432/haikudb"
```

不要忘记将`password`替换为你的实际密码。此外，正如我们之前所学的，你不应该将此文件提交到 Git。

接下来，在应用的根目录中创建我们的应用入口文件——比如说，`main.py`——，其中的代码如列表 8.3 所示。

##### 列表 8.3 main.py

```py
import streamlit as st
from backend.database import Database

st.title('Haiku Haven')

connection_string = st.secrets['config']['connection_string']
database = Database(connection_string)
query_results = database.execute_query('SELECT * FROM haikus')
st.write(query_results)
```

这就是一切汇聚在一起的地方！首先，我们使用以下方式导入我们刚刚创建的`Database`类：

```py
from backend.database import Database
```

这行代码的意思是“从`backend/database.py`导入数据库类”。注意，在模块导入路径中，路径分隔符变成了点。

为了使这一行代码能够正常工作，Python 需要识别`backend`的父文件夹作为起点，从该起点它可以查找模块。我们可以通过将`backend`父文件夹的路径添加到`sys.path`（Python 中确定此内容的路径列表）来实现这一点。

幸运的是，当你运行`streamlit run <script.py>`命令时，`<script.py>`的父文件夹会自动添加到`sys.path`中。在这种情况下，由于`main.py`的父文件夹也是`backend`目录的父文件夹，我们不需要做任何额外的事情。

接下来，在显示标题之后，我们像之前章节中做的那样，从`st.secrets`中提取连接字符串：

```py
connection_string = st.secrets['config']['connection_string']
```

我们使用这个来创建`Database`类的实例：

```py
database = Database(connection_string)
```

你可能在我们之前使用 dataclasses 时已经认识到了这种语法，但这是本书中我们第一次实例化我们编写的传统类，因此需要更深入的说明。

这里发生的事情是，通过将`connection_string`传递给类`Database`，就像传递给一个函数一样，我们实际上是在将其传递给`Database`的`__init__`方法，该方法具有签名行`def __init__(self, connection_string)`。正如我之前提到的，`self`会自动设置为正在创建的对象，所以我们不需要显式地传递它。

相反，我们只传递`connection_string`，这样`__init__`就会为我们设置连接池，并将得到的`Database`实例存储在`database`变量中（注意，即使我们没有写一个显式的`return`语句，`__init__`也会返回实例）。

一旦我们有了`database`实例，我们就用它来执行一个简单的`SELECT`查询：

```py
query_results = database.execute_query('SELECT * FROM haikus')
```

再次强调，尽管`execute_query`将`self`作为其第一个参数，我们不需要显式地传递它。相反，`database`本身被传递给`self`。

这个查询没有参数，所以我们通过不指定它，让`params`参数保持默认值（一个空元组）。

最后，我们使用`st.write`来显示查询的结果：

```py
st.write(query_results)
```

通过输入`streamlit run main.py`来查看页面（确保你首先在`main.py`的包含文件夹中，这样 Streamlit 才能找到`.streamlit`文件夹）。你应该会看到类似于图 8.6 的内容。

![image](img/ch08__image006.png)

##### 图 8.6 使用 Database 类执行 SELECT 查询并使用 st.write 显示的俳句表内容（完整代码请见 GitHub 仓库中的 chapter_8/in_progress_01）

这里有趣的一点是`st.write`如何以易于阅读的格式格式化`query_results`中的元组列表。

无论如何，我们的应用现在已经连接到数据库了！接下来，让我们使用这个来允许用户创建账户！

## 8.3 创建用户账户

由于我们的用户可以创建他们自己的俳句，因此他们需要有一种方式来创建 Haiku Haven 账户来保存他们的俳句。在本节中，我们将连接我们的应用以启用此功能，并注意安全地存储密码。

在我们到达那里之前，让我们花一分钟来谈谈代码组织。

### 8.3.1 将我们的应用拆分为服务

在第三章中，我们讨论了*关注点分离*的原则——即我们的应用程序的每个组件都应该专注于特定的事情，并且与其他组件独立，仅以合同或 API 指定的方式与之交互。

我们在这里将做类似的事情，就像在第三章中那样，将前端和后端分开。由于我们这次使用类和*面向对象编程*，我们可以定义一个后端类*——*让我们称它为`Hub`——它可以作为前端代码调用后端代码的单一点。任何前端可以调用的函数都应该在`Hub`类中是一个方法。这与第三章中的`backend.py`类似，其中每个从前端代码调用的后端函数都在`backend.py`中定义。

我们可能期望`Hub`类拥有满足前端用户可能想要执行的操作的方法，例如`create_user`、`create_haiku`、`update_haiku`等。然而，随着时间的推移，随着我们的应用程序变得越来越复杂，`Hub`类的方法数量会不断增加，慢慢地使其变得难以驾驭和管理。

而不是采取这种*单体*方法，将`Hub`提供的操作分成单独的*服务类*可能更好，每个类都针对特定类型的操作，并使用`Hub`类仅作为协调者。例如，我们可以有一个`UserService`类，它提供与用户相关的操作，如`create_user`，以及一个`HaikuService`类，它提供与俳句相关的操作，如`create_haiku`和`update_haiku`。

这将使我们的应用程序更加模块化，更容易扩展和维护。我们可以独立于`HaikuService`中的俳句相关操作，向`UserService`添加更多与用户相关的功能，如果我们想在以后添加打油诗到我们的应用程序中，我们可以引入一个`LimerickService`而不需要触及任何东西。

考虑到我们的整体代码组织策略，让我们将注意力转向构建组件之一——用户服务。

### 8.3.2 创建用户服务

正如在先前的章节中一样，我们将使用 dataclass 来表示我们关心的基本对象。在第 3、4 和 6 章中，我们有`Unit`、`Task`和`Metric`类。在这里，我们将从`User`类开始。

在`backend`文件夹中创建一个新的 Python 文件，命名为`user.py`，其文本如列表 8.4 所示。

##### 列表 8.4 backend/user.py

```py
from dataclasses import dataclass

@dataclass
class User:
  username: str
  password_hash: str
```

你会看到这直接反映了我们数据库中的`users`表，其中包含`username`和`password_hash`字段。

现在我们来谈谈后一个字段，以及为什么我们称之为`password_hash`而不是`password`。

#### 安全存储密码

密码自然是软件开发者需要处理的最敏感的信息之一，而网络安全领域的很大一部分都集中在保持它们的秘密性。

我们知道我们不应该在我们的代码中存储密码，而是求助于`st.secrets`构造来避免这种情况。但是，将它们存储在数据库中又如何呢？

显然，我们的应用程序需要能够将用户输入的密码与实际关联的用户密码进行比较，因此密码需要以某种形式存储。然而，直接将它们以纯文本形式存储在数据库中会引入安全漏洞。

这是因为任何通过某种安全漏洞获得我们数据库访问权限的人——将能够看到以原始形式存储的密码。但我们如何避免这种情况呢？

答案是：使用**单向加密哈希函数**，或者说，一点复杂的数学。事实证明，有一些数学运算在正常情况下很容易执行，但在反向执行时却极其困难。作为一个简单的例子，考虑将两个质数 a 和 b 相乘得到 c。a 和 b 相乘得到 c 很容易，但如果你只得到 c，识别 a 和 b 就困难了，尤其是当 c 非常大时（想想有几百位长）。

同样，你可以将加密哈希函数视为对密码执行的操作，这种操作很难逆转。

假设某个人的密码是`SomePassword123`。如果你对它应用一个哈希函数`H`，你可能会得到一个看起来像随机字符序列的密码哈希：

而不是直接将字符串`SomePassword123`存储在数据库中，我们存储`g53jkdlgfee09ded8d33rr45t5y5y43f2eff`。然后当有人在我们的应用程序中输入密码时，我们将哈希函数应用于**那个**密码，并将结果与`g53jkdlgfee09ded8d33rr45t5y5y43f2eff`进行比较。如果两者相同，则用户被验证。

这如何帮助提高安全性？好吧，如果黑客现在设法进入我们的数据库，他们只能得到密码的哈希值，而不是实际的密码。正如所述，从密码哈希中获取密码非常困难。

对于黑客来说，密码哈希本身是无用的，因为没有理由在应用程序中输入它——如果你这样做，应用程序会对其应用哈希函数，并得出一个与真实哈希值完全不同的哈希值。

我们如何在我们的应用程序中实现这一点？幸运的是，我们不必从头开始。有一些第三方库为我们做了这件事。我们将使用`bcrypt`，你现在应该使用`pip install bcrypt`来安装它。

让我们在`User`类中添加两个更多的方法，使其现在看起来像这样：

```py
from dataclasses import dataclass
import bcrypt

@dataclass
class User:
  username: str
  password_hash: str

  @staticmethod
 def hash_password(password):
 return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()

  def authenticate(self, password):
 return bcrypt.checkpw(password.encode(), self.password_hash.encode())
```

我们用`@staticmethod`装饰了`hash_password`方法。这使得它属于类本身，而不是属于类的任何特定实例。我们通常使用`@staticmethod`来表示与类逻辑相关但不需要访问特定实例的实用函数。

`hash_password` 函数非常适合这个用途，因为它不需要访问实例的任何属性或方法（注意没有`self`参数）。相反，它只是接受用户输入的密码，使用`bcrypt`将其转换为哈希值，并返回它。

我不会深入讲解其工作原理的细节，但总的来说，我们在对密码进行散列之前，通过添加一个随机的“盐”（`bcrypt.gensalt()`）来采取额外的安全措施。这种盐有助于保护密码免受黑客通过从巨大的预计算散列表（称为*彩虹表*）中查找与密码散列关联的密码的攻击。

我们还有一个`authenticate`方法，当用户输入密码时我们将使用它。`bcrypt.checkpw`比较输入的密码（`password.encode()`）和存储在`User`对象中的密码散列（`self.password_hash.encode()`），如果它们匹配则返回`True`。

#### UserService 类

现在，我们准备创建`UserService`类，这是我们确定将具有用户相关操作方法的类。在`backend/`目录下创建一个名为`user_service.py`的文件，其内容如列表 8.5 所示。

##### 列表 8.5 backend/user_service.py

```py
from backend.user import User

class UserService:
  def __init__(self, database):
    self.database = database

  def get_user(self, username):
    query = "SELECT username, password_hash FROM users WHERE username = %s"
    params = (username,)
    results = self.database.execute_query(query, params)
    return User(*results[0]) if results else None

  def create_user(self, username, password):
    existing_user = self.get_user(username)
    if not existing_user:
      query = '''
        INSERT INTO users (username, password_hash)
          VALUES (%s, %s)
          RETURNING username, password_hash
      '''
      password_hash = User.hash_password(password)
      params = (username, password_hash)
      results = self.database.execute_query(query, params)
      return User(*results[0]) if results else None
    return None
```

`UserService`类有一个`__init__`方法，它接受我们之前创建的`Database`类的一个实例，并将其分配给对象的`database`属性（`self.database`）。

当用户输入用户名和密码以创建用户时，我们首先需要检查是否存在具有该用户名的用户，因此我们有一个`get_user`方法，如果存在用户，则返回用户，如果不存在，则返回`None`。

`get_user`方法在数据库上执行一个参数化的 SQL 查询（`SELECT username, password_hash FROM users WHERE username = %s`），传递给定的用户名作为唯一参数（`(username, )`）。

如我们之前所见，这返回了一个元组的列表。由于`SELECT username, password`，这些将具有形式`(<username>, <password_hash>)`。考虑`get_users`中的最后一行：

```py
return User(*results[0]) if results else None 
```

如果没有给定用户名的用户，结果将是一个空列表，因此它将评估为`False`，导致`get_user`返回`None`。

如果确实存在这样的用户，`results[0]`将是一个形式为`(<username>, <password_hash>)`的元组。在 Python 中，当`*`运算符应用于元组（或列表）时，它会*解构*它以用于函数调用等。

因此`User(*results[0])`等价于`User(<username>, <password_hash>)`，这创建了一个新的`User`数据类实例（您会记得它有两个相应的成员：`username`和`password_hash`）。

`create_user`方法首先使用`self.get_user(username)`来查看是否存在具有给定用户名的用户。如果存在，它将简单地返回`None`。

如果它不存在，它将向数据库发出以下查询：

```py
INSERT INTO users (username, password_hash)
  VALUES (%s, %s)
  RETURNING username, password_hash
```

这是一个`INSERT`查询，我们之前已经见过。这里唯一的新内容是行`RETURNING username, password_hash`。通常，`INSERT`查询不需要返回任何结果，因为它是一个修改操作，而不是读取操作。

添加`RETURNING`子句使其以与`SELECT`查询相同的方式返回指定的字段。在这种情况下，返回新创建行的`username`和`password_hash`。

再次强调，`create_user`使用与`get_user`（`User(*results[0])`）相同的方法来创建并返回一个`User`对象，如果一切顺利的话。

#### `Hub`类

当我们之前提到代码组织时，我们提到了`Hub`类，它将是我们的前端代码访问的单一点。现在让我们编写这个类。创建`backend/hub.py`，并使用列表 8.6 中的代码。

##### 列表 8.6 backend/hub.py

```py
from backend.database import Database
from backend.user_service import UserService

class Hub:
  def __init__(self, config):
    database = Database(config['connection_string'])
    self.user_service = UserService(database)
```

`Hub`类的`__init__`非常简单：它接受一个`config`对象（一个配置选项的字典，例如我们之前解析的`secrets.toml`文件中获得的），创建一个`Database`对象，并将其传递给`UserService`以创建该类的实例。

`Hub`没有其他方法。这很有道理，因为我们之前强调过，`Hub`只是一个协调器类，我们的前端可以使用它来访问各种服务类对象（其中`user_service`是一个`UserService`的实例，这是我们迄今为止创建的唯一一个）。

#### 注册页面

从下往上，我们创建了一个`User`类，一个访问`User`类的`UserService`类，以及一个访问`UserService`类的`Hub`类——目前它只有一个`create_user`方法。

访问`create_user`的我们的 Streamlit 应用程序部分将是注册页面，我们暂时在`main.py`中定义它。

我们之前的`main.py`直接初始化了`Database`对象并执行了一个示例查询。由于数据库现在在`Hub`类中初始化，我们将完全重写`main.py`，如列表 8.7 所示。

##### 列表 8.7 main.py 修订版

```py
import streamlit as st
from backend.hub import Hub

hub = Hub(st.secrets['config'])

with st.container(border=True):
  st.title("Sign up")
  username = st.text_input("Username")
  password = st.text_input("Password", type="password")
  confirm_password = st.text_input("Confirm password", type="password")

  if st.button("Create account", type="primary"):
    if password != confirm_password:
      st.error("Passwords do not match")
    else:
      user = hub.user_service.create_user(username, password)
      if user:
        st.success("Account created successfully")
      else:
        st.error("Username already exists")
```

到这本书的这一部分，你应该能够比较容易地阅读列表 8.7 中的代码。

它首先通过使用`st.secrets`的`'config'`条目来创建一个`Hub`实例（注意这里，`Hub`接受整个`config`对象，而不仅仅是连接字符串，以防需要考虑其他配置）。

接下来，它显示了通常的用户名-密码-确认密码的输入组合，我相信你之前在各种网站上已经见过。点击“创建账户”按钮时，如果两个输入框中的密码不匹配，会显示错误。

如果它们匹配，我们调用在`UserService`类中定义的`create_user`方法来在数据库中创建用户：

```py
user = hub.user_service.create_user(username, password)
```

然后我们根据返回值显示成功或错误消息。

在这个阶段，如果你重新运行应用程序，你应该能够看到图 8.7。

![image](img/ch08__image007.png)

##### 图 8.7 Haiku Haven 的注册页面（完整代码请见 GitHub 仓库中的 chapter_8/in_progress_02）

尝试使用用户名`bob`创建一个账户。为了验证它是否成功并且确实创建了一个用户，你可以在你的`psql`提示符（希望你保持打开状态）中发出查询`SELECT * from users where username = 'bob'`。这应该会给你类似以下的结果：

```py
 username |                        password_hash
----------+--------------------------------------------------------------
 bob      | $2b$12$hVzjZJN7QMTM94H7ZL.ZJe3PFfgyAPgDOH1F2b38IovcuvKrNAu3G
(1 row)
```

如您所见，由于散列，`bob` 的密码不再直接可见。但 `bob` 能登录吗？除非我们完成了下一部分！

## 8.4 设置多页登录流程

我们之前在绘制用户体验草图时看到——Haiku Haven 意味着它应该是一个多页应用，具有注册、登录和俳句相关功能的不同页面，这是我们之前没有遇到过的。

### 8.4.1 Streamlit 中的多页应用

Streamlit 内置了对多页应用的支持。在这个方案中，你单独定义你的各个页面，让你的入口文件（你用 `streamlit run` 运行的文件）充当一个“路由器”，它识别要加载的页面并运行它。

入口文件在这里非常关键；它通常在每次重新运行时加载，并且是选择要加载的“当前页面”的那个文件。

让我们来看一个例子。到目前为止，在我们的应用中，我们已经为用户创建了一个注册页面，以便他们创建自己的账户，但还没有登录页面。一旦我们创建了登录页面，就需要有一种方法将这两个页面联系起来，使它们成为同一个应用的一部分。

我们将通过再次修订 `main.py` 文件来实现这一点，使用上面讨论的多页方法。我们在 `main.py` 中包含的注册流程将不得不移动到不同的文件（`frontend/signup.py`）。新的 `main.py` 如列表 8.8 所示。

##### 列表 8.8 main.py 修订（再次）

```py
import streamlit as st
from backend.hub import Hub

pages = {
 "login": st.Page("frontend/login.py", title="Log in",
 icon=":material/login:"),
 "signup": st.Page("frontend/signup.py", title="Sign up", 
 icon=":material/person_add:"),
}

if 'hub' not in st.session_state:
 config = st.secrets['config']
 st.session_state.hub = Hub(config)

page = st.navigation([pages['login'], pages['signup']])
page.run()
```

你在这里首先会注意到的是 `pages` 字典。`pages` 中的键是 `"login"` 和 `"signup"`，这是我们希望在应用中出现的页面名称。值是 `st.Page` 对象。让我们检查第一个：

```py
st.Page("frontend/login.py", title="Log in", icon=":material/login:")
```

`st.Page` 是 Streamlit 定义多页应用中单个页面的方式。你传递给它的第一个参数是该页面的 Python 脚本路径——在本例中是 `frontend/login.py`，但目前还不存在。我们还传递给它一个合理的标题。

最后一个参数是页面的图标。它有一个奇特的价值：`:material/login:`。

这展示了在 Streamlit 中显示图标的一种巧妙方法。语法 `:material/<icon_name>` 被大多数接受可显示文本的小部件所接受，并在渲染到屏幕上时转换为图像。

你可以在 Google 的 Material Symbols 库中看到支持的图标，网址是 `https://fonts.google.com/icons?icon.set=Material+Symbols`。在这种情况下，我们选择了“登录”图标。每次你需要显示一个图标时，你都可以访问那个 URL，点击你想要的图标，从右侧打开的侧边栏中识别其图标名称，并将其替换为文本 `:material/<icon_name>:` 中的。

现在，将你的注意力转向下面更下面的以下几行：

```py
page = st.navigation([pages['login'], pages['signup']])
page.run()
```

在这里，我们将 `pages` 字典中的两个 `st.Page` 对象（`pages['login']` 和 `pages['signup']`）传递给 `st.navigation`，一个新的 Streamlit 小部件。

`st.navigation` 用于配置多页 Streamlit 应用中可用的页面，显示一个用户可以使用它来选择要访问的页面的导航栏。

它接受一个由形成导航选项的`Page`对象组成的列表，并从列表中返回一个单一的`Page`对象。这个返回项是用户选择的页面，如果没有选择任何项，则是列表中的第一个项目。

一旦返回了一个页面，就可以使用它的`.run()`方法来加载它。

你还会看到我们将`Hub`实例（`hub`）保存到`st.session_state`中，但没有对它做任何其他操作。这是因为会话状态在多页应用中的页面之间是共享的。所以如果你在任意页面将某个东西保存到`st.session_state`中，它也会在其他页面中可访问。在这种情况下，我们将在其他页面中使用保存的`hub`对象。

关于我们之前提到的注册流程呢？嗯，现在我们的应用是多页的，我们将它移动到自己的页面，即`signup.py`，在名为`frontend`的新文件夹中。正如列表 8.9 所示，内容大部分只是直接从我们之前的`main.py`复制过来，没有做任何修改。

##### 列表 8.9 frontend/signup.py

```py
import streamlit as st

hub = st.session_state.hub

with st.container(border=True):
  st.title("Sign up")
  username = st.text_input("Username")
  password = st.text_input("Password", type="password")
  confirm_password = st.text_input("Confirm password", type="password")

  if st.button("Create account", type="primary"):
    if password != confirm_password:
      st.error("Passwords do not match")
    else:
      user = hub.user_service.create_user(username, password)
      if user:
        st.success("Account created successfully")
      else:
        st.error("Username already exists")
```

与列表 8.7 相比，我们做的唯一改变是从我们之前在新的`main.py`中保存的`st.session_state`中获取`hub`变量的值。

### 8.4.2 实现登录

在我们的多页应用基础设施到位后，是时候构建登录功能了。在我们设置登录页面之前，让我们确保我们的后端有我们需要的功能。

#### 在 UserService 中认证用户

正如我们之前讨论的，所有与用户相关的功能都需要在`UserService`中实现。目前，这个类有`create_user`和`get_user`方法。我们将实现一个新的`get_authenticated_user`方法：

```py
from backend.user import User

class UserService:
  ...

  def get_user(self, username):
    ...

  ...
 def get_authenticated_user(self, username, password):
 user = self.get_user(username)
 if user and user.authenticate(password):
 return user
 return None
```

`get_authenticated_user`接受用户名和密码作为参数。它首先调用我们之前定义的`get_user`方法，查看是否存在具有该用户名的用户。如果存在，它将在返回的`User`对象上调用`authenticate`方法。回想一下，`User`类中的`authenticate`方法比较给定密码的哈希值与实际密码的哈希值。

如果认证成功，将返回`User`对象。如果失败，该方法返回`None`，调用代码可以有两种解释：要么不存在这样的用户，要么密码不正确。为了简化，我们不会在返回值中区分这两种情况。

#### 创建登录页面

在`UserService`中我们就需要这些了。现在我们可以继续创建一个登录页面，以补充我们之前创建的注册页面。

在`frontend/`目录下创建一个名为`login.py`的新文件，内容如列表 8.10 所示。

##### 列表 8.10 frontend/login.py

```py
import streamlit as st

hub = st.session_state.hub

with st.container(border=True):
  st.title("Log in")
  username = st.text_input("Username", key="login_username")
  password = st.text_input("Password", type="password")

  if st.button("Log in", type="primary"):
    user = hub.user_service.get_authenticated_user(username, password)
    if user:
      st.session_state.logged_in = True
      st.session_state.user = user
      st.success("Logged in successfully")
    else:
      st.error("Invalid username or password")
```

这个页面与`signup.py`相当相似，应该很容易根据你对 Streamlit 当前的理解来理解。

这里需要关注的部分是当点击“登录”按钮时会发生什么。我们首先调用在`UserService`中定义的认证方法：

```py
user = hub.user_service.get_authenticated_user(username, password)
```

正如我们所见，如果该方法返回`User`对象，则认证成功；如果返回`None`，则认证失败。

我们将这个条件写成`user:`。对于实际的登录，我们将使用一个非常简单的方法——在`st.session_state`下存储一个名为`logged_in`的布尔变量，以及登录用户的`User`对象（简单地命名为`user`）。

我们还会根据登录是否成功显示成功或错误消息。

到目前为止，你应该使用`streamlit run main.py`重新运行你的应用。尝试使用你之前创建的账户登录。你应该看到类似于图 8.8 的内容。

![image](img/ch08__image008.png)

##### 图 8.8 Haiku Haven 的登录页面（完整代码请见 GitHub 仓库中的 chapter_8/in_progress_03）

注意由`st.navigation`创建的导航面板，它占据了侧边栏，并包含导航到任一页面的链接（以及我们添加的图标！）。

### 8.4.3 在页面之间导航

虽然我们目前在注册/登录流程中已经有了最基本的需求，但肯定有改进的空间。例如，如果用户在登录页面但没有账户，应该有一个有用的链接直接注册，反之亦然。

此外，当用户登录时，我们应该带他们到一个登录页面，并给他们提供登出的能力。

图 8.9 展示了我们想要设计的理想的注册/登录/登出流程。

![image](img/ch08__image009.png)

##### 图 8.9 通过重定向和页面链接展示页面之间连接的图表

除了能够在注册和登录页面之间来回切换，并在登录时重定向到主页外，我们还想根据我们是否登录来显示导航面板的不同选项。

如果用户已登录，他们应该看到主页并能够登出——这会再次显示登录页面。如果他们未登录，他们应该看到注册或登录的选项。

#### 将页面字典移动到自己的文件中

由于我们接下来将要在各个页面之间进行导航，因此将`pages`字典（位于`main.py`中，定义了可用的页面）放入一个单独的文件中会更清晰，所以让我们将这部分代码移动到`frontend/pages.py`，如列表 8.11 所示。

##### 列表 8.11 frontend/pages.py

```py
import streamlit as st

pages = {
  "login": st.Page("frontend/login.py", title="Log in",
                   icon=":material/login:"),
  "signup": st.Page("frontend/signup.py", title="Sign up",
                    icon=":material/person_add:"),
  "home": st.Page("frontend/home.py", title="Home",
                  icon=":material/home:"),
  "logout": st.Page("frontend/logout.py", title="Log out",
                    icon=":material/logout:")
}
```

你会注意到我们添加了两个新页面：`home`，它应该代表登录后的主页，以及`logout`，它将用户登出。

#### 页面之间的链接

Streamlit 允许你通过一个名为`st.page_link`的小部件在多页面应用中创建页面之间的链接。

让我们使用这个链接在`login`和`signup`页面之间建立联系。

`login.py`应该看起来像这样，在底部添加了页面链接：

```py
import streamlit as st
from frontend.pages import pages

...
with st.container(border=True):
    ...
    else:
      st.error("Invalid username or password")

st.page_link(pages["signup"], label="Don't have an account? Sign up!")
```

`st.page_link`非常容易理解；第一个参数是我们想要链接到的`st.Page`对象（从`pages`导入，位于`pages.py`），第二个是标签文本。

你也可以将常规 URL 作为第一个参数传递，以防你想链接到外部页面。`signup.py`有非常相似的改变：

```py
import streamlit as st
from frontend.pages import pages

...
with st.container(border=True):
    ...
    else:
      st.error("Invalid username or password")

st.page_link(pages["signup"], label="Don't have an account? Sign up!")
```

#### 动态更改 st.navigation

下一个我们将实现的功能是，在导航栏中显示用户当前上下文正确的页面，即当用户注销时显示`signup`和`login`，当用户登录时显示`home`和`logout`。

编辑`main.py`使其看起来像这样：

```py
import streamlit as st

from backend.hub import Hub
from frontend.pages import pages

if 'hub' not in st.session_state:
  config = st.secrets['config']
  st.session_state.hub = Hub(config)

if 'logged_in' in st.session_state and st.session_state.logged_in:
page = st.navigation([pages['home'], pages['logout']])
else:
 page = st.navigation([pages['login'], pages['signup']])

page.run()
```

显然，`pages`字典现在定义在`pages.py`中，并导入到`main.py`中。

如上图所示，为了动态更改导航面板中的内容，我们使用用户登录时保存的`logged_in`会话状态变量，并相应地改变分配给`page`的`st.navigation`对象。

如果用户已登录，导航栏现在将显示`home`和`logout`选项。由于`pages['home']`是传递给`st.navigation`列表中的第一个项目，因此当用户登录时，默认加载的页面就是`home`页面。

现在，让我们在`frontend/home.py`（列表 8.12）中实际设置一个占位符页面，以便让已登录用户有所见。

##### 列表 8.12 frontend/home.py

```py
import streamlit as st

user = st.session_state.user
st.title(f"Welcome, {user.username}!")
```

目前这里没有什么惊天动地的事情；我们只是显示一个包含已登录用户用户名的问候语。回想一下，我们在`login.py`中将已登录的`User`对象保存到`st.session_state.user`中。

#### 登录和注销的自动重定向

我们提出的理想登录流程要求用户在登录和注销时自动重定向。这究竟是如何工作的呢？

记住，当用户点击“登录”按钮时，`logged_in`会话状态变量被设置为`True`。

这意味着在下次重新运行时，`main.py`将获取`logged_in`的更改值，显示新的导航面板并加载`home.py`。

为了真正实现无缝操作，我们必须触发重新运行。因此，在`login.py`中添加一个`st.rerun()`。

```py
user = hub.user_service.get_authenticated_user(username, password)
    if user:
      st.session_state.logged_in = True
      st.session_state.user = user
      st.rerun()
```

至于注销？嗯，那将逆转登录时发生的所有事情。创建一个包含列表 8.13 内容的`logout.py`。

##### 列表 8.13 frontend/logout.py

```py
import streamlit as st

st.session_state.user = None
st.session_state.logged_in = False
st.rerun()
```

这样就完成了我们的注册/登录/注销流程！重新运行应用程序并尝试一下！当你登录时，你现在应该看到一个不同的导航面板和加载的首页（见图 8.10）。

![image](img/ch08__image010.png)

##### 图 8.10 登录页面中导航栏的不同选项（完整代码请见 GitHub 仓库中的 chapter_8/in_progress_04）

点击导航栏中的“注销”将重新加载登录和注册页面，底部有页面链接相互跳转。

## 8.5 创建、读取、更新和删除俳句

现在用户身份验证已经处理完毕，我们终于可以着手处理我们应用程序的核心功能：创建、读取、更新和删除俳句。我们将从创建俳句开始，将这种行为封装在`HaikuService`类中，然后对前端进行适当的修改。

### 8.5.1 定义 HaikuService 类

我们在俳句服务中遵循的代码结构与我们已经在`UserService`中拥有的结构非常相似。

让我们从表示俳句的 `Haiku` 数据类开始。在 `backend/` 文件夹中创建 `haiku.py`，如列表 8.14 所示。

##### 列表 8.14 backend/haiku.py

```py
from dataclasses import dataclass

@dataclass
class Haiku:
  haiku_id: int
  created_at: str
  author: str
  text: str
```

与 `User` 类一样，字段反映了我们相应的数据库表（`haikus`）中的字段。

`HaikuService` 本身如列表 8.15 所示。

##### 列表 8.15 backend/haiku_service.py

```py
from backend.haiku import Haiku

class HaikuService:
  def __init__(self, database):
    self.database = database

  def create_haiku(self, author, haiku_text):
    query = '''
      INSERT INTO haikus (author, text)
      VALUES (%s, %s)
      RETURNING haiku_id, created_at, author, text
    '''
    params = (author, haiku_text)
    results = self.database.execute_query(query, params)
    return Haiku(*results[0]) if results else None
```

再次，代码与 `UserService` 的代码相当类似，因此不需要详细的解释。

正如我们所看到的，我们只需要在我们的 SQL 查询中提供 `author` 和 `text` 字段；数据库自动提供 `haiku_id` 和 `created_at`，并且所有字段都按照 `RETURNING` 子句返回。

为了总结后端更改，让我们在 `hub.py` 中添加一个 `HaikuService` 实例：

```py
...
from backend.haiku_service import HaikuService

class Hub:
  def __init__(self, config):
    database = Database(config['connection_string'])
    self.user_service = UserService(database)
    self.haiku_service = HaikuService(database)
```

这将使俳句创建功能可以从前端访问，正如我们即将看到的。

### 8.5.2 允许用户创建俳句

我们之前的 `home.py` 当然只是一个占位符。我们的实际登录主页理想上应该有创建俳句并显示俳句的方式。

为了创建俳句，让我们创建一个类似于第七章中创建的模态对话框。

创建一个新文件，`frontend/haiku_editor.py`，如列表 8.16 所示。

##### 列表 8.16 frontend/haiku_editor.py

```py
import streamlit as st

@st.dialog("Haiku editor", width="large")
def haiku_editor(hub, user):
  haiku_text = st.text_area('Enter a haiku')
  if st.button('Save haiku', type='primary'):
    haiku = hub.haiku_service.create_haiku(user.username, haiku_text)
    if haiku:
      st.success('Haiku saved successfully!')
    else:
      st.error('Failed to save haiku')
```

`haiku_editor` 函数被 `st.dialog` 装饰器装饰——正如我们在上一章中看到的，它在模态屏幕中执行其主体。

`haiku_editor` 的主体很简单。我们首先接受用户在 `st.text_area` 小部件中输入的俳句文本：

```py
haiku_text = st.text_area('Enter a haiku')
```

`st.text_area` 正如您所期望的那样——一个用于输入多行文本的区域。

在点击“保存俳句”时，我们调用 `HaikuService` 下的 `create_haiku` 方法将其保存到数据库，并显示相应的成功/失败消息。

通过在 `home.py` 中包含一个“添加俳句”按钮来关闭循环，该按钮触发我们刚刚定义的 `haiku_editor` 对话框：

```py
import streamlit as st
from frontend.haiku_editor import haiku_editor

hub = st.session_state.hub
user = st.session_state.user
st.title(f"Welcome, {user.username}!")

if st.button(':material/add_circle: Haiku', type='primary'):
 haiku_editor(hub, user)
```

如您所见，我们再次使用了一个 Material 图标，这次是在按钮标签中的“添加”一词的位置。

让我们看看到目前为止一切是否正常工作！重新运行您的 Streamlit 应用程序，登录，并添加一个俳句（请参阅图 8.11）。

![image](img/ch08__image011.png)

##### 图 8.11 创建俳句（有关完整代码，请参阅 GitHub 仓库中的 chapter_8/in_progress_05）

为了让你确信确实创建了一个俳句，你可以查询你的 `haikus` 表。

### 8.5.3 其他 CRUD 操作：读取、更新、删除

为了使我们的 CRUD 应用程序功能完整，我们还需要实现三个更多操作：

+   *读取* 当前用户的俳句从数据库中读取并在应用程序中列出

+   *更新* 给定的俳句

+   *完全删除* 一个俳句

#### 在 `HaikuService` 中定义操作

如前所述，让我们首先在 `haiku_service.py` 中的 `HaikuService` 类中定义这些操作：

```py
from backend.haiku import Haiku

class HaikuService:
  ...
  def create_haiku(self, author, haiku_text):
    ...

  def get_haikus_by_author(self, author):
 query = 'SELECT * FROM haikus WHERE author = %s'
 params = (author,)
 results = self.database.execute_query(query, params)
 return [Haiku(*row) for row in results]

 def update_haiku(self, haiku_id, haiku_text):
 query = 'UPDATE haikus SET text = %s WHERE haiku_id = %s RETURNING *'
 params = (haiku_text, haiku_id)
 results = self.database.execute_query(query, params)
 return Haiku(*results[0]) if results else None

 def delete_haiku(self, haiku_id):
 query = 'DELETE FROM haikus WHERE haiku_id = %s RETURNING *'
 params = (haiku_id,)
 results = self.database.execute_query(query, params)
 return Haiku(*results[0]) if results else None
```

我们为每个操作定义一个方法：`get_haikus_by_author` 用于读取俳句，`update_haiku` 用于更新，`delete_haiku` 用于删除俳句。

在每种情况下，我们都使用之前看到的相同模式——我们运行一个 SQL 命令，将结果转换为 Haiku 对象，并返回它们。

在`get_haikus_by_author`的情况下，由于可能存在多个作者，我们返回一个包含 Haiku 对象的*列表*作为结果。

在`update_haiku`和`delete_haiku`中，我们分别使用`haiku_id`——俳句的唯一标识符作为参数，分别使用`UPDATE..SET`和`DELETE FROM` SQL 命令来实现所需的结果。

#### 创建用户界面

我们的后端方法返回`Haiku`类的实例，但我们如何在前端显示这些实例？

我们可能需要创建一个`display_haiku`函数，它接受一个`Haiku`对象并在屏幕上显示它。下一个自然的问题是：我们显示什么？用户可能想看到什么？

`Haiku`类有四个属性：`haiku_id`、`created_at`、`author`和`text`。

在这些中，`haiku_id`是一个内部标识符，对最终用户没有意义，因此我们可以排除它。`created_at`可能有助于唤醒用户对俳句最初创建时间的记忆。由于我们只将显示当前登录用户的俳句，因此显示`author`字段将是多余的。而`text`是俳句的内容，所以我们显然想显示它。

这就给出了：`created_at`和`text`。还有其他什么吗？嗯，我们还想给用户编辑或删除特定俳句的选项，所以让我们加入几个按钮。

##### 列表 8.17 frontend/haiku_display.py

```py
import streamlit as st

hub = st.session_state.hub
user = st.session_state.user

def get_haiku_created_display(haiku):
  day = haiku.created_at.strftime('%Y-%m-%d')
  time = haiku.created_at.strftime('%H:%M')
  return f':gray[:material/calendar_month: {day}  \n :material/schedule: {time}]'

def get_haiku_text_display(haiku):
  display_text = haiku.text.replace('\n', '  \n')
  return f':green[{display_text}]'

def edit_button(haiku):
  if st.button(':material/edit:', key=f"edit_{haiku.haiku_id}"):
    pass

def delete_button(haiku):
  if st.button(':material/delete:', key=f"delete_{haiku.haiku_id}"):
    pass

def display_haiku(haiku):
  with st.container(border=True):
    cols = st.columns([2, 5, 1, 1])
    created_col, text_col, edit_col, delete_col = cols

    created_col.markdown(get_haiku_created_display(haiku))
    text_col.markdown(get_haiku_text_display(haiku))
    with edit_col:
      edit_button(haiku)
    with delete_col:
      delete_button(haiku)
```

这里要关注的关键函数是最后一个：`display_haiku`。给定`haiku`，即`Haiku`类的一个实例，它为我们要显示的四个事物创建了四个列：`created_at`、`text`、编辑按钮和删除按钮。

这些的实际渲染在每个自己的函数中完成。

`get_haiku_created_display`接受俳句的`created_at`属性——一个时间戳，并使用名为`strftime`的方法将其分解为日期和时间，该方法用于根据*格式字符串*格式化时间戳。在这种情况下，`%Y-%m-%d`将其格式化为仅日期，而`%H:%M`提取小时和分钟。

```py
f':gray[:material/calendar_month: {day}  \n :material/schedule: {time}]'
```

这里有几个事情在进行。在 Streamlit 中，`:<color>[<text>]`语法用于显示不同颜色的文本。例如，字符串`:red[Hello]`会被`st.write`或`st.markdown`等小部件解释为红色文本的单词`Hello`。

我们还看到了之前看到的图标语法。在这里，我们使用日历图标表示日期，使用时钟图标（`schedule`）表示时间，为日期和时间创建了一个用户友好的显示。

`get_haiku_text_display`的目的是显示俳句的内容。为什么我们有以下替换方法？为什么不直接显示内容呢？

```py
haiku.text.replace('\n', '  \n')
```

这是一种解决方案。Streamlit 处理文本中的换行符相当奇怪。为了使文本小部件如`st.markdown`正确显示换行符`\n`，我们必须在它前面加上两个空格，即`' \n'`而不是仅仅`'\n'`。

`edit_button`和`delete_button`函数简单地显示`st.button`小部件。你会注意到我们使用图标作为它们的标签，并给它们小部件键——这是 Streamlit 在页面上显示许多俳句时区分它们所必需的。我们现在给它们带有`pass`的占位符主体；我们稍后会回来处理它们。

由于我们在整个应用程序中使用了这么多图标，如果我们给它们更好的名字（例如，`CLOCK`而不是`:material/schedule:`）并将它们放在更中心的位置，我们的代码实际上会更容易阅读。

让我们把所有的图标放入它们自己的文件中，`frontend/icons.py`（列表 8.18）并*导入*它们。

##### 列表 8.18 frontend/icons.py

```py
LOGIN = ":material/login:"
SIGNUP = ":material/person_add:"
HOME = ":material/home:"
LOGOUT = ":material/logout:"
ADD = ":material/add_circle:"
CALENDAR = ":material/calendar_month:"
CLOCK = ":material/schedule:"
EDIT = ":material/edit:"
DELETE = ":material/delete:"
```

我们现在可以更改`haiku_display.py`：

```py
import streamlit as st
from frontend.icons import CALENDAR, CLOCK, EDIT, DELETE

...
def get_haiku_created_display(haiku):
  ...
  return f':gray[{CALENDAR} {day}  \n {CLOCK} {time}]'

...
def edit_button(haiku):
  if st.button(f'{EDIT}', key=f"edit_{haiku.haiku_id}"):
    pass

def delete_button(haiku):
  if st.button(f'{DELETE}', key=f"delete_{haiku.haiku_id}"):
    pass
...
```

这样就更容易阅读了！现在让我们调用`display_haiku`来显示用户创建的俳句列表！我们将编辑`home.py`来完成此操作：

```py
import streamlit as st
from frontend.haiku_editor import haiku_editor
from frontend.haiku_display import display_haiku
from frontend.icons import ADD

...
if st.button(f'{ADD} Haiku', type='primary'):
  haiku_editor(hub, user)

haikus = hub.haiku_service.get_haikus_by_author(user.username)
if len(haikus) == 0:
 st.info("You haven't written any haikus yet.")
else:
 for haiku in haikus:
 display_haiku(haiku)
```

这里的更改相当简单。在之前创建的添加按钮之后——我们现在将其更改为使用从`icons.py`导入的图标——我们显示一个标题，说明`<username>'s haikus`，以及一个分隔符（由`divider="gray"`控制的水平线）。

然后，我们调用在`HaikuService`中定义的`get_haikus_by_author`方法，并遍历结果，对每个结果调用`display_haiku`。如果没有俳句，我们显示一个`st.info`消息来说明这一点。

现在重新运行你的应用程序，以查看图 8.12 中的更改（在添加另一个俳句后）！

![图片](img/ch08__image012.png)

##### 图 8.12 列表创建的俳句（在 GitHub 仓库的 chapter_8/in_progress_07 中查看完整代码）

注意，我们还在一个地方使用了图标：`pages.py`，在那里我们定义了多页应用程序的页面。请继续更新该文件：

```py
import streamlit as st
from frontend.icons import LOGIN, LOGOUT, SIGNUP, HOME

pages = {
  "login": st.Page("frontend/login.py", title="Log in", icon=LOGIN),
  "signup": st.Page("frontend/signup.py", title="Sign up", icon=SIGNUP),
  "home": st.Page("frontend/home.py", title="Home", icon=HOME),
  "logout": st.Page("frontend/logout.py", title="Log out", icon=LOGOUT)
}
```

#### 添加更新和删除功能

让我们回到编辑和删除按钮，它们目前下面有占位符。编辑按钮应该允许用户编辑他们现有的俳句之一。我们可以重用我们已创建的俳句编辑器对话框来与“添加俳句”按钮一起使用：

```py
import streamlit as st

@st.dialog("Haiku editor", width="large")
def haiku_editor(hub, user, haiku=None):
  default_text = haiku.text if haiku else ''
  haiku_text = st.text_area('Enter a haiku', value=default_text)
  if st.button('Save haiku', type='primary'):
    if haiku:
 new_haiku = hub.haiku_service.update_haiku(haiku.haiku_id, haiku_text)
 else:
 new_haiku = hub.haiku_service.create_haiku(user.username, haiku_text)
    if new_haiku:
      st.success('Haiku saved successfully!')
      st.rerun()
    else:
      st.error('Failed to save haiku')
```

`haiku_editor`函数现在接受一个`haiku`参数，默认值为`None`。如果我们调用编辑器来编辑现有的俳句，我们可以将相应的`Haiku`实例传递给`haiku`。否则，我们正在调用对话框来添加俳句，因此我们传递`None`。

在函数的其余部分，我们将使用条件`if haiku`来检查我们是在执行编辑操作还是添加操作。

在接下来的两行中，如果我们处于编辑操作，我们将使用`st.text_area`的`value`参数将现有的俳句文本预先填充到文本区域作为默认值。

```py
default_text = haiku.text if haiku else ''
haiku_text = st.text_area('Enter a haiku', value=default_text)
```

然后，一旦点击保存按钮，我们就从 `HaikuService` 中选择 `update_haiku` 或 `create_haiku` 方法来执行。在前者的情况下，我们传递现有俳句的 `haiku_id` 来标识我们想要编辑的俳句。

如果操作成功——我们通过检查返回值 `new_haiku` 来确定——我们发出 `st.rerun()`。这会重新运行整个应用程序，在这个过程中关闭对话框，因为最初触发它的按钮现在处于“未点击”状态。

现在，我们可以替换 `haiku_display.py` 中编辑按钮下的占位符：

```py
def edit_button(haiku):
  if st.button(f'{EDIT}', key=f"edit_{haiku.haiku_id}"):
    haiku_editor(app, user, haiku)
```

在同一页面上，让我们也让删除按钮触发删除操作：

```py
def delete_button(haiku):
  if st.button(f'{DELETE}', key=f"delete_{haiku.haiku_id}"):
    deleted_haiku = app.haiku_service.delete_haiku(haiku.haiku_id)
 if deleted_haiku:
 st.rerun()
 else:
 st.error("Failed to delete haiku.")
```

在这里，我们在 `HaikuService` 中调用 `delete_haiku` 方法。如果删除成功，我们执行 `st.rerun()` 以更新俳句列表并不再显示已删除的俳句。如果由于任何原因删除失败，我们将显示错误。

现在重新运行应用程序并尝试编辑或删除俳句（图 8.13）！

![图片](img/ch08__image013.png)

##### 图 8.13 删除俳句（完整代码请参阅 GitHub 仓库中的 chapter_8/in_progress_08）

俳句天堂现在已经完全建成——至少我们有一个适合部署到生产的版本。不过，在那样做之前，还有一些最终问题需要我们解决。

## 8.6 多用户考虑

虽然“俳句天堂”可以由许多用户同时使用，但我们需要能够在这类用户之间有效地共享资源。我们在这里负责管理的主要资源是数据库。

### 8.6.1 使用 st.cache_resource 共享数据库连接池

让我们考虑当有多个用户同时访问 Streamlit 应用程序时，应用程序是如何工作的。虽然有一个单独的 Streamlit 服务器提供应用程序服务，但每次用户访问它时，都会创建一个新的应用程序实例，为该用户创建所有运行应用程序所需的对象。

大多数时候，我们希望这样；这确保了不同的用户会话不会相互干扰。然而，有些东西我们 *不* 每次有人在新浏览器标签页中加载应用程序时都希望重新创建。

一个关键示例是我们 `Database` 类中创建的数据库连接池。事实上，拥有数据库连接池的全部意义在于，当多个用户会话需要访问数据库时，他们可以通过从 *共享* 池中请求连接来实现，完成使用后将其返回。

这意味着应该只创建 `Database` 类的一个实例，以便所有用户共享。然而，我们还没有这样设置。目前，*每次* 新用户加载应用程序——或者用户在新的浏览器标签页中加载应用程序——就会开始一个新的会话，并创建一个新的 `Hub` 类实例，这意味着也会创建一个新的 `Database` 实例。

幸运的是，Streamlit 提供了一个解决方案：`st.cache_resource`。

就像我们在第六章中使用的 `st.cache_data`——它使我们的度量仪表板中的数据加载更快——`st.cache_resource` 是确保在应用程序的所有用户中只存在一个实例的一种方式。

虽然 `st.cache_data` 用于缓存像 Pandas 数据框或 API 调用结果这样的东西，但 `st.cache_resource` 用于像数据库连接这样的资源。在这种情况下，我们将用它来存储我们想要创建的 `Database` 类的单个实例。

这需要我们对应用程序进行一点重构。回想一下，目前（在 `hub.py` 中）我们的 `Hub` 类接受一个 `config` 对象并自己创建 `Database` 实例：

```py
...
class Hub:
  def __init__(self, config):
    database = Database(config['connection_string'])
    self.user_service = UserService(database)
    self.haiku_service = HaikuService(database)
```

但在我们的新方案中，`Hub` 类将为每个会话创建一个新的实例，这意味着一个新的 `Database` 实例也将持续被创建。相反，我们将让 `Hub` 的 `__init__` 接受一个已经创建的 `Database` 实例，它可以简单地将其传递给 `UserService` 和 `HaikuService`。

这样，我们就可以始终传递 `Database` 类的相同实例，避免每次都创建它。`hub.py` 现在将看起来像这样：

```py
...
class Hub:
  def __init__(self, database):
    self.user_service = UserService(database)
    self.haiku_service = HaikuService(database)
```

那么，我们在哪里创建 `Database` 实例呢？在 `main.py` 中，这是我们使用 `st.cache_resource` 的地方：

```py
...
from backend.database import Database
from frontend.pages import pages

@st.cache_resource
def get_database():
 connection_string = st.secrets['config']['connection_string']
 database = Database(connection_string)
 return database

if 'hub' not in st.session_state:
  st.session_state.hub = Hub(get_database())
...
```

就像 `st.cache_data` 一样，`st.cache_resource` 是一个应用于函数的装饰器。在这里，我们定义了一个新的函数来装饰，称为 `get_database`，它将返回缓存的 `Database` 对象。

`get_database` 现在将只为启动的特定 Streamlit 服务器运行一次，即单个 `streamlit run` 命令——当 `main.py` 首次加载时。对于所有后续运行，`get_database` 将返回缓存的 `Database` 实例，从而确保永远只创建这样一个实例。这个缓存的实例将一直持续到服务器进程终止。

### 8.6.2 使用 atexit 清理数据库连接

我们确保每个 Streamlit 应用服务器运行实例只创建一个 `Database` 实例。这是高效管理数据库资源的一部分。另一部分是安全地清理我们创建的任何连接。

我们的 `Database` 类有一个 `close_all` 方法：

```py
def close_all(self):
  print("Closing all connections...")
  self.connection_pool.closeall()
```

然而，如果你检查我们到目前为止的代码，我们实际上并没有在任何地方调用这个方法。

为了正确清理我们的连接，我们希望这个方法只在 Streamlit 服务器终止时被调用一次。我们如何实现这一点？

解决方案是 `atexit` 模块，它是 Python 内置的。`atexit` 允许你在 Python 解释器即将退出时自动执行函数。

这是我们将如何修改 `main.py` 以使用 `atexit` 注册一个清理数据库连接的函数：

```py
import streamlit as st
import atexit

...
@st.cache_resource
def get_database():
  connection_string = st.secrets['config']['connection_string']
  database = Database(connection_string)
  atexit.register(lambda db: db.close_all(), database)
  return database
...
```

`atexit.register` 接受一个要注册的函数，以及我们想要传递给该函数的任何参数的值。在上面的代码中，我们注册的函数是一个单行 lambda 函数：`lambda db: db.close_all()`。

它接受一个参数——`db`，这是数据库实例。这个函数所做的只是调用 `db` 的 `close_all` 方法。我们传递给 `atexit.register` 的第二个参数是 `database`，这是上面一行创建的 `Database` 实例。

调用 `atexit.register` “安排”一个函数在 Python 本身退出时执行，在 `streamlit run` 的情况下，这发生在 Streamlit 服务器关闭时。

为什么我们把 `atexit.register` 的调用放在 `get_database` 中？为什么不放在文件的其他地方，比如文件末尾？嗯，就像创建 `Database` 对象一样，我们只想在整个用户中执行一次 `atexit` 函数的注册——因为只有一个 Streamlit 服务器。这意味着我们必须在带有 `st.cache_resource` 装饰器的函数中调用 `atexit.register`，否则这个注册将会多次发生。

要查看这一变化的效果，尝试使用 `streamlit run` 重新启动应用服务器，然后按 `Ctrl+C` 退出它。您应该会看到消息 `Closing all connections...`，这表明已经调用了 `close_all`。

## 8.7 部署 Haiku Haven

由于 Haiku Haven 在本地已经运行，现在是时候在 Community Cloud 上将我们的应用投入生产。这个过程与我们在第五章以来一直遵循的过程相同，但这里有一个额外的复杂因素：我们的应用需要一个正在运行的 PostgreSQL 服务器来托管我们的数据库。

### 8.7.1 在生产中设置托管 PostgreSQL 服务器

在我们本地开发时，在相同机器上安装 PostgreSQL 是一件简单的事情，但 Streamlit Community Cloud 并不提供这样的选项。相反，我们需要在某个地方设置一个外部 PostgreSQL 服务器。

我们将使用一个名为 Neon 的基于云的托管 PostgreSQL 服务，这使得这个过程变得非常简单，并且有显著的免费配额。现在就使用 Neon 创建一个账户，网址为 `https://neon.tech/`。注册过程相当简单；如果您愿意，可以选择使用 GitHub 或 Google 账户注册。

您将被要求输入一个项目名称和一个数据库名称。项目名称可以是您喜欢的任何名称（例如 `Haiku Haven`？），而数据库名称应该是您在本地 Postgres 中命名数据库的名称——如果您一直忠实地跟随，那么应该是 `haikudb`。

您可能还会被要求选择一个云提供商和位置——这些可以是您喜欢的任何内容，尽管我选择了 AWS 作为提供商。

一旦您的账户设置好，导航到快速入门页面，您会看到一个看起来像这样的连接字符串：

```py
postgresql://haikudb_owner:Dxg2HFXreSZ3@ep-flower-dust-a63e8evn.us-west-2.aws.neon.tech/haikudb?sslmode=require
```

这是我们在生产中使用的连接字符串。Neon 已经为您设置了一个用户名（`haikudb_owner`）和密码。请将此字符串保存在安全的地方。

接下来，你需要在 Neon 中再次设置 `users` 和 `haikus` 表。为此，转到 SQL 编辑器标签。这就是你可以像在 `psql` 提示符中一样输入 SQL 命令的地方。要创建表，请参考第 8.2.4 节并获取我们本地执行的 `CREATE TABLE` 命令。你可以在 Neon 的 SQL 编辑器中执行这些命令，无需任何更改。

### 8.7.2 部署到社区云

部署过程的其余部分应该很简单，基本上与我们在第五章中做的是一样的。

确保创建一个 `requirements.txt` 文件，这样社区云就会知道安装所需的第三方模块——主要是 `psycopg2` 和 `bcrypt`。

为了参考，列表 8.19 显示了我使用的 requirements.txt。

##### 列表 8.19 requirements.txt

```py
streamlit==1.40.2
psycopg2-binary==2.9.10
bcrypt==4.2.0
```

注意，我使用了 `psycopg2-binary` 而不是 `psycopg2`；这是因为当我尝试使用后者时，社区云抛出了一个错误，但使用前者时没有。

完成部署后，你需要复制 `secrets.toml` 的内容，并将其粘贴到社区云的秘密设置中（参考第五章以刷新记忆），用从 Neon 复制的连接字符串替换连接字符串。

就这些了，朋友们！Haiku Haven 现已上线！告诉你的所有朋友，他们可以在你全新的网络应用上释放他们十七音节的创造力！

至于我们，让我们翻过 CRUD 一页，尝试构建 AI 应用程序。

## 8.8 摘要

+   CRUD 代表创建-读取-更新-删除，这是大多数应用执行的四项基本操作。

+   关系型数据库如 PostgreSQL 根据模式在表中组织数据，具有行和列。

+   设计应用程序的数据模型涉及识别实体、定义它们之间的关系、列出它们的属性，并将这些转换为模式，通常在实体-关系（ER）图的辅助下完成。

+   SQL（结构化查询语言）支持创建表（`CREATE TABLE`）、插入行（`INSERT INTO`）、读取数据（`SELECT..FROM`）、更新行（`UPDATE..SET`）、删除行（`DELETE FROM`）和删除表（`DROP TABLE`）的命令。

+   `psycopg2` 是一个用于通过共享连接池连接到 PostgreSQL 的 Python 模块。

+   永远不要以纯文本形式存储密码；相反，使用像 `bcrypt` 这样的库对它们进行散列，并存储散列版本。为了进行身份验证，散列给定的密码并与存储的散列进行比较。

+   `st.Page` 对象对应于 Streamlit 中多页应用中的单个页面。

+   `st.navigation` 用于创建导航栏并指定应用中的页面。

+   `st.page_link` 在多页应用中创建页面之间的链接。

+   `st.cache_resource` 用于缓存资源，如数据库连接，并在用户之间共享。

+   使用内置的 `atexit` 模块的 `atexit.register` 来注册一个在 Streamlit 服务器关闭时执行的函数。

+   当部署到生产环境时，你需要单独设置你的数据库服务器，可能使用 Neon 这样的托管服务来处理 PostgreSQL。
