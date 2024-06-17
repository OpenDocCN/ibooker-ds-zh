# 第九章. 获取数据

> 要写它，用了三个月；要构思它，用了三分钟；要收集其中的数据，用了一生。
> 
> F. 斯科特·菲茨杰拉德

要成为一名数据科学家，你需要数据。事实上，作为数据科学家，你将花费大量时间来获取、清理和转换数据。如果必要，你可以自己键入数据（或者如果有下属，让他们来做），但通常这不是你时间的好用法。在本章中，我们将探讨将数据引入Python及其转换为正确格式的不同方法。

# stdin和stdout

如果在命令行中运行Python脚本，你可以使用`sys.stdin`和`sys.stdout`将数据*管道*通过它们。例如，这是一个读取文本行并返回匹配正则表达式的脚本：

```py
# egrep.py
import sys, re

# sys.argv is the list of command-line arguments
# sys.argv[0] is the name of the program itself
# sys.argv[1] will be the regex specified at the command line
regex = sys.argv[1]

# for every line passed into the script
for line in sys.stdin:
    # if it matches the regex, write it to stdout
    if re.search(regex, line):
        sys.stdout.write(line)
```

这里有一个示例，它会计算接收到的行数并将其写出：

```py
# line_count.py
import sys

count = 0
for line in sys.stdin:
    count += 1

# print goes to sys.stdout
print(count)
```

然后你可以使用它们来计算文件中包含数字的行数。在Windows中，你会使用：

```py
type SomeFile.txt | python egrep.py "[0-9]" | python line_count.py
```

在Unix系统中，你会使用：

```py
cat SomeFile.txt | python egrep.py "[0-9]" | python line_count.py
```

管道符号`|`表示管道字符，意味着“使用左侧命令的输出作为右侧命令的输入”。你可以通过这种方式构建非常复杂的数据处理管道。

###### 注意

如果你使用Windows，你可能可以在该命令中省略`python`部分：

```py
type SomeFile.txt | egrep.py "[0-9]" | line_count.py
```

如果你在Unix系统上，这样做需要[几个额外步骤](https://stackoverflow.com/questions/15587877/run-a-python-script-in-terminal-without-the-python-command)。首先在你的脚本的第一行添加一个“shebang” `#!/usr/bin/env python`。然后，在命令行中使用`chmod` x egrep.py++将文件设为可执行。

同样地，这是一个计算其输入中单词数量并写出最常见单词的脚本：

```py
# most_common_words.py
import sys
from collections import Counter

# pass in number of words as first argument
try:
    num_words = int(sys.argv[1])
except:
    print("usage: most_common_words.py num_words")
    sys.exit(1)   # nonzero exit code indicates error

counter = Counter(word.lower()                      # lowercase words
                  for line in sys.stdin
                  for word in line.strip().split()  # split on spaces
                  if word)                          # skip empty 'words'

for word, count in counter.most_common(num_words):
    sys.stdout.write(str(count))
    sys.stdout.write("\t")
    sys.stdout.write(word)
    sys.stdout.write("\n")
```

然后你可以像这样做一些事情：

```py
$ cat the_bible.txt | python most_common_words.py 10
36397	the
30031	and
20163	of
7154	to
6484	in
5856	that
5421	he
5226	his
5060	unto
4297	shall
```

（如果你使用Windows，则使用`type`而不是`cat`。）

###### 注意

如果你是一名经验丰富的Unix程序员，可能已经熟悉各种命令行工具（例如，`egrep`），这些工具已经内建到你的操作系统中，比从头开始构建更可取。不过，了解自己可以这样做也是很好的。

# 读取文件

你也可以在代码中直接显式地读取和写入文件。Python使得处理文件变得非常简单。

## 文本文件的基础知识

处理文本文件的第一步是使用`open`获取一个*文件对象*：

```py
# 'r' means read-only, it's assumed if you leave it out
file_for_reading = open('reading_file.txt', 'r')
file_for_reading2 = open('reading_file.txt')

# 'w' is write -- will destroy the file if it already exists!
file_for_writing = open('writing_file.txt', 'w')

# 'a' is append -- for adding to the end of the file
file_for_appending = open('appending_file.txt', 'a')

# don't forget to close your files when you're done
file_for_writing.close()
```

因为很容易忘记关闭文件，所以你应该总是在`with`块中使用它们，在块结束时它们将自动关闭：

```py
with open(filename) as f:
    data = function_that_gets_data_from(f)

# at this point f has already been closed, so don't try to use it
process(data)
```

如果你需要读取整个文本文件，可以使用`for`循环迭代文件的每一行：

```py
starts_with_hash = 0

with open('input.txt') as f:
    for line in f:                  # look at each line in the file
        if re.match("^#",line):     # use a regex to see if it starts with '#'
            starts_with_hash += 1   # if it does, add 1 to the count
```

通过这种方式获取的每一行都以换行符结尾，所以在处理之前通常会将其`strip`掉。

例如，假设你有一个文件，其中包含一个邮箱地址一行，你需要生成一个域名的直方图。正确提取域名的规则有些微妙，可以参考[公共后缀列表](https://publicsuffix.org)，但一个很好的初步方法是仅仅取邮箱地址中“@”后面的部分（对于像*joel@mail.datasciencester.com*这样的邮箱地址，这个方法会给出错误的答案，但在这个例子中我们可以接受这种方法）：

```py
def get_domain(email_address: str) -> str:
    """Split on '@' and return the last piece"""
    return email_address.lower().split("@")[-1]

# a couple of tests
assert get_domain('joelgrus@gmail.com') == 'gmail.com'
assert get_domain('joel@m.datasciencester.com') == 'm.datasciencester.com'

from collections import Counter

with open('email_addresses.txt', 'r') as f:
    domain_counts = Counter(get_domain(line.strip())
                            for line in f
                            if "@" in line)
```

## 分隔文件

我们刚刚处理的假设的邮箱地址文件每行一个地址。更频繁地，你将使用每行有大量数据的文件。这些文件往往是逗号分隔或制表符分隔的：每行有多个字段，逗号或制表符表示一个字段的结束和下一个字段的开始。

当你的字段中有逗号、制表符和换行符时（这是不可避免的）。因此，你不应该尝试自己解析它们。相反，你应该使用Python的`csv`模块（或pandas库，或设计用于读取逗号分隔或制表符分隔文件的其他库）。

###### 警告

永远不要自己解析逗号分隔的文件。你会搞砸一些边缘情况！

如果你的文件没有表头（这意味着你可能希望每行作为一个`list`，并且需要你知道每一列中包含什么），你可以使用`csv.reader`来迭代行，每行都会是一个适当拆分的列表。

例如，如果我们有一个制表符分隔的股票价格文件：

```py
6/20/2014   AAPL    90.91
6/20/2014   MSFT    41.68
6/20/2014   FB  64.5
6/19/2014   AAPL    91.86
6/19/2014   MSFT    41.51
6/19/2014   FB  64.34
```

我们可以用以下方式处理它们：

```py
import csv

with open('tab_delimited_stock_prices.txt') as f:
    tab_reader = csv.reader(f, delimiter='\t')
    for row in tab_reader:
        date = row[0]
        symbol = row[1]
        closing_price = float(row[2])
        process(date, symbol, closing_price)
```

如果你的文件有表头：

```py
date:symbol:closing_price
6/20/2014:AAPL:90.91
6/20/2014:MSFT:41.68
6/20/2014:FB:64.5
```

你可以通过初始调用`reader.next`跳过表头行，或者通过使用`csv.DictReader`将每一行作为`dict`（表头作为键）来获取：

```py
with open('colon_delimited_stock_prices.txt') as f:
    colon_reader = csv.DictReader(f, delimiter=':')
    for dict_row in colon_reader:
        date = dict_row["date"]
        symbol = dict_row["symbol"]
        closing_price = float(dict_row["closing_price"])
        process(date, symbol, closing_price)
```

即使你的文件没有表头，你仍然可以通过将键作为`fieldnames`参数传递给`DictReader`来使用它。

你也可以使用`csv.writer`类似地写出分隔数据：

```py
todays_prices = {'AAPL': 90.91, 'MSFT': 41.68, 'FB': 64.5 }

with open('comma_delimited_stock_prices.txt', 'w') as f:
    csv_writer = csv.writer(f, delimiter=',')
    for stock, price in todays_prices.items():
        csv_writer.writerow([stock, price])
```

如果你的字段本身包含逗号，`csv.writer`会处理得很好。但是，如果你自己手动编写的写入器可能不会。例如，如果你尝试：

```py
results = [["test1", "success", "Monday"],
           ["test2", "success, kind of", "Tuesday"],
           ["test3", "failure, kind of", "Wednesday"],
           ["test4", "failure, utter", "Thursday"]]

# don't do this!
with open('bad_csv.txt', 'w') as f:
    for row in results:
        f.write(",".join(map(str, row))) # might have too many commas in it!
        f.write("\n")                    # row might have newlines as well!
```

你将会得到一个如下的*.csv*文件：

```py
test1,success,Monday
test2,success, kind of,Tuesday
test3,failure, kind of,Wednesday
test4,failure, utter,Thursday
```

而且没有人能够理解。

# 网页抓取

另一种获取数据的方式是从网页中抓取数据。事实证明，获取网页很容易；但从中获取有意义的结构化信息却不那么容易。

## HTML及其解析

网页是用HTML编写的，文本（理想情况下）被标记为元素及其属性：

```py
<html>
  <head>
    <title>A web page</title>
  </head>
  <body>
    <p id="author">Joel Grus</p>
    <p id="subject">Data Science</p>
  </body>
</html>
```

在一个完美的世界中，所有网页都会被语义化地标记，为了我们的利益。我们将能够使用诸如“查找`id`为`subject`的`<p>`元素并返回其包含的文本”之类的规则来提取数据。但实际上，HTML通常并不规范，更不用说注释了。这意味着我们需要帮助来理解它。

要从HTML中获取数据，我们将使用[Beautiful Soup库](http://www.crummy.com/software/BeautifulSoup/)，它会构建一个网页上各种元素的树，并提供一个简单的接口来访问它们。在我写这篇文章时，最新版本是Beautiful Soup 4.6.0，这也是我们将使用的版本。我们还将使用[Requests库](http://docs.python-requests.org/en/latest/)，这是一种比Python内置的任何东西都更好的方式来进行HTTP请求。

Python内置的HTML解析器并不那么宽容，这意味着它不能很好地处理不完全形式的HTML。因此，我们还将安装`html5lib`解析器。

确保您处于正确的虚拟环境中，安装库：

```py
python -m pip install beautifulsoup4 requests html5lib
```

要使用Beautiful Soup，我们将一个包含HTML的字符串传递给`BeautifulSoup`函数。在我们的示例中，这将是对`requests.get`调用的结果：

```py
from bs4 import BeautifulSoup
import requests

# I put the relevant HTML file on GitHub. In order to fit
# the URL in the book I had to split it across two lines.
# Recall that whitespace-separated strings get concatenated.
url = ("https://raw.githubusercontent.com/"
       "joelgrus/data/master/getting-data.html")
html = requests.get(url).text
soup = BeautifulSoup(html, 'html5lib')
```

然后我们可以使用几种简单的方法走得相当远。

我们通常会使用`Tag`对象，它对应于表示HTML页面结构的标签。

例如，要找到第一个`<p>`标签（及其内容），您可以使用：

```py
first_paragraph = soup.find('p')        # or just soup.p
```

您可以使用其`text`属性获取`Tag`的文本内容：

```py
first_paragraph_text = soup.p.text
first_paragraph_words = soup.p.text.split()
```

您可以通过将其视为`dict`来提取标签的属性：

```py
first_paragraph_id = soup.p['id']       # raises KeyError if no 'id'
first_paragraph_id2 = soup.p.get('id')  # returns None if no 'id'
```

您可以按以下方式一次获取多个标签：

```py
all_paragraphs = soup.find_all('p')  # or just soup('p')
paragraphs_with_ids = [p for p in soup('p') if p.get('id')]
```

经常，您会想要找到具有特定`class`的标签：

```py
important_paragraphs = soup('p', {'class' : 'important'})
important_paragraphs2 = soup('p', 'important')
important_paragraphs3 = [p for p in soup('p')
                         if 'important' in p.get('class', [])]
```

你可以结合这些方法来实现更复杂的逻辑。例如，如果你想找到每个包含在`<div>`元素内的`<span>`元素，你可以这样做：

```py
# Warning: will return the same <span> multiple times
# if it sits inside multiple <div>s.
# Be more clever if that's the case.
spans_inside_divs = [span
                     for div in soup('div')     # for each <div> on the page
                     for span in div('span')]   # find each <span> inside it
```

这些功能的几个特点就足以让我们做很多事情。如果你最终需要做更复杂的事情（或者你只是好奇），请查阅[文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)。

当然，重要的数据通常不会标记为`class="important"`。您需要仔细检查源HTML，通过选择逻辑推理，并担心边缘情况，以确保数据正确。让我们看一个例子。

## 例如：监控国会

DataSciencester的政策副总裁担心数据科学行业可能会受到监管，并要求您量化国会在该主题上的言论。特别是，他希望您找出所有发表关于“数据”内容的代表。

在发布时，有一个页面链接到所有代表的网站，网址为[*https://www.house.gov/representatives*](https://www.house.gov/representatives)。

如果“查看源代码”，所有指向网站的链接看起来像：

```py
<td>
  <a href="https://jayapal.house.gov">Jayapal, Pramila</a>
</td>
```

让我们开始收集从该页面链接到的所有URL：

```py
from bs4 import BeautifulSoup
import requests

url = "https://www.house.gov/representatives"
text = requests.get(url).text
soup = BeautifulSoup(text, "html5lib")

all_urls = [a['href']
            for a in soup('a')
            if a.has_attr('href')]

print(len(all_urls))  # 965 for me, way too many
```

这返回了太多的URL。如果你查看它们，我们想要的URL以*http://*或*https://*开头，有一些名称，并且以*.house.gov*或*.house.gov/*结尾。

这是使用正则表达式的好地方：

```py
import re

# Must start with http:// or https://
# Must end with .house.gov or .house.gov/
regex = r"^https?://.*\.house\.gov/?$"

# Let's write some tests!
assert re.match(regex, "http://joel.house.gov")
assert re.match(regex, "https://joel.house.gov")
assert re.match(regex, "http://joel.house.gov/")
assert re.match(regex, "https://joel.house.gov/")
assert not re.match(regex, "joel.house.gov")
assert not re.match(regex, "http://joel.house.com")
assert not re.match(regex, "https://joel.house.gov/biography")

# And now apply
good_urls = [url for url in all_urls if re.match(regex, url)]

print(len(good_urls))  # still 862 for me
```

这仍然太多了，因为只有435位代表。如果你看一下列表，会发现很多重复。让我们使用`set`来去重：

```py
good_urls = list(set(good_urls))

print(len(good_urls))  # only 431 for me
```

总会有几个众议院席位是空缺的，或者可能有一个没有网站的代表。无论如何，这已经足够了。当我们查看这些站点时，大多数都有一个指向新闻稿的链接。例如：

```py
html = requests.get('https://jayapal.house.gov').text
soup = BeautifulSoup(html, 'html5lib')

# Use a set because the links might appear multiple times.
links = {a['href'] for a in soup('a') if 'press releases' in a.text.lower()}

print(links) # {'/media/press-releases'}
```

注意这是一个相对链接，这意味着我们需要记住原始站点。让我们来做一些抓取：

```py
from typing import Dict, Set

press_releases: Dict[str, Set[str]] = {}

for house_url in good_urls:
    html = requests.get(house_url).text
    soup = BeautifulSoup(html, 'html5lib')
    pr_links = {a['href'] for a in soup('a') if 'press releases'
                                             in a.text.lower()}
    print(f"{house_url}: {pr_links}")
    press_releases[house_url] = pr_links
```

###### 注意

通常情况下，自由地抓取一个网站是不礼貌的。大多数网站会有一个*robots.txt*文件，指示您可以多频繁地抓取该站点（以及您不应该抓取的路径），但由于涉及到国会，我们不需要特别礼貌。

如果你看这些内容滚动显示，你会看到很多`/media/press-releases`和`media-center/press-releases`，以及各种其他地址。其中一个URL是[*https://jayapal.house.gov/media/press-releases*](https://jayapal.house.gov/media/press-releases)。

请记住，我们的目标是找出哪些国会议员在其新闻稿中提到了“数据”。我们将编写一个稍微更通用的函数，检查新闻稿页面是否提到了任何给定的术语。

如果你访问该网站并查看源代码，似乎每篇新闻稿都有一个在`<p>`标签中的片段，所以我们将用它作为我们的第一个尝试：

```py
def paragraph_mentions(text: str, keyword: str) -> bool:
    """
 Returns True if a <p> inside the text mentions {keyword}
 """
    soup = BeautifulSoup(text, 'html5lib')
    paragraphs = [p.get_text() for p in soup('p')]

    return any(keyword.lower() in paragraph.lower()
               for paragraph in paragraphs)
```

让我们为此写一个快速的测试：

```py
text = """<body><h1>Facebook</h1><p>Twitter</p>"""
assert paragraph_mentions(text, "twitter")       # is inside a <p>
assert not paragraph_mentions(text, "facebook")  # not inside a <p>
```

最后，我们准备好找到相关的国会议员，并把他们的名字交给副总裁：

```py
for house_url, pr_links in press_releases.items():
    for pr_link in pr_links:
        url = f"{house_url}/{pr_link}"
        text = requests.get(url).text

        if paragraph_mentions(text, 'data'):
            print(f"{house_url}")
            break  # done with this house_url
```

当我运行这个时，我得到了大约20位代表的列表。你的结果可能会有所不同。

###### 注意

如果你看各种“新闻稿”页面，大多数页面都是分页的，每页只有5或10篇新闻稿。这意味着我们只检索了每位国会议员最近的几篇新闻稿。更彻底的解决方案将迭代每一页，并检索每篇新闻稿的全文。

# 使用API

许多网站和Web服务提供*应用程序编程接口*（API），允许您以结构化格式显式请求数据。这样可以避免您必须进行抓取的麻烦！

## JSON和XML

因为HTTP是一个用于传输*文本*的协议，通过Web API请求的数据需要被*序列化*为字符串格式。通常这种序列化使用*JavaScript对象表示法*（JSON）。JavaScript对象看起来非常类似于Python的`dict`，这使得它们的字符串表示易于解释：

```py
{ "title" : "Data Science Book",
  "author" : "Joel Grus",
  "publicationYear" : 2019,
  "topics" : [ "data", "science", "data science"] }
```

我们可以使用Python的`json`模块解析JSON。特别地，我们将使用它的`loads`函数，将表示JSON对象的字符串反序列化为Python对象：

```py
import json
serialized = """{ "title" : "Data Science Book",
 "author" : "Joel Grus",
 "publicationYear" : 2019,
 "topics" : [ "data", "science", "data science"] }"""

# parse the JSON to create a Python dict
deserialized = json.loads(serialized)
assert deserialized["publicationYear"] == 2019
assert "data science" in deserialized["topics"]
```

有时API提供者会讨厌你，并且只提供XML格式的响应：

```py
<Book>
  <Title>Data Science Book</Title>
  <Author>Joel Grus</Author>
  <PublicationYear>2014</PublicationYear>
  <Topics>
    <Topic>data</Topic>
    <Topic>science</Topic>
    <Topic>data science</Topic>
  </Topics>
</Book>
```

你可以像从HTML中获取数据那样，使用Beautiful Soup从XML中获取数据；请查看其文档以获取详细信息。

## 使用未经身份验证的API

大多数API现在要求你先进行身份验证，然后才能使用它们。虽然我们不反对这种策略，但这会产生很多额外的样板代码，使我们的解释变得混乱。因此，我们将首先看一下[GitHub的API](http://developer.github.com/v3/)，它可以让你无需身份验证就能进行一些简单的操作：

```py
import requests, json

github_user = "joelgrus"
endpoint = f"https://api.github.com/users/{github_user}/repos"

repos = json.loads(requests.get(endpoint).text)
```

此时`repos`是我GitHub账户中的公共仓库的Python `dict`列表。（随意替换你的用户名并获取你的GitHub仓库数据。你有GitHub账户，对吧？）

我们可以用这个来找出我最有可能创建仓库的月份和星期几。唯一的问题是响应中的日期是字符串：

```py
"created_at": "2013-07-05T02:02:28Z"
```

Python自带的日期解析器不是很好用，所以我们需要安装一个：

```py
python -m pip install python-dateutil
```

其中你可能只会需要`dateutil.parser.parse`函数：

```py
from collections import Counter
from dateutil.parser import parse

dates = [parse(repo["created_at"]) for repo in repos]
month_counts = Counter(date.month for date in dates)
weekday_counts = Counter(date.weekday() for date in dates)
```

同样地，你可以获取我最近五个仓库的语言：

```py
last_5_repositories = sorted(repos,
                             key=lambda r: r["pushed_at"],
                             reverse=True)[:5]

last_5_languages = [repo["language"]
                    for repo in last_5_repositories]
```

通常情况下，我们不会在低层次（“自己发起请求并解析响应”）处理API。使用Python的好处之一是，几乎任何你有兴趣访问的API，都已经有人建立了一个库。如果做得好，这些库可以节省你很多访问API的复杂细节的麻烦。（如果做得不好，或者当它们基于已失效的API版本时，可能会带来巨大的麻烦。）

尽管如此，偶尔你会需要自己编写API访问库（或者更有可能，调试为什么别人的库不起作用），因此了解一些细节是很有用的。

## 寻找API

如果你需要从特定网站获取数据，请查找该网站的“开发者”或“API”部分以获取详细信息，并尝试在网上搜索“python <sitename> api”来找到相应的库。

有关Yelp API、Instagram API、Spotify API等等，都有相应的库。

如果你在寻找Python封装的API列表，[Real Python在GitHub上](https://github.com/realpython/list-of-python-api-wrappers)有一个很好的列表。

如果找不到你需要的内容，总有一种方法，那就是网页抓取，数据科学家的最后避风港。

# 示例：使用Twitter的API

Twitter是一个非常好的数据来源。你可以用它来获取实时新闻，也可以用它来衡量对当前事件的反应。你还可以用它来查找与特定主题相关的链接。你可以用它来做几乎任何你能想到的事情，只要你能访问到它的数据。通过它的API，你可以获取到它的数据。

要与Twitter的API交互，我们将使用[Twython库](https://github.com/ryanmcgrath/twython)（`python -m pip install twython`）。目前有许多Python Twitter库，但这是我使用最成功的一个。当然，也鼓励你探索其他库！

## 获取凭证

为了使用 Twitter 的 API，你需要获取一些凭据（你需要一个 Twitter 帐户，这样你就可以成为活跃且友好的 Twitter #datascience 社区的一部分）。

###### 警告

像所有与我无法控制的网站相关的说明一样，这些说明可能在某个时候过时，但希望能够一段时间内工作。（尽管自我最初开始写这本书以来，它们已经多次发生变化，所以祝你好运！）

以下是步骤：

1.  前往 [*https://developer.twitter.com/*](https://developer.twitter.com/)。

1.  如果你没有登录，点击“登录”并输入你的 Twitter 用户名和密码。

1.  点击申请以申请开发者帐户。

1.  为你自己的个人使用请求访问。

1.  填写申请。 它需要 300 字（真的）解释为什么你需要访问，所以为了超过限制，你可以告诉他们关于这本书以及你有多么喜欢它。

1.  等待一段不确定的时间。

1.  如果你认识在 Twitter 工作的人，请给他们发电子邮件，询问他们是否可以加快你的申请。 否则，继续等待。

1.  一旦你获得批准，返回到 [developer.twitter.com](https://developer.twitter.com/)，找到“Apps”部分，然后点击“创建应用程序”。

1.  填写所有必填字段（同样，如果你需要描述的额外字符，你可以谈论这本书以及你发现它多么有启发性）。

1.  点击创建。

现在你的应用程序应该有一个“Keys and tokens”选项卡，其中包含一个“Consumer API keys”部分，列出了一个“API key”和一个“API secret key”。 记下这些密钥； 你会需要它们。（另外，保持它们保密！ 它们就像密码。）

###### 小心

不要分享密钥，不要在书中发布它们，也不要将它们检入你的公共 GitHub 存储库。 一个简单的解决方案是将它们存储在一个不会被检入的 *credentials.json* 文件中，并让你的代码使用 `json.loads` 来检索它们。 另一个解决方案是将它们存储在环境变量中，并使用 `os.environ` 来检索它们。

### 使用 Twython

使用 Twitter API 的最棘手的部分是验证身份。（事实上，这是使用许多 API 中最棘手的部分之一。） API 提供商希望确保你被授权访问他们的数据，并且你不会超出他们的使用限制。 他们还想知道谁在访问他们的数据。

认证有点痛苦。 有一种简单的方法，OAuth 2，在你只想做简单搜索时足够使用。 还有一种复杂的方法，OAuth 1，在你想执行操作（例如，发推文）或（特别是对我们来说）连接到 Twitter 流时需要使用。

所以我们被迫使用更复杂的方式，我们会尽可能自动化它。

首先，你需要你的 API 密钥和 API 密钥（有时也称为消费者密钥和消费者密钥）。 我将从环境变量中获取我的，但请随意以任何你希望的方式替换你的：

```py
import os

# Feel free to plug your key and secret in directly
CONSUMER_KEY = os.environ.get("TWITTER_CONSUMER_KEY")
CONSUMER_SECRET = os.environ.get("TWITTER_CONSUMER_SECRET")
```

现在我们可以实例化客户端：

```py
import webbrowser
from twython import Twython

# Get a temporary client to retrieve an authentication URL
temp_client = Twython(CONSUMER_KEY, CONSUMER_SECRET)
temp_creds = temp_client.get_authentication_tokens()
url = temp_creds['auth_url']

# Now visit that URL to authorize the application and get a PIN
print(f"go visit {url} and get the PIN code and paste it below")
webbrowser.open(url)
PIN_CODE = input("please enter the PIN code: ")

# Now we use that PIN_CODE to get the actual tokens
auth_client = Twython(CONSUMER_KEY,
                      CONSUMER_SECRET,
                      temp_creds['oauth_token'],
                      temp_creds['oauth_token_secret'])
final_step = auth_client.get_authorized_tokens(PIN_CODE)
ACCESS_TOKEN = final_step['oauth_token']
ACCESS_TOKEN_SECRET = final_step['oauth_token_secret']

# And get a new Twython instance using them.
twitter = Twython(CONSUMER_KEY,
                  CONSUMER_SECRET,
                  ACCESS_TOKEN,
                  ACCESS_TOKEN_SECRET)
```

###### 提示

此时，你可能希望考虑将`ACCESS_TOKEN`和`ACCESS_TOKEN_SECRET`保存在安全的地方，这样下次你就不必再经历这个烦琐的过程了。

一旦我们有了一个经过身份验证的`Twython`实例，我们就可以开始执行搜索：

```py
# Search for tweets containing the phrase "data science"
for status in twitter.search(q='"data science"')["statuses"]:
    user = status["user"]["screen_name"]
    text = status["text"]
    print(f"{user}: {text}\n")
```

如果你运行这个程序，你应该会得到一些推文，比如：

```py
haithemnyc: Data scientists with the technical savvy &amp; analytical chops to
derive meaning from big data are in demand. http://t.co/HsF9Q0dShP

RPubsRecent: Data Science http://t.co/6hcHUz2PHM

spleonard1: Using #dplyr in #R to work through a procrastinated assignment for
@rdpeng in @coursera data science specialization. So easy and Awesome.
```

这并不那么有趣，主要是因为Twitter搜索API只会显示出它觉得最近的结果。在进行数据科学时，更多时候你会想要大量的推文。这就是[Streaming API](https://developer.twitter.com/en/docs/tutorials/consuming-streaming-data)有用的地方。它允许你连接到（部分）巨大的Twitter firehose。要使用它，你需要使用你的访问令牌进行身份验证。

为了使用Twython访问Streaming API，我们需要定义一个类，该类继承自`TwythonStreamer`并重写其`on_success`方法，可能还有其`on_error`方法：

```py
from twython import TwythonStreamer

# Appending data to a global variable is pretty poor form
# but it makes the example much simpler
tweets = []

class MyStreamer(TwythonStreamer):
    def on_success(self, data):
        """
 What do we do when Twitter sends us data?
 Here data will be a Python dict representing a tweet.
 """
        # We only want to collect English-language tweets
        if data.get('lang') == 'en':
            tweets.append(data)
            print(f"received tweet #{len(tweets)}")

        # Stop when we've collected enough
        if len(tweets) >= 100:
            self.disconnect()

    def on_error(self, status_code, data):
        print(status_code, data)
        self.disconnect()
```

`MyStreamer`将连接到Twitter流并等待Twitter提供数据。每次接收到一些数据（在这里是表示为Python对象的推文）时，它都会将其传递给`on_success`方法，如果推文的语言是英语，则将其追加到我们的`tweets`列表中，然后在收集到1,000条推文后断开流。

唯一剩下的就是初始化它并开始运行：

```py
stream = MyStreamer(CONSUMER_KEY, CONSUMER_SECRET,
                    ACCESS_TOKEN, ACCESS_TOKEN_SECRET)

# starts consuming public statuses that contain the keyword 'data'
stream.statuses.filter(track='data')

# if instead we wanted to start consuming a sample of *all* public statuses
# stream.statuses.sample()
```

这将持续运行，直到收集到100条推文（或遇到错误为止），然后停止，此时你可以开始分析这些推文。例如，你可以找出最常见的标签：

```py
top_hashtags = Counter(hashtag['text'].lower()
                       for tweet in tweets
                       for hashtag in tweet["entities"]["hashtags"])

print(top_hashtags.most_common(5))
```

每条推文都包含大量的数据。你可以自己探索，或者查看[Twitter API文档](https://developer.twitter.com/en/docs/tweets/data-dictionary/overview/tweet-object)。

###### 注意

在一个非玩具项目中，你可能不想依赖于内存中的`list`来存储推文。相反，你可能想把它们保存到文件或数据库中，这样你就能永久地拥有它们。

# 进一步探索

+   [pandas](http://pandas.pydata.org/)是数据科学家们用来处理数据，特别是导入数据的主要库。

+   [Scrapy](http://scrapy.org/)是一个用于构建复杂网络爬虫的全功能库，可以执行诸如跟踪未知链接等操作。

+   [Kaggle](https://www.kaggle.com/datasets)拥有大量的数据集。
