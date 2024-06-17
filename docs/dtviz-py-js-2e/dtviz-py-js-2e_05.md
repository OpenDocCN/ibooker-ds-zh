# 第三章\. 使用Python读写数据

任何数据可视化专家的基本技能之一是能够移动数据。无论您的数据是在SQL数据库中，CSV文件中，还是其他更奇特的形式中，您都应该能够舒适地读取数据，转换数据，并在需要时将其写入更方便的格式。Python在这方面的一个强大功能就是使得这样的数据操作变得异常简单。本章的重点是让您迅速掌握我们数据可视化工具链中这一关键部分。

本章既是教程，又是参考资料的一部分，并且后续章节将参考本章的部分内容。如果您了解Python数据读写的基础知识，您可以挑选本章的部分内容作为复习。

# 容易上手

我还记得当年我开始编程（使用像C这样的低级语言），数据操作是多么的笨拙。从文件中读取和写入数据是样板式代码、临时 improvisations 等的令人讨厌的混合体。从数据库中读取同样困难，至于序列化数据，回忆起来仍然痛苦。发现Python就像是一阵清新的空气。它并非速度的鬼才，但打开一个文件几乎就是可以做到的最简单的事情了：

```py
file = open('data.txt')
```

那时候，Python让从文件中读取和写入数据变得令人耳目一新，它复杂的字符串处理功能也使得解析这些文件中的数据变得同样简单。它甚至有一个叫做Pickle的神奇模块，可以序列化几乎任何Python对象。

近年来，Python已经在其标准库中添加了强大成熟的模块，使得处理CSV和JSON文件（Web数据可视化工作的标准）变得同样容易。还有一些很棒的库可以与SQL数据库进行交互，比如SQLAlchemy，我强烈推荐使用。新型NoSQL数据库也得到了很好的服务。MongoDB是这些新型基于文档的数据库中最流行的，Python的PyMongo库（稍后在本章中演示）使得与它的交互相对轻松。

# 传递数据

展示如何使用关键数据存储库的一个好方法是在它们之间传递单个数据包，边读取边写入。这将让我们有机会看到数据可视化器使用的关键数据格式和数据库的实际操作。

我们将要传递的数据可能是在Web可视化中最常用的，是一组类似于字典的对象列表（见[示例 3-1](#data_dummydata)）。这个数据集以[JSON格式](https://oreil.ly/JgjAp)传输到浏览器，正如我们将看到的那样，可以轻松地从Python字典转换过来。

##### 示例 3-1\. 我们的目标数据对象列表

```py
nobel_winners = [
 {'category': 'Physics',
  'name': 'Albert Einstein',
  'nationality': 'Swiss',
  'gender': 'male',
  'year': 1921},
 {'category': 'Physics',
  'name': 'Paul Dirac',
  'nationality': 'British',
  'gender': 'male',
  'year': 1933},
 {'category': 'Chemistry',
  'name': 'Marie Curie',
  'nationality': 'Polish',
  'gender': 'female',
  'year': 1911}
]
```

我们将从创建一个CSV文件开始，以Python列表的形式显示[示例 3-1](#data_dummydata)，作为打开和写入系统文件的演示。

以下各节假定您处于具有*data*子目录的工作（根）目录中。您可以从Python解释器或文件中运行代码。

# 使用系统文件

在本节中，我们将从Python字典列表（[示例 3-1](#data_dummydata)）创建一个CSV文件。通常，您会使用`csv`模块来执行此操作，我们将在此节后演示，因此这只是演示基本的Python文件操作的一种方式。

首先，让我们打开一个新文件，使用`w`作为第二个参数表示我们将向其写入数据。

```py
f = open('data/nobel_winners.csv', 'w')
```

现在，我们将从`nobel_winners`字典（[示例 3-1](#data_dummydata)）创建我们的CSV文件：

```py
cols = nobel_winners[0].keys() ![1](assets/1.png)
cols = sorted(cols) ![2](assets/2.png)

with open('data/nobel_winners.csv', 'w') as f: ![3](assets/3.png)
    f.write(','.join(cols) + '\n') ![4](assets/4.png)

    for o in nobel_winners:
        row = [str(o[col]) for col in cols] ![5](assets/5.png)
        f.write(','.join(row) + '\n')
```

[![1](assets/1.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO1-1)

从第一个对象的键（即`['category', 'name', ... ]`）获取我们的数据列。

[![2](assets/2.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO1-2)

按字母顺序对列进行排序。

[![3](assets/3.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO1-3)

使用Python的`with`语句来保证在离开块或发生任何异常时关闭文件。

[![4](assets/4.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO1-4)

`join`从字符串列表（这里是`cols`）创建一个连接字符串，由初始字符串（即“category,name,..”）连接。

[![5](assets/5.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO1-5)

使用列键创建对象中的列表（`nobel_winners`）。

现在我们已经创建了我们的CSV文件，让我们使用Python读取它，确保一切都正确：

```py
with open('data/nobel_winners.csv') as f:
    for line in f.readlines():
        print(line)

Out:
category,name,nationality,gender,year
Physics,Albert Einstein,Swiss,male,1921
Physics,Paul Dirac,British,male,1933
Chemistry,Marie Curie,Polish,female,1911
```

如前面的输出所示，我们的CSV文件格式正确。让我们使用Python内置的`csv`模块首先读取它，然后正确创建CSV文件。

# CSV、TSV和行列数据格式

逗号分隔值（CSV）或其制表符分隔的同类（TSV）可能是最普遍的基于文件的数据格式，作为数据可视化者，这些通常是你会收到的形式，用于处理你的数据。能够读取和写入CSV文件及其各种古怪的变体，比如以管道或分号分隔，或者使用*`*代替标准双引号的格式，是一项基本技能；Python的`csv`模块能够在这里做几乎所有的繁重工作。让我们通过读写我们的`nobel_winners`数据来展示它的功能：

```py
nobel_winners = [
 {'category': 'Physics',
  'name': 'Albert Einstein',
  'nationality': 'Swiss',
  'gender': 'male',
  'year': 1921},
  ...
]
```

将我们的`nobel_winners`数据（见[示例 3-1](#data_dummydata)）写入CSV文件非常简单。`csv`有一个专门的`DictWriter`类，它会将我们的字典转换为CSV行。我们唯一需要做的显式记录就是写入CSV文件的标题，使用我们字典的键作为字段（即“category, name, nationality, gender”）：

```py
import csv

with open('data/nobel_winners.csv', 'w') as f:
    fieldnames = nobel_winners[0].keys() ![1](assets/1.png)
    fieldnames = sorted(fieldnames) ![2](assets/2.png)
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader() ![3](assets/3.png)
    for w in nobel_winners:
        writer.writerow(w)
```

[![1](assets/1.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO2-1)

您需要明确告知写入器要使用哪些 `fieldnames`（在本例中是 `'category'`、`'name'` 等键）。

[![2](assets/2.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO2-2)

我们将 CSV 标题字段按字母顺序排序以提高可读性。

[![3](assets/3.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO2-3)

写入 CSV 文件的标题（“category, name,…​”）。

您可能经常读取 CSV 文件，而不是写入它们。^([1](ch03.xhtml#idm45607798394048)) 让我们读取刚刚写入的 *nobel_winners.csv* 文件。

如果您只想将 `csv` 作为一个优秀且非常适应的文件行读取器使用，几行代码将生成一个便捷的迭代器，可以将您的 CSV 行作为字符串列表传递：

```py
with open('data/nobel_winners.csv') as f:
    reader = csv.reader(f)
    for row in reader: ![1](assets/1.png)
        print(row)

Out:
['category', 'name', 'nationality', 'gender', 'year']
['Physics', 'Albert Einstein', 'Swiss', 'male', '1921']
['Physics', 'Paul Dirac', 'British', 'male', '1933']
['Chemistry', 'Marie Curie', 'Polish', 'female', '1911']
```

[![1](assets/1.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO3-1)

遍历`reader`对象，消耗文件中的行。

注意数字以字符串形式读取。如果要对其进行数值操作，需要将任何数值列转换为其相应的类型，这种情况下是整数年份。

更方便地消耗 CSV 数据的方法是将行转换为 Python 字典。这种*record*形式也是我们作为转换目标（`list` of `dict`）使用的形式。`csv` 提供了一个方便的 `DictReader` 就是为了这个目的：

```py
import csv

with open('data/nobel_winners.csv') as f:
    reader = csv.DictReader(f)
    nobel_winners = list(reader) ![1](assets/1.png)

nobel_winners

Out:
[OrderedDict([('category', 'Physics'),
              ('name', 'Albert Einstein'),
              ('nationality', 'Swiss'),
              ('gender', 'male'),
              ('year', '1921')]),
 OrderedDict([('category', 'Physics'),
              ('name', 'Paul Dirac'),
              ('nationality', 'British'),
              ... ])]
```

[![1](assets/1.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO4-1)

将所有 `reader` 项插入列表中。

正如输出所示，我们只需将 `dict` 的年份属性转换为整数，就可以使 `nobel_winners` 符合本章的目标数据（[示例 3-1](#data_dummydata)），如下：

```py
for w in nobel_winners:
    w['year'] = int(w['year'])
```

为了更灵活地创建 Python `datetime`，我们可以轻松从年份列创建它：

```py
from datetime import datetime

dt = datetime.strptime('1947', '%Y')
dt
# datetime.datetime(1947, 1, 1, 0, 0)
```

`csv`读取器不会从你的文件中推断数据类型，而是将所有内容解释为字符串。pandas，Python 中领先的数据处理库，会尝试猜测数据列的正确类型，通常能够成功。我们将在后面专门讲解 pandas 的章节中看到这一点。

`csv` 有一些有用的参数来帮助解析 CSV 家族的成员：

`dialect`

默认情况下，`'excel'`；指定了一组特定于方言的参数。`excel-tab` 是一种偶尔使用的替代方式。

`delimiter`

文件通常是逗号分隔的，但也可以使用 `|`、`:` 或 `' '`。

`quotechar`

默认情况下，使用双引号，但偶尔会使用 `|` 或 `` ` `` 替代。

您可以在 [在线 Python 文档](https://oreil.ly/9zZvt) 中找到完整的 `csv` 参数集。

现在我们已经成功地使用 `csv` 模块编写和读取了目标数据，让我们将我们的基于 CSV 的 `nobel_winners` `dict` 传递给 `json` 模块。

# JSON

在本节中，我们将使用 Python 的 `json` 模块编写和读取我们的 `nobel_winners` 数据。让我们回顾一下我们使用的数据：

```py
nobel_winners = [
 {'category': 'Physics',
  'name': 'Albert Einstein',
  'nationality': 'Swiss',
  'gender': 'male',
  'year': 1921},
  ...
]
```

对于诸如字符串、整数和浮点数等数据原语，可以使用Python字典将其轻松保存（或在JSON术语中*转储*）到JSON文件中，使用`json`模块。`dump`方法接受一个Python容器和一个文件指针，将前者保存到后者：

```py
import json

with open('data/nobel_winners.json', 'w') as f:
     json.dump(nobel_winners, f)

open('data/nobel_winners.json').read()
```

```py
Out: '[{"category": "Physics", "name": "Albert Einstein",
"gender": "male", "year": 1921,
"nationality": "Swiss"}, {"category": "Physics",
"nationality": "British", "year": 1933, "name": "Paul Dirac",
"gender": "male"}, {"category": "Chemistry", "nationality":
"Polish", "year": 1911, "name": "Marie Curie", "gender":
"female"}]'
```

读取（或加载）JSON文件同样简单。我们只需将打开的JSON文件传递给`json`模块的`load`方法即可：

```py
import json

with open('data/nobel_winners.json') as f:
    nobel_winners = json.load(f)

nobel_winners
Out:
[{'category': 'Physics',
  'name': 'Albert Einstein',
  'nationality': 'Swiss',
  'gender': 'male',
  'year': 1921}, ![1](assets/1.png)
... }]
```

[![1](assets/1.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO5-1)

请注意，与CSV文件转换不同，年列的整数类型被保留。

`json`具有方法`loads`和`dumps`，它们分别对应于文件访问方法，将JSON字符串加载到Python容器中，并将Python容器转储为JSON字符串。

## 处理日期和时间

尝试将`datetime`对象转储为`json`会产生`TypeError`：

```py
from datetime import datetime

json.dumps(datetime.now())
Out:
...
TypeError: datetime.datetime(2021, 9, 13, 10, 25, 52, 586792)
is not JSON serializable
```

当序列化诸如字符串或数字之类的简单数据类型时，默认的`json`编码器和解码器效果很好。但对于诸如日期之类的更专门化数据，您需要自己进行编码和解码。这并不像听起来那么难，并且很快就会变得日常。让我们首先看一下如何将您的Python [`datetime`s](https://oreil.ly/aHI4h)编码为明智的JSON字符串。

编码Python数据中包含`datetime`的最简单方法是创建一个自定义编码器，就像在[示例 3-2](#data_json_time)中所示的一样，它作为`json.dumps`方法的`cls`参数提供。该编码器依次应用于数据中的每个对象，并将日期或日期时间转换为其ISO格式字符串（参见[“处理日期、时间和复杂数据”](#sect_datetimes)）。

##### 示例 3-2\. 将Python `datetime`编码为JSON

```py
import datetime
import json

class JSONDateTimeEncoder(json.JSONEncoder): ![1](assets/1.png)
    def default(self, obj):
        if isinstance(obj, (datetime.date, datetime.datetime)): ![2](assets/2.png)
            return obj.isoformat()
        else:
            return json.JSONEncoder.default(self, obj)

def dumps(obj):
    return json.dumps(obj, cls=JSONDateTimeEncoder) ![3](assets/3.png)
```

[![1](assets/1.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO6-1)

为了创建一个定制的日期处理类，需要从`JSONEncoder`派生子类。

[![2](assets/2.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO6-2)

测试`datetime`对象，如果为真，则返回任何日期或日期时间的`isoformat`（例如，2021-11-16T16:41:14.650802）。

[![3](assets/3.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO6-3)

使用`cls`参数来设置自定义日期编码器。

让我们看看我们的新`dumps`方法如何处理一些`datetime`数据：

```py
now_str = dumps({'time': datetime.datetime.now()})
now_str
Out:
'{"time": "2021-11-16T16:41:14.650802"}'
```

`time`字段已正确转换为ISO格式字符串，准备解码为JavaScript的`Date`对象（参见[“处理日期、时间和复杂数据”](#sect_datetimes)进行演示）。

虽然您可以编写一个通用解码器来处理任意JSON文件中的日期字符串，^([2](ch03.xhtml#idm45607797415616))但这可能并不明智。日期字符串有各种各样的奇特变体，最好手动处理几乎总是已知的数据集。

可靠的 `strptime` 方法，属于 `datetime.datetime` 包，非常适合将已知格式的时间字符串转换为 Python 的 `datetime` 实例：

```py
In [0]: from datetime import datetime

In [1]: time_str = '2021/01/01 12:32:11'

In [2]: dt = datetime.strptime(time_str, '%Y/%m/%d %H:%M:%S') ![1](assets/1.png)

In [3]: dt
Out[2]: datetime.datetime(2021, 1, 1, 12, 32, 11)
```

[![1](assets/1.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO7-1)

`strptime` 尝试使用各种指令（例如 `%Y`（带世纪的年份）和 `%H`（零填充的小时））将时间字符串与格式字符串匹配。如果成功，它会创建一个 Python 的 `datetime` 实例。请参阅 [Python 文档](https://oreil.ly/Fi40k) 查看所有可用指令的完整列表。

如果 `strptime` 被传入一个与其格式不匹配的时间字符串，它会抛出一个便捷的 `ValueError`：

```py
dt = datetime.strptime('1/2/2021 12:32:11', '%Y/%m/%d %H:%M:%S')
-----------------------------------------------------------
ValueError                Traceback (most recent call last)
<ipython-input-111-af657749a9fe> in <module>()
----> 1 dt = datetime.strptime('1/2/2021 12:32:11',\
    '%Y/%m/%d %H:%M:%S')
...
ValueError: time data '1/2/2021 12:32:11' does not match
            format '%Y/%m/%d %H:%M:%S'
```

因此，要将已知格式的日期字段转换为 `datetime`，并应用于一个由字典组成的 `data` 列表，您可以像这样做：

```py
data = [
    {'id': 0, 'date': '2020/02/23 12:59:05'},
    {'id': 1, 'date': '2021/11/02 02:32:00'},
    {'id': 2, 'date': '2021/23/12 09:22:30'},
]

for d in data:
     try:
         d['date'] = datetime.strptime(d['date'],\
           '%Y/%m/%d %H:%M:%S')
     except ValueError:
         print('Oops! - invalid date for ' + repr(d))
# Out:
# Oops! - invalid date for {'id': 2, 'date': '2021/23/12 09:22:30'}
```

现在我们已经处理了两种最流行的数据文件格式，让我们转向重点，看看如何从 SQL 和 NoSQL 数据库中读取和写入数据。

# SQL

对于与 SQL 数据库交互，SQLAlchemy 是最流行且在我看来是最好的 Python 库。它允许您在速度和效率是问题的情况下使用原始 SQL 指令，同时提供一个强大的对象关系映射（ORM），使您能够使用高级、Pythonic API 操作 SQL 表，本质上将其视为 Python 类。

使用 SQL 进行数据读取和写入，同时允许用户将这些数据视为 Python 容器是一个复杂的过程，尽管 SQLAlchemy 比低级 SQL 引擎更加用户友好，但它仍然是一个相当复杂的库。我将在这里介绍基础知识，以我们的数据作为目标，但建议您花点时间阅读关于 [SQLAlchemy](https://oreil.ly/mCHr8) 的出色文档。让我们先回顾一下我们打算读取和写入的 `nobel_winners` 数据集：

```py
nobel_winners = [
 {'category': 'Physics',
  'name': 'Albert Einstein',
  'nationality': 'Swiss',
  'gender': 'male',
  'year': 1921},
  ...
]
```

首先，让我们使用 SQLAlchemy 将目标数据写入 SQLite 文件，开始创建数据库引擎。

## 创建数据库引擎

在开始 SQLAlchemy 会话时，第一件事是创建一个数据库引擎。该引擎将与所需的数据库建立连接，并对SQLAlchemy生成的通用SQL指令和返回的数据执行任何所需的转换。

几乎每种流行的数据库都有相应的引擎，还有一个 *memory* 选项，将数据库保存在 RAM 中，用于快速访问测试。^([3](ch03.xhtml#idm45607796976800)) 这些引擎的伟大之处在于它们是可互换的，这意味着您可以使用方便的基于文件的 SQLite 数据库开发代码，然后通过更改单个配置字符串在生产环境中切换到一些更工业化的选项，如 PostgreSQL。请查看 [SQLAlchemy](https://oreil.ly/QmIj6) 获取可用引擎的完整列表。

指定数据库 URL 的格式如下：

```py
dialect+driver://username:password@host:port/database
```

因此，要连接到运行在本地主机上的 `'nobel_winners'` MySQL 数据库，需要类似以下方式。请注意，在此时 `create_engine` 并没有真正发出任何 SQL 请求，而只是为执行这些请求设置了框架:^([4](ch03.xhtml#idm45607796966400))

```py
engine = create_engine(
           'mysql://kyran:mypsswd@localhost/nobel_winners')
```

我们将使用基于文件的 SQLite 数据库，并将 `echo` 参数设置为 `True`，这样 SQLAlchemy 生成的任何 SQL 指令都会输出。请注意冒号后面的三个反斜杠的用法：

```py
from sqlalchemy import create_engine

engine = create_engine(
            'sqlite:///data/nobel_winners.db', echo=True)
```

SQLAlchemy 提供了多种与数据库交互的方式，但我建议使用更近代的声明式风格，除非有充分理由选择更低级和细粒度的方法。本质上，使用声明式映射，您可以从一个基类子类化您的 Python SQL 表类，并让 SQLAlchemy 自动检查它们的结构和关系。详细信息请参阅 [SQLAlchemy](https://oreil.ly/q3IZf)。

## 定义数据库表

我们首先使用 `declarative_base` 创建一个 `Base` 类。这个基类将用于创建表类，从而让 SQLAlchemy 创建数据库的表结构。您可以使用这些表类以相当 Pythonic 的方式与数据库交互：

```py
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
```

注意，大多数 SQL 库要求您正式定义表结构。这与无模式 NoSQL 变体如 MongoDB 相反。本章后面我们将看到 Dataset 库，它支持无模式 SQL。

使用这个 `Base`，我们定义我们的各种表，例如我们的单个 `Winner` 表。[示例 3-3](#data_sql_base) 展示了如何子类化 `Base` 并使用 SQLAlchemy 的数据类型来定义表结构。请注意 `__tablename__` 成员，它将用于命名 SQL 表和作为检索它的关键字，还有可选的自定义 `__repr__` 方法，用于在打印表行时使用。

##### 示例 3-3\. 定义 SQL 数据库表

```py
from sqlalchemy import Column, Integer, String, Enum
// ...

class Winner(Base):
    __tablename__ = 'winners'
    id = Column(Integer, primary_key=True)
    category = Column(String)
    name = Column(String)
    nationality = Column(String)
    year = Column(Integer)
    gender = Column(Enum('male', 'female'))
    def __repr__(self):
        return "<Winner(name='%s', category='%s', year='%s')>"\
%(self.name, self.category, self.year)
```

在 [示例 3-3](#data_sql_base) 中声明了我们的 `Base` 子类后，我们使用其 `metadata` 的 `create_all` 方法和我们的数据库引擎来创建我们的数据库。因为在创建引擎时设置了 `echo` 参数为 `True`，所以我们可以从命令行看到 SQLAlchemy 生成的 SQL 指令：

```py
Base.metadata.create_all(engine)

2021-11-16 17:58:34,700 INFO sqlalchemy.engine.Engine BEGIN (implicit)
...
CREATE TABLE winners (
	id INTEGER NOT NULL,
	category VARCHAR,
	name VARCHAR,
	nationality VARCHAR,
	year INTEGER,
	gender VARCHAR(6),
	PRIMARY KEY (id)
)...
2021-11-16 17:58:34,742 INFO sqlalchemy.engine.Engine COMMIT
```

使用我们新声明的 `winners` 表，我们可以开始向其中添加获奖者实例。

## 使用会话添加实例

现在我们已经创建了我们的数据库，我们需要一个会话来进行交互：

```py
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)
session = Session()
```

现在我们可以使用我们的 `Winner` 类创建实例和表行，并将它们添加到会话中：

```py
albert = Winner(**nobel_winners[0]) ![1](assets/1.png)
session.add(albert)
session.new ![2](assets/2.png)
Out:
IdentitySet([<Winner(name='Albert Einstein', category='Physics',
             year='1921')>])
```

[![1](assets/1.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO8-1)

Python 的便捷 ** 操作符将我们的第一个 `nobel_winners` 成员解包为键值对：`(name='Albert Einstein', category='Physics'...)`。

[![2](assets/2.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO8-2)

`new` 是任何已添加到此会话中的项目的集合。

注意所有的数据库插入和删除都是在 Python 中进行的。只有当我们使用 `commit` 方法时，数据库才会被修改。

###### 提示

尽可能少地提交，允许 SQLAlchemy 在后台完成其工作。当您提交时，SQLAlchemy 应该将您的各种数据库操作总结起来，并以高效的方式进行通信。提交涉及建立数据库握手和协商事务，这通常是一个缓慢的过程，您应尽可能地限制提交，充分利用 SQLAlchemy 的簿记能力。

如 `new` 方法所示，我们已将一个 `Winner` 添加到会话中。我们可以使用 `expunge` 移除对象，留下一个空的 `IdentitySet`：

```py
session.expunge(albert) ![1](assets/1.png)
session.new
Out:
IdentitySet([])
```

[![1](assets/1.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO9-1)

从会话中移除实例（还有一个 `expunge_all` 方法，用于移除会话中添加的所有新对象）。

此时，还没有进行任何数据库插入或删除操作。让我们将我们的 `nobel_winners` 列表中的所有成员添加到会话并提交到数据库：

```py
winner_rows = [Winner(**w) for w in nobel_winners]
session.add_all(winner_rows)
session.commit()
Out:
INFO:sqlalchemy.engine.base.Engine:BEGIN (implicit)
...
INFO:sqlalchemy.engine.base.Engine:INSERT INTO winners (name,
category, year, nationality, gender) VALUES (?, ?, ?, ?, ?)
INFO:sqlalchemy.engine.base.Engine:('Albert Einstein',
'Physics', 1921, 'Swiss', 'male')
...
INFO:sqlalchemy.engine.base.Engine:COMMIT
```

现在我们已将我们的 `nobel_winners` 数据提交到数据库，让我们看看我们可以用它做些什么以及如何在 [Example 3-1](#data_dummydata) 中重新创建目标列表。

## 查询数据库

要访问数据，您可以使用 `session` 的 `query` 方法，其结果可以进行过滤、分组和交集操作，允许完整范围的标准 SQL 数据检索。您可以在 [SQLAlchemy 文档](https://oreil.ly/2rEB4) 中查看可用的查询方法。现在，让我快速浏览一下我们诺贝尔数据集上一些最常见的查询。

让我们首先计算一下我们获奖者表中的行数：

```py
session.query(Winner).count()
Out:
3
```

接下来，让我们检索所有瑞士获奖者：

```py
result = session.query(Winner).filter_by(nationality='Swiss') ![1](assets/1.png)
list(result)
Out:
[<Winner(name='Albert Einstein', category='Physics',\
  year='1921')>]
```

[![1](assets/1.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO10-1)

`filter_by` 使用关键字表达式；它的 SQL 表达式对应物是 `filter` ——例如，`filter(Winner.nationality == *Swiss*)`。请注意在 `filter` 中使用的布尔等价 `==`。

现在让我们获取所有非瑞士物理学获奖者：

```py
result = session.query(Winner).filter(\
             Winner.category == 'Physics', \
             Winner.nationality != 'Swiss')
list(result)
Out:
[<Winner(name='Paul Dirac', category='Physics', year='1933')>]
```

根据 ID 号获取行的方法如下：

```py
session.query(Winner).get(3)
Out:
<Winner(name='Marie Curie', category='Chemistry', year='1911')>
```

现在让我们按年份排序获取获奖者：

```py
res = session.query(Winner).order_by('year')
list(res)
Out:
[<Winner(name='Marie Curie', category='Chemistry',\
year='1911')>,
 <Winner(name='Albert Einstein', category='Physics',\
year='1921')>,
 <Winner(name='Paul Dirac', category='Physics', year='1933')>]
```

当我们通过会话查询返回的 `Winner` 对象转换为 Python `dict` 时，重建我们的目标列表需要一些努力。让我们写一个小函数来创建一个从 SQLAlchemy 类到 `dict` 的映射。我们将使用一些表内省来获取列标签（参见 [Example 3-4](#data_to_dict)）。

##### Example 3-4\. 将 SQLAlchemy 实例转换为 `dict`

```py
def inst_to_dict(inst, delete_id=True):
    dat = {}
    for column in inst.__table__.columns: ![1](assets/1.png)
        dat[column.name] = getattr(inst, column.name)
    if delete_id:
        dat.pop('id') ![2](assets/2.png)
    return dat
```

[![1](assets/1.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO11-1)

访问实例的表类以获取列对象的列表。

[![2](assets/2.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO11-2)

如果 `delete_id` 为 true，则删除 SQL 主 ID 字段。

我们可以使用 [示例 3-4](#data_to_dict) 重建我们的 `nobel_winners` 目标列表：

```py
winner_rows = session.query(Winner)
nobel_winners = [inst_to_dict(w) for w in winner_rows]
nobel_winners
Out:
[{'category': 'Physics',
  'name': 'Albert Einstein',
  'nationality': 'Swiss',
  'gender': 'male',
  'year': 1921},
  ...
]
```

你可以通过更改其反映对象的属性轻松更新数据库行：

```py
marie = session.query(Winner).get(3) ![1](assets/1.png)
marie.nationality = 'French'
session.dirty ![2](assets/2.png)
Out:
IdentitySet([<Winner(name='Marie Curie', category='Chemistry',
year='1911')>])
```

[![1](assets/1.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO12-1)

获取 Marie Curie，国籍为波兰。

[![2](assets/2.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO12-2)

`dirty` 显示尚未提交到数据库的任何已更改实例。

让我们提交 `Marie` 的更改，并检查她的国籍是否从波兰变为法国：

```py
session.commit()
Out:
INFO:sqlalchemy.engine.base.Engine:UPDATE winners SET
nationality=? WHERE winners.id = ?
INFO:sqlalchemy.engine.base.Engine:('French', 3)
...

session.dirty
Out:
IdentitySet([])

session.query(Winner).get(3).nationality
Out:
'French'
```

除了更新数据库行外，你还可以删除查询结果：

```py
session.query(Winner).filter_by(name='Albert Einstein').delete()
Out:
INFO:sqlalchemy.engine.base.Engine:DELETE FROM winners WHERE
winners.name = ?
INFO:sqlalchemy.engine.base.Engine:('Albert Einstein',)
1

list(session.query(Winner))
Out:
[<Winner(name='Paul Dirac', category='Physics', year='1933')>,
 <Winner(name='Marie Curie', category='Chemistry',\
 year='1911')>]
```

如果需要，你也可以使用声明类的 `__table__` 属性删除整个表：

```py
Winner.__table__.drop(engine)
```

在本节中，我们处理了一个单独的获奖者表，没有任何外键或与其他表的关系，类似于 CSV 或 JSON 文件。SQLAlchemy 提供了相同的方便程度，用于处理多对一、一对多和其他数据库表关系，就像处理使用隐式连接进行基本查询一样，通过为查询提供多个表类或显式使用查询的 `join` 方法。请查看 SQLAlchemy 文档中的示例以获取更多详细信息。

## 使用 Dataset 更轻松的 SQL

我最近发现自己经常使用的一个库是 [Dataset](https://oreil.ly/aGqTL)，这是一个设计用于使与 SQL 数据库的交互比现有的强大工具如 SQLAlchemy 更容易和更符合 Python 风格的模块。 Dataset 尝试提供与无模式 NoSQL 数据库（如 MongoDB）工作时获得的便利程度相同，通过移除许多更传统库要求的形式化样板代码，比如模式定义。Dataset 建立在 SQLAlchemy 之上，这意味着它可以与几乎所有主要数据库一起工作，并且可以利用该领域最佳的库的功能、健壮性和成熟性。让我们看看它如何处理读取和写入我们的目标数据集（来自 [示例 3-1](#data_dummydata)）。

让我们使用刚刚创建的 SQLite *nobel_winners.db* 数据库来测试 Dataset 的功能。首先，我们连接到我们的 SQL 数据库，使用与 SQLAlchemy 相同的 URL/文件格式：

```py
import dataset

db = dataset.connect('sqlite:///data/nobel_winners.db')
```

要获取我们的获奖者列表，我们从我们的 `db` 数据库中获取一个表，使用其名称作为键，然后使用不带参数的 `find` 方法返回所有获奖者：

```py
wtable = db['winners']
winners = wtable.find()
winners = list(winners)
winners
#Out:
#[OrderedDict([(u'id', 1), ('name', 'Albert Einstein'),
# ('category', 'Physics'), ('year', 1921), ('nationality',
# 'Swiss'), ('gender', 'male')]), OrderedDict([('id', 2),
# ('name', 'Paul Dirac'), ('category', 'Physics'),
# ('year', 1933), ('nationality', 'British'), ('gender',
# 'male')]), OrderedDict([('id', 3), ('name', 'Marie
# Curie'), ('category', 'Chemistry'), ('year', 1911),
# ('nationality', 'Polish'), ('gender', 'female')])]
```

注意，Dataset 的 `find` 方法返回的实例是 `OrderedDict`。这些有用的容器扩展了 Python 的 `dict` 类，并且行为类似于字典，只是它们记住了插入项的顺序，这意味着你可以保证迭代的结果，弹出最后插入的项等操作。这是一个非常方便的附加功能。

###### 提示

对于数据处理者来说，最有用的Python“内置工具”之一是`collections`，其中包括数据集的`OrderedDict`。`defaultdict`和`Counter`类特别有用。请查看[Python文档](https://oreil.ly/Vh4EF)中提供的内容。

让我们使用Dataset重新创建我们的获奖者表，首先删除现有表：

```py
wtable = db['winners']
wtable.drop()

wtable = db['winners']
wtable.find()
#Out:
#[]
```

要重新创建我们删除的获奖者表，我们不需要像SQLAlchemy那样定义模式（请参阅[“定义数据库表”](#sql_schema)）。Dataset将从我们添加的数据中推断出模式，并隐式执行所有SQL创建。这是在使用基于集合的NoSQL数据库时所习惯的方便之一。让我们使用我们的`nobel_winners`数据集（[示例 3-1](#data_dummydata)）来插入一些获奖者字典。我们使用数据库事务和`with`语句来高效地插入对象，然后提交它们：^([7](ch03.xhtml#idm45607795163536))

```py
with db as tx: ![1](assets/1.png)
     tx['winners'].insert_many(nobel_winners)
```

[![1](assets/1.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO13-1)

使用`with`语句确保事务`tx`提交到数据库。

让我们检查一切是否顺利进行：

```py
list(db['winners'].find())
Out:
[OrderedDict([('id', 1), ('name', 'Albert Einstein'),
('category', 'Physics'), ('year', 1921), ('nationality',
'Swiss'), ('gender', 'male')]),
...
]
```

获奖者已经被正确插入，并且它们的插入顺序被`OrderedDict`保留。

数据集非常适合基于SQL的基本工作，特别是检索您可能希望处理或可视化的数据。对于更高级的操作，它允许您使用`query`方法进入SQLAlchemy的核心API。

现在我们已经掌握了使用SQL数据库的基础知识，让我们看看Python如何使得与最流行的NoSQL数据库一样轻松。

# MongoDB

像MongoDB这样以文档为中心的数据存储为数据处理者提供了很多便利。与所有工具一样，NoSQL数据库有好的和坏的用例。如果您的数据已经经过精炼和处理，并且不预期需要基于优化表连接的SQL强大查询语言，那么MongoDB可能最初会更容易使用。MongoDB非常适合Web数据可视化，因为它使用二进制JSON（BSON）作为其数据格式。BSON是JSON的扩展，可以处理二进制数据和`datetime`对象，并且与JavaScript非常兼容。

让我们再次回顾一下我们要写入和读取的目标数据集：

```py
nobel_winners = [
 {'category': 'Physics',
  'name': 'Albert Einstein',
  'nationality': 'Swiss',
  'gender': 'male',
  'year': 1921},
  ...
]
```

使用Python创建MongoDB集合只需几行代码：

```py
from pymongo import MongoClient

client = MongoClient() ![1](assets/1.png)
db = client.nobel_prize ![2](assets/2.png)
coll = db.winners ![3](assets/3.png)
```

[![1](assets/1.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO14-1)

创建Mongo客户端，使用默认主机和端口。

[![2](assets/2.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO14-2)

创建或访问`nobel_prize`数据库。

[![3](assets/3.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO14-3)

如果获奖者集合存在，则将其检索出来；否则（如我们的情况），它会创建它。

MongoDB 数据库默认在本地主机端口 27017 上运行，但也可能在网络上的任何地方。它们还可以使用可选的用户名和密码。[示例 3-5](#data_get_mongo)展示了如何创建一个简单的实用函数来访问我们的数据库，使用标准默认值。

##### 示例 3-5\. 访问 MongoDB 数据库

```py
from pymongo import MongoClient

def get_mongo_database(db_name, host='localhost',\
                       port=27017, username=None, password=None):
    """ Get named database from MongoDB with/out authentication """
    # make Mongo connection with/out authentication
    if username and password:
        mongo_uri = 'mongodb://%s:%s@%s/%s'%\ ![1](assets/1.png)
        (username, password, host, db_name)
        conn = MongoClient(mongo_uri)
    else:
        conn = MongoClient(host, port)

    return conn[db_name]
```

[![1](assets/1.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO15-1)

我们在 MongoDB URI（统一资源标识符）中指定数据库名称，因为用户可能没有对数据库的通用权限。

现在我们可以创建一个诺贝尔奖数据库并添加我们的目标数据集（[示例 3-1](#data_dummydata)）。让我们首先获取一个获奖者集合，使用访问的字符串常量：

```py
db = get_mongo_database(DB_NOBEL_PRIZE)
coll = db[COLL_WINNERS]
```

插入我们的诺贝尔奖数据集就像变得如此容易：

```py
coll.insert_many(nobel_winners)
coll.find()
Out:
[{'_id': ObjectId('61940b7dc454e79ffb14cd25'),
  'category': 'Physics',
  'name': 'Albert Einstein',
  'nationality': 'Swiss',
  'year': 1921,
  'gender': 'male'},
 {'_id': ObjectId('61940b7dc454e79ffb14cd26'), ... }
 ...]
```

结果数组的`ObjectId`可以用于将来的检索，但 MongoDB 已经在我们的`nobel_winners`列表上留下了自己的印记，添加了一个隐藏的`id`属性。^([8](ch03.xhtml#idm45607794582448))

###### 提示

MongoDB 的`ObjectId`具有相当多的隐藏功能，不仅仅是一个简单的随机标识符。例如，您可以获取`ObjectId`的生成时间，这使您可以访问一个方便的时间戳：

```py
import bson
oid = bson.ObjectId()
oid.generation_time
Out: datetime.datetime(2015, 11, 4, 15, 43, 23...
```

在[MongoDB BSON 文档](https://oreil.ly/NBwsk)中找到完整的详细信息。

现在我们已经在我们的获奖者集合中有了一些项目，使用它的`find`方法非常容易找到它们，使用一个字典查询：

```py
res = coll.find({'category':'Chemistry'})
list(res)
Out:
[{'_id': ObjectId('55f8326f26a7112e547879d6'),
  'category': 'Chemistry',
  'name': 'Marie Curie',
  'nationality': 'Polish',
  'gender': 'female',
  'year': 1911}]
```

有许多特殊的以美元为前缀的运算符，允许进行复杂的查询。让我们使用`$gt`（大于）运算符找到 1930 年后的所有获奖者：

```py
res = coll.find({'year': {'$gt': 1930}})
list(res)
Out:
[{'_id': ObjectId('55f8326f26a7112e547879d5'),
  'category': 'Physics',
  'name': 'Paul Dirac',
  'nationality': 'British',
  'gender': 'male',
  'year': 1933}]
```

您还可以使用布尔表达式，例如，查找所有 1930 年后的获奖者或所有女性获奖者：

```py
res = coll.find({'$or':[{'year': {'$gt': 1930}},\
{'gender':'female'}]})
list(res)
Out:
[{'_id': ObjectId('55f8326f26a7112e547879d5'),
  'category': 'Physics',
  'name': 'Paul Dirac',
  'nationality': 'British',
  'gender': 'male',
  'year': 1933},
 {'_id': ObjectId('55f8326f26a7112e547879d6'),
  'category': 'Chemistry',
  'name': 'Marie Curie',
  'nationality': 'Polish',
  'gender': 'female',
  'year': 1911}]
```

您可以在[MongoDB 文档](https://oreil.ly/1D2Sr)中找到可用查询表达式的完整列表。

作为最终测试，让我们将我们的新获奖者集合转换回 Python 字典列表。我们将为此创建一个实用函数：

```py
def mongo_coll_to_dicts(dbname='test', collname='test',\
                        query={}, del_id=True, **kw): ![1](assets/1.png)

    db = get_mongo_database(dbname, **kw)
    res = list(db[collname].find(query))

    if del_id:
        for r in res:
            r.pop('_id')

    return res
```

[![1](assets/1.png)](#co_reading_and_writing_data__span_class__keep_together__with_python__span__CO16-1)

一个空的`查询字典 {}`将在集合中找到所有文档。`del_id`是一个删除默认情况下项目中的 MongoDB`ObjectId`的标志。

现在我们可以创建我们的目标数据集：

```py
mongo_coll_to_dicts(DB_NOBEL_PRIZE, COLL_WINNERS)
Out:
[{'category': 'Physics',
  'name': 'Albert Einstein',
  'nationality': 'Swiss',
  'gender': 'male',
  'year': 1921},
  ...
]
```

MongoDB 的无模式数据库非常适合独自工作或小团队快速原型设计。可能会到达一个点，尤其是对于大型代码库，当正式模式成为一个有用的参考和理智检查时；在选择数据模型时，文档形式可以轻松适应的便利是一个优点。能够将 Python 字典作为查询传递给 PyMongo 并且可以访问客户端生成的`ObjectId`是其他一些便利的例子。

我们现在已经通过所有必需的文件格式和数据库传递了[示例 3-1](#data_dummydata)中的`nobel_winners`数据。让我们考虑处理日期和时间之前的特殊情况。

# 处理日期、时间和复杂数据

舒适地处理日期和时间是数据可视化工作的基础，但可能会相当棘手。有许多方法可以将日期或日期时间表示为字符串，每种方法都需要单独的编码或解码。因此，在自己的工作中选择一个格式并鼓励其他人也这样做是很好的。我建议使用 [国际标准化组织（ISO）8601 时间格式](https://oreil.ly/HePpN) 作为你的日期和时间的字符串表示，并使用 [协调世界时（UTC）形式](https://oreil.ly/neP2I)。^([9](ch03.xhtml#idm45607794006016)) 以下是几个 ISO 8601 日期和日期时间字符串的示例：

| 2021-09-23 | 日期（Python/C 格式代码 `'%Y-%m-%d'`） |
| --- | --- |
| 2021-09-23T16:32:35Z | 一个 UTC（时间后加上 *Z*）日期和时间（`'T%H:%M:%S'`） |
| 2021-09-23T16:32+02:00 | 与协调世界时（UTC）相比为正两小时的时区偏移量（+02:00）（例如，中欧时间） |

注意准备处理不同时区的重要性。时区并不总是在经线上（参见 [维基百科的时区条目](https://oreil.ly/NZyE4)），而通常推导出准确的时间的最佳方式是使用 UTC 时间加上地理位置。

ISO 8601 是 JavaScript 使用的标准，也很容易在 Python 中使用。作为网络数据可视化者，我们的主要关注点是创建一个字符串表示，该表示可以在 Python 和 JavaScript 之间通过 JSON 传递，并在两端轻松地编码和解码。

让我们以一个 Python `datetime` 的形式取一个日期和时间，将其转换为字符串，然后看看该字符串如何被 JavaScript 使用。

首先，我们生成我们的 Python `datetime`：

```py
from datetime import datetime

d = datetime.now()
d.isoformat()
Out:
'2021-11-16T22:55:48.738105'
```

然后，这个字符串可以保存到 JSON 或 CSV 中，被 JavaScript 读取，并用于创建一个 `Date` 对象：

```py
// JavaScript
d = new Date('2021-11-16T22:55:48.738105')
> Tue Nov 16 2021 22:55:48 GMT+0000 (Greenwich Mean Time)
```

我们可以使用 `toISOString` 方法将日期时间返回为 ISO 8601 字符串形式：

```py
// JavaScript
d.toISOString()
> '2021-11-16T22:55:48.738Z'
```

最后，我们可以将字符串读取回 Python。

如果你知道你正在处理的是 ISO 格式的时间字符串，Python 的 `dateutil` 模块应该可以胜任这个工作。^([10](ch03.xhtml#idm45607793910272)) 但你可能希望对结果进行一些合理性检查：

```py
from dateutil import parser

d = parser.parse('2021-11-16T22:55:48.738Z')
d
Out:
datetime.datetime(2021, 11, 16, 22, 55, 48, 738000,\
tzinfo=tzutc())
```

请注意，从 Python 到 JavaScript 再返回时，我们丢失了一些分辨率，后者处理的是毫秒，而不是微秒。在任何数据可视化工作中，这不太可能成为问题，但需要记住，以防出现一些奇怪的时间错误。

# 摘要

本章旨在使您能够舒适地使用 Python 在各种文件格式和数据库之间传输数据，这是数据可视化者可能会遇到的。有效和高效地使用数据库是一个需要一段时间学习的技能，但现在您应该对大多数数据可视化用例的基本读写感到满意。

现在我们已经为我们的数据可视化工具链提供了重要的润滑剂，让我们先快速掌握一下你在后面章节中所需的基本网络开发技能。

^([1](ch03.xhtml#idm45607798394048-marker)) 我建议您将JSON作为首选数据格式，而不是CSV。

^([2](ch03.xhtml#idm45607797415616-marker)) Python模块`dateutil`有一个解析器，可以合理地解析大多数日期和时间，可能是此操作的良好基础。

^([3](ch03.xhtml#idm45607796976800-marker)) 值得注意的是，在测试和生产环境中使用不同的数据库配置可能是个坏主意。

^([4](ch03.xhtml#idm45607796966400-marker)) 查看关于[SQLAlchemy](https://oreil.ly/winYu)的详细信息，了解*延迟初始化*。

^([5](ch03.xhtml#idm45607796702064-marker)) 这假设数据库尚不存在。如果存在，则将使用`Base`来创建新的插入和解释检索。

^([6](ch03.xhtml#idm45607795314160-marker)) Dataset的官方座右铭是“懒人数据库”。它不是标准Anaconda包的一部分，因此您需要通过命令行使用`pip`进行安装：`$ pip install dataset`。

^([7](ch03.xhtml#idm45607795163536-marker)) 详细了解如何使用事务来分组更新，请参阅[此文档](https://oreil.ly/vqvbv)。

^([8](ch03.xhtml#idm45607794582448-marker)) MongoDB的一个很酷的特性是`ObjectId`在客户端生成，无需查询数据库获取它们。

^([9](ch03.xhtml#idm45607794006016-marker)) 要从UTC获取实际本地时间，可以存储时区偏移量或更好地从地理坐标派生；这是因为时区并不完全按经度线精确地遵循。

^([10](ch03.xhtml#idm45607793910272-marker)) 要安装，只需运行`pip install python-dateutil`。`dateutil`是Python的`datetime`的一个相当强大的扩展；详细信息请查看[Read the Docs](https://oreil.ly/y6YWS)。
